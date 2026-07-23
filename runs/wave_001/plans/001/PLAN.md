# Pocket Aviary — V1 Implementation Plan

Comprehensive implementation plan for a frontier engineering team. Interprets the PRD into executable architecture, data model, APIs, simulation, sync, rendering, audio, accessibility, performance, rollout, and risks. **Do not implement product code from this document alone without engineering review of calibration constants** — named defaults below are starting points to tune in build.

---

## 1. Scope

### 1.1 In scope (v1)

| Area | Deliverable |
|------|-------------|
| **Surface** | Browser-only SPA; single horizontal aviary scene; thin top bar; no native apps |
| **Accounts** | Single-user accounts; email magic-link auth; synthetic account UUID; session list + revoke; email change with verify; export JSON; soft-delete 30d → hard-delete |
| **Aviary** | One canonical aviary per account; 2 starter birds; age-gated adoption up to **7** birds; user-assigned names; ~6 species pool |
| **Engine** | Server-side ~1/min tick; personality vectors (hidden); mood; procedural call grammars; monotonic drift toward expressive; bird-to-bird influence |
| **Interactions** | Return-greeting; presence accounting; listen-in; offer (seed / song fragment / still pool); settle (+ 5s undo); field notebook (read-only, sparse) |
| **Sync** | Multi-device via server-canonical snapshots + append-only client events; no client personality writes |
| **Social** | Visit invitations OFF by default; per-email invite; read-only ambient visitor; revoke; 30d unused invite expiry; silent visit log; optional visit-notification toggle (off by default) |
| **A11y** | Naturalist SR narration; designed reduced-motion; call captions; WCAG AA chrome; full keyboard path |
| **Perf** | Initial JS ≤ 2MB gzipped; time-to-first-bird ≤ 500ms (mid-tier mobile / 4G); 60fps idle on 5yo laptop; no client memory growth over 30 min |

### 1.2 Explicitly out of scope (enforce in review / lint culture)

- Native iOS/Android apps; offline-first multi-day client simulation ownership
- Gamification: achievements, streaks, scores, badges, levels, XP, calendars of visits, “days visited,” counters of birds adopted as engagement UX
- Tamagotchi mechanics: death, hunger, distress, decaying happiness meters, punishment for absence
- Social network: profiles, follows, public discovery, comments, chat, avatars-in-scene, mutual co-presence, leaderboards, show-off rendering
- Payments, multi-aviary accounts, shared/household aviaries, customizable scenes
- Push/email notifications about aviary life (except magic-link / export / optional visit notify / transactional account mail)
- Numerical personality UI (any tier, any debug-for-users surface)
- Recorded-audio call libraries as primary or fallback path
- Welcome toasts, “you’ve been gone N days,” visit-frequency surfaces in product or notebook

### 1.3 Resolved ambiguities (defensible calls)

| Ambiguity | Decision |
|-----------|----------|
| Exact tick period | **60s** default; configurable 45–90s; instrument p99 tick wall time |
| Presence activity window | **180s** without pointer/key after last activity → presence ends; tune toward longer if watching-without-moving feels broken |
| Mood enum | Fixed v1 set: `wary`, `content`, `curious`, `drowsy`, `alert` (+ internal `settled_sleep` pose tag at night, still one of the five moods mapped) |
| Personality scalar range | Continuous **[0.0, 1.0]**; new birds seeded in **[0.25, 0.55]** with species-biased means and small jitter |
| Presence ping cadence | Client sends presence heartbeats every **30s** while all three conditions hold; server aggregates duration, not count of pings — drop pings when idle/hidden |
| Notebook generation | Tick-adjacent **notebook worker** every 15 min evaluates sparse templates; max ~1 entry / 2–4 active days unless “noteworthy” (first +1 bird, unusual greeting order, first rain of week) |
| Visit “duration” | Server sums visitor snapshot-session open time with 2 min idle timeout; no presence semantics wired to host drift |
| Stack | **TypeScript** monorepo; **React** client; **Canvas 2D** primary scene (SVG birds optional hybrid); **WebAudio** synth; API **HTTPS JSON + SSE/WebSocket optional for live tick push** — v1 may be pull-only with keepalive |
| Auth delivery | Transactional email provider; magic link single-use; CSRF + HttpOnly secure cookies or rotatable bearer refresh for API |
| Locale | Product copy English v1; timezone from browser `Intl` / account-stored preferred TZ set at first session |

---

## 2. Architecture

### 2.1 High-level shape

```
┌─────────────────────────────────────────────────────────────┐
│ Browser client                                              │
│  App shell · Auth · Settings · Visit accept                 │
│  Scene graph (layout, perches, weather ornament)            │
│  Bird renderers + idle / transition FSMs                    │
│  Audio graph (call synth, chorus mix, listen-in ramps)      │
│  Presence monitor · Interaction event emitter               │
│  Narration / captions / reduced-motion adapters             │
└───────────────┬──────────────────────────▲──────────────────┘
                │ events + snapshot pulls  │ snapshots / auth
┌───────────────▼──────────────────────────┴──────────────────┐
│ Edge / API tier                                             │
│  Auth service · Account · Visit · Export/Delete             │
│  Aviary snapshot API · Event ingest API                     │
│  Static CDN (SPA + free low-risk assets)                    │
└───────────────┬─────────────────────────────────────────────┘
                │
┌───────────────▼─────────────────────────────────────────────┐
│ Simulation & data                                           │
│  Sim tick workers (per-shard schedule)                      │
│  Event log (append-only) · Canonical aviary state store     │
│  Notebook generator · Email outbox · Audit for sessions     │
│  Ops telemetry (aggregate only) — isolated from sim PII DB  │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Service boundaries

| Service | Responsibility | Owns writes |
|---------|----------------|-------------|
| **auth-api** | Magic link issue/consume, sessions, revoke, email change | `accounts`, `sessions`, encrypted email |
| **aviary-api** | Snapshots, event ingest, offer cooldown checks (rate), settle, names, adoption eligibility read | event log append; name updates; note: personality only via sim |
| **sim-worker** | Consume events, drift, mood, weather scheduling, bird-to-bird, perch intent, call schedule seeds, notebook candidates | canonical `birds`, `aviary_ambient`, notebook rows |
| **visit-api** | Invites, token validate, visitor snapshot (stripped-write), revoke, visit log | `visits`, `visit_sessions` |
| **account-lifecycle** | Soft/hard delete, export job | orchestrates redaction |
| **notify-mailer** | Magic links, export links, optional visit email | outbox only |

Clients **never** write personality, mood absolute values, perch as layout commands, or notebook content.

### 2.3 Client / server / render boundary

- **Server**: truth for continuous time, personality, mood, weather schedule, perch *intent*, call grammar seeds / next-call windows, notebook text, visit ACLs.
- **Client**: presentation interpolation, idle micro-motion detail within sides allowed by snapshot, procedural audio synthesis from grammar params, presence sensing, local settle undo window (5s) reconciled with server.
- **Render pipeline** does not advance proverbial “sim time” for drift; if client is offline, no local drift applied on reconnect beyond pulling server state.

### 2.4 Monorepo layout (suggested)

```
apps/web/                 # SPA
apps/api/                 # HTTP gateway (auth + aviary + visit)
services/sim-worker/
packages/sim-core/        # pure TS: drift, mood FSM, call grammar AST, notebook templates
packages/shared-types/
packages/audio-grammar/   # motif defs shared client+server caption strings
packages/a11y-narration/
infra/                    # terraform/k8s, migrations
```

`sim-core` is pure and unit-tested with golden vectors so client narration helpers and server tick share mood/call caption semantics without duplicating rules.

---

## 3. Data model

### 3.1 Account

```text
Account {
  id: UUID                     // synthetic; ONLY external key
  email_ciphertext: bytes
  email_hash: bytes            // for lookup; not used as partition key elsewhere
  created_at: timestamptz
  preferred_timezone: string   // IANA
  soft_deleted_at: timestamptz?
  hard_delete_after: timestamptz?
  settings: jsonb {
    visit_notifications: boolean = false
    captions_default: boolean
    reduced_motion_override: 'system'|'on'|'off'
    audio_enabled_preference: boolean
  }
}
Session {
  id: UUID
  account_id: UUID
  created_at, last_seen_at, user_agent, ip_hash
  revoked_at?
}
MagicLink {
  id, account_id?, email_hash, token_hash, expires_at, consumed_at?
}
```

### 3.2 Aviary

```text
Aviary {
  id: UUID
  account_id: UUID UNIQUE
  created_at: timestamptz      // age clock for bird #3+
  bird_cap: int = 7
  settled_until?: timestamptz  // client settle soft state may also round-trip
  last_host_presence_at?
  version: bigint              // monotonic snapshot version
}
```

### 3.3 Bird

```text
Bird {
  id: UUID                     // stable identity forever
  aviary_id: UUID
  species_id: string           // from pool
  display_name: string
  adopted_at: timestamptz
  sort_seed: float             // stable visual tie-break
  // Personality — server only canonical
  boldness: float
  social_warmth: float
  vocal_frequency: float
  plumage_saturation: float
  curiosity: float
  personality_updated_at: timestamptz
  // Mood
  mood: enum
  mood_updated_at: timestamptz
  // Placement intent (sim)
  perch_zone: 'front'|'middle'|'back'
  perch_slot: int              // discrete slots per zone to avoid stacking
  pose_hint: string            // preen|scan|rest|call|approach|...
  motion_phase: float          // 0–1 seed so first client frame is mid-action
  call_state: {
    grammar_seed: uint64
    next_call_at: timestamptz
    last_motif_id: string
  }
}
```

**Invariant:** rename changes `display_name` only. Species pool hotfixes never recycle `bird.id`.

### 3.4 Species pool (v1 ~6)

Each species defines: silhouette key, default plumage bases, motif library id, night-active flag (exactly one nightjar-like), baseline trait biases, silhouette scale.

### 3.5 Interaction event log (append-only)

```text
InteractionEvent {
  id: UUID                     // client-generated UUIDv7 ok if collision-safe
  account_id: UUID
  aviary_id: UUID
  device_session_id: UUID
  type: enum
  payload: jsonb
  client_ts: timestamptz
  server_received_at: timestamptz
  dedupe_key: string           // idempotency
}
```

**Event types:**

| type | payload (core) |
|------|----------------|
| `presence_ping` | `{ window_start, window_end, focus: true }` duration segment |
| `listen_in_start` / `listen_in_end` | `{ bird_id }` |
| `offer` | `{ kind: seed\|song\|pool, target_hint?: bird_id, offer_id }` |
| `settle` / `settle_undo` | `{}` |
| `return_visible` | `{ last_hidden_at?, absence_seconds }` |
| `name_change` | `{ bird_id, name }` (or separate CRUD; either way not personality) |
| `session_hello` | `{ tz, viewport, a11y_flags }` |

Sim consumes events **in `server_received_at` order** (ties by id). Personality updates are **deltas computed only inside sim-worker**.

### 3.6 Cooldowns & rate state

```text
OfferCooldown {
  bird_id, offer_kind, available_at
}
// Global soft rate: max offers per aviary per hour to prevent spam without “punishment” UX
```

### 3.7 Field notebook

```text
NotebookEntry {
  id: UUID
  aviary_id: UUID
  created_at: timestamptz
  day_key: local-date string
  body: string                 // naturalist prose, lowercase present tense
  salience: float              // internal only
  source_event_ids: UUID[]     // for debug, not user-visible
}
```

No user edit/delete. Infinite scroll retention for v1 (practical archive later if needed without UI “archive”).

### 3.8 Ambient schedule

```text
AmbientState {
  aviary_id
  weather: 'clear'|'rain'|'wind'
  weather_until: timestamptz
  next_weather_roll_at: timestamptz
}
```

Weather is examined on tick; rarity: few rains/week — poisson-like schedule keyed by aviary_id hash so multi-device identical.

### 3.9 Visits

```text
VisitInvite {
  id: UUID
  host_account_id: UUID
  visitor_email_ciphertext / hash
  token_hash
  created_at, expires_at       // unused → 30d
  revoked_at?
  accepted_at?
}
VisitSession {
  id, invite_id, started_at, ended_at?, approx_duration_s
}
// Visit log is query over sessions + outstanding invites — settings UI only
```

Visitor browsing uses host snapshot **with write APIs denied** (capability token scoped `visit:read`).

### 3.10 What we never store in analytics warehouse

Per-bird traits, event payloads tied to bird behavior, notebook bodies, listen-in targets. Ops metrics = counts/latencies/histograms without account/bird dimensions that reconstruct relationships.

---

## 4. API surface

All authenticated host APIs require session; IDs in paths are UUIDs, never email.

### 4.1 Auth

| Method | Path | Notes |
|--------|------|-------|
| POST | `/auth/magic-link` | body `{ email }`; always generic 200; rate limit per email_hash + IP |
| GET | `/auth/magic-link/consume?token=` | single-use; 15 min TTL; sets session |
| POST | `/auth/sign-out` | |
| GET | `/account/sessions` | list devices |
| DELETE | `/account/sessions/:id` | revoke |
| POST | `/account/email-change` | start |
| POST | `/account/email-change/confirm` | |
| GET | `/account` | settings bundle |
| PATCH | `/account/settings` | matter-of-fact validation errors |
| POST | `/account/export` | async; email download link |
| POST | `/account/delete` | soft |
| POST | `/account/delete/cancel` | within 30d |

Error copy: matter-of-fact (see PRD samples).

### 4.2 Aviary state

| Method | Path | Notes |
|--------|------|-------|
| GET | `/aviary` | full snapshot for host |
| GET | `/aviary/snapshot?since_version=` | delta or full if gap |
| POST | `/aviary/events` | batch append; idempotent `dedupe_key` |
| PATCH | `/aviary/birds/:id` | `{ name }` only |
| GET | `/aviary/adoption` | `{ eligible: bool, next_eligible_at? }` |
| POST | `/aviary/adoption/accept` | when eligible; server assigns species; client may send names |
| GET | `/aviary/notebook?cursor=` | chronological |
| POST | `/aviary/settle` | optional explicit; also Accept via event |
| POST | `/aviary/settle/undo` | only if within server-validated 5s of settle |

**Snapshot JSON (illustrative):**

```json
{
  "version": 184422,
  "server_time": "2026-07-24T15:01:02Z",
  "timezone": "America/Los_Angeles",
  "local_day_phase": "afternoon",
  "lighting": { "palette_key": "day", "settle_blend": 0.0 },
  "weather": { "kind": "clear", "intensity": 0 },
  "birds": [
    {
      "id": "…",
      "name": "Pip",
      "species_id": "warbler_a",
      "mood": "content",
      "perch": { "zone": "front", "slot": 1 },
      "pose_hint": "preen",
      "motion_phase": 0.37,
      "plumage_saturation": 0.41,
      "call": {
        "grammar_id": "warbler_a",
        "seed": "0x…",
        "next_call_at": "…",
        "mix_weight_hint": 1.0
      },
      "personality_public": {
        "boldness_band": "mid",
        "warmth_band": "high"
      }
    }
  ],
  "greeting": {
    "primary_bird_id": "…",
    "absence_band": "short|medium|long",
    "stagger_ms": [0, 420],
    "variant_seed": "…"
  },
  "active_offer_fx": null,
  "caps": { "bird_count": 2, "bird_cap": 7 }
}
```

Note: snapshot may expose **coarse bands** for rendering only if needed; **never** raw traits in any user UI. Prefer deriving pose gravitationally on client from mood + prey fields already listed without sending numbers to the DOM/a11y tree.

### 4.3 Event ingest contract

```http
POST /aviary/events
{
  "events": [
    {
      "id": "uuid",
      "dedupe_key": "session:…:presence:1710000000",
      "type": "presence_ping",
      "client_ts": "…",
      "payload": { "duration_ms": 30000 }
    }
  ]
}
```

Response: `{ accepted: [...], duplicates: [...], rejected: [...] }` with matter-of-fact reasons (cooldown, visit token invalid, etc.).

### 4.4 Visit flow

| Method | Path | Actor |
|--------|------|-------|
| POST | `/visits/invites` | host `{ email }` |
| GET | `/visits/invites` | host list outstanding + recent |
| DELETE | `/visits/invites/:id` | host revoke |
| GET | `/visits/log` | host |
| GET | `/visit/:token` | visitor bootstrap (read token) |
| GET | `/visit/:token/snapshot` | visitor pull only |
| POST | `/visit/:token/beacon` | optional duration ping; **does not** create host presence events |

Visitor snapshot omits notebook write APIs; notebook may be readable or hidden—**decision: hide notebook from visitors** to keep journal private to host relationship; still show birds/audio/day/weather. (Defensible privacy call; note in API docs.)

Revoked/expired: matter-of-fact “This visit is no longer available.”

### 4.5 Transport cadence

- On load / `visibilitychange` → visible: immediate snapshot.
- While visible + focused: snapshot keepalive every **20–30s** OR SSE `snapshot_version` nudge when tick bumps version (prefer SSE if ops ready; else poll).
- After long `requestAnimationFrame` gap (>2s): treat as resume, full snapshot.
- Events: coalesce presence to ≤1/30s; flush offers/listen-in immediately.

---

## 5. Simulation engine design

### 5.1 Tick loop (~60s)

For each aviary due (shard by `aviary_id`):

1. **Load** canonical birds + ambient + last tick timestamp.
2. **Ingest** new events since last cursor; **fold** into intermediate signals:
   - presence_seconds in window
   - per-bird listen_in_seconds
   - offers (kind, target, accepted?)
   - settle markers
3. **Advance ambient wall clock** from previous tick time → now (handle multi-minute catch-up if worker lagged): day phase from aviary timezone; weather transitions if due.
4. **Mood step** (fast timescale) per bird.
5. **Bird-to-bird** propagation (alarm/wary bleed, chorus clustering).
6. **Perch intent** from mood × boldness.
7. **Call schedule** from vocal_frequency × mood × weather dampers × night rules.
8. **Drift step** (slow timescale) — tiny deltas only from signals; clamp [0,1]; **no negative drift**.
9. **Notebook evaluator** (usually no-op).
10. **Adoption eligibility** age gates.
11. **Persist** new state; bump `version`; emit optional pub/sub for connected clients.

Catch-up: if last tick was hours ago, run **multi-step** mood/daylight progression in ≤N substeps (e.g. max 60 steps) to avoid leaping mood absurdly, but drift uses integrated presence from events only (absence ≠ negative drift).

### 5.2 Drift function

Treat traits as slow LPGs over attention signals.

**Target calibration (testable):**

- After **7 days** simulated “regular visits” (e.g. 20–40 min presence/day on fixtures): each quality metric moves by instrument-detectable Δ (e.g. **≥0.01** on at least one trait under CI harness).
- After **~21 days**: human-visible behavior change in staging dogfood checklist (perch bias, greet order stability, call rate).
- **Single session** Max trait Δ hard-capped (e.g. **≤0.005** total L2) so nothing “clicks up a number.”

**Weights (initial, tune):**

```text
presence_seconds       → mild lift all expressive traits, especially plumage_saturation & social_warmth
listen_in_seconds      → stronger social_warmth + vocal_frequency on that bird
offer_near_bird        → small boldness
offer_accepted         → small curiosity
settle                 → no drift direction (presence end only)
neglect / absence      → zero negative Δ; ambient schedule continues
```

**Monotonic expressivity:**  
`trait := min(1, trait + max(0, delta))`  
Never decrease trait on neglect. “Quieter” behavior under low historical presence is achieved by **not raising** greeting/call rates that high-warmth birds get — i.e. baseline rates use absolute trait levels; low attention birds simply stay near seed means.

**Implementation tip:** store traits as float; apply deltas only in sim-core with pure functions:

```ts
applyDrift(traits, signals, dtHours) -> traits
```

Golden tests with fixed seeds.

### 5.3 Mood transitions

State machine on `{ mood, mood_updated_at }` with rates from inputs:

| Input | Tendency |
|-------|----------|
| Early local morning | → `alert` |
| Dusk / settle lighting | → `drowsy` |
| Offer accepted | → `content` / `curious` |
| Rain | temporary vocal damp; slight `wary` or calmer `content` species-dependent |
| Nearby bird `wary` + alarm call flag | → `wary` contagion short half-life |
| High boldness | lower transition probability into `wary` |

**Persistence:** no reset on tab open. Client first frame uses server mood.

### 5.4 Greeting planner (on host `return_visible` / first snapshot after absence)

- Select primary greeter: rank by `boldness * social_warmth` × mood modifiers × RNG seed from `(bird_id, day_key, absence_band)`.
- Absence bands: `<10m` glance; `10m–6h` short call; `>6h` fuller re-orientation (front step, longer call). Multi-day uses longer band without “shame” framing.
- Secondary birds stagger 200–800ms random offsets — **never** simultaneous on-cue chorus.
- Greeting is **snapshot field** + client animation; not a toast.

### 5.5 Call-grammar runtime

Server side: schedules `next_call_at`, assigns `grammar_seed`, motif family.

Client side `audio-grammar`:

- Motif library per species: small graph of notes (freq envelope, duration, vibrato, gaps).
- At call time: expand seed → motif sequence with personality pitch/timing skew (vocal_frequency, mood).
- Recognizability: keep **carrier motif identity** stable per species+bird seed salt so Pip stays Pip across mood.
- Chorus: multiple simultaneous voices mixed; slight humanization offsets avoid phase-lock artifacts of loops.

Caption string generated from **same expansion** (`describeMotif(seq)` → “a soft three-note rise”).

### 5.6 Offer resolution

On event `offer`:

1. Enforce per-bird cooldown (~3–5 min, kind-specific).
2. Choose receiving bird: spatial proximity if client sent target_hint else curiosity/mood weighted.
3. Roll TTC reaction: approach seed / ignore / delayed approach / drink-bathe / join-or-counter song.
4. Write short-lived `active_offer_fx` into snapshot for clients to animate/audio.
5. Feed signals into next drift step.

### 5.7 Bird-to-bird

Each tick: for calls in a shared time window, raise reply probability vs neighbor `social_warmth` and `vocal_frequency`. Wary contagion with exponential decay. Chorus event flag for audio densification without stacking identical loops.

### 5.8 Adoption eligibility (age, not engagement score)

Example ladder (tune):

| Aviary age | Max birds |
|------------|-----------|
| 0 | 2 (starters) |
| 45d | 3 |
| 120d | 4 |
| 210d | 5 |
| 300d | 6 |
| 400d | 7 |

Server assigns species to maximize pool diversity in aviary; user names only. Presentation: “birds that arrived,” not catalog.

### 5.9 Nightjar exception

Species flag `nocturnal_active`: remains pose-active and may call at night when others use sleep poses; night is not a dead scene.

---

## 6. Sync model

### 6.1 Single canonical writer

- Only **sim-worker** mutates personality & authoritative mood/perches/call schedule.
- Clients append events; multiple devices → one log → one fold.

### 6.2 Why not LWW

Absolute client personality submits are rejected at API schema level. Conflicts based on “latest snapshot blob wins” must never apply to bird vectors. If two devices presence-ping concurrently, durations **add** (capped per wall-clock to avoid double-count FPS quirks) via merge of time ranges — interval merge in sim when folding presence.

**Presence double-count defense:** lift presence by **union of intervals** per account across devices, not sum of independent “I’m present” if wall clocks overlap. Device A and B both open rare; union keeps calibration honest.

### 6.3 Snapshot versioning

Monotonic `version`. Clients with `since_version` get:

- if `server.version - since < threshold` and deltas retained: patch  
- else full snapshot  

After hard gaps (sleep), always full.

### 6.4 Client offline

Queue events in IndexedDB with dedupe keys; flush on reconnect. Do not apply speculative drift. On restore, presence only elapsed while actually meeting three conditions (client must not backfill fake presence for offline).

### 6.5 Settle across devices

Settle is host gesture; sets lighting converge + call quiet. Other device pulling snapshot sees settled blend. Undo muted if undo window expired server-side.

### 6.6 Visit isolation

Visitor tokens cannot POST `/aviary/events`. Visitor beacons only hit visit-api duration. Host never gets push by default.

### 6.7 Failure surfaces (voice)

Matter-of-fact only: expired magic link, session timeout, aviary load failure, visit unavailable. No naturalist evasion.

---

## 7. Frontend rendering pipeline

### 7.1 Scene composition

Layers back → front:

1. Sky / day-night gradient (local time + settle blend)
2. Distant foliage (soft)
3. Back perch zone birds
4. Mid foliage / middle perch birds  
5. Front perch birds + still-pool FX when active
6. Occasional foreground leaf ornaments (client-only)
7. Captions layer (a11y)
8. Top bar (DOM, not canvas)

**No** buttons inside scene. Birds are hit targets for listen-in / keyboard focus only.

### 7.2 Coordinate system

Logical scene width/height with letterbox/pillarbox as needed; **compress horizontally** on narrow viewports without cropping birds. Recompute perch world-x on resize; animate layout reflow gently (respect reduced motion).

### 7.3 Three perch zones

Sim assigns zone + slot. Client maps to world coordinates with safe padding. User cannot drag birds.

### 7.4 First frame already alive

- Prefetch: inline critical CSS + tiny boot script; snapshot endpoint cacheable briefly per session token at edge if safe.
- Boot path: quiet field (soft sky, minimal leaf optional) **without spinner** while waiting cold snapshot.
- On snapshot: instantiate birds at `motion_phase`, begin update loop mid-cycle — **no** fade-from-static hero.
- Empty aviary only during first adoption; birds soft fly-in once; never empty again.

### 7.5 Idle micro-motion

Continuous local FSMs: preen, weight-shift, scan, head-tilt toward call direction, fluff for drowsy. Parameters from mood. **Simulation of personality doesn’t live here** — only expression.

When `document.hidden`, cancel rAF and pause audio; sim continues server-side.

### 7.6 Interpolation

Between snapshots: lerp perch positions if zone change; cross-fade poses when reduced motion; full path tween when motion OK. Call triggers from schedule even if between snapshots (client trusts `next_call_at` + local clock skewed to `server_time`).

### 7.7 Top bar

Icons: account/settings, a11y, notebook, offer. Optional settle control. After ~3s cursor stillness → opacity ~0.15; restore on pointer/keyboard activity. Sparse.

### 7.8 Day/night & weather rendering

Palette shifts continuous over clock. Rain: light particle veil + darker leaves; never storms. Wind: leaf vector field boost. Client ornaments not in sim.

### 7.9 Reduced-motion mode

Detect `prefers-reduced-motion` + settings override. Replace continuous animation with **slow cross-fades** between authored still poses; remove ambient leaf drift; keep slowed day color shifts. Audio and true sim state unchanged.

### 7.10 Color / design system

Calm naturals; no loud accents in scene. External design tokens for WCAG AA chrome. Focus ring soft high-contrast halo valid on day and night palettes.

### 7.11 Tech choice detail

**Canvas 2D** with pre-rendered sprite sheets or lightweight vector paths per species; avoid heavy WebGL unless sprint proves CPU bottleneck. Keep main-thread budget: audio work in AudioWorklet where possible; ornment spawn pool.

---

## 8. Audio pipeline

### 8.1 Graph

```
MotifSynth (per voice) → birdGain → chorusBus → listenInPanner/weight → masterGain → destination
Ambient soft bed (optional very low) 
```

No sampled call libraries in bundle.

### 8.2 Procedural synthesis

- Oscillators + filtered noise bursts + envelope ADSR per note token.
- Species motifs in compact JSON/binary in `packages/audio-grammar`.
- Variation: seed-derived micro pitch, duty, gap jitter each call.

### 8.3 Listen-in mix

On engage: **ramp ~1.2–2.0s** focused birdGain up; others down to floor **≠ 0** (e.g. multiply by 0.25–0.4). On disengage: reverse ramp. Disengage: second click, other bird, empty scene click, focus leave, Escape.

Feel “listening,” not bank of solo mute buttons.

### 8.4 Chorus

Allow overlapping calls; slight start offsets; ducking only mild to prevent mud. Cap simultaneous peak voices with priority (focused bird, greeter, nightjar).

### 8.5 Settle / night

Multiply global call rate and master activity; settle gesture pulls envelope down.

### 8.6 WebAudio fallback

If `AudioContext` unavailable or blocked: **silence + captions default ON**. No MP3 fallback path. Prompt to enable audio is matter-of-fact, non-Naggy, not a blocking modal every visit (session-remembered dismiss).

### 8.7 Autoplay policy

First user gesture may be required to resume context; birds can animate pre-audio; on gesture unlock, gentle fade-in — not a fanfare.

### 8.8 Memory

Pooled buffers; no per-call undiscussed allocations retained; CI soak test 30 min heap delta ≈ 0.

---

## 9. Accessibility surfaces

### 9.1 Screen-reader narration

- Live region (polite) fed with **naturalist prose paragraphs**, not trait dumps.
- Idle cadence **30–60s**; continuous replacements that don’t interrupt mid-utterance carelessly (queue management).
- Priority bump: return-greeting, offer reactions, settle — still observational phrasing.
- Generation: pure function `narrate(snapshot, lastSummary)`; share voice rules with notebook.
- Do **not** bind ARIA to raw boldness numbers.

Example:

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

### 9.2 Captions

Settings opt-in (and auto-on when audio fails). Short prose near bird, fade with call, from grammar descriptor. AA contrast.

### 9.3 Keyboard

| Key | Action |
|-----|--------|
| Tab | Top bar ↔ birds |
| Arrows | Move bird focus |
| Enter / Space | Toggle listen-in |
| Escape | Exit listen-in / close sheets |
| Shortcuts in bar | Offer menu, settle, notebook |

Visible focus rings always.

### 9.4 Reduced motion

Designed surface (see §7.9), ships day one.

### 9.5 Contrast & semantics

DOM for chrome/settings; canvas has accessibility tree bridge: each bird as focusable button/role with accessible name = bird name + short pose phrase updated slowly (not every frame).

### 9.6 Visit mode a11y

Visitor gets narration + captions; no interaction controls presented.

---

## 10. Performance budgets and observability

### 10.1 Budgets (CI / synthetic gates)

| Metric | Budget |
|--------|--------|
| Initial JS (gzipped, critical path) | **< 2MB** |
| Time to first bird pixels | **< 500ms** mid-tier mobile 4G synthetic |
| Idle FPS | **60** on reference 5-year laptop profile |
| Heap growth 30 min soak | **~0** (threshold e.g. < 5MB true leak) |
| Snapshot payload | low single-digit KB typical |
| Sim tick p99 | **« 5s** alarm; target p99 < 200ms/aviary healthy |

### 10.2 Techniques to hit budgets

- Code-split settings, account, visit accept, notebook virtualized list.
- Procedural audio & lean art.
- Critical snapshot with HTML boot or HTTP2 early data where applicable.
- rAF pause when hidden.
- Avoid large monolyth UI libs on landing path.

### 10.3 Observability (aggregate only)

Synthetic browsers in major geos on schedule: load, first-bird, 5 min FPS, audio context errors.

RUM: navigation timing, first-bird mark, long tasks, audio errors, API latency — **no per-bird; no account-linked behavioral maps**.

Sim metrics: tick duration histogram, lag (time since due), event backlog depth, error rate.

**Privacy pipeline split:** sim DB credentials never attached to warehouse ETL. Separate ops project.

### 10.4 Deliberately not measured for product analytics

Visit streaks, DAU gamification funnels, trait distributions for “balancing engagement,” recommendation features. Operational health only.

---

## 11. Rollout

### 11.1 Phased delivery (engineering milestones)

| Milestone | Outcome |
|-----------|---------|
| **M0 Foundations** | Monorepo, auth magic link, account UUID discipline, empty shell |
| **M1 Canonical sim** | Aviary+2 birds, tick, snapshots, presence ingest, drift pure fn + tests |
| **M2 Scene & audio** | Canvas scene mid-motion first frame, WebAudio grammar, listen-in ramps |
| **M3 Interactions** | Greetings, offers+cooldowns, settle+undo, notebook generator sparse |
| **M4 A11y** | Narration, captions, reduced-motion, keyboard, AA audit |
| **M5 Multi-device & lifecycle** | Sync soak, export/delete, session revoke |
| **M6 Visits** | Invite/revoke/visitor read-only, log, optional email notify |
| **M7 Perf hardening** | Bundle basement, 500ms path, 30m soak, synthetic fleet |
| **M8 Closed dogfood** | 3–6 week drift calibration, adjust weights |
| **M9 V1 launch** | Gradual accountcreate; bird-cap remains 7; age ladder live |

### 11.2 Birds-per-aviary ramp

- Launch: everyone starts at **2**.
- Age ladder unlocks 3–7 without ads or engagement scores.
- Do not raise cap above 7 without new +listening tests for call recognizability.

### 11.3 Instrumentation from day one

- Auth success/fail rates (no email contents in logs).
- Snapshot latency, tick lag.
- Client first-bird timing.
- AudioContext failure rate.
- Event ingest reject rates (cooldowns).
- **Not:** feeder charts that look like streak dashboards internally that later leak.

### 11.4 Feature flags

Kill switches: visits, weather, notebook writer, SSE. Never flag that “enables achievements.”

### 11.5 Content freeze discipline

Copy review for naturalist vs matter-of-fact surfaces. PR template checklist includes “no Welcome back toast / streak / traits UI.”

---

## 12. Risks and mitigations

| Risk | Why it hurts | Mitigation |
|------|--------------|------------|
| **Drift too fast** | Tamagotchi / trait-chasing | Hard session caps; week/3-week golden tests; dogfood; monotonic + presence union |
| **Drift too slow** | Screensaver feel | Instrument weekly Δ; increase presence weight before adding new shiny UX |
| **Presence definition wrong** | Silent global calibration corruption | Enforce 3-condition client; server validates timestamps sane; refuse “tab open” heuristics |
| **Client personality write bug** | Identity death invisible | Schema forbid; only sim-worker role credentials can UPDATE traits; anomaly monitors on large Δ |
| **LWW multi-device** | Lost mornings | Event-sourced deltas only; integration tests two-device escort |
| **Canned audio** | Spell breaks | No samples path; review gate; procedural variation tests (spectral/hash diversity) |
| **Listen-in hard cuts** | Channel-switcher feel | Enforce min ramp; UX QA |
| **Greeting chorus unison** | Announces arrival | Stagger algorithm tests |
| **Notebook as event log** | Voice collapse | Template lint; human editorial fixtures; sparsity quota |
| **A11y treated as ARIA stickers** | Second-class product | Narration design review; ship with M4 before public |
| **Reduced motion “animations:none”** | Broken scene | Pose cross-fade asset pack |
| **Bundle bloat** | Miss 500ms lived experience | Budget CI fail; code-split |
| **Audio uncanny / harsh** | Users mute forever | Composer/designer tuning; soft limiter; species motifs playtests |
| **Visit feature grows social network** | Product center of gravity shift | Code owners on visit-api; refuse discover endpoints in RFC |
| **PII via email IDs** | Compliance failure | Synthetic UUID lint in CI (grep bans email as FK) |
| **Worker lag** | Mood jump / stale return | Tick lag alarms; catch-up substeps |
| **Offer spam saturates curiosity** | Engine collapse | Cooldowns + acceptance probability rules |
| **Memory leaks in audio/orphans** | Tab sluggish | 30m soak CI |
| **Empty promises of aliveness on load** | Spinner culture regresses | Forbidden spinner in aviary critical path; quiet field only |
| **Internal gamification metrics** | Cultural backdoor | Explicit non-goal; no green-dot even in admin for “engagement of presence” peddling |

### 12.1 Calibration playbook

1. Define fixture user scripts: “daily gentle presence,” “weekend only,” “listen-in heavy,” “neglect 14d return.”
2. Run sim-core offline accelerated time.
3. Compare trait trajectories & greeting rates tables.
4. Staging dogfood ≥3 weeks before calling drift “done.”
5. Only then freeze default weights for launch; keep server-side config for emergency slowdown.

### 12.2 Security notes (v1 proportional)

Magic link entropy + single use; session revoke; visit tokens unguessable; rate limits; export links expiring; soft-delete recovery path; no password DB.

---

## 13. Client interaction map (implementation checklist)

| User action | Client | Server |
|-------------|--------|--------|
| Open tab | Snapshot; presence monitor start; play greeting motion | Log return; greeting fields in snap |
| Idle watch | presence_ping intervals if visible∧focus∧recent input | Union intervals → drift |
| Listen-in | Ramps + focus ring; events start/end | Listen seconds → drift weights |
| Offer | Top bar sheet; wait reaction FX | Cooldown; fx; curiosity/boldness signals |
| Settle | Lighting blend; 5s undo | settle event; undo window |
| Close tab | flush events; stop rAF | presence end |
| Notebook open | Fetch entries | Read-only |
| Invite friend | Email form in settings | Mail token |
| Visitor open link | Read snapshot loop | Visit sessions; no drift |

---

## 14. Testing strategy

### 14.1 sim-core unit

- Drift monotonicity property tests.
- Mood transition tables.
- Presence interval merge.
- Call seed stability / caption pairing.
- Notebook sparsity limits.
- Greeting stagger non-collision.

### 14.2 API integration

- Magic link consume once.
- Event idempotency.
- Visitor cannot event-write.
- Personality column unchanged on name PATCH.
- Delete soft/hard.

### 14.3 Client e2e (Playwright)

- First paint quiet field → birds mid-motion without spinner assertion.
- Keyboard listen-in path.
- Reduced motion class behavior.
- No “Welcome back” string anywhere in DOM hardening test.

### 14.4 Perf

Lighthouse CI subset + custom first-bird mark; bundle size artifact gate; heap soak.

### 14.5 A11y

axe on chrome surfaces; screen-reader scripted glance tests for live region cadence.

---

## 15. Voice & copy system

Centralize strings:

- `copy/naturalist/*` — notebook, narration, captions, offer prompts.
- `copy/system/*` — auth, errors, settings, visits unavailable.

Lint: naturalist samples lowercase-forward; system scripts normal English prose. Ban list: “achievement,” “streak,” “welcome back,” “level up,” “happiness.”

---

## 16. Privacy architecture (enforcement)

| Data | Allowed use |
|------|-------------|
| Interaction events | Drive **that** user’s sim only |
| Personality / notebook | User export + sim; no secondary aggregate ML |
| Ops telemetry | Aggregate performance/errors |
| Visit log | Host transparency |

Concrete: separate database roles; ETL allowlist columns; code review rule “no cross-account bird arrays.”

---

## 17. Success criteria for v1

Ship when:

1. Dogfooders can recognize ≥2 birds by call alone after 2 weeks.
2. Instruments show weekly drift; humans notice over ~3 weeks without trait UI.
3. Multi-device same mood/perch within one tick period.
4. Accessibility dogfood: SR user describes a “place,” not a dashboard.
5. Perf budgets green on synthetic mid-tier profile.
6. Zero gamification/streak surfaces in production strings.
7. Neglect 14 days → quieter expressivity without distress UX.

---

## 18. Non-goals residual reminder for implementers

If a proposal is easier because adjacent products do it (profiles, push “friend visited,” hunger, wide scenes, more birds for FOMO), it is probably wrong for Pocket Aviary. Depth is slow psychiatric weather of small birds under a quiet window — not content volume.

---

## 19. Open implementation tickets (not blockers for planning)

- Final motif DSP by sound design pass.
- Exact species silhouettes in design system doc.
- Whether notebook partially visible to visitors (**default hide** per §4.4).
- SSE vs poll for snapshot nudge.
- Precise age-ladder days after dogfood aesthetic.

Each can be decided in-build without reversing architecture.

---

*End of plan. Deliverable is planning only; no product implementation accompanies this document.*
