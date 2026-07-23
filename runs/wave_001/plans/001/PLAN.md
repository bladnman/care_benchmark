# Pocket Aviary — System Architecture & Implementation Plan (v1)

## Executive Summary & Core Architectural Guarantees

Pocket Aviary is a modern, web-only virtual aviary application where users adopt a small group of birds (2 at launch, growing up to 7 based on aviary age) that live within a single horizontal, unpanned browser scene. The core product value is defined by slow, ambient relationship-building: birds notice the user's presence over days and weeks, drifting their personality traits in response to subtle presence signals and low-frequency interactions.

### Non-Negotiable System Invariants
1. **Server-Authored Canonical State**: The simulation tick runs server-side on a periodic background loop (~1 minute cadence). Clients are stateless renderers that ingest snapshots; clients never mutate personality vectors or compute local state drift.
2. **Strict Conjunction Presence Model**: A presence-event is recorded only when `document.visibilityState === 'visible'`, the window has active focus, and pointer/keyboard activity has occurred within the preceding calibrated activity window (3 minutes).
3. **Monotonic Positive-Only Personality Drift**: Personality traits move up toward expressiveness on positive presence/interactions and **never** move down on neglect. Absence results in ambient stillness, never mistrust, distress, or decay.
4. **Procedural WebAudio Synthesis**: Audio calls are procedurally synthesized client-side via WebAudio nodes. Pre-recorded audio loops are prohibited to prevent phase cancellation, acoustic repetition, and bundle bloat.
5. **No Announcement UI & Naturalist Voice Integrity**: No toasts, banners, streaks, levels, or achievement popups exist on product surfaces. Product surfaces strictly adhere to lowercase, present-tense naturalist field-notebook prose. System/error/auth surfaces drop to a matter-of-fact tone.
6. **Performance & Privacy Boundaries**: Initial JS bundle strictly `< 2MB` gzipped, time-to-first-bird `< 500ms` over 4G, 60fps idle motion, zero memory growth over 30 minutes. Per-bird interaction telemetry is isolated strictly to the user's simulation and never ingested into analytics data warehouses.

---

## 1. Scope & Boundaries (V1 Inclusion & Non-Goals Enforcement)

### 1.1 In-Scope V1 Feature Set
- **Bird Engine & Population**: Starter allocation of 2 birds (assigned procedurally from a 6-species pool); growth cap at 7 birds unlocked strictly by aviary calendar age (e.g., month 2, month 4, etc.).
- **Authentication & Accounts**: Magic link authentication via email; single-user account mapped 1:1 to a single canonical aviary; session token issuance with device revocation; 30-day soft deletion windows; JSON account data export.
- **Interactions**: Procedurally staggered return-greetings, listen-in focus mixing, three offer gestures (seed, song fragment, still pool) with per-bird cooldowns, opt-in settle lighting shifts with 5s undo affordance, read-only field notebook with low-frequency naturalist entries.
- **Visual & Audio Environment**: Single horizontal canvas scene, 3 perch depth zones (front, middle, back), local-time day/night color/lighting shifts, ambient rain/wind weather effects, low-density ambient leaf/feather drift, top-bar auto-fade.
- **Social Affordance**: Read-only visit invitations issued via single-use email links; visitor rendering is non-interactive, isolated from host presence/drift mechanics, with immediate revocation capabilities.
- **Accessibility & Fallbacks**: Screen-reader naturalist prose narration stream via ARIA live regions, reduced-motion cross-fade rendering mode (`prefers-reduced-motion`), procedural call captions, full keyboard navigation, WCAG AA contrast compliance, WebAudio silent fallback with captions enabled.

### 1.2 Explicit Non-Goals & Architectural Enforcement

| Non-Goal Category | Explicit Refusal Rule | Technical Enforcement Mechanism |
| :--- | :--- | :--- |
| **Native Apps** | Web-browser context only. No iOS/Android native wrappers or SDKs. | Architecture built strictly around HTML5 Canvas/WebGL + WebAudio standard browser APIs. |
| **Gamification** | No streaks, scores, levels, badges, visit counters, green-dot calendars, or XP. | Data models contain zero fields for visit counts, streak counters, or milestones. DB schema explicitly omits engagement metrics. |
| **Tamagotchi Mechanics**| No hunger, health meters, distress states, bird death, or negative trait decay. | Drift mathematical function is strictly monotonic ($V_{t+1} \ge V_t$). Simulation tick contains no starvation/decay subroutines. |
| **Social Network Features** | No public discovery, leaderboards, feeds, comments, chat, avatars, or co-presence. | No public API endpoints. Visit tokens map strictly to 1:1 read-only scoped rendering sessions. No multi-user WebSocket channels. |

---

## 2. Architecture & Component Boundary Design

### 2.1 System Architecture Overview

```
                          +-----------------------------------+
                          |     CDN Edge / Static Assets      |
                          |  (HTML, JS App Bundle < 2MB, CSS) |
                          +-----------------------------------+
                                            |
                                            v
+-----------------------------------------------------------------------------------+
|                            Client Web Application                                 |
|                                                                                   |
|  +--------------------+   +---------------------+   +--------------------------+  |
|  | Canvas Render Engine|   | WebAudio Synthesizer|   | Presence & Input Monitor |  |
|  |  (WebGL / 2D Canvas|   | (Procedural Chorus, |   | (Visibility, Focus,      |  |
|  |   Cross-Fade Mode) |   |  Listen-In Ramp)    |   |  Pointer Activity)       |  |
|  +--------------------+   +---------------------+   +--------------------------+  |
|            ^                         ^                           |                |
|            | Interpolated Snapshots  | Call Triggers             | Heartbeats &   |
|            +-------------------------+------------------+        | Interaction    |
|                                                         |        | Events         |
+---------------------------------------------------------|--------|----------------+
                                                          |        |
                                           HTTPS / WSS    |        v
                                   +------------------------------------------------+
                                   |           API Gateway & Auth Service           |
                                   |  (Magic Link Issuer, Session Validator, SSE)   |
                                   +------------------------------------------------+
                                                           |
                                 +-------------------------+-------------------------+
                                 |                                                   |
                                 v                                                   v
               +-----------------------------------+               +-----------------------------------+
               |    Simulation Event Ingestion     |               |    Aviary State Snapshot API      |
               | (Append-Only Event Store / Kafka) |               | (Redis Hot Cache / Read Replica)  |
               +-----------------------------------+               +-----------------------------------+
                                 |                                                   ^
                                 v                                                   | Writes Updated
               +-----------------------------------+                                 | State & Snapshots
               |    Server Simulation Tick Worker  |---------------------------------+
               |  (Periodic ~1m Tick, Low-Pass     |
               |   Drift, Mood Engine, Notebook)   |
               +-----------------------------------+
                                 |
                                 v
               +-----------------------------------+
               |  Primary Relational DB (PostgreSQL|
               |  Accounts, Birds, Vectors, Log)   |
               +-----------------------------------+
```

### 2.2 Component Roles & Boundaries

1. **Frontend Client (Web SPA)**:
   - Responsible for rendering the scene graph, animating micro-motions, running procedural WebAudio nodes, capturing user presence triple-conjunction, and issuing append-only interaction events to the gateway.
   - Interpolates smoothly between server state snapshots $S_n$ and $S_{n+1}$.
2. **API Gateway & Auth Service**:
   - Handles magic link generation and verification (`POST /api/auth/magic-link`, `/verify`).
   - Issues short-lived access JWTs and session tokens tied to synthetic Account UUIDs.
   - Enforces rate limiting on magic links and event submission.
3. **Event Ingestion Store**:
   - Stores append-only streams of client-emitted interaction events (`presence_ping`, `offer_placed`, `listen_in_started`, `settle_triggered`).
   - Prevents client-driven state corruption by acting as a write-ahead log for the simulation tick.
4. **Server Simulation Tick Service**:
   - Runs background ticks at 60-second intervals per active/inactive aviary.
   - Reads uncomputed interaction events, updates bird personality vectors using the low-pass filter formula, evaluates mood state machine transitions, generates notebook entries, and updates the Redis snapshot cache.
5. **Data Persistence Layer (PostgreSQL & Redis)**:
   - **PostgreSQL**: Permanent store for accounts, birds, synthetic UUID mappings, field notebook entries, and persistent vectors.
   - **Redis**: Caches the latest computed aviary state snapshot per account for ultra-fast (`<50ms`) REST/SSE snapshot delivery.

---

## 3. Data Schemas & Domain Data Models

All internal references to accounts use a synthetic UUIDv4 (`account_id`). Raw user emails are encrypted using AES-GCM-256 and stored exclusively on the `accounts` record.

### 3.1 Relational Schema Definitions (PostgreSQL DDL Specification)

```sql
-- Accounts Table
CREATE TABLE accounts (
    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    encrypted_email TEXT NOT NULL UNIQUE,
    email_hash VARCHAR(64) NOT NULL UNIQUE, -- For lookup without decryption
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    status VARCHAR(20) NOT NULL DEFAULT 'active', -- active, pending_deletion
    deletion_requested_at TIMESTAMPTZ NULL
);

-- Active Sessions Table
CREATE TABLE sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    device_user_agent TEXT NOT NULL,
    ip_address INET NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at TIMESTAMPTZ NULL
);

-- Aviary Instance Table
CREATE TABLE aviaries (
    aviary_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(account_id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    is_settled BOOLEAN NOT NULL DEFAULT FALSE,
    settled_at TIMESTAMPTZ NULL
);

-- Birds Table
CREATE TABLE birds (
    bird_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL, -- e.g., 'warbler_grey', 'finch_gold'
    display_name VARCHAR(64) NOT NULL,
    position_index INT NOT NULL CHECK (position_index BETWEEN 0 AND 6),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Personality Vectors Table (Server-Controlled Monotonic Vectors)
CREATE TABLE personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(bird_id) ON DELETE CASCADE,
    boldness NUMERIC(5, 4) NOT NULL DEFAULT 0.2000 CHECK (boldness BETWEEN 0.0 AND 1.0),
    social_warmth NUMERIC(5, 4) NOT NULL DEFAULT 0.2000 CHECK (social_warmth BETWEEN 0.0 AND 1.0),
    vocal_frequency NUMERIC(5, 4) NOT NULL DEFAULT 0.2000 CHECK (vocal_frequency BETWEEN 0.0 AND 1.0),
    plumage_saturation NUMERIC(5, 4) NOT NULL DEFAULT 0.2000 CHECK (plumage_saturation BETWEEN 0.0 AND 1.0),
    curiosity NUMERIC(5, 4) NOT NULL DEFAULT 0.2000 CHECK (curiosity BETWEEN 0.0 AND 1.0),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Bird Mood State Table
CREATE TABLE bird_moods (
    bird_id UUID PRIMARY KEY REFERENCES birds(bird_id) ON DELETE CASCADE,
    current_mood VARCHAR(20) NOT NULL DEFAULT 'wary', -- wary, content, curious, drowsy, alert
    last_perch_zone VARCHAR(10) NOT NULL DEFAULT 'back', -- front, middle, back
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Interaction Event Ledger (Append-Only)
CREATE TABLE interaction_events (
    event_id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    event_type VARCHAR(32) NOT NULL, -- presence_ping, listen_in, offer_seed, offer_song, offer_pool, settle
    target_bird_id UUID NULL REFERENCES birds(bird_id),
    duration_seconds INT NULL DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Field Notebook Table
CREATE TABLE notebook_entries (
    entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    entry_text TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Visit Invitations Table
CREATE TABLE visit_invitations (
    invite_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    invitee_email_hash VARCHAR(64) NOT NULL,
    invite_token VARCHAR(128) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ NULL
);
```

---

## 4. API Surface Specifications

### 4.1 Authentication & Account Management

#### `POST /api/v1/auth/magic-link`
- **Request**: `{ "email": "user@example.com" }`
- **Response**: `{ "status": "sent" }` (Matter-of-fact response; standard HTTP 200 regardless of account existence to prevent email enumeration).
- **Behavior**: Generates single-use 15-minute token. Rate limit: 3 requests per 15 minutes per IP/email.

#### `POST /api/v1/auth/verify`
- **Request**: `{ "token": "magic_link_token_string" }`
- **Response**: `{ "access_token": "JWT", "session_id": "UUID", "account_id": "UUID" }`

#### `DELETE /api/v1/auth/sessions/:session_id`
- **Response**: `{ "status": "revoked" }`

### 4.2 Aviary State & Interaction Endpoints

#### `GET /api/v1/aviary/snapshot`
- **Headers**: `Authorization: Bearer <JWT>`, `If-None-Match: "<ETag>"`
- **Response (200 OK)**:
```json
{
  "aviary_id": "c7b2e9d0-1122-3344-5566-778899aabbcc",
  "server_timestamp": "2026-07-24T06:15:00Z",
  "is_settled": false,
  "birds": [
    {
      "bird_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "display_name": "pip",
      "species_id": "warbler_grey",
      "perch_zone": "front",
      "mood": "content",
      "plumage_saturation": 0.3540,
      "vocal_frequency": 0.4120,
      "last_call_motif": "motif_warbler_rise_02"
    },
    {
      "bird_id": "8a3d42e1-99bb-4112-88aa-1f2e3d4c5b6a",
      "display_name": "wren",
      "species_id": "finch_gold",
      "perch_zone": "middle",
      "mood": "wary",
      "plumage_saturation": 0.2100,
      "vocal_frequency": 0.2800,
      "last_call_motif": "motif_finch_trill_01"
    }
  ],
  "weather": "clear",
  "time_of_day_phase": "morning"
}
```

#### `POST /api/v1/aviary/events`
- **Request**:
```json
{
  "events": [
    {
      "event_type": "presence_ping",
      "duration_seconds": 30,
      "timestamp": "2026-07-24T06:14:30Z"
    },
    {
      "event_type": "listen_in_start",
      "target_bird_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "timestamp": "2026-07-24T06:14:40Z"
    }
  ]
}
```
- **Response**: `{ "accepted_count": 2 }`

### 4.3 Social Visit API

#### `POST /api/v1/social/invites`
- **Request**: `{ "invitee_email": "friend@example.com" }`
- **Response**: `{ "invite_id": "UUID", "expires_at": "2026-08-23T06:15:00Z" }`

#### `GET /api/v1/social/visit/:invite_token/snapshot`
- **Response**: Returns canonical aviary snapshot for host aviary. Read-only permissions enforced. Presence event submission endpoints return 403 Forbidden for visit tokens.

---

## 5. Simulation Engine Design

### 5.1 Server-Side Tick Service & Event Processing
The simulation tick worker runs as a distributed background process (Go / Node.js worker pool) executing every 60 seconds per aviary.

```
                  +-----------------------------------+
                  |  Read Unprocessed Events for      |
                  |  Aviary ID from interaction_events|
                  +-----------------------------------+
                                    |
                                    v
                  +-----------------------------------+
                  | Evaluate Presence Triple-Condition|
                  | Valid Presence Seconds = Sum(Pings|
                  +-----------------------------------+
                                    |
                                    v
                  +-----------------------------------+
                  | Execute Monotonic Low-Pass Drift  |
                  |  V(t+1) = V(t) + Alpha * Signal   |
                  +-----------------------------------+
                                    |
                                    v
                  +-----------------------------------+
                  | Evaluate Mood State Machine       |
                  | (Local Time + Weather + Events)   |
                  +-----------------------------------+
                                    |
                                    v
                  +-----------------------------------+
                  | Evaluate Field Notebook Triggers  |
                  | (Write entry if criteria met)     |
                  +-----------------------------------+
                                    |
                                    v
                  +-----------------------------------+
                  | Commit Vector/Mood/Notebook DB    |
                  | & Invalidate Redis Snapshot Cache |
                  +-----------------------------------+
```

### 5.2 Monotonic Low-Pass Personality Drift Function
Drift is computed per trait $V \in \{\text{boldness, social\_warmth, vocal\_frequency, plumage\_saturation, curiosity}\}$:

$$V_{t+1} = V_t + \alpha \cdot S_{\text{effective}} \cdot (1.0 - V_t)$$

Where:
- $V_t$ is the scalar trait value bounded in $[0.0, 1.0]$.
- $\alpha = 0.00005$ is the base calibration scaling factor for a 1-minute tick interval.
- $S_{\text{effective}}$ is the composite interaction signal score computed from the tick window:
  $$S_{\text{effective}} = w_p \cdot T_{\text{presence\_min}} + w_l \cdot T_{\text{listen\_min}} + w_o \cdot N_{\text{accepted\_offers}}$$
  - Weights: $w_p = 1.0$, $w_l = 2.5$, $w_o = 1.5$.
- **Monotonic Safety Invariant**: If $S_{\text{effective}} = 0$ (user absent or idle background tab), $V_{t+1} = V_t$. **No subtraction occurs under any code branch.**

#### Calibration Verification Bounds
- **1-Week Instrument Threshold**: With 15 minutes of daily active presence for 7 days, $\Delta V \approx +0.035$ to $+0.050$ (detectable via unit test assertion suites).
- **3-Week Visual Threshold**: After 21 days, $\Delta V \approx +0.120$ to $+0.160$ (triggers perch zone preference shifts from `back` to `middle`/`front`, increased plumage saturation shaders, and higher greeting probabilities).

### 5.3 Mood Transition State Machine
Moods $M \in \{\text{wary}, \text{content}, \text{curious}, \text{drowsy}, \text{alert}\}$ transition based on local time and session events:

```
                  +-----------------------------------+
                  |               WARY                |
                  +-----------------------------------+
                     |              ^              |
    Offer Accepted / |              | Noise /      | Dusk /
    High Warmth      |              | Alarm Call   | Night
                     v              |              v
                  +-----------------------------------+
                  |              CONTENT              |<------+
                  +-----------------------------------+       |
                     |              ^              |          |
    Offer Placed /   |              | Cooldown     | Dawn /   | Settle Undo /
    New Call Motif   |              | Expiry       | Morning  | Re-engagement
                     v              |              v          |
                  +-----------------------------------+       |
                  |              CURIOUS              |       |
                  +-----------------------------------+       |
                                                           |  |
                                                           v  v
                                              +---------------------+
                                              |   DROWSY / SETTLED  |
                                              +---------------------+
```

- **Perch Selection Rules**:
  - `wary`: 80% back perch, 20% middle perch, 0% front perch.
  - `content`: 20% back, 60% middle, 20% front.
  - `curious`: 0% back, 30% middle, 70% front.
  - `drowsy`: 50% back, 50% middle, 0% front (fluffed feather posture).

### 5.4 Naturalist Field Notebook Generator
The notebook engine evaluates entry generation during the server tick.
- **Sparsity Constraint**: Maximum 1 entry per 48-72 hours per aviary under regular usage.
- **Generation Logic**: Evaluates specific state transitions rather than generic milestones:
  - *Example Trigger*: `bird_A.boldness` crosses 0.40 OR `bird_A` greets before `bird_B` for the first time in 7 days.
- **Prose Template Engine**: Outputs lowercase, present-tense naturalist observations:
  - `"tuesday — pip greeted before wren today, first time this week."`
  - `"wren is fluffed against the cool air, watching the back perch. low calls only."`

---

## 6. Multi-Device Sync & State Propagation Architecture

1. **Single Source of Truth**: The server database and Redis hot snapshot cache hold the canonical aviary state. Clients maintain no local persistence for personality vectors or mood states.
2. **Preventing Last-Write-Wins (LWW) Corruption**:
   - Clients emit append-only events (`interaction_events` table).
   - Server simulation tick consumes event batches sequentially ordered by server arrival timestamp.
   - Deltas are calculated server-side; clients never submit vector state replacements.
3. **Multi-Tab & Multi-Device Propagation**:
   - Client establishes an SSE (Server-Sent Events) connection (`GET /api/v1/aviary/stream`).
   - When the server tick commits a new snapshot to Redis, a pub/sub message notifies the Gateway to broadcast the updated state snapshot to all connected sessions for that `account_id`.
   - Both phone and desktop clients receive the exact same snapshot simultaneously and interpolate visual transitions.

---

## 7. Frontend Rendering Pipeline & Visual Scene Architecture

### 7.1 Viewport & Scene Layout Rules
- **Canvas Container**: Fixed 16:9 aspect-ratio container with responsive fluid fitting. No horizontal/vertical scrollbars (`overflow: hidden`).
- **3 Depth Perch Layers**:
  - `Layer 0 (Background)`: Sky gradient, distant horizon silhouettes, soft day/night filter.
  - `Layer 1 (Back Perch Zone)`: High/distant branches, low-contrast foliage.
  - `Layer 2 (Middle Perch Zone)`: Main perching branches, ambient leaf drift plane.
  - `Layer 3 (Front Perch Zone)`: High-detail foreground perches, pool/seed interaction zone.
  - `Layer 4 (Foreground Overlay)`: Top-bar chrome, focus indicator rings, captions.

### 7.2 Initial Frame Render ("Mid-Action" Guarantee)
To satisfy the requirement that the aviary appears mid-motion on frame 1:
1. The HTML payload includes an inline edge-cached snapshot JSON script tag if available.
2. The Canvas engine initializes rendering **immediately** upon asset load, placing birds directly at their snapshot-defined perch positions with procedurally offset idle micro-animation phase angles ($\theta = \text{random}(0, 2\pi)$).
3. If snapshot fetching requires a network roundtrip (`>200ms`), the canvas renders the quiet ambient background field (soft sky, gentle foliage breeze) **without showing loading spinners, skeleton frames, or black screens**.

### 7.3 Idle Micro-Motion & State Transitions
- **Idle Motion Subroutines**:
  - `Preen`: Sub-pixel feather tilt, beak sweep (runs when mood = `content`).
  - `Scan`: Dual-phase head rotation with randomized hold duration (runs when mood = `wary` or `alert`).
  - `Weight Shift`: 2px vertical spring-damper displacement on perch.
- **Reduced-Motion Rendering Mode (`prefers-reduced-motion`)**:
  - Replaces 60fps skeletal/bezier path movement with slow cross-fades ($1.2\text{s}$ opacity blend) between static keyframe poses.
  - Disables ambient drifting leaf/feather particles entirely.
  - Retains day/night lighting transitions with doubled fade durations.

---

## 8. WebAudio Procedural Synthesis & Audio Pipeline

```
  +-------------------------------------------------------------------------------+
  |                      Per-Bird WebAudio Voice Engine                           |
  |                                                                               |
  |  +---------------------+      +---------------------+      +---------------+  |
  |  | Sine/Triangle Osc   | ---> | Biquad Filter Node  | ---> | Gain Node     |  |
  |  | (Motif Pitch Sweep) |      | (Tone & Formant)    |      | (Envelope)    |  |
  |  +---------------------+      +---------------------+      +---------------+  |
  |                                                                    |          |
  |  +---------------------+      +---------------------+              v          |
  |  | Pink Noise Gen      | ---> | Bandpass Filter     | ---> +---------------+  |
  |  | (Breath & Chiff)    |      | (Call Texture)      |      | Summer / Mixer|  |
  |  +---------------------+      +---------------------+      +---------------+  |
  +--------------------------------------------------------------------|----------+
                                                                       |
                                                                       v
                                                           +----------------------+
                                                           | Master Bus / Chorus  |
                                                           | Polyphony Suppressor |
                                                           +----------------------+
                                                                       |
                                                                       v
                                                           +----------------------+
                                                           | Listen-In Gain Ramp  |
                                                           | (Focused vs Ambient) |
                                                           +----------------------+
                                                                       |
                                                                       v
                                                           +----------------------+
                                                           | Audio Destination    |
                                                           +----------------------+
```

### 8.1 Procedural Call Generation Engine
Each bird species possesses a library of FM synthesis parameter templates (motifs).
- **Pitch Sweep**: Frequency modulation using custom exponential envelope curves ($f_{\text{start}} \to f_{\text{peak}} \to f_{\text{end}}$).
- **Timbre Control**: Dual biquad filters modulated by bird `plumage_saturation` and `mood`.
- **Chorus Anti-Collision Algorithm**: When multiple birds call simultaneously, the audio engine applies a randomized delay jitter ($50\text{ms} - 300\text{ms}$) between triggers to prevent phase cancellation and unnatural acoustic stacking.

### 8.2 Listen-In Mix Ramping
When a user engages Listen-In on Bird $A$:
- Bird $A$ Gain Node: Ramps smoothly from $0.7 \to 1.0$ over $1.5\text{s}$ (`exponentialRampToValueAtTime`).
- All Other Birds ($B \dots G$) Gain Nodes: Duck smoothly from $0.7 \to 0.15$ over $1.5\text{s}$ (never muted to $0.0$).
- Disengaging Listen-In ramps all voices back to standard ambient baseline ($0.7$) over $2.0\text{s}$.

### 8.3 Silent WebAudio Fallback Mode
If `AudioContext` fails to initialize or permission is withheld:
1. WebAudio engine disables output silently.
2. System automatically enables procedural **Call Captions** in top-bar accessibility settings.
3. Captions render naturalist call descriptions (e.g., `"pip: a soft three-note rise from the front perch"`) near the calling bird.

---

## 9. Accessibility Surfaces Specification

### 9.1 Screen-Reader Narration Stream
- **Element Structure**: Hidden ARIA Live region (`<div id="aviary-narration" aria-live="polite" aria-atomic="true" class="sr-only"></div>`).
- **Cadence Engine**: Evaluates state every 45 seconds at idle. Updates text only when meaningful state drift or position shifts occur.
- **Tone Compliance**: Lowercase, present-tense naturalist prose:
  - *Correct*: `"a small grey bird perches on the front rail, calling softly. light is gentle."`
  - *Prohibited*: `"Screen reader update: Pip position 1, status active."`

### 9.2 Keyboard Navigation Map

| Key / Shortcut | Target Context | Executed Action |
| :--- | :--- | :--- |
| `Tab` | Top Bar / Chrome | Cycles through Settings, Accessibility, Notebook, Offer icons. |
| `Tab` -> `Enter` | Aviary Canvas | Enters Canvas interaction mode; moves focus ring to first bird. |
| `Left Arrow` / `Right Arrow` | Canvas Birds | Navigates focus sequentially between perched birds. |
| `Enter` / `Space` | Focused Bird | Triggers **Listen-In** focus mode on target bird. |
| `Escape` | Active Listen-In | Disengages Listen-In; ramps audio back to ambient. |
| `O` | Global Shortcut | Opens the **Offer** selection panel in the top bar. |
| `S` | Global Shortcut | Triggers the **Settle** lighting shift (5s undo prompt active). |

---

## 10. Performance Budgets, Optimization & Observability

### 10.1 Quantitative Performance Budget Contract

```
+-----------------------------------------------------------------------------------+
|                        POCKET AVIARY PERFORMANCE BUDGETS                          |
+-----------------------------------+-----------------------------------------------+
| METRIC                            | STRICT BUDGET CEILING                         |
+-----------------------------------+-----------------------------------------------+
| Initial JS Bundle (gzipped)       | < 2.0 MB (Strict CI build gate)               |
| Time-To-First-Bird (4G Mid-Tier)  | < 500 ms (Navigation to Canvas First Render)  |
| Idle Motion Frame Rate            | 60 fps (Tested on 5-year-old mid-spec laptop) |
| Memory Allocation Growth          | 0 KB net growth over 30-minute session        |
| Server Tick Latency               | p99 < 5,000 ms (Alarm threshold)              |
+-----------------------------------+-----------------------------------------------+
```

### 10.2 Architectural Asset Optimization
1. **No External Audio Assets**: Procedural WebAudio synthesis eliminates all `.mp3`/`.wav` downloads.
2. **Vector Graphics & Procedural Palette**: Bird bodies rendered via compact SVG path definitions; plumage saturation shaders applied procedurally in canvas.
3. **Code Splitting**: Dynamic `import()` for non-critical surfaces:
   - `auth_settings_bundle.js` (loaded on settings click)
   - `notebook_viewer_bundle.js` (loaded on notebook icon click)
   - `social_invite_bundle.js` (loaded on invite action)

### 10.3 Telemetry & Privacy Wall

```
+-----------------------------------------------------------------------------------+
|                           PRIVACY & TELEMETRY BOUNDARY                            |
+-----------------------------------------------------------------------------------+
|  PERMITTED OPERATIONAL TELEMETRY           |   STRICTLY FORBIDDEN TELEMETRY       |
|  (Injected into Datadog / CloudWatch)      |   (Never exported or logged)         |
+--------------------------------------------+--------------------------------------+
| - Gateway HTTP request rate & status codes  | - Per-bird personality vector values |
| - API Endpoint p50/p95/p99 latencies       | - Per-bird mood state histories      |
| - Server simulation tick execution duration| - Individual user presence duration  |
| - WebAudio initialization error count      | - Field notebook entry text contents |
| - Synthetic ping performance test timings  | - User email strings or hashes       |
+--------------------------------------------+--------------------------------------+
```

---

## 11. Phased Rollout & Progression Plan

### 11.1 Aviary Age Growth Progression Schedule
To adhere to the principle that adoption reflects relationship deepening rather than gamified rewards:

| Aviary Calendar Age | Unlocked Bird Population Cap | Mechanics & Selection |
| :--- | :--- | :--- |
| **Day 0 (Onboarding)** | 2 Starter Birds | System auto-selects 2 distinct species from 6-species pool. User assigns names. |
| **Month 2 (~60 Days)** | 3rd Bird Offer | Quiet offer appears in top bar to welcome a 3rd species. |
| **Month 4 (~120 Days)**| 4th Bird Offer | 4th bird offer unlocked. |
| **Month 7 (~210 Days)**| 5th Bird Offer | 5th bird offer unlocked. |
| **Month 10 (~300 Days)**| 6th Bird Offer | 6th bird offer unlocked. |
| **Month 14 (~420 Days)**| 7th Bird Offer (Maximum Cap) | Final bird offer unlocked. Hard cap enforced by engine. |

### 11.2 Engineering Release Sequence
1. **Phase 1: Core Engine & Data Model Validation** (Simulation tick worker, DB schemas, monotonic drift unit tests).
2. **Phase 2: Canvas & Procedural Audio Prototype** (WebGL/Canvas renderer, WebAudio motif generator, listen-in mix ramps).
3. **Phase 3: Auth, Sync & Privacy Integration** (Magic link auth, synthetic UUID isolation, SSE snapshot stream, Redis cache).
4. **Phase 4: Accessibility & Naturalist Polish** (Screen-reader narration generator, reduced-motion cross-fader, notebook generator).
5. **Phase 5: Canary Load & Synthetic Performance Verification** (CI bundle size checks, 30-minute memory leak profiling, synthetic 4G latency benchmarks).

---

## 12. Risk Matrix & Technical Mitigations

| Risk | Impact | Root Cause | Technical Mitigation |
| :--- | :--- | :--- | :--- |
| **Drift Over-Calibration** | High | Drift factor $\alpha$ set too high, causing birds to change noticeably between consecutive days. | Calibration test suite asserts $\Delta V \le 0.05$ after 7 days of 15-min daily presence. Staging automated drift check. |
| **Audio Phase Cancellation** | Medium | Multiple procedural calls firing at exact same sample frame. | Mandatory anti-collision jitter ($50\text{ms}-300\text{ms}$) injected in chorus voice dispatcher. |
| **Multi-Device Race Conditions** | High | Client sending absolute state overrides resulting in lost presence/drift data. | Complete deprecation of client state mutation. Server tick is 100% sole author of vector deltas. |
| **Accessibility Degradation** | High | Screen reader flooded with high-frequency ARIA updates. | ARIA live region rate-limiter throttles updates to max 1 per 45 seconds during idle watching. |
| **Memory Growth in Canvas/WebAudio** | High | Unreleased audio nodes or canvas texture allocations during 30m+ sessions. | Node recycling pool for WebAudio nodes; explicit GC recycling of canvas offscreen buffers in animation loop. |

---
*Plan complete and ready for execution by engineering team.*
