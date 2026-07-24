# Pocket Aviary — Phase 1 Implementation Plan

## 1. Scope

### 1.1 In scope (v1)

Pocket Aviary is a browser-only, single-aviary-per-account product: two starter birds growing to a hard cap of seven, server-authoritative simulation, and a calm interaction surface (presence, listen-in, offer, settle, field notebook). Ship:

| Area | v1 deliverable |
|---|---|
| Client | Single-page web app (modern browsers only) |
| Auth | Email magic link; per-device sessions; revoke from settings |
| Simulation | Server tick ~1/min; personality drift; mood; bird-to-bird prompt |
| Interactions | Return-greeting, listen-in, offer (seed / song fragment / still pool), settle, presence accounting |
| Content surfaces | Field notebook (auto, sparse, read-only), screen-reader narration, call captions |
| Visual | Single horizontal scene, 3 perch zones, local day/night, rare weather, top-bar chrome |
| Audio | Client-side procedural WebAudio calls + chorus mix; silence+captions fallback |
| Social | Opt-in visit invites by email; read-only ambient; revocable; off by default |
| A11y | Narration, reduced-motion as designed surface, captions, keyboard, WCAG AA chrome |
| Ops | Export JSON, soft-delete 30d then hard, aggregate-only telemetry |

### 1.2 Explicitly out of scope

Respect `non_goals.md` and product brief refusals:

- No native apps; web protocols only—do not design for native-client constraints.
- No gamification: streaks, badges, XP, levels, counters, green-dot calendars, “days visited,” milestone celebrations (including camouflaged notebook variants about the user’s behavior).
- No Tamagotchi: no death, hunger, distress, decay meters; neglect → ambient quietness only; drift never decreases on absence.
- No social network: no profiles, follows, discovery, comments, chat, avatars, leaderboards, co-presence, mutual-visit mechanics.
- No multi-aviary accounts, shared aviaries, payments, push/email about the aviary (visit-notify is off-by-default settings only), customizable scenes, public aviary directory.
- Personality vectors never shown as numbers—no debug panel in product builds.
- No recorded-audio fallback path.

### 1.3 Defensible calls where PRD is open

| Topic | Decision | Rationale |
|---|---|---|
| Tick period | 60s nominal; configurable 45–90s | Matches “~once per minute”; calibrable without schema change |
| Presence activity window | 180s without pointer/key ends presence | “Few minutes,” lean long so still watching counts |
| Presence ping cadence | Client emits presence heartbeat every 30s while triple-condition holds | Enough resolution for tick without event spam |
| Offer cooldown | 4 minutes per bird per offer-type | “Few minutes”; prevents curiosity saturation |
| Mood set (final) | `wary`, `content`, `curious`, `drowsy`, `alert`, `settled` | PRD examples + settled for night/settle continuity |
| Personality traits | float64 in `[0.0, 1.0]`; starters seeded ~0.35–0.55 with species bias | Hidden scalars; room to drift up only |
| Species pool | 6 species repositories (silhouette + palette + call motifs), incl. one night-active | Coherent local set; night not dead |
| Third+ bird unlock | Aviary age gates: day 45 → offer 3rd; day 120 → 4th; day 210 → 5th; day 330 → 6th; day 450 → 7th | Age-only, not engagement; gradual relationship rhythm |
| Snapshot pull | On open, on `visibilitychange`→visible, on >2s rAF gap, keepalive every 45s while visible | Covers focus return, sleep/wake, multi-device freshness |
| Notebook generation | Max ~2 prose entries/week baseline; burst on noteworthy events (first greet-order change, rare weather, long presence) | Sparse observer log |
| Auth link TTL | 15 minutes; single-use | PRD |
| Visit invite TTL | 30 days unused | PRD |
| Framework | TypeScript monorepo: React SPA + Node/TypeScript API + Postgres + Redis | One language, clear client/server split |
| Render | Canvas 2D primary scene; DOM for chrome/notebook/settings | Fit budgets; avoid WebGL complexity in v1 |
| Edge bootstrap | Signed anonymous snapshot cookie not required; HTML shell + `/api/v1/aviary/bootstrap` latency path from edge | TTFA <500ms |
| IANA timezone | Store user timezone on account (from browser at signup + settings); tick uses account TZ for local day/night | Stable multi-device day cycle |

---

## 2. Architecture

### 2.1 Service shape

```
┌─────────────────────┐     HTTPS/JSON      ┌──────────────────────────┐
│  Browser SPA        │◄──────────────────►│  API Gateway (HTTPS)     │
│  - scene renderer   │   snapshots + events │  - auth middleware       │
│  - WebAudio engine  │                     │  - rate limits           │
│  - presence sensor  │                     └───────────┬──────────────┘
│  - a11y surfaces    │                                 │
└─────────────────────┘                                 ▼
                                          ┌──────────────────────────┐
                                          │  Application services    │
                                          │  - AuthService           │
                                          │  - AviaryQueryService    │
                                          │  - EventIngestService    │
                                          │  - VisitService          │
                                          │  - AccountService        │
                                          │  - NotebookQueryService  │
                                          │  - ExportService         │
                                          └───────────┬──────────────┘
                                                      │
                    ┌─────────────────────────────────┼────────────────────────┐
                    ▼                                 ▼                        ▼
           ┌────────────────┐              ┌──────────────────┐      ┌─────────────────┐
           │ Postgres       │              │ Redis            │      │ Tick Worker     │
           │ (canonical)    │              │ - sessions       │      │ (per-shard cron │
           │ birds, vectors │              │ - rate limits    │      │  / queue jobs)  │
           │ moods, events  │              │ - tick locks     │      │ consumes events │
           │ notebook, etc. │              │ - snapshot cache │      │ writes state    │
           └────────────────┘              └──────────────────┘      └─────────────────┘
                    │                                │
                    │                         ┌──────▼──────┐
                    │                         │ Transactional│
                    │                         │ email (magic │
                    │                         │ links/invites)│
                    │                         └─────────────┘
                    ▼
           ┌────────────────┐
           │ Object storage │  (account export zip/json download links)
           └────────────────┘
```

**Hard architectural invariants**

1. **Server is sole writer of personality vectors and mood canonical state.** Clients never PATCH numeric traits.
2. **Clients write only append-only interaction events** (+ account/settings mutations that are not simulation state).
3. **Tick is the only path that applies drift deltas** and most mood transitions (session-local micro-reactions may be predicted client-side for latency but are corrected by next snapshot; authoritative mood is server).
4. **Synthetic account UUID** everywhere except encrypted email on membership row.
5. **Simulation DB is not joined into analytics warehouse.** Telemetry pipelines get operational metrics only.
6. **Visitors never write presence/interaction events** that feed host drift.

### 2.2 Client / server responsibility split

| Concern | Owner | Notes |
|---|---|---|
| Personality vector | Server | Stored columns; tick updates |
| Mood | Server | Tick + event-driven micro-updates on tick pass |
| Perch intents / positions | Server base pose + phase seeds; client interpolates | Snapshot carries zone, pose id, phase, seed |
| Call timing decisions | Server emits entropy seeds + vocal schedule hints; client realizes audio | Deterministic from seed+grammar for caption match |
| Idle micro-motion | Client from mood/personality params + seeds | Not simulated at 60fps server-side |
| Ambient leaves/feathers | Client-only ornaments | No per-leaf state |
| Presence qualification | Client sensors; server accepts presence events and enforces max rates | Server does not invent presence |
| Field notebook prose | Server generators writing notebook rows | Query via API |
| Narration prose | Prefer server templates from same observation engine as notebook; client may rephrase rate for a11y queue | Same voice rules |
| Call captions | Client from grammar motif params of the call just synthesized | Match audio |
| Day/night lighting | Client from account TZ + server “settled” flag + now | Consistent multi-device via TZ field |

### 2.3 Render pipeline boundary

```
StateSnapshot ──► SceneGraphBuilder ──► PoseResolver (mood/personality)
                         │
                         ▼
              MotionLayer (idle / transition / reduced-motion)
                         │
                         ▼
              AmbientLayer (weather, leaves, parallax)
                         │
                         ▼
              CanvasComposer ──► first bird paint path (critical)
                         │
              AudioEngine ◄── CallIntent stream (from snapshot + local listen-in)
                         │
              A11yNarration ◄── Observation prose stream
```

Server never sends frame-by-frame animation. Snapshots (~every 45–60s + event pull) describe **targets**; client runs the continuous feel-alive layer.

### 2.4 Monorepo layout (suggested)

```
/apps/web                 # SPA
/apps/api                 # HTTP API
/apps/tick-worker         # Simulation tick consumer
/packages/shared          # Types, event schemas, call-grammar shared constants
/packages/sim-core        # Drift, mood FSM, notebook triggers (pure TS, unit-tested)
/packages/call-grammar    # Motif libraries + caption templates (shared client/worker for tests)
/infra                    # Terraform/k8s, CI
```

Pure `sim-core` enables identity: same drift math in tests as production worker.

---

## 3. Data model

### 3.1 Core entities (Postgres)

**accounts**

| Column | Type | Notes |
|---|---|---|
| id | uuid PK | Synthetic; never email |
| email_ciphertext | bytea | Encrypted at rest |
| email_hmac | bytea unique | Lookup without storing plain email in indexes elsewhere |
| timezone | text | IANA |
| created_at | timestamptz | Aviary age base |
| marked_for_deletion_at | timestamptz null | Soft delete start |
| settings jsonb | | a11y defaults, visit_notify_opt_in=false, etc. |

**sessions**

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| account_id | uuid FK | |
| token_hash | bytea | |
| device_label | text | User-facing revoke list |
| created_at / last_seen_at / revoked_at | | |

**auth_magic_links**

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| account_id or email_hmac | | Create-on-first-login path |
| token_hash | bytea | |
| expires_at | | 15 min |
| consumed_at | | Immediate invalidate |

**aviaries** (1:1 account v1)

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| account_id | uuid unique | |
| bird_cap | int | default 7 |
| next_species_offer_at | timestamptz null | Age-gated adoption |
| settled_until | timestamptz null | Active settled state |
| simulation_version | bigint | Monotonic for optimistic clients |
| last_ticked_at | timestamptz | |

**birds**

| Column | Type | Notes |
|---|---|---|
| id | uuid | **Stable identity forever** |
| aviary_id | uuid | |
| species_id | text | From closed pool |
| display_name | text | User-assigned |
| sort_order | int | Adoption order |
| adopted_at | timestamptz | |
| # personality (never expose to UI as numbers)
| boldness | double | [0,1] |
| social_warmth | double | |
| vocal_frequency | double | |
| plumage_saturation | double | |
| curiosity | double | |
| personality_updated_at | timestamptz | |
| mood | text enum | |
| mood_updated_at | timestamptz | |
| perch_zone | enum front/middle/back | Intent |
| pose_id | text | Server illustrative |
| motion_seed | bigint | Client idle determinism |
| call_seed | bigint | Rotating entropy |
| cooldown_offers_until jsonb | | per offer type timestamps |

**interaction_events** (append-only)

| Column | Type | Notes |
|---|---|---|
| id | bigserial / uuid | Ordered tick consumption |
| aviary_id | uuid | |
| account_id | uuid | Actor; visitors use separate visit_events |
| bird_id | uuid null | When bird-specific |
| type | text | see §4 |
| payload jsonb | | durations, offer kinds, client timestamps |
| client_event_id | uuid | Idempotency |
| created_at | timestamptz | Server receive time |
| client_occurred_at | timestamptz | For ordering within skew budget |
| processed_at | timestamptz null | Tick watermark |

Indexes: `(aviary_id, created_at) where processed_at is null`; unique `(account_id, client_event_id)`.

**presence_segments** (optional materialization; can derive from events)

Used for drift math auditability: start/end presence intervals computed nerver-critical nightly, but tick can sum unprocessed `presence_ping` with merge rules.

**notebook_entries**

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| aviary_id | uuid | |
| observed_at | timestamptz | |
| prose | text | Naturalist lowercase |
| trigger_code | text | Internal; not user-facing |
| created_at | | |

**visit_invites**

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| host_account_id | uuid | |
| visitor_email_hmac | bytea | |
| visitor_email_ciphertext | bytea | For host log display |
| token_hash | bytea | One-time link |
| status | enum pending/accepted/revoked/expired | |
| expires_at | | +30d |
| accepted_at / revoked_at | | |

**visit_sessions**

| Column | Type | Notes |
|---|---|---|
| id | uuid | |
| invite_id | uuid | |
| started_at / ended_approx_at | | Duration estimate from last pull |
| No path writes host interaction_events |

**visit_log** (or query from sessions)

Host settings UI.

**species_definitions** (config table or code constant table)

silhouette key, default palette, motif library id, night_active flag, seed trait biases.

### 3.2 Event types (client → server)

| type | payload highlights |
|---|---|
| `presence_ping` | `{window_ok: true}` only when triple-condition; server records duration since last ping if continuous |
| `listen_in_start` / `listen_in_end` | `bird_id`, optional duration on end |
| `offer` | `offer_kind: seed\|song_fragment\|still_pool`, optional `near_bird_id` |
| `settle` | `{}` |
| `unsettle` | within undo window |
| `rename_bird` | actually account API not sim event—prefer REST mutation |
| `tab_hidden` / `session_end` | optional for presence closure |

### 3.3 Snapshot projection (API JSON sketch)

```json
{
  "simulation_version": 184422,
  "server_time": "ISO-8601",
  "timezone": "America/Los_Angeles",
  "day_phase": "morning|midday|evening|night",
  "weather": { "kind": "clear|rain|wind", "until": "..." },
  "settled": false,
  "birds": [
    {
      "id": "uuid",
      "species_id": "warbler_a",
      "name": "pip",
      "mood": "content",
      "perch_zone": "front",
      "pose_id": "preen_2",
      "motion_seed": 9912,
      "call": {
        "grammar_id": "warbler_a_v1",
        "vocal_frequency": "hidden-but-parametric",
        "next_call_hint_ms": 4200,
        "pitch_jitter_seed": 44
      },
      "visual": {
        "plumage_saturation_norm": 0.0,
        "saturation_band": "soft|rich"
      },
      "offer_cooldowns": { "seed": null, "song_fragment": "...", "still_pool": null }
    }
  ],
  "greeting": {
    "primary_bird_id": "uuid",
    "absence_bucket": "short|medium|long",
    "variant_seed": 17,
    "stagger_ms": [0, 380]
  },
  "ambient": { "leaf_intensity": 0.3 },
  "active_species_offer": null
}
```

**Critical:** snapshot may include **derived presentation bands** for plumage (not raw 0.62). Personality raw floats never leave API to product UI. Audio/visual parameters are allowed only as non-revealing continuous controls or coarse bands if needed; prefer parametric invariants that do not expose trait names.

**Engineering exception for audio engine:** the client needs continuous params that correlate with vocal frequency and boldness for motion. Expose as **anonymous simulation params** (`v_call_rate`, `v_approach`, …) not labeled trait names, and never in a user-readable stats surface. Do not ship a “debug stats” page in production.

### 3.4 Starter adoption

On account creation (first successful magic link (or reserved create)):

1. Create account UUID + aviary.
2. Draw 2 distinct species from pool (deterministic from uuid bits + rejection sampling).
3. Seed personalities with species bias + small noise.
4. Moods: morning–aligned content/curious mix.
5. Names not set until onboarding UI pass; temporary internal labels then user names.
6. Empty scene brief fly-in once birds accepted.

---

## 4. API surface

Base: `/api/v1`. Auth: `Authorization: Bearer <session>` except magic-link and visit token routes.

### 4.1 Auth

| Method | Path | Purpose |
|---|---|---|
| POST | `/auth/magic-link` | `{email}` → 202 always (anti-enumeration); send mail |
| GET | `/auth/magic-link/consume?token=` | Sets session cookie or returns token; invalidate link |
| POST | `/auth/sign-out` | Revoke current |
| GET | `/account/sessions` | List devices |
| DELETE | `/account/sessions/:id` | Revoke |
| POST | `/account/email-change` | Start verify flow |
| POST | `/account/email-change/confirm` | Commit |

Errors: matter-of-fact copy (`We couldn't sign you in...`).

### 4.2 Aviary state

| Method | Path | Purpose |
|---|---|---|
| GET | `/aviary/bootstrap` | Minimal snapshot + signed short cache; TTFA critical path |
| GET | `/aviary/snapshot` | Full snapshot; `If-None-Match: simulation_version` → 304 |
| POST | `/aviary/events` | Batch append events; idempotent by `client_event_id` |
| GET | `/aviary/notebook?cursor=` | Paginated entries newest-first, infinite history |
| POST | `/aviary/settle` | Convenience or via events |
| PATCH | `/birds/:id` | `{name}` only |
| GET | `/aviary/adoption-offer` | Age-gated next bird offer if due |
| POST | `/aviary/adoption-offer/accept` | Name + adopt when age allows |

**Event batch body**

```json
{
  "events": [
    {
      "client_event_id": "uuid",
      "type": "presence_ping",
      "client_occurred_at": "...",
      "payload": {}
    }
  ]
}
```

Response: `{ accepted: [...], simulation_version_hint }` — may trigger immediate tick for offer reactions in hot path optional; usually rely on next tick ≤60s + optimistic client reaction.

### 4.3 Account lifecycle

| Method | Path | Purpose |
|---|---|---|
| GET | `/account` | Settings + flags |
| PATCH | `/account/settings` | a11y, timezone, visit notify |
| POST | `/account/export` | Queue export; email link |
| POST | `/account/delete` | Soft delete |
| POST | `/account/delete/cancel` | Within 30 days |

### 4.4 Visits

| Method | Path | Purpose |
|---|---|---|
| POST | `/visits/invites` | Host: `{email}` |
| GET | `/visits/invites` | Outstanding + visit log |
| DELETE | `/visits/invites/:id` | Revoke immediate |
| GET | `/visits/view?token=` | Visitor bootstrap read-only snapshot (no event write routes) |
| GET | `/visits/snapshot` | Visitor session token scoped |

Visitor auth: parting token / short-lived visit JWT with `role=visitor`, `host_aviary_id`, **no** event ingest permissions.

Revoked: next snapshot → 410 with matter-of-fact body.

### 4.5 Health / internal

Ops-only tick metrics endpoints behind network ACL; not public product surface.

### 4.6 Rate limits

- Magic link: e.g. 5/hour/email_hmac.
- Events: burst 30/min, sustained designed for presence 30s cadence.
- Visit invite: small daily cap to reduce abuse.

---

## 5. Simulation engine design

Package: `packages/sim-core` used only by tick-worker (and unit tests).

### 5.1 Tick loop

For each aviary due (`last_ticked_at + period`):

1. Acquire Redis lock `tick:{aviary_id}` short TTL.
2. Load birds + aviary + unprocessed events ordered by `(created_at, id)`.
3. **Close presence**: merge presence_ping into continuous segments; clip by settle/session_end; discard pings that fail sanity (future timestamps).
4. **Apply fast session effects** from events since last tick:
   - Listen-in durations per bird.
   - Offers: resolve acceptance/approach using mood × curiosity; set cooldowns; mood nudges (curious→content on accept; wary waits).
   - Settle: set dusk lighting intent + quieter mood bias; end presence.
5. **Ambient world step**
   - Compute local solar proxies from timezone + `server_time`.
   - Possibly schedule rare weather (few/week poisson process seeded by aviary_id+day).
   - Weather short dampen vocal / alertness nudges.
6. **Bird-to-bird**
   - Alarm-like mood contagion: if one bird wary, neighbors soft-shift.
   - Chorus eligibility windows from vocal schedules.
7. **Mood FSM transition** (fast timescale)

```
inputs: personality traits, local time band, weather, recent offers/listen-in,
        neighbor mood, settle flag, previous mood

states: wary | content | curious | drowsy | alert | settled

rules (examples):
- night: majority → settled/drowsy; night_active species may stay alert/curious
- morning: +alert bias for high boldness
- rain: vocal activity down; content ↔ drowsy drift
- accepted offer recent: → content
- high boldness reduces transition probability into wary
- do not snap to neutral on “session start” (there is no session in tick)
```

Transition uses tempered softmax / stickiness so moods don’t thrash every minute.

8. **Personality drift** (slow timescale) — monotonic expressive>

```
For each trait t:
  delta = 0
  delta += k_presence * f(presence_minutes_in_window) * w_t
  delta += k_listen  * listen_minutes_on_this_bird * w_t_listen
  delta += k_offer_accept * accept_count * w_t_cur (curiosity)
  delta += k_offer_near   * near_count * w_t_bold (boldness)
  # settle does not add targeted trait deltas beyond closing presence

cumsum_week for instrumentation
t_new = min(1.0, t_old + low_pass(delta))
# NEVER: t_new < t_old for neglect; ambient quiet is mood/greeting rate, not negative drift
```

**Calibration targets**

- Instrument: after ~7 days regular (~30–60 min presence/week initially, calibrate load tests), mean |Δtrait| detectable above noise floor (>0.005).
- Human-visible: ~3 weeks for perch boldness / greeting order / plumage richness band changes.
- Single session: max trait delta bounded (e.g. ≤0.002) so clicking cannot Tamagotchi traits.

Store running filters (EMA park float on bird) to stabilize.

9. **Perch / pose intents** from mood + boldness for next snapshot.
10. **Greeting plan** if last host presence absence > threshold: choose greeter by boldness×warmth×mood; absence bucket short/medium/long; stagger seeds.
11. **Notebook triggers** (sparse): evaluate noteworthy predicates; write ≤N entries per week unless high saliency.
12. **Adoption offer**: if `now >= next_species_offer_at` and bird_count < 7 and no pending offer, set offer species.
13. Mark events `processed_at`; bump `simulation_version`; `last_ticked_at=now`; release lock.
14. Invalidate snapshot cache key.

### 5.2 Drift asymmetry (implementation backstop)

Unit tests assert:

- Zero presence over simulated 14 days → traits non-decreasing; greeting/call rates may fall via mood defaults and “ambient mode” flags optionally tracked separately as **expressiveness_recent** non-personality ratio derived from interaction recency—if needed without pillars violating monotonic personality.
- Preferred model for “quieter after neglect”: **recent_attention EMA** (not a personality trait) depresses greeting probability while boldness/plumage stay put. Personality only rises. This matches “quieter not mistrustful.”

Add `attention_ema` server field (can decrease with time) affecting greeting/call frequency rates but **not user-visible as a meter** and **not negative personality**.

### 5.3 Call-grammar runtime (authoritative design)

- Per species: motif atoms (short frequency envelopes, FM chirp params, gap rhythms).
- Runtime sequence: pick motif chain weighted by mood + anonymous vocal params + seeds.
- Identity: fixed species motif palette + bird-specific pitch center offset stored once at adoption (stable).
- Variation every call so no identical loop; seed refresh on call complete.
- Chorus: independent generators; mixer ducking when listen-in (client).

Client synthesizes; server does not stream PCM.

### 5.4 Autonomous offline continuity

Tick continues with empty event log: day/night mood, weather, idle bird-to-bird, attention_ema decay, pose/seeds refresh. User return → snapshot + greeting by absence bucket from `last_host_presence_at`.

---

## 6. Sync model

### 6.1 Single canonical aviary

- One DB row set per account.
- All devices: same `GET /snapshot`.
- No peer CRDT; no last-write-wins on vectors.

### 6.2 Conflict prevention

| Risk | Mitigation |
|---|---|
| Dual-device presence double count | Presence events attributed to sessions; tick **union-merge overlapping** presence intervals per aviary (not sum double) when concurrent; optional prefer max concurrent single occupancy |
| Duplicate events | `client_event_id` unique |
| Stale client optimistic mood | `simulation_version`; client rebases on snapshot |
| Magic-link replay | single consume |
| Mid-write timeout | events append transactional; tick reentrant with processed watermark |
| Personality overwrite | clients cannot write personality columns at all |

### 6.3 Concurrent multi-device sensation

If laptop and phone both open: both render same snapshot progression. Presence merge avoids double-speed drift. Listen-in from two devices: both logs; drift gains once per real-time minute of attention per bird via merge-on-interval for listen-in as well.

### 6.4 Visitor path

Separate token; read snapshots; **no** host presence contribution. Host visit log records visitor session length from visitor pull heartbeats that go to `visit_sessions` table only.

### 6.5 Soft delete / export

Export dumps birds (with numeric vectors allowed in private export file—user’s copy), notebook, settings. After hard delete, cascade all tables by `account_id` and shred email ciphertext.

---

## 7. Frontend rendering pipeline

### 7.1 Boot sequence (first frame)

1. Shell HTML + critical CSS (quiet field sky)—not a spinner.
2. Parallel: auth session check + `/aviary/bootstrap`.
3. As soon as bird vectors exist, **paint birds mid-pose immediately** using snapshot pose + phase offset `now % cycle`.
4. Start audio context on first user gesture (autoplay policies); until then, captions-ready optional ambient silence.
5. No entry animation of “aviary powering on.” Exception: true first adoption empty→fly-in once.

### 7.2 Scene composition

Layers back→front: sky gradient (time), background foliage, weather particles (light), perch silhouettes (back/mid/front zones), birds, rare nearest leaves, settle vignette.

Three logical zones map to y/scale depth; responsive width scales spacing; always all birds on-screen; never crop.

### 7.3 Idle micro-motion

State machine per bird: preen, scan, shuffle, head-tilt, rest—weights by mood. Seeds produce non-synced rhythms across birds. Never fully freezed if reduced-motion off.

### 7.4 Transitions

- Perch change: curved path ease ~1.2–2.5s (not teleport).
- Listen-in focus: slight compositional emphasis (pose/attention toward viewer)—no selection box chrome in scene; chrome focus ring for a11y only when keyboard-focus.
- Settle: palette warm→evening 3–5s, call mix down; 5s undo on any pointer/key in scene.

### 7.5 Reduced-motion mode

Trigger: `prefers-reduced-motion: reduce` OR settings toggle.

- Replace continuous animation with slow cross-fades between key stills (preen frames, perch A/B).
- Remove leaf drift.
- Keep day/night color drifts slowed.
- Keep audio + captions + narration + notebook + drift fully intact.

This is a first-class art pass, not `animation: none` on everything.

### 7.6 Top bar

Icons: account, a11y, notebook, offer; settle affordance included (PRD top-bar set + settle from interactions). Fade opacity after ~3s cursor idle; restore on movement/keyboard. No badges, no visit count flashes.

### 7.7 Offer UI

Top-bar opens small naturalist chooser (seed / song fragment / still pool)—not bird-click. Placement near scene; doesn’t stamp permanent chrome on birds.

### 7.8 Color

Calm naturalist palette; contrast for labels WCAG AA; scene itself mostly non-text.

---

## 8. Audio pipeline

### 8.1 Graph

```
CallSynth nodes (per bird) → BirdGain → ChorusBus → Master → destination
Listen-in controller automates BirdGain targets:
  focused: ramp up over ~1.5–3s
  others: ramp down to ambient floor (>0, never mute)
  leave: reverse ramp
```

### 8.2 Procedural synthesis

WebAudio: oscillators + noise buffers + bandpass + envelopes driven by motif IR-like param scripts in `call-grammar`. Pool and reuse AudioBuffers; **no per-call unbounded allocation**.

### 8.3 Identity + variation

- Stable bird pitch center.
- Motif selection + micro-timing jitter each call.
- Mood shapes tempo, gap, brightness.
- Vocal rate param shapes Poisson call schedule when unobserved + chorus join probability.

### 8.4 Listen-in semantics

Engage: click/tap/Enter on focused bird. Disengage: second activate, other bird, empty space, Escape, focus leave. Mix rebalance not hard cut.

### 8.5 Fallback

If AudioContext fails: force captions on for session; silent chorus addresses; no MP3 pack.

### 8.6 Song-fragment offer

Soft premade motif **also procedural** (small library of generative presets)—not a radio hit file. Birds respond by joining / quiet / against based on vocal + mood.

---

## 9. Accessibility surfaces

### 9.1 Screen-reader narration

- `aria-live="polite"` region with running naturalist prose.
- Cadence: 30–60s idle; faster on greeting, offer, settle.
- Content from observation service (shared patterns with notebook)—**not** trait dumps.
- Example tone matches PRD.

Priority queue: user events > ambient observations; drop stale ambient if queue backs up rather than flooding.

### 9.2 Captions

Settings opt-in; always-on in WebAudio failure. Short prose near bird; generated from **actual** motif params of that call instance.

### 9.3 Keyboard

Tab: top bar → enter birds. Arrows between birds. Enter listen-in. Esc exit. Offer + settle reachable without pointer. Focus ring high-contrast across day/night.

### 9.4 Contrast & settings

AA for all chrome/copy/captions. A11y settings use matter-of-fact voice. Product scene prose uses naturalist voice.

### 9.5 Shipping

A11y ships day-one with visuals/audio—not v1.1.

---

## 10. Performance budgets and observability

### 10.1 Budgets

| Metric | Budget |
|---|---|
| Initial JS gzipped | < 2 MB |
| Time to first bird visible (mid mobile 4G) | < 500 ms |
| Idle FPS (5-year mid laptop) | 60 fps sustained |
| Memory over 30 min session | no growth (CI leak test) |
| Snapshot size | low KB |
| Tick p99 | alarm > 5s |

### 10.2 Budget tactics

- Code-split settings, visits admin, export UI.
- SVG/procedural birds; limited bitmap atlas.
- No large audio asset packs.
- Canvas path thrifty; pause rAF hard when `document.hidden`.
- Worker optional for syntax-heavy audio param gen if main thread presses.

### 10.3 Observability (allow)

Synthetic geo browsers, RUM: TTFB, first-bird paint, long tasks, FPS histograms, audio context errors, API latency, tick latency.

### 10.4 Observability (forbid)

No per-bird traits, offers, presence minutes, notebook text in analytics. No warehouse replica of simulation DB. Error logs use account UUID not email.

### 10.5 Browser support

Last two major Chrome/Safari/Firefox/Edge; else matter-of-fact unsupported page.

---

## 11. Interaction implementation notes (product-critical)

### 11.1 Return-greeting

On bootstrap if host session: compute `absence = now - last_host_presence_at`.

| Bucket | Absence | Greeting intensity |
|---|---|---|
| short | < 30 min | glance / head-tilt |
| medium | < 36h | call + step forward possible |
| long | ≥ 36h | longer call, more re-orient |

Select primary greeter: max score `boldness * warmth_factor * mood_weight`. Secondary may stagger 200–800ms random—never unison welcome chorus on cue.

**No** welcome toast/banner/modal/"missed you"/days-gone copy.

### 11.2 Presence triple gate

Client PresenceSensor requires all of:

1. `document.visibilityState === 'visible'`
2. `document.hasFocus()`
3. pointermove or keydown within last 180s

Else stop heartbeats; emit session-end if needed. Settle ends window cleanly.

### 11.3 Offers

Cooldowns ~4 min/bird/type; server authoritative reject if early. Reactions animated client-side after event ack or predicted then corrected.

### 11.4 Field notebook

Read-only infinite scroll. Sparse generators:

- Fully avoid user-behavior moralizing (“you visited every day”).
- Prefer bird-centric observations and first-time-this-week greeter order etc.
- Prose templates curated; light grammar slots with bird names lowercase presentation rules.

### 11.5 Voice split

Naturalist: aviary chrome labels optional lowercase where brand allows, notebook, narration, captions, offer reconstructive microcopy.

Matter-of-fact: auth, errors, sync, account, a11y settings chrome, unsupported browser, visit revoked.

---

## 12. Social (visit) implementation

- Default off: zero invites.
- Host POST email → send one-time link.
- Visitor read-only SPA mode: hide offer/settle/listen-in write paths; still hear calls/render.
- No co-presence indicators.
- No host push unless `visit_notify_opt_in` (email degrades quiet; still no mobile push infrastructure required—if email notify, keep restrained).
- Visit log in settings without badge ornaments.
- Revoke → next pull 410 matter-of-fact.
- Age-out invites 30d.

---

## 13. Rollout

### 13.1 Build phases (engineering, still plan-only)

1. **Foundations**: account UUID auth, aviary schema, empty quiet field client shell, bootstrap API.
2. **Tick + two birds**: seed species, moods, snapshots, idle canvas, no audio polish yet.
3. **Presence + drift**: sensors, event ingest, monotonic drift tests, attention_ema.
4. **Audio + listen-in**: grammars for 6 species, mix, captions.
5. **Offers + settle + greeting**.
6. **Notebook + narration**.
7. **A11y reduced-motion art pass + keyboard**.
8. **Visits + export + deletion**.
9. **Perf hard gates** CI + synthetic RUM; bird load ramp.

### 13.2 Birds-per-aviary ramp

- Launch: 2 birds only; unlock schedule as age gates activate for early cohorts.
- Do not sell bird packs.
- Monitor call recognizability qualitative playtests at 5–7 before expanding pool (pool size 6 species; cap count 7).

### 13.3 Day-one instrumentation

- Operational SLOs listed §10.
- Drift calibration dashboard **internal-only** on synthetic birds / load accounts—not product users’ birds in warehouse.
- Feature flags: visit subsystem, weather intensity, notebook rate—for tuning without schema churn.

### 13.4 Privacy policy surface

Settings link plain text listing aggregates; exclude per-bird interaction use.

---

## 14. Risks and mitigations

| Risk | Failure mode | Mitigation |
|---|---|---|
| Drift too fast | Tamagotchi feel; session-visible trait gaming | Hard per-session caps; weeklong slow filters; QA harness measures |
| Drift too slow | Screensaver | Instrument weekly Δ; tune k without UI numbers |
| Soft presence definition | Background tab inflates population drift | Triple-gate + union merge; offline tests with laptop-open sim |
| Last-write personality | Silent history loss | Code owners forbid client writes; SQL grants; tests |
| Email as ID | PII blast radius | UUID-only invariant reviews + lint on logs |
| Canned audio temptation | Spell break | No asset pipeline for call loops CI-forbidden |
| Unison greeting | Announcement feel | Stagger + single primary |
| Toast “welcome” PR | Breaks product | Design checklist reject announcements |
| Reduced-motion as off switch | A11y users lose product | Dedicated cross-fade art; ship with v1 |
| Tick lag p99 | Aviary “stuck” | Shard workers, lock timeouts, alarm 5s |
| Chorus mud at 7 birds | Cap futility | Mix rules + recognizability playtest |
| Multi-device double presence | Accelerated drift | Interval merge |
| Memory leaks WebAudio | Tab diverge | Buffer pools; 30m CI |
| TTFA miss | Load feel | Edge bootstrap, quiet field, <2MB |
| Notebook becomes feed | Voice death | Rate limiter; human editorial templates |
| Visit feature creep | Social net gravity | Explicit non-goals; no discovery tables |
| attention_ema confused with punishment | Trust damage | Keep subtle; never distress art; quieter only |
| Autoplay audio blocked | Silent first minute | Gesture unlock; captions path |

### 14.1 Testing strategy (high level)

- **sim-core**: pure property tests on monotonic drift, concurrent event order, mood stickiness.
- **Contract tests**: clients cannot serialize personality write fields.
- **Presence**: headless scenarios with combinations of visibility/focus/activity.
- **Visual regression**: reduced-motion vs full; day/night palettes.
- **A11y**: axe on chrome; manual SR scripted sessions.
- **Perf**: bundle size CI gate; fps bench; leak test.
- **Privacy**: static analysis that analytics events schemas exclude bird fields.

---

## 15. Security & privacy engineering checklist

- Encrypt email at rest; HMAC for login lookup.
- Magic links single-use, 15m, rate-limited.
- Session revoke list.
- Visit tokens scoped read-only.
- Soft delete 30d then hard shred.
- Export only to verified email.
- Separate operational metrics store.
- No third-party ad pixels on aviary surface.

---

## 16. Open calibration backlog (build-time, not blockers for architecture)

1. Exact presence quiet seconds (start 180s).
2. Drift coefficients per trait for 1-week instrument / 3-week human targets.
3. Weather Poisson rate (“few times a week”).
4. Notebook saliency thresholds.
5. Offer cooldown exact minutes.
6. Age gates for birds 3–7 (starting proposal in §1.3).
7. Night species behavior intensity.

Each opens a **notation in sim config**, not a redesign.

---

## 17. Success criteria for v1 (qualitative + quantitative)

**Qualitative**

- First frame already live; bird notices without toast.
- User can leave two weeks and return to quieter, intact birds—no guilt UX.
- Pip identifiable by ear after two weeks.
- Notebook feel like field notes not logs.
- Reduced-motion and SR users report aliveness, not a status board.

**Quantitative**

- Budgets in §10 pass on release candidate.
- Drift harness hits week-1 measurable / week-3 visible targets on synthetic regular presence without single-session spikes.
- Tick p99 within alarm threshold under projected load.
- Zero paths writing personality from clients in penetration/code audit.

---

## 18. What this plan deliberately does not do

- Does not implement product code in this phase.
- Does not expand social into feeds or co-presence.
- Does not introduce payments or multiplayer simulation.
- Does not expose trait dashboards under any tier.
- Does not add engagement notifications about the aviary.

---

*End of plan. Ready for a separate engineering team to execute against PRD voice, architecture invariants, and calibration targets without further product-scope clarification.*
