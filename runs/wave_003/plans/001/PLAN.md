# Pocket Aviary — System Implementation Plan (v1)

## 1. Executive Summary & Scope Boundary

Pocket Aviary is a persistent, browser-based ambient aviary where users form slow, observational relationships with a small group of animated birds. This document specifies the complete engineering architecture, data schemas, simulation mechanics, rendering and audio pipelines, accessibility provisions, operational budgets, and rollout strategies for Version 1 (v1).

### 1.1 In-Scope for v1
- **Platform**: Modern web browsers (desktop and mobile viewports) with no native application dependencies.
- **Population**: Exactly two starter birds per aviary at adoption, expanding strictly by aviary age up to a hard ceiling of seven birds. A fixed initial pool of six biological species silhouettes.
- **Visual Presentation**: Single horizontal viewport scene; no panning, zooming, or scrolling. Three distinct perch zones (front, middle, back). Dynamic day/night palette tied to user local time; rare, subtle ambient weather (rain, leaf/feather drift).
- **Core Interactions**:
  - *Presence*: Passive idle attention computed from a strict three-variable conjunction.
  - *Return-Greeting*: Procedurally varied, non-uniform greeting upon tab focus/navigation shaped by absence length, boldness, and mood.
  - *Listen-In*: Smooth dynamic audio mix re-balancing focusing on a single bird while keeping others as ambient background.
  - *Offer*: Soft interactions (seed, song fragment, still pool) gated by functional per-bird cooldowns.
  - *Settle*: Soft user-initiated session-ending gesture shifting scene to evening lighting, with a 5-second undo grace window; engine-equivalent to tab-close.
  - *Field Notebook*: Rare, auto-generated naturalist observations in lowercase present-tense prose; read-only with infinite historical scroll.
- **Audio Architecture**: 100% procedural WebAudio synthesis for all avian vocalizations (calls and choruses) with spatial positioning; zero recorded audio samples shipped in client bundle. Graceful fallback to silence with automatic call captions when WebAudio is unavailable.
- **Accounts & Multi-Device Sync**: Single-user accounts authenticated via 15-minute email magic links; revocable per-device session tokens; synthetic UUIDs separating identity from encrypted PII; server-authoritative simulation tick advancing canonical state; append-only client event streaming preventing Last-Write-Wins (LWW) anomalies; JSON account state export; 30-day soft deletion.
- **Quiet Social (Optional)**: Host-initiated read-only ambient visit invitations via one-time email links with 30-day expiration; immediate revocation; isolated visit log; zero visitor co-presence and zero visitor-induced drift.
- **Accessibility & Voice**:
  - Dual-register voice architecture: Naturalist (lowercase, present-tense, observational) for aviary, narration, notebook, and captions; Matter-of-Fact (standard capitalization, direct, functional) for auth, errors, settings, and sync notices.
  - Screen-reader running prose narration via `aria-live="polite"` on a 30–60 second cadence.
  - Fully articulated reduced-motion mode (slow alpha cross-fades between resting poses, removal of particle drift).
  - Dynamic call captions matching procedural audio parameters.
  - WCAG AA contrast compliance and complete keyboard navigation with dual-tone focus rings.

### 1.2 Explicit Non-Goals (Out of Scope for v1 and Beyond)
- **No Native Applications**: No iOS or Android binaries. Browser-only execution.
- **No Gamification Surfaces**: Absolutely no streak counters, visit tallies, green-dot activity calendars, badges, achievements, levels, experience points, or progress bars.
- **No Tamagotchi / Custodial Dynamics**: Birds never die, starve, fall sick, or express distress. Absence results in ambient quietness, never punitive decay or negative drift.
- **No Social Network Mechanics**: No public aviary discovery feeds, user profiles, following lists, comments, chat overlays, visitor avatars, or leaderboards.
- **No Monetization / Commercial Friction**: No paywalls, paid currency, cosmetic microtransactions, or tiered bird limits.
- **No Direct Avatar or Scene Placement Control**: Users cannot drag birds, customize perch layouts, or redecorate the aviary. Perch positions are strictly behavioral signals.
- **No Push / Out-of-App Engagement Pings**: No web push notifications, SMS, or marketing emails enticing users back to the aviary.

---

## 2. System Architecture & Service Topology

```
                                      +-------------------------------------------------------+
                                      |                     Web Browser                       |
                                      |  +-------------------------------------------------+  |
                                      |  | Presentation & Interaction Layer                |  |
                                      |  | - Canvas2D / WebGL Render Pipeline (60 FPS)     |  |
                                      |  | - WebAudio Procedural Syrinx Synthesizer        |  |
                                      |  | - Presence Monitor (Visibility + Focus + Input) |  |
                                      |  | - Screen-Reader Live Narrator & Call Captions   |  |
                                      |  +-------------------------------------------------+  |
                                      +---------------------------+---------------------------+
                                                                  |
                                             HTTPS / WSS          |  (State Snapshots &
                                            REST + Events         |   Event Stream Batches)
                                                                  v
                                      +-------------------------------------------------------+
                                      |                   Edge Proxy (CDN)                    |
                                      | - TLS Termination & Rate Limiting                     |
                                      | - Edge Cache for Static Assets (<2MB JS/CSS Bundle)   |
                                      | - Initial Aviary Snapshot Injection into HTML Shell   |
                                      +---------------------------+---------------------------+
                                                                  |
                                                                  v
+-------------------------------------------------------------------------------------------------------------------------+
|                                                   Aviary Core Services                                                  |
|                                                                                                                         |
|  +-----------------------------------+     +----------------------------------+     +--------------------------------+  |
|  |       API & Ingestion Service     |     |      Auth & Session Service      |     |     Visit & Social Service     |  |
|  | - Validates & batches events      |     | - Magic link generation/verify   |     | - Creates 30-day visit tokens  |  |
|  | - Serves canonical snapshots      |     | - Device session tracking        |     | - Serves read-only snapshots   |  |
|  | - Appends to Aviary Event Stream  |     | - Soft/Hard deletion workflows   |     | - Immediate invite revocation  |  |
|  +-----------------+-----------------+     +-----------------+----------------+     +---------------+----------------+  |
|                    |                                         |                                      |                   |
+--------------------|-----------------------------------------|--------------------------------------|-------------------+
                     |                                         |                                      |
                     v                                         v                                      v
+-------------------------------------------------------------------------------------------------------------------------+
|                                                    Persistence Tier                                                     |
|                                                                                                                         |
|  +----------------------------------------------------+    +---------------------------------------------------------+  |
|  |      Append-Only Event Store (PostgreSQL / WAL)    |    |        Transactional Relational Store (PostgreSQL)      |  |
|  | - Immutable interaction events                     |    | - Accounts (UUID synthetic keys, encrypted email)       |  |
|  | - Presence pings with client & server timestamps   |    | - Aviaries & Birds (Persistent identities)              |  |
|  | - High-throughput partitioned write log            |    | - Canonical State Snapshots & Personality Vectors       |  |
|  +-------------------------+--------------------------+    | - Field Notebook Entries & Visit Records                |  |
|                            |                               +----------------------------+----------------------------+  |
+----------------------------|------------------------------------------------------------|-------------------------------+
                             |                                                            |
                             +-----------------------------+------------------------------+
                                                           |
                                                           v
+-------------------------------------------------------------------------------------------------------------------------+
|                                           Simulation Engine Cluster (Background)                                        |
|                                                                                                                         |
|  +-------------------------------------------------------------------------------------------------------------------+  |
|  | Aviary Tick Workers (Stateless, Partitioned by Aviary UUID, 60-Second Cadence)                                    |  |
|  | - Validates presence-time integrity & evaluates monotonic low-pass drift filter                                   |  |
|  | - Computes fast-timescale mood transitions (diurnal solar cycle, weather, social contagion)                       |  |
|  | - Runs call-grammar scheduling & bird-to-bird antiphonal timing seeds                                             |  |
|  | - Evaluates rare naturalist notebook observation heuristics                                                      |  |
|  | - Commits new canonical snapshot atomically with `tick_version` increment                                         |  |
|  +-------------------------------------------------------------------------------------------------------------------+  |
+-------------------------------------------------------------------------------------------------------------------------+
                                                           |
                                                           v
+-------------------------------------------------------------------------------------------------------------------------+
|                                               Isolated Telemetry Pipeline                                               |
|  - InfluxDB / Prometheus / Datadog: Operational latency, tick runtimes, bundle delivery, audio errors, RUM FPS         |
|  - HARD BOUNDARY: Zero per-bird, per-vector, or per-user interaction events are ingested into analytics                 |
+-------------------------------------------------------------------------------------------------------------------------+
```

### 2.1 Service Boundaries & Responsibilities
1. **Presentation & Interaction Client**: Single-Page Application constructed with vanilla TypeScript and lightweight reactive primitives (no heavy UI frameworks). Renders the aviary to a Canvas2D or WebGL context, runs the WebAudio procedural voice synthesizer, monitors window visibility/focus/input, and dispatches batched interaction events.
2. **Edge Proxy / CDN**: Terminates TLS, caches static bundles, performs token validation, applies IP-rate limits, and injects the initial aviary snapshot payload directly into the root HTML document stream to guarantee the <500ms time-to-first-bird metric.
3. **Aviary API Service**: Stateless HTTP application service handling authentication magic-link flows, state snapshot queries, event ingestion into an append-only log, and visit link routing.
4. **Simulation Engine Cluster**: Distributed background worker cluster that drives the 60-second aviary tick. Workers acquire exclusive distributed locks (PostgreSQL row locks or Redis distributed leases) per aviary partition, consume unprocessed events, calculate drift, transition moods, generate notebook prose, and publish canonical snapshots.
5. **Persistence Tier**: Multi-AZ PostgreSQL database acting as both the relational system of record and the append-only event log.
6. **Isolated Operational Telemetry**: Segregated time-series datastore capturing operational metrics (p99 tick duration, HTTP latencies, audio error rates). Strictly isolated by network policy and ingestion schema from user identity and simulation state.

---

## 3. Data Model & Database Schemas

All internal references to accounts leverage synthetic UUIDv4 primary keys. Email addresses are encrypted at rest using AES-256-GCM. Personality vectors are stored exclusively on the server and are strictly excluded from client serialization payloads.

### 3.1 Relational Schemas (PostgreSQL DDL)

```sql
-- Core Accounts Table
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_hash VARCHAR(64) NOT NULL UNIQUE, -- SHA-256 with salt for lookups
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    deletion_requested_at TIMESTAMPTZ,      -- Soft delete timestamp; hard delete at +30 days
    notify_on_visit BOOLEAN NOT NULL DEFAULT FALSE
);
CREATE INDEX idx_accounts_email_hash ON accounts(email_hash);
CREATE INDEX idx_accounts_deletion ON accounts(deletion_requested_at) WHERE deletion_requested_at IS NOT NULL;

-- Device Sessions Table
CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    session_token_hash VARCHAR(64) NOT NULL UNIQUE,
    user_agent TEXT,
    ip_prefix VARCHAR(45),                 -- Truncated /24 or /48 IP for audit without PII
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    last_seen_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    revoked_at TIMESTAMPTZ
);
CREATE INDEX idx_sessions_account ON sessions(account_id) WHERE revoked_at IS NULL;

-- Magic Links Table
CREATE TABLE magic_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    expires_at TIMESTAMPTZ NOT NULL,       -- Exactly created_at + 15 minutes
    consumed_at TIMESTAMPTZ
);
CREATE INDEX idx_magic_links_lookup ON magic_links(token_hash) WHERE consumed_at IS NULL;

-- Aviary Table (Single aviary per account)
CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    tick_version BIGINT NOT NULL DEFAULT 0,
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    settled_at TIMESTAMPTZ,
    settled_until TIMESTAMPTZ,
    active_weather VARCHAR(32) NOT NULL DEFAULT 'clear', -- 'clear', 'soft_rain', 'leaf_breeze'
    weather_until TIMESTAMPTZ
);
CREATE INDEX idx_aviaries_tick ON aviaries(last_tick_at);

-- Birds Table (Stable identities)
CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL,       -- 'warbler', 'chickadee', 'finch', 'nuthatch', 'sparrow', 'nightjar'
    name VARCHAR(32) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    current_perch_zone VARCHAR(16) NOT NULL DEFAULT 'middle', -- 'front', 'middle', 'back'
    current_mood VARCHAR(16) NOT NULL DEFAULT 'content',      -- 'wary', 'content', 'curious', 'drowsy', 'alert'
    mood_entered_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    last_greeting_at TIMESTAMPTZ
);
CREATE INDEX idx_birds_aviary ON birds(aviary_id);

-- Personality Vectors (Server-side canonical only; NEVER sent to client)
CREATE TABLE bird_personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(id) ON DELETE CASCADE,
    boldness REAL NOT NULL CHECK (boldness >= 0.0 AND boldness <= 1.0),
    social_warmth REAL NOT NULL CHECK (social_warmth >= 0.0 AND social_warmth <= 1.0),
    vocal_frequency REAL NOT NULL CHECK (vocal_frequency >= 0.0 AND vocal_frequency <= 1.0),
    plumage_saturation REAL NOT NULL CHECK (plumage_saturation >= 0.0 AND plumage_saturation <= 1.0),
    curiosity REAL NOT NULL CHECK (curiosity >= 0.0 AND curiosity <= 1.0),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

-- Append-Only Interaction Event Log
CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    session_id UUID NOT NULL REFERENCES sessions(id) ON DELETE CASCADE,
    bird_id UUID REFERENCES birds(id) ON DELETE CASCADE,
    event_type VARCHAR(32) NOT NULL,       -- 'presence_ping', 'listen_in_start', 'listen_in_end', 'offer', 'settle', 'settle_undo'
    event_payload JSONB NOT NULL DEFAULT '{}',
    client_timestamp TIMESTAMPTZ NOT NULL,
    server_received_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    processed_by_tick BIGINT               -- NULL until consumed by simulation tick
);
CREATE INDEX idx_events_unprocessed ON interaction_events(aviary_id, id) WHERE processed_by_tick IS NULL;

-- Field Notebook Observations
CREATE TABLE notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    recorded_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    observation_text TEXT NOT NULL,        -- Naturalist lowercase present-tense prose
    trigger_type VARCHAR(32) NOT NULL      -- Operational categorization, e.g. 'pip_greeted_first'
);
CREATE INDEX idx_notebook_aviary ON notebook_entries(aviary_id, recorded_at DESC);

-- Visit Invitations
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    visitor_email_encrypted BYTEA NOT NULL,
    visitor_email_masked VARCHAR(64) NOT NULL, -- e.g. 'e***@example.com'
    created_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    expires_at TIMESTAMPTZ NOT NULL,           -- created_at + 30 days
    revoked_at TIMESTAMPTZ
);
CREATE INDEX idx_visit_invitations_token ON visit_invitations(token_hash) WHERE revoked_at IS NULL;

-- Visit Audit Log (Viewable by host)
CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invitation_id UUID NOT NULL REFERENCES visit_invitations(id) ON DELETE CASCADE,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    visited_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    duration_seconds INT NOT NULL DEFAULT 0
);
CREATE INDEX idx_visit_logs_aviary ON visit_logs(aviary_id, visited_at DESC);
```

---

## 4. API Surface & Protocols

All API endpoints communicate using JSON over HTTPS. Authenticated requests present an `HttpOnly`, `Secure`, `SameSite=Strict` session cookie containing a cryptographically signed token.

### 4.1 Authentication Endpoints
- `POST /api/v1/auth/magic-link`: Submits `{ "email": "user@example.com" }`. Rate-limited to 3 requests per 15 minutes per IP/email. Returns `200 OK` with `{ "status": "sent" }` (constant-time response preventing account enumeration).
- `POST /api/v1/auth/verify`: Consumes `{ "token": "..." }`. Validates single-use hash, invalidates token immediately, sets session cookie, returns `200 OK` with account metadata.
- `POST /api/v1/auth/session/revoke`: Revokes current or specified `session_id`.
- `POST /api/v1/account/export`: Generates JSON snapshot of user's birds, names, moods, and notebook history. Dispatched via signed download URL to verified email.
- `POST /api/v1/account/delete`: Initiates 30-day soft deletion.
- `POST /api/v1/account/restore`: Cancels pending deletion within the 30-day window.

### 4.2 Aviary State & Event Protocol
- `GET /api/v1/aviary/state`: Retrieves the current canonical state snapshot. Returns ETag based on `tick_version`.
  ```json
  {
    "tick_version": 10452,
    "timestamp": "2026-09-06T14:48:00Z",
    "solar_phase": "morning",
    "weather": "clear",
    "settled": false,
    "birds": [
      {
        "id": "7b88ec7b-9442-4f76-8ff4-934304e768e1",
        "species_id": "warbler",
        "name": "pip",
        "perch_zone": "front",
        "mood": "curious",
        "facing": "right",
        "active_motif_id": "warbler_call_high_rise",
        "call_seed": 928371
      },
      {
        "id": "e93db941-8631-482a-8984-25e1aaefea29",
        "species_id": "chickadee",
        "name": "wren",
        "perch_zone": "back",
        "mood": "content",
        "facing": "left",
        "active_motif_id": "chickadee_fee_bee",
        "call_seed": 104928
      }
    ]
  }
  ```
  *(Note: Personality vector floating-point numbers are strictly absent from this payload).*

- `POST /api/v1/aviary/events`: Client flushes batched interaction events every 15–30 seconds.
  ```json
  {
    "events": [
      {
        "event_type": "presence_ping",
        "client_timestamp": "2026-09-06T14:48:15Z",
        "payload": { "duration_ms": 15000 }
      },
      {
        "event_type": "listen_in_start",
        "bird_id": "7b88ec7b-9442-4f76-8ff4-934304e768e1",
        "client_timestamp": "2026-09-06T14:48:22Z",
        "payload": {}
      }
    ]
  }
  ```

- `POST /api/v1/aviary/offers`: Submits an offer.
  ```json
  {
    "offer_type": "seed",
    "client_timestamp": "2026-09-06T14:49:00Z"
  }
  ```
  Returns `200 OK` with recipient bird's accepted/ignored reaction, or `429 Too Many Requests` (Matter-of-Fact: `"Offer is on cooldown. Please wait a few moments."`).

- `POST /api/v1/aviary/settle`: Triggers settle gesture.
- `POST /api/v1/aviary/settle/undo`: Reverses settle gesture if received within 5,000ms.
- `GET /api/v1/notebook`: Returns paginated notebook entries ordered by `recorded_at DESC`.

### 4.3 Social & Visit Endpoints
- `POST /api/v1/visits/invite`: Host submits `{ "email": "friend@example.com" }`. System creates single-use 30-day invitation token.
- `DELETE /api/v1/visits/invite/:id`: Host revokes invitation immediately.
- `GET /api/v1/visits/log`: Host inspects list of past visits with masked emails and durations.
- `GET /api/v1/visits/:token/state`: Visitor endpoint. Validates token validity and active status. Returns read-only canonical aviary snapshot. Rejects any event submissions (`403 Forbidden`). When revoked, immediately returns `404 Not Found` or `410 Gone` with Matter-of-Fact copy: `"This visit invitation is no longer active."`

---

## 5. Simulation Engine Design

The simulation engine is the computational core executing the server-side tick. It runs independently of whether any client browser is active.

```
+-----------------------------------------------------------------------------------------------+
|                             Server-Side Simulation Tick (Every 60s)                           |
+-----------------------------------------------------------------------------------------------+
                                                |
                                                v
             +---------------------------------------------------------------------+
             | 1. Ingestion & Event Verification                                   |
             | - Query unprocessed events for aviary with ROW LOCK                 |
             | - Validate presence pings against server timestamps (Anti-Tamper)   |
             | - Total valid presence seconds: S_p in [0, 60]                      |
             +----------------------------------+----------------------------------+
                                                |
                                                v
             +---------------------------------------------------------------------+
             | 2. Monotonic Low-Pass Drift Filter Evaluation                       |
             | - For each bird trait T in {bold, warm, freq, plum, cur}:           |
             |     delta_T = alpha_T * Weight(T, events) * (S_p / 60)              |
             |     T_new = min(1.0, T_old + max(0.0, delta_T))                     |
             | - STRICT INVARIANT: delta_T >= 0.0 (Zero negative drift on neglect) |
             +----------------------------------+----------------------------------+
                                                |
                                                v
             +---------------------------------------------------------------------+
             | 3. Fast-Timescale Mood Transitions & Contagion                      |
             | - Evaluate diurnal solar angle from client timezone offset          |
             | - Factor recent offers, weather, and peer alarm/contagion           |
             | - Compute target perch zones: Front / Middle / Back                 |
             +----------------------------------+----------------------------------+
                                                |
                                                v
             +---------------------------------------------------------------------+
             | 4. Call Grammar & Antiphonal Stagger Scheduling                     |
             | - Generate Poisson call frequency rates from vocal_frequency        |
             | - Stagger chorus responses: Delta_t in [1.2s, 3.5s] via warmth      |
             +----------------------------------+----------------------------------+
                                                |
                                                v
             +---------------------------------------------------------------------+
             | 5. Rare Field Notebook Prose Synthesizer                            |
             | - Evaluate noteworthy predicates (e.g. Pip greeted before Wren)     |
             | - Apply sparsity gate: p ~ 0.05 / day (Max 1 entry per 3-5 days)    |
             +----------------------------------+----------------------------------+
                                                |
                                                v
             +---------------------------------------------------------------------+
             | 6. Atomic Snapshot Commit                                           |
             | - Write new canonical state, increment tick_version, release lock   |
             +---------------------------------------------------------------------+
```

### 5.1 Presence Accounting Engine
A presence event is valid if and only if:
$$\text{Presence} = V_{\text{visible}} \land F_{\text{focus}} \land A_{\text{activity}}$$
- $V_{\text{visible}}$: `document.visibilityState === 'visible'`.
- $F_{\text{focus}}$: `document.hasFocus() === true`.
- $A_{\text{activity}}$: At least one hardware input event (`pointermove`, `keydown`, `wheel`) has occurred within the calibrated rolling activity window $W_{\text{act}} = 180\text{ seconds}$ (3 minutes).

The client pings every 15 seconds with accumulated presence. The server validates that the reported elapsed presence $\Delta t_{\text{client}}$ does not exceed the wall-clock interval $\Delta t_{\text{server}} + \epsilon$. Clocks that drift or attempt burst pings are clamped to $\min(\Delta t_{\text{client}}, \Delta t_{\text{server}})$.

### 5.2 The Drift Function
Drift governs slow-timescale personality evolution. It is mathematically modeled as an asymmetric leaky low-pass integrator with zero negative leakage:

$$T_{i}(t + \Delta t) = T_{i}(t) + \alpha_{i} \cdot \Phi_{i}(\mathbf{E}) \cdot \frac{S_{\text{presence}}}{60.0}$$

Where:
- $T_{i} \in [0.0, 1.0]$ is trait $i \in \{\text{boldness}, \text{social\_warmth}, \text{vocal\_frequency}, \text{plumage\_saturation}, \text{curiosity}\}$.
- $\alpha_{i}$ is the calibrated learning rate:
  - Instrument threshold ($1$ week regular use $\approx 7$ hours presence): $\Delta T \approx 0.04$.
  - User-noticeable threshold ($3$ weeks regular use $\approx 21$ hours presence): $\Delta T \approx 0.12$.
  - Nominal $\alpha \approx 1.58 \times 10^{-6}$ per presence second.
- $\Phi_{i}(\mathbf{E}) \ge 0$ is the interaction weight vector:
  - $\Phi_{\text{boldness}} = 1.0 + 0.5 \cdot \mathbf{1}_{\{\text{offer\_proximity}\}}$.
  - $\Phi_{\text{social\_warmth}} = 1.0 + 1.2 \cdot \mathbf{1}_{\{\text{listen\_in}\}} + 0.3 \cdot \mathbf{1}_{\{\text{chorus\_joined}\}}$.
  - $\Phi_{\text{vocal\_frequency}} = 1.0 + 0.8 \cdot \mathbf{1}_{\{\text{listen\_in}\}}$.
  - $\Phi_{\text{plumage\_saturation}} = 1.0$ (Strictly dependent on presence accumulation).
  - $\Phi_{\text{curiosity}} = 1.0 + 1.5 \cdot \mathbf{1}_{\{\text{offer\_investigated}\}}$.
- **Monotonicity Law**: $\Delta T_{i} \ge 0.0$ always. When presence is zero ($S_{\text{presence}} = 0$), $\Delta T_{i} = 0$. Neglect never reduces a trait.

### 5.3 Mood State Machine
Fast-timescale mood $M \in \{\text{wary}, \text{content}, \text{curious}, \text{drowsy}, \text{alert}\}$ operates on a continuous-time Markov process:
1. **Diurnal Cycle**: Solar elevation angle $\theta_{\text{sun}}$ is derived from local timezone offset. When $\theta_{\text{sun}} < -6^\circ$ (night), transition probability to `drowsy` / `settled` increases to $0.95$ (except for nocturnal species like `nightjar`, which becomes `alert`). Morning dawn raises `alert` and `content`.
2. **Interaction Modulation**:
   - Accepted offer: $P(\text{curious} \to \text{content}) = 0.8$.
   - Settle gesture: Forces transition of all birds to `drowsy` / `settled` over 4 seconds.
   - Long absence return: Initial state defaults to `wary` or `alert` for low-boldness birds, `content` for high-boldness birds.
3. **Social Contagion**: If bird $A$ enters `wary`, neighboring birds within the same perch zone roll transition to `wary`:
   $$P(\text{contagion}) = (1.0 - \text{boldness}_{B}) \times \text{social\_warmth}_{B} \times 0.4$$

### 5.4 Call Grammar Runtime & Antiphony
Vocalizations follow a generative motif grammar:
- Each species defines 4–6 motif graphs (sequences of syllable envelopes, frequency sweeps, and harmonic ratios).
- Call trigger rate follows an inhomogeneous Poisson process:
  $$\lambda(t) = \lambda_{0} \times \text{vocal\_frequency} \times \mu_{\text{mood}} \times \mu_{\text{weather}}$$
  where $\mu_{\text{drowsy}} = 0.2$, $\mu_{\text{content}} = 1.0$, $\mu_{\text{alert}} = 1.5$, $\mu_{\text{rain}} = 0.4$.
- **Antiphonal Staggering**: When bird $A$ completes a call, bird $B$ calculates reply probability:
  $$P(\text{reply}) = \text{social\_warmth}_{B} \times 0.6$$
  If triggered, bird $B$'s call is scheduled with a randomized delay $\Delta t_{\text{stagger}} \sim \mathcal{U}(1.2, 3.5)\text{ seconds}$. This prevents synchronous cacophony and creates natural avian dialogue.

### 5.5 Field Notebook Generation Engine
The notebook is an automated naturalist diary.
- **Trigger Detection**: Evaluated at tick boundaries:
  - `FIRST_GREET_SHIFT`: Bolder bird Pip greeted before Wren for the first time in 5+ sessions.
  - `LONG_QUIET_PREEN`: Aviary experienced >15 minutes of uninterrupted presence with zero offers and low vocal frequency.
  - `WEATHER_RESPONSE`: Transition into soft rain with birds fluffing on lower perches.
- **Sparsity Filter**: Gate probability $P_{\text{emit}} = 0.05$ per tick where a trigger is active, capped at maximum 1 entry per 72 hours under regular visitation.
- **Prose Synthesizer**: Produces lowercase, present-tense, evocative naturalist prose:
  - *"tuesday — wren is on the low perch this morning, fluffed against the cool air. pip greeted first today — only by a beat, but first."*
  - *"a long stretch of quiet this morning. pip preened for several minutes without looking up."*

---

## 6. Multi-Device Synchronization & Conflict Resolution

```
[Device A: Laptop (Work)]                         [Cloud Simulation Core]                 [Device B: Mobile (Commute)]
           |                                                 |                                          |
           |-- 1. Ingest Events (Presence, Offers) --------->|                                          |
           |   (Appended to Postgres Event Log)              |                                          |
           |                                                 |                                          |
           |                                                 |<- 2. Connects & Requests State Snapshot -|
           |                                                 |-- 3. Returns Canonical Snapshot S_104 ->|
           |                                                 |      (Birds mid-motion, mood synced)     |
           |                                                 |                                          |
           |                                                 |<- 4. Sends Presence Ping ---------------|
           |                                                 |                                          |
           |                                     [Tick Runs at t=60s]                                   |
           |                                     - Consumes events A & B                                |
           |                                     - Applies monotonic drift                              |
           |                                     - Commits Snapshot S_105                               |
           |                                                 |                                          |
           |<- 5. Poll / Keepalive Pull ---------------------|----------------------------------------->|
           |-- Receives S_105                                |   Receives S_105                         |
           |-- Smoothly interpolates to new perch/mood       |   Smoothly interpolates to new perch/mood|
```

### 6.1 Server-Authoritative State Invariant
Clients are strictly render and input capture nodes. Under no circumstances does a client calculate, store, or submit personality vectors, mood states, or aviary clock variables.
- **No Last-Write-Wins (LWW)**: Because clients never submit absolute state representations, the classic distributed system race condition—where a mobile device with stale state overwrites updates made by a desktop—is architecturally impossible.
- **Additive Server Deltas**: All state mutations are server-calculated derivations of the ordered, append-only event stream.
- **Concurrent Device Behavior**: If a user has both a laptop tab and mobile browser open simultaneously, both stream presence pings. The server simulation tick coalesces overlapping intervals, preventing double-counting of physical presence hours.

### 6.2 Snapshot Consumption & Client Interpolation
- **Polling Cadence**: Clients poll `GET /api/v1/aviary/state` every 30 seconds while `document.visibilityState === 'visible'`.
- **Immediate Re-Sync Triggers**:
  - `document.addEventListener('visibilitychange')`: When transitioning from `hidden` to `visible`.
  - `window.addEventListener('online')`: Reconnection following network drop.
  - Frame delta gap: When `performance.now() - last_frame_time > 5000ms` (detecting OS sleep/wake).
- **Hermite Coordinate & Pose Interpolation**: When snapshot $S_{k+1}$ arrives with a bird at a different perch or orientation than $S_{k}$, the client does not snap. It initiates a procedural hop or flight arc taking 1,200ms using a cubic Hermite spline for position and smooth rotation vectors.

---

## 7. Frontend Rendering Pipeline

The visual presentation is engineered to deliver 60 FPS performance on legacy hardware while guaranteeing that the aviary appears mid-motion from frame 0.

### 7.1 Visual Scene Composition & Layer Hierarchy
The scene is rendered within a fixed aspect-ratio virtual canvas ($1920 \times 1080$ virtual coordinates) dynamically mapped to the viewport with letterboxing/pillarboxing protection:

```
+-------------------------------------------------------------------------------+
| Layer 6: Top-Bar Chrome (Icons: Account, Accessibility, Notebook, Offer)      |
|          Fades to opacity 0.0 after 3,000ms cursor stillness; returns on input|
+-------------------------------------------------------------------------------+
| Layer 5: Ambient Particulate Drift (Client-side procedural leaves & feathers) |
+-------------------------------------------------------------------------------+
| Layer 4: Front Perch Zone & Water Basin / Offer Drop Landing                  |
|          Scale: 1.0x, Full Saturation, 0px Blur                                |
+-------------------------------------------------------------------------------+
| Layer 3: Middle Perch Zone & Midground Foliage                                |
|          Scale: 0.82x, Neutral Saturation, 0px Blur                           |
+-------------------------------------------------------------------------------+
| Layer 2: Back Perch Zone & Deep Canopy                                        |
|          Scale: 0.65x, -10% Saturation, 1.5px Depth Blur                      |
+-------------------------------------------------------------------------------+
| Layer 1: Parallax Landscape Silhouette (Subtle +/-12px mouse parallax)       |
+-------------------------------------------------------------------------------+
| Layer 0: Sky Dome (Continuous diurnal gradient: Dawn -> Noon -> Dusk -> Night)|
+-------------------------------------------------------------------------------+
```

### 7.2 The "First Frame Mid-Action" Invariant
Traditional web applications render a loading spinner or static splash screen that fades into active animation. Pocket Aviary strictly forbids entry transitions, spinners, or static fades.
- **Deterministic Phase Initialization**:
  Upon receiving snapshot $S$, the client calculates each bird's continuous animation cycle phase $\theta_{\text{anim}}$:
  $$\theta_{\text{anim}} = \left( \frac{\text{client\_boot\_time} + \text{seed}_{\text{bird}}}{\tau_{\text{cycle}}} \right) \pmod{1.0}$$
  Frame 0 immediately draws the bird with its wings partially tucked, chest expanded in mid-breath, or head tilted.
- **Cold Cache / Slow Connection Fallback**:
  If the network snapshot requires >200ms to arrive, the client renders the empty ambient sky and foliage with leaf drift immediately active (the "quiet field"). Once the snapshot arrives, the birds take their perches via a natural soft flight entrance. A loading spinner is never displayed.

### 7.3 Micro-Motion Procedural Engine
Idle aliveness is driven by procedural kinematics rather than repetitive sprite loops:
- **Respiration**: Sinusoidal chest scaling ($y$-scale modulation of $\pm 1.8\%$ at $0.3\text{ Hz}$).
- **Head Saccades**: Discrete, quick head rotations ($\Delta \theta \in [-25^\circ, +25^\circ]$ over $60\text{ ms}$) triggered on Poisson intervals, reflecting bird visual scanning.
- **Preening**: Periodic feather-ruffling shaders modulating vertex offsets on plumage contours.
- **Perch Shuffle**: Weight-shifting side hops when mood is `alert` or `curious`.

### 7.4 Reduced-Motion Mode Architecture
When `(prefers-reduced-motion: reduce)` matches or when toggled in Accessibility Settings:
1. Continuous skeletal oscillations (breathing, tail wagging, leaf flutter) are disabled.
2. Ambient leaf and feather drift particles are removed entirely.
3. Flight paths between perches are eliminated.
4. **Cross-Fade Pose System**: Birds transition between discrete static poses (resting, alert scan, tuck) via a slow 1,500ms alpha cross-fade. Position changes between perches are executed via an elegant cross-dissolve: the bird softly dissolves from the back perch while materializing on the front perch.
5. Day/night lighting transitions extend from seconds to minutes with smoothed color interpolation.

---

## 8. Procedural Audio Pipeline

Audio is synthesized 100% procedurally on the client using the WebAudio API. No audio samples (WAV/MP3/OGG) exist in the application bundle.

```
+-----------------------------------------------------------------------------------------------+
|                                WebAudio Procedural Syrinx Graph                               |
|                                                                                               |
|  +---------------------------+                                                                |
|  | Modulator Oscillator      |                                                                |
|  | (FM Synthesis: 800-3200Hz)|                                                                |
|  +-------------+-------------+                                                                |
|                |                                                                              |
|                v Frequency Mod                                                                |
|  +---------------------------+       +-------------------+       +-------------------------+  |
|  | Carrier Oscillator        | ----> | Biquad Bandpass   | ----> | Gain Envelope (ADSR)    |  |
|  | (Sine/Triangle Syrinx)    |       | (Resonant Syrinx) |       | (Exponential Ramp)      |  |
|  +---------------------------+       +-------------------+       +------------+------------+  |
|                                                                               |               |
|  +---------------------------+       +-------------------+                    |               |
|  | White Noise Generator     | ----> | Highpass Filter   | -------------------+               |
|  | (Air breath / aspiration) |       | (Breath Whisper)  |                    |               |
|  +---------------------------+       +-------------------+                    v               |
|                                                                  +-------------------------+  |
|                                                                  | Bird Channel Gain Node  |  |
|                                                                  +------------+------------+  |
+-------------------------------------------------------------------------------|---------------+
                                                                                |
                                                                                v
+-----------------------------------------------------------------------------------------------+
|                                      Spatial & Chorus Mixer                                   |
|                                                                                               |
|  +----------------------------------------------------+                                       |
|  | Stereo Panner Node (-0.8 to +0.8 based on X pos)   |                                       |
|  +-------------------------+--------------------------+                                       |
|                            |                                                                  |
|                            v                                                                  |
|  +----------------------------------------------------+                                       |
|  | Distance Filter & Attenuation Node                 |                                       |
|  | - Front Perch:  0 dB, 16 kHz lowpass               |                                       |
|  | - Middle Perch: -3 dB, 9 kHz lowpass               |                                       |
|  | - Back Perch:   -6 dB, 5 kHz lowpass + Reverb      |                                       |
|  +-------------------------+--------------------------+                                       |
|                            |                                                                  |
|                            v                                                                  |
|  +----------------------------------------------------+       +----------------------------+  |
|  | Master Dynamic Range Compressor & Limiter          | ----> | AudioContext.destination   |  |
|  | (Prevents clipping during multi-bird chorus)       |       +----------------------------+  |
|  +----------------------------------------------------+                                       |
+-----------------------------------------------------------------------------------------------+
```

### 8.1 Physical Syrinx Synthesis Architecture
Avian vocalization is generated by simulating the bipartite syrinx:
- **Dual FM Core**: A carrier sine oscillator modulated by a harmonic modulator oscillator creates species-accurate frequency chirps, trills, and complex overtones without phase cancelation.
- **Vocal Tract Resonance**: Two cascading `BiquadFilterNode` instances (peaking and bandpass) shape formant characteristics.
- **Aspiration Noise**: A lightweight procedural noise buffer passed through a high-pass filter ($f_c = 4.5\text{ kHz}$) injects breathiness into call onsets.
- **Motif Parameters**: Stored as compact JSON structs (~400 bytes per species) defining frequency envelopes, FM depths, and attack/decay time constants.

### 8.2 Spatial Chorus & Dynamic Mix Engine
Each active bird connects to a dedicated channel strip:
- **Stereo Panning**: `StereoPannerNode` maps horizontal screen coordinate $x \in [0.1, 0.9]$ to pan value $[-0.8, +0.8]$.
- **Depth Filtering**:
  - Front zone: Direct signal, $0\text{ dB}$, high-frequency cutoff $18\text{ kHz}$.
  - Middle zone: $-3\text{ dB}$ attenuation, $9\text{ kHz}$ cutoff.
  - Back zone: $-6\text{ dB}$ attenuation, $5\text{ kHz}$ cutoff with $15\%$ wet send to an ambient convolution reverb simulating outdoor space.
- **Master Bus**: A `DynamicsCompressorNode` (threshold: $-12\text{ dB}$, ratio: $4:1$, attack: $0.005\text{ s}$, release: $0.1\text{ s}$) ensures multi-bird chorus events blend cleanly without clipping.

### 8.3 Listen-In Dynamic Mix Decay
When a user engages listen-in on bird $k$:
1. **Focused Channel**: The gain of bird $k$ transitions to $+3\text{ dB}$ over $800\text{ ms}$ using `gainNode.gain.exponentialRampToValueAtTime(1.41, ctx.currentTime + 0.8)`. A high-shelf filter applies $+2\text{ dB}$ at $3.5\text{ kHz}$ to enhance intimacy and presence.
2. **Background Attenuation**: All other bird channels ramp down by $-10\text{ dB}$ over $800\text{ ms}$. Crucially, **they are never silenced**, maintaining background aliveness.
3. **Disengagement**: When focus is released, all channel gains return to standard baseline levels over a slow $1,200\text{ ms}$ ramp.

### 8.4 WebAudio Fallback & Failure Modes
If `AudioContext` is unsupported, blocked by browser autoplay policies, or fails to initialize:
- The aviary automatically runs in **Graceful Silence Mode**.
- Call captions are automatically enabled by default.
- Under NO circumstance does the system fall back to recorded MP3/WAV loops (strictly adhering to the "no canned audio" constraint).

---

## 9. Accessibility Surfaces & UX Voice Architecture

Accessibility is implemented as an emotional, first-class design surface rather than a compliance checklist.

### 9.1 The Dual-Register Voice System
The voice architecture is divided into two mutually exclusive registers with zero cross-contamination:

| Dimension | Naturalist Voice (Product & Aviary) | Matter-of-Fact Voice (System & Infrastructure) |
| :--- | :--- | :--- |
| **Applicable Surfaces** | Aviary canvas, Screen-reader narration, Call captions, Field notebook, Offer prompts, Settle confirmation. | Sign-in, Magic-link emails, Session timeout, Sync errors, Account settings, Accessibility toggles, Visit revocation. |
| **Stylistic Rules** | Strictly lowercase; present-tense; specific to individual bird and moment; observational naturalist tone; no tech jargon; no exclamations; no announcement framing. | Standard sentence capitalization; direct, clear, declarative English; functional instructions; no false warmth or performative charm. |
| **Examples** | - *"wren is on the low perch this morning, fluffed against the cool air."*<br>- *"pip called softly from the front rail."* | - *"We couldn't sign you in. The link may have expired. Try requesting a new link."*<br>- *"This visit invitation is no longer active."* |

### 9.2 Screen-Reader Running Prose Narration
Rather than emitting robotic ARIA attribute updates (`aria-label="pip at perch 1"`), the system maintains an `aria-live="polite"` live region (`#aviary-live-narration`) populated with running naturalist prose:
- **Idle Cadence**: Generates an observational sentence every 30–60 seconds matching the visual state:
  *"a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."*
- **Event-Driven Interrupts**: Significant interactions bump the queue immediately with observational prose:
  - *Return-Greeting*: *"pip glances up from the front perch and calls twice."*
  - *Offer Reaction*: *"wren hops down to investigate the scattered seed, pausing to tilt her head."*
  - *Settle Gesture*: *"the light softens into evening; calls quiet across the perches."*

### 9.3 Procedural Call Captions
When call captions are enabled, small floating caption cards render adjacent to the calling bird:
- Captions are derived from the procedural motif parameters in real time.
- Phrasing examples:
  - *"a soft three-note rise"*
  - *"a low trill, paused, low trill again"*
  - *"a single sharp call from the back perch"*
- Visual styling: Muted translucent background with WCAG AA compliant text ($4.5:1$ minimum contrast ratio against aviary lighting), fading in and out synchronously with the audio envelope.

### 9.4 Keyboard Navigation & Focus Ring Standards
- **Tab Sequence**: Top bar items (Account $	o$ Accessibility $	o$ Notebook $	o$ Offer $	o$ Settle) $	o$ Aviary Canvas $	o$ Birds ordered left-to-right across perches.
- **Perch Traversal**: Left/Right arrow keys traverse birds sequentially; Up/Down keys step across perch zones (Front $\leftrightarrow$ Middle $\leftrightarrow$ Back).
- **Action Keys**:
  - `Enter` / `Space`: Toggle Listen-In on focused bird.
  - `Escape`: Disengage Listen-In or dismiss open modal overlay.
  - `O`: Open Offer palette.
  - `S`: Trigger Settle gesture.
- **Focus Rings**: Dual-concentric outline ($2\text{px}$ inner white `#FFFFFF`, $2\text{px}$ outer deep slate `#1A202C`) guaranteeing distinct visibility across bright morning and dim evening lighting.

---

## 10. Performance Budgets, Asset Optimization & Observability

### 10.1 Hard Performance Budgets

| Metric | Budget Ceiling | Enforcement Mechanism |
| :--- | :--- | :--- |
| **Initial JS Bundle Size** | $< 2.0\text{ MB}$ (gzipped) | Automated CI check via `bundlesize` failing pull requests exceeding $1.8\text{ MB}$. |
| **Time to First Bird (TTFBird)** | $< 500\text{ ms}$ on mid-tier mobile (4G) | Edge CDN snapshot inlining; critical render path with zero blocking asset downloads. |
| **Runtime Frame Rate** | $60\text{ FPS}$ sustained over 30 mins | Automated headless Chrome benchmark on 5-year-old reference CPU allocation. |
| **Memory Growth** | $0.0\text{ MB}$ leak over 30 mins | Headless Puppeteer heap snapshot delta test in CI nightly build. |
| **Server Simulation Tick Latency**| $p99 < 5.0\text{ s}$ across all partitions | Real-time Datadog / Prometheus alerting on tick worker duration. |

### 10.2 Architectural Optimizations for Sub-500ms TTFBird
1. **Zero External Media Requests**: No MP3/WAV audio downloads; no large raster sprite sheets. Birds and foliage are constructed via compact vector paths and procedural shaders.
2. **Edge-Injected Snapshot**: The root HTML payload served by the CDN edge dynamically embeds the current aviary snapshot inside a `<script id="initial-state">` tag, eliminating a secondary client-to-server roundtrip.
3. **Aggressive Code-Splitting**: Account settings, accessibility management modals, visit management, and export UI are loaded asynchronously on-demand.

### 10.3 Privacy-Preserving Observability
The telemetry pipeline enforces strict privacy invariants:
- **Allowed Aggregate Metrics**:
  - Edge request rates, HTTP status distribution, and edge cache hit ratios.
  - Core Web Vitals (LCP, INP, CLS) and custom `time-to-first-bird` histograms.
  - RUM FPS percentiles and dropped frame percentages.
  - WebAudio initialization success vs failure counts.
  - Simulation tick duration percentiles ($p50, p95, p99$) and worker queue depths.
- **Strictly Prohibited Telemetry**:
  - No per-account or per-bird personality vector values.
  - No user interaction counts, visit frequencies, or presence duration logs.
  - No bird names, offer choices, or notebook contents.
  - Simulation database is completely air-gapped from analytics warehouses.

---

## 11. Rollout, Aviary Growth & Species Pool

### 11.1 Species Pool Specifications (v1)
Six coherent biological species silhouettes with distinct behavioral and vocal traits:
1. **Pip (Warbler)**: Slender silhouette, high perch affinity, bright agile FM frequency sweeps ($2.2\text{ kHz} - 4.8\text{ kHz}$).
2. **Wren (Cactus/House Wren)**: Compact rounded body, cocked tail, lower raspy trills and rhythmic chirps ($1.5\text{ kHz} - 3.2\text{ kHz}$).
3. **Chickadee**: Small, distinct cap, vocal motif includes multi-note whistling and fast fee-bee syllables.
4. **Nuthatch**: Low-creeping posture, nasal single-pitch repetitions, tends toward middle and back perches.
5. **Finch**: Conical bill, undulating call motifs, frequent social chorus participant.
6. **Nightjar**: Nocturnal silhouette, remains settled during daylight, active during evening and night with soft continuous purrs and churring.

### 11.2 Aviary Growth Schedule (Paced Strictly by Aviary Age)
New bird arrivals are governed entirely by calendar age since aviary creation. There are no interaction requirements, attendance streaks, or adoption fees:
- **Birds 1 & 2 (Starter Pair)**: Arrive at Day 0 upon initial sign-up. System selects two distinct species from the pool; user names them.
- **Bird 3**: Arrives at **Day 60** (~2 months).
- **Bird 4**: Arrives at **Day 150** (~5 months).
- **Bird 5**: Arrives at **Day 270** (~9 months).
- **Bird 6**: Arrives at **Day 365** (1 year).
- **Bird 7 (Hard Ceiling)**: Arrives at **Day 500** (~16 months).

*Empirical Justification for the 7-Bird Cap*: Procedural audio recognizability breaks down above seven concurrent callers; choruses degenerate into generic ambient noise. Restricting the population to seven preserves intimate, individual bird recognition by ear.

### 11.3 Rollout Milestones
- **Milestone 1: Simulation & Audio Test Harness (Weeks 1–4)**:
  - Implement tick loop, drift filter math, and WebAudio procedural syrinx engine.
  - Validate with a synthetic 100x time-warp test suite verifying drift calibration over 12 virtual weeks.
- **Milestone 2: Client Canvas & Interaction Core (Weeks 5–8)**:
  - Build single horizontal scene, responsive viewport scaling, and procedural micro-motion.
  - Implement presence accounting conjunction and listen-in audio decay curves.
- **Milestone 3: Auth, Sync & Notebook (Weeks 9–12)**:
  - Ship magic-link auth, synthetic UUID architecture, multi-device snapshot sync, and naturalist notebook generator.
- **Milestone 4: Accessibility, Social & Security Audit (Weeks 13–16)**:
  - Complete screen-reader prose live regions, reduced-motion cross-fades, visit link generation, and contrast verification.
- **Milestone 5: Staged Production Launch (Weeks 17–20)**:
  - *Internal Canary (100 accounts)*: 2 weeks of real-world continuous presence validation.
  - *Private Beta (1,000 accounts)*: Verification of tick worker horizontal auto-scaling and multi-device sync integrity.
  - *General Availability*: Public access with strict rate-limiting and operational monitoring.

---

## 12. Technical Risk Matrix & Mitigation Strategies

| Risk Category | Potential Failure Mode | Technical Impact | Mitigation & Engineering Controls |
| :--- | :--- | :--- | :--- |
| **Drift Calibration** | Drift rate too aggressive or too sluggish. | Birds change overnight (feels like a clicker game) or appear static over months (screensaver feel). | Comprehensive CI simulation suite running $10^5$ virtual days across 50 usage archetypes. Automated assert: $\Delta \text{trait} \in [0.03, 0.05]$ at 7 days; $[0.10, 0.15]$ at 21 days. Monotonic clamping strictly enforced. |
| **Sync Race Conditions** | Multi-device concurrent sessions causing ghost drift or data corruption. | Divergent aviary states; lost drift history across laptop and mobile transitions. | Single-authoritative server tick architecture. Client event streaming into an append-only log with monotonic server sequencing. Clients never submit absolute trait states. |
| **Audio Fatigue & Uncanniness** | Procedural calls sound harsh, robotic, or grating over long sessions. | Users mute audio, breaking the primary affective relationship. | Syrinx physical modeling incorporating non-linear modulation, randomized micro-jitter in pitch ($ \pm 1.2\% $), and breath noise. Strict master compression preventing harsh harmonic accumulation. Graceful silent mode with call captions. |
| **Presence Spoofing & Background Leaks**| Tab left open in background window inflating presence metrics. | Unintended rapid personality drift without actual user attention. | Tripartite presence verification: `document.visibilityState === 'visible'` $\land$ `document.hasFocus()` $\land$ active input within 180 seconds. Inactive tabs stop rendering and emit zero presence pings. |
| **Accessibility Regression** | Naturalist narration degraded into mechanical ARIA attribute spam. | Screen-reader users excluded from the affective aliveness of the aviary. | Hard lint and integration test rules forbidding state-label dumps in `#aviary-live-narration`. Automated validation requiring valid naturalist prose templates. |
| **PII Contamination** | Email addresses leaking into telemetry or partition logs. | Privacy breach violating core architectural commitments. | Synthetic UUIDv4 primary keys used everywhere. Email encrypted via AES-256-GCM in a single table. Telemetry completely decoupled and schema-isolated from user databases. |

---

## 13. Verification Checklist for Execution Teams

Before declaring v1 ready for deployment, engineering teams must verify:
- [ ] Initial gzipped JavaScript bundle is $< 2.0\text{ MB}$.
- [ ] Time-to-first-bird is $< 500\text{ ms}$ on 4G network profile.
- [ ] Zero MP3, WAV, or OGG files exist in the repository or CDN.
- [ ] Personality vector numbers are completely unexposed in client payloads, DOM attributes, and network tabs.
- [ ] Drift function is demonstrably monotonic ($\Delta \text{trait} \ge 0.0$ under all test cases).
- [ ] Closing tab and clicking Settle produce equivalent presence-accounting terminations.
- [ ] Screen-reader live region speaks evocative naturalist prose, not state changes.
- [ ] Reduced-motion mode renders soft pose cross-fades without particle drift.
- [ ] Visit invitation links expire in 30 days and are instantly revocable from account settings.
- [ ] Absolutely no streak counters, gamification widgets, or announcement toasts exist anywhere in the application.
