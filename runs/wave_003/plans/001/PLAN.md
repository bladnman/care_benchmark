# Pocket Aviary — v1 Implementation Plan

This plan turns the Pocket Aviary PRD into something an engineering team can build. It doesn't restate the spec. It makes the decisions the spec leaves open, names the invariants every workstream has to protect, and orders the work so the most important properties are built first and are the hardest to erode later: felt aliveness, honest presence, monotonic drift, a server-authoritative simulation, and first-class accessibility.

Where the PRD is ambiguous or contradicts itself, §15 records the call made and the reason for it.

---

## 0. Invariants (the non-negotiables every PR is checked against)

These are enforced in code, schema, CI, and review. They aren't just guidelines.

| # | Invariant | Where it is enforced |
|---|-----------|----------------------|
| I1 | Only the server simulation tick writes personality vectors. No client, and no API handler, mutates them on any code path. | Postgres role grants (`api_role` has no `UPDATE` on `bird_personality`). Code ownership. An integration test tries the write as `api_role` and expects it to fail. |
| I2 | Personality drift is additive, non-negative, and bounded by a per-bird ceiling. No trait ever decreases. | Property tests on the drift function. A runtime guard in the tick clamps any negative delta to 0 and increments `drift_negative_clamped_total`, which must stay at 0 and alarms otherwise. |
| I3 | Presence = `visibilityState==='visible'` ∧ `document.hasFocus()` ∧ (pointer/key activity within window W). Nothing else counts. | A single `PresenceMonitor` module with an exhaustive truth-table test. Server clamps (§6.4). |
| I4 | Bird identity is permanent. `bird.id`, voiceprint, and appearance seed never change. Birds are never regenerated, swapped, or re-seeded. | Immutable columns (trigger rejects UPDATE). No `DELETE` except the hard-delete job. A migration continuity test in CI. |
| I5 | Personality values are never sent to any client and never rendered, labelled, logged, or put in telemetry. The single, documented exception is the account data export (§15, D6). | The snapshot schema has no trait fields. A serializer allowlist. A log/telemetry scrubber. A contract test that greps API responses. |
| I6 | No announcement surfaces: no toasts, banners, welcome text, streaks, counters, badges, confetti, or push/email notifications about the aviary (the opt-in visit notification is the only exception). | The design system has **no toast/snackbar component**. A copy lint (banned lexicon). Review checklist. |
| I7 | Calls are procedural and synthesized client-side. No recorded audio ships, including as a fallback. | Build check: no audio file extensions or media MIME types in the bundle or CDN manifest. |
| I8 | Email appears in exactly one place (the encrypted account record, plus encrypted visitor email on invites). Every other reference uses the synthetic account UUID. | Schema review. A PII scanner on logs and telemetry in staging and prod. A lint on metric label names. |
| I9 | Per-bird and per-account interaction state never enters aggregate telemetry, analytics, or ML. | The telemetry client only accepts registered metrics with enumerated low-cardinality labels. The warehouse has no network path to the simulation DB. |
| I10 | The aviary's first rendered frame is the aviary in motion. No spinner, no entry animation, no fade-from-static. | Visual regression on first frame. A lint bans spinner components. A `first-bird` RUM mark. |
| I11 | Accessibility surfaces (narration, captions, reduced-motion, keyboard) ship with v1 and gate every release. | Release checklist. Accessibility CI plus a manual screen-reader pass per release. |

---

## 1. Scope

### 1.1 In v1

- Browser-only product. Supports the last two major versions of Chrome, Safari, Firefox, and Edge, on desktop and mobile.
- Email magic-link sign-in, per-device revocable sessions, verified email change, account export, and account deletion (30-day soft delete, then hard delete).
- One canonical aviary per account. Two starter birds chosen by the system from a pool of six species, named by the user. New birds become available by **aviary age** only, with a hard cap of 7.
- A server-side simulation tick about once a minute for every aviary. It covers personality drift, mood, perch choice, weather, bird-to-bird social events, and notebook observation.
- The client renders snapshots with interpolation. There is no client simulation.
- Interactions: presence (idle watching), return-greeting, listen-in, offer (seed, song fragment, still pool), settle (with a 5-second undo), and the field notebook (read-only, auto-generated, sparse).
- Single horizontal responsive scene: three perch zones, a day/night cycle on the user's local time, rare ambient weather, ambient leaf and feather drift, a thin fading top bar.
- Procedural WebAudio call synthesis, chorus, the listen-in mix, and a silent-with-captions fallback.
- Accessibility: naturalist screen-reader narration, call captions, a designed reduced-motion mode, full keyboard navigation, WCAG AA contrast on all copy.
- Visits: per-invite, read-only, revocable, expire after 30 days if unused, off by default. A silent visit log. An opt-in visit notification that is off by default.
- Aggregate-only operational telemetry, synthetic monitoring, and RUM.

### 1.2 Explicitly not in v1 (each is refused in code, not just deferred)

- Native apps. We also don't design protocols around native constraints.
- Any gamification: streaks, counters, visit calendars, achievements, badges, levels, XP, scores, "birds adopted: N". None of these exist as a data model, API, or setting. We also don't compute the underlying per-user visit statistics anywhere they could be surfaced.
- Tamagotchi mechanics: hunger, death, distress, decaying happiness meters. Neglect produces quietness and nothing else.
- Social-network surfaces: profiles, follows, feeds, discovery, comments, chat, avatars, co-presence, leaderboards, and "show-off" rendering.
- Push notifications and aviary-related email. The only exception is the host's opt-in visit notification.
- Payments, shared or multi-aviary accounts, customizable scenes, user-controlled bird placement, species catalogs, SSO, and passwords.
- Any UI that exposes personality numbers, at any tier, including internal "debug" views reachable from a user account.
- Recorded audio of any kind.
- Compatibility paths for older browsers. They get a matter-of-fact unsupported-browser page.

---

## 2. Architecture

### 2.1 Shape

```
                    ┌──────────────────────────── Browser ─────────────────────────────┐
                    │  Boot shell (inline) → Scene engine (Canvas2D) → Chrome (Preact) │
                    │  PresenceMonitor · Outbox (IndexedDB) · SyncClient               │
                    │  Audio engine (AudioWorklet synth + mixer) · Prose engine (shared)│
                    │  Service Worker (shell/asset cache, no state authority)          │
                    └────────────▲──────────────────────────────┬──────────────────────┘
                                 │ HTML+inline snapshot / JSON  │ events (append-only)
                    ┌────────────┴──────────── Edge (CDN + edge fn) ─────────────────────┐
                    │ SSR shell, session check (cached ≤60s), inline snapshot from cache  │
                    └────────────▲──────────────────────────────┬──────────────────────┘
                                 │                              │
      ┌──────────────────────────┴──────┐        ┌──────────────▼──────────────┐
      │ Snapshot cache (Redis, per-region│◄──────┤ API service (stateless, TS) │── Email provider
      │ replicas; latest per aviary)     │       │ auth · events · offers ·     │   (magic links,
      └──────────────▲───────────────────┘       │ greetings · notebook · visits│    exports, visit
                     │ publish after commit      └──────────────┬──────────────┘    notification)
      ┌──────────────┴───────────────────┐                      │ INSERT events only
      │ Tick workers (sharded, leased)   │◄─────────────────────┤
      │ sim-core (pure, deterministic)   │   SELECT events       ▼
      │ sole writer of aviary state      ├──────────────► Postgres (primary + replicas)
      └──────────────────────────────────┘                accounts · birds · personality ·
                                                          bird_state · events · notebook · visits
      ┌──────────────────────────────────┐
      │ Jobs: notebook writer, export,   │   Telemetry pipeline (metrics/logs/RUM) is a
      │ hard-delete, invite expiry,      │   separate system with NO access to Postgres
      │ integrity checks                 │   simulation schemas.
      └──────────────────────────────────┘
```

### 2.2 Components and ownership

| Component | Tech | Owns | Never does |
|-----------|------|------|------------|
| `sim-core` | TypeScript library, pure functions, seeded PRNG, no I/O | Drift, mood, perch selection, weather, social events, greeting planner, offer-reaction planner, observation extraction | Touch the DB or network |
| Tick workers | Node.js services running `sim-core` | Advancing every aviary once a minute, publishing snapshots | Accept client input directly |
| API service | Node.js (TypeScript), stateless, horizontally scaled | Auth, sessions, event ingest, greeting and offer plans (via `sim-core` planners, read-only on state), notebook reads, settings, visits, exports | Write personality or mood |
| Edge function | CDN edge runtime | Serving the HTML shell with inlined critical JS/CSS and the inlined snapshot | Hold state |
| Postgres | Managed Postgres 16, one primary region, 2 read replicas | Canonical state | Replicate to the analytics warehouse |
| Redis | Managed, primary plus regional read replicas | Latest snapshot per aviary, rate-limit buckets, short-lived session cache | Hold the only copy of anything canonical |
| Object storage | S3-compatible | Export files (signed URLs, 7-day expiry) | Hold anything else user-derived |
| `prose` package | TypeScript, shared by server and client | Notebook entries, narration, captions, greeting and reaction narration strings | Use a third-party LLM or anything that leaves our boundary |
| `species` package | Data plus TypeScript | Species definitions: silhouette rig, palette ranges, motif libraries, caption vocabulary | Change existing species in place (it's versioned, §5.9) |

Language choice: TypeScript everywhere. The deciding reason is shared code. `sim-core`, lighting curves, prose, and species data are consumed by both server and client, and one implementation prevents drift between them. The tick is cheap arithmetic, so Node throughput is enough (§5.1 sizing). If profiling says otherwise, `sim-core` can be ported to Rust/WASM behind the same interface.

### 2.3 Client/server split (the render-pipeline boundary)

The boundary is the **snapshot**. The server decides *what is true*: which bird is on which perch, mood, scheduled flights, weather, greeting and reaction choreography, and derived call and motion parameters. The client decides *how it looks and sounds*: rig animation, micro-motion, per-call synthesis and variation, captions, narration phrasing, lighting grade, and leaves and feathers.

Rules:

- The client never advances simulation time and never infers mood or personality. It interpolates and embellishes within server-given parameters.
- Every server-side choice that another device must agree on goes in the snapshot. That includes perch changes, weather, social events such as alarms and chorus windows, greetings, and offer reactions. Anything that only needs to look alive locally, like micro-motion, exact call timing inside a rate, and leaves, stays client-side.
- Derived parameters are sent instead of traits. Examples: `callRatePerMin`, `chorusJoinP`, `responseP`, `scanRate`, `preenRate`, `fluff`, `plumageColors[]`, `perch`. Each is a quantized blend of mood, personality, attunement, time of day, and weather, so no field maps one-to-one onto a trait (I5).

### 2.4 Environments

`local` (docker-compose Postgres/Redis, a time-warp tick), `ci`, `staging` (prod-like, synthetic accounts only), and `prod`. Staging and CI support a **virtual clock** for the sim so the calibration harness and e2e tests can compress weeks into minutes. In prod the virtual clock is compiled out of the tick worker.

---

## 3. Data model

Postgres. All primary keys are UUIDv7 unless noted. Every table carrying account data has `account_id uuid` and an FK with `ON DELETE CASCADE` from `accounts`, so the hard-delete job is a single delete plus storage cleanup.

### 3.1 Accounts and auth

```sql
accounts (
  id                 uuid pk,                 -- synthetic; the ONLY identifier used anywhere
  email_ciphertext   bytea not null,          -- AES-256-GCM with per-account DEK (envelope-encrypted via KMS)
  email_lookup_hmac  bytea unique not null,   -- HMAC-SHA256(normalized email, server secret); used ONLY by auth lookup
  dek_wrapped        bytea not null,          -- per-account data key; destroyed on hard delete (crypto-shred)
  created_at         timestamptz not null,
  timezone           text not null,           -- IANA; canonical aviary tz (see §6.6)
  timezone_candidate text, timezone_candidate_since timestamptz,
  status             text not null check (status in ('active','pending_deletion')),
  deletion_requested_at timestamptz,
  settings           jsonb not null default '{}'  -- visit_notifications=false, captions, narration, reduced_motion_pref, shortcuts_enabled
)
magic_links (id, account_id null, email_ciphertext null /* signup */, email_lookup_hmac,
             token_hash bytea unique, purpose text /* sign_in | email_change */,
             created_at, expires_at /* +15 min */, consumed_at null)
sessions (id, account_id, token_hash unique, device_label text /* coarse: "Safari on iPhone" */,
          created_at, last_seen_at /* day granularity */, revoked_at null, expires_at)
email_changes (id, account_id, new_email_ciphertext, token_hash, expires_at, verified_at null)
```

Notes:
- `email_lookup_hmac` is derived from email, but it is used only in the auth lookup index. It never appears in logs, never keys a partition, and is never joined outside the auth module. It's needed because the email is encrypted and we still have to find the account at sign-in. A schema lint rejects any other column or FK referencing it.
- IP addresses are never persisted. Rate limiting uses Redis buckets keyed by `HMAC(ip)` with a 1-hour TTL.

### 3.2 Aviary and birds

```sql
aviaries (
  id uuid pk, account_id uuid unique not null,
  created_at timestamptz not null,             -- drives new-bird availability
  tick_seq bigint not null default 0,          -- fencing token for single-writer ticks
  last_tick_at timestamptz, next_tick_due timestamptz,
  event_cursor bigint not null default 0,      -- last consumed per-aviary event seq
  rng_state bytea not null,                    -- sim PRNG state (deterministic replay in tests)
  weather jsonb not null,                      -- {kind: none|rain|wind, started_at, ends_at, intensity, next_rain_at, next_wind_at}
  social jsonb not null,                       -- recent alarm/chorus windows, cooldowns
  presence jsonb not null,                     -- {last_presence_end_at, present_now, present_devices, today_presence_s, local_day}
  schedule jsonb not null,                     -- choreography horizon (next ~90s of scheduled actions)
  state_version bigint not null                -- increments on every committed tick or overlay change
)
aviary_event_heads (aviary_id pk, next_seq bigint not null)   -- serialized per-aviary seq allocator (§6.2)

birds (
  id uuid pk,                      -- IMMUTABLE identity (I4)
  aviary_id uuid not null,
  species_id text not null, species_version int not null,       -- pinned; species updates never re-skin existing birds
  voiceprint jsonb not null,       -- IMMUTABLE: base pitch, timbre, signature syllable, motif preferences
  appearance_seed bigint not null, -- IMMUTABLE: markings, proportions within species ranges
  name text not null, name_version int not null,
  status text not null check (status in ('resident','wanderer')),
  arrived_at timestamptz not null, adopted_at timestamptz null
)
bird_personality (                 -- WRITTEN ONLY BY tick_role (I1)
  bird_id uuid pk,
  boldness float8, warmth float8, vocal float8, plumage float8, curiosity float8,  -- [0,1]
  ceil_boldness float8, ceil_warmth float8, ceil_vocal float8, ceil_plumage float8, ceil_curiosity float8, -- immutable, per-bird
  drive jsonb not null,            -- low-pass drive accumulators per trait (§5.3)
  updated_tick bigint not null
)
bird_state (                       -- WRITTEN ONLY BY tick_role
  bird_id uuid pk,
  mood text check (mood in ('alert','curious','content','wary','drowsy','settled')),
  mood_since timestamptz, mood_min_until timestamptz,
  attunement float8,               -- medium-timescale "recently observed" (§5.4); not personality; never exposed
  perch_zone text check (perch_zone in ('front','middle','back')), perch_slot smallint,
  activity text, activity_until timestamptz,
  offer_cooldown_until timestamptz,
  last_greeted_at timestamptz,
  listen_in_s_today float8, offers_today smallint, local_day date
)
bird_personality_daily (bird_id, day date, vector jsonb, pk(bird_id, day))  -- backup/integrity only, 90-day retention, never read by product
```

### 3.3 Events (the append-only log)

```sql
interaction_events (
  aviary_id uuid, seq bigint,                       -- per-aviary, gapless, commit-ordered (§6.2)
  client_event_id uuid,                             -- idempotency key from client outbox
  session_id uuid null,                             -- device session; null for server-generated events
  type text,   -- presence_interval | listen_in_start | listen_in_end | offer | settle | greeting (server) | tz_report
  bird_id uuid null,
  payload jsonb,                                    -- small, typed per event type; validated by zod schema
  client_ts timestamptz, received_at timestamptz,
  consumed_tick bigint null,
  primary key (aviary_id, seq),
  unique (aviary_id, client_event_id)
) partition by range (received_at);   -- daily partitions
```

Retention: events are deleted **14 days after consumption**. They exist to drive the user's own simulation and to support short-horizon incident debugging, and nothing else. Personality is never recomputed from them in the product path (§5.3.5).

### 3.4 Notebook

```sql
notebook_entries (
  id uuid pk, aviary_id uuid, written_at timestamptz, local_date date,
  text text not null,                    -- final rendered prose (immutable)
  observation_kind text, bird_ids uuid[],
  generator_version text, facts jsonb    -- the observation facts used (for QA of the prose engine)
)
observation_buffer (aviary_id, id, observed_at, kind, bird_ids, facts jsonb, salience float4, consumed bool)
```

The API role has `SELECT` only on `notebook_entries`. Only the notebook writer job can `INSERT`. Nothing has `UPDATE` or `DELETE` except the hard-delete job.

### 3.5 Visits

```sql
visit_invites (id, host_account_id, visitor_email_ciphertext, token_hash unique,
               created_at, expires_at /* +30d */, bound_at null, bound_client_hash null,
               revoked_at null, suspended bool default false)
visit_sessions (id, invite_id, token_hash, started_at, last_seen_at, ended_at null)  -- duration = last_seen - started
```

### 3.6 Jobs and settings

`export_jobs(id, account_id, requested_at, status, object_key, expires_at)`. `deletion_jobs` are derived from `accounts.status`. Per-device settings (mute and a device reduced-motion override) live in `localStorage`, not on the server. See §15 D9.

---

## 4. API surface

Everything is JSON over HTTPS (HTTP/2 and HTTP/3). Request bodies are validated with shared zod schemas. Errors use a small machine code plus matter-of-fact copy that lives in the client, not in the response. Auth uses an HttpOnly, Secure, SameSite=Lax session cookie (opaque 256-bit token, hashed at rest). Visitor auth is a separate cookie scoped to `/visit` with a visitor-only permission set.

### 4.1 Page and bootstrap

| Route | Behavior |
|-------|----------|
| `GET /` | Edge function. With a valid session it returns the HTML shell with inline critical CSS, the inline boot script, and an inline `<script type="application/json" id="snap">` containing the snapshot. Without one it returns the sign-in page, which is a system surface in matter-of-fact voice. |
| `GET /sw.js` | Service worker. It caches the shell, JS chunks, and species data. It never caches snapshots for longer than 2 minutes and never authors state. |

### 4.2 Auth and account (system surfaces)

| Endpoint | Notes |
|----------|-------|
| `POST /auth/link {email}` | Always returns `202`, whether or not the email exists (no enumeration). Rate limited per `email_lookup_hmac` (5 per 15 minutes, 20 per day) and per IP bucket. Creates a `magic_links` row with a 15-minute expiry. For an unknown email, the account is created **on consumption**, not on request. |
| `GET /auth/verify?t=` | Returns a page that auto-POSTs the token with JS and falls back to a "Continue" button. It never consumes on GET, so email-scanner prefetches can't burn links. |
| `POST /auth/verify {token}` | Checks the hash, expiry, and `consumed_at`. It consumes atomically (`UPDATE … WHERE consumed_at IS NULL RETURNING`). On success it issues a session and sets the cookie. Replayed or expired tokens return `401 link_invalid`, and the client shows "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| `POST /auth/signout` | Revokes the current session. |
| `GET /account/sessions`, `DELETE /account/sessions/:id` | Device list with coarse label and last-active day. Revocation takes effect within 60 seconds (the edge session cache TTL). |
| `POST /account/email {new_email}` → `POST /account/email/verify {token}` | The old email keeps working until the new one verifies. |
| `POST /account/export` | Queues a job. A signed-URL link is emailed to the verified address. |
| `POST /account/delete`, `POST /account/restore` | Soft delete and restore. While `pending_deletion`, every signed-in page shows a system banner: "Your account is scheduled for deletion on {date}. [I changed my mind]". This is a system surface, not an aviary announcement. |
| `PATCH /account/settings` | Accessibility, visit notification, and shortcut settings. |
| `PATCH /v1/birds/:id {name, if_name_version}` | Rename. Names aren't personality, so API-side writes are allowed, with optimistic concurrency. |

### 4.3 Aviary state and events (product surfaces)

**`GET /v1/aviary/snapshot?context=session_start|return|keepalive&hidden_ms=`**
Returns the snapshot (§4.5). `If-None-Match: <state_version>` returns `304`. When `context` is `session_start` or `return`, the server may attach a `greeting` plan (§7.3). That request also writes a server-generated `greeting` event so the notebook can observe "pip greeted first".

**`POST /v1/aviary/sync`**. This is the workhorse, called about every 60 seconds while the tab is visible and on key transitions.
```json
{ "known_version": 18231,
  "events": [ {"client_event_id":"…","type":"presence_interval","start":"…","end":"…","audible":true},
              {"client_event_id":"…","type":"listen_in_end","bird_id":"…","start":"…","end":"…"} ] }
```
Response: `{ "acked": ["…"], "snapshot": {…} | null }`. Events are inserted in one transaction per request (§6.2). It is idempotent on `client_event_id`.

**`POST /v1/aviary/offer {client_event_id, kind: "seed"|"fragment"|"pool", fragment_id?, near_bird_id?}`**
It inserts the `offer` event and synchronously runs `sim-core.planOfferReaction(currentState, offer, seed)`. The planner is read-only on canonical state. It returns `{ reaction: Choreography }`: which birds approach, drink, bathe, join the fragment, go quiet, or glance, along with timings. The reaction is also written as a short-lived **overlay** (Redis, TTL 3 minutes, version-bumped) so every device and visitor pulling a snapshot sees the same reaction. Mood and drift consequences are applied by the next tick from the event. Birds in cooldown still appear in the choreography (they glance or ignore), but the event is flagged `effective=false` for them. There is never a cooldown error or a timer in the UI.

**`GET /v1/notebook?before=<cursor>&limit=20`**. Pages back without limit. Entries never expire.

**`POST /v1/birds/adopt {wanderer_id, name}`**. Promotes a wanderer to resident through a `SECURITY DEFINER` DB function. The function creates the personality row, which is the only non-tick insert path into `bird_personality`, and never updates it.

### 4.4 Visits

| Endpoint | Notes |
|----------|-------|
| `POST /v1/visits/invites {email}` | Host only. Creates an invite (30-day expiry if unused) and emails the visitor a one-time link. The email template is matter-of-fact and contains no aviary data. |
| `GET /v1/visits` | Visit log (visitor email, date, approximate duration, newest first) plus outstanding invites. Emails are decrypted for the host only, at read time. |
| `DELETE /v1/visits/invites/:id` | Immediate revocation: sets `revoked_at` and ends all `visit_sessions`. Returns no confirmation surface beyond the list updating. |
| `GET /visit/:token` → `POST /visit/bind` | The first use binds the invite to that browser (a visitor cookie holds a separate `visit_session` token). The raw link is then spent. Opening the spent link on another browser shows the matter-of-fact unavailable page. |
| `GET /v1/visit/snapshot` | The same snapshot builder in `visitor` mode (§4.5). If the invite is revoked, expired, or suspended, or the host is pending deletion, it returns `410 {code}` and the client shows "This visit is no longer available." Updates `visit_sessions.last_seen_at`. |

The visitor permission set is snapshot reads only. Visitor tokens are rejected by `/sync`, `/offer`, notebook, settings, and every other write route. The rejection is enforced in middleware and covered by tests. Visitor clients also don't instantiate `PresenceMonitor` or the outbox at all.

The visit notification, if the host has opted in, is one email per visit session start. It is debounced to at most one per invite per 6 hours and reads "{visitor email} visited your aviary." in matter-of-fact voice.

### 4.5 Snapshot schema (render-ready, trait-free)

Target size: under 6 KB gzipped for 7 birds.

```ts
type Snapshot = {
  v: number;                 // state_version
  serverTime: number;        // epoch ms at send (client estimates offset)
  tickIntervalMs: number;
  mode: 'host' | 'visitor';
  aviary: {
    tz: string;              // canonical tz; lighting computed client-side from this + clock (shared fn)
    weather: { kind: 'none'|'rain'|'wind'; startedAt?: number; endsAt?: number; intensity?: number };
    ageBand: 'new'|'settled'; // only used to pick empty-state fly-in; no counts
  };
  birds: Array<{
    id: string; name: string; species: string; speciesVersion: number;
    status: 'resident'|'wanderer';
    look: { seed: number; colors: string[] };          // plumage already resolved to render colors
    voice: { seed: number; motifWeights: number[] };   // voiceprint render params (immutable)
    mood: MoodName;                                     // needed for motion/call styling; never displayed as a label
    perch: { zone: 'front'|'middle'|'back'; slot: number };
    motion: { scanRate: number; preenRate: number; fluff: number; tiltBias: number; restlessness: number };
    call:   { ratePerMin: number; chorusJoinP: number; responseP: number; intensity: number; register: number };
    activity?: { kind: string; since: number; until: number };
  }>;
  schedule: Array<{ id: string; at: number; kind: 'fly'|'hop'|'alarm'|'chorus'|'sleep'|'wake'; birds: string[]; to?: {zone,slot}; dur?: number }>;
  overlays: Array<Choreography>;   // active greeting / offer reactions
  greeting?: Choreography;         // only on session_start/return for host mode
};
```

Visitor mode drops `greeting`. It keeps everything else "exactly as it is", including names, because narration uses them (§15 D10). Visitor mode never includes notebook, settings, or account data.

---

## 5. Simulation engine

### 5.1 Tick scheduling and execution

- **Cadence:** each aviary ticks every 60 s. That's configurable from 30 to 120 s and calibrated in dogfood; the setting is expected to stay at 60 s. Each aviary gets a fixed phase `hash(aviary_id) mod 60s` so load spreads evenly across the minute.
- **Sharding:** 4,096 logical shards by `hash(aviary_id)`. Worker processes hold shard leases in a Postgres `shard_leases` table (TTL 30 s, renewed every 10 s). Rebalancing happens automatically.
- **Per-aviary tick transaction:**
  1. `SELECT … FROM aviaries WHERE id=$1 AND tick_seq=$expected FOR UPDATE`. The `tick_seq` check is the fencing token, so a stalled worker that lost its lease can't commit.
  2. Read events with `seq > event_cursor` in `seq` order. Seqs are gapless and commit-ordered, as explained in §6.2.
  3. Run `sim-core.step(state, events, now, dt, rng)`. It is pure and deterministic given the PRNG state.
  4. Write `bird_personality` (additive deltas only), `bird_state`, the aviary fields, the observation buffer, `tick_seq+1`, `event_cursor`, and `state_version+1`. Commit.
  5. After commit, build the snapshot and write it to Redis `snap:{aviary_id}` with the version. Redis is a cache, and the API can rebuild the snapshot from the DB if it's missing.
- **Batching:** workers process a shard's due aviaries in batches of about 200 with one DB round-trip per phase (bulk load, compute, bulk upsert in a single transaction per batch, with fencing checked per row). A failed row falls back to the single-aviary path.
- **Catch-up:** `dt` is the actual elapsed time. Every rule in `sim-core` is dt-aware: exact exponentials for low-pass filters, and hazard-based mood sampling `p = 1 − e^{−h·dt}`. When `dt > 5 min` (after an outage), the step is subdivided into 5-minute sub-steps, up to 24 hours. Past 24 hours, a coarse "overnight" step handles mood (time-of-day prior) and drift (closed-form decay of drive into traits). The aviary always comes back correct, just coarser.
- **Sizing:** the tick is about 5–20 µs of arithmetic per bird. Wall time is dominated by the DB. At 100k aviaries that's roughly 1,700 aviaries/s, which fits one primary with batching. At 1M it's about 17k/s, which would need partitioning the simulation schema across 4–8 Postgres shards keyed on `aviary_id`. The schema is designed for this: every query is aviary-scoped and there are no cross-aviary joins. That upgrade isn't needed for v1 launch.
- **SLOs:** tick latency (scheduled time → commit) p99 < 5 s pages on-call, per the PRD. Tick lag (aviaries not ticked for more than 3 minutes) > 0.1% alerts. Tick errors retry with backoff, and a poison aviary goes to quarantine after 5 failures. Quarantine only alerts. It never resets state.

### 5.2 Personality representation

Five traits in [0, 1]: boldness, warmth, vocal, plumage, curiosity.

- **Seeds:** species baseline plus individual jitter, drawn in the range 0.20–0.45 so there's headroom for expression. The seeds define the bird's starting character, for example a species with naturally higher boldness.
- **Per-bird ceilings:** `ceil_*` ∈ [0.70, 0.95], fixed at adoption from species plus individual variance. Ceilings keep birds distinct as they mature. Without them, every well-loved aviary would converge on identical maxed-out birds, which is the homogenization risk in §14.

### 5.3 Drift function

Drift is a two-stage low-pass filter with headroom saturation. It is monotonic by construction.

**Stage 1: signals.** At each tick, for each bird b and trait k, compute a non-negative instantaneous signal `s_bk` from events consumed this tick:

| Input | Contribution (per unit) | Traits | Gating |
|-------|------------------------|--------|--------|
| Presence seconds (account-level union across devices, §6.4) | `w_p` per second with daily soft cap: effective seconds `P_eff = C·(1 − e^{−P/C})`, C = 45 min | all five, weighted by species sensitivity. `vocal` only accrues from **audible** presence (not muted, audio running) | Visitors: never. Background, unfocused, or inactive time: never (I3). |
| Listen-in seconds on bird b (only while presence holds) | `w_l = 2.5·w_p` per second; per-bird daily soft cap 20 min | warmth, vocal of b | Auto-ends on presence loss (§7.4) |
| Offer accepted by b (effective, not in cooldown) | `w_oa` impulse | curiosity of b (+ small warmth) | Per-bird cooldown 4 min (calibrate 3–6) |
| Offer placed near b (effective) | `w_on` impulse (≈ 0.3·w_oa) | boldness of b | Same cooldown |
| Settle | 0 | none | Only closes the presence window and sends a mood nudge |

**Stage 2: drive.** `drive_bk` is an exponential moving average of `s_bk` with τ_drive = 3 days. Because it decays slowly, drift continues after the user leaves, based on input from before they left. That matches the PRD's statement that personality "drifts during the user's absence based on inputs from before they left".

**Stage 3: integration.** For every trait:

```
Δx = k_k · drive_bk · dt · ((c_k − x) / c_k)^γ      with γ ≈ 1.5
Δx = clamp(Δx, 0, Δmax_per_tick)                     // never negative (I2); Δmax_per_tick ≈ 2e-4
x  = min(x + Δx, c_k)
```

- **Headroom saturation** `(c−x)/c` makes early drift feel responsive and later drift slower. A year-old bird keeps maturing but doesn't cap out.
- **The negative clamp** exists because of I2. `drive` and `s` are non-negative by construction, and the clamp is belt-and-braces. Any clamp hit increments an aggregate counter that must be zero.
- **Plumage** renders as saturation and feather-detail level. It only ever increases.

**Calibration targets.** These are CI assertions in the harness (§5.10). Values are on the [0, 1] scale.

| Persona (virtual time) | 1 day | 7 days | 21 days | 84 days |
|------------------------|-------|--------|---------|---------|
| Regular: 5 visits/week × 12 min present, 1 listen-in × 3 min, 1 offer | Δ ≤ 0.006 per trait (**below the JND of 0.03**) | mean Δ over traits ∈ [0.015, 0.035] (**instrument-detectable**) | mean Δ ∈ [0.06, 0.10]; at least 2 traits ≥ 0.06 (**user-visible**) | mean Δ ≤ 0.30 |
| Heavy: 7 visits/week × 2 h present | ≤ 0.008 | ≤ 1.6× regular | ≤ 1.6× regular | — |
| Light: 1 visit/week × 10 min | — | > 0 | ≥ 0.015 | — |
| Tab open 48 h in background or unfocused, or no activity | 0 | 0 | — | — |
| Two devices present at the same time for 30 min | same as one device for 30 min | — | — | — |
| Regular for 3 weeks, then absent 2 weeks | — | — | never decreases; greeting frequency drops (§5.4) | — |
| Any single session, however long | Δ ≤ 0.008 per trait | — | — | — |

The perceptual mapping is defined in the render spec so that a change of 0.06 in any trait is a noticeable change in its visible expression:
- boldness: front-perch dwell share moves about 10 percentage points, and greeting order shifts
- warmth: greet-first probability and call-back rate move about 12%
- vocal: unobserved call rate moves about 15%
- plumage: saturation moves about 7% and detail moves one step
- curiosity: offer-approach latency moves about 20%

The **JND of 0.03** is validated with the internal perceptual panel (§12.3).

**Constants** (`w_p`, `w_l`, `w_oa`, `w_on`, `k_k`, species sensitivities) live in a versioned `drift_params` config that is part of the sim release. Changing them requires re-running the harness and getting sign-off from the drift owner.

**Persistence.** The vector is stored and updated by deltas, never recomputed from the log. `bird_personality_daily` keeps a daily copy for disaster recovery and integrity checks only.

**Repairs.** If a drift bug is found in prod, we pause drift application with the kill switch `drift_apply=false`. Drive keeps accumulating, and mood and everything else keep running. We then fix forward. We never write lower values, because that would itself be negative drift. If a bug inflated values, we accept the overshoot and slow future accumulation for the affected cohort by scaling `k` for a bounded period, with documented reasoning. Restoring from `bird_personality_daily` is reserved for data-loss incidents, where rows were lost or corrupted.

### 5.4 Attunement: how neglect produces quietness without negative drift

The PRD says neglected birds are "greeting less often because less often is what's been observed", while also requiring that traits never go down. Those two statements force a separate state. Each bird gets an **attunement** value `a ∈ [0.2, 1]` that is not part of personality:

- It rises with presence, listen-in, and effective offers. Its time constant is hours.
- In absence it decays toward its floor of 0.2 with a half-life of about 4 days. That means about 0.35 after two weeks away, and the floor is reached after about a month.
- It **only** scales expressive frequency toward the user: greeting probability, greeting magnitude cap, front-perch pull while the user is present, and call rate while observed (±25%).
- It is **never** an input to wary, distress, or any valence. Returning after absence never raises the wary transition rate. A test asserts that the mood transition matrix is invariant to attunement.
- It recovers within a normal session or two. Because it can't go below the floor, birds always keep calling and always still notice the user.
- It is never exposed, labelled, or exported. It's a transient simulation variable, not personality (§15 D3).

### 5.5 Mood

States: `alert`, `curious`, `content`, `wary`, `drowsy`, `settled`. Night sleep is `settled`, which reads as eyes closed and low on the perch.

At each tick, for each bird, build a target distribution `π` over moods as a product of experts (normalized):

- **Time-of-day prior** in the aviary's local time: dawn (05:30–08:00) favors alert, midday favors content and curious, dusk (18:00–20:30) favors drowsy, night favors settled. The nocturnal species (the nightjar analogue) inverts the prior. It is alert and curious at dusk and night and drowsy at midday.
- **Weather:** rain → drowsy/content, plus call-rate damping of 0.5× for the duration plus 10 minutes. Wind → alert for high boldness, wary for low boldness.
- **Recent interactions:** an effective offer accepted → content (decays over 20 minutes). Listen-in → content or curious. Sustained presence with the bird on the front perch → curious. Settle → drowsy nudge for 10 minutes.
- **Social contagion:** a nearby bird in wary, or an alarm event, gives a wary weight of `0.6·(1 − boldness)`. Chorus participation → content.
- **Personality:** boldness suppresses wary, curiosity raises curious, warmth raises content when other birds are near.

**Transitions.** The hazard is `h = 1/τ` with τ between 25 and 45 minutes, depending on the bird. There is a minimum dwell of 8 minutes, except for strong triggers (alarm, offer reaction, rain onset). A hysteresis margin prevents A↔B flicker.

**Daily-ish reset.** At local dawn, each bird re-samples from the dawn prior blended with its personality baseline. It's a soft pull, not a hard snap. Staggering by bird between dawn and dawn plus 40 minutes makes the reset read as waking up. Because the reset happens on the server at dawn, the user never sees mood "snap" when they open the tab. They open the tab into whatever the server has.

**Across sessions.** Mood lives only in `bird_state` and is never reset by a client connecting.

**Alarm events.** A wary bird emits an alarm with probability 0.02 per tick, capped at one alarm per aviary per 30 minutes. This prevents contagion loops.

### 5.6 Perch selection and movement

- Slot layout is 3 front, 3 middle, 4 back. That's 10 slots for at most 7 birds plus one wanderer, so there's always somewhere to go.
- Each bird's perch utility combines several terms:
  - boldness × (zone front-ness)
  - mood terms (wary → back, curious → front, drowsy → current perch, since staying put is preferred)
  - warmth × proximity to other warm birds
  - while the user is present, attunement × boldness × front-ness (birds come forward to watch the watcher)
  - an inertia term
- Perch changes are Poisson-scheduled with a mean of 6–20 minutes depending on mood. Each change is written into `schedule` as `fly`/`hop` with a server timestamp and duration, and it is scheduled at least 5 seconds into the future so every client can play it.
- The **choreography horizon** is 90 seconds. Every tick re-plans the part of the horizon that hasn't started yet. Actions that have already started are never revised.

### 5.7 Weather

- Per-aviary Poisson processes with deterministic seeds. Rain averages about 3 per week, lasts 8–25 minutes, and has soft intensity. Wind averages about 1 per day and lasts 3–10 minutes. Rain is suppressed for 36 hours after the previous rain, which prevents clusters.
- There is no storm, snow, or thunder. Weather lives in the snapshot, so every device and visitor sees the same rain.

### 5.8 Bird-to-bird social layer

The server schedules **chorus windows**. A window opens when at least 2 birds have `call.ratePerMin × chorusJoinP` above a threshold, their moods are not in {wary, settled}, and they aren't in a rain damp. A window lasts 20–60 seconds, with at most about 1 per hour during day and fewer at dusk.

Call-and-response between individual calls is client-side, within the `responseP` parameters (§8.3). It doesn't need cross-device agreement, and keeping it local makes it feel immediate.

Wary contagion and alarms are server-side (§5.5), because they change mood.

### 5.9 Species pool and new birds

- **Six species,** designed as one coherent habitat:
  - wren-like (small, loud trills)
  - warbler-like (high, ornamented)
  - finch-like (bright, chattery)
  - tit-like (two-note whistles)
  - dove-like (low coos)
  - nightjar-like (nocturnal churr)

  Each species defines a rig, a silhouette, palette ranges, a motif library, and caption vocabulary.
- **Species versioning:** a `(species_id, version)` pair is immutable once shipped. Improvements ship as new versions for **new** birds only. Existing birds stay pinned forever to the version they were adopted with (I4). We will retain renderer and synth support for every shipped species version.
- **Starter pair:** two distinct species chosen by the system so that their call registers are far apart. Starter voiceprints must differ in base pitch by at least 5 semitones and must have different signature syllable classes. The nightjar is excluded from starters, because a night-only bird would make the first-day encounter quieter than it should be.
- **Individual voiceprints:** each bird gets a unique base pitch offset, a timbre (harmonic balance), a **signature syllable** that appears in at least 70% of its calls, and motif preferences. When two birds of the same species live in one aviary (which is unavoidable at 7 birds and 6 species), their voiceprints are required to differ by at least 4 semitones and to use distinct signature syllables.
- **Availability by aviary age only:** the 3rd bird is eligible at 90 days, the 4th at 150, the 5th at 240, the 6th at 330, and the 7th at 540. The schedule lives in config. No input other than `aviaries.created_at` affects it (I6, no gamification).
- **Arrival mechanics:** when a bird becomes eligible, the tick spawns a **wanderer**. It's a bird of a species not yet present where possible, with its own permanent id, voiceprint, and personality seed. It appears on a back slot for part of the day. It calls, perches, and is fully alive. The notebook may observe it ("a finch has been at the back perch two mornings running.").
  - Adoption is reached from the wanderer itself. Focusing it, by click or Enter, opens a small naming sheet in naturalist voice: "name this finch?", with a name field, suggestions, "stay" and "not now". There is no badge, no top-bar indicator, and no prompt pushed at the user.
  - "Not now" has no consequence. The wanderer keeps visiting occasionally, and eligibility never expires.
  - Account settings also list the wanderer under "Birds" for users who look there.
  - The PRD says birds aren't "acquired", so the wanderer is not called "new", "unlocked", or "available" anywhere.

### 5.10 Calibration harness (built in the first milestone, before any UI)

`sim-harness` runs `sim-core` against scripted **personas** on a virtual clock. Personas are event generators: Regular, Heavy, Light, Weekend-only, Background-tab, Two-device overlap, Absent-2-weeks, Muted-always, Night-owl (always at 01:00 local), Traveller (timezone change), and Visitor-heavy host (visitors should have zero effect).

The harness asserts the drift table in §5.3, the mood distribution per time-of-day band, greeting-order distributions (§7.3), notebook sparsity (§7.6), and invariants I2 and I3. It runs on every PR that touches `sim-core`, `drift_params`, or `species`, and produces a calibration report artifact with plots for the drift owner. A 12-week run of 1,000 personas finishes in under 2 minutes in CI.

---

## 6. Sync model

### 6.1 Principle

There is one canonical record per aviary, one writer (the tick), and many readers (devices and visitors). Clients are renderers with an outbox. Multi-device sync is not a feature to build. It falls out of the fact that both devices read the same row.

### 6.2 Event log ordering (no silent gaps)

Plain `bigserial` sequence gaps would let the tick skip events whose transaction commits after a later sequence value. To prevent that:

```sql
-- inside the ingest transaction
UPDATE aviary_event_heads SET next_seq = next_seq + n WHERE aviary_id = $1 RETURNING next_seq;
INSERT INTO interaction_events (aviary_id, seq, …) VALUES …;   -- seqs allocated from the returned range
COMMIT;
```

The row lock on `aviary_event_heads` serializes ingest per aviary. That's cheap, because per-aviary write rates are about one per minute. It also guarantees that **commit order equals seq order**, so the tick's `seq > event_cursor` read never skips anything. Idempotency comes from `unique(aviary_id, client_event_id)`: `ON CONFLICT DO NOTHING`, and the existing id is still acked.

### 6.3 Client write path (outbox)

- Every event gets a UUID `client_event_id` when it is created and is written to an IndexedDB outbox first. `SyncClient` flushes the outbox on `/sync` every 60 seconds (±10 s jitter), on visibility change, on offer or settle, and via `navigator.sendBeacon` on `pagehide`.
- Acked events are deleted. Unacked events are retried with exponential backoff, capped at 5 minutes.
- **Late events:** presence intervals and listen-ins older than 24 hours are dropped by the server. Events older than 10 minutes when consumed count toward **drift only**. They don't affect mood, because a stale offer shouldn't make a bird content now.

### 6.4 Presence ingestion and the server-side guard

- The client reports closed **intervals** `[start, end]` in server-time-corrected ms, one at least every 60 seconds while present, plus one on every present→not-present transition.
- **Server clamps:**
  - `end ≤ received_at + 5s`
  - `end − start ≤ 75s` for a normal ping (longer intervals are truncated)
  - no intervals from visitor tokens
  - no intervals from a revoked session
- **Union across devices:** the tick merges all presence intervals for the account into a union timeline before computing presence-seconds. Two devices watching at once count as one person watching (see the two-device persona in §5.10).
- `aviaries.presence.last_presence_end_at` is the account-level "last seen" used for absence length (§7.3). It's never displayed.

### 6.5 Read path

- **Initial:** the edge inlines the latest snapshot from the regional Redis replica (§9.1).
- **Keepalive:** `/sync` returns a new snapshot only when `state_version` has changed.
- **Triggers for an immediate pull:** `visibilitychange→visible`, `pageshow` with `persisted` (bfcache), `online`, and a long-frame-gap detector (a rAF delta or `Date.now()` jump over 5 s, which covers laptop suspend).
- **Client reconciliation:**
  - Snapshots with `v < current` are ignored.
  - A new snapshot merges schedules by action id, so actions already playing keep playing.
  - A bird whose perch changed while unseen plays a normal flight (or a cross-fade in reduced-motion) and never teleports.
  - A mood change applies by blending motion parameters over 3–5 seconds.
  - Lighting never waits for the network, because it's computed locally from clock plus timezone.
- **Clock:** `offset = serverTime + rtt/2 − Date.now()`, smoothed over the last 5 samples. All schedule times are server epoch.

### 6.6 Timezone

The client reports `Intl.DateTimeFormat().resolvedOptions().timeZone` in a `tz_report` event when it differs from `aviary.tz`. The canonical timezone switches only after the new zone has been reported with at least 10 minutes of presence and no presence from the old zone in between. That handles travel without flip-flopping between two devices set to different zones. The lighting and time-of-day mood prior use the canonical timezone, so both devices show the same light.

### 6.7 Conflicts and why they can't corrupt drift

| Scenario | Handling |
|----------|----------|
| Laptop and phone both open | Both read the same snapshot. Events from both append to one log. Presence is unioned. Listen-ins are per device and both count, but only while presence holds, and the per-bird daily soft cap applies. |
| Offers from two devices on the same bird within the cooldown | The planner reads the cooldown from `bird_state`, and the tick re-checks it in `seq` order. The second offer is marked ineffective. Both devices still see a reaction choreography. |
| A stale device (asleep for a day) wakes | It sends queued presence intervals, which are clamped and date-limited, and receives the current snapshot. It never pushes state. |
| Two tick workers race on an aviary | The fencing token (`tick_seq`) makes the loser's commit update 0 rows, and the loser aborts. |
| Name edited on two devices | Optimistic concurrency with `if_name_version`. The loser gets `409` and re-reads. This is the only user-mutable bird field. |
| Deploy with a new snapshot schema | Snapshots carry `schemaVersion`. The server serves N and N-1 for 7 days, and the client hard-reloads its shell when it sees `schemaVersion > supported`. |

### 6.8 Session and error surfaces

All of these are matter-of-fact system copy. They appear in a single-line **system strip** directly under the top bar. It is not a toast and it isn't dismissed on a timer. It persists until the condition clears.

| Condition | Copy |
|-----------|------|
| `401` on `/sync` | "Your session timed out. Sign in again to keep watching." with a sign-in link. The aviary keeps rendering from the last snapshot. Presence isn't recorded while signed out. |
| Initial snapshot fails (no inline snapshot and fetch errors after 3 retries) | "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." shown over the quiet field. |
| Offline for more than 2 minutes | "You're offline. Your aviary will catch up when you reconnect." The aviary keeps animating on the last parameters. The offer affordance stays usable and the offer is queued. Reactions are withheld until online (§15 D12). |

---

## 7. Interactions

### 7.1 Presence monitor (client)

```ts
present = document.visibilityState === 'visible'
       && document.hasFocus()
       && (now - lastActivityAt) < W
```

- **Activity:** `pointermove`, `pointerdown` (which covers touch taps, since they are pointer events), and `keydown` on `window`, captured passively. Wheel, scroll, `deviceorientation`, and audio playback do **not** count (§15 D2).
- **W:** 4 minutes by default. It's configurable from 2 to 8 and calibrated in dogfood, leaning longer as the PRD asks.
- **Evaluation:** on each activity event (throttled to one update per second), on `visibilitychange`, `focus`, and `blur`, and on a 5-second timer. Transitions emit interval events. The monitor is ~2 KB and fully unit-tested with a truth table of all 8 combinations plus timing edges.
- **Visitor mode:** not instantiated.
- **Settle:** ends presence immediately and suppresses it until the user re-engages (§7.5).

### 7.2 "Idle attention is interaction"

While presence holds, the server knows within about a minute. The tick applies its effects: bold, attuned birds come forward, curious moods become more likely, and presence-seconds feed drift. The user never sees a counter or indicator of this.

### 7.3 Return-greeting

**Triggers:**
- `session_start` (fresh navigation)
- `return`: visibility back after being hidden for 15 seconds or more, or a long-frame-gap resume

Shorter hides don't greet. Otherwise every alt-tab would produce a greeting, and constant greetings are canned.

**Planner** (`sim-core.planGreeting`, run in the API at snapshot request):

- **Absence** `A = now − last_presence_end_at` (account-level, any device) sets the band, which in turn sets the magnitude ceiling:

  | Band | A | Magnitude ceiling |
  |------|---|-------------------|
  | a | < 5 min | glance up |
  | b | 5 min – 3 h | glance, head-tilt, or quiet 2-note call |
  | c | 3 h – 2 days | tilt plus step or hop toward the front, or a call |
  | d | > 2 days | re-orientation: fly to a nearer perch and/or a longer call, possibly answered by a second bird |

- **Greeter selection:** sample one bird with weight
  `boldness^1.5 · warmth · moodFactor · attunement · perchProximity`,
  where `moodFactor` is wary 0.3, drowsy 0.4, settled 0.05, alert 1.2, curious 1.3, content 1.0. The bolder or warmer bird tends to go first. A wary bird greets later or not at all. The selection sometimes produces no greeter at night, which is honest.
- **Additional greeters:** each other bird joins with probability `0.35·warmth·attunement·moodFactor`, staggered by a random 0.6–3.0 second offset from the previous greeter. There is never a simultaneous chorus on cue.
- **Form:** chosen from the gesture grammar (below), within the band's ceiling, using the bird's **greeting style**. Style is a stable per-bird preference over gestures derived from voiceprint plus personality, which is why the same bird greets the same way across visits. The specific instance is then procedurally varied: timing jitter, gaze path, call motif instance, hop distance.
- **Timing:** the first greeter starts 400–1,400 ms after first frame, which keeps it inside the PRD's "first second or two".
- A server `greeting` event is recorded. The notebook uses it to observe order ("pip greeted before wren today").

**Gesture grammar** (composable primitives in the client choreography player):
`glanceUp`, `pausePreen`, `headTilt(dir)`, `stepToward(n)`, `hopForward`, `flyToPerch(zone, slot)`, `call(motifClass=greet, length=short|long)`, `answerCall(from=birdId)`, `fluffSettle`.

A greeting is a sequence such as `[pausePreen, glanceUp, call(short)]`. Each primitive has continuous parameters (durations, angles, and pitch are drawn at play time), so no two instances are identical.

**Test:** over 1,000 plans for the same state, no two rendered choreographies are identical (hash of the resolved parameter vector). The order distribution must match boldness ranking, with a Kendall τ of at least 0.4.

Because the greeting is the only welcome surface, there is no text anywhere on return (I6). Narration voices the greeting as an observation (§10.1).

### 7.4 Listen-in

- **Engage:** click or tap a bird (canvas hit test), or press Enter on a focused bird.
- **Disengage:** click the focused bird again, focus a different bird (which switches), click empty space, press Escape, move keyboard focus out of the scene, or presence ends (so a user who has walked away doesn't keep inflating listen-in).
- **Mix change:** see §8.5.
- **Visual:** the only visual response is subtle and diegetic. The focused bird may tilt toward the viewer, and a keyboard-focused bird shows the focus ring. There is no highlight, halo, or label.
- **Events:** `listen_in_start` and `listen_in_end` are sent as one interval event on end, plus a periodic interval every 60 seconds for long listens.

### 7.5 Offer

- **Top-bar affordance:** a seed glyph. The keyboard shortcut is `o` (disable-able, §10.4). It opens a small popover with three items: **a seed**, **a song fragment** (which reveals six short fragments, each played softly on focus and hover, since there is no text catalog), and **a still pool**.
- **Placement:** automatic. If a listen-in is active, the offer goes near the listened-in bird. Otherwise a seed goes front-center on the ground line, a pool always goes at the front, and a fragment plays from the front center. The user never places items manually (§15 D11).
- **Flow:** the item appears in the scene immediately with a local visual that is not state. At the same moment, `POST /v1/aviary/offer` runs, and the returned reaction choreography plays. If the RTT is over 600 ms, the birds' "noticing" micro-motion (heads turning to the item) is a local embellishment that covers the gap.
- **Reactions** (planner inputs: mood, curiosity, boldness, perch distance, cooldown):
  - **seed:** approach, peck, and return, or a slow cautious approach after a delay, or a watch without approaching. A drowsy bird may not move.
  - **fragment:** the bird joins in with its own motif against the fragment's contour (the synth harmonizes within the bird's voiceprint), goes quiet and tilts, or calls against it. Driven by vocal and mood.
  - **still pool:** drink, bathe (with a splash micro-motion and a soft procedural splash sound), or watch. The pool stays for about 4 minutes and then fades as a normal scene transition.
- **Cooldown:** 4 minutes per bird, enforced server-side. It is invisible in the UI. A bird in cooldown reacts with low interest, such as a glance, rather than with nothing. There is no disabled state, no timer, and no message.

### 7.6 Field notebook

- **Observation extraction** (tick): writes structured facts into `observation_buffer`. Kinds include:
  - `greet_order_first` (first time this week a given bird greeted first)
  - `perch_shift_trend` (a bird has spent more time on the front perch this week than last; computed from dwell, never from traits directly)
  - `long_preen`
  - `rain_passed` (+ bird reactions)
  - `wind_reaction`
  - `chorus` (which birds, time of day)
  - `night_calls` (nightjar)
  - `offer_notable` (first bath in a pool, first joining a fragment)
  - `wanderer_seen` / `arrival`
  - `fluffed_cold_morning` (derived from time and mood)
  - `quiet_stretch` (aviary-level quiet, never phrased relative to the user)
- **Writer job:** runs every 6 hours per aviary, and additionally after a high-salience fact. It chooses at most one fact, subject to:
  - A token bucket: capacity 2, refill one token per 60 hours. That gives roughly one entry every 2–3 days on average, a maximum of about two in a short burst for noteworthy events, and a hard cap of 3 per 7 days.
  - Minimum spacing of 18 hours unless salience ≥ 0.8.
  - Novelty: no repeat of the same (kind, bird) within 10 days.
  - Very active users hit the same caps, so sparsity is preserved no matter what.
- **Rendering** is done by the `prose` engine (§7.7). The entry is prefixed with the lowercase local weekday ("tuesday — …") when the fact is day-specific.
- **What is forbidden in facts:** the fact schema has **no user-behavior fields** (no visit counts, durations, session times, absence lengths). Greetings are facts about birds ("pip greeted first"), which the PRD explicitly allows. Writing that the user was present every day is structurally impossible because that data isn't in the fact schema.
- **UI:** the "Field Notebook" sheet slides from the right on desktop and from the bottom on mobile, and the aviary stays audible and visible behind it. The list is virtualized with recycled DOM nodes and infinite back-scroll through `/v1/notebook`. It is read-only, with no edit, delete, or share controls. It is lazy-loaded on first open and prefetched at idle.

### 7.7 Prose engine (shared by notebook, narration, captions, greeting and reaction narration)

- **Approach:** a hand-authored **constraint grammar** in the Tracery or Expressionist style, with typed slots filled from facts: bird name, species noun, perch phrase, time phrase, weather phrase, verb sets per species and mood. A staff writer owns the phrase libraries: about 300 notebook templates, about 200 narration fragments, and a caption lexicon per species (see §8.4). The engine is seeded and deterministic per (aviary, entry). It keeps a recent-phrase memory so it avoids reusing any template within N uses.
- **Why not an LLM:** sending per-bird facts to a third-party model would break the privacy commitment. A self-hosted model adds voice-drift risk and a whole ops surface for v1. The grammar gives total control over voice and supports lint validation. A self-hosted model can be evaluated post-v1 behind the same interface, subject to the same lint.
- **Output lint,** in CI against the full grammar expansion and at runtime before persisting, with fallback to another template:
  - lowercase (except proper UI labels)
  - no `!`
  - no second person (`you`, `your`)
  - banned lexicon: `welcome`, `back`, `streak`, `days in a row`, `visit`, `achievement`, `unlock`, `level`, `badge`, `score`, `happier`, `mood:`, and digits followed by `%`
  - no numbers except natural time words
  - 6–40 words
  - present tense (verb-list heuristic)

---

## 8. Audio pipeline

### 8.1 Engine structure

```
AudioContext (latencyHint: 'playback')
  └─ AudioWorkletNode "aviary-synth" (stereo out; all voices rendered inside; preallocated)
        voices: up to 8 birds (7 + wanderer) · offer voice (fragment/splash) · ambient bed (wind/rain/leaves)
        per-voice: synth → gain (listen-in) → equal-power pan (scene x) → distance LPF + dry/wet send (perch depth)
  ├─ ConvolverNode (IR generated procedurally at startup: filtered noise with exp decay ~1.2 s — no files)
  ├─ DynamicsCompressor (gentle bus glue) → master Gain (mute, settle, time-of-day level) → destination
```

- **One worklet renders everything.** Voices and grains are preallocated, and parameters flow through a `MessagePort` with a fixed-size message pool. That means no per-call node allocation and no GC churn, which is how we satisfy "no memory growth" and "buffers reused" by design.
- **Fallback 1:** if `audioWorklet.addModule` fails, a node-graph synth builds pooled `OscillatorNode`s and `GainNode`s per syllable. It's heavier, but still procedural.
- **Fallback 2:** if there's no `AudioContext` or it can't be created, the aviary runs in silence with **captions on by default**. A matter-of-fact note in accessibility settings says: "Sound isn't available in this browser, so captions are on." There's no toast (I6). Recorded audio is never used (I7).

### 8.2 Autoplay policy

Browsers suspend `AudioContext` until a user gesture. Returning users on Chromium may be exempt through the Media Engagement Index. We don't show a "tap to enable sound" overlay, because that would announce the product.

- The context is created at boot, and we attempt `resume()` immediately.
- If it's still suspended, the scene and call *visuals* (beak open, throat flutter) keep playing. Captions play if the user has enabled them. Calls go through the synth scheduler into a suspended context, so they're effectively silent.
- On the first `pointerdown`, `keydown`, or `touchend`, we call `resume()` and ramp the master gain from 0 to the target over 1.5 seconds. The aviary then *becomes audible* mid-motion, like stepping outside, rather than starting.
- This is a platform-imposed deviation from "calls already audible" on the first frame, recorded in §14.

### 8.3 Call grammar

**Species motif library:** 8–14 motifs per species. Each motif is a sequence of 1–6 **syllables**, and each syllable is a parametric description:
- `type`: whistle (sine plus harmonics), FM chirp, noise-band trill, coo (formant), or churr (AM noise)
- `f0` contour: a Bézier with 2–4 control points, in semitones relative to the voiceprint base
- duration, amplitude envelope (attack, decay, release)
- trill rate, vibrato depth, harmonic balance

**Grammar:** a small weighted grammar per species:
```
Call → [Intro] Body{1..n} [Coda]
Body → Signature | Motif_i
```
The bird's **signature syllable** is guaranteed to occur in at least 70% of calls. It's the recognizability anchor.

**Per-call resolution** (client, at schedule time). The motif sequence is sampled from the grammar with weights shaped by mood and the bird's `motifWeights`. Parameters are then perturbed as follows:

| Mood | Tempo | Pitch range | Amplitude | Onset | Length |
|------|-------|-------------|-----------|-------|--------|
| wary | +10% | narrow | lower | sharp | 1–2 syllables |
| content | 0 | normal | normal | soft | longer phrases |
| curious | 0 | rising end contours | normal | normal | — |
| drowsy | −25% | −2 st | low | slow | short |
| alert | +15% | +1 st | crisp | — | — |
| settled | — | — | very low | — | rare |

Individual micro-variation is applied on top: ±30 cents per syllable, ±12% duration, ±2 dB, and a random drop or repeat of a non-signature syllable.

**Uniqueness guard:** a 64-bit hash of the quantized resolved parameter vector is stored in a ring of the last 512 calls. On collision the call re-rolls. Collisions are astronomically unlikely, and the guard exists so that "calls never repeat exactly" is a checkable property.

**Recognizability invariants** hold under any mood or drift:
- base pitch stays within ±1.5 semitones of the voiceprint
- signature syllable present at ≥70%
- timbre class fixed

Drift in `vocal` changes **how often** a bird calls, never **what it sounds like**.

**Scheduling:** a look-ahead scheduler (100 ms horizon, run on a 25 ms timer from a dedicated tick in the scene loop) draws call onsets from a Poisson process with rate `call.ratePerMin`. The rate is modulated by:
- recent calls from other birds, which bring a response with probability `responseP` and a 0.4–2.0 s delay
- chorus windows from the schedule, which raise the rate and bias toward overlapping onsets
- rain (damping)
- night, where everything except the nightjar is very sparse

Calls are also emitted by choreographies: greetings, offer reactions, and alarms.

**Caption hook:** every resolved call is passed to the caption generator at the same moment it is scheduled (§8.4 and §10.2).

### 8.4 Caption generation from the call grammar

A deterministic function maps the resolved call to a phrase. It reads:
- syllable count: "single", "two-note", "three-note", "a run of"
- overall contour: rise, fall, rise-fall, flat
- register relative to species: high, low
- texture: trill, whistle, coo, churr
- rhythm: steady, "paused, … again" when there's a gap over 250 ms
- dynamics: soft, sharp
- position: "from the back perch", when the call comes from the back zone

That gives phrases like "a soft three-note rise", "a low trill, paused, low trill again", and "a single sharp call from the back perch". The caption always matches what was actually synthesized, because both are generated from the same resolved object.

### 8.5 Listen-in mix and its decay

- **Engage:** the focused bird's gain ramps to +5 dB and its reverb send drops by 30%, so it comes "closer". Other birds ramp to −12 dB. Their floor is −15 dB (never silent). The ambient bed ramps to −4 dB.
- **Ramps** use `setTargetAtTime`-equivalent exponential approaches inside the worklet (one-pole smoothing per voice gain), with a time constant of 0.6 s on engage (about 2 s to settle) and **0.9 s on disengage** (about 3 s to settle). The slow disengage is the listen-in mix decay. It returns to the ambient balance gradually and never as a cut.
- **Switching** between birds is a crossfade of the two voices' targets with the same time constants. Other birds never pass through silence.

### 8.6 Chorus mixing

- Two or more simultaneous voices go through per-voice pan by scene x and depth filtering, so they separate spatially.
- A bus compressor (threshold −18 dBFS, ratio 2:1, slow release) plus a soft limiter at −1 dBFS prevent chorus build-up from pumping.
- Loudness target for the full aviary at day: about −24 LUFS short-term, which is intentionally quiet.
- There's no phase-cancelling artifact, because there are no loops. Each voice is independently synthesized with its own jitter.

### 8.7 Settle, day/night, and visibility

- **Settle:** master gain drops by 8 dB over 6 seconds, and call rates drop to evening levels. Undo restores the previous gain over 2 seconds.
- **Night:** the master level drops by 4 dB. The nightjar voice stays at full level.
- **Tab hidden:** fade out over 2 seconds, then `suspend()` (§15 D8). On visible, `resume()` and fade in over 1.5 seconds.

### 8.8 Audio QA tooling

- An `OfflineAudioContext` render farm runs in CI. It renders 1,000 calls per species per mood and checks for clipping, loudness bounds, the uniqueness hash, and spectral-fingerprint distance (between calls of the same bird > ε, so no exact repeats; between birds in one aviary > δ, so they're distinguishable).
- A small classifier on MFCC features must identify a bird's voice among its aviary-mates with at least 95% accuracy across all mood and drift combinations. This is an automated proxy for recognizability. Human listening panels are the real test (§12.3).

---

## 9. Frontend rendering pipeline

### 9.1 Boot and first frame (the < 500 ms path)

1. The **edge** returns the HTML (target ≤ 28 KB gzipped). It contains inline critical CSS, the inline **boot chunk** (≤ 45 KB gzipped), and the inline snapshot JSON. The boot chunk includes the scene core, the Canvas2D renderer, the bird rigs for the species present, the lighting function, and the choreography player. There's also `<link rel=preload>` for the main chunk.
2. The inline pre-script sets the `<body>` background to the sky gradient for the current local time, computed from `Intl` in about 0.2 ms. The quiet field is therefore correct from the very first paint.
3. The boot chunk parses the snapshot and places each bird at its **current perch, mid-activity**. Idle phases are randomized, so one bird is mid-preen, another mid-scan, and so on. It seeks any in-progress schedule items and overlays to `now − startedAt` and draws the first frame on the next rAF. The code sets `performance.mark('first-bird')` after the first frame that contains at least one bird has been committed.
4. After first frame, the following load: the main chunk (the full motion library, ambient system, and chrome in Preact), then the audio engine chunk and worklet, then the narration and caption prose chunk. Settings, account, visits, and notebook are lazy chunks.
5. **Service worker:** on repeat visits the shell and chunks come from the SW cache. The edge then only has to supply the snapshot. The SW fetches the page from the network with a 1.2-second race. If it loses, it serves the cached shell plus a cached snapshot, but only if that snapshot is under 2 minutes old. Otherwise it shows the quiet field until the snapshot arrives. A stale snapshot older than 2 minutes is never presented as current.
6. **No spinner, ever.** The quiet field is a soft sky gradient with occasional faint ambient motion (a drifting mote, leaves in far-background silhouette). It never has indeterminate-progress semantics.

**Empty-aviary state** (post-adoption only): the quiet field. Then the first starter flies in to its perch over about 2.5 seconds, and the second arrives 4–9 seconds later, with a short call on landing. The fly-in only happens when the client finds `adoptedAt` within the last 5 minutes and no prior render flag in `localStorage`. After that, the aviary is never empty again.

### 9.2 Renderer choice

The renderer is **Canvas2D**, using a custom retained scene graph with no rendering framework.

- **Why:** at most 8 birds of about 25 vector parts each, three background planes, and around 10 particles is well within Canvas2D's GPU-accelerated budget on a 5-year-old laptop. Canvas2D has universal support (no WebGL blocklist risk) and a tiny footprint (fits the 45 KB boot chunk), and the same path draws the first frame and every later frame.
- **Escape hatch:** the scene graph talks to a `Renderer` interface. A WebGL2 backend can be added if perf gates fail on reference hardware, and we'd decide that by the end of M2.

**Layers:**
1. Sky and background foliage. Pre-rendered to `OffscreenCanvas` at 4 light keyframes (dawn, day, dusk, night) and rebuilt on resize. The current light is a cross-fade of two adjacent keyframes (two `drawImage` calls with alpha) plus a global tint. Cost is about 0.5 ms.
2. Mid plane: perches and branches, cached per keyframe the same way.
3. Birds. Drawn each frame from rigs. Colors come from `look.colors` × the light tint, and are computed per frame per part (cheap).
4. Foreground: occasional branch or leaf, weather (rain streak particles pooled at a fixed count, a wind sway parameter), ambient leaves and feathers (pooled).
5. Focus ring and captions. Captions are DOM, not canvas, so they stay crisp and screen-reader-invisible (`aria-hidden`, because narration covers them).

**Parallax:** only a gentle shift driven by pointer position, at most 6 px on the background and 3 px on the foreground, eased. It's disabled in reduced-motion and on touch devices.

**DPR** is capped at 2. There is **adaptive resolution**: if the rolling p90 frame time exceeds 14 ms over 5 seconds, the backing-store scale drops in steps of 0.85, down to a minimum of 0.6 of the DPR.

### 9.3 Scene layout (responsive, never cropping a bird)

- The world coordinate system has a height of 1.0 and a width equal to the viewport aspect, clamped to [0.55, 3.2]. Outside that range, the sky and ground extend to fill the remaining space, and the playfield (the **safe rect**) stays centered.
- Perch zones are depth planes with scale factors of front 1.0, middle 0.8, and back 0.62. Slot x-positions are fractions of the safe-rect width, so narrow viewports compress spacing horizontally and wide viewports spread it out.
- Bird sizes are clamped by the smaller of the safe-rect width divided by (slots per zone × 1.2) and the zone's height budget.
- **Invariant test:** for a matrix of viewports from 320×480 to 3840×1600, including portrait phones and ultrawide screens, the layout checks for every perch slot and every flight path at 20 sample points. The bird's bounding box, including its tail and wings in flight pose, must be inside the viewport minus the top-bar height. This runs as a property-based test in CI.
- The top bar sits **above** the scene (in layout flow, not overlaid). The scene fills the remaining height.

### 9.4 Idle micro-motion system

**Rig:** each bird has about 10 bones: root, body, chest, head, beak (upper and lower), tail, two folded wings, and legs. The shapes are species-defined Bézier parts with procedural plumage detail (feather edge noise scaled by plumage level).

**Layered procedural motion,** all `dt`-based:
- **Breathing:** chest scale at 0.3–0.6 Hz, with per-cycle period jitter of ±15%. The amplitude is mood-shaped, and drowsy breathing is slower and deeper.
- **Micro-noise:** 1D simplex noise on body sway and head, amplitude ~1–2 px, frequency < 0.5 Hz.
- **Behavior selector:** a utility AI picks among `scan`, `preen(sequence)`, `tilt(toward sound source)`, `shuffle(weight reset)`, `fluff`, `blink`, `tailFlick`, `lookAt(item | bird)`, and `rest`. Weights come from `motion.*` and mood (wary scans more, content preens, curious tilts, drowsy fluffs and rests low). Durations are drawn from distributions, never fixed. Preen sequences come from 6–10 authored curves per species with procedural parameter variation.
- **Sound attention:** when a call starts, other birds get a `tilt` toward the caller with probability `curiosity-ish` (`motion.tiltBias`), after a 150–500 ms delay. Offers work the same way.
- **Flight:** a Bézier path between perch anchors with a height arc scaled by distance, a wingbeat cycle of 5–9 Hz on small birds, a glide at the end, and a landing flutter.
- **Anti-strobe rule:** no visual property oscillates with amplitude above 2 px at more than 3 Hz, except wingbeats, which are brief and natural motion. There are no hard-cut pose changes. Every transition is eased over at least 120 ms.
- **Never paused-looking:** every bird always has at least breathing and micro-noise running. The motion system has no "idle" state that is fully static.

### 9.5 Transitions

- **Mood change:** blend motion-parameter targets over 3–5 seconds.
- **Day and night:** continuous lighting from the shared curve in the aviary's timezone. Keyframe cross-fades never jump.
- **Settle:** a 4-second eased transition of lighting to the "settled evening" grade (warm, dimmed). Birds drift toward drowsy poses in local presentation, and the server applies a drowsy nudge on its next tick.
  - **Undo window:** for 5 seconds, **any click or tap anywhere in the aviary** reverses the transition over about 1.5 seconds. The settle event is **held in the outbox and only committed after 5 seconds** (or flushed on `pagehide`), so an undone settle never reaches the log.
  - After the window, the aviary stays settled until the tab closes or the user actively re-engages: a click on the scene, a listen-in, an offer, or a key press in the scene. Re-engaging lifts the settle over 3 seconds and presence can resume. Mouse movement alone doesn't un-settle, but it *does* count toward presence if the user stays. Settle suppresses presence reporting until an explicit re-engagement, so "settle = presence ends" holds.
  - Settle is device-local presentation. Another device doesn't dim, but it sees the drowsy nudge in the canonical moods.

### 9.6 Top bar

- **Items, left to right:** notebook, offer, settle. The system items are on the right: accessibility settings and account/settings. All are icon buttons with accessible names and tooltips that appear on focus or hover **in the bar only** (no tooltips in the scene). §15 D1 explains the settle placement.
- **Fade:** after 3 seconds with no pointer movement over the page, the bar fades to 8% opacity over 1.2 seconds. Pointer movement, a key press, or focus within the bar restores it to 100% over 200 ms. It never fades while keyboard focus is inside it, while a popover is open, or while the system strip is showing. On touch devices it fades after 4 seconds, and a tap in the top 48 px restores it (the tap does not un-settle or trigger listen-in).

### 9.7 Reduced-motion mode (a designed surface)

**Activation:** `prefers-reduced-motion: reduce`, or the accessibility setting (on / off / follow system). The OS preference always wins toward reduce.

**Stillness renderer:**
- **Poses:** each behavior's pose library (about 12–18 key poses per species: perched-neutral, preen-a/b/c, scan-left/right, tilt, fluff, drowsy-low, alert-upright, …) is sampled at discrete times.
- **Pose changes:** each bird changes pose every 6–14 seconds (jittered) by **cross-fading** the old and new renders over 1.2–2.0 seconds. Both are rendered to per-bird offscreen canvases, then alpha-blended.
- **Perch changes:** fade out at A over 1 second, a beat, then fade in at B over 1.2 seconds. There are no paths.
- **Removed:** leaves, feathers, parallax, rain streaks. Rain is shown as a soft cross-faded darkening and a sheen layer.
- **Kept:** lighting color shifts, at 2× duration. Settle takes 8 seconds.
- **Greetings** become a pose cross-fade (glance-up or tilt) plus the call. Offer reactions become cross-fades between pose states at the item's position.
- **Unchanged:** calls, captions, drift, mood, and notebook.

Reduced-motion gets its own visual QA baseline and its own design review. It's in scope for M2, not a follow-up.

### 9.8 Memory discipline (enforced by CI soak)

- Object pools for particles, schedule items, choreography nodes, and caption DOM nodes (max 3 live).
- Offscreen canvases are reused and only reallocated on resize, with debounced resize.
- No closures captured per frame, and no array allocation in the frame loop (lint via a custom ESLint rule inside `scene/frame/**`).
- The notebook list is virtualized, with row nodes recycled and data kept in a bounded LRU of 200 entries. Older entries are refetched.
- Event listeners are registered once. Snapshot objects are replaced, not accumulated, and old schedules are pruned after their end time.

---

## 10. Accessibility surfaces

### 10.1 Screen-reader narration

- **Structure:** the aviary region is `role="region" aria-label="the aviary"`. It contains a visually hidden narration live region with `aria-live="polite"` and `aria-atomic="true"`, and the bird focus group (§10.4).
- **Source:** the **same snapshot plus scene events** that drive the visuals, fed to the `prose` engine's narration grammar on the client. That makes it immediate for greetings and reactions and avoids an extra fetch.
- **Idle cadence:** one update every 30–60 seconds (jittered). The composer picks 1–2 salient subjects, such as a bird that recently moved, called, or is doing something notable. It adds a time or weather clause no more than every third update, and it rotates subjects. Example: "wren is on the low perch, fluffed against the cool air. pip calls softly from the back."
  - It never enumerates every bird, never uses perch numbers, and never states mood as a label ("content"). Behavior conveys mood.
  - It avoids repeating any sentence within its last 20 outputs.
- **Priority events:** greeting, offer reaction, settle and undo, a wanderer's arrival, and rain onset are each narrated within about 1 second of the visual moment. When one arrives, the composer clears any pending idle text and delays the next idle update. Priority events are written as observations: "pip looks up from preening and gives two quiet notes."
- **Throttle:** at most one priority update every 8 seconds. Extra events merge into the next composed update, which keeps the screen reader's queue from flooding.
- **Settings** (accessibility, matter-of-fact labels): Narration on/off (default on; it only affects screen-reader output), and Narration pace calm (45–75 s) or regular (30–60 s). A "Describe the aviary now" shortcut (`d`) composes one update immediately.
- **Visitor mode:** narration is identical, including greetings being absent, since the visitor gets none.

### 10.2 Captions

- **Opt-in** in accessibility settings (default off). They're forced on when WebAudio is unavailable.
- **Presentation:** DOM elements positioned near the calling bird's head. Offset avoids overlapping birds by picking one of 4 anchor sides that fits. They fade in with the call onset over 250 ms and fade out 1.5 seconds after the call ends. In reduced-motion they cross-fade without movement.
- **Limits:** at most 3 on screen. In a chorus window, overlapping calls merge into one caption, e.g. "pip and wren call over each other, bright and quick", generated from the voices involved.
- **Contrast:** caption text sits on a soft, rounded, translucent scrim. The scrim's opacity is chosen per placement by sampling the rendered background luminance in the caption box (a cheap 4×4 average from the cached layer), so text contrast stays at or above 4.5:1. A CI check renders captions over all 4 light keyframes and the rain state and asserts contrast.
- **Content:** captions for a muted user (muted but captions on) still reflect what *would* have played.

### 10.3 Contrast

All copy (top bar, popovers, sheets, settings, system strip, captions) meets WCAG AA: 4.5:1 for body text and 3:1 for large text and UI component boundaries. The design tokens carry contrast pairs, and a token-level test checks every pair.

The **top bar** has its own solid-enough background at full opacity. When it fades to 8% it's non-interactive decoration until it's restored, and any interaction restores it before use.

The **focus ring** is two-tone: a 2 px light inner ring (#F6F1E7) and a 2 px dark outer ring (#1E2A2A). That makes it at least 3:1 against every light keyframe, and it is verified in CI across the keyframes.

### 10.4 Keyboard and focus

**Tab order:** skip link ("Skip to the aviary"), then the top-bar items (notebook, offer, settle, accessibility, account), then the aviary scene group, then back to the top.

**Aviary focus group:**
- Structure: a `div role="group" aria-label="birds in the aviary"` with one visually transparent, absolutely-positioned `button` per bird that tracks the bird's on-screen bounds (updated at 10 Hz, not per frame). Roving `tabindex` means Tab enters on the first bird and Tab again leaves the group.
- Arrow keys: ←/→ move in screen x-order, and ↑/↓ move by depth (toward back or front).
- The **accessible name** is naturalist and short: "pip, a wren, on the front perch". The **description** (`aria-description`, refreshed only when the bird moves or changes activity) reads like "preening". `aria-pressed` reflects listen-in. There are no mood labels or numbers.
- Enter toggles listen-in and Escape exits it. Moving focus out of the group disengages listen-in, as the PRD specifies.
- A wanderer's button opens the naming sheet on Enter.

**Shortcuts:** `o` offer, `d` describe, and `m` mute or unmute (device-local). WCAG 2.1.4 compliance: shortcuts can be turned off in accessibility settings, they only fire when focus isn't in a text field, and `?` lists them in accessibility settings.

**Popovers and sheets:** the offer popover is a menu with arrow-key navigation, Enter to select, and Escape to close, returning focus to the offer button. Sheets (notebook, settings) trap focus, close on Escape, and return focus to their trigger.

**Focus indicator:** the canvas draws the two-tone ring around the bird's silhouette (a padded ellipse that follows the bird), shown only in the `:focus-visible` equivalent state. DOM focus rings use the same tokens.

### 10.5 Other

- **Zoom:** the layout supports 200% browser zoom. The top bar reflows, and the scene just renders smaller birds.
- **Settings copy** is in matter-of-fact voice. Example: "Reduce motion: On / Off / Use system setting".
- **Language:** `lang="en"`. v1 is English-only (§15 D13).

---

## 11. Performance budgets and observability

### 11.1 Budgets (all CI-enforced unless noted)

| Budget | Target | Hard limit | Enforcement |
|--------|--------|-----------|-------------|
| Initial JS at first paint (gzipped) | ≤ 350 KB total, of which inline boot ≤ 45 KB | **2 MB** (PRD) | `size-limit` in CI fails the build above the target by more than 10% and always fails above the hard limit |
| HTML + inline snapshot (gzipped) | ≤ 28 KB | 40 KB | CI |
| Snapshot payload, 7 birds (gzipped) | ≤ 6 KB | 12 KB | Contract test |
| Time to first bird visible, mid-tier Android (Moto G Power-class), 4G profile (9 Mbps / 70 ms RTT), warm DNS | p75 ≤ 450 ms | **500 ms** (PRD) | Lab test in CI (WebPageTest private instance) on every main build, plus RUM p75 |
| Time to first bird, cold connection, same device | p75 ≤ 800 ms | tracked, not gated (§14 R-P1) | Synthetic checks |
| Frame time, reference laptop (2020 i5 with Iris Plus, 8 GB, Chrome stable), 7 birds, rain, 1440p @ DPR 2 | p95 ≤ 16.7 ms, dropped frames < 1% over 30 min | 60 fps sustained (PRD) | Nightly perf rig on physical hardware |
| Scene JS per frame | ≤ 4 ms update + ≤ 6 ms draw | — | Perf rig traces |
| Audio worklet CPU | ≤ 20% of the render quantum budget at 8 voices plus ambient | 0 underruns in 30 min | Perf rig |
| Memory over a 30-min session | heap slope after forced GC ≤ 0.5 MB per 10 min, DOM node count constant ±5%, AudioNode count constant | **no growth** (PRD) | CI soak test (Playwright plus CDP, 30 minutes of virtual activity with 60 compressed ticks, samples at 5, 15, and 30 min) on Chromium every night and Firefox and WebKit weekly |
| API `/sync` and snapshot | p95 ≤ 60 ms server time | — | SLO |
| Tick latency | p50 ≤ 500 ms | **p99 alarm at 5 s** (PRD) | SLO and alert |

### 11.2 What we measure

All of it is aggregate-only and has no account, bird, or session identifiers.

- **RUM** (beacon on `pagehide`, with sampling):
  - `first-bird` ms (bucketed histogram)
  - TTFB, FCP, LCP-equivalent
  - boot and main chunk load times
  - frame-time histogram (30-second windows), long-task counts
  - audio context state outcomes (running / suspended-until-gesture / unavailable), worklet load failures, underrun counts
  - renderer adaptive-resolution steps
  - JS error counts by fingerprint
  - session-duration histogram (bucketed, computed client-side, no id)

  Dimensions are limited to browser family and major version, device class (mobile, tablet, desktop), coarse region (continent from the edge, never stored with an IP), and build id.
- **Server:** request counts, latency, and error rates by route (route template, never path parameters). Tick latency, lag, batch size, and errors. Event ingest rate (total, **not** broken down by interaction type, §11.3). Email send and bounce counts. DB and Redis health.
- **Integrity counters** (aggregate, must be zero or near zero): `drift_negative_clamped_total`, `drift_tick_clamp_hits_total`, `presence_interval_truncated_total`, `presence_rejected_visitor_total`, `personality_row_missing_total` (daily integrity job), `bird_identity_mutation_blocked_total`, `tick_fencing_aborts_total`.
- **Synthetic monitoring:** a headless browser fleet in 6 geographies runs every 5 minutes against **synthetic accounts**, whose drift we are free to inspect because they aren't users. It checks sign-in (through a test mailbox), first bird, sync, offer round-trip, visit link, and that the audio worklet loads.

### 11.3 What we deliberately don't measure

- Any per-account or per-bird metric in telemetry: drift distributions, mood distributions, offer counts per bird, listen-in durations, and "average drift across accounts" dashboards.
- Visit frequency, retention cohorts, streak-like statistics, DAU per account, or funnel analytics tied to identity. We track only aggregate counts of sign-ins, sessions started, and visit invites sent, with no per-account dimension.
- Interaction-type breakdowns of event ingest. That would amount to population-level analysis of how birds are interacted with.
- Product analytics SDKs. No third-party analytics, session replay, or heatmaps.

### 11.4 How it's enforced

- `@aviary/telemetry` is the only allowed emitter. Metrics are declared in a registry with typed, enumerated label values. At runtime, it rejects unknown labels and any value that matches UUID or email patterns. A CI lint bans direct use of vendor SDKs.
- **Logs:** structured, with a 14-day retention. They may contain `account_id` for operational debugging, since the PRD permits UUIDs in logs, but never email, event payloads, names, or state. A scrubber in the log shipper drops disallowed fields, and a staging job greps logs for email-shaped and personality-field-shaped content daily.
- **Errors:** error reporting is self-hosted or runs with a strict scrubber. It captures no request bodies, and bird names are redacted.
- **Network isolation:** the analytics warehouse, if one exists, ingests only from the telemetry pipeline. It has no credentials for, and no network route to, the simulation DB.

### 11.5 Drift calibration without looking at users

Because production drift can't be aggregated, calibration relies on three sources:
- the offline harness (§5.10)
- **consented staff dogfood accounts**, flagged `diagnostic_consent=true` at the DB level and inspected only by the drift owner through a separate internal tool (never through the product UI)
- synthetic prod accounts driven by bots with persona scripts

This is the only visibility into drift, and it is deliberate.

---

## 12. Testing and quality strategy

### 12.1 Automated

- `sim-core`:
  - Property tests (fast-check): monotonicity, ceilings, dt-invariance (one 10-minute step ≈ ten 1-minute steps within tolerance), union presence, attunement never affecting mood valence, cooldown gating.
  - Golden replay tests with fixed seeds.
  - The calibration harness (§5.10).
- **Sync:** Jepsen-lite concurrency tests with parallel ingest, tick races, worker kills mid-transaction, and duplicate events. The assertion is that every event is consumed exactly once and no personality write happens outside the tick.
- **Migrations:** the continuity test runs every migration against a fixture of 10k synthetic birds and asserts that ids, voiceprints, appearance seeds, and vectors are byte-identical before and after (I4).
- **Contract:** snapshot and API schemas are verified against a denylist that includes trait names and `email`.
- **E2E (Playwright)** on Chromium, Firefox, WebKit, and Android Chrome (via device farm). Scenarios: sign-in, adoption and naming, first-bird with no spinner (the DOM never shows a busy indicator, and frame 1 contains a bird), greeting, listen-in (engage and disengage paths), offer, settle and undo, notebook, visits (invite, bind, revoke, the 410 surface), presence truth table (simulated visibility, focus, idle), offline outbox, and session timeout.
- **Accessibility:** axe on every surface. Keyboard-only scripted journeys. Live-region cadence tests (count announcements over 10 virtual minutes: idle between 10 and 20, and priority items throttled). Reduced-motion snapshot tests assert that no transform animations exceed thresholds.
- **Copy lint:** the product-surface string catalog and the full prose-grammar expansion are checked against the banned lexicon and style rules. There's also a separate system-voice lint that checks capitalization and flags naturalist markers in system strings.
- **Visual regression:** first frame at 4 times of day, each weather state, reduced motion, and 8 viewports.
- **Audio:** the offline render suite (§8.8).

### 12.2 Manual, every release

- **Screen reader passes:** VoiceOver with Safari (macOS and iOS), NVDA with Firefox and Chrome, JAWS with Chrome, TalkBack with Chrome. Each follows a 20-minute scripted session and a "does it feel alive" rubric scored by the accessibility lead and at least one external tester who uses a screen reader.
- **Reduced-motion design review.**
- **Voice review:** the writer reviews 100 randomly generated notebook entries and narrations.

### 12.3 Studies (pre-launch)

- **Call recognizability panel:** 30+ participants. After a 10-minute familiarization, they identify a target bird by call in 3-, 5-, and 7-bird mixes across moods. The target is at least 80% accuracy at 7. This gates the birds-per-aviary ramp (§13.3).
- **Drift perception:** dogfood participants do blind A/B comparisons of their 3-week-old bird's current rendering against its day-1 rendering. The target is that at least 70% notice a difference at 3 weeks and at most 20% notice one at 1 week.
- **Uncanniness:** listening panels rate audio on a naturalness and pleasantness scale. Calls are iterated until the median is at least 4 out of 5 and no motif is flagged as "electronic" by more than 20% of raters.

---

## 13. Rollout

### 13.1 Milestones (team of about 9)

The team is 2 sim/backend, 2 rendering/frontend, 1 audio DSP, 1 accessibility and frontend, 1 infra/SRE, 1 writer, and 1 designer, plus a PM/lead.

| Milestone | Weeks | Exit criteria |
|-----------|-------|--------------|
| M0 Foundations | 1–2 | Repo, CI, environments, schema v1, telemetry registry, virtual clock, copy lint, no-toast design system seed. |
| M1 Sim core and harness | 1–6 | `sim-core` drift, mood, perch, weather, social, greeting and offer planners. The harness is green on the §5.3 table. Tick workers with fencing and ordering. |
| M2 Renderer and first frame | 2–8 | Canvas2D scene, 6 species rigs (2 at production quality by week 5), idle motion, choreography player, responsive layout invariant, **reduced-motion renderer**, first-bird under 500 ms in lab. |
| M3 Audio | 3–10 | Worklet synth, 6 species motif libraries, voiceprints, mixer, listen-in, captions hook, offline QA suite, first uncanniness panel. |
| M4 Accounts and sync | 2–8 | Magic link, sessions, edge SSR with snapshot, `/sync` outbox, presence monitor, timezone handling, export, deletion. |
| M5 Interactions | 6–11 | Greeting, listen-in, offer (all 3), settle and undo, wanderer and adoption, top bar and fade, system strip. |
| M6 Voice surfaces | 5–12 | Prose engine, notebook writer and UI, narration, captions text. Full lexicon lint. |
| M7 Visits | 10–13 | Invites, binding, visitor snapshot mode, revoke, log, opt-in notification. |
| M8 Hardening | 11–16 | Perf rig green, soak green, security review (auth, visitor scope), privacy review (telemetry audit, log scan, crypto-shred test), screen-reader passes, DR drill (restore from backup, verify vectors). |

Accessibility is not a milestone. It's a gate on M2, M5, M6, and M7 exit.

### 13.2 Release phases

1. **Dogfood alpha** (week 10 onward, at least 5 weeks). About 50 staff accounts on prod infra with diagnostic consent. It must run long enough to cross the 3-week visible-drift horizon. Presence window W, tick cadence, drift constants, notebook sparsity, and greeting feel are tuned here. Time-warped staff test aviaries (`age_override`, available only to staff accounts and compiled out of the user path) exercise 3–7-bird aviaries early.
2. **Closed beta** (about week 16). 1–2k invited users. Visits are enabled at week 18 behind a flag after abuse review of invite email volume. Kill switches are ready: `drift_apply`, `notebook_write`, `visits_enabled`, `new_signups`, and `max_birds_enabled`.
3. **Public launch** (about week 22). Open sign-up with a capacity gate on `new_signups`, adjusted from tick headroom metrics.

### 13.3 Ramping birds per aviary

- The engine cap is 7 (config constant `MAX_BIRDS=7`), and the age schedule is fixed (§5.9). A separate operational flag, `max_birds_enabled`, is 3 at launch, rises to 5 after the recognizability panel passes at 5, and rises to 7 after it passes at 7 and after perf-rig runs at 8 voices (7 plus a wanderer). In practice no real user reaches bird 4 until about 5 months after launch, so the flag is mostly a safety net that lets us hold if the audio work isn't ready. It is never presented to users. An aviary that is age-eligible while gated simply doesn't spawn a wanderer yet. Nothing is surfaced.
- If recognizability fails at 7, the PRD's cap stays as the engine limit, but the flag holds at the highest passing count until audio-mix work improves. This is the PRD's "we may revisit" path, used in the conservative direction.

### 13.4 Instrumented from day one

Everything in §11.2 goes live on the first deploy. Canary synthetic accounts run from the first staging deploy. The DR drill (backup restore and personality integrity check) runs before beta and quarterly after that.

---

## 14. Risks and mitigations

### Drift calibration

| Risk | Mitigation |
|------|-----------|
| **R-D1 Too fast.** Birds change visibly within a week and it feels Tamagotchi-like. | Harness assertion that the 1-day single-session Δ is ≤ 0.006, below the JND. Headroom saturation. Daily soft caps. The dogfood perception study. Kill switch `drift_apply`. |
| **R-D2 Too slow.** Nothing seems to matter. | 21-day visible band assertion. Dogfood A/B perception test. Constants are config, retunable within a release. |
| **R-D3 Homogenization.** All loved birds converge to the same maxed-out bird. | Per-bird ceilings, species sensitivities, individual seeds, and voiceprint-driven expression. The harness asserts that the inter-bird trait distance after 84 days of the Regular persona is at least 70% of the initial distance. |
| **R-D4 Presence inflation.** An automation or wiggler tool, or a laxer check slipping in during refactors. | The single `PresenceMonitor` module with truth-table tests and code ownership. Server clamps and cross-device union. Daily soft cap. A mouse jiggler is accepted as out of scope, since it's single-user and there is no competitive incentive. |
| **R-D5 Presence deflation on mobile.** Touch users watch without touching and lose presence after W. | Lean W long (4 minutes, calibrate up to 8). Measure in dogfood with a diagnostic comparison of mobile and desktop presence share. If mobile under-counts badly, bring a product decision (not a silent broadening) to widen W on touch devices. The definition itself stays intact. |
| **R-D6 Neglect reading as punishment** through attunement. | Attunement never touches mood valence (tested), has a floor so birds always call and greet, and recovers within a session or two. Writer review of how notebook entries after a long absence read. |
| **R-D7 A drift bug inflates values,** with no rollback allowed. | Pause and fix forward (§5.3). Harness gating on every sim change. Two-person review on `drift_params` and `sim-core`. |

### Sync correctness

| Risk | Mitigation |
|------|-----------|
| **R-S1 Lost events** from sequence gaps or cursor races. | The per-aviary serialized seq allocator (§6.2). An exactly-once consumption test under fault injection. |
| **R-S2 Double tick or stale writer.** | The fencing token. Lease TTLs. The aggregate `tick_fencing_aborts_total` counter. |
| **R-S3 Personality loss** from a migration, bug, or bad restore, which is invisible and catastrophic. | Role grants. Immutable identity columns. The migration continuity test. Daily vector backups. `personality_row_missing_total`. A DR drill that verifies vectors byte-for-byte. No code path deletes birds except hard delete. |
| **R-S4 Clock skew** misplaces scheduled actions. | Server-epoch schedules with an estimated offset. A 5-second minimum lead time on scheduled actions. Actions already started are never revised. |
| **R-S5 Timezone flip-flop** across devices. | The hysteresis rule (§6.6). |
| **R-S6 Snapshot cache staleness** after a Redis failover. | Versioned snapshots. The API falls back to building from the DB when the Redis version is behind the DB. Clients ignore older versions. |

### Audio

| Risk | Mitigation |
|------|-----------|
| **R-A1 Uncanny or "synthy" calls.** This is the biggest affective risk. | A DSP specialist from week 3. Species motif design references real bird-call structure (syllable types, trills, frequency sweeps). Naturalness panels with iterations. Procedural convolution reverb and distance filtering for placement. No pure sines. Every voice has noise and harmonic texture. |
| **R-A2 Repetition** that the ear notices. | Grammar variety plus per-call micro-variation, the uniqueness guard, an audit of motif-library size per species (at least 8 motifs), and Markov-style avoidance of the same motif back-to-back. |
| **R-A3 Recognizability lost** at 5–7 birds. | Voiceprint separation rules, spatial pan and depth, signature-syllable anchors, the MFCC classifier gate, human panels, and the `max_birds_enabled` hold. |
| **R-A4 Autoplay policy** prevents "calls already audible" on the first frame. | Visual call cues and captions continue. A mid-motion fade-in on the first gesture. No enable-sound overlay. This is accepted as a platform constraint and documented. |
| **R-A5 Safari and Firefox WebAudio quirks** (sample-rate mismatch, context interruption on iOS when the device locks or a call comes in, `interrupted` state). | Handle `statechange` and re-`resume()` on the next gesture. Resample-independent synthesis (all DSP parameterized by `sampleRate`). A device-farm audio smoke test. |
| **R-A6 CPU on low-end phones** with 8 voices plus reverb. | Worklet voice-count-aware quality levels (drop harmonic partials, shorter IR). Measured on the perf rig. |

### Accessibility regressions

| Risk | Mitigation |
|------|-----------|
| **R-X1 Narration spam or queue flood.** | Cadence and throttle tests in CI. A manual screen-reader pass per release. |
| **R-X2 Narration flattened into state lists** by a well-meaning contributor. | The narration grammar is owned by the writer. Lint bans perch numbers and mood labels. Tests assert no enumerations ("pip, wren, and ..." lists of more than 2 are rejected). |
| **R-X3 The canvas is inaccessible** or focus buttons drift from bird positions. | DOM focus proxies updated at 10 Hz. An e2e test asserts that the focus proxy's bounds intersect the rendered bird's bounds during flight. |
| **R-X4 Reduced-motion looks broken** or lags behind features. | It has its own renderer, QA baselines, and design review. Every new choreography primitive must ship with its reduced-motion pose mapping, enforced by a type-level requirement (`Primitive` requires a `stillnessPoses` field). |
| **R-X5 Top-bar fade hides focus or contrast.** | Never fades with focus inside. Restores on any key. Covered by tests. |

### Performance

| Risk | Mitigation |
|------|-----------|
| **R-P1 500 ms is unreachable on a cold high-RTT 4G connection** (DNS, TCP, and TLS alone can take about 4 RTT). | HTTP/3 with 0-RTT resumption, a global edge, inline snapshot, SW-cached shell on repeat visits, and a ≤ 45 KB boot chunk. The gate is defined on warm DNS for mid-tier 4G. Cold-start is tracked and reported separately. **This is an explicit interpretation to confirm with product.** |
| **R-P2 Memory growth from subtle leaks.** | The nightly soak gate. Pooled allocations. Frame-loop allocation lint. |
| **R-P3 Tick cost at scale.** | Batching, phase spreading, and aviary-sharded schema readiness. The capacity gate on sign-ups. |

### Product voice and privacy

| Risk | Mitigation |
|------|-----------|
| **R-V1 An announcement creep** ("just a small toast"). | No toast primitive exists. The copy lint. A product-principles review item on every UI PR. The system strip only accepts system-voice error codes from an enum. |
| **R-V2 Generic notebook prose.** | Writer-owned grammar, a specificity requirement (every template has at least one bird-specific and one moment-specific slot), and the sparsity bucket. |
| **R-V3 PII leakage** through logs, metrics, or error reports. | The telemetry registry, scrubbers, the daily staging log scan, crypto-shredding on delete, and email appearing only in the account and invite records. |
| **R-V4 Magic-link deliverability and scanner prefetch.** | POST-to-consume. A reputable transactional provider with SPF, DKIM, and DMARC. Monitoring of aggregate bounce rates. |
| **R-V5 Visit invites used for spam.** | Per-host invite rate limits (10 per day), a matter-of-fact template with no user-authored text, and one-click unsubscribe-from-invites for recipients. The suppression list is keyed by `HMAC(email)`. |

---

## 15. Decisions on PRD ambiguities

| # | Ambiguity | Decision | Rationale |
|---|-----------|----------|-----------|
| D1 | `aviary_layout.md` lists the top bar as four icons with "nothing else", but `interactions.md` and `accessibility_perf.md` put **settle** in the top bar. | The top bar has **five** items: the four named plus a quiet settle glyph (a small evening/moon mark). | Settle is specified as top-bar in two places and needs a keyboard-reachable home. Nesting it inside the offer menu would misclassify it. The addition is flagged for design sign-off. The rule against *other* additions stays absolute. |
| D2 | Presence activity: "pointermove or keypress". | Count `pointermove`, `pointerdown` (touch taps are pointer events), and `keydown`. Exclude wheel, scroll, orientation, and audio. | Stays faithful to the definition's intent while making touch devices workable. Anything broader is a product decision, not an engineering tweak. |
| D3 | "Traits never move down on neglect" versus neglected birds "greet less often". | A separate, non-personality **attunement** variable with a floor that only scales expressive frequency (§5.4). | It is the only way to satisfy both statements, and it keeps the personality vector strictly monotonic. |
| D4 | "Mood resets on a daily-ish cadence" versus mood persisting across sessions. | A soft server-side re-anchor at local dawn. Clients never reset mood. | Persistence across sessions holds, and "daily-ish" is satisfied without any visible snap. |
| D5 | Offer target: "offer the aviary" versus per-bird reactions and per-bird cooldown. | Placement is automatic (near the listened-in bird, else front-center). The server chooses respondents. The cooldown is per bird and invisible. | Keeps offers as gestures, not targeting UIs, and respects "not reached by clicking a bird". |
| D6 | The export includes "current personality vectors", but numbers are never exposed "in any surface". | The export includes the vectors in a `simulation_state` section of the JSON file, as the PRD's export spec explicitly lists them and data portability is a user right. No product UI ever renders them. | The export is a data-rights artifact, not a product surface. **Flag for product/legal confirmation.** |
| D7 | "Calls already audible" on the first frame versus browser autoplay policy. | Audible when the browser allows it. Otherwise a fade-in on the first gesture with visual call cues continuing, and never an enable-sound overlay. | Platform constraint. An overlay would violate "notice, never announce". |
| D8 | Audio while the tab is hidden. | Fade and suspend. | "Stops rendering" in spirit, saves battery, and avoids timer throttling glitches. Presence doesn't count while hidden anyway. |
| D9 | Where the settings live. | Captions, narration, and reduced-motion preference are per account (they follow the user). Mute and the shortcut toggle are per device. | The user's needs follow them. Mute is situational to a device. |
| D10 | Whether visitors see bird names (through narration). | Yes. Visitors see "exactly as it is". | The PRD prohibits special rendering for visitors. The host chose to share. |
| D11 | Offer placement input. | No manual placement. | The PRD rejects user arrangement of the scene. Auto-placement keeps the gesture light. |
| D12 | Offering while offline. | The offer UI still works. The item appears locally, but birds only glance until the connection returns. The event is queued. Its drift counts, but its mood effect is dropped if it's more than 10 minutes stale. | The server stays authoritative, and the user is never shown a broken affordance. |
| D13 | Localization. | English only in v1. The prose grammar is structured per locale for later. | Keeps v1 scope in check. Naturalist grammar is language-specific work. |
| D14 | "Visit" link: one-time versus ongoing access. | The link binds on first use to one browser. Access continues until revoked. Unused invites expire after 30 days. | "One-time link" and "revoke active invites" both hold. |
| D15 | New-bird arrival without announcing. | A wanderer that is present in the scene and is adopted by focusing it, with a notebook observation. No indicator anywhere. | Birds "arrive" rather than being acquired. It is noticed, not announced. |
| D16 | Tick cadence for long-dormant aviaries. | Every aviary ticks at the same cadence. | The PRD says the tick runs whether or not anyone is connected. Cost is manageable (§5.1). |
| D17 | Soft-deleted accounts. | The tick continues, visits are suspended, and on restore everything resumes. After 30 days the data is hard-deleted and the DEK destroyed. | A restored aviary should have continued without the viewer, like any other. |

---

## 16. Definition of done for v1

- All invariants I1–I11 have automated enforcement, and it is green.
- The calibration harness is green. The dogfood perception study meets its targets at 1 and 3 weeks.
- First bird is visible at ≤ 500 ms p75 in the lab profile. The 30-minute soak shows no growth. 60 fps holds on the reference laptop with 7 birds.
- The recognizability panel passes at the launched `max_birds_enabled` value.
- Screen-reader passes on 4 screen readers score "feels alive" at least 4 out of 5 from the accessibility lead and an external tester. The reduced-motion design review is signed off.
- The privacy audit is complete: telemetry registry reviewed, log scan clean for 7 days, crypto-shred deletion verified, warehouse isolation verified.
- No toast, spinner, counter, badge, or streak exists anywhere in the codebase (verified by lint and grep audit).
