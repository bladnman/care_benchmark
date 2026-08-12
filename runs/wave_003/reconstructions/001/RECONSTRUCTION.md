## System-level intent

- Server-owned life, client-rendered attention. The plan states this in the product line ("Server owns life; client renders and reports attention"), repeats it in the client/server split ("Clients never send trait values, mood writes, perch commands, or notebook text"), and makes it the sync model ("Clients are projectors").

- Quiet, non-gamified, non-punitive life. This shows up in the implementation bans on "achievements, streaks, levels, scores, badges," the Tamagotchi ban on "death, hunger, distress" and "neglect-as-punishment," the drift rule that "Neglect never decrements a trait," and the success criterion that the person should "never encounter a score, a streak, a death, or a 'welcome back.'"

- Naturalist voice for the aviary, matter-of-fact system voice for account mechanics. The plan makes a "Voice split" with `copy/naturalist/*` as "lowercase, present tense, no 'you,' no exclamation, bird-named, specific," while `copy/system/*` uses "Normal English capitalization, direct, no naturalist metaphor."

- Privacy is "load-bearing." The plan names "Privacy architecture (load-bearing)," isolates `simdb` from `opsdb`, says "No ETL from `simdb` to analytics," and repeats under observability that RUM is "privacy-safe" and must never include "per-bird events, trait values, offer kind histograms by account, notebook text, emails."

- Deterministic simulation and catch-up equivalence are product correctness, not just implementation style. The tick is "deterministic," "catch-up must equal N live ticks," `packages/sim` is "side-effect free," and CI asserts "Catch-up == N live ticks."

- Attention should matter slowly and over weeks. The calibration target says honest presence over 7 and 21 days should move traits, but "single-session trait delta is always < 0.012." The drift risk says too fast becomes "Tamagotchi numbers the user can feel session-to-session," while too slow becomes "screensaver."

- "Aliveness is the first frame." The boot sequence uses a "quiet field," "No spinner," birds already "mid-cycle," and no "ready" event toasts. The first-bird budget defines success as bird pixels drawn, since "Quiet field does not count."

- Accessibility is a v1 requirement, not an add-on. The plan says "Accessibility ships on the same day as the visual aviary. Not a v1.1," and rollout says not to flag-gate "reduced-motion or narration" because "Shipping without them is not v1."

- Scope discipline is expressed as schema and protocol discipline. Out-of-scope items "must not leak into schema or protocol design," non-goals are "implementation bans, not backlog," and scope-creep mitigations say reviewers reject schemas that store visit-frequency for display.

## Per-feature whys

**Locked decisions and scope**

- Personality trait numeric bounds and starter seed ranges: NOT RECOVERABLE FROM PLAN

- Mood enum exactly `{wary, content, curious, drowsy, alert}`: NOT RECOVERABLE FROM PLAN

- Deterministic 60s simulation tick and catch-up: The rationale is equivalence and continuity. The tick is deterministic so catch-up can "equal N live ticks," tests can hash state, and returning accounts do not see frozen mood, lighting, or nightjar activity.

- Presence activity window and pings: The plan ties this to "honest presence." Presence requires visible, focused, and recent pointer/key activity, while the server records only "the intersection of client-claimed seconds and elapsed wall time" so claimed attention cannot inflate drift.

- Offer cooldown: The plan gives the why in "Offer saturation": without cooldown, "curiosity slams into 0.95 in one evening." The 180s server cooldown plus small `k_c` and the approach term keep offers from overwhelming drift.

- Age-gated new-bird offers and no catalog: The plan says adoption pacing is aviary age, "not operational gradualism," and says not to accelerate it for "retention" or sell a third bird. "No catalog" and "system picks species" keep adoption from becoming shopping, rarity, or monetization.

- Six-species pool and starter pair: The plan says the starter pair comes from a "complementary pairing table," the user "does not pick," and "Never two nightjars as starters." Later duplicate species remain recognizable because recognizability comes from "call grammar seed per bird id, not species alone."

- Notebook sparsity: The plan says "Zero entries on a quiet week is correct" and "Sparsity > filler." The notebook writes only when something specific can be said; otherwise, "write nothing."

- Six named song-fragment motifs: NOT RECOVERABLE FROM PLAN

- React chrome plus custom Canvas 2D scene renderer: The plan's rationale is the render boundary. React mounts "chrome" only, "React never mounts birds," and the scene consumes a pure render snapshot so the aviary continues while sheets are open.

- Node 22, Fastify, Postgres 16, Redis 7, and transactional email provider choices: NOT RECOVERABLE FROM PLAN

- Magic-link auth and session cookie: The plan keeps auth in system voice, avoids passwords/SSO in v1, uses single-use 15-minute links, revocable devices, and same response bodies to "avoid account enumeration."

- Visit invite TTL, revocation, and read-only visitor snapshots: The plan frames visits as off by default, revocable, and read-only. "Visitor snapshot endpoint never accepts events," visitor clients cannot write, and every snapshot re-checks revocation so visits cannot alter host drift.

- Drift weekly calibration target: The plan explicitly balances "too fast" against "too slow": too fast makes session-to-session Tamagotchi numbers, too slow makes a screensaver. Calibration gives attention visible long-term weight without visible stat-chasing.

- IANA account timezone: The plan says day/night and mood-of-day use this timezone "never UTC wall clock as 'local,'" and the risk names the failure as "UTC night for a Tokyo morning."

- Quiet-field first paint with no spinner: The plan wants a calm loaded surface. The quiet field holds until first snapshot, warm sessions draw from embedded snapshot, and the mitigation for first-frame failure is "quiet-field HTML" plus "mid-pose boot."

- Settle undo window: The plan allows a 5s client undo and says an undone settle is treated "as if it never happened for mood" while presence still "ends/restarts" cleanly. The why is reversible quieting without corrupting mood or presence accounting.

- Listen-in mix with other birds still audible: The plan says the focused bird ramps to `1.0`, others to `0.22`, "Never `0`," with equal-power crossfade. Later notes say "others stay audible" and "Do not let listen-in mute the chorus to zero."

- Hot live ticks, catch-up-only idle accounts, and the daily sweeper: The plan's why is continuity without excessive live work. Hot accounts get live ticks; 7-90 day idle accounts get daily catch-up so "mood/day-night do not freeze"; very long idle accounts catch up on next access.

- Browser support for the last two major versions of Chrome, Safari, Firefox, and Edge: NOT RECOVERABLE FROM PLAN

- One account, one aviary, two to seven birds: The plan supports this with scope control: multi-aviary accounts and shared/household aviaries are out of scope and should not be designed toward, while the seven-bird cap is gated by audio recognizability.

- Explicitly out-of-v1 bans: The plan says these are "implementation bans, not backlog" so native apps, gamification, Tamagotchi mechanics, social surfaces, payments, and notifications do not shape schema, protocol, copy, or tests.

- Visit notifications off by default: The plan's rationale is avoiding aviary notifications and social pressure. Visit emails exist only as the invite link, and visit-happened email is sent only if the host opted in.

- Account export: The plan allows numbers only here because "the PRD says the user may take a copy of their birds." Export is "matter-of-fact," a settings action, and "the only user-reachable place numbers appear."

- Soft delete then hard delete: The plan gives a restore window through matter-of-fact system copy, then deletes child rows, mail payloads, visit invites, and the account after 30 days so "No backup warehouse retains per-bird rows."

- Copy catalogs and banned-token lint: The plan's why is product voice enforcement. Naturalist copy must not say "you" or use exclamation; system copy must not use metaphor; CI greps banned tokens like `achievement`, `streak`, `welcome back`, `level`, and `xp`.

**Architecture, data, and API**

- No client-to-client channel and no required websocket: The plan says snapshots are HTTP pull and a future SSE hint "must not become a second state channel." The why is keeping one canonical state path.

- Browser/API/event-log/sim-worker/canonical-store architecture: The plan separates event ingestion from canonical state. Events are append-only, the sim worker consumes them, and Postgres is the system of record so simulation state is never Redis-only.

- Server/client ownership split: The rationale is authority. The server owns account, bird identity, personality, mood, event log, presence seconds, notebook, weather, settle flag, and visits; the client owns render clock, interpolation, ornaments, and sensors.

- Render snapshot boundary: The plan makes `packages/scene-state` produce a pure serializable view model so React chrome cannot pause or own bird life. Sheets are "chrome, not modes that pause life."

- Mail outbox worker: The plan says to "Never send mail inline in the request path except 'accepted, queued.'" This protects request latency and keeps mail side effects out of API transactions.

- `simdb` and `opsdb` isolation: The rationale is privacy. Email exists in exactly one encrypted column, telemetry has no notebook text or bird behavior, and there is "No 'average boldness' dashboard."

- `packages/sim` as isomorphic, side-effect-free code: The plan says the same functions must run in worker, catch-up, and CI calibration harnesses, and must not import Fastify, Redis, or DOM.

- UUIDv7 ids and stable bird ids: The plan states ids are "time-ordered, not email-derived," and bird ids remain stable across rename, species-pool edits, and migrations so identity is not reminted.

- Client event UUID plus server `seq` deduplication: The rationale is retry safety. Events are idempotent on `client_event_id`, POST is transactional, and clients retry with the same id after outages.

- Snapshot omits raw traits, presence totals, visit counts, and streak-like fields: The plan says `plumage.saturation` is only a render parameter and "Never in snapshot" includes stats-like surfaces. This preserves the no-score/no-streak product.

- Persistent account mute setting: The plan says mute is "user chrome, not a toast" and is persistent "so devices agree." It is not treated as neglect.

- Notebook entry storage with rendered prose and internal stimulus: The plan says body is already-rendered naturalist prose and stimulus is internal, "never exported as numbers-to-user."

- Magic-link endpoint always returns 202 and rate limits by email-hash/IP: The plan explicitly says the same body is used "to avoid account enumeration."

- Snapshot pull triggers and `since_rev`: The plan gives pull triggers for navigation, visibility return, bfcache, sleep/suspend, keepalive, and event echo so clients stay current through ordinary browser lifecycle without a second state channel.

- Interaction event validation: The plan accepts safe no-ops and rejects only invalid presence events. Unknown birds are dropped, cooldown offers are flagged ignored, and visitors get 403 so bad client events do not crash or affect host state.

- Notebook API reverse chronological, no search, no edit/delete: NOT RECOVERABLE FROM PLAN

- Adoption and naming endpoint presentation: The plan says adoption is "not a catalog modal"; naturalist copy says "another bird has found the aviary," and first-run is naming arrivals, "not shopping."

- Visit settings, log, revocation, and visitor client: The rationale is read-only ambient visiting without social-network surfaces. The visit log is settings chrome with no badge count, notebook is host-private, and account icon becomes "leave visit."

- Additive `/v1` compatibility: The plan says if a field changes meaning, "add a new field; do not silently reinterpret `mood` strings." The why is client compatibility and stable snapshot semantics.

**Simulation and sync**

- Pure tick contract and event order: The plan applies events in `seq` order, then ambient processes, with RNG seeded from aviary and tick index. This makes live and catch-up behavior reproducible.

- Fast-forward for long empty gaps: The plan says a year-idle return should not block first snapshot more than about 50ms, but fast-forward still advances lighting and nightjar activity and does not apply drift without presence.

- Monotonic personality drift: The plan states "Neglect never decrements a trait" and "Ambient quietness" is behavior over unchanged or slowly raised traits. The why is avoiding punishment while preserving long-term expressiveness.

- Mute and no-audio still count for attention: The plan says "Mute is not neglect" and listening while audio is muted still counts because "they are watching" and attention focus should not require sound.

- Settle as no trait delta: The plan says settle ends presence pings and gives only a "small mood nudge" because settling is quieting, not a drift signal.

- Mood transition with time-of-day, weather, interactions, and personality gates: The plan uses mood to translate local time, rain/wind, offers, listen-in, settle, boldness, and curiosity into state. Hysteresis exists to "Prevent flicker."

- Perch, pose, and call behavior policy: The rationale is turning traits plus mood into visible behavior. Boldness affects front/back, social warmth pulls toward or away from occupied zones, mood selects pose intent, and vocal frequency shapes call intervals.

- Ambient quietness after long absence: The plan says do not lower traits; instead greet less often and call less often through a recency window because "less often is what has been observed." It is "not negative drift."

- Return-greeting: The plan chooses one primary greeter and never unison, with style based on absence length but "No toast" and no "you've been gone N days." The why is a bird noticing without a system welcome.

- Exact offer kinds `seed`, `song`, and `pool`: NOT RECOVERABLE FROM PLAN

- Offer reactions and cooldown no-op behavior: The plan gives each offer a bird-behavior effect and says cooldown no-op is "functional, not a scolding UI." Gestures stimulate birds without user-facing punishment.

- Weather schedule as server state: The plan says weather is scheduled on the aviary, "not per client," and included in the snapshot "so all devices agree." The risk section names weather desync as a failure.

- Bird-to-bird chorus, wary spread, and social perch pull: The plan includes these to make birds influence one another, but bans "flocking paths or fight/play minigames" so interaction remains ambient.

- Adoption mint on account create: The plan mints the aviary and starter birds in the same transaction so the first real view is not empty; the only empty state is "first-run fly-in."

- Server call plan instead of synthesized audio: The plan says the server does not synthesize audio; it schedules `CallPlan` and the client expands grammar, so a fresh tab can start mid-chorus without a tick delay.

- Notebook writer conditions: The plan only writes after sparse timing and noteworthy stimuli, using present-tense naturalist prose. The why is avoiding generic "session started" filler and attendance phrases.

- One row-set per aviary, no CRDT, no personality sync endpoint: The plan says personality has a single writer and "Personality LWW is unreachable because clients cannot write it."

- Multi-device presence max-not-sum: The plan says "Honest attention is not additive across devices (user is not two people)." Multiple devices merge by max seconds, not sum.

- Conflict handling: The plan uses single-use links, bird cooldown, last-received rename on name only, reengage clearing settle, visitor no write, server received time, and transactional retries so conflicts stay narrow and non-personality.

- Online-required aviary and short offline event queue: The plan avoids local personality by queuing no local personality, optionally queues events for only 5 minutes, and drops stale events so a wake after a weekend does not dump false presence.

- Session revocation device list: The plan sets `revoked_at` and the next API call 401s with matter-of-fact copy, giving per-device control without naturalist metaphor.

**Frontend, audio, accessibility, performance, rollout, and testing**

- Boot sequence: The plan says "aliveness is the first frame." It avoids spinner, logo splash, fade-from-black, and ready toast; first draw is mid-pose except true first-run fly-in.

- Fixed 16:9 logical scene with letterboxing: The plan gives the rationale "never crop a bird." Narrow phones compress spacing but keep x coordinates in range.

- No pan, zoom, scroll, or bird dragging in the scene: NOT RECOVERABLE FROM PLAN

- Procedural bird visuals and micro-motion: The plan uses small compiled art tables for bundle size and requires continuous idle motion so birds never hold a perfectly still pose longer than 400ms except reduced-motion stills and sleep breathing.

- Snapshot interpolation and no teleporting: The plan says perch changes ease over 2200-4000ms and late snapshots retarget mid-ease, so sparse network snapshots still look continuous.

- Ambient leaves and feathers: The plan makes them client-only, subtle, and disabled in reduced motion, so they add scene life without sim state or vestibular burden.

- Day/night and settle lighting: The plan makes sky and fill lights functions of snapshot lighting and settle; settle eases toward evening and quiet call gain independent of actual hour.

- First-run fly-in only: The plan says starters already exist server-side, so fly-in is just the first arrival moment. After that, the aviary is "never empty, never fly-in on later sessions."

- Sparse top bar, fade, and no badges: The plan allows only account, accessibility, notebook, offer, and settle, and says "No badges, no unread dots on notebook, no visit-count pip" to avoid gamified notification surfaces.

- Day/night color tokens and AA chrome text: The plan's why is readability against both midday and night bar backgrounds.

- Reduced-motion path: The plan says this is a "designed second renderer path," not `if (reduce) return`, so reduced motion remains alive through still poses, slow crossfades, lighting shifts, and audio.

- Hidden-tab frame loop behavior: The plan cancels rAF and suspends canvas when hidden while "Simulation continues on server," so rendering saves work but life does not pause.

- Bundle split: The plan keeps boot, scene, and audio initial, while notebook/settings/auth split by use, so initial JS stays under the budget and boot path target.

- Procedural audio: The plan explains "Looped files break recognizability-under-variation and chorus" and "Bundle budget forbids a rich recorded library," so there is "No recorded-call fallback."

- Audio graph, voice pool, and buffer reuse: The plan's why is avoiding leaks and clipping. It requires a flat heap after 30 minutes and says no per-call oscillator leak.

- Per-bird audio signature and recognizability QA: The plan derives signature from bird id and requires listeners identify birds above 70 percent; this gates the seven-bird cap.

- Chorus mixing and listen-in ramps: The plan avoids shared loop clocks, uses light compression to prevent clipping, and ramps listen-in instead of cuts so chorus remains overlapping and audible.

- Listen-in focus UX not being a selection box with a menu: NOT RECOVERABLE FROM PLAN

- WebAudio fallback silence plus captions: The plan says if audio is unavailable, use silence and force captions on for the session; "Do not decode MP3s." The why is accessibility without recorded fallback.

- Screen-reader narration: The plan uses a polite live region in naturalist prose, mentions at most two birds, and avoids trait numbers, perch indexes, and "mood: content" so the accessible surface is observational instead of a stats table.

- Client-side narration: The plan explicitly decides this so narration can include listen-in focus, such as "pip's call is closer now," without a server round trip.

- Keyboard path: The plan provides top-bar order, bird cycling by spatial order, listen-in toggles, Escape behavior, and focus rings so there is a "full keyboard path."

- Captions: The plan generates captions from the "same atom sequence" as the synth so captions match the call, and avoids using `aria-live` for every caption because that would be too fast.

- Contrast and copy accessibility: The plan requires WCAG AA minimum for chrome, sheets, errors, and captions, with AAA preferred for system errors.

- Refusals in accessibility: The plan will not expose personality vectors to ARIA, ship a static screenshot as the accessible version, announce "welcome back," or use assertive live regions except visit/session loss. The why is avoiding stats, dead fallback, and live-region spam.

- Performance budgets: The plan makes budgets CI gates or alarms because failures include bundle growth, memory leaks, seven-bird frame drops, and slow TTFBird.

- TTFBird strategy: The plan uses edge-served HTML, inline sky CSS, injected boot snapshot, scene-first import, no scene webfonts, and no critical-path images to get one RTT and first bird visible.

- Runtime discipline: The plan uses a single rAF, audio voice pool, virtualized notebook, OffscreenCanvas when available, and an ornament governor so 30-minute sessions stay flat and smooth.

- Aggregate-only observability: The plan measures operational health such as TTFBird, fps, audio init failure, API latency, and session duration histogram without account dimension. It explicitly excludes bird ids, traits, emails, notebook text, and engagement dashboards.

- Deliberately not measuring DAU/WAU, engagement clicks, settle funnels, population drift averages, or visit leaderboards: The plan says "Operational health only" and product quality is calibration tests plus "qualitative watch-throughs."

- Rollout phases: Dogfood checks canned-call detection and greeting sameness; closed preview enables visits and paid accessibility audit; public removes allowlist. This stages quality and accessibility while still shipping "one v1 product."

- Feature flags as off-ramps: The plan allows `visits`, `weather`, and `audio` flags only for outages, "not product experiments," and warns flags must not create a second personality writer.

- Day-one calibration harness and tests: The plan uses CI simulated traces for drift, double-device presence, absence, catch-up equality, and notebook banned phrases so product invariants are verified outside prod telemetry.

- Content and ops launch checks: The plan requires species art and grammar signoff, copy review, privacy policy, unsupported browser page, and export/delete end-to-end so launch includes voice, privacy, and account lifecycle.

- Adoption pacing after launch: The plan says not to accelerate the age gates for "retention," not to sell a third bird, and treats any higher cap as "a new PRD."

- Testing strategy excluding wrong-product assertions: The plan says no test should assert "a streak, a welcome toast, or a hunger state" because those tests would encode the wrong product.

- Implementation notes that prevent usual mistakes: The plan gives negative guardrails such as no `welcome` in auth success, no mood hover tooltips, no last-visited display, no email-as-account-id, no client LWW bird row, no settle-as-logout, no notebook archive, and no native offline sync tables because each would violate the product shape above.

- Success criteria: The final why is the complete v1 experience: sign in, meet two birds already in motion, feel over weeks that attention mattered, listen in, offer, settle or close, read sparse field notes, invite and revoke a friend, export/delete, use keyboard/SR/captions/reduced motion, and never encounter a score, streak, death, or "welcome back."
