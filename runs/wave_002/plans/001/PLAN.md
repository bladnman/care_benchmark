# Pocket Aviary — v1 implementation plan

This document turns the PRD into an executable plan for a frontier engineering team. It does not restate the spec; it makes the engineering decisions the spec leaves open, pins numbers where the spec says "calibrate during build," and names the tests that keep each product promise true. Where the PRD is ambiguous or internally tense, the call is made here and recorded in §17 (decision log).

Conventions used below: **MUST** = a release-blocking invariant with an automated check; **default** = an initial value exposed as server-side configuration, expected to move during calibration; `code` = a concrete identifier the team should use.

---

## 1. Engineering invariants

The PRD's five principles become fourteen invariants. Each has an enforcement mechanism, because a principle without a test is a suggestion.

| # | Invariant | Enforcement |
|---|---|---|
| I1 | **Single writer.** Only the simulation tick writes canonical aviary state (personality, mood, perch, weather, notebook). No API handler mutates these tables. | DB role separation: the `api` role has no UPDATE grant on `birds`, `aviaries`, `notebook_entries`; only the `sim` role does. Integration test asserts grants. |
| I2 | **Drift is monotonic and bounded.** Every trait delta is ≥ 0, ≤ a per-day cap, and traits never exceed 1.0. | Property tests over random event streams; a runtime assertion in the tick that aborts the transaction if any delta is negative. |
| I3 | **Presence is the conjunction.** A presence interval exists only when `visibilityState === 'visible'` AND `document.hasFocus()` AND activity within the activity window. | Client presence tracker is a pure state machine with unit tests; server unions and clamps intervals; a synthetic-browser test leaves a tab open unfocused for 10 minutes and asserts zero credited presence. |
| I4 | **Raw personality values never leave the simulation service** except inside the account export file. Snapshots carry derived, quantized expression bands only. | Snapshot schema is a closed allowlist validated at the API boundary; a contract test fails if any field named `boldness`, `warmth`, `vocal`, `plumage`, `curiosity`, or `attention` appears in a snapshot. |
| I5 | **Bird identity is stable.** `bird.id` is assigned once at arrival and never reissued, re-derived, or replaced by any migration or sync path. | Migrations are reviewed against a checklist; a migration test replays a fixture aviary through every migration and asserts bird ids and personality values are unchanged. |
| I6 | **No recorded audio, anywhere.** All sound is synthesized at runtime. | CI rejects any `.mp3/.wav/.ogg/.m4a/.flac/.aac/.webm` file and any binary blob > 64 KB under `client/`. No `<audio>` elements; ESLint rule. |
| I7 | **No announcement surfaces.** No toasts, banners, modals-on-return, badges, counters, or celebratory states. | No `Toast`/`Banner`/`Badge`/`Snackbar` component exists; ESLint `no-restricted-syntax` bans those identifiers; PR template has a "no new announcement surface" attestation; design review gate. |
| I8 | **No user-behavior surfaces.** The notebook, narration, and captions describe the aviary, never the user. No visit-frequency, streak, or count of any user action is computed for display. | Copy templates are linted for second person and for forbidden variables (`visitCount`, `daysVisited`, `streak`); no such columns exist. |
| I9 | **Two voices, one registry.** Every user-visible string comes from a copy registry that tags it `naturalist` or `system`; naturalist strings are lowercase, present tense, no exclamation, no "you"; system strings are sentence case, direct, no bird vocabulary. | Copy lint on the registry; ESLint bans string literals in UI components. |
| I10 | **Privacy boundary is architectural.** Telemetry never carries an account or bird dimension; the analytics stack has no network path to the simulation database; per-event interaction data is deleted after it is consumed. | Metrics client accepts only an allowlisted label set; network policy denies analytics→sim-db; a retention job with a test. |
| I11 | **Synthetic UUID only.** Email appears on exactly one column (encrypted) plus a keyed blind index used solely for sign-in lookup. | Log redaction middleware; grep-based CI check that no log statement or metric label references `email`; schema review. |
| I12 | **The first frame is the aviary.** No spinner component exists. The boot path draws birds from the inlined snapshot before the main bundle loads. | Bundle budget CI; synthetic time-to-first-bird measurement; no `Spinner` component. |
| I13 | **Accessibility ships in v1.** Reduced-motion register, narration, captions, keyboard navigation, and AA contrast are release-blocking. | Each is a milestone exit criterion (§18); axe + manual screen-reader runs in the release checklist. |
| I14 | **The engine is deterministic.** Same state + events + time + calibration → same outputs, on server and client, on host and visitor devices. | Engine package is pure (no I/O, no `Date.now`, no `Math.random`); replay tests; cross-device cue equality test. |

---

## 2. Scope

### 2.1 In v1

- Single-user accounts; email magic-link sign-in; per-device sessions with revocation; email change with verification; JSON export; soft-then-hard deletion.
- One aviary per account; two starter birds arriving at account creation; growth to a cap of seven birds on an age-based schedule; user-assigned, renameable names; six species.
- Server-side simulation: presence accounting, monotonic personality drift, attention accumulator, mood model, weather, bird-to-bird responses, cue planning, notebook observer.
- Client: single-screen scene with three perch zones, day/night anchored to the host's local time, ambient weather, ornaments, procedural idle motion, return-greeting, listen-in, offers (seed, song fragment, still pool), settle with 5-second undo, field notebook, thin fading top bar.
- Procedural audio: per-bird call signatures, call grammar, chorus mixing, listen-in mix ramps, procedural rain/wind beds, silence-with-captions fallback.
- Accessibility: naturalist screen-reader narration, call captions, keyboard navigation, focus treatment, reduced-motion register, AA contrast.
- Social: read-only visit invitations by email, revocation, expiry, visit log, opt-in visit notification email (off by default).
- Multi-device consistency as a property of the single-writer architecture.
- Operational telemetry, synthetic performance fleet, aggregate-only RUM.

### 2.2 Out of v1 (and what the codebase must therefore not contain)

- Native apps: no React Native/Capacitor shells; no API affordances designed for them.
- Gamification: no tables, columns, metrics, or copy for achievements, streaks, levels, scores, badges, counts of birds/visits/days, calendars.
- Tamagotchi mechanics: no hunger, health, decay, death, distress states; the engine has no negative drift path at all (not a disabled one — none).
- Social network: no profiles, follows, feeds, discovery, comments, chat, co-presence, avatars, leaderboards; no aggregate computed across aviaries for ranking.
- Push notifications, marketing email, re-engagement email of any kind. Transactional email only: magic link, invite, export link, email-change verification, optional visit notification.
- Payments, shared/multi-aviary accounts, customizable scenes, catalog-style bird picking, personality numbers in any product surface, WebGL renderer (deferred; see §8), recorded-audio fallback.

---

## 3. Architecture

### 3.1 Topology

```
 browser (host)          browser (visitor)
   │  HTML+inlined snapshot     │
   ▼                            ▼
 ┌──────────── edge (CDN + edge function) ────────────┐
 │  static immutable assets · HTML shell per request  │
 │  session cookie → in-region call to api/snapshot   │
 └───────────────┬────────────────────────────────────┘
                 ▼
 ┌──────────── api (stateless HTTP + SSE) ────────────┐
 │ auth · snapshot read · event append · notebook read │
 │ settings · visits · on-demand tick trigger          │
 └──────┬──────────────┬─────────────────┬────────────┘
        ▼              ▼                 ▼
   Postgres (sim)   Redis            sim-scheduler ──► sim-worker pool
   accounts,        leases, active    due-aviary        engine.tick()
   aviaries, birds, set, snapshot     queue, dormant    single writer
   events, notebook cache, pub/sub    cadence
        │
        ▼
   jobs: export · hard-delete · retention purge · mailer (transactional email provider)

   observability (metrics/traces/logs, allowlisted labels) ── NO path to Postgres(sim)
   synthetic browser fleet ── hits production like a user, staff accounts only
```

Services:

- **edge**: serves the HTML shell. For a signed-in request it calls `api` in-region with the session cookie, receives the snapshot (with greeting plan), and inlines it as `<script type="application/json" id="snap">`. HTML is `Cache-Control: private, no-store`; assets are content-hashed and immutable.
- **api**: TypeScript (Node 22, Fastify). Owns auth, reads, event append, SSE fan-out, visit tokens, settings. It never computes or writes simulation state; when responsiveness needs the engine (offer reaction, listen-in bias, greeting), it enqueues an **on-demand tick** and awaits its result (§6.2).
- **sim-scheduler**: enqueues aviaries when due (active cadence 60 s, dormant cadence 15 min). Single logical instance with leader election via Redis.
- **sim-worker**: pool of workers consuming the due queue; each runs `engine.tick` under a per-aviary lease and commits in one transaction.
- **jobs**: export generation, deletion, retention purge, invite expiry, visit-notification digests.
- **Postgres** (primary, one region at v1, synchronous replica for failover). **Redis** for leases, the active-aviary set, snapshot cache, pub/sub, rate limits. **Object storage** for export files (24-hour signed URLs).

### 3.2 The shared engine package

`packages/engine` is a pure TypeScript library used by `sim-worker`, `api` (via on-demand tick), and the client. It has no I/O, no wall clock, no global randomness; every function takes `now` and a seed.

```
tick(state, events, now, dt, calib) → { state', cues, notebookEntries, greetingRecord, metrics }
planGreeting(state, absenceSec, helloSeed)  → GreetingPlan            // server
planOfferReaction(state, offer, now, seed)   → { cues, moodPressure }  // server (inside on-demand tick)
expressionProfile(bird)                      → ExpressionProfile      // server only; produces snapshot-safe bands
callPhrase(signature, mood, intensity, seed) → Phrase                 // client audio, client captions, server salience
caption(phrase, birdContext)                 → string                 // client
narrate(snapshot, recentCues, sessionMemory) → string | null          // client
observe(prev, next, history, calib, seed)    → NotebookEntry[]        // server
prng(seed: string)                           → () => number           // sfc32 over a 128-bit hash of the seed
```

The client bundle includes only the client-safe subset (`callPhrase`, `caption`, `narrate`, interpolation helpers, species rigs). Trait→behavior mapping (`expressionProfile`, drift, mood) is excluded from the client build by package entry points, which is how I4 is kept structurally rather than by discipline.

### 3.3 Client/server split

| Concern | Server (tick) | Client |
|---|---|---|
| Personality, attention accumulator, drift | owns | never sees raw values |
| Mood transitions | owns | renders mood-shaped idle |
| Perch choice, flights, call schedule, responses, preen bouts | plans as **cues** with timestamps | executes cues on its clock, interpolates |
| Micro-motion (breathing, head scan, blink, shuffle) | — | generates deterministically from `(birdId, mood, tickNo)` |
| Ornaments (leaves, feathers, parallax) | — | client-only, not state |
| Weather | owns (canonical, same for all devices) | renders |
| Day/night | knows host timezone for mood | computes lighting from host timezone |
| Greeting | plans on `hello` | performs |
| Offer reaction | plans via on-demand tick | performs, shows item |
| Settle | receives event (mood-quieting) | device-local evening grade + quieting |
| Notebook | writes | reads, virtualized list |
| Narration, captions | — | generates from snapshot + cues + phrase |
| Presence | unions, clamps, credits | detects conjunction, emits intervals |

### 3.4 The render-pipeline boundary, precisely

The server output for a tick is a **snapshot** = canonical state (render-safe) + a **cue sheet** covering `[tickAt, tickAt + 120 s)`. The client is a performer of the cue sheet, not a simulator. Anything a second device or a visitor must see identically is a cue. Anything that is purely texture (micro-motion phase, leaves) is client-local and seeded so it looks like the same aviary without needing to match frame-for-frame.

### 3.5 Stack

- TypeScript everywhere (engine sharing is the reason). Node 22 + Fastify for `api`; worker processes for `sim-*`; Postgres 16; Redis 7; Vite for the client with manual chunking; Preact for chrome/settings/notebook (≈4 KB gz); the scene renderer is custom Canvas2D (no game engine, see §8).
- Playwright for synthetic checks and CI soak tests; Vitest for unit/property tests; fast-check for properties.
- Transactional email provider with webhook delivery status (magic links must arrive quickly; alarm on p95 delivery > 60 s).
- Deployed as containers behind a regional load balancer; one region for v1; edge functions at the CDN in every POP.

---

## 4. Data model

All ids are ULIDs except `accounts.id` (UUIDv4, the synthetic account id). Times are `timestamptz`. Personality traits are `double precision` in `[0,1]`.

### 4.1 Identity and accounts

```sql
accounts (
  id                 uuid PK,                 -- the only identifier used anywhere else
  email_enc          bytea NOT NULL,          -- AES-GCM, key in KMS; decrypted only by auth + mailer
  email_bidx         bytea UNIQUE NOT NULL,   -- HMAC-SHA256(normalized email, server key): sign-in lookup only
  created_at, timezone text,                  -- last-seen IANA tz from the host's device
  settings           jsonb,                   -- captions, reducedMotion(override), audio, narrationVisible, visitNotifications
  deletion_requested_at timestamptz NULL,
  pending_email_enc  bytea NULL, pending_email_bidx bytea NULL, pending_email_token_hash bytea NULL
)
sessions (id ulid PK, account_id uuid FK, token_hash bytea UNIQUE, device_label text, ua_family text,
          created_at, last_seen_at, revoked_at NULL)
magic_links (token_hash bytea PK, email_bidx bytea, account_id uuid NULL, created_at, expires_at, consumed_at NULL, ip_hash bytea)
```

The blind index is an interpretation of "email stored once, encrypted": a sign-in by email needs a lookup key, and an HMAC with a server-held key is not the email, is not reversible, and is never used as an identifier outside the auth module (decision D12).

### 4.2 Aviary and birds

```sql
aviaries (
  id ulid PK, account_id uuid UNIQUE FK, created_at,          -- age = now - created_at
  state_version bigint NOT NULL DEFAULT 0,                    -- monotonic; snapshot version
  last_tick_at timestamptz, calibration_version text,
  weather jsonb,          -- {kind:'clear'|'rain'|'wind', started_at, ends_at, intensity, next_roll_at}
  rng_state jsonb,        -- engine PRNG state for the aviary
  history jsonb,          -- rolling 14-day observation facts for the notebook (per-bird greeting order per day,
                          -- front-perch minutes per bird per day, chorus events, rain days, quiet spells)
  last_snapshot jsonb,    -- the last emitted snapshot (render-safe), for fast reads and edge inlining
  last_presence_end timestamptz,                              -- for absence-length on hello
  paused boolean NOT NULL DEFAULT false                       -- soft-deleted aviaries do not tick
)
birds (
  id ulid PK, aviary_id FK, species text, name text, arrived_at, named_at NULL,
  boldness, warmth, vocal, plumage, curiosity double precision,   -- canonical personality vector
  attention double precision,                                      -- attention accumulator A ∈ [0,1]
  signature_seed text,           -- fixed at arrival; derives call signature (§6.14)
  mood text, mood_since timestamptz, mood_pressure jsonb,           -- integrators
  perch jsonb,                   -- {zone:'front'|'middle'|'back', slot:int}
  last_offer_reaction_at timestamptz NULL,
  daily_delta jsonb              -- per-trait drift applied today (for the per-day cap), keyed by local date
)
```

Traits are columns (not a JSON blob) so the per-day cap and monotonicity can be asserted with a `CHECK (boldness BETWEEN 0 AND 1)` and a trigger that rejects any decrease.

### 4.3 Events (append-only, short-lived)

```sql
interaction_events (
  id ulid PK,                       -- client-generated; idempotency key
  aviary_id FK, session_id FK,      -- host session only; visitor sessions cannot insert
  type text,                        -- 'hello' | 'presence' | 'listenin.start' | 'listenin.end' | 'offer' | 'settle' | 'settle.undo' | 'bye'
  payload jsonb,                    -- see §5.4
  client_at timestamptz, server_at timestamptz DEFAULT now(),
  processed_version bigint NULL     -- set by the tick that consumed it
) PARTITION BY RANGE (server_at)    -- daily partitions; dropped by the retention job 7 days after processing
```

Presence pings are stored as intervals (`{from,to}`), not instants, and are purged with the same 7-day retention. Nothing in this table is ever read by anything but the tick and the retention job (I10).

### 4.4 Notebook, growth, visits

```sql
notebook_entries (id ulid PK, aviary_id FK, at timestamptz, local_date date, text text, salience real, kind text)
bird_arrivals    (id ulid PK, aviary_id FK, bird_id FK, due_at timestamptz, arrived_at NULL)  -- age schedule materialized at creation
visit_invites    (id ulid PK, aviary_id FK, visitor_email_enc bytea, token_hash bytea UNIQUE, created_at,
                  expires_at, consumed_at NULL, revoked_at NULL)
visit_sessions   (id ulid PK, invite_id FK, token_hash bytea UNIQUE, started_at, last_seen_at, ended_at NULL)
exports          (id ulid PK, account_id FK, requested_at, object_key text, expires_at)
```

Sizes: a snapshot is 3–8 KB; an aviary row with `last_snapshot` ≈ 10 KB; events for an active user ≈ 200/day; notebook ≈ 150 entries/year. Postgres is comfortable to low millions of accounts on one primary.

---

## 5. API surface

### 5.1 Conventions

- JSON over HTTPS; session cookie `pa_s` (HttpOnly, Secure, SameSite=Lax); `Origin` check on mutations; visitor cookie `pa_v` in a separate namespace with read-only scope.
- Every response carries `x-aviary-version` when aviary-scoped. Clients ignore any snapshot whose version is lower than the one they hold.
- Errors return `{ code, message }` where `message` is registry copy in the system voice. No naturalist strings in error paths.

### 5.2 Auth and account

| Method & path | Behavior |
|---|---|
| `POST /auth/magic-link {email}` | Always `202` (no enumeration). Creates a 15-minute single-use token; rate limit default 5/email/hour and 20/IP/hour. |
| `GET /auth/consume?t=` | Consumes the token (atomic `UPDATE … WHERE consumed_at IS NULL`), creates an account if new (and its aviary + two starters, §6.15), issues a session, redirects to `/`. Failure renders the system-voice page: "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| `GET /account/sessions` / `DELETE /account/sessions/:id` | List and revoke devices. Revoked sessions fail on next request with "Your session timed out. Sign in again to keep watching." |
| `POST /account/email-change {newEmail}` → `POST /account/email-change/verify {token}` | Old email keeps working until verification commits the switch. |
| `PATCH /account/settings` | Cross-device settings (captions, audio, narration visible, reduced-motion override, visit notifications). |
| `POST /account/export` | `202`; the job emails a 24-hour signed link. |
| `POST /account/delete` / `POST /account/delete/cancel` | Soft delete (sets `deletion_requested_at`, pauses the aviary); cancel within 30 days restores and triggers catch-up ticks. |
| `GET /account/privacy` | Plain-text policy page listing aggregate categories and excluding per-bird state. |

### 5.3 Aviary state

`GET /aviary/snapshot?hello=1&tz=Europe/Lisbon&hiddenFor=1830`

- Ensures freshness (§6.17), then returns the snapshot. With `hello=1` it also logs a `hello` event and includes a `greeting` plan computed from absence length (`now − last_presence_end`).
- Supports `If-None-Match` with the version as ETag → `304`.

```jsonc
{
  "version": 18234, "tickAt": "2026-09-06T14:03:00Z", "serverNow": "2026-09-06T14:03:21Z", "nextTickAt": "…",
  "aviary": { "id": "01J…", "timezone": "Europe/Lisbon", "ageDays": 212,
              "weather": { "kind": "rain", "startedAt": "…", "endsAt": "…", "intensity": 0.4 } },
  "birds": [ {
      "id": "01J…", "name": "pip", "species": "grey_finch", "descriptor": "small grey bird",
      "mood": "content", "moodSince": "…",
      "perch": { "zone": "front", "slot": 1 },
      "expression": { "paletteStep": 6, "idleProfile": "content", "approachBias": "near", "sizeVariant": 2 },
      "signatureSeed": "s_9f3c…",         // derives synthesis params + caption grammar; not a personality value
      "familiarity": "familiar"           // 'quiet' | 'familiar' | 'close' — coarse band of the attention accumulator
  } ],
  "cues": [
      { "id": "c_…", "t": 12.4, "bird": "01J…", "kind": "call", "phraseSeed": "p_…", "intensity": 0.6 },
      { "id": "c_…", "t": 13.1, "bird": "01J…", "kind": "call", "phraseSeed": "p_…", "inReplyTo": "c_…" },
      { "id": "c_…", "t": 40.2, "bird": "01J…", "kind": "move", "to": { "zone": "middle", "slot": 0 }, "dur": 1.6 },
      { "id": "c_…", "t": 55.0, "bird": "01J…", "kind": "preen", "dur": 9.0 },
      { "id": "c_…", "t": 71.5, "bird": "01J…", "kind": "lookToward", "target": "01J…" }
  ],
  "greeting": { "primary": { "bird": "01J…", "form": "approach_call", "t": 0.9, "phraseSeed": "p_…" },
                "secondary": [ { "bird": "01J…", "form": "glance", "t": 2.4 } ] },
  "offer": { "active": null },           // or { kind:'seed', spot:'front_left', placedAt, expiresAt }
  "newcomer": null                       // or { birdId, since } for an unnamed arrival
}
```

`paletteStep` is a 0–15 quantization of rendered plumage richness; `expression` values are categorical. This is what "the user never sees the numbers" means at the wire level.

`GET /aviary/stream` (SSE): emits `version` events when a tick commits (Redis pub/sub → api). Clients fetch the snapshot on receipt. Heartbeat every 25 s. If SSE is unavailable (some proxies), the client polls at `nextTickAt + jitter(0–5 s)`.

### 5.4 Events

`POST /aviary/events { events: [...] }` → `{ accepted: [ids], rejected: [{id, code}], reactions: [cues…] }`

Batched, idempotent by `id`. Server stamps `server_at`; `client_at` is advisory only. Payloads:

- `hello { tz, device: {ua_family, viewport, reducedMotion, audio: 'on'|'off'|'blocked'}, hiddenFor }`
- `presence { from, to }` — an interval ≤ 30 s during which the conjunction held continuously; sent every 20 s and on visibility loss.
- `listenin.start { bird }`, `listenin.end { bird }`
- `offer { kind: 'seed'|'song'|'pool', spot?: 'front_left'|'front_center'|'front_right', fragment?: 1..6 }` — triggers an on-demand tick; the response carries reaction cues.
- `settle {}`, `settle.undo {}`
- `bye { reason: 'pagehide'|'settle' }` — best-effort via `navigator.sendBeacon`.

Server-side validation: presence intervals are clipped to `[last accepted to, now]`, capped at 30 s each, and rejected if from the future by > 5 s; listen-in without overlapping presence earns no credit (§6.3); offers are rejected with `offer_active` while one is active.

### 5.5 Notebook, birds, growth

- `GET /aviary/notebook?before=<cursor>&limit=30` → entries newest-first with a cursor; entries are immutable.
- `PATCH /aviary/birds/:id { name }` — rename (1–24 chars, any script). Also used to name a newcomer (sets `named_at`).
- `GET /aviary/birds` — ids, names, species descriptors, arrival dates. No traits.

### 5.6 Visits

Host:

- `POST /visits/invites { email }` → `201`, emails a one-time link; 30-day expiry; default rate limit 10 outstanding invites per aviary.
- `GET /visits/invites` — outstanding invitations. `DELETE /visits/invites/:id` — revoke (immediately adds token hashes to a Redis revocation set and marks the DB row).
- `GET /visits/log` — visits newest-first: visitor email, date, approximate duration.

Visitor (no account):

- `GET /visit/:token` — consumes the invite on first use, sets `pa_v` (30 days from consumption), serves the visitor HTML with an inlined visitor snapshot.
- `GET /visit/snapshot` — same shape as the host snapshot minus `greeting`, `offer` placement controls, `newcomer` naming, and without names of anything but birds. Every pull checks revocation; revoked/expired → `410` and the system-voice page: "This visit is no longer available."
- No events endpoint accepts `pa_v`. Visitor pulls mark the aviary active (so cues are planned) but write nothing to `interaction_events`.

---

## 6. Simulation engine design

### 6.1 Time and determinism

- The engine integrates in continuous time: every rate constant is "per day" or "per hour," and `tick` takes `dt` explicitly. Active aviaries tick with `dt ≈ 60 s`; dormant ones with `dt = 15 min` (§6.17). Results must not depend on `dt` beyond integration error (tested: 15 one-minute ticks ≈ one fifteen-minute tick within tolerance).
- Randomness comes from one `sfc32` stream per aviary (state persisted in `aviaries.rng_state`) plus derived per-purpose seeds: `seed(aviaryId, version, birdId, purpose, n)`. Cues, phrases, greetings, and notebook variants are reproducible from their seeds; the client re-derives phrase details from `phraseSeed` and the visitor gets the same result.
- Local time uses the host's stored IANA timezone. Dawn/dusk are a per-month table (default dawn 06:30 / dusk 19:30 ± seasonal offset up to 1 h) — no geolocation. Visitors' lighting follows the host's timezone (D18).

### 6.2 The tick pipeline (one aviary, one transaction)

1. Acquire a Redis lease `sim:lease:{aviaryId}` (SET NX PX 30000, fenced token; refreshed every 10 s for long catch-ups).
2. Load aviary, birds, and unprocessed events ordered by `(server_at, id)`. Clients' clocks are never trusted for ordering.
3. **Normalize events**: drop duplicates and malformed payloads; clip presence intervals into `[last_tick_at, now]`; union presence across all sessions of the account into one interval set; intersect listen-in spans with the presence set; validate offers against the active-offer state.
4. **Advance environment**: weather state machine; local time-of-day phase.
5. **Attention accumulators** per bird (§6.4) from credited presence.
6. **Drift** (§6.5): compute non-negative deltas, apply per-day caps, add to traits; assert monotonicity.
7. **Mood** (§6.7): integrate pressures, apply transitions with hysteresis and dwell.
8. **Growth**: fire any due `bird_arrivals` (§6.15).
9. **Cue planning** (§6.9) for `[now, now + 120 s)` — skipped for dormant ticks.
10. **Observer** (§6.16): notebook candidates → sparsity budget → entries.
11. Commit in one transaction with `WHERE state_version = expected` (optimistic check in addition to the lease); set `processed_version` on consumed events; write `last_snapshot`; bump `state_version`.
12. Publish `{aviaryId, version}`; store the snapshot in Redis (`snap:{aviaryId}`, TTL 5 min); record `tick_duration_ms`, `events_consumed` (no account labels).

**On-demand ticks** use the same function with the same guarantees. The API enqueues `{aviaryId, reason:'offer'|'listenin'|'hello'|'ensureFresh'}` on a priority queue and awaits the resulting version (timeout 1.5 s, then the client proceeds with the next scheduled snapshot). Debounce: at most one on-demand tick per aviary per 2 s; subsequent triggers coalesce. Because a tick is the only writer, responsiveness never creates a second writer.

### 6.3 Presence accounting

Client side (`presence.ts`, a pure state machine):

- Inputs: `visibilitychange`, `focus`/`blur` on `window`, and an activity heartbeat set by `pointermove`, `pointerdown`, `keydown`, `wheel`, `touchstart`, and `focusin` (the last one so keyboard and assistive-technology users who navigate without pointer or key repeat still register activity).
- State `present` is true iff `visible && focused && (now − lastActivity) ≤ ACTIVITY_WINDOW`. Default `ACTIVITY_WINDOW = 5 min` (calibration range 3–8 min; lean long, per the PRD).
- While `present`, the client emits `presence {from, to}` every 20 s (intervals of ≤ 30 s including flush jitter), and one final interval on any transition to not-present, on `pagehide`, and on settle.
- The tracker never counts a visitor session; visitor builds do not include the presence module.

Server side (in the tick):

- Intervals are unioned across sessions, so two devices open at once credit at most wall-clock time (D9). Credited presence per tick ≤ `dt`.
- Listen-in credit for bird *b* = duration of `[start, end] ∩ presence`. Listen-in that runs while the tab is hidden earns nothing; the client also auto-ends listen-in after 60 s hidden.
- `last_presence_end` is updated to the end of the latest credited interval; absence length for greetings is measured from it.
- Anti-inflation guards (calibration protection, not security): per-session ping cadence ≥ 15 s; intervals with `to > now + 5 s` rejected; daily presence credit used for the attention accumulator is capped (§6.4).

### 6.4 Attention accumulator (the "ambient quietness" mechanism)

Each bird has `A ∈ [0,1]`, the low-pass-filtered presence signal. It is engine state, not a trait: it decays when the user is away, and it is what makes an ignored aviary quieter without ever lowering a trait.

Per tick, with `dt` in days:

```
A ← A · exp(−dt / τ_A)                              τ_A = 12 days (default)
c ← min(P_tick, max(0, DAILY_CAP − credited_today)) / DAILY_CAP    DAILY_CAP = 1800 s (30 min/day)
A ← A + (1 − A) · g · c                              g = 0.20 (default)
```

Behavior at defaults: 15 min/day → equilibrium A ≈ 0.55 after ~3 weeks; five days away → ≈0.37; two weeks away → ≈0.17; three regular days after that → back above 0.35. `familiarity` bands in the snapshot: `quiet` < 0.2, `familiar` 0.2–0.5, `close` > 0.5. Bands affect greeting readiness, response probability, and how far forward birds tend to sit; they never affect traits.

### 6.5 Drift function and calibration

Traits `T ∈ {boldness B, warmth W, vocal V, plumage P, curiosity C}`, each in `[0,1]`. Seeds by species (baseline ± uniform jitter), all in the lower-middle of the range so decades of monotonic drift have headroom:

| species (working names) | B | W | V | P | C | night-active |
|---|---|---|---|---|---|---|
| grey finch ("small grey bird") | 0.38±0.06 | 0.40±0.06 | 0.35±0.06 | 0.30±0.05 | 0.40±0.06 | no |
| warbler ("yellow-green warbler") | 0.32±0.06 | 0.35±0.06 | 0.45±0.06 | 0.36±0.05 | 0.42±0.06 | no |
| wren ("small brown wren") | 0.42±0.06 | 0.38±0.06 | 0.48±0.06 | 0.28±0.05 | 0.45±0.06 | no |
| thrush ("speckled thrush") | 0.30±0.06 | 0.42±0.06 | 0.40±0.06 | 0.34±0.05 | 0.36±0.06 | no |
| blue tit ("blue-grey tit") | 0.45±0.06 | 0.45±0.06 | 0.42±0.06 | 0.38±0.05 | 0.50±0.06 | no |
| nightjar ("dusky nightjar") | 0.25±0.05 | 0.30±0.06 | 0.35±0.06 | 0.26±0.05 | 0.32±0.06 | yes |

Two drift channels, both additive, both ≥ 0:

**Presence channel (continuous, driven by A, applies to all traits).** This is the low-pass filter: drift proceeds at a rate proportional to the accumulated attention, including during absence, so "personality drifts during the user's absence based on inputs from before they left."

```
ΔT = κ_T · A · (1 − T)² · dt_days
κ_B = κ_W = κ_V = 0.023 /day     κ_P = 0.028 /day     κ_C = 0.018 /day
```

**Interaction channel (impulses, presence-clamped).**

```
listen-in on b, per credited minute:   ΔW_b += 0.0015 (1−W)²    ΔV_b += 0.0010 (1−V)²
offer accepted by b (take/drink/bathe/join):  ΔC_b += 0.0040 (1−C)²   (≤ 1 credit / bird / 5 min)
offer near b (approach or watch from an adjacent zone):  ΔB_b += 0.0015 (1−B)²
settle:  no drift; mood pressure only
```

**Caps.** Per trait per bird per local day: total Δ ≤ 0.008; interaction-channel Δ ≤ 0.004. Caps are what make "a single session never moves a personality value visibly" a guarantee rather than a tendency.

**Why (1−T)².** Linear headroom would saturate a regular user's bird within a year. Quadratic headroom gives a long tail: at A = 0.55 a trait seeded at 0.35 reaches ≈0.84 after one year and ≈0.91 after two, still moving.

**Calibration targets** (asserted by the harness in §14.3 for the "regular visitor" persona: 12 min/day, 6 days/week, one 3-minute listen-in most days):

| checkpoint | expected trait change (presence channel, B/W/V) | criterion |
|---|---|---|
| day 7 | ≈ 0.012–0.016 | ≥ 0.010 (measurable: 3× the float-storage and daily-cap granularity) |
| day 21 | ≈ 0.07–0.09 | ≥ 0.06 and the expression mapping (§6.6) shows a retrospectively visible change |
| any single day | ≤ 0.008 | no single-session visibility |
| day 365 | trait ≈ 0.80–0.86 | still below 0.9 (headroom remains) |
| absent 14 days after day 21 | traits unchanged; A from ≈0.5 to ≈0.16 | monotonicity + quieting |

Constants live in `calibration/v1.json` with a `calibration_version` stamped on each tick; changes to constants affect only future deltas (drift is additive), which keeps re-calibration safe for existing aviaries.

### 6.6 Expression mapping (trait → observable behavior)

The server maps traits and `A` to behavior when it plans cues and derives the snapshot's `expression` bands. Design rule, tested in the harness by measuring the planned cues: **a 0.08 trait change must be noticeable in retrospect (≥ 8 percentage points in a primary behavior share, ≥ 10 % in call rate, or ≥ 1 palette step) and a 0.008 change must not be.**

| behavior | mapping (defaults) |
|---|---|
| perch zone choice (on dwell expiry) | softmax over `front: 5(B−0.5) + m_front(mood) + 0.4·[A>0.5]`, `middle: 0`, `back: 5(0.5−B) + m_back(mood)`; `m_front`: curious +0.4, wary −1.0, drowsy −0.3; `m_back`: wary +1.2, drowsy +0.4. Dwell median 10 min (curious 6, drowsy 25, roosting ∞). At B = 0.35 → ≈16 % front time; at 0.43 → ≈25 %. |
| call rate λ (per min) | `species_base(0.5–0.9) · (0.4 + 1.2 V) · mood_f · weather_f · tod_f`; `mood_f`: alert 1.2, curious 1.0, content 1.0, wary 0.4, drowsy 0.3, roosting 0.05 (nightjar inverts day/night); rain 0.5; night 0.1 except nightjar. |
| response to another bird's call | `p = min(0.8, 0.15 + 0.5 W) · mood_f · familiarity_f(quiet 0.7, familiar 1.0, close 1.15)` |
| greeting weight | `exp(2.5 B + 2.0 W + mood_bonus) · familiarity_f(quiet 0.6, familiar 1.0, close 1.2)`; mood_bonus: alert +0.5, curious +0.3, content 0, drowsy −0.5, wary −0.8, roosting −3 |
| chance the warier bird greets at all today | secondary greeting `p = 0.35 W · mood_f` per non-primary bird |
| offer approach | `sigmoid(3(C−0.5) + 2(B−0.5) + proximity + m_offer(mood))`; m_offer: curious +1.0, content +0.5, alert +0.2, wary −1.2, drowsy −1.5, roosting −∞; wary birds that "approach" do so after 20–60 s and may only watch |
| head-tilt toward sounds | `lookToward` cue probability on others' calls: `0.25 + 0.5 C` |
| plumage | `paletteStep = round(16 · P)`; each step is designed as a just-noticeable increase in chroma/feather detail (design system defines the 16-step ramp per species) |
| size/posture | `sizeVariant` from the signature seed (fixed), `idleProfile` = mood |

### 6.7 Mood model

States: `wary, alert, curious, content, drowsy, roosting`. `roosting` is the night state the PRD calls "settled/sleeping" for birds; it is named differently in code to avoid colliding with the user's settle gesture (D5).

Each bird integrates a pressure vector `p` (leaky integrator, `τ_p = 45 min`): `p ← p · (1 − dt/τ_p) + inputs · dt`, plus impulses. Inputs per hour:

- **time of day** (host tz): `[dawn − 0.5 h, dawn + 2.5 h]` alert +1.0, curious +0.6; midday content +0.6; late afternoon content +0.3, curious +0.3; `[dusk − 1 h, dusk + 1 h]` drowsy +1.0; night roosting +2.0. Nightjar: night alert +1.0 and curious +0.4, midday roosting +1.0.
- **weather**: rain content +0.3 (sheltering) and vocal ×0.5 (expression only); wind alert `+0.8 C`, wary `+0.8 (1 − B)`.
- **interactions**: offer accepted → content impulse +1.5; offer watched → curious impulse +0.6; listen-in active → alert +0.4/h, curious +0.4/h; settle → drowsy impulse +1.0; performing a greeting → alert impulse +0.5.
- **contagion**: each other bird in `wary` → wary `+0.6 (1 − B)` per hour; a chorus event → content impulse +0.3 for participants.
- **gating by personality**: wary pressure × `(1.3 − B)`, curious × `(0.7 + C)`, alert × `(0.7 + 0.6 V)`.

Transition rule: `candidate = argmax p`; switch iff `p[candidate] − p[current] ≥ 0.25` and dwell ≥ 10 min (leaving `roosting` needs ≥ 30 min and daylight or an impulse ≥ 1.0). Seeded noise ±0.1 breaks ties so birds are not synchronized. **Daily-ish reset** is implemented as a dawn damping `p ← 0.3 p` so the day's mood re-derives from the day's context without ever snapping the visible state; mood persists across sessions because it is simply state that continues to tick.

### 6.8 Weather

Per aviary, a state machine rolled at most once per 10 minutes: rain with a mean of 2.5 events/week (duration 3–8 min, intensity 0.3–0.7), wind 3 events/week (2–5 min). Never both at once; never at night beyond a soft wind. Weather is canonical so all devices and visitors see the same rain; it feeds mood pressures and call-rate factors as above and is recorded in `history` for the notebook.

### 6.9 Cue planning (active ticks only)

For the horizon `[now, now + 120 s)`:

- **Calls**: per bird, Poisson arrivals at rate λ (§6.6), each with `phraseSeed`, `intensity` from mood (wary 0.4, drowsy 0.35, content 0.6, curious 0.7, alert 0.8), and a listen-in bias (×1.6 rate, +0.1 intensity) while that bird is being listened in on.
- **Responses**: for each planned call at `t`, roll each other non-roosting bird with the response probability; a response is a `call` cue at `t + U(0.4, 2.5) s` with `inReplyTo`; chains stop at depth 2. If ≥ 3 birds call inside any 6-second window the tick records a chorus event (notebook candidate) and adds the content impulse.
- **Moves**: when a bird's dwell timer expires, sample a zone (§6.6) and a free slot (capacity 3 per zone, 9 slots for ≤ 7 birds; the sampler never lets two birds target one slot); emit `move` with duration 1.2–2.0 s by distance. Roosting birds do not move.
- **Bouts**: `preen` (content/drowsy: 0.3/min, 6–15 s), `scan` (wary/alert: 0.6/min, 3–6 s), `lookToward` on others' calls, `fluff` on cold-hour mornings and rain.
- Cue ids are `hash(aviaryId, version, birdId, seq)`; the horizon overlaps the next tick by 60 s so a late snapshot never leaves the client without cues.

### 6.10 Return-greeting

Computed on `hello` (`GET /aviary/snapshot?hello=1`, or the edge-inlined equivalent), seeded by the hello event id, and re-derived identically by the next tick to record it in `history`.

- **Absence band** from `now − last_presence_end`: `< 10 min` → *glance* (look up from current activity; no call); `10 min – 6 h` → *short* (two-note call or head-tilt plus one step forward); `6 h – 3 d` → *approach* (move toward the front zone if B > 0.4, then a call); `> 3 d` → *reorient* (longer call, a secondary response scheduled from another bird, a move to the front for bold birds).
- **Primary greeter**: sampled by greeting weight (§6.6); exactly one. Starts at `t = U(0.6, 1.6) s` after first frame.
- **Secondary**: each other bird independently with `p = 0.35 W · mood_f`, staggered `t + U(0.8, 3.0) s`, forms *glance* or *short*. Never simultaneous.
- **Night**: if all birds are roosting, the nightjar (if present) greets; otherwise the boldest bird *stirs* (eye opens, feathers shift) — still a notice, never nothing.
- Return from a hidden tab counts as a hello only if hidden ≥ 20 s (a five-second alt-tab does not re-greet).

### 6.11 Offers

- Initiated from the top bar. `seed` requires a placement spot (`front_left | front_center | front_right`, chosen by tapping the ground/rail area or by arrow keys in the sheet); `pool` always lands `front_center`; `song` has no location (fragments 1–6 are short motif sequences played by the client's synth, not recordings).
- One active offer per aviary; lifetime seed 4 min or until taken, pool 5 min, song 30 s of playback plus a 2-minute response window. While active, the sheet shows the item quietly ("a seed is out on the rail") and refuses a new one; no timer, no countdown.
- The on-demand tick plans per-bird reactions (approach delay, outcome: take / drink / bathe / join / go quiet / call against / watch / ignore) and returns them as cues in the events response (target ≤ 300 ms end-to-end). Credits (§6.5) are applied by that same tick.
- Per-bird reaction credit is limited to one per 5 minutes; this, plus the single-active-offer rule, is the "per-bird cooldown of a few minutes."

### 6.12 Settle

Server: the `settle` event ends the presence window, ends any listen-in, and applies the drowsy impulse. Nothing else changes canonically; the settled evening lighting is device-local (D2). `settle.undo` within 5 s cancels the drowsy impulse if the tick has not yet run (the API drops both events if they pair within the window) and the client reverses the grade.

### 6.13 Bird-to-bird interaction

Covered by responses, contagion, chorus, and `lookToward` cues above. Additionally, high-warmth birds prefer slots adjacent to other birds (slot sampler bias +0.5 for adjacency when `W > 0.5`), which makes warmth visible as birds perching near one another.

### 6.14 Call signatures and the call-grammar runtime

**Signature** (fixed for life, derived from `signature_seed`):

- base pitch offset within the species band (±3 semitones), tempo (syllable rate ±20 % of species), timbre (harmonic tilt, noise mix, formant center), a **tag motif** (one 2–4-syllable figure unique to the bird that appears in ≥ 70 % of its phrases), motif preference weights.
- When a bird arrives, its signature is chosen by **farthest-point selection**: 64 candidate signatures are sampled from the species distribution and the one maximizing the minimum perceptual distance (weighted: pitch 0.35, tempo 0.2, timbre 0.25, tag contour 0.2) from the existing birds' signatures is kept. This is the engineering mechanism behind "know Pip from Wren by ear" at seven birds.

**Grammar** (per species, 4–6 motifs such as `rise`, `fall`, `trill`, `chip`, `double`, `whistle`, plus the nightjar's `churr`): a phrase is 1–5 motifs sampled from a first-order Markov chain over motifs with signature-weighted transitions. Each motif carries parameter distributions (syllable count, interval, glide, duration) sampled from `phraseSeed`.

**Mood/trait modulation stays inside recognizability bounds**: pitch ±2 semitones, tempo ±20 %, intensity −12 dB..0 dB, phrase length by mood (drowsy 1–2 motifs, alert 3–5). Timbre, tag motif, and base pitch never modulate.

`callPhrase(signature, mood, intensity, seed)` is in the shared engine, so the client's synth, the client's captions, and the server's salience scoring all agree on what was sung.

### 6.15 Arrival: starters and growth by age

- **Account creation**: the consume-link handler creates the aviary and two starters of different species (random from the pool, nightjar excluded from starters so the first encounter is daytime-active), each with a signature chosen by farthest-point against the other. The adoption screen (§8.12) names them; naming does not alter identity.
- **Growth schedule** (default, config): arrivals due at aviary age 75, 150, 240, 365, 540 days for birds 3–7, materialized as `bird_arrivals` rows at creation so the schedule is auditable and shiftable. On the due tick, the newcomer arrives on the back perch with a fly-in cue, slightly lower boldness seed (−0.05), an unused species when possible (the seventh repeats one), and a farthest-point signature. It calls, moods, and drifts from arrival. The notebook records it (bypass salience). The snapshot carries `newcomer` until it is named from the birds panel; the newcomer never expires and cannot be declined (a bird that arrived is part of the aviary). Nothing counts visits or interactions toward arrivals (D7).

### 6.16 The notebook observer

Runs at the end of each tick (active and dormant) over `history`:

| candidate | salience |
|---|---|
| a bird greeted first for the first time this week | 0.7 |
| a bird's first time on the front perch | 0.8 |
| newcomer arrival | 1.0 (bypasses budget) |
| chorus of ≥ 3 | 0.5 |
| rain (first in 14 days → 0.6) | 0.4 |
| a long quiet spell (no calls for > 90 min in daylight) | 0.4 |
| a seed/pool/song outcome, told from the birds' side | 0.5 |
| a bird on one perch all day; a preen during a greeting; nightjar calling late; a roost change | 0.3 |

**Sparsity budget**: token bucket capacity 2, refill one token per 72 h; emit when salience ≥ 0.5 and a token is available, or salience ≥ 0.9 (bypass). Minimum 20 h between entries except bypass. Yields roughly one entry every 2–4 days for a regular user and fewer during absence ("a long stretch of quiet this morning" still appears, because the aviary keeps ticking).

**Rendering**: each candidate has 6–10 template variants with slot grammar (bird name, perch phrase, time-of-day phrase, weather clause, an optional ambient detail such as "a leaf drifted down past the back perch and neither bird looked up"), chosen by seed. A weekday header ("tuesday —") is prepended when the local date differs from the previous entry. Templates are authored by the writer and linted (§11): lowercase, present tense, no second person, no counts of the user's behavior, no exclamation. No LLM is involved (D16): generation is deterministic, reviewable, and keeps per-bird state inside the simulation boundary.

### 6.17 Dormant cadence and catch-up

- `active:{aviaryId}` in Redis (TTL 10 min) is refreshed by any host or visitor snapshot pull or event post. Active aviaries tick every 60 s with cue planning; all other non-paused aviaries tick every 15 min with `dt = 15 min` and no cues. The aviary therefore always advances on the server whether or not a client is connected; only the wall-clock granularity changes when nobody is watching (D3).
- `ensureFresh(aviaryId)` on every snapshot read: if `last_tick_at < now − 90 s`, run an on-demand tick first (it integrates the elapsed `dt`; if `dt > 2 h` — outages, restored accounts — it steps in 1-hour chunks then a final short chunk). Under normal operation this path adds < 40 ms.
- Capacity: 100 k accounts with 5 % active → ≈ 83 active ticks/s plus ≈ 105 dormant ticks/s; at ≈ 5 ms CPU each this is a handful of workers. Backlog and lease-wait are the metrics to watch (§13).

### 6.18 Engine invariant tests

- Monotonic, capped drift under random event streams (fast-check, 10 k cases).
- `dt`-independence within tolerance (1×15 min vs 15×1 min).
- Identity: replaying an aviary through every migration leaves `bird.id` and traits unchanged.
- Determinism: same inputs produce byte-identical snapshots; host and visitor derive identical phrases and captions from a cue.
- Presence: intervals from two sessions overlapping fully credit once; listen-in outside presence credits zero; a hidden tab credits zero.
- Greeting: exactly one primary greeter; never two cues at the same instant; night produces a stir.
- Growth: arrivals depend only on age; a fixture with 10 000 interactions and age 10 days has two birds.
- Notebook: the budget never emits more than 2 entries in 72 h without a bypass; template lint passes.

---

## 7. Sync model

### 7.1 One canonical state

`aviaries.state_version` is the sync primitive. Every snapshot is a projection of one committed version; every client holds exactly one snapshot version at a time and replaces it wholesale. There is no client-side merge because there is nothing to merge: clients hold render state and an outbox of events, never canonical state.

### 7.2 Snapshot delivery

1. **Inline** in the HTML on navigation (edge → api, `hello=1`).
2. **SSE** `/aviary/stream` for `version` notifications while the tab is visible; on notification the client fetches with `If-None-Match`.
3. **Pull triggers** independent of SSE: `visibilitychange → visible` (hello if hidden ≥ 20 s), a render-frame gap > 5 s (laptop resumed), a keepalive every 60 s aligned to `nextTickAt + jitter(0–5 s)` when SSE is not connected, and after any event post that returns a newer `x-aviary-version`.
4. **Clock mapping**: each response's `serverNow` is paired with `performance.now()` at receipt; the offset is smoothed (EMA, α = 0.3) and used to place cue times on the local timeline. A jump > 2 s resets the filter.

### 7.3 Client cue reconciliation

- Cues are keyed by id. On a new snapshot: cues already started continue to completion; not-yet-started cues from the old sheet are dropped; the new sheet's cues are scheduled. Because the server re-plans the overlapping 60 s deterministically from the same state, most overlapping cues are identical and the swap is invisible.
- Positions: the client interpolates from the last known pose to the snapshot pose only if they differ by more than a slot (a resumed laptop); otherwise it trusts the in-progress animation. A large discrepancy (bird in another zone) is resolved with a normal flight cue, never a teleport.
- Mood changes arrive as state; the idle profile cross-fades over 3–5 s.

### 7.4 Event submission

- Outbox in memory, bounded at 200 events (oldest presence intervals are coalesced first). Flush every 20 s, immediately for `hello`, `offer`, `listenin.*`, `settle*`, and on `pagehide` via `sendBeacon`.
- Ids are ULIDs generated client-side; the server's idempotent insert makes retries safe. Exponential backoff 1 s → 30 s; after 5 minutes offline the outbox drops presence intervals (they are not replayable honestly) but keeps the last `listenin.end`.
- Ordering is by `server_at`; a late-arriving `listenin.start` after its `end` is discarded by the normalizer.

### 7.5 Multi-device semantics

| situation | behavior |
|---|---|
| laptop and phone both open | presence unions to wall-clock; both render the same version; both hear the same cues |
| listen-in on both devices to different birds | both credited (each ∩ presence); cues bias both birds |
| offer from phone while laptop watches | one active offer; laptop sees the item and reactions on its next snapshot (≤ 60 s, usually < 2 s via SSE) |
| settle on one device | that device shows evening; the other continues normally; presence continues only from the other device |
| rename on one device | name is state; the other device updates on next snapshot |
| device in a different timezone | last hello wins for `aviaries.timezone` |
| visitor watching | sees the same version and cues; contributes nothing |

### 7.6 Failure handling

- **Lease lost mid-tick**: the commit's `state_version` check fails; the tick's work is discarded; the next tick reprocesses the same unconsumed events. No duplicates because `processed_version` is set in the same transaction as the state.
- **API down**: the client keeps performing its current cue sheet, then falls back to mood-shaped idle plus micro-motion with no new calls after the horizon ends (birds go quiet rather than looping); an unobtrusive system-voice line appears only if the outage exceeds 2 minutes ("We're having trouble reaching your aviary. Trying again."). It sits in the top bar area, never over the scene, and is not a toast: it is a persistent status line that disappears when the connection returns.
- **Stale snapshot from the edge** (version lower than one the client already has from SSE): ignored.
- **Visitor revocation**: every visitor pull checks the revocation set; revoked → `410` → system-voice page.
- **Clock skew**: server times only; client timestamps are advisory.

---

## 8. Frontend rendering pipeline

### 8.1 Renderer choice: custom Canvas2D, no engine

Canvas2D with cached static layers comfortably renders ≤ 7 rigged vector birds, subtle parallax, and rain at 60 fps on a five-year-old laptop, needs no context-creation cost on the critical path (WebGL context creation costs 50–150 ms on mid-tier phones, which is a third of the first-bird budget), and keeps the renderer ≈ 30 KB gz. A general-purpose game engine is rejected for bundle weight, for frame-pacing opacity, and because the reduced-motion register (§8.8) is easier to build as a sibling render path than as a special case inside an engine. WebGL is deferred; the scene graph is written so a WebGL backend could replace the rasterizer later without touching cue execution or the rig format.

### 8.2 Boot and the first frame (critical path)

The first frame is the aviary, mid-motion, drawn from the inlined snapshot before the main bundle exists.

- **HTML shell** (edge-rendered, ≤ 20 KB): inline critical CSS; the quiet-field sky as a CSS gradient chosen at the edge from the host's local hour (so even the first paint has the right sky); `<script type="application/json" id="snap">` with the snapshot; inline compact rigs for the species present in this aviary (≈ 4 KB each; the edge knows the species from the snapshot); `<script type="module" src="boot.[hash].js">` and `modulepreload` for `scene.[hash].js`.
- **`boot` chunk (≤ 45 KB gz)**: renderer core, rig evaluator, pose/idle generator, cue timeline evaluator, clock map, presence tracker. It draws the first frame by evaluating the cue sheet at `(serverNow − tickAt) + local elapsed`, so a bird mid-flight is mid-flight and a preening bird is mid-preen.
- **`scene` chunk (≤ 250 KB gz)**: audio worklet loader and synth, weather, ornaments, narration/caption prose, chrome (Preact), SSE client. It extends the running renderer; it is the same module graph, not a second renderer, so takeover has no visual discontinuity.
- Lazy chunks: notebook (≤ 40 KB), settings/account (≤ 60 KB), invites (≤ 20 KB), adoption (≤ 25 KB).
- Fonts: system font stack; the lowercase naturalist voice reads well in system fonts and a webfont would risk a flash on the critical path. If design adds a variable font later it loads with `font-display: optional` and ≤ 30 KB.
- Budget breakdown for the 500 ms ceiling on a mid-tier phone over 4G: TTFB ≤ 180 ms (edge → in-region `api` ≤ 80 ms), boot fetch overlapping HTML parse, boot parse+execute ≤ 120 ms with 4× CPU throttling, first `drawImage`-free canvas frame ≤ 16 ms. Typical result ≈ 350–400 ms; 500 ms is the CI ceiling.
- If the inlined snapshot is absent (edge failure), the boot chunk shows the quiet field with at most two faint motion cues (one slow leaf, a light shimmer) and fetches the snapshot; after 8 s it shows the system-voice load-failure line in the chrome area. No spinner exists in the codebase (I12).

### 8.3 Scene composition

Logical stage coordinates with three variants chosen by viewport aspect: wide (≥ 1.4) 1600×700, standard 1200×800, portrait (< 1.0) 900×1000 with the perch zones stacked more vertically (back perch higher and smaller). Anchors for 9 perch slots (3 per zone) and 3 offer spots are designer-specified per variant and always in-bounds with margins, so every bird is always in frame by construction; flight paths are precomputed in-bounds arcs between slots.

Layers (back to front), each an offscreen canvas where static: sky (gradient by time of day) → far foliage (parallax 0.15) → mid foliage and perches (1.0) → birds (per-zone scale back 0.8 / middle 0.9 / front 1.0) → offer items and pool → near foliage (1.25) → ornaments → weather → grade. Static layers re-rasterize only on resize and on time-of-day steps (every 60 s); the day/night tint is applied as multiply and soft-light passes over cached layers rather than re-drawing paths. Parallax is a slow autonomous sway (≈ 0.5 % over minutes) plus a pointer-linked offset ≤ 4 px; both are off in reduced motion.

### 8.4 Bird rig and procedural animation

- **Rig format**: each species is authored as an SVG with named parts (body, head, beak upper/lower, eye, near wing, far wing, tail, legs, feather-detail overlays) and pivots, compiled at build time into compact path commands (`rigc`), ≈ 4 KB per species. The design system defines a 16-step palette ramp per species; `paletteStep` selects the ramp position and feather-detail overlays appear from step 8 upward. The user never chooses appearance; plumage is drift made visible.
- **Animation is parametric, never keyframed loops**: layered continuous signals combined per frame — breathing (0.4–0.8 Hz, amplitude by mood), head scan (Ornstein–Uhlenbeck target; wary fast and wide, drowsy slow and narrow), blink (Poisson ≈ 0.2/s; drowsy half-lidded), weight shuffle (every 20–60 s), fluff (body scale 1.00–1.12 by mood, cold hours, rain), tail flick (alert), plus scripted bouts driven by cues: `preen` (6–15 s parametric sequence), `scan`, `lookToward` (yaw ease-out toward the target's x), `call` (beak envelope driven by the phrase's syllables so the beak opens on the actual notes), `move` (bezier arc with wingbeat 8–10 Hz and a landing settle), `fluff`, `stir` (night greeting).
- **Mood → idle profile**: wary (back-leaning posture, high scan rate), content (preens, relaxed), curious (head tilts, frequent `lookToward`, small hops within the slot), drowsy (low posture, fluff 1.10, half-closed eyes), alert (upright, tail flicks), roosting (eyes closed, fluff 1.12, head tucked, breathing only).
- All idle parameters are seeded by `(birdId, tickNo)`: reproducible within a tick, never repeated across sessions, and visually consistent with the visitor's view without frame-level matching.

### 8.5 Cue execution and interpolation

A timeline scheduler places cue times on the local clock via the clock map (§7.2). `move` cues run along the precomputed arc; a snapshot that disagrees with the current pose by more than a slot (laptop resumed) is resolved with a normal flight, never a teleport. Mood changes cross-fade the idle profile over 3–5 s. Call cues hand the phrase to the audio scheduler with ≥ 500 ms lead and schedule the beak envelope at the same audio time. Offer reactions (approach, peck, drink, bathe, watch) are cues like any other.

### 8.6 Day/night and settle grading

Palette keyframes at 00:00, 05:00, dawn, dawn+1.5 h, 12:00, 17:00, dusk−1 h, dusk, dusk+1.5 h, 23:00 (host timezone; dawn/dusk from §6.1), interpolated per minute with different tint strengths per layer. Settle applies an additional evening grade over 4 s (ease-in-out) and a local quieting: idle profiles lean toward drowsy visuals over 10 s, call intensities ×0.5, master gain −9 dB. Undo or re-engagement (any interaction with a bird, an offer, or a listen-in) reverses over 2 s and resumes presence. Settled is a client render mode; the canonical aviary is unchanged except for the drowsy impulse (D2).

### 8.7 Weather and ornaments

Rain: 120–300 pooled streak particles by intensity (halved under adaptive quality), slight sky darkening, ripple rings on an active pool, birds fluff. Wind: foliage sway amplitude and more frequent leaves. Ornaments: a pooled set of at most six concurrent leaves/feathers, emitted every 20–90 s on gentle sinusoidal falls; a feather gets a brief highlight. Ornaments are client-only and not state.

### 8.8 Reduced-motion register (its own render path)

Triggered by `prefers-reduced-motion: reduce` or the account override (`system | on | off`). `PoseRenderer` replaces `MotionRenderer`:

- Each bird shows one of 3–5 still poses per mood per species, rasterized by the same rig with fixed parameters; a pose changes at most every 6–12 s at idle; transitions are 900 ms opacity cross-fades between two offscreen rasters.
- `preen` = three poses cross-faded in sequence; `move` = cross-fade out at slot A, cross-fade in at slot B, 1.2 s total, no path; `call` = the calling pose (beak open) cross-faded in for the phrase duration with the caption; `lookToward` = a head-turned pose; greeting forms map to pose sequences.
- Ornaments and parallax off; rain is a static streak texture faded in/out over 3 s with the sky darkening; wind is a slow foliage brightness shift; day/night and settle grades remain, slowed ×2.
- Calls, drift, mood, notebook, captions, narration are untouched. This path has its own screenshot fixtures in CI and its own design review; it is a release gate (I13).

### 8.9 Hidden tab, suspend, resize

`visibilitychange → hidden`: cancel the frame loop, stop presence, keep the audio scheduler alive (it is clock-driven inside the worklet, §9.8). `visible`: resume; hidden ≥ 20 s → hello pull; a frame gap > 5 s → plain pull. `pageshow` from bfcache is treated as visible. Resize is debounced 100 ms and re-caches static layers; device pixel ratio is capped at 2.

### 8.10 Top bar and chrome

- DOM (Preact) above the canvas: four icon buttons — account/settings, accessibility, field notebook, offer — and a lowercase text affordance "settle" at the right end (D20). Nothing else: no badges, no counts, no status dots.
- Fade: after 4 s without pointer or keyboard activity the bar fades to opacity 0.12 over 1.2 s; any activity restores it within 150 ms; it never fades while it or an open sheet has focus; on touch devices the same timing runs from the last touch.
- Sheets (notebook, offer, settings, accessibility) slide in as side panels over the right portion of the scene, `role="dialog"`, Escape or outside-click closes, focus returns. The scene carries no persistent chrome; a user-opened, transient panel is the interpretation of "no UI chrome inside the aviary" (D21).
- Connection trouble after 2 minutes shows a persistent system-voice status line in the chrome area (not over the scene) that disappears when the connection returns. It is a status, not a toast (no animation in, no dismiss button, no stacking).

### 8.11 Loading, empty, and adoption states

- Loading and empty aviary share the quiet field (§8.2). After adoption, the first bird flies in at ≈ 1.5 s and the second at 4–7 s to their starting perches; the user never sees an empty aviary again.
- Adoption is a single naturalist-voice page reached once after the first magic-link consume: "two birds arrived this morning. a small grey finch and a yellow-green warbler. they'll need names." Two inputs pre-filled from a curated list of ≈ 60 short names, one action ("go to the aviary"). No species choice, no catalog, no preview cards.

### 8.12 Frame loop and adaptive quality

Logic at a fixed 60 Hz step with interpolation; render on `requestAnimationFrame`. If p95 frame time exceeds 20 ms over 3 s: DPR 2 → 1.5 → 1, halve rain particles, drop the pointer parallax; step back up after 30 s of headroom. Bird animation fidelity is never reduced; ornaments go first.

### 8.13 Memory discipline

Typed-array particle pools, reusable cue/phrase objects, no per-frame closures in the hot path, a fixed set of offscreen canvases, listeners registered once, snapshots replaced not appended, `EventSource` closed before reconnect, persistent audio nodes, and a virtualized notebook (window of 40 entries; an LRU of 200 entries backs scroll-back, everything else is refetched by cursor). The CI soak test (§14.5) is the enforcement.

---

## 9. Audio pipeline

### 9.1 Graph

```
per bird:  BirdVoice (AudioWorkletNode) → Gain (listen-in) → StereoPanner (perch x, ±0.6)
           → Biquad lowpass (zone depth: back 2.5 kHz −6 dB, middle 5 kHz −3 dB, front open) → chorusBus
ambient:   AmbientBed (AudioWorkletNode: wind + rain from filtered noise) → Gain → chorusBus
chorusBus → reverb send (Convolver with a runtime-generated exponential-decay noise IR, ≈ 1.2 s)
chorusBus → DynamicsCompressor (−18 dB, 2:1, slow attack/release) → masterGain (volume × settle × night) → destination
```

### 9.2 Voice synthesis (AudioWorklet)

One persistent `BirdVoice` processor per bird: two oscillators (sine carrier plus one partial at 2× or 3× with a harmonic tilt), an optional FM modulator (15–60 Hz) for trills, a noise source through a bandpass for chips and the nightjar's churr, a formant peaking filter set by timbre, per-syllable pitch envelopes (linear, exponential, rise-fall glides) and amplitude ADSR. Syllable lists arrive over the message port with absolute audio times; the processor keeps its own queue. No nodes are created per call; buffers are preallocated; CPU target < 1 % per voice on the reference laptop. Fallback when the worklet module fails to load: a standard-node voice using short-lived `OscillatorNode`s from the same phrase→parameter mapping (still procedural, still bounded).

### 9.3 Phrases, variation, and captions

The client derives the signature from `signatureSeed`, then `callPhrase(signature, mood, intensity, phraseSeed)` (shared engine, §6.14) → syllables. Per-syllable micro-variation (±30 ms timing, ±15 cents, ±1.5 dB) comes from the seed; `phraseSeed`s are unique per cue, so no two phrases are ever identical, and a property test over 100 k seeds asserts it. `caption(phrase, ctx)` produces the caption from the same syllables, so the caption is what was actually sung.

### 9.4 Chorus

Independent voices, independent phase, mixed at runtime through the shared reverb and a gentle compressor; responses and choruses arrive as cues, so two devices hear the same chorus. Panning and zone filtering give depth. Maximum 7 voices plus ambient; the compressor prevents loudness build-up without pumping (tested with 7 simultaneous alert-mood phrases).

### 9.5 Listen-in mix

Engage: the focused bird's gain ramps to +3 dB and its lowpass opens fully; every other bird ramps to −10 dB (`setTargetAtTime`, τ = 0.6 s, ≈ 2.0 s to settle). Floor is −14 dB: never a mute. Disengage: reverse ramps ≈ 2.5 s. The server's listen-in bias (rate ×1.6) only takes effect via the on-demand tick and the next cue sheet; the mix change is immediate.

### 9.6 Ambient beds

Wind: pink noise → lowpass 400 Hz with a slow LFO, −36 dB nominal rising to −24 dB during wind events. Rain: white noise → bandpass 3–6 kHz plus synthesized droplet ticks at −30 dB. Both fade over 3–5 s. No recorded ambience of any kind (I6).

### 9.7 Settle and night

Settle: master −9 dB over 4 s and call intensity ×0.5 locally. Night: master −4 dB; the engine already reduces call rates; the nightjar keeps calling.

### 9.8 Scheduling and time base

Server time → `AudioContext.currentTime` via the clock map and `getOutputTimestamp()`. Cues are posted to the worklet ≥ 500 ms ahead with absolute audio times; the worklet triggers sample-accurately regardless of main-thread timer throttling in background tabs (D19: audio continues in a hidden tab if the user has it on; presence does not). The beak envelope uses the same audio time so sound and motion are one event.

### 9.9 Autoplay policy and the silence fallback

The `AudioContext` is created at boot. If it starts `suspended` (autoplay policy), the aviary renders normally with **captions shown by default** until the first pointerdown/keydown anywhere resumes the context and fades audio in over 2 s (D6). If construction fails, or both the worklet and fallback voice fail, the aviary plays in graceful silence with captions on and a system-voice note inside accessibility settings ("Sound isn't available in this browser. Captions are on."). iOS `interrupted` states resume on visibility. There is no recorded-audio path and no "enable sound" banner over the scene.

### 9.10 Audio quality gates

- Signature-distance test: every pair of birds in any aviary has perceptual distance ≥ threshold (unit test over the farthest-point selector with 10 k simulated aviaries of 7).
- Listening study (§14.4) before beta and after any motif change: ≥ 80 % identification for 5 birds, ≥ 70 % for 7.
- Chorus artifact test: render two voices offline, check for periodic comb-filter energy (no shared phase source).
- Golden-ear review by the sound designer on a fixed seed set each release.

---

## 10. Accessibility surfaces

### 10.1 Narration engine

- Client-side `narrate()` from `@aviary/prose` (the same template library and vocabulary the notebook uses, split into a client entry). Two alternating `aria-live="polite"` regions outside the canvas (alternation forces re-announcement of repeated text).
- Cadence: idle lines every 45 s ± 15 s jitter; at most one idle line queued (a newer one replaces it). Event lines — greeting, offer reaction, settle acknowledgment, newcomer arrival — emit within 500 ms and clear any pending idle line. No line is a state list: templates are scene sentences that pick 1–2 salient facts from the last 60 s of cues and the snapshot ("pip is on the front rail, calling softly. wren sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.").
- First mention of a bird per session is "pip, a small grey bird"; later mentions use the name; unnamed newcomers use "a newcomer, a speckled thrush" (D10). Visitor narration is identical minus greeting lines.
- Optional visible narration line (setting) rendered beneath the scene in the chrome area, AA contrast on a backdrop.

### 10.2 Captions

From `caption(phrase, ctx)`: syllable-count word ("three-note"), contour ("rise", "trill", "fall"), intensity adjective from intensity and timbre ("soft", "low", "sharp", "bright"), and an origin clause when the bird is not at the front ("from the back perch"). Rendered as a small pill near the bird with a translucent backdrop (AA guaranteed against any scene state), fade in 150 ms, out 400 ms after the phrase; at most two visible. Default on when audio is blocked or off; toggle in accessibility settings; synced per account.

### 10.3 Keyboard and focus

- Tab order: top-bar buttons → settle affordance → scene group. Inside the scene: roving `tabindex` over bird hotspots (invisible DOM buttons positioned over each bird, updated at 10 Hz from the renderer). Left/Right arrows move by x order; Enter/Space toggles listen-in; Escape ends listen-in, or leaves the group if none is active; Tab exits the group.
- Offer sheet: radio group for the item, a three-spot placement group for seeds, an "offer" button; Escape cancels. Settle: activating the affordance settles; for 5 s a second activation or any click undoes; the narration says "the aviary settles. press again to undo."
- Single-character shortcuts (o, s, n) are opt-in in accessibility settings (WCAG 2.1.4).
- Focus indicator: a 2 px outline with a 2 px offset halo in DOM over the hotspot, colors from the design system validated in CI against day and night screenshots at ≥ 3:1.

### 10.4 Semantics

Scene container `role="group" aria-label="aviary"`; the canvas is `aria-hidden`; hotspots are `role="button"` named "pip, a small grey bird, on the front perch" with `aria-pressed` for listen-in; sheets are dialogs with focus trap and focus return; the notebook is a list labeled "field notebook"; live regions live outside the canvas.

### 10.5 Reduced-motion and settings

The system preference applies automatically; the account override lives in accessibility settings (system voice) with captions, audio on/off, visible narration, and opt-in shortcuts. Settings sync per account; the reduced-motion system preference is evaluated per device.

### 10.6 Contrast

Chrome text ≥ 4.5:1, large text and icons ≥ 3:1; captions and narration on backdrops; the top bar is never faded while focused or hovered, so faded-state contrast is never the reading state. Screenshot-based contrast checks run against dawn, midday, dusk, night, and settled palettes.

### 10.7 Testing

axe-core in CI on every route; Playwright keyboard-flow tests (greeting → listen-in → offer → settle → undo); a manual matrix per milestone (VoiceOver + Safari, NVDA + Firefox, JAWS + Chrome, TalkBack + Chrome Android); reduced-motion screenshot fixtures; a writer's review of narration transcripts captured from synthetic sessions (the transcript must read as prose, not a log).

---

## 11. Voice and copy as an engineering system

- `packages/copy` holds two registries: `naturalist/` (notebook templates, narration templates, caption vocabulary, adoption page, offer sheet, settle line, newcomer line) and `system/` (auth, sessions, errors, settings, privacy, visitor pages, unsupported browser, connection line). Every key carries a `surface` tag; components assert the tag at build time so a naturalist string cannot render in a system surface or vice versa.
- **Copy lint (CI)**: naturalist — lowercase (user-typed bird names inserted verbatim), no exclamation marks, no "you/your", no forbidden vocabulary (welcome, congratulations, unlocked, achievement, streak, level, score, badge, reward, days), no numerals for user actions, present-tense heuristics; system — sentence case, ≤ 2 sentences, errors include a next step, no bird vocabulary.
- ESLint bans string literals in UI components (`t('key')` only) and bans the identifiers `Toast`, `Banner`, `Badge`, `Snackbar`, `Spinner`, `Streak`, `Achievement`.
- System-voice copy set for v1 (draft): magic link sent ("Check your email for a sign-in link. It expires in 15 minutes."); consume failure, session timeout, and load failure (as in the PRD); unsupported browser ("Pocket Aviary needs a current version of Chrome, Safari, Firefox, or Edge."); visit unavailable ("This visit is no longer available."); invite sent ("Invitation sent. It expires in 30 days."); deletion pending ("Your account is scheduled for deletion on <date>. Sign in before then to keep it." with an "I changed my mind" action); export requested ("We'll email a download link. It works for 24 hours."); sound unavailable; connection trouble.
- Naturalist copy set for v1 (draft): adoption page, newcomer line ("a small bird has been lingering at the back perch. give it a name, or let it be for now."), active-offer line ("a seed is out on the rail"), settle acknowledgment narration, ≈ 80 notebook templates, ≈ 40 narration templates, caption vocabulary.

---

## 12. Accounts, privacy, and security

### 12.1 Magic-link sign-in

Normalize the email (trim, lowercase; no plus-address stripping), compute the blind index, mint a 32-byte token, store its SHA-256 with a 15-minute expiry, and email the link. Consumption is a single atomic `UPDATE … WHERE consumed_at IS NULL RETURNING`; success creates the account on first use (with the aviary, two starters, and the materialized `bird_arrivals` schedule), issues a session (32 random bytes, hashed at rest, 180-day sliding cookie, `last_seen_at` updated at most hourly), and redirects. Requests always return `202`. Limits: 5 per email per hour, 20 per IP per hour, plus a global breaker. Replayed or expired links render the system-voice failure page.

### 12.2 Sessions and email change

The sessions list shows a device label (browser family and first-seen date) with the current session marked. Revocation writes the DB row and a Redis revoked-set entry; the revoked device's next request lands on sign-in with the timeout message. Email change stores the pending address encrypted with its own blind index and a 15-minute verification token; the old address works until the switch commits, at which point `email_bidx` rotates.

### 12.3 Export and deletion

- **Export** (job): account settings, aviary age and timezone, birds (id, name, species, arrival date, current personality vector, attention accumulator, mood), notebook entries, outstanding invitations. Written to object storage, emailed as a 24-hour signed link. This is the one place raw personality values leave the simulation boundary; the PRD names it explicitly and it is data portability, not a product surface (D1).
- **Deletion**: soft delete sets `deletion_requested_at` and `aviaries.paused = true` (the aviary stops ticking; restore triggers catch-up, §6.17). Sessions remain so the user can sign in; every signed-in page shows the deletion-pending system line with an "I changed my mind" action. After 30 days a job hard-deletes every row tied to the account UUID in one transaction, revokes invitations, deletes exports, purges Redis keys, and requests suppression at the email provider. A deletion audit record holds only the UUID and timestamps for 90 days.

### 12.4 The privacy pipeline

- The simulation database is network-isolated to `api`, `sim-*`, and `jobs`. The observability stack ingests metrics, traces, and logs from services, never rows. No analytics warehouse connects to it; there is no event-tracking SDK on the client.
- **Metrics label allowlist**: `service, route, status, region, browser_family, device_class, country, tick_reason, phase, setting_bucket`. The metrics client throws on any other label, so `account_id`, `aviary_id`, `bird_id`, and `session_id` cannot be attached even by accident.
- **Logs**: structured; a redaction layer rejects records containing `email`, event `payload`, or bird trait fields. The account UUID may appear for request correlation (the PRD designates it as the internal reference); retention 14 days.
- **Retention**: `interaction_events` partitions are dropped 7 days after their date (events are consumed within minutes); RUM raw samples 30 days, aggregates 13 months; visit sessions persist because the visit log is a product feature.
- Feature flags for rollout are global or keyed by a salted hash of the account UUID; no flag or experiment conditions on bird or interaction state.

### 12.5 Security

Cookies are `HttpOnly; Secure; SameSite=Lax`; mutations check `Origin` and `Sec-Fetch-Site`; a strict CSP allows only hashed external modules (the inline snapshot is a `type="application/json"` data block, which is not executed); HSTS; JSON-schema validation on every body; dependency audit in CI; secrets and encryption keys in KMS with envelope encryption for email; least-privilege DB roles (I1). Visitor tokens are 32 bytes, hashed at rest, consumed once to bind a `pa_v` cookie valid 30 days from consumption (D22); the visitor router serves a projected snapshot and has no write routes at all.

### 12.6 Visit privacy

Visitors see bird names, species, moods, perches, weather, and host-timezone lighting; never the host's email, settings, notebook, or visit log (D11). The host's visit log shows the visitor's email (the host typed it), the date, and duration rounded to 5 minutes. Visit notifications, if opted in, are one email per visitor per day at most, off by default, and never mentioned during onboarding (D23).

### 12.7 Unsupported browsers

Boot feature-detects ES2022 modules, Canvas2D, `visibilityState`, and `AudioContext` (optional). Failing the required set renders the system-voice unsupported page; audio-only failures fall to §9.9.

---

## 13. Performance budgets and observability

### 13.1 Budgets (gates unless marked informational)

| metric | budget | measured by |
|---|---|---|
| JS before first bird (HTML inline + boot) | ≤ 80 KB gz | bundle CI |
| total JS across all chunks | internal target ≤ 600 KB gz; hard ceiling 2 MB gz | bundle CI |
| time to first bird | ≤ 500 ms at p75 of 5 runs, mid-tier phone profile (4× CPU throttle, 4G: 9 Mbps / 170 ms RTT) | synthetic Playwright in CI and fleet |
| time to first bird, real users | p75 ≤ 700 ms (informational) | RUM |
| idle frame time | p95 ≤ 16.7 ms across a 30-minute session on the reference laptop (2021 mid-range, integrated GPU) | nightly lab run |
| memory | JS heap growth ≤ 5 % from minute 5 to minute 30; DOM node count flat; exactly one `AudioContext` | CI soak |
| snapshot size / read latency | p95 ≤ 8 KB; `api` p95 ≤ 60 ms; edge inline hop ≤ 120 ms | server metrics |
| tick duration | p50 ≤ 20 ms, p99 ≤ 500 ms; **alarm at p99 > 5 s** | server metrics |
| on-demand tick round trip (offer reaction) | p95 ≤ 300 ms | server metrics |
| event post latency | p95 ≤ 100 ms | server metrics |
| magic-link email delivery | p95 ≤ 60 s | provider webhooks |
| worklet load failure | ≤ 0.5 % of sessions | RUM |

### 13.2 What we measure

- **Server** (OpenTelemetry → Prometheus/Grafana): request rate, latency, and errors by route; tick duration histograms by `tick_reason` (scheduled, dormant, on-demand, catch-up); tick backlog and lease waits; events consumed per tick (histogram, no ids); snapshot bytes; SSE connection counts; email delivery latency; job durations; retention-job row counts.
- **Client RUM** (aggregate-only, 10 % sampling, coarse dimensions, one beacon at session end): TTFB, first-bird time, boot execute time, frame-time p50/p95 computed client-side, long tasks, adaptive-quality step-downs, `AudioContext` state at first frame (running / suspended / failed), worklet load result, snapshot fetch latency, event post failures, session duration in 5-minute buckets, presence-loss reason counts (visibility / focus / inactivity) by narration-setting bucket (to watch R4).
- **Synthetic fleet**: Playwright runners in five geographies every 10 minutes against staff accounts: navigation, first-bird time, greeting cue executed within 2 s, a 2-minute frame-time sample; nightly 30-minute soak with heap snapshots; weekly full accessibility run.
- **Alarms**: tick p99 > 5 s over 5 minutes; any active aviary without a tick for 3 minutes; `api` 5xx > 1 %; synthetic first-bird p75 > 500 ms for three consecutive runs; email delivery p95 > 120 s; worklet failure > 2 %; RUM audio-failed > 3 %; retention job skipped.

### 13.3 What we deliberately do not measure

Anything with a per-account or per-bird dimension; drift distributions across users; retention, streak, or engagement dashboards keyed to presence; visit counts as a metric; notebook content; experiments conditioned on bird state. Calibration evidence comes only from the harness and staff accounts (D24). The single behavioral aggregate is the anonymized, bucketed session-duration histogram.

---

## 14. Testing and calibration strategy

1. **Unit and property tests**: engine invariants (§6.18), presence state machine, clock map, copy lint, phrase and caption determinism, signature distance.
2. **Contract tests**: snapshot allowlist (I4), visitor projection, event validation, DB grants (I1), trait triggers (I2), metrics label allowlist (I10).
3. **Calibration harness** (`tools/harness`, runs in CI in < 2 minutes): the engine at accelerated time over 400 simulated days with personas — *regular* (12 min/day × 6 days, a 3-minute listen-in most days), *weekend hour*, *absent-and-return* (3 weeks regular, 2 weeks away, regular), *open-but-idle laptop* (visible and focused 10 h/day, activity 10 min → credits ≈ 10 min + activity window, not 10 h), *two-device* (overlapping sessions credit once), *visitor-heavy* (identical to regular), *night owl*. Asserts the §6.5 targets, the §6.6 visibility rule measured from planned cues (front-perch share, call rate, greeting share), notebook sparsity (6–15 entries per 30 days for regular, ≥ 2 during absence), mood sanity (no bird in one mood > 80 % of daylight), and chorus frequency (≥ 1 per day with five or more birds). Constant changes require harness sign-off.
4. **Listening studies**: 12 internal participants at M2 and 12 external before launch; 10 minutes of exposure to a seven-bird aviary with names shown on each call, then 40 identification trials — targets ≥ 80 % for five birds and ≥ 70 % for seven — plus a repetition rating over a 5-minute listen (≤ 10 % report any repeated call).
5. **Performance and soak CI**: per-chunk budgets; throttled synthetic first-bird gate; a 30-minute headless soak with heap snapshots at minutes 5 and 30 (growth ≤ 5 %), flat DOM count, long-task count; nightly frame-time run on the reference laptop.
6. **Accessibility** (§10.7).
7. **Visual regression**: seeded screenshot fixtures across five times of day × three stage variants × motion/reduced-motion × weather states, for both render paths.
8. **End-to-end**: magic link with a mail sink; adoption; a greeting cue executed within 2 s of first frame; listen-in ramps verified by offline audio rendering; offer round trip; settle and undo; two browser contexts holding equal versions and cues; visitor flow including revocation; export; deletion and restore with catch-up; dormant catch-up equivalence (three dormant days equal continuous ticking within tolerance).
9. **Voice QA**: the writer reviews 1,000 harness-generated notebook and narration samples each milestone; a test asserts no aviary repeats an entry's text within 30 days.

---

## 15. Rollout

### 15.1 Milestones (≈ 22 weeks, team of eight)

| milestone | weeks | scope | exit criteria |
|---|---|---|---|
| M0 foundations | 1–2 | monorepo; CI with budgets, copy lint, banned identifiers from day one; schema and DB roles; auth; engine skeleton with PRNG; harness skeleton; copy registry | lints and grants tested; magic-link e2e green |
| M1 engine and first frame | 3–6 | tick pipeline, presence, attention accumulator, drift, mood, weather, cue planning; snapshot API and edge inline; boot renderer; two species rigs; interpolation | harness passes calibration targets; synthetic first bird < 500 ms |
| M2 audio | 7–10 | worklet synth; six motif libraries; signatures with farthest-point; chorus; listen-in mix; captions; ambient beds; autoplay handling | listening study #1 meets the five-bird target; no-repeat and signature-distance tests green |
| M3 interactions and voice | 11–14 | greeting; offers; settle; notebook observer and templates; narration; keyboard and focus; reduced-motion register; top bar and sheets; settings; error surfaces | accessibility matrix pass on both render paths; writer sign-off on corpora |
| M4 growth, social, account | 15–17 | arrivals; adoption page; visits; export; deletion and retention jobs; synthetic fleet; RUM; alarms | staff alpha on backdated aviaries with 3–7 birds (backdating is a staging-only operation) |
| M5 closed beta | 18–21 | ≈ 300 invited users; calibration review from harness and staff accounts; audio round two and the external listening study; external screen-reader sessions; device soak; bug bash | launch checklist green |
| M6 launch | 22 | open sign-up, web only | no in-product launch surfaces (I7) |

### 15.2 Ramping birds per aviary

The age schedule is the ramp: the launch cohort cannot reach a third bird before ≈ day 75, while staff aviaries will have run six and seven birds for months. Two server knobs: the schedule days, and a global pause on arrivals. If the seven-bird chorus fails the listening study, the 540-day slot is delayed rather than shipped blurry. Schedules never shift earlier for some users as a reward; changes apply uniformly by age.

### 15.3 Instrumented from day one

Everything in §13, the harness in CI from M0, a weekly voice audit of copy diffs, the listening-study cadence, and monthly alarm reviews.

### 15.4 Launch checklist

Budgets green; accessibility matrix on both render paths; privacy review (label allowlist test, retention job exercised on a test account, export and delete e2e); security review (auth, visitor scope, CSP, rate limits); runbooks (tick backlog, email provider outage, Redis loss — leases and the active set are reconstructible, Postgres failover); on-call rotation.

### 15.5 After launch

Weekly patch cadence; calibration changes only through versioned constants with harness sign-off; no feature that touches I7 or I8 without a PRD change.

---

## 16. Risks

**Drift calibration**

- **R1 Drift too fast or too slow.** Harness targets and versioned, additive constants make re-calibration safe; staff longitudinal aviaries act as sentinels. Production aggregates are off-limits, so persona realism in the harness is reviewed quarterly.
- **R2 Presence inflation through lax detection.** The activity window and `focusin` inclusion are the levers; the open-but-idle synthetic test and per-day caps bound the damage.
- **R3 Double counting across devices.** Interval union; tested.
- **R4 Assistive-technology users under-credited** (no pointer or key events while reading a live region). `focusin` and key events count; the RUM presence-loss reason by narration-setting bucket watches for skew; if skew appears, extend the activity window for sessions with narration enabled (a setting-level, not account-level, rule).
- **R5 Heavy users saturate headroom.** Quadratic headroom and caps; the year-two harness check.

**Sync correctness**

- **R6 A second writer after lease expiry.** Fenced lease plus optimistic version check plus `processed_version` in one transaction; a chaos test kills workers mid-tick.
- **R7 The edge serves a stale or wrong-account snapshot.** `Cache-Control: private, no-store`; a test asserts the header and that the inlined aviary id matches the session; a canary alternates two staff accounts.
- **R8 Cue timing drift.** Server time everywhere; the clock map resets on jumps; e2e asserts cue execution within ±250 ms.
- **R9 Event loss offline.** Outbox and beacon; presence intervals are dropped after 5 minutes offline rather than fabricated.

**Audio**

- **R10 Synthesis sounds synthetic.** Sound designer embedded from M2; formant filtering, noise components, shared reverb; golden-ear review; the repetition rating in the study. Fallback plan: richer additive or modal synthesis, still procedural.
- **R11 Birds indistinguishable at six or seven.** Farthest-point signatures and tag motifs; the seven-bird study gate; delay the last schedule slots if needed.
- **R12 Autoplay blocks first-frame audio.** Captions on while suspended; first gesture resumes; RUM tracks the first-frame audio state.
- **R13 CPU spikes cause glitches on old laptops.** Worklet processing budget, bounded voices, visuals degrade first, offline render tests.
- **R14 Background-tab audio suspended on iOS.** Accept silence there; resume on visible.

**Accessibility**

- **R15 Narration decays into state lists as templates grow.** The writer owns templates; lint flags list-like patterns; transcript review each milestone.
- **R16 The reduced-motion path rots.** Every cue kind must implement both renderers (a type-level exhaustiveness check) and both paths have screenshot fixtures.
- **R17 Hotspots drift from rendered positions on resize or DPR change.** Hotspots use the renderer's anchor math; an e2e test asserts overlap.
- **R18 The top-bar fade hides focus.** Never fade while focused; tested.

**Performance**

- **R19 Bundle creep.** Per-chunk budgets and a size report on every PR.
- **R20 Memory growth from audio or notebook.** Pools, persistent nodes, virtualization; the soak gate.
- **R21 Far-region users miss the first-bird budget** because the edge→`api` hop is cross-region in v1. Accepted for launch markets; follow-up: regional snapshot replicas in Redis.

**Privacy, voice, product**

- **R22 Email leaks into logs or metrics.** Redaction, label allowlist, CI grep, periodic audit.
- **R23 "Harmless" announcement surfaces creep in** (a toast, an unread dot on the notebook icon). Lint bans, PR attestation, design review; the notebook icon explicitly has no unread state.
- **R24 The notebook is too chatty or too silent.** Harness sparsity asserts; salience tuning; bypass only for arrivals.
- **R25 Timezone and DST edge cases.** Last hello wins; IANA math; continuous mood pressures mean a jump never snaps mood.
- **R26 Magic-link deliverability.** Reputable provider, SPF/DKIM/DMARC, delivery alarm, an easy "request a new link" path.
- **R27 Forwarded visitor links.** One-time consumption binds a cookie; the host's visit log and revocation cover the rest.
- **R28 A user signs up at night and meets roosting birds.** New arrivals are `alert` for their first 10 minutes regardless of hour (D26), then the time-of-day pressures take over.

---

## 17. Decision log (ambiguities resolved)

| # | decision |
|---|---|
| D1 | The account export includes raw personality vectors because `accounts_sync.md` lists them; the "never exposed numerically" rule is read as a product-surface rule. No in-product surface, including the birds panel and narration, ever shows a number. |
| D2 | Settle is device-local: evening grade, quieting, and presence end on that device; the server records the event and a drowsy impulse only. |
| D3 | Dormant aviaries tick every 15 minutes with `dt = 15 min` and no cue planning; active aviaries tick every 60 s. The aviary always advances server-side; only granularity changes. |
| D4 | Seed offers take a placement spot (three keyboard-selectable spots along the front); the pool lands front-center; song fragments have no location. "Near a bird" is spot-to-zone proximity. |
| D5 | The bird night state is `roosting` in code to avoid colliding with the user's settle gesture; product copy may still say "settled." |
| D6 | Under autoplay policy the aviary renders with captions on until the first gesture, then audio fades in. No "enable sound" banner. |
| D7 | Birds 3–7 arrive on the back perch on the age schedule; the notebook records the arrival; naming happens in the birds panel; arrivals cannot be declined and never expire. |
| D8 | The host's timezone is the last device's IANA zone reported on `hello`. |
| D9 | Presence is unioned across devices to wall-clock time. |
| D10 | Narration names a bird with its species descriptor on first mention per session, then by name. |
| D11 | Visitors get no notebook and no listen-in (not even local); visits are strictly render-only. |
| D12 | An HMAC blind index of the email exists for sign-in lookup only; it is not the email and is never used as an identifier elsewhere. |
| D13 | Traits are floats in [0,1] with species seeds in 0.25–0.50; drift uses quadratic headroom. |
| D14 | Activity window default 5 minutes (range 3–8); presence intervals every 20 s. |
| D15 | Listen-in credit is clamped to presence intervals; listen-in auto-ends after 60 s hidden. |
| D16 | Notebook and narration are template grammars, deterministic and in-house; no LLM, so per-bird state never leaves the boundary and the voice is reviewable. |
| D17 | Weather is canonical server state; ornaments are client-only. |
| D18 | Visitors' day/night follows the host's timezone. |
| D19 | Audio keeps playing in a hidden tab if the user has it on; rendering and presence stop. |
| D20 | The top bar has the four specified icons plus a lowercase "settle" text affordance, reconciling the layout file's four-icon list with the interaction and accessibility files placing settle in the top bar. |
| D21 | Notebook, offer, and settings open as transient side panels over the scene; the scene itself carries no persistent chrome. |
| D22 | A consumed visit link binds a visitor cookie valid 30 days from consumption; revocation and invite expiry override it. |
| D23 | Opt-in visit notifications are email only (there is no push), at most one per visitor per day. |
| D24 | Calibration evidence comes only from the harness and staff accounts; no production aggregate of drift or interaction exists. |
| D25 | Canvas2D renderer with cached layers; WebGL deferred. |
| D26 | Newly arrived birds are `alert` for their first 10 minutes regardless of hour. |
| D27 | Song-fragment offers are procedural motif sequences played by the client synth. |
| D28 | Starters exclude the nightjar so the first encounter is daytime-active. |
| D29 | Names are 1–24 characters in any script and are inserted verbatim into prose. |
| D30 | The connection-trouble line is a persistent status in the chrome area shown only after 2 minutes; it is not a toast and never overlays the scene. |

---

## 18. Work breakdown and team

**Workstreams and owners** (eight people): engine (2 engineers: tick, drift, mood, cues, observer, harness); client rendering (2: boot, rigs, animation, reduced-motion, chrome, notebook); audio (1 engineer + a sound designer: worklet synth, motif libraries, signatures, mix, captions); platform (2: auth, API, edge, sim scheduling, jobs, privacy pipeline, observability, security); prose and design (a writer part-time owning both copy registries and template libraries; a visual designer owning rigs, palettes, stage anchors, focus treatment). Accessibility QA is shared, with external screen-reader testers engaged at M3 and M5.

**Critical dependencies**: copy registry before any UI (M0); two rigs before renderer polish (M1); motif libraries before the first listening study (M2); the harness before any calibration claim (M1); both render paths before the accessibility matrix (M3); backdated staff aviaries before the seven-bird study (M4).

**Definition of done for v1**: every invariant in §1 has its automated check green; every budget in §13.1 is met; the launch checklist in §15.4 is signed; and a person who opens the tab meets a bird that notices them, in under half a second, with no text telling them so.
