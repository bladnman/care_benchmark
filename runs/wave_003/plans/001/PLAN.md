# Pocket Aviary v1 — Comprehensive Implementation Plan

## Executive Summary & Design System Alignment

Pocket Aviary is a modern, browser-native virtual aviary where users observe two to seven animated birds living in a single horizontal scene. Over days and weeks, each bird's personality vector slowly drifts in response to measured user presence and quiet interactions. 

The implementation plan defined herein strictly operationalizes the core design principles:
1. **Feels alive, not robotic**: Server-side simulation continuous ticking, instant snapshot hydration with motion mid-progress, procedural audio synthesis via WebAudio, and mood-driven micro-motion.
2. **Notice, never announce**: Zero return toasts, zero streak counters, zero push notifications, zero gamification dialogs. Greetings are performed purely by bird behavior (glance, step forward, quiet call).
3. **Charm comes from specificity**: Naturalist, present-tense, lowercase field-notebook observations and screen-reader narration over generic state logs.
4. **Restraint over richness**: A capped single-screen scene (max 7 birds, 3 perches), minimal top-bar chrome with cursor-idle fading, and web-only architecture.
5. **Naturalist voice vs. Matter-of-fact system voice**: Naturalist prose across product/notebook/narration surfaces; matter-of-fact, clear language across auth, error, sync, and settings surfaces.

---

## 1. Scope & System Boundaries

### 1.1 In-Scope for V1
- **Starter & Cap**: Starts with 2 automatically assigned starter birds; cap of 7 birds total unlocked progressively based strictly on aviary age.
- **Authentication**: Single-user accounts authenticated via email magic links (15-minute expiration, synthetic UUID internal mapping).
- **Canonical Simulation Engine**: Server-side 1-minute simulation tick driving additive monotonic personality drift, mood state machines, and field notebook generation.
- **Multi-Device Synchronization**: Client snapshot consumption + append-only event log submission. Server is the sole canonical writer of personality state.
- **Interactions**: Measured presence accounting (conjunction of `visibilityState === 'visible'`, window focus, and recent pointer/key activity), listen-in audio focus, seed/song/pool offers with cooldowns, soft settle session-end gesture, and read-only field notebook.
- **Social (Optional & Quiet)**: Host-initiated read-only ambient visit invitations via one-time email link. Fully revocable, default off, no co-presence, no visitor drift impact.
- **Accessibility**: Naturalist screen-reader narration (ARIA live region), reduced-motion mode (cross-fade pose rendering), procedural call captioning, keyboard navigation, and WCAG AA contrast.
- **Performance**: <2MB initial JS bundle, <500ms time-to-first-bird render on 4G, 60fps runtime performance, and zero memory growth over 30 minutes.

### 1.2 Non-Goals & Explicit Exclusions
- **No Native Apps**: Web-only (modern evergreen web browsers).
- **No Gamification**: Absolute exclusion of streak counters, daily visit dots, levels, scores, badges, adoption counters, XP, or ranking.
- **No Tamagotchi Mechanics**: Birds never die, starve, decay, or exhibit distress. Neglect produces quiet ambient behavior, never negative personality drift.
- **No Social Network Features**: No public profiles, comments, chat, co-presence cursors, public discovery feeds, or leaderboards.

---

## 2. System Architecture & Technical Topology

```
                                 ┌─────────────────────────────────────────┐
                                 │              Browser Client             │
                                 │  - Single Horizontal Scene Render       │
                                 │  - WebAudio Procedural Call Engine      │
                                 │  - Presence & Event Collector           │
                                 └────────────────────┬────────────────────┘
                                                      │ HTTPS / WSS
                                                      ▼
                                 ┌─────────────────────────────────────────┐
                                 │            CDN / Edge Tier              │
                                 │  - Static Asset Delivery (<2MB)         │
                                 │  - Initial Snapshot Bootstrap           │
                                 └────────────────────┬────────────────────┘
                                                      │
                                                      ▼
                                 ┌─────────────────────────────────────────┐
                                 │          Aviary API Gateways            │
                                 │  - Auth / Magic Link Service            │
                                 │  - Snapshot Query Endpoint              │
                                 │  - Interaction Event Ingestion API      │
                                 │  - Social Visit Proxy                   │
                                 └──────────┬───────────────────┬──────────┘
                                            │                   │
                                            ▼                   ▼
┌──────────────────────────────────────────────┐     ┌──────────────────────────────────────────────┐
│        Server-Side Simulation Service        │     │         Event Store & Primary DB             │
│ - 1-Minute Cron/Worker Tick                  │ ──> │ - Append-only Event Log                      │
│ - Monotonic Drift Filter                     │     │ - Canonical Aviary & Bird Vectors            │
│ - Mood State Machine & Weather               │ <── │ - Field Notebook & Visit Log                 │
│ - Field Notebook Generator                   │     │ - Synthetic UUID Account Index               │
└──────────────────────────────────────────────┘     └──────────────────────────────────────────────┘
```

### 2.1 Component Responsibilities
1. **Client Tier**: Standard Web App (Vanilla CSS + HTML5 Canvas/SVG + WebAudio API). Responsible for state snapshot polling, interpolating motion between snapshots, local presence evaluation, rendering micro-motion, synthesizing call motifs, and emitting append-only user interaction events.
2. **API Gateway Tier**: Stateless HTTP endpoints for magic link auth, reading aviary snapshots, submitting event batches, managing settings/exports, and validating visit invitation tokens.
3. **Simulation Worker Service**: Background service running an asynchronous 1-minute tick per active/ticking aviary. Reads un-ingested interaction events, executes low-pass personality drift equations, advances mood states according to time-of-day and weather, and writes updated canonical state.
4. **Data Persistence Tier**: Relational PostgreSQL database for canonical state and event store. All tables key off synthetic account UUIDs.

---

## 3. Data Model & Schema Specifications

### 3.1 Database Schema (PostgreSQL)

```sql
-- Accounts Table (PII Encrypted at Rest, Synthetic UUID Keyed)
CREATE TABLE accounts (
    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_hash VARCHAR(64) UNIQUE NOT NULL, -- For auth lookups only
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    deletion_scheduled_at TIMESTAMPTZ DEFAULT NULL
);

-- Aviaries Table (One per account at v1)
CREATE TABLE aviaries (
    aviary_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID UNIQUE NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    age_days INT NOT NULL DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Birds Table
CREATE TABLE birds (
    bird_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL,
    name VARCHAR(64) NOT NULL,
    slot_index INT NOT NULL, -- 0 to 6
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Personality Vectors (Hidden, Normalized Scalars [0.0 - 1.0])
CREATE TABLE personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(bird_id) ON DELETE CASCADE,
    boldness NUMERIC(5,4) NOT NULL DEFAULT 0.2000,
    social_warmth NUMERIC(5,4) NOT NULL DEFAULT 0.2000,
    vocal_frequency NUMERIC(5,4) NOT NULL DEFAULT 0.2000,
    plumage_saturation NUMERIC(5,4) NOT NULL DEFAULT 0.2000,
    curiosity NUMERIC(5,4) NOT NULL DEFAULT 0.2000,
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Bird Mood States (Fast-timescale daily/session state)
CREATE TYPE mood_enum AS ENUM ('wary', 'content', 'curious', 'drowsy', 'alert');

CREATE TABLE bird_moods (
    bird_id UUID PRIMARY KEY REFERENCES birds(bird_id) ON DELETE CASCADE,
    current_mood mood_enum NOT NULL DEFAULT 'content',
    last_interaction_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Append-Only Interaction Event Log
CREATE TYPE event_type_enum AS ENUM (
    'presence_ping', 'listen_in_start', 'listen_in_end', 
    'offer_seed', 'offer_song', 'offer_pool', 'settle'
);

CREATE TABLE interaction_events (
    event_id BIGSERIAL PRIMARY KEY,
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    bird_id UUID REFERENCES birds(bird_id) ON DELETE CASCADE,
    event_type event_type_enum NOT NULL,
    duration_seconds INT DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_events_account_tick ON interaction_events(account_id, created_at);

-- Field Notebook Entries
CREATE TABLE notebook_entries (
    entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(aviary_id) ON DELETE CASCADE,
    prose_text TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Social Visit Invitations
CREATE TYPE invite_status_enum AS ENUM ('pending', 'active', 'revoked', 'expired');

CREATE TABLE visit_invitations (
    invite_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    visitor_email_hash VARCHAR(64) NOT NULL,
    token VARCHAR(128) UNIQUE NOT NULL,
    status invite_status_enum NOT NULL DEFAULT 'pending',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL
);

-- Visit Audit Log
CREATE TABLE visit_log (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    visitor_email_masked VARCHAR(128) NOT NULL,
    visited_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    duration_seconds INT NOT NULL DEFAULT 0
);
```

---

## 4. API Surface & Contract Specifications

### 4.1 Authentication & Accounts API
- `POST /api/v1/auth/magic-link`: Request sign-in magic link.
  - Request: `{ "email": "user@example.com" }`
  - Response: `{ "status": "sent" }` (Matter-of-fact registration voice on failure).
- `GET /api/v1/auth/verify?token=<token>`: Verify magic link token, issue session JWT cookie.
- `POST /api/v1/auth/revoke-session`: Revoke current or specified session.
- `GET /api/v1/account/export`: Request JSON export of aviary state.
- `DELETE /api/v1/account`: Initiate 30-day soft deletion process.

### 4.2 Aviary State & Interaction API
- `GET /api/v1/aviary/snapshot`: Fetch current canonical state for client rendering.
  - Response payload (JSON):
    ```json
    {
      "aviary_id": "8f3b2a1c-...",
      "timestamp": "2026-07-24T07:18:00Z",
      "weather": "clear",
      "light_phase": "morning",
      "birds": [
        {
          "bird_id": "c1a2b3...",
          "name": "Pip",
          "species_id": "sp_warbler",
          "perch_zone": "front",
          "mood": "content",
          "call_motif_seed": 4092,
          "plumage_saturation_render": 0.45
        },
        {
          "bird_id": "d4e5f6...",
          "name": "Wren",
          "species_id": "sp_sparrow",
          "perch_zone": "back",
          "mood": "wary",
          "call_motif_seed": 8112,
          "plumage_saturation_render": 0.22
        }
      ]
    }
    ```
- `POST /api/v1/aviary/events`: Batch submit append-only interaction events.
  - Request payload:
    ```json
    {
      "events": [
        { "type": "presence_ping", "duration_seconds": 60 },
        { "type": "listen_in_start", "bird_id": "c1a2b3...", "timestamp": "..." },
        { "type": "offer_seed", "bird_id": "c1a2b3...", "timestamp": "..." }
      ]
    }
    ```

### 4.3 Field Notebook & Social API
- `GET /api/v1/notebook`: Fetch paginated list of naturalist notebook observations.
- `POST /api/v1/visits/invite`: Create a new visit invitation.
  - Request: `{ "visitor_email": "friend@example.com" }`
- `DELETE /api/v1/visits/invite/:invite_id`: Immediately revoke an invitation.
- `GET /api/v1/visits/snapshot/:token`: Read-only snapshot fetch for visitors.

---

## 5. Server-Side Simulation Engine & Drift Mechanics

### 5.1 The Server Simulation Tick
The simulation tick runs asynchronously every 60 seconds per account:
1. **Ingest Events**: Pulls all unconsumed records from `interaction_events` since the last tick timestamp.
2. **Calculate Presence & Interaction Weights**: Sums qualified presence seconds and interaction weights.
3. **Execute Monotonic Low-Pass Personality Drift**:
   For each trait $T \in \{\text{boldness}, \text{social\_warmth}, \text{vocal\_frequency}, \text{plumage\_saturation}, \text{curiosity}\}$:
   $$\Delta T = \alpha \cdot f_{\text{presence}}(\text{presence\_seconds}) + \beta \cdot g_{\text{interaction}}(\text{events})$$
   $$\text{Trait}_{n+1} = \min(1.0, \text{Trait}_n + \max(0, \Delta T))$$
   *Crucial Invariant*: $\Delta T \ge 0$. Traits never decrease due to neglect or absence.
4. **Calibration Targets**:
   - **1 Week (~10,080 ticks)** of regular presence: Instrument-detectable shift ($\Delta T \approx +0.05$).
   - **3 Weeks (~30,240 ticks)**: Visibly perceptible change to user ($\Delta T \approx +0.18$).
5. **Mood State Machine & Time-of-Day/Weather Modulator**:
   - Updates mood using inputs from local timezone hour, recent offer responses, ambient weather events (e.g. rain dampening vocal frequency), and underlying personality vector threshold.
   - Mood is persisted across sessions; it does not reset on tab open.
6. **Field Notebook Generation**:
   - Evaluates rule triggers on a sparse cadence (once every 2-3 days per active aviary).
   - Generates naturalist, present-tense, lowercase prose (e.g., *"tuesday — pip greeted before wren today, first time this week."*).

---

## 6. Multi-Device Sync & Conflict Prevention

1. **Single Canonical Writer**: Only the server simulation tick writes to `personality_vectors` and `bird_moods`.
2. **No Last-Write-Wins (LWW)**: Clients never write absolute trait values. Clients submit un-opinionated interaction events (`listen_in`, `presence_ping`, `offer`).
3. **State Hydration & Interpolation**: Clients pull snapshots on tab visibility restore, long frame gaps, or 30-second keepalives. Position changes between snapshots are smoothly interpolated client-side using easing curves over 1,500ms.

---

## 7. Frontend Rendering Pipeline & Scene Design

### 7.1 Visual Scene Architecture
- **View Boundary**: Fixed single horizontal scene fitting any viewport size (mobile to desktop) without horizontal scrolling or panning.
- **Perch Zones**: Three distinct depth planes (Front, Middle, Back). Bird placement is determined solely by personality/mood (e.g., high boldness $\rightarrow$ Front perch; wary mood $\rightarrow$ Back perch). User placement is strictly disabled.
- **Top Bar Chrome**:
  - Icons: Settings/Account, Accessibility Settings, Field Notebook, Offers.
  - Fading Behavior: Fades to 5% opacity after 3 seconds of cursor/keyboard stillness; returns to 100% opacity on input.
- **First Frame Load**: Scene initializes instantly with birds mid-action (preening, soft calling). Loading state fallback is a quiet soft sky field (zero loading spinners).
- **Reduced-Motion Mode**: Replaces continuous animation loops with slow cross-fades (2,000ms pose transitions). Disables leaf/feather particles while preserving full audio, drift, and notebook functionality.

---

## 8. WebAudio Procedural Synthesis & Audio Pipeline

```
┌────────────────────────────────────────────────────────────────────────┐
│                   Procedural Audio Engine (WebAudio)                   │
├────────────────────────────────────────────────────────────────────────┤
│  Species Motif Library  ──>  Oscillators / FM Synthesis Node           │
│                                           │                            │
│                                           ▼                            │
│                               Dynamic Gain & Panner Nodes              │
│                                           │                            │
│                                           ▼                            │
│  Listen-In Gain Ramp    ──>   Master Chorus Bus Mixer                  │
│                                           │                            │
│                                           ▼                            │
│                                   Audio Destination                    │
└────────────────────────────────────────────────────────────────────────┘
```

1. **Motif Synthesis**: Synthesizes bird calls at runtime via WebAudio FM synthesis and filtered noise nodes based on per-species motif parameters. No audio samples are downloaded.
2. **Chorus Dynamics**: Micro-randomizes pitch and call timing offsets between calling birds to prevent phase cancellation. Cap of 7 birds preserves distinct audible call signatures.
3. **Listen-In Re-balance**: Selecting a bird initiates a logarithmic gain ramp (+6dB over 1,500ms) for the focused bird, while reducing non-focused birds to an ambient background level (-12dB over 1,500ms). Other birds are never fully muted.
4. **Audio Fallback**: If WebAudio is unsupported or permission is denied, the engine runs silently and automatically enables call captions.

---

## 9. Accessibility Implementation

1. **Naturalist Screen-Reader Narration**:
   - Dedicated ARIA live region (`aria-live="polite"`).
   - Generates naturalist, present-tense prose summaries updated every 30-60 seconds (or immediately on user-initiated events like return-greeting or offer).
   - Example: *"a small grey bird is perched on the front rail, calling softly."*
2. **Procedural Call Captions**:
   - Displays short naturalist text near calling birds (e.g., *"a soft three-note rise"*).
   - Driven directly by the active WebAudio motif generator parameters.
3. **Keyboard Focus & Navigation**:
   - Full keyboard accessibility (Tab cycles top bar; Arrow keys navigate birds; Enter triggers Listen-in; Escape disengages).
   - High-contrast visual focus ring passing WCAG AA contrast against all day/night palettes.

---

## 10. Performance Budgets & Observability

### 10.1 Hard Performance Budgets
- **Initial JS Bundle Size**: < 2MB gzipped at first paint.
- **Time-to-First-Bird**: < 500ms on 4G mid-tier mobile device.
- **Frame Rate**: 60fps continuous idle motion on 5-year-old laptop hardware.
- **Memory Footprint**: 0MB memory leak growth over a 30-minute session (verified via CI Chrome DevTools protocol tests).

### 10.2 Privacy & Telemetry Boundary
- **Allowed Operational Metrics**: Aggregate request rates, simulation tick execution latency p99 (alert threshold > 5s), WebAudio context initialization error counts, and anonymized RUM page load histograms.
- **Strict Data Exclusion**: Per-bird personality vectors, per-account interaction events, notebook prose, and user presence logs are strictly excluded from telemetry warehouses and analytics pipelines.

---

## 11. Rollout & Risk Management

### 11.1 Phased Rollout Schedule
- **Phase A (Internal Canary)**: Synthetic tick validation, drift low-pass filter verification, WebAudio cross-browser testing.
- **Phase B (Beta Release)**: 2 starter birds enabled, age-based unlocks active, 10% account cohort.
- **Phase C (General Availability)**: 100% rollout, multi-device sync, social visit invitations enabled.

### 11.2 Key Risks & Mitigations
| Risk Description | Severity | Mitigation Strategy |
|---|---|---|
| **Drift Calibration Failure** (Drift occurs too fast or feels static) | High | Automated CI test harness asserting numerical vector bounds across simulated 7-day and 21-day tick streams. |
| **Multi-Device LWW Data Loss** | Critical | Enforce strict append-only event log pattern; server tick is sole mutator of personality vectors. |
| **Robotic Screen-Reader Narration** | Medium | Screen-reader prose generator shares exact template engine with Field Notebook. |
| **WebAudio Autoplay Blocking** | Medium | Graceful initial muted state with soft user gesture listener restoring audio context seamlessly. |
