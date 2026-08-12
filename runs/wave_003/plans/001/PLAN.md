# Pocket Aviary — v1 implementation plan

This plan is an executable engineering document. It interprets the PRD into architecture, data, APIs, simulation math, client pipelines, accessibility, performance, rollout, and risks. It does not restate the product as a brief. Where the PRD is silent, this plan makes a locked call and marks it `DECISION`.

**Product:** Pocket Aviary — browser-only virtual aviary. One account, one aviary, two to seven birds. Server owns life; client renders and reports attention.

**Team assumption:** one product-engineering team can ship this in a single v1 train. Native apps, payments, multi-aviary accounts, and social-network surfaces are out of scope and must not leak into schema or protocol design.

---

## 0. Locked decisions (read first)

These are the ambiguities the PRD leaves open. Treat them as spec.

| ID | Decision |
|---|---|
| D1 | Personality traits are `float64` in `[0.15, 0.95]`. New birds seed uniformly in `[0.32, 0.52]` with species-keyed offsets of at most `±0.06`. Clamp after every tick. |
| D2 | Mood enum is exactly `{wary, content, curious, drowsy, alert}`. No additional moods in v1. |
| D3 | Simulation tick period `T = 60s`. The tick function is deterministic given `(state, consumed_events, wall_clock, timezone)`. Idle aviaries may be catch-up-ticked; catch-up must equal N live ticks. |
| D4 | Presence activity window `W = 240s` (4 minutes). Pointermove, pointerdown, keydown, or touchstart reset the window. Wheel-only does not. |
| D5 | Presence pings every `30s` while the three-way conjunction holds. A ping carries `seconds_since_last_valid` capped at `30`. Server records only the intersection of client-claimed seconds and elapsed wall time. |
| D6 | Offer cooldown is `180s` per bird, shared across offer kinds. Cooldown is server-authoritative. |
| D7 | New-bird offers are gated only by aviary age: 3rd at 56 days, 4th at 150, 5th at 270, 6th at 420, 7th at 600. Offer stays available until accepted or the aviary is at cap. No catalog; system picks species. |
| D8 | Species pool is six: `warbler`, `sparrow`, `wren`, `finch`, `dove`, `nightjar`. Nightjar is the night-active species. Starter pair is chosen by `hash(account_id)` from a complementary pairing table; user does not pick. |
| D9 | Notebook sparsity: at most 2 entries per rolling 7 days, typically 1 per 3–4 days of regular presence. Zero entries on a quiet week is correct. |
| D10 | Song-fragment library is 6 named motifs (`dusk-interval`, `two-note-ask`, `rain-soft`, `high-branch`, `still-water`, `late-hush`). Not user-uploadable. |
| D11 | Client stack: TypeScript monorepo, Vite, React 19 only for chrome (auth, settings, notebook, visit surfaces). Aviary scene is a custom Canvas 2D renderer with an optional OffscreenCanvas worker. No game engine. |
| D12 | Server stack: TypeScript on Node 22, Fastify, Postgres 16, Redis 7, transactional email provider. Shared Zod contracts in `packages/schema`. |
| D13 | Auth: magic link, 15-minute TTL, single-use. Session cookie `HttpOnly; Secure; SameSite=Lax; Path=/`, 30-day idle expiry, revocable per device. |
| D14 | Visit invite unused TTL is 30 days. Active visit sessions re-check revocation on every snapshot. Visitor snapshot endpoint never accepts events. |
| D15 | Drift weekly calibration target: ~20 minutes/day of honest presence for 7 days moves at least one trait by `≥ 0.015` and no trait by `> 0.05`. After 21 such days, at least one trait has moved `≥ 0.06`. Single-session trait delta is always `< 0.012`. |
| D16 | Timezone is IANA, stored on the account, refreshed from the client on snapshot if it changed. Day/night and mood-of-day use this timezone, never UTC wall clock as “local.” |
| D17 | First-paint strategy: inline a tiny “quiet field” CSS background in HTML. First bird draws from the snapshot embedded in the HTML when the user has a valid session and a warm edge cache; otherwise the quiet field holds until the first snapshot returns. No spinner. |
| D18 | Settle undo window is 5.0s client-side. If undo fires, client sends `settle_undo` referencing the settle event id. Tick treats a settle that was undone inside the window as if it never happened for mood, and still ends/restarts presence cleanly. |
| D19 | Listen-in mix: focused bird gain ramps to `1.0` over `1800ms`; others ramp to `0.22` over `1800ms`. Never `0`. Equal-power crossfade. |
| D20 | Catch-up vs live: accounts with a snapshot or event in the last 7 days are “hot” and receive live 60s ticks. Others are catch-up-only on next read, plus a daily sweeper that catch-up-ticks accounts idle 7–90 days so mood/day-night do not freeze for a returning user who expects continuity. Accounts idle >90 days catch up only on next access. |

---

## 1. Scope

### 1.1 In v1

- Browser client for last two major versions of Chrome, Safari, Firefox, Edge.
- Email magic-link accounts. One synthetic account UUID. One aviary per account.
- Two starter birds; age-gated adoption up to seven.
- Server-side simulation tick, personality persistence, mood persistence, bird-to-bird influence.
- Presence accounting (visible ∧ focused ∧ recent pointer/key).
- Listen-in, offer (seed / song fragment / still pool), settle + 5s undo.
- Field notebook, read-only, sparse naturalist entries, infinite scroll-back.
- Multi-device sync as a property of server-owned state (not a merge protocol).
- Visit invitations: email link, read-only ambient, revocable, off by default.
- Visit log in account settings; visit-notification toggle off by default.
- Account export (JSON emailed as download link). Soft-delete 30 days, then hard-delete.
- Accessibility: naturalist screen-reader narration, designed reduced-motion mode, call captions, WCAG AA chrome, full keyboard path.
- Performance budgets in §10. Operational telemetry only; no per-bird analytics warehouse.

### 1.2 Explicitly out of v1 (and do not design toward)

From `non_goals.md` and the product brief. These are implementation bans, not backlog:

- Native iOS/Android apps, app-store packages, push-notification certificates, deep-link schemes beyond ordinary HTTPS.
- Gamification of any flavor: achievements, streaks, levels, scores, badges, “birds adopted: N”, green-dot calendars, XP, ranks, tiers, milestone celebrations, visit-frequency surfaces, notebook lines about the user’s attendance.
- Tamagotchi mechanics: death, hunger, distress, decaying happiness, guilt-on-return, neglect-as-punishment, negative personality drift.
- Social network surfaces: profiles, follows, public discovery, feeds, comments, chat, avatars, co-presence, mutual-visit graphs, leaderboards, “show-off” visitor rendering.
- Payments, customizable scenes, multi-aviary accounts, shared/household aviaries, password/SSO (deferred, do not build adapters).
- Recorded-audio call libraries. Personality numbers on any user-visible surface, including “debug” in production.
- Notifications about the aviary (no email/push “your birds miss you”). Visit emails exist only as the invite link itself; visit-happened email only if the host opted in.

### 1.3 Voice split (engineering implication)

Two copy catalogs, two lint rules:

- `copy/naturalist/*` — aviary, notebook, narration, captions, offer prompts. lowercase, present tense, no “you,” no exclamation, bird-named, specific.
- `copy/system/*` — sign-in, session errors, sync errors, account settings, accessibility settings, unsupported browser, visit-revoked. Normal English capitalization, direct, no naturalist metaphor.

CI greps product-surface strings for banned tokens: `achievement`, `streak`, `welcome back`, `great to see you`, `level`, `badge`, `xp`, `days in a row`.

---

## 2. Architecture

### 2.1 Shape

A small three-plane system:

```
┌──────────────────────────────────────────────────────────┐
│  Browser                                                 │
│  chrome (React, code-split)                              │
│  scene loop (Canvas 2D, rAF)                             │
│  audio graph (AudioContext + AudioWorklet)               │
│  presence probe                                          │
│  snapshot interpolator                                   │
└─────────────┬──────────────────────────▲─────────────────┘
              │ events, presence pings   │ snapshots, HTML bootstrap
              ▼                          │
┌──────────────────────────────────────────────────────────┐
│  API (Fastify)                                           │
│  auth · accounts · aviary snapshot · events ingest       │
│  visits · notebook read · export · settings              │
└─────────────┬──────────────────────────▲─────────────────┘
              │                          │
              ▼                          │
┌──────────────┐  ┌──────────────┐  ┌──────────────────────┐
│ Event log    │  │ Sim worker   │  │ Canonical store      │
│ append-only  │─▶│ tick/catchup │─▶│ aviary, birds, moods │
│              │  │ notebook gen │  │ personality vectors  │
└──────────────┘  └──────────────┘  └──────────────────────┘
         Redis: session, rate limits, hot-account tick set, locks
         Email: magic links, visit invites, export links, optional visit notices
```

There is no client-to-client channel. There is no websocket required for v1 correctness. Snapshots are HTTP pull. A future SSE “tick available” hint is optional and must not become a second state channel.

### 2.2 Process topology

- **`api`**: stateless HTTP. Horizontal. Owns auth, ingest, snapshot assembly, visit authorization.
- **`sim-worker`**: consumes a Redis ZSET of due aviary ids (`next_tick_at`). For each due id, acquires `lock:tick:{aviary_id}` (25s TTL), runs one tick or a catch-up burst, writes state, advances `next_tick_at`. Concurrency = CPU-bound; start at 4 workers.
- **`sweeper`**: daily job. Marks 7–90 day idle aviaries for a single catch-up burst. Enqueues hard-deletes for accounts with `deleted_at < now()-30d`. Expires unused visit invites. Purges consumed magic links older than 24h.
- **`mailer`**: outbox table + worker. Never send mail inline in the request path except “accepted, queued.”
- **Postgres**: system of record. **Redis**: ephemeral. Simulation state is never Redis-only.

### 2.3 Client/server split (non-negotiable)

| Owned by server | Owned by client | Owned by neither (derived) |
|---|---|---|
| Account, sessions, email | Render clock, pose interpolation | Call audio waveform |
| Bird identity, species, name | Leaf/feather ornaments | Call captions from grammar |
| Personality vector | Quiet-field / first paint | Narration prose may be client- or server-built from snapshot (see §9) |
| Mood, perch intent, call timing intent | Local listen-in mix ramps | |
| Event log, presence seconds | Presence probe (sensors only) | |
| Notebook entries | Top-bar fade, focus rings | |
| Weather schedule, settle flag | Reduced-motion pose crossfades | |
| Visit invites + log | | |

Clients **never** send trait values, mood writes, perch commands, or notebook text. Clients send verbs. The tick interprets verbs.

### 2.4 Render-pipeline boundary

The scene renderer consumes a **render snapshot** — a pure, serializable view model produced by `packages/scene-state` from the network snapshot plus local interpolation time. React never mounts birds. React may mount:

- top bar icons and their panels
- auth pages
- account / accessibility / visit settings
- notebook sheet
- system error surfaces
- unsupported-browser surface

The canvas is a sibling, not a child of the notebook sheet. When a sheet is open, the scene keeps running (aviary continues). Sheets are chrome, not modes that pause life.

### 2.5 Privacy architecture (load-bearing)

Two databases or, if one cluster, two logical databases with separate credentials:

1. **`simdb`** — accounts (UUID PK), encrypted email column, birds, vectors, events, notebook, visits. No warehouse replica.
2. **`opsdb`** — request logs (no account email), metrics rollups, synthetic-check results.

Rules:

- Email exists in exactly one column: `simdb.accounts.email_ciphertext`. Decrypt only in auth and mailer.
- Telemetry events may carry `account_id` only for error traces retained ≤ 14 days, and never bird ids, trait values, offer kinds, or notebook text.
- No ETL from `simdb` to analytics. No “average boldness” dashboard. Not even internally.
- Visit log is host-private in `simdb`. Not aggregated.

### 2.6 Repo layout

```
apps/web/                  # Vite client
apps/api/                  # Fastify
apps/sim-worker/
apps/sweeper/
apps/mailer/
packages/schema/           # Zod + generated types + OpenAPI
packages/sim/              # pure tick, drift, mood, grammar schedule
packages/prose/            # notebook + narration + caption templates
packages/scene-state/      # snapshot → render model
packages/copy/             # naturalist vs system catalogs
infra/                     # migrate, terraform/helm, synthetic checks
```

`packages/sim` is isomorphic and side-effect free so the same functions run in worker, in catch-up, and in CI calibration harnesses. It must not import Fastify, Redis, or DOM.

---

## 3. Data model

### 3.1 Identifiers

- All public and internal entity ids are UUIDv7 (time-ordered, not email-derived).
- Bird `id` is stable for the life of the account. Rename, species-pool edits, and client migrations never mint a new bird id.
- Event ids are client-generated UUIDv7 plus server `seq` (bigserial per aviary). Dedup on `(aviary_id, client_event_id)`.

### 3.2 Tables (Postgres)

```
accounts
  id                    uuid pk
  email_ciphertext      bytea not null unique
  email_lookup_hash     bytea not null unique   -- hmac(email, pepper); for login lookup only
  timezone              text not null default 'Etc/UTC'
  created_at            timestamptz not null
  deleted_at            timestamptz null
  deletion_restore_until timestamptz null
  settings              jsonb not null default '{}'
      -- reduced_motion_override: null|true|false
      -- captions: boolean
      -- visit_notifications: boolean (default false)
      -- audio_enabled: boolean

sessions
  id                    uuid pk
  account_id            uuid not null references accounts
  token_hash            bytea not null unique
  device_label          text not null            -- parsed UA, matter-of-fact
  created_at            timestamptz not null
  last_seen_at          timestamptz not null
  revoked_at            timestamptz null

magic_links
  id                    uuid pk
  account_id            uuid not null
  token_hash            bytea not null unique
  expires_at            timestamptz not null
  consumed_at           timestamptz null

aviaries
  id                    uuid pk
  account_id            uuid not null unique references accounts
  created_at            timestamptz not null     -- age gate for adoption
  settled_until         timestamptz null         -- client settle; cleared on re-engage
  weather               jsonb not null           -- {kind, started_at, ends_at} | none
  lighting_override     text null                -- 'settled' only
  sim_rev               bigint not null default 0
  last_ticked_at        timestamptz not null
  next_tick_at          timestamptz not null
  last_presence_at      timestamptz null
  adoption_offers       jsonb not null default '[]'
      -- [{available_from, accepted_at, species}]

birds
  id                    uuid pk
  aviary_id             uuid not null references aviaries
  species               text not null            -- check constraint on pool
  name                  text not null
  adopted_at            timestamptz not null
  sort_index            smallint not null        -- 0..6, adoption order
  -- personality (canonical; only sim-worker writes)
  boldness              double precision not null
  social_warmth         double precision not null
  vocal_frequency       double precision not null
  plumage_saturation    double precision not null
  curiosity             double precision not null
  -- fast state
  mood                  text not null
  mood_since            timestamptz not null
  perch_zone            text not null            -- front|middle|back
  perch_slot            smallint not null        -- 0..n in zone, collision avoid
  pose_intent           text not null            -- preen|scan|tilt|shuffle|rest|sleep|approach|drink|bathe
  call_next_at          timestamptz not null
  last_offer_at         timestamptz null
  unique (aviary_id, sort_index)

interaction_events
  seq                   bigserial
  aviary_id             uuid not null
  client_event_id       uuid not null
  account_id            uuid not null            -- actor; visitors never insert
  bird_id               uuid null
  kind                  text not null
  payload               jsonb not null
  client_ts             timestamptz not null
  received_at           timestamptz not null
  consumed_at           timestamptz null
  primary key (aviary_id, seq)
  unique (aviary_id, client_event_id)

notebook_entries
  id                    uuid pk
  aviary_id             uuid not null
  observed_at           timestamptz not null
  body                  text not null            -- already-rendered naturalist prose
  stimulus              jsonb not null           -- internal; never exported as numbers-to-user
  unique (aviary_id, observed_at, body)

visit_invites
  id                    uuid pk
  aviary_id             uuid not null
  host_account_id       uuid not null
  guest_email_ciphertext bytea not null
  guest_email_hash      bytea not null
  token_hash            bytea not null unique
  created_at            timestamptz not null
  expires_at            timestamptz not null     -- created_at + 30d if unused
  consumed_at           timestamptz null
  revoked_at            timestamptz null

visit_sessions
  id                    uuid pk
  invite_id             uuid not null
  started_at            timestamptz not null
  last_snapshot_at      timestamptz not null
  ended_at              timestamptz null
  approx_duration_s     int not null default 0

mail_outbox
  id                    uuid pk
  kind                  text not null            -- magic_link|visit_invite|export|visit_notice
  account_id            uuid null
  payload               jsonb not null           -- recipient resolved at send time
  created_at            timestamptz not null
  sent_at               timestamptz null
  attempts              int not null default 0
```

Indexes:

- `interaction_events (aviary_id, consumed_at) where consumed_at is null`
- `aviaries (next_tick_at) where deleted_at is null` via join or denormalized `tick_eligible`
- `sessions (account_id) where revoked_at is null`
- `visit_invites (aviary_id) where revoked_at is null`

### 3.3 Event kinds

```
presence_ping
  { seconds: 1..30, visibility: 'visible', focused: true, activity_age_ms: number }

listen_in_start
  { bird_id }

listen_in_end
  { bird_id, duration_ms }

offer
  { kind: 'seed'|'song'|'pool', bird_id | null, song_id? }
  -- bird_id is the nearest/receiving bird at gesture time; server may reassign
  -- if that bird is on cooldown (see §5.6)

settle
  { }

settle_undo
  { settle_event_id }

reengage
  { }                     -- any click after settled; clears settled_until

rename_bird
  { bird_id, name }       -- not a drift input

accept_adoption
  { offer_index }         -- creates bird; not a drift input

visibility_hidden / session_end
  { reason: 'hidden'|'blur'|'unload'|'settle' }
```

`rename_bird` and `accept_adoption` and settings writes are **account commands**, not sim inputs. They can live in a `commands` table or in the same log with `consumed` immediately by API (not tick). Prefer a separate `account_commands` applied synchronously in the API transaction so rename is instant on next snapshot.

### 3.4 Snapshot document

Returned by `GET /v1/aviary` and optionally inlined into HTML:

```ts
type AviarySnapshot = {
  rev: number;                    // aviaries.sim_rev
  server_time: string;            // ISO
  timezone: string;
  lighting: {
    phase: 'dawn'|'morning'|'midday'|'evening'|'night';
    sun_01: number;               // 0..1 continuous
    settled: boolean;
  };
  weather: { kind: 'clear'|'rain'|'wind'; intensity: number; ends_at: string } | null;
  birds: Array<{
    id: string;
    species: Species;
    name: string;
    mood: Mood;
    perch: { zone: 'front'|'middle'|'back'; slot: number; x: number; y: number };
    pose: { intent: PoseIntent; phase_01: number };
    plumage: { saturation: number };          // visual only; not labeled
    call: { due_in_ms: number; motif_seed: number; last_caption_key: string | null };
    greeting: null | { style: GreetingStyle; stagger_ms: number };
  }>;
  adoption: { available: boolean; hint_species: null }; // never a catalog
  notebook_latest_id: string | null;          // for chrome, not a badge
  visit: null | { mode: 'host'|'visitor'; readonly: boolean };
};
```

`x,y` are normalized scene coordinates (`0..1` across the single horizontal frame) so the client can interpolate without knowing perch layout constants twice. Server is source of perch choice; client only eases.

**Never in snapshot:** raw trait names as a stats block, presence totals, visit counts, “days since adopted,” streak-like fields. `plumage.saturation` is a render parameter, not a labeled stat. Do not surface it in UI.

Export JSON **does** include personality vectors, because the PRD says the user may take a copy of their birds. Export is a settings action, matter-of-fact, not an in-aviary panel. Export is the only user-reachable place numbers appear, and it is a file, not a dashboard.

### 3.5 Settings JSON

```ts
type AccountSettings = {
  captions: boolean;                       // default false; true if WebAudio fallback
  reduced_motion_override: null | boolean; // null = follow prefers-reduced-motion
  visit_notifications: boolean;            // default false
  audio_enabled: boolean;                  // user mute; mute is an interaction signal (see §5.4)
};
```

Mute is user chrome, not a toast. Persistent per account so devices agree.

### 3.6 Soft delete

`accounts.deleted_at` set immediately. API sessions fail with matter-of-fact “This account is scheduled for deletion” plus a restore action. Sim-worker skips deleted aviaries. After 30 days, one transaction deletes all child rows, mail outbox payloads, visit invites, and the account. No backup warehouse retains per-bird rows.

---

## 4. API surface

Base: `https://{host}/v1`. JSON. Session cookie. CSRF: double-submit `X-CSRF` on mutating routes (cookie is Lax; magic-link landing is GET that upgrades to session then 303 to `/`).

All error bodies:

```json
{ "error": { "code": "magic_link_expired", "message": "We couldn't sign you in. The link may have expired. Try requesting a new link." } }
```

Messages come from `copy/system`. Never naturalist.

### 4.1 Auth

| Method | Path | Notes |
|---|---|---|
| `POST` | `/v1/auth/magic-link` | `{ email }`. Always 202 with the same body (`If an account exists or was created, we sent a link.`) to avoid account enumeration. Rate limit 5 / hour / email-hash and 20 / hour / IP. Creates account on first request (empty aviary + two birds minted server-side immediately; see §5.9). |
| `GET` | `/v1/auth/callback?token=` | Consumes link, sets session cookie, 303 `/`. Invalid/expired/used → HTML system surface. |
| `POST` | `/v1/auth/logout` | Revokes this session. |
| `GET` | `/v1/account` | Profile: email masked, devices, settings, deletion state. |
| `PATCH` | `/v1/account` | Settings, timezone (also auto-updated from snapshot). |
| `POST` | `/v1/account/email` | `{ new_email }` starts verify-to-commit; old email works until verify. |
| `GET` | `/v1/account/sessions` | Device list. |
| `DELETE` | `/v1/account/sessions/:id` | Revoke. |
| `POST` | `/v1/account/export` | Enqueue export mail. |
| `POST` | `/v1/account/delete` | Soft delete. |
| `POST` | `/v1/account/restore` | Only while in 30-day window. |

Magic-link email is system voice, short, one button. No “your birds miss you.”

### 4.2 Aviary state

| Method | Path | Notes |
|---|---|---|
| `GET` | `/v1/aviary` | Canonical snapshot. Triggers catch-up if `now - last_ticked_at > 90s`. Query `?since_rev=` allowed; if current, `304` with empty body. |
| `GET` | `/v1/aviary/bootstrap` | Same payload, used by SSR/edge include. Cache: `private, no-store`. |

Pull triggers (client):

1. First paint / navigation.
2. `visibilitychange` → `visible`.
3. `pageshow` after `persisted` (bfcache).
4. Render-frame gap `> 5000ms` (sleep/suspend).
5. Keepalive every `25s` while visible (cheap; `since_rev`).
6. After posting an event that should echo (offer, settle, rename, adopt) — snapshot once the POST returns `{rev}`.

### 4.3 Interaction events

`POST /v1/events`

```ts
{
  events: Array<{
    client_event_id: string;
    kind: EventKind;
    bird_id?: string;
    payload?: object;
    client_ts: string;
  }>;
}
```

- Max 20 events / request, 2 requests / second / session (presence + gestures fit).
- Idempotent on `client_event_id`.
- Response: `{ accepted: number, rev: number }`.
- **Visitors: 403.** No write path on the visit host.

Server validation:

- Unknown bird → drop event, do not 500.
- Offer during cooldown → accept event (`kind` stays) but payload flagged `ignored: cooldown`; tick will not restimulate. Client already hid the affordance; this is belt-and-suspenders.
- Presence ping without the three flags → reject that event only.

### 4.4 Notebook

| Method | Path | Notes |
|---|---|---|
| `GET` | `/v1/notebook?before=&limit=` | Reverse chronological. `limit` default 20, max 50. No search. No edit/delete. |

### 4.5 Adoption and naming

| Method | Path | Notes |
|---|---|---|
| `POST` | `/v1/birds/adopt` | Mints next bird if an age offer is open and count `< 7`. Body empty. Returns snapshot. |
| `PATCH` | `/v1/birds/:id` | `{ name }` 1–24 chars, trimmed, no emoji restriction required. Instant. |

Adoption presentation: not a catalog modal. Naturalist copy: “another bird has found the aviary.” Then name field with a suggested name. First-run is the same pattern for two birds already minted (user is naming arrivals, not shopping).

### 4.6 Visits

| Method | Path | Notes |
|---|---|---|
| `POST` | `/v1/visits` | Host: `{ email }`. Creates invite, mails one-time link. Default-off in the sense that nothing exists until this call. Rate limit 10 / day / account. |
| `GET` | `/v1/visits` | Host: outstanding invites + visit log (email, day, approx duration). Settings chrome, not aviary chrome. No badge count. |
| `DELETE` | `/v1/visits/:id` | Revoke. Immediate. |
| `GET` | `/v1/visit/:token` | Visitor HTML app shell. Sets a **visit cookie** (not an account session) scoped to that invite. |
| `GET` | `/v1/visit/:token/snapshot` | Read-only snapshot of host aviary. If revoked/expired: `410` with “This visit is no longer available.” |
| `POST` | `/v1/visit/:token/heartbeat` | Updates `last_snapshot_at` / duration. **Does not** write `interaction_events`. |

Visitor client: scene + audio + captions optional. No top-bar offer/settle/notebook-write. Notebook is host-private — **not shown** to visitors (PRD: visitor cannot notebook-scroll in a way that affects host; safest v1: hide notebook entirely). Account icon becomes “leave visit.”

Host is not notified unless `visit_notifications` is true, in which case one system-voice email per visit session after `approx_duration_s ≥ 60` (avoid bounce-open mail). Still no in-product toast, still no badge.

### 4.7 Accessibility settings

Reached from top-bar icon. Matter-of-fact labels. Endpoints are `PATCH /v1/account`. Reduced-motion override, captions, mute.

### 4.8 Versioning and compatibility

`/v1` only. Snapshot is additive. If a field must change meaning, add a new field; do not silently reinterpret `mood` strings.

---

## 5. Simulation engine

All of this lives in `packages/sim` as pure functions. Worker supplies `now`, event batch, and previous state.

### 5.1 Tick contract

```ts
function tick(input: {
  state: CanonicalState;
  events: readonly Event[];
  now: Date;                  // tick boundary, 60s grid aligned per aviary created_at offset
  rng: Mulberry32;            // seeded from hash(aviary_id, tick_index)
}): { state: CanonicalState; notebook: NotebookEntry[] }
```

- `tick_index = floor((now - aviary.created_at) / 60s)`.
- RNG is deterministic per tick so catch-up matches live.
- Events are applied in `seq` order, then ambient processes run once.

Catch-up:

```
while last_ticked_at + 60s <= now:
  batch = events where received_at in (last_ticked_at, last_ticked_at+60s]
  state = tick(...)
  last_ticked_at += 60s
```

Cap catch-up work per request at 200 ticks (~3.3 hours) and continue the rest in the worker so a year-idle return does not block first snapshot more than ~50ms of compute. For long gaps, use a **fast-forward** path: collapse empty ticks (no events, no weather start/end) into a closed-form mood-of-day + call_next_at refresh, still advancing `sim_rev` once at the end. Fast-forward must not apply drift (no presence in empty ticks). It must advance lighting and nightjar activity.

### 5.2 Personality vector and drift

Traits: `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`.

**Monotonic toward expressive.** Neglect never decrements a trait. “Ambient quietness” is a **behavior policy** over unchanged (or slowly raised) traits plus recency of presence, not a negative drift.

Define a per-tick **attention mass** from consumed events:

```
P  = sum(presence_ping.seconds) / 60          // 0..0.5 typical; 1.0 if somehow full minute
L_i = listen_in seconds on bird i / 60
O_near_i = 1 if an offer was placed with bird i as receiver this tick else 0
O_accept_i = 1 if bird i accepted (approach/join/drink) this tick else 0
```

Per-trait increments for bird `i`:

```
approach = (1 - (trait - 0.15) / (0.95 - 0.15))   // remaining headroom

Δboldness           = k_b * approach * (0.70 * P + 0.30 * O_near_i)
Δsocial_warmth      = k_s * approach * (0.45 * P + 0.55 * L_i)
Δvocal_frequency    = k_v * approach * (0.40 * P + 0.60 * L_i)
Δplumage_saturation = k_p * approach * (1.00 * P)
Δcuriosity          = k_c * approach * (0.35 * P + 0.65 * O_accept_i)
```

Calibrated constants (`DECISION`, lock in sim tests):

```
k_b = k_s = k_v = k_p = k_c = 0.0048   // per tick at full signal, before approach
```

Worked check: 20 minutes honest presence / day = 20 pings-worth ≈ 40 ticks with `P ≈ 0.5` and others 0.

`Δ ≈ 0.0048 * 1 * (0.70*0.5) = 0.00168` per such tick for boldness, times ~40 = `0.067` if `approach=1`. That is too fast. Presence is not `P=0.5` on every tick of a 20-minute session only — 20 minutes is 20 ticks, not 40.

20 minutes = 20 ticks, `P ≈ 1.0` if they truly watched the whole minute? Presence pings every 30s with 30s credit → two pings/tick → `P = 1.0`. Then `Δboldness = 0.0048 * 0.70 = 0.00336/tick * 20 = 0.067/day` — still too fast vs D15.

Set:

```
k_* = 0.00055
```

Then 20 ticks * `0.00055 * 0.70 * 1.0` = `0.0077/day`. Week ≈ `0.054` at `approach≈1`, but seeds are ~0.4 so `approach≈0.69` → week ≈ `0.037`. Three weeks ≈ `0.11` before approach shrinks. Single 5-minute session: 5 ticks * `0.00055 * 0.70` ≈ `0.0019`. Matches D15.

**Mute / no-audio:** sitting with audio muted still counts as presence (they are watching). Mute is not neglect. Listen-in with audio muted still counts as listen-in (attention focus). Drift must not require sound.

**Settle:** no trait delta. Ends presence window (client stops pings). Small mood nudge only (§5.3).

**Ignore / absence:** `P=0` for empty ticks. Traits hold. Greeting policy uses `hours_since_last_presence` to choose glance vs re-orientation, not to punish.

Clamp to `[0.15, 0.95]` after apply. Persist the five floats. Never recompute from logs.

### 5.3 Mood

Mood is a discrete state with a **score vector** under the hood for transition, but only the enum is stored.

Transition each tick:

1. Start from current mood (persisted; do not reset on session start).
2. Apply time-of-day prior in account timezone:

| Local hour | Prior |
|---|---|
| 05–08 | alert + |
| 09–15 | content + |
| 16–18 | curious / content |
| 19–21 | drowsy + |
| 22–04 | drowsy ++; nightjar: alert/curious instead |

3. Ambient: rain → vocal-dampen flag + shift toward drowsy/wary; wind → alert or wary (low boldness → wary, high → alert); another bird’s `alarm` call (rare, weather or startle) → nearby birds wary.
4. Interactions this tick: accepted offer → content/curious; listen-in on this bird → curious/content; settle → drowsy.
5. Personality gates: `boldness > 0.7` heavily down-weights wary; `curiosity > 0.7` up-weights curious; `vocal_frequency` does not force alert.

Hysteresis: stay in a mood at least 8 ticks (8 minutes) unless a high-priority event (offer accept, alarm, settle) fires. Prevents flicker.

Night: most species’ pose_intent → `sleep` when mood is drowsy and hour ≥ 22 or lighting phase is night. Nightjar never sleeps as the default night pose; it may `scan` and call.

### 5.4 Behavior policy (how traits + mood become motion)

Each tick the server picks:

- **Perch zone** via softmax over `{front, middle, back}`:

```
front  = boldness + (mood==alert|curious ? 0.15 : 0) - (mood==wary ? 0.35 : 0)
back   = (1-boldness) + (mood==wary ? 0.25 : 0) + (mood==drowsy ? 0.10 : 0)
middle = 0.55
```

Plus a **social term**: high `social_warmth` pulls a bird toward a zone already occupied. Low warmth prefers an empty zone. Resolve slot collisions by offsetting `perch_slot`.

- **Pose intent** from mood: wary→scan, content→preen, curious→tilt, drowsy→rest/sleep, alert→scan/shuffle.
- **Call schedule** `call_next_at`:

```
base_interval = lerp(18s, 90s, 1 - vocal_frequency)   // high VF → more often
mood_mult = wary 1.4, content 1.0, curious 0.85, drowsy 2.2, alert 0.9
weather_mult = rain 1.6 else 1.0
unobserved_mult = 1.0
next = now + base * mood_mult * weather_mult * rng(0.7, 1.3)
```

Chorus: if another bird called in the last 4s and this bird’s `social_warmth` and `vocal_frequency` are both `> 0.45`, with p = `0.25 * social_warmth`, schedule a reply in 400–1200ms.

Alarm: only weather startle or a rare RNG (`p=0.002` per tick, suppressed at night except wind). Not a user-facing “event.”

**Ambient quietness after long absence:** do not lower traits. Instead multiply greeting probability and unobserved call rate by:

```
quiet = clamp(hours_since_presence / 336, 0, 0.7)   // 2 weeks → 0.7
call_interval *= (1 + quiet)
greet_p *= (1 - quiet)
```

Birds still live and still call. They greet less often because less often is what has been observed. Returning presence gradually removes `quiet` as hours_since_presence shrinks — this is not negative drift; it is a recency window.

### 5.5 Return-greeting

Computed when assembling a snapshot if `last_client_visible_at` (from last presence or last host snapshot) is older than 8s (tab return) or this is a new session.

Pick **one** primary greeter:

```
score = 0.6 * boldness + 0.3 * social_warmth + 0.1 * (mood in {alert,curious,content})
- 0.4 if mood==drowsy
- 0.5 if mood==wary
```

Highest score greets. Others greet only if `score > 0.55` and `rng < 0.35`, staggered `+ 400–1800ms`. Never unison.

Style from absence length:

| Absence | Style |
|---|---|
| < 3 min | `glance` |
| 3–120 min | `two_note` or `head_tilt` |
| 2–48 h | `step_forward` or `longer_call` |
| > 48 h | `reorient` (step_forward + call; possible second-bird reply) |

Procedural variation: `motif_seed = hash(bird_id, tick_index, absence_bucket)` so the same bird rhymes with itself but is never byte-identical. Client maps style → motion + grammar. **No toast. No “you’ve been gone N days.”**

Visitor snapshots **do not** include greetings. Visitors are not noticed. Host opening the tab is noticed even if a visitor is also watching (they are not co-present).

### 5.6 Offers

Kinds:

- **seed** — placed at front-middle of scene. Receiver = highest `curiosity + boldness - wary_penalty` among birds not on cooldown. Reaction: curious+content → approach + peck (accept). Wary → wait 1–3 ticks then maybe approach if curiosity `> 0.4`. Drowsy → ignore (no accept, still `O_near` if they were closest).
- **song** — play motif id through a dedicated bus (client). Birds with high vocal_frequency + non-drowsy: `join` (call overlapping motif). Wary: go quiet one interval. Others: `call_against` or ignore.
- **pool** — reflective ellipse at front. Reactions: drink, bathe, or watch, weighted by species + curiosity. Dove/warbler bathe more; nightjar watches.

Cooldown: `last_offer_at` per bird, 180s, any kind. Offering during cooldown is a no-op at tick. This is functional, not a scolding UI.

Offers are triggered from top bar, never from clicking a bird (click is listen-in).

### 5.7 Weather

Server schedules weather on the aviary, not per client.

- Rain: 2–4 times per week, duration 8–18 minutes, intensity 0.3–0.6. Never thunderstorm.
- Wind: 1–3 times per week, 4–10 minutes, leaf-ripple flag.
- No overlap. No snow.

Schedule next event on tick using weekly quota and RNG. Weather is in the snapshot so all devices agree.

### 5.8 Bird-to-bird

Already: chorus replies, wary spread (if ≥2 birds wary, others get +wary prior), social perch pull. Do not add flocking paths or fight/play minigames.

### 5.9 Adoption mint

On account create, in the same transaction:

- Insert aviary.
- Draw starter pair from complementary table (e.g. `{wren,warbler}`, `{sparrow,finch}`, `{dove,wren}`, `{finch,warbler}`, `{sparrow,wren}`, `{dove,finch}`) via `hash(account_id) % n`. Never two nightjars as starters. Nightjar can appear as a later age offer.
- Seed traits (D1) with species offsets (nightjar: lower vocal daytime bias handled in mood, not as a hidden stat).
- Mood: `content` and `curious` split. Perches: one middle, one back or front depending on boldness.
- Suggested names from a species name list; user can edit immediately.

Age offers: when `now - aviary.created_at` crosses a threshold and `bird_count < 7`, push an offer record. Species = unused species first; if all used, allow duplicate species (recognizability still comes from call grammar seed per bird id, not species alone). **No rarity.**

### 5.10 Call-grammar runtime (server half)

Server does **not** synthesize audio. It schedules:

```
CallPlan { bird_id, when, motif_family, duration_hint_ms, intensity, reply_to }
```

`motif_family` is species+bird keyed. `intensity` from mood. Client grammar expands family+seed into notes (§8). Snapshot includes the next due call so a freshly opened tab can start mid-chorus without waiting a tick.

### 5.11 Notebook writer

Runs at end of tick if:

- no entry in the last 48h, and
- rolling 7-day count `< 2`, and
- a stimulus is “noteworthy” OR a 3–4 day silence timer elapsed with at least some presence in that window.

Noteworthy stimuli (internal, never shown as codes):

- first time bird A greeted before bird B in ≥ 5 days
- long preen (> 4 ticks) without scanning
- rain while a bird bathed
- a bird accepted a seed after a wary streak
- nightjar called after midnight
- a long quiet morning (low call count, presence > 10 min)

Prose: `packages/prose` templates with slot-fills (`name`, `perch`, `weekday lowercase`, species common noun). Present tense, lowercase, no “you,” no trait numbers, no “visited every day.”

Example generator output only if slots match; do not emit generic “session started.”

If nothing specific can be said, **write nothing**. Sparsity > filler.

---

## 6. Sync model

### 6.1 Canonicality

One row-set per aviary. Personality has a single writer: `sim-worker` (and the API only when minting a bird at seed values). Clients are projectors.

There is no CRDT, no LWW on vectors, no “sync personality” endpoint.

### 6.2 Why multi-device just works

Laptop and phone each hold a session cookie. Each pulls `/v1/aviary`. Each posts its own events into the same log. Tick applies both, in `received_at`/`seq` order. Two devices both sending presence for the same minute: **do not double-count**. Presence merge rule:

```
presence_seconds_tick = min(60, max(device_A_seconds, device_B_seconds))
```

Honest attention is not additive across devices (user is not two people). Listen-in: if both devices listen to different birds, both `L_i` may apply (unusual; accept it). If both listen to the same bird, take `max(duration)`.

### 6.3 Conflict surface that remains

| Case | Handling |
|---|---|
| Magic-link replay | `consumed_at` set; second click shows expired/used system copy. |
| Two tabs offer at once | Both events land; second bird-cooldown ignore. Fine. |
| Rename vs rename | Last `received_at` wins on `name` only (name is not drift). |
| Settle on phone, watch on laptop | Laptop `reengage` or presence ping clears `settled_until`. |
| Visitor + host | Separate cookies; visitor cannot write; host presence only. |
| Clock skew | Server `received_at` orders sim; `client_ts` is diagnostic only, clamped ±10 min. |
| Mid-write outage | Events POST is transactional; client retries with same `client_event_id`. Snapshot catch-up fills gaps. |

Personality LWW is unreachable because clients cannot write it.

### 6.4 Offline

v1 is online-required for the aviary view. If snapshot fails: quiet field + system line “Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch.” Queue **no** local personality. Optionally queue events in `sessionStorage` for up to 5 minutes and flush on return — yes, for presence/offer continuity during a blip — but drop them if older than 5 minutes so a laptop that wakes after a weekend does not dump a false presence burst.

### 6.5 Session revocation

Settings device list. Revoke sets `revoked_at`. Next API call 401s. Matter-of-fact: “Your session timed out. Sign in again to keep watching.” (also used for idle expiry).

---

## 7. Frontend rendering pipeline

### 7.1 Boot sequence (aliveness is the first frame)

1. HTML arrives with calm sky CSS (quiet field). No spinner, no logo splash, no “loading aviary.”
2. If bootstrap snapshot is inlined (warm session), JS parses it synchronously and the first `rAF` draws birds at `pose.phase_01` already mid-cycle. Audio context starts on first pointer/key (browser policy); until then, captions-if-enabled only.
3. If snapshot is not inlined, quiet field remains; fetch `/v1/aviary`; first draw is still mid-pose, not an entrance, **except** true first-run empty aviary (§7.7).
4. No fade-from-black. No “ready” event toasts.

### 7.2 Scene layout

One horizontal frame. No pan, zoom, or scroll. Three depth planes:

- Background: sky gradient (time-of-day), soft foliage silhouettes.
- Middle: perches + birds.
- Foreground: occasional branch/leaf, pool when offered.

Normalized coordinates. Viewport:

- Width drives inter-perch spacing.
- Height preserves a fixed scene aspect (`DECISION`: 16:9 logical, letterboxed with sky color, never crop a bird).
- Narrow phones: compress X spacing; all `x` remain in `0.06..0.94`.

Perch zones: back y-smaller (higher on screen, more atmospheric perspective), front y-larger, slight scale 0.85 / 1.0 / 1.18.

User cannot drag birds.

### 7.3 Bird visuals

- Species = silhouette + default palette. Plumage saturation from snapshot shifts chroma toward richer hues; never a number on screen.
- Draw as procedurally posed 2D rigs (5–8 bones: body, tail, head, two legs, optional wing). SVG path libraries compiled into JS, not large PNGs. Target `< 30KB` gzipped art tables for all six species.
- Idle micro-motion: continuous. Preen cycles, scan arcs, weight-shift, head tilt toward the last call’s x position. Periods 3–12s, personality-keyed (high curiosity → more tilts).
- Never hold a perfectly still pose for `> 400ms` except reduced-motion stills and sleep (sleep still has 0.3px breathe).

### 7.4 Interpolation

Network snapshot is sparse (~every 25s + events). Renderer:

```
display_pos = lerp(prev.perch, next.perch, easeInOut(alpha))
```

Perch changes from the sim take `2200–4000ms` to ease (walk/hop along a short cubic path). Do not teleport. If a snapshot arrives late, retarget mid-ease.

Pose cycles are client-time loops seeded by `bird_id` so two devices look similar but not lockstep-identical (acceptable; life is visual). Sleep/settle lighting is snapshot-authoritative.

### 7.5 Ambient ornaments

Leaves/feathers: client-only, Poisson interval ~12–25s, 1–2 sprites, no sim state. Disabled in reduced motion. Subtle parallax: background foliage shifts at 5% of a theoretical camera — almost none.

### 7.6 Day/night and settle

Sky and fill lights are functions of `lighting.sun_01` plus `settled`. Settle: 3.2s ease toward evening palette and lower call gain (`* 0.35`), regardless of actual hour. Undo in 5s eases back. After 5s, settle holds until tab close or reengage (any aviary click / offer / listen-in).

Night: dim, most eyes closed. Nightjar motif allowed.

### 7.7 Empty aviary

Only between “account created” and first bird posed — in practice the two starters already exist server-side, so the empty state is **first-run fly-in only**: quiet field, then each starter eases in along a short arc to its perch, staggered 600ms. After that, never empty, never fly-in on later sessions.

### 7.8 Top bar

Icons: account, accessibility, notebook, offer. Settle lives in the same bar (`DECISION`: a fifth icon, “settle,” because the PRD names it as a top-bar gesture and the bar is allowed this sparse set). No other chrome.

Fade: after 3.5s without pointer/key, opacity → 0.12 over 800ms. Activity restores 1.0 in 180ms. Keyboard focus on a bar control forces opacity 1.0.

No badges, no unread dots on notebook, no visit-count pip.

### 7.9 Color

Calm naturalist palette. Implementation consumes design-system tokens (separate doc). Engineering constraint: all chrome text ≥ WCAG AA against both midday and night bar backgrounds. Provide two token sets (`day`, `night`) and switch with lighting.

### 7.10 Reduced-motion mode

If `prefers-reduced-motion: reduce` XOR account override:

- Replace looping micro-motion with still poses and **slow crossfades** (1200–2000ms) when intent changes.
- Perch travel = crossfade, no hop path.
- No leaf/feather drift.
- Lighting shifts remain, duration `* 1.8`.
- Audio unchanged.
- First frame is still a mid-life still, not a blank load.

This is a designed second renderer path (`renderBirdReduced`), not `if (reduce) return`.

### 7.11 Frame loop and hidden tabs

```
visible && !document.hidden → rAF loop
else → cancel rAF, suspend canvas, hold AudioContext (see §8)
```

Simulation continues on server. On return: snapshot pull, resume mid-motion.

### 7.12 Bundle split

| Chunk | When |
|---|---|
| `boot` + `scene` + `audio-grammar` | initial |
| `notebook` | icon click |
| `settings` / `account` / `visits` | icon click |
| `auth` | signed-out routes |

Initial JS gzipped **< 2MB**, target **< 400KB** gzipped for boot+scene+audio (the 2MB cap is a ceiling, not a goal).

---

## 8. Audio pipeline

### 8.1 Why procedural

Looped files break recognizability-under-variation and chorus. Bundle budget forbids a rich recorded library. **No recorded-call fallback.**

### 8.2 Graph

```
AudioContext
  ├─ birdBus[i] ── gain_listen[i] ── chorusComp ── masterGain ── destination
  ├─ songOfferBus ── gain_offer ── masterGain
  └─ caption clock (offline from same grammar events)
```

Each bird: `AudioWorkletNode` or a small pool of `Oscillator`+`Biquad`+`Gain` voices allocated from a ring (no per-call `new Oscillator` leak). **Reuse buffers.** CI heap snapshot after 30 min must be flat (§10).

### 8.3 Grammar

Per species, a motif library of 4–6 atoms (interval patterns in cents, durations in ms, timbre params: filter cutoff, FM index, noisy burst mix).

Per bird, a **signature** derived from `bird_id` (stable): base pitch offset ±3 semitones, preferred atom order, gap rhythm. Mood shapes tempo and filter:

- wary: shorter, sharper attack
- content: mid, rounded
- curious: extra grace note
- drowsy: quieter, longer gaps
- alert: slightly brighter

Recognizability test (QA): listeners identify bird at `> 70%` in a 6-bird lineup after 10 minutes familiarization, across two moods. This test gates the seven-bird cap; if it fails at 6, fix grammar before shipping 7.

### 8.4 Chorus mixing

Calls overlap in time with independent phases. No shared loop clock. Light bus compression (2:1 above a high threshold) so two birds do not clip. Listen-in: D19 ramps. Others stay audible.

Master gain duck ~1.5dB during song-fragment offer, then recover 2s.

### 8.5 Listen-in UX vs audio

Focus bird (click/tap/Enter). Click again, click empty, focus another, or Escape: disengage. Mix ramps both ways. Focusing is not a selection box with a menu.

### 8.6 Mute and WebAudio fallback

- User mute: `masterGain = 0`, captions follow user setting (do not force on).
- WebAudio missing / `AudioContext` denied / create fails: **silence**, force captions **on** for the session (and persist captions true if they had no preference). System copy in settings: “Calls aren’t available in this browser. Captions are on.”
- Do not decode MP3s.

Autoplay: wait for first user gesture to `resume()`. Until then, visual-only + optional captions. Greeting call plays when context unlocks if greeting is still within 4s of session start; otherwise skip rather than fire a late canned welcome.

### 8.7 Settle and night

Global envelope toward quiet. Nightjar exempt from the harshest duck (nightjar `* 0.7` instead of `* 0.35` at night).

---

## 9. Accessibility surfaces

Accessibility ships on the same day as the visual aviary. Not a v1.1.

### 9.1 Screen-reader narration

A live region (`aria-live="polite"`) updated with naturalist prose.

Cadence:

- Idle: every 45s (`DECISION`, midpoint of 30–60).
- Immediate (still observational, not “state changed to X”): return-greeting, offer reaction, settle, adoption fly-in.

Generator: `packages/prose/narrate(snapshot, lastNarration)` — same voice as notebook. Mention two birds at most per utterance. No trait numbers, no perch indexes as “perch 2,” no “mood: content.”

Example: “a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.”

Implementation: visually hidden live region adjacent to canvas. Canvas has `role="img"` and `aria-labelledby` pointing at a short static title (“the aviary”) plus the live region. Birds are also in a **keyboard focus list** (not announced as a data table).

`DECISION`: generate narration **client-side** from snapshot so it can include listen-in focus (“pip’s call is closer now”) without a server round trip. Share templates with notebook via `packages/prose`. Server notebook remains the durable observer; narration is ephemeral.

### 9.2 Keyboard

- Tab: top-bar icons in order (account, accessibility, notebook, offer, settle), then first bird.
- Arrow left/right: cycle birds (spatial order by `x`).
- Enter / Space on bird: listen-in toggle.
- Escape: exit listen-in; if a sheet is open, close sheet first.
- Offer sheet: full tab cycle, three offer buttons, Esc closes.
- Focus ring: 2px soft outline, contrast-checked on day and night (`#1a1a1a` on day sky, `#f4efe6` on night). Never rely on color alone.

### 9.3 Captions

Opt-in (or forced on WebAudio fallback). Short naturalist fragments generated from the **same atom sequence** the synth just scheduled:

- “a soft three-note rise”
- “a low trill, paused, low trill again”
- “a single sharp call from the back perch”

Position: near the bird’s head, screen-space, fade with call envelope. AA contrast. Reduced-motion: captions still fade (opacity is not vestibular-heavy); if we must, cut on/off without slide.

Do not use `aria-live` for every caption (too fast). Sighted/hard-of-hearing users read them visually; SR users get the slower narration, which may mention calling.

### 9.4 Reduced motion

See §7.10. Settings override exists for users whose OS preference doesn’t match their wish for this product.

### 9.5 Contrast and copy

All user-copy in chrome, sheets, errors, captions: WCAG AA minimum. Scene itself has no required reading text. Prefer AAA for system errors.

### 9.6 What we will not do

- Expose personality vectors to ARIA.
- Ship a static screenshot as the “accessible version.”
- Announce “welcome back.”
- Use `aria-live="assertive"` except visit-revoked / session-lost.

---

## 10. Performance budgets and observability

### 10.1 Budgets

| Budget | Gate |
|---|---|
| Initial JS gzipped | `< 2MB` (fail CI), target `< 400KB` boot path |
| Time to first bird visible | `< 500ms` on mid-tier mobile / 4G (p75 RUM, p50 synthetic) |
| Idle scene | `60fps` on a 5-year-old mid laptop, 30-minute session |
| Memory | no growth over 30 minutes (heap ±2MB after GC, CI) |
| Snapshot size | `< 16KB` gzipped at 7 birds |
| Tick p99 | `< 5s` alarm; expected `< 40ms` hot tick, `< 250ms` 3-hour catch-up |
| Event ingest p99 | `< 200ms` |

First-bird definition: at least one bird sprite has drawn non-transparent pixels. Quiet field does not count.

### 10.2 How we hit TTFB / TTFBird

- Edge-served HTML + tiny CSS sky.
- For authenticated HTML requests at the edge: if a cache-aside snapshot is warm (`private` store keyed by session is **not** shared CDN — use the origin to inline snapshot on the document request instead of a second RTT). Pattern: document request hits API, API injects `window.__BOOT_SNAPSHOT__`. One RTT.
- Scene module imported first; settings chunks after idle.
- No webfonts required for the scene; system UI font for chrome. If a notebook serif is desired, load it only with the notebook chunk.
- Images: none on critical path.

### 10.3 Runtime discipline

- Single rAF. No CSS animations on birds (except reduced-motion CSS fades on chrome).
- Audio voice pool size 12.
- Notebook virtualized list; detach DOM for offscreen entries; no retained closures over old snapshots.
- `OffscreenCanvas` when available; still 60fps fallback on main thread with a frame-time governor (drop leaf ornaments if frame `> 18ms`).

### 10.4 Observability (aggregate only)

Synthetic fleet: 4 geos, Chrome+Safari, hourly, scripted session (open, wait, listen-in, offer, settle). Metrics: TTFBird, fps p5, audio-context errors, tick latency (from a canary account that is **not** mined for bird behavior — canary events are synthetic and excluded from any future research; still do not warehouse canary vectors).

RUM (privacy-safe):

- navigation timing, TTFBird (no bird ids)
- fps histogram
- audio init failure boolean
- API latency/status
- session duration histogram **without account dimension**

**Never:** per-bird events, trait values, offer kind histograms by account, notebook text, emails.

Alarm: tick p99 `> 5s` for 5 minutes. Alarm: 5xx `> 2%`. Alarm: TTFBird p75 `> 800ms`.

Logging: `account_id` UUID only, no email. Request ids. No payload dumps of events in info logs.

### 10.5 What we deliberately do not measure

- DAU/WAU as a product success dashboard that drives streaks.
- “Engagement” as click counts.
- Funnel optimization on settle vs close.
- Population drift averages.
- Visit leaderboards (do not compute).

Operational health only. Product quality is calibration tests + qualitative watch-throughs.

---

## 11. Rollout

### 11.1 Phased ship (still one v1 product)

**Internal dogfood (week −4).** Staff accounts, two birds, sim on, no visits. Daily watch tests for canned-call detection and greeting sameness.

**Closed preview (week −2).** ≤ 100 accounts, invite-only magic-link allowlist. Enable visits among testers. Accessibility audit with SR and reduced-motion users (paid, not a hallway check).

**v1 public.** Remove allowlist. Start every aviary at two birds. Age gates as in D7 — do **not** ramp bird caps by launch cohort. The PRD’s ramp is aviary age, not operational gradualism.

Operational ramp (infra, not product):

1. Tick worker pool 2 → 4 → N as hot-account ZSET grows.
2. Mail provider warmup.
3. Feature flags that are **off-ramps for outages**, not product experiments: `flag.visits`, `flag.weather`, `flag.audio`. If audio flag is off, captions on. Flags must not create a second personality writer.

Do not flag-gate reduced-motion or narration. Shipping without them is not v1.

### 11.2 Day-one instrumentation

- Synthetic TTFBird + fps + tick p99.
- Auth success/fail (no email in metrics).
- Event ingest error rate.
- Audio-context failure rate (aggregate).
- Hard-delete sweeper heartbeat.

Calibration harness (CI, not prod telemetry):

- 7-day and 21-day simulated presence traces assert D15.
- Double-device presence does not double drift.
- Absence of 14 days does not lower traits.
- Catch-up ≡  N live ticks (hash equality of state).
- Notebook never contains banned attendance phrases.

### 11.3 Content/ops at launch

- Six species art + grammar signed off.
- Copy catalogs reviewed for voice split.
- Privacy policy page in settings (plain text): names aggregate categories; explicitly excludes per-bird interaction state.
- Unsupported-browser page.
- Export and delete tested end-to-end.

### 11.4 Adoption pacing after launch

Do not accelerate D7 because “retention.” Do not sell a third bird. If audio recognizability work later raises the cap, that is a new PRD; v1 stays at seven.

---

## 12. Risks

### 12.1 Drift calibration (highest product risk)

**Failure:** too fast → Tamagotchi numbers the user can feel session-to-session. Too slow → screensaver. Wrong presence definition → silent population-wide acceleration.

**Mitigations:** D4/D5 conjunction; server min/max presence clamp; D15 CI harness; no client-authored traits; 30-day post-launch human review of **instrument** deltas on dogfood accounts only (engineers looking at their own export files, not a warehouse). If dogfood median weekly Δmax `> 0.05`, cut `k_*` before public.

**Do not** “fix” slow drift by adding negative neglect drift.

### 12.2 Sync correctness

**Failure:** double presence, missed catch-up, settle fights, visitor events leaking into host drift.

**Mitigations:** max-not-sum presence; visit endpoints cannot insert `interaction_events`; catch-up determinism tests; `sim_rev` monotonic; event idempotency; 5-minute offline event TTL.

### 12.3 Audio uncanniness

**Failure:** users hear a loop, or chorus phase-cancels, or all warblers sound like one warbler.

**Mitigations:** per-bird signature from id; no sample loops; QA lineup test; motif RNG per call; listen-in ramps not cuts. If a species fails recognizability, fix grammar, do not add a visual nametag in the scene.

### 12.4 Accessibility regressions

**Failure:** live region spam; reduced-motion as `animation: none` leaving a dead tableau; captions that don’t match the call; focus rings invisible at night.

**Mitigations:** narration rate limiter; second renderer path; caption-from-same-atoms; contrast fixtures in two lighting phases; a11y launch criterion, not a patch.

### 12.5 First-frame “app woke up”

**Failure:** spinner, fade-in, unison greeting, welcome toast from a helpful library.

**Mitigations:** lint banned copy; no toast library; greeting stagger tests; quiet-field HTML; mid-pose boot. Code review checklist item on every chrome PR: “does this announce?”

### 12.6 Performance cliffs

**Failure:** 2MB becomes 2.4; 30-minute leak from audio nodes; 7 birds drop to 40fps; TTFBird 1.2s on 4G.

**Mitigations:** CI bundle budget; heap test; ornament governor; one-RTT bootstrap; code-split settings.

### 12.7 Privacy leakage

**Failure:** email as Kafka key, per-bird events in RUM, visit emails used as social graph.

**Mitigations:** synthetic UUID rule in schema review; `opsdb` isolation; no visit graph queries; mailer decrypts email at send only.

### 12.8 Scope creep (cultural)

**Failure:** “just a streak toggle,” “just a public gallery,” “just hunger that’s cute.”

**Mitigations:** this plan’s bans are implementation-level (no columns, no endpoints, no copy). Reviewers reject schemas that store visit-frequency for display.

### 12.9 Weather / day-night timezone bugs

**Failure:** UTC night for a Tokyo morning; weather desync across devices.

**Mitigations:** timezone on account, snapshot lighting computed server-side, weather in canonical state.

### 12.10 Offer saturation

**Failure:** cooldown missing → curiosity slams into 0.95 in one evening.

**Mitigations:** 180s server cooldown; small `k_c`; approach term.

---

## 13. Delivery workstreams

Parallelizable after schema freeze.

1. **Platform** — Postgres, Redis, auth, sessions, synthetic ids, mail outbox, delete/export.
2. **Sim core** — `packages/sim` + worker + catch-up + calibration tests. No UI.
3. **Snapshot API + event ingest** — contracts, idempotency, presence merge.
4. **Scene renderer** — canvas, perches, day/night, idle motion, quiet field, reduced-motion path.
5. **Audio grammar** — worklet, listen-in ramps, fallback silence+captions.
6. **Interactions chrome** — top bar, offer, settle undo, naming, first-run fly-in.
7. **Notebook + narration prose** — shared package, sparsity, banned-phrase tests.
8. **Visits** — invite, readonly client, revoke, log, optional email.
9. **A11y + perf gates** — keyboard, contrast, CI budgets, synthetics.
10. **Launch hardening** — unsupported browser, privacy copy, dogfood calibration.

Critical path: 2 → 3 → 4/5. Chrome and visits hang off snapshots. Do not start visits before visitor snapshots are guaranteed event-free.

---

## 14. Testing strategy

- **Unit:** drift monotonicity, presence conjunction, mood hysteresis, catch-up ≡ live, prose banned tokens, invite expiry.
- **Property:** for random event traces, traits never decrease; `sim_rev` increases; bird ids stable across rename.
- **Integration:** magic-link consume-once; two sessions presence-max; visitor 403 on `/v1/events`; revoke → 410.
- **Visual:** screenshot quiet field, midday, night, settled, rain — no chrome badges.
- **Audio:** grammar snapshot tests (caption string + atom list), not WAV goldens.
- **Perf CI:** bundle, 30-min memory, tick bench.
- **A11y CI:** axe on settings/auth; manual SR pass is a release checkbox.
- **No test** should assert a streak, a welcome toast, or a hunger state — those tests would encode the wrong product.

---

## 15. Implementation notes that prevent the usual mistakes

- Do not put `welcome` in the auth success path. After magic-link, 303 to `/` and let a bird notice.
- Do not add hover tooltips with mood names on birds.
- Do not show “last visited” anywhere.
- Do not use email as `account_id` “just in staging.”
- Do not last-write-wins a full bird row from a client “save.”
- Do not pause idle motion because `!hasFocus` while still visible — presence may drop, but the place keeps moving; only `document.hidden` stops rendering.
- Do not implement settle as account logout.
- Do not let listen-in mute the chorus to zero.
- Do not archive notebook entries.
- Do not add a third offer type in v1.
- Do not design tables for native offline sync.

---

## 16. Success criteria (v1 done)

A person can sign in with a magic link, meet two named birds already in motion, sit without clicking and later see (in instruments, then over weeks in feel) that attention mattered, listen in, offer a seed, settle or just close the tab, read a sparse notebook that sounds like a field note, invite one friend to look and revoke them, export or delete their account, use the product with a keyboard, a screen reader, captions, or reduced motion, and never encounter a score, a streak, a death, or a “welcome back.”

That is the whole ship.
