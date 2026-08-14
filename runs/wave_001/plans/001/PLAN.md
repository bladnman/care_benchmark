# Pocket Aviary — Comprehensive Engineering Implementation Plan (v1)

## 1. Executive Summary & Product Scope

### 1.1 Product Mission & Vision
Pocket Aviary is an ambient, browser-based virtual aviary designed around low-key, observational relationships with procedural virtual birds. The core premise is that **idle attention is real interaction**: birds notice the user's presence, their long-term personalities drift across weeks of presence, and their short-term moods reflect daily rhythms and subtle session events.

### 1.2 Core Scope Boundaries (v1)
- **Bird Capacity**: Starts strictly with 2 starter birds upon adoption; unlocks up to a hard cap of 7 birds strictly modulated by aviary age (time elapsed since creation).
- **Platform**: Web-only (modern evergreen browsers: Chrome, Safari, Firefox, Edge). No native apps (iOS/Android) or wrappers.
- **Authentication**: Single-user accounts via passwordless email magic links (15-minute expiration, immediate invalidation on use).
- **Scene**: Single horizontal responsive 2D scene with 3 perch zones (front, middle, back), continuous local-time day/night lighting cycle, and subtle procedural weather. No panning, zooming, or multi-room layout.
- **Core Interactions**:
  1. *Presence*: Measured attention (conjunction of tab visibility, window focus, and recent pointer/key activity).
  2. *Return-Greeting*: Procedurally varied first-second greeting from one bird keyed by boldness, mood, and absence duration.
  3. *Listen-in*: Audio focusing on a single bird with smooth exponential gain ramps.
  4. *Offer*: Gestural gift (seed, song fragment, still pool) with a per-bird cooldown (~3 minutes) affecting short-term mood and curiosity/boldness.
  5. *Settle*: User-initiated soft session-end shifting scene to evening and quieting audio (with 5-second undo affordance).
  6. *Field Notebook*: Auto-generated, sparse, read-only naturalist observation log.
- **Social (Optional & Quiet)**: One-to-one email invitations for read-only, non-co-present ambient observation. No presence or drift recorded from visitors. Revocable at will.
- **Accessibility**: Screen-reader naturalist running narration (30–60s cadence), real-time call captions, dedicated reduced-motion cross-fade mode (not just disabled animations), keyboard navigation, and WCAG AA contrast.

### 1.3 Strict Non-Goals & Anti-Patterns Enforcement
1. **No Gamification**: Absolute prohibition on streaks, scores, levels, badges, XP, adoption counters, visit calendars, and green-dot graphs.
2. **No Tamagotchi Mechanics**: Birds never die, starve, fall ill, or show distress. Neglect results in ambient quietness rather than negative drift or punishment.
3. **No Social Network Surfaces**: No public directories, discovery feeds, profiles, follows, shared aviaries, co-presence cursors/avatars, comments, or leaderboards.
4. **No Push Notifications**: No external pings, reminders, web push, or email digests urging visits.
5. **No Announcement UI**: No "Welcome back!" toasts, achievement modals, or arrival banners.
6. **No Client-Authoritative Drift / No Last-Write-Wins**: Clients never write personality state directly or submit absolute numerical traits.

### 1.4 Voice and Tone Protocol
- **Naturalist Register** (*Product Surfaces: Aviary, Notebook, Narration, Captions, Offer Prompts*): Lowercase by default, present-tense, bird-named, specific, observational, devoid of exclamation marks, gamified terminology, or second-person demands.
- **Matter-of-Fact Register** (*System Surfaces: Magic link auth, session timeout, sync conflict, account settings, accessibility toggles, error states*): Standard English capitalization, clear, concise, direct, helpful, and completely devoid of faux warmth or naturalist disguise.

---

## 2. System Architecture & Component Boundaries

```
                       +---------------------------------------+
                       |           Browser Client              |
                       |  - Procedural WebAudio Synth Engine   |
                       |  - HTML5 Canvas/SVG Render Pipeline   |
                       |  - Presence & Event Tracker           |
                       |  - Accessibility & Caption Manager    |
                       +---------------------------------------+
                                  |                 ^
       Interaction Events (HTTPS) |                 | Snapshots (SSE / HTTPS)
                                  v                 |
                       +---------------------------------------+
                       |             Edge / API Gateway        |
                       |  - Magic Link Auth & Session Verifier |
                       |  - Rate Limiting & Request Routing    |
                       |  - Synthetic UUID Context Mapping     |
                       +---------------------------------------+
                                  |                 |
                   +--------------+                 +---------------+
                   | Event Ingest Stream                            | Read State
                   v                                                v
+---------------------------------------+         +-----------------------------------+
|      Append-Only Event Store          |         |    Aviary State Read Store        |
|  (Partitioned by Synthetic AccountID) |         |     (Fast Snapshot Cache)         |
+---------------------------------------+         +-----------------------------------+
                   |                                                ^
                   | Consumes Logs                                  | Writes Canonical
                   v                                                | Snapshots
+-------------------------------------------------------------------+-----------------+
|                         Simulation Engine Worker Service                            |
|  - Server-Side Tick (60s loop per active/scheduled aviary)                          |
|  - Drift Low-Pass Filter Processor (Additive Deltas)                                |
|  - Mood State Transition Engine & Weather Simulator                                 |
|  - Field Notebook Heuristics Generator                                              |
|  - Account & Aviary Lifecycle Manager                                               |
+-------------------------------------------------------------------------------------+
                   |                                                |
                   v                                                v
+---------------------------------------+         +-----------------------------------+
|      PostgreSQL Primary Relational    |         |    Operational Telemetry Sink     |
|   (Accounts, Birds, Vectors, Log)     |         | (Aggregates Only - Strict Privacy)|
+---------------------------------------+         +-----------------------------------+
```

### 2.1 Service Breakdown
1. **Edge API Gateway (Node.js/Go)**:
   - Terminates TLS, verifies session tokens (JWT/HMAC keyed against Redis revocation list).
   - Validates synthetic account UUIDs.
   - Serves initial static bundle, static SSR snapshot for `<500ms` first frame, and handles REST/SSE endpoints.
2. **Simulation Engine Worker Service (Go / Rust / Node.js Cluster)**:
   - Executes the server-side simulation tick every 60 seconds per aviary.
   - Processes batches of unconsumed interaction events from the append-only log.
   - Computes low-pass filter personality drift, updates mood state machines, and writes updated snapshots to PostgreSQL and Redis.
   - Generates naturalist field notebook entries upon notable heuristic triggers.
3. **Primary Relational Store (PostgreSQL)**:
   - Stores normalized relational data: accounts, birds, personality vectors, current moods, field notebook entries, and visit invitations.
4. **Append-Only Event Log (PostgreSQL Partitioned Tables / Kafka / Redis Streams)**:
   - Ingests raw interaction and presence events from clients. Serves as the sole input stream for simulation tick delta calculations.
5. **Operational Telemetry Service**:
   - Collects aggregate-only metrics (request latencies, tick runtimes, render FPS distributions, audio errors). Strictly isolated from any account ID or bird state.

---

## 3. Data Model & Storage Schema

```sql
-- Schema Definition: Pocket Aviary Core

-- 1. Accounts Table
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(), -- Synthetic Account ID
    encrypted_email BYTEA NOT NULL,                -- PII Encrypted at rest (AES-GCM-256)
    email_hash VARCHAR(64) NOT NULL UNIQUE,       -- Blind index (HMAC-SHA256) for lookups
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    deletion_requested_at TIMESTAMPTZ NULL,        -- Soft deletion timestamp
    settings JSONB NOT NULL DEFAULT '{
        "accessibility": {
            "captions_enabled": false,
            "reduced_motion": false
        },
        "notifications": {
            "visit_notifications_enabled": false
        }
    }'::jsonb
);

-- 2. Auth Tokens & Magic Links
CREATE TABLE magic_links (
    token_hash VARCHAR(64) PRIMARY KEY,
    account_id UUID REFERENCES accounts(id) ON DELETE CASCADE,
    email VARCHAR(255) NOT NULL,
    expires_at TIMESTAMPTZ NOT NULL,
    consumed_at TIMESTAMPTZ NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE active_sessions (
    session_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    device_name VARCHAR(128) NOT NULL,
    user_agent TEXT NOT NULL,
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- 3. Aviaries Table
CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    settled_at TIMESTAMPTZ NULL,                  -- Non-null if aviary is currently settled
    weather_state JSONB NOT NULL DEFAULT '{
        "type": "clear",
        "intensity": 0.0,
        "ends_at": null
    }'::jsonb,
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    version BIGINT NOT NULL DEFAULT 1              -- Monotonic snapshot version
);

-- 4. Birds Table
CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(), -- Stable Bird Identifier
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL,              -- e.g., 'song_sparrow', 'mourning_dove'
    name VARCHAR(48) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    perch_zone VARCHAR(16) NOT NULL DEFAULT 'middle', -- 'front', 'middle', 'back'
    
    -- Hidden Personality Vector (Scalars normalized to [0.0, 1.0])
    personality_boldness FLOAT8 NOT NULL DEFAULT 0.3,
    personality_social_warmth FLOAT8 NOT NULL DEFAULT 0.3,
    personality_vocal_frequency FLOAT8 NOT NULL DEFAULT 0.3,
    personality_plumage_saturation FLOAT8 NOT NULL DEFAULT 0.3,
    personality_curiosity FLOAT8 NOT NULL DEFAULT 0.3,
    
    -- Short-Term Fast-Timescale Mood
    mood_state VARCHAR(24) NOT NULL DEFAULT 'content', -- 'wary', 'content', 'curious', 'drowsy', 'alert'
    mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_offer_accepted_at TIMESTAMPTZ NULL,
    
    CONSTRAINT bird_count_limit CHECK (perch_zone IN ('front', 'middle', 'back'))
);

-- 5. Append-Only Interaction Event Log
CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(id) ON DELETE CASCADE,
    event_type VARCHAR(32) NOT NULL, -- 'presence_ping', 'listen_in_start', 'listen_in_end', 'offer', 'settle', 'unsettle'
    payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    consumed_by_tick BOOLEAN NOT NULL DEFAULT FALSE
);
CREATE INDEX idx_events_unconsumed ON interaction_events (aviary_id, id) WHERE consumed_by_tick = FALSE;

-- 6. Field Notebook Entries
CREATE TABLE field_notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(id) ON DELETE SET NULL,
    observation_text TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_notebook_aviary ON field_notebook_entries (aviary_id, created_at DESC);

-- 7. Visit Invitations (Quiet Social)
CREATE TABLE visit_invites (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    invitee_email_hash VARCHAR(64) NOT NULL,
    invite_token VARCHAR(64) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL DEFAULT NOW() + INTERVAL '30 days',
    revoked_at TIMESTAMPTZ NULL
);

-- 8. Visit Log (Host Visible)
CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    visitor_email_masked VARCHAR(255) NOT NULL, -- e.g., 'f****d@example.com'
    started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    duration_seconds INT NOT NULL DEFAULT 0
);
```

---

## 4. API Surface & Wire Protocols

### 4.1 Authentication & Account Lifecycle Endpoints
- `POST /api/v1/auth/magic-link`: Submits email; rate-limited to 5 requests / 15 min per IP/email. Sends 15-minute one-time link.
- `GET /api/v1/auth/verify?token=...`: Validates link, invalidates token, issues `HttpOnly`, `Secure`, `SameSite=Lax` session cookie.
- `POST /api/v1/auth/logout`: Revokes current session ID.
- `GET /api/v1/account/sessions`: Lists active sessions with device names and last active timestamps.
- `DELETE /api/v1/account/sessions/:id`: Revokes specific device session.
- `POST /api/v1/account/export`: Generates JSON aviary export snapshot and emails download link.
- `DELETE /api/v1/account`: Initiates 30-day soft-deletion grace period.
- `POST /api/v1/account/restore`: Cancels soft deletion if within 30-day window.

### 4.2 Aviary State & Stream
- `GET /api/v1/aviary/state`: Fetches current canonical snapshot.
  ```json
  {
    "aviary_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
    "version": 14208,
    "server_time": "2026-08-13T18:07:00Z",
    "local_time_offset": -25200,
    "settled": false,
    "weather": { "type": "clear", "intensity": 0.0 },
    "birds": [
      {
        "id": "e4b2d56a-1234-4567-89ab-cdef01234567",
        "species_id": "song_sparrow",
        "name": "Pip",
        "perch_zone": "front",
        "mood": "curious",
        "plumage_saturation": 0.42,
        "vocal_frequency": 0.58,
        "call_motif_seed": 94821
      },
      {
        "id": "f5c3e67b-5678-4321-ba98-fedcba987654",
        "species_id": "mourning_dove",
        "name": "Wren",
        "perch_zone": "back",
        "mood": "content",
        "plumage_saturation": 0.35,
        "vocal_frequency": 0.31,
        "call_motif_seed": 33102
      }
    ]
  }
  ```
- `GET /api/v1/aviary/stream`: Server-Sent Events (SSE) channel pushing state delta updates and notebook additions every minute or upon notable state mutation.

### 4.3 Client Interaction Ingestion (Append-Only)
- `POST /api/v1/aviary/events`: Appends interaction batch to server-side event log.
  - Body:
    ```json
    {
      "events": [
        {
          "event_type": "presence_ping",
          "timestamp": "2026-08-13T18:06:30Z",
          "payload": { "duration_seconds": 30 }
        },
        {
          "event_type": "offer",
          "bird_id": "e4b2d56a-1234-4567-89ab-cdef01234567",
          "timestamp": "2026-08-13T18:06:45Z",
          "payload": { "offer_type": "seed" }
        }
      ]
    }
    ```

### 4.4 Visit (Social Optional) Endpoints
- `POST /api/v1/visits/invite`: Host creates an invite link for `invitee_email`.
- `GET /api/v1/visits/list`: Host retrieves outstanding invites and the historical visit log.
- `DELETE /api/v1/visits/invite/:id`: Host revokes an active/outstanding invite.
- `GET /api/v1/visits/view/:token`: Visitor access endpoint. Authenticates token; streams read-only aviary snapshot. Rejects all interaction event submissions.

---

## 5. Simulation Engine & Mathematical Models

### 5.1 The 60-Second Server-Side Simulation Tick
The simulation tick runs on an independent worker cluster every 60 seconds per aviary, irrespective of client connection status.

```
+-------------------------------------------------------------+
|                      60-Second Tick Flow                    |
|                                                             |
| 1. Fetch unconsumed events from interaction_events          |
| 2. Aggregate valid Presence Time (T_p) & Interaction Counts |
| 3. Apply Asymmetric Low-Pass Filter Drift (ΔP)              |
| 4. Evaluate Mood Transitions (Markov / Event State Machine) |
| 5. Advance Day/Night & Weather Simulation                   |
| 6. Check Aviary Age for New Bird Adoption Eligibility       |
| 7. Evaluate Field Notebook Heuristics (Sparse Observation)  |
| 8. Commit State & Broadcast Snapshot via SSE                |
+-------------------------------------------------------------+
```

### 5.2 Presence Verification & Calibration
To prevent corrupted drift signals from background tabs or unattended machines, client presence is computed strictly on the conjunction:
$$\text{Presence} = \text{document.visibilityState} == \text{'visible'} \land \text{window.hasDocumentFocus} \land (\Delta t_{\text{last\_activity}} \le 180\text{s})$$
- Pings are batched every 30 seconds by the client.
- The server validates and clamps presence duration to a maximum of 60 seconds per tick cycle.

### 5.3 Personality Vector Drift Mathematics
Drift is modeled as a continuous low-pass filter with exponential approach toward maximum expressiveness ($P_{\max} = 1.0$), with absolute **monotonic asymmetry**:
$$P_i(t + \Delta t) = P_i(t) + \Delta P_i$$
Where the delta for trait $i$ is defined by:
$$\Delta P_i = \alpha_i \cdot (1.0 - P_i(t)) \cdot \left[ w_{p,i} \cdot \frac{T_p}{3600} + \sum_{k} w_{k,i} \cdot I_k \right]$$
- $T_p$: Presence time in seconds during the tick window.
- $I_k$: Interaction events (listen-in, offer accepted).
- $\alpha_i$: Trait-specific drift speed constant.
- $w_{p,i}, w_{k,i}$: Calibrated input weights.
- **Monotonicity Rule**: $\Delta P_i \ge 0$. If $T_p = 0$ and $I_k = 0$, $\Delta P_i = 0$. Traits **never decay** due to absence or neglect.

#### Drift Calibration Targets
- **1 Week of Regular Visits (30 min/day)**: $\Delta P \approx +0.03$ to $+0.05$ (measurable in backend instrumentation, subtle in runtime variance).
- **3 Weeks of Regular Visits**: $\Delta P \approx +0.12$ to $+0.18$ (visibly and audibly noticeable in bird approach proximity, greeting precedence, and plumage richness).
- **2-Week Extended Absence**: $\Delta P = 0.00$. Birds return with their exact previously accumulated personality traits intact.

```
Trait Drift Response Curve (3 Weeks Regular Visits vs 2 Weeks Away)
Trait Value
1.0 |                                    (Asymptotic Ceiling)
0.8 |
0.6 |                          /------------------- (Flat during absence)
0.4 |              /----------/                      (Monotonic, never drops)
0.2 |  -----------/
0.0 +------------------------------------------------------------------> Time (Weeks)
       Wk 0      Wk 1       Wk 2       Wk 3       Wk 4       Wk 5
```

### 5.4 Fast-Timescale Mood State Machine
Mood operates on a fast-timescale discrete Markov process modulated by time of day, weather, and recent interaction events.

```
               +-------------+  Morning Light / Alert Call
               |    ALERT    |<-----------------------------+
               +------+------+                              |
                      |                                     |
    Quiet Morning /   |   Offer Accepted /                  |
    Gentle Perch      |   Listen-in                         |
                      v                                     |
               +-------------+  Offer Presented             |
     +-------->|   CONTENT   |------------------>+          |
     |         +------+------+                   |          |
     |                |                          v          |
     |                | Dusk / Quiet       +-------------+  |
     |                v                    |   CURIOUS   |  |
     |         +-------------+             +------+------+  |
     |         |   DROWSY    |                    |         |
     |         +------+------+                    |         |
     |                |                           | Alarm   |
     |                | Sudden Noise / Alarm      | Call    |
     |                v                           v         |
     |         +-------------+                              |
     +---------|    WARY     |<-----------------------------+
    Calm 15m   +-------------+
```

1. **Wary**: Triggered by sudden absence return after long time or ambient alarm calls. Bird retreats to `back` perch zone, reduces vocalization rate, increases scanning micro-motion.
2. **Content**: Default resting equilibrium. Bird preens on `middle` or `front` perch, produces melodic motifs.
3. **Curious**: Triggered by offers and listen-in focus. Bird perches forward, tilts head toward sound origin.
4. **Drowsy**: Triggered as local time approaches dusk/night. Bird fluffs plumage, reduces eye blink cadence.
5. **Alert**: Triggered at early morning sunrise or soft wind.

### 5.5 Procedural Call-Grammar Runtime
Each bird's species possesses a motif grammar library comprising:
- Fundamental frequency envelope ($f_0 \in [1.2\text{ kHz}, 4.5\text{ kHz}]$).
- Glissando curves (linear, exponential rise, frequency modulated trill).
- Timbre harmonics and formant filter profiles.
- Inter-syllable duration and jitter shaped by `vocal_frequency` and `curiosity`.

### 5.6 Field Notebook Generator Heuristics
Notebook observations are generated sparsely (target: ~1 entry per 2–4 days for active aviaries) using naturalist template combinators evaluated on notable simulation milestones:
- *Greeting Precedence*: When a bird with recently increased `social_warmth` greets before the historical first greeter for the first time.
- *Perch Zone Transition*: When a previously wary bird spends >15 minutes on the `front` rail.
- *Weather Interactions*: When birds remain calm during a passing drizzle.
- *Drowsy Synchrony*: When two adjacent birds settle simultaneously at twilight.

---

## 6. Sync Model & Conflict Avoidance

```
Device A (Laptop)                 Server Canonical State              Device B (Phone)
      |                                     |                                |
      |--- POST /events (Presence 30s) ---->|                                |
      |                                     | (Tick computes ΔP)             |
      |                                     |--- SSE Push (New Snapshot) --->|
      |                                     |                                |
      |                                     |<-- POST /events (Offer) -------|
      |                                     | (Tick transitions Mood)        |
      |<-- SSE Push (New Snapshot) ---------|                                |
```

### 6.1 Strict Single-Writer Architecture
- **Server is the Sole Canonical Writer**: Clients are pure projection engines that submit intent and sensor data (events). Clients **never write state**.
- **No Last-Write-Wins (LWW) on Personality or Mood**: Conflicting concurrent sessions (e.g., laptop active at work while phone is open in pocket) append events to the same unified account stream. The server processes these events in timestamp order.
- **Deduplication**: Client events carry idempotency UUIDs. Replayed requests from flakey mobile networks are deduplicated at ingestion.

### 6.2 Client Lifecycle & Reconnection State Machine
1. **Visibility Change (`document.visibilityState`)**:
   - `hidden` $\to$ `visible`: Client immediately halts rendering sleep, triggers a fresh HTTP snapshot fetch, resynchronizes scene state, and opens SSE stream.
2. **Connection Loss & Resume**:
   - Client detects network drop via SSE heartbeat timeout (30s).
   - Enters reconnecting state with exponential backoff (1s, 2s, 4s, max 10s).
   - Upon reconnect, requests full snapshot with `If-None-Match: "v{version}"`. If version is unchanged, resumes stream without visual disruption.
3. **Tab Sleep / Laptop Lid Close**:
   - Render clock stops. Upon wake, client detects time jump ($\Delta t > 5\text{s}$), skips internal simulation catch-up, and pulls fresh server snapshot.

---

## 7. Frontend Rendering Pipeline

```
+-------------------------------------------------------------------+
|                        HTML5 Canvas Scene                         |
|                                                                   |
| [Layer 0: Sky & Atmospheric Gradient (Local-Time Color Shader)]    |
| [Layer 1: Background Silhouette & Distant Foliage (Parallax 0.2)] |
| [Layer 2: Back Perch Zone (Distant Birds, Scale 0.75)]            |
| [Layer 3: Middle Perch Zone (Primary Perches, Scale 0.90)]        |
| [Layer 4: Front Perch Rail (Foreground Birds, Scale 1.05)]        |
| [Layer 5: Ambient Particles (Drifting Leaves / Feathers)]         |
| [Layer 6: Weather Composite (Soft Rain / Wind Shimmer)]           |
+-------------------------------------------------------------------+
| Top-Bar DOM Chrome (Fade to opacity 0 on 5s cursor inactivity)    |
+-------------------------------------------------------------------+
```

### 7.1 Scene Composition & Three Perch Zones
- **Canvas / WebGL Engine**: Ultra-lightweight 2D Canvas rendering context with zero external heavy game engine dependencies.
- **Three Perch Zones**:
  - `Back Perch`: Z-index 2, scale 0.75, opacity 0.85, desaturated by 15%.
  - `Middle Perch`: Z-index 3, scale 0.90, natural lighting.
  - `Front Perch`: Z-index 4, scale 1.05, rich feather detail, foreground focus.
- **Smooth Interpolation**: When a bird changes perch zone on snapshot update, it executes a procedural parabolic flight arc (300–600ms) with wing flapping micro-cycles.

### 7.2 Idle Micro-Motion Generators
Every bird executes continuous procedural idle motion using sine/cosine oscillators and Perlin noise:
1. **Breathing Cycle**: Subtle Y-scale modulation (0.5–1.2 Hz) based on mood (`drowsy` is slower, `alert` is rapid).
2. **Head Tilting & Scanning**: Randomized angular saccades every 3–8 seconds toward ambient sound sources or other calling birds.
3. **Preening**: Periodic feather shuffle animation triggered when `mood == 'content'`.
4. **Tail Wag & Foot Reset**: Small weight adjustments to prevent visual stagnation.

### 7.3 Day/Night & Atmospheric Shaders
Lighting follows a continuous trigonometric curve based on the user's local solar time:
- **Dawn (05:30–07:30)**: Soft warm ochre and rose hues (`#F5E6D3`, `#E8C5B0`).
- **Midday (11:00–15:00)**: Crisp, bright natural daylight (`#FFFFFF`, `#EAF4F8`).
- **Dusk / Settled (18:30–20:30)**: Deep amber, indigo, and warm brown hues (`#D9822B`, `#4A3B32`).
- **Night (21:00–05:00)**: Muted midnight blue (`#0D1B2A`, `#1B263B`). Nightjar species remains awake with luminous eye glints; other birds adopt sleeping tucked-head posture.

### 7.4 Reduced-Motion Cross-Fade Mode
- Activated via system `prefers-reduced-motion` or accessibility settings toggle.
- **Mechanics**:
  - Disables frame-by-frame limb/wing skeletal animations.
  - Replaces perch-to-perch flight paths with slow, gentle alpha cross-fades (800ms duration).
  - Idle states cross-fade between static illustrated poses (preen, glance, rest) every 10–15 seconds.
  - Eliminates drifting leaf and feather particle layers.

### 7.5 Loading Sequence & Empty-State Handling
- **No Spinners or Loading Bars**: The initial load displays a tranquil atmospheric sky gradient with soft ambient wind audio immediately.
- **Initial Frame Mid-Action**: As soon as snapshot resolves (<500ms), birds are instantiated directly into their ongoing micro-motion phase without fade-in or "pop" entry animations.
- **Empty Aviary (Post-Signup)**: Shows the quiet scenic landscape; starter birds arrive via an organic initial flight onto their perches.

---

## 8. Audio Pipeline & Procedural Soundscape

```
                          WebAudio Graph Architecture

+----------------------+
| Bird 1 Synth (Voice) |---> [PannerNode] ---> [GainNode (Listen-In)] ---+
+----------------------+                                                 |
                                                                         v
+----------------------+                                            +----------+
| Bird 2 Synth (Voice) |---> [PannerNode] ---> [GainNode (Listen-In)] --->|  Chorus  |
+----------------------+                                                 |  Mix Bus |
                                                                         +----+-----+
+----------------------+                                                      |
| Ambient Rain / Wind  |----------------------------------------------------->|
+----------------------+                                                      v
                                                                        +-------------+
                                                                        | Master Gain |
                                                                        +------+------+
                                                                               v
                                                                        [AudioContext]
```

### 8.1 Procedural WebAudio Synth Architecture
To adhere strictly to the `<2MB` bundle budget and eliminate canned audio loops, all vocalizations are synthesized in real-time via WebAudio primitives:
- **Oscillator Types**: High-precision `CustomWaveTable` / modulated `SineOscillator` with exponential frequency ramps (`exponentialRampToValueAtTime`).
- **Harmonics & Formants**: `BiquadFilterNode` configured as a bandpass filter with resonant peak to simulate avian syrinx acoustics.
- **Spatial Positioning**: `StereoPannerNode` panning calls to match the bird's horizontal canvas coordinate ($X \in [-0.8, +0.8]$).

### 8.2 Dynamic Chorus Mixing & Natural Staggering
- **Anti-Phase-Canceling**: When multiple birds vocalize concurrently, procedural pitch micro-jitter ($\pm 15\text{ cents}$) and randomized attack offsets (80–250ms) prevent phase cancellation and synthetic unison artifacts.
- **Chorus Density Limiting**: Maximum 2 concurrent foreground calls; secondary calling birds dynamically soften their volume and yield to dominant callers.

### 8.3 Listen-In Focus Dynamics
When a bird is focused:
1. Focused bird's gain node ramps up by $+6\text{ dB}$ over an exponential curve of $400\text{ms}$.
2. All non-focused birds ramp down by $-12\text{ dB}$ over $600\text{ms}$ (never fully muted, preserving ambient presence).
3. Background ambient atmospheric sound drops by $-6\text{ dB}$.
4. Disengaging smoothly returns all gain nodes to equalized baseline ($0\text{ dB}$) over $800\text{ms}$.

### 8.4 WebAudio Fallback
If the browser environment blocks `AudioContext` autoplay, lacks WebAudio support, or audio permissions are denied:
- The aviary enters graceful silent mode.
- Call captions are automatically enabled by default near active birds.
- **No recorded audio fallbacks** are loaded, preventing bundle bloat and maintaining auditory fidelity.

---

## 9. Accessibility Architecture

### 9.1 Screen-Reader Narration Engine
- An `aria-live="polite"` region continuously narrates the aviary scene in the naturalist field-notebook voice.
- **Cadence**: 30–60 seconds between periodic observations during idle viewing.
- **Immediate Observation Dispatch**: User actions (e.g., return greeting, offer reaction, settle) prioritize a descriptive narration update within 1.5 seconds.
- **Naturalist Prose Examples**:
  - *"wren is perched on the low branch, calling softly in the morning light."*
  - *"pip hops forward to the front rail, tilting her head toward the offered seed."*
  - *"the aviary is settling into dusk; calls have quieted across the perches."*

### 9.2 Real-Time Call Captions
- Positioned dynamically as non-obtrusive, accessible text bubbles floating gently adjacent to the calling bird.
- Prose description of procedural calls generated directly from the audio grammar motifs:
  - `motif_trill_high` $\to$ *"a clear, rising three-note trill"*
  - `motif_dove_coo` $\to$ *"a low, double coo from the back perch"*
  - `motif_chirp_short` $\to$ *"a quiet, single chip"*
- Captions fade in over 150ms with the audio attack and fade out 400ms after the audio release.

### 9.3 Keyboard Navigation & Focus Manager
- `Tab`: Moves focus seamlessly between Top-Bar controls and Aviary scene birds.
- `Left / Right Arrow`: Cycles focus between birds across the 3 perch zones.
- `Enter / Space`: Triggers *Listen-in* on the currently focused bird.
- `Escape`: Disengages *Listen-in*, closes open menus/notebooks.
- `Key O`: Opens Offer selection drawer.
- `Key S`: Triggers Settle gesture.
- **Visual Focus Outline**: High-contrast, 2px soft-rounded outline (`rgba(255, 255, 255, 0.85)` with `rgba(0, 0, 0, 0.5)` drop-shadow) ensuring WCAG AA contrast against both dawn and midnight scene palettes.

---

## 10. Performance Budgets, Optimization & Observability

### 10.1 Budgets Breakdown
| Metric | Budget Target | Implementation Strategy |
| :--- | :--- | :--- |
| **Initial JS Bundle** | `< 2.0 MB` (gzipped) | Zero game engine runtime; vanilla WebAudio synth; code-splitting settings/notebook. |
| **Time-to-First-Bird** | `< 500 ms` on 4G | Inline critical CSS/SVG; edge-rendered SSR snapshot; zero blocking asset waterfalls. |
| **Runtime Framerate** | `60 fps` sustained | 2D Canvas dirty-rect rendering; offscreen canvas buffering; RAF throttling on idle. |
| **Memory Leakage** | `0 MB` delta over 30m | Reusable WebAudio buffer pools; static object pools for particle drift; no closure leaks. |
| **Tick Latency p99** | `< 5.0 s` across fleet | Redis snapshot caching; partitioned batch processing for unconsumed event logs. |

### 10.2 Privacy-Preserving Observability Boundary
```
+-------------------------------------------------------------+
|             STRICT DATA ISOLATION BOUNDARY                  |
|                                                             |
| [ALLOWED Operational Telemetry]                             |
|  - Synthetic HTTP 5xx / 4xx error rates                     |
|  - Simulation Tick execution latency histograms             |
|  - Client FPS performance aggregates (25th, 50th, 95th)     |
|  - WebAudio initialization failure counts                   |
|                                                             |
| ===================== HARD BARRIER ======================== |
|                                                             |
| [STRICTLY FORBIDDEN in Telemetry / Analytics]               |
|  - Account IDs / Email hashes                               |
|  - Bird names, species, or personality vectors              |
|  - User presence durations or session frequency logs        |
|  - Field notebook contents or interaction event history     |
+-------------------------------------------------------------+
```

---

## 11. Rollout Strategy, Verification & Growth Ramp

### 11.1 Progressive Bird Unlock Algorithm
To maintain auditory recognizability and deep emotional attachment, aviaries unlock additional bird capacity strictly based on aviary age (not visit count or payment):
- **Day 0 (Adoption)**: 2 starter birds.
- **Month 2**: 3rd bird arrives.
- **Month 5**: 4th bird arrives.
- **Month 8**: 5th bird arrives.
- **Month 12**: 6th bird arrives.
- **Month 16+**: 7th bird arrives (Hard Cap reached).

### 11.2 Verification & Test Harnesses
1. **Simulation Drift Calibration Harness**:
   - Automated 10,000-tick headless runner simulating 30 days of synthetic presence profiles (daily 30m user, weekly 2h user, 2-week neglected user).
   - Validates that trait growth matches the target curve and strictly enforces zero negative drift on neglect.
2. **Audio Uncanny Valley & Psychoacoustic Audit**:
   - Automated WebAudio test verifying 7 concurrent procedural bird calls produce zero clipping, zero phase distortion, and maintain $>12\text{ dB}$ signal separation during listen-in.
3. **Memory & Long-Running Soak Suite**:
   - Headless Chrome Playwright test running 60-minute active aviary sessions verifying heap size stability ($\Delta \text{Heap} \le 1.5\text{MB}$ total GC fluctuation).
4. **Accessibility Compliance Test**:
   - Axe-core automated contrast and ARIA live-region audit across all 24 solar cycle lighting states.

---

## 12. Risk Analysis & Mitigation Matrix

| Risk Category | Potential Failure Mode | Technical Mitigation Strategy |
| :--- | :--- | :--- |
| **Drift Calibration** | Users exploit automated clicks or presence drift saturates within days. | Enforce 3-way conjunction presence verification; clamp presence accumulation to max 60s per tick; apply low-pass asymptotic scaling. |
| **Multi-Device Race** | Two devices submit conflicting actions simultaneously, overwriting state. | Append-only event store; simulation tick as sole canonical writer; additive deltas applied in chronological log order. |
| **Audio Uncanniness** | Procedural calls sound robotic, screechy, or repetitive. | Use complex multi-segment frequency modulation envelopes with calibrated micro-pitch jitter and randomized syllable motifs. |
| **Screen-Reader Flooding** | Rapid aviary updates spam screen-reader live regions, creating annoyance. | Throttle idle narration to 30–60s intervals; prioritize user actions with polite queuing and concise naturalist prose. |
| **PII / Privacy Leakage** | User emails inadvertently leak into telemetry or inter-service logs. | Synthetic UUID generation on account creation; email encrypted with AES-GCM; blind index for lookups; strict CI linting against email keys. |
