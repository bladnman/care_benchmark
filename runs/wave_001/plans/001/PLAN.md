# Pocket Aviary — System Implementation Plan (v1)

## 1. Scope & Architectural Principles

### 1.1 In-Scope for v1
- Single-user virtual aviary running strictly in modern web browsers (Chrome, Safari, Firefox, Edge; latest two major versions).
- Aviary population starting with 2 adopted birds, scaling via aviary chronological age up to a hard cap of 7 birds, chosen from a 6-species pool.
- Single horizontal scene fitting within the browser viewport without scrolling, panning, or zooming, featuring 3 distinct perch zones (front, middle, back), subtle parallax, ambient micro-motion (leaf/feather drift), dynamic local-time day/night cycles, and rare ambient weather (gentle rain, soft wind).
- Procedural client-side audio synthesis using the Web Audio API with emergent chorus behavior, dynamic mix rebalancing for the **listen-in** gesture, and naturalistic call captioning.
- Core gestures: idle attention (**presence**), **return-greeting**, **listen-in**, **offer** (seed, song-fragment motif, still pool), **settle** (with 5-second cancel affordance), and the auto-generated naturalist **field notebook**.
- Single-user accounts with 15-minute magic links, synthetic UUID tenant partitioning, revocable per-device session tokens, self-serve JSON export, and 30-day soft-deletion grace period.
- Strict canonical server-side simulation tick (~1 minute cadence) advancing drift and moods independently of client connections.
- Optional, private, read-only visit invitations sent via email link, with immediate host revocation and silent visit logging.
- Screen-reader naturalist prose narration via ARIA live regions, dedicated reduced-motion mode (slow cross-fading static poses instead of frame-by-frame animation or path interpolation), and WCAG AA contrast compliance.

### 1.2 Out-of-Scope (Strict Non-Goals)
- **No native applications:** Strictly web-only; zero iOS/Android native codebases, zero app-store packaging.
- **No gamification:** Zero achievements, streaks, levels, XP, scores, green-dot visit calendars, visit counters, or badges.
- **No Tamagotchi mechanics:** Birds never die, starve, fall ill, or show distress. No happiness meters; absence produces quiet ambient behavior, never punishment.
- **No social network surfaces:** No user profiles, follow graphs, public directory, aviary discovery feeds, leaderboards, visit comments, visitor avatars/cursors, or co-presence.
- **No notifications or announcements:** Zero push notifications, promotional emails, "friend visited" toasts, or "Welcome back!" banners. The bird greeting is the sole return surface.
- **No client-authoritative state:** Clients never calculate or mutate personality vectors or canonical positions.

---

## 2. System Architecture & Boundaries

```
                 +-------------------------------------------------------------+
                 |                         Browser Client                      |
                 |  +-------------------------------------------------------+  |
                 |  | Viewport Scene (Canvas / WebAudio / DOM ARIA Live)    |  |
                 |  +-------------------------------------------------------+  |
                 |        ^ Snapshot Pull                | Interaction Events  |
                 |        | (HTTP GET Poll/Keepalive)    | (HTTP POST Append)  |
                 +--------|------------------------------|---------------------+
                          |                              v
+-------------------------|------------------------------------------------------------+
| Server / Edge           |                              |                             |
|  +--------------------------------+          +------------------------------------+  |
|  |  Snapshot & Auth API Service   |          |  Event Ingestion Service           |  |
|  |  (Edge CDN Cached / Read-Only) |          |  (Validates, Authenticates, Enque) |  |
|  +--------------------------------+          +------------------------------------+  |
|                 ^                                              |                     |
|                 | Reads Snapshot                               v Writes Events       |
|  +--------------------------------------------------------------------------------+  |
|  |                          Append-Only Event Store (Postgres)                    |  |
|  +--------------------------------------------------------------------------------+  |
|                                         |                                            |
|                                         v Consumes batch & computes                  |
|  +--------------------------------------------------------------------------------+  |
|  |                       Simulation Engine Worker (Tick Runner)                   |  |
|  |  - Personality Drift Low-Pass Filter     - Mood Transitions & Ambient Events   |  |
|  |  - Perch Spatial Assignment             - Notebook Naturalist Observation Gen  |  |
|  +--------------------------------------------------------------------------------+  |
|                                         |                                            |
|                                         v Atomic commit                              |
|  +--------------------------------------------------------------------------------+  |
|  |                 Canonical Aviary State Store (Postgres / Read-Replica)         |  |
|  +--------------------------------------------------------------------------------+  |
+--------------------------------------------------------------------------------------+
```

### 2.1 Service Decomposition
1. **Edge/Gateway & Auth Service:**
   - Issues and verifies 15-minute magic links via email.
   - Manages revocable per-device session tokens stored in HttpOnly, Secure, SameSite cookies.
   - Translates verified user sessions into the internal synthetic UUID (`account_id`).
   - Serves the static web bundle and bootstraps the initial HTML document with an inlined, edge-rendered initial state snapshot for sub-500ms first-bird visibility.
2. **Event Ingestion API:**
   - Append-only write endpoint accepting user interaction events (`presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `bird_rename`).
   - Rejects unauthorized or malformed events; enforces rate limits and validates event structure.
3. **Simulation Tick Worker Daemon:**
   - Runs a periodic cron/worker loop (~60 seconds cadence) across active accounts.
   - Processes accumulated interaction events, advances personality drift vectors via additive low-pass filtering, evaluates mood transitions based on diurnal cycles and ambient weather, assigns target bird perches, and generates field notebook observations.
4. **State Snapshot Read API:**
   - Returns lightweight, compact JSON snapshots (~2–5 KB) containing birds, moods, coordinates, diurnal phase, weather, and call schedules.

### 2.2 Privacy & Isolation Boundary
- The external identity (`email`) is stored strictly in the isolated `accounts` table, encrypted at rest.
- All inter-service communication, simulation state tables, audit logs, and telemetry pipelines identify tenants exclusively via the synthetic UUID (`account_id`).
- Per-bird interaction logs and personality vectors are strictly partitioned by `account_id` and are physically isolated from analytics warehouses. No aggregate machine learning or population-level behavior clustering is permitted on interaction records.

---

## 3. Data Model & Storage Specifications

### 3.1 PostgreSQL Relational Schema

```sql
-- Core Account & Auth
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    encrypted_email BYTEA NOT NULL UNIQUE,
    email_bidx VARCHAR(64) NOT NULL UNIQUE, -- HMAC blind index for query lookup
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    soft_deleted_at TIMESTAMPTZ NULL,
    settings JSONB NOT NULL DEFAULT '{"visit_notifications": false, "reduced_motion": null}'::jsonb
);

CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    device_fingerprint TEXT NOT NULL,
    user_agent TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_seen_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at TIMESTAMPTZ NULL
);

CREATE TABLE magic_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    expires_at TIMESTAMPTZ NOT NULL,
    consumed_at TIMESTAMPTZ NULL
);

-- Aviary & Birds
CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    current_weather VARCHAR(32) NOT NULL DEFAULT 'clear', -- clear, gentle_rain, soft_wind
    weather_until TIMESTAMPTZ NULL,
    settled_at TIMESTAMPTZ NULL,
    tick_sequence BIGINT NOT NULL DEFAULT 0,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL, -- e.g., 'song_sparrow', 'mourning_dove', etc.
    name VARCHAR(64) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    perch_zone VARCHAR(16) NOT NULL DEFAULT 'front', -- front, middle, back
    current_mood VARCHAR(16) NOT NULL DEFAULT 'alert', -- wary, content, curious, drowsy, alert
    mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    -- Hidden personality vector (scalars 0.0 to 1.0, strictly server-authoritative)
    boldness DOUBLE PRECISION NOT NULL DEFAULT 0.35,
    social_warmth DOUBLE PRECISION NOT NULL DEFAULT 0.40,
    vocal_frequency DOUBLE PRECISION NOT NULL DEFAULT 0.50,
    plumage_saturation DOUBLE PRECISION NOT NULL DEFAULT 0.30,
    curiosity DOUBLE PRECISION NOT NULL DEFAULT 0.35,

    CONSTRAINT chk_boldness CHECK (boldness >= 0.0 AND boldness <= 1.0),
    CONSTRAINT chk_warmth CHECK (social_warmth >= 0.0 AND social_warmth <= 1.0),
    CONSTRAINT chk_vocal CHECK (vocal_frequency >= 0.0 AND vocal_frequency <= 1.0),
    CONSTRAINT chk_plumage CHECK (plumage_saturation >= 0.0 AND plumage_saturation <= 1.0),
    CONSTRAINT chk_curiosity CHECK (curiosity >= 0.0 AND curiosity <= 1.0)
);

-- Append-Only Interaction Event Log
CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    session_id UUID NOT NULL REFERENCES sessions(id) ON DELETE CASCADE,
    event_type VARCHAR(32) NOT NULL, -- presence_ping, listen_in_start, listen_in_end, offer, settle
    bird_id UUID NULL REFERENCES birds(id) ON DELETE SET NULL,
    payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    client_timestamp TIMESTAMPTZ NOT NULL,
    server_timestamp TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_at TIMESTAMPTZ NULL
);
CREATE INDEX idx_events_unprocessed ON interaction_events (account_id, id) WHERE processed_at IS NULL;

-- Naturalist Field Notebook
CREATE TABLE notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    prose TEXT NOT NULL
);
CREATE INDEX idx_notebook_aviary ON notebook_entries (aviary_id, created_at DESC);

-- Visit Invitations (Optional Social)
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    encrypted_visitor_email BYTEA NOT NULL,
    visitor_email_bidx VARCHAR(64) NOT NULL,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invitation_id UUID NOT NULL REFERENCES visit_invitations(id) ON DELETE CASCADE,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    ended_at TIMESTAMPTZ NULL
);
```

---

## 4. API Surface & Protocols

### 4.1 Client Authentication & Account Management
- `POST /api/v1/auth/magic-link/request`: `{ "email": "user@example.com" }` -> Generates token, dispatches email. Returns generic `200 OK` regardless of whether email exists.
- `GET /api/v1/auth/magic-link/verify?token=...`: Verifies token, creates session cookie, redirects to `/`.
- `POST /api/v1/auth/session/revoke`: Revokes targeted session ID or current session.
- `GET /api/v1/account/export`: Generates downloadable sanitized JSON snapshot of birds, vectors, moods, and notebook.
- `POST /api/v1/account/delete`: Initiates 30-day soft deletion.
- `POST /api/v1/account/restore`: Reverses soft deletion if within 30-day window.

### 4.2 Aviary State & Interaction Stream
- `GET /api/v1/aviary/snapshot`:
  - Returns canonical aviary snapshot.
  - Response JSON:
    ```json
    {
      "aviary_id": "8b9a12c4-...",
      "tick_sequence": 48291,
      "server_time": "2026-09-06T15:57:00Z",
      "weather": "clear",
      "is_settled": false,
      "birds": [
        {
          "id": "5f1b2c3d-...",
          "species_id": "song_sparrow",
          "name": "Pip",
          "perch_zone": "front",
          "current_mood": "curious",
          "plumage_saturation": 0.42,
          "target_coords": { "x": 0.32, "y": 0.65 }
        }
      ]
    }
    ```
- `POST /api/v1/aviary/events`:
  - Batch appends interaction events to the event store.
  - Body:
    ```json
    {
      "events": [
        {
          "event_type": "presence_ping",
          "client_timestamp": "2026-09-06T15:57:10Z",
          "payload": { "focus_duration_seconds": 60 }
        },
        {
          "event_type": "listen_in_start",
          "bird_id": "5f1b2c3d-...",
          "client_timestamp": "2026-09-06T15:57:15Z"
        }
      ]
    }
    ```

### 4.3 Social Visit API
- `POST /api/v1/visits/invite`: Host issues invite for `visitor_email`.
- `GET /api/v1/visits/view?token=...`: Returns guest-scoped read-only session and initial host aviary snapshot.
- `POST /api/v1/visits/revoke`: Host revokes invitation ID.
- `GET /api/v1/visits/log`: Returns list of visits for host review.

---

## 5. Simulation Engine Design

### 5.1 Server-Side Simulation Tick Loop
The simulation tick runs on an asynchronous worker pool every 60 seconds per aviary.

```
+-------------------------------------------------------------------------------+
|                               TICK EXECUTION PIPELINE                         |
|                                                                               |
|  1. Ingest unprocessed interaction events for account                         |
|  2. Calculate verified presence time (windowed & clamped)                     |
|  3. Compute personality vector drift deltas (monotonic, low-pass filter)      |
|  4. Evaluate Diurnal & Ambient Weather state transitions                      |
|  5. Transition Mood states based on interactions, time-of-day, and weather    |
|  6. Evaluate Perch Placement preferences based on boldness and mood           |
|  7. Sparsely generate Field Notebook naturalist observations (~3-5 days)      |
|  8. Increment tick_sequence and write atomic snapshot                         |
+-------------------------------------------------------------------------------+
```

### 5.2 Presence Verification & Calibration
To prevent background tab drift inflation, the client emits `presence_ping` events every 60 seconds only when:
1. `document.visibilityState === "visible"`
2. `document.hasFocus() === true`
3. Last pointermove or keydown timestamp was within the last 180 seconds (3 minutes).

The server verifies the presence ping interval, rejects pings with invalid timestamps, and clamps accumulated presence to at most 60 seconds per tick.

### 5.3 Monotonic Drift Function
Drift is strictly additive and positive toward expressiveness. Neglect never degrades traits.
Let $T$ be the current trait value, $\Delta t_{presence}$ be valid presence seconds in the tick, and $w_{gesture}$ be gesture weights.

$$\Delta_{raw} = (\alpha \cdot \Delta t_{presence}) + \sum (\beta_g \cdot G_g)$$
$$\Delta_{filtered} = \text{clamp}\left(\Delta_{raw} \cdot \frac{1.0 - T}{K}, 0, \Delta_{max}\right)$$
$$T_{new} = T_{old} + \Delta_{filtered}$$

- **Presence weight ($\alpha$):** Dominant driver ($~0.00002$ per minute of active presence).
- **Listen-in gesture weight ($\beta_{listen}$):** Drifts social warmth and vocal frequency ($+0.0001$ per event).
- **Offer gesture weight ($\beta_{offer}$):** Offering seeds/pools drifts curiosity and boldness ($+0.00015$ per event).
- **Asymmetry guarantee:** $\Delta_{filtered} \ge 0$. If an account is inactive for weeks, $\Delta t = 0$, so $\Delta_{filtered} = 0$. Traits remain unchanged; birds never become wary or dull from absence.
- **Calibration target:** Under regular daily sessions (10–15 minutes/day), measurable instrument delta ($\Delta \approx 0.02$) occurs at ~7 days; discernible visual/auditory behavioral delta occurs at ~21 days.

### 5.4 Mood Transitions & Diurnal Modeling
Moods belong to `{ wary, content, curious, drowsy, alert }`:
- **Diurnal curve:** Morning hours (06:00–11:00 local time) bias probabilities toward `alert` and `curious`. Midday biases toward `content`. Dusk and evening bias toward `drowsy`. Full night transitions birds to sleeping postures (`drowsy`/settled) on middle/back perches, with the nightjar species remaining active.
- **Ambient weather:** Rain dampens `vocal_frequency` by 30% and nudges birds toward `drowsy` or `content` under leaf cover. High wind increases vigilance (`alert`).
- **Session events:** Accepted offers nudge mood to `content` or `curious`.

### 5.5 Species Expansion Schedule
New bird adoption offers trigger based strictly on aviary chronological age:
- **Aviary Creation:** 2 starter birds assigned from the 6-species pool.
- **Day 60 (2 months):** 3rd bird arrives.
- **Day 150 (5 months):** 4th bird arrives.
- **Day 270 (9 months):** 5th bird arrives.
- **Day 400 (13 months):** 6th bird arrives.
- **Day 550 (18 months):** 7th bird arrives (maximum cap reached).

---

## 6. Sync Model & Conflict Prevention

### 6.1 Single Canonical Writer
- The server tick runner is the **sole writer** of bird personality vectors, moods, and aviary state.
- Multiple clients (e.g. mobile Safari and desktop Chrome) never exchange peer-to-peer state or compute predictive local personality drift.
- Clients submit events to the append-only event log.
- Clients poll `GET /api/v1/aviary/snapshot` at a 30-second keepalive interval while the tab is active and immediately upon `visibilitychange` transitioning to `visible`.

### 6.2 Interpolation vs. Teleportation
When a client receives a new snapshot indicating a bird has moved from Perch 1 (`x1, y1`) to Perch 2 (`x2, y2`), the client does not snap the sprite. It executes a procedural bezier hop or flutter transition over 1.2–2.0 seconds to interpolate between states smoothly.

### 6.3 Sync Error Handling
Any session invalidation or revocation surfaces matter-of-fact messaging in standard system typography:
> "Your session timed out. Sign in again to keep watching."

---

## 7. Frontend Rendering Pipeline

### 7.1 Viewport & Scene Structure
- Single responsive HTML5 Canvas layered beneath a lightweight SVG/HTML accessibility DOM overlay.
- Scene aspect ratio is bounded (min 4:3, max 21:9). On mobile viewports, background margins crop while all three perch zones and all active birds remain strictly on-screen.
- Parallax layers:
  1. Sky and distant tree line (0.15x parallax offset).
  2. Back perch zone (0.4x parallax offset).
  3. Mid perch zone with primary foliage (0.7x parallax offset).
  4. Front perch zone (1.0x parallax offset).
  5. Foreground canopy and occasional leaf drift (1.2x parallax offset).

### 7.2 Immediate First Frame (Sub-500ms Render)
- Static sky gradient and initial perch geometry are rendered immediately upon script evaluation using inlined snapshot data in the HTML.
- Birds are rendered directly in their current idle poses (mid-preen, head-tilted) on frame 1.
- No loading spinners, entry splash animations, or fade-from-black sequences are ever displayed.

### 7.3 Idle Micro-Motion
- Procedural sinusoidal breathing cycle (0.3 Hz).
- Stochastic head-tilts and micro-hops scheduled via Poisson intervals influenced by bird curiosity and alertness.
- Plumage rendering dynamically scales color vibrancy from HSV color definitions based on `plumage_saturation`.

### 7.4 Reduced-Motion Mode
When `prefers-reduced-motion` is detected or toggled:
- Canvas animations at 60fps are disabled.
- Perch changes and preening cycles are replaced with slow, soft CSS cross-fades (1.5-second opacity cross-fade) between static rendered poses.
- Continuous leaf and feather drift particles are removed from the scene.
- Lighting transitions (day to dusk) are smoothed across 30-second linear ramps.

---

## 8. Audio Pipeline

### 8.1 Procedural Call Synthesis (Web Audio API)
Calls are synthesized dynamically at runtime using Web Audio nodes, avoiding pre-recorded audio loops entirely:

```
+-----------------------------------------------------------------------------+
|                          PROCEDURAL CALL SYNTHESIZER                        |
|                                                                             |
|  [ Motifs Library ] -> Frequency & Pitch Envelope Modulator                 |
|                                 |                                           |
|         +-----------------------+-----------------------+                   |
|         | Syringeal Oscillator  | Overtone Harmonic Osc |                   |
|         | (Sine / Triangle)     | (FM Modulation)       |                   |
|         +-----------------------+-----------------------+                   |
|                                 |                                           |
|                         [ Formant Filter ]                                  |
|                                 |                                           |
|                   [ Dynamic Envelope GainNode ]                             |
|                                 |                                           |
|                     [ Spatial PannerNode ]                                  |
|                                 |                                           |
|                   [ Bird Individual GainNode ]                              |
|                                 |                                           |
|                 +---------------+---------------+                           |
|                 |                               |                           |
|                 v                               v                           |
|      [ Master Aviary Bus ]             [ Listen-In Bus ]                    |
|                 |                               |                           |
|                 +---------------+---------------+                           |
|                                 |                                           |
|                                 v                                           |
|                     [ AudioContext Destination ]                            |
+-----------------------------------------------------------------------------+
```

- Each species possesses a distinct grammar of frequency motifs (e.g. rising whistle, frequency-modulated trill, soft percussive click).
- Per-bird variance: Base frequency is modulated by individual pitch offsets; note duration and cadence are scaled by the bird's `vocal_frequency` trait.
- Recognizability: A user easily distinguishes Pip's specific motif envelope and timbre from Wren's.

### 8.2 Chorus & Listen-In Mix Rebalancing
- **Ambient Chorus:** Calling scheduling uses Poisson timing. When Bird A calls, Bird B's social warmth determines the probability of emitting a counter-call after a 400–800ms offset.
- **Listen-in Gesture Dynamics:**
  - Clicking or keyboard-focusing Bird A initiates an exponential gain ramp over 1.8 seconds.
  - Bird A's channel increases to $+3\text{ dB}$.
  - All other birds' channels ramp down by $-12\text{ dB}$ (attenuated to soft ambient, never muted).
  - De-selection ramps all birds back to unity gain over 2.5 seconds.

### 8.3 Web Audio Fallback
If Web Audio initialization fails or is blocked by browser autoplay policies:
- The aviary operates in graceful silence.
- No recorded audio files are substituted.
- Naturalist call captions are automatically activated.

---

## 9. Accessibility Surfaces

### 9.1 Naturalist Screen-Reader Narration
- An invisible ARIA live region (`aria-live="polite"`, `role="status"`) announces aviary status in running naturalist prose every 30–60 seconds.
- Sample outputs:
  - *"pip is on the low perch this morning, fluffed against the cool air. wren called softly from the high branch."*
  - *"a gentle rain has begun. the birds have moved under the pine needles."*
- Immediate priority updates are issued only on explicit user actions (e.g. return greeting or accepting an offer).

### 9.2 Call Captions
- Positioned dynamically as subtle HTML overlay badges adjacent to the vocalizing bird.
- Prose is procedurally matched to the synthesized motif:
  - *"a soft three-note rise"*
  - *"a low trill, paused, low trill again"*
- Captions smoothly fade in over 300ms and fade out over 500ms following call completion.

### 9.3 Keyboard Navigation & Focus
- Interactive elements inside the aviary are mapped into an accessible DOM order:
  1. Top bar items: Settings, Accessibility, Field Notebook, Offer.
  2. Aviary canvas container: Tab enters the aviary, focusing the first bird.
  3. Arrow keys (`ArrowRight`, `ArrowLeft`) cycle focus between birds.
  4. `Enter` / `Space` engages listen-in on the focused bird.
  5. `Escape` disengages listen-in or closes modals.
- Focus outlines use high-contrast dual rings (inner white, outer dark charcoal) to ensure visibility across daytime and nighttime backgrounds.

---

## 10. Performance Budgets & Observability

### 10.1 Budgets
- **Initial JS Bundle Size:** $\le 2.0\text{ MB}$ gzipped (target $\le 850\text{ KB}$ core engine; settings, visits, and notebook code-split via dynamic imports).
- **Time to First Bird Visible:** $< 500\text{ ms}$ on mid-tier mobile (Moto G series) over 4G LTE.
- **Framerate Budget:** Continuous 60fps on a 5-year-old mid-range laptop (Intel Core i5 8th gen, integrated UHD graphics).
- **Memory Footprint:** Flat memory graph over 30 minutes continuous execution. Zero leaking Web Audio nodes, reusable object pools for particles, and DOM recycling for notebook entries.

### 10.2 Synthetic & Real User Monitoring (RUM)
- Track p50, p90, p99 for:
  - First Bird Render (FBR).
  - Simulation Tick execution latency (p99 alarm set at $> 5.0\text{ s}$).
  - Audio buffer underrun / context creation failure rate.
  - WebGL/Canvas frame drop percentage.
- **Telemetry Boundaries:**
  - Metrics collected are aggregate operational numbers only.
  - Zero logging of bird names, bird personality traits, user notebook observations, or presence timestamps to analytics databases.

---

## 11. Rollout & Ramp Plan

### 11.1 Phase Progression
1. **Phase 1: Simulation & Audio Foundation (Weeks 1–4)**
   - Implement PostgreSQL event log, simulation daemon, and monotonic drift math.
   - Develop Web Audio procedural call synthesis engine for 6 starter species.
2. **Phase 2: Canvas Rendering & Naturalist UX (Weeks 5–8)**
   - Construct responsive canvas scene with 3 perch zones, day/night cycles, and micro-motion.
   - Build ARIA screen-reader narration generator and reduced-motion cross-fades.
   - Implement top bar chrome with auto-fade behavior.
3. **Phase 3: Auth, Sync & Field Notebook (Weeks 9–11)**
   - Implement magic link auth, synthetic UUID tenant partitioning, and 30-day soft deletion.
   - Construct field notebook naturalist observation generator.
   - Implement read-only social visit invitation flow.
4. **Phase 4: Calibration, Hardening & Staged Launch (Weeks 12–14)**
   - Synthetic load testing with 50,000 simulated accounts executing 1-minute ticks.
   - Validate monotonic drift curves over simulated 30-day presence runs.
   - Production launch with 2 birds per new aviary.

---

## 12. Risk Matrix & Mitigations

| Risk Domain | Potential Failure Mode | Architectural Mitigation |
| :--- | :--- | :--- |
| **Drift Calibration** | Drift moves too quickly (turning into a Tamagotchi) or too slowly (feeling like a static screensaver). | Multi-stage presence filter (visibility + focus + recent activity); server clamps drift additions; automated test harness asserts $\Delta$ thresholds across simulated 7-day and 21-day user profiles. |
| **Multi-Device Desync** | Race conditions or overwrites between laptop and mobile sessions. | Enforce single-canonical-writer model: only the server simulation tick updates state; clients append events only; strict additive deltas prevent last-write-wins collisions. |
| **Audio Uncanniness** | Synthesized calls sounding harsh, synthetic, or phase-canceling during chorus events. | Pure mathematical procedural synthesis using harmonic overtones and formant filtering; randomized micro-delays between responsive calls; graceful silent fallback with captions if Web Audio fails. |
| **Accessibility Degradation** | Accessible surfaces feeling like clinical state dumps rather than a living aviary. | Naturalist field-notebook prose engine powers ARIA live narration and call captions; reduced-motion mode designed as an intentional, contemplative cross-fade aesthetic. |
| **Privacy / PII Leak** | User emails leaking into operational logs, partitioned shards, or telemetry. | Immediate translation of email to synthetic UUID at auth boundary; HMAC blind indexing for queries; strict physical firewall between simulation DB and analytics systems. |

