# Pocket Aviary — System Architecture & Implementation Plan (v1)

## 1. Executive Summary & Design Principles Alignment

Pocket Aviary is an ambient, browser-based virtual aviary hosting between two and seven procedurally animated birds in a single, responsive horizontal scene. The system models an observational relationship rather than a custodial or gamified one: birds live in a state of continuous server-side simulation, noticing the user upon arrival, reacting with subtle procedural vocalizations and micro-motion, and undergoing slow, monotonic personality drift over weeks of idle attention (presence).

The technical architecture is built directly around the core design principles defined in the PRD:
1. **Feels Alive, Not Robotic**: The aviary is simulated on an authoritative server tick (~60s) regardless of whether client tabs are open. Calls are procedurally synthesized via client-side WebAudio without recorded audio loops. The initial paint renders birds mid-motion at a calculated phase offset, avoiding "wake-up" transitions, entry animations, or loading spinners.
2. **Notice, Never Announce**: There are no "Welcome back!" toasts, banners, streaks, milestone badges, or arrival dialogs. The return-greeting is performed exclusively by the birds through absence-modulated procedural calls and posture shifts.
3. **Charm Comes from Specificity**: The dual-voice architectural separation is strictly enforced:
   - *Naturalist Voice*: Lowercase, present-tense, bird-centric prose used in all product surfaces, including the field notebook, screen-reader live narration, call captions, and offer interactions.
   - *Matter-of-Fact Voice*: Standard capitalized, concise technical clarity used strictly on system surfaces (magic-link authentication, session expiration, sync errors, settings, and invite revocation).
4. **Restraint Over Richness**: Exactly one horizontal scene fitting the viewport without scrolling, panning, or zooming. Two starter birds upon adoption, scaling strictly with aviary age up to a hard ceiling of seven birds to preserve psychoacoustic call discriminability.
5. **Presence as Real Interaction**: Idle attention is the primary input to personality drift. A user sitting and watching without clicking provides the dominant signal. Neglect is never penalized: drift is monotonic toward expressive, with zero negative decay on absence.

---

## 2. Scope & Boundary Definition

### 2.1 In-Scope for v1
- **Single Horizontal Scene**: Fixed 1-screen viewport across desktop, tablet, and mobile, structured into three depth zones: Front, Middle, and Back perches.
- **Bird Population & Capacity**: Two system-assigned starter birds chosen from a 6-species pool; customizable bird names; capacity capped at seven birds, with additional birds unlocked exclusively by aviary age milestones.
- **Procedural Call Synthesis**: Real-time client-side WebAudio synthesis utilizing FM synthesis, resonant formant filters, and dynamic ADSR envelopes. No audio loops or recorded audio files.
- **Chorus & Spatial Mixing**: Real-time multi-bird audio graph with stereo panning mapped to perch positions and ambient convolution reverb.
- **Core Interactions**:
  - *Return-Greeting*: Procedural notice on session start, modulated by absence duration, boldness, and mood; staggered multi-bird response.
  - *Listen-In*: User focus on a single bird; smooth gain boost (+4 dB) for the focused bird, smooth attenuation (-12 dB non-zero ambient floor) for non-focused birds.
  - *Offer*: Top-bar affordance for offering seed, song fragment, or still pool, gated by per-bird cooldowns (~3-5 minutes) with reactions shaped by curiosity and mood.
  - *Settle*: User-initiated evening shift with quieting calls, soft lighting, and a 5-second undo window; functionally equivalent to closing the tab.
  - *Field Notebook*: System-authored naturalist observer log; sparse cadence (~1 entry every few days); uncurated, read-only, non-gamified.
  - *Presence Sentinel*: Strict multi-factor presence tracking (`visibilityState === "visible"` AND window focused AND pointer/key activity within window).
- **Accounts & Sync**: Single-user accounts authenticated via 15-minute email magic links; synthetic UUID account isolation; single canonical aviary per account synced across multiple devices via authoritative server-side tick and additive event processing.
- **Social Visits**: Optional, quiet, email-invited read-only ambient viewing; revocable; no co-presence, no visitor drift impact, no visitor interactions, notifications disabled by default.
- **Accessibility**: Naturalist running screen-reader narration (`aria-live="polite"`), procedural call captioning, full keyboard navigation with high-contrast outlines, reduced-motion cross-fade mode, WCAG AA contrast compliance.
- **Data & Privacy**: JSON aviary state export, 30-day soft deletion transitioning to permanent purge, strict telemetry isolation (zero per-bird data in analytics).

### 2.2 Explicit Out-of-Scope (Non-Goals)
- **No Native Applications**: Web-only architecture. No iOS, Android, or desktop wrappers.
- **No Gamification Elements**: No achievements, badges, levels, experience points (XP), scores, daily streaks, green-dot calendars, visit counters, or celebratory toasts.
- **No Tamagotchi / Custodial Mechanics**: No hunger, feeding schedules, illness, death, neglect penalties, or distress states. Absence causes quietness, never suffering.
- **No Social Network Mechanics**: No user profiles, public feeds, aviary directories, global search/discovery, leaderboards, comments, chat, visitor avatars, or co-presence indicators.
- **No Recorded Audio Fallback**: If WebAudio is unavailable, the aviary plays in graceful silence with captions enabled; no sample audio files are shipped.
- **No Client Authoritative State**: Clients never compute or mutate personality vectors or moods directly.

---

## 3. High-Level Architecture & System Boundaries

### 3.1 Architectural Topology

```
+-----------------------------------------------------------------------------------+
|                                 CLIENT PLATFORM                                   |
|                                                                                   |
|  +--------------------------------+   +----------------------------------------+  |
|  |       Presentation Layer       |   |             Audio Pipeline             |  |
|  |  - Canvas / WebGL (60fps)      |   |  - WebAudio Procedural Synthesizer     |  |
|  |  - 3 Perch Zones & Parallax    |   |  - Real-Time Chorus & Stereo Mixer     |  |
|  |  - Idle Micro-Motion Engine    |   |  - Listen-In Crossfade Controller      |  |
|  |  - Reduced-Motion Cross-Fade   |   |  - Silent Fallback & Caption Trigger   |  |
|  +--------------------------------+   +----------------------------------------+  |
|                  |                                        |                       |
|  +--------------------------------+   +----------------------------------------+  |
|  |     State & Presence Engine    |   |         Accessibility Surfaces         |  |
|  |  - Interpolation & Lerp Buffer |   |  - ARIA Live Polite Prose Narrator     |  |
|  |  - Presence Sentinel           |   |  - Real-Time Call Captions             |  |
|  |  - Optimistic Event Queue      |   |  - High-Contrast Keyboard Navigation   |  |
|  +--------------------------------+   +----------------------------------------+  |
+-----------------------------------------------------------------------------------+
                                   |                 ^
             POST Interaction Events |                 | State Snapshots (SSE/GET)
             Presence Heartbeats    |                 |
                                   v                 |
+-----------------------------------------------------------------------------------+
|                             EDGE & API GATEWAY TIER                               |
|  - Edge CDN: Critical Asset Delivery (<2MB Bundle) & Inlined Initial Snapshot     |
|  - Reverse Proxy & Rate Limiting (Token Bucket per IP / Session)                  |
|  - Magic-Link Authentication & Session Token Validation                           |
+-----------------------------------------------------------------------------------+
                                   |
                   +---------------+---------------+
                   |                               |
                   v                               v
+------------------------------------+   +------------------------------------------+
|          API GATEWAY SERVICE       |   |        SIMULATION WORKER DAEMON          |
|  - /api/v1/auth/*                  |   |  - 60s Authoritative Simulation Tick     |
|  - /api/v1/aviary/snapshot         |   |  - Append-Only Event Consumer            |
|  - /api/v1/aviary/events (Ingest)  |   |  - Monotonic Drift Calculator (LPF)      |
|  - /api/v1/visits/*                |   |  - Diurnal & Mood State Machine          |
|  - Account Export & Deletion Jobs  |   |  - Naturalist Notebook Generator         |
+------------------------------------+   +------------------------------------------+
                   |                               |
                   +---------------+---------------+
                                   |
                                   v
+-----------------------------------------------------------------------------------+
|                               PERSISTENCE STORAGE                                 |
|  - PostgreSQL: Relational DB (Accounts, Aviaries, Birds, Vectors, Moods, Events)  |
|  - Redis: Tick Locks, Active Session Caches, SSE Event Pub/Sub                    |
+-----------------------------------------------------------------------------------+
```

### 3.2 Client / Server Responsibility Boundary
- **Server Responsibilities**:
  - Acts as the single source of truth for aviary state, time, weather, bird identity, personality vectors, moods, and notebook observations.
  - Executes the authoritative 60-second simulation tick.
  - Consumes client interaction events and presence heartbeats sequentially; calculates additive personality drift deltas.
  - Generates immutable state snapshots for connected clients and visitors.
- **Client Responsibilities**:
  - Renders visual frames at 60fps, smoothly interpolating between authoritative snapshots.
  - Generates procedurally randomized visual idle micro-motions (preening, head scans, breathing) and non-authoritative ambient particles (falling leaves, feathers).
  - Synthesizes procedural audio calls in real time via WebAudio based on motif grammar instructions in the snapshot.
  - Gathers presence signals and sends debounced heartbeats to the server every 30 seconds.
  - Manages screen-reader polite live-region updates and call captions.

---

## 4. Comprehensive Data Model & Database Schemas

### 4.1 Synthetic Account ID Architecture
To prevent Personally Identifiable Information (PII) leakage into logs, caches, metrics, and database foreign keys:
- The user’s email address is stored exclusively on the `accounts` record in an encrypted column (`email_encrypted`) using AES-256-GCM.
- All internal tables, foreign keys, partition keys, event logs, background worker queues, and telemetry records refer exclusively to a synthetic `account_id` (UUIDv4).
- Email addresses are decrypted only during authentication link generation, account settings display, and export operations.

### 4.2 Relational DDL (PostgreSQL)

```sql
-- Core Account & Session Tables
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_hash VARCHAR(64) NOT NULL UNIQUE, -- SHA-256 for login lookups
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    soft_deleted_at TIMESTAMPTZ,
    settings JSONB NOT NULL DEFAULT '{
        "visit_notifications_enabled": false,
        "reduced_motion_override": null,
        "captions_enabled": false
    }'::jsonb
);

CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    session_token_hash VARCHAR(64) NOT NULL UNIQUE,
    user_agent_hint TEXT,
    ip_subnet VARCHAR(45),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ
);

CREATE TABLE magic_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL, -- NOW() + INTERVAL '15 minutes'
    consumed_at TIMESTAMPTZ
);

-- Aviary & Species Tables
CREATE TYPE weather_state AS ENUM ('clear', 'light_rain', 'soft_wind');
CREATE TYPE perch_zone AS ENUM ('front', 'middle', 'back');
CREATE TYPE bird_mood AS ENUM ('wary', 'content', 'curious', 'drowsy', 'alert');

CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    current_weather weather_state NOT NULL DEFAULT 'clear',
    weather_expires_at TIMESTAMPTZ,
    settled_at TIMESTAMPTZ,
    settled_until TIMESTAMPTZ,
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id SMALLINT NOT NULL CHECK (species_id BETWEEN 1 AND 6),
    custom_name VARCHAR(64) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    perch_zone perch_zone NOT NULL DEFAULT 'middle',
    perch_slot SMALLINT NOT NULL DEFAULT 0,
    CONSTRAINT uq_aviary_perch UNIQUE (aviary_id, perch_zone, perch_slot)
);

-- Personality Vector (Strictly Server-Side, Hidden from User)
CREATE TABLE personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(id) ON DELETE CASCADE,
    boldness REAL NOT NULL CHECK (boldness >= 0.0 AND boldness <= 1.0),
    social_warmth REAL NOT NULL CHECK (social_warmth >= 0.0 AND social_warmth <= 1.0),
    vocal_frequency REAL NOT NULL CHECK (vocal_frequency >= 0.0 AND vocal_frequency <= 1.0),
    plumage_saturation REAL NOT NULL CHECK (plumage_saturation >= 0.0 AND plumage_saturation <= 1.0),
    curiosity REAL NOT NULL CHECK (curiosity >= 0.0 AND curiosity <= 1.0),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE bird_moods (
    bird_id UUID PRIMARY KEY REFERENCES birds(id) ON DELETE CASCADE,
    current_mood bird_mood NOT NULL DEFAULT 'content',
    mood_entered_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_call_at TIMESTAMPTZ,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Append-Only Interaction & Presence Event Log
CREATE TYPE interaction_type AS ENUM (
    'presence_ping',
    'listen_in_start',
    'listen_in_end',
    'offer_seed',
    'offer_song',
    'offer_pool',
    'settle',
    'settle_undo'
);

CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    event_type interaction_type NOT NULL,
    bird_id UUID REFERENCES birds(id) ON DELETE SET NULL,
    client_timestamp TIMESTAMPTZ NOT NULL,
    server_timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    processed_at TIMESTAMPTZ
);

CREATE INDEX idx_interaction_unprocessed ON interaction_events (aviary_id, id) WHERE processed_at IS NULL;

-- Field Notebook Observations
CREATE TABLE notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    observed_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    prose_text TEXT NOT NULL,
    focal_bird_ids UUID[] NOT NULL DEFAULT '{}',
    observation_trigger VARCHAR(64) NOT NULL
);

CREATE INDEX idx_notebook_aviary ON notebook_entries (aviary_id, observed_at DESC);

-- Social Visits
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    invite_token_hash VARCHAR(64) NOT NULL UNIQUE,
    visitor_email_encrypted BYTEA NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL, -- NOW() + INTERVAL '30 days'
    revoked_at TIMESTAMPTZ
);

CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    invitation_id UUID NOT NULL REFERENCES visit_invitations(id) ON DELETE CASCADE,
    started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    ended_at TIMESTAMPTZ,
    duration_seconds INTEGER
);
```

### 4.3 Snapshot Data Contract (JSON)
Delivered to clients on session open, periodic synchronization, and SSE updates:

```json
{
  "aviary": {
    "id": "c8b417c8-7c87-4318-8f81-5d09f6e4a2e1",
    "server_time": "2026-09-06T14:35:00Z",
    "solar_phase": "morning",
    "weather": {
      "type": "clear",
      "intensity": 0.0
    },
    "is_settled": false,
    "birds": [
      {
        "id": "e43d3b76-4d24-4f81-9b16-5e5d3fae1201",
        "name": "Pip",
        "species_id": 1,
        "perch": {
          "zone": "front",
          "slot": 0,
          "x_norm": 0.28,
          "y_norm": 0.65
        },
        "mood": "content",
        "plumage_saturation": 0.68,
        "call_signature": {
          "base_freq_hz": 2840,
          "motif_id": "warbler_triple_rise",
          "pitch_shift_cents": 12
        },
        "next_call_estimate_s": 14.5
      },
      {
        "id": "31b1990c-f38b-4b2a-a92c-e1a3bc47f002",
        "name": "Wren",
        "species_id": 2,
        "perch": {
          "zone": "back",
          "slot": 1,
          "x_norm": 0.72,
          "y_norm": 0.38
        },
        "mood": "wary",
        "plumage_saturation": 0.52,
        "call_signature": {
          "base_freq_hz": 3450,
          "motif_id": "wren_short_trill",
          "pitch_shift_cents": -8
        },
        "next_call_estimate_s": 38.0
      }
    ],
    "notebook_snippet": "pip greeted before wren today, first time this week."
  }
}
```

---

## 5. API Surface & Communication Protocols

### 5.1 Endpoints Specification

| Endpoint | Method | Role / Auth | Description |
|---|---|---|---|
| `/api/v1/auth/magic-link` | `POST` | Public | Request magic link email. Rate-limited to 5 requests per hour per email. |
| `/api/v1/auth/verify` | `POST` | Public | Verify link token; sets HttpOnly `__Host-Session` cookie and returns session metadata. |
| `/api/v1/auth/sessions` | `GET` | Authenticated | List all active sessions with user-agent hint and creation date. |
| `/api/v1/auth/sessions/:id` | `DELETE` | Authenticated | Revoke a specific session. |
| `/api/v1/aviary/snapshot` | `GET` | Authenticated | Fetch current aviary snapshot. Supports `ETag` and conditional `If-None-Match`. |
| `/api/v1/aviary/stream` | `GET` | Authenticated | Server-Sent Events (SSE) channel for tick snapshots, weather shifts, and live updates. |
| `/api/v1/aviary/events` | `POST` | Authenticated | Ingest batch of client interaction events & presence heartbeats. |
| `/api/v1/aviary/notebook` | `GET` | Authenticated | Paginated field notebook observations (cursor-based, ordered by `observed_at DESC`). |
| `/api/v1/birds/:id/rename` | `PATCH` | Authenticated | Rename a bird. Only mutates `custom_name`. |
| `/api/v1/visits/invitations` | `POST` | Authenticated | Host issues new email visit invite. Rate-limited to 10 per day. |
| `/api/v1/visits/invitations/:id` | `DELETE` | Authenticated | Host revokes an invitation immediately. |
| `/api/v1/visits/log` | `GET` | Authenticated | Host views chronological visit history log. |
| `/api/v1/visits/view/:token` | `GET` | Public (Token) | Visitor retrieves read-only snapshot. Returns 403 if revoked or expired. |
| `/api/v1/visits/stream/:token` | `GET` | Public (Token) | Visitor SSE stream for read-only aviary updates. |
| `/api/v1/account/export` | `POST` | Authenticated | Asynchronously packages account state and emails a secure download link. |
| `/api/v1/account` | `DELETE` | Authenticated | Flags account for 30-day soft deletion. |
| `/api/v1/account/restore` | `POST` | Authenticated | Cancels soft deletion within 30 days. |

### 5.2 Interaction Event Ingestion Payload (`POST /api/v1/aviary/events`)
Clients submit batched events every 30 seconds (or immediately on key interaction boundaries):

```json
{
  "batch_id": "0191c944-934e-7b78-9e60-449eef3d21b4",
  "events": [
    {
      "event_type": "presence_ping",
      "client_timestamp": "2026-09-06T14:34:30.000Z",
      "payload": {
        "duration_seconds": 30,
        "window_focused": true,
        "pointer_active": true
      }
    },
    {
      "event_type": "listen_in_start",
      "bird_id": "e43d3b76-4d24-4f81-9b16-5e5d3fae1201",
      "client_timestamp": "2026-09-06T14:34:42.100Z",
      "payload": {}
    }
  ]
}
```

### 5.3 Error Handling & Voice Division
When an API error occurs, responses strictly adhere to the Matter-of-Fact system voice:
- `401 Unauthorized`: `{"error": "Your session timed out. Sign in again to keep watching."}`
- `403 Forbidden` (Revoked Visit): `{"error": "This visit invitation is no longer active."}`
- `404 Not Found` (Expired Link): `{"error": "We couldn't sign you in. The link may have expired. Try requesting a new link."}`
- `500 Internal Error`: `{"error": "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."}`

---

## 6. Simulation Engine Design & Drift Mathematics

### 6.1 Simulation Worker Daemon & 60s Authoritative Tick
The simulation daemon runs as an asynchronous cluster worker. Every 60 seconds, it iterates over all active aviaries (or triggers on-demand lazy simulation catch-up when an inactive aviary becomes visible):
1. **Acquire Aviary Lock**: Redis distributed lock (`SET aviary:{id}:lock PX 5000 NX`) prevents duplicate tick execution across workers.
2. **Drain Event Log**: Pulls all unconsumed `interaction_events` for the aviary ordered by `id ASC`.
3. **Calculate True Presence Seconds**: Accumulates valid `presence_ping` durations satisfying the 3-factor rule.
4. **Compute Personality Drift Deltas**: Executes the monotonic low-pass filter update.
5. **Update Diurnal & Weather Dynamics**: Updates solar lighting angles based on local timezone; steps ambient weather state.
6. **Evaluate Mood State Machine**: Applies diurnal shifts, contagion effects, and interaction impacts.
7. **Evaluate Return-Greeting Queue**: Prepares greeting schedules for newly returning sessions.
8. **Evaluate Field Notebook Heuristics**: Generates naturalist observations if threshold conditions are met.
9. **Commit Canonical State**: Writes updated records transactionally to PostgreSQL; publishes snapshot diff to Redis SSE channel.

### 6.2 Presence Accounting Mechanics
Presence is strictly defined by the simultaneous conjunction of three conditions:
$$ \text{PresenceActive}(t) = ( \text{visibilityState} = \text{visible}) \land ( \text{WindowFocused} = \text{true}) \land (\Delta t_{ \text{last\_input}} \le 180 \text{s})$$
- The client-side Presence Sentinel listens to `visibilitychange`, `window.onfocus`, `window.onblur`, `pointermove`, and `keydown`.
- Pointer/key events reset an internal idle timer. If no input occurs for 180 seconds, presence reporting halts until the next input event.
- Every 30 seconds, if all three conditions are continuously met, a `presence_ping` of 30 seconds is recorded in the client dispatch queue.
- If the browser tab is hidden or backgrounded, no presence is reported. Background tabs never generate drift.

### 6.3 Personality Vector Mathematical Model & Drift Calibration
Each bird has 5 scalar traits $T \in [0.0, 1.0]$:
- $T_{ \text{boldness}}$: Likelihood to perch in the front zone and greet first.
- $T_{ \text{warmth}}$: Likelihood to return calls and initiate flock chorus.
- $T_{ \text{vocal\_freq}}$: Frequency of unobserved calling and rate parameter $\lambda$ in call Poisson distribution.
- $T_{ \text{plumage}}$: Visual saturation and detail level of procedural feather shaders.
- $T_{ \text{curiosity}}$: Likelihood to inspect offers and head-tilt toward ambient sounds.

#### Asymmetric Monotonic Drift Function
Traits move up on positive presence and interactions; they **never** move down due to absence or neglect.
For trait $i$ at tick $k$:
$$T_i^{(k)} = \min\left(1.0, T_i^{(k-1)} + \Delta T_i^{(k)}
ight)$$
$$\Delta T_i^{(k)} = lpha_i \cdot \left( w_{ \text{presence}} \cdot \Delta t_{ \text{presence}} + w_{ \text{listen}} \cdot \Delta t_{ \text{listen}} + w_{ \text{offer}} \cdot N_{ \text{offer}} 
ight)$$
where:
- $\Delta t_{ \text{presence}}$ is valid presence time in hours during the tick.
- $\Delta t_{ \text{listen}}$ is active listen-in duration for that specific bird.
- $N_{ \text{offer}}$ is the count of accepted offers near that bird.
- Default weights: $w_{ \text{presence}} = 1.0$, $w_{ \text{listen}} = 2.5$, $w_{ \text{offer}} = 1.2$.

#### Calibration Constants:
- **1-Week Instrument Threshold**: A typical active user visits 15 minutes daily (1.75 hours/week). The target is a measurable instrument delta of $\Delta T \approx +0.03$.
  $$lpha_i \approx rac{0.03}{1.75} \approx 0.0171 \text{ per presence-hour}$$
- **3-Week Felt-Aliveness Threshold**: Across 3 weeks of regular visits (~5.25 total hours), $\Delta T \approx +0.09$ to $+0.12$. This corresponds to a perceptible shift in front-perch frequency and noticeably richer plumage saturation.
- If absence occurs (e.g., user is away for 2 weeks), $\Delta t = 0 \implies \Delta T = 0$. Traits remain exactly at their historical maximums.

### 6.4 Mood State Machine & Fast-Timescale Transitions
Mood is a fast-timescale emotional state ($M \in \{ \text{wary}, \text{content}, \text{curious}, \text{drowsy}, \text{alert}\}$).

```
          +-----------------+
          |      ALERT      | <------- Sunrise / Soft Wind
          +-----------------+
             ^           |
Passing Rain |           | Morning Warmup
             v           v
          +-----------------+
          |      WARY       | <------- Sudden Movement / Neighbor Alarm
          +-----------------+
             ^           |
   Alarm End |           | Calm / Accepted Offer
             |           v
          +-----------------+
          |     CONTENT     | <------- Default Midday / Post-Interaction
          +-----------------+
             ^           |
   Disengage |           | Offer Presented
             |           v
          +-----------------+
          |     CURIOUS     | <------- Seed / Song / Pool
          +-----------------+
             ^           |
   Awakening |           | Dusk / Settle Gesture / Rain
             |           v
          +-----------------+
          |     DROWSY      | <------- Night / Settled Aviary
          +-----------------+
```

- **Transition Modulators**:
  - *Boldness Buffer*: $P( \text{transition to wary}) = P_{ \text{base}}     imes (1.0 - 0.7 \cdot T_{ \text{boldness}})$. High boldness strongly resists fear transitions.
  - *Social Contagion*: If bird $A$ enters `wary`, adjacent perched birds evaluate a contagion probability $P_{ \text{contagion}} = 0.4     imes (1.0 - T_{ \text{boldness}})$.
  - *Inter-Session Persistence*: Mood does not snap to neutral upon tab opening. When the user returns, the bird's mood is initialized to the exact state computed by the server tick during the user's absence (e.g., if the user opens at 6:00 AM, birds will have transitioned naturally into `alert`).

### 6.5 Call Grammar & Return-Greeting Logic
Calls are generated via stochastic process combined with species motif graphs.
- **Unobserved Call Frequency**: Modeled as an inhomogeneous Poisson process with rate:
  $$\lambda(t) = \lambda_{ \text{species}} \cdot T_{ \text{vocal\_freq}} \cdot \mu_{ \text{mood}} \cdot \mu_{ \text{diurnal}} \cdot \mu_{ \text{weather}}$$
  - $\mu_{ \text{content}} = 1.0$, $\mu_{ \text{alert}} = 1.6$, $\mu_{ \text{wary}} = 0.3$, $\mu_{ \text{drowsy}} = 0.1$.
  - $\mu_{ \text{light\_rain}} = 0.25$ (rain significantly dampens calling).
- **Species Pool (6 Species)**:
  1. *Warbler*: Rapid ascending multi-note sweeps (2.4 kHz – 4.2 kHz).
  2. *Wren*: Energetic, syncopated trills and staccato clicks (3.0 kHz – 5.5 kHz).
  3. *Dove*: Soft, low-frequency resonant dual-tone coos (450 Hz – 800 Hz).
  4. *Finch*: Bright, cheerful chirps with harmonic overtones (3.2 kHz – 6.0 kHz).
  5. *Thrush*: Melodic flute-like two-part phrases (1.8 kHz – 3.6 kHz).
  6. *Nightjar*: Steady, rhythmic churring resonance (1.2 kHz – 2.1 kHz); active during night phase.
- **Absence-Modulated Return-Greeting Algorithm**:
  1. On session open after absence duration $\Delta t_{ \text{absence}}$:
  2. Candidate scoring:
     $$ \text{Score}_i = 0.6 \cdot T_{ \text{boldness}, i} + 0.4 \cdot T_{ \text{warmth}, i} + \text{random}(-0.1, 0.1)$$
  3. Top-scoring bird $G_1$ is selected as primary greeter. It initiates greeting within $1.0 \text{s} - 2.0 \text{s}$.
  4. Greeting structure based on absence:
     - $\Delta t_{ \text{absence}} < 1 \text{ hour}$: Subtle head-tilt and short single-note glance.
     - $1 \text{ hour} \le \Delta t_{ \text{absence}} < 24 \text{ hours}$: Two-note melodic greeting; bird hops closer on its perch.
     - $\Delta t_{ \text{absence}} \ge 24 \text{ hours}$: Extended signature motif; step toward front perch; secondary greeter $G_2$ answers after a $1.8 \text{s} - 3.5 \text{s}$ staggered delay.

---

## 7. Multi-Device Synchronization & Conflict Prevention

### 7.1 Single Canonical State & Anti-Conflict Model
Pocket Aviary completely avoids split-brain and reconciliation conflicts by strictly enforcing server authority:
- **No Last-Write-Wins (LWW) on Client State**: Clients never emit state replacements (e.g., "Pip is at Front Perch"). Clients emit events (e.g., "User clicked front rail at timestamp $T$").
- **Additive Server Accumulation**: Simultaneous sessions on phone and desktop both submit presence heartbeats. The server event consumer aggregates presence additively up to a capped ceiling of 60 seconds per clock minute per aviary.
- **Snapshot Propagation via SSE / Polling**:
  - Connected clients maintain a persistent Server-Sent Events (SSE) connection to `/api/v1/aviary/stream`.
  - When the 60-second tick updates canonical state, a differential snapshot is pushed to all active connections for that `aviary_id`.
  - If SSE fails or disconnects, the client falls back to `GET /api/v1/aviary/snapshot` every 15 seconds.

### 7.2 Client-Side State Reconciliation & Interpolation
When a fresh snapshot $S^{(k)}$ arrives while the client is rendering at 60fps:
- Birds are **never teleported** between perches.
- If bird position in $S^{(k)}$ differs from local render position, the client schedules an organic flight or hop transition over 1.2 seconds using cubic bezier ease-in-out curve:
  $$x(t) = \text{bezier}(x_{ \text{current}}, x_{ \text{target}}, t)$$
  with an arc apex displacement $\Delta y = -40 \text{px} \cdot \sin(\pi t)$ to simulate flight trajectory.
- Plumage saturation smoothly lerps to target value over 3.0 seconds.
- Audio parameters (base frequency and next call delay) smoothly update in the local synthesis sequencer.

---

## 8. Frontend Rendering Pipeline

### 8.1 Scene Composition & Visual Architecture
The aviary is rendered to a full-viewport HTML5 Canvas element (with WebGL 2D acceleration context):
- **Aspect Ratio & Responsive Framing**: Fixed vertical coordinate space normalized from 0.0 to 1.0. Horizontal space expands responsively. On mobile viewports (narrow aspect), perches contract horizontally along the rail without clipping birds. All perches remain 100% visible on screen at all times.
- **Visual Stacking Layers**:
  - `Layer 0 (Sky & Solar Backdrop)`: Dynamic gradient mapped to solar elevation angle. Smooth dawn ochres, bright midday azure, warm evening amber, deep twilight navy.
  - `Layer 1 (Far Background)`: Distant foliage, soft hills, subtle horizontal parallax shift (factor 0.03) tied to smooth cursor movement.
  - `Layer 2 (Middle Ground - Perches & Birds)`:
    - Back Perch (Y = 0.38, Scale = 0.75, Desaturation = 15%)
    - Middle Perch (Y = 0.52, Scale = 0.88)
    - Front Perch (Y = 0.68, Scale = 1.05)
    - Bird mesh rendering with plumage saturation shader and procedurally driven eye blinks.
  - `Layer 3 (Foreground)`: Occasional soft leaf branch overhang (parallax factor 0.08).
  - `Layer 4 (Ambient Particles)`: Procedurally generated falling leaves and drifting down feathers (client-side only, zero simulation state).
  - `Layer 5 (UI Chrome)`: Top-bar navigation overlay and call captions.

### 8.2 60fps Idle Micro-Motion Engine
Idle motion runs continuously and avoids robotic looping:
- **Breathing**: Procedural chest scale fluctuation:
  $$S_{ \text{chest}}(t) = 1.0 + 0.02 \cdot \sin\left(rac{2\pi t}{3.6} + \phi_{ \text{bird}}
ight)$$
- **Scanning & Head-Tilts**: Randomized micro-rotations ($-8^\circ$ to $+12^\circ$) triggered at randomized intervals (every 4–9 seconds). Curvature and duration derived from bird curiosity trait.
- **Preening**: Occasional multi-frame procedural preen sequence triggered when bird mood is `content`.
- **First Frame Rendering**: Upon canvas mount, the animation engine calculates $t_{ \text{initial}} = \text{Date.now}()$. Bird poses are initialized with $\phi_{ \text{bird}} = \text{hash}( \text{bird\_id}) \pmod{2\pi}$. The scene renders immediately in mid-motion without a black frame, fade-in, or loader animation.
- **Battery Optimization**: When `document.visibilityState === "hidden"`, the `requestAnimationFrame` render loop pauses completely.

### 8.3 Reduced-Motion Mode (`prefers-reduced-motion`)
When active (via OS media query or user accessibility setting):
- Particle system (falling leaves, drifting feathers) is completely disabled.
- Parallax cursor response is disabled.
- Continuous 60fps skeletal micro-motion is disabled.
- Micro-motion is replaced by **slow cross-fading still poses**: every 8–12 seconds, the bird smoothly cross-fades over 1.5 seconds to a new still posture (e.g., from scanning to relaxed).
- Perch transitions do not use animated flight trajectories; instead, the bird dissolves from the departure perch and gently appears at the destination perch via a 1.2-second cross-fade.
- Day/night lighting transitions remain active but are smoothed over longer intervals.

### 8.4 Top-Bar Chrome Auto-Fade
- Contains minimal icons: Account/Settings, Accessibility, Field Notebook, and Offer Tray.
- After 4.0 seconds of cursor stillness within the browser window, top-bar opacity ramps down to 0.08 over 1.5 seconds.
- Any pointer movement or keyboard interaction immediately restores opacity to 1.0 over 150ms.

---

## 9. Procedural Audio Pipeline (WebAudio)

### 9.1 Algorithmic Sound Generation Architecture
The audio engine operates strictly without audio files or sample loops:
- **WebAudio Node Topology**:
  ```
  [ Carrier Oscillator ] --(FM)--> [ WaveShaper / Modulator ]
                                              |
                                              v
                                  [ Biquad Formant Filter ]
                                              |
                                              v
                                   [ ADSR Gain Envelope ]
                                              |
                                              v
                                    [ StereoPannerNode ]
                                              |
                                  +-----------+-----------+
                                  |                       |
                                  v                       v
                          [ Bird GainNode ]      [ Convolver Reverb ]
                                  |                       |
                                  +-----------+-----------+
                                              |
                                              v
                                      [ Master GainNode ]
                                              |
                                              v
                                    [ AudioContext.dest ]
  ```

### 9.2 Species Acoustic Formulas
- **FM Carrier Synthesis**: Carrier frequency $f_c$ modulated by modulator $f_m$ with modulation index $eta$:
  $$y(t) = A(t) \sin(2\pi f_c t + eta \sin(2\pi f_m t))$$
- **Warbler Synthesis**: High-speed linear frequency sweep:
  $$f_c(t) = f_{ \text{start}} + (f_{ \text{end}} - f_{ \text{start}}) \cdot rac{t}{    au}$$
  accompanied by sharp resonant lowpass filter ($Q = 4.5$).
- **Wren Synthesis**: Dual-oscillator harmonic flutter with rapid square-wave amplitude chopping ($15 \text{Hz}$) producing a distinct dry trill.
- **Dove Coo**: Low triangular carrier ($f_c \approx 520 \text{Hz}$) with smooth sinusoidal tremolo ($5 \text{Hz}$, depth $0.15$) and heavy low-pass filtering at $900 \text{Hz}$.

### 9.3 Real-Time Chorus Mixing & Listen-In Dynamics
- **Chorus Superposition**: Because each call is synthesized mathematically with randomized pitch offsets ($\pm 15$ cents) and timing offsets, simultaneous calls blend naturally in the master mix without the acoustic phase cancellation characteristic of layered audio samples.
- **Spatial Positioning**: Each bird's `StereoPannerNode` maps its normalized perch X coordinate ($x \in [0.0, 1.0]$) to pan value $p \in [-0.75, +0.75]$.
- **Listen-In Interaction**:
  - When bird $B_{ \text{focus}}$ is selected:
    - $B_{ \text{focus}}$ GainNode: Smooth exponential ramp from $0 \text{dB}$ to $+4 \text{dB}$ over $1.5 \text{s}$.
    - All other birds $B_{j 
e \text{focus}}$ GainNodes: Smooth exponential ramp from $0 \text{dB}$ down to $-12 \text{dB}$ (the non-zero ambient floor) over $2.0 \text{s}$.
    - High-shelf filter attenuates frequencies above $6 \text{kHz}$ on non-focused birds by $-6 \text{dB}$ to push them perceptually into the background.
  - On disengagement:
    - All gains ramp back to unity ($0 \text{dB}$) over $2.0 \text{s}$.

### 9.4 Graceful WebAudio Fallback
- If `window.AudioContext` fails to initialize (unsupported browser, user audio permission denied, or hardware initialization failure):
  - The audio engine transitions to **graceful silence**.
  - No synthetic error sound or recorded sample fallback is loaded.
  - Call captions are automatically enabled in the presentation layer.

---

## 10. Accessibility Surfaces

### 10.1 Running Screen-Reader Narration
- Delivered via a dedicated visually hidden HTML region: `<div aria-live="polite" aria-atomic="true" class="sr-only">`.
- **Narration Engine Cadence**:
  - In idle state: Generates one observation sentence every 45–60 seconds.
  - On user actions: Dispatches immediate observation with prioritized queueing.
- **Prose Generator Rules**: Strictly naturalist, lowercase, present-tense, bird-centric:
  - *Idle Dawn*: `"it is early morning in the aviary. pip is perched on the front branch, calling softly."`
  - *Midday Preen*: `"wren perches on the high rail with feathers fluffed against the breeze."`
  - *Listen-In*: `"pip's three-note call rises above the ambient hush."`
  - *Offer Accepted*: `"pip hops to the front rail to investigate the offered seed."`
  - *Settle*: `"the light softens into evening. the aviary settles into quiet."`

### 10.2 Procedural Call Captions
- When captions are toggled on (or audio context is disabled):
  - Call captions appear as soft text bubbles adjacent to the calling bird.
  - Text is generated directly by the motif sequencer based on the active synthesis parameters:
    - Warbler sweep: `"a soft three-note rise"`
    - Wren trill: `"a low trill, paused, low trill again"`
    - Dove coo: `"a quiet resonant two-part coo"`
    - Finch chirp: `"a bright quick chirp from the middle perch"`
  - Text fades in over 200ms, persists for the duration of the audio call, and fades out over 400ms.

### 10.3 Keyboard Navigation & Focus Ring
- Full interactive control without a pointing device:
  - `Tab` / `Shift+Tab`: Cycles through Top-Bar controls and enters the aviary container.
  - When aviary container is focused:
    - `ArrowRight` / `ArrowLeft`: Move focus between birds in spatial horizontal order.
    - `ArrowUp` / `ArrowDown`: Move focus between front, middle, and back perch zones.
    - `Enter` / `Space`: Toggle listen-in on focused bird.
    - `Escape`: Cancel listen-in or close open overlays (notebook, settings).
    - `KeyO`: Open Offer menu tray.
    - `KeyS`: Trigger Settle gesture (with 5-second cancel affordance on any keypress).
    - `KeyN`: Open Field Notebook drawer.
- **Focus Indicator**: 2px solid border with 2px offset in `#FFFFFF` with an outer `#000000` drop-shadow, ensuring WCAG AA visibility against bright midday sky and dark evening backdrops.

### 10.4 Contrast Verification
- All UI chrome text, captions, and notebook entries adhere to WCAG 2.1 AA:
  - Normal text: Minimum contrast ratio of 4.5:1 against background.
  - Large text / icons: Minimum contrast ratio of 3.0:1.
  - Top-bar icons use SVG paths with solid fill and high-contrast bounding backing.

---

## 11. Performance Budgets & Observability Boundary

### 11.1 Performance Budgets

| Metric | Budget Target | Measurement Method & Condition |
|---|---|---|
| **Initial JS Bundle Size** | $< 2.0 \text{ MB}$ (gzipped) | Production build output at entrypoint; verified in CI. |
| **Time to First Bird (TTFBird)** | $< 500 \text{ ms}$ | Mid-tier mobile device (Moto G4 class) over simulated 4G LTE. |
| **Idle Render Frame Rate** | $60 \text{ fps}$ continuous | 5-year-old mid-range laptop (Intel UHD 620); 30-minute session. |
| **Client Memory Growth** | $0 \text{ MB}$ leak over 30 mins | Chrome DevTools heap allocation profiler in automated test harness. |
| **Simulation Tick Latency (p99)** | $< 5.0 \text{ seconds}$ | Server worker APM metric across all active aviaries. |

### 11.2 Architectural Bundle & Load Optimization
- **Critical Path HTML Inlining**: The initial aviary snapshot JSON is inlined directly into the HTML document at the edge worker. The client render engine does not wait for a secondary network fetch before painting the first frame.
- **Aggressive Code Splitting**:
  - `bundle-core.js` (<350 KB): Canvas renderer, WebAudio synthesis engine, presence sentinel, snapshot parser.
  - `bundle-settings.js` (lazy-loaded on click): Magic-link account management, session revocation, export.
  - `bundle-notebook.js` (lazy-loaded on click): Infinite scroll notebook history viewer.
- **Zero Recorded Audio Assets**: Eliminating sample libraries saves ~15–30 MB of audio data, easily fitting within the 2MB budget.

### 11.3 Strict Privacy Boundary & Observability
- **Hard Architectural Partition**:
  - *Allowed Telemetry*: HTTP request rate, response latency, simulation worker tick duration, edge CDN cache hit ratio, client-side average FPS, dropped frame percentage, WebAudio initialization error count.
  - *Strictly Prohibited Telemetry*: Bird names, species selections, personality vector values, mood states, individual presence durations, notebook entry texts, offer choices, or visitor email addresses.
- **Logging Rule**: Application logs contain synthetic `account_id` and `aviary_id` UUIDs only. User email addresses are never written to standard output or logging pipelines.

---

## 12. Social Visits Flow & Implementation

### 12.1 Invitation Lifecycle
1. **Creation**: Host opens Settings $    o$ Invite a Friend. Enters visitor email.
2. **Token Generation**: Server creates a cryptographically secure 256-bit token. Stores SHA-256 hash in `visit_invitations` table with `expires_at = NOW() + INTERVAL '30 days'`.
3. **Dispatch**: Server emails visitor a link: `https://pocketaviary.com/visit/:token`.
4. **Access**: Visitor opens link. The edge gateway validates token hash and checks `revoked_at IS NULL` and `expires_at > NOW()`.
5. **Session**: Visitor client connects to read-only snapshot endpoint `/api/v1/visits/view/:token` and SSE channel `/api/v1/visits/stream/:token`.
6. **Revocation**: Host clicks "Revoke" in Visit Log. Server sets `revoked_at = NOW()`. The next SSE heartbeat terminates visitor stream immediately, rendering the matter-of-fact message: `"This visit invitation is no longer active."`

### 12.2 Read-Only Ambient Constraints
- Visitors receive identical visual and audio rendering of the host's aviary (same birds, perches, moods, weather, daylight phase).
- **Interaction Stripping**: The visitor interface contains zero UI controls (no offer tray, no settle button, no listen-in click handlers, no bird renaming).
- **Zero Presence Attribution**: Visitor sessions emit no `presence_ping` events. Visitor attention generates zero drift for the host's birds.
- **No Co-Presence Markers**: Host has no visual or audio indication that a visitor is watching.
- **Notifications Disabled by Default**: Visits are logged silently in `visit_logs`. The host receives no push notification or email unless explicitly opted in via account settings.

---

## 13. Implementation Phases, Rollout & Capacity Ramp

### 13.1 Implementation Milestones

```
+-----------------------------------------------------------------------------------+
| MILESTONE 1: Simulation Core & Data Engine (Weeks 1-4)                            |
| - PostgreSQL schemas, synthetic UUID isolation, AES-256 email encryption.         |
| - 60s Simulation Worker Daemon: monotonic drift LPF, diurnal solar clock.         |
| - Magic-link authentication service and session token management.                 |
+-----------------------------------------------------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
| MILESTONE 2: WebAudio Procedural Synthesis (Weeks 5-7)                            |
| - Procedural synthesis recipes for 6 bird species (FM + Formant filtering).       |
| - Real-time chorus mixer, stereo positioning, and listen-in gain curves.          |
| - Graceful silence fallback and motif-to-caption generator.                       |
+-----------------------------------------------------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
| MILESTONE 3: Canvas Rendering & Micro-Motion (Weeks 8-10)                         |
| - 60fps responsive Canvas pipeline, 3 perch zones, dynamic sky gradient.          |
| - Skeletal idle micro-motion engine with randomized phase zero-delay paint.        |
| - Reduced-motion cross-fade rendering mode.                                       |
+-----------------------------------------------------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
| MILESTONE 4: Interactions, Field Notebook & Accessibility (Weeks 11-13)           |
| - Absence-modulated return greeting and presence sentinel (3-factor rule).        |
| - Offer interaction tray and Settle evening transition with 5s undo.             |
| - ARIA live polite naturalist prose narrator and full keyboard navigation.        |
+-----------------------------------------------------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
| MILESTONE 5: Social Visits, Hardening & Verification (Weeks 14-16)                |
| - Read-only ambient visit flow and instant revocation.                            |
| - 30-minute memory leak stress testing and bundle size budget enforcement.        |
| - End-to-end integration and drift calibration verification.                      |
+-----------------------------------------------------------------------------------+
```

### 13.2 Aviary Capacity Ramp (Age-Based Unlocks)
To preserve the sense of relationship depth, bird capacity expands strictly by account age:
- **Day 0 (Adoption)**: 2 Starter Birds (system-selected from 6-species pool).
- **Day 30**: 3rd Bird arrives.
- **Day 90**: 4th Bird arrives.
- **Day 180**: 5th Bird arrives.
- **Day 270**: 6th Bird arrives.
- **Day 360**: 7th Bird arrives (Hard Capacity Ceiling).
*No mechanic exists to accelerate this progression via interaction volume, streaks, or purchases.*

---

## 14. Technical & Design Risks and Mitigations

### 14.1 Drift Calibration Risk (Drifting Too Fast or Too Slow)
- **Risk**: If the low-pass filter coefficient is miscalibrated, birds could either reach maximum boldness in 3 days (feeling robotic and game-like) or show zero change after 2 months (feeling static and unresponsive).
- **Mitigation**: Implement automated headless simulation test suites in CI that execute accelerated virtual timelines (1-day, 1-week, 3-week, 6-month, and 1-year profiles). Assert that standard deviation and delta milestones match targets ($\Delta T \approx +0.03$ at 1 week, $+0.10$ at 3 weeks) before shipping engine changes.

### 14.2 Presence Spoofing & Background Drain
- **Risk**: Users leaving open browser tabs indefinitely in background windows, falsely accumulating weeks of presence time and distorting drift.
- **Mitigation**: Strict enforcement of the 3-factor conjunction rule in the client Presence Sentinel (`document.visibilityState === "visible"` AND `window.hasSubframeFocus` AND user input within 180 seconds). The server drops heartbeats lacking client proof-of-activity.

### 14.3 Multi-Device State Inconsistency & Split-Brain
- **Risk**: A user alternating between phone and laptop might experience disjointed bird moods or lost drift progress.
- **Mitigation**: Clients are strictly prohibited from holding authoritative state or computing deltas. The server simulation tick is the sole writer to the database. All client actions are append-only events processed in FIFO order.

### 14.4 Audio Uncanniness & Acoustic Fatigue
- **Risk**: Algorithmic FM synthesis could sound metallic, shrill, or fatiguing over extended 30-minute listening sessions.
- **Mitigation**:
  - Incorporate bioacoustic formant frequencies measured from real bird vocalizations.
  - Apply organic micro-randomization to pitch ($\pm 15$ cents) and timing ($\pm 8\%$).
  - Maintain a non-zero ambient floor ($-12 \text{dB}$) during listen-in to prevent unnatural silence.
  - Test audio across a variety of hardware speakers and headphones.

### 14.5 Screen-Reader Live Region Flooding
- **Risk**: Rapid state updates could flood the screen reader's speech queue, rendering the experience annoying and unusable.
- **Mitigation**:
  - Throttle idle narration to a strict 45–60 second cadence.
  - Use `aria-live="polite"` so screen readers finish current announcements before reading aviary prose.
  - Ensure all generated narration text strictly follows the naturalistic, calm, field-notebook style.

### 14.6 Memory Leaks Over Long Sessions
- **Risk**: Continuously running 60fps canvas animation and dynamic WebAudio node creation could cause heap memory to climb, crashing browser tabs after 20–30 minutes.
- **Mitigation**:
  - Implement fixed-capacity object pools for ambient canvas particles (leaves, feathers).
  - Pre-allocate and reuse WebAudio AudioNode graphs per bird; modulate parameters via `AudioParam.setValueAtTime` rather than instantiating new oscillator nodes on every call.
  - Enforce CI performance test running a 30-minute automated session with zero net heap growth.
