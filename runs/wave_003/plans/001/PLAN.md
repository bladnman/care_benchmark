# Pocket Aviary - Implementation Plan

This document outlines the v1 implementation plan for Pocket Aviary, synthesized from the Product Requirements Document.

## Scope
**In Scope for v1:**
- Browser-based virtual aviary (one horizontal scene).
- Start with 2 birds, capping at 7 (availability based on aviary age).
- Single-user accounts with email/magic-link authentication.
- Server-side simulation tick for canonical state, bird personality vectors, and mood.
- Multi-device sync (inherent to the canonical server state).
- Interactions: Return-greeting, Listen-in, Offer, Settle.
- Auto-generated Field Notebook (naturalist prose).
- Exact presence accounting (visibility AND focus AND pointer/key activity).
- Optional, opt-in, read-only social visit feature.
- Full accessibility surfaces (prose screen-reader narration, cross-fade reduced-motion mode, runtime call captions, WCAG AA, keyboard navigation).
- Client-side procedural audio synthesis (WebAudio).

**Out of Scope for v1:**
- Native mobile applications (iOS/Android).
- Gamification (achievements, streaks, score, levels, counters).
- Tamagotchi mechanics (no starvation, death, or negative drift/punishment for neglect).
- Social network surfaces (no discovery, profiles, mutual visits, avatars, comments, leaderboards).
- In-app or push notifications.
- Direct manipulation of birds or scene layout.
- Monetization or paid tiers.

## Architecture
Pocket Aviary operates on a thin-client, thick-server architecture to ensure data continuity and cross-device sync.
- **Client (Browser):** A lightweight rendering and audio synthesis engine. It pulls small state snapshots from the server, interpolates positions, synthesizes procedural audio locally via WebAudio, and appends interaction events to the server. It handles no authoritative simulation state.
- **Server:** The authoritative state machine. A continuous background simulation tick (running ~once per minute) processes the append-only event log from clients, updates personality vectors and moods, logs field notebook entries, and manages day/night/weather states tied to the user's timezone.
- **Database:** Stores user accounts, synthetic UUIDs (never email as primary key), bird identity/personality/mood, and the append-only interaction event log.

## Data Model
All data is keyed by a synthetic UUID; emails are encrypted and stored only once.
- **Account:** Synthetic UUID, encrypted email, verified status, timezone, outstanding visit invites, visit log.
- **Bird:** Stable internal identifier (UUID), species ID, user-assigned name.
- **Personality Vector (Slow-timescale):** Persisted server-side numerical traits: Boldness, Social Warmth, Vocal Frequency, Plumage Saturation, Curiosity. Monotonic drift (only increases).
- **Mood (Fast-timescale):** Enumerated states (e.g., wary, content, curious, drowsy). Persisted across sessions and modulated by the server tick.
- **Presence & Interaction Log:** Append-only log of client events (presence windows, listen-in start/end, offer accepted/rejected, settle triggers).
- **Field Notebook:** Auto-generated text entries (timestamp, prose content).

## API Surface
The API is designed for snapshot-pull and event-append, not CRUD.
- `POST /auth/magic-link`: Request sign-in link.
- `GET /aviary/snapshot`: Returns the current canonical state of the aviary (bird positions, moods, active calls, day/night state). Polled on visibility change and low-frequency keepalive.
- `POST /aviary/events`: Submit an array of interaction events (presence blocks, listen-in, offers, settle). Additive only.
- `POST /aviary/invites`: Create a revocable visit link for a specific email.
- `DELETE /aviary/invites/:id`: Revoke an active or pending visit link.
- `GET /notebook`: Fetch paginated field notebook entries.

## Simulation Engine Design
The simulation runs entirely server-side, uncoupled from client rendering.
- **The Tick:** A cron-like process (approx. 1-minute cadence) processes the `/aviary/events` log.
- **Drift Function:** A low-pass filter over presence-time and interactions. Drift is monotonic toward expressive. It requires ~1 week for instrument-measurable change and ~3 weeks for user-visible change.
- **Mood Transitions:** Evaluated during the tick based on recent events, local time of day, ambient weather, and the bird's personality vector.
- **Call Grammar Runtime:** The server defines the behavioral "intent" to call based on vocal frequency and mood, but the client runtime handles the exact procedural synthesis of the motif.
- **Presence Validation:** Strict gating requires `visibilityState === 'visible'`, window focus, and pointer/key activity within a recent calibration window to count as presence.

## Sync Model
Sync is achieved by eliminating client-side state ownership.
- There is **no client-to-client sync** or **last-write-wins** resolution.
- Both laptop and phone clients read the exact same server snapshot.
- Clients submit interaction events (not state mutations). The server-side tick processes these events in order and calculates additive deltas to the personality vector.

## Frontend Rendering Pipeline
The client visualizes the server snapshot without traditional load states.
- **Initial Load:** HTML is delivered with a tiny inline snapshot. The aviary loads immediately in-motion. No spinners; an empty aviary uses a quiet sky field.
- **Scene Composition:** Three plane perches (front, middle, back) with subtle foreground/background parallax. Birds are placed based on mood/boldness (determined by server).
- **Micro-motion:** Idle animations (preening, head tilts) and ambient particle drift (leaves, feathers) run independently of the server tick.
- **Transitions:** Client interpolates between snapshot coordinates to ensure smooth movement.
- **Reduced-Motion Mode:** Designed aesthetic where animations are replaced with slow cross-fades between static poses. Ambient drift is disabled.

## Audio Pipeline
Audio is the core affective spine and runs client-side to fit bundle budgets and avoid phase-canceling artifacts.
- **Procedural Synthesis:** Uses WebAudio API to synthesize calls from a motif library. No recorded audio loops are used for calls.
- **Chorus Mixing:** Multiple birds calling are mixed in real-time.
- **Listen-in:** Focusing a bird smoothly ramps its gain while smoothly attenuating (but not muting) all other birds and ambient noise.
- **Fallback:** If WebAudio is blocked, the scene plays in graceful silence with procedural captions enabled.

## Accessibility Surfaces
Accessibility is a designed, first-class experience, not an afterthought.
- **Screen-Reader Narration:** Server/client generates slow-cadence, naturalist prose describing the scene (e.g., "a small grey bird is perched on the front rail..."). Matches notebook voice.
- **Call Captions:** Procedurally generated text descriptions of calls (e.g., "a soft three-note rise") placed near the calling bird.
- **Reduced-Motion Mode:** As described in rendering, an alternative cross-fade aesthetic.
- **Keyboard Navigation:** Full tab-indexing of the top bar, arrow-key focus for birds (Enter to listen-in), high-contrast focus rings.
- **Contrast:** All user-copy text strict WCAG AA compliance.

## Performance Budgets and Observability
- **Bundle Size:** Initial JS payload < 2MB (gzipped). Requires heavy use of procedural assets, SVGs, and lazy-loading for non-critical UI (settings, notebooks).
- **Time-to-First-Bird (TTFB):** < 500ms on mid-tier 4G mobile.
- **Runtime:** 60fps idle motion on a 5-year-old laptop. No memory leaks over a 30-minute session.
- **Observability:** Synthetic checks and aggregate Real User Monitoring (RUM) for TTFB, frame timings, and simulation latency (p99 < 5s alarm).
- **Privacy Boundary:** Telemetry explicitly excludes per-bird state or per-account interaction history.

## Rollout
- **Launch Strategy:** Ship the web app with the core 2-bird experience, magic-link auth, and the foundational server tick.
- **Age-Gated Scaling:** Additional birds (up to 7) unlock slowly based on the chronological age of the aviary, not usage metrics.
- **Instrumentation:** Day 1 focus on TTFB, rendering frame drops, WebAudio initialization failure rates, and server tick execution latency.

## Risks
- **Drift Calibration:** If presence is too easily triggered or the low-pass filter is too loose, personality will drift too fast, making it feel like a game. If too slow, it feels broken. Must be heavily tested in staging with simulated daily interaction logs.
- **Sync/Event Race Conditions:** If client event logs batch poorly during poor network conditions, the server tick might process bursts of interactions inappropriately.
- **Audio Uncanniness:** Procedural synthesis failing to sound natural or multiple birds creating dissonant chords. Requires intensive auditory review against the motif library.
- **Accessibility Regressions:** Developers adding UI elements without corresponding naturalist narration prose, breaking the affective experience for screen-reader users. Strict review gates required.
