## System-level intent

- **A continuing place, not a loading or restart story.** The delivery goal says the first visible frame must "read as a continuing place." This shows up again in the first-frame path: "server-rendered quiet field," "no spinner," and drawing a bird "already mid-preen/call rather than starting an entry sequence."
- **Server-authoritative continuity.** The plan repeatedly says the server is the "sole authority" and that clients "submit facts and render derived state." It appears in the canonical-state/render boundary, API prohibition on direct simulation mutation, tick transaction, and sync rules where "only that transaction writes personality/mood."
- **Stable bird identity with slow, non-negative change.** The plan protects the bird's "stable UUID and personality vector" across renames, devices, deployments, migrations, and species-library revisions. Drift is "slow and non-negative," with "zero input yields zero delta" and rollbacks forbidden from rewriting vectors.
- **Honest presence as a trusted input.** Presence requires "visible document, focused window, and recent pointer or keyboard activity" together, and concurrent devices "cannot double-count." The presence model, acceptance scenarios, and risk table all return to clamping, timeouts, and cross-device interval union.
- **Ambient after absence without punishment.** The plan excludes hunger, illness, distress, decaying happiness, required visits, and guilt-style copy. It separates "accumulated personality" from "recent-expression modifiers," so absence can lower expressiveness through mood/recency but "never subtracts" from traits.
- **Quiet anti-gamification.** The plan rejects "toast, streak, score, status meter, public discovery, caretaking penalty" and excludes achievements, counters, leaderboards, rarity, and attention-based unlocks. Age-gated growth is "never a reward schedule," and repeated clicking must not become an "optimization strategy."
- **Privacy by schema, pipeline, and product surface.** Normal APIs, logs, analytics, ARIA data, and UI "never expose numeric personality values"; telemetry is "operational and aggregate-only"; raw events stay inside the simulation store; exports are the one explicit exception and are isolated.
- **Naturalist voice for product experience, direct system language for system surfaces.** The invariant names this split directly. Notebook, captions, narration, offers, and cooldowns use reviewed naturalist grammar, while auth, settings, unsupported-browser, snapshot failures, and errors use "matter-of-fact" or "direct system" language.
- **Accessible modes are first-class, not substitutes.** V1 ships only when ordinary, screen-reader, captioned/silent, and reduced-motion experiences are "complete and tested." Reduced motion must remain alive through "still-pose cross-fades and color/audio," and accessibility regressions block release like a broken visual scene.
- **Versioned contracts and staged gates before growth.** The plan emphasizes schema boundaries, versioned engine/calibration/snapshot/grammar, property tests, golden sequences, milestones, dogfood, preview, beta, and bird-count ramp gates. Expansion waits for audio recognizability, layout, memory, snapshot, and accessibility gates.

## Per-feature whys

### V1 scope

- **Responsive browser aviary:** The plan grounds this in a "web-only" product and release gates for ordinary and accessible browser experiences.
- **Last two major versions of Chrome, Safari, Firefox, and Edge:** NOT RECOVERABLE FROM PLAN
- **Email magic-link authentication:** The scope excludes passwords, SSO, and native protocol work; the auth contract also uses neutral responses and short-lived links.
- **Revocable device sessions:** The plan includes a user's session list and immediate revocation paths so account control does not depend on hidden device state.
- **Verified email changes:** The old address is retained "until verification commits," preventing an unverified change from taking over account delivery.
- **Account export:** The export is the explicit product exception to hidden numeric personality values and includes warnings that trait values are normally hidden.
- **30-day recoverable deletion:** The plan uses a recoverable mark first, immediate revocation, and later retryable erasure; the exact 30-day duration is NOT RECOVERABLE FROM PLAN.
- **One canonical aviary per account:** This supports the single-account/single-aviary product and avoids shared or multi-aviary reconciliation.
- **Two starter birds:** The goal requires two initially adopted birds whose continuity and recognizability are preserved over weeks.
- **User naming and renaming:** Names are durable bird product data, but the plan limits mutation to name changes only so users cannot write personality, mood, perch, or vectors.
- **Approximately six-species coherent pool:** NOT RECOVERABLE FROM PLAN
- **Age-gated path to no more than seven birds:** The plan says growth is based on aviary age, not behavior, score, payment, rarity, or visits; seven is also a hard cap for layout, audio, access, and performance.
- **Persistent personality and mood with server simulation:** This is how birds continue changing "whether a client is open or not" while keeping the server authoritative.
- **Local-day cycle:** The aviary uses the account's IANA time zone so day phase is canonical and visitors see the host aviary's day.
- **Rare ambient weather:** Weather is rare, non-destructive, canonical across owner devices and visitors, and "not a response to user behavior."
- **Bird-to-bird responses and chorus behavior:** These preserve a living multi-bird scene while keeping timing and response plans canonical.
- **Procedural greetings, calls, and chorus behavior:** Calls must come from browser grammars, not recorded assets or loops, so they can vary while remaining bird-identifiable.
- **Honest presence accounting:** Presence is a trusted input only when visible, focused, and recently active; server clamping and union prevent background tabs or multiple devices from accelerating drift.
- **Listen-in:** The plan makes listen-in a focused presentation and input fact that strengthens targeted warmth/vocal dimensions without turning other birds silent.
- **Three offer types:** The existence of server-evaluated offers is tied to small curiosity/boldness inputs and cooldowns; the exact `seed`, `song_fragment`, and `still_pool` choices are NOT RECOVERABLE FROM PLAN.
- **Settle with five-second local undo:** Settle ends local presence and moves the scene toward evening without penalty; undo/reengage prevents accidental settling and is idempotent.
- **Sparse read-only field notebook:** The notebook records specific, occasional naturalist observations rather than event logs, metrics, or editable history.
- **Multi-device snapshot consumption with no client merge:** The plan avoids last-write-wins personality paths; devices may render different interpolation frames but must converge on one semantic revision.
- **Per-email one-time visit invitations:** Visits are private, revocable, and not public links, profiles, follows, feeds, chat, or co-presence.
- **Visitor rendering:** Visitors render the current ambient state only; their activity never enters the host simulation.
- **Host visit log:** The log is visible only in settings, stores rounded approximate duration, and is deleted with the account.
- **Optional visit notifications off by default:** The plan rejects push and default notifications; the only notification considered is explicit account-level email opt-in, absent from onboarding.
- **Naturalist screen-reader narration:** Narration uses structured scene facts and naturalist grammar so screen-reader users perceive the aviary without raw mood labels or trait numbers.
- **Runtime call captions:** Captions are generated from the exact expanded call event so captions cannot disagree with audio.
- **Complete keyboard interaction:** Keyboard reachability is part of the accessibility release gate and covers bird navigation, listen-in, offers, settle, notebook, invitations, export, and deletion.
- **WCAG AA text contrast:** Contrast is required for top-bar labels, captions, narration, settings, and system/error copy across scene states.
- **Designed reduced-motion rendering:** Reduced motion keeps character through still-pose cross-fades and color/audio instead of replacing the scene with a static illustration.
- **Launch/runtime performance budgets:** Budgets keep the first bird, frame rate, memory, snapshot size, and tick latency from undermining the continuing-place experience.
- **Privacy-preserving operational observability:** Observability exists for operations, but cannot be joined to birds, accounts, invitations, interaction streams, or per-user history.

### Proposed system shape

- **Small TypeScript monorepo with separately deployable packages:** The plan says this avoids "premature service boundaries" while separating API request latency from worker load.
- **Preact plus TypeScript for controls and account surfaces:** NOT RECOVERABLE FROM PLAN
- **Custom Canvas 2D scene renderer:** The plan wants the aviary rendered as a scene while keeping actionable UI and semantic equivalents in DOM layers.
- **DOM-based semantic controls, focus proxy elements, captions, and live narration:** These let Canvas visuals share a presentation snapshot with accessible interaction and narration.
- **WebAudio for calls:** Audio is procedural and synthesized in the browser, with no recorded call assets or fallback loops.
- **Route-level code splitting:** Account, notebook, accessibility, and invitation code stay out of the first-aviary chunk so the first bird can paint quickly.
- **Edge web layer:** It serves shell, assets, compatibility checks, and bootstrap snapshot so the first frame can start from minimal current scene state.
- **Authenticated bootstrap snapshot with no private HTML in shared cache:** This preserves privacy while improving first-frame latency.
- **API service:** It authenticates actors, validates events and idempotency, and exposes no direct simulation mutation endpoint.
- **Simulation workers:** Workers claim due aviaries, fold events transactionally, advance state, and publish immutable render snapshots, keeping simulation out of clients.
- **PostgreSQL canonical store:** The plan chooses row transactions and constraints rather than distributed client reconciliation.
- **Scheduler/queue:** It enqueues only aviary UUID and due logical time; at-least-once delivery is safe because ticks are idempotent by boundary and watermark.
- **Private snapshot cache:** It improves snapshot latency but is encrypted, revisioned, disposable, and never authoritative.
- **Transactional outbox:** It carries cache invalidations and authorized email jobs without moving personality values or raw interaction payloads into general infrastructure.
- **Email provider:** It sends auth, invite, export, verification, and opted-in visit emails while provider metadata avoids plaintext email in logs.
- **Operational telemetry pipeline:** It accepts aggregate counters, histograms, and errors with bounded dimensions and has no route to the simulation database.
- **Canonical-state and render boundary:** The database stores semantic state; snapshots include only presentation data needed to reproduce the current experience.
- **Mood-derived presentation tags instead of raw values:** This prevents numeric mood or personality leakage in normal product surfaces.
- **Client interpolation and scheduled-call synthesis:** The client can reproduce timed transitions and calls but cannot choose mood, perch, weather, call frequency, offer outcomes, or personality deltas.
- **Deterministic client-local ornaments:** Leaves, feathers, and parallax stay ornamental because they never feed back into state.

### Data model and retention

- **UUIDv7/UUID identity and UTC timestamps:** Internal identity remains stable across devices, deployments, and migrations.
- **Account IANA time zone:** The plan seeds it from the browser, shows it in settings, and requires confirmation before changes so devices do not silently fight.
- **Visitor rendering in host time zone:** This keeps visitors aligned with the canonical aviary day rather than their own device day.
- **Encrypted email plus blind index:** Equality lookup is possible while the blind index remains auth-service-only and cannot become an identifier, log field, partition key, or telemetry dimension.
- **`visit_notifications_enabled` default false:** This supports the off-by-default notification posture.
- **Device session hashed bearer-token identifier:** It supports revocable sessions without storing raw bearer tokens.
- **No IP history in device sessions:** If IP is needed for abuse controls, it stays briefly in the auth gateway with no account join key.
- **Magic links and email-change tokens:** Hashed random tokens, short expiry, and single conditional consume prevent concurrent clicks from issuing two sessions.
- **Aviary revision, tick, weather, day, and seed fields:** These are the canonical coordinates for server simulation and snapshot publication.
- **No user-facing visit count, streak, score, or engagement total on aviaries:** The data model enforces the anti-gamification surface.
- **Stable bird ID, species version, adoption order, and voice/behavior seed:** These preserve recognizable identity through renames, migrations, and species revisions.
- **Five normalized traits in typed bounded columns:** Database bounds avoid an unvalidated JSON blob and keep trait values valid.
- **Per-trait low-pass accumulator and calibration version:** Engine migrations are explicit and replay-free.
- **No foreign key from analytics or email systems to birds:** This blocks easy joining of bird state into infrastructure outside the simulation store.
- **Interaction events with idempotency key, sequence, receipt time, and consumed tick:** These support ordered, retry-safe event folding.
- **Short raw event retention inside the simulation store:** Raw events are compacted after consumption, expire after a short window, and are not exported to a warehouse.
- **Presence sessions:** Accepted intervals are clamped, merged across devices, and folded into an accumulator; raw pings expire with events.
- **Offer commands and cooldowns:** A unique per-bird cooldown prevents two devices from accepting offers in the same short window, and rejected cooldown attempts create no drift.
- **Simulation ticks:** Unique `(aviary_id, logical_tick_boundary)` rows make retries idempotent and contain no prose rationale.
- **Notebook entries:** Immutable naturalist prose is retained until deletion; internal structured source contains no personality number and allows future wording migrations without rewriting history.
- **Invitations:** Encrypted invitee email, hashed token, expiry, revocation, and generation keep visits scoped without profiles, friend edges, or permanent public URLs.
- **Visitor sessions and visit logs:** Visitor sessions have no mutation permissions, and visit logs are host-only settings data deleted with the account.
- **Account export job:** It reads the canonical store, encrypts the object, emails a short-lived signed URL, and avoids analytics and ordinary CDN logs.
- **Deletion and recovery:** Deletion immediately revokes access, then a 30-day hard-delete job removes canonical rows, cache entries, queued emails, exports, and account-keyed operational records.

### API contracts

- **`/v1` JSON endpoints:** Versioned contracts keep API behavior explicit for auth, snapshots, commands/events, notebook, account operations, and invitations.
- **Secure same-site owner cookies and CSRF protection:** These protect state-changing owner sessions.
- **Opaque bearer cookies for invitation sessions:** Visit access stays scoped to the invitation session.
- **Idempotency keys on mutations:** Retries can return a canonical revision or command ID without double-applying state.
- **Matter-of-fact RFC 9457-style problem responses:** Error surfaces use direct system language rather than themed prose.
- **Magic-link request endpoint:** It always returns the same neutral response and rate-limits by short-lived keyed email digest and network bucket to avoid disclosing account state.
- **Magic-link consume endpoint:** Atomic token consumption establishes one device session.
- **Account read/update endpoints:** They return settings and revocable devices, never personality or interaction history; updates use optimistic versioning.
- **Email-change endpoints:** The old address remains until verification commits.
- **Session revocation endpoint:** It gives the owner direct control over device sessions.
- **Export endpoint:** It queues a job and sends delivery only to the verified email.
- **Deletion/recovery endpoints:** They implement the recoverable deletion flow.
- **Aviary bootstrap endpoint:** It returns the smallest authenticated snapshot and asset manifest, supports ETag, and bypasses shared caches.
- **Aviary snapshot endpoint:** It returns `304`, full small snapshots, or bounded deltas with authoritative server time so clients do not simulate missing semantic state.
- **Aviary session endpoint:** It opens or resumes rendering, provides return/absence context, and can attach exactly one primary greeting; visitors cannot call it.
- **Events batch endpoint:** It accepts sequenced presence/listen-in/settle facts, returns accepted sequence and clamped intervals, and never returns drift deltas.
- **Offers endpoint:** It asks the server to evaluate gifts and returns reaction cue or quiet cooldown/unavailable result without trait explanation.
- **Settle endpoint:** It ends only this device's presence window; settling is per-session presentation plus small mood input so one device cannot force another active device into an overlay.
- **Bird name patch endpoint:** It permits name changes only, with length, Unicode, and product-safe rendering validation.
- **Notebook endpoint:** It returns immutable entries in reverse chronological pages.
- **Adoption accept endpoint:** It accepts only server-created age-gated offers and cannot exceed rollout or seven-bird caps.
- **No generic PATCH for birds, aviaries, moods, perches, or vectors:** Generated clients should make forbidden state writes impossible by construction.
- **Invitation create/list/delete endpoints:** Invites are one-use, 30-day, non-public, revocable, and listed without unread counts.
- **Visit consume/snapshot/end endpoints:** Visitor sessions are scoped, rechecked on every pull, return scene/audio only, and exclude owner controls, notebook, presence, offers, listen-in, settle, and mutations.
- **Same "visit no longer available" response:** Expired, used-without-session, and revoked links return the same direct response so token state is not disclosed.

### Simulation engine

- **Once-per-minute logical UTC tick transaction:** Ticks advance due aviaries under row lock and one canonical transaction so personality and mood are written once.
- **Missed-boundary catch-up:** Long outages integrate slow accumulators over elapsed time rather than replaying millions of render moments.
- **Server-receipt event folding:** Events are read in validated order with idempotency and sequence rules so clients do not control clocks.
- **Transactional render revision and outbox publication:** A retry sees the unique tick and watermark and cannot apply drift twice.
- **Pure, versioned engine function:** Database and queue adapters wrap it, enabling weeks-long accelerated tests without clocks, browsers, or services.
- **Presence state machine:** A qualifying interval starts only while visible, focused, and recently active, preserving presence as a trusted input.
- **Three-minute activity window and 30-second heartbeat starting point:** NOT RECOVERABLE FROM PLAN
- **Presence calibration through synthetic/user-research sessions:** The plan avoids production per-account analysis for tuning.
- **Heartbeat interval reporting:** The client reports locally observed qualifying intervals, not unbounded totals.
- **Server presence rejection, clamping, union, closure, and timeout:** Background tabs, future times, long gaps, device overlap, missing closes, revoked sessions, and auth expiry cannot create continuing or double presence.
- **No pointer/keyboard content capture:** Only a recent-activity timestamp is used, protecting input privacy.
- **Drift function:** Non-negative deltas, saturating response, and `(1-trait)` make growth slow, bounded, and impossible to reverse through absence.
- **Presence, listen-in, offers, and settle inputs:** Presence dominates expressive dimensions, listen-in targets warmth/vocal dimensions, offers add small curiosity/boldness, and settle affects current mood/presence only.
- **Drift calibration cohorts:** No-visit, quiet, long, short, multi-device, offer, and listen-in cohorts enforce no absence change, seven-day imperceptibility, three-week coherence, anti-clicking, and multi-year bounds.
- **Calibration version on birds:** Engine upgrades operate forward on current values and never rebuild vectors from event history.
- **Mood transition matrix with dwell/hysteresis:** Mood changes respond to time, weather, interactions, birds, and personality without flicker.
- **Daily baseline around local dawn:** Mood evolves from persisted prior mood instead of resetting on tab open.
- **Accumulated personality separate from recency/mood:** A bird can greet less after absence while boldness, warmth, color, and learned traits remain intact.
- **Server perch selection:** Personality and mood influence zones, while occupancy and spacing keep up to seven birds legible and non-colliding.
- **Bird-to-bird wary cues and chorus plans:** Neighbor effects can shape mood probabilities and compatible call windows without client invention.
- **Local day from IANA time zone:** DST and local phase are canonical, continuous in lighting, and gradual in mood.
- **Weather from per-aviary seeded calendar:** Rain and soft wind are canonical, rare, non-destructive, free of thunder/snow, and never a response to user behavior.
- **Return greeting:** Absence comes from the last accepted owner-presence endpoint, not last HTTP request; one primary bird and optional staggered response start from plausible mid-action context.
- **No visitor greetings and no textual welcome:** Greetings are owner-only and never paired with text UI.
- **Notebook observation candidates:** Entries arise only from unusual greeter order, quiet/preening, weather-linked behavior, first perch use, chorus, or visible slow change.
- **Notebook sparsity gate:** Time since last entry, novelty category, and recent wording keep the notebook sparse, roughly every few days for regular visits.
- **Notebook deterministic prose templates:** Reviewed grammar avoids third-party private-state calls and forbids visits, streaks, user behavior, numeric traits, generic achievements, and system timestamps.
- **Persisted notebook prose:** Old entries never change with a template deployment.

### Client rendering and interaction pipeline

- **Server-rendered quiet field:** The first frame shows place immediately with no spinner.
- **Minimal bootstrap snapshot and tiny initial atlas/grammar manifest:** The first bird can paint before nonessential code and assets.
- **Synchronous first bird in current phase:** Server action times make the bird mid-action, supporting the continuing-place intent.
- **Lazy account, notebook, invite, and advanced settings chunks:** Noncritical panels do not block first aviary paint.
- **Slow snapshot fallback quiet field:** On a cold or slow dynamic snapshot, the scene uses faint ambient color motion and never a progress spinner or fake bird.
- **First-bird performance mark:** Instrumentation measures the actual painted bird, not the quiet field.
- **One Canvas 2D scene with DOM controls:** The visual scene stays separate from actionable and semantic layers.
- **Server-driven pose/state graphs:** The client blends poses but does not invent semantic behaviors.
- **Responsive perch zones:** Narrow phones compress gaps and scale birds inside tested bounds without cropping, pan, zoom, or scroll.
- **Rendering performance techniques and hidden pause:** `requestAnimationFrame`, pooling, caching, and hidden-state pause support frame and memory budgets.
- **Reduced-motion ornament removal:** Leaf drift is removed entirely in reduced-motion mode.
- **Canonical light palette and settle overlay:** Color/day/settle visuals come from canonical day phase and local settle state.
- **Sparse top bar fade:** Controls fade nearly transparent after inactivity but restore on keyboard activity/focus so access is not pointer-only.
- **Two-revision snapshot interpolation:** Server time offset, bounded plans, and full snapshot recovery keep local interpolation aligned with authority.
- **Cross-fade/reconcile recovery:** Revision gaps, long frames, resume, visibility return, and cache mismatch recover without resetting the whole scene.
- **Temporary network-unavailable behavior:** The client finishes scheduled micro-actions, avoids alarm toasts, does not simulate semantic state, and holds plausible idle pose if stale.
- **Sustained/auth failure surface:** The product leaves the naturalist register and gives a concise system action.
- **Listen-in interaction:** Click/tap or Enter toggles bird focus, transfers focus between birds, and sends start/end facts while local audio mix ramps immediately.
- **Offer interaction:** The user picks one of three gifts, but only the authoritative receiving-bird reaction is animated; cooldown copy avoids countdown meters.
- **Settle interaction:** Settle ramps scene and calls down, ends local presence, and can be reversed for five seconds by re-engagement.
- **Notebook UI:** It is lazy-loaded, read-only, virtualized, cursor-paginated, and never exposes event-log rows or edit/delete affordances.
- **Adoption UI:** Age policy creates the offer independent of attention; copy preserves the "bird that arrived" framing and avoids catalogs, rarity, counters, or reward language.

### Procedural audio

- **Reviewed motif library per species:** Pitch contours, envelopes, gaps, timbre/noise, and legal combination rules make calls varied but controlled.
- **Immutable per-bird voice seed:** Register, timbre, and microtiming stay stable within species-safe bounds so birds remain recognizable.
- **Mood and call-plan parameters:** Tempo, intensity, spacing, and response likelihood can change without erasing identity.
- **Scheduled call snapshot fields:** Bird ID, version, seed, start time, motif intent, and bounded parameters make client expansion deterministic.
- **Oscillator/noise/envelope synthesis and pooled buffers:** The client generates calls without downloading or caching recorded calls.
- **Look-ahead scheduling:** Calls remain stable across render-frame variance.
- **Voice, compressor, reverb, and gain limits:** Up to seven-bird chorus should avoid clipping and blur.
- **Canonical bird-to-bird chorus timing:** Client scheduling adds only inaudible sub-frame precision.
- **Ambient/listen-in mix:** Every bird remains audible at low level; focused birds ramp up and others ramp down rather than hard-cutting or muting.
- **Settle and reengage audio ramps:** The master/call-density ramp slows down and reverses smoothly.
- **Browser autoplay handling:** The aviary draws immediately, then audio starts on the first permitted gesture; blocked audio is explained only in accessibility/settings.
- **Audio lifecycle cleanup:** Hidden-state suspension, node/buffer reuse, capped queues, and context closure prevent leaks.
- **Structured captions:** Captions come from the exact expanded call event, so audio and captions cannot disagree.
- **Captioned-silence fallback:** If WebAudio fails, the session uses graceful silence with captions by default, not recorded fallback.
- **Blind listening recognizability tests:** Bird counts above two wait for recognizability and uncanniness thresholds.

### Accessibility implementation

- **DOM scene model parallel to Canvas:** The aviary region and bird focus targets let screen-reader and keyboard users perceive and navigate the same presentation snapshot.
- **No routine ARIA mood labels or trait numbers:** The semantic layer keeps personality and mood values hidden like the visual product.
- **Naturalist narration grammar:** Structured scene facts produce reviewed prose observations instead of robotic labels.
- **Narration queues:** Idle observations, greetings, offer reactions, and settle observations coalesce in a polite live region to preserve calm cadence.
- **Pause/repeat controls:** Accessibility settings let users control narration without changing the underlying aviary.
- **Assistive-technology validation:** Acceptance is "comprehension and calm queue behavior," not merely labels.
- **Keyboard and focus model:** Top-bar controls, birds, dialogs, arrows, Enter, and Escape provide full non-pointer operation.
- **Visible focus rings across palettes:** Focus must remain AA-compatible in morning, midday, evening, night, rain, and settle states.
- **No hover-only behavior:** Touch targets and pointer alternatives meet platform sizing guidance.
- **Reduced motion as OR of preference and account setting:** The local preview and account setting let users verify the mode.
- **Curated still poses and cross-fades:** Reduced motion removes flights/parallax/leaf drift but keeps color/day/settle change and an alive scene.
- **Visual access testing:** Zoom, 200% text, narrow viewports, forced colors, and color-vision simulations keep state from depending only on plumage color.
- **Accessibility release gate:** Narration cadence, reduced-motion completeness, captions, keyboard reachability, and contrast block release if they regress.

### Sync, consistency, and failure handling

- **Aviary tick serialization:** Row locks and unique logical boundaries make personality/mood a single-writer path.
- **Command/event deduplication:** Actor scope plus idempotency key avoids repeated effects under retry.
- **Server receipt order with client sequence only for gaps/duplicates:** Client clocks are not trusted.
- **Rename/settings optimistic conflict response:** The user sees current editable values, never a choice between personality states.
- **Post-commit snapshot publication:** Cache reads are revisioned; stale cache entries are replaced or bypassed and never written back.
- **Different interpolation frames, same semantic revision:** Phone and laptop can animate differently but converge on canonical state.
- **Session presentation versus canonical state:** Settle/listen-in are session presentation states; offers, mood, birds, notebook, weather, and drift are canonical.
- **Worker outage catch-up:** Due ticks queue, catch up by elapsed time and input watermark, and clients do bounded idle rendering without simulating.
- **Direct failure surfaces:** Expired auth, failed links, revoked visits, unsupported browsers, and snapshot errors leave naturalist language and provide concise actions.
- **Concurrency tests:** Multiple devices, reordered events, simultaneous offers/renames, mid-pull revocation, tick retry, DST, and suspension assert no lost/double drift and no client-authored canonical fields.

### Performance budgets and observability

- **Initial JavaScript budget:** The 2 MB hard cap and tighter aviary target keep asset growth visible.
- **First bird under 500 ms on shaped 4G and target hardware:** The gate measures the actual user condition, not a fast developer machine.
- **Idle 60 fps trace:** Day lighting, calls, and interactions must remain smooth for continuous ordinary use.
- **Memory retained-heap budget:** Audio nodes, animation objects, listeners, snapshots, notebook rows, workers, and contexts are inspected to prevent leaks.
- **Snapshot payload budget:** Two- and seven-bird budgets stop schema growth and keep bootstrap independent from notebook, visits, settings panels, and noncritical assets.
- **Tick latency alarm:** p99, queue-age, and retry/dead-letter alerts protect server simulation freshness.
- **Permitted production telemetry:** Requests, route templates, latency, queue/tick/cache/email/export/deletion health, browser major version, coarse device class, first-bird timing, frame-time, long-task, memory-warning, audio error, caption fallback, and nonlinkable session-duration histograms are allowed because they are operational and aggregate.
- **Forbidden telemetry:** Account, aviary, bird, invitation, token, email/blind-index, IP, names, species, mood, personality, notebook, presence, listen-in, offer, settle, visitor recipient, and per-user history cannot appear as dimensions or payloads.
- **Telemetry enforcement:** Typed wrappers, CI scans, payload tests, separate credentials, short logs, and privacy review keep operational telemetry from becoming engagement analytics.
- **Synthetic aviaries and explicit research studies:** Bird behavior tuning must not rely on silent production aggregation.

### Verification strategy

- **Engine property tests:** They directly prove drift monotonicity, bounds, zero no-input drift, idempotent retry, deterministic replay, presence union, visitor isolation, and seven-bird cap.
- **Accelerated simulations:** Minutes-to-years runs cover local time, DST, absence, interaction, weather, multi-device, outages, and calibration versions without exposing those instruments in product code.
- **Migration tests:** Stable IDs, vector values, and notebook prose survive versioned fixture migration; vector loss is severity one.
- **Contract/security tests:** Owner/visitor privilege, snapshot denylist, token lifecycle, session revoke, CSRF, authorization, export isolation, and deletion completeness guard privacy and authority boundaries.
- **Golden visual sequences:** Ordinary and reduced-motion modes are checked across day phases, weather, 2-7 birds, viewports, settle/undo, returns, and slow bootstrap.
- **Audio deterministic tests:** Grammar/caption agreement, cleanup, mix ramps, clipping, chorus timing, unavailable WebAudio, and stable voice keep audio procedural and recognizable.
- **Editorial tests:** Forbidden announcement/gamification phrases, notebook/narration/caption templates, specificity, and repetition are reviewed.
- **End-to-end multi-browser flows:** Auth, adoption, two-device convergence, presence, offers/cooldowns, notebook sparsity, invites, export, and deletion are tested together.
- **Accessibility and performance gates:** They run on every release candidate rather than after launch.
- **Acceptance scenarios:** Product, design, accessibility, privacy, and engineering owners must see the continuing-place, honest-presence, accessibility, reduced-motion, audio, visitor, performance, telemetry, and deletion claims demonstrated before public v1.

### Work breakdown, rollout, and open decisions

- **Milestone 0 - contracts, prototypes, and calibration harness:** The plan resolves invariants, schemas, time zone, retention, telemetry, engine interfaces, first-bird, procedural audio/captions, reduced motion, narration, hardware, and research before schema/API freeze.
- **Milestone 1 - identity and canonical aviary:** Stable accounts, encrypted email lookup, sessions, time zone, schema, two-bird adoption/naming, ticks, snapshots, and cache publication must work across devices, worker restarts, and duplicate deliveries.
- **Milestone 2 - behavioral core and ordinary scene:** Presence, drift, mood, perches, day/weather, greetings, bird effects, calls, Canvas, responsive layout, listen-in, offers, settle, undo, and recovery must pass two-bird calibration and performance gates.
- **Milestone 3 - notebook, accessibility, and account lifecycle:** Sparse prose, read-only notebook, semantic scene, narration, keyboard, captions, fallback, reduced motion, contrast/focus, email change, export, deletion, and unsupported-browser path must reach ordinary/access parity.
- **Milestone 4 - quiet visits and scale readiness:** Invite tokens, scoped visitor snapshots, session/logs, revocation/expiration, optional off-by-default notifications, no visitor simulation inputs, load tests, threat model, and telemetry audit must prove read-only access.
- **Milestone 5 - release hardening:** Soak, outage, migration/restore, DST, seven-bird synthetic, browser, editorial, security, accessibility, performance, and privacy tests precede version freeze and runbooks.
- **Team dogfood:** Synthetic and staff accounts with fixed two-bird cap validate continuity, privacy boundary, daily clock, call repetition, and recovery while keeping staff data under production privacy rules.
- **Invited accessibility/performance preview:** Two birds, accessible modes, and account lifecycle are tested with visits disabled and direct consented feedback instead of per-bird behavior analytics.
- **Limited beta:** Visits turn on for a small random cohort while engine/calibration is pinned and SLOs, support, audio fallback, accessibility, and deletion correctness are watched.
- **General v1:** All new accounts start with two birds, and caps rise only after recognizability, layout, memory, snapshot, and accessibility gates pass.
- **Age-gated growth:** Initial eligibility is around aviary age, not behavior; adoption offers are quiet, rollout-capped, and "waiting accounts lose nothing."
- **Server kill switches:** New offer creation, visit creation, notification email, calibration version, and max bird count can be stopped while preserving existing birds and vectors.
- **Independent client/render/audio rollback:** Assets can roll back separately from canonical data migrations so incidents are not solved by regenerating birds.
- **Day-one dashboards:** They cover first-bird latency, lab frame/memory, snapshot/API/tick health, queue, cache, auth/email, audio errors, visit auth failures, and erasure/export jobs, but deliberately avoid named or pseudonymous behavior.
- **Principal risk mitigations:** The risk table ties each failure signal back to versioning, property tests, clamping, allowlists, manual sign-off, or privacy separation.
- **Open implementation decisions:** Presence window, mood constants, trait ranges, species/motifs, keepalive, palette/focus tokens, notification wording, and age-gate dates must be resolved by prototype or explicit decision; none permit new features or relaxed invariants.
