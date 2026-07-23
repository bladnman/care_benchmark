# Pocket Aviary — Phase 1 Engineering & Implementation Plan

This implementation plan defines the system architecture, component design, data models, API contracts, sync protocols, audio/visual pipelines, accessibility patterns, performance budgets, rollout strategy, and risk mitigations for **Pocket Aviary v1**.

---

## 1. Scope & System Boundaries

### 1.1 In-Scope for V1
- **Target Platform**: Modern desktop and mobile web browsers (Chrome, Safari, Firefox, Edge - last 2 major versions).
- **Core Experience**: Single-screen, non-panning horizontal aviary scene with continuous time-of-day/ambient weather rendering.
- **Bird Engine & Population**: 
  - Starter population: 2 birds; capacity capped strictly at 7 birds.
  - Progressive adoption unlocked via aviary age milestones.
  - 6 initial bird species in pool with unique visual silhouettes, color palettes, and procedural call motif libraries.
  - User-assignable and renameable bird names with persistent internal UUIDs.
- **Interactions**:
  - Presence tracking (strict multi-signal conjunction).
  - Return-greeting (procedurally staggered, personality/absence-weighted).
  - Listen-in (gradual WebAudio re-balance/solo focus).
  - Offers (seed, song fragment, still pool) with per-bird cooldowns.
  - Settle (soft evening shift session end gesture with 5s undo window).
  - Field Notebook (naturalist, prose-based observation log generated server-side).
- **Accounts & Authentication**:
  - Email magic-link auth (15-minute link TTL, single-use).
  - Synthetic UUID account keying (PII isolation).
  - Multi-device sync driven by a single canonical server-side simulation tick (~1 min).
  - Soft deletion (30-day window) & full JSON account export.
- **Social (Optional & Quiet)**:
  - Opt-in, host-initiated read-only ambient visits via one-time email invite link (30-day expiry, immediate revocation).
  - Private visit log in settings.
- **Accessibility & Performance**:
  - Screen-reader narration (slow naturalist prose stream via ARIA live region).
  - Designed reduced-motion mode (pose cross-fades instead of skeletal animation).
  - Procedural call captioning (naturalist text snippets).
  - WebAudio procedural synthesis with silent-caption fallback.
  - First-paint bundle < 2MB (gzipped), time-to-first-bird < 500ms over 4G, 60fps render on 5-year-old hardware.

### 1.2 Non-Goals & Explicit Exclusions (V1 and Beyond)
- **No Gamification**: No streaks, green-dot calendars, badges, XP, levels, scores, or visit-count displays.
- **No Tamagotchi Mechanics**: No bird death, hunger, sickness, distress meters, or negative personality drift on neglect.
- **No Social Network Features**: No public feeds, discovery/explore tabs, leaderboards, user profiles, comments, or co-presence/shared cursors.
- **No Native Apps**: Web-only; no iOS/Android native wrappers or platform-specific builds in v1.
- **No Client State Ownership**: No client-side simulation authority; clients never mutate personality vectors or compute canonical drift.

---

## 2. Architecture & Service Topology

The system uses a decoupled client-server architecture with an edge delivery layer, an event-driven ingestion pipeline, a stateful tick engine, and a read-optimized snapshot service.

```
┌─────────────────────────────────────────────────────────────┐
│                 Client Browser Application                  │
│  (React/TypeScript + Canvas/WebGL + WebAudio Synthesizer)   │
└──────────────┬──────────────────────────────▲───────────────┘
               │ Event Streams                │ State Snapshots & SSE
               │ (Presence, Offers, etc.)     │ (JSON Snapshots)
               ▼                              │
┌─────────────────────────────┐  ┌────────────┴────────────────┐
│      Edge API Gateway       │  │    Snapshot Read Service     │
│   (Auth, Rate Limiting)     │  │     (Edge CDN + Redis)       │
└──────────────┬──────────────┘  └────────────▲────────────────┘
               │                              │
               ▼                              │ Cache Invalidation / Writes
┌─────────────────────────────┐               │
│  Interaction Event Log      │               │
│  (Append-Only / Kafka-Pulsar)               │
└──────────────┬──────────────┘               │
               │ Consumption                  │
               ▼                              │
┌─────────────────────────────────────────────┴────────────────┐
│              Server-Side Simulation Engine                   │
│   - Cron/Worker Cluster running slow tick (~1 min)          │
│   - Evaluates presence-time & interaction logs               │
│   - Executes Low-Pass Drift Filter & Mood Transitions        │
│   - Synthesizes Field Notebook prose entries                 │
│   - Persists state to Canonical PostgreSQL Cluster           │
└──────────────────────────────────────────────────────────────┘
```

### 2.1 Services Breakdown
1. **Edge Gateway (Stateless Node.js/Go)**:
   - Handles magic-link authentication, session token issuance/validation, and CORS.
   - Validates incoming client interaction events (schema, rate limits, token scopes).
   - Appends valid events to the internal message broker.
2. **Simulation Engine (Background Worker Cluster)**:
   - Runs a periodic cron-based evaluation tick (e.g., every 60 seconds per active/ticking aviary).
   - Consumes pending interaction events and presence pings.
   - Computes mood decay, time-of-day/weather state, procedural call parameters, and personality vector low-pass drift.
   - Generates Field Notebook observations when noteworthy triggers occur.
   - Writes canonical state updates to PostgreSQL and pushes updated JSON state snapshots to Redis / CDN edge cache.
3. **Snapshot Read Service (Edge Service)**:
   - Delivers lightweight JSON state snapshots to clients on request or via Server-Sent Events (SSE) / WebSocket keepalives.
4. **Data Stores**:
   - **PostgreSQL**: Canonical store for accounts, bird persistent records, personality vectors, species data, field notebook logs, and visit invitations.
   - **Redis**: High-speed cache for current active aviary state snapshots and ephemeral presence session flags.
   - **Kafka / Apache Pulsar**: Immutable append-only log for raw interaction events.

---

## 3. Data Model & Schema Definitions

### 3.1 Account & Aviary Model (`accounts`, `aviaries`)

```sql
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_hash VARCHAR(64) UNIQUE NOT NULL, -- Blind index for lookup
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    marked_for_deletion_at TIMESTAMPTZ NULL,
    visit_notifications_enabled BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    last_ticked_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    settled_at TIMESTAMPTZ NULL,
    CONSTRAINT uq_account_aviary UNIQUE(account_id)
);
```

### 3.2 Bird & Personality Vector Model (`birds`)

```sql
CREATE TABLE species (
    id VARCHAR(32) PRIMARY KEY, -- e.g., 'warbler_grey', 'nightjar_dark'
    name VARCHAR(64) NOT NULL,
    default_palette JSONB NOT NULL,
    call_grammar_config JSONB NOT NULL
);

CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL REFERENCES species(id),
    given_name VARCHAR(64) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    -- Hidden Personality Vector (Values bounded 0.0000 to 1.0000)
    trait_boldness NUMERIC(6,5) NOT NULL CHECK (trait_boldness BETWEEN 0 AND 1),
    trait_social_warmth NUMERIC(6,5) NOT NULL CHECK (trait_social_warmth BETWEEN 0 AND 1),
    trait_vocal_frequency NUMERIC(6,5) NOT NULL CHECK (trait_vocal_frequency BETWEEN 0 AND 1),
    trait_plumage_saturation NUMERIC(6,5) NOT NULL CHECK (trait_plumage_saturation BETWEEN 0 AND 1),
    trait_curiosity NUMERIC(6,5) NOT NULL CHECK (trait_curiosity BETWEEN 0 AND 1),
    
    -- Current Mood State
    current_mood VARCHAR(16) NOT NULL DEFAULT 'content', -- wary, content, curious, drowsy, alert
    mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    -- Spatial Perch State
    current_perch_zone VARCHAR(16) NOT NULL DEFAULT 'middle' -- front, middle, back
);
```

### 3.3 Interaction Events & Presence Log (`interaction_events`)

```sql
CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(id) ON DELETE CASCADE,
    event_type VARCHAR(32) NOT NULL, -- 'presence_ping', 'listen_in_start', 'listen_in_end', 'offer_seed', 'offer_song', 'offer_pool', 'settle'
    duration_seconds INT NULL,
    payload JSONB NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_events_aviary_created ON interaction_events(aviary_id, created_at DESC);
```

### 3.4 Field Notebook (`notebook_entries`)

```sql
CREATE TABLE notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    entry_prose TEXT NOT NULL, -- Lowercase naturalist prose
    triggered_by VARCHAR(64) NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 3.5 Visits & Invites (`visit_invitations`)

```sql
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    visitor_email_encrypted BYTEA NOT NULL,
    visitor_email_hash VARCHAR(64) NOT NULL,
    token_hash VARCHAR(64) UNIQUE NOT NULL,
    expires_at TIMESTAMPTZ NOT NULL,
    revoked_at TIMESTAMPTZ NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invitation_id UUID NOT NULL REFERENCES visit_invitations(id) ON DELETE CASCADE,
    visited_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    duration_seconds INT NOT NULL
);
```

---

## 4. API Surface & Protocols

### 4.1 Client -> Server REST & Event Endpoints

- `POST /api/v1/auth/magic-link`: Request sign-in magic link.
- `POST /api/v1/auth/verify`: Exchange token for JWT session token.
- `GET /api/v1/aviary/snapshot`: Retrieve current canonical aviary state JSON snapshot.
- `POST /api/v1/aviary/events`: Send interaction event payload (e.g., presence ping, offer, settle).
  - *Payload*: `{ event_type: "presence_ping" | "offer_seed" | "listen_in_start" | "settle", bird_id?: "...", timestamp: 1784793477 }`
- `GET /api/v1/notebook`: Fetch paginated field notebook entries.
- `POST /api/v1/social/invite`: Issue a visit invite link.
- `DELETE /api/v1/social/invite/:id`: Revoke an active visit invite.
- `GET /api/v1/social/visit/:token`: Fetch visitor read-only snapshot (visitor auth).

### 4.2 State Snapshot Schema (`GET /api/v1/aviary/snapshot`)

```json
{
  "aviary_id": "9b1deb4d-3b7d-4bad-9bdd-2b0d7b3dcb6d",
  "timestamp": "2026-07-24T08:00:00Z",
  "time_of_day": "morning",
  "weather": "clear",
  "settled": false,
  "birds": [
    {
      "id": "e4d2a1b0-5c6d-4f3e-8a9b-0c1d2e3f4a5b",
      "given_name": "pip",
      "species": "warbler_grey",
      "mood": "curious",
      "perch_zone": "front",
      "plumage_saturation": 0.642,
      "call_motif_seed": 482910,
      "idle_animation_state": "scanning"
    },
    {
      "id": "a1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
      "given_name": "wren",
      "species": "sparrow_wood",
      "mood": "drowsy",
      "perch_zone": "back",
      "plumage_saturation": 0.510,
      "call_motif_seed": 192834,
      "idle_animation_state": "fluffed"
    }
  ]
}
```

---

## 5. Simulation Engine Design

### 5.1 Server-Side Tick Loop
The Simulation Engine executes a tick loop every 60 seconds for each active aviary.

```
                  ┌───────────────────────────────┐
                  │      Read Event Log Delta     │
                  └───────────────┬───────────────┘
                                  │
                                  ▼
                  ┌───────────────────────────────┐
                  │    Evaluate Presence Time     │
                  │   (Conjunction Validation)    │
                  └───────────────┬───────────────┘
                                  │
                                  ▼
┌───────────────────────────────────────────────────────────────────┐
│              Execute Low-Pass Drift Filter Math                   │
│                                                                   │
│   T_new = T_old + α * Weight(signal) * (1 - T_old)               │
│   where α = 0.00005 (slow monotonic drift target: 3 weeks)        │
└─────────────────────────────────┬─────────────────────────────────┘
                                  │
                                  ▼
┌───────────────────────────────────────────────────────────────────┐
│                  Compute Fast-Timescale Mood                      │
│   Mood_t = f(Personality, Weather, LocalTime, RecentInteractions) │
└─────────────────────────────────┬─────────────────────────────────┘
                                  │
                                  ▼
┌───────────────────────────────────────────────────────────────────┐
│               Evaluate Field Notebook Triggers                    │
│   If noteworthy event condition met & cooldown expired:           │
│   Generate naturalist prose -> insert notebook_entries            │
└─────────────────────────────────┬─────────────────────────────────┘
                                  │
                                  ▼
                  ┌───────────────────────────────┐
                  │ Write Canonical State & Cache │
                  └───────────────────────────────┘
```

### 5.2 Low-Pass Drift Calibration
- **Drift Target**: Measurable numerically at ~1 week; visually felt by user at ~3 weeks.
- **Formula (Additive & Monotonic)**:
  $$\Delta T = \alpha \cdot w_{\text{signal}} \cdot (1.0 - T_{\text{current}})$$
  Where $\alpha = 0.00005$ per tick hour of valid presence.
- **Weights**:
  - `Presence-time`: $w = 1.0$ (Primary driver for boldness & plumage saturation).
  - `Listen-in duration`: $w = 1.5$ (Targeted driver for social warmth & vocal frequency).
  - `Offer interaction`: $w = 1.2$ (Targeted driver for curiosity & boldness).
- **Asymmetrical Monotonic Rule**: Neglect results in zero change ($\Delta T = 0$). Traits never decrease. Absent birds gradually shift toward ambient calling without mistrust or decay.

### 5.3 Mood State Machine
Mood transitions depend on:
1. Local Time: Dawn $\rightarrow$ Alert, Dusk $\rightarrow$ Drowsy.
2. Weather: Rain $\rightarrow$ Dampened Vocal / Wary.
3. Interaction: Offer acceptance $\rightarrow$ Content / Curious.
4. Personality Vector Dampening: High boldness reduces Wary transition probability by 60%.

---

## 6. Presence & Multi-Device Sync Protocol

### 6.1 Multi-Signal Presence Conjunction
The client emits a `presence_ping` event every 30 seconds **ONLY** when all three conditions hold:
1. `document.visibilityState === 'visible'`
2. `document.hasFocus() === true`
3. Pointer movement or keypress detected within the trailing 180 seconds.

If any condition fails, presence pings cease immediately.

### 6.2 Canonical State Engine & Sync Resolution
- The server is the **sole writer** of bird personality vectors.
- Clients submit raw interaction events to the append-only event log.
- Multi-device sync is inherently coherent: both phone and desktop clients read identical JSON snapshots generated by the server tick.
- **No Last-Write-Wins (LWW)**: Client-side vector mutation is impossible, eliminating state overwrite bugs across concurrent devices.

---

## 7. Frontend & Audio Rendering Pipeline

### 7.1 Visual Scene Architecture
- Rendered on HTML5 `<canvas>` via 2D Context (or lightweight WebGL fallback).
- Layout partitioned into 3 depth perches: **Back**, **Middle**, **Front**.
- **First Frame Continuity**: Render initialization draws birds at explicit vector coordinates fetched from snapshot zero. No loading spinners or fade-ins.

```
┌─────────────────────────────────────────────────────────────┐
│ Top Bar UI (Chrome: Settings, Notebook, Offers, A11y)      │
│ [Fades to 0% opacity after 3s cursor stillness]             │
├─────────────────────────────────────────────────────────────┤
│ Sky & Weather Background (Time-of-day gradient, rain, wind) │
│                                                             │
│  [Back Perch Zone]      -- Wary / Low-boldness birds        │
│                                                             │
│  [Middle Perch Zone]    -- Content / Standard perching      │
│                                                             │
│  [Front Perch Zone]     -- Bold / Attentive birds           │
│                                                             │
│ Foreground Foliage & Subtle Parallax Drift                  │
└─────────────────────────────────────────────────────────────┘
```

### 7.2 WebAudio Synthesis Engine
- **Procedural Motif Synthesizer**: Uses WebAudio API native oscillators (`sine`, `triangle`), biquad filters, and custom envelope gain nodes (`gainNode.gain.exponentialRampToValueAtTime`).
- **Chorus Re-balancing (Listen-in)**:
  - Focused bird gain node ramps up slowly (+4dB over 1.2s).
  - Non-focused birds ramp down slowly (-12dB over 1.2s, never zeroed out).
- **Graceful Fallback**: If `AudioContext` is blocked or unavailable, system enters quiet mode with automatic call captioning enabled. No pre-recorded MP3/WAV files are shipped or used.

---

## 8. Accessibility Surfaces

1. **Screen-Reader Narration**:
   - Hidden ARIA Live region (`aria-live="polite"`, `aria-atomic="true"`).
   - Server-generated naturalist prose updated every 30–60 seconds.
   - Example: `"a grey warbler is perching on the front branch, calling softly."`
2. **Reduced-Motion Mode**:
   - Listens to `(prefers-reduced-motion: reduce)` media query and custom user toggle.
   - Replaces skeletal canvas frame-by-frame updates with gentle 800ms cross-fades between static poses.
   - Disables leaf and feather drift particles.
3. **Call Captioning**:
   - Opt-in visual overlays near calling birds.
   - Naturalist prose descriptions generated dynamically from motif parameters: `"a soft three-note rise"`.
4. **Keyboard & Focus**:
   - Tab navigation for top-bar items. `Tab` into scene selects first bird; `ArrowLeft`/`ArrowRight` cycles birds; `Enter` activates listen-in; `Esc` clears focus.
   - High-contrast focus rings tailored for AA compliance on dynamic scene backgrounds.

---

## 9. Performance Budgets & Observability

### 9.1 Technical Budgets
| Metric | Budget Ceiling | Verification Method |
|---|---|---|
| **Initial JS Bundle (gzipped)** | `< 2 MB` | CI Webpack/Vite Bundle Analyzer Check |
| **Time-to-First-Bird Visible** | `< 500 ms` | Lighthouse / WebPageTest over simulated 4G |
| **Idle Render FPS** | `60 fps` | Automated Chrome Performance Tracing (5-year-old laptop profile) |
| **Memory Growth (30 min)** | `0 MB leakage` | Playwright heap dump assertion before & after 30 min idle run |
| **Simulation Tick p99 Latency** | `< 5000 ms` | Datadog/Prometheus alert rule on worker queue |

### 9.2 Privacy-Preserving Telemetry
- Aggregated operational telemetry only (request rates, HTTP 5xx errors, CDN cache hit ratio, render frame drops, WebAudio error codes).
- **Strict Privacy Rule**: Telemetry data pipelines are physically isolated from simulation databases. No per-account bird interactions or notebook logs are ingested into analytics stores.

---

## 10. Rollout & Staging Strategy

1. **Phase 1: Synthetic Simulation Validation (CI/CD)**:
   - Run simulation tick engine against 10,000 synthetic aviaries for 30 simulated days. Validate low-pass drift convergence and notebook prose generation rates.
2. **Phase 2: Closed Internal Alpha**:
   - Deploy to internal staging environment with magic-link auth restricted to team domains. Verify multi-device sync across desktop and mobile browsers.
3. **Phase 3: Beta Rollout**:
   - Canary deploy to 5% of new signups. Monitor CDN snapshot latency, WebAudio initialization success rates, and simulation tick queue depths.
4. **Phase 4: General Availability**:
   - 100% rollout with automated tick queue auto-scaling.

---

## 11. Risk Matrix & Mitigations

| Identified Risk | Severity | Impact | Architectural Mitigation Strategy |
|---|---|---|---|
| **Drift Calibration Imbalance** | High | Birds drift too fast (Tamagotchi feel) or too slow (feels frozen). | Enforce automated low-pass mathematical assertions in CI. Alpha test with automated simulated visits across 3-week virtual timelines. |
| **Multi-Device State Conflict** | High | Devices overwrite personality drift histories. | Server-only canonical simulation writer pattern. Additive event log consumption; zero client-side state authority. |
| **Audio Uncanniness / Phase Cancellation** | Medium | Overlapping calls sound repetitive or phase-canceled. | Pure procedural synthesis via WebAudio motif variations with randomized pitch/timing micro-offsets. Zero stacked audio loops. |
| **Accessibility Degradation** | Medium | Accessible surfaces feel like generic state logs. | Share single naturalist prose generation engine across notebook, narration live region, and call captions. |
| **PII Leakage in Observability** | High | User emails leak into logs/spans. | Strict synthetic UUID primary keying. Email stored in single encrypted table column with blind hash index. |

---
*End of Implementation Plan.*
