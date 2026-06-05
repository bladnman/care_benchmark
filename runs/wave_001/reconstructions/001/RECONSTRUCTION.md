## System-level intent

- **Observational, not custodial or gamified.** The plan repeats this through the out-of-scope list ("No achievements, no streaks, no levels," "Birds do not die, do not get hungry") and the conclusion's product framing: "observational, not custodial" and "specific, not gamified."

- **Quiet, not announced.** This appears in the explicit bans on a "Welcome back!" toast, push notifications, friend notifications, and in "Notification Service (Quiet)." The conclusion names the desired feeling as "noticed rather than announced at."

- **A place that continues without the viewer.** The plan ties this to server-side tick, persistent mood, initial load with birds "mid-action," ambient motion, and the success criterion that users "feel the aviary continues without them."

- **Attention changes the relationship over weeks.** Presence is the "primary drift input," "Presence-time" is dominant, listen-in duration is strong, and drift should be "measurable" after about a week and "visible" after about three weeks.

- **Continuity is protected by server authority.** "Server is sole writer" appears across state, mood, personality vectors, and sync. The plan rejects "client-to-client sync," "client-submitted absolute values," and "last-write-wins on personality state" to avoid birds that "reset" or corrupted drift.

- **Birds must remain individually knowable.** The plan gives birds "stable internal identity," user names, persistent personality vectors, mood, and recognizable call signatures. The 7-bird cap exists because "chorus must remain individually recognizable."

- **Naturalist voice for product surfaces, matter-of-fact voice for system surfaces.** The plan uses "naturalist prose" for the field notebook, captions, and screen-reader narration, while account, error, sync, and accessibility-settings surfaces use "capitalized, direct, no naturalist phrasing."

- **Accessibility is the same product, not a degraded fallback.** Reduced-motion mode is "not 'animations off'" but "its own designed surface." Accessibility success requires screen-reader, reduced-motion, and keyboard-only users to have the "same affective experience."

- **Performance should disappear into the experience.** The plan insists on "No spinner," "Time to first bird <500ms," "60fps idle motion," and "No memory growth." Recorded audio fallback is rejected because it would cause "bundle budget collapse" and "break spell."

- **Telemetry should not reconstruct the user's relationship.** Observability measures aggregate timings and errors, while "What We Don't Measure" excludes per-bird state, per-account interaction history, and any telemetry that could reconstruct the relationship with the aviary.

## Per-feature whys

### Scope

- **2 starter birds** - The rollout section says v1 starts with 2 birds as the "minimum for small social system."

- **Starter birds selected by system**: NOT RECOVERABLE FROM PLAN

- **Max 7 birds per aviary** - The plan states the cap is because "per-bird call signatures must remain individually recognizable" and "chorus must remain individually recognizable."

- **Pool of about 6 species**: NOT RECOVERABLE FROM PLAN

- **Stable internal identity, user-assigned name, persistent personality vector** - These support the conclusion's "small social system the user gets to know intimately" and success around noticing that "Pip is bolder than she used to be."

- **Listen-in** - Listen-in lets the user focus a bird so its call "rises in mix"; listen-in duration is a "strong" drift input, so focused attention helps shape personality.

- **Offer** - Offers are small interactions: "offers accepted" and offers near a bird produce small drift, and an accepted offer can move mood toward "content." The plan keeps them non-custodial by making them small, cooldown-limited events rather than hunger or happiness upkeep.

- **Specific offer types: seed, song fragment, pool**: NOT RECOVERABLE FROM PLAN

- **Settle** - Settle is an "evening transition" and a "quieting" gesture that affects mood, not drift.

- **Settle undo**: NOT RECOVERABLE FROM PLAN

- **Field Notebook** - The notebook carries "auto-generated observations" in "naturalist prose" and records daily summaries, notable events, or session boundaries so the aviary is narrated in the same voice as the rest of the product.

- **Return-greeting** - It is a "procedural bird notice"; user-initiated events get a screen-reader priority bump, and the conclusion says the product should make the user feel "noticed rather than announced at."

- **Magic-link sign-in**: NOT RECOVERABLE FROM PLAN

- **Email plus UUID synthetic ID**: NOT RECOVERABLE FROM PLAN

- **Per-device session tokens** - The sync-conflict surface includes an account settings "session list with revocation affordance," so per-device tokens support session control across devices.

- **Soft-delete for 30 days, then hard-delete** - The account endpoint says delete sets `soft_delete_at` with a "30-day window to recover," then hard-deletes after the window.

- **Multi-device sync through server-side canonical state** - The plan avoids client reconciliation: clients pull the same snapshots, there is "No client-to-client sync," and there is "No eventual consistency to reconcile."

- **Server-side simulation tick** - The tick advances canonical state and the conclusion says it runs "whether or not anyone is watching," supporting continuity while the user is away.

- **Client-side interpolation between snapshots** - The render pipeline uses interpolation "for smooth motion" between server snapshots.

- **Precise presence definition** - Presence is the "primary drift input"; the risk section says if presence is "too lax" it can corrupt drift, so all three conditions are required.

- **Procedural WebAudio calls** - Procedural calls avoid pre-recorded loops and support real-time variation; the audio risk says canned calls would "break spell."

- **Chorus mixing and listen-in mix decay** - A focused bird rises while other birds quiet "to ambient (not silence)," preserving both focus and the living background without "hard cut."

- **Silent captions fallback for WebAudio failure** - The fallback keeps the experience accessible when WebAudio is unavailable; recorded fallback is rejected because of "bundle budget collapse" and "canned audio break spell."

- **Screen-reader narration** - Narration uses "naturalist prose" rather than state lists so screen-reader users get the same affective experience, not a mechanical report.

- **Reduced-motion mode** - It is a "designed surface, not fallback," using cross-fades and slowed color shifts so reduced-motion users still get the same product.

- **Call captions** - Captions expose calls through "naturalist prose" near the calling bird, matching the field notebook voice for accessibility.

- **WCAG AA contrast** - The plan requires all user-copy text to pass WCAG AA so text remains readable across top bar, settings, account, error, caption, and narration surfaces.

- **Full keyboard navigation** - Keyboard-only users must be able to "complete all interactions"; tab order, arrow focus, Enter listen-in, Escape exit, and offer shortcut support that.

- **Read-only visit invitations** - The optional social layer allows visits while preserving the no-social-network intent: no co-presence, no chat, no friend notifications, and no interaction writes.

- **One-time visit links**: NOT RECOVERABLE FROM PLAN

- **Revocable visit invitations** - Revocation gives the host control over an invitation; the plan also keeps visits "logged silently."

- **Single horizontal screen and three perch zones**: NOT RECOVERABLE FROM PLAN

- **Day/night cycle using local time** - Time of day drives drowsy and alert moods and supports the aviary as a continuing place.

- **Ambient weather** - Weather affects mood and calls, such as rain dampening vocal frequency and wind moving birds toward alert or wary.

- **Ambient micro-motion** - Leaf and feather drift are a "visual cue that aviary continues without viewer."

- **Calm naturalist color palette** - The design notes call for "Soft blues, greens, warm browns, muted ochres" and deliberately absent saturated accents to keep a calm, naturalist aesthetic.

### Architecture

- **State Service** - A single canonical aviary state per account supports snapshot reads and event writes while keeping server ownership of state.

- **Simulation Service** - It processes events, updates personality, transitions moods, and advances ambient state so the aviary has continuity beyond the tab.

- **Auth Service**: NOT RECOVERABLE FROM PLAN

- **Event Log Service** - Append-only events are consumed by simulation tick, preventing last-write-wins personality conflicts and preserving ordered drift inputs.

- **Notification Service (Quiet)** - It is limited to visit invitation emails, export emails, and deletion reminders, with "No push, no in-product alerts," preserving quietness.

- **React or Preact SPA** - The plan names the reason directly: "lightweight, mature ecosystem."

- **Worker threads for audio** - Audio synthesis runs separately "to avoid main-thread blocking."

- **Optional service worker** - It caches static assets and enables offline state snapshots, but keeps offline state read-only with "no interaction writes."

- **Server-rendered initial HTML with embedded state** - This is "for speed" and enables immediate hydration with no spinner or ready animation.

- **No client-side simulation** - Clients "never compute drift or mood transitions" so personality and mood remain server-authored and sync-safe.

- **Bundle, first-bird, frame-rate, and memory guardrails** - These are the technical success conditions and protect first-frame immediacy and long-session stability.

### Data Model and API Surface

- **Aviary current time of day and current weather** - These fields support day/night cycle, mood transitions, ambient state, and weather timers.

- **Bird mood, perch zone, idle animation state, last call time, call timing offset** - These fields make mood visible through idle motion and keep calls varied and recognizable.

- **Offer cooldown**: NOT RECOVERABLE FROM PLAN

- **Interaction Events append-only payloads** - The events carry offers, listen-in, settle, and presence pings into the simulation tick without clients writing absolute state.

- **Field Notebook Entry source** - The source distinguishes daily summary, notable event, and session boundary, matching the open question about exact notebook triggers.

- **State snapshot endpoint** - The endpoint returns the full aviary snapshot in "kilobytes, not megabytes," supporting fast state reads.

- **Incremental state endpoint with `since`** - This is explicitly an "Optimization for long-running sessions."

- **Event endpoint returns 202 Accepted** - There is "no immediate state update" because the event log is processed by the server-side simulation tick.

- **`notebook_scroll` event with no simulation impact**: NOT RECOVERABLE FROM PLAN

- **Account export endpoint** - It returns a JSON snapshot of aviary state; the plan specifies email delivery because the response is a "large payload."

- **Visit invite read-only endpoint** - It returns aviary state with "no interaction allowed," preserving the read-only nature of visits.

- **Visit log** - It records visits silently for the host, consistent with revocable, quiet sharing.

### Simulation Engine Design

- **Tick cadence around once per minute**: NOT RECOVERABLE FROM PLAN

- **Drift function weighted by presence-time, listen-in, and offers** - This makes the user's "idle attention and small interactions" shape personality over weeks.

- **Monotonic drift toward expressive** - Traits "never move down on neglect," preserving the non-Tamagotchi rule that birds do not decay or punish absence.

- **Calibration target of one week measurable, three weeks visible** - This supports the promise that birds "feel alive over weeks" and that users notice personality drift over time.

- **Server-computed cumulative drift from event log** - The plan rejects client-submitted absolute values to avoid corrupted personality drift.

- **Mood transitions from time, interactions, ambient events, and personality** - Mood responds to aviary conditions and bird traits so behavior feels situated rather than reset.

- **Mood persistence across sessions** - Mood "does not reset on tab open," maintaining continuity from session end to session start.

- **Gradual mood transitions** - Gradual transitions avoid instant state flips and support the plan's broader "no hard cuts" aesthetic.

- **Mood-shaped idle motion** - Wary scanning, content preening, curious tilting, and drowsy posture let the user read mood from behavior.

- **Species motif libraries and unchanged core motifs** - Core motifs remain recognizable while timing and pitch vary, preserving individual call signatures across drift.

- **Reusable audio buffers** - Buffer reuse supports the "No memory growth" performance budget.

### Sync Model and Rendering Pipeline

- **Snapshot pulls on visibility change, frame gaps, and keepalive** - These keep clients aligned with canonical state after tab changes, laptop resume, and long visible sessions.

- **No eventual consistency reconciliation** - With a single canonical aviary and ordered event log, the plan avoids reconciliation surfaces for personality state.

- **Matter-of-fact sync conflict surfaces** - Expired magic links, timeouts, and outages use direct copy such as "The link may have expired," not naturalist prose, matching the voice split.

- **First frame with birds mid-action** - "Motion already in progress" and no fade-in make the aviary feel like it was already there.

- **Reduced-motion cross-fade rendering** - Cross-fades replace frame-by-frame animation while keeping the same product in a quieter visual register.

- **Client-side ambient motion with no per-leaf state** - Ambient motion cues continuity, while no per-leaf state keeps it lightweight client-side generation.

### Audio Pipeline

- **Client-side synthesis with no downloaded audio files** - This protects bundle budget and avoids canned audio.

- **Mood-shaped pitch and personality-shaped timing** - These make calls feel responsive to the bird's state while keeping the motif recognizable.

- **Listen-in engage and disengage ramps** - Smooth 200-500ms ramps avoid hard cuts and preserve calm transitions.

- **Captions generated from call grammar** - Captions describe calls as naturalist prose such as "a soft three-note rise," aligning audio accessibility with product voice.

### Accessibility Surfaces

- **Slow screen-reader narration cadence** - Updates every 30-60 seconds at idle because "high-frequency narration would overwhelm screen reader."

- **User-initiated narration priority bump** - Return greetings and offer reactions get priority so direct interactions are noticed.

- **Soft high-contrast focus indicators** - They must read against both bright and dim aviary states while remaining visually gentle.

### Performance Budgets and Observability

- **Synthetic monitoring** - Automated browsers measure load, first-bird render, frame timing, audio errors, and simulation latency across schedules and geographies.

- **Aggregate-only real user monitoring** - RUM measures performance and errors without a per-account dimension or relationship-reconstructing telemetry.

- **Simulation-tick p99 alarm above 5 seconds** - The alarm "catches degradation early, before users notice."

- **Last two major browser versions only** - The plan rejects very old browser paths because "bundle bloat not justified."

### Rollout

- **Single account per deployment** - This restates "one canonical aviary per account" and keeps v1 aligned with server-side simulation and canonical sync.

- **No bird-count ramp** - The cap is "empirical, not arbitrary," so the plan does not require a ramp beyond starting with two and capping at seven.

- **Initial single region with CDN**: NOT RECOVERABLE FROM PLAN

- **Later multi-region and failover sync** - Multi-region happens "if needed"; cross-region sync is "only for failover (rare)."

### Risks, Success Criteria, and Design System

- **Drift calibration testing** - It mitigates the risk of breaking the "feels alive over weeks" promise.

- **Audio uncanniness user testing** - It asks whether calls sound like "happening now" or "playing back," directly targeting the "break spell" risk.

- **Accessibility audit and test harness** - These prevent accessibility features from degrading product quality and enforce narration, reduced motion, captions, and contrast.

- **Code-splitting and lazy loading** - These keep account settings, accessibility settings, and visit invitations from harming first render or the 2MB bundle budget.

- **Affective success over engagement metrics** - Users should return because they "want to see what the birds are doing," not because of a streak counter.

- **System fonts** - The plan uses system fonts with "no webfont load," supporting performance and avoiding extra asset cost.

- **Absent saturated UI accent colors** - Bright reds and electric blues are "deliberately absent" to preserve the calm, naturalist aesthetic.

- **Lowercase, present-tense naturalist product voice** - This keeps product surfaces specific and naturalist, while system surfaces remain direct and matter-of-fact.

### Open Questions

- **Exact tick cadence configurability**: NOT RECOVERABLE FROM PLAN

- **Presence activity window exact value**: NOT RECOVERABLE FROM PLAN

- **Call caption pre-generation versus runtime generation** - Runtime is "more flexible but costs CPU," so the plan articulates the tradeoff but not a final choice.

- **Reduced-motion cross-fade duration** - The plan states the tradeoff: "Too long feels sluggish, too short defeats the purpose."

- **Notebook entry trigger rule**: NOT RECOVERABLE FROM PLAN

- **Ambient weather frequency exact probability**: NOT RECOVERABLE FROM PLAN

- **Bird adoption pacing for a third bird**: NOT RECOVERABLE FROM PLAN
