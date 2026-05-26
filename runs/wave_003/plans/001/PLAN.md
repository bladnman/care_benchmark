# Implementation Plan: Pocket Aviary (v1)

## 1. Scope
Pocket Aviary is a browser-based, low-fidelity virtual aviary designed as an observational relationship rather than a game.

### In Scope (v1)
- **Core Experience**: A single horizontal scene with 2-7 birds.
- **Bird Engine**: Personality vectors, mood system, procedural calls, and monotonic drift based on presence.
- **Interactions**: Return-greetings, "Listen-in" focus, "Offer" gestures, "Settle" session-end, and a read-only "Field Notebook".
- **Infrastructure**: Single-user accounts (magic link), server-side simulation tick, multi-device sync.
- **Social**: Opt-in, read-only visitor invitations via email.
- **Accessibility**: Naturalist prose screen-reader narration, reduced-motion cross-fade rendering, call captioning.
- **Performance**: <2MB initial bundle, <500ms time-to-first-bird, 60fps idle motion.

### Out of Scope (Non-Goals)
- **No Gamification**: No streaks, achievements, levels, XP, or badges.
- **No Custodial Mechanics**: No hunger, death, or distress (not a Tamagotchi).
- **No Social Network**: No profiles, follows, public discovery, or chat.
- **No Native Apps**: Web-only for v1.

---

## 2. Architecture

### Service Shape
- **Frontend (Client)**: A thin rendering layer. It pulls state snapshots and synthesizes audio/visuals. It writes interaction events to the server.
- **Backend (Simulation Service)**: The source of truth. It runs a periodic simulation tick, manages the event log, and maintains canonical bird states.
- **Database**: Stores account metadata (UUIDs), encrypted emails, bird personality vectors, mood states, and the Field Notebook log.

### Client/Server Split
- **Server owns**: Personality vectors, mood transitions, drift calculations, and the canonical simulation time.
- **Client owns**: Procedural synthesis (WebAudio), frame-by-frame interpolation, and presence detection.

### Render Pipeline Boundary
The client receives a state snapshot (e.g., `{ birdId: "b1", mood: "content", perch: "front", callTiming: 0.4 }`) and maps these to visual assets and audio motifs.

---

## 3. Data Model

### Account
- `id`: Synthetic UUID (non-PII).
- `email`: Encrypted string.
- `settings`: Accessibility preferences, notification toggles.

### Bird
- `id`: Stable internal UUID.
- `species`: Identifier from the species pool (6 species).
- `name`: User-assigned string.
- `personality_vector`: `{ boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity }` (scalars).
- `current_mood`: Enum (`wary`, `content`, `curious`, `drowsy`, `alert`).
- `perch_zone`: Enum (`front`, `middle`, `back`).

### Presence & Events
- `event_log`: Append-only list of `{ timestamp, eventType, value }` (e.g., `LISTEN_IN_START`, `OFFER_ACCEPTED`, `PRESENCE_PING`).

### Field Notebook
- `entries`: List of `{ timestamp, prose }` (Naturalist voice).

---

## 4. API Surface

### State Pull
- `GET /state`: Returns the current canonical snapshot of the aviary (birds, mood, weather, time-of-day).

### Event Submission
- `POST /events`: Appends interaction events to the log (e.g., `presence_time`, `offer_type`).

### Auth & Account
- `POST /auth/request-link`: Sends magic link to email.
- `GET /auth/verify`: Validates link, issues session token.
- `POST /account/export`: Triggers JSON export via email.

### Social (Visits)
- `POST /visit/invite`: Sends invitation link to visitor email.
- `DELETE /visit/revoke`: Invalidates a specific invitation.
- `GET /visit/{token}/state`: Read-only state pull for visitors.

---

## 5. Simulation Engine Design

### Server-Side Tick
- **Frequency**: ~1 minute.
- **Process**: 
  1. Read event log since last tick.
  2. Calculate `presence_time` (conjunction of visibility, focus, and activity).
  3. Update personality vectors using the drift function.
  4. Transition mood based on time-of-day, ambient events, and recent interactions.
  5. Advance mood timers.
  6. Write updated state to DB.

### Drift Function
- **Mechanism**: Low-pass filter over presence and interaction signals.
- **Direction**: Monotonic toward expressive (traits only move up).
- **Calibration**:
  - 1 week: Measurable numerical drift in instruments.
  - 3 weeks: Visible drift perceived by the user.

### Call Grammar Runtime
- **Motifs**: Species-specific sound fragments.
- **Variation**: Client-side synthesis varies pitch and timing based on the bird's `vocal_frequency` trait and current mood.

---

## 6. Sync Model
- **Canonical State**: Only the server writes personality/mood.
- **Additive Deltas**: Drift is applied as server-authored deltas based on events, preventing "last-write-wins" conflicts.
- **Multi-Device**: Both laptop and phone pull the same `GET /state` snapshot.

---

## 7. Frontend Rendering Pipeline

### Scene Composition
- **Layout**: Single horizontal scene, no panning/scrolling.
- **Z-Axis**: Three perch zones (Front, Middle, Back).
- **Cycle**: Local-time anchored day/night palette shifts.
- **Weather**: Rare, low-impact ambient events (e.g., passing rain).

### Motion & Transitions
- **Idle Motion**: Continuous, mood-shaped micro-motions (preening, scanning).
- **Entry**: First frame renders birds mid-action; no "wake-up" animations.
- **Interpolation**: Smooth movement between state snapshot positions.

### Reduced-Motion Mode
- Replace frame-by-frame animations with slow cross-fades between still poses.
- Remove ambient leaf/feather drift.

---

## 8. Audio Pipeline

### Procedural Synthesis
- **Technology**: WebAudio API.
- **Process**: Combines motifs and varies them in real-time to create a unique chorus.
- **Listen-in**: Gradual mix re-balance (focused bird rises, others drop to ambient).

### Fallback
- Graceful silence with captions enabled by default if WebAudio is unavailable.

---

## 9. Accessibility Surfaces

### Screen-Reader Narration
- **Voice**: Naturalist prose (lowercase, present-tense).
- **Cadence**: Slow updates (30-60s) to avoid queue flooding.
- **Priority**: Immediate updates for return-greetings and offer reactions.

### Call Captioning
- **Generation**: Runtime mapping of procedural calls to descriptive prose (e.g., "a low trill").
- **Display**: Fading text near the calling bird.

### Keyboard Navigation
- Tab through top bar $\rightarrow$ Focus birds $\rightarrow$ Enter for Listen-in $\rightarrow$ Escape to exit.

---

## 10. Performance Budgets & Observability

### Budgets
- **Bundle**: <2MB gzipped.
- **TTFB (Time to First Bird)**: <500ms on 4G/mid-tier mobile.
- **Runtime**: 60fps on 5-year-old hardware; zero memory growth over 30 mins.

### Observability
- **Metrics**: Page load, TTFB, render-frame timing, simulation-tick latency (p99 < 5s).
- **Privacy**: Per-bird interaction data is strictly excluded from aggregate telemetry.

---

## 11. Rollout Plan
1. **Infrastructure Alpha**: Magic link auth, basic state pull, and simulation tick.
2. **Engine Beta**: Implementation of personality drift and procedural audio synthesis.
3. **Surface Gamma**: Naturalist UI, Field Notebook, and accessibility layers.
4. **V1 Launch**:
   - Start with 2 birds per aviary.
   - Implement age-based bird addition (max 7).
   - Instrument performance metrics from day one.

---

## 12. Risks & Mitigations
- **Drift Calibration**: Risk of drift being too fast (feels like a game) or too slow (feels like a screensaver). *Mitigation*: Use test harness to verify the 1-week/3-week targets.
- **Audio Uncanniness**: Procedural calls sounding robotic. *Mitigation*: Invest in motif variation and avoid looped samples.
- **Sync Correctness**: Potential for event loss. *Mitigation*: Append-only event log with server-side processing.
- **Accessibility Regressions**: Accessibility feeling like a "stripped" version. *Mitigation*: Design narration and reduced-motion as first-class aesthetic experiences.
