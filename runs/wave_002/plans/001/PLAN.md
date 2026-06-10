# Pocket Aviary — v1 Implementation Plan

This is the executable engineering plan for Pocket Aviary v1, derived from the nine PRD files (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`). It is written for an engineering team to execute without further clarification. Where the PRD leaves a decision open, this plan makes the call and records it in §17 (Decision Log). Tunable constants are named and collected in Appendix A so calibration changes one table, not scattered magic numbers.

Assumed team: 8 engineers (2 backend/platform, 2 simulation engine, 2 client rendering/audio, 1 accessibility-focused client engineer, 1 SDET), plus a product designer, a part-time audio designer, and a PM. Target GA ≈ week 26 (§15).

---

## 0. How the PRD's principles bind the engineering

The five design principles are treated as enforceable engineering constraints, not tone guidance. Each one maps to a concrete mechanism that appears throughout this plan:

| Principle | Engineering enforcement |
|---|---|
| Feels alive, not robotic | Server-side tick is canonical (§5); first frame renders mid-action from snapshot (§7.4); procedural calls only, repetition detector in CI (§8, §14); no spinner anywhere — quiet-field load state is a designed component. |
| Notice, never announce | No toast/banner/modal components exist in the component library at all (you cannot use what is not there); PR checklist item "does this add an announcement surface?"; voice-lint CI blocks announcement lexicon (§14.4). |
| Charm from specificity | Shared prose library (`@aviary/voice`) is the only legal source of user-facing product copy; template review process; no-repeat memory in generators (§5.9, §10 narration). |
| Restraint over richness | Bird cap (7) and scene constraints encoded as engine constants and schema CHECK constraints, not UI-level limits. |
| Naturalist vs. matter-of-fact voice | Two copy registries (`voice/naturalist`, `voice/system`); every surface is assigned one at design time; lint forbids cross-register imports. |

Vocabulary rule: code, schemas, APIs, and metrics use PRD vocabulary exactly — `bird`, `call`, `listen_in`, `offer`, `settle`, `presence`, `visit`, `tick`. Never `solo`, `mute`, `select`, `pet`, `chirp`, `song`. This is a lint rule on identifiers in shared packages, because vocabulary drift in code is how vocabulary drift reaches the UI.

---

## 1. Scope

### 1.1 In scope for v1

- Web client (last two major versions of Chrome, Safari, Firefox, Edge), responsive desktop + mobile browser.
- Single-user accounts; email magic-link auth; per-device revocable sessions; email change with verification; account export; soft-then-hard deletion.
- One aviary per account; two starter birds at adoption; cap of seven; new-bird offers gated purely on aviary age.
- Server-authoritative simulation: ~1/min tick, personality drift (monotonic toward expressive), mood machine, weather, bird-to-bird interaction, notebook generation.
- Client: single-screen horizontal scene, three perch zones, day/night by user local time, ambient weather and micro-motion, top bar (account/settings, accessibility, notebook, offer) with fade.
- Interactions: presence accounting (three-condition conjunction), return-greeting, listen-in, offers (seed / song fragment / still pool) with per-bird cooldown, settle with 5s undo, field notebook (read-only, sparse, infinite scrollback).
- Audio: client-side procedural WebAudio synthesis, per-bird stable call signatures, chorus mixing, listen-in mix ramps, graceful-silence fallback with captions defaulted on.
- Accessibility shipped with v1, not after: screen-reader narration (naturalist prose, slow cadence), reduced-motion mode as a designed cross-fade renderer, runtime-generated call captions, full keyboard navigation, WCAG AA contrast on all user copy.
- Visits: per-invite, email-link, read-only ambient view; revocable; 30-day invite expiry; visit log in settings; visit notifications off by default with per-account opt-in toggle.
- Privacy architecture: synthetic account UUIDs everywhere; per-bird interaction data never leaves the simulation boundary; aggregate-only operational telemetry.
- Performance: <2MB gz initial bundle (internal target much lower), time-to-first-bird <500ms (warm path; see §12.2), 60fps idle on a 5-year-old laptop, zero memory growth over 30 minutes (CI-enforced).

### 1.2 Out of scope for v1 (and enforced, not just omitted)

Per `non_goals.md` and the brief: native apps; gamification of any kind (no streaks, achievements, levels, counters, visit calendars — including in settings, exports of visit frequency, or notebook entries about user behavior); Tamagotchi mechanics (no death, hunger, distress, decay meters); social surfaces beyond the single visit affordance (no profiles, follows, discovery, comments, chat, avatars, leaderboards — and no computation of the stats that would feed them); push notifications/emails about the aviary; payments; shared or multiple aviaries; customizable scenes; panning/zooming; user-controlled bird placement; recorded-audio fallback; exposure of personality vector values in any product UI.

Enforcement is architectural where possible: the analytics pipeline cannot read simulation tables (§10.5), the component library contains no toast/badge primitives, the notebook generator's lexicon denylist forbids user-behavior observations (§5.9), and "most-visited"-style aggregates are simply never computed.

---

## 2. System architecture

### 2.1 Shape: modular monolith + tick workers, single region

One TypeScript codebase, deployed as two processes from the same image:

- **API service** — HTTP: auth, snapshots, event ingest, notebook, settings, visits, export/deletion. Stateless, horizontally scaled.
- **Tick worker pool** — executes the simulation tick per aviary (§5.1), the notebook observer, export jobs, deletion jobs, email sends. Scaled by queue depth, isolated from API latency.

Backing services: **PostgreSQL** (single primary, regional; all canonical state), object storage for export bundles, a transactional email provider (Postmark primary, SES fallback) for magic links, invites, and export links. No Redis at v1 — job claiming and queues use Postgres `FOR UPDATE SKIP LOCKED` (§17 #19). CDN (Cloudflare or Fastly) serves the static app shell and species asset packs; the API is not edge-cached (authenticated, per-account).

Why a monolith: at v1 scale the tick is the only computationally interesting workload, and its correctness depends on transactional access to events + bird state — exactly what a single Postgres gives us for free. Module boundaries (`auth`, `sim`, `notebook`, `visits`, `telemetry`) are enforced in-repo via package boundaries and import lint so a later service split is mechanical, not archaeological.

Why TypeScript end-to-end: three load-bearing libraries must run identically on server and client — see §2.2. A polyglot stack would force us to implement the call grammar, the deterministic reaction functions, and the naturalist voice library twice and keep them bit-identical. That risk is worth more than any single-language downside here.

### 2.2 Shared isomorphic packages

- **`@aviary/engine-shared`** — deterministic presentation functions used by both server and client: the call-grammar runtime (§8.2), offer-reaction resolution (§5.7), greeting execution parameters, pose/behavior vocabulary, the seeded PCG32 RNG. Server uses it to decide canonical outcomes; client uses it to render the same outcome without a round-trip.
- **`@aviary/voice`** — the naturalist prose grammar (templates, slot fillers, no-repeat memory, lexicon) and the matter-of-fact system copy registry. Used server-side by the notebook generator and client-side by narration and captions, so a screen-reader user hears the same product the notebook writes.
- **`@aviary/schemas`** — zod schemas for snapshots, events, and API payloads; the single source of truth for the render-pipeline boundary contract.

### 2.3 The render-pipeline boundary (server decides *what*, client decides *how*)

This is the central architectural contract:

- **Server-canonical (semantic) state**: per-bird perch zone, behavior tag (`preen`, `scan`, `roost`, …), mood, presentation parameters derived from traits (quantized — §4.3), call-rate parameters and signature seed, greeting directive, weather, time-of-day phase, settled flag, offer cooldowns. Advanced only by the tick (plus snapshot-time greeting directive computation, §5.6).
- **Client-presentation (cosmetic) state**: frame-by-frame pose, micro-motion, exact call instants and motif realizations, ambient leaves/feathers, lighting interpolation, mix levels. Generated locally, seeded from server values, never reported back.

Rule of thumb encoded in review guidelines: if losing it would change the bird's future, it is server state; if losing it would only change this frame, it is client state. Presence and interactions flow upward only as events (§4.4); personality flows downward only as quantized presentation parameters.

### 2.4 Architecture sketch

```text
 Browser client                          Edge/CDN              Region
 ┌───────────────────────────────┐   ┌─────────────┐   ┌──────────────────────┐
 │ scene renderer (Canvas2D)     │   │ app shell    │   │ API service (N×)     │
 │ audio engine (WebAudio synth) │◄──│ species packs│   │  auth/snapshots/      │
 │ narration + captions          │   └─────────────┘   │  events/notebook/     │
 │ presence tracker              │──── events ────────►│  visits/settings      │
 │ snapshot store + interpolator │◄─── snapshots ──────│          │            │
 │ chrome (Preact, lazy)         │                     │      PostgreSQL       │
 │ service worker (shell+snap)   │                     │          ▲            │
 └───────────────────────────────┘                     │ tick workers (M×)     │
                                                       │  sim/notebook/jobs    │
                                                       │ email provider, S3    │
                                                       └──────────────────────┘
 telemetry: ops metrics/RUM (aggregate-only) → separate pipeline; NO path from sim DB to analytics (§10.5)
```

---

## 3. Data model

PostgreSQL. All primary keys are UUIDv7 (time-ordered, index-friendly). **The synthetic-ID rule is absolute**: `accounts.id` is the only identifier that ever appears in other tables, logs, queue payloads, or support tooling. Email exists in exactly two places, both encrypted: `accounts.email_enc` and `visit_invites.visitor_email_enc`.

### 3.1 Identity and auth

```sql
accounts (
  id              uuid PK,                 -- synthetic; the only cross-system identifier
  email_enc       bytea NOT NULL,          -- KMS envelope encryption
  email_hmac      bytea UNIQUE NOT NULL,   -- HMAC-SHA256(email, server pepper) for lookup; not reversible
  tz              text NOT NULL DEFAULT 'UTC',  -- last client-reported IANA zone (§17 #5)
  settings        jsonb NOT NULL DEFAULT '{}',  -- audio_enabled, captions, reduced_motion_override,
                                                -- visit_notifications (default false)
  created_at      timestamptz NOT NULL,
  deletion_requested_at timestamptz NULL   -- soft-deletion marker; hard delete at +30d
)

sessions (
  id uuid PK, account_id uuid FK, token_hash bytea UNIQUE,  -- SHA-256 of 256-bit token
  device_label text,                       -- parsed from UA, e.g. "Safari on macOS"
  created_at, last_seen_at timestamptz, revoked_at timestamptz NULL
)

magic_links (
  id uuid PK, email_hmac bytea, token_hash bytea UNIQUE,
  created_at timestamptz, expires_at timestamptz,           -- created_at + 15 min
  consumed_at timestamptz NULL                              -- single-use; set under row lock
)

pending_email_changes (account_id FK, new_email_enc, new_email_hmac, token_hash, expires_at)
```

### 3.2 Aviary and birds

```sql
aviaries (
  id uuid PK, account_id uuid UNIQUE FK,   -- one aviary per account, enforced
  created_at timestamptz NOT NULL,         -- drives new-bird offers (age, nothing else)
  rng_seed bigint NOT NULL,                -- per-aviary deterministic RNG root
  tick_no bigint NOT NULL DEFAULT 0,
  last_tick_at timestamptz NOT NULL,
  next_tick_at timestamptz NOT NULL,       -- scheduler claim column (§5.1)
  weather jsonb NOT NULL,                  -- {kind, started_at, ends_at} | {kind:"clear"}
  settled boolean NOT NULL DEFAULT false,  -- cleared on next presence
  rhythm numeric NOT NULL,                 -- expression-rhythm EMA, minutes/day (§5.4)
  presence_today numeric NOT NULL,         -- union-of-intervals minutes, rolling (§5.3)
  last_presence_ended_at timestamptz NULL,
  greeting_cache jsonb NULL,               -- directive for current presence gap (§5.6)
  pending_bird_offer jsonb NULL,           -- visiting-bird state (§5.8)
  snapshot_json jsonb NOT NULL             -- serving cache, rebuilt each tick
)

birds (
  id uuid PK, aviary_id uuid FK, species_id text NOT NULL,  -- stable id = bird identity, never reused
  name text NOT NULL,                      -- renameable; identity unaffected
  adopted_at timestamptz NOT NULL,
  signature_seed bigint NOT NULL,          -- call signature; immutable for the bird's life
  -- personality vector: server-only columns; NEVER serialized into snapshot_json
  boldness numeric NOT NULL CHECK (boldness BETWEEN 0 AND 1),
  social_warmth numeric NOT NULL CHECK (social_warmth BETWEEN 0 AND 1),
  vocal_frequency numeric NOT NULL CHECK (vocal_frequency BETWEEN 0 AND 1),
  plumage_saturation numeric NOT NULL CHECK (plumage_saturation BETWEEN 0 AND 1),
  curiosity numeric NOT NULL CHECK (curiosity BETWEEN 0 AND 1),
  drift_day_accum jsonb NOT NULL,          -- per-trait rolling 24h applied-delta, for the daily cap
  drift_pending jsonb NOT NULL,            -- leaky-integrator pool (§5.4)
  mood text NOT NULL,                      -- wary|content|curious|drowsy|alert; persists across sessions
  mood_since timestamptz NOT NULL,
  mood_scores jsonb NOT NULL,              -- hysteresis state
  perch text NOT NULL CHECK (perch IN ('front','middle','back')),
  behavior text NOT NULL
)
```

Monotonic-drift invariant is enforced in code (the only personality writer is the tick's drift step, whose deltas are clamped ≥ 0) and verified by a DB trigger in non-prod that raises on any decrease — a tripwire for bugs, removed in prod for write cost.

### 3.3 Events, notebook, visits

```sql
interaction_events (                        -- append-only; the ONLY upward data path
  seq bigserial PK,                         -- server-assigned global order (§6.2)
  id uuid UNIQUE,                           -- client idempotency key
  aviary_id uuid FK, session_id uuid FK,
  type text CHECK (type IN ('presence_ping','listen_in_start','listen_in_end',
                            'offer','settle','settle_undo','bird_welcome')),
  payload jsonb,                            -- e.g. {bird_id}, {offer_kind, zone}, {span_s:30}
  client_t timestamptz,                     -- recorded for context; never used for ordering
  received_at timestamptz NOT NULL,
  processed_tick bigint NULL
) PARTITION BY RANGE (received_at);         -- monthly partitions; dropped after 30 days (§10.4)

notebook_entries (
  id uuid PK, aviary_id uuid FK, created_at timestamptz,
  body text NOT NULL,                       -- final prose; lowercase naturalist voice
  template_id text NOT NULL,                -- for per-template cooldowns and repetition audits
  bird_ids uuid[] NOT NULL DEFAULT '{}'
)                                           -- read-only; retained for account lifetime; in export

visit_invites (
  id uuid PK, aviary_id uuid FK,
  visitor_email_enc bytea NOT NULL,         -- shown only to the host in their visit log
  token_hash bytea UNIQUE,                  -- one-time consumption
  created_at, expires_at timestamptz,       -- +30 days
  consumed_at timestamptz NULL, revoked_at timestamptz NULL
)

visit_sessions (
  id uuid PK, invite_id uuid FK, cookie_hash bytea,
  started_at, last_heartbeat_at timestamptz -- approximate duration for the host's log; not telemetry
)

bird_trait_log (bird_id, tick_no, deltas jsonb, created_at)
  -- calibration/debug audit. Same privacy class as the simulation DB: never exported to analytics,
  -- queried only for synthetic-cohort and consented-internal accounts (§10.5). 90-day retention.
```

`species` is a static, code-shipped catalog (six species): silhouette part-set, palette ranges, motif library id, time-of-day activity profile (one nightjar-like species is night-active), trait seed offsets.

### 3.4 What is deliberately not modeled

No visit counts on accounts, no per-account interaction aggregates, no "stats" tables, no denormalized engagement rollups. Absences are the cheap way to keep §1.2's refusals refused: features that would need those tables would have to add them in a reviewed migration, which is the moment to say no.

---

## 4. API surface

Conventions: JSON over HTTPS; session cookie (httpOnly, Secure, SameSite=Lax) + CSRF token on mutations; all error copy from the matter-of-fact registry; rate limits per session and per IP; ETag/If-None-Match on snapshot and notebook; brotli. Account-scoped endpoints derive the aviary from the session — aviary ids never appear in URLs the user can edit.

### 4.1 Auth & account

```text
POST /auth/magic-link        {email}                 → 202 always (no account-existence oracle)
POST /auth/consume           {token}                 → sets session cookie; 15-min expiry, single-use
GET  /api/sessions                                   → device list (label, created, last seen)
POST /api/sessions/:id/revoke
POST /api/account/email-change        {new_email}    → verification mail to new address
POST /api/account/email-change/confirm {token}       → old email works until this commits
GET/PATCH /api/account/settings                      → audio, captions, reduced-motion override,
                                                       visit notifications, tz (client-reported)
POST /api/account/export                              → 202; job emails a signed, expiring link
POST /api/account/delete                              → soft-delete now; restore via any signed-in page
POST /api/account/restore
PATCH /api/birds/:id          {name}                  → rename only; nothing else is client-writable
POST /api/birds/welcome       {offer_id, name}        → accept a pending new-bird offer (§5.8)
```

### 4.2 State down: snapshot

```text
GET /api/aviary/snapshot      → 200 + ETag (tick_no + greeting nonce); 304 when unchanged
```

Pulled on tab-becoming-visible, on long render-frame gap (resume from suspend/bfcache), and on a keepalive every `SNAPSHOT_POLL_S = 45s` while visible. Target payload ≤ 8KB brotli. Shape (abridged; canonical zod schema in `@aviary/schemas`):

```json
{
  "tick_no": 482113,
  "server_time": "2026-06-09T14:21:08Z",
  "aviary": { "tod_phase": "morning", "weather": {"kind": "rain", "ends_in_s": 140},
              "settled": false, "rhythm_level": 5 },
  "greeting": { "bird_id": "b_pip", "tier": 2, "seed": "9f3a…", "stagger_ms": [0, 1800] },
  "birds": [{
      "id": "b_pip", "name": "pip", "species": "reed_warbler",
      "perch": "front", "behavior": "preen", "mood": "content",
      "presentation": {
        "plumage_level": 4, "approach_level": 5, "greet_eagerness_level": 4,
        "call": { "rate_level": 5, "signature_seed": "c41d…", "motif_set": "rw_a" },
        "pose_seed": "77b2…"
      }
  }],
  "offer_cooldowns_s": { "b_pip": 0, "b_wren": 132 },
  "absence_s": 86400,
  "bird_offer": null
}
```

### 4.3 The quantization rule (personality never ships raw)

Trait values never leave the simulation module — not to the client, not to logs, not to telemetry. The tick maps each trait to coarse `*_level` integers (1–7) plus seeded dither for rendering variety. Rationale: the PRD's "never exposed numerically, not in a debug view" must survive devtools. Levels move on the weeks timescale, so a user watching the network tab sees nothing to optimize; behavior thresholds in the client key off levels, which also gives us the user-visible drift threshold for calibration (§5.4: one level step ≈ 0.10 of trait range). The one sanctioned exception is the account export (§10.3, Decision #11).

### 4.4 State up: events

```text
POST /api/events   { "events": [
   {"id":"uuidv7","type":"presence_ping","span_s":30,"client_t":"…"},
   {"id":"…","type":"listen_in_start","payload":{"bird_id":"b_pip"}},
   {"id":"…","type":"offer","payload":{"offer_kind":"seed","zone":"front"}},
   {"id":"…","type":"settle"} ] }      → 202 {accepted, duplicates}
```

Batched (flush every 15s or 20 events), idempotent by event `id`, retried with backoff; `navigator.sendBeacon` on pagehide for the final batch. Server stamps `seq` and `received_at`; client timestamps are advisory only. Per-session server-side sanity clamps reject implausible volumes (e.g., presence spans summing past wall-clock; §10.6).

### 4.5 Notebook and visits

```text
GET  /api/notebook?cursor=<id>&limit=30      → reverse-chronological, cursor-paginated, indefinite scrollback
POST /api/visits/invites   {email}           → sends one-time link; 30-day expiry
GET  /api/visits                              → outstanding invites + visit log (email, date, ~duration)
POST /api/visits/invites/:id/revoke           → immediate; visitor cut at next pull (§11)

-- visitor scope (separate, unauthenticated-account path; visitor cookie scoped to /visit)
POST /visit/consume        {token}            → one-time; sets visitor cookie bound to invite
GET  /visit/snapshot                          → same snapshot shape minus greeting, cooldowns, absence
POST /visit/heartbeat                         → updates visit_sessions duration only; NOT interaction_events
```

The visitor scope is a separate router with an allowlist of exactly these three routes; visitor cookies cannot authenticate against any `/api/*` route. Read-only is enforced by scope, not by UI.

---

## 5. Simulation engine

### 5.1 Tick mechanics

Cadence `TICK_S = 60s` per aviary. Workers claim due aviaries:

```sql
SELECT id FROM aviaries WHERE next_tick_at <= now()
  ORDER BY next_tick_at FOR UPDATE SKIP LOCKED LIMIT 100;
```

The row lock serializes ticks per aviary — the single-writer invariant (§6.1) is a property of the claim, not a convention. Each tick runs in one transaction:

1. Read events with `processed_tick IS NULL AND seq <= current max` in `seq` order.
2. Update presence accounting (§5.3) and the rhythm EMA (§5.4).
3. Compute drift deltas, apply caps, drain the pending pool into trait columns; append `bird_trait_log`.
4. Run the mood machine per bird (§5.5), including weather and neighbor effects.
5. Assign perch/behavior tags from mood + traits + time of day (sampled with the tick RNG).
6. Advance weather process (§5.5.1).
7. Run the notebook observer (§5.9).
8. Run the new-bird-offer check (§5.8).
9. Mark events processed; rebuild `snapshot_json`; `tick_no += 1`; `next_tick_at += TICK_S`.

**Determinism**: all randomness inside a tick comes from PCG32 seeded by `(aviary.rng_seed, tick_no)`. Re-running a tick from the same inputs yields identical output — the foundation for fast-forward equivalence and for replayable bug reports.

**Dormancy fast-forward**: aviaries with no presence for 48h move to a 15-minute claim cadence; each claim executes the elapsed ticks as a batched fast-forward inside one transaction. Because mood/weather evolution between events is a deterministic function of (state, tick range, RNG), the batch computes multi-tick evolution in O(state) rather than O(ticks) where closed-form (drift with empty event log is zero; mood follows the time-of-day prior path; weather is Poisson-schedulable). On any client snapshot request, if `last_tick_at` is stale the API performs the synchronous catch-up before serving — the user always sees the aviary that kept running. **Golden equivalence tests assert fast-forward ≡ the same range executed as single ticks, exactly** (§14.2). Observable behavior is therefore indistinguishable from a literal per-minute tick, satisfying the PRD's contract at any scale.

Backpressure: queue depth and per-tick latency are first-class metrics; p99 tick latency alarms at 5s (PRD requirement), with target p50 < 250ms.

### 5.2 Engine state vocabulary

- Traits: `boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity` ∈ [0,1]. New birds seed at `U(0.25, 0.45)` plus species offsets (e.g., nightjar `vocal_frequency +0.10`), drawn from the aviary RNG at adoption.
- Moods: `{alert, curious, content, drowsy, wary}` (PRD's enumerated set; finalized here). `roosting` is a **behavior**, not a mood: night + drowsy for non-night species → eyes closed, low posture. Night is not dead — the nightjar species' activity profile keeps it calling late.
- Behaviors (semantic tags the client animates): `perch_idle, preen, scan, hop, drink, bathe, watch, call_bout, approach, retreat, roost`.

### 5.3 Presence accounting (the honest signal)

Client side: a presence sampler runs every 5s and evaluates the three-condition conjunction — `document.visibilityState === 'visible'` AND `document.hasFocus()` AND last input within `PRESENCE_ACTIVITY_WINDOW_S = 240s` (default 4 min; calibrate 2–6, leaning long, per PRD). "Input" is pointermove/keydown on fine pointers; on coarse (touch) pointers, where pointermove is rare by nature, it is any `pointerdown`/`pointermove`/scroll/keydown (Decision #4 — the PRD's pointermove-or-keypress baseline assumes a desktop; touch devices need the equivalent signal, not a stricter one). Listeners are passive and throttled to a timestamp write.

While the conjunction holds, the client emits `presence_ping {span_s:30}` every 30s. The very first ping of a visible session is sent immediately (it also clears `settled` and re-anchors the rhythm).

Server side: pings append to the event log; the tick folds them into **interval unions per account, not sums per session** — two devices visible simultaneously count once (Decision #15). Per-account presence is clamped to wall-clock elapsed since last tick. There is no "tab open" path to presence anywhere in the code; the sampler is the only producer, and it tests the conjunction every sample.

Settle and tab-close are identical to the engine: both simply stop pings. `settle` additionally sets the settled lighting state and quiets the call scheduler; `settle_undo` within 5s reverts it. No penalty, recovery surface, or notification exists for un-settled exits.

### 5.4 Drift: the low-pass filter, with numbers

Definitions, all tunable (Appendix A), with the calibration persona "regular user" = 5 visits/week × 20 min presence:

**Input weights** (per bird, per tick; all deltas multiplied by `(1 − trait)` for soft saturation):

| Source | Trait deltas |
|---|---|
| presence-minute (account-level, applies to all birds) | plumage +3.0e-4 · warmth +1.5e-4 · boldness +1.5e-4 · vocal +1.5e-4 · curiosity +1.0e-4 |
| listen-in focused-minute (focused bird only) | warmth +6.0e-4 · vocal +6.0e-4 |
| offer accepted (receiving bird) | curiosity +2.5e-3 |
| offer placed in a bird's zone | boldness +8.0e-4 |
| settle | no drift; ends the presence window cleanly |

**Low-pass application**: deltas accumulate into `drift_pending` and drain into the trait columns with time constant `DRIFT_DRAIN_TAU = 36h` (a leaky integrator). A burst session therefore surfaces over the following days, and no single session moves a trait visibly — the PRD's explicit requirement, made mechanical.

**Daily cap**: applied drift per trait ≤ `DRIFT_DAILY_CAP = 0.012` per rolling 24h (tracked in `drift_day_accum`). Marathon usage tops out at ~2× the regular pace; drift-farming is structurally bounded (and §10.6 clamps the inputs anyway).

**Calibration targets, asserted in CI** (§14.2): regular persona ⇒ primary traits gain ≈ 0.03–0.04/week ⇒ crosses the instrument threshold (0.02) within week 1 ✔; crosses one presentation level (quantization step 1/7 ≈ 0.10 — the user-visible threshold, §4.3) at ~3 weeks ✔. Weekend-only persona ⇒ visible ~6–8 weeks (acceptable; slower attention, slower current). The instruments-vs-user gap is realized precisely by the quantization step being ~3× the instrument threshold.

**Monotonicity**: every delta above is ≥ 0; neglect contributes zero. Traits never decrease, ever — property-tested over arbitrary event logs (§14.2).

**Becoming ambient without punishment — the expression rhythm**: observable quieting on neglect comes from a separate, non-trait, fast-recovering modulation. `aviaries.rhythm` is an EMA of daily presence-minutes with asymmetric time constants: decay τ = 10 days, recovery τ = 2 days. It maps to a greeting-eagerness multiplier `0.45 + 0.55·min(1, rhythm/30)` and an unobserved-call-rate multiplier `0.6 + 0.4·min(1, rhythm/30)` — floored, never zero (the aviary is always alive, always calling). A two-week absence returns quieter birds whose personalities are fully intact, and a few sessions restore the old rhythm. This is the engine-level realization of "greeting less often because less often is what's been observed," kept deliberately outside the personality vector so it can never read as punishment or decay (Decision #20).

### 5.5 Mood machine

Per bird per tick, compute a score per mood state and apply hysteresis:

- **Time-of-day prior** (account tz): alert peaks early morning; drowsy rises from dusk; content mid-day plateau. Smooth 24h curves per state, per species activity profile (the nightjar's drowsy curve is inverted toward day).
- **Personality affinity**: e.g., wary score scaled by `(1 − 0.6·boldness)`; curious by curiosity; content by warmth. A bold bird is structurally resistant to wary on the same input — the PRD's example, implemented as a multiplier.
- **Event terms**, exponentially decaying with 20-min half-life: accepted offer → content/curious; song-fragment offer → curious or alert by vocal frequency; alarm-call event from a neighbor → wary, with **spatial spread**: birds on the same or adjacent perch zone receive 0.6× of the source's wary impulse (bird-to-bird mood propagation).
- **Weather terms**: rain → small wary + vocal damping for its duration; wind → alert for some species, wary for others (species table flag).
- **Transition rule**: switch only if a challenger exceeds the incumbent by `MOOD_HYSTERESIS = 0.15` for 2 consecutive ticks, with `MOOD_MIN_DWELL = 15 min`. No flapping; mood reads as weather, not noise.
- **Daily-ish reset**: at local ~06:00 (per-bird ±40 min jitter from the day's RNG), scores get a pull toward a personality-tinted baseline — the "morning re-anchor" that gives mood its daily cadence without a visible snap.

Mood persists in `birds.mood` across sessions by construction; the client never resets or invents mood. A bird left drowsy at dusk is asleep (roosting behavior) by night-tick evolution and softening toward content by morning — the user never sees a snap-to-default because there is no default to snap to.

#### 5.5.1 Weather process

Server-side Poisson scheduling per aviary: rain ~2.5 events/week, 4–10 min; soft wind ~1/day, 2–5 min. Never assertive (no storms, no snow — not in the content set at all). Weather is canonical state, so a host and a simultaneous visitor see the same rain.

### 5.6 Return-greeting (the anchor moment)

Decided server-side at **snapshot-serve time** (not tick time — it must reflect the actual return instant), executed client-side:

- Trigger: gap since `last_presence_ended_at` > 90s (below that it's the same sitting, not a return).
- Greeter selection: softmax (temperature 0.7) over birds with logits `1.5·boldness + 1.0·social_warmth + mood_bias(alert +0.3, content +0.1, drowsy −0.3, wary −0.4) + 0.3·rhythm_factor`. The same bold bird tends to greet first across visits; a wary bird on a wary day may effectively never be sampled — both PRD behaviors fall out of the math rather than being special-cased.
- Intensity tier by absence: <30 min → tier 1 (glance up / head-tilt); 30 min–24 h → tier 2 (look + quiet two-note call); >24 h → tier 3 (approach toward front perch + longer call, with probability `0.35 + 0.4·warmth` of a second bird responding). Tier shapes parameters; the realization is procedural.
- Procedural variation: the directive carries a fresh seed; `@aviary/engine-shared` expands it into pose timing, call motif choice, and gaze path. Never a canned clip, never three variants in rotation — variation is continuous in the seed space, and the repetition detector (§14.5) watches greetings too.
- Multi-bird stagger: directive includes randomized offsets of 1.2–3.5s; simultaneous-onset is structurally impossible.
- Idempotence: the directive is cached on the aviary keyed by the presence-gap, so a user opening laptop and phone within the same return sees the same bird greet on both; it is cleared when presence resumes.
- Within 1–2s of tab-open: guaranteed by serving the directive in the snapshot (warm path uses the service-worker-cached snapshot's directive immediately, reconciled silently if the fresh snapshot differs in cosmetic ways).

There is no welcome toast, banner, modal, or text. The greeting is the entire welcome surface; nothing else exists to fire.

### 5.7 Offers (deterministic shared resolution)

Client POSTs the offer event and needs an immediate reaction; the server needs the same outcome for drift. Both call the same pure function from `@aviary/engine-shared`:

```text
resolveOffer(offer_kind, zone, per-bird {mood, curiosity_level, boldness_level, vocal_level}, seed = event.id)
  → per-bird reaction: {approach_after_s | watch | ignore | join_song | quiet | call_against | drink | bathe}
```

Seeded by the event's idempotency id, client and server compute identical outcomes with zero round-trip latency — the curious content bird approaches the seed now, the wary bird waits and edges in late, the drowsy bird doesn't stir; the song fragment plays softly and each bird joins, quiets, or calls against per vocal frequency and mood; the still pool draws drinkers, bathers, watchers. The server's tick records acceptance for curiosity drift from the same resolution.

Cooldown: `OFFER_COOLDOWN_S = 300` per bird, enforced server-side (events during cooldown are accepted but resolve to no-drift, no-reaction) and surfaced in the snapshot so the client affordance simply waits — quietly, with no countdown timer UI (a countdown is a meter; the offer affordance just isn't ready yet). Offers live in the top bar only; clicking a bird is listen-in, never an offer.

### 5.8 Adoption, new-bird offers, the cap

- **Adoption**: account creation → aviary row + two starter birds, species chosen by seeded draw from the pool (no catalog; "the birds that arrived"). Naming affordance with editable suggestions; rename anytime later from settings (Decision #10). Empty-aviary quiet field renders until the first soft fly-in (§7.4).
- **Growth by age only**: offer thresholds at aviary age ≈ 90d for the third bird, then every 90–120d (per-aviary jitter to avoid cohort synchronization), hard cap 7 (schema CHECK + engine constant). No other input — not visits, not interactions, not anything — reaches this check; the function signature takes `aviary.created_at` and nothing else, which makes the rule auditable at a glance.
- **Offer surface (Decision #9)**: a "visiting bird" pattern — the new bird simply appears on the far back perch for a few days (`pending_bird_offer`), unnamed, slightly apart; the notebook may note "an unfamiliar shape on the far branch this morning." The offer affordance in the top bar gains one quiet additional item: *a bird has been visiting* → name-and-welcome flow (`POST /api/birds/welcome`), or decline (it leaves quietly; another arrives at the next threshold). No modal, no announcement, no badge. Requires a design pass; the constraint it must satisfy is written here: the user discovers the bird by watching, not by being told.

### 5.9 Field notebook generator

Runs in the tick (step 7): **detectors** emit candidate observations with salience — first-greeter changes ("pip greeted before wren today, first time this week"), weather moments, long-quiet mornings, offer micro-stories, a wary bird's first front-perch visit, the visiting bird, night calls. A **sparsity governor** then gates publication:

- Token bucket targeting ~1 entry / 3 days for a regularly-visited aviary (`NOTEBOOK_RATE`), salience-weighted; rare events (adoption, a first, a new bird) bypass the bucket.
- Per-template cooldown 30 days; recent-entry similarity check (template id + bird ids + slot hash) so the notebook never rhymes with itself.
- Active users do not get more entries — the bucket is per-aviary-time, not per-event. Sparsity is preserved by construction.

Prose from `@aviary/voice`: lowercase, present tense, bird-named, concrete slot fillers (perch, weather, time words), no-repeat memory across the account's recent entries. **The hard line, lint-enforced in the template corpus**: entries observe the aviary, never the user — denylist includes second person ("you", "your"), visit/streak/frequency language, counts of user actions, and announcement words ("welcome", "achievement", "unlocked", "!"). A template that cannot be written without referring to the user's behavior is not a notebook entry; the linter makes this a build failure, not a review opinion (§14.4).

Notebook is read-only end-to-end: no mutation endpoint exists. Scrollback is cursor-paginated forever; entries are in the export and die with the account.

---

## 6. Sync model and consistency

### 6.1 Invariants

1. **Single writer**: only the tick worker writes personality, mood, perch/behavior, weather, rhythm — under the per-aviary row lock (§5.1). No API code path imports the trait-write DAO; an import-boundary lint enforces this at build time (§14.6).
2. **Additive deltas only**: clients submit events ("user listened in to pip for 3 minutes"), never values ("set boldness"). There is no endpoint whose payload could carry a trait — the schema layer makes last-write-wins unrepresentable, not just forbidden.
3. **Total order**: events are ordered by server-assigned `seq` (single Postgres sequence). Client clocks are recorded but never ordering inputs. Two devices' interleaved events fold in arrival order; because all drift is additive and capped, interleaving order cannot lose data — the morning laptop's listen-in and the lunchtime phone's offer both land, regardless of session overlap.
4. **Idempotence**: event `id` dedup makes client retries safe; tick reprocessing is prevented by `processed_tick` within the same transaction that advances `tick_no`.

### 6.2 Multi-device behavior

Both devices poll the same snapshot and submit to the same log; there is no client-to-client path and no merge anywhere. Concretely: laptop and phone open simultaneously → same `tick_no`, same moods, same weather, same greeting directive (cached per presence-gap); presence counts once via interval union; a listen-in on the phone drifts the same canonical bird the laptop is rendering. Divergence between devices is bounded by the snapshot poll (≤45s) and is cosmetic-only by the §2.3 contract.

### 6.3 Failure handling

| Failure | Behavior |
|---|---|
| Event POST fails | Client queue retries with backoff; sendBeacon on pagehide; idempotent on the server. Lost-forever events lose at most their marginal drift — never corrupt state. |
| Snapshot fetch fails | Client keeps rendering from last snapshot (presentation layer is self-sufficient for minutes); retries; after 90s shows the matter-of-fact reload surface ("Something went wrong loading your aviary…"). |
| Tick worker crash mid-tick | Transaction rolls back; another worker claims; determinism makes the retry identical. |
| Laptop resumes from suspend | Long render-frame gap triggers an immediate snapshot pull; interpolator reconciles (birds cross-fade/move to canonical positions; never teleport). |
| Session revoked / expired | Next API call 401 → matter-of-fact sign-in surface ("Your session timed out. Sign in again to keep watching."). |
| Postgres failover | API serves 503 with quiet-field client behavior; ticks resume and fast-forward — by design, downtime reads as "the aviary kept going without us," because canonically it did. |

Clock policy: server time is the only clock for state; the snapshot carries `server_time` and the client renders time-of-day from account tz computed server-side (`tod_phase`), so a device with a wrong clock still sees the right aviary.

---

## 7. Frontend architecture and rendering pipeline

### 7.1 Stack

- **Scene**: custom renderer on Canvas 2D (no game engine, no React in the scene). ≤7 birds + ≤6 ambient particles + ~6 cached layers is comfortably within Canvas 2D fill budget on the 5-year-old-laptop target; a WebGL backend stays behind the renderer interface as a contingency, not a v1 deliverable (Decision #2).
- **Chrome** (top bar, notebook panel, offer panel, settings): Preact + signals, code-split per panel; the scene never re-renders through the UI framework.
- **Service worker**: precaches app shell + species packs; caches the last snapshot per session for instant warm starts (cleared on sign-out/revocation — shared-computer hygiene).
- Module layout: `net/session`, `presence`, `snapshot store + interpolator`, `behavior presenter`, `scene renderer`, `audio engine`, `narration`, `captions`, `chrome/*` (lazy), `a11y focus manager`.

### 7.2 Scene composition

Layer stack, each an offscreen canvas composited per frame: sky gradient (tod-driven) → background foliage → middle plane (perches + birds) → occasional foreground branch/leaf → caption text layer (DOM, for accessibility/contrast control) → top bar (DOM). Parallax: a slow autonomous sub-pixel sway (no camera control exists) offsets layers by depth — gentle by construction (amplitude ≤ 4px, period ~40s), removed in reduced-motion.

Responsive: scene scales to viewport preserving a band aspect; perch anchors are layout-relative so narrow phones compress spacing without ever cropping a bird (a layout assertion test renders the matrix of viewports and fails if any bird's bounds exit the canvas). No panning, scrolling, or zooming exists in the input model.

Palette: tokens from the design system (calm naturalist set; no saturated accents); day/night cycle interpolates palette keyframes (dawn/morning/midday/dusk/night) on the account-local clock from the snapshot; evening warms, night dims with the nightjar still active.

### 7.3 Bird presenter (semantic → motion → pose → pixels)

1. **Behavior layer**: consumes server behavior tags + greeting/offer directives; schedules behavior bouts locally between snapshots using the same seeded grammar as the server (the client may originate cosmetic bouts — a preen, a head-tilt at a call — freely; it may never originate semantic change).
2. **Motion layer**: continuous procedural micro-motion — breathing (perlin, ~0.2Hz, mood-scaled), weight-shuffle, scan saccades, preen sequences, head-tilts event-keyed to the audio scheduler (a bird tilts toward a sound that actually played). Mood-shaped presets: wary = back perch, taller posture, frequent scans; content = preening; curious = tilt-and-watch; drowsy = low, fluffed, slow. The user reads mood from motion; no label, tooltip, or status icon exists.
3. **Pose solver**: parts-based 2D skeleton per species silhouette (body/head/wing/tail/legs), pose = parameter vector; plumage `*_level` selects palette richness and feather-detail overlays. Composed poses are cached to offscreen sprites; re-render only on pose-parameter change beyond epsilon.
4. **Interpolator**: snapshot deltas (perch change, mood change) become planned transitions — a hop/flight arc (or reduced-motion cross-fade) over 600–1200ms. Birds never teleport, including after suspend-resume reconciliation.

Idle motion runs whenever the tab is visible; when hidden, rAF stops (rendering is wasted), the presence conjunction is already false, and the server keeps simulating — on return, the snapshot pull plus interpolation resumes mid-action.

### 7.4 First frame and loading (no spinner exists)

- **Warm start (the common case)**: service worker serves shell + last snapshot synchronously; first frame composites birds mid-action (pose seeds advance deterministically from snapshot time, so they're literally mid-preen, mid-scan) in well under 500ms; a fresh snapshot fetch reconciles silently. Audio joins per autoplay policy (§8.6).
- **Cold start**: a ≤250KB critical path (bootstrap renderer + sky + species core) races the snapshot fetch; until both land, the **quiet field** renders — soft sky color with one or two faint drifting motion cues. It is a designed component in the scene renderer, not a loader: no spinner, no progress bar, no skeleton UI, no fade-from-static. When state arrives, birds are drawn already in motion (no entry animation); the only fly-in that ever exists is the adoption empty-aviary → first-bird moment.
- The loading-state review rule: any PR that adds a wait state must use the quiet field; spinner components do not exist in the library.

### 7.5 Top bar and input model

Top bar: four items only — account/settings, accessibility settings, notebook, offer. Fades to ~5% opacity after 4s of pointer stillness; returns on pointermove/keyboard/touch. DOM-rendered for a11y; AA contrast at full opacity; fade is suspended entirely when keyboard focus is within it or when a screen reader/keyboard modality is detected (a faded control must never be a focused control).

Pointer/touch: click/tap bird = listen-in toggle; click/tap empty scene = disengage listen-in (and wake top bar); settle in top-bar overflow with its 5s any-click undo; long-press does nothing (no hidden gestures). Keyboard: §9.3.

---

## 8. Audio pipeline

### 8.1 Architecture

One `AudioContext`. Per-bird **voice** = pooled synth graph: 2 oscillators (sine/triangle blend) + filtered-noise breath component → formant-ish bandpass pair → ADSR gain, with pitch/amp LFOs for vibrato and contour. Voices are allocated at aviary load (≤7 + 2 spare) and **reused forever — zero per-call allocation** (the §12.4 memory rule applied to audio). Bus graph: bird voices → per-bird gain (listen-in) → spatial pan (perch-zone-derived, subtle) → chorus bus → master compressor → small feedback-delay-network "open air" reverb (generated IR, no audio assets). Song-fragment offers and ambient weather (rain wash = filtered noise) get their own bus.

CPU budget: ≤3% of one core on the 5-year-old laptop during a full chorus; measured in the perf rig. Synthesis is scheduled 250ms ahead on a lookahead timer; no AudioWorklet requirement at v1 (plain nodes suffice for this voice count), keeping Safari risk low.

### 8.2 Call grammar runtime (`@aviary/engine-shared`)

- **Species motif library**: each species defines 4–6 parametric motifs (e.g., rising two-note, low trill, paused double-trill) as synthesis parameter curves — code, not audio files.
- **Bird signature**: `signature_seed` (immutable for the bird's life) fixes base pitch offset, timbre filter setting, motif preference weights, and rhythmic feel. **Signature parameters are independent of mood and traits** — this is the architectural guarantee that Pip sounds like Pip across drift and mood, satisfiable by construction and testable (§14.5).
- **Modulation**: mood shapes tempo/intensity/inter-call interval (drowsy = sparse, soft, low; alert = brighter, quicker); `vocal rate_level` scales spontaneous call frequency and chorus-join probability; rain damps globally; the rhythm multiplier (§5.4) scales unobserved call rate.
- **Realization**: a weighted stochastic grammar (motif sequences with rests, pitch jitter ±3%, timing jitter ±8%) ensures no two calls are ever bit-identical; an autocorrelation-based repetition detector in CI fails if consecutive realizations are too similar (§14.5).
- The grammar emits a **structured motif trace** per call — consumed by the synth (to play), by captions (to describe what actually played), and by narration (to mention "calling softly"). One source of truth, three surfaces.

### 8.3 Chorus scheduler

Client-side scheduler with bird-to-bird coupling: a call by one bird raises short-term response probability in others (weighted by warmth and vocal level), producing call-and-response and emergent chorus windows when multiple high-vocal birds align. Anti-unison rules: minimum 350ms onset separation enforced at schedule time; pitch-collision avoidance nudges simultaneous birds apart by their signature offsets. Two birds calling at once is a real mixed chorus of two distinct synth realizations — phase-cancellation artifacts are structurally impossible with no loops to stack.

### 8.4 Listen-in mix

Engage: focused bird ramps +6dB toward mix front over `LISTEN_RAMP_S = 2.0s` (exponential ramp); others ramp down to an ambient bed at −14dB relative — **never below the floor, never muted**. Disengage (re-click, focus elsewhere, empty-space click, keyboard blur, Escape): same slow ramp back. Hard cuts do not exist in the gain code path (all level changes go through a ramp utility with a minimum duration), so a channel-switcher feel cannot be introduced casually.

### 8.5 Settle / night / hidden-tab audio

Settle ramps the whole chorus bus down ~8dB and lowers spontaneous-call rates over ~5s alongside the lighting shift; one acknowledging soft call is scheduled (seeded). Night follows the same shape via the tod curves. When the tab goes hidden, audio fades out over 10s and the context suspends (Decision #8b: presence has ended, "the aviary lives where the user visits it," and hidden-tab timer throttling makes reliable scheduling expensive); it resumes with the snapshot pull on visibility.

### 8.6 Autoplay policy and the WebAudio fallback

Browsers block audio before a user gesture; "calls already audible on first frame" is not always reachable on a fresh load. Plan (Decision #8a): attempt `context.resume()` on load (succeeds for users with engagement history in Chrome); if blocked, the scene runs fully alive and silent, a small **audio-waiting glyph** shows in the top bar (system surface, not a banner — no modal, no "click to enable sound" prompt), and the first user gesture anywhere (pointerdown/keydown, captured passively) resumes the context with calls fading in over ~2s. This is the maximum the platform permits without an announcement surface.

If WebAudio is entirely unavailable (ancient browser, context creation fails, hardware): **graceful silence with captions enabled by default** for that session, per the PRD. There is no recorded-audio fallback path in the codebase — not a degraded one, not a temporary one; the absence is the enforcement.

---

## 9. Accessibility surfaces (shipped with v1, staffed from M1)

A dedicated accessibility-focused engineer owns these surfaces from the first vertical slice. They are designed surfaces, not retrofits; each has its own design review alongside the visual scene.

### 9.1 Screen-reader narration

- A client-side **narration composer** consumes the same snapshot + presentation events the renderer does and writes naturalist prose (from `@aviary/voice`) into a visually-hidden ARIA live region (`aria-live="polite"`, never `assertive`).
- Idle cadence: one prose update per `NARRATION_IDLE_S = 45s` (30–60 band), composed from current scene state — "a small grey bird is perched on the front rail, calling softly. it is morning in the aviary; the light is gentle." Not a state list, never "Pip mood: content."
- Event priority: return-greeting, offer reactions, settle, and the visiting bird narrate promptly via a priority queue that **replaces** pending idle updates (no queue flooding); even prioritized events are written as observations ("pip looks up as you arrive" is wrong — "pip looks up from the low perch, calling once" is right; second person is lint-blocked in naturalist templates).
- Voice continuity: same grammar library as the notebook, so the aviary and the notebook are one product to a screen-reader user.
- Generated client-side (Decision #17): it must narrate what is actually rendered, including locally-originated cosmetic bouts; server generation would drift from the presented scene.

### 9.2 Captions

Opt-in via accessibility settings; **on by default in the no-WebAudio fallback**. The call scheduler's motif trace (§8.2) renders to short prose — "a low trill, paused, low trill again" — composed at runtime from what was actually synthesized, with a mood-flavored adjective slot. Displayed as small text near the calling bird, fade in/out with the call, collision-avoided between simultaneous callers, on a subtle auto-contrast plate that guarantees AA against bright and dim scene states. Caption text mirrors into the narration region for screen-reader users with audio off.

### 9.3 Keyboard navigation and focus

- Tab order: top bar items → aviary scene (roving tabindex over birds, ordered front-to-back then left-to-right).
- In-scene: arrow keys move bird focus; Enter = listen-in on focused bird; Escape = disengage; focus loss disengages with the standard slow ramp.
- Offer panel: opened from the top bar (with a documented shortcut), full keyboard operability, focus-trapped while open, returns focus on close. Settle reachable in the top bar; its 5s undo is keyboard-accessible (any key with focus in scene, plus an announced undo affordance in the narration).
- Focus indicator: soft high-contrast outline tested against dawn/midday/dusk/night/settled palettes (visual regression matrix); `:focus-visible` so pointer users don't see rings.
- Hit targets ≥ 44px equivalent on touch; birds get generous focus/hit hulls beyond their sprite bounds.

### 9.4 Reduced-motion mode

Triggered by `prefers-reduced-motion` or the account-synced override. It is a second renderer mode, not a kill-switch:

- Motion layer swaps to **held poses with slow cross-fades** (800–1500ms) at a calm cadence; preening becomes a sequence of preen poses cross-fading; perch changes and flights become cross-fade relocations; greeting tiers map to pose-sequence equivalents.
- Ambient leaf/feather particles off; parallax sway off; day/night palette shifts remain, slowed; top-bar fade still works (opacity, not motion).
- Calls, captions, narration, drift, mood, notebook: identical to the full experience. The aviary is the aviary.
- It gets its own design pass and its own visual-regression suite; "reduced-motion looks broken" is a release blocker, not a polish item.

### 9.5 Contrast and copy

All user copy (top bar, panels, captions, narration-as-text, system surfaces) meets WCAG AA minimum, checked by automated contrast tests across palette states. The scene itself carries no user copy except captions (plated, §9.2). System surfaces (auth, errors, settings, sync) use the matter-of-fact registry; naturalist voice never appears in an error.

---

## 10. Accounts, privacy, security

### 10.1 Magic-link auth

256-bit random token, stored as SHA-256 hash; 15-minute expiry; single-use consumption under a row lock (replay-safe); request endpoint always returns 202 (no account-existence oracle); rate limits per email-HMAC (e.g., 5/hour) and per IP. Consuming a link creates the account on first sign-in (adoption flow follows) or signs into the existing one, issuing a session token (cookie) with sliding 90-day expiry, listed and revocable in settings. Email change: verify-new-then-commit; old address works until confirmation. No passwords, no SSO at v1. No new-device security emails at v1 (Decision #14) — the session list is the audit surface, and the product's no-email posture stays clean (transactional auth/export/invite mail only).

### 10.2 Deletion lifecycle

`POST /api/account/delete` → `deletion_requested_at` set; sessions other than the current revoked; outstanding visit invites revoked; **simulation pauses** (ticks skip soft-deleted accounts; state frozen — restoration resumes exactly where things stood, and absence was never punished anyway; Decision #13). Any signed-in page during the window shows the restore affordance ("I changed my mind"). At +30 days a worker hard-deletes: birds, vectors, events, notebook, invites, visit sessions, export bundles, queue rows — every record keyed by the account UUID. Backups age out on a ≤30-day retention, so total physical erasure completes ≤60 days; this bound is documented in the privacy policy text.

### 10.3 Export

`POST /api/account/export` → background job assembles JSON: account settings, birds (names, species, adoption dates, **current personality vectors**, current moods), notebook entries, visit log. Stored encrypted in object storage; a signed 7-day link is emailed to the verified address. Vectors are included because `accounts_sync.md` explicitly enumerates them in the export — this is a data-portability artifact engaged with as a system surface, not a product UI exposing numbers; the tension with the never-expose rule is recorded as Decision #11, and no product surface ever renders the exported values.

### 10.4 Event retention

Drift is a recursive filter (state carries the accumulated past), and mood looks back minutes — so raw `interaction_events` are needed only until processed plus a debugging margin. Partitions drop at 30 days (Decision #7). This converts the privacy stance into physics: per-bird interaction history older than a month does not exist anywhere, so it cannot leak, be subpoenaed into a new purpose, or tempt an aggregate analysis.

### 10.5 The telemetry boundary (architecture, not policy)

Two physically separate pipelines:

- **Ops pipeline**: metrics/traces/logs — request counts and latencies per endpoint, tick latency, queue depth, error rates, email delivery, anonymized session-duration histograms (no per-account dimension), client RUM (§13). Logs may carry the synthetic account UUID for support/debugging ("is this account having errors" — allowed); they never carry bird state, trait values, event payload contents, or email.
- **Simulation database**: never connected to any analytics warehouse, dashboard tool, or ML system. Enforced by: separate DB credentials with no read grant to telemetry infrastructure; network policy; a **telemetry schema registry** in CI with a field-name denylist (`bird*`, `trait*`, `boldness`, `mood`, `offer*`, `listen*`, `presence*`, `email`) that fails the build if a metric/log schema tries to carry them (§14.6); quarterly manual audit of emitted telemetry against the registry.

**Deliberately not measured, ever**: per-feature engagement funnels, retention cohorts tied to interaction behavior, offer/listen-in usage analytics, drift distributions across real accounts, notebook read rates, A/B tests on engagement, anything answering "how are birds typically interacted with." Endpoint-level request counts exist for capacity only and carry no event-type or account dimensions beyond the route name.

**Calibration without population data** (§5.4's targets still need measurement): synthetic cohort simulation in CI, plus a small set of internal volunteer accounts with explicit, documented consent flagged `calibration_consent = true` — the only accounts whose `bird_trait_log` is ever queried by a human. Production users' simulation data drives their simulation. Full stop.

### 10.6 Abuse and integrity

- Presence inflation (scripted pointermove): server clamps per-account presence to wall-clock union (§5.3) and the daily drift cap (§5.4) bounds the payoff to ~2× regular pace — the attack buys little and harms only the attacker's own aviary. No leaderboards exist for it to matter to.
- Invite spam: per-account invite rate limit (e.g., 10/day), per-recipient dedup, standard email-abuse reporting headers on invite mail.
- Security posture: strict CSP (self-hosted everything — zero third-party scripts, also a privacy stance); CSRF tokens; httpOnly cookies; KMS-managed secrets/pepper; dependency audit in CI; an external security review in M4 covering auth, visit-token scope isolation, and IDOR sweeps (all object access is session-derived; no enumerable ids in URLs).

---

## 11. Visits (read-only, quiet)

Flow: host enters friend's email → invite row + one-time link mailed → visitor opens link → token consumed (single-use; a re-click or expired/revoked link gets the matter-of-fact "this visit is no longer available" surface) → visitor cookie scoped to `/visit` (Decision #12: access persists for that browser until revocation or invite end; the *link* dies on first use) → visitor pulls `/visit/snapshot` on the same cadence as a host client.

What the visitor gets: the same snapshot pipeline minus greeting directive, cooldowns, and absence data — same birds, same moods, same drift-derived presentation, same weather and tod. **No special rendering exists**: the renderer has no "visitor flattering" mode, which is the implementation of no-show-off-mode. The client runs in render-only mode: presence sampler not loaded, event queue not constructed, offer/settle/notebook chrome not mounted, listen-in disabled (clicks do nothing). Enforcement is server-side scope (§4.5): the visitor cookie cannot reach `/api/*` at the router level, so even a modified client cannot write events, and a visitor's hour of watching drifts nothing.

Revocation: `revoked_at` set → visitor's next snapshot pull (≤45s, or the next heartbeat at 30s) returns 410 with the matter-of-fact surface. No notification to the host on revocation success; no notification to the host on visits at all unless they enabled the settings toggle (off by default, not surfaced in onboarding). Visit log in settings: visitor email, date, approximate duration (from heartbeats), outstanding invites — pull-only, no badge anywhere when new visits occur.

Not built, structurally: chat, visitor avatars/markers, comments, co-presence, discovery, visit counts surfaced anywhere outside the host's own log.

---

## 12. Performance engineering

### 12.1 Bundle budget (<2MB gz contractual; internal targets far lower)

Critical path (must paint first bird): bootstrap renderer + sky + species core + net/snapshot ≤ **250KB gz**. Full initial bundle target **1.2MB warn / 1.5MB CI fail** — headroom under the 2MB contract is the defense against erosion. Allocation guide: scene renderer 180KB, engine-shared 120KB, audio synth 100KB, voice lib 80KB, chrome+Preact 120KB, net/state/SW 60KB, species packs 150KB (SVG part-sets + parameter tables; no bitmaps over 30KB; no audio assets by definition), fonts 0 (system stack). Lazy chunks: settings, notebook panel, offer panel, invite flow, export/deletion surfaces. `size-limit` runs per-PR with per-chunk budgets.

### 12.2 Time-to-first-bird <500ms

Honest engineering reading (Decision #21): on mid-tier mobile over 4G, **<500ms is engineered and measured on the warm path** — the product's actual life, where the service worker serves shell + cached snapshot and the first frame is composed locally (target p75 < 400ms). The **cold first-ever load** cannot beat physics through 4G for any real bundle; its budget is **p75 < 2.0s to first bird**, with the quiet field (a designed aviary state, not a loader) covering the gap, critical-path ≤250KB, preload hints, edge-served shell, regional API snapshot p99 < 150ms server-side. RUM marks (`first-bird-render` via PerformanceObserver) are segmented warm/cold from day one; if product wants cold <500ms it requires an inline-snapshot-in-HTML architecture (authenticated edge compute) noted as a post-v1 option.

### 12.3 60fps idle on a 5-year-old laptop

Techniques: per-layer offscreen caching with dirty compositing; pose sprites re-rastered only on pose change; zero per-frame allocations on the hot path (pooled vectors, preallocated draw lists); no shadowBlur/filter per frame; particles capped at 6; rAF-driven with frame-time histogram; if sustained long frames are detected, ambient density degrades before bird motion ever does. The perf rig (§14.7) runs a 2018-class CPU-throttled profile and fails on p95 frame time > 16.7ms over a 5-minute idle scene; this is a runtime budget, so the rig also samples minute 25 of the soak, not just minute 1.

### 12.4 Zero memory growth over 30 minutes (CI gate)

Pools everywhere allocation recurs: audio voices (§8.1), particles, pose buffers, draw lists, narration strings via ring buffer. Notebook list virtualizes; scrolled-out entries release. Snapshot history capped at 2. The CI soak: headless Chromium, scripted 30-minute session (idle + periodic listen-in/offers + notebook scrolls), heap snapshot at minutes 3 and 30 after GC; fail on growth > 2MB or any monotonic detached-node trend. Runs nightly and on release branches.

### 12.5 Browser matrix

Last two major versions of Chrome, Safari, Firefox, Edge — CI smoke + the perf rig on Chromium, manual matrix passes per release including iOS Safari (visibility/lifecycle quirks: §5.3 sampler is re-validated on `pageshow`/bfcache restore). Older browsers get the matter-of-fact unsupported surface at boot via feature detection (Canvas2D, ES2022, cookies; WebAudio absence alone does *not* gate — that's the silence fallback).

---

## 13. Observability and SLOs

- **Server metrics**: snapshot p50/p99 (SLO p99 < 250ms), event ingest p99 < 150ms, **tick latency p50/p99 with the PRD alarm at p99 > 5s**, tick queue depth/lag, fast-forward batch sizes, email delivery success and latency (p95 < 60s), error rates by route, DB health. Availability SLO 99.9% on snapshot/event paths.
- **Synthetic fleet**: scheduled headless browsers from 4–6 geographies running scripted sessions against prod: first-bird timing (warm and cold), audio-context init success, snapshot latency, keyboard-path smoke, caption render. Runs every 15 minutes; alerts on budget breach before users feel it.
- **RUM (self-hosted, aggregate-only)**: first-bird-render mark, long tasks, FPS sample histograms, audio init/fallback rates, SW cache hit rate, JS error counts — no per-account dimensions, schema-registry enforced (§10.5). Sampled (e.g., 10%) since aggregates are the only consumer.
- **Dashboards/alerts**: one product-health board (budgets vs. actuals), one sim board (tick health, drift-cap saturation counts as an aggregate sanity signal), one delivery board (email). On-call: business-hours rotation pre-GA, 24/7 from GA; runbooks for tick backlog, email provider failover, DB failover.
- **What observability deliberately omits**: everything in §10.5's "never measured" list. The RUM schema is reviewed against the registry like any other telemetry.

---

## 14. Testing and CI strategy

1. **Engine unit + property tests**: monotonic drift (∀ event sequences: traits non-decreasing — fast-check property), daily-cap enforcement, presence interval-union correctness (overlapping multi-device pings), offer-resolution determinism (same seed ⇒ same outcome, server vs. client build), mood hysteresis (no flapping under noisy inputs), event idempotency.
2. **Golden simulation personas** (CI, synthetic): daily-20-min, weekend-only, two-week-absence-and-return, marathon-8-hour, two-device-overlap. Assertions: §5.4 calibration windows (instrument-measurable ≈ week 1; one quantization level ≈ 3 weeks ±30%), absence produces zero trait change and rhythm-only quieting with ≤3-session recovery, marathon ≤ 2× regular pace. **Fast-forward equivalence**: N random event logs replayed tick-by-tick vs. batched — bit-identical state required.
3. **Sync/integration**: two simulated clients interleaving events with reordered delivery and retries → identical final state vs. serial baseline; suspend/resume reconciliation; revoked-session and revoked-visit flows.
4. **Voice lint (build-failing)**: naturalist templates — lowercase, present tense, no second person, no exclamation, denylist (`welcome`, `streak`, `achievement`, `badge`, `level`, `score`, `congrat*`, `days in a row`, `you visited`); system copy in system registry only; identifier-vocabulary lint (§0). Notebook generator fuzz: 10k generated entries → zero denylist hits, repetition rate under threshold, sparsity simulation (hyperactive aviary still ≈ 1 entry/3 days).
5. **Audio tests**: OfflineAudioContext renders — signature stability (same bird across all moods and trait levels ⇒ spectral-signature distance under threshold; different birds ⇒ above threshold), repetition detector (autocorrelation across 50 consecutive calls of one bird ⇒ no near-identical pair), chorus onset-separation rule, ramp-floor rule (mix never below floor during listen-in). Plus human listening panels at M2 and M4 (§15) — uncanniness is not unit-testable.
6. **Privacy CI**: telemetry schema registry denylist (§10.5); import-boundary lint (only `sim/tick` may import the trait-write DAO; nothing outside `telemetry/` may import metric emitters with free-form fields); an integration test asserting the analytics sink receives zero per-account-keyed records during a full simulated session; retention-drop test for event partitions.
7. **Perf CI**: `size-limit` per chunk (§12.1); first-bird timing on a throttled profile (warm and cold); frame-time rig (§12.3); 30-minute memory soak (§12.4); audio CPU sample.
8. **Accessibility CI + manual**: axe on every chrome surface and palette state; keyboard-path integration tests (full session start-to-settle without a pointer); narration snapshot tests (scene state ⇒ prose passes voice lint, cadence ≤ limits, no queue flooding); reduced-motion visual regression suite. Manual matrix per release: VoiceOver+Safari (macOS/iOS), NVDA+Firefox, NVDA+Chrome, JAWS+Chrome.
9. **Security**: authz tests on every route (session scoping, visitor-scope isolation), magic-link replay/expiry, rate limits, CSRF; M4 external review.
10. **Release checklist** (human, per release): "does any new surface announce rather than notice?"; "does any new copy bypass `@aviary/voice`?"; "does any new metric carry a per-account or per-bird dimension?"; "does any wait state bypass the quiet field?"

---

## 15. Rollout

### 15.1 Milestones (exit criteria, not date promises; ~26 weeks to GA)

- **M0 — Foundations (wk 1–3)**: repo + module boundaries + import lints; CI skeleton with size-limit, voice lint, schema registry live from the first commit (gates are cheapest before there's anything to grandfather); auth (magic link, sessions); schema v1 + migrations; walking skeleton: static scene + snapshot round-trip + event ingest. *Exit*: a signed-in user sees a static aviary served through the real pipeline.
- **M1 — Alive vertical slice (wk 4–8)**: tick v1 (presence, drift v1, mood v1), two starter birds + adoption, idle micro-motion, procedural calls v1 (2 species), return-greeting, day/night, quiet-field loading, SW warm start; narration v0 and keyboard focus v0 land here (a11y is in the slice, not after it). *Exit*: internal dogfood — "does it feel alive?" review with the design team; first listening session.
- **M2 — Interactions + voice surfaces (wk 9–13)**: listen-in mix, offers (all three kinds, deterministic resolution), settle + undo, full mood machine + weather + bird-to-bird, notebook generator + panel, captions, narration v1, reduced-motion renderer v1, all six species + motif libraries. *Exit*: listening panel #1 (call uncanniness gate); voice-lint corpus complete; dogfood cohort to ~30 internal users.
- **M3 — Accounts + sync hardening + visits (wk 14–17)**: multi-device test matrix green (interleaving, suspend/resume, interval-union presence), export, deletion lifecycle, email change, visits behind a flag end-to-end, dormancy fast-forward + equivalence suite, drift calibration sprint #1 with accelerated-clock synthetic cohorts and consented dogfooders. *Exit*: two-device demo indistinguishable; calibration within ±30% of targets.
- **M4 — Hardening (wk 18–21)**: perf to budget on the device lab (real mid-tier Android + 2018 laptop), memory soak green 10 nights running, chorus tuning at 3/5/7 birds with accelerated-age test aviaries + **listening panel #2 including the 7-bird recognizability test** (can panelists pick Pip out by ear?), full manual a11y matrix + fixes, external security review, privacy audit against §10.5, failure-mode drills (§6.3). *Exit*: all CI gates green at release strictness; go/no-go for beta.
- **M5 — Private beta → GA (wk 22–26+)**: a few hundred invited users; watch ops dashboards and the aggregate sanity signals only (no engagement analytics exist to watch — by design); fix; expand; GA when SLOs hold for 2 weeks at beta scale and the synthetic fleet is green across geographies.

### 15.2 Ramping birds-per-aviary

Organic ramp is built into the product: every GA-cohort aviary starts at two birds, and the first third-bird offers fire ~90 days post-signup. Engineering uses that runway deliberately: accelerated-age test aviaries exercise 3–7-bird choruses and scenes throughout M3–M4; **a chorus-quality re-review is scheduled before GA+90d** (the first organic third birds) and again before the population can reach five. A global flag can pause new-bird offers silently (offers simply don't appear; nothing announces a pause) if mix work needs time. The cap stays 7 at the engine; any future change is a calibration project, not a config flip.

### 15.3 Instrumented from day one

First-bird timings (warm/cold), bundle sizes per release, tick latency + queue depth, snapshot/event latencies, audio init + fallback rates, email deliverability, error rates, SW hit rate, drift-cap saturation aggregate count, synthetic-fleet results. (And the standing list of what is never instrumented: §10.5.)

### 15.4 Post-launch calibration policy

Drift constants are versioned (`calibration_v`). Changes apply prospectively only — existing trait values are never rescaled or migrated (identity continuity includes drift history). Any retune ships behind a sim-replay comparison on synthetic personas plus a consented-dogfood observation window. Mood/grammar tuning is unrestricted (fast-timescale, no accumulated state); personality-path changes are treated with schema-migration gravity.

---

## 16. Risks and mitigations

| # | Risk | Blast radius | Early signal | Mitigation |
|---|---|---|---|---|
| 1 | **Drift miscalibration** (too fast = Tamagotchi; too slow = screensaver) | The core promise | Persona CI drift outside ±30%; dogfood "nothing changes" / "changed overnight" reports | Named tunables in one table; persona CI gates; accelerated-clock cohorts; consented-dogfood observation; prospective-only retunes (§15.4) |
| 2 | **Presence signal corruption** (browser quirks: iOS lifecycle, bfcache, focus-vs-visibility differences, touch devices) | Silent population-wide drift distortion — the PRD's named nightmare | Cross-browser presence conformance suite diffs; aggregate presence-minutes sanity trend | Conjunction sampler with per-browser conformance tests; touch-input adaptation (Decision #4); `pageshow` revalidation; server wall-clock clamps + interval union |
| 3 | **Sync correctness bugs** (lost drift, double-processing) | Invisible relationship damage; "bird feels reset" | Equivalence/property suite failures; support reports of regressed birds | Single-writer row lock; additive-delta-only schema; idempotent ingest; deterministic replay for forensic reconstruction; `bird_trait_log` audit |
| 4 | **Audio uncanniness** (synthetic calls feel canned or alien) | The affective spine | Listening panels score below gate; repetition detector trips | DSP-experienced engineer + audio designer from M1; reference-recording spectral analysis targets; jittered grammar; two panel gates (M2, M4); silence-with-captions is the floor, never recorded audio |
| 5 | **Chorus blur below 7 birds** (signatures not separable at 5+) | Bird-count roadmap | M4 recognizability test failure | Signature-distance automated metric + human panel before any organic 3rd bird (§15.2); pause-offers flag; mix work (pan separation, onset spacing) |
| 6 | **Autoplay restrictions** mute the first session | First-impression aliveness | Audio-init RUM rate by browser | §8.6 gesture-resume design; visual aliveness carries silent seconds; warm-path users mostly unaffected |
| 7 | **Cold-load TTFB physics vs. 500ms budget** | Perceived "loading product" | Cold-path RUM p75 | Warm/cold split with SW (§12.2); 250KB critical path; quiet field as designed state; post-v1 edge-inline option documented |
| 8 | **Perf erosion over releases** (bundle creep, frame regressions, leaks) | Mid-tier users lose 60fps/budget | CI budget gates trending toward limits | Hard CI fails with headroom margins; nightly soak; device-lab releases; ambient-density degradation order (§12.3) |
| 9 | **A11y regression / narration drift into state-lists** | Product integrity for a11y users | Narration snapshot lint; manual matrix findings | Owned surface with named engineer; narration through `@aviary/voice` only; matrix per release; reduced-motion regression suite as release blocker |
| 10 | **Privacy boundary erosion** ("just one engagement metric") | Trust; the product's stated contract | Schema-registry CI rejections; audit findings | Registry denylist; no warehouse connection to sim DB; 30-day event retention makes later aggregation physically impossible; release-checklist question |
| 11 | **Notebook/narration repetition or voice breaks** | Charm engine | Fuzz repetition rate; lint hits; dogfood "it said the same thing" | Template cooldowns + no-repeat memory; corpus growth as content work each milestone; generator fuzz in CI |
| 12 | **Magic-link deliverability** (spam-foldering = login outage) | Auth availability | Delivery-rate dashboards; signup-funnel drop | Reputable provider + warmed domain, SPF/DKIM/DMARC; SES fallback with automatic failover; resend affordance with matter-of-fact copy |
| 13 | **Tick backlog at scale** (per-minute × accounts) | Staleness; latency SLO | Queue-lag metric | Dormancy fast-forward (§5.1) keeps active-set bounded; horizontal workers; synchronous catch-up on read as backstop |
| 14 | **Announcement-surface creep** (a "harmless" toast/badge/counter lands) | The product's identity, per the PRD's own warning | Release checklist; PR review | No toast/badge primitives in the library; voice lint; non-goals enforced in code review culture with §1.2 as the citation |
| 15 | **Timezone edge cases** (travel, DST, two devices in different zones) | Mood/tod oddities | Support reports; tod-phase mismatch logs | Last-reported-tz wins (rare, benign); tod computed server-side once per snapshot; DST handled by IANA lib; no user-visible clocks anywhere to contradict |

---

## 17. Decision log (defensible calls on PRD-open points)

1. **TypeScript end-to-end, modular monolith** — forced by the three isomorphic libraries (grammar, deterministic resolution, voice) that must be bit-identical on both sides (§2.1–2.2).
2. **Custom Canvas 2D renderer, no game engine; Preact for chrome only** — bundle and control; WebGL kept behind the renderer interface as contingency.
3. **Snapshot polling (45s + visibility/long-gap triggers), no WebSocket at v1** — matches the PRD's pull model; the tick cadence makes push latency pointless; one less stateful tier.
4. **Presence activity window 240s default (calibrate 2–6 min, leaning long); touch devices count pointerdown/scroll as activity** — the PRD's pointermove/keypress conjunction, translated honestly to coarse pointers rather than applied stricter-than-intended.
5. **Account timezone = last client-reported IANA zone** — server tick needs a tz; rare multi-tz races are benign (Risk #15).
6. **Traits ship to clients only as 7-level quantized presentation parameters** — "never exposed numerically" must survive devtools; levels also define the user-visible drift threshold (§4.3).
7. **Interaction-event retention 30 days** — drift is recursive, mood is short-window; deleting raw history is privacy as physics (§10.4).
8. **Audio**: (a) autoplay handled by silent-alive start + passive gesture resume + small top-bar state glyph, no prompt; (b) hidden tab fades audio out over 10s and suspends (§8.5–8.6).
9. **New-bird offer = visiting-bird pattern** — appears on the far perch, welcomed via the offer affordance; the one design that grows the aviary without announcing anything (§5.8).
10. **Bird rename lives in settings (birds list)** — clicking a bird must stay listen-in; settings is the established system surface.
11. **Export includes raw personality vectors** — `accounts_sync.md` enumerates them explicitly; treated as a data-portability system artifact; tension with the never-expose rule recorded, no product UI renders them (§10.3).
12. **Visit link is one-time; visitor access then persists per-browser until revocation/expiry** — "one-time link" read as anti-sharing, not as single-viewing; revocation remains instant-effect (§11).
13. **Soft-deleted accounts pause simulation** — frozen, not ticking; restore resumes exactly; consistent with never-punishing absence (§10.2).
14. **No new-device security emails at v1** — session list is the audit surface; keeps the no-email posture clean (§10.1).
15. **Multi-device presence = interval union per account** — watching on two screens is one attention, not double drift (§5.3).
16. **Calibration measurement uses synthetic cohorts + explicitly consented internal accounts only** — the privacy boundary forbids population-level interaction analysis even for our own tuning (§10.5).
17. **Narration generated client-side** — it must describe the actually-rendered scene, including locally-originated cosmetic bouts (§9.1).
18. **Mood set finalized as {alert, curious, content, drowsy, wary}; `roosting` is a night behavior, not a mood** (§5.2).
19. **Postgres-only infrastructure at v1 (SKIP LOCKED queues; no Redis)** — fewer moving parts; the tick's transactionality is the point (§2.1).
20. **Neglect-quieting implemented as a fast-recovering expression-rhythm EMA outside the personality vector** — observable "ambient" behavior with zero trait penalty, the engine-level reconciliation of monotonic drift with "greets less often" (§5.4).
21. **The 500ms first-bird budget is engineered on the warm path; cold loads get a 2.0s budget over the quiet field** — physics of 4G vs. authenticated per-account state; both paths measured separately in RUM from day one (§12.2).
22. **Single region at v1** — sync correctness over geo-latency; SW warm path keeps far-user experience inside budget; multi-region is a post-v1 project with the single-writer invariant preserved.

---

## Appendix A — Named tunables (calibration surface)

| Constant | Default | Band | Owner section |
|---|---|---|---|
| `TICK_S` | 60s | 30–120s | §5.1 |
| `PRESENCE_ACTIVITY_WINDOW_S` | 240 | 120–360 | §5.3 |
| `PRESENCE_PING_S` | 30 | 15–60 | §5.3 |
| `SNAPSHOT_POLL_S` | 45 | 30–60 | §4.2 |
| drift weight: presence→plumage | 3.0e-4/min | ±50% | §5.4 |
| drift weight: presence→warmth/bold/vocal | 1.5e-4/min | ±50% | §5.4 |
| drift weight: listen-in→warmth/vocal | 6.0e-4/min | ±50% | §5.4 |
| drift weight: offer-accept→curiosity | 2.5e-3 | ±50% | §5.4 |
| drift weight: offer-near→boldness | 8.0e-4 | ±50% | §5.4 |
| `DRIFT_DRAIN_TAU` | 36h | 24–72h | §5.4 |
| `DRIFT_DAILY_CAP` | 0.012/trait | 0.008–0.02 | §5.4 |
| trait quantization step | 1/7 (~0.10) | fixed v1 | §4.3 |
| rhythm τ down / up | 10d / 2d | ±50% | §5.4 |
| rhythm floors (greet / call) | 0.45 / 0.60 | ≥0.3 | §5.4 |
| `MOOD_HYSTERESIS` / `MOOD_MIN_DWELL` | 0.15 / 15min | — | §5.5 |
| greeting gap threshold | 90s | 60–300 | §5.6 |
| greeting tiers | 30min / 24h | — | §5.6 |
| `OFFER_COOLDOWN_S` | 300 | 180–600 | §5.7 |
| bird-offer age thresholds | 90d, then 90–120d | ±30d | §5.8 |
| `NOTEBOOK_RATE` | ~1 / 3 days | 2–5 days | §5.9 |
| `LISTEN_RAMP_S` / ambient bed | 2.0s / −14dB (floor −18dB) | — | §8.4 |
| `NARRATION_IDLE_S` | 45 | 30–60 | §9.1 |
| weather: rain / wind frequency | 2.5/wk / 1/day | ±50% | §5.5.1 |
| bundle: critical / warn / fail | 250KB / 1.2MB / 1.5MB | ≤2MB contract | §12.1 |
| first-bird warm / cold (p75) | 400ms / 2.0s | warm ≤500ms | §12.2 |
| tick latency alarm | p99 > 5s | PRD-fixed | §13 |

## Appendix B — Surface-to-voice assignment

Naturalist register: aviary scene, greeting, captions, narration, notebook, offer prompts, adoption naming, visiting-bird flow. Matter-of-fact register: sign-in/magic-link mail, session/timeout/sync errors, account settings, accessibility settings, export/deletion, visit invitations and revocation surfaces, unsupported-browser page, privacy policy. Rule for new surfaces: engaging the system as a system (identity, errors, settings, money) ⇒ matter-of-fact; everything else ⇒ naturalist. No surface may import both registries.
