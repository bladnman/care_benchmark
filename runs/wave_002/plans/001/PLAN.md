# Pocket Aviary — Technical Implementation Plan (v1)

## 1. Scope & Non-Goals

### 1.1 In-Scope for v1
- **Platform**: Modern web browsers (Chrome, Safari, Firefox, Edge; last 2 major versions). No native apps.
- **Bird Population**: Starts with exactly 2 starter birds selected from a 6-species pool; expands over time strictly by aviary age up to a hard cap of 7 birds.
- **Core Interactions**:
  - Unannounced procedural return-greetings shaped by absence length, boldness, and current mood.
  - Idle presence tracking (three-factor conjunction: visible tab, window focus, recent user input).
  - Listen-in interaction (focusing one bird with gradual dynamic mix rebalancing).
  - Offer gestures (seed, song fragment, still pool) via top-bar affordance with per-bird cooldowns.
  - Settle gesture (opt-in soft evening lighting shift with a 5-second undo grace window).
  - Read-only Field Notebook generating sparse, naturalist prose observations.
- **Visuals & Layout**:
  - Single horizontal screen (no scrolling, zooming, or panning).
  - 3 distinct perch zones (front, middle, back) chosen autonomously by bird mood and personality.
  - Real-time day/night cycle anchored to the user's local timezone.
  - Ambient weather (soft rain, gentle wind) and continuous micro-motion (leaf/feather drift).
  - Autohiding top-bar chrome (fades after stillness, restores on interaction).
  - Initial load presents the aviary already mid-motion (quiet field fallback on slow networks).
- **Accounts, Sync & Privacy**:
  - Single-user accounts authenticated via email magic links (15-minute expiration, single-use, per-device revocable session tokens).
  - Server-side simulation tick (~1 minute cadence) as the single canonical author of personality vectors and mood.
  - Append-only client interaction event log; zero client-side personality mutations (no last-write-wins conflicts).
  - Synthetic UUIDs for all internal identities to guarantee zero PII leakage.
  - On-demand JSON snapshot export and 30-day soft deletion before hard purge.
- **Social (Optional & Quiet)**:
  - Host-initiated email invitations for read-only, ambient visitor access.
  - No co-presence, no visitor drift impact, no visitor interactions, instant revocation.
  - Silent visit logging in account settings; no visitor push notifications by default.
- **Accessibility & Performance**:
  - Screen-reader running narration in naturalist prose (updated every 30–60s or on user actions).
  - Dedicated reduced-motion mode (cross-fading still poses and perch transitions, no leaf drift).
  - Dynamic call captions generated from procedural motifs.
  - Full keyboard navigation with WCAG AA compliant contrast indicators.
  - Performance budgets: Initial JS bundle < 2MB (gzipped), time-to-first-bird < 500ms over 4G, 60fps idle motion, zero memory leak over 30+ minutes.
  - WebAudio procedural synthesis with silent + captioned fallback if WebAudio is unavailable.

### 1.2 Non-Goals (Explicit Exclusions)
- **No Native Apps**: No iOS, Android, or desktop wrappers.
- **No Gamification**: No streaks, levels, XP, points, badges, adoption counters, visit calendars, or milestone toasts.
- **No Tamagotchi Mechanics**: Birds never die, starve, decay, or exhibit distress. Drift is monotonic toward expressiveness; neglect creates ambient quietness, never hostility or sadness.
- **No Social Network Features**: No public directory, no discoverability, no comments, no chat, no visitor avatars, no leaderboards, no "show-off" rendering modes.
- **No Push/Notification Surfaces**: No unsolicited emails or push notifications about aviary status.
- **No UI Chrome in Scene**: No tooltips, tags, status bars, or drag-and-drop handles inside the aviary scene.

---

## 2. Architecture & System Topology

### 2.1 High-Level Architecture Diagram
```
                       +---------------------------------------+
                       |             Edge CDN Layer            |
                       | - HTML shell & SSR snapshot injection |
                       | - Static JS/CSS/SVG asset delivery    |
                       +-------------------+-------------------+
                                           |
                   HTTPS (JSON REST)       |      WSS / SSE Keepalive
                   +-----------------------+-----------------------+
                   |                                               |
                   v                                               v
        +--------------------+                           +--------------------+
        |   API Gateway /    |                           | Realtime Gateway   |
        |   Auth Service     |                           | (Snapshot Stream)  |
        | - Magic links      |                           | - Ephemeral state  |
        | - Session JWTs     |                           | - Push state diffs |
        | - Account mgmt     |                           +---------+----------+
        +----------+---------+                                     |
                   |                                               |
                   +-----------------------+-----------------------+
                                           |
                                           v
                       +---------------------------------------+
                       |           Core Backend App            |
                       | - Interaction Ingestion Log           |
                       | - Aviary State Snapshot Generator     |
                       | - Visit Invitation Validator          |
                       | - Field Notebook Prose Engine         |
                       +-------------------+-------------------+
                                           |
                   +-----------------------+-----------------------+
                   |                                               |
                   v                                               v
        +--------------------+                           +--------------------+
        |   PostgreSQL DB    |                           | Redis Cache / Queue|
        | - Accounts & UUIDs |                           | - Tick Lock        |
        | - Birds & Vectors  |                           | - Recent Events    |
        | - Notebook Entries |                           | - Magic Link Otps  |
        | - Visit Records    |                           +---------+----------+
        +--------------------+                                     |
                                                                   v
                                                         +--------------------+
                                                         | Simulation Worker  |
                                                         | (Async Tick Engine)|
                                                         | - Runs every 60s   |
                                                         | - Computes drift   |
                                                         | - Evaluates mood   |
                                                         | - Emits notebook   |
                                                         +--------------------+
```

### 2.2 Client/Server Split
- **Server Responsibility**:
  - Authoritative holder and sole writer of bird personality vectors and base mood timers.
  - Ingestion of append-only interaction events.
  - Background simulation tick execution (~60-second batch interval).
  - Generation of field notebook entries using naturalist prose templates.
  - Magic-link auth issuance and validation; visitor authorization and revocation enforcement.
- **Client Responsibility**:
  - Viewport-responsive Canvas/WebGL/SVG scene composition.
  - Procedural micro-motion generation (preening, head-tilting, weight shifting) parameterized by server-provided mood and personality traits.
  - Procedural call synthesis via WebAudio oscillators and filters.
  - Precise presence event detection (evaluating `document.visibilityState`, `document.hasFocus()`, and recent user pointer/key activity).
  - Accessible running narration queueing, live caption rendering, and keyboard focus routing.

---

## 3. Data Model

### 3.1 Database Schema (PostgreSQL DDL)

```sql
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    encrypted_email BYTEA NOT NULL,
    email_hash VARCHAR(64) NOT NULL UNIQUE, -- HMAC for lookup without decrypting
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    deleted_at TIMESTAMPTZ NULL, -- Soft deletion timestamp (purged after 30 days)
    settings JSONB NOT NULL DEFAULT '{"visit_notifications": false, "reduced_motion": false, "captions": false}'::jsonb
);

CREATE TABLE auth_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    device_label VARCHAR(128) NOT NULL,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at TIMESTAMPTZ NULL
);

CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    weather_state JSONB NOT NULL DEFAULT '{"current": "clear", "transition_at": null}'::jsonb
);

CREATE TYPE mood_type AS ENUM ('wary', 'content', 'curious', 'drowsy', 'alert');
CREATE TYPE perch_zone AS ENUM ('front', 'middle', 'back');

CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL, -- e.g., 'grey_warbler', 'spotted_towhee'
    name VARCHAR(64) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    -- Personality Vector: normalized scalar floats [0.0, 1.0]
    trait_boldness REAL NOT NULL DEFAULT 0.3,
    trait_social_warmth REAL NOT NULL DEFAULT 0.3,
    trait_vocal_frequency REAL NOT NULL DEFAULT 0.3,
    trait_plumage_saturation REAL NOT NULL DEFAULT 0.3,
    trait_curiosity REAL NOT NULL DEFAULT 0.3,
    
    -- Fast-Timescale Dynamic State
    current_mood mood_type NOT NULL DEFAULT 'content',
    current_perch perch_zone NOT NULL DEFAULT 'middle',
    mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_call_at TIMESTAMPTZ NULL,
    
    CONSTRAINT check_personality_ranges CHECK (
        trait_boldness BETWEEN 0.0 AND 1.0 AND
        trait_social_warmth BETWEEN 0.0 AND 1.0 AND
        trait_vocal_frequency BETWEEN 0.0 AND 1.0 AND
        trait_plumage_saturation BETWEEN 0.0 AND 1.0 AND
        trait_curiosity BETWEEN 0.0 AND 1.0
    )
);

CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(id) ON DELETE CASCADE,
    event_type VARCHAR(32) NOT NULL, -- 'presence_ping', 'listen_in_start', 'listen_in_end', 'offer_seed', 'offer_song', 'offer_pool', 'settle'
    duration_ms INT NULL,
    metadata JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE field_notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    entry_prose TEXT NOT NULL,
    noteworthy_event_type VARCHAR(64) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    invited_email_hash VARCHAR(64) NOT NULL,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '30 days'),
    revoked_at TIMESTAMPTZ NULL,
    last_visited_at TIMESTAMPTZ NULL,
    total_visit_duration_seconds INT NOT NULL DEFAULT 0
);
```

---

## 4. API Surface & Protocols

### 4.1 Authentication & Session Management
- `POST /api/v1/auth/magic-link/request`
  - Body: `{"email": "user@example.com"}`
  - Voice: Matter-of-fact response. Sends email with single-use token valid for 15 minutes.
- `POST /api/v1/auth/magic-link/verify`
  - Body: `{"token": "..."}`
  - Response: Sets HTTP-only secure session cookie, returns `{ account_id: UUID }`.
- `POST /api/v1/auth/session/revoke`
  - Body: `{"session_id": UUID}`

### 4.2 Aviary State & Interaction Ingestion
- `GET /api/v1/aviary/state`
  - Returns complete aviary state snapshot:
  ```json
  {
    "aviary_id": "8f0a202a-9f5e-4c12-8e0f-13d80bf940a1",
    "server_time": "2026-08-13T19:00:00Z",
    "local_time_offset": -25200,
    "weather": { "type": "clear", "intensity": 0.0 },
    "birds": [
      {
        "id": "e9b282ca-7c01-4475-bc8e-4a6730248c89",
        "species_id": "grey_warbler",
        "name": "Pip",
        "mood": "curious",
        "perch": "front",
        "visual_params": {
          "plumage_saturation": 0.45,
          "posture": "alert_tilt"
        },
        "audio_params": {
          "base_pitch_offset_cents": 12,
          "vocal_cadence_factor": 1.15
        },
        "last_interaction_delta_seconds": 120
      }
    ]
  }
  ```
  - *Note*: Raw scalar personality vectors (`trait_boldness`, etc.) are never exposed in this API response.
- `POST /api/v1/aviary/events`
  - Ingests batch of client events (appended directly to `interaction_events`):
  ```json
  {
    "events": [
      { "type": "presence_ping", "duration_ms": 30000, "client_ts": "2026-08-13T19:01:00Z" },
      { "type": "listen_in_start", "bird_id": "e9b282ca...", "client_ts": "2026-08-13T19:01:10Z" },
      { "type": "offer_seed", "bird_id": "e9b282ca...", "client_ts": "2026-08-13T19:02:00Z" }
    ]
  }
  ```
- `GET /api/v1/notebook`
  - Returns array of naturalist field notebook observations (paginated from newest to oldest).

### 4.3 Social (Visit) Flow
- `POST /api/v1/visits/invite`
  - Body: `{"visitor_email": "friend@example.com"}`
  - Generates token and dispatches read-only visit email link.
- `GET /api/v1/visits/state?token=...`
  - Validates token against expiration and revocation; returns read-only aviary state.
  - Rejects event submission endpoints for visitor sessions.
- `POST /api/v1/visits/revoke`
  - Body: `{"invitation_id": UUID}` (host-only).

---

## 5. Simulation Engine & Drift Runtime

### 5.1 Server-Side Simulation Tick Loop
1. **Cadence**: Executes every 60 seconds per active aviary or in batched cron workers.
2. **Event Consumption**:
   - Queries `interaction_events` where `created_at > last_tick_at`.
   - Aggregates validated presence minutes, listen-in durations, offers accepted, and settle triggers.
3. **Drift Function Calibration**:
   - Drift is mathematically modeled as an asymmetrical, monotonic leaky accumulator:
     $$\Delta \text{Trait}_i = \alpha_i \cdot \text{Signal}_{\text{presence}} + \beta_i \cdot \text{Signal}_{\text{interaction}}$$
     $$\text{Trait}_i(t + \Delta t) = \min(1.0, \text{Trait}_i(t) + \Delta \text{Trait}_i)$$
   - *Monotonicity Constraint*: $\Delta \text{Trait}_i \ge 0$. Neglect sets $\text{Signal} = 0$, producing zero downward drift. Traits never degrade.
   - *Calibration Rates*:
     - Presence attention (1 hour/day for 7 days) yields $\sim +0.03$ shift (measurable in tests).
     - 3 weeks of consistent presence yields $\sim +0.10$ to $+0.15$ shift (perceptible to user via perch proximity and feather richness).
4. **Mood Transition Markov Model**:
   - Transition probability matrix $P(M_{t+1} \mid M_t, \text{Inputs}, \text{Personality})$ evaluated every tick:
     - Dusk/night local time shifts weights heavily toward `drowsy`.
     - Recent accepted offer shifts weights toward `content` or `curious`.
     - Passing weather (rain) nudges toward `wary` or `drowsy`.
     - High `trait_boldness` dampens transitions into `wary`.
5. **Notebook Observation Generator**:
   - Evaluates rare aviary occurrences (e.g., Bird A greeted before Bird B for the first time; a bird stayed in `content` through rain).
   - Generates at most 1 entry every 2–4 days, formatted in lowercase, present-tense naturalist prose.

---

## 6. Sync Model & State Propagation

- **Single Writer Principle**: Only the backend simulation worker writes to `birds` and updates personality/mood. Clients submit interaction events only.
- **Snapshot Consumption**:
  - The client fetches the canonical snapshot on tab open, tab visibility return (`visibilitychange`), and every 60s while visible.
  - Client-side renderers use cubic Hermite spline interpolation for smooth perch transitions between snapshots, preventing teleportation.
- **Elimination of Last-Write-Wins**: Because clients never send trait updates, concurrent sessions across multiple devices (e.g., mobile phone and desktop) simply append their respective presence pings to the event log. The server tick processes events chronologically.

---

## 7. Frontend Rendering Pipeline & Scene Composition

### 7.1 Scene Composition
- **Rendering Target**: Canvas 2D or SVG scene with responsive aspect ratio preservation.
- **Planes**:
  1. *Background Plane*: Sky gradient matching local sun position; distant silhouettes; subtle parallax on mouse move.
  2. *Middle Ground Plane*: Primary perches (back, middle, front rails/branches); birds mid-action.
  3. *Foreground Plane*: Subtle foreground foliage, soft ambient leaves and feather drift.
- **First-Frame Aliveness**:
  - The render pipeline draws birds immediately in mid-pose based on state snapshot timestamps.
  - If initial network fetch exceeds 150ms, a pre-rendered tranquil ambient sky background renders with soft drifting leaves while the snapshot resolves, avoiding spinners or blank flashes.

### 7.2 Idle Micro-Motion
- Procedural kinematics:
  - Breathing oscillation: sinusoidal scale ($y$-axis $\pm 1.5\%$) on a 2.8s loop.
  - Weight shuffle: micro-rotations ($\pm 2^{\circ}$) every 8–15s.
  - Head tilts: discrete target angles ($\pm 15^{\circ}$) triggered by audio events or curiosity traits.
  - Preening: localized feather ruffle sequences when in `content` mood.

### 7.3 Reduced-Motion Mode
- In active reduced-motion mode (via media query or settings):
  - Procedural micro-motion loops are disabled.
  - Animated perch hops and flight transitions are replaced with smooth 600ms opacity cross-fades between static poses.
  - Ambient leaf/feather particle drift is disabled.
  - Day/night color transitions occur over 5-second gentle cross-fades.

---

## 8. Procedural Audio Pipeline

### 8.1 WebAudio Graph Architecture
```
[Motif Generator (Pitch / Rhythm Grammar)]
                |
                v
    [Oscillator / Formant Shaper]
                |
                v
       [Bandpass Filter (Species Color)]
                |
                v
    [Dynamic Envelope (ADSR GainNode)]
                |
                v
    [Per-Bird Pan & Gain Node] --------> [Listen-In Attenuator Node]
                                                     |
                                                     v
                                          [Master Chorus Bus]
                                                     |
                                                     v
                                             [AudioDestination]
```

### 8.2 Call Synthesis Mechanics
- Calls are constructed from procedural micro-motifs (chirps, trills, harmonic sweeps) parameterized by species and mood:
  - *Pitch & FM*: Dual oscillator FM synthesis reproducing natural avian syrinx harmonics.
  - *Cadence*: Inter-call intervals driven by `trait_vocal_frequency` and ambient weather dampening.
- **Listen-In Mix**:
  - Focusing bird $B_k$: Gain of $B_k$ smoothly ramps up $+4\text{dB}$ over 800ms; all other birds ramp down to $-14\text{dB}$ (never full mute).
  - Unfocusing restores all nodes to $0\text{dB}$ ambient balance over 1200ms.
- **Graceful WebAudio Fallback**:
  - If `AudioContext` fails to initialize, the audio system enters silent mode and automatically enables dynamic call captions. No canned or looped audio files are loaded.

---

## 9. Accessibility Surfaces

- **Screen-Reader Narration**:
  - Dedicated `aria-live="polite"` region containing rolling naturalist descriptions.
  - Queued at a relaxed pace (every 30–60s during idle, or immediately on offer/settle actions).
  - Always rendered in lowercase present-tense prose (e.g., *"a small grey bird rests on the front rail, preening softly in the morning light"*).
- **Call Captioning**:
  - Optional visual caption bubbles floating gently near the calling bird with text derived from the active motif (e.g., *"a soft two-note rise"*), fading out over 2 seconds.
- **Keyboard & Focus Navigation**:
  - Top bar items and aviary birds are accessible via `Tab` / `Shift+Tab`.
  - Arrow keys navigate between birds; `Enter` engages listen-in; `Escape` disengages listen-in.
  - Visible high-contrast focus rings meet WCAG AA contrast standards (>3:1 against aviary backgrounds).

---

## 10. Performance Budgets & Observability

### 10.1 Budgets
- **Initial JS Bundle**: $\le 1.8\text{MB}$ uncompressed ($\le 450\text{kB}$ gzipped).
- **Time-to-First-Bird**: $< 500\text{ms}$ on 4G connections.
- **Runtime Frame Budget**: Rock-solid 60fps on 5-year-old hardware.
- **Memory Footprint**: Flat heap profile over 30+ minute sessions (WebAudio nodes recycled, particle pools pre-allocated).

### 10.2 Privacy-Preserving Observability
- **Measured**:
  - Aggregate API latencies (p50, p95, p99).
  - Server simulation tick execution time (p99 alarm threshold at 5.0s).
  - Client-side render FPS drops and WebAudio context error rates.
  - Anonymized session duration distribution histograms.
- **Strictly Excluded**:
  - Zero logging or aggregation of per-bird trait values, individual interaction frequencies, or user-bird relationship graphs.

---

## 11. Rollout & Aviary Scaling Plan

1. **Phase 1 (Alpha Foundation)**:
   - Core WebAudio synthesis engine and 2 starter species.
   - Server simulation tick and append-only event ingestion.
   - Verification of presence detection conjunction logic.
2. **Phase 2 (Beta Experience)**:
   - Full 6-species pool integration.
   - Field notebook prose generation and naturalist narration.
   - Reduced-motion mode and call captioning verification.
3. **Phase 3 (General Availability)**:
   - Aviary age-based scaling unlocks (Bird 3 available at 30 days aviary age, capping at 7 birds at 1 year).
   - Read-only visit invitation rollout.

---

## 12. Risk Analysis & Mitigation Strategies

| Risk | Impact | Mitigation Strategy |
| :--- | :--- | :--- |
| **Drift Calibration Imbalance** | Traits saturate too quickly or fail to move noticeably over weeks. | Implement automated CI simulation test harness that simulates 1, 7, 21, and 60 days of virtual presence and validates trait drift envelopes. |
| **False Presence Inflation** | Backgrounded tabs inflate presence hours. | Strictly require the conjunction of `document.visibilityState === 'visible'`, `document.hasFocus()`, and user input within last $N$ minutes. |
| **Audio Uncanniness / Fatigue** | Procedural calls sound robotic or grating. | Use multi-oscillator formant synthesis with micro-randomized pitch, attack envelopes, and ambient silence pauses. |
| **Multi-Device Clock Skew** | Client timestamp inconsistencies in event logs. | Server-authoritative ingestion timestamps assigned upon HTTP receipt; client timestamps used only for relative event sequencing. |
| **Accessibility Degradation** | Screen-reader queue becomes flooded or clinical. | Hard cap live region announcements to maximum 1 update per 30 seconds unless user-triggered; strictly enforce naturalist copy templates in automated linting. |
