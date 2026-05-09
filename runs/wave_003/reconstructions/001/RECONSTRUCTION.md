## System-level intent

- Gentle, non-gamified companionship. This shows up in the v1 exclusions of "gamification of any kind," "Tamagotchi-style mechanics," and "social-network surfaces," and again in the slow drift targets, sparse field notebook, "No spinner. No 'Loading...' text," and the "No toast" risk framed as the "notice, never announce" principle.

- Server authority over lived state. The plan repeatedly says the server is the "sole canonical writer," "all canonical state" is server-owned, and "No client-side state is ever authoritative" for personality or mood. This governs the Simulation Service, append-only events, snapshots, multi-device sync, and visitor read-only access.

- Slow time over action pressure. Aviary age, daily cadence, weekly drift, weather "a few times per week," notebook entries "one entry per few days," and bird unlocks by "aviary age (not interaction count, not paid tier)" all make progress time-based instead of count-based.

- Hidden numerical systems, visible expressive outputs. The personality vector exists canonically, but "is never returned to the client in raw form"; snapshots expose "rendered outputs" such as perch zone, mood, call parameters, and plumage level. The risk language warns against birds "feeling like a stat game."

- Accessibility must preserve affective quality. Screen-reader narration uses "naturalist prose"; reduced-motion is "not a stripped fallback" and uses cross-fades that "preserve presence without vestibular risk"; captions are generated from the call grammar; accessibility audit findings are blockers if they affect the "affective quality of the accessible surface."

- Performance is part of the emotional surface. The plan ties the first frame, quiet-field fallback, "<500ms" time to first bird, "60fps idle," and "No memory growth over 30 minutes" to making birds appear immediately, mid-motion, and without disruptive loading surfaces.

- Procedural aliveness without recorded media. Calls are WebAudio-only, generated from motif seeds and grammar rules. The plan wants the same bird's calls "recognizable," no two consecutive calls "identical," and alpha listening tests for whether a bird sounds alive.

- Tunable simulation rather than hardcoded behavior. Mood transitions use a "weighted probability lookup table, not a formula"; learning rates, weather cadence, notebook min gap, and unlock schedule are config artifacts so calibration can happen "without code changes" or "without code deploy."

- Read-mostly, low-friction sync. The client renders locally with "no round-trip per frame," receives snapshots and SSE diffs, posts events and "forgets," and uses ETags so dormant clients can refresh with a cheap 304 when nothing changed.

- Privacy boundaries are product boundaries. Internal references use synthetic UUIDs, email is encrypted and stored once, RUM is anonymized, telemetry is forbidden from joining on account UUID, and the plan deliberately does not measure per-account visit frequency, per-account interaction patterns, or per-bird drift rates.

## Per-feature whys

### Scope and user-facing surfaces

- Browser-only: NOT RECOVERABLE FROM PLAN

- Single-user accounts: NOT RECOVERABLE FROM PLAN

- Magic-link sign-in only: NOT RECOVERABLE FROM PLAN

- One aviary per account: NOT RECOVERABLE FROM PLAN

- New account starts with two birds: the first-sign-in adoption surface presents the two starters as "the birds that arrived" and is "brief by design" because it is the only step before the aviary first renders.

- Seven-bird cap: the cap lets the server enforce bird count, supports a fixed audio node pool sized around the max bird count, and makes the age-based unlock schedule the bird-count ramp.

- Server-side simulation tick: the tick is the sole canonical writer so personality, mood, presence accumulation, notebook generation, weather, and snapshots can be consistent across sessions and devices.

- Procedural call synthesis via WebAudio: calls are synthesized client-side from snapshot `call_params` with "no further server contact," no recorded files, deterministic seeds, and captions generated from the same commands.

- Mood system: the small enumerated state is designed to express time of day, recent interaction, weather, personality modulation, and bird-to-bird contagion while remaining tunable through a JSON table.

- Personality vector: slow drift makes birds change over weeks, with "measurable drift" after about one week and "visible drift" after about three weeks; monotonic clamping means traits "do not decrease on neglect."

- Return-greeting: NOT RECOVERABLE FROM PLAN

- Listen-in: the focused bird's gain ramps up while other birds ramp down, and listen-in events nudge alertness plus social warmth and vocal frequency, making attention audible without a separate mode.

- Offer interaction: offers nudge mood toward content and feed boldness, curiosity, and other drift signals; accepted offers are part of the slow expressive personality system.

- Settle interaction: settle warms the palette toward evening hues and ramps audio down, with undo available during the first five seconds so the ambient state can be reversed quickly.

- Field-notebook browsing: the notebook is read-only and paginated so users browse generated naturalist entries rather than authoring or managing content.

- Presence accounting with three-signal conjunction: visibility state, focus, and recent activity are combined to avoid inflated presence time that could corrupt personality drift.

- Auto-generated field notebook: entries come from state snapshots and noteworthy transitions, use "naturalist field-notebook text," and are kept sparse through a min-gap rule to preserve the quiet cadence.

- Day/night cycle anchored to the user's local timezone: local time drives mood buckets, light quality, sky gradient, and night/settled behavior so the aviary reflects the user's day.

- Ambient weather: rain and wind provide occasional visual and audio variation and also nudge mood and vocal-frequency during active weather ticks.

- Multi-device sync: sync is an "emergent property" of one canonical bird record, server-only personality writes, append-only events, and clients pulling the same snapshots.

- Visit invitations: visits are read-only ambient views so a host can invite by email without creating social-network surfaces or allowing visitor sessions to write to the event log.

- Visit invitations off by default: NOT RECOVERABLE FROM PLAN

- Screen-reader narration: narration uses naturalist prose from the same snapshot as the visual renderer, with slow idle cadence and prompt updates for user-initiated events.

- Reduced-motion mode: cross-fades replace frame-by-frame interpolation to preserve presence while reducing vestibular risk, and the setting applies immediately.

- Call captions: captions come from the same call grammar as synthesis, are forced on when WebAudio is unavailable, and are excluded from ARIA live regions to avoid duplicate narration.

- WCAG AA contrast: all user-copy text, including captions and notebook prose, is validated so readable text remains accessible across aviary palette states.

- Full keyboard navigation: the aviary is reachable as one application region, arrows cycle birds, Enter/Escape manage listen-in, Space opens offers, and a canvas focus indicator keeps focus accurate under responsive scaling.

- Account export: export gives users an on-demand JSON copy of account and bird state, and the risk section names it as a mitigation if a personality vector is lost or corrupted.

- Account deletion with 30-day soft delete: NOT RECOVERABLE FROM PLAN

- Synthetic UUID and encrypted email storage: internal account references use UUIDs only, email is encrypted and stored once, and email hash is used only for uniqueness checks, keeping email out of logs and external services.

- New-bird unlocking by aviary age: age-based unlocking makes time the ramp, explicitly avoiding interaction count and paid tier pressure.

### Architecture, data, and API

- Internal message bus: async operations flow through the bus so services such as simulation, notebook generation, and notification dispatch can remain separated.

- Aviary API Service as the only client-facing reader: the client never calls the Simulation Service directly, preserving the server boundary around canonical simulation state.

- Append-only interaction event log: clients POST events and forget; the simulation tick consumes server-timestamped events in order, avoiding client overwrite conflicts.

- Presence events as append-only records: presence is accumulated as events with start, end, and duration rather than mutable client state, so the tick can consume it canonically.

- State snapshot: snapshots expose rendered state, not raw personality values, letting the client render mood, perch, call parameters, weather, and notebook count without owning simulation state.

- Notebook write endpoints omitted: "No write endpoints" keeps the notebook server-generated rather than user-authored.

- Magic-link token storage: tokens are cryptographically random, bcrypt-hashed, 15-minute, and one-use to make email auth short-scoped.

- Snapshot SSE endpoint: SSE is chosen because delivery is read-mostly, HTTP/2-friendly, and no bidirectional communication is needed.

- ETag caching for snapshots: clients can revalidate after dormancy or keepalive, and unchanged state returns a single 304 round-trip.

- Visitor token endpoints without account auth: the token itself is sufficient and short-scoped, while visitor sessions remain read-only and never write interaction events.

- Account restore endpoint: NOT RECOVERABLE FROM PLAN

- Email-change endpoint: NOT RECOVERABLE FROM PLAN

### Simulation and sync behavior

- Tick runner cadence and stagger: ticks run about once per minute and are staggered to avoid a thundering herd on the database.

- Account simulation lock: each tick locks the account's simulation record so a single tick updates mood, drift, weather, notebook, and snapshot state coherently.

- Mood lookup table rather than formula: a versioned JSON table can be tuned during calibration without code changes.

- Mood persistence across ticks and sessions: mood does not reset on tab open, preserving continuity in the aviary's state.

- Personality drift low-pass filter: the low learning rate makes the system insensitive to short-term event ordering and keeps change gradual.

- Monotonic clamping: deltas are never negative so neglect does not produce visible decay.

- Drift calibration tests: automated tests confirm trait movement after simulated regular visits and no regression after absence.

- Call grammar runtime: species motif libraries, timing rules, and combination rules make calls recognizable, variable, and mood/personality-sensitive.

- Deterministic motif seed updates: the client can re-derive calls and captions deterministically while avoiding identical consecutive calls.

- Bird-to-bird response windows: response timing in the snapshot lets the client synthesize answering calls after another bird finishes, creating overlap or stagger without server round-trips.

- Weather state machine: stochastic transitions target roughly two to three weather events per week, with finite rain and wind durations and mood/audio/visual effects.

- Conflict avoidance instead of conflict resolution: only the server tick writes personality vectors, and append-only events with server timestamps remove personality overwrite conflicts.

- Dormant-client freshness pull: a client suspended or hidden for more than five minutes pulls a fresh snapshot on next visibility so local rendering rejoins canonical state.

- Visit revocation on next snapshot pull: revoked visitor tokens return a 403 and a matter-of-fact "visit no longer available" surface.

### Frontend rendering and audio

- React or Preact with canvas/WebGL aviary scene: DOM is used for top bar, notebook, and settings while canvas controls animation-loop performance and scene rendering.

- Canvas 2D fallback: NOT RECOVERABLE FROM PLAN

- Three scene rendering layers: NOT RECOVERABLE FROM PLAN

- Mood-keyed bird animation rigs: per-species keyframes let mood and actions appear as smooth motion rather than raw state changes.

- First frame strategy with inline snapshot: embedding the initial snapshot in the HTML response gives one round-trip to first bird and lets birds appear before later requests complete.

- Random idle-cycle frame offset: birds start mid-motion so no bird begins from a visible "start" pose.

- Quiet-field loading state: when no snapshot is available, the client shows soft sky and small ambient cues with no spinner or "Loading..." text, preserving the quiet field.

- Top bar fade: inactive chrome drops to low opacity after four seconds, returns on pointer or keyboard activity, and stays fully visible when keyboard focus is inside it.

- Idle micro-motion loop pause on hidden tab: rendering stops when `visibilityState` is hidden because no rendering is needed while the server simulation continues.

- Perch-to-perch fly arc: SSE perch-zone changes trigger a 300-500 ms animation so canonical state changes are rendered as bird movement.

- Listen-in mix: per-bird GainNodes make the focus bird louder and other birds quieter, then restore the default mix when listen-in ends.

- WebAudio-only synthesis chain: motif generation, oscillator, filter, envelope, per-bird gain, and master gain create calls from parameters rather than files.

- Audio node pooling: oscillator and gain nodes are reused to uphold the "no memory growth" contract.

- Compact motif library: motifs are parameter objects instead of audio files, keeping the audio grammar small in the bundle.

- Caption generation from synthesis commands: captions are produced synchronously before the call begins and require no additional server state.

- Chorus mixing through per-bird gain architecture: no special chorus mixer is needed because independent schedulers can naturally overlap or stagger calls.

- WebAudio failure behavior: the aviary renders silently, captions are forced on, and no recorded-audio fallback is attempted.

- Audio context activation on first natural interaction: browser gesture requirements are met by the first pointer or keyboard event, without a special "click to enable audio" surface.

### Accessibility, performance, rollout, and risk controls

- ARIA live narration regions: a polite idle region avoids over-announcing, while an assertive region gives timely updates for return-greeting, offer reaction, and settle.

- Reduced-motion CSS adjustments: top-bar and lighting transition durations are halved, ambient leaf/feather drift is suppressed, and audio is unaffected.

- Captions with semi-transparent chips: caption text is positioned near the bird and uses a background that enforces WCAG AA contrast.

- Caption exclusion from ARIA live regions: this prevents duplicate narration for users who use screen readers and captions.

- Bundle budget below 2MB gzipped: core scene, call synthesis, and presence code stay initial, while settings, notebook, visit, and adoption flows lazy-load.

- Initial assets under tight size limits: compact bird assets and inlined motif parameters keep first load within the bundle budget.

- Time to first bird visible below 500ms: inline snapshots, early canvas placement, and synthetic checks protect the first-bird target.

- 60fps idle budget: requestAnimationFrame, Web Worker offload, and no DOM mutations during the animation loop preserve a 16.7 ms frame budget.

- No memory growth over 30 minutes: fixed pools, virtualization, snapshot diff merging, and old snapshot discard keep heap size bounded.

- Synthetic performance measurement: Lighthouse CI and custom Playwright assertions catch FCP, TBT, bundle, and first-bird regressions.

- RUM measurement: anonymized timings and error counts observe experience quality without per-account or per-bird fields.

- Simulation p99 alerting: p99 tick latency over five seconds triggers an alert because slow simulation would affect canonical state freshness.

- Telemetry privacy boundary: telemetry cannot join on account UUID or read the simulation database, enforced by IAM policy and quarterly audit.

- Deliberately unmeasured per-account and per-bird patterns: the plan avoids metrics that would become per-bird interaction history.

- Internal alpha with accelerated aviary age: acceleration validates drift calibration and all seven species before public launch.

- Accessibility audit before open beta: screen-reader and reduced-motion findings block launch if they harm the accessible surface's affective quality.

- Launching accessibility features at v1.0: narration, reduced motion, and captions ship as core surfaces rather than later fixes.

- No artificial bird-count ramp: the unlock schedule itself is the ramp, and the seven-bird cap is enforced server-side.

- No feature flags on core surfaces: core surfaces launch directly, while deferred-load surfaces are handled by code-splitting.

- Bird-unlock schedule config flag: the age schedule can be adjusted post-launch without code deploy.

- Drift calibration mitigation: learning rates live in config and automated tests validate named targets so birds do not feel like a stat game or fail to change for months.

- Multi-device concurrency mitigation: server-assigned timestamps and low learning rate make unusual short-term ordering acceptable.

- Audio uncanniness mitigation: motif libraries are validated by listening, including a "does Pip sound alive?" evaluation and recognizability tests.

- Accessibility regression mitigation: axe-core, snapshot fixtures, and focus-indicator contrast checks run against representative aviary states.

- Presence-signal inflation mitigation: the three-signal module has clear contracts, browser compatibility tests, and aggregate anomaly review.

- No-toast discipline: code review checks interaction and session-start changes for notification surfaces that violate "notice, never announce."

- Performance regression mitigation: bundle-size and first-bird-render checks fail CI when the thresholds are crossed.

- Personality vector loss mitigation: daily backups, before-and-after audit logs, a soft-corruption canary, and account export protect against birds "forgetting" users.

- Edge inline snapshot: the edge worker embeds the initial snapshot to remove the separate API call before rendering.

- Notebook service generation as a tick side effect: entries are generated from noteworthy state transitions, with a min-gap rule to preserve sparsity.

- SVG visual assets with CSS saturation: CSS `saturate()` renders plumage drift without separate asset variants, keeping the initial bundle within budget.

- Notebook LLM or template choice: either option is acceptable if it honors the naturalist voice; an LLM receives only event type and bird names, not personality vectors or numeric drift values.
