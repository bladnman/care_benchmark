# Pocket Aviary - Implementation Plan

## Scope
**In Scope for V1:**
- Browser-based virtual aviary (single horizontal scene, responsive).
- Single-user accounts with magic-link email sign-in.
- 2 starter birds, capping at 7 birds based on aviary age.
- Client-side procedural audio synthesis via WebAudio for bird calls.
- Server-side simulation tick (~1/min) advancing aviary state independently of clients.
- Multi-device sync (clients read canonical server state).
- Field notebook (auto-generated naturalist observations).
- User interactions: Return-greeting, listen-in, offer (seed, song fragment, pool), settle.
- Presence accounting requiring visibility, window focus, and pointer/key activity.
- Accessibility: Screen-reader narration, reduced-motion mode (cross-fades), call captioning, WCAG AA contrast, keyboard navigation.
- Social: Opt-in, read-only ambient visits via email invite (no co-presence).

**Out of Scope (Strict Non-Goals):**
- Native mobile apps (iOS/Android).
- Gamification (no streaks, levels, scores, badges, achievements).
- Tamagotchi mechanics (no death, hunger, distress; neglect causes quietness, not suffering).
- Social networks (no profiles, public discovery, leaderboards, chat, mutual visits).
- Exposing exact personality vector numerical values to users.

## Architecture
**Client/Server Split:**
- **Client (Frontend):** Renders the scene, handles procedural audio synthesis, interpolates animations between state snapshots, captures interactions, and writes to an append-only event log on the server.
- **Server (Backend):** Holds canonical state. A chron-based simulation tick (~1/min) consumes the event log, computes drift/mood updates, and advances the canonical state. 
- **Render Pipeline Boundary:** The client never writes personality state directly (no last-write-wins). The server authors all deltas.

## Data model
- **Accounts:** `synthetic_uuid` (Primary Key), encrypted email (never used as an identifier elsewhere), session tokens, settings (audio, motion, notifications), visit log.
- **Birds:** Stable internal `bird_id`, `account_uuid`, name, species.
- **Personality Vectors (Server-only):** Boldness, social warmth, vocal frequency, plumage saturation, curiosity. (Drift is monotonic; traits never decay on neglect).
- **Mood (Fast-timescale):** Current state (wary, content, curious, drowsy, alert). Persists across sessions.
- **Event Log:** Append-only log of user interactions (`presence_ping`, `listen_in_start`, `listen_in_end`, `offer_made`, `settle_triggered`).
- **Notebook Entries:** Auto-generated text in naturalist voice, timestamp.
- **Visits:** `invite_id`, `host_account_uuid`, `visitor_email`, `status` (active/revoked), expiration timestamp.

## API surface
- `POST /auth/magic-link`: Request sign-in link.
- `POST /auth/verify`: Consume link, get session token.
- `GET /aviary/snapshot`: Pull current canonical state (bird positions, moods, call timing).
- `POST /aviary/events`: Submit batch of interaction events (presence, listen-in, offers, settle) to the append-only log.
- `GET /notebook`: Fetch paginated field notebook entries.
- `POST /social/invite`: Issue visit invite.
- `POST /social/revoke`: Revoke visit invite.
- `GET /visit/:invite_id`: (Visitor) Pull read-only aviary snapshot.

## Simulation engine design
- **Server-Side Tick:** Runs roughly every minute per active aviary (or batched/lazy-evaluated on next read if dormant).
- **Drift Function:** Low-pass filter over presence and interaction signals. Monotonic towards expressive. Three weeks of regular presence produces visible changes.
- **Mood Transitions:** Evaluated during the tick. Influenced by recent events, local time of day, ambient weather, and base personality vector.
- **Call-Grammar Runtime:** Server dictates the motifs/timing parameters based on vocal frequency; client executes the procedural grammar via WebAudio.

## Sync model
- **Canonical Server State:** The server is the absolute source of truth.
- **Conflict Prevention:** Clients do not hold or push absolute personality state. They only submit events. The server applies events sequentially.
- **Multi-device:** A user opening a laptop and a phone simultaneously will see the exact same snapshot. No client-to-client sync or merge conflicts exist.

## Frontend rendering pipeline
- **Initial Load:** No loading spinners. The first frame shows birds mid-action. If delayed, show a quiet field (sky/background).
- **Scene Composition:** 3 perch zones (front, middle, back). Subtle parallax for background/foreground.
- **Micro-motion:** Idle animations (preening, scanning) and ambient weather (leaves, feathers) run independently of the tick.
- **Reduced-Motion Mode:** Swaps frame-by-frame animations for slow cross-fades. Removes ambient leaf drift. Maintains color shifts and procedural audio.

## Audio pipeline
- **Synthesis:** Client-side WebAudio API synthesizes motifs procedurally to avoid looping artifacts and phase-canceling in chorus.
- **Mixer:** Supports dynamic mixing. The "listen-in" interaction slowly raises the focused bird's volume while lowering (but not silencing) others.
- **Fallback:** If WebAudio is blocked/unavailable, fail gracefully to silence with captions on by default. No recorded audio fallbacks.

## Accessibility surfaces
- **Screen-Reader Narration:** Slow-cadence, naturalist prose updates (e.g., "a warbler perches on the high branch..."). Not a generic ARIA state-change log.
- **Captions:** Procedurally generated text descriptions of calls in the naturalist voice.
- **Keyboard Navigation:** Full tab support for top-bar chrome and aviary scene. Focus outlines designed with high WCAG AA contrast.
- **Contrast:** All user-copy text meets or exceeds WCAG AA.

## Performance budgets and observability
- **Initial JS Bundle:** < 2MB (gzipped).
- **Time-to-first-bird:** < 500ms on mid-tier mobile over 4G.
- **Runtime:** 60fps idle motion on a 5-year-old mid-range laptop. No memory growth over a 30-minute session.
- **Observability:** Synthetic checks, aggregate RUM (page load, render frames, tick latency).
- **Privacy Boundary:** No per-bird state or interaction history is ever sent to aggregate telemetry pipelines. Tick latency p99 alarm set at 5s.

## Rollout
- **V1 Launch:** Web-only, magic-link auth, 2 starter birds per user.
- **Pacing:** New bird species unlocks tied exclusively to aviary age (e.g., third bird unlocked after a month).
- **Instrumentation:** Monitor tick latency, audio-context initialization errors, and presence-ping delivery rates.

## Risks
- **Drift Calibration:** If presence is calculated poorly (e.g., counting minimized tabs), drift happens too fast, breaking the "alive over weeks" illusion. Mitigation: Strict conjunction of visibility, focus, and pointer/key activity.
- **Audio Uncanniness:** Procedural generation might sound robotic if motif variation is too low. Mitigation: Sufficiently diverse motif library and pitch/timing modulation tied to personality.
- **Accessibility Regressions:** Falling back to "cheap" ARIA labels destroys the product's charm for screen-reader users. Mitigation: Dedicated server/client logic for generating naturalist prose narration.
- **Sync Correctness:** Dropped event-log writes could lose user presence time. Mitigation: Reliable retry mechanisms for the append-only event log and robust handling of dormant aviary ticks.