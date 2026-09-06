# Pocket Aviary — System Architecture and Implementation Plan (v1)

## 1. Executive Summary & Design Tenets

Pocket Aviary is a lightweight, web-only ambient virtual aviary where users adopt two starter birds (scaling up to seven over months) that live in a single horizontal scene. The system is designed as an observational relationship rather than a gamified app or a custodial pet simulator.

This plan operationalizes the product requirements across engineering disciplines:
1. **Feels Alive, Not Robotic:** The aviary must never present a "cold start" or frozen state. First frame paints with motion and ambient sounds already in progress. Procedural call generation prevents repetitive audio loops.
2. **Notice, Never Announce:** No arrival toasts, level-up banners, badges, or streak celebrations. All welcoming is expressed purely through natural bird behavior (e.g. glances, head-tilts, staggered greetings).
3. **Charm from Specificity:** Sparse, naturalist field-notebook entries generated from granular behavioral observations rather than templated telemetry counters.
4. **Restraint Over Richness:** A fixed single horizontal viewport with three perch depths (front, middle, back), 2 to 7 birds maximum, and subtle ambient weather.
5. **Dual-Register Voice Discipline:**
   - *Naturalist voice:* Lowercase, present-tense, observational tone for product surfaces (aviary, field notebook, audio captions, screen-reader narration).
   - *Matter-of-fact voice:* Capitalized, direct, plain English for system surfaces (magic-link sign-in, account settings, sync conflicts, errors, accessibility controls).

---

## 2. Scope & Non-Goals

### 2.1 In-Scope for v1
- **Platform:** Modern desktop and mobile web browsers (last two major versions of Chrome, Safari, Firefox, Edge).
- **Bird Population:** Fixed cap of 7 birds per aviary. New aviaries start with exactly 2 system-assigned starter birds selected from a pool of 6 species. Unlocking the 3rd through 7th birds occurs exclusively via aviary age milestones.
- **Bird Customization & Identity:** User-defined naming with renaming support at any time. Stable UUID identities that persist through renames, migrations, and syncs.
- **Interactions:**
  - *Passive presence:* Monitored via strict tripartite validation (tab visibility, window focus, recent input activity).
  - *Return-greeting:* Procedural reaction within 1–2 seconds of session start, staggered across birds, modulated by absence duration and boldness.
  - *Listen-in:* Focusing a bird to elevate its mix level smoothly while attenuating others to ambient levels without hard cuts.
  - *Offer:* Dropping seed, playing a song fragment, or introducing a still pool via a top-bar control with per-bird cooldowns (a few minutes).
  - *Settle:* Soft session-end gesture shifting lighting to evening and quieting calls, with a 5-second undo affordance.
  - *Field Notebook:* Sparse, read-only naturalist observations generated every 2–4 days.
- **Social (Optional & Quiet):** One-to-one, read-only visit invitations issued via email magic links. Revocable at any time; expires in 30 days. No co-presence, no chat, no visitor avatars, no host presence drift from visits.
- **Sync & Accounts:** Single-user accounts authenticated via 15-minute email magic links. Per-device revocable sessions. Canonical server-side simulation tick (~60s cadence) writing to an append-only event architecture. Multi-device consistency through snapshot pulling and event submission.
- **Privacy Controls:** Synthetic account UUIDs decoupled from encrypted email addresses. Self-service JSON state export. 30-day soft deletion with full hard deletion thereafter. Zero cross-account aggregation of bird interaction telemetry.
- **Accessibility:** Continuous naturalist screen-reader live narration (`aria-live="polite"`), call captioning, WCAG AA contrast compliance, keyboard navigation with high-contrast outlines, and a distinct reduced-motion mode (poetic cross-fades rather than animated skeletal motion).
- **Audio:** Real-time client-side procedural WebAudio synthesis with dynamic chorusing and a silent fallback with automatic captions when WebAudio is unavailable.

### 2.2 Explicit Non-Goals (v1 & Future Guardrails)
- **No Native Apps:** No iOS or Android native wrappers/apps. Web standards only.
- **No Gamification:** No achievements, badges, scores, levels, streaks, visit calendars, green dots, or visit-frequency metrics.
- **No Tamagotchi / Custodial Mechanics:** No bird death, hunger, illness, visible distress, or penalty for neglect. Personality drift is monotonic toward expressive; birds become quiet and ambient during absence, never mistrustful.
- **No Social Network Surfaces:** No public directory, discovery feed, user profiles, following, mutual visits, public comments, or leaderboards.
- **No Monitization or Subscriptions:** No microtransactions, paid bird tiers, or cosmetics.
- **No Push Notifications:** No browser push alerts, reminders, or unprompted emails re-engaging absent users.

---

## 3. System Architecture & Service Topology

The system separates high-frequency rendering and audio synthesis on the client from authoritative simulation and data ownership on the server.

```
+-------------------------------------------------------------------------------+
|                                CLIENT BROWSER                                 |
|                                                                               |
|  +------------------------+  +----------------------+  +-------------------+  |
|  |   Rendering Pipeline   |  | Procedural Audio Engine | Accessibility     |  |
|  | (HTML5 Canvas 2D/WebGL)|  | (WebAudio Graph API) |  | (ARIA Narration,  |  |
|  | - Layered Scene        |  | - Procedural Motifs  |  |  Call Captions,   |  |
|  | - Micro-motion & Poses |  | - Chorus Mixer       |  |  Keyboard Nav)    |  |
|  | - Reduced-motion Fades |  | - Listen-in Dynamics |  +-------------------+  |
|  +-----------^------------+  +----------^-----------+                         |
|              |                          |                                     |
|  +-----------+--------------------------+-----------+  +-------------------+  |
|  |         Client State & Sync Coordinator          |  | Presence Monitor  |  |
|  | - Interpolation Buffer  - Snapshot Cache         |  | (Vis+Focus+Input) |  |
|  | - Event Dispatcher Queue                         |  +---------+---------+  |
|  +-------------------------^------------------------+            |            |
+----------------------------|-------------------------------------|------------+
                             | HTTPS / WSS                         |
                             v                                     v
+-------------------------------------------------------------------------------+
|                                EDGE / CDN LAYER                               |
| - Edge Caching of Static Shell Assets (<2MB JS/CSS/SVGs)                      |
| - Edge Worker injects Initial Snapshot Bootstrap into index.html              |
+-------------------------------------------------------------------------------+
                             |
                             v
+-------------------------------------------------------------------------------+
|                           BACKEND SERVICE LAYER                               |
|                                                                               |
|  +-----------------------------------+  +----------------------------------+  |
|  |         API Ingestion Service     |  |     Simulation Tick Worker       |  |
|  | - Magic-Link Auth & JWT Sessions  |  | - Sharded Cron Tick (~60s loop)  |  |
|  | - Snapshot Serving (GET)          |  | - Drift Low-Pass Filtering       |  |
|  | - Append-only Event Ingestion(POST|  | - Mood Transitions & Timers      |  |
|  | - Visit Token Authorization       |  | - Sparse Notebook Entry Engine   |  |
|  +-----------------+-----------------+  +-----------------+----------------+  |
+--------------------|--------------------------------------|-------------------+
                     |                                      |
                     v                                      v
+-------------------------------------------------------------------------------+
|                            PERSISTENCE & CACHE                                |
|                                                                               |
|  +-------------------------------------+  +---------------------------------+ |
|  |       PostgreSQL Primary DB         |  |          Redis Store            | |
|  | - Accounts & Sessions               |  | - Distributed Tick Lock (Redlock|
|  | - Birds & Canonical Vectors         |  | - Latest Aviary Snapshot Cache  | |
|  | - Append-Only Interaction Events    |  | - In-flight Presence Pings      | |
|  | - Field Notebook & Visit Records    |  | - Ephemeral Rate Limits         | |
|  +-------------------------------------+  +---------------------------------+ |
+-------------------------------------------------------------------------------+
```

### 3.1 Component Boundaries
- **Edge Layer:** Serves static assets, pre-populates initial state snapshot in bootstrap HTML payload to satisfy the <500ms time-to-first-bird requirement.
- **API Service:** Stateless REST/HTTP service validating session tokens, handling magic link workflows, receiving batched interaction events, and issuing state snapshots.
- **Simulation Worker Service:** Background worker executing a tick loop every 60 seconds per active aviary. Consumes raw interaction events, updates personality vectors additively, transitions moods, advances daylight cycles, and persists updated canonical snapshots.
- **Client Application:** Lightweight SPA (TypeScript + 2D Canvas). Never calculates personality drift or mood changes authoritatively; acts as an interpolator and reactive consumer of snapshots while synthesizing audio locally via WebAudio.

---

## 4. Data Model & Storage Schema

All relational entities use synthetic UUID primary keys. Email addresses are encrypted at rest using AES-256-GCM and never exposed as foreign keys, partition keys, or logging identifiers.

### 4.1 Relational Schema

```sql
-- Core Account Record
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_hash VARCHAR(64) UNIQUE NOT NULL, -- Blind index HMAC-SHA256 for lookup
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    deletion_status VARCHAR(16) NOT NULL DEFAULT 'active', -- 'active', 'soft_deleted'
    deletion_requested_at TIMESTAMPTZ,
    visit_notification_enabled BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

-- Authentication Magic Links
CREATE TABLE magic_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    token_hash VARCHAR(64) UNIQUE NOT NULL,
    expires_at TIMESTAMPTZ NOT NULL,
    consumed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

-- Device Sessions
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    session_token_hash VARCHAR(64) UNIQUE NOT NULL,
    device_label VARCHAR(128) NOT NULL,
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    revoked_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

-- Aviary Record (1:1 with Account at v1)
CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID UNIQUE NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    settled_at TIMESTAMPTZ,
    current_weather VARCHAR(32) NOT NULL DEFAULT 'clear', -- 'clear', 'soft_wind', 'passing_rain'
    weather_expires_at TIMESTAMPTZ,
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

-- Birds in Aviary (2 to 7)
CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(), -- Stable internal identity
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id SMALLINT NOT NULL, -- 1 through 6
    name VARCHAR(32) NOT NULL,
    perch_zone VARCHAR(16) NOT NULL DEFAULT 'middle', -- 'front', 'middle', 'back'
    perch_offset REAL NOT NULL DEFAULT 0.5, -- Normalized horizontal offset [0.0, 1.0]
    current_mood VARCHAR(16) NOT NULL DEFAULT 'content', -- 'wary', 'content', 'curious', 'drowsy', 'alert'
    mood_entered_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    last_called_at TIMESTAMPTZ,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

-- Persistent Personality Vector (Server Authoritative)
CREATE TABLE personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(id) ON DELETE CASCADE,
    boldness REAL NOT NULL DEFAULT 0.3 CHECK (boldness BETWEEN 0.0 AND 1.0),
    social_warmth REAL NOT NULL DEFAULT 0.3 CHECK (social_warmth BETWEEN 0.0 AND 1.0),
    vocal_frequency REAL NOT NULL DEFAULT 0.3 CHECK (vocal_frequency BETWEEN 0.0 AND 1.0),
    plumage_saturation REAL NOT NULL DEFAULT 0.3 CHECK (plumage_saturation BETWEEN 0.0 AND 1.0),
    curiosity REAL NOT NULL DEFAULT 0.3 CHECK (curiosity BETWEEN 0.0 AND 1.0),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

-- Append-Only Interaction Event Log
CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    device_session_id UUID NOT NULL REFERENCES sessions(id) ON DELETE CASCADE,
    event_type VARCHAR(32) NOT NULL, -- 'presence_ping', 'listen_in_start', 'listen_in_end', 'offer', 'settle', 'settle_undo'
    target_bird_id UUID REFERENCES birds(id) ON DELETE SET NULL,
    event_payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);
CREATE INDEX idx_events_aviary_created ON interaction_events(aviary_id, created_at);

-- Field Notebook Entries
CREATE TABLE field_notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    entry_prose TEXT NOT NULL,
    noteworthy_event_type VARCHAR(32),
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);
CREATE INDEX idx_notebook_aviary_created ON field_notebook_entries(aviary_id, created_at DESC);

-- Visit Invitations
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    host_account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    visitor_email_encrypted BYTEA NOT NULL,
    visitor_email_hash VARCHAR(64) NOT NULL,
    token_hash VARCHAR(64) UNIQUE NOT NULL,
    status VARCHAR(16) NOT NULL DEFAULT 'active', -- 'active', 'revoked', 'expired'
    expires_at TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

-- Visit Log Entries (Read-only observation history)
CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invitation_id UUID NOT NULL REFERENCES visit_invitations(id) ON DELETE CASCADE,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    visitor_email_masked VARCHAR(64) NOT NULL,
    visited_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    duration_seconds INT NOT NULL DEFAULT 0
);
```

---

## 5. API Surface & Protocols

All APIs adhere to REST semantics over HTTPS. Content type is `application/json`. Error responses on system surfaces strictly employ the matter-of-fact register.

### 5.1 Authentication Endpoints
- `POST /api/v1/auth/magic-link`
  - Body: `{"email": "user@example.com"}`
  - Action: Generates 15-minute token, sends email.
  - Rate limit: 5 requests per hour per email.
  - Response (200 OK): `{"message": "If this address is registered, a sign-in link has been sent."}`
- `POST /api/v1/auth/magic-link/consume`
  - Body: `{"token": "string"}`
  - Response (200 OK): `{"session_token": "string", "account_id": "uuid"}`
  - Error (400 Bad Request): `{"error": "We couldn't sign you in. The link may have expired. Try requesting a new link."}`
- `DELETE /api/v1/auth/sessions/:id`
  - Action: Revokes target device session.

### 5.2 Aviary State & Interaction Endpoints
- `GET /api/v1/aviary/snapshot`
  - Headers: `Authorization: Bearer <session_token>`
  - Response:
    ```json
    {
      "aviary_id": "8fa7e26d-49d2-4cf0-9b48-356a81bf6d71",
      "timestamp": "2026-09-06T15:42:00Z",
      "server_daylight_phase": "morning",
      "weather": { "state": "clear", "intensity": 0.0 },
      "settled": false,
      "settled_at": null,
      "birds": [
        {
          "id": "12f0e0bc-2a54-4731-9f2e-07df974ea111",
          "species_id": 1,
          "name": "pip",
          "perch_zone": "front",
          "perch_offset": 0.32,
          "mood": "content",
          "plumage_saturation": 0.45,
          "cooldowns": { "offer": 0 }
        },
        {
          "id": "78a9c14d-1b32-4114-8f11-99ee81bb2222",
          "species_id": 3,
          "name": "wren",
          "perch_zone": "back",
          "perch_offset": 0.78,
          "mood": "drowsy",
          "plumage_saturation": 0.38,
          "cooldowns": { "offer": 120 }
        }
      ]
    }
    ```
- `POST /api/v1/aviary/events`
  - Headers: `Authorization: Bearer <session_token>`
  - Body:
    ```json
    {
      "events": [
        {
          "type": "presence_ping",
          "client_time": "2026-09-06T15:42:15Z",
          "payload": { "focus": true, "visible": true, "active": true }
        },
        {
          "type": "offer",
          "client_time": "2026-09-06T15:42:20Z",
          "target_bird_id": "12f0e0bc-2a54-4731-9f2e-07df974ea111",
          "payload": { "offer_kind": "seed" }
        }
      ]
    }
    ```
- `GET /api/v1/notebook`
  - Headers: `Authorization: Bearer <session_token>`
  - Query: `?limit=50&before=<timestamp>`
  - Response: Array of naturalist prose entries ordered newest to oldest.

### 5.3 Social & Visit Endpoints
- `POST /api/v1/visits/invitations`
  - Body: `{"visitor_email": "friend@example.com"}`
  - Response: `{"invitation_id": "uuid", "expires_at": "timestamp"}`
- `DELETE /api/v1/visits/invitations/:id`
  - Action: Immediately revokes invite token.
- `GET /api/v1/visits/:token/snapshot`
  - Action: Read-only snapshot for visitors. No auth header required; authenticated via URL token.
  - Returns identical visual/audio aviary snapshot.
  - Rejection (403/404): `{"error": "This visit is no longer available."}` (Matter-of-fact voice).
  - Validation: Strictly rejects any `POST` to `/events`. Does not increment presence time for the host.

### 5.4 Account Settings & Lifecycle Endpoints
- `PATCH /api/v1/account/birds/:id`
  - Body: `{"name": "pippa"}`
- `GET /api/v1/account/export`
  - Triggers asynchronous assembly of aviary JSON payload dispatched to user's verified email.
- `DELETE /api/v1/account`
  - Initiates 30-day soft deletion flag.
- `POST /api/v1/account/restore`
  - Clears soft-deletion marker.

---

## 6. Simulation Engine & Drift Dynamics

The simulation engine is authoritative and runs strictly server-side. Clients never write vectors, simulate mood transitions, or compute drift.

### 6.1 Server-Side Tick Loop
- **Cadence:** Once every 60 seconds per aviary.
- **Worker Concurrency:** Distributed workers acquire a non-blocking Redis lock (`lock:aviary:{aviary_id}`) with a 10s TTL to prevent concurrent ticks across partitions.
- **Tick Execution Pipeline:**
  1. *Event Ingestion:* Read unconsumed entries from `interaction_events` since `last_tick_at`.
  2. *Presence Validation:* Sum presence pings that fulfill the tripartite rule. Convert validated pings into fractional presence minutes $M_p \in [0, 1]$.
  3. *Personality Drift Step:* Calculate monotonic additive deltas for each bird's vector.
  4. *Mood Transition Step:* Evaluate current mood against time-of-day, interaction inputs, ambient weather, and bird-to-bird contagion.
  5. *Perch Positioning:* Update preferred perch zones based on boldness and mood.
  6. *Weather & Daylight Evaluation:* Advance weather timers and recalculate sun elevation from account timezone.
  7. *Notebook Evaluation:* Run sparse naturalist observation generator.
  8. *Snapshot Materialization:* Cache serialized snapshot in Redis for high-speed client fetches and update `aviaries.last_tick_at`.

### 6.2 Personality Drift Function
Personality traits are scalars bounded in $[0.0, 1.0]$. Drift is implemented as a monotonic low-pass filter over accumulated signals:

$$\Delta T_i = lpha_i \cdot \sum (	ext{weighted inputs})$$
$$T_i(t + \Delta t) = \min(1.0, T_i(t) + \Delta T_i)$$

Where:
- **Monotonic Expressiveness Invariant:** $\Delta T_i \ge 0$. Traits never decrease due to absence or inactivity.
- **Drift Weights ($lpha_i$):**
  - *Boldness:* Driven by presence-time ($w=0.6$) and offers placed near bird ($w=0.4$).
  - *Social Warmth:* Driven by presence-time ($w=0.5$) and listen-in duration ($w=0.5$).
  - *Vocal Frequency:* Driven by listen-in focus ($w=0.7$) and unprompted ambient presence ($w=0.3$).
  - *Plumage Saturation:* Driven strictly by cumulative presence-time ($w=1.0$).
  - *Curiosity:* Driven by offer interactions accepted ($w=0.8$) and song-fragment plays ($w=0.2$).
- **Calibration Target:**
  - Regular presence defined as ~30–45 minutes/day, 4–5 days/week.
  - *Week 1 (Instruments threshold):* Vector drift $pprox +0.02 - +0.04$. Detectable via telemetry and test asserts; imperceptible to casual eye.
  - *Week 3 (User visible threshold):* Vector drift $pprox +0.10 - +0.15$. Noticeable shift in perch choice frequency, greeting eagerness, and feather saturation.

### 6.3 Mood State Machine
Mood is a fast-timescale state machine with states: `wary`, `content`, `curious`, `drowsy`, `alert`.

```
                    +--------------------+
                    |       ALERT        |
                    +---------^----------+
                              | (Sudden weather / high curiosity)
       (Offer accepted)       |
     +-------------------> CONTENT <-------------------+
     |                        |                        |
     |                        | (Evening / inactivity) | (Time-of-day morning)
     |                        v                        |
  CURIOUS                  DROWSY ---------------------+
     ^                        |
     | (Offer placed)         | (Alarm contagion)
     |                        v
     +--------------------- WARY
```

- **Transitions:**
  - `wary` $	o$ `content`: Occurs naturally over 15–30 minutes of undisturbed presence or calm weather. High-boldness birds soften twice as fast.
  - `content` $	o$ `drowsy`: Triggered by dusk in user's timezone, or upon `settle` event.
  - `content` $	o$ `curious`: Triggered immediately when an offer (seed/song/pool) is introduced.
  - `alert` $	o$ `wary`: Triggered if a neighboring bird enters wary or an abrupt wind shift occurs.
- **Session Continuity:** Bird mood does not snap to neutral upon tab open. The server preserves the mood state computed at the most recent tick.

### 6.4 Bird-to-Bird Contagion & Chorusing
- If Bird A vocalizes while Bird B has `social_warmth > 0.5`, Bird B has an elevated probability ($40\%$) of responding within 1.5–3.0 seconds.
- When Bird A enters `wary`, adjacent birds on the same perch zone have a $30\%$ chance of shifting from `content` to `alert`.

### 6.5 Aviary Aging & Bird Unlocking
New bird arrivals are governed strictly by aviary creation timestamp, never interaction counts or streaks:
- Birds 1 & 2: Adopted at creation.
- Bird 3: Available at 30 days.
- Bird 4: Available at 90 days.
- Bird 5: Available at 180 days.
- Bird 6: Available at 270 days.
- Bird 7 (Max): Available at 365 days.

---

## 7. Multi-Device Sync & Conflict Prevention

### 7.1 Single Canonical Writer Principle
- The server simulation tick is the **only writer** of personality vectors and mood states.
- Client devices never execute simulation math or propose vector modifications.
- Devices communicate interactions as immutable, timestamped event records appended to `interaction_events`.

### 7.2 Conflict Impossibility
Because clients do not write state, standard multi-device conflict conditions (e.g. Last-Write-Wins overwriting morning progress with an afternoon stale read) are eliminated by architecture.
- Device A (Laptop) registers presence from 09:00 to 09:30.
- Device B (Phone) opens at 09:15.
- Both devices stream presence events to the server. The simulation tick sums valid presence minutes across active sessions without duplicate double-counting within the same 60-second window.
- Both devices pull identical snapshots from `/api/v1/aviary/snapshot`.

### 7.3 Snapshot Interpolation & Graceful Reconnection
- The client receives discrete state snapshots every 60 seconds (or immediately on tab focus).
- To prevent visual teleportation, the frontend interpolation engine smoothly pans birds moving between perches over a 2.5-second curved bezier path.
- When an in-flight network drop occurs, the client displays no blocking dialogs; it renders continuous idle micro-motion from its current local state.
- Upon reconnection, if the session is invalid, the client gracefully displays the matter-of-fact notification: `"Your session timed out. Sign in again to keep watching."`

---

## 8. Frontend Rendering Pipeline

### 8.1 Visual Architecture & Tech Stack
- **Technology:** Custom HTML5 2D Canvas engine wrapped in vanilla TypeScript (zero heavy frameworks; bundle cost <60KB uncompressed).
- **Viewport Contract:** Single horizontal canvas, 16:9 responsive aspect ratio locked, dynamically letterboxed/pillarboxed within the browser window. No scrolling, zooming, or panning.
- **Layers (Back to Front):**
  1. *Sky & Atmosphere:* Procedural linear gradient shifting smoothly by time of day (dawn rose, noon pale blue, dusk ochre, midnight deep navy).
  2. *Backdrop Foliage:* Soft, desaturated distant trees with subtle 0.02x parallax offset on mouse move.
  3. *Back Perch Rail:* Distant branch / perch wire.
  4. *Middle Perch Rail:* Primary resting branch.
  5. *Front Perch Rail:* Closest railing / branch near viewer.
  6. *Ambient Particulates:* Client-side leaf and feather drifts (purely ornamental).
  7. *Top-Bar DOM Chrome:* Semantic HTML overlay positioned above canvas.

### 8.2 The "Motion Already in Progress" Contract
- The aviary must never paint a blank canvas or show a loading spinner.
- The initial HTML response from the edge CDN includes the serialized snapshot in an inline script block: `<script id="bootstrap-snapshot">window.__INIT_AVIARY__ = {...};</script>`.
- Frame 1 renders immediately:
  - Sky is painted to exact local sun position.
  - Birds are positioned at their perch coordinates in mid-idle cycle (e.g. phase offset derived from `(currentTime % period)`).
  - Ambient leaf particles are seeded mid-fall across the screen.
- On cold/uncached loads where bootstrap is unavailable, the canvas paints a peaceful, quiet morning sky with a gentle floating leaf while fetching snapshot data (budget <500ms).

### 8.3 Idle Micro-Motion Engine
Birds are animated procedurally via composite trigonometric oscillations rather than sprite flipbooks:
- **Breathing:** Subtle body expansion/contraction along the Y-axis (cycle 2.8s–4.0s).
- **Head Tilting:** Quick procedural angular shifts (duration 120ms, held 1.5–4.0s).
- **Weight Shuffle:** Subtle lateral hop (3–5px) along the perch rail.
- **Preening:** Rhythmic beak-to-wing rotations triggered when mood is `content`.
- **Fluffed Feathers:** Expanded scale envelope applied when mood is `drowsy` or temperature is cool.

### 8.4 Reduced-Motion Mode (`prefers-reduced-motion`)
When active via browser media query or accessibility setting:
- Frame-by-frame continuous motion is disabled.
- Ambient leaf and feather particles are removed.
- Micro-motions (preening, head turns) are replaced by quiet 1.2-second cross-fades between static illustration poses spaced 15–30 seconds apart.
- Perch transitions cross-fade opacity ($A 	o B$) over 2.0 seconds rather than animating a flight path across the scene.
- Day/night shifts remain, executing over a gradual 10-second transition.

---

## 9. Procedural Audio Pipeline

### 9.1 WebAudio Graph Architecture
All audio is synthesized in real time via the Web Audio API. Zero pre-recorded audio loops or samples are transmitted over the network.

```
+-----------------------------------------------------------------------------------+
|                            PER-BIRD SYNTHESIZER VOICE                             |
|                                                                                   |
|  +---------------------+        +--------------------+      +------------------+  |
|  | FM/Carrier Osc      +------->+ Bandpass Filter    +----->+ Envelope Gain    |  |
|  | (Custom Periodic    |        | (Formant Tracking) |      | (Attack/Decay)   |  |
|  |  Wave / Sine)       |        +--------------------+      +--------+---------+  |
|  +---------------------+                                             |            |
|                                                                      v            |
|  +---------------------+                                    +------------------+  |
|  | Panner Node (2D)    |<-----------------------------------+ Bird Voice Gain  |  |
|  | (Perch X/Z Location)|                                    | (Listen-in Ramp) |  |
|  +----------+----------+                                    +------------------+  |
+-------------|---------------------------------------------------------------------+
              |
              v
+-----------------------------------------------------------------------------------+
|                                MASTER MIX BUS                                     |
|                                                                                   |
|  +---------------------+        +--------------------+      +------------------+  |
|  | Aviary Reverb Node  +------->+ Master Dynamics    +----->+ AudioDestination |  |
|  | (Subtle convolution |        | Compressor         |      | (Speakers)       |  |
|  |  impulse)           |        +--------------------+      +------------------+  |
|  +---------------------+                                                          |
+-----------------------------------------------------------------------------------+
```

### 9.2 Procedural Call Grammar
Each of the 6 species possesses a distinct motif grammar:
1. *Motif Structure:* Sequence of frequency trajectories (chirps, whistles, warbles, trills).
2. *Synthesis Mechanics:* Carrier sine oscillator modulated by a pitch envelope generator with micro-vibrato (5–8 Hz LFO). Formant bandpass filters shape the natural avian timbre.
3. *Variation Engine:* At each call trigger, the engine jitters base pitch ($\pm 3\%$), duration ($\pm 8\%$), and intra-note pause times. Pip's call remains immediately identifiable by its melodic contour while never repeating bit-identically.

### 9.3 Chorus Mixing & Staggered Return-Greeting
- When multiple birds call in proximity, the audio engine enforces a minimum 400ms micro-stagger between call onsets to prevent synthetic synchrony and phase cancellation.
- Master dynamics compressor prevents transient clipping when up to 7 birds participate in a chorus.
- On session return, the greeting bird calls within 1.5 seconds. If a secondary bird responds, its call is queued with a 1.2–2.5 second natural delay.

### 9.4 Listen-In Mix Decay Dynamics
When the user triggers listen-in on a bird:
- Focused bird's voice gain transitions from $1.0 	o 1.8$ over 1500ms using an exponential ramp (`exponentialRampToValueAtTime`).
- All other birds' voice gains transition from $1.0 	o 0.25$ over 2000ms. Other birds are **never muted** completely, preserving the natural spatial depth of the aviary.
- Upon disengaging listen-in, gains restore to unity ($1.0$) over 2000ms.

### 9.5 WebAudio Fallback
If WebAudio is blocked by browser autoplay policy, fails initialization, or is unsupported:
- The aviary immediately falls back to silent operation.
- No recorded audio fallback is attempted (honoring the bundle and anti-canniness constraints).
- Call captions are automatically enabled, rendering the procedural descriptions in real-time.

---

## 10. Accessibility Surfaces

Accessibility in Pocket Aviary is treated as an intentional aesthetic experience rather than an afterthought.

### 10.1 Screen-Reader Narration (`aria-live`)
- **Container:** Dedicated hidden DOM region `<div role="status" aria-live="polite" aria-atomic="true" id="aviary-narrator"></div>`.
- **Voice Register:** Naturalist field-notebook prose. Lowercase, present-tense, observational.
- **Cadence & Queue Management:**
  - Background state is narrated at a sparse rhythm of once every 45–60 seconds.
  - Example: `"a small warbler perches on the front rail, preening softly. the morning light is clear."`
  - High-priority events (return-greeting, accepted offer, settle) interrupt background intervals with immediate prose: `"pip notices you and hops to the lower branch with a short call."`
  - Eliminates ARIA queue buildup by overwriting text rather than appending.

### 10.2 Call Captions
- Positioned dynamically as floating naturalist text near the vocalizing bird.
- Runtime Procedural Generation: Captions reflect the exact synthesized motif (e.g. `"a soft three-note rise"`, `"a rapid descending trill"`).
- Transitions: Opacity fades in over 200ms, persists for call duration plus 800ms, and fades out over 600ms.

### 10.3 Keyboard Navigation & Focus Ring
- Full standard keyboard controls:
  - `Tab` / `Shift+Tab`: Cycles through top-bar affordances (Settings, Accessibility, Field Notebook, Offer).
  - `ArrowLeft` / `ArrowRight`: Moves focus sequentially between birds across perch zones.
  - `Enter` / `Space`: Engages listen-in on focused bird.
  - `Escape`: Disengages listen-in; closes modals.
  - `O`: Shortcut to open Offer tray.
  - `S`: Shortcut to initiate Settle gesture.
- Focus Indicator: 2px solid ring with a 2px outer contrast halo (`box-shadow: 0 0 0 2px #fff, 0 0 0 4px #2c3e50`) providing $\ge 4.5:1$ contrast against both dawn/noon light and deep dusk/night palettes.

### 10.4 Contrast & Palette Standards
- All UI text, top-bar icons, modal dialogs, and captions strictly adhere to WCAG AA standards (minimum 4.5:1 contrast for regular copy, 3:0:1 for large display elements).

---

## 11. Performance Budgets, Telemetry & Privacy Boundary

### 11.1 Hard Performance Budgets
| Metric | Budget | Enforcement Mechanism |
| :--- | :--- | :--- |
| **Initial JS Bundle** | **< 2.0 MB** (gzipped) | CI Webpack/Rollup bundle-analyzer gate. Zero heavy runtime frameworks. |
| **Time to First Bird** | **< 500 ms** (4G mobile) | Inline bootstrap snapshot at CDN edge; no blocking CSS/web-fonts before canvas draw. |
| **Idle Framerate** | **60 fps** (5-yr-old laptop) | Lightweight 2D canvas draw routines, offscreen perch caching. |
| **Memory Leakage** | **0 MB growth / 30 min** | Object pooling for particles and audio nodes; automated CI Puppeteer leak test. |
| **Tick Latency p99** | **< 5.0 s** | Automated backend alerting on simulation worker job queue. |

### 11.2 Privacy Boundary & Telemetry Segmentation
To honor the strict privacy commitment:
- **Allowed Aggregate Operational Telemetry:**
  - HTTP request latencies and status code counts.
  - Simulation tick duration histograms.
  - Client-side WebAudio initialization success/failure rates.
  - Coarse session duration histograms (anonymized, zero account ID association).
- **Prohibited Telemetry (Enforced via Pipeline Segregation):**
  - No per-account or per-bird interaction logs streamed to analytics or data warehouses.
  - No logging of bird names, personality trait values, mood states, or notebook text.
  - Telemetry ingestion pipelines have zero read permissions on the `interaction_events` or `personality_vectors` tables.

---

## 12. Rollout & Calibration Strategy

### 12.1 Phased Delivery Phases
1. **Milestone 1: Audio & Engine Testbed (Weeks 1–4)**
   - Implement WebAudio synthesis engine and 6 species motif grammars.
   - Implement server-side simulation tick and headless drift calibration harness.
2. **Milestone 2: Canvas Rendering & Interaction Core (Weeks 5–8)**
   - Build 2D responsive canvas, perch zoning, and procedural micro-motion.
   - Implement return-greeting, listen-in, offers, and settle gesture.
3. **Milestone 3: Auth, Sync & Field Notebook (Weeks 9–11)**
   - Magic link authentication, synthetic account IDs, multi-device snapshot sync.
   - Sparse field notebook generator and quiet visit invitation flows.
4. **Milestone 4: Accessibility, Performance Hardening & Audit (Weeks 12–14)**
   - ARIA live narrator, reduced-motion cross-fader, WCAG AA compliance audit.
   - Automated 30-minute memory leak tests and bundle size budget checks.
5. **Milestone 5: Production Launch (Week 15)**
   - Deploy v1 with 2 starter birds per account.

### 12.2 Drift Calibration Verification Plan
Before general release, run an automated headless simulation suite simulating 10,000 virtual users across 30 simulated days:
- Validate that $100\%$ of accounts exhibit non-zero vector drift after 7 simulated days of standard visits.
- Verify zero accounts experience negative vector drift under total user absence (monotonicity invariant).
- Verify that users visiting for 48 continuous hours (stress test) do not exceed maximum drift saturation before day 21.

---

## 13. Risk Management & Failure Modes

| Risk Description | Severity | Likelihood | Mitigation Strategy |
| :--- | :---: | :---: | :--- |
| **Drift Calibration Imbalance**<br>Drift happens too fast (Tamagotchi feel) or too slow (screensaver feel). | High | Medium | Continuous verification against 1-week instrument / 3-week visible thresholds using headless simulation harness. |
| **Browser Autoplay Blocks Audio**<br>Browser blocks WebAudio context on page load before user interacts. | High | High | Canvas renders in complete silence without error; displays quiet caption notification; audio context awakens on first user touch/click. |
| **Multi-Device Sync Stutter**<br>Rapid tab switching causes birds to snap across perches. | Medium | Medium | Client uses 2.5s curved bezier interpolation buffer for all perch position changes; ignores snapshots with older timestamps. |
| **Screen-Reader Queue Overload**<br>Narration updates flood screen reader speech queue. | Medium | Low | Use single polite ARIA region with atomic replacement; minimum 30s throttle on idle narrations. |
| **PII Leakage into Observability**<br>User emails accidentally logged in application traces. | High | Low | Synthetic account UUIDs generated at creation. Email stored encrypted in isolated table with blind index. Log scrubbers reject email regexes. |
