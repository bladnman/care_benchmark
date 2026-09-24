## System-level intent

- A single, private, stable-identity aviary is the core product shape. This shows up in "one web aviary per magic-link account," "two stable-identity starter birds," "one aviary per active account," "never regenerate an existing bird on migration," and migration rules that "preserve every bird UUID, trait, mood and call signature."

- The experience should feel like quiet naturalist observation rather than a game or dashboard. The plan names "lowercase, present-tense naturalist prose" for observations, narration, captions and offers; rejects "scores, streaks, badges, levels, hunger, death, distress"; says "no textual return welcome" and "no numeric personality surface"; and requires notebook templates that reject "generic status strings, trait numbers and statements about the user's visit frequency."

- Canonical bird life belongs on the server, while the client renders and synthesizes from snapshots. This appears in "database is the sole canonical store," "server-side tick workers," "clients own rendering, procedural synthesis and local interaction state only," and "they never advance the canonical simulation or submit trait values."

- Privacy boundaries are product design, not only storage design. The plan repeats "private/no-store," "verified session," encrypted email fields, "forbid email in partition keys, general logs, trace attributes and analytics," a separate metrics pipeline with "only allowlisted aggregate counters," and visitor snapshots with "no private notebook, account data or owner-only command capability."

- Presence is a validated input for slow character drift, not a public engagement counter. The plan requires visibility, focus and recent input; says "never infer presence from an open tab, a playing audio context or a visitor"; clamps and deduplicates intervals; and states this is "validation for drift integrity, not a public engagement counter."

- Evolution should be gradual, bounded and non-punitive. The trait rules use smoothed exposure, per-day caps and representative regular attendance; "presence drives all expressive traits and carries most weight"; "never subtract from persisted traits for absence or negative mood"; and a two-week absence should read "as quiet without reversing personality or creating distress."

- Recognizable individual identity should persist through mood, time, visits and migrations. The plan anchors identity in stable UUIDs, immutable call-signature seeds, species motif identity, "pitch contour and rhythmic fingerprint," and the rule that mood changes "never the recognizable core motif."

- The scene should start as an aviary, not as app chrome or a loading product. The plan calls for a low-latency edge snapshot, "paint the quiet field immediately" if needed, a "deterministic mid-action phase," "first bird visible within 500 ms," and says if that target fails to reduce critical-path code and snapshot latency before considering a loading animation.

- Accessibility must be an equivalent aviary experience, not a reduced status page. The plan requires "complete accessible render/audio alternatives," reduced-motion rendering that keeps "mood, calls, notebook and drift" intact, captions generated from the actual scheduled motif, naturalist screen-reader narration from the same snapshot/event model, and keyboard access to birds, offers and settle/undo.

- Visits are intentionally read-only and revocable. The visitor projection returns the same bird/day/weather projection but "no private notebook, account data or owner-only command capability"; visitor presence "is never sent to the simulation ledger"; and snapshots recheck grants so revocation is visible "on the next pull."

- Operational rollout is gated by identity, privacy, accessibility, simulation integrity and performance. This appears in vertical slices gated on "stable bird IDs and no client personality writes," automated checks for idempotency, presence, drift and visits, release gates on device/network lab traces, and staged rollout with rollback paths for client bundles and tick algorithm versions.

- Calibration decisions must remain within the anti-gamification and non-punitive product vocabulary. The plan leaves presence window, tick cadence, trait coefficients, offer cooldown, adoption intervals, palette tokens and top-bar settle placement for build calibration, but says none may be tuned by adding "scores, notifications, visible trait values or punitive absence behavior."

## Per-feature whys

**Product contract and scope**

- One web aviary per magic-link account: NOT RECOVERABLE FROM PLAN

- Two stable-identity starter birds from a coherent pool of about six species: stable identity is protected so birds are not regenerated or reset; the plan later requires migrations to "preserve every bird UUID, trait, mood and call signature."

- Owner name or later rename birds: NOT RECOVERABLE FROM PLAN

- Age-gated invitations to adopt additional birds: aviary age alone controls eligibility because the plan says "never gate new birds on engagement." Timezone and clock-change rules also prevent extra adoption eligibility.

- Hard cap of seven birds: the cap is tied to visibility and recognition constraints; rendering must keep "all seven birds visible on narrow phones and wide desktops," and audio is validated at "two through seven birds."

- Presence-sensitive evolution: presence is the main validated exposure signal for slow expressive change, with acceptance targets of "one-week instrument-detectable movement" and "approximately three-week user-perceptible change."

- Mood and bird-to-bird behavior: the plan connects mood and bird responses to a living scene, with mood shaping greeting probability, call activity, perches, weather effects and "occasional chorus and propagated wary/alert reactions." A two-week absence should read as "quiet" rather than distress.

- Procedural calls: the plan rejects "recorded call loops" and "loop fallback" so calls can preserve individual fingerprints while varying rhythm, pitch, pauses and timbre.

- Listen-in: the plan's rationale is to foreground one bird without erasing the aviary; listen-in ramps the target bird up, ramps others down and keeps "non-target birds" at an "audible floor."

- Seed, song-fragment and still-pool offers: offers create bounded interaction reactions and small exposure only once per cooldown, so repeated offers cannot cause visible jumps.

- Settle: settle ends presence, changes short-term mood and overlays slower evening lighting/call attenuation; it has "no special drift reward," keeping it out of engagement optimization.

- Settle undo: a click reverses settle and emits idempotent undo, keeping the settle action reversible inside a bounded server-checked window.

- Five-second undo window: NOT RECOVERABLE FROM PLAN

- Sparse read-only field notebook: entries are sparse naturalist observations from noteworthy state transitions, not status strings or visit-frequency feedback; they support indefinite scrolling until account deletion.

- Local-time lighting: the plan uses stored IANA timezone plus server time for daylight and adoption age, so lighting follows the owner zone and clock changes cannot create extra adoption eligibility or drift.

- Weather mood/audio effects: weather provides temporary mood/audio effects with a seeded per-aviary pattern.

- Rare weather frequency: NOT RECOVERABLE FROM PLAN

- Opt-in read-only visits: visits let a guest see the canonical bird/day/weather projection while protecting owner commands, private notebook, account data and simulation presence.

- Account controls: NOT RECOVERABLE FROM PLAN

- Export: export is an "owner-only data portability surface" and is kept separate so vectors are never presented as a stats panel.

- Deletion: deletion supports privacy and account control by marking deletion immediately, allowing recovery within 30 days, then hard purging dependent records, event history, visit data, outbox references, exports and account-linked telemetry.

- Complete accessible render/audio alternatives: accessibility is required so captions, notebook, focus and controls are available to a screen-reader user, and reduced motion preserves the same poses/events.

- Single horizontal scene: all birds must remain visible across narrow phones and wide desktops in one responsive, unscrollable scene.

- Three perch zones: NOT RECOVERABLE FROM PLAN

- Sparse top bar: the sparse four-icon bar protects the uncluttered scene; settle placement must preserve the "sparse four-icon bar," and faded chrome must remain legible when focused.

- Lowercase, present-tense naturalist prose for observations, narration, captions and offer copy: this carries the product voice of naturalist observation rather than system status.

- Direct matter-of-fact copy for identity, settings, errors and account flows: system and account surfaces should be clear, not narrated as bird behavior; rejected offers or stale commands get "a clear, quiet system error, not an invented bird response."

- No textual return welcome: the plan uses bird greeting choreography instead and says to never show a "welcome toast" or "absence tally."

- No visit announcement: NOT RECOVERABLE FROM PLAN

- No participation metric or numeric personality surface: the plan keeps presence from becoming a public engagement counter and keeps raw personality numbers out of the ordinary UI.

- Rejections of native clients, additional aviaries, collaborative ownership, public discovery, profiles, comments, chats, feeds, rankings, payments, scores, streaks, badges, levels, hunger, death, distress, mandatory goodbye, scene customization and recorded call loops: the articulated rationale is to preserve the non-gamified, private, non-social, non-punitive aviary, with a product-design acceptance review for exclusions and hidden analytics/settings surfaces.

**System boundaries**

- Small web client, authenticated HTTP API, transactional primary database and server-side tick workers: this keeps simulation consistency in one canonical store and prevents clients from advancing canonical state.

- Database as sole canonical store for birds, personality vectors, moods, positions, simulation clocks, account settings and notebook entries: rationale is canonical consistency across devices and ticks.

- Append-only account-scoped interaction ledger: commands and validated presence intervals are carried into the tick in order; later checks emphasize idempotency and simultaneous-device ordering.

- Transactional outbox for invite/export mail: mail delivery is explicitly "not part of simulation consistency," separating email side effects from bird state.

- Separate operational metrics pipeline: privacy rationale; it receives only allowlisted aggregate counters, timings and errors and has no simulation database or event payload access.

- Low-latency edge path for shell and authenticated initial snapshot: supports fast first frame and snapshot delivery while keeping personal snapshots private/no-store and session-keyed.

- Quiet field if initial snapshot is not ready: prevents the first frame from appearing as app loading; the scene can enter at the snapshot's simulated phase.

- Empty field followed by soft first-bird arrival reserved for first adoption only: NOT RECOVERABLE FROM PLAN

- Code split settings, notebook history and invitation flows: supports the critical path to first bird by keeping non-scene flows out of initial load.

- Client owns rendering, procedural synthesis and local interaction state only: prevents client personality/tick writes and preserves canonical simulation integrity.

- Versioned snapshot schema shared by client and service: enables schema upgrades and rendering/audio contracts across service and client.

- Snapshot includes server time, state version, next tick, timezone/day phase, weather, per-bird identity, render-safe appearance, mood, perch/pose, call schedule seed and event cursor: these are the canonical and render-safe inputs needed for scene/audio without exposing raw trait vectors.

- Derived behavior parameters are bounded and server validated: lets client render/audio behavior without accepting raw or untrusted trait state.

- Separate owner-only account export including vectors: supports portability while ensuring vectors are "never presented as a stats panel."

**Persistence and invariants**

- Synthetic UUID account, aviary and bird identifiers: privacy and auditability; email is forbidden in partition keys, logs, traces and analytics.

- Encrypted email only in account and invitation recipient fields: privacy boundary for account and invitation records.

- Account record with timezone, notification preference, accessibility settings and one aviary per active account: NOT RECOVERABLE FROM PLAN

- Separately revocable sessions with opaque token hash and device label: supports per-device session revocation.

- Single-use 15-minute magic links: supports atomically consumed auth and replay prevention.

- Aviary record with simulation tick cursor/version, weather/day state, last owner-presence end and adoption eligibility cursor: supports canonical tick progress, local-time/weather state, presence accounting and age-gated adoption.

- Bird record with stable UUID, species, name, adopted time, five server-only normalized trait values, mood, perch/pose, immutable call-signature seed and version: preserves individual identity, server-only personality and migration continuity.

- Interaction event record with idempotency key and validated bounded payload: supports append-only command handling, retry safety and tick consumption without double applying.

- Presence interval record from bounded qualifying heartbeats, deduplicated across devices: prevents simultaneous owner tabs from multiplying time.

- Notebook entry with event/time anchors, authored naturalist text and generator version: keeps notebook observations immutable, specific and paginated indefinitely.

- Invitation/visit record with token hash, expiry, revocation state, scoped visit-session hash and approximate duration: supports one-time redemption, bounded visit sessions and host-visible logs without exposing email outside encrypted recipient field.

- Export/deletion job with secure artifact pointer or purge cursor: supports owner export and deletion while keeping email out of job records and making purge resumable.

- Point-in-time recovery and private restore sample: protects canonical records and verifies backup restore.

- Schema migrations preserving bird UUID, trait, mood and call signature: prevents identity loss or reset across upgrades/downgrades.

- Hard deletion within documented 30-day schedule: implements account deletion by removing dependent records, event history, visit data, outbox references, exports and account-linked telemetry.

- Deletion jobs resumable and auditable by synthetic ID only: supports reliable purge without email-based audit trails.

**API and command contracts**

- HTTPS JSON endpoints with schema version, request ID and bounded payload size: supports versioned contracts, request tracing and bounded input.

- CSRF-protected cookie sessions with secure/httpOnly/sameSite attributes: protects authenticated browser sessions.

- Authorization by owner UUID or scoped visitor token: enforces owner/visitor boundaries for every lookup.

- Mutations with idempotency keys: allows safe retry and returns accepted event IDs or current account/version.

- Refusal of client-supplied personality, mood, perch or tick state: protects canonical simulation from client writes.

- `POST /auth/link`, `POST /auth/consume`, `POST /auth/logout`: generic request responses prevent email enumeration; rate limits and single-use 15-minute consumption prevent abuse/replay.

- `GET /aviary/snapshot?after=<version>`: gives owner snapshot or no-change response with server time, ETag/version and refresh points after load, visibility restoration, suspension-sized frame gap and keepalive.

- `POST /aviary/events`: appends validated listen, offer, settle, undo and presence events while server checks ownership, cooldowns, library keys and undo deadlines.

- `GET /notebook?cursor=...`: supports reverse chronological reads and historical scrolling over immutable entries.

- `PATCH /birds/{id}`: NOT RECOVERABLE FROM PLAN

- `POST /adoptions`: server-side aviary age and count transaction enforce age-only eligibility and cap under concurrency.

- `GET/PATCH /account`: NOT RECOVERABLE FROM PLAN

- `GET/DELETE /sessions/{id}`: supports device-session management and revocation.

- `POST /account/export`: emails a time-limited export link to the verified address for data portability.

- `POST /account/delete` and `POST /account/restore`: mark deletion immediately, allow recovery within 30 days, then hard purge.

- `POST /invitations`, `GET /invitations`, `DELETE /invitations/{id}`: issue explicitly addressed one-time invitations, list outstanding/active grants and revoke access.

- `POST /visits/redeem`, `GET /visits/snapshot`, `POST /visits/end`, `GET /visits/log`: redeem a token into scoped read-only visit session, recheck grant each snapshot, end visit and show host on-demand log.

- Visitor endpoint returning same canonical bird/day/weather projection as host: lets visitors observe the aviary while excluding private notebook, account data and owner commands.

- Visitor presence excluded from simulation ledger: prevents visitors from changing owner bird evolution.

- Single-use redeemed link with continuing bounded visit session: resolves the "one-time-link/active-visit distinction."

- Frequent visit snapshots and bounded pull interval: revocation becomes visible on the next pull.

- Visit notifications only if host explicitly opts in, never in onboarding: prevents visit notifications from becoming a default/onboarding push.

- Offline commands are not speculative trait writes: preserves canonical state; local visual/audio reaction occurs only after server acceptance.

- Bounded retry with same idempotency key for transient failures: supports retry without double-applying events.

- Expired auth stops event submission and refreshes snapshot after reauthentication: avoids unauthorized event writes and stale state.

- Rejected offer or stale command gets clear quiet system error: avoids inventing bird responses for system failures.

**Presence and simulation engine**

- Presence detector requiring visible state, focus and recent pointer/key activity: keeps drift from background tabs and qualifies actual attendance.

- Monotonic local clock segmentation and interval close on hidden/blur/settle/unload: bounds presence intervals to real foreground use.

- Periodic bounded heartbeats and best-effort `sendBeacon`: lets the server close stale intervals from heartbeat expiry.

- Idle-activity timeout configurable and tested with people who watch without moving: balances drift integrity with quiet watching behavior.

- Server-clamped and deduplicated presence intervals: prevents inflated drift from client clocks and simultaneous tabs.

- Roughly one-minute tick for every active aviary, including no connected client: keeps simulation running independent of browser sessions.

- Partitioned tick jobs by aviary UUID, per-aviary locking/versioning and idempotence on `(aviary_id, tick_time)`: prevents double-application and cross-device divergence.

- Tick transaction reading events, accruing presence, applying trait deltas, evolving moods/perches/weather/calls and notebook observations: ensures ordered, atomic simulation updates.

- Outage replay in order with bounded batch processing: prevents jumping to a freshly initialized state after downtime.

- Tick latency and backlog alerting: protects the simulation cadence, with p99 over five seconds and backlog age as alarms.

- Normalized internal trait values `[0,1]`: NOT RECOVERABLE FROM PLAN

- Stable individual baseline and immutable species motif identity: preserves recognizable bird individuality.

- Smoothed exposure accumulators and additive trait update with per-day caps: produces gradual drift and prevents one long session or repeated offers from visible jumps.

- Presence weighted above listen-in and offers: presence is the main signal for expressive traits, while interactions add modest bounded exposure.

- Settle has no special drift reward: prevents settle from becoming an engagement reward action.

- No subtraction from persisted traits for absence or negative mood: avoids punitive absence and distress.

- Greeting probability and call activity soften through mood/recency: makes a long absence read as quiet without reversing personality.

- Trait coefficients calibrated by simulation harnesses: targets one-week instrument-detectable movement and three-week user-perceptible change.

- Mood state machine with wary, content, curious, drowsy and alert: supports short-term expressive state shaped by trait vector, owner interaction, local time and ambient events.

- Persisted mood and timer across sessions: prevents mood from resetting on page open and supports daily-ish equilibrium movement.

- Seasonal/timezone-independent weather random seed and rare rain/wind: creates repeatable ambient events per aviary with temporary mood/audio effects.

- Stored IANA timezone detected then changeable: supports daylight and adoption age based on server time plus owner zone.

- Safe timezone and daylight-saving behavior: clock changes must not create extra adoption eligibility or drift.

- Immutable per-bird call identity from species motif library and seed: preserves each bird's recognizable core motif across mood changes.

- Seeded call scheduler with responses, chorus and propagated wary/alert reactions: supports bird-to-bird audio behavior with logical call timings in snapshots.

- Client-side waveform synthesis with no per-sample audio persistence: keeps snapshots compact and avoids recorded loops.

- Server-enforced offer cooldown and mood/personality-shaped reaction branches: prevents offer spamming from producing visible jumps while preserving varied reactions.

- Notebook entries from noteworthy transitions with sparsity budget: keeps notebook sparse and specific rather than status-like.

- Deterministic reviewed naturalist templates: binds birds, times, perch/weather/call observations while rejecting generic status, trait numbers and visit-frequency statements.

- Deduplication by underlying observation: prevents repeated notebook entries for the same event.

**Session choreography and scene rendering**

- Current snapshot and absence duration on navigation or return: lets greeting and scene phase reflect server state after absence.

- Exactly one greeting bird selected by weighted draw: creates a return greeting shaped by boldness, warmth, mood and elapsed absence without welcome toast or absence tally.

- Procedural glance/head tilt/step/call greeting pattern and optional staggered second response: gives bird behavior instead of textual return copy.

- Brief absence as glance and long absence as deliberate call/reorientation: lets recency change behavior without distress or metrics.

- Session-local debounce on rapid visibility flaps: prevents duplicate greetings.

- Prompt screen-reader greeting narration: makes the greeting accessible.

- One responsive, unscrollable scene with sky/foliage, perches, birds and foreground ornaments: creates the primary aviary surface rather than page navigation.

- Normalized coordinates and safe bounds for all seven birds: keeps birds visible across narrow phones and wide desktops.

- Compact SVG or procedural primitive birds: supports compact rendering assets and species/palette variation.

- Pose and low-rate idle actions selected by identity, mood and personality: makes stable identity and state visible without showing trait numbers.

- Deterministic phase offsets so first frame is mid-action: avoids a static or loading-like first frame.

- Client interpolation of timestamped transitions only: smooths authoritative snapshots without writing back to canonical state.

- Fresh snapshot after resumed visibility or long frame gap: avoids stale teleportation.

- Stop drawing while hidden while server continues ticking: saves client work without stopping canonical simulation.

- Gradual local-time color transitions from server-time anchor and timezone: aligns scene lighting with owner local time.

- Settle overlay with evening lighting/call attenuation until re-engagement or close: creates a quieter state tied to settle.

- Undo click within five seconds and normal presence after that window: preserves bounded undo while treating later activity as a new presence interval.

- Leaf and feather drift as render-only ornaments: keeps decorative motion out of tick events.

- Top-bar icons for account/settings, accessibility, notebook and offer: NOT RECOVERABLE FROM PLAN

- Settle as reachable top-bar action without fifth always-visible icon if possible: preserves the sparse four-icon bar while keeping settle reachable.

- Top-bar fade after pointer stillness and restore on input: keeps chrome sparse while preserving legibility for focused controls.

- Standard motion mode with subtle transforms and slow parallax at 60 fps: supports idle motion performance.

- Reduced-motion mode with same poses/events, cross-fades, no leaf drift and slower palette shifts: reduces motion while preserving mood, calls, notebook and drift.

- `prefers-reduced-motion` detection and explicit override: respects system preference and user accessibility settings without restarting.

**Audio and accessible equivalents**

- AudioWorklet/WebAudio procedural synthesis from species motif grammars: creates calls from compact grammars rather than recorded loops.

- Voice/buffer reuse, concurrent-call cap, smoothed envelopes and peak limits: protects performance and avoids clicks or excessive output.

- Audible fingerprint preserved across mood changes: keeps bird identity recognizable.

- Gentle ambient chorus: NOT RECOVERABLE FROM PLAN

- Listen-in mix ramp and reversal: focuses the target bird over a perceptible interval while retaining non-target audible floor.

- Offer song fragment entering same mixer: keeps offer audio in the same procedural audio system.

- Listening tests at two through seven birds: validates polyphony, recognizability, overlapping calls and nightjar-like nighttime activity.

- Autoplay handling by rendering scene and scheduled calls immediately: avoids blocking the aviary on audio permission.

- Audio context resume on first permitted gesture preserving call phase: avoids replaying a canned welcome.

- WebAudio failure fallback to silence with call captions on by default: keeps accessible call information when sound is unavailable.

- Explicit mute and caption settings: NOT RECOVERABLE FROM PLAN

- Captions generated from actual scheduled motif, contour, timing and mood: ensures text and sound correspond.

- Short captions near the bird without obscuring other birds: makes captions spatially useful while preserving visibility.

- Accessible text equivalents independent of transient visual positioning: supports screen-reader access regardless of layout changes.

- Screen-reader narration from same snapshot/event model: keeps narration coherent with the actual scene.

- One observation every 30-60 seconds at idle and prompt observations after greeting, offer and settle: balances specificity with sustained listening comfort.

- Queue coalescing for narration: prevents periodic updates from interrupting user-initiated speech or flooding live regions.

- Meaningful bird names/species, perch and audible behavior with no trait numbers/raw labels: preserves naturalist voice and avoids stats exposure.

- Logical tab order and first bird focus on entering scene: makes the scene navigable by keyboard.

- Arrow keys between birds, Enter listen-in, Escape exits, focus leaving disengages: supports keyboard bird interaction.

- Keyboard-operable offer menu and settle/undo: gives full control access without pointer.

- Visible focus rings legible in bright and dim scenes: maintains focus visibility across lighting states.

- WCAG AA contrast for copy surfaces including captions and focused faded chrome: ensures readability.

- Touch target size, zoom/reflow, no focus traps and live-region behavior checks: verifies accessible operation across inputs and display settings.

- Clear system-language labels in account and accessibility settings: keeps settings direct rather than naturalist prose.

**Delivery, measurement and acceptance**

- Vertical slices from auth/two-bird first frame through operational hardening: structures delivery so core account, simulation, rendering, offers, accessibility, visits and deletion are built incrementally.

- Accessibility prototypes and voice review beginning with first slice: prevents accessibility and product voice from being late-stage add-ons.

- Every slice gated on stable bird IDs and no client personality writes: protects identity and canonical state throughout delivery.

- Automated checks for idempotency, simultaneous devices, presence exclusion/deduplication, tick replay, bounded drift, mood persistence, cooldowns, age/count, visits, magic links, deletion and schema upgrades: verifies the main invariants named in the plan.

- Seeded multi-week simulation cases: tunes drift across regular, absent, high-offer, all-night-open and multi-device patterns to the one-week/three-week targets.

- Blinded listening studies: validate call identity, repetition and chorus quality.

- User studies of naturalist narration and reduced motion: validate sustained listening comfort and motion alternatives.

- Assertions of no gamification copy or frequency metrics on public surfaces: enforces the anti-gamification product boundary.

- Initial compressed JS under 2 MB: supports the fast first-bird path.

- First bird visible within 500 ms on mid-tier mobile over 4G: prevents first frame from appearing as app loading.

- 60 fps idle motion on a five-year-old mid-range laptop for 30 minutes: supports standard-mode scene smoothness over time.

- No sustained memory growth over 30 minutes: protects long idle sessions.

- Measurement of navigation-to-first-bird including auth/snapshot delivery: ensures the metric covers the real returning-session path.

- If 500 ms fails, reduce critical-path code and snapshot latency before loading animation: keeps the product intent on immediate aviary rendering rather than app loading.

- Aggregate-only Real User Monitoring and operational telemetry: measures performance and errors while rejecting account IDs, emails, bird IDs, event payloads and per-bird state.

- No per-account engagement dashboards, average bird drift or population interaction analyses: prevents observability from becoming engagement analytics.

- Short-retention transient request correlation IDs with no email: supports debugging without email in logs.

- Alarms for tick p99, backlog, snapshot errors, auth failures and audio failures: protects simulation, access and audio reliability.

- Private invariant checks for canonical state integrity: monitors state without exporting vectors to analytics.

- Rollout from internal accounts to opt-in cohort to staged general availability: reduces risk while reviewing performance, tick integrity, privacy and accessibility gates.

- Start with two birds and enable later age-based invitations behind a server flag: verifies multi-voice recognizability and mobile performance at three, five and seven before broad adoption.

- Seven-bird cap enforced regardless of flags: preserves database/service validation independent of rollout state.

- Visit invitation ramp after read-only authorization and revoke tests pass: prevents visitor control leaks or stale access.

- Rollback for client bundles and tick algorithm versions: allows recovery without resetting birds.

- Data migrations forward-safe and never resetting birds: preserves stable identity and state during rollout.

- Instrument first-bird timing, tick p99/backlog, frame/memory, audio failures and auth/visit errors from day one: provides launch telemetry for the acceptance gates.

**Risks and decisions to close during build**

- Drift too fast, too slow or inflated by background tabs: mitigated by conjunctive presence detector, server bounds/deduplication, per-day caps, simulations and longitudinal observation; one session should produce no perceptible change, regular visits show drift around week one and visible change around week three.

- Canonical state lost or overwritten across devices: mitigated by single server writer, ordered append-only events, atomic tick cursor/version, idempotent retries, restore drills and migration tests.

- Procedural calls sound synthetic, repetitive or indistinguishable: mitigated by immutable motif identity, varied grammar/envelopes, acoustic reviews at two to seven birds, chorus/listen-in tests and no loop fallback.

- Accessibility turns aviary into status messages or a static page: mitigated by naturalist narration/caption review, cross-fade render mode, screen-reader and vestibular user tests and keyboard/contrast gates.

- First frame appears as app loading: mitigated by small edge snapshot, critical-path render, deterministic mid-action phase and quiet-field timeout.

- Visitor access leaks owner controls or survives revocation: mitigated by separate visitor projection, no event endpoint scope, token redemption/recheck each pull and revocation integration tests.

- Privacy data escapes into observability: mitigated by UUID-only references, encrypted email only on account/invitation records, metric allowlists, pipeline separation and automated payload audits.

- Precise presence activity window: decision left for calibration against acceptance tests; it may not be tuned by adding scores, notifications, visible trait values or punitive absence behavior.

- Tick cadence near one minute: decision left for calibration against acceptance tests; it may not be tuned by adding scores, notifications, visible trait values or punitive absence behavior.

- Trait coefficients and cap: decision left for calibration against acceptance tests; it may not be tuned by adding scores, notifications, visible trait values or punitive absence behavior.

- Offer cooldown within a few minutes: decision left for calibration against acceptance tests; it may not be tuned by adding scores, notifications, visible trait values or punitive absence behavior.

- Adoption age intervals: decision left for calibration against acceptance tests; it may not be tuned by adding scores, notifications, visible trait values or punitive absence behavior.

- Palette/contrast tokens: decision left for calibration against acceptance tests; it may not be tuned by adding scores, notifications, visible trait values or punitive absence behavior.

- Top-bar settle placement: decision left for the design specification and usability test, preserving the sparse four-icon bar and the same calibration constraints.
