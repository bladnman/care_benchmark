## System-level intent

1. The aviary should feel continuous, private, and canonical rather than like a repeated session. This shows up in the product contract: “one canonical aviary,” “feels as if it has continued between visits,” “stable identities,” and “multi-device sync.” It also appears in the service shape where the “relational canonical store” remains “authoritative,” in the tick design that “maintains mood and scene state across sessions,” and in release checks for “the same state version and drift.”

2. Presence is the main relationship signal, but it is quiet and non-punitive. The plan says to “treat presence as the main relationship signal,” to count it only under visible/focused/recent-activity conditions, and to make “presence-time” dominant in drift. It also repeatedly states that “absence is never a penalty,” “birds never die, become hungry, show distress,” and traits “only drift upward in the expressive direction.”

3. Personality is slow, private, and server-owned; mood is fast and observable. The plan names this core distinction directly at the end: “personality is slow and private; mood is fast and observable.” It is reinforced by “values never appear in user-facing APIs or downloads,” “only the server writes personality and canonical mood,” “no visible single-session shift,” and personality changes becoming perceptible only after “about three weeks.”

4. The client is an expressive renderer, not an authority. The service shape says the web client is “a renderer and event producer, never a simulation authority.” The API/event flow says clients “never upload absolute simulation state,” and the simulation and privacy sections say the server owns reactions, mood, drift, cooldowns, vectors, event ordering, and tick commits.

5. The product voice is specific, lowercase, naturalist, and matter-of-fact. The plan says the product surface uses “specific, lowercase naturalist prose,” while “sign-in, account, accessibility settings, and error surfaces use direct matter-of-fact copy.” It repeats this in accessibility: “Product copy stays lowercase, present-tense, and specific,” and errors should say “what happened and the next step without exposing internals.”

6. The experience should avoid engagement mechanics and user-scoring behavior. The plan forbids a “welcome toast, visit badge, streak, score, achievement, leaderboard, public feed, push campaign, or other engagement surface,” and later says birds are unlocked by “aviary age only,” “never by visit count, interaction volume, or payment.”

7. Privacy boundaries should be structural, not cosmetic. The plan requires encrypted email, synthetic UUIDs, no emails in logs or telemetry dimensions, no raw pointer coordinates or typed keys, aggregate-only telemetry, no bird/account identifiers in monitoring, no bird state in email, and no account/bird dimensions in dashboards or exports.

8. Accessibility is part of v1 quality, not a fallback or later scope. The delivery sequence says keyboard, narration, captions, reduced-motion, contrast, rendering, first paint, and memory profiling are “release requirements for the same v1, not follow-up scope.” The risks warn against accessibility becoming “a stripped fallback.”

9. The scene should be calm, ambient, and immediately alive. The rendering plan calls for a “responsive one-screen horizontal scene,” “calm natural palette,” “no spinner, wake-up animation, or fade from static,” and a first frame that “starts mid-motion.” Release checks require “an already-moving scene and a single varied bird greeting, without text welcome.”

## Per-feature whys

### 1. Product contract and scope

- Web-only, single-user aviary: The plan frames the product as “a web-only, single-user aviary” so each account owns “exactly one canonical aviary” that feels continuous between visits.

- One canonical aviary per account: Rationale is to preserve a single persistent place: “one canonical aviary,” “one account maps to one aviary,” and “multi-device reads against the same canonical state.”

- Two starter birds, growth to at most seven: The plan gives the rationale for growth limits as recognizability and performance: expand only after “behavior and audio remain recognizable,” and preserve “recognizable signatures and performance.”

- Age-based later birds: The plan explicitly rejects visit count, interaction volume, streaks, payment, and offer count, so availability is based on “aviary age only.”

- Stable bird identities and renameable names: Rationale is continuity: birds keep “stable identities,” and “rename preserves stable bird ID and all history.”

- Presence accounting: Rationale is that “presence is the main relationship signal” and should represent “honest, quiet attention.”

- No penalties for absence: Rationale is explicit: “absence is never a penalty,” traits never drift downward, and birds “never die, become hungry, show distress, or lose accumulated personality.”

- Bird greeting as return welcome: Rationale is to keep the greeting inside the natural aviary experience and avoid engagement surfaces; the plan says “the bird greeting is the return welcome” and forbids a separate “welcome toast” or related surfaces.

- Magic-link accounts: Rationale is partially articulated through account/security requirements: magic links are time-limited, single-use, rate-limited, and exchanged for “a revocable per-device session.”

- Multi-device sync: Rationale is to keep devices attached to “the same canonical state,” with server ordering and unioned presence intervals so concurrent devices do not double-count or diverge.

- Field notebook: Rationale is to provide “read-only observations” from “meaningful aviary moments and sparse periodic conditions,” describing “the aviary rather than praising or tracking the user.”

- Read-only visits by individually invited friends: Rationale is to let invitees see “the aviary scene only” without exposing settings, notebook, or simulation authority, and with revocation checked on every snapshot.

- Procedural calls: Rationale is to keep stable, recognizable bird identity without downloaded recordings: “stable per-bird signature,” “recognizable as mood and personality change,” and “do not use downloaded recordings.”

- Reduced-motion rendering: Rationale is accessibility without changing canonical simulation: reduced motion uses still poses, cross-fades, and removed drift while “mood, calls, notebook, and canonical simulation continue.”

- Screen-reader narration: Rationale is to expose the same scene state as “naturalist prose” without high-frequency state lists or personality values.

- Call captions: Rationale is to describe the “same runtime call grammar actually played,” support WebAudio failure, and align captions with calls.

- Export: Rationale is account data access through a verified-address expiring link, but with the stronger privacy invariant that exports omit numeric vectors and include observable state.

- Account/session settings: Rationale is to support verified email changes, accessibility preferences, visit-notice preference, timezone, and revocable device sessions.

- Account deletion: Rationale is recoverable deletion followed by privacy-preserving hard deletion: “allow sign-in and recovery for 30 days, then hard-delete.”

- Excluded native clients, payments, shared aviaries, multiple aviaries, editable/custom scenes, bird placement controls, public discovery, chat, visitor interaction, and gamification: Rationale is mostly scope and product-boundary preservation; the plan specifically ties exclusions to the single canonical aviary, no payment gates, no public or engagement surfaces, no user bird placement, and no visitor simulation effects.

- Export omitting numeric personality vectors: Rationale is a PRD conflict resolution: “Implement the stronger never-expose invariant.”

- Off-by-default visit notices: Rationale is a PRD conflict resolution: the general brief forbids notifications, so notices remain off by default and the “visit log is the only notification surface” until the opt-in delivery channel is confirmed.

### 2. Service shape and trust boundaries

- Small web client: Rationale is to serve the scene, capture signals, synthesize calls, render snapshots, and submit events while remaining “never a simulation authority.”

- Authenticated application API: Rationale is to centralize sign-in, settings, snapshot reads, event validation, notebook, export/deletion, and invitations while enforcing “owner versus visitor capabilities at every endpoint.”

- Simulation worker: Rationale is to run the slow server-owned tick, consume ordered events, update moods and server-only vectors, write notebook observations, and publish versions transactionally.

- Relational canonical store: Rationale is authoritative persistence with “a per-aviary serialization point and event cursor,” keeping canonical state in the database even if an outbox wakes workers.

- Email provider: Rationale is delivery of time-limited account, invitation, export, and verified-address links without including bird state in email.

- Operational telemetry: Rationale is health and performance monitoring while keeping analytics separate from the simulation database and without bird or account identifiers.

- Fast shell and initial state: Rationale is first-bird immediacy: “the first bird must render from the initial snapshot” without waiting for secondary surfaces.

- Private/no-store personalized snapshots: Rationale is privacy; personalized snapshots “must not enter a shared cache.”

### 3. Data model

- Synthetic UUIDs: Rationale is to avoid using email or other personal data as identifiers.

- Encrypted email at rest and never as identifier/log/partition key: Rationale is privacy and telemetry separation.

- Account record: Rationale is to hold verified email, pending change state, timezone, accessibility, visit-notice preference, and one-account-to-one-aviary mapping.

- Device session record: Rationale is revocable per-device sessions with labels that avoid unnecessary fingerprinting.

- Aviary record: Rationale is canonical state versioning, tick scheduling, event cursoring, local time, lighting/weather, and settled state.

- Bird record: Rationale is stable bird identity, observable render state, and hidden server-side personality; vectors never appear in user-facing APIs or downloads.

- Interaction event record: Rationale is ordered, idempotent, minimal event capture for simulation while excluding visitor activity.

- Notebook entry record: Rationale is read-only, sparse, newest-first naturalist observations.

- Invitation record: Rationale is one-time, expiring, revocable invite access with encrypted invitee email.

- Visit record: Rationale is an on-demand host log with approximate duration and minimum access metadata, without feeding visitor presence to simulation.

- Export/deletion job record: Rationale is expiring export delivery and delayed hard deletion after the recovery window.

- No raw pointer coordinates, typed keys, or general browsing activity: Rationale is to store only “eligibility and timing information needed to calculate an interval.”

- Unioned multiple-device presence: Rationale is so “two simultaneous tabs do not double-count presence-time.”

- On-demand account export: Rationale is user data access through verified email while omitting internal vectors under the never-expose invariant.

- Hard deletion after 30 days: Rationale is to remove account-linked birds, vectors, events, notebook, sessions, invitations, visit history, tied telemetry, and export artifacts after recovery.

### 4. API surface and event flow

- Versioned JSON endpoints over HTTPS: Rationale is stable API evolution and secure transport.

- Idempotency key on every mutation: Rationale is to return a resulting version or clear error without duplicate effects.

- Sign-in endpoint: Rationale is secure passwordless access with 15-minute single-use links, rate limits, revocable sessions, and verified email changes.

- Owner snapshot endpoint: Rationale is to give the client observable render state, transitions, and call plans while never returning personality values and keeping payloads small.

- Event batch endpoint: Rationale is to validate events and append them in order while preventing clients from applying trait deltas or submitting invalid bird/cooldown/order data.

- Notebook endpoint: Rationale is read-only pagination with “no mutation effect” and durable old entries.

- Bird/account settings endpoints: Rationale is narrow, field-specific mutation while preventing client edits to species, mood, perch, or vectors.

- Export/deletion endpoints: Rationale is verified-address export delivery and soft deletion with 30-day recovery before hard deletion.

- Invitation management endpoint: Rationale is named-email invites, outstanding invite lists, visit history, immediate revocation, one-time opaque redemption, and 30-day expiry.

- Visit redemption/read endpoint: Rationale is short-lived read-only visitor access, revocation on next pull, and no authorized visitor event endpoint.

- Owner snapshot pulling rules: Rationale is to recover from navigation, visibility restoration, long frame gaps, suspended devices, and low-frequency visible keepalive while interpolating instead of teleporting birds.

- Conditional requests and state versions: Rationale is to avoid redundant payloads.

- Server ordering for concurrent devices: Rationale is to append both devices’ events with idempotency while preventing clients from uploading absolute state.

- Visitor ambient snapshot: Rationale is to show birds, day/night, and weather only, excluding account settings and host notebook.

- Same unavailable surface for revoked/expired links: Rationale is clear matter-of-fact access handling without exposing extra internals.

### 5. Simulation engine

- Server-side tick around one-minute cadence: Rationale is slow canonical simulation that runs “with no connected client.”

- Per-aviary lease or row lock: Rationale is to serialize event consumption, state updates, version writes, cursor advancement, and atomic commit.

- Replay-safe retries: Rationale is that retrying “must not apply its deltas twice.”

- Delayed-worker handling: Rationale is to process elapsed time safely without inventing presence or replaying consumed interactions.

- Account IANA timezone for local time: Rationale is stable local day/night and daylight-saving handling without letting travel “oscillate the aviary’s clock.”

- Rare seeded rain and wind: Rationale is subtle ambient variety with “small mood effects.”

- Server tick continues while client hidden: Rationale is continuity across sessions and hidden rendering pauses.

- Throttled eligible heartbeat: Rationale is honest presence accounting only while visible, focused, and recently active.

- Stop heartbeats when any eligibility condition fails: Rationale is to prevent hidden, unfocused, or inactive tabs from contributing presence.

- Server-bounded intervals and gap caps: Rationale is so sleeping laptops cannot create hours of inferred presence.

- Listen-in: Rationale is to weight attention toward one bird’s social warmth and vocal frequency and to support audio focus without muting others.

- Offers: Rationale is to modestly inform boldness or curiosity depending on server-decided reaction outcomes.

- Settle: Rationale is to quiet mood and close presence, not drive personality drift.

- Nonnegative expressive drift deltas: Rationale is to prevent inactivity from reducing traits and keep absence non-punitive.

- Synthetic-history calibration: Rationale is to ensure no visible single-session shift, measurable week-level movement, and perceptible changes after about three weeks.

- Mood enum and transitions: Rationale is an observable fast state responding to interactions, local time, ambient events, and personality.

- Transition table or weighted state machine with hysteresis: Rationale is to avoid jitter and avoid “reset-to-neutral on page open.”

- Bird-to-bird calls, wary spread, chorus windows: Rationale is emergent aviary behavior from overlapping call schedules and moods.

- Versioned species pool: Rationale is coherent silhouettes, palettes, poses, and call motifs across about six species.

- System-assigned starter species with user names: Rationale is starter birds arrive by system assignment while the user can choose suggested or custom names.

- Later bird offers by age only: Rationale is to ramp from three toward seven only after behavior and audio remain recognizable, without gating on interaction or payment.

- Server-generated notebook observations: Rationale is to record meaningful aviary moments sparsely, not every session or visit frequency.

- Authored notebook templates/rules keyed to state transitions: Rationale is to avoid generic event logs, trait numbers, or user praise/tracking.

### 6. Client rendering pipeline

- Responsive one-screen horizontal scene: Rationale is a calm natural scene with three depth/perch zones that preserves aspect ratio and keeps birds visible on narrow and wide screens.

- Lightweight canvas renderer plus synchronized DOM layer: Rationale is visual rendering with keyboard focus, labels, and assistive technology support.

- Mood/personality-based perch choice, no user arrangement: Rationale is that birds choose perches from their state; users cannot arrange them.

- Snapshot timestamp initialization: Rationale is for the first frame to start “mid-motion.”

- No spinner, wake-up animation, or static fade: Rationale is to avoid breaking the sense of continuity and immediate life.

- Quiet field before snapshot and soft fly-in after first adoption: Rationale is to keep the field calm while unavailable and introduce the two birds softly.

- No in-scene buttons, badges, hover tooltips, or labels: Rationale is to keep the scene itself natural and uncluttered.

- Thin top bar for account/settings/accessibility/notebook/offer controls: Rationale is to place controls outside the scene while keeping them available.

- Top bar fade on cursor stillness: Rationale is to reduce visual intrusion and restore controls on pointer or keyboard activity.

- Local interpolation, idle pose variation, parallax, leaf/feather drift: Rationale is low-cost visual life between authoritative states; ornaments are not simulation events.

- Seeded or bounded ornament randomness: Rationale is to prevent layout shifts and keep the render loop stable.

- Day/night colors from account timezone: Rationale is consistency with server-local time.

- Subtle rain/wind rendering: Rationale is to reflect ambient events and mood subtly.

- Settle lighting/call ramp and five-second undo: Rationale is to quiet the aviary slowly and allow reversal by any click during the undo window.

- Screen-reader live region: Rationale is naturalist prose from the same snapshot, updated at calm intervals and promptly for notable events, without raw state lists or personality values.

- Keyboard navigation: Rationale is full keyboard operability for top-bar controls, bird focus, listen-in, offer flow, and settle, with visible focus indicators.

- Reduced motion mode: Rationale is to honor preferences while keeping mood, calls, notebook, and canonical simulation active.

- Call captions near the bird: Rationale is to describe the actual call grammar, align with the calling bird, and provide a graceful silent mode when WebAudio is unavailable.

- Contrast and copy review: Rationale is WCAG AA compliance and consistent lowercase, present-tense, specific or matter-of-fact copy.

### 7. Procedural audio pipeline

- Motif grammar per species and stable per-bird signature: Rationale is recognizable calls that can vary with mood and personality.

- Server-supplied call timing and mood/scene cues: Rationale is to shape calls from server-derived plans without exposing raw personality vectors.

- Client synthesis from identity seed, mood, and variation: Rationale is procedural audio with stable identity and per-call variety.

- WebAudio oscillators/envelopes and bounded buffers: Rationale is reusable, bounded allocation over long sessions.

- Chorus mixing: Rationale is simultaneous calls with “slight natural timing variation” and clarity.

- Listen-in audio ramp: Rationale is to raise one bird while lowering others to ambient, but “other birds never go silent.”

- Listen-in reversal triggers: Rationale is to return the mix to normal when focus changes, empty space is clicked, or keyboard focus moves away.

- Song-fragment offer motif: NOT RECOVERABLE FROM PLAN

- Settle lowers call activity gradually: Rationale is to make settle a slow quieting rather than an abrupt stop.

- No downloaded recordings: Rationale is to rely on procedural calls rather than substituted recorded audio.

- AudioContext on first user gesture: Rationale is browser autoplay compliance without blocking initial visual rendering.

- Silent captions fallback when WebAudio unsupported or denied: Rationale is graceful silence and captions by default rather than recorded substitutes.

- Audio tests over 30 minutes: Rationale is recognizability, ramp timing, chorus clarity, caption alignment, and bounded allocation.

### 8. Privacy, security, and sync rules

- Server-only personality and canonical mood writes: Rationale is to prevent stale clients from overwriting vectors or canonical simulation.

- Per-aviary ordering boundary: Rationale is ordered event deltas and consistent canonical state.

- Idempotency keys, event cursor, state version, transactional tick commit: Rationale is replay-safe sync and no duplicated effects.

- Row-level authorization: Rationale is owner-only APIs and read-only active-invite visitor capability.

- Revocation recheck on visitor snapshots: Rationale is immediate next-pull enforcement.

- Single-use, expiring, rate-limited magic links with token hashes: Rationale is account security.

- Verified replacement email: Rationale is to prevent switching addresses before proof of control.

- Matter-of-fact errors: Rationale is to say what happened and the next step without exposing internals.

- Event data only for aviary simulation/account experience: Rationale is to forbid model training, recommendation, population behavior analysis, third parties, and analytics drift.

- Aggregate-only telemetry and separated RUM/synthetic pipelines: Rationale is to keep monitoring from accessing simulation data or account/bird dimensions.

- On-demand visit log: Rationale is settings-based host visibility into visitor email, approximate time/duration, and outstanding invites without a badge.

### 9. Performance, observability, and rollout

- Initial JavaScript under 2 MB gzipped: Rationale is first-load performance; secondary account, settings, notebook, and invite flows are split out.

- First bird visible under 500 ms: Rationale is the affective target of seeing the bird quickly; measure “the bird’s first painted frame.”

- Idle motion at 60 fps: Rationale is smooth ambient motion on older hardware.

- No memory growth over 30 minutes: Rationale is long-session stability through reused buffers, bounded contexts, and released notebook references.

- Tick p99 alert over five seconds: Rationale is operational visibility into delayed canonical simulation.

- Last two major browsers and unsupported-browser surface: Rationale is defined browser support and clear handling for older clients.

- Scheduled synthetic browsers: Rationale is geographic checks for first-bird, load, and tick health.

- Aggregate RUM: Rationale is page load, first-bird render, frame timing, audio-context errors, and tick latency monitoring without account, email, bird, snapshot, or per-account histories.

- Foundation delivery sequence: Rationale is to validate security, canonical records, event append/idempotency, tick worker, snapshot versioning, and multi-device canonical reads before polishing.

- Affective core delivery sequence: Rationale is to tune presence, drift, moods, greetings, bird-to-bird behavior, notebook sparsity, and call grammars against scripted timelines with reversible server-versioned parameters.

- Accessibility and performance delivery sequence: Rationale is these are v1 release requirements, not follow-up scope.

- Private beta: Rationale is to validate tick recovery, sync, and call identity with a small operational cohort and synthetic scenarios while collecting only approved aggregate telemetry.

- General availability and bird ramp: Rationale is to launch with two birds, then increase age-gated bird availability only after chorus recognizability and performance remain within budget.

- Visit rollout: Rationale is to keep visits off until deliberate host invites and launch read-only access, revocation, expiry, host log, and off-by-default notice preference together.

### 10. Release checks and principal risks

- Already-moving return scene with one varied bird greeting: Rationale is to validate continuity and avoid a text welcome.

- Two-device same state and unioned presence: Rationale is canonical sync and no double-counting of overlapping sessions.

- Hidden/unfocused/inactive tabs and heartbeat gaps do not inflate presence: Rationale is honest presence accounting.

- Idempotent offers and listen-in with server-owned reactions: Rationale is authoritative mood, drift, and cooldown handling.

- Personality values never appear in UI, snapshots, logs, RUM, or exports: Rationale is the privacy invariant.

- Visitor view-only access with no simulation effect: Rationale is to keep visitor activity from affecting host drift and enforce revocation.

- Reduced-motion, narration, captions, keyboard, contrast, bundle, first-bird, fps, and memory gates: Rationale is launch-quality accessibility and performance.

- Account recovery, session revocation, email changes, export, deletion: Rationale is end-to-end account lifecycle correctness.

- Drift risk mitigation: Rationale is to calibrate low-pass coefficients and heartbeat caps with fixtures, union intervals, and monotonic bounds.

- Sync risk mitigation: Rationale is to prevent divergent versions, repeated event effects, cursor gaps, and stale-client writes through server-only writes, idempotency, transactions/leases, atomic cursor updates, and replay-safe ticks.

- Call-identity risk mitigation: Rationale is to preserve species/bird recognizability as moods vary and bird count rises.

- Accessibility fallback risk mitigation: Rationale is to build accessible modes alongside the primary renderer and share state/grammar between prose and call plans.

- First-paint/performance risk mitigation: Rationale is to preserve the affective target through small bundles, deferred secondary surfaces, immediate snapshot drawing, reused audio resources, and low-end/long-session profiling.

- Privacy-boundary risk mitigation: Rationale is to prevent account/bird dimensions or numeric fields from appearing in analytics, logs, dashboards, APIs, or exports.

- PRD-contradiction risk mitigation: Rationale is to avoid accidental trait exposure or unwanted notices by applying section 1 decisions, keeping notices off by default, and resolving the delivery channel before opt-in.
