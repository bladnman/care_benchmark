# Pocket Aviary - phase-2A reconstruction

## System-level intent

- Testable product promises, not soft principles. The plan opens by saying the PRD's principles become invariants because "a principle without a test is a suggestion." This shows up again in the meaning of "MUST" as "a release-blocking invariant with an automated check," in the invariant table, the performance budgets as "gates," the calibration harness, and the definition of done: "every invariant in section 1 has its automated check green."

- One canonical simulation writer. The plan repeatedly protects "single writer": only the simulation tick writes canonical aviary state, the API "never computes or writes simulation state," on-demand ticks use the same tick function, and sync has "one committed version." This is the basis for multi-device consistency, idempotent events, and no client-side merge.

- Deterministic engine, reproducible experience. The plan uses a pure `packages/engine` with "no I/O, no wall clock, no global randomness"; every function takes `now` and a seed. It requires "same state + events + time + calibration -> same outputs" and says cues, phrases, greetings, and notebook variants are reproducible from seeds.

- Client as performer, not simulator. The render-pipeline boundary says the server emits a render-safe snapshot plus a cue sheet, and "the client is a performer of the cue sheet, not a simulator." Anything a second device or visitor must see identically is a cue; texture such as micro-motion and leaves is client-local.

- Slow, monotonic, bounded change. The plan's drift philosophy is "monotonic and bounded": deltas are non-negative, capped per day, never exceed 1.0, and no negative drift path exists. It explicitly wants "a single session never moves a personality value visibly" while a 0.08 trait change is "noticeable in retrospect."

- Ambient quietness without punishment. The attention accumulator is named the "ambient quietness" mechanism: it decays when the user is away and makes an ignored aviary quieter "without ever lowering a trait." This preserves the no-hunger, no-health, no-death, no-distress stance while still allowing absence to matter.

- Privacy boundary as architecture. The plan says "raw personality values never leave the simulation service" except export, "telemetry never carries an account or bird dimension," analytics has "NO path to Postgres(sim)," and email appears only encrypted plus a blind index. It treats privacy as schema, network policy, API allowlists, retention, and copy-surface constraints.

- No announcements and no user-behavior surfaces. The plan bans "toasts, banners, modals-on-return, badges, counters, or celebratory states" and says notebook, narration, and captions describe the aviary, "never the user." Out-of-scope items reinforce this with no achievements, streaks, levels, scores, bird/visit/day counts, re-engagement email, rankings, or unread states.

- Two voices, one registry. User-visible strings come from a registry tagged `naturalist` or `system`. Naturalist prose is "lowercase, present tense, no exclamation, no 'you'"; system prose is "sentence case, direct, no bird vocabulary." The plan makes product voice an engineering system through copy lint and string-literal bans.

- The aviary must appear first and calmly. The plan says "the first frame is the aviary," "no spinner component exists," the HTML inlines the snapshot, and the final definition of done is "a person who opens the tab meets a bird that notices them, in under half a second, with no text telling them so."

- Procedural identity over assets. The plan bans recorded audio and gives each bird a fixed call signature, tag motif, and farthest-point-selected perceptual distance so the user can "know Pip from Wren by ear." It pairs this with rigged Canvas2D birds, parametric animation, procedural weather beds, and deterministic phrase/caption generation.

- Accessibility ships in v1. Accessibility is not optional polish: reduced motion, narration, captions, keyboard navigation, focus treatment, and AA contrast are "release-blocking." The reduced-motion register has "its own render path," its own screenshots, and its own design review.

- Calibration from harness and staff accounts, not production behavior mining. The plan states that production aggregates are off-limits for drift distributions and interaction calibration. "Calibration evidence comes only from the harness and staff accounts," while RUM is aggregate-only and telemetry avoids per-account or per-bird dimensions.

## Per-feature whys

### Scope and product surface

- Single-user accounts and email magic-link sign-in: The plan grounds magic links in non-enumeration and transactional simplicity: `POST /auth/magic-link` "Always 202," creates a 15-minute single-use token, and alarms on delivery because "magic links must arrive quickly."

- Per-device sessions with revocation: NOT RECOVERABLE FROM PLAN

- Email change with verification: NOT RECOVERABLE FROM PLAN

- JSON export: The plan says export is the one place raw personality values leave the simulation boundary because it is "data portability, not a product surface."

- Soft-then-hard deletion: The plan gives the reason as reversible deletion before final removal: soft delete pauses the aviary, signed-in pages show deletion pending with an "I changed my mind" action, cancel within 30 days restores, and hard delete later removes rows, exports, Redis keys, invitations, and email-provider state.

- Plain-text account privacy page: The plan's stated purpose is to list aggregate categories and exclude per-bird state, matching the privacy boundary and making the absence of per-bird/account analytics visible.

- One aviary per account: NOT RECOVERABLE FROM PLAN

- Two starter birds and adoption: The plan wants the first experience to be a living aviary, not a catalog or empty state: "two birds arrived this morning," the first bird flies in after adoption, the user "never sees an empty aviary again," and naming does not alter identity.

- User-assigned, renameable names: The rationale is stable identity. `bird.id` is assigned once and naming "does not alter identity"; names are product language, while identity remains the arrival-time id and fixed signature.

- Six species as a count: NOT RECOVERABLE FROM PLAN

- Nightjar handling: The plan excludes the nightjar from starters "so the first encounter is daytime-active," lets the nightjar greet at night if present, and makes newly arrived birds `alert` for their first 10 minutes so a night signup still meets active birds.

- Growth to seven birds on an age schedule: The plan grounds this in non-gamified growth and quality control. Arrivals depend "only on age," nothing counts visits or interactions toward them, the schedule is materialized so it is "auditable and shiftable," and the seventh slot can be delayed if the listening study fails.

- Native app exclusion: NOT RECOVERABLE FROM PLAN

- Gamification exclusion: The plan excludes achievements, streaks, levels, scores, badges, counts, calendars, rankings, and unread states to preserve "no announcement surfaces" and "no user-behavior surfaces."

- Tamagotchi mechanics exclusion: The plan excludes hunger, health, decay, death, distress, and negative drift because the engine must have "no negative drift path at all" and absence should produce quieting, not punishment.

- Social-network exclusion: The plan excludes profiles, follows, feeds, discovery, comments, chat, co-presence, avatars, leaderboards, and cross-aviary ranking so visits remain read-only sharing rather than a social network.

- Push, marketing email, and re-engagement email exclusion: The plan limits email to transactional messages only, preserving the no-announcement/no-re-engagement stance. The optional visit notification is opt-in, off by default, email only, and capped.

- Payments: NOT RECOVERABLE FROM PLAN

- Shared or multi-aviary accounts: NOT RECOVERABLE FROM PLAN

- Customizable scenes: NOT RECOVERABLE FROM PLAN

- Catalog-style bird picking: NOT RECOVERABLE FROM PLAN

- Personality numbers excluded from product surfaces: The plan says "the user never sees the numbers" and implements that at the wire level with `expression` bands, `paletteStep`, categorical values, and contract tests banning raw trait names from snapshots.

- WebGL deferred: The plan chooses Canvas2D because WebGL context creation costs 50-150 ms on mid-tier phones, hurts the first-bird budget, adds renderer weight, and makes the reduced-motion sibling path harder.

- Recorded-audio fallback exclusion: The plan bans recorded audio because all sound should be synthesized at runtime; CI rejects audio files and even fallback voices must remain procedural.

### Architecture, data, and API

- Edge-inlined snapshot HTML: The plan uses an inlined snapshot so the "first frame is the aviary" before the main bundle loads, with private no-store HTML and content-hashed immutable assets.

- Stateless API plus on-demand ticks: The API appends events and reads snapshots, but responsiveness still goes through an on-demand tick so "responsiveness never creates a second writer."

- Sim scheduler and worker pool: The plan uses scheduled active and dormant ticks so the aviary advances server-side whether watched or not, while per-aviary leases and one transaction keep canonical state coherent.

- Shared engine package: TypeScript everywhere is chosen because "engine sharing is the reason"; the same pure package supports workers, API on-demand ticks, client audio/caption/narration helpers, and deterministic replay.

- Client-safe engine subset: The plan excludes trait-to-behavior mapping, drift, mood, and `expressionProfile` from the client build "structurally rather than by discipline," preserving the raw-personality boundary.

- Snapshot plus cue sheet boundary: The rationale is identical shared experience: render-safe canonical state plus cues for things a visitor or second device must see the same, while texture remains local.

- TypeScript, Fastify, Postgres, Redis, Vite, Preact stack: The plan gives explicit reasons only for TypeScript engine sharing and Preact's small chrome footprint; other stack choices are mostly stated as implementation decisions.

- Synthetic account id and email blind index: The blind index exists because "a sign-in by email needs a lookup key," but an HMAC with a server-held key is "not the email," not reversible, and not used as an identifier elsewhere.

- Traits as columns: The plan chooses columns instead of a JSON blob so per-day caps and monotonicity can be asserted with checks and a trigger rejecting decreases.

- Append-only, short-lived interaction events: Events are append-only and partitioned because only the tick and retention job read them, processed events are purged, and privacy requires per-event interaction data to disappear after consumption.

- Snapshot endpoint with `hello`: The plan uses it to ensure freshness, log a hello event, and include a greeting plan computed from absence length; ETags let clients ignore unchanged versions.

- Event endpoint with batching and idempotency: The rationale is safe retry and correct accounting: client-generated ids dedupe, server stamps time, clips presence, rejects future intervals, and keeps listen-in credit clamped to presence.

- Notebook pagination endpoint: NOT RECOVERABLE FROM PLAN

- Bird list and rename endpoint without traits: The endpoint exposes ids, names, species descriptors, and arrival dates but "No traits," matching the rule that product surfaces never show personality numbers.

- Host visit invites and visitor snapshots: The plan makes visits sharing without write access: visitors get projected snapshots, no events endpoint accepts visitor cookies, and visitor pulls can mark the aviary active but write nothing to interaction events.

### Simulation engine

- Continuous-time deterministic integration: The plan wants results not to depend on tick granularity, so `dt` is explicit and 15 one-minute ticks must approximate one fifteen-minute tick.

- Host-local timezone without geolocation: The plan uses the host's stored IANA timezone and a dawn/dusk table with "no geolocation"; visitors follow host lighting so they see the host's aviary, not their own local day.

- One-transaction tick pipeline: The plan uses a Redis lease, event normalization, state advance, cues, observer, optimistic `state_version`, and `processed_version` in one transaction to prevent duplicates and second-writer races.

- On-demand ticks: The plan uses the same tick guarantees for offers, listen-in, hello, and ensure-fresh responsiveness, with coalescing and timeout, so immediate interactions do not bypass the simulation authority.

- Presence accounting: The plan defines presence as visible plus focused plus recent activity to avoid crediting an open unfocused tab, unions devices to wall-clock, and includes `focusin` so keyboard and assistive-technology users can register activity.

- Attention accumulator: The plan says `A` is engine state, not a trait; it decays when away and creates quietness without lowering any trait, while still letting prior attention influence drift during absence.

- Drift function, channels, caps, and quadratic headroom: The plan uses non-negative presence and interaction deltas, daily caps, and `(1-T)^2` because linear headroom would saturate a regular user's bird within a year; quadratic headroom keeps movement over years.

- Expression mapping from traits to behavior: The plan maps traits to perch choice, call rate, responses, greetings, offers, looks, plumage, and posture so change is observable through behavior, not numbers, with a tested visibility threshold.

- Mood model: The plan uses pressures, hysteresis, dwell time, time of day, weather, interactions, contagion, and personality gates so mood persists, changes smoothly, and "re-derives from the day's context without ever snapping."

- Weather state machine: Weather is canonical so all devices and visitors see the same rain; it also feeds mood pressures, call-rate factors, rendering, and notebook history.

- Cue planning: The plan creates a 120-second cue horizon with deterministic ids and a 60-second overlap so a late snapshot "never leaves the client without cues."

- Return greeting: The plan makes return noticed through absence bands, exactly one primary greeter, staggered secondary responses, and a night behavior where there is "still a notice, never nothing."

- Offers: The plan gives users quiet seed, pool, and song interactions without timers or countdowns; one active offer and per-bird cooldowns keep reactions bounded and non-gamified.

- Settle: Settle exists as device-local evening grade and quieting; canonically it ends presence/listen-in and adds a drowsy impulse, with a five-second undo to cancel if the tick has not yet run.

- Bird-to-bird interaction: The plan makes warmth and social response visible through replies, contagion, chorus, `lookToward`, and adjacent-slot bias for high-warmth birds.

- Call signatures and grammar runtime: Fixed signatures, tag motifs, farthest-point selection, and bounded mood modulation exist to make birds recognizable by ear at up to seven birds while keeping phrases varied.

- Starters and newcomer arrivals: Starters are different species and daytime-active; later arrivals are age-based, auditable, cannot be declined, never expire, and are recorded in the notebook because a bird that arrived "is part of the aviary."

- Notebook observer: The plan uses salience, sparsity, deterministic templates, and "told from the birds' side" so the notebook is neither chatty nor a user log, and no LLM needs per-bird state.

- Dormant cadence, catch-up, and ensure-fresh: The plan keeps dormant aviaries ticking every 15 minutes and active ones every 60 seconds so the aviary always advances; ensure-fresh integrates elapsed time before snapshots after gaps or outages.

- Engine invariant tests: The rationale is to make the product promises executable: monotonic drift, dt independence, identity, determinism, presence, greeting, growth, and notebook sparsity all have tests.

### Sync model

- `state_version` as the sync primitive: The plan uses one committed version so clients replace snapshots wholesale; "there is no client-side merge because there is nothing to merge."

- Snapshot delivery by inline HTML, SSE, and pull triggers: The plan combines first-navigation speed, version notifications, proxy fallback, resume handling, and jittered polling to keep clients fresh without stale replacement.

- Client cue reconciliation: Continuing already-started cues, dropping not-yet-started old cues, deterministic overlap, and flight corrections avoid visible swaps and "never a teleport."

- Event outbox and offline handling: Client-generated ULIDs and idempotent inserts make retries safe, while presence intervals drop after five minutes offline because they are "not replayable honestly."

- Multi-device semantics: The plan makes devices share the same version and cues, unions presence to wall-clock, allows settle to be local, lets last hello win timezone, and keeps visitors from contributing anything.

- Failure handling: The plan prefers safe reprocessing and calm degradation: lost leases discard work, stale snapshots are ignored, API outages make birds go quiet after the horizon, and the connection line is persistent status "not a toast."

### Frontend rendering pipeline

- Custom Canvas2D renderer: The plan chooses it for <= 7 rigged vector birds at 60 fps, no WebGL context cost, small renderer size, and easier reduced-motion sibling rendering.

- Boot path and first frame: The plan inlines critical CSS, the snapshot, and compact rigs so the first frame is drawn before the main bundle, targeting about 350-400 ms typical and a 500 ms CI ceiling.

- Scene composition with stage variants: The plan uses viewport-specific stages, anchored perch slots, in-bounds margins, and precomputed flight arcs so every bird is always in frame by construction.

- Bird rig and procedural animation: The rig makes plumage drift visible through palette steps and feather overlays; parametric motion avoids keyframed loops and lets the beak open on the actual notes.

- Cue execution and interpolation: The scheduler maps server time to local time, aligns audio and beak envelopes, cross-fades mood, and resolves large pose disagreements with a normal flight instead of a teleport.

- Day/night and settle grading: Host-time palettes and device-local settle grades let the scene quiet visually and sonically without making settled evening a canonical state.

- Weather and ornaments: Weather renders canonical rain and wind, while leaves and feathers are "client-only and not state" so they add texture without sync burden.

- Reduced-motion render path: The plan gives reduced motion its own `PoseRenderer`, still poses, slow cross-fades, no parallax or ornaments, and unchanged calls, drift, mood, notebook, captions, and narration.

- Hidden tab, suspend, and resize behavior: The plan stops rendering and presence when hidden, resumes with hello or a plain pull depending on hidden time, and caps DPR so performance stays bounded.

- Top bar and chrome: Four icons plus lowercase "settle," fading chrome, transient sheets, no badges, no counts, no status dots, and no persistent scene chrome preserve the quiet aviary surface.

- Loading, empty, and adoption states: The plan uses the quiet field instead of a spinner and ensures that after adoption the user never sees an empty aviary again.

- Frame loop and adaptive quality: Adaptive quality preserves bird animation fidelity first: rain particles and pointer parallax degrade before birds, and step-up waits for sustained headroom.

- Memory discipline: Typed pools, reusable objects, fixed offscreen canvases, listener discipline, closed `EventSource`, persistent audio nodes, and virtualized notebook exist to satisfy the soak-test heap and DOM gates.

### Audio pipeline

- Audio graph: Per-bird voices, panning, depth filters, runtime reverb, compressor, and master gain create spatial depth while keeping seven voices plus ambient bounded.

- AudioWorklet voice synthesis: The plan uses persistent processors, preallocated buffers, no per-call nodes, and procedural oscillators/noise/formants to keep CPU low and obey the no-recorded-audio rule.

- Phrase variation and captions: Phrases derive from `signatureSeed` and `phraseSeed` so no two are identical, and captions come from the same syllables, meaning "the caption is what was actually sung."

- Chorus: Independent voices and shared reverb/compression let two devices hear the same chorus while preventing loudness build-up and shared-phase artifacts.

- Listen-in mix: Gain and filter ramps focus one bird without muting the others; the floor is "never a mute," and server bias changes future cue sheets rather than immediate state.

- Ambient beds: Wind and rain are filtered-noise synths with fades because the plan allows "no recorded ambience of any kind."

- Settle and night audio: Settle lowers master gain and call intensity locally; night reduces the master while the engine already lowers call rates and the nightjar keeps calling.

- Audio scheduling and time base: Server time maps to `AudioContext.currentTime`, cues post at least 500 ms ahead, and the worklet stays sample-accurate even when main-thread timers throttle.

- Autoplay policy and silence fallback: If audio is suspended or unavailable, the aviary still renders normally with captions on, resumes after a gesture when possible, and never shows an "enable sound" banner over the scene.

- Audio quality gates: Signature-distance tests, listening studies, chorus artifact checks, and golden-ear review exist to prove birds are identifiable, varied, and not sonically broken.

### Accessibility surfaces

- Narration engine: Narration uses scene sentences, not state lists, picks one or two salient facts, alternates live regions for re-announcement, and introduces birds with descriptors on first mention.

- Captions: Captions come from the actual phrase, stay near the bird, guarantee contrast with backdrops, limit simultaneous captions, and default on when audio is blocked or off.

- Keyboard and focus: Roving hotspots, listen-in controls, offer placement controls, settle undo, opt-in single-character shortcuts, and visible focus treatment make the scene operable without pointer input.

- Semantics: The plan makes the canvas `aria-hidden`, exposes bird hotspots as named buttons, uses dialogs for sheets, and labels the notebook as a list so assistive technology gets structured controls instead of raw canvas.

- Reduced-motion and settings: System preference applies automatically, account override syncs, and per-device evaluation lets reduced motion be respected without changing canonical simulation.

- Contrast: Chrome, icons, captions, narration, and focus rings have explicit contrast requirements, and the top bar never fades while focused or hovered because faded state is not a reading state.

- Accessibility testing: Axe, Playwright keyboard flows, screen-reader matrices, reduced-motion screenshots, and writer review of narration transcripts enforce that accessibility ships in v1.

### Voice and copy

- Copy registry: The registry makes "two voices, one registry" enforceable by requiring every user-visible string to carry a `naturalist` or `system` tag and banning string literals in UI components.

- Naturalist voice: The rationale is to describe the aviary, not the user: lowercase, present tense, no exclamation, no "you/your," no rewards/streaks/badges/scores, and no numerals for user actions.

- System voice: The rationale is direct operational clarity: sentence case, short messages, next steps for errors, and no bird vocabulary in auth, sessions, errors, settings, privacy, visitor pages, unsupported browser, or connection lines.

- Template grammar instead of LLM prose: The plan says no LLM is involved so generation is deterministic, reviewable, keeps per-bird state inside the simulation boundary, and allows writer-owned voice QA.

### Accounts, privacy, security, and visits

- Magic-link consumption: The consume path is atomic so a token is single-use; replayed or expired links get a system-voice failure page and no account enumeration.

- Session listing labels: NOT RECOVERABLE FROM PLAN

- Export contents: Including personality vectors and attention in export is justified as the one explicit data-portability exception to the no-numbers product rule.

- Deletion jobs and audit: Hard deletion removes account-tied rows and provider state, while the remaining audit record holds only UUID and timestamps for 90 days.

- Privacy pipeline: Network isolation, label allowlists, log redaction, no analytics SDK, and retention jobs implement "privacy boundary is architectural" rather than relying on policy alone.

- Security controls: Cookies, Origin and Sec-Fetch checks, CSP, HSTS, JSON schema validation, dependency audit, KMS, and least-privilege DB roles enforce auth and visitor separation.

- Visit privacy: Visitors see bird names, species, moods, perches, weather, and host-time lighting, but never host email, settings, notebook, or visit log.

- Visit revocation and forwarded links: One-time consumption binds a visitor cookie, and every pull checks expiry and revocation so forwarded links remain bounded by host control.

- Visit log: The plan states the host sees the visitor email because "the host typed it," with date and duration rounded to 5 minutes.

- Visit notifications: They are opt-in, off by default, email only, at most one per visitor per day, and not mentioned during onboarding to avoid push or re-engagement patterns.

- Unsupported browsers: Required feature detection leads to a system-voice unsupported page; audio-only failures fall back to captions and silence.

### Performance, observability, testing, and rollout

- Performance budgets: The first-bird, bundle, frame-time, heap, snapshot, tick, event, email, and worklet budgets are gates because the product promise includes meeting a bird quickly and calmly.

- Server observability: The plan measures routes, ticks, backlogs, leases, snapshot bytes, SSE counts, email latency, jobs, and retention rows so operations can run the single-writer system without account labels.

- Client RUM: RUM is aggregate-only, sampled, coarse, and sent once at session end to measure first-bird time, frame timing, audio state, failures, and session buckets without per-account or per-bird dimensions.

- Synthetic fleet: Staff-account Playwright runners "hit production like a user" to watch first-bird time, greeting execution, frame time, soak, and accessibility without production behavioral calibration.

- Deliberately unmeasured data: The plan refuses per-account/per-bird metrics, drift distributions across users, retention/streak/engagement dashboards, notebook content, and experiments conditioned on bird state to protect the privacy and no-gamification intent.

- Unit, property, and contract tests: These tests turn invariants into executable checks for engine behavior, presence, copy, snapshot allowlists, visitor projection, event validation, grants, trait triggers, and metrics labels.

- Calibration harness: The harness runs personas over 400 simulated days because constant changes require sign-off and production aggregates are off-limits.

- Listening studies: Internal and external studies verify bird identification and repetition; if seven-bird chorus quality fails, the plan delays later arrival slots instead of shipping "blurry."

- End-to-end tests: Magic link, adoption, greeting, listen-in ramps, offer round trip, settle undo, multi-context sync, visitor revocation, export, deletion/restore, and dormant catch-up prove the joined product workflows.

- Rollout milestones: The milestone order reflects dependencies: copy registry and banned identifiers from day one, harness before calibration claims, motif libraries before listening study, both render paths before accessibility matrix, and staff backdated aviaries before seven-bird study.

- Ramping birds per aviary: The age schedule ramps complexity gradually; schedules never shift earlier as a reward, and two knobs allow global schedule changes or pause.

- Launch checklist: Budgets, accessibility, privacy, security, runbooks, and on-call must be green before open signup, with "no in-product launch surfaces."

- After-launch changes: Weekly patches and versioned constants with harness sign-off keep calibration controlled, and no feature touching the no-announcement or no-user-behavior invariants proceeds without a PRD change.
