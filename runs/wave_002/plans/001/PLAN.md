# Pocket Aviary — System Architecture and Implementation Plan (v1)

## 1. Executive Summary & Scope

### 1.1 Product Vision & Core Tenet
Pocket Aviary is an ambient, browser-based virtual aviary where users cultivate a quiet, observational relationship with up to seven procedurally vocalizing birds across days and weeks. The core engineering imperative is **felt aliveness**: the simulation advances continuously on a canonical server-side tick, visual rendering presents birds already in motion upon first frame, procedural WebAudio synthesis generates non-repeating chorus acoustics, and hidden personality vectors drift monotonically toward expressive behavior in response to honest idle presence.

### 1.2 In-Scope for v1
* **Core Simulation Engine:** Server-side simulation loop (60s tick) computing monotonic low-pass personality drift, Markovian/event-driven mood transitions, and autonomous bird-to-bird social dynamics.
* **Bird Population Model:** Initial 2 starter birds selected from a 6-species pool, scaling with aviary calendar age up to a hard cap of 7 birds. User-assigned stable naming with immutable internal UUIDs.
* **Interaction Surfaces:** Honest presence detection (tri-condition), procedural return-greetings, smooth listen-in mix rebalancing, 3 offer types (seed, song fragment, still pool) with per-bird cooldowns, and opt-in settle evening transition with 5s cancel window.
* **Field Notebook Service:** Naturalist, lowercase, sparse event observation generator synthesizing ~1 entry every few days from noteworthy simulation moments.
* **Client Architecture:** Web-only, responsive single horizontal scene (3 perch zones: front/mid/back), HTML5 Canvas / 2D WebGL scene graph, layered idle micro-motion, procedural day/night solar curve, subtle weather (rain, wind), and ambient drifting particles (leaves/feathers).
* **Procedural Audio Engine:** Client-side WebAudio synthesis utilizing parametric FM/additive synthesis and physical modeling for 6 species motif grammars; dynamic spatial panning, chorus mixing, non-muting listen-in ducking, and graceful silence fallback with live captions.
* **Accounts, Auth & Multi-Device Sync:** Passwordless 15-minute magic links, secure cookie session tokens, synthetic account UUIDs isolating PII, append-only client interaction event stream, and canonical snapshot distribution preventing last-write-wins conflicts.
* **Quiet Social (Visits):** Single-user host invitation via email link generating a read-only, non-co-present ambient view with immediate revocation and 30-day link expiry. No interaction recording or drift mutation from visitors.
* **Accessibility (a11y):** First-class naturalist screen-reader prose narration (30–60s cycle), live descriptive call captions, full keyboard navigation with high-contrast indicators, WCAG AA compliance on all chrome, and dedicated reduced-motion mode (cross-fades between resting poses).
* **Telemetry & Privacy:** Zero aggregation or ML usage of per-bird/per-account interaction data; strict operational telemetry (latencies, WebAudio error rates, bundle size, frame times).

### 1.3 Out-of-Scope (Explicit Non-Goals & Prohibitions)
* **No Gamification:** No streaks, XP, level counters, visit calendars, scores, badges, achievements, or "birds adopted" milestones.
* **No Tamagotchi / Custodial Penalties:** No hunger, sickness, death, decay meters, or negative drift on neglect. Absence produces ambient quietness, never distress.
* **No Social Network Features:** No public feeds, discovery directories, friend-of-friend graphs, visitor comments/chat, avatars, or leaderboards.
* **No Native Apps:** Web platform only. No iOS/Android wrappers or native builds in v1.
* **No Intrusive Announcements:** No "Welcome Back" banners, modals, toasts, or push/email notification loops for visits or streaks.
* **No Numerical Personality Exposure:** Vector weights (boldness, warmth, etc.) are strictly internal server state and never exposed via UI or debug APIs.

---

## 2. System Architecture & Service Topology

```
                              +-------------------------------------------+
                              |         Edge CDN / Static Assets          |
                              |  (Vite App Bundle <2MB, Assets, SVGs)     |
                              +-------------------------------------------+
                                                    |
                                      HTTPS/WSS     | Snapshot / Auth
                                                    v
+---------------------------------------------------------------------------------------------------+
|                                      API Gateway & Web Tier                                       |
|  - Auth & Magic Link Verification                                                                 |
|  - Rate Limiting & Session Management (Synthetic UUID routing)                                    |
|  - REST Snapshot Pulls (`GET /api/v1/aviary/snapshot`)                                             |
|  - Append-Only Event Ingestion (`POST /api/v1/aviary/events`)                                      |
|  - Read-Only Visit Proxy (`GET /api/v1/visits/:token/snapshot`)                                   |
+---------------------------------------------------------------------------------------------------+
             |                                              |
     Event Batch Append                             Read Snapshot Cache
             v                                              v
+-----------------------------+               +----------------------------------+
|   PostgreSQL / CockroachDB  |               |       Redis State & Cache        |
| - `accounts` (Encrypted PII)|               | - Live Aviary State Snapshots    |
| - `aviaries` & `birds`      |               | - Visit Token Fast Lookup        |
| - `interaction_events`      |<--------------| - Magic Link Token Store         |
| - `notebook_entries`        |  Write State  | - Active Session Registry        |
| - `visit_invitations`       |               +----------------------------------+
+-----------------------------+                                ^
             ^                                                 | Periodic State
             | Consume Events & Drift                          | Update
             +--------------------+   +------------------------+
                                  |   |
+---------------------------------------------------------------------------------------------------+
|                                Simulation Worker Cluster (Ticks)                                  |
|  - Background Worker Loop (60-second ticks per aviary partition)                                  |
|  - Monotonic Personality Drift Low-Pass Integrator                                                |
|  - Mood State Machine (Markov transitions + ambient weather + time-of-day)                        |
|  - Autonomous Bird Social & Chorus Decision Engine                                                |
|  - Sparse Field Notebook Observation Synthesizer                                                  |
+---------------------------------------------------------------------------------------------------+
```

### 2.1 Service Boundaries
1. **Frontend Client (SPA):** Single-page application written in TypeScript with Vanilla Canvas/WebGL rendering and WebAudio synthesis. No heavyweight runtime frameworks (e.g. pure TypeScript with lightweight reactive store). Initial bundle <2MB gzipped.
2. **API & Edge Gateway:** Stateless Node.js/Go HTTP service handling magic-link auth, session validation, event ingestion, and snapshot serving.
3. **Simulation Tick Worker Engine:** Horizontally partitioned background workers processing active and dormant aviaries every 60 seconds. Partitioned by `account_id` hash.
4. **Primary Relational Store:** PostgreSQL for transactional persistence (accounts, birds, event logs, notebook entries, visits).
5. **Fast State Cache:** Redis for fast snapshot reads, magic-link validation, rate limiting, and ephemeral presence buffering.

### 2.2 Client/Server Execution Boundary
* **Server Authority:**
  * Canonical bird personality vectors ($V$).
  * Canonical bird mood state ($M$) and mood duration timers.
  * Aviary clock, weather cycle generation, and age-based species unlocks.
  * Monotonic drift integration from verified event streams.
  * Sparse field notebook generation and persistence.
* **Client Authority:**
  * Procedural WebAudio sound synthesis and chorus voice scheduling based on server motif seeds.
  * Local high-frequency visual interpolation (60fps micro-motion, smooth flight paths between perch zones).
  * Local time-of-day solar curve visual blending.
  * Tri-condition presence detection and periodic batched event dispatch.
  * Ambient particle physics (leaves, feather drift).

---

## 3. Data Model & Storage Schema

```
  +-------------------+       1:1       +--------------------+
  |     accounts      |---------------->|      aviaries      |
  +-------------------+                 +--------------------+
  | id (UUID PK)      |                 | id (UUID PK)       |
  | email_encrypted   |                 | account_id (FK)    |
  | created_at        |                 | created_at         |
  | soft_deleted_at   |                 | timezone           |
  | settings_json     |                 | current_weather    |
  +-------------------+                 | weather_started_at |
           | 1:N                        +--------------------+
           |                                     | 1:N
           v                                     |
  +--------------------+                         +----------------------+
  |  auth_sessions     |                         |        birds         |
  +--------------------+                         +----------------------+
  | id (UUID PK)       |                         | id (UUID PK)         |
  | account_id (FK)    |                         | aviary_id (FK)       |
  | device_label       |                         | name (VARCHAR)       |
  | refresh_token_hash |                         | species_id (ENUM)    |
  | last_active_at     |                         | adopted_at           |
  | revoked_at         |                         | perch_zone (ENUM)    |
  +--------------------+                         | personality_boldness |
           | 1:N                                 | personality_warmth   |
           |                                     | personality_vocal    |
           v                                     | personality_plumage  |
  +--------------------+                         | personality_curious  |
  | visit_invitations  |                         | current_mood (ENUM)  |
  +--------------------+                         | mood_updated_at      |
  | id (UUID PK)       |                         +----------------------+
  | account_id (FK)    |                                 |
  | token_hash         |                                 | 1:N
  | recipient_email_enc|                                 v
  | created_at         |                 +--------------------------------+
  | expires_at         |                 |       interaction_events       |
  | revoked_at         |                 +--------------------------------+
  +--------------------+                 | id (BIGSERIAL PK)              |
           | 1:N                         | aviary_id (FK)                 |
           v                         +-->| bird_id (UUID FK, nullable)    |
  +--------------------+             |   | event_type (ENUM)              |
  |  notebook_entries  |             |   | client_timestamp               |
  +--------------------+             |   | payload_json                   |
  | id (UUID PK)       |             |   | processed_in_tick_at           |
  | aviary_id (FK)     |             |   +--------------------------------+
  | entry_text (TEXT)  |             |
  | generated_at       |-------------+
  +--------------------+
```

### 3.1 PostgreSQL Relational Schema

```sql
-- Core Account & Privacy Isolation
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_bidx VARCHAR(64) NOT NULL UNIQUE, -- HMAC blind index for deterministic lookup
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    soft_deleted_at TIMESTAMPTZ NULL,
    settings JSONB NOT NULL DEFAULT '{
        "visit_notifications_enabled": false,
        "reduced_motion": false,
        "captions_enabled": false
    }'::jsonb
);

CREATE TABLE auth_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    device_label VARCHAR(128) NOT NULL,
    session_token_hash VARCHAR(64) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at TIMESTAMPTZ NULL
);

-- Aviary and Bird Entities
CREATE TYPE weather_state AS ENUM ('clear', 'passing_rain', 'soft_wind');
CREATE TYPE perch_zone AS ENUM ('front', 'middle', 'back');
CREATE TYPE mood_state AS ENUM ('wary', 'content', 'curious', 'drowsy', 'alert');
CREATE TYPE species_type AS ENUM ('warbler', 'wren', 'sparrow', 'finch', 'chickadee', 'nightjar');

CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    current_weather weather_state NOT NULL DEFAULT 'clear',
    weather_started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_presence_at TIMESTAMPTZ NULL,
    total_presence_seconds BIGINT NOT NULL DEFAULT 0
);

CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species species_type NOT NULL,
    name VARCHAR(32) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    current_perch perch_zone NOT NULL DEFAULT 'middle',
    current_mood mood_state NOT NULL DEFAULT 'content',
    mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    -- Hidden Normalized Personality Vectors (0.0000 to 1.0000)
    boldness NUMERIC(5,4) NOT NULL CHECK (boldness >= 0 AND boldness <= 1),
    social_warmth NUMERIC(5,4) NOT NULL CHECK (social_warmth >= 0 AND social_warmth <= 1),
    vocal_frequency NUMERIC(5,4) NOT NULL CHECK (vocal_frequency >= 0 AND vocal_frequency <= 1),
    plumage_saturation NUMERIC(5,4) NOT NULL CHECK (plumage_saturation >= 0 AND plumage_saturation <= 1),
    curiosity NUMERIC(5,4) NOT NULL CHECK (curiosity >= 0 AND curiosity <= 1)
);

-- Append-Only Interaction Events Stream
CREATE TYPE interaction_type AS ENUM (
    'presence_ping',
    'listen_in_start',
    'listen_in_end',
    'offer_seed',
    'offer_song',
    'offer_pool',
    'settle',
    'settle_undo',
    'bird_rename'
);

CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(id) ON DELETE SET NULL,
    event_type interaction_type NOT NULL,
    client_timestamp TIMESTAMPTZ NOT NULL,
    received_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    processed_at TIMESTAMPTZ NULL
);
CREATE INDEX idx_events_unprocessed ON interaction_events (aviary_id, id) WHERE processed_at IS NULL;

-- Field Notebook
CREATE TABLE notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    entry_text TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_notebook_aviary ON notebook_entries (aviary_id, created_at DESC);

-- Social Visits
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    recipient_email_encrypted BYTEA NOT NULL,
    recipient_email_bidx VARCHAR(64) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ NULL,
    last_visited_at TIMESTAMPTZ NULL
);
```

---

## 4. API Surface & Communication Protocols

All client APIs conform to a minimal REST/JSON protocol over HTTPS. Session identification is provided by an `HttpOnly`, `SameSite=Lax`, `Secure` cookie containing a signed JWT / session token.

### 4.1 Authentication Endpoints
* `POST /api/v1/auth/magic-link/request`
  * **Payload:** `{ "email": "user@example.com" }`
  * **Behavior:** Generates a 15-minute cryptographically random token, hashes it to Redis, sends email link `https://pocketaviary.app/auth/verify?token=...`.
  * **Rate Limit:** 3 requests per 15 minutes per IP/email.
* `POST /api/v1/auth/magic-link/verify`
  * **Payload:** `{ "token": "raw_token_value" }`
  * **Response:** Sets `session_token` cookie. Returns `{ "account_id": "<uuid>", "is_new_account": false }`. Invalidates token immediately.
* `POST /api/v1/auth/session/revoke`
  * **Payload:** `{ "session_id": "<uuid>" }`
  * **Response:** Revokes specified session.

### 4.2 Aviary State & Event Submission
* `GET /api/v1/aviary/snapshot`
  * **Response Status:** `200 OK`
  * **Payload Schema:**
```json
{
  "aviary_id": "8fa3c72b-8a8f-40e1-b4f0-464817a3a968",
  "server_time": "2026-08-13T18:24:00.000Z",
  "timezone": "America/Los_Angeles",
  "weather": {
    "state": "clear",
    "transition_progress": 0.0
  },
  "settled": false,
  "birds": [
    {
      "id": "27c11f7c-7d9a-4c22-b5e1-889895cba120",
      "name": "Pip",
      "species": "warbler",
      "perch_zone": "front",
      "mood": "curious",
      "plumage_saturation": 0.642,
      "motif_seed": 48291
    },
    {
      "id": "e932b130-9bc5-42d8-bfba-cf3b9b4f4c28",
      "name": "Wren",
      "species": "wren",
      "perch_zone": "back",
      "mood": "content",
      "plumage_saturation": 0.510,
      "motif_seed": 10928
    }
  ]
}
```
* `POST /api/v1/aviary/events`
  * **Payload:**
```json
{
  "events": [
    {
      "event_type": "presence_ping",
      "client_timestamp": "2026-08-13T18:23:45.120Z",
      "payload": { "presence_duration_seconds": 30 }
    },
    {
      "event_type": "offer_seed",
      "bird_id": "27c11f7c-7d9a-4c22-b5e1-889895cba120",
      "client_timestamp": "2026-08-13T18:23:55.000Z",
      "payload": {}
    }
  ]
}
```
  * **Response:** `{ "status": "accepted", "queued_count": 2 }`

### 4.3 Field Notebook & Settings
* `GET /api/v1/notebook`
  * **Query Params:** `?limit=50&before=<timestamp>`
  * **Response:**
```json
{
  "entries": [
    {
      "id": "a9881fc0-760a-4712-881c-d784fa72a291",
      "created_at": "2026-08-12T09:15:22.000Z",
      "text": "pip greeted before wren today, first time this week."
    },
    {
      "id": "41e9b251-5122-4467-bc1a-5e72cc872412",
      "created_at": "2026-08-10T16:42:01.000Z",
      "text": "wren is fluffed against the cool air, watching the back perch. low calls only."
    }
  ]
}
```

### 4.4 Visit Invitation Flow (Quiet Social)
* `POST /api/v1/visits/invite`
  * **Payload:** `{ "recipient_email": "friend@example.com" }`
  * **Response:** `{ "invite_id": "<uuid>", "expires_at": "..." }`
* `GET /api/v1/visits/active`
  * **Response:** Returns list of active invites and recent silent visit logs (email blind-index decrypted for host view, visit timestamp, duration).
* `DELETE /api/v1/visits/:invite_id`
  * **Response:** Immediately marks invite revoked.
* `GET /api/v1/visits/view/:token`
  * **Response:** Validates token against Redis/PostgreSQL. Returns read-only aviary snapshot. Rejects with matter-of-fact 404/410 message if revoked or expired.

---

## 5. Simulation Engine Design & Mechanics

### 5.1 The 60-Second Server-Side Tick Loop
The simulation engine executes on a dedicated worker cluster running an asynchronous job queue partitioned by `aviary_id`. Every 60 seconds per aviary:
1. **Event Drain:** Pulls and locks all unconsumed rows from `interaction_events` for the aviary.
2. **Presence Aggregation:** Verifies and sums `presence_ping` intervals.
3. **Monotonic Low-Pass Drift Step:** Calculates delta updates for each bird's hidden vector.
4. **Mood & Behavior State Transitions:** Evaluates Markov transition matrix modulated by current time-of-day, active weather, and recent interaction events.
5. **Perch Selection:** Updates target perch zone ($P \in \{\text{front, middle, back}\}$) based on boldness and mood.
6. **Observation Synthesis:** Evaluates trigger conditions against the rare Field Notebook generator rule.
7. **Snapshot Serialization:** Writes the updated canonical snapshot to Redis (`SET aviary:snapshot:<id>`) with a 5-minute TTL.

### 5.2 Monotonic Low-Pass Drift Function
Personality vectors $V = [v_{\text{bold}}, v_{\text{warm}}, v_{\text{vocal}}, v_{\text{plum}}, v_{\text{curious}}]^T \in [0.0, 1.0]^5$ evolve strictly monotonically toward expressive states.

$$\Delta v_i = \alpha_i \cdot \max(0, S_i(t) - v_i(t)) \cdot \Delta t$$

Where:
* $S_i(t)$ represents the instantaneous interaction/presence stimulus:
  * Presence time $T_p$: Primary driver for $v_{\text{plum}}$ and $v_{\text{warm}}$.
  * Listen-in duration $T_l$: Direct driver for $v_{\text{warm}}$ and $v_{\text{vocal}}$.
  * Offer acceptance $O_{\text{accepted}}$: Direct driver for $v_{\text{curious}}$ and $v_{\text{bold}}$.
* $\alpha_i$ is a calibrated low-pass filter constant calibrated to the PRD benchmarks:
  * **Instrumentally detectable:** $\Delta v_i \approx 0.03$ after ~7 days of regular presence (15 min/day).
  * **Visually/audibly distinct to user:** $\Delta v_i \approx 0.12$ after ~21 days.
  * Formula parameter: $\alpha_{\text{presence}} \approx 1.5 \times 10^{-7} \text{ s}^{-1}$.
* **Asymmetric Non-Decay Invariant:** If $S_i(t) < v_i(t)$ (user is absent for days), $\Delta v_i = 0$. Values never decrease. An unvisited bird maintains its traits and plumage saturation, merely becoming ambient in moment-to-moment behavior.

### 5.3 Mood Transition Engine
Mood $M \in \{\text{wary}, \text{content}, \text{curious}, \text{drowsy}, \text{alert}\}$ operates on a fast timescale (resets/drifts over hours):
* **Local Solar Time Weight:**
  * Dawn / Early Morning (05:00 - 09:00): +40% transition probability toward `alert`.
  * Midday (09:00 - 17:00): +50% baseline `content`.
  * Dusk / Evening (17:00 - 21:00): +60% transition toward `drowsy`.
  * Night (21:00 - 05:00): 100% `drowsy` / sleeping, except species `nightjar` which enters `alert` or `content`.
* **Interaction Nudges:**
  * Accepted offer $\to$ instant transition to `content` or `curious` (duration 10-20 min).
  * Passing rain weather $\to$ dampens call rate by 70%, nudges to `wary` or `drowsy`.
  * High boldness trait suppresses `wary` entry probability by $1 - 0.7 \cdot v_{\text{bold}}$.

### 5.4 Call Grammar & Chorus Engine
Each species defines a formal Markovian call motif grammar $G = (S, \Sigma, T, \mu)$:
* **Motifs ($\Sigma$):** Short frequency-modulated pitch vectors (e.g., rising two-note, trill, drop-peep, chirrup).
* **Call Generation:** At tick time, the server generates a deterministic seed `motif_seed` based on vocal frequency and current mood.
* **Bird-to-Bird Social Call Coupling:**
  * When bird $A$ vocalizes, nearby birds with $v_{\text{warm}} > 0.4$ have a $P_{\text{reply}} = 0.5 \cdot v_{\text{warm}}$ chance of scheduling a response with a random delay offset of $0.8\text{s} - 2.5\text{s}$.
  * Staggered responses prevent synthetic unison calling and produce natural avian chorus dynamics.

### 5.5 Species Pool & Population Age Gating
* **Species Pool (6 v1 Types):**
  1. *Warbler:* High pitch, rapid ascending trills, high base curiosity.
  2. *Wren:* Dynamic staccato chirps, active head-tilting, mid perch affinity.
  3. *Sparrow:* Low dual-tone peep, high social flocking warmth, front perch affinity.
  4. *Finch:* Melodic sliding whistle, high plumage saturation ceiling.
  5. *Chickadee:* Characteristic multi-tone call, high boldness.
  6. *Nightjar:* Nocturnal, soft resonant evening churring, active after dusk.
* **Adoption Unlock Schedule (Calendar Age Based):**
  * Day 0: 2 starter birds assigned from non-nocturnal pool.
  * Day 60 (~2 months): 3rd bird arrives.
  * Day 150 (~5 months): 4th bird arrives.
  * Day 270 (~9 months): 5th bird arrives.
  * Day 365 (1 year): 6th bird arrives.
  * Day 500 (~1.4 years): 7th bird arrives (Hard Cap reached).

---

## 6. Multi-Device Sync & Conflict Prevention

```
                     +---------------------------------------+
                     |    Client Device A (e.g., Laptop)     |
                     |  - Renders Canvas & Procedural Audio  |
                     |  - Collects Local Presence & Gestures |
                     +---------------------------------------+
                                         |
                                         | 1. HTTP POST /api/v1/aviary/events
                                         |    (Append-Only: presence, offer)
                                         v
+---------------------------------------------------------------------------------------------------+
|                                 Server-Side Simulation Authority                                  |
|                                                                                                   |
|   [ interaction_events Table ]                                                                    |
|             |                                                                                     |
|             v                                                                                     |
|   [ 60s Simulation Tick Worker ]                                                                   |
|     - Computes Additive Personality Vector Drift                                                  |
|     - Updates Moods, Perch Positions & Weather                                                    |
|     - Commits Canonical State to PostgreSQL & Redis Snapshot Cache                                |
+---------------------------------------------------------------------------------------------------+
                                         |
                                         | 2. HTTP GET /api/v1/aviary/snapshot
                                         |    (Pulls updated canonical state)
                                         v
                     +---------------------------------------+
                     |     Client Device B (e.g., Phone)     |
                     |  - Reads SAME Canonical State Snapshot|
                     |  - Zero Client-to-Client Conflict     |
                     +---------------------------------------+
```

### 6.1 Strict Single-Writer Principle
* **The Server is the Sole Writer:** Clients NEVER submit personality vector values, mood states, or aviary clocks.
* **No Last-Write-Wins (LWW) Data Corruption:** By transforming all client mutations into append-only interaction events, concurrent sessions across a user's laptop and phone simply append events to the same stream. The server processes them sequentially in timestamp order, accumulating additive deltas.
* **Snapshot Fetch Cadence:**
  * On initial load / tab visibility change (`visibilitychange -> visible`).
  * On wake from system sleep ($\Delta t_{\text{frame}} > 5000\text{ms}$).
  * Low-frequency keepalive poll every 60 seconds while tab remains visible and focused.

### 6.2 Error Surfaces & Voice Separation
System errors use matter-of-fact tone, strictly avoiding naturalist prose:
* *Magic Link Expired:* "We couldn't sign you in. The link may have expired. Try requesting a new link."
* *Session Conflict / Revoked:* "Your session was ended from another device. Sign in again to keep watching."
* *Network Failure:* "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."

---

## 7. Client-Side Rendering Pipeline

```
                                    +----------------------------------+
                                    |     RequestAnimationFrame Loop   |
                                    +----------------------------------+
                                                     |
                         +---------------------------+---------------------------+
                         |                                                       |
                         v                                                       v
         [ Reduced-Motion Mode Active? ]                               [ Normal Render Mode ]
                         |                                                       |
          +--------------+--------------+                         +--------------+--------------+
          |                             |                         |                             |
          v                             v                         v                             v
[ Static Pose Render ]       [ Slow Cross-Fade (800ms) ]  [ Procedural Micro-Motion ] [ Particle Physics Loop ]
- Fixed Perch Offsets        - Alpha blend between poses  - Preening, head-tilts,     - Subtle drifting leaves
- No ambient particles       - Position cross-fade          breathing, weight-shifts    and feather glints
          |                             |                         |                             |
          +--------------+--------------+                         +--------------+--------------+
                         |                                                       |
                         +---------------------------+---------------------------+
                                                     |
                                                     v
                                     +--------------------------------+
                                     |    Perch Zone Compositing      |
                                     | - Back Plane: Foliage & Sky    |
                                     | - Middle Plane: Main Perches   |
                                     | - Front Plane: Fore Perch/Pool |
                                     +--------------------------------+
                                                     |
                                                     v
                                     +--------------------------------+
                                     |     Solar Palette Shading      |
                                     | (Linear color grading for TOD) |
                                     +--------------------------------+
```

### 7.1 Scene Graph & Composition
The visual surface is rendered onto a single responsive `<canvas>` element using a 2D canvas context with sub-pixel crispness ($2\times$ DPR scaling):
1. **Backdrop Layer:** Sky gradient computed dynamically from local solar angle (dawn ochres, midday cerulean, dusk violet-ambers, night deep navy). Background soft foliage with gentle single-pole parallax.
2. **Back Perch Zone ($Z=0$):** Scaled at $0.75\times$, slightly desaturated plumage shading.
3. **Middle Perch Zone ($Z=1$):** Scaled at $1.0\times$, standard branch assets.
4. **Front Perch Zone ($Z=2$):** Scaled at $1.25\times$, full feather detail and specular glints.
5. **Foreground Layer:** Occasional ambient leaf or feather particles drifting along a randomized Perlin noise vector.

### 7.2 Zero-Loading-State Initialization
* **Immediate First Paint:** The static HTML shell contains the pre-computed canvas sky gradient.
* **Fast State Hydration:** If cached locally in `sessionStorage`, birds are rendered immediately at their last known coordinates and motions. When the network snapshot arrives (<500ms), positions and states interpolate smoothly without a pop.
* **No Spinners:** If waiting on a cold network connection, the client renders the quiet empty field (soft sky, subtle branch silhouette) with no loading spinners or progress bars.

### 7.3 Micro-Motion Engine
Idle motion is driven by procedural skeletal math rather than frame-by-frame sprites:
* **Breathing:** Low-frequency sine expansion ($0.2\text{Hz}$) on bird torso geometry.
* **Head-Tilt & Scan:** Stochastic jump-and-hold rotational transforms triggered every $3\text{s} - 8\text{s}$.
* **Preen & Shuffle:** Occasional localized feather jitter and wing fluffs modulated by `content` mood.
* **Flight Transitions:** Smooth cubic Bézier arcs between perch coordinates with wing-beat frequency linked to species mass.

### 7.4 Reduced-Motion Pipeline
When `prefers-reduced-motion: reduce` or account setting is enabled:
* Micro-motion skeletal animations and wing beats are disabled.
* Flight transitions are replaced with an $800\text{ms}$ cross-fade between static resting poses.
* Particle generators (leaves/feathers) are paused.
* Day/night color shifts transition over slow 10-second cross-fades.

### 7.5 Top Bar UI Chrome
* Contains only 4 minimalist SVG icons: Account/Settings, Accessibility, Field Notebook, and Offer Affordance.
* **Auto-Fade Behavior:** After 3.5 seconds of cursor inactivity, opacity transitions to `0.08` over $1.2\text{s}$. Pointer movement or keyboard focus instantly restores opacity to `1.0`.

---

## 8. WebAudio Procedural Sound Architecture

```
+---------------------------------------------------------------------------------------------------+
|                                 Master Audio Context (WebAudio)                                   |
+---------------------------------------------------------------------------------------------------+
                                                  |
           +--------------------------------------+--------------------------------------+
           |                                                                             |
           v                                                                             v
+-----------------------------+                                               +---------------------+
|  Background Ambient Bus     |                                               |  Bird Synthesis Bus |
|  - Soft wind filter (pink)  |                                               +---------------------+
|  - Rain generator (white)   |                                                          |
|  - Gain: 0.15               |                                     +--------------------+--------------------+
+-----------------------------+                                     |                                         |
               |                                                    v                                         v
               |                                     +-----------------------------+           +-----------------------------+
               |                                     | Bird Voice Channel: Pip     |           | Bird Voice Channel: Wren    |
               |                                     | - FM Synth Carrier/Mod      |           | - Dual Biquad Filter Bank   |
               |                                     | - ADSR Gain Envelope        |           | - Stereo Panner (-0.6)      |
               |                                     | - Stereo Panner (+0.5)      |           | - Dynamic Gain Node (Ducking|
               |                                     | - Dynamic Gain Node         |           +-----------------------------+
               |                                     +-----------------------------+                          |
               |                                                    |                                         |
               +----------------------------------------------------+-----------------------------------------+
                                                                    |
                                                                    v
                                                     +-----------------------------+
                                                     |    Master Dynamic Limiter   |
                                                     |     & Soft Saturation       |
                                                     +-----------------------------+
                                                                    |
                                                                    v
                                                     +-----------------------------+
                                                     |   AudioDestination (Output) |
                                                     +-----------------------------+
```

### 8.1 Procedural Synthesis Engine (No Recorded Loops)
Sound generation uses pure WebAudio nodes (`OscillatorNode`, `BiquadFilterNode`, `GainNode`, `WaveShaperNode`):
* **Carrier & Modulator FM:** Fast frequency chirps and whistles synthesized via FM synthesis where pitch modulation envelopes mimic avian syrinx mechanics.
* **Species-Specific Formants:** Biquad bandpass filters shape resonance according to body cavity acoustics of warblers, wrens, or finches.
* **Zero Audio Files:** Total audio bundle size is 0 bytes of audio media (pure mathematical code <35KB).

### 8.2 Mix Matrix & Listen-In Ducking
* **Dynamic Range Management:** Total master bus gain is dynamically scaled by $1 / \sqrt{N}$ where $N$ is active singing voices.
* **Listen-In Focus Transition:**
  * When user engages Listen-In on Bird $K$:
    * Bird $K$ Gain ramps from $1.0 \to 1.8$ over $1200\text{ms}$ using `exponentialRampToValueAtTime`.
    * All other birds $J \neq K$ Gain ramps from $1.0 \to 0.22$ over $1200\text{ms}$.
    * Ambient background noise drops by $6\text{dB}$.
    * Other birds NEVER mute completely (preserving flock presence).
  * On disengage, all buses ramp back to baseline $1.0$ over $1200\text{ms}$.

### 8.3 WebAudio Fallback
If `AudioContext` fails to initialize (user gesture blocked, unsupported device, hardware error):
* The audio engine enters graceful silent mode.
* Call captioning is automatically enabled by default in the visual scene.
* No pre-recorded sample fallback is loaded, protecting bundle budgets and preventing canned audio artifacts.

---

## 9. Accessibility (a11y) Architecture

### 9.1 Naturalist Screen-Reader Narration Surface
* **ARIA Live Architecture:** Utilizes a dedicated `<div role="status" aria-live="polite" aria-atomic="true" class="sr-only">` element.
* **Prose Generation Engine:**
  * Runs every $30\text{s} - 60\text{s}$ at idle.
  * Translates raw positions and moods into literary, lowercase naturalist prose:
    * *"a warbler perches on the high branch, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."*
  * User gestures (offer, settle, return) trigger immediate observation updates.

### 9.2 Call Captioning Subsystem
* Rendered directly adjacent to the calling bird in the visual viewport (as well as mirrored to `aria-live="polite"`).
* Text generated dynamically from the active motif:
  * *"a soft three-note rise"*
  * *"a low trill, paused, low trill again"*
  * *"a single sharp call from the back perch"*
* Text opacity fades in over $200\text{ms}$ and out over $1000\text{ms}$ synchronized with the synth envelope.

### 9.3 Keyboard Navigation & Focus Ring
* `Tab` cycles through Top Bar $\to$ Aviary Birds $\to$ Active Overlays.
* Inside Aviary:
  * `ArrowLeft` / `ArrowRight` navigates focus across birds by horizontal position.
  * `Enter` / `Space` triggers Listen-In on focused bird.
  * `Escape` exits Listen-In or closes Notebook modal.
  * `O` key shortcut opens Offer tray; `S` key triggers Settle confirmation.
* Focus rings use a high-contrast dual outline (`2px solid #FFFFFF` with `1px solid #1A2421` shadow) ensuring WCAG AA visibility against any solar palette lighting.

---

## 10. Performance Budgets & Observability

### 10.1 Hard Performance Budgets
| Metric | Budget Target | Enforcement Mechanism |
| :--- | :--- | :--- |
| **Initial JS Bundle (gzipped)** | $< 2.0\text{ MB}$ (Target: $< 350\text{ KB}$) | CI bundle-size check; automated build failure if breached. |
| **Time to First Bird Visible (TTFBird)** | $< 500\text{ ms}$ over 4G mobile | Edge snapshot delivery + canvas pre-render before network resolution. |
| **Idle Animation Framerate** | $60\text{ fps}$ continuous | requestAnimationFrame delta budget ($< 16.6\text{ ms}$ per frame). |
| **Memory Leak Ceiling** | $0.00\text{ MB}$ growth over 30 min | Reusable WebAudio nodes, pre-allocated particle buffers, CI leak tests. |
| **Simulation Tick Latency (p99)** | $< 5.0\text{ s}$ per worker batch | Prometheus/Grafana alert on tick execution duration. |

### 10.2 Observability & Privacy-Preserving Telemetry
* **Collected Metrics (Operational Only):**
  * HTTP request durations and error status codes.
  * Client render loop frame drop rates (`fps_bucket`).
  * WebAudio initialization failure rate.
  * Simulation tick queue depth and execution latency.
* **Strict Telemetry Exclusions:**
  * NO per-bird trait values, names, or mood distributions.
  * NO per-account interaction histories, offer frequencies, or visit logs.
  * NO cross-session tracking identifiers or advertising analytics.

---

## 11. Rollout & Release Plan

### 11.1 Phased Rollout Schedule
1. **Milestone 1: Core Engine & Single-Device Prototype (Weeks 1–4)**
   * Headless simulation tick engine, PostgreSQL schema, 6-species WebAudio synthesis library.
2. **Milestone 2: Canvas Rendering & Interaction Loop (Weeks 5–8)**
   * Responsive canvas scene, micro-motion physics, tri-condition presence tracker, 3 offer types.
3. **Milestone 3: Auth, Sync & Field Notebook (Weeks 9–11)**
   * Magic link auth, multi-device snapshot propagation, naturalist observation text engine.
4. **Milestone 4: Accessibility & Performance Hardening (Weeks 12–14)**
   * Screen-reader narration, reduced-motion cross-fades, call captions, bundle optimization.
5. **Milestone 5: Quiet Social & Beta Launch (Weeks 15–16)**
   * Read-only visit invitation links, revocation testing, canary deployment to initial user group.

### 11.2 Day-1 Instrumentation
* Synthetic end-to-end browser health runners testing magic link latency and canvas first-frame paint from 5 global edge locations.
* Sentry error logging scrubbed of all email/PII attributes.

---

## 12. Risk Matrix & Mitigation Strategies

| Risk Category | Potential Failure Mode | Technical Mitigation Strategy |
| :--- | :--- | :--- |
| **Drift Calibration Risk** | Personality vectors drift too fast (Tamagotchi effect) or too slow (screensaver effect). | Implement automated headless multi-month simulation test suites asserting that trait changes match the 7-day instrumental / 21-day user-visible milestones across various presence profiles. |
| **Sync Race Conditions** | Concurrent writes from two devices causing state divergence or lost interaction credit. | Enforce strict append-only interaction event ingestion. Personality vectors are computed exclusively by the server tick worker; clients only consume read-only snapshots. |
| **Audio Uncanniness** | Procedural FM synthesis sounds harsh, robotic, or fatiguing over extended listening sessions. | Apply physical modeling resonator filters, subtle randomized pitch micro-deviations ($\pm 1.5\%$), and harmonic overtone saturation matching biological syrinx behavior. |
| **Accessibility Degradation** | ARIA live region spamming screen readers or desynchronizing from scene state. | Throttle narration updates to $30\text{s} - 60\text{s}$ cycles with priority interruption only for direct user gestures (Offer/Settle). Unit test prose generator against WCAG AA standards. |
| **Presence False Positives** | Background tabs or unattended open laptops inflating presence time. | Strict tri-condition enforcement (`document.visibilityState === 'visible'` AND `document.hasFocus()` AND user input within last 3 minutes). Pings drop immediately if any condition fails. |
