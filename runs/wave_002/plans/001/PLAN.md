# Pocket Aviary - Implementation Plan

## Scope
**In Scope (v1):**
*   Browser-based, single-horizontal-scene virtual aviary.
*   Single-user accounts via magic-link email authentication.
*   Starts with 2 birds, capping at 7 over the aviary's lifespan.
*   Interactions: return-greeting, listen-in, offer (seed, song fragment, pool), and settle.
*   Server-side simulation ticking (~1/min) tracking slow personality drift and fast mood changes.
*   Auto-generated, read-only field notebook written in naturalist prose.
*   Procedural bird calls (client-side WebAudio) with chorus mixing.
*   Accessibility: screen-reader narration, reduced-motion cross-fades, call captions, WCAG AA contrast, keyboard navigability.
*   Opt-in, read-only social visits via email invitation.

**Out of Scope (v1 & Non-goals):**
*   Native mobile applications.
*   Gamification (no achievements, streaks, levels, scores).
*   Tamagotchi mechanics (no dying, hunger, negative drift on neglect).
*   Social network mechanics (no public discovery, no leaderboards, no shared co-presence).

## Architecture
*   **Client/Server Boundary**: The web client is a stateless rendering and audio engine that interpolates snapshots and emits interaction events. The server is the authoritative source of truth.
*   **Services**:
    *   **API Server**: Handles auth, accepts append-only interaction events (e.g., presence, offers), and serves state snapshots.
    *   **Simulation Engine**: A background worker ticking at ~1 minute intervals, processing the event log to update moods and drift personality vectors.
    *   **Database**: Stores accounts, bird profiles, interaction event logs, and notebook entries.

## Data model
*   **Account**: Synthetic UUID (primary key everywhere), encrypted email, active sessions, social invites.
*   **Bird**: Stable internal ID, species, user-assigned name.
    *   *Personality Vector*: Boldness, social warmth, vocal frequency, plumage saturation, curiosity (all floats).
    *   *Mood*: Enum (wary, content, curious, drowsy, alert).
*   **Presence Event**: Boolean true only when `visibilityState == visible` AND window is focused AND user input (mouse/key) detected within the last few minutes.
*   **Notebook Entry**: Timestamp, naturalist prose string.

## API surface
*   `POST /api/auth/magic-link`: Request sign-in.
*   `GET /api/aviary/snapshot`: Fetch canonical state (bird positions, moods, active animations).
*   `POST /api/events`: Append interaction/presence events to the log.
*   `GET /api/notebook`: Fetch notebook entries.
*   `POST /api/social/invite`: Generate a read-only visit token.
*   `GET /api/social/visit/:token`: Fetch an ambient snapshot for visitors.

## Simulation engine design
*   **Tick**: ~1 minute cadence. Advances the state independent of connected clients.
*   **Drift Function**: A low-pass additive filter over presence and interactions. Traits are monotonic toward expressive (e.g., plumage never desaturates on neglect).
*   **Mood Transitions**: Evaluated per tick based on time of day (local timezone), recent interactions, ambient weather events, and personality baseline.
*   **Event Processing**: The engine consumes the client's append-only log. Last-write-wins is explicitly avoided to prevent device conflict data loss.

## Sync model
*   Multi-device sync is inherently resolved by the architecture. Since all clients are just snapshot-readers and event-appenders, a user on a laptop and a phone simultaneously sees the same canonical state computed by the server. 

## Frontend rendering pipeline
*   **Scene**: Single responsive viewport (canvas/WebGL) preserving aspect ratio to keep all birds in-frame without panning.
*   **Animation**: Smooth interpolation between server snapshots. Idle micro-motions (preening, head tilts) run continuously. 
*   **Reduced-Motion Mode**: Active frame-by-frame animations are replaced with slow, calm CSS/canvas cross-fades between static poses.
*   **Ambient**: Leaves, feathers, and gentle parallax are generated client-side to maintain aliveness without server roundtrips.

## Audio pipeline
*   **Procedural Synthesis**: Implemented via WebAudio API. Uses species-specific motif libraries with dynamic pitch/timing shaped by mood and vocal frequency.
*   **Mixing**: Real-time chorus mixing avoids phase-canceling artifacts of looped tracks.
*   **Listen-in**: Focusing a bird triggers a slow volume ramp-up for its channel and a simultaneous ramp-down for the rest.
*   **Fallback**: If WebAudio fails or is unavailable, the application degrades to graceful silence with call captions enabled.

## Accessibility surfaces
*   **Screen-Reader Narration**: A slow (30-60s) ARIA-live feed describing the aviary in naturalist prose (e.g., "a small grey bird is perched on the front rail").
*   **Call Captions**: Generative prose describing procedural calls, positioned near the bird.
*   **Focus / Keyboard**: Full keyboard traversal across all interactive elements (top bar, birds) with distinct WCAG AA compliant focus outlines.

## Performance budgets and observability
*   **Budgets**:
    *   Initial JS bundle < 2MB (gzipped) to fit within WebAudio and logic constraints.
    *   Time-to-first-bird visible < 500ms on mid-tier mobile (4G) — avoiding load spinners.
    *   60fps idle motion on a 5-year-old laptop.
    *   Zero memory growth over a 30-minute session.
*   **Observability**:
    *   Aggregate telemetry only (request counts, latency, bundle sizes).
    *   P99 tick latency alarms at 5 seconds.
    *   Strict privacy boundary: No per-bird state or interaction logs are sent to the analytics warehouse.

## Rollout
*   **V1**: Shipped web-only with email sign-in, core simulation, and social visits.
*   **Bird Progression**: 2 starters. Offer for a third bird triggers exclusively on account age (e.g., after 3 months), entirely detached from engagement metrics.
*   **Instrumentation**: Start with RUM for TTI and WebAudio failure rates to ensure the core affective experience is landing.

## Risks
*   **Drift Calibration**: Tuning the low-pass filter so personality changes are visible over weeks but imperceptible over days is challenging and requires extended playtesting.
*   **Audio Uncanniness**: Procedural calls sounding robotic or overly repetitive if the motif library or variation algorithms are too shallow.
*   **Presence Accounting**: Incorrect implementation of the strict 3-condition presence check (visible + focused + active) could inflate drift rates across the entire user base.
*   **Accessibility Experience**: Delivering naturalist prose that stays synced with the slow aviary pace without overwhelming the screen-reader queue.