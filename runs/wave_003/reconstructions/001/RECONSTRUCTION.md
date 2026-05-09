## System-level intent

- **Load-bearing rules as invariants, not defaults.** The plan says the PRD's rules are "invariants, not defaults" and that any line relaxing them "should be read as a mistake and corrected." The pinned summary repeats that the rules are "the substance of the product" and that code, schema, API, and build pipeline should make them "invariants, not aspirations."

- **Server-authoritative continuity.** The plan's center is "the server is the only writer of personality state," "Postgres holds the canonical state," and clients "never write personality state." This shows up in the architecture, API schemas, event log, sync model, database roles, and pinned rules.

- **Ambient relationship rather than engagement game.** The product refuses "gamification of any flavor," "notifications by default," "Welcome back" surfaces, visit-frequency widgets, and social-network surfaces. The voice is "notice, never announce"; neglect produces "ambient quietness, not punishment."

- **Felt aliveness over time.** The plan repeatedly protects "the aviary appears already in motion," "the aviary the user comes back to is the aviary that has been running," and "feels alive over weeks." This appears in the first-frame work, server tick, catch-up tick, drift calibration, persistent mood, and no-spinner loading state.

- **Privacy as an architectural property.** The plan uses "synthetic UUIDs," keeps email encrypted on one row, forbids per-account and per-bird analytics dimensions, and says the telemetry pipeline has no credentials to read the simulation database. It frames privacy as "architectural; not a policy."

- **Naturalist, quiet product voice.** The notebook and narration use "naturalist prose," "lowercase, present-tense," "specific particulars," and avoid "announcement framing." Settings, unsupported browser, audio failure, and email recovery are "matter-of-fact."

- **Procedural recognizability over canned media.** Calls are procedural, per-bird signatures are "individually recognizable," chorus must be "real chorus," and looped or recorded fallback audio is treated as the "audible signature of dead software."

- **Accessibility as a designed surface.** The plan states accessibility is "a designed surface, not a checklist" and reduced motion is "a designed alternate rendering, not a stripped fallback." This appears in narration, captions, keyboard navigation, contrast, ARIA, and CI.

- **Small, strict boundaries.** The plan says "the product is small; the architecture should be small." It uses fewer services, a "thin" gateway, a strict simulation/rendering boundary, push-only WebSocket, route-splitting, and narrow write/audit surfaces.

- **Budgets and tests make product intent enforceable.** Bundle size, first-bird time, 60fps idle motion, no memory growth, tick latency, a11y CI, drift harnesses, replay tests, and alarms turn product qualities into checked properties instead of hopes.

## Per-feature whys

### Scope and product surface

- **Single-user accounts via email magic link.** NOT RECOVERABLE FROM PLAN

- **One canonical aviary per account.** The plan ties this to sync: multi-device sync is "a property of the architecture," because all devices read snapshots from "one canonical record."

- **Two starter birds, cap of seven, and aviary-age-paced offers.** The plan says the ramp follows "aviary age, not engagement" and matches the "rhythm of a relationship deepening." Super-linear intervals make early growth visible and later growth rarer.

- **Six-species count.** NOT RECOVERABLE FROM PLAN

- **Server-side simulation tick.** The tick keeps mood, drift, and call-event scheduling fresh while ensuring the aviary continues without the viewer. The live/dormant/catch-up design keeps the experience continuous and "cost flat as users dip in and out."

- **Slow personality drift across five traits.** The why is the central "feels alive over weeks" promise: drift should be "measurable at week 1, visible at week 3" while remaining monotonic toward expressive.

- **Mood as a fast-timescale persisted state.** Mood is the visible variable that can change without turning neglect into negative personality drift. Persisting mood means the user never sees a "mood reset" on tab open.

- **Procedural call synthesis and real chorus mixing.** Procedural calls are required because recorded or looped audio would sound canned, break the bundle budget, and fail the chorus mechanic's need for real-time variation.

- **Single horizontal scene with day/night, weather, and micro-motion.** The scene supports the plan's first-frame and felt-aliveness goals: birds are already in pose phases, the sky reflects local time, and ambient state keeps the aviary from reading as paused.

- **Three perch-zone count.** NOT RECOVERABLE FROM PLAN

- **Return-greeting.** The plan says there is no "Welcome back" toast because "the bird greeting is the entire welcome." The greeting reads as "the bird noticed me" through motion plus a call.

- **Listen-in.** Listen-in focuses a bird while preserving the aviary as a whole. Other birds "never go silent" because silencing them would teach the user the aviary is "a set of things to switch between."

- **Offer.** Offers are one of the few interaction signals the simulation can interpret: they feed curiosity and a small amount of boldness, while remaining quiet seed, song-fragment, or still-pool gestures rather than game mechanics.

- **Offer per-bird cooldown length.** NOT RECOVERABLE FROM PLAN

- **Settle and settle undo.** Settle is treated as "a goodbye to the aviary, not to a session," so it is canonical across devices. The undo window is named as a fixed 5-second surface.

- **Field notebook.** The notebook is read-only, server-generated, sparse, and about bird-and-aviary observations. It exists to produce naturalist prose and long memory without becoming a user-behavior log.

- **Multi-device sync.** The rationale is explicitly "no last-write-wins for personality state." A single writer, additive event deltas, and one canonical record make sync a property rather than a conflict-resolution feature.

- **Visit invitations.** Visits are opt-in, read-only, revocable, expiring, and render-only so they do not become social networking, co-presence, show-off rendering, or a way for visitor presence to drift the host's birds.

- **Accessibility surfaces.** Accessibility ships as v1 substance, not a later checklist. Narration, reduced motion, captions, keyboard navigation, contrast, and ARIA are part of the designed surface.

- **Performance budgets.** The plan treats first-frame fidelity as product meaning: if the aviary does not appear already in motion, the regression "reads as the product breaking, not as performance."

- **Account export.** The plan's why is ownership: "the user's data is theirs." This is the only narrow exception where personality vectors are exposed, and only in an explicit "your data" JSON export context.

- **Aggregate-only telemetry.** The rationale is trust and privacy. The system deliberately cannot answer "which user is most engaged" and has no aggregate pipeline access to per-bird state.

### Architecture and build shape

- **Small service architecture.** The plan keeps "a small set of services" because "the product is small; the architecture should be small."

- **Web client.** The client renders the aviary, runs WebAudio, holds presence state, emits pings, and pulls snapshots. Its role is intentionally limited: it "never writes personality state."

- **API gateway.** The gateway is thin and exists to verify sessions, rate-limit, attach request context, handle WebSocket upgrade, and enforce visitor/session role gates at the edge of write paths.

- **Aviary service.** The service reads canonical state, returns snapshots, accepts interaction events into the append-only log, and serves visit-mode reads. Its why is separation: "no write to personality state lives here."

- **Simulation worker.** The worker is the "only writer of personality state," plus mood, drift, call scheduling, and notebook entries. This concentrates the product's continuity and no-client-write rules in one place.

- **Postgres schemas.** A single operational database preserves canonical state, while logical `account` and `simulation` schemas enforce privacy boundaries and access-layer separation.

- **Aggregate telemetry pipeline.** It exists for operational metrics only. Its separation from simulation data means the privacy claim is implemented structurally.

- **Client/server split.** The client emits events and interpolates snapshots; the server owns canonical state. This split prevents personality writes from leaking into UI code and keeps multi-device state coherent.

- **Render pipeline boundary and `call_event` seam.** Simulation decides what happens; rendering decides how it is drawn or played. The seam keeps multi-device coherence because two devices receiving the same `call_event` synthesize the same recognizable call.

- **Monorepo with `client/`, `server/`, and `shared/`.** The plan's rationale is schema discipline: shared snapshot and call grammar definitions are versioned together to prevent client/server drift.

### Data model and API surface

- **Synthetic UUIDs and encrypted email.** The plan says email is "never used as a key, partition, log dimension, or telemetry attribute." Synthetic account IDs and encrypted email protect the identity boundary.

- **Session list and revocation.** Per-device sessions let the user see and revoke individual devices without revoking every session.

- **Magic-link expiry length.** NOT RECOVERABLE FROM PLAN

- **Stable bird identity and renaming.** Bird IDs are "never reissued"; names are editable but do not affect identity. This protects continuity across renames, sync events, and migrations.

- **Personality initialization, monotonicity, bounds, and non-exposure.** Initial values put new birds in a "shy / unestablished" range; monotonic updates save "the user's bird" from negative drift; diminishing returns preserve long-timescale calibration; public serializers exclude raw vectors.

- **Append-only event log and rollups.** The event log is the substrate for ordered additive deltas, replay, and crash recovery. Older rows are summarized only for tick warm-up, with no per-bird dimension in the rolled-up form.

- **Notebook and invite records.** Notebook entries are only worker-written. Invites and visit events power a host-local visit log, not cross-account analytics.

- **REST plus optional WebSocket.** REST handles pulls, writes, and account management. WebSocket is used for "affective fidelity" in call timing, not correctness, because polling still works.

- **Auth/account endpoints.** Always returning `201` on magic-link requests avoids email enumeration; session revoke and sign-out support hygiene; delete/cancel-delete and export support account control.

- **`AviarySnapshot`.** The plan calls it the "load-bearing read shape." It carries everything the client needs for rendering and calls while excluding personality vectors.

- **`BirdSnapshot` public mood.** Mood is included because the client needs it for idle-motion shaping; richer numeric mood/personality data is excluded because it would leak personality.

- **Batched event submission.** Events are batched on a one-second tick "to amortize request cost," and responses do not return derived state, keeping writes narrow.

- **Presence ping raw signals.** The client sends visibility, focus, and last-input timing rather than a boolean so the server can validate the conjunction and the client "cannot misrepresent presence."

- **Server timestamp clamp.** The clamp prevents clock-skew abuse by normalizing implausible client timestamps to server time.

- **Push-only WebSocket.** Client writes still go through `POST /v1/aviary/events` "to keep the audit/allowlist surface narrow."

- **Visitor endpoint.** Visitor snapshots set `visit_mode=true`, strip writable affordances and notebook contents, and reject visitor pings so visits cannot affect the host's drift.

- **Notebook endpoint.** It is read-only with opaque pagination, matching the rule that only the simulation worker writes notebook prose.

- **API privacy and shape rules.** The API deliberately omits personality vectors, days visited, session counts, streaks, visit-frequency aggregates, and notebook user-behavior observations so the client cannot surface gamification by accident.

### Simulation engine design

- **Live queue, dormant queue, and catch-up tick.** Live cadence keeps experience continuous; dormant cadence keeps the aviary running without the viewer; catch-up keeps cost flat without losing felt-continuity.

- **Idempotent tick transaction.** Determinism matters for "testability" and "crash recovery." A tick is a transaction, not a stream, and advisory locks prevent simultaneous ticks for one account.

- **Presence signal.** Presence is the drift function's primary input. The 4-minute window leans longer because "watching birds without moving is the actual product," and the 15s/20s cadence absorbs network jitter without inflating presence.

- **Mood transitions.** Mood depends on interactions, time of day, ambient events, and personality. Boldness reduces wary/alert probabilities; social warmth raises content/curious probabilities, making visible mood feel bird-specific.

- **Drift function, calibration harness, and asymmetry rule.** The low-pass drift function supports week-scale change. The asymmetry rule implements "no Tamagotchi" at the engine layer: a negative delta becomes zero, not a decrement.

- **Call grammar runtime.** Motif libraries, mood-conditional usage, and procedural parameters make calls recognizable and varied, while keeping generation server-scheduled and client-rendered.

- **Per-bird variation.** A bird seed perturbs species motifs so two birds of the same species remain "individually recognizable while clearly being the same species."

- **Chorus.** The simulation snaps compatible calls into overlapping windows so the client produces a "real chorus, not stacked tracks."

- **Day/night phase.** Local-time phases feed mood, including drowsy at dusk and alert in early morning, grounding the aviary in the user's time.

- **Weather.** Rare slow weather events feed mood and tinting without becoming constant noise.

- **Aviary age.** Account age, not engagement, drives the next-bird timer so growth is paced by relationship time rather than user optimization.

- **Notebook generation.** Sparse generation, specific particulars, and voice-checked templates avoid generic state dumps and avoid observing user behavior.

- **Snapshot generation.** Snapshots are lazy read views of canonical state, not separate cached documents, so they cannot drift from canonical state.

### Sync model

- **Additive deltas instead of absolute values.** Clients send events such as listening or offers, never "set boldness to X." This prevents the "morning's drift is silently deleted" failure mode.

- **No conflict-resolution algorithm for personality.** The plan says there is "no conflict to resolve" because personality has "one writer." This is sharper than resolving conflicts after two writers exist.

- **Bird names and account prefs use last-write-wins.** The plan allows this because names and prefs are "not load-bearing for identity" and the user can intentionally change them again.

- **Listen-in is per-session.** Each session gets its own audio mix, while the simulation still treats listen-in events as attention signals for drift.

- **Settle is canonical.** The plan rejects per-session settle because settle is "a goodbye to the aviary, not to a session."

- **Snapshot freshness across devices.** The promise is not sub-second propagation; it is that state reflects "within the next visible snapshot."

- **Soft deletion across devices.** Sessions remain valid during soft-delete because "the user must be able to sign in to recover"; hard-delete later revokes sessions and purges account records.

- **Export vector exception.** Export includes personality vectors because redacting the user's own data would be "the wrong shape," while still keeping numeric vectors out of the product UI.

### Frontend rendering pipeline

- **React chrome and Canvas2D aviary.** React manages chrome and wiring, not the scene. Canvas2D is chosen because it is enough for the drawing and avoids GPU/driver quirks on older laptops.

- **First-frame property.** The first frame is "the aviary," not a fade-in or entry animation. This directly protects "the aviary appears with motion already in progress."

- **Quiet field loading state.** Before bird draw, the page shows the correct hue and subtle ambient drift: "never white, never a spinner."

- **WebAudio context deferral.** Browser gesture requirements should not block the visual aviary. If needed, the prompt says "tap to enable sound" in matter-of-fact tone, not naturalist voice.

- **Scene composition, top bar, listen-in indicator, captions, and focus ring.** The layer order keeps the aviary scene primary while preserving controls, captions, and keyboard focus as readable overlays.

- **Frame loop.** The plan targets 8ms of aviary-frame work inside a 16.67ms frame budget and stops rAF when hidden to avoid battery drain.

- **Idle micro-motion.** Mood-shaped pose phases make a wary bird scan, a content bird preen, and a curious bird tilt, producing per-bird-recognizable motion without looking canned.

- **Transitions, greeting, and no departure.** Perch shifts and arrivals support life in motion. Departure does not exist in v1 because the product has no mortality or "leaving." Greeting combines pose and call so the user reads notice.

- **Reduced-motion renderer.** Reduced motion is a separate designed rendering path with cross-fades, no leaf drift, preserved calls, preserved mood/day-night, and focus without animation.

- **Responsive reframe.** The horizontal scene maps normalized coordinates across 16:9 to 4:3 without cropping any bird.

### Audio pipeline

- **WebAudio graph and voice reuse.** One reusable synth voice per bird and fixed node pools support the "no memory growth over 30 minutes" rule.

- **Listen-in mix decay.** A 1.2s ramp changes per-bird gain rather than reconfiguring buses, keeping mix behavior smooth and the aviary audible as a whole.

- **Ambient bus.** The quiet hum and low-level ambience create "the aviary has its own air"; silence between calls would read as paused.

- **Procedural synthesis scheduling.** Calls are scheduled from server timestamps and motif parameters so audio lines up with simulation intent while staying bandwidth-light and varied.

- **Chorus mixing and compressor.** Simultaneous procedural voices produce chorus in real time; a small ducking compressor only prevents clipping during true peaks.

- **WebAudio fallback.** Silence-with-captions is chosen because low-quality audio would be canned, high-quality recorded audio would break the bundle budget, and captions preserve what was about to be heard.

- **Audio safety.** Volume, duration, intensity, click prevention, and context-state handling keep calls from becoming jarring or technically unstable.

### Accessibility surfaces

- **Screen-reader narration.** Narration comes from the same simulation state and gives "a sense of the morning, not a stream of events."

- **Procedural call captions.** Captions are generated from actual call parameters, can be independent from audio, become forced-on when audio fails, and must read against day and night palettes.

- **Keyboard navigation.** The plan gives birds, offers, listen-in, settings, and exit flows explicit keyboard paths so the aviary is operable without pointer input.

- **Accessibility settings drawer.** The drawer is "quiet" and matter-of-fact, matching the product voice while making reduced motion, captions, audio, and volume controllable.

- **Contrast and ARIA.** WCAG AA tokens protect copy against worst-case night backgrounds; ARIA labels expose public mood only, never numeric personality.

- **Accessibility regression suite.** axe, Playwright keyboard tests, narration cadence tests, and reduced-motion rendering tests block deploy so accessibility is not deferred to v1.1.

### Performance, observability, and rollout

- **Bundle strategy.** The boot chunk contains only the first-bird path; notebook, settings, account, accessibility, and visit flows are route-split to protect first paint.

- **First-bird mechanics.** Inline snapshot fetch, edge request coalescing, tiny snapshots, and quiet-field first paint exist to meet the p75/p95 first-bird budgets.

- **No-memory-growth mechanics.** Fixed audio pools, bounded sprite cache, virtualized notebook, and last-two snapshot retention directly support the 30-minute soak test.

- **Operational observability.** The plan measures timings, frame durations, audio errors, tick latencies, API health, WebSocket health, email delivery, and synthetic checks because these indicate product function.

- **Deliberately unmeasured analytics.** The plan refuses per-bird interactions, per-account session duration as a dimension, population personality vectors, drift rates per account, visit-frequency surfaces, and "which user is most engaged."

- **Browser support.** Last-two-major browser support is tied to avoiding old compatibility paths; unsupported browsers get a matter-of-fact surface.

- **v1 scope freeze.** Accessibility, field notebook, and visits are "v1 substantive content," so the plan rejects a partial v1 omitting them.

- **Rollout phases.** Phase alpha proves tick/snapshot/render, drift constants, chorus, and budgets; phase beta validates full v1, real-world drift, sync, a11y, and audio uncanniness; public phase opens sign-ups.

- **Day-one instrumentation.** Operational metrics, drift harness output, synthetic checks, and a11y CI ship from day one, while per-account and per-bird engagement measurements remain absent.

- **Feature flagging.** Flags are for ops fallbacks and incidents, such as disabling WebSocket or visits. The plan explicitly rejects flags for affective or gamified experiments.

- **Migrations.** Add/backfill/switch/drop discipline and `personality.version` preserve identity continuity: migrations never recompute, reset, or rebuild personality from event logs.
