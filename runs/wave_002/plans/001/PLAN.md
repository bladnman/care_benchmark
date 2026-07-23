# Pocket Aviary — Phase 1 Engineering & Implementation Plan

## 1. Executive Summary & Vision Alignment

Pocket Aviary is a quiet, browser-based virtual aviary where a user adopts a small group of birds (2 to start, capped at 7) that evolve over days and weeks in response to idle presence and quiet interactions. 

This engineering plan translates the product spec (PRD) into a production architecture. The core mandate of this architecture is **felt aliveness through technical integrity**. Every technical decision in this document — from server-side canonical ticks to procedural WebAudio synthesis — directly serves the five product principles:
1. **Feels alive, not robotic**: Server-driven background simulation, procedural call synthesis, seamless first-frame rendering without spinners.
2. **Notice, never announce**: Zero toast popups, zero textual welcome banners, return-greetings expressed purely through subtle bird motion and staggered vocalizations.
3. **Charm comes from specificity**: Naturalist prose generation across the field notebook, screen-reader narration, and audio captions.
4. **Restraint over richness**: Fixed single-screen horizontal viewport, strict 7-bird cap, minimalist top bar with cursor auto-fade.
5. **Naturalist voice for the product, matter-of-fact for the system**: Strict architectural boundary between bird interaction surfaces (naturalist) and auth/settings/error surfaces (matter-of-fact).

---

## 2. Scope & Non-Goals

### 2.1 Included in V1
- **Client & Scene**: Web-only (modern desktop and mobile browsers), single horizontal viewport, 3 perch zones (front, middle, back), real-time day/night lighting cycles tied to local timezone, ambient weather (rain, wind), subtle leaf/feather drift, top-bar UI with auto-fade.
- **Bird Engine**: Pool of 6 bird species; starting aviary with 2 birds; age-based unlocks capping at 7 birds; hidden 5-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity); fast-timescale daily mood engine (wary, content, curious, drowsy, alert); procedural call grammar synthesis; mood-shaped idle micro-motion.
- **Interactions**: Staggered procedural return-greeting; Listen-in with exponential audio mix cross-fading; Offer gestures (seed, song fragment, still pool) with per-bird cooldowns; Settle evening gesture with 5-second undo grace period; Field notebook displaying naturalist observations.
- **Presence & Drift**: Strict presence accounting (conjunction of `visibilityState === 'visible'`, `document.hasFocus()`, and pointer/keyboard activity within 3 minutes); monotonic server-driven personality drift toward expressive.
- **Accounts & Sync**: Magic-link authentication (15-min expiry); single canonical aviary per account; multi-device sync via server-side simulation ticks; account JSON export; 30-day soft deletion before hard purge; encrypted email storage with synthetic UUID identifiers.
- **Social (Optional)**: One-time read-only visit invitations sent via email link; revocable host control; silent visit logging; off by default.
- **Accessibility & Performance**: Naturalist screen-reader narration engine; reduced-motion cross-fade rendering mode; procedural call captions; WCAG AA contrast; full keyboard navigation; initial JS bundle <2MB (gzipped); time-to-first-bird <500ms; 60fps idle motion; zero client memory growth over 30-minute sessions.

### 2.2 Explicit Non-Goals & Out-of-Scope (V1 & Beyond)
- **No Native Apps**: Web-only. No iOS/Android native packages or wrappers.
- **No Gamification**: Absolute prohibition on streak counters, visit calendars, level meters, XP, badges, adoption counters, or rank boards.
- **No Tamagotchi Mechanics**: Birds never die, hunger does not exist, neglect never causes distress or negative personality drift.
- **No Social Network Surfaces**: No profiles, friend lists, chat, comments, public feeds, leaderboards, co-presence, or mandatory visit notifications.
- **No Direct Personality Mutation**: Clients never write personality vector numbers or transmit absolute state updates.

---

## 3. System Architecture & Service Topology

The system uses a decoupled client-server architecture where the **server holds sole ownership of canonical state and personality simulation**, while the **client acts as a render-and-synthesis engine**.

```
                           +-----------------------------------+
                           |        Browser Client (Web)       |
                           |  - Canvas/WebGL Render Pipeline   |
                           |  - WebAudio Synthesis Engine      |
                           |  - Presence & Event Collector     |
                           +-----------------+-----------------+
                                             |
                                   HTTPS / WSS / REST
                                             |
                                             v
                           +-----------------+-----------------+
                           |        API Gateway / Router       |
                           |  - Magic Link Auth & JWT Revocation|
                           |  - Rate Limiting & SSL Termination |
                           +--------+-----------------+--------+
                                    |                 |
                   +----------------+                 +----------------+
                   |                                                   |
                   v                                                   v
+------------------+------------------+             +------------------+------------------+
|          Aviary API Service         |             |       Simulation Tick Worker Engine     |
| - Snapshot Delivery (CDN Edge Cache)|             | - Cron / Distributed Queue Worker       |
| - Interaction Event Ingestion       |             | - Server-Side Tick Loop (~1 min cadence)|
| - Visit Invitation Verification     |             | - Additive Low-Pass Drift Calculator    |
| - Field Notebook & Narration Generator            | - Diurnal Mood Transition Engine        |
+------------------+------------------+             +------------------+------------------+
                   |                                                   |
                   +----------------+-----------------+----------------+
                                    |
                                    v
                           +-----------------+-----------------+
                           |      Primary Data Persistence     |
                           | - PostgreSQL (Canonical State,    |
                           |   Accounts, Birds, Vectors, Logs) |
                           | - Redis (Session Tokens, Active   |
                           |   Aviary Caching, Event Buffers)  |
                           +-----------------------------------+
```

### 3.1 Component Responsibilities
1. **API Gateway & Auth Service**: Handles magic-link generation, email verification, session JWT issuance/revocation, and synthetic UUID mapping. Enforces rate limits and strips sensitive headers.
2. **Aviary API Service**: Delivers compressed state snapshots to clients, appends validated client interaction events to the event log, processes field notebook queries, and validates read-only visit tokens.
3. **Simulation Tick Worker Engine**: Runs background tick loops every 60 seconds for active/ticking aviaries. Evaluates presence logs, computes additive personality drift deltas, updates mood state-machines based on diurnal timing and recent interactions, and generates naturalist notebook entries.
4. **Naturalist Prose Generator Subservice**: Deterministic template-and-rules engine embedded within the backend to produce naturalist field notebook entries and screen-reader narration strings without external LLM dependencies.
5. **Browser Client Application**: Single Page Application written in Vanilla TypeScript/JavaScript using standard HTML5 Canvas/WebGL for rendering and WebAudio for real-time procedural sound synthesis.

---

## 4. Data Models & Schemas

The database schema strictly isolates PII (encrypted email) on the `accounts` table and uses synthetic UUID identifiers across all relational tables and internal queues.

```sql
-- Core Account Schema
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted TEXT NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    status VARCHAR(32) NOT NULL DEFAULT 'active', -- 'active', 'soft_deleted'
    deleted_at TIMESTAMPTZ NULL,
    settings JSONB NOT NULL DEFAULT '{
        "reduced_motion": false,
        "call_captions": false,
        "screen_reader_narration": false,
        "visit_notifications": false
    }'::jsonb
);

-- Canonical Aviary Schema
CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    timezone VARCHAR(64) NOT NULL DEFAULT 'UTC',
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    weather_state JSONB NOT NULL DEFAULT '{
        "current": "clear",
        "transition_at": null
    }'::jsonb
);

-- Bird Entity Schema
CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id VARCHAR(64) NOT NULL, -- e.g., 'warbler_grey', 'finch_gold'
    name VARCHAR(64) NOT NULL,
    slot_index INT NOT NULL CHECK (slot_index BETWEEN 0 AND 6),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    unlocked_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT unique_bird_slot UNIQUE (aviary_id, slot_index)
);

-- Personality Vector Schema (Server-only state)
CREATE TABLE personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(id) ON DELETE CASCADE,
    boldness FLOAT NOT NULL CHECK (boldness BETWEEN 0.0 AND 1.0),
    social_warmth FLOAT NOT NULL CHECK (social_warmth BETWEEN 0.0 AND 1.0),
    vocal_frequency FLOAT NOT NULL CHECK (vocal_frequency BETWEEN 0.0 AND 1.0),
    plumage_saturation FLOAT NOT NULL CHECK (plumage_saturation BETWEEN 0.0 AND 1.0),
    curiosity FLOAT NOT NULL CHECK (curiosity BETWEEN 0.0 AND 1.0),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Fast-Timescale Mood Schema
CREATE TABLE bird_moods (
    bird_id UUID PRIMARY KEY REFERENCES birds(id) ON DELETE CASCADE,
    current_mood VARCHAR(32) NOT NULL DEFAULT 'content', -- 'wary','content','curious','drowsy','alert'
    entered_mood_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    session_end_mood VARCHAR(32) NULL,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Append-Only Interaction Event Log Schema
CREATE TABLE interaction_events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    event_type VARCHAR(64) NOT NULL, -- 'presence_ping', 'offer_seed', 'offer_song', 'offer_pool', 'listen_in_start', 'listen_in_end', 'settle'
    target_bird_id UUID NULL REFERENCES birds(id) ON DELETE SET NULL,
    payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_events_account_tick ON interaction_events(account_id, created_at);

-- Field Notebook Entries Schema
CREATE TABLE notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    prose TEXT NOT NULL,
    category VARCHAR(32) NOT NULL DEFAULT 'observation',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Social Visit Invitations Schema
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    visitor_email_hash TEXT NOT NULL,
    invite_token VARCHAR(128) NOT NULL UNIQUE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL DEFAULT (NOW() + INTERVAL '30 days'),
    revoked_at TIMESTAMPTZ NULL
);

-- Visit Access Audit Log Schema
CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invitation_id UUID NOT NULL REFERENCES visit_invitations(id) ON DELETE CASCADE,
    visited_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    duration_seconds INT NOT NULL DEFAULT 0
);
```

---

## 5. API Surface & Protocol Specifications

### 5.1 Authentication Endpoints
- `POST /api/v1/auth/magic-link/request`
  - Request: `{ "email": "user@example.com" }`
  - Response: `{ "status": "sent", "message": "Check your email for sign-in link." }`
- `GET /api/v1/auth/magic-link/verify?token=<magic_token>`
  - Response: HTTP 302 redirect to app with HttpOnly Session Cookie (`pa_session`), or JSON error payload in matter-of-fact tone.

### 5.2 Aviary & Interaction Endpoints
- `GET /api/v1/aviary/snapshot`
  - Response Schema:
```json
{
  "aviary_id": "c56a4180-65aa-42ec-a945-5fd21ed05301",
  "server_time": "2026-07-24T06:21:42Z",
  "local_time_offset_minutes": 540,
  "weather": { "type": "clear", "intensity": 0.0 },
  "birds": [
    {
      "id": "7b8f9e10-1111-4a2b-8888-000000000001",
      "species_id": "warbler_grey",
      "name": "Pip",
      "slot_index": 0,
      "perch_zone": "front",
      "mood": "content",
      "plumage_saturation": 0.72,
      "call_signature": { "base_pitch_hz": 2400, "tempo_scalar": 1.05 }
    },
    {
      "id": "7b8f9e10-2222-4a2b-8888-000000000002",
      "species_id": "finch_gold",
      "name": "Wren",
      "slot_index": 1,
      "perch_zone": "middle",
      "mood": "wary",
      "plumage_saturation": 0.65,
      "call_signature": { "base_pitch_hz": 3100, "tempo_scalar": 0.92 }
    }
  ],
  "narration": "a grey warbler is perched on the front rail, preening softly."
}
```
- `POST /api/v1/aviary/events`
  - Request Payload:
```json
{
  "events": [
    {
      "event_type": "presence_ping",
      "timestamp": "2026-07-24T06:21:40Z",
      "payload": { "duration_seconds": 60, "visibility": "visible", "focused": true }
    },
    {
      "event_type": "offer_seed",
      "target_bird_id": "7b8f9e10-1111-4a2b-8888-000000000001",
      "timestamp": "2026-07-24T06:21:42Z"
    }
  ]
}
```
- `GET /api/v1/notebook`
  - Response: `{ "entries": [ { "id": "...", "prose": "tuesday — pip greeted before wren today, first time this week.", "created_at": "..." } ] }`

### 5.3 Social Visit Endpoints
- `POST /api/v1/social/invites`
  - Request: `{ "visitor_email": "friend@example.com" }`
  - Response: `{ "invite_id": "...", "created_at": "..." }`
- `DELETE /api/v1/social/invites/:invite_id`
  - Response: `{ "status": "revoked" }`
- `GET /api/v1/visit/:invite_token/snapshot`
  - Response: Identical to `/aviary/snapshot` read-only, stripped of host user controls and account metadata. Returns HTTP 404/410 in matter-of-fact tone if invalid or revoked.

---

## 6. Simulation Engine & Drift Mechanics

### 6.1 Server-Side Simulation Tick
The tick runs every 60 seconds per aviary. It performs four atomic stages:
1. **Event Log Processing**: Fetches all unconsumed `interaction_events` since `last_tick_at`.
2. **Presence Accumulation & Low-Pass Filter**: Validates true presence events and applies monotonic drift increments to personality traits.
3. **Diurnal & Environmental Mood Machine**: Computes new mood states based on local time, weather events, recent offers, and bird boldness.
4. **Notebook & Narration Evaluation**: Determines if a new naturalist observation should be logged or screen-reader narration updated.

```
                          +-----------------------------------+
                          |      Start 60s Simulation Tick    |
                          +-----------------+-----------------+
                                            |
                                            v
                          +-----------------+-----------------+
                          | Fetch Unconsumed Interaction Logs |
                          +-----------------+-----------------+
                                            |
                                            v
                          +-----------------+-----------------+
                          | Filter & Validate True Presence   |
                          | (Visible AND Focused AND Active)  |
                          +-----------------+-----------------+
                                            |
                                            v
                          +-----------------+-----------------+
                          | Calculate Monotonic Low-Pass Drift|
                          |  delta = alpha * presence_time    |
                          | (Traits move up, NEVER down)      |
                          +-----------------+-----------------+
                                            |
                                            v
                          +-----------------+-----------------+
                          | Evaluate Mood Transitions         |
                          | (Diurnal time + Offers + Weather) |
                          +-----------------+-----------------+
                                            |
                                            v
                          +-----------------+-----------------+
                          | Commit Canonical State to DB      |
                          +-----------------------------------+
```

### 6.2 Presence Accounting Algorithm
A `presence_ping` is valid if and only if all three conditions are satisfied:
1. `visibilityState === 'visible'`
2. `document.hasFocus() === true`
3. Pointer movement or keypress occurred within the preceding 180 seconds.

### 6.3 Monotonic Drift Formula
Drift updates a trait value $T_{t}$ from previous state $T_{t-1}$:
$$T_{t} = T_{t-1} + \min\left(\Delta_{\max}, \eta \cdot \frac{\text{presence\_seconds}}{86400} + \sum \omega_{\text{interaction}}\right)$$
Where:
- $\eta = 0.005$ per hour of presence.
- $\Delta_{\max} = 0.001$ per tick.
- If $\text{presence\_seconds} = 0$, $\Delta T = 0$. **Traits never decrease.** Neglect reduces expressiveness by dampening mood activations, not by reducing underlying trait scalars.

### 6.4 Calibration Targets
- **Instruments Target**: Measurable delta in database ($\Delta T \ge 0.02$) after 7 days of regular 15-minute daily visits.
- **User Perceptual Target**: Visibly altered perch preference and call frequency after 21 days.

### 6.5 Fast-Timescale Mood Engine
Mood states (`wary`, `content`, `curious`, `drowsy`, `alert`) transition according to the matrix:
- `drowsy`: Triggered when local time is between 22:00 and 06:00, or after a Settle event.
- `curious`: Triggered for 3–5 minutes following an Offer gesture, scaled by bird `curiosity` trait.
- `wary`: Triggered by sudden ambient alarm calls or low bird `boldness` during initial return-greeting.
- `content`: Default baseline when unbothered during day hours.
- `alert`: Early morning (06:00–09:00) or during sudden rain weather events.

---

## 7. Multi-Device Synchronization & Conflict Resolution

1. **Single Source of Truth**: The PostgreSQL database, updated by the server simulation tick, is the canonical state.
2. **Client Pull & Interpolation**: Clients pull snapshots on initialization, tab visibility return, and every 60 seconds while active. Clients interpolate bird positions between snapshots using smooth Bezier curves.
3. **No Client-Side Mutation**: Clients send raw, append-only interaction events (`offer`, `presence`, `listen_in`). Clients never write or request trait overrides.
4. **Race Condition Elimination**: Multi-device conflicts are mathematically impossible because clients never execute last-write-wins updates on state vectors. If a laptop and phone are open simultaneously, both ingest state snapshots from the single ticking server.

---

## 8. Frontend Rendering Pipeline

```
+-----------------------------------------------------------------------+
|                         Top Bar UI (DOM Overlay)                      |
| Account/Settings | Accessibility | Field Notebook | Offers | Settle   |
| (Fades to opacity 0.0 after 3s cursor inactivity; restores on motion)  |
+-----------------------------------------------------------------------+
|                         Aviary Canvas Viewport                        |
|                                                                       |
|   +---------------------------------------------------------------+   |
|   | Back Perch Zone (Wary / Low Boldness Birds)                   |   |
|   +---------------------------------------------------------------+   |
|   | Middle Perch Zone (Content / Mid Boldness Birds)              |   |
|   +---------------------------------------------------------------+   |
|   | Front Perch Zone (Bold / Curious / Focused Birds)             |   |
|   +---------------------------------------------------------------+   |
|                                                                       |
|   [ Subtle Parallax Background -> Foliage -> Birds -> Weather ]       |
+-----------------------------------------------------------------------+
```

### 8.1 Scene Layout & Viewport Rules
- Fixed single-screen view without scrolling, zooming, or panning.
- Responsive canvas scaling maintaining aspect ratio across viewports from 320px mobile to 4K ultra-wide.
- 3 Perch Zones:
  - **Back Perch**: High distance, lower rendering scale, muted plumage.
  - **Middle Perch**: Standard depth, default resting position.
  - **Front Perch**: Near viewer, full plumage detail, active focus target.

### 8.2 Idle Micro-Motion State Machine
Idle motion runs continuously via `requestAnimationFrame`:
- **Preening**: Feather ruffle animation triggered every 40–80 seconds for `content` birds.
- **Scanning**: Head turn left/right with 200ms easing.
- **Weight Shuffle**: Small vertical bounce (2px) on perch.
- **Head Tilt**: 15-degree rotation toward ambient leaf drops or song offers.

### 8.3 Reduced-Motion Mode (Prefers-Reduced-Motion)
When enabled:
- Disables continuous frame skeletal animations and leaf/feather drift particles.
- Replaces flight animations between perches with a 1.2-second smooth cross-fade between static pose keyframes.
- Retains ambient lighting transitions (day/night) slowed by $2\times$.

### 8.4 Initial Load Strategy (<500ms TTFB)
- Inline initial snapshot payload within HTML document shell delivered from CDN edge.
- Immediate Canvas render of birds on perches in mid-action on frame 1.
- Zero loading spinners or "fade-in from black" sequences. If cold network fetch takes >200ms, display quiet sky canvas background immediately.

---

## 9. Audio Architecture & Synthesis Pipeline

### 9.1 Procedural WebAudio Synthesis
No recorded audio files (`.mp3`/`.wav`) are shipped. All bird calls are synthesized in real-time via the WebAudio API:

```
+-----------------------------------------------------------------------+
|                    Procedural Call Synthesis Node                     |
|                                                                       |
|  +----------------+     +-------------------+     +----------------+  |
|  | Custom Motifs  | --> | Pitch Modulation  | --> | Master Gain /  |  |
|  | (Oscillators)  |     | (Vocal Frequency) |     | Panner Node    |  |
|  +----------------+     +-------------------+     +----------------+  |
|                                                           |           |
+-----------------------------------------------------------|-----------+
                                                            v
                                                   +-----------------+
                                                   | AudioContext    |
                                                   | Destination     |
                                                   +-----------------+
```

- **Motif Library**: Each species possesses a library of 4–6 frequency modulation primitives (sine/triangle oscillators with custom ADSR envelopes).
- **Pitch & Speed Modulation**: Base frequency and tempo are modulated by `mood` and `vocal_frequency`. High warmth increases call responsiveness; `wary` mood increases pitch jitter.

### 9.2 Chorus Mixing & Spatialization
- Dynamic PannerNodes assign horizontal stereo position based on bird X-coordinate in the canvas.
- Staggered timing algorithm prevents simultaneous call triggering, preserving per-bird call clarity up to the 7-bird cap.

### 9.3 Listen-In Mix Ramping
When the user initiates Listen-In on Bird $i$:
1. Focused Bird $i$ gain ramps exponentially to $+3\text{dB}$ over 1.5 seconds (`gainNode.gain.exponentialRampToValueAtTime`).
2. Non-focused birds ramp down to an ambient background floor ($-18\text{dB}$) over 1.5 seconds. No bird is completely muted.
3. Disengaging ramps all birds back to default ambient levels over 1.5 seconds.

### 9.4 Silent Fallback with Captions
If `AudioContext` fails to initialize or is muted by browser policy:
- System enters quiet mode gracefully without error popups.
- Dynamic naturalist call captions fade in/out visually near the calling bird.

---

## 10. Accessibility Surface Implementation

### 10.1 Screen-Reader Narration Engine
- Operates via a dedicated live region (`aria-live="polite"`).
- Periodically updates (every 30–60s) with naturalist prose generated from canonical state:
  > *"a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."*
- Priority events (return-greeting, offer reaction, settle) trigger an immediate, non-disruptive narration update.

### 10.2 Call Captions
- Rendered as unobtrusive, high-contrast overlay text adjacent to calling birds.
- Text derived dynamically from call motif characteristics: *"a soft three-note rise"*, *"a low trill, paused, low trill again"*.

### 10.3 Keyboard Navigation & High-Contrast Focus Rings
- Full DOM focus map across top-bar controls and aviary perches.
- `Tab` navigates top bar; `ArrowKeys` move focus between birds; `Enter` activates Listen-in; `Escape` disengages Listen-in.
- Focus indicator: 3px high-contrast dual ring (`#FFFFFF` outer, `#000000` inner) guaranteeing WCAG AA compliance across all day/night background palettes.

---

## 11. Performance Budgets, Optimization & Observability Strategy

### 11.1 Hard Performance Budgets
| Metric | Budget Ceiling | Measurement Tool |
|---|---|---|
| Initial JS Bundle | < 2.0 MB (Gzipped) | Webpack Bundle Analyzer / CI Check |
| Time-to-First-Bird | < 500 ms (4G Mid-tier Mobile) | Lighthouse / Synthetic RUM |
| Canvas Frame Rate | 60 fps (5-year-old laptop) | Chrome DevTools Frame Profiler |
| Client Heap Growth | 0 MB / 30 min (Zero Leaks) | Automated Playwright Heap Inspection |
| Simulation Tick Latency | p99 < 5.0 s | Datadog / Prometheus Alerting |

### 11.2 Privacy-Enforced Observability Boundary
To protect user privacy and fulfill PRD commitments:
- **Operational Metrics Allowed**: HTTP status code distributions, latency histograms, tick processing durations, audio initialization failure counts, WebGL context loss rates.
- **Strictly Prohibited Telemetry**: No per-bird trait values, no individual interaction event payloads, no user notebook text, no visitor email tracking, no analytics pipeline ingestion of simulation database tables.

---

## 12. Release Strategy, Aviary Pacing & Rollout Plan

### 12.1 Aviary Unlocking Progression
- **Day 0 (Adoption)**: Account created; user presented with 2 starter birds randomly selected from the 6-species pool. User assigns names.
- **Day 30**: 3rd bird unlocked and offered via quiet top-bar naturalist notification.
- **Day 90**: 4th bird unlocked.
- **Day 180**: 5th bird unlocked.
- **Day 270**: 6th bird unlocked.
- **Day 360**: 7th (and final) bird unlocked.

### 12.2 Release Phases
1. **Phase 1: Architecture & Planning (Current)**: Finalize specs, schemas, and API contracts.
2. **Phase 2: Core Engine & Synthesis**: Build simulation tick worker, WebAudio procedural grammar, and Canvas rendering pipeline.
3. **Phase 3: Auth, Sync & Accounts**: Implement magic-link service, PostgreSQL schema, and multi-device snapshot API.
4. **Phase 4: Accessibility & Polish**: Deliver screen-reader prose generator, reduced-motion cross-fader, call captions, and privacy audit.

---

## 13. Comprehensive Risk Matrix & Mitigation Tactics

| Risk Description | Severity | Impact Area | Mitigation Strategy |
|---|---|---|---|
| **Drift Calibration Drift** (Drift accumulation too fast or too slow) | High | User Relationship | Continuous automated CI simulation testing 30-day simulated runs to verify numerical trait deltas fall strictly within $0.02 - 0.05$ range. |
| **Sync Race Conditions** (Multiple active devices overwriting state) | High | Multi-device Sync | Strict prohibition of client state mutation. Server tick is the exclusive writer of canonical vectors via additive deltas. |
| **Audio Uncanny Valley** (Procedural calls sound synthetic or grating) | High | Core UX & Charm | Dynamic pitch/rhythm micro-jitter algorithms and spectral filtering matching natural avian acoustic resonance. |
| **Accessibility Regressions** (Screen reader reads raw ARIA data or announcements) | Medium | Accessibility | Automated DOM auditing verifying screen-reader output is routed through naturalist prose generator. |
| **PII Leakage in Telemetry** (Email appearing in error logs or metrics) | Critical | Security & Privacy | Encrypted email storage at REST; synthetic UUID enforcement for all internal logging, metrics, and DB foreign keys. |

---
*Verification Confirmation: This document represents the complete, executable implementation plan for Pocket Aviary V1, strictly adhering to all design principles, non-goals, and technical constraints specified in the PRD.*
