# Pocket Aviary — System Architecture & Engineering Implementation Plan (v1)

## 1. Scope & System Boundaries

Pocket Aviary is an ambient, browser-based virtual aviary hosting a small community of procedural birds that evolve over days and weeks in response to unhurried human presence. This specification establishes the implementation blueprint for the v1 production system.

### 1.1 In-Scope for v1
- **Single-User Accounts & Aviaries**: Exactly one canonical aviary per account.
- **Population Lifecycle**: Aviary begins with exactly two starter birds assigned from a 6-species pool. The aviary grows strictly based on aviary age milestones up to a hard cap of seven birds.
- **Authentication**: Passwordless email magic-link sign-in (15-minute token expiry, single-use, cryptographic revocation). Internal identity uses synthetic account UUIDs; emails are encrypted at rest and never exposed across service domains.
- **Interaction Model**:
  - Unhurried presence (idle attention tracking governed by a 3-part simultaneous conjunction).
  - Procedural return-greeting (varied by absence duration and personality, never simultaneous chorus, never accompanied by banner or toast).
  - Listen-in mix rebalancing (gradual acoustic soloing without muting other birds).
  - Naturalist offerings (seed, song fragment, still pool) subject to per-bird cooldowns.
  - Settle gesture (soft twilight transition with a 5-second undo window; functionally identical to closing the tab).
  - Field notebook (read-only, naturalist prose observations generated sparsely every few days).
- **Simulation Engine**: Server-side continuous tick (~60-second cycle) driving monotonic personality drift, fast-timescale Markovian mood shifts, localized diurnal/solar cycle, and subtle weather patterns.
- **Client Render Engine**: Responsive single-screen Canvas2D/WebGL rendering with zero panning or zoom, continuous idle micro-motion, and immediate "already in motion" first-frame load.
- **Audio Architecture**: Pure client-side procedural audio synthesis via WebAudio (FM/additive synthesis and resonance filtering); no pre-recorded audio loops.
- **Accessibility Systems**: Screen-reader live prose narration (30–60s idle cadence), reduced-motion mode (cross-fading static key-poses), real-time call captions, WCAG AA compliance, and complete keyboard navigation.
- **Quiet Social Visits**: One-time email-based invitations for read-only ambient visits. Revocable at will; strictly isolated from host presence/drift; zero co-presence, avatars, chat, or comments.
- **Lifecycle & Privacy Management**: On-demand JSON state export; 30-day soft-delete grace period followed by an automated hard database purge.

### 1.2 Explicit Non-Goals (Out of Scope for v1 and Invariants)
- **Native Mobile Applications**: Strictly web-only (responsive mobile web supported via modern browser standards).
- **Gamification Mechanics**: Zero streaks, scores, levels, badges, achievements, XP, adoption counters, or green-dot visit calendars. No telemetry or metrics exposed as gamified numbers.
- **Tamagotchi / Custodial Dynamics**: Birds do not die, starve, fall ill, or show distress. No custodial maintenance schedules or hunger meters. Absence produces ambient quietness, never penalty or negative trait drift.
- **Social Network Trappings**: No public directories, global feeds, user profiles, followers, comments, reactions, or leaderboards.
- **Announcements & Notifications**: No "Welcome back" banners, no level-up modals, no toasts, no external push notifications, and no email re-engagement campaigns.

---

## 2. High-Level Architecture & Service Topology

The system enforces a strict boundary: the server owns all state, simulation, drift calculations, and persistence; the client is a stateless rendering and input-capture node.

```
                                      +-----------------------------------+
                                      |        Edge / CDN (Cloudflare)     |
                                      |  - TLS Termination & Static HTML  |
                                      |  - Bootstrap State Snapshot Cache |
                                      +-----------------+-----------------+
                                                        |
                                       HTTPS / SSE Stream
                                                        |
                                                        v
+-------------------------------------------------------+-------------------------------------------------------+
|                                              API Gateway (Go / Node.js)                                       |
|  - Magic-Link Auth & JWT Session Tokens             - Snapshot Delivery (`GET /api/v1/aviary`)               |
|  - Append-Only Event Ingestion (`POST /events`)     - Visit Token Validation & Social Boundary Guard          |
+---------------------------+---------------------------------------------------+-------------------------------+
                            |                                                   |
                   SQL Event Writes                                      Read Snapshots
                            v                                                   |
+---------------------------+---------------------------+                       |
|           PostgreSQL 16 Primary Cluster               |                       |
|  - `accounts` (Encrypted PII, Synthetic UUIDs)        |                       |
|  - `aviaries` & `birds` (Canonical Vectors)           |                       |
|  - `interaction_events` (Append-Only Event Queue)     |                       |
|  - `aviary_snapshots` (Denormalized Snapshot Store)   |<----------------------+
+---------------------------^---------------------------+
                            |
                   Reads Events & Updates Canonical State
                            |
+---------------------------+---------------------------+
|          Simulation Worker Daemon (Go / Rust)         |
|  - Master Tick Scheduler (60s Cadence)                |
|  - Discrete Low-Pass Filter Drift Evaluator           |
|  - Markov Mood State Machine                          |
|  - Diurnal & Weather Transition Engine                |
|  - Sparsity-Gated Field Notebook Generator            |
+-------------------------------------------------------+
```

### 2.1 Service Components
1. **Edge / CDN Layer**:
   - Delivers client assets (HTML, bundle JS < 2MB, CSS, SVGs) with HTTP/3 and Brotli compression.
   - Terminates TLS and proxies dynamic `/api/v1/*` requests.
   - When an authenticated session cookie is present, injects a pre-rendered bootstrap snapshot into the initial HTML document payload, satisfying the <500ms Time-to-First-Bird budget.
2. **API Gateway Service**:
   - Stateless microservice handling authentication, session validation, read snapshot queries, event ingestion, and invitation routing.
   - Enforces rate limiting per IP and per account.
   - Implements the strict boundary between host mutations and visitor read-only access.
3. **Simulation Worker Fleet**:
   - Distributed background workers partitioned by `aviary_id` hash rings.
   - Fires a discrete simulation tick once every 60 seconds per aviary.
   - Processes unhandled events from `interaction_events`, computes drift and mood updates, generates notebook observations, and writes a newly committed canonical snapshot.
4. **Data Tier (PostgreSQL 16)**:
   - Primary transactional datastore utilizing strict foreign keys, read replicas for snapshot queries, and cryptographic row-level encryption for sensitive columns.
5. **Observability Pipeline (Isolated)**:
   - Prometheus and OpenTelemetry agents collecting system-level metrics (tick latency, HTTP throughput, database locks, WebAudio error counters).
   - Structurally isolated from user data; zero interaction logs or bird trait dimensions enter the analytics sink.

---

## 3. Data Model & Storage Schema

All relational entities use synthetic UUIDv4 primary keys. Personal Identifiable Information (PII) is isolated and encrypted using AES-256-GCM with keys managed via an external Key Management Service (KMS).

### 3.1 Database Schemas (PostgreSQL DDL)

```sql
-- Accounts & Authentication
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    encrypted_email BYTEA NOT NULL,
    email_bidx VARCHAR(64) NOT NULL UNIQUE, -- Blind index (HMAC-SHA256) for lookups
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    settled_at TIMESTAMPTZ,
    scheduled_deletion_at TIMESTAMPTZ, -- 30-day soft delete grace period
    status VARCHAR(20) NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'pending_deletion', 'purged'))
);

CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    user_agent TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at TIMESTAMPTZ
);

CREATE TABLE magic_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID REFERENCES accounts(id) ON DELETE CASCADE,
    email_bidx VARCHAR(64) NOT NULL,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    expires_at TIMESTAMPTZ NOT NULL,
    consumed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Aviary & Environment
CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(id) ON DELETE CASCADE,
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    weather_condition VARCHAR(32) NOT NULL DEFAULT 'clear', -- 'clear', 'soft_rain', 'gentle_breeze'
    weather_transition_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Birds & Personality Vectors
CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL, -- e.g., 'song_sparrow', 'mourning_dove', 'goldfinch'
    name VARCHAR(64) NOT NULL,
    -- Personality Vector: normalized continuous values [0.0000, 1.0000]
    boldness NUMERIC(5, 4) NOT NULL DEFAULT 0.2000,
    social_warmth NUMERIC(5, 4) NOT NULL DEFAULT 0.2000,
    vocal_frequency NUMERIC(5, 4) NOT NULL DEFAULT 0.2500,
    plumage_saturation NUMERIC(5, 4) NOT NULL DEFAULT 0.3000,
    curiosity NUMERIC(5, 4) NOT NULL DEFAULT 0.2000,
    -- Current Fast-Timescale State
    current_mood VARCHAR(32) NOT NULL DEFAULT 'wary' CHECK (current_mood IN ('wary', 'content', 'curious', 'drowsy', 'alert')),
    perch_zone VARCHAR(16) NOT NULL DEFAULT 'back' CHECK (perch_zone IN ('front', 'middle', 'back')),
    mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Append-Only Interaction Events Queue
CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    event_type VARCHAR(32) NOT NULL, -- 'presence_heartbeat', 'listen_in_start', 'listen_in_end', 'offer', 'settle', 'settle_undo'
    payload JSONB NOT NULL DEFAULT '{}',
    client_timestamp TIMESTAMPTZ NOT NULL,
    ingested_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_at TIMESTAMPTZ -- Populated by simulation worker
);
CREATE INDEX idx_interaction_events_unprocessed ON interaction_events (aviary_id, id) WHERE processed_at IS NULL;

-- Denormalized Snapshots (Read Model)
CREATE TABLE aviary_snapshots (
    aviary_id UUID PRIMARY KEY REFERENCES aviaries(id) ON DELETE CASCADE,
    tick_sequence BIGINT NOT NULL,
    snapshot_json JSONB NOT NULL,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Field Notebook Entries
CREATE TABLE notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    observation_text TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_notebook_aviary ON notebook_entries(aviary_id, created_at DESC);

-- Social: Visits & Invitations
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    visitor_email_bidx VARCHAR(64) NOT NULL,
    encrypted_visitor_email BYTEA NOT NULL,
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invitation_id UUID NOT NULL REFERENCES visit_invitations(id) ON DELETE CASCADE,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    ended_at TIMESTAMPTZ,
    duration_seconds INT
);
```

---

## 4. API Surface & Contract Specifications

All API responses use standard HTTP status codes. Client error messages in response payloads adhere strictly to the **matter-of-fact** register.

### 4.1 Authentication & Account Management
- `POST /api/v1/auth/magic-link`
  - Body: `{"email": "user@example.com"}`
  - Response: `200 OK` (`{"message": "If that address is registered, a sign-in link has been sent."}`)
- `POST /api/v1/auth/verify`
  - Body: `{"token": "raw_magic_token_string"}`
  - Response: `200 OK` with `Set-Cookie: session_token=...; HttpOnly; Secure; SameSite=Lax; Path=/`
- `POST /api/v1/auth/logout`
  - Response: `200 OK` (Invalidates session record).
- `POST /api/v1/account/export`
  - Response: `202 Accepted` (`{"message": "Your aviary snapshot is being generated and will be sent to your email."}`)
- `POST /api/v1/account/delete`
  - Response: `200 OK` (`{"scheduled_deletion_at": "2026-10-06T08:00:00Z"}`)
- `POST /api/v1/account/restore`
  - Response: `200 OK` (`{"message": "Your account deletion request has been canceled."}`)

### 4.2 Aviary State & Event Stream
- `GET /api/v1/aviary`
  - Returns current canonical state snapshot.
  ```json
  {
    "tick_sequence": 104230,
    "timestamp": "2026-09-06T08:23:00Z",
    "diurnal_phase": "morning",
    "sky_tint": {"r": 0.92, "g": 0.94, "b": 0.96},
    "weather": {"type": "clear", "wind_velocity": 0.1},
    "settled": false,
    "birds": [
      {
        "id": "7b794bf2-3e28-4e89-8d14-3a9ec6728091",
        "species_id": "song_sparrow",
        "name": "Pip",
        "perch_zone": "front",
        "current_mood": "curious",
        "plumage_saturation": 0.4210,
        "active_action": "scanning",
        "call_motif_seed": 48192
      },
      {
        "id": "e4f877d9-3f0a-4712-a1f9-01c9a6efbd15",
        "species_id": "mourning_dove",
        "name": "Wren",
        "perch_zone": "back",
        "current_mood": "content",
        "plumage_saturation": 0.3540,
        "active_action": "preening",
        "call_motif_seed": 19482
      }
    ]
  }
  ```
- `GET /api/v1/aviary/stream`
  - Server-Sent Events (SSE) endpoint emitting `event: snapshot` every 60-second tick or on immediate environmental transitions.
- `POST /api/v1/aviary/events`
  - Ingests batch interaction events from the client.
  ```json
  {
    "events": [
      {
        "event_type": "presence_heartbeat",
        "client_timestamp": "2026-09-06T08:23:00Z",
        "payload": {"duration_seconds": 60}
      },
      {
        "event_type": "listen_in_start",
        "client_timestamp": "2026-09-06T08:23:15Z",
        "payload": {"bird_id": "7b794bf2-3e28-4e89-8d14-3a9ec6728091"}
      }
    ]
  }
  ```

### 4.3 Field Notebook
- `GET /api/v1/aviary/notebook?cursor=...&limit=20`
  - Paginated list of naturalist observations.
  ```json
  {
    "entries": [
      {
        "id": "90e2cf0d-27b9-43c2-b3da-8e2b85bf2d8f",
        "observation_text": "tuesday — pip greeted before wren today, first time this week.",
        "created_at": "2026-09-05T07:14:22Z"
      }
    ],
    "next_cursor": null
  }
  ```

### 4.4 Visit & Social Surface
- `POST /api/v1/invitations`
  - Body: `{"visitor_email": "friend@example.com"}`
  - Response: `201 Created` (`{"invite_id": "...", "expires_at": "..."}`)
- `DELETE /api/v1/invitations/{invite_id}`
  - Response: `200 OK` (Immediate revocation).
- `GET /api/v1/invitations/log`
  - Returns recent visits for host review.
- `GET /api/v1/visit/{visit_token}`
  - Public read-only endpoint for invited visitors. Validates token; returns snapshot stream. Disallows any write operations.

---

## 5. Simulation Engine Design

The simulation worker runs independently of client connections. It is the sole authority mutating personality vectors, determining mood transitions, generating notebook entries, and driving environmental state.

### 5.1 Tick Scheduler Execution Cycle (Every 60s)
1. **Event Pull**: Dequeue unprocessed interaction events for `aviary_id` up to `current_tick_time`.
2. **Presence Accounting**:
   - Verify presence heartbeat validity. Deduplicate simultaneous heartbeats across multiple devices/tabs (max 60 seconds presence credited per 60-second tick).
   - If user presence is recorded, increment cumulative active attention counter.
3. **Drift Function Execution**:
   - Compute delta for each personality trait using a discrete Low-Pass Filter (LPF):
     $$\vec{P}_{t} = \vec{P}_{t-1} + \Delta \vec{P}_{t}$$
     $$\Delta \text{boldness} = \min(\mu_b, \alpha_b \cdot T_{\text{presence}} + \beta_b \cdot N_{\text{offer\_proximity}})$$
     $$\Delta \text{social\_warmth} = \min(\mu_w, \alpha_w \cdot T_{\text{presence}} + \gamma_w \cdot T_{\text{listen\_in}})$$
     $$\Delta \text{vocal\_frequency} = \min(\mu_v, \alpha_v \cdot T_{\text{presence}} + \gamma_v \cdot T_{\text{listen\_in}})$$
     $$\Delta \text{plumage\_saturation} = \min(\mu_p, \alpha_p \cdot T_{\text{presence}})$$
     $$\Delta \text{curiosity} = \min(\mu_c, \alpha_c \cdot T_{\text{presence}} + \beta_c \cdot N_{\text{offer\_accepted}})$$
   - **Monotonic Invariant**: $\Delta \vec{P}_t \ge 0$. If $T_{\text{presence}} = 0$, $\Delta \vec{P}_t = 0$. Traits never decrement.
   - **Calibration Constants**:
     - Baseline presence coefficient: $\alpha = 0.00004$ per minute of active presence.
     - With ~60 minutes daily presence over 7 days (420 minutes): $\Delta P \approx 0.0168 - 0.0350$ (measurable in telemetry/instruments).
     - Over 21 days (1260 minutes): $\Delta P \approx 0.0800 - 0.1500$ (distinct visible changes: higher front-perch frequency, rich plumage shaders, active calling).
     - Saturation ceiling: Max trait value is capped at $1.0000$.
4. **Fast-Timescale Mood State Machine**:
   - State transition matrix governed by:
     - Local solar elevation: Dawn $\to$ Alert; Midday $\to$ Content/Curious; Dusk $\to$ Drowsy; Night $\to$ Settled/Drowsy.
     - Ambient Weather: Soft rain increases probability of Drowsy/Quiet by $0.35$.
     - Interaction Events: Recent accepted offer transitions bird to `content` for 15–30 minutes.
     - Social Contagion: If an adjacent bird triggers an alarm call, probability of entering `wary` increases by $0.40 \times (1.0 - \text{boldness})$.
5. **Perch Selection Model**:
   - Birds evaluate perch zones on mood shift or every 5–15 minutes:
     $$P(\text{front}) = \text{clamp}(0.1 + 0.6 \cdot \text{boldness} + 0.2 \cdot \mathbb{I}_{\text{content}} - 0.3 \cdot \mathbb{I}_{\text{wary}}, 0.05, 0.90)$$
     $$P(\text{back}) = \text{clamp}(0.7 - 0.5 \cdot \text{boldness} + 0.3 \cdot \mathbb{I}_{\text{wary}} - 0.2 \cdot \mathbb{I}_{\text{content}}, 0.05, 0.90)$$
     $$P(\text{middle}) = 1.0 - P(\text{front}) - P(\text{back})$$
6. **Procedural Field Notebook Generator**:
   - Evaluated at tick boundary with a strict sparsity gate (minimum 48 hours between standard entries).
   - Generates entries based on notable differential events:
     - Relative greeting order: First greeter of the day compared to historic logs.
     - Extended quiet or weather reaction.
     - Distinct perch choices under specific weather.
   - Prose rendered through deterministic naturalist grammar templates (all lowercase, present-tense, bird-focused, zero exclamation points, zero user-centric metrics).
7. **Adoption Threshold Evaluator**:
   - Evaluates aviary age:
     - Day 0: 2 starter birds.
     - Day 30: 3rd bird arrives.
     - Day 90: 4th bird arrives.
     - Day 180: 5th bird arrives.
     - Day 270: 6th bird arrives.
     - Day 360: 7th bird arrives (hard cap).

---

## 6. Multi-Device Synchronization & Conflict Prevention

### 6.1 Canonical Single-Writer Model
- The server simulation tick is the sole author of canonical state.
- Clients never run a local simulation tick and never push vector mutations.
- Multi-device sessions (e.g. laptop open in study, phone opened in kitchen) receive identical canonical snapshots from the SSE stream or snapshot queries.

### 6.2 Presence Disambiguation & Deduplication
- Each client independently evaluates presence rules:
  1. `document.visibilityState === 'visible'`
  2. `document.hasFocus() === true`
  3. User input (`pointermove`, `keydown`, `touchstart`) recorded within the last 180 seconds.
- Every 60 seconds, a conforming client issues a `presence_heartbeat`.
- If the server receives heartbeats from multiple sessions belonging to the same account within a single 60-second window, the server takes the union of presence intervals, crediting at most 60 seconds of presence time. Double-counting attention is impossible.

### 6.3 Settle State Synchronization & Undo Window
- When a user triggers "Settle", the client immediately initiates a visual twilight ease and sends a `settle` event.
- The client starts an internal 5-second undo timer. If the user clicks anywhere in the aviary during this window, the client dispatches a `settle_undo` event and restores normal lighting.
- If the 5-second window elapses without undo, the server commits `settled: true`. Other connected devices receive the updated snapshot via SSE and smoothly ease into the settled lighting state over 4.0 seconds.

---

## 7. Frontend Rendering Pipeline

The visual rendering engine runs inside a fixed-dimension virtual viewport ($1920 \times 1080$ coordinate space) rendered into an HTML5 Canvas using Canvas2D or WebGL, scaled via CSS `object-fit: contain` to ensure no bird or perch is ever cropped on any screen ratio.

```
+-----------------------------------------------------------------------------------+
| Top Bar Chrome (Z-Index: 10)                                                      |
| [Settings / Account] [Accessibility]               [Field Notebook] [Offer Gift]  |
+-----------------------------------------------------------------------------------+
| Aviary Canvas (Fixed Virtual Coordinates: 1920 x 1080)                            |
|                                                                                   |
|  [Background Layer: Diurnal Sky Gradient, Sun/Moon Elevation, Soft Weather Tint]  |
|                                                                                   |
|  [Back Perch Zone: Subtle Parallax Foliage, Muted Birds (Low Boldness / Wary)]    |
|                                                                                   |
|  [Middle Perch Zone: Central Branches, Resting Birds]                            |
|                                                                                   |
|  [Front Perch Zone: Detailed Perch Rail, Prominent Birds (High Boldness / Alert)]|
|                                                                                   |
|  [Foreground Layer: Subtle Parallax Twigs, Rare Drifting Leaves / Down Feathers]  |
+-----------------------------------------------------------------------------------+
```

### 7.1 "Already in Motion" First Frame Load
- To meet the requirement that the aviary never "wakes up" or shows a loading spinner:
  1. Initial HTML payload contains the bootstrapped state snapshot.
  2. The renderer parses the snapshot synchronously before first paint.
  3. Bird animation phases are initialized using deterministic mathematical offsets:
     $$\text{phase} = (\text{timestamp}_{\text{now}} + \text{seed}_{\text{bird}}) \pmod{\text{period}}$$
  4. Birds are rendered mid-pose (mid-preen, head-tilted, wings tucked) on frame 1.
  5. If the network snapshot is delayed on a cold load, the renderer immediately paints the ambient diurnal sky and resting foliage, fading birds in softly as the snapshot resolves within the 500ms window.

### 7.2 Return-Greeting Choreography
- When the tab gains visibility after an absence, the client evaluates absence duration:
  - $< 5$ minutes: Ambient glance or head-tilt only.
  - $5$ minutes to $24$ hours: Bolder bird perches forward, produces a characteristic two-note greeting call.
  - $> 24$ hours: Extended procedural sequence; primary greeter steps forward with a full motif call, followed after a 400–900ms randomized offset by a second bird calling from the middle perch.
- Greetings are strictly staggered and procedurally varied; simultaneous choruses are prohibited.

### 7.3 Idle Micro-Motion Engine
- 60 FPS animation loop driven by `requestAnimationFrame`.
- Micro-motions are procedural combinations of low-frequency Perlin noise and sine waves:
  - Respiration: Subtle chest expansion ($0.8\text{ Hz}$).
  - Weight-shift: Small angular rocking on the perch ($0.15\text{ Hz}$, randomized pause).
  - Head scanning: Discrete, rapid rotational glances between fixations, modulated by the bird's `curiosity` and `wary` mood.
- When the browser tab is hidden (`visibilityState === 'hidden'`), the animation loop halts completely to conserve battery and CPU resources.

### 7.4 Reduced-Motion Mode
- Automatically enabled when `prefers-reduced-motion: reduce` is detected or toggled via accessibility settings.
- Continuous skeletal and vertex animations are completely disabled.
- Motion is replaced with slow cross-fades (600–900ms ease) between static photographic-style key-poses.
- Flight transitions between perches are rendered as a gentle cross-fade from the starting perch to the destination perch.
- Ambient leaf and feather drift particles are removed entirely.

---

## 8. Procedural Audio Pipeline

The audio engine relies exclusively on the WebAudio API (`AudioContext`). Zero audio files or sample loops are loaded over the network.

```
                                  +-------------------------------------------------+
                                  |                 AudioContext                    |
                                  +-----------------------+-------------------------+
                                                          |
                                           Master Gain (0dB / Settle Ease)
                                                          |
                                                Dynamics Compressor
                                                          |
                      +-----------------------------------+-----------------------------------+
                      |                                                                       |
                      v                                                                       v
          [Ambient Soundscape Bus]                                                  [Bird Chorus Bus]
          - Wind: Pink noise + BiquadFilter                                         Sub-buses per bird (up to 7)
          - Rain: White noise burst impulses                                                  |
          - Gain: -18dB (Drops to -24dB in listen-in)                                          v
                                                                             +---------------------------------+
                                                                             | Per-Bird Voice Generator (x7)   |
                                                                             | - Carrier (Sine / Triangle)     |
                                                                             | - FM Modulator Oscillator       |
                                                                             | - Syrinx Biquad Bandpass Filter |
                                                                             | - Stereo PannerNode (-0.8..0.8) |
                                                                             | - Bird Channel GainNode         |
                                                                             +---------------------------------+
```

### 8.1 Syrinx Synthesis Model
Each vocalization is synthesized via frequency-modulated (FM) oscillators passed through a formant-shaping bandpass filter simulating the avian vocal organ (syrinx):
- **Carrier Node**: Generates core pitch ($1.8\text{ kHz} - 6.5\text{ kHz}$).
- **Modulator Node**: Applies fast pitch modulation ($20\text{ Hz} - 120\text{ Hz}$) for trills and chirps.
- **Formant Filter**: Dynamic `BiquadFilterNode` (`type="bandpass"`, $Q=8.0$) tracking pitch contours to impart biological resonance.
- **Micro-Jitter**: Every call instance injects $\pm 1.5\%$ pitch variation, $\pm 5\%$ envelope duration variance, and randomized harmonic balance so no vocalization is ever identical.

### 8.2 Chorus Interaction & Avoidance
- Each bird's vocalization schedule follows a Poisson point process with rate parameter $\lambda$ derived from `vocal_frequency`.
- To prevent acoustic crowding:
  - If Bird A starts calling, adjacent birds defer scheduled calls by $500\text{ms} + \text{random}(1500\text{ms})$ unless triggered as a reciprocal chorus response.
  - High `social_warmth` birds have a $40\%$ probability of answering another bird's call within a 300–800ms window.
  - The chorus accommodates up to seven distinct call signatures before spatial and frequency masking limits recognizability.

### 8.3 Listen-In Mix Dynamics
- When the user focuses Bird $k$:
  - Focused bird gain ramps smoothly from $0\text{ dB}$ to $+4\text{ dB}$ over $1.2$ seconds using an equal-power crossfade curve.
  - All other bird channels ramp down from $0\text{ dB}$ to $-14\text{ dB}$ over $1.2$ seconds (preserving presence without total muting).
  - Ambient noise bus attenuates from $-18\text{ dB}$ to $-24\text{ dB}$.
- On disengage, all buses ramp back to nominal levels over $1.5$ seconds.

### 8.4 WebAudio Graceful Fallback
- If `AudioContext` initialization fails, is blocked by browser autoplay policies, or is unsupported:
  - The system enters **Graceful Silence Mode**.
  - No synthetic error banner is shown to the user.
  - Call captions are automatically enabled and rendered adjacent to vocalizing birds.
  - Pre-recorded sample fallbacks are strictly prohibited to prevent bundle bloat and preserve product integrity.

---

## 9. Accessibility Surfaces

Accessibility is implemented as a primary aesthetic surface rather than an afterthought.

### 9.1 Screen-Reader Narration Engine
- An invisible ARIA live region (`<div aria-live="polite" aria-atomic="true" class="sr-only">`) receives naturalist prose updates.
- **Cadence**: Idle narration updates occur every 30–60 seconds.
- **Tone**: Prose matches the naturalist field notebook register (lowercase, present-tense, observational):
  > *a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.*
- **Event Priority**: User actions (return-greeting, offer acceptance, settle) generate immediate priority narration entries without interrupting screen-reader speech queues with raw state announcements.

### 9.2 Procedural Call Captions
- Subtitle pills appear floating adjacent to calling birds, synchronized with the audio envelope.
- Text is generated directly from the active call motif grammar:
  - `"a soft three-note rise"`
  - `"a low trill, paused, low trill again"`
  - `"a single sharp call from the back perch"`
- Captions fade in over 200ms and fade out over 600ms upon call completion.

### 9.3 Keyboard Navigation & Visual Focus
- Logical Tab sequence:
  1. Top Bar: Settings $\to$ Accessibility $\to$ Notebook $\to$ Offer Gift $\to$ Settle.
  2. Aviary Canvas: Focuses the front-most bird.
  3. Arrow keys (`Left` / `Right` / `Up` / `Down`) shift focus between birds based on 2D scene proximity.
  4. `Enter` / `Space`: Engages listen-in on focused bird.
  5. `Escape`: Disengages listen-in or closes open modals.
- **Focus Rings**: Dual-stroke focus outline (inner white $2\text{px}$, outer dark slate $2\text{px}$) guaranteeing a minimum $4.5:1$ contrast ratio against all diurnal sky conditions (dawn, midday, dusk, midnight).

### 9.4 Contrast & Typography
- All UI chrome, navigation labels, and modal dialogs conform to WCAG AA contrast standards ($> 4.5:1$ for normal body text, $> 3:1$ for large text).
- Top bar chrome automatically adjusts text and icon contrast when background diurnal lighting shifts.

---

## 10. Performance Budgets & Observability

### 10.1 Hard Performance Budgets
| Metric | Budget Target | Measurement Condition | Enforcement Mechanism |
| :--- | :--- | :--- | :--- |
| **Initial JS Bundle Size** | $< 2.0\text{ MB}$ (Gzipped) | Fresh cold load | Webpack/Vite bundle analyzer in CI; build fails if exceeded. |
| **Time to First Bird (TTFBird)** | $< 500\text{ ms}$ | Mid-tier mobile device over 4G | Inline bootstrap snapshot in initial HTML payload; critical path CSS. |
| **Animation Frame Rate** | $60\text{ fps}$ continuous | 5-year-old laptop (Intel i5 8th Gen) | Lightweight canvas draw calls; zero GC allocation inside render loop. |
| **Memory Growth** | $0.00\text{ MB}$ net leak | Over a continuous 30-minute session | Bounded audio node pools; reusable canvas scratch buffers; leak tests in CI. |
| **Simulation Tick Latency** | $\text{p99} < 5.0\text{ s}$ | Server worker processing 10,000 aviaries | Prometheus histogram alarm triggers if worker tick exceeds 5s. |

### 10.2 Observability & Privacy Boundary
- **Allowed Telemetry**:
  - Operational aggregates: HTTP status rates, edge cache hit ratios, database connection pool saturation, worker tick duration histograms.
  - Client technical health: AudioContext initialization failure rates, WebGL context loss count, aggregate session duration buckets ($<5\text{m}$, $5\text{-}15\text{m}$, $>15\text{m}$).
- **Strict Architectural Wall**:
  - Telemetry payloads must never include `account_id`, `bird_id`, personality trait values, interaction event details, or notebook content.
  - Telemetry pipeline operates on a physically distinct network egress path with independent credentials.

---

## 11. Rollout & Phased Deployment Strategy

### 11.1 Deployment Phases
- **Phase 1: Audio & Engine Verification (Weeks 1–4)**
  - Validate WebAudio procedural syrinx synthesis across Chrome, Safari, Firefox, and Edge.
  - Run synthetic 1-year simulations in accelerated headless environments to verify personality drift stability and monotonic convergence.
- **Phase 2: Closed Internal Dogfooding (Weeks 5–8)**
  - Deploy with 2 starter birds to internal cohort.
  - Calibrate presence accounting activity idle windows (tuning the 3-minute inactivity threshold).
  - Verify multi-device sync behavior between desktop and mobile browsers.
- **Phase 3: Quiet Alpha Cohort (Weeks 9–12)**
  - Expand to small external cohort using magic-link sign-in.
  - Validate visit invitation flows and verify zero cross-contamination of host drift.
- **Phase 4: General Availability v1 (Week 13)**
  - Launch public access with strict adherence to the 2-to-7 bird age progression schedule.

### 11.2 Day-One Operational Dashboards
- Simulation worker tick duration distribution (p50, p95, p99).
- Magic-link email delivery latency and bounce rates.
- Client-side WebAudio initialization success vs. silent fallback rate.
- Database replication lag between primary and snapshot read replicas.

---

## 12. Technical Risks & Mitigation Strategies

| Risk Category | Failure Scenario | Impact | Engineering Mitigation Strategy |
| :--- | :--- | :--- | :--- |
| **Drift Calibration** | Personality values drift too quickly, making birds feel erratic, or too slowly, feeling like a static screensaver. | Affective bond breaks; product feels like either a Tamagotchi or a dead app. | Automated CI drift test suites simulating 30, 90, and 360 days of user presence; strict mathematical clamping of $\Delta P$ per tick. |
| **Multi-Device Race Conditions** | User interacts on phone and laptop simultaneously, causing split-brain personality states. | Personality vector corrupted or desynchronized. | Single-writer architecture: clients submit only atomic events; server tick resolves events sequentially. Zero last-write-wins. |
| **Audio Uncanniness & Fatigue** | Procedural calls sound harsh, synthetic, or repetitive over long listening sessions. | User mutes audio, collapsing the core emotional affordance. | Dual-stage formant filtering modeled on biological avian acoustics; mandatory randomized micro-pitch and timing jitter on every note. |
| **Accessibility Regressions** | Screen-reader live regions flood the assistive technology queue with high-frequency updates. | User is forced to silence screen reader; accessibility contract fails. | Strict throttling of live region updates (30–60s idle cadence); naturalist prose generation prioritizing semantic stillness. |
| **Presence Invalidation** | Background tabs or forgotten browser windows accumulate false presence time. | Birds drift without genuine attention, corrupting the emotional bond. | Triple-conjunction verification (`visibilityState` + `hasFocus` + recent user input within 180s). Background tabs immediately halt presence emission. |
| **Voice Inconsistency** | Developers introduce celebratory toasts or gamified modals during feature extensions. | Violates "Notice, never announce" and breaks the naturalist tone. | Lint rules blocking common toast libraries; strict separation of naturalist prose (product surface) and matter-of-fact text (system errors). |
