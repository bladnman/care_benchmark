# Pocket Aviary — System Architecture & Implementation Plan (v1)

---

## 1. Executive Summary & Scope

### 1.1 Product Purpose & Vision
Pocket Aviary is an ambient, browser-based virtual aviary designed around low-key, observational relationships. Users adopt two starter birds (scaling up to seven over months based on aviary age). The aviary lives in a single horizontal viewport rendered in the browser. Birds exhibit personality drift over weeks in response to genuine user attention (measured through a strict definition of *presence* and discrete interactions).

The core technical mandate is to deliver an environment that **feels alive, not robotic**, adhering to the foundational design tenet: **Notice, never announce**. The system has no gamification, no win conditions, no counters, and no custodial burden.

---

### 1.2 In-Scope for v1
- **Single-User Accounts & Auth:** Passwordless authentication via 15-minute email magic links; revocable per-device session tokens; account export (JSON) and soft deletion (30-day grace period followed by hard purge).
- **Aviary Scene & Visuals:** Single horizontal responsive scene fitting any viewport without panning/scrolling; 3 perch zones (front, middle, back); day/night cycle matching user's local solar/clock time; subtle ambient weather (rain/wind); continuous ambient micro-motion (leaves, feathers).
- **Bird Engine:** 
  - Species pool of 6 initial species with distinct visual silhouettes, palettes, and call grammars.
  - Two starter birds assigned at adoption; birds 3–7 unlocked strictly by aviary calendar age.
  - Hidden 5-dimensional scalar personality vector (`boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`).
  - Monotonic upward drift function driven by low-pass filtered presence and attention signals (zero decay or negative drift on neglect).
  - Fast-timescale mood state machine (`wary`, `content`, `curious`, `drowsy`, `alert`, `settled`).
  - Personality and mood-shaped idle micro-motion (preening, scanning, head-tilting, fluffed feathers).
- **Core Interactions:**
  - Procedural return-greeting upon tab open/return, staggered and varied by absence length, boldness, and mood.
  - *Listen-in* focus mode (smooth dynamic audio rebalance favoring focused bird).
  - *Offer* interaction (seed, song fragment, still pool) with per-bird cooldowns.
  - *Settle* session-end gesture (soft evening light shift, audio quieting, 5-second undo grace).
- **Field Notebook:** Auto-generated naturalist observation log; sparse cadence (~1 entry every few days); read-only, persistent, un-curated.
- **Server-Side Simulation & Sync:** Canonical simulation ticking at ~1-minute cadence independent of client connection; append-only interaction event stream; snapshot polling + visibility wakeups; zero client-side personality authoring.
- **Social (Optional & Quiet):** One-time, email-based visit invitations (read-only ambient viewport, non-co-present, zero visitor drift contribution, host revocable).
- **Accessibility:** 
  - Naturalist prose screen-reader live narration (30–60s idle cadence, priority event bumps).
  - Procedurally generated call captions positioned near vocalizing birds.
  - Dedicated reduced-motion mode (cross-fading still poses, disabled particle drift).
  - Full keyboard navigation and visible WCAG AA focus indicators.
- **Audio Engine:** 100% client-side procedural synthesis via WebAudio API; procedural chorus coordination; graceful silence + automated caption fallback when WebAudio is unavailable.

---

### 1.3 Out-of-Scope (Explicit Non-Goals)
- **No Native Applications:** Web-only (desktop/mobile modern browsers). No iOS/Android native apps or wrappers.
- **No Gamification:** No streaks, visit counters, green-dot activity calendars, XP, badges, levels, adoption counts, or achievement toasts.
- **No Tamagotchi Mechanics:** Birds never die, starve, fall ill, or show distress. No hunger or happiness meters. Neglect produces ambient quietness, never punitive degradation or mistrust.
- **No Social Network Surfaces:** No public discovery directory, aviary search, user profiles, follower models, visitor comments/guestbooks, chat overlays, avatars, or leaderboards.
- **No Push / Out-of-Band Engagement:** No push notifications, no reminder emails, no "your friend visited" toasts or unprompted emails.
- **No Scene Customization:** No drag-and-drop bird placement, furniture/perch rearrangement, or theme editors.
- **No Real-Time Co-Presence:** Visitors observe a snapshot of the host's aviary without shared cursor, visitor indicators, or interaction rights.

---

### 1.4 Voice & Tone Boundary
- **Naturalist Register (Product Surface):** Used across aviary narration, bird call captions, field notebook entries, and offer responses.
  - *Characteristics:* Lowercase by default, present-tense, specific, bird-centric verbs (`notice`, `perch`, `settle`, `preen`, `call softly`). Never uses exclamation points or gamified terminology.
- **Matter-of-Fact Register (System Surface):** Used strictly for system mechanisms: magic-link sign-in, session timeouts, sync/loading failures, account export/deletion dialogs, and accessibility configuration.
  - *Characteristics:* Standard sentence casing, direct, clear, devoid of faux-naturalist whimsy or corporate cheerfulness (e.g., *"We couldn't sign you in. The link may have expired. Try requesting a new link."*).

---

## 2. System Architecture & Boundaries

```
                              +-------------------------------------------------------+
                              |                  Client Browser                       |
                              |  +--------------------+     +-----------------------+ |
                              |  | Canvas / DOM Render|     | WebAudio Engine       | |
                              |  | - Micro-motion     |     | - Procedural synth    | |
                              |  | - Parallax / Leaves|     | - Dynamic chorus mix  | |
                              |  | - Reduced-motion   |     | - Listen-in curve     | |
                              |  +--------------------+     +-----------------------+ |
                              |  +--------------------+     +-----------------------+ |
                              |  | Accessibility      |     | Interaction Controller| |
                              |  | - Live ARIA prose  |     | - Presence detector   | |
                              |  | - Dynamic captions |     | - Event batcher       | |
                              |  +--------------------+     +-----------------------+ |
                              +---------------------------^---------------------------+
                                                          | HTTPS / WSS
                                                          v
+----------------------------------------------------------------------------------------------------+
|                                    Edge Gateway / CDN                                              |
| - SSL termination, rate limiting, static asset delivery (<2MB bundle)                              |
| - Edge injection of initial state snapshot into index HTML (sub-500ms time-to-first-bird)          |
+-------------------------------------------------+--------------------------------------------------+
                                                  |
                                                  v
+----------------------------------------------------------------------------------------------------+
|                                 Aviary Application Core (Node.js / Go)                              |
|                                                                                                    |
|  +-----------------------+    +-----------------------+    +------------------------------------+  |
|  | Auth & Session Mgr    |    | Interaction Ingestion |    | State Snapshot API                 |  |
|  | - Magic link token gen|    | - Append-only log     |    | - Read-only canonical view         |  |
|  | - Device session auth |    | - Presence validation |    | - Visitor token validation         |  |
|  +-----------------------+    +-----------------------+    +------------------------------------+  |
+-------------------------------------------------+--------------------------------------------------+
                                                  |
                                                  v
+----------------------------------------------------------------------------------------------------+
|                              Simulation Tick Worker Fleet (Scheduled ~1m)                          |
|                                                                                                    |
|  +----------------------------------------------------------------------------------------------+  |
|  | Partitioned Worker Process (by Synthetic Account UUID)                                       |  |
|  | 1. Ingest unprocessed interaction events from append-only log                                |  |
|  | 2. Compute presence accumulation & low-pass personality drift deltas                         |  |
|  | 3. Step mood state machines (solar time, weather, event stimuli, social contagion)           |  |
|  | 4. Generate sparse naturalist notebook observations (stochastic rule evaluation)             |  |
|  | 5. Persist atomic canonical snapshot to PostgreSQL                                           |  |
|  +----------------------------------------------------------------------------------------------+  |
+-------------------------------------------------+--------------------------------------------------+
                                                  |
                                                  v
+----------------------------------------------------------------------------------------------------+
|                                     Persistence & Isolation                                         |
|                                                                                                    |
|  +-------------------------------------+           +---------------------------------------------+ |
|  | PostgreSQL Canonical Database       |           | Privacy & Telemetry Firewall                | |
|  | - `accounts` (encrypted PII)        |           | - Aggregate-only metrics pipeline           | |
|  | - `aviaries`, `birds`, `drift_state`|           | - STRICTLY FORBIDDEN: Per-bird data,        | |
|  | - `interaction_events` (append log) |           |   per-account interaction histories, or     | |
|  | - `field_notebook_entries`          |           |   individual behavioral vectors in analytics| |
|  | - `visit_invitations`               |           |                                             | |
|  +-------------------------------------+           +---------------------------------------------+ |
+----------------------------------------------------------------------------------------------------+
```

### 2.1 Component Responsibilities
1. **Client Browser Application:** 
   - Renders the aviary scene at 60fps using Canvas/WebGL with SVG/DOM hybrid overlay for UI chrome.
   - Executes procedural audio synthesis via WebAudio.
   - Monitors user input to calculate the 3-factor presence condition.
   - Buffers and streams discrete interaction events to the server.
   - Interpolates between canonical snapshots delivered by the server.
2. **Edge Gateway & API Layer:**
   - Handles magic-link verification and session token authentication.
   - Injects the latest aviary snapshot directly into the root HTML response to ensure the first bird renders within <500ms.
   - Exposes REST endpoints for interaction events, snapshot polling, visits, and account management.
3. **Simulation Tick Worker Fleet:**
   - Background worker service executing a distributed tick loop every 60 seconds.
   - Consistently shards processing across accounts using synthetic UUID hashing.
   - Applies the mathematical drift filter, updates moods, steps weather cycles, and evaluates field notebook generation rules.
4. **Data Isolation & Privacy Firewall:**
   - The canonical simulation database contains all account state keyed exclusively by synthetic UUID.
   - Observability tools receive only coarse aggregate metrics (latency, HTTP error rates, frame render times). No per-bird, per-account, or interaction telemetry is ever routed to data warehouses or analytics pipelines.

---

## 3. Data Model & Database Schemas

### 3.1 Relational Schema (PostgreSQL DDL)

```sql
-- Core Accounts Table
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(), -- Synthetic Account UUID (Internal reference)
    email_encrypted BYTEA NOT NULL,                 -- Encrypted PII (AES-GCM-256)
    email_hash VARCHAR(64) NOT NULL UNIQUE,         -- HMAC-SHA256 for lookup without decryption
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    settled_at TIMESTAMPTZ,                         -- Timestamp if currently in settled state
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',    -- IANA Timezone for local day/night calculation
    deletion_requested_at TIMESTAMPTZ,              -- Soft-deletion timestamp (30-day purge grace)
    visit_notifications_enabled BOOLEAN NOT NULL DEFAULT FALSE -- Opt-in host notification toggle
);

CREATE INDEX idx_accounts_email_hash ON accounts(email_hash);
CREATE INDEX idx_accounts_deletion ON accounts(deletion_requested_at) WHERE deletion_requested_at IS NOT NULL;

-- Active Auth Sessions
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    session_token_hash VARCHAR(64) NOT NULL UNIQUE,
    user_agent TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_seen_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at TIMESTAMPTZ
);

CREATE INDEX idx_sessions_account ON sessions(account_id) WHERE revoked_at IS NULL;

-- Aviary Canonical State
CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    version BIGINT NOT NULL DEFAULT 1,              -- Monotonic state version
    current_weather VARCHAR(32) NOT NULL DEFAULT 'clear', -- 'clear', 'soft_rain', 'gentle_wind'
    weather_expires_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Birds Table
CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(), -- Stable Bird Identifier
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL,              -- e.g., 'grey_warbler', 'masked_finch', 'nightjar'
    name VARCHAR(48) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    perch_zone VARCHAR(16) NOT NULL DEFAULT 'middle', -- 'front', 'middle', 'back'
    perch_slot INT NOT NULL DEFAULT 0,
    
    -- Current Mood State (Fast-timescale)
    current_mood VARCHAR(16) NOT NULL DEFAULT 'content', -- 'wary', 'content', 'curious', 'drowsy', 'alert', 'settled'
    mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    -- Personality Vector (Slow-timescale, normalized 0.000 to 1.000)
    trait_boldness NUMERIC(5, 4) NOT NULL DEFAULT 0.2000,
    trait_social_warmth NUMERIC(5, 4) NOT NULL DEFAULT 0.2000,
    trait_vocal_frequency NUMERIC(5, 4) NOT NULL DEFAULT 0.2000,
    trait_plumage_saturation NUMERIC(5, 4) NOT NULL DEFAULT 0.2000,
    trait_curiosity NUMERIC(5, 4) NOT NULL DEFAULT 0.2000,
    
    -- Drift Accumulators (Raw presence-seconds & stimulus counts since last drift step)
    accum_presence_seconds INT NOT NULL DEFAULT 0,
    accum_listen_in_seconds INT NOT NULL DEFAULT 0,
    accum_offer_interactions INT NOT NULL DEFAULT 0,
    
    last_offer_at TIMESTAMPTZ
);

CREATE INDEX idx_birds_aviary ON birds(aviary_id);

-- Append-Only Interaction Events Log
CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    bird_id UUID REFERENCES birds(id) ON DELETE SET NULL,
    event_type VARCHAR(32) NOT NULL, -- 'presence_heartbeat', 'listen_in_start', 'listen_in_end', 'offer', 'settle'
    payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    client_timestamp TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_by_tick BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE INDEX idx_interaction_unprocessed ON interaction_events(account_id, id) WHERE processed_by_tick = FALSE;

-- Field Notebook Entries
CREATE TABLE field_notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    observed_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    body_prose TEXT NOT NULL,                        -- Naturalist prose, lowercase
    noteworthy_event_type VARCHAR(32)               -- 'first_greeter', 'long_quiet', 'fluffed_cool_air'
);

CREATE INDEX idx_notebook_aviary ON field_notebook_entries(aviary_id, observed_at DESC);

-- Visit Invitations & Access Logs
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    invite_token_hash VARCHAR(64) NOT NULL UNIQUE,
    visitor_email_encrypted BYTEA NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '30 days'),
    revoked_at TIMESTAMPTZ
);

CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invitation_id UUID NOT NULL REFERENCES visit_invitations(id) ON DELETE CASCADE,
    visited_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    duration_seconds INT NOT NULL DEFAULT 0
);
```

---

## 4. API Surface & Protocols

All error responses and system confirmations follow the **Matter-of-Fact** voice guidelines.

### 4.1 Authentication & Account Management

#### `POST /api/v1/auth/magic-link`
Initiates passwordless sign-in. Rate-limited to 5 requests per 15 minutes per IP/email.
- **Request:** `{"email": "user@example.com"}`
- **Response (200 OK):**
  ```json
  {
    "status": "ok",
    "message": "We sent a sign-in link to your email. It will expire in 15 minutes."
  }
  ```

#### `GET /api/v1/auth/verify?token=<token>`
Consumes the single-use token and sets an HTTP-only secure cookie containing the session token.
- **Response (302 Redirect):** Redirects to root `/` on success.
- **Error Response (401 Unauthorized):**
  ```json
  {
    "error": "invalid_link",
    "message": "We couldn't sign you in. The link may have expired. Try requesting a new link."
  }
  ```

#### `GET /api/v1/account/export`
Generates a JSON snapshot of the user's complete aviary state and emails a secure download link.
- **Response (200 OK):**
  ```json
  {
    "status": "ok",
    "message": "Your aviary export is being prepared and will be emailed to your verified address."
  }
  ```

#### `POST /api/v1/account/delete`
Flags account for soft-deletion (30-day recovery window).
- **Response (200 OK):**
  ```json
  {
    "status": "ok",
    "message": "Your account has been scheduled for deletion in 30 days. You can sign in anytime before then to cancel deletion."
  }
  ```

---

### 4.2 State Ingestion & Interaction Streams

#### `GET /api/v1/aviary/snapshot`
Retrieves current canonical aviary state. Used on initial page boot, tab visibility changes, and background polling.
- **Response (200 OK):**
  ```json
  {
    "aviary_id": "8f3b1406-8d63-4c91-a1b4-e49c71c4fa21",
    "version": 1042,
    "server_time": "2026-08-13T18:30:00Z",
    "local_time_offset_seconds": -25200,
    "weather": "clear",
    "settled": false,
    "birds": [
      {
        "id": "c1a3b849-2e6f-42e1-a083-d922a7f5c711",
        "name": "pip",
        "species_id": "grey_warbler",
        "perch_zone": "front",
        "perch_slot": 1,
        "mood": "content",
        "plumage_saturation": 0.342,
        "call_frequency_factor": 0.450
      },
      {
        "id": "7d9e2114-1f8a-49c8-bf40-5e839e248b90",
        "name": "wren",
        "species_id": "masked_finch",
        "perch_zone": "middle",
        "perch_slot": 2,
        "mood": "drowsy",
        "plumage_saturation": 0.280,
        "call_frequency_factor": 0.210
      }
    ]
  }
  ```
  *(Note: Numerical trait vectors like boldness and curiosity are strictly excluded from client snapshots; only direct visual/audio rendering scalars are exposed).*

#### `POST /api/v1/aviary/events`
Batch endpoint for client interaction events.
- **Request:**
  ```json
  {
    "events": [
      {
        "event_type": "presence_heartbeat",
        "client_timestamp": "2026-08-13T18:31:00Z",
        "payload": { "duration_seconds": 60, "visibility": "visible", "focused": true, "active_input": true }
      },
      {
        "event_type": "listen_in_start",
        "bird_id": "c1a3b849-2e6f-42e1-a083-d922a7f5c711",
        "client_timestamp": "2026-08-13T18:31:15Z"
      },
      {
        "event_type": "offer",
        "bird_id": "c1a3b849-2e6f-42e1-a083-d922a7f5c711",
        "payload": { "offer_type": "seed" },
        "client_timestamp": "2026-08-13T18:31:40Z"
      }
    ]
  }
  ```
- **Response (202 Accepted):** `{"status": "accepted", "queued_count": 3}`

---

### 4.3 Field Notebook API

#### `GET /api/v1/notebook`
Returns historical naturalist observations. Supports cursor pagination.
- **Response (200 OK):**
  ```json
  {
    "entries": [
      {
        "id": "e9384b01-3482-41e9-9182-3d719a842f10",
        "observed_at": "2026-08-12T07:14:00Z",
        "body_prose": "tuesday — pip greeted before wren today, first time this week."
      },
      {
        "id": "2840b129-8734-4530-9b62-11c78490aef4",
        "observed_at": "2026-08-09T16:42:00Z",
        "body_prose": "wren is fluffed against the cool air, watching the back perch. low calls only."
      }
    ],
    "next_cursor": null
  }
  ```

---

### 4.4 Visit Invitation Flow

#### `POST /api/v1/visits/invite`
Host generates an invitation link for a friend.
- **Request:** `{"visitor_email": "friend@example.com"}`
- **Response (200 OK):**
  ```json
  {
    "status": "ok",
    "invite_id": "a4b12345-9876-4321-abcd-ef0123456789",
    "message": "An invitation link has been sent to friend@example.com."
  }
  ```

#### `GET /api/v1/visits/view?token=<invite_token>`
Visitor endpoint to fetch read-only snapshot.
- Returns standard aviary state marked with `"read_only": true`.
- **Error Response (403 Forbidden / 410 Gone):**
  ```json
  {
    "error": "visit_unavailable",
    "message": "This visit is no longer available. The invitation may have expired or been revoked by the host."
  }
  ```

---

## 5. Simulation Engine Design

The simulation tick is the canonical heart of the product. It runs server-side every 60 seconds per active aviary, processing accumulated events and progressing time-based dynamics.

```
+----------------------------------------------------------------------------------------------------+
|                                    60-Second Simulation Tick Cycle                                 |
+----------------------------------------------------------------------------------------------------+
| 1. EVENT INGESTION & VALIDATION                                                                    |
|    - Fetch unprocessed events for account; validate 3-way presence conjunction                     |
|    - Discard unverified presence intervals                                                         |
+----------------------------------------------------------------------------------------------------+
                                                  |
                                                  v
+----------------------------------------------------------------------------------------------------+
| 2. SLOW-TIMESCALE PERSONALITY DRIFT (Low-Pass Filter)                                              |
|    - Compute raw stimulus vector: S = [Presence, Listen-in, Offers, Settle]                        |
|    - For each trait t: Δt = α_t * (1.0 - trait_val) * S_weight                                     |
|    - Apply monotonic clamp: trait_val_new = MAX(trait_val, trait_val + Δt)                         |
+----------------------------------------------------------------------------------------------------+
                                                  |
                                                  v
+----------------------------------------------------------------------------------------------------+
| 3. FAST-TIMESCALE MOOD STATE TRANSITIONS                                                           |
|    - Compute Solar Time (local dawn/dusk) -> Base bias (drowsy, alert, content)                   |
|    - Ambient weather step (Markov transition: clear -> soft_rain -> clear)                         |
|    - Evaluate recent stimulus (offers nudge to content/curious, alarm contagion)                   |
|    - Soft decay towards trait-attractor equilibrium                                                |
+----------------------------------------------------------------------------------------------------+
                                                  |
                                                  v
+----------------------------------------------------------------------------------------------------+
| 4. FIELD NOTEBOOK OBSERVATION EVALUATION (Sparse Trigger)                                           |
|    - Evaluate cooldown timer (minimum 48 hours since last observation)                             |
|    - Check noteworthy predicate triggers (e.g. first greeter inversion, 30m sustained quiet)       |
|    - If trigger matches and random roll passes -> Emit lowercase naturalist entry                  |
+----------------------------------------------------------------------------------------------------+
                                                  |
                                                  v
+----------------------------------------------------------------------------------------------------+
| 5. ATOMIC SNAPSHOT PERSISTENCE & VERSION BUMP                                                      |
|    - Write updated traits, moods, weather to PostgreSQL                                            |
|    - Mark interaction events as processed                                                          |
+----------------------------------------------------------------------------------------------------+
```

### 5.1 Mathematical Drift Model

#### Calibration Targets
- **1 Week Regular Use (~30 min/day):** Measurable instrument shift ($\Delta \approx +0.02 - 0.04$ on a $0.0 - 1.0$ scale).
- **3 Weeks Regular Use:** Noticeable behavioral shift visible to the user (e.g., perch zone migrations, greeting priority shifts, richer plumage saturation).

#### Drift Formulation
Let $P$ be valid presence seconds in the tick, $L_i$ be listen-in seconds for bird $i$, and $O_i$ be offer events.
For each trait $T \in \{\text{boldness}, \text{social\_warmth}, \text{vocal\_frequency}, \text{plumage\_saturation}, \text{curiosity}\}$:

$$\Delta T_i = \alpha_T \cdot (1.0 - T_i) \cdot \left[ w_{p} \left(\frac{P}{3600}\right) + w_{l} \left(\frac{L_i}{3600}\right) + w_{o} O_i \right]$$

- **Coefficients:**
  - Presence weight $w_p = 1.0$
  - Listen-in weight $w_l = 2.5$ (amplifies focused bird's `social_warmth` and `vocal_frequency`)
  - Offer weight $w_o = 0.05$ (amplifies `curiosity` and `boldness`)
  - Scaling factor $\alpha_T \approx 0.0015$ (tuned so 21 days $\times$ 30 mins yields $\Delta \approx +0.15 - 0.20$).
- **Monotonicity Enforcement:**
  $$T_i(t+1) = \max\left(T_i(t), \, T_i(t) + \Delta T_i\right)$$
  Traits **never decay** upon user absence or neglect. Absence simply yields $\Delta T_i = 0$.

---

### 5.2 Mood State Machine & Ambient Influences

```
                  +----------------------------------------------+
                  |                 [ Content ]                  |
                  |           (Default baseline state)           |
                  +--------+----------------------------+--------+
                           |                            |
       Offer Accepted /    |                            | Threat / Startle /
       Motif Echo          |                            | Nearby Alarm Call
                           v                            v
                  +-----------------+          +-----------------+
                  |   [ Curious ]   |          |    [ Wary ]     |
                  +--------+--------+          +--------+--------+
                           |                            |
          Quiet / Post-Feed|                            | Time Elapsed /
                           |                            | Soft Vocal Call
                           v                            v
                  +-----------------+          +-----------------+
                  |    [ Alert ]    |<---------+   [ Drowsy ]    |
                  |  (Early Morning)|  Dawn    | (Dusk / Settled)|
                  +-----------------+          +-----------------+
```

- **Solar Time Calculation:** Solar zenith angle calculated from client timezone and UTC timestamp.
  - *Dawn/Morning (06:00–10:00):* Nudges birds toward `alert` or `content`; increases baseline call probability.
  - *Midday (10:00–17:00):* Equilibrium state (`content`).
  - *Dusk/Evening (17:00–21:00):* Transition to `drowsy`; calls soften.
  - *Night (21:00–06:00):* `settled` state (eyes shut, fluffed feathers) for all diurnal species; nocturnal species (e.g. *Nightjar*) shift to `alert`/active.
- **Weather Generator:** Low-probability Markov process. Rain occurs ~2–3 times/week for 15–30 minutes, temporarily reducing vocal frequency and prompting birds to seek sheltered perch slots.
- **Bird-to-Bird Social Dynamics:** If Bird A enters `wary`, neighboring birds within 1 perch distance have a 40% probability of transitioning to `wary` on the subsequent tick.

---

### 5.3 Sparse Field Notebook Generator

The notebook generation engine runs during the simulation tick.
- **Rate Limit & Sparsity:** Hard minimum of 48 hours between generated entries unless a major milestone occurs (e.g., adoption of bird 3).
- **Rule Evaluators:**
  1. *Greeter Priority Inversion:* Evaluates if Bird B greeted before Bird A after a sustained streak of Bird A greeting first.
     - *Template:* `"{day_of_week} — {bird_b} greeted before {bird_a} today, first time this week."`
  2. *Weather Observation:* Passing rain during morning hours with specific bird perching posture.
     - *Template:* `"{bird_name} is fluffed against the cool air, watching the back perch. low calls only."`
  3. *Extended Idle Watch:* User completed a 20+ minute presence session with zero offers.
     - *Template:* `"a long stretch of quiet this morning. {bird_name} preened for several minutes without looking up."`
- **Output Validation:** All candidate entries are forced to lowercase, present-tense, and verified against an anti-gamification keyword filter (rejects numbers, stats, streaks, or exclamation marks).

---

## 6. State Synchronization & Concurrency Model

### 6.1 Server-Authoritative Architecture
To eliminate data corruption and last-write-wins races across multiple client devices (e.g., laptop open in morning, phone open at lunch):
1. **Clients NEVER write state:** Clients never send trait updates, mood assignments, or coordinate vectors.
2. **Append-Only Event Stream:** Clients push timestamped interaction records to `/api/v1/aviary/events`.
3. **Deterministic Sequential Consumption:** The server-side simulation tick consumes events in strict chronological order and applies additive deltas.
4. **Client Interpolation:** Clients poll `/api/v1/aviary/snapshot` every 30 seconds (or immediately upon window focus / `visibilitychange`) and interpolate visual positions and transitions smoothly over 1.5 seconds.

---

### 6.2 Presence Detection & Concurrency Rules

```
                      +---------------------------------------------------------+
                      |                 Client Presence Detector                |
                      +---------------------------------------------------------+
                      | Condition A: document.visibilityState === 'visible'     |
                      | Condition B: document.hasFocus() === true               |
                      | Condition C: User activity within last 300s (5 min)     |
                      |              (pointermove, keydown, touchstart)         |
                      +----------------------------+----------------------------+
                                                   |
                             All 3 TRUE?           |
                                                   v
                                        +--------------------+
                                        | Record Presence    |
                                        | (Heartbeat active) |
                                        +----------+---------+
                                                   |
                             Any condition FALSE?  |
                                                   v
                                        +--------------------+
                                        | Stop Presence      |
                                        | (Heartbeat paused) |
                                        +--------------------+
```

- **Dual-Device Coexistence:** If a user has both a laptop tab and mobile tab open simultaneously, presence seconds are unioned by UTC timestamp intervals during server-side tick ingestion. Presence time is capped at exactly 60 seconds per wall-clock minute, preventing drift duplication.

---

## 7. Frontend Rendering Pipeline

```
+----------------------------------------------------------------------------------------------------+
|                                    Viewport & Scene Composition                                    |
|                                                                                                    |
|  +----------------------------------------------------------------------------------------------+  |
|  | Layer 0: Sky & Atmospheric Gradient (CSS / WebGL, Solar Time modulated)                      |  |
|  +----------------------------------------------------------------------------------------------+  |
|  | Layer 1: Background Elements (Soft-focus foliage, subtle 0.2x parallax)                      |  |
|  +----------------------------------------------------------------------------------------------+  |
|  | Layer 2: Back Perch Zone (Far branches; high-wary / distant birds)                           |  |
|  +----------------------------------------------------------------------------------------------+  |
|  | Layer 3: Middle Perch Zone (Main horizontal perches; equilibrium birds)                      |  |
|  +----------------------------------------------------------------------------------------------+  |
|  | Layer 4: Front Perch Zone (Foreground rail/basin; bold birds, offer interactions)            |  |
|  +----------------------------------------------------------------------------------------------+  |
|  | Layer 5: Ambient Particle System (Leaves, drifting feathers; disabled in reduced-motion)   |  |
|  +----------------------------------------------------------------------------------------------+  |
|  | Layer 6: UI Chrome & Top-Bar (Sparse SVG icons; auto-fades to opacity: 0 on cursor idle)     |  |
|  +----------------------------------------------------------------------------------------------+  |
+----------------------------------------------------------------------------------------------------+
```

### 7.1 Scene Mechanics & Responsive Scaling
- **Fixed Aspect Ratio Bounded Box:** The aviary scene renders inside a 16:9 to 21:9 responsive container. On ultra-wide screens, lateral background foliage extends; on narrow mobile screens (down to 320px width), perch coordinates scale proportionally using CSS `contain: layout size` and Canvas coordinate transforms. Birds are never cropped or pushed offscreen.
- **Zero-Entry-Pop Loading Architecture:**
  - Initial HTML includes an inline state snapshot payload embedded at the edge server.
  - The rendering engine immediately draws the first frame with birds positioned mid-action (e.g. bird 1 preening on front perch, bird 2 scanning middle perch).
  - No loading spinner, fade-from-black, or splash sequence is used. If network latency delays initial data, a soft sky-colored quiet field renders with subtle ambient leaf drift.

---

### 7.2 Idle Micro-Motion Engine
Idle motion is procedurally generated using hierarchical sine/cosine perturbation and Perlin noise rather than rigid loop cycles:
- **Preening:** Occurs primarily during `content` mood. Micro-rotations of the head ($15^\circ - 30^\circ$) coupled with plumage fluffing.
- **Scanning:** Occurs during `alert` or `wary` moods. Quick horizontal saccades ($40^\circ$ turns with 100ms dwell time).
- **Head-Tilts:** Triggered by ambient sounds, another bird's call, or cursor proximity.
- **Weight Shuffle:** Subtle vertical bobbing ($1 - 2\text{px}$) and claw repositioning every 8–15 seconds.

---

### 7.3 Reduced-Motion Implementation
When `prefers-reduced-motion: reduce` is detected or toggled in accessibility settings:
- Frame-by-frame continuous skeletal animation is bypassed.
- Birds transition between distinct behavioral poses using a 1.2-second CSS cross-fade opacity dissolve.
- Perch zone relocation (flight) is replaced by a gentle 1.5-second cross-fade between origin and destination perches.
- Ambient drifting leaves and floating feather particles are disabled.
- Atmospheric day/night lighting shifts are extended to smooth, slow 10-second cross-fades.

---

### 7.4 Top-Bar Chrome Auto-Fade
- **Idle Timer:** After 4.0 seconds of cursor inactivity or lack of keyboard input, the top navigation bar smoothly transitions to `opacity: 0.05` via `transition: opacity 1.2s ease-out`.
- **Instant Wake:** Any pointer movement, touch start, or keyboard navigation instantly restores `opacity: 1.0` within 100ms.

---

## 8. Audio Engine & Procedural Synthesis

Audio is synthesized 100% procedurally client-side via the WebAudio API. No static audio samples or looped sound files are loaded over the network.

```
                                  +---------------------------+
                                  | Procedural Call Generator |
                                  | - Motif grammar           |
                                  | - Pitch / timing variation|
                                  +-------------+-------------+
                                                |
                                                v
               +-----------------------------------------------------------------+
               |                    WebAudio Voice Graph                         |
               |                                                                 |
               |  +------------------------+      +---------------------------+  |
               |  | Dual Carrier Osc       |      | Modulator Osc (FM/AM)     |  |
               |  | (Custom PeriodicWave)  |      | (Vibrato / Trill)         |  |
               |  +-----------+------------+      +-------------+-------------+  |
               |              |                                 |                |
               |              +----------------+----------------+                |
               |                               |                                 |
               |                               v                                 |
               |                  +--------------------------+                   |
               |                  | Dynamic Biquad Filter    |                   |
               |                  | (Formant Shaping)        |                   |
               |                  +------------+-------------+                   |
               |                               |                                 |
               |                               v                                 |
               |                  +--------------------------+                   |
               |                  | GainEnvelope (ADSR)      |                   |
               |                  +------------+-------------+                   |
               +-------------------------------|---------------------------------+
                                               |
                                               v
                                  +---------------------------+
                                  | Per-Bird Stereo Panner &  |
                                  | Spatial Gain Node         |
                                  +-------------+-------------+
                                                |
                                                v
               +-----------------------------------------------------------------+
               |                    Master Aviary Chorus Mixer                   |
               |                                                                 |
               |  +-----------------------------------------------------------+  |
               |  | Listen-In Dynamic Gain Matrix                             |  |
               |  | - Focused Bird: Ramp to 1.0 (over 800ms)                  |  |
               |  | - Ambient Birds: Duck to 0.25 (never complete mute)       |  |
               |  +-----------------------------+-----------------------------+  |
               |                                |                                |
               |                                v                                |
               |  +-----------------------------------------------------------+  |
               |  | Soft Convolver Reverb (Ambient Room Response)             |  |
               |  +-----------------------------+-----------------------------+  |
               |                                |                                |
               |                                v                                |
               |  +-----------------------------------------------------------+  |
               |  | Master Dynamics Compressor / Limiter                      |  |
               |  +-----------------------------------------------------------+  |
               +-------------------------------|---------------------------------+
                                               |
                                               v
                                  +---------------------------+
                                  | AudioDestinationNode      |
                                  | (Hardware Speakers)       |
                                  +---------------------------+
```

### 8.1 Species Call Grammars & Synthesizer Architecture
Each of the 6 species possesses a distinct procedural motif grammar:
- **Grey Warbler:** Ascending 3-note frequency sweeps ($2.4\text{kHz} \rightarrow 3.8\text{kHz}$) with subtle $18\text{Hz}$ vibrato.
- **Masked Finch:** Rapid two-note staccato chirps with narrow bandpass filtering.
- **Spotted Towhee:** Low introductory trill followed by a wide-spectrum white noise modulated burst.
- **Hermit Thrush:** Multi-tonal harmonic chords utilizing dual carrier oscillators tuned to natural overtone series.
- **Olive Wren:** Soft descending chirps ($4.2\text{kHz} \rightarrow 2.1\text{kHz}$) with exponential decay envelopes.
- **Nightjar:** Continuous, hypnotic $24\text{Hz}$ rhythmic churring synthesized via low-frequency amplitude modulation.

---

### 8.2 Chorus Coordination & Collision Avoidance
- **Anti-Phase Staggering:** When multiple birds are scheduled to call, a client-side scheduling arbiter applies a pseudo-random stagger offset ($150\text{ms} - 800\text{ms}$). Birds never initiate vocalizations on the exact same audio buffer frame, avoiding unnatural phase reinforcement and audible comb filtering.
- **Contagion Window:** A call from a high `social_warmth` bird opens a 3-second response window during which neighboring birds evaluate a stochastic check to call back.

---

### 8.3 Listen-In Dynamic Audio Transition
- **Activation:** Focusing a bird (click or keyboard selection) initiates an asymptotic exponential gain ramp:
  - Focused bird gain ramps to $1.0$ over $800\text{ms}$ ($\tau = 0.25\text{s}$).
  - All non-focused birds ramp down to an ambient floor gain of $0.25$ over $1200\text{ms}$. Non-focused birds are **never muted completely**, preserving the sense of a shared physical room.
- **Deactivation:** Deselection ramps all bird channels back to baseline unity gain ($0.65$) over $1500\text{ms}$.

---

### 8.4 WebAudio Fallback
If the browser environment denies audio permissions or lacks WebAudio support:
- Audio context creation is bypassed without throwing exceptions.
- The system automatically activates the **Procedural Call Captioning** surface.
- No canned audio files or fallback audio formats are loaded, protecting the <2MB bundle budget.

---

## 9. Accessibility Surfaces

### 9.1 Screen-Reader Live Narration
- **ARIA Live Region:** Dedicated polite live region container (`<div aria-live="polite" aria-atomic="true" class="sr-only">`).
- **Pacing & Cadence:** 
  - Periodic background narration updates emitted every 30–60 seconds.
  - User-initiated actions (return greeting, offer acceptance, settle) bump the update queue immediately.
- **Naturalist Prose Synthesis Engine:** Client-side template grammar converts current snapshot coordinates and moods into naturalist prose:
  - *Idle state:* `"a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."`
  - *Offer reaction:* `"pip steps down to the front rail, tilts her head toward the offered seed, and pauses."`
  - *Settle action:* `"the light in the aviary softens to evening hues. calls grow quiet across the perches."`

---

### 9.2 Procedural Call Captioning
- **Opt-in / Auto-Fallback:** Activated via accessibility settings or automatically when audio context is unavailable.
- **Spatial Positioning:** Captions render as small, semi-transparent text bubbles positioned adjacent to the calling bird in the scene.
- **Grammar-to-Text Mapping:** Synthesizer motif triggers emit synchronized descriptive captions matching the active sound:
  - `"a soft three-note rise"`
  - `"a low trill, paused, low trill again"`
  - `"a single sharp call from the back perch"`
- **Fade Timing:** Captions fade in over $200\text{ms}$, hold for the duration of the call ($1 - 2.5\text{s}$), and fade out over $600\text{ms}$.

---

### 9.3 Keyboard Navigation & Focus Flow
1. **Top-Bar Level:** `Tab` cycles through `[Settings, Accessibility, Field Notebook, Offer Affordance, Settle]`.
2. **Aviary Scene Navigation:**
   - Pressing `Tab` into the aviary focuses the primary front bird.
   - `ArrowLeft` / `ArrowRight` cycles focus horizontally across active birds in spatial order.
   - `Enter` / `Space` engages **Listen-In** mode on the focused bird.
   - `Escape` disengages Listen-In mode and restores global ambient mix.
3. **Offer Flow:**
   - Shortcut `O` opens the offer drawer. Arrow keys select `[Seed, Song Motif, Still Pool]`. `Enter` commits the offer.
4. **Focus Indicator:** High-contrast double-ring outline ($2\text{px}$ white inner, $2\text{px}$ slate outer) visible across dawn, noon, dusk, and settled night scenes.

---

## 10. Performance Budgets, Optimization & Observability

### 10.1 Hard Performance Budgets

| Metric | Target Budget | Optimization Strategy |
| :--- | :--- | :--- |
| **Initial JS Bundle (Gzipped)** | **< 2.0 MB** | Tree-shaken Vanilla JS/TypeScript; procedural audio (zero WAV/MP3 files); SVG vector sprites; dynamic import for settings & modal flows. |
| **Time to First Bird Visible** | **< 500 ms** (4G mid-tier mobile) | Edge snapshot inlining in HTML; CSS-rendered sky gradient; zero blocking network calls before initial paint. |
| **Runtime Frame Rate** | **60 fps** (5-year-old laptop) | Canvas/WebGL batch rendering; zero DOM thrashing; object pooling for particles; CSS transforms for layer composition. |
| **Memory Leak Ceiling** | **0 MB / 30 mins** | Strict WebAudio node pooling and buffer reuse; cyclic reference clearing; bounded virtual scrolling in field notebook. |
| **Simulation Tick p99 Latency** | **< 5.0 s** | Sharded worker execution; indexed batch queries; bulk PostgreSQL updates. |

---

### 10.2 Observability & Privacy Boundaries

```
+----------------------------------------------------------------------------------------------------+
|                                    Telemetry Ingestion Boundary                                    |
+----------------------------------------------------------------------------------------------------+
| [ALLOWED OPERATIONAL TELEMETRY]                    | [STRICTLY FORBIDDEN / PRIVACY VIOLATIONS]     |
| - Edge HTTP request counts & latencies             | - Individual bird names, moods, or traits     |
| - Simulation tick batch execution duration (p99)   | - Per-account presence or interaction counts  |
| - WebAudio initialization success / failure rate   | - Field notebook text contents                |
| - Client-side FPS and render drop histograms       | - User-to-bird relationship metrics           |
| - Client JS error exception stack traces           | - Aggregate trait drift averages across users |
+----------------------------------------------------+-----------------------------------------------+
```

---

## 11. Phased Rollout & Lifecycle Strategy

```
+----------------------------------------------------------------------------------------------------+
|                                     Implementation & Rollout Plan                                  |
+----------------------------------------------------------------------------------------------------+
| PHASE 1: Core Engine & Single-Aviary Foundations (Weeks 1–4)                                      |
| - WebAudio procedural synthesizer & 6-species motif library                                        |
| - PostgreSQL schema, synthetic UUID auth & magic link flow                                         |
| - 60-second server simulation tick & presence validator                                            |
| - Canvas 60fps rendering pipeline with 3 perch zones & day/night lighting                          |
+----------------------------------------------------------------------------------------------------+
                                                  |
                                                  v
+----------------------------------------------------------------------------------------------------+
| PHASE 2: Interactions, Notebook & Accessibility (Weeks 5–7)                                        |
| - Return-greeting procedural engine & absence calibrator                                           |
| - Listen-in audio rebalance, Offer drawer, and Settle gesture                                      |
| - Field notebook sparse observation generator                                                      |
| - Live ARIA narration, call captions, and reduced-motion cross-fades                               |
+----------------------------------------------------------------------------------------------------+
                                                  |
                                                  v
+----------------------------------------------------------------------------------------------------+
| PHASE 3: Multi-Device Sync, Social Visits & Hardening (Weeks 8–10)                                 |
| - Snapshot polling, event streaming, and multi-device presence unioning                            |
| - Read-only visit invitation generation & token validation flow                                    |
| - CI automated performance test harness (bundle analyzer, 30-min memory leak monitor)              |
| - End-to-end drift calibration verification                                                        |
+----------------------------------------------------------------------------------------------------+
```

### 11.1 Progressive Bird Unlocking Schedule
Aviaries start strictly with 2 birds. Subsequent birds are made available at fixed aviary age milestones:
- **Bird 1 & 2:** Initial adoption at account setup.
- **Bird 3:** Aviary age = 30 days.
- **Bird 4:** Aviary age = 90 days.
- **Bird 5:** Aviary age = 180 days.
- **Bird 6:** Aviary age = 270 days.
- **Bird 7 (Cap):** Aviary age = 360 days.

---

## 12. Risk Management & Engineering Mitigations

### 12.1 Drift Calibration & Presence Inflation Risk
- **Risk:** Users leaving tabs open unattended in background windows could artificially accelerate personality drift, turning a multi-week progression into a few days.
- **Mitigation:** The 3-factor presence condition is enforced server-side: heartbeats must contain client-verified focus and recent input flags. Heartbeats exceeding 60s per wall-clock minute are rejected. Drift scaling uses a saturating square-root response curve for single-day presence over 2 hours.

### 12.2 Multi-Device Sync Inconsistency
- **Risk:** Simultaneous sessions on laptop and phone causing conflicting personality updates.
- **Mitigation:** Zero client-side state authoring. Clients only append discrete interaction events. The server simulation tick processes the single unified event log, eliminating last-write-wins hazards.

### 12.3 Procedural Audio Fatigue & Uncanny Valley
- **Risk:** Procedural synthesis sounding harsh, robotic, or repetitive over extended listening.
- **Mitigation:** High-precision periodic waves shaped with micro-Perlin pitch wobble, dynamic formant filtering, and non-repeating motif variations. Master convolver reverb creates natural room acoustics. Automatic mix ducking prevents auditory overload.

### 12.4 Screen-Reader Queue Saturation
- **Risk:** Frequent visual micro-motions spamming ARIA live regions and rendering the interface unusable for assistive tech users.
- **Mitigation:** Narration engine throttles idle updates to 30–60 second intervals. ARIA live container uses `polite` mode, allowing active screen-reader speech to complete naturally before delivering new naturalist observations.
