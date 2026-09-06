## System-level intent

- Make the PRD claims "structurally true rather than true-by-policy." This is stated in "Load-bearing decisions" and then repeated through tooling: DB roles, schema denylist tests, voice lint, no notification component, size limits, soak tests, and privacy filters.
- Keep the aviary alive without the viewer while keeping scale bounded. The pure `tick`, warm/cold scheduling, bit-identical catch-up, daily cold sweep, and "it kept going" notebook entries all carry the intent that "the aviary continues without the viewer."
- Maintain one canonical server truth, with immediate client-visible response but no client-authored bird state. The plan says the tick is "the only writer" of bird state, while the responder writes only events and plans so the client "never waits a tick to react" and "plan and consequence always agree."
- Make personality perceptible but hidden. The snapshot "never carries the personality vector"; it carries only "quantized, named presentation parameters" as an expression profile, while the export is "the one PRD-sanctioned place the vector leaves the server."
- Treat presence as real attention, not an exploitable counter. Presence requires visibility, focus, recent activity, server-observed liveness, and unioning across devices, so "two devices open at once earn one device's worth of presence" and "a click cannot buy anything."
- Keep drift gentle, monotonic, and non-punitive. The plan's "monotonic-toward-expressive rule" means "neglect contributes zero," no term can be negative, and the "No Tamagotchi mechanics" consequence is no negative drift, distress, hunger, or death.
- Preserve a calm, non-gamified product shape. The non-goals forbid counters, streaks, badges, levels, calendars, visit-frequency surfaces, toasts, banners, and social-network mechanics; the call scheduler caps sound so a seven-bird aviary stays "a place, not a soundscape."
- Split naturalist product voice from matter-of-fact system voice in code. `ProductCopy` is "lowercase, present tense, no second person, no exclamation"; `SystemCopy` is "sentence case, direct"; scene and notebook surfaces cannot render the wrong type.
- Make accessibility first-class rather than bolted on. The DOM accessibility overlay, reduced-motion renderer, captions, screen-reader narration, keyboard navigation, contrast gates, and outside AT sessions all support the claim that accessibility is "a v1 deliverable with the same milestone gates as rendering and audio."
- Use procedural, deterministic sensory systems. Species art, call grammar, weather, RNG streams, first-frame poses, and AudioWorklet synthesis are all seeded or generated so devices agree and no recorded audio assets enter the bundle.
- Enforce privacy at schema and infrastructure boundaries. The plan says privacy is enforced "at schema boundaries, not policy": email exists in one encrypted column, telemetry is separate from `sim`, metrics are whitelisted, and the plan deliberately does not measure per-account, bird, email, or engagement data.
- Calibrate with benches, synthetics, consented beta research, and gates before launch. Drift cohorts, live/catch-up equivalence, listening tests, synthetic Playwright sessions, soak tests, and beta interviews make calibration evidence part of the product system.

## Per-feature whys

### Load-bearing decisions

- Pure `tick`: exists so "the aviary continues without the viewer" at scale; warm ticks and catch-up ticks are bit-identical, and CI asserts live/catch-up equivalence.
- Two server-side computation paths: the tick owns canonical consequences while the responder gives immediate greeting and offer plans so the client "never waits a tick to react."
- Snapshot expression profile: keeps the personality vector out of the wire format, DevTools, and client features while still allowing presentation to drift visibly.
- Presence credited by unioned, clipped intervals: prevents two devices from earning extra presence and prevents stale pings from earning presence after snapshot pulling stops.
- Canvas 2D scene with DOM accessibility overlay: seven birds "do not justify WebGL," and the overlay makes keyboard, screen-reader, caption, focus, and hit-testing surfaces first-class.
- One pose library with continuous and reduced-motion renderers: ensures reduced-motion is "a second renderer over the same behavior state machine," not a disabled experience.
- AudioWorklet synthesizer with persistent voices: avoids per-call allocation, supports deterministic memory, and makes captions match the structured call object the synth plays.
- Shared TypeScript `prose` package: keeps notebook entries, narration, and captions in one naturalist voice codebase, with lint to prevent "you," exclamation, visit-frequency language, and trait leakage.
- First bird from the HTML itself: makes "already in motion" survive a 4G cold load and lets repeat visits draw from the last known snapshot before the network answers.
- Privacy enforced at schema boundaries: separates simulation and telemetry systems, strips UUID/email-like attributes, and keeps email in exactly one encrypted column.

### Scope

- Single-user accounts: NOT RECOVERABLE FROM PLAN
- Email magic-link sign-in: NOT RECOVERABLE FROM PLAN
- Per-device revocable sessions: NOT RECOVERABLE FROM PLAN
- Verified email change: NOT RECOVERABLE FROM PLAN
- JSON export by emailed link: the export is the only sanctioned place the personality vector leaves the server, and it gives the account a complete current state including birds, vectors, moods, notebook entries, and settings.
- 30-day soft delete then hard delete: gives a signed-in user during the window an "I changed my mind" restore affordance, then removes account data and export objects; backup retention means deleted data leaves backups within 44 days.
- One canonical aviary per account: avoids merge semantics; sign-in on a new device uses the same record, and `aviaries` is separate so the account-to-aviary join is explicit rather than implied.
- Two starter birds: one starter is biased so a "greets first" bird exists from day one, and first-session arrival plans give the user birds in motion rather than an empty state after adoption.
- Cap of seven birds: protects the call-signature recognizability ceiling and keeps Canvas 2D, memory, chorus, and visual scenes within the plan's performance assumptions.
- Age-gated arrivals: lets production accounts reach new birds slowly by aviary age while beta can accelerate arrivals to validate three-to-seven-bird chorus quality before production gets there.
- Naming and renaming: names feed prose, focus, captions, and settings while preserving stable bird identity; rename changes no other column, so focus and narration continue uninterrupted.
- Hidden five-trait personality vector: supports drift and expression while preventing numeric personality from being exposed except in account export.
- Monotonic drift from presence, listen-in, and offers: avoids Tamagotchi mechanics; neglect contributes zero and cannot decrease a trait.
- Mood system: moods persist across sessions because they are stored rows updated only by live or catch-up ticks, so session start cannot reset a bird.
- Bird-to-bird responses and chorus: makes chorus and social response real behavior, while the calm cap keeps the aviary "a place, not a soundscape."
- Return-greeting: fulfills the promise that "one bird notices"; the planner guarantees a greeter and times the greeting to land within "the first second or two."
- Idle presence accounting: credits real attention and blocks background tabs, always-open unfocused laptops, stale pings, and duplicate devices from inflating drift.
- Listen-in: treats focused attention on a bird as real credit while the mixer emphasizes the bird without fully muting the rest of the aviary.
- Offers: request-path plans make reactions start within one network round-trip; cooldowns are hidden so "the offer affordance is always available" and "the engine decides what happens."
- Settle with five-second undo: NOT RECOVERABLE FROM PLAN
- Read-only field notebook: detectors receive an aviary-only projection with no presence or session data, so the notebook "never observes the user"; the sparsity governor creates a quiet record of the aviary continuing.
- One horizontal non-scrolling scene: NOT RECOVERABLE FROM PLAN
- Local-time day/night: NOT RECOVERABLE FROM PLAN
- Rain and wind: weather is deterministic state in the snapshot so host and visitor see the same event, and no weather is scheduled from 00-05 because "the user would never see it" and it would dampen the nightjar.
- Sparse fading top bar: supports the calm scene by fading chrome to 8 percent opacity, while focus prevents fading so keyboard users keep stable controls.
- Quiet-field loading and empty states: replaces spinners and degraded snapshot waits with a soft field that keeps the product calm even when origin is slow.
- Procedural WebAudio calls and graceful silence: avoids recorded audio assets, preserves generated call identity, and turns captions on by default when audio cannot play.
- Read-only visits by emailed one-time invite: preserves the no-social-network boundary; visitor sessions can view snapshots but have no write endpoint and no presence, listen-in, or offers.
- Invite revocation, expiry, and visit log: lets the host control access and see on-demand approximate visits without creating cross-account social graphs.
- Opt-in visit notification email: is the single allowed notification, is off by default, and is rate-limited so it does not become a notification system.
- Accessibility surfaces: ship in v1 because accessibility has the same milestone gates as rendering and audio, not a follow-up phase.
- Browser support for the last two major versions: NOT RECOVERABLE FROM PLAN

### Architecture and stack

- TypeScript end-to-end monorepo: keeps greeting planner, offer responder, call scheduling, and prose generation identical across server canonical plans and client narration, captions, and first-frame rendering.
- `Postgres only` for v1: rows are small, consistency matters more than throughput, and one datastore keeps the privacy boundary simple.
- Fastify, Preact, and no framework in the scene renderer: keeps the HTTP layer small and fast, uses Preact for chrome only, and keeps the scene independent of framework weight.
- Edge HTML assembly: inlines the authenticated snapshot and first-frame renderer so first bird visible can happen before the full app, audio, or prose bundles load.
- Separate ops telemetry: receives only whitelisted aggregate metrics and has no credentials to the simulation database, matching the privacy boundary.

### Data model

- Encrypted email with blind index: supports email lookup without decryption and prevents email from becoming a foreign key or log field.
- `sessions.last_snapshot_at`: serves as the liveness signal used to clip presence and prevent suspended tabs from continuing to earn credit.
- Separate `aviaries` table: keeps "one aviary per account" explicit rather than implied by the account row.
- Immutable bird identity columns: preserves the stable identity the plan says the PRD requires; no row ever replaces a bird's `id`, `species_id`, or `call_signature_seed`.
- `bird_traits` update permissions: enforce that only the tick-worker role can write personality and the API role can only read for export assembly.
- `daily_credit` and `presence_intervals`: keep saturation bookkeeping internal, short-lived, and never serialized to clients.
- Append-only `interaction_events`: preserves global order, idempotency, and exactly-once tick consumption while avoiding future recomputation from event replay.
- Immutable `notebook_entries`: stores rendered text at creation so the observer never rewrites history.
- Invite and visit rows as host data: keep visitor sessions from linking to a visitor account and delete visitor data with the host account.

### API surface

- HTTPS JSON, cookies, CSRF, CSP, and no third-party origins: secure session and inline-first-frame handling while preserving the no-third-party-scripts boundary.
- Matter-of-fact error catalogue: keeps error copy in `SystemCopy` so the client never composes error text and errors do not appear inside the scene.
- Rate limits with identical pre-auth responses: prevents account enumeration and avoids revealing whether an email has an account.
- Snapshot endpoint with ETag and catch-up: updates liveness, runs bounded catch-up before assembly, and returns `304` when nothing changed.
- Host-only greeting fields: `absence_seconds` and `greeting` are only for host initial/visible pulls because visitors "never receive a greeting plan" and "the birds notice the host, not the visitor."
- Event batching with idempotency keys: acknowledges duplicates without re-applying them and returns responder plans for immediate offer reactions.
- Visitor snapshot and write rejection: uses the same renderer with the interaction layer compiled out, while the server rejects writes regardless.

### Simulation engine design

- Fixed-grid tick contract with explicit RNG: makes `tick_index` fully determine `tick_time`, makes old ticks idempotent no-ops, and makes replay identical across Node and browser builds.
- Warm, cold, catch-up, and daily sweep scheduling: keeps active aviaries live every 60 seconds while bounding cold catch-up cost and keeping all aviaries no more than about 25 hours behind.
- Drift saturation formula: slows growth near ceilings, prevents hard walls, makes a marathon session unable to buy much more than an hour, and gives a click no credit.
- Muted presence half-rate vocal component: implements "whether you mute the calls or let them play" without making muting punitive or negative.
- Adoption seed ranges: starters begin reserved with room to move forward, one starter is warmer so a greeter exists, calls are audible early, plumage can visibly drift, and offers can get an early reaction.
- Calibration cohorts and expression-step tests: turn "regular visits" and three-week visible drift into testable thresholds rather than taste judgments.
- Mood dwell and pressure system: combines time of day, weather, recent interactions, other birds, and personality while preserving mood across sessions.
- Perch zones and relocation: shape position by mood and boldness and record server-timed moves so devices animate the same relocation.
- Weather schedule: deterministic RNG makes weather reproducible during catch-up and shared between host and visitor.
- Call scheduling: covers 90 seconds so a late pull never leaves silence; responses, chorus windows, and caps make social sound controlled and calm.
- Greeting planner: selects the greeter by boldness, warmth, mood, and jitter so a return can always be noticed without unison.
- Offer responder: evaluates every eligible bird by offer kind, mood, and curiosity, returns the plan immediately, and leaves canonical state changes for the next tick.
- Notebook observer and governor: writes sparse, salient, aviary-only observations at a rate of days rather than sessions and never uses presence as an observation source.
- Starter and newcomer arrivals: newcomers arrive quietly as full unnamed birds on the back perch, with naming and "let it move on" reachable but no prompt, banner, or notification.
- Expression profiles with hysteresis: translate traits into coarse rendering words, make the three-week visible drift target testable, and prevent flicker at boundaries.
- Engine versioning and bench: lets engineers answer "what happens if we change" constants before production and prevents constant changes from bypassing bench expectations.

### Sync model

- `state_version`: lets clients ignore older responses on flaky mobile networks and use ETag/`If-None-Match` for cheap idle keepalives.
- Polling instead of WebSockets or SSE: 60-second snapshots are cheap, simpler, and cover every scenario the plan describes through visible, frame-gap, reconnect, and post-event triggers.
- Server-time scheduling: lets two devices side by side hear the same call at the same moment.
- Clients write only events: removes last-write-wins conflict paths because no client submits absolute bird state.
- Multi-device semantics: make concurrent devices additive-but-bounded through presence union, per-account cooldowns, account-level settle, and stable bird ids.
- Conflict prevention: advisory-locked idempotent ticks, `seq`-ordered event consumption, and no optimistic presentation returned as state mean there is nothing to overwrite.
- Failure handling: keeps rendering from the last snapshot, catches up inline after tick-worker lag, retries events with the same keys, and shows a system panel only after prolonged load failure.

### Frontend rendering pipeline

- Canvas 2D renderer: chosen because seven birds and ornaments fit the cost profile, while WebGL would add context-loss handling and library weight "for no visible gain."
- First-frame path: paints birds mid-action under the 500 ms target and allows the core renderer to adopt the existing canvas with no visual discontinuity.
- Service worker warm load: draws from the cached last snapshot in under 100 ms and reconciles with the network by animation.
- Species art and pose library: make plumage and posture drift visible through path parts, palettes, and keyed poses rather than sprite swaps.
- Behaviour state machine: uses mood-keyed micro-actions and seeded timing so "the same bird is recognisably itself and never loops."
- Reduced-motion renderer: preserves calls, captions, narration, greetings, offers, and notebook while removing breathing noise, parallax, leaves, and particles.
- Snapshot reconciliation rules: animate toward server truth and avoid snaps when perch, mood, expression, release, or call schedules change.
- Responsive layout: keeps all birds visible down to 320 by 480 and ensures no bird is culled or clipped.
- Client presence tracker: owns the three presence conditions and closes intervals at the last moment all conditions held, including suspend/resume cases.
- Notebook panel virtualization: allows scroll-back over years without retaining a large DOM.

### Audio pipeline

- AudioWorklet with persistent voices: gives zero per-call allocation, sample-accurate off-main-thread scheduling, deterministic memory, and bird-like timbres that native oscillators make awkward.
- Structured `Call` object: is consumed by both synth and caption generator so text and sound agree.
- Per-bird call signature: makes each bird recognizable while mood and drift affect frequency, length, and sharpness without changing identity.
- Arrival signature selection: avoids assigning a signature sharing three of four dimensions with a resident, protecting the seven-bird recognizability ceiling.
- Chorus mixing: offset onsets, detuned pitch centers, and compression avoid beating, phase artifacts, and unison.
- Listen-in mixer: brings one bird forward while keeping a floor so no bird is ever muted by listen-in.
- Weather audio choice: rain is not rendered as audio because a rain loop would be "recorded-style ambience by another name."
- Autoplay lifecycle: runs the scheduler even while audio is suspended so captions and narration still describe real calls, then fades in on first activation with no banner.
- Silence-and-captions fallback: keeps the aviary usable when AudioContext or AudioWorklet fails without introducing recorded-audio fallback.
- Audio CPU and memory limits: fixed voice arrays, fixed reverb delay lines, and small scheduling messages support the 30-minute no-growth rule.

### Accessibility surfaces

- Screen-reader live region replacement: prevents queued announcements from piling up.
- Narration cadence and priority suppression: keeps idle narration sparse and lets return-greeting, offers, settle, arrivals, and naming speak within one second without flooding.
- Captions near calling birds: use the same call object as audio, stay visually tied to the bird, and are `aria-hidden` so screen readers do not hear duplicates.
- Caption contrast scrim: guarantees WCAG AA across midday sky, night palette, rain, and settled state while staying as light as the palette allows.
- Reduced-motion preference from first frame: prevents a reduced-motion user from seeing an initial burst before the setting applies.
- Keyboard navigation and focus: lets a user focus birds, listen in, offer, settle, and undo without sighted help, with focus following bird id through relocation.
- Focus indicator: uses dark and light halos so it remains visible against bright day and dim night.
- Voice split as code: prevents system copy from leaking into scene/notebook components and product copy from leaking into settings or error panels.
- Assistive-technology testing: validates with screen-reader and reduced-motion users that participants can describe birds, listen in, offer, settle, and experience it as calm.

### Performance budgets and observability

- Initial JS and critical-path budgets: are met through manual chunks, procedural art and audio, code-splitting, and no scene framework, then enforced by `size-limit`.
- First-bird-visible budget: is met through edge-inlined snapshot, inline first-frame rendering, service-worker cache, and delayed audio/prose loading.
- 60 fps and memory budgets: are met through cached layers, pooled ornaments, DPR cap, preallocated voices, bounded LRU caches, and virtualized notebook.
- Aggregate-only RUM and synthetics: measure load, frames, audio health, snapshot health, and session-duration histograms without account, bird, or email dimensions.
- Deliberately unmeasured engagement data: enforces privacy and anti-gamification by excluding account session counts, retention funnels, offer rates per bird, drift distributions, and visit counts across hosts.
- Privacy boundary infrastructure: separates telemetry and `sim`, strips UUID/email-like attributes, scrubs logs, audits support access, and documents aggregate categories.
- Alarms and on-call: watch tick p99, scheduler lag, snapshot p95, mail failures, first-bird p75, worklet errors, catch-up storms, hard-delete failures, and export failures.

### Testing, rollout, and sequencing

- Engine unit and property tests: enforce determinism, monotonic drift, zero drift without presence, presence union, mood dwell, and perch rules.
- Live-vs-catch-up property test: proves the continuity promise for warm and cold aviaries.
- Client visual, reconciliation, accessibility, voice, first-frame, and bundle tests: keep both renderers, accessibility surfaces, copy rules, and takeover behavior from drifting apart.
- Soak tests: prove no memory growth while exercising calls, listen-in, offers, notebook scrolling, weather, and relocation storms.
- Data and migration tests: protect bird identity, trait values, notebook text, immutability triggers, hard delete, and export object deletion.
- API and security tests: enforce schema contracts, snapshot denylist, magic-link behavior, no enumeration, CSRF, rate limits, visitor write rejection, and revoked-invite `410`.
- Audio golden and spectral tests: preserve deterministic PCM output, distinguishable bird signatures, and chorus onset/pitch constraints.
- Load tests: verify the budgets while 50,000 warm aviaries tick, 10,000 clients poll, and 2,000 cold catch-ups per minute run.
- Milestones: sequence foundations, private alpha, closed beta, launch, and post-launch calibration so the bench, first-frame path, accessibility, listening, privacy, and voice gates mature before v1.
- Accelerated beta arrivals: validate chorus and recognizability for three to seven birds before any production account reaches those ages; production never accelerates.
- Feature flags and kill switches: allow visits, audio worklet, newcomer arrivals, notebook observer, RUM, and edge inlining to pause or fall back without deleting history or breaking user-visible promises.
- Beta research and consent: is the sole way to validate drift calibration on real behavior because production never analyzes bird or interaction data.
- Week-one protocol freeze and workstreams: unblock engine, platform, scene, audio, and prose/accessibility work in parallel from shared schemas.
