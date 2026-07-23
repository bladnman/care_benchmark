# Implementation Plan: Pocket Aviary (v1)

## Executive Summary

Pocket Aviary is a calm, browser-based virtual aviary where users adopt a small handful of animated birds whose personalities drift over weeks in response to quiet presence and subtle interactions. This document outlines the end-to-end technical implementation plan for v1. It bridges product specifications into executable engineering designs spanning system architecture, data schemas, simulation engine algorithms, multi-device synchronization, rendering and audio pipelines, accessibility surfaces, performance budgets, and operational telemetry.

---

## 1. Scope & Non-Goals

### 1.1 In-Scope for V1
- **Platform & Delivery:** Single Page Web Application (SPA) targeting modern desktop and mobile web browsers.
- **Aviary Capacity & Adoption:** 2 starter birds assigned at account creation from a 6-species pool; age-based unlocked additions up to a strict cap of 7 birds.
- **Core Interactions:**
  - *Return-Greeting:* Procedurally varied, non-announcing greeting by a bird upon tab arrival, staggered when multiple birds respond, scaling with absence duration.
  - *Listen-in:* Focus single bird to rebalance audio mix via WebAudio smoothly without muting background birds.
  - *Offers:* Interactive gifts (seed, song fragment, still pool) with per-bird cooldowns (3–5 minutes).
  - *Settle:* User-initiated soft session end with evening lighting transition and 5-second undo window.
  - *Field Notebook:* Auto-generated, sparse (1 entry every few days), read-only naturalist prose log.
  - *Presence Accounting:* Concurrent validation of `visibilityState === 'visible'`, window focus, and recent pointer/keyboard activity (3-minute activity window).
- **Social Feature:** Optional, read-only visit invitations issued via email magic links; revocable, auto-expiring in 30 days, with silent visit logging for host.
- **Authentication & Accounts:** Magic-link email auth (15-minute token expiration); single canonical aviary per account; JSON account data export; 30-day soft deletion window.
- **Accessibility Surfaces:** Screen-reader naturalist prose narration (`aria-live="polite"`), reduced-motion mode (slow cross-fades replacing micro-motion/drift paths), procedural call captions, keyboard navigation (Tab/Arrow/Enter/Esc), WCAG AA contrast.

### 1.2 Explicit Non-Goals & Architectural Exclusions
- **No Native Apps:** Web-browser only; no iOS/Android client codebase or native protocol abstractions.
- **No Gamification:** Absolute exclusion of achievements, levels, scores, badges, streaks, calendars of visits, XP, ranks, or visit counters.
- **No Tamagotchi Mechanics:** No bird mortality, hunger meters, distress states, or decay of happiness. Personality drift is strictly monotonic toward expressive; neglect produces ambient quietness without negative numerical drift.
- **No Social Network Infrastructure:** No user profiles, friend lists, public feeds, global discovery directories, leaderboards, co-presence (shared cursors), visit comments, or chat.

---

## 2. Architecture & Service Boundaries

The platform employs a decoupled client-server architecture where the server acts as the sole authoritative simulator and state writer, and the client operates as a render-and-synthesis engine.

```
                    +---------------------------------------+
                    |           Client Browser              |
                    |  - SPA Frontend (TypeScript / HTML5)  |
                    |  - 2D Canvas/WebGL Render Engine      |
                    |  - WebAudio Synthesis Runtime         |
                    |  - Presence & Input Event Collector   |
                    +-------------------+-------------------+
                                        |
                             HTTPS / WSS| JSON API
                                        v
+---------------------------------------+---------------------------------------+
|                            Backend Infrastructure                             |
|                                                                               |
|  +------------------------+  +------------------------+  +-----------------+  |
|  |     API Gateway /      |  |  Simulation Engine     |  | Auth & Account  |  |
|  |     Snapshot API       |  |  Worker (Tick Engine)  |  | Service         |  |
|  +-----------+------------+  +-----------+------------+  +--------+--------+  |
|              |                           |                        |           |
|              v                           v                        v           |
|  +-------------------------------------------------------------------------+  |
|  |                 Authoritative Database (PostgreSQL)                     |  |
|  |   - Encrypted PII (Emails)   - Canonical State Snapshots                |  |
|  |   - Synthetic Account UUIDs  - Append-Only Event Log                    |  |
|  |   - Personality Vectors      - Field Notebook Entries                   |  |
|  +-------------------------------------------------------------------------+  |
+-------------------------------------------------------------------------------+
```

### 2.1 Server Responsibilities
- **Authentication & Identity:** Issues magic links, verifies email tokens, generates synthetic account UUIDs, manages encrypted PII storage.
- **Authoritative Simulation Engine:** Executes background tick worker (~1-minute cadence) to process client event logs, update personality vectors, transition mood states, advance daylight/weather cycles, and write canonical state snapshots.
- **API & Data Access Layer:** Serves lightweight aviary snapshots to clients, appends interaction events, handles account exports, and manages visit invitations.

### 2.2 Client Responsibilities
- **Render Engine:** 60fps interpolation between server state snapshots; procedural ambient micro-motions (preening, head tilts, scanning, leaf drift).
- **WebAudio Audio Runtime:** Real-time synthesis of procedural bird calls from motif definitions; chorus rebalancing and listen-in mix modulation.
- **Presence & Accessibility Runtime:** Monitors presence conjunction signals (`visibilityState`, focus, user input); drives screen-reader narration live region and call captions.

---

## 3. Data Model & Schema Design

All tables reference accounts strictly via a synthetic UUID (`account_id`). Email PII is encrypted at rest using AES-256-GCM and stored solely in the `accounts` table.

```sql
-- Core Accounts Table
CREATE TABLE accounts (
    account_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email_encrypted BYTEA NOT NULL,
    email_hash VARCHAR(64) UNIQUE NOT NULL, -- SHA-256 for lookup
    status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE', -- ACTIVE, PENDING_DELETION
    deleted_at TIMESTAMPTZ NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Birds Table
CREATE TABLE birds (
    bird_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL,
    user_assigned_name VARCHAR(64) NOT NULL,
    is_starter BOOLEAN NOT NULL DEFAULT FALSE,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT fk_birds_account FOREIGN KEY (account_id) REFERENCES accounts(account_id)
);

-- Personality Vectors Table (Server Authoritative)
CREATE TABLE personality_vectors (
    bird_id UUID PRIMARY KEY REFERENCES birds(bird_id) ON DELETE CASCADE,
    boldness FLOAT NOT NULL DEFAULT 0.20,         -- 0.0 to 1.0 (approach vs retreat)
    social_warmth FLOAT NOT NULL DEFAULT 0.20,    -- 0.0 to 1.0 (greeting & chorus response)
    vocal_frequency FLOAT NOT NULL DEFAULT 0.20, -- 0.0 to 1.0 (unobserved call rate)
    plumage_saturation FLOAT NOT NULL DEFAULT 0.20,-- 0.0 to 1.0 (visual detail & richness)
    curiosity FLOAT NOT NULL DEFAULT 0.20,       -- 0.0 to 1.0 (offer investigation likelihood)
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    CONSTRAINT check_vector_bounds CHECK (
        boldness BETWEEN 0.0 AND 1.0 AND
        social_warmth BETWEEN 0.0 AND 1.0 AND
        vocal_frequency BETWEEN 0.0 AND 1.0 AND
        plumage_saturation BETWEEN 0.0 AND 1.0 AND
        curiosity BETWEEN 0.0 AND 1.0
    )
);

-- Current Bird Mood Table
CREATE TABLE bird_moods (
    bird_id UUID PRIMARY KEY REFERENCES birds(bird_id) ON DELETE CASCADE,
    current_mood VARCHAR(20) NOT NULL DEFAULT 'CONTENT', -- WARY, CONTENT, CURIOUS, DROWSY, ALERT
    mood_entered_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    cooldown_until TIMESTAMPTZ NULL
);

-- Append-Only Interaction Event Log
CREATE TABLE interaction_events (
    event_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(bird_id) ON DELETE SET NULL,
    event_type VARCHAR(32) NOT NULL, -- PRESENCE_PING, LISTEN_IN_START, LISTEN_IN_END, OFFER_SEED, OFFER_SONG, OFFER_POOL, SETTLE
    duration_seconds INT DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_interaction_events_acc_time ON interaction_events(account_id, created_at);

-- Field Notebook Entries
CREATE TABLE notebook_entries (
    entry_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(bird_id) ON DELETE SET NULL,
    entry_prose TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Social Visit Invitations
CREATE TABLE visit_invitations (
    invite_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    visitor_email_hash VARCHAR(64) NOT NULL,
    invite_token_hash VARCHAR(64) UNIQUE NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'PENDING', -- PENDING, REVOKED, EXPIRED
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    expires_at TIMESTAMPTZ NOT NULL
);

-- Host Silent Visit Log
CREATE TABLE visit_logs (
    visit_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    host_account_id UUID NOT NULL REFERENCES accounts(account_id) ON DELETE CASCADE,
    visitor_email_masked VARCHAR(64) NOT NULL,
    started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    ended_at TIMESTAMPTZ NULL
);
```

---

## 4. API Surface & Protocols

Communication uses HTTPS with JSON payloads. Error responses on public surfaces strictly use the matter-of-fact register.

### 4.1 Authentication & Account APIs
- `POST /api/v1/auth/magic-link`
  - Request: `{ "email": "user@example.com" }`
  - Response: `{ "status": "sent" }` (Matter-of-fact response regardless of email existence to prevent enumeration).
- `POST /api/v1/auth/verify`
  - Request: `{ "token": "magic_link_token_str" }`
  - Response: `{ "session_token": "jwt_session_string", "expires_at": "..." }`
- `GET /api/v1/account/export`
  - Auth required. Generates and returns a signed URL for JSON data export download.
- `DELETE /api/v1/account`
  - Auth required. Marks account status as `PENDING_DELETION` (initiates 30-day recovery window).

### 4.2 Aviary Snapshot & Interaction APIs
- `GET /api/v1/aviary/snapshot`
  - Auth required. Returns current canonical aviary state.
  - Response payload:
    ```json
    {
      "aviary_time": "2026-07-24T06:38:58Z",
      "daylight_state": "MORNING",
      "weather_state": "CLEAR",
      "birds": [
        {
          "bird_id": "8f3b2c1a-...",
          "name": "Pip",
          "species_id": "WARBLER",
          "perch_zone": "FRONT",
          "mood": "CURIOUS",
          "plumage_saturation": 0.45,
          "call_motif_seed": 1042
        }
      ]
    }
    ```
- `POST /api/v1/aviary/events`
  - Auth required. Accepts an array of client interaction events (presence pings, offer actions, listen-in windows, settle triggers).

### 4.3 Social Visit APIs
- `POST /api/v1/invitations`
  - Host Auth required. Input: `{ "visitor_email": "friend@example.com" }`. Issues invitation.
- `DELETE /api/v1/invitations/:invite_id`
  - Host Auth required. Immediately revokes invitation.
- `GET /api/v1/visits/:token`
  - Public. Resolves token to read-only aviary snapshot. Returns matter-of-fact error if expired or revoked.

---

## 5. Simulation Engine & Drift Runtime

The server-side simulation tick runs asynchronously every ~60 seconds per active aviary.

```
                    +------------------------------------+
                    |    Server Simulation Tick (1m)     |
                    +-----------------+------------------+
                                      |
       +------------------------------+------------------------------+
       |                              |                              |
       v                              v                              v
+--------------+              +---------------+              +---------------+
| Evaluate     |              | Apply Drift   |              | Transition    |
| Presence     |              | Low-Pass      |              | Fast-Timescale|
| & Events     |              | Filter        |              | Mood States   |
+--------------+              +---------------+              +---------------+
```

### 5.1 Presence & Event Evaluation
The tick fetches unprocessed `interaction_events` for the aviary. Presence time $P$ (in minutes) is calculated strictly from validated `PRESENCE_PING` events.

### 5.2 Personality Drift Function
Drift is computed using an exponential low-pass filter. Trait values move monotonically upward toward expressive limits ($T_{max} = 1.0$) upon presence; they never decay upon absence.

$$\Delta T = \alpha \cdot (1.0 - T_{current}) \cdot f(P, E)$$

Where:
- $\alpha \approx 0.0005$ per tick (calibrated so measurable numerical drift occurs at 7 days and user-perceivable drift at 21 days).
- $f(P, E)$ weighs presence duration $P$, listen-in duration, and offer interactions.
- If $P = 0$ (user absent), $\Delta T = 0$. (No negative drift).

### 5.3 Mood State Machine
Mood is a fast-timescale state ($M \in \{\text{WARY, CONTENT, CURIOUS, DROWSY, ALERT}\}$).
- **Triggers:**
  - Local timezone time-of-day (Drowsy near dusk; Alert at morning).
  - Recent interactions (Accepted offer $\rightarrow$ Curious/Content).
  - Ambient events (Rain $\rightarrow$ Drowsy/Wary; peer alarm call $\rightarrow$ Wary).
  - Personality trait modulators (High Boldness reduces Wary duration).
- Mood persists across sessions and transitions smoothly during server ticks while user is away.

### 5.4 Call-Grammar Engine
Each species possesses a procedural motif library. Call generation computes:
- **Base Pitch & Motif Sequence:** Species signature + bird seed.
- **Tempo & Vocal Frequency:** Scaled by `vocal_frequency` trait and current `mood`.
- **Micro-Variation:** Random pitch jitter ($\pm 2\%$) per call to guarantee procedural freshness and avoid phase cancellation.

---

## 6. Synchronization & Conflict Prevention

### 6.1 Canonical Single-Writer Model
- The server is the exclusive writer of `personality_vectors` and `bird_moods`.
- Clients never compute or submit absolute personality values.
- Multi-device consistency is guaranteed because all client devices (laptop, phone) pull snapshots from the single server canonical record.

### 6.2 Conflict-Free Append-Only Logging
- Clients submit events (`PRESENCE_PING`, `OFFER`, `LISTEN_IN`) to `interaction_events`.
- Events are processed sequentially by the server tick worker in timestamp order.
- This entirely eliminates Last-Write-Wins (LWW) data loss. A session on a phone and a session on a laptop both append events to the log without overwriting vectors.

---

## 7. Frontend Rendering Pipeline

### 7.1 Visual Composition & Layout
- **Single Horizontal Viewport:** Fixed single-screen scene without panning, scrolling, or zooming.
- **Three Perch Zones:**
  - *Front Perch:* High proximity, reserved for bold/curious birds.
  - *Middle Perch:* Default resting zone.
  - *Back Perch:* Distant, favored by wary or drowsy birds.
- **Perch Selection:** Automated signal driven by bird mood and boldness trait; user cannot manually drag or place birds.

### 7.2 Idle Micro-Motion & Loading Sequence
- **Loaded Mid-Motion:** Scene initializes immediately with birds mid-action (preening, head-tilting, scanning). No entry animations or "wake-up" transitions.
- **Loading Fallback:** If snapshot fetch is delayed, client renders a calm empty sky field (no loading spinners or app frame graphics).
- **Chrome Auto-Fade:** Top bar controls fade to near-zero opacity after 4 seconds of mouse/keyboard inactivity.

### 7.3 Reduced-Motion Mode
- Automatically enabled via `prefers-reduced-motion` media query or accessibility setting.
- Replaces frame-by-frame animations and motion interpolation with slow (1.5s) opacity cross-fades between static pose snapshots.
- Removes ambient leaf and feather drift while maintaining daylight color transitions and full WebAudio synthesis quality.

---

## 8. Audio Pipeline & WebAudio Runtime

```
                    +------------------------------------+
                    |     Client WebAudio Engine         |
                    +-----------------+------------------+
                                      |
       +------------------------------+------------------------------+
       |                              |                              |
       v                              v                              v
+--------------+              +---------------+              +---------------+
| Procedural   |              | Dynamic Chorus|              | Listen-in     |
| Motif Synth  |              | Mixer         |              | Mix Ramp      |
+--------------+              +---------------+              +---------------+
```

### 8.1 Procedural Synthesis (No Recorded Audio)
- Calls are synthesized via WebAudio API oscillators (sine/triangle blends) and gain envelopes.
- Absolutely zero pre-recorded audio loops are shipped, preserving bundle size and preventing repetitive audible patterns.

### 8.2 Chorus & Listen-In Mix Control
- **Chorus Rebalance:** Up to 7 birds calling concurrently are mixed dynamically into ambient space.
- **Listen-In Interaction:**
  - Engaging listen-in triggers a 1.5-second logarithmic gain ramp raising the target bird's gain to 1.0.
  - Non-targeted birds decay smoothly to ambient background gain (0.15) but are **never fully muted**.

### 8.3 WebAudio Fallback Path
- If WebAudio is unsupported or context initialization fails/is blocked, the engine drops into graceful silent mode.
- Call captions are automatically enabled by default in silent mode. No recorded audio fallback is attempted.

---

## 9. Accessibility Surfaces

### 9.1 Screen-Reader Narration Engine
- Uses an `aria-live="polite"` DOM container.
- Serves naturalist prose updates every 30–60 seconds at idle or immediately following user-triggered events.
- **Tone:** Lowercase, present-tense, specific (e.g., *"a small grey bird perches on the high rail, calling softly."*). Strictly refrains from UI announcement phrasing or state dumps.

### 9.2 Real-Time Call Captioning
- Optional floating text captions rendered near the calling bird during vocalizations.
- Captions are generated procedurally from call grammar parameters (e.g., *"a soft three-note rise"*, *"a low trill, paused"*).

### 9.3 Keyboard Navigation & Contrast
- Full keyboard traversal using Tab (Top Bar controls $\rightarrow$ Aviary Scene $\rightarrow$ Birds), Arrow keys (switch focused bird), Enter (trigger listen-in), and Esc (cancel listen-in / undo settle).
- High-contrast visual focus rings optimized for both light midday and dark night aviary palettes. All text passes WCAG AA contrast thresholds.

---

## 10. Performance Budgets & Observability

### 10.1 Quantitative Performance Budgets
- **Initial JS Bundle Size:** $\le 2.0\text{ MB}$ gzipped (achieved via WebAudio synthesis instead of audio samples, compact SVG assets, and route code-splitting).
- **Time-to-First-Bird Visible:** $< 500\text{ ms}$ on mid-tier mobile hardware over 4G connections.
- **Render Frame Rate:** Consistent 60fps idle micro-motion on a 5-year-old laptop.
- **Memory Growth Limit:** Zero memory leak growth over continuous 30-minute sessions (verified via CI automated browser heap tests).

### 10.2 Telemetry & Privacy Boundaries
- **Permitted Telemetry:** Aggregate operational metrics only (API latency histograms, simulation tick processing duration, error rates, WebAudio context error counts).
- **P99 Alarm Threshold:** Alert triggers if simulation tick p99 latency exceeds 5 seconds.
- **Strict Privacy Rule:** Zero collection or aggregation of PII, email addresses, individual bird vector states, or per-user interaction logs in telemetry warehouses.

---

## 11. Rollout & Aviary Growth Pacing

### 11.1 V1 Release Target
- Launch single-page web application featuring magic link authentication, 2 starter birds per user, core aviary engine, field notebook, and complete accessibility suite.

### 11.2 Bird Capacity Growth Pacing
- New bird adoption offers are unlocked strictly by **aviary age** (creation date), never by visit count or engagement metrics.
  - *Day 1:* 2 starter birds.
  - *Month 2:* 3rd bird offer available.
  - *Month 6+:* Additional birds offered gradually up to the strict cap of 7.

---

## 12. Risk Matrix & Mitigations

| Risk Factor | Impact | Mitigation Strategy |
| :--- | :--- | :--- |
| **Drift Calibration Imbalance** | High (Drift too fast feels like Tamagotchi; drift too slow feels static) | Automated test suite checking numerical drift at 7 days of simulated visits and user-perceivable drift at 21 days. |
| **Audio Uncanniness / Synthetic Tone** | High (Synthesized calls sounding harsh or robotic breaks immersion) | Micro-pitch jitter ($\pm 2\%$), soft gain envelope attack/release curves, and sine/triangle oscillator blending in WebAudio. |
| **Multi-Device State Divergence** | Medium (LWW race conditions overwriting vector drift) | Strict enforcement of server-side append-only event log; clients never write personality vectors directly. |
| **Accessibility Surface Regressions** | Medium (Screen-reader narration dropping into announcement/UI register) | CI integration running prose-linter checks against live-region outputs to enforce lowercase naturalist grammar. |

---
*End of Implementation Plan.*
