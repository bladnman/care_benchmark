## System-level intent

- **Feels alive before the host does anything.** The plan repeatedly treats aliveness as something that must be visible immediately: "the first rendered bird is already mid-action," the "single horizontal scene shows birds already occupied with their own lives," and "the first frame is the server-rendered ongoing pose." The final risk response makes this explicit: "feels alive" is a "system property."
- **Continuity is canonical, cross-device, and session-spanning.** The plan's experience tests require "two devices show one continuous aviary," while architecture says the worker changes canonical state and clients interpolate snapshots. The timezone decision says "Never independently calculate two different canonical mornings on two devices."
- **Restraint over game pressure.** The plan rejects "leaderboards, achievements, scores, levels, streaks, visit-frequency displays," and says "absence never harms a bird." Adoption intervals are "never a progress bar, visit reward, score, notification, or countdown."
- **Gestures are invitations, not survival mechanics.** The product boundary says "An offer is a gesture, never food required for survival." Drift and settle repeat this ethic: "no neglect or settle event subtracts a trait" and settle has "no independent drift reward or penalty."
- **Server-owned life, browser-owned presentation.** Architecture assigns personality, mood, perch intent, weather, call schedule, and notebook facts to the worker alone; the browser "owns only local presentation state." API contracts reinforce this with "Never accept client-written vectors, mood, or absolute perch."
- **Hidden personality, qualitative surface.** The plan forbids personality numbers in the aviary, notebook, settings, captions, narration, or visitor view. Snapshot payloads carry qualitative state and "do not contain numerical personality values"; export is a "narrow data-portability exception."
- **Privacy boundaries are part of the product, not just operations.** The model says never use email as a foreign key, shard key, telemetry dimension, or log identifier. Telemetry is "aggregate-only" and has no feed that can read "per-bird events/vectors/notebook."
- **Product voice is naturalist for the aviary and direct for systems.** The product contract says copy is "lowercase, present-tense, bird-specific naturalist prose," while sign-in, errors, sync, account, and accessibility settings use "direct, matter-of-fact language." Notebook and narration are "observations," not labels like "mood changed."
- **Equal access must preserve aliveness.** Accessibility is named "a full product surface"; reduced motion must "not merely freeze the normal scene." Captions come from actual motif parameters, narration comes from the same snapshot/facts, and equal-access work must not be deferred after launch.
- **Identity is durable through names, migrations, restore, and rollout.** The plan preserves immutable bird UUIDs, call-signature seeds, grammar versions, backups, and checksums. It says "A rename never changes identity or seed" and rollout flags must contain regressions "without rewriting existing bird identities."
- **Performance is an experience requirement.** The first-bird path is separated from deferred code so a bird appears before hydration/audio, with a 500 ms first-bird gate and no spinner. The risk table frames missing that mark as "First bird feels like a loading transition."
- **Calibration uses fixtures and qualitative review, not engagement analytics.** Drift constants are fitted against "synthetic fixtures and consented qualitative observation," "never population interaction analytics." Release tests use "synthetic bird histories" so real per-bird data never enters analytics.

## Per-feature whys

### 1. Product contract and delivery boundary

- **One private, browser-based aviary per account**: The plan frames the core product as private continuity: exactly one aviary, cross-device continuity, private bootstrapped snapshots, and no public discovery or extra aviaries.
- **A new account meets two server-selected birds and names them**: The plan wants starters to be "arrivals with suggested editable names, not a species catalog," so the initial experience is meeting birds rather than shopping species.
- **Eventually adopt up to seven birds as the aviary ages**: Age-based adoption avoids activity pressure; the plan says unlocks are by "Age, not activity" and are "never a progress bar, visit reward, score, notification, or countdown."
- **The single horizontal scene**: It supports the governing test that birds are "already occupied with their own lives" and avoids panning, scrolling, or zooming in rendering.
- **Watching**: NOT RECOVERABLE FROM PLAN
- **Listen-in**: The plan uses listen-in as a local attention gesture that changes mix immediately and logs bounded duration for later drift, while keeping other birds audible at an ambient floor.
- **Offers of seed, song fragment, or still pool**: Offers are framed as gestures, not survival food; the server chooses a reaction from mood/personality and rejections return cooldown without a "punitive message."
- **Settle**: Settle is a quieting gesture that ends presence, crossfades lighting/calls, and has no independent drift reward or penalty; it is reversible in the first five seconds.
- **Sparse, system-written field notebook**: The notebook is an "observation log, not a session/event dump," generated from canonical facts with sparse budgets and immutable prose.
- **Email magic-link accounts**: The plan uses opaque, short-lived, single-use tokens with rate limits and revocable per-device sessions, giving sign-in without stored bearer tokens.
- **Cross-device continuity**: The rationale is one continuous aviary: canonical timezone, revision checks, server timestamps, and no independently calculated mornings.
- **Optional read-only visits**: Visits allow viewing the same canonical scene while preserving host privacy and bird integrity: visitor paths are separate, read-only, and cannot trigger greetings, listen-in, or mutations.
- **Export**: Export is a data-portability exception that includes current vectors only in verified-email JSON while keeping numeric traits out of interactive UI and analytics.
- **Deletion and restore**: Deletion disables sessions, visits, invites, and ticks immediately while preserving a 30-day explicit restore window before hard purge and cryptographic key erasure.
- **Call captions**: Captions provide access when WebAudio is blocked, missing, or off, and they are generated from the actual realized motif so they agree with the call.
- **Naturalist screen-reader narration**: Narration keeps the same naturalist voice as the scene and uses snapshot/facts to describe concrete species, positions, light, sound, and notable behavior.
- **Designed reduced-motion scene**: The plan says reduced motion must be a "separate render mode" that preserves canonical mood, calls/captions, notebook, and drift, not a frozen normal scene.
- **Support for the last two major Chrome, Safari, Firefox, and Edge versions**: NOT RECOVERABLE FROM PLAN
- **Lowercase, present-tense, bird-specific naturalist prose**: The plan uses this voice to keep aviary copy observational and bird-specific, while keeping sign-in/errors/settings "direct, matter-of-fact."
- **No native clients, payments, extra aviaries, public discovery, social surfaces, scores, streaks, distress, death, or push reminders**: The plan's rationale is restraint: avoid game mechanics, social pressure, hunger/distress loops, and reminders that turn absence into obligation.
- **No personality numbers in product surfaces**: The plan protects the qualitative bird experience by hiding numeric traits from the aviary, notebook, bird settings, captions, narration, visitor view, and telemetry.
- **No textual return toast or banner**: The returning host should be noticed by a bird within 1-2 seconds "without textual welcome," preserving the bird-led greeting.

### Decisions that settle PRD ambiguities

- **Settle inside the offer icon's gesture menu**: The plan wants settle reachable from the top bar "without adding a fifth permanent icon or controls inside the scene."
- **Named account icon in the four-icon top bar**: NOT RECOVERABLE FROM PLAN
- **Accessibility icon in the four-icon top bar**: NOT RECOVERABLE FROM PLAN
- **Notebook icon in the four-icon top bar**: NOT RECOVERABLE FROM PLAN
- **Offer icon in the four-icon top bar**: Its rationale is partly structural: it opens the gesture menu containing the three offers and settle, avoiding a fifth permanent icon and scene controls.
- **Canonical `aviary_timezone`**: The rationale is cross-device consistency: all clients receive the same new zone and lighting phase, and the server avoids "two different canonical mornings."
- **Foreground timezone update and settings correction**: These let travel or correction change canonical local time through versioned account mutation while smoothing the visual transition.
- **Downloaded export containing current personality vectors**: The plan treats this as a "narrow data-portability exception" to the no-numeric-traits product rule.
- **Off-by-default visit notification email**: The rationale is an explicit, host-controlled exception to "no notifications," limited to one quiet email per visit and not push, badges, or reminders.
- **Browser audio policy**: The plan keeps the live visual aviary available when audio is suspended, enables captions, and resumes WebAudio on the next deliberate gesture instead of using recorded loops or a splash screen.

### 2. Architecture and ownership

- **Small web client, authenticated API, relational primary database, scheduled simulation worker**: The split protects canonical state by making the worker alone change personality, mood, perch intent, weather, call schedule, and notebook facts.
- **Append-only, per-aviary ordered events**: This gives the worker a reliable event stream and supports transactional ticks, idempotent replay, and no skipped late commits.
- **Browser rendering/interpolation of snapshots**: The browser can keep presentation responsive while not fabricating canonical drift or accepting client-written mood/perch/personality.
- **Separate read-only visitor capability path**: The rationale is isolation: visitors never open an interaction or presence channel.
- **First paint path distinct from settings, notebook, invite, and export code**: The plan wants first bird visible before hydration/audio and keeps noncritical surfaces from gating first paint.
- **Private, no-store bootstrapped state with initial HTML**: This enables personalized first paint while preventing shared CDN cache leakage.
- **Server-rendered initial SVG bird/perch geometry**: This makes a bird appear before hydration or audio initialization.
- **Embedded revision, server timestamp, active motion phase, and deterministic scene seed**: These let hydration continue the current pose rather than replaying an entry animation.
- **Quiet sky/field shell on slow network, with no spinner**: The plan avoids making the scene feel like a loading transition while waiting for the first state.
- **Only one-time adoption transition may begin from an empty field**: The rationale is that ordinary openings should already show life; emptiness is reserved for adoption arrival.
- **Primary-state transaction for every tick and event append**: This protects monotonic canonical state and ordered event consumption.
- **Read replica or private edge read-through cache only with monotonic revisions**: The rationale is to prevent older snapshots from replacing newer state.
- **Client keeps newer state when receiving an older revision**: This preserves continuity and avoids regression from stale reads.
- **Last observed scene on unavailable snapshot**: The plan says to keep the last observed scene and not "fabricate canonical drift locally."

### 3. Persistent model and privacy boundaries

- **UUIDs for account, aviary, bird, device session, notebook entry, invite, and visit**: The plan pairs this with never using email as a database key, telemetry dimension, or log identifier.
- **Encrypted account email plus HMAC lookup**: This supports sign-in lookup while avoiding plain email identifiers in database relations and logs.
- **Short-lived encrypted recipient address on pending magic-link and invite delivery records**: The plan allows addresses only for their "short transactional lifetime."
- **Token hashes instead of bearer tokens**: The rationale is security if storage is exposed; logs and records do not contain usable tokens.
- **Exactly one aviary per account**: NOT RECOVERABLE FROM PLAN
- **Email change pending until new-address verification**: The plan preserves the old active address until the new one is verified.
- **Device sessions with hashed rotating token and revocation checks**: Every request checks revocation so sessions can be listed, revoked, and disabled on deletion.
- **Aviary state with timezone, weather, presence end, expression factor, revision, event cursor, and logical tick**: These fields support canonical time, weather, presence-derived greeting, monotonic snapshots, and deterministic tick progress.
- **Bird immutable UUID and call-signature seed**: The rationale is persistent identity across rename, restore, migration, and procedural calls.
- **Five normalized personality scalars stored canonically**: They support calibrated drift while remaining hidden from product surfaces except verified export.
- **Only the tick database role may update personality**: This enforces server-owned life and prevents clients or ordinary API writes from changing vectors.
- **Rename changes only `name`**: The plan says a rename never changes identity or seed.
- **Interaction events include idempotency UUID, device sequence, validated payload, and consumed tick**: This prevents retries, duplicate devices, and late events from multiplying drift or being skipped.
- **Notebook entries immutable and pageable oldest-to-newest**: The rationale is preserving history as system-written observations rather than editable notes.
- **Invitation and visit records with no visitor presence or bird events**: This prevents visitors from affecting simulation or being treated as host attention.
- **Snapshot payloads omit numerical personality values, interaction history, account email, and hidden visitor data**: The client needs derived rendering parameters, never raw vectors.
- **Schema and grammar versions with migrations that do not regenerate IDs or seeds**: The rationale is continuity of bird identity and recognizability.
- **Backups and restore tests for vectors and seeds**: The plan requires proving existing birds retain identity and values after restore.
- **Separate operational telemetry infrastructure**: It protects the simulation database by accepting only counts, distributions, timings, and anonymous histograms.
- **No population drift, per-bird engagement, or cross-account interaction aggregates**: The rationale is avoiding analytics that erode the privacy boundary.
- **Consumed raw event retention then pruning**: Raw events remain only for tick recovery, while notebook and canonical vectors remain as product data.

### 4. API and command contracts

- **Versioned `/api/v1` API with secure cookies, CSRF, origin checks, schema validation, ownership checks, body limits, and idempotency**: The rationale is secure host mutation and predictable contracts.
- **Separate host and visitor authorization paths**: The plan says this separation is "by design" so visitors cannot mutate or open presence/listen-in channels.
- **Magic link request and consume endpoints**: Short expiry, atomic single use, rate limits, and direct errors prevent replay and make sign-in failure matter-of-fact.
- **Host snapshot endpoints with `ETag`/revision and low visible-tab polling**: These provide current server time, interpolation horizon, and convergence without client ticks.
- **Immediate pull on visibility return or long frame gap**: The rationale is to resume from canonical phase rather than advance hidden/local state.
- **Batched event endpoint**: It accepts bounded, validated events and returns exact accepted/rejected results so clients cannot write vectors, mood, or absolute perch.
- **Offer endpoint with cooldown and server-seeded reaction**: It prevents spam, validates song fragments, keeps reaction authoritative, and avoids punitive rejection copy.
- **Adoption and rename endpoints**: These enforce age eligibility, seven-bird cap, server-selected species, idempotent creation, and name validity.
- **Notebook read endpoint with no edit/delete**: The rationale is immutable system-written observation history.
- **Settings, session revocation, and email change endpoints**: These expose account control, timezone correction, preferences, session revocation, and verified email switching in system voice.
- **Export, delete, and restore endpoints**: They give consistent JSON export, 30-day soft deletion, and explicit restore during the window.
- **Invite and visit endpoints**: They keep invitations deliberate, per-email, revocable, and visible in settings without default prompts, badges, or alerts.
- **Visitor snapshot endpoints checking expiry/revocation on every request**: The rationale is revocation that terminates the view at the next pull and never grants host controls or notebook.
- **Visitor receives the same canonical scene, calls, time, and weather as host**: This preserves the shared scene while still denying mutation, greeting, listen-in, and notebook access.
- **Approximate visit duration from grant activity**: The plan records visits without treating them as host presence.

### 5. Server simulation and calibration

- **One logical tick per aviary per minute, including offline**: This keeps the aviary's life continuous whether or not a device is connected.
- **Per-aviary transactional worker claim and lock**: The rationale is consistent state, ordered event consumption, and atomic bird/aviary/notebook/cursor writes.
- **Same lock for event sequence assignment**: It ensures a late commit cannot be skipped by a tick watermark.
- **Unique `(aviary_id, logical_minute)` tick key**: This makes retries idempotent.
- **Replay missing logical minutes after worker outages**: The plan says never jump directly to a freshly recomputed bird, preserving continuity.
- **Tick latency alarms and due-tick lag alerts**: The rationale is catching simulation staleness from day one.
- **Stable stochastic seeds plus logical minute and grammar version**: This makes crash retries produce the same result and supports regression renders.
- **Persisted vectors, additive migrations, backups, and checksums**: The rationale is not rebuilding bird identity or drift from session logs.
- **Presence based on visible, focused, recent pointermove or keypress activity**: The plan wants honest attention; "simply leaving a tab open does not count."
- **Presence ends on blur, hide, settle, or pagehide with beacon and lease expiry**: This prevents unlimited client claims and stale presence.
- **Server validation of host intervals**: Monotonic device sequence, bounded length, plausible clock skew, and no continuation after settle/revocation prevent inflated presence.
- **Union overlapping presence across devices**: The rationale is that two open devices do not double-count attention.
- **Presence test matrix**: It verifies hidden/focused, visible/unfocused, inactive, suspend/resume, overlap, late beacons, mobile/touch, assistive input, and undercount risk.
- **Drift model with normalized traits and daily caps**: The rationale is week-measurable, three-week-perceptible, session-invisible change without repeated heartbeats/offers multiplying it.
- **Presence carrying at least 70% of relevant daily input**: This keeps drift from becoming a clicking reward.
- **Nonnegative, diminishing trait deltas**: The rationale is no neglect harm and no settle penalty.
- **Transient expression gain**: It resolves "quieter after two weeks" while never changing stored vectors downward or producing distress.
- **Settle ending presence without drift reward or penalty**: This prevents settle from becoming an optimization action.
- **Mood enum and seeded semi-Markov transitions**: Mood persists across sessions and changes after dwell time from time, offers, weather, nearby calls, wariness, and personality rather than resetting on open.
- **Perch and pose intent tied to mood/personality**: The rationale is visible qualitative difference: wary birds back-perch, bold birds front-perch, content birds preen, curious birds tilt, and drowsy birds fluff low.
- **Bird-to-bird response windows**: They create "real call-and-answer/chorus events, not independent loops."
- **Rare rain or wind windows**: These add persistent soft weather that temporarily affects mood and calls.
- **Local time lighting and chorus attenuation**: The rationale is gradual local day/night behavior, with night settling most birds while a nightjar-like species may remain active.
- **Return greeting cue**: It satisfies the governing test that a returning host is noticed within 1-2 seconds after an already-active first frame, without a text welcome.
- **No visitor greeting**: The rationale is that visitor access is read-only and not a host presence channel.
- **Notebook entries from eligible factual state changes and events**: They must validate canonical facts, avoid visit-frequency prose, reject repetition, and stay sparse because the notebook is an observation log.
- **Age-based adoption slots around days 90, 180, 270, 365, and 540**: The rationale is access by aviary age, not activity or visit frequency.
- **Coherent six-species pool without rarity tiers**: It balances distinct silhouettes/calls while avoiding rarity as a game mechanic.

### 6. Rendering and interaction pipeline

- **Lightweight SVG/DOM scene with compact vector species shapes**: The plan ties this to first paint, compositor-friendly animation, and avoiding expensive per-frame SVG filters.
- **Three normalized perch zones and safe layout boxes**: These keep birds visible across narrow and wide viewports without cropping, scrolling, panning, or zooming.
- **Subtle depth, parallax, leaves, feathers, and foreground elements**: These provide atmosphere while staying local, seeded, and visible-only.
- **Qualitative render tokens instead of raw vector values**: The rationale is hiding personality numbers while still rendering plumage richness and pose.
- **Snapshot interpolation using server time, transition times, pose phase, and deterministic horizon**: This lets newer revisions change intent without teleporting birds.
- **Fetch fresh snapshot after hidden tab or long frame gap**: The rationale is resuming at canonical current phase.
- **Scene controls kept out of the scene**: The plan treats call captions and focus indication as overlays, not navigation chrome, preserving the scene.
- **Top bar fades nearly transparent after pointer stillness**: The rationale is a quiet scene, while controls return on movement, keyboard activity, or focus and never fade while keyboard focus is inside them.
- **Offer menu navigability and placement**: It should not obscure birds unnecessarily and keeps settle in the top bar.
- **Listen-in start/stop interactions**: Click/tap/Enter, repeat, another bird, empty scene, Escape, and focus leaving define accessible, reversible control.
- **Offer reaction waits for authoritative API cue**: The plan prevents animating failed or noncanonical reactions.
- **Quiet system message on offer failure**: It keeps failure direct and leaves canonical state unchanged.
- **Settling crossfade with undo**: The rationale is a soft quieting that can be undone by scene click or keyboard action in the first five seconds.
- **Closing the tab and settling both terminate presence without penalty**: This reinforces absence/no-penalty and honest presence.
- **Reduced-motion crossfade composition**: It removes flight paths and drifting ornaments while keeping canonical mood, calls/captions, notebook, and drift intact.

### 7. Procedural audio and captions

- **Roughly six species motif libraries**: The plan wants distinct contours, rhythmic signatures, and timbres for species identity.
- **Stable bird signature seed**: This gives each bird a recognizable subset while allowing mood/personality variation within identity-preserving limits.
- **Server-supplied seeded call score and active-call phase**: It synchronizes calls to snapshots without replaying calls.
- **WebAudio oscillators/noise/envelopes/light filters instead of downloaded loops**: The rationale is procedural identity and avoiding recorded loops.
- **Bounded audio pools and capped simultaneous voices**: This protects performance in a seven-bird chorus.
- **Master bus with headroom and soft limiting**: The rationale is avoiding persistent masking and keeping the chorus listenable.
- **Listen-in mix ramps with ambient floor for other birds**: The chosen bird becomes clearer without muting the rest of the aviary.
- **Time-of-day and rain attenuation**: These change activity without muting the night scene.
- **Song-fragment offer using procedural motif approach**: It keeps offers in the same sound language as bird calls and allows mood-shaped response.
- **Call captions from actual realized motif parameters**: Captions stay synchronized with what sounded, where the bird is, and its mood.
- **Captions default on when WebAudio is missing, blocked, or off**: This preserves access when audio cannot play.
- **Blind recognition sessions and seven-bird chorus test**: The plan uses these to prove familiar birds remain recognizable and calls do not sound looped or wrongly synthetic.

### 8. Accessibility as a full product surface

- **Slow naturalist narration from the same snapshot/facts**: It gives screen-reader users concrete species, positions, light, sound, and behavior in the same product voice.
- **Idle narration updates every 30-60 seconds with coalescing**: The rationale is avoiding screen-reader queue flooding.
- **Prioritized narration for greeting, accepted offer reaction, or settle**: These are notable moments worth surfacing without turning every state change into output.
- **One controlled live region plus on-demand scene summary**: This controls speech queue behavior and provides a deliberate summary surface.
- **Keyboard order through top-bar controls then bird region**: The rationale is complete keyboard access to controls and birds.
- **Arrow keys between birds, Enter listen-in, Escape exit/close**: These make bird interaction and menus keyboard navigable.
- **High-contrast focus, hit targets, stable focus across updates/renames**: The rationale is reliable operation against day/night palettes and changing snapshots.
- **WCAG AA checks for captions, top-bar text, settings, notebook, errors, and focus cues**: The plan avoids relying on color or audio alone.
- **Vestibular audit of reduced-motion crossfades**: It does not assume any fade is safe.
- **Keyboard and screen-reader testing for adoption, host, visit, export, and deletion paths**: Equal access applies to all v1 paths before launch.

### 9. Performance, operations, and acceptance gates

- **Initial JavaScript under 2 MB gzipped and lazy-loaded secondary surfaces**: The plan keeps notebook/settings/invite/export from delaying first paint.
- **First bird visible within 500 ms on mid-tier mobile/4G**: This directly supports the first-bird experience test and avoids loading-transition feel.
- **Dedicated navigation-to-first-bird paint mark**: The rationale is measuring the actual product promise.
- **Aggregate-only real-user first-bird distributions**: They catch field regressions without account or bird identifiers.
- **30-minute seven-bird weather/chorus stress scene at 60 fps idle target**: The plan wants release gating for sustained frame and memory behavior, not a short demo.
- **Post-GC heap and audio/timer/listener/notebook count inspection**: This catches leaks over the soak.
- **Browser-specific audio fallback and reduced-motion performance variants**: These ensure non-mainline surfaces meet the same gates.
- **Operational monitoring from day one**: Tick timing, queue lag, snapshot size/latency, HTTP errors, first-bird paint, frame timing, and AudioContext failures expose health regressions.
- **Release tests for drift, sync, visitor rejection, invitations, magic links, deletion/recovery, export, no toast, and no counters**: The rationale is proving product promises and negative commitments before release.
- **Synthetic bird histories for engine calibration**: Real per-bird data never enters analytics.

### 10. Security, deletion, and recovery

- **Rate-limited magic-link and invitation email issuance**: The rationale is abuse prevention.
- **Token entropy, hashing, atomic expiry/consume, rotating sessions, and CSRF protection**: These secure authentication and mutations.
- **Visitor grants scoped to one aviary and one read-only route**: The plan prevents notebook, account, and event permissions from leaking to visitors.
- **Transactional revocation before API response**: Revocation must take effect before the API says it has happened.
- **Encrypted invitation recipient addresses only where needed**: The plan limits address use to sending and displaying the host log.
- **No logging of tokens, email, bird-event bodies, or vector values**: This protects privacy boundaries in operational records.
- **Consistent export transaction snapshot**: It ensures birds, IDs, names, vectors, moods, notebook, and settings correspond to one state.
- **Short-lived single-use export download link to verified email**: The rationale is private delivery of the data-portability package.
- **Deletion disables active sessions, visits, invites, and new tick work immediately**: This stops access and simulation while recoverable state remains for 30 days.
- **Hard deletion at day 30 with backup key erasure**: The rationale is proving no recoverable account-scoped copy remains.
- **Deletion-job receipts free of per-bird or email data**: The receipt itself must not violate the privacy boundary.
- **Restore-before-day-30 and failed-restore-after-hard-deletion tests**: These prove both recovery and final deletion behavior.

### 11. Build sequence and rollout

- **Foundation phase**: It locks schemas, boundaries, ownership, timezone, ordering, fixtures, and first-bird marks before ornamental polish so core constraints shape the build.
- **Continuity core phase**: It proves persistent birds, deterministic seeds, snapshots, two-device convergence, reload survival, catch-up, restore, and rename before adding polish.
- **Life and gestures phase**: It tunes mood, perch, calls, weather, greetings, presence, listen-in, offers, settle, and notebook while reviewing for canned cues, rapid drift, and quiet absence.
- **Equal-access surface phase**: The plan explicitly says not to defer reduced motion, narration, captions/WebAudio silence, keyboard controls, and AA QA after launch.
- **Account and social completion phase**: It completes sessions, email change, export/deletion, invites, visits, notifications, and privacy isolation after core life/access surfaces.
- **Release gates phase**: It removes any product-surface counters/toasts and validates multi-browser, audio, accessibility, soak, first-bird, tick-lag, restore/deletion, and abuse behavior.
- **Internal and invited beta first**: The plan widens only after paint, tick, sync, audio, and accessible-surface gates pass.
- **Synthetic 3-, 5-, and 7-bird aviaries before general release**: This exercises future density before real accounts reach higher bird counts.
- **Capability flags for optional invites and higher bird counts**: The rationale is containing regressions without rewriting existing bird identities.
- **Halt expansion criteria**: The plan stops rollout on vector loss/reset, cross-device divergence, visitor writes, hidden-tab presence inflation, accessibility regressions, first-bird misses, frame/memory failure, or recurrent tick lag.

### 12. Principal risks and responses

- **Drift tuning response**: The why is to keep drift from being too fast, too slow, or click-rewarding through caps, fixtures, qualitative review, nonnegative deltas, and preserved vectors.
- **Presence inflation/exclusion response**: The plan uses conjunction tests, bounded leases, union intervals, and calibrated activity windows so presence is neither "tab open" nor unfairly excluding touch/screen-reader users.
- **Sync corruption response**: Single server writer, ordered locks, idempotent ticks, revision checks, checksums, and stale-snapshot fail-closed behavior protect identity and drift.
- **Procedural-call response**: Grammar variation, signature anchors, headroom, blind listening, and seed-based regression renders address canned sound and chorus masking without recorded loops.
- **First-bird response**: Private SSR, tiny state, deferred code, seed/phase continuity, and quiet-field slow path keep first bird from feeling like a loading transition.
- **Accessible-aliveness response**: Separate reduced-motion composition, paced fact narration, realized-grammar captions, and screen-reader/vestibular gates keep accessible versions from losing aliveness.
- **Analytics privacy response**: Separate telemetry credentials, allowlisted metrics, log redaction tests, and metric privacy review prevent the privacy boundary from eroding.
- **Invitation-link response**: One-time redemption, per-pull grant checks, visible polling within 30 seconds, validation on return, and revocation tests prevent links from outliving intent.
