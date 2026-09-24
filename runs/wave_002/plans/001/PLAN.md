# Pocket Aviary — v1 Implementation Plan

Status: executable plan for a frontier engineering team. Source: the Pocket Aviary PRD (`product_brief`, `concepts`, `bird_engine`, `interactions`, `aviary_layout`, `accounts_sync`, `social_optional`, `accessibility_perf`, `non_goals`).

This plan turns the spec into concrete engineering decisions. When the PRD is ambiguous or contradicts itself, the plan makes a call and records it in §15 (Decision log). Every number labelled *initial* is a starting calibration constant. It lives in versioned engine config and is tuned with the harnesses in §5.9 and §17.

---

## 0. The seven invariants everything else serves

These are the rules that break the product silently if they're violated. Each one has a named enforcement mechanism, and a PR that weakens any of them needs sign-off from both the engine lead and the product lead.

| # | Invariant | Enforcement |
|---|---|---|
| I1 | **Server is the only writer of personality, mood, perch, and weather.** Clients write only interaction events. | DB role grants: the API role has no `UPDATE` on `bird_traits` or `aviary_state`. Only the tick role does. No API route accepts trait values. |
| I2 | **Drift is monotonic non-decreasing, additive, and server-authored.** | A DB trigger on `bird_traits` rejects any decrease. The tick asserts the invariant before commit and fails closed (keeps the previous state and pages someone). Property tests cover it. |
| I3 | **Presence = visible ∧ focused ∧ recent pointer/key activity.** Presence-time is the union of intervals across devices, never the sum. | Client presence state machine with exhaustive tests. Server interval merge and clamping. Playwright e2e tests toggle each signal separately. |
| I4 | **Personality values are never shown numerically on any product surface.** | Snapshots carry derived, quantized *expression parameters*, not raw traits. A lint rule forbids trait field names in client code. There is no debug overlay in production builds. |
| I5 | **Stable bird identity.** `bird_id` is minted once and never reissued, replaced, or regenerated. | UUIDv7 minted at first appearance. Migrations are forbidden from deleting or re-creating `bird` rows (CI migration linter). Species definitions are versioned and kept forever. |
| I6 | **Email is PII and lives in exactly one place.** Every other reference uses the synthetic `account_id` UUID. | Encrypted email column plus a keyed blind index on the account row, both touched only by the auth module. Structured-log field allowlist. PII scanner on the log pipeline. CI test that greps staging logs and telemetry for email patterns. |
| I7 | **No announcement, gamification, or user-behavior surfaces.** No toasts, no streaks, no counts, no "welcome back." | Toast, snackbar, and badge libraries are banned in the dependency allowlist. A copy linter runs on every string and generator output. A design-review gate covers new surfaces. The notebook generator has no access to user-behavior aggregates (§5.8). |

---

## 1. Scope

### 1.1 In v1

- Single-user accounts with email magic-link sign-in, per-device revocable sessions, verified email change, JSON export, and soft-then-hard deletion over 30 days.
- One canonical aviary per account. Two starter birds chosen by the system, named by the user. The cap is 7 birds. New birds become available by aviary age only.
- A server-side simulation tick (~60 s) covering personality drift, mood, perch choice, weather, bird-to-bird mood contagion, and notebook observation extraction.
- A browser client with:
  - a single horizontal responsive scene, three perch zones, local-time day/night, rare weather, and ambient micro-motion
  - procedural idle motion
  - procedural WebAudio calls, chorus, and listen-in mix
  - offer (seed, song fragment, still pool) and settle with a 5 s undo
  - the field notebook
  - the return-greeting
- Multi-device sync as an architectural property: pull snapshots and push append-only events.
- Visit invitations: read-only ambient view, per-invite opt-in, revocable, 30-day expiry if unused, silent visit log, optional host notification (off by default).
- Accessibility, all at launch:
  - naturalist screen-reader narration
  - call captions generated from the actual call grammar
  - a designed reduced-motion mode
  - full keyboard navigation
  - WCAG AA contrast on all copy
- Performance budgets: <2 MB gz initial JS, first bird visible in <500 ms on mid-tier mobile over 4G, 60 fps idle on a 5-year-old laptop, no memory growth over 30 minutes (tested in CI).
- Aggregate-only operational telemetry and synthetic monitoring.
- A matter-of-fact unsupported-browser surface (last two majors of Chrome, Safari, Firefox, and Edge are supported).

### 1.2 Explicitly not in v1 (and not designed for)

Native apps, payments, shared or household aviaries, multiple aviaries per account, customizable scenes, bird catalogs, drag-to-place, public discovery, profiles, follows, comments, chat, avatars, co-presence, leaderboards, achievements, badges, levels, streaks, visit calendars, XP, "birds adopted" counters, push notifications, product-initiated email about the aviary, hunger or death or distress, happiness meters, a recorded-audio fallback, trait dashboards, trait debug views in production, population-level analytics on interaction data, third-party analytics or session-replay SDKs, and ML training on per-bird data. Protocols and data models are **not** shaped for native clients (per `non_goals.md`).

System emails that do exist: the magic link, email-change verification and the notice to the old address, the export-ready link, the deletion-scheduled confirmation, visit invitations (to the visitor), and visit notifications only when the host has opted in. None of them is about the aviary's state.

---

## 2. Architecture

### 2.1 Service shape

```
                 ┌───────────────────────── Browser ─────────────────────────┐
                 │ Boot renderer (inline) → Scene renderer (WebGL2/Canvas2D) │
                 │ Behavior layer · Audio engine (AudioWorklet) · A11y layer │
                 │ Presence FSM · Event outbox · Snapshot client             │
                 └──────────────┬───────────────────────────┬───────────────┘
      HTML + inline snapshot    │ GET /v1/aviary/snapshot   │ POST /v1/aviary/events
      (streamed from edge)      │ (ETag = tick_seq)         │ (batched, idempotent)
                 ┌──────────────▼───────────────────────────▼───────────────┐
                 │ Edge (CDN + edge function): shell HTML, static assets,    │
                 │ session-cookie verify, inline snapshot from regional cache│
                 └──────────────┬────────────────────────────────────────────┘
                 ┌──────────────▼──────────────┐     ┌──────────────────────┐
                 │ API service (stateless, TS) │     │ Jobs service         │
                 │ auth · events · snapshot    │     │ exports · deletions  │
                 │ notebook · settings · visits│     │ invite expiry · email│
                 │ offer reaction resolver     │     └─────────┬────────────┘
                 └───────┬───────────────┬─────┘               │
                  append │ events   read │ snapshot            │
                 ┌───────▼───────────────▼─────────────────────▼─────────────┐
                 │ Postgres (logical shards by aviary_id)                     │
                 │ accounts · sessions · events · aviary_state · bird_traits  │
                 │ birds · notebook · observations · invites · visits         │
                 └───────▲─────────────────────────────┬─────────────────────┘
                         │ state + traits (tick role)  │ snapshot projection
                 ┌───────┴─────────────────────────┐   ▼
                 │ Simulation service (tick workers│  Redis: snapshot cache,
                 │ + shard lease scheduler, TS)    │  rate limits, revocations
                 └─────────────────────────────────┘

   Separate plane (no network path to the simulation DB):
   RUM collector (cookie-less) → metrics store · Synthetic browser fleet · OTel metrics/logs
```

**Language and runtime.** TypeScript end to end. A shared `@aviary/engine` package holds pure, deterministic functions: the tick step, the offer-reaction resolver, the expression projection, the greeting planner, the call grammar, the caption and narration generators, and the voice linter. The simulation service and API run on pinned Node LTS. The client imports only the presentation-side modules (greeting planner, call grammar, caption and narration generators). The server-only modules (drift, mood, tick) are excluded from the client bundle by package export maps. That keeps invariant I1 structural and keeps trait semantics out of the browser.

**Datastores.**
- **Postgres** is the single system of record. There are 1024 logical shards keyed by `hash(aviary_id)`, mapped to physical clusters. Launch runs on one HA cluster with a synchronous standby and PITR, and the logical sharding is in place from day one so clusters can be split later without re-keying.
- **Redis** holds the snapshot cache, rate-limit counters, and the session-revocation set. It's a cache and never a source of truth.
- **Object storage** holds exports, which are encrypted and expire.

**Why not an event-streaming platform.** Event volume per aviary is tiny: tens of events per session. A per-shard append-only Postgres table with a per-aviary monotonic sequence gives ordered, idempotent consumption with a single transactional commit alongside the state write, and there's no second system to operate. We'd revisit this only if write volume exceeds what sharded Postgres handles comfortably, which is far beyond the v1 horizon.

### 2.2 Client/server split and the render-pipeline boundary

The boundary is **canonical simulation state** (server) versus **presentation realization** (client). Nothing crosses from client to server except interaction events and a small set of observation reports.

| Owned by server (canonical, in snapshot) | Owned by client (presentation, never sent back as state) |
|---|---|
| Bird roster, identity, species, voice print, names | Micro-motion selection and animation (preen, scan, tilt, shuffle, breathe, blink) |
| Personality (never sent raw) → expression parameters | Call realization from grammar, call scheduling, call-and-response, emergent chorus |
| Mood label + mood intensity per bird | Ambient ornaments (leaves, feathers, parallax) |
| Perch assignment + scheduled perch moves (timeline) | Lighting interpolation from time of day |
| Weather state + schedule for the next window | Audio mix, listen-in ramps, reverb |
| Scheduled salient events (alarm calls, offer reactions) | Narration prose timing, caption rendering |
| Aviary timezone, absence anchor for the greeting | Greeting realization (planner runs client-side over server inputs) |
| Active offer items (seed/pool) with lifetimes | Settle lighting (device-local), top-bar fade |

Observation reports sent from client to server: `greeting.observed` (which birds greeted and in what order), used only by the notebook. The server treats them as observations. They never mutate traits.

### 2.3 Environments

- `dev`, `staging`, and `prod`, plus **`calibration`**: a physically separate deployment with its own database, used for staff dogfooding with explicit consent and for time-compressed simulations (§5.9, §14.2). Production interaction data never flows into `calibration`, and `calibration` data never flows into analytics.

---

## 3. Data model

All IDs are UUIDv7 unless noted. All timestamps are server UTC. The "local date" is computed from the aviary timezone.

### 3.1 Identity and accounts (auth module only)

```
account
  account_id            uuid PK                -- synthetic, the only identifier used anywhere
  email_ciphertext      bytea                  -- AES-256-GCM, per-account DEK wrapped by KMS
  email_blind_index     bytea UNIQUE           -- HMAC-SHA256(pepper, normalized email); lookup only
  dek_wrapped           bytea                  -- destroyed at hard delete (crypto-shredding)
  timezone              text                   -- IANA, e.g. "America/Chicago"
  tz_updated_at         timestamptz
  created_at            timestamptz
  deletion_requested_at timestamptz NULL       -- soft delete marker
  hard_delete_after     timestamptz NULL       -- deletion_requested_at + 30d
  settings              jsonb                  -- see §3.6

email_change_request
  request_id, account_id, new_email_ciphertext, new_email_blind_index,
  token_hash, expires_at (15 min), consumed_at

magic_link_token
  token_hash bytea PK   -- SHA-256 of 256-bit random token; raw token only in the email
  purpose               -- 'sign_in' | 'email_change' | 'export_download'
  account_id NULL       -- NULL for first sign-in (account created on consumption)
  pending_email_ciphertext NULL, pending_blind_index NULL
  created_at, expires_at (created_at + 15 min), consumed_at NULL

session
  session_id uuid PK, account_id, token_hash, created_at, last_seen_at,
  device_label text     -- coarse: "Safari on iPhone", derived from UA at creation
  revoked_at NULL, expires_at (sliding 60 days of inactivity)
```

The blind index is stored on the account row and read only by the auth module during sign-in. It's never logged, never used as a key elsewhere, and never leaves the auth module. That preserves "email stored once, on the account record" (I6) while still allowing lookups.

### 3.2 Aviary and birds

```
aviary
  aviary_id uuid PK, account_id uuid UNIQUE, created_at
  -- created_at drives new-bird availability (§5.7)

bird                                            -- identity; writable by API only for name
  bird_id uuid PK                               -- minted once, never reissued (I5)
  aviary_id, species_id, species_version        -- species defs are versioned & retained forever
  name text                                     -- user-assigned, renameable
  voice_print jsonb                             -- fixed at mint; see §10.1
  status  'visiting' | 'adopted'                -- 'visiting' = candidate new bird (§5.7)
  first_seen_at, adopted_at NULL
  seed bigint                                   -- per-bird PRNG root

bird_traits                                     -- canonical personality; tick role only
  bird_id PK, aviary_id
  boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity   -- float8 in [0,1]
  res_boldness, res_warmth, res_vocal, res_plumage, res_curiosity           -- drift reservoirs (§5.2)
  daily_applied jsonb                           -- per-trait delta applied in current local day (cap)
  updated_tick_seq bigint
  -- trigger: NEW.x >= OLD.x for every trait, else RAISE (I2)

aviary_state                                    -- canonical fast-timescale state; tick role only
  aviary_id PK
  tick_seq bigint                               -- CAS guard
  engine_version text
  event_cursor bigint                           -- last consumed event seq
  sim_time timestamptz                          -- time the state represents
  state jsonb                                   -- typed & versioned (zod/protobuf schema):
     birds[]: { bird_id, mood_latent{arousal,ease,interest}, mood_label, mood_since,
                attunement, perch{zone,slot,since}, offer_cooldown_until,
                listen_in_today_s, credited_offers_today }
     weather: { kind: none|rain|wind, started_at, ends_at, intensity }
     weather_schedule_day: date
     timeline[]: scheduled events for next ~2 min (perch moves, alarm calls, offer reactions)
     active_items[]: { item_id, kind, placed_at, expires_at }
     presence: { last_presence_end_at, presence_today_eff_min }
     rng_state
  snapshot jsonb                                -- precomputed client projection (§4.2)
  visitor_snapshot jsonb                        -- precomputed visitor projection (§7)

bird_trait_checkpoint                           -- daily per-bird copy for recovery & invariant audits
  bird_id, local_date, traits(5), reservoirs(5)  -- retained for account lifetime; deleted with account
```

### 3.3 Interaction events (append-only)

```
interaction_event
  aviary_id, seq bigint                          -- server-assigned per-aviary monotonic; PK(aviary_id, seq)
  event_id uuid, session_id uuid, client_seq int -- UNIQUE(session_id, client_seq) → idempotency
  type text, payload jsonb (schema per type, small), resolution jsonb NULL (offers)
  client_ts timestamptz (advisory only), received_at timestamptz
  retention: raw rows deleted 30 days after consumption
```

Event types (full list, versioned schemas):

| type | payload | notes |
|---|---|---|
| `session.start` | `{tz, reduced_motion, audio_mode}` | the tz report drives aviary timezone (§5.6) |
| `presence.segment` | `{start, end, audio: audible\|captions\|muted}` | emitted every 15 s while present; server clamps and merges |
| `presence.end` | `{reason: hidden\|blur\|idle\|settle\|pagehide}` | terminal; no semantic difference between reasons for drift (§5.2) |
| `listen_in.start` / `listen_in.end` | `{bird_id}` / `{bird_id, duration_s}` | server recomputes duration from its own clock and caps it |
| `offer` | `{kind: seed\|song\|pool, fragment_id?, focus_bird_id?}` | resolved synchronously (§5.5); resolution stored in row |
| `settle.commit` | `{}` | sent only after the 5 s undo window elapses |
| `greeting.observed` | `{bird_ids_in_order, absence_bucket}` | notebook input only |
| `audio.mode` | `{audible\|captions\|muted}` | minor vocal-drift input (§5.2) |

No raw pointer coordinates, keystrokes, or timings are ever sent. Presence is derived on the client and sent only as interval segments.

### 3.4 Presence

```
presence_interval
  aviary_id, start, end, merged boolean          -- union across devices; 30-day retention
```

### 3.5 Notebook and observations

```
observation                                      -- structured aviary happenings, input to notebook
  aviary_id, obs_id, local_date, kind, bird_ids[], facts jsonb, salience float
  retention: 60 days (enough for "first time this week/month" comparisons)

notebook_entry
  entry_id, aviary_id, written_at, local_date, weekday_label
  text                                           -- final prose (lowercase, naturalist)
  kind, bird_ids[], generator_version, source_obs_ids[]
  retention: account lifetime; read-only for the user (no update/delete endpoints)
```

### 3.6 Settings, visits

```
account.settings: {
  a11y:  { reduced_motion: 'system'|'on'|'off', captions: bool, narration_visible: bool,
           keep_top_bar_visible: bool },
  audio: { mode: 'audible'|'muted' },
  visits:{ notify_on_visit: false }             -- default OFF
}

visit_invite
  invite_id, host_account_id, visitor_email_ciphertext (host's DEK), token_hash
  created_at, expires_at (created_at + 30d, applies while unused), first_used_at NULL, revoked_at NULL

visitor_session
  visitor_session_id, invite_id, token_hash, created_at, last_seen_at, ended_at, revoked_at
  -- duration shown to host = last_seen_at - created_at, rounded ("about 20 minutes")
```

Visitor emails are encrypted under the host's DEK. They're host-owned data and are deleted with the host account.

### 3.7 Data retention summary

| Data | Retention |
|---|---|
| Traits, bird identity, notebook, settings | account lifetime |
| Raw interaction events, presence intervals | 30 days after consumption |
| Observations | 60 days |
| Export files | 72 h |
| Backups (PITR) | 35 days; per-account DEK destruction makes deleted accounts unreadable in older backups |

---

## 4. API surface

REST/JSON over HTTPS, versioned under `/v1`. Auth uses the httpOnly, Secure, `SameSite=Lax` session cookie. State-changing requests require an `Origin` check and a CSRF token header. All errors return matter-of-fact copy keys (§10.6), not naturalist ones.

### 4.1 Auth and account

| Method & path | Purpose |
|---|---|
| `POST /v1/auth/link` `{email}` | Always `202` (no enumeration). Rate limits: per blind index 5 per 15 min and 20 per day; per IP 30 per hour. Sends the magic link. |
| `GET /v1/auth/confirm?t=…` | Renders a minimal page that **auto-POSTs** via JS. Email link scanners that prefetch GETs don't consume the token. |
| `POST /v1/auth/consume` `{t}` | Validates hash, expiry (15 min), and unused status. Marks consumed atomically. Creates the account and aviary on first sign-in. Issues the session cookie. |
| `POST /v1/auth/signout` | Revokes the current session. |
| `GET /v1/account` · `GET /v1/account/sessions` · `DELETE /v1/account/sessions/:id` | Session list and revocation. Revocation is written to Redis immediately and checked at edge and origin. |
| `POST /v1/account/email-change` `{email}` → `GET/POST …/verify` | The old email keeps working until the new one verifies. A matter-of-fact notice goes to the old address. |
| `POST /v1/account/export` | `202`. A job builds the JSON and emails a link to the verified address. `GET /v1/exports/:id?t=…` requires the link token **and** a signed-in session. |
| `POST /v1/account/delete` · `POST /v1/account/restore` | Soft delete, and "I changed my mind." |
| `GET/PATCH /v1/settings` | Per-field last-write-wins is acceptable here (§6.4). |

### 4.2 Aviary state

**`GET /v1/aviary/snapshot`** returns the precomputed projection. Uses `ETag: "<tick_seq>"`, with `304` when unchanged. Typical size is 2–6 KB gz. Example (abridged):

```json
{
  "v": 3, "tick_seq": 918273, "sim_time": "2026-10-02T13:04:00Z", "server_now": "2026-10-02T13:04:07Z",
  "tz": "America/Chicago",
  "absence": { "last_presence_end_at": "2026-10-01T02:11:00Z" },
  "weather": { "kind": "rain", "started_at": "...", "ends_at": "...", "intensity": 0.4 },
  "birds": [{
    "id": "0192…", "name": "pip", "species": "wren_like@2",
    "voice": { "base_pitch_st": 3.2, "timbre": [0.62, 0.18, 0.4], "motifs": ["m3","m7","m11"], "rhythm": "lilt" },
    "mood": { "label": "curious", "intensity": 0.6 },
    "perch": { "zone": "front", "slot": 1, "since": "..." },
    "expr": { "greet": 0.70, "approach": 0.55, "call_rate": 0.45, "pitch_span": 0.35,
              "plumage": 0.53, "tilt": 0.65, "preen": 0.30, "scan": 0.40, "quiet": 0.20 }
  }],
  "timeline": [
    { "at": "...+00:23", "kind": "perch_move", "bird": "0192…", "to": {"zone":"middle","slot":2} },
    { "at": "...+01:10", "kind": "alarm_call", "bird": "0193…" }
  ],
  "items": [{ "id": "…", "kind": "pool", "placed_at": "...", "expires_at": "..." }],
  "adoption": { "visiting_bird_id": null }
}
```

`expr.*` values are quantized to 1/20 steps and bounded. They're behavior propensities computed from traits, mood, attunement, and time of day (§5.4). They're rendering inputs, not traits. Raw trait values never appear in snapshots (I4).

**`POST /v1/aviary/events`** takes `{events: [{client_seq, type, ts, payload}]}` with up to 50 per batch. Returns `{acked_through: client_seq, resolutions: {client_seq: …}}`.
- Idempotent on `(session_id, client_seq)`.
- The client flushes every 15 s, immediately for `offer`, and on `pagehide` via `fetch(…, {keepalive: true})` or `sendBeacon`.

**`POST /v1/aviary/offer`** is a convenience wrapper for a single `offer` event. It returns the **reaction script** synchronously, targeting ≤150 ms p95 server time (§5.5).

**`GET /v1/aviary/notebook?before=<entry_id>&limit=20`** paginates backwards indefinitely. It's read-only, and no mutation endpoints exist.

**`PATCH /v1/birds/:bird_id` `{name}`** renames. The name is stored lowercase-insensitive for display and trimmed to 1–24 characters. Renaming affects nothing else.

**`POST /v1/adoption/:bird_id/accept` `{name}`** adopts a visiting bird (§5.7). **`POST /v1/onboarding/names` `{names: {bird_id: name}}`** names the starter pair.

### 4.3 Visits

| Method & path | Purpose |
|---|---|
| `POST /v1/invites` `{email}` | Host only. Creates an invite and emails the visitor a one-time link. |
| `GET /v1/invites` | Outstanding invites plus the visit log (visitor email, date, approximate duration), newest first. |
| `DELETE /v1/invites/:id` | Revokes immediately. Marks the invite and its visitor sessions revoked and pushes to the revocation set. |
| `GET /v1/visit/confirm?t=…` → `POST /v1/visit/claim` `{t}` | Consumes the one-time token and mints a visitor-session cookie scoped to `(aviary_id, invite_id)` with a separate audience. |
| `GET /v1/visit/snapshot` | Checks invite and visitor-session status **on every call** (not cached). If revoked, expired, or the host is soft-deleted, returns `410` with the copy key `visit_unavailable`. Otherwise returns `visitor_snapshot`. |

Visitor tokens are rejected by every other endpoint. No event endpoint accepts visitor audiences, so visitors can't create presence or interaction events (enforced at the auth middleware and covered by tests).

---

## 5. Simulation engine

### 5.1 Tick architecture

- **Cadence.** Every aviary ticks every 60 s (initial; configurable between 30 and 120 s) whether or not any client is connected.
- **Scheduling.**
  - The 1024 logical shards are leased to tick workers using Postgres advisory locks and a lease table, with a 30 s renewal.
  - A worker iterates its shards' aviaries on a staggered schedule: aviary `a` ticks at `hash(a) mod 60` seconds past each minute, which spreads load evenly.
  - Worker holds hot state in memory while it holds the lease, and reloads on lease acquisition.
- **Step function.** `state', traits' = step(state, traits, events(cursor, head], t, dt, config, rng)`. It's pure and deterministic. The PRNG is xoshiro128** seeded from `(aviary.seed, tick_seq)`. No `Date.now` or `Math.random` inside the engine (lint-enforced).
- **Commit.** One transaction:
  1. `UPDATE aviary_state … WHERE aviary_id=$1 AND tick_seq=$prev` (CAS)
  2. `UPDATE bird_traits` (monotonic trigger)
  3. `INSERT observation`, and `notebook_entry` when warranted
  4. Advance `event_cursor`.

  If the CAS fails, another worker won. Discard and reload.
- **After commit.** Write `snapshot` and `visitor_snapshot` to Redis (`snap:{aviary_id}`), then to the edge-readable regional cache.
- **Catch-up.** Every continuous process in the step uses exact exponential solutions, so `step` accepts variable `dt`:
  - mood relaxation `x += (target − x)(1 − e^{−dt/τ})`
  - reservoir drain `Δ = R(1 − e^{−dt/τ_r})`
  - attunement decay

  After an outage, a single catch-up step with `dt = gap` (sub-stepped per local hour so time-of-day attractors stay right) produces the same continuous state as minute-by-minute ticking. Discrete stochastic events (alarms, perch moves) are regenerated from the deterministic per-day schedule rather than replayed.
- **Latency.** Target p50 <5 ms per aviary step and p99 <50 ms. The alarm is tick-latency p99 >5 s (per PRD), measured from scheduled time to commit, so it also catches scheduling lag.
- **Scale math.** 200k aviaries is ~3.3k commits/s, each updating ~2 rows of ~2–4 KB. That's comfortable on one HA cluster with HOT updates. Splitting logical shards across clusters takes us past 2M aviaries.
- **Dormant aviaries.** v1 does not reduce cadence for dormant accounts; the PRD says the tick runs regardless. A lower-cadence dormancy tier would be a later cost lever, allowed only once the variable-`dt` equivalence test (§17) proves it produces results identical to minute ticking for dormant aviaries.

### 5.2 Drift function (slow timescale)

Traits `T ∈ [0,1]`. Drift is a **two-stage low-pass filter**: signals flow into per-trait reservoirs `R`, and reservoirs drain into traits over days. Because reservoirs keep draining after the user leaves, "personality drifts during absence based on inputs from before they left" is literally true.

For each bird and trait `k`, each tick:
```
R_k   += Σ inputs_k(events consumed this tick)          -- inputs ≥ 0 always
drain  = R_k · (1 − e^{−dt/τ_r})                         -- τ_r = 4 days (initial)
ΔT_k   = min( drain · (1 − T_k)^γ ,  cap_day − applied_today_k )   -- γ = 1, cap_day = 0.006
T_k   += max(0, ΔT_k);   R_k −= drain;   applied_today_k += ΔT_k
```

- **Monotonic.** Inputs and deltas are non-negative, and there is no decay term on `T`. Neglect means the reservoirs empty and drift stops. Nothing goes down (I2).
- **Headroom `(1−T)`.** Traits approach 1 asymptotically, so long-lived aviaries keep drifting slowly for years instead of hitting a ceiling.
- **Daily cap (`cap_day`).** Hard guard against saturation from heavy or fraudulent presence. No single day can move a trait by more than 0.006, and the visibility threshold is ~0.05 (§5.9), so **a single session can never visibly move a trait**.

**Inputs** (initial gains, into reservoirs):

| Source | Definition | Traits affected (gain) |
|---|---|---|
| Presence (dominant) | Effective presence per local day: `p_eff = C·(1 − e^{−p/C})`, C = 20 min, where `p` is merged presence minutes that day. Each tick adds the increment in `p_eff`. | boldness 1.0e-3/min, warmth 0.8e-3, plumage 1.0e-3, curiosity 0.6e-3, vocal 0.6e-3 (+0.3e-3 when audio mode is `audible` or `captions`) — applied to every bird |
| Listen-in | Server-measured seconds focused on bird *b*, capped at 10 min per bird per local day | *b*: warmth +0.6e-3/min, vocal +0.6e-3/min |
| Offer, near | Primary recipient of any offer, if not on cooldown | *b*: boldness +0.002 |
| Offer, accepted | Resolution says `accepted` | *b*: curiosity +0.004 |
| Settle | None for drift. Mood-quieting impulse only (§5.3). Ends presence the same as tab close. | — |

Presence saturation (`C`) and the per-bird offer cooldown (4 min, initial) plus a cap of 6 credited offers per bird per day stop curiosity from saturating in one session, which is the PRD's stated reason for the cooldown.

**Why captions count as audible for vocal drift.** The brief says muting versus letting calls play shapes drift. Deaf and hard-of-hearing users attend to calls through captions and must not get a slower-drifting aviary. Only an explicit mute without captions forgoes the small vocal bonus, and even then nothing decreases.

### 5.3 Mood (fast timescale)

**Representation.** Each bird has a continuous latent `m = (arousal, ease, interest) ∈ [0,1]³` and a discrete label derived with hysteresis.

**Label set (final for v1):** `alert`, `curious`, `content`, `wary`, `drowsy`, `roosting`. `roosting` is the internal name for the "settled / eyes closed low on the perch" night state; in prose it renders as "settled". It's distinct from the user's settle gesture.

| Label | Region (initial thresholds with ±0.05 hysteresis) |
|---|---|
| roosting | arousal < 0.15 and local night |
| drowsy | arousal < 0.30 |
| wary | ease < 0.35 |
| alert | arousal > 0.70 and ease ≥ 0.35 |
| curious | interest > 0.60 |
| content | otherwise (ease ≥ 0.55 typical) |

Priority order is roosting > wary > drowsy > alert > curious > content. There's a minimum dwell of 10 min per label unless an impulse crosses a threshold by more than 0.15 (so an alarm can flip a bird to wary immediately).

**Dynamics per tick.** `m += (A(t) − m)(1 − e^{−dt/τ_m}) + impulses`, with τ_m = 90 min by day and 45 min for the dusk and dawn transitions. The attractor `A(t)` is composed from:
- **Time of day** (aviary local time): dawn raises arousal (alert), midday is neutral, dusk lowers arousal (drowsy), night pulls toward roosting. The nightjar-like species inverts the arousal curve and stays active at night.
- **Personality:** boldness raises ease; curiosity raises interest; warmth raises ease when near other content birds.
- **Attunement** (§5.3.1): low attunement lowers arousal and interest (quieter), and **never lowers ease**, so absence never produces wariness.
- **Weather:** rain lowers arousal and applies a vocal-rate multiplier of 0.5 while it lasts plus 10 min afterward. Wind raises arousal for birds with boldness ≥0.5 and lowers ease by 0.1 for birds below.

**Impulses.**

| Trigger | Effect |
|---|---|
| Accepted offer | +ease 0.15, +interest 0.1 |
| Offer ignored while wary | none |
| Listen-in | +interest 0.05 per minute on the focused bird |
| Settle | −arousal 0.1 on all birds |
| Alarm call from bird X | −ease 0.2·(1 − boldness) on birds in the same or adjacent zone |

Wary contagion follows from the alarm impulse. High-warmth content birds nearby add +ease 0.02 per tick to their neighbours ("wary spreads", and so does calm, more slowly).

**Daily-ish reset without snapping.** There's no reset operation. The overnight roost attractor washes out session impulses over ~8 h, which produces the "daily-ish cadence" while mood persists continuously across sessions and never snaps on tab open.

#### 5.3.1 Attunement (the "ambient, not punished" mechanism)

The PRD requires both that traits never decrease and that a user returning after two weeks finds birds that are "quieter than they were" and "greeting less often". Monotonic personality can't produce the second, so a separate hidden medium-timescale scalar handles it:

- `attunement ∈ [0.25, 1]` per bird.
- It rises quickly with presence: `+ (1 − a)·p_eff/30` per effective minute, so it recovers within one or two sessions.
- It relaxes toward the floor of 0.25 with a 7-day half-life when there is no presence.
- It only scales **expression**: greeting propensity, unobserved call rate, and front-perch preference. It never affects ease (no wariness), mood labels such as distress, or traits.
- It is never shown, never named in copy, and has no "low" visual state other than birds being calmer and quieter.

This is not a Tamagotchi meter. It has no failure state, no visible decay, a floor that's still a living, calling aviary, and full recovery by being present. Recorded in the decision log (D3).

### 5.4 Perch choice and expression projection

- **Perch utility.** `U(zone) = w_b·boldness·z + w_a·attunement·z − w_w·[wary]·z + w_s·social(neighbours) + w_d·[drowsy/roosting]·(−z) + noise`, with front `z = 1`, middle `0`, back `−1`.
- **Moves.** A bird re-evaluates every tick but moves only if utility exceeds the current zone by a margin of 0.2. There's a minimum dwell of 8 min except for impulses. Within-zone slot hops happen at most every 3 min.
- **Timeline.** Moves are written to `timeline` with a random offset inside the next tick window (0–55 s), so clients animate them at the right moment instead of teleporting.
- **Slots.** Front has 2, middle 3, back 3. Seven birds always fit. The roost positions at night are the back and middle slots, low on the perch.
- **Expression projection.** `expr` is computed per tick from traits, mood, attunement, time of day, and weather. Examples:
  - `greet = σ(3·(0.6·boldness + 0.4·warmth − 0.45)) · attunement · moodReceptivity`
  - `call_rate = vocal · attunement · weatherMult · timeOfDayMult`
  - `plumage = plumage_saturation` (quantized)

  Everything is quantized and bounded, so the client can render personality without learning trait values.

### 5.5 Offer reaction resolver (synchronous, deterministic)

Offers must react immediately, not on the next tick.

1. The API receives the `offer` event, loads the current canonical state from Redis or the DB, and calls `resolveOffer(state, traits, offer, seed = event_id)`. This is a pure function from `@aviary/engine`.
2. **Recipient selection.** The listen-in bird is primary if one is focused. Otherwise pick the highest of `curiosity·moodApproachability·zoneProximity`, excluding roosting birds and birds on cooldown.
3. **Reaction choice**, by kind and mood:
   - **Seed:** curious or content birds approach, with delay 0.5–2 s. Wary birds wait 8–40 s, then approach with probability ~boldness. Drowsy birds glance at most. Alert birds approach fast.
   - **Song fragment:** join in (vocal high and content/curious), go quiet (wary or drowsy), or call against (alert with high vocal).
   - **Still pool:** drink, bathe, or watch, weighted by boldness and curiosity. Up to 2 other birds may also visit the pool within its 8 min lifetime.
4. **Output.** A reaction script: a list of `{bird, action, at_offset_ms, duration_ms}` plus `accepted` per bird. It's stored in the event row's `resolution`, returned to the client, and inserted into the next snapshot's `timeline` and `items` so other devices render the same outcome.
5. The tick later consumes the event and reads `resolution` to apply drift and mood. It never re-rolls.

**When every eligible bird is on cooldown,** the item still appears and birds glance at it without approaching. **No cooldown message ever appears.** The cooldown stays invisible (I7).

**Latency hiding.** On click, the client immediately places the item and starts a universal "notice" beat: nearby birds turn their heads toward it over 300–600 ms. The reaction script then takes over. If the request fails, the notice beat resolves into birds watching and the event goes into the outbox for retry.

### 5.6 Time, timezone, weather

- **Timezone.** Each `session.start` reports the IANA timezone. The aviary timezone changes to the newest device-reported value only if that value has been reported by sessions spanning ≥2 h, or immediately if the account has only one active device. Lighting and mood attractors glide to the new local time over 20 min, with no jump. DST is handled by IANA conversion.
- **Day curve.** Fixed civil schedule: dawn 05:30–08:00, day, dusk 18:00–20:30, night. There's no geolocation, so no latitude or seasonal sunrise. (Decision D8.)
- **Weather.** Generated deterministically per aviary per local day from `(aviary.seed, local_date)`:
  - Rain is a Poisson process at ~3 per week, 8–25 min each, soft intensity.
  - Wind gusts come ~1–2 per day, 1–5 min each.
  - There are no storms or snow.

  Weather is canonical and in the snapshot, so every device and visitor agrees.
- **Alarm calls.** Rare (~0–3 per day). Triggered by wind-gust startle (probability scales with `1 − boldness`) or a wary transition. They're scheduled into the `timeline` so every client plays the alarm and the contagion stays consistent.

### 5.7 Adoption and new birds

- **Starter pair.** On account creation the engine selects two distinct species from the pool of ~6. Sampling is constrained so their seeded boldness differs by ≥0.15 (one bird reliably greets first; greeting order stays legible) and their voice prints are ≥ the recognizability distance apart (§10.6).
  - Seed traits: species baseline ±0.08 jitter, typically 0.20–0.50.
  - `bird_id` values are minted at this point.
- **Onboarding.** Onboarding is a product surface in naturalist voice: "two birds have arrived." There's one name field per bird with a suggested name drawn from a per-species list, editable, plus one "done" control.
  - Then comes the quiet empty field, the first bird's soft fly-in to its seeded perch after ~1.5 s, and the second bird's fly-in 2–6 s later.
  - This is the only time an empty aviary is shown.
- **New-bird availability by aviary age (initial).** Third bird at ≥90 days, 4th at ≥150, 5th at ≥240, 6th at ≥365, 7th at ≥540. That lines up with "a few months → third; a year → five or six." Presence and visit count never factor in.
- **How a new bird appears (no announcement).**
  - Once an aviary becomes eligible, a new-species bird (status `visiting`, `bird_id` minted now) begins occasionally appearing on a back-perch slot during some ticks. It's alive, calls, and is listen-in-able.
  - After it has appeared on ≥3 local days, the notebook writes one observation ("a small finch has been visiting the far branch these past mornings.").
  - That entry carries a quiet inline control, "welcome it," which opens naming. The same option is listed under Settings → Birds.
  - If the user never acts, the visitor keeps visiting occasionally with no reminders. The visiting bird's identity carries over into adoption: it's the same bird.
- **Rollout gate.** `max_birds` in engine config sets the adoptable ceiling (§14.2).

### 5.8 Notebook generation

- **Observation extraction** runs in the tick. It emits structured `observation` rows for noteworthy aviary happenings:
  - greeting order, with comparisons ("first time this week pip greeted first")
  - a bird settling on the front perch for the first time in N days
  - chorus events: ≥2 birds with high `call_rate` in the same window, reported by the tick's timeline, or inferred from the chorus rule
  - weather plus bird reactions
  - notable offer reactions (a first pool bath)
  - the nightjar calling late
  - a visiting bird
  - long quiet preening stretches
  - plumage drift crossing a perceptual threshold ("wren's wing bars look brighter than they did in the spring")
- **Sparsity controller.** Token bucket per aviary: +1 token every 3 days, capacity 2. Writing an entry costs 1. Observations with salience ≥0.9 may overdraw once. Hard limits are 1 entry per local day and 3 per rolling 7 days, **regardless of how active the user is**. Low-salience observations just expire.
- **Prose generation.** A hand-authored generative grammar (Tracery-style with semantic slots, bird names, species descriptors, perch phrases, time and weather phrases), written by a staff naturalist-writer. There are ≥400 templates at launch, and variation is picked with the aviary PRNG.
  - Format: lowercase, present tense or recent past, with a weekday prefix when it fits ("tuesday — …"). No "you", no exclamation marks, no numbers (except weekday names and small words like "twice").
  - **No LLM.** Sending per-bird state to a third-party model would breach the privacy commitment, and a hosted model risks generic phrasing. (D6.)
- **Hard content rule.** Entries observe the aviary, never the user. The observation extractor has **no access** to presence totals, visit counts, session counts, or absence duration. Those fields aren't in its input type, so the generator can't write "you visited every day this week" or "after a long absence…". The copy linter (§11.7) also blocks second person and visit or streak vocabulary.

### 5.9 Drift calibration targets and harness

**Reference user** ("regular visits"): 5 sessions per week, 8 min of presence each, occasional listen-in, 1–2 offers per session.

**Worked example** with the initial constants, for boldness starting at 0.35:
- One session-day gives `p_eff` = 20·(1 − e^{−0.4}) ≈ 6.6 min. A week is ≈33 effective min, which adds ≈0.033 into the reservoir. Headroom (0.65) takes that to ≈0.021 deliverable.
- **After 1 week:** a fraction 1 − (4/7)(1 − e^{−1.75}) ≈ 0.53 has drained, so ΔT ≈ **0.011**. That's above the **instrument threshold (0.005)**. ✓ measurable at ~1 week.
- **After 3 weeks:** drained ≈ 0.81 of ≈0.064, so ΔT ≈ **0.052**. That's at the **visible threshold (0.05)**. ✓ visible at ~3 weeks.
- **Heavy user** (60 min daily): `p_eff` ≈ 19 per day, but `cap_day` 0.006 means visible drift takes at least ~9 days. That's never session-scale.
- **One session:** ≤0.006, an order of magnitude below the visible threshold. ✓

**Defining "visible".** The perceptual threshold 0.05 is operationalized in behavior space rather than numbers:
- boldness +0.05 ≈ +8 percentage points of time on the front perch
- warmth +0.05 ≈ +10% relative first-greeter probability
- plumage +0.05 ≈ ΔE ≈ 3 in the plumage shader output

Design validates these with a side-by-side perception study, and constants are retuned so that "visible" matches ~3 weeks of reference use.

**Calibration harness** (`engine-sim` CLI, runs in CI and in `calibration`):
- Synthetic personas: reference, heavy daily, weekend-only, sporadic, two-week-absence, returns-after-month, background-tab-all-night (presence 0), multi-device-overlap, mute-only, captions-only.
- 90 days of simulated time at 1-minute ticks, run in seconds.
- CI assertions:
  - band tests: reference `ΔT(7d) ∈ [0.005, 0.02]`, `ΔT(21d) ∈ [0.04, 0.07]`; heavy user `ΔT(7d) ≤ 0.042`
  - background-tab persona has zero drift
  - no trait ever decreases
  - absence persona: traits unchanged or higher, attunement at floor, greeting propensity lower but never zero
  - captions-only equals audible
- Every engine config change prints a calibration report diff in the PR.

---

## 6. Sync model

### 6.1 Single canonical state

- Exactly one writer of simulation state exists: the tick holding the shard lease, guarded by the `tick_seq` CAS.
- Clients never hold authoritative state. Every device reads the same precomputed snapshot, so two devices can't disagree about traits, moods, perches, or weather, beyond the gap between their last pulls.
- There's no client-to-client sync and no merge logic.

### 6.2 Event log semantics

- **Ordering.** Server-assigned `seq` per aviary. The API allocates it with `UPDATE aviary_seq SET next = next + 1 RETURNING` in the same transaction as the insert. The tick consumes events strictly in `seq` order, up to the head read at tick start.
- **Idempotency.** `UNIQUE(session_id, client_seq)`. Retries after network failures are no-ops, and the client outbox keeps events until they're acked.
- **Additive deltas only.** No event carries an absolute value. Events are inputs to additive, non-negative reservoir increments, so order effects are negligible, and a late event just contributes on a later tick. Last-write-wins on personality is unreachable: nothing to "write" exists outside the tick (I1).
- **Clock trust.** `client_ts` is advisory. Presence segments are clamped:
  - `end ≤ received_at + 5 s`
  - `start ≥ max(previous acked end for this session, received_at − 10 min)`
  - segments that arrive more than 6 h late (offline outbox) are dropped for drift purposes
  - the union across sessions and devices goes into `presence_interval`, so two devices watching simultaneously count once
- **Late events for a past local day** credit into the current day's caps. This keeps the daily cap logic simple, and the difference is immaterial at these magnitudes.

### 6.3 Snapshot propagation

- The tick writes to Redis after commit. The edge reads the regional cache; the API reads Redis and falls back to the DB.
- **Pull triggers:**
  - initial load (inline in HTML)
  - `visibilitychange → visible`
  - render-loop frame gap >5 s (suspend or resume)
  - keepalive every 60 s while visible (ETag makes most of these `304`s)
  - right after an offer resolution (to pick up other birds' timeline)
  - `online` event
- **Interpolation.** The client keeps `snapshot[n]` and a playback clock (`server_now` offset estimated per pull, smoothed). Timeline entries play at their scheduled server times. When a new snapshot contradicts the rendered state (for example after a suspend, a bird now sits on a different perch with no timeline entry), the client schedules a natural flight within 0.5–3 s instead of teleporting. In reduced motion it's a cross-fade.
- **Staleness.** If the snapshot is >3 min old and pulls are failing, the client keeps extrapolating presentation (micro-motion, calls, lighting) indefinitely. After 2 min of failures it shows the matter-of-fact inline notice "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." as a small line in the top bar. This is a system surface, not an in-scene overlay.

### 6.4 Non-simulation, user-owned fields

Names and settings are user intent, not simulation state:
- **Names** use last-write-wins by server receive time, with an `If-Match` version. On `412` the client re-fetches and shows the current name. No conflict dialog is needed, since the user is renaming a single field.
- **Settings** are per-field last-write-wins, applied on the next snapshot pull or settings fetch on other devices.

### 6.5 Settle across devices

- The settled *lighting* is device-local and session-scoped. Settling on the phone doesn't dim the laptop. The *engine* effect (the small mood-quieting impulse) is canonical and reaches every device through the next snapshot.
- The `settle.commit` event is sent only after the 5 s undo window, so accidental settles never reach the server.
- After settle, presence heartbeats stop until the user re-engages. Re-engagement means a click or tap in the aviary, or keyboard interaction. A mere pointermove doesn't count, so nudging the mouse on the way to close the tab doesn't unsettle.

### 6.6 Failure scenarios (designed outcomes)

| Scenario | Outcome |
|---|---|
| Laptop and phone both open, both present | Presence unioned once. Both render the same state. Offers from either resolve against the same canonical state, and the second offer sees the first bird's cooldown. |
| Tick worker crashes mid-transaction | Rollback. The next lease holder re-runs from the last committed `tick_seq` and cursor. |
| Two workers briefly both think they hold a shard | CAS on `tick_seq`: one commit succeeds, the other is discarded. |
| DB failover | Synchronous standby, RPO ≈ 0. The tick resumes, and catch-up handles the gap. |
| Client offline for an hour | Presentation continues. The outbox holds events (capped at 500, oldest presence dropped first). On reconnect it flushes, then pulls a snapshot. |
| Replayed magic link | Already consumed → "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| Session revoked on another device | The next API call returns `401 session_revoked` → "Your session timed out. Sign in again to keep watching." |

---

## 7. Visits

- **Invite.** The host enters an email. A 256-bit token goes into the visitor's email. The GET→POST confirm pattern applies (as for sign-in) so scanners can't consume it.
- **Claiming** mints a visitor session bound to that browser (cookie). The link can't be claimed twice. A visitor on another device needs a new invite from the host (D10).
- **Lifetime.**
  - Unused invites expire 30 days after creation.
  - Used invites stay valid until revoked.
  - Visitor sessions lapse after 30 days without a visit; the visitor then needs a new invite.
  - There is no auto re-invitation.
- **What the visitor sees.** The host's aviary exactly as it is: same birds, moods, perches, weather, and **the host's local time of day**. The `visitor_snapshot` is the same projection minus the absence anchor, the greeting inputs, the adoption state, and all account data.
  - The visitor client runs in **render-only mode**: no presence FSM, no event outbox, no greeting, no listen-in, no offer, no settle, and no notebook.
  - Audio plays using the visitor's device settings. The visitor can mute locally, and that's never sent.
  - The visitor's local accessibility preferences (reduced motion, captions, narration) apply locally.
  - There's no marker, cursor, or avatar, and the host sees nothing new in the scene.
- **Server guarantees.** The visitor audience can't call event endpoints, and host drift never includes visitor time (test-enforced).
- **Revocation.** Immediate. `/v1/visit/snapshot` checks status on every pull, and visitor clients pull every 60 s and on visibility change. The visitor sees: "This visit is no longer available." (matter-of-fact). The host gets no confirmation beyond the log.
- **Visit log** (Settings → Visits, lazy chunk): the visitor email (decrypted server-side for the host only), the date, approximate duration ("about 20 minutes"), and outstanding invites with revoke controls. There's no badge, count, or highlight for new visits.
- **Notifications.** `notify_on_visit` is default off and never offered during onboarding. When enabled, the host gets at most one email per visitor per day: "Someone you invited visited your aviary." Matter-of-fact, no counts.
- **No aggregates.** We don't compute visits-per-aviary or any cross-account visit statistics, so no data exists that could later turn into leaderboards.

---

## 8. Frontend rendering pipeline

### 8.1 Stack and bundle layout

- **UI shell:** Preact (~4 KB) for the top bar, panels, settings, and notebook. **Scene:** a custom thin WebGL2 renderer with a Canvas2D fallback when WebGL2 is unavailable or the context is lost. No general-purpose game engine (bundle and control).
- **Build:** Vite/Rollup. `size-limit` enforced in CI. Chunks:

| Chunk | Contents | Budget (gz) |
|---|---|---|
| Inline boot (in HTML) | critical CSS, boot renderer (rig + pose sampler + Canvas2D path), snapshot JSON, species rig data for present birds | ≤ 60 KB |
| `main` | full renderer (WebGL2), behavior layer, presence FSM, outbox, snapshot client, top bar | ≤ 220 KB |
| `audio` | AudioWorklet synth, grammar, mixer, captions (loaded right after first frame) | ≤ 90 KB |
| `a11y` | narration generator + lexicon (loaded after first frame; immediately if screen-reader heuristics or a stored preference) | ≤ 60 KB |
| Lazy | notebook panel, offer menu, settings, account, visits, onboarding, export/deletion | ≤ 250 KB total |

Total initial JS (boot + main + audio + a11y) is ~430 KB, a quarter of the 2 MB ceiling. The headroom is deliberate: the PRD cap is a hard fail line, and our internal CI budget fails at 600 KB.

### 8.2 Bird rendering

- **Procedural 2D rigs.** Each species is parametric data (~2–4 KB): silhouette bezier outlines per body part (body, head, beak, wing, tail, legs, eye), palette, feather-detail pattern params, and a proportions range. Rigs are tessellated to triangle meshes at load and cached. There are no bitmaps for birds.
- **Plumage saturation** is a shader uniform per bird. It drives saturation and feather-detail contrast, quantized to 1/20 steps and interpolated over minutes so changes never pop.
- **Animation.** A skeletal pose blend tree plus secondary motion: spring-damper tail and feathers, breathing oscillation always on (amplitude and rate by mood), and blinks at stochastic intervals.
- **Responsive layout solver.** The scene uses normalized coordinates. The perch layout is computed from viewport aspect ratio: wider screens spread perches, while portrait phones stack zones more vertically (front lower-left to back upper-right) and scale birds down to a floor size.
  - Solver invariant: every bird's bounding box plus its flight corridors stays inside the safe area below the top bar, at every viewport from 320×480 to 3840×1600.
  - A property test renders all 7 birds in all slots across 200 random viewports and asserts no clipping.

### 8.3 Behavior layer (idle micro-motion)

- A per-bird utility selector picks micro-actions every 2–12 s: preen, scan, head-tilt, body-shuffle, fluff, look at another bird, look toward a sound, look toward a leaf or item, and hop within slot.
- Weights come from `mood.label`, `mood.intensity`, and `expr` (preen for content, scan for wary, tilt for curious, fluff low for drowsy). Each action has 2–5 procedural variants with randomized timing and amplitude, so nothing loops identically.
- Sound-directed head tilts listen to the audio engine's call events (bird X called from position P), which gives visible bird-to-bird attention.
- **Seeding.** Seeded per `(bird_id, page-load)` random stream. Two devices show different micro-motion but the same canonical state, which is correct.
- **Continuous from the first frame.** The boot renderer samples each bird's current micro-action phase from a deterministic function of wall clock and seed. Frame 1 therefore shows mid-preen or mid-scan, never a neutral T-pose. On handoff, the full renderer resumes from the exact same pose state (shared pose sampler). No entry animation exists anywhere.

### 8.4 Scene composition

Layers, back to front:
1. sky gradient: time-of-day lookup table, lit by a lighting uniform
2. far foliage: soft parallax factor 0.02
3. perches and back and middle birds
4. front perch and birds
5. offer items: seed, pool with a reflective shader
6. ambient ornaments: leaves and feathers, client-generated at a Poisson ~1 per 20–40 s, no state
7. occasional foreground branch sway
8. weather: rain streak particles capped at 150, wind shaping sway amplitudes

Parallax is driven by a very slow drift plus a tiny pointer influence (≤4 px), and is disabled under reduced motion.

### 8.5 Frame loop and budgets

- A single `requestAnimationFrame` loop. Per-frame main-thread work targets ≤6 ms on the reference laptop (a 2020–2021 mid-range i5 with integrated GPU). One draw pass, instanced where possible, with no per-frame allocations: preallocated typed arrays and object pools, lint-checked in hot paths.
- **Adaptive quality.** When p95 frame time exceeds 14 ms over 10 s, degrade in this order:
  1. ornament count
  2. rain particle count
  3. render scale (DPR capped at 2, then 1.5, then 1)
  4. parallax off

  Bird motion fidelity is never degraded first.
- **Hidden tab.** rAF stops naturally. The audio engine fades out over 1.5 s and suspends the `AudioContext` to save battery. That's intentional and not "presence." On visible, the client pulls a snapshot, resumes rendering from wall-clock-sampled poses, and fades audio in with the chorus mid-phrase.
- **Frame gap >5 s** (suspend or resume) triggers a snapshot pull and a greeting evaluation (§9.1).

### 8.6 Loading states and first frame

- **HTML is streamed from the edge.** The head flushes immediately with critical CSS and the boot script, and the snapshot `<script type="application/json">` follows as soon as the edge has it from the regional cache (≤60 ms target).
- **If the snapshot isn't available within 250 ms** (cold cache, slow link), the boot renderer draws the **quiet field**: the time-of-day sky, faint foliage, one slow leaf. No spinner, no text. When the snapshot arrives, birds are placed at their current perches in their current actions, with no fade-in and no "wake-up".
  - The one allowed exception is onboarding's empty-to-first-bird fly-in.
- **Warm start.** The client keeps the last snapshot in IndexedDB. If it's under 10 min old and the network snapshot is late, render from it immediately, then reconcile with natural transitions. Older cached snapshots aren't used; a stale mood could visibly "snap".
- **Unsupported browser** (no ES2020, no Canvas, no `Intl` timezone): a static, matter-of-fact page, "Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge." WebAudio isn't required (§9.5).

### 8.7 Top bar

- **Items, left to right:** field notebook, offer, settle, accessibility settings, account/settings. All are icons with accessible names, and each opens a small panel or popover. Settle's inclusion is decision D1.
- **Fade.** After 3 s without pointer movement or keyboard activity, the bar fades to 8% opacity over 1.2 s. It returns to 100% over 200 ms on pointermove, touch, or key. It never fades while it has keyboard focus or an open panel, or when "keep top bar visible" is set.
- No badges, dots, or counts on any icon, ever.

### 8.8 Reduced-motion mode (a designed surface)

**Activation:** `prefers-reduced-motion: reduce`, overridable in settings (`system` / `on` / `off`). It switches the renderer to the **pose cross-fade presenter**:
- **Idle.** Each micro-action is authored as 2–4 key poses. The presenter holds each pose for 3–8 s and cross-fades over 1.5–2.5 s, so a preen becomes a slow sequence of preen poses. Breathing and blinks are dropped.
- **Flight becomes a cross-fade.** The bird fades out at perch A over 1.2 s while fading in at perch B.
- **Offer reactions** become a cross-fade to the approach pose near the item. The pool is a still reflective surface.
- **Scene.** Leaf and feather drift, parallax, and foreground sway are removed. Rain becomes a still, softly-textured veil with a darkened sky, cross-faded in and out, and the rain sound still plays.
- **Lighting** keeps the day and evening shifts at 2× the normal duration. The settle shift runs over ~8 s.
- **Unchanged:** calls, captions, drift, mood, notebook, greetings. The greeting becomes a cross-fade to a head-up or front-facing pose plus the call.
- **Design deliverable:** a pose library per species, authored and reviewed as its own aesthetic, with sign-off by design and by vestibular-sensitive testers.

---

## 9. Interactions: implementation specifics

### 9.1 Return-greeting

**Triggers:** a fresh navigation, `visibilitychange → visible`, or a frame gap >5 s, but only when the absence exceeds 30 s. Absence is `now − max(snapshot.absence.last_presence_end_at, local last-presence-end)`, so it's account-wide across devices. Returning after a quick tab switch doesn't re-greet.

**Absence buckets and greeting shapes:**

| Absence | Shape palette |
|---|---|
| 30 s – 10 min | a glance up from the current action (one bird) |
| 10 min – 4 h | head-tilt toward the viewer, or a quiet 1–2 note call |
| 4 h – 2 days | step or hop toward the front of its zone plus a call, and sometimes a second bird responds |
| > 2 days | re-orientation: the greeter moves to the front perch (if not wary) with a longer call phrase, and a second bird answers after 2–6 s |

**Greeter selection:** sample without replacement weighted by `expr.greet`. Roosting birds are excluded unless the species is the nightjar-like one. At night, a roosting bird may "open one eye," a minimal head turn, so someone always notices. Exactly one bird greets first, within 0.6–1.8 s of the first frame (randomized). Additional greeters, only in the longer buckets, stagger by a random 1.5–6 s. There's never a unison chorus.

**Procedural variation:** shape → pose sequence parameters (angles, timings) → a call phrase generated by the grammar with a greeting-flavor bias. A property test generates 10,000 greetings per bird-mood-bucket and asserts no two identical (pose params plus phrase) and a minimum parameter distance between consecutive greetings for the same bird.

**Consistency:** the same bird greets in a stable *style* across visits because style comes from its `expr` and voice print, while the details vary.

Reports `greeting.observed`. Audio may be locked at first view (§9.5), so a greeting call scheduled before unlock is shown only visually, and captions if enabled.

**No textual welcome of any kind.** A test asserts no DOM text changes in the first 10 s of a session except the narration live region (and captions if enabled).

### 9.2 Listen-in

- **Engage:** click or tap a bird (hit-testing via the rig's convex hull), or Enter on a focused bird proxy.
- **Disengage:** click the focused bird again, focus or click another bird (which switches), click empty scene space, move keyboard focus away (Tab out of the scene or into the top bar), or press Escape.
- **Mix ramps:** see §10.3. There's no visual chrome. A subtle, personality-appropriate response is allowed: the bird may glance toward the viewer once if its mood is receptive. The focus outline shows **only** for keyboard focus (`:focus-visible`).
- **Events:** `listen_in.start` and `listen_in.end`. The server computes duration by its own clock and caps it at 10 min per bird per day for drift.

### 9.3 Offer

The top-bar offer icon, or the shortcut `O`, opens a small popover (keyboard navigable, a `menu` role) with **seed**, **song fragment ▸** (a submenu of 8 named fragments: "a falling three-note figure", …), and **still pool**.
- On choosing, the item is placed at a front-scene location chosen deterministically from the scene layout. The popover closes, and the resolver flow runs (§5.5).
- Song fragments are **synthesized** motifs played on a neutral "offer voice" in the same synth. No samples.
- Only one active seed and one pool may exist at a time. A repeat seed offer while one is active adds a few more seeds to the same patch.

### 9.4 Settle

1. Choose the settle icon, or press `S` when the top bar has focus. Lighting eases toward the evening palette over 4 s (~8 s in reduced motion). The master bus eases −6 dB, the call-rate multiplier drops to 0.3, and birds bias toward drowsy poses locally.
2. **Undo window of 5 s:** any click or tap in the aviary reverses it over 1.5 s. No undo text is shown. (For keyboard users, Escape also undoes within 5 s; see §11.2.)
3. **After 5 s:** send `settle.commit` and `presence.end{reason: settle}`, and stop presence heartbeats.
4. The settled state lasts until the tab closes or the user re-engages (click, tap, or key in the aviary). Re-engagement eases lighting back to the time-of-day palette over 3 s and resumes the presence FSM.
5. If it's already evening or night, settle deepens toward night.

There's no "you didn't settle" surface anywhere, and tab close is equivalent at the engine level.

### 9.5 Audio unlock and the "calls already audible" goal

Browsers block `AudioContext` output until a user activation. Plan:
- **At boot:** create the context and attempt `resume()`. Chrome's media-engagement heuristics may allow it for returning users.
- **If suspended:** the call scheduler **runs silently anyway**, advancing its phrase state. On the first activation (`pointerdown`, `keydown`, or `touchend` anywhere), resume the context and fade the master in over 800 ms, joining the chorus mid-phrase. It never starts a phrase from zero.
- **No "tap to enable sound" prompt.** That would announce. The first listen-in click or any tap unlocks audio naturally. (Risk R7.)
- **iOS:** set `navigator.audioSession.type = 'playback'` where supported, so the hardware silent switch doesn't mute calls the user expects to hear. Honour the in-app mute setting.
- **WebAudio unavailable** (no API, context creation throws, or the worklet fails to load after 2 retries): graceful silence, and **captions default on** for that device (a local preference, reversible). There's no recorded-audio path anywhere in the codebase; a lint rule rejects `<audio>` and `decodeAudioData` imports.

### 9.6 Presence FSM (client)

```
signals: visible (document.visibilityState), focused (document.hasFocus(), window focus/blur),
         lastActivity (pointermove | pointerdown | keydown; passive, throttled 250 ms)
present  = visible ∧ focused ∧ (now − lastActivity ≤ W) ∧ ¬settled
W        = 240 s initial (calibrate 180–360 s; "lean long")
evaluate = 1 Hz while visible; on every signal change immediately
```

- While present, accumulate the segment and emit `presence.segment` every 15 s.
- On any transition to not-present, emit `presence.end{reason}` and close the segment.
  - For the idle reason, the segment ends at `lastActivity + W`. That's the literal PRD definition: activity within the window counts.
- `pagehide` flushes via keepalive. `keydown` replaces the deprecated `keypress`.
- Touch devices get pointer events from touches. A touch-only watcher who doesn't touch for W loses presence, which is a known calibration risk (R2). Coarse-pointer devices may use a longer `W` (initial 300 s) under the same conjunction rule. We never add new signals such as scroll or device motion, because that would weaken the definition.
- **Visitors:** the FSM isn't instantiated.

### 9.7 Field notebook panel

- Opened from the top-bar icon or shortcut `N`. It's a paper-textured side sheet (bottom sheet on narrow viewports) over the aviary. The aviary keeps running and audible behind it, and presence continues.
- Entries are fetched 20 at a time with virtualized rendering. Off-screen entries are recycled, so no references are retained (memory rule).
- Entries are read-only text: no edit, delete, annotate, share, like, or count. The empty state for a brand-new aviary is a single naturalist line: "the notebook is new. the first pages are still blank."
- Adoption "welcome it" controls appear inline only on the relevant visiting-bird entry.

---

## 10. Audio pipeline

### 10.1 Synthesis engine

- **One `AudioWorkletNode`** hosts the synthesizer, with a fixed voice pool of 16 voices (7 birds × 2 overlapping phrases + offer voice + alarm headroom). All buffers are preallocated in the worklet at construction. Scheduling is by messages: `{voice, bird, phrase:[syllables], startTime}`. There are no per-call `OscillatorNode`s, so node churn and GC are zero, which satisfies "no per-call allocation that isn't freed."
- **Syllable model.** Each syllable has a pitch contour (start, peak, end in semitones relative to the voice's base), duration, amplitude envelope (ADSR with curvature), a timbre vector (FM ratio and index, noise mix, band-pass formant center and Q), and optional vibrato or trill (rate and depth).
- **Voice-print** (fixed at mint, never drifts): base pitch, timbre vector, preferred motif subset (4–6 motifs from the species library), rhythmic signature (inter-syllable timing template), and a small "accent" (for example a characteristic final down-slur). This is what makes pip sound like pip in every mood.

### 10.2 Call grammar runtime (client)

- **Species motif library:** 10–14 motifs, each a short syllable sequence with parameter ranges.
- **Phrase generation:** a stochastic grammar: `Phrase → Intro? Motif (Motif | Repeat | Variation){0..n} Coda?`
  - Length range depends on mood: short for wary, longer for content.
  - Each syllable parameter is perturbed within ±jitter (pitch ±0.4 st, timing ±12%, amplitude ±2 dB). Jitter is bounded so the signature stays recognizable.
  - Mood shifts the tempo (±15%), pitch span (±2 st), loudness (±4 dB), and phrase length. Traits shift **rate** (`call_rate`) and **span** (`pitch_span`), never the signature.
- **Scheduling:** per-bird Poisson process with rate `expr.call_rate × moodMult × timeOfDayMult × weatherMult × settleMult`.
- **Call-and-response:** after bird X calls, each other bird *y* has a response window of 0.8–3 s with probability `0.15 + 0.5·expr_y.greet_social` (warmth-driven). Some birds answer, and all of them look (§8.3).
- **Chorus:** when ≥2 birds with `call_rate ≥ 0.6` fall within a shared 20 s window, the scheduler opens a chorus: overlapping phrases with loose rhythmic entrainment (±80 ms toward a shared pulse), ≤25 s, at most once per ~10 min.
- **Alarm calls:** played at the server `timeline` time, a sharp single-syllable motif.
- **Night:** the nightjar-like species keeps calling. Everyone else is silent or very rare.

### 10.3 Mixing and listen-in

- **Graph.** Worklet outputs per-bird stems (a 7-channel output, or the worklet applies per-bird gain and pan internally with parameters exposed as `AudioParam`s) → per-bird equal-power pan by x-position → per-bird distance filter (back perch: −3 dB and a gentle low-pass at 6 kHz) → shared reverb send (convolution with an **impulse response generated procedurally at startup**, no download) → ambient bed (procedural filtered noise for air and leaves, −30 dBFS) → master gain → `DynamicsCompressor` limiter.
- **Listen-in engage.** The focused bird gets +5 dB over a ~2 s time constant (`setTargetAtTime`, τ = 0.6 s). The others go to −11 dB relative, with a 2.5 s τ = 0.8 s, and pick up a gentle low-pass (to 4 kHz) for a "leaning in" feel. The **floor is −18 dB, never muted**. The focused bird's call rate isn't changed; listening changes the mix, not behaviour.
- **Disengage** uses the same time constants in reverse, back to the ambient mix.
- **Weather** ducks the birds by 2 dB during rain, with procedural rain noise.

### 10.4 Captions (from the grammar)

- The caption generator consumes the **actual phrase object** just scheduled and maps its features to prose:
  - syllable count → "single", "two-note", "three-note", "a run of"
  - contour → rise, fall, level, "rising then falling"
  - trill flag → "trill"
  - pauses or repeats → "paused, … again"
  - loudness → soft, clear, sharp
  - perch → "from the back perch"
  - mood colour words, used sparingly
- Examples: "a soft three-note rise", "a low trill, paused, low trill again". Because the caption is derived from the same object as the sound, it always matches what played (test: caption features are asserted against the phrase features).
- **Display:** small text near the calling bird, anchored above its head inside the safe area. It fades in with the call onset (300 ms), holds for call length + 1.2 s, then fades out.
  - At most 3 captions show at once, prioritized listen-in bird > greeting > alarm > others. The rest are skipped, not queued.
  - Reduced motion uses opacity fades only.
  - The caption box has a translucent backing so the text passes AA against every sky state (§11.5).

### 10.5 Performance and robustness

- Worklet CPU target is <5% of one core on the reference laptop at 7 birds in chorus. A worklet underrun counter is reported as aggregate telemetry.
- `AudioContext` state machine: `running`, `suspended` (hidden, or awaiting activation), and `interrupted` (iOS). It recovers automatically, and repeated failures fall back to silence plus captions.
- **Offline test rendering.** `OfflineAudioContext` renders thousands of phrases in CI for:
  - glitch detection (clicks and DC offset)
  - loudness range
  - recognizability metrics (§10.6)

### 10.6 Recognizability (the load-bearing audio property)

- **Metric.** An embedding of each rendered phrase (log-mel statistics plus pitch-contour features). For every pair of birds in a 7-bird aviary, **inter-bird distance** must exceed **intra-bird distance across all moods and trait extremes** by margin *M*. This is enforced:
  - at voice-print minting, via rejection sampling against existing birds in the aviary. Two birds of the same species must still separate.
  - in CI across randomly generated 7-bird aviaries.
- **Human validation.** Listening panels identify birds in 2-, 3-, 5-, and 7-bird aviaries after short familiarization. Pass criterion: ≥80% correct identification at 7 birds. The per-tier results gate the `max_birds` ramp (§14.2).

---

## 11. Accessibility

### 11.1 Screen-reader narration

- **Live region.** A visually hidden container, `role="log"` with `aria-live="polite"` and `aria-relevant="additions"`, sits outside the canvas. Old narration nodes are pruned to the last 5, so memory stays bounded. Polite only, never `assertive`.
- **The generator** (client-side, `a11y` chunk) reads the same snapshot plus behavior-layer state the renderer uses. Its steps:
  1. **Salience:** score candidate observations (a bird's perch, current action, calls in the last window, weather, light, chorus, items). Prefer changes since the last narration and anything the user is focused on.
  2. **Select 1–3 observations.**
  3. **Compose prose** from the shared naturalist grammar: lowercase, present tense, bird names or species descriptors ("pip", "a small grey bird").
  4. **Avoid repetition:** novelty memory of the last 20 sentence templates, and never the same template twice in a row.
- **Cadence.**
  - Idle: one update every 30–60 s (randomized).
  - Priority events narrate promptly (≤1 s), jumping the idle timer but still polite: the return-greeting, offer reactions, settle and its undo, a listen-in start ("pip's calls come closer."), and weather starting.
  - Minimum spacing between updates is 8 s. Overflow is merged into the next update, not queued.
- **Example output:** "pip is on the front rail, preening. wren sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
- **Forbidden:** "Pip is at perch 2", "mood: content", any numbers, any trait words used as stats. The linter checks every generated string (§11.7).
- **Visible narration.** The "show narration as text" setting renders the same stream in a small, contrast-safe line under the top bar for users who want it.

### 11.2 Bird focus proxies and keyboard model

- Each bird has an invisible `<button>` proxy positioned over its on-canvas bounds. It moves by `transform` only when the bird changes perch, and is positioned absolutely within an `aria-label`led `role="group"` "the aviary".
  - **Accessible name:** the bird's name plus a short live description regenerated on focus ("pip, on the front perch, preening").
  - **Accessible description:** "Press Enter to listen in."
- **Keyboard order and keys:**

| Key | Action |
|---|---|
| Tab | Top-bar items in order, then into the scene (focuses the frontmost-left bird, with roving tabindex), then out |
| ←/→ | Move between birds in visual x-order |
| ↑/↓ | Move between depth zones, nearest in x |
| Enter / Space | Listen-in on the focused bird. Pressing again disengages. |
| Escape | Exit listen-in. Undo settle within 5 s. Close any panel and return focus to its opener. |
| O / N / S | Offer menu, notebook, settle. Active only when focus isn't in a text field. Documented in accessibility settings. |

- **Focus indicator:** a 3 px soft-white ring with a 2 px dark halo (dual ring). It's ≥3:1 against every aviary lighting state, verified by the automated check in §11.6. It follows the bird smoothly during flights, or cross-fades in reduced motion.
- Top-bar fade never hides a focused control.
- **Panels:** the notebook, offer, and settings panels trap focus while open and return focus on close. Every panel has a visible close control.

### 11.3 Reduced motion

See §8.8. Also: CSS transitions in panels use opacity only when reduced motion is active.

### 11.4 Captions

See §10.4. They're opt-in from accessibility settings, and on by default when WebAudio is unavailable.

### 11.5 Contrast and text

- All user-copy text passes WCAG AA (4.5:1 body, 3:1 large and UI glyphs). This covers the top bar, panels, settings, errors, captions, and visible narration.
- Text rendered over the scene (captions, visible narration) uses a translucent backing whose opacity adapts to the sampled lighting, so AA holds at dawn, noon, dusk, and night.
- Top-bar icons meet 3:1 at full opacity. The faded state is decorative, and any interaction restores full opacity before a user can act.

### 11.6 Automated accessibility checks (CI)

- axe-core on every panel and page.
- A contrast sweep renders the scene at 12 times of day and 3 weather states and samples the backgrounds behind captions, focus rings, and the top bar.
- Keyboard e2e (Playwright) covers the full journey: sign-in, onboarding naming, listen-in, offer, settle and undo, notebook, settings, invites.
- Live-region tests capture narration over 10 simulated minutes. They assert cadence bounds and that the linter passes.
- A reduced-motion visual regression asserts no transforms on birds except cross-fade opacity, and no particles.

### 11.7 Voice linter (shared by notebook, narration, captions, onboarding copy)

**Naturalist surfaces must:**
- be lowercase
- contain no `!`
- contain no second person ("you", "your")
- contain no digits
- avoid a banned-vocabulary list: welcome, achievement, unlock, level, streak, badge, points, score, visited, days in a row, happy/happier as a stat, mood:, boldness, curiosity (as a stat word)
- avoid announcement grammar (no "X perched at Y" without an article or verb; a template schema requires a subject + verb + prepositional phrase)

**System surfaces** must use sentence case and must not use naturalist-only vocabulary markers.

Every template is fuzzed with 1,000 renders per template in CI.

### 11.8 Manual test matrix (pre-launch and each release)

- Screen readers: VoiceOver on macOS Safari, VoiceOver on iOS Safari, NVDA with Firefox and Chrome, JAWS with Chrome, TalkBack with Chrome.
- 200% zoom and 400% reflow for panels.
- Windows High Contrast / forced colors: panels and top bar must stay usable, and the scene is left to render as art.
- Participants include blind, low-vision, deaf/HoH, and vestibular-sensitive testers, compensated, with qualitative feedback on whether the aviary "feels alive" in their modality. This is the pass criterion, alongside conformance.

---

## 12. Performance budgets and observability

### 12.1 Budgets (all CI-enforced or synthetic-monitored)

| Budget | Target | Enforcement |
|---|---|---|
| Initial JS (gz) | PRD cap 2 MB; internal fail at 600 KB; boot inline ≤60 KB | `size-limit` in CI |
| Time to first bird visible | <500 ms from navigation start, mid-tier Android (Moto G Power class) on a 4G profile (≈9 Mbps / 170 ms RTT) | Lighthouse-CI custom metric plus WebPageTest synthetic from 5 regions, p75 gate |
| TTFB (edge) | ≤150 ms p75 on the synthetic profile | synthetic |
| Snapshot availability at edge | ≤60 ms p95 from the regional cache | server metric |
| Idle frame rate | 60 fps: p95 frame ≤16.7 ms, <1% dropped over 30 min on the reference laptop | nightly device lab run on real hardware (2020 i5 MacBook Air / Intel UHD laptop) |
| Memory | no growth over 30 min | CI soak test (below) |
| Tick latency | p50 <5 ms/aviary; **alarm at p99 >5 s** end to end | server metrics and alerting |
| Offer resolution | ≤150 ms p95 server time | server metrics |

**First-bird metric definition:** `performance.mark('first-bird')` in the rAF callback after the first frame containing ≥1 fully drawn bird has been committed (double-rAF). It's measured from `navigationStart`.

**Critical path for 500 ms:**
1. Edge TLS (session resumption, HTTP/3).
2. Stream the head and inline boot in the first ~14 KB congestion window.
3. Snapshot JSON inline.
4. The boot renderer uses Canvas2D (no WebGL shader compile on the critical path) to draw the first frame.
5. WebGL2 renderer takes over at the next idle, sharing pose state (seamless).

Fonts are system fonts in the scene, and the top-bar icons are inline SVG.

**Memory soak test (CI, nightly plus pre-release).** Playwright + Chrome DevTools Protocol runs a scripted 30-min session: idle, listen-ins, offers, notebook scroll over 500 entries, settle/undo, visibility toggles, and a 7-bird aviary in chorus with rain. It collects:
- JS heap after forced GC at 5, 15, and 30 min: the slope must be ≤0.05 MB/min, with total growth ≤2 MB
- DOM node count: constant ±50
- audio voice pool: fixed
- GPU texture count: constant

A failure blocks release.

### 12.2 Observability: what we measure

All of it is aggregate, and none of it carries per-bird state or per-account interaction history.

- **Client RUM**, via a separate cookie-less endpoint (`credentials: 'omit'`). There are no account, session, or bird IDs; each page view gets a random ID that isn't persisted. It collects:
  - navigation timings, first-bird timing, snapshot-arrival timing
  - frame-time histograms (per minute, bucketed)
  - long-task counts
  - AudioContext start/resume failures, worklet underruns, autoplay-locked duration buckets
  - WebGL context-loss counts
  - JS error counts plus stack signatures (with message scrubbing)
  - session-duration histogram buckets sent at pagehide, with no identifiers
- **Server metrics** (OTel → metrics store): request rates, latencies, and errors by route; tick latency and lag; CAS conflicts; catch-up steps; event ingestion rate and cursor lag; Redis hit rates; email send and bounce rates; magic-link failure reasons (expired, consumed, invalid) as counts; export and deletion job health; **invariant-violation counters** (monotonic breach attempts, bounds, bird-count, identity), which page immediately.
- **Synthetic monitoring:** a fleet of automated browsers loads a synthetic test account every 5 min from 5 regions, asserting first-bird timing, audio worklet boot (headless with fake audio), and error-free console.
- **Logs:** structured, with a field **allowlist** per log type. `account_id` and `aviary_id` UUIDs are allowed for debugging. Event payloads, names, notebook text, emails, and tokens are never allowed. A PII scanner drops and alerts on email-shaped strings.

### 12.3 What we deliberately don't measure

- Per-account engagement, DAU/MAU, retention cohorts, session counts per user, visit frequency.
- Offer, listen-in, settle, or notebook-open rates. No funnels.
- Drift or trait distributions across the population, and mood distributions. We don't calibrate on production users at all (§5.9 uses the harness plus the consented `calibration` environment).
- Visits per aviary, or any cross-account ranking stat.
- A/B tests on affective or engagement outcomes.
- Third-party analytics, ad pixels, session replay, or heatmaps.

**Architectural enforcement.** The analytics/metrics plane has no network route or credentials to the simulation database. The simulation database has no export job to the warehouse. The `calibration` environment is physically separate. A quarterly privacy review audits these boundaries.

---

## 13. Accounts, auth, privacy: implementation notes

- **Magic links:**
  - 256-bit token, SHA-256 hash stored, 15 min expiry, single use via atomic `UPDATE … WHERE consumed_at IS NULL AND expires_at > now() RETURNING`.
  - The link signs in the browser that opens it.
  - Emails are plain, matter-of-fact, one link, and carry no aviary content.
  - The provider is a transactional email subprocessor, named in the privacy policy. Only email addresses and system messages go to it, never interaction data.
- **Sessions:**
  - The cookie holds an opaque token, plus a short-lived signed edge token (5 min) for the edge snapshot inline. Revocation is written to Redis and the edge KV and takes effect within seconds.
  - Settings → Sessions lists the device label, created date, and last seen date, with a revoke control.
- **Email change:** verify the new address (15 min link), then swap the ciphertext and blind index atomically. The old address keeps working until then and gets a notice.
- **Export:**
  - A job assembles the JSON: account settings, birds (id, name, species, adopted date, current personality vector, current mood label), notebook entries, and visit log.
  - The file is encrypted at rest and expires after 72 h. The download needs the link token and a signed-in session.
  - Including trait values honours the PRD's export requirement. They appear only in a downloaded data file, never on an in-product surface (D5).
- **Deletion:**
  - Soft delete sets `deletion_requested_at`. Every signed-in page then shows a matter-of-fact strip under the top bar: "This account is scheduled for deletion on 14 November. [I changed my mind]".
  - Invites are suspended (visitors see "no longer available"). The aviary keeps ticking so a restored aviary has continued living. Restoring clears the marker.
  - At day 30 the hard-delete job removes every row for the `account_id` across shards (birds, traits, checkpoints, events, presence, notebook, observations, invites, visitor sessions, exports, Redis keys) and **destroys the account DEK**, which crypto-shreds encrypted fields in backups. Backups age out at 35 days.
  - A deletion-verification job confirms zero rows remain and emits only a count metric.
- **Privacy policy:** a plain-text link in account settings naming the aggregate telemetry categories and explicitly excluding per-bird interaction state.

### 13.1 Copy registry (matter-of-fact system strings)

Keys include:
- `signin_link_expired`: "We couldn't sign you in. The link may have expired. Try requesting a new link."
- `session_timed_out`: "Your session timed out. Sign in again to keep watching."
- `aviary_load_failed`: "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."
- `visit_unavailable`: "This visit is no longer available."
- `unsupported_browser`, `export_requested`, `deletion_scheduled`, `email_change_pending`, `rate_limited`

All system copy lives in one registry reviewed by the content lead. Naturalist copy lives in the grammar packages. The voice linter checks both against their respective register rules.

---

## 14. Rollout

### 14.1 Build sequence (≈7 months to public v1; teams in parallel)

**Workstreams:**
- Engine: 2 engineers
- Platform/API/auth/SRE: 3
- Client render and behavior: 3
- Audio: 1 engineer + 1 sound designer
- Accessibility: 1 engineer + an a11y consultant
- Content: 1 naturalist-writer
- Design: 1 visual + motion designer
- QA/perf: 1

| Phase | Weeks | Exit criteria |
|---|---|---|
| M0 Foundations | 1–4 | Engine step with property tests. Calibration harness running personas. Postgres schema with roles and triggers. Auth (magic link and sessions). Boot renderer drawing a static-pose bird from inline JSON under 500 ms on the synthetic profile. |
| M1 Living aviary | 5–10 | Tick service with leases, CAS, and catch-up. Snapshot projection and edge inlining. Full renderer with rigs for 6 species, the behavior layer, lighting, and weather. AudioWorklet synth with grammar and voice prints, plus the recognizability CI metric. Presence FSM and event outbox. |
| M2 Interactions | 11–16 | Return-greeting, listen-in mix, offer resolver and reactions, settle with undo, notebook extraction and grammar (≥400 templates), onboarding, naming, adoption flow, captions, narration, reduced-motion presenter with the pose library, keyboard model. **Staff dogfood begins in `calibration` (week 12) to accumulate real 3-week drift experience.** |
| M3 Hardening | 17–22 | Visits (invite, claim, visitor view, revoke, log), export, deletion, email change, memory soak green, 60 fps device lab green, a11y manual matrix pass #1, privacy boundary audit, chaos tests (worker kill, DB failover, duplicate/replayed events). Calibration retune from the dogfood qualitative review plus the harness. |
| M4 Closed beta | 23–28 | Invite-only beta (~2,000 accounts, waitlist). A11y matrix pass #2 with disabled testers. Listening panel at 2/3/5/7 birds. SLOs held for 4 weeks. |
| M5 Public v1 | 29+ | Open sign-up. Staged traffic ramp behind edge rate limits (10% → 50% → 100% of waitlist over 2 weeks, then open). |

The engine, calibration harness, and privacy boundaries come first because they're the hardest to retrofit. Dogfood starts at M2 because drift can only be felt after about three real weeks.

### 14.2 Ramping birds per aviary

- The engine supports 7 from day one and CI tests 7-bird aviaries for rendering, audio, and perf.
- The **`max_birds` config** (global, server-side) gates adoptable birds:
  - **Launch:** 2, plus 3 enabled once the 3-bird listening panel passes. Aviary-age thresholds mean no production aviary reaches 3 birds before ~90 days after launch anyway.
  - **Raised stepwise to 4, 5, 6, 7** as each tier passes the human recognizability panel (≥80% ID) and the 60 fps and memory runs at that count on reference hardware.
- If a tier fails, eligible aviaries simply don't see a visiting bird yet. Nothing is announced, and nothing is taken away.
- Existing birds are never removed if the cap is lowered for a regression. The cap only gates new adoptions.
- The `calibration` environment uses **time-compressed aviaries** (aviary age artificially advanced) so staff can live with 5- to 7-bird aviaries months before production users reach them.

### 14.3 Feature flags and kill switches

- Flags: `visits_enabled`, `adoption_visitors_enabled`, `weather_enabled`, `chorus_enabled`, `captions_generator_version`, `narration_generator_version`, and `engine_config_version` (versioned constants with forward-only application).
- Kill switches must be **presentation-safe**. For example, disabling weather ends the current rain gracefully. No kill switch may touch traits.

### 14.4 Engine versioning and migrations

- **Engine versions apply forward only.** Stored traits are never recomputed from history, and a new drift function starts from the current values.
- Migrations touching `bird`, `bird_traits`, or `aviary_state` require a dry run on a production snapshot copy (in an isolated environment with the same privacy controls), plus identity and trait-equality verification: every `bird_id` and trait value is preserved bit-for-bit unless the migration's explicit purpose is additive.
- Species definitions are append-only versioned. Retired species keep their definitions forever for existing birds.

### 14.5 Day-one instrumentation (must exist before beta)

All metrics in §12.2, with alerts:
- tick p99 >5 s
- tick lag >2 min
- invariant violations >0 (page)
- first-bird p75 synthetic >500 ms
- snapshot error rate >0.5%
- magic-link send failure >2%
- worklet failure rate spike
- deletion job failure
- PII scanner hit (page)

Also: a daily backup-restore drill that verifies a sampled restored `bird_traits` row equals its checkpoint.

---

## 15. Decision log (ambiguities resolved)

| # | Ambiguity | Decision | Rationale |
|---|---|---|---|
| D1 | `aviary_layout` lists four top-bar icons ("nothing else"). `interactions` and `accessibility_perf` both put settle in the top bar. | Settle is a fifth, visually subdued top-bar icon. | Three statements require settle to be reachable from the top bar. The sparse spirit is preserved. Flagged for design review; the alternative is folding settle into the offer popover. |
| D2 | Mood "resets daily-ish" vs "mood never resets / never snaps". | No reset operation. The overnight roost attractor washes out session impulses continuously. | Both requirements are met, and there's never a discontinuity. |
| D3 | Drift never decreases, yet returning after absence shows "quieter" birds that greet less. | A hidden, bounded `attunement` scalar (floor 0.25, recovers fast) that scales expression only, never ease or traits. | Only way to satisfy both statements. It's not a Tamagotchi meter: no failure state, never shown, no wariness. |
| D4 | The brief says muting shapes drift. The engine's input list omits it. | Audible *or captioned* presence adds a small vocal-frequency bonus. Mute without captions forgoes it. Nothing decreases. | Honours the brief without penalizing deaf/HoH users. |
| D5 | Traits are "never exposed numerically", but the export includes "current personality vectors". | Export includes them (PRD-mandated data portability) in a downloaded file only. No in-product surface ever renders them. Snapshots carry only quantized expression parameters. | Reconciles portability with the product-surface rule. |
| D6 | Notebook prose generation method is unspecified. | Hand-authored generative grammar, no LLM. | Privacy (no third-party processing of per-bird data), voice control, determinism, testability. |
| D7 | "Calls already audible" at first frame vs browser autoplay policy. | Silent-running scheduler, and audio fades in mid-phrase on first activation. No enable-sound prompt. | Platform constraint. A prompt would announce. |
| D8 | Day/night by local time, but no location. | Fixed civil day curve by IANA timezone. No seasonal or latitudinal sunrise. | Avoids collecting location. Revisit post-v1. |
| D9 | Offers go "to the aviary" from the top bar, not to a bird. | The listen-in bird is primary recipient if focused. Otherwise the engine picks by curiosity, mood, and proximity. | Keeps "not by clicking a bird" while giving users a natural way to direct attention. |
| D10 | Visit link "one-time", but visitors may want to return. | The one-time link mints a browser-bound visitor session valid until revoked or 30 days idle. Other devices need a new invite. | Honors one-time links, keeps visits deliberate, and avoids a self-updating visitor list. |
| D11 | Visitor time of day. | The host's local time. | "exactly what the host would see at this moment." |
| D12 | Settle across devices. | Lighting is device-local. The mood-quieting effect is canonical. | Settle is a per-session goodbye. The engine effect is aviary-wide. |
| D13 | Tick cadence for long-dormant accounts. | Full 60 s cadence in v1. A dormancy tier only after an equivalence proof. | The PRD says the tick runs regardless. Cost is modest at v1 scale. |
| D14 | Hidden-tab audio. | Fade out and suspend while hidden. | Battery, timer throttling that makes calls erratic, and the fact that hidden means not present. |
| D15 | How new birds are offered without announcing. | A visiting bird appears, the notebook notes it, and "welcome it" lives in that entry and in Settings → Birds. There are no reminders. | Notices rather than announces, adds no scene chrome, and keeps pacing age-based. |
| D16 | Presence on touch devices (no hover pointermove). | Same conjunction, and touch pointer events count. Coarse-pointer `W` starts at 300 s. No new signals. | Keeps the definition exact. The window length is the PRD's calibration knob. |
| D17 | Same species twice (7 birds, ~6 species). | Allowed. Voice-print rejection sampling guarantees separation. | Required by the arithmetic. Recognizability is protected by the metric. |
| D18 | Localization. | English only for v1. The grammar architecture supports future locales with native writers. | Naturalist voice doesn't machine-translate. |

---

## 16. Risks and mitigations

| # | Risk | Likelihood / impact | Mitigation |
|---|---|---|---|
| R1 | **Drift miscalibration:** too fast (Tamagotchi-like) or too slow (screensaver). We can't observe production drift, by privacy design. | High / High | Two-stage filter with daily cap and headroom. Persona harness with CI band tests. Consented dogfood in the isolated `calibration` env starting at M2. A perception study defining "visible" in behavior space. Constants in versioned config, applied forward only. Any retune is re-validated against all personas. |
| R2 | **Presence misaccounting:** inflation (background tabs, multiple devices, spoofed pings) or deflation (touch watchers, browser focus quirks such as Safari `hasFocus` in split view or iframes). | Medium / High | Strict conjunction FSM with a 1 Hz check plus events. Server union of intervals and clamping. Daily soft saturation and caps. Cross-browser Playwright matrix toggling each signal. Background-all-night persona must produce zero drift. The coarse-pointer window is tunable. Spoofing only affects the spoofer's own aviary and is bounded by caps. |
| R3 | **Sync correctness:** lost drift from races, double-applied events, a tick worker split-brain. | Low / Critical | Single writer with lease plus CAS. Idempotent event keys. Additive deltas. DB monotonic trigger. Invariant checks that fail closed. Daily checkpoints plus PITR. Chaos tests (kill workers mid-commit, duplicate and reorder events, failover) in CI and staging. |
| R4 | **Personality loss** (worst failure): a migration, a bad deploy, or a restore error. | Low / Critical | Role-restricted writes. The trigger forbids decreases. Migration linter forbids bird deletes and re-creates. Dry-run-and-verify on production copies. Daily checkpoints with restore drills. Deleting a bird has no code path at all outside the hard-delete job. |
| R5 | **Audio uncanniness:** synthesized calls sound chiptune-like or robotic, or the chorus sounds mechanical. | High / High | A dedicated sound designer from M0. Physically inspired syllable model (FM, noise, formant, jitter, reverb). Listening panels every milestone. A "sounds alive" qualitative bar reviewed by product. Bounded jitter tuned against recognizability. |
| R6 | **Recognizability collapse at higher bird counts.** | Medium / High | Embedding-distance metric in CI plus rejection sampling at mint. Human panels gate each `max_birds` tier. The cap is only raised when evidence supports it. |
| R7 | **Autoplay and iOS audio behaviour** make the first seconds silent or mute calls unexpectedly. | High / Medium | Silent-running scheduler with mid-phrase fade-in. `audioSession` playback type. RUM counters for autoplay-locked duration (aggregate). Captions available. Accepted as a platform constraint (D7). |
| R8 | **Accessibility regressions:** narration drifts toward state-list style, the reduced-motion presenter bit-rots, focus rings lose contrast in new lighting. | Medium / High | Shared voice linter over all generated prose. Reduced-motion visual regression tests. Contrast sweep across times of day. A manual screen-reader matrix every release. Disabled testers in beta. Accessibility is a release-blocking gate, not a follow-up. |
| R9 | **"Harmless" announcement or gamification creep** (a toast, a count, a streak-ish notebook line). | High over time / High | Banned dependencies and components. Copy linter. The notebook input type excludes user-behavior data. Design-review checklist tied to §0 I7. Product lead holds veto. |
| R10 | **Time-to-first-bird misses on real 4G** (TLS, cold edge cache). | Medium / High | Inline boot, early flush, Canvas2D first frame, warm-start cache (≤10 min), regional snapshot cache, synthetic monitoring from 5 regions, quiet field (never a spinner) when missed. |
| R11 | **Memory growth or jank over long sessions** (worklet buffers, DOM proxies, notebook scroll, texture churn). | Medium / Medium | Preallocated pools. Fixed voice pool. Virtualized notebook. CI soak with slope assertions. Device-lab 30-min runs. |
| R12 | **PII leakage** (email in logs, error trackers, partition keys). | Medium / High | Blind index and encryption confined to the auth module. Field allowlists. Scrubbers. PII scanner paging. CI grep of staging telemetry. Synthetic UUIDs everywhere. |
| R13 | **Magic-link deliverability and link scanners** consume tokens. | Medium / Medium | GET→POST confirmation. Reputable ESP with SPF/DKIM/DMARC. Bounce metrics. Clear matter-of-fact retry copy. |
| R14 | **Tick cost growth** with dormant accounts. | Medium / Low | Cheap step (~ms). Logical shards spread across clusters. Dormancy tier possible later behind the equivalence proof (D13). |
| R15 | **Timezone flapping** from travelling users with multiple devices. | Low / Low | Two-hour consistency rule and a 20 min glide. |
| R16 | **Greeting feels canned** despite procedural variation. | Medium / High | Parameterized shapes per bucket, per-bird style from `expr`, a 10k-sample uniqueness property test, and qualitative dogfood review of first-session greetings specifically. |

---

## 17. Test strategy summary

- **Engine:**
  - property tests: monotonicity, bounds, determinism, variable-`dt` equivalence, identity stability
  - golden replay files per engine version
  - persona calibration bands
- **Sync:**
  - concurrency tests with two simulated devices
  - duplicate, reordered, and late events
  - CAS races, worker kill, and failover chaos in staging
- **Client:**
  - presence FSM unit and e2e tests across browsers
  - layout solver property test
  - first-frame "motion in progress" test: frame 1 poses are non-neutral and there's no opacity ramp on birds
  - no-welcome-text test
  - greeting uniqueness test
- **Audio:**
  - offline rendering glitch and loudness checks
  - recognizability metric
  - caption-to-phrase correspondence
  - no-recorded-audio lint
- **Accessibility:** §11.6 automated, plus the §11.8 manual matrix.
- **Performance:** size-limit, Lighthouse-CI first-bird, device-lab 60 fps, 30-min memory soak.
- **Privacy:**
  - log schema allowlist tests
  - PII scans
  - a network-policy test that the analytics plane can't reach the simulation DB
  - a visitor-audience test that event endpoints reject visitors
  - a deletion-completeness test
- **Voice:** linter fuzzing over every generator (notebook, narration, captions, onboarding).
