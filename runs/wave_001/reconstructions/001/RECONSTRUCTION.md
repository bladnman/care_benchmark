## System-level intent

1. The aviary is a quiet, non-gamified presence system, not a reward loop. This shows up in the binding vocabulary and banned words: `bird`, `call`, `mood`, `personality`, `drift`, `presence`, `listenIn`, `offer`, `settle`, `notebook`, `visit`, `tick` are required, while `pet`, `creature`, `streak`, `achievement`, `score`, `level`, `badge`, and `toast` are banned. It also shows up in the invariant "No announcement surfaces" and in the spell-break checklist: "no spinner, no entry animation, no toast, no welcome text, no streak, no counter, no number for a trait." The risks call "Announcement creep" a violation of "the product's central register."

2. Personality and drift are hidden, server-owned, and monotonic. The plan says "The server simulation tick is the only writer of personality vectors and canonical aviary state," "Drift is monotonic toward expressive," and "Raw personality numbers never reach any product surface." The same intent appears in the data model, sync model, DB triggers, export decision, drift tests, and no "last-write-wins anywhere near personality."

3. The product should feel alive without becoming a Tamagotchi. The drift risk says "too fast -> Tamagotchi feel; too slow -> screensaver," and the drift function uses low-pass attention, headroom, daily saturation, and high-gain behavior mappings so change is "visible after three weeks" but not announced. Drift "never does" decrease on absence, mute, visitor activity, visit counts, or another account.

4. The architecture protects one canonical aviary while allowing many devices. The plan repeatedly grounds sync in "one canonical record, many readers," one event log, idempotent additive events, row locks, monotonic snapshot sequence, and "no client-to-client sync and no merge." The rule of thumb is "if two devices should see it the same way, it comes from the server"; if devices "may legitimately differ," it is client-local.

5. The scene should appear continuous from the first frame, with no loading theater. The request flow targets the "500 ms path," inlines `snapshot0`, renders an edge first-frame SVG, swaps to canvas with "no visual discontinuity," and says "No spinner exists in the codebase." The risk table says a visible SVG-to-canvas pop "is an entrance animation by another name."

6. The product voice is divided between naturalist surfaces and matter-of-fact system surfaces. The observer and narration use "naturalist voice": lowercase, present tense, no second person, no counts of user behavior. Settings, account, sign-in, sync errors, unsupported-browser, and accessibility settings use "matter-of-fact voice." The string-table lint enforces surface and voice tags.

7. Privacy is a product boundary, not only a security implementation. The telemetry invariants say per-account interaction state never enters telemetry, the account identifier is a synthetic UUID, and visitors are read-only. The observability section says "What we deliberately do not measure" includes per-account or per-bird telemetry, "average drift," and "most-used offer," because those are "the first step to a data product the privacy commitment forbids."

8. Accessibility ships as part of v1 and has equal status with the visual surface. The plan says "Accessibility ... ships in v1, not after" and "Accessibility is a designed surface here, built by the same people building the visual one, in the same sprints." Reduced motion is "a designed surface with its own visual regression baselines" and "a launch gate."

9. Audio must be procedural, recognizable, and calm rather than canned. The invariant says "No recorded audio anywhere"; the audio risks call audio "the affective spine" and say canned or synthetic-sounding calls "break the spell." The call grammar, per-bird signature, bounded variation, recognizability separation, listening reviews, and listening study all support this intent.

10. Cost and performance are treated as launch-shaping constraints while preserving exact simulation semantics. The plan uses hot/cold tiers, catch-up, and on-read repair because the aviary is "defined at every logical tick whether or not anyone is connected" while cost stays "proportional to live use." Budgets for first bird, heap, frame time, tick compute, and snapshot size are enforced in CI and production.

11. The notebook observes birds, not the user. The invariant says the notebook is "read-only, sparse, and observes birds, never the user's behavior." The observer may use presence as a condition but "never as content," and the voice linter rejects generated entries that mention visits, days, or the user.

12. Social access is intentionally narrow and read-only. The scope includes "per-invite read-only visits by email link" and excludes public discovery, profiles, follows, feeds, comments, chat, avatars, and co-presence. Visitor sessions "cannot call `/aviary/*`," do not produce presence or interaction events, and never start the presence monitor.

13. Tunability is versioned and operational, not an experiment product. Drift constants, arrival schedule, weather rates, observer budget, greeting rate limit, and `max_birds` live in versioned server config with a change log. Flags are for operational rollout, while the plan says there is "no per-account experiment assignment dimension in telemetry."

## Per-feature whys

### Reading guide and the twelve load-bearing invariants

- Binding PRD vocabulary and banned identifiers: to make the PRD vocabulary binding "in code as well as in copy" and keep gamification, announcement, and wrong-concept terms out of product code and product strings.

- Server simulation tick as only writer of personality vectors and canonical aviary state: to make canonical state and personality server-owned; clients only append interaction events.

- Monotonic drift toward expressive: to ensure no code path decreases a trait.

- Presence requiring visible, focused, and active at the same time: to prevent partial conditions from counting as presence; the risk table frames presence inflation as corrupting drift population-wide.

- No raw personality numbers in product surfaces or client snapshots: to keep users from inspecting trait numbers; the snapshot carries phenotype projection instead.

- No recorded audio: to ensure calls are synthesized client-side from the call grammar; fallback is silence with captions.

- No announcement surfaces: to preserve the central register; no toast, banner, modal, badge, streak, visit-frequency display, or friend-visited notice.

- No per-account interaction state in telemetry: to preserve the telemetry boundary and avoid aggregate pipelines over user behavior.

- Synthetic UUID as account identifier and encrypted email stored once: to keep email out of logs, metrics, queue keys, URLs, and normal identifiers.

- Stable bird UUID: to make bird identity survive rename, sync, migration, and species-pool changes.

- Read-only sparse notebook: to observe birds, never the user's behavior.

- Read-only visitors: to ensure visits never produce presence or interaction events.

- Accessibility in v1: to make narration, reduced-motion renderer, captions, and keyboard launch gates rather than later additions.

### Scope

- Web app for last two major browser versions, unsupported browsers matter-of-fact page: NOT RECOVERABLE FROM PLAN

- Single-user accounts, magic-link sign-in, revocable sessions, email change, JSON export, soft delete then hard delete: NOT RECOVERABLE FROM PLAN

- One canonical aviary per account with server-side simulation tick: to make multi-device consistency "a property of the architecture."

- Two starter birds chosen from six species and named by the user: NOT RECOVERABLE FROM PLAN

- Seven-bird cap and new birds by aviary age: the rollout says age-gated arrivals make the birds-per-aviary ramp "naturally slow," and the cap lets recognizability problems be held at a lower count "without touching any bird that already exists."

- Hidden personality vector, monotonic drift, mood persistence, bird-to-bird interaction, procedural calls, chorus: to make birds visibly expressive while keeping raw personality numbers hidden and generated behavior deterministic.

- One horizontal scene, perch zones, local-time day/night, rare weather, micro-motion, sparse fading top bar, no chrome inside scene, first-frame motion, quiet-field loading, adoption-only empty state: the plan grounds this in the scene staying "chrome-free," the bar staying sparse, no spinner, and the aviary reading as already alive.

- Return-greeting, presence accounting, listen-in, offers, settle, field notebook: NOT RECOVERABLE FROM PLAN as a scope list; individual flow rationales appear below.

- Social read-only visits by email link with 30-day expiry, revocation, visit log, opt-in email off by default: to keep social narrow, read-only, and non-pushy; the only exception to no friend-visited notice is an opt-in email toggle.

- Naturalist screen-reader narration, reduced motion, procedural captions, WCAG AA, keyboard navigation: to make accessibility a designed v1 surface with launch gates.

- Initial bundle and first-bird, frame, memory, synthetic monitoring, aggregate-only RUM performance targets: to keep first bird visible under the PRD cap, preserve long sessions, and monitor only aggregate behavior.

- Excluded native apps, payments, shared accounts, custom scenes, public social surfaces, gamification, Tamagotchi mechanics, recorded audio, catalog selection, arranged perches, personality numbers in UI, native-client data model accommodations: to avoid scaffolding for out-of-v1 concepts and preserve the quiet non-gamified product register.

- Deferred SSO/password auth, raised cap, push, more species, localization: NOT RECOVERABLE FROM PLAN beyond "allowed later, not designed for now."

### Architecture

- One TypeScript monorepo: to share call grammar, prose grammar, bird geometry, PRNG, and state types across client, API, simulation worker, and edge, making captions match calls, narration match scene, first frame match canvas takeover, and simulation deterministic across replay.

- Node 22 LTS and Fastify: NOT RECOVERABLE FROM PLAN

- Postgres as single system of record: to avoid a second datastore for canonical state.

- Redis only for rate limiting and short-lived locks: to ensure nothing canonical lives there.

- CDN edge compute and edge key-value store: to hold the HTML shell, static assets, and last snapshot for first paint.

- Object storage for export artifacts: NOT RECOVERABLE FROM PLAN

- Transactional email provider with DKIM/SPF/DMARC and failover: because magic link is "the only way in."

- Preact, Canvas 2D, AudioWorklet, no animation library, no game engine, no audio middleware: Canvas 2D keeps the critical bundle small and avoids WebGL context loss handling; procedural audio avoids recorded assets.

- Simulation service responsibilities: to centralize personality drift, mood transitions, perch choice, weather, calls, greetings, arrivals, notebook, canonical state writes, and edge publish in the tick worker.

- API service responsibilities: to handle auth, sessions, events, offer resolution, state reads, settings, visits, export, and deletion without writing traits or normal aviary state outside the shared offer path.

- Edge responsibilities: to serve session lookup, snapshot0, first-frame render, static assets, and unsupported-browser page without writes or state decisions.

- Client responsibilities: to own rendering, interpolation, micro-motion, ornaments, audio, presence detection, narration, captions, and local UI state without owning personality, mood, drift, or tick.

- Mailer responsibilities: to keep outbound email in matter-of-fact voice and away from bird state.

- Server behavioral intent versus client motion, sound, and prose: to keep cross-device sameness server-derived and legitimate device differences client-local.

- Session-start request flow: to get first bird visible fast with edge `snapshot0`, an inline SVG first frame, canvas takeover on the same frame, and no visual discontinuity.

- Quiet field when no snapshot exists: to avoid a spinner and still show a soft sky until API state arrives.

### Data model

- Timestamps as `timestamptz`: NOT RECOVERABLE FROM PLAN

- UUIDv7 generated server-side: NOT RECOVERABLE FROM PLAN

- Fixed-point `int4` trait values: so drift arithmetic is integer-exact and replay is bit-identical across Node versions and CPUs.

- Accounts table using `id` as the only identifier elsewhere: to keep the account identifier synthetic and avoid email as identifier.

- Account email ciphertext and blind index: to store email once, encrypted, with lookup only.

- Magic-link token hash and request IP hash: to support hashed tokens and rate limiting without plaintext token storage.

- Device sessions with hashed tokens, UA family, revoke state, and sliding expiry: to support per-device revocable sessions and session list display.

- Aviary row with species seed, cap, tick index, hot state, consumed event sequence, and settled flag: NOT RECOVERABLE FROM PLAN

- Birds table with immutable id, species, signature seed, traits, attention, mood, perch, cooldown, and unnamed newcomer state: to hold durable per-bird truths as columns so constraints and triggers can guard them.

- `aviary_state.state` as canonical JSONB regenerated by tick: to be the canonical record while `birds` columns enforce durable truths.

- Append-only `interaction_events`: to let clients append what the user did while the tick consumes ordered events; hard deletion uses a dedicated role.

- Monthly event-log partitions dropped after consumption and notebook need: NOT RECOVERABLE FROM PLAN

- Notebook entries with hidden structured facts: to support dedup while never showing the facts.

- Visit invites, visitor sessions, and visit log: to support invite expiry, revocation, active sessions, and approximate visit duration display.

- Runtime roles and grants: to structurally enforce which service can mutate events, traits, aviary state, account state, visits, and purges.

- Trait monotonic trigger: to reject trait decreases except reviewed migration under `sim_migrator`.

- Bird id immutability trigger: to preserve stable identity through rename, sync, migration, and species-pool changes.

- Bird-count check against `max_birds`: to enforce the cap.

- Trait range check: NOT RECOVERABLE FROM PLAN

- Edge key-value `session:`, `visit:`, and `snap:` entries: to support first-paint and edge validation while keeping edge KV out of analytics.

### API surface

- JSON over HTTPS under `/v1`: NOT RECOVERABLE FROM PLAN

- Device-session and visitor-session cookies: NOT RECOVERABLE FROM PLAN

- Error body `{code, message}` with matter-of-fact display text: to make errors ready to display in the system voice.

- Magic-link request returning 202 always: to prevent account enumeration.

- Magic-link consume creating account on first sign-in and redirecting to `/` or `/adopt`: to make magic link the sign-in door and route new accounts until two birds exist.

- Sign-out deleting edge `session:` key: to revoke current session at edge and API.

- Account endpoint returning settings, sessions, deletion state, and email plaintext only on the account page: to avoid plaintext email outside that surface.

- Account settings patch: NOT RECOVERABLE FROM PLAN

- Email change verification to new address while old continues until consumed: NOT RECOVERABLE FROM PLAN

- Device-session revocation: NOT RECOVERABLE FROM PLAN

- Export enqueue, one per day, emailed signed link: to deliver data portability without exposing raw traits in product surfaces.

- Soft delete and undelete: to allow "I changed my mind" within the window before hard delete.

- State endpoint running catch-up and recording session starts on return: to keep snapshots current and mark the aviary hot.

- Event batch endpoint idempotent by `event_id`: to make retries safe.

- Offer endpoint synchronous resolution: because offers need a reaction within a second, not at the next tick.

- Bird rename endpoint clearing `arrived_unnamed`: NOT RECOVERABLE FROM PLAN

- Notebook pagination: NOT RECOVERABLE FROM PLAN

- Aviary config endpoint cached one hour: to deliver calibration values like presence window and heartbeat interval.

- `sendBeacon`-friendly `text/plain` event endpoint: so `presence.end` can be delivered on unload.

- Snapshot as the only state the client renders from: to keep client rendering from a trait-free phenotype projection and server-made decisions.

- Snapshot omits raw traits and uses `temperament`, `palette`, geometry jitter, call signature seed, and server decisions: to let client render personality-shaped behavior without receiving trait values.

- Visitor invite creation limits: NOT RECOVERABLE FROM PLAN

- Visit invite list with decrypted email on demand and approximate duration: to support host visit management without broad social surfaces.

- Visit revoke endpoint: to immediately revoke links and end visitor sessions.

- Visitor page validating edge or API key and matter-of-fact expired/revoked page: to show "This visit is no longer available" when access ends.

- Visitor state omitting greeting candidates, offer availability, and absence timestamp: to remove host affordances and presence-related state for visitors.

### Simulation engine design

- Pure engine function `tick(state, inputs, config, rng) -> state'`: to make simulation deterministic and wrapped by persistence outside the pure library.

- API limited to `resolveOffer` and read-only `rankGreeting`: to keep engine logic from running outside ticks except the explicitly synchronous offer path.

- Logical 60-second tick: NOT RECOVERABLE FROM PLAN

- Counter-based PRNG seeded by aviary, tick, purpose, and bird: to remove `Math.random` and support deterministic replay.

- Fixed-point trait and mood math: to avoid float divergence in feedback state.

- Catch-up equivalence tests: so hot, cold, and on-read execution produce bit-identical results users cannot tell apart.

- Hot/cold/on-read cadence tiers: to keep cost proportional to live use while preserving the aviary's state at every logical tick.

- Returning last snapshot and enqueueing catch-up after large lag: to avoid blocking the API after sweeper outage.

- Tick algorithm consuming events under row lock and writing state transactionally: to avoid skipped or double-consumed events and keep state/event sequence consistent.

- Publish snapshot outside transaction, at least once: NOT RECOVERABLE FROM PLAN

- Low-pass attention with one-day time constant: to integrate attention inputs gradually.

- Drift proportional to attention and remaining headroom: so drift slows near ceiling and remains non-negative.

- Daily saturation after about 30 minutes: to prevent an eight-hour session from producing a week's drift in a day.

- Regular-profile drift calibration: to make change measurable after one week and visible after three weeks without large movement during a single session.

- High-gain behavior mapping around mid-range traits: so +0.15 is felt without being announced.

- No drift decrease on absence, mute, visitor activity, visit counts, or another account: to preserve no-punishment and privacy boundaries.

- Starter trait seeds with boldness contrast: to guarantee a "bolder bird greets first, warier bird later" contrast from day one while leaving room for change.

- Mood pressure model: to combine local hour, species, personality, recent events, weather, other birds, and inertia into persistent mood transitions.

- `recent` decays with two-hour half-life and zeroes at local dawn: to provide the "daily-ish reset."

- Nightjar night-active curve: so night is not a dead state.

- Persisted mood evolving in absence: so the bird a user returns to is "never a default."

- Perch-choice scoring: to make front-perch time a legible drift signal, with boldness gain "large on purpose."

- Seven birds across nine slots: to leave room for movement.

- Call scheduling horizon for next 120 seconds: so the client never runs dry.

- Solo call rate shaped by species, vocal trait, mood, local hour, and weather: NOT RECOVERABLE FROM PLAN

- Bird responses based on warmth and mood: NOT RECOVERABLE FROM PLAN

- Chorus merging: to produce several unlocked voices rather than one instrument when eligible birds call near each other.

- Alarm calls feeding mood contagion: NOT RECOVERABLE FROM PLAN

- `plan_seed` expansion on clients: so the same seed on two devices yields the same call.

- Local schedule extension during network stall: to keep calls going until the next successful pull and re-sync at the next boundary.

- Weather as tick-driven `none | rain | wind`: to modulate mood, call rates, rendering, and sound; no thunder, snow, or storms because the config schema has no room for them.

- Arrivals keyed to aviary age only and dawn timing: arrivals happen "at dawn, when the aviary is quiet."

- Arrival jitter: so arrivals are "not on the nose."

- Newcomer as a real bird from the first tick: so it has stable id, seeded traits, active drift, and nothing is lost if unnamed.

- Newcomer back perch and no announcement: because "the bird being there is the notice."

- Species choice for arrivals by call separation: to keep call signatures separable.

- No decline in v1 and pause future arrivals setting: to preserve "birds that arrived, not birds the user chose" and minimal chrome.

- Observer fact extraction: to write about novel specific bird facts such as greeting order, first front perch, rain, night call, newcomer, pool, quiet stretch, chorus, and rename.

- Observer sparsity budget: to keep regular users at one entry every 2-4 days and very active users at the same cadence because the budget is time-based, not interaction-based.

- Landmark override lane: to allow newcomer arrival, first-ever chorus, or first front-perch by the shy bird to spend into a small negative balance.

- Observer prose realization: to produce naturalist voice with names, current detail, varied phrasing, no second person, and no counts of user behavior.

- Observer no visitor/session/presence totals as subject: to keep the notebook observing birds, never the user's behavior.

- Synchronous offer resolution under row lock: because offers need a reaction within a second and must serialize with ticks.

- Per-bird offer cooldown: to shape reactions and prevent a cooled-down bird from accepting again immediately.

- One active offer per kind: to keep the tray from being "a button to mash."

- Tick converting offer acceptances into drift later: to keep trait writes in the tick path.

### Sync model

- No client-to-client sync and no merge: because every device reads `aviary_state` and appends to one event log.

- Single writer for durable state: to keep state writes correct rather than nominal.

- Additive idempotent events: to make retries safe and ensure events carry no absolute state.

- In-order consumption with `last_consumed_event_seq`: so a crash rolls back both event consumption and state write; no event is consumed twice or skipped.

- Monotonic snapshot sequence: so clients discard stale out-of-order responses on flaky networks.

- No personality field sent by device: to keep last-write-wins away from personality.

- Canonical versus device-local state split: to keep traits, mood, perch, weather, calls, offers, cooldowns, newcomer, notebook, and settled lighting server-side while listen-in, top-bar opacity, caption cache, narration timers, and undo timer stay local.

- Multi-device presence cap: to make two devices or two tabs count as one presence stream.

- Pull on session return, resume after long render gap, visible keepalive, hidden-tab stop, and beacon flush: to align polling with the tick cadence, avoid hidden rendering/audio, and limit lost presence to at most 30 seconds.

- Server timestamping and heartbeat caps: to prevent client clocks from influencing drift.

- API-down behavior rendering last snapshot and buffering events: to avoid an error surface unless needed while the aviary keeps rendering.

- Dropping oldest presence heartbeats first when event buffer is full: NOT RECOVERABLE FROM PLAN

- Session expired sign-in surface: to use matter-of-fact voice for system state.

- Edge stale snapshot after revoke accepted briefly: because API 401 takes precedence after KV convergence.

### Frontend rendering pipeline

- Canvas 2D scene: because with seven procedural birds, parallax layers, and a small particle pool it hits 60 fps with headroom, avoids WebGL context loss handling, and keeps the critical bundle small.

- Offscreen canvas layers and static re-rasterization only on palette change or resize: to control frame cost.

- DPR capped at 2: NOT RECOVERABLE FROM PLAN

- Scene logical space mapped with contain: NOT RECOVERABLE FROM PLAN

- Frame loop stopping when hidden and re-seating after sleep: because a bird cannot be mid-flight after a laptop wakes.

- Parametric vector rigs instead of sprite sheets: to share geometry with edge SVG render and drive birds from palette and pose parameters.

- Procedural poses with local-time lighting: NOT RECOVERABLE FROM PLAN

- Motion controller from mood, temperament, and activity: to turn snapshots into continuous pose changes.

- Breathing never stops: to keep motion alive.

- Activity generators by mood: to make mood visible in preen, scan, tilt, shuffle, fluff, rest, watch, posture, and gaze behavior.

- Reactive hooks to calls and ornaments: to let birds respond to nearby calls and passing leaves according to curiosity.

- 1/f noise in generators: so no two preens are identical.

- Snapshot activity phase on boot: so the first frame shows the bird mid-action.

- Perch-change transitions: NOT RECOVERABLE FROM PLAN

- Mood-change blends: to avoid a pop.

- Greeting pose scripts: NOT RECOVERABLE FROM PLAN

- Settle visual/audio transition and undo reversal: to make settle reversible within the undo window.

- Adoption fly-in only once per account: to be the only entrance animation in the product.

- Continuous day/night palette by local clock: NOT RECOVERABLE FROM PLAN

- Settled overriding to dusk key: NOT RECOVERABLE FROM PLAN

- Particle pool reused for leaves, feather, rain, wind gusts: to avoid allocation per spawn.

- No pointer parallax: NOT RECOVERABLE FROM PLAN

- Rain darkening sky and rippling pool offer: NOT RECOVERABLE FROM PLAN

- Responsive perch anchors: so seven birds remain fully visible with no cropping.

- Resize short hop: NOT RECOVERABLE FROM PLAN

- Top bar with five items: because two files require settle in the top bar, the scene must stay chrome-free, and the bar stays sparse and fades.

- Icons with visually hidden labels and no tooltips: NOT RECOVERABLE FROM PLAN

- Top bar fade and focus behavior: to stay sparse visually while remaining in the accessibility tree and fully visible with keyboard focus.

- Transparent DOM bird buttons over canvas: to provide naturalist accessible names and keyboard focus targets while focus ring is drawn by renderer.

- Side sheets over the scene without pausing it: to keep notebook, offer tray, and settings from stopping scene/audio.

- Offer tray unavailable kinds reduced opacity with no countdown: to avoid cooldown displays and button-mashing cues.

- Reduced-motion renderer: to be a designed surface sharing scene graph and geometry where mood still shows without normal motion.

- Reduced-motion cross-fades instead of flight arcs: to remove motion while preserving state changes.

- Reduced-motion ornaments off and rain static: NOT RECOVERABLE FROM PLAN

- Reduced-motion launch-gate review and baselines: because the mode is its own register, not an afterthought.

### Audio pipeline

- AudioWorklet synth graph with fixed voices, per-bird buses, ambient bus, runtime reverb, master gain, compressor: to synthesize calls and weather without assets while controlling mix.

- Per-bird pan and mix bias by perch zone: NOT RECOVERABLE FROM PLAN

- Main thread sends compact call plans and avoids per-call allocation: to support performance and memory rules.

- Motif-based call grammar: to realize species calls from parametric motifs rather than recorded audio.

- Per-bird signature from `signature_seed`: so the signature never changes and a bird stays recognizable.

- Mood-shaped realization: to make calls reflect wary, content, curious, drowsy, alert, and resting states.

- Seeded micro-variation bounded by signature: so no two calls are identical while a bird stays recognizable through mood and drift.

- Realization uniqueness and signature envelope tests: to enforce variation and recognizability.

- Species/signature separation at adoption and arrival: to maximize separation and reject candidates too close to existing birds.

- Listening study before cap raised: to validate that listeners identify calling bird at 80% before moving beyond five birds.

- Listen-in mixing: to make the focused bird nearer and drier without muting other birds; focus changes cross-ramp without silence.

- Settle mixing: to quiet master audio and halve local extension call rate, then restore on undo.

- Time-of-day volume curve: NOT RECOVERABLE FROM PLAN

- Hidden-tab fade and scheduling stop: to avoid audio continuing while hidden.

- Calm global loudness target: to keep the aviary at a calm level.

- Mute not a drift input: no-punishment principle; muting for a meeting must not count against anyone.

- Procedural wind and rain sound: to keep ambient/weather audio procedural only and quiet under calls.

- Autoplay fallback to silence with captions: because browser policy can block audio and no prompt is consistent with no announcements.

- First gesture resumes audio with fade-in: NOT RECOVERABLE FROM PLAN

- Main-thread fallback synth: to preserve the same grammar when AudioWorklet fails, without assets.

- Captions on when AudioContext unavailable: to provide graceful silence and audio-off parity.

- iOS silent switch note in settings: because hardware silent switch mutes WebAudio and cannot be detected.

- Fixed audio pools and no allocation after boot: to prevent memory growth and audio graph leaks.

### Interaction flows

- Return-greeting trigger on session start or visibility return: to greet through birds, with no textual welcome or absence counter.

- Greeting rate limit per device: to prevent tab-switch churn from turning into a "twitchy bird."

- Absence buckets: NOT RECOVERABLE FROM PLAN

- Greeter ranking by boldness, warmth, mood, and noise: to make bolder/warmer birds more likely while allowing the shyer bird to greet first on some days, making greeting-order notebook facts real.

- Resting birds not greeting: so a user can see a sleeping aviary.

- Greeting forms and stagger limit: to avoid more than two birds greeting or greeting in unison.

- Reporting greeting in `session.start`: so the observer can use it for greeting-order facts.

- No textual welcome or gone-days text: to leave "nothing but the bird."

- Presence monitor single module: to make visible, focused, and active checks centralized and tested.

- Four-minute activity window: calibrated long because watching without moving is the product.

- Presence heartbeats and end flush: to account for presence while bounding lost presence.

- Settle stopping heartbeats until re-engagement: to close the presence window cleanly.

- Visitor build never instantiates presence monitor: so visitors produce no presence or drift.

- Listen-in pointer and keyboard controls: NOT RECOVERABLE FROM PLAN

- Listen-in client-local immediate mix: because it is audio focus for the person listening on that device; syncing would surprise another device.

- Listen-in events and motion bias: to feed listen-in seconds to observer/drift inputs and make the focused bird tilt toward viewer.

- Offers from top bar only: NOT RECOVERABLE FROM PLAN

- Offers not by clicking a bird: NOT RECOVERABLE FROM PLAN

- Offer reaction rendering: to show offer objects and bird reactions at absolute times from the server plan.

- Unavailable offers no countdown or cooldown display: to avoid countdown/cooldown surfaces.

- Offer reactions narrated and captioned: to keep accessibility parity.

- Settle top bar item and keyboard shortcut: NOT RECOVERABLE FROM PLAN

- Settle emits event, ramps palette/audio, arms undo, stops presence: to make settle a goodbye and close presence.

- Canonical settle cleared by next session or re-engagement: because opening the aviary anywhere is a return.

- Field notebook side sheet newest first, cursor-paginated, virtualized: to handle many entries without memory growth.

- Notebook no edit, delete, annotate, share, or export from surface: because the notebook is read-only; account export includes entries separately.

- Notebook opening does not pause scene/audio: to keep the aviary continuous.

- Adoption line "two birds have arrived" and seeded name suggestions: NOT RECOVERABLE FROM PLAN

- Adoption opens on quiet field then birds fly in: to make the fly-in the only entrance animation and run once per account.

- Names editable later in settings: NOT RECOVERABLE FROM PLAN

- Host visits invite/list/revoke/log/toggle: to manage read-only email-link visits with immediate revocation and opt-in notification.

- Visit notification email off by default, one line, at most once per visitor per day: because the product never pushes and the social file allows only an opt-in toggle.

- Visitor shell with no offer, settle, notebook, or account items: to make visits read-only.

- Visitor top bar only accessibility settings and local preferences: to allow captions, reduced motion, and volume without account access.

- Visitor listen-in disabled but keyboard focus works for narration: because visitor cannot interact but should still get narration.

### Accessibility surfaces

- Accessibility built in same sprints by same people: to make it a designed surface with launch gates.

- Single polite narration log and aviary role labeling: NOT RECOVERABLE FROM PLAN

- Client-side narration from snapshot plus local scene state: so voice is continuous with notebook while reflecting current local scene details.

- Narration cadence with skipped repeats: so narration is not a state list and nobody gets a firehose.

- Narration cadence setting: so screen-reader users who want less get less.

- Visitor narration: to give visitor mode the same accessibility surface.

- Reduced-motion activation by system preference or setting: to follow system default and explicit user choice.

- Reduced-motion designer sign-off: because it must be its own aesthetic before beta.

- Captions default off unless audio unavailable or blocked: to avoid unnecessary captions while preserving fallback parity.

- Caption text from realized call plan: so captions match what played.

- Captions anchored near calling bird with AA backdrop: to make them legible against any sky key.

- Captions `aria-hidden`: to avoid double-speaking because narration covers screen readers.

- Keyboard tab order and scene group navigation: to provide full keyboard navigation through top bar and birds.

- Shortcuts documented in accessibility settings: NOT RECOVERABLE FROM PLAN

- Panels trap and restore focus: NOT RECOVERABLE FROM PLAN

- Renderer-drawn focus indicator with dual-tone outline: to stay visible against every sky key and at night.

- Focus indicator never suppressed for pointer users: NOT RECOVERABLE FROM PLAN

- Top bar fully visible with focus: so fade never hides a focused control.

- WCAG AA for all user copy: to enforce readable copy across surfaces.

- Naturalist voice versus matter-of-fact voice by surface: to keep voice consistent with surface role.

- String-table lint with surface tags: to reject mismatches.

- Audio-off parity: so sound-off/captions-on users receive the same call events, chorus timing, greeting, and narration, and no surface is the "full" one.

### Performance budgets and observability

- CI and production budgets: to enforce first-bird, frame, heap, snapshot, tick, and lag performance rather than leave them aspirational.

- Critical chunking and deferred chunks: to keep the first-paint path small while loading audio/chrome after first paint and heavier surfaces on demand.

- Server RED metrics, tick metrics, edge publish, auth/email/export/deletion/DB metrics: NOT RECOVERABLE FROM PLAN

- Client RUM sampled without identity or bird fields: to monitor first bird, takeover, frame time, pull latency, audio outcome, errors, and feature usage without account/session identifiers.

- Presence-duty histogram as release canary: because a sudden population shift indicates a presence monitor bug.

- Synthetic fleet: to measure first-bird time, audio init, and greeting from five regions every ten minutes.

- Drift canary synthetic accounts: to answer calibration questions with synthetic profiles, not user data.

- Deliberately not measuring per-account or per-bird telemetry: to avoid the first step toward a forbidden data product.

- OpenTelemetry allowlist: to drop unknown keys and enforce allowed metric attributes.

- Log redaction layer: to strip event payloads, bird fields, email, name, trait, and mood keys.

- Metrics store separated from simulation database: to prevent analytics credentials from reading simulation data.

- Privacy review for sim-table dashboards and no BI connector: to guard against helpful dashboards that leak the boundary.

- Initial alarms: to catch tick latency, backlog, event lag, API errors, edge failures, magic-link deliverability, first-bird regressions, audio unavailability jumps, and heap soak failure.

### Accounts, privacy, and security engineering

- Magic-link token hashing, expiry, single use, and blind-index binding: to make sign-in secure without storing plaintext tokens.

- Magic-link consume signs in the browser that opened the link: because the PRD says it "signs them in on that browser."

- No cross-device polling in v1: NOT RECOVERABLE FROM PLAN

- Generic magic-link 202 and rate limits: to prevent account enumeration.

- Adoption flag until two birds exist: to route new accounts to `/adopt`.

- Opaque hashed session tokens and session list metadata: to support secure per-device sessions.

- One-device sign-out not affecting others: NOT RECOVERABLE FROM PLAN

- Encrypted email plus blind index: to prevent PII leakage and make it hard to retrofit email identifiers.

- Lint forbidding decryption helper imports and email in logs/metrics/keys: to keep email localized to auth/account display.

- Export JSON including engine values: to satisfy the data-portability artifact while no product surface renders personality values.

- Export signed URL emailed to verified address: NOT RECOVERABLE FROM PLAN

- Soft-delete window with "I changed my mind": to allow recovery and show matter-of-fact system status.

- Ticks continue cold during deletion window: so a recovered aviary is continuous.

- Purger role deleting account data, edge keys, mail, exports, and tombstone: to complete hard deletion and re-purge after restored backup.

- Backup retention max 35 days linked from privacy policy: NOT RECOVERABLE FROM PLAN

- Visitor tokens hashed, 30-day expiry, every pull re-validates revoke/expiry: to make revocation effective and sessions bounded.

- Visitor sessions with no account and no write scopes: to keep visitors read-only.

- Threat model centered on corrupting a user's own drift: to bound what any client can do through idempotent events, per-tick caps, and server-side cooldowns.

- CSP, no third-party scripts, SRI, HSTS, dependency audit: NOT RECOVERABLE FROM PLAN

### Testing, calibration harness, and CI gates

- Engine drift tests: to prove non-negativity, monotonicity, visitor-only zero drift, caps, saturation, determinism, and catch-up equivalence.

- Mood tests: to prove persistence, dawn reset, contagion, nightjar night activity, and no resting greeter.

- Perch tests: to prove no slot overflow, hysteresis, and valid seven-bird assignment.

- Call tests: to prove schedule horizon, chorus eligibility, uniqueness/signature envelope, and captions for every motif.

- Observer tests: to prove sparsity bounds, landmark lane, and no user-behavior facts.

- API role/grant tests: to prove API cannot update traits or `aviary_state` and event table has no update/delete path.

- Trigger tests: to prove trait decrease rejection and immutable bird id.

- Concurrency tests: to cover simultaneous offers, offer during tick, catch-up during pull, duplicate batches, and out-of-order snapshots.

- Revocation test: to prove visitor pull after revoke returns 410 within one pull.

- Drift calibration harness profiles: to assert trait curves for regular, light, binge, absent, and visitor-only usage.

- Observable behavior curves in harness: so "visible after three weeks" is asserted on behavior, not only numbers.

- Versioned calibration changes with harness report: to control drift constants and never rewrite existing traits.

- PresenceMonitor client matrix: to enforce the three-condition presence definition.

- Renderer golden images and responsive no-crop tests: to ensure species/mood/zone visuals and seven-bird layouts work across target viewports.

- SVG-first-frame versus canvas pixel diff: to prevent takeover discontinuity.

- Audio offline tests and fallback tests: to ensure call sample generation, uniqueness, underrun behavior, fallback path, and captions under suspended context.

- Accessibility axe, keyboard, narration, and caption tests: to keep accessible surfaces functioning and free of banned tokens.

- Voice linter for naturalist and matter-of-fact surfaces: to enforce lowercase naturalist voice, no `you`/`your`, no banned tokens, no trait names, and tagged strings.

- Nightly soak and per-PR accelerated soak: to catch heap slope, worklet underruns, frame p95, bundle, Lighthouse, and engine benchmark regressions.

- Manual QA with screen readers, designer review, listening review, and spell-break walkthrough: to catch announcement, spinner, canned cue, and gamification residue that automated tests might miss.

### Rollout

- Internal alpha with two birds, arrivals off, visits off, notebook on: to meet invariant, first-bird, frame, screen-reader, reduced-motion, and spell-break criteria before broader release.

- Closed beta with three birds, arrivals and visits on: to validate tick/latency alarms, drift canary bands, privacy lint, email deliverability, and memory soak.

- Open launch with public sign-up and three birds: to proceed once synthetic fleet is green and on-call runbooks are exercised.

- Ramp to five and seven birds: to gate higher bird counts on listening study identification and chorus mix review.

- Slow age-gated arrivals and config cap: to hold recognizability problems at lower count without touching existing birds.

- Versioned tunables and operational flags: to manage rollout without per-account experiment telemetry.

- Listening study with familiarization and 80% pass threshold: to empirically validate bird-call recognizability before each cap ramp.

- Day-one instrumentation: to have production metrics, canaries, synthetic fleet, alarms, budgets, soak, and presence-duty release canary ready immediately.

- Runbooks before beta: to prepare tick backlog recovery, edge staleness, email failover, drift rollback, vector corruption response, and purge failures.

### Risks

- Drift calibration risk and mitigation: because the central promise sits in a narrow band between "Tamagotchi feel" and "screensaver."

- Presence inflation risk and mitigation: because background tabs could silently corrupt drift population-wide.

- Sync correctness risk and mitigation: because lost or duplicated drift is unreachable by users.

- Catch-up non-determinism risk and mitigation: because cold and hot aviaries could diverge and replay tests could flap.

- Audio uncanniness risk and mitigation: because audio is the affective spine and synthetic-sounding calls break the spell.

- Autoplay policy risk and mitigation: because first-frame audible calls are not always possible.

- First-frame budget risk and mitigation: because above 500 ms the aviary reads as loading.

- SVG-to-canvas takeover pop risk and mitigation: because discontinuity is an entrance animation by another name.

- Memory growth risk and mitigation: because 30-minute laptop sessions and audio graphs are leak-prone.

- Accessibility regression risk and mitigation: because narration cadence, caption matching, and focus visibility can drift.

- Announcement creep risk and mitigation: because even "just one toast" violates the central register.

- Notebook voice drift risk and mitigation: because the notebook is where the voice is most concentrated.

- Magic-link deliverability risk and mitigation: because it is the only door.

- Email identifier PII risk and mitigation: because compliance findings would be impossible to retrofit.

- Privacy boundary leakage risk and mitigation: because a helpful dashboard could convert the relationship into a data product.

- Tick cost at scale risk and mitigation: because always-on ticks for many cold aviaries require hot/cold exact catch-up.

- Timezone and clock edge-case risk and mitigation: because dawn resets and night states depend on the right hours.

- Safari/iOS WebAudio risk and mitigation: because the aviary may be silent without explanation.

- Seven-bird recognizability risk and mitigation: because the cap is empirical and chorus may blur earlier.

- Newcomer discoverability risk and mitigation: because no announcement means some users may never name the bird.

### Decision log

- Five top-bar items: because two files require settle in the top bar, the scene must stay chrome-free, and the bar stays sparse and fades.

- Mood words and `settled` split: to avoid one word meaning two things in code and prose.

- Logical tick for all aviaries with hot live ticking and cold catch-up: to preserve the architectural property exactly while keeping cost proportional to use.

- Export includes personality vectors as labeled engine values: because export is a data-portability artifact and the exposure rule targets product surfaces.

- Phenotype projection in snapshot: to keep raw numbers out of network payloads a user could inspect.

- New-bird arrival without announcement: because the birds are "birds that arrived, not birds the user chose," with no announcement surface and minimal chrome.

- Listen-in device-local: because it is audio focus for the person listening on that device and syncing would surprise the other device.

- Settle canonical flag cleared by return or re-engagement: because settle is a goodbye and opening the aviary anywhere is a return.

- Lowercase bird names in naturalist surfaces but stored as entered: for voice consistency in naturalist surfaces while account surface remains matter-of-fact.

- Mute not a drift input: because of no-punishment; muting for a meeting must not count against anyone.

- Presence activity window initially four minutes: because the PRD says lean long.

- Offer cooldown semantics: to match per-bird cooldown while keeping the tray from being a button to mash.

- Greeting rate limit: to prevent twitchy birds without touching greeting variety.

- Polling instead of WebSocket/SSE: because the tick is 60 seconds and polling matches it and is cheaper to operate.

- Visit notification email only, opt-in, limited: because the product never pushes, the social file allows an opt-in toggle, and email is the only channel.

- Cross-device magic-link consumption in opened browser: because the PRD says "signs them in on that browser."

- Loading-state audio silent with captions: because of browser policy and no prompt is consistent with no announcements.

- Nine-slot seven-bird layout: to leave movement room and prevent cropping at any viewport.

### Work breakdown and sequencing

- Foundations phase: to put monorepo, CI budgets, lints, schema, engine skeleton, geometry, edge shell, palette, and string table in place before parallel work.

- Engine alpha phase: to deliver `tick()` with drift, mood, perch, weather, calls, greetings, arrivals, offers, observer, calibration, and determinism tests inside bands.

- Scene renderer phase: to deliver fixture-driven scene at 60 fps with procedural birds, motion, transitions, responsive layout, reduced motion, focus, and adoption states.

- Audio phase: to deliver worklet synth, grammar, signatures, captions, mixing, weather sound, autoplay handling, fallback, and listening reviews.

- Backend and sync phase: to deliver auth, API, event log, workers, sweeper, catch-up, edge first frame, export/deletion, mailer, telemetry, alarms, and two-device end-to-end behavior.

- Interactions and presence phase: to complete presence, event queue, greeting, listen-in, offers, settle, pulls, and error surfaces against real API.

- Notebook, narration, chrome, accessibility phase: to make prose, narration, keyboard, settings, adoption, unsupported-browser, axe, keyboard scripts, and screen-reader review green.

- Visits phase: to deliver invites, visitor sessions, visitor mode, visit log, revocation, opt-in email, and zero visitor events.

- Hardening phase: to complete soak tests, first-bird budget on real devices, synthetic fleet, runbooks, canaries, spell-break walkthrough, listening study kit, and privacy review.

- Definition of done: every invariant has a passing test or structural enforcement, every budget is enforced and observed, accessibility gates are met before beta, and the spell-break checklist is empty.
