## System-level intent

- **Quiet everyday session, not game framing.** The plan opens by saying to "Build the quiet everyday session first" and repeatedly rejects "game framing, counters, streaks, distress, care schedules" and "notification-driven return loops." This shows up again in the offer language, where offers are "a gesture, not nutrition," and in the risk section, which says to reject "engagement counters/notifications" and social comparison.

- **Server canonical, client observational.** The plan's invariants say "the server alone owns persistent mood and personality" and "clients submit events, never state." This reappears in architecture, API contracts, simulation, sync, and release gates: the renderer has "no authority over canonical state," no client endpoint accepts mood/personality values, snapshots are observations, and launch checks include "no client trait writes."

- **Presence shapes birds slowly, without guilt or punishment.** Personality drift is "presence-based," "additive and monotonic toward expressive," and "neglect never subtracts traits." The simulation section calibrates change to be measurable after "about one week" and felt after "about three weeks," with "no visibly attributable session-sized jump." The risks reinforce that late return should feel "calm, not guilt-inducing."

- **Privacy is a product boundary, not only a security layer.** The plan forbids private interaction history in analytics, keeps operational telemetry "isolated from simulation storage," uses encrypted email and synthetic UUIDs, avoids account dimensions in metrics, and requires deletion cascades through events, notebooks, vectors, sessions, invitations, visit logs and telemetry linkage.

- **Continuity and identity matter more than optimization shortcuts.** Birds have "stable immutable ID[s]," IDs are never reused, calls preserve a "recognizable signature," and identities and vector values are preserved through migrations and catalog changes. Multi-device snapshots and canonical revisions also serve continuity across devices.

- **Accessibility ships with v1 as a parallel experience.** The product scope includes screen-reader narration, captions, and reduced-motion rendering. The accessibility section says to "Ship accessibility surfaces with v1," generate narration from the same snapshot, keep captions grammar-derived from the rendered sound, and make reduced motion a designed cross-fade system rather than a frozen scene.

- **Generated content must be constrained, specific, and grounded.** Procedural calls use a compact grammar and deterministic seed. Captions come from "the same grammar decision" as audio, and notebook prose comes from "typed observations using constrained templates and state facts." The plan says to avoid unsupported claims and exclude attendance frequency.

- **Visits are optional, read-only, and host-controlled.** The plan allows optional read-only visits but says a visitor "never contributes host presence or events." Invite tokens are single-invite scoped, expiring and revocable, with a host transparency log. Visit notifications default off, and visitor credentials reject mutating routes.

- **Performance and correctness are launch requirements.** The plan treats bundle size, first-bird time, 60 fps idle motion, memory stability, tick idempotency, deletion, accessibility, contrast, and caption/audio matching as "release gates rather than deferred polish."

- **Product voice has two registers.** The plan calls for naturalist voice in the scene and notebook, with lowercase naturalist prose, while sign-in, settings, sync, errors, unsupported browsers, and account surfaces use "direct matter-of-fact copy."

## Per-feature whys

### 1. Product scope and invariants

- **Browser-only aviary:** NOT RECOVERABLE FROM PLAN

- **Single-user aviary:** The plan frames v1 around a "quiet everyday session" and explicitly avoids "shared aviaries," "public discovery," and social comparison, so the single-user shape protects the private, non-public tone.

- **Two system-selected starter birds:** The new aviary should begin already alive, with "two species from a coherent pool" and "distinct but stable identities"; this supports the first-paint session where the aviary is "already moving" and one bird can notice the returning user.

- **Room to grow to seven:** The cap is tied to recognizability and audio manageability: the plan says to "limit to seven" when preventing calls from blurring and to ramp additional birds only after "call recognizability" is acceptable.

- **Email magic-link accounts:** The plan uses magic links for account access, with one-use 15-minute links, per-email rate limiting, and per-device sessions, giving account continuity without putting email into IDs, logs, trace attributes, shard keys or event payloads.

- **One canonical aviary per account:** The plan wants "one canonical aviary per account" so all devices read the same revision, snapshots observe canonical state, and clients do not fork or resolve personality/mood with last-write-wins.

- **Multi-device snapshots:** Snapshots let clients paint from the latest canonical state, refresh on return or gaps, and preserve continuity "through sign-out/sign-in and two devices" without exposing conflict dialogs for ordinary stale state.

- **Field notebook:** The notebook records sparse, immutable observations from typed state facts so it can feel "specific, lowercase, present tense, and sparse" without claiming unsupported history or turning visit frequency into content.

- **Presence-based personality drift:** Presence is the main input because the plan wants accumulated personality change from regular presence, while requiring monotonic deltas, no negative drift on absence, no per-session jump, and no exposed numbers.

- **Optional read-only visits:** Visits allow sharing the live host scene while preserving privacy and canonical correctness: visitor scope is read-only, checks revocation, does not receive host controls by default, and never generates host presence, drift or interaction events.

- **Procedural calls:** Calls are procedural so each bird can keep a recognizable signature across mood and drift, vary timing/pitch/motif, synthesize locally, avoid recordings, and produce captions from the same grammar decision as the rendered sound.

- **Screen-reader narration:** Narration exists so screen-reader users receive one coherent scene from the same snapshot, in naturalist prose, without raw state lists, trait values, duplicate speech or a flooded queue.

- **Captions:** Captions make rendered calls available when audio is off or unavailable; the plan keeps them grammar-derived from the same decision as audio so the caption "always describes the sound rendered."

- **Reduced-motion rendering:** Reduced motion preserves the aviary experience by replacing flights and frame motion with still poses and cross-fades while mood, calls and notebook behavior continue.

- **Visit notifications default off:** The default supports the rejection of "notification-driven return loops"; if enabled, it must use only the stated setting and not create push infrastructure in v1.

- **No welcome banner, visit badge, or visit-frequency notebook entry:** These omissions support the plan's avoidance of textual welcome or absence messages, visit-frequency observations, engagement counters, and guilt-producing return loops.

- **Account export and deletion:** Export and deletion serve privacy and lifecycle control: export is generated on demand and delivered through verified email, while deletion is recoverable for 30 days and then hard-deletes account-linked records and telemetry linkage.

- **Device-session revocation:** Device-specific revocable sessions let account access continue across devices while giving users a way to end individual sessions.

- **Per-bird rename:** Renaming changes only display name while preserving immutable bird ID, species identity, personality values, and continuity through migrations or catalog changes.

- **Age-paced additions:** New bird availability depends only on elapsed aviary age, below the seven-bird cap, so additions are presented as offers and not as reward counters, interaction scores, visit counts or payments.

- **Visit log and invitation revocation/expiry:** These give the host transparency and control: a minimal visit record shows visitor email and approximate duration, while invites expire after 30 unused days and can be revoked.

- **Matter-of-fact account/accessibility/error surfaces:** The plan gives support surfaces a direct register so sign-in, settings, accessibility settings, sync and errors stay clear rather than adopting the scene's naturalist voice.

- **Lazy-loaded supporting surfaces:** Account settings, accessibility settings and visit flows are lazy-loaded to keep the initial scene small and draw the first bird before noncritical UI.

### 2. Architecture and ownership

- **Web app / renderer:** The renderer paints fast from the latest canonical snapshot, interpolates locally, handles input and procedural audio, and stays non-authoritative so transient animation cannot mutate canonical state.

- **Identity and account API:** The account service owns magic-link issue/consume, sessions, email changes, export and deletion so email access and lifecycle operations stay bounded to account logic.

- **Synthetic UUID account key:** Synthetic UUIDs keep email out of IDs, logs, telemetry, service boundaries and most payloads, limiting where decryption is needed.

- **Aviary API:** This API centralizes snapshot reads, append-only event writes, naming, notebook, settings and visit tokens, with every operation authorized against account ID and resource ownership.

- **Simulation worker:** The worker processes ordered events and commits mood, personality deltas, world state and notebook candidates transactionally, with idempotent ticks so retries cannot double-apply drift.

- **Invitation/mail boundary:** This boundary sends sign-in and invite links while storing only token digests and invite metadata, so visit access is a read-only capability to one aviary until revoked or expired.

- **Operational telemetry:** Telemetry is isolated and aggregate-only so request, latency, error, frame and audio metrics cannot reconstruct account, bird, mood, offer, presence or personality state.

- **Transactional relational store:** The plan chooses transactional persistence so event consumption, canonical state, cursors, revisions and outbox work can commit together.

- **Append-only interaction events:** Events remain append-only for deterministic tick consumption and operational debugging within retention and deletion policy.

- **Canonical current state:** Keeping current state separately enables fast snapshots from the canonical revision.

- **Outbox or equivalent transactional publication:** The outbox prevents committed domain changes from being lost between the database and queue for email, notebook generation and tick scheduling.

### 3. Data model

- **Account record with encrypted email, timezone, preferences and deletion fields:** The account record supports encrypted mail handling, local day-cycle inputs, accessibility/audio preferences, visit notification settings and account deletion lifecycle.

- **DeviceSession record:** Device-specific token hashes and revoked timestamps support revocable per-device sessions.

- **Aviary record with unique account, revision, cursor and settle state:** A unique account aviary enforces one canonical aviary; revision rejects stale snapshots, and cursor supports deterministic simulation progress.

- **Bird record with immutable UUID, species, name, personality, mood, perch and seed state:** The record preserves identity, hides server-only trait scalars from product UI, and keeps mood, perch and call/animation seed state canonical.

- **InteractionEvent record:** Typed, idempotent, sequenced events let clients submit evidence while the server validates ownership, caps payloads, orders by receipt time, and applies rate limits.

- **PresenceInterval or compact presence-event segments:** Presence segments capture only qualifying visible, focused, recently active intervals, avoid raw pointer/key streams, and union overlapping devices so one account cannot accrue parallel presence time.

- **NotebookEntry record:** Immutable, sparse entries keep observation prose tied to a source revision and exclude visit-frequency observations.

- **VisitInvite record and minimal visit record:** Token digests, expiry, revocation, first use and approximate duration provide scoped access plus host transparency without treating visits as presence.

- **BirdOfferEligibility:** Deriving eligibility from age and adoption history ensures it does not depend on visit count, interaction score or payment.

- **Stable enumerations for moods, offers and species/motifs:** Stable enums keep mood, offer and motif contracts predictable across server, client and content generation.

- **Versioned server configuration for calibration parameters:** Versioned configuration lets trait ranges, seeds, cooldowns and presence windows change only through reviewed calibration releases.

### 4. API and event contracts

- **Versioned JSON endpoints or typed RPC:** Versioning and typed contracts keep clients and services aligned while using authenticated sessions, request IDs, idempotency keys and a consistent error envelope.

- **Matter-of-fact error envelope:** The error envelope supports the direct account/sync/error copy register and keeps account-level failures clear.

- **No client endpoint accepts personality or mood values:** This enforces the invariant that clients submit events, never state, and that no client can write canonical trait or mood values.

- **Magic-link issue and consume endpoints:** One-use 15-minute links, atomic invalidation and per-email rate limiting protect authentication and issue per-device sessions.

- **Email change verification:** Verifying the new address before changing encrypted canonical email protects account access.

- **Snapshot endpoint:** The snapshot returns enough derived presentation cues to reproduce the same scene while omitting raw personality values and keeping server time, local inputs and current canonical revision authoritative.

- **Snapshot refresh triggers:** Refresh on visibility return, long frame gap and visible-tab keepalive keeps clients synced with canonical state without client absolute writes.

- **Events endpoint:** Bounded typed events with idempotency keys, server sequencing and ownership validation allow concurrent devices and retries without duplicate events or direct trait updates.

- **Host-only presence signals:** Accepting presence only from authenticated host sessions prevents visitors from contributing drift.

- **Notebook paging endpoint:** Reverse chronological immutable pages expose notebook observations without making entries mutable.

- **Display-name patch endpoint:** Display-name-only changes allow per-bird rename without changing identity, species, traits or mood.

- **Available-bird and adoption endpoints:** Age-derived availability below the cap presents a new bird as an offer, never as a reward counter.

- **Account preference, session, export, deletion and policy endpoints:** These cover account control, revocation, verified-email export delivery, recoverable deletion, hard deletion and privacy policy access.

- **Host invitation endpoints:** Invite creation, outstanding invite list, visit log and revocation give hosts explicit control while token digests, unguessable capabilities, rate limits and non-enumerable IDs reduce abuse.

- **Visitor snapshot endpoint:** Capability validation returns the exact live host scene in read-only form, checks revocation on every pull, rejects mutating routes, and records approximate duration without treating it as presence.

- **Same unavailable response for revoked or expired links:** Returning the same response avoids leaking invite status.

- **Optimistic revision checks for account/settings edits:** Revision checks help ordinary account/settings conflicts without using last-write-wins for simulation state.

### 5. Simulation and content engine

- **Due-tick scheduler with locks or compare-and-swap:** Locking or revision compare-and-swap lets each aviary tick be applied once.

- **Single-transaction tick processing:** Consuming events, accruing presence, computing summaries, applying deltas, transitioning mood, advancing world state, generating observations, and persisting cursor/revision/outbox together prevents partial or repeated state changes.

- **Tick p99 alert at five seconds:** The alert makes simulation lag observable before it breaks snapshot freshness.

- **Synthetic controlled drift schedules:** Synthetic schedules calibrate drift so it is measurable after about one week, perceptible after about three weeks, and not visible as a session-sized jump.

- **Presence-dominant drift weights:** Presence dominates because the experience is about regular presence rather than rewards; listen-in, offers and boldness effects are small and focused.

- **Settle effect:** Settle ends presence and quiets mood without trait direction, supporting a calm close rather than changing personality.

- **No negative drift on absence:** Absence must not subtract traits or create neglect pressure.

- **Persisted mood across disconnected clients:** Mood evolves while clients are disconnected so the aviary remains server-canonical and alive beyond an open tab.

- **Time-of-day and rare weather mood inputs:** Time and weather add ambience while the plan caps frequency and avoids distress or care obligation.

- **Compact call grammar and motif library:** Grammar, species motif families, deterministic seeds and fresh variation preserve identity while allowing mood and personality to affect timing, pitch and motif selection.

- **Local client audio synthesis:** The client synthesizes calls locally to avoid downloading or looping recordings.

- **Grammar-derived caption prose:** Caption text comes from the same call decision so captions match the rendered sound.

- **Constrained notebook prose:** Typed observations, templates and state facts keep entries specific, lowercase, present tense, sparse and supported by stored history.

- **Perch-zone choice from mood/personality and scene constraints:** Perches express bird state while keeping all birds within the scene composition.

- **Coherent starter species pool:** Selecting two species from a coherent pool gives new aviaries distinct but stable identities.

- **New bird availability by elapsed aviary age:** Age-only additions avoid rewards based on visits, interaction scores or payment and preserve the hard cap of seven.

### 6. Client rendering, session flow, and audio

- **Single responsive horizontal composition:** The horizontal composition keeps foreground, bird/perch plane and soft background together as one watchable scene.

- **All birds visible at all viewport sizes with no pan, zoom or scene scrolling:** Keeping birds visible supports the quiet session and avoids hiding birds or making the scene feel like a navigation task.

- **Thin top-bar interactive chrome:** Thin chrome keeps interaction available without crowding the scene.

- **Snapshot-to-progressing-pose first render:** Rendering directly from a snapshot into already-progressing poses makes the aviary feel alive on first paint and avoids entry animation or spinner framing.

- **Cold/offline quiet sky field and brief empty aviary:** This is the graceful slow-network or adoption state, explicitly "never a spinner."

- **Returning-user greeter:** One greeter based on boldness, mood and absence length lets "one bird notice the returning user" without textual welcome, absence-length copy or a chorus on cue.

- **Local interpolation and micro-motion:** Interpolation makes motion smooth while avoiding invented persistent outcomes.

- **Mood-shaped idle motion:** Wary, content, curious and drowsy poses make mood legible through behavior instead of raw state lists or numbers.

- **Client-only leaves, feathers and parallax:** These ornaments add scene life while staying outside simulation entities.

- **Local day/night and rare ambient rain/wind:** Local timezone and rare weather add ambience and lightly affect mood/calls without creating distress.

- **Settle transition with five-second undo:** Settle provides slow evening/call quieting and an immediate reversal path through any aviary click within five seconds.

- **Hidden-tab rendering stop and server ticks continue:** Hidden tabs stop client work while canonical simulation continues on the server.

- **Listen-in mix state:** Listen-in brings one bird forward while others remain audible, submits begin/end events, and disengages through clear clicks or focus departure.

- **Offer flow for seed, song fragment and still pool:** Offers are chosen in the top bar and sent to a receiving bird, with server cooldown and mood/curiosity-shaped reactions to preserve the gesture framing.

- **Bounded AudioContext and reusable audio resources:** Bounded audio resources, gain nodes, voice caps and cleanup prevent clipping, memory growth and runaway workers.

- **Browser audio permission handling:** If WebAudio is unavailable or denied, the aviary renders silently with captions enabled by default and never substitutes recordings.

### 7. Accessibility and product voice

- **Coherent live narration region:** The narration region gives screen-reader users one scene summary from the same snapshot, with idle cadence and prioritized events but no flood of duplicate speech.

- **No trait values or raw state lists in narration:** This preserves the hidden-trait product invariant and keeps narration naturalist rather than diagnostic.

- **Semantic named controls and useful scene summary:** Semantic controls, names and focus behavior make the scene operable without duplicate speech.

- **Explicit reduced-motion setting plus prefers-reduced-motion:** The explicit setting and media query honor user preference while continuing mood, calls and notebook behavior.

- **Caption preference and fallback default:** Captions are optional generally and default on when audio fallback is active, making silent rendering understandable.

- **Legible captions near the caller:** Near-caller placement connects caption prose to the sound source while WCAG AA contrast keeps it readable.

- **Keyboard order and shortcuts:** Top bar, scene entry, arrow movement, Enter listen-in, Escape disengage, offer flow and settle make the aviary usable by keyboard.

- **High-contrast visible focus:** Visible focus keeps keyboard location clear.

- **Naturalist and matter-of-fact copy split:** Scene/notebook prose carries naturalist voice while sign-in, settings, accessibility settings, sync and errors stay direct.

### 8. Security, privacy, sync, and lifecycle

- **Email encryption at rest and restricted decryption:** This limits email exposure to account and mail use.

- **Hashed magic-link, session and invite tokens:** Token hashing protects links and sessions if storage is exposed.

- **One-use magic links, revocable sessions and expiring invite tokens:** Token lifecycle limits account and visitor access over time.

- **Authentication and invitation rate limits:** Rate limits reduce abuse of sign-in and invitation sends.

- **Snapshot protection against cross-account IDOR and visitor escalation:** Authorization prevents one account or visitor capability from reading or mutating another aviary.

- **Visitor scene-only default:** Scene-only visitor access protects host controls and notebook unless explicitly required.

- **Server-canonical sync across devices:** Same-revision reads, ordered append-only writes, idempotency keys and transactional ticks prevent duplicate events, lost deltas and client last-write-wins.

- **Stale reload behavior without conflict dialogue:** Fetching a fresh snapshot and interpolating from the local frame keeps stale state recovery quiet unless the account/session cannot continue.

- **Retention schedule before launch:** Retention is required because event history should last only as long as needed for tick, notebook provenance and user operation.

- **Aggregate-only operational analytics:** Analytics can monitor request counts, latency, durations, frames and audio errors without account dimensions or state reconstruction.

- **Soft deletion and hard deletion:** A 30-day recoverable period protects accidental deletion, and hard deletion removes account-linked records and telemetry linkage.

- **On-demand export:** Export includes birds, names, vectors, moods, notebook and settings and is delivered through verified email for account-controlled access.

### 9. Performance, observability, and release gates

- **Initial JavaScript below 2 MB gzipped:** The bundle budget helps draw the first scene before noncritical assets.

- **First bird visible under 500 ms on mid-tier mobile over 4G:** The first-bird budget supports the invariant that the aviary is moving on first paint.

- **60 fps idle motion on older laptop and no 30-minute memory growth:** These requirements keep the everyday watching session smooth and stable.

- **Small first scene payload and cached minimum state:** Kilobyte-scale payloads, inline or edge-cached state, compact art and deferred noncritical UI support fast initial render.

- **Bounded resources:** Audio contexts, workers, voices, snapshot payloads and notebook DOM retention are bounded to protect performance and memory.

- **Scheduled synthetic browsers and aggregate-only RUM:** Synthetic and real-user measurements track load, first bird, frame timing, audio failures, API errors and tick latency without account-keyed telemetry.

- **Device/browser matrix and unsupported-browser explanation:** Browser coverage sets launch expectations, and unsupported older browsers receive a matter-of-fact explanation.

- **Acceptance checks as release gates:** Canonical invariants, concurrency, invites, deletion, caption/audio grammar, keyboard, narration, reduced motion, contrast, memory, frame rate, bundle and first-bird budgets must pass before launch.

### 10. Rollout and calibration

- **Internal foundations:** Schema, versioning, identity, immutable bird IDs, snapshot contract, event ingestion and server tick come first so continuity can be verified through sign-out/sign-in and two devices.

- **Engine calibration cohort:** A limited cohort and synthetic schedules let the team review aggregate drift calibration and latency without exposing trait values or creating per-account analytics.

- **Accessible beta:** Narration, reduced motion, captions, keyboard paths and audio fallback ship together so accessibility and performance are validated before traffic increases.

- **Progressive public launch:** Gradual account ramp, initial two-bird hold, age-based third-bird availability and opt-in social invites wait on stability, call recognizability, token scope, revocation, expiry and privacy checks.

- **Operational ownership:** On-call, aggregate telemetry, qualitative research or synthetic fixtures, parameter rollback and invitation disabling protect canonical correctness and privacy after launch.

### 11. Risks and mitigations

- **Versioned drift parameters and aggregate calibration:** These mitigate drift that is too fast or invisible while keeping per-session changes imperceptible and numbers hidden.

- **Three-signal presence qualification:** Visibility, focus and recent activity, plus server-validated pings and interval union, mitigate overcounted presence without raw input capture.

- **Server sequence, idempotent event keys and single-writer worker locking:** These mitigate lost or doubled drift during concurrent use, retries and worker crashes.

- **No distress states, negative movement or punitive weather:** These mitigate mood or weather feeling punitive and keep late return calm.

- **Distinct species motif families, chorus headroom and seven-bird limit:** These mitigate calls becoming synthetic or losing identity as bird count grows.

- **Narration cadence, event prioritization and screen-reader review:** These mitigate narration overwhelming or flattening the scene.

- **Edge-delivered snapshot, deferred UI and synthetic mobile monitoring:** These mitigate missing the first-bird budget.

- **Telemetry allowlists, visitor-scope audits and deletion-cascade tests:** These mitigate privacy leaks through observability or visits.

- **Sparse notebook, no counters and copy review against two voice registers:** These mitigate content becoming generic or gamified.
