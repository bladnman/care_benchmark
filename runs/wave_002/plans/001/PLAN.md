# Pocket Aviary — v1 implementation plan

This plan is an execution document for a separate engineering team. It interprets the PRD into architecture, contracts, algorithms, sequencing, and acceptance checks. It does not implement the product.

Wherever the PRD leaves a number, algorithm, or stack choice open, this plan makes a named decision and states why. Those decisions are binding for v1 unless calibration during build proves them wrong; if a calibration changes a number, keep the invariant the number was serving.

---

## 0. Binding product invariants

These are not style notes. They are implementation constraints. A PR that violates one is incorrect even if tests are green.

1. **Aliveness is load-bearing.** First painted frame is mid-action. No spinner, no fade-from-static, no ready-pop, no entry theatrical. Quiet field only while the snapshot is genuinely not yet available.
2. **Notice, never announce.** No welcome toast, no “you’ve been gone N days,” no visit badge, no streak, no achievement copy anywhere — including the notebook.
3. **Idle attention is the primary interaction.** Presence is the conjunction of `visibilityState === 'visible'`, window focus, and pointer/key activity inside a 4-minute window. Any laxer definition is a product bug.
4. **Personality is server-canonical and monotonic toward expressive.** Clients never write trait values. Neglect never decrements traits. Ambient quietness is a separate derived envelope, not negative drift.
5. **The user never sees personality numbers.** Not in UI, not in ARIA, not in captions, not in a debug panel, not behind a flag.
6. **Calls are procedural.** No recorded call loops, including in the WebAudio-unavailable path. Fallback is silence plus captions.
7. **One aviary per account. Two birds at start. Hard cap seven.** New birds arrive by aviary age, never by engagement score or payment.
8. **Bird identity is a stable UUID.** Rename, species-pool edits, client migrations, and export/reimport never mint a new bird in place of an old one.
9. **Synthetic account UUID everywhere.** Email lives in one encrypted column plus a lookup hash. Email is not a partition key, log field, or analytics dimension.
10. **Per-bird interaction history never enters aggregate telemetry, warehouses, or training sets.**
11. **Voice split is mechanical.** Product surfaces: lowercase naturalist present-tense. System surfaces (auth, errors, sync, account, accessibility settings): matter-of-fact sentence case.
12. **Visits are read-only, opt-in per invite, off by default, non-co-present.** Visitor attention does not drift host birds.
13. **No native app, no payments, no multi-aviary, no public discovery, no gamification, no Tamagotchi decay, no push about the aviary.**

---

## 1. Scope

### 1.1 In v1

- Browser-only SPA + API. Last two major versions of Chrome, Safari, Firefox, Edge.
- Email magic-link auth, per-device revocable sessions, email change with verify-before-swap.
- One canonical aviary per account, synced by architecture (server is the only personality writer).
- Server-side simulation: 60s logical tick, event-triggered micro-tick for interactive events, deterministic catch-up after absence.
- Two starter birds chosen by the system; user names them. Age-gated arrivals up to seven.
- Six-species pool with distinct silhouettes, palettes, and call-grammar motifs. One night-active species.
- Interactions: return-greeting, sit/watch (presence), listen-in, offer (seed / song fragment / still pool), settle with 5s undo, field notebook (read-only).
- Scene: single horizontal no-pan no-zoom view, three perch zones, local-time day/night, rare weather, ambient leaf/feather drift, chrome-free scene, fading top bar.
- Accessibility shipping on day one: naturalist screen-reader narration, designed reduced-motion mode, call captions, WCAG AA chrome, full keyboard path.
- Visit invitations: email magic-link, read-only ambient view, 30-day unused expiry, immediate revoke, silent visit log, notification toggle off by default.
- Account export (JSON emailed), soft-delete 30 days then hard-delete.
- Performance budgets: initial JS ≤ 2MB gzipped (target much lower), first bird < 500ms on mid-tier 4G, 60fps idle on a 5-year-old laptop, no client memory growth over 30 minutes.

### 1.2 Explicitly out of v1

Native iOS/Android clients; passwords/SSO; payments and tiers; shared/household/multi-aviary accounts; customizable scenes; panning/zoom; drag-to-place birds; recorded-audio fallback; LLM-authored notebook/narration (see decision N1); public discovery, profiles, follows, comments, chat, avatars, leaderboards; push/email about the aviary except magic links, export links, optional visit-notify, and transactional account mail; any streak/score/badge/level/XP/calendar-of-visits surface; hunger/death/distress meters; client-owned personality; last-write-wins on vectors.

### 1.3 Named decisions (ambiguities resolved)

| ID | Ambiguity | Decision | Why |
|---|---|---|---|
| N1 | How notebook/narration/captions are authored | Constrained prose grammar + fact assembler. No third-party LLM in v1. | Voice control, privacy boundary, offline-determinism, no per-bird data leaving the sim DB. |
| N2 | Tick when no client is connected | Logical 60s tick is deterministic in `(state, event_log, aviary_rng, timezone)`. Executor is **lazy catch-up on snapshot/event**, plus an eager worker for aviaries with a presence ping in the last 7 days so interactive sessions stay warm. | Semantically “continues without the viewer”; operationally affordable; catch-up of 2 weeks must stay < 100ms. |
| N3 | Offers/greetings cannot wait a minute | Interactive events run a **micro-tick** in the ingest transaction: immediate action + mood nudge. Personality deltas still only in the slow tick. | Aliveness of gesture without letting clients author traits. |
| N4 | “Quieter after neglect” vs monotonic traits | Add `expression_gain` ∈ `[0.35, 1.00]`, a medium-timescale envelope. Traits never fall. Greeting/call *expression* scales with gain. | Implements ambient quietness without Tamagotchi punishment. |
| N5 | Personality on the wire | Snapshots send **derived render params** only. Raw vectors exist in DB + account export. | Prevents a stats panel from being one DevTools copy away from becoming UI. |
| N6 | Presence activity window | **4 minutes**, presence ping every **30s** while the triple conjunction holds. | PRD says lean long; watching without moving is the product. |
| N7 | Scene renderer | **Canvas 2D scene graph** + procedurally parameterized vector birds. Preact (or equivalent ~3KB) **only for chrome/settings**. | 60fps control, mid-action first frame, reduced-motion crossfades, tiny assets. |
| N8 | Transport | HTTP pull + event POST. No WebSocket in v1. Snapshot poll 20s while visible; immediate pull on visibility/focus/resume. Event responses may include a fresh snapshot. | PRD model is pull/interpolate; WS adds presence-sync complexity we do not need. |
| N9 | Species pool | Six named species in §5.4. Night-active: **nighthawk**. | Coherent temperate-edge set; one nocturnal call signature. |
| N10 | Age gates for arrivals | 3rd @ 10 weeks, 4th @ 5 months, 5th @ 9 months, 6th @ 14 months, 7th @ 20 months of aviary age. | Matches “few months → third; year-old → five or six.” |
| N11 | How a new bird appears | No catalog. After the age gate, next host session the bird is already in the snapshot and enters with a soft fly-in. Default name applied; rename in bird settings. Notebook may notice it. | First encounter is meeting, not configuring. |
| N12 | Visitor auth | Visit token is sufficient. Visitor need not have an account. Token hashed at rest. | Matches “follow the link and see.” |
| N13 | Soft-delete vs tick | Freeze personality/notebook/weather *generation* during soft-delete. On restore, catch up time-of-day and mood priors only. | Recovered birds are the same birds, not 30 days of unattended drama and not stuck in last night forever. |
| N14 | Offer cooldown | **4 minutes per bird**, all offer kinds share one cooldown. | Prevents curiosity saturation; keeps the gesture register. |
| N15 | Listen-in ramp | **1000ms** equal-power fade; others floor at **0.28** linear gain. | Feels like listening, not channel-switching; never mute the place. |
| N16 | Top-bar fade | Begin fade after **4s** input stillness; 800ms to 12% opacity. Restore on pointer/key. Always 100% opacity while a panel is open or keyboard focus is in the bar. | Stillness without trapping keyboard users. |
| N17 | Timezone | IANA zone reported by the client on session start and whenever it changes. Server stores it on the account. Day/night and mood priors use that zone. | Local morning must be aviary morning. |
| N18 | Starter pair | Deterministic from `hash(account_id)`: two different species, different register (one higher motif, one lower), complementary seeds (one higher boldness, one higher warmth). | Retry/refresh cannot re-roll identity; first chorus is separable by ear. |
| N19 | Initial JS target | Cap is 2MB gz. **Engineering target: ≤ 350KB gz to first bird**, remainder code-split. | 2MB cannot hit 500ms on mid-tier 4G; the cap is a ceiling, not a goal. |
| N20 | Inlined snapshot | Authenticated `GET /` HTML includes a `<script type="application/json" id="aviary-boot">` snapshot so the scene core can draw without a second round trip. | Primary lever for time-to-first-bird. |

---

## 2. Architecture

### 2.1 Shape

Three deployables, one monorepo.

```
apps/web            client (scene + chrome)
apps/api            auth, snapshots, events, visits, account
apps/sim-worker     eager tick for recently-present aviaries; catch-up helper
packages/sim        deterministic tick, drift, mood, greeting, notebook facts
packages/prose      naturalist grammars (notebook, narration, captions)
packages/species    silhouettes, palettes, call motifs, pose tables
packages/protocol   snapshot/event TypeScript types, schema_version
packages/telemetry  allowlisted metric names + redaction
```

`packages/sim` must run identically in the API process (micro-tick + lazy catch-up) and in the worker. No hidden wall-clock I/O inside the step function.

### 2.2 Client / server split

**Server owns:** account identity, sessions, canonical bird records, personality vectors, `expression_gain`, mood, perch intent, weather seed/state, notebook entries, visit grants, append-only event log, tick time, adoption age gates, export/delete.

**Client owns:** rendering, interpolation, idle micro-motion within the current snapshot’s motion contract, procedural audio, listen-in mix, presence detection, optimistic settle lighting (5s undo), caption placement, chrome.

**Client never owns:** personality values, notebook authorship, greeter selection as a permanent fact, weather generation, adoption unlocking, visit authorization.

### 2.3 Render pipeline boundary

```
boot HTML + inlined snapshot
        ↓
scene-core (first paint: place birds at snapshot phases, start RAF)
        ↓
audio-worklet (lazy, after first bird is on screen)
        ↓
chrome (top bar, panels) — code-split, after first bird
        ↓
poll /aviary/snapshot  +  POST /aviary/events
        ↓
interpolator consumes snapshot.seq, does not reset motion clocks unless seq jump or resume
```

The scene loop may read snapshot fields and local time. It may not infer new personality. If a snapshot is late, keep interpolating the last motion contract; do not freeze to a T-pose.

### 2.4 Process topology

- **API:** horizontally scalable, sticky-session not required. All writes go through Postgres transactions keyed by `aviary_id`.
- **Postgres 16:** source of truth. Logical replication later if needed; not v1.
- **Redis:** session store optional (sessions can live in Postgres); use Redis only for (a) magic-link one-time tokens, (b) per-email magic-link rate limit, (c) short snapshot cache ≤ 5s for hot aviaries. Cache must invalidate on micro-tick.
- **Object-free email:** transactional provider (SES or Postmark). Templates: magic link, email-change verify, export-ready, optional visit-notify, visit invite.
- **CDN:** immutable hashed static assets. HTML for `/` is origin-dynamic (cookie).
- **No analytics warehouse connection to the sim database.** Operational metrics emit from API/worker process memory, never by selecting bird rows.

### 2.5 Consistency model

One writer for bird physiology: `sim.apply`. Two entry points call it:

1. `POST /aviary/events` for interactive types → lock aviary row → catch-up to now → apply event micro-tick → persist.
2. Worker or snapshot catch-up → lock aviary row → apply N logical minutes → persist.

Use `SELECT … FOR UPDATE` on `aviaries.id` to serialize. Event insert is in the same transaction. Personality updates happen only inside `sim.apply` as additive deltas.

There is no CRDT and no client merge. Two devices are two event sources, one ordered log.

### 2.6 Time

- `server_now` = timestamptz, authoritative for tick count.
- `logical_minute = floor((server_now - aviary.created_at) / 60s)`.
- `last_tick_minute` stored on the aviary. Catch-up applies `last_tick_minute+1 … current_minute`, collapsing empty spans (see §6.2).
- Client `performance.now()` is only for animation and audio scheduling.
- Client sends `client_event_at` for ordering hints; server stores both timestamps and **applies in `server_received_at` order**. Client time is never trusted for drift.

### 2.7 Suggested runtime stack

Binding enough to start, replaceable if a choice fails a budget:

- TypeScript everywhere (shared protocol types).
- API: Node 22 + Fastify (or equivalent), behind TLS terminator.
- Client: Vite, Preact for chrome, Canvas 2D scene, AudioWorklet.
- DB: Postgres 16, `pgcrypto` or app-level AES-GCM for email.
- Tests: Vitest for sim determinism; Playwright for first-bird / keyboard / reduced-motion; axe on system surfaces.
- CI: size-limit, soak, voice lint, telemetry allowlist lint.

Do not introduce a native-client protocol, GraphQL, or a general realtime platform in v1.

---

## 3. Data model

### 3.1 Accounts and sessions

```
accounts
  id                    uuid pk          -- synthetic; the only identifier elsewhere
  email_ciphertext      bytea not null   -- AES-GCM
  email_lookup_hash     bytea unique     -- HMAC-SHA256(email_normalized, EMAIL_PEPPER)
  email_pending_cipher  bytea null
  email_pending_hash    bytea null
  tz                    text not null default 'Etc/UTC'  -- IANA
  created_at            timestamptz
  deletion_marked_at    timestamptz null
  visit_notify          boolean not null default false
  reduced_motion_opt_in boolean not null default false  -- OR with prefers-reduced-motion
  captions_opt_in       boolean not null default false
  -- no email in any other table
```

```
sessions
  id                    uuid pk
  account_id            uuid fk
  token_hash            bytea unique     -- only hash stored
  created_at            timestamptz
  last_seen_at          timestamptz
  user_agent_coarse     text             -- browser family + OS family only
  revoked_at            timestamptz null
```

```
magic_links
  id                    uuid
  account_id            uuid null        -- null if creating on first consume
  email_lookup_hash     bytea
  purpose               enum('sign_in','email_change','visit','export')
  token_hash            bytea unique
  expires_at            timestamptz      -- sign-in: +15m
  consumed_at           timestamptz null
  meta                  jsonb            -- visit_id / new email hash / export job; never raw email
```

Rate limit: 5 magic-link requests / email_hash / 15 minutes; 20 / hour. Same limiter for visit invites from a host.

### 3.2 Aviary

```
aviaries
  id                    uuid pk
  account_id            uuid unique
  created_at            timestamptz      -- age-gate clock
  last_tick_minute      bigint not null default 0
  last_tick_at          timestamptz
  rng_seed              bytea not null   -- 32 bytes, fixed at creation
  weather               jsonb not null   -- { kind, started_at, ends_at }
  lighting              enum('day_cycle','settled') not null default 'day_cycle'
  settled_at            timestamptz null
  expression_gain       real not null default 0.72
  notebook_rarity       jsonb            -- last_entry_at, last_boost_keys
  bootstrapped          boolean not null default false  -- false until two starters exist
```

`expression_gain` is aviary-level (attention to the place), with per-bird listen-in also feeding that bird’s social/vocal drift. Per-bird quietness still happens because greeting uses both aviary gain and that bird’s traits.

### 3.3 Birds

```
birds
  id                    uuid pk          -- stable identity
  aviary_id             uuid fk
  species_id            text not null    -- see §5.4
  name                  text not null    -- user-facing, renameable
  adopted_at            timestamptz
  arrival_seen_at       timestamptz null -- first host snapshot that included this bird
  -- personality, server-only
  boldness              real not null
  social_warmth         real not null
  vocal_frequency       real not null
  plumage_saturation    real not null
  curiosity             real not null
  -- fast state
  mood                  enum('wary','content','curious','drowsy','alert')
  mood_entered_at       timestamptz
  perch_zone            enum('front','middle','back')
  perch_slot            smallint         -- 0..n-1 within zone
  action                jsonb            -- current micro-action or null
  last_offer_at         timestamptz null
  call_signature        jsonb not null   -- { seed, register_offset, interval_bias }
```

Trait storage: `real`, clamped `[0, 1]` in `sim.apply`. Seed new birds in `[0.28, 0.44]` with species priors (nighthawk: lower vocal_frequency daytime, higher curiosity at night). Never recompute traits from logs.

Indexes: `(aviary_id)`, unique `(aviary_id, id)`. Max 7 enforced by a trigger and by API.

### 3.4 Event log (append-only)

```
aviary_events
  id                    uuid pk          -- client-generated, idempotent
  aviary_id             uuid fk
  account_id            uuid             -- actor; visitors never write
  session_id            uuid null
  bird_id               uuid null
  type                  text not null
  payload               jsonb not null
  client_event_at       timestamptz
  received_at           timestamptz
  applied_at            timestamptz null
```

`UNIQUE (id)`. No updates except `applied_at`. No deletes except hard account deletion.

Event types:

| type | payload (min) | micro-tick? | presence-ending? |
|---|---|---|---|
| `session_open` | `{ absence_seconds, visibility: 'fresh'\|'return'\|'resume' }` | yes — greeting | no |
| `session_close` | `{ reason: 'unload'\|'hidden'\|'settle' }` | yes — clear live flags | yes |
| `presence_ping` | `{ conjunction: true }` | no — buffered for slow tick | no |
| `listen_in_start` | `{ bird_id }` | no (mood none; drift later) | no |
| `listen_in_end` | `{ bird_id, duration_ms }` | no | no |
| `offer` | `{ kind: 'seed'\|'song'\|'pool', near_bird_id? }` | yes — reaction | no |
| `settle` | `{}` | yes — lighting | yes (ends current presence window) |
| `settle_undo` | `{}` | yes — if within 5s server-side as well | no |
| `rename` | `{ bird_id, name }` | no (direct name write, not sim) | no |
| `arrival_ack` | `{ bird_id }` | no | no |

Invalid types are rejected. Visitors have no event writer.

### 3.5 Notebook

```
notebook_entries
  id                    uuid pk
  aviary_id             uuid fk
  written_at            timestamptz
  local_date            date             -- in account tz, for “tuesday — …”
  prose                 text not null
  fact_keys             text[]           -- e.g. greet_order_inverted; never shown
  source_event_ids      uuid[]
```

No user edit columns. Indefinite retention until hard delete. `fact_keys` exist so rarity control can say “already wrote greet_order_inverted this week.” Never expose `fact_keys` on the client if they encode trait names; use opaque observation ids.

### 3.6 Visits

```
visit_invites
  id                    uuid pk
  host_account_id       uuid
  aviary_id             uuid
  visitor_email_cipher  bytea
  visitor_email_hash    bytea
  token_hash            bytea unique
  created_at            timestamptz
  expires_at            timestamptz      -- created_at + 30d if unused
  revoked_at            timestamptz null
  first_used_at         timestamptz null
```

```
visit_sessions
  id                    uuid pk
  invite_id             uuid
  started_at            timestamptz
  last_snapshot_at      timestamptz
  ended_at              timestamptz null
```

Duration ≈ `last_snapshot_at - started_at` if no explicit end. Approximate is enough. Host visit log shows hashed-to-plaintext email only after decrypt in the account service, never in logs.

### 3.7 Derived snapshot (what the client is allowed to see)

Not a table. Built by `protocol.snapshotFromState`.

```ts
type Snapshot = {
  schema_version: 1
  seq: number                 // monotonic per aviary, ++ on every persist
  server_now: string          // ISO
  tz: string
  lighting: 'day_cycle' | 'settled'
  local_solar: { phase: SolarPhase; warmth: number; dim: number }
  weather: { kind: 'clear' | 'rain' | 'wind'; intensity: number }
  greeting?: GreetingDirective
  arrival?: { bird_id: string }  // fly-in once
  offer_cooldown_until: Record<string, string>
  birds: BirdRender[]
  motion_epoch_ms: number     // for mid-action phase continuity
}

type BirdRender = {
  id: string
  species_id: string
  name: string
  mood: Mood
  perch: { zone: Zone; slot: number; x: number; y: number }
  plumage: { hue: number; sat: number; val: number; pattern: number }
  action: { kind: string; t0: number; dur: number; params: object }
  call: { signature: CallSig; next_window: [number, number] }
  listen_in_eligible: true
}

type GreetingDirective = {
  primary_bird_id: string
  form: 'glance' | 'two_note' | 'approach' | 'long_call'
  stagger_ms: number[]        // for any secondary acknowledgements
}
```

No trait names, no trait numbers, no presence totals, no visit counts, no “days since last session” field. Absence affects greeting **server-side** and arrives only as `form`.

### 3.8 Export document

Emailed JSON, schema versioned:

```ts
type ExportV1 = {
  export_version: 1
  exported_at: string
  account_id: string
  aviary: { created_at: string; tz: string; settings: object }
  birds: Array<{
    id: string; species_id: string; name: string; adopted_at: string
    personality: { boldness: number; social_warmth: number; vocal_frequency: number; plumage_saturation: number; curiosity: number }
    mood: Mood
  }>
  notebook: Array<{ written_at: string; prose: string }>
}
```

Export is the one user-facing place personality numbers exist. Do not link to it from the aviary chrome; it lives under account settings, matter-of-fact voice.

### 3.9 What we deliberately do not store

- Click counts as scores.
- Daily visit calendars.
- Visitor presence as drift input.
- Per-leaf or per-feather entities.
- Raw pointer traces (presence is a boolean ping, not a heatmap).
- Email plaintext.

---

## 4. API surface

Base: `https://api.<host>/v1`. Cookies: `HttpOnly; Secure; SameSite=Lax` session. CORS only the web origin. All JSON. System-voice error bodies:

```ts
type ApiError = { error: string; code: string; retryable: boolean }
```

Examples: `We couldn't sign you in. The link may have expired. Try requesting a new link.` / `Your session timed out. Sign in again to keep watching.` / `Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch.` / `This visit is no longer available.`

### 4.1 Auth

| Method | Path | Notes |
|---|---|---|
| `POST` | `/auth/magic-link` | `{ email }`. Always 202 with the same body (`If an account exists or can be created, we sent a link.`) to avoid enumeration. Send mail only if rate limit allows. |
| `GET` | `/auth/consume?token=` | One-time. Invalidates token. Sets session cookie. New email → create account + aviary + two starter birds in one transaction. |
| `POST` | `/auth/sign-out` | Revoke this session. |
| `GET` | `/me` | Account settings DTO, no birds. |
| `PATCH` | `/me` | `{ tz? }` and notification toggle. |
| `GET` | `/me/sessions` | Device list. |
| `DELETE` | `/me/sessions/:id` | Revoke. |
| `POST` | `/me/email-change` | `{ email }` sends verify link. Old email works until consume. |
| `POST` | `/me/export` | Enqueues export; 202. |
| `POST` | `/me/delete` | Soft-delete now. |
| `POST` | `/me/undelete` | Allowed only if `deletion_marked_at` within 30d. |

Soft-deleted accounts may sign in only to recover. Snapshot routes return `410` with a recover affordance.

### 4.2 Aviary state and events

| Method | Path | Notes |
|---|---|---|
| `GET` | `/aviary/snapshot` | Auth required. Catch-up then return `Snapshot`. If `?seq=N` and current is N, `304` empty. |
| `POST` | `/aviary/events` | `{ events: ClientEvent[] }` max 32. Idempotent by `id`. Returns `{ accepted: id[], snapshot?: Snapshot }`. Always return snapshot for micro-tick types. |
| `PATCH` | `/aviary/birds/:id` | `{ name }` only. 1–24 chars, trimmed, no control chars. Identity unchanged. |
| `GET` | `/aviary/notebook?before=&limit=` | Reverse chrono. Default 20, max 50. `{ id, written_at, prose }[]`. |
| `GET` | `/` (web origin) | HTML shell + inlined snapshot when session cookie valid. |

`ClientEvent`:

```ts
type ClientEvent = {
  id: string              // uuid
  type: EventType
  bird_id?: string
  payload: object
  client_event_at: string
}
```

Reject events on someone else’s bird, on visitor tokens, while soft-deleted, or when `offer` is inside cooldown (`409` with `offer_cooldown_until`, matter-of-fact: `That bird isn't ready for another offer yet.`).

### 4.3 Accessibility prefs

Stored on `accounts`. Read/write via `GET/PATCH /me/accessibility`:

```ts
{ reduced_motion: boolean; captions: boolean }
```

Client still honors `prefers-reduced-motion` even if the stored flag is false (`OR`). Captions-on-by-default when WebAudio is missing, regardless of stored flag, until the user explicitly turns them off.

### 4.4 Visit flow

Host (authenticated):

| Method | Path | Notes |
|---|---|---|
| `GET` | `/visits` | Outstanding invites + log `{ visitor_email, at, duration_minutes }`. Settings surface only. |
| `POST` | `/visits` | `{ email }`. Creates invite, emails one-time link, 30d unused expiry. |
| `DELETE` | `/visits/:id` | Revoke immediately. |

Visitor (token, not account):

| Method | Path | Notes |
|---|---|---|
| `GET` | `/visit/:token` | HTML read-only aviary, or consume token → `visit_session` cookie. |
| `GET` | `/visit/snapshot` | Cookie-auth. Same `Snapshot` **minus** greeting, arrival fly-in-as-new, offer cooldowns, notebook. No `POST /events`. If revoked/expired: `410` + matter-of-fact page. |

Revocation takes effect on the next visitor snapshot (≤ 20s, plus immediate if they reload). Do not push.

Visit-notify (host opt-in): at most one email per visitor per 24h, matter-of-fact, no bird details: `A visitor opened your aviary around 3:12pm.` No in-product badge.

### 4.5 Keepalive / resume

Not a separate API. Client:

1. On `visibilitychange` → visible: `GET /aviary/snapshot` then `session_open` with `absence_seconds`.
2. On `pageshow` after `persisted` or after `document.timeline` gap > 5s: treat as resume, same as (1).
3. While visible and focused: poll snapshot every 20s; send `presence_ping` every 30s only if conjunction holds.
4. On hide/unload: `session_close` via `fetch keepalive` or `sendBeacon` to `/aviary/events`.

### 4.6 Idempotency and replay

Magic-link consume is single-use. Replaying a consumed sign-in link returns the expired-link error, never a new session.

Event ids are UUIDs from the client. Re-POST is a no-op success and may return the current snapshot.

### 4.7 Authorization matrix

| Actor | Snapshot | Events | Notebook | Rename | Invite | Visit log |
|---|---|---|---|---|---|---|
| Host session | yes | yes | yes | yes | yes | yes |
| Visitor token | yes (stripped) | no | no | no | no | no |
| Anonymous | no | no | no | no | no | no |
| Other account | no | no | no | no | no | no |

---

## 5. Simulation engine

Module: `packages/sim`. Pure functions + a persistence adapter.

### 5.1 Trait ranges and seeds

All five traits ∈ `[0, 1]`.

Starter seeds (then jitter ±0.03 from `rng_seed`):

| role | boldness | social_warmth | vocal_frequency | plumage_saturation | curiosity |
|---|---|---|---|---|---|
| complementary A | 0.40 | 0.32 | 0.36 | 0.34 | 0.38 |
| complementary B | 0.30 | 0.42 | 0.34 | 0.36 | 0.33 |

Subsequent arrivals: species prior midpoint 0.33 + hash jitter ±0.05, still inside `[0.28, 0.44]`.

### 5.2 Drift (slow tick only)

**Invariant:** `new_trait = max(old_trait, old_trait + delta)` with `delta ≥ 0`. Clamp to 1.

Presence-time for a tick window is the sum of accepted 30s pings whose three-way conjunction the client attested **and** whose `received_at` falls in a session that the server also saw as `session_open` without a later close. Discard pings after `session_close`, while hidden, or from visit sessions (impossible if we never accept them).

Let `P` = presence hours in the last 7×24h of *applied* pings (not this minute only).  
Let `L_i` = hours of listen-in on bird `i` in last 7d.  
Let `O_i_accept` = count of offers bird `i` actually responded to (not ignored) in last 14d, capped at 8.  
Let `O_i_near` = count of offers whose `near_bird_id = i` in last 14d, capped at 8.

Per bird, per trait, per slow tick (once per logical minute, but use rates so catch-up is equivalent to one integrated step over `Δt` minutes):

```
headroom(t) = 1 - t
rate_week = 0.03          // ≈ 0.03 toward 1.0 per week of “regular” presence
regular_hours = 80/60     // 80 minutes/week defined as “regular”
presence_weight = (P / regular_hours)

# integrate over Δt_weeks = Δt_minutes / 10080
drift_presence = rate_week * presence_weight * headroom * Δt_weeks

# listen-in strongly on warmth + vocal for that bird
drift_listen_warmth = 0.045 * (L_i / regular_hours) * headroom(social_warmth) * Δt_weeks
drift_listen_vocal  = 0.040 * (L_i / regular_hours) * headroom(vocal_frequency) * Δt_weeks

# offers
drift_curio = 0.008 * O_i_accept_in_this_window * headroom(curiosity)
drift_bold  = 0.006 * O_i_near_in_this_window * headroom(boldness)

# plumage: presence only, slowest
drift_plumage = 0.7 * drift_presence
```

Apply:

- `boldness += drift_presence * 0.35 + drift_bold`
- `social_warmth += drift_presence * 0.40 + drift_listen_warmth`
- `vocal_frequency += drift_presence * 0.30 + drift_listen_vocal`
- `plumage_saturation += drift_plumage`
- `curiosity += drift_presence * 0.25 + drift_curio`

Settle does not add drift. It closes the presence window cleanly so the next ping cannot leak.

**Calibration targets (test harness, not user-visible):**

- Fixture `regular`: 20 min presence × 4 days. After 7 days: at least one trait +0.020 to +0.045.
- Same fixture after 21 days: +0.07 to +0.14 on warmth or boldness, enough to change greeter order or perch prior in the behavior functions.
- Fixture `neglect`: 14 days zero presence after a warm period: **all traits unchanged** (± 1e-6). `expression_gain` may fall. Greeting probability falls.
- Fixture `mash`: 40 offers in 10 minutes with cooldown honored (~3 applied): curiosity move **< 0.01**. Cooldown is what saves the engine.

Tune `rate_week` only via this harness. Do not tune by watching a single session.

### 5.3 Expression gain (ambient quietness)

Aviary-level, **may fall**, not a personality trait, never shown.

```
target = lerp(0.35, 1.00, saturate(P_7d / regular_hours))
gain += (target - gain) * (1 - exp(-Δt_hours / tau))
tau_up = 18h
tau_down = 10d
```

Behavior uses `gain` as a multiplier on greeting chance and unobserved call rate. Floor 0.35 keeps the place alive.

### 5.4 Species pool

Coherent temperate hedgerow / garden edge. No rarity stat.

| `species_id` | silhouette | default palette | motif character | night |
|---|---|---|---|---|
| `warbler` | slim, tail flick | olive-yellow muted | high 3-note rise | no |
| `wren` | tiny, cocked tail | warm brown | rapid low jumble | no |
| `sparrow` | compact, stout bill | drab brown-grey | two-note chip | no |
| `finch` | conical bill, notched tail | dusty rose-brown | sweet even hops | no |
| `chickadee` | black cap, round | grey/ink/cream | clear whistled pairs | no |
| `nighthawk` | long wings, wide mouth | bark-grey | hollow low trill | **yes** |

Visual assets: vector path kits in `packages/species`, parameterized by plumage sat/hue. No large bitmaps.

Starter pair algorithm: from `account_id`, pick two distinct non-nighthawk species with different `register` (warbler/chickadee vs wren/sparrow/finch). Nighthawk is never a starter; it may arrive as a later age-gated bird.

### 5.5 Mood

Final v1 set: **`wary | content | curious | drowsy | alert`**.

Priors by local solar phase (account tz):

| phase | prior |
|---|---|
| dawn–morning | alert, curious |
| midday | content, curious |
| late afternoon | content |
| dusk | drowsy |
| night | drowsy; nighthawk → alert/curious |

Transitions, evaluated each logical minute or on micro-tick:

```
score(m) = prior(m, solar, species)
         + interaction(m)
         + weather(m)
         + neighbor(m)
         + personality(m)
         + small_noise(rng)
```

- Offer accepted → `content` +0.8, `curious` +0.4.
- Offer ignored → no punishment.
- Rain → `vocal` behavior dampen; mood +`content`/`drowsy` +0.3, `alert` −0.2.
- Wind → `alert` +0.3 or `wary` +0.2 (low boldness prefers wary).
- Neighbor `wary` in same or adjacent zone → `wary` +0.25 (spread).
- High boldness: `wary` −0.4.
- High curiosity: `curious` +0.3.
- High vocal_frequency does not force `alert`.

Pick argmax if it beats current by 0.35, else stay (hysteresis). Persist across sessions. Never snap to `content` on `session_open`.

Daily-ish reset: at local 4:30, blend 40% toward solar prior. Not a hard wipe.

### 5.6 Perch intent

Three zones × 3 slots (9 seats). Seven birds always fit; leave empty seats.

```
desired_zone =
  wary     → back, or middle if boldness > 0.7
  drowsy   → middle/back, low body pose
  curious  → middle, facing ambient movers
  alert    → front/middle
  content  → anywhere, warmth pulls toward occupied adjacent slots
  boldness shifts one zone toward front when > 0.55 and mood ≠ wary
```

Re-pick at most every 3–8 minutes (rng), plus on offer/greeting. Client interpolates the path. User cannot assign perches.

### 5.7 Bird-to-bird

Evaluated in slow tick and when a call is emitted:

- **Answer:** if bird A called in the last 2s, bird B answers with  
  `p = 0.18 * vocal_B * warmth_B * gain * proximity_factor`  
  `proximity_factor` 1.0 same zone, 0.6 adjacent, 0.3 far.
- **Chorus cap:** at most 3 simultaneous callers (4 if all vocal_frequency > 0.7). Queue others.
- **Wary spread:** see mood.
- No fighting, no dominance meters, no user-visible “bonds.”

### 5.8 Call-grammar runtime (server contract + client synth)

Server does **not** render audio. It emits a `CallSig` and a next-window so devices agree who is “about to call.”

```ts
type CallSig = {
  species_id: string
  seed: number            // individual, stable
  register_offset: number // cents
  interval_bias: number   // -1..1
}
```

Client grammar (`packages/species/calls`):

- Atom: oscillator mix (sine+triangle or pulse+noise), formant pair, exponential pitch envelope, duration 40–220ms.
- Motif: 2–5 atoms, species interval set (e.g. warbler +3 +2 semitone steps; wren irregular −1 +4 −2).
- Phrase: 1–3 motifs, gap 80–400ms jittered by seed + mood (drowsy longer, alert shorter).
- Schedule: unobserved inter-call interval  
  `base_ms / (0.35 + 0.65 * vocal_frequency) / gain`  
  species `base_ms` 8000–20000. Listen-in does not change the bird’s schedule, only mix.
- Mood morphs amplitude and gap, **not** the interval set. Recognizability lives in interval set + timbre, which stay fixed for the bird’s life.
- Never replay an identical atom-timing-pitch tuple; jitter ≥ 4ms or ≥ 8 cents every realization.

Night: non-nighthawk interval × 3.5, amplitude −8dB. Nighthawk inverts: more active after dusk.

### 5.9 Return-greeting (micro-tick on `session_open`)

Compute `absence_seconds` from last host `session_close`/`visibility hidden`. Do not use visit sessions.

Pick **one** primary greeter:

```
score = 0.50*boldness + 0.30*social_warmth
      + mood_mod + 0.08*rng
mood_mod = alert 0.20 | curious 0.15 | content 0.05 | drowsy -0.20 | wary -0.25
```

Form from absence × boldness:

| absence | form |
|---|---|
| < 5 min | `glance` (almost always) |
| 5–240 min | `two_note` if warmth/vocal high, else `glance` |
| 4–48 h | `approach` if boldness > 0.38 else `two_note` |
| > 48 h | `long_call` or `approach`; still **one** primary |

Secondaries: other birds with score > primary−0.15 may acknowledge with `glance` only, stagger 400–1800ms rng. Never unison.

Greeting is a `GreetingDirective` on the snapshot for this open only; next poll omits it. Procedural variation is mandatory: form + seed + `seq` + `absence` feed the animation/audio so two greetings never share timing.

If `bootstrapped` is false, skip greeting; run empty-field + first fly-in instead.

### 5.10 Offers (micro-tick)

Shared 4-minute cooldown per bird from `last_offer_at`.

**Seed.** Place a seed sprite at front-center (client also draws from action). Receiving bird = `near_bird_id` if in front/middle, else highest curiosity×proximity among non-drowsy, else nearest.

| mood × curiosity | reaction |
|---|---|
| curious/content and curiosity > 0.32 | approach 0.6–1.4s, peck, `accepted` |
| wary | wait 3–6s, then approach if curiosity > 0.30 else watch, `accepted` if approached |
| drowsy | 70% ignore, 30% slow approach |
| alert | inspect quickly, maybe peck |

**Song fragment.** Play one library motif (procedural, 4–6 fragments, not recordings) softly in the scene bus. Bird response:

- join if vocal_frequency > 0.34 and mood in {content, curious, alert}
- quiet if wary
- ignore if drowsy
- call-against if alert and vocal_frequency > 0.5

**Still pool.** Reflective ellipse, front plane, ~45s lifetime. Species bias: sparrow/warbler bathe more; finch/chickadee drink; nighthawk watch; wren watch then sip. Curiosity and non-wary raise approach.

Write `birds.action` and a short mood nudge. Mark `accepted | ignored` in the event `applied` payload for later drift. Do not change traits here.

### 5.11 Settle

Micro-tick sets `lighting = settled`, `settled_at = now`, birds (except nighthawk) bias `drowsy`, call windows stretch. Client starts the 3s lighting shift immediately.

`settle_undo` accepted if `now - settled_at ≤ 5.5s` (slack for RTT). After that, a click is just a click; user can still watch the settled aviary. Closing the tab is equivalent to ending presence; no scold path.

Re-engage after settle: any host click/key in the scene after the undo window sets lighting back to `day_cycle` via a `session_open`-like micro path (`payload.reengage = true`) without a textual welcome.

### 5.12 Weather

Rare, never a feature surface.

- Rain: 2–4 times per local week, 6–18 minutes, from `rng_seed` + ISO week. Dampens vocal schedule × 1.6, slight drowsy/content.
- Wind: 1–3 times per week, 4–10 minutes, rustle leaves (client ornament + server mood nudge).
- No thunder, snow, or user-facing forecast.

During catch-up, instantiate weather intervals that fall in the span so the user can return *during* rain. Do not write a notebook entry for every weather event.

### 5.13 Age-gated arrival

On catch-up, if `now - aviary.created_at` crosses a gate and `bird_count < 7` and no pending unseen arrival:

- Draw species from remaining pool (allow duplicates only if pool exhausted; prefer unused).
- Insert bird with new UUID, default name from species name-list (Pip, Wren, Moth, Ash, Nettle, Soot, Pebble, … skip taken names).
- Set `arrival_seen_at = null`. First **host** snapshot includes `arrival.bird_id`; client fly-in; client sends `arrival_ack`; server sets `arrival_seen_at`.
- Visitors do not trigger fly-in or ack; they just see the bird if already present.

Empty aviary only when `bootstrapped = false`. After starters named (or defaults accepted) and first fly-ins complete, `bootstrapped = true`. Never empty again, including after revoke/delete of visits.

### 5.14 Notebook writer

Runs at the end of a catch-up or once daily on the eager worker, not per tick.

Rarity: if last entry < 48h ago, skip unless a **boost fact** is new this local week:

- greeter identity swapped vs last 7 greetings
- rain coincided with a long preen (action histogram)
- new arrival
- stretch of > 3h presence with no offers (quiet morning)
- first accepted pool-bath for a bird

Forbidden facts: visit counts, presence totals as “you were here,” trait deltas, achievement language, second person.

Assembler: pick a template from `packages/prose/notebook` keyed by fact, fill with bird names, zone words, weather words, weekday lowercase. Output one to three short sentences. Example shapes already in the PRD; implement those families, do not invent gamified ones.

Voice lint in CI: banlist `you`, `your`, `achievement`, `streak`, `unlocked`, `level`, `xp`, `score`, `congrats`, `welcome back`.

### 5.15 Tick function (reference)

```ts
function applyCatchup(state: AviaryState, events: Event[], fromMin: bigint, toMin: bigint, rng: Rng): AviaryState {
  const q = events.filter(e => e.received_at in (fromMin, toMin]).sort(by received_at, id)
  let t = fromMin
  while (t < toMin) {
    const nextEventMin = minute(q[0]) ?? toMin
    const nextBoundary = nextScheduled(state, t) // mood 4:30, weather start/end, perch rechoice, arrival gate
    const jump = min(nextEventMin, nextBoundary, toMin)
    if (jump > t + 1n) {
      integrateSlow(state, jump - t)  // closed-form drift + gain + solar
      t = jump
    } else {
      stepMinute(state, t)
      t += 1n
    }
    while (q[0] && minute(q[0]) === t) applyEvent(state, q.shift())
  }
  maybeWriteNotebook(state)
  state.last_tick_minute = toMin
  return state
}
```

`integrateSlow` must be algebraically equivalent to `stepMinute` repeated for empty minutes (test this with 10k-minute fuzz).

Determinism: same `(state0, events, rng_seed, tz)` → same `state1`. Timezone changes mid-span use the tz in force at each minute (store tz-change as an event).

---

## 6. Sync model

### 6.1 Single canonical record

Laptop and phone both `GET /aviary/snapshot` and `POST /aviary/events`. They never exchange state. Personality cannot diverge because only `sim.apply` writes it, under the aviary row lock, in received order.

### 6.2 Why not last-write-wins

A client must never send `{ boldness: 0.62 }`. If such a field appears in a payload, reject the event and fail CI protocol tests.

The failure mode we are deleting: Device B opened an old snapshot, computed a full vector, wrote it after Device A’s morning presence. Additive deltas from the log make that unrepresentable.

### 6.3 What can conflict, and how we handle it

| Field | Strategy |
|---|---|
| personality, mood, perch intent, weather | server sim only |
| name | last received `rename` wins (user-visible, not drift) |
| settle vs undo | timestamp vs 5.5s window |
| accessibility prefs | last `PATCH /me` wins |
| tz | last report wins; tz-change event for sim |
| two offers same second | both enter log; second hits cooldown |
| two devices presence-pinging | both accepted; presence-time is union of minutes with ≥1 valid ping, **not** summed across devices (cap 1 ping / 30s / account) |

Presence de-dupe: unique `(aviary_id, date_trunc('30s', received_at))` for `presence_ping` so a laptop+phone on the same desk cannot double-count attention.

### 6.4 Offline / flaky network

Client queues events in `IndexedDB` with their ids. On restore, flush. Optimistic UI only for settle lighting and listen-in mix (purely local). Offers wait for the micro-tick snapshot before playing the *accepted* reaction; show the seed/pool immediately as a gesture, then the bird’s reaction when the server returns. If the POST fails, the object remains 8s then fades; no error toast in the scene. System-voice inline in the offer panel only if the panel is open: `We couldn't offer that. Try again.`

Do not run a client-side sim that later “uploads the truth.”

### 6.5 Soft-delete, export, hard-delete

Soft-delete: set `deletion_marked_at`, revoke visit invites, stop eager worker, reject events except undelete. Snapshots 410.

Hard-delete job (daily): wipe birds, events, notebook, visits, sessions, telemetry keys tied to `account_id`, then the account row. No backup warehouse copy. Export links expire 24h and are deleted with the account.

### 6.6 Snapshot interpolation contract

Client stores `last.seq` and `last.birds[id].perch`. On new snapshot:

- If `seq` is last+1 or small gap: tween perch over `min(4s, distance-based)` using a short ease-in-out hop/walk; do not teleport.
- If resume gap or `seq` jump > 50: retarget without a long flight across the whole scene; cross-fade pose (even in full-motion mode) over 400ms so the bird does not warp.
- Actions with `t0` in the past start mid-cycle (`phase = (now - t0) % dur`). This is the mid-preen first frame.
- Greeting directive plays once, keyed by `seq`, then is dropped locally even if a retry snapshot still contains it (client remembers `playedGreetingSeq`).

---

## 7. Frontend rendering pipeline

### 7.1 Surfaces

Routes:

- `/` aviary (host)
- `/signin` email field, matter-of-fact
- `/visit` visitor view (token consumed)
- `/unsupported` old browser
- settings, notebook, offer, settle are **panels over `/`**, not routes, except account-heavy settings may be `/settings` code-split

No marketing carousel in-app. First-run: sign-in → quiet field → name two suggested names (naturalist copy, not a catalog) → fly-in → already-in-motion scene.

### 7.2 Scene composition

One viewport-fitting horizontal world. World units: width 100, height 56. Three depth planes:

- **Back:** sky gradient (solar-driven), distant foliage silhouette, no interaction.
- **Mid:** perches, birds, pool, seed.
- **Front:** occasional branch/leaf, very low parallax (max 4 world-units shift over full width).

No scroll, pan, pinch-zoom. `touch-action: none` on the canvas. Resize: recompute perch `x` from zone layout so **every bird remains in frame**. Narrow phone: compress inter-perch spacing; never crop.

Palette: soft blues, greens, warm browns, muted ochres. No electric accents. Exact tokens live in a design-system file the visual designer owns; engineering uses CSS variables / a `palette.ts` keyed by `solar.warmth` and `solar.dim`. Night dims mid-plane; nighthawk stays readable (contrast against dusk, not neon).

### 7.3 Bird drawing

Each species is 6–8 vector parts (body, tail, far wing, near wing, head, beak, eye, optional cap). Plumage saturation from snapshot `plumage.sat` scales chroma only. Parts deform via a tiny bone chain (body, neck, head, tail) — not a general skeletal engine.

Pose table per species: `rest`, `fluff`, `preen_a`, `preen_b`, `scan`, `tilt`, `peck`, `sleep`, `hop`, `fly_arc` (5 keys). Idle chooser is **mood-shaped** and continuous:

| mood | idle mix |
|---|---|
| content | preen, weight-shift |
| wary | scan, back-zone bias already in perch |
| curious | tilt toward last call or leaf spawn |
| drowsy | fluff, sleep-adjacent, low crouch |
| alert | scan + more head snaps (still slow) |

Idle never hard-stops. Overlay a 0.15–0.35px (world) breathing sine on body scale, period 2.8–4.2s keyed by id, so “paused” is impossible.

### 7.4 First frame and loading

Boot sequence:

1. Paint quiet field immediately from CSS (sky color by local clock even before JS) — 0 spinner.
2. Parse `#aviary-boot` JSON.
3. Scene-core constructs birds at `action.t0` phases and perch positions; first RAF presents mid-action.
4. If boot JSON missing (cold cookie, slow origin): stay on quiet field, fetch snapshot, then step 3. Still no spinner. Optional one faint leaf if fetch > 300ms.
5. Audio context resumes only after a user gesture or after first presence activity; until then, captions-if-needed and visual calls (throat motion) still run.

Empty first-run: quiet field, then each starter fly-in 600–900ms apart. After that, `bootstrapped`.

### 7.5 Top bar

Icons only: account, accessibility, notebook, offer, settle. No counts, no badges, no unread pip for visits or notebook.

Fade: §N16. Hit targets ≥ 44px. Labels via `aria-label` in matter-of-fact or naturalist? **Icon labels are short naturalist verbs** (`listen` is in-scene; bar: `notebook`, `offer`, `settle`, `account`, `accessibility`). Account/accessibility open system-voice panels.

### 7.6 Offer / notebook / settle UX

- **Offer:** small panel from bar, three items, keyboardable. Confirming posts `offer`. Seed/pool appear in-scene; panel can close.
- **Notebook:** slide-over list, infinite scroll backward, read-only, naturalist type. No search, no share, no like.
- **Settle:** one click. 3s lighting. 5s undo on any scene click. No confirm modal.

Clicking a bird never opens offer. Clicking a bird is listen-in.

### 7.7 Reduced-motion mode

Trigger: `prefers-reduced-motion: reduce` OR account flag.

Designed surface, not “off”:

- Replace looping micro-motion with still poses; cross-fade 1.2–2.0s when pose or perch changes.
- Flight → dissolve between perches, 1.5s, no arc.
- No leaf/feather drift.
- Solar color still shifts, 2× slower.
- Greeting glance = one cross-fade to `tilt` and back, not a dart.
- Audio unchanged (unless user muted / no WebAudio).
- Drift, mood, notebook, captions all unchanged.

Store both pose textures from the same pose table so art is shared.

### 7.8 Visibility and battery

When `document.hidden`, cancel RAF, suspend AudioContext, stop leaf spawns. Do not stop sending `session_close`. On return, new snapshot + resume AudioContext + restart RAF mid-phase. Simulation has continued on the server.

### 7.9 60fps budget

Scene graph budget (idle, 7 birds, laptop 2019 i5-class):

- ≤ 7 bird draws × 8 parts
- ≤ 12 ambient particles
- no per-frame allocation in the hot path (preallocate particle pool, path2d cache)
- shadows: at most one soft blob per bird, not blur filters
- no canvas `filter` in the idle path
- hit-testing: zone buckets, not pixel walking

If frame time p95 > 18ms in CI laptop profile, drop particles first, then shadow blobs, never bird idle.

### 7.10 Memory

- Particle pool fixed size 16.
- Audio buffers recycled; motif render into a ring of 8 AudioBuffers.
- Notebook DOM virtualized; detach offscreen entries.
- No unbounded event queue in memory; IndexedDB for offline, in-memory max 64 pending.
- 30-minute soak test in CI: heap at 30m ≤ heap at 2m + 8MB.

---

## 8. Audio pipeline

### 8.1 Graph

```
[motif scheduler per bird] → gain_bird_i → chorus_bus
library fragment / pool drip (procedural) → gesture_bus
very low procedural air (filtered noise) → air_bus
chorus_bus + gesture_bus + air_bus → master (soft compressor, makeup)
```

AudioWorklet renders atoms; main thread only schedules and sets gains.

### 8.2 Listen-in mix

On engage (click/tap/Enter on focused bird):

- `gain_focus` → 1.0 over 1000ms equal-power
- `gain_other` → 0.28 over 1000ms
- air_bus unchanged
- others **never** 0

Disengage (second click, empty-scene click, other bird, Escape, focus leaving the scene): reverse ramp, same 1000ms.

Switching A→B: cross-ramp 1000ms, no dip to silence.

### 8.3 Chorus

Independent schedulers + answer hooks from §5.7. No shared LFO. Slight stereo by perch `x` (±0.3). Master compressor threshold high so two birds do not pump.

### 8.4 WebAudio fallback

If `window.AudioContext` missing, construction throws, or `resume()` denied after gesture:

- Stay silent.
- Force captions on (user can turn off).
- Do not fetch mp3/ogg “approximate calls.”
- Do not show a broken-speaker toast in the scene. If the accessibility panel is open, matter-of-fact: `Calls aren't available in this browser. Captions are on.`

### 8.5 Mute

If the user mutes the tab or sets OS mute, we cannot always know. Still run visual call poses + captions-if-enabled. Do not treat mute as neglect; presence is visual attention, not audio-on. (Do **not** use “mute the calls” as a negative drift input. The product brief mentions mute as something birds notice over weeks; v1 interpretation: if we can read `AudioContext` destination or a user “quiet aviary” control, feed a **weak** listen-in-negative only on `vocal_frequency` … **Rejected.** Monotonic expressive + no punishment. A future “quiet preference” must not reduce traits. Calls simply play quieter. **v1: no vocal-frequency penalty for mute.**)

### 8.6 Caption generation

From the realized motif descriptor, not a fixed map:

```
atoms=3, contour=up, amp=soft → “a soft three-note rise”
species=wren, repeated=true → “a low trill, paused, low trill again”
zone=back, atoms=1, amp=sharp → “a single sharp call from the back perch”
```

Place small text near the bird, fade with the phrase. AA contrast. Naturalist lowercase. Generated at the same time as the audio schedule so caption and sound match; in silence fallback, generate from the would-have-been phrase.

---

## 9. Accessibility surfaces

Accessibility ships in the same release train as the scene. Not a v1.1.

### 9.1 Screen-reader narration

One `aria-live="polite"` region, visually hidden, updated:

- idle: every 45s (jitter 30–60)
- priority: greeting, offer reaction, settle, arrival fly-in — as they happen
- never faster than 8s between updates
- interrupt policy: priority replaces the queued idle sentence; do not stack five utterances

Prose from `packages/prose/narration`, same voice as the notebook, built from snapshot (names, zones, solar, weather, one action). Example register: `a small grey bird is perched on the front rail, calling softly.`

Do **not** map ARIA to `mood=content` or `boldness=0.4`. Birds in the accessibility tree:

```
list “birds”
  button “Pip, warbler”
  button “Wren, wren”
```

State changes come through narration, not live-region spam on each pose.

### 9.2 Keyboard

- `Tab` / `Shift+Tab`: top-bar icons, then birds in reading order (front-to-back, left-to-right), then nothing else in the scene.
- `←/→/↑/↓` among birds when focus is in the scene.
- `Enter` / `Space`: toggle listen-in on focused bird.
- `Escape`: exit listen-in; if a panel is open, close panel first.
- Offer panel: arrows + Enter to choose.
- Settle: focus the bar icon, Enter.
- Focus ring: 2px token, dual-color (ink on day, cream on night) so it holds 3:1 against both palettes. Designer specifies; engineer does not ship a default blue ring.

### 9.3 Contrast and copy

All user-copy (bar tooltips if any, panels, auth, errors, captions, settings) WCAG AA. Scene itself has no body copy except captions. Test captions on midday and night backgrounds.

Auth/settings/errors: matter-of-fact. Notebook/narration/captions/offer verbs: naturalist.

### 9.4 Reduced-motion

See §7.7. Also disable CSS transitions on chrome longer than 200ms when reduced-motion is on, except the designed 1.2s pose cross-fades in-canvas.

### 9.5 Visit view a11y

Visitor gets the same narration and captions, no listen-in controls, no offer/settle. Keyboard can still move a **non-activating** highlight for “which bird am I hearing about,” but Enter does nothing. Do not imply the visitor can listen-in.

---

## 10. Performance budgets and observability

### 10.1 Budgets

| Budget | Gate |
|---|---|
| Initial JS (first paint path) | ≤ 2MB gz hard fail; **≤ 350KB gz** team target |
| Time to first bird | < 500ms mid-tier mobile / 4G synthetic (P75 RUM alarm at 800ms) |
| Idle FPS | 60 on reference 5-year laptop profile, 30-min session, 7 birds |
| Memory | no growth: 30m heap ≤ 2m heap + 8MB |
| Snapshot payload | < 8KB gz typical, < 24KB max |
| Tick / catch-up | p99 < 5s alarm; **SLO p99 < 80ms** for ≤ 14d catch-up |
| Magic-link send | p99 < 2s API time (email provider async) |

### 10.2 How we hit first bird < 500ms

1. Inlined snapshot on `GET /`.
2. `scene-core` split: Canvas + species paths for the two/N birds in the snapshot, no settings, no notebook, no worklet.
3. Preload only the scene chunk from the HTML.
4. Sky painted in CSS before JS.
5. Fonts: one small body face for chrome, optional; scene needs no webfont. If notebook uses a serif, load it when the panel opens.
6. No client-side A/B, no tag manager, no third-party analytics SDK.

### 10.3 What we measure

Allowlisted metrics only (`packages/telemetry`):

- `http_request_total{route,status}`
- `http_latency_ms{route}`
- `sim_tick_ms`, `sim_catchup_minutes`, `sim_catchup_ms`
- `snapshot_bytes`, `snapshot_build_ms`
- `rum_first_bird_ms` (no account id)
- `rum_frame_ms_p95` (session-aggregated, no account id)
- `audio_context_error_total{reason}`
- `magic_link_consumed_total`, `magic_link_expired_total`
- `visit_snapshot_denied_total`

Dimensions allowed: `route`, `status`, `reason`, `browser_family`.  
Dimensions forbidden: `account_id`, `email`, `bird_id`, `species_id`, `mood`, any trait, presence minutes, offer kind.

RUM page-load timings are similarly stripped. Synthetic browsers in 3–4 geos run the aviary on a schedule with a **dogfood account excluded from “per-bird” export** — synthetics use a dedicated aviary whose events are flagged `synthetic=true` and excluded from any future research temptation; still do not warehouse bird fields.

Logs: request id + account UUID allowed; never email; never event payload dumps.

### 10.4 What we deliberately do not measure

- Funnel “activation” beyond “snapshot 200.”
- DAU/streak-like dashboards in product.
- Average boldness across accounts.
- “Most listened-in species.”
- Visit leaderboards (do not compute them).

Error budget: page `sim_tick_ms` p99 > 5s for 10m → page. That is a ceiling alarm, not the SLO.

### 10.5 Browser support

Last two majors of Chrome, Safari, Firefox, Edge. Feature-detect Canvas2D, `visibilityState`, `crypto.subtle`. Fail closed to `/unsupported` with matter-of-fact copy. No polyfill jungle that blows the bundle.

---

## 11. Privacy and security (implementation)

- Email AES-GCM + HMAC lookup; pepper in a secret manager, not the image.
- Session and visit tokens: 256-bit random, only hashes stored, rotate nothing on each request (avoid write storms); revoke list checked every request.
- Magic link 15m, single consume, HTTPS-only.
- Export link 24h, single download.
- CSRF: SameSite cookies + `Origin` check on POST.
- Rate limits: magic link, events (60/min/session), snapshots (30/min/session), visit create (10/day/account).
- Visitors cannot read notebook, names-are-visible (host chose to share the scene including names), no export.
- Simulation DB network-isolated from any warehouse role. CI lint: no `SELECT` from `birds`/`aviary_events` outside `apps/api` and `apps/sim-worker`.
- Account deletion is the only bulk erase path.

---

## 12. Frontend module map (so the team does not invent a second product)

```
apps/web/src/scene/boot.ts          parse snapshot, first RAF
apps/web/src/scene/loop.ts          RAF, interpolation, particles
apps/web/src/scene/birds.ts         pose / bones / mood idle
apps/web/src/scene/layout.ts        zones, responsive x/y
apps/web/src/scene/solar.ts         local palette
apps/web/src/scene/weather.ts       rain/wind draw
apps/web/src/scene/reduced.ts       cross-fade renderer
apps/web/src/audio/worklet.ts
apps/web/src/audio/grammar.ts
apps/web/src/audio/mix.ts           listen-in ramps
apps/web/src/audio/captions.ts
apps/web/src/presence.ts            triple conjunction
apps/web/src/sync.ts                poll, queue, beacon
apps/web/src/chrome/topbar.ts
apps/web/src/chrome/offer.ts
apps/web/src/chrome/notebook.ts
apps/web/src/chrome/settle.ts
apps/web/src/a11y/narration.ts
apps/web/src/a11y/keyboard.ts
apps/web/src/system/*               sign-in, settings, visits, errors
```

Scene files must not import system toasts. System files must not import trait names.

---

## 13. Rollout

### 13.1 Engineering slices (build order)

Do not start visits or notebook chrome before the tick is deterministic. Suggested increments, each shippable to dogfood:

1. **Skeleton:** monorepo, protocol types, Postgres schema, magic-link sign-in, empty quiet field, unsupported page.
2. **Canonical aviary:** create two birds on first consume, snapshot+inline, Canvas first bird mid-pose, no audio.
3. **Tick core:** catch-up, mood, perch intent, expression_gain, presence pings with the real conjunction, drift harness green.
4. **Greeting + settle + offer micro-ticks** with server actions; client interpolation.
5. **Audio grammar + chorus + listen-in + captions + silence fallback.**
6. **Day/night, weather, ambient ornaments, top-bar fade.**
7. **Notebook + narration grammars, voice lint.**
8. **Reduced-motion renderer, keyboard, AA pass.**
9. **Multi-device tests, presence de-dupe, laptop+phone soak.**
10. **Visits, revoke, visit log, opt-in mail.**
11. **Export, soft/hard delete, session revoke, email change.**
12. **Perf CI, RUM allowlist, synthetic geos, age-gate arrivals.**

### 13.2 Bird-count ramp

Do not feature-flag “everyone gets seven.” Age gates are the ramp. Dogfood accounts may set `aviaries.created_at` backward **only in non-prod** to exercise arrivals. Prod support must not mint birds by hand; if a bird must be restored, reinsert the **same UUID** and last known traits from backup taken for that incident, never a new identity.

### 13.3 Launch shape

- Closed dogfood (team + a few trusted users) until: first-bird synthetic < 500ms p75, drift harness stable, no personality in client JSON (contract test), a11y keyboard path complete, reduced-motion designed path complete.
- v1 public: two starters, visits off by default, no blog copy that says “collect all seven.”
- Instrument from day one with the allowlist in §10.3. Do not add a product analytics plan later that reads the event log.

### 13.4 Ops

- On-call owns API 5xx, tick p99, email provider, Postgres.
- Playbook: if catch-up falls behind, snapshot still computes catch-up in-request; worker is optional warmth.
- If sim lock contention spikes (two devices + worker), drop worker first.

### 13.5 Content freeze

Species art, motif libraries, notebook templates, and name lists freeze one week before public v1 except bugfixes. Changing a species call interval set after users have learned a bird is an identity break — treat it like a data migration, not a CSS tweak.

---

## 14. Testing strategy

### 14.1 Must-pass invariants (CI)

1. Drift monotonic: 1e4 random event traces, no trait decreases.
2. Neglect 14d: traits frozen, gain may fall, greet p falls.
3. Regular 7d / 21d numeric windows in §5.2.
4. Catch-up ≡ stepped minutes (empty span fuzz).
5. Presence conjunction: each missing signal drops pings.
6. Dual-device pings de-dupe to one 30s bucket.
7. Clients posting `boldness` rejected; snapshot fixture has no trait keys.
8. Two realized calls with same bird+mood differ in timing or cents.
9. Visitor POST `/aviary/events` is 403; visitor presence does not change gain or traits.
10. Voice lint on all `packages/prose` strings.
11. Telemetry allowlist lint on metric names/labels.
12. Bundle size gate.
13. 30-min memory soak.
14. `prefers-reduced-motion` does not call the full-motion flyer.
15. First-paint path contains no spinner DOM.
16. Magic-link replay does not mint a second session.
17. Soft-delete 410 + undelete restores same bird ids and traits.
18. Age gate does not fire on visit count (test: 10k visits, aviary age 1 day → still 2 birds).

### 14.2 Playwright

- Sign-in (stub mail), first fly-in, greeting within 2s of snapshot, listen-in mix hook (gain values exposed on `window.__audioTest` in test builds only), offer cooldown, settle undo <5s and not >5s, notebook opens read-only, keyboard listen-in, visitor cannot offer, captions render on forced silent path.

Test builds may expose `__audioTest` and `__sceneTest`. Production builds strip them.

### 14.3 What “green unit tests” will not catch

- Presence defined as “tab open.”
- A welcome toast added “just for onboarding.”
- Personality leaked into ARIA.
- Recorded audio “temporarily.”
- Streak disguised as a notebook sentence.
- Last-write-wins on a new client field that happens to include a vector.

Review checklist for every PR that touches chrome or telemetry: walk the 13 invariants in §0.

---

## 15. Risks

### 15.1 Drift calibration

**Risk:** Too fast → Tamagotchi. Too slow → screensaver. Silent presence inflation → population-wide rush.

**Mitigations:** Triple-conjunction + 30s de-dupe + 4-minute activity window; harness in CI; `expression_gain` absorbs neglect instead of traits; no client trait writes; cooldown on offers; change `rate_week` only with harness diffs reviewed as a product change.

**Residual:** Real users sit still longer than 4 minutes. If greeting/drift feels dead, lengthen the window (6–8m) before loosening conjunction.

### 15.2 Sync correctness

**Risk:** Double application of catch-up, lost events, two micro-ticks clobbering `action`, phone overwriting name mid-offer.

**Mitigations:** Aviary row lock; idempotent event ids; `seq`; presence bucket unique index; names LWW only; actions replaced explicitly, not merged; property tests on the log.

**Residual:** Extremely long catch-up after a year offline. Collapse empty minutes; still cap in-request catch-up work at 50ms slices with a continuation if needed (rare). Prefer one-shot integrate.

### 15.3 Audio uncanniness

**Risk:** Motifs sound like ringtones; chorus phases into a machine; every warbler sounds the same; user hears an exact repeat.

**Mitigations:** Species interval sets + individual seed; mandatory jitter; chorus cap; no looped buffers; listen with humans weekly during slice 5; nightjar/nighthawk kept sparse.

**Residual:** WebAudio differences across Safari vs Chrome. Golden-test perceptual hashes are weak; rely on descriptor tests + device QA. If Safari formants break, prefer simpler oscillators over shipping oggs.

### 15.4 Accessibility regressions

**Risk:** Live-region flood; reduced-motion as `animation: none` empty scene; focus ring invisible at dusk; captions failing AA on rain overlays; shipping a11y “later.”

**Mitigations:** Designed reduced-motion renderer in the same slice as motion; cadence caps; dual-color focus; captions contrasted on a chip, not raw on foliage; launch gate includes keyboard + SR smoke.

**Residual:** Some SR + Canvas combinations expose a blank canvas. Mitigate with the parallel list of bird buttons and the live narration, not by putting the whole scene in a bitmap `alt`.

### 15.5 Aliveness leaks

**Risk:** Quiet-field overused; spinner “just this once”; greeting three canned variants; first frame T-pose because `t0` is `now`.

**Mitigations:** `motion_epoch` and past `t0` required in snapshot fixture tests; greeting must hash to >K distinct timing outcomes; visual review checklist for slice 2.

### 15.6 Privacy / PII spray

**Risk:** Email as a convenient key in Redis, logs, or visit URLs.

**Mitigations:** Tokens in URLs are random, not emails; lookup hashes only; lint for `@` in logs; review Redis key patterns.

### 15.7 Social gravity

**Risk:** “Just a public gallery,” visit badges, notify-on by default.

**Mitigations:** Defaults off; no badge code path; visit log buried in settings; refuse discovery endpoints even internally.

### 15.8 Perf vs procedural beauty

**Risk:** Plumage detail and particles blow the 60fps and 350KB targets.

**Mitigations:** Particle budget, path cache, drop ornaments first, species kits as compact path commands not SVG DOM.

### 15.9 Offer / greeting latency

**Risk:** Micro-tick under lock makes POST slow; birds feel unresponsive.

**Mitigations:** Lock scope is one aviary; function is pure and tiny; return snapshot in the POST; client shows the gesture immediately.

### 15.10 Identity continuity in migrations

**Risk:** A species redesign “replaces” birds; a bad migration reseeds traits.

**Mitigations:** `birds.id` immutable; migrations that touch trait columns require a two-person review and a restore drill; never `DELETE/INSERT` birds to change species art.

---

## 16. Team working agreements

- Copy in `apps/web/src/scene` and `packages/prose`: naturalist, lowercase, no second person.
- Copy in `apps/web/src/system` and API errors: matter-of-fact.
- If a new surface is about money, identity, errors, or settings, it is a system surface.
- Do not add a toast component to the scene package.
- Do not add a `stats` route.
- Do not add recorded audio “for QA.”
- When unsure whether a feature is on-tone, check `non_goals.md` and §0 of this plan. If it teaches the user to manage a number, it is out.

---

## 17. Acceptance snapshot (v1 done when)

A new user can magic-link in, meet two already-moving birds within 500ms on a mid-tier phone, be noticed (not announced), sit without clicking and still accumulate honest presence, listen in with a slow mix, offer a seed and see a mood-and-curiosity-shaped reaction, settle or just close the tab without guilt, read a sparse naturalist notebook, use the product with keyboard, screen reader, reduced motion, or silence+captions, sign in on a second device and see the same birds in the same moods with the same names and the same hidden history, invite one friend to look and not to touch, export or delete their account, and never encounter a score, streak, hunger bar, welcome toast, or recorded chirp.

That is the whole v1. Stop there.
