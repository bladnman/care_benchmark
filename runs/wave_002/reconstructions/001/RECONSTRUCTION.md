## System-level intent

- **Server-authoritative simulation with a thin client.** This is the plan's spine in the executive summary: a "server-authoritative simulation" and a "thin, stateless client." It appears again in the load-bearing invariants ("The server is the only writer of canonical state; clients only append events and render snapshots"), in the client/server split ("Server owns: everything canonical"), and in sync ("One record, many readers").

- **An ambient aviary that is already alive, not an app that announces itself.** The plan describes "a single horizontal scene that is always already in motion," requires the "first frame has motion already in progress," and says the quiet field is "the only load state that exists." This same intent appears in the ban on "toasts, banners, badges" and in arrivals being "noticed, not announced."

- **Notice-never-announce product manners.** The plan repeatedly rejects announcement surfaces: no "welcome back," no badges or unread indicators, no notebook icon badge, no re-invitation, no frequent-visitor status, and no host notification of revocation success. Where product growth appears, the plan applies "notice-never-announce" through quiet top-bar affordances, ambient silhouettes, and matter-of-fact system surfaces.

- **Slow, honest, non-punitive expressiveness.** The executive summary says personality "drifts slowly and monotonically toward expressive on honest presence signals." The drift section says single sessions must not visibly move traits, absence causes "zero negative movement," and "the birds get quieter, not warier." The non-goals also reject Tamagotchi mechanics: "no death, no hunger, no distress, no decaying happiness meter."

- **Presence is attention, not page-open time.** The invariant names the "honest triple condition (visible ∧ focused ∧ recent pointer/key activity)." The presence section says watching without moving is part of the product, but "tab open all night" must be zero presence. Cross-device union-dedupe also says "presence measures the user's attention, not their screens."

- **Bird identity and recognizability are permanent.** The invariant says "Bird identity is permanent," and the data model makes `birds.id`, `birds.signature`, and adoption identity immutable. Audio repeats the same intent: signature params are "for life," so a bird is "recognizable by ear across moods and drift."

- **Procedural, varied, deterministic life.** Calls are "procedurally synthesized client-side," all audio is procedural, greetings are "never a canned cue," and chronicler entries avoid identical repeats. At the same time, randomness is deterministic from `(aviary_id, time-bucket, purpose)` seeds so ticks are "catch-up-safe."

- **One naturalist voice, separated from system copy.** The executive summary says the field notebook and screen-reader narration come from "one shared naturalist-voice module." The chronicler/narrator section defines "lowercase, present tense, bird-named, specific, no 'you', no exclamations, no numbers, no achievement phrasing." Appendix C keeps system surfaces "matter-of-fact" and says the two catalogs "never mix registers."

- **Accessibility is a designed surface, not a fallback.** Scope calls accessibility "designed surfaces"; the reduced-motion renderer is "a designed surface, not a fallback"; and the ship-together rule says accessibility ships "with v1, not after." Narration, captions, keyboard parity, contrast, and reduced motion are exit criteria.

- **Privacy boundaries are structural, not policy.** The plan says telemetry never has per-account or per-bird dimensions, telemetry stores are physically separate, and the "Privacy boundary" is "architectural, not policy." Personality vectors do not leave the server raw except account export, and email appears only in tightly named encrypted/blind-index columns.

- **Performance and calibration are gates, not aspirations.** The plan uses "CI gates, not aspirations," gives hard budgets for first bird, bundle, frame timing, memory, payload, and tick latency, and makes the calibration harness "the source of truth" for drift targets. The risk table says the drift band is "the product."

- **Visits are the entire social feature.** The scope and visits sections repeat that visits are opt-in, read-only, revocable, and "the entire social feature." The plan excludes profiles, follows, feeds, discovery, comments, co-presence, leaderboards, show-off rendering, and visitor influence on host drift.

## Per-feature whys

### 1. Executive summary and scope

- **Browser-only ambient virtual aviary:** The plan frames the product as a browser-only "ambient virtual aviary" and explicitly says native-client constraints are out of scope; the data model and protocols are "not designed around native-client constraints."

- **Two starter birds:** NOT RECOVERABLE FROM PLAN

- **Hard cap of seven birds:** The cap is tied to empirical audio recognizability: the birds-per-aviary ramp says slot N unlocks only when the recognizability discrimination test passes at N signatures, and the 7-bird cap can effectively hold at 6 if the test fails.

- **New-bird availability gated on aviary age:** The plan says slots open by aviary age "never on engagement or payment," aligning the feature with the no-gamification and no-payment boundaries.

- **One canonical aviary per account:** The rationale is sync correctness: "one canonical record," "no client-side state to merge," and multi-device sync as "a property of the architecture, not a feature."

- **Single-user accounts:** NOT RECOVERABLE FROM PLAN

- **Email magic-link sign-in:** The plan chooses "no passwords, no SSO" and later specifies identical response copy regardless of account existence, atomic single-use consumption, and rate limiting to avoid account enumeration and keep the flow matter-of-fact.

- **Per-device revocable sessions:** The rationale is user account control across devices: sessions have device labels, list + revoke surfaces, and revocation is checked per API call.

- **Email change with verification:** The new address is verified before commit and the old email remains valid until then, preserving account access while the change is unfinished.

- **Server-side simulation tick:** The tick runs "with or without connected clients" so the aviary continues living whether anyone is watching, and canonical state is advanced by the server rather than client defaults.

- **Client snapshot pulling with interpolation:** The client renders snapshots and interpolates activity phases so both current motion and multi-device continuity come from canonical state.

- **Append-only interaction event log:** Clients send observations, not state values, making the laptop-morning/phone-lunch overwrite scenario "unreachable" and avoiding last-write-wins.

- **Return-greeting:** The greeting is absence-shaped and procedurally varied so it is not a canned cue; it is rendered as an ordinary plan overlay and "No text accompanies it, ever."

- **Listen-in:** The feature is "re-balance, not mute" and "listening, not channel-switching"; other birds never go silent, preserving the ambient chorus.

- **Offers:** The plan gives offers a synchronous reaction path because a "~1-minute tick cannot drive in-session reactions." The reactions are ephemeral directives, while mood and drift are ratified by the tick.

- **Settle:** Settle is a deliberate end-of-attention gesture that "ends the presence window cleanly"; it suppresses presence until click/keypress re-engagement and is not treated as punishment.

- **Field notebook:** The notebook observes "the aviary, never the user's visit behavior." Its rarity governor keeps it sparse so it is "not a feed," and banned content prevents user-behavior, numeric traits, and achievement framing.

- **Single horizontal screen:** The scene is "exactly one screen" with "No panning, no zoom, no scroll," supporting the ambient single-field experience and guaranteeing birds are not cropped or offscreen.

- **Rare ambient weather, leaf/feather drift, and parallax:** NOT RECOVERABLE FROM PLAN

- **No in-scene UI chrome and fading top bar:** The rationale is ambient restraint: scene text is absent, controls live in top-bar DOM chrome, and the top bar fades on cursor stillness while remaining keyboard-focusable.

- **Visits:** Visits are the whole social surface, constrained to host-invited, opt-in, read-only ambient view so there is no social-network or show-off rendering.

- **Visit notifications off by default:** The plan keeps visit notifications off by default and away from onboarding to avoid re-engagement and visit-frequency framing.

- **Accessibility surfaces in v1:** Narration, captions, reduced motion, keyboard navigation, and contrast are exit criteria so accessibility is not retrofitted in "v1.1."

- **Performance budgets:** The plan makes first bird, frame rate, memory, bundle size, and payload budgets CI gates because the product depends on an ambient scene that appears alive quickly and stays stable.

- **Account export:** Export is user-pulled, emailed as a 24-hour link, and is the sole sanctioned numeric exposure of personality vectors.

- **Soft-then-hard deletion:** The quiet 30-day soft-delete window permits "I changed my mind," while the hard purge plus backup aging completes erasure.

- **Aggregate-only operational telemetry:** The rationale is operational health without product analytics: allowed telemetry is aggregate, while per-account, per-bird, funnels, streaks, and behavior-keyed cohorts are deliberately absent.

### 2. Architecture and data model

- **`sim-core` shared TypeScript library:** Behavior rules live "exactly once" and run identically in API, worker, and calibration harness, making divergence structurally impossible.

- **API service:** NOT RECOVERABLE FROM PLAN

- **Sim worker:** The worker owns tick scheduling/execution and pushes post-tick snapshots, supporting the server-authoritative simulation and edge snapshot path.

- **Web client:** The client is optimized for first-bird under 500 ms and keeps rendering separate from canonical decision-making.

- **Audio engine:** The engine exists to synthesize multi-voice procedural calls, mix chorus/listen-in, and generate captions from call specs.

- **Accessibility layer:** The layer packages narration, captions, focus, keyboard, and reduced-motion as first-class surfaces rather than fallbacks.

- **Accounts and privacy infrastructure:** The rationale is the "hard telemetry boundary," encrypted email, blind-index lookup, sessions, export, and deletion.

- **Observability and calibration:** The synthetic fleet, RUM, budget gates, and accelerated-clock harness exist to own performance and drift targets.

- **TypeScript everywhere:** One language and one package make behavior-rule divergence "structurally impossible."

- **Node.js 22 + Fastify:** The workload is "I/O-bound and small-compute"; the plan says there is "no need for a second backend language."

- **Postgres 16 as the only datastore:** Scale math keeps events/ticks within Postgres capacity, and an append-only table with partitioning gives ordering guarantees "with far less machinery" than a streaming platform.

- **Redis due-tick ZSET, rate limits, and mailer queue:** Redis stores no durable state; loss "degrades cadence, never correctness" because ticks are catch-up-safe.

- **Cloudflare Worker and KV edge snapshot delivery:** This exists to hit the 500 ms first-bird budget by inlining the latest snapshot into HTML with "No origin round-trip on the critical path."

- **React DOM chrome plus custom canvas-2D scene renderer:** React is limited to chrome/routes and the render loop "never touches React state"; the canvas decision gives "one compositor" for cheap 60 fps while DOM keeps text crisp and focus native.

- **AudioWorklet synth:** The worklet synthesizes sample-accurately off the main thread and avoids per-call AudioNode churn.

- **Immutable-versioned static client assets and N-1 API compatibility:** These choices support instant rollback and deployed-client compatibility.

- **Accelerated-clock staging and injected `Clock`:** The `Clock` interface lets the calibration harness run accelerated time against the real engine.

- **Monorepo package layout:** The packages mirror ownership boundaries: `sim-core` for all behavior rules, `voice` for naturalist prose, render/audio/presence/config/telemetry/db packages for reusable shared surfaces.

- **Server/client boundary rule:** The server decides "what a bird does"; the client decides how the scheduled thing "looks and sounds right now."

- **Client idle filler exception:** The exception exists for plan-horizon lapses; filler is restricted to low-commitment visuals, never canonical, and overridden by the next plan.

- **Synthetic UUIDs and encrypted/blind-indexed email:** The rationale is PII containment: email is nowhere in logs, events, metrics, queue payloads, or error messages.

- **Stored personality vectors:** Vectors are stored and never recomputed from event logs because raw history ages out and events feed only deltas.

- **Immutable bird IDs and call signatures:** Permanent identity protects continuity and recognizability across renames, species-pool changes, mood, and drift.

- **Personality writes only through tick-context `applyDriftDelta`:** The single guarded method enforces monotonic drift and server-only canonical mutation.

- **Raw event retention folded into rollups after 90 days:** Drift never needs raw history because vectors are persisted, so raw events can age out while notebook entries persist.

- **`ticks.drift_applied`:** The field is for "calibration harness only" and canary/audit use; it is never exposed through the API.

- **Read-only notebook entries:** Entries are "Read-only forever"; the only delete path is account purge, preserving the notebook as an observation record rather than an editable feed.

- **Audited config table:** Tunables are audited, and drift-gain changes require the calibration harness to pass before production apply.

- **Versioned static config for species, song fragments, and copy catalogs:** Static config is versioned in repo and linted so behavior/audio/copy stay reviewable and testable.

- **Backups with PITR and daily snapshots retained no more than 35 days:** This keeps hard deletion complete within the 30-day soft-delete plus 5-day backup-aging window.

### 3. API surface

- **REST/JSON under `/v1` with additive snapshot versioning:** The snapshot `v` field and unknown-field tolerance let clients survive additive evolution.

- **httpOnly Secure SameSite=Lax JWT session cookie:** The cookie can be verified at the edge for snapshot inlining, while the API additionally checks revocation.

- **Visitor scoped read-only token:** The visitor token constrains visits to read-only snapshot access.

- **Magic-link request endpoint:** It always returns the same matter-of-fact success copy to prevent account enumeration.

- **Magic-link verification endpoint:** Atomic single-use consumption and 15-minute expiry enforce link safety.

- **Snapshot endpoint with `ctx=open|keepalive|resume`:** `ctx=open` runs greeting resolution and timezone update; ETag/state_rev enables cheap 304s when unchanged.

- **Event batch endpoint:** It accepts append-only events, dedupes retries, and returns offer reactions synchronously because offer feedback cannot wait for the next tick.

- **Notebook pagination:** Newest-first pagination with `before` supports indefinite scrollback.

- **Bird rename validation:** NOT RECOVERABLE FROM PLAN

- **Bird adoption endpoint:** The server verifies the age gate so slot availability cannot be driven by client state or engagement.

- **Account settings endpoint:** It centralizes accessibility preferences, visit-notification toggle, and timezone in canonical account state.

- **Account export endpoints:** Exports are generated on demand and downloaded through a short-lived link, matching the user-pulled numeric-exposure exception.

- **Account delete/restore endpoints:** They implement soft delete and the "I changed my mind" recovery window before hard purge.

- **Visit invitation endpoints:** They support host-created invite/redeem/revoke/log flows, keeping social interaction on demand and revocable.

- **Visitor snapshot endpoint:** It omits greeting and narration priority events and never records visitor presence so visitors "trigger nothing."

- **Unsupported browser surface:** NOT RECOVERABLE FROM PLAN

- **No endpoint accepting personality, mood, position, or notebook writes:** This enforces server-only canonical state and raw-personality secrecy.

- **No unread or badge endpoint:** This enforces the no-announcement-surface invariant.

- **Client-generated idempotency keys:** Retries and duplicate sends become no-ops by dedupe on `(aviary_id, idem_key)`.

- **Silent in-product event rejections:** Rejections produce codes for logs/tests only; no rejection renders a toast, and the aviary keeps living.

### 4. Simulation engine design

- **Redis ZSET tick scheduler with sharded batches:** Batching and sharding avoid contention while workers process due aviaries and reinsert next due times.

- **Active and dormant cadence tiers:** Dormant aviaries tick less often for throughput, while catch-up equivalence keeps state the same within epsilon.

- **Pure-ish `runTick` pipeline:** The pipeline gathers canonical state/events, computes ambient state, mood, drift, plans, chronicler/narrator output, persists changes, bumps `state_rev`, and pushes a snapshot so the tick is the canonical change point.

- **Throughput design target:** The 100k-account math and 2x headroom are included to justify the Postgres/worker architecture before GA.

- **Daily drift accumulators:** Presence, listen-in, accepted offers, and near offers feed capped daily signals so drift is based on honest bounded signals.

- **Low-pass EMA and hourly micro-steps:** The EMA keeps drift slow; hourly micro-steps avoid an observable daily boundary discontinuity.

- **Monotonic drift clamps:** Deltas clamp to `[0, remaining headroom]`, so traits move toward expressive and never decay.

- **Settle excluded from drift:** The plan says settle contributes nothing beyond ending the presence window cleanly, preserving non-punitive behavior.

- **Calibration targets:** The harness owns the 1-week, 3-week, single-session, and absent-persona targets because the plan treats drift pace as the narrow product band.

- **Expressiveness recency multiplier:** Greeting probability, call rate, and front-perch bias can quiet after absence without touching traits; "the birds get quieter, not warier."

- **Mood state set:** The finalized moods include roosted to give night a rendered posture, with the nightjar exemption for nocturnal behavior.

- **Mood weighted transitions:** Time of day, weather, interactions, bird-to-bird effects, and personality make mood an ambient social system rather than a fixed label.

- **Minimum mood dwell:** The dwell prevents mood flicker.

- **Mood persistence:** Mood and `mood_since` are canonical so returning users do not see a reset-to-neutral client default.

- **Behavior planner horizon:** Server-scheduled 2-5 minute plans ensure birds are never plan-idle and the client does not decide behavior.

- **Explicit `fly_to` and `hop_to` plan entries:** The client never infers transitions; this prevents teleporting.

- **Mood rendered without scene labels:** Wary, content, curious, and drowsy are expressed through posture/activity, keeping labels out of the scene.

- **Species call grammar and call specs:** The grammar gives procedural variety; the same spec deterministically renders the same call, while different seeds must not render identical calls.

- **Call scheduling with weather, time, mood, personality, and recency factors:** Scheduling makes calls arise from the same ambient state as the birds rather than from static loops.

- **Chorus coupling:** Call-and-response and overlapping calls emerge from bird-to-bird social warmth and are mixed as a real chorus.

- **Song-fragment library of five motifs:** NOT RECOVERABLE FROM PLAN

- **Greeting greeter selection:** Selection weights awake birds by boldness, mood, and social warmth, while rotation prevents one bird from always greeting first and creates real observable greeting-order variation.

- **Greeting form by absence and boldness:** Longer absence and higher boldness change the form, making returns absence-shaped without "welcome back" text.

- **Greeting procedural jitter and stagger:** Jitter prevents canned repeated cues, and staggered second-bird responses avoid unison that would "announce" arrival.

- **Offer-reaction fast path:** Ingest-time reactions exist because a 1-minute tick cannot drive in-session reactions; the same `sim-core` function preserves behavior consistency.

- **Offer reaction writes no personality and no mood at ingest:** This preserves the server-only-writer invariant by emitting ephemeral overlays and letting the next tick ratify mood/drift.

- **Per-bird shared offer cooldown of 180 seconds:** NOT RECOVERABLE FROM PLAN

- **Quiet cooldown affordance with no timer text:** The rationale is no punitive copy and no announcement surface.

- **Still pool lasting three minutes:** NOT RECOVERABLE FROM PLAN

- **Settle undo and re-engagement rules:** The 5-second undo and click/keypress re-engagement keep settle a deliberate, reversible end-of-attention gesture; mere mouse movement does not re-engage.

- **Aviary-local day/night from account timezone:** The user sees the aviary in their local time, and every device still sees one canonical phase at each instant.

- **Timezone update on `ctx=open`:** Last active device's IANA timezone preserves one canonical aviary time while honoring that "the user's local time is where the user currently is."

- **Chronicler salience detectors:** Entries are based on aviary observations such as greeting-order firsts, rain, mood juxtapositions, quiet stretches, offer reactions, co-perching, and arrivals.

- **Chronicler rarity governor:** The gap and weekly cap keep very active aviaries sparse; "the notebook is not a feed."

- **Narrator server-side generation:** Server generation from the shared `voice` module keeps voice consistency.

- **Narrator min/max cadence and priority queue:** Cadence limits stale or flooding narration, while greeting/offer/settle lines can be inserted promptly as observations.

- **Age-gated new-bird arrivals as back-perch silhouettes:** Arrivals are "noticed, not announced"; they do not greet, call loudly, expire, or nag.

- **Species selection weighted to complement current signatures:** This protects recognizability at seven birds.

- **Deterministic RNG from aviary/time/purpose seeds:** Late ticks can recompute the same ambient history as real-time ticks, enabling catch-up equivalence.

### 5. Sync model

- **One record, many readers:** This makes multi-device sync an architecture property with no merge, no conflict resolution, and no eventual consistency.

- **Pull triggers on load, visibility, wake, keepalive, and post-event state_rev:** These keep clients aligned to canonical snapshots while conditional 304s "cost ~nothing."

- **Clients send observations, never values:** This structurally prevents last-write-wins and raw state overwrite bugs.

- **Server-assigned per-aviary sequence:** Ticks consume events strictly in sequence so additive deltas are ordered.

- **Late-event 10-minute window:** Brief offline and clock drift are tolerated, but older data is dropped because "presence honesty beats completeness."

- **Offline queue capped at 200 events/10 minutes:** The cap follows the same honesty-over-completeness rationale.

- **Clock offset EWMA:** Offset correction keeps plan and call phase interpolation aligned with server time while clamping skew.

- **Presence triple condition with 240-second activity window:** This defines honest presence while honoring that "watching without moving is the product."

- **Presence pings every 30 seconds with flush on condition break/pagehide:** This captures covered intervals accurately and closes presence windows promptly.

- **Settle suppresses presence pinging:** Settle and tab-close both end presence cleanly and are "neither penalized nor distinguished by the engine."

- **Union-dedupe across devices/tabs:** Attention is counted once "no matter how many screens show the aviary," preventing multi-device drift inflation.

- **Daily presence sanity clamp and alarm:** The clamp detects client bugs that would silently corrupt drift.

- **Matter-of-fact auth/session/outage surfaces:** Errors do not mention birds and do not block rendering; the aviary keeps living while events retry.

### 6. Frontend rendering pipeline

- **Quiet field boot state:** The soft sky/horizon paints immediately and replaces a spinner; it is the only load state because the aviary should appear alive, not loading.

- **Edge-inlined snapshot on load:** Inlining removes the origin round-trip from the critical path for first bird.

- **Critical JS chunk with renderer core, one rig, and audio stub:** This keeps the first-bird path small enough for the 500 ms budget.

- **Initial birds painted mid-activity:** The first pose is computed from plan timestamps and clock offset so the aviary appears as if it "has been rendering all along."

- **First-ever post-signup fly-in:** This is the sole entry animation and happens once per account, preserving the no-entry-animation rule after creation.

- **Layer stack with one canvas and DOM captions/focus/chrome:** Canvas gives one compositor for scene layers; DOM gives crisp text and native focus/ARIA.

- **Procedural rig-based bird rendering:** Rigs and render params express species, mood, and personality-derived values without shipping raw traits.

- **Responsive scene metrics with no crop/offscreen:** The scene remains one screen across viewport sizes, with no panning, zooming, or scroll.

- **Cross-fade plan interpolation:** New plan entries blend from current pose so snapshots never cause a snap.

- **Local idle filler on network stall:** Filler is calm, deterministic, non-committal, and excludes calls/flights/greetings so it does not create canonical behavior.

- **Hidden-tab lifecycle:** Rendering and audio suspend while hidden; visible/resume pulls re-phase the aviary that kept running on the server.

- **Reduced-motion renderer:** It uses the same snapshot data in a "different register," making the aviary "calmer and slower, not broken."

- **Settle lighting and plan bias:** Lighting, call gains, and doze/roost bias support settle as a quiet end-of-attention state with undo and re-engagement.

### 7. Audio pipeline

- **Worklet graph with per-bird gain and panning:** Per-bird gains enable listen-in; panning maps perch position into spatial spread.

- **Procedural synthesis with no samples:** This enforces the no-recorded-audio invariant and avoids stacked-loop/phase-cancel failure modes.

- **Immutable bird signature shaping every note:** Signature continuity makes the same bird recognizable by ear across moods and drift.

- **Runtime-mixed choruses:** Overlapping generated voices create a real chorus rather than loops.

- **Listen-in gain ramps:** Gradual in/out ramps make the interaction feel like listening rather than switching channels, while the 0.30 floor keeps others audible.

- **Autoplay first-gesture resume:** The plan calls this the platform-forced resolution of "calls already audible"; visual aliveness is not gated on audio.

- **Graceful silence with captions auto-enabled:** When WebAudio fails, captions preserve access and no recorded fallback violates the audio invariant.

- **Mute setting:** NOT RECOVERABLE FROM PLAN

- **Preallocated worklet voice pool and bounded allocation:** This supports the zero steady-state heap growth and 30-minute soak gate.

### 8. Accessibility surfaces

- **Single polite live region for screen-reader narration:** One live region plus a line-rate cap protects the screen-reader queue and avoids pushing the user into silencing it.

- **Priority narration lines for greeting, offer, and settle:** Priority lines keep user-event observations prompt while staying in naturalist prose.

- **Same `voice` module for narration and notebook:** A screen-reader user moving between surfaces hears "one product."

- **Visible narration setting:** It renders the same lines for users who want text alongside or instead of screen-reader output.

- **Call captions from exact call specs:** Captions always match what actually played and require no stored strings.

- **Caption scrim:** The scrim guarantees AA contrast against bright and dim scene states.

- **Keyboard navigation map:** Keyboard parity is required for full access to top bar, birds, listen-in, offer sheet, and dialogs.

- **Bird focus proxies:** Invisible DOM buttons provide standard focusable controls while narration carries the atmosphere; accessible names are terse for usability.

- **Dual-tone focus indicator:** The focus treatment is designed to read against midday and night palettes.

- **Dialog focus trapping and restoration:** NOT RECOVERABLE FROM PLAN

- **WCAG AA contrast on user copy:** The plan gates palette/token changes with automated contrast checks.

- **Reduced-motion preference and override:** OS preference plus in-product override prevent motion from being a fallback or retrofit.

- **No strobing or fast flashing:** Motion safety is enforced through animation-curve limits.

- **Accessibility ship-together rule:** Narration, captions, reduced motion, and keyboard parity are GA exit criteria owned from M1.

### 9. Accounts, auth, privacy

- **Synthetic-ID rule:** Account UUID is the only identifier across DB keys, event payloads, queue messages, cache keys, logs, and metrics to keep PII out of operational systems.

- **Encrypted email and HMAC blind index:** Email remains lookup-capable without appearing raw outside the narrow account/invitation storage.

- **Magic-link token storage and rate limiting:** Hashed 256-bit tokens, TTL, atomic consume, and rate limits protect sign-in links.

- **Transactional sign-in email copy:** The copy is direct and dull, matching matter-of-fact system voice and avoiding product emotion around auth.

- **Session device labels and revocation:** These give account settings a device-control surface; edge revocation may lag up to 60 seconds, documented as accepted.

- **Email change flow:** New email verification before commit keeps the old email valid until the change completes.

- **Export JSON including personality vectors:** The plan says this is explicit and the only numeric personality exposure; it is user-pulled and never displayed in UI.

- **Deletion pending bar:** A system bar is allowed because it is an account-deletion surface, not a product announcement surface.

- **Hard-delete purge job:** Purge deletes account, aviary, birds, events, rollups, ticks, notebook, narration, visits, sessions, exports, and edge snapshots, completing erasure after the soft window.

- **Telemetry physical separation and forbidden dimensions:** The boundary prevents account/bird reconstruction and keeps per-bird interaction data inside that account's simulation only.

- **Allowed telemetry categories:** Request counts, latencies, errors, histograms, first-bird timings, delivery rates, and fallback rates are allowed because they serve operational health.

- **Forbidden telemetry categories:** The plan forbids per-account dashboards, visit-frequency analytics, drift aggregates, average-bird dashboards, engagement funnels, and ML training on per-bird fields to preserve the privacy boundary and no-gamification posture.

### 10. Visits

- **Invite flow on demand from settings:** Invites exist only when a host asks for them; there is no global discoverability, onboarding mention, re-invitation, or frequent-visitor status.

- **One-time redemption to seven-day read-only pass:** The decision balances "one-time link" with outstanding/active invite language and remains reversible by configuration.

- **Visitor mode uses the same renderer:** Visitors see identical birds, moods, drift, lighting, and weather, preventing show-off rendering.

- **Visitor mode disables event POST, presence, settle, notebook, offers, and listen-in:** Visitors "cannot trigger anything," and their attention never touches host drift.

- **Visitor local accessibility settings:** Captions, mute, and reduced motion remain device-local because they are not interactions with the host's aviary.

- **Revocation and expiry surface:** Revoked/expired links render "This visit is no longer available," keeping the state matter-of-fact.

- **No host notification of revocation success:** The plan says the visit log's absence of the visitor is the confirmation, avoiding another announcement surface.

- **Visit log:** The log is on-demand in settings and has no badge, containing visitor email, date, approximate duration, and outstanding invitations.

- **Visit notification email:** It is off by default and deliberately dull, with no visit count or engagement framing.

### 11. Performance, observability, testing, rollout, and team

- **Initial JS, first-bird, frame-rate, memory, payload, API, tick, and no-audio budgets:** The plan calls them CI gates, not aspirations, making performance part of product correctness.

- **Synthetic fleet:** Common geography/device profiles measure first-bird paint, frame timing, audio errors, and latency without account dimensions.

- **Aggregate-only RUM:** RUM captures health and performance without identifiers.

- **Operational alarms:** Tick lag, snapshot latency, magic-link failures, audio errors, frame-drop regressions, trait-decrease canaries, presence spikes, and edge-KV lag provide early signals for the risks the plan names.

- **Deliberately unmeasured product analytics:** Visit frequency, streaks, funnels, retention cohorts keyed to behavior, per-bird data, and drift aggregates are absent by architecture.

- **`sim-core` unit/property tests:** They enforce monotonic drift, catch-up equivalence, greeting variation, mood persistence, chronicler sparsity/bans, and weather determinism.

- **Calibration harness:** Accelerated-clock personas own drift targets and gate any `drift_gains` change.

- **Golden replays:** Fixture logs to expected snapshot trajectories detect behavior-rule regressions.

- **Audio variation, chorus, and recognizability tests:** They guard against repetition, loop artifacts, and unrecognizable seven-bird audio.

- **Client e2e and load tests:** They prove boot, keyboard, focus, listen-in, settle, presence, offline, viewport, memory, and load behavior before release.

- **Voice and copy lint:** Lint makes no-announcement and voice-register violations fail CI instead of relying on review.

- **Security and privacy tests:** Token flows, schema gates, log-scrubber audits, and revocation tests enforce account and privacy guarantees.

- **Manual release matrices:** Screen-reader, audio, and visual walkthroughs catch experiential issues the automated gates cannot fully judge.

- **Milestone sequencing:** `sim-core` and CI gates land first so every later merge is measured against invariants from day one; later milestones depend on engine, scene, and audio surfaces.

- **Server config feature flags:** Tunables ship in the snapshot envelope so cadence, drift gains, presence windows, cooldowns, arrivals, narration, visits, audio captions, and edge inlining can change without a client deploy.

- **Birds-per-aviary ramp:** Slots unlock only after recognizability tests pass at that count, making the cap empirical.

- **Phased exposure:** Internal alpha, closed beta, and GA allow human review of notebook voice, recruited screen-reader testing, and drift recalibration from aggregate presence distributions.

- **Instrumented from day one:** The plan starts RUM, alarms, histograms, delivery metrics, fallback rate, and nightly calibration early while keeping them non-account-specific.

- **Rollback posture:** Versioned assets, N-1 API compatibility, config rollback, forward-only migrations with dual reads, and catch-up-safe ticks make recovery possible without corrupting simulation state.

- **Team shape and dependency spine:** Work is split across engine/backend, client/render, audio, a11y/QA, platform, and visual design so `sim-core`, CI gates, scene, audio, interactions, a11y, accounts/social, hardening, and GA can sequence along the dependency spine.
