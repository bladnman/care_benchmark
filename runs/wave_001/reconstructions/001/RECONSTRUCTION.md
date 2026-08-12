## System-level intent

- Aliveness is the product and an architectural property. The plan states "Aliveness is the product" at the top, then repeats in the design thesis that "the aviary has been continuing without you" is "not a visual trick." It reappears in the first-frame blocker, "quiet field" loading state, server tick, mid-action render, and procedural audio.

- Presence is the primary interaction, and presence means attention rather than engagement. This shows up in "Presence is the primary interaction," the three-signal conjunction for presence, the calibration comment "lean long; watching is the product," and the final refusal: "Presence is attention, not engagement. The code must not optimize for click count."

- Canonical life belongs to the server; the browser is presentational. The plan says "The server is the only writer of canonical bird state," "Clients never tick," and "A closed laptop and a pocketed phone do not each hold a private universe." This principle governs sync, drift, snapshots, cache, catch-up, and column grants.

- Absence is acceptable and must not become punishment. The plan rejects "Hunger, death, distress, decaying happiness, any neglect punishment," defines "settle: no delta personality" and "neglect: no negative delta," and ends with "Absence is fine. The engine does not punish it. The UI does not mention it."

- Non-goals are constraints, not backlog. The plan says native apps, gamification, Tamagotchi decay, and social-network surfaces "must not appear as 'harmless' extras." This carries through out-of-v1 scope, "What is not an API," no badges, no counters, no "what's new," and scope-creep risk.

- Product voice is quiet, observational, and non-announcing. It appears in "No announcement toasts," "No welcome chrome," naturalist lowercase notebook/narration, matter-of-fact auth/account errors, and "In-app, birds notice; the system does not announce."

- Privacy is structural rather than only policy. The plan requires "Operational telemetry only," "No per-bird / per-account relationship data," no analytics warehouse connection to simulation tables, encrypted email, aggregate-only metrics, and a credential split that is "not a policy paragraph."

- Accessibility ships as part of the aviary. The plan names "Accessibility shipping on day one," says it is "a designed product surface," requires narration, captions, reduced motion, keyboard, WCAG AA chrome, DOM hit targets, and says a canvas-only scene would "silently fail."

- Naturalist product language is separate from system language. The plan uses "naturalist screen-reader narration," lowercase notebook prose, "matter-of-fact" settings/errors, and a `packages/prose` split between `naturalist` and `system`.

- Procedural and deterministic behavior preserves voice, budget, and privacy. This appears in "No LLM-generated live copy," deterministic prose templates, "A call is not a file," procedural audio, shared call grammar for captions, and the refusal to fetch MP3 fallbacks.

- Identity is stable while expressiveness grows slowly. The plan repeatedly protects stable bird IDs, calls personality drift "additive" and "monotonic non-decreasing," forbids last-write-wins, and treats export as "a possession, not a dashboard."

- The aviary is a small social system, not a social network. The plan says "The aviary is a small social system, not seven independent timers," while visits are "Small on purpose" and read-only, with no chat, comments, avatars, explore, rankings, or visible visitor presence.

- Performance serves aliveness instead of loader chrome. The plan ties <500ms first bird, quiet field fallback, no spinner, cache/bootstrap snapshot, rAF pause, 2D canvas, code splitting, and memory soak to the same aim: the first visible state should be "a quiet field or an already-moving aviary."

## Per-feature whys

### Scope and locked decisions

- Web client for last two major versions of Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN

- Email magic-link auth as the only sign-in mechanism: NOT RECOVERABLE FROM PLAN

- Per-device revocable sessions: NOT RECOVERABLE FROM PLAN

- Email change with verify-first: NOT RECOVERABLE FROM PLAN

- 30-day soft delete then hard delete: the plan gives users a recovery window ("session can undelete") while preserving a later "hard wipe job" that removes events, birds, vectors, notebook, invites, sessions, jobs, exports, and the account row.

- On-demand JSON export: the export includes personality vectors because "the relationship is the user's to copy," while the plan stresses the export file is "a possession, not a dashboard."

- One canonical aviary per account: the rationale is sync and product identity. The plan calls laptop and phone "two cameras on one aviary" and rejects private client universes.

- Two starter birds chosen by the system: the plan ties this to "no catalog," contrast in silhouettes and motif families, and a first meeting that is "gentle, not parade."

- Age-gated adoption up to seven: the plan says gates use "Aviary age, not engagement," and that a later arrival is "a quiet arrival, not a catalog."

- User-assigned renameable names: the rationale is relationship formation. The plan says "Users meet names, not a dex" and the narration should include names "so relationships form."

- Stable bird IDs: the plan says bird IDs are "stable for the life of the account," "never recycled," and identity migrations must not allocate a new ID except hard-delete.

- Server-side simulation tick: it exists because server time owns the aviary's continuity: "Server owns time, personality, mood, weather, notebook candidacy," and "Clients never tick."

- Client rendering of a single non-pannable scene: NOT RECOVERABLE FROM PLAN

- Three perch zones: the plan gives "Three zones x 3 slots" as "enough for 7 birds without overlap."

- Local-time day/night: the plan uses account timezone and local hour for diurnal mood, lighting, and nightjar exceptions, making the aviary local to the account's current IANA zone.

- Rare weather: the rationale is aliveness and mood/audio variation; weather changes mood scores, vocal multipliers, day catch-up, notebook triggers, and visual overlay.

- Ambient ornaments: the plan keeps ornaments client-only because if no second client needs to share "a particular leaf," it does not belong in simulation state.

- Mid-action first frame: the plan forbids spinners, T-poses, skeletons, and fade-from-empty because first frame aliveness is a ship blocker.

- Return-greeting: the plan uses a client greeting director so greeting can fire within 1-2 seconds and does not wait on the tick; quiet aviaries stay quiet through low `greet_weight`.

- Idle presence: presence exists because "watching is the product," and the plan treats visibility, focus, and recent input as the minimum proof of honest attention.

- Listen-in: the plan frames it as "listening, not soloing tracks"; it changes the mix without muting others completely or changing which birds are allowed to call.

- Offer: offers are to the aviary rather than bird-click food meters; server receiver selection and cooldown keep reactions functional, quiet, and not punitive.

- Settle with 5s undo: settle is a local goodbye surface. The engine treats it as presence end, the undo avoids accidental exit, and there is "No scolding surface."

- Field notebook: the notebook is sparse, read-only, naturalist, and never about user behavior so it avoids streaks, achievements, and attendance records.

- Multi-device sync: sync is a property of server-authored state because last-write-wins could silently erase a morning of drift.

- Visit invitations: visits are read-only, off by default, revocable, and "Small on purpose" to avoid turning the app into chat, explore, rankings, or co-presence.

- Accessibility shipping on day one: the plan says accessibility is a designed product surface and a canvas-only scene would silently fail keyboard and screen-reader users.

- Performance budgets: the budgets protect first-frame aliveness, 60fps idle, no memory growth, and tick reliability instead of allowing loader chrome or leaking audio/caption nodes.

- Operational telemetry only: the rationale is privacy and future product restraint; per-bird relationship data must not enter warehouses or training paths.

- Native apps out of v1: NOT RECOVERABLE FROM PLAN

- Gamification, Tamagotchi decay, and social-network surfaces out of v1: the plan says there is "no version of those features that is compatible with this product."

- No LLM-generated live copy on request path: the plan gives three reasons directly: "latency, voice drift, privacy."

- No personality numbers in UI, debug panels, ARIA, or client payloads: the rationale is that "The user never manages a number," and personality stays server-only except in export.

- Tick cadence at 60s nominal: NOT RECOVERABLE FROM PLAN

- Presence activity window at 240s: the plan says to "Lean long so watching without moving still counts."

- Presence ping interval and server drop window: the rationale is presence integrity and rate shape; the server "validates shape and rate" and defends the log.

- Multi-device presence cap: the plan states "Two focused devices cannot double-speed drift."

- Seed traits clamped below max: the plan says this leaves "room to drift up" and prevents any bird from starting maxed.

- Mood enum with five states: NOT RECOVERABLE FROM PLAN

- Offer cooldown: the plan calls it "Functional, not punitive," and later says without cooldown "curiosity spikes in one sitting."

- Song library as 8 score fragments: the plan says fragments are "not audio files" and are shared with the call synth.

- New-bird age gates: the rationale is "Aviary age, not engagement," avoiding rewards for click volume or attendance.

- Greeting latency handled client-side: the plan says it must fire quickly and "No waiting on the tick."

- Personality never on the wire: the plan protects raw vectors by sending only quantized behavior hints and visual params.

- HTTPS JSON pull and no WebSocket in v1: NOT RECOVERABLE FROM PLAN

- 2D canvas scene plus DOM hit-target overlay: 2D canvas keeps "the bundle and GPU story simple," while DOM overlay preserves focus, keyboard, captions, and hit targets.

- Deterministic prose composer: the rationale is no live model calls, which avoids latency, voice drift, and privacy exposure.

- Audio unlock without a modal: the plan says "Silence is preferable to a toast."

- Settle as fifth top-bar icon: the plan says interactions require settle from the top bar and "Sparse fifth icon beats burying settle."

- Visitor chrome restricted to scene, calls, day/night, and weather: the rationale is that visitors are read-only and cannot write events or contribute presence.

- Visit notify opt-in by email only: the plan keeps notify off by default, email-only, not onboarding-visible, and "Still no push."

- English-only v1: the plan says "Naturalist lowercase is English-specific."

- IANA timezone from client: the tick uses timezone for diurnal mood and local lighting, defaulting to UTC only until first session.

- One primary greeter with staggered reactions: the plan says additional birds may react but "never unison," preserving quiet and avoiding a cue chorus.

- Empty aviary only during first adoption: the plan says after first adoption "the scene is never empty."

- Notebook sparsity target and hard cap: the plan makes entries sparse and forbids user-behavior observations to avoid a diary of attendance or achievements.

- TypeScript monorepo, Postgres, Redis, object storage, transactional email: NOT RECOVERABLE FROM PLAN

### Architecture and data

- Four deployables and one database of record: the plan says this is "One logical product," with Postgres as the system of record and Redis only ephemeral.

- No analytics warehouse connection to simulation tables: the rationale is privacy; it is enforced by "network ACL and a credential split, not a policy paragraph."

- Shared `packages/simulation` pure functions: the plan says the package must run in CI without I/O so drift calibration stays testable without production accounts.

- Snapshot as a state photograph rather than a frame stream: the plan keeps canonical state compact and lets the client interpolate pixels, audio, and ornaments.

- Client/server split for local ornaments and canonical state: the rationale is divergence control; shared state belongs on the server, non-shared ornaments stay client-side.

- Trust boundary with secure cookies, CSRF, hashed tokens, and no email logs: the plan protects account access and ensures email is "not a log field."

- No client-tick plus CRDT: the plan says merging two drift integrals is how "you silently delete a morning."

- Account UUID as the only account key: the plan uses UUIDs in FKs, logs, traces, queues, object keys, and rate-limit buckets so email does not spread.

- Email ciphertext plus lookup hash: the rationale is private storage plus login lookup without plaintext email.

- Append-only interaction events: ordered events are the source for additive server deltas, with no updates/deletes except hard-delete wipe.

- Notebook entries store final lowercase prose only: the plan keeps the notebook read-only and avoids recomputing or exposing internal triggers to users.

- Visit invite token hashes: invite and magic-link secrets are unguessable and hashed at rest.

- Export omitting raw event logs: the plan says "presence pings are not a souvenir."

- Export omitting visitor emails: the plan says not to include them "to reduce spreading third-party PII."

- Six-species pool with no rarity: the plan wants "one coherent temperate-edge set," not a collection or rarity system.

- Starter pair sampled for contrast: the plan avoids identical starters and specifically says never two nightjars as the only pair.

- Derived attunement via recency factor: the plan calls this the mechanical expression of "quieter after absence" without birds that "mistrust you."

- Personality vectors in export only: the plan allows numbers in a file because the relationship is copyable, while screens and payloads stay number-free.

### API and sync

- Magic-link request always returns generic 202: the plan gives the reason as "no account oracle."

- Matter-of-fact error bodies: the plan says there is "No naturalist voice on 4xx/5xx."

- Snapshot endpoint with no personality: the rationale is that the client receives only behavior hints and visual params, never raw vectors.

- Event batch endpoint idempotency: the plan says retries are "free" because existing IDs return accepted without reinserting.

- Immediate `OfferResult`: the plan says it lets the client animate without waiting for the next tick.

- Receiver selection jitter: the plan adds tiny jitter "so it is not robotic."

- Visitor snapshot without notebook, arrival, cooldowns, or interaction: the plan keeps visitor role read-only with `can_interact: false`.

- Visitor heartbeat that does not write `interaction_events`: the rationale is visitors cannot contribute presence or drift.

- No personality, stats, streaks, explore, live websocket, or client-submitted perch APIs: the plan excludes numbers, gamification, public discovery, client simulation, and absolute client positioning.

- Client-time clamp: the plan uses client time only as a hint and clamps it so drift stays safe despite skew.

- Snapshot pulls on visibility, sleep/resume gap, and 25s visible keepalive: the plan says lighting and weather should move even without presence, while hidden tabs do not poll.

- Single canonical record sync: laptop and phone are "two cameras on one aviary"; neither patches birds.

- Last-write-wins forbidden: the plan says a stale phone session would erase a morning of laptop presence without an error.

- Event ordering by server sequence: the rationale is two-device safety "without clock sync."

- Presence guards: the plan defends the log with conjunction checks, rate limits, wall-clock caps, no visitor presence, and a ship-blocker against "tab open."

- Conflict and error surfaces without sync picker: the plan says true personality conflicts cannot occur, so there is no "choose a device."

- Local snapshot cache for first bird: the plan says this is how <500ms and "already in motion" coexist on repeat visits.

- Online-only offline handling with in-memory queue: the plan avoids localStorage because it could become "a second source of truth" that replays days later.

### Simulation engine

- Single-writer tick loop per aviary: the plan says "Never two ticks on one aviary."

- Tick input limits: the tick must not read other accounts or write telemetry containing bird fields, preserving privacy and account isolation.

- Monotonic personality drift: the plan uses headroom and delta >= 0 so personality becomes expressive without decay or punishment.

- Drift calibration fixtures: the rationale is that too fast becomes "Tamagotchi numbers" and too slow becomes "screensaver."

- No Tamagotchi ambient model: the plan says "Do not implement 'lonely -> wary'" because wary is not a moral judgment.

- Expressiveness through `recency_factor`: the plan keeps a high-vocal bird still that bird, just quieter until presence returns.

- Mood persistence and hold time: the plan says there is "No snap-to-neutral on tab open" and a 12-minute hold "prevents flicker."

- Daily-ish mood blend: the plan calls it "Not a hard reset," preserving continuity while following diurnal prior.

- Server perch intent: the rationale is shared state and non-overlap; users never place birds.

- Bird-to-bird contagion and chorus: the plan says the aviary is "a small social system, not seven independent timers."

- Catch-up jump after long absence: the plan says not to run 20,000 full ticks after two weeks, while seeded weather keeps snapshot pulls agreeing.

- Piecewise event replay during catch-up: the rationale is applying events at their effective time rather than on top of the wrong mood.

- Procedural call grammar: the plan says "A call is not a file," consecutive calls must be unequal, and recognizability comes from species motif, pitch offset, and timbre.

- Shared grammar for captions and audio: the plan says if server and client disagree, "captions lie."

- Return-greeting director: the plan makes it presentational and weighted so quiet aviaries stay quiet and others do not chorus on cue.

- Adoption first-fly-in and later arrivals: the plan uses gentle first meeting, staggered entry, and no catalog grid to avoid collection mechanics.

- Identity migrations by additive columns: the rationale is stable identity; grammar renames get a map, and birds are never deleted except account hard-delete.

- Notebook trigger allowlist and forbidden triggers: the plan uses naturalist observations and forbids visit counts, presence minutes, streaks, trait deltas, and session starts.

### Frontend, audio, interactions, and accessibility

- Quiet-field HTML shell: the plan says the sky is "the place, not a loader," and the user sees quiet field if snapshot is slow.

- Mid-action bird placement from snapshot phase: the plan avoids T-poses, fade-ins, and wake animations so birds are already alive.

- Idle micro-motion: the plan says birds should never be "still in a way that reads paused."

- Hidden-tab render pause: the client cancels rAF while the simulation continues on the server, then returns with a fresh snapshot rather than old time.

- Continuous lighting and weather transitions: the plan avoids stepped lighting, teleports, and thunderclaps to keep the scene subtle and continuous.

- Reduced-motion renderer: the plan calls it "designed," not `animation: none`, preserving day/night, settle, audio, captions, notebook, and drift.

- Top bar fading and no badges: the plan keeps chrome quiet and forbids unread dots, visit counts, and other engagement cues.

- Responsive safe rect and DOM hit overlay: the plan protects birds from cropping and guarantees 44x44 tap targets.

- Offer visuals as props: the plan says seed, song, and pool props are not food meters and despawn after reaction or timeout.

- 2D frame loop: the plan chooses 2D canvas to keep bundle and GPU story simple.

- Code splitting: the rationale is first-bird performance; account, visit, notebook, and settings surfaces defer behind the aviary chunk.

- Procedural audio instead of looped files: the plan says looped files break on exact repeat, choruses become stacked karaoke, and PCM would exceed budget.

- Oscillator pooling and node reuse: the rationale is memory stability; the plan forbids per-call oscillator leaks.

- Listen-in gain mix: the plan says it should feel like listening, not soloing tracks, with other birds never at zero.

- Chorus answer limit: the plan caps two answers per call "to avoid pile-on at seven birds."

- WebAudio fallback to silence plus captions: the plan rejects MP3 fallback and banners; silence is acceptable and captions are the pressure valve.

- Runtime captions from grammar: the rationale is naturalist voice and synchronization, not debug labels like "CHIP_03."

- Presence sensor inputs: the plan uses pointer/key, visibility, focus, and activity because "watching is the product" and touch/pen should count.

- Listen-in input without pointer highlight: the plan wants pointer listen-in to stay "visually quiet" while keyboard and SR users get the focus ring.

- Offer controls and labels: the plan uses naturalist fragment labels, not "Track 3," and prefers inert quietness over cooldown messaging.

- Settle implementation: settle is local lighting/audio, sends presence end, undoes within 5s, and produces no extra drift.

- No welcome chrome checklist: the rationale is no toasts, no "welcome back," no absence-length copy, no visit banner, no confetti, and no notebook attendance.

- Screen-reader narration live region: the plan provides observational prose instead of mood codes, includes names so relationships form, and avoids speech pileups.

- Bird hit targets as buttons with species/perch description: the plan keeps birds keyboard/SR accessible while hiding trait numbers from ARIA.

- Keyboard path and DOM focus ring: the focus ring is DOM so it is not lost to compositor tricks.

- Captions opt-in and auto-on fallback: the rationale is access to calls when audio is unavailable, with AA contrast and placement near the bird.

- Visitor accessibility path: the plan says visitors get narration and reduced motion, because otherwise a "deafblind-adjacent SR user" could be locked out of the only social view.

### Performance, privacy, social, rollout, and risk controls

- Aggregate-only observability: the plan measures route, tick, RUM, auth, and visit counts without account IDs or bird fields.

- Deliberately unmeasured relationship data: the plan refuses per-bird moods, traits, offer popularity, average drift, and visit graphs to avoid future leaderboard exposure.

- Browser support without broad polyfills: the plan says unsupported browsers get matter-of-fact copy and "No polyfill tax in the aviary chunk."

- PII storage limits: email appears only as ciphertext/hash in account and invite storage, with object keys based on account and job IDs.

- Interaction data purpose limitation: events exist to feed "that aviary's tick," not partners, training, cross-account similarity, or product datasets.

- Hard delete wipe: the rationale is complete removal of events, birds, vectors, notebook, invites, visit sessions, sessions, jobs, export objects, and then account row.

- Voice split lint: the plan keeps naturalist strings out of account/auth routes and title-case marketing out of aviary live regions.

- Visits "Small on purpose": the rationale is to allow a read-only glimpse without chat, comments, avatars, explore, rankings, co-presence, or visible visitor effects.

- Visit abuse limits: the plan uses outstanding invite caps, daily caps, token entropy, and no public index to limit abuse.

- Pure simulation tests: the plan tests monotonic drift, zero-event bit identity, presence cap, mood hold, age gates, and catch-up so server life remains correct.

- Grammar tests and listen review: golden captions and unequal emits catch determinism, while weekly human listening covers "Feels canned" failures tests cannot catch.

- API and client sync tests: dual-session interleaves, idempotency, visitor 403, soft-delete restore, presence sensor truth table, no unison greeting, and no toast import protect the core thesis.

- Accessibility CI: the plan tests live region, birds as buttons, captions AA, keyboard path, and reduced-motion screenshots because accessibility is not a later polish pass.

- Contracts freeze before renderer and tick diverge: the plan says snapshot/event schemas freeze first and weekly compatibility loads fixture snapshots from week 1.

- M0 through M5 delivery plan: the rationale is to reach something that can be "watched" early while public launch waits until aliveness, drift, access, visits, budgets, privacy, and calibration are real.

- Accessibility work parallel to relationship milestone: the plan says it is "not a polish milestone after M2" and does not wait for v1.1.

- No public M1 with drift off: the plan says a week of fake stasis "teaches the wrong relationship."

- Ramp by age gates only: the plan avoids reward framing and says if load is on fire, raise gates rather than sell bird count as a reward.

- Global tick cadence changes only: the plan says if load requires, drop globally rather than per-user "premium ticks."

- Instrumentation from day one without average boldness: the plan separates operational timing/error telemetry from relationship metrics.

- No in-product rollout announcements: the rationale is "birds notice; the system does not announce."

- Drift calibration mitigations: the plan treats "Pip felt different yesterday" as a rate bug and says never add negative drift to fix expressiveness.

- Silent presence inflation mitigations: the plan says a single visibility shortcut could over-drift the whole population, so tests, server guards, caps, and review labels are required.

- Sync correctness mitigations: the plan names ORM conveniences as the threat and uses DB column grants because they are "stronger than a comment."

- Audio uncanniness mitigations: the plan warns identical motifs, phase-cancelled doubles, harsh oscillators, and ringtone timbre make the product feel like a toy.

- Accessibility regression mitigations: the plan requires DOM overlay, CI, narration cadence, and reduced-motion review because canvas-only scenes silently fail.

- First-frame aliveness mitigations: the plan calls a spinner the likely aesthetic regression and bans spinner components on aviary routes.

- Notebook/narration genericizing mitigations: the plan expects engineers to write "session started" because it is easy, so it uses trigger allowlists, forbidden substrings, and editorial ownership.

- Tick cost mitigations: catch-up jump and optional deferred ticking avoid stale returns and wasted work while preserving server-owned personality changes.

- Privacy leakage mitigations: synthetic IDs, log allowlists, warehouse ACLs, and export omissions prevent email traces, event debug exports, and shared visit logs.

- Scope-creep mitigations: the plan identifies first streaks, friend-visited toasts, public aviaries, and hunger bars as incompatible with this product.

- Seven-bird chorus cap risk: the plan says if recognizability fails at five, do not raise the cap or ship eight.

- Offer saturation risk: the plan says the four-minute server-side cooldown is mandatory because without it curiosity spikes in one sitting.
