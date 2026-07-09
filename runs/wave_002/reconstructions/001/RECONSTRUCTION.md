## System-level intent

- **Continuity before breadth.** The opening contract says the "primary quality bar is continuity": first frame "already in progress," the "same birds persist across devices," and attention changes them "slowly without making absence harmful." The delivery sequence repeats this by requiring repeated/simultaneous clients to read "one state" before feature expansion.

- **One canonical, server-authored aviary.** The plan repeatedly makes the server authoritative: "one canonical server-authored aviary per account," clients "submit facts" and "render snapshots," and the server owns "semantic facts" while the client owns presentation. This shows up in the architecture, data invariants, API contracts, tick execution, and multi-device convergence rules.

- **Care without obligation or punishment.** The plan rejects negative absence effects: "Absence produces no negative trait delta, distress, hunger, death, guilt, or loss." It also excludes care schedules, punitive decay, push engagement loops, streaks, achievements, and "notifications intended to drive return." The risks table names the danger as product center shifting to "obligation/comparison."

- **Personality is encountered through behavior, not numbers.** The plan says users "encounter personality through behavior" and forbids personality numbers in live product, accessibility tree, snapshot API, logs, metrics, and support tooling. It allows only a "sole narrow exception" for user-requested export.

- **Attention must be honest.** Presence is not a page view: it is "the conjunction of visible document, focused window, and recent pointer or keyboard activity," and "an open tab alone never counts." The presence section says the goal is "accurate normal operation, not invasive surveillance."

- **Aliveness should be procedural, specific, and non-canned.** Calls are "small versioned grammar program[s], not an audio asset." The plan blocks recorded calls, requires recognizable individual birds, and rejects repeated calls that sound identical or captions unrelated to playback.

- **Accessibility is part of v1 value, not an alternate shell.** Accessibility ships "with v1, not after it," and acceptance is "affective as well as mechanical." Reduced-motion and screen-reader sessions must contain "continuing, specific aviary behavior" and not degrade into "a state list or static placeholder."

- **Social contact is bounded and host-preserving.** Visits are "named, revocable, read-only, and off by default." Visitor activity "must not enter the host's simulation," and visitor risks are framed as the host relationship being "reshaped or exposed."

- **Voice has two registers.** The plan reserves "naturalist voice" for aviary, notebook, narration, captions, and offers, while authentication, settings, accessibility controls, errors, export, and deletion use "direct matter-of-fact language." Editorial review explicitly checks these two voice registers.

- **Determinism and idempotency protect identity.** Retries must be deterministic; old projections remain readable during work; interaction events are immutable; bird IDs are never reused; and a vector reset is a "severity-one data-loss event." The intent is that devices do not disagree and bird identity is not lost or overwritten.

- **Privacy-safe operations are a product invariant.** Email is encrypted and HMACed, production per-account interaction data is not copied to warehouses or training streams, metrics are aggregate only, and dashboards answer "is the service healthy?" rather than "how do people use their birds?"

- **Performance is part of the feeling of the aviary.** The plan treats "first bird visible" and "spinner-first established aviary" as product gates, not only engineering metrics. Budgets, edge projections, lazy chunks, and soaks exist so the aviary does not visibly load or degrade.

## Per-feature whys

### Product contract and implementation decisions

- **Web-only, single-account/single-aviary experience** - Why: the plan ties this shape to continuity: one account sees one ongoing aviary, with the first rendered frame already in progress and the same birds persisting across devices.

- **Exactly two system-selected birds at account start, hard maximum of seven, and additional birds by aviary age only** - Why: the plan blocks visits, actions, payment, or engagement score from driving birds, keeping eligibility from becoming a reward loop.

- **One canonical server-authored aviary per account** - Why: clients render snapshots and submit interaction facts, but never submit personality values or advance simulation time, preserving canonical state.

- **Honest presence definition** - Why: treating presence as visible document plus focused window plus recent activity prevents an open tab alone from counting.

- **Nonnegative and slow personality movement** - Why: the plan wants attention to change birds slowly and absence to remain harmless, with no distress, hunger, death, guilt, or loss.

- **No live personality numbers** - Why: users should encounter personality through behavior, while numeric surfaces would become stats, controls, or privacy leakage.

- **Procedural call synthesis only** - Why: recorded calls, even as fallback, would undermine the grammar-driven, non-canned aliveness the plan requires.

- **Screen-reader narration, call captions, keyboard access, and reduced-motion renderer in v1** - Why: accessible modes are part of the release value and cannot trail the beta.

- **Named, revocable, read-only visits off by default** - Why: visitor activity must not enter the host's simulation, and visits should not become an open social surface.

- **Naturalist voice for aviary surfaces and matter-of-fact language for system surfaces** - Why: the plan separates the poetic field experience from account, error, export, deletion, and settings operations.

- **No scores, counters, streaks, achievements, levels, feeds, follows, comments, public profiles, or behavior summaries disguised as notebook observations** - Why: the plan blocks obligation, comparison, engagement optimization, and stats-like recasting of the notebook.

- **Canonical aviary timezone** - Why: one IANA timezone gives the server one coherent day/night and mood timeline; travel is handled by explicit update rather than simultaneous devices silently alternating it.

- **Daily-ish mood pull toward time-of-day baseline instead of reset** - Why: stored mood persists through ticks so opening a tab cannot snap it to neutral.

- **Settle control in the top bar** - Why: hiding settle in account settings would make a core session gesture undiscoverable.

- **Personality export as a narrow exception** - Why: the account export requires vectors for portability, but the plan keeps raw values absent from interactive surfaces and ordinary APIs and explains they are not controls or scores.

- **Graceful silence and automatic captions when audio autoplay is blocked** - Why: the first session should not be interrupted by a modal or patched with recorded fallback; future procedural audio can start after a qualifying gesture.

- **Mute as presentation preference, not simulation input** - Why: reducing drift or expression when muted would penalize Deaf users, muted environments, and autoplay-blocked sessions.

- **No push or badges for visits, optional plain email only after explicit opt-in** - Why: the default is silent logging, and visit notifications stay out of onboarding to avoid return-driving infrastructure.

### V1 scope and acceptance boundary

- **Email magic-link authentication rather than passwords or SSO**: NOT RECOVERABLE FROM PLAN

- **Neutral, hashed, single-use magic-link handling** - Why: same neutral response, HMAC/IP rate limits, 15-minute expiry, and atomic first use address email enumeration and replay.

- **Per-device sessions and revocation** - Why: device labels/times let the owner revoke one device, and revoking the current device ends it cleanly.

- **Verified email changes**: NOT RECOVERABLE FROM PLAN

- **30-day recoverable deletion followed by hard deletion** - Why: soft deletion allows authenticated recovery, while the purge worker deletes account-linked simulation, events, notebook, visit, outbox, and object data after the deadline.

- **On-demand account export** - Why: the export is user-requested portability, generated from a consistent snapshot, delivered by a short-lived single-purpose emailed link, and audited without logging content.

- **Adoption of two system-selected starter species** - Why: every account starts with exactly two birds and no catalog, rarity, payment, or engagement path to shape the initial flock.

- **User naming and renaming**: NOT RECOVERABLE FROM PLAN

- **Stable bird identity and profile pinning** - Why: a bird pins a profile version so later asset releases cannot silently replace its identity; IDs are never reused or changed by rename, migration, sync, or export/import work.

- **Age-based bird availability up to seven birds** - Why: age-only eligibility keeps additional birds from being earned through visits, actions, payment, or engagement score; rollout flags control safety, never user reward.

- **Single responsive horizontal aviary with front/middle/back perch zones** - Why: responsive constraints keep every bird fully in frame without pan, scroll, zoom, crop, drag, or direct placement.

- **Rare ambient weather** - Why: frequency constraints prevent clustering and assertive weather; the plan excludes high-impact weather and per-leaf server state.

- **Return-greeting** - Why: the server calculates absence from accepted owner presence and issues at most one primary greeter plus optional staggered response so returning feels alive without replay.

- **Listen-in** - Why: listen-in is an attention signal whether represented by sound or captions, and audio ramps focus on one bird without fully muting the flock.

- **Seed, song-fragment, and still-pool offers with per-bird cooldowns** - Why: offers provide small behavior inputs without targeting a trait; server selection and cooldowns prevent trait manipulation and countdown-style play.

- **Settle with five-second undo** - Why: settle is a core session gesture that closes presence and affects short-term mood only, while ordinary close has identical engine consequences and no guilt prompt.

- **Sparse read-only field notebook** - Why: the notebook should stay rare, specific, and naturalist, not become a generic feed, event log, user journal, or visit-frequency export.

- **Persisted personality, mood, render, call, weather, and notebook state advanced by roughly one-minute server tick** - Why: the aviary continues without connected clients and avoids client-authored catch-up.

- **Small versioned snapshots** - Why: compact projections support first-bird performance, owner/visitor separation, vector-free contracts, and stable transition resumption.

- **Append-only interaction ingestion** - Why: accepted events form an input ledger; idempotency keys and immutable events make retries safe.

- **Owner multi-device consistency** - Why: simultaneous devices feed one ordered ledger, union overlapping presence, enforce cooldown once, and converge on the next canonical snapshot.

- **Suspend/resume recovery** - Why: hidden tabs stop animation and audio scheduling, then discard stale interpolation and fetch fresh state instead of replaying entry animations.

- **Idempotent conflict handling** - Why: retries, duplicate events, and replayed greetings cannot create double drift, double schedules, or divergent versions.

- **Client procedural audio engine and chorus management** - Why: seven birds need recognizable, non-canned call variation with headroom, voice caps, and stable individual signatures.

- **Captions derived from the exact call program and silence-plus-captions fallback** - Why: captions must match the sound actually scheduled, and unavailable WebAudio should preserve full visual/caption behavior.

- **Screen-reader prose narration, complete keyboard navigation, focus treatment, WCAG AA text, and reduced-motion path** - Why: accessible sessions must preserve the actual aviary rather than becoming a static list.

- **Named email invitations and one-time redemption into read-only visitor sessions** - Why: invitations are scoped and explicit; they do not prompt contact import, suggest recipients, or create public links.

- **Silent visit logging, revocation, and 30-day unused-invite expiry** - Why: hosts retain transparency and access control, while unused invitations expire and revocation ends access without falsifying visit history.

- **Privacy-separated telemetry, synthetic performance monitoring, launch flags, and production safety controls** - Why: operations should watch health and safety without copying per-account behavior into product analytics.

### System architecture, persistence model, and API/client contracts

- **Small modular service shape first** - Why: hard boundaries are preserved in code and data so components can split only when load requires it.

- **Edge web application/BFF** - Why: it can deliver a minimal shell plus private initial projection, validate sessions and CSRF, and reach the first-bird target without putting email or interaction history at the edge.

- **Browser client state adapter, renderer, audio, accessibility layer, and event outbox** - Why: the client owns presentation and responsiveness but not drift, canonical mood transitions, or authoritative bird state.

- **Simulation worker and scheduler** - Why: due aviaries advance through deterministic minute-scale steps without connected clients, and leases prevent two workers from advancing the same version.

- **Relational primary database with PostgreSQL rather than full event sourcing** - Why: transactions, foreign keys, encrypted PII, idempotency constraints, and explicit current simulation state are enough for the starting shape; the append-only log is an input ledger.

- **Email and lifecycle worker with transactional outbox** - Why: magic links, invites, export links, cleanup, and purge run out of band using opaque IDs, with encrypted email retrieved only at send time.

- **Operational telemetry pipeline with aggregate allowlist only** - Why: it must have no credentials or route to simulation tables, and production interaction data must not feed warehouse, training corpus, or product-analysis stream.

- **Server-semantic/client-presentation render boundary** - Why: the server owns identity, perch, pose, mood, calls, greeting, weather, day phase, settle state, and transition times; the client only interpolates and presents them.

- **Snapshot server time and stable transition identifiers** - Why: refetching the same snapshot resumes in the same phase instead of replaying an entry animation.

- **UUIDs, UTC timestamps, IANA aviary timezone, encrypted email, and HMAC lookup** - Why: email is never a foreign key, partition key, log field, or message key, while timezone remains coherent.

- **Device-session tokens without email** - Why: access tokens are short-lived, audience-scoped, and do not contain email.

- **Account settings preference fields and optimistic versioning**: NOT RECOVERABLE FROM PLAN

- **Trait columns inaccessible to the web read role** - Why: personality vectors remain hidden from interactive and ordinary API surfaces.

- **Immutable versioned species profiles pinned per bird** - Why: later asset releases cannot silently replace a bird's identity.

- **Immutable interaction events with idempotency keys and no email in payloads** - Why: retries are safe and interaction storage avoids PII.

- **Presence intervals normalized and unioned across owner devices** - Why: overlapping tabs/devices do not multiply attention before drift calculation.

- **Offer cooldowns enforced transactionally on the server** - Why: the client may predict availability for responsiveness, but the server prevents bypass and preserves canonical behavior.

- **Notebook entries with source facts and template version** - Why: immutable prose can be tested against the canonical facts and grammar version that produced it.

- **Owner and visitor snapshot projections without hidden vectors or raw interaction history** - Why: the snapshot remains a behavior projection rather than a stats or history API.

- **Visit records separate from active invitation list** - Why: revocation removes access but does not falsify historical transparency.

- **Outbox messages and export jobs with encrypted references, redaction, and expiry** - Why: messages and exports stay operationally useful without exposing provider payloads or long-lived objects.

- **Database roles allowing only simulation worker to update personality, mood, canonical perch/pose, weather, or simulation version** - Why: server-only writing protects personality and canonical state from web or client mutation.

- **Immutable bird IDs and aviary ownership** - Why: identity cannot be lost through rename, profile migration, device sync, export/import, or account relationship changes.

- **Vector clamping and nondecrease audit assertion** - Why: personality movement must be monotonic, bounded, and incapable of negative drift.

- **Visitor write prohibition at permission and route levels** - Why: visitors cannot affect interaction, presence, notebook, settings, bird, or simulation tables.

- **Versioned JSON contracts and generated schema validation**: NOT RECOVERABLE FROM PLAN

- **Mutations returning operation/event ID and current canonical version, not speculative personality result** - Why: APIs acknowledge facts without exposing or predicting hidden personality.

- **Secure cookies, CSRF tokens, strict origin checks, rate limits, and generic responses** - Why: the plan threat-models CSRF, replay, IDOR, and email enumeration.

- **Adoption endpoint available once with two validated names and timezone** - Why: it transactionally creates the single aviary, two distinct starter birds, and the canonical timezone anchor.

- **Bird patch endpoint permitting name changes only** - Why: clients cannot change species, vector, mood, perch, color, call, or identity fields.

- **Age-unlocked adoption endpoint with quiet product copy and no progress bar/countdown** - Why: additional birds remain age-only arrivals, not gamified goals.

- **Snapshot API excluding vectors, drift deltas, visit counts, streak summaries, raw event history, hidden mood labels, and days-until-next-bird counters** - Why: the render projection displays behavior without becoming a stats API.

- **Arrival endpoint with server-calculated absence and expiring greeting directive** - Why: greetings are based on accepted presence and cannot be replayed by reusing an idempotency key.

- **Batch event ingestion with time clamps, ordering checks, bounded offline queue, and stale-presence drop** - Why: discrete events can survive short replay windows, but stale presence is not credited.

- **Read-only notebook API** - Why: immutable entries are returned by cursor; there are no create, update, or delete endpoints.

- **Invitation creation with one email and no contact import or suggested recipients** - Why: visits remain explicitly user-initiated and do not become a growth or discovery mechanic.

- **Visitor HTML without owner controls, notebook/account endpoints, greeting request, event outbox, or presence detector** - Why: even though server authorization is primary, the visitor client should not contain mutation pathways.

### Simulation engine

- **60-second tick cadence with leases and compare-and-swap** - Why: workers advance due aviaries without double-claiming or racing the same version.

- **Analytical quiet-span catch-up after long outage** - Why: the system avoids millions of literal ticks and does not invent client-side catch-up.

- **Deterministic pseudorandom seeding from aviary seed, tick, subsystem, and bird ID** - Why: a retry against the same input yields the same weather, greeting candidate, mood choice, and call schedule.

- **Local presence state machine and low-rate samples** - Why: it captures visible/focused/recent-activity presence without sending every browser event.

- **Server presence validation and interval union** - Why: hidden/focus-false claims, future times, excessive gaps, visitor tokens, and replayed sequences cannot inflate attention.

- **Personality drift exposure channels** - Why: presence supplies low-weight input, listen-in adds stronger targeted exposure, offers add small boldness/curiosity exposure, and settle contributes no trait direction.

- **Saturating low-pass drift update with fractional accumulators** - Why: changes stay slow, nonnegative, bounded, and not lost to storage precision.

- **Drift calibration with synthetic schedules and consented internal aviaries** - Why: the plan wants measurable internal deltas by day 7, felt differences around week 3, no obvious one-day jump, and no production population mining.

- **Small v1 mood enum with time-of-day baseline and hysteresis** - Why: mood follows local day, interactions, weather, personality, dwell time, and nearby birds without flapping or reset on open.

- **Weather, settle, offers, and bird-to-bird effects in mood** - Why: rain, wind, offers, and nearby birds can color short-term behavior while strict caps and decay avoid synchronized state flips.

- **Weighted perch choices with spatial capacity and deterministic conflict resolution** - Why: birds reflect mood/personality without overlap or teleport.

- **Persisted pose/activity plans with start/end times and pose phase** - Why: clients can join mid-action rather than restarting animations.

- **Owner greeting candidate weighting and staggered responses** - Why: return greetings reflect boldness, warmth, mood, time, and absence while visitors never generate greetings.

- **Future call schedule in each projection** - Why: calls preserve each bird's base interval/timbre signature while allowing seeded variation and coordinated chorus windows.

- **Notebook generation from structured server-observed facts and deterministic grammar** - Why: entries stay specific, novel, sparse, supported by canonical state, and not an unconstrained model or generic activity log.

### Multi-device synchronization and failure behavior

- **Serialized canonical writes, compare-and-swap simulation, optimistic settings/name changes, and append-only interaction facts** - Why: no client chooses a winner, and canonical state converges.

- **IndexedDB event outbox** - Why: transient network loss can retry with the same IDs in client-sequence order, while stale presence is discarded instead of synthesized after reconnect.

- **Snapshot pulls on navigation, visible keepalive, visibility return, online return, and suspension/large frame gap with randomized jitter** - Why: low-frequency polling is sufficient for v1, with tighter polling only during active transitions or revocation-sensitive visitor sessions.

- **Snapshot reconciliation by stable transition ID** - Why: newer snapshots continue an in-progress transition at current server-time phase and do not replay greetings, offer reactions, or fly-ins.

- **Serving last complete projection during simulation outage** - Why: clients never see a half-updated flock, and the system does not claim drift occurred until committed.

- **Matter-of-fact auth/session conflict handling** - Why: stale devices should get clear system copy and a sign-in/reload action, not a naturalist error, reset bird, or overwrite.

### Frontend rendering and interaction pipeline

- **Lightweight SVG/DOM scene graph and lazy chunks** - Why: seven birds do not justify a large game engine, and account/notebook/accessibility/invitation/export/deletion UI should not block the critical path.

- **First render: quiet field, then at least one bird and perch from initial projection** - Why: established aviaries never show empty, spinner, skeleton card, wake animation, global fade-in, or "ready" state.

- **Normalized scene coordinates, depth zones, and client-only ornaments** - Why: all birds remain in frame across responsive layouts, while leaf/feather drift and parallax stay non-semantic and session-seeded.

- **Single requestAnimationFrame scheduler with compositor-friendly transforms and hidden-tab stop** - Why: the scene needs sustained performance without per-bird timers, layout reads, background rAF, or leaking particles/pose objects.

- **Restrained full-motion activities** - Why: flight, preen, scan, tilt, and shuffle should avoid strobing and high-frequency loops.

- **Reduced-motion still poses, slow cross-fades, and removal of drift/parallax** - Why: reduced motion avoids full-motion flash while preserving mood, calls/captions, drift, weather facts, notebook, and greetings.

- **Top bar fade behavior** - Why: controls can fade nearly transparent during inactivity but must restore on input and never fade while focused, open, or keyboard-only navigation is active.

- **Listen-in interaction behavior** - Why: click/tap or Enter focuses attention, repeat/Escape/focus-away disengages, and immediate audio response later reconciles with server acknowledgment.

- **Offer interaction behavior** - Why: users choose seed/song/still pool without targeting a trait, and unavailable gestures use quiet copy rather than countdown meters.

- **Settle interaction behavior** - Why: warm lighting/call reduction begins immediately, undo reverses from current progress, and ordinary close has no recovery prompt.

- **Notebook virtualized panel with offscreen disposal and focus restoration** - Why: read-only pagination should not retain stale row references and should return focus to the invoking control.

### Procedural audio pipeline

- **Call grammar program instead of audio assets** - Why: species motifs, pinned profiles, identity-derived parameters, seeds, and mood create recognizable signatures with bounded variation.

- **Immutable audio intermediate representation** - Why: notes, timing, articulation, timbre, and caption tokens come from the same parsed call event.

- **Scheduled oscillator/noise/filter/envelope nodes with deterministic cleanup** - Why: audio should align to server-clock offset and avoid long-session node/resource growth.

- **Per-bird gain/pan buses, ambient/ducking groups, compressor, limiter, and simultaneous voice cap** - Why: seven birds need headroom and chorus management without clipping or blur.

- **Listen-in audio ramps** - Why: focused birds rise gradually while the rest of the flock remains at an audible ambient floor; no hard cuts or full mute.

- **Microtiming and phase variation with event deduplication** - Why: calls avoid stacked-loop artifacts while retaining motif identity, and the same event ID cannot schedule twice after refresh.

- **Captions derived from the scheduled intermediate representation** - Why: note count, contour, repetition, quality, source bird, and perch match playback instead of generic stored captions.

- **Audio autoplay and lifecycle fallback** - Why: failed creation/resume switches to silence and captions, records only aggregate error category, and starts future calls after gesture without replaying missed ones.

### Accessibility implementation

- **Semantic DOM built from the same snapshot adapter as the visual scene** - Why: accessibility text should not be inferred from pixels and should not expose raw state labels.

- **Named region and stable focusable bird elements with roving tabindex** - Why: birds are available by names/species, with keyboard listen-in and standard focus behavior rather than personality values.

- **Prose narration composer with reviewed naturalist templates** - Why: live updates coalesce every 30-60 seconds and prioritize greetings, offer reactions, and settle observations without repeated interruption.

- **Consistent lowercase, present-tense, specific captions/narration** - Why: visual and screen-reader surfaces should share the same field voice and facts.

- **Control to pause running narration** - Why: users can quiet live narration without losing keyboard use or static bird descriptions.

- **Call captions near the calling bird plus caption log/live region** - Why: captions identify source while rate limits prevent flooding assistive technology.

- **Reduced-motion preference honored before animation starts** - Why: the app avoids a full-motion flash during hydration.

- **Contrast, touch target, zoom, reading order, names, errors, and focus restoration requirements** - Why: WCAG AA and real usability must hold across bright, dim, rain, settled palettes, 200% zoom, keyboard, muted audio, and reduced motion.

- **Manual screen-reader and vestibular testing** - Why: automated checks are necessary but insufficient for the affective requirement that accessible modes preserve the actual aviary.

### Security, privacy, and lifecycle

- **Threat model before external beta** - Why: the plan names token theft/replay, invitation forwarding, CSRF, IDOR, visitor escalation, email enumeration, event replay/inflation, stored-name injection, export leakage, and worker over-privilege as pre-beta risks.

- **Hashed magic-link/invite/session secrets, at least 128 bits randomness, single-use consumption, secure cookies, token rotation, CSP, escaping, and ownership checks** - Why: these directly mitigate token replay, theft, CSRF, stored injection, and cross-aviary access.

- **Narrow identity service role for email encryption keys** - Why: account and invitation emails remain encrypted except for narrowly authorized identity operations.

- **Application log redaction and short-lived trace IDs** - Why: emails, URLs/tokens, unnecessary names, event payloads, notebook prose, snapshot bodies, and account identifiers should not leak through operations.

- **Audited break-glass tooling without routine vector exposure** - Why: investigations may require account access, but support UI still must not reveal vectors.

- **Physical/logical separation of simulation storage and analytics plus typed metric allowlist** - Why: analytics cannot accept arbitrary account, bird, mood, offer, species/name, session, email, or interaction dimensions.

- **Consistent export snapshot with exclusions** - Why: export includes specified settings, birds/names, vectors/moods, and notebook entries but excludes tokens, other users' data, HMACs, and raw visitor credentials.

- **Soft deletion followed by day-30 purge and non-identifying tombstone** - Why: social access, interactions, and optional notifications stop immediately, recovery remains possible, and aggregate telemetry cannot reconstruct or target the deleted account.

### Performance budgets and observability

- **Initial JavaScript hard cap and lazy chunks** - Why: shell, renderer, snapshot adapter, and audio runtime must fit the first-bird critical path.

- **First-bird-visible target measured after actual paint** - Why: quiet-field paint is not counted, because the product promise is seeing a bird, not a loaded shell.

- **Idle rendering, memory, and audio-node soak budgets** - Why: the aviary should not visibly degrade across a 30-minute session.

- **Kilobyte, gzip-friendly, vector-free snapshots** - Why: projections must stay small at two and seven birds and must not leak personality vectors.

- **Simulation latency and backlog monitoring** - Why: lag, failed/retried ticks, stale snapshots, and due-aviary backlog threaten canonical continuity.

- **Privacy-safe aggregate signals only** - Why: metrics should include route latency, paint, release-fingerprint JS errors, frame timing, audio-context failure category, tick lag, outbox backlog, provider status, and deletion completion, but not which bird, mood, offer, interaction, account, or visit produced them.

- **Synthetic journeys with isolated accounts** - Why: monitoring can test sign-in, snapshot, first bird, soak, reduced motion, and visitor revocation without becoming product conclusions.

- **Dashboards and alerts for service health, not bird usage** - Why: the plan explicitly rejects engagement, retention, average drift, popular species, offer frequency, per-account session, and notebook-content dashboards.

### Verification strategy

- **Simulation unit/property tests** - Why: deterministic retry, monotonic drift, no absence delta, DST, weather bounds, stable identity, age-only adoption, seven-bird cap, cooldowns, chorus limits, and notebook rules are release invariants.

- **Data and concurrency tests** - Why: duplicate/reordered events, overlapping devices, worker double-claim, crash timing, version conflicts, snapshot atomicity, session revocation, invite lifecycle, and deletion/recovery races can break canonical continuity.

- **Contract tests** - Why: owner versus visitor projections, hidden-trait exclusion, schema compatibility, ETag behavior, stale-client handling, matter-of-fact error copy, and naturalist prose linting protect boundary promises.

- **Frontend tests** - Why: first frame already in phase, no replay on refresh, all-birds-visible constraints, top-bar focus, settle undo, hidden-tab stop/resume, fallback, cleanup, and reduced-motion no-flash are core UX gates.

- **Audio tests** - Why: grammar/caption equivalence, absence of recorded assets, deduplication, ramps, headroom, fallback, cleanup, and browser lifecycle prevent canned, clipping, or leaking audio.

- **Accessibility tests** - Why: static rules, keyboard map, focus, live-region cadence, captions, contrast, zoom, and reduced-motion snapshots support the requirement that access modes preserve the aviary.

- **Performance/security tests** - Why: bundle/snapshot caps, synthetic budgets, soaks, authorization, CSRF, token hashing/replay, PII log scans, metrics lint, and export-link expiry guard release blockers.

- **Longitudinal internal test and multiweek dogfood** - Why: drift should be measurable at week 1 and felt, not announced, around week 3; perceived drift cannot be fully compressed.

- **Blind listening sessions** - Why: stable individual call recognition and non-canned variation must hold from two through seven birds.

- **Editorial review** - Why: greetings, notebook, narration, captions, offers, settings, and errors must match the two voice registers.

- **Manual assistive-technology, vestibular, and cross-device testing** - Why: screen reader, cross-fade renderer, suspension, offline recovery, DST, device timezones, simultaneous sessions, and revoked visitor access need human/device evidence.

### Delivery sequence and rollout

- **Phase A contracts and vertical skeleton** - Why: terminology, ambiguity decisions, schemas, data classification, roles, migrations, deterministic tick, private projection, minimal scene, and CI gates freeze core invariants before expansion.

- **Phase B behavioral core** - Why: honest presence, ingestion, drift, mood, perches, interpolation, greetings, offers, settle, weather, and bird-to-bird effects need canonical histories and dogfood because perceived drift cannot be compressed completely.

- **Phase C audio, notebook, and complete interaction surface** - Why: grammars, synthesis, exact captions, sparse notebook, top bar, responsive scene, and age-based adoption must pass recognition and 30-minute soak gates at seven birds with no recorded audio.

- **Phase D accessibility, social, and lifecycle completion** - Why: semantic/narration layer, reduced motion, keyboard/focus/contrast, visitor flows, export, email change, sessions, and deletion must complete before beta because accessibility is not allowed to trail.

- **Phase E staged production rollout** - Why: staff dogfood, invite-only cohort, percentage ramp, later-bird capability flag, and GA gates watch operational SLOs, support reports, accessibility regressions, qualitative research, and restore/deletion drills rather than engagement optimization.

- **Server capability flag for third and later birds** - Why: seven-bird rendering/audio capacity must be proven first; the flag controls system safety, never user reward.

- **Independent kill switches** - Why: new interactions, notebook generation, weather scheduling, optional visit email, invites, and additional-bird eligibility can freeze or remove peripheral effects without resetting birds, vectors, identity, mood, or notebook history.

- **Backups, point-in-time recovery, vector checksums, and restore drills** - Why: personality state loss is a severity-one event, so recovery paths must be tested before public beta.

- **General availability gates** - Why: three-week real-time cohort evidence, browser/performance SLOs, deletion rehearsal, and severity-one restore drill are required before release.
