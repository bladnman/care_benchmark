# Pocket Aviary — System Architecture and Comprehensive Implementation Plan

## 1. Executive Summary & Architectural Vision

Pocket Aviary is a browser-based, ambient virtual aviary designed to cultivate a slow, observational relationship between the user and a small flock of procedural birds (starting with two, capping at seven). Unlike virtual pets or gamified habit trackers, Pocket Aviary rejects custodial mechanics, hunger meters, scores, streaks, and push notifications. Instead, it operates on the foundational principle that **idle attention is real interaction**. Over weeks of quiet presence, birds undergo monotonic personality drift toward expressiveness, their behaviors and procedural calls evolving naturally.

The system is architected around a **server-side canonical simulation tick** coupled with a lightweight, declarative web client. The client renders fluid visual scenes and synthesizes procedural audio in real time via WebAudio, while delegating all long-term state mutations, drift functions, and mood transitions to an authoritative server-side simulation engine. This split guarantees seamless multi-device continuity without peer-to-peer sync conflicts, prevents client-side tampering or drift corruption, and ensures the aviary continues living even when the user is away.

```
+-----------------------------------------------------------------------------------+
|                                 Client (Browser)                                  |
|                                                                                   |
|  +---------------------+   +---------------------+   +-------------------------+  |
|  |   Scene Renderer    |   | WebAudio Synthesis  |   | Accessibility / Chrome  |  |
|  |  (Canvas/SVG Layers |   |  (Procedural Calls, |   | (Live Region Narration, |  |
|  |  & Micro-motion)    |   |  Chorus & Spatial)  |   |  Captions, Focus Ring)  |  |
|  +----------^----------+   +----------^----------+   +------------^------------+  |
|             |                         |                           |               |
|             +-------------------------+---------------------------+               |
|                                       |                                           |
|                      +----------------+-----------------+                         |
|                      | Client State Machine & Presence  |                         |
|                      | Accounting (Focus/Vis/Activity)  |                         |
|                      +----------------^-----------------+                         |
+---------------------------------------|-------------------------------------------+
                                        | HTTPS / REST / SSE
                                        | (Snapshots & Append-Only Events)
+---------------------------------------v-------------------------------------------+
|                              Server Infrastructure                                |
|                                                                                   |
|  +--------------------+   +---------------------+   +--------------------------+  |
|  | Auth & Gateway     |   | Event Ingestion API |   | Snapshot Delivery CDN    |  |
|  | (Magic Link, UUID) |   | (Append-Only Log)   |   | (Edge Cached Snapshots)  |  |
|  +---------+----------+   +----------+----------+   +------------^-------------+  |
|            |                         |                           |                |
|  +---------v-------------------------v---------------------------+-------------+  |
|  |               Authoritative Background Simulation Engine                    |  |
|  |  - 60s Account Simulation Tick Loop                                          |  |
|  |  - Low-Pass Monotonic Personality Drift Function                             |  |
|  |  - Markov Mood Transition Matrix & Environmental Drivers                    |  |
|  |  - Naturalist Field Notebook Generator                                      |  |
|  +-----------------------------------+-----------------------------------------+  |
|                                      |                                            |
|  +-----------------------------------v-----------------------------------------+  |
|  |                    Persistent Storage & Isolation Layer                     |  |
|  |  - PostgreSQL: Canonical Aviary, Birds, Vectors, Notebook, Magic Links      |  |
|  |  - Redis: Append-Only Event Log Stream & Session Store                      |  |
|  |  - Zero Per-Bird Analytics Pipeline (Strict Privacy Boundary)               |  |
|  +-----------------------------------------------------------------------------+  |
+-----------------------------------------------------------------------------------+
```

---

## 2. Scope & Boundary Definitions

### 2.1 In-Scope for v1
* **Single-User Accounts**: Email-based magic link authentication (15-minute expiration, single-use, per-email rate limiting, revocable per-device sessions, synthetic UUID keys throughout all internal layers).
* **Aviary Lifecycle & Scaling**: Exactly one canonical aviary per account. Starts with 2 starter birds selected automatically from a pool of ~6 species; expands up to a hard ceiling of 7 birds unlocked strictly by aviary age (e.g., bird 3 at 1 month, bird 4 at 3 months, bird 5 at 6 months, bird 6 at 9 months, bird 7 at 12 months).
* **Procedural Bird Engine**:
  * Persistent 5-dimensional personality vector (`boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`) with monotonic drift toward expressiveness.
  * Fast-timescale mood state machine (`wary`, `content`, `curious`, `drowsy`, `alert`).
  * Return-greeting engine dynamically parameterized by absence duration, boldness, and mood.
* **Core Interaction Mechanics**:
  * **Presence Accounting**: Tripartite presence detection (visible tab + window focus + user activity within sliding window).
  * **Listen-in**: Focusing a bird to smoothly elevate its procedural call in the audio mix while gently ducking peer birds to ambient levels without muting them.
  * **Offer Gestures**: Seed, song fragment, and still pool affordances from the top chrome with per-bird cooldowns (3–5 minutes).
  * **Settle Gesture**: User-initiated evening shift with 5-second click-undo affordance, ending presence cleanly.
* **Naturalist Field Notebook**: Auto-generated, read-only observation log in lowercase present-tense prose; sparse generation cadence (~1 entry per 2–4 days).
* **Optional & Quiet Social Visits**: One-time, email-generated read-only visit links (30-day expiry, revocable immediately, visit log in settings, no co-presence, no visitor drift impact).
* **First-Class Accessibility**: Screen-reader live region narration in naturalist voice (30–60s cadence), procedural call captions, reduced-motion cross-fade rendering, WCAG AA contrast, full keyboard navigation.
* **Procedural WebAudio Synthesis**: Real-time synthesis of bird vocalizations using WebAudio oscillators, envelope shaping, and FM synthesis, with a graceful silent fallback (with auto-enabled captions).

### 2.2 Explicit Out-of-Scope (Non-Goals)
* **No Native Applications**: No iOS or Android native binaries; strictly standard web platform.
* **No Gamification Elements**: No streaks, no visit counters, no badges, no levels, no scores, no daily green-dot calendars, no XP, no milestone toasts.
* **No Custodial / Tamagotchi Mechanics**: No hunger, no feeding schedules, no health/happiness bars, no bird illness, no bird death, no negative drift on absence.
* **No Social Network Mechanics**: No public discovery, no global aviary directory, no leaderboards, no follower feeds, no co-presence avatars, no chat/comments, no visit badges/counters.
* **No Push Notifications or Marketing Emails**: No unsolicited outreach; aviary only exists when the user opens the tab.
* **No Numerical Personality Exposure**: Personality vector numbers are strictly hidden from all UI, debug panels, tooltips, and client payloads.

### 2.3 Voice and Tone Architectural Boundary
* **Naturalist Voice (Product Surface)**: Used across the aviary scene, field notebook, screen-reader narration, call captions, and offer prompts. Characteristics: lowercase by default, present-tense, bird-named, observational, quiet, zero exclamation marks, zero gamification jargon.
* **Matter-of-Fact Voice (System Surface)**: Used strictly for auth errors, magic-link landing, session revocation, sync conflicts, account settings, and accessibility menus. Characteristics: standard sentence capitalization, direct, functional, zero simulated warmth or evasive whimsy.

---

## 3. System Architecture & Component Topography

```
                                  [ Browser Client ]
                                          |
                        +-----------------+-----------------+
                        |                                   |
                (Public HTTP/REST)                   (Secure SSE / WS)
                        |                                   |
                        v                                   v
             [ Traefik / API Gateway ]             [ Snapshot Streamer ]
                        |                                   |
         +--------------+--------------+                    |
         |                             |                    |
         v                             v                    |
  [ Auth Service ]            [ Aviary API Service ]        |
         |                             |                    |
         |                    (Append Interaction)          |
         |                             |                    |
         |                             v                    |
         |                    [ Redis Stream / Queue ]      |
         |                             |                    |
         |                             v                    |
         |                 [ Simulation Worker Engine ]-----+
         |                   - 60s Cron / Account Tick
         |                   - Personality Drift Calc
         |                   - Mood & Weather State
         |                   - Notebook Generator
         |                             |
         +--------------+--------------+
                        |
                        v
              [ PostgreSQL Database ]
```

### 3.1 Service Boundaries
1. **Auth & Account Service**:
   * Issues, rate-limits, and verifies 15-minute magic links via transactional email.
   * Generates immutable synthetic `account_id` (UUIDv4) upon creation; stores encrypted email.
   * Manages revocable per-device bearer session tokens (`session_id`).
   * Handles GDPR/privacy workflows: JSON aviary data export and 30-day soft-deletion lifecycle.
2. **Aviary API & Ingestion Service**:
   * Serves current aviary state snapshots (`/api/v1/aviary/snapshot`).
   * Ingests append-only client interaction events (presence batches, offers, listen-in start/stop, settle) into an asynchronous ingestion queue (Redis Stream).
   * Validates visitor access tokens and serves read-only snapshots to guests.
3. **Authoritative Simulation Worker Engine**:
   * Executes continuous background simulation ticks at a nominal cadence of 60 seconds per active/scheduled account.
   * Dequeues interaction events, calculates additive personality deltas, updates mood timers, and steps environmental models (weather, day/night cycles).
   * Evaluates notebook entry triggers and writes prose observations to storage.
   * Writes canonical snapshots to PostgreSQL and publishes snapshot deltas to the client notification bus.
4. **Data Isolation & Storage Layer**:
   * Primary relational store: PostgreSQL with strict foreign keys tied to synthetic `account_id`.
   * Fast cache & event log: Redis Stream for un-ticked interaction logs.
   * **Telemetry Isolation**: Operational metrics (latencies, error rates, anonymous session lengths) are routed to Prometheus/Grafana. Per-bird vectors and interaction logs are structurally excluded from all analytics pipelines.

---

## 4. Canonical Data Model & Schema Specifications

```sql
-- Core Account Entity
CREATE TABLE accounts (
    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    encrypted_email BYTEA NOT NULL,
    email_hash VARCHAR(64) NOT NULL UNIQUE, -- HMAC-SHA256 for lookup without decryption
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deletion_requested_at TIMESTAMPTZ NULL, -- Soft-delete timestamp (30-day purge)
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    settings JSONB NOT NULL DEFAULT '{
        "reduced_motion": false,
        "call_captions": false,
        "visit_notifications_enabled": false
    }'::jsonb
);

-- Device Sessions
CREATE TABLE sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    user_agent TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at TIMESTAMPTZ NULL
);

-- Canonical Aviary
CREATE TABLE aviaries (
    aviary_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(account_id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    settled_until TIMESTAMPTZ NULL,
    current_weather VARCHAR(32) NOT NULL DEFAULT 'clear', -- 'clear', 'soft_rain', 'gentle_wind'
    weather_expires_at TIMESTAMPTZ NULL,
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Individual Birds
CREATE TABLE birds (
    bird_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL, -- e.g., 'grey_warbler', 'spotted_towhee', 'pine_siskin'
    name VARCHAR(64) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    perch_zone VARCHAR(16) NOT NULL DEFAULT 'middle', -- 'front', 'middle', 'back'
    current_mood VARCHAR(16) NOT NULL DEFAULT 'content', -- 'wary', 'content', 'curious', 'drowsy', 'alert'
    mood_entered_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    offer_cooldown_until TIMESTAMPTZ NULL,
    last_call_at TIMESTAMPTZ NULL
);

-- Server-Authoritative Personality Vectors (Strictly Hidden from Client UI)
CREATE TABLE bird_personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(bird_id) ON DELETE CASCADE,
    boldness DOUBLE PRECISION NOT NULL DEFAULT 0.20,
    social_warmth DOUBLE PRECISION NOT NULL DEFAULT 0.20,
    vocal_frequency DOUBLE PRECISION NOT NULL DEFAULT 0.20,
    plumage_saturation DOUBLE PRECISION NOT NULL DEFAULT 0.20,
    curiosity DOUBLE PRECISION NOT NULL DEFAULT 0.20,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT vector_bounds CHECK (
        boldness BETWEEN 0.0 AND 1.0 AND
        social_warmth BETWEEN 0.0 AND 1.0 AND
        vocal_frequency BETWEEN 0.0 AND 1.0 AND
        plumage_saturation BETWEEN 0.0 AND 1.0 AND
        curiosity BETWEEN 0.0 AND 1.0
    )
);

-- Append-Only Interaction Events (Consumed by Simulation Tick)
CREATE TABLE interaction_events (
    event_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(bird_id) ON DELETE CASCADE,
    event_type VARCHAR(32) NOT NULL, -- 'presence_batch', 'listen_in_start', 'listen_in_stop', 'offer_seed', 'offer_song', 'offer_pool', 'settle', 'settle_undo'
    event_payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    client_timestamp TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_at TIMESTAMPTZ NULL
);
CREATE INDEX idx_unprocessed_events ON interaction_events(account_id, created_at) WHERE processed_at IS NULL;

-- Field Notebook Entries
CREATE TABLE field_notebook_entries (
    entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    recorded_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    prose_text TEXT NOT NULL,
    observation_context VARCHAR(64) NOT NULL -- e.g., 'first_greeter', 'quiet_morning', 'rain_hush'
);
CREATE INDEX idx_notebook_aviary ON field_notebook_entries(aviary_id, recorded_at DESC);

-- Visit Invitations
CREATE TABLE visit_invitations (
    invite_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    host_account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    recipient_email_hash VARCHAR(64) NOT NULL,
    encrypted_recipient_email BYTEA NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '30 days'),
    revoked_at TIMESTAMPTZ NULL
);

-- Visit Logs
CREATE TABLE visit_logs (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invite_id UUID NOT NULL REFERENCES visit_invitations(invite_id) ON DELETE CASCADE,
    host_account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    visitor_email_masked VARCHAR(64) NOT NULL,
    visited_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    duration_seconds INTEGER NOT NULL DEFAULT 0
);
```

---

## 5. API Surface & Inter-Service Protocol Contracts

### 5.1 Authentication & Session Management

#### `POST /api/v1/auth/magic-link`
* **Purpose**: Request sign-in magic link.
* **Rate Limit**: 5 requests per IP / email per hour.
* **Request**:
  ```json
  { "email": "naturalist@example.com" }
  ```
* **Response (200 OK)**:
  ```json
  { "status": "sent", "message": "Check your email for your magic sign-in link." }
  ```

#### `POST /api/v1/auth/verify`
* **Purpose**: Exchange magic token for a secure bearer session.
* **Request**:
  ```json
  { "token": "mgtok_8f7b2c1e4d..." }
  ```
* **Response (200 OK)**:
  ```json
  {
    "session_token": "sess_9a8b7c...",
    "account_id": "d3b07384-d113-4f40-9a29-0123456789ab",
    "settings": {
      "reduced_motion": false,
      "call_captions": false,
      "visit_notifications_enabled": false
    }
  }
  ```
* **Error (400 / 410)**: Matter-of-fact tone: `"We couldn't sign you in. The link may have expired. Try requesting a new link."`

---

### 5.2 Aviary State & Event Streaming

#### `GET /api/v1/aviary/snapshot`
* **Purpose**: Fetches the authoritative rendering snapshot of the aviary.
* **Headers**: `Authorization: Bearer <session_token>`
* **Response (200 OK)**:
  ```json
  {
    "aviary_id": "c1f2e3d4-5678-90ab-cdef-1234567890ab",
    "server_time": "2026-08-13T17:51:15Z",
    "local_time_offset_seconds": -25200,
    "day_phase": "morning",
    "weather": "clear",
    "settled": false,
    "birds": [
      {
        "bird_id": "b1111111-2222-3333-4444-555555555555",
        "species_id": "grey_warbler",
        "name": "pip",
        "perch_zone": "front",
        "current_mood": "content",
        "plumage_rendering_factor": 0.35,
        "call_motif_seed": 48291,
        "idle_state": {
          "primary_action": "preen",
          "action_started_at": "2026-08-13T17:51:00Z"
        }
      },
      {
        "bird_id": "b2222222-3333-4444-5555-666666666666",
        "species_id": "spotted_towhee",
        "name": "wren",
        "perch_zone": "middle",
        "current_mood": "drowsy",
        "plumage_rendering_factor": 0.28,
        "call_motif_seed": 19482,
        "idle_state": {
          "primary_action": "fluffed_rest",
          "action_started_at": "2026-08-13T17:50:45Z"
        }
      }
    ],
    "screen_reader_narration": "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
  }
  ```
  *(Note: Numerical personality vectors like `boldness` or `curiosity` are strictly absent from client payloads. Only derived rendering factors like `plumage_rendering_factor` are exposed).*

#### `POST /api/v1/aviary/events`
* **Purpose**: Appends batched presence and discrete interaction events.
* **Headers**: `Authorization: Bearer <session_token>`
* **Request**:
  ```json
  {
    "events": [
      {
        "client_event_id": "e001-...",
        "event_type": "presence_batch",
        "client_timestamp": "2026-08-13T17:51:00Z",
        "payload": { "active_duration_seconds": 60 }
      },
      {
        "client_event_id": "e002-...",
        "event_type": "offer_seed",
        "bird_id": "b1111111-2222-3333-4444-555555555555",
        "client_timestamp": "2026-08-13T17:51:10Z",
        "payload": { "zone": "front" }
      }
    ]
  }
  ```
* **Response (202 Accepted)**:
  ```json
  { "status": "queued", "accepted_count": 2 }
  ```

---

### 5.3 Field Notebook & Social Visits

#### `GET /api/v1/notebook`
* **Query Parameters**: `?limit=20&cursor=...`
* **Response (200 OK)**:
  ```json
  {
    "entries": [
      {
        "entry_id": "n1-...",
        "recorded_at": "2026-08-12T08:14:22Z",
        "prose_text": "pip greeted before wren today, first time this week."
      },
      {
        "entry_id": "n2-...",
        "recorded_at": "2026-08-09T16:30:00Z",
        "prose_text": "wren is fluffed against the cool air, watching the back perch. low calls only."
      }
    ],
    "next_cursor": null
  }
  ```

#### `POST /api/v1/visits/invite` & `GET /api/v1/visits/stream/:token`
* **`POST /api/v1/visits/invite`**: Creates a visit invitation with recipient email and generates a secure URL token.
* **`GET /api/v1/visits/stream/:token`**: Validates visitor token and streams read-only aviary snapshots. Interaction submissions over this endpoint are rejected with `403 Forbidden`.

---

## 6. Simulation Engine & Mathematical Drift Architecture

### 6.1 Server Simulation Tick Loop
The simulation engine executes an atomic tick for each account once every 60 seconds.

```
[ Read Unprocessed Events from Redis/Postgres ]
                    |
                    v
[ Calculate Presence Duration & Interaction Weights ]
                    |
                    v
[ Update Personality Vector via Monotonic Low-Pass Filter ]
                    |
                    v
[ Step Mood State Machine (Markov + Environmental Drivers) ]
                    |
                    v
[ Update Perch Positions & Dynamic Idle Behaviors ]
                    |
                    v
[ Evaluate Sparse Field Notebook Triggers ]
                    |
                    v
[ Save Canonical Snapshot & Invalidate/Push to Cache ]
```

### 6.2 Mathematical Formulation of Personality Drift
Personality drift governs the slow, multi-week character evolution of each bird. Drift is calculated using a **low-pass discrete accumulator with monotonic clamping**.

For a personality trait $T \in \{\text{boldness}, \text{social\_warmth}, \text{vocal\_frequency}, \text{plumage\_saturation}, \text{curiosity}\}$ with normalized range $[0.0, 1.0]$:

$$T_{k+1} = T_k + \Delta T_k$$

Where the instantaneous delta $\Delta T_k$ is defined by:

$$\Delta T_k = \min\left(\alpha_T \cdot \sum_{i} w_{i, T} \cdot \Phi(E_i), \; \Delta_{\text{max\_tick}}\right)$$

#### Weighting Matrix $w_{i, T}$ and Decay Factors:
* $\Phi(\text{presence\_hour}) = \text{duration in hours with valid focus/visibility/activity}$.
* $\Phi(\text{listen\_in\_minute}) = \text{duration in minutes focused on bird}$.
* $\Phi(\text{offer\_accepted}) = 1.0$ per accepted offer (subject to 3-minute cooldown).

| Interaction Signal ($E_i$) | Boldness ($w_{\text{bold}}$) | Social Warmth ($w_{\text{warm}}$) | Vocal Freq ($w_{\text{vocal}}$) | Plumage ($w_{\text{plum}}$) | Curiosity ($w_{\text{curious}}$) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Presence (per hr)** | $0.0015$ | $0.0020$ | $0.0010$ | $0.0025$ | $0.0010$ |
| **Listen-in (per min)** | $0.0005$ | $0.0012$ | $0.0015$ | $0.0008$ | $0.0005$ |
| **Offer: Seed** | $0.0020$ | $0.0010$ | $0.0002$ | $0.0005$ | $0.0025$ |
| **Offer: Song** | $0.0005$ | $0.0025$ | $0.0030$ | $0.0005$ | $0.0015$ |
| **Offer: Water Pool** | $0.0015$ | $0.0015$ | $0.0005$ | $0.0030$ | $0.0020$ |

#### Monotonicity Invariant:
$$\Delta T_k \ge 0 \quad \forall k$$
Neglect does **not** decay $T$. A bird that is not visited for two weeks experiences $\Delta T = 0$. Its personality remains at its current expressive level, while its mood relaxes into an ambient, quiet state.

#### Calibration Targets:
* **1 Week of Daily Regular Attention (20 min/day)**: Trait changes $\Delta T \approx 0.015 - 0.025$ (detectable by telemetry/test harnesses; subtle to the user).
* **3 Weeks of Sustained Visits**: Trait changes $\Delta T \approx 0.06 - 0.10$ (visibly apparent in perch choice, greeting frequency, plumage saturation, and call complexity).

---

### 6.3 Mood Transition Engine
Mood is a fast-timescale state machine transitioning on a discrete Markov matrix modulated by time of day, weather, and personality traits.

```
                    +--------------------+
                    |       WARY         |
                    +---+------------^---+
                        |            |
         Gentle Presence|            | Alarm / Sudden Event
         & High Warmth  |            | / Low Boldness
                        v            |
                    +----------------+---+
        +---------->|      CONTENT       |<---------+
        |           +---+------------^---+          |
        |               |            |              |
        |  Accepted     |            | Cooldown /   |
        |  Offer        |            | Quiet Time   |
        |               v            |              |
        |           +----------------+---+          |
        +-----------+      CURIOUS       |          |
                    +--------------------+          |
                        |                           |
              Dusk /    |            Time of Day    | Dawn / Morning
              Night     v            (Morning)      | / High Vocal
                    +--------------------+          |
                    |       DROWSY       |          |
                    +---+------------^---+          |
                        |            |              |
                        +------------+--------------+
                                     |
                                     v
                             +---------------+
                             |     ALERT     |
                             +---------------+
```

* **Wary**: Prefers `back` perch zone; head scan rate increased by 2x; calls rare and short. Triggered by long absence return or low boldness baseline.
* **Content**: Balances `middle` and `front` perches; engages in regular preening micro-motion; joins chorus readily.
* **Curious**: Steps toward front perch; tilts head toward ambient noises; investigates offers.
* **Drowsy**: Fluffed plumage pose; low vocalization; occurs naturally during user evening/night hours.
* **Alert**: Upright posture; responsive to passing wind or other birds' calls.

---

### 6.4 Naturalist Field Notebook Generator
To maintain the authenticity of a naturalist's field observations:
1. **Sparsity Filter**: Generator evaluates conditions once daily during the simulation tick. Max 1 entry every 48–72 hours unless a rare milestone occurs (e.g., first greeting from a historically wary bird).
2. **Context Engine**: Evaluates relative event patterns:
   * First greeter comparison: `"pip greeted before wren today, first time this week."`
   * Weather & mood correlation: `"wren is fluffed against the cool air, watching the back perch. low calls only."`
   * Quiet observation periods: `"a long stretch of quiet this morning. pip preened for several minutes without looking up."`
3. **Voice Enforcer**: All strings are passed through a compiler asserting: all lowercase, present-tense, bird names present, zero gamification nouns/verbs.

---

## 7. Multi-Device Synchronization & Conflict Prevention Model

### 7.1 Single-Writer Authority
The server-side simulation tick is the **sole writer** of aviary state and personality vectors. Clients are strictly read-only renderers of canonical snapshots and producers of append-only interaction events.

```
+----------------+                       +----------------+
|  Laptop Tab    |                       |   Phone Tab    |
+-------+--------+                       +--------+-------+
        |                                         |
        | 1. Pull Snapshot (v42)                  | 1. Pull Snapshot (v42)
        |<-------------------+------------------->|
        |                    |                    |
        | 2. Post Listen-In  |                    | 2. Post Offer Seed
        |    (Event E101)    |                    |    (Event E102)
        |                    v                    |
        +----------------> [Server Event Log] <---+
                             |
                             | 3. Simulation Tick (v42 -> v43)
                             |    - Ingests E101 & E102
                             |    - Computes Additive Deltas
                             |    - Publishes Snapshot v43
                             v
        |<-------------------+------------------->|
        | 4. Pull Snapshot (v43)                  | 4. Pull Snapshot (v43)
```

### 7.2 Append-Only Ingestion & Idempotency
1. Client generates UUIDv4 `client_event_id` for every interaction.
2. Ingestion pipeline uses Redis `HSETNX` on `processed_events:<account_id>` with a 24-hour TTL to reject duplicates from network retries.
3. Event deltas are commutative: `listen_in` from Device A and `offer` from Device B are both appended to the event log and processed sequentially in the next 60s tick.

### 7.3 Client Interpolation & State Blending
* When snapshot $N+1$ arrives, the client does not snap or teleport birds.
* Positions, plumage saturation, and mood poses are smoothly interpolated using spring-damper smoothing over a 1.2-second transition window.

---

## 8. Frontend Rendering Pipeline & Visual Scene Architecture

### 8.1 Layered Canvas / SVG Scene Composition
The aviary scene is rendered on a single non-scrollable, responsive HTML5 2D Canvas / WebGL plane with structured semantic overlay:

```
+-------------------------------------------------------------------+
| Top Bar Chrome (Opacity: 1.0 on motion -> 0.05 on idle after 4s)  |
| [Settings] [Accessibility]               [Notebook] [Offer Menu]  |
+-------------------------------------------------------------------+
| Layer 4: Ambient Foreground (Subtle leaf drift, soft light bloom) |
+-------------------------------------------------------------------+
| Layer 3: Middle Perch Plane (Birds, Perches, Still Pool Surface)  |
|          - Front Perch (y: 75%, scale: 1.00)                      |
|          - Middle Perch (y: 55%, scale: 0.85)                     |
|          - Back Perch (y: 35%, scale: 0.70)                       |
+-------------------------------------------------------------------+
| Layer 2: Background Foliage (Parallax Factor: 0.05x on pointer)   |
+-------------------------------------------------------------------+
| Layer 1: Sky & Atmosphere (Time-of-day gradient, rain particles)  |
+-------------------------------------------------------------------+
```

```
                                  [ Viewport (100vw, 100vh) ]
                                               |
                   +---------------------------+---------------------------+
                   |                                                       |
        [ Canvas Layer (Visuals) ]                             [ DOM Overlay (A11y/UI) ]
                   |                                                       |
  +----------------+----------------+                     +----------------+----------------+
  | Sky / Weather Gradient          |                     | Top Bar Chrome (Auto-Fade)      |
  | Background Foliage (0.05x Para) |                     | ARIA Live Narration Region      |
  | Perches & Birds (Micro-motion)  |                     | Call Captions (Near Bird Poses) |
  | Foreground Leaf / Feather Drift |                     | Interactive Keyboard Hit-Boxes  |
  +---------------------------------+                     +---------------------------------+
```

### 8.2 Immediate First Frame ("Already in Motion")
* To satisfy the principle that the aviary has been continuing without the viewer, the client embeds the initial snapshot directly in the initial HTML or fetches it via preload within $<150\text{ms}$.
* Procedural oscillators and idle animations initialize with a computed phase offset $\theta = (\text{now} - \text{action\_started\_at}) \pmod T$, ensuring the very first rendered frame depicts birds mid-preen, mid-hop, or mid-call without an entry fade or spinner.

### 8.3 Idle Micro-Motion Systems
* **Sinusoidal Breathing**: Body scale oscillates gently at $0.2\text{Hz}$ with a random phase offset per bird.
* **Procedural Head Tilts**: Performed via cubic Hermite interpolation between target angles $[-25^\circ, +25^\circ]$ triggered every 4–8 seconds, modulated by `curiosity`.
* **Preening Sequences**: Randomized multi-step spine deformation and beak-to-feather vector positioning.

### 8.4 Reduced-Motion Rendering Mode
When `prefers-reduced-motion` is active or enabled via accessibility settings:
* Continuous frame-by-frame skeletal transforms and wing flutters are disabled.
* Motion is replaced with **slow cross-fades between static poses** (1.5-second opacity dissolve).
* Perch changes occur via cross-dissolve rather than flight trajectory.
* Ambient drifting leaves and parallax depth shifts are removed.

---

## 9. WebAudio Procedural Synthesis & Soundscape Engine

### 9.1 Procedural Call Grammar
Calls are synthesized dynamically at runtime using WebAudio nodes rather than audio loops.

```
[ Motif Generator ] ---> [ FM Modulator (Sine Osc) ]
                                |
                                v (Frequency Modulation)
[ Envelope (ADSR) ] ---> [ Carrier Oscillator (Sine/Triangle) ]
                                |
                                v
                         [ Formant Biquad Filter ]
                                |
                                v
                         [ Per-Bird Gain Node ] ---> [ Stereo Panner ] ---> [ Master Aviary Bus ]
```

* **Motif Library**: Each species possesses 4–6 parameterized melodic motifs consisting of frequency contours, harmonic ratios, and duration vectors.
* **Variation Engine**: Runtime calls introduce $\pm 3\%$ pitch jitter, micro-timing modulation, and harmonic emphasis shaped by bird `vocal_frequency` and `mood`.
* **Chorus Assembly**: Multiple birds calling stagger their start times by $180\text{ms} - 450\text{ms}$ using an organic collision-avoidance scheduler to produce realistic counterpoint rather than phase-canceling overlap.

### 9.2 Listen-In Mix Architecture
When a user listens in on bird $B_{\text{target}}$:
* Target bird gain ramps from $0\text{dB}$ to $+3\text{dB}$ over $600\text{ms}$ using `exponentialRampToValueAtTime`.
* All other active birds $B_{\text{peer}}$ ramp their gain down from $0\text{dB}$ to $-14\text{dB}$ over $800\text{ms}$ (gentle ambient ducking; never hard-muted).
* Reverb wet-mix on target bird is reduced by $50\%$ to bring it acoustically closer to the listener.
* On disengagement, all gains return to unity ambient mix ($0\text{dB}$) over an $800\text{ms}$ linear ramp.

```
Master Mix
 ^
 |    [ Listen-in Engaged on Bird A ]              [ Listen-in Disengaged ]
 |        +-------------------------------+
 |       /                                 \
 |      /                                   \
+3dB --+   <-- Bird A Gain (Target)          +----------------------- (0dB)
 |                                            
 0dB --+                                     +----------------------- (0dB)
 |      \                                   /
 |       \                                 /
-14dB ----+-------------------------------+    <-- Peer Birds Gain (Ducked)
 +------------------------------------------------------------------------> Time
```

### 9.3 WebAudio Fallback & Error Strategy
If WebAudio context creation fails (browser policy, blocked autoplay, unsupported hardware):
* The audio engine transitions smoothly to **Graceful Silence Mode**.
* Call captioning is automatically enabled in the UI settings.
* **No recorded audio fallback is shipped**, strictly preserving the bundle budget and preventing canned, repetitive audio loops.

---

## 10. Accessibility Architecture & Inclusive Design Surfaces

### 10.1 Screen-Reader Narration Engine
A dedicated DOM element `<div id="aviary-narration" aria-live="polite" class="sr-only">` is maintained.
* **Slow Observation Cadence**: Prose updates are pushed every 30–60 seconds.
* **Naturalist Style**:
  ```
  "a warbler perches on the high branch, calling softly. wren watches from the lower rail."
  ```
* **Event Priority**: User-triggered events (accepted offer, return-greeting) update the live region immediately, framed as observations rather than system status updates.

### 10.2 Procedural Call Captions
When enabled, captions render as floating, translucent speech badges positioned adjacent to the calling bird:
* Rendered text matches the exact procedural motif played (e.g., *"a soft three-note rise"*, *"a low trill, paused, low trill again"*).
* Badges fade in over $200\text{ms}$, hold for the duration of the call, and fade out over $600\text{ms}$.

### 10.3 Keyboard Interaction & Focus System
* **Roving Tabindex**: Pressing `Tab` enters the aviary and focuses the top bar, then the first bird. Arrow keys (`Left` / `Right` / `Up` / `Down`) navigate focus between birds based on spatial proximity.
* **Key Bindings**:
  * `Enter` / `Space`: Toggle Listen-in on focused bird.
  * `Escape`: Exit Listen-in or close overlay panels.
  * `O`: Open Offer menu.
  * `S`: Trigger Settle gesture.
  * `N`: Open Field Notebook.
* **High-Contrast Focus Indicator**: A 3px dual-tone outline (`#FFFFFF` inner / `#1A1A1A` outer) ensures WCAG AA visibility against any daylight or night scene background.

---

## 11. Performance Budgets, Resource Constraints & Observability

### 11.1 Hard Performance Budgets
| Metric | Budget Target | Enforcement Mechanism |
| :--- | :--- | :--- |
| **Initial JS Bundle Size** | $< 2.0\text{MB}$ gzipped | Webpack/Vite bundle analyzer in CI; tree-shaking; procedural generation. |
| **Time to First Bird Visible** | $< 500\text{ms}$ on 4G / mid-tier | SSR/Edge initial snapshot injection in HTML payload; zero render-blocking assets. |
| **Idle Render Frame Rate** | $60\text{fps}$ on 5-yr-old hardware | Lightweight 2D canvas draw routines; zero DOM thrashing during render loops. |
| **Memory Stability** | $0\text{MB}$ net leak over 30 min | WebAudio node pooling; canvas path reuse; zero object allocations in tick loop. |
| **Server Tick Latency** | $p99 < 5.0\text{s}$ | Batched database transactions; background worker horizontal scaling. |

### 11.2 Telemetry & Privacy Boundary
* **Permitted Operational Telemetry**: Aggregated request rates, simulation tick execution latency histograms, WebAudio error counts, bundle load times, anonymized session duration buckets.
* **Strict Privacy Firewall**: No bird names, no personality vectors, no individual presence logs, and no per-account interaction histories are ever routed to external telemetry, analytics warehouses, or machine-learning datasets.

---

## 12. Rollout Strategy, Calibration Framework & Progression Pacing

### 12.1 Phased Implementation Milestones

```
[ Phase 1: Core Engine & Audio ] (Weeks 1-4)
  - Data schema & synthetic UUID auth pipeline.
  - WebAudio procedural synthesis & motif library.
  - Basic 2D canvas renderer & micro-motion loop.

[ Phase 2: Simulation & Drift Calibration ] (Weeks 5-8)
  - 60s server tick worker & Redis event ingestion.
  - Monotonic drift math & headless simulation test harness.
  - Field notebook generator & prose validation.

[ Phase 3: Accessibility & Polish ] (Weeks 9-11)
  - Screen reader live region narration & call captions.
  - Reduced-motion cross-fade rendering.
  - Single-user visit invitation & revocation flow.

[ Phase 4: Performance Hardening & Launch ] (Weeks 12-14)
  - 30-minute memory leak stress tests in headless Chrome.
  - Bundle optimization (<2MB) & edge snapshot delivery.
  - Production deployment & phased user onboarding.
```

### 12.2 Flock Progression Pacing
* **Day 1**: 2 starter birds from pool.
* **Month 1**: 3rd bird arrives.
* **Month 3**: 4th bird arrives.
* **Month 6**: 5th bird arrives.
* **Month 9**: 6th bird arrives.
* **Month 12**: 7th bird arrives (flock capacity reached).

---

## 13. Comprehensive Risk Matrix & Mitigation Strategies

| Risk Category | Specific Failure Mode | Impact | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **Drift Calibration** | Presence signal inflated by inactive background tabs. | Birds drift too fast; relationship feels artificial. | Enforce strict tripartite presence check (`visible` + `focused` + `user event within window`). Background tabs cease presence accounting immediately. |
| **Audio Uncanniness** | Procedural calls sound harsh, synthetic, or robotic. | Breaks immersion and emotional attachment. | Use multi-stage formant filters, randomized micro-pitch jitter ($\pm 3\%$), and organic ADSR envelopes tuned directly against avian bioacoustics. |
| **Sync Race Conditions** | Concurrent writes from multi-device usage corrupt drift. | Drift progress lost or overwritten. | Strictly enforce single-writer architecture: server tick computes all deltas; clients only append interaction events to log. |
| **Accessibility Regressions** | Screen reader receives raw state dumps instead of prose. | Accessible experience feels like a debugging tool. | Route all live region updates through the naturalist prose compiler; test with VoiceOver and NVDA in CI. |
| **Bundle Bloat** | Heavy dependencies push initial payload $> 2\text{MB}$. | First bird render exceeds 500ms on mobile. | Avoid heavy UI frameworks; use Vanilla/lightweight TypeScript, procedural SVGs, and WebAudio synthesis in place of audio files. |

---
*End of Implementation Plan — Pocket Aviary v1.*
