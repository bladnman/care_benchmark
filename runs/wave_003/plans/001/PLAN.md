# Pocket Aviary — Phase 1 Implementation Plan

**Wave:** 003  
**Run:** 001  
**Model:** qwen3-coder  
**Effort Level:** unknown  
**Harness:** opencode  

---

## 1. Scope

### V1 In Scope

- **Bird count:** 2 starter birds, max 7 per aviary
- **Account model:** Single-user accounts with email magic-link sign-in
- **Aviary:** One horizontal scene, three perch zones (front/middle/back), day/night cycle tied to user's local time
- **Bird mechanics:** Personality vectors (boldness, social warmth, vocal frequency, plumage saturation, curiosity), mood states (wary, content, curious, drowsy, alert), procedural call synthesis
- **Interactions:** Return greeting, listen-in (focus one bird), offer (seed/song/pool), settle (soft session end), field notebook
- **Simulation:** Server-side tick (~1 minute cadence), monotonic personality drift toward expressive
- **Sync:** Multi-device via server canonical state, additive deltas only
- **Social (optional):** Read-only visit invitations, opt-in per host, no co-presence
- **Accessibility:** Screen-reader narration, reduced-motion mode, call captions, keyboard navigation, WCAG AA contrast
- **Performance:** Initial bundle <2MB, time-to-first-bird <500ms, 60fps idle motion, no memory growth over 30 minutes

### V1 Out of Scope (per `non_goals.md`)

- Native mobile applications (web-only)
- Gamification: no achievements, no streaks, no levels, no scores, no badges, no XP
- Tamagotchi mechanics: no bird death, no hunger, no distress, no happiness decay
- Social network surfaces: no profiles, no follows, no public feed, no comments on visits
- Push notifications
- Payments or monetization
- Shared households or multi-user accounts

---

## 2. Architecture

### Service Shape

```
┌─────────────────────────────────────────────────────────────────────┐
│                           Client (Browser)                          │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Render Pipeline                                               │  │
│  │  - Scene composition (perch zones, day/night, weather)        │  │
│  │  - Bird animations (idle motion, transitions, procedural)     │  │
│  │  - Audio synthesis (WebAudio procedural calls, chorus mix)    │  │
│  │  - Accessibility (narration, captions, reduced-motion)        │  │
│  │  - Interaction handling (listen-in, offer, settle, presence)  │  │
│  └───────────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  State Management                                              │  │
│  │  - Snapshot caching (last N states for interpolation)         │  │
│  │  - Event queue (interaction events to server)                 │  │
│  │  - Animation interpolation between snapshots                  │  │
│  └───────────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Network Layer                                                 │  │
│  │  - GET /api/state (pull snapshots)                            │  │
│  │  - POST /api/events (append interaction events)               │  │
│  │  - GET /api/notebook (field notebook entries)                 │  │
│  │  - Magic-link auth flow                                        │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              │ HTTPS / WebSockets (optional)
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        Server (Service Layer)                       │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  State Service (Canonical State)                              │  │
│  │  - Bird records (personality vectors, moods, positions)       │  │
│  │  - Aviary metadata (age, species, names)                      │  │
│  │  - Notebook entries                                           │  │
│  │  - Visit log (host/visitor mapping)                           │  │
│  └───────────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Simulation Engine (Tick)                                     │  │
│  │  - Consumes event log per account                             │  │
│  │  - Updates personality drift (additive deltas)                │  │
│  │  - Transitions moods (time-of-day, interactions, ambient)     │  │
│  │  - Advances ambient events (weather, leaf drift)              │  │
│  │  - Writes canonical state snapshot                            │  │
│  └───────────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Event Log (Append-only)                                      │  │
│  │  - Interaction events (offer, listen-in, settle, presence)    │  │
│  │  - Timestamped, ordered per account                           │  │
│  │  - Input to simulation tick                                   │  │
│  └───────────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Auth Service                                                 │  │
│  │  - Email + magic-link generation/verification                 │  │
│  │  - Session token management                                   │  │
│  │  - Account export/deletion workflows                          │  │
│  └───────────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │  Visit Service (Optional)                                     │  │
│  │  - Invite generation/revocation                               │  │
│  │  - Visitor session validation                                 │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                         Database (PostgreSQL)                       │
│  - Accounts (synthetic UUID, encrypted email)                       │
│  - Birds (stable ID, personality vectors, moods, positions)         │
│  - Aviaries (metadata, species pool membership)                     │
│  - Event log (append-only, per-account)                             │
│  - Notebook entries (naturalist prose, timestamped)                 │
│  - Visits (invite records, visitor sessions)                        │
└─────────────────────────────────────────────────────────────────────┘
```

### Render Pipeline Boundary

The render pipeline is entirely client-side. The server provides:
- Canonical state snapshots (per-bird positions, moods, call timing)
- Notebook entries
- Aviary metadata

The client:
- Interpolates between snapshots for smooth motion
- Synthesizes procedural calls in real-time
- Generates ambient micro-motion (leaf drift, feather fall)
- Renders day/night cycle and ambient weather
- Produces screen-reader narration and call captions
- Handles reduced-motion mode rendering

**Key principle:** The client never owns state. It only renders snapshots and submits interaction events. The server is the only writer of personality vectors.

---

## 3. Data Model

### Accounts

```sql
accounts (
  id UUID PRIMARY KEY,                    -- synthetic, never email-based
  email TEXT UNIQUE ENCRYPTED,           -- encrypted at rest
  created_at TIMESTAMP,
  deleted_at TIMESTAMP NULL,             -- soft-deletion window
  last_sign_in TIMESTAMP
)
```

### Aviaries

```sql
aviaries (
  id UUID PRIMARY KEY,
  account_id UUID NOT NULL REFERENCES accounts(id),
  created_at TIMESTAMP,
  bird_count INT,                         -- current count, max 7
  species_pool_version INT               -- for future species additions
)
```

### Birds

```sql
birds (
  id UUID PRIMARY KEY,                    -- stable internal ID
  aviary_id UUID NOT NULL REFERENCES aviaries(id),
  name TEXT,                              -- user-assigned
  species_id UUID NOT NULL,               -- from species pool
  created_at TIMESTAMP,
  
  -- Personality vector (normalized 0.0–1.0, never exposed to user)
  boldness FLOAT,
  social_warmth FLOAT,
  vocal_frequency FLOAT,
  plumage_saturation FLOAT,
  curiosity FLOAT,
  
  -- Current mood (enum: wary, content, curious, drowsy, alert)
  mood TEXT NOT NULL,
  
  -- Position (enum: front, middle, back)
  perch_zone TEXT NOT NULL,
  
  -- Call timing (server state for procedural generation)
  last_call_at TIMESTAMP NULL,
  call_phase FLOAT,                       -- 0.0–1.0 within call motif
  call_tempo FLOAT                        -- per-bird tempo modifier
)
```

### Event Log (Append-only)

```sql
events (
  id UUID PRIMARY KEY,
  account_id UUID NOT NULL,
  bird_id UUID NULL,                      -- null for aviary-level events
  event_type TEXT NOT NULL,               -- offer, listen_in_start, listen_in_end, settle, presence_ping
  created_at TIMESTAMP NOT NULL,
  payload JSONB NULL                      -- event-specific data
)
```

### Notebook Entries

```sql
notebook_entries (
  id UUID PRIMARY KEY,
  aviary_id UUID NOT NULL REFERENCES aviaries(id),
  created_at TIMESTAMP NOT NULL,
  content TEXT NOT NULL                   -- naturalist prose, lowercase
)
```

### Visits (Optional Feature)

```sql
visits (
  id UUID PRIMARY KEY,
  host_account_id UUID NOT NULL,
  visitor_email TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL,
  expires_at TIMESTAMP NOT NULL,
  revoked_at TIMESTAMP NULL,
  visitor_session_id UUID NULL           -- active session if any
)
```

### Session Tokens

```sql
session_tokens (
  id UUID PRIMARY KEY,
  account_id UUID NOT NULL,
  device_info JSONB,                      -- browser, OS, etc.
  created_at TIMESTAMP NOT NULL,
  last_used_at TIMESTAMP NOT NULL,
  revoked_at TIMESTAMP NULL
)
```

---

## 4. API Surface

### Authentication

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/request-link` | Request magic link for email |
| POST | `/api/auth/verify-link` | Verify magic link, return session token |
| POST | `/api/auth/refresh` | Refresh session token |
| GET | `/api/auth/session` | Get current session info |
| DELETE | `/api/auth/session` | Revoke current session |

### State Management

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/state` | Get current canonical state snapshot |
| POST | `/api/events` | Append interaction events to event log |

**State Snapshot Response Schema:**
```json
{
  "aviary_id": "uuid",
  "timestamp": "2026-06-05T12:00:00Z",
  "local_time": "2026-06-05T12:00:00Z",
  "birds": [
    {
      "id": "uuid",
      "name": "Pip",
      "species": "warbler",
      "mood": "content",
      "perch_zone": "front",
      "call_timing": {
        "last_call_at": "2026-06-05T11:59:45Z",
        "call_phase": 0.73,
        "call_tempo": 1.2
      },
      "idle_motion_state": "preening",
      "idle_motion_phase": 0.45
    }
  ],
  "ambient": {
    "day_cycle": 0.62,                    // 0.0–1.0, 0=midnight, 0.5=noon
    "weather": "clear",                   // clear, rain, wind
    "weather_phase": 0.0                  // for rain/wind transitions
  }
}
```

### Notebook

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/notebook` | Get recent notebook entries |

### Account

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/account/export` | Request account export (JSON, emailed) |
| POST | `/api/account/delete` | Initiate soft deletion |
| POST | `/api/account/restore` | Restore soft-deleted account |
| DELETE | `/api/account` | Hard delete (after 30-day window) |

### Visits (Optional)

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/visits/invite` | Create new visit invitation |
| DELETE | `/api/visits/{id}` | Revoke visit invitation |
| GET | `/api/visits` | Get visit log (host only) |
| GET | `/api/visits/validate` | Validate visitor session |

---

## 5. Simulation Engine Design

### Tick Cadence

- **Primary tick:** ~60 seconds (calibrated during build)
- **Slow ambient tick:** ~10 minutes (for ambient events like leaf drift)

### Tick Processing Flow

1. **Read event log:** Fetch all events for accounts with pending events since last tick
2. **Process interactions:** For each account, consume events in timestamp order:
   - `offer`: Update bird curiosity (small drift up if accepted)
   - `listen_in_start`: Update bird social warmth and vocal frequency (small drift up)
   - `listen_in_end`: No drift change (just audio mix reset)
   - `settle`: End presence window cleanly, no drift change
   - `presence_ping`: Accumulate presence-time for this session
3. **Compute drift:** For each bird, apply additive personality drift:
   - `boldness`: +0.001–0.003 per presence-hour, +0.002 per offer-near
   - `social_warmth`: +0.002–0.004 per presence-hour, +0.003 per listen-in
   - `vocal_frequency`: +0.002–0.004 per presence-hour, +0.003 per listen-in
   - `plumage_saturation`: +0.001–0.002 per presence-hour
   - `curiosity`: +0.002 per offer accepted, +0.001 per presence-hour
   - **Monotonic rule:** Traits only move up, never down on neglect
4. **Update moods:** For each bird, compute new mood:
   - Time-of-day signal (drowsy near dusk, alert morning)
   - Recent interactions (offer accepted → content; ignored offer → wary)
   - Ambient events (rain → dampened vocal frequency, wary mood)
   - Personality-weighted (high-boldness bird less likely wary)
5. **Ambient events:** Randomly trigger ambient events:
   - Leaf drift (client-side, but server tracks phase for consistency)
   - Weather transitions (clear ↔ rain, clear ↔ wind)
   - Bird-to-bird interaction (one bird's call prompts another)
6. **Write state:** Update canonical state with new moods, positions, call timing
7. **Generate notebook entry:** With low probability (~5% per tick), generate a naturalist observation

### Drift Calibration

- **Target:** Measurable drift in instruments after ~1 week of regular visits
- **Target:** Visible drift to user after ~3 weeks of regular visits
- **Regular visits:** Defined as ~15 minutes per day, 5 days per week
- **Calibration method:** Simulate 30 days of regular visits, verify personality vectors shift by 0.1–0.2 per trait

### Mood Transition Matrix (Example)

| Current Mood | Input | Next Mood |
|--------------|-------|-----------|
| content | offer accepted | content (maintain) |
| content | offer ignored | wary |
| content | listen-in | content (maintain) |
| wary | presence-time accumulation | curious |
| wary | rain ambient | wary (maintain) |
| curious | low presence-time | content |
| drowsy | morning time-of-day | alert |
| alert | prolonged quiet | drowsy |

---

## 6. Sync Model

### Multi-Device Sync

**Architecture:** Single canonical state on server. Clients are read-only.

1. **Client opens tab:** Requests current state snapshot from server
2. **Server returns:** Full state snapshot with timestamp and version
3. **Client renders:** Interpolates between last known state and new snapshot
4. **Client writes events:** Appends to append-only event log (never writes state)
5. **Server processes:** Tick consumes event log, updates canonical state
6. **Next client request:** Gets updated canonical state

### Conflict Resolution

**Rule:** No last-write-wins for personality state.

- Personality drift is implemented as **additive server-authored deltas**, never as client-submitted absolute values
- Clients send `user listened in to Pip for 3 minutes`, server decides the drift delta
- Event log is append-only and consumed in timestamp order
- The simulation tick processes events in order, applying additive deltas

**Why this matters:** A laptop session in the morning writes personality update A. A phone session at lunch (which started before laptop session ended) reads old state and writes personality update B. With last-write-wins, B might overwrite A's drift. With additive deltas, both A and B's contributions are applied in event-log order.

### Presence-Time Accumulation

- Presence-events recorded only when all three conditions hold:
  1. `document.visibilityState === 'visible'`
  2. `document.hasFocus() === true`
  3. Pointer-movement or keypress in last 3 minutes
- Presence-time accumulates per session
- Session ends on: `visibilitychange` to hidden, `blur` with no recent activity, or explicit `settle` gesture
- Server-side tick consumes presence-time from event log, applies additive drift

---

## 7. Frontend Rendering Pipeline

### Scene Composition

```
┌──────────────────────────────────────────────────────────────────────┐
│                         Top Bar (Chrome)                             │
│  [Account] [Accessibility] [Notebook] [Offer] [Settle]              │
│  (Fade to transparent after 5s cursor stillness)                    │
└──────────────────────────────────────────────────────────────────────┘
┌──────────────────────────────────────────────────────────────────────┐
│                         Aviary Scene                                 │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  Background: Sky + distant foliage (parallax, slow)           │ │
│  │  Ambient: Leaf drift, feather fall (client-side ornaments)    │ │
│  │                                                                │ │
│  │  Perch Zone: Front  |  Middle  |  Back                        │ │
│  │            [Bird]   |  [Bird]   |  [Bird]                     │ │
│  │  (Birds positioned by perch_zone, mood, personality)          │ │
│  │                                                                │ │
│  │  Foreground: Branch/leaves (parallax, slow)                   │ │
│  └────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

### Rendering Pipeline Steps

1. **Initial load:**
   - Fetch state snapshot from server
   - Place birds at their current positions
   - Start ambient motion (leaf drift, feather fall)
   - Begin rendering day/night cycle

2. **Interpolation:**
   - Cache last N snapshots (N=5–10)
   - When new snapshot arrives, interpolate positions/moods between snapshots
   - Use linear interpolation for positions, cross-fade for moods

3. **Idle motion:**
   - Client-side procedural animation
   - Mood-shaped: wary bird scans more, content bird preens
   - Independent of server tick (server only sends "current pose")

4. **Day/night cycle:**
   - Client computes local time from `Date.now()`
   - Maps to 0.0–1.0 day cycle
   - Updates palette, call volume, bird positions accordingly

5. **Reduced-motion mode:**
   - Replace frame-by-frame animations with cross-fade between still poses
   - Keep ambient motion but slow it
   - Remove leaf drift (client-side ornament)
   - Keep call synthesis and captions

6. **Accessibility surfaces:**
   - Screen-reader narration: Server-side prose generation, client-side TTS
   - Call captions: Procedural text near calling bird
   - Keyboard navigation: Tab through top bar, arrow keys between birds

### Performance Budgets

- **Initial bundle:** <2MB gzipped
- **Time-to-first-bird:** <500ms on mid-tier mobile over 4G
- **Idle motion:** 60fps on 5-year-old laptop
- **Memory growth:** No growth over 30-minute session

---

## 8. Audio Pipeline

### Procedural Call Synthesis

**WebAudio architecture:**

```
┌──────────────────────────────────────────────────────────────────────┐
│                    Bird Call Synthesis                              │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  Motif Library (per species)                                   │ │
│  │  - Basic call motifs (3–5 per species)                        │ │
│  │  - Timing patterns (per personality vector)                   │ │
│  │  - Pitch variations (per personality vector)                  │ │
│  └────────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  Motif Player                                                  │ │
│  │  - Selects motif based on mood, personality                    │ │
│  │  - Applies timing/pitch variation                             │ │
│  │  - Generates audio buffer                                     │ │
│  └────────────────────────────────────────────────────────────────┘ │
│  ┌────────────────────────────────────────────────────────────────┐ │
│  │  Chorus Mixer                                                  │ │
│  │  - Individual bird mix levels (listen-in, ambient)            │ │
│  │  - Real-time mixing (no pre-recorded loops)                   │ │
│  │  - Spatial panning (per bird position)                        │ │
│  └────────────────────────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────────────────────────┘
```

### Call Generation Flow

1. **Motif selection:** Based on mood and personality vector
   - High vocal frequency: More motifs, faster transitions
   - High boldness: Simpler motifs, louder
   - High curiosity: More varied motifs

2. **Timing application:** Based on vocal frequency trait
   - High frequency: Short inter-call intervals, quick motifs
   - Low frequency: Long inter-call intervals, slower motifs

3. **Pitch variation:** Based on personality + mood
   - Content mood: Slightly higher pitch
   - Wary mood: Slightly lower pitch
   - Curiosity: Variable pitch within motif

4. **Chorus mixing:**
   - Listen-in: Raise focused bird's mix level, lower others
   - Ambient: All birds at baseline level
   - Spatial panning: Based on perch zone (front=center, back=side)

### Call Captioning

- Procedural caption text generated per call
- Examples:
  - "a soft three-note rise"
  - "a low trill, paused, low trill again"
  - "a single sharp call from the back perch"
- Captions appear near calling bird, fade in/out with call
- Same naturalist voice as field notebook

### WebAudio Fallback

- If WebAudio unavailable: Play in graceful silence
- Captions on by default
- No recorded audio fallback (would violate bundle budget or sound canned)

---

## 9. Accessibility Surfaces

### Screen-Reader Narration

**Narration cadence:**
- Idle: 1 prose update per 30–60 seconds
- Events: Prompt on return-greeting, offer reaction, settle, etc.

**Narration prose:**
- Naturalist field-notebook voice
- Lowercase, present-tense, specific
- Example: "a warbler perches on the high branch, calling softly"

**Implementation:**
- Server generates prose from state snapshot
- Client uses Web Speech API for TTS
- Queue management to prevent overwhelming screen reader

### Reduced-Motion Mode

**Not "animations off":** A designed surface in its own right

**Visual changes:**
- Idle motion: Cross-fade between still poses instead of frame-by-frame
- Bird transitions: Cross-fade between perches instead of animated paths
- Ambient motion: Removed (leaf drift)
- Ambient color shifts: Slowed (day/night cycle)

**Audio unchanged:** Calls still play at full quality

**Narration unchanged:** Still generated at same cadence

### Call Captions

- Procedural text per call
- Small text near calling bird
- Fade in/out with call
- Same naturalist voice

### Keyboard Navigation

| Action | Key |
|--------|-----|
| Focus first bird | Tab (after top bar) |
| Move between birds | Arrow keys |
| Listen-in on focused bird | Enter |
| Exit listen-in | Escape |
| Open offer affordance | Top bar shortcut |
| Open notebook | Top bar shortcut |
| Trigger settle | Top bar shortcut |

**Focus indicators:**
- High-contrast outline
- Visible against both bright and dim aviary states

### WCAG AA Contrast

- All user-copy text passes AA contrast
- Top bar labels, settings, error surfaces, captions, narration visual display
- Aviary scene itself has no user copy except top bar

---

## 10. Performance Budgets and Observability

### Performance Budgets (Per `accessibility_perf.md`)

| Metric | Target | Measurement |
|--------|--------|-------------|
| Initial JS bundle | <2MB gzipped | Webpack bundle analysis |
| Time to first bird | <500ms on mid-tier mobile 4G | Synthetic monitoring |
| Idle motion FPS | 60fps on 5-year-old laptop | RUM render-frame timing |
| Memory growth (30 min) | No growth | RUM heap snapshots |
| Simulation-tick latency p99 | <5 seconds | Server metrics |

### Observability

**Synthetic monitoring:**
- Automated browsers run aviary on schedule
- Measure: page load times, first-bird-render, render-frame timing
- Geographies: US East, US West, EU, APAC

**Real User Monitoring (aggregate only):**
- Page load timings
- First-bird-render timings
- Render-frame timings
- Audio-context errors
- Simulation-tick latencies
- Error rates (4xx, 5xx)

**Telemetry boundary:**
- Aggregate metrics: Allowed (counts, latencies, error rates)
- Per-account interaction state: Never part of telemetry
- Per-bird state: Never aggregated for any purpose

**Error budget:**
- Simulation-tick latency p99 >5 seconds: Alarm
- Audio-context errors >1% of sessions: Alarm
- Time-to-first-bird p95 >750ms: Alarm

---

## 11. Rollout

### v1 Shipping

**Deployment:**
- Single deploy to production
- No feature flags for core features (all or nothing)
- Optional features (visits) can be toggled via feature flag

**Bird-per-aviary ramp:**
- Start with 2 birds (starter pair)
- No ramp needed — 2 is the minimum, 7 is the cap
- Bird count cap is built into engine

**Instrumentation from day one:**
- Synthetic performance checks (immediate)
- Aggregate RUM (immediate)
- Error rates (immediate)
- No per-account telemetry (by design)

### Post-v1 Considerations (Not in Scope for This Plan)

- Native app support (if demand justifies)
- Bird species additions (if species pool exhausted)
- Visit feature opt-in rate (if uptake is low)
- Notebook entry frequency tuning (based on user feedback)

---

## 12. Risks

### Drift Calibration Risk

**Risk:** Drift function too fast or too slow, making birds feel robotic or unresponsive

**Mitigation:**
- Simulate 30 days of regular visits pre-launch
- Verify personality vectors shift by 0.1–0.2 per trait
- A/B test drift rates on internal accounts
- Monitor user feedback on "birds feel different" reports

### Sync Correctness Risk

**Risk:** Multi-device sync produces inconsistent states, causing user confusion

**Mitigation:**
- Strict server-authoritative state model
- Additive deltas only (no last-write-wins)
- Event log consumed in timestamp order
- Test multi-device scenarios in staging with realistic network conditions

### Audio Uncanniness Risk

**Risk:** Procedural calls sound canned or artificial, breaking the spell

**Mitigation:**
- Procedural calls must vary every time (no looped audio)
- Chorus mixing must be real-time (no pre-recorded stacks)
- Test with users who are sensitive to audio quality
- Instrument audio-context errors and user feedback

### Accessibility Regression Risk

**Risk:** Accessibility surfaces degrade or break, excluding users

**Mitigation:**
- Accessibility work ships with v1 (not post-launch)
- Screen-reader testing with actual users
- Reduced-motion mode designed as its own surface, not a fallback
- WCAG AA contrast tested on all user-copy text
- Keyboard navigation tested on all interactive surfaces

### Performance Degradation Risk

**Risk:** Bundle grows beyond budget, time-to-first-bird exceeds 500ms

**Mitigation:**
- Bundle budget enforced in CI (2MB cap)
- Code-splitting for rarely-used surfaces (settings, visits)
- Lazy-load non-critical assets
- Synthetic performance monitoring in CI and production

### Mood-Drift Confusion Risk

**Risk:** Users conflate mood (fast, temporary) with personality (slow, permanent), expecting immediate changes

**Mitigation:**
- Clear separation in documentation and comments
- Personality vectors never exposed numerically
- Mood transitions visible in motion (bird scans when wary, preens when content)
- Field notebook observations focus on mood states, not personality numbers

### Presence-Time Signal Corruption Risk

**Risk:** Presence-time signal too loose, corrupting drift across user base

**Mitigation:**
- Strict definition: visibility + focus + recent activity
- Activity window calibrated during build (3–5 minutes)
- Test with users who leave tabs open without attention
- Monitor drift rates across accounts (should correlate with active usage, not tab-open time)

---

## 13. Implementation Notes

### What This Plan Does NOT Include

- Database schema migrations (lives with DB team)
- Exact color palette values (lives with visual designer)
- Exact animation timing values (lives with animation engineer)
- Exact prose templates for field notebook (lives with content designer)
- Exact caption templates for calls (lives with audio engineer)

### Load-Bearing Decisions from PRD

1. **Server-side simulation tick:** Non-negotiable for "feels alive over weeks"
2. **Additive personality drift:** Non-negotiable for drift correctness
3. **Procedural calls:** Non-negotiable for call variety and chorus
4. **No gamification:** Non-negotiable for relationship shape
5. **Monotonic drift:** Non-negotiable for "no Tamagotchi" rule
6. **Server canonical state:** Non-negotiable for multi-device sync

### Calibration Points (Implementation Details)

- Tick cadence: ~60 seconds (test 30s, 60s, 90s)
- Presence-time activity window: 3–5 minutes (test)
- Notebook entry probability: ~5% per tick (test sparsity)
- Reduced-motion cross-fade duration: 2–3 seconds (test)
- Call caption generation: Per call, not pre-stored

---

## 14. Success Criteria

### Technical Success

- [ ] Initial bundle <2MB gzipped
- [ ] Time-to-first-bird <500ms on mid-tier mobile 4G
- [ ] 60fps idle motion on 5-year-old laptop
- [ ] No memory growth over 30-minute session
- [ ] Simulation-tick p99 latency <5 seconds
- [ ] All accessibility surfaces functional (screen reader, keyboard, reduced motion)

### Product Success (User-Facing)

- [ ] Users report birds "feel alive" (not robotic)
- [ ] Users report birds change visibly over weeks (drift working)
- [ ] Users report birds remember them (return greeting working)
- [ ] Users report birds react to interactions (offer, listen-in working)
- [ ] Users report birds have distinct personalities (species + personality working)

### Accessibility Success

- [ ] Screen-reader users hear naturalist prose, not state lists
- [ ] Reduced-motion users see a designed surface, not broken animations
- [ ] Keyboard users can complete all interactions without mouse
- [ ] Call caption users understand bird calls without audio

---

## 15. References

- `prd/product_brief.md` — headline concept, design philosophy, voice and tone
- `prd/concepts.md` — domain vocabulary (presence, drift, mood, personality)
- `prd/bird_engine.md` — personality vector, mood, calls, drift
- `prd/interactions.md` — return greeting, listen-in, offer, settle, notebook
- `prd/aviary_layout.md` — visual scene, perch zones, day/night, top bar
- `prd/accounts_sync.md` — auth, simulation tick, multi-device sync, privacy
- `prd/social_optional.md` — visit invitations, read-only ambient
- `prd/accessibility_perf.md` — screen reader, reduced motion, captions, performance
- `prd/non_goals.md` — explicit out-of-scope (no native app, no gamification, etc.)

---

**End of Plan**
