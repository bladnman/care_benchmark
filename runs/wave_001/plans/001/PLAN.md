# Pocket Aviary — Comprehensive Engineering Implementation Plan (v1)

## 1. Executive Summary & Scope Boundary

Pocket Aviary is a tranquil, browser-based virtual aviary designed as an observational relationship rather than a gamified application. Users adopt two starter birds (scaling up to a maximum of seven over months based purely on aviary age) inhabiting a single horizontal scene rendered directly in modern evergreen web browsers. The birds notice the user; over weeks of presence and gentle interaction, hidden personality traits drift monotonically toward expressiveness. 

### 1.1 In-Scope for v1
- **Single-User Accounts & Auth:** Email-based magic link authentication (15-minute token TTL, single-use, session token generation with device revocation). Single aviary per account.
- **Bird Population Mechanics:** Two starter birds automatically assigned from a 6-species pool at onboarding; naming flow at onboarding and via settings. Population growth capped at 7 birds, paced strictly by aviary chronological age.
- **Core Interactions:**
  - *Presence Accounting:* Strict conjunction of three signals: `document.visibilityState === "visible"`, active window focus (`hasWindowFocus`), and user activity (pointer movement or keypress) within an idle window ($T_{idle} \le 180\text{s}$).
  - *Return-Greeting:* Procedural, non-uniform greeting by one bird within 1–2 seconds of session return, modulated by absence duration, boldness, and current mood.
  - *Listen-in:* Focusing a single bird via pointer or keyboard navigation; smooth exponential audio mix rebalance (+6dB focused bird, -8dB ambient birds over 1.5s) without full mute.
  - *Offers:* Three gift affordances (seed, song fragment, still pool) initiated from the top bar with a per-bird cooldown (3 minutes) and reactions shaped by curiosity and mood.
  - *Settle Gesture:* Soft session-end gesture transitioning scene lighting to evening over 3–5 seconds and quieting calls; includes a 5-second undo grace window.
  - *Field Notebook:* Read-only naturalist observation log generated server-side at sparse intervals (~1 entry every 2–4 days), recording unique moments in lowercase present-tense prose.
- **Social Affordance (Visits):** Opt-in, read-only ambient visits shared via revocable email magic link (30-day expiration). Zero co-presence, zero interaction from visitor, zero presence/drift recorded from visits. Default disabled.
- **Rendering Pipeline:** Responsive single horizontal scene with 3 perch zones (front, middle, back), continuous local-time day/night lighting cycle, rare ambient weather (gentle rain/wind), ambient leaf/feather drift, and zero loading spinners (instant render or quiet sky field).
- **Audio Pipeline:** Client-side procedural call synthesis via WebAudio API modeling avian syrinx dynamics; dynamic chorus mixing; graceful fallback to silence with auto-enabled captions if WebAudio is unavailable. No recorded audio loops.
- **Accessibility & Inclusion:** First-class accessibility surfaces including continuous naturalist screen-reader narration (`aria-live="polite"`), reduced-motion mode (cross-fading still poses, no leaf drift), dynamic call captions, full keyboard navigation, and WCAG AA contrast compliance.
- **Sync & Simulation:** Server-side simulation tick (~60s interval) advancing canonical state independently of client connections; additive server-authored personality deltas; multi-device read consistency.

### 1.2 Out-of-Scope (Non-Goals)
To preserve the emotional integrity and core thesis of Pocket Aviary, the following features are strictly prohibited:
- **No Native Applications:** Web-only (desktop and mobile browser viewports). No iOS/Android native packages or wrappers.
- **No Gamification Elements:** Zero streaks, zero visit counters, zero green-dot calendars, zero XP, zero achievements, zero badges, zero levels, zero adoption counters. The system never measures or reflects back user engagement habits.
- **No Tamagotchi / Custodial Mechanics:** Birds cannot starve, sicken, or die. Neglect never causes distress, sadness, or negative personality regression; it merely results in ambient quietness. No hunger meters or custodial chores.
- **No Social Network Mechanics:** No public discovery feeds, no searchable directories, no user profiles, no avatars, no comments/guestbooks, no leaderboards, no follower graphs, and no real-time co-presence.
- **No Push/Engagement Notifications:** Zero browser push notifications, marketing emails, or retention reminders. The aviary exists only where and when the user opens the window.

### 1.3 Voice and Tone Separation Architecture
System UI and copy are strictly partitioned into two mutually exclusive linguistic registers:
1. **Naturalist Voice (Product Surface):** Used across aviary visual scene, field notebook entries, screen-reader narration, and call captions. Characteristics: strictly lowercase, present-tense, bird-named, observational, evocative, devoid of exclamation marks, gamified terminology, or second-person directives (*"pip is on the low perch this morning, fluffed against the cool air"*).
2. **Matter-of-Fact Voice (System Surface):** Used across authentication flows, account management, sync conflict resolution, error boundaries, and accessibility settings. Characteristics: standard sentence capitalization, concise, clear, helpful, neutral, devoid of false warmth or simulated naturalist charm (*"We could not sign you in. The link may have expired. Try requesting a new link."*).

---

## 2. System Architecture & Component Topology

```
+------------------------------------------------------------------------------------+
|                                BROWSER CLIENT                                     |
|                                                                                    |
|  +------------------------+  +------------------------+  +-----------------------+ |
|  |     Rendering Engine   |  |   WebAudio Syrinx Synth|  |  Presence Monitor     | |
|  | (Canvas2D / WebGL Graph|  | (Procedural Synthesis, |  | (VisibilityState,     | |
|  |  60fps Interpolator)   |  |  Chorus Mixer, Decay)  |  |  Focus, User Activity)| |
|  +------------------------+  +------------------------+  +-----------------------+ |
|             ^                            ^                           |             |
|             |                            |                           v             |
|  +-------------------------------------------------------------------------------+ |
|  |                  Client State Manager & Sync Agent                            | |
|  |        (Snapshot Cache, Optimistic Event Queue, A11y Narration Driver)        | |
|  +-------------------------------------------------------------------------------+ |
+------------------------------------------^-----------------------------------------+
                                           | HTTPS / JSON
                                           v
+------------------------------------------------------------------------------------+
|                                EDGE / API GATEWAY                                  |
|     (TLS Termination, Edge Snapshot Caching, Rate Limiting, PII Scrubbing)         |
+------------------------------------------+-----------------------------------------+
                                           |
    +--------------------------------------+-----------------------------------+
    |                                                                          |
    v                                                                          v
+-----------------------+  +-----------------------+  +----------------------------+
|     Auth Service      |  |  Aviary State & Event  |  |   Visit Gateway Service    |
| (Magic Link, Sessions,|  |        Service         |  | (Read-Only Token Validation|
|  Device Revocation)   |  | (Append Event Stream,  |  |  Zero-Presence Filter)     |
|                       |  |  Snapshot Query)       |  |                            |
+-----------+-----------+  +-----------+-----------+  +-------------+--------------+
            |                          |                            |
            +--------------------------+----------------------------+
                                       |
                                       v
                     +-----------------------------------+
                     |     PostgreSQL Primary Cluster    |
                     |  (Accounts, Birds, Personalities, |
                     |   Append-Only Interaction Events, |
                     |   Notebook Entries, Visit Logs)   |
                     +-----------------+-----------------+
                                       ^
                                       | Read/Write
                     +-----------------+-----------------+
                     |    Simulation Engine Worker       |
                     | (~60s Tick, Low-Pass Drift Filter,|
                     |  Markov Mood Engine, Age Unlock,  |
                     |  Naturalist Notebook Generator)   |
                     +-----------------------------------+
```

### 2.1 Service Boundaries & Responsibilities
1. **Client Application (SPA):** Single-page web application compiled to a lean, self-contained bundle (<2MB gzipped). Responsible for rendering the scene graph at 60fps, synthesizing procedural audio in real time via WebAudio, tracking strict 3-condition presence, handling keyboard/pointer inputs, and presenting accessible DOM projections.
2. **Edge API Gateway:** Reverse proxy handling TLS, security headers, rate limiting (per-IP and per-account), and CDN caching for static assets. Extracts synthetic account UUIDs from verified session tokens; scrubs all PII from operational logging.
3. **Auth Service:** Issues 15-minute single-use magic links, verifies HMAC-SHA256 signature tokens, issues cryptographically secure session cookies/bearer tokens, and manages device session records.
4. **Aviary State & Event Service:** Ingests append-only client interaction events into PostgreSQL; serves hydrated aviary snapshots (`GET /api/v1/aviary/snapshot`) to authorized owners and read-only visitors.
5. **Simulation Engine Worker:** Background distributed daemon executing a deterministic simulation tick (~60-second cycle). Evaluates accumulated interaction events, advances mood state machines, applies the monotonic low-pass filter drift function, triggers species unlock checks based on account age, and generates sparse naturalist field notebook entries.
6. **Transactional Notification Service:** Headless worker processing transactional email delivery for magic links, visit invites, and account data exports via external SMTP/SES. Strictly fire-and-forget; completely isolated from analytics.

### 2.2 Client/Server Boundary Invariants
- **Client is a Projection:** The client never runs the simulation tick and never mutates personality or mood directly. The client receives canonical state snapshots, interpolates between positions, and drives local micro-motions (e.g. idle breathing, preening, feather flutter).
- **Server is Canonical:** All personality vector drifts, mood shifts, aviary unlocks, and field notebook entries are authored exclusively by the server simulation worker.
- **Append-Only Interactions:** Clients only report factual observations of what the user did (`presence_ping`, `listen_in_start`, `offer_made`, `settle_triggered`). The server interprets these events into numerical deltas.

---

## 3. Data Model & Storage Specifications

### 3.1 Synthetic Account ID & PII Airgap
In strict conformance with `accounts_sync.md`, internal database schemas, inter-service messaging, cache keys, partition tokens, and telemetry tags MUST use a synthetic `account_id` (UUIDv4). The user's email address is stored exclusively in the `accounts` table encrypted at rest using AES-256-GCM. No database queries outside the auth service may join on or inspect email addresses.

### 3.2 Relational Database Schema (PostgreSQL DDL)

```sql
-- Core Account & Auth
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_hash VARCHAR(64) NOT NULL UNIQUE, -- HMAC-SHA256 for lookup without decryption
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    soft_deleted_at TIMESTAMPTZ NULL,
    visit_notifications_enabled BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE TABLE auth_magic_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    expires_at TIMESTAMPTZ NOT NULL,
    used_at TIMESTAMPTZ NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE user_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    session_token_hash VARCHAR(64) NOT NULL UNIQUE,
    device_label VARCHAR(128) NOT NULL,
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at TIMESTAMPTZ NULL
);

-- Aviary and Birds
CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    settled_until TIMESTAMPTZ NULL,
    current_weather VARCHAR(32) NOT NULL DEFAULT 'clear',
    weather_started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    weather_expires_at TIMESTAMPTZ NOT NULL DEFAULT NOW() + INTERVAL '4 hours'
);

CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL, -- e.g. 'grey_warbler', 'spotted_towhee'
    name VARCHAR(64) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    perch_zone VARCHAR(16) NOT NULL DEFAULT 'middle', -- 'front', 'middle', 'back'
    perch_index INT NOT NULL DEFAULT 0,
    visual_seed INT NOT NULL,
    call_seed INT NOT NULL,
    CONSTRAINT check_perch_zone CHECK (perch_zone IN ('front', 'middle', 'back'))
);

-- Hidden Personality Vectors (Server Canonical Only, Never Exposed to Client)
CREATE TABLE bird_personalities (
    bird_id UUID PRIMARY KEY REFERENCES birds(id) ON DELETE CASCADE,
    boldness NUMERIC(5,4) NOT NULL DEFAULT 0.5000,          -- [0.0000, 1.0000]
    social_warmth NUMERIC(5,4) NOT NULL DEFAULT 0.5000,     -- [0.0000, 1.0000]
    vocal_frequency NUMERIC(5,4) NOT NULL DEFAULT 0.5000,   -- [0.0000, 1.0000]
    plumage_saturation NUMERIC(5,4) NOT NULL DEFAULT 0.5000,-- [0.0000, 1.0000]
    curiosity NUMERIC(5,4) NOT NULL DEFAULT 0.5000,         -- [0.0000, 1.0000]
    last_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT check_personality_ranges CHECK (
        boldness BETWEEN 0 AND 1 AND
        social_warmth BETWEEN 0 AND 1 AND
        vocal_frequency BETWEEN 0 AND 1 AND
        plumage_saturation BETWEEN 0 AND 1 AND
        curiosity BETWEEN 0 AND 1
    )
);

-- Fast-Timescale Mood States
CREATE TABLE bird_moods (
    bird_id UUID PRIMARY KEY REFERENCES birds(id) ON DELETE CASCADE,
    current_mood VARCHAR(16) NOT NULL DEFAULT 'content',
    mood_intensity NUMERIC(3,2) NOT NULL DEFAULT 0.50,
    mood_entered_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT check_mood_enum CHECK (current_mood IN ('wary', 'content', 'curious', 'drowsy', 'alert'))
);

-- Append-Only Interaction Event Log
CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    session_id UUID NULL REFERENCES user_sessions(id) ON DELETE SET NULL,
    event_type VARCHAR(32) NOT NULL,
    payload JSONB NOT NULL DEFAULT '{}',
    client_timestamp TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_by_tick_at TIMESTAMPTZ NULL
);
CREATE INDEX idx_interaction_events_unprocessed ON interaction_events (aviary_id, id) WHERE processed_by_tick_at IS NULL;

-- Field Notebook Observations
CREATE TABLE field_notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    entry_prose TEXT NOT NULL,
    observed_bird_ids JSONB NOT NULL DEFAULT '[]',
    observation_context VARCHAR(32) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_notebook_aviary_created ON field_notebook_entries (aviary_id, created_at DESC);

-- Optional Social Visits
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    visitor_email_encrypted BYTEA NOT NULL,
    visitor_email_hash VARCHAR(64) NOT NULL,
    invite_token_hash VARCHAR(64) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL DEFAULT NOW() + INTERVAL '30 days',
    revoked_at TIMESTAMPTZ NULL
);

CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invitation_id UUID NOT NULL REFERENCES visit_invitations(id) ON DELETE CASCADE,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    visited_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    duration_seconds INT NOT NULL DEFAULT 0
);
```

### 3.3 Telemetry Isolation & Data Privacy Architecture
- **Complete Airgap:** The operational time-series database (Prometheus/OpenTelemetry) is strictly prohibited from accepting any payload originating from `interaction_events`, `bird_personalities`, or `field_notebook_entries`.
- **Metrics Collected:** Request rate, HTTP latency histograms, simulation tick processing duration, client render FPS aggregate percentiles, WebAudio audio context error counts, memory heap size distributions.
- **Excluded Telemetry:** Account identifiers, email domains, bird count, individual bird traits, user presence duration per account, offer frequencies.

---

## 4. API Surface & Contract Specifications

### 4.1 Authentication & Account Management
All endpoints return standard HTTP status codes. Errors are returned formatted with matter-of-fact text.

#### `POST /api/v1/auth/magic-link`
- **Request:** `{"email": "naturalist@example.com"}`
- **Response (200 OK):** `{"message": "If this address is registered, a sign-in link has been sent."}`
- **Rate Limit:** 5 requests per IP / email per hour.

#### `POST /api/v1/auth/verify`
- **Request:** `{"token": "c7a8b89e...f1", "device_label": "Firefox on macOS"}`
- **Response (200 OK):** Sets `HttpOnly; Secure; SameSite=Lax` session cookie. Body: `{"session_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d"}`
- **Response (401 Unauthorized):** `{"error": "We could not sign you in. The link may have expired. Try requesting a new link."}`

#### `POST /api/v1/auth/session/revoke`
- **Request:** `{"session_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d"}`
- **Response (200 OK):** `{"status": "session_revoked"}`

#### `POST /api/v1/account/export`
- **Response (202 Accepted):** `{"message": "Your aviary archive is being prepared and will be sent to your email address."}`

#### `DELETE /api/v1/account`
- **Response (200 OK):** Marks `soft_deleted_at = NOW()`. Body: `{"message": "Your account is scheduled for deletion in 30 days. Signing in before then will restore your aviary."}`

### 4.2 Aviary State & Interaction Endpoints

#### `GET /api/v1/aviary/snapshot`
Returns the canonical state required for client-side rendering. Note: personality vectors are stripped; only visible attributes (perch, mood enum, plumage saturation scalar) are included.
- **Headers:** `Authorization: Bearer <token>` or `Cookie: session=...`
- **Response (200 OK):**
```json
{
  "aviary": {
    "id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
    "server_time": "2026-09-06T15:04:05.123Z",
    "timezone": "America/Los_Angeles",
    "settled": false,
    "weather": {
      "type": "clear",
      "intensity": 0.0
    }
  },
  "birds": [
    {
      "id": "e4b02b5e-8511-4777-a89c-a5b6d9e5b021",
      "name": "pip",
      "species_id": "grey_warbler",
      "perch_zone": "front",
      "perch_index": 1,
      "mood": "content",
      "plumage_saturation": 0.54,
      "visual_seed": 42189,
      "call_seed": 91823,
      "vocal_frequency_tier": 0.62
    },
    {
      "id": "c1f72a44-2451-4f76-857c-2b28cf9b4412",
      "name": "wren",
      "species_id": "spotted_towhee",
      "perch_zone": "back",
      "perch_index": 0,
      "mood": "drowsy",
      "plumage_saturation": 0.51,
      "visual_seed": 11029,
      "call_seed": 55192,
      "vocal_frequency_tier": 0.40
    }
  ],
  "return_greeting": {
    "greeter_bird_id": "e4b02b5e-8511-4777-a89c-a5b6d9e5b021",
    "greeting_motif_id": "greeting_soft_two_note",
    "stagger_delay_ms": 1250
  }
}
```

#### `POST /api/v1/aviary/events`
Batch upload of interaction events logged while the tab was active.
- **Request:**
```json
{
  "events": [
    {
      "event_type": "presence_ping",
      "client_timestamp": "2026-09-06T15:04:00.000Z",
      "payload": { "duration_seconds": 60 }
    },
    {
      "event_type": "listen_in_start",
      "client_timestamp": "2026-09-06T15:04:15.000Z",
      "payload": { "bird_id": "e4b02b5e-8511-4777-a89c-a5b6d9e5b021" }
    },
    {
      "event_type": "offer_made",
      "client_timestamp": "2026-09-06T15:04:45.000Z",
      "payload": { "offer_type": "seed", "target_perch_zone": "front" }
    }
  ]
}
```
- **Response (200 OK):** `{"processed": 3}`

#### `GET /api/v1/aviary/notebook`
Returns historical naturalist log entries (paginated, infinite scroll backward).
- **Response (200 OK):**
```json
{
  "entries": [
    {
      "id": "1a2b3c4d-...",
      "timestamp": "2026-09-06T08:12:00.000Z",
      "text": "pip greeted before wren today, first time this week."
    },
    {
      "id": "5e6f7a8b-...",
      "timestamp": "2026-09-03T17:40:00.000Z",
      "text": "a long stretch of quiet this morning. pip preened for several minutes without looking up."
    }
  ],
  "next_cursor": "2026-09-03T17:40:00.000Z"
}
```

### 4.3 Social Visit API Surface

#### `POST /api/v1/visits/invite`
- **Request:** `{"visitor_email": "friend@example.com"}`
- **Response (201 Created):** `{"invite_id": "...", "expires_at": "2026-10-06T15:04:05Z"}`

#### `DELETE /api/v1/visits/invite/:invite_id`
- **Response (200 OK):** `{"status": "revoked"}`

#### `GET /api/v1/visits/view/:token`
Public-facing, read-only snapshot for invited guests.
- **Behavior:** Validates token against `visit_invitations` (checking expiry and revocation). If invalid, returns 404 with matter-of-fact text: `{"error": "This visit invitation is no longer available."}`.
- **Output:** Identical payload to `/api/v1/aviary/snapshot` except all interaction actions are disabled; `POST /events` is completely blocked for this session. The visit duration is logged to `visit_logs` upon disconnect without recording presence or triggering drift.

---

## 5. Simulation Engine Design & Drift Dynamics

The simulation engine is the persistent heart of Pocket Aviary. It runs as a distributed background worker executing a tick every 60 seconds per aviary.

```
+-------------------------------------------------------------------------------+
|                       SIMULATION ENGINE TICK (~60s)                           |
|                                                                               |
|  1. Fetch Unprocessed Events (presence_pings, listen_ins, offers, settles)    |
|  2. Calculate Input Vector:                                                   |
|     - Presence attention duration (T_pres)                                    |
|     - Listen-in focus duration per bird (T_listen)                            |
|     - Accepted offer gestures (N_offer)                                       |
|  3. Apply Monotonic Low-Pass Drift Filter to Personality Vectors              |
|  4. Evaluate Mood State Transitions (Markov chain + context + contagion)      |
|  5. Check Aviary Age Milestone -> Spawn New Bird Species if Eligible          |
|  6. Evaluate Field Notebook Observation Criteria -> Generate Naturalist Prose |
|  7. Commit Canonical Aviary State Snapshot to DB                             |
+-------------------------------------------------------------------------------+
```

### 5.1 Drift Function Mathematics & Calibration
Personality drift acts as a very slow, leaky low-pass filter on positive attention signals. Crucially, drift is **monotonic toward expressive**: traits increase with positive interaction and presence, but **never decrease** due to absence or neglect.

#### Formal Mathematical Definition
Let $\mathbf{P}_i(t) = [B_i(t), W_i(t), F_i(t), S_i(t), C_i(t)]^T$ be the personality vector for bird $i$ at tick $t$, representing Boldness, Social Warmth, Vocal Frequency, Plumage Saturation, and Curiosity, bounded within $[0.0, 1.0]$.

For each tick interval $\Delta t \approx 1\text{ minute}$:
$$\mathbf{P}_i(t + \Delta t) = \mathbf{P}_i(t) + \Delta \mathbf{P}_i(t)$$
Where the delta is strictly non-negative:
$$\Delta \mathbf{P}_i(t) = \max\left(0, \mathbf{K} \cdot \mathbf{I}_i(t) \cdot \left(1.0 - \mathbf{P}_i(t)\right)\right)$$
- $(1.0 - \mathbf{P}_i(t))$ creates asymptotic natural saturation, preventing runaway explosion.
- $\mathbf{I}_i(t) = [I_{pres}, I_{listen, i}, I_{offer, i}, I_{ambient}]^T$ is the normalized input vector for this interval:
  - $I_{pres} = \min(1.0, rac{\text{valid presence seconds}}{60})$
  - $I_{listen, i} = \min(1.0, rac{\text{listen seconds on bird } i}{60})$
  - $I_{offer, i} = \text{count of accepted offers near bird } i$ (capped at 1 per cooldown)
- $\mathbf{K}$ is the calibrated coupling matrix:
$$\mathbf{K} = egin{bmatrix}
k_{B, pres} & 0 & k_{B, offer} & 0 \
k_{W, pres} & k_{W, listen} & 0 & 0 \
0 & k_{F, listen} & 0 & k_{F, ambient} \
k_{S, pres} & 0 & 0 & 0 \
0 & 0 & k_{C, offer} & 0
\end{bmatrix}$$

#### Empirical Drift Calibration Targets
- **Session Scale (1 day, ~15 mins presence):** $\Delta P \approx 0.001 - 0.003$. Completely imperceptible to the human eye/ear; eliminates any Pavlovian feedback loop.
- **Instrument Measurability Target (1 week of daily 15-minute visits):** $\Delta P \approx 0.02 - 0.04$. Detectable by automated test assertions and telemetry instruments, but subtle enough that the user experiences no sudden jump.
- **User Perception Target (3 weeks of regular visits):** $\Delta P \approx 0.10 - 0.15$. The bird visibly perches closer to the front, feather color is noticeably richer, calls are slightly more frequent, and response to offers is markedly bolder.
- **Neglect Behavior:** If unvisited for weeks, $I_{pres} = 0 \implies \Delta \mathbf{P} = 0$. The bird does not revert or regress; it remains at its earned trait values.

### 5.2 Mood State Machine & Dynamic Contagion
Mood operates on a fast timescale ($\approx 10\text{ minutes}$ to $24\text{ hours}$). The mood states are:
- `wary`: Elevated vigilance, perches further back, infrequent soft calls, cautious reactions to offers.
- `content`: Relaxed posture, preening idle motion, frequent harmonic calls, approaches offers smoothly.
- `curious`: Tilts head toward sounds, approaches front perch, actively investigates offers and still pool.
- `drowsy`: Low stance, fluffed plumage, slow blinking, long pauses between soft calls.
- `alert`: Upright neck stretch, scanning horizon, rapid call responses.

#### Transition Factors:
1. **Local Time of Day:** Sun calculation based on user timezone. Dusk shifts weights toward `drowsy` (70% probability at night). Dawn shifts weights toward `alert` and `content`.
2. **Ambient Weather:** Rain events increase `wary` (or quiet `drowsy`) probability by +30% and reduce vocalization rate across all birds by 50%.
3. **Recent Interactions:** Accepting a seed offer nudges mood directly to `content` or `curious`. A rapid series of unhandled UI movements nudges wary birds toward `wary`.
4. **Social Contagion:** If bird $A$ enters `wary` or sounds an alarm call, neighboring birds on adjacent perches have a 45% probability of transitioning to `wary` on the subsequent tick.

### 5.3 Bird Species Pool & Population Scaling
The v1 species pool consists of 6 distinct ornithological archetypes:
1. `grey_warbler`: Small, agile, high-frequency procedural warble, moderate boldness.
2. `spotted_towhee`: Ground/low-perch feeder, two-note call with trill, high curiosity.
3. `black_capped_chickadee`: Social, rapid greeting response, high vocal frequency.
4. `hermit_thrush`: Melodic flute-like harmonics, prefers back perch, low initial boldness.
5. `mourning_dove`: Drowsy, low soft cooing, high plumage drift responsiveness.
6. `nightjar`: Active late evening/night, rhythmic rhythmic mechanical trill, nocturnal specialist.

#### Age-Based Adoption Pacing
To ensure birds represent relationships deepened over time rather than unlocked achievements:
- **Day 1 (Account Creation):** Exactly 2 starter birds randomly selected from the diurnal pool (warbler, towhee, chickadee).
- **Month 2 (Day 60):** Third bird arrives at dawn.
- **Month 4 (Day 120):** Fourth bird arrives.
- **Month 7 (Day 210):** Fifth bird arrives.
- **Month 10 (Day 300):** Sixth bird arrives.
- **Month 14 (Day 420):** Seventh (and final) bird arrives.
*No user clicks, purchase triggers, or engagement volume can accelerate this timeline.*

### 5.4 Naturalist Field Notebook Generation Engine
The simulation engine evaluates aviary events every tick. If a noteworthy naturalist moment occurs, an entry is generated.
- **Rate Limit:** Maximum 1 entry per 48–72 hours of real time, even under continuous high presence.
- **Grammar Engine:** Selects from parameterized naturalist observation templates.
- **Trigger Heuristics:**
  - First Greeter Shift: Bird $A$ greets before Bird $B$ for the first time in $\ge 7$ days.
  - Weather Introspection: A rain event passes while both birds remain perched close together.
  - Extended Quietness: A session with $\ge 20$ minutes of continuous presence with zero offers made.
  - Perch Shift: A historically back-perch bird perches on the front rail for over 15 minutes.
- **Style Invariant:** Strictly lowercase, present-tense, bird-named, no exclamation marks (*"thursday — a long stretch of quiet this morning. wren stayed on the low branch, feathers ruffled against the wind."*).

---

## 6. Multi-Device Sync & Conflict Prevention

### 6.1 Architectural Principle: Single Canonical Server
Pocket Aviary completely rejects client-side simulation, distributed client clocks, and client-side last-write-wins (LWW) conflict resolution for personality data.
- **Server is Canonical:** Only the server simulation worker writes to `bird_personalities` and `bird_moods`.
- **Clients Pull Snapshots:** Clients pull snapshots at key lifecycle events:
  1. Initial page load.
  2. Document visibility transition from `hidden` to `visible`.
  3. Window refocus after gap.
  4. Periodic keepalive poll (every 60 seconds while visible).
- **Clients Stream Append-Only Events:** When a user interacts on Laptop A, Laptop A sends an append-only event (`POST /api/v1/aviary/events`). The server queues and commits it. When Phone B polls a minute later, it pulls the newly calculated canonical state.

### 6.2 Preventing Overwrites & Drift Inflation
- **No Absolute State from Client:** A client cannot send `{"boldness": 0.65}`. Any client attempting to send trait values receives a 400 Bad Request.
- **Bounded Presence Deduplication:** If a user accidentally leaves both Laptop A and Phone B open simultaneously, both may emit presence pings. The server simulation tick merges concurrent presence intervals using an interval union algorithm:
$$\text{PresenceCredit}(t_1, t_2) = \text{Union}\left(\bigcup_{s \in \text{Sessions}} [T_{s, start}, T_{s, end}]\right)$$
Concurrent open tabs can never double-count presence time or inflate drift speed.
- **Clock Skew Mitigation:** Events with client timestamps drifting more than $\pm 120\text{ seconds}$ from server UTC are normalized to `server_received_at`.

---

## 7. Frontend Rendering Pipeline

```
+-------------------------------------------------------------------------------+
|                       CANVAS2D / WEBGL RENDER GRAPH                           |
|                                                                               |
|  [Layer 0] Sky & Horizon (Gradient dynamically computed from solar time)      |
|  [Layer 1] Distant Silhouette Canopy (Subtle Parallax Shift ~0.02x)           |
|  [Layer 2] Back Perch Zone (Z=0.6 Scale, Dappled Lighting, Soft Contrast)     |
|  [Layer 3] Middle Perch Zone (Z=0.8 Scale, Main Habitat Branches)             |
|  [Layer 4] Birds Entity Renderer (Procedural Feather Shading, Skeletal IK)    |
|  [Layer 5] Front Perch Rail (Z=1.0 Scale, High Sharpness, Foreground Perch)   |
|  [Layer 6] Atmospheric Weather (Procedural Rain Streaks / Wind Distortion)    |
|  [Layer 7] Ambient Particles (Procedural Leaf & Feather Drifts)               |
+-------------------------------------------------------------------------------+
```

### 7.1 Single Horizontal Scene Composition
- The entire aviary is rendered to a single `<canvas>` element filling the viewport.
- **No Panning / No Scrolling:** The coordinate system maps to a virtual logical resolution of $1920 \times 1080$. On narrower viewports (mobile), the scene scales with letterboxing or horizontal branch compression, ensuring all perches and active birds remain 100% visible at all times.
- **Three Perch Zones:**
  - *Front Perch:* Nearest rail ($Y \approx 850$, scale factor $1.0$). Birds here convey intimacy and boldness.
  - *Middle Perch:* Central branches ($Y \approx 550$, scale factor $0.8$). Standard resting area.
  - *Back Perch:* High/distant boughs ($Y \approx 320$, scale factor $0.6$, slight atmospheric haze). Used by wary or drowsy birds.

### 7.2 Zero-Spinner Instant Start Architecture
To uphold the core principle that the aviary has been continuing without the user:
1. **Inlined Initial Snapshot:** On initial navigation, the SSR / edge server inlines the latest canonical snapshot directly into the HTML payload inside a `<script id="aviary-seed">` tag.
2. **First-Frame Mid-Action Paint:** Before executing network requests, the client instantiates birds directly into their snapshot perch positions, with procedural idle timers seeded so they are mid-breath, mid-preen, or mid-scan on frame 1.
3. **Quiet Field Cold Fallback:** In the event of a cache miss or cold connection, the engine renders a quiet sky gradient with a soft breeze particle. **No spinner, skeleton loader, or loading progress bar is ever shown.**

### 7.3 Procedural Idle Micro-Motion
Birds are driven by a procedural state machine running in the client animation loop ($60\text{ fps}$):
- **Saccadic Eye & Head Movement:** Birds execute rapid, discrete head turns every $2–6\text{ seconds}$ governed by personality curiosity and current mood.
- **Respiratory Sway:** Sine-wave feather fluff and chest expansion ($0.8\text{ Hz}$).
- **Preening Routine:** Triggered during `content` mood: bird turns beak toward wing feathers with micro-shiver oscillations.
- **Weight Shuffle:** Periodic small adjustment of claw grip on the branch.

### 7.4 Reduced-Motion Mode (`prefers-reduced-motion: reduce`)
Reduced motion is treated as an intentional, high-craft alternate aesthetic:
- Continuous skeletal animations are disabled.
- Movements are replaced with **soft, slow cross-fades (duration $1.2\text{s}$)** between key still poses (e.g. perching still $\to$ head turned $\to$ preening).
- Flight hops between perches do not traverse the screen; the bird fades out from Perch A and gently dissolves in on Perch B.
- Ambient leaf and feather drift particles are completely removed.
- Day/night lighting transitions occur over 10-second subtle cross-fades.

---

## 8. Audio Pipeline & Procedural Syrinx Synthesis

```
+------------------------------------------------------------------------------------+
|                         WEBAUDIO PROCEDURAL PIPELINE                               |
|                                                                                    |
|  [Motif Generator] -> [Dual FM Oscillators (Syrinx Model)]                         |
|                                |                                                   |
|                                v                                                   |
|                       [Formant Filter Bank]                                        |
|                                |                                                   |
|                                v                                                   |
|                  [Dynamic Gain Envelope (ADSR)]                                    |
|                                |                                                   |
|        +-----------------------+------------------------+                          |
|        | (Bird 1)                                       | (Bird 2)                 |
|        v                                                v                          |
|  [Bird PannerNode] (Stereo X)                     [Bird PannerNode] (Stereo X)     |
|        |                                                |                          |
|        v                                                v                          |
|  [Listen-In GainNode]                             [Listen-In GainNode]             |
|  (0dB to +6dB / -8dB)                             (0dB to +6dB / -8dB)             |
|        |                                                |                          |
|        +-----------------------+------------------------+                          |
|                                |                                                   |
|                                v                                                   |
|                       [Master Master Gain]                                         |
|                                |                                                   |
|                                v                                                   |
|                     [AudioContext.destination]                                     |
+------------------------------------------------------------------------------------+
```

### 8.1 Syrinx Physical Modeling via WebAudio
Rather than loading static MP3/WAV samples (which violate the bundle budget and destroy chorus dynamics through phase cancellation), bird calls are synthesized entirely via procedural audio:
- **Acoustic Model:** Two FM oscillator pairs simulating the dual tympaniform membranes of the avian syrinx.
- **Carrier & Modulator:** Fast frequency sweeps controlled by `ExponentialRampToValueAtTime`. Modulator frequency ($150–600\text{ Hz}$) introduces natural avian timbre and flutter.
- **Formant Filtering:** A series of `BiquadFilterNode` peaking filters reproducing the acoustic resonances of the bird's trachea and beak aperture.
- **Motif Grammars:** Each species defines a combinatorial grammar of 3–5 motifs (e.g. chirp, rising whistle, terminal trill, alarm chip). Motifs vary pitch ($\pm 5\%$), duration, and micro-vibrato on every individual vocalization.

### 8.2 Chorus Mixing & Natural Stagger
When multiple birds vocalize:
- **Avoidance of Synchronization:** A global acoustic scheduler enforces stochastic staggering. If Bird A calls, Bird B is delayed by a randomized offset ($t_{stagger} \in [800\text{ms}, 2400\text{ms}]$) unless engaging in a duet call.
- **Spatial Positioning:** Each bird's audio feeds into a `StereoPannerNode` mapped to its normalized horizontal perch coordinate ($X \in [-0.8, 0.8]$).

### 8.3 Listen-In Mix Dynamics
When a user clicks or focuses a specific bird:
1. **Target Bird:** Smoothly increases volume from $0\text{ dB}$ to $+6\text{ dB}$ using `linearRampToValueAtTime` over $1.5\text{ seconds}$.
2. **Ambient Birds:** Smoothly attenuate from $0\text{ dB}$ to $-8\text{ dB}$ over $1.5\text{ seconds}$.
3. **No Hard Cut / No Full Mute:** Background birds remain distinctly audible to preserve the feeling of an unbroken ecosystem.
4. **Disengage Curve:** On click-away or Escape, all gains interpolate back to $0\text{ dB}$ baseline over $2.0\text{ seconds}$.

### 8.4 WebAudio Fallback & Autoplay Policy
- **Autoplay Handling:** Browser policies block audio until first user gesture. The UI starts in graceful silence. On the first user click or keypress anywhere on the document, `AudioContext.resume()` is dispatched silently without showing a banner or modal.
- **Graceful Fallback:** If WebAudio is unsupported or fails to initialize, the application defaults to **silent mode with call captions automatically enabled**. In adherence to `accessibility_perf.md`, **no recorded audio fallback path will be built**.

---

## 9. Accessibility Surfaces & Inclusive Design

### 9.1 Screen-Reader Narration (`aria-live="polite"`)
Screen-reader users experience a fully realized naturalist accompaniment rather than mechanical state change announcements.
- **DOM Container:** An off-screen live region:
  ```html
  <div id="aviary-live-narration" aria-live="polite" aria-atomic="true" class="sr-only"></div>
  ```
- **Prose Generator:** Converts the visual scene into continuous, poetic naturalist observations:
  - *Idle State (every 30–45s):* `"a small grey warbler sits on the front rail, preening softly in the morning light. another bird watches quietly from the high branch."`
  - *Interaction Priority Bump:* On an offer being accepted: `"pip hops down toward the scattered seeds, tilting her head before taking one."`
  - *Settle Gesture:* `"the evening light softens across the perches as the aviary settles into quiet."`

### 9.2 Real-Time Call Captions
For hard-of-hearing users or users in quiet environments:
- Captions appear as floating, low-contrast translucent text capsules adjacent to the vocalizing bird.
- **Dynamic Prose:** Derived from procedural synthesis parameters:
  - Rising tone: `"a soft three-note rise"`
  - Trill: `"a low trill, paused, low trill again"`
  - Alarm chip: `"a single sharp call from the back perch"`
- Text fades in over $200\text{ms}$, persists for the duration of the call $+ 1.0\text{s}$, then gently dissolves.

### 9.3 Keyboard Navigation Model
Full functional parity is guaranteed without mouse interaction:
- `Tab`: Moves focus through top-bar actions (`Settings`, `Accessibility`, `Notebook`, `Offer`).
- `Tab` into Scene: Focuses the first bird on the front perch with an accessible high-contrast SVG focus ring.
- `Arrow Left` / `Arrow Right`: Cycles focus between birds ordered left-to-right across perches.
- `Enter` / `Space`: Activates `listen-in` on the focused bird.
- `Escape`: Disengages `listen-in` or closes open notebook/settings panels.
- `O`: Direct shortcut to expand the Offer menu; `1`, `2`, `3` selects Seed, Song Fragment, or Still Pool.
- `S`: Triggers Settle gesture (with 5-second undo prompt).

### 9.4 Contrast & Visual Hierarchy
All text elements (top bar labels, notebook typography, captions, settings dialogues) enforce a minimum contrast ratio of **4.5:1 (WCAG AA)** against both morning-light and midnight-dim scene backgrounds.

---

## 10. Performance Budgets, Verification & Observability

### 10.1 Quantitative Performance Budgets
| Metric | Budget Ceiling | Verification Gate |
| :--- | :--- | :--- |
| **Initial JS Bundle (Gzipped)** | $\le 2.0\text{ MB}$ | CI Webpack/Vite Bundle Analyzer build check |
| **Time to First Bird Visible (TTFBird)**| $< 500\text{ ms}$ (4G, Mid-Tier Mobile)| Synthetic Lighthouse / WebPageTest mobile run |
| **Runtime Framerate** | $60\text{ fps}$ sustained | Chrome tracing on simulated 5-year-old laptop CPU |
| **Memory Heap Growth (30 min)** | $\Delta \text{Heap} \le 0\text{ MB}$ (net zero leak) | Automated Puppeteer soak test running in CI |
| **Simulation Tick Latency** | $p99 < 5.0\text{ seconds}$ | Server Prometheus Alerting rule |

### 10.2 Architectural Strategies to Meet Budgets
1. **Bundle Size Enforcement:**
   - Zero heavyweight UI libraries (no React/Angular overhead). Clean vanilla TypeScript with lightweight reactive state stores.
   - Vector graphics compiled to compact mathematical coordinate paths; zero raster texture atlases.
   - Dynamic code-splitting for non-critical dialogs (`import(./notebook)`, `import(./settings)`, `import(./export)`).
2. **Time to First Bird (<500ms):**
   - Edge server returns pre-computed HTML shell containing inline initial state.
   - First frame is painted synchronously during script execution before external network handshakes occur.
3. **Zero Memory Leak Guarantee:**
   - Procedural WebAudio nodes use a recycled object pool. Disconnected `GainNodes` and `BiquadFilterNodes` are re-used rather than allocated per call.
   - Notebook DOM virtualizer recycles elements outside the viewport.

### 10.3 Observability Architecture & Privacy Guardrails
- **Client RUM:** Collects page load timing, `performance.measure("time-to-first-bird")`, WebAudio context failure rate, and FPS degradation events.
- **Server Metrics:** Simulation worker tick duration, PostgreSQL query latency, queue depth, error rate.
- **Privacy Enforcement:** Automated CI integration tests assert that no telemetry event payload contains `account_id`, `bird_id`, `name`, `personality_vectors`, or interaction history.

---

## 11. Rollout, Testing & Calibration Strategy

### 11.1 Phased Implementation Milestones
- **Sprint 1: Core Engine & Audio Syrinx Prototype**
  - WebAudio procedural synthesis engine and motif grammar generator.
  - Basic Canvas2D scene graph rendering 2 starter species on 3 perches.
  - Unit tests for syrinx audio synthesis without clicks or pops.
- **Sprint 2: Server Tick & Simulation Dynamics**
  - PostgreSQL schema migration and Redis queue setup.
  - Background simulation worker executing 60s ticks.
  - Implementation of monotonic low-pass drift filter and Markov mood machine.
- **Sprint 3: Interaction Surface & Presence Accounting**
  - Triple-condition presence monitor (`visibilityState` + `focus` + activity window).
  - Return-greeting engine and listen-in audio rebalance curves.
  - Offer cooldowns and Settle gesture with 5-second undo grace window.
- **Sprint 4: Accessibility, Polish & Notebook Generation**
  - Screen-reader naturalist live narration and procedural call captions.
  - Reduced-motion cross-fade rendering mode.
  - Naturalist notebook observation heuristic engine.
- **Sprint 5: Multi-Device Sync, Visits & Performance Hardening**
  - Read-only visit invitation link creation, validation, and revocation.
  - Multi-device snapshot reconciliation tests.
  - 30-minute memory leak soak testing and bundle optimization under 2MB.

### 11.2 Simulation & Drift Calibration Harness
A dedicated headless test harness (`scripts/calibrate_drift.ts`) simulates aviary progression over accelerated time horizons:
1. **1-Week Run (7 daily 15-min sessions):** Asserts all traits increase between $+0.02$ and $+0.04$. Asserts instruments detect change.
2. **3-Week Run (21 daily 15-min sessions):** Asserts traits increase by $+0.10$ to $+0.15$. Asserts birds migrate forward from back perch.
3. **6-Month Neglect Run (14 days active followed by 160 days offline):** Asserts traits remain identical to day 14 values (proving zero negative drift).
4. **Saturation Guard:** Asserts no trait exceeds $1.0000$ under any pathological click frequency.

---

## 12. Risk Management & Failure Modes

| Risk Description | Severity | Likelihood | Architectural Mitigation |
| :--- | :--- | :--- | :--- |
| **Drift Runaway / Tamagotchi Effect** (Users find way to click repeatedly to level up birds) | High | Medium | Strictly decouple clicks from drift. Presence-time is the dominant weight; offers enforce a 3-minute cooldown per bird. Low-pass filter asymptotes near 1.0. |
| **Multi-Device Drift Overwrite** (Session on phone clobbers drift from laptop) | Critical | Low | Ban client-submitted personality values. Clients only append factual event logs. Server tick is the sole writer. |
| **Procedural Audio Uncanniness / Fatigue** (Calls sound metallic, robotic, or repetitive) | High | Medium | Dual-FM modulation with continuous stochastic jitter on pitch, envelope duration, and vibrato speed. Chorus scheduler prevents unnatural unison calls. |
| **Browser Autoplay Blocks Audio** | Medium | High | Silent start by default. AudioContext resumes transparently on first user interaction. WebAudio failure triggers instant caption fallback. |
| **Accidental Streak Gamification** (Contributor attempts to add visit counters) | Critical | Low | Architectural rule enshrined in PRD and linted in PR review: zero engagement counters, zero streaks, zero dot calendars. |
| **Screen-Reader Queue Overload** | Medium | Medium | Narration rate strictly throttled to 1 update per 30–45 seconds during idle, using `aria-live="polite"` so it never interrupts the user. |
| **PII Leakage into Telemetry** | High | Low | Enforce synthetic UUIDs globally. Automated CI test fails build if `email` appears in telemetry payload keys or values. |

---

## 13. Conclusion
This implementation plan translates every affective requirement and technical constraint of the Pocket Aviary product specification into an actionable, resilient, and high-performance architecture. By maintaining strict discipline around non-goals, treating accessibility as a primary aesthetic surface, and grounding the simulation in a server-canonical, monotonic drift engine, the engineering team can deliver an experience that feels truly alive, quiet, and lasting.
