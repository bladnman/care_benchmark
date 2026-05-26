# PLAN.md — Pocket Aviary v1 Implementation Plan

## Table of Contents

1. [Scope](#1-scope)
2. [Architecture](#2-architecture)
3. [Data Model](#3-data-model)
4. [API Surface](#4-api-surface)
5. [Simulation Engine Design](#5-simulation-engine-design)
6. [Sync Model](#6-sync-model)
7. [Frontend Rendering Pipeline](#7-frontend-rendering-pipeline)
8. [Audio Pipeline](#8-audio-pipeline)
9. [Accessibility Surfaces](#9-accessibility-surfaces)
10. [Performance Budgets and Observability](#10-performance-budgets-and-observability)
11. [Rollout](#11-rollout)
12. [Risks](#12-risks)
13. [Appendices](#13-appendices)

---

## 1. Scope

### 1.1 In Scope for v1

| Domain | Deliverables |
|---|---|
| **Core aviary** | Single horizontal scene, 2 starter birds (cap 7), 3 perch zones, day/night cycle, ambient weather, ambient micro-motion |
| **Bird engine** | Personality vector (5 traits), monotonic drift, mood state machine, procedural call grammar, idle motion system, bird-to-bird interaction, stable identity |
| **Interactions** | Return-greeting, listen-in, offer (seed/song/pool), settle, field notebook, presence accounting |
| **Accounts** | Magic-link email auth, synthetic UUID account ID, per-device session tokens, account export (JSON), soft-then-hard deletion (30-day window) |
| **Sync** | Server-side simulation tick (~1 min cadence), snapshot-based client consumption, multi-device coherence via single canonical record |
| **Social** | Visit invitations (off by default, per-invite opt-in, revocable, 30-day expiry), read-only ambient visitor view, visit log, no co-presence |
| **Accessibility** | Screen-reader narration (naturalist prose), reduced-motion mode (designed cross-fade surface), call captions, WCAG AA contrast, full keyboard navigation |
| **Performance** | <2MB initial JS bundle (gzipped), <500ms time-to-first-bird, 60fps idle motion on 5-year-old laptop, no memory growth over 30 min |
| **Adoption** | System-selected 2 starter birds, user-assigned names, rename at any time |
| **Bird growth** | New bird offers at age-based intervals (months → third bird, year → five or six) |

### 1.2 Out of Scope (Explicit Non-Goals)

- **Native apps** — web-only at v1; no iOS, no Android, no design accommodation for native constraints
- **Gamification** — no achievements, streaks, levels, scores, badges, green-dot calendars, XP, ranks, tiers, or any engagement counter; this is absolute and non-negotiable
- **Tamagotchi mechanics** — birds do not die, get hungry, show distress, or have decaying happiness meters; neglect produces ambient quietness, not visible suffering
- **Social network surfaces** — no profiles, follows, public feeds, discovery, friend-of-friend chains, comments, leaderboards, or "show-off" mode
- **Payments/billing** — no monetization surface at v1
- **Shared aviaries** — one account, one aviary, one user
- **Multi-aviary accounts** — single aviary per account
- **Push notifications** — no push, ping, or email about aviary state
- **Customizable scenes** — no user-controlled layout, panning, scrolling, or zooming
- **Recorded audio** — all calls are procedural; no recorded audio fallback path

### 1.3 Design Decisions Requiring Engineering Judgment

Where the PRD leaves calibration details to implementation:

| Decision | PRD Guidance | Engineering Call |
|---|---|---|
| Presence activity window | "a few minutes, leaning longer" | Start at 3 minutes; calibrate via instrumented beta cohort |
| Personality vector range | "scalar, normalized to a small range" | Use [0.0, 1.0] per trait; seed new birds at species-default ± 0.1 random |
| Tick cadence | "~once per minute" | 60-second tick; configurable per deployment |
| Mood enum set | "wary, content, curious, drowsy, alert; exact set finalized in implementation" | 6 states: `wary`, `content`, `curious`, `drowsy`, `alert`, `settled` |
| Offer cooldown | "a few minutes" | 5-minute per-bird cooldown |
| Notebook entry frequency | "roughly one entry every few days" | Probabilistic: ~0.3 entries per day for regular visitors, with event-triggered entries for noteworthy moments |
| Species pool size | "about six species" | Exactly 6 species at v1 |
| Third bird timing | "a few months" | 90-day aviary age for third bird offer; subsequent birds at 180, 270, 365 days |

---

## 2. Architecture

### 2.1 Service Topology

```
┌─────────────────────────────────────────────────────────┐
│                     CDN Edge                             │
│  (static assets, initial state snapshot inlined in HTML) │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────────────────▼─────────────────────────────────┐
│                   API Gateway                            │
│  Auth verification, rate limiting, request routing       │
└──┬──────────┬──────────┬──────────┬─────────────────────┘
   │          │          │          │
   ▼          ▼          ▼          ▼
┌──────┐ ┌────────┐ ┌────────┐ ┌──────────┐
│ Auth │ │Aviary  │ │ Social │ │ Account  │
│Svc   │ │API Svc │ │ Svc    │ │ Svc      │
└──┬───┘ └───┬────┘ └───┬────┘ └────┬─────┘
   │         │          │           │
   ▼         ▼          ▼           ▼
┌──────────────────────────────────────────────────────────┐
│                  Simulation Service                       │
│  Tick loop, drift computation, mood transitions,         │
│  call scheduling, notebook entry generation              │
└──────────────────────┬───────────────────────────────────┘
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
    ┌──────────┐ ┌──────────┐ ┌──────────┐
    │ Primary  │ │ Event    │ │ Session  │
    │ DB       │ │ Log      │ │ Store    │
    │(Postgres)│ │(append   │ │(Redis)   │
    │          │ │ only)    │ │          │
    └──────────┘ └──────────┘ └──────────┘
```

### 2.2 Service Descriptions

**Auth Service**
- Issues magic links via email provider (transactional email)
- Validates magic links, issues per-device session tokens (JWT, short-lived access + long-lived refresh)
- Manages session revocation
- Rate-limits magic link requests per email

**Aviary API Service**
- Serves state snapshots (current aviary state for a given account)
- Accepts interaction events (offer, listen-in start/end, settle, presence pings) and writes them to the append-only event log
- Serves field notebook entries (paginated, newest-first)
- Handles snapshot caching at the edge for fast initial load

**Simulation Service**
- Runs the tick loop: reads unprocessed events from the event log, computes drift deltas, transitions moods, schedules calls, generates notebook entries, writes updated canonical state
- Runs whether or not any client is connected
- One simulation worker per account shard; horizontally scalable by account UUID partition

**Social Service**
- Manages visit invitations (create, revoke, expire)
- Serves read-only state snapshots for visitors (same snapshot endpoint, filtered to exclude interaction capabilities)
- Maintains visit log
- Handles invite expiration (30-day TTL, background cleanup job)

**Account Service**
- Account CRUD (email change with verification, deletion with 30-day soft-delete window)
- Account export generation (JSON snapshot, emailed as download link)
- Account settings (visit notification toggle, accessibility preferences)

### 2.3 Client/Server Split

| Responsibility | Client | Server |
|---|---|---|
| Rendering (visual scene, animations) | ✅ | ❌ |
| Procedural audio synthesis | ✅ | ❌ |
| Presence detection (visibility + focus + activity) | ✅ | ❌ |
| Interaction event capture | ✅ (emit) | ✅ (consume) |
| Personality vector storage & mutation | ❌ | ✅ (sole writer) |
| Mood state transitions | ❌ (interpolate) | ✅ (authoritative) |
| Drift computation | ❌ | ✅ |
| Call scheduling (when a bird should call next) | ❌ (timing hint in snapshot) | ✅ |
| Notebook entry generation | ❌ | ✅ |
| Day/night cycle computation | ✅ (from local time) | ✅ (in snapshot for consistency) |
| Snapshot interpolation | ✅ | ❌ |
| State snapshot serving | ❌ | ✅ |

The client is a **renderer and event emitter**. The server is the **sole state authority**. This split is non-negotiable — it is what makes multi-device sync correct and prevents personality vector corruption.

### 2.4 Render Pipeline Boundary

The render pipeline consumes state snapshots and produces visual + audio output. It never writes to the simulation state. The boundary is:

```
Snapshot (server) → State Interpolator → Scene Graph → Renderer (Canvas/WebGL)
                                          ↘ Audio Scheduler → WebAudio Engine
```

The interpolator smooths between discrete snapshot states. The scene graph maps bird states to visual representations. The renderer draws frames. The audio scheduler uses call-timing hints from the snapshot to trigger procedural synthesis.

---

## 3. Data Model

### 3.1 Core Entities

#### Account

```
Account {
  id: UUID (synthetic, generated at creation — never derived from email)
  email_encrypted: bytea (AES-256-GCM, key from KMS)
  email_hash: bytea (SHA-256, for uniqueness check without decrypting)
  created_at: timestamp
  deleted_at: timestamp? (soft-delete marker; null = active)
  hard_delete_scheduled_at: timestamp? (30 days after soft-delete)
  settings: JSONB {
    visit_notifications_enabled: boolean (default: false)
    reduced_motion: enum('system' | 'on' | 'off') (default: 'system')
    captions_enabled: boolean (default: false)
    audio_enabled: boolean (default: true)
  }
}
```

#### Bird

```
Bird {
  id: UUID (stable internal identifier, never changes)
  account_id: UUID → Account.id
  species_id: enum (one of 6 species)
  name: string (user-assigned, renameable)
  adopted_at: timestamp
  personality_vector: {
    boldness: float [0.0, 1.0]
    social_warmth: float [0.0, 1.0]
    vocal_frequency: float [0.0, 1.0]
    plumage_saturation: float [0.0, 1.0]
    curiosity: float [0.0, 1.0]
  }
  current_mood: enum ('wary' | 'content' | 'curious' | 'drowsy' | 'alert' | 'settled')
  current_perch: enum ('front' | 'middle' | 'back')
  last_mood_transition_at: timestamp
  last_interaction_at: timestamp?
  offer_cooldown_until: timestamp?
  call_signature_seed: uint32 (deterministic seed for procedural call grammar)
}
```

#### Personality Vector Drift Record

```
DriftRecord {
  id: UUID
  bird_id: UUID → Bird.id
  tick_id: UUID → SimulationTick.id
  delta: {
    boldness: float
    social_warmth: float
    vocal_frequency: float
    plumage_saturation: float
    curiosity: float
  }
  input_summary: JSONB (what events contributed to this delta)
  created_at: timestamp
}
```

Stored separately from the bird record to maintain a full drift audit trail. The bird's `personality_vector` is the running sum of all deltas applied to the seed values.

#### Interaction Event (Append-Only Log)

```
InteractionEvent {
  id: UUID
  account_id: UUID → Account.id
  bird_id: UUID? (null for aviary-wide events like settle)
  event_type: enum (
    'presence_ping' |
    'listen_in_start' |
    'listen_in_end' |
    'offer_seed' |
    'offer_song' |
    'offer_pool' |
    'settle' |
    'unsettle' |
    'tab_visible' |
    'tab_hidden' |
    'session_start' |
    'session_end'
  )
  payload: JSONB (event-specific data, e.g., listen_in duration, offer target bird)
  client_timestamp: timestamp
  server_received_at: timestamp
  processed_by_tick: UUID? → SimulationTick.id (null until consumed)
}
```

#### Simulation Tick

```
SimulationTick {
  id: UUID
  account_id: UUID → Account.id
  tick_number: int64 (monotonically increasing per account)
  events_consumed: UUID[] (interaction event IDs processed in this tick)
  state_snapshot: JSONB (full aviary state after this tick)
  drift_deltas: UUID[] → DriftRecord.id
  mood_transitions: JSONB[] ({bird_id, from_mood, to_mood, reason})
  notebook_entries_generated: UUID[] → NotebookEntry.id
  executed_at: timestamp
  duration_ms: int
}
```

#### Notebook Entry

```
NotebookEntry {
  id: UUID
  account_id: UUID → Account.id
  prose: text (naturalist voice, lowercase, present-tense)
  trigger: enum ('periodic' | 'greeting_event' | 'offer_event' | 'drift_milestone' | 'weather_event' | 'bird_to_bird_event')
  related_bird_ids: UUID[]
  created_at: timestamp
}
```

#### Presence Session

```
PresenceSession {
  id: UUID
  account_id: UUID → Account.id
  started_at: timestamp
  ended_at: timestamp?
  presence_time_seconds: float (accumulated valid presence time)
  is_active: boolean
  last_activity_at: timestamp (last pointermove/keypress)
  last_visibility_check: { visible: boolean, focused: boolean, at: timestamp }
}
```

This is a client-side record that gets flushed to the event log as `presence_ping` events at regular intervals (every 30 seconds of valid presence). The server aggregates these into presence-time for drift computation.

#### Visit Invitation

```
VisitInvitation {
  id: UUID
  host_account_id: UUID → Account.id
  visitor_email_encrypted: bytea
  visitor_email_hash: bytea
  token: string (one-time use, URL-safe)
  status: enum ('pending' | 'used' | 'revoked' | 'expired')
  created_at: timestamp
  expires_at: timestamp (created_at + 30 days)
  used_at: timestamp?
  used_by_account_id: UUID? (if visitor has an account)
}
```

#### Visit Log Entry

```
VisitLogEntry {
  id: UUID
  host_account_id: UUID → Account.id
  invitation_id: UUID → VisitInvitation.id
  visitor_email_encrypted: bytea
  visit_started_at: timestamp
  visit_ended_at: timestamp?
  duration_seconds: int?
}
```

#### Session Token

```
SessionToken {
  id: UUID
  account_id: UUID → Account.id
  device_fingerprint: string (browser + OS heuristic, not PII)
  issued_at: timestamp
  expires_at: timestamp
  revoked_at: timestamp?
  last_used_at: timestamp
}
```

### 3.2 Species Definition (Static Configuration)

```
Species {
  id: enum ('warbler' | 'finch' | 'wren' | 'sparrow' | 'thrush' | 'nightjar')
  display_name: string
  silhouette_asset: path (SVG)
  default_plumage_palette: { primary: hex, secondary: hex, accent: hex }
  call_grammar_motifs: MotifLibrary (see Audio Pipeline §8)
  default_personality: {
    boldness: float
    social_warmth: float
    vocal_frequency: float
    plumage_saturation: float
    curiosity: float
  }
  nocturnal: boolean (only nightjar = true)
}
```

### 3.3 Indexes and Partitioning

- **Account table**: primary key on `id`; unique index on `email_hash`
- **Bird table**: primary key on `id`; index on `account_id`; unique constraint on `(account_id, id)`
- **InteractionEvent**: partitioned by `account_id` (hash partition, 16 partitions); index on `(account_id, processed_by_tick)` for tick consumption queries; append-only, no updates
- **SimulationTick**: partitioned by `account_id`; index on `(account_id, tick_number)` for sequential access
- **NotebookEntry**: index on `(account_id, created_at DESC)` for paginated retrieval
- **VisitInvitation**: index on `token` (unique) for lookup; index on `(host_account_id, status)` for management
- **DriftRecord**: index on `bird_id` for audit trail queries

All tables use the synthetic account UUID. Email is never used as a key, partition value, or log field.

---

## 4. API Surface

### 4.1 Authentication Endpoints

| Method | Path | Description |
|---|---|---|
| POST | `/auth/magic-link` | Request a magic link; body: `{email}`. Rate-limited per email. |
| POST | `/auth/verify` | Verify magic link token; returns session tokens. Body: `{token}` |
| POST | `/auth/refresh` | Refresh access token using refresh token |
| DELETE | `/auth/sessions/:id` | Revoke a specific session |
| GET | `/auth/sessions` | List active sessions for the account |

### 4.2 Aviary State Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/aviary/snapshot` | Current aviary state snapshot. Returns full state: birds (positions, moods, personality-derived rendering hints), day/night phase, weather, active transitions. Response is <4KB. |
| GET | `/aviary/snapshot?since=:tick` | Delta snapshot: only changes since the given tick number. Falls back to full snapshot if delta is too large. |

**Snapshot Response Shape:**

```json
{
  "tick_number": 14523,
  "server_time": "2026-05-26T14:30:00Z",
  "day_phase": "afternoon",
  "day_phase_progress": 0.62,
  "weather": { "type": "clear", "event": null },
  "birds": [
    {
      "id": "uuid",
      "name": "Pip",
      "species": "warbler",
      "mood": "content",
      "perch": "front",
      "idle_state": "preening",
      "idle_state_progress": 0.4,
      "call_hint": {
        "next_call_at": "2026-05-26T14:30:45Z",
        "motif_suggestion": "rising_three_note",
        "intensity": 0.7
      },
      "render_hints": {
        "plumage_intensity": 0.78,
        "posture": "relaxed",
        "orientation": "facing_right"
      }
    }
  ],
  "ambient": {
    "leaf_drift_active": true,
    "foreground_elements": [...]
  },
  "settled": false
}
```

Note: `render_hints` and `call_hint` are derived from personality but never expose raw trait values. The client uses them for rendering decisions without ever knowing the underlying numbers.

### 4.3 Interaction Event Endpoints

| Method | Path | Description |
|---|---|---|
| POST | `/aviary/events` | Submit an interaction event. Body: `{event_type, bird_id?, payload?, client_timestamp}`. Server stamps `server_received_at`. Returns `{event_id}`. |
| POST | `/aviary/events/batch` | Batch submission for presence pings. Body: `{events: [...]}`. Used for flushing accumulated presence pings. |

Events are fire-and-forget from the client perspective. The client does not wait for simulation results. The next snapshot pull will reflect processed events.

### 4.4 Field Notebook Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/aviary/notebook` | Paginated notebook entries. Query: `?before=:timestamp&limit=20`. Returns entries in reverse chronological order. |

### 4.5 Account Endpoints

| Method | Path | Description |
|---|---|---|
| GET | `/account` | Current account info and settings |
| PATCH | `/account/settings` | Update settings (reduced_motion, captions, audio, visit notifications) |
| POST | `/account/email-change` | Initiate email change; sends verification to new address |
| POST | `/account/email-change/verify` | Complete email change with verification token |
| POST | `/account/export` | Request JSON export; emailed as download link |
| POST | `/account/delete` | Initiate soft deletion |
| POST | `/account/recover` | Cancel pending deletion (within 30-day window) |

### 4.6 Social Endpoints

| Method | Path | Description |
|---|---|---|
| POST | `/social/invitations` | Create a visit invitation. Body: `{visitor_email}`. Returns invitation details (token included for host to share). |
| GET | `/social/invitations` | List outstanding invitations |
| DELETE | `/social/invitations/:id` | Revoke an invitation |
| GET | `/social/visits` | Visit log (paginated) |
| GET | `/visit/:token` | Visitor endpoint: serves read-only snapshot of host's aviary. No auth required (token is the credential). |

### 4.7 Adoption Endpoints

| Method | Path | Description |
|---|---|---|
| POST | `/aviary/adopt/accept` | Accept a new bird offer (when one is available based on aviary age). Body: `{name}`. |
| GET | `/aviary/adopt/available` | Check if a new bird offer is available |
| PATCH | `/aviary/birds/:id/name` | Rename a bird. Body: `{name}`. |

### 4.8 Initial State Delivery

For time-to-first-bird performance, the initial state snapshot is inlined in the HTML response:

```
GET / → HTML with embedded <script>window.__AVIARY_STATE__ = {...}</script>
```

The server renders the HTML shell with the current snapshot serialized inline. The client hydrates from this embedded state without a separate API call. Subsequent snapshot pulls use the API endpoint.

---

## 5. Simulation Engine Design

### 5.1 Tick Loop

The simulation service runs a continuous tick loop per account shard:

```
loop every 60 seconds:
  for each active account in shard:
    1. Fetch unprocessed interaction events (processed_by_tick IS NULL)
    2. Compute presence-time delta since last tick
    3. Compute drift deltas from events + presence-time
    4. Apply drift deltas to personality vectors (additive, monotonic toward expressive)
    5. Evaluate mood transitions for each bird
    6. Schedule next calls based on vocal_frequency and mood
    7. Evaluate notebook entry generation (probabilistic)
    8. Write updated canonical state
    9. Write SimulationTick record
    10. Mark consumed events with tick_id
```

### 5.2 Drift Function

The drift function is a low-pass filter over presence-and-interaction signals. Implementation:

```
drift_delta(trait, events, presence_time) = 
  base_rate * presence_weight(trait) * presence_time_normalized
  + interaction_weight(trait, events)
  + bird_to_bird_weight(trait, aviary_context)
```

**Constants (calibration targets):**

| Parameter | Value | Rationale |
|---|---|---|
| `base_rate` | 0.002 per hour of presence | Yields ~0.014/week at 1hr/day; visible after ~3 weeks (0.06 cumulative) |
| `presence_weight(boldness)` | 0.8 | Presence is the dominant driver |
| `presence_weight(social_warmth)` | 0.7 | |
| `presence_weight(vocal_frequency)` | 0.6 | |
| `presence_weight(plumage_saturation)` | 1.0 | Plumage drifts fastest with attention |
| `presence_weight(curiosity)` | 0.5 | |

**Interaction modifiers:**

| Event | Trait affected | Delta modifier |
|---|---|---|
| `listen_in_start` (per minute of duration) | social_warmth, vocal_frequency | +0.001 each |
| `offer_seed` (accepted) | curiosity | +0.003 |
| `offer_seed` (any, near bird) | boldness | +0.001 |
| `offer_song` | vocal_frequency | +0.002 |
| `offer_pool` | curiosity | +0.002 |

**Monotonicity enforcement:**

```
new_value = current_value + max(0, drift_delta)
new_value = min(new_value, 1.0)  // cap at upper bound
```

Drift deltas are clamped to non-negative before application. Traits never decrease. This is the implementation of "monotonic toward expressive."

**Calibration verification:**

- Automated test: simulate 7 days of 1-hour-daily presence; assert each trait has moved ≥ 0.01 from seed
- Automated test: simulate 21 days of 1-hour-daily presence; assert at least one trait has moved ≥ 0.05 (user-visible threshold)
- Automated test: simulate 30 days of zero presence; assert no trait has decreased

### 5.3 Mood State Machine

Mood is a per-bird enumerated state with transitions governed by a weighted evaluation:

```
evaluate_mood(bird, context):
  weights = {}
  
  // Time-of-day influence
  weights[drowsy]  += time_weight(hour, 'evening') * 0.3
  weights[alert]   += time_weight(hour, 'morning') * 0.3
  weights[content] += time_weight(hour, 'midday') * 0.2
  
  // Recent interaction influence
  if last_interaction was offer_accepted within 5 min:
    weights[content] += 0.4
    weights[curious] += 0.2
  if last_interaction was listen_in within 5 min:
    weights[content] += 0.3
  
  // Ambient event influence
  if weather == rain:
    weights[drowsy] += 0.2
    weights[wary]   += 0.1
  if nearby_bird_mood == wary (within last 2 min):
    weights[wary] += 0.2  // mood contagion
  
  // Personality influence
  weights[wary]    *= (1.0 - bird.boldness * 0.5)
  weights[curious] *= (0.5 + bird.curiosity * 0.5)
  weights[content] *= (0.5 + bird.social_warmth * 0.5)
  
  // Select highest-weighted mood; retain current if no strong signal
  new_mood = argmax(weights) if max(weights) > 0.4 else bird.current_mood
  
  // Settled overrides during night or after settle gesture
  if context.settled or (context.hour >= 23 and bird.species != nightjar):
    new_mood = settled
  
  return new_mood
```

Mood transitions are evaluated every tick. The transition is not instantaneous in the client — the snapshot includes the new mood, and the client cross-fades the bird's idle animation to the new mood's motion set over ~2 seconds.

### 5.4 Call Grammar Runtime

Each bird's call is generated from a procedural grammar:

```
CallGrammar {
  motifs: Motif[]           // species-specific library (~8-12 motifs per species)
  timing_params: {
    base_interval: float    // seconds between calls (modified by vocal_frequency)
    jitter: float           // random variation on interval
    chorus_response_delay: float  // delay before responding to another bird's call
  }
  pitch_params: {
    base_pitch: float       // Hz, species-specific
    pitch_range: float      // variation range
    pitch_drift: float      // how much pitch shifts with personality drift
  }
}
```

**Motif structure:**

```
Motif {
  name: string
  notes: Note[]
  variation_rules: {
    pitch_shift_range: float    // ± semitones
    timing_stretch_range: float // ± percentage
    ornament_probability: float // chance of adding grace notes
  }
}

Note {
  frequency: float     // relative to base_pitch
  duration: float      // seconds
  envelope: ADSR       // attack, decay, sustain, release
  waveform: enum       // sine, triangle, noise_burst, fm_pair
}
```

**Call scheduling (server-side):**

The server computes `next_call_at` for each bird and includes it in the snapshot. The client uses this as a scheduling hint, not a hard deadline. The actual synthesis happens client-side.

```
next_call_interval = base_interval 
  * (1.0 - vocal_frequency * 0.6)     // higher vocal_frequency = more frequent calls
  * mood_modifier(current_mood)        // wary = 1.5x interval, content = 0.8x
  * time_of_day_modifier(hour)         // night = 2.0x (except nightjar)
  * random_jitter(jitter)
```

**Chorus mechanic:**

When bird A calls and bird B has high vocal_frequency and social_warmth, bird B may respond within `chorus_response_delay` seconds. The server flags this in the snapshot as a `chorus_hint`. The client schedules B's response call with a motif that is harmonically related to A's motif (same key, complementary interval).

### 5.5 Bird-to-Bird Interaction

Evaluated each tick:

1. **Mood contagion**: If any bird transitioned to `wary` in the last 2 ticks, adjacent birds (same or neighboring perch) have an elevated probability of transitioning to `wary`
2. **Chorus emergence**: If 2+ birds with vocal_frequency > 0.6 are in `content` or `alert` mood, flag a chorus event hint in the snapshot
3. **Call response**: When a bird calls, birds with high social_warmth have a probability of calling back within the next tick

### 5.6 Notebook Entry Generation

The notebook entry generator runs probabilistically each tick:

```
entry_probability = base_rate (0.3/day ÷ 1440 ticks/day ≈ 0.0002 per tick)
  + event_bonus(noteworthy_events_this_tick)  // greeting first, chorus, drift milestone
  - recent_entry_penalty(entries_last_24h)     // suppress if entries already generated recently
```

When an entry is generated, the server composes naturalist prose from a template library parameterized by:
- Current bird states and positions
- Recent interaction events
- Time of day
- Weather state
- Drift milestones (e.g., "boldness crossed 0.7 for the first time")

Template examples:
- `"{bird_a.name} greeted before {bird_b.name} today, {first_time_qualifier}."`
- `"{bird.name} is {mood_posture} on the {perch} perch, {ambient_detail}."`
- `"a long stretch of quiet {time_of_day}. {bird.name} {idle_activity} without looking up."`

The template library must be curated to produce prose that reads as naturalist observation, not event logging. Each template is reviewed for voice compliance (lowercase, present-tense, specific, no gamification language, no user-behavior observations).

---

## 6. Sync Model

### 6.1 Single Canonical Record

The sync model is architecturally simple because it is designed to be simple:

1. The server maintains one canonical aviary state per account, stored in the primary database
2. The simulation service is the **sole writer** of personality vectors and mood states
3. Clients are **read-only consumers** that submit interaction events to an append-only log
4. Multi-device coherence is a natural consequence: all devices read the same record

There is no client-to-client sync, no CRDT, no operational transform, no last-write-wins arbitration on personality state.

### 6.2 Snapshot Pull Triggers

The client pulls a fresh snapshot on:

| Trigger | Rationale |
|---|---|
| Initial page load | Bootstrap state (inlined in HTML for performance) |
| `visibilitychange` to `visible` | Tab was hidden, state may have advanced |
| Window focus regained | Window was unfocused, state may have advanced |
| Keepalive timer (every 30 seconds while visible) | Stay current with server ticks |
| After submitting settle/unsettle | Confirm state change reflected |
| Render-frame gap > 5 seconds detected | Laptop may have been suspended |

### 6.3 Snapshot Interpolation

The client maintains two state buffers: `current` and `next`. When a new snapshot arrives:

1. `current` = previous `next`
2. `next` = new snapshot
3. Over the interpolation window (time between snapshots, typically ~30-60s), the renderer linearly interpolates bird positions, idle state progress, and ambient elements

For discrete state changes (mood transitions, perch changes), the client cross-fades over 2 seconds rather than interpolating linearly.

### 6.4 Event Submission

Clients submit events via POST to the event endpoint. Events are:

- **Idempotent**: Each event has a client-generated UUID; the server deduplicates on `(account_id, event.id)`
- **Timestamped**: Both client and server timestamps are recorded; the server uses client timestamp for ordering within a batch, server timestamp for processing cadence
- **Append-only**: Events are never modified or deleted; the `processed_by_tick` field is the only mutation (set once when consumed)

### 6.5 Conflict Prevention

The architecture prevents conflicts by design:

- **Personality vectors**: Only the simulation tick writes them. No client can submit absolute values. Deltas are computed server-side from the event log.
- **Mood states**: Only the simulation tick transitions them. Clients render what the snapshot says.
- **Interaction events**: Append-only log; no conflicts possible (events are additive facts, not competing mutations).
- **Account settings**: Standard last-write-wins with optimistic concurrency (version field). Settings are low-stakes and rarely change simultaneously from two devices.

### 6.6 Offline / Reconnection Behavior

If the client loses connectivity:

1. Continue rendering from the last known snapshot (birds stay in their last known state)
2. Queue interaction events locally
3. On reconnection: flush queued events, pull a fresh snapshot
4. The simulation has been ticking in the background; the fresh snapshot reflects the current state
5. The client interpolates from the stale state to the fresh state over ~5 seconds

The user sees a brief period where the aviary appears "frozen" in its last known state, then catches up. No error surface is shown unless reconnection fails for > 30 seconds, at which point a matter-of-fact message appears: "Connection lost. The aviary will catch up when you're back online."

---

## 7. Frontend Rendering Pipeline

### 7.1 Technology Choice

- **Rendering**: HTML5 Canvas with a lightweight scene graph library (custom, ~50KB). Not WebGL — the visual complexity does not require it, and Canvas 2D is simpler to maintain and more broadly supported.
- **Framework**: Lightweight reactive framework for UI chrome (top bar, notebook panel, settings). The aviary scene itself is imperative Canvas rendering, not framework-managed.
- **Asset strategy**: Bird sprites are procedurally generated SVGs with plumage-saturation-driven color parameters. Background elements are small SVGs. No raster images in the critical path.

### 7.2 Scene Composition

```
Scene Graph:
├── Background Layer
│   ├── Sky gradient (day/night interpolated)
│   ├── Background foliage (static SVG, subtle parallax)
│   └── Weather effects (rain particles, wind ripples)
├── Middle Layer (birds and perches)
│   ├── Back perch + bird(s)
│   ├── Middle perch + bird(s)
│   └── Front perch + bird(s)
├── Foreground Layer
│   ├── Foreground branch/leaf (subtle parallax)
│   └── Ambient drift particles (leaves, feathers)
└── UI Overlay
    ├── Top bar (fades on cursor stillness)
    ├── Notebook panel (slide-in overlay)
    └── Offer panel (slide-in overlay)
```

### 7.3 Day/Night Cycle

The client computes the day/night phase from the user's local time:

```
hour = local_time.hour + local_time.minute / 60

day_phase:
  5.0 - 7.0:   dawn     (sky warms from dark blue to soft gold)
  7.0 - 11.0:  morning   (gentle light, cool-to-warm transition)
  11.0 - 15.0: midday    (brightest palette)
  15.0 - 18.0: afternoon (warm hues deepening)
  18.0 - 20.0: evening   (warm amber, calls quieting)
  20.0 - 22.0: dusk      (deepening blue-purple)
  22.0 - 5.0:  night     (dark, most birds settled, nightjar active)
```

Sky gradient colors are interpolated continuously (not stepped). The palette transitions are smooth enough that the user never notices a discrete change.

### 7.4 Idle Micro-Motion System

Each bird has a set of idle motion states keyed to mood:

| Mood | Idle States | Transition Cadence |
|---|---|---|
| content | preening, sitting relaxed, slow head turn | Every 8-15 seconds |
| curious | head tilt, scanning, watching leaves | Every 5-10 seconds |
| wary | scanning rapidly, sitting back, flinching at sounds | Every 4-8 seconds |
| drowsy | sitting low, fluffed feathers, slow eye close | Every 15-25 seconds |
| alert | upright posture, quick head turns, listening | Every 5-10 seconds |
| settled | low on perch, eyes closed, minimal motion | Every 30-60 seconds |

Each idle state is a short animation (2-5 seconds) that loops with variation. Transitions between idle states are cross-faded over 0.5 seconds. The motion is never perfectly periodic — timing jitter and variation in animation parameters prevent the eye from detecting a loop.

### 7.5 Return-Greeting Implementation

On session start (first snapshot load after absence):

1. Determine absence duration from `last_interaction_at` in the snapshot
2. Select greeting bird: highest `boldness * social_warmth` among birds not in `settled` mood
3. Select greeting type based on absence and mood:

| Absence | Mood | Greeting |
|---|---|---|
| < 1 hour | any | Glance up from current idle state |
| 1-24 hours | content/curious | Head tilt + quiet call |
| 1-24 hours | wary/drowsy | Look up, no call |
| > 24 hours | content | Step toward front perch + longer call |
| > 24 hours | curious | Head tilt + call + another bird responds |
| > 24 hours | wary | Glance, then look away |

4. The greeting animation is procedurally varied: timing, pitch, and motion parameters are randomized within species-specific ranges
5. If multiple birds would greet, stagger by 1-3 seconds (randomized)

### 7.6 Listen-In Rendering

When listen-in is engaged:

1. **Visual**: Focused bird gets a subtle highlight (soft glow, not a border). Other birds continue their normal animations but at slightly reduced visual emphasis (5% opacity reduction on non-focused birds — barely perceptible, just enough to guide the eye).
2. **Audio**: Focused bird's call mix rises to 0dB; other birds drop to -12dB. Transition is a 2-second linear ramp.
3. **Disengage**: Reverse the ramp over 2 seconds. Visual highlight fades.

### 7.7 Offer Interaction Rendering

When the user triggers an offer:

1. Offer panel slides in from the top bar (3 items: seed, song fragment, still pool)
2. User selects an offer type
3. A small visual element appears in the scene (seed on the front perch, pool reflection on the ground, musical notes drifting)
4. The nearest non-settled bird reacts based on mood and curiosity:
   - Curious + content: approaches within 1-2 seconds
   - Curious + wary: approaches slowly over 5-8 seconds
   - Drowsy: may not approach; small head turn toward the offer
   - Alert: quick glance, then approach if curious enough
5. The reaction animation is 3-5 seconds, then the offer element fades

### 7.8 Settle Rendering

1. User triggers settle from top bar
2. Over 3 seconds: sky gradient shifts to evening/night palette, ambient light dims
3. Bird calls fade to silence over 2 seconds
4. Birds transition to `settled` idle state over 3-5 seconds
5. 5-second undo window: any click reverses the transition
6. Settled state persists until tab close or user re-engages (click/keypress)

### 7.9 Reduced-Motion Mode

When `prefers-reduced-motion` is set or user opts in:

- Idle animations are replaced with still poses that cross-fade every 8-15 seconds
- Bird transitions between perches are cross-fades (1.5s) instead of animated paths
- Ambient leaf/feather drift is disabled
- Day/night transitions are slowed to 10-second cross-fades
- Weather effects are static (rain shown as a subtle overlay, not animated particles)
- Call audio plays normally; captions remain available
- All interactive transitions (listen-in, offer, settle) use cross-fades instead of motion

### 7.10 Loading State

- **Fast path** (< 500ms): HTML arrives with inlined snapshot; Canvas renders first frame immediately. No loading state visible.
- **Slow path** (> 500ms): A quiet field is shown — soft sky color, perhaps one or two faint ambient motions (a distant leaf drift). No spinner. The field transitions seamlessly to the full aviary when the snapshot arrives.
- **Error path**: If the snapshot fails to load after 10 seconds, show a matter-of-fact message: "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."

### 7.11 Responsive Layout

| Viewport | Behavior |
|---|---|
| ≥ 1200px wide | Full scene, perches spaced generously |
| 768-1199px | Scene compresses horizontally, perch spacing reduced |
| < 768px (phone) | Scene compresses to fit; birds remain visible and uncropped; top bar icons become touch-friendly (44px targets) |
| Height < 500px | Scene scales proportionally; top bar overlays rather than consuming vertical space |

The scene never crops a bird out of frame. Aspect ratio is maintained by adjusting inter-perch spacing, not by scaling the entire scene uniformly.

---

## 8. Audio Pipeline

### 8.1 Architecture

```
AudioScheduler (receives call_hints from snapshots)
  │
  ├── CallSynthesizer (WebAudio-based procedural synthesis)
  │     ├── OscillatorBank (sine, triangle, FM pair generators)
  │     ├── NoiseGenerator (for breathy/whispered call components)
  │     ├── EnvelopeShaper (ADSR per note)
  │     └── MotifSequencer (plays motifs from species library with variation)
  │
  ├── ChorusMixer
  │     ├── Per-bird gain nodes (for listen-in mix control)
  │     ├── Master gain
  │     └── Spatial panner (per-bird stereo position based on perch)
  │
  └── AmbientLayer
        ├── Background ambient (soft wind, distant leaves)
        └── Weather audio (rain, wind — subtle, loop-free procedural)
```

### 8.2 Procedural Call Synthesis

Each species has a motif library of 8-12 motifs. Each motif is a sequence of notes with variation rules. At synthesis time:

1. Select a motif (weighted by species, mood, and call context — greeting vs. idle vs. chorus response)
2. Apply variation:
   - Pitch shift: ± 0-2 semitones (random per synthesis)
   - Timing stretch: ± 10% on note durations
   - Ornamentation: 20% chance of adding a grace note between two notes
   - Envelope variation: ± 15% on attack/release times
3. Synthesize via WebAudio oscillators with ADSR envelopes
4. Route through per-bird gain node and spatial panner

**Waveform palette:**

| Waveform | Use |
|---|---|
| Sine | Pure tones, whistles |
| Triangle | Softer, breathier tones |
| FM pair (carrier + modulator) | Complex, warble-like calls |
| Filtered noise | Breathy components, whisper calls |

### 8.3 Chorus Mixing

When multiple birds call simultaneously:

1. Each bird's call is synthesized independently on its own audio graph
2. Per-bird gain nodes control individual levels
3. Spatial panning places each bird in the stereo field based on perch position (back = center-narrow, front = wider)
4. The master mix applies a gentle limiter to prevent clipping

**Listen-in mix:**

- Default: all birds at -6dB, ambient at -18dB
- Listen-in engaged: focused bird at 0dB, others at -12dB, ambient at -18dB
- Transition: 2-second linear gain ramp on engage and disengage

### 8.4 Call Scheduling

The server provides `call_hint.next_call_at` in the snapshot. The client:

1. On snapshot receipt, schedules the next call for each bird using `setTimeout` or `requestAnimationFrame` timing
2. At the scheduled time, synthesizes and plays the call
3. After the call plays, computes the next call time locally using the same interval formula the server uses (from personality-derived `render_hints`), adding jitter
4. On the next snapshot, reconciles with the server's `next_call_at` to stay in sync

This hybrid approach ensures calls play with low latency (not waiting for the next snapshot) while staying synchronized with the server's state over time.

### 8.5 WebAudio Fallback

If WebAudio is unavailable:

1. Detect on initialization: attempt to create an `AudioContext`; if it fails, set `audio_available = false`
2. When audio is unavailable:
   - Enable captions by default (override user setting)
   - Render the aviary in visual-only mode
   - No error surface; the captions provide the audio information
3. If audio becomes available mid-session (e.g., user grants permission), seamlessly enable synthesis

### 8.6 Audio Budget

- Audio synthesis code: ~80KB (minified, before gzip)
- Motif library data (JSON): ~15KB per species × 6 species = ~90KB
- Total audio pipeline: ~170KB of the 2MB bundle budget
- No audio files are downloaded; all sounds are synthesized at runtime

---

## 9. Accessibility Surfaces

### 9.1 Screen-Reader Narration

**Implementation:**

A dedicated ARIA live region (`role="log"`, `aria-live="polite"`, `aria-atomic="false"`) receives narration updates. The narration engine:

1. Runs on a 30-60 second cadence at idle
2. Generates naturalist prose from the current aviary state
3. Prioritizes user-initiated events (greeting, offer reaction) for immediate narration
4. Uses the same voice as the field notebook: lowercase, present-tense, specific

**Narration template examples:**

```
Idle: "a small {species} is perched on the {perch_position} rail, {idle_activity}. 
       another bird sits {relative_position} with {posture_detail}. 
       it is {time_of_day} in the aviary; the light is {light_description}."

Greeting: "{bird_name} notices you — {greeting_description}."

Offer reaction: "{bird_name} {approach_description} the {offer_type}."

Settle: "the aviary settles into evening. calls grow quiet."
```

**Narration queue management:**

- Maximum 3 items in the ARIA live region queue at any time
- New idle narration replaces a pending idle narration (no stacking)
- User-initiated events preempt idle narration
- If the queue is full, the oldest non-priority item is dropped

### 9.2 Reduced-Motion Mode

Detection: `window.matchMedia('(prefers-reduced-motion: reduce)')` + user setting override.

**Rendering changes:**

| Element | Full Motion | Reduced Motion |
|---|---|---|
| Bird idle animation | Frame-by-frame sprite animation | Cross-fade between still poses (1.5s) |
| Bird perch transition | Animated flight/hop path | Cross-fade between perch positions (1.5s) |
| Greeting animation | Procedural motion sequence | Single pose with subtle scale pulse |
| Ambient leaves/feathers | Animated drift through frame | Disabled |
| Day/night cycle | Continuous gradient interpolation | 10-second cross-fade at phase boundaries |
| Weather (rain) | Animated particle system | Static semi-transparent overlay |
| Offer element appearance | Animated entry | Fade-in (1s) |
| Settle transition | 3-second animated lighting shift | 5-second cross-fade |

**What remains unchanged:** Audio, captions, narration, field notebook, all interactive functionality. The reduced-motion mode is a visual rendering change only.

### 9.3 Call Captions

When captions are enabled:

1. Each call synthesis event generates a prose description from the call grammar:
   - Motif name → prose mapping: `rising_three_note` → "a soft three-note rise"
   - Mood modifier: wary → "tentative"; content → "warm"; curious → "bright"
   - Composed caption: `"{mood_modifier} {motif_prose}"` → "a warm, soft three-note rise"
2. Caption appears as small text near the calling bird, fading in over 0.5s and out over 1s after the call ends
3. Caption text uses the naturalist voice
4. In reduced-motion mode, captions fade in/out without positional animation

### 9.4 Keyboard Navigation

**Focus order:**

```
Top bar: [Settings] [Accessibility] [Notebook] [Offer] [Settle]
  ↓ (Tab into aviary)
Bird 1 (front perch) → Bird 2 → Bird 3 → ... (ordered by perch, front to back)
  ↓ (Tab out of aviary)
Notebook entries (if panel open)
```

**Key bindings:**

| Key | Context | Action |
|---|---|---|
| Tab | Global | Move focus through interactive elements |
| Enter | Bird focused | Toggle listen-in on focused bird |
| Escape | Listen-in active | Disengage listen-in |
| Arrow Left/Right | Bird focused | Move focus to adjacent bird |
| Arrow Up/Down | Bird focused | Move focus to bird on different perch level |
| Space | Bird focused | Same as Enter (listen-in toggle) |
| O | Global (aviary focused) | Open offer panel |
| N | Global (aviary focused) | Open notebook |
| S | Global (aviary focused) | Trigger settle |

**Focus indicators:**

- Focused bird: soft, high-contrast outline (2px, white with dark shadow) visible against both bright and dim aviary states
- Focused top bar item: underline indicator
- Focus ring is always visible when keyboard navigation is active; hidden during mouse use (`:focus-visible`)

### 9.5 Contrast and Text

- All text on the top bar: minimum 4.5:1 contrast ratio against the sky gradient (tested at all day/night phases)
- Captions: white text with semi-transparent dark background pill; 4.5:1 minimum
- Notebook panel: dark text on light background; 7:1 ratio
- Settings and account surfaces: standard high-contrast text
- Error surfaces: high-contrast text with clear visual hierarchy

---

## 10. Performance Budgets and Observability

### 10.1 Bundle Budget

| Component | Budget (gzipped) | Notes |
|---|---|---|
| Core framework + runtime | 150KB | Lightweight reactive framework |
| Rendering engine (Canvas scene graph) | 80KB | Custom, minimal |
| Audio pipeline (synthesis + motifs) | 170KB | WebAudio synthesis + species motif data |
| Bird assets (SVGs, procedurally parameterized) | 60KB | 6 species × ~10KB each |
| Background/ambient assets | 40KB | SVGs + procedural ambient |
| Accessibility (narration engine, caption system) | 30KB | |
| UI chrome (top bar, notebook, settings, offer panel) | 100KB | Code-split: settings and social loaded on demand |
| API client + event submission | 20KB | |
| State interpolation + snapshot management | 15KB | |
| **Subtotal (critical path)** | **665KB** | |
| Code-split: account settings | 40KB | Loaded on demand |
| Code-split: accessibility settings | 25KB | Loaded on demand |
| Code-split: visit invitation flow | 30KB | Loaded on demand |
| Code-split: adoption flow | 20KB | Loaded on demand |
| **Total initial bundle** | **~665KB gzipped** | Well under 2MB cap |

### 10.2 Time-to-First-Bird Budget

| Phase | Budget | How |
|---|---|---|
| DNS + TLS + TTFB | 150ms | CDN edge serving; HTML is pre-rendered with snapshot |
| HTML download | 50ms | HTML is ~15KB (includes inlined snapshot) |
| HTML parse + script execution | 100ms | Critical JS is minimal; non-critical code deferred |
| Canvas initialization + first frame render | 150ms | Scene graph constructs from inlined state |
| **Total** | **450ms** | Under 500ms target on mid-tier mobile over 4G |

**Critical path optimization:**

1. HTML response includes inlined state snapshot (no separate API call for initial load)
2. Critical JS (renderer + scene graph) is in the initial bundle; everything else is deferred
3. Bird SVGs are inlined in the JS bundle (no separate asset requests)
4. Audio initialization is deferred until after first visual frame (audio starts ~200ms after first bird visible)
5. Non-critical code (notebook, settings, social) is loaded asynchronously after first paint

### 10.3 Runtime Performance Budget

| Metric | Target | Measurement |
|---|---|---|
| Frame rate (idle motion) | 60fps sustained | `requestAnimationFrame` timing; alarm if < 55fps for > 5s |
| Frame time p99 | < 16.6ms | Per-frame timing in render loop |
| Memory (initial) | < 50MB heap | Chrome DevTools measurement |
| Memory growth (30 min) | 0MB net growth | CI test: 30-minute automated session, assert heap delta < 2MB |
| Audio context count | ≤ 1 per page | Single shared AudioContext |
| Audio buffer allocation | Zero per-call | Pre-allocated buffer pool, reused |
| DOM nodes | < 200 | Canvas-based rendering keeps DOM minimal |
| Snapshot parse time | < 5ms | Snapshot is < 4KB JSON |

### 10.4 Simulation Tick Performance

| Metric | Target | Alarm |
|---|---|---|
| Tick execution time p50 | < 50ms | |
| Tick execution time p99 | < 500ms | |
| Tick execution time p99.9 | < 2s | |
| Tick execution time max | < 5s | Alarm at p99 > 5s |
| Tick cadence accuracy | ± 5s of 60s target | Alarm if > 10% of ticks are > 70s apart |
| Event log consumption lag | < 2 ticks behind | Alarm if unprocessed events > 2 minutes old |

### 10.5 Observability

**Aggregate-only metrics (no per-account, per-bird data):**

| Category | Metrics |
|---|---|
| **Client performance** | First-bird-render timing (histogram), frame rate (p50/p99), audio context errors (count), snapshot parse time |
| **Server performance** | Tick latency (p50/p99/p99.9), snapshot response time, event ingestion rate, API error rates |
| **Operational health** | Request counts, 5xx rates, database query latencies, email delivery success rate |
| **Session metrics** | Session duration histogram (anonymized, no per-account dimension), daily active accounts (count only) |

**Explicitly NOT measured:**

- Per-bird personality values or drift rates
- Per-account interaction patterns
- Which birds users listen in to or offer to
- Notebook entry content
- Visit patterns between specific accounts

**Monitoring stack:**

- Synthetic checks: automated browser fleet running the aviary from 5 geographies every 15 minutes
- Real User Monitoring (RUM): aggregate-only page load and render timings
- Server-side: standard APM (distributed tracing, latency histograms, error rates)
- Alerting: PagerDuty-style escalation for p99 tick latency > 5s, snapshot response time > 1s, or 5xx rate > 1%

---

## 11. Rollout

### 11.1 Phase Plan

**Phase 0: Internal Alpha (Weeks 1-4)**
- Deploy to internal team (10-20 accounts)
- Validate core loop: tick → drift → mood → rendering → audio
- Calibrate drift function against the 1-week/3-week targets
- Identify and fix rendering/audio bugs
- Verify multi-device sync correctness

**Phase 1: Closed Beta (Weeks 5-10)**
- Invite 200-500 users
- Ramp birds-per-aviary: start all accounts at 2 birds; no third-bird offers during beta
- Instrument drift calibration at scale: are birds drifting at the expected rate across the population?
- Monitor performance budgets against real user data
- Collect qualitative feedback on "feels alive" perception
- Tune notebook entry frequency and prose quality

**Phase 2: Open Beta (Weeks 11-16)**
- Open registration
- Enable third-bird offers for accounts reaching 90-day age
- Enable visit invitations
- Monitor infrastructure scaling
- Continue drift calibration; adjust `base_rate` if population-level drift is too fast or too slow

**Phase 3: General Availability (Week 17+)**
- Remove beta labeling
- Full bird count ramp (up to 7, at age-based intervals)
- Ongoing monitoring and calibration

### 11.2 Birds-Per-Aviary Ramp

| Aviary Age | Max Birds | Offer Timing |
|---|---|---|
| 0 days | 2 | Initial adoption |
| 90 days | 3 | Third bird offer appears |
| 180 days | 4 | Fourth bird offer |
| 270 days | 5 | Fifth bird offer |
| 365 days | 6 | Sixth bird offer |
| 450 days | 7 | Seventh bird offer (cap) |

Each offer is a gentle, non-intrusive notification in the aviary: a small naturalist-voice message in the notebook ("a new species has been spotted near the aviary") with an accept affordance. The user can accept at any time; the offer does not expire.

### 11.3 Day-One Instrumentation

From the first internal alpha user, measure:

1. **Drift calibration**: per-tick drift deltas, cumulative drift per bird per week. Target: 0.01-0.02 per week per trait at 1hr/day presence.
2. **Performance budgets**: all metrics in §10, with alerting active
3. **Mood transition frequency**: how often do moods change? Target: 2-5 transitions per bird per hour of active simulation.
4. **Notebook entry quality**: sample 50 entries per week for voice compliance review
5. **Audio health**: WebAudio context creation success rate, synthesis error count
6. **Sync correctness**: snapshot consistency checks (compare client-rendered state against server state at random intervals)
7. **Presence accuracy**: compare client-reported presence pings against server-side session data to detect presence inflation

### 11.4 Feature Flags

| Flag | Default (GA) | Purpose |
|---|---|---|
| `social.visits_enabled` | true | Kill switch for visit feature |
| `adoption.third_bird_enabled` | true | Ramp control for new bird offers |
| `audio.procedural_enabled` | true | Fallback to silence+captions if audio pipeline has systemic issues |
| `accessibility.reduced_motion_forced` | false | Emergency override to force reduced motion for all users |
| `simulation.tick_enabled` | true | Emergency pause for simulation service |
| `notebook.generation_enabled` | true | Pause notebook entry generation without affecting simulation |

---

## 12. Risks

### 12.1 Drift Calibration Risk

**Risk**: The drift function is too fast (birds change visibly between sessions) or too slow (users never notice change).

**Likelihood**: Medium. The calibration targets are narrow and depend on real user behavior patterns that are hard to simulate.

**Mitigation**:
- Instrument drift at the per-tick level from day one
- Run automated calibration tests with simulated presence patterns before each phase
- The `base_rate` constant is server-configurable without deployment; adjust in real-time during beta
- Monitor population-level drift distributions weekly; flag accounts drifting > 2σ from mean
- If drift is too fast: reduce `base_rate`. If too slow: increase `base_rate`. The monotonicity constraint means we can always slow drift without corrupting existing state.

**Contingency**: If calibration proves fundamentally unstable (e.g., high-variance user behavior makes a single `base_rate` inadequate), implement per-account adaptive calibration that adjusts `base_rate` based on observed presence patterns. This is a significant complexity increase and should be avoided if possible.

### 12.2 Sync Correctness Risk

**Risk**: A bug in the simulation tick causes personality vector corruption (e.g., a negative delta slips through, or a tick processes events out of order).

**Likelihood**: Low (architecture is simple), but impact is catastrophic (the bird the user knows is gone).

**Mitigation**:
- Every drift delta is recorded in the `DriftRecord` table with full audit trail
- Personality vector is reconstructed as seed + sum(all deltas); the running value is cross-checked against this reconstruction periodically
- Automated test: for every account, assert `current_vector == seed + sum(deltas)` daily
- The append-only event log means any corruption can be replayed and corrected
- Simulation tick is single-writer per account shard; no concurrent writes to the same bird's personality

**Contingency**: If corruption is detected, the audit trail allows reconstruction. Notify affected users in matter-of-fact voice; do not attempt to silently repair.

### 12.3 Audio Uncanniness Risk

**Risk**: Procedural calls sound mechanical, repetitive, or unpleasant. Users perceive them as "synthesizer noises" rather than bird-like calls.

**Likelihood**: Medium-high. Procedural audio is hard to make natural-sounding, and the uncanny valley for audio is real.

**Mitigation**:
- Invest significant design time in the motif library; each species' motifs should be based on real bird call structures (interval patterns, rhythmic structures)
- Variation rules must produce enough diversity that the same motif is not recognizable across repetitions
- Internal alpha should include dedicated audio listening sessions with the team
- Beta user feedback should specifically ask about audio quality
- The call grammar should be designed by someone with musical training, not just engineering

**Contingency**: If procedural calls cannot be made pleasant-sounding, fall back to a hybrid approach: use short recorded samples as the base waveform (a few hundred milliseconds each) and apply procedural variation on top (pitch shift, time stretch, reordering). This preserves the "never identical twice" property while grounding the sound in natural audio. This increases the bundle size but stays within the 2MB budget.

### 12.4 Accessibility Regression Risk

**Risk**: Accessibility surfaces (narration, reduced-motion, captions) are treated as secondary and ship late, incomplete, or broken.

**Likelihood**: Medium. Accessibility work is often deprioritized under schedule pressure.

**Mitigation**:
- Accessibility is in the critical path from day one, not a post-launch addition
- Automated tests for: ARIA live region content, keyboard navigation completeness, reduced-motion rendering, caption generation
- Include screen-reader users in the closed beta cohort
- The narration engine shares code with the notebook entry generator; this creates a natural incentive to keep both high-quality
- Reduced-motion mode is implemented as a rendering flag that propagates through the entire scene graph; it cannot be "partially" implemented

**Contingency**: If narration quality is insufficient at launch, ship with a simpler narration system (state descriptions rather than prose) and iterate. This is a degraded experience but not a broken one.

### 12.5 "Feels Alive" Perception Risk

**Risk**: Despite correct implementation of all mechanics, users perceive the birds as robotic or canned. The "feels alive" principle is met technically but not affectively.

**Likelihood**: Medium. This is the hardest risk to mitigate because it is subjective and emergent.

**Mitigation**:
- The return-greeting must be genuinely varied; invest in procedural variation depth
- Idle motion must never show a detectable loop; use long-period noise functions for timing
- Calls must never repeat identically; the variation rules must be deep enough that a user listening for an hour cannot identify a repeated call
- The first-frame-already-running conceit must be flawless; any loading artifact breaks the spell
- Internal alpha should include "Turing test" sessions: team members watch the aviary for 10 minutes and note any moment that felt mechanical

**Contingency**: If specific elements feel canned, increase procedural variation depth for those elements. If the overall experience feels flat, the issue is likely in the call grammar or idle motion — prioritize those systems for rework.

### 12.6 Privacy Boundary Violation Risk

**Risk**: An engineer inadvertently logs per-bird interaction data to the aggregate telemetry pipeline, or a new analytics feature reads from the simulation database.

**Likelihood**: Low-Medium. The boundary is architectural and must be maintained across every future change.

**Mitigation**:
- The simulation database and the analytics pipeline use different database instances with no shared credentials
- Telemetry emission code is in a separate module with a strict allowlist of fields; per-bird fields are not in the allowlist
- Code review checklist includes "does this change cross the privacy boundary?"
- Automated test: assert that no telemetry event contains a `bird_id`, `personality_vector`, `mood`, or `notebook_entry` field

### 12.7 Gamification Creep Risk

**Risk**: A well-meaning contributor adds a "harmless" engagement feature (visit counter, streak, milestone celebration) that initiates the gamification slide.

**Likelihood**: Medium-High over the product's lifetime. The temptation is constant and the argument is always reasonable-looking.

**Mitigation**:
- This plan documents the absolute refusal of gamification as a load-bearing design decision, not a preference
- Code review checklist includes "does this change introduce any engagement counter, streak, badge, or milestone surface?"
- The architecture does not compute the underlying metrics for leaderboards or streaks; there is no "just expose it" path
- The field notebook explicitly cannot write observations about user behavior (only aviary observations)

---

## 13. Appendices

### Appendix A: Species Pool (v1)

| Species | Silhouette | Default Palette | Call Character | Nocturnal |
|---|---|---|---|---|
| Warbler | Small, slender | Olive-green, yellow breast | Rapid, melodic rising phrases | No |
| Finch | Compact, conical bill | Warm brown, rust accents | Short, bright intervals | No |
| Wren | Tiny, upright tail | Brown, barred wings | Loud, complex trills for its size | No |
| Sparrow | Medium, rounded | Grey-brown, subtle markings | Simple, cheerful chirps | No |
| Thrush | Medium-large, spotted breast | Brown, spotted ochre breast | Flutelike, varied phrases | No |
| Nightjar | Medium, wide bill | Mottled grey-brown, cryptic | Low, rhythmic churring | Yes |

### Appendix B: Mood → Idle Motion Mapping

| Mood | Primary Idle | Secondary Idle | Perch Preference |
|---|---|---|---|
| Content | Preening | Sitting relaxed, slow head turn | Front or middle |
| Curious | Head tilt, scanning | Watching leaves, investigating | Front |
| Wary | Rapid scanning | Sitting back, flinching | Back |
| Drowsy | Sitting low, fluffed | Slow eye close | Middle or back |
| Alert | Upright, quick turns | Listening posture | Middle or front |
| Settled | Low on perch, eyes closed | Minimal breathing motion | Any (stays put) |

### Appendix C: Voice and Tone Reference

**Naturalist voice** (product surfaces: aviary, notebook, narration, captions, offer prompts):
- Lowercase by default
- Present-tense
- Specific to the bird and the moment
- Bird-related verbs preferred (notice, perch, settle, listen in, offer)
- No exclamation marks
- No "you"
- No announcement framing

**Matter-of-fact voice** (system surfaces: sign-in, errors, account settings, accessibility settings, sync conflicts):
- Standard capitalization
- Direct and clear
- No naturalist phrasing
- No warmth pretending to be useful
- Tells the user what happened and what to do

### Appendix D: Engineering Milestones

| Milestone | Target | Dependencies |
|---|---|---|
| M1: Simulation tick + drift function working | Week 3 | Database schema, event log |
| M2: Rendering pipeline (Canvas scene, bird sprites, day/night) | Week 4 | Species assets, snapshot API |
| M3: Audio pipeline (procedural synthesis, motif library) | Week 6 | Species motif data, WebAudio integration |
| M4: Interactions (listen-in, offer, settle, greeting) | Week 7 | M1 + M2 + M3 |
| M5: Field notebook generation | Week 8 | M1, prose template library |
| M6: Accounts + auth (magic link, sessions) | Week 5 | Email provider integration |
| M7: Multi-device sync validation | Week 8 | M1 + M6 |
| M8: Accessibility (narration, reduced-motion, captions, keyboard) | Week 9 | M2 + M3 + M4 |
| M9: Social (visit invitations, read-only view) | Week 10 | M6 + M7 |
| M10: Performance optimization + budgets met | Week 12 | All above |
| M11: Internal alpha launch | Week 4 | M1 + M2 (minimum viable aviary) |
| M12: Closed beta launch | Week 10 | M1-M9 |
| M13: Open beta launch | Week 16 | M1-M10 + beta feedback addressed |
| M14: General availability | Week 17+ | All above |
