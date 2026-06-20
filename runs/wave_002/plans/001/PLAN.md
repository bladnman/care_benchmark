# Pocket Aviary — V1 Implementation Plan

This plan translates the Pocket Aviary PRD into an executable engineering roadmap. It assumes a frontier team shipping a production web product with separate frontend, simulation service, and auth/sync infrastructure. **No product code is produced in this phase** — this document is the sole deliverable.

---

## 1. Scope

### 1.1 In scope (v1)

| Area | V1 deliverable |
|------|----------------|
| Platform | Modern browsers only (last two major versions of Chrome, Safari, Firefox, Edge). Responsive single-screen horizontal aviary. |
| Accounts | Single-user accounts; one aviary per account; email magic-link auth (15-minute link TTL); per-device revocable sessions. |
| Birds | Two starter birds at adoption (system-selected species); up to seven birds total; species pool ~6; user-assigned renameable names; stable internal bird IDs. |
| Simulation | Server-side tick (~1/min); personality drift (monotonic toward expressive); mood system; bird-to-bird interactions; ambient weather; local-time day/night cycle. |
| Interactions | Return-greeting; presence accounting; listen-in; offer (seed, song fragment, still pool); settle; field notebook (auto-generated, read-only). |
| Sync | Multi-device canonical state via server snapshots + append-only event log; no client-authored personality writes. |
| Social | Visit invitations (off by default): email invite, read-only ambient view, revocable, 30-day invite expiry, visit log in settings. |
| Accessibility | Screen-reader narration (naturalist prose); reduced-motion designed surface; call captions; keyboard navigation; WCAG AA on all user-copy chrome. |
| Audio | Client-side procedural WebAudio call synthesis; chorus mixing; listen-in mix ramps; silence + captions fallback when WebAudio unavailable. |
| Performance | <2MB gzipped initial JS; <500ms time-to-first-bird on mid-tier mobile/4G; 60fps idle on 5-year-old laptop; no memory growth over 30 minutes. |

### 1.2 Explicitly out of scope (v1)

Per `non_goals.md` and PRD scope — **do not build, stub, or leave extension hooks for**:

- Native iOS/Android apps
- Gamification of any kind (achievements, streaks, levels, badges, visit calendars, XP)
- Tamagotchi mechanics (death, hunger, distress meters, decay-on-neglect)
- Social network surfaces beyond single read-only visit invitations (profiles, follows, discovery, comments, leaderboards, co-presence)
- Push notifications, email digests about aviary state (visit notifications opt-in only, off by default)
- Payments, shared aviaries, multi-aviary accounts, customizable scenes
- Recorded-audio fallback path
- Password auth, SSO (deferred)
- Exposing personality vector numerically anywhere (including debug/admin toggles for end users)

### 1.3 Defensible implementation calls (ambiguities resolved)

| Topic | Decision | Rationale |
|-------|----------|-----------|
| Presence activity window | **5 minutes** initial calibration; tune via shadow metrics in first month | PRD says "few minutes, lean longer" because watching without moving is the product |
| Mood enum | `wary`, `content`, `curious`, `drowsy`, `alert`, `settled` | PRD examples + settle state; `settled` is post-settle evening quiet |
| Personality trait range | `[0.0, 1.0]` floats, seeded per species template ± small jitter | Normalized scalars; never shown to user |
| Tick cadence | **60 seconds** default; configurable per environment | PRD "~once per minute" |
| Third+ bird unlock | Aviary age thresholds: 3rd at 21 days, 4th at 60, 5th at 120, 6th at 210, 7th at 365 (tunable) | "Age not visit count"; avoids gamification |
| Offer cooldown | **3 minutes** per bird per offer type | PRD "few minutes"; prevents drift saturation |
| Tech stack | TypeScript monorepo: React 19 + Vite client; Node/Fastify simulation API; PostgreSQL canonical store; Redis tick scheduler + snapshot cache | Team velocity; WebAudio in browser; fits budgets |
| Hosting | Edge-cached static client; API + tick workers in single region initially with CDN for snapshots | Meets <500ms first-bird with small snapshot payloads |

---

## 2. Architecture

### 2.1 Service topology

```
┌─────────────────────────────────────────────────────────────────┐
│                         Browser Client                          │
│  React scene renderer │ WebAudio engine │ A11y narration layer  │
│  Presence detector    │ Event emitter   │ Snapshot interpolator │
└────────────┬───────────────────────────────┬────────────────────┘
             │ HTTPS (REST + SSE optional)   │
             ▼                               ▼
┌────────────────────────┐      ┌───────────────────────────────┐
│   API Gateway / BFF    │      │   Auth Service (magic link)   │
│   - snapshot read      │      │   - email OTP links           │
│   - event append       │      │   - session tokens            │
│   - visit tokens       │      └───────────────────────────────┘
└────────────┬───────────┘
             │
             ▼
┌────────────────────────────────────────────────────────────────┐
│                    Simulation Service                          │
│  - Tick worker (cron/queue, ~1/min per active aviary)          │
│  - Drift + mood engine                                         │
│  - Notebook entry generator                                    │
│  - Snapshot materializer                                       │
└────────────┬───────────────────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────────────────────────┐
│  PostgreSQL (canonical)  │  Redis (snapshot cache, tick locks) │
│  - accounts, birds, vectors, moods, events, notebook, visits   │
└────────────────────────────────────────────────────────────────┘
```

### 2.2 Authority boundaries

| Layer | Owns | Never does |
|-------|------|------------|
| **Simulation tick (server)** | Personality vectors, mood transitions, perch assignments, weather rolls, notebook generation, canonical positions | Render frames; synthesize audio |
| **Client** | Rendering interpolation, procedural audio, presence detection, UI chrome, narration/caption presentation | Write personality state; advance simulation clock |
| **Event log** | Append-only interaction + presence records | Store derived simulation state as source of truth |

### 2.3 Render pipeline boundary

The client receives **snapshots** containing: bird IDs, species, display names, perch zone, pose/motion phase, mood, active call descriptors (motif IDs + timing params, not audio), day/night phase, weather state, settled flag, chorus state.

The client **does not** replay the full event log. It interpolates between snapshot N and N+1 for motion, and runs the call grammar locally using snapshot-provided parameters. This keeps snapshots kilobyte-scale and preserves procedural audio variation client-side.

### 2.4 Repository layout (proposed)

```
pocket-aviary/
├── apps/
│   ├── web/                 # React client
│   └── api/                 # Fastify BFF + route handlers
├── packages/
│   ├── simulation/          # Tick engine, drift, mood, notebook prose templates
│   ├── call-grammar/        # Shared motif definitions (used client + server for captions)
│   ├── contracts/           # OpenAPI types, event schemas, snapshot schemas
│   └── prose/               # Naturalist voice templates + lint rules
├── workers/
│   └── tick-worker/         # Scheduled simulation passes
└── infra/                   # IaC, synthetic perf checks
```

---

## 3. Data Model

### 3.1 Core entities

#### `accounts`
| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | Synthetic account ID — **only** identifier in logs, telemetry, sharding |
| `email_encrypted` | bytes | Encrypted at rest; decrypted only for auth emails |
| `email_hash` | string | For lookup/rate-limit without decrypt |
| `created_at` | timestamp | Drives aviary age for bird unlocks |
| `deletion_scheduled_at` | timestamp nullable | Soft delete window |
| `settings` | JSONB | a11y prefs, visit notification toggle (default off) |

#### `sessions`
| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | Per-device session token |
| `account_id` | UUID FK | |
| `device_label` | string | User-editable in settings |
| `created_at`, `last_seen_at` | timestamp | |
| `revoked_at` | timestamp nullable | |

#### `aviaries`
| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | 1:1 with account in v1 |
| `account_id` | UUID FK unique | |
| `timezone` | string | IANA; default from browser at first sign-in, changeable in settings |
| `settled_until` | timestamp nullable | Evening settle state |
| `last_tick_at` | timestamp | Worker bookkeeping |

#### `birds`
| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | Stable identity — never regenerated |
| `aviary_id` | UUID FK | |
| `species_id` | string | From species pool |
| `name` | string | User-assigned; renameable |
| `adopted_at` | timestamp | |
| `personality` | JSONB | `{ boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity }` each ∈ [0,1] |
| `mood` | enum | Current mood state |
| `perch_zone` | enum | `front` \| `middle` \| `back` |
| `pose_key` | string | Server-authored motion phase for interpolation |
| `last_offer_at` | JSONB | Per-offer-type cooldown timestamps |

**Hard rule:** personality JSON is written **only** by tick worker transactions.

#### `interaction_events` (append-only)
| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | |
| `aviary_id` | UUID FK | |
| `session_id` | UUID FK nullable | Null for tick-generated ambient events |
| `type` | enum | See §3.2 |
| `payload` | JSONB | Type-specific |
| `client_seq` | bigint | Per-session monotonic for ordering |
| `recorded_at` | timestamp | Server receipt time |

#### `presence_segments`
Derived table updated by tick from `presence_ping` events — not client-writable directly.

| Field | Type |
|-------|------|
| `aviary_id`, `session_id` | UUID |
| `started_at`, `ended_at` | timestamp |
| `duration_seconds` | int |

#### `notebook_entries`
| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | |
| `aviary_id` | UUID FK | |
| `body` | text | Lowercase naturalist prose |
| `created_at` | timestamp | |
| `trigger_event_id` | UUID nullable | Source event if any |

#### `visit_invitations`
| Field | Type | Notes |
|-------|------|-------|
| `id` | UUID | |
| `aviary_id` | UUID FK | |
| `visitor_email_hash` | string | |
| `token_hash` | string | One-time link |
| `created_at`, `expires_at` | timestamp | 30-day expiry |
| `revoked_at` | timestamp nullable | |

#### `visit_sessions` (host audit log only)
| Field | Type |
|-------|------|
| `invitation_id`, `started_at`, `ended_at`, `approx_duration_seconds` | |

### 3.2 Event types

| Type | Client-authored | Payload highlights |
|------|-----------------|-------------------|
| `presence_ping` | Yes | `{ focused: bool }` — emitted every 30s while presence conditions met |
| `listen_in_start` / `listen_in_end` | Yes | `{ bird_id }` |
| `offer` | Yes | `{ offer_type: seed\|song\|pool, target_bird_id optional }` |
| `settle` | Yes | `{}` |
| `settle_undo` | Yes | Within 5s window |
| `session_visible` | Yes | Tab became visible — triggers return-greeting eligibility |
| `tick_ambient` | Server | Weather, mood nudges, bird-to-bird call responses |

### 3.3 Snapshot schema (API response)

```typescript
type AviarySnapshot = {
  version: number;           // monotonic per aviary
  captured_at: string;       // ISO
  timezone: string;
  day_phase: 'night' | 'dawn' | 'morning' | 'midday' | 'afternoon' | 'dusk' | 'evening';
  weather: 'clear' | 'rain' | 'wind' | null;
  settled: boolean;
  absence_seconds: number;     // for return-greeting calibration
  birds: BirdSnapshot[];
  active_calls: CallDescriptor[];
  greeting: GreetingDirective | null;  // which bird, which template family
};

type BirdSnapshot = {
  id: string;
  name: string;
  species_id: string;
  mood: Mood;
  perch_zone: PerchZone;
  pose_key: string;
  pose_t: number;            // 0-1 phase within pose
  plumage_saturation_render: number; // derived from personality, not raw vector
};
```

Personality vectors are **omitted** from snapshots sent to clients.

---

## 4. API Surface

Base path: `/api/v1`. All authenticated routes require `Authorization: Bearer <session_token>` except magic-link exchange and visit read tokens.

### 4.1 Auth

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/auth/magic-link` | Body: `{ email }`. Rate-limited. Sends link. |
| POST | `/auth/exchange` | Body: `{ token }`. Returns session + account bootstrap if new. |
| POST | `/auth/logout` | Revokes current session. |
| GET | `/auth/sessions` | List revocable device sessions. |
| DELETE | `/auth/sessions/:id` | Revoke device. |

**New account bootstrap:** creates aviary, selects two starter species (deterministic seed from account UUID for reproducibility), prompts naming flow client-side, writes `bird_adopted` internal events.

### 4.2 Aviary state

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/aviary/snapshot` | Current `AviarySnapshot`. Supports `?since_version=` for 304. |
| POST | `/aviary/events` | Append batch of interaction events. Body: `{ events: Event[] }`. Returns `{ accepted_through_seq }`. |
| GET | `/aviary/notebook` | Paginated entries, newest first. Cursor-based. |

### 4.3 Birds

| Method | Path | Purpose |
|--------|------|---------|
| PATCH | `/birds/:id` | Rename only: `{ name }`. |
| GET | `/birds/available` | Returns next age-gated adoption offer if eligible; else 204. |
| POST | `/birds/adopt` | Accept offered species; body `{ species_id, name }`. |

### 4.4 Account

| Method | Path | Purpose |
|--------|------|---------|
| GET/PATCH | `/account/settings` | Timezone, a11y prefs, visit notification toggle. |
| POST | `/account/export` | Queues JSON export email. |
| POST | `/account/delete` | Schedules soft delete. |
| POST | `/account/delete/cancel` | Restores within 30 days. |

### 4.5 Visits (host)

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/visits/invite` | Body: `{ email }`. Sends one-time link. |
| GET | `/visits/invitations` | Outstanding + history. |
| DELETE | `/visits/invitations/:id` | Revoke. |
| GET | `/visits/log` | Who visited, when, duration. |

### 4.6 Visits (visitor — separate token)

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/visits/exchange` | Body: `{ token }`. Returns read-only session. |
| GET | `/visits/snapshot` | Same schema as host snapshot; **no event write endpoints**. |

Visitor sessions: no `POST /aviary/events` except heartbeat for session duration logging. Presence from visitors **discarded** at API layer.

### 4.7 Real-time (optional v1.1 within v1 if time)

Server-Sent Events: `GET /aviary/stream` pushes snapshot version bumps. **Fallback:** client polling per §4.2 pull rules. Ship polling first; add SSE if snapshot latency matters in dogfood.

### 4.8 Error contract

HTTP problem+json. Matter-of-fact copy in `detail` field — never naturalist voice on errors.

---

## 5. Simulation Engine Design

### 5.1 Tick pipeline (per aviary, each minute)

```
1. Acquire Redis lock `tick:{aviary_id}`
2. Load birds + personality + mood + last processed event cursor
3. Fetch new interaction_events since cursor
4. Update presence_segments from presence_ping events
5. Apply mood transitions (time-of-day, weather, recent interactions, bird-to-bird)
6. Compute personality drift deltas (see §5.2)
7. Update perch assignments from mood + boldness
8. Roll ambient weather (low probability per tick)
9. Generate bird-to-bird call events if vocal_frequency + timing align
10. Evaluate notebook entry triggers (sparse)
11. Materialize snapshot + increment version
12. Cache snapshot in Redis (TTL 5 min)
13. Commit transaction; release lock
```

Tick runs for **all aviaries** regardless of connected clients. Inactive aviaries (no events in 30 days) move to **slow tick** (every 15 min) to save compute — mood/time still advance.

### 5.2 Drift function

**Principle:** monotonic toward expressive; no negative drift on neglect.

Per trait, per tick:

```
delta = 0
delta += w_presence * normalize(presence_minutes_last_24h)
delta += w_listen * listen_in_minutes_last_7d[target_bird]
delta += w_offer_accept * offers_accepted_last_7d
delta += w_offer_near * offers_near_bird_last_7d

// Per-trait routing:
boldness_delta        = delta * k_bold * offer_near_signal
social_warmth_delta   = delta * k_social * listen_in_signal
vocal_frequency_delta = delta * k_vocal * (presence + listen_in)
plumage_saturation_delta = delta * k_plumage * presence_only
curiosity_delta       = delta * k_curiosity * offer_accept_signal

personality[trait] = min(1.0, personality[trait] + delta)
```

**Calibration targets (CI instrumentation):**
- Simulated "regular visit" script (15 min/day presence, 2 listen-ins/week): measurable vector change ≥ 0.02 L1 norm in 7 days
- Same script: user-visible behavior change (greeting frequency, perch forward bias) detectable in playtest rubric by day 21

Constants `w_*` and `k_*` live in `simulation/config/drift_v1.json` — tune without code deploy via feature flag.

**Neglect behavior:** if `presence_minutes_last_14d == 0`, drift deltas are zero; mood may still shift via time-of-day; birds become **ambient** (lower greeting probability via mood/perch, not lower personality).

### 5.3 Mood transitions

Finite state machine per bird, evaluated each tick:

**Inputs:** local hour, weather, recent offer/listen-in/settle events, neighboring bird moods, personality (boldness dampens wary transitions).

**Example rules:**
- `dusk` hour window → bias toward `drowsy`
- `rain` active → vocal dampening modifier; bias `content` or `wary` by curiosity
- neighbor `wary` + low boldness → increase `wary` probability
- offer accepted in last 10 min → bias `content`
- `settle` active on aviary → push all toward `settled` / `drowsy`

Mood persists across sessions; no reset on tab open.

### 5.4 Return-greeting (server directive, client execution)

On `session_visible` event after absence:

1. Server computes `absence_seconds` since last presence segment end
2. Selects greeting bird: highest `social_warmth * boldness`, modulated by mood (wary birds may defer)
3. Emits `GreetingDirective` in next snapshot: `{ bird_id, template_family, absence_tier: short|medium|long, stagger_ms[] }`
4. Client plays procedural greeting motion + call; **no toast**

Absence tiers (initial): short < 30 min, medium < 48h, long ≥ 48h.

### 5.5 Call grammar runtime (server for captions; client for audio)

Each species has a **motif library**: 8–12 motifs (frequency envelopes, pitch contours as parametric curves).

`CallDescriptor` in snapshot:
```json
{
  "bird_id": "...",
  "motif_id": "warbler_rising_triplet",
  "params": { "base_hz": 4200, "duration_ms": 340, "jitter": 0.08 },
  "mood_modifier": "soft",
  "started_at": "..."
}
```

Client synthesizes via WebAudio oscillators + filtered noise bursts. Captions generated from same params: *"a soft three-note rise"*.

**Chorus:** multiple descriptors overlapping; client mixer applies per-bird gain. Listen-in adjusts gains (see §8).

### 5.6 Notebook generation

Trigger engine (sparse):

| Trigger | Example entry |
|---------|---------------|
| First greeting order of day | *"tuesday — pip greeted before wren today, first time this week."* |
| Notable mood stretch | *"a long stretch of quiet this morning. pip preened for several minutes without looking up."* |
| Weather passed | *"rain passed through briefly. wren stayed on the back perch throughout."* |

**Rate limit:** max 1 entry per 48h rolling unless `noteworthy` flag (first-time-week patterns, new bird adoption). Template slots filled by `packages/prose` with bird names lowercased.

Notebook never logs user visit frequency or streaks.

### 5.7 Bird-to-bird interaction

When two+ birds have active call descriptors in same 2s window and vocal_frequency > threshold → `chorus` event; may trigger response motif from neighbor.

Wary mood in one bird: 30% chance per tick to nudge adjacent birds (same perch zone) toward wary.

---

## 6. Sync Model

### 6.1 Single canonical writer

All personality and mood authority lives in tick worker. Clients are pure event emitters.

### 6.2 Event ordering

Per session: client assigns monotonic `client_seq`. Server orders by `(recorded_at, session_id, client_seq)`.

Duplicate submission (retry): idempotency key `event.id` UUID — server ignores duplicates.

### 6.3 Multi-device behavior

| Scenario | Behavior |
|----------|----------|
| Laptop + phone both open | Both pull same snapshot version; both append events; tick merges in log order |
| Concurrent listen-in on two devices | Both events logged; tick uses combined listen-in minutes for drift |
| Phone offline 2 hours | On reconnect, pull latest snapshot; client discards stale local interpolation state |
| No last-write-wins | **Impossible** — clients cannot send personality absolutes |

### 6.4 Conflict surfaces

| Failure | UX |
|---------|-----|
| Expired magic link | Matter-of-fact retry prompt |
| Revoked session | Sign in again |
| Snapshot load failure | Reload CTA; support contact |
| Event reject (aviary deleted) | Redirect to account recovery |

### 6.5 Visit read path isolation

Visitor token maps to `aviary_id` read-only. API middleware blocks all write event types except `visit_heartbeat`. Host drift unaffected by visitor presence.

---

## 7. Frontend Rendering Pipeline

### 7.1 Boot sequence (critical path)

```
HTML shell (inline critical CSS: quiet sky field)
  → load JS chunk `core` (<800KB gz target)
  → auth check / magic-link landing
  → GET /aviary/snapshot (parallel with species SVG prefetch)
  → first frame: birds mid-pose, ambient leaf optional, NO spinner
  → start rAF loop + audio context unlock on first gesture
  → load deferred chunks: settings, visits, notebook drawer
```

**Slow connection:** show quiet field (sky gradient + subtle shader noise) until snapshot arrives — never a spinner.

### 7.2 Scene composition

Layers (back to front):
1. Sky gradient (day_phase driven)
2. Background foliage (parallax 0.2x)
3. Perch structures (front/middle/back depth)
4. Birds (SVG or lightweight canvas sprites)
5. Foreground leaves/feathers (client ornament, not server state)
6. Offer props (seed, pool) when active
7. Chrome: top bar only (outside scene bounds)

**Perch zones:** fixed x-ranges; birds lerp between zones on snapshot change.

### 7.3 Motion system

| Mode | Behavior |
|------|----------|
| Standard | Skeletal 2D rig: preen, scan, fluff, hop — mood selects animation set |
| Reduced motion | Still poses + 2s cross-fade between poses; no leaf drift; day/night color shifts slowed 2x |

Interpolation: between snapshots, client extrapolates pose phase for up to 90s; on new snapshot, smooth correction over 300ms (standard) or cut cross-fade (reduced).

### 7.4 Idle micro-motion

Always-on per bird unless settled/asleep. Wary → more scanning; content → preen; curious → head tilt toward ambient sounds.

### 7.5 Top bar chrome

Icons: account, accessibility, notebook, offer. Auto-fade to 10% opacity after 3s pointer idle; restore on move/key.

### 7.6 Interaction wiring

| Gesture | Handler |
|---------|---------|
| Click/tap bird | listen-in focus |
| Click empty | listen-in end |
| Keyboard | Tab to bar → arrows between birds → Enter listen-in → Escape clear |
| Offer menu | seed / song / pool → POST event → server mood reaction in next snapshot |
| Settle | triggers settle event + local immediate lighting lerp; 5s undo listener |

### 7.7 Visit (read-only) client

Strip write affordances. Same renderer. Revoked invite → matter-of-fact full-page message.

---

## 8. Audio Pipeline

### 8.1 Architecture

```
CallDescriptor[] → CallGrammarEngine → Voice nodes per bird
                                      → Master bus
                                      → ListenInMix (per-bird gains)
                                      → Limiter → destination
```

### 8.2 Procedural synthesis

- Motif = sequence of `{ type: tone|noise, freq, dur, envelope }`
- Mood modifies attack/decay and filter cutoff
- Personality `vocal_frequency` scales call scheduling probability client-side between snapshots
- **No audio files shipped**

### 8.3 Chorus mixing

Default gains: 1.0 per active caller, normalized if >3 simultaneous to prevent clipping.

### 8.4 Listen-in mix

| State | Focused bird | Others |
|-------|--------------|--------|
| Engaged | ramp to 1.0 over 800ms | ramp to 0.35 over 800ms (never 0) |
| Disengaged | return to chorus default over 800ms | |

### 8.5 WebAudio fallback

If `AudioContext` fails:
- All call playback disabled
- Auto-enable captions
- Settings banner (matter-of-fact): *"Audio isn't available in this browser. Captions are on."*
- No recorded fallback — ever

### 8.6 Resource management

- Pool of 8 oscillator nodes reused
- Buffer pool for noise bursts (pre-allocated)
- No per-call `AudioBuffer` allocation
- CI: 30-min soak test asserts heap stable ±5%

---

## 9. Accessibility Surfaces

### 9.1 Screen-reader narration

- `aria-live="polite"` region **outside** the canvas
- Narration scheduler: 30–60s idle cadence; immediate on greeting, offer reaction, settle
- Prose generated from same snapshot + templates as notebook voice
- **Not** positional ARIA labels on birds (`aria-label="Pip perch 2"` forbidden)

### 9.2 Reduced motion

- Honor `prefers-reduced-motion` + settings override
- Distinct pose cross-fade renderer — not `animation: none`
- Audio + notebook + drift unchanged

### 9.3 Captions

- Opt-in setting (default on when audio off)
- Rendered near calling bird; fade with call envelope
- Generated from CallDescriptor + mood adjectives

### 9.4 Keyboard + focus

- Full tab order documented in QA script
- Focus ring: 3px `#F5F0E8` outer glow, passes contrast on dawn/dusk backgrounds

### 9.5 WCAG AA

- All chrome text: minimum 4.5:1 (design tokens in `apps/web/styles/tokens.css`)
- Captions: semi-opaque backing plate for contrast

---

## 10. Performance Budgets and Observability

### 10.1 Budgets

| Metric | Budget | Measurement |
|--------|--------|-------------|
| Initial JS (gzip) | < 2 MB | Webpack/Vite bundle analyzer CI gate |
| Time to first bird | < 500 ms | Lighthouse + synthetic RUM (p75) |
| Idle FPS | ≥ 60 on reference laptop | Playwright frame timing |
| Memory @ 30 min | ±5% of minute-5 baseline | Chrome DevTools protocol in CI |
| Snapshot payload | < 4 KB typical | API metrics |
| Tick duration | p99 < 2s | Worker histogram |

Reference device: 5-year-old MacBook Air equivalent + Moto G Power on 4G throttled.

### 10.2 Observability (privacy-safe)

**Collect (aggregate only):**
- `snapshot.latency_ms` histogram
- `tick.duration_ms` by stage
- `first_bird_render_ms` RUM
- `audio.context_init_fail` count
- `bundle.bytes` per deploy

**Never collect:**
- Per-bird state, personality values, notebook content, per-account interaction sequences

**Alerting:** tick p99 > 5s pages on-call.

**Synthetic checks:** hourly Playwright from 3 regions — load, assert bird visible < 500ms, 60 frames in 1s idle.

### 10.3 CI gates (merge blockers)

- Bundle size
- Drift calibration integration test (7-day simulated presence)
- Memory soak (headless, 5 min compressed)
- Prose lint (no gamification words, no exclamation marks in notebook templates)
- a11y: axe on settings + notebook surfaces

---

## 11. Rollout

### 11.1 Phase 0 — Internal dogfood (weeks 1–2)

- Simulation tick + API only; CLI snapshot inspector
- Drift constants tuned against automated scripts

### 11.2 Phase 1 — Private alpha (weeks 3–6)

- Full client: 2 birds, listen-in, presence, day/night
- No visits, no notebook yet
- 20 internal users; daily snapshot of drift metrics

### 11.3 Phase 2 — Closed beta (weeks 7–10)

- Notebook, offers, settle, magic-link auth, multi-device
- Accessibility narration + reduced motion
- Visits behind feature flag

### 11.4 Phase 3 — Public v1 (week 11+)

- Gradual traffic ramp 5% → 100%
- Bird unlock pacing enabled
- Visits on for all; invite flow monitored

### 11.5 Birds-per-aviary ramp

Ship with **2 birds** only for first 2 weeks of public beta. Enable age-gated 3rd bird after drift calibration validated in production RUM.

### 11.6 Day-one instrumentation

- `first_bird_render_ms`
- `snapshot.version` lag (client behind server)
- `presence.minutes` (aggregate histogram, no account dimension in dashboard — engineering query with UUID only for support)
- `audio.init_fail_rate`
- `notebook.entries_per_aviary_per_week` (aggregate)

---

## 12. Risks

| Risk | Severity | Mitigation |
|------|----------|------------|
| **Drift too fast/slow** | High — breaks relationship promise | Isolated config; simulated 7/21-day CI; playtest rubric; kill switch to freeze drift |
| **Presence signal inflation** | High — silent drift corruption | Strict 3-condition AND gate; integration tests for background tab, minimized window; shadow metric comparing strict vs lax |
| **Sync / personality loss** | Critical | Server-only vector writes; additive deltas; event log immutability; backup + PITR on Postgres |
| **Audio uncanny valley** | High — breaks aliveness | Procedural jitter bounds; user testing; caption fallback; per-species motif QA listening sessions |
| **Bundle budget blowout** | Medium | Code-split; procedural assets; CI gate |
| **First-frame loading violation** | Medium — breaks core conceit | Inline shell; snapshot CDN; defer all non-critical JS |
| **Accessibility regression** | Medium — legal + product ethics | Ship a11y with v1, not after; screen-reader QA cohort |
| **"Welcome back" creep** | Medium — culture drift | Prose linter; design review checklist; reject PRs with announcement patterns |
| **Gamification creep** | Medium | Explicit non-goals in CONTRIBUTING; notebook template review |
| **Visit privacy leak** | High | Visitor write block middleware; audit visitor session logs; pen test invite tokens |
| **Tick worker backlog** | Medium | Per-aviary locks; slow-tick tier; horizontal worker scaling |
| **WebAudio autoplay policy** | Low | Unlock on first interaction; aviary audible before unlock is optional ambient |

---

## 13. Team Workstreams and Milestones

| Workstream | M1 (week 4) | M2 (week 8) | M3 (week 12) |
|------------|-------------|-------------|--------------|
| Simulation | Tick + drift + mood | Notebook + bird-to-bird | Age unlock + weather |
| API/Auth | Magic link + events | Multi-device + export | Visits |
| Client render | Scene + interpolation | Interactions + day/night | Reduced motion |
| Audio | Single-bird calls | Chorus + listen-in | Caption sync |
| A11y | Keyboard + contrast | Narration | Captions + reduced motion QA |
| Infra | CI budgets | Synthetic RUM | Production ramp |

**Estimated team:** 2 backend, 2 frontend, 1 audio/render specialist, 1 QA/a11y, 0.5 design for tokens/chrome.

---

## 14. Acceptance Criteria (v1 ship gate)

1. User signs in via magic link; sees two birds mid-motion within 500ms on reference mobile.
2. No welcome toast/banner on any return path.
3. Presence only recorded when visibility ∧ focus ∧ recent input (verified by automated tab-focus tests).
4. Listen-in ramps mix; other birds never silent.
5. Settle shifts lighting; undo within 5s works; tab-close without settle not penalized.
6. Notebook entries sparse, naturalist voice, no user-behavior tracking prose.
7. Personality drift measurable at 7d in test harness; not exposed in UI.
8. Multi-device: rename on phone reflects on laptop within 1 tick.
9. Visitor read-only; host birds unchanged after 1h visitor session.
10. Reduced motion is cross-fade aesthetic, not broken static.
11. WebAudio failure → silence + captions, no crash.
12. Bundle < 2MB gzip; 30-min memory flat.
13. Account delete soft/hard flow works; export JSON correct.

---

*End of plan.*
