# Pocket Aviary — Comprehensive v1 Implementation Plan

## 1. Executive Summary & Scope Boundary

Pocket Aviary is a calm, browser-based virtual aviary where 2 to 7 animated birds live and drift in response to a user's quiet presence over days and weeks. This document establishes the complete, production-grade technical implementation plan for a frontier engineering team to execute without further clarification.

### 1.1 In-Scope Features (v1)
*   **Aviary Population:** Starts with 2 starter birds from a 6-species pool; expands up to a maximum cap of 7 birds tied to aviary age thresholds.
*   **Authentication & Accounts:** Single-user accounts signed in via 15-minute email magic links. Synthetic UUID internal keying with encrypted email storage.
*   **Multi-Device Sync:** Server-side authoritative simulation tick (~60s interval); client reads immutable state snapshots. Zero peer-to-peer or last-write-wins state conflicts.
*   **Presence Accounting Engine:** Measures user attention via strict conjunction of `visibilityState === 'visible'`, active window focus, and pointer/keyboard activity within a rolling multi-minute window.
*   **Bird Engine:** Server-persisted hidden personality vectors (boldness, social warmth, vocal frequency, plumage saturation, curiosity) drifting monotonically toward expressiveness over weeks. Fast-timescale mood state machine (wary, content, curious, drowsy, alert).
*   **Interactions:** Natural return-greeting on session load; listen-in spatial mix focusing; gesture offers (seed, song fragment, still pool) with per-bird cooldowns; opt-in settle evening transition.
*   **Field Notebook:** Auto-generated, read-only, sparse log of naturalist prose observations.
*   **Social Affordance (Optional & Quiet):** Read-only ambient visit invitations via one-time email link. Opt-in per invitation, instantly revocable, off by default, zero co-presence or interaction power for visitors.
*   **Accessibility Surfaces:** Naturalist screen-reader narration stream (`aria-live="polite"`), dynamic call captions generated from procedural motifs, full keyboard navigation, WCAG AA top-bar contrast, and a dedicated reduced-motion cross-fade rendering engine.
*   **Audio Pipeline:** Client-side WebAudio procedural call synthesis with motif libraries, spatial chorus mixing, listen-in gain decay, and a graceful silent fallback with captions.

### 1.2 Explicit Non-Goals (Out of Scope)
*   **No Gamification:** No streak counters, levels, scores, badges, adoption count badges, XP, rankings, or visit calendars.
*   **No Tamagotchi Mechanics:** No bird death, hunger, distress meters, or happiness decay. Absence produces quietness, never negative drift or punishment.
*   **No Social Network Features:** No public feeds, user profiles, comments, leaderboards, public discovery directories, avatars, or co-presence.
*   **No Native Mobile Apps:** Browser-only application targeting modern desktop and mobile viewports.
*   **No Push Notifications / Marketing Email:** Zero outbound re-engagement messaging.

---

## 2. Architecture & Service Boundaries

The system is structured as a client-server architecture where the server retains sole authority over simulation state, personality vectors, and session history, while the client acts as a render-and-synthesis view layer.

```
[ Web Browser Client ]
  ├── View Layer: HTML5 Canvas / 2D WebGL (Scene, Micro-Motion, Parallax)
  ├── Audio Layer: WebAudio Synth (Procedural Motifs, Spatial Panner, Compressor)
  └── Sync Layer: Snapshot Fetcher & Event Batch Emitter
          ▲                             │
   GET /snapshot (CDN/Edge)      POST /events (HTTPS)
          │                             ▼
┌────────────────────────────────────────────────────────────────────────┐
│                        API Gateway / Edge Router                       │
└────────────────────────────────────────────────────────────────────────┘
     │                    │                                   │
     ▼                    ▼                                   ▼
┌─────────┐      ┌──────────────────┐               ┌────────────────────┐
│ Auth &  │      │ Event Ingestion  │               │ Visit & Social     │
│ Account │      │ Service          │               │ Service            │
└─────────┘      └──────────────────┘               └────────────────────┘
     │                    │                                   │
     │            (Append-Only Log)                           │
     │                    │                                   │
     ▼                    ▼                                   ▼
┌────────────────────────────────────────────────────────────────────────┐
│                   Server-Side Simulation Tick Engine                   │
│   - Reads Recent Interaction Log & Presence Pings                      │
│   - Computes Low-Pass Personality Drift                                │
│   - Evaluates Mood State Machine & Time-of-Day / Weather               │
│   - Generates Sparse Naturalist Notebook Entries                       │
│   - Writes Canonical Aviary Snapshot                                   │
└────────────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌────────────────────────────────────────────────────────────────────────┐
│                    PostgreSQL / Redis Primary Store                    │
│   - Accounts (UUID, Encrypted PII)                                     │
│   - Aviaries & Bird Entities                                           │
│   - Canonical Personality Vectors & Mood States                        │
│   - Notebook Records & Visit Invitations                               │
└────────────────────────────────────────────────────────────────────────┘
```

### 2.1 Edge & Application Gateway
*   **HTML Shell & Initial Snapshot Injection:** On initial navigation, the edge server validates the session JWT, fetches the current aviary snapshot, and embeds it directly into the initial HTML document payload within a `<script id="initial-state">` tag. This guarantees initial frame render under 500ms without a secondary client fetch.
*   **Static Asset Delivery:** Compiled JS bundles, CSS design system tokens, and SVG environmental assets are cached and served via global CDN edge nodes.

### 2.2 Microservices & Processing Subsystems
1.  **Auth & Account Service:** Manages magic link request/verification cycles, issues device-bound session JWTs, handles soft-delete workflows (30-day window), and executes JSON account data exports.
2.  **Event Ingestion Service:** High-throughput HTTP service receiving append-only event batches (`presence_ping`, `listen_in_start`, `offer_gesture`, `settle_action`). Validates event signatures and pushes to an append-only stream.
3.  **Simulation Tick Engine:** Autonomous background worker running on a ~60-second periodic schedule. Processes queued interaction events, executes low-pass drift math, shifts mood states, advances environmental time/weather, and commits canonical snapshots.
4.  **Social & Visit Subsystem:** Issues single-use 30-day visit tokens via email, validates incoming visitor requests, and serves read-only snapshots while blocking event ingestion for visitor sessions.

---

## 3. Data Model & Database Schemas

All internal keys use synthetic UUIDv4 identifiers. User emails are stored encrypted in a isolated column and never referenced in logs or foreign keys.

### 3.1 Relational Schema (PostgreSQL)

```sql
-- Accounts Table
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deleted_at TIMESTAMPTZ NULL,
    visit_notifications_enabled BOOLEAN NOT NULL DEFAULT FALSE
);

-- Aviaries Table (1 per Account at v1)
CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL UNIQUE REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    settled_at TIMESTAMPTZ NULL
);

-- Birds Table
CREATE TYPE species_enum AS ENUM ('warbler', 'sparrow', 'nightjar', 'finch', 'chickadee', 'nuthatch');
CREATE TYPE perch_zone_enum AS ENUM ('front', 'middle', 'back');

CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species species_enum NOT NULL,
    name VARCHAR(64) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    perch_zone perch_zone_enum NOT NULL DEFAULT 'middle'
);

-- Hidden Personality Vectors (Server Canonical Only - Never Exposed to UI)
CREATE TABLE personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(id) ON DELETE CASCADE,
    boldness FLOAT NOT NULL DEFAULT 0.5,
    social_warmth FLOAT NOT NULL DEFAULT 0.5,
    vocal_frequency FLOAT NOT NULL DEFAULT 0.5,
    plumage_saturation FLOAT NOT NULL DEFAULT 0.5,
    curiosity FLOAT NOT NULL DEFAULT 0.5,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT boldness_range CHECK (boldness BETWEEN 0.0 AND 1.0),
    CONSTRAINT warmth_range CHECK (social_warmth BETWEEN 0.0 AND 1.0),
    CONSTRAINT vocal_range CHECK (vocal_frequency BETWEEN 0.0 AND 1.0),
    CONSTRAINT plumage_range CHECK (plumage_saturation BETWEEN 0.0 AND 1.0),
    CONSTRAINT curiosity_range CHECK (curiosity BETWEEN 0.0 AND 1.0)
);

-- Fast-Timescale Mood States
CREATE TYPE mood_enum AS ENUM ('wary', 'content', 'curious', 'drowsy', 'alert');

CREATE TABLE bird_moods (
    bird_id UUID PRIMARY KEY REFERENCES birds(id) ON DELETE CASCADE,
    current_mood mood_enum NOT NULL DEFAULT 'content',
    entered_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    cooldown_until TIMESTAMPTZ NULL
);

-- Append-Only Interaction & Presence Event Log
CREATE TYPE event_type_enum AS ENUM (
    'presence_ping', 'listen_in_start', 'listen_in_end', 
    'offer_seed', 'offer_song', 'offer_pool', 'settle'
);

CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    event_type event_type_enum NOT NULL,
    target_bird_id UUID NULL REFERENCES birds(id) ON DELETE SET NULL,
    duration_seconds INT NULL DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_interaction_events_tick ON interaction_events(aviary_id, created_at);

-- Field Notebook Observations
CREATE TABLE notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    observation_text TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Visit Invitations
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    visitor_email_encrypted BYTEA NOT NULL,
    token_hash VARCHAR(64) NOT NULL UNIQUE,
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 4. API Surface & Contract Specifications

### 4.1 Authentication & Account Management
*   `POST /api/v1/auth/magic-link`
    *   Request: `{ "email": "user@example.com" }`
    *   Response `202 Accepted`: `{ "status": "sent", "message": "Check your inbox for a sign-in link." }`
*   `GET /api/v1/auth/verify?token=<string>`
    *   Response `200 OK`: Sets HTTP-Only Secure JWT cookie (`SessionToken`), redirects to `/`.
    *   Response `400 Bad Request`: `{"error": "We couldn't sign you in. The link may have expired. Try requesting a new link."}` (Matter-of-fact register).
*   `POST /api/v1/account/export`
    *   Response `202 Accepted`: Triggers JSON payload generation delivered to verified email address.
*   `DELETE /api/v1/account`
    *   Response `200 OK`: Marks account `deleted_at = NOW()`. Enters 30-day soft-delete grace period.

### 4.2 Aviary Snapshot & Interaction Event Stream
*   `GET /api/v1/aviary/snapshot`
    *   Response `200 OK`:
        ```json
        {
          "aviary_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
          "server_time": "2026-07-24T06:52:03Z",
          "settled": false,
          "weather": "clear",
          "birds": [
            {
              "id": "c1a2b3d4-0001-4000-8000-000000000001",
              "species": "warbler",
              "name": "Pip",
              "perch_zone": "front",
              "mood": "curious",
              "plumage_saturation": 0.68,
              "vocal_frequency": 0.72
            },
            {
              "id": "c1a2b3d4-0002-4000-8000-000000000002",
              "species": "sparrow",
              "name": "Wren",
              "perch_zone": "middle",
              "mood": "content",
              "plumage_saturation": 0.54,
              "vocal_frequency": 0.45
            }
          ]
        }
        ```
*   `POST /api/v1/aviary/events`
    *   Request Body (Batch):
        ```json
        {
          "events": [
            { "type": "presence_ping", "duration_seconds": 60 },
            { "type": "offer_seed", "target_bird_id": "c1a2b3d4-0001-4000-8000-000000000001" }
          ]
        }
        ```
    *   Response `200 OK`: `{ "accepted": 2 }`

### 4.3 Social & Visit Endpoints
*   `POST /api/v1/social/invite`
    *   Request: `{ "visitor_email": "friend@example.com" }`
    *   Response `201 Created`: `{ "invite_id": "...", "expires_at": "..." }`
*   `GET /api/v1/visit/:token/snapshot`
    *   Response `200 OK`: Read-only snapshot payload. Interaction submission routes return `403 Forbidden` for visit sessions.
*   `DELETE /api/v1/social/invite/:id`
    *   Response `200 OK`: Immediately revokes invite. Active visitor clients receive `404/410` on next snapshot pull and render matter-of-fact surface: `"This visit is no longer available."`

---

## 5. Simulation Engine Design & Drift Physics

### 5.1 Server-Side Simulation Tick Protocol (~60s Schedule)
Every 60 seconds, the Simulation Engine executes the following pipeline per active aviary:

```
[ Fetch Pending Interaction Events ] ──► [ Evaluate Presence Windows ]
                                                   │
                                                   ▼
[ Apply Mood State Machine ] ◄── [ Calculate Low-Pass Personality Drift ]
             │
             ▼
[ Check Notebook Trigger Criteria ] ──► [ Commit Canonical State Snapshot ]
```

### 5.2 Presence Accounting Algorithmic Logic
A presence event is valid if and only if all three conditions are satisfied on the client:
1.  `document.visibilityState === 'visible'`
2.  `document.hasFocus() === true`
3.  `time_since_last_input < 180 seconds` (Keyboard or Pointer activity)

The client sends a `presence_ping` event every 60 seconds when all three conditions hold. The simulation engine aggregates valid presence pings into `presence_time` seconds.

### 5.3 Monotonic Personality Drift Physics
Personality traits drift via a low-pass discrete filter. For trait vector $V = [B, W, F, S, C]^T$:

$$V_{t+1} = V_t + \eta \cdot \Phi(\text{PresenceTime}, \text{Interactions}) \cdot (1 - V_t)$$

Where:
*   $\eta = 1.5 \times 10^{-6}$ per tick (calibration parameter ensuring visible changes require ~21 days of visits).
*   $\Phi \ge 0$ represents the weighted sum of valid presence time and gentle interactions (listen-in, accepted offers).
*   **Monotonicity Constraint:** $\Delta V = V_{t+1} - V_t \ge 0$. Traits never decrease due to neglect or absence. Absence yields $\Phi = 0 \implies V_{t+1} = V_t$.

### 5.4 Mood State Transition Matrix
Fast-timescale mood $M \in \{\text{wary}, \text{content}, \text{curious}, \text{drowsy}, \text{alert}\}$ transitions based on local time, weather, and recent gestures:

```
                 ┌──────────────┐
                 │    alert     │ (Early Morning / High Boldness)
                 └──────┬───────┘
                        │ time > 10:00
                        ▼
┌──────────┐  offer   ┌──────────────┐   dusk   ┌──────────────┐
│  wary    │─────────►│   curious    │─────────►│    drowsy    │
└────┬─────┘          └──────┬───────┘          └──────────────┘
     │                       │ quiet
     └──────────────────────►▼
                      ┌──────────────┐
                      │   content    │ (Baseline Idle)
                      └──────────────┘
```

---

## 6. Sync Architecture & Multi-Device Consistency

To eliminate last-write-wins race conditions and state divergence across devices:
1.  **Single Writer Principle:** Only the server-side simulation engine writes to `personality_vectors` and `bird_moods`.
2.  **Append-Only Event Sourcing:** Clients send gesture notifications (`offer`, `listen_in`), never absolute trait overrides.
3.  **Snapshot Read Replicas:** Multiple signed-in devices (e.g., laptop and phone) pull the same canonical snapshot from the server.
4.  **Client-Side Interpolation:** When a new snapshot arrives, the client smooths bird positional updates over a 2.5-second lerp window, preventing visual snapping or jumping.

---

## 7. Frontend Rendering Pipeline & Visual Scene Architecture

### 7.1 Scene Layering Stack (2D Canvas / WebGL)
The visual scene is rendered on a responsive, fixed-aspect-ratio 2D WebGL canvas:

1.  **Layer 0 (Background):** Dynamic sky color gradient (driven by local timezone sunrise/sunset math) + subtle cloud/weather vectors.
2.  **Layer 1 (Parallax Backing):** Distant tree silhouettes with low-frequency wind swaying.
3.  **Layer 2 (Back Perch Zone):** Perch geometry for wary/drowsy birds.
4.  **Layer 3 (Middle Perch Zone):** Primary perching area.
5.  **Layer 4 (Front Perch Zone):** High-boldness perch near the front rail.
6.  **Layer 5 (Foreground Particle System):** Ambient falling leaves and feather drift (client-side purely cosmetic micro-motion).
7.  **Layer 6 (UI Chrome & Accessibility):** Top bar overlay (fades after 3s cursor stillness) + call caption text bubbles.

### 7.2 Micro-Motion & Idle Procedural Physics
Birds are animated using procedural skeletal micro-motion:
*   **Breathing Cycles:** Soft sinus-wave chest scale variations (12–18 cycles/min).
*   **Head Tilts:** Discrete rotational jumps (15°–40°) triggered by curiosity and ambient calls.
*   **Preening Sequences:** Procedural beak-to-wing rotation keyframes when mood is `content`.

### 7.3 Reduced-Motion Engine Path (`prefers-reduced-motion`)
When reduced-motion is detected or enabled:
*   Continuous skeletal spring updates and leaf particle generators are disabled.
*   Bird pose transitions (e.g., preen pose to scan pose) are executed via slow 800ms cross-fades between static pose snapshots.
*   Perch changes cross-fade in place rather than translating along motion paths.

---

## 8. Audio Pipeline & Procedural WebAudio Synthesis

### 8.1 Procedural Call Motif Generator
Zero recorded audio files are shipped. All vocalizations are synthesized via WebAudio API nodes:

```
[ Master Compressor Node ] ◄── [ Spatial GainNode ] ◄── [ Biquad Filter ]
                                                              ▲
                                                              │
                                            [ FM Modulated Oscillator Pair ]
```

*   **Warble Motif:** Frequency-modulated sine wave pair with rapid exponential pitch ramps (1.2kHz to 3.4kHz over 120ms).
*   **Trill Motif:** Pulse-width modulated oscillator with a 15Hz vibrato LFO applied to gain and frequency.
*   **Nightjar Call:** Low-frequency resonant triangle wave (400Hz–800Hz) with soft attack envelope.

### 8.2 Chorus Panning & Spatial Mixing
Each bird possesses a dedicated `GainNode` and `StereoPannerNode` mapped to its horizontal perch position (`-0.8` left to `+0.8` right). A master `DynamicsCompressorNode` prevents clipping during simultaneous chorus events.

### 8.3 Listen-In Focus Dynamics
When the user engages listen-in on bird $B_k$:
*   $Gain(B_k)$ ramps smoothly from $1.0$ to $1.8$ over 1.5s using `.exponentialRampToValueAtTime`.
*   $Gain(B_j)$ for all $j \neq k$ ramps from $1.0$ down to $0.15$ (-16dB ambient floor) over 1.5s. Other birds remain subtly audible, preserving aviary co-presence.

### 8.4 WebAudio Fallback Architecture
If the WebAudio context fails to initialize or is blocked by browser policy:
1.  The audio engine enters `SILENT_CAPTIONED` mode without presenting any popups or error alerts.
2.  Visual call captions automatically activate in the UI.

---

## 9. Accessibility Surfaces & Naturalist Narration

### 9.1 Naturalist Screen-Reader Narration Stream
Screen readers are fed via an invisible `aria-live="polite"` region. Prose is formatted in naturalist, present-tense, lower-case text:

> *Current Narration:* "a warbler perches on the high branch, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."

*   **Cadence:** Updated every 45 seconds during idle, or immediately upon user actions (offer accepted, listen-in engaged).

### 9.2 Procedural Call Captions
Captions appear near calling birds in soft typography:
*   `"a soft three-note rise"`
*   `"a low trill, paused, low trill again"`

### 9.3 Keyboard Navigation Contract
*   `Tab`: Cycles top-bar controls (Settings, Accessibility, Notebook, Offers).
*   `Tab` into Scene: Focuses the front-most bird with a high-contrast focus ring (WCAG AA ratio 4.8:1).
*   `Left / Right Arrow`: Moves focus between birds ordered by perch depth and horizontal position.
*   `Enter`: Toggles listen-in on focused bird.
*   `Escape`: Disengages listen-in or closes modals.

---

## 10. Performance Budgets & Observability

### 10.1 Quantitative Performance Budgets
*   **Initial JS Bundle Size:** $\le 1.8\text{ MB}$ gzipped (enforced via webpack/vite bundle analyzer in CI).
*   **Time-to-First-Bird Visible:** $\le 450\text{ ms}$ on 4G connection / mid-tier mobile CPU (achieved via inline server snapshot injection).
*   **Runtime Frame Rate:** Steady 60fps idle motion on 5-year-old mid-range hardware (Intel HD Graphics 620).
*   **Memory Footprint:** Zero heap accumulation over 30 continuous minutes (verified by automated Memory Leak Chrome Driver CI tests).

### 10.2 Privacy-Preserving Observability
*   **Allowed Telemetry:** Edge request latency histograms, simulation tick runtime p99, bundle load time (RUM), WebAudio initialization error rates.
*   **Strictly Prohibited Telemetry:** Individual bird drift values, user interaction counts, listen-in durations per bird, or per-account behavioral profiles.

---

## 11. Rollout Strategy & Verification Plan

### 11.1 Release Phasing
1.  **Phase 1 (v1.0 Launch):** Release single-user accounts, 2 starter birds, full simulation engine, notebook, and optional visit links.
2.  **Aviary Population Ramp:** 
    *   Day 0: 2 starter birds.
    *   Day 30: Unlocks 3rd bird invitation in top bar.
    *   Day 90: Unlocks 4th bird.
    *   Day 180+: Slow unlocks up to 7 bird maximum cap.

### 11.2 Verification Matrix

| Subsystem | Automated Verification Test | Target Criterion |
|---|---|---|
| **Bundle Size** | CI `bundlesize` check | `< 1.8MB` gzipped |
| **First Paint** | Playwright WebPageTest on 4G | First bird rendered `< 500ms` |
| **Drift Math** | Unit test suite over 10,000 simulated ticks | Monotonic increase; numerical change detectable at Day 7, visible at Day 21 |
| **Sync Safety** | Multi-client concurrent write test | Zero last-write-wins overwrites; DB state matches server tick delta |
| **Accessibility** | Axe-core + VoiceOver playback script | 0 WCAG AA violations; prose narration matches spec voice |
| **Memory Leak** | 30-minute Playwright canvas loop | Memory delta `< 2MB` growth |

---

## 12. Risk Matrix & Mitigations

1.  **Risk: Drift Calibration Misalignment (Too Fast or Too Slow)**
    *   *Mitigation:* Tune discrete low-pass filter constants ($\eta$) against telemetry aggregate distributions without inspecting individual accounts.
2.  **Risk: WebAudio Context Autoplay Rejection**
    *   *Mitigation:* Soft user-gesture listener on first click anywhere in canvas resumes WebAudio context seamlessly; captions display automatically if context remains suspended.
3.  **Risk: Browser Clock Skew in Multi-Device Sync**
    *   *Mitigation:* Server timestamps anchor all snapshots; client interpolates delta relative to server time rather than local system clock.
4.  **Risk: Voice / Tone Drift into System Error Surfaces**
    *   *Mitigation:* Enforce automated string linter: Naturalist register reserved strictly for `.notebook`, `.narration`, and `.captions`; matter-of-fact register enforced for `.auth`, `.errors`, and `.settings`.
