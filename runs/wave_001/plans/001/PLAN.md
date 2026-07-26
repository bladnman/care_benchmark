# Pocket Aviary — v1 Implementation Plan

**Status:** Phase-1 plan, ready for engineering execution. No product code is written by this document.
**Audience:** A frontier engineering team (client, backend/simulation, audio, accessibility, infra) executing v1 end to end.
**Source of truth:** `prd/` (product_brief, concepts, bird_engine, interactions, aviary_layout, accounts_sync, social_optional, accessibility_perf, non_goals). Where the PRD is deliberately unspecified, this plan makes a call and records it in §14 (Decisions & assumptions). Where the PRD is explicit, this plan does not renegotiate it.

---

## 0. Reading this plan

Three things in the PRD are load-bearing in the sense that getting them wrong invalidates the rest of the build. They are called out here so that no section of this plan is read in isolation from them:

1. **The aviary continues without the viewer.** This is an architectural property (server-side tick, canonical state, no entry animation, no spinner), not a visual effect. §2, §5, §7.
2. **Drift is monotonic toward expressive.** Neglect produces quietness, never regression. This is enforced at the math layer, the database layer, and the test layer. §5.2, §12.
3. **Notice, never announce.** Every "harmless" announcement surface (toast, badge, streak, counter, welcome banner, friend-visited ping) is refused, and the refusal is mechanized as CI-enforced invariants rather than left to reviewer discipline. §11.4, §12.6.

Everything else in this plan hangs off those three.

---

## 1. Scope

### 1.1 In scope for v1

| Area | v1 content |
|---|---|
| Accounts | Single-user account, email + magic-link sign-in, per-device revocable sessions, verified email change, JSON export, soft-delete (30d) then hard delete |
| Aviary | One canonical aviary per account; 2 starter birds at adoption; hard cap 7; age-gated new-bird offers |
| Bird engine | 5-trait hidden personality vector, monotonic drift, 6-state mood FSM, bird-to-bird interaction, procedural call grammar, mood-shaped idle motion |
| Simulation | Server-side tick (~60s logical), append-only interaction event log, server-authored additive deltas |
| Interactions | Return-greeting, presence accounting, listen-in, offer (seed / song fragment / still pool), settle (+5s undo), field notebook (read-only) |
| Scene | Single horizontal non-scrolling scene, three perch zones, local-time day/night, ambient weather, ambient micro-motion, fading top bar, no in-scene chrome |
| Social | Visit invitations by email, read-only ambient visitor view, revocation, 30-day invite expiry, visit log, off-by-default visit notification toggle |
| Accessibility | Naturalist screen-reader narration, designed reduced-motion mode, procedural call captions, full keyboard navigation, WCAG AA on all user copy |
| Performance | ≤2 MB gz initial JS (ceiling), <500 ms time-to-first-bird, 60 fps idle on a 5-year-old laptop, zero memory growth over 30 min |
| Privacy | Synthetic account UUID everywhere, per-bird interaction data never aggregated, hard data-plane separation from analytics |

### 1.2 Explicitly out of scope for v1

Native iOS/Android apps. Payments/billing/tiers. Shared, team, or household aviaries. Multi-aviary accounts. Customizable or purchasable scenes. Public discovery, directories, feeds, profiles, follows, mutual visits, comments on visits. Leaderboards or any cross-account ranking — *including the underlying metrics that would make one possible*. Push notifications, marketing email about the aviary, re-engagement email. All gamification: achievements, badges, levels, XP, scores, streaks, day-counters, visit calendars, "birds adopted: N", milestone celebrations — in any tier, setting, or opt-in form. Tamagotchi mechanics: death, hunger, distress, decaying happiness meters, care obligations. Any surface that displays personality-vector values, in any tier, debug build, or support tool. Recorded-audio call playback of any kind, including as a fallback. Co-presence or shared cursors during visits. SSO/password auth. Compatibility paths for browsers older than the last two major versions of Chrome, Safari, Firefox, Edge.

### 1.3 Scope guards (how out-of-scope stays out)

Non-goals are not documentation; they are tests. §12.6 specifies an executable `product-invariants` suite that fails CI on: a banned-lexicon match in any user-facing string bundle; the existence of a toast/snackbar/banner primitive in the design system; any route, component, or API field matching the gamification vocabulary; any snapshot payload field carrying a raw trait value; any analytics event type carrying an account or bird identifier. A non-goal that only lives in a doc gets re-litigated in six months; a non-goal that fails the build does not.

---

## 2. Architecture

### 2.1 Shape

Five deployable services plus a static/edge tier. Deliberately few: the product's complexity is in the simulation and the client, not in the topology.

```
                       ┌──────────────────────────────────────────┐
   browser ───────────▶│ edge (CDN + edge render)                 │
   (aviary client)     │  · static shell, hashed assets           │
                       │  · document render w/ inlined snapshot   │
                       └───────┬──────────────────────────────────┘
                               │ (private origin)
       ┌───────────────────────┼───────────────────────┬────────────────────┐
       ▼                       ▼                       ▼                    ▼
 ┌───────────┐          ┌────────────┐          ┌────────────┐       ┌────────────┐
 │  auth-svc │          │  api-svc   │          │ visit-svc  │       │  mail-svc  │
 │ magic link│          │ snapshots  │          │ invites,   │       │ magic link,│
 │ sessions  │          │ events in  │          │ visitor    │       │ invite,    │
 │ account   │          │ notebook   │          │ snapshots  │       │ export     │
 │ lifecycle │          │ settings   │          │            │       │            │
 └─────┬─────┘          └─────┬──────┘          └─────┬──────┘       └────────────┘
       │                      │                       │
       └──────────────┬───────┴───────────────────────┘
                      ▼
            ┌──────────────────────┐        ┌───────────────────────────┐
            │  sim-db (Postgres)   │◀───────│  sim-tick (worker fleet)  │
            │  canonical state     │  ONLY  │  sharded by account_uuid  │
            │  + append-only log   │ writer │  leases, tick_seq, drift  │
            └──────────────────────┘  of    └───────────────────────────┘
                      ▲               traits
                      │ (read-only, cache)
            ┌──────────────────────┐
            │  snapshot cache      │   Redis: snapshot-by-tick_seq, rate limits,
            │  (Redis)             │   magic-link throttles, tick lease registry
            └──────────────────────┘

   ┌─────────────────────────────────────────────────────────────────────┐
   │ TELEMETRY PLANE — physically separate. No network path to sim-db.   │
   │ Aggregate-only metrics, structurally incapable of carrying IDs.     │
   └─────────────────────────────────────────────────────────────────────┘
```

**Language and runtime.** TypeScript everywhere, Node 22 LTS on the server (Fastify for HTTP). The decisive reason for one language is that the simulation kernel must be **one package, three consumers**: `sim-tick` (authoritative execution), the calibration lab (§12.3, accelerated-time replay), and the client (§7.4, render-only extrapolation). Three reimplementations of a drift function is three drift functions.

`@aviary/sim` is a pure, dependency-free, deterministic package. Rules enforced by lint and by a determinism test: no `Math.random` (a seeded xoshiro128** PRNG is the only entropy source), no `Date.now()` inside kernel functions (time is always a parameter), no floating-point reduction whose order depends on map/set iteration. The package is split into `sim/core` (mood, motion, call scheduling — shipped to the client) and `sim/drift` (personality traits — **never** shipped to the client; enforced by a bundle-content CI check). The client is structurally unable to compute or hold a trait value.

### 2.2 Client/server split — the invariant

| Concern | Owner | Never |
|---|---|---|
| Personality vector | Server (`sim-tick` only) | Never computed, held, transmitted, or written by any client |
| Mood, perch, motion state, call intent, weather, greeting intent | Server (canonical), client interpolates/extrapolates | Client never persists these back |
| Interaction events (presence, listen-in, offer, settle) | Client emits, server owns the log | Client never emits derived state ("set boldness to X") |
| Rendering, audio synthesis, caption text, narration prose | Client | Server never sends pixels, audio, or pre-rendered animation |
| Notebook entries | Server (generated post-tick) | Client never authors or edits |

The one-line statement of the contract: **clients send observations of the user; servers send observations of the aviary.**

### 2.3 Render pipeline boundary

The boundary sits at the **snapshot**. A snapshot is a small, versioned, timestamped description of the aviary's canonical state plus enough forward-looking intent (next call windows, in-flight transitions, greeting intent, active weather) for the client to render smoothly for the next ~90 seconds without another round trip. Everything on the client side of that boundary — pose blending, spring interpolation, noise-driven breathing, oscillator graphs, caption prose, leaf drift — is presentation. Everything on the server side is simulation.

Two properties make this boundary hold:

- **Snapshots carry derived render parameters, not traits.** See §4.4. A user with DevTools open sees `motionEnergy: 0.41` and `plumageStep: 3`, not `boldness: 0.62`. This is how "never exposed numerically" is enforced literally rather than aspirationally.
- **Ambient ornament is not simulated.** Leaves and feathers have no server state (per `aviary_layout.md`); they are client-side rendering ornaments on an idle cadence. Two devices will show different leaves. That is correct and intended; leaves are not part of the aviary's identity.

---

## 3. Data model

Postgres 16. Schema `sim` (canonical, restricted) and schema `iam` (accounts, auth). All identifiers are UUIDv7 (time-ordered, index-friendly).

### 3.1 Identity and accounts

```sql
-- iam.accounts — the ONLY place an email exists.
CREATE TABLE iam.accounts (
  id                uuid PRIMARY KEY,                    -- synthetic; the ONLY cross-service identifier
  email_ciphertext  bytea NOT NULL,                      -- AES-256-GCM, envelope key from KMS
  email_blind_index bytea NOT NULL UNIQUE,               -- HMAC-SHA256(pepper, lower(email)) for lookup
  email_verified_at timestamptz,
  pending_email_ct  bytea, pending_email_bi bytea,       -- email-change staging
  tz_offset_minutes smallint,                            -- coarse; see §14 (no geolocation)
  status            text NOT NULL,                       -- active | pending_delete | deleted
  delete_after      timestamptz,                          -- soft-delete horizon (+30d)
  created_at        timestamptz NOT NULL
);

CREATE TABLE iam.sessions (
  id uuid PRIMARY KEY, account_id uuid NOT NULL REFERENCES iam.accounts(id),
  token_hash bytea NOT NULL UNIQUE,                       -- SHA-256 of the opaque cookie value
  device_label text, user_agent_family text,              -- "Safari on iPhone" — no full UA string retained
  created_at timestamptz NOT NULL, last_seen_at timestamptz NOT NULL, revoked_at timestamptz
);

CREATE TABLE iam.magic_links (
  id uuid PRIMARY KEY, email_blind_index bytea NOT NULL,
  token_hash bytea NOT NULL UNIQUE, expires_at timestamptz NOT NULL,  -- issue + 15 min
  consumed_at timestamptz, requested_ip_hash bytea, created_at timestamptz NOT NULL
);
```

**The synthetic-UUID rule, mechanically.** `iam.accounts.id` is the only account reference permitted outside `iam`. `email_ciphertext` and `email_blind_index` are `REVOKE SELECT`-ed from every role except `auth_svc`. A CI check greps service code for email-shaped identifiers in log statements, metric labels, queue keys, and partition keys. The blind index exists so sign-in lookup never requires decrypting the corpus; the pepper lives in KMS, not in the database.

### 3.2 Aviary and birds

```sql
CREATE TABLE sim.aviaries (
  id uuid PRIMARY KEY,
  account_id uuid NOT NULL UNIQUE REFERENCES iam.accounts(id),   -- one aviary per account (v1)
  created_at timestamptz NOT NULL,                               -- drives age-based bird offers
  tick_seq bigint NOT NULL DEFAULT 0,                            -- monotonic canonical version
  last_tick_at timestamptz NOT NULL,
  next_offer_eligible_at timestamptz,                            -- age-gated third-bird offer
  scene_seed bigint NOT NULL                                     -- stable ornament/weather phase
);

CREATE TABLE sim.birds (
  id uuid PRIMARY KEY,                                           -- STABLE FOREVER. Never reissued.
  aviary_id uuid NOT NULL REFERENCES sim.aviaries(id),
  species_id text NOT NULL,                                      -- references versioned species config
  name text NOT NULL,                                            -- user-assigned, freely renameable
  identity_seed bigint NOT NULL,                                 -- FROZEN: call signature + motion phase
  adopted_at timestamptz NOT NULL,
  ordinal smallint NOT NULL                                      -- 1..7, adoption order
);

-- Separated from sim.birds so that write privilege can be revoked independently.
CREATE TABLE sim.bird_traits (
  bird_id uuid PRIMARY KEY REFERENCES sim.birds(id),
  boldness           real NOT NULL, social_warmth real NOT NULL,
  vocal_frequency    real NOT NULL, plumage_saturation real NOT NULL,
  curiosity          real NOT NULL,
  signal_state       jsonb NOT NULL,      -- low-pass filter state S (per §5.2)
  calibration_version smallint NOT NULL,  -- which η/α constants produced this history
  applied_through_tick bigint NOT NULL,   -- exactly-once watermark against the event log
  updated_at timestamptz NOT NULL,
  CONSTRAINT traits_in_range CHECK (
    boldness BETWEEN 0 AND 1 AND social_warmth BETWEEN 0 AND 1 AND
    vocal_frequency BETWEEN 0 AND 1 AND plumage_saturation BETWEEN 0 AND 1 AND
    curiosity BETWEEN 0 AND 1)
);
```

Three pieces of enforcement live on `sim.bird_traits` and they are the technical spine of the engine's core promises:

1. **Only the tick writes it.** `GRANT UPDATE ON sim.bird_traits TO sim_tick_role;` and nothing else. `api_svc_role`, `visit_svc_role`, and `auth_svc_role` have no write grant, and `visit_svc_role` has no `SELECT` grant at all. A well-meaning future endpoint physically cannot mutate a personality.
2. **Monotonicity is a database trigger, not a convention.** `BEFORE UPDATE` raises unless every trait column is `>=` its previous value (within a `1e-7` float tolerance). If a code path ever tries to decrement a trait, it fails loudly in staging, not silently in production. The one authorized exception path is a versioned migration that must set a session GUC and is logged.
3. **Exactly-once consumption.** `applied_through_tick` is the watermark against `sim.interaction_events`; a tick that crashes mid-write cannot double-apply presence on retry.

### 3.3 Mood and render state

```sql
CREATE TABLE sim.bird_state (
  bird_id uuid PRIMARY KEY REFERENCES sim.birds(id),
  mood text NOT NULL,                    -- wary|content|curious|drowsy|alert|roosting
  mood_since timestamptz NOT NULL,
  mood_dwell_min_until timestamptz NOT NULL,   -- hysteresis floor (§5.3)
  mood_impulse jsonb NOT NULL,           -- decaying per-mood impulse accumulators
  perch_zone smallint NOT NULL,          -- 0 front, 1 middle, 2 back
  perch_slot smallint NOT NULL,          -- stable index within the zone
  transit jsonb,                         -- in-flight move {from,to,started_at,duration_ms}
  next_call_at timestamptz,              -- scheduled call intent
  last_call_at timestamptz,
  last_greeted_at timestamptz,
  updated_at timestamptz NOT NULL
);

CREATE TABLE sim.weather_events (
  id uuid PRIMARY KEY, aviary_id uuid NOT NULL REFERENCES sim.aviaries(id),
  kind text NOT NULL,                    -- rain | wind
  intensity real NOT NULL,               -- 0..1, capped at 0.6 (never assertive)
  started_at timestamptz NOT NULL, ends_at timestamptz NOT NULL
);

CREATE TABLE sim.greeting_intents (
  id uuid PRIMARY KEY, aviary_id uuid NOT NULL,
  bird_id uuid NOT NULL, form text NOT NULL,           -- glance|call_short|approach|reorient
  absence_bucket text NOT NULL, stagger_ms integer NOT NULL,
  seed bigint NOT NULL, issued_at_tick bigint NOT NULL,
  consumed_at timestamptz                              -- single-consumption across devices (§6.4)
);
```

### 3.4 Event log

```sql
CREATE TABLE sim.interaction_events (
  id               bigint GENERATED ALWAYS AS IDENTITY,
  aviary_id        uuid NOT NULL,
  client_event_id  uuid NOT NULL,                 -- client-generated; idempotency key
  kind             text NOT NULL,                 -- presence_window|listen_in|offer|settle|rename|adopt
  bird_id          uuid,
  payload          jsonb NOT NULL,
  occurred_start   timestamptz NOT NULL,          -- client clock, server-clamped
  occurred_end     timestamptz NOT NULL,
  received_at      timestamptz NOT NULL,
  session_id       uuid NOT NULL,
  PRIMARY KEY (aviary_id, client_event_id)        -- exact-once ingest, replay-safe
) PARTITION BY RANGE (received_at);               -- daily partitions; 90-day retention, then drop
```

Append-only by grant: `sim_api_role` has `INSERT` and no `UPDATE`/`DELETE`. Retention is 90 days — long enough for a calibration audit (§13.1), short enough to honor the privacy posture. Personality does not depend on the log surviving; the vector is canonical (`bird_engine.md`), the log is transient input.

**Presence is stored as intervals, not points.** A `presence_window` event carries `[occurred_start, occurred_end]` asserted by the client only when all three conditions of the `concepts.md` definition held continuously. The tick computes presence-time as the **union of intervals across all devices**, never the sum — see §5.2. This is the single most consequential detail in this table.

### 3.5 Notebook

```sql
CREATE TABLE sim.notebook_entries (
  id uuid PRIMARY KEY, aviary_id uuid NOT NULL,
  observed_on date NOT NULL,               -- user-local date, for the "tuesday —" prefix
  created_at timestamptz NOT NULL,
  text text NOT NULL,                      -- final naturalist prose, immutable
  template_id text NOT NULL,               -- for anti-repetition and corpus review
  subject_bird_ids uuid[] NOT NULL,
  salience real NOT NULL,                  -- gate input; retained for tuning on shadow accounts only
  generator_version smallint NOT NULL
);
```

Read-only to the user by construction: no mutation endpoint exists, and `sim_api_role` holds `SELECT` only. Entries are never archived or hidden (`interactions.md`); the client paginates backward indefinitely.

### 3.6 Visits

```sql
CREATE TABLE sim.visit_invites (
  id uuid PRIMARY KEY, aviary_id uuid NOT NULL,
  invitee_email_ct bytea NOT NULL, invitee_email_bi bytea NOT NULL,
  token_hash bytea NOT NULL UNIQUE,
  created_at timestamptz NOT NULL,
  expires_at timestamptz NOT NULL,          -- created_at + 30 days
  revoked_at timestamptz, first_used_at timestamptz
);

CREATE TABLE sim.visit_sessions (
  id uuid PRIMARY KEY, invite_id uuid NOT NULL REFERENCES sim.visit_invites(id),
  started_at timestamptz NOT NULL, last_seen_at timestamptz NOT NULL,
  duration_bucket_minutes smallint          -- 5-minute buckets; approximate by design
);
```

`sim.visit_sessions` rows are never written into `sim.interaction_events`. A visitor's attention is structurally incapable of reaching the drift function because it lands in a different table that the tick does not read (`social_optional.md`).

### 3.7 Species configuration (code, not database)

Six species ship in v1, defined as versioned TypeScript config in `@aviary/species`: silhouette path data, default plumage palette ramp, motion-parameter ranges, and a motif library. One species has a nightjar-like signature and remains active into the late hours (`aviary_layout.md`). Config is versioned and additive-only; changing a species definition never re-identifies an existing bird (`sim.birds.id` and `identity_seed` are untouched).

---

## 4. API surface

REST/JSON over HTTP/2. `Cache-Control: private, no-store` on every authenticated response. Session cookie: `__Host-av_s`, `HttpOnly`, `Secure`, `SameSite=Lax`. Versioned at `/api/v1`. Snapshot schema carries its own `schema_version`.

**Transport decision: polling, not WebSockets.** The tick is ~60 s, snapshots are single-digit KB, and there is no co-presence requirement anywhere in the product (`social_optional.md` rules out the only feature that would need push). Polling with `ETag`/`304` is dramatically cheaper to operate, degrades gracefully on flaky mobile networks, and removes a whole class of connection-lifecycle bugs. Revisit only if a future feature needs sub-second server→client latency; none in v1 does.

### 4.1 Auth

| Endpoint | Behavior |
|---|---|
| `POST /api/v1/auth/magic-link` `{email}` | Always `204`, always the same latency envelope (constant-time padding). Never reveals whether an account exists. Rate-limited per email blind index (5/hour) and per IP hash (20/hour). |
| `GET /auth/consume?t=…` | Validates unexpired, unconsumed token; marks consumed atomically (`UPDATE … WHERE consumed_at IS NULL RETURNING`); issues session; `302` to `/`. Replay → matter-of-fact surface: *"We couldn't sign you in. The link may have expired. Try requesting a new link."* |
| `GET /api/v1/auth/sessions` | Device list with `device_label`, `created_at`, `last_seen_at`. |
| `POST /api/v1/auth/sessions/{id}/revoke` | Immediate; revoked session's next request → `401` and *"Your session timed out. Sign in again to keep watching."* |

### 4.2 Aviary state

```http
GET /api/v1/aviary/snapshot            → 200 Snapshot | 304 Not Modified
  If-None-Match: "av:<aviary_id>:<tick_seq>"
```

The same payload is **inlined into the initial HTML document** at the edge (§10.2) so the first render needs no round trip. The polling endpoint exists for subsequent refreshes.

Client pulls a snapshot on: (a) `visibilitychange` → visible; (b) a render-frame gap > 5 s (laptop suspend / throttled tab); (c) a 25 s keepalive while visible; (d) immediately after a `settle`, an `offer`, or a listen-in change, to pick up server-authored consequences. Nothing else. Hidden tabs do not poll at all.

```jsonc
// Snapshot (abridged) — note: NO trait values anywhere in this document.
{
  "schema_version": 1,
  "aviary_id": "…", "tick_seq": 84213, "server_time": "2026-07-26T14:03:11Z",
  "next_tick_eta_ms": 41000,
  "scene": {
    "solar_phase": 0.61,               // 0..1 through the user's local day
    "palette_key": "afternoon",
    "weather": { "kind": "rain", "intensity": 0.35, "ends_at": "…" }
  },
  "birds": [
    { "id": "…", "name": "pip", "species": "warbler_grey", "identity_seed": 771043,
      "mood_render": {                  // behavioral consequences of mood, not a label to display
        "posture": "upright", "scan_rate": 0.7, "fluff": 0.1, "eyes": "open" },
      "perch": { "zone": 0, "slot": 1 },
      "transit": null,
      "motion_energy": 0.41,            // derived from boldness+mood; NOT boldness
      "call_rate_per_min": 0.8,         // derived from vocal_frequency+mood
      "plumage_step": 3,                // quantized 0..7 — deliberately coarse (§4.4)
      "call_plan": [ { "at": "…", "seed": 99120223, "energy": 0.5 } ],
      "affinity": { "reply_prob": 0.42 }
    }
  ],
  "greeting_intent": { "bird_id": "…", "form": "approach", "stagger_ms": 0, "seed": 4471,
                       "id": "…" },     // present only if unconsumed (§6.4)
  "offers": { "cooldowns": { "<bird_id>": { "seed": 0, "song": 118, "pool": 0 } } },
  "settled": false
}
```

### 4.3 Interaction events

```http
POST /api/v1/aviary/events
{ "events": [
    { "client_event_id": "uuid", "kind": "presence_window",
      "start": "2026-07-26T14:01:02Z", "end": "2026-07-26T14:03:02Z" },
    { "client_event_id": "uuid", "kind": "listen_in",
      "bird_id": "…", "start": "…", "end": "…" },
    { "client_event_id": "uuid", "kind": "offer",
      "bird_id": null, "offer": "seed", "start": "…", "end": "…" },
    { "client_event_id": "uuid", "kind": "settle", "start": "…", "end": "…" }
] }
→ 202 { "accepted": 4, "duplicates": 0, "next_tick_eta_ms": 41000 }
```

Rules: `ON CONFLICT (aviary_id, client_event_id) DO NOTHING` — retries and multi-tab duplicates are free. Server clamps `start`/`end` to `[now − 10 min, now + 30 s]` and rejects windows longer than 120 s (a client that "wakes up" after a suspend cannot claim an hour of presence). Batches flush every 20 s while visible, on `visibilitychange` → hidden via `sendBeacon`, and on `pagehide`. Max 32 events/batch, 6 batches/minute per session.

**There is no endpoint that accepts a trait value.** No `PATCH /birds/{id}/personality` exists at any version. This is stated as an API-design rule so a future contributor has to argue against a written line rather than fill an obvious gap.

### 4.4 Why derived values, and why quantized

`bird_engine.md` forbids exposing personality numerically — "not in a stats panel, not in a debug view … there is no toggle for it." A snapshot that carried `boldness: 0.62` would make that promise false for anyone with a network tab. So the snapshot carries only consequences: `motion_energy`, `call_rate_per_min`, `plumage_step`, `reply_prob`. Each is a many-to-one function of multiple traits plus mood plus time of day, so it is not invertible into a trait. `plumage_step` is quantized to 8 levels specifically to prevent it from acting as a high-resolution readout of `plumage_saturation`, which is otherwise the trait with the most direct single-input rendering.

### 4.5 Notebook, account, settings

| Endpoint | Notes |
|---|---|
| `GET /api/v1/notebook?before=<cursor>&limit=30` | Reverse-chronological, cursor-paginated, unbounded history. No write verbs exist. |
| `GET /api/v1/account` | Email (masked), sessions, settings, aviary age. No trait data, no visit counts, no session counts. |
| `PATCH /api/v1/account/settings` | `{reduced_motion, captions, audio_enabled, always_show_controls, visit_notifications}` |
| `POST /api/v1/account/email` `{new_email}` | Stages `pending_email_*`; old email works until the new one verifies. |
| `POST /api/v1/account/export` | Enqueues generation; emails a signed, single-use, 24 h link to the **verified** address. Contains birds, names, current vectors, moods, notebook, settings. This is the one authorized place raw trait values leave the system, and it goes to the data subject, not to a UI. |
| `POST /api/v1/account/delete` | Sets `pending_delete`, `delete_after = now()+30d`. |
| `POST /api/v1/account/restore` | Available to any signed-in request during the window; the affordance is a quiet line on signed-in pages, matter-of-fact voice. |

Hard deletion runs as a daily job: cascade across `sim.*` and `iam.*`, purge the account's rows from event-log partitions, purge export artifacts from object storage, and record only a tombstone `{account_id_hash, deleted_at}` for audit. Telemetry rows contain no account dimension, so there is nothing to purge there — which is itself a benefit of the privacy boundary.

### 4.6 Visit invitation flow

```
host: POST /api/v1/visits/invites {email}
        → 201 {invite_id, expires_at}      (invite stored; mail-svc sends one-time link)
        → 409 if 5 active invites already outstanding (anti-spam cap)

visitor: GET /visit/{token}
        → 200 HTML shell + inlined VisitorSnapshot   (no account, no cookie required)
        → 410 if revoked/expired/consumed-invalid → matter-of-fact surface

visitor: GET /api/v1/visit/{token}/snapshot   (25 s keepalive, same as host client)
        → 200 VisitorSnapshot | 410 Gone

host: GET    /api/v1/visits/invites          → outstanding invites
      DELETE /api/v1/visits/invites/{id}     → 204, effective at the visitor's next pull
      GET    /api/v1/visits/log              → [{email, visited_on, duration_bucket_minutes}]
```

`VisitorSnapshot` is the host snapshot minus `greeting_intent`, `offers`, `settled`, and any account-scoped field. Birds render exactly as the host would see them — same moods, same drift-derived render parameters, no prettification (`social_optional.md`: no show-off mode). The visitor endpoint accepts **no** POST verbs of any kind; there is no code path from a visitor request to `sim.interaction_events`.

Abuse controls: invite emails are rate-limited per host (10/day) and per invitee blind index; the invite email does not disclose whether the invitee has an account; tokens are 256-bit, stored hashed, and single-invite scoped.

---

## 5. Simulation engine design

### 5.1 The tick

**Logical model.** Each aviary advances one simulation step per minute of wall-clock time, forever, independent of clients (`accounts_sync.md`).

**Physical model.** Running 60 writes/hour/aviary for every account regardless of activity is a large, mostly wasted bill. The resolution is a kernel property, not a shortcut:

> **Fast-forward equivalence.** `advance(state, t0, t1)` is deterministic and exactly equals the composition of per-minute steps from `t0` to `t1`, for all states and seeds.

This holds because every stochastic draw is keyed by `PRNG(identity_seed, minute_index, channel)` rather than by call order, and every continuous accumulator is closed-form integrable over a minute range. It is verified by a property-based test (`advance(s, t, t+n) === fold(step, s, n)` over 10⁴ random states × 10³ seeds) that runs on every commit and is a merge blocker.

Given that property, execution is tiered:

| Tier | Condition | Cadence |
|---|---|---|
| **Hot** | Client polled within 5 min, or unconsumed events exist | Every 60 s |
| **Warm** | Activity within 24 h | Every 15 min (fast-forwarded) |
| **Cold** | No activity > 24 h | Every 6 h (fast-forwarded), plus on-demand before any read |

A read of a cold aviary always fast-forwards to `now` before responding, so the user who returns after two weeks gets an aviary that *has been running for two weeks* — not one that resumes. The distinction the PRD cares about is preserved exactly; only the compute schedule differs.

**Execution mechanics.** `sim-tick` workers own hash ranges of `account_id`; each aviary is leased (Redis, 90 s TTL, fenced by `tick_seq`) so two workers cannot tick one aviary. A tick transaction: read state + unconsumed events (`id > applied_through_tick`) → compute → write `bird_traits`, `bird_state`, `weather_events`, `greeting_intents`, bump `tick_seq`, advance `applied_through_tick` → commit → invalidate snapshot cache. `tick_seq` increments monotonically and is the client's `ETag` basis.

**Per-tick pipeline order** (order matters and is fixed): ingest events → presence union → drift update → weather roll → mood transitions → bird-to-bird contagion → perch decisions → call scheduling → greeting-intent issuance (if a presence window just opened after a gap) → notebook observation pass → snapshot materialization.

### 5.2 Drift function

Traits are `t ∈ [0,1]`. New birds seed at `U(0.25, 0.45)` per trait, biased ±0.08 by species, drawn from `identity_seed` so adoption is reproducible.

**Stage 1 — low-pass the signal.** Per bird, per trait, maintain a smoothed signal `S`:

```
S_k = (1 − α)·S_{k−1} + α·x_k          α = 3.5e-4 per minute-step  (τ ≈ 2 days)
```

`x_k ≥ 0` is the normalized per-step input, mixed from:

| Input | Weight | Normalization |
|---|---|---|
| Presence-time (union across devices) | 1.00 | minutes present in step ÷ 1.0, credited-capped at 90 min/day |
| Listen-in on this bird | 0.55 → `social_warmth`, `vocal_frequency` | seconds ÷ 60, capped 20 min/day/bird |
| Offer accepted by this bird | 0.30 → `curiosity` | per event, cooldown-limited |
| Offer made while this bird present | 0.15 → `boldness` | per event |
| Settle | 0.00 | mood-quieting only; no drift direction (`bird_engine.md`) |

`plumage_saturation` is driven by presence alone — it is the "sustained attention" trait.

**Stage 2 — accumulate monotonically.**

```
Δt = η · (1 − t) · S_k                  η = 0.0016 per minute-step
t  ← min(1, t + max(0, Δt))
```

Four properties fall out, each mapping to a PRD requirement:

- **Monotone.** `S ≥ 0` and `max(0, ·)` mean `t` never decreases. Neglect drives `x → 0`, `S` decays with a 2-day time constant, `Δt → 0` — the bird stops becoming *more* expressive but loses nothing. This is `bird_engine.md`'s asymmetry, implemented rather than asserted.
- **Saturating.** `(1 − t)` slows growth near the ceiling, so a very heavy user's birds approach an asymptote instead of pinning.
- **Un-farmable within a session.** The 2-day filter time constant means a single 3-hour session moves `S` a fraction of the way toward its steady state; combined with a hard per-trait ceiling of `Δt ≤ 0.006/day`, no session produces a visible change. This is the mechanical form of "no single session shifts a trait visibly."
- **Calibratable.** `α`, `η`, weights, and caps live in a versioned `CalibrationSet` recorded on each bird's row.

**Calibration targets** (`bird_engine.md`: measurable at ~1 week, visible at ~3 weeks). Define the reference profile *regular visitor* = 5 sessions/week × 12 min presence + 2 listen-ins + 1 offer.

| Horizon | Target | Test |
|---|---|---|
| 1 week | `Δt ≥ 0.020` on presence-driven traits (≥ 20× the 1e-3 instrument noise floor) | Calibration lab assertion |
| 3 weeks | Cumulative `Δt` crosses the per-trait **perceptual threshold** | Perceptual-mapping assertion |
| 3 weeks, heavy user (3× reference) | `Δt` ≤ 2.2× the reference user's | Anti-farming assertion |
| 2 weeks absent after 3 weeks present | `Δt = 0` exactly; call rate down ≥ 35%; greeting probability down ≥ 40% | Regression-free-quietness assertion |

**Perceptual thresholds** are defined on the *rendered consequence*, because that is what a user actually perceives:

| Trait | Rendered consequence | Perceptible when |
|---|---|---|
| boldness | P(front perch) over a session | shifts ≥ 15 pp |
| social_warmth | P(greets first \| greeting occurs) | shifts ≥ 15 pp |
| vocal_frequency | calls/min when unobserved | shifts ≥ 25% relative |
| plumage_saturation | `plumage_step` | advances ≥ 1 step |
| curiosity | P(approach within 20 s of an offer) | shifts ≥ 15 pp |

The mappings from trait to consequence are smooth (logistic, no thresholds inside the mapping) so drift never "snaps." `plumage_step` is the exception — it is quantized — and so it uses 1.5 pp of hysteresis at each boundary to avoid flicker on a bird sitting exactly on a step edge.

### 5.3 Mood transitions

Six states: `wary`, `content`, `curious`, `drowsy`, `alert`, `roosting`. (`bird_engine.md` lists five and leaves the set open; `roosting` is added for the full-night state `aviary_layout.md` describes as birds "settled, eyes closed, low on the perch." See §14.) **No mood name ever appears in the UI, narration, captions, or notebook.** Mood is read from motion (`bird_engine.md`); the moment we render a label, that contract is broken.

Each step computes a logit per candidate mood and samples softmax with temperature 0.6:

```
logit(m) = Σ_i w_i · f_i(m)   over
  timeOfDay(m, solar_phase)          // drowsy↑ at dusk, alert↑ early morning, roosting gated to night
  recentImpulse(m)                   // decaying accumulator from events, half-life 12 min
  weather(m, active_event)           // rain: vocal↓ + content↑; wind: alert↑ for some species, wary↑ others
  contagion(m, neighbors)            // alarm/wary spreads, weight ∝ neighbor social_warmth, ∝ 1/(1+boldness)
  personalityPrior(m, traits)        // high boldness strongly suppresses wary; high curiosity raises curious
  baselinePull(m)                    // toward a personality-conditioned baseline, τ ≈ 20 h → the "daily-ish reset"
  + speciesBias(m)
```

Guards: **minimum dwell** of 4–12 min (scaled by boldness — bolder birds settle faster) blocks flicker; **transition damping** requires the winning logit to exceed the incumbent by a margin; **roosting** is admissible only when local solar elevation is below the night threshold, except for the nightjar-species, which stays admissible for `alert` and `curious` into the late hours.

Mood persists trivially because it is server state that keeps advancing during absence (`bird_engine.md`). There is no "session start" event in the mood machine at all — which is precisely why the user never sees mood snap to a default on tab open. The absence of a reset path is enforced by a test that asserts no code writes `bird_state.mood` outside the tick.

### 5.4 Call-grammar runtime

**Layers.**

1. **Motif library** (per species, static config): 4–7 motifs. A motif is a sequence of syllables; a syllable is `{contourPoints[3..5] (semitones over normalized time), durationMs, ampEnvelope(ADSR), timbre{harmonicWeights, fmIndex, noiseRatio, formantHz}, vibrato{rateHz, depthCents}}`.
2. **Per-bird identity parameters** — derived once from `identity_seed`, **frozen for the life of the bird**: pitch center (±4 semitones from species base), timbre fingerprint (harmonic weight vector + formant offset), characteristic interval, syllable-duration ratio, and the motif-transition weights of a small Markov chain over the species' motifs.
3. **Expressive modulation** — what mood and drift are allowed to touch: phrase length, inter-syllable timing, overall energy/amplitude, pitch *spread* (not center), call rate, and reply probability.

> **The recognizability invariant: drift and mood modulate expression; they never touch identity.** This is what makes "a user who has spent two weeks with Pip should know Pip's call by ear, even when Pip's vocal frequency has drifted up" (`bird_engine.md`) true by construction rather than by luck. It is tested in §12.4 with a spectral-fingerprint assertion across the full mood × drift cross-product.

**Scheduling.** The server issues *intent* — `call_plan` entries with `{at, seed, energy}` for the next ~90 s — so every device agrees about who calls when and the caption text matches the audio. The client realizes intent with a lookahead scheduler (25 ms tick, 150 ms horizon) against `AudioContext.currentTime`, applying a ±180 ms humanizing jitter derived from the same seed.

**Chorus** is emergent, not scripted: overlapping `call_plan` windows from birds with high vocal frequency produce simultaneous calls. Two rules keep it musical rather than muddy: a **density limiter** (no more than 3 concurrent voices; a 4th defers 400–900 ms) and an **anti-unison rule** (two onsets within 250 ms of each other are separated unless the second is a call-and-response reply).

**Call and response.** A bird with high `social_warmth` schedules a reply 0.6–2.5 s after hearing a neighbor, with pitch biased toward its own characteristic interval relative to the heard call. This is the audible form of "the aviary is a small social system rather than a row of independent NPCs."

### 5.5 Return-greeting

The greeting gets specific treatment because `interactions.md` gives it multi-paragraph weight and explicitly warns against compressing it into "play arrival animation."

**Absence** `A` = `now − last presence-window end`, computed **server-side** (the client cannot know about presence from another device).

| A | Form | Character |
|---|---|---|
| < 2 min | *none* | The aviary just continues. Stepping away for ten seconds is not an arrival. |
| 2–20 min | `glance` | A look up from preening. No call. |
| 20 min – 6 h | `call_short` | Glance plus a two-note call. |
| 6–48 h | `approach` | A step toward the front perch plus a longer call. |
| > 48 h | `reorient` | Longer call, move to front, elevated chance a second bird answers. |

**Greeter selection** — softmax sample over eligible birds:

```
score(b) = 1.1·boldness(b) + 0.9·social_warmth(b) + moodAffinity(mood(b))
         − 0.6·recentGreeterPenalty(b)          // rotates the first-greeter across days
         − 0.8·[mood(b) == wary] − 1.4·[mood(b) == roosting]
         + 0.25·noise(PRNG(identity_seed, day_index))
```

A wary or low-boldness bird can fail the eligibility gate entirely and simply not greet today — required by `interactions.md` ("the warier bird greets later or not at all on a given day"). **One bird greets**, not all of them. If a second bird responds, its stagger is 800–2600 ms, jittered, never within 250 ms of the first — a simultaneous chorus on cue would announce the arrival, which is the wrong register.

**Variation is real, not rotational.** The greeting composes a call from the grammar and a motion path from the motion synthesizer, both seeded per greeting. A test asserts that 1,000 sampled greetings for one bird under fixed mood/traits yield > 950 distinct `(motif sequence, quantized timing, motion path)` tuples. Three pre-recorded variants in rotation would fail this test, which is the point of having it.

**Single consumption across devices.** `greeting_intents.consumed_at` is set by the first client to acknowledge it. A phone opened 30 seconds after the laptop does not replay the same greeting; it sees the aviary already continuing. See §6.4.

### 5.6 Field notebook generation

A post-tick observer pass, deliberately stingy.

1. **Candidate detection** over a rolling 24 h window: first-greeter changed from the usual bird; a bird reached the front perch for the first time this week; an unusually long quiet stretch; a chorus with ≥ 3 participants; an offer approached by a bird that usually waits; weather co-occurring with a mood shift; a bird's first call after a long silence.
2. **Salience scoring**: `rarity × recency × subject-diversity`, with a novelty penalty against the last 30 entries' `template_id` and subject birds.
3. **Sparsity gate**: emit only if `salience > θ` **and** ≥ 40 h since the last entry — unless salience is in the top 2% (a genuinely notable morning), which allows a floor of 16 h. Target ≈ 1 entry / 2–4 days for a regular visitor, per `interactions.md`. The gate is tuned so that an unusually active user does not get a feed; the sparsity is the feature.
4. **Realization** through a template lattice — slot-filled naturalist prose, lowercase, present-tense, bird-named, with a lexicon varied by species, weather, time of day, and mood-derived behavior. Optional weekday prefix (`"tuesday — "`) on ~30% of entries.

**Forbidden by lint on generated output** (fails the build, not review): any digit that refers to the user; the words *streak, visit, visited, day(s) in a row, welcome, back, achievement, unlocked, session, logged, score, level*; second-person pronouns; any percentage. The line `interactions.md` draws is exact and the linter encodes it: **the notebook makes observations of the aviary, never observations of the user's behavior.** "pip greeted before wren today" is in bounds; "you've been here every day this week" is not, in any phrasing.

Each release regenerates a 500-entry corpus from shadow accounts (§13.3) for human review. A corpus that reads like an event log fails the release gate.

---

## 6. Sync model

### 6.1 Why there is almost nothing to sync

Because the server is the only writer of canonical state and clients render snapshots, multi-device sync is a *property* rather than a *feature* (`accounts_sync.md`). There is no client-to-client channel, no CRDT, no merge, no vector clock, no last-write-wins arbitration. Two devices are two readers of one row.

### 6.2 The four mechanisms that make it correct

1. **Additive server-authored deltas.** Clients submit observations ("listened in to Pip for 3 minutes"); the server decides what that means. No client-submitted absolute value is accepted by any endpoint. The overwrite failure `accounts_sync.md` describes — phone's stale read clobbering the laptop's morning drift — is unreachable because the write shape that would cause it does not exist in the API.
2. **Ordered, exactly-once consumption.** The event log is append-only and ordered by `(aviary_id, id)`. `applied_through_tick` is the watermark. Duplicate submissions collapse on the `(aviary_id, client_event_id)` primary key. A tick retry after a crash reprocesses the same range and produces the same result (the kernel is deterministic), then commits once.
3. **Presence union, not sum.** Two devices with the aviary open and the user actually in front of one of them must not double-credit presence. The tick merges all `presence_window` intervals into a union before feeding the drift function. This is the single most important line of sync code in the product: getting it wrong doubles the drift rate for exactly the multi-device users the sync model exists to serve, and it would be invisible in every test that runs one client.
4. **Monotonic `tick_seq` as the only version.** Every snapshot carries it; the `ETag` derives from it; the client discards any snapshot with a `tick_seq` lower than the highest it has seen. Out-of-order responses on a flaky mobile network cannot make the aviary go backwards.

### 6.3 Client-side reconciliation

The client keeps a two-entry snapshot buffer and renders `t_render = now − 120 ms`. On a new snapshot the client does not jump: perch and pose targets are handed to critically damped springs (ζ = 1.0, ω tuned per property, 250–900 ms settling), so a correction reads as the bird changing its mind, not as a network artifact. If a delta is too large to absorb (bird moved zones while the tab was hidden), the client renders a real flight transition rather than a teleport.

### 6.4 Single-consumption events across devices

Greeting intents, offer reactions, and settle acknowledgments are one-shot. Each carries an id; the first client to render it `POST`s an acknowledgment and the server sets `consumed_at`. Other devices receive a snapshot without it. This prevents the "opened on my phone and got greeted again" artifact, which would break the fiction that the aviary is one place.

### 6.5 Failure surfaces

All matter-of-fact voice (`product_brief.md`, `accounts_sync.md`), rendered in chrome and never inside the aviary scene:

| Condition | Surface |
|---|---|
| Expired/replayed magic link | *"We couldn't sign you in. The link may have expired. Try requesting a new link."* |
| Session revoked / expired | *"Your session timed out. Sign in again to keep watching."* |
| Snapshot fetch fails ≥ 3× | *"Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."* |
| Visit revoked/expired | *"This visit is no longer available."* |

Transient single failures show nothing at all. The client keeps rendering from the last snapshot and extrapolating (§7.4) — a brief network blip should be invisible, because a place does not display an error when a packet drops. The error surface appears only after the client can no longer honestly claim to be showing the aviary.

---

## 7. Frontend rendering pipeline

### 7.1 Technology choices

| Choice | Decision | Reason |
|---|---|---|
| Scene renderer | **Canvas2D**, single canvas, DPR-aware | ≤ 7 birds + ~4 parallax layers + ~20 ornaments is well inside Canvas2D's budget on 2020 hardware; WebGL would cost bundle, complexity, and a driver-bug surface for effects the PRD explicitly does not want (it is "not parallax-heavy"). |
| Scene framework | **None.** Plain TS + a fixed-step loop | Zero framework overhead on the critical path; the scene is not a component tree. |
| Chrome/settings framework | **Preact** (~4 KB gz), code-split | Top bar, notebook, settings, account, invites are ordinary UI. They are not on the first-bird path. |
| Assets | **Vector path data → runtime-rasterized pose atlases** | Ships kilobytes instead of sprite sheets; keeps the bundle ceiling comfortable; lets plumage saturation modulate rendering rather than requiring per-level art. |

### 7.2 Layers and composition

| Layer | Parallax | Contents |
|---|---|---|
| Sky | 0.0 | Day/night gradient, recomputed at most 1/s, cached to an offscreen canvas |
| Far foliage | 0.25 | Soft shapes, slow wind response |
| Mid plane | 1.0 | Perches (3 zones × slots), **birds**, still-pool offer |
| Near ornament | 1.4 | Occasional branch, drifting leaves and feathers |

Parallax offset is driven by pointer position (max ±6 px) plus a slow ambient sway. The scene is one screen, never panned, scrolled, or zoomed. Responsive layout re-solves perch positions from a normalized layout model so every bird stays fully in frame at any viewport; birds are never cropped and never drift offscreen (`aviary_layout.md`). Narrow viewports compress horizontal spacing and reduce perch-slot separation; they never crop.

### 7.3 Bird rendering and idle micro-motion

Each bird is a five-part rig (body, head, tail, near wing, far wing) with a procedural feather-detail overlay whose density and palette saturation are driven by `plumage_step`.

Motion is **synthesized, never looped**:

- **Breathing** — 1-D simplex noise, period 3.5–6 s, phase from `identity_seed`, amplitude scaled by mood.
- **Head micro-saccades** — Poisson-scheduled, rate from mood `scan_rate`, with a small settle overshoot.
- **Weight shuffle** — semi-Markov, λ mood-scaled; the small foot-reset a perched bird does.
- **Preen bouts** — semi-Markov, entered more often in `content`, with variable bout length.
- **Sound-tilt** — a head turn toward another bird's call onset, probability scaled by `curiosity`.
- **Flight** — arc path with squash/stretch, wing-beat frequency by species, easing keyed to `motion_energy`.

Mood is legible from motion alone: `wary` sits further back and scans more; `content` preens; `curious` tilts and tracks passing leaves; `drowsy` sits low and fluffed; `roosting` closes eyes and lowers on the perch. This is the whole delivery mechanism for mood — there is no label, tooltip, icon, or status indicator anywhere in the scene, ever (`bird_engine.md`, `aviary_layout.md`).

Because every motion channel is noise- or Poisson-driven with per-bird phase, no two birds are ever in step and no bird ever repeats a cycle. A CI check flags any timeline-based animation primitive appearing in the scene package.

### 7.4 The first frame — "already in motion"

This is the central conceit rendered as an engineering sequence (`aviary_layout.md`). There is **no spinner, no fade-from-static, no entry animation, no "ready" state**.

1. **Edge-rendered document** carries an inlined snapshot and an inlined bootstrap (≤ 20 KB gz) containing the Canvas2D loop, one species' path data, and the pose sampler.
2. **Warm seed (repeat visits).** The last snapshot is persisted to IndexedDB. On navigation, before the network resolves, the bootstrap fast-forwards that cached snapshot locally using `sim/core` (mood/pose/scheduling only — never drift) and draws the first frame **already in motion**: a bird mid-preen, another calling from the high perch, a leaf crossing frame. When the server snapshot lands (typically 60–200 ms later) it is reconciled through the springs of §6.3 — invisibly. This does not violate the client-never-owns-state rule: the warm seed is render-only, never written back, and superseded on arrival.
3. **Cold start (first visit, cold cache).** The inlined snapshot is authoritative and the first frame draws from it directly.
4. **Genuinely slow network.** The fallback is the **quiet field** the PRD specifies: soft sky gradient at the correct time of day plus one or two faint motion cues. Never a spinner, never a progress bar, never a percentage. A spinner says machine.

**The word `spinner`, and any loading-indicator primitive, is banned from the codebase by the invariants suite (§12.6).** This is the failure mode `aviary_layout.md` names explicitly, and it is the one most likely to be introduced by a well-meaning contributor reaching for the safe pattern.

The **empty-aviary state** (post-adoption, pre-first-bird) uses the same quiet field; the first bird then enters with a soft fly-in to its starting perch. After that the user never sees an empty aviary again.

### 7.5 Day/night, weather, ornament

Solar phase is computed from the device's timezone offset against a generic civil-twilight curve — deliberately no geolocation request, no IP geolocation, no latitude (§14). Sunrise warms gradually, midday is brightest, evening warms and quiets, night dims. Palette interpolation runs on a ~1 s cadence; there are no discrete palette switches.

Weather is server-authored (so devices agree) and rendered as rain streaks with a wet-sheen shift, or wind as increased foliage response and leaf frequency. Intensity is capped at 0.6: no thunder, no snow, nothing the user must notice.

Leaves and feathers are pure client ornament on an idle cadence (`aviary_layout.md`) — allocated from a fixed pool, never per-frame.

### 7.6 Top bar

Four icons only: account/settings, accessibility settings, field notebook, offer. Nothing else, ever. The bar sits *above* the aviary scene; no chrome is drawn inside the scene.

Fade: after 3.5 s of cursor stillness and no keyboard activity, the bar eases to a floor opacity over 600 ms; it returns to full on pointer move, keyboard activity, or focus entering it. The fade is suppressed entirely when keyboard focus is inside the bar, when `always_show_controls` is on, or in reduced-motion mode. The floor opacity is **not** near-zero: it is set so that composited glyph contrast stays ≥ 3:1 against both the brightest and dimmest sky states we ship (§9.5) — the PRD asks for "nearly transparent," and we honor the intent while refusing to ship controls that fail non-text contrast.

### 7.7 Reduced-motion mode

Not a flag threaded through the renderer — a **second sampler behind one interface** (`MotionProfile`), so the scene graph, state, audio, drift, notebook, and narration are identical and only the temporal sampling differs.

| Aspect | Full | Reduced |
|---|---|---|
| Idle micro-motion | Continuous noise-driven | Slow cross-fades between still poses, 0.9–1.4 s |
| Flight | Animated arc | Cross-fade between perch poses |
| Leaf/feather drift | Present | Removed |
| Parallax | ±6 px | 0 |
| Day/night ramp | Continuous | Continuous, 4× slower |
| Calls, drift, mood, notebook | Full | **Identical — unchanged** |

Activated by `prefers-reduced-motion` **or** the accessibility setting; the setting can also force full motion for a user whose OS-level preference is set for unrelated reasons. Reduced-motion is a designed register with its own calm — not a degraded fallback (`accessibility_perf.md`), and it ships in the launch build, not as a v1.1 fix.

### 7.8 Loop and lifecycle

`requestAnimationFrame` loop with a fixed 16.67 ms simulation substep and an interpolated draw. Hidden tab: cancel rAF, suspend the `AudioContext`, stop polling, flush pending events via `sendBeacon`. Visible again: pull a snapshot, resume audio, and **resume rendering at the current state with no entry animation** — the same rule as first paint. `pagehide`/`freeze` are handled for bfcache; restoration re-pulls rather than resuming stale state.

---

## 8. Audio pipeline

### 8.1 Graph

Native WebAudio nodes throughout; no `AudioWorklet` in v1 (avoids a class of cross-browser lifecycle bugs; node counts are well within budget, and this is revisited only if profiling demands it).

```
per voice (pooled ×8):
  osc_a (triangle/saw) ┐
  osc_b (detuned)      ├─▶ partial gains ─▶ formant biquad ×2 ─▶ voice ADSR gain ─┐
  noise band (breath)  ┘                                                          │
                                                          per bird bus ◀──────────┘
                            per bird bus: gain ─▶ lowpass (listen-in "distance") ─▶ stereo panner (by perch zone)
                                             └─▶ reverb send ─▶ shared convolver (tiny procedural IR)
                                                          │
                            aviary bus ─▶ master gain ─▶ compressor (gentle) ─▶ destination
                            ambient bed ─▶ aviary bus     (procedural, non-looping, very low level)
```

Voices are **pre-allocated at startup and recycled**; nothing in the audio path allocates per call. This is a direct dependency of the no-memory-growth budget (§10.4).

### 8.2 Synthesis

A call renders as a syllable sequence sampled from the bird's Markov chain over its species' motifs, with identity parameters frozen and expressive parameters modulated by mood and drift (§5.4). Per syllable: pitch contour → `setValueCurveAtTime` on the oscillator frequencies; amplitude → ADSR on the voice gain; timbre → partial gain weights + formant biquad frequencies; vibrato → a low-frequency oscillator on detune. All values derive from `PRNG(identity_seed, call_seed, syllable_index)`, so the call is exactly reproducible — which is what allows the caption (§9.3) to describe precisely what was played.

**No recorded audio exists in the product, at any quality, in any fallback path.** `bird_engine.md` and `accessibility_perf.md` are unconditional. A CI check fails the build on any audio file in the bundle or any `decodeAudioData` call site.

### 8.3 Chorus

Chorus is what emerges when overlapping `call_plan` windows fire, plus call-and-response. Two mixing rules keep it a chorus rather than a pile: the density limiter (≤ 3 concurrent voices, 4th defers 400–900 ms) and the anti-unison rule (onsets within 250 ms are separated unless the second is a reply). Per-bird panning by perch zone (front narrow/center, back wider/quieter) plus per-bird reverb send by zone gives the chorus depth without stereo gimmickry.

### 8.4 Listen-in mix

Engage: the focused bird's bus → 0 dB; all other bird buses → **−12 dB, never to silence** (`interactions.md`: a re-balance, not a mute); the ambient bed is untouched. Unfocused buses additionally sweep their low-pass from 8 kHz → 3.5 kHz and increase reverb send, so the quieting reads as *distance* rather than as a fader move. All ramps use `setTargetAtTime` with τ ≈ 0.35 s (≈ 900 ms perceived), on both engage and disengage. A hard cut would turn the aviary into a set of soloable tracks, which is a different product.

Disengage on: clicking the focused bird again, focusing a different bird, clicking empty scene, moving keyboard focus away, or `Escape`. Listen-in duration is emitted as an interaction event on end (drift input per §5.2).

### 8.5 Autoplay policy — the one genuinely hard problem

Browsers suspend `AudioContext` until a user gesture. The PRD says calls are "already audible" on the first frame. These conflict, and the resolution has to avoid the obvious fix, which is a "click to enable sound" banner — an announcement, and a bad one.

**Resolution.** The context is created suspended. The scene renders fully, in motion, from frame one; captions are shown while audio is locked (regardless of the caption setting). On the first genuine user gesture anywhere — pointer, key, tap, including the gestures the user makes for entirely other reasons — the context resumes, and audio **joins mid-phrase of whatever is currently scheduled**, not from the start of a call. The user's experience is of turning toward a room that was already making noise, which is the correct affective register. No banner, no prompt, no icon-with-a-slash, no "sound on" toast, ever.

Where the browser grants autoplay (media engagement heuristics, returning users, PWA-installed contexts), audio starts immediately and this path is never exercised. This is the highest-risk affective detail in the build and is tracked as R5 in §13.

### 8.6 WebAudio unavailable

If `AudioContext` construction fails, is blocked, or errors on resume: the aviary plays in **graceful silence with captions on by default** (`accessibility_perf.md`). No recorded-audio fallback path is shipped. No error surface appears in the scene; the accessibility settings panel states the audio status matter-of-factly for a user who goes looking. One aggregate `audio_context_failure` metric is emitted.

---

## 9. Accessibility surfaces

The stance from `accessibility_perf.md` governs every choice below: the accessible surface is the *actual product*, not a semantically-labeled variant of it. Accessibility work is in the launch build; nothing here is deferred.

### 9.1 Architecture

The narration composer, the caption generator, and the visual renderer all read the **same interpolated scene state object**. They cannot diverge, because there is only one state. A parallel "accessible description" pipeline fed by different data is exactly how these surfaces drift apart over a year of changes, and the single-source rule is the structural prevention.

The scene canvas is `aria-hidden`. Beside it sits a **shadow accessibility tree**: a visually-hidden-but-focusable list of birds, plus two live regions. Focus, hit-testing, and AT semantics all operate on real DOM; the canvas is purely a paint surface.

```html
<div class="aviary">
  <canvas aria-hidden="true"></canvas>

  <ul class="a11y-birds" aria-label="the aviary">
    <li><button id="b1" tabindex="0"
        aria-describedby="cap-b1">pip, a small grey warbler, on the front perch, calling softly</button></li>
    <li><button id="b2" tabindex="-1">wren, further back, feathers fluffed</button></li>
  </ul>

  <div id="narration" aria-live="polite" aria-atomic="true"></div>
  <div id="events"    aria-live="polite" aria-atomic="true"></div>
</div>
```

### 9.2 Screen-reader narration

Running naturalist prose, not a state list (`accessibility_perf.md`):

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Generation: a template lattice over `{species descriptor, perch zone, mood-derived behavior, time of day, weather, recent notable event}`, with the same lowercase present-tense voice as the notebook, and an anti-repetition ring buffer (no template or distinctive phrase reused within 12 utterances).

Cadence: one update every 30–60 s at idle, jittered. User-initiated events (return-greeting, offer reaction, settle, listen-in engage) get a priority bump into the `#events` region, whose content is cleared before writing so a burst never queues up behind itself.

**`aria-live="assertive"` is never used anywhere in the product.** Assertive interrupts whatever the user is listening to — it is the auditory equivalent of a toast, and the "notice, never announce" principle applies with at least as much force in this modality. This is enforced by the invariants suite.

Priority events are still written as *observations*: "pip steps toward the front rail and calls" — not "greeting event: pip."

### 9.3 Call captions

Opt-in from accessibility settings; **on by default** whenever audio is unavailable or locked (§8.5, §8.6).

Captions are generated at runtime from the call's actual parameter set — never a per-call fixed string (`accessibility_perf.md`). The generator maps synthesis parameters to prose:

| Parameter | Contribution |
|---|---|
| syllable count | "single", "two-note", "three-note", "a run of notes" |
| contour direction | "rise", "fall", "rise and fall", "level" |
| dynamics / energy | "soft", "clear", "sharp" |
| timbre (noise ratio, formant) | "trill", "whistle", "buzz", "burr" |
| inter-syllable gaps | "paused", "quick" |
| perch zone | "from the back perch" |

Producing exactly the register the PRD samples: *"a soft three-note rise"*, *"a low trill, paused, low trill again"*, *"a single sharp call from the back perch."*

Rendering: DOM text (not canvas) near the calling bird's projected position, on a subtle scrim that guarantees AA contrast against every sky state, fading in and out with the call. Captions are also mirrored into the bird's `aria-describedby` so a screen-reader user gets them inline rather than as a separate stream.

### 9.4 Keyboard navigation

Exactly as `accessibility_perf.md` specifies. Tab moves through top-bar items; Tab into the scene focuses the first bird; arrow keys move focus between birds (roving `tabindex`, ordered by on-screen position); `Enter` triggers listen-in on the focused bird; `Escape` exits listen-in; the offer affordance opens from the top bar and is fully keyboard-navigable; settle is reachable from the top bar. The settle 5-second undo is triggerable by any key as well as any click.

Focus indication is drawn twice — a soft high-contrast outline painted on the canvas around the bird, and a matching outline on the shadow DOM element — so browser/AT focus tracking and the visual scene agree. The outline uses a dual-tone (light core, dark halo) treatment so it reads against both the brightest midday sky and the dimmest night.

### 9.5 Contrast

All user copy — top-bar labels and tooltips, settings, account and error surfaces, captions, visible narration — meets WCAG AA (4.5:1 body, 3:1 large). Non-text UI components meet 3:1, **including the top bar in its faded state** (§7.6), which is why the fade has a floor rather than going to near-zero. The design-system spec owns the exact ratios per surface; the CI contrast check (§12.5) evaluates every copy token against the full set of shipped sky palettes, at both fade extremes, in both motion modes.

---

## 10. Performance budgets and observability

### 10.1 Budgets

| Budget | Limit | Internal target | Gate |
|---|---|---|---|
| Initial JS at first paint | **≤ 2 MB gz** (PRD ceiling) | ≤ 420 KB gz total; ≤ 20 KB gz inline bootstrap | `size-limit` in CI, blocking |
| Time to first bird visible | **< 500 ms** (mid-tier mobile, 4G) | p75 ≤ 380 ms, p95 ≤ 480 ms | Lab CI on throttled device profile, blocking; RUM in prod |
| Idle frame rate | **60 fps** on a 5-year-old mid-range laptop, sustained 30 min | ≤ 6 ms main-thread per frame at 7 birds | Nightly soak on reference hardware |
| Memory over 30 min | **No growth** | Heap slope < 0.4 MB/min; detached nodes flat; audio nodes constant | CI soak test, blocking |
| Snapshot payload | — | ≤ 6 KB gz at 7 birds | Contract test |
| Tick latency | p99 < 5 s (alarm threshold) | p50 < 120 ms, p99 < 800 ms | Prod alarm |

The 2 MB figure is a ceiling, not a target. Treating it as a target would put time-to-first-bird out of reach on the same device class the PRD names, since 2 MB of gzipped JS costs far more in parse/execute on a mid-tier phone than in transfer. Code-splitting is aggressive: account settings, accessibility settings, the notebook, the offer panel, and the visit-invitation flow are all separate chunks, prefetched on idle, never on the critical path.

### 10.2 Time-to-first-bird budget breakdown

| Window | Spend | Mechanism |
|---|---|---|
| 0–120 ms | Network + edge TTFB | Edge-computed document (per-account, so not CDN-cacheable — this is edge *compute*, not a cache hit), early hints/preconnect, HTML streamed |
| 120–200 ms | HTML parse + bootstrap execute | ≤ 20 KB gz inline: loop, sampler, one species' path data, inlined snapshot |
| 200–330 ms | First canvas draw | Draw directly from `Path2D` for the first bird; atlas rasterization deferred to `requestIdleCallback` |
| 330–500 ms | Margin | Absorbs slow TLS, cold edge, mid-tier CPU variance |

Repeat visits are substantially faster because the warm seed (§7.4) draws before the network resolves. Everything non-critical — remaining species atlases, audio graph construction, chrome chunk, notebook prefetch — happens after the first bird is on screen.

### 10.3 Frame budget

At 7 birds: ~30 draw calls (4 layers + 7 birds × ~2 + ornaments), each from a cached atlas or a short `Path2D`. Sky gradients cached to offscreen canvases, redrawn at most once per second. Motion sampling is O(birds), a few hundred float ops. Audio scheduling runs on a 25 ms `setTimeout` lookahead loop, well off the frame path. The dominant risk is text and caption layout thrash, so captions use `transform`-only positioning with no layout reads inside the frame.

### 10.4 No memory growth

Enforced structurally: a fixed voice pool (§8.1); a fixed ornament pool; pre-allocated typed arrays for motion state; the notebook list virtualized with recycled rows and no retained references after scroll-out; a bounded snapshot buffer (2 entries); exactly one `AudioContext` for the page lifetime; every `setTimeout`/`rAF`/listener owned by a disposable scope tied to the scene lifecycle. The CI soak drives a 30-minute session headless with heap sampling and fails on slope, on detached-node growth, or on audio-node count drift. `accessibility_perf.md` calls this "a real test in CI, not a guideline" — it is a merge blocker.

### 10.5 What we measure

**Synthetic**: a scheduled fleet of automated browsers from several geographies running full sessions — first-bird timing, frame-time distributions, audio-context success, sign-in success, snapshot latency.

**RUM, aggregate only**: page-load timings, first-bird-render, frame-time histograms, audio-context error counts, snapshot fetch latency/error rates, event-submission error rates, client error rates by release.

**Server**: request rate/latency/error by route, tick latency (p50/p99), **tick lag** (`now − last_tick_at`, p99 — the metric that actually detects "the aviary stopped running," which tick latency alone would miss), event-log ingest rate, queue depth, DB saturation, magic-link issue/consume/expire counts, invite issue/redeem/revoke counts, export job success.

Alarms: tick latency p99 > 5 s (PRD-specified); tick lag p99 > 10 min; first-bird p75 > 500 ms sustained 15 min; audio-context failure rate > 2%; magic-link consume success < 95%; any DB write to `sim.bird_traits` by a role other than `sim_tick_role` (this should be impossible — an alarm on the impossible is how you find out it became possible).

### 10.6 What we deliberately do not measure

This list is as much a part of the design as §10.5, and it is enforced at the metric-definition layer rather than by policy (`accounts_sync.md`).

- **No per-account dimension on any metric.** Not as a label, not as a high-cardinality tag, not "just for debugging."
- **No per-bird state anywhere in telemetry.** No mood distributions, no trait aggregates, no "average drift across accounts" — the PRD names that specific anodyne-sounding dashboard as forbidden, and it is.
- **No per-user interaction counts, session counts, retention cohorts, or visit frequencies.** These are the raw material of a streak counter; not computing them means the feature cannot be "just exposed" later.
- **No cross-account aggregation of visits, bird counts, or aviary ages** — the metrics a leaderboard would need (`social_optional.md`).
- **No per-bird interaction data in any ML or training pipeline, ever.**

Mechanically: the telemetry SDK accepts only a closed `MetricEvent` union whose types make an account or bird identifier **unrepresentable** — there is no field of that shape to populate. The telemetry plane has no network route and no credential to `sim-db`; the analytics warehouse has no ingestion path from `sim.*`; a CI dependency check fails the build if the analytics service acquires a transitive import of the sim schema package. This reads as a privacy claim in the PRD and behaves as an architectural rule here.

**Validation without measurement.** We still need to know the drift function is healthy in production, and we cannot look at users' birds. The answer is **shadow accounts** (§13.3): a small fleet of system-owned accounts, driven by scripted behavior profiles against production, whose birds we may measure freely because they are ours. Real users' aviaries are never inspected for product analytics.

---

## 11. Rollout

### 11.1 Milestones

| # | Milestone | Content | Exit criteria |
|---|---|---|---|
| **M0** | Foundations (wk 1–2) | Repo, CI, IaC, Postgres, edge shell, `@aviary/sim` skeleton, determinism lint | Deterministic-kernel property test green in CI |
| **M1** | Engine core (wk 3–6) | Traits, drift, mood FSM, tick worker, leases, event log, fast-forward | Fast-forward equivalence test green; **calibration lab** (§12.3) reproducing 1-wk/3-wk targets |
| **M2** | Voice (wk 4–8) | Motif libraries for 6 species, identity params, call runtime, chorus, listen-in mix | Recognizability study passes at 2 and 4 birds (§12.4); zero recorded audio in bundle |
| **M3** | Scene (wk 5–10) | Canvas renderer, rig, idle synthesis, day/night, weather, perches, top bar, warm seed | First-bird p75 < 500 ms on reference profile; 60 fps soak green; no-spinner invariant green |
| **M4** | Accounts & sync (wk 7–11) | Magic link, sessions, snapshot/event APIs, multi-device, export, delete | Multi-device presence-union test green; monotonicity trigger verified; no-LWW audit |
| **M5** | Surfaces (wk 9–13) | Greeting, offers, settle, notebook, narration, captions, keyboard, reduced motion | Full a11y CI suite green; notebook corpus review passes; greeting-variation test green |
| **M6** | Visits & hardening (wk 12–15) | Invites, visitor view, revocation, visit log, rate limits, security review, soak | Visitor-write-path proof; pen-test findings closed; 30-min memory soak green |
| **M7** | Closed beta (wk 15–19) | ~150 invited users, 4 weeks minimum | ≥ 3 weeks of real drift observed on shadow accounts; qualitative "does it feel alive" review |
| **M8** | Public v1 (wk 20) | Staged ramp 5% → 25% → 100% over 10 days | All alarms quiet; launch gate (§11.5) signed |

Five workstreams run in parallel: engine/backend, client/scene, audio, accessibility, infra/edge. Accessibility is a workstream with its own engineer from M0, not a review step at M5 — `accessibility_perf.md` is explicit that a reduced-motion mode arriving as a "v1.1 fix" is a v1 launch that told those users the product wasn't for them.

### 11.2 The beta must be long enough to observe drift

A two-week beta cannot validate a three-week calibration target. The closed beta runs a **minimum of four weeks** with participants recruited for genuine intermittent use, and the primary qualitative question at the end is not "did you like it" but "did anything about the birds seem different from when you started" — asked without naming personality, drift, or traits, so the answer measures the thing the product actually promises.

### 11.3 Ramping birds per aviary

V1 launches every account at 2 birds. The cap is 7 and new birds are offered on **aviary age** — not visits, not interactions, not a paid tier (`bird_engine.md`).

| Bird | Aviary age |
|---|---|
| 3rd | 90 days |
| 4th | 180 days |
| 5th | 300 days |
| 6th | 450 days |
| 7th | 640 days |

No account reaches even the third-bird gate for 90 days after launch, which is deliberate slack: it gives us a full quarter to complete the **call-recognizability study at 3, 5, and 7 concurrent birds** before any real user hears a 3-bird chorus. The offer ships behind a config flag that stays off until that study passes. If recognizability degrades at 5, we lower the cap for v1 and say so — the cap exists to protect the per-bird relationship, and shipping 7 muddy birds to defend a number in a doc would defeat its own purpose.

The new-bird offer itself is a quiet surface, in naturalist voice, reached in the aviary — never a notification, never a badge, never a celebration.

### 11.4 What we instrument from day one

Everything in §10.5, live before the first beta user. Plus, from day one and specifically because they are the things that fail silently:

- **Tick lag** per shard (the "is the aviary actually running" metric).
- **Presence-union sanity**: on shadow accounts, assert union-time ≤ wall-clock time in every window. A regression here doubles drift for multi-device users and would otherwise be invisible.
- **Trait-write provenance**: an alarm on any `sim.bird_traits` write not attributable to a tick transaction.
- **Monotonicity violations**: the DB trigger's rejection count. Expected to be exactly zero forever; non-zero means a code path exists that tries to take something away from a user's bird.
- **Product-invariant CI results** tracked as a release-health signal, not just a build gate.

### 11.5 Launch gate

Sign-off requires: all budget gates green on reference hardware; a11y suite green plus a manual pass with VoiceOver/Safari, NVDA/Firefox, and JAWS/Chrome; the calibration lab reproducing both drift targets on the shipping constants; zero recorded audio in the bundle; the visitor-write-path proof (no code path from a visitor request to the event log or the trait table); the invariants suite green; a privacy review confirming no per-account dimension exists in any shipped metric; and a security review of magic-link, invite-token, and export-link flows.

---

## 12. Testing strategy

### 12.1 Kernel property tests (merge-blocking)

- **Determinism**: same `(state, seed, time range)` → byte-identical output, 10⁴ random cases.
- **Fast-forward equivalence**: `advance(s, t, t+n) === fold(step, s, n)`, 10⁴ states × 10³ seeds.
- **Monotonicity**: no input sequence — including adversarial ones (long absence, clock skew, duplicated events, out-of-order delivery, negative-duration windows) — ever decreases a trait.
- **Presence union**: overlapping windows from N devices credit at most wall-clock time.
- **Idempotency**: replaying any event batch changes nothing.
- **Bounds**: traits stay in `[0,1]`; per-day `Δt` respects its ceiling under maximal input.

### 12.2 Sync and integration tests

Two simulated clients against one account, scripted through: simultaneous presence, offset presence, offline queueing then replay, a tick crash mid-transaction, an out-of-order snapshot, a revoked session mid-session, and a greeting intent raced by two devices. Assertions: no double-credited presence, no lost drift, no duplicated greeting, `tick_seq` monotone at both clients.

### 12.3 The calibration lab

The most important non-shipping artifact in the project. An accelerated-time harness that runs `@aviary/sim` over synthetic behavior profiles — *regular visitor*, *heavy user*, *weekend-only*, *lapsed at week 2*, *bursty*, *multi-device* — across 12 simulated weeks in seconds, asserting every target in §5.2, emitting drift curves as build artifacts, and diffing them against the previous release. Any constant change that moves a curve outside tolerance fails the build with the before/after plot attached. Drift calibration is the highest-consequence tuning decision in the product; it gets a purpose-built instrument rather than judgment.

### 12.4 Audio tests

- **Recognizability (automated)**: render 200 calls per bird across the full mood × drift cross-product; extract MFCC + F0-contour fingerprints; assert within-bird cluster separation from between-bird clusters exceeds a fixed margin. This is the machine proxy for "you know Pip by ear."
- **Recognizability (human)**: a listening study at 2, 4, 5, and 7 birds; participants identify which bird called after a short familiarization. Gates the bird-count ramp (§11.3).
- **Variation**: 1,000 calls from one bird under identical state produce > 950 distinct parameter tuples. No rotation-of-N-variants passes.
- **No recorded audio**: bundle scan for audio files and `decodeAudioData`.
- **Mix**: listen-in ramp shape verified via `OfflineAudioContext`; unfocused buses never reach silence.

### 12.5 Accessibility tests

axe-core on every chrome surface; keyboard-path E2E covering the entire spec of §9.4; narration-quality assertions (cadence within 30–60 s at idle; no repeated template within 12; forbidden-lexicon lint; no `assertive` regions); caption-correctness tests asserting the caption matches the synthesized call's parameters for 500 seeded calls; reduced-motion screenshot-diff suite; contrast checks across every sky palette × fade state × motion mode; and a manual screen-reader pass per release on the three reader/browser pairs.

### 12.6 Product-invariants suite

Executable non-goals. Fails CI on:

| Check | Rejects |
|---|---|
| Banned lexicon in user-facing strings | streak, achievement, unlocked, level, badge, XP, score, rank, "welcome back", "days in a row", "you visited", "birds adopted" |
| Toast/banner/snackbar primitives | Any such component, route, or hook existing at all |
| Loading-indicator primitives | `spinner`, `loader`, progress bars on the aviary path |
| Snapshot schema | Any field named for a trait, or any raw trait value in `VisitorSnapshot`/`Snapshot` |
| Telemetry schema | Any account/bird identifier field in any `MetricEvent` |
| Trait exposure | Any client-bundle import of `sim/drift`; any API route matching `personality\|trait\|boldness\|vector` |
| Visitor write paths | Any route reachable from a visit token that reaches `sim.interaction_events` or `sim.bird_traits` |
| `aria-live="assertive"` | Any usage |
| Notebook output lint | User-referential phrasing, digits about the user, second person |

The PRD says the temptation to add "just one" is strong and predictable, and that the only durable defense is refusing the first one. A refusal that lives in a document gets re-argued; a refusal that fails the build gets a conversation with this plan attached.

### 12.7 Performance tests

`size-limit` on every bundle; first-bird timing on a throttled device profile (4× CPU, 4G) per PR; nightly 30-minute soak on reference laptop hardware asserting frame-time p95 and memory slope; snapshot payload contract test; a load test of the tick fleet at 10× projected accounts to validate the tiering model before it is needed.

---

## 13. Risks

Ordered by expected cost × probability. Each carries a detection mechanism, because a risk you cannot detect is not managed.

**R1 — Drift calibration is wrong (highest consequence).**
Too fast and the product becomes a Tamagotchi where clicking moves a number; too slow and it becomes a screensaver. The band is narrow and the feedback loop is three weeks long.
*Mitigations*: the calibration lab (§12.3) collapses the feedback loop to seconds; versioned `CalibrationSet` recorded per bird; shadow accounts in production; conservative launch constants (slightly slow is recoverable, too fast is not).
*The correction path is asymmetric, deliberately.* If we ship η too fast, we do **not** roll traits back — that would violate monotonicity and would be perceptible as a bird regressing, which is the exact experience the engine exists to prevent. We slow η going forward and let saturation absorb it. If we ship η too slow, we apply a one-time **additive forward** correction through the normal delta path, computed offline from the retained event log, never as a recompute-and-overwrite. This preserves both `bird_engine.md`'s "never recomputed from event logs at runtime" and the monotonicity invariant.
*Detection*: shadow-account drift curves versus lab predictions, weekly.

**R2 — Silent sync corruption (double-counted presence, lost drift).**
Fails invisibly; the user just gets a bird that drifts wrong, and no log line says so.
*Mitigations*: presence union not sum; additive server-authored deltas only; exactly-once watermark; DB-level write restriction to `sim_tick_role`; monotonicity trigger; multi-client integration suite; production alarms on trait-write provenance and monotonicity rejections.
*Detection*: presence-union sanity assertion on shadow accounts; monotonicity rejection count expected to be exactly zero.

**R3 — Audio uncanniness.**
Procedural synthesis can land in a valley where calls sound synthetic rather than avian, or the chorus turns to mud. This risk is qualitative and cannot be unit-tested away.
*Mitigations*: a sound designer embedded from M2, not consulted at M6; per-species voicing reviews; the identity/expression split (§5.4) protecting recognizability; the density limiter and anti-unison rule; MFCC clustering as an automated proxy plus real listening studies as the actual gate; a contingency to reduce v1 to four species at higher voicing quality rather than ship six mediocre ones.
*Detection*: listening studies at M2, M5, and beta.

**R4 — Accessibility regressions after launch.**
The narration and caption surfaces are the easiest things in the product to let rot, because most contributors never experience them.
*Mitigations*: single-source state (§9.1) makes visual/narration divergence structurally difficult; a11y checks are merge-blocking, not advisory; reduced-motion has screenshot-diff coverage; a manual screen-reader pass is a release gate; the accessibility engineer owns these surfaces continuously rather than handing them off.
*Detection*: CI plus per-release manual pass.

**R5 — The autoplay policy undermines "already in motion."**
If the first session is silent for many users, the audio spine — which the PRD calls the affective spine — is missing exactly when first impressions form.
*Mitigations*: join-mid-phrase on unlock (§8.5); captions on while locked; no banner under any circumstance; measure unlock latency in aggregate RUM.
*Contingency*: if unlock latency is bad at scale, invest in a quieter path (e.g. a first-frame interaction affordance that is diegetic rather than chrome) — but never a "click to enable sound" prompt.

**R6 — The 500 ms budget versus per-account HTML.**
Per-account documents cannot be CDN-cached, so the first byte costs real edge compute.
*Mitigations*: warm seed from IndexedDB (§7.4) makes repeat visits — the overwhelming majority of sessions in a product people return to daily — nearly instant; edge compute co-located with a read replica; snapshot materialized on tick so the edge does a key-value read, not a join; the inline bootstrap is ≤ 20 KB.
*Contingency*: if cold-start p75 misses on the reference profile, serve a cacheable shell and race an inline-snapshot fetch, accepting a slightly later first bird on cold start only. The quiet field, never a spinner, covers that gap.

**R7 — Tick cost at scale.**
Naive per-minute ticking for every account is a large bill dominated by dormant aviaries.
*Mitigations*: the tiering model in §5.1, which is safe precisely because fast-forward equivalence is a proven kernel property rather than an approximation; load test at 10× projected accounts at M6.
*Detection*: tick lag p99 per shard; cost per active aviary tracked weekly.

**R8 — Notebook prose degrades to event-log-ese.**
`interactions.md` is explicit that one "session started at 7:43" entry tells the user the rest of the product's voice is performance.
*Mitigations*: template lattice with anti-repetition; the sparsity gate; the forbidden-lexicon lint; a 500-entry corpus review as a release gate; writer involvement in the template lattice, not just engineering.

**R9 — Gamification creep after launch.**
The most likely long-run failure, and it arrives as a reasonable-sounding pitch, not as a bad-faith one.
*Mitigations*: the invariants suite (§12.6) plus a PR template question — "does this surface tell the user something about themselves rather than about the aviary?" — plus this plan's §1.2 as the referenced authority.

**R10 — Privacy-boundary erosion.**
A future "just one dashboard" request that needs a per-account dimension.
*Mitigations*: the boundary is architectural (unrepresentable types, no network path, no credential, CI dependency check), not procedural; shadow accounts exist specifically so that the legitimate need behind such a request has a compliant answer ready.

**R11 — Visit-flow abuse.**
Invite emails as a spam or enumeration vector.
*Mitigations*: per-host and per-invitee rate limits; max 5 outstanding invites; constant-time, non-disclosing responses; 256-bit hashed single-scope tokens; 30-day expiry; immediate revocation at next pull.

---

## 14. Decisions and assumptions

Every call made where the PRD is deliberately unspecified. Each is defensible, cheap to revise, and recorded so a reviewer can overrule it with full context.

| # | Question | Decision | Rationale |
|---|---|---|---|
| 1 | Presence activity window | **4 minutes**; sampled every 15 s; single window capped at 120 s | `concepts.md` says "a few minutes… leaning toward the longer side because watching birds without moving is the actual product" |
| 2 | Tick cadence | 60 s logical; hot/warm/cold physical tiering under fast-forward equivalence | Preserves "runs whether or not a client is connected" at sane cost |
| 3 | Mood set | 6: wary, content, curious, drowsy, alert, **roosting** | `bird_engine.md` leaves the set open; `aviary_layout.md` requires a distinct full-night state |
| 4 | Trait range and seeds | `[0,1]`, seeded `U(0.25, 0.45)` ± species bias | Leaves headroom for weeks of monotonic drift without pinning |
| 5 | Drift constants | `α = 3.5e-4`/step (τ ≈ 2 d), `η = 0.0016`/step, `Δt ≤ 0.006`/trait/day | Derived from the 1-week/3-week targets; owned by the calibration lab, expected to move before launch |
| 6 | Day/night source | Device timezone offset + generic civil-twilight curve; **no geolocation, no IP geo** | `aviary_layout.md` needs local time, not location; asking for location for a bird app is a poor trade |
| 7 | Offer cooldown | 4 min per bird per offer type; 90 s global | `interactions.md` says "a few minutes"; per-type keeps all three offers reachable in a session |
| 8 | Notebook sparsity | ≥ 40 h between entries (16 h floor for top-2% salience) | Targets the "one entry every few days" cadence and protects sparsity for active users |
| 9 | Bird-count ramp | 90 / 180 / 300 / 450 / 640 days, gated on the recognizability study | `bird_engine.md` gives "a few months" for the third and "five or six" by a year |
| 10 | Species pool | 6, including one nightjar-like species active at night | `bird_engine.md` "about six"; `aviary_layout.md` requires the night-active signature |
| 11 | Transport | HTTP polling (25 s keepalive) + `ETag`/`304`, not WebSockets | 60 s tick, KB payloads, no co-presence; far cheaper to operate |
| 12 | Renderer | Canvas2D, no WebGL | Object count is small; the PRD explicitly does not want a parallax-heavy showpiece |
| 13 | Server language | TypeScript/Node for API **and** tick | One simulation kernel shared by tick, calibration lab, and client extrapolation |
| 14 | Audio unlock | Suspended context; resume on first gesture; join mid-phrase; captions until unlocked; **never a prompt** | Only resolution consistent with both browser policy and "notice, never announce" |
| 15 | Visit duration granularity | 5-minute buckets | `social_optional.md` says "approximate"; buckets resist inference about the visitor's habits |
| 16 | Export delivery | Signed, single-use, 24 h link to the verified address | `accounts_sync.md`; keeps trait values off any UI surface |
| 17 | Event-log retention | 90 days | Enough for a calibration audit; traits are canonical so nothing depends on the log surviving |
| 18 | Top-bar fade floor | Opacity floor preserving ≥ 3:1 glyph contrast, plus an `always_show_controls` setting | Honors "fades nearly to transparent" without shipping controls that fail non-text contrast |
| 19 | Snapshot content | Derived, partly quantized render parameters only — never traits | Makes "never exposed numerically" true against DevTools, not just against the UI |
| 20 | Warm-seed rendering | Cached snapshot fast-forwarded client-side for the first frame, render-only, never written back | The only way to hit < 500 ms on repeat visits without a load state; does not violate server-authority |

---

## 15. Open questions for the product owner

None are blocking; each has a working default already implemented in this plan.

1. **Third-bird offer presentation.** The plan treats it as a quiet in-aviary naturalist surface. If the intended register is even quieter — the bird simply arrives one morning, with the notebook noticing it — that is a small change and arguably more in keeping with "notice, never announce."
2. **Settle persistence across devices.** Currently a settled aviary is settled on every device (canonical state). The alternative — settle as a per-device gesture — is defensible but would make the aviary two aviaries, so canonical is the default.
3. **Bird naming at adoption.** The plan offers default suggestions with inline renaming during the first-bird fly-in. If the first encounter should carry no text input at all, naming could defer to the first return visit.
4. **Beta length.** Four weeks is the floor for observing three-week drift. If schedule pressure compresses it, the calibration lab and shadow accounts carry the validation load, and we should say plainly that the three-week claim was validated in simulation rather than in the wild.
