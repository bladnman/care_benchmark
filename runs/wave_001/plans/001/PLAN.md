# Pocket Aviary — v1 Implementation Plan

Status: execution plan for the v1 build, written for the engineering team that will build it.
Source: the Pocket Aviary PRD (`product_brief`, `concepts`, `bird_engine`, `interactions`, `aviary_layout`, `accounts_sync`, `social_optional`, `accessibility_perf`, `non_goals`).

This plan turns the PRD into specific decisions: services, schemas, algorithms, numeric starting values, test gates, and a rollout. Where the PRD is ambiguous or contradicts itself, §17 records the call made and the reason. All numeric constants below are **starting values**. They live in versioned engine-parameter files and are calibrated through the harnesses described here. Engineers should not treat them as final. They should treat them as the defaults the calibration work starts from.

---

## 0. The ten invariants

Every component in this plan serves these invariants. Each one has an automated check (named in brackets). A pull request that breaks one does not merge.

1. **Only the server tick writes personality.** No client code path and no API endpoint accepts a personality value. Personality changes only through additive deltas computed by the tick from the event log. [`engine.invariants.single-writer`, API schema test, DB role grants]
2. **Personality is monotonic toward expressive.** For every bird, trait, and tick: `x_new ≥ x_old`. Absence never produces negative drift, and no mood input grows with absence. [property tests; DB trigger; tick pre-commit assertion]
3. **Bird identity is permanent.** A `bird_id` is never reassigned, regenerated, or replaced. No migration may rewrite a personality vector except through the tick. [migration lint; restore runbook]
4. **Presence is the three-signal conjunction.** A presence credit requires `visibilityState === 'visible'` ∧ `document.hasFocus()` ∧ recent pointer/key activity, all at once. The server clamps credit to server wall-clock time. Visitors never generate presence. [presence harness; server clamp tests]
5. **Personality numbers never reach a product surface.** They appear on no UI, in no debug overlay in production builds, in no ARIA attribute, and in no snapshot payload. The one exception is the account export file; see D-3. [snapshot schema test; build-time strip of debug modules]
6. **Notice, never announce.** The product has no toasts, banners, welcome text, badges, streaks, visit counts, push, Notification API, or confetti. [lint rules; copy lint; banned-component list]
7. **Calls are procedural.** The product ships no recorded call or ambient audio in any code path, including the fallback. [bundle scan for audio MIME/asset types; CSP `media-src 'none'`]
8. **Email lives in exactly one place.** Only the accounts record holds email, and it is encrypted there. Every other reference uses the synthetic account UUID. [log/telemetry schema allowlists; PII scanner in CI and on log sinks]
9. **Per-bird and per-account interaction data never enter aggregate pipelines.** The analytics and telemetry stack holds no credentials to the simulation database. [IAM policy tests; network policy]
10. **The first frame is the aviary, already in motion.** The product has no spinner, entry animation, or fade-from-static. [visual regression on first-frame capture; banned-component list]

---

## 1. Scope

### 1.1 In v1

| Area | Included |
|---|---|
| Accounts | Magic-link sign-in (15-min, single-use); per-device sessions, listed and revocable; email change with verification; JSON export by emailed link; soft delete (30 days) then hard delete; synthetic account UUIDs |
| Aviary | One aviary per account; 2 starter birds; age-based arrivals up to a cap of 7; species pool of 6 (one nocturnal); user-assigned names, renameable any time |
| Engine | Server-side tick at ~60 s; personality vector (5 traits) with monotonic low-pass drift; mood system; perch selection; call planning; bird-to-bird call/response and chorus; weather; circadian cycle in the account's local time |
| Interactions | Return-greeting; listen-in; offer (seed / song fragment / still pool) with per-bird cooldown; settle with a 5 s undo; presence accounting; field notebook (read-only, sparse) |
| Scene | One horizontal scene with no panning; three perch zones; day/night; ambient weather; parallax and leaf/feather drift; responsive layout; top bar that fades |
| Audio | Client-side procedural synthesis (AudioWorklet); per-bird voiceprints; chorus mixing; listen-in mix ramps; procedural ambient bed; silence plus captions when audio is unavailable |
| Social | Per-invite, email-addressed, read-only visits; revocable; 30-day invite expiry; visit log; optional visit-notification email, off by default |
| Accessibility | Naturalist screen-reader narration; reduced-motion mode designed as its own surface; call captions generated from the actual synthesized call; full keyboard model; WCAG AA contrast |
| Perf/ops | Initial bundle < 2 MB gz (internal target 400 KB); first bird visible < 500 ms on mid-tier mobile over 4G; 60 fps idle on a 5-year-old laptop; zero memory growth over 30 min (CI-enforced); aggregate-only RUM plus synthetic checks; tick p99 alarm at 5 s |
| Browsers | Last two major versions of Chrome, Safari, Firefox, Edge; a matter-of-fact unsupported-browser page for anything else |

### 1.2 Explicitly out of v1 (and structurally prevented where possible)

- Native apps. The protocols are designed for the web only. No native-client accommodations.
- All gamification. No achievements, streaks, levels, scores, badges, counters, visit calendars, or "days visited" in any surface, setting, or export (see D-4).
- All Tamagotchi mechanics. No hunger, death, distress, or decaying happiness. The engine has no state that can express distress caused by neglect.
- Social-network surfaces: profiles, follows, feeds, discovery, comments, chat, avatars, leaderboards, co-presence, "show-off" rendering. The metrics that would drive leaderboards are never computed.
- Push notifications, web push subscriptions, and re-engagement email. The only product email is the opt-in visit notification. Transactional email covers magic links, email change, export links, deletion confirmation, and visit invitations.
- Payments, shared or team aviaries, multiple aviaries, customizable scenes, user-placed birds, a species catalog, SSO, passwords.
- Recorded audio of any kind.
- Engagement analytics: DAU/retention cohorts per account, funnels, A/B tests on engagement (see §13.4).

---

## 2. Architecture

### 2.1 Service shape

```
                 ┌─────────────────────────── Browser ───────────────────────────┐
                 │ Boot shell (inline) → Scene (Canvas2D) → Animation → Audio     │
                 │ (AudioWorklet) → Voice (captions/narration) → Presence tracker │
                 │ → Event outbox (IndexedDB) → Snapshot client + clock sync      │
                 │ Service Worker: shell + static asset cache (no push)           │
                 └──────────────┬───────────────────────────────▲─────────────────┘
                     events,    │                               │ HTML+inline snapshot,
                     heartbeats │                               │ snapshots (ETag)
                 ┌──────────────▼───────────────────────────────┴─────────────────┐
                 │ Edge (CDN + edge worker): static assets, HTML stream,          │
                 │ snapshot fetch from regional snapshot cache, geo routing       │
                 └──────────────┬───────────────────────────────▲─────────────────┘
                 ┌──────────────▼───────────┐          ┌────────┴──────────────┐
                 │ API service (stateless)  │          │ Snapshot cache (Redis,│
                 │ auth, account, events,   │─────────▶│ primary + regional    │
                 │ snapshot read, notebook, │          │ read replicas)        │
                 │ birds, visits            │          └────────▲──────────────┘
                 └──────┬───────────────────┘                   │ publish per tick
                        │ append events                         │
                 ┌──────▼──────────────────────────────────────┴──────────────┐
                 │ Postgres (simulation DB): accounts, aviaries, birds,        │
                 │ personality, mood, events, observations, notebook, visits   │
                 └──────▲───────────────▲──────────────────▲──────────────────┘
                        │               │                  │
          ┌─────────────┴───┐  ┌────────┴────────┐  ┌──────┴──────────────────┐
          │ Tick workers    │  │ Naturalist      │  │ Jobs worker: email,     │
          │ (sharded leases)│  │ worker (notebook│  │ export, deletion, invite│
          │                 │  │ generation)     │  │ expiry                  │
          └─────────────────┘  └─────────────────┘  └─────────────────────────┘

   Separate, one-way, no credentials to the simulation DB:
   Metrics/RUM collector → aggregate TSDB/dashboards; log pipeline (PII-scrubbed, UUID-only)
```

**Stack decision.** TypeScript across the stack (Node 22 LTS for services). The main reason is a shared, pure `engine-core` package. The server tick, the client's reactive behaviors (greeting, offer reaction, idle scheduling), the drift-calibration lab, and the test harnesses all import it. Engine logic therefore exists in exactly one implementation. The tick is cheap: under 1 ms CPU per aviary with 7 birds. Node covers the throughput needed (§5.8).

**Packages (monorepo):**

| Package | Runs on | Contents |
|---|---|---|
| `engine-core` | server + client + lab | Pure functions: tick step, drift, mood dynamics, perch choice, call planner, greeting planner, offer resolver, weather scheduler, seeded PRNG (PCG32). No I/O, no `Date.now()`; time is injected. |
| `voice` | client + naturalist worker | Naturalist grammar engine, lexicons, caption/narration/notebook generators, copy lint |
| `species` | client + server | Six species definitions: silhouette parameters, palettes, motif libraries, circadian profile |
| `protocol` | client + server | Zod schemas for snapshot, events, API DTOs; version constants |
| `client` | browser | Boot shell, scene, animation, audio, a11y, panels |
| `api`, `tick`, `naturalist`, `jobs` | server | Services |
| `drift-lab` | CI / offline | Persona simulator, calibration reports |

### 2.2 Client/server split

| Concern | Server (authoritative) | Client |
|---|---|---|
| Personality vector | Stores it and alone writes it (tick) | Never sees raw values. Receives quantized render parameters derived from them |
| Mood, perch, behavior mode, weather, call plan | Computed per tick; the canonical snapshot carries a 180 s forward "script" | Plays the script; interpolates; reconciles on a new snapshot |
| Idle micro-motion (breathing, preening frames, blinks, head saccades) | Not modeled | Generated procedurally from behavior mode + mood + seeded noise |
| Reactive moments (greeting, offer reaction, listen-in acknowledgment) | Receives events; credits drift; logs observations | Runs `engine-core` planners locally for zero-latency response, seeded by server nonces |
| Presence | Clamps, dedupes across devices, credits | Detects the three-signal conjunction; sends heartbeats |
| Day/night palette | Supplies the account's canonical timezone | Computes lighting from the canonical timezone and current time |
| Leaves, feathers, parallax | Not modeled (PRD: pure ornament) | Generated locally |
| Notebook | Generates and stores entries | Reads (paginated) |
| Narration, captions | — | Generated from rendered state and the synthesized call parameters |
| Settle | Receives `settle` after the undo window; applies a mood nudge | Owns the per-device settled lighting state (D-7) |

### 2.3 Render pipeline boundary

The boundary is the **snapshot plus script**. The server says what is true now and what is planned for the next 180 s: which perch, which behavior mode, which mood, which calls at which server times. The client decides how it looks and sounds: bone angles, feather fluff, oscillator frequencies, caption text. The client never extrapolates canonical state past the script horizon. If the horizon runs out without a new snapshot, birds keep idling in their last behavior mode on their last perch. That is always a valid, living state.

---

## 3. Data model

Postgres 16 is the simulation database. All primary keys are UUIDv7 unless noted. `account_id` is the partition/shard key everywhere and is never derived from email.

### 3.1 Accounts and auth

```sql
accounts (
  account_id            uuid PK,
  email_ciphertext      bytea NOT NULL,          -- envelope-encrypted (KMS data key)
  email_blind_index     bytea UNIQUE NOT NULL,   -- HMAC-SHA256(pepper_k, normalized_email); auth lookup ONLY
  status                text CHECK (status IN ('active','pending_deletion')),
  deletion_requested_at timestamptz,
  created_at            timestamptz NOT NULL,
  settings              jsonb NOT NULL,          -- a11y/audio/visit-notification prefs (§11.7)
  timezone              text NOT NULL,           -- IANA; canonical aviary tz (D-6)
  timezone_changed_at   timestamptz
)
email_change_requests (request_id, account_id, new_email_ciphertext, new_email_blind_index,
                       token_hash, expires_at, consumed_at)
magic_links (token_hash bytea PK, email_blind_index, purpose ('sign_in'|'email_change'),
             account_id NULL, expires_at, consumed_at, created_at)
sessions (session_id, account_id, token_hash, device_label, created_at, last_seen_at, revoked_at)
security_audit (account_id, kind, at, device_label)   -- sign-ins, revocations; 90-day retention
```

The blind index exists only so sign-in can find the account. It is never logged, never used as a foreign key, and never leaves the auth module. A code-owner rule on the `auth/` directory enforces this.

### 3.2 Aviary and birds

```sql
aviaries (
  aviary_id        uuid PK,
  account_id       uuid UNIQUE REFERENCES accounts,
  created_at       timestamptz NOT NULL,       -- "aviary age" drives arrivals
  state_version    bigint NOT NULL,            -- increments per committed tick
  tick_index       bigint NOT NULL,
  last_tick_at     timestamptz NOT NULL,
  event_cursor     bigint NOT NULL,            -- last consumed per-account event seq
  shard            smallint NOT NULL,          -- hash(aviary_id) mod 1024
  rng_seed         bigint NOT NULL,
  engine_params_version int NOT NULL,
  weather          jsonb NOT NULL,             -- current + scheduled events
  presence_state   jsonb NOT NULL,             -- per-device last heartbeat, merged-interval tail, last_presence_end_at
  arrival_state    jsonb NOT NULL,             -- next_arrival_at, pending newcomer
  script           jsonb NOT NULL              -- last published forward script (for snapshot rebuild)
)

birds (
  bird_id       uuid PK,                        -- permanent identity
  aviary_id     uuid REFERENCES aviaries,
  species_id    text NOT NULL,
  name          text NOT NULL,                  -- user data; never in logs
  status        text CHECK (status IN ('resident','newcomer')),
  adopted_at    timestamptz,
  voiceprint    jsonb NOT NULL,                 -- immutable after creation (trigger-enforced)
  appearance    jsonb NOT NULL                  -- immutable per-bird markings seed
)

bird_personality (
  bird_id        uuid PK REFERENCES birds,
  boldness       double precision NOT NULL,
  warmth         double precision NOT NULL,
  vocal_freq     double precision NOT NULL,
  plumage_sat    double precision NOT NULL,
  curiosity      double precision NOT NULL,
  ceilings       jsonb NOT NULL,                -- per-trait asymptote, fixed at adoption
  filter_state   jsonb NOT NULL,                -- per-channel low-pass accumulators
  day_budget     jsonb NOT NULL,                -- per-trait delta used in current local day
  version        bigint NOT NULL,
  updated_at     timestamptz NOT NULL
)
-- trigger: every trait NEW >= OLD, else RAISE; only the role `tick_writer` has UPDATE.

bird_state (                                    -- fast-timescale, rewritten by tick
  bird_id, mood text, arousal real, valence real, mood_entered_at, mood_min_dwell_until,
  attention_envelope real,                      -- recency-of-attention, NOT a trait (§5.4)
  perch_zone text, perch_slot smallint, behavior_mode text, behavior_since,
  offer_cooldown_until timestamptz, last_called_at
)

bird_personality_daily (bird_id, local_date, vector jsonb)  -- restore points; 35-day retention
```

**Species pool (6):** four diurnal passerine-like species (a small grey warbler type, a wren type, a finch type, a tit type), one dove-like species with low calls, and one nightjar-like nocturnal species. Rarity is not a feature. Starter selection picks two distinct diurnal species by seeded weighted draw, with pairings chosen for timbral contrast. Arrivals draw from species not yet present first. After all six are present, duplicates are allowed; each bird still has a distinct voiceprint.

### 3.3 Events and observations

```sql
event_counters (account_id PK, next_seq bigint)         -- per-account ordering, row-locked
interaction_events (
  account_id uuid, seq bigint,                          -- PK (account_id, seq)
  client_event_id uuid UNIQUE,                          -- idempotency
  device_session_id uuid, type text, bird_id uuid NULL,
  payload jsonb, client_ts timestamptz, received_at timestamptz
) PARTITION BY HASH (account_id);
-- retention: deleted 30 days after consumption (the tick cursor has passed them)

observations (                                          -- notebook substrate; 60-day rolling
  account_id, observation_id, kind, bird_ids uuid[], facts jsonb,
  observed_at, local_date, salience real, used_in_entry uuid NULL
)
notebook_entries (entry_id, aviary_id, local_date, written_at, text,
                  grammar_version, template_ids text[], bird_ids uuid[])   -- permanent until account hard-delete
```

**Event ordering.** Global `bigserial` ids can commit out of order. A tick could read seq 105 before seq 104 commits and skip 104 forever. So each insert takes the next per-account sequence with `UPDATE event_counters … RETURNING` inside the same transaction. The row lock serializes that account's inserts, so seq order equals commit order. Per-account write rates are low (heartbeats every 20 s plus rare interactions), so contention is negligible.

**Event types** (the `protocol` package defines the payload schemas):

| Type | Payload | Engine use |
|---|---|---|
| `session_start` | `{device_session_id, kind: 'navigate'|'visible_return'}` | observation only |
| `presence_heartbeat` | `{hb_seq, present_ms_since_last, monotonic_ms}` | presence credit |
| `presence_end` | `{hb_seq, present_ms_since_last, reason: 'hidden'|'blur'|'inactive'|'settle'|'pagehide'}` | closes window |
| `listen_in_start` / `listen_in_end` | `{bird_id, duration_ms}` (end) | warmth/vocal drift; mood |
| `offer` | `{kind: 'seed'|'fragment'|'pool', fragment_id?, near_bird_id?, offer_nonce, client_resolution}` | curiosity/boldness drift; mood; cooldown |
| `settle` | `{}` (sent after the 5 s undo window) | mood quieting; presence end |
| `greeting_shown` | `{order: bird_id[], forms: string[]}` | observation (notebook: "pip greeted before wren") |
| `audio_state` | `{state: 'unlocked'|'muted'|'unmuted'|'unavailable'}` | "heard" drift channel |
| `newcomer_welcome` / `newcomer_not_now` | `{bird_id, name?}` | arrival state |

### 3.4 Visits

```sql
visit_invitations (invite_id, host_account_id, visitor_email_ciphertext, visitor_email_blind_index,
                   token_hash, created_at, expires_at /* +30d */, redeemed_at, revoked_at)
visitor_sessions  (visitor_session_id, invite_id, token_hash, created_at, last_seen_at, idle_expires_at, ended_at)
visit_log         (visit_id, invite_id, host_account_id, started_at, last_seen_at)  -- duration = last_seen - started
```

The host's visit log shows visitor emails. This is the one place a second email is stored. It is encrypted with the same envelope scheme and belongs to the host account (deleted with it).

### 3.5 Exports

`exports (export_id, account_id, requested_at, status, object_key, expires_at)`. The object store bucket has a 7-day lifecycle.

---

## 4. API surface

JSON over HTTPS, versioned under `/v1`. Session auth uses an opaque token in an `HttpOnly; Secure; SameSite=Lax` cookie. Every mutating request must carry `X-Aviary-Client: 1` (CSRF defense on top of SameSite). All error bodies are `{code, message}`, where `message` is matter-of-fact English (§12.2).

### 4.1 Auth and account

| Method & path | Notes |
|---|---|
| `POST /v1/auth/magic-link {email}` | Always returns 202 (no account enumeration). Rate limits: 5 per 15 min and 20 per day per blind index; 30 per hour per IP /24. The email contains `https://…/auth/callback#t=<token>`. |
| `GET /auth/callback` | Static page. JavaScript reads the token from the **fragment** and POSTs it. Link-scanning mail gateways that GET the URL never consume the token, and the token never appears in server or CDN access logs. |
| `POST /v1/auth/consume {token}` | Atomic `UPDATE magic_links SET consumed_at=now() WHERE token_hash=$1 AND consumed_at IS NULL AND expires_at>now()`. Zero rows → `link_invalid`. Creates the account and aviary on first sign-in. Issues a session. Returns `{new_account: bool}`. |
| `GET /v1/sessions`, `DELETE /v1/sessions/:id` | Device list ("Safari on macOS · last active today") and revocation. Revocation takes effect on the next request (session cache TTL ≤ 30 s). |
| `POST /v1/account/email-change {new_email}` → `POST /v1/account/email-change/confirm {token}` | The old email keeps working until the new one verifies. A security notice goes to the old address after the switch. |
| `GET/PATCH /v1/account/settings` | a11y/audio prefs, visit-notification toggle |
| `POST /v1/account/export` | 202; the job emails a link to `/account/exports/:id`, which requires a signed-in session to download. No bearer URLs. |
| `POST /v1/account/delete`, `POST /v1/account/restore` | Soft delete / "I changed my mind" |

### 4.2 Aviary

| Method & path | Notes |
|---|---|
| `GET /v1/aviary/snapshot` | `ETag: "v<state_version>"`; returns 304 when the version is unchanged. Served from the regional snapshot cache; falls back to a DB rebuild. |
| `POST /v1/aviary/events {events:[…]}` | Batch of ≤ 50; idempotent per `client_event_id`; returns `{accepted, latest_version}`. Rejects: unknown `bird_id`, `client_ts` more than 10 min old, durations longer than wall time, visitor credentials. |
| `POST /v1/aviary/presence` | Lightweight heartbeat endpoint that also accepts `navigator.sendBeacon` (text/plain JSON) for `presence_end` on `pagehide`. |
| `GET /v1/notebook?before=<entry_id>&limit=20` | Reverse-chronological; unbounded history. |
| `PATCH /v1/birds/:bird_id {name}` | 1–24 chars, trimmed, Unicode-normalized. Changes only `name`. |
| `POST /v1/birds/:bird_id/welcome {name}` / `…/not-now` | Newcomer adoption (§5.10) |

**Snapshot payload** (target ≤ 6 KB gz at 7 birds):

```jsonc
{
  "v": 184213, "tick": 184213, "server_time": "2026-…Z", "tick_ms": 60000,
  "tz": "America/Chicago", "tz_blend": null,             // or {from, to, started_at} while easing (D-6)
  "weather": {"kind": "rain", "intensity": 0.4, "start": "…", "end": "…"},
  "greeting_ctx": {"last_presence_end_at": "…"},          // host only
  "offer_nonce": "b64…",                                  // host only; seeds offer resolution
  "birds": [{
    "id": "…", "name": "pip", "species": "grey_warbler",
    "voice": {…},                                         // immutable voiceprint
    "look":  {"sat_q": 37, "markings_seed": 9912},        // quantized render param (0..63), not a trait
    "expr":  {"call_rate_q": 21, "approach_q": 30, "tilt_q": 18, "greet_q": 25}, // derived, quantized
    "mood": "content", "zone": "front", "slot": 1,
    "behavior": {"mode": "preening", "since": "…"},
    "script": [{"at": "…", "do": "fly", "to": {"zone": "middle", "slot": 0}}, …],
    "calls":  [{"at": "…", "motif": 3, "seed": 771, "gain_q": 40, "reply_to": null}, …],
    "cooldown_until": "…"                                 // host only
  }],
  "newcomer": null
}
```

`expr` values are computed server-side from traits combined with mood and attention envelope, then quantized to 64 steps. No field is a raw trait. The client needs these values to render and schedule. They are not a product surface, and no UI or debug view in the production build reads them out.

### 4.3 Visits

| Method & path | Notes |
|---|---|
| `POST /v1/visits/invitations {email}` | Host only; ≤ 10 outstanding invites; per-host rate limit of 20/day. Sends a one-time link `…/visit#t=<token>`. |
| `GET /v1/visits/invitations`, `DELETE /v1/visits/invitations/:id` | List and revoke. Revocation also ends any visitor sessions tied to the invite. |
| `GET /v1/visits/log` | Visitor email, date, approximate duration, most recent first |
| `POST /v1/visit/redeem {token}` | Single-use (same atomic pattern as magic links). Sets a `visitor` cookie scoped to `/v1/visit/` and `/visit`. Visitor session: 30-day idle expiry, ended by revocation (D-9). |
| `GET /v1/visit/snapshot` | Read-only projection. Strips `greeting_ctx`, `offer_nonce`, `cooldown_until`, notebook, newcomer adoption controls. Returns `410 {code:"visit_unavailable"}` once the invite is revoked, expired, or suspended (host pending deletion). |
| `POST /v1/visit/heartbeat` | Updates `visit_log.last_seen_at` only. Structurally separate from presence: a different table, never read by the tick. |

Visitor credentials cannot reach any `/v1/aviary/*` endpoint (separate cookie, separate middleware). This is tested.

---

## 5. Simulation engine

### 5.1 Tick loop

- **Cadence:** every aviary is evaluated every 60 s ± 2 s. Each aviary's phase within the minute is `hash(aviary_id) mod 60000 ms`, so load spreads evenly.
- **Sharding:** 1024 logical shards. Tick worker pods hold time-bounded leases on shards (a lease table in Postgres with fencing tokens). A pod runs a timer wheel over its shards' aviaries.
- **Tick transaction** (per aviary):
  1. `BEGIN`; load the aviary row (`state_version = expected`), birds, personality, state.
  2. Read `interaction_events WHERE account_id=$1 AND seq > event_cursor ORDER BY seq LIMIT 2000`.
  3. `next = engine.tick(prev, events, now, rng(aviary.rng_seed, tick_index))`. The step is pure.
  4. Run invariant assertions (monotonic traits, bird count ≤ 7, bird ids unchanged, per-day drift budget not exceeded, finite numbers). On failure: roll back, increment the aggregate `tick_invariant_violation` counter, alarm, and leave state untouched. The next tick retries. Persistent failure quarantines the aviary to "hold": the last good snapshot keeps being served, and birds keep idling client-side.
  5. Write personality (only if changed), bird_state, aviary (`state_version+1`, `event_cursor = max(seq)`, `last_tick_at`, script); insert observations.
  6. `UPDATE aviaries … WHERE state_version = expected` (optimistic fence; a stale worker loses). `COMMIT`.
  7. After commit, publish the snapshot to the snapshot cache (key `snap:<account_id>`, value includes the version). If the publish fails, the snapshot endpoint rebuilds from the DB.
- **Exactly-once consumption:** the cursor advance and the state write commit atomically. A crash anywhere before commit replays the same events against the same prior state. That is safe because the step is pure and deterministic given `(prev, events, now, rng)`.
- **Catch-up after outages:** `dt = now − last_tick_at`. If `dt > 90 s`, the engine sub-steps: 60 s steps up to 6 h, 5 min steps beyond that, capped at 14 days of simulated time per tick call. Every integrator uses exponential or closed-form updates, so results are stable regardless of step size. Drift budgets are per local day, so a catch-up cannot exceed what real ticks would have produced.
- **Dormant aviaries still tick.** The PRD requires it. The cost is kept low by skipping personality writes when deltas are below 1e-9 and by batching `bird_state` writes for aviaries without a connected client: the in-memory worker state is flushed every 5 ticks, and a lease handoff forces a flush. Mood timers are stored as absolute timestamps, so a delayed flush loses nothing.

### 5.2 Personality vector

Five traits in `[0,1]`: `boldness`, `warmth`, `vocal_freq`, `plumage_sat`, `curiosity`.

- **Seeds at adoption:** species baseline in `[0.20, 0.40]` plus per-bird jitter of ±0.05. Seeds sit low so each bird has room to drift.
- **Ceilings:** a per-bird, per-trait asymptote in `[0.78, 0.95]`, drawn at adoption. Ceilings are the main defense against every long-lived bird converging to an identical maxed-out profile (risk R-1.3).
- **Voiceprint and markings** are separate and immutable. Drift never touches identity features.

### 5.3 Drift function

Drift is a two-stage low-pass filter with saturating inputs, headroom scaling, a daily budget, and a floor at zero.

**Stage 1: per-bird input channels.** Each tick converts consumed events into per-bird raw input `u_c` for a tick of length `dt`:

| Channel `c` | Raw input | Applies to |
|---|---|---|
| `presence` | credited presence seconds this tick (account-level, deduped across devices) | all resident birds |
| `heard` | credited presence seconds while `audio_state = unlocked ∧ ¬muted` | all resident birds |
| `listen` | listen-in seconds on this bird (capped to credited presence overlap) | that bird |
| `offer_accept` | 1 per accepted offer (server-resolved, outside cooldown) | the accepting bird |
| `offer_near` | 1 per offer placed near this bird (listen-in target or front-zone occupant) | that bird |

`settle` has no drift channel. Per the PRD it only ends the presence window and nudges mood.

**Stage 2: low-pass filter.** For each channel, the accumulator follows an EMA with time constant `τ_f = 3 days`:

```
f_c ← f_c · e^(−dt/τ_f) + u_c · (1 − e^(−dt/τ_f)) · (τ_f / dt_norm)
```

`dt_norm` normalizes `f_c` to "seconds of input per day" for presence-type channels and "events per day" for offer channels. A 12-minute session therefore raises `f_presence` a little, and the effect decays over days. Drift continues gently after the user leaves ("drifts during absence based on inputs from before they left") and then stops. The filter never goes negative, and nothing drives traits downward.

**Stage 3: trait rate.**

```
dx_i/dt = r_i · Σ_c W[i][c] · g_c(f_c) · h_i(x_i)
g_c(f) = f / (f + f_half_c)                     # saturating: binges give diminishing returns
h_i(x) = max(0, (ceil_i − x) / (ceil_i − seed_i))   # headroom: slows toward asymptote
Δx_i  = clamp(dx_i/dt · dt, 0, day_budget_remaining_i)
```

**Weight matrix `W` (starting values, rows = traits):**

| | presence | heard | listen | offer_accept | offer_near |
|---|---|---|---|---|---|
| boldness | 0.30 | 0 | 0.10 | 0 | 0.60 |
| warmth | 0.30 | 0 | 0.70 | 0.10 | 0.10 |
| vocal_freq | 0.20 | 0.30 | 0.60 | 0 | 0 |
| plumage_sat | 0.70 | 0 | 0.20 | 0 | 0 |
| curiosity | 0.25 | 0 | 0.10 | 0.80 | 0.10 |

Rows are not normalized on purpose. Plumage mostly tracks sustained presence ("drifts up with sustained attention"). Listen-in mostly moves warmth and vocal frequency. Offers mostly move curiosity (accepted) and boldness (placed near the bird).

**Daily budget:** `Δ_day_max = 0.005` per trait per local day. With a visibility JND of about 0.08 (§5.3.1), no single session can move a trait visibly, and even a user who watches for hours every day needs at least 16 days to reach visible change. The budget is the hard guarantee. The filter and `g` shape typical behavior.

#### 5.3.1 Calibration targets and harness (`drift-lab`)

The PRD names two targets: measurable in instruments after about 1 week of regular visits, and visible to the user after about 3 weeks. We make them testable:

- **Instrument threshold:** Δ ≥ 0.01 on at least one trait.
- **Visibility threshold (JND):** Δ ≥ 0.08 on a trait that maps to rendered or heard behavior. Before calibration begins, the JND is validated perceptually with internal viewers, using side-by-side renders of the same bird at trait `x` vs `x+δ`. It is retuned if people notice smaller or larger steps. The trait→render mappings (§7.6, §8.3) are designed so that 0.08 produces a noticeable but unremarkable change: for example, front-zone occupancy shifts by about 10 percentage points, and plumage chroma rises by about ΔE₀₀ 4.

**Personas** (event streams generated at accelerated time against the real `engine-core`):

| Persona | Pattern | Assertion |
|---|---|---|
| `regular` | 5 sessions/week, 12 min presence, 2 listen-ins of 90 s, 1 offer per session | Instrument threshold crossed on day 5–9; visibility crossed on day 18–28 for plumage and ≥1 other trait |
| `binger` | 3 h presence daily, constant listen-in, offers at every cooldown expiry | Visibility no earlier than day 16; no single day exceeds budget |
| `occasional` | 1 session/week, 10 min | Visibility after day 60; still monotonic |
| `returning` | `regular` for 3 weeks, absent 14 days, resumes | Zero negative deltas; P(wary) during absence is not elevated; envelope recovers within 3 sessions |
| `absent` | Adopts, never returns | Traits flat (tiny post-adoption filter residue only); mood cycles normally |
| `multi_device` | `regular` split across two overlapping devices | Credited presence equals the union, not the sum |
| `listen_heavy` / `offer_heavy` | Channel extremes | Correct traits move; others move from presence only |
| `year` | `regular` for 365 days | No bird exceeds its ceilings; inter-bird trait distance stays ≥ 0.1 on ≥ 2 traits (anti-homogenization) |

`drift-lab` runs on every change to `engine-core` or engine params. It emits trajectory plots as CI artifacts and fails the build on any assertion miss. Engine params are versioned (`engine_params_version`). A params change applies only going forward. **Vectors are never recomputed or back-filled** under new params.

### 5.4 Attention envelope (how neglect makes birds "ambient" without negative drift)

The PRD says an ignored bird "becomes ambient … greeting less often because less often is what's been observed", and it also says traits never move down. These fit together only if "ambient" is a separate, reversible, fast-ish state that is not part of personality. That state is `attention_envelope ∈ [0.35, 1]` per bird:

- It rises toward 1 with credited presence (time constant about 2 h of presence) and faster with listen-ins on that bird.
- It relaxes toward its floor of 0.35 during absence (half-life 5 days).
- It scales greeting probability, greeting intensity, call rate toward the viewer's side of the scene, and front-perch preference while the user is present.
- It has **no connection to mood valence**. It cannot produce wary, distress, or silence. The floor keeps every bird calling, preening, and alive.
- It is never exposed and never in the export (it isn't personality; it is transient state, like mood).

A user returning after two weeks finds birds that are "quieter than they were". After two or three sessions the envelope has mostly recovered.

### 5.5 Mood model

**States:** `alert`, `curious`, `content`, `wary`, `drowsy`, `roosting`. (`roosting` is the night sleep state. The name avoids a clash with the aviary's user-triggered "settled" lighting state; see D-8.)

**Latent dynamics.** Each bird has a continuous `(arousal a, valence v)` in `[-1,1]²`:

```
da/dt = (b_a(t) − a)/τ_m + Σ impulses_a        τ_m = 3 h (daily-ish relaxation)
dv/dt = (b_v(t) − v)/τ_m + Σ impulses_v
b_a(t) = circadian_species(t_local) + 0.2·(curiosity − 0.5) + weather_a
b_v(t) = 0.3 + 0.25·(boldness − 0.5) + 0.15·(warmth − 0.5)
```

**Impulses** (applied at event time, scaled by personality):

| Source | Δa | Δv | Scaling |
|---|---|---|---|
| Offer accepted by this bird | +0.10 | +0.25 | — |
| Listen-in on this bird (per minute, capped) | +0.05 | +0.10 | — |
| Alarm call from a neighbor within one zone | +0.30 | −0.30 | × (1 − boldness) |
| Rain start | −0.20 | −0.05 | — |
| Wind gust | +0.20 | ±0.10 | sign by boldness (bold → +) |
| Settle (all birds) | −0.25 | +0.05 | — |
| Dawn transition | toward alert via circadian baseline | | species-specific |

**Discretization** maps `(a, v)` to a state with hysteresis bands of 0.08 and a minimum dwell of 10 min (roosting ↔ alert at dawn and dusk is exempt):

- `v < −0.15 ∧ a > 0.1` → wary
- `a < −0.55 ∧ night` → roosting
- `a < −0.2` → drowsy
- `a > 0.35 ∧ novelty > 0` → curious (novelty comes from active offers, new sounds, weather onset, a newcomer)
- `a > 0.35` → alert
- otherwise → content

**Absence invariant:** absence length does not appear as an input to `a`, `v`, or impulses. A property test asserts that, over randomized histories, P(wary) is statistically independent of time since last presence.

**Persistence:** mood is stored and ticked continuously. Opening a tab never resets it. The snapshot carries the current mood and the client renders it as-is.

**Contagion:** wary spreads through alarm calls (above). Content spreads weakly: a bird within one slot of a content, preening bird gets `+0.02 v/min`. Bird-to-bird effects are gated by proximity, so the perch layout matters.

**Alarm-call source:** a wary bird with `a > 0.5` emits an alarm call with probability `0.02/min × (1 − boldness)`, and wind gusts can trigger one. These events are rare, short, and never tied to the user.

### 5.6 Perch selection

Each bird re-evaluates its perch at a random interval of 8–25 min (shorter when wary or curious), or on impulses above 0.25.

`score(zone) = w_b·boldness·zone_front + w_m·mood_pref(zone) + w_w·warmth·neighbor_affinity(zone) + w_e·envelope·present·zone_front + noise(σ=0.15)`

- Zone capacities by layout (§7.5) cap occupancy. With 7 birds the front zone holds at most 3.
- Moves are planned into the script with a flight start time. The client animates the flight (or cross-fades it in reduced-motion mode).
- Users have no input into this. It is signal, not layout.

### 5.7 Call-grammar runtime (server half)

The server decides **when** each bird calls and **how intensely**. The client decides **how it sounds** (§8).

- **Per-bird call process:** an inhomogeneous Poisson process with a 4 s refractory period.
  `λ = λ_species · (0.4 + 1.2·vocal_freq) · mood_mult · daylight_mult · weather_mult · (0.6 + 0.4·envelope_if_present)`
  Mood multipliers: alert 1.3, curious 1.1, content 1.0, wary 0.5 (plus alarm calls), drowsy 0.3, roosting 0.02 (nightjar-like species: inverted circadian, so roosting by day and active at night).
- **Call/response:** when bird A calls, each other bird B schedules a reply with probability `p = 0.15 + 0.5·warmth_B` (× proximity factor), at a 0.6–2.5 s offset, so replies stagger.
- **Chorus:** when ≥2 birds with `vocal_freq > 0.55` and non-drowsy mood both have calls in the same 20 s window, the planner may promote the window to a chorus event: 3–6 interleaved calls across birds, planned together to avoid exact overlap. Emits an observation.
- **Planning horizon:** each tick plans 180 s of calls (overlapping the previous plan; the first 60 s of the previous plan are kept to preserve continuity). Each call carries `{at, motif_index, seed, gain}`. The motif is chosen by the species grammar weights plus the bird's voiceprint preferences.
- Rain multiplies λ by 0.5 while raining and for 10 min after. Night multiplies diurnal species by about 0.05.

### 5.8 Weather scheduler

- Rain: Poisson, about 3 per week, 5–15 min each, mostly during local daylight, soft intensity 0.2–0.5. Wind: about 5 gust episodes per week, 2–6 min each. No storms or snow exist in the model.
- Weather is scheduled per aviary 24 h ahead from the seeded RNG and stored in `aviaries.weather`, so both devices and visitors see the same weather.

### 5.9 Return-greeting planner (shared, runs client-side)

This runs in `engine-core` on the client at session start and on `visible_return`, from snapshot state plus absence length. That gives zero network latency on the anchor moment.

- **Absence** `A = now − max(last_presence_end_at (server), local last-present timestamp)`, measured account-wide. A laptop opened 5 minutes after using the phone gets a short-return greeting.
- **Candidate score per awake bird:** `s = 0.45·boldness_proxy + 0.35·greet_q + 0.2·mood_greet(mood) + 0.15·envelope + Gumbel noise(β=0.12)`. `greet_q` and `approach_q` are the quantized derived params. Roosting birds are ineligible except the nocturnal species at night.
- **Primary greeter** is the arg-max. **Additional greeters** join with probability `sigmoid(k·(s − θ(A)))`, capped at 3. The threshold `θ` falls as `A` grows, so longer absences bring more of the aviary around. Low-scoring (warier) birds often don't greet on a given day.
- **Stagger:** the primary starts 300–1400 ms after the first rendered frame, so it lands within the "first second or two". Additional greeters follow at randomized offsets of 700–2600 ms. Birds never greet in unison.
- **Form** is a continuous blend, not a pick from a list. The inputs are `A` (log-scaled: minutes → glance; hours → head-tilt + two-note call; days → step toward the front + longer call; more than 3 days → a short hop or flight to a closer perch, a longer phrase, and a possible reply from a second bird), the bird's **greeting signature** (seeded from `bird_id`, so Pip greets like Pip), and the mood.
  - The parameters are head-turn angle, delay-before-look, step count, call-phrase length, pitch-contour seed, and tilt amplitude.
- **Anti-repetition:** the last 20 greeting parameter vectors are kept in IndexedDB per device, and the recent history also travels in `greeting_shown` observations. If a sampled greeting falls within distance ε of one of them, it is resampled. The continuous parameters and seeds make exact repetition practically impossible anyway.
- **Hidden → visible:** if the tab was hidden for less than 20 s, there is no greeting (it is not a return).
- `greeting_shown` is posted with the order. It feeds notebook observations like "pip greeted before wren today, first time this week".

### 5.10 Arrivals (third bird and beyond)

- **Schedule by aviary age only:** 3rd bird at about 90 days, 4th at about 180, 5th at about 270, 6th at about 365, 7th at about 540, each ±10 days of seeded jitter. The schedule never depends on visits, presence, or interaction.
- **Arrival:** the tick creates a `newcomer` bird (full identity, voiceprint, personality seeds) that perches in the back zone and behaves as a shy visitor. The notebook may record it within a day or two ("a small finch has been at the back perch since tuesday, watching.").
- **Adoption:** in account/settings → Birds, the newcomer appears with a suggested name and two actions: **Let it stay** (with name) and **Not now**. No badge, dot, or prompt points there. If the user does nothing for 30 days, the newcomer quietly stops visiting, and the next opportunity comes at the next scheduled interval (D-10). **Not now** has the same effect right away.
- A newcomer counts toward the cap of 7. The hard cap is enforced in the tick and by a DB constraint trigger.

### 5.11 Offer resolution

- **Placement:** seed → the front rail near the listen-in target if there is one, else the front-center rail; pool → the front ground plane; song fragment → played softly from off-scene front center (8 fragments in the library, each a short motif in the product's synth voice).
- **Resolution** (`engine-core.resolveOffer(snapshot, offer, offer_nonce)`), per eligible bird (awake, not in cooldown): `approach = σ(3·(curiosity_proxy + mood_approach(mood) + 0.3·(zone==front) − 0.9))`. The highest-approach bird responds first. Wary birds get a delayed approach (20–60 s) with probability tied to approach. Drowsy birds usually just watch.
  - Fragment reactions: `join in` (vocal_freq high, content/alert), `go quiet` (drowsy/wary), `call against` (curious, high boldness). Joining in means the bird replies with its own motif keyed to the fragment's contour.
  - Pool reactions: drink / bathe / watch, weighted by mood and boldness.
- The client resolves immediately for display. The server re-resolves with the same nonce and its own cooldown view, and the **server result credits drift**. Divergence happens only when a second device changed cooldowns in the last few seconds; it is cosmetic and never corrupts state.
- **Cooldown:** 4 min per bird from an accepted offer. During cooldown the bird may glance at a new offer but does not engage, and gets no drift credit. The offer affordance is never disabled and never shows a countdown (D-11).

---

## 6. Sync model

### 6.1 Canonical state

A single canonical record per aviary lives in Postgres. The tick is its only writer. Clients read snapshots and append events. The model has no client state to merge and no last-write-wins anywhere.

### 6.2 Client pull schedule

| Trigger | Action |
|---|---|
| Navigation | Snapshot is inlined in the HTML (edge fetch from cache); no extra request |
| Visible return (`visibilitychange → visible`) | Immediate `GET snapshot` with `If-None-Match` |
| Frame gap > 5 s (rAF delta or wall/monotonic clock divergence, e.g. laptop resume) | Immediate pull |
| Keepalive while visible | Every 60 s, phase-aligned to 10 s after the aviary's tick phase (published in the snapshot) |
| After posting interaction events | Response carries `latest_version`; client pulls if newer |
| Visitor | Every 30 s (bounds revocation latency) |

### 6.3 Clock sync

The snapshot response carries `server_time`, and the client measures RTT. Offset = `server_time + RTT/2 − local_now`, smoothed with a median of the last 5 samples. Scripts and call plans are in server time, so two devices open side by side show the same calls within about 100 ms.

### 6.4 Reconciliation (never teleport)

When a new snapshot arrives:
- **Same perch, same mode:** nothing visible changes. Future script entries replace the old ones.
- **Different perch** (a planned move the client hadn't reached, or a change made while hidden): the bird flies from its rendered position to the new one on a natural arc (reduced-motion: cross-fade). A bird flying reads as the aviary moving, not as a correction.
- **Mood change:** behavior blends over 1.5–4 s through the animation state machine.
- **Unknown new bird** (a newcomer arrived): it flies in to the back perch.
- **Return after long hidden periods:** the browser may briefly show the stale last frame. On `visible` the client re-renders immediately from the retained script if it is still within horizon. Otherwise it keeps the stale positions for at most 250 ms while the pull completes, then reconciles by flights. For hidden periods over 10 min, a planned relocation happens as a "birds shift around as you look" sequence staggered over 3–8 s, not all at once.

### 6.5 Event outbox (client → server)

- Events go into an IndexedDB outbox with `client_event_id = UUIDv7`. Batches flush every 5 s (interactions: immediately, debounced 300 ms) with exponential backoff while offline.
- Heartbeats buffered during short offline spells (up to 10 min) flush on reconnect. The server accepts them if `client_ts` is within 10 min and the monotonic clock deltas are consistent. Anything older is dropped (aggregate counter only).
- `pagehide`: `sendBeacon` for `presence_end` plus any pending interaction events.

### 6.6 Presence dedupe across devices and tabs

The server keeps a per-account presence timeline tail in `aviaries.presence_state`. Each heartbeat claims the interval `[recv − min(claimed, recv − prev_recv_same_device + 5 s, 25 s), recv]`. The tick merges intervals across all devices and tabs for the account and credits the **union**. Two tabs or a laptop and phone watching at once count once. Tabs share one device-session per browser profile (BroadcastChannel leader election), and only the leader tab sends heartbeats, which cuts traffic. The union makes correctness independent of that optimization.

### 6.7 Conflicts that can still happen, and their surfaces

| Case | Handling | Surface (matter-of-fact) |
|---|---|---|
| Session revoked or expired mid-use | Next request 401 → sign-in page | "Your session timed out. Sign in again to keep watching." |
| Magic link replayed or expired | Consume fails | "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| Snapshot unavailable (outage) | Retry with backoff (1, 2, 4, 8 s); keep rendering the last script, then idle birds in place | After 30 s without a snapshot on a fresh load: "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." In a running session nothing is shown until 5 min of failure, and then the same text sits in the top bar region, never over the scene. |
| Events rejected (validation) | Dropped; aggregate counter | None; the user did nothing wrong |
| Tick quarantined | Last good snapshot served | None; birds idle normally |

---

## 7. Frontend rendering pipeline

### 7.1 Boot path (first bird < 500 ms)

The critical path is: HTML → inline boot → critical JS → first frame with birds.

1. **Edge worker** streams HTML: an inline `<style>` with a quiet-field sky gradient chosen by an inline 300-byte script from local time. The first paint therefore shows the right sky. No spinner, no white flash.
2. The edge worker fetches the account's snapshot from the nearest regional snapshot cache (p50 target 25 ms, p95 80 ms) and inlines it as `<script type="application/json" id="snap">`. If the fetch takes longer than 120 ms, the HTML streams without it and the boot script fetches it directly.
3. `<link rel="modulepreload">` for the critical bundle (≤ 90 KB gz: renderer, scene, animation core, `engine-core` subset, snapshot client, presence tracker) and for **the species chunks needed by this aviary only** (≤ 12 KB gz each). HTTP/3 with 0-RTT resumption for returning visitors.
4. The boot script rasterizes bird part sprites for the **front-most / greeter bird first**, then the rest, and draws the first frame with all birds in their scripted poses at `now`. The target is all birds in frame 1. If rasterization of the rest would exceed 40 ms on a slow device, those birds appear on frame 2–4, already mid-action. That cannot read as an entry, because they are already on their perches.
5. Deferred until idle (`requestIdleCallback`, 1.5 s timeout): audio module + AudioWorklet, voice/narration, notebook, offer panel. Loaded on demand: settings, accessibility settings, visits, adoption flow.
6. **Service worker** (no push, no notifications) caches the shell and hashed assets with an immutable, stale-while-revalidate policy. Returning visits need only HTML (the snapshot rides along) and hit the cache for everything else.

**Budgets** (CI-enforced with `size-limit`): HTML + inline ≤ 20 KB gz; critical JS ≤ 90 KB gz; total initial (everything eager) ≤ 400 KB gz, warn; 2 MB gz hard fail per PRD.

**Metric:** `first_bird_visible` = time from `navigationStart` to the first `requestAnimationFrame` callback after the frame that drew at least one bird. It is marked via `performance.mark` and confirmed through the next rAF, which approximates presentation. It is reported as an aggregate histogram (§13).

**Reference devices:** mid-tier Android (Moto G Power class, 2023) over a real 4G profile (RTT about 70 ms, 9 Mbps down) and emulated "fast 4G". Target p75 < 500 ms on warm DNS, cold cache; p75 < 300 ms for returning visits.

**Contingency:** if the phase-0 spike misses by more than 20%, add server-rendered inline SVG of the first frame's birds. It is drawn from the same species vector definitions, and the canvas replaces it pixel-aligned on its first frame.

### 7.2 Renderer choice

The renderer is **Canvas2D with pre-rasterized part sprites**, behind a `Renderer` interface that has a WebGL2 implementation as a fallback path.

Rationale: at most 7 birds of about 10 parts each is ≈70 `drawImage` calls with transforms, plus a few cached background layers and ≤30 particles. That fits comfortably at 60 fps with `drawImage` on GPU-accelerated Canvas2D. It also avoids shader compilation on the critical path and context-loss handling. The phase-0 spike (§15) must show ≤ 6 ms main-thread plus ≤ 8 ms GPU per frame on the reference laptop (2020 Intel i5 MacBook Air / Intel UHD 620 Windows laptop) at DPR 2, 2560×1440. If it can't, switch to the WebGL2 implementation. The scene graph API does not change.

### 7.3 Scene composition (back to front)

1. **Sky:** a gradient from a time-of-day palette LUT (48 keyframes across 24 h, interpolated in OKLab), cached to an offscreen canvas and re-rendered every 30 s or on change.
2. **Far foliage:** 2 static layers with very slow wind sway, a sine translate of ≤ 0.6% of width with a 20–40 s period.
3. **Back perch zone:** branches drawn higher and smaller, with aerial-perspective desaturation of 15% and slight haze.
4. **Middle perch zone.**
5. **Birds:** z-sorted by zone then y. Each bird is a 2D puppet (§7.4).
6. **Front perch rail + ground plane** (pool offer appears here).
7. **Ambient particles:** leaves and feathers from a preallocated pool of 12, spawned every 6–20 s with gentle curl-noise paths.
8. **Occasional foreground branch/leaf:** one every 1–3 min, soft focus via a pre-blurred sprite.
9. **Weather overlay:** rain streaks from a pool of 120 short strokes at low alpha, plus a sky tint. Wind raises sway amplitude.
10. **Color grade:** time-of-day and settle warmth via one full-canvas `fillRect` with `globalCompositeOperation: 'soft-light'`, plus night dimming via `'multiply'`. The per-bird plumage shift is baked into its sprites.
11. **DOM overlay (not canvas):** captions, focus rings, and screen-reader proxy elements for birds (§10).

The top bar is DOM above the canvas and never inside the scene.

### 7.4 Bird puppets and idle micro-motion

- **Species definition:** parametric vector parts (body ellipse+tail curve, head, beak, eye, wing, tail, legs) as `Path2D` generators. Per-bird `markings_seed` adds small variations such as cap shade, wing-bar width, and eye-ring. Plumage saturation (`sat_q`) sets chroma in OKLCH before rasterization. Sprites are re-rasterized only when `sat_q` changes, which happens rarely.
- **Rig:** about 8 bones (root, body, neck, head, beak-open, wing, tail, feet). Pose = bone angles/scales + a body "fluff" scale + eye openness.
- **Behavior state machine** (driven by the snapshot `behavior.mode` + mood): `perched_idle`, `scanning`, `preening`, `tilting`, `dozing`, `calling`, `hopping`, `flying`, `drinking`, `bathing`, `approaching_offer`.
- **Continuous layers** (all procedural, with every frequency < 3 Hz except wingbeats in flight):
  - Breathing: sine on body scale, 0.4–0.9 Hz, amplitude by arousal.
  - Head saccades: Poisson saccades whose rate depends on mood (wary high, content low). The head holds, then moves with ease-out over 120–220 ms.
  - Head tilt toward sound: on any call event (including other birds' calls and the user's song fragment), a tilt toward the source, gated by the `tilt_q` curiosity proxy.
  - Weight shuffle every 20–90 s. Blinks every 3–10 s (drowsy: slow half-blinks).
  - Preen: key-pose sequences (wing-lift, beak-to-shoulder, shake), blended with 2nd-order critically damped springs.
  - Fluff: drowsy and roosting birds and cold mornings raise fluff by 10–20%.
  - Calling: beak-open and throat pulse driven by the **actual synthesized envelope** from the audio module. It stays in sync even with audio locked or muted (the pulse is still computed).
- **Mood → motion mapping:** the table below is owned by animation design and reviewed with sound and voice.

| Mood | Posture | Motion character |
|---|---|---|
| wary | upright, sleeked, back zone | frequent scans, short holds, minimal preen |
| content | relaxed, slightly fluffed | long preens, slow scans |
| curious | forward lean | tilts, hops toward novelties, watches leaves |
| alert | upright, sleek | quick scans, more calls |
| drowsy | low on the perch, fluffed | slow blinks, head tuck starts |
| roosting | head tucked, eyes closed | breathing only, occasional shuffle |

- **Entrance exceptions:** a new account's starter birds fly in (PRD), and so do newcomers. Nothing else ever has an entrance animation.

### 7.5 Responsive layout

- **Scene space:** normalized coordinates with three layout templates by aspect ratio: **portrait** (< 0.8), **standard** (0.8–1.9), **wide** (> 1.9). Each template defines perch slots per zone (portrait stacks zones vertically: back high, front low; wide spreads horizontally).
- **Slot capacity** per template guarantees that 7 birds fit without overlap: front 3, middle 3, back 3 slots. Bird scale is `clamp(viewport_min_dim / 9, 36 px, 110 px)` CSS, with depth scaling by zone.
- **No-crop guarantee:** a layout solver computes a safe inset rectangle. Every slot's bird bounding box, including maximum preen extents and tilt, fits inside it. Flight arcs are clamped to the viewport. This is tested across a matrix of 40 viewport sizes from 320×568 to 3440×1440 at 7 birds (visual regression plus bounding-box assertions).
- Resize and orientation change: re-layout within one frame. Birds keep their slot identity and move to the new slot coordinates without animation.
- Canvas DPR = `min(devicePixelRatio, 2)`. A dynamic quality governor, triggered when the 2 s frame-time p95 exceeds 14 ms, first drops the background layers to DPR 1, then the particle count, then foliage sway.

### 7.6 Trait → visual mappings (the only ways personality shows)

| Trait | Visible expression |
|---|---|
| boldness | front-zone preference in perch scoring; greeting approach distance; offer approach speed |
| warmth | greeting likelihood; call replies; perching near other birds |
| vocal_freq | call rate; phrase length; chorus participation |
| plumage_sat | sprite chroma (OKLCH C × (0.55 + 0.6·sat)); feather detail layer alpha |
| curiosity | tilt frequency; offer approach; leaf-watching |

### 7.7 Day/night and settle lighting

- **Lighting schedule** (in the canonical timezone): dawn 05:30–07:30 (warming), day 07:30–17:30 (midday brightest), dusk 17:30–20:30 (warm, calls quieter), night 21:00–05:00 (dim to about 35% luminance, with the birds and perches still legible). A mild seasonal shift of ±45 min comes from month and hemisphere, inferred from the IANA zone's country. No location data is collected.
- **Night is not dead:** if a nocturnal bird is present, it is active and calls. Either way, there is a faint procedural night ambience, roosting birds shift occasionally, and a leaf still drifts now and then.
- **Settle:** the grade moves toward the evening palette over 4 s (reduced-motion: 7 s), the audio master falls −8 dB over 4 s, and the local call plan thins to 30%. Birds visually tend toward drowsy postures locally. Any click, tap, or key in the aviary within 5 s reverses it over 1.5 s, and no event is sent. After 5 s the `settle` event is sent and presence ends. The aviary stays settled on this device until a deliberate click, tap, or key, which re-engages: lighting returns to the current time-of-day over 3 s and presence resumes. Pointermove alone does not un-settle.

### 7.8 Reduced-motion mode (a designed surface)

It activates from `prefers-reduced-motion: reduce` (followed live) or the accessibility setting (on / off / follow system).

- **Presenter layer:** the animation state machine still runs. The RM presenter samples it into **key poses** at a 3–8 s cadence (per behavior) and **cross-fades** between consecutive poses over 900–1400 ms. Each bird is drawn twice at complementary alpha during a fade. Preening becomes a slow sequence of 3–4 preen poses.
- **Flights** become cross-fades: the bird fades out at A while fading in at B over 1.4 s.
- **Removed:** leaf and feather drift, foreground passes, parallax sway, rain streak motion (replaced by a static soft rain veil, a slight desaturation, and a cooler grade), breathing, and saccades.
- **Kept, slowed:** day/night grading (unchanged pace, already slow), the settle transition (7 s), and the top-bar fade (1.2 s).
- **Unchanged:** calls, captions, narration, drift, mood, the notebook, greetings (rendered as a pose change: head-up and turn toward the viewer as a cross-fade, plus the call).
- Designer-owned pose libraries per species and behavior are a named deliverable. RM gets its own visual QA pass and its own screenshot baselines.

### 7.9 Hidden tab and power

- Hidden: the rAF loop stops (browser-native, plus explicit guards). Audio fades over 300 ms, then `ctx.suspend()`. Presence ends. The outbox flushes. Timers are reduced to the keepalive only, which is itself suspended while hidden.
- Visible: pull snapshot → reconcile → greeting planner (if hidden ≥ 20 s) → audio resume with a 600 ms fade-in mid-call (never restarting a call).
- Idle CPU target on a laptop: < 8% of one core at 60 fps, measured. Background tabs cost 0%.

---

## 8. Audio pipeline

### 8.1 Graph

```
AudioWorkletNode "aviary-synth" (8 outputs: 7 bird buses + 1 ambient)
  └─ bird bus i → GainNode (listen-in mix) → BiquadFilter (distance/depth lowpass)
                 → StereoPannerNode (x position in scene) ─┐
  └─ ambient bus → GainNode ───────────────────────────────┤
                                                           ▼
                           master GainNode (settle/night/mute) → DynamicsCompressor (gentle limiter,
                           threshold −10 dBFS, ratio 4, soft knee) → destination
```

- **One worklet, preallocated:** 24 voices, each with 2 oscillators (sine/triangle, FM-capable), a noise generator, a state-variable filter, and 2 envelopes. Pitch contours are breakpoint arrays with up to 8 points. **No `OscillatorNode` or `AudioBufferSourceNode` per call.** The node count is constant for the life of the page, which satisfies "audio buffers reused; no per-call allocation" by construction.
- Main-thread scheduler: every 50 ms it looks ahead 250 ms in the merged call plan (server plan + local reactive calls), converts server time → `AudioContext.currentTime` using the clock offset and `ctx.getOutputTimestamp()`, and posts compact note messages to the worklet over its `MessagePort`. Messages use transferable `Float32Array` batches from a 4-buffer pool.
- `latencyHint: 'playback'` for power. The cost is 40–100 ms of output latency, acceptable for calls. Listen-in ramps are slow by design anyway.

### 8.2 Call grammar (client half)

- **Species motif library:** 6–12 motifs per species. Each motif is a sequence of syllables with typed parameters: `{dur, f0_start, f0_end, contour: 'rise'|'fall'|'arch'|'flat'|'warble', fm_index, fm_ratio, noise_mix, am_rate (trills), attack, release, amp}`. A phrase grammar per species, e.g. warbler `A B{1,3} C?`, wren `T{3,8} (A|B)`, nightjar-like `CHURR{1} (pause CHURR)?`.
- **Voiceprint** (per bird, immutable, generated at adoption from `bird_id`): pitch offset (±3 semitones within the species range), timbre bias (FM index/ratio offsets), preferred motif order (weights), signature interval (a characteristic leap used at phrase starts), and rhythm swing. **These are what make Pip recognizably Pip.** Nothing else is allowed to move them.
- **Mood and trait modulation** (bounded so identity features are untouched): tempo ±15%, amplitude ±4 dB, pitch jitter 0–25 cents, phrase length ±1 syllable group, trill rate ±10%. Vocal-frequency drift changes call rate and phrase length, never pitch or timbre.
- **Per-rendition variation:** every syllable gets seeded micro-variation (duration ±6%, f0 ±15 cents, amplitude ±1.5 dB). With continuous parameters, two identical renditions have effectively zero probability, so a user never hears the same call twice exactly.
- **Anti-uncanny rules:** attacks ≥ 8 ms (no clicks or startles), nothing perfectly periodic (jitter on every repeated element), spectral range 1.5–8 kHz for songbirds and 0.6–2.5 kHz for the dove and nightjar types, no formant structures resembling human speech, peak per-call level ≤ −12 dBFS before the master. A sound designer owns the libraries.
- **Song-fragment offers:** 8 authored fragments, rendered by the same synth with a distinct "offered" timbre (softer, rounder) so they read as a gift, not another bird.

### 8.3 Mixing, chorus, and listen-in

- **Ambient mix (no listen-in):** each bird bus at −6 dB. Depth filtering: back zone lowpass at 4.5 kHz and −3 dB, middle 7 kHz, front open. Pan from x position (±0.6 max; never hard-panned).
- **Chorus:** calls are real simultaneous voices summed in the worklet (no loops, no phase-cancel artifacts). When more than 3 voices overlap, a bus-level gain rider applies −1.5 dB per extra voice to keep loudness steady.
- **Listen-in engage:** the focused bus goes to 0 dB and the depth filter opens. Other buses drop to −15 dB (floor; **never muted**) with a lowpass at 2.5 kHz. The ramp uses `setTargetAtTime` with τ = 0.55 s on all nodes, about 1.6 s to 95%, so it feels like attention shifting, not a switch. Disengage uses the same ramp back to ambient. Switching focus from bird A to bird B crossfades both ramps at once.
- **Ambient bed:** procedural wind and leaf rustle (filtered noise with slow LFOs) at −30 dB. Rain adds a filtered-noise patter bed at −26 dB. Night adds a sparse insect texture (seeded pulse trains) at −32 dB. All procedural, no samples.
- **Settle, dusk, night:** master gain curve −3 dB at dusk, −6 dB at night, plus −8 dB while settled.

### 8.4 Autoplay unlock (important UX constraint)

Browsers keep a new `AudioContext` suspended until a user activation (click, tap, or key). Pointermove does not count. The product must feel audible without announcing anything, so:

- There is no "click to enable sound" button, modal, or overlay (that would be an announcement).
- The context is created at boot and resumes on the **first activation gesture anywhere**. Audio then fades in over 1.5 s, mid-call where calls are in progress. The first click on a bird is simultaneously listen-in and unlock, which is naturally the moment of paying attention.
- **New accounts:** the adoption flow requires clicks (naming), so audio unlocks there. The first bird's fly-in is heard.
- Chrome's media-engagement heuristics will let returning desktop users autoplay. Safari and Firefox will usually require a gesture.
- While locked, calls still animate beaks and throats. If captions are enabled, they still show.
- Aggregate metric: the share of sessions reaching audio-unlocked within 60 s (no per-account dimension). If it is low, the product team revisits this (R-3.4).

### 8.5 WebAudio fallback

On `AudioContext` construction failure, `audioWorklet.addModule` failure, a context stuck in `interrupted` for more than 10 s, or permission denial, the aviary plays in **silence with captions on by default**. The caption toggle stays user-controllable. No recorded-audio path exists. The failure increments an aggregate `audio_pipeline_error{kind}` counter. The a11y settings panel says, matter-of-factly: "Sound isn't available in this browser right now. Captions are on."

iOS: the hardware silent switch mutes WebAudio. This cannot be detected, so we accept it and document it in help. It is not a fallback case.

### 8.6 Mute

Mute lives in the accessibility settings panel (audio: on / muted) with a keyboard shortcut (`m`, remappable). A mute ramps the master to −∞ over 400 ms, then suspends the context to save battery. It emits `audio_state` events. Muting has no negative consequence. Unmuted presence feeds the small `heard` drift channel toward vocal frequency, which is how the PRD's "whether you mute the calls or let them play" shapes birds without punishing a muter (D-12).

---

## 9. The naturalist voice system (notebook, narration, captions)

All three surfaces share the `voice` package: a typed generative grammar (Tracery-like, with typed slots, constraints, and weights) plus lexicons, authored by a staff writer. Using an LLM was considered and rejected. Hosted LLMs would send per-bird state to a third party (a privacy violation). Self-hosted models risk voice drift and hallucinated observations. And a grammar is testable by lint.

### 9.1 Style rules, machine-enforced (copy lint)

For every generated product-surface string from a 100k-sample fuzz corpus per build:

- Lowercase except proper UI labels. No `!`. Present tense favored (a tense heuristic flags past-tense verbs outside date-anchored notebook lines).
- Banned lexicon (sample): `you`, `your`, `welcome`, `back!`, `achievement`, `unlocked`, `streak`, `level`, `badge`, `points`, `score`, `visited`, `days in a row`, `congrat`, `%`, `mood:`, `boldness`, `curiosity` (as a trait noun), `vocal frequency`, `perch 1/2/3`, digits other than in date or time words.
- No user-behavior facts: the grammar's slot types include no absence length, visit count, or session count. The observation schema has no such fields, so they cannot be generated.

### 9.2 Field notebook generation

- **Observations** are emitted by the tick and by client-reported `greeting_shown`, all about the aviary: greeting order (and "first time this week/month" relative to the aviary's own observation history), perch shifts relative to a bird's 14-day perch baseline, chorus events, weather with notable behavior ("low calls only during the rain"), long preens, first offer acceptance of a kind by a bird, pool bathing, night calls from the nocturnal bird, newcomer arrival or departure, plumage change past a threshold ("pip's wing-bars look richer this week" — phrased as appearance, never numbers).
- **Salience:** each observation kind has a base salience times a novelty factor (decays with repetition of the same kind for the same bird).
- **Sparsity controller** (per aviary): a token bucket of 1 token per 3 days, capacity 2. Writing an entry costs a token. Observations with salience ≥ 0.9 (arrival, a first chorus, first night call) may borrow up to 1 extra token. There is a hard maximum of 1 entry per local day, and 2 on arrival days. The salience threshold rises with recent entry count, so a heavy user gets the same rhythm as a light user.
- **Timing:** the naturalist worker evaluates at presence-window close (5 min after the last presence end) and at local-day rollover. At most one entry per evaluation. The entry is dated with the aviary's local date ("tuesday — …").
- **Composition:** it picks 1–2 observations, fills a grammar template with bird names, place words (the high branch, the back perch, the front rail), and time words (this morning, just after the rain), and may add a coda from a sensory lexicon ("a leaf drifted down past the back perch and neither bird looked up"). It rejects candidates whose template id or n-gram signature matches any of the aviary's last 30 entries.
- **Notebook UI:** a side sheet from the top bar. Entries are grouped under lowercase day headings, reverse-chronological, with infinite scroll back to the first entry. The list is virtualized with a DOM window of about 40 items and references dropped after scroll-out. No unread badge, no "new" marker, no count. Read-only: no edit, delete, annotate, share, or export controls in the panel. Entries are included in the account export.

### 9.3 Captions

- Captions are generated client-side from **the exact parameters sent to the synth** for that call. Syllable count → "three-note"; contour → "rise / fall / dip"; AM rate → "trill"; attack and amplitude → "soft / sharp"; gaps → "paused"; repetitions → "again"; zone → "from the back perch" (only when the bird is not the nearest caption anchor, to keep captions short).
- **Examples:** "a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch".
- **Display:** a DOM label anchored above the calling bird. It fades in with the call onset (200 ms), holds for the call duration + 1.2 s, and fades out (400 ms). There are at most 3 captions on screen; older captions for chorus calls collapse to "pip and wren, calling together". Text sits on a soft translucent pill whose backdrop is computed to guarantee ≥ 4.5:1 contrast against the pixels behind it (sampled from the scene luminance at the anchor point, clamped to a tested min/max pill opacity).
- Captions are `aria-hidden`. Screen-reader users get calls through narration (no double-reading).

### 9.4 Screen-reader narration

- **Live region:** one visually hidden `<div aria-live="polite" aria-atomic="true">` whose text is replaced per update. Never `assertive`.
- **Cadence:** idle updates every 30–60 s, randomized. Prioritized events (return-greeting, offer reaction, settle, a newcomer's first appearance, the start of rain) are narrated within 1–2 s, debounced so the prose describes what actually happened. After a prioritized update the idle timer resets, so events don't produce bursts.
- **Composer:** reads **rendered** state (not the snapshot) so narration matches what a sighted user sees: time-of-day phrase, weather, 1–2 birds chosen by recent notable change (rotating attention across birds), their place and posture expressed bodily ("fluffed against the cool air", "very still, watching from the back"), and recent calls from the caption descriptors. It alternates naming and description ("pip, the small grey one, …") so it reads like prose and never like a state list. A repetition guard keeps the last 12 narration signatures.
- **Examples:** "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle." / "wren looks up from preening and turns toward the front. a quiet two-note call."
- **Settings:** narration **running** (default), **events only**, or **off**; **show narration as text** (a strip beneath the scene, outside the scene proper, AA-contrast); **pause narration** shortcut (`n`, remappable).
- Screen readers cannot be detected, so the live region is always active. Sighted users are unaffected because it is visually hidden.

---

## 10. Accessibility surfaces

Accessibility ships in v1 with the rest of the product, on the same milestones (§15), not after.

### 10.1 Keyboard model

| Context | Keys | Behavior |
|---|---|---|
| Page | `Tab` | Top bar items in order: Offer, Field Notebook, Accessibility, Settings → aviary scene → (panels when open) |
| Entering the scene | `Tab` | Focuses the "first bird": the front-most, left-most resident bird. The aviary is a single tab stop with roving tabindex over birds. |
| Scene | `←/→` | Moves focus to the spatially nearest bird left/right (same zone first) |
| Scene | `↑/↓` | Moves focus toward the back/front zone |
| Scene | `Enter` / `Space` | Toggles listen-in on the focused bird |
| Scene | `Esc` | Ends listen-in; focus stays on the bird |
| Scene | `Tab` / `Shift+Tab` away | Leaves the scene; ends listen-in (PRD: "moves keyboard focus away") |
| Global | `o` offer panel, `s` settle, `b` notebook, `m` mute, `n` pause narration | Single-key shortcuts are active only when focus is in the aviary or top bar. All can be turned off or remapped in accessibility settings (WCAG 2.1.4). |
| Offer panel | arrows, `Enter`, `Esc` | Menu of: a seed, a song fragment (submenu of 8), a still pool, and settle the aviary (D-5). Focus returns to the trigger. |

Moving focus to a different bird ends any active listen-in (PRD). Engaging listen-in on the new bird requires `Enter` (D-2).

**Focus follows identity:** if the focused bird flies to another perch, focus stays on that bird and the ring travels with it.

### 10.2 Bird proxies for assistive tech

- Each bird has a DOM proxy element (`role="button"`, `aria-pressed` = listen-in state) absolutely positioned over its canvas bounds and updated every frame through `transform`, with no layout thrash. This supports keyboard focus, pointer hit-testing parity, and touch exploration with VoiceOver and TalkBack.
- **Accessible name:** the bird's name. **Accessible description** (`aria-describedby`, regenerated on focus and at most every 20 s while focused): a naturalist sentence ("on the front rail, preening in the morning light"). No state tokens, no numbers, no mood labels.
- The scene container is `role="group"` with `aria-roledescription="aviary"` and a short accessible name ("the aviary"). No `role="application"`, so screen-reader browse modes still work.

### 10.3 Focus indicator

A two-tone ring (2 px near-black inner + 2 px near-white outer, soft 4 px blur halo) drawn in the DOM overlay as an ellipse around the bird. It keeps ≥ 3:1 against both day and night backgrounds by construction (one of the two tones always contrasts) and is verified in visual tests at 6 times of day × weather states. It shows only for `:focus-visible` (keyboard), not for mouse or tap listen-in, which uses the audio shift plus the bird turning toward the viewer.

### 10.4 Pointer and touch

- Click or tap a bird → listen-in toggles. Tapping empty scene area, clicking another bird, or clicking the focused bird again disengages (PRD).
- Hit targets are the bird's bounds expanded to at least 44×44 CSS px.
- **Touch and the top bar:** it fades after 4 s without touch. A tap in the top 15% of the screen reveals it without also disengaging listen-in.

### 10.5 Contrast

- All user-copy text (top bar labels, panels, settings, errors, captions, visual narration strip, notebook) must pass WCAG AA: 4.5:1 for body text, 3:1 for large text and non-text UI.
- Tokens come from the design-system spec. CI runs axe-core on every panel in light-sky and night states, plus a custom check that samples caption-pill contrast over recorded scene frames.
- **Top bar fade:** faded icons (opacity 0.08) are decorative in that state. Hover, focus, or cursor movement restores full opacity within 200 ms, and focused elements are always at full opacity. An accessibility setting ("keep the top bar visible") disables the fade for users who need persistent controls.

### 10.6 Accessibility settings panel (matter-of-fact voice)

Motion (follow system / reduce / full) · Captions (off / on; size S/M/L) · Narration (running / events only / off; show as text) · Sound (on / muted) · Keyboard shortcuts (on / off; remap) · Keep top bar visible. Settings sync per account. "Follow system" is evaluated per device.

### 10.7 Accessibility testing

- **Automated:** axe-core, keyboard traversal scripts (Playwright), live-region cadence tests (no more than 1 idle update per 30 s; events within 2 s), RM screenshot baselines, a flashing check (no luminance flashes > 3 Hz; WCAG 2.3.1).
- **Manual release gate:** VoiceOver (macOS Safari, iOS Safari), NVDA (Firefox and Chrome), JAWS (Chrome), TalkBack (Chrome), plus a paid panel of at least 6 disabled testers (screen-reader users, vestibular-sensitive users, Deaf/HoH users) in the beta. Their brief goes beyond "can you operate it": "does it feel alive?"

---

## 11. Accounts, security, privacy (implementation detail)

### 11.1 Magic links

Tokens are 256-bit random, base64url, stored as a SHA-256 hash, expire in 15 minutes, and are single-use via an atomic update. Cross-device use is allowed (request on laptop, open on phone). Email copy is matter-of-fact: "Sign in to Pocket Aviary. This link expires in 15 minutes."

### 11.2 Sessions

Opaque 256-bit tokens, stored hashed. Device label comes from UA parsing (browser + OS family only). `last_seen_at` is updated at most every 10 min, and the settings panel shows it as "today / this week / earlier". Revocation deletes the token hash, and the API's session cache TTL is ≤ 30 s.

### 11.3 Encryption and PII

Emails (account and visitor) use AES-256-GCM envelope encryption with KMS-managed keys. Blind indexes use a separate HMAC key. Structured logging goes through a field **allowlist**, so unknown fields are dropped. A CI test greps service logs from the E2E suite for email patterns and bird names and fails on any hit. A sink-side PII scanner alarms in production. Error reports (Sentry-style, self-hosted) scrub request bodies and include only the account UUID.

### 11.4 Export

The jobs worker builds a JSON file with: `schema_version`, account (email, created_at, settings, timezone), birds (id, name, species, adopted_at, **personality vector**, current mood), notebook entries, visit invitations and visit log. It does **not** include presence history, session history, heartbeat logs, or interaction events (D-4). The file is stored in object storage for 7 days. The email links to `/account/exports/:id` (sign-in required). Rate limit: 3 exports per day.

### 11.5 Deletion

`POST delete` sets `status = pending_deletion` and `deletion_requested_at`. Effects:
- The aviary keeps ticking (a restore finds living birds).
- Visits are suspended (visitor snapshot → 410).
- Every signed-in page shows a matter-of-fact strip in the top bar region: "This account will be deleted on October 24. [I changed my mind]".
- Restore flips the status back.

At day 30 a jobs worker hard-deletes in one transaction per table group: events, observations, notebook, personality (plus the daily restore points), birds, visits, exports (and objects), sessions, then the account. It writes a UUID-only tombstone to the security audit trail for 30 days. Backups expire at 35 days, so deleted data leaves backups within 35 days of hard delete; the privacy policy says so. RUM and metrics hold no account identifiers, so nothing there needs deleting. Logs age out at 30 days.

### 11.6 Privacy boundary as architecture

- The simulation DB sits in a private network segment. Only `api`, `tick`, `naturalist`, and `jobs` roles have credentials. The analytics/metrics stack has **no network route and no credentials** to it (Terraform-enforced; a policy test runs in CI).
- Services emit metrics through a metrics library with a **schema registry**. Metric label keys come from an allowlist (`route`, `status`, `browser_family`, `device_class`, `region`, `build`, `kind`). `account_id`, `bird_id`, `species`, `mood`, and trait names are forbidden label keys, and the registry rejects them at compile time.
- No ML training pipeline exists. If one is ever proposed, the rule is written down in advance: it receives no per-bird fields.

### 11.7 Settings storage

`accounts.settings` jsonb: `{motion, captions, caption_size, narration, narration_text, sound, shortcuts, keymap, topbar_pinned, visit_notifications: false}`.

---

## 12. Product surfaces and voice split

### 12.1 Surface inventory

| Surface | Voice | Notes |
|---|---|---|
| Aviary scene | (no text) | Birds and place only |
| Top bar icons + tooltips/labels | Naturalist for **offer**, **Field Notebook**; matter-of-fact for **Settings**, **Accessibility** | 4 icons only |
| Offer panel | Naturalist ("a seed", "a song fragment", "a still pool", "settle the aviary") | Settle lives here (D-5) |
| Field notebook | Naturalist | Read-only |
| Narration, captions | Naturalist | |
| Adoption / starter naming | Naturalist ("two birds have found the aviary. what will you call them?") with a matter-of-fact name field label | Suggested names are pre-filled |
| Sign-in, sessions, email change, export, deletion, errors, unsupported browser | Matter-of-fact | |
| Account settings → Birds (rename, newcomer) | Matter-of-fact labels ("Name", "Let it stay", "Not now") | System surface |
| Accessibility settings | Matter-of-fact | |
| Visit invitations, visit log, visitor "unavailable" | Matter-of-fact ("This visit is no longer available.") | |
| Visit notification email (opt-in) | Matter-of-fact ("Someone you invited visited your aviary: ada@example.com, today.") | Max 1 per visitor per day |

### 12.2 Anti-announcement enforcement

- ESLint rules ban imports of any toast, snackbar, or notification component, `Notification`, `PushManager`, `serviceWorkerRegistration.showNotification`, and `navigator.setAppBadge`.
- Copy review is a required PR label for any string change in product surfaces.
- A "welcome-back" canary test: E2E sign-in after simulated 1 h, 1 day, and 14 day absences asserts that no new text nodes appear outside the top bar, panels, and captions within 10 s of load.

### 12.3 New-account flow

Magic link → consume → the naming view appears over the quiet field (sky only). The two starter species are shown as small still portraits with suggested names. The user can edit or keep them, then continues. The naming view closes (these clicks also unlock audio). The quiet field stays empty for 0.8–1.5 s, then the first bird flies in to its perch, calling once. The second follows 2–5 s later. After this, the empty aviary never appears again.

---

## 13. Performance budgets and observability

### 13.1 Budgets

| Budget | Target | Enforcement |
|---|---|---|
| Initial JS (all eager) | ≤ 400 KB gz internal; **< 2 MB gz** PRD hard cap | `size-limit` in CI (fail) |
| Critical path (HTML+inline+critical JS+species) | ≤ 140 KB gz | CI fail |
| First bird visible | p75 **< 500 ms** mid-tier mobile over 4G | Lab (WebPageTest private instance, real devices) per release; RUM p75 alarm at 450 ms |
| Idle frame rate | 60 fps; p95 frame ≤ 16.7 ms over 30 min on 5-yr-old laptop | Device-lab soak per release; RUM long-frame ratio |
| Main-thread per frame | ≤ 6 ms p95 | Perf test with tracing |
| Memory | Heap slope ≈ 0 over 30 min after 3-min warmup (< 50 KB/min); DOM nodes constant ±2%; AudioNode count constant | **CI**: nightly 30-min soak in headless Chrome + Firefox via CDP/`measureUserAgentSpecificMemory` (site is cross-origin isolated: COOP/COEP); 5-min variant on every PR |
| Snapshot size | ≤ 6 KB gz at 7 birds | Protocol test |
| Snapshot latency | p95 ≤ 80 ms from edge | Server metric |
| Tick latency | p50 < 20 ms, p99 < 500 ms; **alarm at p99 > 5 s** (PRD) | Server metric + pager |
| Tick lag (now − last_tick_at) | p99 < 90 s | Alarm at > 3 min for > 1% of aviaries |

**Memory discipline rules** (code review checklist plus lint):
- No closures allocated in the frame loop.
- Particle and caption pools are preallocated.
- Typed arrays for the animation state.
- Notebook virtualization.
- The event outbox is bounded (≤ 500 events; oldest heartbeats dropped first).
- One `AudioContext` and one worklet for the page's lifetime.
- No workers other than the audio worklet and (optionally) the service worker.

### 13.2 Capacity

At 100k aviaries: about 1,700 ticks/s × < 1 ms CPU → about 2 vCPU for compute. Writes: about 1,700 tx/s at peak with dormant aviaries' state flushed every 5 ticks, so under 600/s typical. One Postgres primary (8 vCPU) plus a sync replica handles it. At 1M aviaries: 1024 shards across roughly 12 tick pods, Postgres partitioned by `hash(account_id)` into 4 clusters via a shard map, and a Redis snapshot cache cluster. The design scales horizontally without protocol changes. The event table is the heaviest writer (heartbeats). The mitigation is that only the leader tab heartbeats, every 20 s, and only while present.

### 13.3 What we measure (aggregate only, from day one)

- **RUM** (a first-party endpoint, beaconed at session end or `pagehide`; no account id, no session id, no cross-event join key): `first_bird_visible` histogram, TTFB, long-frame ratio, p95 frame time bucket, audio-unlocked-within-60 s (boolean bucket), `audio_pipeline_error{kind}`, WebGL/Canvas fallback usage, reduced-motion active (boolean), session-duration histogram bucket. Dimensions: `browser_family`, `device_class`, coarse region, `build`.
- **Server:** request rate, latency, and errors per route; tick latency, lag, and invariant violations; event ingest and rejects by kind; presence heartbeat rejects (implausible claims); snapshot cache hit ratio; magic-link send and consume rates plus email-provider bounces; export and deletion job health.
- **Synthetic:** a headless-browser fleet in 6 geographies every 5 min against dedicated synthetic accounts, checking first-bird time, snapshot freshness, audio graph construction, and E2E sign-in with a test inbox.

### 13.4 What we deliberately do not measure

We don't measure DAU/WAU/MAU per account, retention cohorts, funnels, visit frequency, streak-like distributions, per-account engagement, population drift distributions ("average drift across accounts"), mood distributions, offer popularity, listen-in counts, or notebook open rates. We run no A/B tests on engagement. Capacity planning uses total aviary count and request rates only. Drift calibration uses `drift-lab` plus the consented calibration panel in a separate environment (§14.2), **never production data**.

---

## 14. Rollout

### 14.1 Environments

`dev` → `staging` (production-like, synthetic data, **time-travel** clock injection for aviary age and tick acceleration) → `panel` (a separate production-grade environment and database for the consented calibration panel) → `prod`.

### 14.2 Phases

| Phase | When | Who | Exit criteria |
|---|---|---|---|
| **0. Spikes** | wk 0–6 | team | Renderer spike meets frame budgets on the reference laptop; first-bird ≤ 400 ms on the reference phone; audio synth prototype passes the 7-bird recognizability pilot (§16 R-3.1); `engine-core` + `drift-lab` skeleton hits persona targets in simulation |
| **1. Vertical slice** | wk 6–14 | internal | Sign-in → adoption → aviary → greeting → listen-in → offer → settle → notebook, end to end, with narration, captions, and RM present (not polished) |
| **2. Feature complete** | wk 14–22 | internal dogfood (staging) | All §1.1 features; perf, memory, and a11y CI gates green; security review; privacy architecture review (IAM/network tests) |
| **3. Calibration panel beta** | wk 20–30 (**≥ 8 weeks**; must begin ≥ 8 weeks before GA to observe the 3-week target twice) | about 200 recruited participants, informed consent, paid, incl. ≥ 6 disabled testers | Week-1 survey: most report "about the same" (drift not user-visible); week-3–4: most report a bird "seems different than at first" without being told; no reports of distress or guilt after absences; audio "alive vs. canned" ratings ≥ target; a11y panel sign-off. Panel data is inspected **only in the panel environment** and deleted after the study. |
| **4. GA ramp** | wk 30+ | public | Waitlist ramp 1k → 10k → 50k → open, each step gated on tick p99, snapshot p95, first-bird p75, error rates, and email deliverability (magic links) |

### 14.3 Ramping birds per aviary

- Every aviary starts with 2 (hard rule). Arrivals depend on age, so **production will not see 3-bird aviaries until about 90 days after GA**, and 7-bird aviaries until about 18 months. The 3–7-bird paths therefore need proving before then:
  - **Staging time-travel:** automated aviaries aged to 7 birds, then run through perf soak, recognizability tests, layout tests, and narration tests at every count from 2 to 7.
  - **Panel:** consenting panel aviaries run on an accelerated arrival schedule (weeks instead of months) to exercise arrival and adoption UX and 3–5 bird mixes with real people.
- **Server flags:** `arrivals.enabled` (global kill switch; pausing delays arrivals and loses nothing) and `arrivals.max_count` (starts at 7). If recognizability or perf problems show up at high counts, cap at 5 while fixing, which delays arrivals but never removes a bird. Birds are never removed.
- Other kill switches: `weather.enabled`, `notebook.generation.enabled`, `narration.idle_cadence_s`, `audio.worklet.enabled` (forces silence + captions), `greeting.max_extra_greeters`.

### 14.4 Engine parameter changes after launch

Parameters are versioned files, reviewed by the engine owner. Changes apply forward only, with a `drift-lab` report attached to the PR. Personality vectors are never migrated or recomputed. Changing a drift rate in production is treated like a database migration: staged to staging, then panel, then prod.

### 14.5 Day-one instrumentation checklist

Everything in §13.3; synthetic checks live; tick lag/latency/invariant alarms paging; PII log scanner alarms; email deliverability dashboard; snapshot cache hit ratio; a restore runbook drill completed (restoring a single bird's vector from daily restore points and PITR).

---

## 15. Workstreams and team

| Workstream | Owner(s) | Key deliverables |
|---|---|---|
| Engine | 2 eng | `engine-core`, tick service, `drift-lab`, invariants, restore tooling |
| Client scene & animation | 2 eng + 1 animation designer | Renderer, puppets, behavior machine, RM presenter + pose library, layout solver |
| Audio | 1 eng + 1 sound designer | Worklet synth, motif libraries, voiceprints, mixer, recognizability studies |
| Voice | 1 writer + 0.5 eng | Grammar, lexicons, notebook, narration, captions, copy lint |
| Platform/backend | 2 eng | API, auth, visits, jobs, edge, Postgres/Redis, IaC, privacy boundary |
| Accessibility | 1 eng (embedded across client) + external a11y consultant | Keyboard model, proxies, live region, audits, disabled-tester panel |
| Perf & observability | 0.5 eng | Budgets, CI gates, device lab, RUM, synthetic fleet |
| Design | 1 visual designer | Palette/contrast spec, species art direction, focus ring, panels |

Accessibility, RM, captions, and narration have tickets in **every** milestone from phase 1. They are not a phase of their own.

---

## 16. Risks

### R-1 Drift calibration

| # | Risk | Mitigation |
|---|---|---|
| 1.1 | Drift too fast: users see change between sessions, and it feels like a Tamagotchi | Hard daily budget (0.005/trait/day) makes visible change in < 16 days impossible; `drift-lab` `binger` persona; panel week-1 survey |
| 1.2 | Drift too slow: it feels like a screensaver | Panel week-3/4 survey; JND validated perceptually; trait→render mappings tuned so 0.08 is perceivable. Mappings can be retuned without touching vectors (render-side), which is the preferred lever. |
| 1.3 | Long-term homogenization (all birds maxed and identical after a year) | Per-bird ceilings, headroom scaling, channel-specific weights so different attention patterns shape different traits, immutable voiceprints and markings; `year` persona assertion |
| 1.4 | Production can't be observed for calibration (privacy) | Accepted as a trade-off. Mitigated by drift-lab, a panel environment with consent, and structural caps that bound the worst case regardless of calibration |
| 1.5 | Presence inflation (lax detection, multi-tab, spoofed clients) | Three-signal conjunction; server clamps to wall-clock; cross-device union; leader-tab heartbeats; aggregate implausibility counters. A spoofing client can only move its own birds, capped by the daily budget. |
| 1.6 | "Ambient" state after absence misread as sadness | Envelope floor 0.35; envelope decoupled from valence; absence-independence property test; panel interviews on the returning experience |

### R-2 Sync correctness

| # | Risk | Mitigation |
|---|---|---|
| 2.1 | Lost events (sequence gaps) | Per-account row-locked sequence; cursor committed atomically with state; chaos tests killing workers mid-tick; replay-equality tests |
| 2.2 | Double processing / split-brain ticks | Lease fencing + optimistic `state_version` check; deterministic pure step |
| 2.3 | Personality loss or reset (worst failure) | Single writer role; monotonic trigger; daily restore points; PITR; immutable `bird_id`/voiceprint triggers; migration lint forbidding writes to `bird_personality` outside the tick role; quarterly restore drill |
| 2.4 | Clock skew between devices → mismatched calls or scripts | Server-time scripts; median-filtered offsets; tolerance of ±100 ms is inaudible for ambient calls |
| 2.5 | Timezone ambiguity (travel, two devices in two zones) | Canonical account timezone follows the most recent presence-bearing device; lighting eases over 30 min on change (D-6) |
| 2.6 | Stale-frame "snap" on tab return | Retained script, 250 ms bounded stale hold, flight-based reconciliation |
| 2.7 | Visitor traffic influencing the host | Separate credentials, endpoints, and tables; the tick never reads visit tables; tests assert visitor calls cannot reach `/v1/aviary/*` |

### R-3 Audio uncanniness and audio UX

| # | Risk | Mitigation |
|---|---|---|
| 3.1 | Synth calls sound toy-like, alarm-like, or robotic | Sound designer ownership; reference-listening sessions; anti-uncanny rules (§8.2); panel "alive vs. canned" rating gate before GA |
| 3.2 | Seven birds not individually recognizable | Voiceprint design maximizes separation (pitch band + timbre + signature interval + rhythm); ABX/identification study at 2, 5, and 7 birds with a target of ≥ 80% correct identification after a short familiarization; an objective MFCC-classifier proxy in CI to catch voiceprint collisions at adoption (re-roll the voiceprint at creation if too close to an existing bird's; never after) |
| 3.3 | Listening fatigue over 30 min | Loudness ceiling, rate caps at high vocal_freq, dusk/night reductions, chorus gain rider |
| 3.4 | Autoplay lock means many sessions are silent | Gesture-unlock design; unlock during adoption; aggregate unlock-rate metric; revisit if < 50% of sessions unlock within 60 s |
| 3.5 | AudioWorklet glitches on low-end devices | Voice cap (24); `latencyHint: 'playback'`; underrun counter; the kill switch falls back to silence + captions |

### R-4 Accessibility regressions

| # | Risk | Mitigation |
|---|---|---|
| 4.1 | New animations ship without RM variants | Every animation PR must add RM poses (lint: behavior modes without an RM pose set fail the build); RM screenshot baselines |
| 4.2 | Narration becomes a state list or too chatty | Copy lint; cadence tests; repetition guard; SR-user panel |
| 4.3 | Live-region behavior differs across screen readers | Manual matrix gate per release; single atomic polite region (the most consistent pattern) |
| 4.4 | Focus lost when birds move or panels close | Identity-bound focus; focus-return tests; proxies never removed while focused (a departing newcomer waits until unfocused) |
| 4.5 | Caption contrast fails over bright or dark scene regions | Adaptive pill with measured contrast; frame-sampled contrast tests |
| 4.6 | Keyboard shortcuts collide with AT | Scoped to aviary focus; can be turned off or remapped |

### R-5 Product-principle erosion

| # | Risk | Mitigation |
|---|---|---|
| 5.1 | "Just one" toast, counter, or streak creeps in | Lint bans; welcome-back canary test; copy-review label; §0 invariants in the PR template |
| 5.2 | Notebook becomes repetitive after months | Large grammar; 30-entry n-gram guard; sparsity; the writer continues extending lexicons after launch as a deploy, not a migration |
| 5.3 | Telemetry scope creep ("just one engagement metric") | Metric schema registry with forbidden labels; the analytics stack structurally cannot reach the simulation DB |

### R-6 Performance and platform

| # | Risk | Mitigation |
|---|---|---|
| 6.1 | 500 ms first bird missed on real 4G | Edge-inlined snapshot, tiny critical path, SW caching, HTTP/3; SVG first-frame contingency |
| 6.2 | Canvas2D can't hold 60 fps at high DPR | WebGL2 renderer behind the same interface; quality governor |
| 6.3 | Magic-link email deliverability (sign-in depends on it) | Reputable transactional provider with dedicated IP warm-up; SPF/DKIM/DMARC; deliverability dashboard; fragment-token callback defeats link scanners |
| 6.4 | Tick cost at scale | Sharding, batched flushes for dormant aviaries, closed-form integrators |

---

## 17. Decisions made where the PRD is ambiguous or in tension

| ID | Question | Decision | Why |
|---|---|---|---|
| D-1 | "Pointermove or keypress" on touch devices (taps don't always emit pointermove) | Activity = `pointermove`, `pointerdown`, `keydown`. Activity window starts at **5 min**; calibrate in the 3–10 min range, leaning long | Taps are pointer activity. The PRD leans long because watching without moving is the product. Visibility + focus remain mandatory. |
| D-2 | Listen-in by "keyboard-focusing" vs "Enter triggers listen-in" | Arrow focus moves focus (and ends a listen-in on the previous bird); `Enter` engages | Engaging on every arrow press would sweep the mix around while navigating, which feels like channel switching and conflicts with the slow-ramp intent. |
| D-3 | Export includes "current personality vectors" vs "never exposed numerically" | The export file includes vectors; no product UI, debug view, ARIA, or snapshot does. The export panel does not preview contents. | The export is an explicit data-portability feature named in the PRD, and the no-exposure rule targets product surfaces. A flag allows switching to an opaque encoded field if the product owner decides otherwise. |
| D-4 | What the export excludes | No presence history, session history, heartbeat logs, or interaction events | The PRD bans "an exportable visit log" as a streak disguise. |
| D-5 | Settle is "from the top bar", but the top bar has "nothing else" beyond 4 icons | Settle is the last item in the offer affordance's menu ("settle the aviary"), plus the `s` shortcut | Both are gestures toward the aviary. This keeps the top bar at exactly 4 icons. |
| D-6 | "Local time" with multiple devices in different zones | One canonical account timezone, updated from the most recent presence-bearing device; lighting and mood ease over 30 min when it changes; visitors see the host's time | The PRD requires every device (and visitor) to show the same aviary in the same mood. |
| D-7 | Is "settled" canonical or per-device? | Per-device session state; the server receives only a mood-quieting `settle` event | Settle is a goodbye from this session; the other device shouldn't dim. |
| D-8 | Mood name for night sleep | `roosting` | Avoids a collision with the aviary-level "settled" term defined in `concepts`. |
| D-9 | "One-time link" for visits vs repeat visits in the log | The link is single-use to establish a visitor session on one browser. That session persists until revoked or 30 days idle. | Matches "one-time link", allows the visit log to show repeated visits, and gives abandoned access a natural end. |
| D-10 | How a new-bird "offer appears in the user's flow" without announcing | A newcomer bird appears in the scene; the notebook may note it; adoption is in Settings → Birds without a badge. It leaves quietly after 30 days if not welcomed. | Noticing, not announcing. The user chooses without being prompted. |
| D-11 | Offer cooldown UI | No disabled state or countdown; birds in cooldown glance but don't engage | Keeps the offer a gesture, not a button with a timer. |
| D-12 | "Whether you mute the calls" as a drift input | Unmuted presence feeds a small `heard` channel into vocal frequency; muting never subtracts | Honors the brief without punishing muting. |
| D-13 | Where mute lives (not a top-bar icon) | Accessibility settings + `m` shortcut | The top bar is capped at 4 icons. |
| D-14 | Do visitors get notebook, listen-in, captions, narration? | Visitors get captions and narration (local a11y); no notebook, no listen-in, no bird focus. Their top bar has a single icon (accessibility settings). | The PRD makes visitors render-only; a11y remains a right. |
| D-15 | Visit notifications vs "does not email the user about the aviary" | The opt-in, off-by-default visit email is the single named exception | Both rules come from the PRD; the specific rule wins. |
| D-16 | Seasonal daylight without location | Month + hemisphere inferred from the IANA timezone; ±45 min | No location collection; good enough for a felt day. |

---

## 18. Acceptance checklist for v1 GA

- [ ] All §0 invariant checks exist in CI and pass.
- [ ] `drift-lab` persona suite passes; panel surveys meet the week-1 and week-3/4 targets.
- [ ] First-bird p75 < 500 ms (lab, mid-tier Android, 4G); returning p75 < 300 ms.
- [ ] 60 fps soak for 30 min on the reference laptop; memory slope test green (nightly, 3 consecutive nights).
- [ ] Bundle < 2 MB gz (actual ≤ 400 KB).
- [ ] Recognizability study ≥ 80% at 7 birds.
- [ ] Manual SR matrix passes; disabled-tester panel sign-off, covering "feels alive" and not only operability; RM baselines approved by design.
- [ ] Welcome-back canary, copy lint, and banned-import lint green.
- [ ] Privacy architecture tests (IAM/network) green; PII log scan clean on 7 days of staging logs.
- [ ] Restore drill completed; tick p99 alarm verified by fault injection.
- [ ] Unsupported-browser page and WebAudio silence+captions fallback verified on each supported browser.
