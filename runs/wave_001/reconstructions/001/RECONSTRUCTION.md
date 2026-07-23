## System-level intent

- **Quiet, sparse, non-dashboard product shape.** The plan repeatedly frames Pocket Aviary as a calm browser scene rather than a product cockpit: "single horizontal aviary scene," "thin top bar," "No buttons inside scene," "Sparse," "quiet field" instead of a spinner, and a final reminder that depth is "small birds under a quiet window" rather than "content volume."

- **Anti-gamification and anti-pressure by design.** The out-of-scope list bans "achievements, streaks, scores, badges, levels, XP," "days visited," "Tamagotchi mechanics," "death, hunger, distress," and "punishment for absence." The sim reinforces this with "no negative drift," "absence != negative drift," and return copy that avoids "shame" framing.

- **Server-canonical aliveness, client-only expression.** The plan assigns the server "truth for continuous time, personality, mood, weather schedule, perch intent, call grammar seeds," while the client owns "presentation interpolation" and "idle micro-motion detail." It repeats that clients "never write personality" and that only the "sim-worker" mutates authoritative mood, perches, call schedule, and traits.

- **Slow, monotonic expressivity rather than visible stats.** The engine is "monotonic drift toward expressive"; the target calibration is detectable after "7 days," human-visible after "~21 days," and single sessions are capped so nothing "clicks up a number." Raw traits are hidden, and the plan bans "Numerical personality UI."

- **Bird identity should feel stable and recognizable.** The plan protects "stable identity forever," says species hotfixes never recycle `bird.id`, keeps a "carrier motif identity" stable so "Pip stays Pip," and makes success depend on dogfooders recognizing birds "by call alone."

- **Naturalist voice for the aviary, matter-of-fact voice for system surfaces.** The plan separates "naturalist prose" for narration, notebook, captions, and offer prompts from "matter-of-fact" auth, errors, settings, and unavailable visits. It explicitly rejects "naturalist evasion" for failures.

- **Privacy-by-default and relationship boundaries.** Account design uses a synthetic UUID as the "ONLY external key," encrypted email, and a ban on analytics warehouse storage for per-bird traits, behavioral event payloads, notebook bodies, and listen-in targets. Social visits are "OFF by default," read-only, scoped by email, and notebook is hidden from visitors "to keep journal private."

- **Social is ambient, not a network.** The plan excludes "profiles, follows, public discovery, comments, chat, avatars-in-scene, mutual co-presence, leaderboards," and constrains visits to invitations, read-only ambient viewing, revoke, expiry, a silent log, and optional visit notification off by default.

- **Accessibility is part of the product surface, not a compliance sticker.** The plan calls for "Naturalist SR narration," "designed reduced-motion," "call captions," "WCAG AA chrome," and "full keyboard path." The risk table says "A11y treated as ARIA stickers" would make it a "Second-class product," and success requires an SR user describing a "place," not a dashboard.

- **Performance is part of the lived feeling.** The plan treats first-bird time, 60fps idle, bundle size, heap stability, and "forbidden spinner in aviary critical path" as product requirements, not only engineering optimization. It names "500ms lived experience" and "Empty promises of aliveness on load" as risks.

- **Procedural audio and animation preserve the spell.** The plan rejects "recorded-audio call libraries," says "No sampled call libraries in bundle," and identifies "Canned audio" as a risk because the "Spell breaks." Procedural variation, caption pairing, and motif stability carry the intended aliveness.

- **Calibration and dogfood are expected before launch.** The plan warns not to implement product code "without engineering review of calibration constants," calls named defaults "starting points to tune," and requires "3-6 week drift calibration," week/3-week golden tests, and a calibration playbook before freezing weights.

## Per-feature whys

### Scope

- **Browser-only SPA, single horizontal aviary scene, no native apps** — The plan frames this as the v1 surface and reinforces it through non-goals: "Native iOS/Android apps" and "wide scenes" are out because adjacent product moves are "probably wrong" for Pocket Aviary's quiet-window depth.

- **Thin top bar** — The rationale is sparseness and scene primacy: the top bar is "Sparse," fades after cursor stillness, and the scene itself has "No buttons inside scene."

- **Email magic-link auth** — The plan's rationale is proportional v1 security and simplicity: "Magic link entropy + single use," "15 min TTL," rate limits, session revoke, and "no password DB."

- **Synthetic account UUID** — The rationale is privacy and compliance: account id is "synthetic; ONLY external key," IDs in paths are "never email," and a risk calls out "PII via email IDs" as "Compliance failure."

- **Session list + revoke** — The plan grounds this in account security and lifecycle: sessions have revoke state, `/account/sessions` lists devices, DELETE revokes, and security notes include "session revoke."

- **Email change with verify** — NOT RECOVERABLE FROM PLAN.

- **Export JSON / export job** — The rationale is user data access and lifecycle control: account lifecycle "orchestrates redaction," export is async with email download link, and privacy architecture allows personality/notebook for "User export + sim."

- **Soft-delete 30d to hard-delete** — The rationale is a recovery path plus lifecycle cleanup: account fields include `soft_deleted_at` and `hard_delete_after`, APIs support delete/cancel "within 30d," and security notes name "soft-delete recovery path."

- **One canonical aviary per account** — The rationale is canonical identity and sync simplicity: the aviary has `account_id UNIQUE`, and multi-aviary accounts are explicitly out of scope.

- **Two starter birds** — The plan states that everyone starts at "2" and empty aviary only occurs during first adoption, but the specific rationale for two starters is NOT RECOVERABLE FROM PLAN.

- **Age-gated adoption up to seven birds** — The rationale is avoiding engagement scores and preserving listenability: "age, not engagement score," "without ads or engagement scores," and "Do not raise cap above 7 without new +listening tests for call recognizability."

- **User-assigned bird names** — The plan limits user naming while preserving identity: "user names only," rename changes `display_name` only, and personality remains unchanged on name PATCH.

- **About six-species pool** — The plan gives diversity and identity mechanics: server assigns species to "maximize pool diversity," each species defines silhouette, motif library, trait biases, scale, and one nightjar-like flag; the rationale for exactly six is NOT RECOVERABLE FROM PLAN.

- **Hidden personality vectors** — The rationale is to avoid trait-chasing and dashboard thinking: no "Numerical personality UI," "never raw traits in any user UI," and no ARIA binding to raw boldness numbers.

- **Mood** — The rationale is persistent bird state that drives expression without client ownership: mood has a server-side state machine, "no reset on tab open," and the client first frame uses server mood.

- **Procedural call grammars** — The rationale is compact, alive, non-canned identity: motif libraries are procedural, "No sampled call libraries," "Canned audio" would break the spell, and captions come from the same expansion.

- **Monotonic drift toward expressive** — The rationale is long-term change without punishment: traits only increase, neglect produces "zero negative delta," and quieter behavior comes from birds staying near seed means.

- **Bird-to-bird influence** — The rationale is social aliveness within the aviary: shared call windows raise reply probability, wary contagion has a short half-life, and chorus flags densify audio "without stacking identical loops."

- **Return-greeting** — The rationale is recognition of return without product-y toasts or shame: greeting is a snapshot field plus animation, "not a toast," absence bands avoid "shame" framing, and birds stagger rather than chorus in unison.

- **Presence accounting** — The rationale is calibration honesty for drift: the server aggregates duration, not ping counts, requires visible/focused/recent activity, and merges intervals so overlapping devices do not double-count presence.

- **Listen-in** — The rationale is focused attention that still feels like listening to a place: focused bird gain ramps up while others drop to a floor not zero, and the plan says it should feel "listening," not "solo mute buttons."

- **Offer seed / song fragment / still pool** — The rationale is gentle interaction that feeds curiosity, boldness, and offer reaction FX while rate limits prevent spam; exact why for those three offer kinds is NOT RECOVERABLE FROM PLAN.

- **Offer cooldowns** — The rationale is to prevent spam without punitive UX and protect the engine: "max offers per aviary per hour to prevent spam without 'punishment' UX" and risk "Offer spam saturates curiosity."

- **Settle plus five-second undo** — The rationale is a quiet host gesture with recoverability: settle makes lighting converge and calls quiet, drift has "no drift direction," and undo is available only inside the server-validated five-second window.

- **Field notebook, read-only and sparse** — The rationale is naturalist observation rather than an event log or task surface: entries are "naturalist prose," no user edit/delete, max sparse cadence, and risk "Notebook as event log" causes "Voice collapse."

- **Multi-device sync through server-canonical snapshots and append-only client events** — The rationale is to avoid lost state and personality conflicts: "Why not LWW," "latest snapshot blob wins" must never apply, and two-device presence is folded as intervals.

- **No client personality writes** — The rationale is identity protection: absolute client personality submits are rejected at schema level, "client personality write bug" causes "Identity death invisible," and only sim-worker role credentials can update traits.

- **Visit invitations off by default** — The rationale is privacy and anti-network restraint: social network mechanics are out of scope, visits can be killed with a flag, and host never gets push by default.

- **Per-email invite** — The rationale is scoped access rather than discovery: invite rows bind visitor email ciphertext/hash, and the plan refuses discover endpoints in RFC.

- **Read-only ambient visitor** — The rationale is relationship isolation: visitor browsing uses a `visit:read` capability token, write APIs are denied, visitor tokens cannot post events, and no drift is created.

- **Visit revoke** — The rationale is host control and proportional security: hosts can delete invites, revoked visits return "This visit is no longer available," and security notes require unguessable visit tokens.

- **Thirty-day unused invite expiry** — The plan states unused invites expire after 30 days, but the specific rationale is NOT RECOVERABLE FROM PLAN.

- **Silent visit log** — The rationale is transparency without notification pressure: visit log is host settings UI only, allowed use is "Host transparency," and optional visit mail is off by default.

- **Optional visit-notification toggle off by default** — The rationale is to avoid social pressure and push life: push/email notifications about aviary life are out of scope except optional visit notify and transactional mail, and host never gets push by default.

- **Naturalist screen-reader narration** — The rationale is to make the experience a place, not data: live region uses "naturalist prose paragraphs, not trait dumps," and success is an SR user describing a "place," not a dashboard.

- **Designed reduced-motion** — The rationale is accessibility without breaking the scene: reduced motion ships day one, replaces continuous animation with "slow cross-fades," and the risk calls "animations:none" a "Broken scene."

- **Call captions** — The rationale is accessibility tied to the same audio identity: captions are generated from the same motif expansion, auto-on when audio fails, and keep audio/caption semantics paired.

- **WCAG AA chrome** — The rationale is readable account/settings controls around a visual scene: plan specifies WCAG AA chrome, AA caption contrast, and focus rings valid on day and night palettes.

- **Full keyboard path** — The rationale is complete non-pointer access: Tab moves between top bar and birds, arrows move bird focus, Enter/Space toggles listen-in, Escape exits, and focus rings are always visible.

### Resolved ambiguities

- **Sixty-second tick default, configurable 45-90s** — The plan gives an operational rationale: ticks are instrumented with p99 wall time, catch-up substeps avoid mood leaps, and drift remains tied to integrated presence.

- **Presence activity window of 180 seconds** — The rationale is to distinguish actual watching from stale tab presence while tuning if "watching-without-moving feels broken."

- **Fixed mood enum** — The plan gives implementation discipline by fixing `wary`, `content`, `curious`, `drowsy`, `alert` and mapping night sleep as a pose tag; deeper rationale is NOT RECOVERABLE FROM PLAN.

- **Continuous personality scalar range [0.0, 1.0] and seeded new birds** — The rationale is testable calibration, clamping, and species-biased variation; exact seed range rationale is NOT RECOVERABLE FROM PLAN.

- **Presence heartbeat every 30 seconds while conditions hold** — The rationale is duration accounting without ping-count gaming: server aggregates duration, and clients drop pings when idle or hidden.

- **Notebook worker every fifteen minutes** — The rationale is sparse, tick-adjacent naturalist output; exact fifteen-minute cadence rationale is NOT RECOVERABLE FROM PLAN.

- **Visit duration as snapshot-session open time with two-minute idle timeout** — The rationale is to keep visitor duration separate from host drift: "no presence semantics wired to host drift."

- **TypeScript monorepo, React client, Canvas 2D, WebAudio, HTTPS JSON with SSE/WebSocket optional** — Canvas/WebAudio choices are justified by performance and procedural audio; the rationale for TypeScript/React specifically is NOT RECOVERABLE FROM PLAN.

- **Transactional email provider for auth delivery** — The rationale is magic-link/export/optional visit/transactional account mail; specific provider rationale is NOT RECOVERABLE FROM PLAN.

- **English v1 and browser/account timezone** — The plan uses timezone to drive local day phase, weather/day-night, and day keys; rationale for English-only v1 is NOT RECOVERABLE FROM PLAN.

### Architecture

- **Auth service boundary** — The rationale is ownership of sensitive identity writes: magic links, sessions, revoke, email change, encrypted email, and account/session records.

- **Aviary API boundary** — The rationale is to allow event append, names, settle, cooldown checks, and snapshot reads while keeping personality mutation out of the client path.

- **Sim-worker boundary** — The rationale is a single canonical writer for drift, mood, weather, bird-to-bird, perch intent, call schedules, and notebook candidates.

- **Visit API boundary** — The rationale is visitor isolation and scoped read access: invites, token validate, stripped-write snapshots, revoke, and visit logs live away from host event ingest.

- **Account lifecycle service** — The rationale is redaction and delete/export orchestration.

- **Notify mailer / outbox** — The rationale is limiting mail side effects to magic links, export links, and optional visit email.

- **Pure `sim-core` package with golden vectors** — The rationale is shared, testable rules: drift, mood FSM, call grammar AST, and notebook templates are pure so server tick and client narration helpers share semantics.

- **Shared `audio-grammar` package** — The rationale is pairing motif definitions and caption strings across client and server, so captions describe the same procedural call.

### Data model and storage

- **Encrypted email plus email hash** — The rationale is lookup without using email as an external or partition key.

- **Account settings for captions, reduced motion, audio, visit notifications** — The rationale is user control of accessibility/audio/social defaults; specific setting bundle shape is NOT RECOVERABLE FROM PLAN.

- **Aviary version** — The rationale is monotonic snapshot versioning for deltas, full snapshots after gaps, and live tick push.

- **Stable bird id forever** — The rationale is identity continuity: rename changes only display name, and species pool hotfixes never recycle ids.

- **Sort seed** — The plan says it is a stable visual tie-break, but deeper rationale is NOT RECOVERABLE FROM PLAN.

- **Server-only bird personality fields** — The rationale is canonical hidden drift and protection from client writes.

- **Perch zone and slot** — The rationale is readable layout and no stacking: slots are discrete "to avoid stacking," and client maps zones safely.

- **Motion phase** — The rationale is first-frame aliveness: the first client frame is "mid-action" and birds begin mid-cycle rather than fading from static.

- **Call state with grammar seed, next call, last motif** — The rationale is scheduled, stable, recognizable procedural calls across snapshots.

- **Species pool fields** — The rationale is compact definition of silhouette, plumage, motif identity, night activity, trait biases, and scale; exact contents beyond these implementation needs are NOT RECOVERABLE FROM PLAN.

- **Append-only interaction event log** — The rationale is idempotent, ordered folding into canonical sim state; sim consumes by `server_received_at` order and dedupe keys protect retries.

- **Presence event type** — The rationale is drift input based on duration segments.

- **Listen-in event types** — The rationale is per-bird listen seconds feeding social_warmth and vocal_frequency drift.

- **Offer event type** — The rationale is offer reaction and drift signals for boldness/curiosity.

- **Settle / settle undo event types** — The rationale is cross-device quieting and bounded undo.

- **Return-visible event type** — The rationale is greeting planning after absence.

- **Name-change event type** — The rationale is name updates without personality writes.

- **Session-hello event type** — The plan lists timezone, viewport, and a11y flags, but the specific rationale is NOT RECOVERABLE FROM PLAN.

- **Offer cooldown table** — The rationale is rate state that prevents spam without punishment.

- **Notebook entry salience and source ids internal-only** — The rationale is debug and sparse generation while keeping debug linkage invisible to users.

- **Ambient weather state** — The rationale is multi-device identical weather from an aviary_id-hash poisson-like schedule with only a few rains per week.

- **Visit invite and visit session tables** — The rationale is scoped access, expiry/revoke/accept lifecycle, duration accounting, and host transparency.

- **Analytics warehouse exclusions** — The rationale is privacy: the plan never stores per-bird traits, behavioral event payloads, notebook bodies, or listen-in targets in analytics, only counts/latencies/histograms.

### API surface

- **Generic 200 for magic-link request** — The rationale is auth privacy/security: rate limiting by email hash and IP without revealing account state.

- **Single-use magic-link consume** — The rationale is security: fifteen-minute TTL, one consume, and session setting.

- **Matter-of-fact validation/errors** — The rationale is voice consistency on system surfaces: failures should be direct, with "No naturalist evasion."

- **Aviary snapshot and delta endpoints** — The rationale is server-canonical rendering and efficient multi-device sync through versions.

- **Batched `/aviary/events` with dedupe keys** — The rationale is idempotent client event append and safe offline/retry flushing.

- **Bird PATCH restricted to name only** — The rationale is preventing personality mutation through user-edit APIs.

- **Adoption eligibility and accept endpoints** — The rationale is server-owned age-gated adoption and species assignment.

- **Notebook read endpoint** — The rationale is read-only access to sparse notebook entries.

- **Settle and undo endpoints** — The rationale is explicit quieting and server-validated undo window.

- **Coarse `personality_public` bands in snapshot only if needed** — The rationale is rendering support without exposing raw traits to user UI, DOM, or a11y tree.

- **Visitor snapshot hides notebook** — The rationale is privacy: "hide notebook from visitors" to keep the journal private to host relationship while showing birds/audio/day/weather.

- **Visitor beacon that does not create host presence events** — The rationale is visit isolation: visitor duration is not host drift.

- **Snapshot keepalive or SSE nudge** — The rationale is live tick updates while visible/focused; transport choice depends on ops readiness.

- **Event flushing cadence** — The rationale is coalescing presence to avoid excessive pings while sending offers/listen-in immediately for responsiveness.

### Simulation engine

- **Shard tick loop** — The rationale is scalable server-side canonical updates per aviary due; deeper shard design rationale is NOT RECOVERABLE FROM PLAN.

- **Event fold into intermediate signals** — The rationale is converting append-only user actions into drift/mood inputs without trusting client state.

- **Ambient wall-clock advancement** — The rationale is local day phase and weather continuity even when workers catch up after lag.

- **Mood step** — The rationale is fast-timescale bird expression driven by local time, settle, offers, rain, social contagion, and boldness.

- **Perch intent from mood times boldness** — The rationale is server-owned placement intent while the client handles presentation.

- **Call schedule from vocal frequency, mood, weather, night rules** — The rationale is coherent audio behavior from server state and species rules.

- **Drift step from signals** — The rationale is slow behavior change from presence/listen-in/offers with hard caps and no negative neglect path.

- **Adoption eligibility age gates** — The rationale is growth by aviary age, not engagement score.

- **Multi-step catch-up** — The rationale is avoiding absurd mood leaps while preserving integrated presence-only drift.

- **Weekly and three-week drift calibration** — The rationale is ensuring instruments show change and humans can notice without trait UI.

- **Presence seconds lifting expressive traits** — The rationale is regular presence making birds more expressive, especially plumage_saturation and social_warmth.

- **Listen-in lifting targeted social warmth and vocal frequency** — The rationale is focused attention changing that bird's relationship to sound and warmth.

- **Offer-near-bird lifting boldness and accepted offer lifting curiosity** — The rationale is offer proximity and acceptance becoming small expressive signals.

- **No negative neglect drift** — The rationale is anti-punishment: absence must not lower traits or create distress.

- **Greeting primary greeter ranking** — The rationale is return behavior based on boldness, social warmth, mood, and stable daily randomness.

- **Greeting absence bands** — The rationale is differing return intensity without shame: glance, short call, or fuller re-orientation.

- **Greeting stagger** — The rationale is avoiding "simultaneous on-cue chorus" and "announces arrival."

- **Call motif expansion with personality pitch/timing skew** — The rationale is recognizable species/bird identity with mood-personality variation.

- **Chorus humanization offsets** — The rationale is avoiding "phase-lock artifacts of loops."

- **Offer resolution receiver choice** — The rationale is spatial or curiosity/mood weighting for believable reaction; the specific TTC reaction acronym is NOT RECOVERABLE FROM PLAN.

- **Active offer FX in snapshot** — The rationale is short-lived client animation/audio after server resolution.

- **Nightjar exception** — The rationale is that "night is not a dead scene."

### Sync model

- **Single canonical writer** — The rationale is deterministic personality/mood/perch/call mutation through one sim-worker path.

- **Reject latest-write-wins for bird vectors** — The rationale is avoiding lost mornings and identity-destroying conflicts.

- **Presence union across devices** — The rationale is honest calibration when multiple devices are open at the same wall-clock time.

- **Snapshot patch or full by version gap** — The rationale is efficient sync when deltas are retained and correctness after hard gaps.

- **Offline queue in IndexedDB** — The rationale is retrying real events with dedupe keys while forbidding speculative drift or fake offline presence.

- **Settle across devices** — The rationale is one host gesture affecting all host devices through canonical snapshot state.

- **Visitor isolation from event writes** — The rationale is preventing visitor read access from changing host sim or drift.

### Frontend rendering pipeline

- **Layered scene composition** — The rationale is depth-ordered rendering of sky, foliage, zones, captions, and chrome; deeper rationale for exact order is NOT RECOVERABLE FROM PLAN.

- **Letterbox/pillarbox with horizontal compression on narrow viewports** — The rationale is avoiding cropped birds while keeping one horizontal scene.

- **Three perch zones** — The rationale is server-assigned layout zones and slots with safe padding, not user dragging.

- **No user drag for birds** — The rationale is that server owns perch intent and the client expresses, rather than commands, bird placement.

- **First frame already alive** — The rationale is avoiding "spinner culture" and "fade-from-static hero"; birds instantiate at motion_phase and begin mid-cycle.

- **Quiet field while cold snapshot loads** — The rationale is no spinner in critical path and a calm scene before server state arrives.

- **Idle micro-motion FSMs** — The rationale is local expression from mood without moving personality simulation into the client.

- **Cancel rAF and pause audio when hidden** — The rationale is performance and correct presence semantics while sim continues server-side.

- **Interpolation between snapshots** — The rationale is smooth presentation between server states while respecting reduced motion.

- **Top bar icons and fading opacity** — The rationale is sparse controls that stay out of the aviary scene.

- **Continuous day/night and gentle weather rendering** — The rationale is ambient place feel: palette shifts over clock, rain is light, wind affects leaves, and client ornaments do not alter sim.

- **Calm naturals / no loud accents** — The rationale is visual voice: the scene stays calm while chrome remains WCAG AA.

- **Canvas 2D with sprite sheets or lightweight vector paths** — The rationale is performance and bundle control; avoid WebGL unless CPU bottleneck is proven.

### Audio pipeline

- **MotifSynth graph with chorus bus and listen-in weighting** — The rationale is per-bird procedural voices, focused listening, and a master mix; deeper rationale for graph topology is NOT RECOVERABLE FROM PLAN.

- **Oscillators, filtered noise, ADSR note tokens** — The rationale is procedural bird calls without recorded libraries.

- **Seed-derived pitch, duty, and gap jitter** — The rationale is variation each call while retaining motif identity.

- **Listen-in ramp timing** — The rationale is avoiding hard cuts and "Channel-switcher feel."

- **Other birds reduced but not muted during listen-in** — The rationale is to feel like listening in a place, not operating solo mute buttons.

- **Overlapping chorus with mild ducking and voice cap** — The rationale is preventing muddy audio while prioritizing focused bird, greeter, and nightjar.

- **Settle/night global activity reduction** — The rationale is quieting the aviary when settled or at night.

- **Silence plus captions when WebAudio is unavailable** — The rationale is accessible fallback without introducing MP3 fallback path.

- **Non-naggy audio enable prompt** — The rationale is matter-of-fact, not blocking, and not repeated every visit.

- **Autoplay gentle fade-in after gesture** — The rationale is policy compliance without "a fanfare."

- **Audio buffer pooling and soak tests** — The rationale is avoiding memory leaks and tab sluggishness.

### Accessibility surfaces

- **Polite live region with queued naturalist prose** — The rationale is observational narration that does not interrupt mid-utterance carelessly.

- **Priority narration for greetings, offer reactions, settle** — The rationale is surfacing meaningful changes while keeping observational phrasing.

- **Bird accessibility tree bridge** — The rationale is making canvas birds focusable with names and slow pose phrases without per-frame churn.

- **Visitor mode a11y without interaction controls** — The rationale is read-only visitor access with narration and captions but no write controls.

### Performance and observability

- **Initial JS under two megabytes** — The rationale is meeting the first-bird path and avoiding bundle bloat.

- **Time to first bird under 500ms** — The rationale is the "lived experience" of aliveness on mid-tier mobile 4G.

- **Sixty FPS idle** — The rationale is a smooth ambient scene on a five-year laptop profile.

- **Near-zero heap growth over thirty minutes** — The rationale is preventing tab sluggishness and audio/render leaks.

- **Low single-digit KB snapshot payload** — The rationale is efficient snapshot polling/push; deeper rationale is NOT RECOVERABLE FROM PLAN.

- **Sim tick p99 alarms** — The rationale is avoiding worker lag, stale returns, and mood jumps.

- **Code-splitting settings/account/visit/notebook** — The rationale is keeping the landing path lean.

- **Procedural audio and lean art** — The rationale is bundle size and no sampled audio path.

- **Synthetic browser monitoring** — The rationale is checking load, first-bird, FPS, and audio errors across major geos.

- **RUM without per-bird/account behavioral maps** — The rationale is operational health only, not behavioral analytics.

- **Separate privacy pipeline / no sim DB credentials in warehouse ETL** — The rationale is preventing secondary analytics use of sim PII or bird relationships.

- **No product analytics for streaks, funnels, trait distributions, recommendations** — The rationale is blocking internal gamification metrics as a cultural backdoor.

### Rollout

- **M0 foundations through M9 launch milestones** — The rationale is sequencing foundations, canonical sim, scene/audio, interactions, accessibility, lifecycle, visits, performance, dogfood calibration, and gradual launch; detailed rationale for each milestone boundary is NOT RECOVERABLE FROM PLAN.

- **Birds-per-aviary ramp starts at two and unlocks three to seven by age** — The rationale is growth without ads or engagement scores and preserving call recognizability.

- **Instrumentation from day one** — The rationale is operational visibility without building "streak dashboards internally that later leak."

- **Kill switches for visits, weather, notebook writer, SSE** — The rationale is operational control over risky or optional subsystems; plan explicitly forbids an achievements flag.

- **Content freeze discipline** — The rationale is protecting voice and non-goals: copy review for naturalist vs matter-of-fact surfaces and PR checklist for no welcome-back/streak/traits UI.

### Risks, testing, privacy, success

- **Hard session drift caps** — The rationale is preventing drift from feeling like Tamagotchi trait-chasing.

- **Increasing presence weight before new shiny UX if drift too slow** — The rationale is preserving core slow expressivity rather than adding content-volume features.

- **Three-condition presence validation** — The rationale is preventing silent global calibration corruption from "tab open" heuristics.

- **Schema forbids client trait update and anomaly monitors large deltas** — The rationale is preventing invisible identity death.

- **Integration tests for two-device sync** — The rationale is preventing LWW multi-device "Lost mornings."

- **Procedural variation tests** — The rationale is keeping audio from feeling canned.

- **Minimum listen-in ramp QA** — The rationale is preventing channel-switcher feel.

- **Greeting stagger non-collision tests** — The rationale is preventing unison greetings that announce arrival.

- **Notebook template lint and human editorial fixtures** — The rationale is preventing notebook voice collapse into an event log.

- **Narration design review before public** — The rationale is ensuring accessibility is not second-class.

- **Pose cross-fade asset pack** — The rationale is making reduced motion a designed scene instead of removing all animation.

- **Bundle budget CI fail** — The rationale is protecting first-bird performance.

- **Composer/designer tuning and soft limiter** — The rationale is avoiding harsh, uncanny audio that users mute forever.

- **Code owners on visit-api and refusal of discover endpoints** — The rationale is preventing visits from growing into a social network.

- **Synthetic UUID lint in CI** — The rationale is avoiding email-as-FK compliance failure.

- **Tick lag alarms and catch-up substeps** — The rationale is avoiding mood jumps and stale returns.

- **Thirty-minute soak CI** — The rationale is catching audio/orphan memory leaks.

- **Forbidden spinner in aviary critical path** — The rationale is preventing "spinner culture" and preserving first-load aliveness.

- **Calibration fixture scripts** — The rationale is testing daily gentle presence, weekend-only, listen-in-heavy, and neglect-return trajectories before freezing weights.

- **Magic link entropy, unguessable visit tokens, rate limits, expiring export links** — The rationale is proportional v1 security without passwords.

- **Centralized naturalist and system copy strings** — The rationale is copy discipline, lintable voice, and banned terms like achievement, streak, welcome back, level up, happiness.

- **Interaction events drive only that user's sim** — The rationale is privacy architecture: no secondary aggregate ML.

- **No cross-account bird arrays** — The rationale is enforcement against relationship reconstruction and secondary use.

- **Recognize at least two birds by call after two weeks** — The rationale is validating stable bird identity through audio.

- **Humans notice drift after about three weeks without trait UI** — The rationale is validating slow expressive change without stats.

- **Multi-device same mood/perch within one tick period** — The rationale is validating server-canonical sync.

- **Zero gamification/streak strings in production** — The rationale is enforcing the core anti-gamification intent.

- **Neglect fourteen days returns quieter expressivity without distress UX** — The rationale is validating absence without punishment or distress.

### Open implementation tickets

- **Final motif DSP by sound design pass** — The rationale is audio quality tuning; exact choices are intentionally left in-build.

- **Exact species silhouettes in design system doc** — The rationale is design-system completion; exact silhouettes are NOT RECOVERABLE FROM PLAN.

- **Notebook partially visible to visitors defaults hide** — The rationale is privacy for the host relationship.

- **SSE vs poll for snapshot nudge** — The rationale is ops readiness and transport choice without reversing architecture.

- **Precise age-ladder days after dogfood aesthetic** — The rationale is calibration without reversing architecture.
