## System-level intent

1. **Server-canonical honesty.** The plan's load-bearing architecture is that "the server is the only writer of personality state" and "the client is the only producer of interaction events." This shows up in the client/server split, data model, event endpoints, simulation tick, sync model, risks, and summary of load-bearing decisions. The plan says this boundary "makes the rest of the architecture honest" because drift correctness, multi-device sync, and the "no-last-write-wins invariant" depend on it.

2. **The aviary continues without the viewer.** The simulation tick is called the architectural unit of "the aviary continues without the viewer." This shows up in the render pipeline, tick loop, snapshot shape, transitions, loading state, and sync model. The first client frame should show birds "mid-action" so the aviary feels "already in motion," not woken up by the user's arrival.

3. **Slow, additive, asymmetric drift.** The plan repeatedly frames drift as "additive," "server-authored," and "monotonic toward expressive, never downward on neglect." This shows up in the defensible calls, event log, drift function, calibration targets, sync model, risks, and load-bearing decisions. Absence contributes zero; it does not subtract.

4. **No Tamagotchi, no gamification, no engagement accounting.** The plan restates an "absolute" refusal of streaks, XP, achievements, happiness meters, distress, decay, and visible or hidden visit-frequency widgets. This shows up in out-of-scope, notebook generation, bird-count ramping, telemetry, rollout, and the "just one streak counter" risk. The line between "observation of the aviary" and "observation of the user's behavior" is hard.

5. **Privacy by architectural absence.** Privacy is not just policy in the plan; it is designed as missing fields, missing tables, missing roles, and missing access. This shows up in synthetic UUIDs, the email encrypted/hash split, aggregate-only telemetry, "what is not in the data model," RUM, and "what we deliberately do not measure." The plan calls "architectural absence" the "durable defense."

6. **Naturalist observation, with matter-of-fact exceptions.** The product voice is observational and quiet in the notebook, captions, and narration: "naturalist prose," "observation-style sentence," "no exclamation, no second-person, no announcement framing." In errors, settings, deletion, unsupported browser, visit termination, and accessibility settings, the plan uses the "matter-of-fact voice" because naturalist phrasing in an error context "reads as evasive."

7. **The product notices rather than announces.** The plan rejects "welcome-back toasts, banners, modals, or any textual greeting surface," puts "no UI chrome inside the aviary scene," and says changing this would make the product "announces instead of notices." This shows up in out-of-scope, top bar chrome, loading state, rollout ramping, notebook voice, and load-bearing decisions.

8. **Accessibility is the actual product.** The plan's accessibility rule is "designed surface, not a checklist" and "the user gets the actual product, not a stripped-down fallback." This shows up in reduced-motion, narration, captions, keyboard navigation, WCAG AA contrast, settings, risks, rollout monitoring, and load-bearing decisions.

9. **Emergent procedural sound is an affective spine.** Calls are "procedural"; recorded audio is forbidden. Chorus should feel "emergent rather than cued," not a "phase-locked stack." This shows up in scope, out-of-scope, call grammar, audio pipeline, WebAudio fallback, performance budget, risks, and load-bearing decisions.

10. **Identity continuity matters to the relationship.** Birds have stable `bird_id`s, renaming does not change personality, mood, or call, and the user never sees personality vector values because otherwise "the bird becomes a number and the relationship collapses." This shows up in the birds table, rename endpoint, identity-continuity risk, export, and load-bearing decisions.

11. **Performance is affective, not only technical.** The plan ties bundle size, "time to first bird visible <500ms," 60fps, no memory growth, and CDN/static delivery to felt aliveness. It calls the 500ms threshold the "affective-perf bridge": below it, "the aviary feels like it was already running."

## Per-feature whys

### 1. Scope

- **Web-only single-page application:** NOT RECOVERABLE FROM PLAN

- **Last two major versions of Chrome / Safari / Firefox / Edge:** NOT RECOVERABLE FROM PLAN

- **Single-user accounts:** The plan refuses social-network surfaces, shared aviaries, public profiles, comments, chat, and co-presence; the account model preserves one user's aviary rather than a networked product.

- **Email magic-link sign-in:** NOT RECOVERABLE FROM PLAN

- **No passwords and no SSO at v1:** NOT RECOVERABLE FROM PLAN

- **One canonical aviary per account:** This supports the server-canonical model: every device reads "the same record" and the aviary is one account's state, not a set of client-local copies.

- **Hard cap of seven birds:** NOT RECOVERABLE FROM PLAN

- **Starting roster of two starter birds:** The plan uses two birds to make the empty-aviary state brief and to give the new account an immediate small aviary; a third bird is withheld until aviary age makes it available.

- **Six-species starter pool:** NOT RECOVERABLE FROM PLAN

- **Random starter assignment with no catalog and no user selection:** The plan's rationale is avoidance of catalog, rarity, and user-pick surfaces; "the system picks," and the user names the birds at adoption.

- **Server-side simulation tick:** The tick is the unit of "the aviary continues without the viewer," the sole writer of mood and personality state, and the mechanism that keeps the client from running its own state machine.

- **Multi-device sync:** It is "an architectural property of the server-canonical model" because two clients receive the same snapshot and do not reconcile local simulations.

- **Return-greeting:** NOT RECOVERABLE FROM PLAN

- **Listen-in:** The plan's rationale is a focused auditory surface that re-balances the bus without muting other birds; muting would convert the aviary into "soloable tracks," which is "a different audio surface."

- **Offer seed / song fragment / still pool:** Offers create event bumps for drift, but the per-bird cooldown is a guard against "curiosity-trait saturation." Acceptance is server-determined so the client never reports state.

- **Settle:** Settle creates a "gentle evening shift," softens visual lighting and audio, records notebook-relevant state, and ends the presence window without becoming a penalty or drift mechanic.

- **Field notebook:** The notebook is "read-only," "sparse," and observation-style so it is not a feed, not a generic event log, and not a record of the user's behavior.

- **Per-bird adoption and rename:** Adoption lets the user name the birds the system picked. Rename preserves identity because it "does not change personality, mood, or call."

- **Bird-count expansion tied to aviary age:** Age-based expansion avoids visit count, interaction score, level-up, tier, and milestone language. New birds appear as a "soft prompt," not a notification or congratulations surface.

- **Visit invitations:** The visit is "observation, not co-presence." Read-only sessions let a visitor see "what the host sees" without submitting events or drifting the host's birds.

- **Per-invite opt-in, revocable invites, host-side visit log:** Opt-in and revocation support host control over who can observe the aviary.

- **30-day visit-invite expiry:** NOT RECOVERABLE FROM PLAN

- **Procedural WebAudio call synthesis:** Procedural calls preserve variation, support chorus, avoid recorded audio, and help keep the initial bundle under the 2MB budget.

- **Screen-reader narration, call captions, reduced-motion mode, keyboard navigation, WCAG AA contrast:** These are required because accessibility is a "designed surface" where the user gets "the actual product."

- **Aggregate-only telemetry pipeline:** It prevents telemetry from reconstructing "a user's relationship with their aviary" and keeps per-bird interaction state inside the per-account simulation database.

- **Account export:** The export is "the user's own data"; the plan includes current bird records, moods, notebook entries, settings, and personality vectors in structured JSON.

- **Account deletion with soft then hard delete:** Soft deletion preserves a cancel path before hard deletion.

- **30-day account deletion window:** NOT RECOVERABLE FROM PLAN

- **Synthetic account UUIDs and email only on account record:** This "cuts PII out of every log line and every partition key from the source."

### 2. Architecture

- **Static web client:** The client renders state, submits interaction events, and pulls snapshots, but "holds no personality state" so it cannot become a competing simulation.

- **Aviary API:** The API validates, persists events, and returns; "no business logic for simulation in the request path" preserves the worker as the only state writer.

- **Simulation worker:** The worker owns tick processing, reads the event log, writes canonical state, and is the mechanism for slow drift and no-last-write-wins.

- **Auth service in same deployable for v1:** NOT RECOVERABLE FROM PLAN

- **Account data CRUD endpoints:** These collect export, delete, session, email, visit, and invite operations so account control is explicit and not mixed into the simulation path.

- **No queueing system at v1 scale:** "A queueing system is not required at v1 scale"; the load profile does not justify the operational complexity.

- **Hard client/server split:** This is the single line that keeps drift correctness, multi-device sync, and no-last-write-wins from collapsing.

- **Client-side rendering from compact snapshots:** This makes "aviary appears already in motion" achievable because the client places birds in their current positions and motions from the snapshot.

- **Single-region origin:** The plan says global low-latency reads are unnecessary because "the aviary is one user's account" and a single-region read is fine at v1 scale.

- **CDN edge for static assets:** CDN caching supports fast initial HTML, CSS, JS, and asset delivery, feeding the time-to-first-bird budget.

### 3. Data model

- **`accounts`:** The table centralizes email and deletion state; the email encrypted/hash split is "non-negotiable" because email is PII.

- **`email_encrypted` and `email_lookup_hash`:** Encryption protects the address at rest; the HMAC lookup lets the system find an account without comparing plaintext email in SQL.

- **`aviary_created_at`:** This drives bird-count expansion by aviary age, preserving the rule that expansion is not tied to visit count or interaction score.

- **`visit_notifications_enabled` default false:** The plan says visit notifications are "off by default" and other notification surfaces are silent.

- **`birds.bird_id`:** The stable UUID preserves identity continuity; "the day-one bird is that bird forever."

- **`birds.personality_vector`:** It is hidden, fixed-shape, server-written state. Keeping it out of API write paths prevents personality from becoming client-set or user-visible numbers.

- **`birds.current_mood`, perch fields, and `state_version`:** Persisted mood/perch make the next session start from the current aviary rather than a default; `state_version` supports snapshots and conflict detection.

- **`species`:** NOT RECOVERABLE FROM PLAN

- **`event_log`:** It is the "source of truth for all user-originated inputs" and the only path by which client activity influences state.

- **Append-only event log:** Append-only order and fixed delta shape make crash re-processing idempotent and keep clients from setting absolute state.

- **Closed event type set:** A small, fixed, versioned event set keeps v1 inputs bounded and validates user activity as events rather than state.

- **`presence_tick`:** Presence requires visible, focused, and recently active signals; it feeds drift through normalized presence, not ranking.

- **`presence_end`:** It closes the presence window when signals drop so presence does not continue silently.

- **`greeting_observed`:** It records that a rendered greeting was actually seen, because some clients may close before rendering.

- **`listen_in_start` / `listen_in_end`:** These produce per-bird listen-in time used for drift weighting and listen-in caps.

- **`offer_seed` / `offer_song` / `offer_pool`:** The event records the offer attempt, while acceptance remains server-determined from current mood and personality.

- **`settle_gesture`:** The event is for notebook use and the gentle evening shift, and is "not used for drift beyond ending the presence window."

- **`notebook_entry_write`:** This internal event records the fact that the worker generated an entry while keeping entry text in `notebook_entries`.

- **`presence_state`:** It is the operational source for current presence and accumulated session presence, but drift still reads from `event_log`; this makes it a sanity check, not a second drift source.

- **`notebook_entries`:** Entries are sparse, read-only, naturalist prose, not generic logs. They remain available indefinitely rather than archived or hidden.

- **`visits` and `visit_invites`:** These support observation without co-presence; visitors get read-only snapshots and cannot drift the host's birds.

- **`sessions`:** Session listing and revocation give account control; revocation invalidates bearer tokens immediately.

- **`telemetry_aggregate`:** This table holds operational counts, histograms, and p99 latencies only, with no per-account, per-bird, or per-event linkage.

- **No global tables:** The absence of `all_birds`, `leaderboard_view`, and public directories makes social-network reappearance "harder rather than easier."

### 4. API surface

- **Versioned JSON HTTPS API:** Stable `v1` responses let the SPA and snapshot schema stay deployable without changing client assumptions.

- **`POST /v1/auth/magic_link`:** Returning 204 for success and "email not found" gives enumeration resistance.

- **Magic-link consume:** Used links are invalidated and links expire after 15 minutes; this prevents replay beyond the short sign-in window.

- **Sign-out and session revocation:** These invalidate the current session so account control reaches the API edge immediately.

- **Email change request and confirm:** NOT RECOVERABLE FROM PLAN

- **`GET /v1/aviary/snapshot`:** The snapshot is the single small document the client renders from; ETag/304 support avoids unnecessary payloads.

- **`GET /v1/aviary/snapshot/stream`:** Streaming or long-poll pushes snapshots on `state_version` change; polling fallback keeps the surface working when the stream is unavailable.

- **Notebook endpoint:** It pages newest-first read-only entries, supporting scroll-back without turning the notebook into an editable surface.

- **Single-bird detail endpoint:** It returns recent notebook mentions but never the personality vector; a fixed response-shape test preserves the hidden-number rule.

- **Interaction event endpoints:** They return 202 once the event is durable and do not block on simulation processing, preserving the request path as append-only.

- **Event validation:** Session validity, ownership, rate limits, and payload shape prevent a "possibly-malicious renderer" from mutating state or submitting another account's bird event.

- **Event transaction never writes `birds`:** This is the structural guarantee that no client write path can mutate personality state.

- **Account summary:** It returns no email and no PII beyond what the user can see in settings.

- **Account export endpoint email delivery:** NOT RECOVERABLE FROM PLAN

- **Account delete and cancel:** Soft deletion gives a cancel path until hard delete; cancel is available during the window.

- **Visit notification toggle:** Visit notifications are off by default, matching the silent-notification rule.

- **Visit log and invite endpoints:** These give the host control over outstanding and historical visit access.

- **Bird rename endpoint:** Rename "does not change personality, mood, or call" and does not change `state_version`, preserving identity and avoiding false personality-state churn.

- **Snapshot shape:** It excludes personality vector, includes mood as enum, includes pose-slot and pose-phase for mid-action rendering, and exposes `visual_saturation` as the only personality-shaped rendering hint.

- **`is_settled` snapshot flag:** It drives the soft-settle visual and motion suppression state from canonical state.

- **Visit-invitation sequence:** The flow is "frictionless" and does not gate a visit behind a sign-up wall; the visitor sees the host snapshot exactly.

- **Read-only visitor sessions:** They reject `/v1/events/*` with a matter-of-fact error so visits never become co-presence or interaction.

### 5. Simulation engine design

- **Tick loop:** The loop turns events into "a slowly-drifting aviary" and serializes each account's state changes in one transaction.

- **Pure transform functions:** Keeping transforms as pure functions from `(event, current_state)` to `(delta, side_effects)` makes event handling testable and prevents request-path state mutation.

- **Atomic per-account tick:** Atomic updates mean all personality deltas from a tick commit together or none do.

- **60s tick pace:** It is fast enough for mood transitions to feel continuous and slow enough that one tick's drift is small.

- **Active account set:** Active accounts are those with recent events or active clients; idle accounts are dropped after 24 hours to avoid unnecessary ticks.

- **Catch-up tick:** Re-activated idle accounts process queued events at once, safe because drift is small and additive.

- **Drift as low-pass filter:** The formula creates slowly varying trait accumulators and saturation so relationship growth feels like "settling rather than escalating."

- **Presence signal:** It is monotonic with diminishing returns so more watching can matter while a few hours/day does not dominate.

- **Listen-in signal:** It localizes drift to the bird being listened to without exposing a management meter.

- **Offer signal:** It gives accepted or near-accepted offers small curiosity or boldness bumps without letting offers directly set traits.

- **Saturation term:** `(1 - current_T)` slows drift near expressive ceilings, making "feels alive over weeks" plausible.

- **Neglect contributes zero:** The plan explicitly avoids subtract, decay, and penalty so ignored birds remain ambient rather than distressed.

- **Instrument-level drift after ~1 week:** The harness needs measurable deltas so the team can test drift before users can necessarily feel it.

- **User-visible drift after ~3 weeks:** The intended felt shape is not session-by-session change but noticing when looking back.

- **Daily synthetic-account CI harness:** It gates calibration windows without auto-tuning, forcing review when constants or drift function fall out of range.

- **Mood transitions:** Mood is fast-timescale, shaped by time of day, recent interactions, ambient events, and personality.

- **Persisted mood across sessions:** Birds do not reset to neutral on tab open; the first snapshot is the mood the server has computed.

- **Perch selection:** Perch is a "signal the user reads, not a layout the user controls," so bird position communicates mood/personality without becoming direct manipulation.

- **Bird-to-bird interaction:** Chorus, wary contagion, and greeting chains make the aviary feel like "a small social system."

- **Chorus events:** The worker provides windows and the client mixes overlap so chorus feels emergent rather than cued.

- **Wary contagion:** Bounded, decaying spread lets nearby birds react without taking over the aviary.

- **Greeting chains:** Staggered nearby responses make returns feel social without a textual welcome surface.

- **Runtime call grammar:** Client-side synthesis from motifs avoids audio assets while matching mood, personality-shaped hints, and call windows.

- **No runtime grammar download:** Bundling the grammar keeps calls available in the initial client and lets old clients degrade gracefully.

- **Notebook generation:** Templating rather than LLM keeps prose bounded, sparse, structural, and testable against voice rules.

- **Notebook cadence:** Sparsity prevents "an entry per session" and preserves the notebook as observation rather than feed.

- **Empty-aviary and starter-bird state:** The brief empty state and soft fly-ins create the beginning without a catalog or user-pick flow.

- **New-bird availability schedule:** Age-based availability prevents the bird ramp from becoming a streak, score, or level.

### 6. Frontend rendering pipeline

- **Single canvas scene:** A small 2D layered scene keeps the aviary as the main surface rather than a page of components.

- **Background / midground / foreground layers:** The layers give depth; parallax is "subtle" and exists "not to show off."

- **Bird silhouette, plumage, pose, and micro-motion:** Composite rendering uses snapshot state plus small render-time noise so birds are expressive without client-owned simulation.

- **Birds never still:** Even settled birds breathe, preventing the scene from reading as paused.

- **60fps frame loop:** Dirty-rect compositing and cached framebuffers keep cost bounded at `O(birds_in_motion)` and support a 30-minute idle session.

- **No memory growth over 30 minutes:** Reused buffers, framebuffers, fixed pools, and bounded DOM mutation keep long sessions from degrading.

- **Flight transitions:** Renderer interpolation prevents birds from "teleporting" between server-written perch targets.

- **Fresh sign-in first frame:** There is no wake-up or fade-from-static because the aviary should already be running.

- **Loading state as quiet field:** A quiet field avoids spinner language and matches the empty-aviary state until the first bird appears.

- **Top bar chrome:** Four icons keep controls available while fading chrome preserves the aviary surface; "the fade is the small cost of putting controls anywhere."

- **No UI chrome inside the aviary scene:** This prevents buttons, badges, labels, and hover-tooltips from breaking the central surface.

- **Reduced-motion mode:** It changes the "visual register" while keeping simulation, calls, mood, drift, and notebook behavior intact.

- **Same code path for reduced motion:** Shared code with "motion budget" parameters keeps motion and reduced-motion modes in lockstep.

- **Modal surfaces:** Settings, account, notebook detail, visit log, and rename are keyboard-navigable, focus-trapped, and code-split to protect the initial bundle.

### 7. Audio pipeline

- **WebAudio graph:** A small node graph per call keeps synthesis lightweight and mixes through a chorus bus.

- **Lazy audio context:** Browser autoplay policies require a user gesture, so the audio context is created on first interaction that requires audio.

- **Procedural call synthesis:** Motif, pitch, duration, oscillators, filters, envelope, and glide create calls that reflect mood and `visual_saturation`.

- **Seven simultaneous calls:** The plan designs procedural synthesis to support the seven-bird cap on a five-year-old laptop.

- **Chorus mixing:** Pan separation, gain reduction, pitch jitter, and timing jitter prevent clipping and avoid a phase-locked stack.

- **Listen-in mix:** Gain ramps focus one bird while keeping the others audible; the rationale is that the aviary should not become soloable tracks.

- **Settle and evening shift:** Audio softening and high-frequency falloff match the visual lighting shift and persist until re-engagement.

- **WebAudio fallback:** Silence with captions preserves the no-recorded-audio rule; recorded variation would blow the bundle budget, while lower variation would feel canned.

- **Audio captions:** Captions are generated from the actual call parameters, so text matches what was played rather than a fixed motif label.

### 8. Accessibility surfaces

- **Screen-reader narration:** A slow polite live region describes the same snapshot state in the naturalist voice, giving the actual aviary rather than a separate summary.

- **Narration cadence:** The slow 30-60 second idle cadence prevents a constant stream of announcements; user-initiated events are prompt but still framed as observations.

- **Call captions:** Captions help when audio is off, unavailable, or not getting through, and they match the synthesized call.

- **Reduced-motion setting:** It can come from OS preference or account setting and persists per account.

- **Keyboard navigation:** Keyboard reachability covers top bar, scene birds, listen-in, offer, settle, notebook, and modal exit so all interactive surfaces are operable without pointer input.

- **Focus indicators:** They must be visible against bright and dim aviary states because focus lives on top of the scene.

- **WCAG AA contrast:** Contrast applies to all user copy, especially chrome and captions over the aviary, where soft background plates guarantee readability.

- **Accessibility settings modal:** The surface uses matter-of-fact voice because users need to know what toggles do, "not to be charmed."

### 9. Performance budgets and observability

- **Initial JS bundle under 2MB gzipped:** The cap drives procedural audio, compact assets, and code-splitting for less-frequent surfaces.

- **Time to first bird visible under 500ms:** This is the "affective-perf bridge" that makes the aviary feel like it was already running.

- **60fps idle motion:** The constraint covers 30 minutes, not just startup, preserving felt aliveness over real sessions.

- **No memory growth:** CI checks a synthetic 30-minute Chrome session so long-running render and audio paths stay bounded.

- **Synthetic performance checks:** Automated browsers measure first bird, interactivity, frame timing, memory, audio-context latency, and tick latency from common geographies.

- **Throwaway synthetic accounts:** The fleet avoids testing on real accounts, preserving the privacy boundary.

- **Aggregate-only RUM:** Metrics are defined without per-account dimensions so they cannot reconstruct a relationship with an aviary.

- **Error budgets:** p99 tick latency, endpoint error rate, and snapshot delivery thresholds catch degradation before users notice slow-running aviaries.

- **What not to measure:** Per-account presence, drift, offer acceptance, listen-in, and notebook rates are not defined because they would be "observation of the user."

### 10. Sync model

- **Server as only canonical source:** Both devices read the same canonical state and therefore do not "sync" in an operational sense.

- **Database-role enforcement:** The API role lacks update permission on personality and mood, making the boundary IAM/role-level rather than code-review-level.

- **Additive event-log-order processing:** This makes last-write-wins personality loss unreachable.

- **No-last-write-wins rule:** The plan rejects LWW because a stale device could silently delete another session's drift.

- **Conflict surfaces:** Known 409, 410, and 503 states use matter-of-fact copy; stale state usually triggers re-pull and re-render without visible drama.

- **Visitor session termination:** Host revocation takes effect at the next snapshot pull and returns a matter-of-fact "visit no longer available" state.

- **No client-to-client sync:** There is nothing to sync because clients read canonical server state.

- **No offline-first:** Offline mode is rejected because the simulation is server-side and there is no local simulation to fall back to.

- **No service-worker state caching:** Static assets may cache, but state is always server-fresh so stale aviaries are not displayed as live state.

### 11. Rollout

- **Single fully-featured v1:** The plan refuses a soft launch missing v1 features because it would reopen design decisions and create a place where "a Streak Counter quietly slips in."

- **Internal alpha:** The team uses the product and synthetic accounts support the calibration pass.

- **Closed beta:** External users characterize real usage, calibration, and load profile while preserving the same v1 build.

- **Public launch:** Public launch uses the same build as closed beta, with beta data preserved.

- **Ramp the bird count:** The soft prompt appears when the aviary is old enough and avoids "level up" or "tier" language.

- **Day-one instrumentation:** Drift velocity, mood distribution, notebook cadence, latency, tick queue, synthetic performance, and aggregate RUM are present from day one so calibration and regressions are visible.

- **No per-account instrumentation:** Day-one instrumentation excludes per-account, per-bird, and per-event metrics because the data pipeline cannot read the simulation database.

- **Post-launch drift monitoring:** Real-usage drift is compared to synthetic calibration and reviewed when it differs by more than a small factor.

- **Audio uncanniness monitoring:** Procedural call grammar is the affective spine, so "calls sound off" reports are investigated at the synthesis level.

- **Accessibility regression monitoring:** Accessibility surfaces are CI-tested and any regression is P0 because accessibility is on the same critical path.

- **Sync correctness monitoring:** No-last-write-wins test failure is P0 because it compromises canonical drift.

### 12. Risks

- **Drift calibration drift:** The risk is that real usage makes drift too fast or too slow; mitigations keep constants reviewed and gated.

- **Sync correctness:** The risk is silent lost drift; mitigations put the API role, E2E test, monotonic `state_version`, and append-only log around the boundary.

- **Audio uncanniness:** The risk is calls sounding unnatural or mechanical; mitigations use human review, tuned chorus parameters, P0 reports, and special nightjar review.

- **Accessibility regressions:** The risk is a stripped or broken accessibility surface; mitigations test screen reader, reduced-motion, contrast, keyboard, and paired review.

- **Bundle budget regressions:** The risk is degrading felt-aliveness on slow connections; mitigations enforce initial bundle size and code-split less-frequent surfaces.

- **Notebook voice drift:** The risk is event-log or gamification language; mitigations include voice review, generator variable tests, and weekly audits.

- **Identity continuity:** The risk is replacing a bird without saying so; mitigations preserve `bird_id`, migration assertions, two-engineer review, and export.

- **"Just one streak counter" temptation:** The risk is organizational; mitigations restate non-goals, reject gamification PRs, and perform quarterly surface review.

### 13. Open questions resolved

- **Species pool composition:** One nightjar-like species creates a night-active role; the other five fit the "soft blues, greens, warm browns, muted ochres" palette.

- **Starter-bird assignment without replacement:** This ensures the user gets two distinct species.

- **New-bird introduction animation:** Soft fly-in and initial `curious` mood introduce a bird gently without level-up ceremony.

- **Naming defaults:** NOT RECOVERABLE FROM PLAN

- **Name constraints:** NOT RECOVERABLE FROM PLAN

- **Notebook revision history:** Server edits are rare and visible with matter-of-fact "edited at" markers, preserving read-only user semantics while allowing generated-text fixes.

- **Visit notifications on host side:** Off by default and opt-in; when enabled, the email uses matter-of-fact voice.

- **Account export JSON format:** JSON includes bird records and vectors because "the export is the user's own data" and should show what is stored.

- **One-time export download link valid for 24 hours:** NOT RECOVERABLE FROM PLAN

- **Account deletion cancellation window:** NOT RECOVERABLE FROM PLAN

- **Session token rotation every 30 days:** NOT RECOVERABLE FROM PLAN

- **Browser support floor:** Older browsers get matter-of-fact unsupported-browser copy; the exact "last two major versions" threshold is NOT RECOVERABLE FROM PLAN

### 14. What the team will build first

- **Simulation database, event log, and simulation worker first:** "Without these, no other surface has state to read."

- **Auth flow and account record second:** "Without these, no event has an `account_id`."

- **Snapshot endpoint and client renderer third:** "Without these, no event has anywhere to land visually."

- **Call grammar and audio pipeline fourth:** "Without these, the aviary is silent."

- **Drift function fifth:** The engine can produce mood and perch first; drift lands when "the basic engine is honest."

- **Field notebook generator sixth:** It is small, but "it is the voice of the product," so it lands after the engine.

- **Listen-in, offer, settle seventh:** They need audio for listen-in and the worker for offer acceptance.

- **Accessibility surfaces eighth:** They are "designed alongside the client, not after."

- **Visit flow ninth:** It is a read-only client/API mode that needs the snapshot endpoint first.

- **Account export, deletion, session list, visit log tenth:** These are CRUD surfaces that need the account record first.

### 15. Summary of load-bearing decisions

- **Server-only personality writer:** If changed, sync collapses and no-last-write-wins becomes unreachable.

- **Additive monotonic drift:** If changed, "the product becomes a Tamagotchi."

- **Procedural calls and no recorded audio:** If changed, "the chorus mechanic collapses and the bundle budget is unrecoverable."

- **No visible personality vector values:** If changed, "the bird becomes a number and the relationship collapses."

- **Read-only sparse notebook:** If changed, "the notebook becomes a feed and the voice is diluted."

- **No aviary-scene UI chrome, fading top bar, no toasts/banners/welcome-back text:** If changed, "the product announces instead of notices."

- **Server-side simulation tick:** If changed, the aviary no longer continues without the viewer.

- **Synthetic UUID account rule:** If changed, PII leaks into logs and aggregates.

- **Telemetry pipeline without simulation-database access:** If changed, privacy is broken at the architectural level.

- **Accessibility as designed surface:** If changed, the product rations quality by sensory ability.
