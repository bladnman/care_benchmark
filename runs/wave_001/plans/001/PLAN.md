# Pocket Aviary — System Implementation Plan (v1)

This document provides the comprehensive engineering design and implementation plan for **Pocket Aviary**, a browser-based ambient virtual aviary. The design interprets and operationalizes all constraints, architectural contracts, affective principles, and non-goals defined in the product specification.

---

## 1. Scope & System Boundaries

### 1.1 In-Scope for v1
- **Platform**: Modern web browsers (latest two major versions of Chrome, Safari, Firefox, Edge).
- **Aviary Scene**: Single horizontal viewport-constrained scene with 3 perch zones (Front, Middle, Back), local-time day/night cycle, subtle ambient weather (rain, wind, falling leaves/feathers), and micro-motion.
- **Bird Population**: Starting pair (2 starter birds selected from a 6-species pool); capacity cap strictly at 7 birds. Additional bird adoption gated solely on aviary age.
- **Interactions**:
  - Passive presence tracking (tri-condition attention validator).
  - Procedural return-greeting (varied by bird boldness, mood, and absence duration).
  - Listen-in mix focusing on an individual bird.
  - Top-bar gesture offers: Seed, Song fragment, Still pool of water (per-bird cooldowns).
  - Settle gesture (evening lighting shift and call quieting with 5s cancel/undo affordance).
  - Field Notebook (auto-generated naturalist prose observations).
- **Accounts & Auth**: Single-user accounts authenticated via email magic links (15-minute validity, single-use).
- **Multi-Device State**: Single canonical server-simulated aviary state synced via snapshot distribution; device session management and revocation.
- **Quiet Social Visits**: One-time, revocable, read-only visitor access via magic link (no co-presence, no chat, no visitor drift impact).
- **Accessibility**: Naturalist screen-reader running narration, procedural call captions, reduced-motion cross-fade mode, full keyboard navigation, WCAG AA contrast.
- **Data Rights & Export**: JSON aviary snapshot export; 30-day soft deletion before permanent purge.

### 1.2 Explicit Non-Goals & Architectural Prohibitions
- **No Native Applications**: No iOS or Android native wrappers/builds.
- **No Gamification Elements**: Zero streak counters, visit tallies, green-dot activity grids, badges, achievements, levels, XP, or score counters.
- **No Tamagotchi / Custodial Mechanics**: No bird illness, hunger, death, distress animations, or negative drift on neglect.
- **No Social Network Features**: No public directory/feed, no follower model, no leaderboards, no mutual visit co-presence, no visitor comments/avatars.
- **No Announcement UI in Aviary**: No "Welcome back!" toasts, modal popups, or banner announcements.
- **No Numerical Trait Exposure**: Personality vector scalars are strictly hidden from UI, debug overlays, and telemetry.

---

## 2. Architecture & Service Topology

The architecture divides strictly between a stateless edge/API layer, a centralized authoritative simulation daemon, and a lightweight client-side procedural synthesis/render engine.

```
                  ┌──────────────────────────────────────────────────────────┐
                  │                   Browser Client                         │
                  │  ┌────────────────────┐   ┌───────────────────────────┐  │
                  │  │ 2D Canvas / WebGL  │   │ WebAudio Synthesis Engine │  │
                  │  │ (Interpolated Sim) │   │ (Procedural Motif Player) │  │
                  │  └──────────▲─────────┘   └─────────────▲─────────────┘  │
                  │             │                           │                │
                  │   ┌─────────┴───────────────────────────┴────────────┐   │
                  │   │      Client State & Presence Coordinator         │   │
                  │   └───────────────▲───────────────────────┬──────────┘   │
                  └───────────────────┼───────────────────────┼──────────────┘
                                      │ HTTP / SSE            │ HTTP POST
                           Snapshots  │                       │ Event Log
                                      │                       ▼
┌─────────────────────────────────────┴──────────────────────────────────────────────────────┐
│ Edge / API Gateway                                                                         │
│  - Magic Link Auth & Session Token Validation                                              │
│  - Synthetic UUID Translation (PII isolation)                                              │
│  - Snapshot Cache (Redis / Memory) & Rate Limiting                                         │
└─────────────────────────────────────┬──────────────────────────────────────────────────────┘
                                      │
              ┌───────────────────────┴───────────────────────┐
              ▼                                               ▼
┌───────────────────────────────┐               ┌───────────────────────────────┐
│ Primary Relational DB         │               │ Simulation Engine Worker      │
│ (PostgreSQL)                  │               │ (Authoritative Server Tick)   │
│ - accounts (encrypted email)  │◄──────────────┤ - Runs 60s discrete tick loop │
│ - aviaries & birds (UUIDs)    │ Snapshot Read │ - Consumes append-only events │
│ - personality_vectors         │ & State Write │ - Monotonic drift filter      │
│ - notebook_entries            │               │ - Mood transition Markov fsm  │
│ - append_only_event_log       │               │ - Auto-generates notebook log │
│ - visits_and_invitations      │               └───────────────────────────────┘
└───────────────────────────────┘
```

### 2.1 Component Boundaries
1. **Client Application**:
   - Downloads static bundle (<2MB gzipped).
   - Manages tri-condition presence detector (page visibility + window focus + user activity).
   - Submits append-only interaction events (`presence_ping`, `offer`, `listen_in_start`, `listen_in_end`, `settle`).
   - Renders scene smoothly by interpolating between server state snapshots.
   - Generates procedural audio motifs via WebAudio API based on bird species and state grammar.
2. **API Gateway & Ingress**:
   - Issues and verifies 15-minute cryptographic magic links.
   - Converts authenticated session tokens to synthetic internal Account UUIDs.
   - Serves cached aviary snapshots directly to reduce database read load.
   - Appends incoming interaction events into the transaction log.
3. **Authoritative Simulation Engine (Worker Pool)**:
   - Evaluates active aviaries on a discrete 60-second tick loop.
   - Aggregates presence pings and interactions into drift calculations.
   - Advances mood states and circadian lighting according to local timezone.
   - Synthesizes naturalist notebook entries when trigger conditions occur.
   - Writes immutable snapshots to Redis/PostgreSQL.
4. **Data & Telemetry Boundary**:
   - Internal DB keys are synthetic UUIDs. Email addresses are encrypted at rest with envelope encryption and never logged.
   - Aggregate operational telemetry (p99 tick latency, error rates, bundle load time) is decoupled from simulation tables. Per-bird vectors and interaction logs are completely blocked from analytics pipelines.

---

## 3. Data Model

### 3.1 Relational Schema (PostgreSQL DDL)

```sql
-- Synthetic Account Identity
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    encrypted_email BYTEA NOT NULL,
    email_hash VARCHAR(64) UNIQUE NOT NULL, -- Blind index for login lookup
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    soft_deleted_at TIMESTAMPTZ NULL,
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    notify_on_visit BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE TABLE auth_sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    session_token_hash VARCHAR(64) UNIQUE NOT NULL,
    user_agent TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_active_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    revoked_at TIMESTAMPTZ NULL
);

-- Aviary and Birds
CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID UNIQUE NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    settled_until TIMESTAMPTZ NULL,
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL,
    custom_name VARCHAR(48) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    current_mood VARCHAR(24) NOT NULL DEFAULT 'content',
    current_perch_zone VARCHAR(16) NOT NULL DEFAULT 'middle', -- 'front' | 'middle' | 'back'
    last_mood_update TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Hidden Personality Vectors (Server Canonical Only)
CREATE TABLE personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(id) ON DELETE CASCADE,
    boldness DOUBLE PRECISION NOT NULL DEFAULT 0.5,        -- [0.0, 1.0]
    social_warmth DOUBLE PRECISION NOT NULL DEFAULT 0.5,   -- [0.0, 1.0]
    vocal_frequency DOUBLE PRECISION NOT NULL DEFAULT 0.5, -- [0.0, 1.0]
    plumage_saturation DOUBLE PRECISION NOT NULL DEFAULT 0.5, -- [0.0, 1.0]
    curiosity DOUBLE PRECISION NOT NULL DEFAULT 0.5,       -- [0.0, 1.0]
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Append-Only Interaction & Presence Event Log
CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(id) ON DELETE SET NULL,
    event_type VARCHAR(32) NOT NULL, -- 'presence_ping', 'offer_seed', 'offer_song', 'offer_pool', 'listen_in_start', 'listen_in_end', 'settle'
    duration_seconds INTEGER NULL,
    payload JSONB NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_in_tick BOOLEAN NOT NULL DEFAULT FALSE
);
CREATE INDEX idx_events_aviary_unprocessed ON interaction_events(aviary_id) WHERE processed_in_tick = FALSE;

-- Field Notebook Observations
CREATE TABLE notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    observed_date DATE NOT NULL,
    prose_text TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_notebook_aviary_date ON notebook_entries(aviary_id, observed_date DESC);

-- Visit Invitations & Logs
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    token_hash VARCHAR(64) UNIQUE NOT NULL,
    invitee_email_encrypted BYTEA NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '30 days'),
    revoked_at TIMESTAMPTZ NULL
);

CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invitation_id UUID NOT NULL REFERENCES visit_invitations(id) ON DELETE CASCADE,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    visited_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    duration_seconds INTEGER NOT NULL DEFAULT 0
);
```

---

## 4. API Surface & Protocols

All communication uses standard HTTPS REST endpoints returning structured JSON.

### 4.1 Authentication
- `POST /api/v1/auth/magic-link`: Request sign-in link with `{ "email": "user@example.com" }`.
- `POST /api/v1/auth/verify`: Consume magic token, exchange for session cookie and CSRF token.
- `GET /api/v1/auth/sessions`: List active login sessions.
- `DELETE /api/v1/auth/sessions/:id`: Revoke a session.

### 4.2 Aviary State & Events
- `GET /api/v1/aviary/snapshot`:
  Returns the current canonical aviary state:
  ```json
  {
    "aviary_id": "8f03c401-4473-455b-8025-06bead71a391",
    "server_time": "2026-08-13T18:50:00Z",
    "timezone": "America/Los_Angeles",
    "settled": false,
    "weather": { "type": "clear", "intensity": 0.0 },
    "birds": [
      {
        "id": "e93f77df-9828-444a-89aa-c92ba73595f1",
        "species_id": "chickadee",
        "name": "pip",
        "mood": "content",
        "perch_zone": "front",
        "perch_slot": 2,
        "plumage_saturation_visual": 0.62,
        "idle_pattern": "preen_soft"
      },
      {
        "id": "1fa0de32-b7e1-456c-a511-fb457a4128f7",
        "species_id": "wren",
        "name": "wren",
        "mood": "curious",
        "perch_zone": "middle",
        "perch_slot": 1,
        "plumage_saturation_visual": 0.54,
        "idle_pattern": "head_tilt_listen"
      }
    ],
    "eligible_for_new_bird": false
  }
  ```
- `POST /api/v1/aviary/events`:
  Batched client event submission (validated against session credentials):
  ```json
  {
    "events": [
      { "type": "presence_ping", "duration_seconds": 60 },
      { "type": "offer", "bird_id": "e93f77df-9828-444a-89aa-c92ba73595f1", "offer_type": "seed" }
    ]
  }
  ```

### 4.3 Field Notebook
- `GET /api/v1/notebook?limit=20&cursor=...`: Paginated list of read-only observation logs.

### 4.4 Social Visits
- `POST /api/v1/visits/invite`: Host creates an invite for `visitor@example.com`.
- `GET /api/v1/visits/log`: View history of visits and active links.
- `DELETE /api/v1/visits/invite/:id`: Instantly revoke an outstanding invitation.
- `GET /api/v1/visit/:token/snapshot`: Read-only snapshot access for visitor browser. Rejects if revoked or expired.

---

## 5. Simulation Engine Design

### 5.1 Tick Loop & State Progression
The simulation runs asynchronously on the backend on a 60-second tick interval for each active aviary:
1. **Event Ingestion**: Pulls all unprocessed `interaction_events` since the last tick.
2. **Presence Validation**: Confirms `presence_ping` validity against rate-limit bounds (maximum 60s presence accrued per 60s real-world interval).
3. **Drift Function (Monotonic Low-Pass Filter)**:
   Personality vector updates apply an asymmetric leaky accumulator:
   $$\Delta v_i = \alpha_i \cdot (\text{PresenceWeight} \cdot T_{\text{presence}} + \text{InteractionWeight} \cdot I_{\text{events}})$$
   $$v_i(t) = \min\left(1.0, v_i(t-1) + \Delta v_i\right)$$
   - **Monotonicity Rule**: $\Delta v_i \ge 0$. Traits never decrement upon neglect.
   - **Calibration Target**: For daily 15-minute visits, measurable vector drift in telemetry after 7 days ($\Delta v \approx 0.05$), visible behavioral/visual drift after 21 days ($\Delta v \ge 0.15$).
4. **Mood State Machine & Circadian Modulation**:
   Moods $\in \{\text{wary}, \text{content}, \text{curious}, \text{drowsy}, \text{alert}\}$ transition via Markov transition probabilities conditioned on:
   - Time of day (dusk $\to$ drowsy, dawn $\to$ alert).
   - Recent accepted offers (nudge $\to$ content/curious).
   - Boldness trait (high boldness suppresses transitions into wary).
   - Weather events (rain increases dampening factor).
5. **Naturalist Notebook Generation**:
   At least once every 2–4 days, or when an anchor event happens (e.g., first greeting swap, weather passage, long quiet session), the engine selects a naturalist template matching the state:
   - Evaluated using a grammar engine configured with bird names and lower-case present-tense rules.
   - Preserves sparsity: will not emit entries if one was created in the last 48 hours unless a rare observation threshold is crossed.

---

## 6. Sync Model & Conflict Prevention

1. **Single Writer Principle**: The server-side simulation tick is the sole author of personality vectors, bird placements, and mood states.
2. **Delta-Based Event Ingestion**: Clients submit discrete events (`offer`, `listen_in`, `presence_ping`), never absolute state.
3. **Multi-Device Convergence**:
   - Laptop and mobile both read the same snapshot cache.
   - When a tab resumes or regains focus, it fetches `GET /api/v1/aviary/snapshot` and performs a smooth 1.5s interpolation from current client animation state to the canonical server state.
   - No client-to-client peer sync is attempted.
4. **Offline / Background Tab Handling**:
   - When a browser tab is hidden or minimized, client-side rendering halts to conserve battery and CPU.
   - Simulation on the backend continues at standard 60-second ticks.
   - Upon tab refocus, the client requests the fresh state snapshot, rendering the aviary as having continued uninterrupted.

---

## 7. Frontend Rendering Pipeline

### 7.1 Scene Construction
- **Canvas / 2D Context**: Rendered on an HTML5 `<canvas>` sized to the viewport, maintaining aspect ratio constraints to ensure all birds remain in frame across mobile and wide displays.
- **Three Depth Planes (Perch Zones)**:
  - *Back Plane*: Soft sky gradient, distant ambient trees, subtle leaf drift.
  - *Middle Plane*: Main branches/perches, bird rigs, water basin/seed tray.
  - *Front Plane*: Low rail/branch, gentle foreground foliage parallax.
- **Micro-Motion System**:
  - Procedural skeleton/rig for each species: body bob, tail wag, preening cycle, eye blink, and head-tilt angle.
  - Perch hops use cubic Bézier trajectory paths with velocity curves calculated to prevent teleports.
- **Instant Aliveness (Zero Entry Transition)**:
  - First frame renders immediately with birds placed mid-cycle according to snapshot state and timestamp hash.
  - No loading spinners or fade-ins. Cold network requests display a calm empty sky gradient before the starter birds gently glide to perches.

### 7.2 Reduced-Motion Mode
- Activates automatically via `@media (prefers-reduced-motion: reduce)` or manual accessibility toggle.
- Replaces skeletal continuous keyframes with gentle 1.2-second alpha cross-fades between static naturalist bird poses.
- Parallax and falling leaf/feather particle simulations are deactivated.
- Lighting transitions (day to dusk) transition via extended 10-second linear color fades.

---

## 8. Audio Pipeline & WebAudio Synthesis

### 8.1 Procedural Call Synthesizer
- **No Looped Audio Samples**: 100% synthesized client-side using `AudioContext` nodes to fit the bundle constraint (<2MB) and eliminate repetition.
- **Species Motif Grammars**:
  - Each species has a structural synthesizer graph (Carrier Oscillator $\to$ Formant Filter $\to$ Modulator Oscillator $\to$ Envelope Gain).
  - Calls are composed of 1–4 sequential pitch/frequency sweeps with micro-randomized jitter in frequency ($\pm 3\%$) and tempo ($\pm 5\%$).
- **Spatial Positioning**: Stereophonic panning and distance filtering corresponding to the bird's perch zone (Front = full stereo & dry; Back = centered & slightly low-pass filtered).

### 8.2 Chorus & Listen-In Mix Matrix
- **Ambient Chorus**: When multiple birds call in sequence, a dynamic gain bus prevents clipping and applies ducking.
- **Listen-In Focus**:
  - When the user focuses a bird, the focused bird's gain ramps to +3dB over 1.5s via `AudioParam.linearRampToValueAtTime`.
  - Unfocused birds ramp down to -14dB (remaining gently audible in ambient background; never muted).
  - Disengaging listen-in smoothly returns the audio mix to equal ambient balance over 2.0s.
- **WebAudio Fallback**: If `AudioContext` is blocked, unavailable, or permissions are denied, the audio engine switches gracefully to silence, automatically activating the call captions surface.

---

## 9. Accessibility Surfaces

1. **Screen-Reader Narration (`aria-live="polite"`)**:
   - Dedicated hidden live region delivering naturalist prose updates every 30–60 seconds:
     > *"a small grey bird perches on the front rail, calling softly. another bird sits on the high branch preening."*
   - Immediate narration for user gestures (e.g., offering a seed, entering listen-in, settling).
2. **Procedural Call Captioning**:
   - Floating subtitle tags rendered near calling birds using high-contrast text tags:
     > *"pip: a soft two-note rise"*
   - Synchronized precisely with the WebAudio envelope triggers.
3. **Keyboard Navigation & Focus Traps**:
   - `Tab` navigates through top-bar actions and into the aviary bird nodes.
   - `ArrowLeft` / `ArrowRight` cycles through visible birds.
   - `Enter` triggers Listen-In; `Escape` clears Listen-In.
   - High-contrast focus rings meet WCAG AA requirements across both daytime and night palettes.

---

## 10. Performance Budgets & Observability

### 10.1 Budgets
- **Initial JS Bundle**: $\le 1.8 \text{MB}$ uncompressed, $\le 450 \text{KB}$ gzipped.
- **Time to First Bird Visible**: $\le 400 \text{ms}$ on 4G connection from edge CDN.
- **Frame Rate**: Steady 60fps on a 2021 mid-tier laptop; 30fps throttle on low-power mobile mode.
- **Memory Footprint**: Flat memory profile over 30 minutes. All WebAudio buffer nodes and canvas frame buffers are recycled; zero per-frame garbage collector pressure.

### 10.2 Observability & Privacy Protection
- **Operational Metrics**:
  - Simulation tick duration (p95, p99; alert if p99 $> 5.0\text{s}$).
  - API endpoint latency and error rates (5xx / 4xx).
  - Synthetic client lighthouse performance probes (LCP, INP, CLS).
- **Privacy Enforcement**:
  - Analytics pipeline explicitly forbids tracking bird names, personality vectors, offer choices, or notebook logs.
  - Session duration histograms are stored in bucketed anonymized intervals.

---

## 11. Rollout & Milestone Plan

| Phase | Milestone | Deliverables |
|---|---|---|
| **Phase 1** | **Core Simulation & DB Engine** | Relational schema, 60s tick daemon, presence validator, monotonic drift filter, magic-link auth. |
| **Phase 2** | **Procedural Audio & WebAudio Synthesis** | 6 species motif graphs, dynamic chorus mixer, listen-in audio ramp, caption generator. |
| **Phase 3** | **Canvas Rendering & Micro-Motion** | 3 perch zones, skeletal bird rigs, day/night lighting shader, reduced-motion cross-fader. |
| **Phase 4** | **Interactions, Notebook & Gestures** | Return-greeting procedural engine, offer tray, settle flow, naturalist notebook logger. |
| **Phase 5** | **Social Visits & Privacy Layer** | Tokenized read-only visit flow, visit revocation, account export, GDPR/30-day soft purge. |
| **Phase 6** | **Accessibility, Perf Audit & Launch** | Screen-reader prose validation, WCAG AA compliance, 30-minute memory leak stress test, edge CDN cache warmup. |

---

## 12. Risks & Mitigations

1. **Drift Calibration Imbalance**:
   - *Risk*: Personality traits drift too quickly (feeling like a toy) or too slowly (feeling static).
   - *Mitigation*: Comprehensive simulation unit tests verifying drift curves across simulated 7-day and 30-day usage profiles before production release.
2. **Audio Uncanniness / Phase Cancellation**:
   - *Risk*: Multiple procedural calls layering improperly or sounding artificial.
   - *Mitigation*: Formant filter modulation with organic randomized pitch offsets and chorus ducking bus.
3. **Presence Signal Spoofing / Background Tab Drift**:
   - *Risk*: Unattended open tabs inflating presence-time.
   - *Mitigation*: Strict client-side conjunction check (`document.visibilityState === 'visible'` AND `document.hasFocus()` AND user input within last 3 minutes) emitting signed presence pings.
4. **Screen-Reader Queue Flooding**:
   - *Risk*: Frequent state updates clobbering screen-reader queues.
   - *Mitigation*: Enforce a strict 30-second throttle on idle narrations and utilize `aria-live="polite"`.
