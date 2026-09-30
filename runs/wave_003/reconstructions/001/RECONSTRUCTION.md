## System-level intent

- Private, single-user calm rather than social product machinery. This shows up in Product boundaries as “private, single-user aviary per account” and the exclusions of “shared aviaries,” “public discovery,” “rankings,” “push notifications,” “scores,” “progression counters,” “streaks,” and “rewards for attention.” It returns in Budgets/observability as “Do not instrument retention, streaks, offer popularity, per-bird growth, visit rankings or production average drift.”

- Preserve a hidden affective invariant: birds may be expressive, but the user never sees the machinery. Product boundaries says to “Preserve the stricter affective invariant” by omitting “numerical personality vectors and hidden derived trait coefficients.” Architecture repeats “No raw vectors, trait labels or one-to-one numerical trait aliases,” and verification forbids “traits and stat language.”

- Canonical simulation belongs on the server; clients render authorized projections. Architecture says clients receive “only an observable projection” and “cannot choose canonical mood, consume events, advance drift, or simulate offline evolution.” Simulation says “No lazy client-side catch-up on opening a tab,” while rollout says rollback must “never restore older vector snapshots over newer drift.”

- Identity and continuity are durable. The plan emphasizes “persisted bird identities,” “immutable UUID,” “stable call signature,” “restore preserves identities/vectors,” and “restoration never regenerates birds.” Rollback and migration must preserve “bird signature.”

- Absence is non-punitive. Product boundaries excludes “hunger, death, distress from absence.” Drift says “Absence makes signal decay, not traits decline,” and “absence cannot induce distress, loss of color, mistrust, hunger or punishment.” Deletion recovery resumes without “inventing absence presence.”

- Attention must be honest but not invasive. Presence requires visibility, focus, and recent pointer/key activity; the plan says not to substitute “scroll, timer callbacks, audio playback or a network heartbeat.” Multi-device accounting says two screens “cannot double drift,” while also retaining “no pointer positions or key contents.”

- Product voice is naturalist and quiet. Product boundaries specifies “lowercase, present-tense, bird-specific naturalist observation” for product prose and “matter-of-fact language” for identity, errors, account, sync and accessibility. It also says to “avoid general-purpose toast infrastructure in the aviary.”

- Accessibility is part of the living product, not a late fallback. Product boundaries includes “all accessibility modes at launch.” Accessible experience says automated audits alone do not establish “an alive accessible aviary,” and rollout says “No launch waiver that postpones a mode.”

- Quiet notification restraint governs both owner and visitor surfaces. Visit notifications are limited to “a quiet textual notice inside the visit-log settings page only,” with “no badge, toast, modal, email or push.” Connection failure is a “quiet inline system connection state,” and invitations are revoked “without success toast.”

- Privacy minimization is structural, not just copy. Persistent model says email is encrypted, lookup uses a “restricted keyed blind index,” and contact storage is isolated. Observability allows only aggregate timings and anonymous histograms, with “no analytics warehouse or model-training export” access to simulation tables.

- Aliveness should feel continuous rather than canned. The plan calls for “continuous bird-to-bird behavior,” contingent response motifs, greetings that do not “rotate a small canned list,” procedural motif grammars, “no exact call repetition,” and a first frame that does not feel “started rather than ongoing.”

- Release quality is gate-based. Budgets says thresholds are “release gates, not aspirations.” Delivery milestones have explicit exits, and the launch ramp is gated on “operational health and designed-surface reviews,” not launch progress indicators or production interaction aggregation.

## Per-feature whys

### 1. Product boundaries and decisions

- Browser-only delivery: NOT RECOVERABLE FROM PLAN

- Private, single-user aviary per account: The plan keeps the aviary out of social mechanics by excluding shared aviaries, profiles, follows, feeds, public discovery and rankings, and by saying “Never compute social rankings or population interaction statistics.”

- Two system-selected starter birds: NOT RECOVERABLE FROM PLAN

- Coherent six-species pool: The plan ties this to distinguishability and recognizability through “six motif grammars,” “six-species distinguishability,” and the milestone gate that six species remain recognizable when the count grows.

- Hard seven-bird ceiling: The plan gives legibility, recognizability, performance, and cap enforcement as the reasons: “seven birds remain legible,” “recognizability remains satisfactory at seven and budgets hold,” and “Cap enforcement is transactional.”

- Naming and renaming: Names are allowed as editable display text while identity stays stable; renaming “never changes identity or traits,” and names are “text-escaped everywhere, including narration and captions.”

- Continuous bird-to-bird behavior: The rationale is aliveness through system-level reactions: a “contingent response motif” lets birds “react as a system,” and “Chorus emerges from overlapping calls.”

- Owner return-greetings: The plan wants returns to feel noticed quickly and continuously, with “actual bird notice start after navigation/return,” “continuous variations,” and no “small canned list.”

- Listen-in: Listen-in is the explicit attention signal and audio focus mechanism. It counts “qualifying attention duration, not button presses,” and sound preference remains “an accessibility choice.”

- Three offers: NOT RECOVERABLE FROM PLAN

- Offer interactions: Offers give subtle interaction without becoming feeding, inventory, or reward loops; the plan specifies “No offer inventory, feeding schedule or penalty for rejection,” caps credit so “repeated clicks cannot dominate,” and uses cooldowns across devices.

- Settle: Settle is for local quieting without social or trait pressure: it is “session-local lighting and mixing,” “has zero trait credit,” and “one device cannot force another device into a session-ending UI.”

- Sparse read-only notebook: The notebook is sparse to avoid a feed or measurement surface. The plan says to ensure active users “do not receive a feed,” never record “visit streaks, trait numbers or ‘session started,’” and keep entries read-only.

- Email magic links: The plan grounds them in security and account access: generic accepted responses “avoid enumeration,” redemption is “single-use” with a “15-minute transaction,” and replay returns an actionable system error.

- Device sessions: Device sessions support revocation and stateless secure cookies; the model says cookies “never contain state,” and API/lifecycle sections require revoked credentials to be rejected on the next request.

- Multi-device canonical state: The reason is one canonical aviary and no double credit. Milestone 2 exits when “two devices share one canonical state,” and presence is merged so “two screens cannot double drift.”

- Account export: Export exists for owner account lifecycle while preserving hidden-trait boundaries: it includes names, identities, species, moods, qualitative behavior, notebook and settings, but “no numeric vectors.”

- Account deletion: Deletion protects privacy and exact lifecycle semantics: soft deletion disables access, ticks, visitors and links; hard deletion cascades private data, removes keys/files, and uses tombstones so erased data “cannot reappear.”

- Individually issued read-only visit invitations: Invitations let a specific recipient view a bounded projection without public identity or influence. The plan says links grant “one bounded read-only session,” visitors cannot issue interaction events, and the host log keeps transparency.

- Accessibility modes at launch: The plan treats accessibility as part of v1 completeness: “all accessibility modes at launch,” “This milestone includes accessibility rather than scheduling it later,” and “No launch waiver that postpones a mode.”

- Export hidden-traits exclusion: The why is the “stricter affective invariant”; exports must not show numerical vectors or disguised user-visible numbers even though internal backups preserve exact vectors.

- Quiet visit-log notice toggle: The why is compatibility with headline principles: it is “the narrowest useful interpretation” of visit notifications, off by default and limited to a quiet settings-page notice.

- One account IANA timezone: The reason is shared canonical experience across devices and visitors; a traveling device should not “change everyone else's aviary.”

- Four top-bar primary affordances with settle in the offer/action popover: The plan keeps reachability while avoiding another persistent icon; this “satisfies top-bar reachability without adding another persistent icon.”

- Age-based adoption pacing: Adoption depends on elapsed aviary age to avoid urgency and monetized pressure: “no urgency, countdown, paid acceleration or missed opportunity,” and availability is “independent of visits.”

- Product prose and copy catalog: The rationale is consistent affective tone and error clarity: aviary prose is “bird-specific naturalist observation,” while system surfaces are “matter-of-fact,” with reviewed forbidden language.

### 2. Architecture and ownership

- TypeScript web client: NOT RECOVERABLE FROM PLAN

- TypeScript HTTP application: NOT RECOVERABLE FROM PLAN

- PostgreSQL canonical state and private simulation data: The rationale is canonical persistence for identities, vectors, moods, cursors and revisions that survive crashes, restores and multi-device access.

- Independently deployed simulation workers: Workers own simulation updates so ticks can run with no clients present, and only workers have permission to update personality columns.

- Immutable CDN assets and non-shared personalized HTML/snapshots: The plan uses global caching for public assets while saying to “never shared-cache personalized HTML or snapshots.”

- Authenticated HTML/bootstrap endpoint with inline authorized snapshot: This supports first-frame performance and continuity by putting a “small authorized snapshot into HTML” and avoiding serial fetch gates.

- Isolated email outbox worker: The why is containment: email delivery is isolated for authentication, invitations, verification and requested export, and mail payloads avoid bird histories or states.

- Modular backend instead of microservices: The plan says a “modular backend is sufficient for v1” and explicitly avoids separate microservices for birds, notebook and weather.

- Database grant boundary for personality columns: This enforces the hidden-vector and continuity boundary: “Event intake cannot update them,” and deployment/database grants enforce it.

- Observable projection: Clients receive silhouettes, palette tokens, poses, calls and timing because observable animation/audio may reflect personality but must be “bounded presentation instructions rather than a trait model to optimize.”

- Split behavioral and presentation clocks: The server chooses canonical moods/plans while the client interpolates and synthesizes calls, allowing smooth presentation without client-side canonical simulation.

- Server-issued presentation acknowledgment: The why is immediate feedback that does not contradict the later canonical tick; the next tick commits consequences using the same seed and decision.

### 3. Persistent model and retention

- Random UUID internal keys and no email identifiers: The rationale is privacy and authorization hygiene: email is “never a partition key or identifier,” and UUID uniqueness is “not authorization.”

- Account record with exactly one aviary: It carries the single-user product boundary and stores encrypted verified/pending email, timezone, settings and deletion state.

- Magic-link challenge storage: Hashed tokens, expiry and atomic one-use consumption support the security model for magic-link authentication.

- Bird record with immutable ID, persisted traits, accumulators and stable call signature: The reason is durable identity and drift continuity; vectors are persisted, identities are not regenerated, and call signature survives years of drift.

- Interaction event sequence and uniqueness: Monotonic aviary sequence, event UUID and session sequence support deterministic order, retries and duplicate detection.

- Presence intervals for owner attention only: The model records bounded accepted intervals and listen-in target “for owner attention, never visitors.”

- Notebook entry immutability and cursor pagination: The plan keeps notebook observations private, chronological, sparse and read-only while supporting long-term browsing.

- Invitation, visitor contact and visit log records: These support explicit recipient authorization, encrypted contact isolation, and host transparency without creating public identity.

- Recent event retention target of 30 days after consumption: Events are kept only for “consumption, bounded recovery and cooldown/presence validation,” while unconsumed events are not removed.

- Persistent vectors, accumulators and compact summaries: The plan keeps canonical state while saying “vectors are never rebuilt from events,” preserving exact identity and drift.

- Indefinite notebook retention until deletion: The notebook remains readable as part of the account’s private aviary history, ending at account deletion.

- Visit records retained until deletion: The reason is “host transparency,” with clear settings disclosure.

- Export object lifetime of 24 hours and token/contact cleanup: The plan limits private download exposure and requires explicit expiry cleanup.

### 4. API contracts

- Route-level TLS, resource authorization, schema validation, CSRF, request IDs and no sensitive caching: The rationale is secure account and visitor boundaries; the plan explicitly says UUID uniqueness is not authorization.

- Magic-link request endpoint: Generic accepted responses avoid enumeration, while per-email and IP limits reduce abuse.

- Magic-link redeem endpoint: Atomic single-use redemption creates the session, redirects to a clean URL, and makes replay an actionable system error.

- Owner snapshot endpoint: It returns projection, time, revision, plans and capabilities so the client can render authorized canonical state and align clocks.

- Batched events endpoint: Stable IDs, session sequence and acknowledgments allow retries, duplicate detection and immediate presentation without client trait mutation.

- Event status endpoint: The plan says it is “useful after an ambiguous network timeout.”

- Notebook endpoint: It is owner-only with immutable observations and “no stats or editable routes,” preserving the read-only, non-dashboard intent.

- Rename endpoint: If-Match protects against conflicting rename while “never” changing identity or traits.

- Adoption endpoint: It returns offers only when “age eligible” and with “no scores or progression counts,” keeping adoption age-based rather than gamified.

- Adoption acceptance endpoint: Idempotent signed-offer acceptance and transactional cap check preserve the seven-bird ceiling and stable new identity.

- Sessions list/revoke endpoint: The next request rejects revoked credentials, which supports device control and account security.

- Account settings endpoint: Timezone, accessibility/audio/caption settings and optional visit-log notice are explicit settings with optimistic metadata revision.

- Email change verification: The current address remains valid until the new address is verified and switched atomically.

- Account export endpoint: It produces a consistent snapshot as a bounded async job and emails a private link to the current verified address.

- Account delete/recover endpoints: Soft deletion starts a 30-day window and blocks normal simulation and visits while preserving recovery.

- Invitation creation/list/delete endpoints: They require explicit host-provided email, one-use tokens, 30-day unused expiry, owner list plus visit log, and revocation without success toast.

- Visitor redeem endpoint: Atomic single-use token exchange gives a scoped read-only cookie and redirects to a clean URL.

- Visitor snapshot endpoint: Visitors receive the same authorized scene projection but no notebook/account/private metadata or greeting, with revocation rechecked every pull.

- Export-token download endpoint: The link is single-purpose, expiring, authenticated, not shared-cached, and excludes numeric vectors.

- Event type restrictions: Rejecting unknown types, URLs/audio, trait mutations, client mood proposals and visitor credentials keeps event intake from becoming a simulation or security bypass.

- Atomic new-account onboarding: The first aviary and two birds are created atomically, with two different starter species and no catalog UI.

- Name escaping: Names are escaped everywhere, including narration and captions, to preserve plain text safety across surfaces.

### 5. Simulation, drift and continuity

- Scheduled atomic ticks: Tick transactions lock aviary and birds, read ordered events, apply deltas, update plans and commit cursor atomically so duplicate delivery or crash “cannot apply drift twice.”

- Event sequence allocation under aviary lock: The reason is no invisible sequence holes, no cursor advancement past deferred events, and deterministic server sequence order.

- Client-independent ticks and server catch-up: Ticks run with no clients present, mood/daylight/weather advance regardless of owner activity, and outages catch up on the server while preserving ordered interactions and day boundaries.

- Honest presence conditions: Visibility, focus and recent pointer/key input define qualifying attention; this prevents an “open tab” from becoming attention while allowing “still watching.”

- Monotonic timing and interval bounds: The plan uses monotonic clocks, freshness bounds and hidden-gap rules to avoid crediting suspension, clock changes or wall-clock gaps.

- Multi-device owner coverage union: Overlap is merged so “two screens cannot double drift,” and listen-in credit is capped to owner attention.

- Five-trait drift with low-pass accumulators: The plan wants gradual, bounded, positive development: presence remains at least 80% of typical signal weight, clicks cannot dominate, absence decays signal instead of traits, and saturation keeps identities distinct.

- Synthetic drift calibration: Fictional histories validate one-week measurable change, three-week blind perceptibility and no production population metrics.

- Reduced greeting frequency after absence: The plan frames this as “transient familiarity/expression, not personality damage,” with no days-away text or punishment.

- Five moods: Moods provide bounded affective variety through transition weights, dwell times, day phase, interaction, weather and personality, without turning wary into alarm or distress.

- Perch zones and collision-free slots: These keep seven birds legible, preserve identity through movement, and avoid unreachable or overlapping birds.

- Bird call responses and chorus: Contingent answers and staggered overlapping calls create relationship behavior without a synchronized welcome chorus or feedback lock-in.

- Seeded weather and daylight: Weather is not live location weather; persisted seeded schedules and timezone-based daylight create ambient variation with DST handling and “No dead night scene.”

- Owner-return events and greetings: The server derives absence from session intervals, chooses a primary bird, varies the greeting from an event seed, acknowledges within 1-2 seconds, and prevents duplicate or visitor-generated greetings.

- Offer recipient plans and cooldowns: Server selection by proximity, curiosity and mood plus three-minute per-bird cooldown keeps reactions coherent across devices and prevents repeated credit farming.

### 6. Synchronization and failure handling

- Owner snapshot pull cadence: Pulling every 30 seconds and on return/restoration/render gaps keeps plans fresh, discards out-of-order responses and avoids interrupting calls or motion.

- Projection horizons and ETags: Returning fresh plan timing even when revision is unchanged lets the client keep authorized motion aligned without inventing canonical state.

- Short-lived pending event queue: Stable IDs and retry backoff handle ambiguous writes, while the queue limit and no local-storage offline presence prevent days of retroactive credit.

- Disconnected rendering fallback: The client renders the last authorized ambient plan briefly, then quiet resting poses and captions, instead of extrapolating new canonical moods or calls indefinitely.

- Inline connection state: The plan chooses “quiet inline system connection state in the top bar” and “no failure toast.”

- Metadata If-Match conflict handling: Simultaneous renames return the latest name and retry path; personality conflicts cannot occur because metadata endpoints do not touch vectors.

- Settle and undo-settle: Settle affects local lighting/presence without ending other devices. Undo cancels unconsumed events or applies a compensating mood intent, never reversing trait deltas.

### 7. Scene rendering and interaction surfaces

- Small inline SVG first scene and lightweight SVG renderer: The rationale is first visible bird performance, requestAnimationFrame motion, and avoiding a large 3D engine.

- Server-seeded first pose mid-action: This prevents every animation from starting at zero and supports the risk response that the first frame should feel ongoing.

- First adoption fly-in exception: The quiet empty field and soft fly-in are reserved as “the deliberate exception” for true first adoption.

- Layered scene composition with bounded ornaments and subtle parallax: The plan wants living ambience without server leaf state, scroll/pointer spectacle, or hidden rendering work.

- Stop drawing when hidden: This supports battery/performance and pairs with restoring from a fresh snapshot rather than replaying missed motion.

- Normalized layout slots and fit-to-viewport stage: The reason is to retain all seven silhouettes across phones, 320px viewports, short landscapes and 200% text zoom without panning/zooming geography.

- Top-bar icon sizing and fade behavior: Minimum touch targets, accessible names and focus visibility keep controls usable while fade-on-stillness preserves quiet scene presentation.

- Semantic DOM bird interaction layer: Aligned semantic targets enable click/tap and keyboard listen-in, roving order, Escape exit and deterministic nearest-center selection without drag placement.

### 8. Procedural audio and captions

- Six motif grammars: Species-specific pitch, envelope, contour, gap, timbre and response rules create distinguishable birds and avoid “Audio uncanny or indistinct.”

- Stable call signature seed per bird: The seed lets motif proportions and timbre survive renaming, mood, engine changes and years of drift, preserving identity.

- Server-issued call descriptors and client grammar expansion: This keeps audio/caption output tied to authorized plans and bounded acoustic parameters.

- Captions from actual note descriptors: Captions describe scheduled calls, not hidden mood values, and work in silence mode rather than as fixed species labels.

- One AudioContext with procedural synthesis: Bounded lookahead, stale-plan cancellation, reusable buffers, voice limits and node cleanup prevent missed-call bursts and unbounded audio history.

- Chorus gain staging and spatial pan: Staggered contours, conservative gain, soft limiter, mono/phone tests and headroom keep multi-bird sound legible.

- Listen-in audio ramp: A 1.5-2 second ramp raises focused gain while ambient gains stay above zero, so focus does not erase other birds.

- Autoplay and failure handling: If WebAudio is blocked or fails, the same scene continues in “graceful silence” with captions; the plan says never claim guaranteed autoplay or use recorded fallback.

- User mute: Mute is respected across devices and “never harms drift,” preserving accessibility choice separate from attention.

### 9. Notebook and accessible experience

- Deterministic notebook generation from canonical facts: Curated phrase grammars avoid an LLM service and generic logs while keeping private observations grounded in bird names, timing and context.

- Notebook sparsity and deduplication: One ordinary observation every 2-4 days plus rare noteworthy observations prevents a feed and avoids streaks, trait numbers and session logs.

- Accessible notebook pagination and virtualization: Cursor pagination keeps the notebook indefinitely readable, while virtualization releases resources without preventing screen-reader browsing.

- Idle narration live region: Narration uses the same scene projection and naturalist grammar, emits polite evolving paragraphs, suppresses unchanged meaning and avoids assertive announcements for ordinary motion.

- Priority observations for greetings, offers and settle: These events get prompt observations coalesced into the queue without interrupting navigation or notebook reading.

- Transcript and narration pause: The visible transcript and pause control give users control over narration while system errors remain separately announced.

- Reduced motion mode: Reduced motion preserves calls, captions, moods, drift and notebook while replacing micro-motion/flights with cross-fades; the plan explicitly says not to ship a static frozen fallback.

- Call captions: Captions beside birds provide contrast-backed readable text, collision avoidance, slow fades and transcript access when seven calls would overcrowd the scene.

- Real buttons, dialogs, focus containment and keyboard routes: These make offer choices, song motifs, Escape dismissal, focus restoration and Alt+O usable without relying on pointer interaction.

- Contrast, non-color meaning and user testing: WCAG AA, visible focus outlines, VoiceOver/Safari, NVDA/Firefox, keyboard-only, touch readers and accessibility users are required because audits alone do not prove an alive accessible aviary.

### 10. Visits, account security and lifecycle

- Bounded one-use visitor links: Links are read-only, two-hour sessions, 30-day unused expiry and consumed-token dead ends so access is explicit and limited.

- Token-bearing visitor page protections: No third-party scripts, no-referrer and clean URL redirects reduce token leakage.

- Invite rate limiting and no share prompts: The plan prevents email abuse and avoids pre-populating visitor lists or onboarding share prompts.

- Visitor polling and revocation: Every pull checks invite/session/deletion status; revoked access ends on next pull, audio stops, private state is discarded and offline snapshots expire after the grace period.

- Visitor non-influence: Visitors never get greetings, attention events, presence pings, listen-in, offers, settle or notebook access, so visits cannot influence simulation.

- Visitor local accessibility controls: Audio/caption controls affect only the visitor device, while host timezone and identical projection keep the viewed aviary faithful.

- Approximate visit duration from pulls: Duration comes from successful pulls, not simulation presence, keeping logs transparent without turning visitors into presence.

- Host visit log and revocation record: The log shows recipient email, dates, duration and outstanding invitations without badges; revocation retains an ended/revoked transparency record.

- Atomic magic-link and visitor redemption with token hashes: This supports replay/expiry security and avoids storing raw tokens.

- Session, email, authorization and CSRF release gates: The plan treats these tests as release gates for account security.

- Sensitive data logging controls: Email must not leak in URLs, logs or errors; query strings and request bodies are stripped or not logged by default.

- Soft deletion: It immediately disables regular aviary access, ticks, visitors and links while permitting recovery and relevant account functions for 30 days.

- Recovery: Recovery restores exact bird state and server-time catch-up without inventing absence presence.

- Hard deletion and restore tombstones: Cascades, key erasure, private object removal and backup tombstones ensure erased data cannot reappear.

### 11. Budgets, observability and verification

- Initial JavaScript budget: The <2MB gzip ceiling and <250KB core target keep the critical aviary/bootstrap/audio path small, with noncritical panels dynamic-loaded.

- First bird paint budget: Inline critical scene/snapshot and avoiding serial gates target first actual bird paint under 500ms rather than only background paint.

- Snapshot kilobyte budget: A 5-20KB raw target for seven birds excludes hidden vectors and notebook/history to keep projection compact.

- Greeting latency budget: Measuring actual bird notice start within 1-2 seconds keeps greetings felt rather than toast-like.

- 60fps and memory budgets: Frame and plateau checks protect the calm long-running aviary experience across seven birds and repeated panels/calls.

- Tick latency budget: Queue and computation/commit latency instrumentation protect the one-minute simulation contract.

- Critical-path and TTFB budgets beyond 2MB: The plan says “The 2MB ceiling alone does not achieve 500ms,” so HTML/SVG/snapshot size and TTFB are separate gates.

- Slow-load quiet sky: This is only the exceptional slow-load field and “never a spinner,” preserving product tone during delay.

- Privacy-safe RUM and operational logs: RUM is aggregate and anonymous; logs may use account UUID only for narrowly scoped errors, with no interaction payloads or vectors.

- No engagement or production drift instrumentation: The plan forbids retention, streaks, offer popularity, per-bird growth, visit rankings and production average drift metrics.

- Verification suites: Engine, presence, transaction, audio/visual, accessibility, security/lifecycle and performance soak tests are release evidence for identity, no absence penalty, no double drift, accessibility and bounded resources.

- Browser support policy: Supporting the last two major browsers avoids legacy compatibility bundles, while WebAudio failure still gets silence/captions.

### 12. Delivery sequence and rollout

- Milestone 1 contracts and experience foundation: The why is to establish schema/projection, hidden-vector boundary, copy, contrast, prototype specificity/aliveness and privacy-safe metrics before deeper implementation.

- Milestone 2 canonical persistence and identity: The reason is to prove shared canonical state, synthetic trajectories, absence safety and restore-preserved identities/vectors.

- Milestone 3 complete two-bird experience: The plan concentrates greetings, moods, replies, audio, offers, settle, notebook and accessibility in the two-bird scope so “independent accessible user sessions pass” before scaling.

- Milestone 4 account lifecycle and quiet visits: The rationale is proving visitors cannot influence simulation, revocation/expiry pass, deletion purge works and private data stays out of metrics.

- Milestone 5 full species/count envelope: The why is to test aged 3/5/7-bird aviaries, stabilize captions/mix/perches and withhold seven until recognizability and budgets hold.

- Launch ramp: Internal synthetic accounts, opted-in closed beta and small cohorts are gated on operational health and designed-surface reviews, with no launch progress indicators or production interaction aggregation.

- Versioning and migrations: Engine coefficients and motif grammars are versioned, data migrations reversible, and render-plan decoder compatibility retained so bird signatures and newer drift survive rolling deploys and rollback.

- Incident and rollback behavior: Tick pauses retain events/cursors, audio failure switches to silence/captions instead of recordings, and invite disablement preserves revocation access.

### 13. Principal risks and responses

- Drift too fast/slow or saturation convergence: The response is bounded positive deltas, presence-dominant weights, persisted signal decay, synthetic histories and blind review, with prospective coefficient changes only.

- Lost continuity or duplicate drift: The response is worker transactions, unique event IDs, cursor atomicity, restricted writer roles and restore drills, blocking releases that overwrite traits with old snapshots.

- Presence inflation and battery waste: The response is conjunction tests, interval bounds, device union, hidden render/audio suspension and no “open tab” attention.

- Audio uncanny or indistinct: The response is six signature grammars, stable bird signatures, bounded continuous variation, listening trials and delaying higher-count rollout if seven is not recognizable.

- First frame feels started rather than ongoing: The response is inline snapshot, time-offset pose evaluation, small critical payload and reserving first-adoption fly-in as the sole empty-scene exception.

- Accessibility regressions: The response is a separate reduced-motion renderer, shared call-caption descriptors, narration tests and hands-on users in every milestone.

- Privacy creep: The response is database permission separation, strict telemetry schemas, no body/state logging, contact isolation and contract tests; product health is not engagement dashboards.

- Visit misuse or revocation delay: The response is scoped tokens, expiry/replay protection, every-pull authorization, short polling, no offline persistence and explicit visitor-event rejection.

- Operational tick cost: The response is due-work partitioning, queue-age monitoring, bounded event batches, deterministic server catch-up and scaling capacity rather than moving simulation to clients.

- Requirements ambiguity: The response is to retain documented export, notification, timezone and top-bar interpretations because they preserve the strongest product constraints until specification review changes them deliberately.
