## System-level intent

- **Server is the only writer of personality state.** This is named as the first load-bearing rule and then repeated through the architecture, data model, events path, simulation tick, sync model, and risk mitigations. The plan says personality vectors are "never set by clients," client events are append-only, the tick "consumes them in order," and there is "no last-write-wins on personality, ever." It shows up again in the single writer `sim ticker`, the absence of personality PATCH/PUT endpoints, and the CI check that only `services/sim/` may mutate bird trait fields.

- **Aliveness is the product.** The second load-bearing rule says procedural calls, mood-shaped idle motion, absence ticks, and "first frame is mid-action" compose one felt property. It recurs in the no-spinner loading state, generated calls rather than loops, continuous idle micro-motion, bird-to-bird response, return-greeting as the entire welcome surface, and the refusal of "recorded-audio fallback," "spinner-then-fade," ARIA state-list narration, and "animations off" reduced motion.

- **Privacy is architectural, not policy.** The third load-bearing rule says per-bird interaction events "never reach aggregate telemetry," "never train models," and the boundary is enforced by "separate stores, no cross-reads." It reappears in aggregate-only telemetry, analytics warehouse network policy, UUID-only identifiers, the single encrypted email column, and "no read connection from the analytics warehouse" to the simulation database.

- **The product is built from refusals made structural.** The plan's concluding read says the product is "what's left after a long list of refusals" and that refusals are honored "as an architectural property rather than as a policy." This appears in the schema absences for streaks, hunger, public aviaries, and personality history; in the absence of social surfaces beyond visits; and in code paths that simply cannot set personality from clients.

- **Felt experience is preferred over exposed metrics.** Personality is stored as five server-side scalars but "never exposed numerically anywhere." The plan says personality history is omitted because it would create a tempting surface to "show the user how their bird has changed," exposing what is "supposed to be felt rather than read." The same principle governs no stats panel, no debug toggle, no mood-history surface, and notebook prose that observes the aviary rather than the user.

- **Absence must read as quiet, never suffering.** This principle is carried by "monotonic toward expressive," the Tamagotchi refusal, and the drift function's `max(0, ...)` rule. The plan says a bird uninvolved for two weeks "loses no boldness, no warmth, no plumage saturation" and that absence reads as "quiet, never as suffering."

- **No gamification, including hidden gamification.** The plan refuses achievements, streaks, levels, scores, visit counters, days-in-a-row fields, and engagement experiments. It also makes new-bird availability "aviary age only, never interaction-gated," and says metrics not collected "can't leak into product decisions that pull the product toward gamification."

- **A single naturalist voice, with a strict system-tone exception.** The notebook, screen-reader narration, and captions use `packages/naturalist/` so "the voice is one voice across surfaces." Product-side prose is naturalist, lowercase, present-tense, and sparse; system surfaces such as accessibility settings, errors, visit emails, and account UI are "matter-of-fact" and normally capitalized. The tone boundary is linted.

- **Accessibility users get the actual product.** The plan says accessibility is "designed alongside the rest of the product" and that screen-reader, reduced-motion, and audio-off users get "the actual product, not a stripped variant." This intent appears in cross-fade reduced-motion rendering, naturalist narration rather than state lists, captions generated from the same call grammar, keyboard navigation across interactive surfaces, and WCAG AA copy.

- **Performance budgets protect the felt product.** The plan treats first bird visible within 500ms, 60fps idle, no memory growth, bundle budget, pooled audio nodes, and tick latency as product constraints. These are not just operational targets: first paint must be the aviary, idle cannot look paused, procedural audio cannot become heavy, and slow ticks are an early signal before users feel the product "running slow."

## Per-feature whys

### 1. Scope

- **Single-user accounts.** The account model supports one user's canonical aviary, sessions, email changes, export, and deletion without adding household profiles, shared aviaries, payments, or social identity. The plan's rationale is mostly architectural containment: v1 is "one logical product," web-only, with account state owned by auth and settings.

- **Email plus magic-link auth.** The plan's why is security and privacy: email is PII, stored once encrypted, and auth responses use `202 Accepted (always; do not leak whether the email exists)`. Magic links are short-lived and "one-shot" so replay becomes "this link has expired."

- **Per-device session tokens revocable from settings.** The plan gives the rationale through the session list: `user_agent` and `ip_country` exist so the list is recognizable to the user, while country-only geolocation is "intentionally not finer-grained." Revocation gives account control without making device metadata more precise than needed.

- **Email-change with verification.** NOT RECOVERABLE FROM PLAN

- **Account export.** The plan gives secure mechanics, not product rationale: emailed download link, verified address, 24h validity, one-shot consume. The reason the account export exists is NOT RECOVERABLE FROM PLAN.

- **30-day soft-delete then hard-delete.** The soft-delete window preserves recovery: ticks continue until hard-delete so a user who clicks "I changed my mind" finds the aviary as it was, with "30 days of zero-drift quietude." Hard-delete completes the deletion path.

- **One canonical aviary per account.** NOT RECOVERABLE FROM PLAN

- **One horizontal aviary scene with three perch zones and no panning, scrolling, or zoom.** NOT RECOVERABLE FROM PLAN

- **Day/night cycle, rare weather, and ambient drift.** These support aliveness: weather and day/night are tick-authored, interpolate visually, modulate mood and call expression, and make the aviary continue to feel alive during ordinary idle time.

- **Top-bar chrome that fades and no UI chrome inside the aviary scene proper.** The rationale is to keep the aviary itself as the product surface. Chrome recedes after cursor stillness, returns on activity, and top-bar focus forces visibility so the interaction layer does not erase keyboard accessibility.

- **Two starter birds at adoption.** NOT RECOVERABLE FROM PLAN

- **Cap of seven birds.** The exact cap is not directly explained, but the plan repeatedly tests render, audio, and memory budgets at "7-bird capacity." The recoverable rationale is performance containment for the v1 scene, audio pool, and simulation budget.

- **Species pool with no catalog.** The plan frames adoption as assignment from a pool rather than shopping. The recoverable rationale is refusal of customization/catalog mechanics that would pull toward game or collection surfaces.

- **Nightjar-like late-active singer.** The plan's why is behavioral variety tied to local time: this species stays `alert` longer at night, has a quieter palette, slower call cadence, and a distinct call-grammar motif library.

- **New-bird availability gated by aviary age.** The rationale is explicit: it "deliberately refuses to reward interaction" and is the load-bearing rule for "no gamification." Users with high and low presence get birds on the same cadence.

- **Bird rename with stable internal `bird_id`.** The stable `bird_id` preserves identity across renames, syncs, species-pool migrations, and database migrations. The risk section names "bird identity loss" as effectively replacing a user's bird with a new version.

- **Server-side personality vector, never exposed numerically.** The rationale is to make personality felt rather than read, prevent client writes, and reduce telemetry leakage. The plan forbids stats panels, debug toggles, and admin-only number views.

- **Persisted mood state.** The rationale is continuity and expression: mood persists across sessions, and `mood_entered_at` drives idle expression so a freshly-entered `wary` differs from a long-held `wary`.

- **Personality drift monotonic toward expressive.** The rationale is anti-Tamagotchi and anti-punishment. Drift must never decrease, absence must not move personality backward, and the visible absence effect is quietude rather than distress or decay.

- **Mood transitions from recent events, local time, weather, neighbor moods, and personality.** The rationale is aliveness through canonical simulation. Mood is sampled deterministically from server state so all clients replay the same state while birds still appear responsive.

- **Procedural calls via WebAudio.** The rationale is aliveness, recognizability, bundle discipline, and refusal of recorded fallback. Calls are "generated, not stored," recognizable across mood and drift, similar but varied, and cannot become stacked loops.

- **Idle micro-motion.** The rationale is direct: it must be "continuous, mood-shaped" and "never reads as paused." It is one of the components of aliveness.

- **Bird-to-bird interaction.** The rationale is emergent life from canonical state: calls can prompt responses, mood can spread, and chorus events emerge from overlapping high-vocal-frequency birds rather than from the client inventing behavior.

- **Return-greeting.** The rationale is that the bird greeting is "the entire welcome surface." The plan refuses welcome-back text, toasts, banners, and arrival announcements so return is expressed by a bird noticing.

- **Listen-in.** The rationale is focused attention without erasing the rest of the aviary. The focused bird rises in the mix, others quiet to an ambient floor and "never reach -infinity"; server-controlled mix levels keep multiple devices coherent.

- **Offer surface for seed, song fragment, and still pool.** The plan gives interaction effects on curiosity and nearby boldness, but the specific rationale for these three offer types is NOT RECOVERABLE FROM PLAN.

- **Per-bird offer cooldown.** NOT RECOVERABLE FROM PLAN

- **Settle gesture with evening lighting and 5s undo.** The plan explains mechanics and that tab-close is equivalent at engine level, but the product rationale for the settle gesture is NOT RECOVERABLE FROM PLAN.

- **Field notebook.** The rationale is sparse naturalist observation of the aviary, not the user. Cadence, read-only immutability, and anti-pattern tests prevent streak language, second-person address, and user-behavior observations.

- **Visits.** The rationale is a deliberately tiny social surface: opt-in, off by default, one-time link, no chat, no avatars, no co-presence, no default host notification, and visitor events rejected so visitors cannot drift the host's birds.

- **Multi-device sync.** The rationale is the architectural property of server-only writing: all devices pull snapshots from one canonical state and emit events into the same log, with no client merge and no "device X is the master."

- **Reduced-motion mode.** The rationale is accessibility without stripping the product. Cross-fade rendering shows the same birds, perches, and moods in a calmer register rather than turning animations off.

- **Screen-reader narration.** The rationale is a top-level naturalist surface rather than ARIA state-list scattering. Narration is paced so the queue does not fill and prioritized for user-initiated events.

- **Call captioning.** The rationale is accessibility and audio fallback parity. Captions come from the same grammar that produced the audio, so a user hears and reads the same call, or sees captions when audio is unavailable.

- **Keyboard navigation and visible focus.** The rationale is full access to all interactive surfaces, including the aviary scene. Focus must escape overlays and remain visible against bright and dim aviary states.

- **WCAG AA on user copy.** The rationale is baseline accessibility across copy surfaces and contrast against changing day/night aviary states.

- **Aggregate operational telemetry only.** The rationale is privacy by architecture and anti-gamification. Allowed metrics cover operations; forbidden metrics include per-account session duration, per-bird interactions, visit counts, days active, and notebook opens per account.

### 2. Architecture

- **Single monorepo with service-shaped boundaries.** The rationale is to ship one v1 product while making a later split possible without schema changes. Boundaries are "service-shaped" for auth, sim, visits, telemetry, and related modules.

- **Auth service.** The rationale is containment of account-sensitive operations: magic links, sessions, email change, deletion, and export coordination live in one owner.

- **Aviary snapshot service.** The rationale is read-only delivery of canonical state. It serves REST and WebSocket snapshots without owning writes to simulation state.

- **Events ingest service.** The rationale is append-only input isolation. It validates events and source tokens but "does not compute anything" and "never writes personality vectors."

- **Visits service.** The rationale is to own invitation issuance, validation, host visit log, and revocation while never writing personality state.

- **Notebook service.** The rationale is to generate notebook entries from simulation deltas and recent events while keeping entries server-generated only.

- **Settings service.** The rationale is persistence for notification preferences, accessibility preferences, and session management UI back-end.

- **Sim ticker.** The rationale is single-writer authority. It consumes the event log, computes drift and mood transitions, and writes canonical state per shard.

- **Client/server split.** The rationale is to keep simulation state authoritative on the server while letting the client render, interpolate, synthesize audio, and emit events. The client "never persists personality, mood, drift, or any field that drives the simulation."

- **Snapshot render boundary.** The rationale is fast, safe rendering: the snapshot exposes only what the client needs, keeps personality opaque, enables first paint from inline JSON, and makes every visible thing derive from server state.

- **Allowed-fields snapshot serializer.** The rationale is to enforce that personality fields are not leaked to the client. There is "one allowed-fields list" and personality fields are not on it.

- **`packages/naturalist/` shared voice module.** The rationale is enforceable voice consistency across notebook, narration, and captions, plus a linted boundary where system-error surfaces cannot accidentally use naturalist phrasing.

- **Synthetic UUID identifier discipline.** The rationale is PII containment. Email appears once in encrypted form; every other reference is a UUID, and telemetry and Kafka partition keys never see email.

### 3. Data model

- **Personality fields as columns on `birds`.** The rationale is schema clarity and leakage resistance. A long-form trait table would invite ad-hoc joins, expand accidental telemetry leakage surface, and make "no client write to personality" harder to grep.

- **Append-only `events` table.** The rationale is ordered tick consumption, replay safety, and no mutation path. Duplicates are dropped by `event_id` uniqueness.

- **`notebook_entries` immutability.** The rationale is that entries are server-generated observations with generator versioning so format drift can be reasoned about.

- **Coarse session metadata.** The rationale is recognizable session management without precise tracking: "Chrome on macOS, US" is useful; geolocation finer than country is intentionally excluded.

- **Encrypted visitor email and tokenized visit sessions.** The rationale mirrors account email privacy: visitor email is stored once, encrypted, and future loads use a visitor session token rather than the original link token.

- **Separate security `audit_log`.** The rationale is security-relevant tracking without per-bird information and without mixing with the interaction event log.

- **No streak, visit-count, or days-in-a-row fields.** The rationale is that the schema must not be able to answer gamified questions because the existence of the answer is a foothold for surfacing it.

- **No bird health, hunger, distress, or last-fed fields.** The rationale is the Tamagotchi refusal: birds do not die, show distress, or decay through absence, even as ignored columns.

- **No public aviary, discovery, featured, or like fields.** The rationale is architectural absence of social-network surfaces.

- **No personality history table.** The rationale is that trajectory logging is not needed and would create a tempting "show the user how their bird has changed" surface.

### 4. API surface

- **Auth endpoints returning non-enumerating responses.** The rationale is to avoid leaking whether an email exists and to make expired or replayed links matter-of-fact rather than revealing account state.

- **Snapshot REST plus WebSocket stream.** The rationale is first-paint reliability and ongoing liveness. REST supports SSR, visibility changes, long frame gaps, and reconnect recovery; WebSocket streams deltas and event acks.

- **REST event fallback.** The rationale is transport resilience while preserving the same append-only, idempotent event path.

- **No `POST /notebook`.** The rationale is that notebook entries are server-generated only, preserving the observational naturalist surface.

- **Visitor read-only route.** The rationale is social containment and simulation integrity: a visitor can see the host snapshot through a chokepoint but has write capability stripped, and visitor events are rejected.

- **Visit revocation producing 410.** The rationale is host control with a matter-of-fact "visit no longer available" surface.

### 5. Simulation engine

- **Transactional tick.** The rationale is consistency: no partial tick is observable, event consumption, drift, mood, weather, notebook handoff, and snapshot fanout are tied to one canonical update.

- **Sharded worker pool with advisory locks.** The rationale is operational safety: one worker owns a shard at a time, and a crashed worker frees its shard within the lease TTL.

- **Low-pass drift function.** The rationale is slow, calibrated change: instrument-detectable around one week, user-perceptible around three weeks, and never large from a single session.

- **Drift calibration tests.** The rationale is to make calibration failures block the build before users later experience "changed visibly between sessions" or "nothing ever changes."

- **Weighted Markov mood transitions.** The rationale is varied but replayable behavior: recent events, local time, weather, neighbors, and personality feed a deterministic sample so clients agree.

- **Server decides when a bird calls; client synthesizes how.** The rationale is coherent multi-client timing with lightweight procedural sound. The `call_grammar_seed` lets two clients hear the same calls without storing audio.

- **Runtime chorus mixer.** The rationale is emergent chorus without stacked loops. Overlapping live oscillator voices create chorus, and procedural variation prevents phase-cancellation artifacts.

- **Canonical bird-to-bird interactions.** The rationale is that response calls, wary propagation, and chorus events are authored by the sim, not invented separately by clients.

- **Notebook salience scoring and cadence enforcement.** The rationale is sparse, noteworthy prose: if no entry appears for about three days the floor lowers, but if one appeared today the floor rises sharply.

- **Notebook anti-pattern tests.** The rationale is to enforce no visit count, no "every day this week," no "you," no user subject, and no numerical personality values.

- **Weather and day/night at tick time.** The rationale is server-authored environmental continuity. Local-time phases drive rendering; weather affects mood and vocal expression.

- **Age-based adoption offer that can be ignored.** The rationale is refusal to reward interaction and respect for ignoring: the offer remains available indefinitely rather than becoming pressure.

- **Long-absence tick slowdown with catch-up.** The rationale is operational efficiency without changing product authorship: ticks still come from the server, drift over zero presence is zero, and the next visit catches up.

- **Soft-delete ticks continuing until hard-delete.** The rationale is recovery: the aviary remains recoverable during the deletion window.

- **Snapshot fanout recovery through keepalive.** The rationale is resilience: if fanout fails, the client pulls a fresh snapshot and interpolation absorbs the gap.

### 6. Sync model

- **Single canonical record for personality, mood, and per-bird state.** The rationale is that sync is not a separate merge feature; it is the consequence of server-only writing.

- **Multi-device event accumulation.** The rationale is coherent shared state: laptop and phone events enter the same log, idempotency prevents double-counting, and both devices see the same next snapshot.

- **No client-to-client messaging, no client-side merge, no device master.** The rationale is to eliminate conflict modes that could produce last-write-wins.

- **Concurrent listen-in and settle behavior.** The rationale is "no winner": concurrent listen-in episodes both count, and settle events produce canonical lighting through the next snapshot.

- **Out-of-order snapshot dropping and keepalive.** The rationale is stale-state recovery within 20 seconds.

- **Soft-deletion sign-out across devices.** The rationale is consistent account state: all devices leave the aviary and recovery re-authenticates.

### 7. Frontend rendering pipeline

- **React for chrome, custom WebGL for the scene.** The rationale is fit to surface: chrome is small and traditional; React reconciliation is not used for the 60fps aviary scene.

- **Code-splitting account settings, visits, notebook, and visitor mode.** The rationale is first-paint budget and headroom for polish.

- **Inline first-snapshot JSON in SSR HTML.** The rationale is first bird visible within 500ms and no blank boot state.

- **First frame as the aviary, mid-action.** The rationale is aliveness. There is no entry animation, no fade-from-black, and tests assert non-zero pose phase at first paint.

- **Audio may begin mid-call on first paint.** The rationale is the same first-frame aliveness: if a call window is already in the past relative to server time, the call starts immediately.

- **Mood-shaped idle programs.** The rationale is expressive differentiation: `wary`, `content`, `curious`, `drowsy`, `alert`, `settled`, and `sleeping` each carry a motion register.

- **Blend mood transitions over about two seconds.** The rationale is to avoid hard pose snaps that would collapse the living scene.

- **Responsive layout with no cropped birds.** The rationale is preserving the one-screen aviary across viewports: birds shift toward each other on narrow screens but never leave frame.

- **Slow-connection loading as soft sky and faint motion cues.** The rationale is refusal of spinner-then-fade; even loading should belong to the aviary palette.

- **Empty-aviary adoption transition.** The rationale is that the user is never shown an empty aviary again after first bird arrival. The first bird enters softly, then the product remains inhabited.

- **Render-budget CI gates.** The rationale is to protect smooth idle life: frame-time, GC pause, and memory growth regressions block deployment.

### 8. Audio pipeline

- **Pooled WebAudio voice nodes.** The rationale is no memory growth and efficient procedural calls. Nodes size to the bird-count cap plus chorus overlap rather than allocating per call.

- **Per-call synthesis with motif, mood, and personality modulators.** The rationale is sameness-with-variation: calls from one bird remain recognizable but differ enough to feel alive.

- **Conservative masterbus compressor.** The rationale is to prevent transient peaks during co-firing without flattening normal dynamics.

- **Equal-power listen-in ramps.** The rationale is smooth attention shifting and "room sound" return, not hard audio switches.

- **WebAudio fallback as silence plus captions.** The rationale is refusal of recorded-audio fallback while preserving access. The aviary remains a quiet aviary with call descriptions.

- **Zero recorded audio in the bundle.** The rationale is both product and budget discipline: no fallback samples, no emergency loops, and a smaller 2MB bundle.

- **Mute and volume controls.** The rationale is matter-of-fact accessibility control. Muting affects output only, not simulation, and captions can still represent calls.

### 9. Accessibility surfaces

- **`aria-live="polite"` narration region.** The rationale is paced naturalist description that does not fill the screen-reader queue.

- **Narration from `packages/naturalist/`, not state lists.** The rationale is product parity and voice continuity: "a small grey bird..." instead of "Pip is at perch 2."

- **Caption overlays near calling birds.** The rationale is call-local comprehension and parity with audio grammar.

- **Keyboard path through top bar and aviary scene.** The rationale is full interaction access, including listen-in by Enter and exiting listen-in by Escape.

- **Focus indicators visible against day/night phases.** The rationale is keyboard usability in a changing visual environment.

- **Top-bar fade preserving focused visibility.** The rationale is that a keyboard user never has focus on an invisible bar.

- **Matter-of-fact settings and error tone.** The rationale is a named exception to naturalist voice so system surfaces stay clear and normally capitalized.

### 10. Performance budgets and observability

- **Bundle-size gate.** The rationale is fast first paint and deliberate review of large assets.

- **TTFB first bird visible budget.** The rationale is that the first user experience must be the aviary, not loading machinery.

- **60fps idle budget.** The rationale is that idle motion must remain continuous at max bird count on a 5-year-old laptop.

- **Zero memory growth budget.** The rationale is session stability over a 30-minute aviary session.

- **Tick-latency alarms.** The rationale is early detection before users feel that the aviary is "running slow."

- **Allowed observability surfaces.** The rationale is operational visibility without per-account or per-bird dimensions.

- **Deliberately unmeasured product metrics.** The rationale is that uncollected metrics cannot tempt stakeholders into gamified dashboards or engagement optimization.

### 11. Rollout

- **Population ramp instead of scope ramp.** The rationale is that v1 scope is not incrementally reduced; the full feature set launches to small populations first.

- **Closed beta with shortened bird-availability cadence.** The rationale is to gather drift-calibration data within a four-week beta cycle while returning to PRD defaults for public launch.

- **Day-one synthetic drift accounts.** The rationale is continuous validation that calibration targets hold in production.

- **No A/B tests on return-greeting or notebook prose.** The rationale is that optimizing felt procedural variation would compress it into an "optimal" version and defeat the premise.

- **No engagement experiments.** The rationale is direct: there is no engagement metric to optimize for.

- **Week-by-week soft-launch sequencing.** The rationale is to watch tick latency, audio errors, render distributions, visits, revocation, and drift fan-out before open signup.

- **Unsupported-browser hard stop.** The rationale is clarity and product quality: a degraded aviary on an old browser would likely look bad and reflect on the product.

- **Accessibility surfaces not behind flags.** The rationale is that reduced motion, captions, and other access surfaces are part of v1, not optional experiments.

- **Engineering-only emergency flags.** The rationale is operational recovery, not product segmentation: tick cadence, snapshot rate, and audio kill-switch are levers for incidents.

- **Launch documentation.** The rationale is privacy clarity, user orientation, on-call readiness, and explicit documentation of the simulation-database / analytics-warehouse boundary.

### 12. Risks

- **Drift-calibration mitigation.** The rationale is that wrong gains may be silent until months later, so build-time and production synthetic checks make drift visible.

- **Sync-correctness mitigation.** The rationale is to prevent client personality writes, out-of-order event consumption, and double-applied events through schema absence, CI grep checks, idempotency, transactions, and invariants.

- **Audio-uncanniness mitigation.** The rationale is that procedural audio can become synthy, repetitive, or uncanny, so external review, spectral similarity bounds, ear tests, and the audio kill-switch control launch risk.

- **Accessibility-regression mitigation.** The rationale is that future surfaces could miss narration, captions, reduced motion, or focus visibility, so shared voice, contrast tests, keyboard-trap tests, and tone linting guard the path.

- **Privacy-boundary erosion mitigation.** The rationale is that convenient logs or warehouse reads could leak interaction state, so network policy, audit queries, telemetry allowlists, security review, UUID-only messages, and logger email-pattern linting enforce the boundary.

- **Notebook voice-drift mitigation.** The rationale is to prevent bland or user-behavior prose through tests, weekly beta sampling, and aggregate quality metrics.

- **Empty-loading regression mitigation.** The rationale is that a future spinner would violate aliveness, so loading is SSR-rendered, visually tested for no spinner glyphs, and reviewed as a high-care edge area.

- **Bird-identity-loss mitigation.** The rationale is to prevent migrations or sync bugs from replacing a user's bird. `bird_id` is immutable, migration review checks it, and monthly invariants verify no reuse or shifted adoption dates.

- **Subtle-gamification-creep mitigation.** The rationale is to force any per-account aggregate or mood-history idea through PRD non-goal review and schema change scrutiny.

- **Tick-worker outage mitigation.** The rationale is to keep the aviary stale rather than broken: leases recover shards, alarms page on missing ticks, the event log survives, and clients keep showing the last completed snapshot.
