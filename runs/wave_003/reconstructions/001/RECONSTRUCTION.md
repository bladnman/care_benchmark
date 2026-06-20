## System-level intent

- **One canonical aviary, server-authored.** This shows up in Scope hard rules ("Only the server simulation tick writes personality vectors"), Architecture ("the client never owns canonical state"), Simulation service ("the only writer of aviary canonical state"), and Sync ("one canonical record"). The intent is to make divergence and "last-write-wins on personality" unreachable.
- **The aviary continues without the viewer.** This shows up in the server-side tick ("runs whether or not any client is connected"), the tick scheduler ("the aviary continues without the viewer"), and first-frame rules ("birds mid-action"; "no entry animation, no spinner"). The plan treats Pocket Aviary as a continuous living scene, not a page that starts when the user arrives.
- **Drift is expressive, never punitive.** This shows up in the hard rule "Drift is monotonic toward expressive; neglect produces ambient quietness, never negative drift," the drift function ("all coefficients >= 0; no negative terms"), and the risks section contrasting "Tamagotchi" and "screensaver." The product lives in a narrow band where care deepens expression without punishment.
- **No engagement economy.** This shows up in non-goals ("no achievements, streaks, levels, scores, badges, calendars, XP, rank, tier"), rollout ("aviary-age only, never engagement"), observability ("Per-account visit frequency" is deliberately not measured), and risk language calling streak-like additions "a regression of the product's identity."
- **Naturalist observation over stats.** This shows up in the field notebook ("naturalist prose"), screen-reader narration ("running naturalist prose, not a state list"), caption generation ("same naturalist voice"), idle motion ("The user reads mood from motion; no label"), and personality rules ("never exposed numerically to the user"). The plan prefers observations of the aviary to state panels or user-behavior readouts.
- **Privacy boundary by architecture, not hope.** This shows up in hard rules and schema rules: per-bird interaction state never enters aggregate telemetry, the analytics warehouse, or ML pipelines; the warehouse has "no read path" and "no credentials" for the simulation database. The intent is enforced at data-pipeline and credentials layers.
- **Accessibility is a first-class v1 surface.** This shows up in Scope, Accessibility, and Risk: "Accessibility ships with v1, not after"; reduced motion is a "designed cross-fade surface"; captions, keyboard navigation, and WCAG AA contrast are part of launch. The plan treats a delayed accessibility surface as telling users "the product wasn't for them."
- **Sound is procedural, recognizable, and gentle.** This shows up in hard rules ("No recorded audio"), call grammar ("parameter envelopes, not audio"), audio ("per-bird recognizability is the cross-mood invariant"), and listen-in ("listening, not switching channels"). Graceful silence plus captions is the fallback.
- **Calibrated continuity rather than fixed certainty.** This shows up in "defensible calls," the drift calibration harness, dogfood loops, p99 tick alarms, and consolidated open calibrations. The plan names starting values but requires calibration around felt continuity, sparsity, recognizability, and the 7-day / 21-day drift targets.
- **Calm chrome around an unadorned scene.** This shows up in "thin top bar," "nothing else," "no UI chrome inside the aviary scene," "calm palette," "all birds always visible," and no "Welcome back" surface. The visual product voice is restrained and non-instrumental.

## Per-feature whys

### Scope and product boundaries

- **Single-user accounts with email magic-link sign-in.** NOT RECOVERABLE FROM PLAN
- **Per-device session tokens, revocable from settings.** The plan supports session revocation by storing token hashes, listing active sessions, and letting users revoke specific sessions; it does not articulate a deeper product rationale beyond account control and token safety.
- **One canonical aviary per account.** The rationale is sync correctness: one server-side canonical record means many clients read the same truth and no client has divergent state to merge.
- **Two starter birds at adoption.** NOT RECOVERABLE FROM PLAN
- **Hard cap of seven birds per aviary.** NOT RECOVERABLE FROM PLAN
- **Server-side simulation tick.** The rationale is that "the aviary continues without the viewer" and canonical mood, personality, ambient events, and day/night/weather advance even when no client is connected.
- **Multi-device sync as architecture.** The rationale is conflict prevention: "one canonical record, many read-only clients," no client-to-client sync paths, and no "laptop morning session overwrites phone lunch session" failure mode.
- **Return-greeting interaction.** The rationale is to respond to absence length using current bird state: greeter choice is weighted by boldness and mood, greeting form changes for minutes/hours/days, and the result is procedurally varied so it is "never identical twice."
- **Listen-in interaction.** The rationale is to focus attention without turning birds into soloable tracks: the focused bird rises, others fall to an ambient floor, and a hard cut is forbidden because it would become "a UI of soloable tracks."
- **Offer interaction.** The rationale is to keep reactions canonical and behaviorally grounded: the event goes to the server, the tick computes the reaction, and the client does not predict a wary bird's "wait and eventually come near."
- **Settle interaction.** The rationale is to make settle and night first-class rather than overloading `drowsy`; the plan adds `settled` to the mood set for the settle gesture and night state.
- **Field notebook.** The rationale is sparse naturalist observation: entries are "read-only," "lowercase, present-tense," "observations of the aviary, never of the user's visit pattern," and never one-per-session.
- **Presence accounting.** The rationale is signal integrity for drift: presence is the conjunction of visibility, focus, and recent activity; a laxer definition would "silently inflate drift."
- **Procedural call synthesis.** The rationale is that calls must be generated client-side from motif parameters, keep per-bird recognizability across moods, and avoid recorded-audio paths entirely.
- **Graceful-silence fallback with captions on by default.** The rationale is to honor the unconditional "no recorded audio" rule while still surfacing what would be playing when WebAudio is unavailable.
- **Visit-invitation feature.** The rationale is to allow a single bounded social affordance while avoiding profiles, follows, feeds, discovery, comments, and leaderboards; visit sessions are read-only and ignored for presence and drift.
- **Optional host notifications off by default.** The rationale is consistent with the plan's no-pings boundary: emails about the aviary, push notifications, and pings about visits are out of scope by default.
- **Thirty-day invite expiration.** NOT RECOVERABLE FROM PLAN
- **Account export by emailed JSON.** NOT RECOVERABLE FROM PLAN
- **Soft-delete for 30 days, then hard-delete.** The rationale articulated is recoverability during the window and eventual purge of all rows tied to the account UUID after 30 days.
- **Privacy boundary for per-bird interaction events.** The rationale is to keep per-bird interaction events out of aggregate telemetry, analytics warehouses, and ML pipelines so the user's relationship with the birds cannot be reconstructed from telemetry.
- **No gamification.** The rationale is product identity: streaks, calendars, scores, and similar additions are treated as a regression, not a feature addition.
- **No Tamagotchi mechanics.** The rationale is to avoid death, hunger, distress, decaying happiness, and negative drift on neglect; neglect produces ambient quietness instead.
- **No social network surfaces beyond visits.** The rationale is to keep the visit affordance bounded and avoid public discovery, feeds, follows, comments, leaderboards, and social comparison.
- **No compatibility paths for older browsers.** The rationale is that bundle bloat "isn't justified" for browsers older than the last two major versions.

### Architecture

- **Edge / CDN with inline initial snapshot.** The rationale is first-bird-visible under 500ms on 4G: the HTML shell carries the initial state snapshot, and static assets use content-hash caching.
- **API gateway.** The rationale is to terminate TLS, validate session tokens, route internally, stay stateless and horizontally scaled, and own magic-link rate limiting.
- **Simulation service as single writer per account.** The rationale is tick ordering and canonical authorship: partitioning by account UUID preserves order and prevents competing writes to aviary state.
- **Event log service.** The rationale is append-only ordered ingestion for hot interaction writes and deterministic tick consumption; export and delete flows can read the same log.
- **Auth service.** NOT RECOVERABLE FROM PLAN
- **Export service out of band.** The rationale is to keep export generation off the hot path while still emailing the JSON download link on demand.
- **Single Postgres cluster partitioned/sharded by account UUID.** The rationale given is shared canonical storage for state, event log, and account/auth tables with partitioning by account UUID.
- **Object storage for exports and large blobs.** The rationale is that no large blobs are expected in v1 beyond exports.
- **Client/server split.** The rationale is that the client renders, synthesizes, captions, narrates, interpolates, and submits events, while the server owns canonical state and the tick.
- **Client-side render pipeline reading snapshots only.** The rationale is the "load-bearing split" that lets the server run on a slow 60s tick while the client renders at 60fps without owning state.

### Data model

- **Synthetic UUIDs and encrypted email stored once.** The rationale is privacy and key hygiene: email is never a foreign key, partition key, or log field.
- **Account row with `visit_notify` default false and settings JSON.** NOT RECOVERABLE FROM PLAN
- **Session token hashes, not raw tokens.** The rationale is token safety: raw session tokens are never stored.
- **Magic-link `code_hash`, expiration, consumed and revoked timestamps.** The rationale is safe issuance and consumption: links expire, can be consumed once, and can be revoked.
- **Aviary `bird_count` denormalized cap check and `next_bird_eligible_at`.** The rationale is enforcement of the bird cap and aviary-age gate.
- **Stable bird id.** The rationale is identity continuity and "the foundation of drift validity"; rename, sync, species-pool changes, and migrations must not replace a bird.
- **Five-scalar personality vector.** The rationale is canonical server-side drift: traits are stored, not derived at runtime, never recomputed from event logs, never rebuilt by the client, and never user-visible.
- **Mood persistence with state, timer, and seed.** The rationale is continuity across sessions and absence: the tick advances timer and transitions; the client renders mood-shaped idle motion but never owns mood.
- **Presence window rows.** The rationale is to record true presence windows and bound data loss with periodic pings if a tab dies without unload.
- **Append-only interaction event log.** The rationale is ordered, idempotent simulation input; UUIDv7 gives time-ordered ids without a separate sequence.
- **Notebook entries with no edit/delete columns.** The rationale is "read-only by design"; old entries are never archived or hidden.
- **Static versioned species pool.** NOT RECOVERABLE FROM PLAN
- **No species rarity.** NOT RECOVERABLE FROM PLAN
- **Privacy boundary as schema rule.** The rationale is credential-level separation: aggregate telemetry and ML pipelines have no read path to the simulation database.

### API surface

- **Magic-link request endpoint.** The rationale is rate limiting by email hash and matter-of-fact response regardless of account existence, preventing user enumeration.
- **Magic-link consume endpoint.** The rationale is to validate code, issue a session token, and produce matter-of-fact invalid/expired/used handling.
- **Email change endpoint.** The rationale articulated is that old email continues to work until the new address is verified.
- **Active sessions list and revoke endpoints.** The rationale is per-device session visibility and revocation.
- **Snapshot endpoint.** The rationale is a small canonical payload, delivered inline on first paint when possible, so the client can render from current state without owning truth.
- **Notebook scroll-back endpoint.** The rationale is indefinite access to old entries through pagination.
- **Event submission endpoint.** The rationale is idempotent, small-batch event ingestion so retries after transport failure do not double-count.
- **Offer options endpoint and cooldowns.** The rationale is that the client view is advisory and server-side enforcement prevents curiosity-trait drift saturation within a session.
- **Submitting offers through events.** The rationale is that the next snapshot carries the tick-computed reaction; the client never sends mood or trait values.
- **Settle and settle.undo events.** The rationale is to record the gesture canonically while allowing the client to render the lighting shift and a 5s undo reversal.
- **Visit invite, enter, snapshot, revoke, and log endpoints.** The rationale is revocable, read-only ambient access: visitors may not need an account, revoked invites take effect on next snapshot pull, and visit sessions cannot mutate account or aviary state.
- **Visit notifications setting.** NOT RECOVERABLE FROM PLAN
- **Account settings endpoint.** NOT RECOVERABLE FROM PLAN
- **Account delete endpoint.** The rationale is recoverable soft deletion followed by hard-delete purging across all account UUID tables.
- **Matter-of-fact error surface.** The rationale is that auth/account/sync/visit-revocation errors are the named exception to the naturalist product voice and render without adornment.

### Simulation engine

- **Per-account tick at 60s cadence.** The rationale is felt continuity with a p99 tick latency budget; the value is a calibration target that may move to 45-90s.
- **Tick idempotency on `(account_id, tick_ts)`.** The rationale is safe retries without applying deltas twice.
- **Ordered tick phases.** The rationale is deterministic state application: ingest events, reconcile presence, apply drift, transition mood, update ambient events, process bird-to-bird effects, reselect perches, generate notebook entries, and commit canonical state in one transaction.
- **Presence reconciliation.** The rationale is to turn start/ping/end events into presence-time deltas and treat the last ping as an end when a window is incomplete.
- **Drift function as low-pass filter.** The rationale is slow, measurable relationship change: regular visits show instrument drift after about 7 days, user-visible drift after about 21 days, and no single session visibly shifts a trait.
- **All drift coefficients non-negative.** The rationale is the monotonic-toward-expressive invariant and no negative drift on neglect.
- **Per-bird listen-in effects.** The rationale is that focused listening affects `social_warmth` and `vocal_frequency` for that bird.
- **Offer effects.** The rationale is that offers affect `curiosity` and `boldness`, while cooldowns prevent saturation.
- **Mood state machine.** The rationale is to bias visible behavior by current mood, timer, interactions, time of day, weather, personality, and bird-to-bird effects while persisting across absence.
- **Call-grammar runtime.** The rationale is recognizable procedural calls: motif envelopes plus per-bird, per-mood seeds keep a bird's call signature recognizable.
- **Return-greeting computed at snapshot time.** The rationale is that absence length is known when the client pulls, not at tick time.
- **Notebook generator.** The rationale is sparse, specific naturalist prose driven by noteworthiness, avoiding both one-per-session output and observations of user behavior.
- **Drift calibration harness.** The rationale is to protect the product's central claim in CI: 7-day measurable, 21-day visible, single-session invisible, absence non-negative, offer cooldown effective.

### Sync model

- **One canonical record.** The rationale is that clients read snapshots and never write state.
- **Conflict prevention by absence of client-owned truth.** The rationale is that there is no reconciliation to do when both laptop and phone pull from the same record.
- **No last-write-wins on personality.** The rationale is that personality drift is additive, server-authored, and computed from ordered event logs rather than client-sent absolute trait values.
- **Snapshot refresh triggers.** The rationale is freshness after hidden-to-visible transitions, laptop suspend/resume, and visible keepalive while keeping payloads cheap.
- **Client interpolation between snapshots.** The rationale is smooth motion across snapshot boundaries for perch moves and palette shifts.
- **Event ordering and idempotency.** The rationale is safe retry and deterministic consumption: UUIDv7 gives monotonic ids and event id dedupe avoids double counts.
- **Sync error surface.** The rationale is bounded retries and matter-of-fact surfacing for magic-link replay, timeout, or outage.

### Frontend rendering pipeline

- **Small, fast renderer with streaming SSR and code splitting.** The rationale is to stay within the less-than-2MB bundle and first-bird-visible budget while keeping account, accessibility, and visit surfaces out of first paint.
- **No entry animation or spinner.** The rationale is first-frame continuity: the first frame has birds mid-action; if loading is slow, the state is a quiet field, not a spinner.
- **Empty-aviary quiet field and first bird fly-in.** NOT RECOVERABLE FROM PLAN
- **Single horizontal scene.** The rationale is that all birds remain visible across viewport sizes, with no pan, scroll, zoom, or cropping.
- **Three perch zones and subtle parallax.** The rationale is mood-readable positioning and depth without a layered-illustration-heavy scene.
- **Calm palette and no saturated scene accents.** The rationale is a calm visual voice: soft blues, greens, warm browns, muted ochres, and no saturated UI accents inside the scene.
- **Day/night local-time palette and behavior.** The rationale is ambient continuity: evening warms and quiets calls, night dims, and a nightjar-like species remains active into late hours.
- **Continuous idle micro-motion.** The rationale is that mood is visible through motion and the scene remains alive regardless of user attention.
- **Mood-shaped motion with no labels.** The rationale is that the user reads mood from motion rather than labels, tooltips, status icons, or stats.
- **Smooth transitions from server `active_transition`.** The rationale is to avoid teleporting and allow the client to continue motion from canonical transition progress.
- **Settle lighting transition and undo.** The rationale is a gentle evening shift with reversible behavior within 5s.
- **Reduced-motion rendering.** The rationale is a designed version of the aviary: cross-fades replace motion, leaf drift is removed, but birds still drift, mood changes, calls play, and the notebook notices things.
- **Thin top bar.** The rationale is to keep account/settings, accessibility settings, notebook, and offer affordance available while avoiding extra chrome.
- **Top bar fades after cursor stillness.** The rationale is to reduce chrome over the scene while returning on cursor movement or keyboard activity.
- **No UI chrome inside the aviary scene.** The rationale is to keep the scene unadorned: no buttons, badges, hover-tooltips, overlay icons, or inline labels.
- **Offer affordance in top bar.** The rationale is to avoid putting interaction chrome on birds and keep the offer panel keyboard-navigable.
- **Responsive all-birds-visible invariant.** The rationale is explicit: never crop a bird out and never let one drift offscreen.

### Audio pipeline

- **WebAudio procedural synthesis.** The rationale is runtime generation from motif libraries with per-bird variation and no recorded audio.
- **Per-bird recognizability.** The rationale is that a user who has spent two weeks with Pip should know Pip's call by ear across moods and vocal-frequency drift.
- **Vocal-frequency trait driving calls.** The rationale is personality expression: high-vocal-frequency birds call more often unobserved and join choruses more readily.
- **Chorus mixing through independent synthesis.** The rationale is to create a real chorus and avoid phase-canceling artifacts from layered recordings.
- **Listen-in gain ramps.** The rationale is to avoid hard cuts and preserve ambient birds; the product is "listening, not switching channels."
- **WebAudio fallback to silence plus captions.** The rationale is graceful degradation under the unconditional no-recorded-audio rule.
- **AudioContext on first user gesture.** The rationale is browser autoplay policy.
- **Captions from call-grammar parameters.** The rationale is that captions describe the same generated call in the same naturalist voice, not a fixed string table.
- **Audio memory discipline.** The rationale is no memory growth over a 30-minute session, enforced in CI.

### Accessibility surfaces

- **Screen-reader narration.** The rationale is parity with the visual voice: running naturalist prose from the same snapshot, observations rather than state transitions, with a slow cadence unless user events need priority.
- **Reduced-motion accessibility setting.** The rationale is that reduced motion must be a designed surface, not a post-launch stripped fallback.
- **Call captioning.** The rationale is accessibility and graceful silence: captions are opt-in normally and on by default when WebAudio fails.
- **Keyboard navigation.** The rationale is full access to top bar, birds, listen-in, offer, and settle without trap states.
- **Visible focus indicators.** The rationale is usability across both bright and dim aviary states.
- **WCAG AA contrast.** The rationale is legible user-copy text across top bar labels, settings, account surfaces, errors, captions, and visual narration.

### Performance and observability

- **Initial JS bundle under 2MB gzipped.** The rationale is first paint performance; code splitting, procedural audio, and compact assets keep the budget.
- **Time to first bird visible under 500ms.** The rationale is first-frame continuity on mid-tier mobile over 4G.
- **60fps idle motion for 30 minutes.** The rationale is sustained liveliness, "not just the first minute."
- **No memory growth over 30 minutes.** The rationale is long-session stability, enforced as a real CI test.
- **Synthetic performance checks.** The rationale is scheduled, geography-spread detection of load, render, audio, and tick problems.
- **Aggregate-only Real User Monitoring.** The rationale is measuring timings and errors without per-account dimensions.
- **Simulation tick p99 alarm at 5 seconds.** The rationale is catching degradation before users notice the aviary "running slow."
- **Telemetry privacy review.** The rationale is to ensure new metrics do not touch per-bird or per-account interaction state.
- **Not measuring per-bird drift trajectories.** The rationale is that it would aggregate the user's relationship with their birds.
- **Not measuring per-account visit frequency.** The rationale is that it would be the data substrate of the streak counter the plan refuses to build.
- **Not computing leaderboard-like metrics.** The rationale is that absence of the metric makes reappearance of the feature harder.
- **Unsupported-browser matter-of-fact surface.** The rationale is no compatibility bloat for very old browsers.

### Rollout

- **Internal dogfood to private beta to general availability.** The rationale is staged validation before GA.
- **At least four weeks of dogfood.** The rationale is real-data validation for 7-day measurable and 21-day visible drift targets.
- **Feature flags per major surface.** The rationale is disabling a regressed surface without disabling the product; flags default on for GA.
- **Birds-per-aviary ramp by aviary age.** The rationale is that pacing should match "a deepening relationship, not reward engagement."
- **New-bird moment in naturalist voice.** The rationale is to avoid counters or rewards while surfacing "a new bird is here."
- **Aggregate RUM and synthetic checks from private beta.** The rationale is instrumenting from day one.
- **Drift harness blocking merge.** The rationale is that drift regressions break the central claim.
- **Privacy boundary reviewed for telemetry changes.** The rationale is that every new metric must show it does not touch per-bird or per-account interaction state.
- **Notebook generation rate metric.** The rationale is to confirm the sparsity target without surfacing a user-facing measure.
- **Drift-coefficient calibration metrics.** The rationale is instrument-only tuning, not user-facing product.

### Risks and mitigations

- **Drift calibration risk.** The rationale for the mitigation is that coefficients too fast become Tamagotchi and too slow become screensaver; CI and dogfood tune the narrow band.
- **Sync correctness risk.** The rationale for the mitigation is construction over policy: single writer, client never owns state, additive deltas, UUIDv7 idempotent events.
- **Audio uncanniness risk.** The rationale for the mitigation is preserving procedural recognizability and avoiding canned audio, phase-canceling choruses, and jarring fallback.
- **Accessibility regression risk.** The rationale for the mitigation is reviewing narration as observation, reduced motion as its own aesthetic, captions from grammar, and keyboard nav in acceptance criteria.
- **Presence signal integrity risk.** The rationale for the mitigation is preventing inflated drift through the three-signal conjunction and dogfood calibration of the 180s activity window.
- **Privacy boundary erosion risk.** The rationale for the mitigation is credentials-layer separation and absence of leaderboard-like queries.
- **Engagement-feature temptation risk.** The rationale for the mitigation is that streak counters, badges, and user-visit notebook entries violate the product identity.
- **Personality vector exposure risk.** The rationale for the mitigation is serialization-layer enforcement: snapshots include mood and perch, not traits, and the response builder lacks trait access.
- **Identity continuity risk.** The rationale for the mitigation is permanent `bird.id`; replacing a bird would invalidate drift and break the central promise.
- **First-frame continuity risk.** The rationale for the mitigation is that inline snapshot and non-blocking render path make birds mid-action achievable without spinner/fade-in.

### Open calibrations

- **Tick cadence starting at 60s.** The rationale is p99 tick latency under 5s and felt-continuity in dogfood.
- **Presence activity window starting at 180s.** The rationale is "watching without moving is the product," calibrated through active vs. passive watcher cohorts.
- **Mood state set including `settled`.** The rationale is to give settle and night a first-class state.
- **Drift coefficients seeded from synthetic-pop simulation.** The rationale is CI calibration against 7-day / 21-day targets.
- **New-bird pacing at 60/120/240/365/540 days.** The rationale is aviary-age only, never engagement.
- **Notebook sparsity around one entry every few days.** The rationale is the target sparse naturalist cadence, enforced through generation-rate metrics and a throttle.
- **Offer cooldown starting at 180s.** The rationale is preventing curiosity saturation.
- **Listen-in ramp at 800ms engage/disengage.** The rationale is "listening, not switching channels."
- **Top-bar fade after 3s cursor stillness.** The rationale is dogfood review of the chrome fade timing.
