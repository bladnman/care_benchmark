# Pocket Aviary — Comprehensive Engineering Implementation Plan (v1)

This engineering implementation plan specifies the complete technical design and execution roadmap for Pocket Aviary v1. It translates the product specification into an executable architecture that can be implemented by an engineering team without further clarification.

---

## 1. Scope & Boundary Enforcement

### 1.1 In-Scope for v1
- **Platform**: Modern web browsers (last two major versions of Chrome, Safari, Firefox, Edge).
- **Core Experience**:
  - Single horizontal aviary scene fitting viewport without scrolling, panning, or zooming.
  - Three depth perch zones: Front, Middle, Back.
  - Two starter birds at onboarding chosen automatically from a 6-species pool; user naming during adoption and subsequent renaming.
  - Age-paced unlocking of additional bird adoption slots up to a strict cap of seven birds.
  - Continuous day/night lighting cycle tied to user local timezone.
  - Rare subtle ambient weather (gentle rain, soft wind) and micro-motion (procedural drifting leaves/feathers, subtle parallax).
  - Procedural micro-motion for birds reflecting mood and personality.
  - First-frame aliveness: rendering birds mid-action with ambient motion and sound already active; quiet sky field for initial network fetch.
- **Interactions**:
  - **Presence Tracking**: Strict conjunction of `document.visibilityState === 'visible'`, window focus, and recent pointer/keyboard activity window.
  - **Return-Greeting**: Staggered, procedural greeting from one bird within 1–2 seconds based on absence duration, boldness, and mood; zero welcome toasts or banners.
  - **Listen-In**: Smooth audio focus on a selected bird (raising call volume, ducking other birds to ambient floor without complete muting).
  - **Offers**: Top-bar gestures (seed, song-fragment, still water pool) with per-bird cooldowns and mood/curiosity-dependent reactions.
  - **Settle Gesture**: Opt-in gentle evening fade and call dampening with a 5-second click-to-undo window; equivalent to tab close at the engine level.
  - **Field Notebook**: Sparsely generated naturalist prose observations (~1 entry every few days) in read-only modal/panel.
- **Social (Quiet Visits)**:
  - Email-based single-use invitation links (30-day expiration, instant host revocation).
  - Pure read-only ambient viewing of host's canonical aviary (no visitor interactions, no visitor presence counted, zero co-presence, no chat/comments/avatars).
  - On-demand host visit log in settings; visit notifications off by default.
- **Accounts & Multi-Device Sync**:
  - Email magic link authentication (15-minute token validity, single-use).
  - Single canonical aviary per account driven by server-side authoritative simulation tick (~60s).
  - Multi-device consistency through server-authored additive deltas; zero client-side simulation or last-write-wins collisions on personality state.
  - Account data JSON export and 30-day soft deletion lifecycle.
- **Accessibility & Performance**:
  - Screen-reader naturalist running prose narration (30–60s cadence; priority bump for user events).
  - Procedural call captions in matching naturalist voice.
  - Reduced-motion mode replacing continuous animations with slow cross-fading poses and removing drifting leaf particles.
  - Full keyboard accessibility and WCAG AA contrast compliance across chrome.
  - Initial JS bundle <2MB (gzipped), time-to-first-bird <500ms, 60fps steady rendering on 5-year-old hardware, zero memory growth over 30 minutes.

### 1.2 Explicit Non-Goals (Hard Architectural Boundaries)
- **No Native Mobile Apps**: No React Native, Flutter, Swift, or Kotlin codebases; no hybrid app wrappers.
- **No Gamification / Engagement Mechanics**: Strictly no streak counters, day-count badges, level-up confetti, achievements, green-dot visit grids, or progress bars. No database tables or analytics tracking visit frequency.
- **No Custodial / Tamagotchi Mechanics**: Birds never die, starve, fall ill, or show distress. Absence produces ambient quietness, never negative trait drift.
- **No Social Network Surfaces**: Strictly no public directory, user profiles, following, leaderboards, global feeds, or public bird showcases.
- **No PII Leakage / Data Monetization**: Synthetic UUIDs isolate all internal systems; per-bird interactions are never exported to data lakes, shared with third parties, or used for model training.

---

## 2. Architecture & Service Topology

The system separates into two decoupled boundaries: a deterministic, server-side simulation and persistence engine, and a low-latency, WebGL/WebAudio client.

```
+-----------------------------------------------------------------------------------+
|                                 CLIENT (Browser)                                  |
|                                                                                   |
|  +--------------------+   +---------------------+   +--------------------------+  |
|  |   HTML/Canvas/CSS  |   |   WebAudio Engine   |   |   Accessibility Layer    |  |
|  | - 2D Canvas / WebGL|   | - Procedural Synth  |   | - ARIA Live Narration    |  |
|  | - Micro-motion     |   | - Spatial / Chorus  |   | - Dynamic Captions       |  |
|  | - Viewport scaling |   | - Mix Bus & Ducking |   | - Keyboard Navigation    |  |
|  +---------^----------+   +----------^----------+   +------------^-------------+  |
|            |                         |                           |                |
|            +-------------------------+---------------------------+                |
|                                      |                                            |
|                        +-------------v-------------+                              |
|                        |     Client State Sync     |                              |
|                        | - Snapshot Interpolator   |                              |
|                        | - Presence Monitor        |                              |
|                        +-------------^-------------+                              |
+--------------------------------------|--------------------------------------------+
                                       | HTTPS (REST / SSE / WebSockets)
+--------------------------------------v--------------------------------------------+
|                                  EDGE / API GW                                    |
|  - TLS Termination & HTTP/2                                                       |
|  - Magic Link Auth Validation (JWT Session Cookies)                               |
|  - Rate Limiting & PII Redaction                                                  |
+--------------------------------------^--------------------------------------------+
                                       |
+--------------------------------------v--------------------------------------------+
|                             APPLICATION BACKEND                                   |
|                                                                                   |
|  +-----------------------+  +----------------------+  +------------------------+  |
|  |   Event Ingestion     |  | Simulation Worker    |  | Query & Snapshot Svc   |  |
|  | - Append-only log     |  | - Autorun Tick (~60s)|  | - Cached Aviary State  |  |
|  | - Presence aggregates |  | - Drift & Mood logic |  | - Read-only Visit View |  |
|  | - Offer/Listen events |  | - Notebook generator |  | - Account Export/Delete|  |
|  +-----------+-----------+  +----------+-----------+  +------------+-----------+  |
|              |                         |                           |              |
+--------------|-------------------------|---------------------------|--------------+
               |                         |                           |
+--------------v-------------------------v---------------------------v--------------+
|                            PERSISTENCE TIER                                       |
|                                                                                   |
|  +--------------------------+               +----------------------------------+  |
|  |    PostgreSQL Primary    |               |         Redis Cluster            |  |
|  | - Accounts (Encrypted)   |               | - Active Aviary Snapshots        |  |
|  | - Aviaries & Birds       |               | - Presence Leases & Rate Limits  |  |
|  | - Interaction Events Log |               | - SSE Pub/Sub for Live Sync      |  |
|  | - Field Notebook Entries |               +----------------------------------+  |
|  | - Visit Invitations/Logs |                                                     |
|  +--------------------------+                                                     |
+-----------------------------------------------------------------------------------+
```

### 2.1 Component Separation
1. **API Gateway & Auth Service**: Handles magic link dispatch, token verification, route access control, and visitor token validation.
2. **Event Ingest Service**: Receives client interaction events and validated presence heartbeat batches; writes append-only records into PostgreSQL.
3. **Simulation Tick Worker**: Background daemon processing active and dormant aviaries every 60 seconds. Reads event queues, advances mood states, computes personality vector deltas, applies age-based adoption eligibility, generates field notebook entries, and saves new canonical snapshots.
4. **Snapshot & State Distribution Service**: Delivers lightweight JSON state snapshots upon initial load, tab visibility restoration, or SSE stream updates.
5. **Client Engine**:
   - **Scene Renderer**: Canvas2D/WebGL rendering responsive horizontal scenery, perch zones, day/night lighting shaders, and ambient leaf/feather particles.
   - **Audio Synthesizer**: WebAudio graph generating procedural bird calls and environmental sound beds with dynamic ducking and fallback handling.
   - **Accessibility & Narration Controller**: Dispatches naturalist text descriptions to ARIA live regions and synchronized call captions.

---

## 3. Data Model & Storage Schema

PostgreSQL schema ensures strict entity isolation using synthetic UUIDs and monotonic numerical limits.

### 3.1 Relational Schema Definitions

```sql
-- Accounts table: Email stored encrypted; internal operations strictly use account_id
CREATE TABLE accounts (
    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_hash VARCHAR(64) NOT NULL UNIQUE, -- For lookup without decrypting
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    settled_at TIMESTAMPTZ NULL,
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    deletion_requested_at TIMESTAMPTZ NULL, -- Soft delete timestamp
    notify_on_visit BOOLEAN NOT NULL DEFAULT FALSE
);

-- Aviaries table: One canonical aviary per account
CREATE TABLE aviaries (
    aviary_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(account_id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    version BIGINT NOT NULL DEFAULT 1
);

-- Birds table: Stable bird identity, personality vector, and current mood
CREATE TABLE birds (
    bird_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL, -- e.g., 'song_sparrow', 'mourning_dove', 'goldfinch'
    name VARCHAR(40) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    slot_index INT NOT NULL CHECK (slot_index BETWEEN 0 AND 6),
    
    -- Hidden personality vector (Normalized 0.000 to 1.000, never exposed to user)
    boldness NUMERIC(5, 4) NOT NULL DEFAULT 0.3000 CHECK (boldness BETWEEN 0.0 AND 1.0),
    social_warmth NUMERIC(5, 4) NOT NULL DEFAULT 0.3000 CHECK (social_warmth BETWEEN 0.0 AND 1.0),
    vocal_frequency NUMERIC(5, 4) NOT NULL DEFAULT 0.3000 CHECK (vocal_frequency BETWEEN 0.0 AND 1.0),
    plumage_saturation NUMERIC(5, 4) NOT NULL DEFAULT 0.3000 CHECK (plumage_saturation BETWEEN 0.0 AND 1.0),
    curiosity NUMERIC(5, 4) NOT NULL DEFAULT 0.3000 CHECK (curiosity BETWEEN 0.0 AND 1.0),
    
    -- Fast-timescale mood state
    current_mood VARCHAR(20) NOT NULL DEFAULT 'content' 
        CHECK (current_mood IN ('wary', 'content', 'curious', 'drowsy', 'alert')),
    mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    -- Visual & Spatial State
    current_perch_zone VARCHAR(10) NOT NULL DEFAULT 'middle'
        CHECK (current_perch_zone IN ('front', 'middle', 'back')),
    perch_slot INT NOT NULL DEFAULT 0,
    
    CONSTRAINT uq_aviary_slot UNIQUE(aviary_id, slot_index)
);

-- Append-only interaction event log consumed by simulation tick
CREATE TABLE interaction_events (
    event_id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(bird_id) ON DELETE CASCADE,
    event_type VARCHAR(32) NOT NULL, -- 'presence_ping', 'listen_in_start', 'listen_in_end', 'offer_seed', 'offer_song', 'offer_pool', 'settle'
    payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_at TIMESTAMPTZ NULL
);
CREATE INDEX idx_interaction_unprocessed ON interaction_events (aviary_id, created_at) WHERE processed_at IS NULL;

-- Field notebook entries: Sparsely generated naturalist observations
CREATE TABLE field_notebook_entries (
    entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    entry_text TEXT NOT NULL,
    recorded_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_notebook_aviary_date ON field_notebook_entries (aviary_id, recorded_at DESC);

-- Visit invitations: Read-only guest access tokens
CREATE TABLE visit_invitations (
    invitation_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    guest_email_hash VARCHAR(64) NOT NULL,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ NULL
);

-- Visit log entries: Record of guest visits
CREATE TABLE visit_logs (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invitation_id UUID NOT NULL REFERENCES visit_invitations(invitation_id) ON DELETE CASCADE,
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    visited_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    duration_seconds INT NOT NULL DEFAULT 0
);
```

---

## 4. API Surface & Protocols

All communication adheres strictly to the defined voice: errors return matter-of-fact text, while domain payloads carry pure structural data.

### 4.1 Client-to-Server Endpoints

#### Authentication
- `POST /api/v1/auth/magic-link`: Submits `{ email }`. Issues token via email. Rate-limited to 5 requests per hour per email.
- `GET /api/v1/auth/verify?token=...`: Validates 15-minute token, establishes HTTP-only secure session cookie containing signed session JWT, invalidates magic link immediately.
- `POST /api/v1/auth/logout`: Revokes active session token.

#### Aviary State & Stream
- `GET /api/v1/aviary/state`: Fetches current canonical aviary snapshot.
  - *Response*:
    ```json
    {
      "aviary_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
      "version": 4128,
      "server_time": "2026-09-06T16:11:00Z",
      "weather": { "condition": "clear", "intensity": 0.0 },
      "is_settled": false,
      "eligible_for_new_bird": false,
      "birds": [
        {
          "bird_id": "1fa85f64-5717-4562-b3fc-2c963f66afa6",
          "species_id": "song_sparrow",
          "name": "Pip",
          "perch_zone": "front",
          "perch_slot": 1,
          "mood": "curious",
          "plumage_saturation": 0.54,
          "vocal_frequency": 0.62,
          "boldness": 0.58
        },
        {
          "bird_id": "2da85f64-5717-4562-b3fc-2c963f66afa7",
          "species_id": "mourning_dove",
          "name": "Wren",
          "perch_zone": "back",
          "perch_slot": 3,
          "mood": "content",
          "plumage_saturation": 0.41,
          "vocal_frequency": 0.35,
          "boldness": 0.31
        }
      ]
    }
    ```
- `GET /api/v1/aviary/events/stream`: Optional Server-Sent Events (SSE) stream broadcasting periodic simulation tick updates and real-time settle state.

#### Interactions
- `POST /api/v1/aviary/events`: Submits client interaction events.
  - *Payload*:
    ```json
    {
      "event_type": "presence_ping", // or 'listen_in_start', 'offer_seed', etc.
      "bird_id": "1fa85f64-5717-4562-b3fc-2c963f66afa6",
      "payload": { "duration_seconds": 60 }
    }
    ```
- `POST /api/v1/aviary/settle`: Triggers session settle. Can be undone within 5 seconds via `POST /api/v1/aviary/unsettle`.
- `POST /api/v1/aviary/adopt`: Adopts an eligible bird when age threshold is reached.
  - *Payload*: `{ "name": "Fern" }`.

#### Field Notebook & Account Settings
- `GET /api/v1/notebook`: Returns paginated list of read-only observation entries.
- `POST /api/v1/social/invitations`: Generates visitor invite for friend email.
- `GET /api/v1/social/logs`: Retrieves recent visit logs.
- `DELETE /api/v1/social/invitations/:id`: Revokes visit link immediately.
- `GET /api/v1/account/export`: Initiates async compilation of JSON snapshot and emails download link.
- `POST /api/v1/account/delete`: Initiates 30-day soft deletion.

#### Visitor Surface
- `GET /api/v1/visit/:token`: Validates guest token; returns read-only aviary snapshot. Rejects all interaction event post attempts with `403 Forbidden: This visit is read-only.`

---

## 5. Simulation Engine Design

The simulation engine is authoritative, executing on a server-side cron/worker daemon every 60 seconds (`TICK_INTERVAL = 60s`).

### 5.1 Presence Calculation Pipeline
1. Client monitors:
   - `document.visibilityState === 'visible'`
   - `window.hasFocus() === true`
   - Active user input event (`pointermove`, `keydown`, `touchstart`) within trailing window `ACTIVITY_WINDOW = 180s` (3 minutes).
2. While all three conditions hold, client transmits a heartbeat `presence_ping` every 30 seconds.
3. If any condition fails, transmission halts immediately.
4. Worker aggregates `presence_time` by summing non-overlapping validated ping intervals.

### 5.2 Drift Function Calibration
Drift transforms cumulative presence and interaction signals into personality vector adjustments using a low-pass filter:

$$\Delta T_i = \alpha_i \cdot \left( \sum w_k E_k \right) \cdot (1.0 - T_i)$$

Where:
- $T_i$ is the trait scalar in $[0.0, 1.0]$.
- $(1.0 - T_i)$ enforces diminishing returns as traits approach ceiling.
- Monotonic constraint: $\Delta T_i \ge 0.0$ always. Neglect ($E_k = 0$) results in $\Delta T_i = 0$, guaranteeing traits never decrease.

#### Weight Matrix & Calibration Targets
- **Presence-time**: Primary input. 1 hour of validated presence adds $+0.003$ raw trait potential distributed across traits.
- **Listen-in**: $+0.005$ to `social_warmth` and `vocal_frequency` per 5 minutes of focused listening.
- **Offers**: $+0.004$ to `curiosity` on offer accept; $+0.002$ to `boldness`.
- **Target Velocity**:
  - **7 Days (Instrumental Drift)**: 15 minutes/day $\approx$ 1.75 hours/week yields $+0.015$ to $+0.025$ delta across active traits. Detectable by automated assertions; imperceptible to casual eye.
  - **21 Days (Visible Drift)**: $\sim$5 hours total presence yields $+0.07$ to $+0.12$ shift. Bird visibly perches on front branch 35% more often; plumage saturation increases noticeably; calls occur with higher cadence.

### 5.3 Mood Transition Finite State Machine (FSM)
Mood is a fast-timescale state updated on each tick:
- States: `wary`, `content`, `curious`, `drowsy`, `alert`.
- Transition Factors:
  - **Diurnal Clock**: Local time mapping:
    - 05:00–09:00: Tendency toward `alert` (probability 0.50).
    - 09:00–17:00: Tendency toward `content` (0.60) or `curious` (0.30).
    - 17:00–21:00: Tendency toward `drowsy` (0.70).
    - 21:00–05:00: Most birds `drowsy` (settled/asleep); nocturnal species remain `alert` or `content`.
  - **Recent Session Interactions**:
    - Offer accepted in last 5m: shifts `wary` $\to$ `curious` $\to$ `content`.
    - Extended listen-in: nudges toward `content`.
  - **Ambient Events**: Passing rain temporarily damps calls and nudges `curious` $\to$ `content` or `drowsy`.
  - **Social Contagion**: If a low-boldness bird enters `wary` and issues an alarm chirp, adjacent birds have a 40% probability of shifting to `wary` for 3–5 minutes.
  - **Absence Persistence**: When tab is closed, mood continues to cycle with diurnal rhythm on server tick. Re-opening tab reveals the current real-time mood without artificial resets.

### 5.4 Procedural Call-Grammar Runtime
Each species possesses a library of 4–8 basic audio motifs (micro-chirps, glides, descending whistles, trills):
- A procedural call combines 1 to 3 motifs with stochastic variations in pitch ($\pm 5\%$), duration ($\pm 8\%$), and interval spacing.
- Grammar rules determine syntax based on mood:
  - `alert`: Sharp, high-frequency, single-motif call.
  - `content`: Multi-motif descending phrase with soft vibrato.
  - `curious`: Upward inflection glide at end of motif.
  - `drowsy`: Low-amplitude, extended intervals between single notes.
- **Chorus Generator**: When a bird vocalizes, other birds evaluate:
  $$\text{Response Probability} = \text{social\_warmth} \times \text{vocal\_frequency} \times \text{mood\_factor}$$
  If triggered, the second bird issues an answering call with a randomized humanized stagger (350ms to 1100ms).

### 5.5 Field Notebook Generation Engine
The simulation tick evaluates aviary history every 24 hours:
- Rule: Sparsity enforcement — maximum 1 entry per 3–5 days unless an inflection event occurs (e.g., first time a bird perches in front zone, or unusual greeter priority).
- Generator uses naturalist grammar templates compiled into present-tense, lowercase observations:
  - Example template: `"{bird_a} greeted before {bird_b} today, first time this week."`
  - Example template: `"{bird} is fluffed against the cool air, watching the back perch. low calls only."`
- Entries are saved directly to `field_notebook_entries` as immutable historical records.

---

## 6. Multi-Device Synchronization & Conflict Prevention

### 6.1 Strict Single-Writer Architecture
- **Server as Sole Author**: The client never mutates state. The client only generates event logs (`interaction_events`).
- **No Last-Write-Wins (LWW)**: Personality traits and mood cannot be overwritten by conflicting clients because no client has permission to send vector values. If Client A (laptop) and Client B (phone) are open simultaneously, both stream presence pings and interaction events to the server. The server appends both to the event log in chronological sequence, and the simulation tick aggregates deltas additively.
- **Snapshot Interpolation**:
  - The client maintains a local rendering state buffer with snapshots $S_t$ and $S_{t+1}$.
  - When a new snapshot arrives, bird perch transitions glide via smooth bezier interpolation over 1200ms rather than snapping.

### 6.2 Settle State & Visibility Transitions
- When settle is triggered, the server marks `settled_at = NOW()` on the aviary.
- Connected clients receive an SSE event (or poll update) and initiate the 4-second warm sunset lighting fade.
- Tab restoration: If a tab becomes visible after being backgrounded or computer wake, it immediately issues a `GET /api/v1/aviary/state` to retrieve the latest state, re-anchoring positions smoothly.

---

## 7. Frontend Rendering Pipeline

### 7.1 Scene Composition & Canvas Architecture
- Implemented in a single lightweight `<canvas>` driven by Canvas2D or lightweight WebGL context.
- **Scene Layering (Back to Front)**:
  1. *Sky Layer*: Procedural gradient background reflecting local solar elevation (dawn pastel $\to$ midday soft cyan $\to$ golden sunset $\to$ deep navy dusk).
  2. *Background Layer*: Distant soft-focus tree canopies with slow parallax offset ($0.02 \times$ pointer movement).
  3. *Back Perch Zone*: Back branches, ambient resting birds.
  4. *Middle Perch Zone*: Primary branch system, perching birds.
  5. *Front Perch Zone*: Foreground railing, visiting bold birds, offered water pool/seeds.
  6. *Particle Layer*: Client-side ambient leaves/feathers drifting at randomized intervals.
- **Responsive Viewport Fitting**:
  - The scene maintains a fixed 16:9 bounding anchor that auto-scales dynamically with CSS `contain: strict`.
  - On narrow mobile viewports, the scene centers horizontally and perch spacing contracts without horizontal scrollbars, ensuring 100% bird visibility.

### 7.2 Idle Micro-Motion & Procedural Animation
- Zero pre-rendered video or heavy GIF/spritesheet loops. Birds are composed of hierarchical procedural SVG/2D-path segments (body, head, beak, wing, tail).
- **Idle Motion Generators**:
  - *Breathing*: Rhythmic sinusoidal scale expansion (0.98 to 1.02) on bird torso (period 2.4s).
  - *Head Cock*: Stochastic Poisson timer triggering subtle $15^\circ$ head rotation and glance toward viewer or other birds.
  - *Tail Bob*: Periodic damped harmonic oscillation triggered after perching or vocalizing.
  - *Preening*: Procedural wing ruffle and head-to-flank rotation when mood is `content`.
- **First-Frame Aliveness Guarantee**:
  - Engine initializes procedural phase timers using `Date.now() % PERIOD_MS`.
  - The very first frame drawn has birds in the middle of natural breathing or preening cycles. No static start poses.

### 7.3 Reduced-Motion Implementation
When `window.matchMedia('(prefers-reduced-motion: reduce)')` is true or user activates reduced-motion in accessibility settings:
- Frame-by-frame skeletal movement and procedural micro-motion are disabled.
- Birds render in static, composed naturalist poses.
- Perch transitions replace flight curves with an 800ms gentle cross-fade between initial and destination poses.
- Ambient leaf and feather particles are completely culled.
- Diurnal lighting transitions remain but are slowed to 10-second subtle cross-fades.

---

## 8. Audio Pipeline & Synthesis Architecture

```
+-------------------------------------------------------------------------------+
|                             WebAudio Context Bus                              |
|                                                                               |
|  +--------------------+                                                       |
|  | Bird Synth Node 1  |--+                                                    |
|  | (Pip - Custom FM)  |  |                                                    |
|  +--------------------+  |                                                    |
|                          |    +-------------------+    +-------------------+  |
|  +--------------------+  +--->| Bird Gain Node 1  |--->|   Mixer Channel   |  |
|  | Bird Synth Node 2  |  |    +-------------------+    | (Listen-in Focus) |  |
|  | (Wren - Dual Osc)  |--+                             +---------+---------+  |
|  +--------------------+  |    +-------------------+              |            |
|                          +--->| Bird Gain Node 2  |              |            |
|  +--------------------+  |    +-------------------+              |            |
|  | Environmental Bed  |--+                                       v            |
|  | (Wind / Soft Rain) |-------> [Ambient Gain] -------------> Master Bus      |
|  +--------------------+                                          |            |
|                                                                  v            |
|                                                            AudioDestination   |
+-------------------------------------------------------------------------------+
```

### 8.1 Procedural WebAudio Call Synthesis
- Sound generation uses native WebAudio API (`AudioContext`) without external sound sample files.
- Each bird species uses a tailored synthesis graph:
  - Dual band-limited oscillators (Sine + gentle Triangle) modulated by a fast pitch envelope (exponential frequency sweep).
  - FM (Frequency Modulation) synthesis node for species with warbles or trills.
  - Low-pass filter (BiquadFilterNode) softening harsh digital harmonics, providing organic resonance.
  - Amplitude envelope (GainNode) with shaped attack, decay, sustain, and release curves.

### 8.2 Listen-In Audio Mixing & Ducking
- **Default State**: All active birds mix to the master bus with natural spatial panning based on horizontal perch coordinate. Master volume sits at a calm ambient level (-18dB).
- **Listen-In Engagement**:
  - When user clicks/focuses Bird $X$, its dedicated `GainNode` smoothly ramps up by $+6\text{dB}$ over an exponential 800ms transition.
  - All other bird channels ramp down by $-12\text{dB}$ over 800ms to a quiet background bed (never fully muted).
  - High-frequency ambient environment gently rolls off.
- **Listen-In Disengagement**: All channels smoothly interpolate back to equal ambient levels over 1200ms.

### 8.3 WebAudio Fallback
- If `window.AudioContext` fails to initialize (unsupported browser, strict autoplay restriction before user interaction, or hardware failure):
  - The application enters graceful silent mode.
  - Procedural call captions are automatically enabled by default.
  - Strictly no recorded audio fallbacks are loaded, preserving the <2MB bundle budget and preventing canned audio artifacts.

---

## 9. Accessibility Surfaces

Pocket Aviary treats accessibility as a primary aesthetic surface rather than an compliance checklist.

### 9.1 Naturalist Screen-Reader Narration
- An accessible live region (`<div role="status" aria-live="polite" class="sr-only">`) continuously receives observational prose updates.
- Narration is updated at a gentle 30–60 second cadence during idle watching:
  - *"a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."*
- User-triggered events (e.g., offering a seed, selecting listen-in) get a prioritized update:
  - *"pip hops down to the front perch and investigates the offer, head tilted."*
- Strictly avoids announcement style (never outputs *"Button clicked"* or *"State updated: Pip content"*).

### 9.2 Real-time Procedural Call Captions
- When captions are toggled on, an unobtrusive floating caption element appears adjacent to the vocalizing bird.
- Caption strings are generated directly from the call-grammar runtime matching the procedural motifs:
  - *"a soft three-note rise"*
  - *"a low trill, paused, low trill again"*
  - *"a single sharp call from the back perch"*
- Captions gently fade in over 200ms and fade out 1000ms after the call completes.

### 9.3 Keyboard Navigation & Visual Focus
- `Tab` moves focus sequentially through the top-bar icons (Offer, Notebook, Accessibility, Account).
- Pressing `Tab` into the aviary focuses the first bird.
- `ArrowLeft` and `ArrowRight` navigate focus among active birds based on spatial perch position.
- `Enter` or `Space` toggles listen-in on the focused bird. `Escape` disengages listen-in.
- Focus indicator renders as a subtle, high-contrast double ring conforming to bird silhouette, exceeding WCAG AA 3:1 contrast ratio against both daylight and night backgrounds.
- All modal dialogs (Field Notebook, Settings) feature strict focus-trapping and return focus to triggering elements on dismiss.

---

## 10. Performance Budgets & Observability

### 10.1 Hard Performance Budgets
| Metric | Budget Ceiling | Measurement & Enforcement Strategy |
|---|---|---|
| **Initial Gzipped JS Bundle** | $< 2.0\text{ MB}$ | Enforced in CI build via Webpack/Vite bundle analyzer. Split non-critical chunks. |
| **Time-to-First-Bird (TTFBird)** | $< 500\text{ ms}$ | Measured from navigation start to initial canvas render on mid-tier mobile over 4G. |
| **Steady-state Framerate** | $60\text{ fps}$ | Measured over 30-minute idle sessions on 5-year-old baseline laptop hardware. |
| **Memory Growth** | $0.0\text{ MB} \text{ leak}$ | Heap snapshot diffing in headless Chromium after 30-minute active sessions. |
| **Authoritative Tick Latency** | $p99 < 5.0\text{ s}$ | Background simulation worker batch processing deadline. |

### 10.2 Bundle Budget Engineering
- Zero heavy 3D engines (no three.js or Babylon.js). Custom lightweight 2D canvas renderer.
- Zero audio asset bloat. 100% procedural WebAudio synthesis saves dozens of megabytes of audio samples.
- Code-splitting: Account settings, visit management, and export dialogs are loaded as dynamic imports (`import()`) only when opened.

### 10.3 Privacy-Preserving Observability
- Strict boundary between operational metrics and bird simulation data:
  - **Permitted Operational Telemetry**: Aggregated HTTP request counts, endpoint latencies, client render FPS percentiles, audio error counts, simulation tick durations.
  - **Prohibited Telemetry**: User IDs tied to bird names, per-bird personality vectors, session interaction sequences, or visitor graphs.
- Telemetry collectors discard account UUIDs and sanitize payloads before sending to logging backends.

---

## 11. Rollout & Aviary Scaling Strategy

### 11.1 Progressive Delivery Phases
1. **Phase 0 — Internal Dogfooding (2 Birds)**: Verify drift calibration over 3 weeks of continuous usage across internal team devices. Calibrate presence detection window.
2. **Phase 1 — Closed Alpha**: Magic-link authentication, baseline aviary with 2 starter birds, full audio chorus, field notebook generation.
3. **Phase 2 — Beta with Social Visits**: Introduce opt-in visitor links and multi-device sync validation across cross-device sessions.
4. **Phase 3 — General Availability**: Full rollout with species unlocks enabled.

### 11.2 Bird Count Ramp-Up (Age-Paced Progression)
To preserve recognizability and emotional connection, aviaries scale strictly by account age:
- **Day 1 (Adoption)**: 2 Starter Birds.
- **Month 2 (~60 days)**: 3rd Bird adoption unlocked.
- **Month 4 (~120 days)**: 4th Bird adoption unlocked.
- **Month 7 (~210 days)**: 5th Bird adoption unlocked.
- **Month 10 (~300 days)**: 6th Bird adoption unlocked.
- **Month 13 (~390 days)**: 7th Bird adoption unlocked (Permanent Cap).
- *Rationale*: Rejects gamified task-completion unlocks; honors the slow deepening of an observational relationship.

---

## 12. Risk Management & Failure Modes

| Risk Description | Potential Impact | Prevention & Mitigation Strategy |
|---|---|---|
| **Drift Calibration Drift-Too-Fast** | Users notice traits shifting session-to-session; feels like a Tamagotchi widget. | Strict unit & integration tests asserting mathematical bounds over simulated 7-day and 21-day event streams. Low-pass filter smoothing. |
| **Ghost Presence Leaks** | Background tab counts presence; birds drift while user is away; calibration ruins. | Triple-conjunction verification (`visibilityState` + window focus + active input within 3m). Server rejects heartbeats failing window constraints. |
| **Multi-Device Divergence** | Conflicting writes between phone and laptop destroy personality history. | Strictly no client-authored vectors or LWW. Server append-only interaction log with authoritative server tick computation. |
| **Procedural Audio Uncanniness** | Synthesized calls sound harsh, robotic, or grating over long sessions. | Micro-pitch dithering, organic FM modulation, low-pass resonance filtering, and maximum chorus limiter. Silent fallback with captions if audio fails. |
| **Accessibility Degradation** | Screen-reader narration becomes spammy or robotic, ruining affective experience. | Pacing narration to 30–60s intervals in naturalist prose; dedicating screen-reader testing in CI as a release-blocking quality gate. |
| **Memory Leaks in Audio/Canvas** | WebAudio nodes or detached DOM elements cause tab crashes during long open sessions. | Buffer reuse, explicit AudioNode disconnection on note completion, and CI automated 30-minute memory leak profiling. |

---
