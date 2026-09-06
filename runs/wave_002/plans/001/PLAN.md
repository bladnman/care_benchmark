# Pocket Aviary — v1 Implementation Plan

- **Slot:** `runs/wave_002/plans/001/` · **Phase:** 1 (planning only — no product code is written here)
- **Source spec:** `prd/` (product_brief, concepts, bird_engine, interactions, aviary_layout, accounts_sync, social_optional, accessibility_perf, non_goals)
- **Audience:** the engineering team executing v1. This plan is intended to be executable without further clarification; every ambiguity encountered in the PRD is resolved with a defensible call and logged in §17.

---

## 1. Executive summary

Pocket Aviary is a browser-only ambient virtual aviary: 2 starter birds growing to a hard cap of 7 by aviary age, living in a single horizontal scene that is always already in motion. The product's spine is a **server-authoritative simulation** (a ~1×/minute tick per aviary that runs whether or not anyone is watching) and a **thin, stateless client** that renders snapshots and appends interaction events. Personality drifts slowly and monotonically toward expressive on honest presence signals; mood moves on a daily cadence; calls are procedurally synthesized client-side via WebAudio from server-scheduled call specs; the field notebook and screen-reader narration are generated from one shared naturalist-voice module.

The build decomposes into nine workstreams:

1. **`sim-core`** — shared TypeScript library: drift, mood, behavior planning, call grammar/scheduling, greeting resolution, offer-reaction resolution, weather, chronicler (notebook), narrator. Used by the API service, the tick worker, and the calibration harness so behavior rules exist exactly once.
2. **API service** — snapshot delivery, event ingestion, notebook, accounts, visits.
3. **Sim worker** — tick scheduler and executor; pushes post-tick snapshots to edge cache.
4. **Web client** — canvas scene renderer + DOM chrome (React), boot path optimized for first-bird < 500 ms.
5. **Audio engine** — AudioWorklet multi-voice procedural synth, chorus mixing, listen-in ramps, caption generation from call specs.
6. **Accessibility layer** — narration live region, captions, keyboard/focus system, reduced-motion renderer (a designed surface, not a fallback).
7. **Accounts & privacy infrastructure** — magic-link auth, sessions, export, deletion, encrypted email with blind-index lookup, hard telemetry boundary.
8. **Visits** — invite/redeem/revoke/log flow and the read-only visitor client mode.
9. **Observability & calibration** — CI budget gates, synthetic fleet, aggregate-only RUM, and the accelerated-clock drift-calibration harness that owns the 1-week/3-week drift targets.

**Load-bearing invariants** (violating any one breaks the product; each is enforced structurally, not by policy — see the "enforcement" column):

| # | Invariant | Enforcement |
|---|-----------|-------------|
| I1 | The server is the only writer of canonical state; clients only append events and render snapshots. | No API endpoint accepts state values; contract tests assert. |
| I2 | Personality vectors never leave the server in raw form (sole exception: the user-pulled account export). Clients receive derived render/synthesis params. | API schema review; snapshot serializer whitelists fields; lint. |
| I3 | Drift is monotonic toward expressive. No code path decreases a trait, ever. | Trait columns have a CHECK-free but code-enforced clamp; property test: no input sequence produces a negative delta; tick-level canary alarm on any observed decrease. |
| I4 | Presence is the honest triple condition (visible ∧ focused ∧ recent pointer/key activity). | Client monitor unit + e2e tests; server-side daily sanity clamp and alarm. |
| I5 | No announcement surfaces: no toasts, banners, badges, "welcome back", streaks, or visit-frequency anywhere. | The design system ships **no toast/banner/badge primitive**; copy-lint CI forbids the phrase families; PR checklist item. |
| I6 | All audio is procedural. No recorded audio assets exist. | CI asset lint fails the build on any audio file in `apps/web` output. |
| I7 | The aviary's first frame has motion already in progress — no spinner, no entry animation (sole exception: the one-time post-signup fly-in). | e2e asserts no spinner role and first-bird paint from an in-progress activity phase. |
| I8 | Bird identity is permanent: stable UUID for life; rename/species-pool changes never replace a bird. | `birds.id` immutable; migration invariant tests. |
| I9 | Telemetry never contains per-account or per-bird dimensions; telemetry stores are physically separate from the simulation DB. | Separate credentials/datastores; schema-registry gate fails CI on forbidden columns. |
| I10 | All ambient randomness is deterministic from `(aviary_id, time-bucket, purpose)` seeds, making ticks catch-up-safe. | Property test: 1-minute ticking ≡ hour-late catch-up within ε. |

---

## 2. Scope

### 2.1 In v1

- Single-user accounts; email **magic-link** sign-in (no passwords, no SSO); per-device revocable sessions; email change with verification.
- One canonical aviary per account; 2 starter birds; cap 7; new-bird availability gated on **aviary age** (never on engagement or payment).
- Server-side simulation tick (~1×/minute, catch-up-safe) advancing mood, behavior plans, call schedules, weather, drift, notebook, narration — with or without connected clients.
- Client snapshot pulling with interpolation; append-only interaction event log (presence pings, listen-in, offers, settle, renames, adoption).
- Interactions: return-greeting (procedurally varied, absence-shaped), listen-in (gradual mix re-balance, others never silent), offers (seed / song fragment / still pool, per-bird cooldown), settle (with 5-second undo), field notebook (auto-generated, read-only, sparse, indefinite scrollback).
- Scene: single horizontal screen, three perch zones, user-local-time day/night, rare ambient weather, leaf/feather drift, subtle parallax, no in-scene UI chrome, top bar (account/settings, accessibility, notebook, offer, settle) that fades on cursor stillness.
- Multi-device sync as an architectural property (one canonical record; no client-side state to merge).
- Visits: host-invited, per-invite opt-in, read-only ambient view; revocable; 30-day invite expiry; silent visit log; visit notifications off by default.
- Accessibility as designed surfaces: naturalist screen-reader narration (30–60 s idle cadence), call captions generated from the live call grammar, reduced-motion cross-fade renderer, full keyboard navigation, WCAG AA contrast on all user copy.
- Performance: < 2 MB gz initial JS; first bird visible < 500 ms (mid-tier mobile / 4G); 60 fps idle on a 5-year-old laptop sustained 30 min; zero client memory growth over 30 min (CI test); graceful silence + captions when WebAudio is unavailable.
- Account export (JSON, emailed link; includes personality vectors per PRD), soft-then-hard deletion (30 days).
- Aggregate-only operational telemetry + synthetic perf fleet; tick-latency p99 > 5 s alarm.

### 2.2 Out of v1 (non-goals, respected verbatim)

- Native apps of any kind; the data model and protocols are **not** designed around native-client constraints.
- All gamification: achievements, streaks, levels, scores, badges, XP, ranks, tiers, "birds adopted: N", green-dot calendars, "you've been here every day this week" — including quiet/opt-in variants and notebook disguises. The notebook observes the aviary, never the user's visit behavior.
- Tamagotchi mechanics: no death, no hunger, no distress, no decaying happiness meter. Neglect produces ambient quietness (a fast-timescale expressiveness modulation), never trait decay and never visible suffering.
- Social-network surfaces: no profiles, follows, feeds, discovery, comments, co-presence, leaderboards, or show-off rendering. Visits are the entire social feature.
- Payments, shared aviaries, multi-aviary accounts, customizable scenes, push notifications, any email about aviary activity except the opt-in visit-notification toggle and transactional account emails.
- Recorded audio in any form, at any quality, anywhere — including as a fallback.
- Numeric personality exposure in any product surface, debug view, or tier (export excepted, §11.4).

### 2.3 Definition of done for v1 (acceptance sketch)

- First session: aviary paints mid-motion < 500 ms on the reference throttled profile; one bird greets within ~1–2 s, procedurally varied; no spinner or welcome text exists in the DOM at any point.
- Week-3 simulated regular-use persona shows ≥ 0.04 cumulative trait movement with perceptible behavior change; single sessions move traits < 0.002 (below instrument noise for users).
- A two-week-absent persona returns to quieter-but-unchanged birds: zero negative trait deltas, greeting expressiveness modulated by presence recency.
- Screen-reader pass (VoiceOver/NVDA/JAWS) hears naturalist prose at ≤ ~2 lines/min idle, with prompt narration of greeting/offer/settle.
- Two devices signed into one account show the same aviary, same mood, same drift; no merge code exists.
- All CI gates green: bundle, first-bird, frame timing, memory soak, axe, contrast, audio-variation, drift monotonicity, catch-up equivalence, telemetry-schema lint, no-audio-assets lint, copy lint.

---

## 3. Architecture

### 3.1 Component diagram

```
                         ┌──────────────────────────────────────────────┐
                         │ CDN edge (Cloudflare)                        │
 Browser ── HTTPS ──────►│  • static app shell (HTML/CSS/critical JS)   │
                         │  • Worker: verify session JWT at edge,       │
                         │    inline snapshot JSON from KV into HTML    │
                         │  • KV: latest snapshot per aviary (pushed    │
                         │    by sim worker after each tick)            │
                         └───────────────┬──────────────────────────────┘
                                         │ origin calls (API)
                         ┌───────────────▼──────────────┐
                         │ api service (Node/Fastify)   │
                         │  /v1 snapshot, events,       │
                         │  notebook, auth, account,    │
                         │  visits, narration           │
                         │  uses sim-core for greeting  │
                         │  + reaction resolvers        │
                         └──────┬───────────────┬───────┘
                                │               │
                   ┌────────────▼───┐      ┌────▼──────────────────────┐
                   │ Postgres 16    │      │ Redis                     │
                   │ (single store: │      │ • due-aviary ZSET (ticks) │
                   │  accounts,     │      │ • rate limits             │
                   │  aviaries,     │      │ • mailer queue            │
                   │  birds, events │      └───────────────────────────┘
                   │  (partitioned),│
                   │  ticks,        │      ┌───────────────────────────┐
                   │  notebook,     │◄─────│ sim worker (Node)         │
                   │  visits, …)    │─────►│ • tick loop (batched,     │
                   └────────────────┘      │   sharded) via sim-core   │
                          ▲                │ • chronicler + narrator   │
                          │                │ • snapshot → edge KV push │
                   ┌──────┴─────────┐      │ • weather (seeded)        │
                   │ mailer worker  │      └───────────────────────────┘
                   │ (transactional │
                   │  email only)   │      ┌───────────────────────────┐
                   └────────────────┘      │ Telemetry (SEPARATE):     │
                                           │ RUM vendor + ClickHouse;  │
                                           │ no account/bird dims;     │
                                           │ separate credentials;     │
                                           │ no network path to PG.    │
                                           └───────────────────────────┘
```

### 3.2 Service shape and tech choices (with rationale)

- **TypeScript everywhere.** `sim-core` must run identically inside the API (fast-path resolvers), the sim worker (ticks), and the calibration harness; one language and one package makes behavior-rule divergence structurally impossible.
- **Node.js 22 + Fastify** for api and workers. The workload is I/O-bound and small-compute; no need for a second backend language.
- **Postgres 16 as the only datastore.** Scale math: the tick is ~1×/min per aviary and events are a few per active session-minute; 100 k accounts ⇒ well under 1 k events/s and ~1.7 k ticks/s worst case (§6.2 batching + dormant tiering brings this to ~50 worker-cores). The PRD's Kafka mention is a PII-leak example, not a requirement; an append-only Postgres table with monthly partitioning gives the same ordering guarantees with far less machinery. Revisit only if > 500 k active aviaries.
- **Redis** for the due-tick ZSET, rate-limit counters, and the mailer queue. No durable state lives in Redis; loss degrades cadence, never correctness (ticks are catch-up-safe by I10).
- **Cloudflare Worker + KV at the edge** to hit the 500 ms first-bird budget: the sim worker pushes each aviary's post-tick snapshot JSON to KV; the edge Worker verifies the session JWT (signature-only check plus a ≤ 60 s-TTL revocation cache) and inlines the snapshot into the static HTML shell. No origin round-trip on the critical path.
- **Client:** React 18 for DOM chrome only (top bar, dialogs, settings, notebook, auth pages — all code-split); a **custom canvas-2D renderer** (no game engine) for the scene; an **AudioWorklet** synth. The render loop never touches React state.
- **Deploy:** containers on a PaaS (Cloud Run/Fly-class), blue-green; client assets are immutable-versioned static files (instant rollback); API is backward compatible N−1 against deployed clients.
- **Environments:** `dev`, `staging` (with an accelerated-clock mode used by the calibration harness — the tick reads wall time through a `Clock` interface injected everywhere in `sim-core`), `prod`.

### 3.3 Monorepo layout

```
apps/
  web/            # SPA + edge shell template
  api/            # HTTP surface
  sim-worker/     # tick scheduler/executor, KV push, purge jobs
  mailer/         # transactional email consumer
packages/
  sim-core/       # ALL behavior rules: drift, mood, planner, grammar,
                  # greeting/reaction resolvers, weather, chronicler, narrator
  voice/          # naturalist prose grammar shared by chronicler/narrator/captions
  render-engine/  # canvas scene graph, bird rigs, reduced-motion renderer
  audio-engine/   # AudioWorklet synth, scheduler, mix, caption-spec mapping
  presence/       # client presence monitor (triple condition) + ping queue
  config/         # server-side tunables registry (§15.2, Appendix E)
  telemetry/      # aggregate-only RUM client (schema-gated)
  db/             # migrations, repositories
tooling/
  calibration/    # accelerated-clock persona harness (§14.2)
  synthetic/      # perf fleet scripts
```

### 3.4 Client/server split and the render-pipeline boundary

- **Server owns:** everything canonical — personality vectors, mood, behavior plans (activity windows with absolute timestamps), call schedules (specs, not audio), weather, day phase, notebook entries, narration lines, greeting selection, offer-reaction selection, cooldowns.
- **Client owns:** rendering and synthesis only — interpolating activity phases against a clock offset, drawing, synthesizing audio from call specs, generating caption text from the specs it just played (the caption grammar is a pure function of the spec), ambient ornaments (leaves/feathers — explicitly *not* tick-driven), focus proxies, and the presence monitor.
- **The boundary rule:** the client never decides *what a bird does*; it decides *how the thing the server scheduled looks and sounds right now*. The single sanctioned exception is idle filler when the plan horizon runs out before the next snapshot (§8.5) — filler is restricted to low-commitment visuals (scan, weight-shift) and is always overridden by the next plan.

---

## 4. Data model

All internal references key on synthetic UUIDs (I9/§11.1). Email exists in exactly two columns of `accounts` (ciphertext + blind index) and one of `visit_invitations`; nowhere else — not in logs, events, metrics, queue payloads, or error messages.

### 4.1 Tables

**accounts**
| column | type | notes |
|---|---|---|
| id | uuid PK | synthetic, generated at creation |
| email_enc | bytea | AES-256-GCM under KMS key |
| email_idx | bytea | HMAC-SHA256 blind index (unique) for lookup |
| timezone | text | IANA; updated from active client at session_open (§6.9) |
| created_at / deleted_at / hard_delete_at | timestamptz | soft delete 30 d → purge job |
| settings | jsonb | `{reduced_motion: bool?, captions: bool, narration_visible: bool, muted: bool, visit_notifications: bool}` (null reduced_motion = follow OS) |

**sessions** — `id uuid PK, account_id FK, token_hash, device_label text, created_at, last_seen_at, revoked_at, expires_at` (30-day sliding).

**magic_links** — `id, email_idx, token_hash, created_at, expires_at (15 min), consumed_at` (consumption is an atomic `UPDATE … WHERE consumed_at IS NULL`).

**email_verifications** — for email-change flow: `id, account_id, new_email_enc, new_email_idx, token_hash, expires_at, confirmed_at`.

**aviaries** — `id uuid PK, account_id FK unique, created_at (drives bird-slot offers), state_rev bigint (bumped on every canonical change; snapshot ETag), last_tick_at, last_tick_id, rng_epoch int (bumped only by explicit migration, never casually)`.

**birds**
| column | type | notes |
|---|---|---|
| id | uuid PK | **immutable for life** (I8) |
| aviary_id | uuid FK | |
| species_id | text | FK to static species config |
| name | text | user-assigned; renameable; default suggestions at adoption |
| adopted_at | timestamptz | |
| boldness / social_warmth / vocal_frequency / plumage_saturation / curiosity | real | range [0.10, 1.00]; seed at adoption = species baseline + N(0, 0.05) clamped |
| mood | text enum | `wary | alert | content | curious | drowsy | roosted` (§6.4) |
| mood_since | timestamptz | |
| perch_zone | smallint | 0=back, 1=middle, 2=front |
| perch_x | real | 0..1 along zone |
| plan | jsonb | current activity plan horizon (§6.5) |
| plan_until | timestamptz | |
| last_greeted_at | timestamptz | |
| offer_cooldown_until | timestamptz | per-bird, all offer kinds share it |
| signature | jsonb | **call signature, derived once at adoption from `hash(bird_id)`, immutable**: `{pitch_center, timbre: {brightness, vibrato, attack}, tempo_bias}` — invariant across mood and drift (recognizability) |

Personality columns are written **only** by the tick code path (single repository method `applyDriftDelta`, guarded by a runtime assertion that the caller holds a tick context; deltas are clamped ≥ 0 per I3).

**events** (append-only; monthly partitions; the only client write surface)
| column | type | notes |
|---|---|---|
| aviary_id | uuid | partition-adjacent index `(aviary_id, seq)` |
| seq | bigint | per-aviary monotonic, assigned at insert |
| idem_key | uuid | client-generated; unique per aviary (dedupe retries) |
| type | text | taxonomy in Appendix B |
| bird_id | uuid null | |
| client_ts / server_ts | timestamptz | late window: server_ts − client_ts ≤ 10 min else dropped (§7.2) |
| payload | jsonb | e.g. presence interval, offer kind, listen-in bounds |
| processed_tick_id | bigint null | tick marks consumed |

Retention: at 90 days, raw events are folded into **presence_daily** / **interaction_daily** per-bird rollups, then dropped. Drift never needs raw history because vectors are persisted (I3/§6.3). Notebook entries persist for account life.

**ticks** — `id bigserial, aviary_id, from_ts, to_ts, events_consumed_lo/hi (seq range), drift_applied jsonb (per-trait deltas, for the calibration harness only — never exposed via API), duration_ms, created_at`.

**notebook_entries** — `id bigserial, aviary_id, aviary_date text, body text, salience_kind text (internal), created_at`. Read-only forever; no update/delete paths exist except account purge.

**narration_state** — `aviary_id PK, line text, rev int, generated_at, priority_pending jsonb` (queue of user-event lines awaiting insertion).

**visit_invitations** — `id, host_account_id, visitor_email_enc, visitor_email_idx, token_hash, created_at, expires_at (30 d), redeemed_at, revoked_at, status (outstanding|active|revoked|expired)`.

**visit_sessions** — `id, invitation_id, token_hash, started_at, last_seen_at, ended_at` (feeds the visit log's approximate duration).

**account_exports** — `id, account_id, requested_at, generated_at, download_token_hash, expires_at (24 h), downloaded_at`.

**config** — `key PK, value jsonb, updated_at, updated_by, reason text` (every tunable in Appendix E; all changes audited; drift-gain changes additionally require the calibration harness to pass in staging before prod apply).

Static config (versioned in repo, not DB): **species** (6 entries: silhouette rig id, palette, motif-grammar id, diurnal/nocturnal flag, baseline traits), **song_fragments** (5 procedural motifs), **arrival_schedule** (§6.11), **copy catalogs** (naturalist + matter-of-fact, linted).

### 4.2 Identity/continuity invariants

- `birds.id`, `birds.signature`, and `aviaries.created_at` are immutable; migrations that would rewrite them are rejected by invariant tests in CI.
- Personality vectors are stored, never derived: no code path recomputes them from event logs (the event log feeds *deltas* only).
- Backups: continuous PITR + daily snapshots; retention window ≤ 35 days so hard deletion is complete within 30 d soft + 5 d backup aging.

---

## 5. API surface

REST/JSON under `/v1`, HTTPS-only. Auth: httpOnly Secure SameSite=Lax session cookie (JWT, edge-verifiable signature; API additionally checks the revocation list). Visitors use a scoped read-only token (§12). Snapshot schema carries a `v` field; evolution is additive; clients tolerate unknown fields.

### 5.1 Endpoints

| Method & path | Auth | Purpose |
|---|---|---|
| `POST /v1/auth/magic-link` `{email}` | none | Rate-limited (5/h/email_idx); always returns the same matter-of-fact success copy (no account enumeration); queues email. |
| `GET /v1/auth/verify?token=` | none | Atomic single-use consumption; 15-min expiry; sets session cookie; 302 → `/aviary`. Expired/used → matter-of-fact error surface (Appendix C). |
| `POST /v1/auth/signout` | session | Revokes current session. |
| `GET /v1/sessions` / `DELETE /v1/sessions/{id}` | session | Device list + revocation. |
| `GET /v1/aviary/snapshot?ctx=open|keepalive|resume` | session | Canonical snapshot (§5.2). `ctx=open` additionally runs the **greeting resolver** (§6.7) and returns a `greeting` block; `ctx` also updates account timezone from the `X-Aviary-TZ` header (§6.9). ETag = `state_rev`; 304 on unchanged. |
| `POST /v1/aviary/events` | session | Batch append (≤ 32 events). Response: `{accepted:[seq…], rejected:[{idem_key, code}], reactions:[…], state_rev}` — the **reaction fast-path** (§6.8) runs synchronously for `offer` events. |
| `GET /v1/notebook?limit=50&before={id}` | session | Paginated entries, newest first, indefinite scrollback. |
| `PATCH /v1/birds/{id}` `{name}` | session | Rename (validated: ≤ 24 chars, no control chars). |
| `POST /v1/birds/adopt` `{arrival_id, name?}` | session | Adopt an offered arrival (§6.11); server verifies age-gate; 409 if slot unavailable. |
| `GET /v1/account` / `PATCH /v1/account` | session | Settings (a11y prefs, visit-notification toggle, timezone). |
| `POST /v1/account/export` | session | Generates JSON (§11.4), emails 24-h download link. |
| `GET /v1/account/export/{token}` | link token | Download. |
| `POST /v1/account/delete` / `POST /v1/account/restore` | session | Soft delete / "I changed my mind". |
| `POST /v1/visits/invitations` `{email}` | session | Host invites; emails one-time link. |
| `GET /v1/visits/invitations` / `DELETE /v1/visits/invitations/{id}` | session | Outstanding/active list; revoke (immediate effect, §12.3). |
| `GET /v1/visits/log` | session | Who/when/approx-duration + outstanding invites. |
| `GET /v1/visits/redeem?token=` | invite token | One-time redemption → visitor session token (7-day read-only pass, §17 D10); expired/revoked → matter-of-fact surface. |
| `GET /v1/visit/snapshot` | visitor token | Read-only snapshot variant: **no `greeting` block, no narration priority events**; visitor presence is never recorded. |
| `GET /v1/unsupported` | none | Static matter-of-fact browser-support surface. |

There is deliberately **no** endpoint that accepts personality values, moods, positions, or notebook writes (I1/I2). There is no "unread" or badge endpoint anywhere (I5).

### 5.2 Snapshot payload (abridged; full schema in Appendix A)

```jsonc
{
  "v": 3, "state_rev": 918273, "server_ts": "2026-09-06T07:42:03Z",
  "aviary": {
    "local_time": "07:42", "day_phase": "morning",
    "light": { "palette_key": "dawn_warm", "blend": 0.35 },
    "weather": { "type": "rain", "intensity": 0.4,
                 "started_at": "…", "ends_at": "…" },   // or null
    "settled": false, "pool_until": null,
    "arrival": null            // or {arrival_id, species_silhouette, perches_back: true}
  },
  "birds": [{
    "id": "…", "name": "pip", "species": "warbler",
    "mood": "content", "mood_since": "…",
    "perch": { "zone": 2, "x": 0.32 },
    "plan": [ { "activity": "preen", "starts_at": "…", "ends_at": "…",
                "params": { "intensity": 0.6 } }, … ],   // 2–5 min horizon
    "render": { "plumage": { "base": "#8a6f4d", "sat_level": 2, "detail": 3 },
                "idle_energy": 0.55, "size": 0.92 }       // derived; no raw traits (I2)
  }],
  "calls": [ { "bird_id": "…", "at": "…",
               "spec": { "motif_seed": 8841, "contour": "rise-3",
                         "energy": 0.6, "pitch_mul": 1.0,
                         "repeat": null } } ],             // ~next 90 s of schedule
  "greeting": null,   // or §6.7 block when ctx=open
  "narration": { "rev": 42, "line": "a warbler perches on the high branch, calling softly." }
}
```

Payload target ≤ 8 KB gzipped p95 (7 birds, 90 s call horizon, 5-min plan horizon).

### 5.3 Event submission semantics

- Every event carries a client-generated `idem_key` (UUID); retries dedupe on `(aviary_id, idem_key)`.
- Batched POST every ~10 s or on flush triggers (visibility change, settle, close via `navigator.sendBeacon`).
- Rejections are silent in-product (codes exist for logs/tests only): `cooldown`, `late` (> 10 min skew), `duplicate` (accepted idempotently, not an error), `settled` (presence while settled, §7.3), `rate_limited`. No rejection ever renders a toast (I5).

---

## 6. Simulation engine design

### 6.1 Tick loop

- **Scheduler:** every aviary has a due time in a Redis ZSET (score = next_due_ts). Sim workers `ZPOPMIN` batches of 100 due aviaries (sharded by `aviary_id % worker_count` to avoid contention), tick sequentially with pipelined DB writes, re-insert with `next_due = now + cadence`.
- **Cadence tiers:** 60 s for aviaries with presence in the last 72 h; 300 s for dormant ones. Ticks are catch-up-safe (I10): each tick computes `elapsed = to_ts − from_ts` and applies time-based transitions in bulk, so a dormant aviary's 5-minute tick yields the same state as five 1-minute ticks within ε. Redis loss merely delays ticks; the next scheduler sweep re-derives due times from `last_tick_at`.
- **Tick pipeline** (one pure-ish function in `sim-core`, `runTick(state, events, interval, rng) → {state', driftDeltas, newEntries, narration, snapshotPatch}`):
  1. Load canonical state + unprocessed events (`seq` range, in order).
  2. Aggregate signals: presence-seconds (union-deduped, §7.3), listen-in minutes per bird, offers (accepted/near) per bird, settles.
  3. Ambient: derive weather for the interval from the seeded schedule (§6.6); compute aviary-local time and day phase from `accounts.timezone`.
  4. Mood transitions per bird (§6.4).
  5. Drift deltas (§6.3); apply via `applyDriftDelta` (I3).
  6. Behavior planning: extend each bird's plan horizon to ≥ now + 2 min (§6.5); schedule calls (§6.6); update `render` params from traits (plumage color/detail from `plumage_saturation`, `idle_energy` from boldness+curiosity — derived values only, I2).
  7. Chronicler pass (§6.10): evaluate salience, maybe write 0–1 notebook entries.
  8. Narrator pass (§6.10): maybe regenerate the narration line.
  9. Persist state + `ticks` row (with `drift_applied` for harness eyes only), mark events processed, bump `state_rev`, push snapshot JSON to edge KV.
- **Latency budget:** ≤ 100 ms p50, ≤ 500 ms p99 per tick; worker emits duration histogram; **alarm at p99 > 5 s** (PRD error budget).

### 6.2 Throughput design target

100 k accounts: ~30 k active-tier (500 ticks/s) + ~70 k dormant (233 ticks/s) ⇒ ≈ 75 worker-cores at 100 ms/tick; deploy 2× headroom. Load-test staging at 50 k ticking aviaries before GA (§14.5).

### 6.3 Drift function

**Signals → daily accumulators (per bird, server-side):**

| signal | source | daily cap |
|---|---|---|
| `S_pres` | union-deduped presence hours | 8 h |
| `S_listen(b)` | listen-in minutes focused on b | 60 min |
| `S_accept(b)` | offers b accepted | 5 |
| `S_near(b)` | offers made while b was nearest/eligible receiver | 5 |

**Per-day raw targets** (gains from `config`, initial values below; the harness — not these numbers — is the source of truth):

```
raw_B = 0.10·(S_pres/8)          + 0.05·(S_near/5)
raw_W = 0.10·(S_pres/8)          + 0.15·(S_listen/60)
raw_V = 0.10·(S_pres/8)          + 0.15·(S_listen/60)
raw_P = 0.12·(S_pres/8)
raw_C =                          + 0.08·(S_accept/5)
```

**Low-pass:** `EMA_trait(day) = α·raw + (1−α)·EMA_trait(prev)`, α = 0.15; committed daily delta = `EMA_trait(day)`, applied in twelve hourly micro-steps during ticks so no daily boundary discontinuity is ever observable. All deltas clamped to **[0, remaining headroom]** — monotonic by construction (I3). Traits cap at 1.00; the EMA keeps running (no visible saturation cliff).

**Settle** contributes nothing to drift beyond ending the presence window cleanly (per PRD).

**Calibration targets (owned by the harness, §14.2):**
- "Regular visits" persona (≈ 20 min presence/day, occasional listen-in, ~1 offer/day): cumulative |Δ| ≥ **0.01 on ≥ 2 traits by day 7** (instrument-measurable), **0.04–0.06 by day 21** with perceptible behavior change (greeting latency ↓, call rate ↑, front-perch bias ↑, plumage visibly richer), ≈ 0.12 by day 90.
- Single-session delta < **0.002** for any persona (a session must never move a trait visibly — anti-Tamagotchi gate).
- Absent persona (2 weeks dark): exactly 0.00 negative movement; expressiveness quieting comes from the fast-timescale recency factor below, not traits.

**Expressiveness recency (fast timescale, not drift):** greeting probability, spontaneous-call rate, and front-perch bias are multiplied by `recency = 0.55 + 0.45·exp(−days_since_presence/4)`. This is how "a bird that gets ignored greets less often" is implemented without ever touching a trait — the birds get quieter, not warier.

### 6.4 Mood system

- **Finalized state set:** `wary, alert, content, curious, drowsy, roosted` (roosted = night sleep posture: eyes closed, low on perch; the nightjar species is exempt and stays `alert/content` at night).
- **Transitions** evaluated each tick as a weighted categorical draw:
  - time-of-day priors (drowsy weight ↑ near dusk, alert ↑ early morning, roosted dominant at full night for diurnal species),
  - weather (rain: vocal-activity dampener + slight drowsy/content bias; wind: alert ↑ for some birds, wary ↑ for low-boldness birds),
  - recent interactions (accepted offer → content nudge for 20–40 min; listen-in → content/curious nudge for the focused bird),
  - bird-to-bird: wary spreads to same-zone neighbors with probability ∝ their social_warmth; a content bird adjacent to a wary bird damps the spread (the "small social system"),
  - personality gating: P(wary) scaled by `(1 − boldness)`; P(curious) scaled by curiosity; P(drowsy at dusk) scaled by `(1 − alertness-from-V)`.
- **Minimum dwell:** 10 min per mood (reaction-driven exceptions: offer acceptance may shorten). Prevents flicker.
- **Persistence:** mood and `mood_since` are canonical; snapshots always carry them, so a returning user sees the mood the tick produced overnight — never a reset-to-neutral (no client-side default mood exists in the codebase).

### 6.5 Behavior planner

Each tick extends every bird's plan to a 2–5 min horizon: an ordered list of `{activity, starts_at, ends_at, params, perch}`. Activity vocabulary: `preen, scan, doze, weight_shift, head_tilt, call, fly_to, hop_to, drink, bathe, watch_pool, investigate, greet, roost`. Rules:

- Perch choice on each `fly_to/hop_to`: zone weights from mood (wary → back ×3, content → middle, bold/curious → front ×(1+boldness)) × personality; birds already adjacent with high social_warmth bias toward co-perching.
- `fly_to` is always an explicit plan entry with duration 1.5–4 s by distance — the client never has to infer a transition (no teleporting, ever).
- Idle micro-motion (weight shifts, head tilts, scanning) is interleaved at 8–25 s intervals so a bird is never plan-idle; amplitude/timing keyed by `render.idle_energy` (personality-derived).
- Mood reads without labels: wary ⇒ more scanning, back perches; content ⇒ preening; curious ⇒ head tilts toward sound events (calls, weather, offers); drowsy ⇒ low posture, fluffed, slow blink intervals. No mood is ever rendered as text in the scene (I5).

### 6.6 Calls: grammar, scheduling, chorus, weather

- **Grammar:** each species has a motif grammar — a generative description (note-count distributions, contour set {rise, fall, flat, trill, churr}, interval ranges, rhythm patterns, repetition forms like "trill, pause, trill again"). A **call spec** = `{motif_seed, contour, energy, pitch_mul, repeat}`; the client expands spec → notes using the species grammar + the bird's immutable signature (§4.1). The same spec deterministically renders the same call; different seeds never render identical calls (variation entropy test, §14.4).
- **Scheduling:** per-bird inhomogeneous Poisson process, rate
  `λ = λ_species · (0.4 + 0.9·vocal_frequency) · mood_factor · tod_factor · weather_factor · recency`,
  with `tod_factor` ≈ 1.2 at dawn, 1.0 day, 0.5 dusk, 0.05 night (nightjar: inverted — active late); `weather_factor` 0.5 during rain. Scheduled 90 s ahead into snapshots.
- **Chorus coupling (bird-to-bird):** a scheduled call raises λ ×3 for 3–8 s in birds with social_warmth > 0.5 (call-and-response); simultaneous overlapping calls emerge naturally — the client mixes them as a real chorus (§9.3). Alarm-ish sharp calls (wary mood) push neighbors toward wary (§6.4).
- **Song fragments** (offer): one of 5 library motifs, played softly center-panned; response scheduled by the reaction resolver (join-in for high-V content birds, quiet-listen otherwise, call-against for alert/wary).

### 6.7 Greeting resolver (runs in API at `ctx=open`)

- `absence = now − last_presence_end` (from presence rollups).
- **Greeter selection:** weight per awake bird `∝ boldness · (mood factor: alert 1.2, content 1.0, curious 0.9, drowsy 0.4, wary 0.3, roosted 0)` × social_warmth tiebreak; a bird that greeted first last session gets ×0.6 (rotation so the notebook's "pip greeted before wren today, first time this week" is a real observable).
- **Form by absence bucket × boldness:**

| absence | low boldness | mid | high boldness |
|---|---|---|---|
| < 10 min | glance up from current activity | head-tilt | head-tilt + short two-note call |
| 10 min – 6 h | head-tilt | quiet two-note call | call + step toward front perch |
| 6 h – 2 d | glance + slow hop forward | call, longer contour | re-orientation: fly to front perch + call |
| > 2 d | hesitant approach, soft call | call + second bird response | long call; second bird answers, staggered |

- **Procedural variation:** form params (call spec seed, step distance, glance duration, timing offsets) jittered from `rng(aviary_id, session_open_ts)` — never a canned cue, never identical twice (harness asserts pairwise-distinct over 1 000 simulated opens).
- **Stagger:** when a second bird responds, offset = uniform(1.5 s, 4 s) — never unison (unison would "announce" arrival).
- Response block: `{bird_id, form, at_offset (0.8–2.0 s from first paint), call_spec?, second?: {bird_id, form, at_offset}}`. Client renders it as an ordinary plan overlay. **No text accompanies it, ever** (I5).

### 6.8 Offer-reaction fast-path (ingest-time resolver)

A ~1-minute tick cannot drive in-session reactions, so `POST /events` runs the **same `sim-core` reaction function** synchronously against canonical state and returns directives in the response. The resolver writes no personality and no mood (I1): it emits ephemeral plan overlays and appends a `server_reaction` event that the next tick consumes to fold mood effects (accepted offer → content nudge) and drift signals.

Reaction matrix (seed offer; song/pool analogous):

| mood \ curiosity | low | mid | high |
|---|---|---|---|
| content | glance, stay | hop closer (5–10 s) | approach + peck (3–6 s) |
| curious | watch | approach after pause | approach immediately, head-tilts |
| alert | freeze-look | watch, slow approach | approach, cautious |
| wary | ignore | wait 15–30 s, maybe approach | long wait, then approach |
| drowsy | no reaction | head lift only | head lift + slow hop |
| roosted | none | none | none |

- **Cooldown:** 180 s per bird, all kinds (config). Enforced at ingest; the top-bar offer affordance reflects cooldown state from the snapshot (quiet dimming of the target bird's entry in the offer sheet — no timer text, no punitive copy).
- **Still pool:** sets `aviary.pool_until = now + 3 min`; drink/bathe/watch activities are scheduled into plans over that window by the reaction resolver and ratified by ticks.
- **Settle:** event ends the presence window immediately; client renders the 5-s lighting shift; tick applies the mood-quieting nudge. Undo within 5 s sends `settle_undo` (reverses client state; server marks the settle event superseded — presence resumes). After 5 s, any click/keypress in the scene re-engages (slow 3-s lighting return); mere mouse movement does not (§17 D22).

### 6.9 Day/night and timezone

- Aviary-local time = `accounts.timezone` (IANA). The server computes `day_phase` and light-palette keys; the client interpolates palette blends between snapshots. Devices anywhere see the same aviary in the same phase (canonical single state).
- TZ update policy: at each `ctx=open`, the client sends its IANA TZ; the server updates the account TZ if different. The user's local time is where the user currently is; one canonical aviary time still holds at every instant (§17 D8).
- Phase curve: dawn 05–08 (warming), midday 08–16 (brightest), evening 16–19 (warm, calls quieting), night 19–05 (dim, roosted; nightjar active). Settle overlays the evening palette regardless of local time until re-engagement.

### 6.10 Chronicler and narrator (the `voice` package)

One prose module generates notebook entries, narration lines, and caption text — same grammar tables, same voice rules (lowercase, present tense, bird-named, specific, no "you", no exclamations, no numbers, no achievement phrasing). Copy lint (§14.6) runs over every string the module can emit.

- **Chronicler (notebook):** each tick evaluates salience detectors — greeting-order firsts (needs a rolling 7-day per-aviary greeting-order memory), first rain after dry days, unusual mood juxtapositions, long quiet stretches, notable offer reactions, co-perching events, arrival of a new bird. Each candidate gets a salience score; a **rarity governor** enforces: ≥ 36 h since last entry, ≤ 3 entries/week, stochastic acceptance ∝ score (so very active aviaries stay sparse — the notebook is not a feed). Entries are template-family + slot-fill + word-order variation from the seeded RNG (no identical repeats within 30 days). **Banned content (test-enforced):** any observation of the *user's* behavior ("you visited…", week-streaks), any numeric trait reference, any achievement framing. The line is aviary-observations only.
- **Narrator:** maintains one current scene line, regenerated on salient state change (perch moves, mood shifts, weather start/end) with min interval 30 s and max staleness 60 s; user-event lines (greeting, offer reaction, settle) enter a small priority queue consumed by the client (§10.1). Generation is server-side for voice consistency (§17 D13).

### 6.11 New-bird arrivals (age-gated ramp)

- `arrival_schedule` config: aviary age → slots: `[(0 d, 2), (60 d, 3), (120 d, 4), (210 d, 5), (300 d, 6), (365 d, 7)]`. Never visit-count, never interaction-score, never paid.
- When a slot opens, the tick sets `aviary.arrival`: an unfamiliar silhouette appears on the back perch in subsequent sessions (rendered ambiently — it does not greet, does not call loudly; it is *noticed*, not announced). A quiet top-bar affordance (a small perch-icon, no badge, no text popup) opens the naming/adoption sheet in naturalist voice ("a young finch has been sitting at the back of the aviary since tuesday. name them, or leave them be."). Adopt → `POST /birds/adopt` → bird enters with a soft fly-in to a middle perch. Ignore → the arrival keeps appearing intermittently; the offer never expires and never nags.
- Species selection draws from the pool weighted to complement current signatures' pitch/timbre spread (protects recognizability at 7).

### 6.12 Deterministic RNG (I10)

All ambient randomness (weather, call jitter, chronicler variation, greeting jitter) derives from `seed = hash(aviary_id, rng_epoch, floor(unix_ts / bucket), purpose_tag)`. Consequence: a tick run late over a large interval recomputes the same ambient history it would have produced in real time — the catch-up-equivalence property test (§14.1) depends on this.

---

## 7. Sync model

### 7.1 Canonical propagation

One record, many readers. Laptops and phones pull the same snapshot (edge KV inline on load; origin API thereafter); both render the same moods, plans, calls, drift. There is no client-side canonical state, hence no merge, no conflict resolution, no eventual consistency — multi-device sync is a property of the architecture, not a feature.

**Pull triggers:** (a) initial load (edge-inlined), (b) `visibilitychange` → visible, (c) render-frame gap > 10 s (laptop wake detection), (d) keepalive every 20 s while visible, (e) after any `POST /events` response whose `state_rev` exceeds local. Pulls are conditional (`If-None-Match: state_rev`) — 304s cost ~nothing.

### 7.2 Event log semantics (no last-write-wins, structurally)

- Clients send **observations** ("listened in on pip for 3 min"), never values ("boldness = 0.62"). The API has no vocabulary for absolute state (I1).
- Server assigns per-aviary `seq` at insert; ticks consume strictly in `seq` order; drift is additive deltas computed by the server. The PRD's laptop-morning/phone-lunch overwrite scenario is unreachable because no client holds or writes vector state.
- **Idempotency:** `idem_key` dedupe makes retries and double-sends no-ops.
- **Late events:** accepted up to 10 min skew (clock drift, brief offline); older events are dropped — presence honesty beats completeness (§7.3).
- **Offline queue:** localStorage, capped at 200 events / 10 minutes; flushed on reconnect; anything beyond the cap or window is discarded silently.
- **Clock offset:** each response gives `server_ts`; client estimates `offset = server_ts − client_ts` (EWMA, clamped ±2 s) and uses it for all plan/call phase interpolation.

### 7.3 Presence accounting (the honesty-critical path)

Client monitor (`packages/presence`):

- Tracks the triple condition continuously: `document.visibilityState === 'visible'` ∧ `document.hasFocus()` (plus focus/blur events) ∧ `now − lastActivity < ACTIVITY_WINDOW` where `lastActivity` updates on throttled passive `pointermove`/`keydown` listeners. `ACTIVITY_WINDOW` initial **240 s** (config; PRD says "a few minutes, lean longer" — watching without moving is the product).
- While all three hold, accrue presence; **ping every 30 s** with the covered interval `[since, until]`; flush partial interval immediately on any condition break; flush + beacon on `pagehide`.
- **Settle suppresses pinging** until click/keypress re-engagement (§6.8) — settle ends the presence window cleanly, tab-close does the same, neither is penalized or distinguished by the engine.

Server side:

- Intervals from all devices/tabs of one account are **union-deduped by wall-clock overlap** — attention is measured once no matter how many screens show the aviary (§17 D9).
- Daily sanity clamp: presence-hours per account-day capped at 16 h; exceeding it alarms (would indicate a client bug inflating drift — the silent-corruption class the PRD warns about).
- Presence feeds drift only through the daily accumulators (§6.3); raw pings fold into rollups and age out at 90 days.

### 7.4 Conflict/error surfaces

The only "conflicts" possible are auth/session-level (magic-link replay, session expiry mid-write, outage). All render in matter-of-fact voice from the Appendix C catalog; none mention birds. A failed event POST never blocks rendering — the aviary keeps living; events retry from the queue.

---

## 8. Frontend rendering pipeline

### 8.1 Boot sequence (first bird < 500 ms)

1. Edge serves the static shell with critical CSS: the **quiet field** (soft sky gradient, faint horizon) paints immediately — this is the only load state that exists; no spinner component is in the codebase (I7).
2. The edge Worker inlines the latest snapshot JSON (from KV) into the HTML for authenticated requests; unauthenticated/cold-edge falls back to an origin snapshot fetch in parallel with JS load.
3. Critical JS chunk (≤ 300 KB gz): renderer core + one species rig + audio scheduler stub. Birds paint **mid-activity**: initial pose = phase computed from `plan` timestamps + clock offset. The aviary appears as if it has been rendering all along.
4. Non-critical chunks stream in: remaining species rigs, audio worklet, React chrome, notebook/settings/visits routes.
5. First-ever session (post-signup): naming interstitial (naturalist voice, defaults pre-filled, skippable) → empty quiet field → bird 1 soft fly-in → bird 2 fly-in staggered 2–5 s. This is the only entry animation in the product, and it happens once per account.

### 8.2 Layer stack

```
z0 sky gradient (CSS, palette-blended by day_phase + settle state)
z1 far foliage (parallax 0.2×, static art + slow sway)
z2 weather backdrop (rain haze)         ── canvas A (scene)
z3 perches + birds + pool
z4 weather foreground (rain streaks, wind-biased leaves)
z5 ambient ornaments (leaves/feathers — client-generated, tick-free)
z6 caption layer (DOM, positioned near calling bird)
z7 focus proxies (DOM buttons, invisible, tracked to bird positions @10 Hz)
z8 top bar (DOM/React; fades to ~8% opacity after 3 s cursor stillness,
   returns on pointer/keyboard activity; keyboard-focusable at all times)
```

Single `<canvas>` for z2–z5 (one compositor, cheap 60 fps); DOM for z6–z8 (text crispness, native focus/ARIA).

### 8.3 Bird rendering

- Per-species **rig**: a small skeleton (body, head, beak, tail, 2 wings, 2 legs) with procedural feather strokes; silhouette params from species config; plumage color/detail level from `render.plumage` (server-derived from the saturation trait — the trait itself never ships, I2).
- Activities are pose state-machines over the rig; phase from plan timestamps. Micro-motion: Perlin-noise jitter on head/tail/weight at amplitude ∝ `render.idle_energy`, mood-shaped (wary → faster scan saccades; drowsy → slow blink, fluffed contour; content → preen strokes).
- Flight: bezier path between perch anchors, ease-in-out, 1.5–4 s per the plan's explicit `fly_to` entry; wing-beat rate by species.

### 8.4 Responsive layout

Scene metrics are a pure function of viewport: perch-zone y-bands and x-extents scale with width; bird scale clamps to [0.7, 1.1]; positions clamp so **no bird is ever cropped or offscreen** at any viewport (property asserted in e2e across a viewport matrix 320×568 → 2560×1440, portrait/landscape). Narrow phones compress inter-perch spacing; wide desktops widen it. No panning, no zoom, no scroll — the scene is exactly one screen.

### 8.5 Interpolation, extrapolation, lifecycle

- All motion interpolates against `offset`-corrected server timestamps; consecutive snapshots cross-fade plans (new plan entries blend from the bird's current rendered pose over ≤ 400 ms — never a snap).
- If the plan horizon exhausts before the next snapshot (network stall), the client runs **idle filler**: only `scan/weight_shift/doze-hold` chosen locally, seeded by `bird_id + minute` (deterministic, calm, non-committal). The next real plan overrides with a blend. Filler never includes calls, flights, or greetings.
- Hidden tab: rAF halts (browser-throttled anyway); rendering stops, audio context suspends; on visible → immediate `ctx=resume` snapshot pull, clock-offset re-estimate, plan re-phase. Frame-gap > 10 s (wake) triggers the same path. The aviary returned to is the aviary that has been running.

### 8.6 Reduced-motion renderer (a designed surface)

Triggered by `prefers-reduced-motion` or the settings override (settings wins when set). Same snapshot data, different register:

- Micro-motion → **slow cross-fades (≈ 2 s) between still poses** from a per-activity pose library (preen = 4-pose cycle cross-faded, not animated).
- Flights → cross-fade between origin and destination perch poses (1.2 s), no animated path.
- Ambient leaf/feather ornaments removed; weather reduced to palette/haze shifts only.
- Day/evening color transitions retained but slowed ×3.
- Calls still play at full quality; captions unaffected; drift/mood/notebook untouched — the aviary is the aviary, calmer and slower, not broken.

### 8.7 Settle rendering

Top-bar settle → lighting blends to evening palette over 5 s, call gains ramp down (audio §9.4), birds' next plan entries bias to `roost/doze` (server directive in the event response). Any click in the scene ≤ 5 s → `settle_undo`, palette returns over 2 s. Settled state persists across keepalive pulls (`aviary.settled`) until re-engagement or tab close.

---

## 9. Audio pipeline

### 9.1 Graph

```
AudioWorklet synth (single module, N voices ≤ 10: 7 birds + fragment + weather)
  └─ per-voice output → per-bird GainNode (listen-in mix)
       → StereoPannerNode (pan = perch_x mapped −0.6..0.6)
       → master GainNode (settle/mute) → DynamicsCompressor → destination
```

The worklet synthesizes sample-accurately off the main thread; the main thread only schedules (250 ms look-ahead queue fed by snapshot `calls` + reaction directives, timed against `AudioContext.currentTime` and the server clock offset). No per-call AudioNode churn on the main thread; voices are worklet-internal.

### 9.2 Procedural synthesis (no samples, ever — I6)

Note = 2–3 detuned partial oscillators (species-timbre ratios) + ADSR envelope + species formant filter + optional vibrato/noise component (trills, churrs, breath). The call spec (`motif_seed, contour, energy, pitch_mul, repeat`) expands through the species grammar into note events; the bird's immutable `signature` (pitch center, brightness, vibrato, attack, tempo bias) shapes every note — **signature params are for life**, so Pip is recognizable by ear across moods and drift; mood only modulates energy/contour choice. Two birds calling overlap as true runtime-mixed choruses — no stacked loops, no phase-cancel artifacts (the failure mode the PRD names).

### 9.3 Chorus mixing

Server scheduling (§6.6) produces natural overlaps; the client simply sums voices. Per-bird panners give spatial spread by perch position. Voice cap 10 with steal-oldest on overflow (should never trigger at 7 birds).

### 9.4 Listen-in mix

- Engage (click/tap/Enter on a bird): focused bird gain → 1.0, others → **floor 0.30 (never 0 — re-balance, not mute)**, both via `setTargetAtTime` with τ ≈ 0.5 s (≈ 1.2 s perceptual rise). Gradual in *and* out — listening, not channel-switching.
- Disengage triggers: second click on the focused bird, focusing a different bird (cross-ramp), click on empty scene, keyboard focus leaving the scene (Escape or Tab-out). Return ramp τ ≈ 0.6 s.
- Mix state is purely client-side; the server receives `listen_in_start/end` events (drift signal), not gains.

### 9.5 Autoplay policy vs. "calls already audible"

Browsers require a gesture before audio. Resolution (documented, not a design failure): the AudioContext is created suspended; **any** first pointerdown/keydown resumes it with a 1.0 s fade-in of the ambient mix; returning users with browser autoplay permission (MEI) get sound immediately on load. The visual aviary is fully alive regardless; audio joins at the earliest moment the platform allows (§17 D16).

### 9.6 Fallback and settings

- No WebAudio / worklet construction fails / context permanently blocked → **graceful silence with captions auto-enabled**. No recorded-audio path exists to fall back to (I6).
- Mute setting (accessibility panel) suspends the context; captions remain independent.

### 9.7 Memory discipline

Worklet voice pool preallocated; envelope state in ring buffers; spec→note expansion is allocation-bounded; zero steady-state heap growth per call (asserted by the 30-min soak test, §13).

---

## 10. Accessibility surfaces

### 10.1 Screen-reader narration

- A single `aria-live="polite"` region (the **only** live region besides captions) receives the server-generated narration line (§6.10). Cadence: one prose update per 30–60 s at idle; priority lines (return-greeting, offer reaction, settle acknowledgment) inserted promptly as observations — "a soft call from the front perch, then a small step closer", never "greeting detected".
- Rate limiter client-side: ≤ 2 lines/min at idle regardless of server churn (protects the SR queue; the system never pushes the user into silencing it).
- Prose is the same `voice` module as the notebook — a screen-reader user moving between surfaces hears one product. State-list narration ("Pip is at perch 2") is a banned output class (copy lint + voice-module types make it unrepresentable).
- Optional **visible narration** (accessibility setting): renders the same lines as a quiet transcript strip (AA contrast, naturalist voice) for users who want text alongside/instead of SR output.

### 10.2 Call captions

- Toggle in accessibility settings; auto-on in silent fallback (§9.6).
- Caption text is generated **client-side from the exact call spec just synthesized** (the `voice` caption grammar maps contour/note-count/energy/repeat → "a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch"). Captions always match what actually played; no stored strings.
- Rendered as small text near the calling bird (z6 DOM layer), fading in/out with the call envelope, on a soft scrim guaranteeing AA contrast against both bright and dim scene states.

### 10.3 Keyboard and focus

| key | action |
|---|---|
| Tab | top-bar items in order (account, accessibility, notebook, offer, settle) → then into scene: focuses first bird proxy |
| ← / → (in scene) | move focus between birds (perch order) |
| Enter (on bird) | toggle listen-in on that bird |
| Escape | exit listen-in / close dialog / return focus to top bar |
| Offer sheet | fully navigable (arrow keys across kinds, Enter to offer, Escape closes); opens from top-bar button or its shortcut |

- Birds are exposed as focus proxies: invisible DOM buttons at bird screen positions (updated 10 Hz), accessible name `"{name}, {species}"` (e.g., "pip, warbler") — naming a focused control is standard practice; the *atmosphere* lives in the narration prose, not the proxy label (§17 D14).
- Focus indicator: dual-tone outline (light core + dark edge) specified by the visual designer to read against both bright midday and dim night palettes; visible on proxies and all chrome.
- Dialogs (notebook, settings, offer sheet): standard focus trapping, `aria-modal`, Escape to close, focus restored to trigger.

### 10.4 Contrast and motion safety

- All user copy (top-bar labels, settings, captions, visible narration, error surfaces) ≥ WCAG AA; exact ratios per the design system spec; automated contrast checks in CI on every palette/token change.
- The scene itself carries no text (captions are the exception, on scrim).
- Reduced motion is honored from OS preference with in-product override (§8.6); no strobing or fast-flashing effects exist in any animation curve (max luminance-change-rate lint on rig keyframes).

### 10.5 Ship-together rule

Accessibility ships **with** v1, not after: narration, captions, reduced motion, and keyboard parity are exit criteria for GA (§2.3), each owned by a named engineer from M1 (not retrofitted in "v1.1").

---

## 11. Accounts, auth, privacy

### 11.1 Synthetic-ID rule (the boring detail with teeth)

Account UUID is the only identifier in DB keys, event payloads, queue messages, cache keys, log lines, and metrics. Email lives encrypted (`email_enc`, AES-256-GCM, KMS-managed key) with an HMAC blind index (`email_idx`) solely for lookup. Enforced by: repository-layer typing (no raw email crosses service boundaries), a log-scrubber sampling audit in CI, and lint banning `email` in any schema outside the two tables in §4.1.

### 11.2 Magic-link auth

`POST /auth/magic-link` → 256-bit random token, stored hashed, 15-min TTL, single-use via atomic consume; rate-limited 5/h per `email_idx` (plus global IP limits); identical response copy regardless of account existence (no enumeration). Email is transactional-provider-sent (SPF/DKIM/DMARC), matter-of-fact copy: "Your Pocket Aviary sign-in link. It expires in 15 minutes. If you didn't request it, ignore this email." Verify → session cookie (JWT: `account_id, session_id, exp`, 30-day sliding) + redirect. Failure surfaces use the Appendix C catalog ("We couldn't sign you in. The link may have expired. Try requesting a new link.").

### 11.3 Sessions and devices

Sessions table with device labels ("Chrome on macOS", derived from UA at sign-in); list + revoke from account settings; revocation checked per API call; edge revocation cache TTL ≤ 60 s (read-only snapshots may survive ≤ 60 s post-revocation — accepted, documented). Email change: new address verified before commit; old email valid until then.

### 11.4 Export

On demand → JSON `{generated_at, account: {settings, timezone}, aviary: {created_at}, birds: [{id, name, species, adopted_at, personality: {boldness, …}, mood}], notebook: [entries]}` — **includes current personality vectors per the PRD**; this is the sole sanctioned numeric exposure and it is user-pulled, emailed as a 24-h link, never displayed in any UI (§17 D11).

### 11.5 Deletion

Soft: `deleted_at` set; signing in during the 30-day window shows a quiet system-voice bar ("Your account is scheduled for deletion on {date}. [Keep my account]") on any signed-in page — a system surface, so a bar is allowed here (I5 governs product surfaces). Hard: purge job at day 30 deletes account, aviary, birds, events, rollups, ticks, notebook, narration, visits, sessions, exports, edge-KV snapshots; PITR window ≤ 35 days guarantees backup aging completes the erasure. Mailer queue is drained of account references before purge.

### 11.6 Privacy boundary (architectural, not policy)

- Per-bird interaction data drives only that account's simulation. Telemetry (RUM + ClickHouse) has **no** `account_id`, `bird_id`, email, or any reconstructible dimension — enforced at the schema registry (CI gate) and by separate credentials with no network route to the simulation DB.
- Allowed aggregate telemetry: request counts, latencies (snapshot/events/tick), error rates, anonymized session-duration histograms, render-frame timing histograms, audio-pipeline error counts, first-bird timings, presence-volume histograms (bucketed, no account dimension), magic-link delivery rates.
- Forbidden forever: per-account dashboards, visit-frequency analytics, cross-account drift aggregates, "average bird" dashboards, engagement funnels, ML training on per-bird fields. The analytics warehouse never receives a grant on the simulation DB.
- Privacy policy: plain-text link in account settings naming the aggregate categories and explicitly excluding per-bird interaction state.

---

## 12. Visits (the entire social feature)

### 12.1 Invite flow

Host: account settings → "Invite a friend to visit" → enters email → `POST /visits/invitations`. System emails a one-time link (30-day expiry). No global discoverability flag, no re-invitation, no frequent-visitor status, no onboarding mention. Invites default OFF by existing only on demand.

### 12.2 Visitor session

Link → `GET /visits/redeem` → visitor token: **7-day read-only pass** (§17 D10), stored hashed in `visit_sessions`. Visitor client = the same renderer in visit mode:

- Pulls `GET /visit/snapshot` (no greeting block — visitors trigger nothing; no narration priority events; identical birds/moods/drift/lighting/weather — no show-off rendering).
- Event POST, presence recording, listen-in server events, settle, notebook, offers: **all disabled** (listen-in remains as a local audio-mix affordance? No — "cannot trigger anything": visit mode disables listen-in engagement too; ambient mix only. Local a11y settings — captions, mute, reduced motion — remain, stored device-locally).
- Visitor attention never touches host drift: no presence intervals are recorded for visitor tokens (API rejects at the route level).

### 12.3 Revocation and expiry

Revoke from settings → `status=revoked` immediately; visitor's next snapshot pull (≤ 20 s keepalive) returns 410 → matter-of-fact surface: "This visit is no longer available." Unused revoked/expired links render the same surface at redeem. No host notification of revocation success — the visit log's absence of the visitor is the confirmation.

### 12.4 Logging and notifications

Visit log (settings, on-demand, no badge): visitor email, date, approximate duration per session, outstanding invitations. Host visit-notification toggle: off by default, never surfaced in onboarding; when on, the *only* aviary-activity email the product ever sends ("A friend visited your aviary today.") — deliberately dull, no visit count, no engagement framing.

---

## 13. Performance budgets and observability

### 13.1 Budgets (all are CI gates, not aspirations)

| Budget | Value | Gate |
|---|---|---|
| Initial JS (first paint, gz) | < 2 MB (warn 1.6 MB) | bundle-size check on every PR; critical chunk ≤ 300 KB tracked separately |
| Time to first bird visible | < 500 ms p75, mid-tier mobile / 4G profile | Playwright + throttled synthetic run per commit to `main`; edge-inlined snapshot asserted |
| Idle frame rate | 60 fps (frame p95 ≤ 20 ms) sustained 30 min on reference 5-year-old laptop class | frame-timing harness in CI (headless with CPU throttle ×4) |
| Client memory | no growth over 30-min session (heap slope < 5 MB) | Puppeteer soak test in nightly CI — a real test, per PRD |
| Snapshot payload | ≤ 8 KB gz p95 | API contract test |
| Snapshot API latency | p95 ≤ 150 ms origin; ≤ 400 ms alarm | RUM + synthetic |
| Tick latency | p99 ≤ 500 ms target; **alarm at p99 > 5 s** | worker histogram + alert |
| No recorded audio assets | 0 bytes | asset lint (I6) |

### 13.2 Synthetic fleet

Automated browsers on a schedule from 5 common geographies × {desktop, mid-tier mobile} profiles: load the aviary hourly, measure first-bird paint, frame timings, audio-context errors, snapshot latency; results to ClickHouse (no account dimension — fleet uses dedicated synthetic accounts flagged out of all aggregates except perf).

### 13.3 RUM (aggregate-only)

Navigation/load timings, first-bird render, frame-time histograms, audio-context error counts, event-POST latency, snapshot latency/304-rate, presence-ping volume histogram (bucketed), WebAudio-fallback rate. Emitted without identifiers; schema-gated (§11.6).

### 13.4 Alarms

tick p99 > 5 s; snapshot p95 > 400 ms; magic-link delivery failure > 2 %; audio-context error rate > 1 % of sessions; frame-drop rate regression > 20 % week-over-week; **personality-trait-decrease canary** (any negative delta observed in `ticks.drift_applied` ⇒ page on-call — I3); presence > 16 h/account-day rate spike (I4 bug class); edge-KV push lag > 2 × tick cadence.

### 13.5 What we deliberately do not measure

Visit frequency per account, session counts per account, streaks of any kind, engagement funnels, retention cohorts keyed to behavior, per-bird anything, cross-account drift aggregates, notification-style re-engagement signals. There is no product-analytics dashboard; operational health + aggregate adoption counts (signups, sign-in success rate) only. This absence is architectural (I9) so it stays absent.

---

## 14. Testing and calibration strategy

### 14.1 `sim-core` unit/property tests

- **Drift monotonicity:** property test over arbitrary event sequences — no trait ever decreases (I3).
- **Catch-up equivalence:** for random event/weather histories, 1-minute ticking ≡ single catch-up tick over the same interval, within ε (I10) — the property that makes cadence tiering and outage recovery safe.
- **Greeting variation:** 1 000 simulated opens per absence bucket — pairwise-distinct form params; boldness/mood ordering honored; stagger never unison.
- **Mood:** minimum dwell respected; no reset-to-default path exists; roosted at night for diurnals; nightjar exempt.
- **Chronicler:** rarity governor bounds hold under hyper-active personas; banned-pattern lint (no user-behavior observations, no numerals, no achievement phrasing); no identical entries within 30 simulated days.
- **Weather determinism:** same seed ⇒ same schedule; rain frequency lands 2–3/week over simulated months.

### 14.2 Calibration harness (owns the drift targets)

Accelerated-clock staging (1 s = 1 sim-minute) running scripted personas against the real engine: **regular** (20 min/day), **devoted** (2 h/day), **sporadic** (3×/week), **absent** (2 weeks dark), **night-owl** (evening presence). Assertions: §6.3 targets at days 7/21/90; single-session Δ < 0.002; absent persona ⇒ zero negative drift + measurable expressiveness-recency quieting in behavior stats (greeting rate, call λ). Runs nightly and gates any `drift_gains` config change. Output: calibration report (aggregate curves — harness data never leaves staging).

### 14.3 Golden replays

Fixture event logs → expected state trajectories (snapshot-level) for regression detection in behavior rules.

### 14.4 Audio tests

- Offline AudioContext rendering of 500 calls/species: pairwise spectral-similarity metric asserts no two calls identical above threshold (the "hear it twice, spell breaks" test).
- Chorus artifact test: render 7-signature overlap windows; assert no loop-phase artifacts (spectral peak stationarity check) + structured human listen-test protocol per release.
- **Recognizability gate:** discrimination task (≥ 5 listeners, identify which of 7 signatures is calling) must pass ≥ 80 % before the 7th bird slot is enabled in prod (§15.3).

### 14.5 Client e2e (Playwright) + load

- Boot: no spinner/`role=status` ever present; first-bird paint < 500 ms on throttled profile; first frame shows mid-activity pose.
- Full keyboard map (§10.3); focus-proxy positions track birds; axe clean; `aria-live` rate ≤ 2/min idle.
- Listen-in: gain-ramp params asserted via mocked AudioParam timeline; others never reach 0.
- Settle: 5-s undo window; lighting ramp timing; presence suppression until click/keypress.
- Presence monitor: triple-condition truth table (visible/unfocused/focused/idle-too-long permutations) incl. "tab open all night ⇒ zero presence" regression.
- Offline: event queue caps, late-window drops, flush-on-reconnect ordering.
- Viewport matrix: no bird cropped at any size.
- Memory soak: 30-min session heap slope gate (§13.1).
- Load (staging): 50 k aviaries ticking, snapshot 5 k rps, events 1 k rps, magic-link burst.

### 14.6 Voice and copy lint

Every emittable string in `voice` + copy catalogs passes lint: naturalist surfaces — lowercase, no "you", no "!", banned families (welcome-back, streak, achievement, level, badge, days-visited); system surfaces — sentence case, direct, no naturalist phrasing. The lint is a CI gate, making I5 violations fail the build rather than the review.

### 14.7 Security and privacy tests

Token entropy/expiry/single-use tests for magic links, visit invites, export links; pre-GA pen test of the three token flows; telemetry schema-registry gate; log-scrubber sampling audit; session-revocation propagation test (API immediate, edge ≤ 60 s).

### 14.8 Manual matrices (per release)

Screen readers: VoiceOver/Safari (macOS+iOS), NVDA/Chrome, JAWS/Chrome, TalkBack/Chrome. Audio: structured listen sessions (chorus at 2/4/7 birds; mood-contour recognition; caption/what-you-hear match). Visual: reduced-motion walkthrough, night palette contrast, settle/undo feel.

---

## 15. Rollout

### 15.1 Milestones (≈ 22 weeks, team per §18)

| Milestone | Weeks | Exit criteria |
|---|---|---|
| M0 Foundations | 1–3 | Monorepo, CI with all gates scaffolded, design tokens, DB schema v1, edge shell + KV pipeline walking skeleton, `Clock` interface, auth spike (magic link e2e in dev). |
| M1 Engine core | 3–7 | Tick loop + scheduler, drift/mood/planner in `sim-core`, event log + ingest, snapshot API, calibration harness v0 running nightly; monotonicity + catch-up-equivalence green. |
| M2 Living scene | 5–10 | Canvas renderer, 2 species rigs, plan interpolation, day/night, weather, ornaments, responsive matrix, boot path with edge-inlined snapshot hitting 500 ms p75 in staging. |
| M3 Audio | 8–12 | Worklet synth, grammar expansion, signature system, chorus scheduling, listen-in mix, caption generation, silent fallback, variation + memory tests green. |
| M4 Interactions complete | 11–14 | Greeting resolver + client overlay, offers/reaction fast-path, settle/undo, notebook UI + chronicler tuning, presence monitor (triple condition) e2e, adoption/naming + first-fly-in, arrival mechanic. |
| M5 Accessibility & perf | 12–16 | Narration pipeline + rate limiter, reduced-motion renderer, keyboard/focus system, contrast CI, all perf gates green on reference hardware; SR manual matrix pass #1. |
| M6 Accounts & social | 14–18 | Settings surfaces, sessions/devices, export, deletion + purge job, visits end-to-end (invite/redeem/revoke/log/toggle), privacy schema gates. |
| M7 Hardening & closed beta | 17–20 | Load tests at target, pen test, drift recalibration vs. beta presence distributions (aggregate), all 6 species + 4 remaining rigs, voice review of chronicler output, beta runs 4 weeks. |
| M8 GA | 20–22 | All acceptance criteria (§2.3) met; runbooks; on-call rotation; rollback drills done. |

### 15.2 Feature flags (server config, audited)

`tick_cadence_active/dormant`, `drift_gains` (harness-gated), `presence_activity_window` (240 s), `presence_ping_interval` (30 s), `offer_cooldown` (180 s), `arrival_schedule`, `narration_cadence`, `visit_feature_kill` (global), `audio_default_captions`, `edge_inline_enabled`. Client flags ship in the snapshot envelope's `client_config` block so tuning never requires a deploy.

### 15.3 Birds-per-aviary ramp

- Launch: every aviary at 2 birds; arrival schedule active from day 0 of each account's life (60/120/210/300/365-day gates). Beta aviaries keep their age (no reset — I8 spirit).
- **Slot gates:** slot N unlocks in prod only when the recognizability discrimination test passes at N signatures (§14.4). Slot 7 is gated last; if the 7-bird test fails, the cap effectively holds at 6 in config while audio work continues — the cap is empirical and the gate makes that literal.
- Monitor (aggregate only): audio-error rate and caption-usage by bird-count bucket to detect mix degradation at higher counts.

### 15.4 Phased exposure

Internal alpha (team + ~20 accounts, 2 weeks; every notebook entry human-reviewed for voice) → closed beta (~500 invited accounts, 4 weeks; ≥ 5 recruited screen-reader users on real devices; drift recalibration from aggregate presence distributions) → GA (public signup; staged by edge region if needed).

### 15.5 Instrumented from day one

All §13.3 RUM + §13.4 alarms + tick/event/presence volume histograms + magic-link delivery + WebAudio-fallback rate + calibration-harness nightly report. Nothing per-account (§13.5).

### 15.6 Rollback posture

Client: immutable versioned assets, instant edge rollback. API: N−1 compatibility; snapshot `v` negotiation. Engine: drift-gain and cadence changes are config (revert in seconds); schema migrations forward-only with dual-read windows; tick state is catch-up-safe so a rolled-back worker fleet simply ticks late without corrupting anything (I10).

---

## 16. Risks and mitigations

| # | Risk | Sev × Lik | Mitigation | Early signal |
|---|---|---|---|---|
| R1 | **Drift miscalibration** — too fast ⇒ Tamagotchi; too slow ⇒ screensaver. The narrow band is the product. | High × Med | Harness owns targets (§14.2); gains in config, harness-gated; single-session Δ < 0.002 gate; beta recalibration from aggregate presence distributions. | Nightly calibration report curves; beta-era trait-movement histograms (aggregate). |
| R2 | **Personality-vector loss/corruption** — the invisible worst failure. | Catastrophic × Low | Server-only writer (single guarded repository method); PITR + dailies; monotonicity canary alarm (any negative delta pages); migration invariant tests; `ticks.drift_applied` audit trail. | Canary alarm; restore-drill results each quarter. |
| R3 | **Presence inflation bug** (the "tab open all night" class) silently corrupting drift population-wide. | High × Med | Triple-condition unit + e2e truth table; server 16 h/day clamp + alarm; union dedupe; late-window drops; harness persona asserts. | Presence-hours histogram tail; clamp-alarm rate. |
| R4 | **Audio uncanniness** — repetition, loop artifacts, unrecognizable signatures at 7 birds. | High × Med | No samples exist (asset lint); variation entropy test; chorus artifact test; recognizability gate before slot unlocks; structured listen tests per release. | Test metrics; caption/heard mismatch reports from beta SR+hearing-diverse users. |
| R5 | **Sync correctness** — duplicate/late/out-of-order events, clock skew, wake-from-sleep snaps. | Med × Med | Idem keys, `seq` ordering, 10-min late window, offset EWMA + clamp, plan cross-fade (≤ 400 ms), catch-up equivalence property test. | Event rejection-rate histogram; 304-rate; e2e wake tests. |
| R6 | **Announcement creep** — a well-meaning toast/badge/streak slips in. | High × Med | No toast/banner/badge primitive in the design system; copy lint CI (I5); PR checklist; notebook banned-pattern tests. | Lint failures (should trend zero after M2). |
| R7 | **Accessibility regression** — narration floods the SR queue, focus proxies drift, reduced-motion ships late. | High × Med | Rate limiter; a11y in every milestone exit criteria (§15.1); SR matrix per release; axe + contrast + live-region-rate CI gates; named owner from M1. | Manual matrix findings; narration-rate metric. |
| R8 | **Autoplay policy vs. "calls already audible."** | Med × High (certain on first visit) | Documented gesture-resume with 1 s fade; MEI opportunism; captions available; visual aliveness never gated on audio (§9.5). | WebAudio-fallback/suspend-rate RUM. |
| R9 | **500 ms first-bird missed on slow networks/cold edge.** | High × Med | Edge KV inline (no origin hop); quiet-field shell paints instantly regardless; critical chunk ≤ 300 KB; synthetic fleet p75 gate per commit. | Fleet first-bird p75 by geo. |
| R10 | **Tick scale** at growth beyond design target. | Med × Low | Catch-up-safe ticks ⇒ cadence tiering is free; sharded batched workers; load-tested at 50 k; Postgres partitioning; documented path to shard DB by `aviary_id` if > 500 k. | Worker queue depth; tick p99. |
| R11 | **Privacy-boundary erosion** — an account dimension sneaks into telemetry. | High × Low | Schema-registry CI gate; separate credentials/stores with no network route; log-scrubber audits; review rule: any metric PR touching `packages/telemetry` needs privacy sign-off. | Gate failures; quarterly audit. |
| R12 | **Notebook voice decay** — entries drift toward event-log phrasing. | Med × Med | Single `voice` module; banned-pattern tests; alpha/beta human review of entries; copy lint. | Beta voice-review notes. |
| R13 | **Mood "snap" perception on tab open.** | Med × Low | Mood always from snapshot with `mood_since`; no client default exists; plan cross-fade; e2e asserts continuity after hidden interval. | e2e; beta session-recording observation (consented, aggregate). |
| R14 | **Visit-token abuse / link forwarding.** | Low × Low | Hashed tokens, 7-day pass, instant revocation (≤ 20 s client + ≤ 60 s edge), rate limits, visit-log transparency. | Redeem-rate anomalies. |
| R15 | **Timezone flapping** across devices in different zones. | Low × Med | TZ updates only at `ctx=open`; lighting/day-phase blends smoothly (no snap); documented decision D8. | Support contacts; snapshot TZ-change rate. |

---

## 17. Decisions and interpretations (ambiguity log)

Defensible calls made where the PRD left room; each is a named, reversible-where-noted decision:

- **D1 — Canvas scene + DOM focus proxies** (vs. all-SVG/DOM): one compositor for birds+weather+ornaments makes the 60 fps/30-min and memory budgets easy; accessibility handled via proxies + live regions rather than per-element ARIA (which the PRD explicitly rejects as the "cheap version").
- **D2 — Postgres-only, no Kafka/queue backbone:** PRD's Kafka mention is a PII example; scale math (§6.2) doesn't justify a streaming platform. Append-only table + `seq` gives the ordering the no-LWW rule needs.
- **D3 — Presence parameters:** ping 30 s; activity window 240 s initial (PRD: "a few minutes, lean longer"); both config-tuned during build, harness-validated.
- **D4 — Offer cooldown 180 s per bird, shared across kinds** (PRD: "a few minutes"); config.
- **D5 — Mood set finalized:** wary, alert, content, curious, drowsy, roosted (PRD delegates finalization to implementation; roosted gives night a rendered posture and the nightjar its exemption).
- **D6 — Reaction fast-path at ingest:** a 1-min tick can't drive in-session offer reactions; the ingest response returns ephemeral directives computed by the same `sim-core` code, and the next tick ratifies mood/drift. Server-only-writer invariant is preserved (no personality/mood written at ingest).
- **D7 — Greeting computed server-side** at `ctx=open` (absence length is server knowledge; selection logic must honor boldness/mood/rotation and be procedurally varied — keeping it in `sim-core` makes it testable).
- **D8 — Account timezone = last active device's TZ**, updated at session open. Preserves one canonical aviary time (same mood across devices at any instant) while honoring "the user's local time is the aviary's time" for a single user who moves.
- **D9 — Presence union-dedupe across devices/tabs:** presence measures the user's attention, not their screens; overlapping intervals count once. Prevents multi-device drift inflation.
- **D10 — Visit link semantics:** "one-time link" interpreted as single *redemption* yielding a 7-day read-only pass (revocable, expiring), rather than single-use-then-dead — balances the PRD's "outstanding or active invite" language with the literal text. Reversible config (pass length).
- **D11 — Export includes personality vectors** (explicit in PRD) and is the *only* numeric exposure; no product surface renders them (I2 stands).
- **D12 — Arrival mechanic:** new-bird offers appear as an ambient back-perch silhouette + quiet top-bar affordance — no modal, no toast, no expiry, no nag (notice-never-announce applied to growth).
- **D13 — Narration generated server-side** from the shared `voice` module (voice consistency; client only rate-limits and inserts into the live region).
- **D14 — Focus-proxy accessible names** are `"{name}, {species}"`; the naturalist atmosphere lives in the running narration, not control labels (labels must be terse for SR usability; PRD's ban is on state-list *narration*, and a control name is not narration).
- **D15 — Presence daily caps** (8 h signal cap, 16 h sanity clamp) are anti-corruption hygiene, not product surfaces; no user-visible effect for honest use.
- **D16 — Autoplay:** first-gesture resume with 1 s fade is the platform-forced resolution of "calls already audible"; documented in §9.5, measured via fallback-rate RUM.
- **D17 — Client idle filler** when the plan horizon lapses: deterministic, low-commitment visuals only; never canonical; overridden by the next plan (the sanctioned exception to "client decides nothing").
- **D18 — No badges/unread indicators anywhere**, including the notebook icon (notice-never-announce; the notebook simply opens on its latest entries).
- **D19 — Offline event queue:** localStorage, ≤ 200 events / ≤ 10 min; older data dropped — presence/event honesty beats completeness.
- **D20 — Edge snapshot delivery:** sim worker pushes post-tick snapshots to Cloudflare KV; edge Worker inlines into HTML after JWT signature check (+ ≤ 60 s revocation cache). This is how the PRD's "snapshot delivered from a CDN edge with the HTML" is realized.
- **D21 — Initial drift gains** (§6.3) are starting points; the calibration harness is the source of truth and gates every change.
- **D22 — Settled + watching:** after settle, presence pings are suppressed until a click/keypress re-engages (mouse movement alone does not) — settle is a deliberate end-of-attention gesture per the PRD's "ends the presence window cleanly."
- **D23 — Visitor mode disables listen-in** (it is a trigger; visits are render-only) while keeping device-local a11y settings (captions/mute/reduced motion), which are not interactions with the host's aviary.
- **D24 — Species pool (6), sketch — final art/audio by design:** warbler (high perches, three-note rises), wren (busy short phrases), finch (cheerful chatter), thrush (fluted dawn phrases), nightjar (nocturnal soft churrs), dove (low calm coos). Song-fragment library: 5 procedural motifs ("morning rise", "rain thread", "two-note question", "low hum", "bright scatter").

---

## 18. Team shape and sequencing

≈ 22 weeks, 7 engineers + 1 visual designer (+ PM part-time):

- **2 engine/backend** — `sim-core`, tick worker, API, data model, privacy infrastructure (M0–M1 lead, then accounts/visits M6).
- **2 client/render** — renderer, rigs, boot path, responsive layout, reduced-motion register (M0 shell → M2 lead → M4 interactions).
- **1 audio** — worklet synth, grammar expansion, mix, captions (M3 lead; species grammar authoring with designer).
- **1 a11y/QA-infra** — narration pipeline, keyboard/focus, CI gates, harnesses, e2e, SR matrices (M5 lead; gates from M0).
- **1 full-stack/platform** — edge Worker/KV, CI/CD, observability, load tests, mailer (M0–M8 continuous).
- **Visual designer** — palette/contrast spec, rig silhouettes, focus-indicator treatment, reduced-motion pose libraries, top-bar iconography.

Dependency spine: M0 → M1 (engine) ∥ M2 (scene) → M3 (audio needs call specs from M1) → M4 (interactions need M1+M2+M3) → M5 (a11y needs M2+M3 surfaces) → M6 (accounts/social largely parallel from M4) → M7 → M8. `sim-core` and the CI gates land first so every later merge is measured against the invariants from day one.

---

## Appendix A — Snapshot JSON schema (v3, full)

```jsonc
{
  "v": 3,
  "state_rev": 918273,                 // ETag; monotonic per aviary
  "server_ts": "2026-09-06T07:42:03.214Z",
  "client_config": {                    // flag mirror (§15.2), small
    "presence_ping_s": 30, "presence_window_s": 240,
    "snapshot_keepalive_s": 20, "offer_cooldown_s": 180
  },
  "aviary": {
    "id": "uuid",
    "local_time": "07:42",
    "day_phase": "dawn|morning|midday|evening|night",
    "light": { "palette_key": "dawn_warm", "blend": 0.35 },
    "weather": null | { "type": "rain|wind", "intensity": 0.0-1.0,
                        "started_at": "…", "ends_at": "…" },
    "settled": false,
    "pool_until": null | "…",
    "arrival": null | { "arrival_id": "uuid", "species_silhouette": "finch",
                        "perch": { "zone": 0, "x": 0.7 }, "since": "…" },
    "age_days": 143
  },
  "birds": [                            // 2–7 entries
    {
      "id": "uuid", "name": "pip", "species": "warbler",
      "mood": "wary|alert|content|curious|drowsy|roosted",
      "mood_since": "…",
      "perch": { "zone": 0|1|2, "x": 0.0-1.0 },
      "plan": [
        { "activity": "preen|scan|doze|weight_shift|head_tilt|call|fly_to|hop_to|drink|bathe|watch_pool|investigate|greet|roost",
          "starts_at": "…", "ends_at": "…",
          "perch": { "zone": 2, "x": 0.31 },       // destination for fly_to/hop_to
          "params": { /* activity-specific, render-only */ } }
      ],
      "plan_until": "…",
      "offer_cooldown_until": null | "…",
      "render": {
        "plumage": { "base": "#8a6f4d", "accent": "#c9a86a",
                     "sat_level": 0-4, "detail": 0-4 },
        "idle_energy": 0.0-1.0, "size": 0.7-1.1,
        "signature": { "pitch_center_hz": 2400, "brightness": 0.6,
                       "vibrato": 0.3, "attack": 0.02, "tempo_bias": 1.05 }
      }
    }
  ],
  "calls": [                            // ~90 s schedule horizon
    { "bird_id": "uuid", "at": "…",
      "spec": { "motif_seed": 8841, "contour": "rise-3|fall-2|flat-4|trill|churr",
                "energy": 0.0-1.0, "pitch_mul": 0.9-1.15,
                "repeat": null | { "times": 2, "gap_ms": 420 } } }
  ],
  "fragment": null | { "fragment_id": "morning_rise", "at": "…" },
  "greeting": null | {
    "bird_id": "uuid",
    "form": "glance|head_tilt|two_note|step_call|reorient|long_call",
    "at_offset_ms": 800-2000,
    "call_spec": { /* as above */ } | null,
    "second": null | { "bird_id": "uuid", "form": "…",
                       "at_offset_ms": 2300-6000, "call_spec": null }
  },
  "narration": { "rev": 42, "line": "a warbler perches on the high branch, calling softly.",
                 "priority": null | { "line": "…", "event": "greeting|offer|settle" } },
  "visit_mode": false                   // true on visitor snapshots; disables events
}
```

## Appendix B — Event taxonomy (`POST /v1/aviary/events`)

| type | payload | validation | engine use |
|---|---|---|---|
| `presence` | `{since, until}` (≤ 35 s span) | triple-condition asserted client-side; server: span ≤ 35 s, late ≤ 10 min, union-dedupe | drift `S_pres` |
| `listen_in_start` | `{bird_id}` | bird in aviary | drift `S_listen` opens |
| `listen_in_end` | `{bird_id, duration_s}` | ≤ 4 h cap/session | drift `S_listen` closes |
| `offer` | `{kind: seed|song|pool, song_id?, target_bird_id?}` | cooldown 180 s/bird else reject `cooldown`; reaction returned synchronously | drift `S_accept`/`S_near`; mood nudge at tick |
| `settle` | `{}` | — | ends presence window; mood-quieting nudge |
| `settle_undo` | `{}` | ≤ 5 s after settle | supersedes settle |
| `reengage` | `{}` | only valid while settled | resumes presence |
| `rename` | `{bird_id, name}` | ≤ 24 chars | canonical rename (also via PATCH; event logged for notebook context) |
| `adopt` | `{arrival_id, name?}` | slot open per age gate | creates bird (also via POST /birds/adopt; event mirrors) |
| `session_open` | `{tz}` | — | greeting resolver trigger; TZ update |
| `server_reaction` | (server-appended) | — | tick continuity for fast-path directives |

All events: `idem_key` required; unknown types rejected; **no event type carries state values** (I1).

## Appendix C — System-surface copy catalog (matter-of-fact; exhaustive for v1)

| code | copy |
|---|---|
| `auth_link_expired` | "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| `auth_link_used` | "This link has already been used. Request a new one to sign in." |
| `session_timeout` | "Your session timed out. Sign in again to keep watching." |
| `aviary_load_error` | "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." |
| `unsupported_browser` | "Pocket Aviary needs a recent browser: the last two versions of Chrome, Safari, Firefox, or Edge. Yours isn't supported." |
| `visit_revoked_or_expired` | "This visit is no longer available." |
| `export_ready_email` | "Your Pocket Aviary export is ready. Download it within 24 hours: {link}" |
| `deletion_pending_bar` | "Your account is scheduled for deletion on {date}. [Keep my account]" |
| `email_change_verify` | "Confirm your new email address to finish the change: {link}. Your old email keeps working until you do." |
| `visit_notify` | "A friend visited your aviary today." |
| `rate_limited` | "Too many requests. Wait a moment and try again." |

Naturalist-voice surfaces (narration, notebook, captions, naming/adoption sheets, offer sheet) draw from the `voice` module and linted catalogs only; the two catalogs never mix registers (brief §Voice).

## Appendix D — Species pool (sketch; design finalizes)

| id | silhouette | palette family | grammar family | diurnal |
|---|---|---|---|---|
| warbler | small, high-branch | olive/gold | three-note rises, thin clear | yes |
| wren | compact, busy | russet/buff | rapid short phrases, loud-for-size | yes |
| finch | stocky, seed-forward | ochre/green | cheerful chatter pairs | yes |
| thrush | plump, upright | brown/cream | fluted melodic phrases (dawn peak) | yes |
| nightjar | long-winged, low | grey-brown mottled | soft churrs, frog-like | **no** |
| dove | round, calm | slate/blue-grey | low coos, slow tempo | yes |

## Appendix E — Config registry (initial values)

| key | initial | notes |
|---|---|---|
| `tick_cadence_active_s` / `tick_cadence_dormant_s` | 60 / 300 | dormant = no presence 72 h |
| `drift_gains` | §6.3 table | harness-gated changes |
| `drift_alpha` | 0.15 | EMA low-pass |
| `presence_ping_s` / `presence_window_s` | 30 / 240 | calibrate during build, lean longer |
| `presence_daily_cap_h` / `presence_sanity_clamp_h` | 8 / 16 | drift input cap / bug alarm |
| `offer_cooldown_s` | 180 | per bird |
| `mood_min_dwell_s` | 600 | |
| `greeting_offset_ms` | 800–2000 | first-greet window |
| `greeting_stagger_ms` | 1500–4000 | second bird |
| `snapshot_keepalive_s` | 20 | while visible |
| `plan_horizon_s` | 120–300 | per tick |
| `call_horizon_s` | 90 | in snapshot |
| `narration_min_s` / `narration_max_s` | 30 / 60 | idle cadence |
| `notebook_min_gap_h` / `notebook_weekly_cap` | 36 / 3 | rarity governor |
| `arrival_schedule_d` | [0:2, 60:3, 120:4, 210:5, 300:6, 365:7] | slot 7 gated on recognizability test |
| `visit_pass_days` / `invite_expiry_d` | 7 / 30 | D10 |
| `magic_link_ttl_min` / `rate_per_h` | 15 / 5 | |
| `session_ttl_d` | 30 sliding | |
| `soft_delete_days` | 30 | then hard purge |
| `event_late_window_min` / `offline_queue_cap` | 10 / 200 | |
| `listen_in_floor` / `ramp_tau_s` | 0.30 / 0.5 | others never silent |
| `weather_rain_per_week` | 2–3 | seeded schedule |

— End of plan. Deliverable is this document only; no product code has been written.
