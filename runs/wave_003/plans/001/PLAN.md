# Pocket Aviary — Phase 1 Implementation Plan

## 0. Executive summary

Pocket Aviary is a browser-only, single-aviary-per-account product. Two starter birds live in one horizontal scene; the user sits with them, listens in, offers small gestures, settles evenings, and reads sparse naturalist notebook entries. There is no game loop. Depth lives in slow personality drift driven by honest presence-time, fast mood, procedural calls, and a server-side simulation tick that advances whether or not anyone is watching.

This plan translates the PRD into an executable engineering program: service shape, data model, APIs, simulation engine, sync, render/audio pipelines, accessibility, performance budgets, rollout, and risks. Ambiguities are resolved with named decisions so a second team can build without further product clarification.

**Product one-liner for implementers:** build a place that continues without the viewer; measure attention, not clicks; never punish absence; never announce.

---

## 1. Scope

### 1.1 In scope for v1

| Area | v1 commitment |
|---|---|
| Platform | Modern web only (last two major versions of Chrome, Safari, Firefox, Edge) |
| Accounts | Single-user, email magic-link auth, synthetic account UUID, session list + revoke |
| Aviary model | One canonical aviary per account; 2 starter birds; hard cap 7 |
| Bird engine | Personality vector, monotonic-expressive drift, mood enum, call grammar, idle motion signals |
| Interactions | Return-greeting, presence accounting, listen-in, offer (seed / song fragment / still pool), settle (+ 5s undo), field notebook (read-only) |
| Layout | Single-screen horizontal scene; three perch zones; local day/night; rare ambient weather; top-bar chrome that fades |
| Sync | Server-only personality writes; client snapshot pull + event append; multi-device via shared canonical record |
| Social | Opt-in visit invites by email; read-only ambient; visit log; revoke; default off |
| Accessibility | Naturalist SR narration, designed reduced-motion, call captions, WCAG AA chrome, full keyboard nav |
| Performance | Initial gzipped JS ≤ 2MB; first bird ≤ 500ms on mid-tier 4G; 60fps idle on 5-year laptop; no mem growth over 30 min |
| Ops | Export JSON snapshot; soft-delete 30d then hard; aggregate-only telemetry |

### 1.2 Explicit non-goals (do not build)

- Native iOS/Android apps
- Any gamification: streaks, scores, badges, levels, XP, green-dot calendars, “birds adopted” counters, milestone celebrations
- Tamagotchi mechanics: death, hunger, distress, decay-on-neglect happiness meters
- Social network surfaces: profiles, follows, public discovery, feeds, comments, co-presence, leaderboards, avatar visitors
- Multi-aviary accounts, shared aviaries, customizable scenes, payments, push/email re-engagement about the aviary
- Numerical personality exposure in any UI tier/debug product surface
- Recorded-audio fallback path for calls
- “Welcome back”, absence-duration banners, visit-frequency surfaces, friend-visited push by default

### 1.3 Defensible decisions on PRD ambiguity

| Ambiguity | Decision |
|---|---|
| Exact personality ranges | Each trait `0.0–1.0` float32; starters seeded in `0.35–0.55` with small species bias |
| Mood enum | Fixed set: `wary`, `content`, `curious`, `drowsy`, `alert`, `settled` (settled used at night / post-settle) |
| Tick cadence | 60s target; allow 45–90s adaptive under load; design all rates against 60s |
| Presence activity window | 180s without pointer/key ends presence (lean long; watching is stillness) |
| Offer cooldown | 4 minutes per (bird, offer-kind) |
| Species pool size | Exactly 6 species at launch |
| Third+ bird unlock | Time-since-account-creation gates only: bird 3 at day 21, bird 4 at day 60, bird 5 at day 120, bird 6 at day 210, bird 7 at day 300 |
| Notebook sparsity | Avg ≤ 1 entry / 3 active user-days; hard cap 1/ day; plus rare noteworthy-event entries |
| Snapshot format | Versioned JSON over HTTPS; optional later binary without protocol break |
| Prose generation | Template + constrained slot-filler (not free-form LLM) for notebook/narration/captions — shippable, on-brand, offline-stable |

---

## 2. Architecture

### 2.1 Service shape

```
┌─────────────────────────────────────────────────────────────────┐
│                        Clients (browser SPA)                    │
│  Render (canvas/SVG) │ Audio (WebAudio) │ Presence │ A11y narr  │
└───────────────┬───────────────────────────────┬─────────────────┘
                │ HTTPS JSON + long-poll/SSE    │ events POST
┌───────────────▼───────────────────────────────▼─────────────────┐
│                         Edge / API gateway                        │
│  auth middleware │ rate limits │ CDN static + bootstrap snapshot  │
└───────┬──────────────────┬───────────────────┬──────────────────┘
        │                  │                   │
┌───────▼──────┐  ┌────────▼────────┐  ┌───────▼────────────┐
│ Auth service │  │ Aviary API      │  │ Simulation worker  │
│ magic links  │  │ snapshots,      │  │ ~60s tick per      │
│ sessions     │  │ events, visits, │  │ active aviary shard │
│ account CRUD │  │ notebook, export│  │ consumes event log │
└───────┬──────┘  └────────┬────────┘  └───────┬────────────┘
        │                  │                   │
        └──────────────────▼───────────────────┘
                    Canonical store
         accounts | birds | vectors | moods | events
         notebook | visits | sessions | invites
```

**Monolith-first posture:** one deployable backend (`aviary-api`) with internal modules (auth, state, sim, visits, prose). Split workers when tick fan-out requires it. Do not invent a microservices mesh for v1.

### 2.2 Client/server split (hard rules)

| Responsibility | Owner | Rule |
|---|---|---|
| Personality vector | Server only | Clients never send absolute trait values |
| Mood | Server authoritative; client may locally ease animation toward next snapshot mood | Never invent permanent mood |
| Presence, offers, listen-in, settle | Client detects/emits events | Append-only event log |
| Positions / idle phase | Server publishes logical pose targets + phase seeds; client interpolates and ornaments | Server is continuity; client is smoothness |
| Procedural call audio | Client synthesizes from grammar params in snapshot | Server may include call seed/timing hints |
| Leaf/feather drift | Client-only ornaments | No sim state |
| Notebook/narration/captions text | Server-generated prose artifacts preferred for notebook; client may compose captions from call params | Same voice rules |

### 2.3 Render pipeline boundary

Three client layers:

1. **State layer** — snapshot store, event outbox, presence monitor, focus/listen-in UI state.
2. **Simulation presentation layer** — bird controllers map inventorial state → pose targets, mood-shaped idle chooser, perch zone placement, greeting choreography. No writing personality.
3. **Render/audio layer** — scene graph (sky, foliage BG, perch mid, birds, FG ornaments), WebAudio graph, captions, reduced-motion cross-fades, focus rings.

Boundary rule: presentation may *predict* motion for the next few hundred ms after a local interaction (e.g., offer response anticipation marker hailed by server ack), but all durable state rebounds to next snapshot.

### 2.4 Technology recommendations (defaults)

- **Client:** TypeScript, Vite, lightweight state (Zustand or similar), canvas 2D or lightweight WebGL for birds + SVG assets for silhouettes; RUM via PerformanceObserver.
- **Backend:** TypeScript (Node 22) or Go — pick one stack for the monorepo team; prefer TS shared types with client if small team.
- **Store:** Postgres primary (accounts, birds, vectors, moods, notebook, invites, sessions); Redis optional for session cache + rate limits + tick leases.
- **Event log:** Postgres append-only `interaction_events` partitioned by `account_id` + time; tick cursor per aviary.
- **Jobs:** in-process tick scheduler initially; move to leased worker pool when accounts exceed process soft limit.
- **Email:** transactional provider for magic links + visit invites + export links only.
- **CDN:** static SPA + edge-cached public assets; authenticated snapshot never on public CDN without signed short-TTL URL if used for bootstrap.

### 2.5 Repo layout (suggested)

```
/apps/web                 # SPA
/apps/api                 # HTTP API + tick worker entry
/packages/shared          # types, event schemas, call grammar param codecs, prose templates
/packages/sim             # pure simulation functions (unit-test dense)
/packages/audio-grammar   # motif tables + caption templates (isomorphic)
/infra                    # deploy, migrations, synthetic perf runners
```

---

## 3. Data model

### 3.1 Core entities

#### accounts
| Field | Type | Notes |
|---|---|---|
| id | UUID | Synthetic; only external id |
| email_ciphertext | bytea | PII; single storage site |
| email_hash | bytea | for lookup; keyed hash |
| created_at | timestamptz | used for bird unlock age gates |
| timezone | text | IANA; client-reported, server-validated |
| settings_json | jsonb | a11y prefs, visit-notify opt-in, caption default |
| deleted_at | timestamptz null | soft delete start |
| hard_delete_after | timestamptz null | deleted_at + 30d |

#### sessions
| Field | Type | Notes |
|---|---|---|
| id | UUID | |
| account_id | UUID | |
| device_label | text | coarse UA-derived |
| created_at / last_seen_at | timestamptz | |
| revoked_at | timestamptz null | |

#### aviaries
| Field | Type | Notes |
|---|---|---|
| id | UUID | 1:1 with account in v1 |
| account_id | UUID unique | |
| established_at | timestamptz | = account create |
| lighting_state | enum | `day_cycle` \| `settled_override` |
| weather_state | jsonb | optional active weather + expires_at |
| sim_cursor_event_id | bigint | last consumed event |
| sim_version | int | optimistic concurrency for snapshot generation |
| last_ticked_at | timestamptz | |

#### birds
| Field | Type | Notes |
|---|---|---|
| id | UUID | stable identity forever |
| aviary_id | UUID | |
| species_id | text | one of 6 |
| display_name | text | user-assigned |
| adopted_at | timestamptz | |
| sort_index | smallint | adoption order |
| boldness…curiosity | float | 5 traits 0–1 (or `traits jsonb`) |
| mood | enum | |
| mood_updated_at | timestamptz | |
| perch_zone | enum | `front` \| `middle` \| `back` |
| pose_phase | float | seed for continuous motion |
| call_seed | bigint | signature identity within species grammar |
| last_offer_at_by_kind | jsonb | cooldown tracking |

#### interaction_events (append-only)
| Field | Type | Notes |
|---|---|---|
| id | bigserial | global order within shard |
| account_id | UUID | never email |
| aviary_id | UUID | |
| bird_id | UUID null | if bird-specific |
| type | enum | see 3.2 |
| payload | jsonb | typed per event |
| client_ts | timestamptz | advisory |
| server_ts | timestamptz | authoritative order |
| device_session_id | UUID | |
| visit_context | bool | true if visitor → **must not affect drift** |

#### notebook_entries
| Field | Type | Notes |
|---|---|---|
| id | UUID | |
| aviary_id | UUID | |
| created_at | timestamptz | |
| prose | text | naturalist lowercase |
| trigger | text | internal reason code; never shown |
| salience | smallint | for sparsity budget |

#### visit_invites
| Field | Type | Notes |
|---|---|---|
| id | UUID | |
| host_account_id | UUID | |
| visitor_email_hash / ciphertext | | PII confined |
| token_hash | bytea | one-time / session token |
| created_at / expires_at | | unused expire 30d |
| revoked_at | null | |
| accepted_at | null | first use |
| last_visit_at | null | |

#### visit_sessions / visit_log
Immutable log rows: invite_id, started_at, ended_at, approx duration. No badges.

#### magic_link_tokens
token_hash, account_id or email pending, expires_at (15m), consumed_at.

### 3.2 Interaction event types

```
presence_ping          { dt_ms, visibility, focused, activity_fresh }
listen_in_start        { bird_id }
listen_in_end          { bird_id, duration_ms }
offer                  { bird_id_hint?, kind: seed|song|pool, song_id? }
offer_resolved         { bird_id, kind, reaction }  // server-emitted to log optional
settle_start           {}
settle_undo            {}
session_open           { absence_ms, viewport... }   // for greeting selection context
session_close          { reason: tab|settle|logout }
rename_bird            { bird_id, name }             // settings path may bypass if direct write with audit
adopt_bird             { species assigned server-side }
```

**Invariant:** events describing visitor activity are stored for audit if desired but **filtered out** of drift inputs entirely.

### 3.3 Personality vector

Traits: `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`.

- Stored as float32 0.0–1.0.
- Updated only by sim tick non-negative deltas (clamped at 1.0).
- Never recomputed from full history; never client-derived; export may include current values (user’s dump), product UI never displays numbers.

### 3.4 Mood

Enum transitions via priority fan-in on tick and on interaction consumption:

```
recent offer accept → content/curious
dusk local time → drowsy
night → settled (most species); nightjar-like species → alert/content night calls
rain weather → lower calling propensity (mood nudge drowsy/wary)
alarm call neighborhood → wary canvas
high boldness reduces transition weight into wary
```

Mood persists across sessions; tick evolves it while offline.

### 3.5 Species pool (v1)

Six species, coherent visual/acoustic set. Each defines:

- silhouette asset key
- base plumage palette (± saturation modulation by trait)
- call grammar module id + motif banks
- idle motion biases
- `nocturnal_active` flag (exactly one night-active species)

No rarity. New birds: uniform or slight complementarity selection among species not yet in aviary when possible.

### 3.6 Indexes and privacy partitioning

- All FKs and partition keys use UUIDs, never email.
- Telemetry tables live in a separate schema/DB user with **no SELECT** on birds/events.
- PII columns encrypted at rest; access audited.

---

## 4. API surface

Base: `https://api.<host>/v1`. Auth: `Authorization: Bearer <session_token>` unless noted. All responses exclude email except account settings bare minimum.

### 4.1 Auth

| Method | Path | Purpose |
|---|---|---|
| POST | `/auth/magic-link` | body `{email}` → 202 always (no enumerate); send link |
| GET | `/auth/consume?token=` | consume magic link → set session cookie/token; 15m/single-use |
| POST | `/auth/logout` | revoke current session |
| GET | `/me` | account settings safe view |
| PATCH | `/me` | timezone, a11y prefs, visit notify toggle, email change start |
| POST | `/me/email/verify` | complete email change |
| GET | `/me/sessions` | list devices |
| DELETE | `/me/sessions/:id` | revoke |
| POST | `/me/export` | enqueue JSON export; email download link |
| POST | `/me/delete` | start soft delete |
| POST | `/me/delete/cancel` | restore within 30d |

System voice errors only.

### 4.2 Aviary state

| Method | Path | Purpose |
|---|---|---|
| GET | `/aviary` | canonical snapshot (servers as source of truth) |
| GET | `/aviary/stream` | optional SSE/long-poll: snapshot version bumps (~on tick or relevant event) |
| POST | `/aviary/events` | batch append interaction events; idempotency-key header |
| POST | `/aviary/bootstrap` | first-time adopt starter pair naming (if empty) |
| PATCH | `/birds/:id` | rename only |
| POST | `/aviary/adopt` | unlock next bird when age gate open; server picks species |

#### Snapshot schema (conceptual)

```json
{
  "sim_version": 18402,
  "server_time": "...",
  "local_phase": { "solar": "morning", "lighting": "day_cycle|settled_override", "weather": null },
  "greeting": {
    "due": true,
    "primary_bird_id": "...",
    "absence_ms": 7200000,
    "style": "glance|call|approach|reorient",
    "stagger_ms": [0, 420, 900]
  },
  "birds": [
    {
      "id": "...",
      "name": "pip",
      "species_id": "warbler_a",
      "mood": "content",
      "perch_zone": "front",
      "pose": { "action": "preen", "phase": 0.42 },
      "plumage": { "sat": 0.51 },
      "call": { "grammar_id": "...", "seed": 99, "vocal_rate": 0.48 },
      "cooldowns": { "seed_ms": 0, "song_ms": 0, "pool_ms": 0 }
    }
  ],
  "chorus_hints": { "window_ms": 8000 },
  "unlock": { "next_bird_at": "ISO|null", "birds": 2, "cap": 7 }
}
```

**Important:** snapshot includes *derived rendering knobs* (pose action, perch) and call seeds,—not raw “optimize me” dashboard. No trait numbers in product SPA stores for display (may keep opaque if needed for call timing only via already-encoded `vocal_rate`).

### 4.3 Field notebook

| Method | Path | Purpose |
|---|---|---|
| GET | `/notebook?cursor=` | reverse chrono entries; infinite scroll |
| (no write APIs for users) | | |

### 4.4 Visits

| Method | Path | Purpose |
|---|---|---|
| POST | `/visits/invites` | host: `{email}` create invite; email one-time link |
| GET | `/visits/invites` | host outstanding + history |
| DELETE | `/visits/invites/:id` | revoke immediately |
| GET | `/visits/log` | host visit log |
| GET | `/visit/:token` | visitor: obtain short-lived visitor session |
| GET | `/visit/session/aviary` | visitor snapshot: same scene fields; `interactions: false` |
| POST | `/visit/session/heartbeat` | duration accounting only; **no drift events** |

Visitor snapshot deliberately omits host settings/PII. On revoke: next pull returns matter-of-fact `{ "error": "visit_unavailable" }`.

### 4.5 Interaction submission semantics

- Client batches events every 2–5s while present, and flushes on listen-in edges, offer, settle, visibility hidden, `pagehide`.
- Server validates: auth, bird belongs to aviary, rate limits, visit_context forbids drift types.
- Idempotency: client UUIDs on events; unique (account_id, client_event_id).
- Server never applies personality in the POST handler—only appends; tick applies.

### 4.6 Offer resolution API path

On `offer` event:

1. API records event.
2. Optionally runs **fast path mini-resolve** to return 200 body with chosen bird + reaction for low-latency animation (does **not** write personality; may write a scheduled mood delta into a side queue the tick also honors, or stamps `pending_reaction` on bird consumed by tick and next snapshot).
3. Preferred: return provisional reaction from pure function shared with sim package; tick is source of truth for durable mood/drift and may reconverge.

### 4.7 Rate limits (approx)

- magic-link request: 5 / email / hour
- events POST: 120 / min / session
- invites: 20 / day / account
- export: 3 / day

---

## 5. Simulation engine design

Package: `packages/sim` pure functions + worker orchestration.

### 5.1 Tick loop

Every ~60s per aviary (or on wake if behind):

1. **Lease** aviary row (`FOR UPDATE SKIP LOCKED` or Redis lock).
2. **Load** birds, weather, lighting, sim_cursor, account timezone.
3. **Read** new events where `id > sim_cursor` ordered by `server_ts, id`.
4. **Filter** visitor and invalid events.
5. **Aggregate** presence windows; listen-in durations; offers; settle markers; session_open absences.
6. **Advance environment:** solar phase from timezone; weather RNG (rare); settled_override expiry on re-engage tokens.
7. **Mood step** per bird (see 5.3).
8. **Drift step** per bird (see 5.2).
9. **Perch/pose intent** selection (5.4).
10. **Bird–bird coupling** (5.5).
11. **Notebook candidate** (5.6).
12. **Write** new state, bump `sim_version`, advance cursor, `last_ticked_at`.
13. **Release** lease; optionally publish SSE version bump.

Catch-up: if server was down, run multi-minute fast-forward in chunks of 60s virtual time using stored events + time-of-day only (no invented presence).

### 5.2 Drift function

Design goals:

- Measurable instrument drift after ~7 days regular presence
- User-visible characterized change after ~21 days
- Single session never loses visible “I grinded boldness”
- **Monotonic toward expressive:** δ ≥ 0 always; neglect does not decrease traits

#### Presence effective minutes

```
presence_minutes = sum(clamped continuous intervals where presence_ping validity holds)
```

Validity requires all of: `visibilityState==visible`, window focus true, activity within 180s.

#### Per-tick deltas (illustrative calibration starting point)

Let `P` = presence minutes in tick window (usually 0–1).  
Let `L_b` = minutes listened-in on bird b.  
Let `O_c`, `O_b`, `O_accept` = offer near / accept signals.

```
δ_social_warmth     += k_sw * (0.6*P + 1.4*L_b)
δ_vocal_frequency   += k_vf * (0.5*P + 1.2*L_b)
δ_boldness          += k_bo * (0.4*P + 0.8*offer_near)
δ_curiosity         += k_cu * (0.3*P + 1.0*offer_accept)
δ_plumage_sat       += k_pl * (0.5*P + 0.3*L_b)
```

Scale constants so that ~30 min presence/day × 7 days moves a trait by ~0.01–0.02 (instrument), ~0.04–0.06 over 21 days (felt). Put constants in config; CI golden tests lock “week of synthetic presence → trait delta band.”

**Ambient quietness without negative drift:** greeting probability and unsolicited call rate use an *attention recency* envelope separate from traits:

```
attention_recency = exp(-hours_since_meaningful_presence / τ)  // τ ≈ 72h
call_rate_effective = f(vocal_frequency, mood, attention_recency, weather)
greet_weight = f(boldness, social_warmth, attention_recency, absence_ms)
```

Neglect → quieter greetings via recency envelope, **not** trait decay.

Settle events: end presence cleanly; small mood quieting only; zero trait delta special-case beyond loss of presence.

### 5.3 Mood transitions

Discrete-time Markov-like with external forces:

```
base = personality-conditioned stationary bias
+ time_of_day_vector(local hour)
+ weather_vector
+ interaction_impulse (decays over ~30–90 wall minutes)
+ neighbor_wary contagion if another bird wary recently
```

Softmax or weighted pick once per tick, with hysteresis so moods don’t thrash every minute. Map moods to motion profiles client-side.

After long offline: apply multi-hour TOD path so morning return isn’t yesterday’s dusk frozen incorrectly—mood evolves along TOD while offline **without** fabricating user presence impulses.

### 5.4 Call-grammar runtime (logical)

Server/client share grammar ids:

- Motif banks per species (pitch envelopes, rhythm cells, ornaments)
- Runtime combined by seed + mood modifiers (wary: shorter, softer; alert: sharper attacks; drowsy: longer gaps)
- Vocal frequency trait → Poisson-like free call intensity when not listen-in focused
- Chorus: if ≥2 birds scheduled call windows overlap within N ms and warmth high, second bird bias to antiphonal response motifs

Server snapshot provides: rate params, next-window tips optional; **audio samples never leave client**.

### 5.5 Bird-to-bird

- Response call bias when neighbor calls (mood + social_warmth)
- Wary contagion limited radius (all birds in small aviary = full set)
- Physical spacing: two high-boldness birds both front-pref → soft conflict resolution stagger middle

### 5.6 Idle motion & perch selection (server intents)

Per tick assign:

- `perch_zone` from boldness/mood (wary→back, bold+content→front)
- `pose.action` ∈ preen | scan | fluffed_rest | head_tilt | weight_shift | sleep_light
- phase continues continuously (phase += dt * speed(mood))

Client ornaments micro-variation so birds never look paused when visible.

### 5.7 Return-greeting planner

On `session_open` (or first snapshot after absence > 5s):

1. Rank birds by greet_weight (boldness, warmth, mood, species).
2. Choose primary greeter (stochastic among top).
3. Map `absence_ms` → style: short <10m glance; medium call; long reorient/approach.
4. If secondary greeters eligible, assign stagger 200–1200ms random, never unison on cue.
5. Emit greeting block once; client plays choreography; figure is not a toast.

### 5.8 Offers

Kinds:

- **seed** — approach/peck/ignore by curiosity+mood; boldness affects latency
- **song fragment** — pick from small library (~8); bird join/quiet/counter by vocal_frequency+mood
- **still pool** — drink/bathe/watch decorations in front plane; mood-shaped

Cooldown 4 min per bird per kind. Accept → curiosity micro-impulse; offer placement near bird → boldness micro-impulse (applied on tick).

### 5.9 Adoption

- Account create → two birds assigned complementary species; user names in bootstrap UI; soft fly-in once only.
- Later birds: when `now >= unlock_at(n)`, `POST /aviary/adopt` allowed; server selects species; fly-in once.

### 5.10 Notebook generation

Candidate triggers (examples): first-greeter-of-day swap, long collectively quiet window, weather passage, new bird, unusually long listen-in, first pool bathe.

Scoring:

- must exceed salience threshold
- respect sparsity budget
- prose from templates with filled slots (bird names, zone, weather word, weekday)

Examples (output style, not system message):

> tuesday — pip greeted before wren today, first time this week.

Never log user streaks or “you visited.”

### 5.11 Identity continuity

Migrations may reshape species asset packs but **must never** reissue bird UUIDs or reset vectors. Idempotent backfills only. Soft-deleted accounts retain rows until hard delete, then cascade destroy.

---

## 6. Sync model

### 6.1 Canonical prop

One aviary row family on server. Devices A and B are dual readers + dual event writers. Personality merges are unnecessary because absolute traits are never dual-writer.

### 6.2 Client lifecycle

1. Auth session.
2. Fetch `/aviary` snapshot; if multi-second delay, show **quiet field** (not spinner).
3. Start render mid-action using poses/phases from snapshot (seed RNGs so ornaments differ but birds are mid-motion).
4. Start presence monitor + audio (after user gesture if required by browser).
5. Subscribe SSE or poll every 30–45s while visible; **immediate** refetch on `visibilitychange→visible` and on long rAF gap (resume from sleep).
6. Interpolate birds toward new perch/pose on snapshot apply (300–1200ms ease; reduced-motion: cross-fade).
7. Flush events aggressively on hide.

### 6.3 Conflict prevention

| Risk | Mitigation |
|---|---|
| LWW wipe of traits | Forbidden path: no API accepts trait absolutes |
| Dual offline personality branching | Impossible: traits only online via server tick |
| Duplicate events | client_event_id uniqueness |
| Magic link replay | single consume + short TTL |
| Mid-write timeout | events idempotent; snapshots versioned |
| Visitor affecting host | visit_context filter + separate endpoints |
| Clock skew | server_ts orders events; client_ts cosmetic |

### 6.4 Settled lighting override

Client settle → event → snapshot lighting `settled_override`. Any subsequent host non-visitor interaction (click) within 5s may send `settle_undo`. Tab close without settle ≡ presence end, no penalty, no nag.

### 6.5 Multi-device interaction overlap

Two devices present simultaneously:

- Both may emit presence; tick sums presence carefully (cap concurrent presence to 1× wall clock to avoid gaming via two devices left open).
- **Decision:** presence minutes per aviary capped by wall-clock elapsed in tick (`min(sum_device_presence, tick_wall_minutes)`). Protects calibration.
- Listen-in/offers from either device apply normally as host events.

### 6.6 Error surfaces (system voice)

- Sign-in expired link
- Session timeout
- Aviary load failure
- Visit unavailable

No naturalist cosplay for failures.

---

## 7. Frontend rendering pipeline

### 7.1 Scene composition

Layers back→front:

1. Sky gradient (TOD + weather tint)
2. Background foliage (subtle parallax factor 0.1)
3. Back perch zone
4. Middle perch zone + birds sorted by zone/depth
5. Front perch + offer props (pool, seed)
6. Foreground branch/leaf ornaments (parallax 0.3)
7. Captions / focus ring / reduced-motion stills
8. DOM top bar (outside canvas) sparse icons: settings, a11y, notebook, offer; settle control here too

No chrome inside scene proper: no badges, tooltips on birds, inline labels.

### 7.2 Layout rules

- Single viewport fit; no pan/zoom/scroll of scene.
- Responsive: maintain all birds on-screen; compress inter-perch spacing on narrow widths; widen on desktop.
- Three zones remains readable at phone widths.

### 7.3 First paint / mid-action

- Critical path: HTML shell + crit CSS + tiny boot JS + inline or HTTP/2 push of last-known cached snapshot if any + SVG silhouettes compact.
- Birds placed with randomized phase offsets drawn from server phase.
- Loading without snapshot: quiet field sky only.
- Empty pre-adopt exception: quiet field → soft fly-in after bootstrap once.

### 7.4 Idle micro-motion

Continuous loops mood-shaped; never fully still if emotive motion allowed. When tab hidden: cancel rAF, suspend audio optional, stop ornaments—**do not** claim sim stopped.

### 7.5 Transitions

- Perch moves: eased path solids; reduced-motion: opacity cross-fade between poses at zones.
- Day/night: continuous palette shift, not cut.
- Settle: 2–4s evening grade + call gain down.
- Listen-in visual: optional soft vignette sparsity — **decision:** prefer **no retain chromatic chrome**; rely on audio + subtle feather highlights on focused bird under a11yInnocent contrast. Keep “notice not announce.”

### 7.6 Top bar fade

After ~3s pointer/keyboard idle, opacity → ~0.15; activity restores 1.0 over ~200ms. Always keyboard/focus-accessible even when faded (full opacity on focus-within).

### 7.7 Color system

Calm naturalist palette; no electric accents. Text-on-chrome AA. Scene contains almost no text except captions optionally.

### 7.8 Asset strategy

- Prefer procedural/silhouette shade + limited feather detail bitmaps atlas.
- Plumage saturation trait modulates color matrix, not unique textures per trait step.
- Code-split settings, notebook panel, visit admin.

---

## 8. Audio pipeline

### 8.1 Graph

```
[CallSynth voice nodes per bird] → [per-bird gain] → [listen-in bus]
                                        ↘
                                   [ambient bus] → master gain → destination
[song offer player] → send to atmosphere
[soft environment bed optional very low] → master (subtle; not music bed competing)
```

### 8.2 Procedural synthesis

- WebAudio oscillators + noise buffers + formant filters + amplitude envelopes driven by motif graphs.
- Motif graph picks non-deterministically each call with seed stream per bird.
- Recognizability: fixed species motif family + per-bird seed bias (interval preferences, tempo center).
- Never loop a PCM chirp sample as the primary design.

### 8.3 Listen-in mix

- Engage / disengage ramps **1.5–3.0s** logarithmic.
- Focus bird gain → strong; others → low ambient floor **≠ 0**.
- Disengage on second activate same bird, other bird focus, empty-scene click, Escape / focus leave.

### 8.4 Chorus mixing

Scheduler avoids exact phase-locked duplicates; small humanizing jitter. Cap simultaneous voices intelligibility ≤7 birds.

### 8.5 Captions

From motif descriptor tokens at synth time:

> a soft three-note rise

Rendered near bird, fade with call. Same naturalist voice.

### 8.6 Autoplay / permissions

If AudioContext blocked: run silent with **captions default on** until unlock gesture; then fade captions only if user preference was off.

### 8.7 WebAudio unavailable

Graceful silence + captions on. **No recorded-audio pack.**

### 8.8 Memory

Reuse buffer sources pools; disconnect nodes; CI heap diff over 30 min call stress.

### 8.9 Mute

User mute is allowed (browser or settings). Muted still generates presence if watching; optional note: muting slightly reduces vocal-frequency drift weight? **Decision:** **no** — muting is accessibility/environment, not neglect. Presence remains valid.

---

## 9. Accessibility surfaces

Accessibility ships **with** v1, not as lagging parity.

### 9.1 Screen-reader narration

- Live region (`aria-live=polite`) fed by slow naturalist paragraphs every 30–60s idle.
- Priority flush on greeting, offer reaction, settle.
- Content from same snapshot facts as visuals; **not** “mood: content” telegrams.
- Avoid flooding: collapse multiple micro-events.

### 9.2 Reduced motion

Detect `prefers-reduced-motion` + settings override.

- Replace continuous skeletons with still poses eases.
- Perch travel = cross-fade.
- Kill leaf drift.
- Keep palette TOD slow shifts, audio, drift, notebook.

### 9.3 Captions

Settings opt-in; default on when audio fails.

### 9.4 Keyboard

- Tab: top bar controls
- Enter aviary region → first bird
- Arrows between birds
- Enter: listen-in toggle
- Escape: exit listen-in / close panels
- Offer + settle fully keyboard operable
- Visible focus ring designed for bright and night scenes

### 9.5 Contrast & semantics

AA for all chrome and captions. Multimodal: SR narration + captions + visuals mutually reinforcing, not clipped.

### 9.6 Visit mode a11y

Visitor gets same narration dominate促read-only (no offer controls exposed).

---

## 10. Performance budgets and observability

### 10.1 Budgets (gate CI / release)

| Metric | Budget |
|---|---|
| Initial JS gzipped | < 2MB |
| Time to first bird visible | < 500ms mid-tier mobile 4G synthetic |
| Idle FPS | 60 on reference 5-year laptop profile |
| JS heap growth 30 min | ~0 beyond GC noise (test threshold) |
| Snapshot payload | few KB typical |
| Tick p99 | alarm > 5s |

### 10.2 How to hit TTFB / first bird

- Critical CSS inlined; defer notebook/settings chunks
- Edge static hosting + compressed assets
- Snapshot fetch parallel to main parse; cookie session
- Draw birds before WebAudio resume
- Quiet field immediately paintable from CSS alone

### 10.3 Observability (allowed)

Aggregate RUM: navigation timing, first-bird paint custom mark, long animation frames, AudioContext errors, API latency histograms, tick latency, 5xx rates, session duration histograms **without account id and without bird dimensions**.

Synthetic browser fleet multi-geo schedule.

### 10.4 Deliberately do not measure / export to warehouse

- Per-bird interaction sequences for aggregation
- Personality distributions for “content” ML
- Funnel optimizations on streaks (none exist)
- Cross-account bird ranking

**Hard wall:** analytics DB role cannot read simulation tables.

### 10.5 Logging

Structured logs keyed by `account_id` UUID only. Lint/CI forbid `email` fields in log footprints outside auth service redact path.

---

## 11. Rollout

### 11.1 Delivery phases (engineering sequence)

**M0 — Foundations (week 1–2)**  
Monorepo, auth magic link, account UUID, empty quiet field SPA shell, session model, telemetry scaffold with privacy firewall.

**M1 — Canonical aviary + tick (week 2–4)**  
Birds table, snapshot API, event log, tick worker, mood+TOD, folder of pure sim tests + drift golden baselines, two starter adoption.

**M2 — Render/audio vertical slice (week 3–6, overlaps)**  
Scene, three zones, idle motion, procedural calls two species, listen-in, presence pings, mid-action load, top bar fade, settle.

**M3 — Interactions polish (week 5–7)**  
Offers + cooldown, greeting planner, notebook generator + UI, rename, third-bird age gate plumbing (even if unlock far out).

**M4 — Multi-device & visits (week 6–8)**  
SSE/poll solidity, presence cap multi-device, invites, visitor read-only mode, visit log, revoke.

**M5 — A11y & perf hardening (week 7–9)**  
Narration, reduced-motion mode designed, captions, keyboard, AA pass, bundle budget, heap soak, synthetic 500ms gate, WebAudio fallback path.

**M6 — Soft launch**  
Friend/family → limited cohort. Birds-per-aviary natural ramp (everyone starts 2). Watch tick p99, drift distributions vs calibration harness (not products for product analytics of birds).

### 11.2 Birds-per-aviary ramp

- Launch: force 2 = starters only.
- Age gates enable 3–7 over months without push notification.
- Feature flag: max birds override for internal dogfood only.

### 11.3 Day-one instrumentation

- Auth success/fail, tick latency, snapshot size, first-bird mark, audio error rate, event append rate, visit invite funnel counts **aggregate**, JS error rate.
- Internal-only drift calibration dashboard on synthetic accounts (not production user birds piped into analytics).

### 11.4 Feature flags

- `visits_enabled` (default on infrastructure, UX opt-in still required per invite)
- `nightjar_species_enabled`
- `notebook_salience_threshold`
- `tick_cadence_ms`

### 11.5 Migration & backup

- Continuous backup of aviary DB; PITR.
- Export verifies user can recover emotional state offline (JSON).
- Hard delete job cascades + email provider suppression.

### 11.6 Support surfaces

Matter-of-fact help blurb in settings; contact email. No in-aviary chatbots.

---

## 12. Risks

### 12.1 Drift calibration (highest product risk)

**Failure modes:** too fast → Tamagotchi grind; too slow → screensaver; negative implementer “fixes”; multi-device double-counting presence; laxer presence (“tab open”) inflating world drift.

**Mitigations:** pure function golden tests; synthetic week simulations; wall-clock presence cap; 180s activity window; review any PR touching drift weights; never expose numbers (prevents user-driven pressure to speed uncaping).

### 12.2 Sync correctness

**Failure modes:** accidental trait write API; client offline cache becoming authoritative; visitor events leaking into cursor; clock/order bugs.

**Mitigations:** schema-level privileges (only sim role updates trait columns); contract tests; visit_context enforced in tick filter with adversarial tests; versioned snapshots.

### 12.3 Audio uncanniness

**Failure modes:** looping artifacts; phase-cancellation chorusing; same motif repetition; latency on resume; mobile AudioContext quirks.

**Mitigations:** professional motif design pass; listen tests across devices; intentionally humanize; cap polyphony; caption parity; silence fallback better than pack of samples.

### 12.4 Accessibility regressions

**Failure modes:** live region spam; reduced-motion as “off”; missing keyboard listen-in; night focus ring invisible; narration as state dumps.

**Mitigations:** a11y acceptance journeys in CI (axe + manual scripts); design review of reduced-motion flats; narration snapshot tests for voice; ship gate: M5 complete before public launch.

### 12.5 “Helpful” engagement creep

**Failure modes:** welcome toast PR, streak in notebook, visit push default on, badge on settings.

**Mitigations:** non-goals checklist in PR template; codeowners on UI shell; design critique checklist: Notice/never announce, no gamification language.

### 12.6 Performance

**Failure modes:** asset bloat; memory leaks in audio; 7-bird bridge device jank; blocking main thread synth.

**Mitigations:** budgets in CI; audio work in small slices; object pools; adaptive quality (fewer ornaments) before dropping bird recognizability.

### 12.7 Privacy / PII leakage

**Failure modes:** email as partition key; bird stories in logs; warehouse join.

**Mitigations:** synthetic UUID rule + lint; separate analytics role; encryption of email; privacy test that warehouse fixtures lack interaction tables.

### 12.8 Operational tick delay

Users return to “frozen” time if workers stall.

**Mitigations:** p99 alarm 5s; backlog catch-up; SSE freshness warnings only system-voice if multi-minute stall (rare).

### 12.9 Emotional trust breaks

Reset birds, renumber IDs, restoring backup abnormalities.

**Mitigations:** identity continuity runbooks; never “regenerate aviary”; soft delete recovery; export.

### 12.10 Scope fracture into social network

Visit feature pressure toward chat/feed.

**Mitigations:** social_optional refusals as test cases (assert no endpoints); product sign-off required for any new social surface.

---

## 13. Cross-cutting engineering standards

### 13.1 Voice system

- Product/notebook/narration/captions: naturalist lowercase present-tense specific.
- Auth/errors/settings/a11y settings chrome chrome docs: matter-of-fact complete sentences capitalized.

Shared ESLint-ish or contentlint rules for copy catalogs where feasible.

### 13.2 Testing strategy

- **Unit:** sim drift banding, mood hysteresis, presence legitimacy, greet stagger never simultaneousbuses, offer cooldown, unlock ages, visit filter.
- **Contract:** API schema, event idempotency.
- **Client visual:** smoke storybook-like scenes day/night/rain/settle/reduced-motion.
- **Audio:** offline render of motif fingerprint tests (energy envelopes), not golden WAV of full cache.
- **E2E:** magic link (test double mail), adopt two birds, listen-in, offer, settle undo, notebook open, visit invite revoke.
- **Perf soak:** 30m puppeteer heap + fps.
- **Privacy:** static analysis + data-path integration forbidding PII propagation.

### 13.3 Security

- Session tokens secure, httpOnly if cookie, rotatable, revoke-all.
- Magic links single-use.
- CSRF as appropriate for cookie sessions.
- Invite tokens high entropy; rate limit enumeration.
- No public aviary ids ever listed.

### 13.4 Legal/compliance baseline

Privacy policy link in settings names aggregate categories and excludes per-bird state. Export + delete honor data subject expectations.

---

## 14. Named component checklist (build order inside modules)

1. `presence/monitor.ts` — visibility ∧ focus ∧ activity window  
2. `sim/drift.ts` — non-negative deltas + recency envelope  
3. `sim/mood.ts` — TOD + impulses  
4. `sim/greeting.ts` — absence-styled single-lead greeter  
5. `sim/tick.ts` — orchestrator  
6. `api/snapshot.ts` — strip forbidden fields  
7. `api/events.ts` — validate + append  
8. `web/scene/AviaryScene` — mid-action start  
9. `web/audio/CallEngine` — grammar + listen-in buses  
10. `web/a11y/NarrationClock`  
11. `web/a11y/ReducedMotionRenderer`  
12. `web/notebook/NotebookPanel`  
13. `api/visits/*`  
14. `jobs/hardDelete` + `jobs/export`

---

## 15. Success criteria for v1 launch

- New user can magic-link in, meet two named birds mid-motion under 500ms feel on target device class, hear distinct procedural calls, listen in, offer once, settle with undo, read at least the possibility of notebook sparsity (may be empty first day).
- Return after hours: non-canned greeting, mood continuity sensible, no welcome toast.
- Second device shows same birds/moods without merge UX.
- Optional friend visit works read-only; revoke kills access; no host push by default.
- Reduced-motion + SR + captions are charming naturalist surfaces, not dumps.
- No gamification surface accidental; menagerie ≤7; maps not hang.
- Drift harness shows week/three-week bands; neglect does not debuff traits.
- Telemetry privacy wall holds under review.

---

## 16. What implementers must not “cleverly” add

Even if easy:

- Welcome modal or confetti  
- Hunger or hearts UI  
- Public explore page  
- PCM chirp pack “for Safari”  
- Personality radar chart  
- Streak repair notification  
- Last-write-wins “offline traits”  
- Spinner as primary load metaphor  
- User beats on birds  
- Chat on visits  

If a proposal conflicts with notice-never-announce, monotonic expressive drift, or server-authored personality, it is out of product—not a backlog item.

---

## 17. Plan status

This document is the phase-1 comprehensive implementation plan for Pocket Aviary v1. It is intended to be executable by a frontier eng team without further product clarification. **No product code is implemented as part of this deliverable.**

End of plan.
