# Pocket Aviary — V1 Implementation Plan

This plan turns the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`) into an executable build plan. It does not restate the spec; it interprets it into architecture, data model, APIs, algorithms, and rollout sequencing. Where the PRD is ambiguous or internally inconsistent, this plan makes a defensible call and flags it inline with **[CALL]**.

---

## 1. Scope

**In for v1** (per `product_brief.md` scope statement): two starter birds, cap of seven; magic-link single-user accounts; one canonical aviary per account; multi-device sync; field notebook; presence accounting; visit-invitation (opt-in, off by default); screen-reader narration; reduced-motion mode; call captioning. Web only.

**Out for v1** (per `non_goals.md`, enforced as standing constraints, not just absent features): native apps, any gamification surface (achievements, streaks, levels, scores, badges, counters), Tamagotchi mechanics (death, hunger, decaying happiness), social-network surfaces beyond the single visit affordance (profiles, follows, public feed, discovery, leaderboards). These are treated as **architectural exclusions**, not just product decisions — e.g., no schema field anywhere stores a "consecutive days visited" count, because a field that exists is a field someone eventually surfaces.

**[CALL]** Visit-invitation ships in the same v1 release as the rest of the surface. `accessibility_perf.md` is explicit that accessibility cannot ship late ("a reduced-motion mode that lands two months after launch... quietly told reduced-motion users the product wasn't for them"); no equivalent urgency is stated for visits, so if engineering capacity forces a sequencing choice, visits is the correct thing to slip a few weeks (behind a feature flag, OFF for everyone, indistinguishable from "not built yet" since the feature is opt-in anyway) — never narration, reduced-motion, or captioning.

---

## 2. Architecture

**Service shape** — five deployables, deliberately not more for v1:

1. **Edge layer** — CDN + edge functions (e.g. Cloudflare Workers/Vercel Edge) serving the static app shell and, critically, the initial HTML with an embedded state snapshot read from a fast edge-adjacent cache. This is what makes the <500ms time-to-first-bird budget achievable (see §10) — there is no client round trip to origin before the first bird renders.
2. **API service (BFF)** — stateless HTTP service: auth, snapshot reads (post-edge-cache, for polls after the first paint), event ingestion, account/settings, bird rename, visit invite/list/revoke, notebook reads, export trigger. Horizontally scalable, no in-process state.
3. **Simulation tick service** — a pool of workers consuming a "due accounts" queue, running the tick algorithm (§6) and writing canonical state + the snapshot cache. Decoupled from the API service because its workload (batch, CPU-light but latency-sensitive at the p99-5s SLO) and scaling profile (proportional to account count, not request volume) are different.
4. **Scheduler / sweep jobs** — cron-style jobs: enqueue due accounts for ticking, sweep expired visit invites (30 days), sweep accounts past hard-delete date (30 days post soft-delete), evaluate age-gated species-unlock eligibility, expire stale magic links.
5. **Transactional email service integration** — magic links, email-change verification, export download links, visit invites. A managed provider (e.g. SES/Postmark), not a bespoke mailer.

**Data stores:**
- **Primary datastore**: a single relational database (Postgres) as the system of record — accounts, birds, the append-only event log, notebook entries, sessions, visit invites. Relational + transactional because the no-LWW personality-write rule (§7) depends on the tick being able to atomically read-event-log-then-write-vector inside one transaction with a per-account lock.
- **Snapshot cache**: a low-latency KV store (Redis or equivalent), one entry per account, write-through from the tick service on every tick. This is what the edge layer reads for the embedded first-paint snapshot and what the API service reads to answer polls cheaply without hitting Postgres on every request.
- **Object storage**: account export JSON files, served via short-lived signed URLs emailed to the user.

**Why not push/WebSocket**: `accounts_sync.md` specifies polling explicitly ("a low-frequency keepalive while the tab is visible," refetch "on visibility change," "on long render-frame gaps"). This is also the right call independent of the spec — moment-to-moment liveliness (idle motion, calls) is a client-local procedural simulation driven by slow-changing server parameters (mood, personality, current-action descriptors), not a server-pushed animation stream. A WebSocket would be solving a problem this product doesn't have and would cost real complexity in the sync model.

**Why a relational DB and not an event-sourced/Kafka pipeline at v1**: per-account event volume is low (presence pings at multi-minute granularity while present, plus a handful of discrete interactions per session). A dedicated streaming platform is the right answer at a scale this product doesn't have yet. Documented as a forward scaling lever (§ Risks) rather than built now — if account count grows by orders of magnitude, event ingestion can move behind a queue (SQS/Kinesis) ahead of the same tick-consumption logic without changing the data model.

---

## 3. Data model

```
Account
  id: uuid (pk, synthetic — see Privacy in §9)
  email_encrypted: bytes
  created_at, deleted_at: timestamp
  deletion_state: enum(active, pending_deletion, hard_deleted)
  timezone: string (IANA tz, derived client-side at signin and stored for narration/day-night fallback when no client is present to supply "now" in local time during a tick)
  accessibility_prefs: jsonb { reduced_motion: bool, captions_on: bool, narration_on: bool }
  visit_notifications_enabled: bool (default false)
  last_tick_at, last_tick_sequence: timestamp, bigint

Bird
  id: uuid (pk — stable identity, never reissued; see bird_engine.md "Bird identity")
  account_id: fk
  species_id: fk
  name: string (user-assigned, renameable)
  adopted_at: timestamp
  personality_vector: jsonb { boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity } — floats in [0,1]
  mood: enum(wary, content, curious, drowsy, alert, settled)
  mood_started_at: timestamp
  mood_bias: jsonb (small transient contagion weights consumed by next tick's transition, decayed each tick — see §6)
  last_offer_accepted_at, last_offer_at: timestamp (per-bird cooldown enforcement)
  last_greeted_session_at: timestamp (absence-length input + anti-repeat-within-session guard)
  call_seed: bigint (deterministic seed for client-side call/animation scheduling, rotated only on adoption — never on rename)

Species (seed data, not user-writable)
  id, name, silhouette_asset_ref, default_palette, call_motif_library_ref,
  starter_personality_ranges: jsonb

InteractionEvent (append-only; never updated, never deleted except by hard account deletion)
  id: bigint (pk, server-assigned, globally monotonic — the sequencing authority)
  account_id: fk
  bird_id: fk nullable (aviary-wide events e.g. settle have no bird)
  event_type: enum(presence_ping, listen_in_start, listen_in_end, offer, settle, settle_undo)
  payload: jsonb (offer_kind, etc.)
  client_event_id: uuid (idempotency key, unique per account)
  client_occurred_at: timestamp (client clock, used only for duration math, never for ordering)
  received_at: timestamp (server clock, authoritative ordering input via id)

ObservedFact (internal, never user-facing directly — ground truth the notebook prose is rendered from; see §6)
  account_id, date, fact_type (e.g. first_greeter, long_quiet_stretch, weather_event, drift_milestone), payload jsonb

NotebookEntry (read-only to users)
  id, account_id, generated_at, text, bird_ids: uuid[], source_fact_id: fk -> ObservedFact

DeviceSession
  id, account_id, device_label, created_at, last_used_at, revoked_at

VisitInvite
  id, host_account_id, visitor_email_encrypted, token_hash, status: enum(pending, active, revoked, expired),
  created_at, expires_at (created_at + 30d), last_viewed_at, viewed_duration_estimate_seconds
```

**[CALL]** `personality_vector` is stored as floats server-side (it must be, to do additive drift math) but — see §5 — is **never serialized into any API response, export, or log in scalar form.** This is stricter than a naive reading of `accounts_sync.md`'s export feature; see §5 for the conflict and resolution.

---

## 4. API surface

All authenticated endpoints use a per-device session token (cookie, `httpOnly`, `secure`, `sameSite=lax`).

**Auth**
- `POST /v1/auth/request-link {email}` — rate-limited per email.
- `GET /v1/auth/consume?token=` — single-use, 15-minute expiry, issues a `DeviceSession` + sets session cookie. First-ever consumption for a new email triggers adoption (§ below).
- `POST /v1/auth/logout`
- `GET /v1/account/sessions` / `DELETE /v1/account/sessions/:id` — device list + revoke.

**Aviary state**
- `GET /v1/aviary/snapshot` — the read path. Returns per-bird: id, name, species, mood, mood_started_at, perch_zone (derived, see §6), `current_action {type, started_at, duration, seed}` for deterministic client animation, plumage rendering tokens (derived — never the raw saturation float, **[CALL]**, see §5), call-timing parameters (vocal_frequency expressed as an inter-call interval distribution parameter, not the raw trait), plus aviary-level `weather`, `settled`, day/night is **not** sent — computed client-side from account timezone + wall clock, since it's a pure function of "now" and shipping it would just be a stale snapshot of a fast-changing-relative-to-poll-interval value. Also returns a `greeting` directive (see §6) computed at request time, present only on the first snapshot fetch of a session.
- `POST /v1/events` — body `{event_type, bird_id?, client_event_id, client_occurred_at, payload?}`. Idempotent on `client_event_id`. Server-side cooldown validation for `offer` (benign no-op response with a naturalist micro-copy reason, not an HTTP error — this is a product surface, not a system error per `product_brief.md`'s voice split).
- `GET /v1/notebook?cursor=` — reverse-chronological, cursor-paginated, unbounded scrollback.

**Account**
- `GET/PATCH /v1/account/settings`
- `POST /v1/account/email {new_email}` → `GET /v1/account/email/confirm?token=`
- `POST /v1/account/export` → async job, emails a signed download link
- `POST /v1/account/delete` (soft) / `POST /v1/account/restore`

**Birds**
- `PATCH /v1/birds/:id {name}` — the only client-writable bird field. No endpoint, under any role or flag, ever accepts a personality or mood value from a client.

**Visits**
- `POST /v1/visits/invite {email}` (host)
- `GET /v1/visits` (host visit log + outstanding invites)
- `DELETE /v1/visits/:id` (host revoke)
- `GET /v1/visit/:token` → visitor snapshot (same shape as `/v1/aviary/snapshot`, read-only, unauthenticated-but-tokenized, no event-submission endpoint exists for this path, checked against status/expiry on every call so revocation takes effect on next poll with no push needed)

---

## 5. Personality vector exposure — a PRD conflict and its resolution

`bird_engine.md` states the no-numeric-exposure rule four separate times with escalating force: "never... in a debug view," "not at any version, not in any tier," "There is no toggle for it." `concepts.md` calls it "a hard rule, not a default."

`accounts_sync.md`'s account-export feature literally lists "current personality vectors" as exported content.

These conflict. **[CALL]**: the emphatic, repeated, multiply-stated rule in `bird_engine.md` governs. Export ships a **personality expression snapshot** — the same class of derived, qualitative presentation data already sent to the rendering client (e.g. plumage color tokens, a coarse boldness-derived perch-tendency descriptor, vocal-frequency-derived call-rate descriptor) — never the raw `[0,1]` scalars. This is flagged for product confirmation before launch as a genuine spec conflict, not silently resolved. The same rule applies to every other surface: API responses, logs, debug tooling, and the export all go through one shared "presentation projection" function that is the *only* code path allowed to read `personality_vector` and the *only* thing ever serialized outward. No second code path may format the raw vector for any audience, including internal tooling — an internal admin panel that prints "boldness: 0.62" for support purposes would violate the rule as stated just as much as a user-facing one.

---

## 6. Simulation engine design

**Tick cadence and scheduling.** A scheduler enqueues "due" accounts onto a work queue roughly every minute. An account is due if (a) it has unprocessed events since `last_tick_sequence`, or (b) enough wall-clock time has passed that time-of-day mood shaping, ambient weather rolls, or notebook sparsity windows need re-evaluation even with zero new events — the PRD requires the tick to "run whether or not any client is connected." **[CALL]** Accounts with no events in the last 24h are ticked at a relaxed ~5-minute cadence rather than every minute — nothing in the spec needs sub-5-minute precision for an absent account, and "exact cadence... calibrated during build" gives explicit latitude. Tick workers acquire a per-account lock (row-level `SELECT ... FOR UPDATE` or equivalent) for the duration of processing, preventing double-processing if a worker retries after a partial failure.

**Per-tick algorithm:**
1. Load account + birds + events since `last_tick_sequence` (ordered by event `id`, the server-assigned sequence — never by client timestamp).
2. Reconstruct presence-time from `presence_ping` events: sum inter-ping gaps, capping any single gap at the ping interval ceiling (e.g. 2× expected ping period) so a missed ping or a stale background tab can't be credited as continuous presence.
3. Reconstruct per-bird interaction signals: `listen_in_start`/`listen_in_end` pairs → duration (orphaned starts with no matching end are capped at the presence-window end, not left open-ended); `offer` events outside cooldown → curiosity/boldness nudge; `settle` → mood-quieting input only, no drift direction per `bird_engine.md`.
4. Apply drift: for each trait, `new = old + k_input * (1 − old)`, additive and monotonic-only (negative deltas are clamped to zero, never applied) — see calibration constants below.
5. Compute mood transitions from: recent interaction valence, time-of-day (account timezone), active weather, the bird's own personality (a high-boldness bird's transition table down-weights `wary`), and `mood_bias` carried over from neighbor contagion (next bullet). `mood_bias` decays to zero over a few ticks if not reinforced.
6. Neighbor contagion pass: birds currently `wary` add a small positive bias to nearby birds' `wary` transition weight for the *next* tick (a statistical nudge, not an instant flip) — this is the tick-level, slow-cadence reading of "a wary mood tends to spread," consistent with the engine's once-a-minute granularity. **[CALL]** Moment-to-moment "alarm call startles nearby bird" responsiveness within a session is implemented separately as a transient, client-local cosmetic flinch animation layered on top of the server-canonical mood (reset on reload, never written back) — see §8. This keeps "what mood is this bird, canonically" single-sourced on the server (no risk of a client-side mood mutation racing the sync model that §7 is built specifically to avoid) while still delivering the felt bird-to-bird reactivity within a live session.
7. Roll ambient weather: a small per-tick probability of starting a short rain/wind event (independently per account — **[CALL]** this is a simulated in-fiction random process, not a real-world weather API tied to user location; the spec never mentions geolocation and pulling it in would cut against the account's minimal-data privacy posture for no stated product benefit).
8. Recompute `perch_zone` per bird from boldness + current mood modifier, with a minimum dwell time before a bird can change zones again, to prevent visual flicker on rapid mood swings.
9. Update `ObservedFact` ledger with anything tick-derived (today's first greeter, a long quiet stretch, a weather event, a drift-milestone crossing) and run notebook trigger scoring (§ below).
10. Evaluate age-gated species-unlock eligibility (aviary age vs. unlock schedule — **[CALL]** example default thresholds: 3rd bird ~8–12 weeks, 5th by ~6 months, up to the cap of 7 by ~1 year; tunable, owned by a config table, not hardcoded, since this is exactly the kind of calibration the spec says to revisit).
11. Persist new vectors/moods/zones, advance `last_tick_sequence`/`last_tick_at`, write through to the snapshot cache.

**Drift calibration target** (`bird_engine.md`): instrument-detectable after ~1 week of regular visits, user-visible after ~3 weeks, never visible within a single session. Default starting constants: presence contributes the dominant per-minute `k`, sized so ~15–30 daily minutes of presence over a week produces a raw trait delta on the order of 0.01–0.02 (instrument-visible, not user-visible), and over three weeks compounds (via the `(1 − old)` approach-to-ceiling shape) to roughly 0.08–0.12 (user-visible). Listen-in and offers contribute smaller, signal-specific `k` values per the PRD's stated weight ordering (presence > listen-in > offers). These constants are not load-bearing guesses to ship as-is — they're the seed values for the calibration harness in §11.

**Notebook generation.** **[CALL]** Naturalist prose is generated by a curated, weighted phrase-bank/template engine with slot-filling against `ObservedFact` rows — not a per-entry LLM call. Reasoning: entries are deliberately rare (the spec explicitly wants sparsity preserved even for active users), so generative diversity at LLM scale isn't needed to avoid repetition; a backend tick worker running across many accounts needs deterministic latency and cost, not a network call to a model provider; and voice consistency (the lowercase, present-tense, specific-but-never-invented register) is safer guaranteed by author-reviewed phrase banks than by prompting. Critically, **the engine only renders facts that exist in `ObservedFact`** — it never invents a comparison ("first time this week") that isn't backed by a query against the ledger, because a notebook caught stating something false breaks trust in the whole product's voice in one stroke. Entry sparsity is enforced by a trigger-scoring step (interaction-novelty + minimum days-since-last-entry, looser when something crosses a notability threshold) rather than a flat per-session check.

**Return-greeting.** Computed at **read time** in the API service (snapshot endpoint), not in the tick — it must fire within the first second or two of a fresh tab open, which can't wait for the next scheduled tick. Inputs: absence length (`now − last_seen_at`), per-bird boldness/current mood (bolder/more-alert birds win selection more often, but procedurally varied, not a hard top-1 rule), and a per-bird randomized stagger offset when more than one bird would plausibly greet (rendered client-side as offsets within the directive, never simultaneous). A `last_greeted_session_at` guard prevents the directive from re-firing on background keepalive polls within the same session.

---

## 7. Sync model

The server is the sole writer of personality and mood; clients write only to the append-only event log via `client_event_id`-deduped `POST /v1/events`, never an absolute state value (`accounts_sync.md`'s explicit example: a client sends "listened in for 3 minutes," never "set boldness to 0.62"). The tick consumes the log in server-assigned `id` order and applies **additive** deltas inside a per-account locked transaction — this is what makes the two-device race in the spec ("laptop morning session, phone lunch session that started before laptop's ended") unreachable: there is no "latest write wins" because there is no absolute write at all, only ordered increments.

Multi-device sync is consequently not a separate sync protocol — both devices call the same `GET /v1/aviary/snapshot` and get the same canonical record. No client-to-client reconciliation exists or is needed.

**Polling cadence**: aligned to tick cadence (~60s) while the tab is visible, with immediate refetch on `visibilitychange` (hidden→visible) and on detection of a large `requestAnimationFrame` gap (suspend/resume). First paint never waits on this poll — it reads the edge-cached snapshot embedded in the initial HTML (§2).

**Interpolation**: a snapshot's `current_action {type, started_at, duration, seed}` lets the client compute "what is this bird doing right now, mid-action" without waiting for the next poll — this is what makes "the aviary appears already in motion" possible on cold load (§8). When a new snapshot arrives mid-transition with a different target than the client was rendering toward, the client retargets the in-flight tween smoothly rather than snapping.

---

## 8. Frontend rendering pipeline

**Rendering technology**: birds (max 7, each with a handful of simultaneously animated layers) are rendered as DOM/SVG with CSS-transform-driven animation, not canvas/WebGL — at this element count, DOM is well within the 60fps budget, GPU-accelerates transforms/opacity for free, and keeps focus/ARIA semantics native instead of reimplemented. **Ambient ornaments** (drifting leaves, falling feathers, weather) are rendered on a separate `<canvas>` layer, `aria-hidden`, since they're unbounded-count decoration with no accessibility semantics and benefit from a single draw call rather than N DOM nodes.

**Pose/animation**: each bird runs a small client-local state machine (preen, scan, head-tilt, shuffle, calling) selected via personality- and mood-weighted pseudo-random choice, seeded from the bird's `call_seed` plus the `current_action.started_at` time bucket from the snapshot — so a fresh page load reconstructs "what pose, how far into it" deterministically without an extra round trip, satisfying `aviary_layout.md`'s "no spinner, no wake-up animation" requirement structurally rather than by convention.

**Bird visual assets**: procedurally parameterized layered SVG (body/wing/head parts), recolored at runtime from the server's derived plumage tokens, rather than per-species bitmap sprite sheets — keeps the bundle small and lets plumage-saturation drift recolor a bird without shipping new assets.

**Loading/empty states**: cold load with no cached snapshot yet renders the quiet-field placeholder (soft sky color, one or two faint motion cues) specified in `aviary_layout.md` — never a spinner. The empty-aviary state between signup and first-bird-arrival reuses the same quiet field, then the first bird flies in once adoption completes; this is the only entrance animation in the product, and it never recurs after the first session.

**Code-splitting**: the critical bundle is the aviary scene + audio engine + top bar only. Account settings, accessibility settings, and the visit-invitation flow are separate lazy chunks behind their top-bar icons, loaded on demand — they don't compete for the 2MB/500ms budget that's reserved for "bird visible."

**Reduced-motion mode** (full design, not a fallback — see `accessibility_perf.md`): the same pose state machine renders discrete still poses cross-faded via opacity transition instead of continuously tweened motion; ambient canvas leaf/feather layer is disabled entirely; day/night color shift remains but slowed. Calls and captions are unaffected — audio and mood are orthogonal to the motion-reduction axis.

---

## 9. Audio pipeline

**Graph shape**: one WebAudio `AudioContext` per session (created lazily on first user gesture if the browser requires it, suspended on tab-hidden to save battery, resumed on visible). Each bird has its own call-generator subgraph (oscillator/noise source + envelope, shaped by its species' motif library) feeding a per-bird `GainNode`, all summed into a shared "listen-in mix" bus. Listen-in engage/disengage ramps the focused bird's gain up and all others down via `AudioParam.linearRampToValueAtTime` over ~500–800ms — a ramp, never a hard cut, per `interactions.md`'s explicit "must feel like listening, not switching channels." Other birds drop to a quieter ambient floor but never reach zero gain.

**Call scheduling**: each bird independently schedules its next call via a Poisson-ish process whose rate is driven by its vocal-frequency trait (expressed to the client as an interval-distribution parameter, never the raw scalar — §5) and shaped by current mood (drowsy slows/lowers, alert speeds/raises). Because scheduling is independent per bird and genuinely concurrent in the WebAudio graph, two birds landing in the same window produce a **real-time mixed chorus**, not a pre-mixed/stacked loop — this is the mechanism `bird_engine.md` requires to avoid the phase-cancellation artifact of layered recordings.

**Captioning**: caption text is looked up from the same motif/mood parameters that drove the call's synthesis (not stored as a fixed string per call instance), so the caption always matches what actually played; rendered as small fading text positioned near the calling bird.

**Fallback**: WebAudio unavailability (old browser, denied audio permission, hardware issue) is feature-detected once at startup; the app runs in graceful silence with captions defaulted on. No recorded-audio fallback path is built or shipped — per `accessibility_perf.md`, this is unconditional, both because it would feel canned and because shipping it at sufficient quality would blow the bundle budget anyway.

**Memory discipline**: oscillator/envelope nodes are disposed (`disconnect()`) immediately after their envelope completes; concurrent voice count is capped at the bird cap (7) plus a small ambient-layer allowance; no per-call heap allocation that survives the call — this is what the 30-minute no-memory-growth CI test (§10) is actually testing against.

---

## 10. Accessibility surfaces

**Screen-reader narration**: a client-side narration generator shares the phrase-bank/template engine described in §6, parameterized for "describe current state" rather than "describe a noteworthy moment," fed by the same snapshot the visual surface reads — so a screen-reader user and a sighted user are guaranteed to be experiencing the same product, voiced two ways, not two products. Pushed into an `aria-live="polite"` region at a 30–60s idle cadence; user-initiated events (return-greeting, an offer reaction, settle) get an immediate, prioritized update rather than waiting for the next idle tick — still phrased as an observation, never an event-log line.

**Reduced-motion**: see §8 — a fully designed alternate rendering, not a strip-down.

**Captioning**: see §9.

**Keyboard navigation**: top-bar icons in natural tab order. Tab into the aviary scene focuses the first (or most recently focused) bird; arrow keys move focus among birds (roving tabindex, not N separate tab stops); Enter toggles listen-in on the focused bird; Escape exits listen-in. The offer affordance opens via a top-bar shortcut and is itself fully keyboard-operable. Focus rings use a treatment specified to hold visible contrast against both bright daytime and dim night/settled aviary backgrounds (e.g., an outline + drop-shadow combination rather than a single flat-color ring that could wash out against one state).

**Contrast**: WCAG AA enforced for all chrome/user-copy text via design tokens, checked in CI against each token's defined background pairing — not a manual audit that drifts as new surfaces ship.

---

## 11. Performance budgets and observability

- **2MB gzip bundle cap**, enforced by a CI size-budget gate (fails the build, not just warns) broken down per chunk (core scene+audio vs. lazy settings/accessibility/visit chunks).
- **<500ms time-to-first-bird** on mid-tier mobile/4G: achieved by the edge-cached embedded-snapshot first paint (§2) plus a render path that draws the first bird before non-critical assets (fonts beyond the critical subset, lazy chunks) have loaded.
- **60fps idle motion** on a 5-year-old laptop, sustained over a 30-minute session: enforced by a CI/perf-lab check on throttled CPU, using transform/opacity-only animation (no layout-triggering properties) and object/audio-node pooling.
- **No memory growth over 30 minutes**: an automated headless-browser soak test in CI takes periodic heap snapshots and asserts bounded growth — a real gate, not a guideline, per the spec's explicit framing. Notebook scroll virtualizes off-screen entries; audio nodes are disposed per §9.
- **Observability**: synthetic multi-geography browser checks against time-to-first-bird/page-load; aggregate-only RUM (page load, first-bird-render, frame timing, audio-context error rate) with **no per-bird or per-account dimension on any telemetry event** — enforced structurally by a fixed, schema-validated allowlist of telemetry fields at the ingestion boundary, not by policy alone, per `accounts_sync.md`'s privacy section. Simulation-tick latency p99 alarms at 5s.
- Browser support: last two major versions of Chrome/Safari/Firefox/Edge; older browsers get a matter-of-fact unsupported-browser surface, no compatibility shimming.

---

## 12. Privacy and identifier hygiene

Every cross-service reference to an account uses the synthetic `Account.id` UUID — logs, queue messages, cache keys, telemetry — never the email. Email exists in exactly one encrypted column. Per-bird interaction data (events, vectors, moods) is never read by the analytics/telemetry warehouse and never aggregated for any cross-account purpose, including internal-sounding ones like an "average drift across accounts" dashboard — `accounts_sync.md` names that exact example as the thing not to build. This has a direct consequence for calibration validation (§13 below): we cannot tune drift constants by aggregating real users' personality data, even internally, even for a good reason.

---

## 13. Rollout

1. **Internal dogfood** on a small set of team-owned test accounts.
2. **Drift calibration via a synthetic harness**, not real-user aggregation (§12 forecloses that path): a scripted test suite drives QA-owned accounts through simulated presence/interaction sessions at controlled cadences (e.g., "15 min/day for 7 days," "3 hours once") and asserts the resulting trait deltas land in the instrument-visible / user-visible bands targeted in §6. This is the place calibration constants actually get tuned before any real user sees the product, and it's re-run as a regression gate whenever the drift function changes.
3. **Invite-only beta** — real users, full feature set including visits, narration, reduced-motion, and captions live from day one (not a later add — §1). Synthetic perf checks and aggregate RUM running from this point.
4. **Public web launch.**
5. **Post-launch**: age-gated species-unlock thresholds (§6 step 10) and drift constants remain owned by a config table specifically so they can be retuned without a deploy, since both are explicitly named in the PRD as values likely to need adjustment after real (but only ever aggregate-safe) signal comes in.

---

## 14. Risks

- **Drift calibration risk** — too fast reads as a Tamagotchi the user can game by clicking; too slow reads as a screensaver. Mitigated by the synthetic calibration harness (§13) run before any real-user exposure, and re-run as a gate on any change to the drift function.
- **Sync correctness risk** — double-processing an account's events on tick-worker retry would double-apply a delta. Mitigated by the per-account lock during tick processing (§6) and by deltas being idempotent-safe only because they're additive over a strictly-once-consumed event range tracked by `last_tick_sequence`.
- **Audio uncanniness risk** — procedural calls that sound synthetic instead of charming would undermine the product's "affective spine." Mitigated by early, dedicated audio-design prototyping and listening review passes before the calibration beta, independent of the engineering build-and-ship cadence.
- **Accessibility regression risk** — narration/reduced-motion/captions bit-rotting as features that ship after the visual surface in later iterations. Mitigated by treating accessibility sign-off as a release gate equal to the visual surface for every feature, not a parallel track that can lag.
- **Notebook voice risk** — as the phrase bank grows with more contributors, entries drift toward generic or announcement-style phrasing, or a fact-binding bug lets the engine state something untrue ("first time this week" when it isn't). Mitigated by binding every rendered phrase to a queried `ObservedFact` row (§6) and by treating new phrase-bank entries as content review, not just code review.
- **Performance budget creep** — the 2MB/500ms budgets erode silently as features accumulate. Mitigated by CI-enforced hard gates (§10), not dashboards that can be ignored.
- **Privacy-boundary risk** — a future contributor reaches for email or per-bird state as a convenient identifier or analytics dimension. Mitigated by the synthetic-UUID rule plus the schema-level telemetry allowlist (§11, §12) that makes the violation a validation failure, not a code-review miss.
- **Gamification-creep risk** — the long-tail risk named explicitly in `non_goals.md`: a well-intentioned "harmless" engagement feature (a badge, a notification, a streak) reintroduces exactly what the product refuses. Mitigated by treating `non_goals.md` as a standing design-review gate for any new feature proposal indefinitely, not a v1-only document.
- **PRD conflict on export contents** (§5) — flagged for explicit product confirmation before launch rather than silently resolved one way in code with no record of the tension.
