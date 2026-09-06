## System-level intent

1. The aviary should feel already alive, not launched for the user.
   - Shows up in the product boundary: "appears to have been continuing before it was opened."
   - Shows up in initial render: draw "existing motion" before any greeting descriptor, slow paths show "a quiet field of sky" without a "spinner," and the first-adoption fly-in is "never" reused on return.
   - Shows up in risk language: avoid a "Dead/canned arrival"; V1 is ready when "the first scene already feels in progress."

2. The birds are durable individuals, recognizable across time, devices, drift, migration, and rollback.
   - Shows up in the product boundary: "Two individually recognizable birds" with "persistent identities and slowly changing personalities underpin the relationship."
   - Shows up in the data model: stable `bird_id`, "persistent identity seed and voice signature," "hidden trait vector," and "No path renames or replaces the ID when the name changes."
   - Shows up in durability and risks: "never reconstruct" the vector, "never silent reseeding," "No release is allowed to use 'regenerate birds' as a rollback strategy," and "Loss of identity during migration or rollback."

3. Attention matters, but never becomes duty, scoring, punishment, or a reward loop.
   - Shows up directly in the product boundary: "Attention matters without becoming a duty or a score."
   - Shows up in exclusions: "every form of gamification," "no hunger or happiness meters," "distress or death from absence," "visit streaks," "adoption counters," "reminders to return," or "rewards for frequency of use."
   - Shows up in drift: absence, muted audio, failed requests, declined offers, settle, ordinary closure, and visitor watching supply "no negative dose"; no "reward for leaving the aviary open."

4. Authority belongs to the server-owned canonical simulation, while the browser owns presentation only.
   - Shows up in architecture: simulation worker is the "sole writer of personality, mood, drift-filter state, canonical bird actions and call schedules."
   - Shows up in ownership path: browsers "render the same canonical state"; "Only local presentation differs."
   - Shows up in interactions: an offer "cannot optimistically change personality, mood, or acceptance"; the client "Never advance[s] personality or choose[s] a new mood."

5. Product surfaces expose behavior, not hidden numbers.
   - Shows up in the conflict decision: ordinary surfaces, DOM/ARIA, diagnostics, snapshots, and visitor responses "never expose trait numbers."
   - Shows up in snapshots: projections "do not expose a normalized trait vector or hidden daily totals" and use allowlists plus negative serialization tests.
   - Shows up in notebook and telemetry: entries do not "expose trait numbers," and operational metrics must not attach "vector values."

6. Privacy is structural, not just policy language.
   - Shows up in architecture: telemetry has "no simulation role," operational collector "Cannot query the simulation database," and owner snapshots are private.
   - Shows up in email handling: plaintext addresses are resolved only by the mail dispatcher, jobs carry opaque references, and mail carries "no bird observations."
   - Shows up in observability: no "analytics export, training feed, recommendation input, session-replay product, or third-party behavioral SDK."

7. The voice is quiet, naturalist, sparse, and matter-of-fact.
   - Shows up in the product boundary: "one quiet scene," "occasional noticing," and a "sparse field notebook."
   - Shows up in scene rules: no permanent labels, mood icons, hover tooltips, offer buttons, status badges, or score-like numbers in the scene.
   - Shows up in copy and errors: use "naturalist vocabulary," no welcome/away-time text, no guilt or failure labels, and direct "matter-of-fact" error or connection language.

8. Accessibility is the same aviary experience, not a fallback state-list.
   - Shows up in V1 scope: "Full-quality accessible narration, call captions, keyboard control, designed reduced-motion presentation, and graceful silence."
   - Shows up in frontend sections: semantic DOM for top bar/dialogs/notebook/narration/captions/keyboard controls, roving tab stop, live region, caption placement, contrast gates, and reduced-motion as "a separate rendering design."
   - Shows up in rollout: screen-reader, caption, and reduced-motion surfaces are "launch blockers, not a post-launch phase."

9. Procedural variation must remain recognizable and non-canned.
   - Shows up in greeting: "no canned sequence or rotating set of three clips" and exact repeats are rejected.
   - Shows up in audio: grammar versions, voice signatures, variation seeds, bounded continuous variation, and recognizable anchor intervals.
   - Shows up in risks: prevent "Audio uncanniness or blurred identity" and "Dead/canned arrival."

10. Absence is safe, while the aviary continues.
    - Shows up in exclusions: no "distress or death from absence," "reminders to return," or frequency rewards.
    - Shows up in simulation: "Advance unobserved aviaries on the same schedule as observed ones."
    - Shows up in drift and tests: traits are nonnegative, every trait after fourteen days absent is "at least its departure value," and "no distress, desaturation, or mistrust appears."

11. Observations and notebook prose must be truthful, sparse, and evidence-based.
    - Shows up in notebook generation: prose comes from "committed behavioral evidence" and "specific facts are true."
    - Shows up in ranking: "Evaluate novelty and evidence, not whether a session ended."
    - Shows up in tests: "Sparse true observations, no session-log substitutes, no user-frequency wording, no trait numbers."

12. Performance is part of the feeling of the product.
    - Shows up in the performance section: "Performance is a launch requirement because a late, static or stuttering first bird changes the experience."
    - Shows up in initial render: no spinner, no fake starter bird, no blocking font/audio/settings chunk on the first-bird path.
    - Shows up in rollout: first-bird, tick, persistence, leak, accessibility, and listening gates must pass before cohorts expand.

13. Sharing is optional, individually addressed, one-time, render-only, and revocable.
    - Shows up in the product boundary: "Optional, individually addressed, one-time visit invitations" where "a visitor sees the actual ambient aviary and has no simulation input."
    - Shows up in visit isolation: visitors receive a "read-only projection" with no owner event dispatcher, greetings, listen-in, offers, settle, notebook access, or social overlays.
    - Shows up in revocation: every pull is authorized, revocation invalidates issued sessions, and stale cached bodies or `304` cannot extend access.

14. Calibration and rollout rely on synthetic fixtures and voluntary review, not production behavior mining.
    - Shows up in drift calibration: use "fabricated histories, deterministic seeds, and explicit research sessions with test aviaries"; production telemetry never collects trait measurements.
    - Shows up in shipping: accelerated histories run only on "marked synthetic fixtures," and a pilot tracks "operational quality only" while soliciting direct qualitative feedback.
    - Shows up in observability: do not instrument "retention goals, daily-use streaks, click funnels, population drift or return-notification conversion."

## Per-feature whys

### 1. Product boundary and decisions

- Browser aviary: The plan wants "one quiet scene" that appears to have been "continuing before it was opened," with no native apps, scene customization, public discovery, profiles, comments, chat, or co-presence.
- Email magic-link accounts: The plan uses purpose-bound transactional mail and excludes passwords, SSO, and marketing/reminder messages; scanner-safe consumption and generic responses prevent accidental sign-in or account enumeration.
- One canonical aviary per owner: This removes personality merge conflicts and ensures devices "read the same canonical record" rather than creating conflicting state.
- Device session management: Sessions are per-device so revoking a device can end its pending attention lease while preserving exactly-once semantics for work accepted before revocation.
- Verified email changes: The old address remains authoritative until valid verification so identity is not moved by an unverified pending address.
- Export: A privately delivered JSON export is the "narrow data-portability exception" that allows current vectors outside ordinary product surfaces.
- Recoverable deletion: The plan keeps birds, vectors, and notebook "durably intact" during the thirty-day window so recovery has continuity, then uses hard erasure so restoration cannot revive deleted data.
- Six coherent species: The plan ties species to silhouettes, palettes, motif/grammar versions, pitch ranges, mood-response coefficients, and nocturnal behavior for coherent identity. Why exactly six launch species: NOT RECOVERABLE FROM PLAN.
- Two server-selected starter birds: Two birds establish the relationship immediately; different species establish "contrasting silhouettes/calls."
- Naming and renaming: Naming personalizes birds while stable IDs preserve identity; renaming has "no simulation side effects" and old notebook prose remains "as originally written."
- Age-based opportunities to adopt additional birds: Additional birds arrive over months by elapsed aviary age, "never interaction frequency," so they are not earned through attention.
- Hard seven-bird limit: The limit is enforced in transactions and interface, and seven-bird layouts, performance, chorus, and rollout are tested. Why exactly seven: NOT RECOVERABLE FROM PLAN.
- Server-owned personality drift: Personality changes slowly from qualified positive evidence while preventing client writes, backdating, subtraction, or population-tuned manipulation.
- Persistent mood: Mood gives daily expression and prompt reactions while remaining distinct from personality; wary can arise from ambient behavior, "never as punishment for user absence."
- Bird-to-bird responses: Probabilistic responses, refractory intervals, and bounded chorus make the flock social without endless echo chains or synchronized arrival cues.
- Time-of-day behavior: One IANA aviary timezone and authored day-phase curves keep the canonical day stable across traveling devices and avoid geolocation/weather dependencies.
- Rare ambient weather: Rain and wind add restrained ambient variation with no thunder, distress, prompts, or real-world weather dependency.
- Return-greetings: A return is a server-observed event so birds can notice promptly, with absence shaping gesture distribution, while no away-time text is shown.
- Listen-in: Listen-in is local audio presentation plus bounded attention evidence; it gives focus without letting audio prove attention or multiply credit.
- The three offers: Seed, song fragments, and still pool are gestures that allow shaped responses without inventory, thirst mechanics, personality targeting, placement commands, or guilt.
- Settle with five-second undo: Settle is an "optional goodbye, not a completion state"; undo consumes an immediate activation to reverse only the active transient envelope.
- Qualified presence accounting: Presence is narrowly defined so attention can matter without surveillance, idle manufacture, offline replay, or device multiplication.
- Indefinitely readable, read-only notebook: The notebook preserves sparse naturalist observations as immutable history; cursor pagination keeps all history accessible without edits, annotations, or deletes.
- Optional visit invitations: Visits let an owner share the actual ambient aviary only by deliberate, individually addressed action; visitors have no simulation input.
- Accessible narration: Narration gives the same canonical scene through a naturalist lexicon without reading vectors, event codes, or mechanical lists.
- Call captions: Captions come from the final procedural score so the text corresponds to the audible phrase and remains available in muted/fallback states.
- Keyboard control: Keyboard paths make bird focus, listen-in, offers, settle, dialogs, and menu controls fully operable without pointing.
- Designed reduced-motion presentation: Reduced motion preserves semantic actions, calls, mood, greeting, notebook, and drift while replacing motion with authored still-pose cross-fades.
- Graceful silence when WebAudio cannot run: The aviary continues with captions and clear settings state rather than blocking behind audio permission or pretending autoplay worked.
- Performance instrumentation and aggregate operational telemetry: The plan measures first-bird, frame, audio, tick, and error quality because performance affects product feel, while structurally separating metrics from private simulation data.

### 1.1 Explicit interpretations of conflicting or incomplete requirements

- Export vector exception: The plan chooses account export as the only place current vectors may be inspected because the absolute trait-number prohibition and explicit export contents "cannot both hold."
- Visit-notification toggle: The plan ships the PRD's named off-by-default exception while preventing onboarding prompts, push, badges, banners, toasts, marketing, or return reminders.
- Four top-bar groups with settle: Grouping settle under "offer or settle" keeps top-bar settle reachable without adding a fifth icon.
- Accessibility exceptions to no scene chrome: Captions and focus indicators are allowed because accessibility requirements override the scene's ban on labels/chrome.
- Expedited reducer pass: Prompt greetings and offer responses use the same transaction/event cursor/elapsed-time mathematics so reactions do not wait a minute and drift is not accelerated.
- Autoplay fallback: The scene renders action and call timing immediately, but audio only starts when permission and preference allow; missed calls are not backlogged or faked.
- Canonical aviary timezone: A single account timezone prevents traveling devices from continually changing the canonical day.
- Listen-in focus and Enter behavior: Focus begins listen-in and Enter ensures engagement so keyboard users do not trigger duplicate attention windows.
- One-time visit session lifetime: Expiry, atomic redemption, and a two-hour render-only session preserve one-time semantics and require new invitations for later visits.
- Revoked visit log preservation: Revocation removes access while preserving a marked historical row so the host can still see what was shared.
- Presence activity definition: The five-minute trusted pointer/key window prevents visibility, focus, timers, synthetic events, or tap-only idle from counting as qualified presence.
- Missing visual design-system document: The plan makes concrete design tokens, silhouettes, contrast combinations, and motion studies a first delivery task because unseen design specifications cannot be assumed.

### 2. Architecture and authority boundaries

- Compact TypeScript web application with server-rendered HTML and hydrated control shell: This supports immediate bootstrap rendering with a lightweight control layer rather than a per-frame framework rerender.
- Dedicated Canvas 2D scene renderer: Canvas allows a bounded scene graph, offscreen caches, compact vector/atlas drawing, and moving birds without per-frame DOM layout.
- Native WebAudio synthesis: WebAudio allows procedural call scores, captions from the same score, graceful fallback, and no recorded calls or downloaded loops.
- TypeScript HTTP service and simulation worker sharing a versioned pure reducer package: Shared reducer logic keeps ordinary and expedited passes equivalent and versioned.
- PostgreSQL authoritative state, append-only events, auth/session records, and transactional outbox: This supports stable IDs, exactly-once event handling, durable simulation state, and no dual-write gaps.
- Scheduler dispatching due aviaries to workers: A due-state scan keeps unobserved aviaries progressing even if queue messages are lost.
- CDN for immutable assets and authenticated edge gateway for owner bootstrap: Static assets can be public while owner HTML/snapshots remain private behind current authorization.
- Modular service with two process roles, not independent microservices: NOT RECOVERABLE FROM PLAN.
- Domain contracts: They protect the boundary between public render projections and private canonical types, and prevent serializing private bird records wholesale.
- Auth/account service: It isolates identity, settings, consent, session, export, deletion, and email workflows from simulation data.
- Interaction ingress: It validates owner events, enforces idempotency, assigns sequence, and has "No personality-write privilege."
- Simulation worker: It is the single writer for personality, mood, drift filters, canonical actions, calls, greetings, offers, weather, and notebook candidates.
- Snapshot projector: It converts canonical state into owner/visitor render projections without vectors, history, email, or private aggregates.
- Visit service: It delivers and redeems invitations without any route to owner interaction ingress.
- Web scene runtime: It samples server action curves and synthesizes calls while never advancing personality or choosing mood.
- Operational collector: It receives only allowlisted counters/histograms and cannot ingest event payloads or query simulation.
- Owner event wait up to 250 ms before pending/poll response: The plan allows prompt results when the committed revision is ready while preserving the canonical server-authored outcome on slower paths.
- No optimistic personality/mood/acceptance changes for offers: User-visible success waits for server-authored reaction so failed writes do not create simulated success.

### 2.1 Initial render and edge delivery

- Session/account validation before snapshot: Authorization, deletion status, and revocation checks stay current even with cached projections.
- Embed minimal render projection and essential SVG poses in HTML: The bootstrap can draw the current scene before the control framework loads.
- Include server time and action phase: Hydration continues a mid-action pose without restarting it.
- Private edge projection cache: Opaque UUID/revision caching can reduce latency while keeping authorization on every request and invalidation explicit.
- Snapshot small enough for HTML: This supports first-bird critical delivery and avoids a separate blocking fetch.
- Optional prompt return event from gateway or post-bootstrap response: Greetings can arrive within one or two seconds without blocking existing motion.
- Slow origin quiet field without spinner: The plan avoids a loading treatment that would make the aviary feel like it begins only after data arrives.
- True first-adoption empty field/fly-in: This is the one intentional beginning sequence, while returns never reuse it.

### 3. Persistent data model

- UUID identifiers, UTC timestamps, IANA zone, monotonic revisions/sequences: These support stable identity, canonical local day behavior, ordering, and cross-device consistency.
- Account-scoped encryption and row-level ownership checks: Durable private payloads remain account-private and service/database boundaries both enforce ownership.
- Separate database roles: Ingress, worker, projector, and telemetry each receive only the privileges their boundary needs.
- `accounts`: Keeps email encrypted and separate from IDs/keys so email is not a shard key, metric dimension, public identifier, or message key.
- `aviaries`: Captures one active canonical aviary per owner with revision, schedule, PRNG, day/weather, greeting, and projection version.
- `birds`: Stores stable identity, hidden vectors, voice, drift filters, mood, action, and cooldown while preserving ID through rename.
- `species_revisions`: Keeps launch species assets/grammar available for living birds so revisions do not break identity.
- `interaction_events`: Append-only validated commands with unique IDs support retries across devices without client blobs or trait values.
- `presence_windows`: Records only eligibility/segment anchors without keys, cursor positions, screenshots, or idle behavior traces.
- `presence_coverage`: Counts overlapping owner devices once and is not a user-facing visit calendar.
- `attention_windows`: Prevents multiple devices from multiplying listen-in credit.
- `action_schedule`: Persists a bounded action/call horizon and compact observation evidence rather than a long unbounded future.
- `notebook_entries`: Immutable entries keep all history accessible and avoid edit/delete/annotate surfaces.
- `observation_memory`: Holds bounded private facts needed for truthful sparse observations without becoming population analytics or a personality-history rebuild.
- `device_sessions`: Stores hashed opaque tokens and coarse labels, never raw tokens.
- `auth_challenges`: Purpose-bound, one-use challenges and temporary pending-email ciphertext support scanner-safe auth and verification.
- `invitations`: One-time token hash and explicit host consent record support private, deliberate sharing.
- `visit_sessions`: Read-only credentials keep visitors separate from owner sessions.
- `visit_log`: Host-only transparency cannot feed drift or notebook generation.
- `export_jobs`: Opaque job references prevent queues from carrying addresses or exported state.
- `outbox_jobs`: Mail, invalidation, exports, cleanup, and wakeups avoid dual-write gaps.
- Trait and mood set: Traits are normalized internally and mood is one of five named states; weather/daily behavior modulates expression and "never subtract[s] from the vector."
- Bird name validation: Unicode names are allowed while trimming/escaping/excluding controls; duplicate names are allowed because UUIDs resolve ambiguity.
- Email and identity handling: Encryption, keyed lookup restriction, conservative normalization, and opaque delivery references keep email out of simulation, logs, analytics, and public identity.
- Durability and migration: Direct vector persistence, one-commit updates, constraints, backups, and restore drills prevent reseeding or identity loss.

### 4. Server simulation

- Sixty-second baseline tick cadence: A regular distributed cadence keeps observed and unobserved aviaries progressing without requiring a browser.
- `next_due_at`, leases, `SKIP LOCKED`, and row locks: These make work claimable in batches while preventing parallel writers.
- Due-state scan as authoritative: A lost queue message cannot stop simulation.
- Separate behavioral time and drift integration clock: Late-but-valid presence can be credited without backdating already-applied personality deltas.
- Same reducer for ordinary and expedited passes: Prompt interactions stay mathematically equivalent to minute ticks except for the inserted interactions.
- Deterministic random choices keyed by seed/boundary/event ID: Results do not depend on worker invocation count.
- Preserve ongoing actions and extend 120-second horizon: Snapshot arrivals and minute boundaries do not restart every bird.
- Sparse notebook candidate evaluation during ticks: Entries are based on committed behavioral facts.
- Transactional commit of state, event results, notebook, revision, and cache invalidation: Mutations and invalidations land atomically.
- Worker retry/fencing behavior: Crashes and ambiguous acknowledgments do not double-apply or partially mutate state.
- Catch-up after missed ticks: Outage recovery advances from durable checkpoints without inventing presence or retrospective notebook entries.

### 4.2 Precise presence and attention accounting

- Five-minute activity window: Trusted recent pointer/key input is required so focus/navigation alone cannot manufacture presence.
- Heartbeat/closed-segment delivery every 15 seconds: The client reports bounded closed intervals rather than coordinates, key contents, trajectories, or behavior samples.
- Suspension and pagehide handling: The plan prevents laptop sleep, hidden tabs, or missing heartbeats from becoming long presence segments.
- Server freshness and segment validation: Sequence, ownership, elapsed duration, receipt age, max size, duplicate IDs, replay, and impossible timestamps are checked without expanding into surveillance.
- Union coverage across owner devices: Overlapping tabs/devices count once and finalize only after the receipt window.
- Listen-in credit intersection with qualified presence: Local audio focus may work without earning idle-attention credit.
- Capped division across different birds: Different-device listening does not multiply total attention.
- Settle/tab-close terminal reason: Ending a session is not positive or negative drift input; other active devices remain valid.

### 4.3 Monotonic slow drift

- Stored leaky input filter per trait: Positive history can continue to matter during absence while the personality value itself cannot decay.
- Presence as dominant dose: Quiet qualified presence should drive most slow change while being capped so all-day open sessions do not compress months into days.
- Listen-in as smaller bird-specific influence: It concentrates on social warmth and vocal frequency without exceeding presence.
- Offers as small influence: Nearby eligible offers and actual acceptance contribute limited boldness/curiosity, with private ceilings and no visible counters.
- No negative inputs for absence, mute, decline, failure, settle, closure, or visitor watching: Muting and absence are never neglect.
- Exact continuous integration and batching invariance: Timing/subdivision cannot change credited dose or allow late backdating.
- Varied seed ranges and persistent ceilings: Birds keep individual differences instead of converging to identical maximums.
- Calibration gates: Synthetic histories and blinded qualitative review verify week-scale movement, no single-session jump, bounded long days, and no absence punishment.
- No production drift tuning: The plan avoids real population interaction histories, average production drift, and trait telemetry.

### 4.4 Mood, weather, and small social behavior

- Time-dependent stochastic mood transitions: Mood responds to local time, inertia, personality, accepted interactions, and weather on scheduled boundaries without flicker.
- Daily-ish renewal: Yesterday's transient influences decay gradually; local dawn changes baseline without resetting moods at midnight or open.
- Wary as ambient, recoverable mood: It is not punishment and not something the user must fix.
- Smooth day-phase curves: Local-clock lighting changes are authored and tested for DST/timezone changes without geolocation or duplicate drift.
- Active nightjar-like species: It gives one species different night activity while the flock is mostly drowsy. Why this exact species archetype: NOT RECOVERABLE FROM PLAN.
- Ambient rain and wind: Restrained weather creates variety without distress, prompts, thunder, or real-world dependency.
- Server-authored timed bird actions: Mood and traits shape behavior distributions while avoiding user-controllable placement commands.
- Bird-to-bird call responses and chorus: Natural delays, response caps, and refractory intervals keep social behavior bounded.

### 5. Session and interaction behavior

- Return-greeting as server-observed owner event: It lets birds notice returns promptly while preventing clients from claiming arbitrary absence.
- Weighted first greeter: Boldness, warmth, mood, and greeting recency make the greeting reflect individual birds and avoid repeats.
- Absence-shaped greeting distribution: Short absence favors glances; hours/days favor orientation, approach, or varied calls, without presenting "away for X days."
- Staggered second bird response: It prevents the whole flock from synchronizing on arrival.
- Continuous greeting parameters: Subtle variation and stable voice/gesture distributions make birds recognizable without canned clips.
- Retry deduplication and coalescing: Two tabs do not cause duplicate flock greetings.
- Listen-in local state machine: Pointer, keyboard focus, and handlers share one model so attention windows are not accidentally doubled.
- Offer menu with seed, song fragments, and still pool: Users choose gestures, not personality targets or placement commands.
- Per-bird offer cooldown and one active prop: The reducer prevents spam, simultaneous bypass, repeated reactions, drift farming, countdowns, prices, or "come back later" prompts.
- Mood/curiosity-driven offer reactions: Offers can be accepted, watched, delayed, ignored, or shaped by vocal frequency without guilt or failure labels.
- Network error handling for offers: Direct system language avoids pretending the bird refused.
- Settle: Ends only the originating presence/attention window, quiets mood and audio transiently, and marks an optional goodbye.
- Undo settle: Five-second undo reverses the active settle envelope and avoids turning the undo click into another action.
- Continued watching after settle: The local settled latch prevents incidental movement from resuming attention, while re-engagement after the window restores normal current-time lighting.
- Notebook prose templates: Authored naturalist text keeps private interactions inside the simulation boundary and avoids external language-model service use.
- Notebook candidate rarity: Novelty/evidence spacing keeps entries sparse and avoids timer promises.
- Notebook fact finalization: Only true, specific facts are persisted; later template changes do not rewrite history.
- Starter onboarding: Two starter IDs/seeds are transactionally created once so retries cannot duplicate birds.
- Later adoption schedule: Age thresholds create gradual growth over months without badges, countdowns, shopping catalogs, modals, or attention rewards.
- Additional species selection: Server selection avoids rarity and prefers new species until exhausted, then distinct individuals.

### 6. HTTP API and contracts

- `/v1` API versioning: NOT RECOVERABLE FROM PLAN.
- Mutation route safeguards: Validated content types, CSRF checks, authorization, bounded payloads, and idempotency prevent unsafe retries and unauthorized writes.
- Separate owner and visitor tokens: Keeps visitors out of owner interaction surfaces.
- Stable error codes and matter-of-fact messages: Errors guide retry or re-authentication without leaking private bird records.
- `POST /auth/magic-links`: Generic response and one-use challenge avoid account enumeration and scanner/prefetch issues.
- `POST /auth/magic-links/consume`: Atomic consumption and per-device cookie creation make replay/expiry safe and useful.
- `POST /auth/logout`: Revoking the session also terminates presence windows.
- `GET /account`, `PATCH /account/settings`: Settings expose timezone/audio/accessibility/notification consent without vectors or interaction statistics.
- `GET /account/sessions`, `DELETE /account/sessions/{id}`: Owners can manage device access, effective on next request.
- Email-change routes: Pending address only becomes authoritative after valid token consumption.
- Export route: Queues a consistent private export and reuses jobs for repeated idempotency keys.
- Deletion/recovery routes: Support thirty-day recovery with authenticated explicit confirmation.
- Adoption route: Finishes starter names once or accepts eligible age-based opportunity while server verifies species/capacity.
- Bird rename route: Version conflicts return refresh guidance instead of silent overwrite.
- Owner snapshot route: Authenticated render projection includes server time/revision/action horizon and supports ETag after auth.
- Owner events route: Small ordered batches return accepted/rejected/applied status, sequence, and optional committed result.
- Event-results route: Lets a timed-out client learn original outcomes without resubmitting under a new ID.
- Notebook route: Owner-only immutable cursor pagination preserves history.
- Invitation routes: Explicit owner action creates invites; list/revoke surfaces active permissions.
- Visit redeem route: Atomic token exchange starts the visit log.
- Visitor snapshot route: Checks invite/session/host status before every response, including conditional requests.
- Visit end route: Ends visitor session without creating an owner simulation event.
- Visit log route: Host-only transparency avoids badge/feed behavior.
- Event envelope schema: Credentials derive identity and payloads cannot assign vectors, moods, or desired reactions.
- Snapshot schema and allowlist: Rendering receives only what it needs and not trait vectors, private seeds, logs, ledgers, emails, cooldowns, or evidence.

### 6.1 Pull cadence, reconciliation, and outages

- Immediate and fifteen-second snapshot pulls: The client stays current on navigation, visible return, long gaps, render gaps, and after owner actions without treating polls as presence.
- Hidden-tab shutdown: Hidden tabs stop rendering, routine pulls, narration, and local audio scheduling while server ticks continue.
- Revision/action reconciliation: Out-of-order responses are discarded, ongoing actions preserved, and server-clock offset smoothed so snapshots do not visibly reset the scene.
- Canonical record with documented staleness: Devices converge without claiming physical simultaneity.
- Version conflicts for stale settings/name edits: Users get refresh guidance while semantic events stay ordered.
- Bounded in-memory pending command queue: Ambiguous network commands use stable IDs and results lookup without durable replay after browser reopening.
- Auth expiry handling: Attention closes, writes stop, private browser state clears, and the sign-in surface appears.
- Outage rendering: The client uses only the valid server-authored horizon, then a quiet known pose and matter-of-fact connection explanation; it never invents mood/call events.

### 7. Frontend scene and accessible surfaces

- Canvas 2D plus SVG/pose bootstrap: It gives first paint and then seamless replacement at the same action phase.
- Semantic DOM for controls/accessibility: Top bar, dialogs, notebook, narration, captions, and keyboard bird controls remain accessible.
- Bounded scene graph with `requestAnimationFrame`: The scene can animate without framework rerender each frame.
- Soft sky/foliage, perch zones, branches/leaves: These compose the quiet aviary scene. Specific visual motif rationale beyond naturalist aviary: NOT RECOVERABLE FROM PLAN.
- Offscreen static caches and compact shapes/atlases: These serve runtime budget and visual QA.
- Server-authored movement sampling: Visual liveness follows canonical action curves; large corrections sample the authoritative pose rather than replaying stale paths.
- Local leaf/feather drift and parallax ornaments: They add slow ambient texture while being explicitly non-semantic and unable to affect simulation or notebook facts.
- One-screen layout: The aviary stays one contained scene without body scrolling, panning, zooming, or drag-to-place.
- Responsive perch anchors: Reflow preserves birds, depth cues, uniform scale, and safe paths from phone widths to ultrawide.
- Invisible hit areas and visible focus treatment: Interaction is usable without permanent scene labels.
- Calm palette and limited copy: Mood is expressed by posture/call, not labels or icons.
- Fading top-bar treatment: Controls recede during stillness while discoverability, contrast, focus, and open menus remain protected.
- Roving tab stop and arrow traversal: Keyboard users can navigate birds in stable spatial order while preserving bird IDs through rearrangement.
- Dialog semantics and shortcut constraints: Settings, offers, and settle remain keyboard/screen-reader usable without seizing assistive-technology commands.
- Contrast/focus requirements: AA text, control contrast, forced colors, zoom, and composited caption backing keep surfaces readable.
- Shared narration lexicon: Narration summarizes the same canonical scene and active calls without a network roundtrip or hidden state exposure.
- Polite bounded live region: User events are announced promptly without interrupting constantly or speaking every frame/poll/focus movement.
- Caption placement and stacking: Calls remain readable and associated with birds, including overlaps and short calls.
- Visitor local accessibility controls: Visitors can adjust caption, narration, volume, and reduced motion without acquiring simulation interactions.
- Reduced-motion first paint: Motion-sensitive users do not briefly receive full motion before settings hydrate.
- Reduced-motion cross-fades and removed ornaments: The mode is a calm separate composition, not merely "no animation errors."

### 8. Procedural audio pipeline

- Species call grammars: Motif vocabularies create species identity through whistle arcs, paired notes, pulses, trills, breathy edges, rests, and alternatives.
- Immutable bird voice signature: Each bird remains recognizable through register, timbre, interval contour, rhythm tendencies, and variation seed.
- Mood/drift modulation inside signature: Expression changes without losing recognizability.
- Server call choice and client deterministic expansion: Behavior decisions stay canonical, while devices can reproduce the same call score locally.
- Bounded continuous variation and fingerprint rejection: Consecutive calls vary while avoiding unbounded randomness and exact repeats.
- Captions from final score: Caption text truthfully matches synthesized notes/envelopes and works even when muted/fallback.
- One AudioContext and bounded voice pool: Resource limits prevent underruns, leaks, and runaway nodes.
- WebAudio-only native synthesis: No recorded calls or downloaded loops preserves procedural identity and fallback constraints.
- Per-bird gain buses, chorus bus, limiter, pan, and low ambience: The mix remains restrained, mono-safe, and gives all identities room.
- Short scheduling lookahead: Calls are scheduled accurately without enqueuing uncancellable minutes of oscillator work.
- Ramp-out cleanup on hidden/cancelled/permission loss/logout: Audio state follows authorization, visibility, and schedule validity.
- Listen-in gain ramps: Focus lifts the target bird while keeping a nonzero floor for other birds, so surroundings are never hard-muted.
- Audio capability states: The product distinguishes not started, running, muted, autoplay suspended, unavailable, and interrupted so controls can be clear.
- Graceful silence and retry on gesture: Browser/audio failures show captions and settings control without blocking the aviary or bombarding prompts.
- Audio release listening gates: Real listening verifies naturalistic appeal and recognizability because oscillator correctness alone can still be mechanical or uncanny.

### 9. Identity, visits, export, and deletion

- 256-bit secrets and hash-only storage: Auth tokens and sessions remain opaque and not recoverable from storage.
- Same-origin POST token exchange: Mail scanners or prefetch GETs cannot consume magic links.
- No third-party resources and no-referrer token landing page: Token leakage is reduced during sign-in.
- Rate limits for mail requests: Abuse is bounded without turning limits into indefinite account lockout or reminders.
- Secure HttpOnly SameSite cookies and session lifetimes: Device sessions stay protected and revocable.
- Session revocation on every protected request: Cached snapshot access and attention leases respect current revocation.
- Recent authentication for sensitive actions: Email change, export, and deletion require fresh proof without adding passwords.
- Deliberate host action for invitations: Sharing is not public visibility and not created by default.
- One-time invitation redemption: Simultaneous opens cannot create multiple visits.
- Visitor render-only bundle: Visitors see the real ambient aviary without event writes or owner controls.
- Visit duration from snapshot requests: The visit log is service transparency, not drift event logging.
- Revocation checks and short freshness lease: Visitor access stops on next pull and cached host activity cannot continue indefinitely.
- Off-by-default visit-start email: It is a narrow transactional notification, deduplicated and consent-checked, without badges or live presence features.
- Consistent export snapshot: The export preserves birds, IDs, names, vectors under the exception, moods, notebook, settings, schema version, and time without mixing revisions.
- Encrypted expiring export object and download token: Export contents do not go to mail, public caches, jobs, or providers.
- Delete marker with exact thirty-day deadline: Access and jobs stop immediately, but recovery preserves bird continuity within the window.
- Hard erasure at deadline: All live account-linked records, grants, logs, jobs, blobs, cache entries, and expired recipient references are deleted.
- Account-scoped key destruction: Backups cannot defeat the deadline because restored ciphertext remains unrecoverable.
- Non-reversible cleanup evidence: The system can record completion without retaining account identifiers.

### 10. Privacy and operational observability

- Interaction events only for owner simulation: They cannot feed analytics, training, recommendations, session replay, or behavioral SDKs.
- Separate credentials/schemas/network access: Operational telemetry cannot read birds, vectors, notebook, visits, or events.
- Seven-day consumed-event retention: Raw simulation events are temporary account-private correctness recovery, not a permanent replay store.
- Indefinite notebook retention: Entries remain part of the account's readable history.
- Ninety-day visit-log detail retention: Host sharing transparency persists briefly while permissions remain visible until resolved.
- Allowed aggregate metrics: Request counts, latency/error histograms, coarse session duration, first-bird timing, frame timing, audio errors, tick runtime, and due-job delay support operations.
- Forbidden metric fields: IDs, emails, URLs, action/call IDs, interaction-reconstructing event names, vectors, bird count, mood, offers, listen duration, and notebook text are excluded to prevent behavioral analytics.
- Coarse buckets, sample rates, no analytics cookie: Client metrics stay aggregate and not user-tracking.
- Exception redaction and no screenshots/DOM recording: Operational convenience cannot leak private scene or token state.
- Privacy policy link and schema review: Retention and operational categories are published, and schema changes require allowlist review with rejected-field tests.

### 11. Performance budgets and verification

- Initial JavaScript cap and targets: Critical JS stays small enough for first-bird delivery; notebook/settings/visits/audio can lazy-load.
- First-bird critical payload target: HTML, compact projection, and starter poses must be small and free of blocking font, external image, settings chunk, or audio permission.
- Time to first visible bird under 500 ms: Real paint on a mid-tier mobile/4G reference is required because marks without a painted bird do not prove the experience.
- Greeting budget: The first noticing action must begin within one or two seconds, measured visibly/accessibly under normal, slow, and interrupted conditions.
- Snapshot size target: The render projection remains bounded and excludes notebook/history.
- Idle rendering budget: Long sessions with two and seven synthetic birds must remain smooth on older mid-range hardware.
- Audio budget: Voice/node counts, one context, ramps, cleanup, and soak runs prevent underruns and leaks.
- Client memory budget: Thirty-minute sessions must not show retained heap growth, detached nodes, contexts, buffers, or queues.
- Simulation latency budget: Minute cadence and tick alarms prove states advance while no client exists.
- Real mobile/reference profiles and geographies: Certification cannot rely only on warmed developer desktops.
- Browser support policy and feature detection: Supported browsers get the graceful-silence aviary when audio is unavailable, while unsupported browsers get concise upgrade copy.

### 12. Build sequence, test gates, and rollout

- Complete vertical increments: Delivery should not leave accessibility or durability for later.
- Contracts and expressive prototypes: Early work establishes private/public schemas, invariants, reducer harnesses, silhouettes, voices, tokens, motion, narration, captions, and product voice without production interaction collection.
- Durable two-bird slice: Identity, vectors, scheduling without clients, snapshots, and first-bird path are validated early across restart and devices.
- Attention and responsive behavior: Presence unions, expedited passes, filters, mood/time/weather, greetings, offers, settle, and state machines prove attention and responsiveness without single-session visible drift.
- Complete accessible aviary: Six species, three-to-seven-bird layouts, chorus, ramps, narration, captions, reduced motion, silence fallback, keyboard, screen reader, and performance all ship together.
- Quiet account depth: Notebook, naming/settings, age adoption, export/delete/recovery, visits, logs, and email consent are gated by lifecycle, visitor-isolation, privacy, and erasure tests.
- Release qualification: Browser/device matrix, geography probes, soak, migration/backup rehearsals, load tests, privacy audit, and editorial review must pass together.
- End-to-end reference scenario: A single scenario exercises account creation through recovery plus weeks/months synthetic continuation so system behavior remains integrated.
- Automated/experiential test mix: Property tests, seeded reducer/grammar tests, database integration, browser E2E, real-device/manual sound and accessibility review cover both invariants and charm.
- Synthetic internal bird-count ramp: Every species and duplicate-species identity is exercised before outside accounts.
- Recruited browser pilot: Complete v1 features are tested with operational metrics and direct qualitative feedback, not production behavior mining.
- Controlled cohort expansion: Account admission waits for first-bird, tick, persistence, leak, accessibility, and listening gates.
- Capacity ramp as quality-control ceiling: Temporary adoption ceilings never remove, hide, reset, or downgrade adopted birds.
- Feature flags and rollback constraints: Invitations, adoption opportunities, grammar/renderer revisions, and drift writes can be paused, but rollback cannot reseed or subtract personality.
- Release artifacts: Versioned tuning, migrations, calibration, matrices, performance, listening/reduced-motion notes, restore/erasure evidence, and runbook document that gates are real.

### 13. Risks and response

- Drift too fast or too faint: Bounded elapsed-time integration, synthetic fixtures, and blinded review allow tuning future influence rates without resetting vectors or collecting population metrics.
- Absence accidentally punishes a bird: Monotonic vector tests and separation of mood/expression remove desaturation, guilt wording, distress, and mute penalties.
- Double-counted presence or dropped drift: Interval union, watermarking, two clocks, idempotency, and transactional cursor protect against concurrent devices and crash/retry boundaries.
- Dead/canned arrival: Current pose, preserved action phase, server-authored greeting variation, and no welcome text/spinner keep arrival alive.
- Audio uncanniness or blurred identity: Grammar/timbre constraints, voice anchors, bounded chorus, nonzero mix, and listening tests prevent mechanical or unrecognizable calls.
- Accessibility becomes a state-list fallback: Shared semantic scene/score data plus manual review preserve product character in screen-reader and reduced-motion modes.
- First-bird budget fails on real networks: Private edge bootstrap, minimal critical bytes, and early cold-condition tests guide architecture fixes rather than fake starters or spinners.
- Unobserved tick cost grows excessively: Distributed due times, indexed batches, bounded O(birds) state, backlog measurement, and worker scaling prevent client-tick or resume-only simulation.
- Privacy leaks through operational convenience: Separate roles/schemas, unsafe-field rejection, token/body stripping, opaque job references, and sample payload reviews prevent leakage.
- Invitation grants excessive or stale access: Separate credentials/endpoints, per-pull authorization, freshness leases, atomic redemption/revocation, and no public caches/shared cookies bound access.
- Loss of identity during migration or rollback: Immutable species/voice references, IDs, vectors, filters, restore comparisons, retained assets, and no resetting protect continuity.
- Feature creep changes the product: New surfaces are reviewed against naturalist/system voice, no gamification, no user-frequency exposure, no distress, and no unsolicited notifications.

