# Pocket Aviary v1 — Implementation Plan

This plan is the executable brief for a separate engineering team. It interprets the PRD into architecture, contracts, algorithms, calibration targets, and delivery sequence. It does not restate the spec for its own sake. Where the PRD is silent, this plan makes a locked call and names it.

**Product:** Pocket Aviary — a browser-only, single-aviary-per-account place where two to seven birds live in one horizontal scene. Aliveness is the product. Presence is the primary interaction. The server is the only writer of canonical bird state.

**Non-goals are constraints, not backlog.** Native apps, gamification, Tamagotchi decay, and social-network surfaces are out of scope for v1 and must not appear as "harmless" extras in any milestone.

---

## 1. Scope

### 1.1 In v1

- Web client for last two major versions of Chrome, Safari, Firefox, and Edge.
- Email magic-link auth, per-device revocable sessions, email change with verify-first, 30-day soft delete then hard delete, on-demand JSON export.
- One canonical aviary per account. Two starter birds chosen by the system. Age-gated adoption up to seven. User-assigned renameable names. Stable bird IDs.
- Server-side simulation tick (~60s) that owns personality vectors, mood, perch intent, weather, and notebook candidacy.
- Client rendering of a single non-pannable scene: three perch zones, local-time day/night, rare weather, ambient ornaments, mid-action first frame.
- Interactions: return-greeting, idle presence, listen-in, offer (seed / song fragment / still pool), settle with 5s undo, field notebook (read-only, sparse, naturalist).
- Multi-device sync as a property of server-authored state, not a merge protocol.
- Visit invitations: off by default, per-invite email, read-only, revocable, 30-day unused expiry, silent visit log, opt-in email notify only.
- Accessibility shipping on day one: naturalist screen-reader narration, designed reduced-motion mode, runtime call captions, WCAG AA chrome, full keyboard path.
- Performance budgets: 2MB gzipped initial JS, <500ms first bird on mid-tier 4G, 60fps idle on a 5-year laptop, no client memory growth over 30 minutes.
- Operational telemetry only. No per-bird / per-account relationship data in any warehouse or training path.

### 1.2 Out of v1 (refuse explicitly)

From the PRD and `non_goals.md`, plus the implied feature class those refusals kill:

- Native iOS/Android clients; protocols designed around native constraints.
- Passwords, SSO, passkeys (defer; magic-link only).
- Payments, tiers, multi-aviary accounts, shared/household aviaries, customizable scenes.
- Quests, scores, streaks, badges, levels, XP, visit calendars, "days visited," "birds adopted" counters, milestone celebrations.
- Hunger, death, distress, decaying happiness, any neglect punishment.
- Profiles, follows, comments, chat, avatars, discovery, leaderboards, show-off visitor rendering, co-presence.
- Push notifications. No email about the aviary except: magic links, email-change verification, export download links, visit invite emails, and (only if the host opts in) visit-notification email.
- Recorded call libraries, audio loops, WebAudio→MP3 fallback.
- Personality numbers anywhere in UI, debug panels, ARIA, or client payloads.
- LLM-generated live copy on the request path (latency, voice drift, privacy).
- Public marketing site is not this product; this plan covers the app.

### 1.3 v1 quality bar (ship blockers)

Do not ship if any of these fail:

1. First painted frame is a quiet field or an already-moving aviary — never a spinner, never a fade-from-static.
2. Personality is server-only, additive, monotonic toward expressive, never last-write-wins.
3. Presence requires visibility ∧ focus ∧ recent input. "Tab open" is not presence.
4. No announcement toasts on return, visit, adoption, or drift.
5. Reduced-motion, captions, and narration ship with the aviary, not after.
6. Visitor sessions cannot write events or contribute presence.
7. Telemetry pipelines cannot read the simulation database.

---

## 2. Locked decisions (ambiguities resolved)

These are implementation law unless a later calibration note updates the numeric constants. The *shape* of each decision is not up for redesign during build.

| Topic | Decision |
|---|---|
| Tick cadence | 60s nominal. Adaptive catch-up when idle (see §6.8). Alarm if p99 compute > 5s. |
| Presence activity window | Start at **240s** since last `pointermove` or `keypress`. Lean long so watching without moving still counts. Calibrate only on dogfood/staging, range 180–300s. |
| Presence ping interval | Client emits at most every **30s** while all three conditions hold. Server drops pings <20s apart per session. |
| Multi-device presence | Union of honest windows, then **cap at 1.0 presence-second per wall-second per aviary**. Two focused devices cannot double-speed drift. |
| Trait domain | Each personality trait is a scalar in `[0, 1]`. Stored as `numeric(6,5)`. |
| Seed traits | Species prior ± `U(-0.06, +0.06)`, then clamped to `[0.12, 0.48]` at adoption so there is room to drift up and no bird starts maxed. |
| Mood enum | `wary`, `content`, `curious`, `drowsy`, `alert`. Five states, finalized. |
| Offer cooldown | **4 minutes per (bird, offer_kind)**. Functional, not punitive. Enforced server-side. |
| Song library | **8 score fragments** (note events + articulations), not audio files. Shared with the call synth. |
| New-bird age gates | Aviary age, not engagement. 3rd @ 70 days; 4th @ 150 days; 5th @ 240 days; 6th @ 365 days; 7th @ 540 days. Offer is a quiet arrival, not a catalog. |
| Greeting latency | Client-side greeting director using server-provided `greet_weight` + absence bucket. Must fire within 1–2s of first interactive frame. No waiting on the tick. |
| Personality on the wire | **Never.** Client receives quantized behavior hints and visual params only. Full vectors appear only in account export and the simulation DB. |
| Snapshot transport | HTTPS JSON pull. No WebSocket in v1. Pull on visible, on long frame gap (>2s), and keepalive every **25s** while visible. |
| Renderer | 2D canvas scene + DOM hit-target overlay for focus/keyboard/captions. Top bar and all system surfaces are DOM. |
| Narration / notebook / captions | Deterministic prose composer from templates + state slots. No live model calls. |
| Audio unlock | No "click for sound" modal. Resume `AudioContext` on first pointer/key. Until then: visual aviary + captions if enabled. Silence is preferable to a toast. |
| Settle chrome | Fifth top-bar icon. Layout PRD listed four icons; interactions require settle from the top bar. Sparse fifth icon beats burying settle. |
| Visitor chrome | Scene + calls + day/night + weather only. No offer, settle, notebook, account, or listen-in. |
| Visit notify opt-in | Email only, off by default, not shown in onboarding. Still no push. |
| i18n | English only in v1. Naturalist lowercase is English-specific. |
| Timezone | IANA zone from client on session start / tzchange. Tick uses last-known zone for diurnal mood. Default `Etc/UTC` until first session. |
| Simultaneous greetings | One primary greeter. Additional birds may react with 180–900ms stagger, never unison. |
| Empty aviary | Only during first adoption, before first fly-in. After that the scene is never empty. |
| Notebook sparsity | Target **1 entry / 2–4 days** for a regularly present aviary. Hard cap **1 / 18 hours** even if noteworthy. Never write user-behavior observations. |
| Stack | TypeScript monorepo, Postgres, Redis, object storage, transactional email. Shared packages for schema, simulation, call grammar, prose. |

---

## 3. Architecture

### 3.1 Design thesis

The product's central conceit — *the aviary has been continuing without you* — is an architectural property, not a visual trick.

- **Server** owns time, personality, mood, weather, notebook candidacy, and visit authorization.
- **Client** owns pixels, interpolation, idle micro-motion ornaments, procedural audio, presence sensing, and interaction emission.
- **Clients never tick.** A closed laptop and a pocketed phone do not each hold a private universe.

If a design discussion starts with "we could simulate this in the tab and sync later," stop. That path is how personality is lost.

### 3.2 Service shape

One logical product, four deployables, one database of record.

```
                    ┌─────────────────────────────────────────┐
                    │              CDN / edge                 │
                    │  static JS/CSS/SVG, long-cache hashed   │
                    │  HTML: private, no-store                │
                    └──────────────────┬──────────────────────┘
                                       │
┌─────────────┐   HTTPS JSON    ┌──────▼──────┐   SQL    ┌──────────────────┐
│  Web client │◄───────────────►│   api       │◄────────►│  Postgres        │
│  (browser)  │  events/snap    │  (stateless)│          │  accounts,       │
└──────┬──────┘                 └──────┬──────┘          │  aviaries, birds,│
       │                               │                 │  events, notebook│
       │ WebAudio                      │ Redis           │  visits, jobs    │
       │ local only                    │ sessions,       └────────▲─────────┘
       │                               │ rate limits,             │
       │                               │ tick leases              │
       │                        ┌──────▼──────┐                   │
       │                        │  tick worker│───────────────────┘
       │                        │  60s loop   │
       │                        └──────┬──────┘
       │                               │
       │                        ┌──────▼──────┐     ┌──────────────┐
       │                        │  mailer /   │────►│  SES/Postmark│
       │                        │  job worker │     └──────────────┘
       │                        └──────┬──────┘
       │                               │
       │                        ┌──────▼──────┐
       │                        │  object     │  exports, (no audio assets)
       │                        │  storage    │
       └────────────────────────┴─────────────┘
```

No analytics warehouse connection to Postgres simulation tables. Operational metrics emit from api/tick as *aggregates* to the metrics backend. That is a network ACL and a credential split, not a policy paragraph.

### 3.3 Monorepo map

```
apps/web/                 // client
apps/api/                 // HTTP, auth, snapshots, events, settings, visits
apps/tick/                // simulation worker
apps/jobs/                // export, hard-delete, invite expiry, mail send
packages/schema/          // zod/types: events, snapshots, export document
packages/simulation/      // pure functions: drift, mood, perch, weather, catch-up
packages/call-grammar/    // motifs, variation, caption phrases, chorus rules
packages/prose/           // notebook + narration + caption sentence assembly
packages/species/         // six-species silhouettes, palettes, grammar ids, priors
```

`packages/simulation` must run in CI without I/O. The tick worker is a thin loop around pure functions. That is how drift calibration stays testable without reading production accounts.

### 3.4 Client / server split

| Concern | Owner | Notes |
|---|---|---|
| Account, sessions, email | api | Synthetic account UUID everywhere except the encrypted email column |
| Personality vector | tick only | Additive deltas from event log |
| Mood, perch intent, weather | tick | Snapshot carries current + end times |
| Notebook entries | tick / jobs | Written rarely; clients only GET |
| Presence detection | client | Three-signal conjunction; server validates shape and rate |
| Greeting performance | client | Uses `greet_weight`, absence bucket, session nonce |
| Offer resolution | api (sync ack) + client animation | Server picks receiver + reaction class; client plays it |
| Call waveforms | client WebAudio | Grammar params from snapshot |
| Leaf/feather ornaments | client only | Not in simulation state |
| Visit authorization | api | Visitor token → read-only snapshot, event POST rejected |
| Render pause when hidden | client | Cancel rAF; keep nothing hot except a visibility listener |

### 3.5 Render-pipeline boundary

The snapshot is a *state photograph*, not a frame stream.

Server sends: who the birds are, where they intend to be, what mood they are in, how saturated they look, how vocal they are allowed to be, what weather/light is active, whether an offer is cooling down, and enough motion phase that the client can start mid-action.

Client does: lerp/flight between perch intents, mood-shaped idle cycles (or reduced-motion crossfades), ambient ornaments, audio graph, listen-in mix, settle lighting override, hit-testing.

If a piece of state would diverge across two honest clients watching the same account, it belongs on the server. If it is an ornament that no second client needs to share (a particular leaf), it stays on the client.

### 3.6 Trust boundary

- Session cookie: `HttpOnly`, `Secure`, `SameSite=Lax`, 30-day idle sliding window, rotated on privilege actions (email change, delete).
- CSRF: double-submit or `Origin` check on mutating routes. Prefer `Authorization: Bearer` in memory + cookie refresh token if it keeps the bundle small; either way, state-changing GETs are forbidden.
- Visit links are unguessable 256-bit tokens, stored as SHA-256 hashes.
- Magic links same: 15-minute TTL, single consume, hashed at rest.
- All logs, traces, and metrics keyed by `account_id` UUID. Email is not a log field. A CI lint fails the build if `email` appears in logger attribute allowlists outside `apps/api` auth mailer.

### 3.7 Why not client-tick + CRDT

A laptop closed at 09:00 and a phone opened at 21:00 would each owe the user a day of life. Merging two drift integrals is how you silently delete a morning. Additive server deltas on an ordered log make that failure unreachable. v1 will not grow a client simulation "for snappiness" except for the greeting director and audio scheduler, both of which are *presentational*.

---

## 4. Data model

Postgres is the system of record. Redis is ephemeral (sessions cache, rate limits, tick locks). Do not put personality in Redis as source of truth.

### 4.1 Identifiers

- `account.id` — UUID v4, generated at insert. This is the only account key used in FKs, logs, traces, queues, object keys, and rate-limit buckets.
- `account.email_ciphertext` — application-encrypted, one row, one column.
- `account.email_lookup_hash` — HMAC-SHA256 of NFKC-normalized lowercase email with a dedicated key. Used only for login lookup.
- `bird.id` — UUID v4, stable for the life of the account. Never recycled. Rename, species-pool art updates, and migrations must not allocate a new bird id.

### 4.2 Core tables

```text
accounts
  id                         uuid pk
  email_ciphertext           bytea not null
  email_lookup_hash          bytea not null unique
  email_pending_ciphertext   bytea null
  email_pending_hash         bytea null
  timezone                   text not null default 'Etc/UTC'
  visit_notify_email         boolean not null default false
  captions_opt_in            boolean not null default false
  reduced_motion_opt_in      boolean null  -- null = follow OS
  created_at                 timestamptz not null
  marked_for_deletion_at     timestamptz null
  deleted_at                 timestamptz null           -- hard-delete marker during wipe
  last_seen_at               timestamptz null           -- operational only; never surfaced

sessions
  id                         uuid pk
  account_id                 uuid not null references accounts
  token_hash                 bytea not null unique
  device_label               text not null              -- parsed UA, matter-of-fact
  created_at                 timestamptz not null
  last_seen_at               timestamptz not null
  revoked_at                 timestamptz null

magic_links
  id                         uuid pk
  email_lookup_hash          bytea not null
  account_id                 uuid null                  -- null if first-time
  purpose                    text not null              -- sign_in | verify_email
  token_hash                 bytea not null unique
  expires_at                 timestamptz not null
  consumed_at                timestamptz null
  created_at                 timestamptz not null

aviaries
  id                         uuid pk
  account_id                 uuid not null unique references accounts
  created_at                 timestamptz not null        -- age-gate source
  last_tick_at               timestamptz not null
  last_event_seq             bigint not null default 0
  weather_kind               text not null default 'clear'  -- clear|rain|wind
  weather_until              timestamptz null
  lighting_phase             text not null              -- derived cache
  bird_offer_available_at    timestamptz null           -- when next arrival may be accepted
  next_bird_slot             int not null default 3     -- 3..7
  last_notebook_at           timestamptz null
  last_host_presence_at      timestamptz null           -- greeting/attunement, not a trait

birds
  id                         uuid pk
  aviary_id                  uuid not null references aviaries
  species_id                 text not null
  name                       text not null
  adopted_at                 timestamptz not null
  sort_index                 int not null               -- stable tab order
  -- personality (tick-only writers)
  boldness                   numeric(6,5) not null
  social_warmth              numeric(6,5) not null
  vocal_frequency            numeric(6,5) not null
  plumage_saturation         numeric(6,5) not null
  curiosity                  numeric(6,5) not null
  personality_rev            bigint not null default 0  -- increments on any delta
  -- fast state
  mood                       text not null
  mood_since                 timestamptz not null
  perch_zone                 text not null              -- front|middle|back
  perch_slot                 int not null
  motion_clip                text not null
  motion_phase               real not null              -- 0..1, for mid-action resume
  call_seed                  bigint not null            -- stable, from bird id
  last_offer_at              jsonb not null default '{}'  -- {seed,song,pool}: iso
  unique (aviary_id, name)                              -- names unique per aviary

interaction_events          -- append-only
  seq                        bigserial primary key      -- global order the tick consumes
  id                         uuid not null unique       -- client idempotency key
  account_id                 uuid not null
  aviary_id                  uuid not null
  session_id                 uuid null
  bird_id                    uuid null
  kind                       text not null
  payload                    jsonb not null
  client_time                timestamptz not null
  received_at                timestamptz not null default now()
  -- no updates, no deletes except hard-delete wipe

notebook_entries
  id                         uuid pk
  aviary_id                  uuid not null
  written_at                 timestamptz not null
  body                       text not null              -- final prose, lowercase
  trigger                    text not null              -- internal enum, never shown
  unique-enough sparsity is enforced in tick, not by unique constraints

visit_invites
  id                         uuid pk
  host_account_id            uuid not null
  aviary_id                  uuid not null
  visitor_email_ciphertext   bytea not null
  visitor_email_hash         bytea not null
  token_hash                 bytea not null unique
  created_at                 timestamptz not null
  expires_at                 timestamptz not null        -- created_at + 30d
  consumed_at                timestamptz null
  revoked_at                 timestamptz null

visit_sessions
  id                         uuid pk
  invite_id                  uuid not null
  started_at                 timestamptz not null
  last_snapshot_at           timestamptz not null
  ended_at                   timestamptz null
  approx_duration_s          int not null default 0     -- updated on snapshot pull

jobs
  id                         uuid pk
  kind                       text not null              -- export|hard_delete|mail|invite_expire
  account_id                 uuid null
  payload                    jsonb not null
  run_after                  timestamptz not null
  locked_at                  timestamptz null
  done_at                    timestamptz null
  error                      text null
```

Indexes: `interaction_events (aviary_id, seq)`, `interaction_events (id)`, `notebook_entries (aviary_id, written_at desc)`, `visit_invites (host_account_id, created_at desc)`, `magic_links (token_hash)`, `sessions (token_hash)`, `accounts (email_lookup_hash)`, `aviaries (last_tick_at)` for the worker.

### 4.3 Event kinds

```text
session_start        { absence_s, tz, viewport_class }
session_end          { reason: hide|unload|settle }
presence             { visible, focused, activity_s_ago }   -- all must be true or drop
listen_in_start      { bird_id }
listen_in_end        { bird_id, duration_s }
offer                { kind: seed|song|pool, fragment_id? }
offer_resolved       -- server-written companion? no: resolution lives in ack, not log
settle               { undone: boolean }
rename               { bird_id, name }   -- also a direct PATCH; event is audit for notebook
adopt                { bird_id }         -- server emits after insert
```

Clients may send only: `session_start`, `session_end`, `presence`, `listen_in_start`, `listen_in_end`, `offer`, `settle`. Rename and adopt are API commands that the server journals if useful to the composer.

Visitor tokens: any of the above → `403`.

### 4.4 Snapshot document (client-facing)

Never include raw personality, email, or other accounts' data.

```ts
type Mood = "wary" | "content" | "curious" | "drowsy" | "alert";
type Zone = "front" | "middle" | "back";
type OfferKind = "seed" | "song" | "pool";

type BehaviorHints = {
  greet_weight: number;          // 0..1, already folded with mood + recency
  approach: "near" | "hesitant" | "distant";
  investigate: "eager" | "mild" | "ignore";
  vocal: "sparse" | "moderate" | "frequent";
  cluster: "alone" | "near_others";
};

type BirdSnap = {
  id: string;
  name: string;
  species_id: string;
  mood: Mood;
  perch: { zone: Zone; slot: number; x: number; y: number };
  motion: { clip: string; phase: number };
  plumage: { sat: number };      // 0..1 visual only
  call: { grammar_id: string; seed: string; vocal_n: number };
  offer_cool_until: Partial<Record<OfferKind, string>>;
  behavior: BehaviorHints;
};

type AviarySnapshot = {
  snapshot_id: string;
  sim_time: string;
  generated_at: string;
  role: "host" | "visitor";
  aviary: {
    id: string;
    created_at: string;
    lighting: { phase: "dawn"|"morning"|"midday"|"evening"|"night"; settle_suggested: false };
    weather: { kind: "clear"|"rain"|"wind"; until: string | null };
    arrival: { ready: boolean } | null;     // third+ bird waiting; no species catalog
  };
  birds: BirdSnap[];
  can_interact: boolean;
};
```

`settle` lighting is **not** in the canonical snapshot. It is a local session overlay. Other devices should not dim because this device said goodbye.

### 4.5 Export document

Generated on demand, emailed as a 24h signed URL. Includes personality vectors because the PRD says the relationship is the user's to copy. This is the only user-facing surface that contains numbers, and it is a file, not a screen.

```json
{
  "exported_at": "ISO",
  "account_id": "uuid",
  "timezone": "IANA",
  "aviary": { "id": "uuid", "created_at": "ISO" },
  "birds": [
    {
      "id": "uuid",
      "name": "pip",
      "species_id": "warbler",
      "adopted_at": "ISO",
      "personality": {
        "boldness": 0.41,
        "social_warmth": 0.37,
        "vocal_frequency": 0.33,
        "plumage_saturation": 0.29,
        "curiosity": 0.36
      },
      "mood": "content"
    }
  ],
  "notebook": [{ "written_at": "ISO", "body": "..." }],
  "settings": { "visit_notify_email": false, "captions_opt_in": false }
}
```

Export omits raw event logs (presence pings are not a souvenir) and omits visitor emails of other people beyond the host's own visit log if we include it — **do not include visitor emails in export** to reduce spreading third-party PII. Include invite status as "invite outstanding / revoked" counts only.

### 4.6 Species pool (v1)

Six species, one coherent temperate-edge set. No rarity. Starter pair is sampled for contrast (different silhouettes, different motif families), never two nightjars as the only pair.

| `species_id` | Silhouette | Motif family | Night active | Trait prior (b, w, v, p, c) |
|---|---|---|---|---|
| `warbler` | slim, tail-flick | high short phrases | no | 0.34, 0.32, 0.40, 0.36, 0.38 |
| `wren` | tiny, cocked tail | rapid stacked notes | no | 0.30, 0.36, 0.44, 0.28, 0.40 |
| `sparrow` | compact, round head | simple chip pairs | no | 0.28, 0.40, 0.30, 0.26, 0.30 |
| `finch` | conical beak | clear tonal steps | no | 0.32, 0.30, 0.36, 0.40, 0.34 |
| `dove` | fuller body | low two-note | no | 0.22, 0.34, 0.18, 0.24, 0.22 |
| `nightjar` | long wing, low perch | hollow churr | **yes** | 0.20, 0.22, 0.28, 0.22, 0.26 |

Display in product prose as common names in lowercase ("a warbler", "a nightjar"). Users meet names, not a dex.

### 4.7 Derived attunement (not a personality trait)

`aviaries.last_host_presence_at` plus a tick-computed `recency_factor ∈ (0,1]` implement "quieter after absence" without downward trait drift.

```
recency_factor = exp(-ln(2) * hours_since_presence / 72)
```

Half-life 72 hours. After two weeks the factor is small; greetings become glances; chorus thins. After twenty minutes of honest presence in a new session it recovers toward 1. Recovery is *fast-timescale* and is not written into the five traits.

This is the mechanical expression of "leave for two weeks and find birds that are quieter, not birds that mistrust you."

---

## 5. API surface

Base: `/v1`. JSON. All host routes require a session. Matter-of-fact error bodies. No naturalist voice on 4xx/5xx.

### 5.1 Auth

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/auth/magic-link` | Body `{ email }`. Always return `202` with generic `{ ok: true }` (no account oracle). Rate limit 5 / 15 min / email hash, 20 / hour / IP. |
| `GET` | `/v1/auth/callback?token=` | Consume magic link. Set session. Used links 410. Expired 410. Redirect to `/` or `/adopt` if new. |
| `POST` | `/v1/auth/sign-out` | Revoke this session. |
| `GET` | `/v1/account` | Settings blob. No personality. |
| `PATCH` | `/v1/account` | `{ timezone?, visit_notify_email?, captions_opt_in?, reduced_motion_opt_in? }` |
| `POST` | `/v1/account/email` | Start change; mail new address. Old email works until verify. |
| `GET` | `/v1/account/sessions` | Device list. |
| `DELETE` | `/v1/account/sessions/:id` | Revoke. |
| `POST` | `/v1/account/export` | Enqueue export; `202`. |
| `POST` | `/v1/account/delete` | Soft-delete now. |
| `POST` | `/v1/account/undelete` | Allowed only if `marked_for_deletion_at` within 30 days. |

Magic-link copy and all of the above UI: matter-of-fact voice.

Example errors (verbatim register):

- `We couldn't sign you in. The link may have expired. Try requesting a new link.`
- `Your session timed out. Sign in again to keep watching.`
- `Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch.`

### 5.2 Aviary and events

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/aviary/snapshot` | Canonical snapshot. `ETag` = `snapshot_id`. |
| `POST` | `/v1/aviary/events` | Batch 1–50 events. Idempotent on `id`. Returns `{ accepted: uuid[], offer_results?: OfferResult[] }`. |
| `GET` | `/v1/notebook?before=&limit=` | Reverse chrono. Default 30. No mutation. |
| `PATCH` | `/v1/birds/:id` | `{ name }` only. 1–24 chars, trimmed, unique in aviary. |
| `POST` | `/v1/aviary/adopt` | Accept age-gated arrival. Server picks species. 409 if not ready or at cap. |

`OfferResult` (immediate, so the client can animate without waiting for the next tick):

```ts
type OfferResult = {
  event_id: string;
  bird_id: string;
  kind: OfferKind;
  reaction: "approach" | "wait_then_near" | "ignore" | "drink" | "bathe" | "watch" | "join" | "quiet" | "countercall";
  cooldown_until: string;
};
```

Receiver selection (server):

1. Eligible birds = not on cooldown for that kind.
2. Score = `curiosity` + mood bias (`curious` +0.25, `content` +0.1, `drowsy` −0.25, `wary` −0.15) + proximity bias (front +0.1).
3. Pick argmax with tiny jitter (`0.02`) so it is not robotic.
4. If none eligible, return `reaction: "ignore"` with `bird_id` of the closest bird and no cooldown write — client shows the gift resting unused.

### 5.3 Visits

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/visits` | `{ email }`. Creates invite, sends one-time link. 429 if >10 outstanding. |
| `GET` | `/v1/visits` | Outstanding invites + visit log (email, day, approx duration). Settings surface. |
| `DELETE` | `/v1/visits/:id` | Revoke immediately. |
| `GET` | `/v1/visit/:token/snapshot` | Read-only snapshot. No cookies required. Token in path is the secret. |
| `POST` | `/v1/visit/:token/heartbeat` | Updates `last_snapshot_at` / duration. **Does not** write `interaction_events`. |

Revoked, expired, or unused-past-30d: `410` with `This visit is no longer available.`

Visitor snapshot: `role: "visitor"`, `can_interact: false`, no `arrival`, no cooldowns, no notebook route.

### 5.4 What is not an API

- No `PUT /birds/:id/personality`.
- No `GET /stats`.
- No `GET /streaks`.
- No public `GET /explore`.
- No websocket `/live`.
- No client-submitted absolute perch coordinates.

### 5.5 Event POST contract

```ts
type ClientEvent = {
  id: string;                 // uuid, client-generated
  kind: "session_start" | "session_end" | "presence" | "listen_in_start" | "listen_in_end" | "offer" | "settle";
  client_time: string;        // ISO
  bird_id?: string;
  payload: Record<string, unknown>;
};
```

Server behavior:

1. If `id` exists, return it in `accepted` without inserting (idempotent).
2. Drop malformed events (missing conjunction on presence, unknown bird, visitor token).
3. Clamp `client_time` into `[now-10m, now+30s]`. Outside that, store `received_at` as the effective time and ignore client_time for drift.
4. Do not apply personality here.

### 5.6 Snapshot pull triggers (client)

1. First load (and adoption complete).
2. `visibilitychange` → `visible`.
3. `pageshow` after bfcache.
4. rAF gap > 2000ms (sleep/resume).
5. Every 25s while `visibilityState === "visible"` (even without presence — lighting/weather should move).
6. After an offer ack, merge `OfferResult` locally; do not wait for keepalive to animate.

Hidden tabs do not poll.

---

## 6. Simulation engine

The tick is a single-writer loop per aviary. Implementation lives in `packages/simulation` with the worker applying results.

### 6.1 Loop

```
every 5s:
  claim up to N aviaries where last_tick_at < now-60s
    using FOR UPDATE SKIP LOCKED
  for each aviary:
    events = events where seq > last_event_seq
    state' = step(state, events, now)
    write birds, aviary, optional notebook
    last_tick_at = now
    last_event_seq = max seq consumed
```

Deleted (soft) aviaries are skipped. Hard-delete job removes rows later.

Concurrency: one lease per aviary (`pg` row lock is enough; Redis lock optional). Never two ticks on one aviary.

### 6.2 Inputs the tick is allowed to read

- Current bird rows (vectors, mood, perch, cooldowns).
- New events in seq order.
- Aviary age, weather, last presence, last notebook time.
- Account timezone.
- Clock (`now`).

The tick must **not** read other accounts. The tick must **not** write telemetry containing bird fields.

### 6.3 Personality drift

Low-pass, slow, **monotonic non-decreasing**, asymptotic toward 1.

```
headroom(x) = 1 - x

presence_minutes = honest_presence_seconds / 60
  after 1x realtime cap across sessions

# dominant
Δp = α_p * presence_minutes * headroom(trait)

# listen-in (that bird only)
Δw += α_w * listen_minutes * headroom(social_warmth)
Δv += α_v * listen_minutes * headroom(vocal_frequency)

# offers
if offer near bird (event exists):  Δb += α_b * headroom(boldness)
if reaction in {approach, join, drink, bathe}: Δc += α_c * headroom(curiosity)

settle: no Δpersonality
neglect: no negative Δ
```

**Calibration constants (starting values):**

```
α_p = 0.00075    # per presence-minute, applied to warmth, vocal, plumage, curiosity, and 0.6×boldness
α_w = 0.0016     # listen-in → social_warmth
α_v = 0.0016     # listen-in → vocal_frequency
α_b = 0.008      # per near-offer event
α_c = 0.010      # per accepted-offer event
```

**Targets to lock in tests with a synthetic week:**

- Fixture: 5 sessions × 8 minutes honest presence, one 90s listen-in on bird A, two accepted offers on bird A, none on bird B.
- After 7 simulated days: at least one trait on A moved by **≥ 0.02** and **≤ 0.06**. B moves less than A. No trait decreases.
- Single 20-minute session: every trait Δ **< 0.015** (not user-visible).
- Three such weeks: some trait Δ **≥ 0.10** (user-visible perch/greeting/plumage).
- Two weeks of zero events: trait vector **bit-identical**. Mood may change. `recency_factor` falls.

If dogfood reports session-to-session visible change, shrink `α_*` by 25% rather than adding decay.

Plumage is the slowest visual: apply `0.7 * α_p` only. Saturation must not jump a palette step in one evening.

### 6.4 Ambient vs expressive (no Tamagotchi)

Do not implement "lonely → wary." Wary is a weather/alarm/mood state, not a moral judgment.

Expressiveness the user *sees* is:

```
greet_weight = clamp01(
  0.45 * boldness
+ 0.35 * social_warmth
+ 0.20 * recency_factor
+ mood_bias
)
vocal_n = vocal_frequency * recency_factor * weather_vocal_mul * diurnal_mul * mood_vocal_mul
```

A neglected high-vocal bird is still that bird; the world is just quieter until presence returns.

### 6.5 Mood

Persists across sessions. No snap-to-neutral on tab open.

**Diurnal prior (local hour):**

| Local hour | Prior |
|---|---|
| 05–08 | `alert` |
| 08–11 | `curious` |
| 11–16 | `content` |
| 16–19 | `content` with pull to `drowsy` |
| 19–22 | `drowsy` |
| 22–05 | `drowsy` (nightjar: `alert` or `curious`) |

Each tick, compute transition scores:

```
score[m] = prior[m]
         + interaction[m]
         + weather[m]
         + contagion[m]
         + personality_gate[m]
```

- Offer accepted → `content` +0.4, `curious` +0.2.
- Listen-in → `curious` +0.2, `alert` +0.1.
- Rain → `drowsy` +0.2, all vocal muls 0.65, `alert` −0.1.
- Wind → `alert` +0.2 or `wary` +0.15 (low boldness prefers wary).
- Neighbor `wary` → own `wary` +0.25 * (1 - boldness).
- High boldness: `wary` score × (1 - 0.6*boldness).
- High curiosity: `curious` +0.15.

Pick argmax. Hold at least **12 minutes** unless a strong event (`offer` or neighbor alarm) fires. This prevents flicker.

Daily-ish reset: at local 04:30, blend 30% toward diurnal prior if no events in the last 6 hours. Not a hard reset.

### 6.6 Perch intent

Three zones × 3 slots (enough for 7 birds without overlap). Server assigns unique `(zone, slot)`.

```
zone_score(front)  = boldness + mood_front - wary*0.5 - drowsy*0.3
zone_score(back)   = (1-boldness) + wary*0.4 + drowsy*0.3
zone_score(middle) = 0.35 + curiosity*0.2
```

High `social_warmth` adds a bonus to slots adjacent to occupied slots. Assignment is greedy by descending `|preference|` so the boldest claim front first.

Client flies over 8–20s (species-scaled). If snapshot gap > 5 minutes, spawn already on the new perch mid-clip — the move happened while away.

Users never place birds.

### 6.7 Bird-to-bird

Slow (tick):

- Wary contagion (above).
- Chorus window flag: if ≥2 birds have `vocal_n > 0.45` and weather is not rain, set `chorus_bias` for the next 2–4 minutes (carried as a slightly raised `vocal` hint).

Fast (client grammar, §9):

- Answer calls within 200–800ms when warmth/vocal high.
- Never phase-lock identical motifs.

The aviary is a small social system, not seven independent timers.

### 6.8 Catch-up (absence of days)

Do **not** run 20,000 full ticks after a two-week gap.

```
if no unprocessed events and now - last_tick_at > 3 minutes:
  jump weather via seeded RNG over calendar days (expected ~3 rains/week, ~2 winds/week)
  set mood from diurnal prior + weather + nightjar exception
  recompute perch intent once
  recency_factor from last_host_presence_at
  personality unchanged
  write last_tick_at = now
```

If events exist, process them in order against the state *as of each event's effective time* (piecewise), then jump the empty tail. This keeps listen-in that happened at 09:12 from being applied on top of tonight's mood incorrectly.

Seeded weather: `rng = hash(aviary_id, yyyy-mm-dd, hour_bucket)` so two snapshot pulls agree.

### 6.9 Call-grammar runtime (shared package)

A call is not a file. It is:

```ts
type Motif = { notes: { interval: number; beats: number; artic: "soft"|"sharp"|"trill" }[] };
type Grammar = {
  id: string;
  tessitura: number;          // Hz center
  scale: number[];            // allowed intervals
  motifs: Motif[];            // 6–12 per species
  timbre: "thin"|"reed"|"hollow"|"round";
};
```

Variation at emit time (must make consecutive calls unequal):

- Motif choice from weighted set, excluding the last two used by that bird.
- Tempo × `U(0.92, 1.08)` × mood (`drowsy` 0.85, `alert` 1.08).
- Pitch × `U(-25, +25)` cents, plus a stable bird offset from `call_seed` (±150 cents) so Pip is recognizable.
- Optional second motif glue (20% if `vocal` frequent).

Recognizability rule: species motif family + bird pitch offset + timbre stay fixed for the life of the bird. Mood and drift change *rate and energy*, not identity.

Server uses the same function to build caption phrases. Client uses it to schedule oscillators. If they disagree, captions lie — share the package and a golden test vector per `(grammar_id, seed, t)`.

### 6.10 Return-greeting director (client, using tick outputs)

Absence buckets from `session_start.absence_s` or `now - last_hidden`:

| Absence | Form |
|---|---|
| < 5 min | Glance / head-tilt, no call required |
| 5–120 min | Quiet two-note from the primary greeter |
| 2–48 h | Step toward front if approach ≠ distant; longer call; possible one staggered answer |
| > 48 h | Re-orientation: primary greeter more complete phrase; others do **not** chorus on cue |

Primary greeter = highest `greet_weight` with random 5% upset so Wren can sometimes beat Pip. If top weight < 0.15, *nobody* performs a big greeting — a glance from the least-distant bird only. That is how quiet aviaries stay quiet.

Procedural variation: seed `(bird_id, session_nonce, date)`. Ban last session's exact motif index + duration pair.

Stagger additional reactions 180–900ms. Never simultaneous arrival flourish.

### 6.11 Adoption and identity

New account path (api, not tick):

1. Create account + empty aviary + two birds.
2. Species: sample two different rows from the pool, preferring contrast on tessitura and silhouette. Never two of the same species as starters.
3. Names: suggest from a small per-species list (`pip`, `wren` as examples only — do not hardcode those two as the only pair). User may edit before confirm.
4. Empty quiet field → first bird soft fly-in → second staggered 1.2–2.0s → greeting director with absence = ∞ treated as first meeting (gentle, not parade).

Later arrivals: `POST /v1/aviary/adopt` when `arrival.ready`. System picks a species not already in the aviary if possible, else allow duplicate species (still a new bird id, new seed, new vector). Fly-in once. Never a catalog grid.

Identity migrations: additive columns only. If a grammar id is renamed, keep a map. Never `DELETE FROM birds` except hard-delete account.

### 6.12 Notebook composer

Runs at end of tick when:

- `now - last_notebook_at ≥ 18h`, and
- one trigger fires, and
- a randomness gate keeps typical cadence near one entry / 2–4 days (`p = 0.35` on ordinary days, `p = 1` on strong triggers).

**Allowed triggers:** first greeter identity changed vs last 7 days; long quiet morning (low vocal_n and rain or drowsy cluster); weather just ended; new bird adopted; unusual perch (historically back bird on front); nightjar calling after midnight; two birds clustered after days of distance.

**Forbidden triggers:** visit counts, presence minutes, streak-like "another morning here", numeric trait deltas, "session started".

Prose: lowercase, present tense, bird names, no `you`, no exclamation, no achievement verbs. Assemble from `packages/prose` templates with slots `{name}`, `{other}`, `{place}`, `{wx}`, `{tod}`. Store the final string only.

The notebook is read-only forever. No archive. Virtualized fetch.

---

## 7. Sync model

### 7.1 Single canonical record

Laptop 07:40 and phone 22:10 are two cameras on one aviary. Both `GET /snapshot`. Both `POST /events`. Neither patches birds.

There is no client-to-client channel and no CRDT.

### 7.2 Why last-write-wins is forbidden

If clients posted `{ boldness: 0.62 }`, a stale phone session would erase a morning of laptop presence. The user would not get an error. Drift would simply be wrong. Therefore:

- Clients post *what happened*.
- Tick writes *what that means*.
- Personality updates are `trait = trait + delta` with `delta ≥ 0`.

`personality_rev` increments for ops/debug. It is not shown and not used as LWW.

### 7.3 Event ordering

Consume by `seq` (server receipt), not client_time. Client_time is a hint for catch-up piecewise replay within a small skew window. This makes two devices safe without clock sync.

Idempotency: unique `interaction_events.id`. Retries are free.

### 7.4 Presence integrity

Server-side guards (cannot fully re-check focus, so defend the log):

- Require `payload.visible === true && payload.focused === true && payload.activity_s_ago ≤ 240`.
- Rate-limit per session.
- Cap aviary presence integral at wall-clock elapsed since last tick.
- Ignore presence from visit heartbeats.
- Ignore presence if `marked_for_deletion_at` is set.

Client-side implementation must use all three signals. Unit-test the sensor with fake document state. A "tab open" shortcut is a ship-blocker.

Settle and tab close both emit `session_end` and stop pings. No scolding surface.

### 7.5 Conflict / error surfaces

True state conflicts on personality cannot occur. Remaining user-visible failures:

| Case | Surface |
|---|---|
| Magic-link replay or expiry | Matter-of-fact sign-in error |
| Session revoked / idle timeout | Same |
| Snapshot 5xx | Quiet field stays; after ~3s show matter-of-fact reload copy. No spinner. |
| Offer 409 cooldown | Client should have hidden it; if not, ignore locally |
| Visit 410 | "This visit is no longer available." |

No "sync conflict picker." No "choose a device."

### 7.6 Local cache for first bird

IndexedDB holds the last successful host snapshot, keyed by `account_id`, marked `cached_at`.

- If cache age < 5 minutes and session valid: paint birds immediately from cache (mid-action), then revalidate.
- If cache stale or missing: paint quiet field (inline CSS sky from local clock — no JS bundle required for the field), then snapshot.
- Never paint a stale *personality-derived* number; we do not cache vectors on the client anyway.

This is how <500ms and "already in motion" coexist on repeat visits. Cold 4G still needs the snapshot in the first payload — see §11.2.

### 7.7 Offline

v1 is online-only. If the network dies mid-session, keep interpolating last snapshot, queue events in memory (max 50), flush on return. If unload happens first, at most a few presence pings are lost — acceptable; do not put events in `localStorage` as a second source of truth that could replay days later.

---

## 8. Frontend rendering pipeline

### 8.1 Scene graph

One screen, no pan, no zoom, no scroll of the scene (notebook/settings scroll separately).

Layers back to front:

1. Sky wash (time-of-day gradient).
2. Far foliage (very slow parallax, amplitude ≤ 2px).
3. Back perch zone.
4. Mid foliage / perches.
5. Birds (sorted by zone, then y).
6. Front perch / occasional foreground branch.
7. Weather overlay (rain streaks, wind-bent leaves) — subtle.
8. Offer props (seed, pool reflection, no UI chrome).
9. DOM overlay: invisible bird buttons, captions, focus ring, top bar.

Color: calm naturalist tokens from the design system. No saturated UI accents in the scene. Chrome contrast AA.

### 8.2 Mid-action first frame

The first *bird* frame must not be a T-pose or a fade-in from empty (except true first adoption).

Procedure:

1. HTML shell paints quiet field using inline CSS. Sky hue from a tiny inline script reading local hours — allowed, because it is the place, not a loader.
2. JS boots, reads cache or network snapshot.
3. For each bird, set `clip` and `phase` from snapshot so a preen is already 40% through.
4. Start rAF. Do not wait for webfonts, notebook, settings chunks, or audio resume.
5. Forbidden: logo splash, spinner, skeleton bird pulse, "wake" stretch animation, fade-from-black.

If snapshot > 500ms: the user has been looking at a quiet field. That is the loading state.

### 8.3 Idle micro-motion

Never still in a way that reads paused. Clips: `preen`, `scan`, `tilt`, `shuffle`, `fluff`, `sleep`, `listen`. Mood maps:

| Mood | Idle bias |
|---|---|
| wary | `scan` heavy, back zone, tighter body |
| content | `preen`, `shuffle` |
| curious | `tilt` toward calls and falling leaves |
| drowsy | `fluff`, low sit, slow blink |
| alert | `scan` + occasional `tilt`, more front |

Motion is personality-keyed in *timing* (bold birds shift weight more often) but the client only sees hints + mood, not the vector.

Hidden tab: cancel rAF, suspend canvas. Simulation continues on the server. On return, new snapshot, new mid-action phases — do not resume the old clip as if no time passed.

### 8.4 Transitions

- Perch change: bezier hop / short hop-glide, 8–20s. No teleport on live sessions.
- Lighting: continuous, not stepped. Dawn 05–07, warm evening 17–20, night after ~21 local, species-dependent.
- Settle: 3–4s wash to evening, calls gain-ramp down. Undo <5s reverses the same curve. After 5s, any click/focus is *re-engage*: 4–6s restore to clock-correct lighting (not a snapped undo).
- Weather: fade in over ~20s, never a thunderclap.

### 8.5 Reduced-motion mode

Trigger: `prefers-reduced-motion: reduce` OR account opt-in. Opt-out in accessibility settings can override OS (user agency).

This is a designed renderer, not `animation: none`.

- Replace clip playback with 2–4 still poses per behavior, cross-faded over 1.5–3s.
- Flights become perch-to-perch crossfades (opacity + small dissolve), no arcs.
- Remove leaf/feather drift and parallax.
- Keep day/night and settle color shifts, slowed ~1.5×.
- Audio and captions unchanged.
- Notebook and drift unchanged.

Ship on day one. Test as a first-class screenshot set, not a checkbox.

### 8.6 Top bar

Icons: account, accessibility, notebook, offer, settle. Nothing in-scene.

After **4s** without pointer/key, fade to ~12% opacity. Restore on any pointer/key. Keyboard focus on a top-bar control forces full opacity.

No badges. No unread-notebook dot. No visit count.

### 8.7 Responsive fit

Compute a virtual stage (e.g. 1600×900 design space) and uniform-scale to the viewport while keeping all perch slots inside the safe rect. Narrow phones compress inter-perch gaps; never crop a bird. Minimum tap target on a bird: 44×44 CSS px via hit overlay.

### 8.8 Empty and quiet-field states

Same visual language: soft sky, maybe one extremely slow gradient breathe (disabled in reduced motion). No silhouettes of missing birds. First-ever fly-in is the only entrance animation in the product.

### 8.9 Offer visuals

Triggered from top bar, not from clicking a bird.

- Seed: small object near front-middle; bird motion per `OfferResult`.
- Song: no object; WebAudio fragment in space; bird join/quiet/countercall.
- Pool: low reflective ellipse in front; drink / bathe / watch.

Props despawn after the reaction or 45s. They are not food meters.

### 8.10 Frame loop

```
onFrame(t):
  if hidden: return
  dt = min(t - last, 50ms)          // avoid sleep spiral
  if t - last > 2000: requestSnapshot(); restart clips from new phase
  updateInterpolation(dt)
  updateIdle(dt)
  updateOrnaments(dt)               // skip if reduced-motion
  updateWeather(dt)
  updateLighting(clock, settleLocal)
  drawCanvas()
  updateCaptionPositions()
  last = t
```

Budget: 16.6ms. Canvas is 2D, not WebGL, to keep the bundle and GPU story simple. If a future art pass exceeds budget, switch the *fill* of the canvas module without changing the snapshot protocol.

### 8.11 Code splitting

Initial chunk: shell, scene, snapshot client, presence sensor, greeting director, audio graph, hit overlay.

Deferred: account settings, session list, export/delete, visit admin, notebook virtual list (prefetch on first top-bar activity), accessibility panel.

Target: initial JS **< 2MB gzipped**, aim **< 400KB gzipped** for the aviary chunk. The 2MB number is a ceiling, not a goal.

---

## 9. Audio pipeline

### 9.1 Why procedural

Looped files break on the second exact repeat and chorus like stacked karaoke tracks. The 2MB budget also cannot hold a real motif library as PCM. v1 ships **no call samples**.

### 9.2 Graph

```
per bird:  scheduler → motif synth (oscillators + noise + simple formant) → birdGain → panner(x by perch)
rain:      filtered noise → wxGain
masterGain → destination
```

- Polyphony cap: 7 birds × 2 overlapping notes + rain. Reuse nodes. **No per-call `new Oscillator` leak** — keep a pool.
- Panning mild (±0.3) so the scene stays one place, not a DAW.

### 9.3 Scheduler

Each bird has a next-time drawn from an exponential distribution with mean:

```
mean_s = lerp(28, 8, vocal_n) * diurnal * (settle ? 2.4 : 1)
night && !nightjar: mean_s *= 4
```

On fire: build motif (`packages/call-grammar`), play, emit caption event, maybe trigger neighbors' answer windows.

Listen-in does not change *who is allowed to call*; it changes mix.

### 9.4 Listen-in mix

Engage / disengage ramps **1.5s** (equal-power).

| Role | Gain |
|---|---|
| Focused bird | 1.0 |
| Others | 0.18–0.28 (distance-tinted), never 0 |
| Rain | unchanged |

Disengage by: second activate on same bird, focus other bird, click empty stage, Escape / focus leaving the scene.

Feel: listening, not soloing tracks. No UI "now playing."

### 9.5 Chorus

When bird A calls, birds with `vocal ≥ moderate` and `cluster/near` or high warmth roll `p = 0.25–0.55` to answer after 200–800ms with a *related but not identical* motif (transpose a second, drop last note). Two answers max per call to avoid pile-on at seven birds.

### 9.6 Settle and night

Settle: master call gain → 0.25 over 3s, mean_s lengthens. Night: most species nearly silent; nightjar grammar remains.

### 9.7 WebAudio fallback

If `AudioContext` missing, `createAudioContext` throws, or context stays `suspended` after first input:

- Stay silent.
- Force captions on for the session (do not persist over user off unless they never set a preference).
- **Do not** fetch mp3s.

Autoplay policy: first frame may be silent. That is acceptable. A toast is not.

### 9.8 Caption generation

At the moment a call is scheduled, the grammar returns a phrase, e.g. `a soft three-note rise`. Render as small AA-contrast text near the bird, fade with the call envelope. Same voice as the notebook. Not "CHIP_03".

---

## 10. Interaction implementation notes

### 10.1 Presence sensor

```
active = visibilityState==="visible"
      && document.hasFocus()
      && (now - lastPointerOrKey) <= 240_000
```

Listen to `pointermove`, `keydown`, `visibilitychange`, `focus`, `blur`. Do not use `mousemove` only (pen/touch). Do not count `scroll` inside the notebook as aviary presence unless the aviary document still has focus and visibility — it does; notebook is an overlay, so presence may continue. That is correct: they are still with the aviary.

Ping every 30s while `active`. On falling edge, send `session_end` reason `hide` once.

Calibration comment in code: *lean long; watching is the product.*

### 10.2 Listen-in input

Pointer on bird, Enter when focused. No highlight except the a11y focus ring (keyboard / SR users need it; pointer listen-in should stay visually quiet — maybe the bird tilts toward the viewer, nothing more).

### 10.3 Offer input

Top bar → three choices, keyboardable. Song picker is a list of 8 named fragments in naturalist labels ("a short low phrase"), not "Track 3".

Cooldown: disable that kind for that bird in the UI from snapshot; the offer is to the aviary, so show a quiet "the aviary is still with the last offering" only if *all* birds are cooling — even then prefer no message and a simply inert control.

### 10.4 Settle

Top-bar control. Local lighting + audio. Event `{ undone: false }`. For 5000ms, any pointer/key in the scene (not just empty space) undoes: reverse lighting, `{ undone: true }`. After 5000ms the aviary is settled until re-engage.

Engine treats settle as presence end, same as close. No extra drift.

### 10.5 No welcome chrome

Implementation review checklist (PR template):

- [ ] No toast library usage on aviary routes
- [ ] No "welcome back"
- [ ] No absence-length copy
- [ ] No visit banner
- [ ] No adoption confetti
- [ ] Notebook does not mention the user's attendance

---

## 11. Accessibility surfaces

Accessibility is a designed product surface. It ships in the same release train as the canvas.

### 11.1 Screen-reader narration

A single `aria-live="polite"` region in the aviary route, updated by replacement (not stacking).

Cadence:

- Idle: one paragraph every **45s** (range 30–60).
- Immediate (still observational): return-greeting, offer reaction, settle, first fly-in, weather start.

Example register (do, lowercase):

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Not: `Pip, perch 2, mood content.`

Composer uses the same package as the notebook, different template set, shorter. Include names once they are known ("pip is on the front rail") rather than only species, so relationships form.

Priority: if a live update is due and a new greeting arrives, replace with the greeting observation; do not queue a speech pileup.

Bird hit targets: `role="button"`, accessible name = bird name, `aria-describedby` pointing at a tiny species + perch phrase. Do not put trait numbers in ARIA.

### 11.2 Keyboard

- `Tab`: top bar icons in DOM order, then first bird.
- `ArrowLeft` / `ArrowRight`: previous/next bird by `sort_index`.
- `Enter` / `Space`: listen-in toggle.
- `Escape`: exit listen-in; if offer sheet open, close it; if settled <5s, undo.
- Offer sheet and settle are in the tab order via the top bar.
- Focus ring: 2px soft outline token that passes contrast on both noon and night skies (design system). Draw the ring in DOM, not canvas, so it is not lost to compositor tricks.

### 11.3 Captions

Opt-in in accessibility settings (`captions_opt_in`), plus auto-on when audio is impossible. Positioned near the calling bird; do not cover the top bar. AA contrast. Runtime from grammar.

### 11.4 Reduced motion

See §8.5. Also honor the setting for top-bar fade (still fade, but slower) and settle wash.

### 11.5 Contrast and copy

All user-copy (chrome, settings, errors, captions, visual narration if ever shown) ≥ WCAG AA. Scene itself has no labels. System surfaces use matter-of-fact English casing. Product surfaces use naturalist lowercase.

### 11.6 Visit + a11y

Visitors get the same narration and reduced-motion path. They cannot listen-in; narration must still mention calls so a deafblind-adjacent SR user is not locked out of the only social view.

---

## 12. Performance budgets and observability

### 12.1 Budgets

| Budget | Gate |
|---|---|
| Initial JS gzipped | < 2MB; fail CI if aviary entry + vendors exceed this. Track uncompressed too. |
| Time to first bird | < 500ms p75 on the synthetic mid-tier 4G profile. |
| Idle fps | 60 on the reference 5-year laptop for a 30-minute session with 7 birds. |
| Memory | Heap after 30 minutes ≈ heap at 3 minutes ± slop; CI soak fails on monotonic growth. |
| Snapshot size | < 16KB gzipped typical. |
| Tick compute | p50 < 50ms / aviary; p99 < 5s alarm. |
| Event POST | p95 < 200ms excluding network. |

### 12.2 Hitting 500ms

1. Inline quiet-field CSS in HTML.
2. Aviary JS is the only blocking script; everything else `import()`.
3. After auth, HTML can include a **private, no-store** bootstrap: `window.__SNAP__ = ...` from the origin (not the CDN cache). This is the primary cold-load strategy.
4. Do not wait on audio, fonts, notebook.
5. SVG species parts are tiny and in the aviary chunk; no sprite CDN round-trip before first bird.

### 12.3 Hitting memory

- Oscillator pool, fixed size.
- Caption DOM nodes recycled (max ~7).
- Notebook: windowed list, drop detached entry nodes.
- No retaining snapshot history beyond last + in-flight.
- rAF stopped when hidden.
- No `setInterval` leaks on route leave.

CI: playwright 30-minute soak with 7 birds, `performance.memory` or allocation timeline, fail on >10% growth after GC.

### 12.4 What we measure

Aggregate only:

- Request counts and latencies by route class.
- Tick duration and lag (`now - last_tick_at` distribution) without account ids in the metric backend.
- RUM: TTFB, time-to-first-bird, long-task counts, rAF dt histogram, AudioContext error counts.
- Auth: magic-link issue/consume/expire rates.
- Visit: invite issued / revoked / 410 counts (no emails).

Synthetic fleet: scheduled browsers in a few geos running a signed-out quiet-field and a dogfood host fixture.

### 12.5 What we deliberately do not measure

- Per-bird moods, traits, perch histograms.
- Offer-type popularity, listen-in duration distributions tied to accounts.
- "Average drift."
- Visit graphs between people.
- Any feature that would later "just expose" a leaderboard.

Pipeline rule: simulation DB credentials are not in the analytics workspace. Warehouse ETL jobs listing `birds` or `interaction_events` are a severity-1 incident.

### 12.6 Browser support

Last two Chrome / Safari / Firefox / Edge. Others: matter-of-fact unsupported page. No polyfill tax in the aviary chunk.

---

## 13. Privacy, security, and voice

### 13.1 PII

Email lives in ciphertext + lookup hash on `accounts` and encrypted visitor email on `visit_invites`. Nowhere else. Object storage keys: `exports/{account_id}/{job_id}.json`.

### 13.2 Interaction data purpose limitation

`interaction_events` exist to feed **that aviary's** tick. They are not a product dataset. No export of events to partners. No training. No cross-account similarity.

On hard delete: wipe events, birds, vectors, notebook, invites, visit sessions, sessions, jobs, export objects, then the account row.

Soft delete (30 days): session can undelete; tick skipped; magic-link still works to sign in and recover.

### 13.3 Privacy policy

Plain-text link in account settings. Names operational aggregates. Explicitly excludes per-bird interaction state.

### 13.4 Voice split (implementation)

`packages/prose` exports two namespaces: `naturalist` and `system`. A lint in `apps/web` forbids `naturalist` strings in `routes/account/**` and `routes/auth/**`, and forbids title-case marketing in `routes/aviary/**` live regions.

---

## 14. Social (visits) implementation

Small on purpose.

1. Host types email in settings (matter-of-fact). Confirm send.
2. Mailer sends one-time URL. 30-day unused expiry.
3. Visitor opens URL → read-only renderer, `can_interact: false`.
4. Heartbeat updates duration only.
5. Host visit log on demand: visitor email, day, approximate duration, outstanding invites. **No badge.**
6. Revoke → next visitor snapshot is 410.
7. Default notify off. If on, email "Someone visited your aviary" matter-of-fact, at most one mail per visitor per 24h, no push.
8. Host sees nothing new in the scene. No cursor, no extra bird, no "friend is here."

Abuse: 10 outstanding invites, 20 / day created, token entropy, no public index.

Do not build chat, comments, avatars, explore, or rankings — including "just internally."

---

## 15. Testing strategy

### 15.1 Pure simulation (`packages/simulation`)

- Monotonic drift; neglect fixture bit-identical traits.
- Week / session / three-week numeric targets (§6.3).
- Presence cap: two sessions' pings cannot exceed wall time.
- Mood hold time and diurnal jump after 2-week catch-up.
- Wary contagion stronger for low boldness.
- Age gates ignore event volume.
- Catch-up does not invent presence.

### 15.2 Grammar

- Golden captions for seeds.
- Consecutive emits unequal (hash of note list).
- Pip-vs-Wren recognizability proxy: pitch offset + motif family distance above threshold.

### 15.3 API / sync

- Dual-session event interleave: both listen-ins applied, no lost delta.
- Idempotent event POST.
- Visitor POST events 403 and no presence increment.
- Magic-link single consume.
- Soft delete hides snapshot; undelete restores same bird ids and vectors.

### 15.4 Client

- Presence sensor truth table (8 combinations of the three signals).
- Greeting never unison.
- Listen-in ramp never mutes others completely (gain > 0).
- Reduced-motion code path has no leaf spawners.
- Memory soak and bundle budget in CI.
- Keyboard path e2e.
- No toast component imported from aviary routes (static check).

### 15.5 Accessibility CI

axe on settings/auth (system surfaces). Custom asserts on aviary: live region exists, birds are buttons, captions AA.

### 15.6 What tests cannot catch (so review must)

- "Feels canned" greetings — keep a human listen pass on the dogfood build every week.
- Telemetry leakage — credential split + schema denylist.
- Voice slippage — prose lint + editorial review of new templates.

---

## 16. Delivery plan and rollout

### 16.1 Workstreams (parallel after contracts freeze)

1. **Contracts** — `packages/schema`, snapshot/event types, this plan's constants as named config.
2. **Simulation** — drift, mood, perch, weather, catch-up, adopt gates. CI fixtures first.
3. **API + auth + data** — tables, magic-link, sessions, export/delete, snapshot, events.
4. **Tick + jobs** — worker, mail, invite expiry, hard delete.
5. **Scene renderer** — canvas, perches, lighting, weather, mid-action, quiet field, reduced-motion.
6. **Audio + captions** — grammar, mix, fallback silence.
7. **Interactions** — presence, listen-in, offer, settle, greeting director.
8. **Notebook + narration prose**.
9. **Settings / visits**.
10. **Perf, a11y, observability, privacy lint**.

Freeze snapshot and event schemas before renderer and tick diverge. A weekly compatibility test loads fixture snapshots from week 1.

### 16.2 Milestones

**M0 — Skeleton (week 1–2).** Auth + empty quiet field + snapshot of two static birds. No spinner. Matter-of-fact errors.

**M1 — Alive (week 3–5).** Tick + mood + idle motion + procedural calls + presence pings that *do not yet* drift (log only). Greeting director. Hidden-tab pause.

**M2 — Relationship (week 6–8).** Drift enabled behind a staging flag. Offers, settle, notebook sparsity, listen-in mix. Export/delete.

**M3 — Access (week 6–8, parallel).** Narration, captions, reduced-motion, keyboard, AA audit. These are not a polish milestone after M2; they merge into M2.

**M4 — Multi-device & visits (week 9–10).** Dual-device QA, visitor read-only, revoke 410, notify opt-in.

**M5 — Hardening (week 11–12).** Budgets, soak, tick alarms, privacy ACL review, dogfood calibration of `α_*` and 240s window. Then GA.

Do not launch M1 to the public with drift off "to iterate" — a week of fake stasis teaches the wrong relationship. Public launch is M5.

### 16.3 Ramp

- All new accounts start at **two** birds. Age gates are the only ramp of bird count. No feature-flag "max 3 for now" unless tick load is actually on fire; if so, raise gates, do not sell it as a reward.
- Visits can be globally disabled via config if invite mail is abused. Aviary host path stays up.
- Tick cadence stays 60s. If load requires, drop to 90s *globally* rather than per-user "premium ticks."

### 16.4 Instrumentation from day one

Ship M0 with route latency, auth rates, and client first-bird timing. Ship M1 with tick duration and audio errors. Never ship a "just for launch" dashboard of average boldness.

Dogfood accounts (employees) live in the same schema. Do not special-case their events into a warehouse. Calibration uses **staging fixtures and local simulation tests**, plus qualitative watching of dogfood aviaries by the people who own those accounts — not an aggregate query.

### 16.5 Rollout communications

There is no in-product "what's new." If a blog exists later, it is not this app. In-app, birds notice; the system does not announce.

---

## 17. Risks

### 17.1 Drift calibration (high)

Too fast → Tamagotchi numbers. Too slow → screensaver. The band is narrow.

**Mitigations:** constants in one config module; CI fixtures for week/session/three-week; only shrink rates if users notice session deltas; never add negative drift to "fix" expressiveness — use `recency_factor`.

**Failure mode to watch in dogfood:** "Pip felt different yesterday." That is a rate bug, not a content bug.

### 17.2 Silent presence inflation (high)

A single `visibilityState` shortcut will over-drift the whole population and no unit test on the tick will see it.

**Mitigations:** sensor truth-table tests; server conjunction checks; 1× wall-clock cap; code review label `presence-critical`.

### 17.3 Sync correctness (high)

Any client write to `birds.boldness` reintroduces LWW. ORM conveniences are the threat.

**Mitigations:** DB role `tick_writer` is the only role with `UPDATE` on personality columns; api role cannot update those five columns (Postgres column grants). This is stronger than a comment.

### 17.4 Audio uncanniness (high)

Identical motifs, phase-cancelled doubles, harsh oscillators, or a "ringtone" timbre will make the product feel like a toy.

**Mitigations:** variation rules, answer transposition, golden listen checklist, nightjar distinct but not comic, no sampled fallback that sounds better-canned than the synth. Budget time for a sound pass in M2, not "placeholder sine waves to GA."

**Autoplay:** first-session silence may be misread as broken audio. Resist the banner. Captions-on-if-suspended is the pressure valve.

### 17.5 Accessibility regressions (high)

A canvas-only scene will silently fail keyboard and SR users. Reduced-motion "off" is a v1 fail.

**Mitigations:** DOM hit overlay is required, not optional; a11y CI; narration cadence tests; reduced-motion screenshots in review; ship with M2, not v1.1.

### 17.6 First-frame aliveness (medium-high)

A safe spinner is the most likely aesthetic regression.

**Mitigations:** no spinner component in `apps/web`; quiet field only; bootstrap snapshot in HTML; PR checklist.

### 17.7 Notebook / narration genericizing (medium)

Engineers will write `session started` because it is easy.

**Mitigations:** trigger allowlist in code; forbidden-substring tests (`you visited`, `achievement`, `streak`, `+0.03`); editorial ownership of `packages/prose`.

### 17.8 Tick cost at scale (medium)

Naive per-minute full sim on abandoned accounts wastes money; naive skip makes return stale.

**Mitigations:** catch-up jump (§6.8); do not tick aviaries marked deleted; optionally defer tick of accounts with `last_seen_at` > 14 days until next snapshot request, then catch-up on demand. **If using deferral:** snapshot path must run catch-up before read so the returning user never sees a frozen yesterday. Personality still only changes from events (there are none).

### 17.9 Privacy leakage (medium)

Email in traces, events in "debug export," visit log in a shared dashboard.

**Mitigations:** synthetic ids; log attribute allowlist; warehouse ACL; export omits third-party emails.

### 17.10 Scope creep via social and "engagement" (process)

The first streak, the first "your friend visited" toast, the first public aviary, the first hunger bar.

**Mitigations:** this plan's ship-blocker list; review against `non_goals.md`; product voice lint. There is no version of those features that is compatible with this product.

### 17.11 Seven-bird chorus blur (medium)

If recognizability fails at 5, do not raise the cap; consider lowering age gates' late slots, not shipping 8.

### 17.12 Offer saturation (low-medium)

Without cooldown, curiosity spikes in one sitting. 4-minute per-kind cooldown is mandatory server-side.

---

## 18. Launch checklist (GA)

- [ ] Two system-chosen starters; no catalog; stable ids
- [ ] Personality column grants: only tick role writes
- [ ] Presence = three signals + wall-clock cap
- [ ] Drift fixtures green; no negative deltas
- [ ] Mid-action first frame; quiet field fallback; zero spinners
- [ ] Procedural calls; silence+captions fallback; no mp3 path
- [ ] Listen-in ramps; others never muted
- [ ] Settle optional; 5s undo; no guilt on tab close
- [ ] Notebook sparse, naturalist, no user-behavior entries
- [ ] Narration, captions, reduced-motion, keyboard, AA
- [ ] Magic-link 15m / one-time; sessions revocable
- [ ] Export email; 30-day soft delete; hard wipe job
- [ ] Visits off by default; read-only; revoke 410; no notify by default
- [ ] Bundle / TTFB / soak / tick p99 gates green
- [ ] Telemetry denylist + DB credential split reviewed
- [ ] Voice split reviewed on every surface
- [ ] Unsupported-browser matter-of-fact page
- [ ] No toast on return

---

## 19. Suggested implementation order for the first engineering week

1. Freeze `packages/schema` and Postgres migrations (including column grants).
2. Implement `step()` with fixtures only — no HTTP.
3. Auth + snapshot of fixture aviary.
4. Quiet field + canvas birds mid-preen from snapshot.
5. Presence sensor + event POST (store only).
6. Wire tick to apply presence (still staging).
7. Audio grammar on one species.
8. Greeting director.
9. A11y overlay + live region, even if templates are few.

That sequence produces something that can be *watched* by day five without building a second product around it.

---

## 20. What this plan refuses to leave implicit

- Aliveness is owned by server time + mid-action render + procedural audio together. Any one of those faked (client tick, spinner, looped chirp) collapses the rest.
- Presence is attention, not engagement. The code must not optimize for click count.
- Charm is specificity of prose and motion, not more birds or more chrome.
- The user never manages a number. The export file is a possession, not a dashboard.
- Absence is fine. The engine does not punish it. The UI does not mention it.

This is sufficient to build v1 without waiting for another product pass. Numeric `α_*` and the 240s window are the only constants expected to move, and they move only under the calibration rules above.
