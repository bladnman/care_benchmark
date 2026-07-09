## System-level intent

- **One canonical, server-owned continuity.** The plan repeatedly centers "one canonical, server-owned aviary" and "one aviary per account." The client "never owns or computes durable bird personality"; it submits facts and renders an authoritative snapshot. This appears in the delivery contract, client/server responsibility split, server-only personality writes, event sequencing, and the definition of done where two devices must show "one continuous canonical aviary."

- **A calm living place, already continuing.** The product is meant to feel like "a two-bird, one-screen place that is already alive on first render," not like a loading app. This shows up in "calm session," "snapshot-driven rendering presents a bird already in motion," "quiet field" fallback, "never a spinner," and the risk that "first paint reads as loading" because it violates the "already continuing" conceit.

- **Bounded care without urgency, guilt, or optimization.** The plan says "No feature should manufacture urgency around absence" and forbids "visible distress" or a "recovery task." Drift is slow, monotonic, and cannot be negative because of lack of presence. This also appears in the exclusions for "streaks, badges, levels, scores, ranks," the offer semantics, settle semantics, and the risk that too-fast or absence-tied drift "turns care into optimization or guilt."

- **Personality is hidden, durable, and expressed through behavior.** The plan treats personality as "hidden server-only normalized values" while public surfaces show mood expression, perch, calls, greetings, notebooks, and render signatures. It forbids "user-visible trait values," mood dashboards, client-side personality writers, and arbitrary state patch APIs.

- **Two copy domains protect product voice.** The plan explicitly splits **Aviary domain** copy from **System domain** copy. Aviary copy is "lower-case, present-tense, specific naturalist observations" for greetings, notebooks, narration, offer responses, and captions. System copy is "conventional, direct, matter-of-fact" for auth, settings, unsupported browsers, deletion, export, and errors.

- **Privacy by minimization and aggregate-only operations.** The plan keeps email encrypted on one account record, forbids email as an identifier or telemetry label, separates the simulation store from telemetry, and allows only "aggregate-safe operational counters." RUM, dashboards, traces, and analytics must not include account IDs, bird IDs, names, exact presence, invitation identity, notebook facts, or personality values.

- **Visits are deliberately separate and read-only.** The account surface for invitations is "deliberately separate," visitor sessions are "read-only," and visitors "cannot affect the host state." The plan rejects a "global social graph," "profiles, discovery, follows, chat, comments, co-presence, avatars, and public sharing."

- **Accessibility is a parallel primary experience.** The plan says accessibility is "not a final ARIA pass" but a "parallel primary experience." This principle appears in prose narration, runtime call captions, keyboard operation, contrast, reduced-motion cross-fades, semantic controls, and manual assistive-technology test requirements.

- **Deterministic, versioned, retry-safe simulation.** Simulation is kept in a "versioned simulation module" with seeded pseudo-random behavior, calibration versions, transactionally updated cursors, row locks, and deterministic tests. The same intent appears in retry safety, duplicate delivery handling, catch-up, seeded call opportunities, mood transitions, and calibration migration.

- **Negative space is part of the product definition.** The plan treats absences as hard requirements: no native clients, password/SSO, multiple/shared aviaries, scene customization, raw trait UI, recorded-call fallback, social feeds, push/email engagement notifications, gamification, public sharing, or extra chrome. Final release review must inspect this "negative space" and says any such item is "a v1 scope failure."

## Per-feature whys

### 1. Delivery contract and scope guardrails

- **Responsive browser application with one canonical, server-owned aviary:** The plan wants a single durable source of truth for each signed-in account; the browser "renders and submits facts" but "never owns or computes durable bird personality."

- **Two-bird, one-screen first shipped experience:** The plan frames the first release as a "calm session of watching, listening in on one bird, making a bounded offer, settling the scene, reading a generated Field Notebook," with the scene "already alive on first render."

- **Email magic-link identity:** The plan excludes password and SSO login and gives magic links 15-minute one-time consumption, rate limiting, and non-account-enumerating responses so authentication stays direct, bounded, and system-domain.

- **Per-device revocable sessions:** Sessions are revocable individually and carry only account-scoped claims so a person can manage devices without changing the aviary itself.

- **One aviary per account:** The plan repeats "one aviary per account" and rejects multiple/shared aviaries so continuity stays canonical rather than split across places.

- **Starter adoption/naming:** NOT RECOVERABLE FROM PLAN

- **Rename:** The plan states bird identity "never changes on rename" and rename must "preserve bird ID," so naming changes do not rewrite the durable relationship.

- **Account export:** Export is a "private account artifact, not an internal analytics feed" and is delivered only to the currently verified email with a short-lived link.

- **30-day recoverable deletion:** Soft-delete makes the account inaccessible immediately, recovery inside 30 days restores the same records and bird identities, and hard-delete purges account-linked rows, objects, tokens, invitations, logs, and mappings after day 30.

- **Server-side, minute-scale simulation:** Persistent birds, personality drift, mood, weather, adoption eligibility, and notebook observations must evolve from server-owned simulation rather than a client clock or client-written state.

- **Visual and audio aviary:** The aviary should support watching, listen-in, offers, settle, day/night, weather, perch zones, personality- and mood-shaped motion, "real procedural calls," and interpolation so the place feels alive without recorded-call fallback.

- **Host-only presence, multi-device consistency, and read-only visitor sessions:** Only host events can contribute to drift, multi-device state must remain canonical, and visitor sessions "cannot affect the host state."

- **Designed accessibility:** The plan includes narration, captions, keyboard operation, contrast, and reduced motion because accessibility users should receive "a designed living aviary."

- **Aggregate-only instrumentation and performance controls:** These protect "the first-bird and long-session budgets" while preventing per-account simulation or interaction data from becoming telemetry.

- **Hard scope exclusions:** Exclusions prevent the product from becoming native-client work, a multi-aviary/social platform, a scene editor, a trait dashboard, a recorded-audio app, or a game with "streaks, badges, levels, scores, ranks."

- **Visit notifications off by default:** The only notification-like social behavior is an explicit host setting, off by default, and "must not become an onboarding or badge surface."

- **Aviary domain and System domain copy:** The copy split keeps naturalist observations lower-case, present-tense, and specific, while authentication, sessions, settings, accessibility, deletion, export, and errors stay "matter-of-fact."

### 2. System shape and bounded responsibilities

- **Modular web application:** Edge client, stateless API, transactional simulation worker, relational source of truth, email delivery, and aggregate-only telemetry divide responsibilities so the simulation, transport, rendering, and operations surfaces do not blur.

- **Simulation data store isolated from telemetry and analytics credentials:** The plan says there should be "no warehouse replication or query path from per-account simulation records," protecting private aviary state from analytics use.

- **Client authenticated account or constrained visitor session:** The browser establishes either host authorization or constrained visitor authorization so host and visitor capabilities remain separate.

- **Snapshot fetching, reconciliation, and ephemeral interpolation:** The client may interpolate render state and keep a short encrypted/session-scoped cache only to render a quiet field and accelerate re-open; it must immediately reconcile with the server and never treat the cache as durable.

- **Client presence signal collection:** The client gathers only the raw browser signals necessary to emit bounded, idempotent presence-window events while all required signals are true, avoiding raw mouse coordinates, keystrokes, or behavior traces.

- **Immutable interaction intents:** The client sends listen-in, offer, settle, adoption/naming, and setting-change intents rather than calculations or absolute bird state so the server remains the authority.

- **Deterministic scene composition from authoritative snapshot plus local ornament seeds:** Client-only leaf/feather ornaments are explicitly "not simulation state," preserving visual variation without durable client state.

- **Procedural audio and captions from server-supplied grammar/signature data:** The client synthesizes calls and captions from authoritative call data while managing AudioContext lifecycle and silent-captioned fallback.

- **Semantic controls, live narration, focus behavior, and reduced-motion renderer:** These are client responsibilities because accessibility is designed as part of the experience rather than bolted on later.

- **API and simulation layer owning identity, authorization, durable records, events, ticks, invitations, notebooks, and snapshots:** The plan assigns these to the server so exact event sequencing, revocation checks, and durable simulation cannot be bypassed.

- **Simulation worker as the only personality and durable mood writer:** Restricting writes to one process role prevents API handlers and rendering code from becoming a "hidden engine."

- **Relational transactions and row-level locking or serializable transactions:** These serialize account/aviary simulation work so retries, overlap, and duplicate jobs cannot corrupt canonical state.

- **Queue plus independent elapsed-tick detection:** The queue can schedule due aviaries, but the transaction must independently detect elapsed ticks and serialize per aviary "so duplicate deliveries or worker retries cannot apply a delta twice."

- **Object storage only for export artifacts with short-lived links:** Export objects are private, single-account artifacts and should not become cached or exposed account state.

- **Versioned simulation module separate from transport code:** This boundary "lets calibration evolve without making HTTP handlers or rendering code the hidden engine."

- **Edge app shell and compact scene renderer for first render:** The plan says not to block initial bird paint on settings panels, notebook history, invitations, export controls, or noncritical audio initialization.

- **Quiet field fallback on slow snapshot:** A slow snapshot should show "a quiet field with restrained ambient cues," never a spinner, skeleton, toast, wake-up animation, or static placeholder presented as the aviary.

### 3. Persistent model, invariants, and retention

- **Opaque UUIDs and encrypted email single-home rule:** Email is never an identifier, event attribute, trace attribute, queue key, partition key, or telemetry label, reducing PII spread.

- **Immutable facts/events distinguished from current simulation projections:** The plan says repair and audit should not rely on reconstructing personality from history.

- **Account record:** The account owns status, deletion lifecycle, timezone, and one aviary; old verified email remains until a new address is verified so email changes do not break access.

- **Auth magic link record:** Hashed opaque tokens, 15-minute expiry, and atomic one-time consumption avoid storing raw tokens and protect against replay.

- **Device session record:** Hashed session token, last-seen/revoked/expires fields, and minimal device display metadata support individual revocation and account-scoped authentication.

- **Aviary record with canonical version, last simulated time, weather, settled state, and adoption eligibility:** This record keeps exactly one aviary per account and increments version for externally observable durable changes.

- **Bird stable UUID:** Bird identity stays stable across rename, migration, and species catalog changes.

- **Bird personality record:** Hidden server-only normalized values are updated only by the simulation transaction as additive, clamped deltas with "no decrease from neglect."

- **Bird mood record:** Mood is a "persistent fast-timescale projection, not a client derivation"; it carries through sessions and evolves between them.

- **Interaction event record:** Immutable, ordered, idempotent events let host events contribute to drift while server sequence remains authoritative and payloads/cooldowns are validated on insertion.

- **Presence window record:** Stores server-validated intervals and bounded accumulated seconds, not raw mouse coordinates or keystrokes; visitor sessions cannot create rows.

- **Simulation cursor:** The cursor makes tick application "resumable and exactly-once relative to event range."

- **Scene snapshot projection:** A compact materialized read model supports API/edge delivery of current perch/action/call schedule, day phase, weather, and greeting eligibility.

- **Notebook entry record:** Entries are sparse, immutable, read-only, and rate-limited so the notebook never becomes "per-session event logging."

- **Offer cooldown:** Server-side cooldowns make offers bounded; refusal is "neutral and not punitive."

- **Visit invitation:** Explicit invitations expire unused after 30 days, are instantly revocable, and do not create "a global social graph."

- **Visit session and visit log:** These provide read-only authorization and a transparency audit while exposing only email/date/approximate duration and outstanding invites in settings.

- **Account setting:** Accessibility overrides, captions, visit-notification opt-in, and privacy acknowledgement live on a system surface with "no engagement flags or public visibility field."

- **Export job:** On-demand JSON is delivered only to the currently verified email using a short-lived link.

- **Soft-delete, recovery, hard-delete, and backups:** Pending deletion blocks normal access, simulation, and invitations; recovery reinstates the same identities; hard-delete purges linked rows and objects; backups must not restore deleted accounts into active service without tombstones.

- **Account export contents:** Export serializes birds, current vectors, moods, notebook entries, and settings with schema version and generated-at timestamp because it is a private account artifact.

### 4. API, authorization, and delivery contracts

- **JSON over HTTPS with versioned routes/media types, idempotency keys, ETags, and matter-of-fact errors:** This gives sync and cache boundaries while keeping auth/sync failures in system-domain copy.

- **Normal API objects omitting hidden personality values:** Hidden personality values stay out of host and visitor responses; export is the explicit account-settings exception.

- **Magic-link routes:** Creation rate-limits without account-enumerating responses, and consumption atomically creates/rotates a per-device session with bootstrap metadata.

- **Session refresh/logout/list/revoke routes:** These issue, rotate, or revoke only the caller's sessions to keep account control per-device and scoped.

- **Email change routes:** The new address is committed only after verification, while the old address remains valid until then.

- **Account settings route:** Reads and writes accessibility and permitted social settings separately from the aviary scene bundle.

- **Account export route:** Enqueues export after reauthentication if required and does not return state in a potentially cached/exposed API response.

- **Account deletion and recovery routes:** These start or cancel the 30-day lifecycle and use system-domain copy.

- **Aviary bootstrap route:** Provides a compact authenticated first-paint snapshot with public bird render information, greeting directive, narration seed, and version, but "never includes numeric traits."

- **Aviary snapshot route:** Supports deltas or 304/minimal response when possible, requires full snapshots after long suspension/version gaps, and is called on visibility regain, frame-gap recovery, and low-frequency visible keepalive.

- **Aviary events route:** Host-only append returns accepted event IDs, not recomputed personalities, and is "not a general patch API"; schemas make personality, mood, perch, or arbitrary notebook text impossible to submit.

- **Notebook route:** Cursor-paginated, immutable entries are lazy-loaded only when opened so history can be indefinite without burdening first paint.

- **Adoption and bird patch routes:** Eligibility, age, and cap are enforced server-side; starter naming happens in onboarding; rename preserves ID; there is no catalog/rarity/purchase endpoint.

- **Visit invitation creation, list, and deletion routes:** A host creates and manages one invitation for a named email without revealing account existence; deletion revokes immediately.

- **Visit consume and visit snapshot routes:** A valid unexpired, unrevoked invite issues narrowly scoped visitor access; snapshot reads exclude greetings, interaction affordances, traits, notebook mutation, account data, and controls, and revocation is checked on every request.

- **Visitor route guards, logging, and notifications:** Mutable controls must not be exposed through DOM, shortcuts, or auth gaps; visit logging starts from authorized snapshot access; duration is approximate; host notification is explicit opt-in with silent log by default.

### 5. Simulation engine and canonical tick

- **Minute scheduler plus on-demand snapshot catch-up:** This keeps active aviaries due for simulation and can catch up at authenticated snapshot read without trusting the client.

- **Canonical tick time from server UTC plus account local timezone:** The plan says to "never trust client wall clock for state" and use local time only for day phase.

- **Per-aviary cursor/row lock and tick transaction:** Each tick consumes accepted events in server sequence order, updates projections and cursor atomically, and releases the lock so overlapping requests do not race.

- **Retry and duplicate safety:** The cursor, event idempotency, deterministic intervals, snapshot catch-up constraints, and calibration versions ensure retries reach the same result and do not apply a delta twice.

- **Extreme downtime degradation path:** After a threshold, time can simulate in coarser deterministic intervals while preserving mood/day boundaries and never inventing interaction or presence input.

- **Client presence state machine:** Presence qualifies only when visible, focused, recently active, and not settled; raw pointer/key data remains local.

- **Server presence validation:** Heartbeats are checked against session age, receipt time, interval length, monotonic client sequence, and clock skew; lost network or frozen tabs "fail closed."

- **Settle:** Settle ends qualified presence exactly like loss of attention, applies visual/audio settle projection, has a five-second undo/reengagement path, and grants neither penalty nor missing-step state.

- **Personality drift evidence:** Qualified presence is the largest small saturating input; listen-in influences the focused bird; offers add small boldness/curiosity signals; settle affects only near-term mood/scene.

- **Nonnegative drift and no absence punishment:** Updates are low-pass, bounded, additive, clamped, and "can never be negative because of lack of presence, inactivity, elapsed time, rejected offers, or a mood transition."

- **Calibration tests and versioned calibration:** Product-outcome tests target roughly seven-day measurable changes, roughly three-week perceptible behavior, no one-session visible shift, and edge cases like absence, overlapping devices, offer spam, clock changes, and visitor watching.

- **Durable finite-state mood:** Mood does not reset on page open and transitions from current mood duration, local day phase, weather, interactions, nearby bird events, and trait-weighted probabilities.

- **Server-authored rare, soft, bounded weather:** Weather is deterministic at aviary level, limited to short rain and occasional wind, with "no severe weather feature."

- **Scene projection and perches:** The projection chooses perch zones from mood and boldness, prevents collisions, keeps every bird in frame, and includes action phase offsets so first paint starts mid-preen/scan/call.

- **Calls as opportunities:** Snapshots include timing and motif seeds rather than durable audio state, allowing clients to render mood/signature while avoiding simultaneous return cues from every bird.

- **Greeting directive:** On qualifying host return/open, an ephemeral bird-led greeting derives from absence duration, mood, traits, and scene, with fairness and optional stagger; it is not a text welcome, not an event, and not for visitors.

- **Offer request and resolution:** Offers are bounded gestures from the top bar, resolved by current state with per-bird cooldown, neutral rejection, structured outcome, and no feeding, maintenance, score progress, or forced trait.

- **Notebook generation:** Entries come from deterministic/reviewable templates over structured noteworthy domain facts, are spaced and deduplicated, and must never report attendance, streaks, raw traits, feed-like timestamps, or visit-frequency behavior.

- **Later adoption:** New-bird eligibility is based only on aviary age and quiet cadence, not clicks, presence totals, subscription, social activity, or performance; there is no rarity economy and max seven is server-enforced.

### 6. Client rendering, interaction, and visual composition

- **Isolated scene component with declarative state adapter:** This keeps the scene renderer focused and lets account/settings/notebook/invitation pages stay outside the scene bundle through code splitting.

- **Scene graph layers:** Back-to-front layers separate sky, perches, birds, weather, local ornaments, captions/focus, and a thin top bar so semantic focus targets and captions can sit over a performant animated scene.

- **Normalized layout, safe viewport constraints, and no scene pan/zoom/scroll:** The plan wants narrow screens to adapt without cropping, hiding, or letting a bird leave frame, and rejects user-positioned scene control.

- **Virtualized Field Notebook rows:** The notebook may scroll as a separate panel, but virtualized rows must not retain unbounded rendered nodes.

- **Snapshot interpolation:** The snapshot is an authoritative target; the client estimates only render time, eases toward snapshots, and uses cross-fade/reposition after long gaps instead of client-simulated long flight.

- **Initial renderer before detail/physics/audio:** First paint should show an existing still or mid-action pose from bootstrap fields without waiting on notebook, weather assets, settings, species library, or audio permission.

- **Mood-shaped micro-actions and day states:** Preen, scan, head tilt, shuffle, call posture, rest, approach, and response combine with deterministic offsets so birds do not synchronize and behavior expresses mood.

- **Day/night and settled rendering:** Local-day color interpolates continuously; evening is warm and quieter; the settled view uses a slow lighting/mix change and five-second reversal.

- **Minimal top bar that fades:** The top bar contains only account/settings, accessibility, Field Notebook, offer, and settle controls; it fades nearly transparent after stillness and returns on pointer or keyboard activity.

- **Pointer and keyboard interaction/focus model:** Shared dispatch, predictable focus order, arrow movement among birds, Enter for listen-in, Escape to exit, and high-contrast focus outline make interaction consistent and accessible.

- **Offer UI and authoritative outcome rendering:** Offers originate in the top bar, use a compact keyboard-operable chooser, send one-time intent, and render only from authoritative response/event/snapshot rather than optimistic trait changes.

### 7. Procedural audio, captions, and fallback

- **Species call grammar and stable bird signature:** Motif primitives, synthesis parameters, descriptor grammar, stable ID/species seed, and mood/trait variation "preserves recognition across weeks while preventing exact repeats."

- **WebAudio voice pool, buses, scheduler, and listen-in ramps:** Reusable voices and bounded scheduling avoid per-chirp allocations; listen-in raises the focused bus while attenuating, never silencing, others so attention moves rather than tracks switching.

- **Autoplay, silence, and recorded-call fallback:** Audio enablement should not block the visual aviary or use coercive prompts; if audio fails, graceful silence with captions is used, and recorded clips are never downloaded.

- **Runtime call captions:** Captions derive from the exact procedural motif/envelope, appear near the bird without covering controls, fade with the call, remain AA-readable, obey reduced motion, and avoid noisy live-region duplication.

- **Audio tests and profiling:** Deterministic motif renders, no-exact-repeat windows, distinguishability, ramp timing, seven-bird stress, cleanup, suspend recovery, denial, fallback captions, and 30-minute profiling prove calls remain bounded and usable.

### 8. Accessibility design and verification

- **Shared state-to-language formatter:** It uses the same public snapshot/procedural event data as rendering, with naturalist lower-case present-tense templates, and never discloses numeric traits or raw mood lists.

- **Screen-reader narration controller:** Initial scene observation, 30-60 second meaningful-change idle updates, prioritized greeting/offer/settle observations, dedupe, interruption policy, and queue caps avoid flooding the reader.

- **Bird, offer, and settle semantic controls:** Birds are named interactive controls for listen-in, while offer and settle have explicit buttons and status feedback without exposing invisible personality values.

- **Reduced-motion rendering adapter:** It honors `prefers-reduced-motion` and persistent override while keeping birds, scene states, day color, audio, captions, mood/drift, and notebook behavior; it uses cross-fades rather than pausing a live loop or showing a static illustration.

- **Contrast, text scaling, zoom, narrow viewports, target sizes, and color-independent cues:** These keep every user-copy surface readable at WCAG AA across morning/evening/night palettes and dynamic backgrounds.

### 9. Security, privacy, and operational boundaries

- **Secure sessions, token hashing, replay protection, rate limits, CSRF mitigation, authorization, and redacted logs:** These protect magic links, sessions, invitations, state-changing routes, cross-account access, and PII-bearing data.

- **Data-access boundaries for service accounts and network policy:** Simulation can read/write canonical store; telemetry receives allowlisted aggregate events only; analytics/ML have no path to simulation/event/notebook data; email receives only delivery address for the transactional message.

- **Threat modeling and CI privacy/authorization tests:** The plan names replay, stolen tokens, forwarding, revocation during visits, cross-account access, duplicate delivery, malicious presence spam, client clock manipulation, renderer/audio denial of service, and accidental PII, and requires CI tests rather than documentation-only controls.

### 10. Performance budgets, telemetry, and quality gates

- **Release performance gates:** JavaScript size, first real bird under agreed p75/p95 targets, 60 fps idle, 30-minute memory stability, tick p99, and kilobyte snapshot payloads protect the living-scene feel and first-bird/long-session budgets.

- **Synthetic monitoring and aggregate RUM:** Synthetic browsers track availability and first-bird/render/snapshot failures; real-user measurements are only coarse, aggregate timings, frame buckets, audio errors, API/tick counts, and anonymized session-duration bins.

- **Dashboards and alerts:** Aggregate health dashboards watch tick p99, bootstrap/auth/visitor errors, first-bird regressions, frame degradation, and audio fallback spikes without segmentation by account or bird.

- **CI quality gates:** Deterministic simulation, monotonic drift, migrations, API authorization, browser integration, accessibility scans/manual scripts, visual regression, audio memory, load contention, and privacy schema linting prevent regressions across the product boundaries.

### 11. Incremental rollout plan

- **Phase A foundations and calibration harness:** Core identity, schema, idempotency, cursor/locking, deterministic simulation, snapshots, fixture aviaries, telemetry allowlists, and performance environment are built before feature work spreads.

- **Phase B private internal vertical slice:** A small internal cohort validates starter birds, bootstrap, renderer, tick, presence, mood, calls/listen-in, settle, narration, and reduced motion; notebook waits until structured facts and rate limits exist; synthetic monitoring calibrates drift rather than staff histories.

- **Phase C controlled external beta:** Magic-link onboarding, notebook, offers, export/deletion, captions, keyboard, and settings go to a small opt-in cohort; later adoption stays unavailable until tick, audio, and long-session metrics are stable, and expansion keeps the product free of progress framing.

- **Phase D visits and broader ramp:** Invitations wait until host-only privacy guarantees and account controls pass review; revocation, expiry, render-only controls, log transparency, and silent-default notifications are tested, while social discovery/co-presence is explicitly not a follow-up.

- **Phase E general availability and ongoing safety:** GA waits for browser budget pass, accessibility sign-off, staffed alerts, audited privacy access controls, and deletion/export drills; calibration changes remain versioned and canaried on synthetic fixture accounts, never recomputing historical personality vectors.

### 12. Risks, decisions, and mitigations

- **Drift calibration controls:** Drift that is too fast, too slow, or absence-tied "turns care into optimization or guilt," so the plan uses low-pass calibration, 1-week/3-week tests, no negative neglect drift, gates, and no retroactive vector rebuild.

- **Presence overcount protection:** Overcounting inactive tabs or spoofed presence corrupts personality and trust, so presence requires visible + focused + recent activity, bounded server validation, fail-closed behavior, no visitor events, and abuse limits.

- **Multi-device race prevention:** Users should not see inconsistent birds, so the plan requires server-only writes, append-only ordered events, per-aviary cursor/lock, idempotency, and no last-write-wins patches.

- **Tick outage and retry mitigation:** Canonical continuity must hold even without clients, so cursor/tick transactions, deterministic seeds, retry-safe intervals, catch-up, lag alarms, and replay drills prevent duplicate or skipped time.

- **First-paint loading prevention:** Loading states break the "already continuing" conceit, so compact bootstrap, mid-action pose fields, bundle gates, quiet-field fallback, and mobile first-bird tests are required.

- **Audio quality and memory mitigation:** Looped, harsh, or leaky audio makes calls lose recognizability/aliveness or degrades long sessions, so the plan uses grammar/signature tests, short-horizon scheduling, gain ramps, reusable voices, profiling, and silent captioned fallback.

- **Reduced-motion parity:** A stripped static fallback would exclude users from product quality, so a separate cross-fade renderer and visual regression suite must preserve feature parity.

- **Screen-reader narration quality:** Flooding or status dumps make the accessible experience unusable or emotionally flat, so the plan uses a shared prose formatter, 30-60 second cadence, dedupe, bounded queue, and manual assistive-tech testing.

- **Notebook anti-log controls:** A generic log or engagement record would destroy naturalist voice and reintroduce behavioral tracking, so templates, rate limits, dedupe, and content tests ban attendance/raw-stat phrasing.

- **Invitation anti-social-platform controls:** Invitation routes could violate privacy and change simulation meaning, so per-invite emails, constrained claims, render-only route, no graph/profile/discovery schema, and immediate revocation checks are required.

- **Telemetry PII prevention:** PII or relationship data in telemetry breaks the privacy commitment, so UUIDs, encrypted email, telemetry allowlists/lint, separate credentials/network boundaries, redaction tests, and access audits are required.

- **Browser and AudioContext degradation handling:** Browser policy differences could look broken or inaccessible, so the plan requires a browser matrix, progressive audio enablement, caption-first silent fallback, unsupported-browser system surface, and canary monitoring.

- **Extra chrome/stat/gamification guardrails:** Toasts, stats, streaks, or extra chrome would reframe the product as an app/game rather than a place, so acceptance checks, UI inventory tests, forbidden data surfaces, and owner sign-off guard new metrics.

### 13. Definition of done

- **V1 readiness definition:** The product is done only when a signed-in person sees one continuous canonical aviary across two devices; greetings are bird-led; host-qualified presence creates slow invisible drift; identities/moods persist; rendering begins in motion; audio/listen-in/offers/settle/notebook stay bounded; accessibility users get a designed living aviary; visitors observe only; revocation works promptly; and all performance, privacy, deletion/export, and browser gates pass.

- **Final negative-space release review:** The plan requires explicit inspection for no raw trait UI, gamification data/UI, absence punishment, recorded audio, scene controls/chrome, native-client commitments, social graph/co-presence/public discovery, or per-account interaction data in telemetry; any presence is a v1 scope failure.
