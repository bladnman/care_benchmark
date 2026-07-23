# Pocket Aviary — Phase 1 Engineering Implementation Plan

## 1. Scope & System Boundaries

### 1.1 In-Scope Features for v1
- **Web-Only Client**: Support modern desktop and mobile browsers (last two major versions of Chrome, Safari, Firefox, and Edge). Zero native apps. Responsive 1-screen viewport without panning, zoom, or horizontal scroll.
- **Single-User Accounts & Magic-Link Auth**: Single-user, single-aviary per account. Passwordless authentication via 15-minute expiring single-use magic links sent via email. Per-device session tokens revocable from settings.
- **Synthetic Account Identifier (`account_id`)**: Generation of a immutable UUID v4 upon account creation. `account_id` is used across all internal database relations, API calls, server-side simulation logs, cache keys, and telemetry. Encrypted email storage (`AES-GCM-256`) restricted strictly to the auth service.
- **Bird Population & Age-Based Unlocks**:
  - Starter aviary initialized with 2 starter birds selected automatically by the system from a pool of 6 species motifs. User can rename birds at any time.
  - Strict cap of 7 birds maximum per aviary (enforced by audio recognizability limits).
  - New bird unlock pacing tied strictly to aviary age (e.g. 3rd bird offered at 30 days, 4th at 90 days, 5th at 180 days, 6th at 270 days, 7th at 365 days), avoiding any visit-count or interaction-frequency gamification.
- **Server-Side Simulation Tick**:
  - Server tick runs at a slow cadence (~60 seconds) independently of client connections.
  - Computes canonical slow-timescale personality vector drift (boldness, social warmth, vocal frequency, plumage saturation, curiosity).
  - Evaluates fast-timescale mood transitions (`WARY`, `CONTENT`, `CURIOUS`, `DROWSY`, `ALERT`).
- **Presence Accounting**:
  - Client presence tracking strictly defined as the simultaneous conjunction: `document.visibilityState === 'visible'` AND `document.hasFocus() === true` AND pointer/keyboard activity within a 3-minute sliding window.
  - Presence time is the primary input to monotonic personality drift toward expressive.
- **Session Interactions**:
  - **Return-Greeting**: Immediate procedural greeting within 1-2s of opening tab, staggered per bird, scaled by absence length and individual bird boldness/mood.
  - **Listen-In**: Focus single bird to raise its call in audio mix while others drop to an ambient floor (never muted). Exponential mix transitions over 1.2s.
  - **Offers**: Top-bar affordance to offer seed, song fragment, or still pool. Per-bird cooldown timer (3-5 minutes) prevents mashing and drift saturation.
  - **Settle**: User-initiated evening lighting shift and audio quiet, with a 5-second undo grace window. Tab closure and settle are structurally equivalent at the engine level.
  - **Field Notebook**: System-generated, read-only log of naturalist observations (sparse generation: ~1 entry every few days).
- **Optional Read-Only Visits**:
  - Host sends one-time magic link email to invite a visitor.
  - Visitor receives read-only ambient view of host's aviary state.
  - Invites default OFF, per-invite opt-in, revocable at any time, expire in 30 days.
  - No visitor co-presence, no visitor cursors, no visitor interaction capability, no visitor presence accounting in host drift.
- **Accessibility Surfaces**:
  - **Screen-Reader Narration**: Dynamic `aria-live="polite"` region updated on a slow cadence (30-60s) with naturalist prose descriptions of aviary state.
  - **Reduced-Motion Surface**: Replaces continuous animation with slow cross-fades between static poses; disables ambient leaf/feather drift particles.
  - **Procedural Call Captions**: Dynamic overlay captions near calling birds generated from procedural audio motif parameters.
  - **Keyboard Nav & Contrast**: Full focus navigation across top bar and birds; WCAG AA contrast compliance across all text/chrome.
- **Performance & Observability**:
  - Initial JS bundle < 2MB (gzipped).
  - Time to first bird visible < 500ms on 4G / mid-tier mobile.
  - 60fps idle rendering; 0 memory growth over 30 minutes.

### 1.2 Explicit Non-Goals (v1 & Future Policy)
- **No Native Applications**: No iOS/Android native wrappers or platform-specific code.
- **No Gamification**: Absolute prohibition of streak counters, "days visited" calendars, level/XP systems, badges, achievements, scores, or public rankings.
- **No Tamagotchi Mechanics**: Birds never die, starve, fall ill, or show distress. No decay of personality vectors on neglect (drift is strictly monotonic toward expressive).
- **No Social Network Features**: No public discovery directory, no global feeds, no follower graphs, no user profiles, no comment sections, no chat, and no public leaderboards.
- **No Client-Authoritative State**: Clients never calculate drift or mutate personality vectors directly.
- **No Recorded Audio Assets**: No audio file downloads (MP3/WAV/OGG loops); 100% WebAudio procedural synthesis.

---

## 2. Architecture & Service Topology

```
+-----------------------------------------------------------------------------------+
|                                 BROWSER CLIENT                                    |
|  +---------------------+   +-------------------------+   +---------------------+  |
|  |  Canvas/WebGL Render|   | WebAudio Synthesizer    |   | Presence Tracker    |  |
|  |  (Interpolation)    |   | (Motif Procedural Engine|   | (Visibility/Focus/  |  |
|  |                     |   |  + Spatial Mixer)       |   |  Activity Window)   |  |
|  +----------+----------+   +------------+------------+   +----------+----------+  |
+-------------|---------------------------|---------------------------|-------------+
              | Pull Snapshots            | Audio Parameters          | Push Events
              v                           v                           v
+-----------------------------------------------------------------------------------+
|                             EDGE & API GATEWAY                                    |
|  +-----------------------------------------------------------------------------+  |
|  | - Magic-Link Auth Verification & Token Issuance                             |  |
|  | - Endpoint Router & TLS Termination                                         |  |
|  | - Rate-Limiting & Input Schema Validation                                   |  |
|  +-------------------------------------+---------------------------------------+  |
+----------------------------------------|------------------------------------------+
                                         |
                       +-----------------+-----------------+
                       |                                   |
                       v                                   v
+------------------------------------------+ +--------------------------------------+
|            API SERVICE                   | |      SIMULATION TICK SERVICE         |
|  - Serves GET /aviary/snapshot           | |  - Runs background loop (~60s tick)  |
|  - Appends to interaction_events log     | |  - Consumes interaction_events       |
|  - Serves GET /notebook                  | |  - Calculates low-pass drift        |
|  - Handles visit invite creation/revoke  | |  - Updates fast-timescale moods    |
|                                          | |  - Generates sparse notebook entries|
|                                          | |  - Writes canonical state snapshot |
+----------------------+-------------------+ +------------------+-------------------+
                       |                                      |
                       +-----------------+--------------------+
                                         |
                                         v
+-----------------------------------------------------------------------------------+
|                                STORAGE LAYER                                      |
|  +------------------------+  +------------------------+  +---------------------+  |
|  | PostgreSQL (Primary)   |  | Redis / Key-Value      |  | Append-Only Event   |  |
|  | - accounts (encrypted) |  | - Canonical Snapshots  |  |   Store             |  |
|  | - birds & vectors      |  | - Session Tokens       |  | - presence_pings    |  |
|  | - notebook_entries     |  | - Edge Cache Keys      |  | - interaction_events|  |
|  | - visit_invitations    |  |                        |  |                     |  |
|  +------------------------+  +------------------------+  +---------------------+  |
+-----------------------------------------------------------------------------------+
```

### 2.1 Component Boundaries
1. **Client Tier**: Web app built with HTML5/TypeScript. Uses WebGL/2D Canvas for rendering and WebAudio for audio synthesis. Thin renderer model: state snapshots received via HTTP JSON are interpolated locally for 60fps rendering.
2. **API & Event Gateway**: Handles client authentication via magic-link JWTs. Exposes REST endpoints for client event submission and snapshot retrieval. Validates request signatures and enforces rate limits.
3. **Simulation Engine Service**: Autonomous background service executing a deterministic ~60-second tick per active account. Consumes unprocessed client interaction events, applies low-pass filter drift formulas, updates bird mood state machines, writes canonical bird state to PostgreSQL, and flushes fresh snapshots to Redis for low-latency client reads.
4. **Storage Layer**: PostgreSQL serves as the persistent system of record. Redis acts as a high-speed snapshot cache (<50ms reads) and session store. An append-only event store guarantees event ordering for the simulation tick.

---

## 3. Data Model & Schema Definitions

```sql
-- Schema version 1.0.0

CREATE TYPE account_status AS ENUM ('ACTIVE', 'PENDING_DELETION');
CREATE TYPE bird_mood AS ENUM ('WARY', 'CONTENT', 'CURIOUS', 'DROWSY', 'ALERT');
CREATE TYPE perch_zone AS ENUM ('FRONT', 'MIDDLE', 'BACK');
CREATE TYPE event_type AS ENUM (
  'PRESENCE_PING', 
  'OFFER_SEED', 
  'OFFER_SONG', 
  'OFFER_POOL', 
  'LISTEN_IN_START', 
  'LISTEN_IN_END', 
  'SETTLE'
);

-- Accounts Table: Internal UUID synthetic key, encrypted email
CREATE TABLE accounts (
    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL, -- AES-GCM-256 encrypted
    email_hash VARCHAR(64) UNIQUE NOT NULL, -- HMAC-SHA256 for lookup without decryption
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    aviary_created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    status account_status NOT NULL DEFAULT 'ACTIVE',
    deletion_requested_at TIMESTAMPTZ NULL,
    settings JSONB NOT NULL DEFAULT '{
        "visit_notifications_enabled": false,
        "reduced_motion_override": false,
        "captions_enabled": false
    }'::jsonb
);

-- Birds Table: Stable bird identity and hidden personality vector
CREATE TABLE birds (
    bird_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL,
    given_name VARCHAR(64) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    boldness DOUBLE PRECISION NOT NULL CHECK (boldness BETWEEN 0.0 AND 1.0),
    social_warmth DOUBLE PRECISION NOT NULL CHECK (social_warmth BETWEEN 0.0 AND 1.0),
    vocal_frequency DOUBLE PRECISION NOT NULL CHECK (vocal_frequency BETWEEN 0.0 AND 1.0),
    plumage_saturation DOUBLE PRECISION NOT NULL CHECK (plumage_saturation BETWEEN 0.0 AND 1.0),
    curiosity DOUBLE PRECISION NOT NULL CHECK (curiosity BETWEEN 0.0 AND 1.0),
    current_mood bird_mood NOT NULL DEFAULT 'CONTENT',
    last_mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    current_perch perch_zone NOT NULL DEFAULT 'MIDDLE',
    CONSTRAINT unique_bird_per_account_species UNIQUE (account_id, bird_id)
);

-- Append-Only Interaction Event Log
CREATE TABLE interaction_events (
    event_id BIGSERIAL PRIMARY KEY,
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    event_type event_type NOT NULL,
    target_bird_id UUID NULL REFERENCES birds(bird_id) ON DELETE SET NULL,
    client_timestamp TIMESTAMPTZ NOT NULL,
    received_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_by_tick BIGINT NULL
);
CREATE INDEX idx_events_account_unprocessed ON interaction_events(account_id, event_id) WHERE processed_by_tick IS NULL;

-- Field Notebook Entries
CREATE TABLE notebook_entries (
    entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    prose_content TEXT NOT NULL,
    is_read BOOLEAN NOT NULL DEFAULT FALSE
);
CREATE INDEX idx_notebook_account_time ON notebook_entries(account_id, created_at DESC);

-- Visit Invitations Table
CREATE TABLE visit_invitations (
    invite_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    visitor_email_hash VARCHAR(64) NOT NULL,
    token_hash VARCHAR(64) UNIQUE NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '30 days'),
    is_revoked BOOLEAN NOT NULL DEFAULT FALSE
);
CREATE INDEX idx_invites_lookup ON visit_invitations(token_hash) WHERE is_revoked IS FALSE;
```

---

## 4. API Surface & Web Protocols

### 4.1 Authentication Endpoints
- **`POST /api/v1/auth/request-link`**
  - Payload: `{ "email": "user@example.com" }`
  - Response: `{ "status": "ok", "message": "If the email is valid, a magic link has been sent." }`
  - Rate limit: 3 requests per email per 15 minutes.
- **`GET /api/v1/auth/verify?token=<magic_token>`**
  - Validates token against database/Redis single-use store.
  - Sets HTTP-Only, Secure, `SameSite=Lax` cookie `session_id`.
  - Redirects to `/aviary`.
- **`POST /api/v1/auth/logout`**
  - Invalidates active `session_id`.

### 4.2 Aviary Sync & Interaction Endpoints
- **`GET /api/v1/aviary/snapshot`**
  - Headers: `Cookie: session_id=...`
  - Response `200 OK`:
    ```json
    {
      "account_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
      "server_timestamp": "2026-07-24T07:05:00.000Z",
      "server_tick_id": 140922,
      "time_of_day_phase": "MORNING",
      "weather": {
        "type": "CLEAR",
        "intensity": 0.0
      },
      "birds": [
        {
          "bird_id": "c39e2e12-4211-4f90-a212-8811d0e12345",
          "given_name": "Pip",
          "species_id": "warbler_grey",
          "mood": "CONTENT",
          "perch_zone": "FRONT",
          "plumage_saturation": 0.42,
          "vocal_motif_seed": 948201,
          "target_pose": "PREEN_IDLE"
        },
        {
          "bird_id": "e8910a11-1234-4567-89ab-cdef01234567",
          "given_name": "Wren",
          "species_id": "sparrow_song",
          "mood": "DROWSY",
          "perch_zone": "BACK",
          "plumage_saturation": 0.38,
          "vocal_motif_seed": 102948,
          "target_pose": "FLUFFED_SLEEP"
        }
      ]
    }
    ```
- **`POST /api/v1/aviary/events`**
  - Submits batch of accumulated interaction events.
  - Request Payload:
    ```json
    {
      "events": [
        {
          "event_type": "PRESENCE_PING",
          "client_timestamp": "2026-07-24T07:04:45.000Z"
        },
        {
          "event_type": "LISTEN_IN_START",
          "target_bird_id": "c39e2e12-4211-4f90-a212-8811d0e12345",
          "client_timestamp": "2026-07-24T07:04:50.000Z"
        }
      ]
    }
    ```
  - Response `202 Accepted`: `{ "accepted_count": 2 }`

### 4.3 Social & Field Notebook Endpoints
- **`POST /api/v1/social/invites`**
  - Payload: `{ "visitor_email": "friend@example.com" }`
  - Response: `{ "invite_id": "...", "expires_at": "..." }`
- **`DELETE /api/v1/social/invites/:invite_id`**
  - Revokes specified invitation immediately.
- **`GET /api/v1/social/visit?token=<visit_token>`**
  - Returns read-only aviary snapshot for visitor. Rejects event post attempts.
- **`GET /api/v1/notebook`**
  - Returns list of generated field notebook entries.

---

## 5. Simulation Engine & Algorithm Specifications

### 5.1 Server-Side Tick Architecture
The simulation tick service executes every 60 seconds. For each account:
1. Queries unprocessed rows from `interaction_events` where `processed_by_tick IS NULL`.
2. Calculates valid presence minutes $T_{\text{presence}}$ in the tick window. A 15s window is valid iff `PRESENCE_PING` was received with `visible = true`, `focused = true`, and `active = true`.
3. Updates slow-timescale personality vectors.
4. Updates fast-timescale bird moods.
5. Determines if a sparse Field Notebook observation should be written.
6. Serializes new snapshot to Redis cache and marks event batch processed.

### 5.2 Personality Drift Mathematical Formulation
Drift is modelled as a continuous first-order low-pass response driven by positive presence and targeted interactions:

$$\vec{P}_{t+1} = \vec{P}_t + \alpha \cdot \mathbf{W} \cdot \vec{I}_t \odot (1.0 - \vec{P}_t)$$

Where:
- $\vec{P}_t = \begin{bmatrix} \text{boldness} \\ \text{social\_warmth} \\ \text{vocal\_frequency} \\ \text{plumage\_saturation} \\ \text{curiosity} \end{bmatrix} \in [0, 1]^5$
- $\alpha = 1.2 \times 10^{-4}$ (scaling constant tuned for ~1 week instrument visibility, ~3 week user visibility).
- $\vec{I}_t = \begin{bmatrix} T_{\text{presence\_min}} \\ N_{\text{listen\_in}} \\ N_{\text{offers}} \\ N_{\text{settle}} \end{bmatrix}$ (interaction signal vector during tick).
- $\mathbf{W}$ is a $5 \times 4$ positive weight matrix mapping interactions to traits:
  $$\mathbf{W} = \begin{bmatrix} 
  0.4 & 0.1 & 0.3 & 0.0 \\
  0.3 & 0.5 & 0.1 & 0.0 \\
  0.2 & 0.4 & 0.2 & 0.0 \\
  0.5 & 0.2 & 0.1 & 0.0 \\
  0.1 & 0.2 & 0.6 & 0.0 
  \end{bmatrix}$$
- $(1.0 - \vec{P}_t)$ ensures monotonic asymptotic convergence to 1.0 without overshoot.
- If $\vec{I}_t = \vec{0}$ (neglect), $\vec{P}_{t+1} = \vec{P}_t$. **Zero negative drift.**

### 5.3 Mood Transition State Machine
Moods transition based on local time, weather, and recent interaction events:

```
                  +--------------+
                  |    WARY      |
                  +------+-------+
                         | (Accepted offer / repeated presence)
                         v
  +--------------+  +----+---------+  +--------------+
  |    ALERT     |<--|  CONTENT    |-->|   CURIOUS    |
  +--------------+  +----+---------+  +--------------+
                         | (Dusk / Settle event)
                         v
                  +------+-------+
                  |   DROWSY     |
                  +--------------+
```

- **WARY $\to$ CONTENT**: High boldness speeds transition; offer nearby accelerates.
- **CONTENT $\to$ CURIOUS**: Offer presented; head-tilt and approach front perch.
- **CONTENT $\to$ DROWSY**: Triggered near sunset or when user initiates `SETTLE`.
- **ALERT**: Triggered by sudden ambient weather (wind ripple) or neighbor alarm call.

---

## 6. Multi-Device Sync & Conflict Prevention

### 6.1 Server-Authoritative Architecture
- The server simulation tick is the **sole writer** of personality vectors (`birds` table) and canonical state snapshots.
- Client browsers (laptop, mobile phone, tablet) operate exclusively as **stateless renderers** and **event submitters**.
- No device ever computes local drift or transmits absolute personality values (`boldness = 0.5`).

### 6.2 Concurrent Session Event Ingestion
- When a user has tabs open across multiple devices, both devices push `PRESENCE_PING` and interaction events to `interaction_events`.
- Events are assigned monotonic database sequence numbers (`BIGSERIAL`).
- The simulation tick processes events in strict `event_id` order. Presence pings from multiple active devices in the same 15s window are deduplicated per account ID to prevent artificial presence inflation.
- Because clients submit events, not state, **Last-Write-Wins (LWW) conflict resolution is entirely avoided.**

---

## 7. Frontend Rendering Pipeline & Visual Design System

### 7.1 Render Loop Architecture
- Built using an HTML5 Canvas 2D / WebGL 2D context.
- Maintains a constant 60fps render loop via `requestAnimationFrame`.
- On snapshot receipt, target bird coordinates and pose vectors are updated. The render pipeline smoothly interpolates position, rotation, and feather movement between snapshot states over time $t$:
  $$X_{\text{rendered}}(t) = X_{\text{current}} + (X_{\text{target}} - X_{\text{current}}) \cdot (1 - e^{-\lambda \Delta t})$$

### 7.2 Scene Composition & Layering
1. **Sky & Lighting Gradient**: Dynamic canvas background updated continuously based on local timezone time-of-day.
2. **Back Perch Layer**: Distant foliage with subtle parallax offset ($0.2\times$ cursor movement).
3. **Middle Perch Layer**: Main perches where birds sit, preen, and rest.
4. **Front Perch & Offer Layer**: Foreground perches and water pool/seed tray offer zone ($1.0\times$ parallax).
5. **Procedural Bird Sprites**: Bird silhouettes rendered with procedural SVG vector paths; color saturation scaled dynamically by `plumage_saturation` trait.
6. **Top-Bar Chrome UI**: Minimal HTML overlay (Settings, Accessibility, Field Notebook, Offers). Auto-fades to `opacity: 0.05` after 3 seconds of pointer inactivity.

### 7.3 Reduced-Motion Surface (`prefers-reduced-motion`)
- When enabled (system preference or accessibility toggle):
  - Frame-by-frame animation loops (preening, head bobbing) are replaced by **slow cross-fades (0.8s transition)** between discrete static pose keyframes.
  - Perch transitions cross-fade position over 1.5 seconds instead of animating a flight arc.
  - Ambient leaf/feather drift particles are completely disabled.
  - Day/night lighting transitions slow down by $2\times$.

---

## 8. Audio Pipeline & WebAudio Runtime

### 8.1 Procedural Call Synthesis Architecture
- **Zero Recorded Audio**: 100% procedural synthesis using WebAudio API (`AudioContext`, `OscillatorNode`, `BiquadFilterNode`, `GainNode`).
- **Motif Generator**:
  - Each species has a motif grammar defined by base frequency $f_0$, frequency modulation envelope $FM(t)$, gain envelope $A(t)$, and trill frequency.
  - Individual bird `vocal_motif_seed` introduces small frequency micro-variations ($\pm 3\%$) so Pip's call is uniquely recognizable from Wren's.

```
+-------------------------------------------------------------------------------+
|                            WEBAUDIO SYNTHESIZER                               |
|                                                                               |
|  +-------------------+      +--------------------+      +------------------+  |
|  |  OscillatorNode   |----->| BiquadFilterNode   |----->|  GainNode        |  |
|  |  (FM Synthesis)   |      | (Formant Shaping)  |      |  (Envelope A(t)) |  |
|  +-------------------+      +--------------------+      +------------------+  |
|                                                                  |            |
|                                                                  v            |
|  +-------------------+      +--------------------+      +------------------+  |
|  | StereoPannerNode  |----->|  DynamicsCompressor|<----|  Listen-In Gain  |  |
|  | (Scene X-Pos)     |      |  (Master Bus)      |      |  Automation      |  |
|  +--------+----------+      +---------+----------+      +------------------+  |
+-----------|---------------------------|---------------------------------------+
            v                           v
     Left Speaker                Right Speaker
```

### 8.2 Chorus Mixing & Listen-In Automation
- **Chorus Anti-Phase Staggering**: When multiple birds call in response to one another, call start times are staggered by randomized intervals (200ms to 750ms) to prevent unnatural phase cancellation artifacts.
- **Listen-In Mix Dynamics**:
  - Unfocused mix state: All birds mixed at ambient level (0dB reference).
  - Listen-in engaged on Bird $A$:
    - Bird $A$ Gain: Ramps up to $+3.0\text{dB}$ over $1.2\text{s}$ via `exponentialRampToValueAtTime`.
    - Other Birds Gain: Ramps down to $-12.0\text{dB}$ over $1.2\text{s}$ (preserving ambient presence floor; **never 0dB muted**).

### 8.3 WebAudio Fallback Strategy
- If `AudioContext` fails to start (e.g. browser autoplay policy restrictions or missing hardware):
  - System enters **Quiet Mode**.
  - Call captions auto-enable globally.
  - Zero network requests are made for fallback MP3 files.

---

## 9. Accessibility Implementation Strategy

### 9.1 Screen-Reader Narration Surface
- Dedicated off-screen HTML element with `aria-live="polite"` and `aria-atomic="true"`.
- Updated at slow 30–60 second intervals with server-generated naturalist prose strings:
  > *"a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."*
- User-initiated actions (return-greeting, accepted offer) push immediate updates to the queue in naturalist voice.

### 9.2 Procedural Call Captioning
- Dynamic floating text captions rendered adjacent to calling birds:
  > *`[a soft three-note rise]`*
  > *`[a low trill, paused, low trill again]`*
- Caption strings generated in real time from WebAudio motif parameter envelopes.

### 9.3 Keyboard Navigation & Contrast
- Full keyboard focus ring reachable via `Tab` key.
- Keybindings: `Tab` (cycle UI/birds), `Enter` (engage Listen-In), `Escape` (exit Listen-In), `O` (Open Offers menu), `N` (Open Field Notebook), `S` (Settle Aviary).
- Focus indicators use a high-contrast dual ring (outer white `#FFFFFF`, inner dark charcoal `#1A1A1A`) passing WCAG AA (>4.5:1 ratio) over all day/night backgrounds.

---

## 10. Performance Budgets & Observability

### 10.1 Quantitative Performance Thresholds

| Metric | Budget / Limit | Measurement & Enforcement Method |
|---|---|---|
| **Initial JS Bundle Size** | $< 2.0\text{ MB}$ (gzipped) | Webpack/Vite bundle analyzer build step check in CI. |
| **Time to First Bird (TTFBird)** | $< 500\text{ ms}$ | PerformanceObserver mark at first canvas bird draw on 4G network profile. |
| **Frame Rate** | $60\text{ fps}$ | Measured over 30-min continuous run on Intel HD 520 GPU baseline hardware. |
| **Memory Allocation Growth** | $0.0\text{ MB}$ over 30 min | Automated Puppeteer heap snapshot comparison test in CI. |
| **Simulation Tick p99 Latency** | $< 5.0\text{ seconds}$ | Datadog/Prometheus server-side timer metric alarm. |

### 10.2 Observability & Privacy Isolation Boundary
- **Allowed Operational Telemetry**:
  - Anonymous HTTP latency histograms, API error rates (`5xx`/`4xx`), edge cache hit/miss ratios, synthetic load testing timing, and WebAudio error initialization counts.
- **Strict Privacy Isolation Boundary**:
  - Per-bird names, personality vectors, mood states, user presence duration, notebook observations, and email addresses are **strictly excluded** from external analytics/telemetry pipelines (e.g. Mixpanel, Datadog logs).
  - Encrypted database stores are isolated from telemetry collection daemons.

---

## 11. Rollout & Operations Strategy

### 11.1 Phase 1 Deliverables
- Single-user web application with magic-link auth.
- 2 starter birds auto-assigned from 6 species motifs.
- Server-side simulation tick for drift, mood, and notebook generation.
- Full WebAudio procedural synthesis and Listen-In mix controls.
- Complete accessibility implementation (screen-reader narration, reduced-motion pose cross-fading, captions, keyboard navigation).
- Optional read-only visit invitations.

### 11.2 Aviary Age Pacing Schedule
- $1^{\text{st}} \& 2^{\text{nd}}$ Birds: Day 0 (Account Creation).
- $3^{\text{rd}}$ Bird Unlock: Day 30.
- $4^{\text{th}}$ Bird Unlock: Day 90.
- $5^{\text{th}}$ Bird Unlock: Day 180.
- $6^{\text{th}}$ Bird Unlock: Day 270.
- $7^{\text{th}}$ Bird Unlock: Day 365 (Maximum Aviary Capacity).

---

## 12. Engineering Risks & Mitigation Strategies

### 12.1 Drift Calibration Instability
- *Risk*: Personality drift feels too rapid (reading as a gamified Tamagotchi) or imperceptible (reading as static).
- *Mitigation*: Daily automated CI simulation script feeding 30 days of synthetic presence data into the drift differential equation, verifying that delta values trigger instrument visibility at 7 days and user-level visual thresholds at 21 days.

### 12.2 Browser WebAudio Autoplay Restrictions
- *Risk*: Browsers blocking `AudioContext` startup, preventing calls from playing on initial tab open.
- *Mitigation*: Initialize `AudioContext` in suspended state; seamlessly resume audio on the first user interaction anywhere on the document. Default to quiet mode with call captions active until audio resumes.

### 12.3 Multi-Tab Presence Duplication
- *Risk*: Users opening 5 tabs simultaneously inflating presence time $5\times$.
- *Mitigation*: Server-side deduplication window collapsing all `PRESENCE_PING` events within the same 15-second tick slice per `account_id`.

### 12.4 Accessibility Framing Regression
- *Risk*: Developers substituting mechanical UI state logs for naturalist screen-reader narration strings during feature additions.
- *Mitigation*: Automated string inspection linter in CI enforcing naturalist prose structure constraints on all screen-reader narration outputs.
