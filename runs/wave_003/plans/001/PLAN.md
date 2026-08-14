# Pocket Aviary — Architecture and Implementation Plan (v1)

## 1. Executive Summary & Product Scope

### 1.1 Headline & Core Intent
Pocket Aviary is a quiet, browser-based virtual aviary where users adopt a small group of procedural birds (2 starters, scaling up to 7 based on aviary age). The product centers on observational attention rather than custodial maintenance or gamified retention loops. Aliveness is expressed through procedural calls, subtle mood-based idle animations, and server-side personality drift driven by honest presence accounting.

### 1.2 Scope Boundaries (In-Scope for v1)
- **Client**: Single-page browser application built for modern evergreen browsers (Chrome, Safari, Firefox, Edge). Responsive horizontal viewport without panning/zooming.
- **Account & Auth**: Single-user accounts, passwordless magic-link sign-in (15-minute expiry), revocable device sessions, synthetic internal UUIDs, data export, 30-day soft deletion.
- **Bird Mechanics**: Hidden 5-dimensional personality vectors per bird; monotonic-expressive drift via low-pass filtering on server ticks (~60s interval); fast-timescale mood state machine; procedural WebAudio synthesis and chorus management; return-greeting engine; 6-species pool.
- **Interactions**: Idle presence detection (visibility + window focus + user activity), listen-in mix focus, 3 offer types (seed, song fragment, still pool) with per-bird cooldowns, top-bar settle gesture (with 5-second undo), field notebook generator (sparse naturalist prose).
- **Social (Optional & Quiet)**: One-time tokenized email visit invitations (read-only ambient observer, no co-presence, no visitor drift impact, revocable, 30-day expiry, silent visit logging).
- **Accessibility & Performance**: WCAG AA compliance, semantic ARIA running prose narration (30–60s pacing), dedicated reduced-motion mode (cross-fading still poses instead of disabling motion), procedural call captioning, initial JS bundle <2MB gzipped, Time-to-First-Bird visible <500ms, 60fps steady rendering, zero memory growth over 30 minutes.

### 1.3 Strict Non-Goals (Explicit Refusals)
- **No Native Mobile Apps**: Web-only. No React Native / Flutter / Swift / Kotlin wrappers.
- **No Gamification**: No streaks, scores, levels, badges, achievements, adoption counters, green-dot visit calendars, or retention push notifications.
- **No Tamagotchi Mechanics**: Birds never die, starve, or show distress. No hunger or happiness decay meters. Neglect results in ambient quietness, never penalty or negative trait drift.
- **No Social Network Surfaces**: No profiles, followers, public discovery directories, visit comments, chat overlays, leaderboards, or visitor avatars.
- **No Direct Trait Exposure**: Personality vector values are strictly hidden server-side (no stats screens, debug toggles, or telemetry leakage).
- **No Recorded Audio Fallback**: No canned MP3/OGG audio loops; fallback for missing WebAudio is graceful silence with procedural captions enabled.

---

## 2. System Architecture & Boundaries

```
                           +-----------------------------------------------+
                           |                 Client Browser                |
                           |  +-----------------------------------------+  |
                           |  | Canvas / WebGL Scene + Micro-Motion     |  |
                           |  | WebAudio Procedural Synth & Chorus Mixer|  |
                           |  | Presence Monitor (Vis + Focus + Input)  |  |
                           |  | Live ARIA Narration & Call Captions     |  |
                           |  +-----------------------------------------+  |
                           +-----------------------+-----------------------+
                                                   |
                             HTTPS / JSON Snapshots| Append-Only Event Log
                             CDN Cached Sky / Shell| Session Ping / Revoke
                                                   v
                           +-----------------------------------------------+
                           |          Edge API Gateway & Ingress           |
                           |  - Magic-link verification & session tokens   |
                           |  - Synthetic UUID resolution (PII isolation)  |
                           |  - Append-only event ingestion buffer         |
                           |  - Read snapshot cache (E-Tag / SWR)          |
                           +-----------------------+-----------------------+
                                                   |
                         +-------------------------+-------------------------+
                         v                                                   v
+------------------------------------+             +------------------------------------+
|     Simulation Engine (Tick Worker)|             |       Relational Database (OLTP)   |
| - Runs ~60s cadence per aviary     |             | - accounts (UUID, enc_email, state)|
| - Ingests batched interaction log  |             | - aviaries (id, account_id, age)   |
| - Computes monotonic drift deltas  |             | - birds (id, vector, mood, perches)|
| - Evaluates bird-to-bird chorus    |             | - event_log (append-only stream)   |
| - Generates sparse notebook prose  |             | - notebook_entries (prose records) |
| - Writes canonical state snapshot  |             | - visit_invitations (tokens, logs) |
+------------------------------------+             +------------------------------------+
```

### 2.1 Service Decomposition
1. **Edge API Gateway (Stateless)**:
   - Terminates TLS, validates Bearer session tokens, verifies magic links.
   - Enforces synthetic Account ID substitution: resolves incoming email to internal UUID; never logs PII.
   - Routes read-snapshot queries and accepts interaction events into the ingest stream.
2. **Simulation Worker Fleet (Stateful/Partitioned Tick Engine)**:
   - Processes active and background aviaries on a ~60-second cron/tick schedule.
   - Partitioned by `account_id` hash (consistent hashing).
   - Ingests presence pings, listen-in spans, and offer gestures; applies drift formulas; transitions mood states; emits field notebook observations; writes atomic snapshot updates.
3. **Database Layer (PostgreSQL with strict isolation)**:
   - Stores accounts, aviary records, bird state, event journals, and notebook entries.
   - PII table (`account_auth`) encrypted at rest with separate key management.
4. **Static CDN / Edge Delivery**:
   - Delivers static application assets (<2MB gzipped bundle, SVG vector silhouettes, motif definitions).
   - Serves initial shell with edge-injected snapshot bootstrap to satisfy <500ms first bird render.

### 2.2 Client/Server Boundary Principles
- **Server**: Sole author and writer of canonical state, personality vectors, mood transitions, and notebook entries.
- **Client**: Pure rendering, synthesis, and observation engine. Interpolates between server snapshots, synthesizes audio in real-time from parametric motifs, and reports raw interaction events.
- **Visitor Boundary**: Visitor sessions authenticate via one-time signed tokens, pull read-only snapshots, and are blocked from submitting presence or interaction events.

---

## 3. Data Models & Database Schemas

### 3.1 Relational Schema (PostgreSQL DDL)

```sql
-- Accounts & Authentication (PII Isolation)
CREATE TABLE accounts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    status VARCHAR(20) NOT NULL DEFAULT 'active', -- active, soft_deleted
    soft_deleted_at TIMESTAMPTZ NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE account_auth (
    account_id UUID PRIMARY KEY REFERENCES accounts(id) ON DELETE CASCADE,
    email_encrypted BYTEA NOT NULL,
    email_hash VARCHAR(64) UNIQUE NOT NULL, -- Blind index (HMAC) for lookup
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE sessions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    device_name VARCHAR(100) NOT NULL,
    token_hash VARCHAR(64) UNIQUE NOT NULL,
    last_seen_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE magic_links (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    token_hash VARCHAR(64) UNIQUE NOT NULL,
    expires_at TIMESTAMPTZ NOT NULL,
    consumed_at TIMESTAMPTZ NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Aviary and Bird Entities
CREATE TABLE aviaries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    account_id UUID UNIQUE NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    settled_at TIMESTAMPTZ NULL,
    weather_state VARCHAR(20) NOT NULL DEFAULT 'clear',
    weather_expires_at TIMESTAMPTZ NULL,
    last_tick_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE birds (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    species_id VARCHAR(32) NOT NULL, -- e.g. 'warbler', 'nuthatch', 'nightjar'
    name VARCHAR(50) NOT NULL,
    adopted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    
    -- Hidden Personality Vector (Normalized [0.000, 1.000])
    boldness NUMERIC(5,4) NOT NULL DEFAULT 0.2000,
    social_warmth NUMERIC(5,4) NOT NULL DEFAULT 0.2000,
    vocal_frequency NUMERIC(5,4) NOT NULL DEFAULT 0.2000,
    plumage_saturation NUMERIC(5,4) NOT NULL DEFAULT 0.2000,
    curiosity NUMERIC(5,4) NOT NULL DEFAULT 0.2000,

    -- Dynamic State
    current_mood VARCHAR(20) NOT NULL DEFAULT 'content', -- wary, content, curious, drowsy, alert
    mood_updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    current_perch_zone VARCHAR(10) NOT NULL DEFAULT 'middle', -- front, middle, back
    last_greeting_at TIMESTAMPTZ NULL,
    last_offer_accepted_at TIMESTAMPTZ NULL
);

-- Append-Only Interaction Event Log
CREATE TABLE interaction_events (
    id BIGSERIAL PRIMARY KEY,
    account_id UUID NOT NULL REFERENCES accounts(id) ON DELETE CASCADE,
    bird_id UUID NULL REFERENCES birds(id) ON DELETE SET NULL,
    event_type VARCHAR(32) NOT NULL, -- 'presence_span', 'listen_in_span', 'offer_gesture', 'settle_gesture'
    duration_seconds INT NULL,
    payload JSONB NOT NULL DEFAULT '{}'::jsonb,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    processed_at TIMESTAMPTZ NULL
);
CREATE INDEX idx_events_unprocessed ON interaction_events(account_id, processed_at) WHERE processed_at IS NULL;

-- Field Notebook Observations
CREATE TABLE field_notebook_entries (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    entry_date DATE NOT NULL,
    prose_text TEXT NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_notebook_aviary_date ON field_notebook_entries(aviary_id, created_at DESC);

-- Visit Invitations & Silent Logs
CREATE TABLE visit_invitations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    aviary_id UUID NOT NULL REFERENCES aviaries(id) ON DELETE CASCADE,
    invite_token_hash VARCHAR(64) UNIQUE NOT NULL,
    recipient_email_hash VARCHAR(64) NOT NULL,
    recipient_email_encrypted BYTEA NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'active', -- active, revoked, expired
    expires_at TIMESTAMPTZ NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE visit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invitation_id UUID NOT NULL REFERENCES visit_invitations(id) ON DELETE CASCADE,
    started_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    duration_seconds INT NOT NULL DEFAULT 0
);
```

---

## 4. API Surface & Protocols

### 4.1 Authentication & Account Management
- `POST /api/v1/auth/magic-link`: Request 15-minute login token for email. Response: `{"status": "ok"}`.
- `POST /api/v1/auth/verify`: Consume magic token; returns HTTP-only Secure session cookie and `session_token`.
- `GET /api/v1/account/sessions`: List active devices (`id`, `device_name`, `last_seen_at`).
- `DELETE /api/v1/account/sessions/:id`: Revoke session.
- `POST /api/v1/account/export`: Triggers export snapshot generation and emails link.
- `POST /api/v1/account/delete`: Initiates 30-day soft deletion.
- `POST /api/v1/account/restore`: Cancels soft deletion within 30-day window.

### 4.2 Aviary State & Stream
- `GET /api/v1/aviary/state`: Fetch current canonical aviary snapshot.
  ```json
  {
    "aviary_id": "8f3b1c94-...",
    "server_time": "2026-08-13T19:20:00Z",
    "local_time_offset_sec": -25200,
    "weather": "clear",
    "is_settled": false,
    "birds": [
      {
        "id": "e2a149b5-...",
        "species_id": "warbler",
        "name": "Pip",
        "mood": "content",
        "perch_zone": "front",
        "plumage_saturation": 0.342,
        "vocal_frequency": 0.410
      },
      {
        "id": "b18d2077-...",
        "species_id": "nuthatch",
        "name": "Wren",
        "mood": "wary",
        "perch_zone": "back",
        "plumage_saturation": 0.215,
        "vocal_frequency": 0.280
      }
    ]
  }
  ```
  *(Note: Boldness, curiosity, and social warmth scalars remain strictly hidden on the server).*

### 4.3 Interaction Event Submission
- `POST /api/v1/aviary/events`: Batch append interaction events.
  ```json
  {
    "events": [
      {
        "type": "presence_span",
        "duration_seconds": 120,
        "timestamp": "2026-08-13T19:18:00Z"
      },
      {
        "type": "listen_in_span",
        "bird_id": "e2a149b5-...",
        "duration_seconds": 45,
        "timestamp": "2026-08-13T19:19:00Z"
      },
      {
        "type": "offer_gesture",
        "bird_id": "e2a149b5-...",
        "offer_kind": "seed",
        "timestamp": "2026-08-13T19:19:30Z"
      }
    ]
  }
  ```

### 4.4 Social & Field Notebook
- `GET /api/v1/aviary/notebook`: Fetch naturalist observation log entries (paginated, descending date).
- `POST /api/v1/social/invites`: Create visitor invite for an email address.
- `GET /api/v1/social/invites`: List active invitations and silent visit logs.
- `DELETE /api/v1/social/invites/:id`: Immediately revoke visit invitation.
- `GET /api/v1/visit/:token`: Fetch read-only ambient snapshot for invited guest.

---

## 5. Simulation Engine Design

### 5.1 Server-Side Tick Architecture
The simulation worker runs an atomic cycle per aviary every 60 seconds:
1. **Acquire Aviary Lease**: Distributed lock via PostgreSQL `FOR UPDATE SKIP LOCKED`.
2. **Consume Interaction Buffer**: Fetch all unprocessed `interaction_events` for the aviary.
3. **Presence Validation**: Verify that recorded presence spans meet all 3 criteria (document visible, window focused, activity verified within 3-minute sliding window). Reject fraudulent/bloated spans.
4. **Compute Monotonic Drift Deltas**:
   $$\Delta T = \alpha \cdot \sum \text{weight}(E) \cdot (1.0 - T)$$
   Where $T \in [0, 1]$ is trait value, and $\Delta T \ge 0$ strictly.
   - *Presence Time*: $\Delta \text{Boldness} += k_p \cdot \text{presence\_seconds}$, $\Delta \text{SocialWarmth} += k_p \cdot \text{presence\_seconds}$.
   - *Listen-In Duration*: $\Delta \text{SocialWarmth} += k_l \cdot \text{listen\_seconds}$, $\Delta \text{VocalFrequency} += k_l \cdot \text{listen\_seconds}$.
   - *Offer Accepted*: $\Delta \text{Curiosity} += k_o$, $\Delta \text{Boldness} += k_{o\_near}$.
   - *Plumage Saturation*: Slowly steps upward with cumulative presence time; never decrements.
   - *Calibration*: 1 week of daily 15-min presence yields ~0.03 delta (instrument-detectable); 3 weeks yields ~0.10 delta (visibly distinct perch preference and plumage luster).
5. **Mood State Transitions**:
   - Evaluate local time of day (dusk $\to$ drowsy, dawn $\to$ alert).
   - Evaluate recent weather (rain $\to$ dampens vocal frequency, wind $\to$ shifts wary/alert).
   - Evaluate bird-to-bird social contagion (wary alarm call has 35% chance to shift neighbor from content $\to$ wary).
   - Perch placement calculation: Front perch probability $= f(\text{Boldness}, \text{Mood})$.
6. **Sparse Notebook Generation**:
   - Evaluates aviary events over the preceding 48–72 hours.
   - If a distinct observation threshold is met (e.g. "Pip greeted first 3 days in a row" or "Wren perched near front during morning rain"), emit 1 entry in lowercase naturalist voice.
   - Rate limit: at most 1 entry per 2–3 days.
7. **Age-Based Adoption Evaluation**:
   - Check `aviary.created_at`. If aviary age surpasses milestones (e.g., 60 days for 3rd bird, 120 days for 4th bird) and current bird count $< 7$, generate a candidate arrival event for next user session.
8. **Write Canonical Snapshot & Release Lease**.

---

## 6. Client Rendering & Motion Engine

### 6.1 Scene Composition & Coordinate System
- **Viewport Canvas**: Canvas2D / WebGL scene container maintaining a fixed 16:9 to 21:9 responsive aspect ratio.
- **Layers**:
  1. *Sky & Ambient Lighting*: Smooth gradient keyed to solar elevation angle computed from local timezone.
  2. *Far Background*: Distant tree silhouettes and subtle parallax mountain/canopy plane.
  3. *Middle Ground (Perches & Birds)*: Three depth zones (Back, Middle, Front). Birds are rendered using procedural SVG path geometry with feather layering.
  4. *Foreground Particles*: Client-side deterministic leaf/feather drift generated by noise functions.
  5. *Top-Bar Chrome*: HTML DOM overlay with CSS opacity fade on idle.

### 6.2 Instant-On Mid-Action Initialization
To satisfy the <500ms time-to-first-bird and "no wakeup/loading spinner" requirement:
- Client receives initial aviary snapshot embedded in HTML or via immediate fetch.
- Client initializes animation clocks with an offset derived from `server_time`, placing bird wings, preen angles, and particle positions mid-cycle on Frame 0.
- If network snapshot is delayed, client renders the calm ambient sky immediately without spinners.

### 6.3 Reduced-Motion Implementation
- Triggered by `prefers-reduced-motion: reduce` or accessibility setting.
- Continuous skeletal animation curves are disabled.
- Motion is replaced by discrete pose cross-fades (duration 800ms, opacity ease-in-out).
- Flight transitions between perches render as a slow 1.2s cross-dissolve from origin perch to destination perch.
- Ambient drifting leaves/feathers are suppressed.

---

## 7. Audio Synthesis & Chorus Pipeline

### 7.1 WebAudio Node Graph
Each bird allocates a dedicated parametric synthesis voice:

```
[Motif Generator (Pitch/Timing)] 
              |
      [Oscillator Array] (Sine + Sawtooth with subtle FM modulation)
              |
      [Formant Filter (Bandpass)] (Species acoustic tract modeling)
              |
      [ADSR Gain Envelope]
              |
      [Bird Mixer Gain Node] <---+ [Listen-In Attenuation Controller]
              |
      [Spatial / Panning Node] (Mapped to Front/Middle/Back perch X/Z)
              |
     [Master Aviary Bus] ---> [Dynamic Compressor] ---> [AudioContext.destination]
```

### 7.2 Procedural Call Grammar
- **Motif Library**: Each of the 6 species has 4 fundamental acoustic primitives (chirp, trill, whistle, click).
- **Runtime Variation**: Timing jitter ($\pm 8\%$), pitch bend variance ($\pm 15 \text{ cents}$), and note count variations ensure no two calls are identical.
- **Vocal Frequency Modulation**: Birds with high vocal frequency trigger calls more frequently unobserved and have shorter response latencies to neighboring calls.

### 7.3 Listen-In Mix Dynamics
- When a user focuses bird $B_k$:
  - Gain of $B_k$ ramps to $+3\text{dB}$ over 1.8 seconds (exponential ramp).
  - Gain of all other birds $B_{j \ne k}$ ramps down to $-14\text{dB}$ (never zero/mute) over 2.2 seconds.
- On disengage, all birds return smoothly to equal ambient bus levels over 2.5 seconds.

### 7.4 WebAudio Fallback
- If `AudioContext` fails to initialize or permission is blocked:
  - Audio pipeline transitions to silent state (0 CPU cycles).
  - Procedural captioning engine activates automatically, generating realtime visual call text near the calling bird.

---

## 8. Accessibility Surfaces

### 8.1 Screen-Reader Narration (ARIA Live)
- An invisible container `<div aria-live="polite" aria-atomic="true" class="sr-only">` receives prose updates.
- Naturalist voice generation engine constructs observations:
  - *"a warbler perches on the high branch, calling softly."*
  - *"the aviary is quiet in the morning sun; a nuthatch watches from the middle rail."*
- Paced to update once every 30–60 seconds during steady observation, bumping immediately upon user interaction (offer, settle).

### 8.2 Call Captioning
- Opt-in via Accessibility modal (and default-on in audio-fallback mode).
- Formats dynamic captions from current procedural motif parameters:
  - *"a soft three-note rise"*
  - *"a low trill, paused, low trill again"*
- Renders as floating, high-contrast, WCAG AA compliant text tags near the bird's active perch, fading out after 3 seconds.

### 8.3 Keyboard Navigation & High-Contrast Focus
- Full keyboard trap management:
  - `Tab`: Cycle through top-bar actions (Settings, Accessibility, Notebook, Offer, Settle).
  - `Tab` into scene: Focuses active bird on front perch.
  - `Arrow Left / Right`: Cycles focus between birds in left-to-right spatial order.
  - `Enter / Space`: Engages listen-in on focused bird.
  - `Escape`: Disengages listen-in or closes modal overlays.
- Focus outlines use high-contrast dual-stroke styling (`2px solid #ffffff` with `1px outline #1a2e1a`) legible across dawn, midday, and night palettes.

---

## 9. Performance Budgets & Observability

### 9.1 Performance Budgets
| Metric | Budget Target | Verification Strategy |
|---|---|---|
| **Initial JS Bundle (Gzipped)** | $< 2.0\text{ MB}$ | CI bundle-analyzer check on PRs |
| **Time to First Bird Visible** | $< 500\text{ ms}$ (4G, Mid-Tier Mobile) | Web Vitals synthetic Lighthouse run |
| **Render Frame Rate** | 60 FPS steady (5-year old laptop) | Automated headless Chrome rAF audit |
| **Memory Growth (30 min)** | $\Delta \text{Heap} \le 0.0\text{ MB}$ | Puppeteer 30-min soak test verifying buffer reuse |
| **Simulation Tick Latency** | $\text{p99} < 5.0\text{ s}$ | Prometheus server histogram alarm |

### 9.2 Telemetry Boundary & Privacy Architecture
- **Strict Privacy Isolation**:
  - Telemetry pipeline only records system metrics: HTTP status codes, API latencies, worker tick durations, client FPS drops, WebAudio error rates.
  - Telemetry payloads **never** contain Account UUIDs, Bird IDs, trait vectors, interaction counts, or notebook contents.
  - Data warehouse pipelines have zero read permissions on the simulation relational database.

---

## 10. Voice & Tone Enforcement Matrix

| Surface / Context | Permitted Tone | Voice Rules & Style Sample |
|---|---|---|
| **Aviary Scene, Narration, Captions** | Naturalist | Lowercase, present-tense, specific, calm. *"wren is fluffed against the cool air. pip greeted first today."* |
| **Field Notebook Entries** | Naturalist | Lowercase observations of aviary events; never game logs or streak notes. *"tuesday — a long stretch of quiet this morning. pip preened for several minutes."* |
| **Offer & Settle Descriptions** | Naturalist | Quiet gesture prompts. *"offer a seed"*, *"settle for the evening"*. |
| **Auth, Errors, Sync Conflicts** | Matter-of-Fact | Clear standard capitalization, direct English, no charm pretense. *"We couldn't sign you in. The link may have expired. Try requesting a new link."* |
| **Account & Session Management** | Matter-of-Fact | Clear and functional. *"Signed in on macOS (Chrome). Revoke session."* |
| **Accessibility Settings** | Matter-of-Fact | Standard accessible UI labels. *"Enable call captions"*, *"Reduced motion"*. |

---

## 11. Engineering Work Breakdown & Rollout

### 11.1 Implementation Phases
1. **Phase 1: Core Foundation & Data Engine (Weeks 1–3)**
   - Database schemas, magic-link authentication, session token manager, synthetic ID isolation.
   - Simulation tick worker framework with PostgreSQL distributed lease locking.
2. **Phase 2: Bird Behavioral & Audio Engine (Weeks 4–6)**
   - Hidden personality vector low-pass drift filter.
   - Fast-timescale mood state machine & time-of-day solar calculator.
   - WebAudio procedural synthesis voices for 6 starter species & chorus mixer.
3. **Phase 3: Client Rendering & Interaction Surface (Weeks 7–9)**
   - Canvas/WebGL single horizontal viewport scene with 3 perch zones.
   - Presence tracker (visibility + focus + user activity conjunction).
   - Return-greeting procedural variation engine.
   - Listen-in mix controller, offer gestures, and settle transition.
4. **Phase 4: Field Notebook, Social & Accessibility (Weeks 10–11)**
   - Naturalist prose generator & sparse notebook scheduler.
   - Read-only visit invitation subsystem with tokenized links and revocation.
   - Live ARIA running prose narration, procedural captions, and reduced-motion cross-fader.
5. **Phase 5: Performance Optimization, Soak Testing & Launch (Weeks 12–13)**
   - 30-minute memory leak soak tests in Puppeteer.
   - Bundle-size minification under 2MB budget.
   - Multi-device sync verification and chaos testing on connection drops.

### 11.2 Risk Matrix & Mitigations
- **Drift Calibration Drift/Runaway**: If user presence is measured too leniently, birds will max out traits in days. *Mitigation: Conjunction of 3 presence signals enforced strictly on client and verified server-side.*
- **Audio Uncanniness / Phase Cancellation**: Layering multiple procedural calls could sound harsh. *Mitigation: Per-species acoustic formant filters and micro-timing jitter in the WebAudio scheduler.*
- **Multi-Device Drift Overwrite**: Two active tabs on different devices submitting conflicting state. *Mitigation: Additive, server-authored deltas processed via append-only event log; clients never submit absolute trait values.*
- **Accessibility Degradation**: Treating accessibility as a secondary checklist. *Mitigation: Screen-reader running narration and call captioning built and tested in parallel from Phase 1.*
