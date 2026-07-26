# Pocket Aviary — v1 Implementation Plan

**Status:** implementation plan, phase 1. No product code is written by this document.
**Audience:** the engineering team that will build v1 end to end.
**Source of truth:** the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`). Where this plan and the PRD disagree, the PRD wins. Where the PRD is silent, this plan makes a call and marks it **[CALL]** with reasoning; every such call is collected in §17.

---

## 0. How to read this plan

Pocket Aviary is unusual in that its correctness conditions are mostly *affective*. A build that passes every functional test and still ships a "Welcome back!" toast has failed. So this plan is organized around two things at once: the systems to build, and the mechanisms that keep the systems from drifting into the wrong product.

Section 1 is the invariant register — the small set of properties that must hold, each with the enforcement mechanism that makes violating it hard rather than merely discouraged. The design sections (§3–§13) reference those invariants by ID. If you read nothing else, read §1 and §17.

The organizing engineering conviction of this plan: **the PRD's hard rules should be made structurally unrepresentable wherever possible, not merely documented.** A rule enforced by a code review checklist survives about six months. A rule enforced by a type that has no method to violate it survives the product. Several sections below spend design budget buying that property — a personality accumulator that has no decrement operation, a notebook generator with no access to a user-fact type, a narration generator that never receives a mood enum, a lint boundary that makes system-voice strings unimportable from product surfaces. That expenditure is deliberate and it is the plan's main structural idea.

---

## 1. Invariant register

These are the properties that must be true of the shipped product. Each has an ID, an owner surface, and a mechanism. The "charm suite" (§15.6) is the CI job that encodes them.

| ID | Invariant | Mechanism |
|---|---|---|
| **INV-1** | Presence requires the conjunction of three signals: `document.visibilityState === 'visible'` **and** window focus **and** a `pointermove`/`keydown` within the activity window. Never any subset. | Single `PresenceGate` module; the three predicates are a typed 3-tuple with one `and` reduction; truth-table unit test covering all 8 combinations; server re-validates the asserted conjunction on every ping. |
| **INV-2** | Personality traits are monotonic non-decreasing. No code path decreases a trait. | Accumulator type exposes only `add(NonNegative)`. No setter, no subtract, no absolute write. Property test over 10⁵ random event sequences asserts non-decrease of every trait. |
| **INV-3** | Personality vector values are never exposed to any user surface, in any form, at any tier — no stats panel, no debug view, no ARIA attribute, no API field, no export field beyond the account-export blob the user explicitly requests. | Vectors live in a `Private<T>` wrapper stripped by the snapshot serializer; serializer has no branch that can emit it; contract test asserts the trait keys never appear in any client-bound response body across the full endpoint matrix. |
| **INV-4** | Only the server simulation tick writes personality state. Clients write events only. | DB grant: the API service role has `INSERT` on `interaction_event` and `SELECT` on `bird`, no `UPDATE` on personality columns. Enforced in migrations, asserted by a test that attempts the write and expects a permission error. |
| **INV-5** | No last-write-wins on personality. All updates are additive server-authored deltas applied in event-log order. | Tick applies `A += δ` inside a transaction keyed by `(aviary_id, tick_seq)`; no code path sets an absolute value; audit table records every δ. |
| **INV-6** | Calls are procedurally synthesized. No recorded audio ships, in any path, including fallbacks. | Build gate: fail if any `.mp3/.wav/.ogg/.m4a/.flac/.opus` asset exists in the bundle or in the CDN manifest. WebAudio-unavailable fallback is silence + captions (§9.6). |
| **INV-7** | No announcement surfaces. No welcome toast/banner/modal, no arrival copy, no "you've been gone X days," no achievement/celebration surface. | Banned-lexicon CI check over all user-facing string bundles; no toast/snackbar primitive exists in the component library at all (§11.2). |
| **INV-8** | No gamification: no streak, score, level, badge, XP, rank, visit count, calendar, "birds adopted: N," or any surface exposing visit frequency. | Banned lexicon; the notebook generator cannot express user-behavior facts (INV-9); no visit-frequency value is computed anywhere in the product code — the query does not exist. |
| **INV-9** | The notebook observes the aviary, never the user. | `NotebookGenerator` accepts `AviaryFact[]` only. There is no `UserFact` type in the module's dependency graph; the import boundary is lint-enforced. |
| **INV-10** | Voice split: naturalist for product surfaces, matter-of-fact for system surfaces (identity, errors, sync, settings, accessibility settings, anything financial). | Two string namespaces; ESLint import boundary permits `system/*` strings only under `src/surfaces/system/**` and `naturalist/*` only outside it. Build fails on violation. |
| **INV-11** | No spinner, skeleton, progress bar, or fade-from-static on the aviary. The load state and the empty-aviary state are the same quiet field. | No spinner/skeleton primitive in the component library. Visual regression suite covers cold load, slow-network load, resume-from-suspend, and empty-aviary. |
| **INV-12** | Bird identity is stable forever. A bird is never reset, regenerated, swapped, or reseeded. | `bird.id` is immutable; personality columns have no reset path; migrations touching `bird` require a documented identity-preservation argument in the migration header, checked by a migration lint rule. |
| **INV-13** | Visitor sessions never contribute presence, interaction events, or drift to the host. | Visitor tokens carry a `read:aviary` scope only; the event-ingest endpoint rejects any token without `write:events`. Visit duration is recorded in a separate `visit` table with no path into the simulation. |
| **INV-14** | Per-bird / per-account interaction state never enters telemetry, analytics, or any aggregate. | Metric definitions are declared in one registry with a typed allowlist of dimensions; simulation DB has no network route or IAM grant to the warehouse; a Terraform CI check fails on any grant that would create one. |
| **INV-15** | Reduced-motion is a designed presenter, not disabled animation. Drift, mood, calls, notebook are identical. | Reduced-motion is a `Presenter` implementation over the same scene state, not a set of `if (reducedMotion) return` guards. Parity test asserts identical simulation output under both presenters. |
| **INV-16** | Screen-reader narration is naturalist prose, never a state list, and never names a mood or a number. | Narration generator's input type is `Posture`/`Behavior` descriptors; the `Mood` enum and trait values are not in its type signature. |
| **INV-17** | Perch position is a signal the user reads, not a layout the user controls. | No API accepts a perch assignment. No drag handler exists on birds. |
| **INV-18** | The product does not push, ping, or email the user about the aviary. The one exception is the opt-in, off-by-default visit notification (§8.6). | No push infrastructure ships — no service worker push handler, no `Notification.requestPermission()` call anywhere. Outbound email is restricted to a hardcoded transactional-template enum (§8.7). |
| **INV-19** | Email is stored once, encrypted, on the account record. Every other reference to an account anywhere in the system is the synthetic UUID. | Lint rule bans `email` as a field name outside `identity/`; log redaction filter; a CI scan of log samples and message schemas for email-shaped strings. |
| **INV-20** | New birds are gated on aviary age only — never on visits, interactions, drift, or payment. | The offer scheduler function's signature takes `(aviaryCreatedAt, now, currentBirdCount)` and nothing else; no engagement data is in scope. |

---

## 2. Scope

### 2.1 In scope for v1

**Account and identity.** Magic-link email sign-in (15-minute expiry, single-use). Per-device revocable sessions. Email change with new-address verification. Account export (JSON, emailed download link). Soft delete with a 30-day recovery window, then hard delete.

**The aviary.** One canonical aviary per account. Two starter birds at adoption, engine cap of seven. Six-species pool. Three perch zones. Local-time day/night cycle. Rare ambient weather. Continuous ambient micro-motion. No chrome inside the scene; a thin, fading top bar above it.

**The bird engine.** Five-trait hidden personality vector per bird, persisted server-side. Monotonic-toward-expressive drift on a slow low-pass filter. Six-state mood FSM on a fast timescale, persisted across sessions. Procedural per-bird call grammar with stable identity across mood and drift. Bird-to-bird interaction and emergent chorus. Mood-shaped idle motion and perch choice.

**Interactions.** Return-greeting varied by absence length, boldness, and mood. Listen-in with slow bidirectional mix ramps. Offers (seed, song fragment, still pool) with a silent per-bird cooldown. Settle with a five-second undo. Presence accounting per the three-signal conjunction. Field notebook: auto-generated, sparse, read-only, infinite scrollback.

**Sync.** Server-side simulation tick at ~60s cadence, running whether or not a client is connected. Snapshot-pull clients that interpolate. Append-only client event log. Additive server-authored personality deltas.

**Social.** Per-invite, email-addressed, revocable visit invitations. Read-only ambient visitor view. 30-day invite expiry. Visit log in account settings. Visit notifications opt-in, off by default.

**Accessibility.** Naturalist screen-reader narration on a slow cadence. Reduced-motion mode as a designed cross-fade presenter. Runtime-generated call captions. Full keyboard navigation with scene-legible focus indicators. WCAG AA contrast on all user copy.

**Performance.** ≤2MB gzipped initial JS. <500ms time-to-first-bird on mid-tier mobile over 4G. 60fps idle on a five-year-old mid-range laptop, sustained over 30 minutes. Zero memory growth over a 30-minute session, tested in CI.

**Observability.** Aggregate-only RUM and synthetic checks. Simulation-tick latency alarm at p99 > 5s.

### 2.2 Explicitly out of scope

Native iOS/Android apps, and no data-model or protocol accommodation for them. Payments and tiers. Shared, team, or household aviaries. Multiple aviaries per account. Customizable or purchasable scenes. Public discovery, profiles, follows, feeds, comments, ratings, featuring. Leaderboards, and the aggregate metrics that would underlie them. Achievements, streaks, levels, scores, badges, XP, counters of any kind. Push notifications and any re-engagement email. Tamagotchi mechanics: hunger, death, distress, decaying happiness, negative drift on neglect. Co-presence during visits. Editable notebooks. User-controlled perch placement. A numeric or debug view of personality. Recorded-audio fallback. Compatibility paths for browsers older than the last two major versions of Chrome, Safari, Firefox, Edge.

### 2.3 Scope risks we accept

We accept that v1 ships with a small feature surface and a large hidden system. The bird engine, the sync architecture, the audio pipeline, and the accessibility surfaces each carry more engineering weight than their UI footprint suggests. Estimating this project from its screen count will produce a wrong number by a factor of three.

We accept that the drift calibration cannot be fully validated before a multi-week beta (§14.3), and that this puts a real floor under the schedule that cannot be compressed by adding people.

---

## 3. Architecture

### 3.1 Shape

Five services plus a static edge tier. All server code is TypeScript on Node 22; the client is TypeScript with no UI framework in the aviary render path.

The language choice is not neutral: the call grammar, the caption writer, the narration writer, and the naturalist phrase engine must produce byte-identical output on client and server (captions must match the audio that played; narration must match the scene). Sharing one compiled package across both is worth more than any per-service language optimization.

```
                         ┌────────────────────────────────┐
   browser ──────────────│  edge (CDN + edge worker)      │
     │                   │  · HTML shell w/ inlined       │
     │                   │    snapshot for signed-in users│
     │                   │  · static asset serving        │
     │                   └───────────┬────────────────────┘
     │                               │ KV read (snapshot cache)
     │  HTTPS/JSON                   ▼
     ├──────────────► ┌──────────────────────────┐
     │                │  api        (stateless)  │──┐
     │                │  · snapshot read         │  │
     │                │  · event ingest          │  │
     │                │  · notebook, settings    │  │
     │                └──────────────────────────┘  │
     │                ┌──────────────────────────┐  │   ┌─────────────────┐
     ├──────────────► │  identity   (stateless)  │──┼──►│  Postgres       │
     │                │  · magic link, sessions  │  │   │  (simulation DB)│
     │                │  · email change/export   │  │   │  · accounts     │
     │                │  · deletion lifecycle    │  │   │  · birds/vectors│
     │                └──────────────────────────┘  │   │  · event log    │
     │                ┌──────────────────────────┐  │   │  · notebook     │
     └──────────────► │  visits     (stateless)  │──┤   │  · invites/visits│
                      │  · invites, visitor auth │  │   └────────┬────────┘
                      └──────────────────────────┘  │            │
                      ┌──────────────────────────┐  │            │
                      │  sim        (workers)    │──┘            │
                      │  · the tick              │◄──────────────┘
                      │  · drift, mood, calls    │
                      │  · notebook generation   │──► KV snapshot cache
                      │  · weather scheduling    │
                      └──────────────────────────┘
                      ┌──────────────────────────┐
                      │  mailer     (worker)     │  transactional enum only
                      └──────────────────────────┘
```

**Isolation boundary.** The simulation Postgres cluster sits in its own VPC and its own cloud account. No analytics or warehouse credential exists that can read it. There is no CDC stream, no replica in the data platform, no nightly dump. This is INV-14 as infrastructure rather than policy, and it is checked in CI against the Terraform plan.

### 3.2 Client/server split — the canonical boundary

This is the most important line in the system. Everything below is stated as a rule the code enforces.

**Server-owned (canonical, in the snapshot):**
personality vectors (never serialized to a client — INV-3); mood and mood timers; perch-zone assignment per bird; the call schedule for the next ~2 minutes; the greeting plan on session start; weather state and schedule; `sim_time`; bird roster, names, species, adoption dates; aviary `created_at` and timezone; notebook entries.

**Client-owned (presentation only; never synced, never in the event log, never authoritative):**
exact pixel positions and interpolation state; idle micro-motion phase; leaf and feather particles; top-bar fade state; listen-in focus; the settled lighting state (§7.5 **[CALL]**); caption rendering; the choice of standard vs reduced-motion presenter; viewport layout.

**Shared-deterministic:**
audio synthesis. The server schedules a call as a seed tuple; the client synthesizes it; the shared grammar package derives the caption from the same tuple. The audio is rendered locally but is not invented locally.

The reason for putting the call *schedule* on the server and the call *synthesis* on the client: the schedule is behavior (it depends on vocal frequency, mood, and what the other birds are doing, and must agree across devices), while synthesis is rendering (it costs CPU, must not cost bandwidth, and can be perfectly reproduced from the seed). Splitting at that seam is what makes the chorus a real emergent property of the simulation rather than a client-side coincidence.

### 3.3 Render pipeline boundary

```
snapshot (server) ──► SceneState ──► BehaviorLayer (client, 30Hz fixed step)
                                          │   idle micro-motion, particle spawn,
                                          │   interpolation toward canonical targets
                                          ▼
                                     PresentationFrame
                                          │
                     ┌────────────────────┴────────────────────┐
                     ▼                                          ▼
            StandardPresenter                          ReducedMotionPresenter
            (rAF, continuous)                          (cross-fade, ~4Hz updates)
                     │                                          │
                     └────────────────────┬─────────────────────┘
                                          ▼
                                   Canvas2D compositor
```

`SceneState` and `BehaviorLayer` are shared by both presenters. INV-15 falls out of the structure: there is no reduced-motion branch inside the simulation or behavior code, because there is no place to put one.

---

## 4. Data model

Postgres 16. All timestamps `timestamptz`. All IDs UUIDv7 (time-ordered, index-friendly).

### 4.1 Identity

```sql
account (
  id                    uuid primary key,          -- synthetic; the only external reference (INV-19)
  email_ciphertext      bytea not null,            -- AES-256-GCM, envelope-encrypted via KMS
  email_lookup_hash     bytea not null unique,     -- HMAC-SHA256(normalized_email, pepper); sign-in lookup only
  email_verified_at     timestamptz,
  created_at            timestamptz not null,
  status                text not null,             -- active | pending_deletion | deleted
  deletion_requested_at timestamptz,
  settings              jsonb not null default '{}'
)
```

The blind index (`email_lookup_hash`) is what lets sign-in find an account without storing plaintext email anywhere but the one encrypted column. The pepper lives in KMS, not in the database, so a database compromise alone does not yield an email-enumeration oracle.

```sql
pending_email_change (account_id, new_email_ciphertext, new_email_lookup_hash, token_hash, expires_at)
magic_link  (token_hash primary key, account_id, email_lookup_hash, created_at, expires_at, consumed_at, request_ip_hash)
session     (id uuid pk, account_id, token_hash, device_label, created_at, last_seen_at, revoked_at)
```

Magic-link tokens are 32 random bytes; only the SHA-256 hash is stored. Consumption is a conditional update (`WHERE consumed_at IS NULL`) so replay is impossible even under concurrency. Rate limit: 5 requests per email per hour, 20 per IP per hour, both with exponential backoff; a rate-limited request still returns the same neutral response so the endpoint is not an account-existence oracle.

### 4.2 Aviary and birds

```sql
aviary (
  id             uuid primary key,
  account_id     uuid not null unique references account(id),
  created_at     timestamptz not null,   -- drives bird-offer schedule (INV-20)
  timezone       text not null,          -- IANA
  tz_candidate   text, tz_candidate_since timestamptz,   -- travel debounce (§6.7)
  sim_time       timestamptz not null,   -- how far the simulation has advanced
  tick_seq       bigint not null,
  seed           bigint not null,        -- deterministic weather/behavior stream
  weather        jsonb not null default '{"state":"clear"}',
  next_tick_due  timestamptz not null,
  tier           text not null default 'active'  -- active | dormant (scheduling only, §6.2)
)

bird (
  id             uuid primary key,       -- immutable forever (INV-12)
  aviary_id      uuid not null references aviary(id),
  species_id     text not null,
  name           text not null,
  adopted_at     timestamptz not null,
  seed           bigint not null,        -- call identity + behavior stream

  -- personality: seeds are fixed at adoption, accumulators only ever increase (INV-2)
  seed_bold      real not null, acc_bold      double precision not null default 0,
  seed_warm      real not null, acc_warm      double precision not null default 0,
  seed_vocal     real not null, acc_vocal     double precision not null default 0,
  seed_plumage   real not null, acc_plumage   double precision not null default 0,
  seed_curious   real not null, acc_curious   double precision not null default 0,
  lp_state       jsonb not null default '{}',  -- low-pass filter state per trait

  mood           text not null,
  mood_since     timestamptz not null,
  perch_zone     smallint not null,      -- 0 front, 1 middle, 2 back
  perch_slot     smallint not null,
  last_greeted_at timestamptz,
  offer_cooldown_until timestamptz
)
```

Traits are stored as `(seed, accumulator)` rather than as a current value. This is deliberate: the displayed trait is a pure function of the pair (§6.3), the accumulator has no decrement path, and the seed is write-once. A bug that "resets a bird" would have to overwrite a write-once column, which the migration lint rule and the audit table both catch.

```sql
personality_delta_audit (
  id bigserial pk, bird_id, tick_seq, applied_at,
  d_bold, d_warm, d_vocal, d_plumage, d_curious double precision,
  source_event_ids uuid[]
)
```

Append-only. Two purposes: recovering a vector if the row is ever corrupted (replay the audit), and a nightly assertion job that `sum(deltas) == accumulator` per bird. Given that losing a vector is the worst failure this product can suffer and would be invisible to monitoring, we pay for a redundant reconstruction path.

### 4.3 Events, notebook, social

```sql
interaction_event (
  id uuid pk, aviary_id, bird_id null, account_id,
  client_event_id uuid not null,        -- idempotency; unique (account_id, client_event_id)
  kind text not null,                   -- presence_ping | listen_in_start | listen_in_end
                                        -- | offer_made | offer_accepted | settle | session_start
  server_seq bigserial not null,        -- authoritative ordering
  server_received_at timestamptz not null,
  payload jsonb not null,
  consumed_at_tick bigint
) partition by range (server_received_at);   -- monthly partitions, dropped after 90 days

notebook_entry (
  id uuid pk, aviary_id, observed_at timestamptz not null,
  prose text not null,                  -- immutable once written
  template_id text not null,
  source_facts jsonb not null
)

invite (
  id uuid pk, host_account_id,
  visitor_email_ciphertext bytea, visitor_email_lookup_hash bytea,
  token_hash bytea not null unique,
  created_at, expires_at,               -- created_at + 30 days
  revoked_at, first_used_at
)

visit (
  id uuid pk, invite_id, started_at, last_seen_at, ended_at
)   -- no foreign key or code path into the simulation (INV-13)
```

Interaction events are retained 90 days. They are inputs to the tick, not a history the product exposes; the personality vector is the durable artifact and it is already canonical. Shortening retention shrinks the blast radius of the most sensitive table in the system.

### 4.4 Static data (in code, not the database)

Species definitions: silhouette geometry (SVG path data), default plumage palette, call-grammar motif library, identity signature (contour class, timbre band, rhythm class), diurnal profile. Six species (§6.6). Held in code because they are versioned with the renderer and the synthesizer and must be perfectly in sync with both.

---

## 5. API surface

JSON over HTTPS. Session tokens in `HttpOnly; Secure; SameSite=Lax` cookies. All error bodies carry a `surface_key` that maps to a matter-of-fact string (INV-10) — the server never sends prose the client renders verbatim, so copy stays in the client's string system where the lint boundary can see it.

### 5.1 Identity

```
POST   /v1/auth/request-link       { email }              → 202 (always, regardless of existence)
POST   /v1/auth/consume            { token }              → 200 sets session cookie | 410 expired/used
GET    /v1/auth/sessions                                  → [{ id, device_label, created_at, last_seen_at, current }]
DELETE /v1/auth/sessions/{id}                             → 204
POST   /v1/account/email           { new_email }          → 202 (verification sent to new address)
POST   /v1/account/email/verify    { token }              → 200
POST   /v1/account/export                                 → 202 (link emailed to verified address)
POST   /v1/account/delete                                 → 200 { hard_delete_at }
POST   /v1/account/delete/cancel                          → 200
GET    /v1/account/settings                               → { ... }
PATCH  /v1/account/settings        { ... }                → 200
```

### 5.2 Aviary state

```
GET /v1/aviary/snapshot?since={tick_seq}&session_start={bool}&absence_ms={int}&tz={iana}
```

Response:

```jsonc
{
  "tick_seq": 918224,
  "sim_time": "2026-07-26T14:31:00Z",
  "server_time": "2026-07-26T14:31:12Z",
  "aviary": {
    "timezone": "America/Chicago",
    "solar": { "dawn": "05:52", "dusk": "20:26" },   // from tz latitude band, not user location
    "weather": { "state": "light_rain", "started_at": "...", "ends_at": "..." },
    "age_days": 412
  },
  "birds": [{
    "id": "018f...", "name": "pip", "species": "warbler",
    "perch": { "zone": 0, "slot": 1 },
    "posture": "preening",                 // behavior descriptor, never the mood enum (INV-16)
    "plumage": { "palette_id": "warbler_a", "saturation_step": 6 },  // quantized; not the raw trait (INV-3)
    "calls": [                              // schedule for the next ~2 min
      { "at_ms": 3400,  "seed": "018f...:41822", "motif": "rise3", "params_hash": "9c2a" },
      { "at_ms": 27100, "seed": "018f...:41823", "motif": "trill_pair", "params_hash": "1de0" }
    ]
  }],
  "greeting": {                             // present only when session_start=true
    "greeter_bird_id": "018f...",
    "form": "step_forward_and_call",
    "delay_ms": 820,
    "call_seed": "018f...:g8891",
    "responder": { "bird_id": "018f...", "delay_ms": 1740, "call_seed": "..." }
  },
  "narration": "a small grey bird is on the front rail, calling softly. rain has just passed; the light is thin.",
  "next_pull_after_ms": 45000
}
```

Notes on the shape. `plumage.saturation_step` is a coarse quantization (8 steps) of the plumage trait — enough to render, too coarse to reconstruct a number, and it satisfies INV-3 while keeping the client able to draw the right colors. The `greeting` block is computed *in the request path*, not on the tick, because it must land within 1–2s of tab open and the tick is a minute wide (§6.5). `narration` is server-generated so notebook and narration share one voice engine and can be tuned without a client deploy.

```
POST /v1/aviary/events        { events: [{ client_event_id, kind, bird_id?, occurred_at, payload }] }
                              → 202 { accepted: [...], duplicate: [...] }
GET  /v1/notebook?before={cursor}&limit=30    → { entries: [...], next_cursor }
POST /v1/birds/{id}/name      { name }        → 200
POST /v1/birds/offer          { kind }        → 200   (kind: seed | song_fragment | still_pool; a small
                                                       motif id for song_fragment)
GET  /v1/aviary/bird-offer                    → { available: bool, species_preview?, offered_at? }
POST /v1/aviary/bird-offer    { accept: bool, name? }  → 200
```

Event submission is batched (the client flushes every ~15s and on `pagehide`/`visibilitychange`) and idempotent on `client_event_id`, so a `sendBeacon` that fires twice does not double-count presence. `POST /v1/birds/offer` returns immediately; reactions arrive via the next snapshot plus a client-side immediate reaction computed from the shared behavior package, so the offer feels instantaneous while the canonical outcome still comes from the server.

### 5.3 Visits

```
POST   /v1/invites               { visitor_email }   → 201 { id, expires_at }
GET    /v1/invites                                   → [{ id, visitor_email, created_at, expires_at, state }]
DELETE /v1/invites/{id}                              → 204   (immediate; INV-13)
GET    /v1/visits                                    → [{ visitor_email, started_at, approx_duration_min }]
POST   /v1/visit/consume         { token }           → 200 sets visitor cookie (scope: read:aviary:{id})
GET    /v1/visit/snapshot                            → same shape as /aviary/snapshot, minus
                                                       greeting, narration-of-host-events, notebook,
                                                       settings, visit log; 403 on revoke/expire
```

The visitor token's scope is a hard capability, not a role check sprinkled through handlers: `POST /v1/aviary/events` requires `write:events` and a visitor token simply does not carry it. There is no branch in the event ingest path that reasons about visitors, which is the point (INV-13).

Visitor clients poll `/v1/visit/snapshot` every 15s rather than the host's 45s. That tightens the worst-case revocation delay to 15 seconds; "immediately" in the PRD is honored to within one poll and we state the bound honestly rather than claiming instantaneity we cannot deliver over stateless HTTP. **[CALL]** We do not open a persistent connection solely for revocation; the operational cost of a WebSocket tier for a feature most accounts never use is not justified by 15 seconds.

### 5.4 What the API deliberately does not offer

No endpoint returns a trait value, a drift history, a visit count, a session count, a "days visited," or any aggregate over the user's own behavior. No endpoint accepts a perch assignment (INV-17), a mood override, or an absolute personality write (INV-4). There is no admin endpoint that displays a personality vector; support tooling can see that an account has N birds and that ticks are healthy, and nothing about who those birds are.

---

## 6. Simulation engine

### 6.1 The tick contract

> The tick is a pure function `tick(state, events, Δt, seed) → (state', facts)`. Scheduling may vary; the resulting state trajectory may not.

Everything in this section depends on that contract. It is what lets us batch, retry, replay, and accelerate the simulation without changing what the user experiences.

Cadence: 60 seconds of simulated time per step. Each step:

1. Open a transaction keyed on `(aviary_id, tick_seq)` — idempotent under retry.
2. Read unconsumed events ordered by `server_seq`.
3. Fold events into per-bird signal buckets for this step (presence seconds, listen-in seconds, offer outcomes).
4. Advance weather (§6.8).
5. Update the drift low-pass filters and accumulators (§6.3). **Only additive.**
6. Evaluate mood transitions (§6.4).
7. Re-evaluate perch choice.
8. Generate the call schedule for the next 2 minutes (§6.5).
9. Emit `AviaryFact`s; run the notebook scorer against them (§10.2).
10. Write `state'`, mark events consumed, append `personality_delta_audit`, publish the snapshot to the KV cache.

Steps 5–8 are pure. Steps 1–2, 10 are the only I/O. That separation is what makes the accelerated calibration harness (§14.3) possible: it runs steps 3–9 in a loop with no database at all.

### 6.2 Scheduling the tick at scale

Every aviary ticks whether or not a client is connected — that is the architectural claim the product rests on, and we do not weaken it. But executing one database transaction per account per minute for the whole population is a cost we should not pay for accounts nobody is watching.

The resolution is to separate *simulated cadence* from *execution cadence*:

- **Active aviaries** (a session in the last 30 minutes, or unconsumed events) tick every 60s of wall clock, one step at a time.
- **Dormant aviaries** tick in **batched catch-up**: a worker wakes every 30 minutes and executes all the intervening 60-second steps for that aviary in one transaction.

The batched execution must produce *exactly* the state that stepwise execution would have. That is achievable because a dormant aviary has no new events, the drift filter has a closed form over a constant (zero) drive, and the mood/weather streams are driven by a seeded PRNG that is stepped the same number of times either way. We enforce it with a property test: for random states and random step counts, `batched(state, n) === fold(step, state, n)`. If a future change breaks the equivalence, the test fails and the change is wrong.

When a client connects to a dormant aviary, we run the catch-up batch synchronously before serving the snapshot — the same computation, executed earlier. The user is never served a stale aviary, and nothing about the resulting state depends on their having arrived.

Sharding: aviaries are assigned to workers by `hash(aviary.id) % N` with a due-time index (`next_tick_due`). Estimated load at 100k accounts: ~5k active at peak (5k tx/min ≈ 83/s) plus dormant batches amortized. Comfortably one Postgres primary with read replicas for snapshot serving.

### 6.3 Drift

**Representation.** Each trait θ has a write-once seed `s_θ ∈ [0,1]` and an accumulator `A_θ ≥ 0`. The value is

```
V_θ  =  s_θ  +  (1 − s_θ) · A_θ / (A_θ + τ_θ)
```

Hyperbolic rather than exponential saturation, deliberately. Exponential saturation with a constant short enough to make three weeks visible pins a year-old bird at the ceiling; hyperbolic keeps a long tail so a bird watched for two years is still measurably more expressive than one watched for one year, without ever exceeding 1.0. Monotonicity is structural: `A` only increases and `V` is monotone in `A`.

**The filter.** Each step of length Δt = 60s:

```
D_θ  =  Σ_i  w[θ][i] · x_i          // raw drive from this step's signals
Ē_θ  ←  (1 − α) · Ē_θ  +  α · D_θ   // low-pass;  α = Δt / T,  T = 6 h
A_θ  ←  A_θ  +  k_θ · max(0, Ē_θ) · Δt
```

The `max(0, ·)` is redundant given non-negative signals and weights, and it stays anyway — it is the second lock on INV-2, at the one place a future signed signal could sneak in.

The six-hour filter constant is chosen so drift smooths across a session boundary and into the next day. Its effect is that a single long session does not immediately convert to trait movement; the drive decays over hours and integrates gently. This is the mechanism behind "no single session shifts a trait visibly," reinforced by explicit budgets below.

**Signal weights (starting values, calibrated in §14.3).** Per presence-second, applied to every bird in the aviary:

| trait | weight | rationale |
|---|---|---|
| plumage | 1.00 | the trait the PRD explicitly ties to sustained attention |
| warm | 0.80 | greeting-first behavior is what "being watched" should produce |
| bold | 0.60 | coming forward is the slowest, most legible change |
| vocal | 0.50 | |
| curious | 0.40 | mostly driven by offers instead |

Listen-in on bird X, per second, to X only: `warm ×2.5`, `vocal ×2.0` (relative to the presence weights above), plus the ordinary presence contribution X already receives. Offer accepted by X: an impulse equivalent to 90 presence-seconds of `curious`. Offer made at all: an impulse equivalent to 30 presence-seconds of `bold`, to every bird. Settle: no drift contribution in any direction — it ends the presence window and nudges mood, nothing more.

**Constant calibration.** Set `τ_θ = 1.0` and solve `k` from the stated target. "Regular visits" ≈ 5 sessions/week × 6 minutes of presence ≈ 30 min/week. With a typical seed `s = 0.35`:

- Three weeks ⇒ 5400 presence-seconds. Target ΔV = 0.10 ⇒ `A/(A+1) = 0.1538` ⇒ `A = 0.182` ⇒ **k ≈ 3.4 × 10⁻⁵ per presence-second** for the top-weighted trait.
- One week ⇒ 1800s ⇒ `A = 0.061` ⇒ ΔV ≈ **0.037** — comfortably above the instrument noise floor (0.02), and far below the perceptual band. This is exactly the instruments-vs-user gap the PRD asks for.
- One year of the same cadence ⇒ `A ≈ 2.7` ⇒ `V ≈ 0.82`. Two years ⇒ `V ≈ 0.90`. Never saturated.

**Budgets.** A 90-minute marathon session would, unbudgeted, move a trait by ~0.09 in one sitting — visible, and wrong. So:

- Per-session budget: ΔV ≤ **0.015** per trait.
- Per-day budget: ΔV ≤ **0.03** per trait.

Implemented as caps on accumulator growth, tracked in `lp_state`. Excess drive is discarded, not banked — banking would let a user "save up" attention, which is a resource mechanic and therefore the wrong product.

**Definitions used by the calibration tests.** *Measurable drift* = ΔV ≥ 0.02 on at least one trait, detectable by the harness. *Visible drift* = ΔV ≥ 0.10 on at least one trait **and** at least one behavior-threshold crossing: the bird's modal perch zone moves one step forward, or its greet-first rate crosses a band boundary, or its call density crosses a band boundary. The second conjunct matters: the user perceives behavior, not values, and a plan that defines "visible" purely numerically has not defined the thing the PRD is asking for.

**What drift never does.** No trait decreases. Neglect produces nothing at all — no decay, no negative signal, no "quiet penalty." The observable "quieter after two weeks away" is not a drift effect; it is a *mood and call-density* effect, and it recovers as soon as the user comes back. A bird that was ignored for a month has exactly the personality it had a month ago, and it will start drifting again from there. This distinction — quietness is fast-timescale, drift is slow-timescale and one-directional — is the mechanical implementation of the no-Tamagotchi rule, and it is the thing to protect in every future change to this module.

### 6.4 Mood

States: **wary, content, curious, drowsy, alert, roosting**. `roosting` finalizes the PRD's night state ("settled or sleeping"); the other five are as written. **[CALL]**

Transition evaluation runs each step. For each candidate mood *m*:

```
score(m) = θ_time(m, localHour)          // drowsy near dusk, alert early morning
         + θ_event(m, recentEvents)       // decaying over ~20 min
         + θ_weather(m, weather)          // rain → quieter; wind → alert or wary by personality
         + θ_personality(m, V)            // boldness suppresses wary; curiosity raises curious
         + θ_social(m, neighbourMoods)    // contagion, weighted by social warmth × perch adjacency
         + θ_inertia(m == current)        // hysteresis
```

Selection is a softmax over scores with temperature drawn from the bird's seeded stream, plus a **minimum dwell time of 4–9 minutes** (per-bird, seeded). The dwell floor is not a detail: idle motion is the visible surface of mood, and mood that flickers at tick resolution produces a bird that twitches between postures, which reads as a machine sampling a distribution.

**Daily reset.** The PRD says mood "resets on a daily-ish cadence" while also demanding mood never snap to a default on tab open. Both are honored by making the reset a *re-derivation at local dawn*, not an assignment: at dawn the inertia term is temporarily suppressed so mood re-forms from time-of-day, personality, and weather, with a weak carryover from yesterday. A bird that ended yesterday wary is likely to be content by mid-morning, but arrived there by transition, not by assignment. **[CALL]**

**Contagion.** An alarm-ish call or a wary transition raises `θ_social(wary)` for birds on adjacent perch zones, scaled by the receiving bird's social warmth. Contagion is damped (a fixed decay per hop and a per-step cap) so a single startle cannot cascade the whole aviary into wary — an aviary that all goes wary at once reads as a scripted event, which is the opposite of the intended effect.

**Never labeled.** No UI anywhere names a mood. Narration receives a `Posture` descriptor ("fluffed, low on the perch, eyes half-closed"), never the enum (INV-16). Captions describe the call, not the state. There is no mood icon, no tooltip, no color coding.

### 6.5 Call grammar and the call scheduler

**Schedule.** Each step plans calls for the next two minutes per bird. Base rate is a Poisson process with λ derived from `V_vocal`, modulated by mood (drowsy ×0.3, alert ×1.4, roosting ×0.05 except for the nocturnal species), by weather (rain ×0.5), by time of day, and by the listen-in state of *no one* — listen-in changes the mix, never the schedule (§9.3).

**Chorus** is emergent, not scripted. When a bird's call lands, other birds get a short *join window*; the probability of joining is `V_vocal × V_warm × mood_factor`. Two or three birds with high vocal frequency will regularly overlap. We do not add a "chorus event" type; if the model produces choruses at the wrong rate, we tune λ and the join probability, not add a special case.

**The call as a seed tuple.** A scheduled call is `(bird_seed, call_index, motif_id, mood, quantized V_vocal)`. From that tuple, the shared grammar package deterministically derives the full parameter set — note count, interval contour, per-note duration and glide shape, register offset, ornament density, timbre parameters — and, from the same parameters, the caption prose. This is the mechanism behind "each call's caption matches what was actually played," and it means a caption can never drift from its audio because they are two projections of one value.

**Identity vs variation.** Each species+bird has an **identity signature** held near-constant across every mood and every drift state:

- *contour class* — the shape of the phrase (rising, falling, arch, flat-with-terminal-drop, rattle)
- *timbre band* — spectral centroid range and roughness character
- *rhythm class* — the phrase's characteristic inter-onset pattern

Free to vary: register (±4 semitones by mood), tempo (±35%), phrase length and repeat count, amplitude envelope sharpness, ornament density, and per-instance jitter on every continuous parameter. The result is that Pip is always recognizably Pip while no two of Pip's calls are ever identical. This split is the single testable definition of the PRD's recognizability requirement, and it is what §15.5 tests against.

**Per-bird individuation within a species.** Two warblers in one aviary must be distinguishable. At adoption, a bird is assigned an individuation offset from its seed — a small fixed transposition, a rhythm-class variant, and a timbre-band offset — with a minimum-distance constraint against the birds already in that aviary. Two same-species birds are never issued adjacent individuation offsets.

### 6.6 Species pool

Six species. The pool is designed as a *set*: the constraint is that all six are mutually distinguishable, and that any two of them make a good starter pair.

| species | contour class | timbre band | rhythm class | diurnal |
|---|---|---|---|---|
| warbler-like | rising 3–5 note phrase | bright, low roughness | even, spaced | day |
| finch-like | short chip clusters | bright, medium roughness | burst-gap-burst | day |
| wren-like | dry rattle + rich terminal phrase | mid, high roughness | dense, accelerating | day |
| thrush-like | flute arch, long notes | dark, very low roughness | slow, wide gaps | day + dusk |
| tit-like | two-note pure whistle; buzzy scold | very bright, bimodal | paired | day |
| nightjar-like | low continuous churr | dark, high roughness | sustained | **night** |

The nightjar is the species that keeps night from being a dead state (per `aviary_layout.md`). It is excluded from starter pairs: a new user whose first day is mostly a bird that barely calls until dusk would read the aviary as broken. **[CALL]**

Starter selection: pick two day-active species with **different contour class and different timbre band**, so the first pair a user ever hears is maximally easy to tell apart. The user does not choose; the two arrive.

### 6.7 Time, timezone, and solar anchoring

Day/night follows the user's local time via the aviary's stored IANA timezone. Two problems the PRD leaves open:

**Travel.** A user flying to another continent should not have their aviary whipsawed. The client reports its timezone on session start; a differing value is stored as `tz_candidate` and adopted only after it has been reported consistently for 24 hours or across two sessions on separate days. A day trip across a boundary changes nothing; a move changes the aviary within a day. **[CALL]**

**Solar times without location.** Real sunrise/sunset needs latitude, which is location data we do not want to hold. Instead we derive dawn and dusk from a static table of representative latitude per IANA zone plus day-of-year. The result is seasonally correct within roughly 20 minutes and requires no user location at all. **[CALL]** This is a good trade: the product needs "evening feels like evening," not astronomical accuracy.

**Scene follows the aviary's timezone, not the device's.** If a traveling user's device says 14:00 but the aviary is still on the old zone at 22:00, the scene renders night and the birds roost — consistent, if briefly odd — rather than showing a bright noon scene full of sleeping birds. Scene and mood must never disagree; that disagreement is exactly the kind of small wrongness that reads as "this thing is fake."

### 6.8 Weather

A seeded Poisson process on the aviary's stream: light rain roughly 2–3× per week for 6–15 minutes; soft wind roughly daily for 3–8 minutes. Nothing else — no thunder, no snow, no storm, no seasonal weather feature. Weather is canonical (it must agree across devices and it feeds mood), scheduled by the tick, rendered by the client.

Effects are small and short: rain multiplies call rate by ~0.5 and biases posture toward fluffed; wind raises `alert` for bold birds and `wary` for timid ones. Both decay within ~20 minutes of the event ending. Weather must never be the most interesting thing on screen.

### 6.9 Bird offers over time

Purely a function of aviary age (INV-20). Thresholds: **75, 165, 270, 400, 560 days** for birds 3–7. A one-year-old aviary that accepted every offer has five birds; a thirteen-month-old aviary has six. That matches the PRD's "a few months → a third; a year-old aviary may have grown to five or six."

**Presentation, which is where this feature can most easily go wrong.** No modal. No celebration. No "You've unlocked a new bird!" On the first session after a threshold, an unfamiliar bird is simply present at the back perch, unnamed, keeping its distance. The notebook may notice it in its own voice ("a new bird at the back perch this morning. it hasn't come forward yet."). A quiet naming affordance appears in the top-bar account area. If the user names it, the bird joins the aviary and begins its own drift history. If the user does nothing for several days, the bird moves on, and the next threshold offers another. Declining is possible without ever opening a dialog.

---

## 7. Interactions

### 7.1 Return-greeting

Computed in the snapshot request path (not on the tick) so it can land within 1–2 seconds of tab open with no additional round trip.

**Absence bands** (`absence_ms` reported by the client, validated against server-side last-seen):

| band | window | greeting form |
|---|---|---|
| continuity | < 2 min | **no greeting** — the aviary simply continues |
| short | 2 min – 2 h | a glance up from what the bird was doing; sometimes a single quiet note |
| medium | 2 h – 2 days | head-tilt and a step toward the front perch; a two- or three-note call |
| long | > 2 days | re-orientation: a longer call, a slower approach forward, a second bird often responding |

The continuity band is load-bearing and easy to omit. A user who tabs away for forty seconds and comes back must not be greeted again — a greeting on every tab-focus is the fastest possible way to make the whole mechanism read as canned.

**Greeter selection.** Weight each bird by `V_bold × V_warm × moodAvailability(mood)`, times an anti-repetition factor that mildly disfavors yesterday's greeter. The anti-repetition term exists because the notebook wants to be able to write "pip greeted before wren today, first time this week," and that sentence is only true and only interesting if the greeter genuinely varies. Bolder birds greet more often; a wary bird may not greet at all today.

**Variation.** The greeting varies across five independent axes — greeter identity, form, call parameters (from the procedural grammar, so never repeated), timing offsets, and whether a second bird responds — which is combinatorial and continuous rather than a rotation among stored variants. A build that ships three pre-authored greeting animations in rotation has failed this feature even though it looks correct in a demo.

**Staggering.** When more than one bird greets, the second fires at a randomized 400–1200ms offset. Never in unison. A synchronized greeting announces the user's arrival to the aviary, which is precisely the wrong register.

**No textual welcome, anywhere** (INV-7). Not a toast, not a banner, not a modal, not a line of copy in the top bar, not a subtle "welcome" in the notebook. The bird is the welcome.

### 7.2 Listen-in

Engage by click, tap, or keyboard `Enter` on a focused bird. Disengage by clicking the same bird, focusing another, clicking empty scene, moving keyboard focus away, or `Escape`.

Mix targets on engage, over a **1.4s equal-power ramp**: focused bird `0 dB`, other birds `−12 dB`, ambient bed `−6 dB`. On disengage, the same ramp back. Never a hard cut, and other birds **never reach silence** — the aviary must stay a place where several things are happening, not a mixer with soloable tracks.

Listen-in changes the mix only. It does not raise the focused bird's call rate, does not bring it forward, does not make it perform. The effect of listening in shows up on the slow timescale instead: `warm` and `vocal` drift for that bird, so over weeks the bird you listen to becomes the bird that calls more and greets more. That is the right place for the reward, and it is worth defending in review when someone proposes making listen-in feel more "responsive."

**Decay.** Listen-in persists while engaged, and additionally decays back to ambient if presence has been absent for 10 minutes (a forgotten listen-in should not leave the aviary permanently unbalanced) and on settle. **[CALL]**

Listen-in focus is session-local presentation state. Listening in on the laptop does not change the mix on the phone. The listen-in *events* are canonical (they drive drift); the mix is not.

### 7.3 Offers

Reached from the top-bar affordance, never by clicking a bird (clicking a bird is listen-in). The panel offers three things: a seed, a song fragment from a small motif library, a still pool.

The offer is made **to the aviary**, not to a bird. It lands in the scene; birds decide independently.

- **Seed** — appears front-middle. Response by `V_curious` and mood: curious/content birds approach; wary birds wait, then may come near; drowsy birds usually do not.
- **Song fragment** — a motif plays softly on the ambient bus. Each bird's response is generated by the call grammar under a `respond-to-motif` transform, so a bird that answers is answering *that motif* — musically related, not a random call. Responses by `V_vocal` and mood: join, go quiet, or call against.
- **Still pool** — a soft reflective surface at the front. Some birds drink, some bathe, some watch.

**Cooldown: 4 minutes per bird, and it is invisible.** No countdown, no greyed-out control, no clock icon, no "try again in 3:41." If every bird is in cooldown, the offer still lands — a seed appears — and the birds simply do not come. That reads as birds not being interested right now, which is naturalistic and true. A visible cooldown timer would convert a gesture into a game action in one move, which is exactly the failure the PRD's cooldown rationale is guarding against.

### 7.4 Settle

Triggered from the top bar. Over ~6 seconds the lighting warms and dims toward evening, the call schedule thins, the mix decays. The aviary is settled and stays there until tab close or active re-engagement.

**Undo:** any click anywhere in the aviary — or `Escape` — within 5 seconds reverses the shift. No toast, no "undo" button, no confirmation. Just the reversal. (`Escape` is added for keyboard parity; a mouse-only undo would make the mercy inaccessible.) **[CALL]**

**Engine equivalence.** Settle emits a `settle` event that terminates the presence window. Tab-close, presence timeout, and settle are handled identically. There is no "you didn't settle" surface, no recovery prompt, no difference in drift. Settle is a ritual for people who want one.

### 7.5 Settled state is session-scoped **[CALL]**

The PRD does not say whether settling on the laptop should dim the phone. We make settled lighting **session-local presentation state**, not canonical scene state: a user who settles on the laptop and picks up their phone two minutes later sees a normal daytime aviary, not a dimmed one. The settle *event* remains canonical (it ends presence and nudges moods quieter through the ordinary tick path).

Reasoning: the alternative makes a goodbye on one device look like a malfunction on another, and settle is explicitly framed as a gesture the user makes toward the aviary at the end of *their* session — not a change to the world.

### 7.6 Presence accounting

Client-side `PresenceGate` (INV-1) evaluates:

```
present  =  document.visibilityState === 'visible'
         && documentHasFocus()                       // focus/blur events + document.hasFocus()
         && (now - lastPointerOrKeyEvent) < ACTIVITY_WINDOW
```

`ACTIVITY_WINDOW = 5 minutes`. **[CALL]** The PRD says "a few minutes, leaning toward the longer side because watching birds without moving is the actual product." Five minutes is long enough that a genuinely watching user is never dropped mid-session, short enough that an abandoned open laptop stops counting well within one tick's worth of meaningful drift. Revisit against beta data (§14.4).

Pings every 20 seconds while present; each carries a monotonic sequence number and the three predicate values.

**Server-side integrity.** Presence duration is computed from *server* receipt times, not client timestamps, and each ping credits at most `1.5 × ping_interval`. Clock skew, a suspended laptop, and a modified client all fail closed. This matters more than it looks: presence is the dominant drift input, so a client that can inflate presence can inflate the whole product's central mechanic.

**Multi-device and multi-tab.** Presence-time across concurrent sessions is the **union of intervals, never the sum.** Two devices both showing the aviary for ten minutes yields ten minutes of presence, not twenty. Summing would silently double the drift rate for anyone with two devices open — the exact class of silent calibration corruption the PRD's presence definition exists to prevent. Within one device, a `BroadcastChannel` leader election ensures only one tab sends pings.

**Backgrounded tabs.** The client stops rendering entirely when hidden (`rAF` naturally halts; we also suspend the audio graph and the behavior loop). The simulation continues server-side. On return, the client pulls a fresh snapshot before resuming.

---

## 8. Accounts, sync, and social

### 8.1 Sync model

There is no sync algorithm, and that is the design. One canonical record on the server; clients read snapshots and write events. No client-to-client channel, no CRDT, no merge, no vector clocks, no offline queue that could diverge.

What could still go wrong, and what prevents it:

| hazard | prevention |
|---|---|
| Two devices write conflicting personality | Impossible — clients cannot write personality (INV-4). |
| An old read overwrites a newer write | Impossible — all updates are additive deltas (INV-5); there is no absolute write to overwrite with. |
| Double-counted presence from two devices | Union-of-intervals (§7.6). |
| Duplicate event delivery (retry, `sendBeacon` double-fire) | Idempotency on `client_event_id`, unique index. |
| Tick double-applies a batch | Transaction keyed on `(aviary_id, tick_seq)`; `consumed_at_tick` set in the same transaction. |
| Concurrent presentation state (listen-in, settled) on two devices | Not canonical; nothing to conflict. |
| Visitor attention drifts host birds | Token scope lacks `write:events` (INV-13). |
| Vector row corrupted by a bad migration | `personality_delta_audit` replay + nightly sum-vs-accumulator assertion. |

### 8.2 Snapshot delivery

The tick writes the serialized snapshot to a KV store (Cloudflare KV or equivalent) keyed by aviary id, with the trait fields already stripped. The edge worker reads it directly, so the hot path is a KV get, not a database query — this is what makes the inlined-snapshot trick in §11.4 fast enough to matter.

Freshness bound: at most one tick (60s) plus KV propagation. That is the model, not a compromise; the client is *supposed* to be looking at the last tick and interpolating forward.

### 8.3 Client interpolation and resume

Between snapshots, the client eases birds toward canonical targets and runs idle micro-motion locally. A bird at the back perch in snapshot N and the front perch in N+1 flies there; it never teleports.

**Resume after a long gap** (laptop suspended, phone pocketed): if `now - lastSnapshot > 5 minutes`, do **not** animate the whole delta. Treat it as a cold start — place birds in their current positions with motion already in progress, exactly as on first load (§11.4). Animating eight hours of change in two seconds would be a "here's what you missed" montage, which is an announcement (INV-7) and also a lie about continuity.

### 8.4 Auth details

Magic links: 32 random bytes, hash stored, 15-minute expiry, invalidated on consumption via conditional update. Request rate limits are per-email and per-IP; every response is identical regardless of account existence.

Sessions: per-device, listed in settings with a device label derived from the user-agent (matter-of-fact voice: "Chrome on macOS · last used 2 hours ago"), individually revocable. Session tokens rotate on use with a short overlap window.

Email change: verification is sent to the new address; the old address keeps working until the new one verifies. On success, both addresses receive a notice — the old one so a compromised session cannot silently move an account.

Export: generated asynchronously, delivered as a signed, expiring link emailed to the verified address. Contents: birds (id, name, species, adopted_at), current personality vectors, current moods, notebook entries, settings, visit log. Note that the export is the **one** place trait values leave the system, and that is correct: INV-3 is about not building a stats surface, not about withholding the user's own data when they explicitly ask for it. The export is JSON in a file, not a rendered view; the product never displays it.

Deletion: soft immediately (account marked, sign-in still works, any signed-in page offers a plain restore control), hard at 30 days. Hard delete removes the account row, aviary, birds, vectors, audit rows, notebook, events, invites, visits, and sessions, and enqueues deletion from backups per the retention policy. A weekly job asserts no orphaned rows survive.

### 8.5 Visits

Host enters a visitor email; `visits` creates an invite (30-day expiry) and mails a one-time link. Following it exchanges the token for a **visitor session valid for 24 hours**, revocable at any moment. After that the invite is spent; a returning friend needs a new invite. **[CALL]** — this reads "one-time link" plus "no automatic re-invitation, no permanent visitor list" together, and lands on a bounded, deliberately re-issued affordance.

The visitor sees the host's aviary as it is: same birds, same moods, same drift, same weather, same time of day, same calls. No special rendering, no prettification (a "show-off mode" would betray the point of letting a friend look). The visitor's client omits — not disables — the interaction surfaces: no offer affordance, no settle, no notebook, no listen-in, no settings beyond accessibility and audio. Disabled-with-a-tooltip would be worse than absent; it advertises a thing the visitor cannot have.

The visitor is invisible in the aviary. No avatar, no cursor, no marker, no "someone is watching" cue to the host. No chat, no comments. Visitor sessions contribute nothing to the simulation (INV-13).

Revocation: checked on every `/v1/visit/snapshot` poll; the visitor sees a matter-of-fact surface within one 15s poll. An unused invite that is revoked or expired stops working silently and produces the same surface.

### 8.6 Visit notification — the one exception, reconciled

`product_brief.md` says the product never emails the user about the aviary. `social_optional.md` says the host may opt into visit notifications. These are reconciled as follows: the opt-in visit notification is about **a person**, not about the aviary's state, and it is the only exception. It is off by default, is never mentioned during onboarding, is email only, and there is no push path in the product at all — no service worker push handler, no `Notification.requestPermission()`, ever (INV-18). We ship no re-engagement email of any kind: no digest, no "your birds miss you," no "you haven't visited," no product marketing to the account list. The mailer accepts a hardcoded template enum and there is no other way to send mail from this system.

### 8.7 Permitted outbound email (exhaustive)

`magic_link`, `email_change_verify`, `email_change_notice`, `account_export_ready`, `account_deletion_scheduled`, `account_deletion_complete`, `visit_invitation`, `visit_notification` (opt-in only). Eight templates. Adding a ninth requires changing an enum, which makes it a visible decision rather than a quiet one.

---

## 9. Audio pipeline

### 9.1 Graph

```
AudioWorkletNode (one per bird slot, created once at boot, never recreated)
        │  synthesizes calls in-place from parameter messages
        ▼
  per-bird timbre EQ ──► panner (x from perch zone) ──► bird gain ──┐
                                                    └──► reverb send ┤
  ambient bed (worklet: wind, distant birds, rain) ──► ambient gain ─┤
  offer bus (song fragment motif) ─────────────────────────────────┤
                                                                     ▼
                                        procedurally-generated convolver (short IR,
                                        synthesized at boot — no IR asset ships)
                                                                     ▼
                                            master gain ──► limiter ──► destination
```

**AudioWorklet is the primary and only synthesis path.** One worklet node per bird slot (7 + 2 spare), instantiated once. Zero per-call node allocation, which is the main structural reason the 30-minute memory test can pass (§12.4), and it keeps synthesis off the main thread, which is the main reason 60fps holds while three birds are calling.

The reverb impulse response is *generated* at boot (a few hundred milliseconds of shaped decaying noise) rather than shipped. A convincing room IR is 100–300KB of audio asset, which is both a bundle cost and, arguably, a recorded-audio asset. Generating it sidesteps both.

### 9.2 Synthesis

Per note: an excitation source (band-limited noise plus 2–3 partials with an FM index for roughness) through 2–3 resonant peaking filters that place the timbre band, shaped by a per-note amplitude envelope with a mood-dependent attack, then a pitch glide along the note's contour segment.

Per call: notes sequenced per the motif grammar with the mood and personality transforms of §6.5, and per-instance jitter on every continuous parameter (pitch ±15 cents, duration ±8%, gap ±12%, amplitude ±1.5dB, filter Q ±10%). The jitter is what guarantees no two calls are identical; the identity signature is what guarantees they are all recognizably the same bird.

### 9.3 Mixing

Ambient bed sits at `−18 dB` and is always present. Individual birds sit at `−6 dB` nominal, ducked by perch zone (back perch is quieter and more reverberant — this is how the user *hears* proximity, which reinforces what they see). A soft limiter on the master catches chorus peaks so three simultaneous calls never clip.

Listen-in ramps as specified in §7.2. The ramp is equal-power, not linear, and eased — a linear crossfade over 1.4s has an audible dip in the middle that reads as a mixing artifact.

### 9.4 Captions

Generated at runtime by `describeCall(params, posture)` in the shared grammar package, from the same parameter set that drove the synthesis. Composed from note count, contour direction, tempo band, register band, and roughness: *"a soft three-note rise"*, *"a low trill, paused, low trill again"*, *"a single sharp call from the back perch"*. Naturalist voice, lowercase, present tense (INV-10).

Rendered as small text near the calling bird, fading with the call, over a soft scrim sized to the text so WCAG AA holds against both the brightest midday sky and the darkest night (§10.4). The scrim is a real tension with "no UI chrome inside the aviary" — we resolve it toward legibility, keep the scrim as soft and as small as contrast permits, and note it as a design-review item. Captions are `aria-hidden`: a screen-reader user already receives the narration, and double-speaking every call would be noise.

### 9.5 Autoplay policy — a real constraint the PRD does not address

Browsers require a user gesture before an `AudioContext` can produce sound. A user who opens a tab has made no gesture. So the first seconds of a session — precisely the return-greeting — may be silent, which conflicts with "calls already audible" in the first frame.

Handling, in order:

1. If the context resumes without a gesture (prior media engagement on the origin, or navigation that carried a gesture, such as following a magic link), audio starts immediately with a 300ms fade-in.
2. Otherwise the greeting plays visually — the bird still notices, still steps forward, still opens its bill — and captions render if enabled.
3. On the **first** pointer, key, or touch event, the context resumes with a slow fade-in, as though the sound had been there all along. If that gesture arrives within ~4 seconds of the greeting, the greeter calls again shortly after — not a replay of the missed call, a new one. A bird calling twice is naturalistic; a replayed cue is not.
4. There is **no** "click to enable sound" prompt, banner, or overlay. That is an announcement (INV-7) and it would be the first thing a new user ever saw.
5. The top bar carries a small audio-state icon — the sanctioned chrome location — so a user who wonders about silence has somewhere to look.

This is the sharpest unavoidable compromise in the plan and it is called out so nobody rediscovers it late.

### 9.6 WebAudio unavailable

Graceful silence with captions forced on. No recorded-audio fallback under any condition (INV-6). The state is not announced with a banner; the top-bar audio icon reflects it, and the accessibility settings surface explains it in matter-of-fact voice if the user goes looking.

---

## 10. Voice, notebook, and narration

### 10.1 The copy system

Two typed namespaces:

```ts
naturalist('notebook.greeted_first', { first: 'pip', second: 'wren' })
system('auth.link_expired')   // "We couldn't sign you in. The link may have expired. Try requesting a new link."
```

ESLint import boundary: `system/*` is importable only under `src/surfaces/system/**` (sign-in, account settings, accessibility settings, sync/error surfaces); `naturalist/*` only outside it. No raw string literals in rendered text nodes anywhere — a lint rule catches them. The voice split becomes an architectural property rather than an editorial one, which is what INV-10 requires.

**Banned lexicon**, checked over every string bundle at build time: *welcome back, welcome, streak, achievement, unlocked, level up, badge, XP, score, rank, milestone, congratulations, keep it up, great job, days visited, you've been, don't forget, come back, we miss you, daily, progress, goal, complete*. Matching fails the build. The list is a tripwire, not a semantic guarantee, but it catches the specific class of copy that most reliably leaks in.

**Second person** is banned in naturalist surfaces (a separate lint check): the naturalist voice describes the aviary, never addresses the user.

### 10.2 The field notebook

**Generation.** The tick emits `AviaryFact`s: *pip greeted before wren; first time in 6 days*, *wren stayed on the back perch through the morning*, *a long chorus during the rain*, *no calls for two hours around midday*, *a new bird at the back perch*. The scorer ranks candidates by novelty (first in N days), contrast (deviation from that bird's own recent baseline), and coincidence (two things happening together).

**Sparsity is enforced, not hoped for.** A rate limiter caps entries at 1 per day and targets ~1 per 2–4 days, with a threshold that decays as time since the last entry grows — so a quiet aviary still gets occasional entries and an intensely-used one does not flood. The PRD is explicit that an observation per session dilutes the ones that matter; a scorer alone will not hold that line under a heavy user, so the limiter does.

**Prose.** Composed by the naturalist grammar from the fact plus slot-filled specifics, with per-aviary anti-repetition memory (no phrasing template reused within the last 20 entries). Not an LLM at runtime — latency, cost, and unpredictability are all wrong for a surface where the voice is load-bearing and a single off-register sentence damages the whole product. **[CALL]** An LLM *is* the right tool offline, for authoring and expanding the template corpus under human review; that corpus is the thing that ships.

**Boundary (INV-9).** `NotebookGenerator` accepts `AviaryFact[]` and nothing else. There is no `UserFact` type in its dependency graph, so "you visited every day this week" is not a sentence the system can construct — not because we remembered not to write it, but because the fact type it would need does not exist. This is the strongest available defense of the line the PRD draws between observations of the aviary and observations of the user.

**Behavior.** Read-only, uneditable, unannotatable, never archived, infinite scrollback via cursor pagination. Entries are immutable once written; if we improve the grammar later, only future entries change — rewriting an entry the user has already read would be editing their record of what happened.

### 10.3 Screen-reader narration

A running naturalist prose description of the aviary, generated server-side (in the snapshot) with a client-side generator for locally-triggered events.

> a small grey bird is perched on the front rail, calling softly. another sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Cadence: one update per 30–60 seconds at idle. Faster only for user-initiated events — a return-greeting, an offer reaction, a settle. Even those are written as observations, never as state transitions.

Implementation: two polite live regions — an ambient one for the slow cadence, and a second, also polite, for prioritized user-initiated events, cleared before writing to prevent queue pileup. **Never `aria-live="assertive"`**: assertive interrupts, and interrupting is announcing.

The generator's input type is `Posture`/`Behavior` descriptors, never the `Mood` enum and never a trait value (INV-16). It cannot say "wren is drowsy" because it never receives that; it says "wren sits low with feathers fluffed," which is what a person watching would say and what a person listening deserves.

### 10.4 Accessibility surfaces

**Semantics.** The canvas is `aria-hidden`. A parallel DOM layer positions one focusable element per bird over the canvas. Each is a `<button>` with a naturalist accessible name — *"wren, on the low perch, fluffed"* — updated on snapshot. **[CALL]** `button` means screen readers announce "button," a small semantic leak against the naturalist register; we accept it because listen-in is genuinely actionable and a user navigating by keyboard needs to know that. Charm loses to navigability here, deliberately, once.

**Keyboard.** Tab through top-bar items → Tab into the scene focuses the first bird → arrow keys move between birds (roving `tabindex`) → `Enter` engages listen-in → `Escape` exits listen-in, and also cancels a settle within its 5-second window. The offer panel opens from the top bar and is fully keyboard-navigable with a focus trap and `Escape` to close. Settle is reachable from the top bar.

**Focus indicator.** A dual-stroke outline (dark inner, light outer) so it reads against both a bright midday sky and a dim night scene, plus a subtle non-color cue for forced-colors mode. Focus visibility is never suppressed, including in reduced-motion.

**Contrast.** All user copy passes WCAG AA. The demanding cases are captions and the top bar over a scene whose luminance varies with the day cycle, so both are tested against the extremes of the palette — midday bright, deep night, and the peak of the sunset warm shift — in an automated contrast suite rather than checked once against a mid-tone mock.

**Reduced motion** (INV-15). A distinct presenter over identical state:

| standard | reduced-motion |
|---|---|
| continuous micro-motion at 60fps | slow cross-fades between still poses, ~4 updates/sec |
| animated flight between perches | cross-fade through one brief mid-air pose, ~900ms |
| ambient leaf/feather drift | removed |
| parallax on scroll-free scene | removed |
| day→evening color transitions | retained, slowed |
| top-bar fade | retained, lengthened |
| calls, drift, mood, notebook, narration | **identical** |

The mid-air intermediate pose in cross-faded flight is deliberate: a straight A→B cross-fade reads as teleportation, which reads as broken. Reduced-motion should be calmer, not wronger.

Triggered by `prefers-reduced-motion` and overridable in both directions from accessibility settings (some users want it without the OS setting; some want full motion despite it).

---

## 11. Frontend rendering pipeline

### 11.1 Renderer choice

**Canvas2D, single path.** **[CALL]**

The scene is seven birds, three or four parallax layers, a handful of particles, and a global palette shift. That is well inside Canvas2D's budget with layer caching. The decisive argument is the 500ms first-bird budget: WebGL context creation plus shader compilation costs 30–100ms on a mid-tier mobile device, and can stall unpredictably on older GPUs. Spending a fifth of the entire budget on graphics initialization for headroom we do not need is the wrong trade. A single path also means one set of visual regression baselines and one performance profile.

Escape hatch: if profiling at milestone M3 shows the frame budget failing on the reference laptop, we revisit WebGL for the compositing layers only, keeping birds in Canvas2D.

### 11.2 Component library

The chrome (top bar, settings, notebook, offer panel, auth surfaces) uses Preact + a small hand-rolled component set. Notably **absent by design**: any toast, snackbar, banner, modal-alert, spinner, skeleton, progress bar, badge, or counter primitive. They do not exist in the library, so adding one is a visible act of creation rather than an import (INV-7, INV-8, INV-11). A "just a small toast" pull request has to build the toast first, which is exactly the friction we want.

### 11.3 Bird rendering

Birds are drawn from per-species vector path data (geometry ships, bitmaps do not) with a **per-bird pose atlas rasterized at session start** using that bird's current plumage palette. Plumage drifts on a weekly timescale, so a per-session cache is always fresh; the atlas is invalidated only when the quantized saturation step changes.

To protect the first-bird budget, only the 2–3 poses needed for the first frame are rasterized synchronously (a few hundred microseconds each); the rest fill in during `requestIdleCallback` over the following second.

Poses are parameterized (head angle, body lean, feather fluff, wing position, eye openness) and driven by the behavior layer, so "preening" is a continuous parameter trajectory rather than a canned clip.

### 11.4 The first frame

The most affectively important 500 milliseconds in the product.

```
0ms      navigation
0–110ms  HTML from CDN edge. The document carries:
           · inline critical CSS
           · inline bootstrap JS (~8KB gz): reads local time, paints the sky
           · for signed-in users, the state snapshot inlined as JSON
             (edge worker reads the KV snapshot cache — no origin round trip)
110–190ms sky painted at the correct time-of-day color. This is already the aviary.
190–370ms renderer module (~45KB gz, modulepreload) arrives, parses, boots
370–450ms first poses rasterized; birds placed at snapshot positions, mid-action;
          behavior loop starts; first frame drawn
450ms+    remaining poses, audio worklets, ambient particles, chrome hydrate
```

Two things make this work. First, **the sky is drawn from local time with zero network dependency** — the very first paint is the aviary's sky, so there is never a blank or white frame to disguise. Second, **the snapshot is inlined in the HTML at the edge**, removing a full round trip; birds can be positioned the moment the renderer boots.

Budget arithmetic worth stating plainly: on 4G (~1.5–3 Mbps effective, ~100–150ms RTT), 500ms permits roughly 60–90KB on the critical path. The PRD's 2MB gzipped cap is the ceiling for the *whole application*; the first-bird path needs its own far tighter sub-budget, and that sub-budget — not the 2MB number — is what the CI gate enforces most strictly (§12.1).

**No entry animation. No fade-from-static. No spinner. No skeleton** (INV-11). Birds appear already in motion, at the positions the simulation says they occupy.

**Slow-network state:** the quiet field — soft sky at the right time of day, a slow gradient breath, perhaps one drifting leaf — held until the snapshot arrives. Identical to the empty-aviary state between adoption and first bird. It reads as the aviary catching up, not as the product loading.

**Failure state:** if the snapshot fails, keep the quiet field behind a matter-of-fact message (INV-10) — "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." Never flash to white, never show an error page chrome.

### 11.5 Scene composition and layout

Layers, back to front: sky gradient (palette from time of day) → far foliage → mid foliage → **perch plane with birds** → occasional foreground branch/leaf. Parallax is subtle — a few pixels of differential offset tied to gentle idle drift, not to pointer position. Pointer-driven parallax would make the scene respond to the cursor, which converts a window into an interactive toy.

Responsive: a fixed logical scene of 3 perch zones × N slots, mapped to the viewport by *widening the gaps*, not by scaling or cropping. All birds are always fully in frame at every viewport from a 320px phone to an ultrawide desktop (`aviary_layout.md` is explicit that no bird is ever cropped). On narrow viewports the perch zones compress horizontally and the vertical separation between zones increases slightly to preserve the front/back reading. No panning, no scrolling, no zooming.

Slot assignment is stable within a session so birds do not shuffle on resize.

### 11.6 Ambient ornaments

Leaves and feathers are pure client-side rendering ornaments with no simulation state (`aviary_layout.md` says so explicitly). Spawned from a bounded pool (max 6 concurrent) at Poisson intervals, with drift paths from layered noise. They are the cheapest possible signal that the scene is continuing, and keeping them out of the simulation keeps snapshots small and the tick cheap.

### 11.7 Top bar

Four icons: account/settings, accessibility settings, field notebook, offer. Nothing else — no bird count, no notification dot, no badge, no status indicator beyond the audio-state icon of §9.5.

Fade to ~8% opacity after 3 seconds of cursor stillness; back to full on pointer move, key press, or focus. Under reduced-motion the fade duration lengthens rather than disappearing. The bar is never fully invisible (a control the user cannot find is worse than a faint one), and it is always fully opaque while any of its controls has keyboard focus.

### 11.8 Behavior loop

Fixed 30Hz behavior step; rAF render with interpolation between behavior steps. Idle micro-motion comes from per-bird phase-offset simplex noise across pose parameters, so it never loops and never lines up between birds. Rate-limited to mood: a drowsy bird's noise amplitude and frequency are both scaled down.

When the tab is hidden: rAF halts naturally; we additionally suspend the audio context and stop the behavior loop, so a background tab costs approximately nothing.

---

## 12. Performance and observability

### 12.1 Budgets and their gates

| budget | value | CI gate |
|---|---|---|
| Total initial JS | ≤ 2MB gzipped | `size-limit` on the entrypoint graph; build fails |
| **First-bird critical path** | **≤ 90KB gzipped** (inline bootstrap + renderer module) | separate, stricter `size-limit` group; build fails |
| Time to first bird | < 500ms, mid-tier mobile, 4G | Playwright + CPU throttle 4× + network throttle, on a real device farm nightly; p75 across 20 runs |
| Idle frame rate | 60fps sustained on a 5-year-old mid-range laptop | reference machine in CI; p95 frame time ≤ 16.7ms over a 5-minute run |
| Memory growth | zero over 30 minutes | heap snapshot soak; fail on >5MB retained growth or any monotonic node-count growth |
| Tick latency | p99 < 5s (alarm) | production alarm; load test in CI |
| Snapshot payload | ≤ 8KB gzipped at 7 birds | contract test |

Code-splitting: account settings, accessibility settings, the notebook, the offer panel, and the invite flow are all lazily loaded. Only the aviary and the top bar are in the initial graph.

### 12.2 The 60fps budget, allocated

Per 16.7ms frame on the reference machine: behavior step (amortized, 30Hz) ~1.5ms; bird rendering (7 birds from cached atlases) ~3ms; parallax layers (cached, blitted) ~2ms; palette overlay composite ~1.5ms; particles ~0.5ms; caption/DOM layer ~1ms. Leaves ~7ms of headroom, which is where reality goes. Audio is on the worklet thread and does not consume the frame budget — a major reason for the AudioWorklet choice.

### 12.3 Time-to-first-bird measurement

Custom mark `first_bird_drawn`, emitted from the render loop on the first frame containing a bird, reported via RUM as an aggregate histogram with **no per-account dimension** (INV-14). Synthetic checks run the same path from four geographies on a schedule and are our primary regression signal, since they are stable and carry no user data at all.

### 12.4 Memory

Structural rather than aspirational: audio worklet nodes are created once and never recreated; call parameter objects come from a pool; the reverb IR is generated once; pose atlases are per-bird and bounded; particles come from a fixed pool; notebook rows are virtualized and detach their DOM and their observers on scroll-out; the snapshot cache retains only the last two snapshots; every `addEventListener` is paired with a teardown registered in a session-scoped disposer.

The soak test is a real CI job on every merge to main, not a manual check — the PRD calls this out specifically and it is the kind of test that stops being run the moment it is optional.

### 12.5 What we measure

Server: request rate/latency/error by endpoint, tick latency p50/p95/p99, tick lag (`now − sim_time`) distribution, events consumed per tick, dormant-batch sizes, snapshot cache hit rate, KV propagation delay, mail delivery success, magic-link request/consume/failure rates, invite consume rates, DB connection and replication health.

Client (aggregate RUM only): page load timings, `first_bird_drawn`, frame-time histograms, audio-context error counts, JS error rates by surface, unsupported-browser counts.

Synthetic: full-journey checks from four geographies — cold load, greeting fires, calls schedule, offer round-trips, notebook loads, magic-link flow against a test account.

### 12.6 What we deliberately do not measure

No DAU/WAU/MAU as a product goal. No retention cohorts framed as engagement targets. No session counts per user. No visit frequency, per user or aggregate — the query does not exist in the codebase (INV-8). No per-bird or per-account dimension on any metric. No aggregate over drift ("average boldness across accounts" is exactly the data product `accounts_sync.md` forbids). **No event-type breakdown on the interaction-event endpoint** — we count that events were accepted and how fast, never what they were.

That last one costs us real product insight: we will not know from production data how often people offer seeds versus song fragments. We accept it, because a strict reading of "never used to inform population-level analysis of how birds are typically interacted with" forbids exactly that aggregate, and because synthetic monitoring gives us functional health without touching a single user's data. Where we need product understanding, we get it from beta interviews and opt-in research sessions with people who know they are in a study.

Metric definitions live in one registry with a typed dimension allowlist. Adding a dimension requires editing the allowlist, which is reviewable.

---

## 13. Privacy and the data boundary

Per-bird interaction events exist to drive that user's own simulation and for nothing else. Never aggregated, never used for training, never shared, never used for cross-user features, never used for population analysis of how birds are interacted with.

Enforcement, in order of strength:

1. **Network and IAM isolation.** The simulation Postgres cluster is in its own VPC in its own cloud account. No warehouse, BI, or ML credential can reach it. No CDC, no replica, no scheduled export.
2. **Terraform CI check** that fails on any resource creating such a path.
3. **Metric dimension allowlist** (§12.6).
4. **Encryption.** Email is envelope-encrypted with KMS. Vectors and events are protected by at-rest volume encryption; the sensitivity here is linkage, and the identifier discipline of INV-19 is what addresses that.
5. **Retention.** Interaction events: 90 days. Notebook and vectors: account lifetime. Everything: hard-deleted 30 days after deletion request, including from backups per the documented backup retention schedule.
6. **Log redaction.** A structured-logging middleware drops any field named `email`, any email-shaped string, and any personality field. Log samples are scanned in CI.

The privacy policy lives in account settings as plain-text prose naming the aggregate categories collected and explicitly excluding per-bird interaction state. It is written in matter-of-fact voice (INV-10).

---

## 14. Rollout

### 14.1 Milestones

| # | milestone | contents | exit criteria |
|---|---|---|---|
| **M0** | Foundations (3w) | repos, CI, infra, Postgres schema, identity service, magic link, sessions | a user can sign in and out; INV-19 lint and log-redaction checks green |
| **M1** | Tick skeleton (4w) | tick loop, event log, snapshot serialization, KV publication, batched-catch-up equivalence test | tick runs at cadence for 10k synthetic aviaries; equivalence property test green |
| **M2** | Engine (5w) | drift filter, mood FSM, perch logic, call scheduler, weather, bird-to-bird contagion | accelerated harness reproduces the §6.3 calibration targets; INV-2 property test green |
| **M3** | Renderer (5w) | Canvas2D pipeline, species geometry, pose system, parallax, day/night, quiet field, first-frame path | first-bird p75 < 500ms on the device farm; 60fps on the reference laptop; renderer-vs-WebGL decision recorded |
| **M4** | Audio (4w) | worklet synthesis, six species grammars, chorus, listen-in mix, captions, autoplay handling | recognizability panel ≥ target at 7 birds; no-two-calls-identical test green |
| **M5** | Interactions + notebook (4w) | greeting, listen-in, offers, settle, presence gate, notebook generation, voice system + lint boundaries | charm suite green; banned-lexicon and namespace-boundary gates enforced |
| **M6** | Accessibility (4w, overlapping M3–M5) | narration, reduced-motion presenter, keyboard nav, focus, contrast suite | axe clean; reduced-motion parity test green; screen-reader review with external testers passed |
| **M7** | Social + account lifecycle (3w) | invites, visitor sessions, revocation, visit log, export, deletion lifecycle | INV-13 tests green; revocation within one poll verified |

Roughly 5.5–6.5 months to closed beta with meaningful overlap, then the beta window below, then GA. Accessibility work (M6) is scheduled *inside* the build, not after it — `accessibility_perf.md` is explicit that a reduced-motion mode shipped as a v1.1 fix is a v1 that told those users the product was not for them, and the schedule has to reflect that or it will not happen.

### 14.2 Team shape

Two backend engineers (simulation, identity/sync), two frontend engineers (renderer/scene, chrome/interactions), one audio-capable engineer (DSP and call grammar — this is a specialist role and hiring for it should start at M0), one accessibility-specialist engineer sharing time with frontend, one designer (visual system, palette, focus treatment, reduced-motion aesthetic), one writer owning the naturalist corpus and the system-voice strings. The writer is not optional: the notebook and the narration are the product's voice, and a template corpus written by engineers under deadline is how a product like this loses its charm.

### 14.3 Validating drift — the schedule constraint that cannot be compressed

Two harnesses, both required.

**Accelerated harness.** The tick's pure core (§6.1, steps 3–9) run in-process with no database, at ~10,000× real time, against scripted user archetypes: the daily 5-minute visitor, the weekly hour-long visitor, the binge-then-vanish user, the two-device user, the listen-in-only user, the offer-masher, the two-week-absentee. Asserts the §6.3 calibration targets, the per-session and per-day budgets, monotonicity across every archetype, and that a year of use lands in a sensible place. This validates the math and runs in CI in under a minute.

**Real-time internal alpha.** Twenty internal accounts running against production infrastructure for at least six weeks. This is the only way to validate the *feel* — whether three weeks of visits produce a change a person actually notices when they look back, which is a perceptual claim no simulation can settle. The wall-clock cost is irreducible: you cannot compress three weeks of felt drift, and adding engineers does not help. Start it at the end of M2, before the renderer is finished, using an instrumented debug client (which never ships, and which is the one place trait values are visible to us — internal accounts only, behind an infrastructure-level flag that cannot be enabled for a real account).

### 14.4 Closed beta

~200 invited users, minimum eight weeks — long enough to evaluate the three-week visible-drift claim with room on either side. What we learn from, and how:

- Qualitative interviews at weeks 1, 4, and 8. The key question is asked without priming: *"has anything about your birds changed?"* If users at week 4 spontaneously describe a change, drift is calibrated. If they say no, `k` is too low. If they describe changes at week 1, it is too high.
- The presence activity window (§7.6) validated against instrumented sessions from consenting beta users only, under explicit study consent, and discarded after analysis.
- Recognizability panels at 3, 5, and 7 birds (§14.5).
- Standard operational telemetry.

We do not instrument beta users the way a normal beta would. The privacy commitment applies to beta accounts too; what we learn beyond operational health, we learn by asking people.

### 14.5 Ramping birds-per-aviary

The age gate means no organic account reaches three birds for 75 days, so "ramping" is really about validating the seven-bird ceiling *before* anyone gets there.

1. Synthetic aviaries at 3, 5, and 7 birds, exercised in the audio harness.
2. **Recognizability panel:** listeners spend a session with a named bird, then identify its calls from a mixed chorus at each bird count, across mood and drift states. Target ≥80% identification at 7 birds. If 7 fails, the cap drops — it is a server-side config value, changeable without a deploy, and `bird_engine.md` explicitly frames 7 as empirical.
3. A cohort of ~20 beta accounts seeded with backdated `created_at` so real people live with 5–7 bird aviaries for weeks before GA.
4. The cap is enforced in the engine, not the UI.

### 14.6 Launch and day one

Public launch is a plain sign-up page and the product. No launch modal, no onboarding tour, no feature callouts, no tooltips-on-first-run, no "here's how to listen in." The aviary teaches itself by being watched; a product whose entire thesis is *notice, never announce* cannot open with a five-step tour.

The one exception is adoption: the user names two birds. That surface is functional and brief, drops into naturalist voice ("two birds have arrived. what will you call them?"), and provides default suggestions the user can accept in one action.

Feature flags: server-side, bucketed on the synthetic account UUID (never email — INV-19). Kill switches for weather, the notebook generator, visits, and the bird-offer scheduler, so any one can be disabled without a deploy.

Rollback: schema changes are expand/contract, always backward-compatible for one release. The tick is versioned; a bad tick version can be rolled back and its ticks replayed from the audit table.

---

## 15. Testing strategy

### 15.1 Simulation

Property tests: monotonicity across 10⁵ random event sequences (INV-2); batched-vs-stepwise tick equivalence (§6.2); tick purity (same inputs → same outputs, across process restarts); idempotency under duplicated and reordered event delivery.

Calibration tests: the archetype suite of §14.3, asserting week-1 measurable and week-3 visible thresholds with tolerance bands, and asserting that a marathon session cannot exceed the per-session budget.

Fuzzing: random event streams including malformed, replayed, out-of-order, and future-timestamped events; assert no crash, no negative drift, no presence inflation.

### 15.2 Presence

Truth-table test over all 8 combinations of the three predicates (INV-1). Clock-skew tests with clients reporting timestamps hours off. Multi-device union tests. Multi-tab leader-election tests. A hostile-client test that submits 100 pings per second and asserts credited presence stays within the server-side cap.

### 15.3 Sync

Two simulated clients driving the same account concurrently, asserting no personality write from either, no double-counted presence, and identical resulting state regardless of interleaving. Visitor-session tests asserting event rejection (INV-13) and revocation within one poll.

### 15.4 Rendering

Visual regression across: cold load, slow-network quiet field, empty aviary, resume-from-suspend, each time-of-day band, each weather state, reduced-motion at each of the above, each viewport breakpoint from 320px to 2560px, and focus states over the brightest and darkest scenes. Frame-time regression on the reference machine. A "no bird cropped" assertion computed geometrically at every breakpoint.

### 15.5 Audio

Determinism: same seed tuple → identical parameters → identical caption. Distinctness: hash 10⁵ generated call parameter sets and assert a near-zero collision rate. Identity preservation: a classifier trained on N calls per bird must identify the bird above threshold across every mood × drift-state combination — this is the automated proxy for the recognizability requirement, backstopped by the human panel of §14.5. Mix tests asserting no bird reaches silence during listen-in and no clipping during a three-bird chorus.

### 15.6 The charm suite

A named CI job that encodes the invariant register: banned lexicon; voice-namespace import boundary; no raw text literals; absence of toast/spinner/skeleton/badge primitives; no audio assets in the bundle; personality fields absent from every client-bound response; notebook generator's dependency graph free of user-fact types; narration generator's signature free of `Mood` and trait values; no `Notification.requestPermission()` anywhere; no push service worker.

It runs on every pull request that touches client code and it blocks merge. It exists because these are the rules most likely to erode quietly, and a rule that only lives in a document erodes on a schedule.

### 15.7 Accessibility

`axe` on every chrome surface. Focus-order snapshots. Contrast assertions against palette extremes. A narration-cadence test asserting no more than two updates per minute at idle. A live-region politeness test asserting `assertive` is never used. Manual screen-reader passes (VoiceOver/Safari, NVDA/Firefox, JAWS/Chrome) each milestone, plus paid sessions with external screen-reader users at M6 and before GA — the naturalist-narration goal is a qualitative one and cannot be validated by automation.

---

## 16. Risks

**R1 — Drift calibration is wrong and we cannot tell for weeks.** *The central schedule and product risk.* Too fast and the product becomes a Tamagotchi where clicking moves numbers; too slow and it is a screensaver. Signal: beta users at week 4 who cannot describe any change, or week-1 users who can. Mitigation: the accelerated harness for the math, the six-week internal alpha for the feel, `k` and the trait weights as server-side config so recalibration does not need a deploy, and a documented procedure for re-running accumulators from the audit table if a calibration change should apply retroactively. Do not gate GA on perfect calibration; gate it on being inside the band and on having the ability to move.

**R2 — Sync correctness failures are silent.** A lost or under-applied delta produces a bird drifting slightly slower than it should. No test fails, no alert fires, and the user feels something is off without being able to name it — the exact failure `bird_engine.md` calls the worst possible. Mitigation: the delta audit table, the nightly sum-vs-accumulator assertion, alerting on tick lag and on unconsumed-event age, transactional consume-and-apply, and the batched-equivalence property test. Treat any personality-vector anomaly as a Sev-1 regardless of user impact, because by the time there is user impact the data is gone.

**R3 — Procedural calls land in the uncanny valley.** Synthesized birdsong is genuinely hard; it can sound like a synthesizer imitating a bird, which is worse than silence. Signal: internal alpha listeners describing the audio as "electronic," "video-gamey," or "a ringtone." Mitigation: budget real DSP time (four weeks at M4 is a floor, not a target); hire or contract someone who has done this; prototype the synthesizer at M0 as a standalone page and evaluate it before committing to the architecture around it; iterate against recordings of real species; hold listening panels early. **Escalation path:** if the calls cannot be made convincing, the correct response is *fewer, sparser, quieter calls with more ambient bed* — not recorded audio (INV-6). A bird that calls rarely and well beats a bird that calls often and synthetically.

**R4 — Accessibility regresses after launch.** Naturalist narration and the reduced-motion presenter are unusual surfaces that a routine feature change can quietly degrade — a new posture with no narration phrase, a new transition with no cross-fade equivalent. Mitigation: parity tests in the charm suite, a named accessibility owner, a PR template question ("does this add a visual state? what does it sound like?"), and recurring external-user sessions rather than a one-time audit.

**R5 — The 500ms first-bird budget fails on real mid-tier devices.** Lab numbers flatter. Signal: device-farm p75 above budget. Mitigation: the separate 90KB critical-path gate (which is the real constraint), the inlined edge snapshot, the local-time sky that needs no network, Canvas2D over WebGL. If it still fails: shrink the initial pose set, defer the parallax layers past first bird, and — last — accept a slightly later first bird on the slowest devices rather than compromising to a spinner. The quiet field is a *good* state; a spinner is not.

**R6 — Charm erodes by a thousand reasonable pull requests.** The PRD says repeatedly that a well-meaning contributor will add the toast, the streak, the cooldown timer, the "you unlocked a new bird" celebration. Mitigation: the invariant register, the charm suite, the absent primitives, the type-level impossibilities. Each of those converts "please don't" into "you cannot without visibly deciding to." This is the risk the plan's structure is most designed around.

**R7 — Autoplay policy makes the first session silent** (§9.5). Signal: RUM audio-context error and deferred-resume rates. Mitigation: the four-step handling; if deferred resume proves common, consider having the greeter call again on the first gesture in a wider window. Do not add a "tap for sound" prompt.

**R8 — Tick cost at scale.** Signal: tick lag distribution creeping, dormant batch sizes growing. Mitigation: the active/dormant scheduling split, sharding by UUID hash, the batched-equivalence guarantee that lets us reschedule freely without changing behavior. Load-tested to 10× projected accounts at M1.

**R9 — Seven birds blur the chorus.** Signal: recognizability panel below target at 5 or 7. Mitigation: individuation offsets with minimum-distance constraints; per-bird mix ducking by perch zone; the cap as server config, lowerable without a deploy. `bird_engine.md` frames 7 as empirical, so lowering it is a valid outcome, not a failure.

**R10 — The naturalist corpus goes stale.** A template grammar large enough to feel fresh for six months may not feel fresh for two years; a returning user who sees the same three sentence shapes learns the notebook is a machine. Mitigation: anti-repetition memory per aviary; corpus size as a tracked metric; a quarterly authoring pass; entries immutable so improvements affect only future writing.

**R11 — Support cannot debug without seeing state.** INV-3 means support has no view into a user's birds. Mitigation: support tooling shows account health only — tick lag, error rates, session count, whether a magic link was delivered. Anything requiring the actual vector requires the user's own account export, which they control. We accept slower support in exchange for the rule holding.

---

## 17. Calls made where the PRD is silent

Every **[CALL]** above, collected. Each is defensible from the PRD's stated intent; each is cheap to revisit.

1. **Mood set finalized** as wary, content, curious, drowsy, alert, **roosting** — the PRD's five plus a night state, which `aviary_layout.md` requires ("most birds are settled... eyes closed, low on the perch"). §6.4
2. **Daily mood reset is a dawn re-derivation, not an assignment** — the only way to honor "resets daily-ish" and "never snaps to a default" simultaneously. §6.4
3. **Presence activity window = 5 minutes**, per the PRD's instruction to lean long. Config-driven; validated in beta. §7.6
4. **Presence across devices is the union of intervals, never the sum.** Summing would silently double drift for two-device users — exactly the corruption class the presence definition exists to prevent. §7.6
5. **Settled lighting is session-scoped, not canonical.** Settling on the laptop should not dim the phone. The settle *event* stays canonical. §7.5
6. **Escape also cancels settle** within the 5-second window, for keyboard parity with the click-anywhere undo. §7.4
7. **Listen-in decays** after 10 minutes without presence, and on settle, so a forgotten listen-in does not leave the mix permanently unbalanced. §7.2
8. **Offer cooldown is invisible** — no timer, no disabled control. An offer during cooldown still lands; the birds simply do not come. A visible cooldown is a game UI. §7.3
9. **Bird-offer thresholds: 75/165/270/400/560 days**, fitted to "a few months → third; a year-old aviary may have grown to five or six." §6.9
10. **A new bird arrives as a bird**, not as a modal or a celebration — present at the back perch, unnamed, declinable by inaction. §6.9
11. **Nightjar excluded from starter pairs**; starters are chosen for maximal call distinctness. §6.6
12. **Timezone changes debounce over 24 hours / two sessions**, so travel does not whipsaw the aviary. §6.7
13. **Solar times derived from a per-timezone latitude table**, not from user location — seasonally correct within ~20 minutes, and no location data held. §6.7
14. **Scene follows the aviary's timezone, not the device's**, so scene and mood never disagree during a timezone migration. §6.7
15. **Canvas2D, single render path**, chosen substantially because WebGL init cost is unaffordable against the 500ms first-bird budget. §11.1
16. **Naturalist prose is a template grammar, not a runtime LLM** — latency, cost, and unpredictability are all wrong where voice is load-bearing. LLMs are an offline authoring tool for the corpus. §10.2
17. **Birds are `<button>` elements** with naturalist accessible names. Screen readers will announce "button," a small register leak accepted once in exchange for navigability. §10.4
18. **Visitor sessions last 24 hours** from a one-time link, then the invite is spent. §8.5
19. **Visitor polling at 15s** bounds revocation delay honestly rather than adding a WebSocket tier for a rarely-used feature. §5.3
20. **The opt-in visit notification is the single email exception**, reconciling `product_brief.md`'s no-notification rule with `social_optional.md`'s opt-in toggle: it is about a person, not the aviary; email only; never push. §8.6
21. **No event-type breakdown in telemetry.** The strict reading of the privacy commitment forbids population-level analysis of how birds are interacted with; we take the strict reading and accept the lost insight. §12.6
22. **First-bird critical path gets its own 90KB budget**, separate from and much stricter than the PRD's 2MB application ceiling, because 500ms over 4G does not permit more. §11.4, §12.1
23. **Interaction events are retained 90 days.** They are tick inputs, not a user-facing history; the vector is the durable artifact. §4.3
24. **The account export is the one place trait values leave the system** — INV-3 forbids building a stats surface, not returning the user's own data on request. §8.4
25. **An internal debug client that displays vectors exists for the alpha only**, gated at the infrastructure level and unable to target a real account. §14.3

---

## 18. Definition of done for v1

A user can sign in with a magic link, name two birds that arrived, and open a tab to find an aviary already in motion within half a second, where one bird notices them differently than it did yesterday. They can listen in, offer a seed, settle the aviary, and read a notebook that has written three specific sentences about the last week and nothing about them. On their phone that evening it is the same aviary in the same mood. Three weeks later, without being told, they notice Pip comes forward now. A screen-reader user has the same three weeks, in prose. A reduced-motion user has the same three weeks, in cross-fades. Nothing in the product has ever counted anything at them.
