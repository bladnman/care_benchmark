## System-level intent

- The server is the canonical aviary. This appears in the opening invariants as "the server is the only writer of canonical aviary state," then recurs in Architecture, API, Simulation, and Sync: clients "submit events and render snapshots," no endpoint accepts "absolute state," and "nothing merges because nothing forks."

- The product is affective before it is utilitarian. The opening says "the product's value is affective" and every surface must read as "a place that was already running." This explains the bans on "spinners, toasts, loops, state-list narration," the first-frame rule that birds appear "mid-action," and the launch audit for "no spinner anywhere."

- Product boundaries are enforced structurally, not only by policy. The Scope says banned items get "structural enforcement." That shows up as absent tables for "days_visited," no API for trait values or visit counts, a visit path with "no events endpoint at all," copy lint, schema review, and telemetry schemas that cannot admit account or bird dimensions.

- Hidden inner life should be expressive but not exposed. The plan repeatedly protects "the never-expose-the-vector rule": personality values do not leave the server, no UI or API exposes trait numbers, screen-reader narration never includes "trait values" or "mood labels as labels," and mood is meant to be "readable from motion alone."

- The aviary evolves gently and never punishes absence. The plan uses "monotonic-toward-expressive drift," bans "death, hunger, distress, decaying meters, negative drift," and later states neglect gives "zero signal, hence zero drift." The phrase "quieter, not punished" is the clearest statement of this philosophy.

- Continuity matters across time, identity, and devices. The plan uses "one canonical record," stable `bird_id` "forever," stable `call_seed`, transactional tick cursors, deterministic catch-up, and "no last-write-wins anywhere in the state path." A bird and an aviary are continuous rather than re-created per session.

- The voice has two registers and must not drift. The plan separates "naturalist" prose for notebook, narration, and captions from "matter-of-fact" copy for auth, errors, settings, sync, and unsupported-browser surfaces. It also calls out "voice erosion" and requires separate copy catalogs plus banned-lexicon lint.

- Accessibility ships as a designed version of the product. Scope says accessibility is "shipped at launch, not after." The Accessibility section rejects a separate "a11y phase," and the risk section warns against "checklist-mode," requiring reduced-motion and narration to still "feel alive."

- Privacy is an architecture boundary. The telemetry section says "aggregate-only" and explicitly forbids measuring anything that could "reconstruct a user's relationship with their aviary." The plan separates the simulation database and telemetry pipeline physically and rejects any ETL path between them.

- Performance is part of the spell. The budgets are framed as CI gates, but also as product invariants: "first bird visible < 500ms," "mid-action," no memory growth, and "first-bird-render timing" as the "affective-perf bridge metric." Slow or stock loading is treated as a product failure.

## Per-feature whys

### 1. Scope

- Single-user accounts: NOT RECOVERABLE FROM PLAN

- Email magic-link auth: The plan uses constant 202 responses, 15-minute links, atomic single-consume, and rate limits to avoid an "account-existence oracle" and reduce "link replay" and email enumeration.

- Per-device revocable sessions: The plan lists sessions in settings and uses revoke as part of account security; later, revoked sessions are treated as "auth-shaped" conflicts rather than state conflicts.

- Email change with verification: NOT RECOVERABLE FROM PLAN

- Account export as JSON by emailed link: NOT RECOVERABLE FROM PLAN

- Soft-delete for 30 days, then hard-delete: The 30-day period supports the restore surface named "I changed my mind"; hard delete later cascades account data.

- One canonical aviary per account: This is the basis for sync being "mostly a non-feature"; there is one record to read and no forked state to merge.

- Two starter birds at adoption: NOT RECOVERABLE FROM PLAN

- Cap of seven birds: The plan ties the cap to chorus-recognizability and audio performance; rollout extends schedules toward 7 only as "listening data and audio-perf telemetry confirm the cap holds."

- New-bird offers gated on aviary age only: The reason is structural anti-gamification. Offers do not depend on visit streaks, presence totals, scores, or other user-behavior measures.

- Server-side simulation tick: The tick advances state "whether or not a client is connected," making the aviary feel like "a place that was already running" and preserving server-only authorship of state.

- Hidden five-trait personality vector, mood, and drift: The vector gives birds expressive change, while hidden server storage and derived render hints enforce that traits never become exposed stats.

- Bird-to-bird interaction: Call/response, mood contagion, and emergent chorus make the scene feel alive as a shared aviary rather than independent animated objects.

- Procedural call synthesis: The plan wants "per-bird recognizable call signatures," "chorus mixing," no recorded-audio fallback, and no loops; synthesis supports recognition, variation, and small assets.

- Return-greeting: It is part of the first-paint affective experience. The plan says greeting begins within the first 1-2s and is "part of the first-paint critical path."

- Idle presence accounting: The plan says "watching without moving is the product"; presence uses visibility, focus, and recent input so attention can matter without encouraging clicking.

- Listen-in: The acceptance criterion is "listening, not channel-switching," hence slow mix ramps and other birds lowered to ambient rather than muted.

- Offer interaction: The plan makes offers server-validated with cooldowns so they can nudge curiosity or boldness without letting a "heavy clicker" saturate drift.

- Settle with 5-second undo: Settle cleanly ends presence and quiets the aviary; the undo reverses the lighting ramp and avoids making the action feel final in the moment.

- Field notebook: Its purpose is sparse "naturalist voice" observation of the aviary. The plan forbids attendance facts, keeps it read-only, and uses source facts for regeneration and QA.

- Single non-scrolling horizontal scene: The rendering section explains the layout solver keeps all birds in frame and must "never crop a bird."

- Three perch zones: NOT RECOVERABLE FROM PLAN

- Local-time day/night cycle and rare weather: These give the aviary a shared ambient world; weather is scheduled by server snapshot so two devices see the same rain or wind.

- Client-side leaves and feathers: The plan says these are "rendering ornaments, not simulation," so they add ambience without becoming canonical state.

- Fading top bar: NOT RECOVERABLE FROM PLAN

- Quiet-field loading and empty states: This implements the no-spinner rule. The fallback is quiet field because a spinner or progress bar would break the "already running" illusion.

- Multi-device sync: The why is "one canonical record"; two devices are readers and event appenders, not peers that need conflict resolution.

- Read-only visit by emailed one-time link: The visitor should see the host's real aviary without "show-off rendering," account-scoped data, or any path that lets visitors drift the host's birds.

- Visit log in settings: NOT RECOVERABLE FROM PLAN

- Visit notifications off by default with opt-in toggle: NOT RECOVERABLE FROM PLAN

- 30-day invite expiry: The risks section frames token abuse as a concern; expiry, revocation checks, and token scope mitigate invite sharing and stale access.

- Accessibility shipped at launch: The plan treats accessibility as part of the product's affective value, not a later checklist; it explicitly says the product does not ship without those surfaces.

- Performance budgets as CI gates: The plan treats speed, frame rate, memory, and tick latency as product constraints; regressions are "blocked merge" issues.

- Aggregate-only telemetry: The plan refuses per-bird or per-account interaction data so telemetry cannot reconstruct a user's relationship with the aviary.

### 2. Architecture

- API service: The plan makes it stateless and horizontally scaled because it handles auth, reads, event appends, notebook, account, visit, export, and deletion flows.

- Simulation service with shard ownership: The simulation worker is split out because tick load scales with total accounts, and shard locks ensure "exactly one worker owns an aviary's tick at a time."

- Notification/mail worker: The plan isolates mail so "mail-provider latency never touches the API path."

- Edge/CDN app shell and bootstrap snapshot: The reason is first paint: "first paint doesn't wait on an origin round trip."

- Postgres event log adjacent to state: The plan keeps events and state transactionally adjacent so the "no-lost-drift guarantee" is easy to prove, while preserving a cursor abstraction for a later stream.

- API and simulation as two services from one codebase: Two services because scaling profiles differ; one codebase because "the data model is shared and small."

- Asymmetric client/server split: The server owns identity, vectors, moods, positions, scheduling, and authorization; the client owns interpolation, rendering, audio, ornaments, and accessibility assembly so canonical state remains server-only while the client can feel rich.

- Snapshot render boundary: The snapshot is small and carries only derived render hints; personality numbers "physically never leave the server," so client bugs and devtools cannot leak them.

### 3. Data model

- Synthetic UUIDs and one encrypted email column: The reason is PII containment: email is never a key, partition value, log field, or telemetry dimension.

- Account settings JSON for timezone, captions, reduced motion, and visit notifications: NOT RECOVERABLE FROM PLAN

- Magic-link single-use atomic consume: The plan uses `consumed_at`, `token_hash`, and atomic consume to prevent replay.

- `aviary.created_at` as the only input to new-bird offers: This enforces "aviary age" gating and prevents visit or engagement metrics from controlling bird growth.

- Stable `bird_id` forever: This is "the identity-continuity rule made concrete"; no migration, sync, or species-pool change may re-issue it.

- Stable `call_seed`: It anchors per-bird randomness so "the call signature survives drift and re-renders."

- `retired` always false in v1: It exists so future migrations "never delete a row."

- Personality table written only by the tick: This preserves the single-writer rule for traits and keeps the hidden vector off client write paths.

- Mood persisted across sessions: The plan says nothing resets mood on connect, so a bird is continuing rather than being reinitialized.

- Append-only interaction events with per-aviary sequence: The tick consumes by cursor, giving ordered event integration and no updates or deletes except account hard-delete.

- Notebook `source_facts`: These support regeneration and QA while staying internal; the allowlist prevents facts about user attendance patterns.

- No stored presence counter or day rollup table: The absence of these tables is the "anti-streak rule enforced structurally."

- Static species pool with exactly one nocturnal nightjar: NOT RECOVERABLE FROM PLAN

### 4. API surface

- Matter-of-fact auth and error bodies: The plan keeps system copy in a different register from naturalist prose; "The link may have expired" is the sample voice.

- Account restore endpoint: The restore flow provides the "I changed my mind" surface during soft delete.

- Snapshot endpoint with greeting block: The snapshot is the render contract, and the greeting block lets the server decide return-greeting while the client performs it immediately.

- Notebook endpoint with keyset pagination and prose only: This supports read-only infinite scrollback without exposing source facts.

- No endpoint for personality values, presence totals, visit counts, ranks, or trait numbers: This enforces hidden traits, privacy, and anti-gamification at the API surface.

- Batched event append with idempotency keys: The plan says retries on flaky mobile networks must not double-record offers.

- Server-side offer validation: The client may disable the affordance, but the server is "the enforcement point" for per-bird cooldowns.

- Tokened visit snapshot matching host snapshot minus account fields: Visitors see the same birds, moods, and weather, with no separate show-off mode.

- Visit path with no events endpoint: Visitor presence and interaction are "structurally unrecordable," so visitors cannot drift host birds.

- Starter adoption where the server picks two species: NOT RECOVERABLE FROM PLAN

- Bird-offer schedule computed from `aviary.created_at`: The schedule is age-gated, lengthening, and capped at 7, keeping growth out of engagement mechanics.

- Rename endpoint that "touches nothing else": Rename does not change identity, species, mood, drift, or call continuity.

### 5. Simulation engine design

- Tick cursor advanced in the same transaction as state writes: This is the "no-lost-drift guarantee"; a crash retries the whole tick instead of consuming events without effects.

- Presence clamps and multi-device deduplication: A client cannot replay pings to inflate drift, and multiple screens count as one watching window.

- Drift dominated by presence rather than clicks: The plan requires a "heavy clicker" profile to drift less than a regular visitor, making watching the primary signal.

- Diminishing returns and per-tick caps: Traits move visibly over time but "asymptote rather than pinning at 1.0," and cooldowns prevent single-session saturation.

- Monotonic drift with no negative term: Neglect gives "zero signal" rather than punishment, and property tests assert no trait can decrease.

- Calibration targets at 7 and 21 simulated days: The plan wants instrument-detectable change within a week and render-visible change within about 21 days.

- Recency factor with a floor: A bird can greet less after absence because "less has been observed recently," but the floor guarantees it still greets sometimes.

- Hysteretic mood machine: Minimum dwell makes mood "read as weather, not flicker."

- Ambient scheduling for day phase and weather: Server scheduling gives shared weather across devices and visitors, with local-time phase computed from account timezone hints.

- Notebook sparsity governor: The plan wants at most about one promoted entry every few days, based on noteworthiness and a decaying threshold.

- Deterministic lazy catch-up for dormant aviaries: This keeps tick-fleet cost "proportional to active users, not total accounts" while requiring the same end state as minute ticks.

- Server schedules call timing while client synthesizes waveforms: Timing must be canonical across devices and visits; waveform realization can be seeded and per-client without shipping audio.

### 6. Presence, greeting, and session semantics

- 4-minute presence input window: The plan chooses this because "watching without moving is the product" and the PRD said to "lean long."

- Presence ping with no content and no ended event: The server treats ping gaps as the window closing, making settle and tab close "identical at the engine level."

- Server-decided, client-performed return-greeting: The server can weight boldness, mood, absence, and seeded jitter; the client can make the greeting immediate and procedural.

- Varied greeter and staggered second-bird response: The plan avoids it being "always the same bird" and prevents simultaneous canned-feeling greetings.

- Settle local lighting ramp and undo: It quiets the aviary and ends presence cleanly, while undo reverses the ramp if the click was accidental.

### 7. Sync model

- Single canonical record for sync: Sync is "mostly a non-feature" because clients only append events and read snapshots.

- No last-write-wins state path: The plan makes overwrite hazards unreachable by accepting events rather than absolute state.

- Snapshot repull and interpolation: Clients re-pull after visibility changes, sleep gaps, and keepalives; interpolation renders flights or cross-fades, "never a teleport."

- Offline event queue with stale presence dropped: Real interactions can flush later, but stale presence is not back-credited.

- No user-facing sync conflict resolver: Remaining conflicts are "auth-shaped, not state-shaped," so a resolver would imply a failure mode the architecture avoids.

### 8. Frontend rendering pipeline

- Renderer choice by one-week spike against 2MB budget: The plan says "the budget, not familiarity, decides."

- Layered sky, foliage, perches, birds, and foreground: NOT RECOVERABLE FROM PLAN

- Procedural skeletal birds rather than frame-by-frame sprite sheets: The plan wants micro-motion from seeded noise so it "never loops detectably."

- Mood readable from motion alone: This is the "no mood labels" contract; wary, content, curious, and drowsy are conveyed by posture and behavior.

- Weather and palette driven by snapshot: This keeps weather and day phase consistent across devices.

- Responsive layout solver: Its purpose is to keep all birds in frame at all sizes and "never crop a bird."

- Critical micro-bundle plus bootstrap snapshot: This serves first bird render under 500ms without waiting for origin.

- First frame mid-pose, no entry animation: The plan wants the bird to appear as if the motion was already happening.

- Code-splitting audio, motif libraries, notebook, and settings: These load after first paint so the critical path stays small.

- Quiet-field slow-path fallback: This satisfies "Never a spinner, never a progress bar" while preserving the product tone.

- Object pools, reused buffers, virtualized notebook, bounded workers and audio contexts: These enforce zero memory growth over a 30-minute session.

- Hidden-tab teardown of draw loop, audio, and presence pings: The client stops pretending presence and resumes from a fresh snapshot when visible again.

- Reduced-motion as cross-fade register: It uses the same snapshot and engine but a different visual register, not a flag that disables charm.

### 9. Audio pipeline

- Per-species motif library as synthesis recipes, not samples: This keeps calls procedural and avoids recorded audio in the product.

- Per-bird call signature from motif, call seed, mood, and trait modulation: The listening requirement is "Pip is Pip by ear at week 0 and week 6."

- WebAudio graph with per-bird voices, pan, chorus bus, and master: Two birds calling are two live syntheses, so "the chorus is real, never layered loops."

- Pre-allocated voices, reused buffers, and AudioWorklet: The reason is performance and keeping synthesis work off the main thread where available.

- Gentle audio fade-in after visual first paint: The plan allows the aviary to be visually alive a beat before sound, but says to fade ambience rather than "popping on."

- Listen-in mix ramps with other birds lowered but never muted: This implements "listening, not channel-switching."

- WebAudio unavailable means graceful silence plus captions auto-enabled: The plan avoids recorded fallback and gives a matter-of-fact explanation in settings.

- Captions generated from actual synthesis parameters: Because captions derive from the realized call, they "never desync from the audio."

### 10. Accessibility surfaces

- Shared naturalist phrase-grammar engine: It keeps notebook, screen-reader narration, and captions in one continuous voice and gives copy review "a single audit point."

- Separate system copy catalog: Matter-of-fact strings live separately so the naturalist/system voice line is enforceable in review.

- Polite ARIA live narration every 30-60s: The plan wants paced, deduplicated observations that do not re-describe unchanged scenes.

- Prompt narration for user-initiated events: These are written as observations, "never state transitions," so narration reads the same scene eyes would.

- Narration never exposes trait values, mood labels, coordinates, or event dumps: It avoids turning accessibility into a stats or debug surface.

- Focusable birds with naturalist accessible names: Names are descriptions like "a small grey bird on the front rail," not stat lines.

- Reduced-motion honors preference and has a settings override: The mode is first-class, vestibular-safe, and still has to pass an "is it still charming" review.

- Keyboard tab order and bird navigation: The plan gives a full keyboard path for top bar, scene, birds, listen-in, offer, and settle.

- Focus ring and top-bar behavior: Keyboard activity restores opacity, and the top bar fade never hides a focused element.

- WCAG AA contrast for all user copy: Automated visual tests check chrome and captions against brightest-day and darkest-night palette extremes.

### 11. Performance budgets and observability

- CI-gated bundle, first-bird, frame-rate, memory, and tick budgets: The plan says "regression = blocked merge."

- Aggregate metrics for request health, tick health, render timing, audio fallback, mail latency, and synthetic fleet: These measure product and operations without per-account identity.

- Deliberately banned measurements: The plan refuses per-bird state, per-account history, presence totals, visit counts, drift rates per account, and even population aggregates over per-bird interaction data.

- Separate telemetry module, typed metric schema, and no warehouse credentials to simulation DB: This is the "privacy boundary as architecture."

### 12. Rollout

- Foundations phase with renderer and audio spikes first: The plan calls them "the highest-risk items" and starts them in week 1.

- Vertical slice with two birds end-to-end: The exit criterion proves the affective path: cold tab, throttled phone, bird mid-action under 500ms, greeting within 2s, and same moods/weather on two devices.

- Full surface phase: It brings offers, settle, notebook, day/night, weather, species, reduced motion, narration, captions, keyboard, settings, export, and deletion into the v1 product.

- Social and hardening phase: Visit flows, perf gates, memory stability, security review, and paid assistive-tech testing all happen before calibration beta.

- Calibration beta: The plan says real users are needed to tune what cannot be calibrated synthetically: drift feel, presence window, notebook sparsity, greeting variety, call recognizability, and audio uncanniness.

- Remote-config knobs: Calibration changes should not require deploys.

- Ramping birds per aviary: The engine ships tested for 7, but offer schedules expand only after field data confirms recognizability and audio performance.

- Day-one instrumentation: The plan measures first-bird-render as the "affective-perf bridge metric" and tracks upstream error-surface spikes, while keeping nothing per-account.

- Launch checklist: The checklist is a product-invariant audit for no spinner, no return toast/banner/welcome, no streak disguise, no trait numbers, charming accessibility, correct voice, and "quieter-not-sad" neglect return.

### 13. Risks and mitigations

- Drift calibration tests and remote-config rates: The why is the top product risk: "Too fast = Tamagotchi; too slow = screensaver."

- Presence module tests, clamps, and unfocused canary: The plan warns that lax presence can "silently inflate drift population-wide."

- Sync chaos test and single-writer leases: These prove no event is double-applied or dropped when workers die mid-tick.

- Audio spike, sound designer, and listening tests: The plan says cheap synthetic sound or mushy chorus would "break the affective spine."

- Accessibility built on the same engines as primary surfaces: This prevents drift toward the "cheap version" of ARIA labels and animations-off.

- Import-boundary lint and per-PR bundle gate: The plan says "2MB and 500ms die by a thousand dependencies."

- Lazy catch-up equivalence tests and shard rebalance runbook: These mitigate tick fleet cost and p99 latency at scale.

- Copy catalogs, banned lexicon, and fact-type allowlist: These prevent "gamified or announce-y copy" from leaking into the naturalist register.

- Magic-link and visit-token expiry, atomic consume, rate limits, and revocation checks: These address "link replay, invite-token sharing, email enumeration."

- Schema review, log-field lint, and typed metrics schema for PII: These keep email from "creeping into logs/keys/metrics."

### 14. Open calls made in this plan

- Presence input window of 4 minutes: The plan grounds this in "a few minutes, lean long" and "watching without moving is the product."

- Offer cooldown of 3 minutes per bird: The plan ties this to "a few minutes" and server enforcement, but gives no deeper rationale for exactly 3 minutes.

- Mood enum of wary, content, curious, drowsy, alert, plus internal `settled_night`: NOT RECOVERABLE FROM PLAN

- Third-bird offer at about 3 months with lengthening intervals: The plan maps this to the stated expectation that a few-month aviary offers a third bird and a year-old aviary may have five or six.

- Event log in Postgres partitions with per-aviary cursor: The plan repeats the v1-scale rationale: transactional adjacency now, cursor abstraction for later migration.

- Renderer library decided by spike: The plan refuses to pre-commit because the bundle budget decides.

- Timezone from client-reported IANA zone with offset fallback: This supports server-side day-phase computation during absence.

- Visitor snapshot reuse with no separate visit renderer: The plan says this structurally enforces "no-show-off-mode."
