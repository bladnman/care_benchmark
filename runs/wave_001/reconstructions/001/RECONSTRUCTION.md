## System-level intent

1. **Affective constraints are engineering constraints.** The plan says it "deliberately treats the affective constraints from the PRD as engineering constraints" and makes "Notice, never announce," "presence is real interaction," "drift is monotonic toward expressive," "personality is server-authored," and the "seven-bird cap" properties of the system. This shows up as runtime invariants, database triggers, CI gates, API boundaries, and code-review questions rather than as style guidance.

2. **The server owns canonical aviary state.** The central boundary is: "The server is the only writer of canonical aviary state. The client renders snapshots and submits append-only events." This appears in the client/server split, database role privileges, event endpoints, simulation tick, and sync model. The why is that "last-write-wins on personality silently destroys drift."

3. **Aliveness should be slow, calibrated, and non-extractive.** Drift is a slow EMA calibrated for "instrument-detectable" change after about 7 days and "user-detectable" change after about 21 days. The plan guards against both "Tamagotchi feeling" and "this is a screensaver." The no-gamification rules, no streaks, sparse notebook cadence, and aviary-age pacing all protect that same intent.

4. **The product notices; it does not announce.** The plan bans a "Welcome back" toast, makes the bird greeting "the entire welcome," makes new-bird offers a "single quiet icon," keeps weather subordinate, and asks in review "does this announce or notice?" The voice is quiet, ambient, and matter-of-fact rather than engagement-oriented.

5. **Personality is hidden, expressive, and never exposed as numbers.** The bird has scalar traits, but the plan repeatedly says there are "No personality numbers exposed to the user." The API returns `plumage_level`, not `plumage_saturation`, because "the user never sees the personality scalar, even at the network boundary."

6. **Accessibility is part of the aviary, not a fallback.** Accessibility surfaces are "designed for charm, not parity-by-checklist." Screen-reader narration uses the same naturalist voice as the notebook; reduced-motion is a "parallel render path" and "not a fallback"; captions come from the call grammar.

7. **Audio must be procedural and socially coupled.** The "procedural-call rule is non-negotiable." The chorus is a real product property: birds calling together are "real interaction, not stacked loops," and recorded fallback is rejected because "canned audio kills the spell."

8. **Privacy and anti-social-network constraints are structural.** The plan enforces "No social network surfaces" through missing data paths: no cross-account aggregation, no public feeds, no engagement telemetry, encrypted visitor email, opt-in visits, and telemetry that separates "operational health" from "user behavior aggregation."

9. **Boring infrastructure protects felt coherence.** The plan prefers Postgres, HTTP polling, REST, and no Redis/Kafka/WebSocket at v1 because extra infrastructure adds stale state, server-state, and failure modes that "hurt felt-coherence" or "silently corrupt the personality vector."

10. **Performance is part of felt aliveness.** The plan says performance is "the bridge between an engineering metric and a felt-aliveness property." First-bird-visible, 60fps idle motion, bundle size, and no-memory-growth are runtime invariants because above budget "the user notices the load."

11. **A bird's identity and relationship history are durable.** Stable bird IDs, monotonic personality, `personality_history`, migration policy, and the cap of seven protect the "per-bird relationship" from silent reset, deletion-and-recreation, or recognizability collapse.

## Per-feature whys

### Scope

- **Single-user accounts with email magic-link sign-in**: NOT RECOVERABLE FROM PLAN.
- **Per-account synthetic UUID**: The synthetic `id` is "the only identifier used in any other table, log line, telemetry record, or partition key," which prevents email from becoming an operational identifier and reduces PII leakage.
- **Per-device session tokens, revocable**: The plan uses short-lived JWTs plus opaque refresh tokens because this is a "standard pattern" and keeps the revocation list small; sessions are user-visible and revocable from settings.
- **Email change with verify-then-switch**: The new address is verified through `email_change_pending` before `email_enc` is swapped, so the account record is not changed until the new email is confirmed.
- **Single canonical aviary per account**: This makes multi-device sync "a property of the architecture" and removes client merge logic; both devices read the same canonical record.
- **Two starter birds at adoption**: NOT RECOVERABLE FROM PLAN.
- **Cap at seven birds**: The rationale is recognizability: raising the cap risks that "per-bird recognizability collapses" and "the per-bird relationship the product is selling thins." The cap is a database constraint, not a config preference.
- **Third-and-beyond gated by aviary age**: The plan chooses weeks-long thresholds because "Months not days" and the pacing "refuses gamification pacing"; the relationship should deepen over time.
- **Hidden personality vector**: The vector drives behavior, mood, calls, and render hints, but the plan forbids stats panels or debug views because the user should see expression, not numbers.
- **Mood enum and mood persistence**: Mood is the "fast-timescale state" that may move freely while personality is monotonic. Persisting it lets the canonical aviary continue across sessions rather than resetting on load.
- **Server-side simulation tick**: The tick advances canonical state regardless of client connectivity; a slow tick is "part of the calibration shape" and avoids over-reacting to every event.
- **Monotonic personality drift**: The drift function protects against "negative drift on neglect" and balances two failures: too fast feels like "my bird changed because I clicked," too slow feels like "nothing I do matters."
- **Procedural call synthesis**: The plan rejects recorded audio because "canned audio kills the spell." Procedural motifs with jitter avoid loop-feeling while allowing mood/personality shaping.
- **Real chorus mechanic**: Multiple birds calling produce physically summed audio and bird-to-bird coupling, so the aviary feels like "a small social system rather than parallel NPCs."
- **Single horizontal scene with three perch zones and no panning or scrolling**: NOT RECOVERABLE FROM PLAN.
- **Day/night and ambient weather**: The server-driven day state and weather make the aviary continue quietly over time. Weather is meant to be "quiet ambient," not something the user looks for.
- **Top bar with account, accessibility, notebook, and offer affordances**: It fades after stillness so controls remain available without making the surface announce itself; CSS opacity keeps elements focusable.
- **Return-greeting anchor moment**: The bird greeting is "the entire welcome"; this replaces textual welcome copy and keeps the return moment in the aviary rather than in UI chrome.
- **Listen-in as slow audio re-balance**: The 1500ms ramp is "the listening tempo." Other birds never go silent because the user should still hear "the aviary, just attenuated."
- **Offer flow: seed, song fragment, still pool of water**: Offers create specific attention signals shaped by mood/personality. Seed nudges curiosity/boldness, song nudges vocal behavior, and pool touches curiosity/plumage as "a specific charm."
- **Settle soft session-end gesture**: The plan connects settle to the `settled` mood, the night/end-of-session state, and narration that describes "the aviary going quiet."
- **Five-second undo for settle**: NOT RECOVERABLE FROM PLAN.
- **Field notebook**: The notebook is sparse so it does not become a feed. It observes the aviary, "never" the user, and uses naturalist prose instead of engagement language.
- **Visits as per-invite read-only ambient views**: The feature is self-limiting by design: opt-in invitations, revocable links, no co-presence, no chat, no avatars, and no default notifications prevent social-network creep.
- **Visit log in account settings**: The host sees the invite email because "it's their invite," and the log supports revocation and historical visibility without creating a public social surface.
- **Multi-device sync**: Sync is not a separate feature; it follows from canonical server state, snapshot pulls, and no client merge logic.
- **Account export**: The plan records notebook slots/prose and personality history for export, but the user-facing rationale for having account export is NOT RECOVERABLE FROM PLAN.
- **Account deletion: soft for 30 days, then hard**: The 30-day soft window makes deletion restorable; the hard cascade is the privacy commitment that deletion is total.
- **Browser support: last two major versions of Chrome, Safari, Firefox, Edge**: NOT RECOVERABLE FROM PLAN.

### Architecture

- **Three server-side components: API tier, simulation worker pool, Postgres**: "The simplicity is part of the design" because fewer moving parts mean fewer ways to "silently corrupt the personality vector."
- **No Redis, Kafka, separate cache, or separate analytics store at v1**: These are avoided because they add stale snapshots, server-state, or failure modes that do not carry their weight at v1.
- **TypeScript on Node for the server**: TypeScript is chosen over Go for iteration speed and shared personality math between the test harness and production.
- **TypeScript, Preact, and signals on the client**: Preact serves bundle size; signals fit the small reactive graph for the top bar and notebook.
- **Canvas 2D rendering**: The one-screen scene, seven-bird cap, and low draw-call count make Canvas 2D enough for 60fps, and it is easier to keep accessible than a premature WebGL path.
- **Hand-rolled WebAudio synthesis**: The team avoids a third-party library because it has to "own the chorus mix."
- **Postgres 16 as the database**: Numeric columns store live trait values; JSONB supports inspectable history payloads; Postgres is enough at v1 volume.
- **JWT plus opaque refresh token auth**: The pattern provides short-lived access, refresh rotation, and server revocation.
- **Transactional email only**: Email is limited to magic links, visit invitations, and exports; the plan explicitly says "No marketing emails ever."
- **CloudFront plus embedded initial state snapshot**: Delivering HTML, bundle preload, and initial snapshot from the edge minimizes round trips for time to first bird.
- **OpenTelemetry, Prometheus, and structured logs**: Observability exists for operational health, with per-account fields stripped at the SDK layer.
- **CI/CD performance gates**: Bundle size, first-bird-render, and no-memory-growth are gates "not warnings" because they are runtime invariants.
- **No WebSocket for sync**: Snapshots are cheap and 30s polling plus visibility-change pulls provide enough felt coherence; WebSockets add server-state and reconnection complexity.
- **SSE only for visit revocation**: Revocation needs one-way, idempotent push, not bidirectional state.
- **No Kafka for event log**: The event rate fits Postgres, and Kafka adds failure modes that hurt felt coherence.
- **No Redis snapshot cache**: Regenerating snapshots is cheap, and Redis would add stale-snapshot bugs.
- **No LLM for notebook prose at v1**: A controlled grammar is deterministic, cheap, voice-consistent, and avoids moderation and latency concerns.

### Data Model

- **`account` table with encrypted email and email hash**: Email is sealed and only read at sign-in/export; the hash uses a server pepper so a database leak does not expose the email set.
- **`session` table**: Per-device sessions are revocable and visible in settings through a best-effort device label.
- **`bird` table with stable `id` and renameable `display_name`**: The bird's UUID is stable across renames, sync conflicts, and migrations; name is user-controlled, personality is server-authored.
- **`personality_history` table**: It lets the test harness verify calibration, supports export, and acts as an audit for discontinuous personality movement; it is never shown as user-facing numbers.
- **`interaction_event` append-only table**: Insert-only events plus `(account_id, client_id)` idempotency make retries safe and preserve deterministic simulation order.
- **`tick_log` table**: The tick records its cursor so reruns are idempotent and worker crashes do not double-count events.
- **`notebook_entry` canonical prose**: Storing final `prose` means old entries read exactly as written even if the grammar evolves.
- **`visit` table**: Visitor email is encrypted at rest, while aggregate `visit_count` and `visit_seconds` support the host's visit log without per-second behavior detail.
- **`account_export` table**: The download URL is short-lived and expires in 24 hours.
- **Soft delete and hard delete cascade**: All related tables cascade on account hard-deletion because the privacy commitment is "deletion is total."

### API Surface

- **HTTP+JSON REST instead of GraphQL**: The endpoint count is low enough that GraphQL's flexibility does not justify the cost of a typed schema layer.
- **`POST /v1/auth/request-link` returning 204 always**: This prevents account enumeration.
- **`POST /v1/auth/consume-link` single-use tokens**: Token consumption invalidates the token so a magic link cannot be replayed.
- **`POST /v1/auth/refresh` rotation**: Refresh rotation invalidates the old token, supporting revocation and reducing token reuse risk.
- **`GET /v1/aviary` snapshot without personality numbers**: The snapshot includes render hints and `plumage_level`, not trait scalars, so the user never sees personality even at the network boundary.
- **Snapshot `call_schedule`**: A forward window lets the client synthesize calls without round-tripping for each call and keeps two devices hearing the same canonical calls.
- **`POST /v1/aviary/events`**: The client submits append-only interaction events and polls for updated render; the endpoint never returns personality state.
- **Notebook pagination**: Cursor pagination supports infinite scrollback without turning the notebook into a feed surface.
- **Bird rename endpoint**: `display_name` is writable because name is user-controlled; personality fields are not.
- **Narration endpoint**: It gives screen readers a naturalist-prose rendering of the same state rather than a separate accessibility-only state.
- **Visit invitation endpoints**: Invites are tokenized, read-only, revocable, and logged for the host without making visits public.
- **Internal worker endpoints**: They are internal VPC plus mTLS only, preserving the worker-only authority for ticks, notebook generation, and narration generation.
- **Matter-of-fact error mapping**: User-facing errors use calm strings like "This visit is no longer available," matching the plan's product voice.
- **Rate limits**: Auth and event rate limits prevent enumeration, abuse, and unrealistic event rates while keeping normal interaction well below the cap.

### Simulation Engine

- **Row-lock per account tick**: `SELECT ... FOR UPDATE SKIP LOCKED` ensures only one worker ticks an account at a time without blocking the pool.
- **Tick reads events, state, computes deltas, writes in one transaction**: This keeps personality, mood, perch, history, and schedule changes consistent.
- **Postgres `NOTIFY` on event arrival**: Interactive moments do not wait for the next round-robin slot.
- **Idle tick at least every 5 minutes**: Background drift of day/night, dusk mood, and nightjar calls continue even when no client is connected.
- **EMA personality drift**: EMA is "the simplest filter that gives the calibration shape" and supports slow, monotonic expressive change.
- **Trait-specific event weights**: Listen-in, seed, song, and pool affect different traits so interactions have character-specific consequences.
- **No drift on `settle_start`**: Settle is mood-only, preventing a session-end gesture from becoming a personality optimization action.
- **History threshold of `0.0001`**: Tiny changes are not written, preventing history-table bloat for mostly idle accounts.
- **Drift calibration harness**: CI asserts the 7-day and 21-day bands so a casual engine change cannot silently alter the felt relationship.
- **Neglect-case harness**: Zero events must produce zero personality delta, catching any regression toward Tamagotchi neglect mechanics.
- **Mood state machine**: Mood responds to recent events, local time, weather, and personality while avoiding flicker through long average dwell times.
- **`settled` absorbing night state**: It supports the night/end-of-session behavior and wakes only through sunrise plus a personality-shaped roll.
- **Perch assignment**: Perch location makes mood and personality visible: bold/content/curious birds trend front; wary/drowsy birds trend back.
- **Notebook cadence and suppression**: Rate limits and non-deferred suppression keep the notebook sparse and prevent noise.
- **Notebook grammar templates**: A grammar over named slots gives deterministic naturalist prose and preserves voice.
- **Notebook banned phrases**: Templates cannot contain second person, streak language, or visit-frequency references because the notebook observes the aviary, not the user.
- **Call grammar runtime**: The server schedules canonical calls; the client varies synthesis parameters but never invents calls outside the schedule.
- **Bird-to-bird interaction**: Mood spread, response calls, and chorus detection make the aviary a coupled social system, not independent birds.
- **New-bird offers**: Eligibility depends on aviary age and count; the affordance "never pings, never emails, never highlights," keeping adoption quiet.
- **Tick performance budget**: p50/p99 wall-time and batched database writes ensure the worker pool can maintain cadence with headroom.

### Sync Model

- **Database single-writer privileges**: If the API tries to write personality, it fails at the database, "not at code review."
- **Snapshot-pull triggers**: Initial load, visibility changes, keepalives, suspend/resume, and post-event pulls keep clients coherent cheaply.
- **No client merge logic**: There is no "synced from another device" surface because both devices read from the same canonical record.
- **Server-time event ordering**: The API's `now()` orders events across devices, and `id` breaks true simultaneity deterministically.
- **Last-write-wins only for `display_name`**: This is acceptable because name is user-controlled and the worst case is a short stale name.
- **Visit revocation propagation**: SSE closes active visits within seconds; polling handles the fallback within about 30 seconds.

### Frontend Rendering Pipeline

- **Boot directly into the aviary scene**: There is no spinner, fade-in, or entry animation because the first thing the user sees should be the aviary.
- **Inline static first frame**: The static frame is "not a load state"; it is the aviary, letting first bird be visible before the main bundle arrives.
- **Calm-field slow path**: On slow connections the user sees an aviary that is not animating yet, never a loading spinner.
- **Single `requestAnimationFrame` render loop**: The loop batches birds, ambient ornaments, weather, and layers into one frame budget with headroom.
- **Bird animation state machines**: Mood and personality tune timing so two birds in the same state still look different.
- **Reduced-motion rendering**: It is a designed parallel path with cross-fades and still poses, so the preference does not cost the user the product.
- **Top bar fade behavior**: Opacity fade makes the controls quiet while keeping them focusable by Tab.
- **Accessibility tree for the canvas**: The canvas has a prose summary and live narration instead of exposing each bird as a list-of-things UI.
- **Sprite atlas asset pipeline**: A single hashed atlas with metadata keeps assets cacheable and within the 600KB budget.
- **Quantized plumage levels in assets**: Visual richness is mapped from the hidden scalar without exposing the scalar or multiplying assets fivefold.
- **Gradual day/night rendering**: Continuous interpolation means a user checking at dusk sees a slight hue shift rather than an event.
- **Weather particle overlay**: Weather fades in/out from server times, keeping it ambient and smooth.
- **Empty-aviary state**: It exists only between adoption and first bird arrival; after that the scene is never empty because bird deletion is not a product action.
- **Ambient ornaments generated client-side**: Leaves, feathers, and motes never leave the client and are skipped in reduced-motion mode.

### Audio Pipeline

- **Single shared `AudioContext`**: It centralizes per-bird buses, ambient bed, listen-in mix, chorus, and master limiting.
- **Per-bird gain and pan buses**: These make listen-in, stereo placement, and overlapping motifs controllable per bird.
- **Procedural motif programs**: Oscillators, filters, envelopes, and jitter make calls variable, mood-shaped, and cheap.
- **Species motif libraries**: Six to ten motifs per species support recognizability without recorded loops.
- **Physically summed chorus**: Simultaneous synth voices interact at the master bus in a way "two recorded loops layered cannot fake."
- **Gentle master limiter**: It prevents clipping when many birds call without audible pumping.
- **Listen-in gain ramp**: The 1500ms ramp avoids both stickiness and switch-like cuts.
- **Other birds audible during listen-in**: The 0.18 floor preserves the aviary rather than muting everything except the target bird.
- **Per-call WebAudio scheduling**: Calls are ambient; if the tab is hidden and a call is missed, nothing depends on it.
- **WebAudio unavailable fallback**: The plan chooses graceful silence plus captions because that is better than a recorded audio path.
- **Audio node lifetime tracking**: Disconnecting nodes and CI heap tests prevent long-session memory growth.
- **Pause when tab hidden**: Hidden tabs suspend audio/rendering and stop presence pings because the user is not present.
- **Resume from a fresh snapshot**: On visibility return, the client catches up to the server-side simulation rather than showing a frozen aviary.

### Accessibility

- **Screen-reader narration**: Naturalist prose generated from the same state gives screen-reader users an aviary that feels alive.
- **Narration cadence**: Idle narration updates only when state changes enough to describe, avoiding chatter.
- **User-event narration priority**: Return-greeting, offer, settle, and listen-in get faster narration so user actions receive timely feedback.
- **Second person only for listen-in narration**: The plan allows "you're listening" there because it describes an action the user just took; the rest stays observational.
- **Call captions**: Captions are generated from motif shape, so they describe the actual procedural call rather than generic sound.
- **Maximum caption count and chorus collapse**: This prevents visual caption clutter during multi-bird calling.
- **Keyboard navigation**: Keyboard users can reach top bar actions, cycle birds, listen in, offer, settle, and use the notebook.
- **Visible focus indicators**: Dual outlines are required because the aviary background changes across morning, dusk, and night.
- **Reduced-motion independent from narration**: Visual motion settings do not change narration, so users can combine preferences without losing surface behavior.
- **Forced captions when audio fails**: The system overrides caption preference because silence with no captions is the worse failure.
- **ARIA structure**: The canvas `aria-label` and polite live region keep a current scene description available even if one mechanism is missed.
- **WCAG AA contrast**: Contrast is enforced for top bar, captions, settings, errors, and account surfaces across day states.
- **English-only v1 for generated naturalist prose**: Matter-of-fact strings are externalized, but generated prose requires locale-specific grammar reauthoring, making it a v2 conversation.
- **Accessibility review process**: Manual NVDA/VoiceOver passes and a paid advisory group catch failures the team would otherwise miss.

### Performance and Observability

- **Initial JS bundle cap**: The 2MB cap discourages drift and leaves headroom for v1.5 features.
- **Lazy-loaded notebook, visits, and account surfaces**: They keep the critical bundle small so first bird is not delayed.
- **First-bird-visible under 500ms**: The plan treats this as felt aliveness; static-first-frame is the trick that makes the budget achievable.
- **60fps idle motion**: Frame budgets keep motion smooth on a five-year-old laptop.
- **No memory growth over 30 minutes**: Memory leaks would be "a real product failure at v1 scale," so CI hard-gates them.
- **Telemetry of operational health**: Request latency, tick latency, snapshot latency, frame histograms, audio errors, and sign-in rates are measured to keep the affective surface working.
- **No per-account or engagement telemetry**: The plan does not measure per-account feature use, retention, or behavior aggregation because that crosses from operational health into user behavior aggregation.
- **Separate observability database with no simulation DB path**: Network policy prevents analytics queries from reading the simulation database.
- **Synthetic monitoring**: Automated Playwright sessions exercise sign-in, first-bird-render, event submission, snapshots, and visit paths from common geographies.
- **Synthetic accounts excluded from aggregates**: Test accounts are "not the user we are designing for."
- **Structured logs without PII**: Logs omit email, names, and prose; retention is limited because logs are operational, not user-facing.
- **SLOs for API, ticks, and first-bird-render**: These targets make user-visible availability and aliveness operationally actionable.

### Rollout, Flags, and Migration

- **Milestone order**: The rollout builds from foundations to one bird, two birds and chorus, drift, narration, ambient rendering, species/adoption, visits, account features, beta, then GA so the product's riskiest aliveness properties are calibrated before launch.
- **Closed beta calibration**: M9 tunes drift, narration cadence, listen-in ramp, and notebook cadence with real feedback and accessibility audits.
- **Daily-active cap at GA**: Registration is capped until tick latency stays within budget as the worker pool scales.
- **Day-one observability dashboards**: They make first-bird-render, tick latency, audio init, sign-in, and the "feels-alive" composite visible from launch.
- **Operational feature flags**: Flags tune cadence, presence window, narration timing, notebook cadence, listen-in ramp, species, and new-bird thresholds without deploys.
- **No product on/off feature flags for readiness**: The plan says the flags are operational only; "we don't ship features behind flags" if the feature is not ready.
- **Forward-only migrations**: Schema changes are new migration files, never in-place edits, reducing risk to shipped state.
- **Personality history migration rule**: It is never deleted except account hard-deletion because "the user's drift history is the user's relationship with their birds."
- **Four-step `bird` table migrations**: Add, backfill, switch reads, switch writes, then drop later protects bird identity and personality through schema changes.
