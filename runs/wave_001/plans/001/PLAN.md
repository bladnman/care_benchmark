# Pocket Aviary — v1 Implementation Plan

This plan turns the Pocket Aviary PRD into an executable engineering program. It is written for a team that has read the PRD once; it does not restate the spec, it interprets it into architecture, schemas, APIs, algorithms, budgets, and a delivery sequence. Where the PRD is ambiguous or internally in tension, this plan makes a call and marks it **[Call]** with the rationale, per the planning instructions.

The one sentence that governs every decision below: **the server simulates, the client performs.** Canonical state (personality, mood, weather, notebook) advances only on the server tick; the client renders an expressive, ephemeral performance layer parameterized by that state and never writes canonical state directly. Almost every correctness property the PRD demands — multi-device sync, no-last-write-wins, "the aviary continued without you," identity continuity — falls out of holding that line.

---

## 1. Scope

### In scope for v1

- Web SPA, evergreen browsers only (last 2 majors of Chrome, Safari, Firefox, Edge); matter-of-fact unsupported-browser surface for older ones.
- Single-user accounts; magic-link email auth; per-device revocable sessions; email change with verification; account export (JSON via emailed link); soft-then-hard deletion (30 days).
- One canonical aviary per account; two starter birds (system-selected species, user-named); growth to a cap of seven gated by **aviary age only**.
- Server-side simulation tick (~1/min): personality drift (monotonic toward expressive), mood transitions, ambient weather, notebook entry generation.
- Client surfaces: the single-screen aviary scene; return-greeting; listen-in; offers (seed / song fragment / still pool) with per-bird cooldown; settle (with 5s undo); field notebook (read-only, sparse); presence accounting (three-signal conjunction).
- Procedural call synthesis client-side via WebAudio; chorus mixing; listen-in mix ramps; graceful-silence fallback with captions default-on.
- Visits: per-invite email links, read-only ambient view, revocable, 30-day expiry, silent visit log, opt-in (default-off) visit notification toggle.
- Accessibility as designed surfaces shipping at launch: naturalist screen-reader narration, reduced-motion mode (cross-fade rendering), call captions, full keyboard navigation, WCAG AA contrast on all user copy.
- Performance budgets as CI-enforced gates: ≤2MB gzipped initial JS, <500ms time-to-first-bird (mid-tier mobile / 4G), 60fps idle on a 5-year-old laptop for 30 minutes, zero memory growth over a 30-minute session.
- Aggregate-only operational telemetry; hard pipeline-level privacy boundary around per-bird/per-account interaction state.

### Out of scope for v1 (enforced, not just deferred)

Native apps; any gamification surface (streaks, badges, levels, counters, visit calendars — anywhere, including settings and the notebook); Tamagotchi mechanics (death, hunger, distress, decaying meters); social-network surfaces beyond the single visit affordance (profiles, follows, feeds, discovery, comments, leaderboards, co-presence); push/email re-engagement of any kind; payments; shared or multiple aviaries per account; user-customizable scenes; SSO/passwords.

Two of these are enforced in code, not just in review:

- A **copy linter** (Section 11) blocks announcement-register strings, streak/visit-frequency phrasing, and user-behavior observations in naturalist surfaces.
- The **telemetry schema registry** (Section 12) rejects any metric with a per-account or per-bird dimension, which makes leaderboards and population-level interaction analysis architecturally unreachable, as `social_optional.md` intends.

---

## 2. Adjudicated ambiguities and interpretation decisions

These are the places where the PRD under-specifies or pulls in two directions. Each is decided here so downstream teams don't relitigate.

1. **[Call] Export includes personality vectors despite the never-expose rule.** `bird_engine.md` says the user never sees vector values in any surface, at any tier, with no toggle. `accounts_sync.md` says the export contains "current personality vectors." Resolution: the never-expose rule governs *product surfaces* (its own examples are stats panels, debug views, "how is my bird doing" surfaces); the export is a data-portability escape hatch — the user's data is theirs, which is the same principle behind deletion. We include vectors in the export JSON, under machine-flavored keys (`engine.traits`), render nothing from them anywhere in-product, and present the export flow entirely in the matter-of-fact register. We do not obfuscate the values; fake opacity in a portability artifact would be dishonest in the wrong direction.
2. **[Call] "Tick runs whether or not anyone is connected" is implemented as hot ticking + exact lazy catch-up.** Ticking every idle aviary every minute forever is wasteful at scale and buys nothing observable. The tick function is pure and deterministic (Section 6); for an aviary with no connected client (host or visitor), we suspend scheduled ticks and, on the next wake (snapshot request, visitor arrival, or maintenance job), fold the elapsed interval through the same tick function in one pass, producing **bit-identical state to per-minute ticking** — drift from pre-absence inputs, mood walking through mornings and evenings, weather events, notebook entries, all back-computed with the same seeded RNG. This preserves the architectural property the PRD is actually asserting (one canonical continuously-advancing simulation; never two divergent client simulations) while keeping compute proportional to attention. An aviary with an active visitor session is hot even if the host is away. Equivalence is enforced by a property test: `foldTicks(state, t0→tN)` ≡ N sequential ticks, exactly.
3. **[Call] The client runs a non-canonical "performance layer."** The PRD says clients never tick, but minute-cadence snapshots cannot drive second-to-second aliveness. Resolution: "clients never tick" means clients never advance *canonical* state. The client runs an ephemeral expressive layer — idle micro-motion, call instance scheduling, greeting performance, leaf/feather drift — parameterized by canonical state (mood, traits, call-grammar params from the snapshot) and locally seeded. Nothing in it persists or is ever sent upstream as state; only *interaction events* go up. This is the load-bearing boundary in the whole system and it gets its own interface contract (Section 3.3).
4. **[Call] Return-greeting is a server-directed, client-performed moment.** The greeting must land within 1–2 seconds of tab-open; the tick cadence is a minute. Resolution: the snapshot response includes a server-computed **greeting directive** (which bird greets, greeting class, intensity bounds) chosen from canonical boldness, mood, and absence length (now − last presence event). The client renders a procedurally varied performance inside that directive. Canonical inputs, server-selected greeter, client-side variation: honors `interactions.md` without waiting on a tick.
5. **[Call] Day/night anchors to a single canonical account timezone.** The aviary follows "the user's local time," but a user has multiple devices and visitors see "the aviary as it is." Resolution: the account stores one canonical IANA timezone, updated to the most recently *present* device's reported zone. All canonical time-of-day inputs (mood, lighting state in snapshots, nightjar activity) derive from it; visitors see the host's clock. On a timezone change (travel), lighting re-anchors over ~30 minutes rather than snapping.
6. **[Call] Drift calibration data comes from synthetic cohorts and consented beta users only.** The privacy commitment bans population-level analysis of interaction behavior, which rules out calibrating drift against production aggregates. Resolution: calibration runs against (a) a deterministic simulated-user harness driving the real tick function in CI, and (b) an explicit-consent closed-beta cohort whose accounts are flagged for instrument export. Production accounts are never used. This constraint is named now because a data scientist will otherwise reach for the production event log in month three.
7. **[Call] Browser autoplay policy means first-ever sessions may open in temporary silence.** "Calls already audible" on the first frame collides with autoplay restrictions: browsers block AudioContext start before a user gesture. Resolution: we attempt to start audio immediately (returning sessions with prior gestures on the origin usually succeed); when blocked, the aviary opens in the same graceful-silence-with-captions register as the WebAudio-unavailable path, and audio fades in (≈2s ramp, not a pop) on the first pointer/keyboard event. No "click to enable sound" banner — that would be an announcement. The captions affordance plus visible calling animation carries the moment.
8. **[Call] Transport is HTTPS polling, no WebSockets at v1.** The PRD's consumption model is pull-based (snapshot on visibility change, long frame gap, and low-frequency keepalive) and nothing requires sub-minute server push except visit revocation, which is allowed to take effect "at the next snapshot pull." Polling at 60s visible-keepalive matches the tick cadence, simplifies edge caching and infrastructure, and removes a whole class of connection-state bugs. Revisit only if a future feature needs real push.
9. **[Call] Presence anti-abuse is explicitly not built.** Presence can be spoofed with a mouse jiggler. There is no leaderboard, no streak, no economy — nothing to win. A user who games their own birds' drift is having a strange private relationship with software, which is permitted. We log nothing about it and build nothing against it; anti-cheat machinery would cost privacy and complexity to protect nothing.
10. **[Call] Renderer is Canvas 2D, not WebGL, not DOM.** Seven birds, three perch planes, subtle parallax, ambient particles: comfortably within Canvas 2D budget on a 5-year-old laptop, with a far smaller bundle than a WebGL engine and none of its context-loss handling. Layered offscreen canvases (Section 8) keep idle cost low. WebGL is the documented escalation path if profiling fails the 60fps gate, with the renderer behind an interface to keep that swap contained.
11. **[Call] Visitors do not need accounts.** The invite link carries a token; the visitor's browser gets a scoped, read-only visit session against visitor-specific endpoints. Requiring sign-up to glance at a friend's birds would convert a quiet affordance into a funnel.

---

## 3. Architecture

### 3.1 System shape

Three deployable server components plus the client, all deliberately boring:

- **API service** (Node.js/TypeScript, stateless, horizontally scaled): auth, snapshots, event ingest, invites/visits, account management, export, notebook reads. TypeScript end-to-end is a real decision, not a default: the call-grammar parameter model, the prose/voice engine, and the deterministic state-shaping code are shared between server and client, and keeping them in one language keeps them in one implementation.
- **Simulation worker** (same codebase, separate deployment): executes ticks for hot aviaries on schedule and catch-up folds on wake; sole writer of canonical bird/aviary state. Scaling unit is "hot aviaries per worker"; aviaries shard across workers by `aviary_id` hash with a lease table so exactly one worker owns an aviary at a time (single-writer invariant is structural, not conventional).
- **Email sender** (queue consumer): magic links, invite links, export links, opt-in visit notifications. Transactional only; there is no other email path in the system, which is how "not a notification surface" stays true — re-engagement email is not a policy we resist, it's a pipeline that doesn't exist.

Backing stores: **PostgreSQL** (canonical state, event log, accounts, invites, notebook), **Redis** (tick scheduling/due-queue, rate limiting, snapshot micro-cache), object storage (export files, short-lived signed URLs). CDN in front of static assets and the app shell.

### 3.2 The two-layer simulation boundary

| | Canonical layer (server) | Performance layer (client) |
|---|---|---|
| Owns | personality vectors, mood, perch zone, weather, notebook, aviary age, greeting selection | pose animation, call instances, gaze/head-tilt reactions, leaf/feather drift, greeting choreography, stagger offsets |
| Advances by | tick (~60s) + catch-up folds | requestAnimationFrame + local schedulers |
| Randomness | seeded PRNG, persisted seed lineage, reproducible | local PRNG, never persisted |
| Writes | personality/mood/state tables | nothing — emits interaction events only |
| Survives reload | yes | no, by design |

Rule with teeth: **if losing it would matter, it is canonical; if losing it is invisible, it is performance.** A bird's mood is canonical (visible cross-device); the particular preen animation playing right now is performance (no one can tell it restarted). Code review enforces this at the event-schema boundary: the client cannot express a canonical write because no API accepts one.

### 3.3 Snapshot/event contract

Client → server: append-only **interaction events** (`presence_ping`, `listen_in_start`, `listen_in_end`, `offer_placed`, `offer_outcome_observed`, `settle`, `settle_undone`, `session_start`, `session_end`), each with client-generated UUID (idempotency key), device id, client timestamp, server receipt timestamp. Server receipt order is canonical order.

Server → client: **state snapshots** — a few KB of JSON: tick index (monotonic version), per-bird canonical state + expressive parameters (call-grammar params, idle-behavior weights), aviary ambient state (lighting phase, weather), greeting directive (on session-start snapshots), notebook "has new entries" flag. Clients render forward from a snapshot and interpolate when the next one arrives; a snapshot with a lower tick index than the one being rendered is discarded (stale read protection).

---

## 4. Data model

PostgreSQL, all account references by synthetic UUID. The email column exists on exactly one table, encrypted at the application layer (AEAD, KMS-managed key). No other table, log line, queue key, or metric ever carries email — enforced by a lint rule on the telemetry/log schema and by code review on partition keys.

```
accounts            id uuid PK, email_encrypted bytea, email_hash bytea UNIQUE (HMAC for lookup),
                    timezone text, created_at, deletion_requested_at nullable,
                    settings jsonb (visit_notifications_opt_in bool default false,
                                    captions_on bool, reduced_motion_override enum)

sessions            id uuid PK, account_id FK, token_hash bytea, device_label text,
                    created_at, last_seen_at, revoked_at nullable

magic_links         id uuid PK, account_id FK, token_hash, created_at,
                    expires_at (15 min), consumed_at nullable

aviaries            id uuid PK, account_id FK UNIQUE, created_at (aviary age root),
                    last_tick_index bigint, last_tick_at, rng_seed bytea,
                    weather_state jsonb, settled_state jsonb,
                    next_adoption_offer_at timestamptz

birds               id uuid PK (stable identity, never reused), aviary_id FK,
                    species_id FK, name text, adopted_at,
                    traits jsonb {boldness, social_warmth, vocal_frequency,
                                  plumage_saturation, curiosity} -- floats [0,1]
                    mood enum(wary|content|curious|drowsy|alert|settled),
                    mood_since timestamptz, perch_zone enum(front|middle|back),
                    call_seed bytea (fixed at adoption; signature anchor)

species             id, silhouette_ref, palette_ref, motif_library jsonb,
                    is_night_active bool  -- exactly one nightjar-like species

interaction_events  id uuid PK (client idempotency key), aviary_id FK,
                    bird_id FK nullable, device_session_id FK, type enum,
                    payload jsonb, client_ts, server_ts, tick_consumed bigint nullable
                    -- partitioned by month; index (aviary_id, server_ts)

presence_rollups    aviary_id FK, window_start, seconds_present int
                    -- written by tick from presence_pings; raw pings prunable after rollup

notebook_entries    id uuid PK, aviary_id FK, written_at, prose text,
                    generator jsonb (template id, inputs — for QA, never displayed)

invites             id uuid PK, host_aviary_id FK, visitor_email_encrypted bytea,
                    token_hash, created_at, expires_at (30 days),
                    revoked_at nullable, consumed_at nullable

visit_sessions      id uuid PK, invite_id FK, started_at, last_poll_at, ended_at
                    -- powers the visit log (email shown decrypts from invite)

tick_leases         aviary_id PK, worker_id, lease_expires_at

export_jobs         id uuid PK, account_id FK, requested_at, completed_at,
                    object_key, link_expires_at
```

Notes that carry weight:

- **`birds.id` is the identity.** Rename, sync, species-pool migrations, schema migrations — nothing ever replaces this row's id or resets `traits`. Migration policy (written into the runbook): bird rows are forward-migrated in place; any migration that cannot preserve `traits` per-bird does not ship.
- **`traits` is stored state, never derived.** No code path recomputes it from the event log at runtime. Event-log replay exists only as an offline integrity check (Section 7), never as a serving path.
- **`call_seed` is fixed at adoption** and anchors call-signature recognizability across mood and drift (Section 9).
- **Notebook prose is stored as written.** Entries are generated once by the tick and persisted; they are not re-rendered from state later (re-rendering would let a template change silently rewrite history, which violates the observer's-record framing).
- **Deletion:** `deletion_requested_at` set → ticks suspend, sign-in shows the restore surface. A daily reaper hard-deletes accounts past 30 days: cascade across all tables above, plus export objects and email-queue residue. Aggregate telemetry carries no account dimension, so there is nothing to scrub there — by construction, not by cleanup job.

---

## 5. API surface

HTTPS JSON, session cookie (HttpOnly, SameSite=Lax) carrying the device session token. All naturalist/matter-of-fact voice rules apply to response copy fields; error payloads are always matter-of-fact.

**Auth and account**

```
POST /auth/magic-link        {email}            -> 202 always (no account enumeration)
GET  /auth/consume?token=    -> sets session cookie; single-use; 15-min expiry
GET  /account/sessions       -> device list      DELETE /account/sessions/:id
POST /account/email-change   {new_email}        -> verify-before-commit flow
POST /account/export         -> 202; emailed signed link when ready
POST /account/delete         -> soft-delete      POST /account/restore
GET  /account/settings       PATCH /account/settings   (timezone, captions, visit-notify opt-in)
```

**Aviary state and events**

```
GET  /aviary/snapshot                 -> full snapshot (see 3.3)
       ?session_start=1              -> include greeting directive; server logs session_start
POST /aviary/events                   -> batch of interaction events; idempotent per event UUID
GET  /aviary/notebook?cursor=         -> reverse-chron entries, infinite scrollback
POST /aviary/adopt                    -> when an adoption offer is open: accepts {name};
                                         server picks species; returns fly-in directive
POST /aviary/birds/:id/rename         {name}
```

Snapshot pull triggers (client-owned): tab becomes visible; render-frame gap >5s (suspend/resume detection); 60s keepalive while visible; after `settle`; on `adopt`. Event flush: batched every ~10s while active and on visibilitychange→hidden via `sendBeacon` (presence end must not be lost to tab close).

**Visits**

```
POST   /invites              {visitor_email}    -> emails one-time link; 30-day expiry
GET    /invites              -> outstanding invitations
DELETE /invites/:id          -> revoke (idempotent)
GET    /visits/log           -> visitor email, date, approx duration, per visit

GET  /visit/:token/snapshot  -> read-only snapshot, no greeting directive,
                                no notebook, no event acceptance; 404-equivalent
                                matter-of-fact surface when revoked/expired
```

The visitor endpoint set is a separate router with a separate, narrower serializer — co-presence and visitor interactivity are not "disabled," they are unrepresentable (no event endpoint exists under `/visit/`). Visitor polls at the same 60s cadence, which bounds revocation latency to one poll interval as specified. Visitor presence writes nothing to the host's event log; `visit_sessions.last_poll_at` exists only to compute the visit-log duration.

**Rate limits** (Redis, per-email and per-IP): magic links 5/hour/email; invites 10/day/account; event batches sane-capped per session; export 2/day. Limit responses are matter-of-fact.

---

## 6. Simulation engine design

### 6.1 Tick function

The heart of the system is one pure function, property-tested to destruction:

```
tick(state, events_in_window, t_start, t_end, rng) -> {state', notebook_drafts, emitted}
```

- Deterministic given inputs; `rng` is a counter-mode PRNG keyed by `(aviary.rng_seed, tick_index)` so any tick is reproducible in tests and incident forensics.
- The scheduler (hot path) calls it with a ~60s window; the catch-up path folds an arbitrary absence interval as N logical windows in one in-memory pass (identical results, one DB write). Property test: fold ≡ sequence, exactly, including notebook output and weather.
- Per-tick pipeline: (1) roll up presence pings → presence-seconds; (2) apply drift; (3) advance weather state machine; (4) mood transitions; (5) perch-zone reselection; (6) adoption-offer timer; (7) notebook noteworthiness pass; (8) write state + mark events consumed, transactionally.

### 6.2 Drift function

Per trait, per tick: a bounded, non-negative delta — a low-pass filter with a saturating daily budget.

```
raw   = Σ (weight_signal × signal_amount_this_window)        // signals below
delta = min(raw, daily_budget_remaining(trait)) × (1 - trait) // ease toward ceiling
trait' = clamp01(trait + delta)                                // never negative
```

Signal weights (initial calibration, all server-side config — tunable without client deploys):

| Signal | Traits moved | Initial weight (per unit) |
|---|---|---|
| presence-seconds (dominant) | all traits, plumage & boldness foremost | sized so 30 min/day ≈ 0.004/day on dominant traits |
| listen-in seconds on bird B | B.social_warmth, B.vocal_frequency | ≈2× presence rate while engaged |
| offer accepted by bird B | B.curiosity | fixed micro-delta per acceptance |
| offer placed near bird B | B.boldness | half of acceptance delta |
| settle | none (mood-quieting only) | 0 drift |

- **Daily budget per trait** (~0.006) is the no-single-session-visibility guarantee: even a marathon session cannot move a trait past instrument-noise level in one day. This is the engine-level enforcement of "a single session never moves a personality value visibly."
- **Monotonicity is structural:** the delta expression cannot produce a negative number; there is no decay term, no neglect penalty, anywhere. Absence simply contributes zero. "Quieter when ignored" is an *expression* effect: greeting probability, call density, and front-perch propensity are functions of recent presence-rollup history at render-parameter time — the underlying traits are untouched. A returning user's birds re-express within a session or two because nothing was ever lost.
- **Calibration targets, as tests:** the CI harness drives synthetic cohorts (daily 20–40 min watcher; twice-a-week visitor; one-week-absence-then-return; mouse-jiggler degenerate) through the real tick. Asserted bands: regular cohort shows instrument-detectable drift (≥0.02 on a dominant trait) by day 7 ±2 and crosses *expression thresholds* (greeting-order change, perch-mix change, call-rate multiplier step) in the day 17–28 band. Expression thresholds are the definition of "visible to the user," which converts the PRD's three-week promise from vibes into an assertable property.

### 6.3 Mood model

Mood is a per-bird state machine evaluated each tick as weighted propensities over `{wary, content, curious, drowsy, alert, settled}`:

```
P(next mood) ∝ base(time_of_day, account_tz)          // drowsy↑ near dusk, alert↑ early morning
             × recent_interaction_modifiers            // accepted offer → content↑ (decays over ~2h)
             × ambient_modifiers                       // rain: vocal-adjacent moods damped; alarm call: wary↑ nearby
             × personality_modulation                  // high boldness suppresses wary entry
             × stickiness(current_mood, mood_since)    // hysteresis: no flapping between ticks
```

- "Daily-ish reset" is implemented as **decay toward the time-of-day baseline**, not a scheduled hard reset — so mood never snaps on tab-open and overnight catch-up naturally walks a drowsy-at-dusk bird into settled-by-night and alert-by-morning. Cross-session persistence costs nothing extra; mood is just canonical state that keeps evolving.
- Mood contagion (bird-to-bird): a wary transition emits an aviary-local modifier consumed by other birds' evaluations for the next few ticks, attenuated by their boldness.
- Weather is a per-aviary state machine driven by the tick's seeded RNG: ~2–4 short rain events and occasional wind per week, duration minutes, never severe. Weather is canonical (visitors and all devices see the same rain) and feeds mood modifiers.

### 6.4 Call-grammar runtime (canonical side)

The server does not synthesize audio; it owns the *parameters* that make calls personal and recognizable:

- Per bird: `signature = f(species.motif_library, bird.call_seed)` — a fixed motif skeleton, pitch center, and timbre anchor chosen at adoption and never re-derived. Mood and traits modulate only *density, tempo, ornamentation depth, and chorus-join eagerness*. Drift therefore changes how often and how elaborately Pip calls, never what Pip sounds like — recognizability across drift is a structural guarantee, not a tuning aspiration.
- Snapshot carries per-bird call parameters: motif weights, rate (calls/min by mood × vocal_frequency), pitch/tempo envelopes, chorus-join propensity, plus an aviary chorus hint (window phase where high-vocal-frequency birds may overlap). The client schedules actual call instances locally (Section 9).

### 6.5 Notebook generation

Runs in the tick, server-side, in the same transaction as state writes.

- **Noteworthiness scoring** over tick-derived observations: firsts and order changes ("pip greeted before wren — first time this week," from greeting-selection history), streak-free comparatives (perch-pattern shifts, unusually quiet mornings, a weather moment two birds reacted to), offer vignettes. Inputs are aviary observations only; the generator's input schema *cannot reference* user-behavior aggregates (no visit counts, no session frequency fields exist in its input type) — `interactions.md`'s line between observing the aviary and observing the user is enforced by the type system.
- **Sparsity governor:** token bucket targeting ~1 entry per 2–4 days for a regularly-visited aviary; high-noteworthiness events may spend ahead, low-grade observations are dropped, not queued. Very active aviaries stay sparse because the bucket, not the event rate, sets the cadence.
- **Prose** comes from the shared voice engine (Section 11): naturalist register, lowercase, present tense, bird names, seeded phrasing variation so templates don't fingerprint. Generator metadata is stored for QA but never displayed.

### 6.6 Adoption pacing

`next_adoption_offer_at` is a function of `aviary.created_at` alone (config: 3rd bird offered ~month 3, then roughly every 2–3 months, cap 7; exact ladder is product-tunable config). The offer surfaces as a quiet moment in the aviary flow (a new bird seen at the edge of the scene; naturalist copy; accept → name → fly-in directive), never as a modal or badge. Declining leaves the offer open; nothing nags. No input other than age exists in the formula — there is no code path from engagement to birds.

---

## 7. Sync model

Single-writer architecture makes sync a property, not a feature; the plan's job is to keep the invariants explicit and tested.

- **Invariant 1 — one writer:** only the simulation worker holding the aviary's lease writes `birds.traits`, `birds.mood`, `aviaries.weather_state`. The API service writes only the event log, invites, and account tables. Enforced by separate DB roles: the API service's role has no UPDATE grant on canonical-state columns. A bug cannot violate the architecture without failing at the database.
- **Invariant 2 — additive deltas in event order:** clients submit observations ("listened in to Pip for 190s"), never values. The tick consumes events ordered by server receipt; replays/retries are absorbed by event-UUID idempotency. The phone-overwrites-laptop failure in `accounts_sync.md` is unrepresentable: there is no API shape that carries a trait value upstream.
- **Invariant 3 — monotonic snapshots:** snapshots carry `tick_index`; clients never render backward. Two devices polling the same record may briefly differ by one tick; they converge at next poll, and because neither ever wrote state, convergence is automatic.
- **Lease correctness:** tick leases (Postgres row, expiry-and-renew) guarantee at-most-one worker per aviary; a worker that loses its lease mid-tick aborts its transaction (state write and event consumption are atomic, so a re-run by the new lease-holder is exact).
- **Offline integrity check:** a nightly job replays a sampled aviary's event log through the tick function from a historical state checkpoint and diffs against stored state. This is the "no log entry says we lost data here" detector — silent drift-loss bugs become a nightly alarm instead of a user's vague unease. (Replay is an audit tool only; serving state is never derived this way.)
- **Conflict surfaces:** the only user-visible "conflicts" are session-level (expired magic link, timed-out session, failed load) and use the matter-of-fact copy from `accounts_sync.md` verbatim. There is deliberately no aviary-state conflict UI because no aviary-state conflict can exist.

---

## 8. Frontend rendering pipeline

### 8.1 Stack and structure

TypeScript SPA; Preact (or React with aggressive code-splitting — final call at M1 by bundle math) for the DOM shell (top bar, notebook, settings, dialogs); the aviary scene is a **Canvas 2D layered renderer** owned by plain TS, no framework in the frame loop.

Layers (separate offscreen canvases, composited per frame; dirty-tracking per layer):

1. Sky/lighting gradient — local-time-driven; updates ~1/min or on settle ramp
2. Background foliage — near-static; slight parallax; redraw on resize/light change
3. Mid plane — perches + birds (the only every-frame layer)
4. Foreground — occasional branch/leaf passes
5. Effects — rain/wind particles (pooled), settle dimming
6. DOM overlays — captions, focus rings, top bar (CSS-faded after cursor stillness)

### 8.2 Bird animation system

- Procedural skeletal poses per species (head, body, tail, wings as parameterized vector parts) with pose-graph blending; **behavior trees per mood** select idle programs: wary → back-perch scanning, longer stillness, head-snaps to sounds; content → preening cycles; curious → head-tilts toward call/leaf events; drowsy → low posture, fluff, slow blink; settled/night → eyes closed, minimal motion (nightjar species stays active).
- All timing jittered by smoothed noise so no two preens are identical; motion frequencies are personality-scaled (boldness → front-perch propensity at perch reselection; the renderer reads canonical `perch_zone` and animates transitions between zones).
- Snapshot interpolation: birds move smoothly to new canonical perch/mood expressions over seconds; never teleport. Mood changes blend via transitional gestures (a shake-out, a reposition), not a switch.
- Greeting choreography: executes the greeting directive — glance / two-note call / step-to-front / call-and-response classes, intensity by absence band, client-jittered stagger (200–900ms) when a secondary bird responds. Variation comes from the pose/timing jitter system, so no two greetings render identically even within a class.

### 8.3 First-paint path (the 500ms budget)

Budgeted milestone list on mid-tier-mobile/4G (CI-enforced with throttled synthetic runs):

1. HTML + inline critical CSS paints the quiet field (soft sky gradient, faint motion via one CSS animation) at ~first paint — this is the loading state and the empty-aviary state; no spinner exists in the product.
2. A ~30KB inline bootstrap module starts the snapshot fetch immediately (parallel with main bundle download) and can draw silhouette-level birds on the mid-plane canvas the moment state arrives.
3. First bird visible (silhouette + motion) < 500ms; detail layers (plumage textures, foliage richness) hydrate progressively afterward without any visual "pop" (cross-fade ≤300ms each).
4. Main bundle (≤2MB gz hard cap; ~1.2MB target) finishes; full pose system takes over seamlessly from the bootstrap renderer mid-motion.

Route-level code splitting: notebook, settings, invite flow, export, adoption flow are all lazy chunks. Audio worklet code loads after first paint (audio is never on the first-paint critical path; see autoplay call).

### 8.4 Reduced-motion mode

Activated by `prefers-reduced-motion` or the in-product setting. A distinct render mode, designed and art-directed, not a flag that zeroes animations:

- Idle motion → curated still-pose sequences per mood with slow cross-fades (2–4s)
- Perch transitions → cross-fade at origin and destination, no flight path
- Leaf/feather drift removed; particle systems off; day/night color shifts retained, slowed
- Greeting → a single held pose change with one cross-fade (the bird *has noticed*; nothing moves fast)
- Captions/narration/notebook/audio unchanged — the aviary is the same aviary

Reduced-motion has its own visual QA checklist and its own screenshot-diff suite in CI; it ships at launch (a post-launch reduced-motion mode is, per the PRD, a launch that excluded those users).

### 8.5 Lifecycle

`visibilitychange→hidden`: stop rAF, suspend AudioContext, flush events via sendBeacon. `→visible`: pull snapshot (with greeting directive — returning from another tab is a return), resume rendering mid-motion. Render-frame gap >5s (laptop lid) → treat as resume. The scene never shows a "reconnecting" state; at worst it renders forward from the last snapshot for a few seconds.

---

## 9. Audio pipeline

### 9.1 Synthesis architecture

WebAudio, AudioWorklet-based voice synthesis (worklet keeps the render thread clean; fallback to main-thread nodes where worklets are unavailable):

- **Per-bird voice**: 2–3 oscillator partials + filtered-noise breath component + pitch/amplitude envelope shaper, configured by species timbre anchor + bird `call_seed`. Seven voices + ambient bed is trivially within budget; all buffers and nodes are pooled and reused (the no-memory-growth rule applies to audio first — per-call node allocation is the classic leak).
- **Call scheduler** (client, performance layer): consumes snapshot call parameters; schedules call instances with seeded jitter on motif selection, inter-call gaps, pitch micro-variation (±2–4% around the fixed pitch center), ornamentation by mood. Signature invariants (motif skeleton, pitch center, timbre) are read-only to the scheduler — recognizability cannot be tuned away by accident.
- **Chorus**: birds whose chorus-join propensity and the aviary chorus hint align overlap naturally; voices are independent synth instances so overlap is true polyphony (no phase-cancel artifacts, the PRD's stated reason for refusing loops). A master bus compressor (gentle, slow) keeps choruses from stacking harshly.
- **Ambient bed**: very quiet procedural wind/foliage noise; rain adds a filtered-noise layer during weather events.

### 9.2 Listen-in mix

Per-bird gain nodes on the master bus. Engage: focused bird ramps to foreground (+4–6dB relative) while others ramp to an ambient floor (≈ −12dB relative, **never −∞**) with exponential ramps of ~2s. Disengage (re-click, focus elsewhere, click empty space, keyboard blur): same ramp back. The ramp constants are product-feel constants — they live in the same tuning config as drift weights and get the same calibration-review treatment. Listen-in start/end are also interaction events (drift inputs) and caption/narration priority hints.

### 9.3 Decay, mute, fallback

- Settle: aviary-wide slow ramp down to near-quiet over the settle lighting shift; undo (≤5s) ramps back.
- User mute (top bar): audio off, captions offered (one-time matter-of-fact prompt in settings, not a toast). Mute state is an interaction signal the engine may read (per product brief: "whether you mute the calls") as a mild ambient-preference input — **[Call]** muted sessions still count presence fully; mute only damps the vocal-frequency drift weight slightly. Never a penalty.
- WebAudio unavailable or autoplay-blocked-pre-gesture: graceful silence, captions on by default, visual calling animations carry the birds' vocal life. There is no recorded-audio path anywhere in the codebase.

---

## 10. Accessibility surfaces

Accessibility items are part of each feature's definition of done from M1 — there is no accessibility milestone, because that's how it ends up shipping late.

- **Screen-reader narration**: an ARIA live region (`aria-live="polite"`) fed by the client-side prose engine reading the same canonical snapshot + local events the visual reads. Idle cadence one prose update per 30–60s (jittered); user-initiated events (greeting, offer outcome, settle) preempt politely at next queue opportunity. Composition rules: running naturalist prose, scene-level ("a small grey bird is perched on the front rail, calling softly…"), never state-list, never trait or mood *labels* — the narration describes what a watcher would see, in field-notebook voice, with seeded variation to avoid template fatigue. Narration copy passes the same voice linter as the notebook.
- **Captions**: generated at synthesis time from the actual scheduled call parameters (motif shape, repetition, position) → "a low trill, paused, low trill again." Rendered as small DOM text near the calling bird, fade in/out with the call, AA contrast against both bright and dim scenes (scrim allowed at night). Because captions derive from the same parameters the synth consumed, they can never describe a call that didn't happen.
- **Keyboard**: top bar is a tab sequence; Tab into the scene focuses the first bird; arrow keys move between birds (DOM focus proxies positioned over canvas birds, with accessible names = bird names); Enter = listen-in; Escape = disengage; offer and settle reachable by top-bar shortcuts; offer dialog fully keyboard-operable; settle-undo reachable by keyboard (any key activation within 5s window counts as the "click anywhere"). Focus indicator: soft high-contrast outline tested against morning, midday, evening, night, and settled palettes.
- **Contrast**: AA floor on all user copy (top bar, captions, dialogs, notebook, system surfaces), verified by automated checks against every lighting state, not just the default.
- **Testing**: axe-core in CI on all DOM surfaces; a manual screen-reader script (VoiceOver + NVDA) runs per release covering first-visit, return-greeting, listen-in, offer, settle, notebook; narration output has snapshot tests through the voice linter.

---

## 11. The voice system (cross-cutting)

Voice is load-bearing and appears in four surfaces (notebook, narration, captions, offer/adoption copy) plus a hard register switch for system surfaces. Build it once:

- A shared **prose engine** (TS, isomorphic): naturalist lexicon, template grammars with slot-level variation, seeded phrasing jitter, lowercase/present-tense enforcement; a matter-of-fact register module for system surfaces with the PRD's sample strings as canonical fixtures.
- A **copy linter in CI** over all user-facing strings and prose-engine output corpora: bans exclamation marks, "welcome", "achievement", "streak", "level", second-person address and announcement framing in naturalist surfaces; bans naturalist phrasing in system surfaces; flags any string referencing user visit frequency anywhere. New surfaces must declare a register; undeclared strings fail the build. This is what makes "notice, never announce" survive contributors, not just survive review.
- Register boundary rule encoded as the PRD states it: money/identity/errors/settings → matter-of-fact; everything else → naturalist.

---

## 12. Performance budgets and observability

### Budgets as CI gates (red = no merge)

| Budget | Gate |
|---|---|
| Initial JS ≤2MB gz (target 1.2MB) | bundle-size check per PR, hard fail at cap |
| First bird <500ms (mid-mobile, 4G) | synthetic throttled run per PR on the first-paint path; device-lab weekly |
| 60fps idle, 5-year-old laptop, 30 min | weekly profiling job, CPU-throttled (4×) frame-time assertion p95 <16.6ms |
| Zero memory growth over 30 min | nightly soak: heap snapshot delta <2% after GC, audio node count constant, listener count constant |
| Tick latency | p99 alarm at 5s (target p50 <250ms); per-tick duration histogram |

### Observability

- Synthetic monitoring: scripted browsers from several geographies load the aviary on schedule, measuring page-load, first-bird-render, frame timing, audio-context errors.
- RUM, **aggregate-only**: load timings, first-bird timings, frame-time histograms, audio errors, API latencies/error rates, anonymized session-duration histograms. The metric schema registry has no account/bird dimension type; adding one is a build failure, not a policy violation. This is the enforcement mechanism for the `accounts_sync.md` privacy boundary and, deliberately, the thing that makes leaderboard-shaped features unbuildable later.
- What we deliberately do not measure: engagement funnels, retention cohorts keyed to interaction behavior, per-bird interaction analytics, visit-frequency distributions per account. Operational health only.
- Infrastructure boundary: the analytics warehouse has no connection to the simulation database; the simulation database has no read replica exposed to analytics; export of `interaction_events` to any pipeline is denied at the DB-role level.

---

## 13. Security and privacy engineering (summary of commitments already woven in)

Synthetic UUIDs everywhere except the one encrypted email column (+HMAC lookup hash); magic-link and session and invite tokens stored only as hashes, 256-bit random, single-use where applicable; 15-min magic-link expiry; per-device revocation; verify-before-commit email change; visitor tokens scoped to a read-only router; rate limits per email/IP; CSRF protection on state-changing routes; CSP locked to self+CDN; soft/hard deletion with full cascade; export via short-lived signed URL to the verified address. Privacy boundary enforced at DB-role and metric-schema level (Sections 4, 12).

---

## 14. Rollout

**M0 — De-risk the spine (weeks 1–4).** Two prototype spikes, both throwaway-allowed: (a) procedural call synthesis + chorus for 3–7 voices with listen-in ramps — perceptual review answers "does this avoid uncanny?" before anything depends on it; (b) Canvas idle-motion at 60fps on the reference old laptop with 7 birds + particles. Exit criteria are listening-panel approval and frame-time numbers, not demos. Tick function skeleton + fold-equivalence property tests land here too.

**M1 — Canonical core (weeks 3–10).** Postgres schema, auth (magic links, sessions), event log + ingest, simulation worker with leases, drift/mood/weather in the tick, snapshot endpoint, calibration harness with synthetic cohorts running in CI. Definition of done includes the nightly replay-integrity job.

**M2 — The aviary (weeks 8–18).** Renderer layers, bird pose/behavior system, greeting directive end-to-end, listen-in (visual + audio mix + events), offers with cooldowns and mood/curiosity-shaped reactions, settle with undo, presence accounting (three-signal conjunction; activity window default 4 min, config range 2–8 for calibration), first-paint path against the 500ms gate, adoption + empty-aviary + fly-in. Accessibility and reduced-motion built per-feature within this milestone, not after it. Voice engine + copy linter land at the start of M2 because notebook and narration both consume them.

**M3 — Notebook, accounts, visits (weeks 14–22).** Notebook generation + reader UI; settings/sessions/export/deletion surfaces (matter-of-fact register); invite issuance/revocation/expiry, visitor read-only path, visit log, opt-in notification toggle.

**M4 — Hardening and beta (weeks 20–28).** Closed beta with a consented calibration cohort (~200 accounts): week-1 instrument check and week-3 expression check against the drift bands; perceptual review of greetings, chorus at 4–7 birds (lab aviaries seeded at high bird counts — production aviaries can't reach 7 for months, so the cap's audio premise is validated in-house), notebook sparsity tuning; performance device-lab passes; accessibility manual passes; privacy red-team (grep logs/metrics for any PII or per-account leakage).

**GA.** Launch with adoption ladder live (everyone starts at two birds; the age-gated third-bird offer means the population ramps bird-count slowly by design — the ramp *is* the pacing mechanic, no separate feature-flag ramp needed). Instrumented from day one: all Section 12 metrics, tick-health dashboards, error budgets. Drift coefficients, mood weights, mix ramps, notebook bucket rates are all server-side config — calibration adjustments post-launch require no client deploy and no schema change.

**Kill-switches and degradations** (each lands with its feature): audio → silence+captions; weather off; notebook generation pause (entries are sparse; a paused generator is invisible for days — safe); invite issuance pause; per-aviary tick-suspend for incident isolation. There is deliberately no kill-switch that degrades into an announcement surface (no banner system exists to abuse).

---

## 15. Risks

| Risk | Why it's real | Mitigation |
|---|---|---|
| **Drift mis-calibration** (too fast = Tamagotchi, too slow = screensaver) | The PRD pins targets but the presence→delta constants are guesses until tested; privacy rules forbid production-data tuning | Deterministic synthetic-cohort harness in CI with asserted week-1/week-3 bands; consented beta cohort verification; all coefficients server-side config; expression thresholds (not raw numbers) define "user-visible" so we tune against the thing users feel |
| **Sync/drift-loss bugs that no one sees** | The failure mode is silent under-drift, invisible to tests and users alike | Single-writer enforced by DB grants; additive-delta-only API shapes; idempotent event log; nightly event-log replay diff against stored state (the designated silent-failure detector); fold≡sequence property tests |
| **Audio uncanniness** | Procedural birdsong is hard; "almost right" birdsong is worse than stylized; chorus blur at high bird counts threatens the 7-cap premise | M0 spike with human listening panel before dependence; stylized-not-imitative sound direction as an explicit aesthetic decision; fixed signature anchors per bird; lab testing at 5–7 birds in M4; compressor + chorus-window scheduling to keep overlaps musical |
| **Autoplay restrictions vs. "calls already audible"** | Browsers will block first-session audio | Designed silent-open state (captions + visual calling) that is indistinguishable in register from the WebAudio-fallback state; gesture-triggered 2s fade-in; measured via audio-start telemetry |
| **First-paint budget slips** | 500ms on 4G is unforgiving; the conceit dies above it | Inline bootstrap renderer + parallel snapshot fetch + silhouette-first drawing; per-PR throttled synthetic gate so regressions are caught at merge, not at launch |
| **Accessibility regression / late slip** | The cheap fallback is always one deadline away | A11y in per-feature definition of done; axe + voice-linted narration snapshots in CI; manual SR script per release; reduced-motion has its own screenshot suite; launch gate includes the a11y checklist |
| **Memory growth in long sessions** | Procedural audio + particles are allocation-heavy by default | Pooled audio nodes/buffers and particles from day one; nightly 30-min soak with heap/node-count assertions as a hard gate |
| **Tick scheduler at scale** | Per-minute ticks across all accounts is a cost cliff | Hot/cold model with exact catch-up folds (Call #2); compute scales with concurrent attention, not account count; p99 tick alarm catches degradation early |
| **Voice erosion by future contributors** | "Just one toast" is how the product dies, per the PRD's own analysis | Copy linter with register declarations in CI; no banner/toast component exists in the design system; PRD non-goals section linked from CONTRIBUTING |
| **Notebook prose quality** | Template prose can read as generated, which breaks the observer fiction | Small hand-crafted template corpus with high slot specificity over a large generic one; seeded variation; editorial review of the full corpus; sparsity keeps exposure low and stakes per-entry high |
| **Magic-link deliverability** | Auth depends entirely on email arrival | Reputable transactional provider, domain auth (SPF/DKIM/DMARC), monitoring on send-to-consume conversion, matter-of-fact resend surface |
| **Timezone edge cases** | Travel/device disagreement makes day/night jump | Single canonical account timezone, most-recently-present device wins, 30-min lighting re-anchor; property tests around DST transitions in mood baselines |

---

## 16. Tuning-constant register (single source, server-side config)

So calibration has one home: drift signal weights and daily budgets; presence activity window (default 4 min); presence ping/flush cadences; mood transition weights and stickiness; weather frequencies; greeting class thresholds by absence band; offer cooldown (default 3 min/bird); listen-in ramp times and ambient floor; notebook bucket rate and noteworthiness thresholds; adoption age ladder; snapshot keepalive interval. Every value above marked "default" or "initial" in this plan lives here, is changeable without deploy, and is logged (value, not per-user) on change.

---

*End of plan. The deliverable is this plan; no product code accompanies it.*
