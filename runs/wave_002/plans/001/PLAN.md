# Implementation Plan — Pocket Aviary V1

## 1. Scope and Boundaries

Pocket Aviary is a quiet, browser-based virtual aviary housing two starter birds (with room to grow up to seven based on aviary age) in a single horizontal scene. This v1 implementation plan establishes the architectural and engineering blueprint to deliver the product according to the strict non-goals and core principles set out in the PRD.

### In Scope for V1
- **Platform & Delivery:** Modern web application (Chrome, Safari, Firefox, Edge - last 2 major versions). Web-only, single-page application architecture.
- **Authentication & Accounts:** Email magic-link authentication (15-minute token expiry), synthetic UUID-based account models, single-user canonical aviary per account, multi-device sync, JSON account export, and soft/hard account deletion (30-day soft window).
- **Bird & Simulation Engine:** Server-side simulation tick (~1 minute cadence) managing personality vector persistence, low-pass filter drift model, mood state machine, local-time day/night cycles, and rare ambient weather events.
- **Interactions:** Return-greeting (absence-weighted, randomized stagger), idle attention (presence accounting based on conjunction of visibility, focus, and pointer/key activity), listen-in audio focus re-balance, offer gestures (seed, song fragment, still pool) with per-bird cooldowns, opt-in settle evening gesture with 5-second undo, and read-only auto-generated naturalist Field Notebook.
- **Visual Rendering:** Responsive single horizontal viewport, 3 perch zones (front, middle, back), mid-action load state (quiet sky field fallback), ambient leaf/feather drift, auto-fading top bar chrome, and reduced-motion mode (cross-fade pose sequences replacing frame animations).
- **Audio Architecture:** Procedural WebAudio client-side call synthesis using bird species call-grammar motifs, dynamic chorus mixing, listen-in mix scaling, and graceful silent fallback with automated captions.
- **Social (Optional & Quiet):** Read-only host-invited visits via one-time email link (30-day expiry), host revocation affordances, and private visit audit logging.
- **Accessibility & Observability:** Screen-reader naturalist prose narration (30-60s idle cadence), call captions derived from runtime audio motifs, WCAG AA compliance, keyboard navigation, <2MB initial gzipped JS bundle, <500ms time-to-first-bird render on mid-tier 4G mobile, 60fps desktop rendering, zero memory leaks over 30 minutes, and aggregate operational telemetry with strict PII/per-bird data isolation.

### Non-Goals (Explicitly Out of Scope)
- **No Native Apps:** Web-only; no iOS/Android native implementations or platform-specific bindings.
- **No Gamification:** Zero achievements, streaks, levels, XP, scores, badges, green-dot visit calendars, or visit frequency counters.
- **No Tamagotchi Mechanics:** No bird death, hunger meters, distress states, happiness meters, or negative personality drift on neglect.
- **No Social Network Features:** No public discovery, leaderboards, user profiles, social feeds, mutual visits, live co-presence, shared cursors, or visit comments.
- **No Canned Audio or Stock Loading Spinners:** No audio loops, pre-recorded sample overlays, or traditional loading spinners.

---

## 2. System Architecture & Component Topology

The architecture separates client rendering from server-side state simulation to ensure multi-device consistency, presence honesty, and zero client-owned personality states.

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                                 Client Browser                                  │
│                                                                                 │
│   ┌───────────────────────────┐        ┌────────────────────────────────────┐   │
│   │   WebGL / Canvas Renderer │        │      WebAudio Synthesizer Engine   │   │
│   │ (Scene, Parallax, Poses)  │        │ (Procedural Grammar, Chorus Mix)   │   │
│   └─────────────▲─────────────┘        └─────────────────▲──────────────────┘   │
│                 │                                        │                      │
│                 └──────────────────┬─────────────────────┘                      │
│                                    │ State Snapshots & Interp                   │
│                        ┌───────────┴──────────┐                                 │
│                        │   Client App Core    │                                 │
│                        │ (Presence Monitor,   │                                 │
│                        │  Keyboard & A11y)    │                                 │
│                        └───────────▲──────────┘                                 │
└────────────────────────────────────┼────────────────────────────────────────────┘
                                     │ HTTPS / WSS API
┌────────────────────────────────────┼────────────────────────────────────────────┐
│ Server Infrastructure              │                                            │
│                                    │                                            │
│                        ┌───────────▼──────────┐                                 │
│                        │   API Gateway & Auth │                                 │
│                        │ (Magic Link, JWT)    │                                 │
│                        └───────────┬──────────┘                                 │
│                                    │                                            │
│        ┌───────────────────────────┼───────────────────────────┐                │
│        │                           │                           │                │
│ ┌──────▼─────────────┐   ┌─────────▼────────────┐   ┌──────────▼──────────────┐ │
│ │ Append-Only Event  │   │  Simulation Engine   │   │  Read-Only Snapshot     │ │
│ │ Ingestion Service  │   │   (~1 min Ticks)     │   │     CDN / Cache         │ │
│ └──────┬─────────────┘   └─────────┬────────────┘   └──────────▲──────────────┘ │
│        │                           │                           │                │
│        └───────────────────┐       │       ┌───────────────────┘                │
│                            ▼       ▼       │                                    │
│                     ┌────────────────────────┐                                  │
│                     │  Relational Storage    │                                  │
│                     │  (PostgreSQL / Redis)  │                                  │
│                     └────────────────────────┘                                  │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### Key Architectural Patterns
1. **Unidirectional Event Pipeline:** Clients submit interaction events (presence pings, offers, listen-in toggles, settle commands) to an append-only event log.
2. **Server-Authored State Engine:** A server-side background simulation worker ticks once per minute per active aviary, processing event logs, calculating personality vector deltas, updating mood states, generating Field Notebook entries, and writing canonical snapshots.
3. **Snapshot Pull & Interpolation:** Clients fetch lightweight JSON state snapshots upon initial load, tab visibility change, system resume, and keepalive intervals. Renderers smoothly interpolate bird positions and states between snapshots.

---

## 3. Data Schema & Persistence Model

Database entities use synthetic UUID primary keys. PII (email) is strictly isolated and encrypted at rest.

### 3.1 Accounts Table (`accounts`)
- `id` (UUID, Primary Key) — Synthetic account identifier used in all internal systems.
- `email_encrypted` (VARCHAR/TEXT, Encrypted) — User email for magic link authentication.
- `created_at` (TIMESTAMPTZ) — Account creation timestamp.
- `status` (ENUM: `active`, `soft_deleted`) — Current account status.
- `deletion_requested_at` (TIMESTAMPTZ, Nullable) — Timestamp when soft deletion was initiated.
- `settings` (JSONB) — User settings (`reduced_motion_override`, `captions_enabled`, `visit_notifications_enabled`).

### 3.2 Aviaries Table (`aviaries`)
- `id` (UUID, Primary Key) — Unique aviary ID.
- `account_id` (UUID, FK -> `accounts.id`) — Owner account.
- `created_at` (TIMESTAMPTZ) — Used to compute aviary age for bird adoption eligibility.
- `settled_at` (TIMESTAMPTZ, Nullable) — Timestamp when settled state was activated.

### 3.3 Birds Table (`birds`)
- `id` (UUID, Primary Key) — Stable internal bird identifier (persists across renames and syncs).
- `aviary_id` (UUID, FK -> `aviaries.id`) — Parent aviary.
- `species_id` (VARCHAR) — Reference to species grammar/visual definition (one of 6 initial species).
- `name` (VARCHAR) — User-assigned bird name (default assigned at adoption, fully editable).
- `adopted_at` (TIMESTAMPTZ) — Adoption timestamp.
- `personality_vector` (JSONB) — Hidden numerical traits (Range `[0.0, 1.0]`):
  - `boldness`: Float
  - `social_warmth`: Float
  - `vocal_frequency`: Float
  - `plumage_saturation`: Float
  - `curiosity`: Float
- `current_mood` (ENUM: `wary`, `content`, `curiosity`, `drowsy`, `alert`) — Fast-timescale state.
- `current_perch_zone` (ENUM: `front`, `middle`, `back`) — Position signal derived from mood & boldness.

### 3.4 Interaction Event Log (`interaction_events`)
- `id` (UUID, Primary Key)
- `aviary_id` (UUID, FK -> `aviaries.id`)
- `bird_id` (UUID, Nullable, FK -> `birds.id`)
- `event_type` (ENUM: `presence_ping`, `listen_in_start`, `listen_in_end`, `offer_made`, `settle_triggered`, `settle_undone`)
- `payload` (JSONB) — Offer type (`seed`, `song_fragment`, `still_pool`), presence duration seconds, etc.
- `created_at` (TIMESTAMPTZ)

### 3.5 Field Notebook Entries Table (`notebook_entries`)
- `id` (UUID, Primary Key)
- `aviary_id` (UUID, FK -> `aviaries.id`)
- `content` (TEXT) — Naturalist prose observation in lowercase present tense.
- `created_at` (TIMESTAMPTZ)

### 3.6 Visits & Invites Table (`visits`)
- `id` (UUID, Primary Key)
- `aviary_id` (UUID, FK -> `aviaries.id`)
- `visitor_email_encrypted` (VARCHAR/TEXT, Encrypted) — Visitor identifier.
- `invite_token_hash` (VARCHAR) — One-time access token hash.
- `status` (ENUM: `pending`, `active`, `revoked`, `expired`)
- `created_at` (TIMESTAMPTZ)
- `expires_at` (TIMESTAMPTZ) — Default `created_at + 30 days`.
- `last_visited_at` (TIMESTAMPTZ, Nullable)

---

## 4. Simulation Engine & Personality Drift Math

The simulation tick executes server-side every 60 seconds per active aviary.

```
                    ┌────────────────────────────────────────┐
                    │      Server Simulation Tick (~1 min)   │
                    └───────────────────┬────────────────────┘
                                        │
                                        ▼
                    ┌────────────────────────────────────────┐
                    │ Fetch Unprocessed `interaction_events` │
                    └───────────────────┬────────────────────┘
                                        │
                                        ▼
                    ┌────────────────────────────────────────┐
                    │   Compute Presence-Time & Attentive    │
                    │         Interactions Delta             │
                    └───────────────────┬────────────────────┘
                                        │
                                        ▼
                    ┌────────────────────────────────────────┐
                    │ Apply Monotonic Low-Pass Drift Filter  │
                    │   (Traits shift UP on presence;        │
                    │    NEVER decay on neglect)             │
                    └───────────────────┬────────────────────┘
                                        │
                                        ▼
                    ┌────────────────────────────────────────┐
                    │   Evaluate Fast-Timescale Mood State   │
                    │   (Local time, weather, events, vector)│
                    └───────────────────┬────────────────────┘
                                        │
                                        ▼
                    ┌────────────────────────────────────────┐
                    │   Evaluate Sparse Notebook Generator   │
                    │   (Generate entry if milestone hit)    │
                    └───────────────────┬────────────────────┘
                                        │
                                        ▼
                    ┌────────────────────────────────────────┐
                    │ Write Canonical State Snapshot & Commit│
                    └────────────────────────────────────────┘
```

### 4.1 Personality Vector Drift Equations
Drift is monotonic toward expressive and modeled using an exponential moving low-pass filter:

$$\Delta T_i = \alpha \cdot f(\text{Inputs}) \cdot (1.0 - T_i)$$

Where:
- $T_i$ is trait $i \in \{\text{boldness}, \text{social\_warmth}, \text{vocal\_frequency}, \text{plumage\_saturation}, \text{curiosity}\}$.
- $\alpha \approx 1.5 \times 10^{-5}$ per presence-minute (calibrated such that measurable instrument drift occurs at ~1 week / 100 presence-minutes, and visible user drift occurs at ~3 weeks / 300 presence-minutes).
- $f(\text{Inputs}) \ge 0$. Specifically:
  - Presence-time adds base drift across all traits.
  - `listen_in` targeting bird $B$ applies $1.5\times$ multiplier to $B$'s `social_warmth` and `vocal_frequency`.
  - Accepted `offer` events boost `curiosity` and `boldness`.
- **Monotonicity Enforcement:** If $f(\text{Inputs}) = 0$ (neglect/absence), $\Delta T_i = 0$. Traits never decrease.

### 4.2 Mood State Machine Transitions
Moods transition dynamically based on:
1. **Local Time of Day:** `drowsy` probability increases near dusk/night; `alert` spikes at local morning.
2. **Recent Events:** Seed accepted $\rightarrow$ `content`; song fragment $\rightarrow$ `curiosity`; alarm call $\rightarrow$ `wary`.
3. **Personality Vector Buffers:** High `boldness` reduces `wary` transition probability by $50\%$.
4. **Session Continuity:** Mood at session start equals mood at last snapshot modified by time-decay toward base mood during long absences.

---

## 5. Client Rendering, Audio & Animation Pipeline

### 5.1 Rendering Engine & Layout
- **Canvas/WebGL Pipeline:** Rendered to a single full-bleed container using a HTML5 Canvas 2D/WebGL context with DPR auto-scaling.
- **Perch Allocation:** 3 perch zones (front, middle, back). Birds select perches autonomously based on current mood and `boldness`. No user drag-and-drop.
- **Initial Load & Pre-Roll:** Scene initializes immediately displaying birds mid-action. When fetching initial state over slow connections, a quiet sky field is rendered instead of a loading spinner.
- **Top Bar UI:** Minimalist overlay containing Settings, Accessibility, Field Notebook, and Offer icons. Auto-fades to $5\%$ opacity after 3 seconds of cursor inactivity; restores on mouse movement or keypress.

### 5.2 Micro-Motion & Reduced-Motion Architecture
- **Standard Mode:** Smooth skeletal/mesh deformation or frame interpolation driving preening, scanning, weight shifting, and head tilts at 60fps.
- **Reduced-Motion Mode (`prefers-reduced-motion`):** Replaces continuous bone animations with slow cross-fades (1.5s transition duration) between static pose keyframes. Removes leaf/feather parallax drift while maintaining color palette shifts.

### 5.3 Procedural Audio Synthesis (WebAudio)
- **Motif Synthesizer:** Synthesizes calls using WebAudio `OscillatorNode`, `BiquadFilterNode`, and `GainNode` envelopes based on species call grammars. No pre-recorded MP3/WAV loops are downloaded.
- **Chorus & Listen-In Mixing:**
  - Standard chorus applies spatial panning and randomized motif offsets.
  - `listen_in` ramps target bird gain to $1.0$ over $1.2\text{s}$ while scaling non-targeted bird gains down to an ambient floor ($0.15$). Muting non-targeted birds entirely is strictly prohibited.
- **Audio Fallback:** If WebAudio is unavailable or blocked by browser policy, the application runs silently with call captions enabled automatically.

---

## 6. Accessibility & Interaction Specifications

### 6.1 Naturalist Screen-Reader Narration
- Delivered via a dedicated `aria-live="polite"` region.
- **Cadence:** Throttled to one update every 30–60 seconds during idle state; immediate dispatch on user actions (offer, listen-in, return-greeting).
- **Prose Style:** Naturalist field notes (e.g., `"a small grey bird is perched on the front rail, calling softly. it is morning in the aviary."`). State lists (e.g., `"Bird 1: Perch 2"`) are strictly forbidden.

### 6.2 Call Captions
- Displayed as soft overlay text near the calling bird with fade-in/fade-out transitions.
- Derived dynamically from active synthesis parameters (e.g., `"a soft three-note rise"`, `"a low trill, paused, low trill again"`).

### 6.3 Presence Accounting Contract
A `presence_ping` event is emitted every 30 seconds **only if all three conditions hold simultaneously**:
1. `document.visibilityState === 'visible'`
2. `document.hasFocus() === true`
3. Pointer movement or keypress registered within the last 3 minutes.

If any condition fails, presence tracking stops immediately.

---

## 7. Performance Budgets & Technical Constraints

| Metric / Dimension | Target / Upper Bound | Enforcement Mechanism |
|---|---|---|
| **Initial JS Bundle (gzipped)** | $< 2.0 \text{ MB}$ | CI build threshold failure check |
| **Time-to-First-Bird Render** | $< 500 \text{ ms}$ (4G mid-tier mobile) | Edge snapshot hydration & asset pre-compilation |
| **Frame Rate (Idle)** | $60 \text{ fps}$ (5-year-old laptop) | WebGL batching & requestAnimationFrame throttling |
| **Memory Growth (30 min session)** | $0 \text{ bytes}$ uncollected growth | Automated CI Puppeteer heap snapshot testing |
| **Simulation Tick Latency (p99)** | $< 5.0 \text{ seconds}$ | Datadog/CloudWatch service alarms |
| **Browser Compatibility** | Chrome, Safari, Firefox, Edge (last 2 major versions) | Automated Playwright browser test matrix |

---

## 8. Release Strategy & Rollout Plan

### 8.1 Phased Delivery Pipeline
1. **Phase 1: Core Engine & Audio Proof-of-Concept**
   - Implement WebAudio motif synthesizer, species grammar definitions, and basic WebGL canvas renderer.
   - Build server-side simulation tick, PostgreSQL schema, and magic-link authentication.
2. **Phase 2: Interaction & Accessibility Integration**
   - Wire return-greeting logic, presence accounting monitor, listen-in audio re-balancer, and offer gestures.
   - Implement `aria-live` naturalist narration engine, call captioning, and reduced-motion cross-fade renderer.
3. **Phase 3: Multi-Device Sync & Social Invites**
   - Validate server-authored vector delta sync across multi-device client sessions.
   - Build read-only visit invitation flows and visit logging.
4. **Phase 4: Optimization, Auditing & Launch**
   - Enforce $<2\text{MB}$ bundle size constraint and $<500\text{ms}$ rendering benchmarks.
   - Conduct security audit on magic links, synthetic UUID isolation, and PII encryption.

### 8.2 Aviary Growth Pacing
- **Day 1 Adoption:** 2 starter birds selected automatically from the 6-species pool.
- **Age-Based Expansion:** Third bird offered at 60 days of aviary age; additional birds offered at multi-month intervals up to the hard cap of 7 birds.

---

## 9. Risk Matrix & Mitigation Strategies

| Risk Description | Severity / Likelihood | Architectural Mitigation |
|---|---|---|
| **Presence Spoofing / Background Tab Drift Inflation** | High / Medium | Strict 3-part presence contract enforced client-side with server-side sanity validation on ping intervals. |
| **Personality Vector Sync Corruption** | High / Low | Clients never submit absolute trait values. Only server simulation ticks execute additive delta updates from event logs. |
| **Audio Uncanniness / Looped Sound Perception** | Medium / Medium | 100% procedural WebAudio call synthesis using pitch/timing perturbation; zero static audio files. |
| **Gamification Leakage / Product Identity Shift** | High / Medium | Automated PR code reviews verifying absence of counters, streak logic, badges, or toast announcement banners. |
| **PII Leakage in Telemetry & Logs** | High / Low | Synthetic UUID assigned at account creation used exclusively in downstream tables, logs, and telemetry. |
