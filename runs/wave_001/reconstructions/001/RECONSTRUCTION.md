## System-level intent

- **Browser-only, single-aviary calm surface.** The plan opens with "browser-only, single-aviary-per-account" and a "calm interaction surface" made of "presence, listen-in, offer, settle, field notebook." The same principle shows up in the non-goals: "No native apps; web protocols only," no "multi-aviary accounts," and no "customizable scenes."
- **No gamification, no Tamagotchi, no guilt UX.** The plan repeatedly refuses "streaks, badges, XP, levels, counters," "death, hunger, distress, decay meters," and "missed you" style announcements. It says "neglect -> ambient quietness only," "quieter not mistrustful," and success means the user can return to "quieter, intact birds--no guilt UX."
- **Server-authoritative simulation, append-only clients.** In the hard architectural invariants, the "Server is sole writer of personality vectors and mood canonical state," "Clients write only append-only interaction events," and "Tick is the only path that applies drift deltas." The sync model reinforces "Single canonical aviary," "No peer CRDT," and no "last-write-wins on vectors."
- **Hidden personality, no trait dashboards.** The plan says "Personality vectors never shown as numbers," raw floats "never leave API to product UI," and "Do not ship a 'debug stats' page in production." The success criteria require "Zero paths writing personality from clients," and the plan "does not expose trait dashboards under any tier."
- **Privacy-minimized operations.** The architecture requires "Synthetic account UUID" except encrypted email, says the "Simulation DB is not joined into analytics warehouse," and limits telemetry to "operational metrics only." Observability forbids "per-bird traits, offers, presence minutes, notebook text in analytics."
- **Social is opt-in, read-only, revocable, and off by default.** Scope says visits are "Opt-in visit invites by email; read-only ambient; revocable; off by default." The non-goals reject "profiles, follows, discovery, comments, chat, avatars, leaderboards, co-presence," and the risk table names "Social net gravity."
- **Accessibility is part of the product, not a later patch.** The plan requires "Narration, reduced-motion as designed surface, captions, keyboard, WCAG AA chrome." Reduced motion is a "first-class art pass," "not `animation: none`," and a11y "ships day-one with visuals/audio--not v1.1."
- **Naturalist voice for the aviary, matter-of-fact voice for account and errors.** Notebook prose is "Naturalist lowercase"; narration comes from the "same observation engine"; the voice split assigns "Naturalist" to notebook, narration, captions, and offer microcopy, while "Matter-of-fact" is for auth, errors, sync, account, a11y settings, unsupported browser, and visit revoked.
- **Aliveness without announcements or loading theater.** Boot starts with "quiet field sky--not a spinner"; birds paint "mid-pose immediately"; there is no "aviary powering on." Return greeting has "No welcome toast/banner/modal/'missed you'/days-gone copy," and success says "First frame already live; bird notices without toast."
- **Procedural identity and variation instead of canned loops.** The call grammar uses "motif atoms," stable pitch centers, seeds, and "Variation every call so no identical loop." The plan forbids "recorded-audio fallback," says "no MP3 pack," and names "Canned audio temptation" as "Spell break."
- **Slow, monotonic relationship rhythm rather than engagement pressure.** Drift "never decreases on absence," personality "only rises," bird unlocks are "Age-only, not engagement," and calibration targets make week-one changes measurable but week-three changes human-visible, with "Single session" deltas bounded so clicking cannot "Tamagotchi traits."
- **Performance budgets are part of product feel.** The plan sets budgets for "< 500 ms" first bird visible, "< 2 MB" JS, "60 fps sustained," "no growth" memory, and tick p99 alarms. Risks connect TTFA miss to "Load feel" and WebAudio leaks to tab divergence.

## Per-feature whys

### Scope and defensible calls

- **Single-page web app, modern browsers only:** The plan's reason is "web protocols only" and "do not design for native-client constraints."
- **Single aviary per account:** The sync reason is "One DB row set per account" and "All devices: same `GET /snapshot`," avoiding peer sync and last-write-wins simulation behavior.
- **Hard cap of seven birds:** The plan ties the cap to "Chorus mud at 7 birds" and says to use "mix rules + recognizability playtest"; the rollout says to monitor recognizability at "5-7 before expanding pool."
- **Email magic link:** The plan gives "202 always (anti-enumeration)," "15 minutes; single-use," and rate limits.
- **Per-device sessions and revoke from settings:** The sessions table includes a "User-facing revoke list," and the security checklist includes "Session revoke list."
- **60s nominal tick, configurable 45-90s:** "Matches '~once per minute'; calibrable without schema change."
- **180s presence activity window:** It matches "Few minutes" and "lean long so still watching counts."
- **30s presence heartbeat:** It is "Enough resolution for tick without event spam."
- **4 minute per-bird per-offer-type cooldown:** It means "Few minutes" and "prevents curiosity saturation."
- **Mood set `wary`, `content`, `curious`, `drowsy`, `alert`, `settled`:** The plan says these are "PRD examples + settled for night/settle continuity."
- **Personality traits as hidden floats in `[0.0, 1.0]`:** The reason is "Hidden scalars; room to drift up only."
- **Six species repositories including one night-active:** The rationale is "Coherent local set; night not dead."
- **Third-through-seventh bird age gates:** The reason is "Age-only, not engagement; gradual relationship rhythm."
- **Snapshot pull on open, visible return, rAF gap, keepalive:** It "Covers focus return, sleep/wake, multi-device freshness."
- **Notebook generation max about two prose entries per week plus bursts:** The rationale is "Sparse observer log."
- **TypeScript monorepo with React SPA, Node API, Postgres, Redis:** The plan says "One language, clear client/server split."
- **Canvas 2D scene with DOM chrome/notebook/settings:** It is to "Fit budgets; avoid WebGL complexity in v1."
- **Edge bootstrap path through `/api/v1/aviary/bootstrap`:** The rationale is "TTFA <500ms."
- **Account IANA timezone:** The plan says it gives a "Stable multi-device day cycle."

### Architecture

- **Server as sole writer of personality vectors and mood canonical state:** The invariant says clients "never PATCH numeric traits"; the risk table frames the failure as "Silent history loss."
- **Clients write append-only interaction events:** This keeps simulation state out of direct client mutation; the tick worker "consumes events" and writes state.
- **Tick as the only drift path:** The rationale is authoritative drift: "Tick is the only path that applies drift deltas."
- **Synthetic account UUID everywhere except encrypted email:** The reason is to reduce "PII blast radius"; logs use account UUID not email.
- **Simulation DB not joined into analytics warehouse:** The plan allows "operational metrics only" and forbids a "warehouse replica of simulation DB."
- **Visitors never write host drift events:** The reason is read-only ambient visiting; visitor heartbeats go to `visit_sessions` only and give "no host presence contribution."
- **Snapshots describe targets while the client runs animation:** The plan says the server "never sends frame-by-frame animation" and the client runs the "continuous feel-alive layer."
- **Pure `sim-core` package:** The reason is test/production identity: "same drift math in tests as production worker."

### Data model and API surface

- **Encrypted email plus HMAC lookup:** The plan says "Encrypted at rest" and "Lookup without storing plain email in indexes elsewhere."
- **Stable bird identity forever:** NOT RECOVERABLE FROM PLAN
- **Append-only `interaction_events` with `client_event_id`:** The reason is idempotency; duplicate events are handled by unique `(account_id, client_event_id)`.
- **`presence_segments` materialization:** The plan gives "drift math auditability."
- **`visit_invites` 30 day unused TTL:** NOT RECOVERABLE FROM PLAN
- **Snapshot derived presentation bands instead of raw personality floats:** The reason is that raw floats "never leave API to product UI" and bands must be "non-revealing."
- **Starter adoption with two distinct deterministic species:** NOT RECOVERABLE FROM PLAN
- **Magic-link API returning 202 always:** The reason is "anti-enumeration."
- **Snapshot `If-None-Match: simulation_version` returning 304:** NOT RECOVERABLE FROM PLAN
- **Event batch response with `simulation_version_hint`:** It lets the client trigger an immediate pull or rely on "next tick <=60s + optimistic client reaction."
- **Account export:** It preserves the user's copy: numeric vectors are allowed in the "private export file--user's copy," and export goes "only to verified email."
- **Soft delete with 30 day cancel then hard delete:** The plan says "Soft delete 30d then hard shred" and cascade/shred email ciphertext after hard delete.
- **Scoped visitor token / visit JWT:** The reason is `role=visitor`, `host_aviary_id`, and "no event ingest permissions."
- **Rate limits:** Magic links are capped; event limits are "designed for presence 30s cadence"; invite caps "reduce abuse."
- **Ops-only health and tick metrics endpoints behind network ACL:** NOT RECOVERABLE FROM PLAN

### Simulation engine

- **Redis tick lock per aviary:** The rationale is safe tick ownership; conflict risks include "tick reentrant with processed watermark" and lock timeouts.
- **Presence close/merge/clip during tick:** The plan uses it to avoid background inflation and double counting: "union-merge overlapping" intervals and "clip by settle/session_end."
- **Offer reaction resolution using mood and curiosity:** It makes offers bird-like without user stats: resolve "acceptance/approach using mood x curiosity" and set cooldowns.
- **Rare weather:** NOT RECOVERABLE FROM PLAN
- **Bird-to-bird mood and chorus step:** NOT RECOVERABLE FROM PLAN
- **Mood FSM stickiness:** The plan says to use tempered softmax/stickiness "so moods don't thrash every minute."
- **Monotonic personality drift:** The reason is the non-Tamagotchi rule: "NEVER: `t_new < t_old` for neglect" and "ambient quiet is mood/greeting rate, not negative drift."
- **`attention_ema`:** It is the preferred model for "quieter after neglect" while keeping "boldness/plumage" intact and personality non-negative.
- **Drift calibration targets:** Week-one change should be "detectable above noise floor"; week-three should be human-visible; a single session is bounded so "clicking cannot Tamagotchi traits."
- **Greeting plan on return:** The plan wants the bird to notice by absence bucket while avoiding "welcome toast/banner/modal" and "unison welcome chorus on cue."
- **Sparse notebook triggers:** The rationale is to keep the notebook from becoming a "feed" and preserve "field notes not logs."
- **Adoption offer during tick:** It follows age gates and bird cap; the rationale elsewhere is "Age-only, not engagement."
- **Call-grammar runtime:** It preserves identity and variation through "fixed species motif palette," "bird-specific pitch center," and "Variation every call so no identical loop."
- **Autonomous offline continuity:** Tick continues through "day/night mood, weather, idle bird-to-bird, attention_ema decay" so return can be snapshot plus greeting rather than a stuck aviary.

### Sync model

- **Single canonical aviary across devices:** The reason is the same snapshot progression for all devices, with "No peer CRDT" and no last-write-wins vectors.
- **Dual-device presence merge:** It prevents "Accelerated drift" and "double-speed drift."
- **`simulation_version` rebase for optimistic client mood:** It corrects "Stale client optimistic mood."
- **Concurrent listen-in merge:** The plan says drift gains "once per real-time minute of attention per bird" rather than per device.
- **Visitor session logging separate from host simulation:** This keeps "no host presence contribution" while still allowing a host visit log.

### Frontend rendering pipeline

- **Quiet field shell instead of spinner:** It supports first-frame calm: "quiet field sky--not a spinner."
- **Immediate bird paint mid-pose:** The reason is "first bird paint path (critical)" and success where the "First frame already live."
- **No entry animation of aviary powering on:** The plan refuses "aviary powering on"; only "true first adoption empty->fly-in once" is allowed.
- **Single horizontal scene with depth zones:** The plan says responsive spacing must keep "always all birds on-screen; never crop."
- **Idle micro-motion:** The reason is aliveness; birds are "Never fully freezed if reduced-motion off."
- **Curved perch transitions:** The plan says "not teleport."
- **Listen-in visual focus without scene selection box:** The rationale is calm scene integrity: no "selection box chrome in scene"; keyboard focus ring is for a11y only.
- **Settle transition and 5s undo:** The plan articulates "palette warm->evening," "call mix down," and "5s undo on any pointer/key in scene."
- **Reduced-motion mode:** The reason is "a first-class art pass" so reduced-motion users do not lose "audio + captions + narration + notebook + drift."
- **Top bar fading and no badges:** The plan says no "badges" and "no visit count flashes," preserving the calm surface.
- **Offer UI as top-bar naturalist chooser:** It is "not bird-click" and "doesn't stamp permanent chrome on birds."
- **Color palette and contrast:** The reason is "Calm naturalist palette" with "WCAG AA" contrast for labels.

### Audio pipeline

- **Procedural WebAudio calls:** The reason is "No large audio asset packs," "no recorded-audio fallback path," and no call loops.
- **Buffer pooling and no unbounded allocation:** The plan names performance and leak risk: "no per-call unbounded allocation" and "Memory leaks WebAudio" mitigation.
- **Stable pitch center plus motif variation:** The rationale is bird identity and no loop: "Pip identifiable by ear" and "Variation every call."
- **Listen-in ramped mix:** The plan says "Mix rebalance not hard cut"; other birds remain at an "ambient floor (>0, never mute)."
- **Audio failure fallback to captions:** The plan says "force captions on for session" and "no MP3 pack."
- **Song-fragment offer as procedural preset:** It is "also procedural" and "not a radio hit file."

### Accessibility surfaces

- **Screen-reader narration:** The reason is aliveness without stats: prose comes from observation service and is "not trait dumps."
- **Narration priority queue:** It prevents flooding by dropping "stale ambient" if the queue backs up.
- **Captions from actual motif params:** The reason is to "Match audio."
- **Keyboard navigation:** The rationale is reachability: offer and settle are "reachable without pointer."
- **AA chrome/copy/captions:** The plan requires contrast across chrome and day/night focus rings.
- **A11y settings matter-of-fact voice:** It follows the voice split: settings use "matter-of-fact voice," while scene prose uses "naturalist voice."
- **A11y shipping day one:** The reason is explicit: "not v1.1."

### Performance and observability

- **Initial JS, TTFA, FPS, memory, snapshot, and tick budgets:** These are release gates; success says "Budgets in §10 pass on release candidate."
- **Code-splitting settings, visits admin, export UI:** The rationale is to meet the JS and first-paint budgets.
- **Procedural/SVG birds and no large audio packs:** The plan uses these to keep assets light and fit performance budgets.
- **Pause rAF when hidden:** NOT RECOVERABLE FROM PLAN
- **Synthetic geo browsers and RUM:** These are allowed for operational SLOs: TTFB, first-bird paint, long tasks, FPS, audio context errors, API latency, tick latency.
- **Analytics forbidding traits/offers/presence/notebook text:** The reason is privacy and the rule that the simulation DB is not replicated into analytics.
- **Unsupported browser matter-of-fact page:** NOT RECOVERABLE FROM PLAN

### Interaction implementation notes

- **Return-greeting absence buckets:** The rationale is bird noticing without product copy: "bird notices without toast."
- **Single primary greeter with staggered secondary:** The plan says unison greeting creates "Announcement feel"; mitigation is "Stagger + single primary."
- **No welcome toast/banner/modal:** The reason is to avoid "Toasts 'welcome' PR" and "Breaks product."
- **Presence triple gate:** It prevents "Background tab inflates population drift" and keeps presence from counting unless visible, focused, and recently active.
- **Offer cooldown server reject:** It enforces authoritative cooldowns and prevents "curiosity saturation."
- **Read-only field notebook:** It must "Fully avoid user-behavior moralizing" and feel like "field notes not logs."
- **Voice split in interaction copy:** Naturalist is for aviary surfaces; matter-of-fact is for auth, errors, sync, account, a11y, unsupported browser, and visit revoked.

### Social visits

- **Visits default off with zero invites:** The reason is quiet and anti-social-network scope.
- **Email one-time invite link:** The plan connects this to opt-in visiting and scoped visit tokens.
- **Visitor read-only SPA mode:** It hides "offer/settle/listen-in write paths" while still allowing visitors to "hear calls/render."
- **No co-presence indicators:** The rationale is the no-social-network refusal.
- **No host push unless opted in:** The plan says "email degrades quiet" and keeps notification infrastructure restrained.
- **Visit log without badge ornaments:** This preserves the no-badges, no-visit-count-flashes posture.
- **Revoke returning 410 matter-of-fact on next pull:** The rationale is revocability without social drama.
- **Age-out invites 30d:** NOT RECOVERABLE FROM PLAN

### Rollout, privacy, and calibration

- **Build phases ordering:** The plan moves from "Foundations" to birds, drift, audio, interactions, notebook, a11y, visits, then "Perf hard gates"; this stages risk around core simulation before peripheral systems.
- **Birds-per-aviary ramp:** The plan says "Do not sell bird packs" and to test call recognizability at "5-7 before expanding pool."
- **Internal-only drift calibration dashboard:** The reason is tuning on "synthetic birds / load accounts--not product users' birds in warehouse."
- **Feature flags for visits, weather intensity, notebook rate:** They allow "tuning without schema churn."
- **Privacy policy surface:** It should list aggregates and "exclude per-bird interaction use."
- **No third-party ad pixels on aviary surface:** The rationale is privacy/security posture.
- **Open calibration backlog as sim config:** Each open item becomes "notation in sim config, not a redesign."
