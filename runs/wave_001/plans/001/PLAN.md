# Pocket Aviary — Phase 1 Comprehensive Implementation Plan (v1)

## 1. Executive Summary & Product Scope

### 1.1 Headline Concept & Design Philosophy
Pocket Aviary is a lightweight, browser-based ambient virtual aviary designed for modern desktop and mobile browsers. The product establishes an observational, unhurried relationship between the user and a small flock of animated birds (starting at 2, capping strictly at 7). The birds live in a single horizontal scene rendered directly in a web tab.

The architecture and design are guided by five fundamental design principles:
1. **Feels alive, not robotic**: The aviary continues independently of the user through a server-side simulation tick. The first frame rendered has motion already underway; audio calls and greetings are procedurally varied so no two moments sound identical.
2. **Notice, never announce**: The system registers user arrival through subtle bird behavioral reactions (glances, head-tilts, soft calls). There are no arrival toasts, welcome banners, level-up confetti, or streak popups.
3. **Charm comes from specificity**: All system prose on the product surface (notebook entries, captions, narration) uses a naturalist field-notebook register (lowercase, present-tense, specific, bird-centric verbs).
4. **Restraint over richness**: Exactly one horizontal scene, no scrolling, no panning, no zooming, no clutter. Two starter birds up to a strict ceiling of seven, preserving individual auditory recognizability.
5. **Naturalist voice for the product, matter-of-fact for the system**: Product surfaces use the naturalist register; auth, settings, error, and sync conflict surfaces use clear, standard matter-of-fact English without faux-warmth.

### 1.2 In-Scope for Version 1
- **Platform**: Modern web browsers (last 2 major versions of Chrome, Safari, Firefox, Edge). Responsive across desktop, tablet, and mobile viewports.
- **Authentication & Accounts**: Single-user accounts via email magic links (15-minute expiration, single-use, rate-limited). Synthetic UUID internal keys (email encrypted at rest). Revocable per-device sessions. 30-day soft deletion with self-recovery, followed by hard purge. JSON snapshot export.
- **Aviary & Bird Mechanics**:
  - Single canonical aviary per account.
  - Flock size: 2 starter birds at creation; progression up to 7 birds strictly gated by aviary age (calendar tenure), never by click count, visit count, or currency.
  - 6 initial bird species in the pool, each with distinct visual silhouettes, feather palettes, and call-grammar motif libraries.
  - Stable bird UUID identity that persists through renaming, sync, and migrations.
  - Hidden 5-dimensional personality vector per bird (boldness, social warmth, vocal frequency, plumage saturation, curiosity).
  - Fast-timescale mood system (wary, content, curious, drowsy, alert) tied to local diurnal cycles, weather, recent interactions, and flock dynamics.
  - Monotonic, asymmetric personality drift: slow low-pass filter over presence and interactions; values drift toward expressive; zero decay or punishment on neglect.
- **Session Interactions**:
  - **Return-Greeting**: Naturalist arrival recognition within 1–2 seconds, scaled by absence length and bird boldness/mood. Procedurally staggered, strictly no textual welcome banners or toasts.
  - **Listen-In**: Smooth audio rebalance focusing a single bird (+gain) while dimming other birds (-gain to ambient floor, never muted). Gradual ramp-up and ramp-down.
  - **Offers**: Top-bar gestural offers (seed, song fragment, still pool of water). Per-bird cooldowns of several minutes to prevent mechanic spamming.
  - **Settle Gesture**: Opt-in gentle session-end gesture triggering a 5-second dusk transition and call dampening, with a 5-second click-to-cancel grace window. Closing the tab without settling is functionally identical at the engine level.
  - **Field Notebook**: Auto-generated, read-only naturalist observation log generated every few days of regular observation. Sparse, poetic, and persistent.
- **Presence Accounting**: Conjunction of three simultaneous signals: `document.visibilityState === 'visible'`, window focus (`document.hasFocus()`), and recent user pointer/keyboard input within a 3-minute sliding window.
- **Social (Quiet & Optional)**:
  - Host-initiated email invitation generating a unique 30-day single-use/revocable link.
  - Read-only ambient guest view. Zero co-presence, no shared cursors, no interaction permissions, no visitor drift impact.
  - Host visit log in settings. Friend-visited notifications default to OFF.
- **Accessibility & Performance Budgets**:
  - Full screen-reader naturalist running prose narration (`aria-live="polite"`, 30–60s cadence, priority bumps on interactions).
  - Designed reduced-motion mode (cross-faded still poses, static camera, disabled leaf/feather particles).
  - Real-time procedural call captions anchored near vocalizing birds.
  - Full keyboard navigation and visible focus rings meeting WCAG AA.
  - Initial JS bundle <= 2.0 MB (gzipped). Time to first bird visible < 500 ms over 4G mobile. 60 fps idle motion on 5-year-old hardware. Zero memory growth over 30 minutes. WebAudio fallback to graceful silence with captions enabled by default.

### 1.3 Out-of-Scope (Explicit Non-Goals)
The engineering team must not implement, stub, or architect hooks for:
- **Native Mobile/Desktop Apps**: No iOS, Android, Electron, or Tauri packages. Pure web standards only.
- **Gamification Mechanics**: Strictly no streaks, daily visit counters, green-dot activity calendars, XP, levels, badges, achievements, quest logs, or leaderboards.
- **Tamagotchi / Custodial Systems**: Birds never starve, sicken, or die. No happiness decay meters, hunger gauges, or cleaning tasks. Neglect produces quiet ambient behavior, never distress or guilt.
- **Social Network Features**: No public directory, no aviary discovery feed, no user profiles, no friend lists, no follows, no visitor comments/guestbook entries, no co-presence or live shared cursors.
- **Push / External Notifications**: No web push, no SMS, no automated marketing or re-engagement emails. Pocket Aviary never reaches out to pull the user back.
- **Paid Upgrades / Microtransactions**: No cosmetic store, bird purchases, or speed-ups.

---

## 2. System Architecture & Topology

```
+------------------------------------------------------------------------------------+
|                                 CLIENT TIER                                        |
|  +------------------------------------------------------------------------------+  |
|  | Modern Web Browser (Chrome, Safari, Firefox, Edge)                           |  |
|  |                                                                              |  |
|  |  +------------------------+  +----------------------+  +------------------+  |  |
|  |  | Canvas2D/WebGL Scene   |  | WebAudio Procedural  |  | Accessible DOM   |  |  |
|  |  | - 3 Perch Zones        |  | Synthesis Engine     |  | - ARIA live      |  |  |
|  |  | - Idle Micro-Motion    |  | - Motif Generator    |  | - Focus Anchors  |  |  |
|  |  | - Parallax & Weather   |  | - Chorus Mixer       |  | - Captions       |  |  |
|  |  +------------------------+  +----------------------+  +------------------+  |  |
|  |              ^                          ^                        ^           |  |
|  |              |                          |                        |           |  |
|  |  +------------------------------------------------------------------------+  |  |
|  |  | Client State Coordinator & Presence Monitor                            |  |  |
|  |  | - 3-Way Presence Conjunction Engine                                    |  |  |
|  |  | - Snapshot Interpolator                                                |  |  |
|  |  | - Event Batcher & Dispatcher                                           |  |  |
|  |  +------------------------------------------------------------------------+  |  |
|  +------------------------------------------------------------------------------+  |
+------------------------------------------+-----------------------------------------+
                                           | HTTPS / WSS
                                           v
+------------------------------------------------------------------------------------+
|                                 EDGE & ROUTING TIER                                |
|  +------------------------------------------------------------------------------+  |
|  | Edge CDN (Cloudflare / Fastly)                                               |  |
|  | - Static Assets Caching (<2MB Bundle Target)                                 |  |
|  | - Edge HTML SSR / Snapshot Injection for TTFB & TTFBird < 500ms              |  |
|  +------------------------------------------------------------------------------+  |
+------------------------------------------+-----------------------------------------+
                                           | Reverse Proxy
                                           v
+------------------------------------------------------------------------------------+
|                               APPLICATION SERVICES                                 |
|  +-----------------------------------+   +--------------------------------------+  |
|  | Gateway & Session Service         |   | Aviary Ingestion Service             |  |
|  | - Magic Link Auth & Token Verif   |   | - Pull Snapshot Endpoint             |  |
|  | - Rate Limiting & Session Admin   |   | - Append Interaction Events Log      |  |
|  | - Social Guest Token Validation   |   | - Host Visit Log Tracker             |  |
|  +-----------------+-----------------+   +------------------+-------------------+  |
|                    |                                        |                      |
|                    +--------------------+-------------------+                      |
|                                         |                                          |
|                                         v                                          |
|  +------------------------------------------------------------------------------+  |
|  | Simulation Worker Service (The Server Tick Engine)                           |  |
|  | - Periodic Aviary Ticks (~60s cadence)                                       |  |
|  | - Asymmetric Drift Low-Pass Filter Evaluation                                |  |
|  | - Diurnal Cycle, Weather & Mood State Machine Transitions                    |  |
|  | - Naturalist Field Notebook Generator                                        |  |
|  | - Screen-Reader Running Prose Generator                                      |  |
|  +--------------------------------------+---------------------------------------+  |
+-----------------------------------------|------------------------------------------+
                                          v
+------------------------------------------------------------------------------------+
|                                DATA STORAGE TIER                                   |
|  +-----------------------------------+   +--------------------------------------+  |
|  | PostgreSQL (Canonical ACID Store) |   | Redis (Low-Latency State Cache)      |  |
|  | - Accounts (UUIDs, Encrypted PII) |   | - Active Session Heartbeats          |  |
|  | - Aviaries, Birds, Vectors, Moods |   | - Latest Canonical Aviary Snapshots  |  |
|  | - Append-Only Interaction Log     |   | - Magic Link Tokens & Rate Limits    |  |
|  | - Field Notebook & Visit Records  |   | - Tick Work Queue Distributed Locks  |  |
|  +-----------------------------------+   +--------------------------------------+  |
+------------------------------------------------------------------------------------+
```

### 2.1 Component Responsibilities & Boundaries
- **Edge Layer**: Serves immutable application bundles and executes a lightweight worker that fetches the user's current aviary snapshot and inlines it directly into the initial `index.html` response. This eliminates client-side round-trip waterfall, satisfying the strict Time to First Bird (<500ms) requirement.
- **Gateway & Session Service**: Manages passwordless authentication via magic links. Issues signed, HTTP-only, secure session cookies mapped to synthetic account UUIDs. Strictly isolates PII (email addresses are encrypted with AES-256-GCM and stored only in the accounts table).
- **Aviary Ingestion Service**: Exposes REST endpoints for client snapshot polling and interaction event submissions. Appends events directly into an append-only event log. Emits no direct personality mutations.
- **Simulation Worker Service**: Background engine running the canonical simulation tick (~60-second cycle per active aviary). Processes batches of append-only events, advances diurnal lighting and weather, executes the drift filter, triggers bird-to-bird chorus events, generates notebook entries, and commits new canonical state snapshots.
- **Data Stores**:
  - PostgreSQL: Single source of truth. Handles transactions with strict isolation to prevent race conditions.
  - Redis: Ephemeral operational cache for cached serialized snapshots, distributed lease locks for simulation workers (`SET aviary:lock:<uuid> NX EX 70`), and rate limiting.

### 2.2 Client/Server Boundary Guarantees
- **Single Writer Principle**: Only the server simulation tick ever updates the canonical personality vectors and moods.
- **Event-Only Ingestion**: Clients submit only interaction gestures (heartbeats, offers, listen-in timestamps, settle triggers). Clients have zero write access to trait values.
- **Render-Only Guests**: Visitors in social sessions pull read-only snapshots and have their interaction endpoints completely disabled.

---

## 3. Data Model & Database Schema

All database entities strictly enforce synthetic UUID primary keys. Email addresses are stored in exactly one column in the accounts table, encrypted using AES-256-GCM with a tenant-isolated encryption key. Telemetry and foreign keys never touch PII.

```sql
-- Core Account Schema
CREATE TABLE accounts (
    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    encrypted_email BYTEA NOT NULL,
    email_bidx VARCHAR(64) NOT NULL UNIQUE, -- Blind index (HMAC-SHA256) for lookup
    status VARCHAR(24) NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'soft_deleted')),
    soft_deleted_at TIMESTAMPTZ NULL,
    export_requested_at TIMESTAMPTZ NULL,
    settings JSONB NOT NULL DEFAULT '{
        "prefers_reduced_motion": false,
        "captions_enabled": false,
        "notify_on_visit": false
    }'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

CREATE INDEX idx_accounts_status ON accounts(status) WHERE status = 'soft_deleted';

-- Session Management
CREATE TABLE sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    device_label VARCHAR(64) NOT NULL,
    user_agent TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    revoked_at TIMESTAMPTZ NULL
);

CREATE INDEX idx_sessions_account_active ON sessions(account_id) WHERE revoked_at IS NULL;

-- Magic Links
CREATE TABLE magic_links (
    token_hash VARCHAR(64) PRIMARY KEY, -- SHA-256 of raw token
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    expires_at TIMESTAMPTZ NOT NULL,
    used_at TIMESTAMPTZ NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

-- Aviary Scene State
CREATE TABLE aviaries (
    aviary_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(account_id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    lighting_state JSONB NOT NULL DEFAULT '{"phase": "midday", "warmth": 0.5, "brightness": 1.0}'::jsonb,
    weather_state JSONB NOT NULL DEFAULT '{"current": "clear", "intensity": 0.0, "expires_at": null}'::jsonb,
    settled_state JSONB NOT NULL DEFAULT '{"is_settled": false, "settled_at": null}'::jsonb,
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    last_presence_at TIMESTAMPTZ NULL,
    cumulative_presence_seconds BIGINT NOT NULL DEFAULT 0
);

-- Birds
CREATE TABLE birds (
    bird_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL CHECK (species_id IN ('sparrow', 'wren', 'warbler', 'finch', 'chickadee', 'nightjar')),
    name VARCHAR(48) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    perch_zone VARCHAR(16) NOT NULL DEFAULT 'middle' CHECK (perch_zone IN ('front', 'middle', 'back')),
    perch_x_normalized REAL NOT NULL DEFAULT 0.5 CHECK (perch_x_normalized >= 0.0 AND perch_x_normalized <= 1.0),
    current_mood VARCHAR(16) NOT NULL DEFAULT 'content' CHECK (current_mood IN ('wary', 'content', 'curious', 'drowsy', 'alert')),
    mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    last_call_at TIMESTAMPTZ NULL,
    last_greeting_at TIMESTAMPTZ NULL
);

CREATE INDEX idx_birds_aviary ON birds(aviary_id);

-- Personality Vectors (Hidden, Server-Only Writes)
CREATE TABLE personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(bird_id) ON DELETE CASCADE,
    boldness REAL NOT NULL DEFAULT 0.5 CHECK (boldness >= 0.0 AND boldness <= 1.0),
    social_warmth REAL NOT NULL DEFAULT 0.5 CHECK (social_warmth >= 0.0 AND social_warmth <= 1.0),
    vocal_frequency REAL NOT NULL DEFAULT 0.5 CHECK (vocal_frequency >= 0.0 AND vocal_frequency <= 1.0),
    plumage_saturation REAL NOT NULL DEFAULT 0.5 CHECK (plumage_saturation >= 0.0 AND plumage_saturation <= 1.0),
    curiosity REAL NOT NULL DEFAULT 0.5 CHECK (curiosity >= 0.0 AND curiosity <= 1.0),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

-- Append-Only Interaction Event Log
CREATE TABLE interaction_event_log (
    event_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(bird_id) ON DELETE SET NULL,
    event_type VARCHAR(32) NOT NULL CHECK (event_type IN (
        'presence_heartbeat', 'listen_in_start', 'listen_in_end',
        'offer_seed', 'offer_song', 'offer_pool',
        'settle_trigger', 'settle_cancel'
    )),
    payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    client_timestamp TIMESTAMPTZ NOT NULL,
    server_timestamp TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    processed_at TIMESTAMPTZ NULL
);

CREATE INDEX idx_event_log_unprocessed ON interaction_event_log(aviary_id, server_timestamp) WHERE processed_at IS NULL;

-- Field Notebook Observations
CREATE TABLE notebook_entries (
    entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    focal_bird_id UUID NULL REFERENCES birds(bird_id) ON DELETE SET NULL,
    observation_prose TEXT NOT NULL,
    weather_tag VARCHAR(32) NOT NULL DEFAULT 'clear'
);

CREATE INDEX idx_notebook_aviary ON notebook_entries(aviary_id, created_at DESC);

-- Social Visits
CREATE TABLE visit_invites (
    invite_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    visitor_email_bidx VARCHAR(64) NOT NULL,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ NULL
);

CREATE TABLE visit_logs (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    invite_id UUID NOT NULL REFERENCES visit_invites(invite_id) ON DELETE CASCADE,
    visitor_email_bidx VARCHAR(64) NOT NULL,
    started_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    ended_at TIMESTAMPTZ NULL,
    duration_seconds INT NOT NULL DEFAULT 0
);
```

---

## 4. API Surface & Contract Specifications

### 4.1 Authentication & Account Management
All endpoints return standard HTTP status codes and JSON envelopes. Error responses on system endpoints strictly adhere to the matter-of-fact tone.

#### `POST /api/v1/auth/magic-link`
Requests an email magic link. Rate-limited to 3 requests per 15 minutes per IP/email.
- **Request Body**:
  ```json
  { "email": "user@example.com" }
  ```
- **Response (200 OK)**:
  ```json
  { "status": "sent", "message": "Check your email for your sign-in link." }
  ```
- **Response (429 Too Many Requests)**:
  ```json
  { "status": "error", "message": "Too many requests. Please wait a few minutes before trying again." }
  ```

#### `GET /api/v1/auth/verify?token=...`
Consumes the single-use token within its 15-minute validity window.
- **Response (302 Found)**: Sets HttpOnly, Secure, SameSite=Lax cookie `pa_session=<session_id>` and redirects to `/`.
- **Response (400 Bad Request / 410 Gone)**:
  ```json
  { "status": "error", "message": "We couldn't sign you in. The link may have expired or already been used. Try requesting a new link." }
  ```

#### `POST /api/v1/account/export`
Triggers on-demand generation of the user's JSON aviary archive.
- **Response (202 Accepted)**:
  ```json
  { "status": "accepted", "message": "Your snapshot export is being prepared and will be sent to your email." }
  ```

#### `POST /api/v1/account/delete`
Initiates the 30-day soft deletion grace period.
- **Response (200 OK)**:
  ```json
  { "status": "soft_deleted", "hard_delete_scheduled_at": "2026-10-06T07:17:21Z", "message": "Your account has been scheduled for deletion. You can sign in within 30 days to cancel this request." }
  ```

### 4.2 Aviary State & Interaction Endpoints

#### `GET /api/v1/aviary/snapshot`
Fetches the current canonical aviary state for rendering and audio scheduling.
- **Headers**: `Cookie: pa_session=<session_id>`
- **Response (200 OK)**:
  ```json
  {
    "aviary_id": "8fa3c072-1b1e-4c7a-9a99-826d7e01b7a2",
    "server_time": "2026-09-06T07:17:21Z",
    "sequence_id": 142857,
    "diurnal": {
      "phase": "morning",
      "solar_progress": 0.22,
      "warmth": 0.65,
      "brightness": 0.85
    },
    "weather": {
      "current": "clear",
      "intensity": 0.0
    },
    "settled": {
      "is_settled": false
    },
    "absence_duration_seconds": 28400,
    "first_greeter_bird_id": "b1b01c34-8c88-4c3e-9021-39512c5bbf12",
    "birds": [
      {
        "bird_id": "b1b01c34-8c88-4c3e-9021-39512c5bbf12",
        "name": "pip",
        "species_id": "warbler",
        "perch_zone": "front",
        "perch_x": 0.32,
        "mood": "content",
        "visual": {
          "plumage_saturation_factor": 0.78
        },
        "audio": {
          "vocal_cadence_base_seconds": 18.5,
          "motif_grammar_seed": 491823
        }
      },
      {
        "bird_id": "c2b09a45-9d99-4d4f-a132-40623d6ccf23",
        "name": "wren",
        "species_id": "wren",
        "perch_zone": "middle",
        "perch_x": 0.68,
        "mood": "drowsy",
        "visual": {
          "plumage_saturation_factor": 0.55
        },
        "audio": {
          "vocal_cadence_base_seconds": 32.0,
          "motif_grammar_seed": 847291
        }
      }
    ],
    "screen_reader_narration": "pip is on the front perch in the quiet morning light, calling softly. wren sits further back on the middle branch, feathers fluffed against the cool air."
  }
  ```

#### `POST /api/v1/aviary/events`
Batch submits client-side interactions and presence heartbeats.
- **Request Body**:
  ```json
  {
    "events": [
      {
        "event_id": "e812d1b4-2b7e-46bb-88b1-9f939e6a9284",
        "event_type": "presence_heartbeat",
        "client_timestamp": "2026-09-06T07:17:00Z",
        "payload": { "duration_seconds": 30 }
      },
      {
        "event_id": "f923e2c5-3c8f-47cc-99c2-00a40f7ba395",
        "event_type": "listen_in_start",
        "bird_id": "b1b01c34-8c88-4c3e-9021-39512c5bbf12",
        "client_timestamp": "2026-09-06T07:17:15Z",
        "payload": {}
      }
    ]
  }
  ```
- **Response (200 OK)**:
  ```json
  { "status": "acknowledged", "processed_count": 2 }
  ```

### 4.3 Social Visit API & Controls

#### `POST /api/v1/visits/invite`
Creates a private, revocable guest invitation.
- **Request Body**:
  ```json
  { "visitor_email": "friend@example.com" }
  ```
- **Response (201 Created)**:
  ```json
  { "invite_id": "d045f3d6-4d9a-48dd-a0d3-11b51a8ca406", "expires_at": "2026-10-06T07:17:21Z", "status": "sent" }
  ```

#### `POST /api/v1/visits/revoke`
Revokes an outstanding or active visit token immediately.
- **Request Body**:
  ```json
  { "invite_id": "d045f3d6-4d9a-48dd-a0d3-11b51a8ca406" }
  ```
- **Response (200 OK)**:
  ```json
  { "status": "revoked" }
  ```

#### `GET /api/v1/visits/guest-snapshot?token=...`
Read-only ambient snapshot for invited visitors.
- **Rules**:
  - Validates invite token. If revoked or expired, returns 403 Forbidden with matter-of-fact copy: `"This visit link is no longer available. Check with your friend to request a new link."`
  - Disables presence recording and ignores any interaction submission attempts.
  - Logs session start and duration in `visit_logs`.

---

## 5. Simulation Engine Design

### 5.1 Server-Side Simulation Tick Architecture
The canonical state of every aviary advances via a deterministic server-side tick running at a target cadence of 60 seconds.

```
                  +-----------------------------------+
                  |  Tick Scheduler (Worker Pool)     |
                  +-----------------+-----------------+
                                    |
                                    v
                  +-----------------------------------+
                  | Acquire Aviary Distributed Lock   |
                  | (Redis: SET aviary:lock:<id> NX)  |
                  +-----------------+-----------------+
                                    |
                                    v
                  +-----------------------------------+
                  | Pull Unprocessed Event Log Batch  |
                  | (interaction_event_log table)     |
                  +-----------------+-----------------+
                                    |
            +-----------------------+-----------------------+
            |                       |                       |
            v                       v                       v
+-----------------------+ +-------------------+ +-----------------------+
| 1. Presence & Drift   | | 2. Diurnal/Weather| | 3. Mood Transitions   |
| - Filter heartbeats   | | - Update sun angle| | - Event stimuli       |
| - Apply asymmetric    | | - Step rain/wind  | | - Diurnal bias        |
|   low-pass filter     | | - Check settle    | | - Social contagion    |
+-----------+-----------+ +---------+---------+ +-----------+-----------+
            |                       |                       |
            +-----------------------+-----------------------+
                                    |
                                    v
                  +-----------------------------------+
                  | 4. Observation & Narration Engine |
                  | - Evaluate notebook criteria      |
                  | - Compose naturalist prose        |
                  +-----------------+-----------------+
                                    |
                                    v
                  +-----------------------------------+
                  | 5. Commit Canonical State         |
                  | - Write PostgreSQL in transaction |
                  | - Invalidate Redis snapshot cache |
                  | - Release distributed lock        |
                  +-----------------+-----------------+
```

### 5.2 Mathematical Formulation of Personality Drift
Drift is modelled as an asymmetric, monotonic low-pass filter (leaky integrator with zero downward leakage). Personality values are strictly bounded scalars T_i in [0.0, 1.0].

Let T_i(t) be the scalar value of trait i at tick t. The update rule is:
  delta T_i(t) = eta * w_i * S_i(t) * (1.0 - T_i(t))
  T_i(t + 1) = T_i(t) + max(0.0, delta T_i(t))

Where:
- eta is the fundamental base learning rate calibrated to temporal targets (eta = 1.6e-5 per valid presence minute).
- w_i is the specific trait sensitivity weight:
  - Boldness: w_presence = 1.0, w_offer_near = 0.8.
  - Social Warmth: w_listen = 1.8, w_chorus = 1.2.
  - Vocal Frequency: w_listen = 1.5, w_presence = 0.6.
  - Plumage Saturation: w_presence = 1.2 (strictly dependent on cumulative calm presence).
  - Curiosity: w_offer_accept = 2.0.
- S_i(t) in [0.0, 1.0] is the normalized stimulus aggregate during tick window t.
- (1.0 - T_i(t)) provides natural diminishing returns as traits approach maximum expression.
- max(0.0, delta T_i(t)) guarantees the monotonic invariant: traits never decrease on user absence or neglect.

#### Calibration Verification Target:
- For a typical user engaging 15 minutes/day:
  - At 7 days (~105 presence minutes): delta T_i ~ 0.015 - 0.025. Readily detected by automated regression test suites; imperceptible to casual visual inspection.
  - At 21 days (~315 presence minutes): delta T_i ~ 0.06 - 0.09. Substantively shifts behavior (Pip perches in the front zone 35% more often, plumage shows visibly richer saturation).
  - At 60 days: Traits settle into high expressive stability (T_i ~ 0.75 - 0.85).

### 5.3 Mood State Machine
Mood is a fast-timescale 5-state discrete Markov chain: `wary`, `content`, `curious`, `drowsy`, `alert`.
- **State Transition Matrix M_jk = P(S_{t+1} = k | S_t = j)**:
  - Influenced by diurnal cycle: Dusk and night strongly bias towards `drowsy` (P -> 0.75). Dawn biases towards `alert` (P -> 0.70).
  - Influenced by interactions:
    - Successful offer acceptance triggers transition to `content` (P = 0.90) or `curious` (P = 0.10).
    - Abrupt return after long absence triggers `wary` if bird boldness < 0.4, but `alert` if boldness >= 0.6.
  - Social contagion: If a nearby bird enters `wary` (e.g. from an unaccepted offer or sudden sound), neighboring birds with social warmth > 0.5 roll a 35% probability to adopt `wary`.

### 5.4 Procedural Call-Grammar Runtime
Each species defines a formal generative grammar of acoustic motifs:
- Motif Types:
  - Whistle (pure sinusoidal tone with gentle pitch inflection).
  - Trill (rapid frequency modulation: 15–30 Hz vibrato over a carrier).
  - Chirp (steep linear or exponential pitch slide up or down: delta f = 400 - 1200 Hz in 40–80 ms).
  - Slur (concave or convex parabolic frequency trajectory).
- Grammar Production Rules:
  - Call -> Prefix [Inflection] Suffix
  - Prefix -> ShortChirp | SoftWhistle
  - Inflection -> Trill | DoubleChirp | SilentPause(100–300ms)
  - Suffix -> SustainedWhistle | DownSlur
- Variation Generator:
  - Micro-timing variance: Note onsets jittered by Gaussian noise N(0, 15ms).
  - Micro-pitch variance: Base fundamental frequency f_0 randomized by N(0, 8 cents).
  - Personality modulation: A bird with high boldness uses higher base pitch and faster tempo; high social warmth increases the probability of choosing harmonic overtones.

---

## 6. Multi-Device Sync & Concurrency Model

### 6.1 Server-Authoritative Architecture
The architecture operates strictly under the **Single Writer Principle**:
- Clients are stateless renderers and event dispatchers.
- No client-to-client peer synchronization (no WebRTC mesh, no local CRDTs).
- Multiple simultaneous browser sessions (e.g. laptop at desk, phone on side table) both pull from the single canonical state snapshot.

### 6.2 Conflict-Free Event Processing (No Last-Write-Wins)
- Interaction events are strictly additive. Even if two devices submit presence heartbeats concurrently, the ingestion engine logs both with unique UUIDs.
- During the simulation tick, the presence engine evaluates overlapping timestamps across devices and computes the **union** of active presence intervals. Simultaneous presence across two devices for 1 minute counts as exactly 1 minute of presence time, preventing artificial drift acceleration.
- Because clients never transmit absolute state (e.g., "bird_position = X"), race conditions and last-write-wins data loss are architecturally eliminated.

### 6.3 State Synchronization Lifecycle
1. **Initial Mount**: Client executes `GET /api/v1/aviary/snapshot` (or reads edge-injected snapshot).
2. **Smooth Interpolation**: When a snapshot arrives, the client compares the bird's current visual coordinates (x_0, y_0) with the new target coordinates (x_1, y_1). If the zone changed, the client initiates an organic hopping or flight path over 1.2–2.5 seconds using cubic bezier curves, avoiding visual snaps.
3. **Visibility & Sleep Resync**:
   - Client listens to `document.visibilitychange`. When `document.visibilityState === 'visible'`, an immediate snapshot pull is triggered.
   - Client runs a monotonic timer watchdog. If a render-frame delta exceeds 5 seconds (indicating OS laptop sleep or background throttling), the client discards local interpolation buffers and immediately fetches a fresh snapshot.

---

## 7. Frontend Rendering & Micro-Motion Pipeline

```
+------------------------------------------------------------------------------------+
|                               RENDER LOOP (requestAnimationFrame)                  |
+------------------------------------------------------------------------------------+
  |
  +--> 1. Check Preferences & State
  |      - prefers-reduced-motion?
  |      - Document visibility? (if hidden, pause rAF loop completely to save battery)
  |
  +--> 2. Standard Mode Pipeline (60 fps Canvas2D/WebGL)
  |      +-- Layer 0: Sky Gradient & Diurnal Sun/Moon (Color matrix lerp)
  |      +-- Layer 1: Distant Canopy Parallax (Mouse offset * 0.02)
  |      +-- Layer 2: Back Perch Zone (Z=0.4, scale 0.55, birds rendered with cool tint)
  |      +-- Layer 3: Mid Canopy & Middle Perch Zone (Z=0.7, scale 0.75)
  |      +-- Layer 4: Ambient Particles (Leaves & feathers drifting with Perlin noise)
  |      +-- Layer 5: Front Perch Zone (Z=1.0, scale 1.0, high plumage detail)
  |      +-- Layer 6: Procedural Micro-Motion Evaluation:
  |      |     - Breathing: Y-scale sin(t * 1.8) * 0.015
  |      |     - Head Tilt: Poisson-distributed discrete angle shifts (tau ~ 4.2s)
  |      |     - Preening: Cyclical feather sweeps triggered during 'content' mood
  |      |     - Weight Shift: Left/right foot balance adjustment (tau ~ 6.5s)
  |      +-- Layer 7: Interactive Hover/Focus Highlights (Soft, high-contrast SVG glow)
  |
  +--> 3. Reduced-Motion Mode Pipeline (Discrete Cross-Fade Engine)
         +-- Disable continuous frame loops; throttle rendering to pose changes
         +-- Disable all ambient leaf/feather particles and parallax offsets
         +-- Static Poses: Pre-rendered SVGs/bitmaps for perch, tilt, preen
         +-- State transitions execute via a 1.2-second smooth alpha cross-fade
```

### 7.1 Viewport Responsiveness & Coordinate System
- Virtual Coordinate Space: Canonical canvas coordinates are defined on a virtual 1920 x 1080 aspect-ratio grid.
- Responsive Clamping: On narrower viewports (mobile portrait or narrow windows), the scene preserves vertical scale while dynamically compressing the horizontal distance between perch anchors.
- Invariant Rule: Birds are guaranteed to remain within the visible viewport bounds (x in [0.10, 0.90]) at all times; no bird is ever cropped or placed offscreen.

### 7.2 The "Already Alive" First Frame Mandate
To satisfy the design principle that the aviary has been continuing without the viewer:
1. The renderer initializes its micro-motion time variable directly from the absolute clock: `t = (Date.now() % 100000) / 1000`.
2. When the snapshot loads, birds are placed into procedural cycle phases computed deterministically from `(bird_seed ^ floor(Date.now() / cycle_period))`.
3. The very first frame presented to the user has birds already mid-action (mid-preen, head tilted, ambient leaves already positioned mid-screen).
4. Stock loading spinners and "fade-from-blank" transitions are strictly forbidden. If a cold network fetch requires a pause, a calm, ambient sky background with faint drifting mist is rendered until birds mount.

---

## 8. Audio Pipeline & Procedural Call Synthesis

### 8.1 WebAudio Graph Architecture

```
+------------------------------------------------------------------------------------+
|                         PER-BIRD PROCEDURAL SYNTHESIZER GRAPH                      |
+------------------------------------------------------------------------------------+
|                                                                                    |
|  +------------------------+      +-----------------------+                         |
|  | Base Oscillator (Sine) |      | Modulator (FM Sine)   |                         |
|  +-----------+------------+      +-----------+-----------+                         |
|              |                               |                                     |
|              |                               v                                     |
|              +-----------------------> Frequency Node                              |
|                                              |                                     |
|  +------------------------+                  v                                     |
|  | Noise Node (Bandpass)  |---------> Gain Envelope (ADSR)                         |
|  +------------------------+                  |                                     |
|                                              v                                     |
|                                      Biquad Filter Node                            |
|                                      (Distance Lowpass)                            |
|                                              |                                     |
|                                              v                                     |
|                                       PannerNode 2D                                |
|                                   (Stereo based on Perch X)                        |
|                                              |                                     |
|                                              v                                     |
|                                     Bird Specific Gain Node                        |
|                                    (Listen-In Dynamic Mix)                         |
|                                              |                                     |
+----------------------------------------------|-------------------------------------+
                                               v
+------------------------------------------------------------------------------------+
|                             MASTER AUDIO BUS & OUTPUT                              |
+------------------------------------------------------------------------------------+
|                                              |                                     |
|                                              v                                     |
|                                DynamicsCompressorNode                              |
|                               (Prevents Chorus Peaking)                            |
|                                              |                                     |
|                                              v                                     |
|                                       Master Gain Node                             |
|                                              |                                     |
|                                              v                                     |
|                                   AudioContext.destination                         |
+------------------------------------------------------------------------------------+
```

### 8.2 Listen-In Mix Dynamics
The listen-in interaction refocuses the acoustic field without unnatural cuts:
- **Nominal Mix**: Every bird operates at a default base gain G_base = 0.60.
- **Engaging Listen-In on Bird K**:
  - Bird K gain smoothly ramps up to G_focus = 1.0 over 1500ms using `exponentialRampToValueAtTime`.
  - All other birds (j != K) ramp down to an ambient floor G_ambient = 0.15 over 1500ms.
  - Non-focused birds are never muted to 0.0; the chorus remains gently audible in the background.
- **Disengaging Listen-In**:
  - All birds smoothly ramp back to G_base = 0.60 over 2000ms.

### 8.3 WebAudio Fallback & Autoplay Resilience
- **Autoplay Handling**: AudioContext is instantiated in `suspended` state. A subtle, natural top-bar affordance or the first user interaction seamlessly invokes `audioContext.resume()`.
- **Absolute Non-Goal: No Pre-Recorded Audio**:
  - If WebAudio fails completely or hardware is unavailable, the aviary plays in **graceful silence**.
  - Call captions are automatically enabled and anchored visually near calling birds.
  - Pre-recorded fallback audio clips are strictly prohibited by PRD mandate.

---

## 9. Accessibility Surfaces

```
+------------------------------------------------------------------------------------+
|                       ACCESSIBILITY ARCHITECTURE & DOM TREE                        |
+------------------------------------------------------------------------------------+
|                                                                                    |
|  <main id="aviary-container" role="region" aria-label="Pocket Aviary">             |
|                                                                                    |
|    <!-- 1. Real-Time Screen-Reader Narration Region -->                            |
|    <div id="sr-narration"                                                          |
|         class="sr-only"                                                            |
|         aria-live="polite"                                                         |
|         aria-atomic="true">                                                        |
|         pip is perched on the front branch, calling softly in the morning light.   |
|    </div>                                                                          |
|                                                                                    |
|    <!-- 2. Semantic Focus Anchors Overlaying Canvas -->                            |
|    <div id="accessible-bird-targets">                                              |
|      <button class="bird-focus-anchor"                                             |
|              aria-label="pip, warbler on front perch. mood is content."            |
|              style="top: 60%; left: 32%; width: 80px; height: 80px;"              |
|              tabindex="0">                                                         |
|      </button>                                                                     |
|      <button class="bird-focus-anchor"                                             |
|              aria-label="wren, wren on middle perch. mood is drowsy."              |
|              style="top: 45%; left: 68%; width: 80px; height: 80px;"              |
|              tabindex="0">                                                         |
|      </button>                                                                     |
|    </div>                                                                          |
|                                                                                    |
|    <!-- 3. Canvas Visual Layer -->                                                 |
|    <canvas id="aviary-canvas" aria-hidden="true"></canvas>                         |
|                                                                                    |
|    <!-- 4. Visual Call Captions Layer -->                                          |
|    <div id="call-captions-overlay" aria-hidden="true">                             |
|      <span class="caption-bubble" style="top: 55%; left: 34%;">                    |
|        a soft three-note rise                                                      |
|      </span>                                                                       |
|    </div>                                                                          |
|                                                                                    |
|  </main>                                                                           |
+------------------------------------------------------------------------------------+
```

### 9.1 Screen-Reader Narration Engine
- An `aria-live="polite"` container receives naturalist prose generated on a slow cadence (30–60 seconds).
- Prose is composed using deterministic templates combining diurnal light, weather, and bird actions:
  - Example: `"pip is on the low perch this morning, fluffed against the cool air. a leaf drifted down past the back branch."`.
- Priority Bumps: Immediate updates are dispatched on discrete user gestures (an accepted offer or settle).
- Hard Rule: Never announce raw system state (no `"bird position: 2"` or `"mood: content"`).

### 9.2 Call Captioning
- Opt-in via settings or automatically activated when audio is silent/unavailable.
- Generated dynamically from the procedural call parameters at runtime:
  - Rising tone: `"a soft three-note rise"`
  - FM trill: `"a low trill, paused, low trill again"`
  - Sharp call: `"a single sharp call from the back perch"`
- Rendered in a high-contrast, translucent callout that fades in and out with audio duration.

### 9.3 Keyboard Navigation & Focus Ring Standards
- Key Bindings:
  - `Tab` / `Shift+Tab`: Cycles through top-bar controls and into aviary bird targets.
  - `ArrowLeft` / `ArrowRight`: Moves focus sequentially between birds based on horizontal screen position.
  - `Enter` / `Space`: Activates **Listen-In** on the focused bird.
  - `Escape`: Deactivates Listen-In.
  - `O`: Opens Offer selection tray in top bar.
  - `S`: Triggers Settle gesture.
- Focus Indicator: An SVG double-outline (2px solid black inner, 2px solid white outer) ensuring a contrast ratio > 7:1 across all lighting phases.

---

## 10. Performance Budgets, Verification & Telemetry

### 10.1 Hard Performance Budgets

| Metric | Budget Target | Verification Method | Enforcement Point |
|---|---|---|---|
| **Initial JS Bundle** | < 2.0 MB gzipped (Target < 1.2 MB) | Webpack/Vite bundle analyzer in CI | PR build failure if bundle > 2.0 MB |
| **Time to First Bird (TTFBird)** | < 500 ms on 4G Mobile | Synthetic Lighthouse check on Moto G4 emulation | Release block on synthetic regression |
| **Idle Render FPS** | 60 fps on 5-year-old laptop | Automated Puppeteer trace measuring frame deltas | CI benchmark run measuring 99th percentile frame times |
| **Memory Growth (Leak Check)** | 0.00 MB net growth over 30 min | 30-minute automated headless heap snapshot diffing | Nightly CI memory leak audit |
| **Simulation Tick Latency** | p99 < 5.0 seconds | Prometheus timer on worker execution batch | PagerDuty alert on p99 > 5.0s for 3 consecutive minutes |

### 10.2 Memory Leak Prevention Strategies
- **WebAudio Node Re-Use**: `OscillatorNode` instances are scheduled and disconnected cleanly; buffer nodes and gain nodes are pooled in an object cache to avoid allocation churn.
- **Canvas Rendering Allocation**: The render loop allocates zero objects per frame: scratch vectors, bounding boxes, and color strings are pre-allocated and reused.
- **DOM Stability**: Captions and screen-reader elements reuse existing DOM nodes via text replacement rather than continuous node creation/removal.

### 10.3 Privacy-First Telemetry Architecture
The telemetry pipeline is architecturally isolated from the application database:
- **Permitted Metrics**: Aggregated operational counts (HTTP request rates, latency percentiles, render frame timings, WebAudio error counts, simulation tick durations).
- **Strictly Prohibited Data**: Telemetry events must never include `account_id`, `bird_id`, bird names, personality vector values, presence timestamps, or interaction logs.
- **Enforcement**: Telemetry ingestion uses a separate gateway with a strict JSON schema validator that drops any event containing unauthorized dimensions.

---

## 11. Rollout & Phased Deployment Strategy

```
+------------------------------------------------------------------------------------+
|                         FOUR-PHASE ROLLOUT TIMELINE                                |
+------------------------------------------------------------------------------------+
|                                                                                    |
|  [PHASE 0: Foundations & Synthetic Audit] (Weeks 1 - 4)                            |
|  - Stand up core DB schema, simulation worker, WebAudio engine, and canvas renderer.|
|  - Execute automated synthetic benchmarks: bundle size, frame rate, 30m leak tests.|
|  - Verify asymmetric drift mathematics across 10,000 simulated account days.       |
|                                                                                    |
|  [PHASE 1: Closed Alpha Dogfooding] (Weeks 5 - 8)                                  |
|  - Deploy to internal team (50 accounts, 2 starter birds).                         |
|  - Calibrate audio synthesis aesthetics to prevent auditory fatigue.               |
|  - Audit screen-reader narration and reduced-motion modes with accessibility team.  |
|                                                                                    |
|  [PHASE 2: Limited Canary Beta] (Weeks 9 - 12)                                     |
|  - 500 external invites via magic links.                                           |
|  - Verify multi-device sync behavior across desktop and mobile browsers.           |
|  - Verify absence/return mechanics after 7-day and 14-day inactivity periods.     |
|                                                                                    |
|  [PHASE 3: General Availability (v1 Launch)] (Week 13+)                            |
|  - Open public signups. Start each account with 2 birds.                           |
|  - Activate aviary age progression timers.                                         |
+------------------------------------------------------------------------------------+
```

### 11.1 Aviary Age Progression Schedule (Flock Ramp)
To preserve the quiet deepening of the user's relationship without introducing gamification:
- **Account Creation**: 2 starter birds chosen randomly from the pool of 6 species.
- **Day 60 (Month 2)**: A 3rd bird arrives quietly at the aviary.
- **Day 150 (Month 5)**: A 4th bird arrives.
- **Day 240 (Month 8)**: A 5th bird arrives.
- **Day 360 (Month 12)**: A 6th bird arrives.
- **Day 480 (Month 16)**: The 7th and final bird arrives (strict flock ceiling reached).

*Note: New arrivals are governed strictly by calendar tenure. A user who visits once a week receives their birds on the exact same schedule as a user who visits daily.*

---

## 12. Risk Matrix & Technical Mitigations

| Risk Category | Severity | Failure Mode | Technical Mitigation |
|---|---|---|---|
| **Drift Calibration** | High | Drift feels too fast (reads as Tamagotchi stat grinding) or too slow (feels like static screensaver). | Unit test suite simulates 100 days of varying presence patterns (1m/day, 15m/day, 60m/day). Base learning rate eta is configurable via server environment variables without code redeploy. |
| **Sync Race Conditions** | High | Concurrent multi-device sessions submit conflicting events, causing dropped presence or jerky bird repositioning. | Event-only append log; server simulation tick runs single-threaded per aviary via Redis distributed lock. Client uses smooth cubic-bezier positional interpolation on snapshot updates. |
| **Audio Fatigue & Uncanniness** | High | Procedural WebAudio synthesis sounds robotic, grating, or chiptune-like over extended listening. | Synthesis engine incorporates subtle FM modulation, bandpass filtered breath noise, micro-timing jitter, and dynamic compressor node. Tested across high-end headphones and budget mobile speakers. |
| **Browser Autoplay Blocks** | Medium | Browser blocks WebAudio AudioContext instantiation before user gesture, breaking initial greeting sound. | AudioContext initializes suspended; visually subtle top-bar interaction or first canvas interaction triggers `resume()`. Captions display automatically while audio is muted or pending. |
| **Accessibility Regression** | High | Screen-reader narration gets spammy or falls back to robotic attribute lists (`"Pip: pos 1"`). | Centralized prose generation engine strictly reviews strings against naturalist style guide. Rate-limited to max 1 update per 30 seconds unless user explicitly performs an offer or settle gesture. |
| **PII / Compliance Leak** | High | User email addresses leak into logs, shard keys, or analytics pipelines. | Strict synthetic UUID architecture. Emails encrypted with AES-256-GCM in accounts table; blind index used for lookups. Telemetry schemas reject PII fields automatically. |

---

## 13. Defensible Implementation Decisions

Where specifications allowed engineering interpretation, the following authoritative technical choices have been made:
1. **Rendering Technology**: Implemented via a bespoke lightweight 2D WebGL/Canvas2D renderer (rather than heavy game engines like Pixi.js or Phaser). This guarantees keeping the initial JS bundle comfortably under the 2.0 MB budget.
2. **Presence Sampling Window**: The activity window for keyboard/pointer movement is calibrated to exactly **180 seconds (3 minutes)**. This respects the product's core reality: birdwatching is an idle activity where the user may sit quietly without touching the mouse for a couple of minutes.
3. **Absence Threshold for Greetings**: Short absence (< 30 minutes) triggers a subtle glance or soft single note; long absence (> 6 hours) triggers a noticeable re-orientation and front-perch approach.
4. **Offer Cooldown Window**: Set to **240 seconds (4 minutes)** per bird. This forecloses any possibility of click-spamming while allowing legitimate experimentation during a casual session.
5. **Session Settle Undo Grace**: Set to **5.0 seconds**. Any click within 5 seconds reverses the lighting shift instantly; after 5 seconds, the aviary remains settled until a distinct new session begins.
