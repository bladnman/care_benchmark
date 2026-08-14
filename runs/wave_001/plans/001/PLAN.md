# Implementation Plan: Pocket Aviary

## 1. Executive Summary & Product Scope

Pocket Aviary is a quiet, browser-based virtual aviary wherein users adopt two to seven procedurally animated and vocalized birds that inhabit a single horizontal window. Over days and weeks, the birds' hidden personality vectors drift monotonically toward expressive aliveness in response to user presence and subtle interactions. The system is designed to provide a low-key, companionable relationship rather than a gamified or custodial experience.

### 1.1 In-Scope for v1
* **Single-User Accounts:** Authenticated via email magic links (15-minute expiration, single-use, rate-limited).
* **Canonical Multi-Device Aviary:** Exactly one aviary per account, simulated continuously server-side and rendered identically across desktop and mobile browsers.
* **Bird Engine & Population:** Starter experience with 2 system-selected birds from a 6-species pool, unlocking up to 7 birds paced strictly by aviary age (calendar duration). User-defined custom bird names with persistent internal UUIDs.
* **Behavioral Dynamics:** Hidden 5-dimensional personality vector with monotonic slow-timescale drift, fast-timescale 5-state mood transitions, procedural idle micro-motion, and procedural call generation via WebAudio.
* **Core Interactions:** Return-greeting (procedural, absence-scaled, staggered), Listen-in (focus mixing with smooth gain transitions), Offers (seed, song fragment, still pool with per-bird cooldowns), Settle gesture (evening lighting shift, quiet audio, 5-second cancel window), Presence accounting (strict 3-factor conjunction).
* **Field Notebook:** Automated, sparse, read-only naturalist observation log generated in lowercase, present-tense naturalist prose.
* **Social Visits:** Private, opt-in, email-invited, read-only ambient visits. 30-day token lifetime, instant revocation, silent visit logging, zero co-presence, zero visitor drift influence.
* **Accessibility Surfaces:** Continuous naturalist screen-reader narration (30–60s cadence via `aria-live="polite"`), procedural call captions, full keyboard navigation with high-contrast outlines, and a distinct Reduced-Motion mode using slow pose cross-fades.
* **Performance & Platform:** Single-page web application (PWA-ready web bundle <2MB gzipped), Time-to-First-Bird <500ms on 4G, 60fps rendering on 5-year-old hardware, zero memory growth over 30-minute sessions.

### 1.2 Out-of-Scope Non-Goals (Strict Guardrails)
* **No Native Applications:** Web-only; no iOS/Android client wrappers or platform-specific builds.
* **No Gamification:** Zero scoreboards, streaks, levels, XP, badges, counters ("birds adopted: 2"), visit calendars, or milestone popups.
* **No Tamagotchi Mechanics:** No bird death, hunger meters, sickness, neglect penalties, visible distress, or custodial chore loops. Absence produces ambient quietness without negative trait degradation.
* **No Social Network Mechanics:** No user profiles, follows, public directories, discovery feeds, leaderboards, visit comments, shared cursors, or live multiplayer overlays.
* **No Push/Notification Engagement Loops:** No push notifications, no re-engagement emails ("Your birds miss you"), and no "friend visited" notifications by default.
* **No In-Aviary UI Chrome or Text Banners:** No "Welcome back!" toasts, level banners, or inline HUD widgets over the aviary scene.

### 1.3 Voice & Tone Boundary
* **Naturalist Voice (Product Surface):** Lowercase by default, present-tense, specific, observational, devoid of exclamation marks, gamification terms, or second-person commands. Used in aviary rendering, notebook entries, screen-reader narration, call captions, and offer interactions.
* **Matter-of-Fact Voice (System Surface):** Standard grammatical capitalization, clear, direct, neutral, and devoid of faux-warmth. Used strictly for magic-link authentication, session expiration, sync errors, unsupported browser warnings, invite revocation notices, and account settings.

---

## 2. Architecture & Service Topology

The Pocket Aviary platform employs a decoupled, snapshot-and-event-stream architecture. The server owns the single source of truth and advances the aviary state via an asynchronous simulation loop, while thin web clients render procedural visuals and audio from state snapshots.

```
                         +-----------------------------------+
                         |           CDN / Edge              |
                         |  (Static Assets, HTML Bootstrapper)|
                         +-----------------+-----------------+
                                           |
                                           | HTTPS / WSS
                                           v
+-----------------------------------------------------------------------------------+
|                                  API Gateway                                      |
|            (Reverse Proxy, Magic-Link Auth, Rate Limiting, Route Guard)           |
+---------------------+-----------------------------+-------------------------------+
                      |                             |
                      v                             v
       +------------------------------+   +------------------------------+
       |         HTTP Services        |   |       WebSocket Server       |
       | - Auth & Account Management  |   | - Live Snapshot Broadcast    |
       | - Append-Only Event Ingest   |   | - Connection Heartbeat       |
       | - Field Notebook & Export    |   | - Presence Ping Consumer     |
       | - Visit Invite Controller    |   +--------------+---------------+
       +--------------+---------------+                  |
                      |                                  |
                      +-----------------+----------------+
                                        |
                                        v
+-----------------------------------------------------------------------------------+
|                              Transactional Data Store                             |
|  - PostgreSQL: Accounts (Synthetic UUIDs), Birds, Vectors, Moods, Invites, Logs   |
|  - Redis / Memory Store: Active Ephemeral Presence, Fast Event Buffers, Pub/Sub   |
+---------------------------------------+-------------------------------------------+
                                        ^
                                        | Reads Events / Writes Canonical State
                                        v
+-----------------------------------------------------------------------------------+
|                         Simulation Worker Fleet (Tick Engine)                     |
|  - Continuous ~60s Global Simulation Tick                                         |
|  - Low-Pass Filter Personality Drift Processing                                   |
|  - Markov Mood State Machine & Environmental Dynamics                             |
|  - Heuristic Naturalist Notebook Observation Generator                            |
+-----------------------------------------------------------------------------------+
```

### 2.1 Component Responsibilities
1. **Edge / CDN Layer:** Serves static JS/CSS bundles, SVG silhouettes, motif definition JSONs, and an edge-injected initial state bootstrap payload to guarantee `<500ms` Time-to-First-Bird.
2. **API Gateway & Auth Service:** Handles magic-link generation, email dispatch via transactional mail provider, token validation, rate-limiting, and PII boundary enforcement.
3. **Event Ingestion Pipeline:** Accepts client interaction events (presence heartbeats, listen-in toggles, offers, settle actions) and appends them to an immutable, ordered event log.
4. **Simulation Service (Tick Engine):** A horizontally scalable cluster of background workers executing periodic simulation ticks (~60s cadence) across all active and idle aviaries. Writes canonical snapshots and generated notebook entries.
5. **Real-Time Gateway (WSS / SSE):** Pushes updated state snapshots to connected client devices, handles presence disconnections, and terminates revoked visit sessions immediately.
6. **Client Application (Browser SPA):** Renders the 2D scene (Canvas 2D / WebGL), computes procedural micro-motion and interpolations, synthesizes audio via WebAudio graph, manages keyboard focus, and updates accessibility live regions.

### 2.2 Architectural Privacy & PII Boundary
To honor the strict privacy requirements:
* Accounts are assigned a cryptographically random synthetic UUID (`account_id`) at creation.
* The user's email address is stored exclusively in an isolated, encrypted `accounts_auth` table.
* All application tables (`aviaries`, `birds`, `personality_vectors`, `mood_states`, `events`, `notebook_entries`, `telemetry_aggregates`) reference only `account_id`.
* Application logs, event queues, simulation state caches, and error traces are scrubbed of email addresses and IP addresses.
* Simulation databases are physically isolated from analytical and operational telemetry data pipelines.

---

## 3. Data Model & Database Schemas

The database schema is defined in relational PostgreSQL with strict constraints, foreign keys, and non-nullable defaults.

```sql
-- Core Account & Auth Isolation
CREATE TABLE accounts_auth (
    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_hash VARCHAR(64) NOT NULL UNIQUE, -- HMAC-SHA256 for lookup without decryption
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    soft_deleted_at TIMESTAMPTZ NULL,
    hard_delete_due_at TIMESTAMPTZ NULL
);

CREATE TABLE auth_magic_links (
    link_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts_auth(account_id) ON DELETE CASCADE,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    expires_at TIMESTAMPTZ NOT NULL,
    consumed_at TIMESTAMPTZ NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE device_sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts_auth(account_id) ON DELETE CASCADE,
    device_label VARCHAR(100) NOT NULL, -- e.g., "Safari on macOS", "Chrome on Android"
    session_token_hash VARCHAR(64) NOT NULL UNIQUE,
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at TIMESTAMPTZ NULL
);

-- Aviary and Birds
CREATE TABLE aviaries (
    aviary_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts_auth(account_id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    timezone VARCHAR(50) NOT NULL DEFAULT 'UTC',
    settled_until TIMESTAMPTZ NULL,
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    current_weather VARCHAR(20) NOT NULL DEFAULT 'clear' CHECK (current_weather IN ('clear', 'soft_wind', 'passing_rain'))
);

CREATE TABLE birds (
    bird_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL, -- e.g., 'grey_warbler', 'spotted_towhee', 'nightjar'
    custom_name VARCHAR(50) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    perch_zone VARCHAR(10) NOT NULL DEFAULT 'middle' CHECK (perch_zone IN ('front', 'middle', 'back')),
    perch_slot_index INT NOT NULL DEFAULT 0,
    CONSTRAINT uk_aviary_species_slot UNIQUE (aviary_id, perch_zone, perch_slot_index)
);

-- Personality Vectors (Hidden, Monotonic Drift)
CREATE TABLE personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(bird_id) ON DELETE CASCADE,
    boldness DOUBLE PRECISION NOT NULL DEFAULT 0.20 CHECK (boldness >= 0.0 AND boldness <= 1.0),
    social_warmth DOUBLE PRECISION NOT NULL DEFAULT 0.20 CHECK (social_warmth >= 0.0 AND social_warmth <= 1.0),
    vocal_frequency DOUBLE PRECISION NOT NULL DEFAULT 0.20 CHECK (vocal_frequency >= 0.0 AND vocal_frequency <= 1.0),
    plumage_saturation DOUBLE PRECISION NOT NULL DEFAULT 0.20 CHECK (plumage_saturation >= 0.0 AND plumage_saturation <= 1.0),
    curiosity DOUBLE PRECISION NOT NULL DEFAULT 0.20 CHECK (curiosity >= 0.0 AND curiosity <= 1.0),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Dynamic Mood State
CREATE TABLE bird_moods (
    bird_id UUID PRIMARY KEY REFERENCES birds(bird_id) ON DELETE CASCADE,
    current_mood VARCHAR(20) NOT NULL DEFAULT 'content' 
        CHECK (current_mood IN ('wary', 'content', 'curious', 'drowsy', 'alert')),
    mood_started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    mood_expires_at TIMESTAMPTZ NOT NULL,
    last_call_at TIMESTAMPTZ NULL,
    last_interaction_at TIMESTAMPTZ NULL
);

-- Append-Only Interaction Event Stream
CREATE TABLE interaction_events (
    event_id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    session_id UUID NOT NULL REFERENCES device_sessions(session_id) ON DELETE CASCADE,
    event_type VARCHAR(30) NOT NULL CHECK (event_type IN ('presence_ping', 'listen_in_start', 'listen_in_end', 'offer_seed', 'offer_song', 'offer_pool', 'settle_trigger', 'settle_undo')),
    target_bird_id UUID NULL REFERENCES birds(bird_id) ON DELETE SET NULL,
    metadata JSONB NOT NULL DEFAULT '{}'::JSONB,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_in_tick_at TIMESTAMPTZ NULL
);
CREATE INDEX idx_events_unprocessed ON interaction_events (aviary_id, created_at) WHERE processed_in_tick_at IS NULL;

-- Field Notebook Observations
CREATE TABLE notebook_entries (
    entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    observed_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    day_of_week VARCHAR(15) NOT NULL, -- e.g., 'tuesday'
    prose_content TEXT NOT NULL,
    bird_ids_involved UUID[] NOT NULL DEFAULT '{}',
    observation_type VARCHAR(30) NOT NULL -- e.g., 'greeting_order', 'plumage_shift', 'weather_interaction', 'quiet_stretch'
);
CREATE INDEX idx_notebook_aviary ON notebook_entries (aviary_id, observed_at DESC);

-- Social Visits & Audit Log
CREATE TABLE visit_invitations (
    invite_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    invite_token_hash VARCHAR(64) NOT NULL UNIQUE,
    recipient_email_encrypted BYTEA NOT NULL,
    recipient_email_hash VARCHAR(64) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL, -- 30 days from creation
    revoked_at TIMESTAMPTZ NULL
);

CREATE TABLE visit_logs (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invite_id UUID NOT NULL REFERENCES visit_invitations(invite_id) ON DELETE CASCADE,
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    visitor_masked_identifier VARCHAR(100) NOT NULL, -- e.g. "f***@domain.com"
    entered_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    exited_at TIMESTAMPTZ NULL,
    duration_seconds INT NULL
);
```

---

## 4. API Surface & Communication Protocols

All client-server interactions use standard HTTPS and WSS protocols. Naturalist surfaces consume JSON payloads formatted for rendering; system errors are emitted with descriptive, matter-of-fact strings.

```
API Error Response Format (Matter-of-Fact):
{
  "error": {
    "code": "AUTH_LINK_EXPIRED",
    "message": "We couldn't sign you in. The link may have expired. Try requesting a new link."
  }
}
```

### 4.1 Authentication & Account Management
* `POST /api/v1/auth/magic-link`: Request a login magic link. Rate limited to 3 per 15 minutes per email.
  * Request: `{"email": "user@example.com"}`
  * Response (200): `{"status": "sent"}`
* `POST /api/v1/auth/verify`: Verify token from link click.
  * Request: `{"token": "hex_string"}`
  * Response (200): `{"session_token": "...", "account_id": "...", "timezone": "America/New_York"}`
* `POST /api/v1/account/sessions/revoke`: Revoke an active session.
  * Request: `{"session_id": "uuid"}`
  * Response (200): `{"status": "revoked"}`
* `GET /api/v1/account/export`: Request an archive of aviary state. Returns signed download link via email.
* `POST /api/v1/account/delete`: Initiate 30-day soft deletion.
* `POST /api/v1/account/restore`: Restore soft-deleted account within 30 days.

### 4.2 Aviary State & Event Submission
* `GET /api/v1/aviary/bootstrap`: Returns complete initial state for cold start. Injected directly into HTML template on initial load to satisfy the `<500ms` First-Bird requirement.
* `POST /api/v1/aviary/events`: Ingests user interaction events into the append-only log.
  * Request:
    ```json
    {
      "events": [
        {
          "event_type": "presence_ping",
          "timestamp": "2026-08-13T17:20:00Z",
          "metadata": { "active_duration_seconds": 60 }
        },
        {
          "event_type": "offer_seed",
          "target_bird_id": "8fa3c0...-...",
          "timestamp": "2026-08-13T17:20:15Z"
        }
      ]
    }
    ```
  * Response (200): `{"accepted_count": 2}`

### 4.3 Real-Time WebSocket Channel (`/ws/v1/aviary`)
* **Client Handshake:** Authenticates via HTTP Authorization header (`Bearer <session_token>`).
* **Server-to-Client `SNAPSHOT_UPDATE` (Every ~60s or on significant interaction):**
  ```json
  {
    "type": "SNAPSHOT_UPDATE",
    "server_time": "2026-08-13T17:21:00Z",
    "local_time_bucket": "morning",
    "lighting_factor": 0.85,
    "weather": "clear",
    "is_settled": false,
    "birds": [
      {
        "bird_id": "8fa3c0d1-1234-4567-89ab-cdef01234567",
        "species_id": "grey_warbler",
        "custom_name": "pip",
        "perch_zone": "front",
        "perch_slot_index": 1,
        "mood": "content",
        "plumage_saturation": 0.42,
        "vocal_frequency": 0.38,
        "activity_state": "preening",
        "last_greeting_order": 1
      },
      {
        "bird_id": "9b12d3e4-5678-4321-98ba-fedcba987654",
        "species_id": "spotted_towhee",
        "custom_name": "wren",
        "perch_zone": "back",
        "perch_slot_index": 0,
        "mood": "drowsy",
        "plumage_saturation": 0.35,
        "vocal_frequency": 0.22,
        "activity_state": "fluffed_rest",
        "last_greeting_order": 2
      }
    ],
    "narration_prose": "pip is on the front rail, preening calmly. wren rests low on the back branch.",
    "new_notebook_entry": null
  }
  ```

### 4.4 Visit Protocols & Revocation
* `POST /api/v1/visits/invite`: Host creates visit invitation.
  * Request: `{"recipient_email": "friend@example.com"}`
  * Response (200): `{"invite_id": "uuid", "expires_at": "..."}`
* `DELETE /api/v1/visits/invite/{invite_id}`: Host immediately revokes invite.
  * Response (200): `{"status": "revoked"}`
* `GET /api/v1/visits/stream?token={invite_token}`: Visitor connects to read-only SSE stream.
  * Visitor snapshot contains only read-only scene parameters (positions, moods, weather, audio motifs).
  * If the invite is revoked or expired:
    ```json
    {
      "type": "VISIT_TERMINATED",
      "reason": "This visit invitation is no longer active."
    }
    ```
  * Client immediately transitions to a clean, matter-of-fact termination card.

---

## 5. Simulation Engine Design & Drift Calibration

The simulation service is the beating heart of Pocket Aviary. It executes a tick loop every 60 seconds per aviary, guaranteeing continuity regardless of client connectivity.

```
+-----------------------------------------------------------------------------------+
|                            Simulation Tick Loop (60s)                             |
+-----------------------------------------------------------------------------------+
| 1. Ingest unprocessed interaction events since last tick                          |
| 2. Compute effective Presence Time (T_pres) using 3-factor validation             |
| 3. Apply Additive Monotonic Personality Drift: P(t+1) = P(t) + delta(T_pres, Ev)   |
| 4. Update Fast-Timescale Mood Markov Transitions (Time-of-day, Weather, Social)   |
| 5. Process Bird Perch Migration & Activity State Dispatch                         |
| 6. Check Observation Triggers -> Synthesize Naturalist Notebook Entry if sparse   |
| 7. Persist Canonical Snapshot to PostgreSQL & Broadcast to Connected Clients      |
+-----------------------------------------------------------------------------------+
```

### 5.1 Presence Accounting Engine
Presence time ($T_{\text{presence}}$) is accumulated if and only if all three conditions are met concurrently during the sample interval:
1. `document.visibilityState === 'visible'`
2. `document.hasFocus() === true`
3. Pointer movement, touch, or keypress registered within the last 300 seconds ($T_{\text{inactivity}} \le 5\text{ min}$).

If any condition fails, the presence accumulation delta for that window is strictly $0.0$.

### 5.2 Monotonic Personality Drift Formulation
Personality vectors consist of 5 normalized floats $\mathbf{P} = [p_{\text{bold}}, p_{\text{warmth}}, p_{\text{vocal}}, p_{\text{plumage}}, p_{\text{curious}}] \in [0.0, 1.0]^5$.

Drift is computed as a low-pass filter over presence and interactions:
$$\Delta p_i = \alpha_i \cdot f_i(\text{Events}, T_{\text{presence}})$$
$$p_i(t+1) = \min(1.0, p_i(t) + \Delta p_i)$$

**Non-Negativity Constraint:**
$$\Delta p_i \ge 0 \quad \forall i$$
Under zero presence or long absences, $\Delta p_i = 0$. Traits **never** decrease. Ignored birds become ambient and quiet; they do not mistrust or regress in trait scores.

#### Calibration Mathematics
* Baseline calibration assumes a regular visitor spends $\approx 20 \text{ minutes/day}$ ($140 \text{ min/week} = 8,400 \text{ seconds/week}$).
* **Target 1 (Instrument Measurability at 1 Week):** Trait increase $\Delta P \approx +0.015$ (sufficient for analytical telemetry and automated test assertions).
  $$\alpha_{\text{presence}} = \frac{0.015}{8400 \text{ s}} \approx 1.785 \times 10^{-6} \text{ s}^{-1}$$
* **Target 2 (Felt Human Perception at 3 Weeks):** Cumulative trait increase after 3 weeks $\Delta P \approx +0.05$ to $+0.08$. This shifts visible behavioral probabilities (perch selection, call frequency, first-greeter probability) into noticeable territory.
* **Interaction Deltas:**
  * `listen_in` (per 60s focused): $\Delta p_{\text{warmth}} = +0.0005$, $\Delta p_{\text{vocal}} = +0.0004$.
  * `offer_seed` (accepted): $\Delta p_{\text{bold}} = +0.0008$, $\Delta p_{\text{curious}} = +0.0012$.
  * `offer_song`: $\Delta p_{\text{vocal}} = +0.0010$, $\Delta p_{\text{warmth}} = +0.0006$.
  * `offer_pool`: $\Delta p_{\text{bold}} = +0.0007$, $\Delta p_{\text{curious}} = +0.0008$.

### 5.3 Fast-Timescale Mood State Machine
Mood is represented by a discrete Markov chain over states $S \in \{\text{wary}, \text{content}, \text{curious}, \text{drowsy}, \text{alert}\}$.

```
                       +------------+
        +------------> |   alert    | <-----------+
        |              +-----+------+             |
        |                    |                    |
        | weather: wind      | time: morning      | offer presented
        |                    v                    |
  +-----+------+       +------------+       +-----+------+
  |    wary    | <---> |  content   | <---> |  curious   |
  +------------+       +-----+------+       +------------+
        ^                    |                    ^
        | alarm call         | time: dusk/night   |
        +--------------------+--------------------+
                             |
                             v
                       +------------+
                       |   drowsy   |
                       +------------+
```

* **Transition Modulators:**
  * **Diurnal Cycle:** Local dawn/morning increases $P(\text{alert})$; dusk and night heavily weight $P(\text{drowsy})$. Settled aviary forces transition to `drowsy` within 2 ticks.
  * **Weather Influence:** `passing_rain` dampens call frequency and nudges birds to `content` or `drowsy` on middle/back perches. `soft_wind` increases $P(\text{alert})$ by $+0.25$.
  * **Social Contagion:** If bird $A$ enters `wary` (low boldness + sudden event), adjacent birds in the same perch zone experience a $0.30$ probability jump to `wary`.
  * **Personality Bias:** High boldness ($p_{\text{bold}} > 0.6$) reduces $P(\text{wary})$ by $60\%$ across all inputs and boosts $P(\text{curious})$.

### 5.4 Naturalist Field Notebook Generation
Notebook generation runs an observation evaluation pass during the simulation tick. To preserve sparsity, the generator produces roughly one entry every 3 to 5 calendar days for active aviaries.

* **Heuristic Triggers:**
  * *Greeter Shift:* Bird $B$ greets before Bird $A$ for the first time in $\ge 7$ days.
  * *Perch Breakthrough:* A historically wary bird perches in the `front` zone for $>15$ continuous minutes.
  * *Weather Harmony:* A passing rain coincides with a multi-bird vocal response.
  * *Extended Stillness:* 20+ minutes of quiet preening during a morning session without user offers.
* **Prose Synthesizer:** Assembles template-free, naturalist prose following the strict grammar rules (lowercase, present-tense, bird-named, no gamification phrasing):
  * Example: `"thursday — wren came to the low branch while rain crossed the glass. pip called twice from the back and preened."`

---

## 6. Multi-Device Sync & Conflict Model

Multi-device consistency is maintained via a single-writer, additive event architecture.

```
  Device A (Laptop)                          Server (Simulation Tick)               Device B (Phone)
         |                                              |                                   |
         | --- 1. POST Event (Listen-in Pip 2m) ------> |                                   |
         |                                              | (Appends to Interaction Log)      |
         |                                              |                                   |
         |                                              | --- 2. Advance Tick (60s)         |
         |                                              |    - Computes Delta (+warmth)     |
         |                                              |    - Updates Canonical Snapshot   |
         |                                              |                                   |
         | <--- 3. WS Broadcast (Snapshot v42) -------- | --- 3. WS Broadcast (Snapshot) -> |
         |                                              |                                   |
```

### 6.1 Elimination of Last-Write-Wins (LWW)
* **Clients never author or submit state values.** A client cannot send `{"boldness": 0.45}` or `{"mood": "content"}`.
* Clients submit exclusively append-only semantic interaction events (`offer`, `listen_in_start`, `presence_ping`).
* The simulation engine is the **sole writer** of bird state. Events from multiple devices are queued into PostgreSQL and processed strictly in transactional time order during the next tick.
* Simultaneous sessions on phone and laptop simply contribute additive presence and interaction records to the shared queue; the resulting snapshot is pushed to both devices over WebSockets.

### 6.2 Client Reconciliation & Interpolation
When a client receives a new snapshot ($S_{k+1}$):
1. **Positional Interpolation:** If a bird's perch zone changes, the client schedules a natural flight arc or hop trajectory over $1.8 \text{s}$ using cubic easing ($C(t) = 3t^2 - 2t^3$).
2. **Mood Transition Blending:** The client smoothly transitions the idle micro-motion parameters (scan frequency, ruffle duration) over $3.0 \text{s}$ rather than snapping animation frames.
3. **Absence Recovery:** If a client re-opens after hours of sleep, it pulls the latest snapshot via HTTP `bootstrap` and initializes the scene immediately with birds mid-action at their canonical server positions.

---

## 7. Frontend Rendering & Animation Pipeline

The visual presentation runs on an HTML5 Canvas / WebGL 2D renderer engineered to feel like an open window on a living habitat.

```
+-----------------------------------------------------------------------------------+
|                            Aviary Scene Graph (2D)                                |
+-----------------------------------------------------------------------------------+
| Layer 0: Sky & Ambient Lighting Gradient (Local solar time calculation)           |
| Layer 1: Distant Background Foliage (Parallax Factor: 0.02)                      |
| Layer 2: Back Perch Zone & Seated Birds (Scale: 0.78, Muted Saturation)           |
| Layer 3: Midground Perches & Ambient Drifting Leaves (Parallax Factor: 0.05)      |
| Layer 4: Middle Perch Zone & Seated Birds (Scale: 0.90)                           |
| Layer 5: Front Rail Perch Zone & Seated Birds (Scale: 1.05, Crisp Detail)         |
| Layer 6: Foreground Branches / Droplets (Parallax Factor: 0.10)                   |
| Layer 7: Call Captions (Floating Naturalist Text Overlays)                        |
+-----------------------------------------------------------------------------------+
```

### 7.1 Scene Layout & Viewport Adaptability
* **Fixed Visual Aspect Bounds:** The scene is rendered to a virtual canvas of $1920 \times 1080$ with a responsive CSS letterbox/pillarbox container maintaining visible bounds between $16:9$ and $21:9$.
* **Zero Bird Cropping:** Perch slots are anchored as relative fractions of the viewport width ($X \in [0.15, 0.85]$). On mobile portrait views, the horizontal space compresses naturally, shifting perches closer together while guaranteeing all 7 birds remain 100% visible on screen without scrolling or panning.
* **Three Depth Perch Zones:**
  * *Front Rail ($Z=1.0$):* Birds render at $105\%$ scale with high feather detail and full saturation.
  * *Middle Branch ($Z=0.6$):* Birds render at $90\%$ scale.
  * *Back Perch ($Z=0.2$):* Birds render at $78\%$ scale with atmospheric haze ($15\%$ sky color tint blend).

### 7.2 Procedural Idle Micro-Motion
Birds are animated parametrically using a 2D skeletal node hierarchy (Root -> Body -> Chest -> Neck -> Head/Beak -> Tail -> Wings):
* **Breathing Rhythm:** Continuous sinusoidal oscillation of chest scale ($f \approx 0.4 \text{ Hz}$, modulated by mood).
* **Head Tilts & Saccades:** Poisson-distributed angular head rotations ($[-25^\circ, +25^\circ]$) with high-velocity snap ($40 \text{ ms}$) and gentle settle.
* **Preening Sequences:** Multi-keyframe procedural kinematics where the beak reaches toward wing primaries, paired with localized feather ruffling.
* **Body Shuffle / Weight Shift:** Occasional foot repositioning and tail twitch ($120 \text{ ms}$ impulse).
* **Mid-Action Startup:** On initial frame render, time accumulator $t_{\text{seed}}$ is initialized from `server_time`, ensuring birds appear immediately mid-breath or mid-preen without an entry animation.

### 7.3 Lighting & Weather Rendering
* **Diurnal Shading:** Smooth celestial color grading mapped to user's local solar angle.
  * *Dawn (05:00–07:30):* Soft rose, amber, and pale lavender ambient gradients.
  * *Midday (11:00–15:00):* Neutral, warm daylight with soft vertical highlights.
  * *Dusk / Sunset (18:00–20:30):* Deep ochre, warm terracotta, and quiet indigo shadows.
  * *Night (21:00–04:30):* Subdued deep navy with soft ambient starlight illumination.
* **Weather FX:**
  * *Passing Rain:* Slanted translucent rain streaks with soft canvas ripples on bottom rails.
  * *Soft Wind:* Increased velocity and flutter frequency for drifting leaf particles.

### 7.4 Reduced-Motion Mode Architecture
When `prefers-reduced-motion: reduce` is active or enabled via accessibility settings:
* Continuous bone interpolation and micro-motion are disabled.
* Birds are rendered in static naturalist poses (Perched, Scanning, Preening, Fluffed).
* State and pose changes transition via a **slow opacity cross-fade** ($1.5 \text{s}$ cross-fade between static pose canvases).
* Flight between perches is replaced with a gentle fade-out at Perch A and fade-in at Perch B.
* Ambient drifting leaf particles are completely suppressed.

---

## 8. Procedural Audio Synthesis & Chorus Engine

The audio system synthesizes all calls client-side using the WebAudio API. No recorded audio files or loops are used.

```
+-----------------------------------------------------------------------------------+
|                         Bird Procedural Audio Graph (WebAudio)                    |
+-----------------------------------------------------------------------------------+
| [Motif Pitch / Env Generator]                                                     |
|       |                                                                           |
|       v                                                                           |
| [FM Carrier Oscillator] ---> [WaveShaper Distortion] ---> [Bandpass Biquad Filter]|
|       ^                                                                |          |
|       | (FM Modulation)                                                v          |
| [Modulator Oscillator]                                          [Dynamic Gain]    |
|                                                                        |          |
| [Air Noise Generator] -----> [Highpass Filter] ----> [Noise Gain] ---->+          |
|                                                                        |          |
|                                                                        v          |
|                                                               [Stereo Panner Node]|
|                                                                        |          |
|                                                                        v          |
| [Listen-In Mix Bus (Active: 0dB / Inactive: -12dB)] <------------------+          |
|       |                                                                           |
|       v                                                                           |
| [Master Limiter / DynamicsCompressor] ---> [AudioDestination (Speakers)]          |
+-----------------------------------------------------------------------------------+
```

### 8.1 Call Grammar & Species Motif Library
Each of the 6 species possesses a procedural call grammar defined by base frequencies, modulation indices, formant filters, and motif transition graphs:
1. **Grey Warbler (*Sylvia grisea*):** High fundamental ($2.8\text{kHz} - 4.2\text{kHz}$), rapid ascending 3-note whistle with gentle vibrato ($6\text{Hz}$).
2. **Spotted Towhee (*Pipilo maculatus*):** Introductory short blip followed by a buzzy, modulated frequency trill ($1.8\text{kHz} - 2.4\text{kHz}$).
3. **Wood Nuthatch (*Sitta europaea*):** Clear, piping repeated monosyllabic notes ($1.5\text{kHz}$) with rapid decay.
4. **Dusk Finch (*Carpodacus umbra*):** Mellow, flute-like descending melodic pairs ($1.2\text{kHz} - 1.9\text{kHz}$).
5. **Pied Wagtail (*Motacilla alba*):** Sharp, cheerful two-syllable "chick-it" with quick frequency modulation.
6. **Nightjar (*Caprimulgus noctis*):** Low rhythmic churring ($700\text{Hz} - 950\text{Hz}$) active predominantly during evening and night cycles.

### 8.2 Real-Time Synthesis Parameters
Calls are synthesized on demand by scheduling nodes on the `AudioContext`:
* **Pitch & Micro-Timing Jitter:** Every call invocation applies randomized Gaussian jitter to fundamental frequency ($\sigma = \pm 1.8\%$) and note duration ($\sigma = \pm 3.5\%$) to eliminate mechanical repetition.
* **Spatial Panning:** Each bird's `StereoPannerNode` maps directly to its horizontal perch position ($X \in [-0.85, +0.85]$).

### 8.3 Listen-In Mix Ramping
* When a user initiates **Listen-In** on Bird $K$:
  * Bird $K$'s mix gain node smoothly ramps from $1.0$ ($0\text{dB}$) to $1.5$ ($+3.5\text{dB}$) over $1200\text{ms}$ using `exponentialRampToValueAtTime`.
  * All other birds' mix gain nodes ramp down from $1.0$ to $0.25$ ($-12\text{dB}$) over $1200\text{ms}$. Non-focused birds remain audible as soft ambient accompaniment and are **never completely muted**.
* Disengaging Listen-In smoothly restores all gain nodes to $1.0$ ($0\text{dB}$) over $1500\text{ms}$.

### 8.4 WebAudio Fallback Architecture
If the browser lacks WebAudio support or audio permissions are denied:
* The audio engine operates in **graceful silence**.
* Call events continue to be scheduled internally.
* Visual call captions are **automatically enabled** on the aviary canvas.
* No low-quality recorded audio files or canned audio fallbacks are loaded.

---

## 9. Accessibility Surfaces & UX Details

Accessibility in Pocket Aviary is an intentionally designed surface providing full parity of charm.

```
+-----------------------------------------------------------------------------------+
|                        Accessibility Notification Layer                           |
+-----------------------------------------------------------------------------------+
| [Simulation Snapshot] ---> [Naturalist Prose Generator]                          |
|                                    |                                              |
|                                    v                                              |
|            <div id="aviary-narration" aria-live="polite" class="sr-only">         |
|              "a grey warbler perches on the front rail, preening softly.          |
|               wren is resting low on the back branch. the morning light is warm." |
|            </div>                                                                 |
+-----------------------------------------------------------------------------------+
```

### 9.1 Screen-Reader Narration (`aria-live="polite"`)
* An invisible, dedicated live region (`#aviary-narration`) maintains running naturalist descriptions of the aviary scene.
* **Pacing & Queue Management:**
  * Background cadence updates every **30 to 60 seconds** to avoid flooding screen-reader speech queues.
  * Priority events (user-initiated offer accepted, return-greeting, settle gesture) trigger an immediate, courteous prose update.
* **Prose Format:** Strictly adheres to lowercase, present-tense naturalist tone:
  * `"pip cocks her head toward the front perch and calls softly. wren sits quietly in the cool morning shade."`

### 9.2 Real-Time Call Captions
* Subtitle bubbles appear floating softly above the vocalizing bird ($Y - 40\text{px}$).
* Captions are generated procedurally to match the synthesized motif:
  * `"a soft three-note rise"`
  * `"a low trill, paused, low trill again"`
  * `"a single sharp call from the high branch"`
* Captions fade in over $200\text{ms}$, hold for the duration of the vocalization, and fade out over $600\text{ms}$.

### 9.3 Keyboard Navigation & Focus Hierarchy
* `Tab`: Cycles through top-bar actions (Settings, Accessibility, Field Notebook, Offer Affordance, Settle).
* `Tab` into Aviary: Focuses the first bird on the front rail.
* `ArrowLeft` / `ArrowRight` / `ArrowUp` / `ArrowDown`: Navigates spatial focus between birds across the 3 perch zones.
* `Enter` / `Space`: Engages **Listen-In** on the focused bird.
* `Escape`: Disengages Listen-In or closes open top-bar panels.
* **Focus Indicator:** A high-contrast, dual-ring outline (inner: `#FFFFFF`, outer: `#1A2B3C`) with 4.5:1 minimum contrast across all lighting states.

### 9.4 Contrast & Legibility Compliance
* All UI chrome, settings dialogs, field notebook text, and caption labels strictly exceed **WCAG AA** standards ($>4.5:1$ for standard text, $>3:1$ for large headings and icons).

---

## 10. Performance Budgets & Observability

### 10.1 Hard Performance Budgets

| Metric | Target Budget | Enforcement Mechanism |
| :--- | :--- | :--- |
| **Initial JS Bundle (gzipped)** | `< 2.0 MB` | Webpack / Rollup bundle analyzer in CI; PR fails if bundle exceeds 1.8MB. |
| **Time-to-First-Bird Visible** | `< 500 ms` | Initial snapshot bootstrap inlined in initial HTML payload; critical SVG silhouettes pre-parsed. |
| **Idle Render Frame Rate** | `60 fps` steady | Tested on 5-year-old reference laptop (Intel Core i5-8250U / 8GB RAM). |
| **Memory Leak Ceiling** | `0.0 MB / 30 min` | Automated Playwright leak detector checking JS heap size every 5 minutes. |
| **Simulation Tick p99 Latency** | `< 5.0 s` | CloudWatch / Datadog alert firing on tick duration degradation. |

### 10.2 Strict Observability & Telemetry Privacy
* **Permitted Operational Telemetry:**
  * Edge HTTP request counts, status code distributions, and p50/p90/p99 latency.
  * Server simulation tick run durations and worker queue depth.
  * Client render frame drop rates (aggregate histograms).
  * WebAudio initialization failure rates.
* **Strictly Prohibited Telemetry:**
  * No per-account interaction event logs in telemetry stores.
  * No bird personality trait distributions or mood state tracking in aggregate analytics.
  * No user email addresses, IP addresses, or location tracking in logging sinks.

---

## 11. Rollout Plan & Aviary Lifecycle

```
Day 0: Account Creation (2 Starters)
   |
   +---> Month 2 (~60 days): 3rd Bird Species Offer
            |
            +---> Month 4 (~120 days): 4th Bird Species Offer
                     |
                     +---> Month 7 (~210 days): 5th Bird Species Offer
                              |
                              +---> Month 10 (~300 days): 6th Bird Species Offer
                                       |
                                       +---> Month 13 (~390 days): 7th Bird (Cap Reached)
```

### 11.1 Starter Adoption Experience
1. User signs in via magic link for the first time.
2. System allocates an aviary and randomly selects 2 distinct species from the 6-species pool.
3. User is presented with a quiet naming prompt (default suggestions provided, fully editable).
4. The aviary loads its initial quiet field; the two starter birds arrive with soft initial flights to their perches.

### 11.2 Age-Based Population Expansion (Max 7 Birds)
* Bird additions are tied **exclusively to aviary calendar age**, not visit streaks, click counts, or interaction points.
* When an aviary reaches an age milestone (e.g., 60 days for Bird 3), a gentle species arrival motif is queued for the next morning session.
* The aviary strictly caps at 7 birds to guarantee procedural audio recognizability.

---

## 12. Risk Analysis & Mitigation Matrix

| Risk Scenario | Root Cause | Impact | Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **Drift Saturation (Tamagotchi Trap)** | Presence filter weights too high; fast trait ramp. | Birds reach max saturation in days; illusion breaks. | Strict mathematical calibration ($\alpha = 1.785 \times 10^{-6}\text{ s}^{-1}$); automated unit tests verifying 3-week perception curve. |
| **Multi-Device State Desynchronization** | Client attempts local personality mutations or LWW overwrites. | Drift overwritten; session history lost. | Server is the sole state writer; clients submit only append-only event records. |
| **Audio Fatigue & Uncanniness** | Mechanical repetition in WebAudio oscillators. | User mutes tab; core aliveness lost. | Procedural call grammar with Gaussian pitch/timing jitter and FM synthesis modeling real syrinx dynamics. |
| **Ghost Presence Inflation** | Background tab or forgotten window accumulating presence. | Birds drift without human presence. | Strict 3-factor presence conjunction (`visibilityState` + `hasFocus` + user input within 5 min). |
| **Voice Tone Contamination** | Developer introduces gamified toasts ("Level Up!", "Streak 5!"). | Breaks core product aesthetic. | Automated lint rules and CI text scanners blocking forbidden phrases and toast components. |
| **Accessibility Regression** | Reduced-motion mode treated as static fallback. | Excludes vestibular/motor-impaired users from experiencing aliveness. | Reduced-motion designed as first-class pose cross-fading surface with full call captioning. |

---

## 13. File & Deliverable Verification Summary

1. **`runs/wave_001/plans/001/PLAN.md`**: Fully detailed, comprehensive implementation blueprint covering all PRD specifications without product code implementation.
2. **`runs/wave_001/plans/001/CANDIDATE_METADATA.json`**: Accurate runtime candidate metadata adhering strictly to benchmark orchestrator requirements.
