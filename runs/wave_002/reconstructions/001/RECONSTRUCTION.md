## System-level intent

- **Invariants are the product, not style preferences.** The plan says the invariants are "binding," "not style preferences," and that "several of them are the product." It repeatedly turns intent into an "enforcement mechanism" plus an "automated check," and later states that "an invariant with no gate is an intention."

- **The aviary must feel continuing, not starting.** This shows up in the definition of done, where manual affective QA must sign off that the aviary "reads as continuing, not starting"; in the first-frame invariant requiring birds "mid-action"; in the quiet field; in snapshot phase carried through transitions; and in the instruction that a client returning from a hidden tab renders "the aviary that has been running."

- **Mechanisms are preferred over carefulness.** The plan says enforcement means "the design makes the violation hard or impossible" and that "reviews catch what mechanisms miss; mechanisms catch what reviews miss." This appears in absent routes, absent UI primitives, database grants, source lints, metric allowlists, dependency allowlists, branded positive-only numbers, and tests named for invariant rows.

- **Use the product's own words and avoid quiet ambiguity.** The glossary "governs all naming, in code as well as in prose," with banned synonyms enforced in identifiers and user-facing strings. The deliberate `roosting` exception exists because overloading "settled" would create "the kind of quiet ambiguity" that could cause a settle gesture to change a bird mood record.

- **The server owns state; the client owns presentation.** This is named as "the single most load-bearing boundary in the system." The client may interpolate, synthesize calls, generate ornaments, and render; it may not decide mood, drift, perch moves, greetings, offer acceptance, or anything the simulation reads back as truth.

- **The relationship is slow, non-punitive, and not a game.** The plan refuses achievements, streaks, levels, scores, counters, decaying happiness, distress, and death. Drift is "monotonic toward expressive"; neglect produces "quietness, never distress, decay, or reversal"; no user-behaviour aggregates are sent to the client; and day-one instrumentation refuses metrics that would reward adding a streak counter.

- **Notice, never announce.** The plan bans welcome toasts, banners, modals, and textual welcome surfaces. It says "the greeting is the welcome," new birds "simply show up," the quiet field is "the aviary catching up," and social visits have no badges or indicators. The new-bird section calls this principle directly: "notice, never announce."

- **Naturalist voice belongs to aviary surfaces; matter-of-fact voice belongs to system surfaces.** The two copy namespaces are mechanically enforced. The plan says a blocked sign-in or error needs "system clarity," while notebook, narration, captions, offer prompts, and naming use naturalist voice without second-person pronouns, gamification words, or log-line register.

- **Procedural freshness must coexist with stable identity.** Calls are procedurally synthesized, a bird's call is never byte-identical twice, and the timbral signature "never changes." Bird identity is "stable forever." The voice contract is that a user who has spent two weeks with Pip knows Pip's call "by ear across mood changes and drift."

- **Emergence is preferred over cues.** Bird-to-bird chorus has "no chorus object and no chorus scheduler"; a chorus is what happens when hazard rates overlap. A scheduled chorus would be "a cue," and "cues are what the product is refusing." Greetings are staggered, not simultaneous, because a cue announcing arrival is "the wrong register."

- **Privacy is architectural, not a promise layered on later.** The plan uses "physical separation," no route from the warehouse to the simulation database, telemetry credentials with no simulation read grant, forbidden metric dimensions, no third-party prose model, synthetic calibration cohorts, and absent visit event routes. It says even anodyne aggregates over bird state are out.

- **Accessibility and performance ship as core product surfaces.** The plan states reduced motion landing in v1.1 would be "a failed v1" and "a v1 launch that told reduced-motion users the product wasn't for them." First-bird timing, no heap growth, call captions, narration, keyboard navigation, WCAG AA, and reduced-motion rendering all have gates and manual QA.

## Per-feature whys

### 0. How to use this document

- **Invariants.** Rationale: they are "the product" in several cases, so they are binding and must have enforcement plus tests rather than being treated as preferences.

- **Specifications.** Rationale: they are the executable design but may deviate where implementation reality demands, as long as deviations are recorded against touched invariants.

- **Assumptions.** Rationale: the plan makes decided calls where the PRD left gaps so "no one has to re-open them mid-build."

- **Glossary-governed naming.** Rationale: shared words across type names, table names, event names, and metric names prevent synonym drift and preserve the product's concepts.

- **`roosting` as the bird night state.** Rationale: `settled` is reserved for aviary lighting after the settle gesture; overloading it could cause a bug where triggering settle changes a bird's mood record.

### 1. Scope

- **One horizontal single-screen aviary with no in-scene chrome.** Rationale: the scene is "birds and place"; later sections reinforce that the aviary is not a surface for badges, labels, buttons, hover tooltips, or overlay icons.

- **Two starter birds and a hard cap of seven.** Rationale: seven is tied to recognizability and audio mix; future raising depends on audio-mix work raising "the recognizability ceiling," and the timbre-separation test makes the cap defensible.

- **Roughly six-species pool.** Rationale: the plan calls for "a coherent set from one place" and includes a nightjar-like species because night "is not a dead state."

- **Hidden five-trait personality vector.** Rationale: raw traits should not become stats, tiers, or debug/product surfaces; hidden numbers protect the relationship, while quantized rendering creates the instruments-vs-user gap.

- **Six-state mood.** Rationale: the plan recovers the rationale only for using `roosting` rather than `settled`; the broader reason for exactly six mood states is NOT RECOVERABLE FROM PLAN.

- **Server-side tick around 60 seconds.** Rationale: the aviary continues without the viewer, and the simulation tick is the only writer of personality state.

- **Return-greeting.** Rationale: the greeting itself carries return presence; there is no textual welcome because "the greeting is the welcome."

- **Listen-in.** Rationale: it should feel "like listening, not like switching channels," with unfocused birds still audible so the aviary remains a place where several things happen at once.

- **Offer.** Rationale: an offer should read as a gesture, not a cooldown timer or direct control over a bird.

- **Settle with five-second undo.** Rationale: undo is "a mercy for accidental clicks," while settle remains optional and equivalent to tab-close at the engine level.

- **Presence as interaction.** Rationale: the plan says watching birds without moving is "the actual product," so presence must account for attention without requiring constant action.

- **Field notebook.** Rationale: it observes the aviary, not the user; read-only immutable entries avoid turning the product into a log or editing surface.

- **Magic-link sign-in as the v1 auth choice.** NOT RECOVERABLE FROM PLAN

- **Per-device revocable sessions.** NOT RECOVERABLE FROM PLAN

- **Email change with verification.** NOT RECOVERABLE FROM PLAN

- **JSON export.** Rationale: it is a portability artifact, and the plan uses that to resolve the tension between export including personality vectors and product surfaces never rendering trait numbers.

- **Thirty-day soft delete then hard delete.** Rationale: deletion excludes the account from ticking and revokes invites immediately, while the restore affordance is an "I changed my mind" window before rows, Redis keys, and encrypted email are purged.

- **One canonical server-side aviary with snapshot pull and append-only client events.** Rationale: it prevents silent drift loss from last-write-wins and ensures two devices reading the same version render the same aviary.

- **Per-invite opt-in visits.** Rationale: social is optional, read-only, revocable, and off by default so visitors cannot reshape the host's relationship with their birds.

- **Naturalist screen-reader narration.** Rationale: a screen-reader user should hear the same product; narration is prose on a slow cadence, not a state list.

- **Designed reduced-motion mode.** Rationale: reduced-motion users should get a calmer Pocket Aviary, "not one that looks broken."

- **Runtime-generated call captions.** Rationale: captions describe the call that actually played because they are derived from the same `CallPlan` consumed by synthesis.

- **Web-only platform and last-two-major browser support.** NOT RECOVERABLE FROM PLAN

### 2. Product invariants

- **No loader, spinner, entry animation, or fade-from-static on first frame.** Rationale: the first frame must show birds mid-action so the aviary reads as already continuing.

- **Quiet field for slow connection or cold cache.** Rationale: the field is the aviary catching up, not the product loading; it avoids a spinner, progress bar, shimmer, or skeleton.

- **No welcome toast, banner, modal, or "gone X days" surface.** Rationale: the return should be carried by bird behavior, not by announcing the user's absence.

- **No user-behaviour aggregates surfaced or sent to the client.** Rationale: a column or metric that exists can later be rendered; refusing the data is the durable defense against gamification.

- **Monotonic drift.** Rationale: neglect produces zero signal, hence quietness rather than distress, decay, or reversal.

- **Procedural audio only.** Rationale: recorded audio would break variation, create chorus artifacts, add fallback paths, and undercut the plan's anti-canned contract.

- **No raw trait numbers on rendered surfaces.** Rationale: the client cannot leak what it does not have, and no future debug overlay can be built from stripped data.

- **Presence requiring visible, focused, recent activity.** Rationale: all three conditions keep presence tied to a person actually being there and close off inflated or stale attention signals.

- **Only the server tick writes personality vectors.** Rationale: client state writes would violate the authority line and make sync correctness depend on convention.

- **Additive deltas processed in event-log order.** Rationale: last-write-wins can silently erase drift from another device; additive server-authored deltas make that state unreachable.

- **Synthetic UUID identity.** Rationale: using email as a convenient key sprays PII across tooling and is impossible to retrofit cleanly.

- **No per-bird or per-account interaction state in analytics or training.** Rationale: interaction events exist only to drive that user's own simulation.

- **Visitors cannot influence the host aviary.** Rationale: visitor attention should not reshape a relationship the host did not sign up for.

- **Visitors see the host aviary exactly as it is.** Rationale: no "show-off" rendering; host and visitor snapshots come from the same serializer.

- **Naturalist voice versus system voice.** Rationale: aviary surfaces need observation and product voice; identity, errors, money, and settings need matter-of-fact clarity.

- **Listen-in as re-balance, never mute.** Rationale: other birds remain part of the place, not tracks switched off by focus.

- **Read-only immutable notebook.** Rationale: the notebook is an observation surface, not a CRUD surface.

- **Notebook observes the aviary, never the user.** Rationale: user-subject entries would turn observations into behavior reflection.

- **Perch position not user-controlled.** Rationale: perch position is a signal birds produce, not a layout selected by the user.

- **Settle optional and equivalent to tab-close for presence and drift.** Rationale: settle should close the presence window cleanly without penalty or recovery mechanics.

- **No push notifications or re-engagement email.** Rationale: only transactional, user-action-triggered email is allowed; re-engagement would contradict the refusal of attention hooks.

- **Stable bird identity forever.** Rationale: replacing or regenerating birds would break the stable relationship the product depends on.

- **New birds arrive on aviary age only.** Rationale: the arrival scheduler excludes engagement so arrivals cannot become a reward for behavior.

- **Server-side simulation tick independent of client connection.** Rationale: the aviary continues whether or not a client is open.

- **Reduced motion as first-class render branch.** Rationale: it is a designed aesthetic, not disabled animation.

- **Screen-reader narration as prose, not state list.** Rationale: state-list automation would build the wrong feature and fail the affective contract.

- **Captions tied to the actual call plan.** Rationale: a stored string could not promise that the caption describes what actually played.

- **Performance budgets.** Rationale: first-bird timing, 60fps idle, and no heap growth are product requirements, not guidelines.

- **No third-party model or service for bird, mood, trait, notebook, or interaction data.** Rationale: sending state to a hosted model would violate the privacy commitment outright.

### 3. Architecture

- **Edge document assembly and snapshot inlining.** Rationale: it supports the first-bird budget by drawing from a cached/inlined snapshot before full app load.

- **Stateless API service.** Rationale: NOT RECOVERABLE FROM PLAN

- **Tick worker fleet.** Rationale: it is the only writer of personality state and can scale by account shard while preserving deterministic simulation.

- **Telemetry emitter with separate credentials.** Rationale: aggregate operational metrics must not have simulation-table access.

- **Canvas 2D rendering instead of WebGL.** Rationale: WebGL shader compilation and context creation cost threaten the 500ms first-bird budget, context loss would visibly violate the first-frame invariant, and Canvas 2D is sufficient for the scene's draw-call count.

- **Preact and DOM chrome for focusable UI.** Rationale: native keyboard navigation, focus rings, and screen-reader semantics are better than reimplementing them in canvas.

- **Persistent WebAudio worklet nodes.** Rationale: one-shot per-call nodes would allocate per call and fight the memory/performance invariant.

- **TypeScript and shared `@aviary/core`.** Rationale: sharing call grammar, prose templates, mood constants, and snapshot schema keeps captions, notebook prose, tick, and client from drifting apart.

- **Postgres canonical state plus Redis snapshot cache.** Rationale: Postgres stores durable truth; Redis loss costs only a cold read and slower first bird, not state.

- **One allowlisted mail provider and six templates.** Rationale: email is transactional only, always triggered by a user action.

- **Dev/staging/prod with time-scaling outside production.** Rationale: staging synthetic accounts expose drift and tick cost before launch; time-scaling must be compiled out of production.

### 4. Data model

- **Account email encrypted with HMAC lookup.** Rationale: email is stored only for sending mail and lookup, not as an identifier.

- **Aviary `created_at` driving new-bird schedule.** Rationale: arrivals depend on aviary age and nothing else.

- **Per-bird ceilings.** Rationale: without ceilings, every long-lived bird converges to the same maximum and the aviary loses individuality in year two.

- **Append-only `interaction_event`.** Rationale: clients submit events, not state; the tick folds them in order.

- **Rendered immutable notebook entries.** Rationale: entries are written once and stored as text so the notebook remains read-only and stable.

- **Visit invite and visit session tables.** Rationale: they support revocation, expiry, and a visit log showing what was visible to whom.

- **Static species and motif libraries in `@aviary/core`.** Rationale: keeping the pool in code keeps client and server "literally identical."

- **Redis revocation cache with five-second TTL.** Rationale: it bounds visit revocation latency.

- **Ninety-day interaction-event retention.** Rationale: recent replay/debug and calibration need them, but indefinite interaction history would contradict the privacy commitment.

- **Notebook entries retained for life of account.** Rationale: the plan says the notebook is scrollable indefinitely, never archived or hidden.

- **Deliberately not stored fields such as IPs, raw email, user-agent strings, visit aggregates, session counts, and days active.** Rationale: a column that exists is a column someone can later render.

### 5. API surface

- **JSON HTTPS API with session auth and separate token-authenticated visit router.** Rationale: the separate visit router makes visitor write paths structurally absent.

- **Idempotent writes with `client_event_id`.** Rationale: duplicates return the original result, preventing repeated application of client write attempts.

- **Matter-of-fact error messages.** Rationale: errors, auth, and settings are system contexts where naturalist warmth would read as evasion.

- **Magic-link request returning 204 regardless of account existence.** Rationale: prevents account enumeration.

- **Magic-link consume via POST from an interstitial.** Rationale: corporate mail-security prefetches cannot silently burn links.

- **Snapshot endpoint with ETag and Redis fallback to Postgres catch-up.** Rationale: warm reads are small and fast while overdue aviaries catch up on read.

- **Three snapshot re-pull triggers.** Rationale: visibility return, render-frame gaps, and keepalive are the named sync moments; no extra rationale beyond that is RECOVERABLE FROM PLAN.

- **Batched event endpoint.** Rationale: events are acknowledged but never applied synchronously because the client does not own simulation truth.

- **No endpoint to set mood, trait, perch, or offer acceptance.** Rationale: absence expresses server authority and prevents user/client placement control.

- **Notebook GET only.** Rationale: no POST, PATCH, or DELETE keeps the field notebook read-only.

- **Account undelete endpoint.** Rationale: provides the "I changed my mind" affordance during the deletion window.

- **Visitor snapshot using same serializer.** Rationale: it guarantees visitors see the host aviary exactly as it is.

- **Snapshot payload carrying transition phase.** Rationale: a client joining mid-flight renders the bird partway through the arc, making first-frame continuity true.

- **Snapshot payload with quantized render block and no traits.** Rationale: the client cannot leak raw trait numbers it never receives.

- **Snapshot greeting present only on first session snapshot.** Rationale: the return is expressed as a bird behavior, not a text surface.

### 6. Simulation engine

- **Deterministic pure `step()` plus persistence shell.** Rationale: deterministic replay makes debugging, calibration, and catch-up possible.

- **No `Math.random()` in simulation.** Rationale: persisted PRNG state is needed for byte-identical replay.

- **`SKIP LOCKED` tick claiming.** Rationale: it gives horizontal scaling without an external coordinator.

- **Catch-up and dormancy.** Rationale: ticking every absent account every 60 seconds is wasteful, but the aviary must continue; absence can be collapsed because no drift inputs occur.

- **`collapseAbsence`.** Rationale: long absences can advance time, weather, mood distribution, and perch state without changing traits or replaying every minute.

- **PRNG sub-streams by domain tag.** Rationale: adding a new random consumer should not perturb existing sequences or break calibration goldens.

- **Four-minute activity window.** Rationale: watching without moving is the product; four minutes is long enough not to drop presence as soon as the mouse stops.

- **Presence clamping, wall-clock cap, day cap, and multi-device cap.** Rationale: these close ways the signal could inflate.

- **Saturating drift formula.** Rationale: drift should be measurable by instruments in a week, visible after roughly three weeks, and never visible from one session.

- **Per-trait input weights.** Rationale: the ordering is presence dominant, then listen-in, then offers, with settle neutral.

- **Quantized rendering of continuous traits.** Rationale: rate tuning alone cannot make drift instrument-visible but user-invisible until later; boundaries create the desired felt delay.

- **Mood Markov chain with context factors.** Rationale: mood changes are shaped by time, personality, weather, interactions, neighbours, and inertia rather than reset on tab open.

- **Dawn reset of priors, not value.** Rationale: it honors a daily-ish cadence without snapping mood to a default.

- **Wary contagion.** Rationale: individuality appears because bold neighbours mostly do not catch a spooked bird's wary state.

- **Call response.** Rationale: social warmth and current mood shape whether other birds answer a call.

- **Emergent chorus.** Rationale: a scheduled chorus would be a cue; overlapping hazard rates let the chorus emerge.

- **Rare ambient weather.** Rationale: weather should be "part of the place" and never a feature the user must notice.

- **Perch selection by mood and approach bias.** Rationale: perch position is a bird-produced signal of mood/personality, not user layout.

- **Server-canonical `next_call_at` with client-local waveform.** Rationale: devices hear the same bird at the same moment, while procedural local synthesis avoids bandwidth and permits per-render variation.

- **Small presence lift to call rate.** Rationale: observation matters, but a capped lift keeps birds from reading as performing for the user.

- **Greeting absence buckets and greeter selection.** Rationale: return behavior scales with absence and personality without textual accompaniment.

- **Greeting stagger.** Rationale: simultaneous chorus on cue announces the user's arrival, which is "the wrong register."

- **Offers landing in scene instead of clicking birds.** Rationale: birds react to a gesture; users do not command a bird.

- **Per-bird offer cooldown with no disabled/countdown UI.** Rationale: prevents curiosity saturation while avoiding a game mechanic wearing a naturalist coat.

- **Settle lighting and mood impulse.** Rationale: settle quiets the aviary, closes presence, and carries no drift penalty.

- **New-bird arrival by simply showing up.** Rationale: it reconciles an "offer that appears in the user's flow" with the ban on announcements; noticing is accepting.

- **Seven-day tentative new bird and twenty-one-day return.** Rationale: NOT RECOVERABLE FROM PLAN

- **Notebook token bucket.** Rationale: sparsity becomes structural, so very active users do not get more entries.

- **Notebook candidates from state comparisons.** Rationale: entries should be observations of noteworthy aviary changes, not event-log lines.

### 7. Sync model

- **Single-writer rule.** Rationale: prevents silent drift loss and makes client trait writes impossible.

- **State versioning and 304 snapshots.** Rationale: unchanged states are cheap and two devices on the same version render the same aviary.

- **HTTP polling instead of WebSocket.** Rationale: a 45-second pull of 4KB is cheaper and simpler than a persistent connection, and a 60-second tick means a socket delivers nothing sooner.

- **Union-over-seconds multi-device presence.** Rationale: two devices open for one person should not double the drift signal.

- **Retain last snapshot during snapshot failures.** Rationale: the aviary keeps rendering because the server-side aviary is fine; only after sustained failure does a small system line appear.

- **No durable offline write queue.** Rationale: an unsent presence window is not worth durable storage.

### 8. Frontend rendering pipeline

- **Layered canvas plus DOM chrome.** Rationale: static layers can be cached, dynamic birds can run at 60fps, and focusable controls remain native DOM.

- **Small skeletal bird rig.** Rationale: compact path data avoids runtime SVG parsing and image decode.

- **Continuous breath and pose.** Rationale: a bird is never still in a way that reads as paused.

- **Mood-shaped action selection.** Rationale: the user reads mood from motion, with no label, tooltip, or status icon.

- **Tier cross-fade for plumage changes.** Rationale: snapshot boundaries should not produce visible pops.

- **Optimistic presentation only for pure presentation.** Rationale: clients may lead on lighting, mix, and offer item appearance, but bird reactions are state and must wait for the server.

- **Critical first-frame module.** Rationale: drawing sky and birds before deferred systems is how first bird appears within budget.

- **No web font on critical path.** Rationale: font loading must not slow first bird.

- **Service worker and local snapshot mirror.** Rationale: repeat visits can paint quickly while live state reconciles through ordinary motion.

- **Stale-cache reconciliation.** Rationale: with no labels or counters, interim state is only out of date, not visibly wrong, and can resolve by birds moving.

- **QuietField.** Rationale: it avoids a load state and provides a quiet version of the aviary catching up.

- **Empty-aviary adoption fly-in only.** Rationale: the fly-in marks genuine first arrival and must not become a load animation.

- **Day/night from local IANA timezone.** Rationale: day/night must follow the user's day, including travel and DST.

- **Subtle weather rendering with no UI.** Rationale: rain and wind are short-lived place signals, not forecasts or features.

- **Ambient leaves and feathers.** Rationale: they visually signal that the aviary continues between bird actions.

- **Top bar with four items and no badges/counts/dots.** Rationale: controls exist without turning the scene into a dashboard or behavior-reflection surface.

- **Top-bar fade pinned by keyboard focus and keep-controls-visible setting.** Rationale: a control that fades while focused is a keyboard trap, and near-transparent chrome can be a low-vision barrier.

- **Reduced-motion `RenderMode.CrossFade`.** Rationale: the calmer mode should have its own quiet aesthetic and still feel alive.

- **Responsive layout with birds never cropped.** Rationale: one horizontal scene must remain fully visible without panning, scrolling, or zooming.

- **Device pixel ratio capped at 2.** Rationale: above 2x, rendering cost is real and visible gain is not.

- **Hidden-tab behavior.** Rationale: the client stops local work while the server-side simulation continues.

### 9. Audio pipeline

- **Fixed audio graph.** Rationale: nothing created per call protects heap and audio performance.

- **Motif and syllable grammar.** Rationale: calls are generated fresh from species templates, mood, drift, and jitter instead of recorded variants.

- **Single `CallPlan` for synthesis and captions.** Rationale: one object with two consumers guarantees captions match the call.

- **Immutable voice fingerprint.** Rationale: prosody may vary, but timbre must not, so users recognize a familiar bird across mood and drift.

- **MFCC recognizability test.** Rationale: listening sessions alone are not enough to defend seven birds and species-pair separation.

- **Live chorus mixing.** Rationale: fresh calls interacting in one bus produce real interference and avoid recorded-loop phase artifacts.

- **Gentle master compressor.** Rationale: it prevents clipping during multi-bird chorus without pumping.

- **Listen-in gain floor and lowpass.** Rationale: focus re-balances attention while keeping other birds present.

- **Dropping stale calls on snapshot load.** Rationale: firing them late would create an audible load burst.

- **Graceful silence with captions on audio fallback.** Rationale: "silence with captions is a better fallback than canned audio."

- **No "click to enable sound" modal.** Rationale: it would be a load-state announcement.

- **Call plan and buffer pooling.** Rationale: target is zero net allocation in steady-state audio.

- **Nightly rendered chorus artifact.** Rationale: audio uncanniness is not detectable by unit test, and pretending otherwise is how it ships.

### 10. Voice, copy, and generated prose

- **Two enforced string namespaces.** Rationale: naturalist and system registers must not bleed into one another.

- **Naturalist lint rules.** Rationale: no second-person, no exclamation, no gamification lexicon, and present-tense observations preserve the product voice.

- **System lint rules.** Rationale: identity, errors, and settings should state what happened and what to do without naturalist evasion.

- **One prose engine for notebook, narration, and captions.** Rationale: the three surfaces should not drift into three different products.

- **Template data with locale seam but only English shipping.** Rationale for the locale seam is recoverable; rationale for only shipping `en` is NOT RECOVERABLE FROM PLAN.

- **Recency penalty in prose slots.** Rationale: prevents repeated phrasing such as the same description appearing three days running.

- **Writer-authored notebook corpus.** Rationale: a developer-authored corpus would make the voice generic; the corpus is part of the product's charm engine.

- **Notebook not using event-log register.** Rationale: a log line tells the user the rest of the voice is performance.

- **Charm-decay mitigations.** Rationale: template prose can flatten over months, so corpus coverage and quarterly templates are content work rather than bug fixes.

- **Polite live-region narration.** Rationale: assertive narration interrupts, and interrupting is announcing.

- **Slow narration cadence.** Rationale: high-frequency narration overwhelms screen-reader queues and forces users to silence it.

- **Client-side narration from same snapshot.** Rationale: visual and screen-reader surfaces describe the same aviary.

- **Caption placement with collision avoidance and scrim.** Rationale: simultaneous calls should not overlap visually and copy must meet contrast in bright and dim states.

- **No hosted model in prose path.** Rationale: it would send bird state, mood, and interaction history to a third party on every entry.

### 11. Accessibility

- **Accessibility built in the same milestones as visual surfaces.** Rationale: shipping it later would tell reduced-motion users the product was not for them.

- **Canvas `role="img"` plus transparent DOM bird buttons.** Rationale: real buttons provide native focus, activation, screen-reader semantics, and focus rings.

- **Roving keyboard navigation.** Rationale: keyboard users can reach every bird and command without relying on pointer gestures.

- **Dual-tone focus ring.** Rationale: it reads against both bright midday sky and dim night.

- **WCAG AA copy contrast.** Rationale: all user copy must remain readable across chrome and captions, while the scene itself carries no copy.

- **Accessibility settings.** Rationale: users need direct control over reduced motion, captions, visible controls, and narration detail in matter-of-fact voice.

- **External screen-reader and keyboard QA.** Rationale: the real bar is not merely operating the app, but whether the aviary feels alive.

### 12. Accounts, auth, and privacy

- **Magic-link expiry, single use, and rate limits.** Rationale: sign-in should be secure without enumeration or scanner-prefetch failures.

- **Ninety-day sliding sessions with revocation.** Rationale: immediate revocation gives user control over devices; broader rationale is otherwise NOT RECOVERABLE FROM PLAN.

- **Email change with old address working until verification.** Rationale: prevents losing access before the new address is verified.

- **Export by signed single-use 24-hour link.** Rationale: export is asynchronous, delivered to the verified address, and treated as portability rather than an in-product surface.

- **Deletion lifecycle.** Rationale: soft delete allows recovery; hard delete purges account-keyed state and alarms on orphans.

- **Synthetic-ID rule.** Rationale: email as a key would spread PII into logs, metrics, cache keys, and messages, and cannot be retrofitted away.

- **Privacy boundary architecture.** Rationale: per-bird interaction events exist only to drive the user's own simulation; even internal aggregates over bird state are refused.

- **Calibration on synthetic cohorts.** Rationale: refusing production population analysis is affordable because synthetic staging cohorts provide calibration.

- **Support access through audited opsconsole.** Rationale: raw state access requires a reason and audit trail.

- **Exactly six transactional email templates.** Rationale: no lifecycle, digest, re-engagement, or "your birds miss you" email can be sent.

### 13. Visits

- **Host-created one-time invite links.** Rationale: visits are per-invite opt-in and never prompted during onboarding.

- **Read-only visitor router with exactly two GET routes.** Rationale: no event endpoint means there is nothing to filter, disable, or re-enable by accident.

- **Visitor client omitting offer, settle, notebook, settings, listen-in, and presence.** Rationale: these omissions match the read-only role, while the security property lives in absent routes.

- **Thirty-day invite expiry and non-revival.** Rationale: NOT RECOVERABLE FROM PLAN

- **Five-second revocation bound.** Rationale: revocation becomes effective quickly while allowing cached invite validity.

- **Visit log with approximate duration.** Rationale: the log exists for transparency about what has been visible to whom; exact duration would be surveillance.

- **No visit badges, indicators, or default notifications.** Rationale: visits should not become an attention hook; notification email is only for hosts who specifically opt in.

- **No social-network surfaces.** Rationale: the product refuses chat, avatars, discovery, comments, leaderboards, co-presence, and the statistics that would make those surfaces possible.

### 14. Performance budgets and observability

- **Working initial bundle ceiling below the hard cap.** Rationale: budgets consumed to their limit on day one leave nowhere to go, and 2MB is the point past which first-bird budget becomes unrecoverable.

- **Procedural audio and procedural bird art.** Rationale: they buy bundle headroom and are required for freshness and non-canned behavior anyway.

- **No allocation in render loop.** Rationale: enforced runtime discipline protects frame timing and heap growth.

- **Time-scaled and real-time memory soaks.** Rationale: time-scaling can hide real-elapsed-time leaks, while real-time-only would be too slow to gate every PR.

- **CI gates mapped to invariants.** Rationale: every invariant must have at least one gate because otherwise it is only an intention.

- **Aggregate-only RUM.** Rationale: operational signals are useful, but per-account dimensions would violate privacy and behavior-reflection commitments.

- **Synthetic browser sessions every fifteen minutes.** Rationale: first-bird timing and core flows must be monitored from multiple geographies and device profiles.

- **Tick latency p99 alarm.** Rationale: tick latency is "the aviary's health in one number" and catches degradation before users feel the aviary running slow.

- **Forbidden metrics.** Rationale: metrics containing account, bird, trait, mood, perch, offer, visit, or days-active dimensions create data someone can later optimize against or expose.

### 15. Testing strategy

- **Golden/replay tests.** Rationale: fixed seed plus fixed event stream should produce byte-identical state, making simulation changes reviewable.

- **Invariant test files named for each invariant.** Rationale: invariant coverage runs on every PR and cannot be skipped casually.

- **Calibration harness.** Rationale: drift constants must satisfy explicit bands across realistic synthetic cohorts before launch.

- **Affective QA.** Rationale: the product has failures users "will not name but will feel," such as announced surfaces, repeated calls, log-like notebook prose, and broken reduced motion.

- **Long-horizon simulation.** Rationale: months-long issues like trait saturation, template exhaustion, tick cost, and cold-account catch-up cannot be surfaced by short tests.

### 16. Rollout

- **M0 gates exist before feature build-out.** Rationale: seeded violations should fail correctly before teams rely on invariants.

- **Writer included in staffing.** Rationale: the writer owns the charm engine; "the writer is not optional."

- **Private beta of at least four weeks.** Rationale: the three-week drift horizon must be observed by real users before public launch.

- **Bird-count ramp over age.** Rationale: audio load reaches seven birds long after launch, but tests still run from M4 so the delayed load is not deferred risk.

- **Server-configurable thresholds and cap.** Rationale: if recognizability ceiling is lower than seven, the cap can be lowered without release.

- **Day-one instrumentation refusals.** Rationale: once retention, DAU, sessions per user, offers per session, or days-active graphs exist, someone will optimize against them; those optimizations are the refused features.

- **Server config and kill switches.** Rationale: tuning can happen without release, but accessibility cannot be disabled and drift cannot become negative.

### 17. Risks

- **Structural mitigations for drift, sync, audio, and accessibility risks.** Rationale: these failures are silent, so defenses need to be structural rather than review-time or detective.

- **Announcement-surface risk.** Rationale: the plan predicts this failure confidently, so there is no toast/banner/modal primitive and affective QA watches for it.

- **Post-launch gamification pressure.** Rationale: no behavior metrics exist to justify gamification; absence of underlying data is the durable defense.

- **Magic-link deliverability risk.** Rationale: scanner-safe POST consumption, mail authentication, delivery SLOs, and clear retry copy keep sign-in failures from masquerading as broken links.

### 18. Assumptions and judgment calls

- **Four-minute activity window.** Rationale: long enough for watching without moving, short enough not to count an empty chair.

- **Four-hour daily presence cap.** Rationale: bounds outlier drift contribution and is above plausible real sessions.

- **Regular visits defined as 75 presence-minutes per week.** Rationale: matches "short and uneventful by design" sessions and anchors calibration.

- **Weather rates.** Rationale: frequent enough to be part of the place, rare enough never to become a feature.

- **Presence affecting call rate.** Rationale: "when unobserved" implies observation matters, but the cap prevents performance-for-user feeling.

- **New-bird pacing.** Rationale: matches the text that a few-month-old aviary offers a third bird and a year-old aviary may have grown to five or six.

- **First-bird percentile target.** Rationale: hard p100 on an unbounded network is unmeasurable; p75 with p95 guardrail is enforceable and honest.

- **Export vectors as portability file.** Rationale: honors export contents while preserving the no-numeric-trait product relationship surface.

- **IANA timezone from client.** Rationale: day/night must follow the user's day, including travel.

- **HTTP polling transport.** Rationale: the 60-second tick means sockets do not deliver state sooner at meaningful operational cost.

- **New-bird "offer" as noticing.** Rationale: the bird arriving quietly is the only reading that satisfies both the offer requirement and the ban on announcement.

- **No hosted language model.** Rationale: it would contradict the privacy commitment.

- **Species pool composition.** Rationale: coherent set from one place plus the night-active species required by the layout spec.

### 19. Open items owned outside this plan

- **Design system spec.** Rationale: exact palettes, contrast, focus treatment, top-bar iconography, reduced-motion pose sets, and offer art are needed before the scene milestone can finish.

- **Prose corpus.** Rationale: notebook, narration, captions, and transactional email bodies need writer-owned content for the voice surfaces.

- **Species art and motif libraries.** Rationale: silhouettes, rig geometry, and motifs are needed for visual and audio milestones.

- **Privacy policy text.** Rationale: it must name aggregate telemetry categories and explicitly exclude per-bird interaction state.
