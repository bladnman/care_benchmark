## System-level intent

1. **Ambient aviary, not a game.** The plan repeatedly protects the product from "Gamification of any kind," "Tamagotchi mechanics," "Social network surfaces," and "Push/email notifications about the aviary." The risks call this "Gamification creep" and say it would cause "Product identity collapse."

2. **Watching without moving is the product.** The 4-minute presence window "leans long" because "watching without moving is the product." Presence time is the "dominant" drift signal, and presence pings are recorded while the visible/focused/activity conjunction holds.

3. **Change should happen over weeks, not sessions, without punishment.** The drift calibration targets "~3 weeks" for user-visible behavior change, and the risk table names the promise as "weeks not sessions." The "Monotonic rule" says neglect produces "no negative delta" and birds become ambient through "fewer greetings via mood/perch behavior, not trait punishment."

4. **Canonical state belongs on the server.** The plan centers "canonical server state," says "Clients never tick, never write personality state," and rejects "client-to-client sync," "CRDT," and "LWW on personality." Conflict prevention relies on ordered events and the rule that "Only tick writes vectors."

5. **Do not turn the relationship into optimization.** The plan bans "Exposing personality vector numerically to users" with "no debug toggle, ever." The API strips trait numbers, the client receives only "visual proxies," and the risk table says personality exposure means the "Relationship becomes optimization."

6. **Use naturalist, matter-of-fact voice instead of announcement UI.** Error copy is "matter-of-fact." Notebook output is "lowercase present-tense prose" with "never numeric traits" and no user-behavior stats. Screen-reader narration must be prose, "Never state-list," and return-greeting has "No textual welcome anywhere."

7. **Charm depends on sparsity and restraint.** The notebook is "read-only" with "sparse entries," the entry rate is a target of "1 entry / 3-5 days," and the risk table says "Notebook too chatty" would "Dilute charm." Rate limits and a salience scorer protect that restraint.

8. **Accessibility ships as part of the product.** Accessibility is in scope with "Screen-reader narration, call captions, reduced-motion mode, keyboard nav, WCAG AA." The risk table warns that "Accessibility retrofit" means "Reduced-motion users get broken product," and rollout gates "a11y complete" before later scope.

9. **Social must remain opt-in, read-only, and bounded.** Visits are "Opt-in," "read-only ambient view," and "off by default." Out of scope excludes social network surfaces, and conflict prevention says "Visitor tokens cannot POST." The visit risk is "Visit co-presence scope creep" and the mitigation is "Read-only token architecture."

10. **Aliveness is procedural, varied, and performant.** Rendering targets "60fps," "Time to first bird < 500 ms," and no spinner. Audio uses procedural motifs with runtime variation, chorus flags, detune, and QA against "Audio uncanniness" because it "Breaks aliveness."

11. **Observe aggregate health without collecting intimate state.** Account rows use a synthetic UUID, encrypted email, and an email hash "for lookup only." Observability is "aggregate only," with "Never collect" for per-bird state, per-account interaction content, or personality values, and "Deliberately not measured" includes engagement scores and streak proxies.

## Per-feature whys

### Scope and v1 deliverables

- **Modern web browsers only**: NOT RECOVERABLE FROM PLAN
- **Single-user, email magic-link auth, one aviary per account**: NOT RECOVERABLE FROM PLAN
- **Two starter birds**: NOT RECOVERABLE FROM PLAN
- **Bird cap at 7**: The rollout says to "Monitor audio mix quality at 5-7 birds in beta before raising cap visibility."
- **Roughly 6 species pool**: NOT RECOVERABLE FROM PLAN
- **Age-gated adoption of additional birds**: The "Third bird unlock" rationale says "Few months" pacing starts earlier for v1 testing and should be tuned in beta.
- **Server-side tick at about once per minute**: The tick cadence decision cites the PRD phrase "~once per minute"; keeping ticks server-side also supports canonical state and ordered event processing.
- **Personality drift**: Drift exists so regular use produces detectable movement and, after about 3 weeks, "user-visible behavior change" such as bolder perch choice, richer plumage, and more frequent calls.
- **Mood system**: Mood persists across sessions and offline ticks so the start mood next session can be modulated by overnight, time of day, weather, recent events, and neighbor mood.
- **Bird-to-bird interaction**: NOT RECOVERABLE FROM PLAN
- **Return-greeting**: The greeting is server-authored from absence, mood, boldness, and socialWarmth, and the plan insists on "No textual welcome anywhere" and "never identical twice."
- **Presence accounting**: The 4-minute window exists because "watching without moving is the product," and presence_time is the dominant drift signal.
- **Listen-in**: The mix focuses a bird while keeping "others -6dB" and "never silent," preserving the ambient chorus while allowing attention.
- **Offer interaction**: Offer signals feed curiosity and boldness, while the 3-minute per-bird cooldown "prevents drift saturation."
- **Settle interaction**: The drift table says settle is "mood quieting only; neutral drift," and rendering/audio transitions make it a lighting and volume change rather than a trait reward.
- **Read-only sparse notebook**: The rationale is the "PRD sparsity requirement," avoiding numeric traits and user-behavior stats so entries stay naturalist rather than evaluative.
- **Multi-device sync through canonical server state**: The plan rejects client-to-client sync, CRDT, and LWW on personality to avoid overwrite and preserve ordered tick updates.
- **Opt-in visit invitations**: Visits are read-only, off by default, and token-scoped so a visitor cannot affect the host or force "co-presence scope creep."
- **Accessibility surfaces**: The plan treats accessibility as in-scope and gates it before later rollout because retrofit would leave reduced-motion users with a "broken product."
- **Client-side procedural WebAudio synthesis**: The plan bans a recorded-audio fallback, avoids looped samples, reuses buffers for the performance budget, and uses variation to prevent identical loops.
- **Single horizontal scene with day/night, weather, and no in-scene chrome**: NOT RECOVERABLE FROM PLAN

### Defensible calls on ambiguity

- **Presence activity window of 4 minutes**: The rationale is explicit: "Leans long per PRD; watching without moving is the product."
- **Personality trait range as normalized floats**: The rationale is "Standard; seed values species-keyed."
- **Offer cooldown of 3 minutes per bird**: The rationale is "Few minutes" and "prevents drift saturation."
- **Notebook entry rate of 1 entry every 3-5 days with bursts**: The rationale is the "PRD sparsity requirement."
- **Third bird unlock at aviary age >= 21 days**: The rationale is that "Few months" pacing starts earlier for v1 testing and can be tuned in beta.
- **TypeScript monorepo stack**: The rationale is "Web-first, team velocity" and that the stack was "not prescribed by PRD."

### Architecture and render boundary

- **CDN / Edge with edge-cached initial snapshot stub**: The feature supports the first-paint path where a first bird should appear within 500ms, even from a stub or optimistic default pose.
- **API Gateway / BFF with auth, snapshots, events, notebook, visits, settings**: NOT RECOVERABLE FROM PLAN
- **Auth Service, Simulation Service, Notebook Gen split**: NOT RECOVERABLE FROM PLAN
- **Client/server split for canonical state versus rendering/audio/presence detection**: The server owns canonical state and personality writes; the client handles interpolation, procedural motion, audio, and event emission without mutating trait truth.
- **Hard rule that clients never tick or send absolute trait values**: This prevents client-side personality writes and supports the principle that trait numbers are never exposed.
- **Snapshot fetch on load, resume, keepalive, and long frame gaps**: The rationale is to keep the client aligned with canonical state after visibility changes or stale rendering gaps.
- **`tickVersion` in snapshots**: It lets clients discard older snapshots and avoid stale state.
- **SimulationInterpolator with last two snapshots**: It renders smoothly at 60fps between canonical snapshots.
- **MotionDirector idle micro-motion**: It layers mood-shaped procedural motion on the base pose so behavior can reflect mood without changing canonical state every frame.
- **AmbientOrnamentRenderer for non-canonical leaf/feather drift**: NOT RECOVERABLE FROM PLAN
- **Stopping `requestAnimationFrame` when the tab is hidden**: It pauses hidden rendering while presence ends and the server continues ticking.

### Data model

- **Synthetic account UUID that is never email**: The inline note "never email" and encrypted email/hash lookup keep identity separate from account IDs.
- **Encrypted email and email hash for lookup only**: The hash exists "for lookup only," and the encrypted email protects account data.
- **Sessions with device revocation**: NOT RECOVERABLE FROM PLAN
- **One aviary per account in v1**: NOT RECOVERABLE FROM PLAN
- **Aviary `created_at` driving bird-unlock age**: The data model ties adoption pacing to aviary age.
- **Aviary `local_tz` from first client load**: It supports local day/night, dusk mood changes, and time-of-day rendering.
- **Stable bird IDs never regenerated**: NOT RECOVERABLE FROM PLAN
- **Personality JSONB on birds**: The plan needs persistent traits so server ticks can apply additive drift and so visual behavior can change over weeks.
- **Personality updated only by tick worker via additive deltas**: This invariant prevents client mutation and supports monotonic drift.
- **Append-only interaction_events**: Tick input is ordered by server time, enabling processed events to become the source for drift, mood, and notebook entries.
- **Notebook source_event_ids**: Entries can be tied back to noteworthy events that triggered generation.
- **Visit invitation token_hash, expiry, and revocation**: The feature supports read-only, revocable, time-bounded visit access.
- **Personality vector hidden from numeric API exposure**: The client receives only visual proxies because numeric trait exposure would make the relationship an optimization surface.
- **Mood enum persisted across sessions**: The rationale is continuity; mood at session end is the next start mood, modulated by offline ticks.
- **Presence pings only under visible/focused/recent activity conjunction**: The strict three-way AND protects the drift signal from being too loose or too tight.
- **No presence recorded for visitors**: This prevents visitors from affecting host state.

### API surface

- **Magic-link auth endpoints**: POST is rate-limited, verify links expire after 15 minutes, and duplicate use is prevented by single-use tokens.
- **Session list and revoke endpoints**: NOT RECOVERABLE FROM PLAN
- **`GET /aviary/snapshot`**: The endpoint provides current canonical state and `tickVersion` for client rendering and stale-snapshot rejection.
- **`POST /aviary/events` with batches up to 10**: NOT RECOVERABLE FROM PLAN
- **`POST /aviary/timezone`**: The endpoint lets the app set local TZ for day/night and time-of-day behavior.
- **`POST /birds/adopt-starters`**: NOT RECOVERABLE FROM PLAN
- **Bird rename endpoint**: NOT RECOVERABLE FROM PLAN
- **Age-gated `adopt-next` endpoint**: The 403 behavior enforces age and cap pacing.
- **Notebook pagination**: NOT RECOVERABLE FROM PLAN
- **Account export and soft delete**: Export gives JSON account data, while soft delete allows 30-day recovery before hard delete cascades all bird data.
- **Visit invite, list, revoke, log, and view endpoints**: These support opt-in read-only visits, host control, and a visitor snapshot that omits notebook write paths.
- **Error body `{ code, message, action? }` with matter-of-fact copy**: The error voice follows the PRD's matter-of-fact copy requirement.

### Simulation engine

- **Active aviaries tick every 60s while cold aviaries tick every 5 min**: NOT RECOVERABLE FROM PLAN
- **Tick worker processing unprocessed events into birds, notebook entries, and processed marks**: The rationale is canonical server mutation from ordered events.
- **Presence_time as the dominant drift signal**: The plan explicitly weights presence_time at 1.0 and says it affects all traits, strongest for plumageSaturation and socialWarmth.
- **Listen-in drift signal**: Listen-in duration affects that bird's socialWarmth and vocalFrequency, connecting attentive listening to behavior and calls.
- **Offer drift signals**: Offer acceptance targets curiosity and offer_near_bird targets boldness.
- **Monotonic drift rule**: Deltas are `max(0, computed_delta)` so neglect never produces negative trait punishment.
- **Weekly and three-week drift calibration**: Calibration exists to make regular use measurable weekly and user-visible after about three weeks without UI stating it.
- **Mood transitions from events, local time, weather, neighbor mood, and personality**: These inputs let mood express context such as dusk, rain, wary neighbors, and boldness.
- **Server-side call-grammar hints instead of server audio synthesis**: NOT RECOVERABLE FROM PLAN
- **Chorus event flag when call windows overlap**: NOT RECOVERABLE FROM PLAN
- **Return-greeting greeter selection by `boldness * socialWarmth`**: The plan makes greeting feel authored by bird traits and mood rather than by an announcement banner.
- **Bird-to-bird call response and mood contagion**: NOT RECOVERABLE FROM PLAN
- **Notebook generation on noteworthy events**: The trigger list, lowercase present-tense prose, rate limiter, and ban on user stats preserve sparse naturalist voice.

### Sync model

- **Event log to tick to canonical DB flow**: This prevents dual-device personality overwrite by making events append-only and tick-authored.
- **No client-to-client sync, no CRDT, no LWW on personality**: The rationale is conflict prevention around personality state.
- **Idempotency key per event POST**: It prevents event replay.
- **Single-use magic-link token**: It prevents duplicate magic-link use.
- **Visitor tokens cannot POST events**: It prevents a visitor from affecting the host.
- **Offline resume with 500ms smooth snap**: The rationale is "no teleport if version jump small" and a hard snap only if the version jump is large.
- **Soft delete blocking snapshot and 30-day recovery**: The rationale is recoverability before hard delete cascades all bird data.

### Frontend rendering pipeline

- **Layered scene composition**: NOT RECOVERABLE FROM PLAN
- **Compact SVG rig with bone-less transform tree**: NOT RECOVERABLE FROM PLAN
- **`plumageSaturation` modulating fill saturation**: It exposes trait change as a visual proxy without numeric personality values.
- **Mood-shaped perch zone, scan frequency, preen, and fluff poses**: It lets mood be perceived through behavior rather than a state-list.
- **Perlin head scan, preen cycle, and weight shift idle motion**: The mood-keyed parameters make birds feel alive while preserving server canonical state.
- **Perch change flight/hop transitions**: NOT RECOVERABLE FROM PLAN
- **Settle lighting lerp and volume duck**: The feature makes settle a quieting transition.
- **Settle undo within 5 seconds**: NOT RECOVERABLE FROM PLAN
- **Inline critical CSS and skeleton sky**: The rationale is fast first paint.
- **Quiet field instead of spinner if snapshot is delayed**: The plan explicitly says "no spinner" and renders a quiet field to preserve the scene's tone.
- **First bird visible within 500ms from partial snapshot or edge stub**: The rationale is the performance budget and first-bird requirement.
- **Reduced-motion cross-fades and removal of ambient leaf drift**: The feature honors reduced-motion while keeping audio, drift, and notebook unchanged.
- **Top bar fade to 10% opacity after pointer idle**: It supports "no in-scene chrome" and keeps the scene visually quiet.
- **Visitor mode using the same renderer with interaction disabled**: It gives a read-only ambient view without event POSTs or social UI.

### Audio pipeline

- **CallScheduler, MotifLibrary, PhraseBuilder, BirdVoiceNode, ChorusBus, ListenInMixer**: NOT RECOVERABLE FROM PLAN
- **Species motifs with runtime pitch/timing variation**: The rationale is procedural variation without looped samples.
- **No per-call allocation through buffer pool reuse**: The rationale is the performance budget.
- **Chorus stereo offset and per-instance detune**: The plan says this avoids phase-cancel artifacts from stacked identical loops.
- **Listen-in 1.5s ramp focus and reverse disengage**: The rationale is focused listening while keeping other birds audible.
- **WebAudio fallback to silence plus forced captions**: It keeps call information available when AudioContext fails and uses a matter-of-fact notice.
- **No recorded fallback**: The plan explicitly keeps recorded audio out of scope.
- **Call captions from motif metadata**: Captions stay aligned with procedural audio because grammar emits the caption at call play time.

### Accessibility surfaces

- **Screen-reader narration with `aria-live="polite"`**: The narration gives naturalist prose from snapshots rather than state-listing.
- **Priority queue for user-initiated events**: It lets greeting, offer, and settle narration happen faster than idle narration.
- **Opt-in captions**: Captions expose call information from the runtime grammar.
- **Keyboard navigation for top bar, birds, listen-in, escape, and offer palette**: The rationale is keyboard access as part of WCAG AA accessibility.
- **High-contrast focus ring on dawn/dusk backgrounds**: It keeps focus visible across changing scene palettes.
- **WCAG AA contrast tokens enforced in CI**: The rationale is meeting contrast requirements for chrome/copy.
- **Automated and manual accessibility testing**: The rationale is to verify VoiceOver, NVDA, reduced-motion, and keyboard-only paths each release.

### Performance and observability

- **Initial JS, first bird, render, memory, snapshot, and tick budgets**: The rationale is to protect first-bird speed, 60fps idle render, flat memory, small snapshots, and bounded tick compute.
- **Code-splitting settings, visits, and onboarding**: The feature supports the initial JS budget.
- **Procedural SVG birds and no large sprite atlases**: The rationale is bundle-size control.
- **Audio buffer pool**: The rationale is memory/performance stability.
- **`requestAnimationFrame` delta cap and pause when hidden**: The rationale is render stability and hidden-tab efficiency.
- **Virtualized notebook list**: The rationale is performance with growing notebook history.
- **Aggregate-only RUM, frame, audio, API, tick, and error metrics**: Observability monitors health without collecting per-bird state, interaction content, or personality values.
- **Synthetic Playwright checks from 3 regions**: The rationale is alerting on first-bird and tick regressions.
- **Not measuring engagement scores, streak proxies, or cross-account bird behavior aggregates**: The rationale is avoiding gamification proxies and cross-account behavior aggregation.

### Rollout and workstreams

- **P0-P4 milestone order**: NOT RECOVERABLE FROM PLAN
- **Birds-per-aviary ramp from 2 starters to adopt-next later**: The rationale is to watch audio mix quality at 5-7 birds before raising cap visibility.
- **Beta drift calibration soak for 3 weeks**: The rationale is tuning drift to the "weeks not sessions" promise.
- **Day-one magic-link funnel counts only**: Counts only keeps launch instrumentation bounded.
- **Staging-only drift delta instrumentation**: It allows tuning without production per-account trait metrics.
- **`visits_enabled` default off flag**: It keeps visits opt-in and bounded during rollout.
- **`weather_enabled` flag**: NOT RECOVERABLE FROM PLAN
- **`max_birds` flag from 2 to 7 ramp**: It supports the controlled birds-per-aviary ramp.
- **Suggested team parallelization workstreams**: The plan labels them as "suggested team parallelization," so the rationale is parallel team execution.
