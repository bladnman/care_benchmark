## System-level intent

- **One private canonical aviary, not a social product.** This appears in the opening boundary: "browser-only, single-user aviary," "one canonical scene and one canonical state per account," "one aviary per account," "no public discovery," "shared aviaries," or "co-presence." Later sections repeat that the server is "the only writer" and that visitors receive only a "read-only scene snapshot stream."
- **Presence is the relationship signal.** The plan explicitly says to "Treat presence as the main relationship signal, not clicks or session opens." The simulation section narrows this to visible, focused, active leases and says "Opening a tab alone is never presence."
- **Continuity without punishment.** The product boundary says "Absence never reduces traits or creates distress." The simulation rules repeat that "neglect does not lower any trait," that long gaps must not "invent presence," and that absence "cannot create illness, distress or personality loss."
- **Server-owned state and deterministic sync.** The architecture, model, API, simulation, and sync sections all insist that the server owns canonical accounts, birds, vectors, moods, events and snapshots; clients "render snapshots and submit typed events," "never absolute state," and there is "no client-to-client merge."
- **Slow expressive personality, fast visible mood.** The product boundary contrasts "Mood" as "visible and fast-changing" with a "persistent personality vector" that is "slow, server-owned, monotonic toward expressive, and never numerically surfaced." The simulation section keeps mood separate from personality deltas.
- **Anti-gamification and quiet product voice.** The plan bans "scores, visit streaks, badges, counters," "reward language," "guilt surface," "welcome toast," "new-visit badge," and "push engagement loop." Notebook copy avoids "you visited" claims and streaks, and account surfaces use "matter-of-fact copy."
- **Privacy by default.** The plan keeps notebook and account controls "host-only," visitors "read-only," email encrypted, secrets hashed, snapshots private, and telemetry "aggregate-only." It repeatedly says not to expose "trait numbers," "raw event log," or account-level interaction histories.
- **Ambient, already-running scene.** The core session is "observing the already-running scene." First render should show a bird "already mid-action," slow snapshots show "the quiet field, never a spinner," and background ticks advance aviaries "whether or not a client is connected."
- **Sparse single-scene interaction.** The scene is a "single horizontal composition" with "no pan, zoom, or scene scroll." UI chrome stays to a "sparse top bar"; offers open from that bar, and birds choose their own perches.
- **Accessibility ships in v1 as part of the product, not a fallback.** The boundary includes "captions, screen-reader narration, keyboard access, and a designed reduced-motion presentation in v1." The accessibility section says to "Ship the accessible surfaces with v1" and to keep reduced motion as a "designed view rather than static fallback."
- **Procedural identity over recorded assets.** The audio sections tie call identity to "species motif library and stable bird identity," require captions from "the same grammar choice," and say to "never download or loop recorded bird calls" and "never fall back to recorded audio."
- **Release only through calibration, gates and rollback.** The plan uses "seeded simulation cohorts, not population behavior analytics," product review, browser/performance/privacy gates, controlled cohorts, and rollback if "state integrity, call identity, a11y or performance regresses."

## Per-feature whys

### 1. Product boundary and decisions

- Browser-only, single-user aviary with one canonical scene and one canonical state per account: supports the plan's private canonical scene boundary and avoids shared aviaries, co-presence, public discovery and client-owned state.
- Two server-selected starter birds for a new account: NOT RECOVERABLE FROM PLAN
- User naming and later renaming of birds: NOT RECOVERABLE FROM PLAN
- Adopting additional birds by age-based intervals up to seven: age, "not attention," visits, interaction count or payment, controls availability so adoption stays quiet and non-reward-like.
- Magic-link accounts: NOT RECOVERABLE FROM PLAN
- Cross-device sync: preserves one canonical account state across devices rather than separate device histories.
- Privacy controls, export and deletion: support explicit user control of account-owned data and the data lifecycle guarantees later described for export and hard deletion.
- Per-invite read-only visits: preserve a "read-only ambient visit" where a visitor "does not affect host presence or state."
- Captions, screen-reader narration, keyboard access and reduced motion in v1: ship accessible surfaces with v1 and keep the scene, calls, mood and notebook available across accessibility modes.
- Presence over clicks or session opens: presence is the "main relationship signal"; opening a tab alone must never count as relationship or drift.
- Mood visible and fast-changing while personality is slow, server-owned and not numerically surfaced: keeps immediate expression separate from persistent trait drift and avoids exposing trait numbers in product UI.
- Absence never reducing traits or creating distress: prevents neglect penalties, distress, illness, personality loss and guilt surfaces.
- No scores, visit streaks, badges, counters, profiles, discovery, shared aviaries, co-presence, payments or push engagement loop: keeps the product non-gamified, private, single-user and outside push-driven engagement.
- Browser IANA timezone for local day/night and mood: derives local daypart from the saved timezone for scene and mood behavior.
- Server-selected offer recipient based on proximity, mood and curiosity rather than clicking a bird: preserves the "top-bar offer flow" and keeps recipient selection server-owned.
- Host-only notebook and account controls while visitors receive scene snapshot and audio/render data only: preserves the "private notebook" and read-only visit boundary.
- Consume invite link once to mint a short-lived visitor session: supports scoped invitation access and limits visitor state.
- Optional visit notification off by default and confined to an in-product account surface: preserves the "no-push policy."

### 2. Architecture and trust boundaries

- Small web client plus server-side account, aviary and simulation service with transactional database and partitioned tick/event queue: supports canonical state, ordered events and simulation that can advance independent of clients.
- Server ownership of accounts, identity, birds, vectors, moods, timestamps, notebook entries, invitation state and snapshot version: ensures no client computes authoritative drift or writes absolute bird state.
- Renderer and WebAudio consuming a versioned presentation snapshot: allows rendering/audio to be replaced "without changing the simulation contract."
- Web shell with critical scene path downloaded up front and settings/notebook/invite management code-split: protects the first scene render while moving noncritical surfaces off the critical path.
- Identity/account API for magic links, sessions, verification, settings, export, deletion and restore: NOT RECOVERABLE FROM PLAN
- Aviary API for snapshot reads, arrival/greeting requests, owner interaction events, notebook, adoption and rename: NOT RECOVERABLE FROM PLAN
- Simulation worker as sole writer with approximately one-minute ticks whether or not a client is connected: keeps behavior canonical and "already-running" when the browser is closed.
- Immediate serialized simulation step for owner actions: makes visible reactions responsive while keeping all state changes through the server writer.
- Visit API issuing, consuming and revoking scoped invitations with read-only snapshot streams and visit records: supports scoped read-only visits and the host's visit log.
- Aggregate-only operational telemetry separated from simulation and account interaction histories: prevents analytics from joining to bird fields, simulation events or account-level histories.
- TLS, protected per-device sessions, CSRF protection, strict host-versus-visitor authorization, schemas and payload limits: uphold trust boundaries for authenticated writes and visitor separation.
- Hashing one-time link and invite secrets at rest: protects bearer secrets if stored records are exposed.
- Encrypted email and avoiding email as internal key, log identifier, partition key or telemetry dimension: keeps email out of operational identifiers and analytics dimensions.
- Generated synthetic account UUID for internal references: gives logs and internal joins a non-email reference.

### 3. Persistent model

- Relational records with explicit ownership and versioning: support account ownership, canonical versions and transactional updates.
- UTC timestamps with local daypart derived from saved IANA timezone: keeps storage stable while supporting local day/night behavior.
- Account record with settings, snapshot version and one aviary reference: NOT RECOVERABLE FROM PLAN
- Soft deletion immediately effective and recoverable for 30 days, followed by hard deletion: provides restore during the recovery window and then removal of account-owned data.
- Device sessions with creation, last-seen, revocation and expiry state: supports per-device sessions and central revocation from account settings.
- Magic links expiring after 15 minutes, single-use and rate-limited per email: limits sign-in token reuse and issuance abuse.
- Aviary record with age, canonical version, tick/event cursors, timezone, day/night phase, weather and settled marker: supports simulation scheduling, catch-up and one canonical aviary state.
- Enforcing one aviary per account: preserves the "one canonical scene and one canonical state per account" boundary.
- Bird record with stable UUID, species, user name, adoption timestamp and age metadata: keeps identity and species stable across renames and migrations while supporting age-based adoption.
- Personality vector with five normalized server-only scalars and bounded server-authored deltas: lets the simulation persist slow expressive traits without exposing numbers in ordinary UI.
- Including the personality vector only in explicit JSON export: satisfies the account-export scope while keeping numbers out of snapshots, APIs, narration and UI.
- Bird runtime state with persisted mood, pose/action, perch, call timing and cooldown: prevents an "open-time reset" and carries visible state between sessions.
- Simulation event log with ordered event IDs, device session, idempotency key and minimal typed payload: supports ordering, retries, deduplication and reconciliation.
- Compaction only after retry/reconciliation horizon while retaining canonical state and cursor: preserves enough raw event history for retry and reconciliation before cleanup.
- Events isolated from analytics: keeps interaction events out of behavior analytics and profiling.
- Notebook entries as sparse read-only naturalist prose with source facts but no trait values or visit-frequency claims: enables regeneration/audit while avoiding exposed trait values and gamified/profiling language.
- Notebook entries retained while the account exists and not archived or hidden: supports indefinite chronological scrolling.
- Invite and visit records with encrypted email, hashed bearer secret, expiry, consume/revoke state and narrow visitor session: limits visitor access to scoped read-only scene snapshots.
- Export/deletion jobs with expiring download-token hash and deletion recovery deadline: supports explicit export access and data lifecycle control.
- Export including birds, names, vectors, moods, notebook and account settings: matches the stated account-export scope.
- Hard delete covering sessions, invites, visit records, vectors, notebook, events, exports and identifiable data: fulfills promised deletion of account-owned data.

### 4. API and event contracts

- API versioning with canonical snapshot version and server time: supports sync, stale response handling and versioned presentation snapshots.
- Idempotency keys on mutating requests: ensure retries cannot duplicate greetings, offers, visits or drift input.
- Matter-of-fact errors for auth, sync, unsupported-browser and account surfaces: keeps account and system copy aligned with the plan's matter-of-fact product voice.
- `POST /auth/magic-links` and consume flow storing only a hashed 15-minute single-use token: supports magic-link sign-in with limited token lifetime and single-use semantics.
- Session list/revoke, email-change verification, account settings, export, delete and restore endpoints: NOT RECOVERABLE FROM PLAN
- Authenticated snapshot endpoint returning current presentation state, scene daypart/weather, version and adoption availability: feeds the renderer from canonical server state.
- Excluding personality numbers and raw event log from ordinary snapshots: keeps trait values and raw interactions out of product UI and ordinary APIs.
- Private, versioned/ETag snapshots with no shared public CDN cache: avoids leaking authenticated personalized scene data.
- Arrival endpoint on fresh entry or visible-tab return: produces return greeting behavior from absence length and bird state without showing a welcome toast.
- Arrival event not counting as presence or drift: prevents fresh entry itself from affecting relationship drift.
- Owner events endpoint for presence, listen-in, offer, settle and undo: centralizes typed interaction intake through the server.
- Binding the account from session rather than trusting body account ID: enforces account authorization on writes.
- Validating bird/offer scope and cooldown before queuing simulation: prevents out-of-scope or repeated offer effects.
- Returning acceptance and new canonical presentation version/action where ready: gives the client responsive feedback without client-owned state.
- Clients never sending mood or personality values: preserves server authority over canonical behavior state.
- Settle ending account-level presence and later qualified interaction re-engaging: makes settle a real quieting boundary without deleting future engagement.
- Five-second settle undo: makes settle reversible for a brief accidental-action window.
- Host-only notebook pages and no visitor notebook/settings endpoint: preserves private notebook and account controls.
- Adoption endpoint accepting a name for age-eligible server-selected species and rename-only bird patch: enforces server-selected adoption, stable identity and rename scope.
- Adoption availability not depending on visits, interaction count or payment: keeps adoption age-based and non-gamified.
- Explicitly addressed invitation creation, one-time visitor consume, read-only visitor snapshot and invite revoke: supports scoped ambient visits without visitor writes.
- Visitor snapshot never emitting owner presence: ensures visitors do not affect host presence, drift or state.
- Account invite status, outstanding invitations and recent visit log with no public discovery, visitor list or host badge: gives host account visibility without social discovery or badge pressure.
- First render delivering a small personalized snapshot with shell or private edge fetch: starts drawing from personalized canonical state as early as practical.
- Slow snapshot showing quiet field, never spinner, and not delaying drawing on noncritical assets: protects the quiet first impression and avoids blocking the scene.

### 5. Server simulation and calibration

- Pure deterministic transition function versioned with rules and explicit prior state, elapsed time, local-time inputs, weather and ordered events: supports reproducible variation, deterministic catch-up and rule rollout/rollback.
- Stable seed per bird/event: produces reproducible variation tied to identity and events.
- Persisting new state and consumed cursor atomically with unique tick/step ID and version check: makes retry and worker failover idempotent.
- Serializing all steps for one aviary: prevents immediate offer steps from racing the minute scheduler.
- Approximately one-minute background ticks advancing all aviaries independent of browser connections: keeps the scene already-running when no client is connected.
- Tick work consuming events, applying personality deltas, advancing moods, calculating daypart/weather, bird calls, actions, adoption availability and notebook rules: NOT RECOVERABLE FROM PLAN
- Long-gap elapsed-time catch-up rather than replaying every missed minute: preserves equivalent day/night and mood outcomes efficiently without inventing presence.
- Presence eligible only with visible state, focus and pointer/key activity inside a grace window: prevents page open from counting as attention while allowing real quiet watching.
- Lease/heartbeat with sequence number and condition flags: lets the server validate freshness and cap credited duration.
- Union overlapping valid intervals across devices: avoids double-counting presence from multiple open clients.
- Ending leases on hidden, unfocused, settle or logout, with expiry for abandoned leases: prevents stale clients from continuing presence.
- Grace window biased long enough to allow quiet watching: aligns presence detection with observing rather than constant clicking.
- Visitors never creating presence leases: preserves the rule that visitors do not affect host state.
- Drift low-pass filter with accumulated presence dominant, listen-in affecting focused bird warmth/vocal frequency, and accepted offers affecting curiosity/boldness: ties trait changes to qualified presence and specific typed interactions.
- Settle with no positive or negative trait delta: makes settle quiet the scene without rewarding or penalizing traits.
- Small nonnegative bounded deltas with no neglect losses: prevents jumps, retries, event bursts and absence from saturating or lowering personality vectors.
- Mood as separate enumerated state driven by recent interactions, local time, weather, calls and personality-conditioned probabilities: keeps fast mood independent of slow persistent traits.
- Mood persisting and evolving during absence without reset or distress: preserves continuity without guilt or distress states.
- Call grammar using species motif library and stable bird identity as recognizable signature: preserves call identity across mood and personality variation.
- Mood and vector altering timing, pitch and expression only within recognition limits: keeps variation expressive but recognizable.
- Snapshot call schedule/seed and client synthesis: lets clients synthesize audio from server-authoritative presentation events.
- Arrival greetings with one first greeter and staggered secondary responses: keeps greetings varied but readable and avoids simultaneous greeting clutter.
- Calibration with seeded simulation cohorts, not population behavior analytics: tunes behavior without using behavioral analytics.
- One-week and three-week calibration profiles with no visible single-session jumps and no two-week-away penalties: ensures drift is measurable but subtle and absence never lowers traits or creates distress.
- Versioned simulation configuration with safe rollout and rollback: lets trait bounds, filters, cooldowns and adoption schedule change without unsafe state changes.
- Product review of naturalist text and call identity alongside thresholds: keeps qualitative voice and recognizable audio part of calibration.

### 6. Canonical sync and conflict handling

- Server as only writer of personality, mood and scene state: prevents client divergence and client-authored personality or mood changes.
- Events rather than absolute client state, with no client-to-client merge and no last-write-wins vector update: avoids overwriting histories or merging conflicting bird states.
- Aviary version, per-device sequence and idempotency keys: supports transactional order and duplicate suppression.
- Worker compare/lock, apply deltas once, advance cursor, write snapshot and publish version atomically: prevents lost deltas and duplicate effects under concurrency.
- Client interpolation between authoritative snapshots: gives smooth presentation while keeping canonical state server-owned.
- Pulling on initial open, visible-tab return, long gaps, accepted actions and low-frequency visible keepalive: keeps the renderer current after meaningful state changes or resumes.
- Ignoring older response versions: prevents stale responses from rolling the renderer backward.
- Timeout plain system message with retry and no choice between histories: avoids asking users to resolve canonical state conflicts.
- Retry with same idempotency key returning prior outcome: prevents duplicate effects on network retry.
- Outage catch-up before returning a current snapshot while preserving event order: keeps delayed simulation consistent after downtime.
- Central device-session revocation: lets settings revoke access across devices.
- Visitor authorization checked at each snapshot pull: makes invite revocation effective no later than the next visitor pull.

### 7. Frontend scene and interaction pipeline

- Separating `Snapshot -> Presentation State -> Renderer` from `UI controls -> typed API event`: keeps the renderer from owning canonical state or persisting behavior choices.
- Single horizontal composition with front/middle/back perch zones and no pan, zoom or scene scroll: keeps all birds in one canonical scene.
- Responsive logical coordinate system with letterbox/scale from phone to desktop: keeps all birds visible across viewport sizes.
- Quiet sky/foliage/perch layers and compact vector/procedural bird silhouettes with cached layers: supports the visual scene with bounded rendering work.
- First visible bird already mid-action using snapshot pose/action phase and server timestamp: reinforces the already-running scene.
- Client-only leaf/feather drift and parallax as bounded render-time randomness with no simulation events: adds ambient motion without changing canonical state.
- No drag-to-place and birds choosing perch by mood/personality: preserves bird-authored behavior and avoids user-placed birds or scene customization.
- Day/night following account timezone with rare quiet weather: grounds the scene in local time while keeping weather ambient.
- Nightjar-like species calling at night: NOT RECOVERABLE FROM PLAN
- No empty-scene return after adoption and soft fly-in for first two birds during account creation: NOT RECOVERABLE FROM PLAN
- Sparse top bar for account/settings, accessibility, notebook and offer: keeps chrome minimal around the scene.
- Top bar fading nearly transparent after cursor stillness and restoring on pointer or keyboard without hiding focus: preserves quiet visuals while retaining keyboard accessibility.
- Listen-in by click, tap or keyboard, with ramp back on click, focus change or empty space: lets users focus on one bird while returning smoothly to ambient.
- Offers opened from top bar with seed/song fragment/still pool and server-chosen recipient response: preserves the top-bar offer flow and state-shaped server selection.
- Settle warming the scene and quieting calls over seconds, with five-second scene-click undo: makes settling gentle and briefly reversible.
- Closing a tab equivalent at the engine layer and never triggering a guilt surface: avoids guilt and treats tab close as presence ending.

### 8. Procedural audio

- One bounded WebAudio graph per active tab: bounds active audio resources.
- Master ambience, per-bird buses and listen-in gain stage: supports ambient mixing and focused listening.
- Species motifs built with oscillator/noise/filter/envelope primitives or generated buffers, never recorded calls: keeps call identity procedural and avoids recorded fallback.
- Parameter automation for ramps, pitch/timing variation and chorus: creates gradual transitions and expressive variation.
- Listen-in raising the chosen bird and lowering but never muting others: focuses attention without destroying ambient presence.
- Song-fragment offers through the same graph and motif library: keeps offer audio consistent with bird call grammar.
- Call identity tied to species and bird seed while mood, frequency trait and grammar variation change expression: preserves recognizable signatures.
- Reusing buffers/nodes, disposing finished nodes, bounding voices and measuring CPU/memory: protects performance on the five-year-old laptop profile.
- Schedule and synthesized caption derived from the same grammar choice: keeps captions synchronized with actual runtime calls.
- Graceful silence when WebAudio is unsupported, fails or is suspended, with captions default on and normal-gesture resume: keeps the scene accessible without intrusive audio prompts.
- Never falling back to recorded audio: NOT RECOVERABLE FROM PLAN

### 9. Accessibility and language

- Semantic, keyboard-reachable top-bar controls and birds with Tab, arrows, Enter and Escape: makes core navigation, listen-in, offer selection and settle operable by keyboard.
- Visible focus outlines over light and dark scene states: keeps keyboard focus visible across day/night presentation.
- WCAG AA or better for user copy, captions, narration and account surfaces: supports accessibility requirements for text contrast and readability.
- Screen-reader live narration from authoritative presentation state: aligns narration with canonical scene state rather than raw events.
- Lowercase, present-tense naturalist prose for narration: shares the notebook voice and content vocabulary.
- Prompt greeting, offer reaction and settle announcements with small priority bump: surfaces user-relevant changes to assistive technology.
- Idle narration every 30-60 seconds with coalescing: prevents assistive technology flooding.
- Never exposing trait numbers or raw event changes in narration: keeps server-only traits and raw event data out of accessibility surfaces.
- Shared notebook and narration vocabulary: keeps product voice consistent.
- `prefers-reduced-motion` and account override: respects OS preference while allowing synced account preference.
- Reduced-motion cross-fades, removed drifting leaves, slower ambient color transitions, while retaining scene, calls, mood and notebook: makes reduced motion a designed presentation rather than a static fallback.
- User-selectable captions near the calling bird from actual runtime motif: ties text to the real call and its source.
- Captions defaulting on if audio cannot run: preserves call information in graceful silence.
- Accessibility preferences syncing with the account while OS preference remains respected: keeps preferences portable without overriding OS signals.

### 10. Notebook, accounts and visits

- Server-generated notebook entries from real stable observations: grounds field-notebook prose in actual aviary state.
- Rare notebook cadence, roughly every few days unless genuinely noteworthy: keeps observations sparse.
- Avoiding session-start logs, trait numbers, "you visited" claims, streaks or generic event text: prevents gamification, exposed trait values and behavior profiling in prose.
- Saving final notebook prose as read-only entries with indefinite chronological scrolling: preserves a stable field notebook while account data remains active.
- Matter-of-fact host session and settings copy: maintains the plan's matter-of-fact account surface voice.
- First sign-in creating an account and two starter birds, then offering naming: NOT RECOVERABLE FROM PLAN
- Later renaming without changing IDs or traits: preserves stable identity and personality across name changes.
- Age, not attention, controlling quiet availability of new birds with no adoption counter or reward language: avoids making adoption a visit or interaction reward.
- Seven maximum birds: NOT RECOVERABLE FROM PLAN
- Device-session revocation, verified email change, JSON export and soft delete/restore/hard delete in settings: gives the host explicit account, security and data lifecycle controls.
- Visits off until the host creates a named email invite: keeps sharing explicit and host-initiated.
- Visitor one-time link and scoped read-only session showing true current scene and calls: allows ambient viewing without write access or special presentation.
- No visitor cursor, avatar, notebook controls or special presentation: avoids co-presence and keeps the visit a simple read-only scene.
- No visitor presence, drift, interactions, greetings or co-presence attributed to them: ensures visitor activity cannot affect host state.
- Host visit log with visitor email, date and approximate duration but no badge or default notification: gives account visibility without engagement pressure.
- Opt-in visit notifications only inside signed-in product, not push/email: preserves no-push policy.
- Revocation blocking next visitor snapshot request and unused invites expiring after 30 days without revival: limits stale or unwanted access.

### 11. Performance, privacy and observability gates

- Release gates measured on representative hardware and last two major Chrome, Safari, Firefox and Edge: ensures v1 behavior is verified on supported browsers and hardware.
- Initial JavaScript below 2 MB gzipped with noncritical surfaces split: protects critical scene load.
- First bird visible under 500 ms on mid-tier mobile over 4G with no spinner or fade-from-static: protects the first impression of an already-running scene.
- Idle 60 fps on five-year-old laptop with seven birds and no 30-minute memory growth: ensures the maximum aviary remains smooth and stable.
- Bounded audio buffers, voices, contexts, workers and notebook reference release: controls CPU and memory growth.
- Simulation tick p99 below 5-second alarm threshold and separate offer reaction/snapshot refresh metrics: monitors server simulation latency distinctly from user-facing reaction paths.
- Synthetic browser fleet and aggregate-only RUM for load, first-bird, frame, audio, API and tick metrics: observes performance without account-level behavior analytics.
- No bird/personality fields, email, per-bird event or per-account interaction dimensions in analytics: prevents telemetry profiling of birds, people or interactions.
- Logs using synthetic UUID and no interaction payload where account identification is necessary: limits operational identifiers to non-email account references.
- Simulation/event storage kept out of analytics warehouse at infrastructure and permission level: enforces the analytics boundary below application code.
- Export access explicit and user initiated: keeps data export under user control.
- Encryption, invite scoping, rate limits, token expiration/single-use, one-time invite consumption and deletion jobs as lifecycle guarantees: makes privacy and data lifecycle enforceable rather than advisory.
- Privacy review verifying hard deletion across primary stores, queues, caches and export staging: ensures deletion removes promised records within lifecycle.

### 12. Delivery sequence and release plan

- Foundation first with schema, UUID accounts, auth/session flow, deletion/export skeleton, simulation config, idempotency, private snapshot and telemetry boundary: builds the trust, state and privacy base before richer behavior.
- Engine vertical slice before visual polish: the plan says seeded simulation calibration should happen "before visual polish."
- Scene and interaction slice with two then seven birds for call and layout testing: verifies recognizable calls and layout at both starter and maximum bird counts.
- Accessibility and reliability before v1 ramp: completes narration, keyboard, reduced motion, captions, WebAudio fallback, serialization, export/delete, revocation, visits and privacy/data-flow review before broader release.
- Performance hardening and controlled v1 ramp with dogfood and cohorts: releases only after baselines, memory run, browser review and capacity checks.
- Releasing with two birds and unlocking later birds only by aviary age using conservative intervals: keeps adoption age-based and cautious during ramp.
- Qualitative calibration through product review rather than behavioral analytics: tunes the experience without population behavior analytics.
- Rollback simulation config or release flag if state integrity, call identity, a11y or performance regresses: gives a recovery path for core regressions.

### 13. Key risks and mitigations

- Reproducible one-, three- and six-week presence profiles for drift: mitigate drift feeling imperceptible or too fast.
- Blind review of subtle visual changes without exact vector values: tunes expressive drift without exposing or relying on visible trait numbers.
- Testing visibility, focus, activity, sleep, still watching, multiple devices and abandoned leases: mitigates presence overcounting.
- Duplicate/out-of-order requests, worker restart and concurrent laptop/phone injection: proves canonical state does not lose deltas or duplicate offers.
- Time-zone/DST, long-gap catch-up and resume tests: prevent mood reset, mood discontinuity and absence distress.
- Blind-listening reviews at two and seven birds: mitigate procedural calls becoming uncanny, repetitive or blurred together.
- Treating suspended audio as normal with captions and no intrusive prompt: protects the first impression when autoplay or WebAudio fails.
- Screen-reader user review, cadence/coalescing and focus testing: prevents accessibility announcements becoming noisy or flat.
- Seven-bird constrained-device profiling, cached layers, split code and hidden-tab render stop: mitigates scene/performance pressure against budgets.
- Invite scoping, revocation-at-pull, hashed secrets, expiry and stale-session tests: mitigates invite leakage or revocation lag exposing private state.
- Notebook and telemetry schema/prose review: prevents gamification or behavior profiling.
- Idempotent deletion tests across database, queues, cache, invitation/visit data and exports: prevents recoverable account-linked traces after hard deletion.
