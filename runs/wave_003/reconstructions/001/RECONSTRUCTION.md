## System-level intent

- Felt continuity and restraint over feature volume. The plan says to optimize for "felt continuity, individual bird recognizability, privacy, accessibility, and restraint rather than feature volume." This appears in the single-user, single-aviary boundary, the server-side simulation clock, the "already in motion" first frame, sparse notebook entries, slow drift calibration, and the long exclusion list for social, gamified, paid, and native surfaces.

- Canonical server continuity. The plan's core invariant is that "Clients never mutate personality, mood, bird positions, or notebook entries directly." This shows up in the three-surface architecture, the simulation service as "the only writer of canonical aviary state," append-only interaction events, canonical snapshots, multi-device sync, and tests for overlapping sessions and no last-write-wins behavior.

- Individual birds should remain recognizable. The plan repeatedly protects "individual bird recognizability" through stable bird identity, stable `call_signature_seed`, stable `visual_seed`, editable names that do not change identity, a small species pool, the seven-bird cap because "the chorus risks becoming undifferentiated," and the recognition requirement that a bird's call signature remains recognizable across mood and personality drift.

- Quiet, non-gamified product behavior. The plan excludes "leaderboards, rankings, badges, achievements, streaks, levels, XP, scores," forbids "welcome back" and similar strings, says APIs must maintain "the product's quietness," and frames offer effects as "gestures, not rewards." It also says the visit log is "account-settings transparency, not a notification feed."

- Split product voice: naturalist prose for aviary surfaces, matter-of-fact prose for system surfaces. The plan says aviary, notebook, captions, narration, and offer reactions use "lowercase, present-tense, specific naturalist prose," while sign-in, account, error, conflict, revocation, unsupported-browser, privacy, export, deletion, and accessibility settings use "matter-of-fact system prose."

- Privacy and data minimization are architectural requirements. The plan states "Security and privacy are architectural requirements, not policy text alone." This appears in encrypted email on the account record only, synthetic UUIDs throughout, aggregate operational telemetry only, no per-bird data in analytics, no ML/model training on per-bird interaction state, hard deletion, private export, and isolated metrics.

- Accessibility is part of the same product, not a fallback. The plan says accessibility "must ship in v1" and "must be built from the same canonical state as the default experience." It also warns against "Accessibility Regression Into Fallback Product" and requires reduced-motion mode to remain "alive and designed."

- Presence means actual host attention, not an open tab. The plan says presence is the "dominant drift input" and "'Tab open' alone never counts." The client detector requires visible, focused, recently active conditions, and the server clamps and deduplicates pings to prevent runaway timers.

- The first frame should feel like the aviary was already running. The definition of done says "The first visible frame feels like the aviary was already running." This shows up in edge/bootstrap snapshots, the 500ms first-bird target, "no spinner," no "loading aviary" text inside the scene, and lazy loading of settings, notebook, visit, export, and deletion flows.

## Per-feature whys

### Product Boundary and Scope

- **Browser-only modern client:** NOT RECOVERABLE FROM PLAN

- **Email magic-link accounts:** The plan later ties magic links to neutral request responses "to avoid account enumeration," 15-minute expiry, single use, hashed tokens, and rate limiting by hashed email.

- **One canonical aviary per account:** The rationale is continuity: a user should return on another device "to the same canonical aviary," and the product is explicitly "single-user, single-aviary."

- **Two starter birds and slow growth up to seven birds:** Two birds are fixed at start, account-age-based additions are "a post-launch v1 ramp, not onboarding," and seven is the cap because beyond that "the chorus risks becoming undifferentiated."

- **Stable bird identity and user-editable names:** The plan ties this to "individual bird recognizability"; renaming a bird must not change "identity, mood, species, call seed, or personality."

- **Species from a small coherent pool:** NOT RECOVERABLE FROM PLAN

- **Hidden personality vectors:** The plan wants expressive behavior without "numerical personality/stat surfaces"; personality values must not appear in user APIs, ARIA labels, notebook text, product UI, debug panels, or telemetry.

- **Persistent moods:** Mood persistence supports continuity: mood "does not reset to neutral on tab open" and "persists across sessions."

- **Procedural calls with stable call signatures:** The rationale is recognition and avoiding uncanny repetition: each bird has a stable call signature seed, motifs vary by mood/personality, and calls should not feel "looped, harsh, repetitive, or blurred in chorus."

- **Idle micro-motion and bird-to-bird interaction:** The plan says "Birds never read as paused" and describes bird-to-bird calls and nearby moods as inputs to the living simulation.

- **Return-greeting:** "There is no textual welcome on return. The bird greeting is the welcome surface." The greeting should be brief, bird-led, and never every-session notebook material.

- **Listen-in:** The plan uses listen-in to focus on an individual bird while keeping the aviary alive: the focused bird rises in the mix and others lower to ambient, "never to silence."

- **Offer:** Offer effects should appear "in-scene as gestures, not rewards"; cooldowns "prevent button-mashing," and copy should avoid punitive language.

- **Settle:** Settle is for a slow evening lighting transition and quieter calls. It ends presence at the engine level, has a five-second undo for accidents, and closing the tab without settle is "never criticized."

- **Field notebook:** The notebook should be "read-only, sparse," and observational rather than a generic event log; it uses naturalist prose and avoids user-behavior frequency such as "you visited every day."

- **Accessibility surfaces:** The plan says accessibility "must ship in v1" and be built from canonical state, with narration, reduced motion, captions, keyboard navigation, contrast, and graceful silence if WebAudio is unavailable.

- **Optional read-only visit invitations:** Visits are read-only so visitor sessions do not trigger greeting, presence, drift, offers, notebook writes, or host notification by default.

- **Aggregate operational telemetry:** The plan allows performance, latency, errors, frame timing, audio failures, and tick health because telemetry is for operations, while disallowing anything that can reconstruct a user's relationship with the aviary.

- **Explicit exclusions for native apps, social surfaces, public discovery, co-presence, payments, push, gamification, hunger, death, and distress:** The rationale is the product's "privacy, accessibility, and restraint rather than feature volume" boundary and the non-goal tests preserving quiet, non-gamified behavior.

### Architecture Overview and Data Model

- **Product API boundary:** NOT RECOVERABLE FROM PLAN

- **Simulation service/worker as only writer:** This protects canonical state and sync: clients submit events, and the simulation tick consumes ordered events and writes personality, mood, positions, notebook entries, and transition state.

- **Static client assets from CDN/edge and bootstrap snapshots:** The rationale is first-bird performance and low-latency first paint; the decision log prefers edge-delivered bootstrap snapshots because first-bird timing should not wait for multiple API calls.

- **Background simulation scheduler with sharding and slight tick jitter:** The scheduler maintains server-side continuity, and jitter avoids "thundering herd behavior"; sharding uses synthetic account UUIDs.

- **Relational database for canonical records:** NOT RECOVERABLE FROM PLAN

- **Append-only event table or durable queue-backed event log:** The plan uses this for ordered client interaction events so ticks can consume events, mark them with `simulation_tick_id`, and avoid last-write-wins drift loss.

- **Object storage or generated-on-demand signed URL for account export JSON:** The rationale is private data portability through a signed expiring link rather than a product loop.

- **Metrics pipeline isolated from per-bird/per-account simulation records:** The rationale is the privacy boundary: operational telemetry must not become bird interaction analytics or per-account state dashboards.

- **Synthetic UUIDs and encrypted email:** The plan says to use synthetic UUIDs in storage, logs, queues, metrics tags, and URLs, and to store email only on the account record, encrypted, "never as an identifier."

- **Hashed session tokens:** Raw session tokens are never stored, supporting the security requirement that sessions are per-device and revocable.

- **Magic-link hashed tokens, hashed email rate limit, 15-minute expiry, and single use:** These support neutral sign-in, rate limiting, expiry, immediate invalidation, and avoiding enumeration.

- **Bird personality hidden from product surfaces:** The rationale is to keep personality expressive but not numeric or stat-like; the plan forbids exposing raw values through render APIs, ARIA labels, settings, notebook text, debug panels, or telemetry.

- **Raw personality values in account export only:** The plan says this exception exists because the PRD explicitly names them, and the export must remain a private account-settings portability artifact.

- **Static species catalog with no rarity field:** NOT RECOVERABLE FROM PLAN

- **Host-only presence pings:** Presence pings are accepted only from authenticated host sessions because visitor sessions "never enter drift calculations."

- **Visitor events limited to snapshot pulls and invite/session lifecycle events:** The rationale is that visitors remain read-only and do not shape the host simulation.

- **Server-side offer cooldown validation:** The plan uses per-bird cooldowns to prevent button-mashing and validates offer attempts server-side.

- **Listen-in duration from start/end events with server receipt bounds:** The rationale is "to prevent runaway durations."

- **Compact canonical snapshots:** Snapshots are "small and optimized for first paint," omit personality raw values, and include only derived render parameters sufficient to draw birds and synthesize calls.

- **Notebook entries as sparse read-only observations:** The plan says they are "never generic event logs" and should avoid user-behavior frequency, keeping the notebook focused on aviary observations.

- **Visit log entries in account settings:** The visit log is for "account-settings transparency, not a notification feed."

### API Surface

- **Structured API errors:** Errors should let clients render matter-of-fact system prose and avoid gamified summaries, streak counts, trait numbers, or engagement rankings.

- **Neutral magic-link request response:** The response is always neutral "to avoid account enumeration."

- **Magic-link consume response including whether starter adoption/naming is needed:** NOT RECOVERABLE FROM PLAN

- **Account/settings endpoint:** NOT RECOVERABLE FROM PLAN

- **Email change requiring new email verification:** The plan requires verification before switching, matching the account/security voice.

- **Account export endpoint:** Export generates a JSON snapshot and emails a signed download link to the verified address as a private data portability artifact.

- **Account deletion and cancellation:** Deletion starts a 30-day soft-delete window, cancellation restores inside the window, and hard deletion later removes account, birds, vectors, notebook, telemetry linkage, invites, visit logs, and interaction records.

- **Session revocation endpoint:** The rationale is per-device session control from settings, with matter-of-fact revocation surfaces.

- **Aviary snapshot endpoint:** It returns canonical render state, server time, day phase, weather, calls, settled state, and affordances while not returning personality raw values.

- **Aviary bootstrap endpoint:** It exists to render the first bird within 500ms on mid-tier mobile over 4G.

- **Batched event ingestion endpoint:** The endpoint validates schema, session ownership, timestamps, cooldowns, and idempotency keys to keep client interactions bounded and canonical.

- **Notebook endpoint:** The rationale is to return read-only notebook entries rather than editable notes or generic logs.

- **Bird rename endpoint:** It allows naming without changing identity, mood, species, call seed, or personality.

- **Offers discoverability endpoint:** The plan says the client needs discoverability, but available offers and cooldowns should avoid "rewards or effects" language and use naturalist prompts client-side.

- **Visitor consume and visitor snapshot endpoints:** Visitor snapshots return the host aviary "as-is," do not trigger host simulation effects, and revoked/expired visits return a matter-of-fact "visit no longer available" state.

### Simulation Engine

- **Deterministic but varied simulation engine:** The engine must be "deterministic enough to test and varied enough to feel alive."

- **Server-side tick and no client tick:** The rationale is canonical sync: do not tick on the client, do not recompute personality from history on demand, and do not let multiple devices produce divergent simulations.

- **Slight tick cadence jitter:** The plan says jitter can avoid "thundering herd behavior."

- **Transactional tick updates with processed markers:** Tick writes state and marks consumed events with `simulation_tick_id`, supporting ordered event consumption and preventing sync loss.

- **Simulation tick p99 latency alarm at 5 seconds:** The rationale is tick health as an operational budget.

- **Presence detector requiring visible, focused, recently active conditions:** Presence is the "dominant drift input," and "'Tab open' alone never counts."

- **Server presence dedupe, idempotency, and caps:** The rationale is to prevent runaway timers and background tabs from inflating drift.

- **Drift as a slow low-pass filter over daily/weekly aggregates:** This keeps personality movement slow: instrument-detectable after about one week, user-visible after about three weeks, and no single session produces visible trait movement.

- **No negative personality drift on neglect:** The plan says neglect never decreases personality values and less recent presence becomes "quieter ambient expression without negative personality drift."

- **Listen-in drift weighting toward social warmth and vocal frequency:** NOT RECOVERABLE FROM PLAN

- **Offer drift weighting toward boldness and curiosity:** NOT RECOVERABLE FROM PLAN

- **Settle drift behavior:** Settle is "mood-quieting only" with no meaningful drift vector beyond ending presence.

- **Golden, property, and regression simulation tests:** These enforce one-day, one-week, and three-week calibration, neglect monotonicity, multi-device sync, and no visitor drift.

- **Mood system as fast-timescale persistent state:** Mood persists across sessions and advances by tick, supporting living continuity rather than tab-open resets.

- **Mood inputs from time, weather, bird interactions, nearby moods, and personality:** The rationale is specific expressive behavior: dusk trends drowsy/settled, rain dampens calls, wind can raise alert/wary probability, and traits shape greeting, calls, and offers.

- **Return-greeting selection with at most one primary greeting bird and possible staggered secondary response:** The rationale is a quiet bird-led welcome in the first one to two seconds rather than a textual announcement.

- **Recent greeting history in return-greeting:** It avoids "identical repeated patterns."

- **No toast, banner, welcome back, absence counter, or textual return message:** The bird greeting is the welcome surface, and product review forbids welcome/gamification language.

- **Procedural call grammar with stable seeds:** The rationale is recognizable individual calls with mood and personality variation, while avoiding recorded loops and chorus blur.

- **Seven-bird cap in the chorus:** The plan states the cap exists because beyond seven "the chorus risks becoming undifferentiated."

- **Listen-in audio mix ramp:** The plan raises the focused bird and lowers others to ambient "never to silence," then ramps back slowly.

- **WebAudio unavailable path:** If WebAudio is unavailable, the product plays in silence with captions on by default.

- **No recorded-audio fallback:** The audio-risk mitigation says "No recorded loops," and the call fallback explicitly says "No recorded-audio fallback."

- **Rule-based notebook observation composer:** The decision log chooses this over generative AI because specificity and privacy matter, per-bird data must not be used for model training or broad analytics, and a deterministic composer is easier to audit, tune, and keep sparse.

- **Notebook candidate triggers and sparsity:** Entries appear only for notable simulation events such as unusual greeting order, long quiet stretch, weather behavior, rare offer reaction, unusual perch time, or bird-to-bird response; regular use should produce about one entry every few days.

- **Notebook content rules:** The rationale is naturalist observation rather than metrics or user judgment; entries use names and observed behavior, never "achievement," "streak," "visited," "score," "level," or frequency deltas.

### Frontend Rendering Pipeline and Interaction UI

- **Small initial JS bundle and code splitting:** The rationale is the 2MB gzipped budget, 500ms first bird, and not blocking first bird on settings, notebook, visit log, export, deletion, or full species assets.

- **Rendering layer separated from simulation logic:** The client renders snapshot-derived state and local interpolation only, preserving the server as the canonical simulation.

- **Prototype validation for Canvas/WebGL/SVG choices:** The rationale is to meet 2MB, 500ms first bird, 60fps idle, and reduced-motion needs while avoiding heavy engines unless they fit the budget.

- **Bootstrap first rendered state as the aviary, with no spinner or visual loading text:** The rationale is the "already in motion" first frame and quiet field experience.

- **Horizontal scene with no panning, scrolling, zooming, or user-controlled camera:** The plan requires birds to remain visible at supported viewport sizes and avoid cropping.

- **Top bar above the scene and fade behavior:** NOT RECOVERABLE FROM PLAN

- **Day/night transitions by account timezone:** The rationale is a living scene with gradual morning, midday, evening, and night phases tied to the account timezone.

- **Rare, subtle weather:** Weather should be rendered from server state, locally ornamented, and have small simulation-handled mood/audio effects without dominating the product.

- **Client-side ambient leaves/feathers drift:** The rationale is that "Birds never read as paused," while these local ornaments do not require canonical state.

- **Bird renderer with species silhouette, palette, visual seed, mood pose, and personality-derived render hints:** This supports stable visual identity and expressive posture/plumage without exposing numbers.

- **No labels, hover tooltips, mood badges, status icons, or inline UI inside the aviary:** The rationale is to avoid raw state labels, visible stat surfaces, and game-like UI in the aviary.

- **Listen-in controls by pointer/tap or keyboard focus plus Enter, with Escape/focus-away disengage:** This makes listen-in accessible, reversible, and subtle without a "selected" label.

- **Offer menu for seed, song fragment, and still pool:** Offer effects stay in-scene as gestures, cooldowns prevent button-mashing, and copy avoids punitive or reward language.

- **Settle with five-second undo:** The undo exists for accidental settle actions.

- **Notebook UI from top-bar icon:** The notebook is read-only, scrollable, sparse, naturalist, and has no edit/delete/annotation controls.

- **Settings/account/accessibility flows:** They use matter-of-fact voice, are keyboard complete, and manage sessions, export, deletion, email change, privacy policy, visit invites/log, and accessibility preferences.

### Accessibility Plan

- **Screen-reader narration composer:** It converts canonical snapshot state into "naturalist prose" so screen-reader users receive the same aviary state through a designed surface.

- **Narration cadence and slow queue:** Idle narration every 30-60 seconds and a slow queue avoid overwhelming screen readers.

- **No raw narration labels such as mood or perch numbers:** The rationale is specific prose rather than technical state labels like "mood: content" or "perch 2."

- **Reduced-motion renderer:** The plan says this is "not a static fallback"; it uses designed still poses, cross-fades, slower transitions, and keeps calls, captions, notebook, mood changes, drift, and field observations.

- **Call captions:** Captions are generated from actual procedural call grammar, appear near the calling bird, pass WCAG AA contrast, and activate when audio is off, WebAudio is unavailable, the user opts in, or audio failure is detected.

- **Keyboard flow:** Keyboard support lets users complete top bar navigation, aviary focus, listen-in, offers, settle, settings, notebook, visit management, export, deletion, and accessibility settings.

- **Unsupported browser page and avoidance of compatibility shims:** Unsupported browsers receive a matter-of-fact page, and the plan avoids shims that inflate the initial bundle and compromise first-bird performance.

### Performance, Observability, Privacy, Rollout, and Risk Controls

- **Performance budgets:** The budgets protect the first-bird experience, idle motion, memory stability, tick health, and small snapshot payloads.

- **Lazy-loading settings, notebook, visit flows, export/deletion, and historical notebook pagination:** The rationale is that these flows should not block the first bird.

- **Stopping rendering when the tab is hidden and resuming with a fresh snapshot:** This saves client work while the simulation continues server-side and restores canonical state on return.

- **Allowed aggregate metrics:** Request counts, latency, errors, snapshot delivery, first-bird timing, frame timing, WebAudio errors, caption fallback counts, tick latency, and anonymous session-duration histograms are allowed because they measure product health.

- **Disallowed analytics:** Per-bird interaction analytics, per-account dashboards, average drift, social ranking metrics, and anything reconstructing a user's relationship with the aviary are disallowed for privacy.

- **Synthetic checks:** Automated browser checks, mobile 4G emulation, 30-minute memory runs, reduced-motion smoke tests, WebAudio fallback tests, and keyboard walkthroughs catch performance and accessibility regressions.

- **No ML/model training on per-bird interaction state:** The rationale is privacy boundary protection.

- **Export kept quiet in account settings:** The plan says not to market export "as a feature loop."

- **Private beta bird-cap ramp from two to three to five to seven:** Bird additions happen after engine validation and in cohorts, not as onboarding or visit-count rewards.

- **Aggregate monitoring during ramp:** The plan monitors aggregate performance, tick health, support issues, accessibility regressions, and audio fallback rates while not instrumenting per-bird engagement analytics.

- **Forbidden-surface audit for user-visible strings and routes:** The audit guards against welcome, streak, score, achievement, badge, level, XP, leaderboard, ranking, hunger, death, sad-because-you-left, public profile, follow, comments, discovery, and adopted-counter surfaces.

- **Drift coefficient feature flags and internal visual/audio diff tools:** These mitigate drift that feels too fast like "stats that move on click" or too slow like "screensavers that never change," while keeping numbers away from users.

- **Presence corruption tests:** Synthetic sessions that leave tabs open without focus verify no presence accrual, protecting the core promise from background-tab inflation.

- **Sync overlap tests:** Ordered events, additive server deltas, transactional ticks, and processed markers mitigate multiple devices overwriting personality state.

- **Audio listening tests:** Listening tests for two, three, five, and seven birds mitigate calls that feel looped, harsh, repetitive, or blurred in chorus.

- **Accessibility milestones and QA reviews:** Accessibility work is included in core milestones and launch acceptance so reduced-motion or screen-reader users do not get a flattened product.

- **Privacy static checks and data access review:** Static checks for PII in metrics/log fields and review of simulation tables mitigate per-bird interaction data leaking into analytics or ML pipelines.

- **Performance CI and early render/audio prototypes:** These mitigate bundle or rendering weight that would compromise the "already in motion" first frame.

- **Visit notifications off by default:** The decision log says they remain an account setting, "never onboarding, never push by default, and never represented as a badge on the main aviary."
