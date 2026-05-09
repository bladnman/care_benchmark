# Pocket Aviary — Implementation Plan (v1)

This plan translates the PRD into an executable specification for an engineering team building Pocket Aviary v1. It does not restate the PRD; it commits to specific architectural shapes, schemas, protocols, calibrations, file layouts, error envelopes, and rollout steps that satisfy the PRD's principles. Where the PRD leaves a calibration "to be tuned during build," this plan picks a defensible starting value and names the metric that will trigger re-calibration.

The plan is organized so an engineer can scroll to a single section and have everything they need to implement that surface without cross-referencing the PRD.

---

## 1. Scope (v1)

### In scope

- Single-user accounts; magic-link email auth; per-device session tokens.
- One canonical aviary per account, 2 starter birds at adoption, capped at 7.
- Six-species bird pool; user-assigned, renameable bird names; stable bird identifier independent of name/species.
- Server-side simulation tick (~60s cadence) that owns canonical personality vector, mood, perch position, call timing.
- Procedural call synthesis client-side via WebAudio; no recorded audio anywhere.
- Three-perch single-screen scene; idle micro-motion; ambient leaf/feather drift; subtle parallax; day/night palette anchored to user local time; rare ambient weather (rain, wind).
- Listen-in interaction with slow audio mix ramps.
- Three offer types: seed, song fragment, still pool. Per-bird, per-offer-type cooldown.
- Settle gesture with 5s undo; tab-close equivalence at engine level.
- Field notebook: read-only, sparse, naturalist-prose entries written by an entry-generation pass on the simulation tick.
- Return-greeting computed server-side from absence length, current personality and mood, and bird's stable ordering on the screen.
- Visit invitations: per-invite opt-in, read-only ambient view, 30-day expiration, revocable, silent visit log.
- Account export (JSON, on-demand, emailed link).
- Soft account deletion with 30-day hard-delete window.
- Accessibility: screen-reader narration in naturalist voice, reduced-motion as a designed surface (cross-fades), procedural call captions, keyboard navigation, WCAG AA contrast on user copy, visible focus indicators.
- Aggregate-only operational telemetry. Privacy boundary enforced at the data-pipeline layer.
- Browser support: latest two majors of Chrome, Safari, Firefox, Edge.

### Out of scope (explicit, per `non_goals.md`)

- Native mobile apps, gamification of any flavor (achievements, streaks, levels, badges, XP, "days visited", green-dot calendar, milestone celebrations), Tamagotchi-style mechanics (death, hunger, distress, decaying happiness), social-network surfaces beyond the read-only visit (profiles, follows, discovery, leaderboards, comments, mutual visits), push notifications, native payment flows, shared/multi-user aviaries, customizable scenes, payments.
- Co-presence during visits.
- Server-derived recommendation features. Cross-account aggregation of per-bird interaction state.
- Recorded audio fallback path. WebAudio-unavailable users get silence + captions.

### Calibration ranges committed by this plan (with rationale)

These are the numbers a separate engineering team would otherwise pick at random; pinning them here prevents drift across surfaces and gives the test harness fixed targets.

| Parameter | Value | Rationale |
|---|---|---|
| Simulation tick cadence | 60 seconds | Slow enough for "feels alive without being twitchy"; fast enough to land mood transitions before user notices a snap. |
| Personality vector dimensionality | 5 traits (boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity), each `float64 ∈ [0.0, 1.0]` | Matches `bird_engine.md` enumeration. Floats clamped at sim layer; never exposed. |
| Drift time-constant (low-pass τ) | 7 days | "Measurable drift in instruments at ~1 week, visible to user at ~3 weeks" implies τ ≈ one week; 3 τ ≈ 21 days for the visible threshold. |
| Drift cap per tick | 0.0005 per trait per tick | At 60s cadence, 1440 ticks/day ⇒ max ~0.72/day. Real drift at moderate presence is ~0.005–0.02/day. |
| Presence activity window | 4 minutes | Long enough to absorb watching-without-mouse; short enough to drop within a session of inactivity. |
| Presence ping cadence | 30 seconds | Two pings per tick; gives the tick room to detect attention without flooding the event log. |
| Per-offer cooldown | 4 minutes per (bird × offer_type) | Long enough that an offer is a gesture; short enough that a 10-minute session can include more than one. |
| Notebook entry rate cap | ≤1 entry per 24h per aviary at idle, soft-cap of 3 entries per 24h on noteworthy days | Preserves sparsity even on heavy-usage days. |
| First-bird-render budget | <500ms p95 on mid-tier mobile/4G | Affective threshold per `accessibility_perf.md`. |
| Simulation tick latency p99 | <5s | Alarm threshold per `accessibility_perf.md`. |
| Magic-link TTL | 15 minutes | Per spec. |
| Session token TTL | 30 days, sliding | Long-lived enough to match the rhythm of a relationship; revocable per-device. |
| Visit invitation TTL | 30 days | Per spec. |

When the PRD says "to be calibrated during build," this plan binds the value above and the calibration metric (e.g., drift τ is recalibrated by the user-study group looking at "did three weeks feel right?", not by an A/B test on engagement).

---

## 2. Architecture

### Service shape (six services + one CDN edge)

All services are independently deployable. The bus between them is a single ordered event log for per-account state changes (interaction events, simulation outputs) and a separate aggregate-telemetry pipeline that is *air-gapped* from the per-account event log at the data-pipeline layer.

```
                    ┌────────────────────────────────────┐
                    │     CDN edge (HTML + initial JS    │
                    │     bundle + first-snapshot blob)  │
                    └──────────────┬─────────────────────┘
                                   │
            ┌──────────────────────┼─────────────────────┐
            │                      │                     │
   ┌────────▼────────┐    ┌────────▼────────┐  ┌─────────▼────────┐
   │   API Gateway   │    │  Snapshot CDN   │  │  Static assets   │
   │  (REST + WS)    │    │  (signed URLs)  │  │   (SVGs/sprites) │
   └───┬─────────┬───┘    └────────┬────────┘  └──────────────────┘
       │         │                 │
       │         │                 │
┌──────▼──┐  ┌───▼─────┐    ┌──────▼──────┐    ┌─────────────────┐
│ Auth    │  │ Aviary  │    │  Snapshot   │    │  Notebook       │
│ Service │  │ API     │────┤  Service    │    │  Service        │
└────┬────┘  └────┬────┘    │  (reads     │    │  (reads sim     │
     │            │         │   sim DB)   │    │   DB; writes    │
     │            │         └─────────────┘    │   notebook DB)  │
     │            │                            └─────────────────┘
     │            │
     │            ▼
     │      ┌──────────────────────────────┐
     │      │   Event Log (Kafka/Pulsar,   │
     │      │   keyed by account UUID)     │
     │      └────────────────┬─────────────┘
     │                       │
     │        ┌──────────────▼───────────────┐
     │        │     Simulation Service       │
     │        │  (consumes events, runs      │
     │        │   the per-account tick,      │
     │        │   writes canonical state)    │
     │        └──────────────┬───────────────┘
     │                       │
     ▼                       ▼
┌────────────────┐   ┌────────────────────────┐
│ Account DB     │   │  Simulation DB         │
│ (Postgres,     │   │ (Postgres + per-       │
│  encrypted     │   │  account row,          │
│  email)        │   │  jsonb canonical state)│
└────────────────┘   └────────────────────────┘

         ┌────────────────────────────────────────┐
         │  Telemetry Pipeline (separate; never   │
         │  reads simulation DB, never receives   │
         │  per-account dimensions)               │
         └────────────────────────────────────────┘
```

### Boundaries

- **Auth Service** owns email + magic links + session tokens + per-device session list. It writes only to the Account DB. It produces no per-bird events.
- **Aviary API** is the user-facing REST + WebSocket gateway. It accepts interaction events (offer, listen-in start/end, settle, presence ping), validates them, writes them to the event log, and returns acknowledgment. It serves snapshot reads via the Snapshot Service.
- **Simulation Service** is the only writer of canonical aviary state. It consumes the event log in account-keyed order, runs the tick, and writes new state to the Simulation DB. It also emits "noteworthy moment" events to the Notebook Service.
- **Snapshot Service** is read-only over Simulation DB. It serves the canonical state snapshot to clients and to visit-readers. It is the only path clients use to read state. It returns small JSON blobs (target ~4–8KB compressed).
- **Notebook Service** owns the field notebook. It consumes "noteworthy moment" events from the Simulation Service and runs the entry-generation logic (sparsity-respecting prose-from-template). It writes notebook entries to the Notebook DB.
- **CDN edge** serves the initial HTML, the JS bundle, and an inlined "first snapshot" placeholder so the aviary can render with no round trip when the user is logged in via cookie.
- **Telemetry pipeline** is a separately deployed pipeline that emits and aggregates request-level metrics. It is forbidden from reading Simulation DB or Notebook DB, and the data contract for what it sees is enforced via a separate IAM role and a schema linter in CI.

The privacy boundary is real, not policy: the Telemetry pipeline's IAM role does not have read access to Simulation DB or Notebook DB; the schema linter rejects metric definitions that include `account_uuid`, `bird_id`, or any field name in a denylist.

### Client/server split

- **Server owns:** personality vectors, mood, mood timers, perch state, call timing, drift, notebook entries, all canonical state. The simulation tick is the only writer of personality.
- **Client owns:** rendering, audio synthesis, presence-event emission, interaction event submission, ambient ornamentation (leaves, feathers — not part of simulation), idle micro-motion interpolation between snapshots.
- **Client never owns:** any decision that affects future state. The client never decides "this listen-in changes Pip's social warmth by 0.01"; it sends the listen-in event and the server decides on the next tick.

### Render pipeline boundary

The client receives a snapshot of the canonical state. From that snapshot, it composes the scene. Between snapshots, it interpolates on the client side using the snapshot's `next_action_at` hints (e.g., "Pip will move to front perch over the next 12 seconds with this easing"). Interpolation is local-only; the server doesn't care how the client rendered, only what events come back.

If the client misses a snapshot (network blip), it continues animating its current state until the next snapshot arrives. If a snapshot is dramatically different from the interpolated state (e.g., a long disconnection), the client cross-fades over 800ms rather than snapping.

---

## 3. Data model

All tables in Postgres. JSONB used for state blobs that are written atomically by the simulation tick. Separate databases per service domain (Account DB, Simulation DB, Notebook DB, Visit DB).

### 3.1 Account DB

```sql
CREATE TABLE accounts (
  account_uuid          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email_encrypted       BYTEA NOT NULL,        -- AES-GCM via KMS-managed key
  email_search_hash     BYTEA NOT NULL UNIQUE, -- HMAC-SHA256(email, server_pepper)
  created_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
  deletion_marked_at    TIMESTAMPTZ,           -- set on soft-delete; NULL = active
  email_change_pending  JSONB,                 -- { new_email_hash, verification_token, requested_at }
  visit_notify_enabled  BOOLEAN NOT NULL DEFAULT FALSE,
  reduced_motion_pref   BOOLEAN NOT NULL DEFAULT FALSE,
  captions_enabled      BOOLEAN NOT NULL DEFAULT FALSE,
  audio_enabled         BOOLEAN NOT NULL DEFAULT TRUE
);

CREATE TABLE sessions (
  session_token_hash    BYTEA PRIMARY KEY,     -- HMAC of token; raw token only in cookie
  account_uuid          UUID NOT NULL REFERENCES accounts(account_uuid),
  device_label          TEXT,                  -- "Chrome on macOS" — derived from UA, user-editable
  created_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
  last_seen_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at            TIMESTAMPTZ NOT NULL,
  revoked_at            TIMESTAMPTZ
);
CREATE INDEX ON sessions (account_uuid, expires_at);

CREATE TABLE magic_links (
  token_hash            BYTEA PRIMARY KEY,
  email_search_hash     BYTEA NOT NULL,        -- so we can rate-limit per email without storing it
  issued_at             TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at            TIMESTAMPTZ NOT NULL,
  consumed_at           TIMESTAMPTZ
);
```

**Critical rule, per `accounts_sync.md`:** every other table in the system uses `account_uuid` as the partition key. Email is never used as a foreign key, never used as a logging dimension, never used as an analytics dimension. The Auth Service is the only service that ever decrypts `email_encrypted`. This is enforced by:

1. Database-level revoke: only the Auth Service's DB role has `SELECT email_encrypted` permission.
2. Code linter: any new metric or log statement that includes a field matching `email`, `mail`, `address` is rejected at PR.
3. Ops: KMS audit log captures every email decryption and is reviewed monthly.

### 3.2 Simulation DB

The canonical state per account lives in two tables: `aviaries` (slow-changing) and `aviary_state` (fast-changing, updated each tick). Splitting them prevents the tick's hot writes from touching cold rows that get backed up in account export.

```sql
CREATE TABLE aviaries (
  account_uuid          UUID PRIMARY KEY,
  created_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
  starter_birds_chosen_at TIMESTAMPTZ,         -- set when adoption flow completes
  age_seconds           BIGINT NOT NULL DEFAULT 0,  -- recomputed lazily; not used as primary clock
  next_offer_eligible_at TIMESTAMPTZ          -- when the next "third bird offered" is eligible
);

CREATE TABLE birds (
  bird_id               UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  account_uuid          UUID NOT NULL REFERENCES aviaries(account_uuid),
  species_id            TEXT NOT NULL,         -- 'warbler', 'sparrow', etc.
  name                  TEXT NOT NULL,
  display_order         INTEGER NOT NULL,      -- left-to-right placement seed
  adopted_at            TIMESTAMPTZ NOT NULL DEFAULT now(),

  -- Personality vector (HIDDEN from API responses; serialized only to Account Export)
  trait_boldness        DOUBLE PRECISION NOT NULL,
  trait_social_warmth   DOUBLE PRECISION NOT NULL,
  trait_vocal_frequency DOUBLE PRECISION NOT NULL,
  trait_plumage_sat     DOUBLE PRECISION NOT NULL,
  trait_curiosity       DOUBLE PRECISION NOT NULL,

  -- Mood (resets daily-ish, modulated by tick)
  mood                  TEXT NOT NULL,         -- 'wary'|'content'|'curious'|'drowsy'|'alert'
  mood_started_at       TIMESTAMPTZ NOT NULL,

  -- Last-known render-influencing state, written by the tick
  current_perch         TEXT NOT NULL,         -- 'front'|'middle'|'back'
  perch_position_x      REAL NOT NULL,         -- 0..1 along the perch
  next_call_at          TIMESTAMPTZ,           -- when this bird will next vocalize
  next_motion_event     JSONB,                 -- { kind, target, ease_ms, started_at }
  call_grammar_seed     BIGINT NOT NULL        -- per-bird random seed for call-grammar choices
);
CREATE INDEX ON birds (account_uuid);

CREATE TABLE aviary_state (
  account_uuid          UUID PRIMARY KEY REFERENCES aviaries(account_uuid),
  state_blob            JSONB NOT NULL,        -- full snapshot; written by tick
  state_version         BIGINT NOT NULL,       -- monotonic per account
  updated_at            TIMESTAMPTZ NOT NULL,
  current_weather       JSONB,                 -- { kind: 'rain'|'wind'|null, started_at, ends_at }
  daynight_phase        TEXT NOT NULL          -- 'morning'|'midday'|'afternoon'|'evening'|'night'
);

CREATE TABLE event_log (
  event_id              BIGSERIAL PRIMARY KEY,
  account_uuid          UUID NOT NULL,
  event_kind            TEXT NOT NULL,         -- 'presence_ping'|'listen_in_start'|'listen_in_end'
                                                -- |'offer'|'settle'|'settle_undo'|'name_change'|'visit_pull'
  event_payload         JSONB NOT NULL,
  client_event_id       UUID NOT NULL,         -- idempotency key
  occurred_at           TIMESTAMPTZ NOT NULL,
  received_at           TIMESTAMPTZ NOT NULL DEFAULT now(),
  consumed_by_tick_at   TIMESTAMPTZ,
  UNIQUE (account_uuid, client_event_id)
);
CREATE INDEX ON event_log (account_uuid, event_id);
CREATE INDEX ON event_log (consumed_by_tick_at) WHERE consumed_by_tick_at IS NULL;

CREATE TABLE tick_records (
  tick_id               BIGSERIAL PRIMARY KEY,
  account_uuid          UUID NOT NULL,
  tick_at               TIMESTAMPTZ NOT NULL,
  consumed_event_count  INTEGER NOT NULL,
  drift_applied         JSONB,                 -- per-bird trait deltas; for debugging only
  duration_ms           INTEGER NOT NULL,
  PRIMARY KEY (tick_id),
  UNIQUE (account_uuid, tick_at)
);
```

**Why split state_blob and birds:** the state blob is the snapshot the client reads; the `birds` table is the queryable canonical record. The tick writes both atomically inside one transaction. The blob exists so that the Snapshot Service can serve a read with no joins.

### 3.3 Notebook DB

```sql
CREATE TABLE notebook_entries (
  entry_id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  account_uuid          UUID NOT NULL,
  occurred_on_date      DATE NOT NULL,         -- date in the user's local timezone, snapshotted
  prose                 TEXT NOT NULL,
  generated_at          TIMESTAMPTZ NOT NULL DEFAULT now(),
  trigger_reason        TEXT NOT NULL          -- 'first_greeter_change'|'long_quiet'|...
);
CREATE INDEX ON notebook_entries (account_uuid, occurred_on_date DESC);
```

### 3.4 Visit DB

```sql
CREATE TABLE invitations (
  invitation_id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  host_account_uuid     UUID NOT NULL,
  visitor_email_hash    BYTEA NOT NULL,        -- HMAC; we don't store visitor email plaintext on host side
  visitor_email_encrypted BYTEA NOT NULL,      -- needed to email the link
  link_token_hash       BYTEA NOT NULL,
  issued_at             TIMESTAMPTZ NOT NULL DEFAULT now(),
  expires_at            TIMESTAMPTZ NOT NULL,
  revoked_at            TIMESTAMPTZ,
  first_used_at         TIMESTAMPTZ,
  last_pulled_at        TIMESTAMPTZ
);
CREATE INDEX ON invitations (host_account_uuid);
CREATE INDEX ON invitations (link_token_hash);

CREATE TABLE visit_log (
  log_id                BIGSERIAL PRIMARY KEY,
  host_account_uuid     UUID NOT NULL,
  invitation_id         UUID NOT NULL,
  pulled_at             TIMESTAMPTZ NOT NULL,
  approx_duration_seconds INTEGER             -- bucketed at 60s granularity
);
```

The visitor's email is encrypted on the invitation row because the host needs to be shown which email an invitation was sent to. Visit pulls do not write to the host's `event_log` and never feed the simulation tick.

### 3.5 Personality vector serialization

The vector is stored as five `DOUBLE PRECISION` columns (not as a JSONB array) for two reasons: (a) cheaper indexed update by the tick; (b) explicit visibility to a code reviewer who can confirm at a glance no other table or service writes these columns. The simulation tick is the *only* SQL `UPDATE` statement in the entire codebase that touches `trait_*` columns. Enforced by a unit test that greps the codebase for `trait_` writes outside the tick module.

### 3.6 Account export schema

Generated on demand:

```json
{
  "schema_version": 1,
  "exported_at": "2026-05-08T12:00:00Z",
  "account": { "created_at": "...", "email": "...", "settings": { ... } },
  "aviary": { "created_at": "...", "current_weather": null, "daynight_phase": "evening" },
  "birds": [
    {
      "bird_id": "...",
      "name": "Pip",
      "species_id": "warbler",
      "adopted_at": "...",
      "personality": {
        "boldness": 0.42, "social_warmth": 0.58, "vocal_frequency": 0.31,
        "plumage_saturation": 0.65, "curiosity": 0.50
      },
      "mood": "content",
      "current_perch": "front"
    }
  ],
  "notebook_entries": [
    { "occurred_on_date": "2026-05-01", "prose": "pip greeted before wren today, first time this week." }
  ]
}
```

The export is the *only* place personality numbers appear to a user, and they appear because the user explicitly asked for the data. The product UI never shows them.

---

## 4. API surface

REST + JWT-bearer for the API (`/api/v1/...`), plus a single WebSocket channel per session for snapshot push. Visit pulls use a separate token-bearer endpoint that does not require a user account.

### 4.1 Auth endpoints

```
POST   /api/v1/auth/magic-link       { email }            -> 202
GET    /api/v1/auth/consume?token=…                      -> sets cookie, redirects to /
POST   /api/v1/auth/sign-out                             -> 204
GET    /api/v1/account/sessions                          -> list of devices
POST   /api/v1/account/sessions/:id/revoke               -> 204
```

Magic-link rate-limit: 5 per email per hour, 20 per IP per hour. Above limits return matter-of-fact "Too many requests; try again in a minute." in the response body.

### 4.2 Aviary read endpoints

```
GET  /api/v1/aviary/snapshot
     -> {
          state_version: 8123,
          fetched_at: "2026-05-08T12:00:00Z",
          server_time: "2026-05-08T12:00:00Z",
          daynight: { phase: "morning", local_offset_minutes: -240 },
          weather: null,
          birds: [
            {
              bird_id: "...",
              species_id: "warbler",
              name: "Pip",
              perch: "front",
              perch_position_x: 0.42,
              mood: "curious",
              next_call_at: "2026-05-08T12:00:08Z",
              next_call_descriptor: { motif_id: "warbler_3", duration_ms: 1200, pitch_bias: 0.04 },
              next_motion: { kind: "preen", duration_ms: 8000, started_at: "..." },
              greeting: { kind: "headtilt", duration_ms: 1400, started_at: "..." } | null,
              listen_in_focused: false
            }
          ],
          ambient: {
            wind_strength: 0.0,
            leaf_drift_rate_hint: 0.2
          }
        }
```

The greeting field is populated only on the first snapshot of a session and is computed server-side per Section 5.5 (return-greeting). This avoids the client deciding the greeting.

```
GET  /api/v1/aviary/snapshot?since=8123    -> 304 if no new state, else fresh blob
```

### 4.3 Aviary interaction endpoints

All event-write endpoints accept a `client_event_id` (UUID) for idempotency. The server's response is a small acknowledgment, not a state delta — the next snapshot pull surfaces the consequences.

```
POST /api/v1/aviary/events/presence_ping
     { client_event_id, occurred_at, visibility_state, has_focus, last_input_at }
     -> 202

POST /api/v1/aviary/events/listen_in
     { client_event_id, action: "start"|"end", bird_id, occurred_at }
     -> 202

POST /api/v1/aviary/events/offer
     { client_event_id, kind: "seed"|"song"|"pool", target_bird_id?: "...", occurred_at }
     -> 202 | 409 (cooldown violation; matter-of-fact body)

POST /api/v1/aviary/events/settle
     { client_event_id, action: "settle"|"undo", occurred_at }
     -> 202

POST /api/v1/aviary/birds/:bird_id/rename
     { name: "Pippa" }
     -> 200 (returns updated bird)
```

The presence_ping endpoint accepts the three signals separately (visibility, focus, last_input_at) so the server can validate the conjunction itself rather than trusting the client to compute presence — which keeps the drift-corruption prevention story honest.

### 4.4 Notebook endpoint

```
GET  /api/v1/notebook?cursor=...&limit=20
     -> { entries: [{ occurred_on_date, prose }], next_cursor: "..." }
```

Read-only. No PUT/POST/DELETE.

### 4.5 Account endpoints

```
GET  /api/v1/account                        -> profile + settings
PATCH /api/v1/account/settings              -> update reduced_motion_pref, captions_enabled, etc.
POST /api/v1/account/email-change           { new_email } -> 202; sends verification
GET  /api/v1/account/email-change/confirm?token=...
POST /api/v1/account/export                 -> 202; emails link
POST /api/v1/account/delete                 -> 200; soft-delete; window: 30 days
POST /api/v1/account/recover                -> 200; cancels pending soft-delete
```

### 4.6 Visit endpoints (host)

```
POST /api/v1/visits/invitations             { visitor_email } -> 201
GET  /api/v1/visits/invitations             -> list
POST /api/v1/visits/invitations/:id/revoke  -> 204
GET  /api/v1/visits/log                     -> visit log
```

### 4.7 Visit endpoints (visitor — token-bearer, no account needed)

```
GET  /v/:token                              -> page render
GET  /api/v1/visit/:token/snapshot          -> stripped snapshot (no listen_in_focused, no greetings)
                                              409 if revoked/expired (matter-of-fact body)
```

Visit snapshots **omit** `next_motion`'s `started_at` precision below 5 seconds — visitors see the aviary "as-is" but we do not give them sub-tick precision that would reveal information about how the simulation works. The visitor cannot send any `events/*` calls; the visit is render-only.

Visit pulls write a row to `visit_log` (bucketed by 60s) but never to the host's `event_log`. This is enforced by IAM: the visit endpoint's DB role does not have `INSERT` on `event_log`.

### 4.8 WebSocket channel

A WebSocket connection at `/ws/aviary` (auth via cookie) replaces snapshot polling for clients that have it. The server pushes a "state changed" notification on each tick that produced visible changes; the client then pulls the next snapshot. The WebSocket is best-effort; clients fall back to keepalive polling on a 60s cadence on disconnect.

### 4.9 Error envelope

All errors return a consistent matter-of-fact JSON body:

```json
{ "error": { "code": "expired_link", "message": "The link may have expired. Try requesting a new link." } }
```

Codes are stable. The `message` is the user-facing string; the client never invents its own. Sign-in, sync, and account error surfaces drop to matter-of-fact tone per `product_brief.md`.

---

## 5. Simulation engine design

The simulation engine is the spine of the product. This section is dense by necessity.

### 5.1 Tick driver

The Simulation Service runs one logical tick per account per ~60 seconds. Implementation: each account is sharded by `account_uuid` to a worker partition; each partition runs a single-threaded tick loop that processes accounts due for a tick.

A scheduler maintains a min-heap of `(next_tick_at, account_uuid)`. The loop pops the earliest entry, runs the tick for that account, writes results, and re-inserts with `next_tick_at = now() + 60s`. The 60s cadence is per-account, slightly jittered (±2s) to spread load.

Accounts sleep when there are no pending events and no active mood timers; the scheduler reduces their cadence to 5 minutes during long unattended periods to save CPU. They wake on the next event ingest.

### 5.2 Per-account tick algorithm

Pseudocode for the per-account tick:

```
def run_tick(account_uuid, now):
    state = load_state(account_uuid)            # birds + aviary_state + recent events
    events = events_since_last_tick(account_uuid)

    # 1. Update presence-time accumulator from presence_pings
    presence_seconds = compute_presence(events, last_tick_at=state.aviary_state.updated_at)

    # 2. Compute drift inputs from events + presence
    drift_inputs = aggregate_drift_inputs(events, presence_seconds)

    # 3. Apply drift to each bird's personality vector (low-pass filtered)
    for bird in state.birds:
        delta = compute_drift_delta(bird, drift_inputs, dt=now - state.aviary_state.updated_at)
        delta = clamp_delta(delta, max_per_tick=0.0005)  # hard cap per trait per tick
        delta = apply_monotonic_rule(delta)              # no negative drift; see 5.4
        bird.traits = clamp01(bird.traits + delta)

    # 4. Update mood
    for bird in state.birds:
        bird.mood, bird.mood_started_at = transition_mood(
            current_mood=bird.mood,
            personality=bird.traits,
            time_of_day=local_time(now, account.tz),
            recent_events=events_for_bird(events, bird.bird_id),
            ambient=state.aviary_state.current_weather,
            now=now
        )

    # 5. Decide ambient: weather, day/night phase
    state.aviary_state.daynight_phase = phase_for(local_time(now, account.tz))
    state.aviary_state.current_weather = maybe_advance_weather(
        state.aviary_state.current_weather, account.tz, now)

    # 6. Advance bird actions: scheduled motion, perch shifts, calls
    for bird in state.birds:
        plan_next_action(bird, state, now)   # writes next_call_at, next_motion_event

    # 7. Bird-to-bird coupling pass: alarms spread, choruses emerge
    coupling_pass(state.birds, now)

    # 8. Detect noteworthy moments → emit to Notebook Service
    moments = detect_noteworthy(events, state, prior_state=state_before_tick)
    for moment in moments:
        emit_notebook_candidate(account_uuid, moment)

    # 9. Compose snapshot blob; bump state_version; write atomically
    snapshot = compose_snapshot(state, now)
    write_state_atomic(account_uuid, state, snapshot, version=state_version+1)

    mark_events_consumed(events)
    record_tick(account_uuid, now, len(events))
```

Each tick is a single DB transaction over Simulation DB. If it aborts (deadlock, network), the events remain unconsumed and the next tick retries; the ordering is preserved because events are consumed in `event_id` order.

### 5.3 Drift function

For each trait `t` of bird `b`, the drift delta on a tick of duration `Δt` (seconds) is:

```
delta_t(b) = α(Δt) * (target_t(b) - current_t(b))

α(Δt) = 1 - exp(-Δt / τ_t)
```

where `τ_t` is the trait-specific time constant (target: 7 days for most traits, 14 days for plumage_saturation since plumage is the slowest visible drift), and `target_t(b)` is the *attractor* for that trait under the recent inputs.

The attractor for each trait:

```
target_boldness            = base + 0.6 * normalize(presence_seconds_per_day)
                                   + 0.3 * normalize(offers_near_b_per_day)
target_social_warmth       = base + 0.5 * normalize(listen_in_seconds_per_day_for_b)
                                   + 0.3 * normalize(presence_seconds_per_day)
target_vocal_frequency     = base + 0.4 * normalize(listen_in_seconds_per_day_for_b)
                                   + 0.2 * normalize(presence_seconds_per_day)
target_plumage_saturation  = base + 0.5 * normalize(presence_seconds_per_day)
                                   + 0.2 * normalize(any_interaction_per_day)
target_curiosity           = base + 0.5 * normalize(offers_accepted_by_b_per_day)
                                   + 0.2 * normalize(offers_near_b_per_day)
```

`base` is the bird's seed value (sampled at adoption from a per-species distribution). `normalize(x)` is `tanh(x / scale)` so heavy use saturates rather than overshooting `[0, 1]`.

**`presence_seconds_per_day`** is computed server-side from the conjunction-validated presence pings, *not* from "events received". A user with 600 pings in a day where each ping reported `visibility_state: visible AND has_focus AND last_input_at within 4min` accrues 600 × ping_interval = up to ~5 hours of presence. Clamped to 12 hours/day.

### 5.4 Monotonic-toward-expressive rule

The product's load-bearing engine asymmetry: traits never drift down on absence.

```
delta = max(0, raw_delta) for each trait
```

This is enforced in `apply_monotonic_rule` and is testable with a property-based test: given any sequence of "low presence" events, a trait's value is non-decreasing across all ticks. A unit test asserts this invariant.

A bird that's been ignored for two weeks will have `target_t(b)` drift toward `base` (because recent inputs decay), but since `delta` is clamped at 0 from below, the actual `current_t(b)` does not move. The bird becomes ambient by *not gaining* expressive traits, not by losing them. This matches "quieter than they were, not learning to mistrust."

The exception: mood can swing both directions (mood is fast-timescale and is allowed to go from content → wary on alarm coupling). Mood is not personality.

### 5.5 Mood transitions

Mood is one of `{wary, content, curious, drowsy, alert}`. Transitions are computed by a stochastic state machine where the transition probability is shaped by:

- Time of day in user's local zone: drowsy weight rises near dusk, alert in early morning.
- Recent events: an offer-accept boosts content; an alarm-call-from-neighbor boosts wary.
- Personality: `boldness` reduces wary→wary self-loop probability (a bold bird leaves wary faster); `social_warmth` boosts content→content loop.
- Ambient weather: rain dampens vocal_frequency for the duration and shifts a fraction of non-drowsy birds toward drowsy; wind shifts curious birds toward alert.

Concretely, each tick produces a transition probability matrix `P[mood_now → mood_next]` per bird, modulated by the inputs above; the next mood is sampled from this matrix using a per-bird PRNG seeded from `bird_id || mood_started_at`. Sampling rather than thresholding produces variation: two days with the same nominal inputs do not produce identical mood paths.

A "mood started_at" timestamp is preserved across sessions so that on session-start, the client renders the mood the bird has been in, not a fresh "just woke up" state. Mood snapping on tab open is forbidden by construction.

### 5.6 Call-grammar runtime

Calls are procedural. Each species has a *call grammar*: a small set of motifs (e.g., for the warbler: rising-trill, two-note-call, soft-warble), grammar rules for combining them, and per-motif synthesis parameters (envelope, harmonics, formants).

The server computes `next_call_at` and `next_call_descriptor` per bird per tick. The descriptor includes:

```json
{
  "motif_id": "warbler_3",
  "starts_at": "...",
  "duration_ms": 1200,
  "pitch_bias": 0.04,           // tiny per-call variation
  "envelope_seed": 8721389,     // per-call seed for amplitude shaping
  "loudness_bias": 0.0
}
```

The client receives the descriptor and synthesizes the actual sound from the motif library + parameters via WebAudio. Per-call seeds give each call enough variation that two calls of the same motif don't sound identical. The server does not synthesize audio.

The client never decides *when* a call happens; the server schedules `next_call_at`. A bird whose `next_call_at` is in the past (e.g., long disconnect, just-reconnected) gets its call descriptor rolled forward by the next tick; the client doesn't try to "make up" missed calls.

Call timing on the server is shaped by `vocal_frequency`:

```
call_interval_seconds(b) = base_interval / (0.3 + 0.7 * trait_vocal_frequency(b))
```

with `base_interval` species-tuned (warbler: 90s; sparrow: 60s; etc.). Plus jitter of ±25% per draw.

### 5.7 Bird-to-bird coupling

The coupling pass implements two interactions:

**Alarm spread.** When bird A's mood transitions to `wary` due to an external trigger (e.g., simulated predator, weather alarm — rare events), bird B at perch within an "earshot" radius gets a probability boost toward `wary` on its next mood transition. Alarm spread is bounded: each tick, at most one alarm event is processed per aviary, preventing oscillation. The probability boost is also dampened by B's `boldness`: a high-boldness B is less likely to absorb the alarm.

**Chorus emergence.** When two or more birds with high `vocal_frequency` have `next_call_at` within a 4-second window, the tick deliberately aligns them: shifts the second bird's `next_call_at` to start a beat after the first, producing an audible chorus for the user. The chorus is procedurally re-emerging — not scripted — and the alignment is a small "pull toward" rather than a hard snap. Birds with low `vocal_frequency` do not get pulled into the chorus.

### 5.8 Notebook entry generation

A "noteworthy moment" candidate is detected by the tick when one of the following predicates fires:

- `first_greeter_change`: the bird that greeted today is different from the recent N-day median first greeter.
- `long_quiet`: presence-time today is high but no calls in the last 10 min from any bird.
- `mood_shift_visible`: a bird transitioned from a long-running mood (>20 min) to a new mood after an interaction.
- `chorus_emerged`: a chorus event occurred.
- `weather_passed`: a rain or wind event began or ended.
- `plumage_threshold`: `trait_plumage_saturation` for a bird crossed a coarse band (low/mid/high) since last entry.
- `bold_step`: a bird with low historical boldness moved to the front perch for the first time in N sessions.

Candidates are passed to the Notebook Service, which applies sparsity and prose generation:

1. **Sparsity gate.** No more than 3 entries per 24h per aviary; no more than 1 entry per 3 hours.
2. **Prose generation.** A small templated grammar produces the entry text. Templates are bird-aware (use the bird's name) and time-aware (use the local day-of-week). Templates are fixed strings with slots, not free-form generation.
   - Example template for `first_greeter_change`: `"<day> — <bold_bird> greeted before <other_bird> today, first time this <window>."`
3. **Human-tunable.** All templates live in a YAML file (`notebook_templates.yaml`) that is reviewed by the product writer. The system never invents template strings at runtime.

The notebook generation step deliberately does not use any free-form LLM at runtime. The PRD's voice is too specific and load-bearing to entrust to a model that drifts; templates produce stable prose, and the writer owns the templates.

### 5.9 Return-greeting computation

When the client connects after an absence, the Aviary API computes the greeting server-side and inserts it into the snapshot:

```
def compute_greeting(account, now):
    last_presence_end = get_last_presence_end(account)
    absence_seconds = (now - last_presence_end).total_seconds()

    # Pick the bird that greets first today
    candidates = [b for b in account.birds if b.mood != 'drowsy']
    if not candidates: return None
    chosen = pick_by_weight(candidates,
        weights={b.bird_id: b.trait_boldness * mood_factor(b.mood) for b in candidates})

    # Pick greeting kind by absence length and chosen.boldness
    if absence_seconds < 5*60:
        kind = "glance"
    elif absence_seconds < 60*60:
        kind = "headtilt" if chosen.trait_boldness < 0.6 else "step_forward"
    elif absence_seconds < 24*60*60:
        kind = "two_note_call"
    else:
        kind = "long_call_then_response"  # triggers a second bird response

    return Greeting(bird_id=chosen.bird_id, kind=kind, started_at=now)
```

When `long_call_then_response` is chosen, the tick also schedules a second bird's response call ~700ms after the first. Multiple-bird greetings are *staggered*, never simultaneous. The greeting is procedurally varied per bird (kinds are templates; the actual motion/call descriptor is parameterized).

The greeting is inserted into one snapshot (the first after the session opens) and is not inserted again until the next return.

### 5.10 Adoption

On account creation:

1. Aviary record created.
2. Two birds drawn from species pool (no user choice). Selection: pseudorandom, biased toward complementary call signatures (we don't pick two warblers; we pick two species whose call grammars contrast).
3. User is prompted to name them; default suggestions are species-flavored ("Wren", "Pip", "Robin"). User can rename anytime.
4. On naming-flow completion, the empty aviary state is set; the first bird flies in over 1.5s; the second flies in 3s later. The fly-in is the *only* time the user sees a "from-empty" transition.

### 5.11 Adding a bird beyond two

A new species offer becomes available based on aviary age:

| Aviary age | Offer | Cap reminder |
|---|---|---|
| ≥ 21 days | Third bird offered | — |
| ≥ 60 days | Fourth bird offered | — |
| ≥ 120 days | Fifth bird offered | — |
| ≥ 200 days | Sixth bird offered | — |
| ≥ 300 days | Seventh bird offered | "this is the last bird the aviary can hold" — quiet copy in naturalist voice |

The offer surfaces as a quiet notebook entry ("a small new bird visited the front perch this morning. you could let it stay.") with an inline accept/decline. Decline is honored; the offer reappears after 30 days. The pacing is age-only — never engagement-driven.

---

## 6. Sync model

The sync model is summarized in one sentence: **the server is the only writer of canonical state; the client is a renderer.**

### 6.1 Snapshot-pull pattern

Clients receive state via snapshot pull only:

- On session start.
- On `visibilitychange` to visible.
- After a long render-frame gap (suspend detection: a frame > 5s indicates the device was suspended).
- On WebSocket "state changed" notification.
- On a 60s keepalive.

A snapshot includes a `state_version` (monotonic per account). Clients send `If-None-Match: <version>` to avoid re-pulling identical state. The Snapshot Service responds with 304 when there's no new state.

### 6.2 No client-side personality writes

The only mutation paths from client to server are interaction events. Personality vectors are never sent by the client. The codebase enforces this with a TypeScript barrier: the client's `ApiClient.events` module has overloads that accept only event payload shapes; the `ApiClient.aviary` module has no `setPersonality` or equivalent method.

### 6.3 Append-only event log; idempotent submission

Events carry a client-generated `client_event_id` UUID. The server upserts on `(account_uuid, client_event_id)`; duplicate submissions are ignored. The client retries failed submissions with the same `client_event_id`. This makes "wrote it twice because the network blipped" not a problem.

The simulation tick consumes events in `event_id` order (server-side, monotonic). Events submitted from two devices are interleaved by server-receive-time, but because drift is computed from the event log (not "current device state"), interleaving doesn't corrupt drift.

### 6.4 Why no last-write-wins

`accounts_sync.md` is explicit: a last-write-wins on personality would silently delete drift. We don't even have an API to write personality from the client; the failure mode is structurally unreachable. The Simulation Service has a single-writer invariant: only `simulation_service` DB role can `UPDATE birds SET trait_*`. Other services have `SELECT` permission only.

### 6.5 Multi-device experience

Two devices sign in. Both pull snapshots. Both render the same state. When user A interacts on device 1, an event flows to the log; the next tick processes it; the next snapshot pull on device 2 reflects the change. Latency: at worst 60s + WebSocket-push (~1s). Felt as: "I set down my laptop and picked up my phone; the aviary looks the same."

Conflict resolution is a non-feature. There's nothing to resolve.

### 6.6 Suspended-laptop detection and resync

When the client detects a long frame gap (> 5s of wall-clock time between rAF callbacks), it pulls a fresh snapshot. If the snapshot's state diverges visibly from the rendered state, the client cross-fades over 800ms.

The server doesn't send "you missed N ticks of state"; it just sends the current state. The client doesn't try to replay missed ticks.

### 6.7 Email change

A user requests an email change. New email is verified via magic link. On verification, the `email_search_hash` and `email_encrypted` are updated atomically. Old sessions remain valid — the email change is a profile change, not a session-revoke event. The user can revoke old devices manually.

---

## 7. Frontend rendering pipeline

### 7.1 Stack

- React 18 (only because team velocity > tech preference; the rendering hot path doesn't go through React).
- The aviary scene is a `<canvas>` rendered with WebGL2 (with Canvas2D fallback) wrapped in a thin React component for mounting only.
- State store: Zustand for UI state; the canonical aviary snapshot lives outside React, in a plain TS module accessible via Zustand's `subscribeWithSelector`.
- Build: Vite + esbuild. Target: modern evergreen browsers.

### 7.2 Scene composition

The scene is composed of:

1. Sky (gradient, day/night-phase-driven).
2. Background foliage (low-detail SVG, parallax × 0.3).
3. Middle plane: perches + birds.
4. Foreground branch (rare; parallax × 1.2).
5. Ambient ornaments: leaves, feathers (client-only, not in snapshot).

Birds are rendered as procedurally-deformed sprites. Each bird has a base sprite per pose (perched, mid-flight, preen-1, preen-2, head-tilt-left, head-tilt-right, settled, low-perch) — about 8 poses per species × 6 species ≈ 48 sprites. Each sprite is a layered SVG (silhouette + plumage layer). The plumage layer's saturation is shader-modulated by `trait_plumage_saturation` to produce visible plumage drift over weeks without re-shipping art.

Idle micro-motion is procedural: a perched bird's center-of-mass shifts on a slow noise field; head tilts on Perlin noise; preen poses cross-fade with seeded jitter. Mood tints the noise amplitudes — wary birds have higher head-scan amplitude, drowsy birds lower.

### 7.3 First-frame strategy

The first frame is critical to the "already-running aviary" promise. To hit <500ms time-to-first-bird:

1. The server inlines a minimal first-snapshot blob into the HTML response (~2KB, just the bird positions and a single first-frame pose per bird).
2. The CDN edge serves the HTML with the inline blob.
3. The client bundle is loaded; while it's loading, an inline `<script>` (≤4KB) renders a static first frame from the inline blob into the canvas (positioning birds at their perches in their current poses).
4. When the main bundle activates, it picks up the canvas state and starts animating from the static first frame — not from a re-render that snaps positions.

This means the user always sees birds first, never a spinner. If the inline blob isn't available (cold session, no cookie), the user sees the quiet field: a gradient sky, no spinner, no text. The aviary "catches up" within ~1s.

### 7.4 Idle micro-motion

Each visible bird has an `IdleAnimator` that runs on rAF and produces:

- Body bob (sinusoidal, slow, mood-amplitude).
- Head tilt (Perlin, mood-amplitude).
- Pose cross-fades (preen-1 → preen-2, etc.).
- Periodic call animations driven by `next_call_at` from the snapshot.

Idle motion runs continuously while the tab is visible. When the tab hides, rendering stops (no rAF). When the tab is re-shown, a fresh snapshot is pulled and rendering resumes.

### 7.5 Transitions between snapshots

When a new snapshot arrives:

- For each bird, compare its rendered state vs. snapshot state.
- If `current_perch` changed: animate a flight from old to new perch over `next_motion.duration_ms`, easing per the snapshot's hint.
- If `mood` changed: cross-fade idle-motion parameters over 1.2s.
- If `name` changed: no visual change, just internal state update.

If the divergence is large (long disconnection, missed several ticks), the client cross-fades to the new state over 800ms instead of trying to animate every transition.

### 7.6 Reduced-motion mode

Active when `prefers-reduced-motion: reduce` is set OR `account.reduced_motion_pref = true`. The render pipeline switches to:

- Idle micro-motion replaced by slow cross-fades between still poses (3–5s per cross-fade).
- Flight transitions become cross-fades between perches (no animated path).
- Ambient leaf/feather drift is removed.
- Day/night palette transitions remain, slowed (60s per transition instead of 30s).
- Calls still play; captions still appear.

Reduced-motion is its own rendering path implemented via a `RenderMode` enum that the renderer reads at scene-compose time. It is *not* implemented as "skip the rAF callbacks" — that would just be broken. It is its own designed surface with its own cross-fade aesthetic.

### 7.7 Top bar

Thin bar at the top of the viewport, ~48px tall. Contains four icons:

- Account/settings (gear).
- Accessibility settings (universal-access glyph).
- Field notebook (small book glyph).
- Offer affordance (a small leaf glyph that opens a panel with three options: seed / song / pool).

Top-bar fade: 4 seconds of cursor stillness fades the bar to 15% opacity. Cursor movement or any keyboard activity returns it to full opacity over 200ms.

Settle is reached by a small glyph at the right of the top bar (a setting-sun icon) — keyboard reachable, label visible to screen readers as "settle the aviary."

### 7.8 Empty-aviary state

The aviary scene with no birds renders the quiet field: gradient sky, soft horizon, no birds, no perches highlighted. Used during loading and the brief moment between adoption-flow completion and the first bird's fly-in. Never seen again after the first session.

### 7.9 Responsive layout

- Narrow phone (≤480px): scene compresses horizontally; perch positions scale; bird sprites scale uniformly down to 60% of desktop size; nothing crops.
- Tablet/desktop: scene widens; perches space out; sprite size remains constant; minimum scene width 320px, maximum 2000px (above which the scene letterboxes with the same gradient).
- All viewports preserve aspect ratio of birds; no stretching.

### 7.10 60fps budget

Render budget per frame: 16.6ms. Allocation:

- Snapshot interpolation/state update: ≤2ms.
- Bird idle animation update (5 birds): ≤4ms (≤0.8ms per bird).
- Canvas draw: ≤8ms.
- Call animation triggers + audio scheduling: ≤2ms.

Profile via Chrome DevTools' Performance recording on the 5-year-old laptop spec (a 2020 Intel MacBook Air for our internal benchmark fleet). Frame-time p99 < 16ms in CI on synthetic playback.

### 7.11 No memory growth

Run a 30-minute session in headless Chromium in CI. Sample heap every 30s. Assert max-heap variance < 8MB over the session. The test fails on any growth pattern that suggests un-released allocations. Specific guards:

- Audio buffers reused via a bounded pool.
- Notebook entries scrolled out of view dropped from the React tree.
- Worker threads bounded to one (audio worker) plus the main thread.
- `OffscreenCanvas` used for sprite generation; sprites cached in a fixed-size LRU.

---

## 8. Audio pipeline

### 8.1 Synthesis architecture

All calls are synthesized client-side in WebAudio. One `AudioContext` per page. Per-bird synthesis runs in an `AudioWorkletNode`; the worklet receives motif descriptors from the main thread and produces sample blocks.

Pipeline:

```
Per-bird synth nodes ──► gain (per-bird) ──► panner (perch-position-driven)
                                                      │
                                                      ▼
                                                 master gain
                                                      │
                                                      ▼
                                                 destination
```

### 8.2 Motif library and grammar

Each species has a motif library — a small set of motif primitives expressed as parameter dicts:

```yaml
warbler:
  motifs:
    - id: warbler_1
      kind: rising_trill
      f0_start: 1800
      f0_end: 2400
      duration_ms: 800
      vibrato_hz: 5.5
      vibrato_depth_cents: 30
      formants: [...]
      envelope: { attack_ms: 30, decay_ms: 200, sustain: 0.6, release_ms: 600 }
    - id: warbler_2
      kind: two_note
      ...
  grammar:
    rules:
      - sequence: [warbler_1, silence_300, warbler_2]
      - sequence: [warbler_3]
        weight: 0.6
```

Motifs are parameterized signal-generators (additive synthesis with formant filtering); the worklet renders them at runtime. The library is checked in as YAML; the server includes the motif IDs in `next_call_descriptor` but never the actual parameters (the client has them).

The grammar runtime selects sequences by weighted random with the per-call seed. This produces variation: the same motif played twice in a row uses different envelope jitter, different vibrato phase.

### 8.3 Listen-in mix

When the user listens-in to a bird, the per-bird gains animate over 1.5 seconds:

- Focused bird: gain ramps from 1.0 to 1.4.
- All other birds: gain ramps from 1.0 to 0.4 (not 0; never 0).

The ramp is a smoothed exponential curve, not linear, to avoid the "switching" feel. Disengage uses the same curve in reverse.

The "never silent" rule is enforced by clamping the unfocused gain at 0.35 minimum.

### 8.4 Chorus mixing

When two or more birds are calling within a 4s window (server-aligned per Section 5.7), the master mix automatically rises by a small amount (+1.5 dB) for the duration to give the chorus presence. This is not a "chorus mode" with switches; it's a continuous gain modulation reading per-bird voice activity.

### 8.5 Per-call variation

Each call uses:

- Pitch jitter: ±15 cents per call, seeded.
- Envelope jitter: ±5% on attack/decay/release, seeded.
- Vibrato phase: random per call.
- Loudness jitter: ±0.5 dB.

The jitter is seeded by the `envelope_seed` in the descriptor, not unseeded random — so on a snapshot replay, the call sounds the same. Per-call, it's never identical.

### 8.6 Captions

When `captions_enabled` is on, each call also produces a caption. The caption is generated by a deterministic mapping from the descriptor + the bird's current mood:

```
caption_for(motif="warbler_1", mood="content")  -> "a soft three-note rise"
caption_for(motif="warbler_1", mood="wary")     -> "a quick three-note rise, paused"
```

Caption strings are checked-in templates (YAML), keyed by `(motif_id, mood)`. The mapping is the authoritative writer-owned voice for the caption; no runtime generation.

The caption appears as small text near the calling bird, fading in at call start, holding for the call duration plus 800ms, fading out over 400ms. Position: above and slightly to the right of the bird, never overlapping the bird sprite. WCAG AA contrast required against the aviary background; the renderer measures the background luminance under the caption and selects a foreground that passes AA.

### 8.7 WebAudio fallback

Unavailability is detected by:

- `AudioContext` constructor missing.
- `audioContext.state === 'suspended'` after `resume()` attempt (autoplay policy block).
- Worklet load failure.

On detection: the audio pipeline disables itself; captions are turned on by default for the session (regardless of user setting); a small one-time matter-of-fact toast in the top bar reads "Audio is unavailable in this browser; captions are on." (This is the rare allowed system-voice surface — the user is talking to the browser as a system at this moment.)

We do not ship a recorded-audio fallback. Confirmed.

### 8.8 Audio context lifecycle

- Created lazily on first user gesture (autoplay policy).
- On tab hide: `suspend()`. On show: `resume()`.
- On account sign-out: `close()`.

### 8.9 Volume control

A volume control lives in accessibility settings. Default: 70%. Stored per-account so cross-device is consistent. There is *no* mute button on the top bar — muting is reached through settings, not as a one-click toggle, because reaching for it should be a deliberate choice ("I want this aviary quiet"), not a thoughtless app-level reflex. (This is a small design call against the obvious; flagged here for review.)

---

## 9. Accessibility surfaces

### 9.1 Screen-reader narration

A non-visual narrator runs on a slow cadence and writes prose into an `aria-live="polite"` region anchored off-screen. The narration source is the same canonical state as the visual scene; the narrator is a separate module that subscribes to snapshot updates.

Cadence:

- Idle: 1 narration update per 30–60 seconds, drawn from a small set of state-aware templates.
- User-initiated: 1 priority update on session-start (the return-greeting), on offer-completion, on settle-engage, on listen-in-engage.

Sample narrations:

```
a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

pip cocks her head and steps toward the front. wren calls once from the high branch.

the light is shifting toward evening. the calls are quieter now.
```

Narration prose is generated from the same template grammar as the notebook. No free-form runtime generation. The narrator exposes the same naturalist voice as the rest of the product.

The `aria-live` region is updated in full sentences (no incremental partials) so screen readers don't read partial state.

### 9.2 Captions

Detailed in Section 8.6.

### 9.3 Keyboard navigation

```
Tab           → focus moves through top bar (account, accessibility, notebook, offer)
Tab again     → focus enters aviary scene; first bird is focused
Arrow keys    → focus moves between birds (left/right)
Enter         → triggers listen-in on focused bird
Escape        → exits listen-in
Tab from bird → exits aviary scene back to top bar
```

Top-bar shortcuts (all keyboard reachable):

```
N → notebook
O → offer
S → settle
A → account
?  → keyboard shortcuts overlay (matter-of-fact)
```

### 9.4 Focus indicators

A 2px outline using a high-contrast color that the renderer chooses based on the local background luminance under the focused bird. The outline animates smoothly when focus moves between birds.

### 9.5 WCAG AA contrast

All chrome text passes AA. Captions and narration (when displayed visually) pass AA. The aviary scene itself contains no body copy, so contrast applies primarily to the chrome. The design system spec owns specific colors; contrast is checked in CI using `axe-core` against rendered chrome surfaces.

### 9.6 Reduced-motion

Detailed in Section 7.6.

### 9.7 Audio off

When the user has audio off (system-level mute, or `audio_enabled = false`), captions auto-enable. The product remains complete.

### 9.8 Settings surfaces

Account settings, accessibility settings, and sign-in are matter-of-fact voice (per `product_brief.md`):

```
Settings
  Audio                                [ enabled / disabled ]
  Captions                             [ on / off ]
  Reduced motion                       [ follow system / always on / off ]
  Volume                               [ slider ]
```

Naturalist voice is reserved for the aviary, the notebook, and the narration.

---

## 10. Performance budgets and observability

### 10.1 Bundle size

- Initial JS bundle ≤ 2MB gzipped.
- CSS ≤ 50KB.
- Static art (SVGs + sprite atlases) ≤ 800KB.
- Inline first-snapshot blob ≤ 4KB.

CI gate: bundle-size check fails the build if these are exceeded. Code-splitting is aggressive: settings, accessibility settings, account, visit-host UI, account export are all chunked separately and lazy-loaded.

### 10.2 Time-to-first-bird

p95 ≤ 500ms on a mid-tier mobile device on 4G. Measured via:

- Lighthouse runs on a synthetic mobile profile (Moto G4 emulation, 4G).
- Real-User Monitoring of `performance.timing` from the inline render moment, recorded as an aggregate metric (no per-account dimension).

### 10.3 Idle motion 60fps

p99 frame-time ≤ 16ms on the 5-year-old laptop benchmark. CI runs synthetic playback on a representative VM; production reads aggregate-only RUM frame-time histograms.

### 10.4 No memory growth over 30 minutes

Headless 30-min session in CI; max-heap variance < 8MB.

### 10.5 Simulation tick latency

p99 < 5s end-to-end (event consumption → state write). Alarm at p99 5s.

### 10.6 What we measure

| Metric | Source | Granularity | Per-account dimension? |
|---|---|---|---|
| Request count by endpoint | API gateway | 1m | No |
| Latency by endpoint (p50/p95/p99) | API gateway | 1m | No |
| Error rate by endpoint | API gateway | 1m | No |
| Simulation tick latency | Simulation Service | 1m | No |
| Active session count (anonymized) | Auth Service | 5m | No |
| Page load time (TTFB, FCP, first-bird) | Client RUM | aggregate | No |
| Render frame-time histograms | Client RUM | aggregate | No |
| Audio context error count | Client RUM | aggregate | No |
| Magic-link issuance/consumption | Auth Service | 1m | No (counts only) |
| Visit pull rate | Visit endpoint | 1m | No |

### 10.7 What we deliberately don't measure

- Anything per-account: no dashboard breaks down session length per account, no "this account's birds drifted X this week," no "this user clicked offer Y times."
- Cross-account aggregates of per-bird state: no "average drift," no "average mood," no "most popular bird species." These would convert per-account interaction into a data product.
- Visit-frequency histograms per account.

The data-pipeline boundary enforces this: the metrics warehouse cannot ingest from Simulation DB or Notebook DB. Periodic audit (quarterly) confirms no metric carries an `account_uuid` or any field that would let a join reconstruct per-account information.

### 10.8 Observability stack

- Metrics: Prometheus + Grafana.
- Logs: structured JSON; logs that mention an account use `account_uuid` only, never email.
- Traces: OpenTelemetry; traces are sampled at 1% and never include event payloads.
- Alarms (PagerDuty):
  - Tick latency p99 > 5s for 5 minutes
  - Error rate on `/api/v1/aviary/snapshot` > 0.5% for 5 minutes
  - Magic-link issuance failure rate > 1%
  - Audio-context error rate > 5% (population)
  - Bundle build size exceeds budget (CI; not paging)

### 10.9 Synthetic probes

A fleet of headless browsers from three geographies runs the aviary every 5 minutes:

- Sign in via test account.
- Pull snapshot.
- Render first bird; assert visible within 500ms.
- Trigger one offer; assert acknowledgment within 200ms.
- Watch for one tick to land in the snapshot.
- Sign out.

Failures page on-call.

---

## 11. Rollout

### 11.1 Phased release

Three phases over ~10 weeks from feature-complete:

**Phase 1 — Internal alpha (weeks 1-2).** ~30 internal users. Two-bird default. All features on. Daily review of error rates, tick latencies, and a manual review of synthetic narrations and notebook entries to catch voice drift.

**Phase 2 — Closed beta (weeks 3-6).** ~500 invited users. Email invitation; no public sign-up. Monitor:
- Drift calibration: do birds at 7 days have measurable trait change in the test harness? Do birds at 21 days feel different to users (qualitative survey)?
- Audio uncanniness: are calls being reported as "robotic" or "samey"? Sample anonymous clips.
- Accessibility regressions: synthetic axe-core runs on every build; manual screen-reader tests on every release.

**Phase 3 — Public open (week 7+).** Sign-up open; bird offer pacing as designed (third bird at 21 days, etc.). Ramp aviary count over 4 weeks via a feature-flag-controlled cap on new sign-ups.

### 11.2 Feature flags (operational, not user-visible)

Flag list:

- `bird_cap_n` (server) — runtime cap on birds-per-aviary, default 7. Reducible to 5 if chorus blurring becomes a problem in the field.
- `tick_cadence_seconds` (server) — default 60. Tunable for load-shedding.
- `drift_tau_days` (server) — default 7 (14 for plumage). Tunable if calibration is wrong.
- `notebook_max_per_24h` (server) — default 3. Tunable for sparsity correctness.
- `webaudio_required` — default true; if false, the WebAudio-unavailable fallback applies to all sessions (used in case of widespread browser-side audio bug, never as user-facing setting).

Flags are server-only. There is no user-visible feature toggle for "experimental greeting" or "new audio engine."

### 11.3 Migration path

V1 launches with a fresh DB; no migration. Schema versioning via Postgres migrations (Flyway/sqitch). Schema changes that touch the personality vector require a code review checklist that explicitly asks whether the change preserves drift identity.

### 11.4 Day-one instrumentation

From day one:

- Synthetic probes (Section 10.9) running.
- Aggregate RUM (Section 10.6) collecting.
- Magic-link issuance/consumption counters.
- Tick latency dashboard.
- Bundle size CI gate.
- 30-min memory test in CI.
- axe-core in CI.

We do *not* ship a "view per-user activity" admin tool, even for ourselves. Internal debugging uses a privileged tool that requires a written incident-response justification and audit-logs every read. The tool is rarely opened.

### 11.5 Privacy launch checklist

Before phase 3 (public open):

- [ ] `email_search_hash` is the only join path between auth and other services (verified in code review).
- [ ] No telemetry metric in the warehouse has `account_uuid` (audit script).
- [ ] No telemetry metric has any per-bird field (`bird_id`, `species_id`, etc.) (audit script).
- [ ] Logs scrubbed for email-shaped strings (regex check on log samples).
- [ ] Privacy policy reviewed by legal and visible from account settings.
- [ ] KMS audit log reviewed for last 30 days; only Auth Service decrypted email.

### 11.6 Voice-drift review

A weekly review by the product writer of all new notebook entries and narrations to catch voice drift. Templates that produce off-voice output are revised in the YAML. This is a real working process, not a checkmark — the voice is the product.

### 11.7 What gets re-calibrated based on phase 2

| Calibration | Trigger to re-tune |
|---|---|
| Drift τ | If user qualitative survey at week 3 says "no, my birds feel the same as week 1" |
| Tick cadence | If tick latency p99 > 3s sustained |
| Notebook entry rate | If users report "too many" or "feels empty" |
| Greeting variation | If users report "feels canned" |
| Per-call jitter range | If audio team flags audible repetition |

Calibration is owned by product + audio + simulation engineers; A/B testing on these is forbidden because A/B testing on the product's affective core would itself shift the relationship the product is selling (see Section 12.5 risk).

---

## 12. Risks

### 12.1 Drift calibration too fast

**Failure mode.** Users notice their birds changing session-to-session. Pocket Aviary becomes a stat-management exercise without ever showing stats.

**Detection.** User survey at week 3 of beta; quantitative: trait deltas per session in the test harness should be < 0.001 for any single session.

**Mitigation.** Conservative initial τ (7 days). Hard per-tick cap on delta. Surface drift only at 21+ days. If the survey indicates drift is too fast, raise τ to 10 or 14 days.

### 12.2 Drift calibration too slow

**Failure mode.** Users at 3+ weeks report no felt change. The "feels alive over weeks" promise fails.

**Detection.** Same survey; quantitative: birds at 21 days of moderate use should have at least one trait change > 0.15 from base.

**Mitigation.** Lower τ; raise the attractor offsets; ensure the per-bird base values aren't too close to mid-range (which dampens visible change).

### 12.3 Sync corruption: personality lost or doubled

**Failure mode.** A code change introduces a path where personality is written from the client, or two ticks for the same account run concurrently and overwrite each other.

**Detection.**
- The DB-role single-writer invariant is verified daily by a script that lists processes with `UPDATE` privilege on `birds.trait_*`.
- Tick-record uniqueness constraint catches concurrent ticks at insert time.
- A property-based test asserts: for any sequence of events, replaying the event log from genesis produces the same final state vector.

**Mitigation.** Hard enforcement at the DB role level; never at code-level only. Concurrent-tick attempt aborts with a clear error.

### 12.4 Audio uncanniness

**Failure mode.** Procedural calls sound robotic, or a motif repeats audibly, or the chorus phase-cancels ugly artifacts.

**Detection.** Sampling of calls during phase 2 (anonymized). Audio team reviews. User survey question: "do the calls sound real?"

**Mitigation.** Per-call jitter ranges tunable. Motif libraries can be expanded (more variants per species) without server changes. If a species' calls are reported as flat, expand its motif library and ship.

The harder failure: if procedural synthesis at the bundle budget *cannot* produce naturalist-feeling calls, the product premise has a problem. Mitigation here is honesty — re-evaluate whether procedural is right; possibly explore "procedural seed + small per-call texture sample" hybrid (still no recorded loops). Decision point: end of phase 2.

### 12.5 Engagement-pressure leak (gamification creeping in)

**Failure mode.** A "harmless" engagement feature is added. The first one cracks the rule. Six months later, the product has streaks.

**Detection.** PR review checklist for any user-visible surface includes "does this announce, count, or rank?" Pre-PR design review for any new surface routes through the design lead, who owns the gamification refusal.

**Mitigation.** This risk is cultural, not technical. The engineering plan reflects it by:

- Not building any infrastructure that could feed a future streak counter (no per-account visit history table, no daily aggregation job).
- Not exposing any internal "user activity" surface in the product or in admin tools.

The non_goals.md is referenced by the design lead at every product review. If the team feels the product needs an engagement feature, the rule is: build a different product, not crack this one.

### 12.6 Privacy regression

**Failure mode.** A new metric or log field includes per-account state. Email leaks into a log. The simulation DB is read by a poorly-scoped service.

**Detection.** Quarterly audit of metric definitions. Code linter on log statements. KMS audit log review.

**Mitigation.** Detailed in Section 11.5. The architectural separation (separate IAM, separate DBs) makes accidental leakage hard.

### 12.7 Accessibility regression

**Failure mode.** A new feature ships without accessibility surfaces. Reduced-motion stops working after a render-pipeline change. The narrator falls behind the visual.

**Detection.** axe-core in CI. Manual screen-reader test on every release. Reduced-motion smoke test in CI.

**Mitigation.** Accessibility is in the definition-of-done. PR template includes "have you checked screen-reader narration / reduced-motion / keyboard navigation?"

### 12.8 First-frame failure

**Failure mode.** The inline first-snapshot path breaks; users see a spinner.

**Detection.** Synthetic probe asserts first-bird-render < 500ms; alarm fires.

**Mitigation.** The "quiet field" fallback is the only acceptable degraded state. A spinner is forbidden by code review.

### 12.9 Bird identity slip

**Failure mode.** A bug in the simulation reassigns a bird's `bird_id` (e.g., during a migration). User feels something is wrong without naming it.

**Detection.** A unit test asserts `bird_id` is never re-issued. A migration checklist explicitly asks about bird_id stability.

**Mitigation.** `bird_id` is a UUID, generated once at adoption, never recomputed. Migrations on the `birds` table are required to preserve `bird_id` (asserted via post-migration checks). If a `bird_id` ever changes, the affected accounts are flagged and manually reviewed.

### 12.10 Visit feature corrupts host drift

**Failure mode.** A visitor's pull is mistakenly written to the host's event log; the host's drift now reflects visitor attention.

**Detection.** The visit endpoint's DB role does not have INSERT on `event_log` (verified daily by audit script).

**Mitigation.** Architectural separation. Even if app-layer code tried to write a visit pull as an event, the DB would reject it.

### 12.11 Notebook prose drifts off-voice

**Failure mode.** A new template added in a hurry sounds like an event log. The notebook, the surface where voice is most concentrated, leaks announcement-style prose.

**Detection.** Weekly voice review by the product writer (Section 11.6).

**Mitigation.** Template additions go through writer review before merge. The YAML is in the same repo and the writer is on the PR review.

### 12.12 Multi-device sign-in race

**Failure mode.** A user clicks magic-link on phone, then the link is replayed on laptop. Two sessions are active; events from both interleave.

**Detection.** Magic-link is single-use (consumed_at). Replay is rejected.

**Mitigation.** Magic links are invalidated on consumption. Two devices can both be signed in via separate magic-links; that's fine — the event log absorbs interleaving without corruption.

### 12.13 Email change race

**Failure mode.** User initiates email change; old email and new email are both used; account is in an indeterminate state.

**Detection.** The pending email-change is stored on the account row; only one pending change at a time; until verification, both old and new email continue to receive sign-in, but both authenticate to the same account.

**Mitigation.** The `email_change_pending` JSONB enforces single-pending. On verification, atomic update. Old email is invalidated on commit.

---

## 13. Engineering work breakdown (sequenced)

This is a coarse-grained sequence for an engineering team to work to. Sub-tasks per service are owned by the service's engineer and not enumerated here.

1. **Foundation (week 1).** DB schemas; account UUIDs; encrypted email; KMS key creation; auth flow with magic links. CI scaffolding, bundle gate, axe-core.

2. **Simulation core (weeks 2-3).** Tick driver; drift function; mood transitions; per-account scheduler; event log; idempotent submission. Property-based tests on drift invariants.

3. **Snapshot service (week 3).** Read API; If-None-Match; WebSocket push.

4. **Bird species pool & call grammar (weeks 3-4).** Six species, motif libraries, grammar runtime, audio worklet, per-call jitter. Audio team review.

5. **Frontend rendering (weeks 4-5).** Canvas scene; first-frame inline path; idle micro-motion; snapshot-driven transitions; reduced-motion mode; top bar with fade.

6. **Listen-in / offer / settle (week 5).** Interaction events; gain ramps; cooldowns; settle undo.

7. **Notebook (week 6).** Templates; sparsity gate; entry generation. Writer review of templates.

8. **Accessibility (week 6).** Narrator module; ARIA region; captions; keyboard navigation. Manual screen-reader test pass.

9. **Visits (week 7).** Invitation issue/revoke; visitor-token endpoint; visit log; visitor-side render path.

10. **Account management (week 7).** Email change; export; soft-delete; session list.

11. **Hardening (week 8).** Synthetic probes; alarm wiring; KMS audit; quarterly-style audit script; privacy launch checklist.

12. **Phase 1 alpha (weeks 9-10).** 30 internal users; daily error/voice review.

13. **Phase 2 closed beta (weeks 11-14).** 500 invited; calibration review.

14. **Phase 3 public (week 15+).**

---

## 14. Outstanding decisions and assumptions

A small list of judgment calls this plan made where the PRD left room. Each is named so the engineering team and design lead can revisit if needed.

- **Tech stack.** React + Zustand + Vite + WebGL2 picked by team familiarity, not by PRD requirement. If the team has stronger Svelte/Solid.js preferences, swap freely; the architectural shape doesn't depend on the framework.
- **Tick cadence (60s).** A defensible balance between "feels alive" and operational cost. If load is fine, 30s would feel slightly snappier on mood transitions.
- **Drift τ (7 days).** Calibration committed for v1; will be re-tuned in phase 2 based on qualitative feedback.
- **Mute reachability.** Volume is in settings, not in the top bar. This is a design call against the obvious that the product brief implicitly supports (no UI chrome inside the aviary; restraint over richness). Flagged for design-lead sign-off.
- **Synthetic UUID via `gen_random_uuid()`.** Postgres native; no separate UUID service. If we anticipate sharding, we may move to a Snowflake-style ID; not needed at v1 scale.
- **WebGL2 vs Canvas2D.** Canvas2D would simplify the renderer at the cost of tighter constraints on plumage saturation modulation. WebGL2 is the chosen path; Canvas2D is the fallback for browsers that lack WebGL2 support (rare in our supported set).
- **Bird-to-bird coupling tunables.** The "earshot" radius for alarm spread and the chorus alignment window are calibrated values; initial settings are educated guesses, subject to phase 2 tuning.
- **Notebook entries language.** v1 is English-only. i18n is a v2 problem; the templates are in YAML keyed only by English at v1.
- **Email-change effect on sessions.** Sessions remain valid on email change. Choice is between "keep sessions, treat as profile update" (chosen) and "force re-auth on email change" (security stance). The product is low-stakes; the chosen path is friendlier.
- **Visitor per-IP rate limit.** Visitors share `/v/:token` paths; a per-token rate limit (10 pulls per minute) prevents accidental hammering. Not a security feature; a politeness one.

---

## 15. Definition of done for v1

The product is v1-shippable when:

- All in-scope features pass functional acceptance.
- p95 time-to-first-bird ≤ 500ms on the mid-tier mobile/4G fleet.
- p99 tick latency < 5s under modeled production load.
- 30-min CI memory test passes.
- axe-core CI passes; manual screen-reader pass on the latest release branch.
- Bundle size ≤ 2MB.
- Synthetic probes green for 7 consecutive days.
- Privacy launch checklist all checked.
- Voice-drift review passes for the last 4 weeks of new notebook templates and narrator templates.
- Phase 2 calibration review concludes drift τ and tick cadence are correct.

The product is *not* shipped when any of:

- A user-visible streak or counter exists anywhere.
- A toast on return-greeting exists.
- The screen reader narration is "state list" rather than naturalist prose.
- WebAudio unavailability triggers a recorded-audio fallback.
- The simulation runs anywhere on the client.
- Any metric in the warehouse carries an `account_uuid` or per-bird field.

These are the load-bearing principles encoded as ship criteria.

---

End of plan.
