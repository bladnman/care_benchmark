# Pocket Aviary — v1 implementation plan

This plan is an executable engineering document for a separate team. It interprets the PRD into service boundaries, schemas, algorithms, APIs, client pipelines, tests, and rollout gates. It does not implement the product.

Wherever the PRD is silent, this plan makes a named call and records it under **Decisions**. Those calls are binding for v1 unless a later change-control note replaces them.

---

## 0. Decisions (ambiguities resolved)

| ID | Topic | Call | Why |
|---|---|---|---|
| D1 | Presence activity window | **4 minutes** without pointermove/keypress before presence drops, given visibility+focus still hold | PRD says “a few minutes, lean longer”; watching without moving is the product |
| D2 | Presence ping cadence | Client emits a presence ping every **30s** while the three-signal conjunction holds; server reconstructs windows | Dense enough to survive a dropped packet; sparse enough to stay cheap |
| D3 | Multi-device presence | **Union of time, never sum.** Two devices present in the same minute count as one minute | Prevents dual-device inflation of the dominant drift input |
| D4 | Tick cadence | **60s** for aviaries with any event or snapshot in the last 24h. **5 min** for 1–14 day idle. **Lazy catch-up** on next snapshot for >14 day idle | Continuity without burning compute on abandoned accounts |
| D5 | Trait range | Each personality trait is `numeric(6,5)` in **[0.00000, 1.00000]** | Small, stable, enough precision for week-1 instrument deltas |
| D6 | Starter trait band | New birds seed in **[0.22, 0.42]** with complementary profiles | Room to drift toward expressive; two starters must read as different animals, not twins |
| D7 | Mood enum | `wary \| content \| curious \| drowsy \| alert` | Matches PRD examples; night sleeping is `drowsy` plus a render flag, not a sixth mood |
| D8 | Offer cooldown | **4 minutes per bird per offer-kind**, server-enforced | Stops curiosity/boldness saturation inside one session |
| D9 | Notebook generation | **Deterministic observation compiler + hand-authored phrase banks.** No live LLM. No warehouse-side generation | Voice control, privacy boundary, reproducibility |
| D10 | Renderer | **Canvas 2D scene + DOM chrome overlay.** Shared scene-graph. Reduced-motion is a second draw path, not “animations off” | 60fps budget, mid-action first frame, captions/focus rings stay in DOM |
| D11 | Event log | **Postgres append-only table**, UUID keys, no Kafka in v1 | Avoids PII-in-partition-key failure; v1 volume does not need a log cluster |
| D12 | Account timezone | IANA string from client on each session start; last-write of timezone is allowed (not personality) | Day/night and mood must follow the user, not UTC |
| D13 | Song library | **6 built-in fragments**, ~2–4s each, motif-params not audio files | Enough variety for “small library”; stays inside bundle and “no recorded audio” |
| D14 | Visit opt-in notify | Email only, off by default, never push, never in-aviary toast | PRD allows a settings toggle; email is the only v1 channel |
| D15 | New-bird pacing | 3rd at **56 days** aviary age; then 4th at 120d, 5th at 210d, 6th at 300d, 7th at 420d | Age, not attention. Months-scale deepening |
| D16 | Greeting absence bands | `<15m` glance; `15m–12h` short call + tilt; `>12h` re-orientation | Makes coffee-return feel different from two-day return |
| D17 | Session snapshot auth | Cookie session (`HttpOnly`, `Secure`, `SameSite=Lax`) plus `Authorization: Bearer` for the same token on fetch | Lets the HTML document and XHR share one session |
| D18 | Fast offer path | Offers and greetings are **synchronous apply-event** responses that mutate *ephemeral* snapshot fields only. Personality still waits for the tick | Reactions must be immediate; drift must stay slow |
| D19 | Narration author | Shared `prose` package runs on the **server** and is attached to snapshots. Client may locally prioritize event lines but does not invent a second voice | One voice across notebook, live region, captions |
| D20 | Export personality numbers | Allowed in the JSON takeout only. Never rendered in any UI, including settings and debug overlays in production builds | PRD explicitly includes vectors in export and forbids showing them |
| D21 | Stack | TypeScript everywhere. Next.js (App Router) for the web shell + edge HTML. Node.js API. Postgres 16. Redis for magic-link/rate-limit/session revoke. Fly/Cloudflare edge for HTML+bootstrap snapshot | One language, last-two-browser support, small team |
| D22 | No client sim | The browser never advances mood, perch intent, weather, or personality. It only interpolates poses and synthesizes audio from snapshot instructions | Prevents dual-device fork |

---

## 1. Scope

### 1.1 In v1

- Browser-only Pocket Aviary. One account, one aviary, two starter birds, hard cap of seven.
- Magic-link auth, per-device revocable sessions, email change with verify-first, soft-delete 30 days then hard-delete.
- Server-side simulation tick. Canonical personality, mood, perch intent, weather, and notebook authorship live only on the server.
- Multi-device sync as a property of “server is the only writer,” not as a merge protocol.
- Interactions: return-greeting, idle presence, listen-in, offer (seed / song fragment / still pool), settle (+ 5s undo), field notebook (read-only, sparse).
- Age-gated adoption of birds 3–7. System chooses species. User names and can rename.
- Optional visit invites: per-invite email, read-only, revocable, 30-day unused expiry, silent visit log, notify-off-by-default.
- Accessibility as designed surfaces: naturalist screen-reader narration, reduced-motion cross-fade renderer, runtime call captions, WCAG AA chrome, full keyboard path.
- Performance floors: gzipped initial JS **< 2MB**, time-to-first-bird **< 500ms** on mid-tier 4G, 60fps idle on a 5-year-old laptop, no client memory growth over 30 minutes.
- Account export JSON emailed on demand. Privacy policy link in settings.

### 1.2 Out of v1 (non-goals, enforced)

Do not build, stub, or leave feature flags for:

- Native iOS/Android clients, or protocols designed around native constraints.
- Any gamification: scores, quests, streaks, badges, levels, XP, ranks, tiers, “birds adopted: N”, green-dot calendars, “you visited every day,” milestone celebrations.
- Tamagotchi mechanics: death, hunger, distress, decaying happiness, guilt-on-return, “you didn’t settle” recovery.
- Social network surfaces: profiles, follows, public discovery, explore, comments, chat, avatars, co-presence, leaderboards, show-off rendering.
- Push/email pings about the aviary itself. (Magic-link, visit-invite, export-link, and the *opt-in* visit-notify email are the only mail.)
- Payments, multi-aviary accounts, shared/household aviaries, customizable scenes, species rarity, recorded-audio fallback, personality-number UI, last-write-wins personality writes.

If a ticket would teach the user that presence is for a counter, it is rejected.

### 1.3 What “done” means

A v1 account can sign in on two browsers, see the same birds mid-action within 500ms, be noticed by one bird, sit without clicking and have that presence counted honestly, listen in, offer, settle or just close the tab, read a sparse notebook, export or delete the account, and — if they choose — invite one friend to watch read-only. After three weeks of regular use, the birds feel more themselves. After two weeks away, they are quieter, not wounded.

---

## 2. Architecture

### 2.1 Shape

```
                    ┌───────────────┐
   browser SPA ────►│  edge / web   │  HTML + quiet-field CSS + bootstrap snapshot
                    └───────┬───────┘
                            │
                    ┌───────▼───────┐
                    │  api (BFF)    │  authn, input validation, voice-split errors
                    └─┬─────┬─────┬─┘
           events     │     │     │ snapshots / account / visits
                      │     │     │
            ┌─────────▼┐ ┌──▼──┐ ┌▼──────────┐
            │ event log│ │auth │ │ sim reader│
            │ (append) │ │+mail│ │ (hot row) │
            └────┬─────┘ └─────┘ └─────▲─────┘
                 │                     │
                 │            ┌────────┴────────┐
                 └───────────►│  sim tick worker│  only writer of personality,
                              │  (per-shard)    │  mood, perch intent, weather,
                              └────────┬────────┘  notebook candidates
                                       │
                              ┌────────▼────────┐
                              │  postgres 16    │  accounts, birds, events,
                              │                 │  snapshots, notebook, visits
                              └─────────────────┘
```

Redis holds: magic-link hashes (15m TTL), per-email request counters, session-revocation denylist, short-lived export-download tokens. Redis is never a source of personality.

### 2.2 Process split

| Process | Owns | Must not own |
|---|---|---|
| `apps/web` | Render, input, WebAudio, presence signals, interpolation | Personality, mood transitions, weather, notebook authorship, visit grant |
| `services/api` | HTTP, auth, validation, snapshot reads, event append, visit tokens | Drift math |
| `services/sim` | Tick, drift, mood, perch intent, weather, greeting plan, offer reaction, notebook compiler | Telemetry warehouse, email content beyond structured facts |
| `services/mail` | Magic links, invites, export links, opt-in visit mail | Any bird field in templates except none |
| `packages/prose` | Naturalist + matter-of-fact string tables and compilers | Numbers for UI |
| `packages/snapshot` | Versioned snapshot/event TypeScript types shared by api, sim, web | — |
| `packages/call-grammar` | Motif graphs, caption templates, timing functions (pure) | Audio buffers |

### 2.3 Render-pipeline boundary

The client consumes a **snapshot** plus a monotonic `server_time`. It builds a scene graph and draws. It does not invent bird intent.

```
snapshot ──► SceneGraph.hydrate()
                 │
                 ├─ Interpolator (perch A→B, pose cross-fade or path)
                 ├─ IdleDirector (loops seeded by snapshot.motion_phase)
                 ├─ WeatherPainter (from snapshot.weather)
                 ├─ PaletteDirector (local clock × snapshot.sun_override for settle)
                 └─ AudioDirector (grammar seeds + mix targets)
                          │
                          ▼
              Canvas2DRenderer  |  ReducedMotionRenderer
                          │
              DOM: top bar, captions, focus rings, aria-live
```

When the tab is hidden: cancel animation frames, suspend AudioContext, stop leaf ornaments. Simulation continues on the server. On visible again: pull a fresh snapshot, hydrate mid-action, resume.

### 2.4 Privacy as architecture

Two databases-or-schemas, one physical Postgres is acceptable if grants enforce the split:

- `sim` schema: birds, vectors, events, notebook, snapshots. **No analytics role.**
- `ops` schema: request logs with `account_id` UUID only, latency, error codes.

Rules:

- Email ciphertext lives on `accounts.email_ciphertext` and nowhere else. Lookup via `email_hash = HMAC-SHA256(pepper, normalized_email)`.
- Every log line, span, queue name, cache key, and filename uses `account_id` UUID.
- RUM and synthetics carry no bird ids, no species, no interaction types beyond HTTP route.
- The warehouse (if any) is fed only from `ops`. A CI check fails the build if a sim table is referenced from analytics dbt/SQL.
- Staff access to `sim` is break-glass, audited, and never used to compute “average drift across users” as a product dashboard.

### 2.5 Repo layout

```
apps/web/                  # Next.js App Router, canvas runtime, audio
services/api/              # Fastify or similar, session middleware
services/sim/              # tick worker + apply-event
services/mail/             # transactional mail
packages/snapshot/         # zod types, snapshot version N
packages/prose/            # compilers
packages/call-grammar/     # pure audio grammar
packages/species/          # six species defs (visual + grammar ids)
infra/                     # migrate, terraform/fly, synthetics
```

---

## 3. Data model

All primary keys are UUIDv7 (time-ordered, not email-derived).

### 3.1 Accounts and sessions

```
accounts
  id                    uuid pk
  email_ciphertext      bytea not null          -- AES-GCM, key in KMS
  email_hash            bytea unique not null   -- HMAC pepper
  timezone              text not null default 'UTC'  -- IANA
  created_at            timestamptz not null
  marked_for_deletion_at timestamptz null
  visit_notify_email    boolean not null default false
  a11y                  jsonb not null default '{}'
      -- { captions: bool, reduced_motion_override: 'system'|'on'|'off' }
  settings              jsonb not null default '{}'

sessions
  id                    uuid pk
  account_id            uuid not null references accounts
  token_hash            bytea unique not null
  created_at            timestamptz not null
  last_seen_at          timestamptz not null
  revoked_at            timestamptz null
  user_agent_label      text not null           -- "Safari on iPhone", not raw UA dump in logs

magic_links
  id                    uuid pk
  email_hash            bytea not null
  account_id            uuid null               -- null if first-time
  purpose               text not null           -- 'sign_in' | 'email_change' | 'export'
  token_hash            bytea unique not null
  new_email_ciphertext  bytea null              -- email_change only
  expires_at            timestamptz not null    -- now()+15m for auth
  consumed_at           timestamptz null
```

First magic-link for an unknown email **creates the account and aviary lazily on consume**, not on request. That avoids account spam from typed-wrong addresses. Rate-limit requests per `email_hash` at 5 / 15 minutes, 20 / day.

### 3.2 Aviary and birds

```
aviaries
  id                    uuid pk
  account_id            uuid unique not null
  created_at            timestamptz not null    -- age gate for new birds
  last_tick_at          timestamptz not null
  last_host_presence_at timestamptz null
  weather               jsonb not null
      -- { kind: 'clear'|'rain'|'wind', started_at, ends_at }
  next_bird_eligible_at timestamptz null
  bird_offer_pending    boolean not null default false

species
  id                    text pk                 -- 'warbler', 'wren', 'finch',
                                                -- 'sparrow', 'dove', 'nightjar'
  nocturnal             boolean not null
  motif_library_id      text not null
  silhouette_id         text not null
  default_palette       jsonb not null

birds
  id                    uuid pk                 -- STABLE forever
  aviary_id             uuid not null
  species_id            text not null
  name                  text not null           -- user-facing, renameable
  adopted_at            timestamptz not null
  grammar_seed          int not null            -- call identity
  visual_seed           int not null            -- plumage variation
  boldness              numeric(6,5) not null
  social_warmth         numeric(6,5) not null
  vocal_frequency       numeric(6,5) not null
  plumage_saturation    numeric(6,5) not null
  curiosity             numeric(6,5) not null
  mood                  text not null
  mood_since            timestamptz not null
  perch_zone            text not null           -- 'front'|'middle'|'back'
  perch_slot            smallint not null
  last_offer_at         jsonb not null default '{}'  -- {seed, song, pool} → iso
  -- CHECK each trait between 0 and 1
  -- UNIQUE (aviary_id, id)
```

Identity rule: migrations may add columns; they may never delete a bird row and insert a replacement with a new id. Species-pool edits remap visuals/motifs in place.

### 3.3 Expressiveness gate (neglect without punishment)

Personality never decreases. Quietness after absence is a separate decaying gate:

```
aviary_expressiveness
  aviary_id             uuid pk
  gate                  numeric(6,5) not null   -- [0.15, 1.00]
  updated_at            timestamptz not null
```

- Presence raises `gate` slowly toward 1.
- Time without host presence lowers `gate` slowly toward 0.15.
- Greeting probability, chorus join rate, and approach-to-offer use `effective = trait * mix(gate, 0.55)` so a bold bird away for two weeks is quieter, not warier, and snaps back by being watched — not by apology.

This is the engine-level implementation of “ambient, not distressed.”

### 3.4 Event log

```
events
  id                    uuid pk
  account_id            uuid not null
  aviary_id             uuid not null
  bird_id               uuid null
  type                  text not null
  payload               jsonb not null
  client_event_id       uuid not null
  session_id            uuid not null
  client_occurred_at    timestamptz not null
  received_at           timestamptz not null default now()
  processed_at          timestamptz null
  UNIQUE (account_id, client_event_id)
```

`type` enum:

- `presence_ping` — `{ window_id, signals: { visible, focused, activity_at } }`
- `listen_in_start` / `listen_in_end` — `{ bird_id, duration_ms? on end }`
- `offer` — `{ kind: 'seed'|'song'|'pool', near_bird_id, song_id? }`
- `settle` / `settle_undo`
- `session_end` — tab close / pagehide (best-effort)
- `rename` — `{ bird_id, name }` (also written through a dedicated endpoint; event is for notebook/tick awareness only)
- `adopt` — server-authored when a new bird is created

Clients never send trait values. A payload validator drops unknown keys.

### 3.5 Snapshots (materialized, not derived from full history)

```
snapshots
  aviary_id             uuid pk
  version               int not null            -- schema version
  body                  jsonb not null
  updated_at            timestamptz not null
```

Snapshot body (conceptual TypeScript):

```ts
type Snapshot = {
  version: 1
  server_time: string
  aviary_id: string
  aviary_age_days: number
  sun: { phase: number; /* 0=midnight.. */ local_hour: number }
  weather: { kind: 'clear' | 'rain' | 'wind'; intensity: number }
  settle: { active: boolean; undo_until?: string }
  greeting?: GreetingPlan
  offer_reaction?: OfferReaction
  bird_offer?: { available: boolean } // no countdown, no “earn this”
  narration: { text: string; priority: 'idle' | 'event'; issued_at: string }
  birds: BirdSnap[]
}

type BirdSnap = {
  id: string
  species_id: string
  name: string
  mood: Mood
  perch: { zone: Zone; slot: number; arrived_at: string }
  next_perch?: { zone: Zone; slot: number; eta: string }
  motion_phase: number          // 0..1 into current idle cycle
  motion_kind: 'preen' | 'scan' | 'tilt' | 'shuffle' | 'rest' | 'call'
  plumage: { seed: number; saturation: number } // saturation is visual only; do not label
  call: {
    grammar_seed: number
    next_window: [string, string] // when a call may start
    last_caption?: string
  }
  listen_in_allowed: true
}
```

**Never in the snapshot:** raw trait names or numbers, visit counts, streak-like fields, “days since last visit,” other accounts.

### 3.6 Notebook

```
notebook_entries
  id                    uuid pk
  aviary_id             uuid not null
  prose                 text not null
  created_at            timestamptz not null
  trigger               text not null           -- internal, never served
  UNIQUE (aviary_id, id)
```

Serve `id, prose, created_at` only, oldest-unbounded, cursor pagination.

Sparsity limiter on `aviaries`: `last_notebook_at`, plus a per-aviary token bucket (1 token / 48h, burst 2 when `trigger` is in `{first_greeter_swap, new_bird, first_rain_together}`).

### 3.7 Visits

```
visit_invites
  id                    uuid pk
  host_account_id       uuid not null
  visitor_email_hash    bytea not null
  visitor_email_ciphertext bytea not null
  token_hash            bytea unique not null
  created_at            timestamptz not null
  expires_at            timestamptz not null    -- +30d
  used_at               timestamptz null
  revoked_at            timestamptz null

visit_sessions
  id                    uuid pk
  invite_id             uuid not null
  started_at            timestamptz not null
  last_snapshot_at      timestamptz not null
  ended_at              timestamptz null
```

Visitor auth is the invite token (separate cookie, `role=visitor`). Visitor snapshot is the host snapshot **minus** `greeting`, `offer_reaction`, `bird_offer`, notebook, and with `listen_in_allowed: false`. No events from visitor sessions are inserted into `events`.

### 3.8 Indexes that matter

- `events (aviary_id, received_at) WHERE processed_at IS NULL`
- `events (account_id, client_event_id)` unique
- `birds (aviary_id)`
- `notebook_entries (aviary_id, created_at desc)`
- `sessions (token_hash)`
- `accounts (email_hash)`
- `visit_invites (token_hash)`

No index or materialized view that ranks aviaries by visits, bird count, or age for a public surface. Do not compute those aggregates “just in case.”

---

## 4. API surface

Base: `/v1`. All host routes require a valid session. Errors use matter-of-fact copy from `packages/prose/system`. Success bodies for product surfaces may include naturalist strings already compiled (narration, notebook). Never return trait numbers except `GET /v1/account/export` (authenticated, generated async).

### 4.1 Auth

| Method | Path | Notes |
|---|---|---|
| `POST` | `/v1/auth/magic-link` | `{ email }`. Always 202 with the same body: “If that address is valid, we sent a link.” Do not reveal account existence. |
| `GET` | `/v1/auth/consume` | `?token=`. One-time. Sets session cookie. Invalid/expired: matter-of-fact page, no naturalist. |
| `POST` | `/v1/auth/sign-out` | Revoke this session |
| `GET` | `/v1/account` | Profile: email masked, timezone, sessions list, deletion state |
| `PATCH` | `/v1/account` | `{ timezone?, a11y?, visit_notify_email? }` |
| `POST` | `/v1/account/email-change` | Sends link to **new** address; old remains until consume |
| `DELETE` | `/v1/account/sessions/:id` | Revoke device |
| `POST` | `/v1/account/delete` | Soft-delete now |
| `POST` | `/v1/account/restore` | Allowed only if `marked_for_deletion_at` within 30d |
| `POST` | `/v1/account/export` | Enqueues export; emails download link to verified address |
| `GET` | `/v1/account/export/:token` | One-time download, 1h TTL |

Magic-link consume algorithm:

1. Hash token, lookup, reject if missing / expired / consumed.
2. Mark consumed in the same transaction as session insert.
3. If no account: create `accounts`, `aviaries`, two `birds` (see §6.8), `aviary_expressiveness`, empty notebook.
4. Issue session token (32 random bytes, store hash).

Replay of a consumed link is an error, not a new session.

### 4.2 Aviary state and events

| Method | Path | Notes |
|---|---|---|
| `GET` | `/v1/aviary/snapshot` | Canonical body. Query `reason=start\|visible\|wakeup\|poll`. `start`/`visible`/`wakeup` may attach a new `greeting` plan. |
| `POST` | `/v1/aviary/events` | Batch `{ events: [...] }`, max 32. Idempotent on `client_event_id`. Returns `{ accepted, snapshot? }`. |
| `POST` | `/v1/aviary/apply` | Fast path for `offer`, `settle`, `settle_undo`, `listen_in_start/end`. Writes event **and** returns an updated snapshot with ephemeral reaction. Does **not** write personality. |
| `PATCH` | `/v1/birds/:id` | `{ name }` only. 1–24 chars, trimmed, no empty. |
| `POST` | `/v1/aviary/adopt` | Allowed iff `bird_offer_pending` and count < 7. Server picks species, seeds traits, clears flag. |
| `GET` | `/v1/notebook?cursor&limit` | `limit` default 20, max 50. Read-only. |
| `GET` | `/v1/offers/library` | Six song fragment ids + naturalist labels, no audio assets |

Event ingest rules:

- Reject if `client_occurred_at` is > 10 minutes in the future or > 24h in the past (presence pings older than 10 minutes are dropped).
- Reject visitor sessions.
- Reject if account `marked_for_deletion_at` is set (except restore).
- `presence_ping` ignored unless payload includes the three signals all true at client; server does not “upgrade” a partial ping.

Snapshot pull reasons:

- `start` — first paint after navigation. Run lazy tick catch-up if needed. Build greeting from absence length.
- `visible` — `visibilitychange` to visible. Same greeting rules; short-absence glance.
- `wakeup` — `document.timeline` gap > 30s (laptop sleep). Treat like visible.
- `poll` — every **45s** while visible. No greeting. Cheap read.

### 4.3 Visit flow

| Method | Path | Notes |
|---|---|---|
| `POST` | `/v1/visits/invites` | `{ email }`. Creates invite, mails one-time link. Default off — this *is* the opt-in. |
| `GET` | `/v1/visits/invites` | Outstanding + recently used, for settings |
| `DELETE` | `/v1/visits/invites/:id` | Revoke immediately |
| `GET` | `/v1/visits/log` | Email (decrypted in-process), date, approx duration. No badge counts in any other payload |
| `GET` | `/v1/visit/:token/snapshot` | Visitor. If revoked/expired/missing: matter-of-fact “This visit is no longer available.” |
| `POST` | `/v1/visit/:token/end` | Best-effort; also inferred from snapshot silence > 10 min |

Visitor client is a stripped route `/visit/[token]`: same renderer, no top-bar offer/settle/notebook, no greeting playback that would look like the birds noticed the visitor. Birds do not re-greet. Host snapshot is not mutated. Host is not told in-session.

Revocation: `revoked_at = now()`. Next visitor snapshot returns 410. No host toast.

### 4.4 Error voice

Map status → `packages/prose/system`:

- 401 consume fail: “We couldn't sign you in. The link may have expired. Try requesting a new link.”
- 401 session: “Your session timed out. Sign in again to keep watching.”
- 500 snapshot: “Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch.”
- 410 visit: “This visit is no longer available.”
- 415/old browser (from edge): “This browser isn't supported. Try the latest Chrome, Safari, Firefox, or Edge.”

No bird names in error strings. No “welcome back” anywhere, including 200 HTML.

---

## 5. Simulation engine

### 5.1 Tick worker

Sharded by `aviary_id` hash into N worker leases (Postgres `FOR UPDATE SKIP LOCKED` on a `tick_lease` table). A tick for one aviary is a single transaction:

1. Lock aviary row.
2. If `now - last_tick_at` is small and no unprocessed events, exit.
3. Load birds, expressiveness, weather, unprocessed events ordered by `(received_at, id)`.
4. Fold events into an `InputWindow` (presence seconds, listen-in seconds per bird, offers, settle).
5. Apply expressiveness gate.
6. Apply drift deltas (clamped).
7. Step weather.
8. Step moods.
9. Step perch intents.
10. Maybe write a notebook entry.
11. Maybe set `bird_offer_pending` from aviary age.
12. Write snapshot JSON.
13. Mark events `processed_at`.
14. Set `last_tick_at`.

Catch-up for long idle: iterate in **simulated 5-minute steps** from `last_tick_at` to now, with empty event windows except the real events placed in their step. Cap work at 500 steps per request (≈41h at 5 min); if more remain, schedule a background continuation and still serve the latest completed snapshot. Mood/sun are functions of local time, so skipped steps still land on the correct time-of-day attractor.

**p99 tick budget:** alarm if apply duration > 5s. Expected p50 < 20ms per active aviary.

### 5.2 Presence reconstruction

From pings sharing a `window_id`:

- Sort by `client_occurred_at`.
- A gap > 90s closes the window.
- Duration = last.ping − first.ping + 30s (one cadence of credit), clipped to the period the three signals were claimed.
- Overlapping windows from two sessions: merge intervals (union).
- `settle` or `session_end` closes any open window immediately.
- Pings after settle without `settle_undo` are ignored.

Credit only host sessions.

### 5.3 Drift function

Traits move **only upward**, via additive server deltas.

Let `P` = presence-minutes in this tick (union).  
Let `L_b` = listen-in minutes on bird `b`.  
Let `O_near_b` = 1 if an offer was placed near bird `b`.  
Let `O_accept_b` = 1 if the offer reaction was `approach` or `join` or `drink`/`bathe`.

Per bird, per tick:

```
raw_boldness      += 0.00040 * P + 0.0040 * O_near_b
raw_warmth        += 0.00035 * P + 0.0100 * L_b
raw_vocal         += 0.00030 * P + 0.0080 * L_b
raw_saturation    += 0.00025 * P
raw_curiosity     += 0.00020 * P + 0.0120 * O_accept_b
```

Then apply a low-pass and caps:

```
delta = clamp(raw, 0, SESSION_CAP)
SESSION_CAP = 0.008 per calendar day per trait
  (accumulate unused cap? No. Unused is discarded. Prevents hoarding.)
trait = min(1.0, trait + delta)
```

**Calibration targets (test fixtures, not product analytics):**

- Fixture “regular week”: 5 days × 8 min presence, one 90s listen-in every other day, two accepted offers in the week. Expect **≥ 0.025 and ≤ 0.060** movement on at least two traits after 7 days.
- Same fixture × 3: **≥ 0.10 and ≤ 0.22** after 21 days on the most-moved trait.
- Fixture “open tab overnight, no activity”: 8h visible unfocused or no pointer — **0.000** presence credit, **0.000** drift.
- Fixture “neglect 14 days after a warm month”: traits **unchanged**; `gate` falls toward 0.15; greeting rate drops.

Settle contributes **zero** trait delta. It only closes presence and nudges mood toward `drowsy`.

### 5.4 Mood transitions

Mood is a small Markov step each tick, scored — not random-walk cosmetics.

Attractors:

| Local hour | Base attractor |
|---|---|
| 05–09 | `alert` |
| 09–16 | `content` |
| 16–20 | `curious` or `content` |
| 20–23 | `drowsy` |
| 23–05 | `drowsy` (nightjar species: `alert` or `curious`) |

Modifiers (additive scores, then softmax over the five moods with temperature 0.7):

- Offer accepted this window: `content +2`, `curious +1`
- Listen-in this window: `curious +1`, `alert +0.5`
- Rain: `vocal` behavior dampened; mood `content −1`, `wary +0.5`, `drowsy +0.5`
- Wind: `alert +1`; low-boldness birds `wary +1`
- Nearby bird entered `wary`: this bird `wary +0.8` (spread)
- High boldness (>0.6): `wary −1.5`
- High curiosity: `curious +0.8`
- Gate < 0.35: shift weight from `alert`/`curious` toward `content`/`drowsy` (quiet, not scared)

Mood persists across sessions. Never snap to `content` on snapshot `reason=start`.

Alarm-call: if any bird is `wary` and its call grammar emits an `alarm` motif (rare), other birds get a 3-minute `wary` bias in the snapshot’s ephemeral field.

### 5.5 Perch intent

Zones: front / middle / back, 3 slots each (enough for 7 birds without overlap). Server assigns unique `(zone, slot)`.

Weights:

- Boldness + `alert`/`curious` → front
- `wary` or low boldness → back
- Social warmth + another bird already in a zone → slight attraction to that zone (perch near others)
- `drowsy` → middle or back, low slot (visually “sits low”)

A bird commits to a new perch at most once every 4–12 minutes (drawn from vocal/curiosity). Snapshot includes `next_perch.eta` so the client interpolates a path. Teleporting is a bug.

User cannot place birds.

### 5.6 Weather

Each tick, if `weather.kind == clear` and a weekly budget remains:

- Rain: probability such that **2–4 rains per week**, duration 8–18 minutes.
- Wind: **1–3 per week**, duration 4–10 minutes.

Never thunder, never snow, never modal. Weather is in the snapshot; client paints softly. Mood modifiers apply only while `now < ends_at` plus 10 minutes after.

Weather RNG is seeded by `aviary_id + date` so catch-up is deterministic.

### 5.7 Call-grammar runtime (server side)

Server does not synthesize audio. It schedules **windows** and chooses a **motif plan**:

```
MotifPlan = {
  bird_id,
  start_at,
  motif_id,          // from species library
  variation: { tempo, pitch_offset, repeat, pause_ms }
  caption: string    // compiled now, matches what the client will play
  role: 'solo' | 'reply' | 'chorus' | 'alarm' | 'greet'
}
```

Timing: base inter-call interval `lerp(45s, 12s, vocal_frequency)` × `lerp(1.6, 1.0, gate)` × mood factor (`drowsy` 1.8, `alert` 0.75, rain 1.5).

Chorus: if two birds’ windows overlap by < 1.2s and both vocal_frequency > 0.4, the later plan is marked `reply` or `chorus` and pitch-offset is detuned 20–40 cents so the client chorus does not phase-lock.

Client must play the plan it was given. If a plan’s `start_at` is already 2s past on arrival, skip rather than dump a backlog (except `greet`, which may start immediately).

### 5.8 Return-greeting

Computed on snapshot `start` / `visible` / `wakeup`, not on poll.

```
absence = now - last_host_presence_at
candidates = birds sorted by
  score = 1.4*boldness + 1.1*social_warmth
        + mood_bonus(alert/curious +0.3, drowsy -0.4, wary -0.5)
        + gate
primary = top score
secondary = next if score within 0.15 and absence > 12h
```

Plan:

- One bird only for `absence < 12h`.
- If two, stagger `secondary` by 400–1200ms (never unison).
- Form from D16 bands, then vary with `hash(bird_id, date, session_id)` so the same bird is stylistically consistent but never byte-identical.
- Persist `greeting.issued_at` so a poll 10s later does not issue a second greeting.
- A second device opening during an active host session does **not** greet again (`last_host_presence_at` within 2 minutes).

Client: play the plan. No toast. No “you’ve been gone.”

### 5.9 Offer reactions (fast path)

`POST /apply` with `offer`:

1. Append event.
2. Identify `near_bird_id` (client hint) and validate distance against last known perch; if mismatch, pick the front-most bird as receiver.
3. If `now - last_offer_at[kind] < 4 min` for that bird: reaction `ignore` (no drift later either — tick sees cooldown and skips `O_*`).
4. Else score:

```
approach if curiosity*gate + mood(curious/content) > 0.55
wait     if wary or low boldness but curiosity > 0.35  (approach after 6–12s)
ignore   if drowsy and curiosity < 0.5
join     if kind==song and vocal_frequency high
quiet    if kind==song and wary
drink/bathe/watch if kind==pool, split by curiosity/boldness
```

5. Write ephemeral `offer_reaction` onto snapshot (ttl ~20s). Nudge mood scores in memory; persist mood if it actually flips.
6. Do not touch traits here.

### 5.10 Adoption

On account create: pick two species.

Rule: different `silhouette_id`, different `motif_library_id`, not two nightjars. Seed complementary traits (bird A higher boldness/warmth, bird B higher curiosity/vocal, both inside [0.22, 0.42]). Suggested names from a species name list; user may replace before first paint.

Age offers: when `now >= created_at + pacing[next_count]` and count < 7, set `bird_offer_pending`. Surface: a quiet notebook-adjacent line in the top-bar offer menu — naturalist, not a reward chest. Example: “a third bird has found the aviary.” No confetti. User accepts via `POST /adopt`; system picks species not already in the aviary if possible. If all six species are present, a duplicate species is allowed (identity is the bird id, not the species).

### 5.11 Notebook compiler

Inputs are **facts**, never user-behavior tallies:

Allowed facts: which bird greeted first today, perch zone, mood-visible behavior (fluffed, scanning, preening), weather, long quiet (no calls for N minutes), first greeter swap vs previous days this week, a new bird’s first morning, rain while a named bird stayed on the front perch.

Forbidden facts: session length, visit streak, click counts, trait deltas, “you,” exclamation, achievement framing.

Pipeline:

1. Tick proposes at most one `ObservationFact`.
2. Token bucket (§3.6) accepts/rejects.
3. `packages/prose/notebook.compile(fact, weekday, names)` selects a template family and fills with lowercase present-tense clauses.
4. Dedup: do not emit the same `trigger + bird_id` two entries in a row.

Examples of compiled output (illustrative, not canned forever):

- “tuesday — pip greeted before wren today, first time this week.”
- “wren is fluffed against the cool air, watching the back perch. low calls only.”

Store prose only. Do not store the fact graph on a user-visible surface.

---

## 6. Sync model

### 6.1 Canonicality

One aviary row, one snapshot row, N birds. Every host device is a projector.

```
device A  --events-->  log  --tick-->  snapshot  --GET-->  device A
device B  --events-->    \__________________________/---->  device B
```

There is no CRDT, no LWW document, no client personality cache that survives a refresh as authority.

### 6.2 What clients may cache

- Species SVG/silhouette modules (hashed assets).
- Phrase banks and grammar graphs (immutable).
- Last snapshot in `sessionStorage` **only** to paint a quiet field faster, discarded if `updated_at` is older than the network snapshot. Never used as a write base.

### 6.3 Conflict prevention

| State | Writer | Conflict rule |
|---|---|---|
| Personality | sim tick only | Additive deltas from ordered events. Unreachable LWW |
| Mood / perch / weather | sim tick | Last tick wins by definition |
| Expressiveness gate | sim tick | Same |
| Bird name | api on `PATCH` | Last write wins — names are not drift |
| Timezone | api | Last write wins |
| a11y settings | api | Last write wins |
| Presence | events | Union |
| Notebook | sim | Append-only |
| Settle ephemeral | apply-event | Last apply |

If two devices offer in the same 4-minute window, both events land; tick/apply enforces cooldown so the second is `ignore`. No user-visible conflict modal for this.

The only user-facing “sync” errors are auth/timeout/load failures (§4.4). Do not build a merge UI.

### 6.4 Clock and sleep

- Server time in every snapshot.
- Client interpolator uses `performance.now()` mapped to `server_time` at hydrate.
- If `document.timeline` jumps > 30s, treat as `wakeup`: pause, refetch, rehydrate. Do not fast-forward local idle by 30s of skipped frames (that reads as a skip-cut).

### 6.5 Deletion and export vs sync

Soft-delete: sessions still authenticate but snapshot returns a matter-of-fact “This account is scheduled for deletion” page with Restore. Tick still runs (so restore is coherent) but events other than restore are rejected.

Hard-delete job (daily): wipe `events`, `birds`, `snapshots`, `notebook_entries`, `visit_*`, `sessions`, `accounts`, mail suppressions. No cold analytics copy exists to forget.

Export job: JSON `{ account, aviary, birds: [{ id, name, species_id, traits, mood, adopted_at }], notebook, settings }`. Emailed as a signed link. This is the sole place traits are serialized to the user.

---

## 7. Frontend rendering pipeline

### 7.1 Boot sequence (central conceit)

1. Edge returns HTML with inline quiet-field background (soft sky color for local hour if we have a timezone cookie; else neutral dawn).
2. No spinner, no logo splash, no fade-from-black.
3. If a bootstrap snapshot was inlined (session cookie valid, edge cache of last snapshot, max-age 10s, private), hydrate immediately.
4. Else show the quiet field (already on screen) and fetch `/snapshot?reason=start`.
5. Place birds at `perch` with `motion_phase` already advanced. Start RAF. Start audio after a user-gesture **or** silently wait; never block first bird on audio.
6. Play greeting plan when the first frame is up.

Empty aviary happens once: post-consume, two names submitted, quiet field, then each starter **soft fly-in** to its perch. After `adopted_at` is set and first snapshot with birds exists, never show empty again — even on errors, stay on last-good or quiet field, not an empty perch set.

Time-to-first-bird is measured as first canvas pixel of a bird silhouette. Budget 500ms on mid-tier 4G. That requires:

- Critical JS for scene + species silhouettes of the two starters in the initial bundle.
- Snapshot < ~8KB gzipped.
- No webfont blocking; system UI font for chrome, one small face for captions loaded async.
- Settings, notebook panel, visit, account code-split.

### 7.2 Scene layout

One horizontal world, no pan, no zoom, no scroll of the scene.

- World units: x ∈ [0, 100], z ∈ {bg, mid, fg}.
- Perch anchors (responsive): front z-mid y-low, middle y-mid, back y-high/smaller scale.
- Viewport: compute uniform scale so all occupied perches plus one empty slot remain on-screen. Narrow phones compress x-spacing; never crop a bird.
- Parallax: bg 0.15, fg 0.08 of pointer-or-time drift — barely there.

Palette: calm naturalist (soft blues, greens, warm browns, muted ochres). Day/night is a continuous LUT driven by local hour. Settle forces a 4s ease into evening LUT regardless of clock; undo eases back.

Ambient ornaments (client-only, not in sim): leaf/feather spawn every 8–20s, 1–2 on screen, object-pooled. Disabled in reduced-motion and when hidden.

### 7.3 Idle micro-motion

Never paused while visible. Each `motion_kind` is a looped procedural clip:

- `preen` — content
- `scan` — wary (higher amplitude, back zone)
- `tilt` — curious, also triggered when another bird calls
- `shuffle` — weight reset, all moods
- `rest` — drowsy, low body, fluffed scale
- `call` — beak/throat overlay timed to audio

Clips are parameterized by `visual_seed` so two wrens do not clone. Loop lengths 3–7s, desynchronized.

**First frame mid-action:** `pose = clip.sample(motion_phase)` — not `clip.sample(0)`.

### 7.4 Transitions

- Perch change: cubic path, 1.8–3.2s, wing-settle at end. Reduced-motion: 1.2s cross-fade between still poses at old/new anchors.
- Offer objects: seed / pool / “song in the air” as soft scene props with 8–12s life.
- Weather: rain as low-contrast streaks, wind as leaf-rate increase + perch sway < 2px.

No page-level route transitions inside the aviary.

### 7.5 Top bar

Icons only: account, accessibility, notebook, offer. Settle lives in the offer/session cluster as a fourth glyph — still chrome, still not in-scene.

Fade: after 3s without pointer/keyboard, opacity → 0.15. Any pointermove/keydown → 1.0 in 180ms. Keyboard focus on a bar control forces opacity 1.

No badges, no unread dots on notebook or visits.

### 7.6 Reduced-motion renderer

Trigger: `prefers-reduced-motion: reduce` OR settings override on.

- Replace looping clips with 2–3 still poses per mood; cross-fade 800–1600ms.
- Flight → cross-fade.
- No leaf/feather spawn.
- Sun LUT still shifts, 2× slower.
- Audio and captions unchanged.
- Greeting still happens (pose change + call), not skipped.

This is a designed aesthetic, tested with screenshots, not a boolean `animate: none`.

### 7.7 Frame budget

RAF path:

1. Advance interpolators (pooled).
2. Draw bg, weather, perches, birds, fg.
3. No `getImageData`, no per-frame `new Path2D` if avoidable — cache species paths.
4. Captions/focus are DOM, not canvas, so screen readers and contrast stay real.

Target: < 8ms on a 5-year-old Intel UHD laptop at 1280×800, 7 birds, rain off. If `rAF` dt > 24ms for 30 consecutive frames, drop ornaments then rain density. Never drop birds.

Hidden tab: no RAF.

### 7.8 Memory

- Audio buffer pool fixed size (§8).
- Notebook virtualized list; detached entries released.
- No unbounded `addEventListener` on birds.
- CI: Playwright 30-minute soak, heap sampled every 60s, fail if `usedJSHeapSize` slope > 0 after first 3 minutes (linear regression, p<0.05, slope > 50KB/min).

---

## 8. Audio pipeline

### 8.1 Why procedural, how

Each species ships a motif graph in `packages/call-grammar`:

- 4–7 motifs (short, rise, trill, alarm, greet, night).
- Each motif: series of grains `{ f0, dur, amp, formant, noise }` not PCM.
- Bird identity = `grammar_seed` mapping to interval set, base f0, vibrato rate.
- Mood maps: drowsy (−3 semitones, ×1.25 dur), alert (+1–2, sharper attack), wary (fewer notes, more noise).

Client `SynthVoice` renders grains into a reused `AudioBuffer` via a worklet (`call-processor`). Two birds calling = two voices into a stereo merger with slight L/R from perch x, plus a shared short convolution (space) and a gentle compressor.

**Recognizability test:** ABX with staff — same bird across moods ≥ 80% identification at 7 birds mixed. If we fail, we do not raise the cap; we retune motifs.

### 8.2 Chorus mix and listen-in

Master mix is “place,” not a DAW.

Listen-in:

- Engage/disengage ramps **2.0s** equal-power.
- Focused bird gain → 1.0 (from ~0.55).
- Others → 0.18 floor (never 0).
- Disengage: click same bird, click empty, focus another, Escape, blur-from-scene.

Chorus: do not start two identical motif_ids at the same sample. Server already detunes; client also delays replies by 80–160ms.

### 8.3 WebAudio fallback

If `AudioContext` missing, `createOfflineContext` fails, or `resume()` denied after a gesture:

- Stay silent.
- Force captions on for the session (settings remain user-owned; show a matter-of-fact one-line in a11y settings: “Calls are captioned because audio isn't available.”).
- **No MP3/OGG sprites. No silent-fail without captions.**

Do not download recorded calls “just for Safari 14.” Unsupported browsers get the unsupported surface instead.

### 8.4 Allocation rules

- Max 8 voices (7 birds + 1 song-offer).
- Buffer pool: 16 buffers × 1.5s × stereo f32 ≈ 0.75MB, reused.
- Worklet never `new Float32Array` in `process()`.
- Suspend context on hidden; resume on visible after snapshot.

### 8.5 Song-fragment offer

The six fragments are themselves motif plans (not files). Playing one is an aviary-wide soft motif through a dedicated voice at low amp. Birds may `join` / `quiet` / `call against` per §5.9.

---

## 9. Accessibility surfaces

Accessibility ships on the same day as the visual aviary. No “v1.1 for reduced motion.”

### 9.1 Screen-reader narration

- A single `aria-live="polite"` region, visually hidden, updated from `snapshot.narration`.
- Idle cadence 45s (server issues a new sentence at 30–60s).
- Event priority: greeting, offer reaction, settle — server sets `priority: 'event'` and the client replaces the live text immediately.
- Voice: lowercase, present-tense, specific. “a warbler perches on the high branch, calling softly.” Never “warbler perched at high branch.” Never trait numbers. Never “mood: content.”
- Do not mirror every perch micro-shift. Narration is weather-to-climate of the scene, matching visual slowness.
- Notebook, settings, auth are normal documents with headings. Settings/auth: matter-of-fact.

### 9.2 Keyboard

| Key | Action |
|---|---|
| Tab | Top bar icons left→right, then first bird |
| Shift+Tab | Reverse |
| ←/→ | Previous/next bird (spatial by x) |
| Enter / Space | Listen-in on focused bird |
| Escape | Exit listen-in; if offer sheet open, close it; if settle undoing, do not auto-undo |
| Letter shortcuts | None that announce themselves on load |

Focus ring: 2px soft outline, contrast ≥ 3:1 against both noon and night palettes (designer token; implement both). Birds are in the tab order only after the bar, not every perch slot.

Offer sheet and settle confirm are full keyboard dialogs (focus trap, Escape closes). Settle still has the 5s click-anywhere undo — keyboard: any key except Tab also undoes during those 5s, documented in the settle control’s accessible name.

### 9.3 Captions

- Opt-in via a11y settings (and auto-on in audio fallback).
- Text comes from the same `MotifPlan.caption` the synth used.
- Position: DOM label anchored to the bird’s canvas bbox, fade with call envelope.
- Examples of compiler output: “a soft three-note rise”; “a low trill, paused, low trill again.”
- Contrast AA. Never cover the top bar.

### 9.4 Contrast and chrome

All user copy (bar, sheets, settings, errors, captions, notebook) ≥ WCAG AA. Scene itself has no copy. Test noon, dusk, night, rain.

### 9.5 What we refuse in the name of a11y

- Exposing personality vectors as ARIA stats.
- A static screenshot mode billed as the accessible product.
- High-frequency live-region spam.
- Naturalist voice on auth failures.

---

## 10. Performance budgets and observability

### 10.1 Budgets

| Metric | Budget | Gate |
|---|---|---|
| Initial JS gzipped (critical path) | < 2MB | CI bundle analyzer, fail build |
| Time to first bird | < 500ms p75 mid-tier 4G synthetic | Synthetic fleet |
| Idle FPS | 60 on 5yo laptop, 30-min session | Lab + nightly |
| Heap slope | ~0 after warmup, 30 min | CI soak |
| Snapshot size | < 16KB uncompressed | Unit test on fixture of 7 birds |
| Tick p99 | < 5s (alarm); target p50 < 20ms | Sim metrics |
| Magic-link send | p99 < 3s | Mail |

### 10.2 What we measure

Aggregate only:

- Navigation timings, TTFB, time-to-first-bird (custom mark `bird_visible`).
- rAF long-task counts, FPS histogram.
- AudioContext resume errors (code, not user).
- API latency/error rate by route.
- Tick duration, event backlog depth, catch-up step counts.
- Session-duration histogram **without account id**.

### 10.3 What we deliberately do not measure

- Per-bird offers, listen-in, presence minutes, names, species popularity as a product funnel.
- Visit frequency per account, DAU-of-aviary as an engagement OKR.
- “Average boldness in the population.”
- Funnel from greeting → offer. That would optimize announcement.

Staff calibration of drift uses **fixture accounts** and the unit/integration tests in §5.3, not a warehouse rollup of real users.

### 10.4 Synthetics

A small fleet (3 geos × Chromium/WebKit/Gecko) loads a fixture aviary every 10 minutes, asserts first bird < 500ms, plays 60s, records FPS and audio errors. Fixture account is flagged `is_synthetic` and excluded from any future analysis by existing in a separate sim shard if possible.

### 10.5 Browser support

Last two major versions of Chrome, Safari, Firefox, Edge. Feature-detect Canvas2D, ES2022, `visibilityState`. Fail closed to the matter-of-fact unsupported page. No polyfill soup that blows the 2MB cap.

---

## 11. Frontend module map (so the team can staff it)

```
apps/web/src/
  boot/quietField.ts          # first paint, no spinner
  scene/graph.ts
  scene/interpolate.ts
  scene/idle.ts
  scene/renderCanvas.ts
  scene/renderReduced.ts
  scene/ornaments.ts
  scene/palette.ts
  audio/context.ts
  audio/synthWorklet.ts
  audio/mix.ts
  presence/monitor.ts         # 3-signal conjunction, 4-min window, 30s pings
  input/pointer.ts
  input/keyboard.ts
  chrome/topBar.ts            # fade, icons only
  chrome/offerSheet.ts
  chrome/notebook.ts
  chrome/settle.ts
  a11y/liveRegion.ts
  a11y/captions.ts
  net/snapshot.ts
  net/events.ts
  visit/readonly.ts
  account/*                   # code-split
```

`presence/monitor.ts` is load-bearing. Implementation checklist:

- Subscribe to `visibilitychange`, `focus`/`blur`, `pointermove`, `keydown`.
- `activity_at` updates on pointer/key.
- Presence true iff `visibilityState==='visible' && document.hasFocus() && now-activity_at < 4min`.
- Do not treat `pointermove` on a background window as activity (blurred ⇒ false anyway).
- On `pagehide`/`beforeunload`, send `session_end` via `fetch keepalive` or `sendBeacon` to `/events`.
- Hidden ⇒ stop RAF + suspend audio, **do not** keep pinging.

---

## 12. Security, privacy ops, mail

- Magic links: 15 minutes, single use, hashed at rest, rate-limited.
- Session tokens: 30-day idle sliding expiry, revoke list in Redis checked on each snapshot.
- Invite tokens: 30-day unused expiry, hashed, revoke immediate.
- Export tokens: 1 hour, single use.
- Email templates: matter-of-fact. Subject “Your sign-in link”, not “your birds miss you.”
- No marketing list. No third-party analytics SDK. No session replay. No heatmap.
- CSP: default-src self; connect-src api; no arbitrary script.
- Visit log shows visitor email to the host only, decrypted in the api process.

---

## 13. Testing strategy

### 13.1 Engine (highest leverage)

- Golden fixtures for drift week-1 / week-3 / neglect / overnight-tab.
- Presence reconstructor: table-driven gaps, dual-device union, settle close, missing activity signal.
- Mood: dusk → drowsy; rain dampens call windows; wary spreads; no snap on start.
- Greeting: one bird; stagger; no second greeting on poll; no greeting for visitor.
- Cooldown and monotonic traits (property test: 10k random event traces, all traits nondecreasing).
- Notebook: forbidden-phrase linter (`you`, `achievement`, `streak`, `visited every`, `vocal frequency`).
- Tick determinism: same events + same clock → same snapshot hash.

### 13.2 Client

- First-frame test: `motion_phase ≠ 0` pose differs from rest pose.
- No toast / “welcome” string in product routes (eslint + snapshot of chrome).
- Keyboard path e2e.
- Reduced-motion screenshot diff (cross-fades present, ornaments absent).
- Listen-in gain floor > 0 on non-focused voices.
- Presence monitor unit tests against a fake document.

### 13.3 Sync

- Two simulated devices interleave offers and listen-ins; assert single personality trajectory.
- Device B never sends traits; contract test on event schema.
- Laptop sleep: wakeup refetch, no teleport.

### 13.4 Perf / a11y

- Bundle budget in CI.
- 30-min heap soak.
- axe on settings/auth/notebook; custom checks on live-region rate.
- Contrast tokens tested against noon/night LUTs.

---

## 14. Rollout

### 14.1 Build sequence (staffing order)

1. **Foundations (week 1–2):** account UUID model, magic link, schema, privacy split, empty quiet-field page.
2. **Snapshot + canvas (week 2–4):** two hardcoded fixture birds, mid-action boot, day/night LUT, top bar fade. No engine yet — fixture snapshot from disk — but the boot conceit must be true before anything else.
3. **Tick + presence (week 4–6):** real events, drift fixtures, expressiveness gate, mood, perch intent.
4. **Greeting + interactions (week 6–8):** listen-in mix, offers, settle+undo, rename.
5. **Audio + captions (week 6–9, parallel):** worklet, chorus, fallback silence.
6. **Prose surfaces (week 8–10):** notebook compiler, narration, offer copy. Voice review is a launch gate.
7. **A11y renderer + keyboard (week 8–10):** reduced-motion as a product, not a flag.
8. **Visits (week 10–11):** after host loop feels right. Default off.
9. **Export/delete, synthetics, soak (week 11–12).**
10. **Age-gate adopt (can ship disabled until pacing dates exist; logic live, first users won’t see bird 3 for 56 days).**

Do not open visit invites or bird 3+ until drift fixtures pass on staging.

### 14.2 Ramp

- Internal staff aviaries first (real presence, not synthetics only).
- Closed list (~50) for two weeks: calibrate the 4-minute activity window if watching-without-moving drops presence too often (may extend to 6 minutes; may not shorten below 3).
- Public v1: two birds, visits available but unprompted (no onboarding share step).
- Bird cap stays 7. Do not “ramp birds-per-aviary” as a growth lever. If audio ABX fails at 5, **lower** the cap rather than ship a blurred chorus.

### 14.3 Day-one instrumentation

Ship with: synthetics, RUM aggregates, tick p99 alarm at 5s, audio error counters, first-bird mark, heap soak in CI, magic-link fail rate.

Do not ship: engagement dashboards, streak-shaped funnels, population trait charts.

### 14.4 Launch checklist (product integrity)

- [ ] First frame is mid-action on cold and warm cache
- [ ] Zero welcome toasts/modals/banners on return
- [ ] Personality numbers unreachable in UI (grep + review)
- [ ] Presence ignores background tabs and unfocused windows
- [ ] Drift fixtures green
- [ ] Reduced-motion, captions, narration, keyboard all on
- [ ] WebAudio fail → silence + captions, no mp3
- [ ] Visit default off, no host ping
- [ ] Bundle < 2MB, first bird < 500ms synthetics
- [ ] Email never appears as an id in logs (scan)

---

## 15. Risks

### 15.1 Drift calibration (highest product risk)

Too fast → Tamagotchi numbers. Too slow → screensaver. Silent presence bugs look like “correct” code and ruin both.

**Mitigations:** fixture tests in CI; 4-minute conjunction hardcoded with comments; union-not-sum; daily trait cap 0.008; expressiveness gate instead of negative drift; no production dashboard that tempts accelerating the filter. If users say birds change overnight, **slow the filter**, do not add a UI explanation.

### 15.2 Sync correctness

Any client write of absolute traits, or LWW snapshot blob, will drop a morning session when a stale phone flushes. The failure is invisible.

**Mitigations:** event schema rejects trait keys; only `services/sim` has DB credentials that `UPDATE birds SET boldness`; contract tests; two-device interleaving in CI. Code review rule: “who writes this column?”

### 15.3 Audio uncanniness

Looped or phasey chorus breaks the spell permanently for that user.

**Mitigations:** no recorded fallback; per-call variation mandatory; detune + delay on replies; ABX recognizability; listen-in floors not mutes; if WebAudio is broken, captions rather than a tinny sprite.

### 15.4 Accessibility regressions

A late “we’ll label the canvas” patch would ship a different, worse product to SR and vestibular users.

**Mitigations:** reduced-motion renderer and narration are milestone 6–7, not post-launch; screenshot tests; live-region rate limiter; contrast on night palette; launch checklist blocks without them.

### 15.5 Load-state leakage

A spinner or fade-in on boot contradicts “already alive.” Slow snapshot on 4G is the likely cause.

**Mitigations:** quiet field is the only waiting surface; inline bootstrap snapshot when possible; 2MB/500ms gates; species of the two starters in the critical bundle.

### 15.6 Announcement creep

Toasts, visit badges, “you’ve been gone 4 days,” notebook entries about the user’s habits.

**Mitigations:** copy linter; no badge component in the design system; visit log demand-only; PR template checkbox “does this announce?”.

### 15.7 Privacy leakage via convenience

Email as a key, per-bird events in RUM, warehouse join “just for calibration.”

**Mitigations:** HMAC email hash, UUID everywhere, schema grants, CI ban on sim-table reads from analytics, no third-party replay.

### 15.8 Tick cost at scale

Naive “every aviary every 60s” dies.

**Mitigations:** activity-tiered cadence (D4), lazy catch-up, deterministic weather/mood functions of time, lease workers, p99 alarm.

### 15.9 Offer/fast-path vs tick races

Apply-event mood flip then tick overwrites from stale read.

**Mitigations:** same-aviary transaction lock for apply and tick; apply does not write traits; tick re-reads after lock.

### 15.10 Naming and identity accidents

A species art update that inserts new rows, or a rename that creates a new bird.

**Mitigations:** immutable `birds.id`; art updates are species-table only; rename is `UPDATE name`.

---

## 16. Team notes (execution hygiene)

- Vocabulary is normative: bird, call, listen-in, offer, settle, notebook, visit, tick, presence. Ban solo/select/chirp/pet/character in code names that surface in UI.
- Two voices, no arguments: product surfaces → naturalist; system surfaces → matter-of-fact.
- The user never owes the birds a visit. Absence is quiet, not failure.
- When torn between a richer feature and restraint, cut the feature.

This is sufficient to implement v1 without waiting on further product clarification. Build the quiet window, not a game around it.
