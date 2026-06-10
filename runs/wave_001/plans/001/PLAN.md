# Pocket Aviary — v1 Implementation Plan

This plan turns the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`) into an executable engineering plan. It does not restate the spec; it makes the decisions the spec leaves open, attaches mechanisms and numbers to the constraints it pins down, and sequences the work.

Numbered decisions on ambiguous points are marked inline as `(D#)` and collected in §17. Constants marked **[tune]** are starting values owned by the calibration harness (§14), shipped as server config so they can change without deploys.

---

## 0. Plan-wide invariants

Every workstream below inherits five non-negotiable invariants. They appear once here and are referenced by number.

- **I1 — Server is the only writer of personality state.** Clients submit events; the simulation tick computes additive deltas; no other code path mutates a personality vector. Enforced at the database-permission level (§6.3), not by convention.
- **I2 — Drift is monotonic toward expressive.** No code path ever decreases a trait. Neglect is expressed through the decaying expressiveness modulator (§5.4), never through trait values. Enforced by a DB CHECK-style guard in the tick plus a property test asserting non-decrease over arbitrary event streams.
- **I3 — Presence is the three-way conjunction.** `visibilityState === 'visible'` AND window focus AND input activity within window W. No surface, metric, or shortcut may use a laxer definition (§5.3).
- **I4 — The privacy boundary is architectural.** Per-account interaction state lives only in the simulation database; the telemetry/analytics path has no per-account dimension and no read path into the simulation DB (§11).
- **I5 — No announcement surfaces, no gamification surfaces, ever.** No toasts, banners, welcome text, streaks, counters, badges, levels, or visit-frequency displays. Enforced by a copy-lint denylist and a non-goals checklist in the PR template (§10, §16 R11).

---

## 1. Scope

### 1.1 In scope for v1

| Area | Contents |
|---|---|
| Aviary | Single horizontal scene, three perch zones, day/night by account-local time, rare ambient weather, ambient micro-motion, no in-scene chrome, fading top bar |
| Birds | 2 starters (system-selected species, user-named), cap 7, ~6-species pool, stable identity, hidden personality vector (5 traits), mood machine, age-gated new-bird offers |
| Engine | Server-side tick (~1/min active), additive drift, expressiveness modulator, mood transitions, weather generation, notebook generation, greeting planner, call-policy generation |
| Interactions | Presence accounting, return-greeting, listen-in, offers (seed / song fragment / still pool, per-bird cooldown), settle (+5s undo), field notebook (read-only, sparse) |
| Audio | Client-side procedural call synthesis (WebAudio), per-bird signatures, real chorus mixing, listen-in mix ramps, graceful-silence fallback with captions |
| Accounts | Magic-link auth, per-device revocable sessions, email change with verification, synthetic UUID account IDs, JSON export, soft→hard deletion |
| Sync | Single canonical server state, snapshot polling, append-only event log, multi-device coherence |
| Social | Email visit invitations (off by default), read-only ambient visitor view, revocation, 30-day invite expiry, visit log, opt-in visit notifications (off by default) |
| Accessibility | Screen-reader narration (naturalist prose, slow cadence), reduced-motion mode (designed cross-fade rendering), call captions, full keyboard navigation, WCAG AA contrast |
| Performance | <2MB gz initial bundle, <500ms time-to-first-bird, 60fps idle on 5-year-old laptop, zero memory growth over 30 min (CI-tested) |
| Ops | Synthetic browser fleet, aggregate-only RUM, tick-latency alarms, deletion runbook |
| Minimal chrome | One quiet landing/sign-in page (system voice), unsupported-browser surface |

### 1.2 Out of scope for v1 (and structurally foreclosed)

Native apps; payments; shared/multi-aviary accounts; customizable scenes; public discovery, profiles, follows, feeds, comments, chat, co-presence; leaderboards (the underlying stats are not even computed — §11); achievements/streaks/levels/scores/visit-frequency surfaces of any kind; push notifications and unsolicited email about the aviary; Tamagotchi mechanics (death, hunger, distress, decaying meters); recorded-audio fallback; anti-cheat hardening of presence (D18); SSO/passwords.

"Foreclosed" is literal: §10's copy lint blocks announcement/gamification lexicon, §11's pipeline design never aggregates the data a leaderboard would need, and I2 makes punitive drift unrepresentable.

---

## 2. System architecture

### 2.1 Shape

A deliberately boring modular monolith plus one worker fleet, one Postgres, one CDN. No microservices, no Kafka, no Redis at v1 (D19). The interesting engineering lives in the simulation tick and the client; the platform around them should be as plain as possible.

```
 Browser (host)                      Browser (visitor)
 ┌────────────────────────┐          ┌──────────────────┐
 │ scene renderer (canvas)│          │ same client,      │
 │ audio engine (WebAudio)│          │ render-only mode  │
 │ presence tracker       │          └────────┬─────────┘
 │ snapshot client        │                   │ GET snapshots only
 │ event queue (batched)  │                   │
 └───────┬───────▲────────┘                   │
   POST events  GET snapshot (poll)           │
         │      │                             │
 ┌───────▼──────┴─────────────────────────────▼──────────┐
 │ aviary-api (Node/TS modular monolith)                  │
 │  auth · accounts · snapshots · event ingest · invites  │
 │  notebook read · export/delete · settings              │
 └───────────────┬───────────────────────────────────────┘
                 │ SQL (api_role: column-restricted, I1)
 ┌───────────────▼───────────────┐   ┌────────────────────┐
 │ Postgres (simulation DB)      │◄──┤ tick-worker fleet   │
 │  accounts · birds · events    │   │ (tick_role)         │
 │  ticks · notebook · invites   │   │ due-queue consumer  │
 └───────────────────────────────┘   └────────────────────┘
 ┌───────────────────────────────┐   ┌────────────────────┐
 │ CDN (static, immutable assets)│   │ transactional email │
 └───────────────────────────────┘   │ provider (links)    │
                                     └────────────────────┘
 Telemetry path (I4): client RUM beacons + server metrics → metrics store.
 No account IDs, no per-bird fields, no connection to the simulation DB.
```

### 2.2 Technology choices (D19)

- **Language:** TypeScript end-to-end. One monorepo: `packages/shared-schema` (zod schemas for events, snapshots, config — single source of truth for both sides), `packages/sim-core` (pure drift/mood/grammar math, isomorphic, no I/O — runs in the tick worker, the calibration harness, and dev tools), `apps/api`, `apps/tick-worker`, `apps/client`.
- **API:** Node 22 + Fastify. JSON over HTTPS. No GraphQL — the API surface is small and snapshot-shaped (§4).
- **DB:** Postgres 16. The event log is a partitioned append-only table, not a message bus. Column-level grants implement I1.
- **Client:** no game engine, no React for the scene. A custom canvas renderer (~40KB gz budget) plus Preact (~4KB) for DOM chrome (top bar, settings, notebook, adoption). Rationale: the scene is ≤7 birds + ornaments on one screen; an engine spends bundle budget (§12) on capabilities we don't use.
- **Audio:** raw WebAudio with a small voice-pool engine; AudioWorklet only for the noise/breath component if feature-detection passes, with a BiquadFilter-based fallback.
- **Email:** one transactional provider behind an internal interface (magic links, visit invites, export links). Provider choice is an ops decision; the interface ships first.
- **Hosting:** containerized API + workers in one region at launch; CDN for static assets; HTML served from origin (it embeds per-user state, §7.2). Multi-region is explicitly deferred.

### 2.3 Environments

`dev` (local, docker-compose Postgres, time-compression console enabled), `staging` (full stack + synthetic fleet + sim console), `prod` (no debug surfaces of any kind — the sim console and any vector-inspection tooling are compile-time excluded from prod builds; this is part of honoring "personality is never exposed," D20).

---

## 3. Data model

Postgres schema, simulation DB. All IDs are UUIDv7 unless noted. All timestamps UTC (`timestamptz`); the account's IANA timezone converts to local time where the engine needs it (D16).

### 3.1 Identity & accounts

```sql
accounts (
  id uuid PK,                        -- synthetic ID; the ONLY identifier used anywhere (I4)
  email_encrypted bytea NOT NULL,    -- AES-GCM, KMS-managed key; stored here and nowhere else
  email_hash bytea UNIQUE,           -- HMAC for sign-in lookup w/o decryption
  timezone text NOT NULL DEFAULT 'UTC',   -- IANA; most recent device report wins (D16)
  settings jsonb NOT NULL DEFAULT '{}',    -- captions, reduced-motion override, visit-notify opt-in…
  created_at timestamptz,
  deletion_requested_at timestamptz NULL,  -- soft-delete marker; hard delete at +30d (§4.6, D11)
)
auth_links    (id, account_id_or_email_hash, token_hash, purpose enum(signin,email_change,export,invite_delivery), expires_at, consumed_at)
sessions      (id, account_id, token_hash, device_label, created_at, last_seen_at, revoked_at)
```

Rules: no foreign system (logs, metrics, queues, support tools) ever sees `email_*`; every cross-reference is `accounts.id`. Magic-link and invite tokens are stored **hashed only**; raw tokens exist only in the email. Link TTL 15 min, single consumption.

### 3.2 Aviary & birds

```sql
aviaries (
  id uuid PK, account_id uuid UNIQUE,        -- one aviary per account
  created_at timestamptz,                    -- drives age-gated bird offers
  expressiveness real NOT NULL DEFAULT 0.8,  -- E ∈ [0.35, 1], tick-maintained (§5.4)
  presence_ewma real NOT NULL DEFAULT 0,     -- input to E
  settled_until timestamptz NULL,            -- settle state (§5.9)
  weather jsonb NOT NULL DEFAULT '{}',       -- current/scheduled events (§5.7)
  bird_offer jsonb NULL,                     -- pending age-gated offer state (§5.10, D12)
  last_tick_at timestamptz, next_tick_at timestamptz, tick_no bigint,
  events_high_watermark bigint NOT NULL DEFAULT 0,
  suspended boolean NOT NULL DEFAULT false   -- true during soft-deletion (D11)
)

birds (
  id uuid PK,                  -- stable identity, never reused or regenerated (engine invariant)
  aviary_id uuid,
  species_id smallint,         -- references the static species pool (code-shipped, not a table)
  name text NOT NULL,          -- user-assigned, renameable; only client-writable bird column
  adopted_at timestamptz,
  rng_seed bigint NOT NULL,    -- derived from id at creation; drives signature + variation streams
  -- personality vector: server-only columns (I1), CHECK (x BETWEEN 0 AND 1)
  boldness real, social_warmth real, vocal_freq real, plumage_sat real, curiosity real,
  drift_headroom_today real NOT NULL DEFAULT 0.01,  -- daily drift cap accounting (§5.2)
  mood text NOT NULL,          -- enum: alert|curious|content|wary|drowsy|resting (D7)
  mood_since timestamptz,
  perch_zone smallint,         -- 0 front / 1 middle / 2 back, tick-chosen
  last_offer_at timestamptz NULL    -- per-bird offer cooldown (§5.8)
)
```

### 3.3 Events (append-only)

```sql
events (
  seq bigserial,                -- server order; governs tick processing (D2)
  id uuid UNIQUE,               -- client-generated idempotency key
  aviary_id uuid, session_id uuid,
  type text,                    -- presence_span | listen_in_start | listen_in_end | offer
                                -- | settle | settle_undo | re_engage | adopt | rename | tz_report
  payload jsonb,
  client_ts timestamptz,        -- advisory only (untrusted clocks, D2)
  server_ts timestamptz DEFAULT now()
) PARTITION BY RANGE (server_ts);  -- monthly partitions; retained (they are the user's history)
```

Visitors have no write path to this table at all (§4.5).

### 3.4 Notebook, invites, visits, ticks

```sql
notebook_entries (id, aviary_id, created_at, body text, kind text, sparsity_score real)
invites (id, aviary_id, visitor_email_encrypted bytea, token_hash, status enum(outstanding,active,revoked,expired),
         created_at, expires_at,            -- created_at + 30d
         consumed_at, visitor_cookie_hash)  -- single consumption, browser-bound (D9)
visits  (id, invite_id, started_at, last_poll_at)   -- approx duration = last_poll − start; feeds visit log
ticks   (aviary_id, tick_no, ran_at, events_through bigint, duration_ms)  -- ops/idempotency record
export_jobs, deletion_jobs (id, account_id, state, timestamps)            -- async job rows
```

### 3.5 Static content (shipped in code, versioned)

Species pool (~6 species): silhouette + pose-parameter set, default palette, call-grammar motif library, night-activity flag (exactly one nightjar-like species). Notebook/narration/caption template fragment libraries with register tags (§10). Server config: all **[tune]** constants.

---

## 4. API surface

All endpoints JSON over HTTPS, session cookie auth (httpOnly, Secure, SameSite=Lax), CSRF token on mutations, per-session and per-IP rate limits. Errors return machine-readable codes; user-facing error copy is the client's, in the matter-of-fact register (§10).

### 4.1 Auth

```
POST /v1/auth/link        {email}            → 202 always (no account enumeration)
POST /v1/auth/consume     {token}            → sets session cookie; creates account+aviary+2 birds
                                               on first sign-in (adoption pending state)
GET  /v1/sessions                            → device list   DELETE /v1/sessions/:id → revoke
POST /v1/account/email-change {new_email}    → verification link to new address; old works until verified
```

### 4.2 Snapshot pull (the read path)

```
GET /v1/aviary/snapshot?known_tick=<n>
```

Behavior: if `last_tick_at` is stale (> 2× active cadence), the API runs an inline bounded catch-up tick first (D1), so every served snapshot is current — visitors and returning hosts never see a lagged aviary. `known_tick` enables a cheap 304-style short response when nothing changed.

Response (illustrative, ~2–6KB gz):

```jsonc
{
  "tick": 48211, "server_time": "…", "account_tz": "America/New_York",
  "aviary": {
    "settled": false,
    "lighting": {"mode": "day_cycle"},            // client computes curve from account_tz (D16)
    "weather": {"active": null, "scheduled": [{"type":"rain","start":"…","dur_s":140,"intensity":0.4}]},
    "expressiveness": 0.74
  },
  "birds": [{
    "id": "…", "species": 3, "name": "pip", "perch_zone": 0,
    "mood": "curious", "mood_since": "…",
    "render": {"size": 0.92, "palette_shift": 0.31, "plumage_detail": 0.55},  // derived, never raw traits
    "call_policy": {                                // server-derived params; calls are client ephemera (D3)
      "rate_per_min": 0.8, "respond_p": 0.55, "chorus_join_p": 0.4,
      "motif_weights": [0.2,0.5,0.1,0.2], "intensity": 0.6, "active": true,
      "stream_seed": 991188                          // per-interval PRNG stream
    },
    "greeting": null                                 // or a greeting plan, present on absence-gap (§5.6)
  }],
  "narration_seed": 5523,
  "notebook_cursor": "…"                             // newest entry id; full entries fetched lazily
}
```

Note what is absent: raw personality values never leave the server (the `render` block carries derived visual parameters only). Mood is exposed because mood is legible on-screen anyway; traits are not.

Poll triggers (client): tab becomes visible; render-frame gap > 5s (suspend/resume); keepalive every 45s while visible **[tune]**; after submitting an interaction event batch; `greeting` consumed.

### 4.3 Event ingest (the write path)

```
POST /v1/aviary/events   [{id, type, payload, client_ts}, …]   → 202 {accepted: [ids]}
```

At-most-once effect via the `id` idempotency key; batches ≤ 32. Terminal events (`presence_span` close, `settle`) use `navigator.sendBeacon` on `pagehide`. Server validates payload shape per type (shared zod schema) and stamps `server_ts`/`seq` (D2). Clients never send state — only events ("user listened in on Pip for 3 minutes," never "set warmth to x"; I1).

Event types and payloads:

| type | payload | notes |
|---|---|---|
| `presence_span` | `{start, end}` | closed spans of continuous presence, ≤60s each (§5.3) |
| `listen_in_start/_end` | `{bird_id}` | duration derived server-side from the pair |
| `offer` | `{kind: seed\|song\|pool, song_id?}` | server validates per-bird cooldown on tick consumption |
| `settle` / `settle_undo` | `{}` | undo valid ≤5s after settle (server enforces) |
| `re_engage` | `{}` | un-settles (§5.9) |
| `adopt` | `{bird_offer_id, name}` | accepts an age-gated bird offer |
| `rename` | `{bird_id, name}` | the one direct state write, restricted to `birds.name` |
| `tz_report` | `{iana_tz}` | updates account tz (D16) |

### 4.4 Notebook, settings, export, deletion

```
GET  /v1/notebook?before=<cursor>&limit=30      -- infinite scroll-back, read-only
GET/PATCH /v1/account/settings
POST /v1/account/export      → 202; job emails a signed, expiring download link (JSON: birds with
                               personality vectors, moods, notebook, settings — D10)
POST /v1/account/delete      → soft-delete now (sessions kept valid for restore), hard at +30d
POST /v1/account/restore     → clears deletion_requested_at; aviary unsuspended with catch-up (D11)
```

### 4.5 Visits

```
POST   /v1/invites {email}        → emails one-time link (system-voice copy)
GET    /v1/invites                → outstanding/active invites + visit log (email, date, approx duration)
DELETE /v1/invites/:id            → revoke, effective at visitor's next poll
GET    /v1/visit/consume?token=…  → single consumption: marks invite active, sets a visitor cookie
                                    bound to that invite (D9), serves the visit page
GET    /v1/visit/snapshot         → visitor-cookie-auth'd, read-only; 410 + matter-of-fact surface
                                    when revoked/expired
```

Visitor properties enforced server-side, not just in UI: the visitor identity has no event-ingest route, generates no presence, and is excluded from drift inputs by construction (there is simply no path from a visit to the events table). The visit page is the same renderer in render-only mode — same snapshot shape, interaction modules not loaded.

---

## 5. Simulation engine

The engine lives in `packages/sim-core` as pure functions: `tick(state, events, interval, config, rng) → (state', emissions)`. Purity is what makes the calibration harness (§14.3), time-compressed QA, and deterministic tests possible.

### 5.1 Tick scheduling (D1)

- **Active aviaries** (a connected client, active visitor, or any event/poll in the last 10 min): tick every 60s **[tune]**.
- **Dormant aviaries**: batch catch-up tick every 15 min, computing the same per-minute evolution in macro-steps. With no events, drift is exactly zero (I2), so dormant ticks are cheap mood/weather/notebook evolution.
- **Equivalence invariant:** tick math is a pure function of (state, event slice, elapsed interval, seeded RNG), so one 15-minute macro-step equals fifteen 1-minute steps for the event-free case; a property test asserts this. No observable surface can reveal the cadence tiering, because every snapshot read performs inline catch-up first (§4.2).
- **Long-gap fast-forward:** for gaps >48h, mood fast-forwards by sampling the time-of-day-conditioned stationary distribution of the mood chain rather than stepping every interval; weather regenerates on schedule; sparse "while away" notebook candidates may emit (the aviary continues without the viewer), capped to preserve sparsity.
- **Mechanics:** workers claim due aviaries via `SELECT … WHERE next_tick_at <= now() FOR UPDATE SKIP LOCKED LIMIT 100`. One transaction per aviary: consume `events.seq ∈ (high_watermark, latest]`, write state, advance watermark, insert `ticks` row, set `next_tick_at`. Idempotent by watermark; a crashed worker's claim simply re-runs. Suspended aviaries (soft-deleted) are skipped (D11).
- **Cost target:** median tick <2ms CPU; p99 latency alarm at 5s (per the PRD's error budget). At 100k aviaries (~10% active) this is ~180 ticks/s — one modest worker pool.

### 5.2 Personality drift

Traits `v ∈ [0,1]`. Per tick, for bird `b`, trait `t`:

```
Δv_t = headroom_clamp( (1 − v_t) · Σ_signals K[s,t] · x_s )
```

- `x_s` = signal quantity in the tick window: presence-seconds (aviary-level), listen-in seconds on `b`, offer outcomes involving `b`.
- `(1 − v_t)` makes drift saturating — fast-feeling early, asymptotic later, never overflowing.
- **Signal weights** (starting values, **[tune]** via §14.3):
  - Presence: `K = 2.0e-5` per presence-second on all five traits (dominant input). Reference: regular use (5 sessions/wk × 10 min) ⇒ ~0.03/week at mid-range — measurable in instruments at 1 week.
  - Listen-in: 4× presence rate, applied to the focused bird's `social_warmth` and `vocal_freq` only.
  - Offer accepted: `+0.0015 curiosity`; offer approached (bird came to front for it): `+0.0008 boldness`.
  - Settle: no drift input; it only closes the presence window cleanly.
- **Daily cap:** total Δ per trait per rolling 24h ≤ 0.01 (`drift_headroom_today`), enforcing "no single session moves a trait visibly" regardless of marathon sessions.
- **Monotonicity (I2):** all `K ≥ 0`, all `x_s ≥ 0`; the tick additionally clamps `v' ≥ v` and a property test fuzzes arbitrary event streams asserting non-decrease.
- **Calibration contract** (tested, §14.3): instrument-measurable drift (Δ ≥ 0.02 on ≥2 traits) at 1 week of regular scripted use; behavior-visible drift (defined as: greeting-first probability +15pp, front-perch dwell +20%, spontaneous call rate +25% — above perceptual just-noticeable-difference thresholds we validate with the panel in §15) at ~3 weeks; no behavior metric moves >5% from any single session.
- **Irreversibility note:** personality is never recomputed from event logs (engine rule), so a mis-calibrated `K` cannot be "replayed away." Consequence: launch at the conservative end of the calibrated band and raise — raising is always safe under I2; see risk R1.

### 5.3 Presence accounting (I3)

- **Client:** listeners on `visibilitychange`, `focus/blur`, and throttled activity (`pointermove`, `pointerdown`, `keydown`, `touchstart` — 1 update/s max). Evaluator every 5s: `visible && focused && (now − lastActivity) < W`, `W = 4 min` default **[tune, lean long — watching without moving is the product]**, value delivered via server config. While true, the client closes and ships a `presence_span` every 30s (`{start,end}`); on loss/pagehide it closes the open span via `sendBeacon`.
- **Server (tick):** accrues presence-seconds as the **union of span intervals across the account's devices** per tick window (D13) — two devices open simultaneously cannot double-count; a minute of wall clock yields at most 60 presence-seconds. Spans are additionally sanity-clamped server-side (no span >60s, no overlap inflation, total ≤ wall-clock window). Spoofing only affects the spoofer's own birds; caps are the only hardening (D18).
- **Settle and tab-close are identical to the engine:** both just end span accrual. No recovery surface, no penalty, no record of "left without settling" (I5).

### 5.4 Expressiveness modulator E (D8)

How neglect becomes "ambient" without ever touching traits (I2):

```
presence_ewma ← decay(presence_ewma, half_life = 5 days) + presence_seconds_this_tick
E = 0.35 + 0.65 · sat(presence_ewma / REF)     -- REF ≈ 20 min/day equivalent; sat = soft saturation
```

`E ∈ [0.35, 1]`, per-aviary, recomputed every tick. E multiplicatively scales: spontaneous call rate, greeting probability and promptness, chorus-join probability. The floor (0.35) guarantees the aviary is never dead — a two-week-absent user returns to birds that are quieter, present, and unresentful. E is fast-recovering (a few good sessions restore it), which is the mechanical shape of "something to ease back into."

### 5.5 Mood machine (D7)

States: `alert, curious, content, wary, drowsy, resting`. Per tick, per bird:

```
P(next | current) = StickyBase[current] ⊙ diurnal_prior(local_time) ⊙ weather_mod
                    ⊙ interaction_nudges(events) ⊙ personality_bias(b) ⊙ contagion(neighbors)
→ renormalize → sample with bird's seeded RNG stream
```

- **Sticky base:** strong self-transition; minimum dwell ~20 min **[tune]** before any switch (prevents flicker).
- **Diurnal prior:** account-local time (D16). Early morning tilts alert/curious; dusk tilts drowsy; night tilts resting for all species except the nightjar-like one, which inverts (active flag at night).
- **"Daily-ish reset":** implemented as **dawn re-anchoring** — at local dawn, the chain re-samples from a morning prior rather than hard-resetting, so mood is fresh each day yet never snaps mid-session and persists across sessions by simply being persisted (the bird that ended drowsy at dusk is resting by night and alert-ish by morning via ordinary chain evolution).
- **Interaction nudges:** accepted offer → toward content/curious; listen-in sustained → mild content; greeting consumed → mild alert→content.
- **Personality bias:** high boldness suppresses wary entry; high curiosity amplifies curious; etc. (small multipliers, **[tune]**).
- **Contagion:** a wary bird raises neighbors' wary multiplier for the next few ticks (alarm spread); co-perched content birds mildly reinforce content.
- **Client reactions vs. canonical state:** the client portrays immediate behavioral *reactions* (a bird approaching an offered seed now) without owning mood; the canonical mood nudge lands at the next tick. The one-minute seam is invisible because reactions and mood-idle differences are continuous, not labeled (§7.6).

### 5.6 Return-greeting planner

Computed server-side and delivered **on the snapshot response** (no extra round trip; fits the 1–2s window — §7.2 gets the snapshot to the client in well under a second).

- **Trigger:** snapshot request where the absence gap (since last presence span) ≥ 30 min for the full greeting; 5–30 min yields the light tier; <5 min none (stepping away for coffee ≠ arriving).
- **Greeter selection:** weighted draw ∝ `boldness^1.5 · social_warmth · mood_multiplier · E`, with a small anti-streak noise term so the same bird doesn't *always* greet first (the notebook's "pip greeted before wren today, first time this week" needs real variation to be worth writing). Wary/resting birds may produce no greeting at all on a given day — allowed and correct.
- **Form selection by absence tier:** `<30min` → glance up / head-tilt; `30min–8h` → two-note call + look; `8h–3d` → step toward front perch + longer call; `>3d` → re-orientation: approach, longer call, and possibly a second bird's staggered response (offsets drawn 0.6–2.4s — never unison, per the staggering rule).
- **Procedural variation:** the plan carries parameter ranges, not a fixed animation id; the client realizes pose timing, call phrase, and approach path from the bird's RNG stream with an **anti-repeat memory** (last 8 greeting realizations per bird kept client- and server-side; enforce minimum parameter distance). "Never identical twice" is thus a tested property, not an aspiration.
- **Multi-device:** the absence gap is account-level, so a phone opened a minute after the laptop gets at most a glance (correct — the user has been present).

### 5.7 Weather generation (D15)

Per-aviary (each user's place has its own sky), server-canonical so all devices and visitors see the same event: Poisson-scheduled — rain ~3×/week, 90–300s, intensity 0.2–0.6; soft wind ~daily, 60–180s **[tune]**. Stored as `(type, start, duration, intensity)` in aviary state and delivered in snapshots (including a short forward schedule so clients render onset smoothly mid-poll-interval). Mood effects: rain multiplies `vocal_freq`-driven call rates down and nudges toward drowsy/content; wind nudges some birds alert, some wary (personality-biased). Effects decay within ~2× event duration. Raindrops/leaf ripple are client-local ornaments off the canonical envelope.

### 5.8 Offers (engine side)

Tick consumes `offer` events: validates per-bird cooldown (3 min default **[tune]**) — the *responding* bird is chosen by the engine (nearest-by-perch curious/bold birds first; drowsy birds may not respond at all), applies mood nudges and drift micro-increments (§5.2), and records outcome (`accepted | observed | ignored`) for notebook candidates. Song-fragment offers carry the motif id so the responding bird's reaction (join / quiet / counter-call) keys off its `vocal_freq` and mood. Cooldown rejections produce no user-facing error — the offer affordance simply reflects availability (a gesture, not a vending machine).

### 5.9 Settle (engine side)

`settle` sets `settled_until = local dawn` (or until `re_engage`/`settle_undo`): call policies quiet (intensity and rate scaled ~0.3×), lighting override `settled`, moods tilt drowsy/resting at next ticks. `settle_undo` within 5s reverses cleanly (server validates the window). `re_engage` (an explicit interaction — click on a bird, listen-in, offer; not mere pointer movement) clears it with a gentle return ramp. Settled state is canonical: a visitor or a second device sees the settled aviary — correct, the visitor sees exactly what the host would see.

### 5.10 Age-gated bird offers (D12)

Schedule on `aviaries.created_at` (server config): offers at days **75, 160, 255, 360, 475** ⇒ cap 7 at roughly 16 months. Mechanically self-ramping for launch (§15.4). The offer is staged as a *visitation*: a candidate bird (species drawn from the pool, weighted toward call-space gaps in the current flock — §8.4) starts appearing occasionally on the back perch; the notebook may observe "a new bird has been about the back perch these last mornings"; the offer panel (behind the existing top-bar offer affordance — **no badge**, I5) gains a quiet entry where the user can adopt (name it) or do nothing. Unadopted after 14 days → it moves on; the next window follows the schedule. Adoption creates the bird with seed traits per species prior. No announcement, no modal, no urgency mechanics.

### 5.11 Seed personalities & adoption

First sign-in creates the aviary + two starters: species pair chosen for contrast (different silhouettes and non-overlapping call registers); seed traits drawn from species priors with deliberate differentiation (one starter seeded bolder, e.g. boldness 0.45–0.60 vs 0.25–0.40) so the first week already shows legible character contrast. The client adoption flow presents them as *the birds that arrived* — naming sheet with curated species-flavored suggestions, rename anytime later (`rename` event). Empty-aviary quiet field → first bird soft fly-in after naming (§7.7).

### 5.12 Notebook generation (D14)

Server-only, from canonical state + event aggregates — never from client-side call ephemera (the server doesn't know which specific calls played, D3, and never pretends to; entries about sound stay at the level the server does know: policy intensity, e.g. "low calls only").

- **Candidate detectors** (run in tick): first-greeter change vs trailing week; offer accepted after a wary stretch; mood streaks (a fluffed cold morning); weather moment + reaction co-occurrence; perch-pattern shifts; long quiet stretches; new-bird visitation; night activity of the nightjar.
- **Sparsity controller:** each candidate gets a noteworthiness score; emit only if score > adaptive threshold θ; θ jumps on each emission and decays toward baseline over ~48h — closed-loop target ~1 entry per 2–4 days regardless of activity level **[tune]**.
- **Prose:** template-grammar over the curated fragment library (no LLM at runtime — voice control, cost, latency, and offline determinism all point the same way), seeded variation, anti-repeat memory per aviary. Voice rules enforced by the same lint as §10. **Structural rule, lint-enforced: templates can reference birds, weather, light, and the aviary — there is no grammatical slot for the user.** "pip greeted before wren today" is expressible; "you visited every day this week" is unrepresentable (I5).
- Entries are immutable rows; read-only API; infinite scroll-back; no archival.

### 5.13 Narration & caption text (shared engine)

The naturalist text engine (fragment library + composer) is one isomorphic module used three ways: notebook entries (server), screen-reader narration (client, §9.1), call captions (client, §9.3). One library means one voice — a screen-reader user moving between narration and notebook hears the same product.

---

## 6. Sync model & state ownership

### 6.1 The ownership table

| State | Owner / writer | Readers | Conflict story |
|---|---|---|---|
| Personality vectors | tick worker only (I1) | tick, snapshot derivation (as derived render params), export | none possible — single writer, additive deltas in `seq` order |
| Mood, perch, E, weather, settled | tick worker (server) | snapshots | same |
| Interaction events | clients (append-only) | tick | idempotency key dedupe; server `seq` ordering (D2) |
| Bird name, settings, tz | api (direct, validated) | all | last-write-wins is acceptable here and only here (names/settings are user intent, not accumulated state) |
| Calls, poses, ornaments | client (ephemeral, D3) | nobody | not state — regenerated continuously from policy params |

### 6.2 Why conflicts are unreachable

The classic failure (laptop and phone each write a personality snapshot; one overwrites the other's drift) requires clients that write state. Here clients cannot express a state write: the event vocabulary (§4.3) has no absolute-value verbs. Both devices append events; the tick consumes them in `seq` order; deltas compose additively and commute in effect (all increments, daily-capped). Multi-device simultaneous use degenerates to "more events in the log," with presence double-count prevented by interval union (§5.3).

### 6.3 Schema-level enforcement (I1)

Postgres column grants: `api_role` has INSERT on `events`, SELECT on state tables, UPDATE on exactly `birds.name`, `accounts.settings`, `accounts.timezone` (+ auth tables); **no** UPDATE grant on personality/mood/E columns. `tick_role` alone updates simulation state. A migration-time CI check diffs grants against a checked-in policy file, so the invariant survives schema evolution. A misrouted write fails loudly at the DB, not silently in review.

### 6.4 Client snapshot lifecycle

Pull triggers per §4.2 → reconcile (§7.6) → interpolate. Between polls the client runs local continuation (idle motion, calls from policy, lighting from clock) — the aviary never waits on the network to be alive. Polling-only at v1 (D6): with a ≤45s visible-poll cadence and inline catch-up on every read, real-time push buys nothing the product needs (revocation latency tolerance is explicitly "next pull" per the spec), and dropping websockets removes a whole class of infra and reconnect-state bugs from the critical path.

---

## 7. Client architecture & rendering pipeline

### 7.1 Module map & bundle plan (budgets, gz)

| Chunk | Contents | Budget |
|---|---|---|
| **inline boot** (in HTML) | snapshot JSON + micro-paint kernel + critical CSS | ≤ 30KB |
| **core** | renderer, pose system, behavior controller, presence tracker, snapshot/event client, lighting, narration+caption text engine, top-bar shell | ≤ 200KB |
| **audio** | synth engine, scheduler, motif/grammar data | ≤ 150KB |
| lazy: notebook / settings+account / adoption / offer panel / visit mode | per-surface | ≤ 60KB each |
| **Total shipped v1** | | ≤ 900KB (cap 2MB; headroom is deliberate, R5) |

CI gate (`size-limit`) fails any PR exceeding a chunk budget (§12.3). Bird art is parametric vector geometry + small palettes (no bitmaps unless a species demands one ≤30KB).

### 7.2 Boot: the road to first-bird <500ms (D5)

1. **HTML from origin** (edge-terminated TLS), ~25–30KB gz: critical CSS, the current snapshot inlined as JSON (the server renders it into the page — one round trip total), and the **micro-paint kernel**: a ≤12KB inline script that draws the full scene composition — sky/lighting for local time, foliage, every bird at its canonical perch in a pose from the shared pose model, breathing-level motion. First frame is the aviary mid-action; there is no spinner anywhere in the product.
2. **Core chunk** (preloaded, async) hydrates within ~0.5–1.5s and adopts the same canvas + pose state — the takeover is invisible because both stages read one pose kernel. M1 includes an explicit validation gate for takeover invisibility; fallback option if a seam is ever visible: inline the full core (it fits — accepted parse cost).
3. **Audio chunk** loads after first paint; audio start is gesture-gated anyway (D4, §8.6).
4. **Service worker** (registered post-load): caches shell + last snapshot per device. Warm starts render last-known state instantly (~0ms network), then refresh-and-reconcile within one poll — for a returning user the aviary is *already there*, which is the truest implementation of the conceit.
5. **Slow-network grace:** if the inline snapshot is somehow absent/stale-beyond-use (cold SW + origin slow), the kernel draws the **quiet field** (soft sky gradient + one or two faint drifting motion cues) — the designed loading state, not a spinner. Same surface serves the brief empty-aviary moment pre-adoption (§5.11).

Budget arithmetic (mid-tier mobile, 4G): DNS+TLS+TTFB ≈ 250–300ms + 30KB transfer ≈ 60ms + parse/paint ≈ 80ms ⇒ first bird ≈ 400–450ms p75. The synthetic fleet (§12.4) enforces this from M1 onward.

### 7.3 Renderer

- Single visible canvas, layered offscreen canvases composited per frame: `sky/lighting → background foliage (slow parallax) → back/middle/front bird planes → foreground occasional branch → weather ornaments`. Static layers cached offscreen and redrawn only on lighting-step or viewport change; per-frame cost is birds + ornaments + composite.
- **Birds:** parametric skeletons (body/head/tail/wing groups as Path2D from build-time-compiled species geometry). Pose graph per species: perch-idle set, preen, scan, head-tilt, weight-shuffle, call posture, hop, short flight arc, drink/bathe/watch (pool reactions), fluffed (cold/wary), low-perch rest. Live pose = base pose + low-amplitude procedural noise layers (breath, micro-shift) keyed by bird seed — continuous, never looped frame sequences.
- **Behavior controller** (per bird, client-side): consumes mood + traits-derived render params + E + local event bus (a call from bird A triggers head-tilts in others; offers trigger approach reactions) and schedules pose transitions + perch moves. Perch *zone* is canonical (tick-chosen); position within a zone and all motion between are client-realized.
- **Lighting:** palette LUT interpolated along a solar curve from account tz (D16) ⊗ weather modifier ⊗ settled override. Continuous drift, stepped redraw of cached layers at imperceptible increments (~30s).
- **Ornaments:** leaves/feathers/raindrops from object pools (zero steady-state allocation, §12.2), client-local randomness, disabled in reduced-motion.
- **Responsiveness:** scene scales to viewport preserving composition; perch spacing compresses on narrow viewports; all birds always in frame (layout solver clamps); DPR-aware backing store with **adaptive resolution** — if sustained frame time >16.6ms, step backing resolution down before dropping frames (60fps on the 5-year-old laptop is a budget, §12.1).

### 7.4 Frame & visibility loop

`requestAnimationFrame` master loop. On `hidden`: stop rAF, suspend AudioContext, presence evaluator handles spans (I3); client-side nothing simulates while hidden — the server is the simulation. On `visible`: immediate snapshot pull → reconcile (§7.6) → resume.

### 7.5 Reduced-motion mode (§9.2 for the surface; here the mechanism)

A presenter-level swap, not a feature flag sprinkle: the behavior controller emits the same target poses; the **cross-fade presenter** renders slow dissolves between held poses (preen = sequence of preen poses cross-fading) instead of continuous animation; flights become cross-fades between perches; ornament system off; lighting transitions slowed ~2×. Activation: `prefers-reduced-motion` at boot OR settings override (tri-state system/on/off). Both presenters share pose assets, so reduced-motion can't drift out of feature parity.

### 7.6 Reconcile rules (no snapping)

- Hidden <30s: continue local state, fold in snapshot deltas via natural transitions (a bird whose canonical zone changed *hops/flies* there; lighting eases ≤2s).
- Hidden longer: render canonical state directly on the first visible frame — things moved while you were away, which is the product telling the truth; mood never visibly "resets" because mood is persisted (§5.5).
- Local reactions never conflict with canonical state because reactions are transient poses, and the next tick's nudges move canonical mood *toward* what the reaction portrayed (§5.5).

### 7.7 Top bar & secondary surfaces

Thin DOM bar above the canvas: account/settings, accessibility settings, notebook, offer affordance — nothing else, ever (I5). Fades to ~5% opacity after 4s of cursor stillness **[tune]**; restores on pointer/keyboard activity; never fades while it or any child holds keyboard focus (a11y). Notebook, settings, adoption, offer panel are lazy-loaded Preact surfaces in the product's two voices (§10). The aviary canvas itself contains zero chrome, labels, tooltips, or overlays (captions and focus indicators are designed a11y surfaces, §9).

---

## 8. Audio pipeline

### 8.1 Synthesis engine

- **Voice pool:** 8 pre-built synth voices (the 7-bird cap + 1 for overlap tails), each a fixed WebAudio subgraph: 2 oscillators (sine/triangle, detuned) + optional FM partial → per-syllable ADSR gain → formant-ish bandpass (per-bird center/Q) → per-bird gain → subtle equal-power pan by perch x → master bus (gentle compressor → limiter → destination). One shared lightweight feedback-delay network supplies "outdoors" space. Breath/noise component via AudioWorklet where supported, filtered-noise-buffer fallback otherwise (feature-detected; both paths conformance-tested, R14).
- **No allocation in steady state** (§12.2): voices, buffers, and parameter-event arrays are pre-allocated and reused; calls are scheduled onto existing nodes via `AudioParam` automation (`setTargetAtTime`/ramps), never by rebuilding graphs.

### 8.2 Motifs & grammar runtime (D3)

- **Motif** = parametric syllable: f0 contour (pitch envelope), amplitude envelope, timbre mix, duration 80–400ms. Each species ships 3–6 motifs + a weighted transition table = its call grammar.
- **Phrase generation:** per bird, a lookahead scheduler (100ms lookahead, 25ms timer against the AudioContext clock) draws next-call times from an exponential distribution at `rate_per_min × E × diurnal × mood` (all folded into the snapshot's `call_policy`), then walks the grammar 2–5 motifs with mood-tilted weights.
- **Per-bird signature** (recognizability is the load-bearing property): deterministic transform from `rng_seed` — transpose ±4 semitones, tempo 0.85–1.2×, brightness tilt, vibrato depth, motif preference skew. The signature transform is **fixed for the bird's lifetime**; mood and drift modulate rate, intensity, and phrase length, never the signature core — Pip stays knowable by ear across moods and months.
- **Anti-repetition:** per-bird memory of the last 12 realized phrases; new phrases enforce a minimum parameter distance; jitter floors on every timing/pitch draw. "Hearing the same call twice, exactly" is structurally impossible, and a regression test renders 500 consecutive phrases per bird asserting the distance floor.

### 8.3 Bird-to-bird & chorus

When a bird calls, others roll `respond_p` (warmth × distance × mood) to schedule a response in 0.5–2s; alarm-class motifs (wary) additionally raise neighbors' wary multipliers visually client-side ahead of canonical contagion (§5.5). Chorus = emergent overlap of independently-scheduled procedural voices through the shared bus — real polyphony, no loop stacking by construction. `chorus_join_p` lets high-`vocal_freq` birds pile on when ≥2 are already calling. The master compressor/limiter keeps N=7 dense passages from mudding (levels validated in the recognizability test, §14.5).

### 8.4 Call-space allocation across the flock

New-bird species selection (§5.10) weights toward unoccupied call space (pitch register × tempo × timbre distance from current flock signatures), directly defending the "knowable by ear at 7" cap. The recognizability metric (§14.5) is computed pre-adoption for the candidate against the existing flock; candidates that would blur are re-drawn.

### 8.5 Listen-in mix

Per-bird gain ramps via `setTargetAtTime` (exponential approach — organic, not linear-faded): focused bird to bed +6dB over ~2s; others down to an ambient floor of −18dB relative — **never to zero**. Disengage (click again / focus elsewhere / click empty space / keyboard blur, Escape) ramps back symmetrically. Listen-in start/end events feed drift (§5.2).

### 8.6 Autoplay policy & fallback (D4)

Browsers gate audio on a user gesture; the brief's "calls already audible" meets reality here:

- On load, attempt `AudioContext.resume()`. If blocked: the scene runs fully alive **visually**; the scheduler keeps generating (and captions, if enabled, keep captioning — they describe generated calls, audible or not); on the first qualifying gesture (pointerdown/keydown/touchstart — pointermove does not qualify per browser rules) audio fades in over ~2s mid-stream, as if the window just opened. **No prompt, no "click to enable sound" banner** (I5) — the cost of a silent first few seconds for idle-mouse users is accepted and documented (R4); the greeting is also visual, so the anchor moment survives.
- **WebAudio unavailable** (old browser, denied context, hardware): graceful silence with **captions enabled by default** for that session (the one setting we flip for the user, since otherwise the product is silently absent), plus a one-line matter-of-fact note inside accessibility settings — not a toast. No recorded-audio path exists in the codebase (non-goal; bundle and uncanniness both).

---

## 9. Accessibility surfaces

Designed surfaces, shipped in v1 with the features they mirror — the M2 exit gate (§15) blocks launch on them, structurally preventing the "a11y lands in v1.1" failure.

### 9.1 Screen-reader narration

- A visually-hidden `aria-live="polite"` region fed by the **narration engine** (client-side instance of the shared naturalist text engine, §5.13, in the core bundle — SR users get it on first load, not after a lazy chunk).
- **Cadence scheduler:** idle narration every 30–60s (randomized), composed from current snapshot + local reaction state ("a small grey bird is perched on the front rail, calling softly. it is morning in the aviary; the light is gentle."). User-initiated events (return-greeting, offer reaction, settle acknowledgment) enter a priority queue narrated promptly, with coalescing so the queue never backs up; everything is written as observation, never state-transition announcement ("warbler perched at high branch" is lint-rejected by register rules, §10).
- Narration never mentions trait values or system internals; it reads the same aviary the eyes would.

### 9.2 Reduced-motion

Mechanism in §7.5. Product framing honored in QA: it is a *register of rendering* with its own calm aesthetic — calls, captions, drift, mood, notebook all fully alive. Visual-regression suite runs every scene scenario in both presenters; feature parity is asserted by running the same e2e flows in both modes.

### 9.3 Call captions

Opt-in (accessibility settings; auto-on for a session when WebAudio is unavailable, §8.6). Generated **from the realized phrase parameters** at schedule time — motif classes + tempo + intensity map through the caption fragment library to prose ("a low trill, paused, low trill again") — so the caption always matches what actually played. Rendered as small DOM text near the calling bird's screen position, fading with the call envelope, AA-contrast against scene tokens. Marked `aria-hidden="true"` to avoid double-speaking against narration (D17).

### 9.4 Keyboard navigation & focus

- DOM focus-proxy overlay for birds (visually aligned to canvas positions): Tab traverses top bar → first bird; arrow keys move between birds (roving tabindex); Enter = listen-in on focused bird; Escape = exit listen-in; offer panel opens via top-bar shortcut and is fully keyboard-operable; settle reachable in the top bar. Proxies carry naturalist-but-functional accessible names ("pip — a small grey bird on the front perch"), updated on state change but announced only on focus (no live-region spam).
- Focus indicator: soft high-contrast outline token pair that swaps with scene luminance (day/dusk/night/settled) to hold ≥3:1 non-text contrast in all lighting states; designed treatment from the visual designer, validated in the visual-regression suite across lighting states.
- The top bar never fades while focus is within it (§7.7).

### 9.5 Contrast & semantics

All user copy (top bar, settings, account, errors, captions, visible narration) meets WCAG AA minimum, tokens specified in the design system; automated axe checks in CI + manual matrix (§14.6). The canvas scene carries no user copy, so the constraint binds on chrome and overlays only.

---

## 10. Voice & copy system

Voice is load-bearing, so it gets infrastructure, not vibes:

- **Central strings/templates registry** (`packages/voice`): every user-visible string and template fragment is tagged `register: naturalist | system`. No literals in components — CI greps for untagged user-facing strings.
- **Register lint (CI-blocking):** naturalist — lowercase, present tense, no exclamation marks, no second person ("you/your"), no announcement framings, bird-verbs preferred; system — sentence case, plain, direct, no naturalist phrasing. Denylist across *both* registers: "welcome back", "achievement", "streak", "level", "badge", "score", "congrats", "don't forget", "you've been here" (I5).
- **Register boundary rule, encoded in the registry:** money/identity/errors/settings/sync ⇒ system; everything else ⇒ naturalist. New surfaces must declare a register to compile.
- **Voice owner:** one named human (PM or writer) approves additions to the fragment libraries; the style samples from the brief live at the top of the registry as the canonical reference.
- The notebook/narration/caption fragment libraries (§5.13) live in this package, so the lint covers generated prose, not just static UI strings — including the structural "no grammatical slot for the user" rule (§5.12).

---

## 11. Privacy & telemetry boundary (I4)

Enforced as architecture, in four layers:

1. **Physical separation:** per-account interaction state (events, vectors, moods, notebook) exists only in the simulation DB. There is no replication, FDW, ETL, or export from the simulation DB to any analytics store. The warehouse ingests *only* the metrics stream.
2. **Metric registry:** every telemetry metric is declared in a checked-in registry with its dimensions; CI rejects any metric carrying `account_id`, `bird_id`, email, or any interaction-history field. RUM beacons use a random per-page-load id (rotating, unlinkable), coarse device class, and coarse geography only.
3. **Allowed aggregate set** (explicitly enumerated): request counts/latencies, tick-latency distributions, error rates, anonymized session-duration histograms (no account dimension), render-frame timing, audio-context error counts, bundle/TTFBird timings, email delivery success, account totals. **What we deliberately don't measure:** per-account engagement, funnels, retention cohorts keyed to interaction patterns, per-bird anything, drift distributions across accounts, "most-X" stats of any kind (also forecloses leaderboards forever, §1.2).
4. **Process:** quarterly boundary audit (registry vs. reality diff); privacy policy in account settings names the aggregate categories and the per-bird exclusion in plain text (system voice).

Calibration data needs are handled without crossing this line — synthetic cohorts and a consented beta, §14.3/§15.3.

---

## 12. Performance budgets & observability

### 12.1 Budgets (CI/fleet-enforced; the felt-aliveness numbers)

| Budget | Value | Enforcement |
|---|---|---|
| Initial JS (sum to interactive aviary) | <2MB gz cap; **internal target ≤900KB** | `size-limit` per chunk, PR-blocking |
| Time to first bird | <500ms, mid-tier mobile / 4G, p75 | synthetic fleet gate from M1; RUM p75 alarm |
| Idle frame rate | 60fps sustained 30 min, 5-year-old mid-range laptop | weekly device-lab run + adaptive-resolution telemetry |
| Client memory | zero net growth over 30 min | nightly CI (Puppeteer, heap sampling, ±2% band); per-PR 5-min heap-diff smoke |
| Simulation tick latency | p99 < 5s alarm; median <2ms CPU | server metrics + alert |
| Snapshot payload | ≤8KB gz | contract test |

### 12.2 Memory discipline (how zero-growth is real)

Pooled: synth voices, parameter arrays, ornament particles, pose scratch vectors, caption DOM nodes (recycled), beacon buffers. Notebook view virtualizes scroll-back (entries beyond viewport release references). No per-call/per-frame allocation in hot paths — enforced by the nightly heap test, plus an allocation-profiling pass in M4.

### 12.3 CI performance gates

Bundle budgets (PR), Lighthouse-style synthetic boot trace on throttled emulation (PR, smoke), full synthetic-fleet assertions (merge-to-main), nightly 30-min memory + frame-time soak, weekly real-device-lab run.

### 12.4 Synthetic fleet & RUM

Scripted browser fleet (cold load → 5-min idle with presence → listen-in → offer → settle → reload-warm) from 4–6 geographies on schedule; asserts every budget and the autoplay/fallback paths. Aggregate-only RUM per §11. Alarms: TTFBird p75, error rates, audio-context failure rate, tick p99, event-ingest backlog age (R2), email delivery failures.

---

## 13. Security

- **Magic links:** 15-min TTL, single-use, hashed at rest, per-email and per-IP rate limits, uniform 202 responses (no enumeration). Sign-in emails are system-voice, no marketing.
- **Sessions:** httpOnly Secure SameSite=Lax cookies, server-side revocation list (the sessions table is the source of truth), device labels, last-seen.
- **CSRF** tokens on all mutations; strict CORS (first-party only); CSP with no inline-script allowance beyond the hashed boot script; Trusted Types where supported.
- **Visit tokens:** single-consumption, hashed, browser-bound via visitor cookie (D9); visitor scope is one read-only route; revocation/expiry checked on every poll.
- **Input validation** via shared zod schemas at the edge of every endpoint; event payload size caps; batch caps.
- **PII:** email encrypted at rest (KMS), HMAC lookup hash, no email in logs/metrics/queues (I4 doubles as the PII rule); deletion pipeline verified by a post-hard-delete audit job that asserts zero rows remain for the account UUID across all tables, recording only the UUID + completion time.
- **Dependency & secrets hygiene:** lockfile audits in CI, secret scanning, least-privilege DB roles (§6.3 is also a security control).
- Pre-launch external pass: auth flows, invite flows, deletion (threat model: account takeover via link interception, invite token leakage, event-flood abuse).

---

## 14. Testing & calibration

### 14.1 Unit & property tests (sim-core)

Drift math (incl. headroom cap edge cases), mood chain (dwell minimums, dawn re-anchoring, contagion), grammar walks, E decay/recovery. **Property tests:** I2 monotonicity under fuzzed event streams; tick equivalence (15×1min ≡ 1×15min, event-free) (D1); idempotent event application under replayed batches; presence interval-union never exceeds wall clock.

### 14.2 Contract & integration

Shared-schema contract tests (client and server compile against the same zod types); API integration suite incl. auth edge cases (expired/reused links), invite lifecycle (consume/revoke/expire/410), deletion lifecycle (soft → restore → catch-up; soft → hard → audit-zero), export content (D10).

### 14.3 Drift-calibration harness (the load-bearing test rig)

Runs `sim-core` in compressed time against scripted user cohorts (e.g., "regular" 5×10min/wk, "intense" daily 40min, "weekend-only", "two-week lapse then return", "listen-in heavy"). Asserts the calibration contract (§5.2): instrument-measurable at 1 week, behavior-visible thresholds at ~3 weeks, no single-session visibility, E-floor behavior on lapse cohorts. Output: a calibration report per config-set; the shipped constants are the report's conservative band (R1). **Post-launch validation without violating I4:** re-run the harness with presence patterns *drawn from the anonymized session-duration histograms* (allowed telemetry) rather than any per-account data; plus the consented beta cohort (§15.3) with disclosed instrumentation, whose calibration data is deleted after the calibration milestone.

### 14.4 Simulation soak & chaos

Time-compressed year-long aviary runs (drift saturation behavior, offer schedule, notebook sparsity over a year — asserts ~1 entry/2–4 days closed-loop); tick-worker kill/restart mid-batch (watermark idempotency); event-backlog flood; clock-skew and DST-transition tests on diurnal logic (tz lib correctness at dawn re-anchoring).

### 14.5 Audio quality gates

- **Signature recognizability metric:** offline-render phrase corpora per bird; compute pairwise signature distance (pitch register / tempo / spectral centroid features); assert worst-case 7-bird flock stays above the distance floor; gate new species pool additions and §8.4 candidate draws on it.
- **Human listening gates:** blind panel identifies 4 birds by ear after 15 min exposure at ≥80% (M2 exit); chorus-density listening pass at 5 and 7 birds before those cap stages unlock in prod config (§15.4).
- **Anti-repeat regression:** 500-phrase render per bird asserts minimum inter-phrase distance (§8.2).
- **Cross-browser synth conformance:** offline-render the same seeds on Chrome/Safari/Firefox/Edge; assert spectral similarity within tolerance (catches Safari WebAudio quirks, R14).

### 14.6 Accessibility tests

axe-core in CI on all DOM surfaces; keyboard-path e2e (full session driven by keyboard only); narration cadence tests (no queue backlog under event storms; coalescing works); register-lint on all generated prose (§10); manual SR matrix per release: VoiceOver/Safari (macOS+iOS), NVDA/Firefox, JAWS/Chrome; reduced-motion visual-regression + feature-parity e2e (§9.2). External audit in M4.

### 14.7 E2E & visual

Playwright: presence conjunction truth table (visibility × focus × activity simulated independently — I3's test), greeting tiers by absence gap, listen-in ramps (audio-param assertions), offer cooldown, settle/undo/re-engage, multi-device coherence (two contexts, one account: settle on A dims B at next poll; no presence double-count), visitor mode (read-only, revocation 410), autoplay-blocked path (D4). Visual regression on pose sets, lighting states, weather, both presenters, focus indicators.

---

## 15. Rollout

### 15.1 Team shape (assumed)

6–8 engineers: 2 simulation/backend, 2 client rendering+audio, 1 platform/infra, 1 frontend+a11y; plus visual designer, sound designer, PM/voice-owner. Adjust milestone durations linearly if staffing differs.

### 15.2 Milestones (durations approximate; gates are hard)

- **M0 — Foundations (wk 1–2):** monorepo, CI/CD, all budget gates wired *and failing-capable from day one* (size-limit, axe, perf smoke), schema v1 + role grants (§6.3), design tokens, voice registry + lint (§10), environments, metric registry skeleton (§11).
- **M1 — Alive vertical slice (wk 3–7):** tick worker + drift + mood + E; snapshot/event API; canvas renderer with 2 birds, idle motion, lighting; presence pipeline end-to-end; basic procedural calls (1 species); boot path (D5) hitting TTFBird on synthetic. **Exit gate ("alive test"):** internal panel watches 10 minutes and finds no detectable repetition or canned cue; synthetic fleet green on boot budgets; I1–I3 property/e2e tests green.
- **M2 — Full product surface (wk 8–13):** greeting planner, listen-in, offers, settle; 6 species + signatures; chorus + bird-to-bird; weather; notebook v1 + sparsity controller; adoption flow; narration, captions, reduced-motion, keyboard nav; top bar + secondary surfaces. **Exit gates:** recognizability listening test (≥80% on 4 birds); internal a11y audit pass; voice owner sign-off on every string/template.
- **M3 — Accounts, sync, social (wk 12–16, overlaps M2):** magic-link auth, sessions, email change; multi-device hardening (union-presence, coherence e2e); export + deletion lifecycle; invites/visits/visit log; privacy-boundary enforcement complete (registry CI, ETL allowlist); security review.
- **M4 — Calibration & hardening (wk 16–20):** calibration harness sign-off (§14.3); audio polish with sound designer; perf hardening (device-lab 60fps, 30-min memory green); external accessibility audit + fixes; chaos drills (tick backlog, email-provider outage, DB failover); deletion runbook rehearsal; on-call setup.
- **Private beta (wk 20+, ≥4 weeks):** 100–300 consented users (calibration instrumentation disclosed, data deleted post-calibration). Four weeks minimum because the product's core claim needs ≥3 weeks to be perceivable. Mid-beta perception check-ins ("do the birds feel different than week 1?") validated against harness predictions.
- **Launch v1:** gates — all budgets p75-green for 2 consecutive weeks; external a11y audit closed; calibration report signed; security checklist done; privacy audit clean; runbooks tested.

### 15.3 Beta consent boundary

Beta accounts opt into named, temporary calibration instrumentation (per-account drift/presence inspection in the simulation DB only — never in telemetry) via explicit consent copy (system voice). At GA, the instrumentation code path is removed, not flagged off.

### 15.4 Birds-per-aviary ramp (self-ramping by design)

Age-gating (§5.10) staggers the population automatically: the earliest accounts reach bird 3 at day ~75 and bird 5 at ~12 months — the team gets months of real chorus data between cap stages. **Stage gates:** before the population first crosses 3→4 (day ~160 earliest) and 4→5+, the chorus listening pass and the recognizability metric must re-clear at that density; the offer schedule is server config, so a failing gate delays the next stage without a deploy (and without ever revoking a bird — gates only ever delay additions).

### 15.5 Instrumented from day one

Everything in §12.4, plus: audio-context failure rate by browser, autoplay-resume latency distribution, magic-link delivery latency/success, snapshot inline-vs-SW hit ratio, reduced-motion adoption rate (aggregate count only), caption adoption rate (aggregate only). All within the §11 registry.

---

## 16. Risks & mitigations

| # | Risk | Why it matters here | Mitigation / early warning |
|---|---|---|---|
| R1 | **Drift mis-calibration** — too fast (Tamagotchi-feel) or too slow (screensaver); effectively irreversible since vectors are never recomputed from logs | The product's central promise lives in a narrow band | Harness-derived constants (§14.3); ship at conservative band edge — under I2, raising rates later is always safe, lowering only slows future drift; daily caps; beta perception validation vs. harness predictions; config-only adjustment |
| R2 | **Event loss / ordering bugs silently eat drift** | Failure is invisible by nature (spec's own warning) | Append-only + idempotency keys; watermark accounting (events_through audit: every seq ≤ watermark provably consumed); backlog-age alarm; property tests; single-writer roles (§6.3) make the worst class unrepresentable |
| R3 | **Audio uncanny or repetitive; chorus mud at 5–7 birds** | Audio is the affective spine; recognizability caps the product | Signature distance metric + listening gates per density stage (§14.5, §15.4); anti-repeat memories with tested floors; call-space-aware species draws (§8.4); limiter/level design; sound designer embedded from M2 |
| R4 | **Autoplay gating silences the first session moments** | "Calls already audible" can't be literally true pre-gesture | D4 fade-in on first gesture, visually-complete greeting, captions path; measure resume-latency distribution; accepted residual, named here so nobody "fixes" it with a click-to-start modal (I5) |
| R5 | **TTFBird >500ms on real networks** | The conceit's felt threshold | Inline snapshot + micro-paint (D5); SW warm path; fleet gate from M1; quiet-field grace state; headroom in bundle target (≤900KB vs 2MB) |
| R6 | **Narration spam or a11y regression** | The accessible product must stay the actual product | Cadence engine with coalescing tests; SR matrix per release; external audit gate; axe CI; reduced-motion parity e2e |
| R7 | **Client memory growth** (audio params, captions DOM, notebook scroll) | 30-min zero-growth is a hard budget | Pooling everywhere (§12.2); nightly soak; per-PR heap smoke; virtualized notebook |
| R8 | **Privacy boundary erosion over time** | One convenient join undoes the architecture | Physical separation + registry CI + quarterly audit (§11); leaderboard-feeding aggregates simply don't exist |
| R9 | **Magic-link deliverability** | Auth is single-path by design | Provider monitoring + latency alarms; resend affordance; enumeration-safe UX; provider abstraction allows failover; matter-of-fact error surfaces |
| R10 | **Tick backlog at scale** | Stale aviaries break "continuing without you" | Tiered cadence + inline catch-up on read (D1) means user-visible staleness can't occur; autoscaled workers; p99 alarm; load tests at 10× projection |
| R11 | **Scope creep into non-goals** ("just one small toast/streak") | The PRD names this as the likeliest failure | I5 enforcement: copy-lint denylist, PR-template non-goals checklist, voice-owner gate; §11 makes engagement-loop data unavailable to argue from |
| R12 | **Weather/ambient feels assertive or random** | Calibration of "its own moments" | Config intensities; dogfood tuning diary in M2–M4; rarity bounds in config |
| R13 | **Multi-tz / DST edge bugs** in diurnal logic | Dawn re-anchoring and night gating depend on local time | IANA tz lib server-side; DST-transition test suite; most-recent-device-tz rule (D16) |
| R14 | **Cross-browser WebAudio divergence** (Safari quirks, Worklet support) | One bad browser path reads as a dead product | Feature-detect + conformance render tests per browser (§14.5); graceful-silence fallback is itself a designed state |
| R15 | **Soft-deleted/restored accounts feel wrong** (frozen vs. drifted) | Continuity is identity (engine rule) | D11: suspend ticks during soft-window, catch-up fast-forward on restore — birds are quieter, not frozen and not reset |

---

## 17. Decisions on PRD-ambiguous points (defensible calls, made so the team doesn't re-litigate)

- **D1** Tiered tick cadence: 60s active / 15min dormant batch catch-up, with a proven step-equivalence invariant and inline catch-up on every snapshot read. Honors "runs whether or not anyone is watching" at ~1/15 the dormant cost, with zero observable difference.
- **D2** Event ordering = server receipt (`seq`); `client_ts` stored as advisory. Client clocks are untrusted; drift deltas are order-insensitive in effect, so fairness beats clock archaeology.
- **D3** Calls are client-generated ephemera from server-supplied call policies; canonical state is policy, not call instances. Two devices may hear different (equally in-character) calls — calls are weather, not ledger.
- **D4** Autoplay: silent visual start, audio fades in on first qualifying gesture, no prompt. Captions don't force-enable for this case (only for WebAudio-unavailable).
- **D5** Two-stage first paint: inline snapshot + ≤12KB micro-paint kernel sharing the core's pose model; core adopts the canvas seamlessly. Fallback if seam visible: inline full core.
- **D6** Polling-only sync at v1 (no websockets): visible keepalive ≤45s + read-time catch-up meets every latency the spec sets (incl. visit revocation "at next pull").
- **D7** Mood set: `alert, curious, content, wary, drowsy, resting`; "daily-ish reset" = dawn re-anchoring of the chain (fresh mornings, no visible snap).
- **D8** Neglect-as-ambient is the aviary-level expressiveness modulator `E ∈ [0.35,1]` (decays, recovers); personality remains strictly monotonic (I2).
- **D9** Visit links: single consumption binds a visitor cookie in that browser for the invite's remaining lifetime (≤30d or until revoked); the raw link is dead after first use (no forwarding), revisits from the same browser work while valid.
- **D10** Export includes raw personality vectors per `accounts_sync.md`'s explicit field list; the never-expose rule is held as **never rendered in any UI** — the export is machine-readable data portability, not a surface. Tension between the two files noted and resolved in favor of the explicit, specific clause.
- **D11** Soft-deleted accounts: simulation suspended (not ticking for a user who asked to leave), full catch-up fast-forward on restore.
- **D12** New-bird offer surface: candidate appears occasionally on the back perch; notebook may observe it; quiet entry in the existing offer panel; moves on after 14 days unadopted. No badge, no modal, no countdown.
- **D13** Presence window `W = 4 min` default (server-config), heartbeat spans 30s, multi-device interval **union** (never sum).
- **D14** Notebook is server-derived only, from canonical state; it never claims specifics only a client could know, and templates have no grammatical slot for the user.
- **D15** Weather is per-aviary and server-canonical (devices and visitors agree); droplets/leaves are client-local ornaments.
- **D16** Account-level IANA timezone (latest device report wins) drives diurnal priors, lighting, and night gating — visitors see the host's time of day, matching "exactly what the host would see."
- **D17** Captions are `aria-hidden` (narration owns the spoken channel; captions own the visual one).
- **D18** No anti-cheat beyond sanity caps on presence: spoofing only drifts your own birds faster, and I2 bounds the blast radius.
- **D19** Stack: TypeScript monorepo; Fastify API + tick workers; Postgres-only infra at v1; custom canvas renderer + Preact chrome; no game engine.
- **D20** Sim console / time-compression / any vector-inspection tooling: staging-only, compile-excluded from prod (the never-expose rule applies to us, too).

---

## 18. PRD traceability

| PRD file | Primary plan sections |
|---|---|
| `product_brief.md` (principles, voice, scope) | §0 invariants, §1, §7.2/7.7, §10, §15 |
| `concepts.md` (vocabulary, presence, timescales) | §5.2–5.5 (clocks), §5.3 + I3 (presence), terminology used throughout |
| `bird_engine.md` (vector, drift, mood, calls, identity, cap, adoption) | §3.2, §5.2/5.4/5.5/5.10/5.11, §8.2–8.4, I1/I2, D7/D8 |
| `interactions.md` (greeting, listen-in, offer, settle, notebook, presence, no-streaks) | §5.3/5.6/5.8/5.9/5.12, §8.5, §4.3, I5 |
| `aviary_layout.md` (scene, perches, day/night, weather, chrome, loading) | §7.3–7.7, §5.7, D5/D15/D16 |
| `accounts_sync.md` (auth, UUID rule, tick, sync, no-LWW, privacy, export, deletion) | §3.1, §4, §5.1, §6, §11, §13, D10/D11 |
| `social_optional.md` (visits and their refusals) | §4.5, §3.4, D9, §11 (no leaderboard aggregates), I5 |
| `accessibility_perf.md` (narration, reduced-motion, captions, keyboard, budgets, fallback) | §9, §7.5, §8.6, §12, §14.5–14.6 |
| `non_goals.md` | §1.2, I5, R11, §11 |

---

## 19. Deliberately deferred to build (named by the PRD as calibration-time)

Exact presence window W; final tick cadence; final mood-set tuning and transition matrices; offer cooldown value; all drift constants (harness-owned); notebook θ dynamics; palette/contrast token values and focus treatment (design system, designer-owned); min/max viewport handling details (rendering spec); species art and motif content (visual + sound designers, against the §14.5 gates); email-provider selection.

Everything else in this document is decided.
