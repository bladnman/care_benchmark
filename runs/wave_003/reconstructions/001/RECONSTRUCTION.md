## System-level intent

- **Affective requirements are hard requirements.** The plan opens by saying the PRD is "unusually opinionated about feel" and that a normal feature decomposition could "pass every acceptance test and still ship the wrong product." This intent shows up again in the "teeth" list, the a11y acceptance question "did it feel like a place?", and the launch gates.

- **This is a simulation product with a rendering client.** The plan's central framing is "a simulation product with a rendering client, not a web app with animations." It repeats this through the server-owned world, the continuously-running tick, snapshots as projections, and the browser as a viewport.

- **The world should be independent of the viewer.** The plan says ambiguous decisions should make "the world more independent of the viewer." Server ticks continue without a connected client; weather happens even when unseen; sleeping-account optimization must be invisible; clients render from snapshots rather than owning state.

- **Mechanical enforcement matters more than intentions.** The plan says anything defended only by prose "will be violated within two quarters." It turns affective invariants into lint rules, CI checks, DB grants, contract tests, visual regression tests, design-review gates, and network boundaries.

- **The product must avoid optimization surfaces.** The plan repeatedly treats scores, streaks, counters, calendars, trait numbers, timers, and exportable vectors as surfaces that invite grinding. It uses phrases like "anti-grind term," "optimization surface," "countdown UI," and "a stats panel with extra steps."

- **Privacy is an architectural boundary, not a policy.** The plan makes this explicit with "I9 is an architectural fact, not a policy." It appears in email stored once, synthetic UUIDs, encrypted visitor emails, event retention, separate telemetry VPC, no account-ID metric API, and no route to `sim-db`.

- **Clients write events, not state.** The sync model says "clients write events, not state," and "there is no endpoint that writes personality state." This supports monotonic drift, avoids last-write-wins, and keeps personality mutation inside the tick.

- **Decision on the server, realization on the client.** The plan names this as a core architectural idea: "decision on the server, realization on the client, seeded for determinism." It appears in greetings, calls, motion schedules, waveform synthesis, and multi-device consistency.

- **Accessibility is the same world through a different surface.** The plan insists narration, captions, audio, and rendering all consume `SceneModel`, so accessibility cannot drift from the visual world. Reduced motion is a "designed alternate rendering," not "animations off."

- **Variation must be procedural but identity must remain recognizable.** The plan rejects canned variants, keyframe clips, and recorded audio. It protects recognizable species timbre, permanent per-bird voice offsets, stable bird IDs, seeded procedural motion, and "never identical twice."

- **Let the user notice; do not announce.** The plan bans welcome surfaces, toasts, banners, badges, unread counts, milestone language, and visit highlights. It asks design review to distinguish "does this surface announce, or does it let the user notice?"

- **The aviary should feel like a place, not a page.** The first frame already contains birds mid-action; no spinner, entry animation, or fade-from-static is allowed. The client keeps breathing, scanning, and preening through late snapshots because "a hitch in the network must not become a hitch in the aviary."

- **Voice is disciplined by surface.** Notebook, narration, and captions use lowercase naturalist prose, present tense, and observational language. Errors, settings, and shortcuts use matter-of-fact copy. The plan treats voice as enforceable through corpus linting.

- **Performance budgets serve the affective claim.** Time-to-first-bird, 60fps, zero memory growth, no audio binaries, small bundles, and snapshot inlining are all tied to the user seeing a living aviary immediately and continuously, not to generic performance pride.

## Per-feature whys

### 0. How to read this plan

- **Implementation-ready plan with recorded judgment calls:** The plan records calls and reasoning so the team "doesn't relitigate them" while executing v1 without further clarification.

- **Affective invariants bound to enforcement:** The plan binds affective requirements to "a lint rule, a CI test, a code-review gate" because prose-only intent will erode.

### 1. Scope

#### Accounts and identity

- **Email + magic-link sign-in:** NOT RECOVERABLE FROM PLAN

- **No passwords, no SSO:** NOT RECOVERABLE FROM PLAN

- **15-minute link expiry, single-use consumption, per-email rate limiting:** NOT RECOVERABLE FROM PLAN

- **Synthetic UUID account identifier and email encrypted at rest, stored exactly once:** The plan uses this as the schema-level expression of privacy: email appears in exactly one encrypted column, while every other reference uses the synthetic UUID.

- **Per-device revocable session tokens and session list:** The session list needs to help a user recognize an unfamiliar device, but `device_label` and country-level region avoid turning it into a fingerprint archive.

- **Email change with verify-before-commit:** NOT RECOVERABLE FROM PLAN

- **On-demand JSON account export via emailed download link:** The plan says the relationship is the user's and they can take a copy, but it avoids trait numbers because a JSON export can become "a stats panel with extra steps."

- **Soft delete followed by hard delete:** NOT RECOVERABLE FROM PLAN

#### The aviary

- **One canonical aviary per account:** The plan's sync model wants "one canonical record" in `sim-db`, with no client-authoritative copy and no reconciliation code.

- **Two starter birds at adoption:** NOT RECOVERABLE FROM PLAN

- **Hard cap of seven birds:** The plan ties the cap to audio recognizability: seven birds must stay "individually distinguishable by ear," and if recognizability degrades, the cap is lowered in config.

- **Server-side simulation tick running whether or not a client is connected:** The plan says the PRD's central claim is that the aviary continues without the viewer; a routine API rollout skipping ticks would falsify that claim.

- **Six-species pool:** Species provide distinct silhouettes, palettes, motif libraries, and recognizable timbral fingerprints; the nightjar-like species is called out as required by the night-activity rule.

- **Stable per-bird internal identity:** Stable `bird_id` protects identity across rename, sync, species-pool changes, and migrations; deleting, resetting, or swapping birds is treated as an identity violation.

- **Age-gated new-bird offers:** The gate uses aviary age rather than visit count, interaction volume, or payment so new birds are not tied to engagement or grind behavior. Declined offers return only after a long interval so two birds forever remains valid.

#### Interactions

- **Return-greeting:** The greeting lets one bird notice the user, but the plan rejects canned variants. The server decides which bird and when because the greeting feeds notebook, drift, and narration; the client realizes motion and waveform from a seed.

- **Listen-in:** Listen-in is a rebalance, not a mute or solo track. Slow ramps and a nonzero floor make it feel like "leaning in" while the rest of the aviary remains ambient.

- **Offer from the top bar only:** Offers stay out of the aviary scene because the plan wants no chrome inside the scene. Cooldowns are hidden as quietly unavailable options because timers would create a countdown UI.

- **Settle with soft evening lighting and 5-second undo:** The plan adds `settled` because full night and the settle gesture need a distinct terminal-ish state. Undo is "a mercy, not a feature," so announcing it would violate the no-announcement stance.

- **Presence accounting from visibility, focus, and recent input:** The plan says watching without moving is the actual product, so the activity window is long enough for genuine stillness but short enough to exclude an abandoned desk. Server validation prevents clients from crediting fake presence.

- **Field notebook:** The notebook is a sparse, immutable "record of what was observed." It must observe the aviary, not the user, and it avoids milestones, counts, and trait labels.

#### Social

- **Per-invite, email-addressed, opt-in visit invitations, off by default:** The plan frames social as "one feature, deliberately," keeping it away from profiles, feeds, discovery, co-presence, and social-network surfaces.

- **Read-only ambient visitor view:** A visit is observation, not co-presence. Visitors get no interaction, no event endpoint, no greeting, no presence contribution, and zero drift input.

- **Immediate revocation:** Revocation is checked on each visitor snapshot pull so a revoked visit terminates within one poll interval, which the plan treats as immediate at human timescale.

- **30-day invite expiry, non-revivable:** NOT RECOVERABLE FROM PLAN

- **Silent visit logging, settings-only visit log, optional notification toggle off by default:** The plan keeps visits out of badges, unread counts, highlights, and announcement surfaces. The log is reachable by navigating to settings.

#### Accessibility

- **Screen-reader narration:** Narration is a first-class surface that reads `SceneModel`, uses naturalist prose, and describes appearance rather than exposing mood labels, so a screen-reader user experiences the same world by behavior.

- **Reduced-motion mode:** Reduced motion is a second renderer with designed pose sets and unchanged calls, drift, mood, notebook, and greeting. The reason is that "animations off" or frozen procedural frames would look broken instead of charming.

- **Runtime-generated call captions:** Captions are generated from the actual synthesis grammar so they match what played. They are allowed inside the aviary scene because they transcribe the aviary's own sound rather than adding chrome.

- **Full keyboard navigation and focus indicators:** Keyboard users need to explore birds, enter listen-in, and use dialogs without losing visible focus. The two-tone indicator exists because the scene can be bright or dim.

- **WCAG AA contrast on user copy:** The aviary palette changes all day, so contrast, especially for captions, needs runtime or per-frame checking rather than design-time assumptions alone.

#### Performance and observability

- **Initial JS budget and time-to-first-bird budget:** The plan treats these as necessary for a first frame that already feels alive. The internal targets are below the stated caps because a budget near 95 percent will be breached.

- **60fps idle motion and zero memory growth:** Continuous motion over a 30-minute session must not turn into dropped frames or leaks; otherwise the living-place illusion breaks.

- **Client-side WebAudio procedural synthesis with no audio files:** Procedural synthesis gives variation, avoids recorded audio, and keeps audio parameterized by species, bird, mood, and drift.

- **Browser support for last two major Chrome, Safari, Firefox, and Edge:** NOT RECOVERABLE FROM PLAN

- **Aggregate-only operational telemetry:** Telemetry measures system health, not per-account or per-bird behavior, because a retention or engagement dashboard would invite product changes that move those numbers.

- **Synthetic performance fleet and tick p99 alarm at 5s:** Tick lag is the one outage users can eventually feel because the world pauses; the alarm is aggressive for that reason.

#### Explicitly out of scope

- **Native mobile apps:** The plan says not to shape the data model or protocols around a future native client.

- **Gamification surfaces:** Achievements, streaks, levels, scores, badges, counters, calendars, XP, ranks, tiers, and milestone celebrations are excluded because they create optimization and engagement surfaces.

- **Tamagotchi mechanics:** Death, hunger, distress, decaying happiness, and negative drift are excluded because neglect is modeled as absence of input, not punishment.

- **Social-network surfaces:** Profiles, follows, feeds, discovery, friend-of-friend, comments, chat, avatars, and leaderboards are excluded because the plan is deliberately not building a social network.

- **Push notifications and marketing email about the aviary:** The plan avoids building notification capability beyond transactional messages because notification surfaces would push the product toward engagement mechanics.

- **Payments:** NOT RECOVERABLE FROM PLAN

- **Shared or multi-aviary accounts:** NOT RECOVERABLE FROM PLAN

- **Customizable scenes:** NOT RECOVERABLE FROM PLAN

- **Panning, scrolling, or zooming the aviary:** The plan keeps the aviary as a fixed, always-legible scene with no route for birds to be cropped or turned into a navigable app surface.

- **UI exposing personality vector numbers:** The plan says trait values must never cross the API boundary because numbers become an optimization surface and violate the "never visible in any tier" rule.

### 2. Architecture

- **Five deployable units:** The split is driven by privacy boundaries and the tick's different runtime profile, "not by microservice fashion."

- **Separate `sim-tick` deployable:** The tick must keep running during API deploys, incidents, and scale-downs; it also scales with account count rather than request rate.

- **Separate append-only `event-log`:** The write path is high-frequency, append-only, and needs different retention, so it should not lock or bloat the canonical state store.

- **Telemetry in a separate VPC:** The plan makes I9 architectural: future per-bird analytics would require a networking change, creating intentional friction.

- **Edge tier with inlined bootstrap snapshot:** This removes a round trip from the critical path and is called the "single highest-leverage decision" for the 500ms budget.

- **Client stack with no UI framework in the critical path and Preact for chrome:** The bird scene needs 60fps for articulated entities; React in the critical path would spend bytes on two buttons and tempt the team into virtual-DOM birds.

- **Go for `aviary-api` and `sim-tick`:** The plan chooses Go for predictable latency, goroutine fan-out, and p99 control under the 5s tick alarm.

- **Postgres plus Redis:** Postgres fits small relational state and strong event ordering; Redis fits scratch presence windows, cooldowns, nonces, and rate limits.

- **Transactional email provider only:** The provider is deliberately incapable of marketing campaigns, because the cheapest way to avoid notification creep is not to build the capability.

- **Server-owned persistent state and client-owned frame realization:** The rule is "if two devices are open at once, must they agree?" If yes, the server owns it; if no, the client can realize it locally.

- **`SceneModel` as the single source for renderer, audio, narration, and captions:** This prevents accessibility skew because narration and captions are generated from the same model as the pixels.

### 3. Domain model

- **Interaction events as the only client-produced simulation input:** Events are immutable and append-only, which lets the tick fold inputs instead of accepting client state writes.

- **Snapshots with a scheduled future horizon:** A snapshot projects current aviary state plus future calls and motions so the client can render smoothly ahead.

- **No score, level, points, progress, or streak in the object graph:** The plan says any PR introducing those names should be rejected because they contradict the product stance.

### 4. Data model

- **HMAC email hash next to encrypted email:** The hash lets magic-link sign-in look up an account without decrypting or indexing plaintext; it is peppered so it is not trivially reversible.

- **No display name, avatar URL, or bio:** Their absence is the schema-level expression of "not a social network."

- **No bird delete, active, or replacement columns:** Birds are not deletable in v1, and stable identity is enforced partly by not creating the mechanism.

- **`drift_carry` for fractional drift:** Without carry, sub-epsilon deltas can round to zero for weeks and make birds silently never change.

- **No page-view, click, or feature-used events in `interaction_events`:** The event log is a simulation input, not an analytics stream; making it both would lose the privacy boundary.

- **`bigserial` event ordering:** The tick needs strict order, and monotonic integer ordering supports the no-last-write-wins sync argument.

- **Rendered immutable notebook prose:** Storing final prose keeps old observations from being retroactively changed by newer templates, protecting the user's history.

- **Derived device labels and country-level region:** The session list should be recognizable for account safety without preserving raw user-agent fingerprints.

- **Encrypted visitor emails:** A visitor who has not signed up still has PII in the system, so the same bar applies.

- **30-day retention for consumed interaction events:** The plan keeps enough for incident replay while avoiding a reconstruct-the-vector-from-history capability.

- **Export excludes personality vectors and includes naturalist paragraphs:** The plan resolves a conflict toward the stronger no-exposure rule, because diffable exports manufacture the optimization surface.

- **Small typed settings object:** Settings are where features get smuggled in; keeping the schema small makes additions visible in review.

### 5. Simulation engine

- **Discrete-time, per-account, deterministic-given-inputs engine:** Determinism makes the engine testable and supports replay, calibration, and seeded multi-device realization.

- **Partitioned tick with exactly-once per account per window:** Optimistic concurrency drops losing writes rather than retrying into a second drift application, because double-applied drift is a silent correctness bug.

- **Elapsed-time-aware catch-up instead of replaying skipped ticks:** Integrating the gap is cheaper and avoids non-linear mood artifacts while matching "the world kept running."

- **Sleeping-account lower cadence:** Accounts with no events can tick at 1/10th cadence as a pure cost optimization, but reconnect forces an immediate tick so the user cannot observe the optimization.

- **Ordered tick pipeline with pure functions:** The plan makes steps 4-11 pure functions of previous state, folded inputs, wall clock, and seed so engine behavior can be property-tested.

- **Monotone, saturating, low-pass drift:** Saturation keeps long-term birds from all pinning to 1.0 and becoming indistinguishable, while monotonicity makes neglect an absence of input rather than a penalty.

- **Novelty factor on interaction signals:** Novelty is the anti-grind term; the 20th listen-in in one session contributes far less than the first so clicking does not work as a strategy.

- **Presence as dominant drift input and plumage tied mostly to presence:** An hour of watching should outweigh a burst of clicking, and the most visible long-term change should be the one the user cannot grind for.

- **Daily presence cap at 4 hours:** A second-monitor user can genuinely be present, but should not drift far faster than an attentive one-hour user; the cap is above ordinary sessions.

- **Weighted stochastic mood transitions:** Sampling rather than thresholding is the mechanism behind "never identical twice" at the mood layer.

- **Added `settled` mood:** Full night and the settle gesture both need a distinct state with birds low on the perch and eyes closed.

- **Mood inertia and minimum dwell time:** Without dwell time, birds would flicker between moods and read as glitchy rather than alive.

- **Ambient mood contagion and weather effects:** Wary contagion, rain, and wind use shared scoring terms so flock behavior emerges without special-casing each bird.

- **Long-absence behavior as expression decay, not trait decay:** After absence, birds greet and call less readily, but vectors do not go down; they warm back up within a session or two.

- **Perch assignment from boldness, mood, social warmth, and hysteresis:** Hysteresis prevents rapid zone switching, keeps movement readable, and avoids overlap.

- **No API to set perch:** Perches are simulation state, not user-controlled state; the only bird patch is name.

- **Weather as wall-clock events with no forecast surface:** Weather can happen unseen, which the plan says is correct because the world runs without the viewer. No icon or severity surface is added.

- **Call grammar split between scheduling and synthesis:** The server decides call timing, bird, and motif structure; the client synthesizes waveform so calls remain deterministic but cheap and varied.

- **Species and individual voice identity:** Species timbre and motif sets never drift, and per-bird `voice_offset` is permanent, so drift cannot erode the ability to recognize Pip by ear.

- **Chorus emergence with a small nudge:** Pure independent scheduling makes chorus too rare to read as social, so close calls become chorus and a high-vocal bird may be pulled forward.

- **Seeded call determinism:** The same seed produces the same waveform on two devices, so laptop and phone hear the same aviary.

- **Notebook sparsity budget:** At most one entry per day and three per week keeps the notebook sparse and prevents repetition from revealing the template machinery.

- **No runtime LLM for notebook prose:** The plan chooses a hand-written structured corpus for determinism, zero latency, no hallucination, no voice drift, and no per-bird-state egress.

- **Notebook detectors observe only the aviary:** The detector interface excludes session counts, visit frequency, and streaks so observations remain about the aviary, never the user.

- **Event validation and unioned concurrent presence:** Clients are not trusted with presence, and two attended devices count as one hour rather than doubling drift by accident.

- **Calibration targets and harness:** The plan operationalizes week-one measurable drift and week-three visible drift, then requires a 1000x synthetic runner so constants are tuned before production cohorts reveal mistakes.

### 6. API surface

- **Auth request always returns 202:** This prevents account-existence oracle behavior and email enumeration.

- **Snapshot `expressiveness` and `plumage_step` instead of traits:** The client needs some render signal, but coarse buckets and quantized steps are meant to be non-reversible and satisfy the no-trait-boundary rule.

- **Polling with jitter and schedule horizon instead of WebSockets:** One small message per minute does not justify a stateful connection tier, and interpolation plus a 90s horizon gives smooth rendering.

- **Batched, idempotent, offline-queued event writes:** Batching and idempotency let clients flush safely across pagehide, reconnect, and duplicate sends without turning events into state writes.

- **No personality mutation endpoint or admin shortcut:** The plan implements the tick-only writer rule with API absence, Go package boundaries, and DB grants because personality-write failures are silent.

- **Offer and settle return 202 with optimistic local realization:** Waiting for a round trip before anything happens would read as lag in a product whose claim is immediate response.

- **Visitor mode without event queue and without greeting:** Visitor code omits the event module and greeting because visits are observation, not co-presence or host-specific drift input.

- **Central matter-of-fact error catalog:** Error copy stays reviewable and deliberately avoids naturalist phrasing, separating system messages from aviary voice.

### 7. Frontend rendering pipeline

- **WebGL2 with Canvas2D fallback:** WebGL2 gives margin for seven articulated birds, parallax, particles, palette LUT blending, 60fps, and 30-minute no-growth; Canvas2D is only a compatibility path.

- **Layered scene composition and responsive virtual coordinates:** Fixed vertical extent, variable horizontal extent, scaling by vertical extent, and `visibleBounds` assertions protect bird legibility across viewports.

- **No panning, zooming, or scene scroll:** The aviary route stays a fixed scene rather than a navigable page or map.

- **Birds rendered with layered procedural oscillators:** Procedural breath, weight shift, scan, preen, blink, fluff, and sound tilt avoid visible animation loops and make "never identical twice" true visually.

- **Mood shown through motion, not labels:** The plan forbids mood labels, tooltips, icons, or aria hints that name mood; users infer from appearance and behavior.

- **Interpolation and schedule horizon:** Late snapshots extend the last model and fill gaps plausibly, because teleporting is visible and a network hitch should not freeze the aviary.

- **Cold load first frame with birds mid-action:** Oscillators are advanced to nonzero phases before the first draw so the scene does not visibly "start."

- **Slow-load quiet field:** If needed, the loading state is a quiet field with time-of-day sky and faint motion cues, not spinner, progress bar, skeleton, or logo.

- **Empty-aviary first-bird fly-in:** This is the only fly-in-from-nothing because the birds genuinely are arriving; other entry animation is forbidden.

- **Listen-in visual realization:** The visual treatment is a subtle focus deepening rather than highlight, border, ring, spotlight, or badge, keeping focus inside the place.

- **Offer sheet and quiet cooldown surfacing:** The sheet closes and reactions begin locally; cooldown appears as the option not being offered again, avoiding timers and disabled countdown controls.

- **Settle undo without label or announcement:** The undo click reverses the blend, but surfacing it as an announced feature would violate the no-announcement rule.

- **Reduced-motion second renderer:** Pose sets, cross-fades, removed particles/parallax, and slowed palette transitions make reduced motion a designed surface with full product quality.

- **Top bar as the only chrome:** It fades with stillness but not during keyboard use or open menus, because hidden controls during keyboard navigation are an accessibility bug.

### 8. Audio pipeline

- **Pre-allocated voice pool:** Pooling filters, gains, and panners avoids the standard WebAudio memory leak and supports the 30-minute no-growth test.

- **Audio graph lazy on first gesture and off the critical path:** Browser autoplay policy requires lazy construction, but the scene must not wait; if audio is unavailable or suspended, captions turn on.

- **Chorus mixing with shared reverb, spectral ducking, and no compressor:** These choices make birds sound like they are in one place and avoid the "mixed audio" pumping signature.

- **Listen-in mix ramps:** The focused bird rises faster than others fall, and others never go silent, making listen-in feel like leaning in rather than soloing tracks.

- **Ambient bed:** Quiet wind, day-phase tone, and rain prevent silence between calls from reading as "audio is off."

- **WebAudio fallback to silence plus captions:** There is no recorded-audio escape hatch; aggregate telemetry records availability issues without account dimensions.

### 9. Accessibility

- **Accessibility ships with v1, not after:** The plan says there is no later "a11y hardening" milestone because that phrasing is how a11y slips.

- **Screen-reader cadence, floor, and queue depth:** The narration should not overwhelm the SR queue; otherwise the designed surface becomes an annoyance users mute.

- **Mood described as appearance:** The composer maps mood to phrases like feathers fluffed rather than "drowsy," so screen-reader users infer mood the same way sighted users do.

- **Focusable bird groups for exploration:** Beyond live narration, a group per bird lets screen-reader users choose where to attend rather than only receiving periodic summaries.

- **Captions generated from grammar and placed near the caller:** They match the sound that played and remain part of the aviary rather than an external UI control.

- **Keyboard shortcuts and dialog behavior:** The plan provides a complete path through top bar, birds, listen-in, offer sheet, and escape behavior so keyboard users can operate the aviary.

- **Two-tone focus indicator:** It remains legible against both bright midday and dark night states without animation.

- **Accessibility settings limited to four toggles:** Reduced motion, captions, narration, and audio on/off are enough; the plan avoids turning settings into a feature drawer.

- **A11y acceptance asks whether it felt like a place:** The launch bar is qualitative and affective, not merely "could you use it" or axe compliance.

### 10. Performance and observability

- **Internal targets below stated caps:** The plan says a budget near its cap will breach in the next sprint, so internal targets are 400KB and 350ms.

- **Time-to-first-bird custom mark:** FCP and LCP would measure the sky gradient, so the plan measures the first frame that actually contains a bird.

- **Runtime performance rules:** One RAF loop, no hot-loop allocation, pooled particles, and pause/resume on visibility changes keep motion smooth and memory flat.

- **Deliberately not measuring DAU, retention, session frequency, or funnels:** The plan says teams with retention dashboards eventually ship streak counters; it measures health rather than whether users come back often enough.

### 11. Sync model

- **Last-write-wins made unreachable:** Since clients produce ordered event log entries rather than state writes, two devices cannot overwrite each other's personality changes.

- **Failure behavior through local rendering and queues:** If databases or logs are unavailable, clients keep rendering from the last snapshot and queue events; tick downtime is singled out as the outage users can eventually feel.

### 12. Testing and enforcement

- **Engine property tests:** Drift monotonicity, saturation, zero-input behavior, determinism, mood dwell, idempotency, ordering, calibration, and long-run simulation make the engine's affective rules testable.

- **I-list enforcement matrix:** Each non-negotiable invariant has a concrete mechanism such as DB grants, schema tests, copy lint, network policy, migration checklist, or visual regression.

- **Visual and behavioral tests:** First-frame, no-crop, loop-detection, greeting variation, and listen-in mix tests convert "alive," "never cropped," and "never identical twice" into checks.

- **Copy linting:** Naturalist and matter-of-fact surfaces have opposite lint rules, and banned lexicon enforcement is the mechanical form of voice discipline.

### 13. Rollout

- **Milestones ordered around risk and parallel work:** The calibration harness is built before tuning; renderer, audio, and accessibility overlap so a11y is not postponed.

- **At least four weeks of private beta:** The plan says drift visibility takes about three weeks, so a shorter beta would ship a drift function nobody has watched work.

- **Bird offer schedule as server config:** The slow schedule matches long-term growth, while config lets the team lower the cap if audio recognizability fails before users reach high counts.

- **Instrumentation from day one:** Metrics are defined against the exclusion list before first emission, so privacy boundaries are present from the first PR rather than retrofitted.

- **Launch gates:** The plan gates v1 on I-list tests, calibration, blind listening, a11y "felt like a place," field TTFB, memory soak, and beta drift behavior.

### 14. Judgment calls

- **Export trait exclusion sign-off:** The plan builds toward no trait numbers because that is the stronger load-bearing rule, but notes the PRD author can overrule it with a one-field change.

- **Mood set finalization:** `settled` is included because both full night and settle need a state beyond `drowsy`.

- **Polling transport:** Polling with jitter is chosen because one small message per minute does not justify WebSockets.

- **Five-minute presence activity window:** Watching without moving is the product; five minutes balances stillness with abandoned-desk exclusion.

- **Unioned concurrent presence:** Summing concurrent devices would create a grind mechanic by accident.

- **Four-hour daily presence cap:** It prevents second-monitor presence from dominating drift while staying above ordinary use.

- **Hand-written prose corpus:** Determinism, latency, privacy, and voice stability outweigh a runtime LLM's best-case prose quality.

- **Six-step plumage quantization:** Continuous saturation drift would be imperceptible, so discrete steps make the visible trait actually visible.

- **Visitor receives no greeting:** Observation is not co-presence, and greeting is host-specific plus drift-relevant.

- **Offer cooldown has no timer UI:** A timer is a counter, and counters are the thing the plan is avoiding.

- **Tick catch-up integrates gaps:** It is cheaper and avoids mood artifacts compared with replaying many skipped ticks.

- **Consumed event retention at 30 days:** This is enough for incidents but short enough to avoid building a vector-reconstruction capability.

### 15. Risks

- **Drift calibration risk:** The plan says wrong drift fails silently and costs a month per iteration, so it mitigates with the calibration harness, config constants, beta length, and aggregate time-to-first-visual-step.

- **Audio uncanniness risk:** Procedural calls have no recorded fallback, so the plan requires dedicated audio work, fresh-ears reviews, blind recognizability gates, and restraint if naturalness stalls.

- **Announcement creep risk:** Toasts and badges are likely because each individual use sounds reasonable; the mitigation is no primitives, lint, copy review, and a design-review question about notice versus announcement.

- **Sync correctness regression risk:** Future features could reintroduce client-side state writes, so I1 is enforced through DB grants, package boundaries, CI grep, ADR citation, and chaos tests.

- **Accessibility fallback risk:** Reduced motion and narration can become "usable" but not good; overlapping milestones, designer pose sets, qualitative acceptance, and `SceneModel` sharing mitigate that.

- **Performance budget erosion risk:** Settings, notebook, visit flow, and libraries can quietly consume the bundle; internal budgets, hard CI, code-splitting, and nightly device tests defend it.

- **Notebook prose staleness risk:** Repeated templates reveal the mechanism and cheapen earlier entries; corpus size floors, linting, simulation tests, and ongoing corpus expansion mitigate it.

- **Seven-bird cap risk:** The asserted cap may fail with this synthesis, so the plan tests counts 4-7, keeps the cap in config, and uses slow offers for runway.

- **Tick outage risk:** The tick is the world-running component; independent deploys, lag alarms, catch-up integration, and unconsumed-event alarms protect it.

- **Privacy boundary erosion risk:** A single analytics join would be unrecoverable, so the plan uses separate VPCs, metric API shapes, quarterly reviews, and absence of engagement dashboards.

### 16. Open items

- **PRD-author confirmation on export contents:** The plan wants confirmation because excluding trait numbers resolves a conflict; implementation proceeds with exclusion unless overruled.

- **Species names and visual direction:** The engine treats species as config, so naming and silhouette design are needed but not on the critical path.

- **Design-system spec:** Palette, contrast, focus treatment, and reduced-motion pose sets are required for M2 and M5.

- **Notebook corpus authorship:** The plan identifies this as the largest non-engineering deliverable and says it should start in week 2 so it is not discovered late.
