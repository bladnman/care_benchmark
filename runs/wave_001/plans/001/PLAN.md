# Pocket Aviary — v1 Implementation Plan

This is an execution plan for a frontier engineering team. It interprets the PRD into buildable systems, names every defensible call made where the PRD left room, and is intended to be executable without further clarification. It does not restate the spec except where a constraint shapes an implementation decision.

Plan structure:

1. [Scope](#1-scope)
2. [Architecture](#2-architecture)
3. [Data model](#3-data-model)
4. [API surface](#4-api-surface)
5. [Simulation engine design](#5-simulation-engine-design)
6. [Sync model](#6-sync-model)
7. [Frontend rendering pipeline](#7-frontend-rendering-pipeline)
8. [Audio pipeline](#8-audio-pipeline)
9. [Accessibility surfaces](#9-accessibility-surfaces)
10. [Performance budgets and observability](#10-performance-budgets-and-observability)
11. [Rollout](#11-rollout)
12. [Risks](#12-risks)
13. [Ambiguities resolved by this plan](#13-ambiguities-resolved-by-this-plan)
14. [Workstreams and sequencing](#14-workstreams-and-sequencing)

---

## 1. Scope

### In v1

- **Aviary**: one horizontal scene per account, two starter birds, cap of seven, three perch zones, local-time day/night cycle, rare ambient weather, client-side ambient ornaments (leaves, feathers).
- **Bird engine**: hidden five-trait personality vector per bird, monotonic-upward drift driven primarily by presence-time, fast-timescale mood with cross-session persistence, procedural call grammar with per-bird recognizable signatures, mood-shaped idle motion, bird-to-bird interaction (call/response, mood spread, emergent chorus).
- **Interactions**: return-greeting (procedurally varied, absence-length- and boldness-shaped), listen-in (slow-ramp mix re-balance, never mute), offers (seed / song fragment / still pool, per-bird cooldown), settle (with 5-second undo), presence accounting (three-signal conjunction), field notebook (sparse, naturalist, read-only).
- **Accounts & sync**: magic-link email auth, per-device revocable sessions, email change with verification, synthetic UUID account IDs (email stored once, encrypted), server-side simulation tick as the single writer of canonical state, snapshot-pull clients, append-only interaction event log, account export (JSON via emailed link), soft delete (30 days) then hard delete.
- **Social**: single feature — read-only ambient visits via per-invite email links; revocable, 30-day expiry, silent visit log, opt-in (default off) visit notification toggle. Visitor presence never feeds the simulation.
- **Accessibility**: screen-reader running narration in naturalist prose (slow cadence, priority bump for user-initiated events), reduced-motion as a designed cross-fade rendering, procedural call captions generated from the grammar at runtime, full keyboard navigation, WCAG AA contrast on all user copy.
- **Performance**: <2MB gzipped initial bundle, <500ms time-to-first-bird on mid-tier mobile/4G, 60fps idle on a 5-year-old laptop sustained over 30-minute sessions, zero memory growth over 30 minutes (CI-enforced), client-side WebAudio synthesis, graceful-silence-plus-captions WebAudio fallback.
- **Observability**: synthetic browser fleet, aggregate-only RUM, simulation-tick latency alarms (p99 > 5s), hard telemetry/privacy boundary at the pipeline level.

### Out of v1 (binding non-goals)

Native apps; any gamification surface (streaks, scores, badges, levels, counters, visit calendars, "you've been here every day" in any disguise including notebook entries about user behavior); Tamagotchi mechanics (death, hunger, distress, decaying meters, negative drift); social-network surfaces (profiles, follows, feeds, discovery, leaderboards, comments, co-presence, chat, avatars); push/email/notification outreach about the aviary; payments; shared or multi-aviary accounts; customizable scenes; recorded-audio fallback; panning/scrolling/zooming; user-controlled perch placement; numeric personality exposure anywhere, ever.

These are enforced structurally where possible, not just by review: no per-account aggregate stats are computed that a leaderboard could later "just expose"; the notebook generator has no access to visit-frequency aggregates; the API has no endpoint that returns personality values to any client (including debug builds — see §5.8).

---

## 2. Architecture

### 2.1 Service shape

Three deployable units plus a static edge:

1. **Edge/CDN** — serves the HTML shell, JS/CSS bundles, and (critically) the bootstrap state snapshot for the time-to-first-bird budget (§10.2).
2. **API service** (stateless, horizontally scaled) — auth, snapshot reads, interaction-event writes, account management, notebook reads, visit/invite flows, export, deletion. Never writes personality or mood.
3. **Simulation service** (stateful workers over a shared DB) — the only writer of canonical aviary state. Runs the per-aviary tick, consumes the event log, applies drift deltas, transitions moods, schedules greetings, writes notebook entries.
4. **Auxiliary jobs** — magic-link/invite/export email sender, hard-delete reaper, invite expirer.

The API/simulation split exists because their failure and scaling modes differ: API load follows client traffic; simulation load follows account count and is steady. Keeping the simulation as the sole writer also makes the no-last-write-wins rule (§6.2) a property of deployment topology, not just code discipline.

### 2.2 Client/server split

- **Server owns**: personality vectors, moods, mood timers, perch assignments, greeting decisions, drift, notebook entries, day/night phase determination inputs, weather scheduling, bird identity, account state, visits.
- **Client owns**: rendering and interpolation, idle micro-motion synthesis between snapshots, procedural audio synthesis, ambient ornaments (leaves/feathers — explicitly stateless per PRD), caption text generation, screen-reader narration text generation, presence-signal detection, local input handling (listen-in mix ramps, settle lighting transition with its undo window).
- **Client never**: ticks the simulation, computes drift, mutates personality or mood, or persists any canonical state. The client is a renderer plus an event reporter.

The boundary rule of thumb used throughout: *if losing it would change the bird, it lives on the server; if losing it only changes this device's current frames, it lives on the client.* Listen-in mix state, the settle lighting transition, and ambient leaves are client-local; the fact that a listen-in or settle happened is an event the server learns about.

### 2.3 Render pipeline boundary

The client receives **semantic state** (bird X is at perch zone front, mood content, mid-preen, call scheduled with motif seed S) and turns it into pixels and audio locally. The server never sends coordinates-per-frame, animation frames, or audio. This keeps snapshots in the low kilobytes, makes reduced-motion a pure client rendering mode over identical state, and lets narration/captions/visuals all derive from one state object so they can never disagree.

### 2.4 Technology choices (defensible calls)

- **Language**: TypeScript end-to-end. Shared package for state types, the call-grammar definitions, and the prose-generation library (used server-side for notebook entries and client-side for narration/captions, guaranteeing one voice).
- **Server runtime**: Node.js for API + simulation workers. The tick math is small (five floats per bird, ≤7 birds); the workload is I/O-bound, not compute-bound. Postgres as the single canonical store.
- **Client scene rendering**: custom renderer on **Canvas 2D with layered offscreen canvases** (background sky/foliage layer, mid bird/perch layer, foreground ornament layer), not a game engine. Pixi/Three would consume a third of the 2MB budget for capability we don't need: ≤7 animated birds, subtle parallax, one screen. Bird art is procedurally-tinted compact vector parts (head/body/wing/tail per species silhouette) rasterized to sprite atlases at runtime, which is how plumage-saturation drift renders without shipping per-saturation art.
- **Chrome UI** (top bar, settings, notebook panel, auth, visit surfaces): Preact (~4KB) with aggressive route-level code-splitting; settings/account/visit bundles load on demand.
- **Audio**: WebAudio with an **AudioWorklet** synthesizer (graceful degradation chain in §8.6).
- **Transport**: HTTPS request/response only — snapshot pull + event POST. **No WebSockets in v1.** The PRD's pull triggers (visibility change, long frame gap, low-frequency keepalive) cover every freshness need at a ~1-minute canonical cadence; a push channel adds connection-state machinery the product doesn't need. Documented as a v1 simplification, revisitable if visit-revocation latency (next-pull) ever needs tightening.

---

## 3. Data model

Postgres schema, canonical store. All IDs are UUIDv7 (time-ordered, index-friendly). All timestamps UTC.

### 3.1 `accounts`

| column | type | notes |
|---|---|---|
| `account_id` | uuid PK | synthetic; the only identifier used anywhere downstream |
| `email_encrypted` | bytea | application-layer encrypted (AES-GCM, KMS-managed key); stored *only* here |
| `email_hash` | bytea unique | HMAC-SHA256 with a dedicated secret, for sign-in lookup without decryption |
| `timezone` | text | IANA zone, client-reported, updatable; drives day/night and mood time-of-day inputs |
| `created_at` | timestamptz | aviary age derives from this |
| `deletion_requested_at` | timestamptz null | soft-delete marker; reaper hard-deletes after 30 days |
| `visit_notifications_enabled` | bool default false | the one opt-in social toggle |

**Hard rule, enforced in CI**: no other table, log statement, message payload, partition key, or telemetry event may contain email. A lint rule bans `email` columns/fields outside this table and the transient mailer payloads; the mailer receives decrypted email at send time only and never logs it.

### 3.2 `sessions`

`session_id` uuid PK, `account_id` FK, `token_hash`, `device_label` (parsed UA family for the settings list), `created_at`, `last_seen_at`, `revoked_at` null. Magic-link tokens live in `magic_links` (`token_hash`, `account_id` or pending-email-hash, `expires_at` = issue+15min, `consumed_at` null; consumed-at set transactionally on first use). Rate limit: 5 link requests per email-hash per hour (defensible default; tune later).

### 3.3 `aviaries`

One row per account (1:1 in v1; separate table so multi-aviary never requires a migration of bird FKs). `aviary_id` PK, `account_id` FK unique, `created_at` (adoption-pacing clock), `bird_count`, `next_species_offer_at` timestamptz (precomputed by the tick from aviary age; see §5.7), `weather_state` jsonb (current event + scheduled next), `last_tick_at`, `tick_seq` bigint (per-aviary monotonic tick counter).

### 3.4 `birds`

| column | type | notes |
|---|---|---|
| `bird_id` | uuid PK | the stable identity; never reused, never regenerated |
| `aviary_id` | FK | |
| `species_id` | smallint | references the static species pool (code-defined, versioned) |
| `name` | text | user-assigned; renameable; no effect on anything else |
| `personality` | jsonb | `{boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity}` floats in [0,1] |
| `mood` | text enum | `wary \| content \| curious \| drowsy \| alert \| settled` |
| `mood_since` | timestamptz | |
| `perch_zone` | text enum | `front \| middle \| back` |
| `call_seed` | bigint | fixed at adoption; the per-bird signature seed for the call grammar (§8.2) |
| `adopted_at` | timestamptz | |
| `drift_daily_applied` | jsonb | per-trait drift applied in the current UTC day, for the daily cap (§5.3) |

Personality is **stored state, never derived**. No code path recomputes it from events; restores come from DB backups only, and the event log is not a rebuild source (rebuilding would itself violate "never recomputed from event logs"). Backup posture follows from this: continuous WAL archiving with point-in-time recovery, because a lost vector is a deleted relationship (§12.2).

### 3.5 `interaction_events` (append-only)

`event_id` uuid PK, `aviary_id` FK, `seq` bigint (per-aviary, assigned server-side at insert from a per-aviary sequence — the total order the tick consumes), `session_id` FK (device attribution for ordering sanity, never surfaced), `kind` enum (`presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `greeting_ack`), `target_bird_id` null FK, `payload` jsonb (e.g. offer type), `client_at` timestamptz, `received_at` timestamptz, `consumed_tick_seq` bigint null (set when a tick has processed it).

Events are never updated or deleted except by account hard-delete. Retention: raw events older than 90 days are dropped after consumption (drift has already been applied; the log is an input queue, not an archive — keeping it forever would contradict the privacy stance that interaction history isn't ours to keep indefinitely). Defensible call; revisit window with privacy review.

### 3.6 `presence_windows`

The tick coalesces presence pings into windows: `aviary_id`, `started_at`, `ended_at`, `seconds`. Used as the drift integrand and for absence-length on return-greetings. Raw pings deleted after coalescing (privacy minimization).

### 3.7 `notebook_entries`

`entry_id` uuid PK, `aviary_id` FK, `written_at`, `body` text (final prose, lowercase naturalist voice), `subject_bird_ids` uuid[]. Read-only by construction: no update/delete API exists. Paginated reads, unlimited scroll-back.

### 3.8 `invites` and `visit_log`

`invites`: `invite_id` PK, `aviary_id` FK, `visitor_email_encrypted` + `visitor_email_hash` (same one-place encryption discipline; the invite is the one other legitimate email holder, since the product must email the visitor), `token_hash`, `created_at`, `expires_at` (= created+30d), `revoked_at` null, `first_used_at` null.

`visit_log`: `visit_id` PK, `invite_id` FK, `started_at`, `approx_duration_seconds` (updated from visitor snapshot-pull keepalives; "approximate" per PRD). Visitor sessions are tokens scoped to `invite_id` with **read-snapshot-only** capability — enforced at the route layer: visitor tokens cannot reach the event-write, notebook, or account endpoints at all.

### 3.9 Species pool (static, in code)

Six species defined in the shared package: silhouette part-set, default palette, call-motif library, day/night activity profile. One species flagged `night_active` (the nightjar-like signature). Versioned data; adding species later never alters existing birds (species_id is fixed at adoption).

---

## 4. API surface

REST over HTTPS, JSON. Auth via session token (httpOnly cookie). All routes keyed by the session's account → aviary; no aviary ID ever appears in client-visible URLs (no enumerable resource space).

### 4.1 State reads

- `GET /api/aviary/snapshot` — the core read. Returns (~2–6KB gzipped):
  ```jsonc
  {
    "snapshot_at": "...", "tick_seq": 41023,
    "scene": { "day_phase": "morning", "phase_progress": 0.31,
               "weather": {"kind": "rain", "ends_in_s": 90} | null,
               "settled": false },
    "birds": [{
      "bird_id": "...", "name": "pip", "species_id": 3,
      "perch_zone": "front",
      "mood": "content",
      "expression": {            // render-facing projection — NEVER raw traits
        "approach_bias": "near", // quantized from boldness for perch micro-position
        "plumage_tier": 4,       // 1..6 quantized saturation tier for tinting
        "idle_profile": "preen-heavy",
        "activity": {"kind": "preening", "since_s": 22}
      },
      "call_plan": { "seed": 88231, "motif_ids": [2,5,2], "earliest_at": "...", "density": 0.4 },
      "greeting": {"kind": "two_note_call", "delay_ms": 800, "absence_class": "hours"} | null,
      "offer_cooldown_remaining_s": 0
    }],
    "narration_context": { /* compact facts the prose generator consumes, §9.2 */ }
  }
  ```
  **The personality vector never crosses this boundary.** The snapshot carries quantized, render-facing projections (`approach_bias`, `plumage_tier`, `idle_profile`, call density) computed server-side. This makes "never exposed numerically" an API property — there is no client build, debug flag, or devtools inspection that can recover trait floats, because they are not sent.
- `GET /api/aviary/notebook?cursor=` — paginated entries, oldest-available to newest, read-only.
- `GET /api/visit/:token/snapshot` — visitor variant: same snapshot shape minus `greeting`, `offer_cooldown`, and `narration_context` personalization; returns `410 {"surface":"visit_unavailable"}` after revocation/expiry.

### 4.2 Interaction writes

- `POST /api/aviary/events` — batched append: `[{kind, target_bird_id?, payload?, client_at}]`. Server assigns `seq`. Returns `{accepted: n}`. Presence pings are batched client-side (one POST per ~60s of continuous presence, plus a flush on `visibilitychange→hidden` via `navigator.sendBeacon`). Idempotency: client supplies a `client_event_id` UUID per event; server upserts on it, so retries never double-count drift inputs.
- `POST /api/aviary/settle` — convenience wrapper writing a `settle` event (kept as a distinct route so the client can fire it via sendBeacon on tab close *only when the user actually settled*; tab-close itself sends no settle).
- Offer submissions validate cooldown server-side (client renders the cooldown from the snapshot, but the server is authoritative: a cooldown-violating offer is accepted as an event with `payload.cooldown_suppressed=true` and contributes zero drift, so the client never needs an error surface inside the aviary — no chrome in the scene).

### 4.3 Auth and account

- `POST /api/auth/request-link {email}` → always 200 (no account-existence oracle); emails magic link; creates account lazily on first link consumption (adoption flow then runs).
- `POST /api/auth/consume {token}` → session cookie; 410 with matter-of-fact copy if expired/consumed.
- `GET/DELETE /api/sessions` — list/revoke device sessions.
- `POST /api/account/email-change {new_email}` → verification link to new address; switch commits only on consumption; old email valid until then.
- `POST /api/account/export` → enqueues export job; JSON snapshot (birds, names, **current personality vectors** — the export is the single named exception to numeric exposure, per the PRD's export contents list; it is a file the user requested about their own data, not a product surface; flagged for product sign-off in §13) emailed as an expiring download link to the verified address.
- `POST /api/account/delete` / `POST /api/account/restore` — soft-delete mark / "I changed my mind" restore within 30 days.
- `POST /api/account/timezone {iana_zone}` — client reports on session start and on detected change.

### 4.4 Visits

- `POST /api/invites {visitor_email}` → emails one-time link (rate-limited: 10 outstanding invites max per account — defensible anti-abuse default).
- `GET /api/invites` / `POST /api/invites/:id/revoke` — list outstanding/active; revoke takes effect on the visitor's next snapshot pull.
- `GET /api/visits` — the visit log (settings surface only).
- Adoption: `POST /api/aviary/adopt {names:[...]}` for starters (species chosen server-side); `POST /api/aviary/adopt-offered {name}` when a species offer is active; `POST /api/birds/:id/rename {name}`.

### 4.5 Voice rule at the API layer

Every error body carries `{surface, message}` with message written in matter-of-fact voice (these are system surfaces by definition). The client never invents error copy; it renders server-provided strings, so the voice boundary is centralized.

---

## 5. Simulation engine design

The simulation service is the heart. Design goals: deterministic per-tick computation, single-writer correctness, calibration knobs in one config file, and structural impossibility of the failure modes the PRD names (drift corruption, vector loss, symmetric drift, exposed numerics).

### 5.1 Tick scheduling

- Canonical cadence: one tick per aviary per 60s (`TICK_SECONDS=60`, calibration constant).
- Workers claim due aviaries with `SELECT ... WHERE next_tick_at <= now() FOR UPDATE SKIP LOCKED LIMIT N` — horizontally scalable, no per-aviary lock contention, at-most-one concurrent tick per aviary by construction.
- Each tick: load aviary + birds, read unconsumed events (`seq` order), compute, write birds + aviary + notebook candidates + mark events consumed, **single transaction**. A crashed tick rolls back wholly; events remain unconsumed; the next tick redoes the work. Tick is idempotent over its inputs.
- **Dormancy tiering (ops optimization, observably identical)**: aviaries with no presence window in 24h tick every 10 minutes; in 7 days, every 60 minutes. Because drift inputs are zero during absence (monotonic rule — neglect changes nothing) and mood/day-night transitions are deterministic functions of elapsed time, a coarse tick computes the *same* canonical state a fine tick would. On any client snapshot request for a dormant aviary, the API service enqueues an immediate catch-up tick and serves the post-tick state (adds ≤ tens of ms; well inside the snapshot path budget). This preserves "the aviary has been running" exactly while making the fleet cost scale with active users. The invariant — coarse and fine ticks produce identical state — gets a property-based test.

### 5.2 Tick computation order

1. Coalesce presence pings since last tick into `presence_windows`.
2. Consume interaction events in `seq` order → per-bird drift input accumulators + mood nudges.
3. Apply **drift** (§5.3).
4. Advance **weather** schedule (§5.5) and apply its mood modifiers.
5. Run **mood transitions** (§5.4) per bird, including bird-to-bird spread.
6. Update **perch assignments** (mood + boldness-projection driven; hysteresis so birds don't oscillate — a perch change requires the target zone to have been preferred for ≥2 consecutive ticks).
7. Update **call plans** (density + earliest-at per bird from vocal frequency, mood, day phase, weather).
8. Decide **greeting plan** if a presence window just opened after absence (§5.6).
9. Evaluate **notebook noteworthiness** (§5.9) and maybe write an entry.
10. Check **species-offer pacing** (§5.7).
11. Write everything; set `next_tick_at`.

### 5.3 Drift function

Per trait *t* per bird, per tick:

```
input_t   = Σ (weight_t,k × signal_k)            // signals from this tick's window
delta_t   = α_t × input_t × (1 - personality_t)   // low-pass + soft ceiling
delta_t   = min(delta_t, daily_cap_t - drift_daily_applied_t)
personality_t ← min(1.0, personality_t + max(0, delta_t))
```

- **Signals and weights (initial calibration, all tunable in `drift_config.ts`)**: presence-seconds (dominant; feeds all traits with trait-specific weights, plumage and social warmth highest); listen-in-seconds targeted at a bird (social_warmth, vocal_frequency for that bird); offer-accepted (curiosity), offer-made-near-bird (boldness); settle (no drift — mood signal only, ends presence window cleanly).
- **Monotonicity is structural**: the final `max(0, delta)` means no code path can decrease a trait. There is no decay term, no neglect penalty, anywhere. A code-review rule plus a property test ("for all event sequences, traits are non-decreasing") makes the asymmetric-drift rule unbreakable rather than observed.
- **Daily caps** are the anti-saturation and anti-gaming control: even a 10-hour presence day moves a trait at most `daily_cap_t` (initial: ~1.5% of range). Combined with the offer cooldown, a single session can never produce visible change.
- **Calibration targets as executable tests**: a simulated "typical user" profile (5 sessions/week, 10 min each, occasional listen-ins and offers) is run through the real tick code in CI. Assert: after 7 simulated days, every presence-fed trait has moved by ≥ the instrument-detectable epsilon (0.5% of range) and ≤ 5%; after 21 days, plumage tier (the quantized render projection) has crossed at least one tier boundary and at least one behavioral projection (`approach_bias` or `idle_profile` or call density band) has changed quantization bucket — i.e., *visible* drift is defined as "a quantized projection changed," which is exactly what the user can perceive. A "screensaver guard" test asserts the inverse bound (heavy 8-week use doesn't pin all traits to 1.0). These tests are the PRD's narrow band, pinned.
- Ambient quietness on neglect is **emergent, not stored**: greeting probability and call density are computed partly from *recent* presence windows, so an unvisited aviary greets less and calls at lower density without any trait moving down. Return after two weeks → quieter birds, identical personality.

### 5.4 Mood model

Mood is a per-bird state machine over `{wary, content, curious, drowsy, alert, settled}` (taking the PRD's "finalized in implementation" license: adding `settled` as the night/settle resting state).

- Transition evaluation each tick scores candidate moods from: time-of-day curve in the account's IANA timezone (alert weighting early morning, drowsy near dusk, settled at night), recent interaction nudges (offer accepted → content; listen-in → curious-leaning), ambient events (rain → vocal damping + mild wary/alert mix; another bird's wary → wary pressure on neighbors), and personality projections as transition resistances (high boldness raises the wary threshold; high curiosity lowers the curious threshold).
- **Inertia**: a mood persists ≥3 ticks unless a user-initiated event or alarm-spread overrides; prevents flicker the user would read as twitchy.
- **Daily-ish reset** is implemented as a slow pull toward the time-of-day baseline rather than a discrete reset, so no user ever observes a snap; overnight ticks naturally walk a bird from yesterday's mood to this morning's. Mood persists across sessions for free because it only ever changes in tick-time.
- Bird-to-bird: an alarm-class call (grammar can emit one when a bird enters wary) applies wary pressure to other birds scaled by their boldness resistance; two+ birds with high call density and compatible moods in the same tick window get a synchronized `chorus_window` in their call plans — the emergent chorus is scheduled server-side, rendered client-side.

### 5.5 Weather and ambient events

Server-scheduled (canonical — host and visitor must see the same rain): Poisson-ish scheduling targeting 2–3 rain events/week of 2–5 min and occasional wind, sampled per-aviary. Effects are mood-modifier inputs with built-in decay (≤15 min residue). Stored in `aviaries.weather_state` and included in snapshots.

### 5.6 Return-greeting decision

Computed server-side at the tick that observes a presence window opening (or synchronously during the snapshot-triggered catch-up tick for dormant aviaries, which is precisely the return moment):

- **Absence classes**: `minutes` (<30m), `hours` (<24h), `days` (≥24h). Class is an input to greeting form, not a surfaced number (no "gone X days" anywhere — the class never reaches user copy).
- **Greeter selection**: weighted lottery over birds — weights from boldness projection (dominant), social warmth, current mood (drowsy heavily down-weighted, wary down-weighted), and a small recent-greeter penalty so the same bird doesn't *always* greet first while the bolder bird still usually does. This makes "pip greeted before wren today, first time this week" a real, observable event the notebook can truthfully report.
- **Form selection**: per (absence class × greeter mood × boldness band) a weighted set of greeting kinds — glance-up-from-preen, head-tilt-and-step-forward, quiet two-note call, longer call with second-bird response. `days` class biases toward re-orientation forms (approach, longer call).
- **Secondary greeters**: if warmth-weighted lottery picks any, they're scheduled with randomized 600–2500ms staggers. Never simultaneous.
- The snapshot carries the greeting plan (kind, delay, motif seed); the client renders it. Variation is real: form weights × motif-grammar synthesis × stagger jitter means no two greetings are frame-identical, satisfying "never canned" without pre-recorded variants.

### 5.7 Species-offer pacing

Pure function of aviary age (never of visits, interactions, or drift): offer #3 at ~90 days, #4 at ~180, #5 at ~300, #6 at ~450, #7 at ~600 (calibration constants). The offer appears as a quiet naturalist surface in the user's flow ("a new bird has been about the aviary's edges lately" — copy in product voice, no badge, no notification); accepting runs the fly-in; declining leaves the offer open with no expiry and no re-prompt. Species selected server-side from pool members not yet in the aviary.

### 5.8 Numeric-exposure firewall

Trait floats exist in exactly three places: the `birds.personality` column, tick-worker memory, and the user-requested account export file. The snapshot projection layer (§4.1) is the firewall. Internal admin/debug tooling renders quantized tiers and drift-activity sparklines, not floats, so even internal screenshares don't normalize numeric personality. (Engineers can query the DB; the rule is about product and tooling surfaces.)

### 5.9 Notebook generation

Runs inside the tick. Two-stage: **noteworthiness scoring**, then **prose rendering**.

- Candidate events with base scores: first-greeter changes ("first time this week" detected against a rolling 7-day greeter record), mood streaks (a bird wary all morning), weather moments coinciding with bird behavior, chorus events, offer reactions with character (the wary bird that finally approached the pool), long quiet stretches, a species-offer bird's first day, night-active species calling late.
- **Sparsity budget**: token bucket per aviary, ~1 entry per 2–4 days steady-state, burstable to consecutive days only above a high score threshold. Active users hit the cap; the bucket guarantees sparsity regardless of activity (the PRD's explicit tuning requirement).
- **Hard content rule, enforced in the generator's input schema**: the scorer's candidate set is *only* aviary observations. User-behavior facts (visit counts, frequency, streak-like patterns, session times) are not in the candidate vocabulary, so "you visited every day this week" is unwritable, not just unwritten.
- Prose rendering uses the shared template-grammar prose library (§9.2): slotted naturalist templates with synonym pools and structural variation, lowercase, present tense, bird names, concrete detail from the actual tick state. Every template is reviewed against the style samples; no numerals for traits ever appear in the vocabulary.

---

## 6. Sync model

### 6.1 Single canonical state, snapshot pull

One Postgres row-set per aviary is the truth. Every client (and visitor) renders `GET snapshot` results and interpolates. Pull triggers (per PRD): tab `visibilitychange → visible`, detected long frame gap (laptop resume: `rAF` delta > 5s), and a keepalive while visible at 45–60s (just under tick cadence, jittered to avoid thundering herds). Multi-device sync is therefore a non-feature: two signed-in devices read one record and are coherent within one pull interval.

### 6.2 No last-write-wins — concretely

- Clients have **no write path to bird state**. The events endpoint appends; the settle endpoint appends; rename and settings hit account-scoped columns that aren't simulation state. There is no `PUT /birds/:id` of any kind.
- Events get a server-assigned per-aviary `seq` at insert; the tick consumes strictly in `seq` order inside one transaction. Two devices interleaving sessions produce one interleaved log and one drift outcome — the "phone overwrites the laptop's morning drift" failure is unrepresentable because no absolute values are ever submitted.
- Idempotent event ingestion (`client_event_id` upsert) makes client retries safe; at-least-once delivery + dedup = effectively exactly-once drift input.
- `SKIP LOCKED` claiming + single-transaction ticks give at-most-one writer per aviary per tick. Personality writes happen at exactly one code site, in the tick, asserted by a DB role: the API service's Postgres role has no UPDATE grant on `birds.personality/mood/perch_zone`. The no-LWW rule is thereby enforced by the database's permission system, not convention.

### 6.3 Conflict surfaces

The only user-visible "conflicts" are auth-shaped (expired/replayed magic link, timed-out session, load failure) and use the PRD's matter-of-fact copy verbatim. Stale-render cases (e.g., user offers a seed milliseconds before a snapshot shows the bird moved) are resolved by the server-authoritative event semantics: the event is recorded against true state; the client's local reaction rendering proceeds from its current view; the next snapshot reconciles. At a 1-minute canonical cadence with client interpolation, divergence is cosmetic and self-healing within one pull.

### 6.4 Visitor sync

Visitors pull the same snapshot pipeline with a restricted projection and a 60s keepalive that also updates `visit_log.approx_duration_seconds`. Revocation = token check on every pull → `410` + matter-of-fact surface. Visitor pulls write **nothing** to the event log — enforced again by route capability and by the visitor DB role having no INSERT on `interaction_events`.

---

## 7. Frontend rendering pipeline

### 7.1 Boot path (the <500ms contract)

1. HTML shell from CDN edge includes: critical CSS, a tiny inline boot script, and — for returning sessions — an **edge-cached bootstrap snapshot** inlined or fetched-first from an edge endpoint that caches each aviary's latest snapshot (written through by the simulation service on every tick; private, session-keyed cache).
2. Boot script paints the **quiet field** immediately (gradient sky tuned to local clock time — computable before any network), then the core bundle (scene renderer + state client; everything else split out) hydrates.
3. First bird draws as soon as core bundle + snapshot are both present — target ≤500ms on mid-4G. Birds render **mid-action from snapshot state**: activity kind + `since_s` seeds the animation phase, so a bird 22s into preening appears 22s into a preen cycle. No entry animation, no fade-from-static, no spinner anywhere in the product (the quiet field is the only pre-state, and it's also the empty-aviary state between adoption and first fly-in).
4. Audio context initializes lazily on first user gesture where the browser requires one (§8.5); visuals never wait on audio.

### 7.2 Scene composition

Three offscreen-canvas layers composited to one visible canvas per frame:

- **Background**: sky gradient + soft foliage; redrawn only on day-phase progress steps (~every few seconds) and weather changes — effectively static between, near-zero per-frame cost.
- **Midground**: perches + birds. Birds are part-based sprites (per-species silhouette parts, runtime-tinted for plumage tier, rasterized once per tier to a small atlas). Skeletal-lite animation: head/body/wing/tail transforms, not frame-flipping, so idle motion is continuous and parameterizable by mood.
- **Foreground**: ornament pass — leaves, falling feather, occasional foreground branch; pure client-side, Poisson-scheduled, stateless. Subtle parallax: background 0.85×, foreground 1.1× of a ±4px ambient camera sway with minutes-long period.

Responsive: scene scales to viewport preserving a min/max aspect band; perch x-positions are layout-percentage-based so narrow viewports compress spacing without ever cropping a bird (layout solver clamps bird positions inside the viewport as a hard constraint).

### 7.3 Idle micro-motion and interpolation

- A per-bird **behavior interpreter** runs client-side between snapshots: given mood + idle_profile + activity, it sequences micro-motions (preen, scan, head-tilt-toward-sound — wired to actual scheduled call events from other birds, so tilts are causal, not random; weight-shuffle; fluff) from per-mood weighted pools with personality-projection-shaped frequencies. Smoothed noise (precomputed permutation-noise tables; no `Math.random()` in the per-frame path) drives the continuous component so motion never loops detectably.
- Snapshot deltas (perch change, mood change, new activity) are **interpolated**: perch changes render as a short flight arc (or cross-fade in reduced motion); mood changes ramp the idle-pool weights over ~10s rather than switching instantly. A bird never teleports and never snaps posture.
- Greeting plans from the snapshot are executed by the same interpreter with their server-assigned staggers.
- Settle: client runs the slow evening lighting ramp immediately on trigger (with the 5s any-click undo reversing it), posts the settle event (sendBeacon-safe), and the server marks the aviary settled for canonical state/visitor view.

### 7.4 Frame loop discipline

Single `requestAnimationFrame` loop; per-layer dirty flags; background tab → loop suspended entirely (no rendering while hidden, per PRD; presence accounting also stops by definition). Frame budget instrumented in dev builds; any subsystem exceeding its slice (birds 4ms, ornaments 1ms, composite 2ms on the reference laptop) fails the perf CI (§10.4). All per-frame allocations forbidden in the hot path (object pools for motion states and ornament particles) — this is the memory-flatness rule made structural.

### 7.5 Top bar and chrome

Preact island above the canvas: account/settings, accessibility settings, notebook, offer affordance — nothing else. Fade-to-near-transparent after 4s of pointer/keyboard stillness; restore on activity or on any chrome element receiving keyboard focus (focus must never land on an invisible control). Notebook and offer open as quiet panels overlaying the scene edge; settings surfaces are code-split routes in matter-of-fact voice.

### 7.6 Presence detection (client side of §3.6)

A presence sampler evaluates every 10s: `document.visibilityState === 'visible'` AND `document.hasFocus()` AND last `pointermove|keydown` within the activity window (**initial value 5 minutes**, config-served so calibration needs no client release; PRD says lean long because motionless watching is the product). While true, accumulate; emit batched `presence_ping` events (60s granularity) and flush on hide/close via sendBeacon. All three signals are independently testable in unit tests with synthetic DOM events.

## 8. Audio pipeline

### 8.1 Architecture

`AudioWorklet` synthesizer node (custom DSP in the worklet: 2-oscillator FM voice + noise-shaped transients + amplitude/pitch envelopes per syllable) → per-bird `GainNode` → chorus bus (gentle compressor) → ambient bed bus (very quiet procedural air/leaf-rustle bed) → master gain. Voices are a fixed pre-allocated pool of 8 (7 birds + 1 ambient overlap slack); zero per-call allocation, satisfying the memory-flatness rule in the audio path.

### 8.2 Call grammar and signature

Per species: a motif library (5–8 motifs: syllable contours — rises, trills, pairs, single sharp notes) + a grammar (probabilistic FSM over motif sequences with mood-conditioned transition weights). Per bird: the immutable `call_seed` (fixed at adoption) derives the bird's **signature parameters** — base pitch offset, formant tilt, characteristic inter-syllable gap, preferred motif bigrams. Per call: the server-provided call-plan seed feeds a seeded PRNG for the runtime variation (micro-pitch jitter, syllable count, timing humanization).

The layering is the recognizability mechanism: **signature parameters never change** (Pip's call stays Pip's through mood and drift), mood and vocal-frequency drift shape *when/how often/how bright*, and per-call randomness guarantees no two renditions are identical. Seeded variation also means a given call plan renders the same on two devices pulling the same snapshot — host and visitor hear the same aviary.

### 8.3 Scheduling and chorus

Client schedules calls from each bird's snapshot call plan (density + earliest-at + chorus windows), using WebAudio clock-domain scheduling (lookahead scheduler pattern) for sample-accurate timing. Chorus windows from the server let 2+ birds' grammars interleave call/response within a shared window — a real mixed chorus of independent procedural voices, never layered loops. One bird's call event also feeds the behavior interpreter (head-tilts toward sound) and the captions/narration generators.

### 8.4 Listen-in mix

Listen-in engage: focused bird's gain ramps to +6dB-relative over **2.5s** (exponential ramp); other birds ramp to −12dB (audible ambient, never −∞) over the same window; ambient bed unchanged. Disengage (re-click, other-bird focus, empty-space click, focus loss) reverses with the same ramp. Pure client-local mix state; `listen_in_start/end` events report it for drift.

### 8.5 Autoplay policy reality

Browsers block audio before a user gesture. Handling: page loads with visuals fully alive and audio context suspended where required; the first user gesture (any click/keypress, including simply focusing the page in browsers that allow it) resumes the context with a slow 2s fade-in of the ambient bed and call bus — reading as "you tuned in," not "audio turned on." No "click to enable sound" banner (announcement-shaped); a quiet first-run hint lives in accessibility settings copy. Calls scheduled while suspended are dropped, not queued (no burst on resume).

### 8.6 Fallback ladder

1. AudioWorklet available → full synthesis.
2. No worklet (old Safari edge cases) → same DSP on a `ScriptProcessorNode` shim at reduced polyphony (4 voices) — same procedural calls, slightly higher latency tolerance (calls aren't rhythm-critical).
3. No usable WebAudio / context permanently denied / hardware failure → **graceful silence with captions auto-enabled** (per PRD), plus a one-time matter-of-fact note in accessibility settings explaining why sound is off. No recorded audio exists anywhere in the product, including this path.

## 9. Accessibility surfaces

Accessibility ships in the same milestones as the features it serves (every workstream in §14 includes its accessibility surface in its definition-of-done). Nothing here is post-launch.

### 9.1 Shared prose engine

One TypeScript library (`@aviary/prose`) generates all naturalist text — notebook entries (server), screen-reader narration (client), call captions (client), offer/adoption copy. Slotted template grammars with synonym pools, structural variation, and a hard style contract enforced by tests: lowercase product voice, present tense, no exclamation marks, no second person in observational text, no numerals for any bird property, banned-word list (`achievement`, `streak`, `level`, `score`, `welcome back`, …) asserted over every template at build time. One library = one voice across every surface, including for a screen-reader user moving between aviary and notebook.

### 9.2 Screen-reader narration

- Implementation: a visually-hidden ARIA live region (`aria-live="polite"`) fed by a narration scheduler. The scheduler consumes the same client state the renderer reads (snapshot + behavior interpreter + call events) and the snapshot's `narration_context` (compact facts: day phase, weather, per-bird activity summaries).
- Cadence: one idle prose update per 30–60s (jittered). User-initiated events (return-greeting at session start, offer reactions, settle acknowledgment) preempt with priority but remain observations ("pip hops down to look at the seed, head low"), never state transitions. Queue discipline: max one pending idle narration; a new one replaces it (never backlog the SR queue).
- The aviary canvas gets `role="img"` with a slowly-updated `aria-label` summary for first contact; the live region carries the running narration; birds are real focusable DOM elements positioned over the canvas (§9.4), each with a naturalist `aria-label` ("pip — a small grey bird on the front rail, preening") refreshed on mood/activity change.

### 9.3 Reduced-motion mode

Triggered by `prefers-reduced-motion` or the settings toggle (toggle wins when explicitly set). It is a **second render mode of the same interpreter**: the behavior interpreter's outputs map to a pose-graph per species (4–6 stills per activity); rendering cross-fades between poses over 1.5–3s instead of continuous skeletal motion. Flight transitions → cross-fade between perch positions. Ornament pass disabled. Day/night color ramps retained, slowed (and implemented as opacity/color fades, never positional motion). Top-bar fade replaced by instant-but-gentle opacity steps. Audio, captions, drift, notebook, greetings: identical. The pose art and fade curves get the same design attention as the animated mode — it's a register, not a removal; QA includes a dedicated reduced-motion review pass per milestone.

### 9.4 Keyboard navigation and focus

Per PRD map: Tab traverses top bar → first bird; arrow keys move between birds (DOM order = left-to-right scene order, maintained as birds move); Enter = listen-in toggle on focused bird; Escape = disengage; offer panel opens via top-bar shortcut and is a standard focus-trapped dialog; settle reachable in top bar (its 5s undo also keyboard-accessible: any key with aviary focus reverses). Focus indicator: soft 2px high-contrast halo with a dual-tone outline (light inner/dark outer) so it reads against dawn, midday, and night scenes — exact treatment with the visual designer, dual-tone is the engineering constraint that guarantees contrast in all lighting states.

### 9.5 Captions

Opt-in via accessibility settings (auto-on in the no-WebAudio fallback). Caption text generated at call-render time from the *actual* synthesized parameters (motif sequence, syllable count, pitch contour) through `@aviary/prose` — "a low trill, paused, low trill again" describes what just played, by construction. Rendered as small text fading in/out near the calling bird, WCAG AA contrast via an adaptive soft backing scrim sampled against the local scene luminance.

### 9.6 Contrast

All chrome, settings, errors, captions, and visually-displayed narration pass WCAG AA minimum (design system owns exact ratios). CI runs axe-core + custom contrast checks against light/dark/dawn/night scene states.

## 10. Performance budgets and observability

### 10.1 Bundle budget (<2MB gzipped initial)

Allocation (gzipped): core renderer + behavior interpreter 350KB; audio worklet + grammar/DSP 200KB; species art parts 450KB; prose library + templates 100KB; state client + boot 100KB; Preact chrome island 50KB; fonts 80KB; headroom 670KB. Enforced by size-limit in CI per-chunk and total; any PR exceeding a chunk budget fails. Settings/account/visit/notebook-panel and the adoption flow are split chunks outside the initial budget.

### 10.2 Time-to-first-bird (<500ms, mid-tier mobile, 4G)

Critical path: edge HTML (with critical CSS + boot inline) → quiet field paint (<100ms) → core chunk + edge-cached snapshot in parallel → first bird (<500ms). Tactics: snapshot written through to edge KV on every tick so the read is an edge hit, not an origin round-trip; core chunk preloaded via header; species art for *this aviary's* birds inlined into the snapshot edge object as compact part-vectors so the first bird never waits on an art request; fonts non-blocking. Measured in CI on throttled Moto-G-class emulation and in synthetic fleet; budget alarm at p75 > 500ms.

### 10.3 Runtime budgets

60fps sustained on the reference laptop (5-year-old mid-range; CI uses a throttled-CPU Chrome profile pinned to that benchmark) across a scripted 30-minute session including listen-ins, offers, weather, and a settle. Memory flatness: same scripted session asserts heap (after forced GC) within ±5% of the 5-minute mark at the 30-minute mark; audio voice pool, motion-state pools, and notebook-panel virtualization (entries release references on scroll-out) are the design features that make it pass. Both run nightly in CI and block release.

### 10.4 Observability

- **Synthetic fleet**: automated browsers from 5+ geographies running boot + 10-minute idle + interaction scripts every 15 minutes; record TTFB, time-to-first-bird, frame timings, audio-context errors, caption/narration generation errors.
- **RUM (aggregate-only)**: page-load and first-bird timings, render-frame histograms, audio errors, API latencies, JS error counts — dimensioned by browser/geo/device-class only. **No account dimension, no bird state, no interaction history.** Enforced in the pipeline: the telemetry SDK's event schema has no account/bird ID fields (typed out of existence), telemetry egress runs through a schema-validating proxy that drops nonconforming events, and the analytics warehouse has no network path or credential to the simulation database. The privacy line is infrastructure, not policy.
- **Server**: tick latency histograms (p99 alarm at 5s per PRD; page at sustained p95 > 2s as the early-warning), tick backlog depth (due-but-unticked aviaries), event-log consumption lag, snapshot latency, auth/email job health, edge snapshot write-through staleness.
- **What we deliberately don't measure**: per-account engagement, retention cohorts keyed to interaction behavior, drift distributions per account, visit-frequency analytics, any funnel over presence. Operational health only. The absence is load-bearing (it's what makes leaderboards/streaks structurally hard to ever add) and is documented in the privacy policy copy.

## 11. Rollout

1. **M0 — engine-on-rails (internal, ~weeks 1–6)**: simulation service + tick + drift + mood with the calibration test harness; snapshot/event API; CLI/debug renderer (quantized projections only). Exit: calibration CI green (1-week instrument drift, 3-week visible drift, screensaver guard, monotonicity property test).
2. **M1 — living scene (internal alpha, ~weeks 5–10)**: canvas pipeline, idle motion, day/night, boot path with quiet field, presence sampler, audio synthesis + listen-in, captions. Team accounts run real multi-week aviaries — drift calibration needs calendar time, so alpha starts as early as the engine allows; this is the schedule's critical path.
3. **M2 — full surface (closed beta, invite-only, ~weeks 9–16)**: adoption flow, notebook, offers, settle, greetings end-to-end, accounts/auth/sync hardening, reduced-motion + narration + keyboard nav complete, account export/delete. Beta cohort ~200 accounts across timezones and device classes; includes screen-reader and reduced-motion users *recruited deliberately*, not found accidentally. Three+ weeks of beta calendar time minimum (one full visible-drift period).
4. **M3 — GA (~week 18+)**: visits feature enabled (it rides the snapshot pipeline, lowest-risk-last), perf budgets green for 2 consecutive weeks in synthetic + RUM, accessibility audit (external) passed, privacy review of telemetry schemas signed.
5. **Bird-count ramp**: GA accounts all start at 2 birds; the species-offer pacing means no account reaches 3 birds before ~day 90, which gives a natural production ramp of chorus complexity (audio mix and recognizability at 4–7 birds get long-bake validation on internal/beta aviaries that we seed with age-advanced clocks in staging — staging-only clock skew, never production accounts).
6. **Instrumented from day one**: tick latency + backlog, calibration-drift canaries (synthetic accounts in production running scripted presence; their drift trajectories are the live regression check on the narrow band), time-to-first-bird, audio error rates, memory-flatness in the synthetic fleet, magic-link delivery success.

Launch-blocking checklist tied to PRD invariants: no-spinner audit; no-toast/no-announcement audit over every surface; banned-word scan over all copy; numeric-exposure scan (no trait floats in any payload — contract test against every API route); monotonic-drift property test; LWW-impossibility check (API role grants); voice review of notebook/narration/caption corpora against the style samples.

## 12. Risks

### 12.1 Drift calibration misses the narrow band (likelihood: high; severity: product-defining)

Too fast → Tamagotchi; too slow → screensaver. Mitigations: calibration constants isolated in one config; CI simulation harness pins both bounds; M1 alpha starts the calendar clock early; production canary accounts watch live trajectories; config is server-side so retuning needs no client release. Residual risk: real-user presence distributions differ from the simulated profile — the beta's primary research question, with a planned mid-beta recalibration window.

### 12.2 Personality-vector loss or corruption (likelihood: low; severity: worst-possible, silent)

A reset bird fails no test and betrays the user weeks later. Mitigations: PITR/WAL backups with restore drills; tick transactionality (no partial writes); a nightly invariant checker (traits in range, non-decreasing vs. yesterday's snapshot, bird_id continuity) that alarms on any violation; personality column changes audited to an internal append-only audit log (tick-internal, never user-visible) so any anomaly is reconstructable.

### 12.3 Sync correctness erosion (likelihood: medium without structure; low with)

The dangerous version is a future convenience endpoint that writes bird state. Mitigations: DB-role enforcement (API role cannot UPDATE bird state), single-writer topology, idempotent event ingestion, property tests over interleaved multi-device event sequences. Residual: event `client_at` clock skew — mitigated by using server `received_at`/`seq` for all ordering and treating `client_at` as advisory.

### 12.4 Audio uncanniness (likelihood: medium; severity: high — audio is the affective spine)

Procedural calls that read as "synthesizer" break the spell as badly as loops would. Mitigations: DSP development against recordings of real bird families as perceptual references (never shipped); blinded listening panels at M1 and M2 gates ("does this sound alive?" and "can you tell Pip from Wren blind?" — the recognizability test at 2, 4, and 7 birds); signature-parameter space designed with minimum perceptual distance between birds in one aviary (server assigns seeds checked for pairwise distance at adoption). Fallback posture: ship at 5-bird cap if 7-bird recognizability fails panels, since the cap is empirical by the PRD's own framing — flagged as a product decision gate, not an engineering unilateral.

### 12.5 Accessibility regressions / second-class drift (likelihood: medium; severity: high, named in PRD)

Risk that narration/reduced-motion lag features added late in M2. Mitigations: definition-of-done per workstream includes its accessibility surface; shared prose engine means new events need templates to ship at all (narration can't silently lag the visual); reduced-motion runs the same interpreter so new behaviors fail loudly (missing pose) rather than silently animating; recruited SR/RM beta users; external audit gate at M3.

### 12.6 Presence-signal dishonesty (likelihood: medium; severity: silent, population-wide)

Browser quirks (focus reporting differences, visibility edge cases on mobile Safari, pointer-event throttling in background) could over- or under-count presence. Mitigations: per-browser presence test matrix with synthetic events; server-side sanity bounds (presence windows capped at plausible session lengths; pings without recent activity flags discarded); cross-browser presence-rate dashboards (aggregate-only) watched for anomalies by browser family — a browser whose users "watch" 10× longer is a bug, not a cohort.

### 12.7 Time-to-first-bird budget erosion (likelihood: medium)

Bundle creep and edge-cache misses. Mitigations: per-chunk CI budgets; edge snapshot write-through monitored for staleness/hit-rate; cold-cache path (new device, new account) explicitly designed — quiet field is the designed experience for the slow case, and the 500ms target is measured at p75 with a p95 ceiling of 1.2s before alarm.

### 12.8 Magic-link friction and abuse (likelihood: low/medium)

Email deliverability is now a product dependency. Mitigations: reputable transactional provider + DKIM/SPF/DMARC from day one; delivery-success monitoring; resend affordance with matter-of-fact copy; per-email rate limits; invite emails on the same hardened path.

### 12.9 Scope gravity toward engagement features (likelihood: certain, over time; severity: total per PRD)

Mitigations are structural where possible: no per-account aggregates computed, notebook vocabulary excludes user behavior, banned-word CI, no notification infrastructure built (nothing to "just turn on"). Process: the non-goals file is part of onboarding; any PR touching copy or adding a user-facing surface requires a named "voice & restraint" reviewer.

## 13. Ambiguities resolved by this plan

Defensible calls made without further clarification, flagged for sign-off where noted:

1. **Mood set**: `wary, content, curious, drowsy, alert, settled` (PRD allows finalization in implementation).
2. **Presence activity window**: 5 minutes initial, server-config-tunable (PRD: "a few minutes," lean long).
3. **Tick cadence**: 60s, with observably-identical dormancy tiering (PRD fixes user-visible behavior, not fleet scheduling).
4. **Offer cooldown**: 3 minutes per bird ("a few minutes").
5. **No WebSockets in v1**: pull triggers satisfy every stated freshness need.
6. **Species-offer schedule**: 90/180/300/450/600 days ("a few months" → third bird; "a year" → five or six).
7. **Account export includes personality floats**: the PRD's export list names "current personality vectors"; treated as the single deliberate exception to numeric non-exposure (user's own data, on demand, not a product surface). **Needs explicit product sign-off** given the strength of the never-expose rule.
8. **Event-log retention**: 90 days post-consumption, aligned with the privacy posture. Privacy review to confirm.
9. **Mood "reset"**: implemented as continuous pull toward time-of-day baseline (no discrete reset a user could observe; satisfies "daily-ish cadence" + "never snaps").
10. **Renderer**: custom Canvas 2D, no engine dependency — bundle budget and scene simplicity justify it; revisit only if M1 perf testing fails on the reference laptop.
11. **Invite cap**: 10 outstanding invites per account (anti-abuse; PRD silent).
12. **Drift daily caps / weights**: initial values in §5.3 are starting points; the calibration tests, not the constants, are the contract.

## 14. Workstreams and sequencing

Six workstreams, sized for a team of ~8–10 engineers + design + audio DSP specialist:

| # | Workstream | Owns | Critical dependencies |
|---|---|---|---|
| W1 | Simulation engine | tick, drift, mood, greetings, weather, species offers, calibration harness | none — starts first; longest calendar bake |
| W2 | Platform & accounts | Postgres schema, auth/magic links, sessions, API service, export/delete, DB-role enforcement, edge snapshot cache | schema with W1 |
| W3 | Scene & rendering | canvas pipeline, behavior interpreter, boot path, responsive layout, reduced-motion render mode, top bar | snapshot shape (W1/W2) |
| W4 | Audio | worklet DSP, call grammar runtime, mix/listen-in, fallback ladder, listening panels | call-plan shape (W1), seeds (W2 schema) |
| W5 | Voice & prose | `@aviary/prose`, notebook generator, narration scheduler, captions, all product copy, style CI | state shapes (W1) |
| W6 | Quality & observability | perf CI (bundle/TTFB/60fps/memory), synthetic fleet, RUM pipeline + privacy proxy, accessibility CI, invariant checkers | all |

Sequencing follows §11 milestones. The two long poles are **calendar time for drift validation** (start W1 immediately; get internal aviaries live by week 5) and **audio believability iteration** (W4 listening panels from week 6, every two weeks). Everything else is conventional engineering with unusually strict invariants — and the invariants are encoded as CI gates and DB grants rather than documentation, which is this plan's central method: every load-bearing PRD rule gets a structural enforcement, not a reminder.
