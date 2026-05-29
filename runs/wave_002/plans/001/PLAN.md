# Pocket Aviary — v1 Implementation Plan

This plan turns the Pocket Aviary PRD into an executable build for a frontier engineering team. It is opinionated where the PRD leaves room, and every defensible call is flagged as such with its reasoning. The organizing principle, inherited from the brief, is that **felt-aliveness is the product**: the server-side simulation tick, procedural audio, monotonic-toward-expressive drift, and the "notice, never announce" discipline are not features layered on a CRUD app — they are the architecture. Where a conventional choice would quietly betray that principle (a spinner, a "welcome back" toast, last-write-wins on personality, a recorded-audio fallback, a streak counter), the plan names the trap and routes around it.

A note on voice that the whole team must internalize: **two registers, one hard line.** The aviary, notebook, narration, captions, and offer prompts use the naturalist field-notebook voice (lowercase, present-tense, specific, bird-named, no "you", no announcement). Sign-in, account settings, sync errors, accessibility settings, unsupported-browser, and revoked-visit surfaces use matter-of-fact voice (normal capitalization, direct, says what happened and what to do). Any surface where the user engages the system *as a system* — identity, money, errors, settings — drops out of the naturalist register. This is encoded as a lint-able copy convention (see §10.4), not a per-PR judgment call.

---

## 1. Scope

### 1.1 In scope for v1

- **Aviary**: one canonical aviary per account; 2 starter birds, hard cap of 7; one horizontal single-screen scene (no pan/scroll/zoom); day/night anchored to user local time; rare ambient weather; continuous ambient micro-motion.
- **Bird engine**: hidden 5-trait personality vector per bird; monotonic-toward-expressive drift; fast-timescale mood; procedural call grammar; mood-shaped idle motion; bird-to-bird interaction; stable internal bird identity; user-assigned renameable names; ~6-species pool; age-gated new-bird offers.
- **Interactions**: return-greeting (absence-length, boldness, mood aware; procedurally varied; staggered when multiple); listen-in (gradual mix re-balance, never mute); offer (seed / song-fragment / still-pool, per-bird cooldown, mood-shaped reaction); settle (soft session end with 5s undo); field notebook (auto, sparse, read-only, naturalist prose); presence accounting (3-signal conjunction).
- **Accounts & sync**: single-user accounts; email magic-link auth (15-min expiry, single-use, per-device revocable session tokens, verified email change); synthetic UUID as the universal identifier; account export (on-demand JSON, emailed link); soft-delete 30 days then hard.
- **Server-side simulation tick**: ~1/min canonical advance; reads append-only event log; writes personality, mood, positions; runs whether or not a client is connected.
- **Multi-device sync**: property of the server-authoritative architecture, not a separate feature; additive server-authored deltas processed in event-log order; no last-write-wins on personality.
- **Social (single affordance)**: per-invite, opt-in, read-only ambient visits by email; revocable; 30-day invite expiry; silent visit log; off-by-default visit notification toggle. No co-presence.
- **Accessibility (first-class)**: naturalist screen-reader narration; reduced-motion as a *designed* surface (cross-fades, not "animations off"); procedural call captions; full keyboard navigation; WCAG AA contrast on all user copy.
- **Performance & observability**: <2MB gzipped initial bundle; <500ms time-to-first-bird on mid-tier mobile/4G; 60fps idle on a 5-year-old laptop; no memory growth over 30 min (CI-enforced); client-side WebAudio procedural synthesis with a silence+captions fallback; aggregate-only telemetry with a hard privacy boundary.

### 1.2 Explicitly out of scope (non-goals, enforced architecturally)

- **No native app.** Web only. We do **not** shape the data model or protocols around hypothetical native-client constraints.
- **No gamification of any flavor.** No achievements, streaks, levels, scores, badges, XP, rank, tier, green-dot calendars, "birds adopted: N", "days visited", or any visit-frequency surface — not even as a settings toggle. The notebook may observe the *aviary* ("pip greeted before wren today"); it may never observe the *user's behavior* ("you visited every day this week"). The aggregate metrics that would back such a feature are deliberately **not computed** (§9.2).
- **No Tamagotchi mechanics.** Birds never die, hunger, show distress, or have a decaying happiness meter. Neglect produces *ambient quietness*, never negative drift, never a guilt surface.
- **No social-network surfaces.** No profiles, follows, public feed, discovery, friend-of-friend, mutual visits, comments, leaderboards, or show-off rendering. The visit is the only social affordance and it is read-only.
- **No notification surface.** No push, no marketing email, no "your friend visited!" ping (host can opt into a quiet per-friend visit notification only).
- **Personality vector is never exposed numerically**, in any view, tier, or debug surface, ever. No toggle.

### 1.3 Defensible calls made where the PRD is intentionally open

These are decisions a frontier team can execute against; each notes that it is a build-time call.

- **Presence activity window**: 4 minutes of pointer/keypress recency, leaning long per the spec ("watching birds without moving is the actual product"). Calibrated in §5.6.
- **Tick cadence**: 60s nominal; jittered ±10s per account to avoid thundering-herd on the worker fleet.
- **Personality range**: each trait a float in `[0.0, 1.0]`; starter seeds drawn per-species from a narrow band (§4.2).
- **Drift calibration constants**: chosen so a "regular visitor" (~15 min/day) crosses an instrument-detectable threshold at ~7 days and a user-visible threshold at ~21 days (§5.3). These constants live in one config file and are the single most test-guarded numbers in the system.
- **Species pool size**: 6, including one nightjar-like nocturnal signature that remains active at night.
- **Mood set**: `wary, content, curious, drowsy, alert, settled` (`settled` is the evening/night low-energy state, distinct from the user's settle gesture but visually aligned).
- **Notebook sparsity target**: ≤1 entry/day for a regular visitor; rate-limited by a per-account cooldown and a salience threshold (§5.7).

---

## 2. Architecture

### 2.1 Service shape

A small set of services with a hard, named boundary between the **simulation plane** (per-account relationship state — private, never aggregated) and the **operational plane** (auth, edge delivery, aggregate telemetry).

```
                    ┌──────────────────────────────────────────────┐
   Browser  ──────► │  Edge / CDN                                   │
   (SPA)            │   • static bundle (<2MB gz, code-split)       │
                    │   • HTML + inlined first-snapshot (TTFB path) │
                    └───────────────┬──────────────────────────────┘
                                    │
                    ┌───────────────▼──────────────┐
                    │  API Gateway (stateless)      │  ── auth check, rate limit, routing
                    └───┬───────────┬───────────┬───┘
                        │           │           │
            ┌───────────▼──┐  ┌─────▼──────┐  ┌─▼────────────────┐
            │ Auth Service │  │ Snapshot   │  │ Event-Ingest     │
            │ (magic link, │  │ Service    │  │ Service          │
            │  sessions)   │  │ (reads     │  │ (append-only     │
            └──────┬───────┘  │  canonical │  │  write path)     │
                   │          │  state)    │  └────────┬─────────┘
                   │          └─────┬──────┘           │
                   │                │                  │
   ┌───────────────▼────────────────▼──────────────────▼───────────────┐
   │  SIMULATION PLANE (private; never read by analytics)               │
   │   • Postgres: accounts(email encrypted), birds, personality        │
   │     vectors, moods, positions, notebook entries, invites, visits   │
   │   • Append-only event_log (partitioned by account_uuid)            │
   │   • Tick Worker fleet (scheduled, server-authoritative writer of    │
   │     personality + mood + position)                                  │
   └────────────────────────────────────────────────────────────────────┘

   ┌────────────────────────────────────────────────────────────────────┐
   │  OPERATIONAL PLANE (aggregate-only; no per-account dimension)        │
   │   • Metrics pipeline (RED + perf + tick latency)                     │
   │   • Synthetic browser fleet (perf checks from geographies)           │
   │   • RUM ingest (page-load, first-bird, frame timing, audio errors)   │
   └────────────────────────────────────────────────────────────────────┘
```

The two planes share **no** data store and **no** ETL path. The simulation database is never connected to the analytics warehouse. This is the architectural form of the privacy commitment in `accounts_sync.md` — enforced by network policy and separate credentials, not by reviewer vigilance.

### 2.2 Client/server split (the load-bearing line)

- **Server owns canonical state and is the only writer of personality, mood, and authoritative position.** Clients never mutate simulation state directly.
- **Client renders snapshots and interpolates.** It writes *interaction events* (offer, listen-in start/end, settle, presence pings) into the append-only log via Event-Ingest. The tick consumes that log.
- This split is what makes "the aviary continues without the viewer" true, makes multi-device sync free (both clients read one record), and makes the no-last-write-wins rule structurally unreachable to violate (clients have no personality-write code path at all).

### 2.3 Technology choices (recommended, with rationale tied to budgets)

- **Frontend chrome (top bar, settings, auth, notebook list, offer tray)**: Preact + TypeScript. Preact for ~3KB runtime vs. React's ~45KB, defending the 2MB bundle. These surfaces are code-split out of the first-paint path.
- **Aviary scene rendering**: a thin custom renderer over a single `<canvas>` using **WebGL2** (2D-textured quads + a small shader set), with a Canvas2D fallback path for the day-one render that is cheap to draw. Rationale: 60fps on a 5-year-old laptop with 7 birds + parallax + leaf drift + reduced-motion cross-fades is comfortably inside WebGL2; a DOM/SVG-animated scene risks layout thrash and fails the runtime budget. Birds are small SVG-authored sprites baked into a compact texture atlas (§7.2).
- **Audio**: WebAudio API directly (no heavy audio library), procedural synthesis nodes per call (§8).
- **Backend services**: TypeScript on Node for Auth/Snapshot/Event-Ingest (shared type definitions with the client over an OpenAPI/`zod` schema — one source of truth for the wire format). **Tick Worker in Go** (recommended): the tick is CPU-bound numeric work across many accounts on a tight cadence; Go's goroutine model and predictable GC make the p99-tick-latency budget (≤5s alarm) easier to hold than a single-threaded Node worker. Acceptable alternative: a Node worker pool if the team prefers one language — flagged as a build call, not a hard requirement.
- **Datastore**: Postgres (primary) for accounts + bird/personality/mood/position + notebook + invites. Event log as a partitioned Postgres table (`event_log` partitioned by `account_uuid` hash, time-subpartitioned) for v1 scale; the ingest interface is written so it could move to a log system (Kafka/Redpanda) later without changing client contracts. Redis for session-token cache, magic-link nonce store, rate limiting, and short-TTL snapshot cache.
- **Edge**: CDN (Fastly/CloudFront-class) serving the static bundle and an HTML document with the **first state snapshot inlined** (§6.1), which is the single biggest lever on the <500ms time-to-first-bird budget.

---

## 3. Domain model overview (terms, kept exact)

Terminology is enforced in code and copy: **bird** (never creature/pet/character), **call** (never song/noise/chirp), **mood** (fast), **personality vector** (slow, hidden), **drift** (slow cumulative, monotonic-up), **presence** (3-signal conjunction), **listen-in**, **offer**, **settle**, **field notebook**, **visit**, **tick**. A shared `glossary.ts`/`glossary.go` exports the canonical enums so the wrong word can't be typed.

---

## 4. Data model

All identifiers are synthetic UUIDv7 (time-ordered, index-friendly). **Email appears in exactly one column, encrypted, on `accounts`, and nowhere else** — not in keys, not in logs, not in events, not in telemetry. This is the non-negotiable rule from `accounts_sync.md`.

### 4.1 `accounts`

| column | type | notes |
|---|---|---|
| `account_uuid` | uuid (PK) | universal identifier; the only account reference used anywhere downstream |
| `email_encrypted` | bytea | envelope-encrypted (KMS data key); the sole storage of email |
| `email_hash` | bytea | HMAC (keyed) for login lookup + per-email rate limiting without storing plaintext as a key |
| `created_at` | timestamptz | aviary age is derived from this (drives age-gated bird offers) |
| `status` | enum | `active`, `pending_deletion` |
| `deletion_requested_at` | timestamptz null | set on soft-delete; hard-delete job fires at +30 days |
| `visit_notify_enabled` | bool | default false |
| `pending_email_encrypted` | bytea null | new address awaiting verification; old email works until verified |

### 4.2 `birds`

| column | type | notes |
|---|---|---|
| `bird_uuid` | uuid (PK) | **stable identity for the life of the account**; never reassigned on rename, sync, species-pool change, or migration |
| `account_uuid` | uuid (FK) | |
| `species_id` | text | one of ~6 pool entries; defines silhouette, default palette, call-motif library |
| `name` | text | user-assigned, renameable; default suggested at adoption |
| `adopted_at` | timestamptz | |
| `personality` | jsonb | the 5-trait vector (below). Server-written only. |
| `personality_updated_at` | timestamptz | last tick that wrote a delta |
| `mood` | enum | current fast-timescale mood; persists across sessions |
| `mood_set_at` | timestamptz | for mood-timer transitions |
| `perch_zone` | enum | `front`/`middle`/`back` — a *signal*, never user-set |
| `last_greeter_rank` | smallint null | supports "pip greeted before wren today" notebook salience without a user-behavior metric |

`personality` shape (floats in `[0,1]`, **never exposed numerically to the user under any code path**):
```json
{ "boldness": 0.31, "social_warmth": 0.40, "vocal_frequency": 0.52,
  "plumage_saturation": 0.18, "curiosity": 0.44, "schema": 1 }
```
Seed values are drawn per-species from a narrow band (e.g. boldness centered on the species default ±0.05) so two starter birds of the same species still differ slightly and feel individual from day one.

### 4.3 `event_log` (append-only, the only client write target for simulation)

Partitioned by `hash(account_uuid)`, time-subpartitioned. **Immutable**: insert-only, no update/delete from the app path.

| column | type | notes |
|---|---|---|
| `event_uuid` | uuid (PK) | |
| `account_uuid` | uuid | partition key |
| `bird_uuid` | uuid null | null for aviary-wide events (e.g. settle) |
| `type` | enum | `presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `settle_undo` |
| `payload` | jsonb | e.g. offer kind (`seed`/`song`/`pool`); listen-in duration is derived from start/end pair |
| `client_ts` | timestamptz | client clock (untrusted; used only for ordering hints) |
| `server_ts` | timestamptz | authoritative ingest time; **ordering is by `server_ts` + `event_uuid`** |
| `tick_consumed_at` | timestamptz null | set when the tick has folded this event into a delta (idempotency / replay-safety) |

### 4.4 `personality_deltas` (audit + correctness)

Every tick-applied delta is recorded additively so drift is reconstructible and never silently lost:

| column | type | notes |
|---|---|---|
| `delta_uuid` | uuid (PK) | |
| `bird_uuid` | uuid | |
| `tick_id` | uuid | the tick run that produced it |
| `delta` | jsonb | per-trait non-negative increments (monotonic-up; see §5.3) |
| `inputs_summary` | jsonb | bounded numeric inputs (presence-seconds, listen-in-seconds, offer count) — **no qualitative text** |
| `applied_at` | timestamptz | |

The current vector on `birds.personality` always equals `seed + Σ deltas`. The delta table is the safety net behind "losing a personality vector is the worst failure": the vector can be rebuilt by replay, and conflict resolution preserves drift rather than arbitrating it.

### 4.5 `mood_history` (bounded)

Recent mood transitions per bird (ring-buffered, ~last N) for narration continuity and debugging; not user-visible.

### 4.6 `notebook_entries`

| column | type | notes |
|---|---|---|
| `entry_uuid` | uuid (PK) | |
| `account_uuid` | uuid | |
| `created_at` | timestamptz | |
| `prose` | text | naturalist, lowercase, present-tense, generated server-side (§5.7) |
| `salience` | real | the score that gated emission (internal) |

Read-only to the user. No edit/delete/annotate column exists — un-editability is structural.

### 4.7 `presence_windows` (derived, bounded)

The tick rolls `presence_ping` events into accumulated presence-seconds per account/day for drift input and for anonymized session-duration histograms (operational plane gets only the anonymized aggregate, never the per-account rows).

### 4.8 `sessions`

| column | type | notes |
|---|---|---|
| `session_uuid` | uuid (PK) | per-device session token id (token itself stored hashed) |
| `account_uuid` | uuid | |
| `device_label` | text | coarse UA-derived label for the revoke list ("Safari on iPhone") |
| `created_at`, `last_seen_at` | timestamptz | |
| `revoked_at` | timestamptz null | |

### 4.9 `magic_links`

Short-lived nonces (15-min expiry), single-use (`consumed_at`), stored hashed; per-email-hash rate limiting in Redis.

### 4.10 `invites` and `visits`

`invites`: `invite_uuid`, `account_uuid` (host), `visitor_email_encrypted`, `visitor_email_hash`, `token_hash`, `created_at`, `expires_at` (+30d), `revoked_at`, `used_at`. `visits`: append-only log of `invite_uuid`, `started_at`, `approx_duration_s` — the host's on-demand visit log. **Visitor events are never written to the host's `event_log`** (visitor attention must not drift the host's birds).

---

## 5. Simulation engine design

The engine is the product. It runs entirely server-side and is the only writer of personality, mood, and authoritative position.

### 5.1 The tick

A scheduled worker advances each account's canonical state on a ~60s cadence (jittered per account). One tick run, per account:

1. **Lease** the account (advisory lock / `SELECT ... FOR UPDATE SKIP LOCKED`) so exactly one worker ticks it; the lease makes the tick the sole writer even under a worker fleet.
2. **Read** unconsumed `event_log` rows for the account in `(server_ts, event_uuid)` order.
3. **Fold** events into bounded numeric inputs per bird (presence-seconds attributable to listen-in / general presence, offer counts by kind, settle markers).
4. **Compute drift deltas** (§5.3) — non-negative per trait — and append to `personality_deltas`; update `birds.personality = old + delta` atomically.
5. **Transition moods** (§5.4) using interactions, local time-of-day, ambient events, and personality.
6. **Advance positions / perch choices** (mood- and personality-shaped) and any active animation intents the snapshot will carry.
7. **Maybe emit a notebook entry** (§5.7) if salience clears the threshold and the sparsity cooldown allows.
8. **Mark** consumed events (`tick_consumed_at`), **write** new canonical state, release lease. Idempotent: re-running a tick on already-consumed events is a no-op.

The tick runs whether or not a client is connected. Accounts with no recent activity still tick (cheap path: time-of-day mood drift, possible drift-toward-quiet expression, no personality *increase* since presence is the dominant positive input). The catch-up after a long absence is bounded (coalesce elapsed time rather than replaying thousands of empty ticks) so a returning user's first tick is fast.

### 5.2 Why a low-pass filter, not an accumulator

Drift is a slow low-pass filter over presence-and-interaction signal so no single session moves a trait visibly. Per-tick delta for a trait is shaped like:

```
raw_signal_t   = w_presence * presence_seconds_in_window
               + w_listen   * listen_in_seconds (for that bird's warmth/vocal)
               + w_offer     * offer_curiosity_boldness_terms
target_t       = saturating_map(raw_signal_t)        // into [0,1]
delta_t        = alpha * max(0, target_t - current)  // MONOTONIC-UP: clamp at 0
```

`alpha` is small (the low-pass constant). The `max(0, …)` clamp is the **load-bearing asymmetry**: a low-activity tick yields delta 0, never negative. Neglect cannot reduce a trait. A bird that's ignored becomes *ambient* (it expresses its existing traits less in mood/position), it never becomes warier, quieter-by-trait, or less colorful.

### 5.3 Drift calibration (the most test-guarded numbers in the system)

All constants live in one `drift_config` file, versioned, with the `personality.schema` field so a recalibration is auditable. Calibration targets, expressed as tests:

- **Instrument-detectable at ~7 days**: a simulated "regular visitor" profile (≈15 min presence/day, occasional listen-in/offer) produces a cumulative trait change ≥ an instrument epsilon (e.g. +0.02 on the dominant trait) by day 7. CI runs this as a deterministic sim against `drift_config`.
- **User-visible at ~21 days**: the same profile crosses a "visible" threshold (e.g. +0.08, the point where mood/position expression changes a user would notice on look-back) by ~21 days.
- **No-Tamagotchi guard**: a "single intense session" profile (one 3-hour binge) must **not** cross the user-visible threshold — proving no single session moves a trait visibly.
- **Neglect guard**: a "two-week absence" profile produces **zero negative** trait movement; expression quiets via mood/position only.
- **Saturation guard**: offer-driven curiosity cannot saturate within one session (enforced jointly with the per-bird offer cooldown, §interactions).

Weights order (per PRD): presence ≫ listen-in > offers (curiosity/boldness) ; settle contributes only a clean presence-window close, no directional drift.

### 5.4 Mood transitions

Mood is an enumerated FSM per bird (`wary, content, curious, drowsy, alert, settled`) evaluated each tick. Transition probabilities are functions of:

- **Recent interactions** (offer just accepted → nudge toward `content`/`curious`; listen-in → mild `alert`/`content`).
- **Local time-of-day** (user timezone): `alert` early morning, `drowsy`→`settled` toward dusk/night.
- **Ambient events**: passing rain dampens vocal frequency and nudges toward calmer moods briefly; wind pushes some birds `alert`, others `wary`, short-lived.
- **Personality**: high-boldness birds resist `wary` on the same input; high-warmth birds re-enter `content` faster.
- **Bird-to-bird spread**: a `wary` mood in one bird raises the `wary` transition weight for nearby birds; chorus events (≥2 high-vocal-frequency birds calling in the same window) raise `content`/`alert`.

Mood **persists across sessions**: session-end mood is session-start mood, modulated by whatever ticks ran in between. There is no "snap to neutral" on tab open — the client simply renders the canonical mood from the snapshot. A bird that ended `drowsy` at dusk and ticked through the night is likely `settled`/sleeping by morning; one that ended `wary` likely softened toward `content` via the time-of-day signal.

### 5.5 Call-grammar runtime (server intent → client synthesis)

The server does **not** synthesize audio; it decides *call intent* and the client renders it procedurally (§8). Each species has a motif library (a small set of pitch/rhythm motifs). The tick (and the client's between-snapshot scheduler) selects motif, count, and timing-shaping from:

- the bird's **vocal_frequency** trait (more frequent calls when unobserved; readier chorus participation),
- current **mood** (drowsy → low, sparse; alert → brighter, higher),
- **ambient** (rain → dampened),
- **chorus context** (overlap with other calling birds).

The snapshot carries near-future call intents (motif id, planned onset, pitch/timing parameters, mood tag) so the client can schedule synthesis and interpolate smoothly between snapshots. **Recognizability is invariant**: a bird's motif identity (its species signature, fingerprinted by the bird's slow traits) is stable across mood and drift — Pip stays recognizably Pip by ear even as vocal_frequency drifts up. This recognizability ceiling is exactly why the 7-bird cap exists, and it is enforced as an audio-design constraint, not a runtime one.

### 5.6 Presence accounting (honest by construction)

A `presence_ping` is emitted by the client **only** when all three conditions hold simultaneously:

1. `document.visibilityState === 'visible'`, AND
2. the document has window focus (`document.hasFocus()`), AND
3. a `pointermove` or `keydown` occurred within the activity window (build call: **4 minutes**, leaning long — watching without moving is the product).

Pings are low-frequency (e.g. every 20–30s while the conjunction holds) and carry no payload beyond the event type and timestamps. The server attributes presence-seconds from ping cadence. **The laxer "tab is open" shortcut is explicitly forbidden** — it would count a laptop left open overnight as equal to an hour of watching and corrupt the drift signal population-wide, a silent failure no test would otherwise catch. We add a server-side sanity guard: implausibly continuous presence (pings spanning many hours with no gaps and no other event variety) is capped per day, defending the population drift signal against stuck or spoofed clients.

When the tab is hidden/blurred the client **stops emitting pings**, stops rendering (battery), and the simulation keeps ticking server-side. On return the client pulls a fresh snapshot — the aviary that *was running*, not a resumed freeze.

Settle and tab-close are both terminal and equivalent at the engine level: both end the presence window. No "you didn't settle" recovery surface, no penalty, no notification.

### 5.7 Notebook generation (sparse, specific, observer-of-the-aviary-only)

A candidate entry is considered at the end of a tick. A **salience score** ranks aviary moments (first-greeter-of-the-day changes, a long quiet stretch, a chorus event, first time a wary bird came to the front this week, a leaf-drift no bird looked up at). An entry is emitted only if salience clears a threshold **and** a per-account sparsity cooldown has elapsed (target ≤1/day for a regular visitor; the cooldown rises with recent activity to *preserve sparsity for very active users* — more visiting must not mean more entries).

Prose is generated from a **template-with-slots grammar keyed to bird names, species, mood, time-of-day, and the salient moment**, in naturalist voice (lowercase, present-tense, specific). Examples the generator must be able to produce:

> tuesday — pip greeted before wren today, first time this week.
> wren is fluffed against the cool air, watching the back perch. low calls only.

The generator is **forbidden** by construction from two things: (a) generic event-log phrasing ("session started at 7:43", "a bird greeted you", "vocal_frequency changed by 0.03"), and (b) **any observation of the user's behavior** ("you visited every day this week"). The line is hard: the notebook observes the aviary, never the user. A copy lint (§10.4) and a template allowlist enforce both. Entries are immutable and never archived; the user scrolls back indefinitely.

> **LLM vs. grammar — build call.** v1 uses a deterministic template/grammar generator, not a model call: it is testable, cheap, offline-safe, privacy-clean (no per-bird data leaves the simulation plane), and incapable of drifting into announcement voice. A generative model is explicitly deferred; if ever revisited it must run inside the simulation plane and never on aggregated cross-account data.

---

## 6. API surface

All endpoints are authenticated by session token except auth bootstrap and visit-by-token. Wire schema is one shared `zod`/OpenAPI definition. The account is always referenced by `account_uuid`; **email never appears in any path, query, log line, or response except the account-settings surface that intentionally shows the user their own address.**

### 6.1 State consumption (read path)

- **First snapshot inlined in HTML** (the TTFB lever): the edge document includes the current snapshot JSON so the client can place birds mid-action on first frame without a round-trip. For cold/uncacheable cases the client requests it directly.
- `GET /v1/aviary/snapshot` → current canonical snapshot: per-bird `{bird_uuid, species_id, name, mood, perch_zone, position, active_motion_intents, near_future_call_intents}`, aviary `{local_time_phase, weather, lighting_state}`, `snapshot_seq`. Kilobytes, not megabytes. Short-TTL edge-cacheable per session.
- Client pulls a fresh snapshot on: **visibility change** (tab visible again), **long render-frame gap** (laptop suspend/resume), and a **low-frequency keepalive** while visible. The client **interpolates** between snapshots (perch A → perch B rendered as smooth motion, never teleport) using the `snapshot_seq` and per-bird motion intents.

### 6.2 Interaction submission (write path → append-only log)

A single guarded ingest endpoint; clients have **no** personality-write path by design.

- `POST /v1/events` with one of: `presence_ping`, `listen_in_start`, `listen_in_end`, `offer {kind: seed|song|pool, bird_uuid}`, `settle`, `settle_undo`. Server stamps `server_ts`, validates offer **per-bird cooldown** (a few minutes; functional, not punitive — prevents curiosity-trait saturation), and appends. Listen-in duration is derived server-side from the start/end pair. The endpoint returns only an ack + (optionally) a freshened snapshot hint; it never returns personality numbers.

### 6.3 Auth & account

- `POST /v1/auth/request-link {email}` → always 200 (no account-existence oracle); emails a 15-min single-use magic link; per-email-hash rate limited. Copy is matter-of-fact.
- `GET /v1/auth/consume?token=…` → validates, invalidates the link immediately, issues a per-device session token, redirects into the aviary.
- `POST /v1/auth/logout`, `GET /v1/account/sessions`, `POST /v1/account/sessions/{id}/revoke`.
- `POST /v1/account/email/change` (verify-new-before-switch; old email works until verified).
- `GET /v1/account/export` → on-demand JSON snapshot (birds, names, **current** personality vectors, moods, notebook, settings) generated and emailed as a download link to the verified address. (The export is the one place a user's own vectors leave the system — to the user, about the user's own birds — which does not violate the never-shown-numerically UI rule.)
- `POST /v1/account/delete` (soft, 30-day window) and `POST /v1/account/recover` ("I changed my mind" on any signed-in page).

### 6.4 Birds

- `POST /v1/birds/{bird_uuid}/rename {name}` — name only; never touches personality/mood/call; `bird_uuid` is invariant.
- New-bird (3rd+) **age-gated offer**: surfaced by the server when `now - account.created_at` crosses pacing intervals (months, not visit count). `POST /v1/birds/adopt` accepts the offered species (server-selected from the pool; user names it; no catalog). Hard cap 7 enforced server-side.

### 6.5 Visit-invitation flow

- `POST /v1/invites {visitor_email}` → host creates a per-invite opt-in invitation; emails the visitor a one-time link; sets 30-day expiry. Defaults OFF (no invite exists until explicitly created).
- `GET /v1/visit?token=…` → visitor's **read-only ambient** session: serves snapshots exactly as the host sees them (same birds, same moods, same drift, **no show-off rendering**), with **no** write capability. Visitor events are never logged to the host's `event_log`; visitor presence never drifts the host's birds.
- `POST /v1/invites/{id}/revoke` → immediate: the visitor's **next snapshot pull** returns a matter-of-fact "visit no longer available" surface; unused links silently stop working.
- `GET /v1/account/visits` → host's on-demand visit log (visitor email, date, approx duration, outstanding invites). No badge, no push. `PUT /v1/account/visit-notify {enabled}` — off by default, not surfaced in onboarding.

### 6.6 Settings & accessibility

- `GET/PUT /v1/account/settings` — reduced-motion preference, captions on/off, audio on/off, visit-notify, privacy-policy link. All matter-of-fact voice.

---

## 7. Frontend rendering pipeline

### 7.1 Scene composition

A single full-viewport `<canvas>` (WebGL2) renders three planes: **background** (sky gradient keyed to local-time phase + soft foliage), **middle** (perches in three zones — front/middle/back — and the birds), **foreground** (occasional branch/leaf passing through, subtle parallax). Parallax is gentle (the scene is "a window on a real morning," not a layered showpiece). The scene is **responsive**: it compresses horizontally on narrow viewports and widens (more inter-perch space) on wide ones, **never cropping a bird out of frame** — a layout solver positions perch anchors as fractions of viewport width with min/max clamps.

### 7.2 Bird rendering

Birds are SVG-authored per species (6 silhouettes), baked at build into a compact **texture atlas** of pose frames (perch, preen poses, scan, head-tilt, weight-shuffle, calling, fly). Plumage saturation is a **shader uniform** (saturation/feather-detail multiplier) driven by the bird's `plumage_saturation` trait, so drift toward richer plumage is a cheap per-frame uniform change, never a re-rasterization. Bird position interpolates between snapshot perch anchors with eased motion.

### 7.3 Idle micro-motion (mood-shaped, continuous, never "paused")

A per-bird idle controller drives continuous micro-motion: preening, scanning, head-tilt toward sounds, weight-shuffle. The controller is **mood-shaped** so the user reads mood from motion *without a label*: `wary` perches further back and scans more; `content` preens; `curious` tilts toward sounds and watches drifting leaves; `drowsy` sits low, feathers fluffed; `settled` eyes-closed, low. There is **no status icon, tooltip, or label** for mood anywhere — if the user must be told what a bird feels, the affective contract is broken. Motion is procedurally varied (phase-randomized, parameter-jittered) so it never reads as a looped cycle.

### 7.4 Transitions and the first frame

- **First frame is the aviary, already in motion.** No spinner, no fade-from-static, no "wake up" animation, no entry sequence. The client places birds at their snapshot positions *mid-action* (mid-preen, one calling from the high perch, a leaf drifting) and starts rendering as if it had been rendering all along — falling directly out of the inlined snapshot (§6.1).
- **Slow-load state** (cold cache / slow link): a **quiet field** — soft sky color, one or two faint motion cues — **never a spinner** ("a spinner says machine; we are not selling a machine"). The quiet field reads as the aviary catching up.
- **Empty-aviary** (post-adoption, pre-first-bird): the same quiet field; the first bird then enters with a soft fly-in to its starting perch. After that the user never sees an empty aviary again.
- **Return-greeting** is rendered from a server-provided greeting intent that encodes **absence length** (glance for a coffee-break return; longer re-orientation/approach/longer call after days), **bird selection** honoring boldness + mood (bolder bird greets first; warier later or not at all today), and **procedural variation** (real variation, not 3 prerecorded variants in rotation). When multiple birds greet, the client **staggers** them by a randomized small offset (never a unison chorus-on-cue, which would announce the user's arrival).

### 7.5 Ambient ornaments

Leaves and feathers drift through at slow random intervals as **pure client-side rendering ornaments** (no per-leaf simulation state, not driven by the tick). They keep the scene alive between bird actions. Day/night palette shifts continuously, anchored to user local time; weather (rare rain/wind) is rendered as gentle overlays that also feed the mood inputs the server already computed.

### 7.6 Top bar chrome

A thin top bar above the scene with exactly four icons: account/settings, accessibility settings, field notebook, offer. **No UI chrome inside the aviary scene** (no buttons, badges, hover-tooltips, overlay icons, inline labels). The bar **fades nearly transparent after a few seconds of cursor stillness** and returns to full opacity on cursor/keyboard activity — a top bar always at full opacity reads as an app frame; a fading one reads as an ignorable thin layer.

### 7.7 Render-loop discipline (budgets)

`requestAnimationFrame` loop with a fixed simulation-interpolation timestep; **rendering halts entirely when the tab is hidden** (visibilitychange) — nothing to see, and it defends battery + the no-memory-growth budget. On resume, pull a fresh snapshot and resume interpolation from the new canonical state.

---

## 8. Audio pipeline

### 8.1 Procedural synthesis (non-negotiable; no recorded audio, ever)

Calls are synthesized client-side via WebAudio from each species' **motif library** — a small set of pitch/rhythm motifs combined and varied at runtime, with **personality-shaped timing and pitch** (vocal_frequency drives rate and chorus-readiness) and **mood-shaped** character (drowsy → low/sparse; alert → brighter). Looped recorded audio is forbidden because (a) the second time a user hears an identical call the spell breaks irrecoverably, and (b) stacking two recorded loops produces a phase-canceling artifact the ear catches — only runtime-varied procedural calls produce a *real* chorus. The bundle budget (<2MB) independently forbids carrying recorded audio at the needed variation.

### 8.2 Synthesis graph

Per active call: a small node graph (oscillator/wavetable + noise component for breathiness → envelope (ADSR) → band/formant filter → per-bird gain → spatial pan by perch zone → master). Motifs are sequenced by a sample-accurate scheduler reading the snapshot's near-future call intents and scheduling onsets ahead of the audio clock. **Buffers and nodes are pooled and reused** — no per-call allocation that isn't freed (a hard input to the no-memory-growth CI test). Bird calls pan subtly by perch zone (front closer/centered, back more ambient) to reinforce the proximity signal.

### 8.3 Chorus mixing

When ≥2 birds call in the same window, their independently-varied procedural calls mix at runtime into an emergent chorus. A master bus with gentle limiting prevents clipping when several high-vocal-frequency birds overlap. Per-bird **call recognizability is preserved** across the mix (distinct motif/timbre fingerprints) — the affordance the 7-bird cap protects.

### 8.4 Listen-in mix

Focusing a bird (click/tap/keyboard Enter) **gradually** raises that bird's gain and **gradually** lowers the others to ambient over a slow ramp (a re-balance, **never a mute** — others drop but never go silent; silencing would teach "soloable tracks", the wrong audio surface). Disengage (click the bird again, focus another, click empty space, move keyboard focus away, Escape) ramps back to ambient at the same slow rate. A hard cut is forbidden — the interaction must feel like *listening*, not *channel-switching*. Listen-in start/end events are sent to the log; their duration is a strong attention signal that drifts that bird's warmth/vocal_frequency.

### 8.5 WebAudio fallback

If WebAudio is unavailable (old browser, denied/blocked audio context, hardware issue), the aviary plays in **graceful silence with captions on by default**. There is **no recorded-audio fallback path** — the no-recorded-audio rule is unconditional. Silence-with-captions is a better fallback than canned audio. The audio context is created/resumed on first user gesture (browser autoplay policy) without any "click to enable sound" announcement chrome; if the gesture never comes, captions carry the experience.

---

## 9. Accessibility surfaces (first-class, designed — never a stripped fallback)

The stance: every alternate path delivers *the actual product that feels alive*, not a semantic-markup transcription of visual states. This ships **with** v1, not as a v1.1 fix — a late reduced-motion mode is "a v1 launch that quietly told reduced-motion users the product wasn't for them."

### 9.1 Screen-reader narration (naturalist running prose)

A live region (`aria-live="polite"`) receives **naturalist prose** generated from the same canonical state the visual reads — *not* a state list, *not* "Pip at perch 2", *not* "Wren mood: content":

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Generated by the same grammar engine as the notebook (shared voice module), so a user moving between aviary and notebook hears **one product**. Cadence is **slow** (~1 update / 30–60s at idle) to avoid flooding the SR queue, with a **priority bump** for user-initiated events (return-greeting on session start, offer reaction, settle) — but even prioritized updates are written as observations, never state transitions. Treating this as ARIA-label automation is explicitly the wrong feature.

### 9.2 Reduced-motion mode (a designed register, not "animations off")

Triggered by `prefers-reduced-motion` or an explicit settings opt-in. It is a **different rendering of the same aviary**: micro-motion becomes slow cross-fades between still poses (preen-poses cross-fade rather than animating frame-by-frame); flight becomes cross-fades between perches rather than animated paths; ambient leaf drift is removed; **ambient color shifts (day→evening) remain, slowed**. **Calls still play at full quality (or caption per audio settings); birds still drift; mood still changes; the notebook still notices.** It is calmer and slower — its own quiet aesthetic — not a broken-looking static scene. Implemented as an alternate render mode in the same pipeline (pose cross-fade compositor), sharing the same canonical state and audio path.

### 9.3 Call captions (procedural, runtime-generated)

Opt-in. Short naturalist prose descriptions of each call **in the bird's current mood**, generated **from the actual procedural call grammar at runtime** (not a fixed per-call string) so the caption matches what was played:

> a soft three-note rise   ·   a low trill, paused, low trill again   ·   a single sharp call from the back perch

Captions appear as small text near the calling bird, fading in/out with the call, in naturalist voice. Useful for audio-off, hearing differences, noisy environments, and the WebAudio-fallback path (where they default on).

### 9.4 Keyboard navigation

Tab cycles top-bar items; Tab into the scene focuses the first bird; arrow keys move focus between birds; **Enter triggers listen-in** on the focused bird; **Escape exits listen-in**; the offer affordance opens via a top-bar shortcut and is fully keyboard-navigable; settle is reachable from the top bar. Focus indicators are a **soft, high-contrast outline that reads against both bright and dim aviary states** (designer-specified exact treatment).

### 9.5 Contrast

All user copy — top-bar labels, settings, account/error surfaces, captions, visually-displayed narration — passes **WCAG AA** as a floor (design system specifies exact ratios per surface). The scene contains no user copy except the top bar, so the constraint applies chiefly to chrome.

---

## 10. Performance budgets and observability

### 10.1 Budgets (treated as gates, not guidelines)

| Budget | Target | How it's held |
|---|---|---|
| Initial JS bundle | **<2MB gzipped** at first paint | Preact for chrome; aviary renderer is small custom WebGL2; procedural audio (no recorded files); SVG/atlas assets; **aggressive code-splitting** of settings, accessibility settings, visit-invitation flow out of first paint. CI bundle-size check fails the build above budget. |
| Time-to-first-bird | **<500ms** on mid-tier mobile / 4G | **First snapshot inlined in the edge-served HTML**; bundle budget; render path draws the first bird before non-critical assets load; CDN-edge HTML + snapshot. |
| Idle motion | **60fps on a 5-yr-old mid-range laptop** | WebGL2 atlas rendering; pooled buffers; halt render when hidden. Runtime budget over a **30-min** session, not just minute one. |
| Memory | **No growth over 30 min** | Pooled audio nodes/buffers; notebook rows virtualized and dereferenced on scroll-out; bounded worker threads and audio contexts. **A real CI test**, not a guideline. |

### 10.2 Observability (aggregate-only; privacy boundary at the metric definition)

- **Operational metrics**: request counts, latencies, error rates (RED); **simulation-tick latency** with a **p99 > 5s alarm**; client RUM — page-load, first-bird-render, render-frame timing, audio-context error counts; anonymized session-duration histograms (**no per-account dimension**).
- **Synthetic perf fleet**: automated browsers run the aviary on a schedule from common geographies to catch first-bird and frame-timing regressions before users do.
- **Hard boundary**: **none** of this telemetry contains per-bird state or per-account interaction history. The metric *definitions* exclude per-account dimensions; the simulation database is never read by the analytics warehouse; ML training (if it ever exists) never receives per-bird fields. This is an architectural rule (separate planes, §2.1), not a policy hope.

### 10.3 Privacy enforcement points

Synthetic UUID everywhere; email encrypted in one column; per-bird events used **only** to drive that user's own simulation — never aggregated, never used for cross-user recommendation, never shared, never population-analyzed. The leaderboard/discovery refusal cascades here: because those metrics are never computed, there is no pipeline that could "just be exposed later."

### 10.4 Copy-voice enforcement (lint)

A build-time copy lint with two surface registries: **naturalist** (aviary/notebook/narration/captions/offer prompts) and **matter-of-fact** (auth/settings/errors/sync/accessibility-settings/unsupported-browser/revoked-visit). The lint flags announcement vocabulary ("welcome back", "achievement", "unlocked", "streak", "you've been here", "level"), flags any notebook/narration template that references *user behavior* rather than the aviary, and flags naturalist phrasing in matter-of-fact surfaces. This is how "notice, never announce" and the voice split survive every future PR.

---

## 11. Rollout

### 11.1 Build sequencing (what to land in what order)

1. **Simulation core first** (the risk concentrates here): data model, append-only event log, tick worker, drift function + `drift_config`, mood FSM — behind a headless harness with the calibration tests (§5.3) green *before any UI*. Drift correctness is cheap to get wrong and expensive to discover late.
2. **Auth + accounts** (magic link, sessions, synthetic-UUID discipline, soft-delete) — the boring-with-teeth layer.
3. **Snapshot read path + event write path + client interpolation** — prove server-authoritative sync and "no client personality write" end-to-end on two devices.
4. **Renderer**: scene, three perch zones, mood-shaped idle, first-frame-already-in-motion, top-bar fade, day/night, ambient ornaments.
5. **Audio**: procedural synthesis, chorus, listen-in mix, captions, WebAudio fallback.
6. **Interactions**: return-greeting (absence/boldness/mood-aware, staggered), offer (+cooldown), settle (+5s undo), notebook generation (sparse).
7. **Accessibility surfaces in lockstep with 4–6** (narration, reduced-motion mode, captions, keyboard nav) — *not* after. Gate launch on these.
8. **Social**: invite/visit read-only flow, revocation, visit log, off-by-default notify.
9. **Observability + synthetic fleet + perf/memory CI gates**, then launch.

### 11.2 Ramp

- Launch with the **2-bird** starter aviary for all new accounts.
- **Birds-per-aviary ramps by aviary age**, not engagement: 3rd-bird offer at a few months, growing toward 5–6 over a year, hard cap 7. Age gating is server-evaluated from `accounts.created_at`. The pacing deliberately never teaches "more attention earns more stuff."
- **Tick fleet scaling**: tick is per-account leased work; scale workers horizontally as account count grows; watch p99 tick latency (alarm at 5s) as the leading indicator.

### 11.3 Instrument from day one (aggregate-only)

First-bird-render timing, frame timing, tick latency, audio-context error rate, bundle size in CI, memory test in CI. **No** per-account engagement metric, **no** retention dashboard backed by per-bird data — those would require the very aggregation the privacy boundary forbids.

---

## 12. Risks

### 12.1 Drift calibration is wrong (highest-leverage risk)

Too fast → Tamagotchi-by-clicking; too slow → screensaver. **Mitigation**: the §5.3 calibration tests are CI gates run against `drift_config` on every change; the monotonic-up clamp is unit-tested directly; a "single intense session" test proves no single session moves a trait visibly; a "two-week absence" test proves zero negative movement. `drift_config` is versioned with `personality.schema` so a recalibration is auditable and reversible. **This is the number we guard hardest.**

### 12.2 Sync correctness / personality loss (worst-case, often invisible)

Losing or silently overwriting a personality vector is the worst failure and the one least likely to fail a naive test. **Mitigations**: server is the *only* writer (clients have no personality-write code path); additive deltas processed in `(server_ts, event_uuid)` order; per-account tick lease guarantees single-writer; `personality_deltas` makes the vector reconstructible (`seed + Σ deltas`) so a corrupted current value is recoverable; **no last-write-wins anywhere on personality**; stable `bird_uuid` invariant across rename/sync/species-pool change/migration — a bird is never "reset", "regenerated", or "swapped". Replaying already-consumed events is idempotent.

### 12.3 Audio uncanniness

A call heard twice identically, or a fake stacked-loop chorus, breaks the spell irrecoverably. **Mitigations**: procedural-only rule enforced (no recorded-audio path exists in the codebase); runtime per-call variation; chorus emerges from independent synthesis; per-bird recognizability fingerprinting validated by listening tests across mood/drift; the 7-bird cap protects the recognizability ceiling; silence-with-captions (never canned audio) as the only fallback.

### 12.4 Accessibility regression to a stripped fallback

The cheap path (label every state, ARIA the vector, "animations off") would ration the product's actual quality by sensory ability. **Mitigations**: narration and reduced-motion are *designed* surfaces sharing the same canonical state and voice module; accessibility ships in lockstep (§11.1 step 7) and **gates launch**; the copy lint (§10.4) blocks state-list narration vocabulary; reduced-motion has its own visual QA pass (cross-fade aesthetic, not broken-static).

### 12.5 "Notice, never announce" erosion

The most reliably-violated principle — a "harmless" welcome toast, a streak, a "your friend visited!" ping. **Mitigations**: the copy lint flags announcement vocabulary and user-behavior observations; code review checklist explicitly forbids toasts/banners/streaks/visit-frequency surfaces; the bird greeting is the *only* welcome surface; visit notifications are off-by-default and never surfaced in onboarding; non-goals are restated in the contributing guide so each reasonable-looking pitch to relax them meets a loud, pre-argued "no."

### 12.6 First-bird performance miss

Above 500ms the central conceit collapses into a visible load. **Mitigations**: inlined first snapshot in edge HTML; bundle CI gate at 2MB; first-bird render before non-critical assets; synthetic mobile/4G perf checks in the fleet; the slow-load state is a *quiet field*, never a spinner, so even a miss degrades gracefully rather than reading as "machine."

### 12.7 Presence-signal corruption

A lax definition (or a stuck/spoofed client) inflates drift population-wide, silently. **Mitigations**: strict 3-signal conjunction enforced client-side; server-side per-day presence cap against implausibly continuous pings; pings stop on tab hide; presence attribution is the single dominant drift input and is unit-tested against the calibration profiles.

### 12.8 Privacy-boundary erosion

An engineer reaching for email as a convenient key, or an "anodyne" cross-account drift dashboard. **Mitigations**: synthetic UUID is the only identifier anywhere (lint/grep gate against email in keys/logs); two physically separate planes with separate credentials; the simulation DB has no network path to the analytics warehouse; leaderboard/discovery metrics are never computed, so they cannot "just be exposed."

---

## 13. Summary of defensible build-time calls (single reference)

- Presence activity window: **4 min** (leaning long). · Tick cadence: **60s ±10s jitter**.
- Personality traits: **5 floats in [0,1]**; per-species seed bands. · Mood set: **wary, content, curious, drowsy, alert, settled**.
- Species pool: **6**, incl. one nocturnal signature. · Notebook: **≤1/day** for regular visitors, salience-gated + sparsity cooldown.
- Drift: low-pass filter, **monotonic-up clamp**; instrument-detectable ~7d, user-visible ~21d.
- Frontend: **Preact chrome + custom WebGL2 scene**; audio via **raw WebAudio**.
- Backend: **Node** for auth/snapshot/ingest, **Go** for the tick worker (alt: Node worker pool); **Postgres** + partitioned event-log table; **Redis** for sessions/nonces/rate-limit/snapshot cache.
- Notebook/narration generated by a **deterministic grammar**, not an LLM (privacy-clean, testable, voice-safe); LLM deferred.

The product is what's left after the subtractions. This plan builds the aviary that has been there all along, and refuses — structurally, not by good intentions — every "harmless" feature that would turn it into something else.
