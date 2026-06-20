# Phase 1 Implementation Plan - Pocket Aviary v1

## Scope

V1 implements a subset of the Pocket Aviary product with careful attention to the "feels alive, not robotic" principle and all explicit non-goals:

**Included in v1:**
- Two birds per aviary (caps at seven as per design constraint)
- Single-user accounts with magic-link sign-in
- Multi-device sync (same canonical aviary across devices)
- Server-side simulation tick (~1 minute cadence)
- Personality vector system with 5 traits (boldness, social warmth, vocal frequency, plumage saturation, curiosity)
- Mood system (wary, content, curious, drowsy, alert)
- Procedural call synthesis with chorus mixing
- Listeners-in: focusing birds to elevate calls in audio mix
- Offer interactions (seed, song fragment, water pool)
- Return-greeting: one bird notices user on session return
- Settle gesture: soft session-end lighting transition
- Field notebook: auto-generated naturalist observations
- Visit invitations: quiet opt-in read-only viewing of another user's aviary
- Screen-reader narration (naturalist prose, slow cadence)
- Reduced-motion mode (cross-fade rendering, not animations off)
- Call captions (prose descriptions of calls)
- Presence accounting (precise 3-signal definition for honest drift)
- Keyboard navigation (full accessibility compliance)
- WebAudio with graceful fallback (silence with captions if unavailable)
- Prior-day bird species pool (~6 species)
- User name assignment per bird

**Excluded from v1 (respecting non-goals):**
- Native mobile apps
- Gamification (no achievements, streaks, levels, scores, badges)
- Tamagotchi-style mechanics (no bird death, hunger, distress meters)
- Social network surfaces (no profiles, follows, public discovery)
- Shared/multi-user aviaries
- Customizable scenes
- Multi-aviary accounts
- Payments, leaderboards, notifications
- Push notifications or user-engagement alerts

## Architecture

**Service Shape:**
- Monolithic simulation service (single-writer architecture)
- Separate authentication service (magic-link workflow)
- Content delivery network for static assets (bundle, bird assets, calls)
- Database with account and per-bird state (separated from telemetry)
- Real-time WebSocket for event streaming (presence, interactions)
- CDN for state snapshots with edge caching

**Client/Server Split:**
- Server owns all simulation state (personality vectors, mood, drift history)
- Server runs slow tick (~60 seconds) that updates canonical aviary state
- Clients pull small state snapshots (kilobytes) and render interpolated motion
- Clients send append-only interaction events to server
- Client handles procedural call synthesis (WebAudio) for variation
- Client manages field notebook UI for human-consumable prose

**Render Pipeline Boundary:**
- Raw simulation state → client snapshot → rendering engine
- Procedural call synthesis stays client-side (CPU-bound, per-user variation)
- Visual bird assets generated/client-rendered (no external dependencies)
- Accessibility surfaces (narration, captions) co-rendered with visual aviary

## Data Model

**Account:**
- Synthetic UUID (never email)
- Email address (encrypted, single write)
- Session tokens (revocable per-device)
- Visit log (read-only history)
- Settings (notification toggles, reduced-motion preference)
- Soft deletion (30 days) → hard deletion

**Bird:**
- Stable internal ID (immutable across all changes)
- Species classification (from 6-v1 pool)
- User-assigned name (renameable at any time)
- Personality vector: 5 float traits (stored server-side, never exposed)
- Current mood (fast-timescale state)
- Per-bird event log (offers, listen-ins, presence timestamps)

**State:**
- Per-bird positions (perch coordinates, motion state)
- Call timing parameters (pitch, rhythm based on vocal frequency)
- Weather effects (rain dampening, wind alertness)
- Notebook entries (naturalist prose, timestamped)

**Event Log:**
- Append-only, log-structured
- Interactions (offer, listen-in start/end, settle)
- Presence pings (only from host devices when all 3 signals true)
- Tick events (server-side updates)

## API Surface

**State Access:**
- `GET /api/aviary/state` → current state snapshot
- JSON format: birds with positions, moods, call parameters, notebook entries
- Small payload (<50KB) for rapid loading

**Event Submission:**
- `POST /api/events` → append interaction event
- JSON format: event type, bird ID (if applicable), metadata
- Single-writer guarantees (no conflict resolution needed)

**Visit Invites:**
- `POST /api/visits` → create invitation (host emails visitor)
- `GET /api/visits/{token}` → validate, return read-only state
- `DELETE /api/visits/{id}` → revoke invitation

**User Operations:**
- `POST /auth/magic-link` → send magic link
- `POST /auth/magic-link/:token` → sign in, return session token
- `GET /account/export` → JSON download of user's aviary state

**Notification Surface:**
- Matter-of-fact error messages only (never announces presence)

## Simulation Engine Design

**Server-Side Tick:**
- Runs ~once/minute, advances canonical state
- Reads recent event log in order
- Updates personality vectors (drift additive, no negative drift)
- Transitions moods (time-of-day, ambient events, recent interactions)
- Advances call timing parameters (vocal frequency trait)
- Writes new state (immutable append to state history)

**Drift Function:**
- Presence-time as dominant input (precise 3-signal definition)
- Listen-in: focused bird gets social warmth/vocal frequency boost
- Offers: acceptance drives curiosity (offering near bird drives boldness)
- Monotonic toward expressive: traits never decrease on neglect
- Calibration: measurable drift after 1 week, visible after ~3 weeks

**Mood Transitions:**
- Base reset: time-of-day (drowsy near dusk, alert mornings)
- Interaction effects: offer acceptance nudges toward content
- Bird-to-bird effects: nearby bird calls shift others toward wary
- Personality bias: high-boldness birds resist wary transitions
- Persistence: mood at session-end becomes next session-start mood

**Call Grammar Runtime:**
- Procedural synthesis: motifs from species library
- Timing shaped by vocal frequency trait
- Pitch modulated by mood and personality
- Real-time per-call variation (never identical recordings)
- Chorus mixing: overlapping calls blend naturally, not layered

## Sync Model

**Multi-Device Synchronization:**
- Single source of truth: server-side canonical state
- No client-to-client syncing required
- Users sign in on devices, both pull same snapshots
- No personality state to sync (server owns it)
- Event log ensures consistency across devices

**No Last-Write-Wins:**
- Personality updates always server-authored (additive deltas)
- Event log ordering guarantees consistent application
- No device can overwrite another's drift history

**Conflict Resolution:**
- No conflict resolution needed for personality state
- Magic-link replay handled by invitation token validation
- Server-side consistency eliminated as failure mode

**Inconsistencies:**
- Visitor sessions see host's exact state (stale snapshot possible)
- No reconciliation needed between devices for host sessions

## Frontend Rendering Pipeline

**Scene Composition:**
- Three-perch zone system (front/middle/back)
- Birds choose perch based on mood/personality
- Local-time day/night cycle affects lighting, calls, moods
- Ambient weather effects (rain dampening calls, wind alertness)
- Subtle parallax foreground/background separation

**Idle Motion:**
- Continuous micro-motion (preening, scanning, head-tilting)
- Mood-shaped: wary birds scan, content preen, curious tilt toward sounds
- No pausing when tab not focused (simulation continues)

**Transitions:**
- Load state: birds appear mid-action (no entry animation)
- Listen-in: gradual mix elevation over 1-2 seconds
- Offer reaction: smooth bird approach/animation
- Settle: slow lighting shift to evening over several seconds

**Reduced-Motion Mode:**
- Cross-fade between pre-rendered poses instead of frame-by-frame animation
- Flight transitions become perch-to-perch cross-fades
- Ambient leaf drift removed, color shifts slowed
- Same aviary, different rendering intensity

**Component Architecture:**
- Rendering engine → WebGL/2D canvas
- Audio engine → WebAudio API (procedural synthesis)
- UI layer → React/Vue (top bar, notebook, settings)
- State management → Redux/Redux-Saga with WebSocket updates

## Audio Pipeline

**Procedural Call Synthesis:**
- Client-side WebAudio nodes
- Motif library per species (from bundle)
- Real-time pitch/timing variation based on personality traits
- Mood-based timbre changes
- Chorus mixing: overlapping calls blend naturally

**Listen-In Mix:**
- Focused bird elevated gradually (1-2 seconds)
- Other birds drop to ambient (not silenced)
- Smooth return to ambient on disengage

**Mix Decay:**
- Quiet power-down when tab not visible
- Gradual return when tab becomes visible again
- No hard cut-downs that break immersion

**WebAudio Fallback:**
- If WebAudio unavailable: captions on by default
- Proceed with visual aviary only
- No recorded audio fallback (preserves variation property)

**Performance Requirements:**
- Initial load: <2MB JS bundle (gzipped)
- Time-to-first-bird: <500ms on mid-tier mobile over 4G
- 60fps idle motion on 5-year-old laptop
- No memory growth over 30-minute sessions

## Accessibility Surfaces

**Screen-Reader Narration:**
- Running prose updated every 30-60 seconds
- Naturalist voice (lowercase, present-tense, specific)
- Server-side or client-side generation from same state
- User-initiated events get priority bump but still prose

**Reduced-Motion Mode:**
- Cross-fade rendering (not animations off)
- Same aviary, different visual intensity
- User-initiated opt-in or `prefers-reduced-motion`

**Call Captions:**
- Real-time prose descriptions (naturalist voice)
- Appear with calls, fade in/out with audio
- Click-to-toggle in settings

**Keyboard Navigation:**
- Full keyboard access (tab through all interactive elements)
- Arrow key movement between birds in aviary
- Escape to exit listen-in
- Top bar shortcut to offer

**Contrast Compliance:**
- All user-copy text passes WCAG AA
- Focus indicators visible against any aviary state
- Design system specifies exact contrast ratios

## Performance Budgets and Observability

**Bundle Size Budget:**
- Target: 1.8MB gzipped (under 2MB ceiling)
- Procedural audio avoids recorded audio files
- Birds assets generated or small SVGs/bitmaps
- Aggressive code splitting for rare surfaces

**Time-to-First-Bird Budget:**
- Target: 350ms on mid-tier mobile over 4G
- Fast initial state-snapshot delivery from CDN edge
- Rendering path optimized to draw first bird before non-critical assets
- Bundle size drives all asset decisions downstream

**Runtime Budgets:**
- 60fps idle motion on 5-year-old laptop (no throttling)
- No memory growth over 30-minute sessions
- WebAudio context restarts within 2 seconds of context loss
- Simulation-tick p99 <5 seconds (alarm on breach)

**Observability:**
- Aggregate-only telemetry (no per-account state)
- Synthetic performance checks from distributed browsers
- Real User Monitoring (anonymized session durations, error rates)
- Client-side render-frame timings, audio-context errors
- Bundle size measurement per deployment

**Privacy Boundary:**
- Per-account interaction events never leave user's own simulation
- Telemetry pipelines never access simulation database
- Analytics warehouse separate from production systems
- Account export includes full aviary state (JSON)

## Rollout

**v1 Launch:**
- Limited beta (1000 users) with instrumentation
- Performance monitoring second-by-second
- Error rates tracked per feature surface
- User feedback collection on affordability and charm

**Bird Cap Ramp:**
- Start: 2 birds per aviary (inverse rule from day one)
- 3-4 months: offer third bird based on aviary age
- Maintain sense of intimacy: 7bird cap never exceeded

**Instrumentation:**
- Session duration, presence-time, audio-play counts
- Bird adoption patterns, species preferences
- Session abandonment vs. settlement rates
- Accessibility feature uptake (narration, captions, reduced-motion)

**User Onboarding:**
- Magic-link sign-in (email-only)
- Immediate aviary experience (no setup required)
- Two starter birds presented as arrivals, not choices

**Phase Gate:**
- Performance budgets met (bundle size, first bird, fps)
- Accessibility surfaces fully functional (not checklist)
- All non-goals strictly enforced (zero gamificati