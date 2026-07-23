# Pocket Aviary — V1 Implementation Plan

## 0. Purpose and posture

This plan turns the Pocket Aviary PRD into an executable engineering blueprint. A separate team should be able to build v1 from this document without further product clarification. Where the PRD leaves range open (exact tick cadence, activity window, default trait seeds), this plan makes a defensible call and marks it as a calibration point.

**Product in one sentence:** a browser-only, single-aviary relationship surface where 2–7 procedural birds notice presence, drift slowly toward expressive over weeks, and never punish absence.

**Non-negotiable engineering consequences of the philosophy:**

1. Server owns all canonical simulation state; clients never write personality or mood.
2. Presence is the conjunction of visibility + focus + recent input activity — not “tab open.”
3. Drift is monotonic toward expressive; neglect never moves traits down.
4. Procedural calls only; no recorded call loops, even as fallback.
5. No announcement UI, streaks, scores, badges, hunger, death, or public social surfaces.
6. Accessibility is a designed surface shipping with v1, not a retrofit.

---

## 1. Scope

### 1.1 In scope (v1)

| Area | Deliverable |
|------|-------------|
| Auth | Email magic-link sign-in, per-device sessions, revoke sessions, email change with verify |
| Aviary | One canonical aviary per account; 2 starter birds; age-gated adoption up to 7 |
| Simulation | Server tick (~60s), personality vectors, mood machine, bird-to-bird influence, weather |
| Clients | Modern web SPA (last 2 versions Chrome/Safari/Firefox/Edge); responsive single scene |
| Interactions | Return-greeting, listen-in, offer (seed / song fragment / still pool), settle, presence |
| Content surfaces | Field notebook (auto, sparse, read-only), screen-reader narration, call captions |
| Sync | Multi-device via shared canonical snapshot + event log; additive personality deltas only |
| Social | Opt-in email visit invites; read-only ambient visitor; visit log; revocable; default OFF |
| Account lifecycle | Export JSON snapshot; soft delete 30d then hard delete |
| A11y | Narration, reduced-motion cross-fade mode, captions, keyboard nav, WCAG AA chrome |
| Perf | Bundle ≤2MB gzipped initial JS; TTFB bird <500ms; 60fps idle; no 30m memory growth |
| Ops | Aggregate RUM + synthetics; privacy-partitioned telemetry; tick p99 alarm at 5s |

### 1.2 Explicitly out of scope (v1 and product identity)

- Native iOS/Android apps
- Gamification (streaks, XP, badges, levels, visit calendars, counters)
- Tamagotchi mechanics (death, hunger, distress, decaying meters)
- Social network surfaces (profiles, follows, discovery, comments, chat, avatars, leaderboards)
- Multi-aviary accounts, shared/household aviaries, co-presence visits
- Payments / tiers / “rarity” species economy
- Push/email notifications about aviary state (visit notify is opt-in per-account only)
- Customizable scenes / drag-to-perch / user-authored species
- Personality numbers exposed in any UI or tier
- Recorded-audio fallback path
- Password/SSO auth (deferred)

### 1.3 Ambiguity calls (fixed for v1 unless calibration overturns)

| Open item | Plan decision | Rationale |
|-----------|---------------|-----------|
| Tick cadence | 60s nominal | Matches “~once per minute”; simple scheduler math |
| Presence activity window | 180s without pointer/key | Long enough for still watching; short enough to stall overnight laptop |
| Presence ping interval | 30s while triad true | Dense enough for tick consumption; light enough for mobile |
| Offer cooldown | 4 minutes per bird per offer type | Prevents single-session curiosity saturation |
| Starter pair selection | Fixed curated pairs from 6 species with complementary traits | Avoid catalog; ensure audible contrast |
| Third bird age gate | Day 45 of aviary age | Deepens without attention-farming |
| Further birds | Days 90, 150, 240, 365 (caps at 7) | Spreads over ~1 year |
| Mood enum | `wary`, `content`, `curious`, `drowsy`, `alert`, `settled` | Settled supports evening/settle/night |
| Trait range | Continuous `[0.0, 1.0]` | Simple; seeds centered ~0.35–0.55 |
| Notebook cadence target | ~1 entry / 3–5 active days; burst on notables | Sparsity over feed |
| Magic-link TTL | 15 minutes (spec) | As PRD |
| Invite TTL | 30 days unused | As PRD |
| Settle undo | 5s click-anywhere | As PRD |
| Soft delete | 30 days | As PRD |

---

## 2. Architecture

### 2.1 Service shape

Monorepo with clear deployable boundaries:

```
apps/
  web/                 # SPA (TypeScript, Vite)
services/
  api/                 # HTTPS API + auth + account + visits
  simulation/          # Tick workers that advance aviaries
  mailer/              # Magic links, invites, export links (queue consumer)
  notebook/            # Optional: notebook entry generation job (can live in simulation)
  narrator/            # Optional: server-side narration pack generation (or client-local)
infra/
  edge/                # CDN, HTML shell, small public config
  observability/       # Metrics, synthetics, privacy-safe RUM
packages/
  shared/              # Types, event schemas, trait/mood enums, call grammar params
  sim-core/            # Pure simulation functions (tick step, drift, mood) — unit-tested
  call-grammar/        # Motif libraries + caption generators (shared with client)
  copy/                # Matter-of-fact + naturalist string templates/tests
```

**Recommended runtime:**

- **API:** Node.js (or Go) behind HTTPS; Postgres as system of record; Redis for session/rate-limit/short-lived tokens.
- **Simulation:** Worker pool pulling due aviaries from a time-bucketed queue (Postgres `FOR UPDATE SKIP LOCKED` or Redis sorted set). Pure logic in `sim-core` so tick is deterministic given seed + inputs.
- **Client:** SPA only. WebGL/Canvas2D or SVG+CSS hybrid; WebAudio AudioContext for calls. No SSR of bird motion required; HTML shell + bootstrap snapshot inline preferred for TTFB.

### 2.2 Client/server split

| Responsibility | Owner |
|----------------|-------|
| Account identity, sessions, invites | Server |
| Personality vectors, mood, perch intent, weather plan, call timing seeds | Server (tick) |
| Interaction events (presence, listen-in, offer, settle) | Client writes events; server appends log |
| Smooth motion interpolation, idle micro-poses, leaf drift ornaments | Client |
| Procedural call audio + captions from grammar params | Client |
| Notebook entry generation | Server job (needs full event history + state) |
| Screen-reader narration pack | Prefer client from snapshot + local voice rules; server can ship phrase packs |
| Visit render | Client with visitor auth token; read-only APIs |

**Hard rule:** no code path, admin tool, migration, or “sync helper” lets a client PATCH personality fields.

### 2.3 Render pipeline boundary

```
Server snapshot (truth)
        │
        ▼
Client state store (immutable snapshot + local clock offset)
        │
        ├── Scene composer (perches, birds, sky, weather, offers-in-flight)
        ├── Motion layer (idle FSM, perch transitions, reduced-motion crossfades)
        ├── Audio graph (per-bird voices, listen-in bus, chorus mix)
        ├── A11y layer (focus rings, caption nodes, live region narration)
        └── Chrome (top bar only; fades on idle)
```

Ornaments (leaves/feathers) are **pure client**, not in the snapshot. Bird identity, mood, perch zone, action intents, and call param seeds come from the snapshot. Client never invents opposite mood/personality for filling time; it only interpolates and ornamentalizes.

### 2.4 Data stores

1. **Postgres (canonical)**
   - accounts, sessions, birds, personality_vectors, mood_state, aviary_state
   - interaction_events (append-only)
   - notebook_entries
   - visit_invites, visit_sessions, visit_log
   - deletion_queue
2. **Redis**
   - magic-link tokens, rate limits, short export URLs
   - optional tick due-set
3. **Object storage**
   - account export artifacts (short-lived signed URLs)
4. **No analytics warehouse join** to simulation DB. Telemetry pipeline is a separate sink with aggregate-only schemas.

### 2.5 Trust and ID model

- Account primary key: UUIDv7 (or UUIDv4) generated at creation — **never email**.
- Bird primary key: UUID stable for account lifetime; rename does not change ID.
- Email stored once on account, encrypted at rest; used only for auth mail / visit invite delivery / export delivery.
- External logs/metrics dimensions: `account_id` UUID only; never email, never bird name if name could be identifying in combination (prefer bird_id if any bird dimension is ever needed — but v1 telemetry should not include bird dimensions at all).

---

## 3. Data model

### 3.1 Core entities (Postgres)

#### `accounts`
```
id                  uuid PK
email_ciphertext    bytea NOT NULL
email_hash          bytea UNIQUE NOT NULL   -- keyed hmac for lookup, not reverseable casually
email_verified_at   timestamptz
created_at          timestamptz
aviary_created_at   timestamptz            -- age gate anchor
locale_tz           text                   -- IANA tz from client preference/bootstrap
settings_json       jsonb                  -- a11y, visit_notify_opt_in, captions_default, etc.
deletion_marked_at  timestamptz NULL
status              enum('active','pending_deletion','deleted')
```

#### `sessions`
```
id                  uuid PK
account_id          uuid FK
token_hash          bytea UNIQUE
device_label        text
created_at          timestamptz
last_seen_at        timestamptz
revoked_at          timestamptz NULL
```

#### `aviaries` (1:1 with account in v1)
```
id                  uuid PK
account_id          uuid UNIQUE FK
bird_count          int
settled_until       timestamptz NULL
weather_state       jsonb                   -- current/next event
day_phase_cache     text                    -- optional denorm
schema_version      int
updated_at          timestamptz
last_ticked_at      timestamptz
tick_version        bigint                  -- monotonic for client freshness
```

#### `birds`
```
id                  uuid PK
aviary_id           uuid FK
species_id          text                    -- from species pool
display_name        text
adopted_at          timestamptz
sort_order          int
call_grammar_seed   bigint                  -- fixed at adoption; continuity
visual_seed         bigint
is_active           bool
```

#### `personality_vectors`
```
bird_id             uuid PK FK
boldness            real CHECK (0..1)
social_warmth       real
vocal_frequency     real
plumage_saturation  real
curiosity           real
updated_at          timestamptz
updated_by_tick     bigint
```

**Rule:** only simulation tick UPSERTs this table.

#### `mood_state`
```
bird_id             uuid PK FK
mood                enum(...)
mood_entered_at     timestamptz
confidence          real                    -- optional internal
perch_zone          enum('front','middle','back')
action_intent       text                    -- preen|scan|tilt|shuffle|sleep|approach_offer|...
pose_phase          real                    -- 0..1 seed for client interpolation
updated_at          timestamptz
```

#### `interaction_events` (append-only)
```
id                  bigserial PK
aviary_id           uuid
account_id          uuid
bird_id             uuid NULL               -- null for aviary-level
type                text                    -- presence_ping|listen_in_start|listen_in_end|
                                            -- offer_seed|offer_song|offer_pool|
                                            -- settle|unsettle|session_open|session_blur
payload             jsonb                   -- durations, device_session_id, client_ts
client_event_id     uuid                    -- idempotency
server_received_at  timestamptz
processed_by_tick   bigint NULL
```

Indexes: `(aviary_id, id)`, `(aviary_id, processed_by_tick) WHERE processed_by_tick IS NULL`.

#### `notebook_entries`
```
id                  uuid PK
aviary_id           uuid
authored_at         timestamptz             -- narrative “when”
body                text                    -- naturalist prose lowercase
salience            real
source_event_ids    bigint[]                -- provenance internal
```

#### Visit tables
```
visit_invites(id, host_account_id, visitor_email_hash, token_hash,
              created_at, expires_at, revoked_at, consumed_at)
visit_sessions(id, invite_id, started_at, ended_at, approx_duration_s)
visit_log_entries(id, host_account_id, visitor_email_redacted, visited_at, duration_s)
```

Visitor email for host log stored redacted (full email only if host originally typed it; keep host-supplied (email as entered) for host transparency; hashed for identity).

### 3.2 Species pool (v1 content pack)

Ship ~6 species as data modules, not code forks:

| species_id | Silhouette notes | Motif family | Night active |
|------------|------------------|--------------|--------------|
| warbler_soft | Small, rounded | Soft rising 2–3 note | no |
| thrush_mid | Medium | Liquid phraselets | no |
| finch_bright | Compact | Staccato twitters | no |
| dove_low | Fuller body | Soft coos / low rolls | no |
| wren_quick | Upright short tail | Fast scratches | no |
| nightjar_dusk | Longer wing | Low churr / single notes | **yes** |

Starter pairing algorithm: pick two species with distinct motif families and seed personality contrast (one higher boldness/social_warmth; one calmer/rear-preferring). User does **not** choose.

### 3.3 Client-local types (not durable)

- Listen-in focus bird id
- Top-bar fade state
- Audio context unlock status
- Local presence triad sensors
- Quiet-field loading vs ready
- Caption overlay instances
- Reduced-motion preference effective flag

### 3.4 Export schema

On-demand JSON:
```json
{
  "export_version": 1,
  "exported_at": "...",
  "account": { "id": "...", "timezone": "...", "settings": {} },
  "aviary": { "created_at": "...", "bird_count": 2 },
  "birds": [{
    "id": "...", "name": "pip", "species_id": "...",
    "personality": { "boldness": 0.41, "...": "..." },
    "mood": "content", "adopted_at": "..."
  }],
  "notebook_entries": [{ "authored_at": "...", "body": "..." }]
}
```
Export includes personality numbers because it is the user’s data portability copy — still never shown in product UI.

---

## 4. API surface

All authenticated routes require session cookie / bearer from magic link. Matter-of-fact error bodies. No “welcome” payloads.

### 4.1 Auth

| Method | Path | Notes |
|--------|------|-------|
| POST | `/auth/magic-link` | `{email}` → 202 always (anti-enumeration); rate limit per email+IP |
| GET | `/auth/magic-link/consume?token=` | one-time; sets session; invalidates token |
| POST | `/auth/sign-out` | revokes current session |
| GET | `/me` | account summary, settings, sessions list (devices) |
| POST | `/me/sessions/:id/revoke` | |
| POST | `/me/email-change` | start verify new email |
| POST | `/me/email-change/confirm` | |
| POST | `/me/export` | enqueue export; email link when ready |
| POST | `/me/delete` | mark soft deletion |
| POST | `/me/delete/cancel` | within 30d |

### 4.2 Aviary state

| Method | Path | Notes |
|--------|------|-------|
| GET | `/aviary/snapshot` | full canonical snapshot for owner; ETag/`tick_version` |
| GET | `/aviary/snapshot?since_tick=` | optional delta if cheap; else full |
| POST | `/aviary/events` | batch append interaction events; idempotent by `client_event_id` |
| POST | `/aviary/bootstrap` | first-time: create aviary + two starters + names |
| POST | `/aviary/birds/:id/rename` | `{name}` only |
| GET | `/aviary/adoption-offer` | if age gate ready: proposed species + default name suggestions |
| POST | `/aviary/adoption-offer/accept` | `{name}` adds bird if under cap and gate open |

**Snapshot payload sketch:**
```json
{
  "tick_version": 184422,
  "server_time": "2026-07-24T12:00:00Z",
  "local_day_phase": "late_morning",
  "weather": { "kind": "clear|rain|wind", "intensity": 0.2, "ends_at": "..." },
  "settled": false,
  "birds": [
    {
      "id": "...",
      "name": "pip",
      "species_id": "wren_quick",
      "personality_public": {
        "plumage_saturation": 0.44
      },
      "mood": "curious",
      "perch_zone": "front",
      "action": { "intent": "scan", "phase": 0.37, "facing": 0.1 },
      "call": {
        "grammar_seed": 9912,
        "next_window": { "earliest_ms": 1200, "latest_ms": 4800 },
        "vocal_frequency": 0.52,
        "mood_mod": 0.9
      },
      "visual": { "seed": 12, "saturation": 0.44 }
    }
  ],
  "active_offer": null,
  "greeting": {
    "bird_id": "...",
    "absence_class": "short|medium|long",
    "form": "glance|soft_call|approach|call_and_answer",
    "stagger_ms": [0]
  }
}
```

**Personality exposure rule on wire:** clients need some continuous params that affect render/audio (saturation, vocal frequency lightness for timing). Prefer sending **derived render params** already shaped by personality rather than raw vector names in client code comments/UI. Engineers may map boldness→front preference server-side and only send perch + motion bias. Do **not** ship a debug panel. Instrument-only admin tools offline.

Decision: snapshot includes `render_hints` per bird (approach_bias, chorus_join_bias, preen_bias, scan_rate) computed server-side from vector+mood — **not** labeled trait names in any user-visible surface. Raw vectors stay server-only except account export.

### 4.3 Events API

`POST /aviary/events`
```json
{
  "events": [
    {
      "client_event_id": "uuid",
      "type": "presence_ping",
      "client_ts": "...",
      "payload": { "focus": true, "visibility": "visible", "activity_age_ms": 1200 }
    }
  ]
}
```

Server validates: session matches account; bird_id belongs to aviary; rate limits insane spam; stores raw claims but **recomputes trust** (e.g., presence paid only if conditions plausible; reject backdated storms). Duration fields preferred over absolute “set state.”

Allowed client event types:
- `session_open`, `session_close`
- `presence_ping`
- `listen_in_start`, `listen_in_end`
- `offer_seed`, `offer_song`, `offer_pool`
- `settle`, `unsettle`
- `focus_bird` (optional for a11y/analytics-free local only — prefer not if redundant with listen-in)

Invalid: any personality/mood write.

### 4.4 Notebook

| Method | Path | Notes |
|--------|------|-------|
| GET | `/aviary/notebook?cursor=` | reverse chrono; infinite history |
| — | no write APIs | |

### 4.5 Visit flow

| Method | Path | Actor |
|--------|------|-------|
| POST | `/visits/invites` | host: `{email}` creates invite, mails link |
| GET | `/visits/invites` | host: outstanding + recent |
| POST | `/visits/invites/:id/revoke` | host |
| GET | `/visits/view?token=` | visitor: issues short visitor session cookie |
| GET | `/visits/snapshot` | visitor session: same visual state, flags `mode:"visit"` |
| — | visitor cannot POST events | (404/403) |
| GET | `/visits/log` | host only |
| PATCH | `/me/settings` | `{visit_notify_opt_in:bool}` default false |

Visitor snapshot omits: notebook, account chrome, offer/settle controls, adoption, personal settings. Includes birds/weather/day/calls. Visit does **not** emit presence for host drift.

If revoked/expired: matter-of-fact page — “This visit is no longer available.”

### 4.6 Keepalive / freshness

- Poll snapshot every 45–60s while visible (or tick_version long-poll if implemented).
- Immediate snapshot on `visibilitychange` → visible and on `pageshow` after bfcache/suspend.
- Event flush every 10–15s or on threshold count; flush on `pagehide` via `sendBeacon`/`fetch keepalive`.

---

## 5. Simulation engine design

### 5.1 Tick loop

Cadence: claim aviaries where `last_ticked_at <= now() - interval '60 seconds'`.

Per aviary step (transactional):

1. Lock aviary row.
2. Load birds, vectors, moods, unprocessed events since last tick, last presence summary.
3. Compute local day phase from `accounts.locale_tz` + `now()`.
4. Advance weather (rare transitions).
5. Summarize event windows: presence_seconds, per-bird listen_in_seconds, offers accepted/rejected, settle flags.
6. Apply **drift deltas** (small).
7. Apply **mood transitions**.
8. Choose perch zones + action intents from mood×personality + bird-to-bird.
9. Schedule call windows / seeds for next minute of client audio.
10. Maybe enqueue notebook generation candidate.
11. Mark events processed; bump `tick_version`; set `last_ticked_at`.

Workers must be idempotent under retry if crash after commit — use tick_version barriers.

### 5.2 Drift function

Traits \( t \in [0,1]^5 \).

Presence weight dominates. Define for each tick interval \(\Delta\):

```
presence_hours = presence_seconds / 3600
listen_hours_b = listen_in_seconds[b] / 3600
offer_curiosity_b = count(accepted offers near b) * w_offer
offer_boldness_b = count(offers initiated near b) * w_near
```

Proposed rates (start points; calibrate in staging):

```
d_express = k_p * presence_hours
  k_p ≈ 0.004   # ≈ 0.028 / week at 1h/day presence → measurable; not session-obvious

per bird b:
  warmth += clamp(k_l * listen_hours_b + 0.3 * d_express)
  vocal  += clamp(k_l * listen_hours_b + 0.5 * d_express * baseline_vocal)
  bold   += clamp(k_b * offer_boldness_b + 0.4 * d_express)
  curio  += clamp(k_c * offer_curiosity_b + 0.3 * d_express)
  plumage+= clamp(0.6 * d_express)
```

**Monotonic expressive rule:** all applied deltas are `max(0, ·)`. No negative deltas from neglect, long absence, or silence. Caps soft: asymptotic approach to 1.0 via `t' = t + d*(1-t)`.

**Ambient quietness without negative drift:** greet probability and approach rate use “recent presence mass” as a separate **expressivity energy** cache (fast state, not personality):

```
expressivity_energy ∈ [0,1]
energy *= exp(-lambda * away_hours)   # decays with absence
energy += f(presence)                 # rises with watching
greeting_chance = f(social_warmth, boldness, energy, mood)
```

So neglected birds are quieter/less greeting-forward **without** becoming less bold/colorful/warm in stored personality. This implements “ambient, not mistrust.”

Calibration tests (CI simulation of synthetic weeks):

- 7 days × 45 min presence/day → measurable Δ mean traits > instrument threshold (e.g. 0.01–0.03).
- 21 days same → human-visible density in seeded snapshot replay (animation/audio parameter diffs).
- 14 days zero presence → personality Δ ≈ 0; energy low; greet rare.
- Spam offers with cooldown respected → curiosity cannot jump > session cap.

### 5.3 Mood machine

States: `wary | content | curious | alert | drowsy | settled`.

Transition inputs:
- Recent offer outcomes, listen-in
- Day phase (early morning → alert bias; dusk → drowsy; night → settled for non-nightjar)
- Weather (rain → lower vocal bias + slight wary/content dampen; wind → alert/wary split by boldness)
- Neighbor moods (wary spreads with adjacency weight)
- Personality priors (high boldness reduces wary entry probability)

Persist mood across sessions. On tick while user away, advance toward phase-consistent attractors (not snap to content on open).

Settle gesture: push aviary `settled=true`, birds force-bias to drowsy/settled for remaining session until unsettle/tab end; does not rewrite personality.

### 5.4 Perch and idle intent

```
zone_score(front) = boldness + approach_bias(mood) + offer_pull - wary_penalty
zone_score(back)  = (1-boldness) + wary + drowsy
```

Only one bird “front-focal” soft-preferred; avoid all birds front crowded (social spacing using social_warmth).

Idle intents loomood mapped:
- wary: scan high, rear, flinch-ready
- content: preen, slow shuffle
- curious: tilt, track ornaments
- alert: head-up scan
- drowsy/settled: fluff, low posture, eyes half/closed

Client renders continuously; server resamples intent every tick or on events.

### 5.5 Call-grammar runtime

Each species has motif library M = sets of parametric atoms (duration, pitch base, interval pattern, timbre noise seed). At call time client composes:

```
phrase = choose_motifs(seed, mood_mod, vocal_freq)
vary: pitch_jitter, duration_stretch, gap, ornament probability
```

Server sends timing windows and trait-shaped rates; client fires calls when due if audio unlocked and tab audible policy allows. Recognizability: keep species motif family identity fixed by `call_grammar_seed` + species; mood changes ornaments not family.

Chorus: when multiple birds schedule overlapping windows, slight phase avoid; high social_warmth birds answer within 400–1200ms of neighbor call with reduced amplitude.

Bird-to-bird: alarm/wary call from one raises neighbor wary propensity on next tick (and immediate client-side startle bias for snappiness optional, if coherent with snapshot).

### 5.6 Weather generator

Server Poisson-like rare events (target: light rain ~2–3×/week sim-local; wind occasional). Duration 5–20 minutes. Effects mood coeffs short-lived. No storms/snow.

### 5.7 Return-greeting planner

On `session_open` (or first snapshot after absence), server (or deterministic client using snapshot fields) computes:

```
absence_class =
  < 20m → short
  < 36h → medium
  else long

greeting_bird = argmax birds of score(boldness, social_warmth, energy, mood_not_drowsy)
form = map(absence_class, mood, boldness)
stagger secondary greeters with RNG offsets 400–1800ms; never unison fanfare
```

No toast. Greeting is animation+call only + a11y narration priority.

### 5.8 Offers

Top-bar opens offer tray (seed, song fragment, still pool). Selection places transient world object; nearest eligible bird (cooldown clear) evaluates acceptance:

```
P(approach) = f(curiosity, boldness, mood, distance)
```

Outcomes:
- seed: approach/eat/ignore/wait
- song: join/quiet/counter-call based on vocal_freq×mood
- pool: drink/bathe/watch

Outcome events feed drift (curiosity/bold on positive engagement). Cooldown 4m privileged per bird per type.

### 5.9 Notebook generation

Async job examines recent ticks/events for **notables**:
- first-greeter swapped after multi-day pattern
- long quiet morning
- rain passed + fluffed posture
- rare multi-bird chorus
- adoption day

Rate limiter: default max ~1 entry / 72h unless salience high; hard max 1/day even for power users.

Prose: template+slot-fill with naturalist constraints (lowercase, present looking/past-day diary ok “tuesday — …”, species/name, no user streak language, no numbers of traits). Golden tests for banlist: “achievement”, “streak”, “XP”, “you visited”, “level”.

### 5.10 Adoption age gate

`aviary_age_days = floor((now - aviary_created_at)/1d)`. Unlock next slot at thresholds; present soft naturalist offer in chrome (not toasty confetti). System picks species; user names. Cap 7 enforced server-side.

---

## 6. Sync model

### 6.1 Canonical single writer

- **One aviary record** per account.
- Tick is sole writer of vectors, moods, weather, scheduled intents.
- Clients append events; multiple devices may append concurrently — both valid.

### 6.2 Conflict prevention

| Data | Strategy |
|------|----------|
| Personality | Additive server deltas ordered by event log + tick time; never LWW absolute |
| Mood | Server computed; clients display last snapshot; no client mood |
| Names | Last write wins OK (low stakes) with `updated_at` |
| Settings | LWW per-field with version |
| Events | Append-only + idempotent client_event_id |
| Sessions | Independent |

No CRDT personality merge. No “choose this device’s birds.”

### 6.3 Multi-device UX

Laptop morning + phone night both `GET snapshot` → same tick_version wallet of truth. If phone has unsent events offline, queue locally and flush on reconnect; tick applies when received (slight delay OK). Visible edge: bird may not reflect last 30s of phone presence on laptop until next tick — acceptable.

### 6.4 Offline / suspend

On resume: pull snapshot immediately; discard stale local interpolations; rebuild motion mid-action from snapshot poses (preserves “already running”). Local ornament RNG may reshuffle — OK.

### 6.5 Error surfaces (matter-of-fact)

- Expired magic link
- Session timeout
- Snapshot load failure
- Visit unavailable

Never naturalist-snipe these.

---

## 7. Frontend rendering pipeline

### 7.1 Scene graph

Layers bottom→top:
1. Sky / gradient day-night
2. Far foliage (soft)
3. Back perch zone + birds in back
4. Mid zone
5. Front zone + offer props
6. Foreground leaf occasional
7. Captions near birds
8. Focus ring (a11y)
9. Top bar chrome (outside scene feel; overlay)

No panning/zoom/scroll of scene. Resize dystorts spacing but keeps all birds framed (`object-fit` style layout math).

### 7.2 Motion systems

- **Idle FSM** per bird driven by `action.intent` + mood; looping micro clips with personality rate multipliers.
- **Perch travel:** bezier or hop arcs between zones; duration 0.8–2.5s mood-dependent.
- **Always in motion:** even “still” includes 0.5–1.5px breath/weight shift at low frequency.
- **First paint:** bootstrap snapshot embedded or parallel fetch; place birds at phase offsets so nothing locks in T-pose. Quiet field only if snapshot not ready; **no spinner**.

### 7.3 Day/night

Client maps phase to palette curves; server authoritative phase based on user TZ (client may smooth sunrise). Night: dim; nightjar species remains active.

### 7.4 Reduced motion

If `prefers-reduced-motion: reduce` or settings toggle:
- Replace loops with still pose crossfades (2–4s)
- Perch moves = opacity/position ease without flap arc spam
- Remove leaf/feather drift
- Keep color phase shifts slowed
- Audio unchanged

Treat as designed aesthetic, coded as `MotionBackend = Full | Crossfade`.

### 7.5 Top bar

Icons: account/settings, a11y, notebook, offer, settle. After ~3s cursor/keyboard idle → opacity ~0.15; restore on activity. No badges/notification dots on icons for visits.

### 7.6 Empty / first adoption

Quiet field → soft fly-in once. Never empty again.

### 7.7 Tech choice recommendation

**Primary:** Canvas2D or WebGL light (Pixi/regl) for birds + CSS for chrome. SVG-only may struggle with 7 birds + filters at 60fps; evaluate prototype week 1. Assets: procedural feathers + small SVG silhouettes ones per species.

---

## 8. Audio pipeline

### 8.1 Graph

```
Per-bird VoiceNode (procedural buffer source / oscillator+noise chain)
    → BirdGain
    → ChorusBus
Listen-in: focused BirdGain → ramp up; others → ambient floor (never 0)
MasterGain → DynamicsCompressor → destination
```

Ambient floor gain e.g. 0.18–0.30 of focused; focus ramp 1.2–2.0s ease-in-out both engage/disengage.

### 8.2 Procedural synthesis

`call-grammar` package:
- Motif tables per species
- `synthesizeCall(params) -> AudioBuffer` or live nodes
- Buffer pool/reuse to honor **no memory growth**
- Caption string from same params (`"a soft three-note rise"`)

Unlock AudioContext on first user gesture; until then silent but visual full; if captions default-off, still paint visual.

### 8.3 WebAudio fallback

If unavailable/blocked: **silence + captions forced on**. No MP3 pack. Banner not needed; settings show matter-of-fact “sound unavailable in this browser.”

### 8.4 Listen-in UX binding

Pointer click / tap / keyboard Enter on focused bird toggles listen-in. Click empty, Esc, or other bird changes focus/disengages with slow ramp. Emit start/end events with bird_id + duration on end.

### 8.5 Mixing constraints for 7 birds

Schedule density budget: max simultaneous full-voice phrases ≤ 3; others pedestal peeps. Preserve recognizability over density.

---

## 9. Accessibility surfaces

### 9.1 Screen-reader narration

- `aria-live="polite"` region for spacious updates (30–60s idle).
- Assertive only for rare critical system errors (sync), not bird gambits.
- Priority queue: return-greeting, offer reaction, settle, then ambient.
- Prose naturalist, same voice as notebook.
- Do not dump trait numbers or perch indices (“perch 2”).
- Generation: client templates from snapshot birds + weather + phase; unit-test tone.

### 9.2 Captions

Toggle in a11y settings; auto-on if audio fails. Short phrase fades near bird bbox.

### 9.3 Keyboard

- Tab: top bar controls
- Tab into scene: first bird
- Arrows: cycle birds
- Enter: listen-in
- Esc: exit listen-in / close trays
- Shortcuts documented in a11y settings (matter-of-fact)

Focus ring high contrast for day/night palettes (designer tokens; AA).

### 9.4 Contrast

All chrome text AA; captions AA against local scrim if needed (soft dark pill behind text).

### 9.5 Reduced motion

See §7.4; ship day one.

---

## 10. Performance budgets and observability

### 10.1 Budgets

| Metric | Budget |
|--------|--------|
| Initial JS gzipped | < 2MB |
| Time to first bird visible (mid mobile 4G) | < 500ms |
| Idle FPS (5yo laptop) | 60 |
| Heap growth over 30 min | ~0 (CI assert) |
| Snapshot size | few KB typical |
| Tick p99 | alarm > 5s |

### 10.2 Tactics for TTFB / bundle

- Critical path: HTML shell + minimal scene runtime + inline/bootstrap snapshot for signed-in users (edge cache per-session carefully — prefer post-auth fast API from edge of region).
- Code-split: settings, visit management, export, heavy account.
- No large audio samples.
- Defer notebook virtual list until open.
- Font subsetting; system fonts preferred for calm UI.

### 10.3 Observability (privacy-safe)

Collect:
- request rate/latency/error by route
- tick duration histogram, queue lag
- RUM: navigation timing, first-bird paint, long tasks, fps buckets
- audio context error counts
- session duration histogram **without account_id dimension in warehouse**

Forbidden:
- per-bird interaction streams into analytics
- personality values in logs
- email in logs

### 10.4 CI perf gates

- Bundle size check on PR
- Headless 30-min soak memory
- sim-core drift calibration suite
- a11y axe on chrome routes + narration banlist tests
- screenshot/visual smoke optional

---

## 11. Security, privacy, compliance engineering

- Magic link single-use, 15m expiry; hash tokens at rest.
- Session revoke list; rotate on email change completion.
- Rate limit auth and invite sends.
- Visit tokens unguessable; revoke live.
- Soft delete 30d: hide login as deleted recovery path only; hard purge birds/events/notebook/telemetry-linkable rows.
- Encryption: email at rest; TLS everywhere.
- Privacy architecture test: CI grep / dataflow check that warehouse connectors cannot SELECT simulation tables.

---

## 12. Rollout plan

### 12.1 Phased engineering milestones

**M0 — Foundations (week 1–2)**  
Monorepo, auth magic link, account UUID model, empty quiet field client, CI.

**M1 — Canonical sim spine (week 2–5)**  
Birds tables, tick worker, snapshot API, event log, two starter birds, mid-action load, day/night.

**M2 — Aliveness (week 4–7)**  
Idle motion, procedural audio v1 (2 species), listen-in ramps, presence triad, drift v1, greetings.

**M3 — Interactions + notebook (week 6–9)**  
Offers+cooldowns, settle/undo, notebook generator, rename, age-gate adoption stub.

**M4 — A11y + polish (week 8–11)**  
Narration, captions, reduced motion, keyboard, AA pass, memory soak.

**M5 — Visits + account lifecycle (week 10–12)**  
Invites, visitor mode, visit log, export, delete, multi-device QA.

**M6 — Soft launch**  
Perf budgets green; 6 species; calibrations on dogfood accounts 3+ weeks drift observation before marketing anything about personality (internal only).

### 12.2 Birds-per-aviary ramp

- Launch: hard max 2 for all new accounts (simpler chorus calibration) **or** 2 start with gates disabled until M6 stability — prefer age gates live but slow.
- Enable 3rd bird adoption after dogfood proves audio recognizability at n=3–4.
- Raise toward 7 only with listening tests each +1.

### 12.3 Day-one instrumentation

- first_bird_paint_ms
- snapshot_latency
- tick_lag_seconds
- audio_init_fail_rate
- presence_ping_rate (aggregate)
- JS error rate
- invite_send_rate / revoke_rate

No product analytics on “engagement funnels” that imply streak thinking.

### 12.4 Content freeze rules

Tone linter in CI for user-visible strings on banlist: welcome back, streak, achievement, hunger, XP, leaderboard, feed the, game over, daily goal.

---

## 13. Testing strategy

### 13.1 sim-core pure tests

- Drift monotonicity
- Neglect energy decay without trait decay
- Mood continuity across tick sequences
- Presence summary ignores incomplete triad events
- Offer cooldown
- Adoption cap
- Idempotent event apply

### 13.2 Property tests

Call caption always non-empty when call fires; buffer reuse counts stable; snapshot round-trip version monotonic.

### 13.3 Client e2e

- Magic link happy path (test mailbox)
- Mid-action load fingerprint: first frame birds not zero-velocity stack
- Listen-in ramp no silence on others
- Settle undo 5s window
- Visit cannot POST events
- Reduced motion flag path
- Keyboard-only offer + listen-in

### 13.4 Human calibration loops

Internal “three week club” watches drift; if visible day-to-day, slow clocks; if invisible at day 21, bump k_p slightly. Store knobs server flag configurable without redeploy.

---

## 14. Risks and mitigations

| Risk | Why it hurts | Mitigation |
|------|--------------|------------|
| Drift too fast | Tamagotchi numbers game; destroys “weeks” | Instrument gates; asymptotic caps; dogfood 3-week hold |
| Drift too slow | Screensaver; users leave | Presence-weighted; ensure energy+greet still respond on shorter scales |
| Presence definition leaks | Population drifts overnight | Strict triad; bot spam detection; integration tests with hidden tab |
| LWW / client personality write sneaks in | Silent identity death | Code owners on sim paths; API schema forbid; audit |
| Audio uncanniness / loops | Spell break | Procedural only; package review ban on sample libraries for calls |
| Chorus mud at 5–7 birds | Cap meaning fails | Dynamic voice budget; listening panel QA |
| Greeting feels canned | Product becomes theater | Procedural forms × absence × bird; ban toast |
| Notebook becomes event log | Voice collapses product-wide | Salience gates + tone tests |
| A11y as labels-only | Exclusion of affect | Narration design reviews evaluate charm, not only WCAG checklist |
| Bundle bloat | >500ms first bird | budget CI; procedural visuals |
| Tick backlog | Aviary feels frozen/outdated | shard workers; p99 alarm 5s; skip-locked claim |
| Visit feature scope-creep | Social network gravity | Only listed endpoints; no presence from visitors |
| Magic-link enumeration / abuse | Spam | constant responses; rate limits; abuse mailbox tooling |
| Memory leaks in audio/ornaments | Degrades long sit sessions | 30m CI soak; object pools |
| Reduced motion afterthought | Betrays promise | Same milestone as full motion |
| Export/PII mishandle | Trust break | same privacy lines; signed short TTL URLs |
| Team adds streak “just in settings” | Product identity loss | non-goals checklist in PR template |

### 14.1 Highest priority risk narrative

The failure mode that is **silent and irreversible emotionally** is identity loss: reset vectors, swapped bird IDs, or merge bugs. Engineering priority: backups of `personality_vectors`, irreversible migrations rehearsed, never rebuild vectors from event replay as sole source of truth (events refine; vectors are stored). Event log can be used for audit, but live path is stored vector + delta.

---

## 15. Work breakdown (execution-oriented)

### 15.1 Backend teams

1. **Identity & accounts** — magic link, sessions, settings, delete/export  
2. **Simulation** — sim-core, workers, drift/mood/weather/greeting  
3. **API** — snapshot, events, notebook read, adoption  
4. **Visits** — invites, visitor authz, log  
5. **Jobs** — notebook author, mailer, purge  

### 15.2 Client teams

1. **Scene & motion** — layout, day/night, idle, reduced motion  
2. **Audio** — grammar, mix, listen-in, captions hooks  
3. **Interactions** — presence sensors, offers, settle, greet playback  
4. **Chrome & account UI** — top bar, settings matter-of-fact pages  
5. **A11y** — live region, keyboard, focus rings  

### 15.3 Shared

- Design tokens / palette  
- Species art + motif content  
- Tone style guide tests  
- Perf + privacy platform  

---

## 16. Directory and module map (suggested)

```
packages/sim-core/
  drift.ts
  mood.ts
  perch.ts
  greeting.ts
  weather.ts
  presence_reduce.ts
  tick.ts
packages/call-grammar/
  species/*.json
  synthesize.ts
  caption.ts
apps/web/src/
  scene/
  audio/
  presence/
  a11y/
  chrome/
  api/
services/api/
services/simulation-worker/
```

---

## 17. Acceptance criteria for v1 “done”

1. New user magic-links in, meets two named birds mid-motion within 500ms budget on reference device profile.
2. One bird greets within ~2s without any text welcome.
3. Presence only accrues under triad; hidden tab does not drift.
4. After simulated 3 weeks regular presence, traits measurably up; after simulated neglect, traits flat and greetings rarer via energy.
5. Listen-in ramps; other birds remain audible.
6. Offers cool down; reactions mood-dependent.
7. Settle dims/quiets; undo 5s; tab close without settle is clean.
8. Notebook sparse naturalist entries; no gamification language.
9. Second device shows same moods/names/drift after tick.
10. Visitor read-only; no host notify default; revoke works next snapshot.
11. Screen reader hears ambient prose; reduced motion crossfade path; captions optional/forced on audio fail.
12. Bundle, FPS, memory, tick alarms within budget.
13. Export and delete honor privacy windows.
14. No achievement/streak/death/hunger/discovery surfaces exist in routes or UI.

---

## 18. What “implement next” looks like (for the doing team)

Do **not** start with visit social or polished palette exploration. Start with:

1. Account UUID + magic link  
2. Aviary row + two birds + personality rows  
3. Tick writing moods/perches  
4. Snapshot + mid-action client render  
5. Presence events + drift monotony tests  
6. Procedural single-species call  
7. Greeting variation  

Only then offers, notebook, a11y expansion, visits.

---

## 19. Glossary lock (engineering)

Use PRD terms in code where user-facing or domainful: `Bird`, `Aviary`, `Call`, `Mood`, `PersonalityVector`, `Drift`, `Presence`, `ListenIn`, `Offer`, `Settle`, `FieldNotebook`, `Visit`, `Tick`. Avoid `pet`, `song` (for calls), `xp`, `hunger`, `solo` (for listen-in).

---

## 20. Closing constraint checklist for every PR

- [ ] Does this announce instead of notice?  
- [ ] Does this punish absence?  
- [ ] Does this expose a number the user optimizes?  
- [ ] Does this write personality on the client?  
- [ ] Does this add recorded call audio?  
- [ ] Does this create a streak/social graph/discovery surface?  
- [ ] Does this ship an a11y hole for a new affective feature?  
- [ ] Does this log PII or per-bird romance into telemetry?  

If any yes → redesign before merge.

---

*End of plan. Do not implement product code in the planning slot; this document is the deliverable for phase 1.*
