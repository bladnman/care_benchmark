## System-level intent

1. **Affective promises must be architectural, not remembered.** The plan opens by saying the PRD's promises only hold if they "survive a year of well-meaning contributions" and that "Reviewer memory isn't enough." That intent shows up in the hard rules, DB grants, triggers, copy lint, banned APIs, launch gates, PR checklist, and "refused, and enforced as refusals."

2. **The aviary should feel alive without announcing itself.** The plan repeatedly pairs aliveness with restraint: "The first frame is mid-action," "There are no spinners," "no canned greetings," "no identical repeats," and "Nothing announces." Boot, rendering, audio, greetings, notebook, top bar, and visits all carry this same quiet-field posture.

3. **Change is slow, cumulative, and never punitive.** Personality drift is "monotonic," "traits never decrease," and "drift slowly over weeks." The plan says calibration "errs slow at launch" because fast drift "can't be undone without decreasing traits." Absence drains attention expression but never creates distress, hunger, sadness, or trait loss.

4. **Presence is narrow on purpose.** The plan insists that presence requires visible, focused, trusted pointer or key activity, and "Nothing else counts." The rationale is visible in the risks around "presence inflation or corruption," the truth-table suite, and the user-product stance that "sitting still and watching is the product" while hidden tabs, unfocused windows, and visitors do not accrue presence.

5. **Privacy is a product boundary, not a reporting choice.** The plan uses phrases like "Privacy boundary as architecture," "synthetic UUIDs everywhere," "Email lives encrypted in exactly one table," and "NO credentials for and NO network route to sim_db." It also says collecting engagement data would "create the data product" the PRD refuses.

6. **Accessibility is a designed surface of the same product.** The plan says accessibility "ships in v1 as designed surfaces" and later says every accessibility surface gets "the actual product." Reduced motion must read as "calmer and deliberate, never as broken"; disabled-user sessions must not find a "degraded variant."

7. **The scene is ambient, not gamified or social-networked.** The plan treats gamification, Tamagotchi mechanics, feeds, follows, counters, badges, push, and re-engagement email as refusals. It makes each refusal "hard to reverse by accident" through missing schema, copy lint, banned APIs, and the non-goals audit.

8. **The server owns reality; the client performs it.** The plan's split is explicit: the server authors "what happens and when" and the client authors "how it looks and sounds." A "single writer" tick prevents client-to-client sync, merge, and last-write-wins personality.

9. **Determinism protects continuity.** Seeded PRNGs, pure packages, golden replays, property tests, and engine versions all support bit-for-bit replay, calibration, and rollback without rebuilding birds or mutating identity.

10. **Bird identity matters more than control.** The plan refuses "removing or releasing a bird," says birds "persist for the life of the account," pins `bird_id`, voice signatures, and art revisions, and forbids migrations that regenerate a bird. Newcomers become birds only when welcomed.

11. **Traits should be felt through behavior, never read as stats.** The plan says "Traits act only through behavior," snapshots carry "derived, quantized expression parameters," and a plain `boldness` key is how "show my bird's stats" features start.

12. **Voice has two registers and no announcement framing.** Naturalist voice is "lowercase by default," present tense, bird-named, with "No exclamation marks" and "No 'you'." System voice is "sentence case, direct, and plain" for identity, errors, settings, and account surfaces.

13. **Operational learning must not become engagement measurement.** The plan allows aggregate operational telemetry, lab gates, synthetic monitoring, and consenting research interviews. It explicitly refuses DAU/MAU, retention cohorts, visit frequency, A/B tests on engagement, and any per-account or per-bird telemetry.

## Per-feature whys

### 0. The hard rules and refusals

- **Server tick as the only writer of personality and mood.** Why: it enforces that traits never decrease, trait values never reach product surfaces, and personality has no last-write-wins path.
- **Presence invariant and single presence module.** Why: the plan treats laxer presence as corruption; the exact visible/focused/activity conjunction prevents background tabs, unfocused windows, visitors, and overlapping devices from inflating drift.
- **Anti-announcement rule: no toasts, welcome text, badges, unread dots, streaks, counters, push, or re-engagement email.** Why: these surfaces would announce, create engagement pressure, or reverse the quiet ambient premise.
- **Procedural calls and rejection of recorded audio.** Why: recorded audio would break the aliveness promise and create repetition; procedural synthesis keeps calls fresh and allows captions from the same grammar.
- **First frame mid-action, no spinners, no canned greetings, no identical repeats.** Why: the aviary should appear already living, not waking up or waiting for the app to be ready.
- **Synthetic UUIDs, encrypted email in one table, no per-account data leaving sim_db.** Why: privacy is enforced through storage, credentials, network paths, logging allowlists, and PII lint rather than convention.
- **Accessibility launch gate with narration, reduced motion, captions, and keyboard reach.** Why: accessibility is part of v1's designed product surface, not a later fallback.
- **Gamification refused.** Why: achievements, streaks, scores, badges, XP, visit calendars, and similar vocabulary would surface user behavior and pull the aviary toward engagement mechanics.
- **Tamagotchi mechanics refused.** Why: death, hunger, distress, happiness decay, negative poses, and absence-driven wary state would contradict monotonic traits and non-punitive absence.
- **Social-network surfaces refused.** Why: profiles, feeds, follows, discovery, comments, chat, avatars, leaderboards, and mutual visits would create cross-account statistics and social pressure instead of ambient visits.
- **Notifications refused.** Why: push, pings, and re-engagement email are announcement mechanics; only transactional email and the opt-in visit notification are allowed.
- **Native apps refused.** Why: v1 is "Web only," and the plan avoids native constraints or installability features beyond a basic web manifest.
- **Payments, shared aviaries, multiple aviaries, custom scenes, and user-arranged perches refused.** Why: the plan makes them absent from the data model so they cannot slip in accidentally.
- **Trait numbers refused in product surfaces, with export as the only exception.** Why: product surfaces should not turn birds into stats; the export exception is for portability and is never rendered or summarized.
- **Removing or releasing a bird refused.** Why: the plan says it would contradict identity continuity; birds persist for the life of the account.
- **SSO and password login.** NOT RECOVERABLE FROM PLAN

### 1. Scope and interpretive decisions

- **One aviary per account and one horizontal scene.** Why: one canonical aviary supports the "same aviary, same mood" model across devices and visitors; one row per account also makes multiple aviaries structurally unavailable.
- **Species pool of six species.** NOT RECOVERABLE FROM PLAN
- **Nightjar-like night caller.** Why: the mood model gives the nightjar-like species an inverted prior so the aviary has late-evening and night activity without making ordinary birds unnaturally active.
- **Two starter birds chosen by the system.** Why: the engine can prefer contrasting silhouettes and call registers, keep the nightjar out of starters, and avoid a catalog-style selection surface.
- **Growth from two birds to seven by aviary age.** Why: newcomers must not depend on visit count, interaction volume, or payment; seven is tied to audio recognizability and is ramped only after performance and recognition gates.
- **Return-greeting.** Why: it lets the birds notice the user's return while varying by absence, boldness, mood, attention, and seed so it does not read as canned.
- **Listen-in with Enter or Space, not focus alone.** Why: focus-driven listen-in would make arrowing across birds "feel like switching channels," which the plan explicitly avoids.
- **Gesture tray under the offer icon, including settle.** Why: it keeps the top bar at exactly four icons while placing settle beside offers as "a gesture toward the aviary" rather than in system-voice settings.
- **Offers that are not aimed at a bird.** Why: the server can pick responding birds by curiosity, mood, proximity, attention, and cooldown, matching the rule that offers cannot be made by clicking a bird.
- **Settle lighting at night.** Why: settle should "never wake up a night scene"; when the scene is already darker than dusk, it deepens and warms instead of brightening.
- **Active visit passes lapse after 90 days without a visit.** Why: abandoned passes should not turn into a permanent visitor list.
- **Client-generated narration.** Why: it avoids extra round trips, keeps timing with visible motion, and shares one grammar with captions and notebook.
- **Mute not used as a drift input.** Why: weighting drift by audio would ration the product by sensory ability and contradict the accessibility stance.
- **Visitor view in the host's time zone with default ambient mix.** Why: mirroring listen-in, settle lighting, greetings, or notebook would amount to co-presence and expose private session-local state.
- **Invite emails naming the host by account email address.** Why: there are no display names or profiles, and the host consents when sending.
- **Top bar behavior on touch devices.** Why: a tap in the faded bar region reveals it without activating anything, preventing accidental activation of invisible controls.
- **Localization deferred.** Why: the naturalist grammar is English-specific in v1.
- **Multi-region writes deferred.** Why: v1 uses one primary region, a global edge, and later read caches as the lever, keeping writes simple.
- **Raising the seven-bird cap deferred.** Why: it needs future work on audio recognizability.
- **Third-party LLMs ruled out for prose.** Why: no third-party model may receive aviary state because the plan says it is "Never shared with any third party."

### 2. Architecture

- **Edge streaming, 103 Early Hints, and an inline shell.** Why: they support first-bird time by getting the quiet field, scene core, and snapshot moving in parallel.
- **Client/server split.** Why: the server stays authoritative for personality, mood, timelines, calls, weather, and interactions, while the client performs ornament, synthesis, captions, narration, and session-local presentation.
- **Intent timeline and pure choreographer.** Why: an open tab can draw a bird mid-preen on the first frame, and two devices or a visitor can see the same perch changes and calls after clock offset.
- **TypeScript shared packages for engine, grammar, voice, and protocol.** Why: the same code can run in client, API, workers, harnesses, and tests.
- **Seeded PRNG.** Why: replays and cross-environment tests are deterministic, and server decisions do not depend on bit-exact JS transcendental math.
- **No framework on the scene path; Preact only for panels.** Why: the scene path stays small and fast while panels can be code-split.
- **Separate ops telemetry plane with no route to sim_db.** Why: observability should not have credentials or network access to per-account interaction and simulation state.

### 3. Data model

- **Encrypted email and blind index in `auth_db.accounts`.** Why: email lookup and rate limiting work without email leaving the auth service; normalization avoids treating dots or plus tags specially.
- **No IP addresses or geolocation in sessions.** Why: session management should not accumulate extra identity or location data.
- **Visit invitations never joined to accounts by email.** Why: visitor passes should not create a friend graph, suggestions, or cross-account identity links.
- **Append-only events and presence intervals.** Why: clients append facts for the server tick to fold; personality has no client-submitted values.
- **DB roles, immutable columns, and monotonicity trigger.** Why: only the tick role can mutate traits and state, immutable identity fields cannot change, and any trait decrease is rejected below the application layer.
- **Snapshots with derived, quantized expression parameters.** Why: devtools is not a product surface, but a client key like `boldness` is how numeric bird-stat features start.
- **Trait daily snapshots.** Why: they make targeted restore possible for the "worst failure" without being used as runtime state.
- **Retention windows for raw events, greetings, happenings, sessions, logs, backups, and exports.** Why: the plan limits operational stores while preserving life-of-account birds, traits, names, signatures, and notebook.

### 4. API surface

- **Schema versioning with client support for N and N-1.** Why: major mismatches reload at a visibility change, not mid-view and not behind a spinner.
- **Opaque cookie sessions plus Origin and custom header checks.** Why: they provide CSRF defense in depth for mutating calls.
- **`server_now` on every response.** Why: clients estimate clock offset for timeline, audio, and snapshot ordering.
- **Client-generated UUIDv7 ids for event and offer submissions.** Why: mutating submissions are idempotent and duplicates can return the original ack.
- **Magic-link GET landing page that does not consume the token.** Why: corporate link scanners that pre-fetch GET URLs should not use up sign-in links.
- **Snapshot endpoint with `ETag` as `tick_seq.projection_seq`.** Why: keepalives can be cheap 304s while clients still compare canonical and projection order.
- **Visitor snapshot variant.** Why: visitors get the canonical ambient aviary without greeting, newcomer, notebook, or interaction fields.
- **Presence events emitted at most every 30 seconds and on loss.** Why: the server validates bounded intervals and a crash or pagehide loses only a small amount of presence.
- **`listen_in` partial intervals every 30 seconds.** Why: a long listen-in can survive crashes with at most 30 seconds of lost credit.
- **`greeting_seen`.** Why: the notebook should only treat a greeting as observed if it actually played.
- **`tz_report`.** Why: the account's canonical IANA time zone can follow the device that most recently accrued real presence.

### 5. Simulation engine

- **Pure `packages/engine`.** Why: the same no-I/O functions can be used by the tick worker, API, calibration harness, and property tests with explicit `now` and seeded randomness.
- **Per-aviary tick phase offset.** Why: load is smooth rather than spiking on the minute.
- **Ticks for every aviary, even pending deletion.** Why: a restored aviary has kept living.
- **`dt`-correct tick integration.** Why: expedited ticks, outage catch-up, and future tiered cadence for dormant aviaries can produce identical results.
- **Ordered tick pipeline.** Why: presence, attention, drift, weather, mood, cooldowns, planning, happenings, and newcomer checks have dependencies the plan says make order matter.
- **Client presence detection with visible, focused, trusted pointer/key activity.** Why: sitting still and watching should count for a while, but hidden, blurred, automated, scrolled, resized, and visitor activity should not.
- **Activity window calibrated between 2 and 6 minutes, initially 4 minutes.** Why: the plan leans longer because "sitting still and watching is the product."
- **Server trimming of suspicious presence intervals.** Why: long, future, late, or overlapping intervals are unverifiable and could inflate drift.
- **Cross-device union of presence.** Why: a laptop and phone open at the same time should credit ten minutes, not twenty.
- **Account-level absence for greetings.** Why: returning to one device soon after using another should be treated as a short absence.
- **Daily saturation for presence, listen-in, and offers.** Why: marathon sessions and button-mashing should not accelerate drift linearly.
- **Low-pass attention filters.** Why: a single session barely moves the filter, while the tail keeps drift going gently for a few days after the user leaves.
- **Headroom drift function.** Why: traits approach one but never reach it, leaving room for years of drift.
- **Anomaly clamp.** Why: at calibrated constants it should almost never fire, but it limits accidental daily drift and reports only aggregate counts.
- **Starter trait separation in boldness.** Why: greeting order is legible from the start.
- **Expression attention that decays with absence.** Why: returning users get quieter greetings and less approach without traits, base call rate, plumage, or mood valence decreasing.
- **Trait-to-behavior mapping.** Why: traits become front-perch time, greeting probability, call hazard, plumage saturation, offer approach, and head-tilt rather than exposed numbers.
- **Calibration harness.** Why: constants are tested against personas so regular use is visible over weeks, heavy use does not saturate too soon, single sessions do not visibly move traits, and launch can "err slow."
- **Mood states with no distress, hunger, sadness, or anger.** Why: the mood model must not become a Tamagotchi.
- **Mood transitions over tens of minutes.** Why: the user should read mood from motion instead of watching it flicker.
- **Night roosting and dawn waking reset.** Why: tab open should show where the server simulation left the birds, not snap to a fresh state.
- **Absence excluded from mood scores.** Why: birds should not become wary because the user was away.
- **Wary spreading through alarm calls only.** Why: wary can spread as an aviary event while still reading as scanning farther back, never cowering or distress.
- **Daylight from IANA time zone representative coordinates.** Why: the aviary gets seasonally plausible light without asking for or storing location.
- **Soft rain and wind generation.** Why: weather can shape mood and expression but stays non-assertive, with no thunderstorms or snow.
- **Perch zones and slots sized for seven birds plus a newcomer.** Why: the scene can hold the planned cap without overlap.
- **Ambient calls with mutual excitation, call-response, and chorus.** Why: bird-to-bird interaction should emerge from scheduled social calls and be consistent across devices.
- **Five-voice audio budget.** Why: recognizability depends on avoiding chorus mush.
- **Greeting selection with stable dispositions and seeded realization.** Why: the same bird can greet in its own recognizable way while each greeting remains fresh.
- **Greeting fingerprints and no-repeat tests.** Why: greetings must never be identical twice or feel canned.
- **Greeting descriptor without text.** Why: "Nothing textual accompanies the greeting."
- **Synchronous offer resolution under a per-aviary lock.** Why: offers from multiple devices are serialized and outcomes are decided before drift credit and cooldowns.
- **Visible offer reactions even on cooldown.** Why: the offer affordance is never disabled and never shows a countdown; button-mashing should read as birds watching, not as an error or timer.
- **Failed offer still appears locally.** Why: birds glancing at an uncredited object is less disruptive than surfacing an error for a transient failure.
- **Song fragments synthesized from a small library.** Why: they sit outside every bird's signature space and are never audio files.
- **Settle with five-second undo.** Why: accidental clicks, taps, or keys can reverse the goodbye before it commits.
- **Settle ending presence without drift effect.** Why: settle is a session-local quieting gesture, not a personality input beyond ending the presence window.
- **No greeting after re-engaging from settle.** Why: the user never left.
- **Genesis with two arrivals and quiet-field fly-in.** Why: the user sees the birds as having arrived once, and after that never sees an empty aviary again.
- **Newcomer schedule by aviary age only.** Why: visits, interaction volume, and payment must never control bird availability.
- **Newcomer presentation as noticing, not announcing.** Why: the visitor bird can keep visiting with no expiry, countdown, reminder, or pressure.
- **Newcomers not counting as birds until welcomed.** Why: they have no personality drift and preserve the cap and identity boundary until the user lets one stay.
- **Absolute maximum of seven birds.** Why: the plan ties the cap to audio recognizability, performance, and rollout gates.
- **Voice distinctness sampling and `d_min`.** Why: every bird, even same-species birds, should be recognizable by call in a seven-bird aviary.
- **Identity-preserving art migrations.** Why: species art revisions must not regenerate a bird or change its silhouette family and palette anchors.

### 6. Sync model

- **One canonical aviary and one writer.** Why: multi-device sync is architectural; two devices read the same record and never reconcile personality.
- **Commutative, idempotent drift inputs.** Why: presence/listen-in unions and serialized offers mean device order does not matter and duplicates are no-ops.
- **Per-aviary event sequence numbers assigned under row lock.** Why: sequence order must equal commit order so a tick cannot skip a later-committing lower sequence.
- **Snapshot as canonical state plus projection.** Why: recent offer outcomes, scene objects, and greetings can appear before the next tick without persisting projection as canonical.
- **Timeline commit horizon.** Why: calls already scheduled into audio lookahead and flights already started should not be contradicted by the next snapshot.
- **Client keeps highest `(tick_seq, projection_seq)`.** Why: late and out-of-order responses should not roll the visible aviary backward.
- **Ambient continuation mode during network failure.** Why: the scene keeps living past the timeline horizon and "nothing ever freezes."
- **System-voice error line instead of spinner.** Why: persistent failures need direct system copy while the scene stays as it is.
- **Bird rename with `If-Match`.** Why: names are user-owned rather than personality, so a field-level compare-and-set is enough.
- **Per-field settings merge.** Why: settings are preferences, not simulation state.
- **Personality-vector repair procedure.** Why: lost or corrupted personality is the "worst failure," so repair is audited data recovery from snapshots, not new drift.

### 7. Frontend rendering pipeline

- **Quiet field before data arrives.** Why: slow snapshots should look like the aviary catching up, not a loading spinner.
- **First bird drawn at full opacity and mid-action.** Why: the first frame must never contain entry animation, fade from static image, welcome text, or a wake-up pose.
- **Service worker warm path with navigation preload and last snapshot.** Why: repeat visits can render birds before the network responds when the cached timeline still covers now.
- **Unsupported browser page.** Why: missing core web capabilities should receive a static, system-voice explanation; missing WebAudio is not a blocker.
- **Scene layers with DOM overlays outside the art.** Why: the scene itself carries no buttons, badges, tooltips, labels, or overlay icons.
- **Calm naturalist palette.** Why: the plan avoids saturated accent colors and imports soft design-system tokens.
- **Parametric bird rigs and procedural curves.** Why: birds can show breathing, preening, scanning, head tilts, shuffles, blinks, tail flicks, and flights without bitmap loops.
- **Mood-shaped idle style.** Why: mood should be read through posture and motion: wary scanning, content preening, curious tilting, drowsy fluffing, roosting eyes closed.
- **Anti-strobe animation rules.** Why: motion must avoid high-frequency periodic movement, toggles, and lockstep phases.
- **No negative-affect pose library.** Why: the rendering vocabulary should not imply distress, sickness, hunger, or sadness.
- **No per-frame allocations in the render loop.** Why: frame rate and memory stability depend on preallocated arrays and pools.
- **Responsive layout safe-area guarantees.** Why: every perch slot, flight path, bird bound, and touch target must fit from small portrait phones to ultrawide screens.
- **Top bar with exactly four icons.** Why: the plan maintains the four-item constraint while moving offers and settle into the gesture tray.
- **Top bar fade.** Why: stillness reveals the scene rather than empty chrome, while focus, panels, touch behavior, contrast preferences, and "Keep visible" prevent usability loss.
- **No indicators on top bar, document title, or favicon.** Why: unread dots, badges, "new" labels, title mutations, and favicon changes would announce.
- **Panels as side or bottom sheets with aviary still visible and audible.** Why: account, notebook, and accessibility work should not replace the aviary experience.
- **Notebook virtualization and bounded LRU.** Why: unlimited scrollback must not violate the memory rule.
- **Reduced-motion pose sequencer.** Why: reduced motion is a designed version of the same timeline, not a broken or static product.
- **Hidden tab suspension.** Why: rendering, audio, and presence stop when hidden, while the server continues ticking.
- **Visible but unfocused rendering and audio without presence.** Why: a user can glance at a second monitor, but the engine still does not count presence.
- **Focus proxies with canvas-drawn focus ring.** Why: keyboard reach follows moving birds without layout thrash, and the double ring remains visible against bright and dim scenes.

### 8. Audio pipeline

- **AudioWorklet `syrinx` with fixed pools.** Why: procedural audio needs predictable CPU and flat memory, with no allocation in `process()`.
- **Syrinx-inspired procedural voice model.** Why: it avoids recorded loops and supports bird-specific harmonics, contours, noise, envelopes, and two-voice species.
- **Safari ambient audio session.** Why: the aviary should mix with music and podcasts and respect the iOS silent switch.
- **Call grammar with signature invariants.** Why: each bird remains recognizable across mood and drift through timbre, pitch range, intervals, rhythm, and tag motif.
- **Mood modulating call expression but not voice identity.** Why: drowsy or alert calls can differ while drift changes how often a bird calls, not who it sounds like.
- **No identical `CallScore` within recent history.** Why: every call should be freshly realized rather than looped.
- **Server-timed call scheduling.** Why: ambient, response, and chorus calls align with the shared timeline across clients.
- **Recognizability gate.** Why: it is what justifies the seven-bird cap, using automated features and human listening panels.
- **Failure policy for recognizability at seven.** Why: the plan says to improve signature design or `d_min`, not quietly lower the cap without a product decision.
- **Listen-in mix ramps.** Why: gradual gain, reverb, and lowpass changes make it feel like listening rather than switching channels.
- **Other birds never muted during listen-in.** Why: the ambient aviary remains present even while one bird is foregrounded.
- **No listen-in visual chrome.** Why: clicking or focusing a bird should not add outlines, highlights, or labels except the keyboard focus ring.
- **No prompt for autoplay.** Why: a "tap for sound" prompt would announce; joining a chorus already in progress preserves the illusion.
- **Sound controls in accessibility settings.** Why: the top bar keeps four icons, and captions remain unaffected by mute.
- **WebAudio fallback to silence with captions on.** Why: accessibility and aliveness continue without introducing recorded-audio fallback.

### 9. Voice system

- **One voice package with `naturalist` and `system` registers.** Why: product surfaces need field-notebook prose, while identity, errors, settings, deletion, export, and visits need direct system copy.
- **Naturalist voice rules.** Why: lowercase, present tense, no "you," no exclamation marks, no trait labels, and no announcement framing keep the aviary from sounding like a status feed.
- **System voice rules.** Why: surfaces where the user engages the system as a system should state what happened and what to do plainly.
- **Voice lint and corpus review.** Why: banned vocabulary, generic lines, state lists, and unlabeled strings should fail before they ship.
- **Runtime lint and regeneration/drop.** Why: nothing unlinted is ever displayed or stored.
- **Field notebook based on happenings.** Why: notebook entries should be aviary-side observations, not observations of the user's behavior.
- **Forbidden notebook detectors for user behavior.** Why: visit counts, session times, absence, streaks, and "while you were away" would turn the notebook into engagement feedback.
- **Notebook salience and sparsity budget.** Why: entries should feel sparse and notable, not a backlog or reward stream.
- **Notebook phrase-usage memory.** Why: wording should not repeat over months.
- **Read-only permanent notebook.** Why: it is a field record, not an editable journal or social surface.
- **Screen-reader narration from snapshot and choreography state.** Why: narration describes the same living scene the renderer shows, with timing tied to actual motion.
- **Narration cadence and queue discipline.** Why: live regions should not spam; event messages replace idle messages and idle messages wait for material change.
- **Visible narration option.** Why: the same prose can be shown visually in an unobtrusive, contrast-compliant strip.

### 10. Accessibility surfaces

- **Accessibility surfaces as "the actual product."** Why: narration, reduced motion, captions, keyboard, contrast, and controls ship in v1 and must feel alive rather than degraded.
- **Bird buttons named by bird name with naturalist descriptions.** Why: assistive technology gets reachable birds without mood labels or trait values.
- **Captions from `CaptionDescriptor` of the synthesized call.** Why: captions describe the sound that actually played, not stored canned strings.
- **Caption placement, visibility limits, scrims, and size controls.** Why: call text must be readable, clamped, non-overlapping, and WCAG AA across lighting states.
- **Keyboard roving tabindex and spatial arrows.** Why: the aviary scene is reachable as one tab stop while bird focus follows identity as birds move.
- **Enter/Space, Escape, and Tab listen-in behavior.** Why: keyboard listen-in should engage deliberately and disengage predictably.
- **Shortcut remapping or disabling.** Why: shortcuts must satisfy WCAG 2.1.4 and avoid capturing text-field input.
- **Settle undo by any key.** Why: the five-second undo should be equally available from keyboard.
- **Double-ring focus indicators.** Why: focus must be visible against bright and dim scenes.
- **Contrast, forced-colors, zoom, and target-size rules.** Why: DOM copy and controls must meet AA, remain usable at 200% text size, and provide at least 44 px targets.
- **Accessibility regression matrix and disabled-user research.** Why: CI checks are not enough; the release question is whether the surface "feels alive," not just whether it passes.

### 11. Accounts, auth, and privacy

- **Magic-link tokens as 256-bit random, hashed, 15-minute, single-use links.** Why: sign-in links are security credentials and must expire and consume atomically.
- **Sign-up as sign-in with genesis on first verification.** Why: the first successful verification creates the account, aviary, and starter-naming flow in one path.
- **Identical auth-link response whether account exists or not.** Why: sign-in lookup should not expose account existence.
- **Rate limits by email blind index and IP prefix.** Why: abuse is limited without storing or exposing raw email.
- **POST-to-consume magic links and in-app webview guidance.** Why: scanners should not consume links, and users can move from email webviews to their browser.
- **Transactional email provider with authentication and secondary provider.** Why: deliverability and failover matter for magic links.
- **Revocable per-device sessions.** Why: users can end a device session immediately, with cache eviction and short TTL backing it.
- **Session token rotation on sensitive actions.** Why: email change, deletion, and restore should not reuse the same privilege token.
- **Email change verified at the new address before commit.** Why: the old email keeps working until the new email is proven reachable, and the old address gets a security notice.
- **Synthetic account id and PII containment.** Why: account references in sim_db, logs, metrics, queues, cache keys, and errors should not be email or location.
- **Logger allowlist, PII lint, no body logging, and scrubbing bird names.** Why: email, names, and request bodies should not leak into logs or error reports.
- **Account export by emailed link.** Why: the export exists for data portability, includes current personality vectors because the plan treats that as specific, and still is never rendered as a product surface.
- **Soft deletion for 30 days.** Why: users can restore, visits are suspended, and the aviary keeps ticking so restoration preserves aliveness.
- **Hard delete with crypto-shredding and deletion ledger.** Why: operational stores and restored backups should not retain account data after the deletion window.
- **No ETL, warehouse replica, ML training, or aggregate dashboards over sim_db.** Why: per-account interaction and simulation state is only for that account's aviary and must not become analytics.
- **Plain-text privacy policy link.** Why: settings should name aggregate telemetry categories and explicitly exclude per-bird interaction state.
- **Security baseline: CSP, Trusted Types, HSTS, Permissions-Policy, cookie hardening, dependency scanning, pentest.** Why: auth flows, visitor scoping, and IDOR on bird and notebook endpoints are expected risk areas.

### 12. Visits

- **Visit invites in account settings, off by default and absent from onboarding.** Why: visits should be deliberate, read-only, and ambient rather than a growth or social prompt.
- **One email per invite with one link and no marketing.** Why: visit email is transactional and should not become re-engagement or promotion.
- **Visitor pass cookie scoped to `/visit/{public_id}`.** Why: visitor authorization is separate from account sessions and constrained to visit routes.
- **Visitor mode with no presence module, events, listen-in, offers, settle, notebook, or host settings.** Why: visitors should see the host's canonical ambient aviary without affecting it or entering co-presence.
- **Visitor heartbeat written only to auth_db.** Why: visit duration exists for the host log but never touches sim_db and is never read by the tick.
- **Revocation returning 410 and quiet field.** Why: access stops immediately while the visitor gets the same plain "no longer available" surface for revoked, expired, or unavailable visits.
- **Visitor-route authorization tests.** Why: read-only visitor passes should be structurally unable to reach non-visit routes.
- **No visitor identity in the host scene.** Why: there should be no cursor, avatar, marker, overlay, or view change that turns visits into co-presence.
- **No account matching for visitor emails.** Why: there should be no friend graph, suggestions, or "people you may know."
- **No public directory, featuring, ratings, or visit counts.** Why: "Most visited" is not computable because visit logs stay per-host and unaggregated.
- **Visitor snapshot built by removing fields, not embellishing.** Why: there is no "show-off" rendering path.
- **Host visit log with rounded duration and no badges or counts.** Why: the host can manage access without turning visits into announcements or metrics.
- **Visit notification toggle off by default.** Why: it is the only aviary-related email that can ever be sent, and only when explicitly opted in.

### 13. Performance budgets and observability

- **Initial JS internal target far below the 2 MB cap.** Why: the plan wants first paint and first bird to rely on a small inline boot, tiny scene core, deferred audio/voice, and lazy panels.
- **First bird under 500 ms.** Why: the core experience depends on seeing a living bird quickly, with cold first visits tracked separately and warm repeat visits prioritized.
- **60 fps idle motion on reference laptops.** Why: the aviary should remain comfortably alive for a full 30-minute session.
- **No memory growth over 30 minutes.** Why: long ambient watching, notebook scrolling, panels, offers, audio, weather, and visibility cycles should not leak.
- **Adaptive quality controller preserving bird motion quality.** Why: if performance slips, DPR, particles, sway, and rain reduce before bird motion quality.
- **Real 30-minute nightly memory CI job.** Why: the PRD's memory rule needs a real session with seven birds and common interactions, not just unit tests.
- **Aggregate RUM with no ids or joins.** Why: operational signals can be measured without per-account, per-bird, session replay, heatmaps, or fingerprinting.
- **Synthetic monitoring with dedicated accounts.** Why: availability, sign-in, visitor view, first-bird, and bundle size can be checked without production user telemetry.
- **Deliberately not measuring DAU, retention, funnels, visit frequency, and engagement A/B tests.** Why: the plan says those would create "the data product" and pressure toward engagement features.
- **Metric registry reviewed by privacy owner.** Why: CI can reject account, aviary, bird, session, device, or email identifiers in aggregate pipelines before they ship.
- **Regional read-cache lever for distant users.** Why: first-bird latency can improve in some regions while writes remain in the single primary region.

### 14. Quality strategy and guardrails

- **Engine property tests and golden replays.** Why: monotonic traits, `dt` invariance, duplicate no-ops, device-order invariance, newcomer invariance, and absence-independent mood must hold across arbitrary sequences.
- **Real-Postgres concurrency and chaos tests.** Why: multi-device presence, listen-ins, offers, duplicates, out-of-order delivery, killed workers, cache outages, and clock skew must still produce the single-writer reference result.
- **Presence truth-table suite and hardware suspend check.** Why: edge cases like visible-but-unfocused windows, overnight laptops, background tabs, and suspend/resume should credit exactly zero when the rule says so.
- **Rendering and audio regression tests.** Why: first-frame, reduced-motion, seven-bird, weather, layout containment, recognizability, loudness, and clipping protect the sensory promise.
- **Voice lint, copy registry, banned APIs, and PR checklist.** Why: announcement creep, trait-number surfaces, engagement metrics, scene chrome, missing reduced-motion variants, and wrong register are expected regressions.
- **Security and privacy tests.** Why: auth flows, log fields, metric labels, and analytics network isolation need automated proof.

### 15. Rollout

- **Staging with time acceleration.** Why: 90-day newcomers, seven-bird aviaries, and months of drift can be seen before any real user gets there.
- **Internal alpha diary study.** Why: staff should test whether it "feels alive" and whether it "ever announce[s]," alongside screen-reader, reduced-motion, and personal-device dogfooding.
- **Closed beta consenting research cohort.** Why: interviews and self-reports are the only real-user calibration evidence the plan allows; telemetry is not used for drift calibration.
- **General availability with capacity guard and quiet launch.** Why: sign-up throttling protects tick headroom and avoids growth-hacking mechanics.
- **Visits shipping dark in alpha.** Why: the feature flag can make invite UI available later while visits remain off by default per account.
- **Notebook worker from alpha.** Why: sparsity constants can be tuned from diary study.
- **Kill switches including drift integration pause.** Why: server flags allow recovery without deploy, and pausing drift is safe because it only delays forward drift.
- **Bird-count ramp from two to seven.** Why: no real aviary gets a third bird before day 90, and each cap increase requires performance, memory, and recognizability checks.
- **Rollback only lowering `max_birds` for new adoptions.** Why: existing birds are never removed or hidden because of identity continuity.
- **Engine-version cohort rollout.** Why: new drift or mood constants can be tested by aviary-hash cohorts, and rate increases are cautious because accrued traits are not reversed.
- **Day-one instrumentation before alpha.** Why: first-bird, frame health, audio, tick, sync, auth, jobs, and alerts must exist before users.
- **Runbooks.** Why: tick backlog, cache loss, failover, magic-link failures, audio regressions, drift anomalies, and telemetry leaks all have expected operational paths.

### 16. Team, workstreams, milestones, and risks

- **Workstream staffing.** NOT RECOVERABLE FROM PLAN
- **M0 starting call grammar, recognizability, and calibration harness.** Why: the plan calls these the critical path with the most unknowns and says they gate the core promise.
- **M1 vertical slice.** Why: it proves the path from sign-in, genesis, tick, presence, drift, timeline, snapshot, first frame, worklet calls, greeting, top bar, and first-bird measurement.
- **M2 feature complete.** Why: it gathers the full v1 surface before hardening: six species, interactions, weather, notebook, accessibility, account flows, visits, newcomer shadow, SW, and unsupported page.
- **M3 hardening.** Why: launch gates, pentest, staging-acceleration runs, runbooks, and internal alpha must converge before beta.
- **Beta research readouts.** Why: drift constants are confirmed or adjusted from research-cohort evidence, only slower or through cautious speed-up.
- **Post-GA max-birds ramp.** Why: the earliest beta cohort reaching day 90 starts the gradual bird-count rollout.
- **Drift miscalibration mitigation.** Why: too fast becomes Tamagotchi and irreversible; too slow becomes a screensaver.
- **Presence inflation mitigation.** Why: a laxer definition, double counting, or jiggler tools would corrupt drift.
- **Lost personality mitigation.** Why: the plan names lost or corrupted personality as "the worst failure, and silent."
- **Audio uncanniness mitigation.** Why: beepy timbre, repetition, chorus mush, and loudness spikes would break the living-bird illusion.
- **Cold first-visit mitigation.** Why: DNS, TLS, and origin fetch can threaten the 500 ms first-bird budget on real 4G.
- **Autoplay mitigation.** Why: browser policies can block first-load sound, so the fallback is join-in-progress on first gesture with no prompt.
- **Accessibility regression mitigation.** Why: narration can drift into state lists, animations can miss reduced variants, and live regions can spam.
- **Gamification or announcement creep mitigation.** Why: well-meaning contributors may add "just a small toast," so lints, registry, refusals, and audits hold the line.
- **Privacy leakage mitigation.** Why: email, bird names, or per-account dimensions could sneak into logs, errors, or metrics.
- **Notebook repetition mitigation.** Why: months of entries can become repetitive or generic without phrase banks, `phrase_usage`, sparsity, and long-horizon review.
