# Pocket Aviary — v1 implementation plan

| | |
|---|---|
| slot | `runs/wave_002/plans/001` |
| source | `prd/1-START_HERE.md` and the nine PRD files it lists |
| planner | claude-fable-5.1 via claude-code, effort extra-high |
| status | executable plan for an engineering team; no product code written |

## 0. How to read this plan

- §1–§13 are the plan. §14 is the single table of calibration constants that every other section refers to; a number quoted elsewhere is a copy and §14 wins if they disagree. §15 logs every call made where the PRD was silent, ambiguous, or internally inconsistent, with reasoning. §16 is the definition of done.
- Vocabulary follows `concepts.md` exactly: bird, aviary, call, mood, personality vector, drift, presence, listen-in, offer, settle, field notebook, visit, tick. Internal terms this plan introduces (attunement, expression band, performer, call plan, reaction plan, quiet field) are defined at first use and never appear on a user-facing surface.
- Product-surface strings are lowercase naturalist prose. System-surface strings (sign-in, account settings, accessibility settings, sync/visit errors, unsupported-browser page) are matter-of-fact English. The split is enforced by a voice linter in CI (§2.5), not by reviewer memory.
- The two PRD "traps" that most often get compressed into the wrong product are named up front so no section quietly regresses them: (1) the first frame is the aviary already in motion, never a spinner-then-fade; (2) nothing on any surface announces the user, counts their visits, or shows a trait number.

---

## 1. Scope

### 1.1 In v1

| Area | What ships | Spec source |
|---|---|---|
| Aviary | one horizontal, non-scrolling scene; three perch zones; day/night from local time; rare rain and wind; ambient leaf/feather drift; sparse top bar that fades | `aviary_layout.md` |
| Birds | 2 starters at adoption, cap 7; six-species pool incl. one night-active species; user-named, renameable; stable identity forever | `bird_engine.md` |
| Engine | server-side tick (~60 s); hidden 5-trait personality vector; monotonic drift from presence, listen-in, offers; mood layer with persistence; procedural call grammar; bird-to-bird responses and chorus; age-gated newcomers (birds 3–7) | `bird_engine.md`, `accounts_sync.md` |
| Interactions | return-greeting; sit-and-watch presence; listen-in; offer (seed, song fragment, still pool); settle with 5 s undo; read-only field notebook | `interactions.md` |
| Accounts | email magic link; per-device revocable sessions; email change with verification; JSON export by emailed link; soft delete 30 d then hard delete; synthetic UUIDs everywhere | `accounts_sync.md` |
| Sync | single canonical aviary per account; multi-device by construction; clients pull snapshots and append events; no last-write-wins | `accounts_sync.md` |
| Social | invite-by-email read-only visits; revocable; 30 d invite expiry; silent visit log; opt-in visit notification toggle (off) | `social_optional.md` |
| Accessibility | running naturalist screen-reader narration; designed reduced-motion mode; runtime call captions; full keyboard model; WCAG AA on all user copy | `accessibility_perf.md` |
| Performance | ≤2 MB gz initial JS; first bird ≤500 ms on mid-tier mobile/4G; 60 fps idle on a five-year-old laptop; zero memory growth over 30 min (CI test); WebAudio-only audio with silence+captions fallback | `accessibility_perf.md` |
| Browsers | last two majors of Chrome, Safari, Firefox, Edge; matter-of-fact unsupported page otherwise | `accessibility_perf.md` |

### 1.2 Not in v1 (and, per `non_goals.md`, mostly not ever)

Native apps; any gamification (streaks, scores, badges, levels, counters, calendars, milestone celebrations); Tamagotchi mechanics (hunger, death, distress, decaying meters); social-network surfaces (profiles, follows, feeds, discovery, comments, co-presence, leaderboards); push/email notifications about the aviary (the one named exception is the opt-in visit notice); shared or multiple aviaries; customizable scenes; payments; any surface that shows personality numbers; recorded audio; SSO/passwords; localization beyond English (§15).

### 1.3 Guardrails that live in code, not policy

These are the places where the PRD says "this will be violated by a well-meaning contributor." Each gets a mechanical check.

1. **No announcement primitives.** The design system ships no `Toast`, `Banner`, `Badge`, `Confetti`, or notification-dot component. A lint rule fails the build on those identifiers and on `aria-live="assertive"` outside the narration composer.
2. **Voice linter** (`packages/prose/lint`) runs over every string in product-surface bundles: rejects capitalized sentence starts, `!`, second person ("you", "your", "welcome", "back"), and a denylist ("streak", "day(s) in a row", "achievement", "level", "score", "unlock", "reward"). System-surface bundles are exempt by path.
3. **Trait containment.** The five trait columns are readable only by the `sim` database role. The API role's view of `birds` excludes them. The snapshot builder emits quantized expression bands (§4.9) and a unit test asserts no float trait appears in any API response fixture.
4. **Monotonic drift** is a database trigger: an `UPDATE` that lowers any trait fails unless the session sets `aviary.engine_migration = on`, which only a reviewed migration job can do.
5. **Bird identity** is immutable: `birds.id` has no update grant; there is no delete grant on `birds` except through the account hard-delete job.
6. **Presence honesty** is an end-to-end test that drives a real browser through the eight combinations of visible/focused/active and asserts presence events only appear for the one true conjunction.
7. **Telemetry allowlist.** Metric names and label keys are enumerated in one file; the exporter drops anything else. No metric or trace may carry `aviary_id`, `bird_id`, mood, trait, or email. Logs pass through a field allowlist (§10.7).
8. **PR template** carries a four-line checklist: no announcement surface, no user-behavior observation, no trait exposure, voice register checked.

---

## 2. Architecture

### 2.1 System shape

```
 browser (laptop, phone, or visitor)
 ├─ GET /            ──► edge worker: HTML shell + critical CSS + inline latest snapshot (from KV)
 ├─ /v1/*            ──► api: auth, open, snapshot, events (+fast-path effects), notebook,
 │                        birds, settings, export, deletion, invites, visits
 └─ client owns: rendering, interpolation, idle micro-motion, call synthesis, narration,
                 captions, presence detection, listen-in mix, day/night lighting

 api ──► postgres (sim db)        accounts, sessions, aviaries, birds, events, ticks, snapshots,
     │                            notebook, invites, visits, drift ledger
     ├─► redis                    per-aviary lock + fencing, tick schedule (zset), rate limits
     ├─► mailer                   magic links, invites, export links, opt-in visit notices
     └─► @aviary/engine           same pure engine code the sim workers run (fast-path effects)

 sim workers ──► every 60 s per active aviary; every 15 min (in 60 s sub-steps) per dormant aviary
             ──► postgres canonical write (single writer under lock) ──► KV snapshot replica

 telemetry ──► OTel from edge/api/sim + aggregate RUM ──► metrics backend
              no network path, credential, or code path from sim db to analytics
```

### 2.2 Services

| Service | Responsibility | Runtime | Scale notes |
|---|---|---|---|
| `edge` | serves HTML shell; verifies short-lived signed edge-hint cookie; inlines KV snapshot; serves static assets with immutable caching | edge worker (Cloudflare Workers + KV or equivalent) | stateless; KV read per page view |
| `api` | HTTP JSON API; session auth; event ingestion with fast-path effects; snapshot reads; notebook; account, invite, visit management | Node 22 / TypeScript, Fastify | stateless; horizontally scaled; per-aviary serialization via Redis lock with fencing token |
| `sim` | tick scheduler + workers; catch-up on open; notebook generation; newcomer scheduling; snapshot publish to KV | Node 22 / TypeScript | partitioned by aviary UUID range; work-stealing from a Redis zset keyed by `next_tick_at` |
| `mailer` | transactional email via provider; templates in matter-of-fact voice; bounce handling | Node 22 | queue-backed, idempotent by message key |
| `jobs` | soft→hard deletion, export generation, invite expiry, session pruning, dormant/active cadence demotion | Node 22 cron | idempotent, single-flight per job |
| `postgres` | canonical store for everything per-account | Postgres 16, primary + replica | one logical DB; `sim` and `api` are separate roles with different grants |
| `redis` | locks, schedule, rate limits, magic-link throttles | Redis 7 | no durable data |
| object store | export files (encrypted, 7-day TTL) | S3-compatible | |

### 2.3 Client/server split: who owns what

| Concern | Canonical owner | Client role |
|---|---|---|
| personality vector | server, tick only | never receives raw values; receives expression bands |
| mood, mood timers | server (tick + fast-path) | displays; blends idle mix on change |
| perch zone/slot | server | animates transit |
| call plan (which motif, when, in response to whom) | server, 120 s horizon | synthesizes; adds sub-perceptual jitter; extends locally with a seeded rule if a tick is late |
| return-greeting selection | server on `open`; client fallback from inline snapshot if `open` is slow | executes; reports `greeting_shown` when locally chosen |
| offer reactions | server fast-path (reaction plan) | executes |
| presence | client detects the three-way conjunction; server unions intervals, caps, and accounts | heartbeat every 30 s |
| listen-in | client mix change immediately; server records start/end for drift | |
| settle | client lighting immediately; server records state and ends presence | |
| day/night lighting | client from local clock | server uses the same function with the account timezone for mood priors |
| weather | server schedule | renders and applies audio effects |
| notebook entries | server | renders read-only |
| narration and captions | client (`@aviary/prose`) from snapshot + audio descriptors | |
| idle micro-motion, leaves, feathers | client, seeded, non-canonical | |

Rule of thumb for any new state: if two devices or a visitor must show the same thing, it crosses the wire in the snapshot; if it may legitimately differ (micro-motion phase, leaf timing, jitter), it is client-local and seeded from `local:` namespace RNG.

### 2.4 Render pipeline boundary

```
canonical state (postgres) ─tick─► snapshot (JSON, KB-scale) ─pull/inline─► snapshot store (client)
   ─reconcile─► scene state (birds, perches, actions, plans, weather, lighting)
   ─performers─► per-bird action timelines and pose params, per-frame
   ─renderer─► WebGL2 frame       ─audio scheduler─► worklet synth events
   ─prose kit─► narration sentences, captions
```

The boundary is the snapshot. Everything left of it is deterministic engine code with no knowledge of pixels or audio. Everything right of it is presentation that must be able to start mid-action from any snapshot, which is what makes "the aviary was already running" true on the first frame.

### 2.5 Monorepo and stack

```
packages/engine     pure TS: types, tick(), drift, mood, perch, weather, call planner, greeting,
                    offers, newcomer schedule, notebook scorer, species config. No IO. Deterministic.
packages/prose      naturalist templates for notebook, narration, captions; voice linter; English only
packages/protocol   zod schemas for snapshot, events, effects; shared by api/sim/web
packages/renderer   WebGL2 scene, bird rigs, performers, reduced-motion pose-fade mode
packages/audio      AudioWorklet synth, voice pool, mixer, ambient bed, caption descriptors
apps/web            client (Vite); code-split by surface
apps/edge           edge worker (HTML shell, inline snapshot)
apps/api            Fastify API
apps/sim            tick scheduler and workers
apps/mailer, apps/jobs
```

TypeScript end to end so the engine, prose kit, and protocol run identically in Node and the browser (the client runs the greeting fallback, call-plan extension, and prose composition from the same code the server uses). Tooling: pnpm workspaces, Vite/esbuild, Vitest, Playwright, size-limit for bundle budgets, OpenTelemetry.

### 2.6 Determinism and seeds

- `aviary.seed` (64-bit, at creation) and `bird.seed` (derived from bird UUID) are the roots. Engine RNG is `rng(aviary_seed, tick_no, purpose)`; every random choice in the tick names its purpose so re-running a tick reproduces it exactly.
- The client uses `rng("local", bird_seed, minuteBucket, purpose)` for non-canonical motion, so two devices are similar but not required to match.
- A tick is a pure function `tick(state, events[], now, seed) → (state', effects[], notebookCandidates[])`. This is what makes dormant coalescing (§5.1), catch-up on open, calibration simulation, and golden tests all the same code path.

---

## 3. Data model

Postgres. `uuid` primary keys generated server-side. All timestamps `timestamptz`. Trait columns are `real` in `[0,1]`.

### 3.1 Identity and accounts

```sql
accounts (
  id                 uuid primary key,          -- synthetic; the only account identifier used anywhere
  email_ciphertext   bytea not null,            -- envelope-encrypted with a KMS-wrapped data key
  email_blind_index  bytea not null unique,     -- HMAC-SHA256(normalized email, server secret); lookup only
  email_verified_at  timestamptz,
  timezone           text not null default 'UTC',   -- IANA; last reported by a signed-in client
  settings           jsonb not null default '{}',   -- captions, reduced_motion, calls, visit_notifications
  created_at         timestamptz not null,
  deleted_at         timestamptz,               -- soft-delete marker
  hard_delete_after  timestamptz                -- deleted_at + 30 days
);
sessions (
  id uuid pk, account_id uuid null, kind text check (kind in ('user','visitor')),
  invite_id uuid null,                          -- visitor sessions only
  token_hash bytea unique, device_label text, created_at, last_seen_at, revoked_at
);
magic_links (id uuid pk, email_blind_index bytea, token_hash bytea unique, expires_at, consumed_at, ip_hash bytea);
email_changes (id uuid pk, account_id, new_email_ciphertext bytea, new_blind_index bytea, token_hash bytea unique, expires_at, verified_at);
```

Email appears in exactly three ciphertext columns (`accounts`, `email_changes`, `invites`). No other table, message, log, metric label, cache key, or partition key ever contains it or anything derived from it except the HMAC blind index, which is never exported.

### 3.2 Aviary, birds, personality

```sql
aviaries (
  id uuid pk, account_id uuid unique, seed bigint, created_at,     -- created_at is the age anchor for newcomers
  engine_version int, tick_no bigint, event_cursor bigint, next_tick_at, cadence text check (cadence in ('active','dormant')),
  settled_at timestamptz, settled_undo_until timestamptz,
  weather jsonb,                                -- {kind, started_at, ends_at, intensity} or null
  last_open_at, last_presence_at,
  newcomer jsonb, next_newcomer_at              -- {species_id, arrived_at, leaves_at} or null
);
birds (
  id uuid pk, aviary_id, ordinal smallint, species_id text, name text, adopted_at, seed bigint,
  boldness real, warmth real, vocal real, plumage real, curiosity real,   -- sim role only
  attunement real,                              -- fast-timescale, hidden (§5.5)
  mood text, mood_since, mood_locked_until,
  perch_zone text check (perch_zone in ('front','middle','back')), perch_slot smallint, perch_since,
  action jsonb,                                 -- {kind, started_at, seed, duration_ms}
  last_offer_reaction_at,
  check (boldness between 0 and 1) ... (same for each trait)
);
drift_ledger (bird_id, tick_no, trait text, delta real, source text, local_date date);   -- calibration instruments, sim db only
trait_caps (bird_id, local_date, trait text, applied real, primary key (bird_id, local_date, trait));  -- daily/weekly cap bookkeeping
```

Species are static config in `packages/engine/species/*.ts` (silhouette params, palette, motif library, time-of-day activity curve, `night_active` flag). Species never change under a bird; adding species to the pool later is additive.

### 3.3 Event log and presence

```sql
events (
  seq bigserial primary key,                    -- server order; the tick consumes by seq
  aviary_id uuid, id text,                      -- id is the client ULID; unique (aviary_id, id) for idempotency
  session_id uuid, type text, bird_id uuid null, payload jsonb,
  occurred_at timestamptz,                      -- client time, server-corrected and clamped (§6.3)
  received_at timestamptz, consumed_tick bigint
);
presence_intervals (aviary_id, session_id, from_at, to_at);     -- materialized from presence events; unioned per tick
presence_days (aviary_id, local_date, seconds int);             -- feeds caps; never leaves sim db
```

The `api` role has `INSERT` on `events` only; `sim` has `SELECT` and `UPDATE consumed_tick`. Nothing has `DELETE` except the hard-delete job. Visitor sessions have no grant path to `events` at all (the API rejects before the DB is reached, and the DB role check is the backstop).

### 3.4 Ticks and snapshots

```sql
ticks (aviary_id, tick_no, ran_at, from_seq, to_seq, sub_steps smallint, duration_ms, engine_version, primary key (aviary_id, tick_no));
snapshots (aviary_id primary key, tick_no, etag text, payload jsonb, updated_at);   -- latest only; replicated to KV
```

### 3.5 Field notebook

```sql
notebook_entries (id text pk /* ULID */, aviary_id, local_date date, kind text, seed bigint, text text, created_at);
notebook_budget (aviary_id primary key, tokens real, refilled_at);   -- sparsity token bucket (§5.15)
```

Entries are immutable; there is no update or delete path except account hard-delete. `kind` is internal (for template balancing), never displayed.

### 3.6 Visits

```sql
invites (
  id uuid pk, aviary_id, visitor_email_ciphertext bytea, visitor_email_blind_index bytea,
  token_hash bytea unique, created_at, expires_at /* +30 d */, consumed_at, active_until /* consumed_at + 30 d */, revoked_at
);
visits (id uuid pk, invite_id, aviary_id, started_at, last_seen_at, ended_at);
```

### 3.7 Exports, deletion, retention

```sql
exports (id uuid pk, account_id, requested_at, object_key text, expires_at /* +7 d */);
```

- Soft delete: `accounts.deleted_at` set; sign-in still works and shows the restore surface; ticks pause (cadence `dormant`, no notebook generation); visits return "no longer available"; invites are suspended.
- Hard delete (job, daily): deletes in dependency order: visits, invites, notebook, drift ledger, caps, events, presence, snapshots (and KV key), ticks, birds, aviary, sessions, exports (and object), magic links, email changes, account. Verified by a post-delete count query that must return zero rows across all tables for the account UUID.
- Backups are encrypted and retained 30 days, so hard-deleted data ages out of backups within 30 days of hard deletion. The privacy policy states this.

### 3.8 Invariants (tested)

1. Every trait update is non-negative (trigger + property test).
2. `birds.id` never changes; renaming touches only `name`.
3. `events` is append-only; `consumed_tick` is the only mutable column.
4. Only the `sim` role can write trait, mood, perch, attunement, or `action`.
5. Email exists only as ciphertext in three tables plus blind indexes; a schema test greps column names and a log test greps output for `@`.
6. A snapshot never contains a raw trait (protocol schema forbids the keys; fixture test).

---

## 4. API surface

### 4.1 Conventions

- JSON over HTTPS under `/v1`. Cookie session (`HttpOnly; Secure; SameSite=Lax`), CSRF token double-submit on all non-GET. Session token is 256-bit random, stored hashed; a separate short-lived signed **edge hint** cookie (10 min, rotating key) carries `{aviary_id, kind}` so the edge can inline a snapshot without touching the database.
- Every response carries `server_time`; the client keeps a clock-offset estimate per session (§6.3).
- Errors are `{error: {code, message}}` where `message` is already the matter-of-fact user copy; the client does not invent error prose.
- Visitor sessions (`kind = visitor`) may call only the §4.7 visitor endpoints; everything else returns 403.
- Rate limits (Redis, sliding window): magic link 5/email/hour and 20/IP/hour; events 120 requests/min/session; invites 10/day/account; export 2/day/account.

### 4.2 Auth

| Method | Path | Behavior |
|---|---|---|
| POST | `/v1/auth/magic-link` `{email}` | always 202 (no account enumeration); creates account lazily on first successful link consumption; link valid 15 min, single use |
| GET | `/auth/callback?token=` | consumes token, issues session + edge hint, redirects to `/` (or to `/adopt` for a brand-new account); expired/used → matter-of-fact page: "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| GET | `/v1/me` | account settings, timezone, session list (device label, created, last seen), deletion state |
| POST | `/v1/me/sessions/{id}/revoke` | revokes a device session immediately |
| PATCH | `/v1/me/settings` | `{captions, reduced_motion, calls: 'on'|'off', visit_notifications, timezone}` |
| POST | `/v1/me/email-change` `{new_email}` | sends verification to the new address; old address works until verified |
| POST | `/v1/me/export` | 202; job builds JSON, uploads, emails a 7-day link to the verified address |
| POST | `/v1/me/delete` | soft delete now; `hard_delete_after = now + 30 d` |
| POST | `/v1/me/restore` | clears soft delete while inside the window ("I changed my mind") |

### 4.3 Open and snapshot

| Method | Path | Behavior |
|---|---|---|
| POST | `/v1/aviary/open` `{client_time, tz, reason: 'navigate'|'visible'|'gap'}` | runs catch-up if the aviary is behind (§5.1), computes absence from `last_presence_at`, selects the return-greeting (§5.11), clears a lingering settled state, updates `last_open_at` and cadence to `active`; returns the full snapshot with `greeting` populated |
| GET | `/v1/aviary/snapshot` (`If-None-Match`) | latest snapshot; 304 when unchanged; used for the visible keepalive |

### 4.4 Events

`POST /v1/aviary/events` accepts a batch and returns effects synchronously for fast-path types.

```json
{ "events": [
  { "id": "01J9…", "type": "presence",        "occurred_at": "…", "payload": { "from": "…", "to": "…" } },
  { "id": "01J9…", "type": "listen_in_start", "occurred_at": "…", "bird_id": "…" },
  { "id": "01J9…", "type": "listen_in_end",   "occurred_at": "…", "bird_id": "…", "payload": { "duration_ms": 184000 } },
  { "id": "01J9…", "type": "offer",           "occurred_at": "…", "payload": { "kind": "seed" } },
  { "id": "01J9…", "type": "offer",           "occurred_at": "…", "payload": { "kind": "song", "song_id": "dusk_three" } },
  { "id": "01J9…", "type": "settle",          "occurred_at": "…" },
  { "id": "01J9…", "type": "settle_undo",     "occurred_at": "…" },
  { "id": "01J9…", "type": "greeting_shown",  "occurred_at": "…", "payload": { "bird_id": "…", "form": "glance", "seed": 7, "source": "local" } }
] }
```

Response:

```json
{ "server_time": "…", "accepted": ["01J9…"], "duplicates": [], "rejected": [{ "id": "…", "reason": "offer_in_progress" }],
  "effects": [ { "event_id": "01J9…", "reaction": { /* reaction plan, §5.12 */ }, "mood_updates": [ { "bird_id": "…", "mood": "content", "since": "…" } ] } ],
  "etag": "48214-1" }
```

- `presence`, `listen_in_*`, `greeting_shown`: appended, no effect object.
- `offer`, `settle`, `settle_undo`: appended **and** applied to canonical mood/aviary state immediately under the aviary lock (server-authored fast path, §5.2). The effect is returned so the acting device animates without waiting for a tick; other devices see it on their next pull.
- Idempotent on `(aviary_id, id)`: a retry returns the original effects.

### 4.5 Birds, adoption, newcomer

| Method | Path | Behavior |
|---|---|---|
| POST | `/v1/aviary/adopt` `{names: [a, b]}` | first-run only; server has already chosen two species; names default to server suggestions if omitted |
| PATCH | `/v1/aviary/birds/{id}` `{name}` | rename; 1–24 chars; no effect on anything else |
| POST | `/v1/aviary/newcomer/adopt` `{name}` | converts the current newcomer into a bird (new stable id, seeded personality) |
| POST | `/v1/aviary/newcomer/decline` | newcomer departs; next eligibility per §5.14 |

### 4.6 Notebook

`GET /v1/aviary/notebook?before=<ulid>&limit=40` returns entries newest first, cursor-paginated, unbounded history. Read-only; no other verbs exist.

### 4.7 Visits

| Method | Path | Behavior |
|---|---|---|
| POST | `/v1/invites` `{email}` | creates invite, emails one-time link (`/visit/<token>`); 30 d expiry |
| GET | `/v1/invites` | outstanding and active invites (masked email as entered, status, expiry) |
| DELETE | `/v1/invites/{id}` | revoke; effective at the visitor's next pull |
| GET | `/v1/visits?before=` | visit log: visitor email, date, approximate duration; newest first; never pushed, never badged |
| GET | `/visit/<token>` | consumes the token, issues a visitor session (30 d, bound to the invite), serves the shell with an inline snapshot; expired, revoked, or already-consumed-elsewhere → "This visit is no longer available." |
| GET | `/v1/visit/snapshot` | visitor keepalive; checks invite status every call; 410 on revocation; records `visits.last_seen_at` |

Visitor sessions cannot call `/open`, `/events`, notebook, or anything else; visitor presence is never recorded; the snapshot served to a visitor is byte-identical to the host's (no greeting, no offers state) so there is no "show-off" rendering path to drift.

### 4.8 Snapshot schema (`packages/protocol/snapshot.ts`)

```json
{
  "v": 1, "aviary_id": "…", "tick_no": 48213, "etag": "48213-3", "server_time": "…", "tz": "America/Los_Angeles",
  "settled": null,
  "weather": { "kind": "rain", "started_at": "…", "ends_at": "…", "intensity": 0.4 },
  "last_presence_at": "…",
  "birds": [
    { "id": "…", "ordinal": 0, "name": "pip", "species": "greywarbler",
      "mood": "curious", "mood_since": "…",
      "perch": { "zone": "front", "slot": 1, "since": "…" },
      "transit": null,
      "expression": { "plumage": 5, "approach": 4, "warmth": 3, "voice": 6, "curiosity": 4, "attunement": 7 },
      "action": { "kind": "preen", "started_at": "…", "seed": 913, "duration_ms": 6200 },
      "call_plan": [ { "at": "…", "motif": "rise3", "seed": 4471, "response_to": null } ],
      "greeting_weight": 0.62 }
  ],
  "greeting": { "bird_id": "…", "form": "call_step", "seed": 7, "start_offset_ms": 900,
                "followers": [ { "bird_id": "…", "form": "glance", "delay_ms": 1800 } ] },
  "offer": { "active": null, "available": true },
  "newcomer": null,
  "next_pull_ms": 60000
}
```

- `expression.*` are integer **expression bands** 0–11: `floor(trait × 12)` clamped. They are what the renderer and call planner consume. Twelve bands means one band ≈ 0.083, which is deliberately the size of "visible drift" in §5.4: a band crossing is the moment drift becomes visible, and the client cross-fades the change over 60 s so it is never a pop.
- `greeting` is present only on `/open` responses.
- `call_plan` covers now → now + 120 s per bird (§5.9).

---

## 5. Simulation engine

All of §5 is `packages/engine`: pure, deterministic, no IO. The sim workers and the API fast path call it; the calibration harness and golden tests call it with synthetic time.

### 5.1 Tick contract and scheduling

- **Cadence.** Active aviaries (a client opened or pulled within 24 h) tick every 60 s. Dormant aviaries tick every 15 min, and each such tick runs fifteen 60 s sub-steps internally so results are bit-identical to having ticked every minute. There is no presence during dormancy, so drift is zero; sub-steps exist so mood, weather, perch, attunement decay, and notebook logic advance on the same clock as always.
- **Catch-up on open.** `/open` first checks `aviaries.next_tick_at`; if the aviary is behind (worker lag, dormant), it runs the missing sub-steps synchronously (bounded to 24 h of sub-steps; older gaps use a proven fast-forward that applies the same per-sub-step functions with zero events) before building the snapshot. The user always opens the aviary as it is now, never as it was.
- **Single writer.** Every tick and every fast-path effect runs inside `withAviaryLock(aviary_id)` (Redis lock with fencing token, 10 s TTL, renewed) and a Postgres transaction. Lock loss aborts the transaction. This is the mechanical form of "only the server writes personality; one writer per aviary."
- **Idempotency.** A tick is keyed by `(aviary_id, tick_no)`; a duplicate worker run is a no-op.
- **Budget.** p50 tick < 40 ms, p99 < 500 ms; alarm at p99 > 5 s (§10.6).

### 5.2 Tick pipeline (one 60 s step)

1. Load aviary, birds, unconsumed events with `seq > event_cursor` and `occurred_at ≤ step_end`.
2. **Presence accounting** (§5.3): union intervals across sessions → `P` seconds in this step.
3. **Listen-in accounting**: per bird, seconds of listen-in overlapping this step.
4. **Offer accounting**: offer events already applied by the fast path; the tick reads their outcomes (which birds approached/accepted) for drift.
5. **Drift** (§5.4): compute non-negative deltas, apply caps, write traits and ledger.
6. **Attunement** (§5.5): rise with presence, decay with absence.
7. **Weather** (§5.8): start/end events.
8. **Mood** (§5.6): compute scores, apply transitions with hysteresis and dwell.
9. **Perch** (§5.7): at most one departure per step.
10. **Idle action refresh**: for birds whose `action` has expired, pick the next canonical action (mood-weighted) with a seed; the client refines micro-motion locally.
11. **Call plan** (§5.9): regenerate the 120 s plan for each bird using `rng(seed, tick_no, "calls")`, preserving already-scheduled calls in the first 30 s so the client never hears a plan swap.
12. **Newcomer** (§5.14): arrivals/departures.
13. **Notebook** (§5.15): score candidate observations, spend tokens, write entries.
14. Advance `event_cursor`, `tick_no`; build and store the snapshot; publish to KV; record `ticks` row.

Fast-path (offer, settle, settle_undo) runs steps 8 and 10 for the affected birds only, plus the reaction plan, immediately at ingestion under the same lock. It never touches step 5.

### 5.3 Presence accounting (server side)

- A presence event carries `[from, to]`. Accepted only if `to − from ≤ 45 s`, `to ≤ received_at + 5 s`, and the session is a `user` session. Longer or future intervals are clipped, not rejected, and counted in an aggregate "clipped presence" metric (no account dimension) so a client bug surfaces as a fleet-wide number.
- Per step, intervals from **all** sessions of the aviary are unioned; overlapping laptop and phone presence counts once. Presence is "the user is watching", not "devices are open".
- `P` for the step is the unioned seconds ∩ the step window, so at most 60.
- Presence never accrues during a settled state (§5.13) or for visitors.
- `last_presence_at` is the end of the latest interval; it drives absence length for greetings and attunement decay.

### 5.4 Drift function

For each bird `b` and trait `t ∈ {boldness, warmth, vocal, plumage, curiosity}` with current value `v`:

```
Δpres   = α_t · (P / 3600) · (1 − v)                      -- presence: all birds, all traits
Δlisten = β_t · (L_b / 3600) · (1 − v)                    -- listen-in seconds on this bird; warmth, vocal only
Δoffer  = γ_accept · accepted_b  (curiosity)  +  γ_near · approached_b  (boldness)
Δraw    = Δpres + Δlisten + Δoffer                        -- every term ≥ 0 by construction
Δ       = min(Δraw, cap_day − applied_today_t, cap_week − applied_week_t, 0)⁺
v'      = min(1, v + Δ)
```

Constants are in §14 (`α_t`, `β_t`, `γ_*`, `cap_day = 0.012`, `cap_week = 0.05`). Properties:

- **Monotonic toward expressive.** No term is negative; absence contributes zero. Neglect changes nothing in the vector. "Quieter after two weeks away" comes from attunement (§5.5), not from personality.
- **Slow.** One 60 s step can move a trait at most ≈ 0.001. A day is capped at 0.012, so no single session (even an eight-hour one) crosses a visible band (0.083). A week is capped at 0.05.
- **Asymptotic.** The `(1 − v)` factor makes late drift slower; a bird "becomes itself" rather than pegging at 1.0.
- **Calibration target math** (regular user: 5 sessions/week × 12 min = 1 h/week; seed values 0.3–0.5 so `(1 − v) ≈ 0.6`): plumage after one week ≈ 0.05 × 1 × 0.6 = 0.030 (instrument threshold 0.01: **measurable**); after three weeks ≈ 0.085 (≥ one band: **visible**). A daily one-hour user hits the weekly cap (0.05) and crosses a band in about two weeks; a lapsed user moves 0. These three personas are the CI calibration test (§5.16).
- The `drift_ledger` row per applied delta is the "instrument" the PRD refers to; it is readable only in the sim DB and only for synthetic and consenting dogfood accounts (§10.5).

### 5.5 Attunement (fast timescale, hidden)

`attunement ∈ [0.35, 1]` per bird. Rises with presence (`a' = a + 0.8 · (P/3600) · (1 − a)`), decays with absence (`a' = a · e^(−Δt/τ)`, τ = 10 days, floor 0.35). It multiplies call rate (`0.5 + 0.5a`), the probability that non-greeter birds follow a greeting, and chorus joining. This is the engine's implementation of "a bird that gets ignored becomes ambient: still alive, still calling, greeting less often." It is not a trait, it is not exposed, and it recovers within roughly an hour of presence, so the return after a fortnight is "quieter, then easing back", never "punished." It is shipped to the client as an expression band so both devices render the same quietness.

### 5.6 Mood engine

Moods: `wary, content, curious, drowsy, alert, roosting`. `roosting` is the PRD's "settled (eyes closed, low on the perch)" night state, renamed internally to avoid colliding with the aviary-level settled lighting state (§15).

Each step computes a score per mood:

```
score(m) = prior_tod[species][m](local_hour)          -- e.g. alert peaks 05–08, drowsy 16–19, roosting 20–05 (inverted for the night-active species)
         + w_weather · weather[m]                     -- rain: drowsy +0.10, vocal damp handled separately; wind: alert +0.15 if boldness ≥ band 6 else wary +0.15
         + w_recent · recent[m]                       -- last 15 min: accepted offer → content +0.30 / curious +0.15; song offer → curious +0.2; listen-in on this bird → content +0.1
         + w_social · social[m]                       -- wary neighbours: wary + 0.25 · (1 − boldness); chorus in progress: alert +0.1
         + w_pers · pers[m]                           -- boldness lowers wary; curiosity raises curious; warmth raises content
         + noise(0.05, rng)
```

Transition rule: switch from current mood `c` to `argmax m` only if `score(m) > score(c) + 0.15` (hysteresis) **and** dwell ≥ 8 min (`mood_locked_until`), then with probability 0.35 per step (so a persistent pressure flips within ≈3 min, not on a hard edge). Event-triggered immediate switches bypass dwell: accepted offer → `content` or `curious`; alarm call → `wary` for birds within one perch zone with probability `(1 − boldness)`; settle → drowsy pressure +0.3 for 20 min.

"Daily-ish reset" is emergent: overnight the `roosting`/time-of-day prior dominates and recent-interaction terms expire, so morning moods re-anchor to the prior without any hard reset the user could notice. Mood persists across sessions because it is simply canonical state advanced by the tick.

### 5.7 Perch selection

Nine slots: front 3, middle 3, back 3. Zone score per bird: `front: 0.5·boldness + 0.3·[mood ∈ {curious, alert}] + 0.2·attunement`, `back: 0.5·(1 − boldness) + 0.4·[mood = wary] + 0.2·[mood ∈ {drowsy, roosting}]`, `middle: 0.4 + 0.2·[mood = content]`. Move only if the best zone beats the current by 0.2 and `perch_since` ≥ 6 min; at most one bird departs per step so flights read as individual decisions. Roosting birds do not move. Slot choice prefers a slot adjacent to a high-warmth bird's slot when the mover has warmth ≥ band 6. The user never sets perches; there is no API for it.

### 5.8 Weather scheduler

Per step, if no weather is active: rain starts with probability `3 / (7·1440)` (≈3 per week), duration 3–8 min, intensity 0.3–0.7; wind with probability `5 / (7·1440)`, duration 2–5 min. Effects: rain multiplies call rate by 0.4 during and 0.7 for 10 min after; wind adds the mood pressures above. Weather ships in the snapshot with start/end so all devices and visitors render the same rain at the same time.

### 5.9 Call planner and call grammar

Data structures (`packages/engine/calls`):

```ts
type Syllable = { dur: [number, number]; f0: { start: number; end: number; curve: 'lin'|'exp'|'sig'; vib?: { rate: number; depth: number } };
                  harm: [number, number]; noise: number; env: { a: number; d: number; s: number; r: number }; formant?: number };
type Motif    = { id: string; role: 'contact'|'song'|'query'|'alarm'|'response'|'night'; syllables: Syllable[]; gaps: [number, number][];
                  moodWeight: Partial<Record<Mood, number>> };
type Signature = { f0Mult: number; tempoMult: number; vibDepth: number; harmTilt: number; motifOrder: string[]; tagSyllable: Syllable };
type PlannedCall = { at: string; motif: string; seed: number; response_to: { bird_id: string; at: string } | null };
```

- Each species has 4–6 motifs. Each bird's `Signature` is derived once from `bird.seed` (f0 ±12 % of species range, tempo ±15 %, a fixed tag syllable at motif start or end). The signature is what stays recognizable across mood and drift: mood changes tempo, gain, syllable count, and motif choice, but never the f0 multiplier, tag syllable, or harmonic tilt.
- Per-bird call rate (calls/min): `λ_b = λ_species(local_hour) · (0.4 + 1.2·vocal) · moodMult[m] · weatherMult · (0.5 + 0.5·attunement) · settledMult`, with `moodMult = {content 1.0, curious 1.1, alert 1.3, wary 0.5, drowsy 0.4, roosting 0}` and `settledMult = 0.3`. The night-active species uses an inverted `λ_species` curve so it calls into the late hours while others roost.
- Calls are sampled as a Poisson process over the 120 s horizon with `rng(seed, tick_no, "calls")`. Motif choice is weighted by `moodWeight`. Each call gets a `seed` that the client uses for sub-perceptual per-syllable jitter (±3–6 % on duration, f0, gain), so no call is ever repeated exactly.
- **Responses.** When bird A has a call at `t`, each other bird B responds at `t + U(0.4, 1.5) s` with probability `0.6 · warmth_B · (1 − [mood_B = wary]) · (0.5 + 0.5·attunement_B)`, choosing a `response` motif. Responses can chain once (A → B → A) but not further, to keep chorus events from cascading.
- **Chorus.** If ≥ 2 birds with vocal ≥ band 7 have calls within a 20 s window, the planner marks a chorus window; birds with vocal ≥ band 5 join with probability `0.4·vocal`. Chorus windows are a notebook candidate.
- **Alarm.** Rare (`≈ 1/day` per aviary): a wind gust or "passing shadow" ambient event triggers an `alarm` motif from the most alert bird; nearby birds react per §5.6.
- The client keeps calls whose `at` has passed and merges new plans by `(bird_id, at)`; if no plan covers the next 20 s (late tick), it extends locally with `rng("local-ext", bird_seed, minuteBucket)` and the same λ formula evaluated from expression bands, and discards the extension when a server plan arrives.

### 5.10 Bird-to-bird interaction

Responses and chorus (§5.9), wary spread and alarm (§5.6), and slot adjacency by warmth (§5.7) are the three channels. They are all computed server-side, so the social texture is identical on every device and for visitors.

### 5.11 Return-greeting selection

Computed on `/open`; executed by the client from the first frames.

1. Absence bucket from `now − last_presence_at`: `short` (< 10 min), `medium` (10 min – 6 h), `long` (6 h – 2 d), `extended` (> 2 d).
2. Greeter weight per bird: `w = (0.3 + 0.7·boldness) · (0.5 + 0.5·warmth) · moodFactor · attunement · U(0.85, 1.15)`, with `moodFactor = {alert 1.2, curious 1.1, content 1.0, drowsy 0.7, wary 0.5, roosting 0.4}`. Sample one greeter proportional to `w`. The bolder, warmer bird usually greets first; the noise term is why "pip greeted before wren today, first time this week" can be true.
3. Form by (bucket × boldness band × mood): `short` → `glance` or `head_tilt`; `medium` → `two_note` or `glance_step`; `long` → `call_step` (call, then a step toward the front rail) or `long_call`; `extended` → `reorient` (longer call, hop forward, second bird responds). Wary birds drop the step; drowsy birds lift the head slowly; roosting birds open one eye and shuffle; the night-active species may call at night.
4. Followers: each other bird follows with probability `0.5·warmth·attunement`, staggered by `U(0.8, 2.5) s` after the greeter, never simultaneously.
5. `start_offset_ms` is 600–1500 ms after first frame so the bird notices the user rather than firing on load.
6. The form's `seed` drives procedural variation inside the form (tilt angle, note count 2–3, call length, step distance); there are no pre-baked variants.
7. If `/open` has not returned within 1.5 s of first frame, the client runs the same selector locally from the inline snapshot's `greeting_weight` values and reports `greeting_shown{source: 'local'}` so the notebook knows who greeted.

No text accompanies the greeting anywhere. The greeting is the welcome.

### 5.12 Offers and reaction plans

- Kinds: `seed` (placed on the front rail, present 120 s), `song` (one of 8 short procedural motifs from a fixed library, played once over ≈6 s through the audio engine's "offered song" voice), `pool` (reflective surface at the front, present 180 s).
- One offer active at a time per aviary; while active, `offer.available = false` and the affordance is quietly unavailable (no timer, no countdown). A second `offer` event during that window is rejected with `offer_in_progress` and has no effect.
- **Reaction plan** (returned as the effect, computed under the lock, then applied to mood):

```json
{ "kind": "seed", "placed_at": { "x": 0.52 }, "expires_at": "…",
  "birds": [ { "bird_id": "…", "response": "approach_eat",        "delay_ms": 2400,  "seed": 11 },
             { "bird_id": "…", "response": "watch_then_approach", "delay_ms": 14000, "seed": 3 },
             { "bird_id": "…", "response": "ignore",              "delay_ms": 0,     "seed": 0 } ] }
```

- Response matrix (seed/pool): `curious` or `content` with curiosity ≥ band 4 → approach (2–5 s); `wary` → watch, then approach after 10–25 s with probability `0.5·curiosity`; `drowsy`/`roosting` → ignore (drowsy: 20 % chance of a slow look); `alert` → approach quickly, brief. Pool adds `drink | bathe | watch` weighted by mood and a per-bird seed. Song: `join` (calls in the song's tempo) if vocal ≥ band 6 and mood ∉ {wary, drowsy}; `quiet` if wary; `call_against` (offset, contrasting motif) otherwise with probability `0.4·vocal`.
- **Per-bird cooldown** (`last_offer_reaction_at`, 4 min): a bird that reacted within the window does not approach the next offer and contributes no drift from it. Drift accounting: `accepted_b = 1` if the response was `approach_eat`, `drink`, `bathe`, or `join`; `approached_b = 1` if the bird moved toward the offer at all.
- Notebook candidates: first bathe, a wary bird finally approaching, a song joined.

### 5.13 Settle

- `settle` sets `settled_at = now`, `settled_undo_until = now + 5 s`; effect returned immediately. Any click, tap, or key inside the aviary before `settled_undo_until` sends `settle_undo`, which clears both (server accepts undo up to 7 s to absorb latency).
- While settled: presence does not accrue (settle is terminal for the presence window, exactly like tab-close); `settledMult = 0.3` on call rate; drowsy pressure (§5.6); the client renders the evening palette regardless of local hour and ramps master audio to 0.35 over 4 s.
- Settled ends on: `/open` (new tab, tab becoming visible again after hidden, or a long frame gap), or an explicit re-engagement: listen-in, offer, or focusing a bird. Pointer movement alone does not un-settle; the user can move the mouse to reach the top bar without waking the aviary. Ending settled resumes presence accounting on the next heartbeat.
- Settle has no drift term. Not settling has no consequence and no surface.

### 5.14 Newcomers (birds 3–7)

- Eligibility is aviary age only: bird 3 at 90 days, 4 at 180, 5 at 300, 6 at 450, 7 at 630 (§14). This matches "an aviary a few months old offers a third bird; a year-old aviary may have grown to five or six."
- At eligibility the tick creates `newcomer = {species_id, arrived_at, leaves_at = +14 d}`. Species is drawn from the pool excluding species already present until the pool is exhausted; the night-active species is excluded from starters and given 2× weight from bird 3 on (§15).
- The newcomer renders on the back perch as a tentative bird (fewer calls, no greeting role, keyboard-focusable). Focusing or clicking it opens a naming sheet in naturalist voice: "a new bird has been at the back perch since tuesday. name it, or let it move on." Adopt creates a bird with a fresh stable id and seeded traits, then the bird takes a normal perch. Decline or 14 days of silence → the newcomer leaves quietly and `next_newcomer_at = now + 30 d`. No prompt, badge, or toast points at it; the bird being there is the offer.

### 5.15 Field notebook generation

- **Sparsity** is a token bucket per aviary: capacity 2, refill 1 token per 72 h, paused while dormant. A candidate observation is written only if its score ≥ 0.6 and a token is available; the most noteworthy candidate of the step wins. Expected rate: about one entry per three days for a regularly visited aviary, more when something notable happens, never one per session.
- **Candidates and scores** (examples): greeter differs from the week's usual greeter (0.8, "pip greeted before wren today, first time this week"); a wary bird approached an offer after waiting (0.7); first bathe in a pool (0.9); a chorus window (0.6); rain passed with a note on who kept calling (0.5); a long quiet stretch with one bird preening through it (0.5); night-active bird calling past midnight (0.5); newcomer arrival (0.9) and adoption (0.9); a bird moved to the front perch for the first time in two weeks (0.7, this is how personality drift becomes narratable without numbers).
- **Voice**: templates in `packages/prose/notebook`, lowercase, present tense, bird-named, dated by the aviary's local weekday; slot-filled and varied by seed; the voice linter runs over every template. The scorer refuses any candidate whose subject is the user (visits, frequency, absence, duration). The notebook can say "pip greeted first today"; it cannot say "you were here every day this week", and there is no template that could.
- Entries are immutable, unbounded, newest first, cursor-paginated, virtualized on the client.

### 5.16 Engine tests and calibration harness

- **Golden ticks**: fixed seeds and event scripts produce byte-identical snapshots across Node and browser builds (the client runs greeting and call-plan extension code).
- **Property tests**: traits never decrease; every step's Δ ≤ 0.001; daily and weekly caps hold; a settled or visitor session never produces presence; presence union never exceeds 60 s per step.
- **Calibration harness** (`apps/sim/calibrate`): simulates 90 days for personas {regular 1 h/week, heavy 7 h/week, lapsed (2 weeks on, 2 off), night-owl} with synthetic presence and listen-in; asserts measurable-at-1-week and visible-at-3-weeks for the regular persona, no-visible-change-in-one-session for all, and zero drift for the lapsed persona's off weeks. It also reports greeting-form distribution, chorus frequency, and notebook entries per week so tuning is a number, not a feeling.
- **Recognizability**: an automated spectral test that a bird's tag syllable and f0 multiplier survive every mood and every expression band, plus a human listening study at 2, 4, and 7 birds before bird 3 can reach real users (§12.4).

---

## 6. Sync model

### 6.1 One canonical aviary, one writer, additive deltas

There is no client state to merge. The server is the only writer of personality, mood, perch, attunement, weather, notebook, and call plans; clients render snapshots and append events. Personality changes are server-computed deltas applied in event-log order; no code path accepts an absolute trait value from anywhere. The lock-with-fencing rule (§5.1) makes "two writers" impossible even inside the server.

### 6.2 When the client pulls

| Trigger | Call | Notes |
|---|---|---|
| navigation / first load | inline snapshot from HTML, then `POST /open` | greeting comes from `/open` |
| `visibilitychange` → visible | `POST /open{reason: 'visible'}` | ends any settled state; greeting if absence ≥ 10 min, else none |
| long frame gap (rAF delta > 5 s, laptop resume) | `POST /open{reason: 'gap'}` | |
| visible keepalive | `GET /snapshot` with `If-None-Match` every `next_pull_ms` (60 s), jittered ±5 s | 304 is the common case |
| after an effect from another device | none needed; next keepalive | cross-device latency ≤ ~65 s (§15) |
| hidden tab | no pulls, no rendering, no audio scheduling; presence heartbeats stop | simulation continues server-side |

Reconciliation is idempotent: a snapshot equal to the current one is a no-op; a newer one updates canonical fields and lets performers finish or gracefully redirect current actions (§7.5).

### 6.3 Event submission semantics

- Client generates a ULID per event, queues in IndexedDB-backed memory, flushes every 5 s or immediately for fast-path types, batches ≤ 50, retries with backoff; `pagehide` flushes via `sendBeacon`.
- Server dedupes on `(aviary_id, id)`, assigns `seq` on receipt, corrects `occurred_at` by the session's measured clock offset and clamps it to `[received_at − 10 min, received_at + 5 s]`. A very stale batch (offline for an hour) is accepted, clipped to those bounds, and counted in a fleet metric.
- Ordering across devices is by `seq`. Two devices offering "simultaneously" resolve to one accepted and one `offer_in_progress`, and both devices converge on the next pull.
- Fast-path effects are authoritative from the moment they are returned; the acting device applies them immediately, and its next snapshot will already contain them.

### 6.4 Multi-device presence de-duplication

Presence intervals from every session are unioned per step (§5.3). Two devices open side by side count as one user watching. This is also why presence is per aviary, not per session, in the drift function.

### 6.5 Conflict and error surfaces (matter-of-fact voice)

| Situation | Behavior | Copy |
|---|---|---|
| session token revoked or expired | client receives 401, keeps rendering the last snapshot silently for 30 s while attempting refresh, then shows the surface | "Your session timed out. Sign in again to keep watching." |
| magic link expired/used/replayed | callback page | "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| snapshot fetch failing > 3 min | aviary keeps rendering from the last snapshot with local call-plan extension; a small matter-of-fact line appears in the top bar region only after 3 min | "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." |
| `/open` slow at boot | local greeting fallback (§5.11); no surface | |
| visitor invite revoked/expired | visitor's next pull → 410 | "This visit is no longer available." |
| unsupported browser | server-rendered page before any bundle | "Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge." |

There is no "syncing…" indicator, no offline banner during brief blips, and no conflict-resolution UI, because there is nothing for the user to resolve.

### 6.6 Snapshot edge replication

Each tick writes the snapshot to Postgres and to KV (`snap:{aviary_id}`, ≤ 8 KB, with `etag`). The edge inlines it into the HTML when the edge hint cookie is valid. KV replication lag is measured (fleet metric); an inlined snapshot that is a few minutes stale is still used to place birds mid-action on the first frame (that is the point), and the client reconciles when `/open` returns. The only case that shows the quiet field alone is a missing snapshot (new account, cold KV, no hint cookie).

---

## 7. Frontend rendering pipeline

### 7.1 Boot sequence to first bird

```
t=0        navigation
t≈80 ms    edge HTML: ~6 KB critical CSS + a 300-byte inline script that paints the quiet field
           (sky gradient keyed to local hour) before any bundle; inline snapshot JSON;
           <link rel=modulepreload> for the critical chunk; system font stack, no webfont on the critical path
t≈150 ms   critical chunk (≤ 180 KB gz: renderer core, rigs for the species in this snapshot,
           performers, protocol parse, presence monitor) executes; WebGL2 context created;
           birds placed at their snapshot action with phase = now − action.started_at
t≤450 ms   first frame: birds mid-action, ambient ornaments already drifting (target ≤ 500 ms on
           mid-tier mobile over 4G; desktop typically < 200 ms)
t≈150 ms→  POST /open in flight in parallel with chunk execution; on return: reconcile, schedule greeting
t≈600 ms   audio chunk (≈ 60 KB gz) loaded; AudioContext created; if running, ambient bed fades in over 2 s
           and call plan is scheduled; if suspended (autoplay policy), captions on until the first gesture (§8.7)
idle       plumage detail textures generated in requestIdleCallback slices; swapped via 2 s cross-fade
on demand  notebook, settings, offer sheet, invites, export, adoption: separate chunks
```

The critical chunk must draw birds without waiting for textures: birds render first as tessellated vector meshes with flat species palettes, then gain feather detail. That ordering is what keeps 500 ms achievable and is invisible because the detail arrives as a slow cross-fade, not a pop.

### 7.2 Scene composition

Layers back to front: sky (procedural gradient, time-of-day keyed) → far foliage (parallax 0.15) → mid foliage and perch structure (parallax 0.4) → props (still pool, seed) → birds (mid-plane, sorted by perch zone then slot) → foreground branch and passing leaves (parallax 1.3, occasional) → weather overlay → DOM overlays (captions, focus ring, top bar).

- Perch slots are normalized coordinates in a 16:9 design space; zones sit at y ≈ 0.62 (front), 0.48 (middle), 0.36 (back) with slight per-slot variation; back-zone birds scale 0.85 and get a −6 % contrast tint to read as distance.
- **Responsive**: the scene scales to the viewport width and clamps the aspect between 21:9 and 3:4. Narrower than 3:4 (phones in portrait), slot x-coordinates compress toward the centre with a minimum spacing of 1.2 bird widths and back-zone birds tuck slightly higher; wider than 21:9, slots spread. No bird is ever clipped: a layout unit test iterates viewports from 320×568 to 3440×1440 with seven birds and asserts every bird bounding box is inside the canvas.
- No panning, scrolling, or zooming; touch gestures on the scene are only tap (listen-in) and tap-empty (disengage).

### 7.3 Bird rig and procedural visuals

- Each bird is a parametric 2D rig (body, head, beak, eye, crest, wing, tail, legs) with ≈ 40 pose parameters; species define proportions, silhouette, and palette; the bird's expression bands set plumage saturation (shader uniform) and feather-detail level (number of procedural feather strokes baked into its atlas tile at idle time).
- Textures: one 2048² atlas generated at runtime (feather strokes, foliage sprites, leaf/feather ornaments), no bitmap assets in the bundle beyond a few KB of SVG paths. Regenerated only when a bird crosses a plumage band, via the 60 s cross-fade.
- Renderer: in-house WebGL2 batcher (single shader program family, ≤ 40 draw calls/frame, instanced ornaments). Canvas2D fallback for a missing WebGL2 context renders the same rigs without parallax tinting; it is a correctness path, not a supported perf path.

### 7.4 Performers: actions, idle scheduler, mood-shaped idle

Each bird has a performer that owns a timeline of actions. Action kinds: `preen`, `scan`, `head_tilt`, `shuffle`, `fluff`, `stretch`, `hop`, `fly` (perch transit), `call` (synchronized with audio), greeting forms (§5.11), offer responses (§5.12), `roost`, `wake`, `one_eye`.

Mood-keyed idle mix (weights, normalized): 

| mood | preen | scan | head_tilt | shuffle | fluff | posture |
|---|---|---|---|---|---|---|
| content | 0.45 | 0.2 | 0.15 | 0.2 | 0 | normal |
| wary | 0.05 | 0.55 | 0.2 | 0.2 | 0 | upright, back-leaning, perch further back |
| curious | 0.15 | 0.3 | 0.4 | 0.15 | 0 | forward lean, tilts toward call sources and leaves |
| drowsy | 0.15 | 0.1 | 0.05 | 0.2 | 0.5 | low, fluffed, slow blinks |
| alert | 0.1 | 0.5 | 0.3 | 0.1 | 0 | tall, quick head moves |
| roosting | 0 | 0 | 0 | 0.1 | 0.9 | eyes closed, low; breathing only |

The canonical `action` in the snapshot anchors the timeline (so two devices agree on "pip is preening now"); between canonical actions the client fills with local seeded idle from the same mix. Head-tilts target real stimuli (another bird's scheduled call, a leaf spawn) so the aviary reads as attentive to itself. Breathing (1–2 % body scale at 0.25 Hz, mood-scaled) never stops; nothing ever reads as paused.

### 7.5 Interpolation and transitions

- Perch change: performer runs `fly` along a slight arc, duration 0.8–1.6 s by distance; a snapshot arriving mid-flight with a different destination redirects smoothly.
- Mood change: idle mix and posture blend over 10 s; no pop.
- Expression band change: 60 s cross-fade of saturation/detail.
- Action phase alignment on boot: start mid-action using `started_at`; if the action is already finished, start the next idle action immediately, still mid-scene.
- Weather: rain fades in over 3 s; wind ripples foliage with a low-frequency vertex offset.
- Settle: evening palette over 4 s; undo reverses along the same curve.
- Nothing ever teleports, snaps, or resets; a test harness replays recorded snapshot sequences and asserts max per-frame position delta for each bird stays under a threshold except inside `fly`.

### 7.6 Day/night, weather, ornaments

- Lighting is a function of local hour: sunrise 06:30, sunset 19:30 ± 45 min by month (fixed schedule, no geolocation, §15). Palette keyframes at 05, 07, 12, 17, 19, 21, 00 interpolate continuously; night dims to ≈ 35 % luminance with a cool cast; evening warms. Roosting birds at night, the night-active species awake.
- Ornaments: leaves and feathers spawn from a pooled set (max 6 live), Poisson at ≈ 1/40 s, lifetime 6–12 s; foreground branch pass ≈ 1/3 min. Purely client-local (no per-leaf state), reduced-motion removes them.
- Still pool renders a flipped, alpha-faded reflection of nearby birds; seed is a small cluster on the rail.

### 7.7 Top bar and chrome

- Four icons only: account/settings, accessibility settings, field notebook, offer. Labels are visually hidden but exposed to assistive tech; icons carry no badges, dots, counts, or hover tooltips.
- Fade: after 4 s of pointer stillness (6 s on touch) the bar eases to 12 % opacity over 1 s; pointer movement, any key, or a tap anywhere restores it in 150 ms. It never fades while it or a sheet has focus, while reduced-motion is on (it stays at 60 % instead, fading only opacity and slower), or while an error line is showing.
- Sheets (notebook, offer, settings) slide from the top bar edge, are dismissible with Escape or tapping the scene, and never cover more than 60 % of the viewport so the aviary stays visible behind them.

### 7.8 Reduced-motion mode (a designed surface)

Triggered by `prefers-reduced-motion: reduce` or the accessibility setting; takes effect without reload.

| Feature | Default rendering | Reduced-motion rendering |
|---|---|---|
| idle actions | continuous procedural motion | 2–4 key poses per action, cross-faded over 1.5–3 s each; breathing reduced to 0.5 % |
| perch transit | flight arc | fade out at origin (1.2 s), fade in at destination (1.2 s), 0.5 s apart |
| greeting | tilt, step, call animation | pose cross-fade (head up → toward viewer); call and caption unchanged |
| offer reactions | approach walk/hop | cross-fade to a nearer pose; eat/drink/bathe as 2–3 pose fades |
| leaves, feathers, foreground branch | present | removed |
| parallax | subtle | off |
| weather | falling rain, rippling foliage | soft translucent overlay fading in/out; no streaks; foliage still |
| day/night | continuous | continuous, transitions slowed 2× |
| top bar fade | 1 s | 2 s, floor 60 % |
| settle | 4 s palette shift | 8 s palette shift |
| focus ring | static | static |

Calls, captions, narration, drift, mood, notebook, and every interaction behave identically. Visual regression tests capture both modes for every action so a new action cannot ship without its pose set.

### 7.9 Loading and empty states

- The quiet field: soft sky for the current local hour, a faint slow gradient drift and at most one leaf ornament, painted by the inline CSS/script before any bundle. It is also the empty-aviary state right after adoption. No spinner, no progress, no logo animation, no fade-from-static; the first bird simply appears mid-action (or, only in the post-adoption empty state, flies in to its perch; fades in under reduced motion).
- Adoption flow (`/adopt`, system voice for the form fields, naturalist voice for the two lines introducing the birds that arrived): two names with suggestions, one button. On submit the client navigates to the aviary; birds appear within seconds.

### 7.10 Interaction surfaces

- **Listen-in**: click/tap a bird, or keyboard focus + Enter. Visual: the focused bird turns slightly toward the viewer and the focus ring shows for keyboard users; there is no highlight glow, badge, or label in the scene. Audio per §8.5. Disengage: click the same bird, another bird, empty scene, Escape, or focus leaving the scene.
- **Offer sheet**: three rows in naturalist voice ("offer a seed", "offer a song", "offer a still pool"); song expands to eight named fragments ("dusk three", "low answer", …). When an offer is active the rows are present but quietly inert with the line "the aviary is still with the last offer"; no timer.
- **Settle**: the settle control lives inside the account/settings sheet's first row and as a keyboard shortcut (§9.3), keeping the top bar to its four icons (§15). Triggering closes the sheet and starts the palette shift; the 5 s undo is any click, tap, or key in the scene.
- **Notebook sheet**: virtualized list, newest first, entries grouped under lowercase weekday headings, infinite scroll backward. Read-only; no search, filters, or share.
- **Settings sheets**: system voice; account (email, sessions, export, delete/restore, invites, visit log, visit-notification toggle, privacy policy link) and accessibility (reduced motion, captions, calls on/off, narration verbosity: normal/quiet).

---

## 8. Audio pipeline

### 8.1 Graph

```
per bird (pool of 7 voices, allocated at boot, never freed):
  AudioWorkletNode(BirdVoice) ─► distance filter (BiquadFilter lowpass; back perch 3.5 kHz, middle 8 kHz, front open)
    ─► mix gain (listen-in) ─► StereoPanner (x of perch slot → pan ±0.6) ─► bird bus
offered-song voice (1) ─► song bus
ambient bed (AudioWorkletNode: filtered pink noise + slow LFO; rain/wind variants) ─► ambient bus
bird bus + song bus + ambient bus ─► DynamicsCompressor (soft, −12 dBFS ceiling, ratio 3:1) ─► master gain ─► destination
```

### 8.2 Synthesis

- `BirdVoice` worklet: two harmonic oscillators (fundamental + tilt-weighted partials), one noise source through a formant band-pass, per-syllable ADSR, f0 contour with optional vibrato, all parameters received as timestamped syllable messages. No allocation per call: syllable messages fill a fixed ring buffer.
- A call = motif syllables × signature transform × mood modulation × per-call seed jitter (±3–6 % on every parameter, distinct random phase per syllable). Two birds never share phase or exact timing, which removes the layered-loop comb-filter artifact the PRD warns about.
- The song library is eight motifs rendered by the same worklet with a softer, more harmonic timbre so it reads as "offered" rather than as another bird.

### 8.3 Scheduling from the call plan

The scheduler wakes every 250 ms, converts `call_plan.at` (server time) to `AudioContext.currentTime` using the clock offset, and posts syllable messages ≥ 300 ms ahead. Each scheduled call also triggers the performer's `call` action at the same timestamp so beak and sound align. Calls in the past by more than 1 s on arrival are skipped, never crammed.

### 8.4 Chorus mixing and loudness

Per-bird gain normalizes species loudness; the bus compressor keeps a seven-bird chorus from pumping; a chorus window adds +1 dB to joined voices and −1 dB to the ambient bed so the chorus reads as an event without a volume jump. Distance filtering by perch zone is what lets the ear separate seven signatures spatially as well as timbrally.

### 8.5 Listen-in mix and decay

- Engage: focused bird mix gain → 1.6 and its distance filter opens; all others → 0.35 with lowpass at 6 kHz; both by `exponentialRampToValueAtTime` over 1.5 s. The others never reach 0.
- Disengage: return to 1.0 / open over 2.5 s.
- Decay: after 8 min of listen-in with no pointer/key activity, the mix eases back toward ambient over 60 s while focus stays on the bird; any activity restores the listen-in mix over 1.5 s. Listen-in end is reported when focus actually leaves (the decay is a mix courtesy, not a state change) (§15).

### 8.6 Ambient bed, weather, settle, night

Ambient bed at −30 dBFS: filtered noise with a 0.05 Hz LFO. Rain: additional band-limited noise with sparse droplet impulses, intensity-scaled; wind: LFO-swept band-pass. Settle: master → 0.35 over 4 s. Night: ambient bed −6 dB, the night-active species' calls carry slightly more reverb (a short convolution-free feedback delay) to read as distance. All procedural; the audio bundle contains no samples.

### 8.7 Autoplay policy and fallback

- On boot the client creates the `AudioContext` and calls `resume()`. If the state is `running`, audio starts with the ambient bed fade. If `suspended` (browser gesture policy), calls are scheduled but silent, captions are shown regardless of the caption setting, and the first `pointerdown` or `keydown` resumes the context, ramps master from 0 over 2 s, and returns captions to the user's setting. No "enable sound" banner exists; the first gesture is also the first presence signal, so the two align naturally.
- If `AudioContext` construction fails, `AudioWorklet` is unavailable, or the context errors at runtime: the aviary runs in graceful silence with captions on by default and an aggregate `audio_context_error` metric increments. There is no recorded-audio path in the codebase; a CI check fails on any `.mp3/.ogg/.wav/.m4a` file or `<audio>` element.

### 8.8 Caption descriptors

Each synthesized call emits a descriptor `{syllables, contour: rise|fall|flat|mixed, tempo, loudness, pauses, role, mood, zone}` from the actual scheduled parameters. `packages/prose/captions` turns it into a short line ("a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch"). Captions are generated from what played, never stored per motif.

### 8.9 Memory and performance rules

Voice pool fixed at 7 + song + ambient; syllable ring buffers preallocated; descriptor objects pooled; scheduler holds ≤ 120 s of plan; `AudioContext` is created once per page and suspended (not closed) when hidden. The 30-minute memory test (§10.3) runs with audio on.

---

## 9. Accessibility surfaces

### 9.1 Screen-reader narration

- A single visually hidden `aria-live="polite"` region under the scene. The narration composer (`packages/prose/narration`) writes running naturalist prose from the same scene state the renderer draws: 1–2 birds per update chosen by rotation and recency, plus light, weather, and the season of the day.
- Cadence: idle updates every 30–60 s (jittered), never repeating the previous sentence's subject twice in a row, at most three sentences per update. User-initiated events (return-greeting, offer reaction, settle, listen-in engage) get a prompt update within 1 s, still written as observation: "pip lifts her head and calls once, then steps toward the front rail."
- Priority queue: event updates replace any pending idle update; idle updates are dropped, not queued, if the previous one is still being read (the region is cleared and rewritten, so the reader never backlogs).
- "Quiet" verbosity setting halves the idle cadence and drops weather lines.
- The composer never emits a mood word as a label ("wren is wary") but describes what the bird does ("wren watches from the back perch, upright, not calling"). Mood is inferred by the listener exactly as it is by the viewer.

### 9.2 Captions

Small text near the calling bird's screen position, fading with the call, WCAG AA against the scene via a soft backplate. When narration is active, captions carry `aria-hidden` so the call is described once (in narration) and not twice. Captions default on when audio is unavailable or suspended (§8.7).

### 9.3 Keyboard and focus

| Key | Context | Action |
|---|---|---|
| Tab / Shift+Tab | page | cycles top bar items, then into the scene (first bird), then out |
| ← / → | scene | move focus between birds in perch order (front-left to back-right) |
| Enter / Space | bird focused | toggle listen-in |
| Escape | listen-in / sheet | exit listen-in; close sheet |
| O | scene focused | open the offer sheet |
| N | scene focused | open the notebook |
| S | scene focused | settle (reversible by the same 5 s undo: any key or click) |
| Home / End | scene | first / last bird |

- Single-key shortcuts (O, N, S) are active only while focus is inside the scene region, which satisfies WCAG 2.1.4 without a remapping surface; the same actions are always reachable from the top bar by Tab.
- Birds are exposed as buttons in a DOM overlay positioned over their bounding boxes each frame (`role="button"`, accessible name "pip, front perch", `aria-pressed` for listen-in). The overlay is the single source of focus so focus never desyncs from the drawing.
- Focus ring: dual ring (2 px near-white inside, 2 px deep-brown outside, 3 px offset) which reads on bright noon sky and dim night; measured ≥ 3:1 against both extremes. It does not animate in any mode.
- No focus traps; sheets return focus to the invoking top-bar item; the greeting never steals focus.

### 9.4 Contrast

All user copy meets WCAG AA (4.5:1 text, 3:1 icons/controls). The top bar has an adaptive scrim that samples scene luminance beneath it each second and sets its opacity to hold the ratio at noon and midnight. Captions use a backplate; error lines use the scrim. A CI test renders the scene at 7 lighting keyframes and asserts computed ratios for every text style.

### 9.5 Reduced motion

§7.8 in full. The setting is honored within the current session and persisted to the account so both devices agree.

### 9.6 Settings surface

Accessibility settings (system voice): reduced motion (system/on/off), captions (on/off), calls (on/off), narration verbosity (normal/quiet). Plain form controls, plain labels, no naturalist phrasing.

### 9.7 Testing and acceptance

axe-core in Playwright on every surface; manual scripts for VoiceOver (Safari, iOS), NVDA (Firefox), and TalkBack (Chrome) run before each milestone exit; a narration transcript test asserts cadence bounds, no repeated subject, no mood labels, voice-linter clean; keyboard-only walkthrough of every interaction; reduced-motion visual snapshots for every action kind. Accessibility work is part of each milestone's exit criteria, not a later phase (§12.1).

---

## 10. Performance budgets and observability

### 10.1 Budgets

| Budget | Target | Where enforced |
|---|---|---|
| initial JS (everything executed before first bird) | ≤ 180 KB gz | size-limit in CI, per-PR diff comment |
| total JS shipped on the aviary page | ≤ 2 MB gz (PRD cap), internal target ≤ 900 KB | size-limit |
| time to first bird, mid-tier Android over throttled 4G | p75 ≤ 500 ms | Lighthouse-style synthetic run on every merge to main; RUM p75 alarm |
| idle frame time, five-year-old mid-range laptop (reference: 2021 dual-core i5 + integrated GPU, Chrome) | 60 fps sustained 30 min; JS ≤ 6 ms/frame; ≤ 40 draw calls | nightly Playwright soak on a dedicated runner |
| memory growth over 30 min | slope ≤ 50 KB/min and post-GC heap ≤ start + 5 MB | nightly 30-min soak; 5-min variant on every PR |
| snapshot size | ≤ 8 KB (7 birds, 120 s plans) | protocol fixture test |
| tick latency | p50 < 40 ms, p99 < 500 ms, alarm p99 > 5 s | sim metrics |
| event ingestion → consumed by tick | ≤ 90 s p99 | sim metrics |
| KV replication lag | ≤ 30 s p99 | edge metric |

### 10.2 How each budget is met

- **Bundle**: no framework on the critical path (the scene is imperative WebGL + a few DOM overlays; sheets use a small reactive layer loaded lazily); species rigs are code, not images; no webfont on the aviary page; audio, notebook, settings, adoption, invites, export are separate chunks; `@aviary/engine` on the client is tree-shaken to greeting + call-plan extension + prose.
- **First bird**: inline snapshot from KV; quiet field painted by inline CSS; meshes before textures; `/open` in parallel; `modulepreload`; HTTP/3 and immutable asset caching at the edge.
- **60 fps**: one atlas, one program family, instanced ornaments, pose math in typed arrays, `requestAnimationFrame` only while visible, DOM overlay updates batched once per frame via transforms (no layout).
- **Memory**: pools for ornaments, poses, descriptors, syllables; notebook virtualization releases DOM rows; audio graph fixed at boot; snapshot store keeps only the current and previous snapshot; no per-frame closures in hot paths (lint rule on the renderer package).

### 10.3 CI gates (block merge)

size-limit; engine golden + property tests; calibration harness personas; presence conjunction e2e; voice linter; no-announcement-primitives lint; no-audio-files check; protocol fixture "no raw trait" test; axe on every surface; reduced-motion snapshots; 5-min memory test; layout "no bird clipped" test; contrast keyframe test. Nightly: 30-min memory and fps soak, synthetic first-bird runs from three geographies.

### 10.4 What we measure (aggregate only)

Edge/API/sim: request counts, latencies, error rates by route and status; tick latency and backlog (aviaries overdue > 2 min); event ingestion lag; clipped-presence and clamped-timestamp counts; KV lag; mailer delivery/bounce rates; magic-link request/consume ratios; rate-limit hits. Client RUM (sampled, no account dimension): first-bird time, frame time p95, long-task count, `audio_context_error`, worklet load failure, WebGL2 unavailable, snapshot 304 ratio, session-duration histogram (bucketed, anonymized). Synthetic: scripted browsers from common geographies opening a synthetic account's aviary on a schedule.

### 10.5 What we deliberately do not measure

Anything per account or per bird beyond operational errors: no interaction funnels, no offers-per-user, no listen-in durations, no greeting outcomes, no visit counts as a product metric, no retention cohorts keyed to interaction behavior, no "average drift" dashboards. The drift ledger exists for calibration and is read only for synthetic accounts and internal dogfood accounts that opted in; production accounts' ledgers are never queried in aggregate. Enforcement is structural (§10.7): the analytics warehouse has no credential to the sim database and the exporter can only emit allowlisted metric names and labels.

### 10.6 Alarms

tick p99 > 5 s (page); tick backlog > 1 % of active aviaries (page); event lag p99 > 90 s; API 5xx > 0.5 % over 5 min; snapshot endpoint p95 > 300 ms; first-bird RUM p75 > 500 ms mobile cohort (ticket); frame time p95 > 20 ms (ticket); `audio_context_error` rate > 2 % of sessions; KV lag > 90 s; mailer failure > 2 %; hard-delete job verification failure (page).

### 10.7 Logging and the privacy boundary

Structured logs pass through an allowlist serializer: permitted fields are request id, route, status, latency, `account_id` (UUID), `session_id`, error class, and enumerated codes. Any other field is dropped at the logger, so an engineer cannot accidentally log an email, a name, a mood, or a trait; a unit test feeds a full bird row and asserts the output contains only the allowlisted keys. Emails are never in URLs (tokens are opaque). Traces carry no payloads. The telemetry package has no import path to the database layer (enforced by a dependency-cruiser rule).

---

## 11. Privacy and security notes

- Email: encrypted at rest with envelope encryption; HMAC blind index for login and invite lookup; decrypted only to send mail and to display in the account and visit-log surfaces.
- Tokens: magic links and invite links are 256-bit random, stored hashed, single-use, expiring (15 min / 30 d); session tokens revocable per device; edge hint cookie is short-lived and carries no PII.
- CSRF double-submit on state changes; `SameSite=Lax`; strict CSP (no inline scripts except the hashed quiet-field script); no third-party scripts on the aviary page.
- Abuse: per-email and per-IP throttles on magic links and invites; invite emails name the host only as the host entered it; a visitor can report an unwanted invite via a link in the email that blocks that host from inviting that address again.
- Export contents: birds (id, name, species, adopted date), notebook entries, settings, visit log, invites, and an opaque, versioned, signed `engine_state` blob per bird that carries the personality vector for portability without rendering it as labeled numbers (§15). Re-import is not a v1 feature; the blob exists so nothing about the bird is lost.
- Deletion: soft 30 d with restore, then verified hard delete (§3.7); backups age out within 30 d after.

---

## 12. Rollout

### 12.1 Milestones and exit criteria (≈ 22 weeks to general availability)

| # | Milestone | Weeks | Exit criteria |
|---|---|---|---|
| M0 | foundations | 1–2 | monorepo, CI gates skeleton, protocol schemas, Postgres schema + roles + triggers, edge shell serving the quiet field, voice linter running on an empty prose package |
| M1 | engine core | 2–6 | `tick()` with presence, drift, attunement, mood, perch, weather, call planner; golden + property tests; calibration harness green for the three personas; single-writer lock; active/dormant cadence; catch-up on open |
| M2 | first bird | 3–8 | renderer, rigs for six species, performers, inline snapshot, `/open`, greeting selection; first bird ≤ 500 ms on the reference device with 2 birds; no-spinner boot verified by a video-frame test |
| M3 | audio | 4–10 | worklet synth, signatures, chorus, listen-in mix, ambient bed, autoplay handling, captions from descriptors, no-audio-files check; internal listening session confirms two birds are distinguishable by ear after a day |
| M4 | interactions and prose | 7–12 | listen-in, offers with reaction plans, settle + undo, notebook generation with sparsity, narration composer; voice linter clean; keyboard model complete |
| M5 | accounts, sync, visits | 6–12 | magic link, sessions, email change, export, soft/hard delete with verification job, invites, visitor sessions, visit log, opt-in notice; two-device sync tests; presence union test |
| M6 | accessibility and performance hardening | 10–15 | reduced-motion pose sets for every action; SR scripts pass on VoiceOver/NVDA/TalkBack; contrast test; 30-min memory and fps soaks green; bundle under internal target |
| M7 | dogfood and calibration | 13–18 | ≥ 30 internal accounts for ≥ 4 weeks; drift instruments confirm week-1 measurable, week-3 visible on real usage; greeting never reported as canned in weekly reviews; tick p99 within budget at 10× synthetic load; aged-aviary cohort validates 3–7 birds |
| M8 | closed beta → GA | 18–22 | invite-only beta (a few hundred accounts) two weeks; alarms quiet; hard-delete drill; privacy policy published; feature flags removed or defaulted; GA |

### 12.2 Team shape

Eight engineers plus a visual designer and a writer for the prose kit: 2 engine/backend, 2 rendering/animation, 1 audio, 1 platform/infra (edge, sim ops, mailer, jobs), 1 accessibility + client integration, 1 lead who also owns the guardrail tooling. The writer owns templates and the denylist with the lead.

### 12.3 Launch sequence and flags

Flags (server-side, per account, removed by GA): `visits`, `newcomers`, `weather`, `notebook_generation`, `narration`, `song_offers`. Everything ships dark to dogfood first, then to beta, then flags default on. There is no marketing surface inside the product; the launch is a sign-in page and a link.

### 12.4 Bird ramp

Every aviary starts at two; the engine supports seven from M1. Real users cannot reach bird 3 before day 90, so the ramp is validated ahead of time with an internal "aged aviary" flag that back-dates `aviaries.created_at` for dogfood accounts, letting us exercise newcomers, the naming sheet, chorus at 4–7 birds, layout at seven on narrow viewports, and the listening study before any real user gets there. If the study fails recognizability at seven, the newcomer schedule (§14) is stretched by config, not by changing the engine.

### 12.5 Instrumented from day one

All §10.4 metrics and §10.6 alarms exist before dogfood, plus the calibration instruments (drift ledger) for synthetic and opted-in dogfood accounts, the presence-clipping counter, the greeting-form distribution (fleet aggregate from `greeting_shown` types, no account dimension), and notebook entries-per-week histogram (aggregate) to confirm sparsity.

### 12.6 Ops essentials

Runbooks for: tick backlog (scale workers; verify lock TTL), KV lag (fall back to no-inline; first-bird degrades gracefully), mailer outage (magic links queue, sign-in page shows a matter-of-fact delay note), Postgres failover (ticks are idempotent and resume from `next_tick_at`), engine version bump (migration job with `engine_migration` session flag; never lowers traits; never changes bird ids), hard-delete verification failure (page, halt job, investigate).

---

## 13. Risks and mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| **Drift calibration off** (too fast → Tamagotchi; too slow → screensaver) | the core promise fails silently | constants in one table; calibration harness in CI with named personas; instruments on dogfood accounts; caps make "too fast" impossible past 0.05/week; band-crossing makes "visible" a measurable event |
| **Presence inflation** (a laxer conjunction, mobile activity semantics, heartbeat bugs) | fleet-wide drift corruption, invisible per account | e2e conjunction test; server-side clipping and union; clipped-presence fleet metric; touch counts as activity, scroll does not exist; activity window is the longer end (4 min) so watching without moving still counts |
| **Sync correctness** (duplicate/out-of-order events, clock skew, two writers) | lost drift, divergent devices | idempotent ULIDs, server `seq`, single writer under fenced lock, additive deltas, clamped timestamps, two-device test suite, presence union |
| **Audio uncanniness** (synth sounds like a synth; chorus blurs; mobile latency; autoplay) | the affective spine breaks | signature invariants; per-call jitter and phase; distance filtering; compressor; listening studies at 2/4/7; autoplay handled without a banner; captions as a first-class fallback |
| **Recognizability at seven** | per-bird relationship collapses | study before day 90; newcomer schedule configurable; cap enforced in engine |
| **Greeting reads as canned** | first-session staleness | forms × bucket × mood × seed variation; distribution metric; dogfood weekly review of "did it feel canned" as a named question |
| **Accessibility regression** | product rationed by sensory ability | reduced-motion pose sets required per action; narration transcript tests; SR scripts at each milestone; axe in CI |
| **Announcement creep** (a "small toast") | breaks notice-never-announce | no announcement primitives; lint; PR checklist |
| **Trait leakage** (debug view, export, snapshot) | bird becomes a number | DB roles; protocol schema; fixture test; opaque export blob |
| **Tick cost at scale** | budget blowout | dormant coalescing (15 min), catch-up on open, idempotent ticks, load test at 10× |
| **Email deliverability / magic-link abuse** | users cannot sign in | reputable provider, bounce handling, throttles, matter-of-fact delay copy |
| **PII in logs** | compliance finding | allowlist logger, blind indexes, no email in URLs, schema and log tests |
| **Notebook repetition or voice drift** | spell breaks in the most concentrated voice surface | template variety per kind, seed-driven phrasing, repetition guard (no same template within 5 entries), writer review of every template, linter |
| **Timezone changes / travel** | odd mood priors for a day | last-reported timezone wins; lighting is client-local anyway |
| **Visitor link forwarding** | unintended viewer | token consumed on first use and bound to one browser; host revokes; visit log shows the email |
| **Hard delete incompleteness** | privacy breach | dependency-ordered job with zero-row verification and paging alarm |
| **WebGL2 or worklet unavailable on a supported browser** | broken scene or silence | Canvas2D correctness fallback; silence + captions; fleet metrics to see how often |

The four the PRD calls out by name (drift calibration, sync correctness, audio uncanniness, accessibility regression) each have a CI gate, a fleet metric, and a named dogfood review question, so none of them can fail silently.

---

## 14. Calibration constants v0 (single source of truth)

| Constant | Value | Section |
|---|---|---|
| tick step | 60 s | §5.1 |
| active cadence window | client seen within 24 h → 60 s ticks; else 15-min ticks in 60 s sub-steps | §5.1 |
| catch-up on open | synchronous, ≤ 24 h of sub-steps; fast-forward beyond | §5.1 |
| presence activity window | 240 s (calibrate 180–300 s) | §5.3 |
| presence heartbeat / max interval | 30 s / 45 s | §5.3 |
| trait range / seed range | [0, 1] / 0.3–0.5 with species offsets ± 0.1 | §5.4 |
| α (per presence-hour): boldness, warmth, vocal, plumage, curiosity | 0.045, 0.045, 0.035, 0.050, 0.030 | §5.4 |
| β (per listen-in-hour, focused bird): warmth, vocal | 0.060, 0.060 | §5.4 |
| γ_accept (curiosity) / γ_near (boldness) per offer | 0.003 / 0.001 | §5.4 |
| cap per trait per local day / per 7-day window | 0.012 / 0.050 | §5.4 |
| measurable drift / visible drift | Δ ≥ 0.01 / one band crossing (12 bands, 0.083) | §5.4, §4.8 |
| attunement: rise per presence-hour, decay τ, floor | 0.8·(1−a), 10 days, 0.35 | §5.5 |
| mood hysteresis / dwell / switch probability | 0.15 / 8 min / 0.35 per step | §5.6 |
| perch move threshold / dwell / max movers per step | 0.2 / 6 min / 1 | §5.7 |
| rain: rate, duration, call multiplier during/after | 3/week, 3–8 min, 0.4 / 0.7 for 10 min | §5.8 |
| wind: rate, duration | 5/week, 2–5 min | §5.8 |
| call-plan horizon / preserved head | 120 s / 30 s | §5.9 |
| response delay / probability base | 0.4–1.5 s / 0.6·warmth | §5.9 |
| chorus window / join threshold | 20 s / vocal ≥ band 7 to seed, ≥ band 5 to join | §5.9 |
| greeting absence buckets | < 10 min, < 6 h, < 2 d, ≥ 2 d | §5.11 |
| greeting start offset / follower stagger | 600–1500 ms / 0.8–2.5 s | §5.11 |
| offer durations: seed, song, pool | 120 s, ≈6 s, 180 s | §5.12 |
| per-bird offer cooldown | 4 min | §5.12 |
| settle undo window (client / server) | 5 s / 7 s | §5.13 |
| settled call multiplier / master gain | 0.3 / 0.35 | §5.13, §8.6 |
| newcomer eligibility (bird 3…7) | 90, 180, 300, 450, 630 days | §5.14 |
| newcomer stay / decline backoff | 14 d / 30 d | §5.14 |
| notebook bucket capacity / refill / score threshold | 2 / 1 per 72 h / 0.6 | §5.15 |
| listen-in ramps in / out; gains focused / others | 1.5 s / 2.5 s; 1.6 / 0.35 | §8.5 |
| listen-in mix decay | after 8 min inactivity, 60 s ease | §8.5 |
| narration idle cadence | 30–60 s | §9.1 |
| top bar fade delay / floor | 4 s pointer, 6 s touch / 12 % (60 % reduced motion) | §7.7 |
| sunrise / sunset | 06:30 / 19:30 ± 45 min by month | §7.6 |
| keepalive pull | 60 s ± 5 s | §6.2 |
| magic link / invite / visitor session / export link TTL | 15 min / 30 d / 30 d / 7 d | §4 |
| soft-delete window | 30 d | §3.7 |

---

## 15. Decisions on ambiguities (with reasoning)

1. **Export vs. "never see the numbers."** `accounts_sync.md` lists personality vectors in the export; `concepts.md` (which wins by its own rule) says the user never sees the numbers. Decision: the export carries each bird's vector inside an opaque, signed, versioned `engine_state` blob, not as labeled fields. The user's data is complete and portable; no product surface renders a trait number.
2. **Bird-level "settled" vs. aviary-level settled.** The PRD uses "settled" for both the evening lighting state and a sleeping bird. Internally the bird mood is `roosting`; user-facing prose still says things like "settled low on the perch." No user surface shows a mood enum.
3. **How neglect makes birds "quieter" with monotonic traits.** Added a hidden fast-timescale `attunement` value that decays with absence and recovers quickly with presence. Personality never moves down; expressiveness dips and returns.
4. **Mute and drift.** `product_brief.md` lists muting among things birds respond to; `bird_engine.md`'s drift inputs do not include it. Decision: calls on/off has no drift effect (anything else would punish a user who needs silence). Listen-in still counts while muted because it is attention.
5. **Presence activity window** set to 240 s, the long end of "a few minutes," because watching without moving is the product. `keydown`, `pointermove`, `pointerdown`, and `touchstart` count; nothing else does.
6. **Fast-path mood updates outside the tick.** Offers and settle apply mood effects immediately, server-authored under the same lock, so reactions are instant on the acting device and consistent everywhere. Personality is still tick-only.
7. **Dormant coalescing.** Ticking every minute for every account forever is unaffordable; dormant aviaries tick every 15 min in 60 s sub-steps with identical semantics, plus synchronous catch-up on open. The state the user sees is exactly what per-minute ticking would have produced.
8. **Cross-device propagation latency.** Polling at 60 s is the PRD's model; a push channel is not needed for v1 and is left as a possible later optimization. Both devices always converge on the same canonical state.
9. **Where settle lives.** The top bar is specified as exactly four icons; settle is described as "from the top bar." Decision: settle is the first row of the account/settings sheet plus the `S` shortcut, which keeps the bar at four icons and settle one tap away. Easy to move to a fifth icon if design prefers.
10. **Visitor link lifetime.** "One-time link" plus "active invite": the token is consumed on first use, creating a 30-day visitor session bound to that browser; the invite is revocable throughout; a new browser needs a new invite.
11. **Newcomer surface.** The offer is the bird itself on the back perch for up to 14 days, with a naming sheet on focus; no prompt or badge. Declining or ignoring has no consequence beyond a 30-day pause.
12. **Starter species.** Two distinct diurnal species; the night-active species enters the pool from bird 3 with elevated weight, so first encounters happen in daylight with two active birds while "night is not a dead state" still arrives within months.
13. **Sunrise/sunset** are a fixed local schedule with a modest seasonal tilt rather than geolocation, avoiding a location permission and keeping a testable function; the server uses the same function with the account's timezone.
14. **Listen-in decay** (mix easing back after 8 min of inactivity) is an interpretation of "listen-in mix decay" in the planning brief; the state stays engaged and any activity restores it.
15. **Rendering stack** is an in-house WebGL2 batcher with a Canvas2D correctness fallback, chosen for the bundle budget and shader-driven day/night tinting; a general-purpose engine would cost budget and control.
16. **English only** at v1: the prose kit's voice rules are language-specific; the architecture keeps templates in one package so localization is additive later.
17. **Autoplay.** Browsers block audio before a gesture; the aviary starts silent with captions visible and ramps audio on the first gesture rather than showing an "enable sound" banner.
18. **Timezone** is last-reported by any signed-in device; a traveling user's moods follow the device they used most recently.
19. **Aviary catch-up bound** of 24 h of sub-steps synchronous, beyond which a proven fast-forward applies the same functions; both paths are covered by golden tests to be identical.

---

## 16. Definition of done for v1

- Every §1.1 row is shipped behind no flag; every §1.2 item is absent and its guardrail (§1.3) is green.
- All §10.3 CI gates pass; nightly soaks have been green for two consecutive weeks; all §10.6 alarms exist and are quiet.
- Calibration harness and dogfood instruments both show measurable drift at one week and visible drift at three for the regular persona, and zero drift during absence.
- Presence conjunction e2e test passes on Chrome, Safari, Firefox, Edge, iOS Safari, and Android Chrome.
- SR scripts pass on VoiceOver, NVDA, TalkBack; reduced-motion snapshots exist for every action kind; contrast test passes at all lighting keyframes.
- A fresh account reaches first bird within 500 ms on the reference mobile profile, sees a greeting within 2 s, and never sees a spinner, a toast, a counter, or a number about a bird.
- Hard-delete drill verified to zero rows; export produced and emailed; privacy policy live with the aggregate categories named and per-bird interaction state excluded.
- Two devices signed into one account show the same birds, moods, perches, weather, and calls, and an offer on one is visible on the other within one keepalive.
