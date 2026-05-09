## System-level intent

- **No client-owned simulation state.** The plan makes this load-bearing in `Scope`, `Architecture`, `Sync model`, and `Summary`: "server-side simulation tick" is "the only writer," "clients never write personality state," "snapshots are always authoritative," and the summary names "no client-owned simulation state" as one of the two core refusals.

- **Single canonical aviary, not client-to-client synchronization.** The plan repeats this in `Architecture` and `Sync model`: "one canonical aviary per account," "multi-device sync as a property of the architecture," "clients reading the same canonical record," "no CRDTs, no last-write-wins, no merging," and "the two devices never disagree because they don't have independent state."

- **Append-only events with deterministic, idempotent server interpretation.** This shows up in the client/server boundary, event log, API, tick, and conflict-prevention sections: clients "write events," the event log is "the only path from client to canonical state," `POST /aviary/events` is idempotent, and ticks are "deterministic from inputs" and "safe to restart."

- **No negative-drift / no Tamagotchi mechanics.** The plan carries this from `Scope` through `Drift function`, `Drift safety / no-tamagotchi enforcement`, `Risks`, and `Summary`: "monotonic-toward-expressive drift," "all non-negative," "personality_after >= personality_before," "no negative-drift," and no "hunger, distress, decay, death, neglect-driven negative drift."

- **No gamification, no announcement surfaces, no welcome-back pressure.** This appears in `Explicitly out of scope`, telemetry, risks, tests, and summary: no "achievements, streaks, levels, scores, badges," no "welcome back" textual surface, no "green-dot calendar," no "days since," and anti-toast tests for "welcome back / streak / achievement."

- **The aviary should feel already alive, not loaded into existence.** The plan emphasizes this in `Aviary first-frame conceit`, `Empty-aviary state`, `First bird perf risk`, and `Mood snap on tab open`: "there is no spinner-resolves-into-aviary transition," "the aviary has been there," the quiet field replaces a spinner, and the "central conceit" is that the first frame already has birds in pose and ambient motion seeded.

- **Affective change is slow, calibrated, and relationship-shaped.** This shows up in drift calibration, notebook cadence, rollout, and the summary: "instruments-detectable in ~7 days, user-perceptible in ~21," a "regular visitor" reference profile, new birds tuned to "match the rhythm of a relationship deepening," and the affective claim that the product "feels alive over weeks."

- **Privacy boundaries are architecture, not policy copy.** The plan grounds this in data model, hosting, telemetry, observability, and tests: email lives in one place, other references use synthetic UUIDs, the simulation database is firewalled from analytics, "telemetry pipelines never touch the simulation DB," aggregate-only RUM has "no per-account or per-bird dimensions," and privacy boundary tests are CI gates.

- **Accessibility is a first-class designed surface.** This is explicit in scope and developed across rendering, audio, and accessibility: "naturalist screen-reader narration," "reduced-motion mode (cross-fade aesthetic)," "procedural call captions," "full keyboard navigation," and reduced motion is "a designed surface, not a stripped fallback."

- **Product voice is naturalist, sparse, matter-of-fact, and non-congratulatory.** The plan uses "sparse naturalist prose," "descriptive of the aviary as a place," "matter-of-fact voice," "templates must be observations of the aviary, not of the user," and rejects both the "stock event-log" failure and the "AI-warbling" failure.

- **Bird identity and recognizability should survive drift and time.** The plan grounds this in call grammar, voice signature, rollout, and migration safety: each bird has a "stable per-bird voice signature," a user "should know Pip's call by ear," bird identity is "stable across migrations," and a bird's `id` is "never changed" and row is "never deleted."

- **Budgets and boundaries are code-level guards, not aspirations.** This appears in performance, observability, risks, tests, and summary: budgets are "CI gates, not goals," simulation observability "cannot ship without it," privacy and monotonicity checks are release blockers, and the two core refusals are "encoded in code-level guards."

- **Visits are ambient, opt-in, host-controlled, and non-social.** The plan carries this in scope, API, visitor session, and risks: visit-invitation is "off by default," "per-invite opt-in," "read-only ambient," "host-revocable," with "no co-presence," host-only notebook, default-off visit notifications, and no social-network surfaces.

## Per-feature whys

### Scope

- **Single-user accounts:** NOT RECOVERABLE FROM PLAN

- **One canonical aviary per account:** The plan's rationale is sync correctness: clients read "the same canonical record," so multi-device sync is not a separate subsystem and devices "never disagree."

- **Magic-link email auth:** NOT RECOVERABLE FROM PLAN

- **Web client only:** NOT RECOVERABLE FROM PLAN

- **Responsive single-screen scene:** The plan ties the scene to a browser product whose first screen is the aviary itself; the first render frame must already have "birds in pose and ambient motion seeded."

- **Server-side simulation tick:** The tick is the authority that makes state durable and shared: it is "the only writer" of personality vectors and mood, consumes append-only events, and lets the user return to "the aviary that has been ticking."

- **Five-dimension personality vector:** NOT RECOVERABLE FROM PLAN

- **Monotonic-toward-expressive drift:** The rationale is no-Tamagotchi enforcement: drift deltas are non-negative, the build fails on decrements, and the product refuses "neglect-driven negative drift."

- **Daily-cadence mood and mood persistence:** The plan gives mood a durable `entered_at` and `next_review_at`, and uses dwell gates "to prevent flicker," so mood reads as an ongoing state rather than a client animation artifact.

- **Procedural call grammar:** The plan uses procedural parameters so calls can be synthesized on the client, captioned from the same parameters, varied without recordings, and tied to a stable per-bird voice signature.

- **Mood-shaped idle motion:** Idle motion is "the visible expression of mood that the user reads without being told" while remaining purely client-local and non-canonical.

- **Approximately six species pool:** NOT RECOVERABLE FROM PLAN

- **Two starter birds:** NOT RECOVERABLE FROM PLAN

- **Hard cap of seven birds:** The plan links the cap to audio and recognizability risk: beta starts at 4 "to keep the audio-mix conservative" and widens to 7 "if the listenability holds"; the cap is tunable if recognizability gets stuck.

- **Age-based new-bird offers:** The plan says the timing is tuned to "match the rhythm of a relationship deepening," appears through a field-notebook entry plus a small top-bar affordance, and can be declined without affecting personality drift.

- **Return-greeting:** NOT RECOVERABLE FROM PLAN

- **Listen-in with gradual mix ramp:** The 800ms equal-power crossfade preserves the chorus; other birds drop to a floor but "never go silent," because hard mute would convert the aviary to "a mixer UI."

- **Offer of seed, song fragment, or still pool:** The offer is a global event resolved by mood, distance, curiosity, and cooldown, so birds may approach, ignore, or react rather than being directly controlled.

- **Settle with 5s undo:** The plan gives the window and animation behavior, but the feature rationale itself is NOT RECOVERABLE FROM PLAN

- **Field notebook:** The notebook is sparse, read-only, tick-written naturalist prose; the plan says this avoids both the "stock event-log" failure and the "AI-warbling" failure.

- **Presence accounting with visibility, focus, and recent input:** The 4-minute window is "long enough to allow watching without moving" and "short enough that an unattended laptop falls out of presence."

- **Three perch zones:** NOT RECOVERABLE FROM PLAN

- **Day/night by user local time:** The plan uses local time in lighting, mood transitions, and drowsy weighting, but a specific feature rationale is NOT RECOVERABLE FROM PLAN

- **Ambient weather and micro-motion:** Ambient motion supports the first-frame conceit: the aviary appears already in motion, and the quiet field uses subtle color modulation to signal continuity instead of brokenness.

- **Top bar with idle fade:** The fade keeps controls available while letting the scene stay quiet; input restores opacity immediately and `:focus-visible` raises opacity for keyboard focus.

- **Multi-device sync:** The rationale is that sync is "a property of clients reading the same canonical record," with no CRDTs, no last-write-wins, and no independent client state.

- **Visit-invitation feature:** The plan makes visits "off by default," opt-in, read-only, host-revocable, and TTL-bound to preserve the non-social, ambient product boundary.

- **Silent visit log and default-off visit notifications:** The plan keeps visits host-visible but non-pushy: no "your friend visited" push, visit notifications default off, and the log is operational rather than social-feed shaped.

- **Naturalist screen-reader narration:** Narration describes "the aviary as a place," not a state list, and shares templates with the notebook for voice continuity.

- **Reduced-motion mode:** Reduced motion is a cross-fade aesthetic and a separate render module so it remains "a designed surface, not a stripped fallback."

- **Procedural call captions:** Captions come from the same call parameters as synthesis, making graceful silence possible when audio is unavailable while keeping the product voice consistent.

- **Full keyboard navigation:** The plan gives a complete keyboard path through the top bar and aviary so canvas-rendered birds and controls are reachable without pointer input.

- **WCAG AA contrast on chrome:** The plan's stated rationale is accessibility as a "first-class designed surface"; all chrome text, captions, and narration overlays follow the contrast rule.

- **Account export:** The plan specifies JSON and an emailed link, plus point-in-time consistency, but the product rationale is NOT RECOVERABLE FROM PLAN

- **Soft delete for 30 days, then hard delete:** The plan gives retention behavior and restore behavior, but the product rationale is NOT RECOVERABLE FROM PLAN

- **Per-device session list and revocation:** The plan exposes session listing and peer-session revocation, but the product rationale is NOT RECOVERABLE FROM PLAN

- **Aggregate-only telemetry:** The rationale is to avoid reconstructing "a user's relationship with their birds"; telemetry has no account, bird, or event references, and analytics never reads the simulation DB.

### Architecture

- **Edge / web service with inline state-snapshot bootstrap:** The plan says this exists for performance and the first-frame conceit: it helps time to first bird and avoids a spinner-resolves-into-aviary transition.

- **Auth service:** NOT RECOVERABLE FROM PLAN

- **Aviary API:** The API is the boundary that reads canonical state, accepts append-only interaction events, and exposes settings, visits, export, narration, and captions without letting clients write personality state.

- **Simulation worker pool:** Workers make each account a "sharded, single-writer unit," compute deltas, write canonical state, and protect tick reruns through deterministic inputs.

- **Mailer:** The plan restricts it to magic links, export-ready notifications, and account-change confirmations because there is "no marketing surface."

- **Postgres canonical state plus append-only event log:** The plan ships v1 with a single relational DB and append-only events, leaving Kafka only "if scale demands later."

- **Read replica for the API path:** NOT RECOVERABLE FROM PLAN

- **Separate analytics warehouse:** Its rationale is privacy boundary enforcement: it "never reads from the canonical DB," and only aggregate categories reach telemetry.

- **Server-owned personality, mood, drift history, notebook, presence ledger, bird identity, and audit log:** These remain server-owned so clients submit events rather than canonical state.

- **Client-owned rendering, audio synthesis, idle motion, ambient effects, chrome, listen-in mix, captions, and narration display:** These are client concerns because they are presentation layers; idle motion has "no canonical effect."

- **Render pipeline snapshots as authoritative:** The plan says the render pipeline "does not introduce any state that survives a snapshot pull," so client aesthetics cannot become state.

- **Synthetic account UUID as sharding key:** The rationale is privacy: "no PII in the partition key."

- **Simulation DB firewalled from analytics:** The plan calls this a network-level rule, not a "we promise not to query" rule.

### Data Model

- **Email stored only on `accounts`:** Email "lives only here"; every other reference uses `accounts.id`, isolating PII from events, telemetry, logs, and partition keys.

- **Device label on `sessions`:** The label is user-visible, but a deeper rationale is NOT RECOVERABLE FROM PLAN

- **Stable bird identifier:** The plan says `birds.id` is "never reassigned," "never changed," and the row is "never deleted," preserving identity across migrations.

- **`retired_at` reserved but unused in v1:** The plan notes "no death" and reserves the column without using it, aligning the data model with the no-Tamagotchi boundary.

- **Personality drift history not surfaced:** Drift history may be reconstructable "for debugging," but the plan refuses personality-vector visibility to the user.

- **Mood `entered_at` and `next_review_at`:** These fields support "mood persists across sessions" and gate transitions to prevent flicker.

- **Aviary `empty` between adoption and first bird-fly-in:** This supports the one-time empty-aviary state; after first arrival, subsequent sessions "never see empty."

- **Append-only `events`:** Events are never updated or deleted except hard-delete; this preserves the only client-to-canonical-state path for deterministic ticks.

- **Read-only notebook entries written by the tick:** The rationale is that notebook prose is authored by the simulation, not the client, keeping it sparse, observational, and non-editable.

- **Visitor IP hash with rolling 7-day TTL:** The plan limits this to "abuse prevention only" and "nothing longer."

- **Telemetry records with no account, bird, or event references:** The rationale is aggregate-only observability that cannot reconstruct per-account or per-bird relationships.

### API Surface

- **`POST /auth/magic-link` returning 204:** The plan gives behavior and rate limiting, but the feature rationale is NOT RECOVERABLE FROM PLAN

- **`GET /auth/consume` single-use 15-minute TTL:** The plan gives behavior, but the feature rationale is NOT RECOVERABLE FROM PLAN

- **`GET /aviary/snapshot`:** It returns rendered-friendly state in a small payload so clients can draw birds, mood, perch, call timing seeds, weather, narration, and captions from canonical state.

- **Snapshot delta form with `since`:** The delta form supports keepalive pulls by returning "just what's changed" after a snapshot sequence.

- **Bulk `POST /aviary/events`:** The bulk endpoint exists so "a flaky network doesn't fragment the event stream"; idempotency drops duplicate `(account_id, client_seq)` inserts.

- **Paginated read-only notebook API:** The read-only part follows the notebook boundary; a separate pagination rationale is NOT RECOVERABLE FROM PLAN

- **`PATCH /account` for accessibility prefs, visit-notify, and bird names:** Bird-name updates "propagate without affecting any other state," keeping naming separate from personality and mood.

- **Account export endpoint:** Export reads at "a single point-in-time consistent snapshot" and "does not block ticks."

- **Visit invitation creation and revocation endpoints:** The flow gives host control; revocation has immediate effect, and the next visitor pull returns a matter-of-fact unavailable surface.

- **Visitor-side snapshot endpoint:** Visitor sessions are a separate auth class, read only scoped fields, cannot submit events, and cannot read settings, invitation list, visit log, or notebook in v1.

- **Visitor notebook excluded in v1:** The plan explicitly calls this out "so it's not added by accident," preserving the host-only notebook boundary.

- **Event ordering by `occurred_at` and `client_seq`:** The tick orders the append-only log before processing, so out-of-order arrivals are processed correctly.

- **Rate limiting:** Auth endpoints are stricter and visit endpoints rate-limit per host; the event rate limit is generous because normal presence pings and user actions are well below the cap.

### Simulation Engine Design

- **60s tick cadence with jitter:** The 60s cadence is "cheap," under the p99 5s alarm budget, and gives mood transitions "believable granularity"; jitter spreads workload.

- **Per-account advisory lock:** The lock ensures "one tick per account at a time," protecting canonical state from concurrent tick writes.

- **Idempotent ticks:** A worker crash mid-tick is safe because reruns are deterministic from inputs.

- **Drift inputs from presence, listen-in, offers, and chorus:** The plan uses these interaction signals to compute non-negative deltas and calibrate visible personality change.

- **Drift weights against a regular visitor profile:** Weights are set so a regular visitor crosses instrument-detectable change in about 7 days and user-perceptible change in about 21 days.

- **Drift clipping:** Once a trait saturates, "additional input is a no-op," keeping traits inside the normalized range.

- **Anti-thrash floor and pending deltas:** Small deltas accumulate until crossing a threshold to avoid storage churn and keep drift "instrument-detectable rather than instrument-noise."

- **Mood transition probabilities:** Mood transitions use recent interactions, time of day, novel inputs, weather, alarm calls, and low-boldness/low-presence recency to keep mood responsive without being arbitrary.

- **Minimum mood dwell time:** `next_review_at` prevents flicker by requiring at least about two ticks per mood.

- **Deterministic bird-engine planner:** Mapping personality, mood, perch layout, and recent events deterministically to next perch and call parameters makes tick reruns idempotent.

- **Bird-to-bird mood spread:** The plan defines the behavior, but the feature rationale is NOT RECOVERABLE FROM PLAN

- **Chorus events:** The plan defines when chorus events occur and records them for drift, but the feature rationale is NOT RECOVERABLE FROM PLAN

- **Server-emitted call parameters:** The server sends `motif_id`, timing, pitch, seeds, and mood inflection while the client renders audio, keeping canonical state small and procedural.

- **Stable per-bird voice signature:** Recognizability across drift comes from immutable formant offsets and timbre coefficients; drift changes timing and frequency, "never the signature."

- **Global offer resolution:** Offers are global and resolved by bird mood, distance, curiosity, and cooldown so reactions emerge from bird state instead of direct targeting.

- **Ticking when no client is connected:** This lets the user return to "the aviary that has been ticking," not "the aviary as it was when they left."

- **Notebook candidate scoring and rate limiting:** Salience scoring plus a 24h floor makes notebook entries sparse and noteworthy rather than a stock event log.

- **Hand-authored notebook templates:** Stable templates avoid the "stock event-log" failure and the "AI-warbling" failure.

- **Shared notebook and narration templates:** Shared templates make the field notebook and screen-reader narration "speak the same product voice."

- **Drift monotonicity unit test:** The test is the implementation-level guard for monotonic-toward-expressive behavior and fails the build on decrements.

### Sync Model

- **Snapshot pulls on load, visibility, focus, suspend recovery, keepalive, and user action commits:** These pulls keep the client aligned with canonical state after normal viewing, return, laptop suspend, and interactions.

- **No client-to-client sync:** The plan rejects CRDTs, last-write-wins, and merging because both devices pull from the same canonical record.

- **Server-delta personality updates:** Personality is mutated by simulation-worker `server_delta` writes, not client absolute values, eliminating accidental client ownership.

- **API DB role without personality write access:** The DB role makes the boundary enforceable even if an API code path tries to write `bird_personality`.

- **Clock skew tolerance and concurrent device events:** Two devices' events need not be strictly cross-ordered because the tick treats them as concurrent inputs to additive deltas.

- **Account export point-in-time snapshot:** The exported JSON is consistent with one tick boundary and does not block ticks.

### Frontend Rendering Pipeline

- **TypeScript and Vite with route code-splitting:** The plan uses code-splitting for account, accessibility, and visit-invitation routes to protect the bundle budget.

- **Canvas2D primary rendering:** Canvas2D is chosen over WebGL because bird visuals are stylized SVG-like, GPU is not needed for the visual budget, and it keeps bundle and cross-browser concerns smaller.

- **Render loop gated by visibility:** Hidden-tab rendering pauses while server simulation continues, so resume is a snapshot pull rather than background client simulation.

- **Layered scene composition:** Ambient background, perches, birds, ornaments, captions, and chrome are separated so the render path can be cached, composed, and budgeted.

- **Snapshot interpolation:** Hermite interpolation makes perch hops render as "a smooth arc, not a teleport."

- **Late-snapshot extrapolation and hold:** The client extrapolates for up to one tick and then holds, avoiding uncontrolled motion when snapshots are missed.

- **Idle motion randomized timing offsets:** This makes "the same bird in the same mood" look different across sessions while keeping the motion aesthetic-only.

- **Inline initial snapshot:** This is "non-negotiable" because the spinner pattern compromises the central conceit.

- **Quiet field fallback:** When no snapshot is available, the quiet field avoids text and spinner while slow color modulation signals continuity.

- **Reduced-motion cross-fade path:** The separate render module keeps the regular path lean and makes reduced motion a designed surface.

- **Top-bar fade controller:** It holds opacity after input, fades to about 0.15, and restores on input or focus so controls remain reachable without dominating the scene.

- **Settle animation reversal:** Reversal is animated rather than a snap, preserving the lighting-and-audio continuity of the settle action.

- **First bird fly-in only for empty aviary:** The first-arrival event gives the initial adoption moment; after that, "subsequent sessions never see empty."

- **Object pooling, bitmap caches, and frame guardrails:** These protect 60fps idle and no memory growth by reducing allocator pressure and dropping particle count under sustained frame-time load.

### Audio Pipeline

- **Procedural call synthesis from motifs:** Motifs plus personality, mood, voice signature, and seeds create calls that vary while staying species- and bird-specific.

- **WebAudio graph with cleanup and node pooling:** Per-call nodes are cleaned on call end and pooled to avoid allocator pressure.

- **Frozen `voice_sig`:** The plan explicitly implements the goal that a user who has spent two weeks with Pip "should know Pip's call by ear."

- **Chorus mixing:** Procedural independent parameters avoid phase-cancel artifacts from stacked recordings, and stereo spread keeps chorus legible.

- **Listen-in mix decay:** Other birds drop to a floor rather than silence so the chorus remains present and the aviary does not become a mixer UI.

- **Procedural ambient bed:** NOT RECOVERABLE FROM PLAN

- **WebAudio fallback to silence plus captions:** The plan rejects recorded-audio fallback; "silence + captions is the fallback."

- **Captioning generation from call parameters:** Captions stay aligned with synthesis and use hand-authored naturalist phrases with anti-overlap handling for chorus events.

- **No autoplay surprise:** First-frame silence is "a feature, not a bug" because the user sees motion before audio, "just like opening a window."

- **Audio observability:** Error counts, dropped calls, and buffer underruns are aggregated, with "no per-call audio fingerprints."

### Accessibility Surfaces

- **Screen-reader live region:** A `role="status"` polite live region provides running prose without visual dependence.

- **Replacing rather than appending narration:** The plan replaces each prose chunk so it does not "overflow the screen reader queue."

- **Narration as place description:** Narration says "a small grey bird is perched..." rather than "Pip: content," preserving naturalist product voice.

- **Separate live regions for captions and narration:** Separate regions allow call captions and slow narration to co-exist.

- **Keyboard tab and arrow navigation through birds:** The plan provides keyboard access to top bar, aviary scene, bird focus, listen-in, offers, and settle.

- **Settle keyboard shortcut:** The shortcut is a "quiet shortcut" for keyboard users who want to settle without locating the icon.

- **Canvas focus indicator:** The two-tone outline is rendered in the canvas overlay so it reads against bright and dim aviary states.

- **Matter-of-fact settings copy:** Accessibility settings use the named matter-of-fact exception and immediate toggles rather than a save flow.

- **Always-emitted narration API:** The API always emits narration; the live region is always present, and captions auto-enable when `AudioContext` is unavailable.

### Performance Budgets and Observability

- **Initial JS bundle budget:** CI fails the build on regressions; code-splitting, tree-shaking, and no recorded audio keep the first paint under the cap.

- **Time to first bird visible:** Edge-rendered HTML, inline snapshot, preloaded bird assets, and non-blocking render paths support the central first-bird budget.

- **60fps idle motion:** Canvas2D, layered canvases, bitmap caches, particle pool, render guardrails, and small-vector idle math exist to keep p99 frame time within budget.

- **No memory growth over 30 minutes:** Object pooling and explicit cleanup on calls, transitions, and notebook scroll-out support the long-run memory gate.

- **Aggregate-only RUM:** RUM measures page load, first-bird render, frame-time histograms, audio errors, reduced motion, and captions without per-account or per-bird dimensions.

- **Synthetic perf fleet:** Scheduled headless browsers in multiple geographies assert all four budgets per release.

- **Simulation observability by shard:** Tick latency and queue depth are keyed by shard, not account, preserving operations visibility without per-account data.

- **What not to measure:** The plan refuses per-bird drift trajectories, per-account session frequency, and interaction rates because they could tempt streaks or reconstruct relationships.

### Rollout, Risks, and Tests

- **Internal alpha:** Alpha is for calibrating drift weights, notebook cadence, reduced-motion aesthetic, and screen-reader feedback.

- **Closed beta with bird cap at 4:** The cap is conservative while chorus behavior and listenability are observed at scale.

- **Open launch with visit invitations on day 1:** Visit-invitation ships with visit-notify default off, preserving the quiet host-controlled visit stance.

- **Per-account tunable bird cap:** The plan allows lowering the cap without a release if recognizability gets stuck during beta.

- **Birds-per-aviary ramp:** The ramp spaces new birds over weeks and months to match "the rhythm of a relationship deepening."

- **Declining a new bird offer:** Declining does not affect personality drift, avoiding punishment or game mechanics.

- **Day-one instrumentation:** Aggregate RUM, synthetic perf, simulation observability, privacy tests, calibration tests, memory tests, bundle gates, and 60fps tests are present from day 1 because the plan "cannot ship without it."

- **Gradual post-launch config ramp:** Drift weights, mood probabilities, notebook thresholds, and tick cadence ramp 5%, 25%, 100% over a week to avoid users feeling "their birds changed" from a config flip.

- **Online migration path for bird tables:** Add-column, dual-write, backfill, switch-read, drop-old preserves stable bird identity through schema changes.

- **Drift calibration test:** It mitigates too-fast Tamagotchi-feel and too-slow screensaver-feel with light, regular, and heavy visitor profiles.

- **Sync correctness tests and DB roles:** They mitigate accidental client personality writes and last-write-wins refactors.

- **Audio listening tests and two-call sameness test:** They mitigate synthesizer-y calls, blurred chorus, and calls sounding the same twice.

- **Accessibility regression gates:** PR templates, reduced-motion render snapshots, and narration coverage keep accessibility from becoming a later "v1.1 fix."

- **Privacy linter and telemetry schema allowlist:** They prevent email-as-key behavior and per-bird state leaking into telemetry.

- **Gamification creep checklist:** Non-goals in PR review and design review prevent engagement surfaces like counters, calendars, streaks, and badges.

- **Performance budget gates:** Bundle, frame-time, and memory checks fail the build rather than remaining goals.

- **Anti-toast string test:** The crude DOM string guard catches obvious "welcome back," "days since," "streak," "achievement," "level up," "badge," and "score" mistakes.

- **Visitor abuse mitigations:** Named-email invitation, per-token rate limits, host revocation, and 7-day IP hash retention address open-ended link abuse.

- **Cold-edge quiet-field mitigation:** Slow color modulation signals continuity, and p95 cold-load time over 2s alerts.

- **Mood snap mitigation:** No client-side default mood exists; the render path either uses the bootstrap snapshot or renders the quiet field.

- **Notebook voice-owner review and linter:** Change control and rejected announcement phrasing keep notebook prose from drifting toward generic or congratulatory language.

- **Cross-cutting test list:** The consolidated CI gates exist for "ease of audit" and encode drift, sync, privacy, performance, audio, accessibility, visit revocation, and snapshot-only source-of-truth boundaries.

### Open Implementation Details

- **Exact normalized ranges for traits:** NOT RECOVERABLE FROM PLAN

- **Final tick cadence value:** The plan defers exact tuning but gives the rationale for the nominal 60s value as cheap, under alarm budget, and believable for mood granularity.

- **Final activity window for presence:** The plan defers calibration but gives the nominal 4-minute rationale: watching without moving while unattended laptops age out.

- **Listen-in disengage trigger on empty aviary:** NOT RECOVERABLE FROM PLAN

- **Nightjar-like night-active species:** NOT RECOVERABLE FROM PLAN

- **Visit log retention:** The plan treats the visit log as host-visible operational data retained while the account exists and hard-deleted on account hard-delete.

- **Bird offer presentation surface:** The proposed field-notebook entry plus small top-bar pulse avoids a modal and remains "refusable."
