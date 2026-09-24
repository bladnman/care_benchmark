## System-level intent

- Server-owned canonical life. The plan repeatedly protects one source of truth: "Only the server tick writes personality," "a single canonical record per aviary lives in Postgres," and "the model has no client state to merge." This shows up in the invariants, the client/server split, the tick transaction, and the sync model.

- Expressive change without punishment. Personality is "monotonic toward expressive"; "absence never produces negative drift"; ignored birds become "ambient" through a separate `attention_envelope`, not through sadness or decay. This intent shows up in the drift function, attention envelope, absence invariant, and the Tamagotchi non-goals.

- Notice, never announce. The plan bans "toasts, banners, welcome text, badges, streaks, visit counts, push" and structures arrivals, notebook, top bar, offers, and visit notifications around "noticing, not announcing." The same intent appears in copy lint, the welcome-back canary, and the decision to hide newcomer adoption in Settings -> Birds without a badge.

- Permanent bird identity. A bird has a permanent `bird_id`; voiceprint and markings are immutable; drift never touches "identity features." This principle appears in the invariants, bird schema, personality model, audio voiceprint design, restore tooling, and anti-homogenization risk mitigations.

- Personality numbers stay out of product surfaces. The plan says "Personality numbers never reach a product surface"; clients receive only "quantized render parameters," captions and narration avoid state tokens, and the one exception is the account export file. This shows up in the snapshot schema, trait mappings, copy lint, accessibility descriptions, and export decision.

- Procedural, state-derived sensory surfaces. Calls, chorus, ambient bed, weather audio, captions, and narration are all generated from state or synthesized parameters. The plan says "Calls are procedural," "no recorded call or ambient audio," captions come from "the exact parameters sent to the synth," and narration reads "rendered state."

- A living aviary first. The first frame must be "the aviary, already in motion"; there is "no spinner, entry animation, or fade-from-static." The boot path, edge-inlined snapshot, no-white-flash sky, first-bird metric, and no-entrance rule all serve this.

- Accessibility as part of the product, not a later mode. Accessibility "ships in v1," reduced motion is "a designed surface," and manual testing asks "does it feel alive?" This appears in the keyboard model, bird proxies, captions, narration, contrast checks, RM pose libraries, and milestone staffing.

- Quiet, matter-of-fact system voice beside naturalist prose. The plan splits surfaces: aviary, offers, notebook, captions, and narration use a naturalist voice; sign-in, sessions, deletion, errors, visits, and accessibility settings use "matter-of-fact" English. Copy lint enforces lowercase, no exclamation, no trait labels, and no user-behavior facts.

- Privacy boundary as architecture. The plan treats privacy as topology and schema, not just policy: email lives in "exactly one place"; analytics has "no network route and no credentials" to the simulation DB; RUM is "aggregate only"; metrics reject account, bird, species, mood, and trait labels.

- Local immediacy under server authority. The client may run shared planners for "zero-latency response" on greetings and offers, but the server re-resolves and credits drift. This shows up in the shared `engine-core`, offer nonces, snapshot plus script boundary, and reconciliation rules.

- Calibration over guesswork. Numeric constants are "starting values"; engine params are versioned; `drift-lab`, perceptual JND validation, a consented panel, and forward-only parameter changes are all used so behavior can be tuned without back-filling vectors or observing production per-bird data.

## Per-feature whys

### 1. Scope

- Magic-link sign-in: The plan uses magic links with 15-minute, single-use tokens, 202 responses, fragment callbacks, and atomic consumption to avoid account enumeration, avoid link-scanner consumption, keep tokens out of server/CDN logs, and make replays invalid.

- Per-device sessions, listed and revocable: The plan exposes device labels and revocation so account access can be inspected and ended per device; labels are coarse browser/OS family only, and revocation takes effect on the next request with a short cache TTL.

- Email change with verification: NOT RECOVERABLE FROM PLAN.

- JSON export by emailed link: The export exists as the explicit data-portability exception to hiding personality vectors; the plan requires a signed-in download and says "No bearer URLs."

- Soft delete, restore, then hard delete: Soft deletion keeps the aviary ticking so "a restore finds living birds"; hard deletion removes simulation data, object exports, sessions, and account data, with backup expiry stated for privacy.

- Synthetic account UUIDs: The plan uses synthetic UUIDs because "email lives in exactly one place"; every other reference uses the account UUID, and account IDs are never derived from email.

- One aviary per account: NOT RECOVERABLE FROM PLAN.

- Two starter birds: NOT RECOVERABLE FROM PLAN.

- Age-based arrivals up to a cap of seven: The schedule "never depends on visits, presence, or interaction," so new birds do not become a reward or pressure mechanic. The cap is enforced because later perf, layout, and recognizability work is designed around at most seven birds.

- Species pool of six, including one nocturnal species: The nocturnal species supports the plan's intent that "Night is not dead"; starter species are chosen for "timbral contrast," while the plan explicitly says "Rarity is not a feature."

- User-assigned names, renameable any time: NOT RECOVERABLE FROM PLAN.

- Server-side engine tick: The tick is the single writer for personality and canonical state, making personality monotonic, event consumption deterministic, and recovery replay-safe.

- Mood system: Mood gives fast-timescale expressiveness while personality remains monotonic; the absence invariant prevents time since last presence from raising `wary`.

- Perch selection: The plan says users have no input into perch placement because "It is signal, not layout"; perches express boldness, mood, warmth, attention envelope, and proximity.

- Call planning, bird-to-bird response, and chorus: Server planning coordinates timing and intensity, replies "stagger," and chorus events avoid exact overlap while keeping bird calls procedural and social.

- Weather: Weather is scheduled and stored per aviary so "both devices and visitors see the same weather"; rain and wind also shape mood, call rates, and scene feel.

- Circadian cycle in account local time: The canonical timezone makes every device and visitor see "the same aviary in the same mood"; timezone changes ease over 30 minutes.

- Return-greeting: The planner runs client-side for "zero network latency on the anchor moment"; longer absences lower the threshold so more of the aviary may come around, while anti-repetition keeps greetings from feeling canned.

- Listen-in: The mix ramp is meant to feel like "attention shifting, not a switch"; listen-in also feeds warmth and vocal-frequency drift for the focused bird.

- Offer: Offers are treated as gestures toward the aviary. They shape curiosity and boldness through server-credited resolution, while no disabled state or countdown keeps the offer from becoming "a button with a timer."

- Settle with a 5-second undo: The plan treats settle as "a goodbye from this session," not a canonical aviary state; undo before 5 seconds sends no event, and another device should not dim.

- Presence accounting: Presence requires the three-signal conjunction because "watching without moving is the product" but visibility and focus remain mandatory. Server-side union across devices prevents multi-tab or multi-device inflation.

- Field notebook: The notebook is sparse and read-only so it stays observational rather than gamified; its token bucket gives "the same rhythm" to heavy and light users, with no unread badge, "new" marker, or count.

- One horizontal scene with no panning: NOT RECOVERABLE FROM PLAN.

- Top bar that fades: The fade keeps controls from dominating the scene, while focused elements restore full opacity and an accessibility setting can "keep the top bar visible."

- Client-side procedural audio: Procedural synthesis enforces the no-recorded-audio invariant and lets voiceprints, motifs, captions, and bird motion derive from the same call parameters.

- Optional read-only social visits: Visitors are "render-only"; separate credentials, endpoints, and tables ensure visitors never influence the host's tick, presence, or notebook.

- Visit-notification email, off by default: The plan treats this as "the single named exception" to not emailing the user about the aviary.

- Naturalist screen-reader narration: Narration is generated from rendered state so it matches what a sighted user sees and feels like prose rather than a state list.

- Reduced-motion mode: RM is "a designed surface" that preserves calls, captions, narration, drift, mood, notebook, and greetings while replacing motion with key poses and cross-fades.

- Call captions: Captions are generated from the actual synth parameters so text matches the heard or muted call; they are `aria-hidden` to avoid double-reading for screen-reader users.

- Full keyboard model: The keyboard model makes the aviary operable by focus and keys; engaging listen-in on `Enter` avoids sweeping the mix around during arrow navigation.

- Performance and operations budgets: The budgets exist to satisfy the first-bird, 60 fps, memory, bundle, snapshot, and tick guarantees, with CI and release gates preventing regressions.

- Matter-of-fact unsupported-browser page: NOT RECOVERABLE FROM PLAN.

- Native apps out of v1: The plan says "The protocols are designed for the web only."

- Gamification out of v1: Achievements, streaks, levels, counters, calendars, and "days visited" are excluded because the product intent is "Notice, never announce" and because such counts could become streak disguises.

- Tamagotchi mechanics out of v1: Hunger, death, distress, and decaying happiness are excluded so neglect cannot cause distress or guilt; the engine has "no state that can express distress caused by neglect."

- Social-network surfaces out of v1: Profiles, follows, feeds, comments, chat, avatars, leaderboards, co-presence, and show-off rendering are excluded, and "the metrics that would drive leaderboards are never computed."

- Push and re-engagement email out of v1: Push, web push, and re-engagement email are excluded by the anti-announcement principle; only transactional email and the opt-in visit notification remain.

- Payments, shared/team aviaries, multiple aviaries, customizable scenes, user-placed birds, species catalog, SSO, and passwords out of v1: NOT RECOVERABLE FROM PLAN.

- Engagement analytics out of v1: The plan avoids DAU, retention cohorts, funnels, streak-like distributions, and engagement A/B tests to prevent telemetry scope creep and preserve aggregate-only measurement.

### 2. Architecture

- TypeScript across the stack: The stated reason is a shared, pure `engine-core` package so server tick, client reactive behaviors, drift lab, and harnesses all use "exactly one implementation."

- `engine-core` as pure functions with injected time: Purity makes tick replay deterministic, enables calibration harnesses, and avoids hidden I/O or `Date.now()` differences between server, client, and lab.

- Client receives quantized render parameters, not raw traits: The split supports the invariant that raw personality numbers never reach product surfaces while still letting the client render expression.

- Client-side reactive planners: Greetings, offer reactions, and listen-in acknowledgments run locally for "zero-latency response," seeded by server nonces so the server can still remain authoritative.

- Snapshot plus 180-second script boundary: The server says what is true and what is planned; the client decides how it looks and sounds. If the horizon runs out, birds idle in their last valid state, which the plan calls "always a valid, living state."

- Edge worker and regional snapshot cache: These exist to inline the snapshot and hit the first-frame budget; cache failure falls back to DB rebuild rather than blocking the model.

- Separate metrics/RUM collector: It is "one-way" and has no simulation DB credentials, which implements the privacy boundary structurally.

- Service worker with shell/static asset cache and no push: The service worker improves returning-load performance while preserving the no-push/no-notification product rule.

### 3. Data model

- `account_id` as shard key and never derived from email: This supports the plan's email-minimization rule and lets all simulation tables key by UUID rather than PII.

- Email blind index: The blind index exists "only so sign-in can find the account"; it is never logged, never a foreign key, and never leaves auth.

- Permanent `bird_id`: Permanent bird identity prevents reset or replacement; migrations may not rewrite personality except through the tick.

- Immutable voiceprint and appearance: These preserve individual identity while personality drifts; the audio section says voiceprint is what makes "Pip recognizably Pip."

- Species starter selection: Starters are distinct diurnal species "chosen for timbral contrast," so the first pair sounds recognizably different.

- Arrivals drawing from species not yet present first: NOT RECOVERABLE FROM PLAN.

- Per-account event sequence: The row-locked sequence avoids the bigserial problem where a tick could read seq 105 before seq 104 commits and "skip 104 forever."

- Interaction event retention after tick consumption: The plan keeps consumed events only briefly, matching the privacy/data-minimization direction; the tick cursor is the durability boundary for simulation.

- Visitor email storage in visit log: The plan allows this one second-email location because "The host's visit log shows visitor emails"; it is encrypted and deleted with the host account.

- Export object 7-day lifecycle: NOT RECOVERABLE FROM PLAN.

### 4. API surface

- `POST /v1/auth/magic-link` always returning 202: This prevents account enumeration.

- Auth callback token in URL fragment: Mail gateway GETs cannot consume the token, and the token does not appear in server or CDN access logs.

- Atomic magic-link consume: Atomic update makes tokens single-use; zero rows become `link_invalid`.

- Session list and revocation: Revocation is intended to take effect quickly, with cache TTL no more than 30 seconds.

- Account export download requiring sign-in: The plan explicitly says "No bearer URLs," so an emailed export link is not itself enough to download data.

- Snapshot endpoint with ETag and regional cache: The ETag avoids resending unchanged snapshots, and regional cache supports the fast first frame.

- Event ingestion idempotency and validation: `client_event_id` prevents duplicates; validations reject stale, impossible, unknown-bird, or visitor events so client events cannot corrupt canonical state.

- Visitor credentials scoped to visit endpoints: Separate cookies and middleware make visitor credentials unable to reach `/v1/aviary/*`; this preserves read-only visits.

- Visitor snapshot stripping host-only fields and controls: Removing greeting context, offer nonce, cooldown, notebook, and adoption controls keeps visits render-only.

- Visitor heartbeat separate from presence: It updates only `visit_log.last_seen_at`; the tick never reads visit tables, so visitor traffic cannot shape the host's birds.

### 5. Simulation engine

- Tick phases by hash within the minute: Hashing the phase spreads load evenly.

- Sharded tick leases with fencing tokens: Leases and optimistic `state_version` checks prevent split-brain ticks and stale writes.

- Pure deterministic tick step: A crash before commit replays the same events against the same prior state; the plan says this is safe because the step is pure and deterministic.

- Invariant rollback and aviary hold: On invariant failure, the plan leaves state untouched; persistent failure serves the last good snapshot while birds idle client-side.

- Catch-up substeps after outages: Substeps, closed-form updates, and local-day budgets keep results stable regardless of step size and prevent catch-up from exceeding real-time drift.

- Dormant aviaries still ticking: The plan says the PRD requires it; batching and skipped tiny writes keep the cost low while mood timers remain absolute.

- Low adoption seeds: Seeds sit low "so each bird has room to drift."

- Per-bird ceilings: Ceilings are "the main defense" against every long-lived bird converging to the same maxed-out profile.

- Voiceprint and markings separate from personality: Drift never touches identity features, so expressive change does not erase recognizability.

- Two-stage low-pass drift with saturation and daily budget: The filter makes attention have a gentle after-effect, saturation gives diminishing returns, and the daily budget guarantees no single session can visibly move a trait.

- `drift-lab` calibration harness: The harness turns "measurable after about 1 week" and "visible after about 3 weeks" into assertions, plots, and CI failures.

- Forward-only engine parameter versions: Parameter changes apply only going forward because "Vectors are never recomputed or back-filled" under new params.

- Attention envelope: The envelope resolves the tension between ignored birds becoming "ambient" and traits never moving down; it is reversible, not valence, and its floor keeps every bird alive and calling.

- Mood absence invariant: Absence length does not feed arousal, valence, or impulses, and a property test asserts `wary` is independent of time since last presence.

- Mood persistence: Mood is stored and ticked continuously so "Opening a tab never resets it."

- Mood contagion by proximity: Bird-to-bird effects are gated by proximity, "so the perch layout matters."

- Perch selection: Perch scores express boldness, mood, warmth, envelope, and noise; user input is excluded because perch choice is "signal, not layout."

- Call/response staggering: Replies are offset by 0.6-2.5 seconds "so replies stagger."

- Chorus planning: The planner promotes compatible call windows to chorus and plans interleaved calls "to avoid exact overlap."

- Weather scheduler: Weather is stored 24 hours ahead so devices and visitors share the same weather.

- Return greeting primary/additional greeters: The threshold falls as absence grows, so longer absences "bring more of the aviary around" while lower-scoring birds may not greet.

- Greeting form as continuous blend: Continuous parameters, signatures seeded from `bird_id`, and anti-repetition make exact repetition practically impossible.

- Hidden-to-visible under 20 seconds not greeting: The plan says this is "not a return."

- Arrival schedule by aviary age only: This prevents arrivals from depending on visits, presence, or interaction.

- Newcomer as shy visitor in back zone: The scene and notebook let the user notice the newcomer before adoption without a badge or prompt.

- Newcomer adoption in Settings -> Birds: The plan's stated why is "Noticing, not announcing. The user chooses without being prompted."

- Offer client immediate resolution and server re-resolution: The client gives immediate display; the server result credits drift, and rare divergence is "cosmetic and never corrupts state."

- Offer cooldown without disabled state or countdown: It keeps the offer "a gesture, not a button with a timer."

### 6. Sync model

- Single canonical state in Postgres: The model has "no client state to merge and no last-write-wins anywhere."

- Snapshot inlined on navigation: Inlining avoids an extra request on first load.

- Visible-return snapshot pull: Pulling immediately on `visibilitychange -> visible` refreshes stale state before reconciliation and greeting.

- Visitor polling every 30 seconds: The plan says this bounds revocation latency.

- Clock sync from server time and RTT: Scripts and call plans use server time so two devices side by side show the same calls within about 100 ms.

- Reconciliation by flights and blends: Birds "never teleport"; a flight reads as aviary motion rather than correction.

- Long-hidden relocation sequence: Staggered "birds shift around as you look" movement avoids all birds snapping at once after stale hidden periods.

- IndexedDB event outbox: The outbox supports offline buffering, idempotent event IDs, batched flushing, backoff, and `pagehide` delivery.

- Presence union across devices and tabs: Two tabs or a laptop and phone watching at once "count once"; correctness does not depend on leader-tab optimization.

- Event validation rejection surfacing none: The plan says no user surface is shown because "the user did nothing wrong."

- Tick quarantine surfacing none: The last good snapshot keeps serving and "birds idle normally," avoiding an alarming product surface.

### 7. Frontend rendering pipeline

- Inline quiet-field sky gradient: The first paint shows "the right sky" with no spinner and no white flash.

- Inline snapshot with timeout fallback: The edge worker inlines the snapshot for speed but streams without it after 120 ms so the boot path does not block indefinitely.

- Critical/species chunk split: Loading only the species chunks needed by the aviary keeps the critical path small.

- Front-most/greeter bird rasterized first: This supports the first-bird-visible target while keeping birds already "mid-action," not appearing as an entry.

- Deferring audio, voice, notebook, offer, settings, and visits: These are delayed until idle or demand so the first frame is not slowed.

- Canvas2D with pre-rasterized part sprites: The rationale is that at most seven birds is about 70 `drawImage` calls; Canvas2D avoids shader compilation and context-loss handling on the critical path.

- WebGL2 renderer behind the same interface: The fallback exists if the phase-0 spike cannot meet frame budgets with Canvas2D.

- DOM overlay for captions, focus rings, and screen-reader proxies: These surfaces need DOM behavior and accessibility semantics rather than being painted only into canvas.

- Sprite re-rasterization only when `sat_q` changes: Plumage saturation changes rarely, so this keeps rendering cheap.

- Calling animation driven by actual synthesized envelope: Beak-open and throat pulse stay in sync even when audio is locked or muted.

- Entrance exceptions only for starter birds and newcomers: This protects the "first frame is the aviary" rule while honoring the new-account and arrival moments.

- Layout templates and safe-inset solver: The solver guarantees seven birds fit without overlap or crop across tested viewport sizes.

- Dynamic quality governor: Quality drops background DPR, particles, then sway when frame-time p95 is too high, preserving smoothness before content.

- Trait-to-visual mappings: These are "the only ways personality shows," keeping raw traits off product surfaces.

- Day/night lighting in canonical timezone: The account timezone gives a shared aviary time, while seasonal shift uses timezone country so no location data is collected.

- Night not dead: The plan keeps nocturnal activity, ambience, occasional shifts, and drifting leaves so night remains alive.

- Settle lighting/audio behavior: The local grade, audio, and call thinning create a quiet goodbye state; deliberate click/tap/key re-engages, while pointermove alone does not.

- Reduced-motion presenter: RM samples the same animation state into key poses and cross-fades so reduced motion remains the same living system, not a static fallback.

- Hidden tab behavior: Stopping rAF, suspending audio, ending presence, and reducing timers target "Background tabs cost 0%."

### 8. Audio pipeline

- One preallocated AudioWorklet: Constant node count and reusable buffers satisfy "no per-call allocation" and avoid per-call `OscillatorNode` or `AudioBufferSourceNode`.

- Main-thread lookahead scheduler: It converts server time to audio time so scheduled calls align with the shared script.

- Per-bird immutable voiceprint: Pitch offset, timbre bias, motif weights, signature interval, and rhythm swing make "Pip recognizably Pip."

- Bounded mood and trait modulation: Modulation changes tempo, amplitude, jitter, phrase length, and trill rate without touching identity features.

- Per-rendition micro-variation: Seeded continuous variation means "a user never hears the same call twice exactly."

- Anti-uncanny rules: Attack minimums, jitter, spectral bounds, no human-like formants, and level caps avoid clicks, startles, robotic repetition, and human speech resemblance.

- Song-fragment offers with distinct offered timbre: The sound is "softer, rounder" so it reads as "a gift, not another bird."

- Chorus as real simultaneous voices: This avoids loops and phase-cancel artifacts.

- Chorus gain rider: It keeps loudness steady when more than three voices overlap.

- Listen-in mix ramps: Slow ramps and a floor for other birds make it feel like attention shifting; other birds are "never muted."

- Procedural ambient bed: Wind, leaf rustle, rain, and night texture are procedural so no sample or recorded-audio path exists.

- Autoplay unlock on first activation anywhere: A visible sound button would be "an announcement"; using the first click/tap/key lets audio arrive naturally.

- Locked-audio behavior: Calls still animate beaks and captions still show, keeping the aviary coherent before audio unlocks.

- WebAudio fallback to silence with captions on: The plan preserves accessibility and the no-recorded-audio invariant when audio fails.

- Mute in accessibility settings: Muting has "no negative consequence"; the small `heard` drift channel honors the brief without punishing a muter.

### 9. Naturalist voice system

- Typed generative grammar instead of LLM: Hosted LLMs would send per-bird state to a third party, self-hosted models risk voice drift and hallucinated observations, and grammar is testable by lint.

- Copy lint style rules: Lowercase, no exclamation, banned gamification terms, no trait labels, and no user-behavior facts enforce the quiet naturalist voice and anti-announcement rules.

- Notebook observations about aviary facts: Observations cover greetings, perch shifts, chorus, weather behavior, preens, offers, pool bathing, night calls, newcomers, and appearance changes, never user-behavior facts or raw numbers.

- Notebook salience and sparsity: Salience plus token bucket keep entries sparse; a heavy user gets "the same rhythm as a light user."

- Notebook repetition guard: Rejecting recent template or n-gram matches prevents repetitive entries after months.

- Notebook UI without unread badge or count: This keeps the notebook observational and read-only rather than a notification or engagement surface.

- Captions generated from synth parameters: The caption text matches syllable count, contour, gaps, amplitude, and zone from the actual rendered call.

- Caption contrast sampling: The pill backdrop is computed from scene luminance so text maintains WCAG contrast over bright or dark scene regions.

- Captions `aria-hidden`: Screen-reader users receive calls through narration, avoiding double-reading.

- Screen-reader narration from rendered state: Narration matches what is visibly happening rather than exposing snapshot state.

- Polite live region and cadence: A single atomic polite region, idle cadence, debouncing, and event prioritization avoid bursts and state-list chatter.

- Always-active live region: The plan says screen readers cannot be detected, so the live region is always active while sighted users are unaffected.

### 10. Accessibility surfaces

- Accessibility shipping in v1: The plan says accessibility ships "with the rest of the product" and is "not after."

- Roving keyboard model over birds: The scene is one tab stop with roving focus so keyboard users can navigate birds spatially.

- `Enter`/`Space` to listen-in rather than arrow focus: The decision avoids channel-switching feel and preserves the slow-ramp intent.

- Focus follows identity: If the focused bird flies, the ring travels with it so focus is not lost when birds move.

- Bird DOM proxy elements: Proxies support keyboard focus, pointer hit-testing parity, and touch exploration with VoiceOver and TalkBack.

- No `role="application"` on the scene: The plan says this keeps screen-reader browse modes working.

- Naturalist accessible descriptions: Descriptions avoid state tokens, numbers, and mood labels while still describing where and how the bird appears.

- Two-tone focus ring: The two tones guarantee contrast against day and night backgrounds; it shows only for `:focus-visible`.

- Expanded touch hit targets: The plan uses at least 44 x 44 CSS px targets for pointer/touch accessibility.

- Tap in top 15% revealing top bar: This lets touch users reveal controls without also disengaging listen-in.

- Top-bar keep-visible setting: It exists for users who need persistent controls when the default fade would hide them.

- Accessibility testing with disabled panel: The manual gate asks not only whether users can operate it, but "does it feel alive?"

### 11. Accounts, security, privacy

- Magic-link token hashing, expiry, and single use: Random tokens are stored only as hashes, expire quickly, and are consumed atomically.

- Cross-device magic-link use: NOT RECOVERABLE FROM PLAN.

- Session token hashing and coarse device labels: Tokens are stored hashed, and device labels use browser plus OS family only.

- Coarse `last_seen_at` display: NOT RECOVERABLE FROM PLAN.

- Email encryption and blind indexes: Envelope encryption protects stored emails; the separate HMAC blind index supports auth lookup without making email a general key.

- Structured logging allowlist and PII scans: Unknown fields are dropped, and tests/scanners fail or alarm on email patterns and bird names.

- Export includes personality vectors: The plan resolves the tension by saying export is the data-portability exception, not a product surface.

- Export excludes presence, sessions, heartbeats, and interactions: The plan says the PRD bans "an exportable visit log" as a streak disguise.

- Pending deletion strip: It appears in the top bar region with matter-of-fact text so the deletion state is visible without covering the scene.

- Hard deletion order and backup expiry: The order removes account-related simulation and access data, while backups expire within the stated window.

- Metrics needing no deletion: RUM and metrics hold no account identifiers, so there is nothing account-specific to delete there.

- Simulation DB private network boundary: Only product services have credentials; analytics has no route or credentials, enforcing the privacy boundary.

- Metric schema registry: Allowlisted labels and forbidden account/bird/species/mood/trait labels prevent telemetry scope creep at compile time.

- No ML training pipeline: If one is proposed, the written rule is that it receives no per-bird fields.

### 12. Product surfaces and voice split

- Aviary scene with no text: The scene stays "birds and place only," preserving the product's non-announcing first surface.

- Four-icon top bar: The plan keeps the top bar minimal; settle is placed inside the offer menu to keep "exactly 4 icons."

- Offer panel naturalist labels: The labels are gestures toward the aviary: "a seed," "a song fragment," "a still pool," and "settle the aviary."

- Adoption and starter naming voice: Naturalist copy introduces the birds, while the name field label stays matter-of-fact.

- Account, auth, deletion, errors, visits, accessibility settings voice: These use matter-of-fact language because they are system surfaces.

- Anti-announcement lint and canary tests: Banned imports, copy review, and welcome-back tests prevent "just one" toast, counter, or streak from creeping in.

- New-account naming flow over quiet field: The required naming clicks unlock audio; starter birds then fly in, and "after this, the empty aviary never appears again."

- Suggested starter names pre-filled: NOT RECOVERABLE FROM PLAN.

### 13. Performance budgets and observability

- Bundle and critical-path budgets: CI-enforced size budgets protect the first-frame and PRD hard-cap requirements.

- First-bird-visible metric: The metric measures when at least one bird was drawn and confirmed through the next rAF, tying instrumentation to the first living frame.

- Memory slope and constant-node tests: Preallocation, virtualization, bounded outbox, one audio context, and fixed pools enforce "zero memory growth" over long sessions.

- Capacity sharding path: The design scales from 100k to 1M aviaries with more tick pods, partitioned Postgres, and Redis cache "without protocol changes."

- Aggregate-only RUM: It records performance and failure buckets without account id, session id, or cross-event join key.

- Synthetic checks: Dedicated synthetic accounts test first-bird time, snapshot freshness, audio graph construction, and sign-in from multiple geographies.

- Deliberately unmeasured engagement metrics: The plan refuses DAU, cohorts, funnels, listen counts, notebook opens, and engagement A/B tests to avoid engagement analytics and product-principle erosion.

- Drift calibration outside production data: Calibration uses `drift-lab` and a consented panel, "never production data."

### 14. Rollout

- Staging with time-travel clock injection: Time travel exercises aviary age and accelerated ticks before production naturally reaches later bird counts.

- Separate panel environment: The panel has consent and a separate database, allowing inspection of calibration data without production per-bird observation.

- Calibration panel beta lasting at least eight weeks: The plan requires enough time to observe the three-week target twice before GA.

- Panel exit criteria around no distress or guilt: The panel must find no reports of distress or guilt after absences, matching the non-Tamagotchi intent.

- GA waitlist ramp: Each ramp step is gated on tick p99, snapshot p95, first-bird p75, error rates, and email deliverability.

- High-bird staging and panel tests: Production will not naturally see 3-bird aviaries until about 90 days or 7-bird aviaries until about 18 months, so those paths must be proved earlier.

- Arrival kill switches and max count flag: Pausing arrivals "delays arrivals and loses nothing"; lowering the cap can delay high-count exposure but "never removes a bird."

- Forward-only engine parameter changes after launch: Changing drift rates is treated like a database migration, with `drift-lab` reports and no vector recomputation.

- Day-one instrumentation checklist: Alarms, PII scanning, restore drill, email dashboards, synthetic checks, and snapshot cache metrics must exist before launch.

### 15. Workstreams and team

- Dedicated engine, client, audio, voice, backend, accessibility, perf, and design workstreams: The plan allocates ownership so each cross-cutting discipline has named deliverables.

- Accessibility, RM, captions, and narration in every milestone: They are "not a phase of their own," preventing accessibility from being deferred.

### 16. Risks

- Drift too fast mitigation: The hard daily budget, binger persona, and week-1 survey prevent change from feeling like a Tamagotchi.

- Drift too slow mitigation: Week-3/4 surveys, perceptual JND validation, and render-side mapping tuning prevent the aviary from feeling like a screensaver.

- Long-term homogenization mitigation: Ceilings, headroom scaling, channel-specific weights, immutable voiceprints/markings, and the `year` persona preserve individual differences.

- Presence inflation mitigation: Three-signal presence, server wall-clock clamps, cross-device union, leader heartbeats, counters, and the daily budget bound spoofing.

- Ambient state misread as sadness mitigation: Envelope floor, valence decoupling, absence-independence tests, and panel interviews protect the return experience.

- Lost events and split-brain tick mitigations: Per-account sequences, atomic cursor commits, lease fencing, deterministic pure steps, chaos tests, and replay-equality tests protect sync correctness.

- Personality loss/reset mitigation: Single writer role, monotonic triggers, restore points, PITR, immutable identity triggers, migration lint, and restore drills address the plan's "worst failure."

- Audio uncanniness mitigation: Sound designer ownership, anti-uncanny rules, reference listening, and panel "alive vs. canned" gates protect procedural audio quality.

- Seven-bird recognizability mitigation: Voiceprint separation, ABX/identification studies, and MFCC collision checks preserve individual bird identity at high counts.

- Accessibility regression mitigation: RM pose lint, screenshot baselines, copy lint, cadence tests, screen-reader matrix, identity-bound focus, contrast tests, and remappable shortcuts guard accessible behavior.

- Product-principle erosion mitigation: Banned components, welcome-back canary, copy-review labels, invariant PR templates, and metric schema restrictions prevent toasts, counters, streaks, repetitive notebook, and telemetry scope creep.

- Performance/platform mitigation: Edge-inlined snapshot, tiny critical path, SW caching, HTTP/3, SVG first-frame contingency, WebGL2 fallback, quality governor, email deliverability work, and tick sharding address launch risks.

### 17. Decisions made where the plan records ambiguity or tension

- Activity signals: `pointermove`, `pointerdown`, and `keydown` are used because taps are pointer activity and "watching without moving is the product," while visibility and focus remain mandatory.

- Keyboard listen-in trigger: Arrow focus moves focus and `Enter` engages because engaging on every arrow press would feel like channel switching and conflict with the slow-ramp intent.

- Export personality vectors: The export includes vectors because it is explicit data portability, while the no-exposure rule targets product surfaces.

- Export exclusions: Presence, session, heartbeat, and interaction histories are excluded because an exportable visit log could become a streak disguise.

- Settle in offer menu: Settle lives in the offer affordance because both are "gestures toward the aviary" and this keeps the top bar at exactly four icons.

- Canonical timezone: One account timezone lets every device and visitor show the same aviary in the same mood.

- Per-device settled state: Settle is local because it is "a goodbye from this session"; another device should not dim.

- `roosting` mood name: The name avoids collision with aviary-level "settled."

- Visit one-time link and persistent visitor session: The link is single-use, but the session persists until revoked or 30 days idle, matching "one-time link" while allowing repeated visits and a natural abandoned-access end.

- New-bird offer without announcing: Newcomers appear in scene, may be noted in the notebook, and adoption is in settings so the user notices and chooses without being prompted.

- Offer cooldown UI: No disabled state or countdown keeps the offer a gesture rather than a timer.

- Mute as drift input: Unmuted presence feeds a small `heard` channel and muting never subtracts, honoring the brief without punishing muting.

- Mute location: Mute lives in accessibility settings and the `m` shortcut because the top bar is capped at four icons.

- Visitor access to captions and narration: Visitors get local a11y but no notebook, listen-in, or bird focus because visitors are render-only and "a11y remains a right."

- Visit notifications: The opt-in, off-by-default visit email is allowed because it is the named exception.

- Seasonal daylight without location: Month plus hemisphere inferred from IANA timezone gives a "felt day" without collecting location.
