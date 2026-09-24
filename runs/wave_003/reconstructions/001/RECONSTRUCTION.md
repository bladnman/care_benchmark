## System-level intent

- Private, account-bound continuity is central. The plan opens with "one private, browser-only aviary per email magic-link account" and repeatedly insists that "Every client renders the same canonical bird identities, moods, and accumulated personality drift." This shows up again in the single source of truth, snapshot versioning, server time, and the rejection of public discovery, feeds, profiles, and shared aviaries.
- The product voice is quiet naturalist observation rather than game language or engagement pressure. The plan specifies "lowercase, specific, present-tense naturalist observation" for product prose and forbids "welcome banner, streak, score, trait stat, distress, hunger, death, public discovery, comparison, or notification by default." Cooldowns, unavailable reactions, later adoption, and visit notifications all preserve that low-pressure voice.
- Canonical life belongs to the server; the browser renders and interpolates. The plan says birds "continue changing on a server tick with no browser open," the simulation service is the "only" writer of bird personality, mood, timeline, weather, call schedule, and notebook observations, and the browser "never advances canonical simulation time."
- Privacy and data minimization are product architecture, not a separate compliance layer. This appears in "Email is never an ID, partition key, telemetry field, or log field," raw personality vectors being limited to the on-demand export, operational telemetry having only "schema-allowlisted aggregate health data," and the instruction not to join per-bird stores to a warehouse, training, recommendation, or "population drift dashboards."
- Bird identity must remain stable across expression changes. The plan says never replace a bird ID on "rename, migration, or asset update," and that grammar or visual releases must not regenerate bird IDs, reset persisted vectors, or rewrite old notebook entries. Persistent vectors, call signatures, names, and notebook history all follow this intent.
- The first screen should feel like arrival into an already-living place, not a loading or onboarding sequence. The plan says the first visible scene is "already at a sampled point in its timeline, never an entry animation," with no spinner or status toast if delayed, and only the first adoption interval allowed to be empty before the first birds arrive.
- Accessibility ships as part of the core visual/audio experience. The plan says "Accessibility ships in the same release gate as the visual/audio path," and makes narration, call captions, reduced motion, keyboard routes, visible focus, WCAG AA contrast, and actual screen-reader testing part of the same implementation flow.
- Visiting is restrained, named, and read-only. The plan allows a host to "invite a named visitor to a read-only view," but rejects comments, feeds, profiles, public discovery, visitor mutations, visitor presence credit, visitor greetings, and host notebook sharing.
- Progression is age-based and non-monetized. Later bird eligibility "depends only on aviary age" and is "never earned from visit count, offers, or payment." Newcomers are shown quietly, can be deferred with "later," and carry no countdown, badge, or loss of eligibility.
- Expression should be procedural but recognizable. Calls use a compact motif grammar, stable signature seed, and bounded variation; the plan forbids recorded calls and loop concatenation while requiring motif contour and timbre to remain recognizable across mood and trait changes.
- Failure handling should be honest and plain. Errors are "matter-of-fact" with stable machine codes and recovery actions. During outages the client keeps a rendered view only briefly, then shows a retry surface; it must not "run a local substitute simulation" or pretend stale state is current indefinitely.
- Performance, sync, privacy, and accessibility are release gates. The rollout waits for "performance, privacy, accessibility, and sync gates," with first-bird, frame-time, heap-slope, tick-latency, privacy-schema, and accessibility checks treated as launch criteria rather than polish.

## Per-feature whys

### 1. Product contract and decisions

- Private browser-only aviary per magic-link account: the plan ties this to having a single private canonical aviary whose bird identities, moods, and drift render the same on every client.
- Two stable, named starter birds: NOT RECOVERABLE FROM PLAN
- Coherent pool of roughly six species and maximum of seven birds: the plan's later rationale is that seven birds must still remain "a small social system" with bounded chorus, load, and recognizability.
- Watching the aviary: the rationale is the core host relationship with a living scene that changes even with no browser open.
- Listen-in on one bird: the plan says listen-in adds a "bird-specific signal" and, in the client, ramps the focused bird up while others remain audible, giving attention to one bird without erasing the ambient aviary.
- Offering seed, song fragment, and still pool: the plan uses offers as bounded interaction signals, where an accepted offer adds curiosity and proximity to an offer adds boldness, while seed and still pool alter reaction without becoming "hunger mechanics."
- Settle as an item in the account icon menu: the plan explicitly says this "preserves the four-icon top bar" while satisfying the top-bar access requirement.
- Automatically written field notebook: the plan's rationale is to preserve sparse, immutable naturalist observations of the aviary, not an event feed, trait readout, visit-frequency report, or user-behavior judgment.
- Named read-only visitor view: the plan uses this to allow sharing while keeping visitor capabilities absent at authorization, rejecting host mutations, private event logs, raw vectors, host settings, greeting cues, presence leases, and notebook mutation.
- Raw personality vectors in account export only: the plan says this resolves the edge between account portability and the ban on numerical traits in product UI by treating the export as the "sole raw-data exception" and labeling it account data, not bird stats.
- Visit notifications as opt-in email only: the plan treats this as an "explicit opt-in exception" to the no-notifications rule, so new accounts keep notifications off, onboarding omits them, badges are absent, and there are no push or engagement reminders.

### 2. Delivery shape and ownership boundaries

- Small web client, authenticated API, transactional store, email worker, and simulation workers: the plan uses this split to keep authentication, validation, reads, invitations, exports, deletion, canonical simulation, email, and presentation in separate ownership boundaries.
- Simulation service as the only writer of personality, mood, timeline, weather, calls, and notebook observations: the rationale is to keep canonical state server-authored and avoid clients advancing simulation time or writing drift directly.
- Browser interpolation, ambient ornament, audio synthesis, presentation, and ephemeral UI state: the plan assigns these to the browser because the browser should render and synthesize the experience without advancing canonical simulation time.
- Operational metrics path with only allowlisted aggregate health data: the plan gives the rationale as privacy isolation: the metrics path has no read access to the simulation database.
- Edge-delivered HTML shell with embedded authenticated initial snapshot: the plan uses this so the first private scene loads close to users while personalized responses remain `private, no-store` and cannot be mixed across accounts by a CDN.
- Server-rendered first bird pose followed by hydration without DOM replacement: the plan's rationale is that the first visible scene is already sampled from the timeline and first-bird performance is protected.
- Quiet field with subtle non-bird motion when the snapshot is delayed: the rationale is to avoid spinner or status-toast language that would break the quiet presentation.
- Versioned data contracts and feature flags for coefficients, assets, grammar, and eligibility: the plan says releases should change future expression without regenerating bird IDs, resetting persisted vectors, or rewriting old notebook entries.
- Forward migrations with compatibility readers and backup recovery practice: the plan's rationale is safe rollout before switching writers and proven recovery before launch.

### 3. Persistent model

- Account record with encrypted verified email, pending new email, timezone, preferences, notification opt-in, and deletion deadline: the plan's rationale is contact privacy, timezone consistency, accessibility/audio persistence, opt-in notifications, and soft deletion.
- Email never used as ID, partition key, telemetry field, or log field: the plan gives this as a direct privacy invariant.
- Device sessions with opaque token digests and individual revocation: the rationale is revocable per-device access control.
- Aviary record with deterministic seed, snapshot version, tick index, consumed event sequence, weather, and day-cycle state: the rationale is deterministic simulation, snapshot sync, and recoverable tick progression.
- Bird record with stable UUID, species, mutable name, call-signature seed, persisted traits, mood, perch, behavior state, and timeline: the plan's rationale is stable identity and continuity across rename, migration, and asset update.
- Interaction events as server-sequenced append-only records: the plan uses this for validation, idempotency, tick recovery, and making clients submit observations/actions rather than absolute trait values.
- Presence leases limited to host sessions and merged across tabs/devices: the plan's rationale is to credit bounded host attention without multi-tab or multi-device drift multiplication; visitors cannot create presence.
- Raw presence interval retention only as long as tick recovery requires: the plan ties this to bounded recovery needs rather than long-term behavioral storage.
- Notebook entries with immutable lower-case prose and no archive cutoff: the plan's rationale is durable field-notebook history that can be paged backward and is not rewritten by later changes.
- Invitation and visit records with encrypted invitee email, token digest, expiry, revocation state, scoped session, and interval log: the plan uses these to keep invitee contact data separate, scoped, one-time, revocable, and minimally duplicated.
- Token hashing and keeping tokens/emails out of recorded URLs and access logs: the rationale is to prevent bearer-link and contact leakage.
- Per-account encryption material: the plan says hard deletion can make backup remnants unreadable while backup rotation removes ciphertext.
- Account timezone captured and user-changeable: the plan says the canonical day/night cycle follows that setting, including daylight-saving changes, and avoids devices in different timezones silently fighting over it.
- Local settle lighting override with only a small canonical mood-quieting event: the plan says this keeps another device on the same bird state without unexpectedly imposing the initiating device's goodbye gesture.

### 4. API and security contracts

- `/v1` versioning, TLS, CSRF protection, and same-site secure cookies: the plan's rationale is stable contracts and browser-session security.
- UUID idempotency key on every write: the plan says retries return the prior result, preventing duplicate effects.
- Matter-of-fact errors with stable machine codes and recovery actions: the plan's rationale is recoverability and keeping errors out of naturalist prose.
- Magic-link consumption by explicit POST after landing: the plan says mail scanners cannot spend the token.
- Magic-link expiry and atomic single consumption: the rationale is replay protection.
- Rate limiting per email inside the isolated auth boundary without logging email as a key: the plan balances abuse control with the no-email-in-logs invariant.
- Email-change verification before switching while the old address continues to work: the plan preserves access during email change.
- Lazy-loaded account, accessibility, invite, and notebook code: the plan uses this to keep noncritical account and notebook paths out of the initial route.
- `GET /aviary/snapshot` without raw vectors or per-bird event history: the plan's rationale is that normal scene snapshots are not trait stat or private history surfaces.
- Conditional snapshot requests that still refresh time-dependent directives: the plan says state version can be unchanged while time directives still need fresh rendering.
- `POST /aviary/events` appending accepted actions rather than updating traits inline: the rationale is that the next simulation tick, not the request handler, authors trait changes.
- `POST /aviary/presence` reporting visibility, focus, and recent activity, with server clamping: the plan's rationale is bounded qualifying presence and rejection of stale or malformed reports.
- Presence endpoint unavailable to visit sessions: the plan prevents visitor traffic from producing host presence or drift credit.
- Host and visitor snapshots sharing visual/audio directives while visitor capabilities are absent at authorization: the plan says visitor restrictions must not be merely disabled buttons.
- Visitor chrome exposing only a way to leave and notebook remaining host-only: the plan gives privacy as the reason for the v1 visitor chrome.
- On-demand visit log without badge or default notification: the plan allows the host to see invitee email, date, approximate duration, and outstanding invitations without introducing notification pressure.
- Invitations created only by explicit host action, expiring after 30 days if unredeemed and never auto-renewed: the rationale is deliberate, bounded sharing.
- One-time invite bearer link exchanged for a scoped visit cookie: the plan uses this to reduce bearer-token exposure after deliberate redemption.
- Revocation invalidating tokens and active sessions immediately, with `visit_no_longer_available` on next pull: the rationale is prompt stale-access removal.
- Bounded visitor polling target of 15 seconds: the plan says this lets revocation reach an open view promptly.

### 5. Server tick, presence, drift, and mood

- Approximately 60-second canonical tick whether a client is connected or not: the plan's rationale is that birds continue changing with no browser open and do not freeze until the next visit.
- Partitioned jobs by aviary UUID with worker leases: the plan uses this to process live aviaries reliably through the worker pool.
- One-transaction tick that locks the aviary, processes events through the cutoff, advances state, appends notebook entries, updates cursors, and commits: the rationale is atomic state/event progression.
- Retrying the same tick returning the committed result: the plan says this avoids applying deltas twice.
- Deterministic missed-tick replay after outage: the plan says not to freeze state until the next visit or fabricate catch-up on the client.
- Alarms on backlog and p99 tick latency over 5 seconds: the rationale is operational detection of simulation lag.
- Presence eligibility as visible, focused, and recent pointermove/keypress: the plan says not to count a page just because it is open, audio is playing, a pointer is down, or focus exists without visibility.
- Four-minute activity window as a build-time calibration parameter: the plan says stationary watching and mobile-browser behavior must be tested, then the value frozen before drift calibration.
- Qualifying pings about every 15 seconds, immediate stops, no backfill, server clamp, missed-ping timeout, and unioned intervals: the plan's rationale is bounded credit with no invented presence or multi-device multiplier.
- Explicit mobile touch behavior resolution if touch-only watching cannot produce pointermove/keypress: the plan says to resolve the product/spec gap before launch rather than silently count taps as presence.
- Presence as dominant slow input, listen-in as bird-specific signal, offers as curiosity/proximity signal, and settle as mood-quieting only: the plan uses these signals to shape drift and mood while keeping settle from adding presence.
- Per-bird offer cooldown and daily cap: the plan says effective offer influence must not become a rapid-click multiplier.
- Nonnegative low-pass trait update, clamped vectors, and zero input decaying the filter without reducing vectors: the plan's rationale is monotonic earned personality; quiet after absence is temporary expression, never reversal.
- Synthetic-account calibration instead of real per-bird histories: the plan's rationale is to tune coefficients without analyzing real interaction histories.
- Acceptance envelope for one week, three weeks, one session, and absence: the plan uses this to prevent drift that is too fast, too slow, visually too early, or distress-like after absence.
- Persisted five-state mood model rather than reset on tab open: the plan says the next session should begin from the actual intervening trajectory, without a snap to neutral.
- Seeded semi-Markov transitions with dwell times, circadian pull, events, weather, social responses, and personality biases: the plan uses these to produce mood continuity that follows time of day, interaction, weather, and bird personality.
- Server-chosen perch zone and behavior intent from personality/mood with dwell times: the plan says dwell avoids jitter and personality makes wary birds tend back while bold birds tend front.
- Bounded cross-bird calls and mood influence with staggered timing: the plan says seven birds should remain "a small social system rather than a row of independent loops."
- Per-leaf and feather motion as client-only ornament: the rationale is that ornament never becomes persisted simulation state.

### 6. Call grammar and greeting runtime

- Compact species motif grammar with contour, envelopes, rhythm, timbre, and variation ranges: the plan's rationale is recognizable procedural calls without downloaded recordings.
- Stable bird-level signature seed and immutable motif preferences at adoption: the plan uses these for per-bird call identity across future mood and trait changes.
- Scheduled call events with bird ID, event ID, absolute server start time, grammar version, motif, parameters, and response relationship: the rationale is canonical, schedulable audio shared through snapshots.
- Browser WebAudio synthesis with bounded voice pooling and reusable nodes/buffers: the plan avoids recorded calls and audio loops while keeping runtime bounded.
- Preserving motif contour and timbre while mood changes amplitude, tempo, and spacing and vocal frequency affects call probability: the rationale is recognizability across personality and mood expression.
- Simultaneous voice caps and quiet ambient floor: the plan says the mix must never become shrill.
- Return greeting derived from last host presence/session time without exposing absence text: the plan allows absence-sensitive greeting behavior without a toast, banner, absence count, or entry animation.
- Primary greeter weighted by boldness, warmth, mood, and absence length: the rationale is a bird-authored greeting shaped by personality and state.
- Seeded motif and pose variation per return event: the plan says greetings should never become a "three-clip rotation."
- Greeting cue within one to two seconds and staggered responses: the plan avoids a synchronized arrival chorus.
- Visitor never triggering greeting: the plan keeps visitor sessions read-only and separate from host presence or return behavior.
- Audio scheduling from server time to `AudioContext.currentTime`: the plan uses this to align sound with canonical scheduled events and resync softly after long gaps.
- Listen-in ramping the focused bird up and others down while others remain audible: the plan supports focused listening without removing the ambient aviary.
- Listen-in disengagement by second click, another bird, empty-space click, or keyboard focus leaving: the rationale is clear reversible focus behavior.
- Offers synthesizing song fragments and scheduling mood/personality-shaped responses: the plan ties offer audio to the same motif system and simulation-shaped reaction.
- Seed and still pool altering scene/reaction without hunger mechanics: the plan prevents offers from becoming feeding, reward, or hunger systems.
- Settle ramping calls down with lighting, cancelable within five seconds before commit: the plan allows a local settle transition to be reversed and restores presence if canceled.
- WebAudio blocked or unavailable behavior with captions on by default, plain audio-enable control, and graceful silence: the plan preserves accessibility and avoids recorded fallback or a fifth top-bar icon.
- Audio-context failure measurement without bird or call content: the rationale is aggregate reliability measurement without recording content.

### 7. Scene and interaction client

- Compact SVG bird/perch assets and DOM/CSS compositor transforms with semantic DOM controls: the plan uses these for the seven-bird scene, performance, and keyboard/screen-reader access.
- Background sky/foliage, middle-plane perches and birds, and foreground branches/leaves: NOT RECOVERABLE FROM PLAN
- Normalized scene coordinates and responsive perch spacing: the plan says every bird must stay visible on narrow phone and wide desktop viewports without crop, pan, zoom, or horizontal scroll.
- Account-local continuous day/night palette: the rationale is that the canonical day/night cycle follows the account timezone and server time in snapshots.
- Nightjar-like bird remaining active at night: NOT RECOVERABLE FROM PLAN
- Soft, rare weather with no dramatic storm mode: NOT RECOVERABLE FROM PLAN
- Snapshot action start/end time, endpoints, pose phase, and easing with client sampling at current server time: the plan's rationale is first-frame accuracy and interpolation without teleporting.
- Pausing rendering and audio when a tab hides: the plan gives battery as the reason while the server keeps ticking.
- New snapshot on visibility return, long animation-frame gaps, or low-frequency visible keepalive, then blending only when the timeline allows: the rationale is convergence without snapping.
- Quiet bounded stale-scene retention followed by recoverable error on network delay: the plan says not to pretend stale state is current indefinitely.
- Ambient leaves and feathers with seeded intervals and pooled DOM nodes: the plan keeps them slow, reusable, and noncanonical.
- Four top-bar icons with Settle in the account menu: the rationale is preserving the specified four-icon top bar.
- Top bar fading nearly transparent after pointer stillness and restoring on pointer movement, keyboard activity, focus, or open menu: the plan balances unobtrusive chrome with keyboard reachability.
- No bird labels, tooltips, badges, or inline controls in the scene, except enabled call captions: the plan grounds this in sparse scene-adjacent chrome and the captions as the intentional accessibility exception.
- Offer sheet opened from the top bar containing seed, song fragment, and still pool: NOT RECOVERABLE FROM PLAN
- Short functional cooldown state without suggesting reward or penalty: the plan uses this to avoid gamified feedback.
- Conditional reactions that can be quiet or absent: the plan says not to guarantee a positive animation.
- Two starter species assigned by the service, then named by the host with renameable suggestions: NOT RECOVERABLE FROM PLAN
- No species catalog: NOT RECOVERABLE FROM PLAN
- Later bird eligibility based only on aviary age at the provisional schedule: the plan says eligibility is never earned from visit count, offers, or payment.
- Available newcomer presented quietly in the account menu after the return greeting has room to occur: the rationale is to avoid interrupting the greeting and to keep adoption quiet.
- `later` without countdown, badge, or loss of eligibility: the plan avoids pressure and notification-like urgency.
- Adoption allocating a new stable UUID/vector/signature, enforcing cap transactionally, and animating only the new bird: the rationale is stable identity, the maximum bird cap, and no disturbance of existing birds.
- Precise adoption copy and placement review with design: NOT RECOVERABLE FROM PLAN

### 8. Notebook and accessible expression

- Notebook entries generated in the simulation transaction from noteworthy aviary observations: the plan uses the same canonical simulation that produces perch changes, weather, greetings, and call exchanges.
- Entries based on unusual greeting order, long quiet interval, perch change, weather passing, or characteristic call exchange: the plan's rationale is aviary observation, not user behavior judgment.
- Editorially reviewed grammar/templates with species, names, time of day, and local context: the plan uses these to keep notebook prose in naturalist voice.
- Dedupe, roughly one entry every few days, and a strict cap: the rationale is to keep even active aviaries sparse.
- Never turning the notebook into an event feed, trait readout, visit-frequency report, or user-behavior judgment: the plan states this directly as the reason for its shape.
- Persisting finished prose: the plan says later renames should not rewrite history.
- Read-only notebook with indefinite backward paging and no public discovery: the rationale is durable private field-note history.
- Narration built from the same scene/call timeline as visual rendering: the plan uses this to keep visual, audio, and screen-reader expression consistent.
- Slow polite `aria-live` updates, duplicate suppression, and concise user-initiated observations: the plan says not to flood the screen reader queue.
- Controls and narration kept separate: the rationale is that keyboard focus should not be stolen.
- No raw state labels or numerical personality values in narration: the plan keeps narration out of trait-stat and raw-state territory.
- Actual screen-reader testing for pace and coherence: the plan says ARIA snapshots alone are insufficient.
- Reduced-motion path with still poses, cross-fades, no leaf drift, and slower color changes: the rationale is motion accessibility while bird mood, calls, notebook, and progression remain the same.
- Captions from actual scheduled grammar events, timed to sound and placed near the bird: the plan makes captions accurate to procedural calls and accessible in the same naturalist voice.
- Avoiding captions that cover another bird or fall off narrow viewports: the rationale is readable responsive caption placement.
- Tab order through top-bar controls, bird focus, arrow movement, Enter listen-in, Escape leave, dialog keyboard operation, and focus restore: the plan's rationale is complete keyboard operability.
- High-contrast visible focus and WCAG AA contrast across day/night palettes: the plan requires controls, captions, settings, errors, and copy to remain readable in every lighting state.
- Accessibility release gate with the visual/audio path: the rationale is that accessibility is not a static afterthought.

### 9. Sync, failure handling, and privacy

- Simulation database as the single source of truth with snapshot versioning and server time: the plan uses this so multiple host devices render the same state.
- Clients not sending absolute mood or personality values: the rationale is server-authored personality and mood, with clients submitting events rather than state.
- Receipt-sequenced events applied once in the next tick with event cursor and state committed atomically: the plan's rationale is no lost or doubled drift.
- Concurrent listens/offers processed in sequence with offer cooldown checked under the aviary lock: the plan gives a clear functional result and prevents duplicate offer influence.
- Rename/adoption/settings optimistic checks or transactions returning current record on conflict: the rationale is conflict safety without last-write-wins.
- Never using last-write-wins for vectors: the plan protects accumulated personality drift.
- Magic-link replay returning expired/used-link error and expired sessions showing sign-in: the rationale is secure recovery from invalid authentication state.
- Read outage behavior retaining a rendered view briefly and then showing retry without local substitute simulation: the plan keeps failure honest and avoids client-side fiction.
- Event-write outage behavior that does not claim an offer was accepted: the rationale is not to misrepresent failed mutations.
- Queued client events with bounded lifetime and idempotency keys: the plan prevents stale absolute state from being reinterpreted.
- Revoked visit rejected on the next visitor snapshot even with cached art assets: the rationale is authorization over cached presentation.
- Per-bird interactions and presence stored only for the aviary simulation with bounded recovery retention and deletion: the plan limits behavioral data to the simulation need.
- No warehouse, training, recommendation, cross-account behavioral analysis, or population drift dashboards: the plan states these as excluded privacy uses.
- Operational telemetry allowlist excluding account UUID, email, bird ID, species, mood, interaction type, and prose: the rationale is avoiding PII and cardinality leakage.
- Automated telemetry-schema checks and production sampling audits: the plan uses these to detect leakage regressions.
- Settings privacy link stating allowed categories and per-bird exclusions: the rationale is plain disclosure of what is and is not collected.
- Host-requested export with encrypted queued file and short-lived one-time emailed download link: the plan supports portability while keeping export access bounded.
- Soft deletion immediately making the account unavailable with 30-day explicit recovery: the plan gives a recovery window while stopping normal availability.
- Hard deletion of account-tied data, exported files, sessions, invites, contact data, and encryption material: the rationale is irreversible purge and unreadable backup remnants.
- No account-identifiable deletion tombstone or per-account telemetry after deletion: the plan keeps deletion from leaving account-identifiable residue.
- Recovery just before the deadline and purge just after it as tests: the plan verifies both sides of the deletion promise.

### 10. Performance, verification, and rollout

- Initial JavaScript under 2 MB gzipped, first bird under 500 ms, 60 fps idle, and no positive heap trend: the plan uses these as CI budgets for first-route performance and sustained sessions.
- Rendering the first bird before account/notebook code or audio initialization: the rationale is protecting first-bird visibility.
- Compact SVGs, pooled DOM/audio resources, and stopping hidden-tab render work: the plan uses these for frame rate, memory, and battery performance.
- Synthetic browsers across supported geographies and recent Chrome, Safari, Firefox, and Edge: the rationale is browser and geography coverage before launch.
- Unsupported-browser surface for older browsers: the plan provides a direct fallback instead of attempting degraded behavior.
- Gates on first-bird, frame-time, heap-slope, and accessibility checks across normal and reduced-motion modes: the plan treats performance and accessibility as release blockers.
- Tick p99 and tick lag/replay backlog monitoring: the rationale is detecting scheduler latency and recovery backlog separately.
- Deterministic simulation fixtures for calibration, monotonicity, bounds, continuity, weather, social response, seven-bird chorus, and call recognition: the plan uses these to prove simulation behavior before launch.
- Presence fixtures for all visibility/focus/activity combinations, inactivity, blur/hide/settle/tab close, suspension, multiple devices, mobile pointer behavior, and background overnight: the rationale is proving exact presence eligibility and zero background credit.
- Sync/security fixtures for concurrency, tick crash boundaries, replay, two-device convergence, auth, revocation, visitor attempts, export scope, and hard deletion: the plan uses these to prove no lost/doubled drift and no unauthorized access.
- Experience fixtures for first-frame mid-action, greetings, no welcome text, responsiveness, stability, WebAudio fallback, screen readers, keyboard, contrast, and reduced motion: the plan uses these to verify the intended quiet, accessible experience.
- Privacy fixtures asserting no raw vectors in normal API or telemetry and no account emails in identifiers/logs: the plan uses these to protect the raw-data and contact-data boundaries.
- Shipping in dependency order from data/auth/tick skeleton through scene, audio/mood/notebook, accessibility, visits/export/deletion, and hardening: the plan's rationale is building on the simulation and data foundations before social/privacy hardening.
- Closed alpha at two birds and limited beta with synthetic aged aviaries at three, five, and seven: the plan uses this to tune recognizability and load before production.
- Production launch with two starter birds while seven-bird capacity is built and tested from day one: the rationale is age-based later additions without untested capacity.
- Flags that pause new adoption offers or a faulty grammar version without deleting existing birds or rolling back a vector: the plan uses flags for safe release control without identity or personality loss.
- Traffic ramp by cohort only after performance, privacy, accessibility, and sync gates pass: the rationale is staged rollout after the core gates.
- Aggregate health instrumentation from day one and never per-bird engagement: the plan's rationale is operational visibility without engagement tracking.
