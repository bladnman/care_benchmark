## System-level intent

1. **Executable engineering specification, not product requirements.** The plan frames itself as an "executable engineering specification" that provides "architecture, data model, API contracts, pipeline designs, and operational boundaries" so a separate team can build v1 "without further clarification." It also says decisions must "trace back" to the plan and deviations require an "explicit engineering decision record."

2. **Ambient product, not engagement product.** The plan repeatedly excludes "achievements, streaks, badges, levels, scores, XP, calendars, leaderboards, visit counters," "Tamagotchi mechanics," push notifications, email digests, "engagement funnels," and "activation" metrics. The product is explicitly "not optimized for engagement time." UI review includes the "notice, never announce" principle.

3. **Gentle maturation rather than earned progress.** New birds are based on `aviary_age_days`, not interaction, and "must never feel earned by attention; they feel like the aviary maturing." Drift is slow: after a week instruments should measure change, and after three weeks drift should be "perceptible to a human observer."

4. **Non-punitive care model.** The drift function is monotonic: "all deltas are >= 0" and "Neglect produces zero delta, never negative." This shows up in the drift implementation notes as a "hard invariant" and in the risks around birds feeling like "Tamagotchis."

5. **Server-owned truth, client as renderer.** The plan insists that the Simulation Service is "the canonical state owner," that "client never computes personality drift," and that the "client is a renderer, not a simulation participant." Sync avoids reconciliation by having all clients read the same canonical snapshot.

6. **No numerical or state-list product voice.** Personality vectors are "never exposed to client," and guardrails forbid "numerical exposure." Screen-reader narration and captions use "naturalist prose," "naturalist voice," and "the same naturalist register as the field notebook"; they "Never state-list format" and "Never expose mood labels or perch zone names."

7. **Living procedural ambience without looping assets.** Calls are synthesized through WebAudio, with "micro-variance" so "no two calls are identical." The plan bans "recorded-audio fallback," external audio assets, and audio file imports. Bird visuals are "procedural SVGs or compact vector paths," and idle motion uses deterministic seeds.

8. **Continuity, softness, and no abrupt jumps.** Rendering uses interpolation, cross-fades, ease-in-out arcs, continuous day/night palette changes, and birds that appear "as if they have always been there." Mood changes have "No sudden snap"; settle ramps lighting and audio rather than stopping instantly.

9. **Accessibility preserves the experience instead of stripping it.** Reduced-motion mode is "cross-fade rendering, not stripped fallback." It removes animated paths and leaf drift, but preserves "call audio," "mood changes, drift, notebook, all interaction affordances." Screen-reader and keyboard flows are part of Definition of Done.

10. **Privacy boundary is architectural, not only policy.** Internal references use "synthetic UUIDs," email is "encrypted at rest," deletion has soft and hard phases, and per-bird interaction data is isolated from telemetry. The plan deliberately does not measure "per-bird behavioral analytics," individual session recordings, or engagement funnels.

11. **Bounded social presence.** Social is "opt-in," "read-only," "ambient," revocable, and has "no notifications by default." Non-goals exclude profiles, follows, discovery feeds, comments, friend-of-friend chains, and explore surfaces.

12. **Performance budgets are product constraints.** The 2MB initial bundle, <500ms time-to-first-bird, 60fps idle motion, and zero memory growth budget recur in scope, rendering, audio, performance, CI gates, rollout, and risks. The plan says not to "silently raise" the 2MB cap because time-to-first-bird depends on it.

13. **Tunable, testable calibration.** Drift constants, notebook heuristics, bird caps, age thresholds, and feature behavior are tunable through configuration or flags. The plan calls for deterministic harnesses, CI calibration tests, synthetic monitoring, RUM, and phased launch checks.

## Per-feature whys

### Scope

- **Aviary runtime: 2 starter birds per new account.** NOT RECOVERABLE FROM PLAN

- **Aviary runtime: cap at 7 birds.** NOT RECOVERABLE FROM PLAN

- **Bird availability by aviary age.** The rationale is explicit: new birds become available based on `aviary_age_days`, "not user interaction," because "new birds must never feel earned by attention; they feel like the aviary maturing."

- **Single-user accounts.** NOT RECOVERABLE FROM PLAN

- **Email magic-link sign-in.** NOT RECOVERABLE FROM PLAN

- **Per-device revocable sessions.** NOT RECOVERABLE FROM PLAN

- **Personality vectors.** They support drift while staying hidden: the server stores canonical personality, seed values vary by species and bird, deltas are additive, and vectors are "never exposed to client" or serialized. The plan's rationale is to let birds change over time without creating numerical surfaces.

- **Five personality traits.** NOT RECOVERABLE FROM PLAN

- **Mood system.** Mood is needed for rendering and audio: snapshots include current mood because "client needs it for rendering," and mood shapes idle motion and call transforms.

- **Drift function.** The rationale is gradual, measurable, non-punitive change. It must be monotonic, produce no negative deltas, show measurable change after regular visits, and become visible only over weeks so birds do not feel inert or like Tamagotchis.

- **Procedural call grammar.** The tick computes when a bird calls and which motif family to use, while the client synthesizes audio. This keeps simulation canonical, avoids recorded loops, supports chorus windows, and lets calls vary by personality and mood.

- **Idle motion system.** Deterministic idle seeds let "two clients viewing the same aviary at the same moment see the same micro-motion without requiring high-frequency sync." Mood-shaped motion also gives visible life to wary, content, curious, drowsy, and alert states.

- **Bird-to-bird interaction.** NOT RECOVERABLE FROM PLAN

- **Strict three-factor presence accounting.** The rationale is to avoid corrupt presence: visibility, focus, and recent input protect against browser behavior changes, sleep/wake edge cases, and a visible tab left open while the user walks away.

- **Listen-in.** Listen-in is meant to feel like moving closer: focused bird gain ramps up, other birds ramp down, ambient bed stays unchanged, and non-focused birds are "Never mute" because they remain part of the aviary.

- **Offer: seed / song fragment / still pool.** NOT RECOVERABLE FROM PLAN

- **Settle gesture.** Settle "ends presence cleanly," shifts lighting toward evening, quiets calls by ramping global mix gain, and supports undo within 5s. The rationale is a clean close to the session without birds stopping instantly.

- **Return-greeting.** The plan places return-greeting in narration priority, but does not articulate its rationale. NOT RECOVERABLE FROM PLAN

- **Field notebook.** The notebook is sparse and read-only, with noteworthiness heuristics for greeting-order reversals, first offer acceptance after a drought, weather events, and drift milestones. The rationale is to create occasional naturalist observations rather than continuous metrics or editable records.

- **Read-only notebook entries.** NOT RECOVERABLE FROM PLAN

- **Single horizontal scene.** The plan excludes panning, zooming, scrolling, customizable scenes, and multi-aviary accounts, but does not give a separate rationale for the single horizontal scene. NOT RECOVERABLE FROM PLAN

- **Three perch zones.** The plan uses front, middle, and back perch zones for composition and snapshots, but does not explain why exactly three zones. NOT RECOVERABLE FROM PLAN

- **Day/night cycle using local time.** It grounds the aviary in the user's local time: scene composition and mood use the local time anchor, and palette interpolates continuously based on local time.

- **Ambient weather.** Weather contributes to mood transitions and notebook noteworthiness, but the plan does not explain why weather is part of the product. NOT RECOVERABLE FROM PLAN

- **Ambient micro-motion.** The rationale is ambience without simulation state: leaf/feather drift is "pure rendering ornaments" owned by the client and does not affect canonical state.

- **Top-bar chrome with auto-fade.** NOT RECOVERABLE FROM PLAN

- **Procedural call synthesis via WebAudio.** The rationale is to avoid recorded or looping audio, keep sound in the client, fit the bundle budget, and generate calls from oscillators, noise buffers, and wavetable lookups.

- **Chorus mixing.** Chorus windows should feel like "birds calling together, not like layered loops"; shared reverb, detune, and dry-gain reduction prevent phase cancellation and clipping.

- **Listen-in mix decay.** The rationale is a gentle audio focus that keeps the aviary present: ramps happen over 1.5s, ambient bed is unchanged, and other birds remain audible at ambient level.

- **Screen-reader narration.** The rationale is accessible observation in product voice: prose comes from snapshots, uses the field notebook's "naturalist register," avoids state lists, and prioritizes user-initiated events.

- **Reduced-motion mode.** The rationale is to honor reduced motion while preserving the product: still poses and cross-fades replace frame-by-frame animation and flight paths, while calls, mood changes, drift, notebook, and interactions remain.

- **Call captioning.** Captions provide access to calls using the same procedural parameters and naturalist voice, with duration matched to the call and WCAG AA contrast.

- **Full keyboard navigation.** The rationale is complete non-pointer operation: top bar, aviary entry, bird focus, listen-in, offer palette, settle, and escape flows are all reachable by keyboard.

- **WCAG AA contrast on chrome text.** The rationale is readability for text surfaces; the scene has no text except captions, so contrast applies to chrome, panels, errors, and caption pills.

- **Server-side canonical simulation tick.** The tick is the single writer for personality and mood, which prevents client conflicts, stale personality writes, and snapbacks.

- **Multi-device sync by shared snapshot reads.** The rationale is conflict prevention: all clients read one canonical account state, and the client is "a renderer, not a simulation participant."

- **Additive personality deltas.** The rationale is conflict prevention and non-punitive drift: deltas are additive, commutative within the monotonic constraint, and no last-write-wins is needed for personality.

- **Opt-in visit invitations.** The rationale is bounded, private sharing: invitations are one-time links, per-email hashed, tokenized, expiring, and revocable.

- **Read-only ambient visitor view.** The rationale is to allow visits without social network or mutation surfaces: visitor snapshots share shape, but all mutation endpoints return `403`.

- **No notifications by default.** The rationale aligns with the product avoiding outbound messaging and engagement pressure.

- **Synthetic UUIDs for all internal references.** The rationale is privacy: all internal references use synthetic identifiers rather than direct personal identifiers.

- **Encrypted email storage.** The rationale is privacy and account lifecycle separation: Auth Service holds the encrypted email-to-UUID mapping.

- **Soft-then-hard deletion.** NOT RECOVERABLE FROM PLAN

- **Per-bird interaction data isolated from aggregate telemetry.** The rationale is to prevent privacy boundary erosion and avoid data that could justify gamification, leaderboards, recommendation features, or population-level analysis.

- **2MB initial JS bundle.** The rationale is first paint and first-bird speed: the plan says the 500ms time-to-first-bird depends on the cap.

- **<500ms time-to-first-bird.** The rationale is immediate presence: the boot script paints a quiet field right away, fetches snapshot in parallel, and fades birds in once ready.

- **60fps idle motion.** The rationale is smooth ambient life on a 5-year-old laptop while staying within an <8ms frame budget.

- **Zero memory growth over 30 minutes.** The rationale is long-running ambient stability: pre-allocation, reuse, virtualization, and CI memory profiling prevent leaks.

### Architecture

- **API Gateway.** It handles HTTPS ingress, auth/session validation, rate limiting, and routing while remaining stateless and auto-scaled. The plan does not provide a product rationale beyond service separation.

- **Simulation Service.** The rationale is canonical ownership: it owns tick scheduling, per-account state, notebook generation, snapshots, and is the single writer for personality state.

- **Auth Service.** The rationale is isolation of authentication and lifecycle concerns: magic links, sessions, account creation/change/export/delete, and encrypted email mapping live here.

- **Web Client.** The rationale is local rendering and audio: it renders the scene, synthesizes WebAudio, captures presence and interactions, and consumes snapshots without owning simulation state.

- **Client/server split.** The rationale is boundary clarity and sync correctness: server owns personality, moods, notebook, account metadata, sessions, logs, and visits; client owns rendering, audio, input capture, interpolation, ornaments, and narration.

- **Render pipeline layers.** Scene composition, animation, and audio are split because they run on different cadences: snapshot cadence, 60fps animation, and audio-thread clock.

### Data model

- **Account settings.** NOT RECOVERABLE FROM PLAN

- **`aviary_age_days`.** It exists to drive age-based bird availability and avoid interaction-earned progress.

- **Bird `species_id`, name, adoption fields.** NOT RECOVERABLE FROM PLAN

- **Server-only personality vector storage.** The rationale is no numerical exposure and no client-side mutation; only the simulation tick updates it.

- **Canonical mood table.** Mood needs timestamps and expiry so the tick can transition states and the client can render current mood.

- **Append-only presence event log.** The rationale is durable, aggregatable presence accounting by block, device, and event type.

- **Append-only interaction event log.** The rationale is to let clients append events while the tick consumes them later, preserving server ownership of state.

- **Ephemeral state snapshot.** The rationale is to serve client rendering needs without exposing personality vectors: it contains mood, pose, call timings, perch zone, idle seed, day phase, weather, and settle state.

- **Notebook entry tags.** Tags are "internal" and support generation logic such as greeting order and weather; the plan does not expose them to users.

- **Visit invitation token model.** The rationale is no account required for visitors while keeping invite lookup private through hashed email and one-time URL tokens.

### API surface

- **Magic-link request and verify endpoints.** NOT RECOVERABLE FROM PLAN

- **Session listing and revocation endpoints.** NOT RECOVERABLE FROM PLAN

- **Authenticated snapshot endpoint.** The rationale is shared canonical state for client rendering.

- **Visitor snapshot endpoint.** The rationale is accountless read-only access through a token, with the same snapshot shape and scoped visit token.

- **Batchable event append endpoint.** The rationale is to reduce request count while preserving interaction and presence events for tick processing.

- **Notebook pagination endpoint.** The rationale is reading immutable sparse text entries; max limits bound the request.

- **Account settings endpoint.** The rationale is settings mutability across devices; optimistic concurrency with `etag`/`version` handles rare preference conflicts.

- **Account export endpoint.** NOT RECOVERABLE FROM PLAN

- **Account delete and recover endpoints.** NOT RECOVERABLE FROM PLAN

- **Visit invitation create/list/revoke endpoints.** The rationale is opt-in, revocable, host-controlled sharing.

- **Snapshot polling on visibility, keepalive, interactions, and wake gaps.** The rationale is to refresh canonical state after lifecycle changes, user actions, and long render-frame gaps.

- **Event batching every 10s or on pagehide.** The rationale is fewer requests "without losing critical events."

- **Event idempotency by `batch_uuid`.** The rationale is safe retries through server deduplication.

- **Visitor token in query parameter.** The rationale is that it "works without an account session."

### Simulation engine design

- **Per-account tick scheduler.** The rationale is account-local simulation with jitter to prevent a thundering herd.

- **Tick retries with same `tick_id` and actual wall-clock delta.** The rationale is idempotent, retrievable ticks where jitter or failure does not affect correctness.

- **Processing events before drift.** The rationale is to compute presence, listen-in, offer, and settle inputs for drift and mood.

- **Trait-specific drift weighting.** NOT RECOVERABLE FROM PLAN

- **Calibration constants.** The rationale is testable and tunable behavior: changes must be measurable after one week and visible after three weeks.

- **Mood transitions from time, weather, bird influence, and personality gates.** The rationale is to make mood respond to ambient context, neighbors, and personality while staying server-computed.

- **Call schedule advancement.** The rationale is to base future calls on vocal frequency, mood, and chorus probability while leaving waveform synthesis to the client.

- **Conditional notebook generation.** The rationale is sparsity plus noteworthiness: entries require a time gap and notable events such as drift milestones or weather.

- **Transactional persistence.** The rationale is consistency across vectors, moods, schedules, processed event markers, and notebook entries.

- **Configurable drift constants.** The rationale is production calibration without deploys.

- **Deterministic drift test harness.** The rationale is CI-enforced calibration and reproducible expected output.

- **Structural no-negative-drift invariant.** The rationale is non-punitive care and unit-testable guarantee.

- **Chorus group computation in tick.** The rationale is to let the client apply chorus mixing when birds call within a two-second window.

### Sync model

- **One canonical aviary state per account.** The rationale is shared snapshot reads across devices and no client-side reconciliation.

- **No client-side state reconciliation.** The rationale is that the client is a renderer, so there are no simulation conflicts to resolve.

- **Last-write-wins for account settings.** The rationale is that preferences are "small, infrequent" and user intent is clear; `etag`/`version` handles concurrency.

- **Last-write-wins for bird names with timestamp check.** The rationale is that rename is a "single-field overwrite," idempotent, and user intent is unambiguous.

- **Offline rendering from last snapshot.** The rationale is continuity: idle motion, ambient drift, and cached call schedules keep the aviary alive during network loss.

- **Reconnect interpolation.** The rationale is to avoid jumps by interpolating from extrapolated local state to canonical state over about one second.

- **Offline event queueing and retroactive drift.** The rationale is to preserve interactions while offline and apply drift based on event timestamps, not arrival time.

- **Per-device presence.** The rationale is that presence across laptop, phone, and tablet is device-specific; additive counting is acceptable because simultaneous active presence still means the user is present.

### Frontend rendering pipeline

- **Full-viewport canvas or WebGL scene.** Canvas 2D is preferred for "performance and pixel-level control"; WebGL is acceptable only if the team can handle the complexity.

- **Layered scene composition.** Back-to-front layers support visual depth, weather overlays, foreground ornaments, and an HTML top bar for accessibility and input handling.

- **Deterministic idle motion seed.** The rationale is same-moment visual consistency across clients without high-frequency sync.

- **Mood-shaped motion sets.** The rationale is to make mood observable through movement: wary scans, content preens, curious tilts, drowsy minimal motion, alert quick scans.

- **Perch transition arcs.** The rationale is to feel like flight: a "gentle arc, not a straight line."

- **Mood cross-fades.** The rationale is to avoid sudden snap between motion sets.

- **Continuous day/night interpolation.** The rationale is no discrete jumps.

- **Settle trigger lighting and audio ramp.** The rationale is a gentle evening shift where "birds don't stop instantly."

- **Undo settle within 5s.** NOT RECOVERABLE FROM PLAN

- **Reduced-motion still-pose sequence.** The rationale is lower motion through cross-fades while keeping the same moods and interactions.

- **Quiet field loading state.** The rationale is a calm first paint with no spinner, progress bar, or loading text; birds fade in as if already present.

- **Empty-aviary entrance animation.** NOT RECOVERABLE FROM PLAN

### Audio pipeline

- **Species motif libraries.** NOT RECOVERABLE FROM PLAN

- **Personality and mood audio transforms.** The rationale is audible individuality and state: vocal frequency affects timing, boldness amplitude/brightness, and mood changes call shape.

- **Micro-variance.** The rationale is explicit: "This ensures no two calls are identical."

- **AudioWorklet.** The rationale is to keep audio off the main thread.

- **Chorus detune, reverb, and dry-gain changes.** The rationale is to avoid phase cancellation, clipping, and layered-loop feel.

- **Listen-in engage/disengage ramps.** The rationale is focused listening without removing the rest of the aviary.

- **Graceful silence fallback.** The rationale is dignified fallback when AudioContext fails: stop synthesis, enable captions, show a small muted indicator, and do not provide recorded audio fallback.

- **Audio sub-budget.** The rationale is protecting the 2MB bundle; audio code and motif data target <300KB.

### Accessibility surfaces

- **Hidden polite live region.** The rationale is screen-reader narration without disrupting the DOM or user flow.

- **Client-side narration generation.** The rationale is to avoid extra server round-trips.

- **Narration cadence and priority queue.** The rationale is to balance ambient updates with immediate feedback for user-initiated events.

- **Narration voice continuity.** The rationale is the same naturalist register as the field notebook and no state-list format.

- **Positioned call captions.** The rationale is to connect text to the calling bird while reflecting the call's procedural parameters.

- **Caption dark background pill.** The rationale is WCAG AA contrast against the scene.

- **Keyboard focus order and bird navigation.** The rationale is complete keyboard operation in DOM order and visual left-to-right bird order.

- **Visible focus indicator.** The rationale is visibility against all day-phase palettes.

- **Chrome contrast requirements.** The rationale is that text appears only in chrome and captions, so those surfaces must pass contrast.

- **Avoid relying on color alone.** The rationale is accessible state indication.

### Performance budgets and observability

- **Code splitting.** The rationale is protecting first paint: core aviary runtime loads synchronously, while account, accessibility settings, and visits can be lazy-loaded.

- **Procedural/vector asset strategy.** The rationale is bundle size: no large bitmap assets, compact species templates, and tiled SVG foliage.

- **Minimal boot script and parallel snapshot request.** The rationale is <500ms time-to-first-bird by painting immediately and overlapping snapshot fetch with bundle download.

- **HTTP/2 server push or 103 Early Hints for snapshot.** The rationale is faster snapshot availability when the auth token is present in a cookie.

- **Frame budget and throttled non-visual updates.** The rationale is 60fps idle motion with <8ms render/update time.

- **Memory reuse and CI heap test.** The rationale is zero net memory growth over 30 minutes.

- **WebAudio latency settings.** The rationale is avoiding glitches through interactive latency and 100ms lookahead.

- **Synthetic monitoring.** The rationale is continuous external measurement of first-bird time, bundle download, snapshot latency, and audio init success across geographies.

- **Aggregate RUM.** The rationale is production performance visibility without per-bird behavioral analytics.

- **Server metrics and alert thresholds.** The rationale is operational response to tick latency, snapshot latency, event lag, endpoint errors, audio failure, and first-bird regressions.

- **No per-bird behavioral analytics.** The rationale is to protect the privacy boundary and avoid data that could justify gamification or leaderboards.

- **No individual-user session recordings.** The rationale is privacy; no FullStory or Hotjar-style replay tools.

- **No engagement funnels.** The rationale is that the product is not optimized for engagement time.

### Rollout

- **Internal alpha.** The rationale is to validate tick cadence, drift calibration, audio mix, and notebook tuning on real devices with 50 accounts.

- **Closed beta.** The rationale is to stress-test sync, multi-device usage, presence accuracy, visit feature, tick latency, and event-log growth with 1,000 invite-only accounts.

- **Public beta.** The rationale is to validate performance budgets at scale, gather accessibility feedback, and tune call-caption heuristics.

- **General availability.** NOT RECOVERABLE FROM PLAN

- **`max_birds_per_aviary` feature flag.** The rationale is controlled ramping of bird caps by environment.

- **Soft chirp cue for pending adoption.** NOT RECOVERABLE FROM PLAN

- **Age thresholds for birds 3-7.** The rationale is maturation rather than earning; exact thresholds are "tunable constants" to validate.

- **Instrumentation from day one.** The rationale is early validation of server metrics, synthetic checks, privacy boundaries, accessibility, and load.

- **Feature flags.** The rationale is configuration, kill switches, and calibration tuning.

### Risks

- **Drift calibration risk mitigation.** The rationale is continuity: tuning too fast makes birds feel like Tamagotchis; too slow makes the product inert. Config, alpha, v2 flags, and CI tests reduce this risk.

- **Sync correctness risk mitigation.** The rationale is to prevent stale personality caching and snapbacks through strict separation, snapshot versioning, and durable event logs.

- **Audio uncanniness mitigation.** The rationale is that users should not "hear the algorithm"; procedural sound design needs expertise, internal pass/fail gates, and dignified silence.

- **Accessibility regression mitigation.** The rationale is to prevent performance work and refactors from breaking live regions, reduced motion, and keyboard reachability.

- **Presence accounting mitigation.** The rationale is to avoid spurious presence or indefinite visible tabs through browser testing, recent input windows, pagehide events, and sanity filters.

- **Privacy boundary mitigation.** The rationale is to prevent analytical creep from offer histograms to average drift speed through network isolation, review approval, and schema contracts.

- **Performance budget mitigation.** The rationale is to prevent "just raising the cap" and instead protect first-bird time through CI gates and sub-budgets.

- **Visit abuse mitigation.** The rationale is to prevent public ambient streams through per-email invitations, expiring tokens, host revocation, no iframe, and ToS prohibition.

### Engineering discipline and guardrails

- **No client-side personality mutation.** The rationale is server-owned personality and sync correctness.

- **No numerical exposure.** The rationale is to keep personality vectors out of client-facing JSON and production debug panels.

- **No gamification primitives.** The rationale is to block streaks, achievements, levels, scores, badges, XP, and similar UI strings from entering the product.

- **No recorded audio.** The rationale is to enforce procedural audio and prevent audio file imports.

- **No announcement toasts.** The rationale is "notice, never announce" in the aviary scene and on session start, while still allowing system surfaces for auth, errors, and settings.

- **PR gates.** The rationale is to ensure tests, bundle reporting, accessibility scanning, and manual review for notice-never-announce on every change.

- **Weekly calibration review.** The rationale is to inspect drift rates during alpha/beta and adjust constants.

- **Pre-release checklists.** The rationale is to verify screen-reader flow, keyboard-only use, reduced motion, memory profile, and throttled 4G loading before release.

### Ambiguities and defensible calls

- **60s +/- 5s tick cadence.** The rationale is to implement "~once per minute" while using actual wall-clock delta so jitter does not affect correctness.

- **3-minute presence activity window.** The rationale is a starting calibration for "a few minutes," with beta tuning and client-side ping stopping rather than server enforcement.

- **Adding `settled` mood.** The rationale is consistency: `settled` is used as an adjective for lighting state and should be represented in the mood enum.

- **Noteworthiness-score notebook model.** The rationale is to implement sparse naturalist entries with tunable, instrumented generation despite the exact scoring function being left to implementation.

- **Canvas 2D choice.** The rationale is performance and visual control; WebGL is acceptable if expertise exists, but the critical requirement is 60fps.

- **Leaving synthesis method to the audio engineer.** The rationale is that the PRD mandates procedural synthesis but not FM, subtractive, wavetable, or another method; the plan instead mandates budget, variation, and WebAudio-only constraints.

- **Graduated new-bird schedule.** The rationale is to make birds arrive through aviary maturation; exact 60/120/180/270/365 day thresholds are tunable and should be validated.
