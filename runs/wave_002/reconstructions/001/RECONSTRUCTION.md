## System-level intent

- **The aviary should already be alive on first sight.** This is named in §0 as a trap: "the first frame is the aviary already in motion, never a spinner-then-fade." It recurs in §2.4, where the snapshot boundary makes "the aviary was already running" true, in §5.1 where `/open` shows the aviary "as it is now, never as it was," and in §7.1/§7.9 where the first bird appears mid-action and there is "No spinner, no progress, no logo animation."

- **The product notices without announcing.** §0 says "nothing on any surface announces the user, counts their visits, or shows a trait number." §1.2 excludes "streaks, scores, badges, levels, counters, calendars, milestone celebrations." §1.3 adds "No announcement primitives," and §5.11 says, "No text accompanies the greeting anywhere. The greeting is the welcome." The same principle appears in §5.14: "No prompt, badge, or toast points at it; the bird being there is the offer."

- **Product prose is naturalist observation; system prose is plain system copy.** §0 says "Product-surface strings are lowercase naturalist prose" while "System-surface strings... are matter-of-fact English." §5.15 keeps notebook prose "lowercase, present tense, bird-named," while §6.5 uses matter-of-fact error copy. §9.6 likewise says accessibility settings use "Plain form controls, plain labels, no naturalist phrasing."

- **The user is not the subject of the aviary.** §5.15 says the notebook scorer "refuses any candidate whose subject is the user" and cannot say "you were here every day this week." §10.5 refuses "interaction funnels," "listen-in durations," and "visit counts as a product metric." §4.7 makes visits "never pushed, never badged."

- **Birds are relationships, not numbers.** §1.3 requires "Trait containment" and asserts no float trait appears in API fixtures. §3.8 says snapshots never contain raw traits. §15.1 resolves export vs. "never see the numbers" by putting the vector in an opaque `engine_state` blob. §13 names "Trait leakage" as making the "bird becomes a number."

- **Personality moves only gently toward expression.** §5.4 calls drift "Monotonic toward expressive": absence contributes zero and "Neglect changes nothing in the vector." §5.5 handles quieter returns through hidden `attunement`, so the return after a fortnight is "quieter, then easing back," never "punished." §13 frames bad calibration as too fast becoming "Tamagotchi" and too slow becoming "screensaver."

- **There is one canonical aviary and one writer.** §6.1 says "There is no client state to merge" and the server is the "only writer of personality, mood, perch, attunement, weather, notebook, and call plans." §5.1 makes this mechanical with `withAviaryLock`; §6.5 says there is "no conflict-resolution UI, because there is nothing for the user to resolve."

- **The snapshot is the product boundary.** §2.4 says "The boundary is the snapshot." Everything left is deterministic engine code; everything right is presentation that can "start mid-action from any snapshot." §2.3 gives the rule: shared state crosses the wire; legitimate differences such as "micro-motion phase, leaf timing, jitter" stay client-local.

- **The aviary is deterministic but not canned.** §2.6 defines seeds and named RNG purposes so a tick reproduces exactly. §5.9 derives each bird's `Signature` once so calls stay recognizable while per-call jitter means "no call is ever repeated exactly." §5.11 uses mood, absence buckets, seed variation, and noise so greetings do not read as canned.

- **Presence means watching, honestly counted.** §5.3 says presence is "the user is watching," not "devices are open," and overlaps count once. §1.3 requires a "Presence honesty" e2e test for visible/focused/active. §15.5 sets a long activity window because "watching without moving is the product."

- **Accessibility is part of the designed surface, not a later pass.** §7.8 calls reduced motion "a designed surface" and says calls, captions, narration, drift, mood, notebook, and interactions behave identically. §9.7 says accessibility work is "part of each milestone's exit criteria, not a later phase."

- **Performance carries the affective promise.** §10.2 ties the first-bird budget to inline snapshots, quiet field, meshes before textures, and `/open` in parallel. §16 defines done as a fresh account reaching first bird within 500 ms and never seeing "a spinner, a toast, a counter, or a number about a bird."

- **Privacy boundaries are structural.** §1.3 includes a telemetry allowlist; §10.5 says the analytics warehouse has "no credential to the sim database"; §10.7 drops non-allowlisted log fields so an engineer cannot accidentally log "an email, a name, a mood, or a trait." §11 keeps email encrypted and tokens opaque.

- **Social exists only as quiet read-only visiting.** §1.2 excludes "profiles, follows, feeds, discovery, comments, co-presence, leaderboards." §4.7 makes visitor sessions read-only, with no `/open`, no `/events`, no notebook, and no visitor presence. The visitor snapshot is byte-identical so there is no "`show-off` rendering path to drift."

## Per-feature whys

### How to read this plan and scope

- **Vocabulary following `concepts.md` and internally defined terms:** The plan keeps bird, aviary, call, mood, personality vector, drift, presence, listen-in, offer, settle, field notebook, visit, and tick consistent, while internal terms "never appear on a user-facing surface."

- **Product-surface/system-surface string split and voice linter:** The split is enforced by CI, "not by reviewer memory," so lowercase naturalist prose and matter-of-fact system copy do not drift.

- **No announcement primitives:** §1.3 says the PRD expects well-meaning contributors to violate this, so the design system omits `Toast`, `Banner`, `Badge`, `Confetti`, and notification dots, and lint blocks those primitives.

- **One horizontal, non-scrolling scene with three perch zones:** NOT RECOVERABLE FROM PLAN

- **Day/night from local time:** §15.13 chooses a fixed local schedule with seasonal tilt "rather than geolocation," avoiding a location permission and keeping a testable function; §7.6 lets lighting be client-local while the server uses account timezone for mood priors.

- **Ambient leaf/feather drift:** NOT RECOVERABLE FROM PLAN

- **Sparse top bar that fades:** The bar stays to "Four icons only," with no badges, dots, counts, or tooltips; sheets "never cover more than 60% of the viewport so the aviary stays visible behind them."

- **Two starter birds at adoption:** §15.12 says starters are two distinct diurnal species so "first encounters happen in daylight with two active birds" while the night-active species arrives later.

- **Cap of seven birds:** §13 says recognizability at seven matters because "per-bird relationship collapses" if it fails; §12.4 validates layout, chorus, and listening before any real user reaches bird 3.

- **Six-species pool including one night-active species:** §15.12 says the night-active species means "night is not a dead state" while starters remain diurnal.

- **User-named, renameable birds:** NOT RECOVERABLE FROM PLAN

- **Stable bird identity forever:** §1.3 makes bird identity immutable, and §13 treats trait or identity leakage as risking the bird becoming a number; stable ids support the same bird relationship across drift, mood, export, and sync.

- **Server-side tick, dormant cadence, and catch-up on open:** §15.7 says per-minute ticks forever are "unaffordable," so dormant aviaries coalesce into 15-minute ticks with identical 60-second sub-steps; `/open` catch-up makes the state "exactly what per-minute ticking would have produced."

- **Hidden five-trait personality vector with expression bands:** The plan hides raw traits so no surface shows a trait number; §4.8 says a band crossing is when drift becomes visible, letting drift become visual without exposing floats.

- **Mood layer with persistence:** §5.6 says mood persists because it is canonical state advanced by tick; the "daily-ish reset" is emergent from overnight priors, not a hard reset the user could notice.

- **Procedural call grammar and bird signatures:** §5.9 says a signature stays recognizable across mood and drift, while seed jitter ensures no call is "repeated exactly"; §8.2 avoids the "layered-loop comb-filter artifact."

- **Bird-to-bird responses, chorus, and alarm:** §5.10 says these channels make the "social texture" identical on every device and for visitors; §5.9 limits response chains so chorus events do not cascade.

- **Age-gated newcomers, birds 3-7:** §5.14 ties eligibility to aviary age because it matches "an aviary a few months old offers a third bird; a year-old aviary may have grown to five or six."

- **Return-greeting:** §5.11 makes the greeting the welcome, with no text; absence bucket, mood, boldness, warmth, attunement, followers, and seed variation keep it from reading as canned.

- **Sit-and-watch presence:** §15.5 says the presence window is the long end of "a few minutes" because "watching without moving is the product"; §5.3 unions sessions so presence is the user watching, not the number of open devices.

- **Listen-in:** §15.4 says listen-in counts while muted because "it is attention"; §8.5 focuses the bird while others "never reach 0," so attention does not erase the aviary.

- **Offers: seed, song fragment, still pool:** §5.12 makes reactions immediate and canonical with a server-authored reaction plan; the offer affordance is "quietly unavailable" while active, with no timer or countdown.

- **Settle with 5-second undo:** §5.13 treats settle as terminal for the presence window, "exactly like tab-close," with no drift term and "no consequence." Pointer movement alone does not wake the aviary so the user can reach the top bar.

- **Read-only field notebook:** §5.15 uses notebook entries to make personality drift "narratable without numbers," refuses any candidate whose subject is the user, and keeps entries sparse so there is "never one per session."

- **Email magic link sign-in:** NOT RECOVERABLE FROM PLAN

- **Magic-link always-202 behavior:** §4.2 says the endpoint always returns 202 for "no account enumeration."

- **Email change with verification:** NOT RECOVERABLE FROM PLAN

- **JSON export by emailed 7-day link:** §15.1 says the opaque signed `engine_state` blob keeps the user's data "complete and portable" while still not rendering trait numbers.

- **Soft delete for 30 days, then hard delete:** §3.7 pauses ticks, suspends visits, and supports restore; hard delete is dependency-ordered and verified by a zero-row query. §11 says backups age out within 30 days after.

- **Synthetic UUIDs, encrypted email, and blind indexes:** §3.1 says the synthetic account id is "the only account identifier used anywhere"; email appears only in ciphertext columns and blind indexes, never logs, metrics, cache keys, or partitions.

- **Single canonical aviary, multi-device sync, append events, no last-write-wins:** §6.1 says there is no client state to merge; server-computed deltas and fenced locks prevent divergent devices.

- **Invite-by-email read-only visits:** §4.7 blocks visitors from `/open`, `/events`, notebook, and presence so a visit cannot alter the aviary.

- **Revocable visits, silent visit log, opt-in visit notice off:** §4.7 says the visit log is "never pushed, never badged"; §1.2 names the opt-in visit notice as the one exception to no aviary notifications.

- **Running screen-reader narration:** §9.1 writes prose from the same scene state the renderer draws, so the narrated aviary and visible aviary match.

- **Runtime call captions:** §8.8 generates captions from the actual scheduled parameters, "from what played," and §8.7 turns captions on when audio is unavailable or suspended.

- **Full keyboard model:** §9.3 makes the same actions reachable from the top bar by Tab, keeps single-key shortcuts only inside the scene region, and uses the DOM overlay as the "single source of focus."

- **Designed reduced-motion mode:** §7.8 preserves calls, captions, narration, drift, mood, notebook, and interactions, and requires pose sets so "a new action cannot ship without its pose set."

- **Performance budgets:** §10.2 explains the budgets through no framework on the critical path, inline snapshot, quiet field, meshes before textures, fixed audio graph, pooling, and snapshot-size tests.

- **Last two majors of Chrome, Safari, Firefox, Edge:** NOT RECOVERABLE FROM PLAN

### Architecture, data model, and API surface

- **Snapshot as render pipeline boundary:** §2.4 makes snapshots KB-scale, canonical, and sufficient to start mid-action; that boundary makes first-frame life and cross-device consistency possible.

- **TypeScript monorepo with shared engine/prose/protocol:** §2.5 says the engine, prose kit, and protocol run identically in Node and browser, so server and client greeting fallback, call extension, and prose composition use the same code.

- **Deterministic seeds and pure `tick()`:** §2.6 says replaying a tick reproduces it exactly, which makes dormant coalescing, catch-up on open, calibration simulation, and golden tests "all the same code path."

- **Edge worker, edge hint cookie, and inline snapshot:** §4.1 lets the edge inline a snapshot "without touching the database"; §6.6 says even a stale inline snapshot places birds mid-action on the first frame.

- **Server-authored API error copy:** §4.1 says the error message is already user copy, so "the client does not invent error prose."

- **Rate limits:** §4.1 and §11 use rate limits for magic-link and invite abuse while keeping request/consume telemetry aggregate.

- **Event batching, ULID idempotency, and fast-path effects:** §4.4 retries return original effects; offers and settle apply immediately under the lock so the acting device animates without waiting and other devices converge on pull.

- **Adoption flow with two names and suggestions:** NOT RECOVERABLE FROM PLAN

- **Visitor snapshot restrictions and byte-identical rendering:** §4.7 says visitors see the same snapshot with no greeting or offer state, so there is no "`show-off` rendering path to drift."

- **Twelve expression bands:** §4.8 says one band is approximately the size of "visible drift"; the client cross-fades band changes over 60 seconds so they are never a pop.

### Simulation engine

- **Presence clipping, clamping, and union:** §5.3 clips long or future intervals and counts a fleet metric so client bugs surface without account dimensions; unioning ensures two devices count as one presence.

- **Drift caps, asymptotic drift, and monotonic updates:** §5.4 says no single session crosses a visible band, late drift slows by `(1 - v)`, and absence contributes zero; caps make "too fast" impossible past 0.05/week.

- **Attunement:** §5.5 exists to make an ignored bird "ambient: still alive, still calling, greeting less often" while personality never moves down and recovery happens within roughly an hour of presence.

- **Mood hysteresis, dwell, and probabilistic switch:** §5.6 avoids hard-edge mood flips; persistent pressure flips "within ≈3 min, not on a hard edge."

- **Perch selection and no perch API:** §5.7 makes flights read as "individual decisions" by moving at most one bird per step; "The user never sets perches."

- **Weather scheduler:** §5.8 ships weather in the snapshot so all devices and visitors render the same rain or wind at the same time.

- **Call-plan horizon, preserved head, and local extension:** §5.9 preserves already-scheduled calls in the first 30 seconds so the client never hears a plan swap; local extension covers late ticks and is discarded when a server plan arrives.

- **Newcomer on the back perch without prompt or badge:** §5.14 says "the bird being there is the offer"; decline or silence has no consequence beyond a 30-day pause.

- **Notebook sparsity token bucket and scorer:** §5.15 keeps entries noteworthy, "about one entry per three days" for regular use, and "never one per session."

- **Golden ticks, property tests, calibration harness, and recognizability study:** §5.16 says calibration tuning is "a number, not a feeling"; §13 says named risks get CI gates, fleet metrics, and dogfood review questions so they cannot fail silently.

### Sync model and frontend rendering

- **Polling pull model and no push channel in v1:** §15.8 says polling at 60 seconds is the PRD's model, push is not needed for v1, and devices converge on canonical state.

- **No sync indicator or conflict UI:** §6.5 says there is no "syncing..." indicator and no conflict-resolution UI because there is nothing for the user to resolve.

- **Snapshot edge replication through KV:** §6.6 accepts a few minutes of KV staleness because the point is to place birds mid-action on first frame, then reconcile when `/open` returns.

- **Quiet field boot sequence:** §7.9 makes the quiet field both pre-bundle loading surface and empty-aviary state; there is no spinner or fade-from-static.

- **Critical chunk draws meshes before textures:** §7.1 says birds must render before textures to keep 500 ms achievable; feather detail arrives by slow cross-fade so it is invisible rather than a pop.

- **Parametric bird rigs and runtime atlas:** §7.3 uses species rigs as code and generated textures to meet bundle budgets while allowing expression-band plumage changes.

- **Responsive scene layout and no clipping:** §7.2 tests viewports from 320x568 to 3440x1440 because no bird should be clipped at seven birds, and no panning, scrolling, or zooming is available.

- **Performers, interpolation, and no teleports:** §7.5 blends mood, weather, settle, and expression changes and asserts position-delta thresholds so nothing "teleports, snaps, or resets."

- **Offer sheet rows quiet while an offer is active:** §7.10 keeps rows present but inert with "the aviary is still with the last offer" and no timer, maintaining quiet unavailability.

### Audio pipeline

- **AudioWorklet synthesis with no samples:** §8.2 generates calls from motif, signature, mood, and jitter; §8.7 says a CI check fails on audio files or `<audio>`, preserving the no recorded-audio path.

- **Chorus mixing, compressor, and distance filtering:** §8.4 lets the chorus read as an event "without a volume jump" and lets the ear separate signatures spatially and timbrally.

- **Listen-in mix:** §8.5 opens and raises the focused bird while lowering others but never to zero, so listen-in is focus, not isolation.

- **Listen-in mix decay:** §15.14 says decay is a "mix courtesy, not a state change"; focus stays engaged and any activity restores the mix.

- **Autoplay silence with captions, no enable-sound banner:** §8.7 says the first gesture resumes audio and is also the first presence signal, so "the two align naturally."

- **Caption descriptors:** §8.8 derives prose from actual synthesized parameters, never stored per motif, so captions describe what happened rather than canned labels.

### Accessibility, performance, privacy, rollout, and operations

- **Screen-reader narration that describes behavior rather than mood labels:** §9.1 says the composer never emits "wren is wary"; mood is inferred by the listener exactly as by the viewer.

- **Captions hidden from assistive tech when narration is active:** §9.2 prevents the same call from being described twice.

- **Adaptive contrast scrim and caption backplates:** §9.4 samples scene luminance to maintain contrast at noon and midnight, and CI tests ratios at seven lighting keyframes.

- **Accessibility test suite:** §9.7 uses axe, VoiceOver, NVDA, TalkBack, narration transcript tests, keyboard walkthroughs, and reduced-motion snapshots before milestone exits.

- **Initial JS and total JS budgets:** §10.1 and §10.2 enforce the PRD cap and an internal target through size-limit, code splitting, no webfont, no critical-path framework, and lazy sheets.

- **Memory and 60 fps budgets:** §10.2 uses pooling, a fixed audio graph, one atlas, batched DOM overlays, and `requestAnimationFrame` only while visible; §10.3 blocks merge on memory tests.

- **Aggregate-only telemetry:** §10.4 measures operational health; §10.5 deliberately avoids per-account or per-bird product metrics so behavior does not become an analytics funnel.

- **Allowlist logging and telemetry package boundary:** §10.7 drops fields at the logger and blocks imports from telemetry to database layers so PII, moods, and traits cannot leak by accident.

- **Magic/invite/session tokens:** §11 uses 256-bit random hashed tokens, single-use links, expiry, revocable sessions, and a short-lived edge hint with no PII.

- **Unwanted invite report link:** §11 lets a visitor block a host from inviting that address again, addressing unwanted invite abuse.

- **Export contents with opaque `engine_state`:** §11 includes birds, notebook, settings, visits, invites, and signed engine state so "nothing about the bird is lost" without trait numbers.

- **Milestone sequencing:** §12.1 sequences foundations, engine, first bird, audio, interactions, accounts/sync/visits, accessibility/performance, dogfood, beta, and GA so CI gates, calibration, accessibility, and privacy drills are exit criteria.

- **Feature flags and launch sequence:** §12.3 ships dark to dogfood, then beta, then GA; there is "no marketing surface inside the product."

- **Bird ramp and aged-aviary flag:** §12.4 validates newcomers, naming, chorus, seven-bird layout, and listening studies before day 90, because real users cannot reach bird 3 before then.

- **Operational runbooks:** §12.6 frames tick backlog, KV lag, mailer outage, failover, engine migration, and hard-delete failures around graceful degradation, idempotent ticks, and verified privacy boundaries.

- **Risk mitigations for named PRD risks:** §13 says drift calibration, sync correctness, audio uncanniness, and accessibility regression each have a CI gate, fleet metric, and dogfood review question so they cannot fail silently.
