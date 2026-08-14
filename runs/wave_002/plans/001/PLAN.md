# Pocket Aviary — Comprehensive Engineering & Implementation Plan (v1)

## 1. Executive Summary & Scope

### 1.1 Product Purpose & Philosophy
Pocket Aviary is a quiet, browser-based virtual aviary. Users adopt an initial pair of birds (growing up to a maximum of seven based strictly on aviary age) inhabiting a single horizontal scene. The product centers on long-term, low-key observational companionship rather than game mechanics, task completion, or custodial burden.

The system is governed by five strict design principles:
1. **Feels alive, not robotic**: Mid-action loading (no entry spinners or "ready" pops), continuous idle micro-motion, procedural WebAudio calls that never repeat identically, and a server-side simulation that advances independently of whether a client is connected.
2. **Notice, never announce**: Welcoming occurs solely through procedurally varied bird behavior (head-tilts, glances, calls, approaching perches). Textual greetings, "Welcome back!" toasts, banners, celebratory confetti, or visit counters are strictly forbidden.
3. **Charm comes from specificity**: Naturalist, lowercase, present-tense observations in the field notebook and screen-reader narration (e.g., *"pip greeted before wren today, first time this week"* instead of generic gamified milestones).
4. **Restraint over richness**: A single fixed horizontal viewport with no scrolling, panning, or zooming; a calm, nature-derived palette; and an aviary scene free of in-world UI chrome.
5. **Naturalist voice for product, matter-of-fact for system**: Product surfaces use lowercase present-tense naturalist prose. Account management, authentication, sync errors, settings, and unsupported browser states use direct, standard-capitalized, matter-of-fact technical prose without artificial charm.

---

### 1.2 In-Scope vs. Out-of-Scope (v1 & Permanent Non-Goals)

| Domain | In-Scope (v1 Execution) | Out-of-Scope / Non-Goals (Strictly Forbidden) |
|---|---|---|
| **Platform** | Modern Web Browsers (Last 2 major versions of Chrome, Safari, Firefox, Edge). Fully responsive single canvas. | Native iOS / Android apps; desktop wrappers; WebAssembly heavier than budget. |
| **Birds & Population** | 2 starter birds assigned from a 6-species pool. Max cap of 7 birds, unlocked strictly by aviary age milestones. | User catalog selection at start; bird count > 7; unlocking birds via visit streaks, payment, or interaction counts. |
| **Gamification & Mechanics** | Monotonic drift toward expressiveness; mood state transitions; presence accumulation. | Streaks, levels, XP, badges, scores, achievements, green-dot calendars, "days visited" counters, adoption counters. |
| **Custodial Mechanics** | Idle observation; gestures (seed, song, pool offers; settle goodbye). | Hunger, feeding meters, health bars, bird mortality, neglect penalties, negative personality drift. |
| **Social Surface** | 1:1 email-based visit invitations; read-only ambient viewing; instant revocation; silent visit log. | Public directories, discovery feeds, profiles, follows, chat, visitor avatars, comments, leaderboards, co-presence. |
| **Authentication & Accounts** | Single-user accounts; passwordless email magic link (15-min TTL); revocable per-device sessions; JSON export; 30-day soft delete. | Social login / SSO; shared / multi-user aviaries; public profile handles; unverified email changes. |
| **Audio** | Real-time procedural WebAudio synthesis; dynamic chorus mixing; gradual listen-in focus; silent fallback with captions. | Looped audio samples; recorded sound files; sudden track soloing/muting; recorded audio fallback. |
| **Accessibility** | Naturalist screen-reader narration (30–60s cadence); designed reduced-motion cross-fade mode; procedural call captions; full keyboard nav; WCAG AA contrast. | Static accessibility fallbacks; ARIA stat dumping; un-designed reduced-motion disabling; noisy high-frequency narration. |

---

## 2. System Architecture & Service Boundaries

Pocket Aviary adopts a client-server architecture with an authoritative, server-driven simulation. Clients function as deterministic renderers and event-stream contributors, eliminating client-side state divergence across multiple devices.

```
                                  +--------------------------------------------------------+
                                  |                     Client Layer                       |
                                  |  (Vanilla JS / Canvas 2D Engine / WebAudio Synthesizer)|
                                  +--------------------------------------------------------+
                                         | ^                                      |
                       1. Auth / SSE /   | | 2. Canonical State Snapshot          | 3. Append Interaction
                       Snapshot Pull     | |    (Kilobytes, Interpolated)         |    Events (Presence,
                                         v |                                      v    Offers, Listen-in)
+---------------------------------------------------------------------------------------------------------+
|                                        API Gateway & Edge (CDN)                                         |
|                 - Edge caching of static assets (<2MB gzipped JS/CSS shell)                             |
|                 - Auth token validation (Synthetic UUID routing, Zero PII exposure)                    |
+---------------------------------------------------------------------------------------------------------+
       |                                                 |                                   |
       v                                                 v                                   v
+-------------------------------+             +-----------------------------+     +-----------------------+
|         Auth Service          |             |       State Snapshot        |     |  Event Ingestion API  |
| - Magic Link Generation       |             |         Cache (Redis)       |     | - Ingest presence     |
| - Session Token Management    |             | - Active aviary states      |     | - Ingest offers/listen|
| - 30-Day Soft Deletion        |             | - Sub-millisecond reads     |     | - Validate host token |
+-------------------------------+             +-----------------------------+     +-----------------------+
       |                                                 ^                                   |
       v                                                 |                                   v
+---------------------------------------------------------------------------------------------------------+
|                                    Simulation Service Worker Cluster                                    |
|                                                                                                         |
|   - Autorun tick (~60s cadence per active aviary)                                                       |
|   - Consumes append-only interaction events                                                             |
|   - Executes monotonic personality drift math (low-pass filter)                                         |
|   - Updates mood state machine & perch positions                                                        |
|   - Evaluates aviary age milestones for bird unlocks                                                    |
|   - Generates sparse naturalist Field Notebook entries & Screen-Reader Narration prose                  |
|   - Publishes updated canonical snapshots to Redis & PostgreSQL                                         |
+---------------------------------------------------------------------------------------------------------+
       |                                                                                     |
       v                                                                                     v
+-------------------------------------------------------------+     +-------------------------------------+
|             Primary Database (PostgreSQL 16)                |     |     Aggregate Telemetry (Datadog)   |
| - Encrypted Accounts (Synthetic UUID keys)                  |     | - Tick latency (p99 < 5s alarm)     |
| - Aviary & Bird State Records                               |     | - Frame timing & bundle budgets     |
| - Append-Only Interaction Event Log                         |     | - Zero per-bird / per-account data  |
| - Field Notebook & Visit Ledger                             |     +-------------------------------------+
+-------------------------------------------------------------+
```

### 2.1 Service Topology
1. **Frontend Client (Single-Page Application)**:
   - Zero-dependency Vanilla JS core for ultra-lean bundle size (<2MB gzipped).
   - High-performance Canvas 2D / WebGL rendering layer.
   - Procedural WebAudio synthesis engine with graph pooling.
   - Live accessibility manager (`aria-live` narration, call caption overlays, focus manager).
2. **API Gateway & Auth Service**:
   - Handles passwordless magic-link authentication, session issuance, and token revocation.
   - Translates incoming session tokens into synthetic `account_id` UUIDs. Email addresses are encrypted at rest using AES-256-GCM and never propagate downstream into worker logs, metrics, or telemetry.
3. **Simulation Service (Worker Fleet)**:
   - Background orchestrator running the 60-second simulation tick for all active aviaries.
   - Processes interaction logs sequentially, updates hidden personality vectors, transitions moods, checks age-based unlocks, and generates naturalist prose.
4. **Primary Store & In-Memory Cache**:
   - **PostgreSQL 16**: Relational storage for accounts, aviaries, birds, notebook entries, and append-only event logs.
   - **Redis 7**: Pub/sub snapshot broker and low-latency cache for state snapshots pulled by connected clients.

---

## 3. Data Model & Database Schemas

All internal references use synthetic UUIDv4 keys. No table outside `accounts` stores or references user email addresses.

```sql
-- Core Accounts Table
CREATE TABLE accounts (
    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_hash VARCHAR(64) UNIQUE NOT NULL, -- HMAC-SHA256 for lookup without decryption
    status VARCHAR(24) NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'pending_deletion', 'deleted')),
    deletion_requested_at TIMESTAMPTZ NULL,
    visit_notifications_enabled BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Active User Sessions
CREATE TABLE sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    device_name VARCHAR(64) NOT NULL,
    session_token_hash VARCHAR(64) UNIQUE NOT NULL,
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at TIMESTAMPTZ NULL
);

-- Canonical Aviary Table
CREATE TABLE aviaries (
    aviary_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID UNIQUE NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    is_settled BOOLEAN NOT NULL DEFAULT FALSE,
    settled_at TIMESTAMPTZ NULL,
    current_weather VARCHAR(24) NOT NULL DEFAULT 'clear' CHECK (current_weather IN ('clear', 'light_rain', 'breeze')),
    weather_expires_at TIMESTAMPTZ NULL,
    last_ticked_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Birds Table
CREATE TABLE birds (
    bird_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL CHECK (species_id IN ('sparrow_pip', 'wren_moss', 'finch_gold', 'warbler_willow', 'thrush_speckle', 'nightjar_dusk')),
    name VARCHAR(32) NOT NULL,
    -- Normalized scalar traits [0.0000 to 1.0000], strictly hidden from UI
    boldness NUMERIC(5, 4) NOT NULL DEFAULT 0.2000 CHECK (boldness BETWEEN 0.0 AND 1.0),
    social_warmth NUMERIC(5, 4) NOT NULL DEFAULT 0.2000 CHECK (social_warmth BETWEEN 0.0 AND 1.0),
    vocal_frequency NUMERIC(5, 4) NOT NULL DEFAULT 0.2000 CHECK (vocal_frequency BETWEEN 0.0 AND 1.0),
    plumage_saturation NUMERIC(5, 4) NOT NULL DEFAULT 0.2000 CHECK (plumage_saturation BETWEEN 0.0 AND 1.0),
    curiosity NUMERIC(5, 4) NOT NULL DEFAULT 0.2000 CHECK (curiosity BETWEEN 0.0 AND 1.0),
    -- Fast-timescale mood state
    current_mood VARCHAR(24) NOT NULL DEFAULT 'content' CHECK (current_mood IN ('wary', 'content', 'curious', 'drowsy', 'alert')),
    mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    perch_zone VARCHAR(16) NOT NULL DEFAULT 'middle' CHECK (perch_zone IN ('front', 'middle', 'back')),
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Append-Only Interaction Event Log (Authoritative Input to Simulation Tick)
CREATE TABLE interaction_events (
    event_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    session_id UUID NOT NULL REFERENCES sessions(session_id),
    event_type VARCHAR(32) NOT NULL CHECK (event_type IN ('presence_ping', 'listen_in_start', 'listen_in_end', 'offer_seed', 'offer_song', 'offer_pool', 'settle', 'unsettle')),
    target_bird_id UUID NULL REFERENCES birds(bird_id),
    payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_by_tick_at TIMESTAMPTZ NULL
);
CREATE INDEX idx_events_aviary_unprocessed ON interaction_events(aviary_id, created_at) WHERE processed_by_tick_at IS NULL;

-- Field Notebook Observations Log
CREATE TABLE notebook_entries (
    entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    observation_text TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_notebook_aviary_date ON notebook_entries(aviary_id, created_at DESC);

-- Visit Invitations Table
CREATE TABLE visit_invitations (
    invite_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    visitor_email_encrypted BYTEA NOT NULL,
    invite_token_hash VARCHAR(64) UNIQUE NOT NULL,
    status VARCHAR(24) NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'accepted', 'revoked', 'expired')),
    expires_at TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Host-Visible Visit Log
CREATE TABLE visit_logs (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    visitor_email_masked VARCHAR(64) NOT NULL, -- e.g. "a***@domain.com"
    visit_started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    duration_seconds INTEGER NOT NULL DEFAULT 0
);
```

---

## 4. API Surface & Protocols

All error responses and system endpoints adhere to the matter-of-fact tone. All client-to-server timestamps are ISO 8601 UTC.

### 4.1 Authentication & Account Management
- `POST /api/v1/auth/magic-link`
  - Body: `{"email": "user@example.com"}`
  - Logic: Rate-limited (max 3 per 15 min per IP/email). Generates 15-minute cryptographically secure single-use token. Emails link.
  - Response (200 OK): `{"message": "If that address is valid, a sign-in link is on its way."}`
- `POST /api/v1/auth/verify`
  - Body: `{"token": "..."}`
  - Response (200 OK): Sets `HttpOnly; Secure; SameSite=Strict` session cookie. Returns initial aviary state bootstrap.
  - Failure (401 / 410): `{"error": "We couldn't sign you in. The link may have expired. Try requesting a new link."}`
- `POST /api/v1/auth/session/revoke`
  - Body: `{"session_id": "..."}`
  - Response (200 OK): `{"status": "session_revoked"}`
- `POST /api/v1/account/export`
  - Logic: Compiles full JSON snapshot of account data (birds, traits, notebook, history) and sends a short-lived download link to the verified email.
  - Response (200 OK): `{"message": "Your aviary export has been queued and will be sent to your email."}`
- `DELETE /api/v1/account`
  - Logic: Transitions status to `pending_deletion`, sets `deletion_requested_at = NOW()`. Hard purge job executes after 30 days.
  - Response (200 OK): `{"message": "Your account has been scheduled for deletion in 30 days. You can sign in anytime before then to cancel."}`

### 4.2 Aviary State & Interaction Streams
- `GET /api/v1/aviary/state`
  - Purpose: Full snapshot retrieval on initial load, tab visibility restoration, or recovery from network disruption.
  - Response (200 OK):
  ```json
  {
    "aviary_id": "8f3b2d1e-...",
    "server_time": "2026-08-13T17:36:00Z",
    "is_settled": false,
    "weather": "clear",
    "lighting_phase": "morning",
    "birds": [
      {
        "bird_id": "b1a2c3d4-...",
        "species_id": "sparrow_pip",
        "name": "pip",
        "perch_zone": "front",
        "current_mood": "content",
        "plumage_saturation": 0.3421,
        "motion_state": {
          "pose": "preening",
          "phase_offset_ms": 1420
        }
      },
      {
        "bird_id": "b2c3d4e5-...",
        "species_id": "wren_moss",
        "name": "wren",
        "perch_zone": "middle",
        "current_mood": "wary",
        "plumage_saturation": 0.2810,
        "motion_state": {
          "pose": "scanning",
          "phase_offset_ms": 850
        }
      }
    ],
    "narration_prose": "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
  }
  ```
- `POST /api/v1/aviary/events`
  - Purpose: Batch append-only ingestion of client interactions.
  - Body:
  ```json
  {
    "events": [
      {
        "event_type": "presence_ping",
        "payload": {
          "duration_seconds": 60,
          "visibility_state": "visible",
          "has_focus": true,
          "recent_user_activity": true
        }
      },
      {
        "event_type": "offer_seed",
        "target_bird_id": "b1a2c3d4-...",
        "payload": { "timestamp": "2026-08-13T17:36:12Z" }
      }
    ]
  }
  ```
  - Validation: Strictly verifies `session_id`. If the session is a read-only visitor, the endpoint rejects the event with HTTP 403.

### 4.3 Social (Visit Invitations)
- `POST /api/v1/invites`
  - Body: `{"visitor_email": "friend@example.com"}`
  - Logic: Generates unique 30-day token. Encrypts visitor email. Dispatches invitation link.
- `DELETE /api/v1/invites/:invite_id`
  - Logic: Sets status to `revoked`. Instantly blocks active snapshot pulls using that token.
- `GET /api/v1/visit/:token/state`
  - Response (200 OK): Returns the host's canonical aviary state snapshot with `read_only: true`.
  - Failure / Revocation (403 Forbidden / 410 Gone):
    `{"error": "This visit is no longer available."}`

---

## 5. Simulation Engine & Mathematical Models

The server-side simulation tick runs every 60 seconds per aviary. It is the sole authoritative writer of bird state.

```
                           +-----------------------------------------------+
                           |          60-Second Server Tick Trigger        |
                           +-----------------------------------------------+
                                                   |
                                                   v
                           +-----------------------------------------------+
                           | 1. Ingest Pending Interaction Event Log       |
                           |    - Filter out non-host events               |
                           |    - Validate presence conjunction criteria   |
                           +-----------------------------------------------+
                                                   |
                                                   v
                           +-----------------------------------------------+
                           | 2. Calculate Validated Presence Time ($T_p$)  |
                           |    - Sum intervals where:                     |
                           |      (visible && focused && active) == TRUE   |
                           +-----------------------------------------------+
                                                   |
                                                   v
                           +-----------------------------------------------+
                           | 3. Apply Monotonic Personality Drift Math     |
                           |    - $\Delta \text{trait} = \alpha \cdot \ln(1 + \beta \cdot \text{input})$   |
                           |    - $\Delta \ge 0$ strictly enforced         |
                           +-----------------------------------------------+
                                                   |
                                                   v
                           +-----------------------------------------------+
                           | 4. Evaluate Fast-Timescale Mood Machine       |
                           |    - Time-of-day solar bias                   |
                           |    - Interaction nudges (offers / listen-in)  |
                           |    - Weather & bird-to-bird alarm contagion   |
                           +-----------------------------------------------+
                                                   |
                                                   v
                           +-----------------------------------------------+
                           | 5. Update Perch Zones & Bird Unlocks          |
                           |    - Perch choice = $f(\text{mood}, \text{boldness})$         |
                           |    - Age-based population ramp (up to 7 birds)|
                           +-----------------------------------------------+
                                                   |
                                                   v
                           +-----------------------------------------------+
                           | 6. Naturalist Prose & Narration Synthesis     |
                           |    - Sparse Field Notebook entries (~2-4 days)|
                           |    - Screen-Reader scene description (30-60s) |
                           +-----------------------------------------------+
                                                   |
                                                   v
                           +-----------------------------------------------+
                           | 7. Commit to DB & Broadcast Cache Snapshot    |
                           +-----------------------------------------------+
```

### 5.1 Presence Accounting Engine
Presence time ($T_p$) is credited **only** when all three conditions hold simultaneously:
$$\text{Presence Valid} \iff (\text{visibilityState} \equiv \text{visible}) \land (\text{window.hasFocus}() \equiv \text{true}) \land (\Delta t_{\text{last\_input}} \le 180\text{s})$$
If any condition fails, $T_p = 0$. This prevents background tabs or forgotten overnight sessions from falsely saturating drift.

### 5.2 Monotonic Personality Drift Function
Personality traits are scalar values normalized to $[0.0, 1.0]$. Drift operates as an asymmetric, monotonic low-pass filter:
$$\text{Trait}_{t+1} = \text{Trait}_t + \Delta \text{Trait}$$
$$\Delta \text{Trait} = \min\left( (1.0 - \text{Trait}_t) \cdot \alpha_{\text{trait}} \cdot \ln\left(1 + \beta \cdot \sum \text{Input}\right), \; \Delta_{\max} \right)$$
$$\text{Constraint: } \Delta \text{Trait} \ge 0 \quad (\text{Strictly Monotonic})$$

Trait Weight Parameters:
- **Boldness**: $\text{Input} = 0.6 \cdot T_p + 0.4 \cdot N_{\text{offers\_near}}$
- **Social Warmth**: $\text{Input} = 0.5 \cdot T_p + 0.35 \cdot T_{\text{listen\_in}} + 0.15 \cdot N_{\text{chorus}}$
- **Vocal Frequency**: $\text{Input} = 0.4 \cdot T_p + 0.6 \cdot T_{\text{listen\_in}}$
- **Plumage Saturation**: $\text{Input} = 1.0 \cdot T_p$
- **Curiosity**: $\text{Input} = 0.7 \cdot N_{\text{accepted\_offers}} + 0.3 \cdot N_{\text{motif\_responses}}$

Calibration Targets:
- $\alpha_{\text{trait}}$ is tuned such that 1 week of daily 15-minute visits yields $\Delta \approx +0.015$ (measurable in instrument tests).
- 3 weeks of daily visits yields $\Delta \approx +0.06$ (clearly noticeable in bird perching distance, plumage depth, and greeting speed).
- Neglect ($T_p = 0$) results in $\Delta = 0$. Traits never decay.

### 5.3 Fast-Timescale Mood State Machine
Mood is evaluated every tick across 5 states: `wary`, `content`, `curious`, `drowsy`, `alert`.
- **Transitions**:
  - `alert` $\to$ `content`: Occurs naturally over 5–10 minutes in the absence of startling stimuli.
  - `content` $\to$ `curious`: Triggered by active offers or nearby bird vocalizations.
  - Any $\to$ `wary`: Triggered by sudden background events or neighboring bird alarm calls. Transition probability is dampened by high `boldness`:
    $$P(\text{wary}) = P_{\text{base}} \cdot (1.0 - 0.7 \cdot \text{boldness})$$
  - Any $\to$ `drowsy`: Governed by local solar time (dusk/evening) or an active `settle` event.
- **Session Continuity**: Mood states persist in the database between sessions. On opening the tab, the bird displays its exact stored mood as updated by interim server ticks.

### 5.4 Procedural Field Notebook Generator
- **Cadence**: Sparsely triggered (once every 2 to 4 active days, or upon significant inflection points such as a change in the first-greeter bird or an emergent three-bird chorus).
- **Prose Engine**: Built from a deterministic naturalist template grammar:
  - *Greeter shift*: `"{bird_a} greeted before {bird_b} today — only by a beat, but first."`
  - *Calm idle*: `"{bird} is fluffed against the cool air, watching the back perch. low calls only."`
  - *Weather event*: `"soft rain through the canopy this afternoon. {bird} tucked low on the middle rail."`

---

## 6. Multi-Device Sync & Conflict Model

```
+---------------------------+                         +---------------------------+
|      Device A (Laptop)    |                         |       Device B (Phone)    |
+---------------------------+                         +---------------------------+
       |                                                             |
       | 1. Append Event:                                            | 2. Append Event:
       |    listen_in(Pip, 120s)                                     |    offer_seed(Wren)
       v                                                             v
+---------------------------------------------------------------------------------+
|               Authoritative Append-Only Event Log (PostgreSQL)                  |
|               - Ordered by server reception timestamp                           |
+---------------------------------------------------------------------------------+
                                         |
                                         | 3. Simulation Tick processes events
                                         v
+---------------------------------------------------------------------------------+
|                       Authoritative Canonical State                             |
|                       - Additive personality deltas calculated                  |
|                       - No Last-Write-Wins overwrites                           |
+---------------------------------------------------------------------------------+
                                         |
                                         | 4. Publish Snapshot
                                         v
+---------------------------------------------------------------------------------+
|                        Snapshot Cache & Broadcaster                             |
+---------------------------------------------------------------------------------+
       |                                                             |
       v                                                             v
[Device A interpolates state]                                 [Device B interpolates state]
```

1. **Zero Client Authority on State**:
   - Clients never send absolute property values (e.g., `set boldness = 0.5`).
   - Clients only submit validated interaction events to the append-only log.
2. **Elimination of Last-Write-Wins Hazards**:
   - Because the server simulation tick is the sole entity calculating and applying additive deltas, concurrent sessions on multiple devices cannot overwrite or erase accumulated drift history.
3. **Client Interpolation & Teleportation Prevention**:
   - Snapshots provide target perch coordinates, mood states, and skeletal animation parameters.
   - The frontend rendering loop smoothly lerps bird coordinates across a 1200ms transition window. If a bird relocated perches between snapshots, it performs a natural flight glide rather than snapping.
4. **Disconnection & Re-Sync**:
   - On tab visibility restore (`visibilitychange` $\to$ `visible`) or after a sleep resume, the client fetches `/api/v1/aviary/state` and smoothly merges the active rendering pipeline with the updated server state.

---

## 7. Frontend Rendering Pipeline

```
+---------------------------------------------------------------------------------+
|                     Single Horizontal Canvas (Fluid Responsive)                 |
|                                                                                 |
|  [Top Bar Chrome: Settings | Accessibility | Field Notebook | Offer Affordance] |
|  (Fades to 0.05 opacity after 3s of inactivity; restored on movement)           |
|                                                                                 |
|  +---------------------------------------------------------------------------+  |
|  | Layer 0: Dynamic Sky Gradient (Local solar time calculation)              |  |
|  +---------------------------------------------------------------------------+  |
|  | Layer 1: Background Perch Zone & Distant Foliage (0.3x Parallax)          |  |
|  +---------------------------------------------------------------------------+  |
|  | Layer 2: Middle Perch Zone & Primary Foliage (0.6x Parallax)              |  |
|  +---------------------------------------------------------------------------+  |
|  | Layer 3: Foreground Perches, Still Pool & Seed Landing Surface            |  |
|  +---------------------------------------------------------------------------+  |
|  | Layer 4: Ambient Particles (Drifting leaves & falling feathers)           |  |
|  +---------------------------------------------------------------------------+  |
|  | Layer 5: Foreground Canopy Framing & Vignette (1.0x Parallax)             |  |
|  +---------------------------------------------------------------------------+  |
|                                                                                 |
+---------------------------------------------------------------------------------+
```

### 7.1 Scene Composition & Viewport Rules
- **Single Horizontal Viewport**: The entire aviary is visible without scrolling, panning, or zooming.
- **Responsive Aspect Scaling**: The canvas maintains a logical 16:9 bounding scene, adapting gracefully from 320px mobile viewports up to 4K ultra-wide monitors. On narrow screens, perch zones compress horizontally with dynamic padding to ensure no bird is ever cropped out of frame.
- **Zero In-Scene Chrome**: No health bars, name tags, hover tooltips, or interaction icons exist within the aviary canvas.

### 7.2 First-Frame Loading Architecture
- **No Spinners or Blank Screens**: Spinners destroy the illusion of a self-sustaining habitat.
- **Bootstrap Shell**: If network latency delays the state snapshot, the canvas instantly paints a calming morning sky gradient (the "quiet field").
- **Mid-Motion Initialization**: Birds are initialized with randomized phase offsets on their idle procedural motion cycles, ensuring they appear mid-preen or mid-scan on the very first frame painted.

### 7.3 Idle Micro-Motion Engine
Idle animations are procedural bone-and-mesh transformations calculated per frame:
- **Preening**: Periodic feather ruffle and beak-to-wing rotation curves.
- **Scanning**: Randomized saccadic head turns and cocks, with angular velocity governed by bird `boldness` and `current_mood`.
- **Weight Shuffle**: Subtle vertical offset and footing adjustments every 4–8 seconds.
- **Breathing**: Continuous sinusoidal scale oscillation (12–18 cycles per minute).

### 7.4 Designed Reduced-Motion Mode
When `prefers-reduced-motion: reduce` is detected or toggled via accessibility settings:
- Frame-by-frame skeletal animations are replaced by soft cross-fades (0.8s opacity transitions) between key resting poses.
- Perch relocations cross-fade directly across perches rather than rendering flight trajectories across the screen.
- Ambient leaf and feather particles are disabled.
- Day/night lighting transitions are extended over 10-second smooth palette blends.

### 7.5 Top-Bar Chrome Fade
The top bar UI sits in a distinct HTML layer above the canvas. After 3.0 seconds of cursor or keyboard inactivity, CSS transitions fade the container to `opacity: 0.05`. Any mouse movement, touch, or keyboard event instantly restores `opacity: 1.0` within 150ms.

---

## 8. WebAudio Procedural Synthesis Pipeline

```
+---------------------------------------------------------------------------------+
|                       Procedural WebAudio Synthesizer Graph                     |
+---------------------------------------------------------------------------------+
|                                                                                 |
|  [Species Motif Grammar Engine]                                                 |
|         |                                                                       |
|         +---> Dual Sine/Triangle FM Oscillators (Chirp & Whistle)               |
|         |        |                                                              |
|         +---> Formant Filter (Bandpass Q=12, Species Resonances)                |
|         |        |                                                              |
|         +---> Dynamic Gain Envelope (ADSR Curve per Motif Note)                 |
|                  |                                                              |
|                  v                                                              |
|         [Per-Bird Channel Strip]                                                |
|         - Channel Gain Node (Governed by Listen-In mix state)                   |
|         - Stereo Panner Node (Mapped to Canvas X position: [-0.85, +0.85])      |
|                  |                                                              |
|                  v                                                              |
|         +-------------------------------------------------------------+         |
|         |               Master Aviary Audio Bus                       |         |
|         | - Convolver Node (Subtle outdoor acoustic impulse response) |         |
|         | - Master Limiter & Dynamic Range Compressor                 |         |
|         +-------------------------------------------------------------+         |
|                  |                                                              |
|                  v                                                              |
|           [AudioContext.destination (Speakers / Headphones)]                    |
|                                                                                 |
+---------------------------------------------------------------------------------+
```

### 8.1 Procedural Synthesis vs. Audio Loops
To satisfy the <2MB bundle budget and prevent audio fatigue, zero recorded audio files are packaged. Calls are synthesized in real-time using the WebAudio API:
1. **Oscillator Structure**: Two primary low-aliasing oscillators (sine and triangle) frequency-modulated to produce organic bird vocal tract harmonics.
2. **Motif Grammar**: Each of the 6 species has 4–8 procedural motifs (combinations of frequency sweeps, trills, glissandi, and chirps).
3. **Parameter Modulation**: Motifs are randomized at runtime based on `personality_vector.vocal_frequency` (interval between motifs) and `current_mood` (pitch modulation, vibrato rate, duration).

### 8.2 Dynamic Chorus Management
When multiple birds vocalize simultaneously:
- The system staggers note triggers by pseudo-random offsets (120ms – 450ms) to avoid unnatural acoustic phase cancellation.
- Master dynamic range compression prevents clipping and harmonizes the multi-bird chorus.

### 8.3 Listen-In Interaction Mix Curves
When a user listens in on a bird (via pointer focus or keyboard selection):
- **Focused Bird Channel**: Channel gain rises smoothly from $0\text{ dB}$ to $+4\text{ dB}$ over 800ms using an equal-power exponential ramp (`linearRampToValueAtTime`).
- **Other Bird Channels**: Ambient birds decay smoothly from $0\text{ dB}$ to $-9\text{ dB}$ over 800ms. They remain distinctly audible at an ambient level and are never hard-muted.
- **Disengage**: Mix curves restore to baseline parity over a 1200ms decay window.

### 8.4 WebAudio Fallback Mode
If `AudioContext` fails to initialize (unsupported browser, blocked autoplay policies, or hardware unavailability):
- The audio engine degrades gracefully to complete silence.
- **Call Captions** automatically activate in the visual interface.
- **Strict Non-Goal**: Under no circumstances does the engine fall back to looped audio files.

---

## 9. Accessibility Surfaces

Accessibility is designed as a first-class, charm-preserving surface rather than a compliance checklist.

### 9.1 Screen-Reader Narration (`aria-live`)
- **Semantic Structure**: A dedicated hidden DOM container configured with `aria-live="polite"` and `aria-atomic="true"`.
- **Narration Engine**: Emits naturalist, lowercase prose summaries every 30–60 seconds during idle sessions:
  > *"a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."*
- **Interaction Priority**: User actions (return greetings, accepted offers, settle commands) preempt the queue with observation-style prose (e.g., *"pip stepped to the front rail to investigate the seed"*), strictly avoiding system-style state announcements.

### 9.2 Real-Time Procedural Call Captions
- **Visual Presentation**: High-contrast, unobtrusive text pills positioned adjacent to the vocalizing bird.
- **Grammar Generation**: Captions are dynamically derived from the procedural motif playing in the audio engine (e.g., *"a soft three-note rise"*, *"a low trill, paused, low trill again"*, *"a single sharp call from the back perch"*).
- **Transitions**: Synchronized with the audio envelope, fading in over 150ms and fading out after the motif finishes.

### 9.3 Keyboard Navigation & Focus Flow
- **Tab Navigation**:
  1. Top Bar Controls: Settings $\to$ Accessibility $\to$ Field Notebook $\to$ Offer Menu.
  2. Aviary Canvas: Focuses the first bird on the front perch.
- **Arrow Keys**: Move focus between birds logically (Left/Right along perches; Up/Down between Front/Middle/Back zones).
- **Enter / Space**: Engages `listen-in` on the focused bird.
- **Escape**: Disengages `listen-in` or closes top-bar modals.
- **Focus Indicators**: Double-ring high-contrast focus outlines (inner white 2px, outer dark brown 2px) ensuring distinct visibility across both bright morning and dim evening sky palettes.

### 9.4 Contrast Compliance
All UI copy, top-bar icons, modal dialogs, and caption overlays achieve a minimum contrast ratio of 4.8:1 against their underlying canvas backgrounds, surpassing the WCAG AA 4.5:1 requirement.

---

## 10. Performance Budgets, Telemetry & Privacy

### 10.1 Strict Performance Budgets

| Metric | Budget Target | Implementation Strategy |
|---|---|---|
| **Initial JS Bundle Size** | $< 2.0\text{ MB}$ (Gzipped) | Zero large frameworks; Vanilla JS core; SVGs; procedural WebAudio; dynamic `import()` for settings/notebook modals. |
| **Time to First Bird Visible** | $< 500\text{ ms}$ (4G Mobile) | Edge-cached HTML shell; pre-rendered canvas background; sub-50ms snapshot retrieval from Redis. |
| **Runtime Frame Rate** | $60\text{ fps}$ continuous | Lightweight 2D canvas draw routines; zero DOM recreation in main loop; object pooling for particles. |
| **Memory Allocation** | $\Delta\text{Heap} \equiv 0\text{ MB}$ over 30 min | Reusable WebAudio node pools; pre-allocated particle arrays; recycled list elements in notebook. |
| **Simulation Tick Latency** | $\text{p99} < 5.0\text{ s}$ | Batched PostgreSQL queries; indexed event logs; asynchronous worker processing pool. |

### 10.2 Observability vs. Privacy Enforcement Boundary

```
[Production Systems]
       |
       +---> [Authoritative Database] 
       |        - Contains: Birds, Personality Vectors, Moods, Notebook Entries, Presence Pings
       |        - STRICT ISOLATION: Never exported to analytics, BI warehouses, or ML pipelines.
       |
       +---> [Aggregate Telemetry Pipeline (Datadog / OpenTelemetry)]
                - Allowed: Request rates, HTTP status codes, DB query latencies, tick runtimes,
                           client FPS percentiles, WebAudio initialization error counts.
                - FORBIDDEN: User IDs, synthetic account IDs, bird names, traits, moods,
                             interaction frequencies, session durations per user.
```

- **Zero PII in Logs**: Synthetic UUIDs are used for inter-service communication. Email addresses are encrypted and strictly prohibited from log lines and partition keys.
- **Zero Behavioral Tracking**: No event telemetry tracks individual bird preferences or user habits.

---

## 11. Rollout & Population Ramp Plan

### 11.1 Aviary Population Scaling Milestones
To preserve the per-bird vocal recognizability that underpins the product's emotional relationship, aviaries expand solely through age milestones:

```
+---------------------------------------------------------------------------------+
| Aviary Age: Day 1        --> 2 Starter Birds (Assigned from 6-Species Pool)     |
| Aviary Age: 3 Months     --> 3rd Bird Arrives                                   |
| Aviary Age: 6 Months     --> 4th Bird Arrives                                   |
| Aviary Age: 9 Months     --> 5th Bird Arrives                                   |
| Aviary Age: 12 Months    --> 6th Bird Arrives                                   |
| Aviary Age: 15+ Months   --> 7th Bird Arrives (Hard Ceiling Cap)                |
+---------------------------------------------------------------------------------+
```

### 11.2 Engineering Execution Phases

```
+---------------------------------------------------------------------------------+
| Phase 1: Core Engine Foundations (Weeks 1–3)                                    |
| - Canvas 2D responsive scene & parallax layering                                |
| - Procedural WebAudio synthesizer for 6 bird species                            |
| - Bone-and-mesh idle micro-motion engine                                        |
+---------------------------------------------------------------------------------+
                                      |
                                      v
+---------------------------------------------------------------------------------+
| Phase 2: Autorun Simulation & Persistence (Weeks 4–6)                           |
| - PostgreSQL schema setup with synthetic UUID architecture                      |
| - Server-side 60s simulation tick & monotonic drift calculator                  |
| - Event log ingestion & snapshot publisher (Redis)                              |
+---------------------------------------------------------------------------------+
                                      |
                                      v
+---------------------------------------------------------------------------------+
| Phase 3: Interaction Mechanics & Accessibility (Weeks 7–9)                      |
| - Procedural return-greeting logic (absence-length & boldness keyed)           |
| - Listen-in audio rebalance, offers cooldown engine, settle gesture             |
| - Screen-reader live narration prose generator & call captioning                |
| - Designed reduced-motion cross-fade mode                                       |
+---------------------------------------------------------------------------------+
                                      |
                                      v
+---------------------------------------------------------------------------------+
| Phase 4: Auth, Social & Synchronization (Weeks 10–11)                           |
| - Passwordless magic-link authentication & session manager                      |
| - Multi-device snapshot pull & smooth lerp reconciliation                       |
| - 1:1 read-only visit invitations, revocation, and silent visit log             |
| - Account JSON export & 30-day soft deletion lifecycle                          |
+---------------------------------------------------------------------------------+
                                      |
                                      v
+---------------------------------------------------------------------------------+
| Phase 5: Hardening, Performance & Privacy Verification (Weeks 12–13)           |
| - Bundle size budgeting (<2MB gzip audit)                                       |
| - 30-minute memory leak profiling in automated headless CI                      |
| - Telemetry data scrubbing audit (confirm zero per-bird leaks)                  |
| - Production canary rollout                                                     |
+---------------------------------------------------------------------------------+
```

---

## 12. Engineering Risks & Mitigations

### 12.1 Drift Function Miscalibration
- **Risk**: Drift tuning that is too aggressive turns the aviary into a game where numbers are manipulated; drift that is too sluggish feels like an unchanging screensaver.
- **Mitigation**: Implement automated simulation test suites running 1,000 synthetic days of varied presence profiles (intermittent, daily, neglected). Validate that 1 week of visits produces measurable instrument drift ($\Delta \approx +0.015$) and 3 weeks produces visible behavioral shifts ($\Delta \approx +0.060$) without trait saturation or decay.

### 12.2 Audio Uncanniness & Repetition Fatigue
- **Risk**: Procedural WebAudio synthesis could sound sterile, metallic, or robotic compared to biological bird calls.
- **Mitigation**: Employ dual-oscillator FM synthesis coupled with bio-acoustic formant filtering (Q=12 resonance curves) and micro-randomized pitch vibrato envelopes. Ensure note intervals and frequency sweeps vary slightly on every invocation.

### 12.3 Multi-Device Presence Race Conditions
- **Risk**: Simultaneous active tabs on multiple devices submitting concurrent presence heartbeats, artificially accelerating drift.
- **Mitigation**: The simulation tick groups incoming `presence_ping` events by overlapping wall-clock time intervals rather than naively summing submitted durations. Presence credited in any 60-second tick is capped at exactly 60 seconds regardless of the number of open devices.

### 12.4 WebAudio Node Leaks During Extended Sessions
- **Risk**: Creating ephemeral oscillator and gain nodes for procedural calls can cause garbage collection churn and memory bloat over 30-minute sessions.
- **Mitigation**: Maintain a fixed, pre-allocated pool of WebAudio nodes (20 reusable channel strips). When a motif finishes playing, disconnect the nodes and return them to the idle pool without instantiating new objects. Verify in CI using automated 30-minute memory leak regression tests.
