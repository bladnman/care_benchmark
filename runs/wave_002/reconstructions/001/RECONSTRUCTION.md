## System-level intent

- **"Aliveness is the product."** The plan names this as its organizing belief in the introduction: every architectural choice should make the aviary feel "like a place that has been continuing without the viewer" and should make "a relationship that deepens over weeks." This shows up in first-frame mid-action, continuous/deterministic catch-up ticks, mood persistence, day/night and weather, idle micro-motion, procedural calls, and sparse notebook entries.

- **Load-bearing rules should be enforced mechanically, not by vigilance.** Section 0 says "Vigilance does not survive a team; tests and types do." The plan repeatedly turns product intent into CI, lint, type, schema, DB-permission, property-test, and budget gates, then consolidates them again in "Guardrails."

- **"Notice, never announce."** The plan treats the bird greeting as "the entire welcome" and forbids toast, banner, modal, confetti, badge, textual welcome, and unison cues. The same intent appears in the thin fading top bar, quiet field load state, no chrome inside the aviary, no status labels, and greeting forms that are procedurally staggered.

- **No gamification and no punishment.** The plan rejects "score, streak, level, badge, XP, visit count, day-dot calendar, milestone celebration" and rejects Tamagotchi mechanics such as death, hunger, distress, or decaying happiness. Drift is monotonic, neglect is "quieter, not mistrustful," and additional birds arrive by "aviary age, not visit count/score/tier."

- **The bird has hidden inner life, not exposed numbers.** The "personality vector is never exposed and never client-owned"; raw trait floats never leave the server, exports carry descriptions rather than numbers, and the snapshot carries only quantized "expressed behavior directives." This protects both product magic and data boundaries.

- **Server-authored canonical state prevents silent drift loss.** The server is "the only writer of personality/canonical state," clients only append events, and the tick applies additive deltas in sequence order. The plan frames sync as something made structurally unreachable rather than patched after conflicts appear.

- **One explicit product voice, split by surface.** The plan requires "naturalist" copy for aviary/notebook/narration/captions/offer prompts and "matter-of-fact" copy for sign-in/account/sync-error/accessibility-settings. The naturalist prose engine is shared so notebook readers, screen-reader users, and caption readers hear "the same product voice."

- **Procedural variation is part of the definition of life.** Calls are procedural, no recorded audio exists, idle motion is parametric, and chorus emerges from overlapping independent schedules. The plan says the "audible signature of dead software is forbidden" and "the user never hears the exact same call twice."

- **Accessibility users get "the actual product."** Section 9 rejects a "stripped, state-announcing fallback." Reduced motion is a designed surface, narration is naturalist running prose, captions match the call params, keyboard navigation is complete, and a11y is launch-blocking because shipping it later says "the product wasn't for them."

- **Privacy is a product boundary, not just compliance.** Email is PII and never an identifier; per-bird interaction data "drives only that user's own simulation"; telemetry is aggregate-only; there is no analytics read path to the canonical DB; and grammar-based prose avoids sending per-bird facts to an external model.

## Per-feature whys

### Scope

- **Single-user accounts:** NOT RECOVERABLE FROM PLAN

- **Email magic-link sign-in:** NOT RECOVERABLE FROM PLAN

- **Per-device revocable sessions:** The plan ties this to account/security control: sessions are user-facing by device label, revocable through account routes, and included in the "magic-link security" mitigations.

- **Email change with verification:** The plan's account routes specify "verify-before-switch," so the rationale is to prevent an unverified email from becoming the account email.

- **Account export:** The export must not expose raw trait numbers; it carries derived "descriptions," preserving the hidden personality boundary while still giving an account export.

- **Soft-then-hard deletion:** The deletion lifecycle includes a "30-day soft window" and account recovery route, so the rationale is recoverability before hard deletion.

- **One canonical aviary per account:** This makes multi-device sync an architectural property: every device reads the same canonical record instead of syncing clients to each other.

- **Two starter birds:** NOT RECOVERABLE FROM PLAN

- **Additional birds by aviary age:** The plan explicitly avoids visit count, score, or tier, keeping bird growth outside gamification.

- **Cap of 7 birds:** The plan calls 7 "the recognizability ceiling of the call mix"; beyond that, the calls would stop being individually legible.

- **Stable bird identity:** A bird is "never replaced on rename/sync/migration," preserving continuity of relationship.

- **Return greeting:** The greeting is the only welcome; it supports "notice, never announce" without textual welcome or product chrome.

- **Field notebook:** The notebook is sparse, read-only, and naturalist so it records genuinely noteworthy aviary moments without becoming a feed or task surface.

- **Read-only ambient visit by emailed one-time link:** Social is intentionally only one feature: ambient, opt-in, revocable, default off, and without profiles, follows, public feed, co-presence, or discovery.

### Architecture

- **Thin client, CDN edge tier, backend services, one canonical datastore:** The client renders while the server simulates, and the CDN/inlined snapshot path supports the "first bird <500 ms" budget.

- **Auth service:** It owns magic-link/session/account flows and "the only place email is stored," concentrating PII handling.

- **Simulation service:** It is the engine and "the only writer" of personality, moods, and positions, preserving single-writer correctness.

- **BFF/API gateway:** It serves snapshots and accepts event-log appends while holding "no canonical write authority over personality."

- **Notebook + Narration generator:** It turns canonical state and moment facts into naturalist prose, shares one voice engine, and remains read-only over canonical state.

- **Visit service:** It isolates invite issuance, revocation, expiry, visit-link resolution, read-only visitor snapshots, and visit logging for the one social surface.

- **TypeScript/Node services:** The plan's call is for shared types and team velocity; the tick math is light enough for Node, and the pure engine can move later if profiling requires it.

- **Client/server split:** The server owns what is true; the client owns rendering, interpolation, synthesis, ornaments, presence detection, and mix. This makes the same aviary "identically-in-mood but never identically-in-detail" across devices.

- **Render pipeline boundary:** Simulation decides "what" and the client decides "how" frame-to-frame, which is the source of procedural variation.

### Data model

- **Synthetic UUIDv7 identifiers:** The plan gives two reasons: UUIDv7 is time-ordered with good index locality, and synthetic IDs keep email from becoming an internal identifier.

- **Encrypted email on the account record:** Email is stored once, encrypted, because "email is PII and is never an identifier."

- **Email lookup hash:** The keyed HMAC lets sign-in find an account "without storing email as a key."

- **Personality vector in a server-only table/package boundary:** The client type graph cannot reach raw trait floats, enforcing hidden personality.

- **Mood `dwell_until`:** The dwell floor exists as hysteresis to "prevent flicker."

- **Position `phase`:** Carrying animation phase into the snapshot lets the first rendered frame be mid-action instead of a reset or intro animation.

- **Append-only `event_log`:** It is the only client-written input, with server-side sequence ordering and idempotency keys so retries/replays dedupe and personality is never last-write-wins.

- **Presence windows:** They are derived and not user-visible; credited seconds are clamped to wall-clock to avoid inflated drift.

- **Notebook `fact_digest`:** It supports dedupe and sparsity so active users do not get a feed.

- **Session `device_label`:** It is coarse and user-facing for the session list.

- **Invite token hash, encrypted visitor email, 30-day expiry:** These preserve privacy and make invites revocable, expiring, and one-time.

### API surface

- **Snapshot read path:** Small snapshot pulls and edge cache support the first-bird budget while keeping the client on canonical state.

- **Quantized behavior directives in the snapshot:** They give renderer/audio enough to express birds while keeping raw personality floats off the wire.

- **Session-start `greeting` directive:** It is the allowed return affordance and avoids a textual welcome.

- **Fresh snapshot on visible/resume/keepalive:** The client re-seats birds at the current canonical phase after tab return or suspend, so there is no intro or fade-from-static.

- **Read-only notebook endpoint:** No edit/delete/annotate routes exist because notebook entries are "read-only forever."

- **Batched idempotent event writes:** `client_event_id` makes client retries and replays safe.

- **Presence server clamp and active filtering:** The server refuses laxer-than-spec presence so drift is not silently inflated by merely open or hidden tabs.

- **Offer events accepted but cooldown enforced by the engine:** A cooled-down offer is logged but produces no curiosity drift, preventing single-session saturation.

- **Invite default off:** "No invite exists until this is called," so social access is explicit opt-in.

- **Visitor snapshot uses the same host snapshot:** There is "no show-off rendering," keeping visits ambient rather than performative.

- **Visitor events never enter the host event log:** "Visitor attention must not drift the host's birds."

- **Account/system routes use matter-of-fact copy:** Sign-in, account, settings, sync-error, and system errors stay in the system voice bundle.

### Simulation engine

- **Pure `tick(state, events, now)`:** Purity makes the engine "testable, replayable, and trivially correct under catch-up."

- **Continuous tick for recently active aviaries plus deterministic catch-up for dormant ones:** This keeps "continues without the viewer" true without ticking millions of idle aviaries every minute.

- **Row-level lock, transaction, and high-water sequence:** Ordered and idempotent ticking means re-running the same sequence range is a no-op.

- **Notebook back-fill during catch-up:** Overnight entries that would have fired are preserved, supporting the feeling that the aviary was still running.

- **Low-pass personality drift:** Drift is intentionally slow; no single session should move a trait visibly.

- **Non-negative monotonic drift:** Traits never decrease, so neglect cannot become punishment.

- **`(1 - P_t)` and `tanh` drift math:** The former slows near the expressive ceiling; the latter bounds per-tick contribution so a marathon session cannot spike traits.

- **Calibration harness:** It tunes constants and protects the one-week instrument / three-week visible targets in CI instead of relying on magic numbers.

- **Quietness on neglect through recency/mood expression:** The plan delivers "quieter, not mistrustful" by lowering expressed greeting frequency while leaving personality untouched.

- **Strict three-signal presence:** Visibility, focus, and recent pointer/key activity are all required because a looser "tab open" definition would inflate drift.

- **Presence activity window leaning longer:** Watching without moving "is the product," so presence should not vanish the instant the mouse stills.

- **Settle and tab-close as terminal and identical:** Neither is penalized, and there is no "you didn't settle" surface.

- **Mood hysteresis:** Mood transitions only after a clear score win and dwell floor, preventing flicker.

- **Mood persistence across sessions:** Session-start mood is session-end mood advanced by ticks/catch-up, so opening a tab never snaps birds to neutral.

- **Bird-to-bird contagion:** NOT RECOVERABLE FROM PLAN

- **Offers reacting to mood and curiosity:** Seed, song-fragment, and still-pool reactions are functions of bird state, so the same offer can approach, wait, ignore, drink, bathe, or watch.

- **Per-bird offer cooldown:** The cooldown is "functional" and prevents "single-session curiosity saturation."

- **Offers via top bar only:** The plan ties this to the layout rule: offers are not triggered by clicking a bird and there is no inline aviary chrome.

- **Listen-in events:** Sustained focus on a bird feeds bird-specific warmth and vocal drift; the engine records attention while the mix remains a client concern.

- **Settle with 5-second undo:** Undo prevents the presence window from being prematurely closed.

- **Species motif library and timbre profile:** These create a recognizable signature for each species.

- **Call scheduling shaped by vocal frequency and mood:** Vocal drift and current mood become audible through frequency, length, pitch, and sparsity.

- **Recognizability invariant for calls:** Motif skeleton and species timbre stay constant so "a user knows Pip by ear after two weeks."

- **Emergent chorus:** Overlapping scheduled calls from high-vocal birds produce chorus without a scripted chorus event.

- **Captions generated from scheduled call params:** Captions match what actually played.

- **Generative phrase grammar instead of runtime LLM:** The plan gives determinism, performance budget, voice control, and privacy as the rationale.

- **Moment facts about the aviary, never user behavior:** This prevents the notebook/prose system from expressing gamified facts such as "you visited."

- **Notebook sparsity:** A per-account rate limiter and novelty gate keep the notebook rare so active users do not get a feed.

- **Narration using the same prose engine:** Running prose, not a state list, keeps screen-reader narration in the same product voice as the notebook.

- **Captions using the same prose engine:** Short descriptions stay naturalist and match the call params.

- **Synthetic calibration patterns and opt-in internal accounts:** Calibration never mines real-user per-bird data.

### Sync model

- **One canonical state, many readers:** There is "nothing to sync" client-to-client; both devices pull the same canonical record.

- **Conflicts prevented, not resolved:** Clients never send absolute personality values, so there is no last-write-wins path to clobber drift.

- **Per-account sequence as ordering authority:** Event order is server-defined, making morning laptop and lunch phone events fold together rather than overwrite.

- **Compile-time personality type boundary:** The client bundle's type graph cannot include `PersonalityVector`, enforcing the no-trait-floats boundary.

- **Interpolation instead of teleporting:** Perch and mood changes render smoothly between snapshots so motion remains continuous.

- **Reconnect/resume re-seating at motion phase:** The aviary resumes "as if it had been rendering all along."

### Frontend rendering pipeline

- **Single horizontal scene with no pan/scroll/zoom:** NOT RECOVERABLE FROM PLAN

- **Three depth planes and three perch zones:** NOT RECOVERABLE FROM PLAN

- **Hand-written Canvas2D/WebGL2 renderer instead of a game engine:** This protects the 2 MB bundle budget and 60 fps floor.

- **Parametric idle micro-motion:** Mood is readable from motion with "no label, tooltip, or status icon," and the scene never reads as paused while visible.

- **Procedural greeting forms with stagger:** Multiple greetings avoid unison cues because unison would "announce."

- **Quiet field slow-snapshot state:** A spinner is rejected because "a spinner says machine."

- **Empty aviary state with first-bird fly-in:** The first bird enters softly and the user never sees an empty aviary again, keeping the product from feeling vacant.

- **Day/night by local time:** The account's local-time basis makes the aviary's morning/evening line up with the user's time.

- **Rare weather and ambient ornaments:** Leaves and feathers are pure client ornaments that "keep the scene alive between bird actions."

- **Thin fading top bar:** Account, settings, accessibility, notebook, and offer affordance stay outside the aviary; the bar fades so chrome does not dominate the living scene.

- **Responsive scene that never crops birds out of frame:** All birds remain visible across narrow and desktop layouts.

- **Fixed-timestep update, interpolation, and object pooling:** These are the tactics for 60 fps and zero memory growth.

### Audio pipeline

- **AudioWorklet synthesis:** Running audio off the main thread protects 60 fps and avoids main-thread garbage collection.

- **Procedural synthesis with no recorded audio:** Procedural calls support variation, the bundle budget, and the no-recorded-audio invariant.

- **Pooled voices and reused buffers:** No per-call allocation supports the no-memory-growth budget.

- **Independent chorus mixing:** Real overlapping calls avoid "two stacked loops" that phase-cancel audibly.

- **Listen-in re-balance, never mute:** Other birds ramp down to ambient rather than silence because the interaction is "listening, not channel-switching."

- **WebAudio fallback to silence with captions:** The plan says silence+captions beats canned audio because recorded audio cannot meet variation or bundle-budget requirements.

### Accessibility surfaces

- **Screen-reader narration:** Naturalist running prose gives a screen-reader user the actual aviary, not "Pip mood: content" state automation.

- **Reduced-motion mode:** It is a designed surface; a reduced-motion user gets a "calmer Pocket Aviary," not broken-looking animations-off.

- **Call captions:** They are generated from played call params and fade near the calling bird, so audio-off users get matched naturalist descriptions.

- **Keyboard navigation:** Keyboard users can reach top-bar items, focus birds, enter/exit listen-in, use offers, and settle without leaving the product surface.

- **WCAG AA contrast on chrome/copy:** User copy remains legible against bright and dim aviary states.

- **A11y acceptance gate:** Narration, reduced motion, captions, keyboard, and contrast block launch because deferring them would make the product not for those users.

### Performance budgets and observability

- **Initial JS bundle under 2 MB gzipped:** The plan's tactics are no game engine, no audio assets, compact/procedural visuals, and code-splitting less frequent surfaces.

- **Time-to-first-bird under 500 ms:** Edge HTML with inlined first snapshot and first-bird rendering before non-critical audio/assets preserve the immediate alive first frame.

- **60 fps idle for 30 minutes:** Fixed-timestep rendering, interpolation, object pooling, worklet audio, and minimal layout thrash make aliveness sustained rather than launch-only.

- **No memory growth over 30 minutes:** Pooled buffers/voices, bounded contexts/workers, and releasing scrolled-out notebook references protect long sessions.

- **Aggregate-only observability:** The metric boundary protects privacy: RUM and synthetic checks carry no per-bird or per-account interaction dimensions.

- **Unsupported-browser matter-of-fact surface:** Older browsers get a system surface instead of compatibility shims that bloat the bundle.

### Guardrails, rollout, and risks

- **Charm/affective suite:** "Charm" is regression-tested like correctness: no spinner, no announcement primitives, and real greeting variation.

- **Engine property tests:** Monotonic drift, hysteresis, catch-up equality, and calibration targets preserve core feel and correctness.

- **No-recorded-audio bundle gate:** Any audio asset in the build graph fails CI, preserving procedural-only calls.

- **Single-writer DB grants:** DB permissions make event ingress unable to write personality, mood, or position by convention or accident.

- **PII and telemetry gates:** Greps, serializers, schemas, and grants prevent email-as-ID and per-account/per-bird analytics dimensions.

- **Voice-bundle lint:** It prevents system surfaces from importing naturalist copy and aviary surfaces from importing matter-of-fact copy.

- **Build sequence independently demoable:** Each stage can prove value and guardrails as it lands, rather than hiding risk until the end.

- **Foundation guardrails wired first:** Single-writer state and PII boundaries come first because later features depend on them.

- **Accessibility before social/perf polish:** A11y is a launch-blocking stage, not a post-v1.1 follow-up.

- **Birds-per-aviary schedule server-side and feature-flagged:** Pacing can be tuned without client releases while staying based on aviary age.

- **Instrument from day one:** Performance and tick health are watched from launch while drift calibration remains synthetic or internal-only.

- **Launch gate:** Release is blocked unless budget, affective, a11y, privacy/PII/telemetry, and engine calibration gates all pass.

- **Drift miscalibration mitigation:** Low-pass math, tanh bounding, CI calibration, and server-side constants keep the product between "Tamagotchi-by-clicking" and "screensaver."

- **Monotonic-drift misread mitigation:** Property tests and documentation keep future engineers from "fixing" neglect with negative drift.

- **Sync correctness mitigation:** Single-writer grants, additive sequence-ordered deltas, and idempotent events make silent personality loss structurally unreachable.

- **Audio uncanniness mitigation:** Procedural-only synthesis, independent voices, pooling, soak tests, and never-the-same-call-twice protect against loops and glitches.

- **Announcement creep mitigation:** AST lint and the greeting directive keep "just a small welcome toast" out.

- **Gamification creep mitigation:** No counting fields and no user-behavior facts keep streaks and visit counts from appearing.

- **Privacy boundary erosion mitigation:** Synthetic UUIDs, encrypted email, no analytics read grant, telemetry schema gates, and grammar-based prose keep per-bird facts inside the simulation boundary.

- **Tick scaling mitigation:** Active continuous ticks plus dormant catch-up avoid wasting work while preserving identical canonical state.

- **Presence inaccuracy/gaming mitigation:** Strict three-signal presence, wall-clock clamp, and discard of non-active pings avoid population-wide drift inflation.

- **Day/night correctness mitigation:** IANA local time basis is stored per account, refreshed on sign-in, and tested across DST boundaries.

- **Magic-link security mitigation:** 15-minute single-use tokens, rate limiting, idempotency, httpOnly secure cookies, and revocable sessions reduce replay/interception risk.

- **Open calibration items:** Exact cadence, activity window, offer cooldown, narration cadence, drift constants, mood weights, dwell floors, age thresholds, species roster, motif libraries, and palette ratios are resolved in build with nominal values and harness/tests.
