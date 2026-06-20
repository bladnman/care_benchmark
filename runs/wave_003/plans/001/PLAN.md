# Pocket Aviary — Implementation Plan (v1)

A comprehensive, executable plan for shipping v1 of Pocket Aviary. It interprets the PRD into specific architectural, data, API, engine, sync, rendering, audio, accessibility, performance, rollout, and risk decisions. It is for a frontier engineering team and assumes no further clarification. Where the PRD left a value to be calibrated during build, the plan names a starting value and a calibration loop.

A short index of how the plan is organized:

1. Scope
2. Architecture
3. Data model
4. API surface
5. Simulation engine design
6. Sync model
7. Frontend rendering pipeline
8. Audio pipeline
9. Accessibility surfaces
10. Performance budgets and observability
11. Rollout
12. Risks
13. Open calibrations (defensible calls noted inline)

---

## 1. Scope

### 1.1 In scope for v1

- Single-user accounts with email magic-link sign-in; per-device session tokens, revocable from settings.
- One canonical aviary per account, two starter birds at adoption, hard cap of seven birds per aviary.
- Server-side simulation tick (~once per minute) advancing canonical personality vectors, mood, mood timers, ambient events, and the day/night/weather state.
- Multi-device sync as a property of the architecture (one canonical record, many read-only clients).
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook, presence accounting.
- Procedural call synthesis client-side via WebAudio; chorus mixing; listen-in mix decay; graceful-silence fallback with captions on by default.
- Field notebook: auto-generated, read-only, sparse, naturalist prose, scrollable indefinitely.
- Screen-reader narration (running naturalist prose, slow cadence), reduced-motion mode (designed cross-fade surface, not "animations off"), call captioning, WCAG AA contrast, full keyboard navigation.
- Visit-invitation feature: opt-in, per-invite, revocable, read-only ambient; visit log; optional host notifications (off by default); 30-day invite expiration.
- Account export (emailed JSON), soft-delete (30-day) then hard-delete.
- Privacy boundary: per-bird interaction events never enter aggregate telemetry, analytics warehouse, or any ML pipeline.

### 1.2 Out of scope (per `non_goals.md`)

- Native iOS/Android apps (web-only).
- Gamification of any flavor: no achievements, streaks, levels, scores, badges, calendars, XP, rank, tier.
- Tamagotchi mechanics: no death, hunger, distress, decaying happiness meter; no negative drift on neglect.
- Social-network surfaces beyond the single visit affordance: no profiles, follows, feeds, discovery, comments, leaderboards.
- Push notifications, emails about the aviary, pings about visits (default).
- Shared/multi-aviary accounts, customizable scenes, payments, public discovery.
- Recorded-audio fallback path (silence + captions is the fallback).
- Compatibility paths for browsers older than the last two major versions of Chrome, Safari, Firefox, Edge.

### 1.3 Hard rules the plan treats as invariants

These are non-negotiable and the architecture is shaped around them:

- The personality vector is never exposed numerically to the user (no stats panel, no debug surface, no toggle, no future tier).
- Drift is monotonic toward expressive; neglect produces ambient quietness, never negative drift.
- Only the server simulation tick writes personality vectors. Clients submit interaction events only. No last-write-wins on personality.
- Presence is the conjunction of (visibilityState == visible) AND (document has focus) AND (pointermove/keypress in last activity window). All three, simultaneously.
- The first frame the user sees has birds mid-action; no entry animation, no spinner, no fade-from-static.
- No "Welcome back" toast/banner/modal anywhere, ever.
- Calls are procedural, synthesized client-side via WebAudio. No recorded audio.
- Per-bird interaction state never enters aggregate telemetry, the analytics warehouse, or any ML pipeline.
- Internal identifiers are synthetic UUIDs; email is stored once, encrypted, never used as a key elsewhere.

### 1.4 Defensible calls (noted where the PRD left room)

- **Tick cadence**: starts at 60 seconds. Calibrated against p99 tick latency budget (5s alarm) and the felt-continuity requirement; may move to 45–90s during build.
- **Presence activity window**: starts at 180 seconds, leaning long (watching without moving is the product). Calibrated in a build-time dogfood with two test cohorts (active vs. passive watchers).
- **Mood state set**: wary, content, curious, drowsy, alert, settled. `settled` is added to the PRD's example set to give the settle gesture and night state a first-class state rather than overloading `drowsy`.
- **New-bird pacing**: third bird offered at aviary age 60 days; fourth at 120 days; fifth at 240 days; sixth at 365 days; seventh at 540 days. Paced to aviary age only, never to visit count or interaction score. Tunable.
- **Caption text generation**: client-side, from the same call-grammar parameters that produced the audio, not from a fixed string table.
- **Narration authoring**: client-side generator backed by a naturalist template lattice, fed by the same snapshot the visual reads. Server-side generation is deferred; the client generator is the v1 path because it keeps narration in sync with the rendered frame without a round-trip.

---

## 2. Architecture

### 2.1 Service shape

Three services plus a CDN edge. All services are owned by one team and deployed together; the boundaries exist for scaling and failure isolation, not for organizational separation.

- **Edge / CDN**: serves the HTML shell and the initial state-snapshot payload inline with the HTML (so first-bird-visible can hit <500ms on 4G). Static assets (JS bundles, SVG sprite sheets, motif library JSON) cached aggressively with content-hash filenames.
- **API gateway**: terminates TLS, validates session tokens, routes to internal services. Stateless and horizontally scaled. Owns rate-limiting for magic-link requests.
- **Simulation service**: the only writer of aviary canonical state. Runs the tick, consumes the event log, computes drift deltas, transitions moods, advances ambient event state. Single-writer per account (partitioned by account UUID) to preserve tick ordering.
- **Event log service**: append-only store for interaction events. Write path is hot, read path is the simulation tick and the export/delete flows. Partitioned by account UUID.
- **Auth service**: magic-link issuance, session token minting/revocation, email-change verification.
- *(Supporting, not on the hot path)* **Export service**: generates the JSON snapshot on demand and emails the download link. Runs out-of-band.

A single Postgres cluster holds the canonical state, the event log, and account/auth tables, partitioned/sharded by account UUID. The simulation service is the only writer of the `aviary_state`, `bird`, and `personality_vector` rows. Object storage holds exports and any large blobs (none expected in v1 beyond exports).

### 2.2 Client/server split

The client never owns canonical state. The client:

- Pulls a state snapshot from the API (inline in HTML on first load, then via a lightweight `GET /aviary/snapshot` endpoint on visibility change, on long render-frame gaps, and on a low-frequency keepalive while visible).
- Interpolates between snapshots for smooth motion (perch moves, mood-driven pose changes, day/night palette).
- Submits interaction events (offer, listen-in start/end, settle, presence pings) to the event log via `POST /aviary/events`.
- Renders the scene, synthesizes audio, generates captions and narration, runs reduced-motion and accessibility surfaces.
- Generates ambient rendering ornaments (leaves, feathers, parallax) client-side; these are not in the canonical state.

The server:

- Owns the canonical aviary state and is the only writer.
- Runs the simulation tick on a per-account scheduler whether or not any client is connected.
- Consumes the event log in order and applies additive personality deltas and mood transitions.
- Hosts the notebook generator (server-side, runs on the tick) and the visit-invitation state.

### 2.3 Render pipeline boundary

The render pipeline is entirely client-side and reads only from snapshots. The server has no concept of frame, animation, or pose; it emits positions, moods, call-timing seeds, and active transitions. The client's job is to take those and produce continuous motion. This is the load-bearing split that lets the server run on a slow ~60s tick while the client renders at 60fps without ever owning state.

---

## 3. Data model

All identifiers are synthetic UUIDs (v4). Email is stored once on the `account` row, encrypted at rest, and never appears as a foreign key, partition key, or log field.

### 3.1 Account

```
account
  id              UUID PK
  email_enc       bytes   -- encrypted, the only place email lives
  email_hash      bytes   -- deterministic hash for lookup at magic-link time
  status          enum    -- active | soft_deleted | hard_deleted
  deletion_at     ts      -- set when soft_deleted; null otherwise
  created_at      ts
  visit_notify    bool    -- default false
  settings_json   jsonb   -- accessibility prefs, etc.
```

### 3.2 Session

```
session
  id              UUID PK
  account_id      UUID FK -> account.id
  device_label    text    -- user-facing, e.g. "MacBook Air — Safari"
  token_hash      bytes   -- never store raw token
  created_at      ts
  last_seen_at    ts
  revoked_at      ts?     -- null if active
```

### 3.3 Magic link

```
magic_link
  id              UUID PK
  account_id      UUID FK
  code_hash       bytes
  expires_at      ts      -- issued_at + 15 min
  consumed_at     ts?
  revoked_at      ts?
```

### 3.4 Aviary

```
aviary
  id              UUID PK
  account_id      UUID FK -> account.id  -- unique
  created_at      ts
  bird_count      int     -- denormalized cap check; cap = 7
  next_bird_eligible_at ts -- aviary-age gate
```

### 3.5 Bird

```
bird
  id              UUID PK         -- stable internal id; never reused
  aviary_id       UUID FK
  species_id      text            -- references species pool, not a separate table at v1
  name            text            -- user-assigned, renameable
  perch_zone      enum            -- front | middle | back (current)
  created_at      ts
```

Stable id is invariant across rename, sync, species-pool changes, and any future migration. This is the foundation of drift validity.

### 3.6 Personality vector

```
personality_vector
  bird_id         UUID FK PK
  boldness        float   -- normalized [0,1]
  social_warmth   float
  vocal_frequency float
  plumage_sat     float
  curiosity       float
  updated_at      ts
  version         int     -- monotonic, incremented by each tick apply
```

Five scalar traits per `bird_engine.md`. Stored server-side, canonical, never derived at runtime, never recomputed from event logs, never rebuilt by the client. Never exposed to the user in any form (no debug surface, no toggle, no future tier).

### 3.7 Mood

```
mood
  bird_id         UUID FK PK
  state           enum    -- wary | content | curious | drowsy | alert | settled
  entered_at      ts
  timer_ms        int     -- time in current state, advanced by tick
  seed            int     -- per-bird RNG seed for procedural variation in this mood
```

Mood persists across sessions; the tick advances `timer_ms` and may transition `state`. The client reads `state` and `seed` and renders mood-shaped idle motion; it never owns mood.

### 3.8 Presence

```
presence_window
  id              UUID PK
  account_id      UUID FK
  session_id      UUID FK
  started_at      ts
  ended_at        ts?
  duration_ms     int     -- finalized on end
```

A presence window opens when all three presence conditions become true simultaneously and closes when any one becomes false (or the session ends). The client sends a `presence.start` event when the window opens and a `presence.end` when it closes, plus periodic `presence.ping` events while open (every ~60s) to bound data loss if a tab dies without an unload event.

### 3.9 Interaction event log (append-only)

```
event
  id              UUID PK
  account_id      UUID FK   -- partition key
  session_id      UUID
  bird_id         UUID?     -- target bird if any
  type            enum      -- presence.start | presence.end | presence.ping
                          -- listen_in.start | listen_in.end
                          -- offer.seed | offer.song | offer.pool
                          -- settle | settle.undo
                          -- return_greeting (server-emitted)
  payload         jsonb
  client_ts       ts        -- client wallclock at emission
  server_ts       ts        -- ingest time
```

Append-only, ordered by `id` (UUIDv7 for time-ordered monotonically increasing ids without a separate sequence). The simulation tick consumes events with `server_ts > last_consumed` per account.

### 3.10 Notebook

```
notebook_entry
  id              UUID PK
  account_id      UUID FK
  written_at      ts
  body            text      -- naturalist prose, lowercase, present-tense
  -- no edit/delete columns; read-only by design
```

Entries are rare and sparse; the notebook generator (§5.7) decides when to write. Old entries are never archived or hidden.

### 3.11 Visit invitation

```
invitation
  id              UUID PK
  host_account_id UUID FK
  visitor_email_enc bytes   -- encrypted; visitor may not be a user yet
  visitor_email_hash bytes
  code_hash       bytes      -- the link code
  state           enum       -- outstanding | used | revoked | expired
  created_at      ts
  expires_at      ts         -- created_at + 30 days
  used_at         ts?
  revoked_at      ts?
```

### 3.12 Visit log

```
visit
  id              UUID PK
  invitation_id   UUID FK
  host_account_id UUID FK
  visitor_email_enc bytes
  started_at      ts
  last_seen_at    ts
  ended_at        ts?
  duration_ms     int?
```

### 3.13 Species pool

Static, versioned JSON shipped to the client (part of the bundle / fetched once and cached): six species, each with silhouette id, default plumage palette, motif-library reference, and a per-species mood-modulation bias. Species rarity is not modeled.

### 3.14 Privacy boundary as schema rule

Per-bird interaction events (`event` rows, `personality_vector`, `mood`, `notebook_entry`) live in the simulation database. Aggregate telemetry lives in a separate telemetry pipeline that has no read path to the simulation database. This is enforced at the data-pipeline layer: the analytics warehouse's ingestion jobs do not have credentials for the simulation database. ML training pipelines (if any ever exist) do not have credentials either.

---

## 4. API surface

All endpoints are JSON over HTTPS, session-token authenticated (except magic-link issuance/consumption and visit-link consumption). Tokens are sent as a `Secure; HttpOnly; SameSite=Strict` cookie; CSRF protection via SameSite plus a double-submit token for state-changing requests.

### 4.1 Auth

- `POST /auth/magic-link/request` — body `{email}`. Rate-limited per `email_hash`. Emails a link. Matter-of-fact response regardless of whether the email exists (no user enumeration).
- `GET /auth/magic-link/consume?code=...` — validates code, issues session token, sets cookie, redirects to the aviary. Invalid/expired/used → matter-of-fact error surface.
- `POST /auth/email/change` — body `{new_email}`. Sends verification to new address; old email continues to work until verify.
- `GET /auth/sessions` — lists active sessions for the account.
- `POST /auth/sessions/{id}/revoke`.
- `POST /auth/sign-out` — revokes current session.

### 4.2 State pull

- `GET /aviary/snapshot` — returns the current canonical snapshot for the signed-in account:

```json
{
  "aviary_id": "...",
  "tick_ts": 1718000000,
  "day_phase": "morning",         // morning | midday | evening | night
  "weather": "clear",             // clear | rain | wind
  "birds": [
    {
      "id": "...",
      "name": "pip",
      "species": "warbler_grey",
      "perch_zone": "front",
      "perch_pos": { "x": 0.32, "y": 0.55 },  // normalized scene coords
      "mood": "curious",
      "mood_timer_ms": 182000,
      "call_seed": 4291,           // seed for the next call window
      "next_call_in_ms": 4200,     // hint; client also varies locally
      "active_transition": null    // or {type:"perch_move", from:..., to:..., t:0.4}
    }
  ],
  "notebook_cursor": "..."        // opaque, for paginated scroll-back
}
```

Snapshot is kilobytes, not megabytes. Delivered from a CDN edge when possible; inline in the HTML on first paint.

- `GET /aviary/notebook?cursor=...&limit=20` — paginated notebook scroll-back. Oldest entries reachable indefinitely.

### 4.3 Event submission

- `POST /aviary/events` — body is a single event or a small batch (≤32) of events from the client. Idempotent on `event.id` (UUIDv7) so retries after transport failure don't double-count. Server stamps `server_ts` on ingest. Returns `{ok: true, ingested: [...]}`.

The client submits events as they happen (offer, listen-in start/end, settle, settle.undo, presence.start/end/ping). The client never submits personality or mood values; those endpoints do not exist.

### 4.4 Offer affordance

- `GET /aviary/offer/options` — returns the available offer types and any per-bird cooldown state:

```json
{ "offers": ["seed","song","pool"],
  "cooldowns": [{ "bird_id":"...", "offer_type":"seed", "available_in_ms":0 }] }
```

The cooldown is enforced server-side (the client's view is advisory). Submitting an offer on cooldown is a no-op (the event is recorded but ignored by the tick).

- `POST /aviary/events` with `type:"offer.seed|offer.song|offer.pool"` and `{bird_id}` payload. The reaction is computed by the tick and shows up in the next snapshot; the client does not predict the reaction.

### 4.5 Settle

- `POST /aviary/events` with `type:"settle"` or `type:"settle.undo"`. The client renders the lighting shift locally (the server records the event; the canonical day_phase may also shift to `evening` in the next tick).

### 4.6 Visit flow

- `POST /visits/invite` — body `{visitor_email}`. Creates an `invitation`, emails the visitor a one-time link. Host-authenticated.
- `GET /visits/invitations` — lists outstanding and past invitations for the host.
- `POST /visits/invitations/{id}/revoke` — host revokes; takes effect immediately at the visitor's next snapshot pull.
- `GET /visits/enter?code=...` — visitor follows link. Validates code; if valid, issues a visit-scoped read-only session token (no account needed; visitor may not be a Pocket Aviary user). Sets a visit cookie. Invalid/expired/revoked → matter-of-fact "visit no longer available" surface.
- `GET /visits/snapshot` — visitor's read-only snapshot pull. Same shape as `/aviary/snapshot` but bound to the host's aviary. Returns 403 with the matter-of-fact surface if the invitation is revoked or expired.
- The visitor's session token has no access to `POST /aviary/events`, `POST /visits/invite`, or any account-mutating endpoint. The simulation service ignores visit-session traffic for presence and drift entirely.
- `GET /visits/log` — host's visit log: list of `{visitor_email, started_at, duration_ms, ended_at}` ordered by most recent.
- `POST /account/settings/visit-notifications` — toggles `account.visit_notify`.

### 4.7 Account settings & data

- `GET /account` — account summary (email masked, settings, active sessions, deletion status).
- `POST /account/settings` — accessibility prefs, etc.
- `POST /account/export` — queues an export; the JSON snapshot is emailed to the verified address as a download link.
- `POST /account/delete` — sets `status = soft_deleted`, `deletion_at = now + 30 days`. Recoverable by signing in during the window. After 30 days, a hard-delete job purges all rows tied to the account UUID across all tables.

### 4.8 Error surface voice

All auth/account/sync/visit-revocation errors drop into matter-of-fact tone, per the named exception in the PRD. Examples are exactly as the PRD wrote them. The error response body is a structured object `{code, message}` where `message` is the matter-of-fact copy; the client renders it without adornment.

---

## 5. Simulation engine design

### 5.1 Tick scheduler

- Per-account tick at 60s cadence (calibration target, may move to 45–90s).
- Tick runs whether or not any client is connected. This is the implementation of "the aviary continues without the viewer."
- Scheduler partitions accounts across worker processes by `account_id` hash; one worker owns an account at a time so tick order is preserved per account.
- Each tick is idempotent on `(account_id, tick_ts)`: a retried tick with the same timestamp applies no deltas twice. Implemented via a `tick_apply` ledger row.

### 5.2 Tick phases (in order, per account)

1. **Ingest events**: pull events with `server_ts > last_consumed_ts` for this account, in id order.
2. **Presence reconcile**: collapse presence.start/ping/end events into presence-time deltas since last tick. Drop incomplete windows whose start has no matching end (treat the last ping as the end at its `server_ts`).
3. **Personality drift apply**: compute additive deltas (§5.3) and apply to `personality_vector`. Clamp to [0,1]. Bump `version`.
4. **Mood transition**: for each bird, evaluate mood transition probabilities (§5.4) using current mood, mood_timer, personality, time-of-day in the account's timezone, and active ambient events. Apply transitions; reset `timer_ms` on transition.
5. **Ambient event update**: advance weather (rare, a few times a week for rain; occasional wind), update day_phase from the account's local timezone (morning/midday/evening/night).
6. **Bird-to-bird interaction**: process calls queued from the prior tick — a wary mood can spread to neighbors; high-vocal-frequency birds in the same call window trigger a chorus event.
7. **Perch re-selection**: each bird may move perch based on mood + personality (bold→front, wary→back). Emit an `active_transition` so the client can interpolate.
8. **Notebook generation**: evaluate notebook generator (§5.7) and write 0 or 1 entries.
9. **Return-greeting precompute**: nothing — return-greeting is computed at snapshot time, not at tick time, because it depends on absence length which is known only when the client pulls.
10. **Write canonical state**: commit the new `aviary_state`, `personality_vector`, `mood`, any new `notebook_entry`, and the `tick_apply` ledger row in one transaction.

### 5.3 Drift function

Drift is a low-pass filter over presence-and-interaction signals. Per trait, per tick:

```
delta_trait = clamp01( k_presence * presence_time_scaled
                     + k_listen   * listen_in_time_for_bird
                     + k_offer    * offer_count_for_bird
                     + k_offer_boldness * offers_near_bird )
              -- all coefficients >= 0; no negative terms
new_trait = clamp01( trait + delta_trait )
```

Notes:

- All terms are non-negative. Drift is monotonic toward expressive, full stop. There is no negative drift on neglect, anywhere in the function.
- `presence_time_scaled` is the presence-time accumulated for this account during this tick window, scaled so that a week of regular visits (~30–60 minutes/day of true presence) produces the calibration target: measurable drift in instruments after ~7 days, visible drift to the user after ~21 days. The coefficients are seeded with values derived from a calibration simulation and tuned in build via the instrument harness (§5.8).
- `listen_in` is per-bird: time spent focused on this bird during the tick window. Affects `social_warmth` and `vocal_frequency`.
- `offer` affects `curiosity` (accepting) and `boldness` (offering near the bird at all).
- `plumage_sat` drifts up with sustained presence; never down.
- The per-trait low-pass time constant is roughly 1 week (i.e., a single session's contribution is small relative to the accumulated value), which is what makes "no single session shifts a trait visibly" true.
- Per-bird offer cooldown (a few minutes) prevents curiosity-trait drift saturation within a session.

### 5.4 Mood transitions

Mood is a small enumerated state machine per bird. Transitions are probabilistic per tick, biased by:

- Current mood + `mood_timer_ms` (long time in one state raises the probability of leaving it).
- Recent interactions in the current tick window (offer accepted → bias toward `content`).
- Time of day in the account's timezone (early morning → `alert`; dusk → `drowsy`; night → `settled`).
- Active ambient events (rain → dampen vocal frequency and bias toward `content`/`drowsy`; wind → split: some birds `alert`, some `wary`).
- Personality vector (high `boldness` → lower probability of entering `wary` on the same input; high `curiosity` → higher probability of `curious` after an offer).
- Bird-to-bird: a neighbor's `wary` raises the probability of `wary`; a chorus event biases toward `content`/`alert`.

Mood persists across sessions and across the user's absence. The tick advances mood during absence based on time-of-day and ambient events alone (no interaction inputs, no presence). A bird that ended yesterday's session in `drowsy` and was at dusk on the server will likely be `settled` by morning.

### 5.5 Call-grammar runtime

- Each species has a motif library: a small set of motifs (short note sequences with pitch, duration, vibrato, articulation parameters). Motifs are described as parameter envelopes, not audio.
- A call is generated by selecting 1–N motifs, varying their parameters by a per-bird, per-mood seed, and concatenating with timing jitter. The per-bird seed is stable across moods so the bird's call signature stays recognizable.
- The call-grammar runtime runs client-side (§8). The server's job is to emit `call_seed` and `next_call_in_ms` hints so the client and any other client (multi-device) are roughly synchronized, but the actual synthesis is client-local.
- Vocal frequency trait drives call frequency: high-vocal-frequency birds call more often when unobserved and join choruses more readily.
- Recognizability is the cross-mood invariant. The motif library and the per-bird seed are designed so that a user who has spent two weeks with Pip knows Pip's call by ear across moods and across vocal-frequency drift.

### 5.6 Return-greeting

Computed at snapshot time when the client signals a fresh session (or a visibility transition from hidden→visible after a long absence):

- Determine absence length = `now - last_session_end_at` (from presence windows).
- Pick the greeter bird: weighted by `boldness` (bolder greets first), filtered by current mood (a `wary` bird may not greet today; a `settled` bird at night won't).
- Greeting form depends on absence length + boldness + mood:
  - Short absence (minutes): a glance up from preening.
  - Medium (hours): a quiet two-note call, head-tilt.
  - Long (days): a longer call, a step toward the front perch, possibly a second bird's staggered response.
- The greeting is procedurally varied within those rules using the bird's per-mood seed, so it is never identical twice.
- If multiple birds would greet, stagger them by a randomized small offset (200–900ms), never in unison.
- The greeting is emitted to the client via the snapshot's `active_transition` field and the audio pipeline; the server also writes a `return_greeting` event to the log for notebook-generation context.

### 5.7 Notebook generator

Runs on each tick, decides whether to write 0 or 1 entry. Target sparsity: roughly one entry every few days for a regularly-visited aviary, more often when something noteworthy happens, never one-per-session.

Generation logic:

- Maintain a small set of "observation templates" — naturalist sentence patterns parameterized by bird name, species, mood, perch, weather, time of day, and small event markers (greeted-first, long quiet, preened-without-looking-up, fluffed-against-cool).
- Each tick, score candidate observations by noteworthiness. Noteworthy = a first-of-this-week event (Pip greeted before Wren today, first time this week), a salient mood shift, an ambient event (a passing rain), or a long quiet stretch.
- Apply a sparsity throttle: if the last entry was within N hours and nothing is sufficiently noteworthy, write nothing. N is calibrated to land at the target rate.
- Prose is lowercase, present-tense, specific. It names birds, not states. It describes the aviary, not the user's behavior (the line from `interactions.md`: observations of the aviary, never of the user's visit pattern).
- Entries are immutable once written. No edit, no delete, no annotation.

### 5.8 Drift calibration harness

A build-time instrument harness that runs a synthetic population of accounts through scripted presence/interaction patterns and checks:

- After 7 days of regular visits, personality vector deltas are above the instrument detection threshold (measurable drift).
- After 21 days, deltas are above the user-visible threshold (visible drift).
- After a single 30-minute session, deltas are below the user-visible threshold (no single-session shift).
- After a 14-day absence, no trait moves down (monotonic-toward-expressive invariant holds; absence produces ambient quietness via mood, not via negative drift).
- Offer-cooldown enforcement prevents curiosity saturation within a session.

The harness is run in CI on every change to the drift function or its coefficients. A regression here is a regression of the product's central claim, not a perf nit.

---

## 6. Sync model

### 6.1 One canonical record

There is exactly one canonical aviary state per account, server-side. The simulation service is the only writer. Clients read snapshots; clients never write state.

### 6.2 Conflict prevention

There are no client-to-client sync paths and no client-owned state to merge. The two clients (laptop, phone) both pull snapshots from the same canonical record. Whichever pulled most recently renders the most recent state; there is no reconciliation to do because the client never holds a divergent truth.

### 6.3 No last-write-wins on personality

Personality drift is additive, server-authored deltas computed from the event log in order. The client never sends an absolute trait value (the endpoint doesn't exist). The tick consumes events in `id` order (UUIDv7 = time-monotonic) and applies deltas to the existing vector. This makes the "laptop morning session overwrites phone lunch session" failure mode unreachable.

### 6.4 Snapshot freshness

- Inline in HTML on first paint (edge-cached, per-account, short TTL).
- `GET /aviary/snapshot` on:
  - Visibility transition hidden→visible (the user came back to the tab).
  - Long render-frame gap (the laptop was suspended; on resume, the client detects a large `requestAnimationFrame` delta and pulls).
  - Low-frequency keepalive while visible (every ~2 minutes; cheap because the payload is small).
- The client interpolates between the last snapshot and the new one for perch moves and palette shifts so motion stays smooth across snapshot boundaries.

### 6.5 Event ordering and idempotency

- Events are UUIDv7, so they are globally time-monotonic without a central sequence.
- `POST /aviary/events` is idempotent on `event.id`: the server dedupes on ingest. Retries after transport failure don't double-count.
- The tick consumes events with `server_ts > last_consumed`; a `tick_apply` ledger row records the high-water mark per account.

### 6.6 Sync error surface

In rare cases (magic-link replay, in-flight session timeout mid-write, server outage), the client shows a matter-of-fact error surface. The PRD's exact copy is used. The client does not retry writes automatically beyond a bounded backoff (≤3 attempts); persistent failures surface to the user.

---

## 7. Frontend rendering pipeline

### 7.1 Stack and budget shape

- Framework: a small, fast renderer (the plan does not mandate a specific framework; the team picks one that supports streaming SSR of the inline snapshot and a 60fps idle loop within the <2MB bundle). Aggressive code-splitting for account settings, accessibility settings, and the visit flow.
- No entry animation, no spinner. The first frame has birds mid-action. If the snapshot is slow to arrive, the loading state is a quiet field (soft sky color, a faint motion cue) — not a spinner.
- Empty-aviary state (between adoption and first bird) is the same quiet field; the first bird enters with a soft fly-in to its starting perch.

### 7.2 Scene composition

- One horizontal scene, no pan/scroll/zoom. Aspect preserved across viewport sizes; on a narrow phone the scene compresses horizontally without cropping any bird out of frame; on wide desktop it widens with more space between perches.
- Three perch zones (front/middle/back) on a middle plane. Soft background foliage and sky behind. Occasional foreground branch/leaf passes through. Parallax is subtle, not layered-illustration-heavy.
- Calm palette: soft blues, greens, warm browns, muted ochres. No saturated UI accents inside the scene.
- Day/night follows the account's local timezone, computed from the snapshot's `day_phase` and the client's wallclock. Evening warms the palette and quiets calls; night dims the scene; the nightjar-like species stays active into late hours.

### 7.3 Idle micro-motion

- Continuous, regardless of user attention. The client may stop rendering when the tab is hidden (saves battery); the simulation continues server-side regardless.
- Mood-shaped: wary → perches further back, scans more; content → preens; curious → tilts toward sounds, watches passing leaves; drowsy → sits low, feathers fluffed; settled → eyes closed, low on perch; alert → upright, head up.
- The motion is the visible surface of mood. The user reads mood from motion; no label, tooltip, or status icon.

### 7.4 Transitions

- Perch moves interpolate smoothly between snapshot positions; the server's `active_transition` field tells the client the target and progress so the client can continue the motion without teleporting.
- Day→evening and evening→night palette shifts are slow continuous ramps driven by the client's wallclock and the snapshot's `day_phase`.
- Settle: lighting shifts to evening over a few seconds; calls quiet. Settle undo (any click within 5s) reverses the shift.

### 7.5 Reduced-motion mode

Reduced-motion is a designed surface, not "animations off." Triggered by `prefers-reduced-motion` or an accessibility-settings toggle.

- Micro-motion is replaced by slow cross-fades between still poses. A preening bird cross-fades through preen-poses instead of frame-by-frame animation.
- Flight transitions cross-fade between perches rather than animating paths.
- Ambient leaf drift is removed. Day→evening color shifts remain, slowed.
- Calls still play at full quality (or caption, per audio settings). Birds still drift. Mood still changes. The notebook still notices things. The aviary is still the aviary.

### 7.6 Top bar and chrome

- Thin top bar above the scene with: account/settings, accessibility settings, field notebook, offer affordance. Nothing else.
- After a few seconds of cursor stillness the top bar fades nearly to transparent; returns to full opacity on cursor movement or keyboard activity.
- No UI chrome inside the aviary scene (no buttons, badges, hover-tooltips, overlay icons, inline labels).
- No "Welcome back" toast/banner/modal anywhere.

### 7.7 Listen-in rendering

- Click/tap/keyboard-focus a bird → its call rises in the audio mix; others quiet to ambient (never silent). Slow ramp on engage and disengage (§8.3).
- Disengage: focus the bird again, focus a different bird, click empty space, or move keyboard focus away.

### 7.8 Offer rendering

- Offer affordance is in the top bar, not on birds. Opens a small keyboard-navigable panel.
- On submit, the event goes to the server; the reaction is computed by the tick and arrives in the next snapshot. The client does not predict the reaction (a wary bird's "wait and eventually come near" is a tick-time behavior, not a client animation).

### 7.9 Responsive and viewport rules

- All birds always visible; never crop a bird out, never let one drift offscreen.
- Min and max viewport handling live in the rendering spec; the invariant the plan enforces is "all birds visible at all times."

---

## 8. Audio pipeline

### 8.1 Procedural synthesis

- WebAudio. Each call is synthesized from the species' motif library at runtime.
- A call is: pick 1–N motifs, vary their parameters (pitch, duration, vibrato, articulation) by the bird's per-mood seed, schedule with timing jitter on a `AudioContext` timeline.
- Per-bird recognizability is the cross-mood invariant: the per-bird seed and the motif set are stable enough that a user knows the call by ear across moods and across vocal-frequency drift.
- Vocal frequency trait drives call frequency (high → calls more often when unobserved, joins chorus more readily).

### 8.2 Chorus mixing

- Each bird has a gain node. The mix is computed from: ambient base level, the bird's current mood, the bird's vocal frequency trait, and whether a chorus event is active (multiple high-vocal-frequency birds calling in the same window).
- Two birds calling at once produce a real chorus via independent synthesis on the same timeline, not stacked audio loops. This avoids the phase-canceling artifact of layered recordings.

### 8.3 Listen-in mix decay

- On engage: the focused bird's gain ramps up over ~800ms; the others ramp down to an ambient floor (never zero) over the same window.
- On disengage: all gains ramp back to ambient over ~800ms.
- A hard cut is forbidden; it would convert the aviary into a UI of soloable tracks.

### 8.4 WebAudio fallback

- If WebAudio is unavailable (old browser, audio context permission denied, hardware issue), the aviary plays in graceful silence with captions on by default.
- No recorded-audio fallback path. The "no recorded audio" rule is unconditional.
- The audio context is created/resumed on first user gesture (browser autoplay policy); until then the aviary renders visually and the captions surface shows what would be playing.

### 8.5 Caption generation

- Captions are generated client-side from the same call-grammar parameters that produced the audio: a short prose description of what the call sounds like in the bird's current mood. Examples from the PRD: "a soft three-note rise," "a low trill, paused, low trill again," "a single sharp call from the back perch."
- Captions appear as small text near the calling bird, fading in and out with the call. Same naturalist voice as the rest of the product.

### 8.6 Memory discipline

- Audio buffers are reused; no per-call allocation that isn't freed.
- The audio context and worker threads are bounded. No memory growth over a 30-minute session (enforced in CI).

---

## 9. Accessibility surfaces

Accessibility ships with v1, not after. A reduced-motion mode that lands post-launch is a v1 launch that quietly told reduced-motion users the product wasn't for them.

### 9.1 Screen-reader narration

- Running naturalist prose, not a state list. Same voice as the field notebook.
- Cadence: one prose update per 30–60s at idle; faster only on user-initiated events (a successful offer, a settle gesture, a return-greeting). User-initiated events get a small priority bump in the screen reader's queue.
- Generated client-side from the same snapshot the visual reads, expressed as observations, not state transitions. ("a warbler perches on the high branch, calling softly," not "warbler perched at high branch.")
- Narration when displayed visually (e.g., a visible captions-like surface) passes WCAG AA contrast.

### 9.2 Reduced-motion mode

Per §7.5. A designed cross-fade surface, not a fallback. Birds still drift; mood still changes; calls still play; the notebook still notices things.

### 9.3 Call captioning

Per §8.5. Opt-in from accessibility settings; on by default in the WebAudio-fallback silence path.

### 9.4 Keyboard navigation

- Tab moves through top-bar items.
- Tab into the aviary scene focuses the first bird.
- Arrow keys move focus between birds.
- Enter triggers listen-in on the focused bird.
- Escape exits listen-in.
- The offer affordance opens with a top-bar shortcut and is fully keyboard-navigable.
- Settle is reachable from the top bar.
- Focus indicators are visible against both bright and dim aviary states (soft, high-contrast outline; exact treatment per visual designer).

### 9.5 Contrast

- All user-copy text (top bar labels, settings, account surfaces, error surfaces, captions, narration when displayed visually) passes WCAG AA.
- The aviary scene itself carries no user copy except in the top bar, so the constraint applies primarily to chrome.

---

## 10. Performance budgets and observability

### 10.1 Budgets

- **Initial JS bundle**: <2MB gzipped at first paint. Code-split aggressively for account/accessibility/visit surfaces. Procedural audio (no recorded audio), procedural visual assets where possible, small SVGs / compact bitmaps otherwise.
- **Time to first bird visible**: <500ms on a mid-tier mobile device over 4G. Requires the bundle budget, inline snapshot delivery from a CDN edge with the HTML, and a render path that doesn't wait for non-critical assets before drawing the first bird.
- **60fps idle motion** on a five-year-old mid-range laptop, for a 30-minute session, not just the first minute.
- **No memory growth over a 30-minute session**. Audio buffers reused; notebook entries scrolled into view don't retain references after scroll-out; worker threads and audio contexts bounded. This is a real CI test, not a guideline.

### 10.2 Observability

- **Synthetic performance checks**: a fleet of automated browsers runs the aviary on a schedule from common geographies, capturing page-load timings, first-bird-render timings, render-frame timings, audio-context errors, simulation-tick latencies.
- **Aggregate-only Real User Monitoring**: page load timings, first-bird-render timings, render-frame timings, audio-context errors, simulation-tick latencies. Anonymized, no per-account dimension.
- **Error budget**: simulation-tick latency p99 alarms if it exceeds 5 seconds. Catches degradation before users notice the aviary "running slow."
- **Privacy boundary in telemetry**: per-bird state and per-account interaction history never enter telemetry. The metric definitions are reviewed against the boundary; the analytics warehouse's ingestion jobs do not have credentials for the simulation database.

### 10.3 What we deliberately don't measure

- Per-bird drift trajectories (would aggregate the user's relationship with their birds).
- Per-account visit frequency (would be the data substrate of the streak counter we refuse to build).
- Anything that could be used to reconstruct a user's relationship with their aviary.
- We do not compute "most-visited aviaries," "longest-running aviary," "most birds," or any underlying metric for those purposes, even silently — the absence of the metric makes the feature's reappearance harder.

### 10.4 Browser support

Last two major versions of Chrome, Safari, Firefox, Edge. Older browsers get a matter-of-fact unsupported-browser surface. No compatibility paths for very old browsers (the bundle bloat isn't justified).

---

## 11. Rollout

### 11.1 Ship v1

- Single environment progression: internal dogfood → private beta (cohorts of invited users) → general availability.
- Dogfood runs at least 4 weeks to give the drift calibration harness real-data validation (the 7-day measurable / 21-day visible targets).
- Feature flags: one flag per major surface (audio, narration, reduced-motion, visits, notebook) so a regression in one surface can be disabled without disabling the product. Flags default ON for GA.

### 11.2 Ramp birds-per-aviary

- v1 ships with a hard cap of 7 and a starting count of 2. The cap is enforced in the data model and in the simulation service.
- New birds are gated on aviary age, not on visit count or interaction score: third at 60 days, fourth at 120, fifth at 240, sixth at 365, seventh at 540 (tunable; the pacing is meant to match a deepening relationship, not reward engagement).
- The age gate is computed at tick time and surfaced as a "a new bird is here" moment in the naturalist voice, not as a counter or a reward.

### 11.3 Instrument from day one

- Aggregate RUM and synthetic checks (§10.2) are on from the start of private beta.
- The drift calibration harness (§5.8) runs in CI from day one; a regression blocks merge.
- The "no memory growth over 30 minutes" CI test is on from day one.
- The privacy boundary is reviewed as part of code review for any new telemetry or analytics change: a new metric must show it does not touch per-bird or per-account interaction state.

### 11.4 What we instrument from day one but don't surface

- Simulation-tick latency (p99 alarm at 5s).
- Audio-context errors (rate of fallback-to-silence).
- First-bird-render timings (the <500ms budget).
- Notebook generation rate (to confirm the sparsity target).
- Drift-coefficient calibration metrics (instrument-only; not user-facing).

---

## 12. Risks

### 12.1 Drift calibration

- **Risk**: coefficients land too fast (Tamagotchi) or too slow (screensaver). The product lives in the narrow band between.
- **Mitigation**: the drift calibration harness (§5.8) is a CI test against the 7-day / 21-day / single-session / absence invariants. Coefficients are seeded from a synthetic-population simulation and tuned in dogfood. The monotonic-toward-expressive invariant is asserted per tick.

### 12.2 Sync correctness

- **Risk**: a divergence between laptop and phone views, or a silent loss of drift recorded on one device.
- **Mitigation**: the server is the only writer; clients never own state; personality deltas are additive and server-authored; events are UUIDv7 + idempotent on `event.id`. The "laptop overwrites phone" failure mode is unreachable by construction, not by policy.

### 12.3 Audio uncanniness

- **Risk**: procedural calls feel canned; chorus phase-cancels; recognizability collapses across moods; the WebAudio fallback is jarring.
- **Mitigation**: procedural synthesis is the only path; per-bird seeds preserve cross-mood recognizability; the chorus is independent synthesis on a shared timeline (not layered loops); the fallback is silence + captions (not canned audio). A/B testing in dogfood checks recognizability (can a listener identify Pip vs. Wren by ear after two weeks?).

### 12.4 Accessibility regressions

- **Risk**: narration reads as a state list; reduced-motion is a stripped fallback; captions are stock strings; keyboard nav has trap states.
- **Mitigation**: accessibility ships with v1 (not after); the narration generator is reviewed against the "observation, not state transition" rule; reduced-motion is a designed cross-fade surface with its own aesthetic; captions are generated from the call-grammar parameters at runtime; keyboard nav is in the acceptance criteria for every interactive surface.

### 12.5 Presence signal integrity

- **Risk**: a laxer presence definition silently inflates drift across the population.
- **Mitigation**: the conjunction of three signals (visibility + focus + recent activity) is asserted client-side before any `presence.start` event is sent; the server rejects presence events whose `client_ts` doesn't reconcile with the session's reported visibility/focus state where available. The activity window (180s start) is calibrated in dogfood with two cohorts.

### 12.6 Privacy boundary erosion

- **Risk**: a well-meaning analytics change quietly pulls per-bird state into an aggregate dashboard.
- **Mitigation**: the simulation database is not readable by the analytics warehouse at the credentials layer. New metrics must show in review that they don't touch per-bird or per-account interaction state. The "we don't even compute the underlying stats" rule for leaderboards is enforced by absence of the query, not by access control on its result.

### 12.7 The "just one harmless engagement feature" temptation

- **Risk**: a streak counter, a green-dot calendar, a "birds adopted: 2" badge, a "you've been here every day this week" notebook entry slips in.
- **Mitigation**: the rule is absolute and named in `non_goals.md`, `interactions.md`, and `product_brief.md`. The notebook generator's templates are reviewed against the "observations of the aviary, never of the user's behavior" line. The plan treats any addition in this class as a regression of the product's identity, not a feature addition.

### 12.8 Personality vector exposure

- **Risk**: a debug surface, a stats panel, a "show me how my bird is doing" view, a future-tier toggle leaks the numbers.
- **Mitigation**: the personality vector is never serialized into any client-facing response. The `/aviary/snapshot` response includes mood and perch, not traits. The rule is enforced at the serialization layer (the response builder doesn't have access to the trait values), not just at the UI layer.

### 12.9 Identity continuity

- **Risk**: a migration, a sync bug, or a species-pool change swaps a bird out from under the user and invalidates drift retroactively.
- **Mitigation**: the `bird.id` is invariant; rename, sync, and species-pool changes never replace a bird. The simulation service treats `bird.id` as the permanent key. Any migration that would replace a bird is a regression of the product's central promise.

### 12.10 First-frame continuity

- **Risk**: a planner reaches for a spinner-then-fade-in because it's the safe pattern.
- **Mitigation**: the first frame has birds mid-action, full stop. The loading state is a quiet field, not a spinner. The inline snapshot in the HTML is what makes the <500ms budget achievable; the render path doesn't wait for non-critical assets before drawing the first bird.

---

## 13. Open calibrations (consolidated)

| Item | Starting value | Calibration loop |
|---|---|---|
| Tick cadence | 60s | p99 tick latency <5s; felt-continuity in dogfood |
| Presence activity window | 180s | two-cohort dogfood (active vs. passive watchers) |
| Mood state set | wary, content, curious, drowsy, alert, settled | dogfood review; `settled` added for settle/night |
| Drift coefficients | seeded from synthetic-pop sim | drift calibration harness in CI; 7-day / 21-day targets |
| New-bird pacing | 60/120/240/365/540 days | dogfood review; aviary-age only, never engagement |
| Notebook sparsity | one entry every few days | notebook generation rate metric; sparsity throttle N |
| Offer cooldown | 180s per bird per offer type | curiosity-saturation test in drift harness |
| Listen-in ramp | 800ms engage / 800ms disengage | dogfood review; "listening, not switching channels" |
| Top-bar fade delay | 3s of cursor stillness | dogfood review |

---

End of plan.
