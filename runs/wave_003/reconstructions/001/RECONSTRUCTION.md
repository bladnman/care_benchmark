## System-level intent

- **Observational relationship rather than game or custody.** The executive summary says Pocket Aviary is "an observational relationship rather than a gamified app or a custodial pet simulator." This shows up again in the non-goals: "No Gamification," "No Tamagotchi / Custodial Mechanics," and no "penalty for neglect."

- **Feels alive, not robotic.** The first design tenet requires that the aviary "must never present a cold start or frozen state." The same intent appears in the "Motion Already in Progress" contract, where frame 1 renders sky, birds in a mid-idle cycle, and leaf particles seeded mid-fall.

- **Notice, never announce.** The plan rejects "arrival toasts, level-up banners, badges, or streak celebrations" and says welcoming is expressed through "natural bird behavior" such as "glances, head-tilts, staggered greetings." This carries through return-greeting, quiet visit invitations, and the no-push-notifications guardrail.

- **Charm from specificity.** The plan wants "sparse, naturalist field-notebook entries generated from granular behavioral observations rather than templated telemetry counters." The same principle appears in call captions that reflect the "exact synthesized motif" and screen-reader narration written as naturalist prose.

- **Restraint over richness.** The plan explicitly names "Restraint Over Richness": a "fixed single horizontal viewport," "three perch depths," "2 to 7 birds maximum," and "subtle ambient weather." The non-goals also enforce restraint by excluding social network surfaces, native apps, subscriptions, scores, streaks, and reminders.

- **Dual-register voice discipline.** The plan splits product and system surfaces into "Naturalist voice" and "Matter-of-fact voice." Aviary, notebook, captions, and screen-reader narration use "lowercase, present-tense, observational" prose; sign-in, account settings, sync conflicts, errors, and accessibility controls use "Capitalized, direct, plain English."

- **Server authority with client interpolation.** The architecture "separates high-frequency rendering and audio synthesis on the client from authoritative simulation and data ownership on the server." The simulation engine is "strictly server-side," the server tick is the "only writer" of vectors and moods, and the client acts as an "interpolator and reactive consumer of snapshots."

- **Monotonic expressiveness and non-punitive absence.** The drift invariant says traits "never decrease due to absence or inactivity." The custodial non-goal says birds become "quiet and ambient during absence, never mistrustful," with no bird death, hunger, illness, visible distress, or neglect penalty.

- **Quiet optional social, not a social network.** Social is labeled "Optional & Quiet" and limited to "one-to-one, read-only visit invitations." The plan forbids co-presence, chat, visitor avatars, host presence drift, public directory, discovery feed, profiles, following, comments, and leaderboards.

- **Strict privacy boundary.** The plan uses "synthetic account UUIDs decoupled from encrypted email addresses," self-service export, soft deletion, hard deletion, and "zero cross-account aggregation of bird interaction telemetry." The telemetry section reinforces this with "pipeline segregation" and no analytics access to interaction events or personality vectors.

- **Accessibility as an intentional aesthetic experience.** The accessibility section says accessibility is "an intentional aesthetic experience rather than an afterthought." Reduced motion uses "poetic cross-fades," narration stays in naturalist voice, and captions are generated from the same procedural call descriptions as the audio.

- **Performance and calibration are part of product feel.** The plan defines hard budgets for "Time to First Bird," framerate, bundle size, leak growth, and tick latency. The risk section frames drift calibration as avoiding either "Tamagotchi feel" or "screensaver feel," using 1-week instrument and 3-week visible thresholds.

## Per-feature whys

### Executive Summary & Design Tenets

- **Pocket Aviary as a lightweight, web-only ambient virtual aviary:** The plan frames the whole product as "lightweight" and "web-only" so the aviary remains ambient rather than a native app or feature-rich simulator.

- **Two starter birds scaling up to seven over months:** The why is restraint: the bird population supports "2 to 7 birds maximum" and age-based growth, keeping the scene a small aviary rather than a collection game.

- **Single horizontal scene:** The why is "Restraint Over Richness"; the aviary lives in a "fixed single horizontal viewport" rather than scrolling, zooming, or panning through a larger world.

- **No cold start or frozen state:** The plan says this is required for "Feels Alive, Not Robotic." The first frame must show motion and ambient sound already in progress.

- **Procedural call generation:** The plan says it "prevents repetitive audio loops," supporting the alive, non-robotic feel.

- **No arrival toasts, level-up banners, badges, or streak celebrations:** The why is "Notice, Never Announce"; welcome and progress are expressed only through bird behavior.

- **Natural bird greetings through glances, head-tilts, and staggered greetings:** The rationale is that the app should notice the user through "natural bird behavior," not system announcements.

- **Field-notebook entries from granular behavioral observations:** The rationale is "Charm from Specificity," with sparse naturalist writing instead of "templated telemetry counters."

- **Three perch depths, 2 to 7 birds, and subtle ambient weather:** The rationale is "Restraint Over Richness," keeping the scene specific and quiet.

- **Naturalist voice:** The plan assigns it to product surfaces because those surfaces should be "lowercase, present-tense, observational."

- **Matter-of-fact voice:** The plan assigns it to system surfaces because those surfaces should be "Capitalized, direct, plain English."

### Scope & Non-Goals

- **Modern desktop and mobile web browsers:** The plan's rationale is "web standards only" and "No Native Apps"; v1 is browser-based rather than wrapped as iOS or Android.

- **Fixed cap of 7 birds:** The cap is part of "Restraint Over Richness" and protects the aviary from becoming a high-volume collection surface.

- **Exactly 2 starter birds from a pool of 6 species:** The plan ties this to new aviaries starting with two birds and the overall 2 to 7 cap.

- **System-assigned starter bird selection:** NOT RECOVERABLE FROM PLAN.

- **Birds 3 through 7 unlocked exclusively by aviary age milestones:** The rationale is anti-gamification: arrivals are "never interaction counts or streaks."

- **User-defined naming and renaming at any time:** NOT RECOVERABLE FROM PLAN.

- **Stable UUID bird identities:** The rationale is explicit: identities "persist through renames, migrations, and syncs."

- **Passive presence monitored through tab visibility, window focus, and recent input activity:** The plan uses strict tripartite validation so presence pings can be summed as "validated" presence minutes rather than any open tab counting as presence.

- **Return-greeting within 1-2 seconds of session start:** The rationale is "Notice, Never Announce"; the welcome is procedural bird reaction, not a toast or banner.

- **Return-greeting staggered across birds and modulated by absence duration and boldness:** The rationale is natural bird behavior and specificity rather than robotic simultaneity.

- **Listen-in:** The plan says it elevates one bird's mix level while attenuating others "without hard cuts"; it creates focus while preserving the ambient aviary.

- **Offer control:** The plan gives offers as quiet interactions that can trigger curiosity and drift.

- **Choice of seed, song fragment, or still pool as offer kinds:** NOT RECOVERABLE FROM PLAN.

- **Per-bird offer cooldowns of a few minutes:** NOT RECOVERABLE FROM PLAN.

- **Settle gesture:** The plan defines it as a "soft session-end gesture" that shifts lighting to evening and quiets calls, aligning session end with the aviary's natural behavior.

- **5-second undo affordance for settle:** NOT RECOVERABLE FROM PLAN.

- **Sparse, read-only Field Notebook observations every 2-4 days:** The rationale is "Charm from Specificity" and sparse naturalist observation, not counters or user-authored status.

- **One-to-one, read-only visit invitations by email magic link:** The rationale is "Optional & Quiet" social sharing without co-presence, chat, visitor avatars, or host presence drift.

- **Revocable visits:** The rationale is quiet social sharing that the host can revoke at any time.

- **30-day visit expiration:** NOT RECOVERABLE FROM PLAN.

- **Single-user accounts authenticated by email magic links:** The plan uses magic links for sign-in.

- **15-minute magic-link expiration:** NOT RECOVERABLE FROM PLAN.

- **Per-device revocable sessions:** The plan includes them under "Sync & Accounts" so sessions can be individually revoked.

- **Canonical server-side simulation tick at about 60 seconds:** The rationale is canonical server simulation and multi-device consistency, with clients pulling snapshots rather than authoritatively simulating.

- **Append-only event architecture:** The rationale is that devices communicate interactions as "immutable, timestamped event records," while the server remains the single canonical writer.

- **Multi-device snapshot pulling and event submission:** The rationale is consistency: all devices pull identical snapshots and submit interaction events for the tick to process.

- **Synthetic account UUIDs decoupled from encrypted email addresses:** The rationale is the privacy boundary and avoiding emails as foreign keys, partition keys, or logging identifiers.

- **Self-service JSON state export:** The plan places export under "Privacy Controls."

- **Soft deletion with full hard deletion thereafter:** The rationale is account lifecycle control under "Privacy Controls."

- **30-day account soft-deletion period:** NOT RECOVERABLE FROM PLAN.

- **Zero cross-account aggregation of bird interaction telemetry:** The rationale is the "strict privacy commitment."

- **Continuous screen-reader live narration:** The rationale is accessibility in naturalist voice, using polite narration as part of the aviary surface.

- **Call captioning:** The rationale is accessibility and silent fallback, with captions matching procedural call descriptions.

- **WCAG AA contrast, keyboard navigation, and high-contrast outlines:** The rationale is accessible operation across visual palettes.

- **Reduced-motion mode with poetic cross-fades:** The rationale is a "distinct reduced-motion mode" that replaces animated skeletal motion without losing the aesthetic experience.

- **Real-time client-side procedural WebAudio synthesis:** The rationale is procedural, non-repetitive bird sound without networked loops or samples.

- **Silent fallback with automatic captions when WebAudio is unavailable:** The rationale is graceful operation when audio is blocked or unsupported, without recorded fallback.

- **No native apps:** The rationale is "Web standards only."

- **No gamification:** The rationale is that the aviary is observational, with no achievements, badges, scores, levels, streaks, visit calendars, green dots, or visit-frequency metrics.

- **No Tamagotchi or custodial mechanics:** The rationale is non-punitive absence: no death, hunger, illness, distress, neglect penalty, or mistrust.

- **No social network surfaces:** The rationale is quiet social behavior, not public discovery, profiles, following, comments, or leaderboards.

- **No monetization or subscriptions:** NOT RECOVERABLE FROM PLAN.

- **No push notifications, reminders, or unprompted emails:** The rationale is no unprompted re-engagement and "Notice, Never Announce."

### System Architecture & Service Topology

- **Client-side high-frequency rendering and audio synthesis:** The rationale is to keep rendering/audio responsive in the browser while server-side systems own authoritative simulation and data.

- **Server-side authoritative simulation and data ownership:** The rationale is canonical drift and mood state, conflict prevention, and consistent snapshots across devices.

- **Edge caching of static shell assets:** The rationale is fast delivery of the lightweight shell under the bundle and time-to-first-bird budgets.

- **Edge worker injecting the initial snapshot bootstrap:** The rationale is the "<500ms time-to-first-bird requirement" and the no-cold-start contract.

- **Stateless REST/HTTP API service:** NOT RECOVERABLE FROM PLAN.

- **Magic-link workflows in the API service:** The rationale is account authentication through matter-of-fact system surfaces.

- **Batched interaction event ingestion:** The rationale is append-only event submission rather than clients writing personality or mood state directly.

- **Visit token authorization:** The rationale is read-only visit access without requiring an auth header for visitors.

- **Simulation tick worker:** The rationale is to consume raw interaction events, update personality vectors, transition moods, advance daylight cycles, and persist canonical snapshots.

- **Drift low-pass filtering:** The rationale is monotonic additive change calibrated over time rather than abrupt, game-like jumps.

- **Sparse notebook entry engine:** The rationale is naturalist observations generated from behavioral specificity at a sparse rhythm.

- **PostgreSQL primary database:** NOT RECOVERABLE FROM PLAN.

- **Redis store:** The rationale is distributed tick locking, latest snapshot caching, in-flight presence pings, and ephemeral rate limits.

- **Client application as lightweight SPA using TypeScript and 2D Canvas:** The rationale is a lightweight client that interpolates snapshots and synthesizes audio locally.

- **Client never authoritatively calculates personality drift or mood changes:** The rationale is server authority and conflict prevention.

### Data Model & Storage Schema

- **Synthetic UUID primary keys for all relational entities:** The rationale is privacy and stable identities decoupled from emails.

- **Encrypted email addresses with blind-index lookup:** The rationale is that emails are "encrypted at rest" and never used as foreign keys, partition keys, or logging identifiers while still supporting lookup.

- **Accounts table with timezone:** The timezone supports daylight and dusk evaluation from the user's timezone.

- **Accounts deletion status and deletion requested timestamp:** The rationale is 30-day soft deletion and restore/hard-deletion lifecycle.

- **Visit notification enabled flag:** NOT RECOVERABLE FROM PLAN.

- **Magic links table:** The rationale is token generation, expiry, and one-time consumption for email sign-in.

- **Device sessions table:** The rationale is per-device revocable sessions and last-active tracking.

- **Aviaries table as one-to-one with account at v1:** NOT RECOVERABLE FROM PLAN.

- **Weather state and weather expiry on aviaries:** The rationale is subtle ambient weather and weather/daylight evaluation.

- **Bird records with species, name, perch zone, mood, and adoption time:** The rationale is stable bird identity, perch/depth rendering, mood continuity, and age/unlock history.

- **Personality vectors:** The rationale is server-authoritative, bounded trait drift for boldness, social warmth, vocal frequency, plumage saturation, and curiosity.

- **Append-only interaction events:** The rationale is immutable event submission from devices and simulation-worker consumption.

- **Target bird on interaction events:** The rationale is per-bird reactions, offers, listen-in, and drift signals.

- **Field notebook entries:** The rationale is sparse, read-only naturalist prose tied to noteworthy behavior.

- **Visit invitations:** The rationale is quiet, revocable, expiring, read-only sharing.

- **Visit logs as read-only observation history:** NOT RECOVERABLE FROM PLAN.

### API Surface & Protocols

- **REST semantics over HTTPS with JSON:** NOT RECOVERABLE FROM PLAN.

- **Matter-of-fact error responses on system surfaces:** The rationale is dual-register voice discipline for auth, conflicts, errors, settings, and accessibility controls.

- **POST `/api/v1/auth/magic-link`:** The rationale is email magic-link sign-in.

- **Generic magic-link response message:** NOT RECOVERABLE FROM PLAN.

- **Rate limit of 5 magic-link requests per hour per email:** NOT RECOVERABLE FROM PLAN.

- **POST `/api/v1/auth/magic-link/consume`:** The rationale is exchanging the magic-link token for a session token and account ID.

- **DELETE `/api/v1/auth/sessions/:id`:** The rationale is revoking a target device session.

- **GET `/api/v1/aviary/snapshot`:** The rationale is snapshot pulling for consistent multi-device state and immediate client rendering.

- **Snapshot fields for daylight, weather, settled state, birds, moods, saturation, and cooldowns:** The rationale is for the client to render and interpolate from canonical server state.

- **POST `/api/v1/aviary/events`:** The rationale is appending presence, offer, listen-in, settle, and undo events rather than client-side state mutation.

- **GET `/api/v1/notebook`:** The rationale is read-only retrieval of naturalist prose entries ordered newest to oldest.

- **POST and DELETE visit invitations:** The rationale is quiet email-link sharing that can be revoked immediately.

- **GET visit snapshot by token:** The rationale is visitor read-only access to the same visual/audio aviary snapshot.

- **Rejecting visitor event posts and not incrementing host presence from visits:** The rationale is no host presence drift from visits and read-only social.

- **PATCH bird name:** The rationale is renaming support while stable UUID identity persists.

- **GET account export:** The rationale is self-service JSON state export.

- **DELETE account and POST restore:** The rationale is 30-day soft deletion with restore before hard deletion.

### Simulation Engine & Drift Dynamics

- **Server-side-only simulation:** The rationale is that clients "never write vectors, simulate mood transitions, or compute drift."

- **Tick cadence once every 60 seconds per aviary:** NOT RECOVERABLE FROM PLAN.

- **Distributed Redis lock with 10-second TTL:** The rationale is "to prevent concurrent ticks across partitions."

- **Event ingestion since `last_tick_at`:** The rationale is that the tick consumes raw interaction events before updating state.

- **Presence validation as fractional presence minutes:** The rationale is converting only tripartite-valid pings into drift inputs.

- **Personality drift step:** The rationale is monotonic expressive growth from accumulated behavioral signals.

- **Mood transition step:** The rationale is fast-timescale reaction to time of day, interactions, weather, and bird-to-bird contagion.

- **Perch positioning from boldness and mood:** The rationale is to make drift and mood visible through perch choice.

- **Weather and daylight evaluation from account timezone:** The rationale is local sun elevation, dusk, and subtle ambient weather.

- **Notebook evaluation in the tick:** The rationale is sparse naturalist observation generation from server-known behavior.

- **Snapshot materialization in Redis:** The rationale is high-speed client fetches.

- **Monotonic expressiveness invariant:** The rationale is that traits never decrease because of absence or inactivity.

- **Boldness driven by presence and nearby offers:** The rationale is recoverable only as the plan's stated weighted input mapping.

- **Social warmth driven by presence and listen-in duration:** The rationale is recoverable only as the plan's stated weighted input mapping.

- **Vocal frequency driven by listen-in and ambient presence:** The rationale is recoverable only as the plan's stated weighted input mapping.

- **Plumage saturation driven strictly by cumulative presence-time:** The rationale is recoverable only as the plan's stated weighted input mapping.

- **Curiosity driven by accepted offers and song fragments:** The rationale is recoverable only as the plan's stated weighted input mapping.

- **Week 1 instrument threshold and Week 3 user-visible threshold:** The rationale is calibration: drift should be detectable by telemetry first and visible later through perch choice, greeting eagerness, and saturation.

- **Mood state machine with wary, content, curious, drowsy, and alert:** The rationale is fast-timescale behavior distinct from slow personality drift.

- **Wary to content after undisturbed presence or calm weather:** The rationale is natural softening, with high-boldness birds softening faster.

- **Content to drowsy at dusk or settle:** The rationale is time-of-day and session-end quieting.

- **Content to curious on offer:** The rationale is immediate bird interest in seed, song, or pool.

- **Alert to wary after neighboring wariness or abrupt wind:** The rationale is alarm contagion and weather response.

- **Mood continuity across tab open:** The rationale is that mood "does not snap to neutral"; the server preserves the most recent computed state.

- **Bird-to-bird vocal response based on social warmth:** The rationale is social chorusing: a warm bird may answer within 1.5-3.0 seconds.

- **Bird-to-bird alert contagion on same perch zone:** The rationale is adjacent birds reacting when one enters `wary`.

- **Aviary aging unlock schedule:** The rationale is that arrivals are governed by creation timestamp, "never interaction counts or streaks."

### Multi-Device Sync & Conflict Prevention

- **Single canonical writer:** The rationale is that only the server writes personality vectors and mood states.

- **Clients submit immutable timestamped event records:** The rationale is conflict prevention; devices never propose vector modifications.

- **Conflict impossibility through event summing:** The rationale is eliminating Last-Write-Wins overwrites and summing valid presence across sessions without duplicate double-counting in a 60-second window.

- **Identical snapshots for all devices:** The rationale is multi-device consistency.

- **Snapshot polling every 60 seconds or immediately on tab focus:** NOT RECOVERABLE FROM PLAN.

- **2.5-second curved bezier perch interpolation:** The rationale is "to prevent visual teleportation."

- **No blocking dialogs during in-flight network drop:** The rationale is continuous idle micro-motion from current local state.

- **Matter-of-fact invalid-session notification:** The rationale is voice discipline for sync/session errors.

### Frontend Rendering Pipeline

- **Custom HTML5 2D Canvas engine in vanilla TypeScript:** The rationale is "zero heavy frameworks" and a small bundle.

- **Bundle cost below 60KB uncompressed for the canvas engine:** The rationale is lightweight rendering and performance.

- **16:9 responsive aspect ratio with letterboxing/pillarboxing:** The rationale is the single horizontal viewport contract.

- **No scrolling, zooming, or panning:** The rationale is the fixed-scene restraint of the aviary.

- **Layered sky, foliage, perch rails, particulates, and top-bar DOM chrome:** The rationale is a single scene with depth from back to front and semantic HTML controls above the canvas.

- **Procedural sky gradient shifting by time of day:** The rationale is local daylight and atmosphere inside the fixed scene.

- **Subtle 0.02x foliage parallax:** NOT RECOVERABLE FROM PLAN.

- **Ambient leaf and feather drifts:** The plan calls them "purely ornamental."

- **Initial snapshot in inline bootstrap script:** The rationale is immediate frame-1 rendering and the time-to-first-bird budget.

- **No blank canvas or loading spinner:** The rationale is "Motion Already in Progress" and no cold/frozen state.

- **Birds seeded in mid-idle cycle:** The rationale is that motion is already in progress rather than starting mechanically at load.

- **Cold-load fallback quiet morning sky and floating leaf:** The rationale is preserving peaceful visual continuity while fetching snapshot data under a sub-500ms budget.

- **Procedural idle micro-motion using composite trigonometric oscillations rather than sprite flipbooks:** The rationale is alive, varied motion without heavy assets.

- **Breathing, head tilting, weight shuffle, preening, and fluffed feathers:** The rationale is mood-specific and continuous natural bird behavior.

- **Reduced-motion mode disabling frame-by-frame continuous motion:** The rationale is accessibility for `prefers-reduced-motion`.

- **Reduced-motion cross-fades between static poses and opacity transitions for perch moves:** The rationale is a "poetic" reduced-motion experience rather than animated skeletal motion.

- **Day/night shifts remaining in reduced motion over 10 seconds:** The rationale is preserving ambient time-of-day change while reducing motion intensity.

### Procedural Audio Pipeline

- **Real-time WebAudio synthesis with zero pre-recorded loops or samples:** The rationale is procedural freshness, bundle restraint, and avoiding repetitive loops.

- **Per-bird synthesizer voice with oscillator, filter, envelope, panner, and gain:** The rationale is spatialized bird voices tied to perch X/Z location and listen-in ramps.

- **Subtle reverb and master dynamics compressor:** The rationale is aviary space and clipping prevention when multiple birds call.

- **Distinct motif grammar for each of 6 species:** The rationale is species-specific voice and "Charm from Specificity."

- **Pitch, duration, and pause jitter on each call trigger:** The rationale is that a call remains identifiable while "never repeating bit-identically."

- **400ms micro-stagger between call onsets:** The rationale is preventing "synthetic synchrony and phase cancellation."

- **Return-greeting call queue with natural secondary delay:** The rationale is staggered, natural response rather than robotic chorus.

- **Listen-in gain ramps:** The rationale is smooth focus on one bird without hard cuts.

- **Never muting other birds during listen-in:** The rationale is "preserving the natural spatial depth of the aviary."

- **Silent fallback when WebAudio is blocked, fails, or is unsupported:** The rationale is graceful operation without error.

- **No recorded audio fallback:** The rationale is honoring "bundle and anti-canniness constraints."

- **Automatic call captions during silent fallback:** The rationale is preserving procedural call information when sound is unavailable.

### Accessibility Surfaces

- **Hidden `aria-live` status narrator:** The rationale is screen-reader narration that participates in the same naturalist register as the aviary.

- **Lowercase, present-tense narration:** The rationale is naturalist voice discipline.

- **Sparse background narration every 45-60 seconds:** The rationale is a quiet cadence rather than constant speech.

- **High-priority narration for return-greeting, accepted offer, and settle:** The rationale is immediate prose for important aviary events.

- **Overwriting ARIA text instead of appending:** The rationale is eliminating ARIA queue buildup.

- **Floating call captions near the vocalizing bird:** The rationale is locating procedural call descriptions in the visual scene.

- **Captions reflecting exact synthesized motif:** The rationale is keeping captions aligned with the generated audio grammar.

- **Caption fade timing:** NOT RECOVERABLE FROM PLAN.

- **Full standard keyboard controls:** The rationale is accessible navigation for top-bar affordances, birds, listen-in, offer, settle, and modal escape.

- **High-contrast focus ring with outer contrast halo:** The rationale is at least 4.5:1 contrast against dawn/noon and dusk/night palettes.

- **WCAG AA contrast for UI text, icons, modals, and captions:** The rationale is accessibility compliance.

### Performance Budgets, Telemetry & Privacy Boundary

- **Initial JS bundle under 2.0 MB gzipped:** The rationale is enforced lightweight delivery, with "zero heavy runtime frameworks."

- **Time to First Bird under 500 ms on 4G mobile:** The rationale is no cold start, using inline bootstrap snapshot and no blocking CSS or web fonts before canvas draw.

- **Idle framerate of 60 fps on a 5-year-old laptop:** The rationale is smooth ambient life through lightweight 2D canvas routines and offscreen perch caching.

- **Zero memory growth over 30 minutes:** The rationale is long ambient sessions without leaks, enforced through object pooling and CI Puppeteer leak tests.

- **Tick latency p99 under 5 seconds:** The rationale is timely simulation worker output, enforced by backend queue alerting.

- **Allowed aggregate operational telemetry:** The rationale is operational measurement without account-associated bird telemetry.

- **Prohibited per-account or per-bird telemetry:** The rationale is the strict privacy commitment and no analytics access to interaction events or personality vectors.

- **Telemetry pipeline segregation:** The rationale is enforcing the privacy boundary through permissions, not only policy.

### Rollout & Calibration Strategy

- **Milestone 1 audio and engine testbed:** NOT RECOVERABLE FROM PLAN.

- **Milestone 2 canvas rendering and interaction core:** NOT RECOVERABLE FROM PLAN.

- **Milestone 3 auth, sync, and Field Notebook:** NOT RECOVERABLE FROM PLAN.

- **Milestone 4 accessibility, performance hardening, and audit:** NOT RECOVERABLE FROM PLAN.

- **Milestone 5 production launch with 2 starter birds per account:** NOT RECOVERABLE FROM PLAN.

- **Headless simulation suite of 10,000 virtual users over 30 days:** The rationale is release calibration proof for non-zero drift, no negative drift under absence, and no premature saturation.

### Risk Management & Failure Modes

- **Drift calibration imbalance mitigation:** The rationale is avoiding drift that feels too fast, "Tamagotchi feel," or too slow, "screensaver feel."

- **Browser autoplay block mitigation:** The rationale is that audio may be blocked before interaction; the canvas should render in silence without error, show captions, and awaken audio on first touch or click.

- **Multi-device sync stutter mitigation:** The rationale is preventing birds from snapping across perches during rapid tab switching.

- **Ignoring snapshots with older timestamps:** The rationale is avoiding stale visual state during sync.

- **Screen-reader queue overload mitigation:** The rationale is preventing narration updates from flooding the screen reader speech queue.

- **PII leakage mitigation:** The rationale is preventing emails from being logged in application traces through synthetic account UUIDs, encrypted isolated storage, blind index, and log scrubbers.
