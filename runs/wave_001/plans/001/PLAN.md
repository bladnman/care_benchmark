# Pocket Aviary v1 — Implementation Plan

## 1. Scope

### In v1
- Single-user accounts; email magic-link sign-in; no passwords, no SSO
- One aviary per account; two starter birds at account creation, cap of seven
- Five personality traits per bird (boldness, social warmth, vocal frequency, plumage saturation, curiosity)
- Server-side simulation tick (~1 min cadence): drift, mood transitions, ambient events
- Multi-device sync as an architectural property of the canonical server state
- Client interactions: return-greeting, listen-in, offer (seed, song fragment, still pool), settle, field notebook
- Presence accounting: triple-condition signal (visibility + focus + activity within window)
- Procedural audio: per-bird call grammar synthesized via WebAudio client-side
- Aviary visual scene: single horizontal scene, three perch zones, day/night cycle, ambient weather (rain, wind)
- Field notebook: sparse naturalist prose, auto-generated, read-only
- Social: one-to-one visit invitations (off by default, email-based, revocable)
- Visit log in account settings; no notifications on visit by default
- Accessibility: screen-reader narration (naturalist prose, slow cadence), reduced-motion mode (cross-fade rendering), call captions
- Account export (JSON on demand), soft-then-hard deletion (30-day window)
- Aggregate operational telemetry only; zero per-account data in analytics pipeline
- WCAG AA contrast on all user-copy text

### Out of v1 (hard non-goals)
- Native iOS or Android apps
- Gamification of any kind (no streaks, achievements, badges, scores, green-dot calendar, visit counters)
- Tamagotchi mechanics (no hunger, distress, or death; no decay on neglect)
- Social network surfaces (no profiles, follows, public discovery, comments, leaderboards)
- Push notifications (email, browser push, or in-product ping)
- Shared or multi-user aviaries
- Multiple aviaries per account
- Password-based or SSO login
- Payments or tiers
- Native-app-targeted protocol design

---

## 2. Architecture

### Service decomposition

```
Browser Client
  ├─ Render Worker (canvas/SVG pipeline, WebAudio context)
  ├─ State Client (snapshot pull, event write)
  └─ Presence Monitor (visibility + focus + activity)

API Gateway (Edge CDN with routing)
  ├─ Auth Service           — magic-link issue/consume, session CRUD
  ├─ Aviary State Service   — snapshot read, event-log write
  ├─ Simulation Service     — tick scheduler, drift, mood, notebook generation
  ├─ Visit Service          — invite CRUD, visit-session read-only access
  ├─ Notebook Service       — entry read, generation (called by Simulation)
  └─ Account Service        — profile, export, deletion lifecycle

Data stores
  ├─ Accounts DB (PostgreSQL) — accounts, sessions, magic links
  ├─ Aviary DB (PostgreSQL)   — birds, personality vectors, moods, aviary state
  ├─ Event Log (append-only, e.g. DynamoDB or Postgres with partition by account)
  ├─ Notebook Store (Postgres) — notebook entries ordered by timestamp
  └─ Visit Store (Postgres)   — invitations, visit sessions, visit log
```

### Client/server split
- **Server owns all state.** Personality vectors, moods, aviary canonical state, notebook entries, presence-time totals — all server-side. The client never writes personality.
- **Client renders snapshots and interpolates.** The client pulls a state snapshot on load, on tab-visibility change, and on a low-frequency keepalive (~30s while visible). Between snapshots the client interpolates bird positions and continues audio synthesis.
- **Client writes events only.** Interaction events (offer, listen-in start/end, settle, presence ping) are written to the append-only event log. Events are fire-and-forget with retry; they are never treated as authoritative state by the client.
- **Simulation tick is server-only.** No tick logic runs on the client. The client never simulates drift or mood transitions.

### Render pipeline boundary
The browser has two workers:
1. **Render Worker** — owns the canvas (or SVG) context, WebAudio context, animation loop. Receives snapshot diffs from the main thread. Does not touch the network.
2. **State Client** (main thread) — handles HTTP, WebSocket keepalive, event write queue, snapshot diffing, and bridges to the Render Worker via `postMessage`.

This boundary protects the animation loop from network jank: slow snapshots never block frame rendering.

---

## 3. Data Model

### Account record
```
accounts
  id              UUID (synthetic, PK — never email)
  email_encrypted TEXT (encrypted at rest; indexed on hash for lookup)
  email_hash      TEXT (indexed for login lookup, non-reversible)
  created_at      TIMESTAMPTZ
  deleted_at      TIMESTAMPTZ (null = active; non-null = soft-deleted)
  export_requested_at TIMESTAMPTZ
```

### Session record
```
sessions
  id              UUID
  account_id      UUID (FK accounts.id)
  device_hint     TEXT (user-agent derived; display only)
  issued_at       TIMESTAMPTZ
  last_seen_at    TIMESTAMPTZ
  revoked_at      TIMESTAMPTZ (null = active)
```

### Magic link record
```
magic_links
  id              UUID
  account_id      UUID (FK accounts.id — null if new account flow)
  email_hash      TEXT
  token_hash      TEXT (hash of the one-time token; never stored plain)
  expires_at      TIMESTAMPTZ (15 min from issue)
  consumed_at     TIMESTAMPTZ (null = not yet used)
```

### Aviary record
```
aviaries
  id              UUID (PK)
  account_id      UUID (FK accounts.id)
  created_at      TIMESTAMPTZ
  day_night_offset_minutes INT (local timezone offset stored at session start)
```

### Bird record
```
birds
  id              UUID (PK; stable forever)
  aviary_id       UUID (FK aviaries.id)
  name            TEXT
  species_id      TEXT (from the species pool enum)
  adopted_at      TIMESTAMPTZ
  -- Personality vector (server-owned; never exposed to client numerically)
  pv_boldness           FLOAT4  -- [0.0, 1.0]
  pv_social_warmth      FLOAT4
  pv_vocal_frequency    FLOAT4
  pv_plumage_saturation FLOAT4
  pv_curiosity          FLOAT4
  -- Mood (fast-timescale)
  mood                  TEXT    -- enum: wary | content | curious | drowsy | alert
  mood_set_at           TIMESTAMPTZ
  -- Perch state (derived/cached from last tick)
  perch_zone            TEXT    -- front | middle | back
  perch_slot            INT     -- lateral position within zone
```

**Design decisions:**
- Personality vector columns live on the bird row (not a separate table) because they are always read/written together by the tick, and row-level locking on the tick write is the correct conflict model.
- `pv_*` fields are never returned in any client-facing API response. API shapes are defined to exclude them.
- `mood` persists across sessions — not reset on load.

### Interaction event log
```
interaction_events
  id              UUID
  account_id      UUID  (partition key for sharding / lifecycle delete)
  bird_id         UUID  (nullable — settle and presence pings are aviary-level)
  event_type      TEXT  -- presence_ping | listen_in_start | listen_in_end
                        -- offer_seed | offer_song | offer_pool
                        -- settle | tab_close
  occurred_at     TIMESTAMPTZ
  payload         JSONB -- e.g. {duration_seconds: 180} for listen_in_end
```

This table is append-only. No update, no delete except via account deletion cascade. The tick reads events since last tick run; processed events are not deleted but are no longer re-read (high-water-mark stored on the tick state record).

### Tick state record
```
tick_states
  aviary_id          UUID (PK)
  last_ticked_at     TIMESTAMPTZ
  event_hwm          UUID  (ID of last interaction_event processed)
  next_tick_at       TIMESTAMPTZ
```

### Notebook entry
```
notebook_entries
  id          UUID
  aviary_id   UUID
  generated_at TIMESTAMPTZ
  prose       TEXT  -- naturalist field-notebook prose; never a template string
```

No update. Read-only after insert. The notebook service generates entries; the client reads them in descending order with cursor pagination.

### Visit model
```
visit_invitations
  id          UUID
  aviary_id   UUID (host)
  visitor_email_hash TEXT
  visitor_email_encrypted TEXT
  issued_at   TIMESTAMPTZ
  expires_at  TIMESTAMPTZ  (30 days from issued_at)
  revoked_at  TIMESTAMPTZ
  consumed_at TIMESTAMPTZ  (set when visitor first opens link)

visit_sessions
  id              UUID
  invitation_id   UUID (FK)
  started_at      TIMESTAMPTZ
  last_ping_at    TIMESTAMPTZ
  ended_at        TIMESTAMPTZ
```

---

## 4. API Surface

All endpoints are REST/JSON over HTTPS. Auth header: `Authorization: Bearer <session_token>`. The API gateway enforces rate limits and session validity.

### Auth endpoints
```
POST /auth/request-link
  body: { email: string }
  → 202 (always; never confirm or deny email existence for enumeration safety)

GET  /auth/confirm?token=<one-time-token>
  → 302 to app with Set-Cookie: session_token=<JWT>; HttpOnly; SameSite=Lax
  → 400 if expired or already consumed (matter-of-fact error body)

DELETE /auth/sessions/:session_id
  → 204 (revoke a specific session; requires auth)

GET  /auth/sessions
  → [{id, device_hint, issued_at, last_seen_at}] (list user's sessions)
```

### Aviary state endpoints
```
GET /aviary/snapshot
  Authorization required.
  Returns current canonical aviary state snapshot:
  {
    aviary_id,
    fetched_at,
    day_night: { phase: "morning"|"midday"|"evening"|"night", progress: 0..1 },
    weather: { active: bool, type: "rain"|"wind", intensity: 0..1 },
    birds: [
      {
        id, name, species_id,
        mood,           // wary | content | curious | drowsy | alert
        perch_zone,     // front | middle | back
        perch_slot,
        // NO personality vector fields here — ever
        call_state: { motif_seed, timing_offset_ms }
          // Deterministic seed for procedural call generation on client
      }
    ],
    tick_scheduled_at    // client can use for keepalive timing hint
  }

  Cache: short (10s edge TTL), vary by account session.
```

```
GET /aviary/snapshot/delta?since=<ISO8601>
  Returns only fields that changed since the given timestamp.
  Same shape as full snapshot but with unchanged fields omitted.
  Used by keepalive poll.
```

### Interaction event endpoint
```
POST /aviary/events
  Authorization required.
  body: {
    events: [
      { event_type, bird_id?, occurred_at, payload? }
    ]
  }
  → 202  (accepted; always async; client does not wait for tick)
  → 207  (partial) if some events failed schema validation (rare)

  Client batches events. Failed writes are retried with exponential backoff.
  Max batch: 50 events per request. Events are idempotent by (account_id, occurred_at, event_type, bird_id).
```

### Field notebook endpoint
```
GET /aviary/notebook?cursor=<opaque>&limit=20
  Authorization required.
  Returns:
  {
    entries: [{ id, generated_at, prose }],
    next_cursor: string | null
  }
  Ordered newest-first. Cursor-based, no offset pagination.
```

### Visit invitation endpoints (host)
```
POST /visits/invitations
  body: { visitor_email: string }
  → 201 { id, expires_at }
  (system sends email to visitor_email with one-time visit link)

GET  /visits/invitations
  → [{ id, visitor_email_masked, issued_at, expires_at, revoked_at, consumed_at }]
  (masked email: first char + *** + domain)

DELETE /visits/invitations/:id
  → 204  (revokes invitation; active visit sessions get 410 on next snapshot pull)
```

```
GET  /visits/log
  → [{ visitor_email_masked, visit_started_at, visit_ended_at }]
  Ordered by visit_started_at desc.
```

### Visit session endpoints (visitor)
```
GET /visits/:invitation_token/snapshot
  No auth required (invitation token is the auth).
  Same snapshot shape as /aviary/snapshot but source is host's aviary.
  Returns 410 if invitation revoked, expired, or consumed-and-expired.
  Visitor cannot POST events.

GET /visits/:invitation_token/snapshot/delta?since=<ISO8601>
  Delta keepalive for visitor.
```

### Account management endpoints
```
GET  /account
  → { email_masked, created_at }

POST /account/export
  → 202 (export queued; JSON will be emailed to verified address)

POST /account/delete
  → 202 (soft-deletion initiated; account recoverable for 30 days)

POST /account/recover
  → 204 (cancel pending deletion; available while within 30-day window)

GET  /account/visit-notification-setting
PUT  /account/visit-notification-setting
  body: { enabled: bool }
```

---

## 5. Simulation Engine Design

### Tick scheduler
The simulation service runs a persistent background scheduler (e.g., a Go goroutine pool or a Node worker cluster). Each aviary has a `tick_states` record with `next_tick_at`. The scheduler polls `tick_states` for records where `next_tick_at <= now` and dispatches tick workers.

Tick cadence: ~60 seconds per aviary. Under load, ticks may slip; slippage is tolerable up to ~5 minutes without noticeable user impact (drift is slow, mood transitions are gradual). The p99 tick latency alarm fires at 5 seconds, catching degradation.

**Tick is idempotent at the row level.** If a tick crashes mid-write, re-running it from the same `event_hwm` produces the same output. The personality vector update is a delta applied to the pre-tick vector; re-running from the same inputs produces the same delta.

### Tick algorithm (per aviary)

```
function tick(aviary_id):
  lock aviary row (SELECT FOR UPDATE)
  load tick_state = tick_states[aviary_id]
  load birds = birds WHERE aviary_id = aviary_id
  load events = interaction_events
                WHERE account_id = aviary.account_id
                AND id > tick_state.event_hwm
                ORDER BY occurred_at ASC

  for each bird:
    1. compute_mood_transition(bird, local_time_of_day, recent_events, weather_state)
    2. compute_personality_delta(bird, events_since_hwm)
    3. apply_delta(bird.pv_*, delta)  -- additive; clamped to [0,1]
    4. update bird.perch_zone based on new boldness + mood
    5. maybe_generate_notebook_entry(aviary, birds, events_since_hwm)

  update tick_state:
    last_ticked_at = now
    event_hwm = max(events.id)
    next_tick_at = now + 60s

  commit
```

### Drift function

```
delta(trait) = learning_rate * sum(weighted_events_contribution(trait, events))
```

Where:
- `learning_rate` ≈ 0.00015 per tick (chosen so that a week of regular sessions = ~10 ticks/day × 7 days × 0.00015 = ~0.01 visible change per trait; user perceives change after ~3 weeks at ~0.03)
- `weighted_events_contribution` maps event types to trait weights:

| Event type       | boldness | social_warmth | vocal_freq | plumage_sat | curiosity |
|-----------------|----------|---------------|------------|-------------|-----------|
| presence_ping   | +0.3     | +0.2          | +0.2       | +0.5        | +0.1      |
| listen_in_end   | 0        | +0.8          | +0.8       | +0.2        | 0         |
| offer_seed (accepted) | +0.2 | 0           | 0          | 0           | +0.8      |
| offer_song (accepted) | 0   | +0.4          | +0.6       | 0           | +0.3      |
| offer_pool (accepted) | +0.1 | 0            | 0          | +0.3        | +0.5      |
| settle          | 0        | 0             | 0          | 0           | 0         |

- Weights are multiplied by `presence_seconds_this_tick / 60.0` for presence pings (normalizing to per-minute units)
- **Monotonic constraint**: deltas are always non-negative. Traits never decrease. If no positive events occurred, delta = 0.
- Clamp after apply: `clamp(pv + delta, 0.0, 1.0)`

Calibration target: instruments detect change after ~1 week of regular visits; users feel change after ~3 weeks. The learning_rate constant is the primary calibration knob; it must be tunable via environment config without code deploy.

### Mood transition function

Mood is an enumerated state: `wary | content | curious | drowsy | alert`

Mood transitions are driven by a weighted probability table:

```
transition_weights(bird, context):
  base = personality_based_prior(bird.pv_*)
    // high boldness reduces wary probability
    // high vocal_frequency increases alert/curious
  tod_adjustment = time_of_day_weights(local_hour)
    // 5-8am: alert↑, drowsy↓
    // 8am-5pm: content↑
    // 5-8pm: drowsy↑
    // 8pm-5am: drowsy↑↑, wary↑ (except nightjar-type species)
  event_adjustment = recent_event_nudge(events_last_30min)
    // accepted offer → content↑
    // alarm call from nearby bird → wary↑
    // listen_in → curious↑
  weather_adjustment:
    // rain active → drowsy↑, alert↓
    // wind → alert↑, wary↑

  return normalize(base + tod_adjustment + event_adjustment + weather_adjustment)
```

Mood transitions once per tick using the probability table. A bird in `wary` has a low probability of transitioning directly to `alert` (non-adjacent transitions have reduced probability). The exact transition matrix is an implementation constant.

Mood persists to the `birds` row on each tick. The client never computes mood transitions.

### Ambient events

Weather state is generated by the simulation service, not the client. Each aviary has a weather generator that, a few times per week, schedules a `rain` or `wind` event of short duration. The weather state is included in the snapshot response. The client renders weather from the snapshot; it does not independently generate weather.

Ambient events (a passing leaf, a nightjar calling at midnight) are client-side rendering ornaments only — they do not exist in server state and do not affect the simulation.

### Notebook generation

The notebook generator runs opportunistically during or after each tick. Entry generation criteria:
- Greeter order changed relative to previous sessions (e.g., Pip greeted first for the first time this week)
- A bird's mood has been stable for an unusual duration
- A notable perch-zone change (a wary bird came to the front perch)
- A long quiet stretch (no calls from any bird for an extended tick period)
- A bird accepted an unusual offer given its current mood

Entries are rare by design: target roughly one entry per 2–4 days per active aviary. A generator with too many triggers must be throttled by a minimum inter-entry interval (configurable, default 36h). The generator writes directly to `notebook_entries`; no entry can reference trait values or event counts numerically — only behavioral observations in naturalist prose.

---

## 6. Sync Model

### Canonical state propagation

The aviary has one canonical record in the Aviary DB. All devices read from this record. No device ever owns or modifies personality state.

**Flow for a multi-device user:**
1. Laptop opens tab → pulls `/aviary/snapshot` → renders state as of tick N
2. User interacts → events written to event log
3. Simulation tick runs → canonical state advances to tick N+1
4. Phone opens tab → pulls `/aviary/snapshot` → renders state as of tick N+1

Both devices see the same state because there is only one state to see.

### Conflict prevention (not conflict resolution)

The architecture avoids the need for conflict resolution by design:
- Clients never write personality state (no conflict possible on personality)
- Event log is append-only (no conflict possible on events)
- Two devices writing events simultaneously is safe: both events are appended, the tick reads both

The only mutable client-write path is the event log, which is append-only and idempotent by `(account_id, occurred_at, event_type, bird_id)`.

### Snapshot caching

Snapshots are cacheable at the CDN edge for short TTL (10s). This means two devices may see up to 10 seconds of divergence in displayed state — acceptable given the slow tick cadence. On tab focus change, the client always issues a fresh snapshot request (bypassing cache with `Cache-Control: no-cache`).

### Handling stale state on reconnect

When the client detects it has been disconnected (tab hidden, laptop suspended), on reconnect it issues a full snapshot pull (not delta) to avoid stale interpolation state. This is the primary reconnect path; the delta endpoint is for normal keepalive only.

---

## 7. Frontend Rendering Pipeline

### Tech choices (defensible defaults, implementation team owns final call)
- **Canvas 2D** for the aviary scene: gives full control over compositing order, parallax, and mood-dependent color grading without fighting CSS layout. Bird sprites can be SVG-based or procedurally generated via canvas.
- **Web Workers**: Render Worker owns the canvas context (OffscreenCanvas); main thread handles network and bridges state.
- **React** (or equivalent) for top-bar chrome and modal surfaces only — not for the aviary scene itself.

### Scene composition layers (back to front)
1. Sky gradient (day/night driven by local time, updated each animation frame)
2. Background foliage (static or slow parallax, mood-independent)
3. Mid-ground perch branches (three zones: front/middle/back)
4. Birds (per-bird render with mood-shaped idle motion)
5. Foreground branch/leaf pass-through elements (rare, slow drift)
6. Weather overlay (rain particles or leaf-rustle wind sprites, when active)
7. Top bar chrome (CSS layer above canvas)

### First-frame strategy (achieving "already in motion")

The client must show birds mid-action within 500ms of navigation. Strategy:
1. State snapshot is embedded in the HTML response at edge (server-side render of initial JSON into `<script type="application/json">`) — eliminates the additional round-trip for the first snapshot.
2. Client initializes the Render Worker with the embedded snapshot before DOMContentLoaded.
3. Render Worker begins the animation loop immediately with birds at their snapshot positions.
4. Birds are placed mid-animation using the `call_state.timing_offset_ms` from the snapshot: the procedural motion clock starts at an offset so the bird is not at the beginning of a motion cycle.
5. No loading spinner anywhere in this path. If the HTML itself is slow (cold CDN miss), the loading state is a quiet field (soft sky gradient, no spinner).

### Idle micro-motion system

Each bird has an idle motion state machine driven by mood:
- `wary`: perch-retreat tendency, head-scanning at ~0.3Hz, low body movement
- `content`: preening cycles (3–8s duration), relaxed posture, occasional head bob
- `curious`: head-tilt toward sounds at ~0.5Hz, body orientation toward events, forward perch lean
- `drowsy`: fluffed posture, slow eye-closing cycle, minimal scanning
- `alert`: upright posture, rapid scanning, slight forward lean

The motion state machine runs on the Render Worker using a deterministic seed derived from the bird's id and the current tick timestamp. This means two clients showing the same bird at the same tick time show the same motion pose — which is correct. Motion within a tick interpolates from the starting pose.

### Mood-to-visual mapping
- `pv_plumage_saturation` maps to a saturation multiplier on the bird's color grading (higher = richer colors). This is the only personality trait with a direct visual expression.
- `boldness` → perch zone selection (server-side, reflected in snapshot)
- `mood` → idle motion pattern, perch position lean

### Transitions
- **Perch-to-perch movement**: short flight animation (200–400ms) with arc. Never teleport.
- **Mood shift**: cross-fade between motion states (500ms). Not a visible color change — a behavioral shift the user perceives over several seconds.
- **Settle gesture**: lighting shifts from current day-state to evening over 4s. Sky gradient warms, call volume decays.
- **Listen-in engage/disengage**: audio mix changes (see Audio Pipeline). No visual change other than a subtle focus ring on the focused bird (visible, not distracting).

### Reduced-motion mode

When `prefers-reduced-motion: reduce` is detected, or when the user opts in via settings:
- Bird idle motion: cross-fades between still poses instead of animated motion frames. A "preening" bird cross-fades between two preen stills at 2s intervals.
- Flight transitions: cross-fade between source and destination perch without the arc animation.
- Ambient leaf drift: disabled.
- Day/night color transitions: preserved (they are gradual, not motion).
- Weather particles: rain becomes a static screen tint overlay; wind is suppressed.
- All WebAudio behavior: unchanged.

The Render Worker checks a `reducedMotion` flag received from the main thread on initialization and switches its rendering path accordingly. The flag is re-sent on `prefers-reduced-motion` media query change.

### Top bar behavior

The top bar is a thin fixed overlay above the canvas. It fades to ~10% opacity after 3s of cursor stillness; it returns to 100% on `pointermove` or `keydown`. The fade uses CSS `transition: opacity 600ms ease`. The top bar contains: account/settings icon, accessibility icon, notebook icon, offer affordance icon. No other elements.

### Empty aviary and loading state

- Loading state: sky gradient renders immediately (no spinner); birds appear as they arrive in the snapshot.
- New-account empty aviary: same sky gradient; first bird enters with a 1s soft fly-in to starting perch from off-screen edge.
- Once two birds are present, the user never sees an empty aviary again.

---

## 8. Audio Pipeline

### Architecture

Calls are synthesized client-side using the **Web Audio API**. The audio context is created on first user gesture (browser autoplay policy constraint). No recorded audio files are shipped.

**Render Worker owns the AudioContext.** This avoids jank from main-thread activity.

### Per-bird call grammar

Each species has a **motif library**: 3–6 short motif definitions (frequency envelope, duration, decay, harmonic ratios). A bird's call at any moment is:

```
call = select_motifs(motif_library, personality_shaped_weights)
       → sequence(motifs, inter-motif_pause)
       → pitch_shift(pv_vocal_frequency_mapping)
       → timing_jitter(±5–15% random on timing, seeded per-call)
```

- `personality_shaped_weights`: high vocal_frequency → more motifs per call, denser sequence
- `pitch_shift`: birds with higher vocal_frequency trend slightly higher; mood-based pitch variance (alert = slight pitch up, drowsy = slight pitch down)
- `timing_jitter`: per-call randomness so no two calls sound identical; seeded with `(bird_id, call_index)` for determinism across devices

Call scheduling: each bird has an independent call timer derived from `pv_vocal_frequency`. High-frequency birds call every 8–20s; low-frequency birds call every 30–90s. The timer has ±20% jitter.

### Chorus mixing

Each bird has its own `GainNode`. The master mix output feeds a compressor before the AudioContext destination to prevent clipping when multiple birds call simultaneously.

Base per-bird gain is set so that the sum of all seven birds at max vocal_frequency does not clip the compressor. In practice, not all birds will be calling simultaneously; the compressor handles the occasional overlap.

### Listen-in mix

When the user initiates listen-in on bird B:
- Bird B's GainNode ramps to 1.0 (from its current value) over 1.5s
- All other birds' GainNodes ramp to 0.15 (ambient) over 1.5s
- On disengage (click bird again, click empty space, keyboard escape): all GainNodes ramp back to their base values over 1.5s

The ramp is implemented with `AudioParam.linearRampToValueAtTime`. No hard cuts.

### Day/night volume modulation

A master day-state GainNode sits between the mix and the compressor. Its gain value reflects time of day:
- Morning: 0.9 (near full)
- Midday: 1.0
- Evening: 0.6 (quieter as settle approaches)
- Night: 0.3 (ambient only; most birds silent in their call timers)

The settle gesture triggers a ramp from current gain to 0.1 over 4s.

### Weather audio effect

During rain: a low-amplitude pink noise generator (GainNode ≈ 0.05) is added to the mix to simulate rain ambient. Individual bird gains are slightly reduced (×0.7) while rain is active. The rain node fades in and out with the weather event.

### Procedural vs. recorded: the boundary

No audio files are downloaded. All synthesis is from Web Audio oscillator nodes, filter nodes, and convolver nodes using programmatic impulse responses. The species motif library is code, not assets.

Concretely: a motif is defined as:
```js
{
  type: "oscillator",           // OscillatorNode
  frequency_hz: 2800,           // fundamental
  harmonics: [1.0, 0.4, 0.15], // amplitudes of f, 2f, 3f
  attack_ms: 20,
  sustain_ms: 80,
  release_ms: 120,
  envelope: "ASR"               // Attack-Sustain-Release
}
```

Each call instance creates a temporary subgraph, plays it, and disconnects nodes on completion (`onended` handler). No permanent per-call allocation after playback.

### WebAudio fallback

If `AudioContext` is not available or is denied:
- The bird's visual surface operates normally.
- Call captions are enabled automatically (the caption generation still runs; it is driven by the call scheduler, not the audio output).
- No recorded-audio fallback. The aviary plays in graceful silence with captions.

---

## 9. Accessibility Surfaces

### Screen-reader narration

The aviary has a visually hidden `<div role="status" aria-live="polite" aria-atomic="false">` that receives narration updates. Updates are generated on a slow cadence (30–60s at idle) and on specific events (return-greeting, offer response, settle).

**Narration generation** is a client-side prose function that reads from the current snapshot:

```js
function generateNarration(snapshot):
  // Produces naturalist prose like:
  // "a small grey bird is perched on the front rail, calling softly.
  //  another bird sits further back with feathers fluffed.
  //  it is morning in the aviary; the light is gentle."
```

The narration function must:
- Never name a trait value ("boldness is 0.6" → forbidden)
- Never use announcement framing ("pip has greeted you")
- Always be lowercase, present-tense, specific to the current snapshot state

Return-greeting narrations are priority-queued (aria-live="assertive" variant) to ensure they are read promptly at session start.

**Narration templates are not templates.** They are prose-generation functions that read current state and produce contextually specific text. A mood of `content` with species `warbler` at `front` perch mid-morning produces different prose from the same mood and species at `back` perch at evening. Prose must vary meaningfully.

### Call captions

When captions are enabled (via accessibility settings or WebAudio fallback):
- Each time a bird initiates a call, a caption appears near the bird as a small text element (absolutely positioned on the canvas, or via an overlay div)
- Caption text is generated from the call grammar at runtime:
  - Example: "a soft three-note rise", "a low trill, paused, low trill again"
  - Caption text is mood-shaped: wary bird calls are described as "a short call from the back perch"; alert bird calls as "a quick sharp sequence"
- Captions fade in with the call onset and fade out 500ms after the call ends
- Maximum two captions visible at once (to avoid visual clutter with multiple calling birds)
- Caption text uses the same naturalist voice as the notebook; never technical ("oscillator at 2800Hz")

Caption text is computed by the same call-scheduler function that produces the audio. When WebAudio is suppressed, the scheduler runs at the same cadence but produces captions only (no audio nodes).

### WCAG AA contrast

- All top-bar icons have sufficient contrast against the aviary sky palette at all day/night states (minimum 4.5:1 for text, 3:1 for icons)
- Focus indicators: 2px solid outline, offset 2px, high contrast against both bright (midday) and dark (night) aviary states. The visual designer specifies exact color per state.
- Call captions: text is rendered against a semi-transparent rounded-rect background (#000000 at 55% opacity) to ensure contrast regardless of aviary background color.
- Screen-reader narration div is visually hidden using `clip` pattern (not `display:none` or `visibility:hidden`, which prevent screen reader access).

### Keyboard navigation

Tab order:
1. Account/settings icon (top bar)
2. Accessibility icon (top bar)
3. Notebook icon (top bar)
4. Offer icon (top bar)
5. Bird 1 (front perch or ordered by perch zone)
6. Bird 2 ... Bird N

Within the aviary scene (when a bird has focus):
- Arrow keys cycle between birds
- Enter: engage listen-in on focused bird
- Escape: disengage listen-in / close any open panel

Offer panel (opened via top bar):
- Tab/arrow navigate between offer types (seed, song, pool)
- Enter confirms offer
- Escape closes panel without offering

Notebook panel:
- Scroll within notebook with arrow keys
- Escape closes notebook

All interactive elements have `tabindex` and `role` attributes appropriate to their function. The aviary canvas is not itself a focusable element; focus management is handled by overlaid interactive elements that map to bird positions.

---

## 10. Performance Budgets and Observability

### Bundle size: <2MB gzipped initial JS

Target budget allocation:
- Core framework (React or equivalent for chrome): ~100KB
- Canvas rendering engine + idle motion state machines: ~200KB
- Web Audio synthesis engine + motif library (all species): ~150KB
- Bird sprite assets (SVG, compiled): ~100KB
- Networking and state client: ~50KB
- Accessibility utilities: ~50KB
- Remaining headroom for dependencies: ~1,350KB

Code splitting:
- Account settings: lazy-loaded chunk (not in initial bundle)
- Accessibility settings: lazy-loaded chunk
- Visit invitation flow: lazy-loaded chunk
- Notebook panel: lazy-loaded chunk (content is dynamic, not bundled)

No recorded audio files. Bird sprites generated procedurally where possible; SVG definitions compiled into the JS bundle.

### Time to first bird visible: <500ms

Prerequisite conditions:
- Initial state snapshot embedded in HTML at edge (avoids second round-trip)
- Render Worker initialized before `DOMContentLoaded` using `module` worker with embedded snapshot
- First bird rendered in first animation frame after worker init
- Sky gradient renders before snapshot arrives (no blank white frame)

Measurement definition: the timestamp at which the first bird pixel appears in the viewport. Measured in synthetic monitoring with a 4G throttle profile on a mid-tier mobile device (e.g., Moto G Power class hardware).

If the embedded snapshot is stale by more than 60s (cold cache), the Render Worker renders from the stale snapshot and triggers a fresh pull in parallel. The stale render is better than a blank state.

### 60fps idle at 30 minutes on a 5-year-old laptop

- The Render Worker uses `requestAnimationFrame` on the OffscreenCanvas
- Per-frame budget: ~16ms (60fps)
- Bird rendering: each bird is a single composite draw call (not multiple overlapping drawImage calls per frame)
- Micro-motion is computed from the idle motion state machine, not re-derived from the snapshot each frame
- Ambient elements (leaf drift) are rendered as a pool of pre-allocated particle objects; no per-frame allocation

The 30-minute CI test: a headless browser runs the aviary with two birds visible for 1800s and asserts that average frame time remains under 16ms and no GC pauses exceed 50ms.

### No memory growth over 30 minutes

Memory invariants:
- Per-call Web Audio subgraph: all nodes disconnected and dereferenced on `onended`
- Notebook entries: the notebook panel renders a virtualized list; off-screen entries are not retained in the DOM
- Snapshot state: the client holds only the current snapshot and the previous snapshot (for delta interpolation); older snapshots are GC'd
- Web Worker messages: structured-clone transfer (not shared references); messages are not accumulated

CI test: Chrome DevTools Protocol memory snapshot diff between minute-1 and minute-30 of a headless session; assert heap growth <5MB.

### Error budget and alarms

| Signal | Alarm threshold | Action |
|---|---|---|
| Simulation tick p99 latency | >5s | Page on-call; investigate tick service scaling |
| Event log write failure rate | >0.5% | Page; check DB connectivity |
| Snapshot pull p95 latency | >800ms | Alert; check CDN edge and DB read replica |
| Audio context error rate (client RUM) | >2% of sessions | Alert; review WebAudio initialization path |
| First-bird-visible p90 (synthetic) | >500ms | Alert; review bundle and snapshot embed |
| Magic link delivery failure | >1% | Alert; check email provider |

### What we measure and what we deliberately don't

**Measured (aggregate, anonymized):**
- Request counts and latencies per endpoint
- Simulation tick duration (p50, p95, p99)
- Session duration histograms (binned, no per-account dimension)
- First-bird-visible timing (synthetic monitoring + RUM percentiles)
- WebAudio context error counts
- Client render frame timing (p50, p95)
- Notebook entry generation counts per day (aggregate, not per-account)

**Not measured (privacy boundary enforced at pipeline level):**
- Per-bird personality vector values (at any time, for any account)
- Per-account interaction history
- Per-account presence-time totals
- Which birds a specific user has listened in on
- Per-user notebook entries (content or count)

The telemetry pipeline is architecturally forbidden from reading the Aviary DB. Aggregate metrics are computed from the API gateway access logs and the simulation service's own instrumentation, never from the simulation database records.

---

## 11. Rollout

### Pre-launch: internal testing (2 weeks)
- All team members create accounts and use the product daily
- Validate drift calibration: instrument personality vectors directly (internal-only flag) to confirm ~0.01 delta per trait after 1 week of regular use
- Validate mood persistence: confirm no mood "snap to default" on tab open
- Validate presence accounting: confirm triple-condition signal fires correctly across browsers
- Validate first-bird-visible timing in synthetic monitoring across geographies
- Screen reader testing on VoiceOver (macOS/iOS) and NVDA (Windows)
- Reduced-motion testing across all interaction paths

### Beta launch: private invites (4 weeks)
- Accounts created by invitation only (no public sign-up)
- Ship with two birds per aviary; bird-count ramping not yet active
- Monitor: tick latency, snapshot latency, event log write volume, session duration histograms
- Watch for: audio context errors (any browser where WebAudio behaves differently), unexpected memory growth, first-bird timing regressions on real user devices
- Notebook entry quality review: human review of generated entries weekly to catch voice regressions

### V1 launch: public sign-up
- Public magic-link sign-up open
- Aviary age-based bird ramping begins: third bird offer appears at aviary age > 90 days (configurable threshold, tunable without deploy)
- Feature flag on visit-invitation feature: ships enabled (it's off by default per account; no user sees it unless they send an invite)
- Visit-notification toggle in account settings (off by default)
- Account export available day-one
- Browser support page for unsupported browsers (matter-of-fact surface)

### Bird-count ramping
Birds 3–7 become available as the aviary ages. Thresholds (tunable via config):
- Bird 3: aviary age ≥ 90 days
- Bird 4: aviary age ≥ 180 days
- Birds 5–7: aviary age ≥ 1 year

A new species offer surfaces in the account settings (not as a popup, not as a streak reward — visible only when the user navigates to account settings). The user can accept or ignore the offer; no pressure, no expiration countdown.

### Day-one instrumentation
From the first day of public launch:
- Synthetic monitoring running from 5 geographies (US East, US West, EU West, Southeast Asia, Australia)
- First-bird-visible timing RUM collecting from all sessions
- Tick latency alarm active
- Memory growth CI test in continuous deployment pipeline
- Error-rate alarms on all endpoints

---

## 12. Risks

### Risk 1: Drift calibration — too fast or too slow

**Risk**: The learning_rate constant is wrong. Too fast: users notice birds changing between sessions, which reads as a Tamagotchi counter, not a slow relationship. Too slow: three months of regular visits show no change the user can feel, and the product's "feels alive over weeks" claim fails.

**Mitigation**:
- The learning_rate is a single environment config constant, tunable without code deploy.
- Internal beta (2 weeks pre-launch) instruments personality vectors directly (internal flag, never exposed to users) to measure actual drift against the calibration target.
- Define explicit pass/fail criteria before beta: after 7 days of regular visits (defined as 5+ presence-pings/day), at least one trait per bird should show a measurable delta of ≥0.005 in instruments. After 21 days, at least one trait should show ≥0.02.
- If calibration is off, adjust learning_rate before public launch. Post-launch adjustments are safe (monotonic drift; changing the rate doesn't retroactively corrupt existing vectors).

### Risk 2: Sync correctness — event log ordering under load

**Risk**: Under high event write volume (many concurrent users), the event log's `occurred_at` ordering may have clock skew between clients (different device clocks). If the tick processes events in DB-insertion order rather than client-reported `occurred_at` order, drift inputs may be disordered.

**Mitigation**:
- The event log uses a database-assigned sequence ID (UUID v7 or auto-increment) as the authoritative ordering for tick consumption. Client-reported `occurred_at` is stored but used only for display (e.g., notebook generation context), not for tick processing order.
- The idempotency key `(account_id, occurred_at, event_type, bird_id)` uses client time, which may cause rare duplicates if clocks are wrong; the deduplication window should use DB-insertion timestamps as secondary key.
- Document explicitly: the tick processes events by DB-insertion order, not by client clock. This is correct for drift (which is cumulative and order-insensitive within a tick window) but would be wrong for mood transitions (which are order-sensitive). Mood transitions use the tick's current state only, not event ordering.

### Risk 3: Audio uncanniness — procedural calls that feel wrong

**Risk**: The procedural call grammar produces calls that sound mechanical, unnatural, or that users describe as "chirpy noise" rather than bird calls. This breaks the affective spine of the product.

**Mitigation**:
- Hire an ornithological audio consultant early to review the motif library for each species before they are implemented.
- Internal beta testing with a focus group that includes birders and non-birders; specifically ask "does this sound like a real bird?" — the target is "yes" from non-birders and "plausible" from birders.
- Reserve time in the beta period specifically for iterating on the motif library. This is not a post-launch polish task.
- The motif library is code, not binary audio assets; iteration is cheap (no re-download, no CDN invalidation).
- Identify a specific fallback position: if a species' procedural calls cannot achieve the quality bar, that species is cut from the v1 pool rather than shipped at lower quality. The pool is small by design; one species cut is survivable.
- The WebAudio fallback (silence + captions) must be tested to confirm it does not produce audio artifacts on browsers where the AudioContext initializes partially.

### Risk 4: Accessibility regression — narration quality degradation

**Risk**: The screen-reader narration, which must be naturalist prose, is hard to maintain as the codebase evolves. A code change that touches mood states, species names, or perch zones may produce awkward or generic narration that fails the "feels alive" bar for screen-reader users.

**Mitigation**:
- The narration generation function has unit tests that assert prose quality properties: no sentence may contain a trait value, no sentence may use announcement framing, all outputs must be valid naturalist-voice prose (validated against a small rubric in test code).
- The narration function is a bounded module; its internal prose templates (which are not literal templates — they are conditional prose generation) are reviewed in code review the same way the notebook generation code is reviewed.
- Accessibility testing is on the v1 launch checklist, not the post-launch backlog. A screen-reader tester reviews the narration from a fresh account through the first two sessions before any public launch.

### Risk 5: Presence accounting drift — presence signal corruption

**Risk**: A subtle bug in the triple-condition presence check causes over-reporting of presence. For example, `visibilityState` is checked correctly but the activity-recency check is too generous, or the activity window resets on scroll events that shouldn't count. Over-reported presence inflates drift across all accounts, causing birds to change faster than the calibration intends.

**Mitigation**:
- The presence accounting code is isolated in a single module with comprehensive unit tests covering: tab in background (no presence), tab visible but no focus (no presence), tab visible and focused but no activity in the last N minutes (no presence), all three conditions satisfied (presence).
- The activity-recency window (target: a few minutes) is set to the conservative end during beta; it can be widened if users report feeling that presence is "lost too quickly."
- Internal beta uses the direct personality-vector instrumentation to monitor drift rates; if drift is unexpectedly high across a cohort, the presence accounting code is the first suspect.

### Risk 6: Tick scheduling failure — aviary states frozen

**Risk**: The tick scheduler fails silently for a subset of aviaries (e.g., a DB partition fails, a worker crashes on specific payload shapes). Affected aviaries stop evolving, but users see a rendered aviary that looks normal (same mood indefinitely, no drift). Users may not report this; the failure is invisible.

**Mitigation**:
- The `tick_states` table is the heartbeat record. A monitoring query alerts if any aviary's `last_ticked_at` is more than 10 minutes behind `next_tick_at`. This query runs every 2 minutes in the monitoring service.
- Tick workers emit a structured log on each successful tick; the absence of logs for an aviary_id is detectable.
- Tick workers must be designed to be re-runnable from the `event_hwm` checkpointed on the `tick_states` row; a failed tick leaves no partial state (the write is transactional).

### Risk 7: Gamification creep — "just one small feature"

**Risk**: Under shipping pressure or user request, a small "harmless" gamification element slips into the product — a visit counter, a quiet streak display, an "achievement unlocked" for the first offer accepted.

**Mitigation**: This is a process risk, not a technical one. Mitigations:
- The non-goals list is checked explicitly in every feature review.
- Any feature that touches visit frequency, interaction counts, streak-adjacent metrics, or user-behavior-observation (as opposed to bird-behavior-observation) requires escalation to the product lead.
- The PLAN.md and non_goals.md are referenced in the engineering onboarding so new team members understand the reason for the rule, not just the rule.

---

*This plan covers every required area from the PRD. Ambiguities have been resolved with defensible calls noted inline. The implementation team owns final selection of specific frameworks, exact color values, and performance constants within the bounds named here.*
