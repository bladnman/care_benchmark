# Pocket Aviary Phase 1 Implementation Plan

## Scope

**In scope for v1:**
- Single-user accounts with magic-link email authentication
- Single canonical aviary per account with 2 starter birds, cap at 7 total
- Multi-device sync with server-side canonical state
- Bird engine with personality vectors (boldness, social warmth, vocal frequency, plumage saturation, curiosity) and drift mechanics
- Mood system (wary, content, curious, drowsy, alert) with daily-ish reset and session persistence
- Procedural call synthesis with recognizable signatures per bird
- Audio pipeline with listen-in capability, chorus mixing, and WebAudio fallback
- Field notebook with naturalist observations (auto-generated, read-only)
- Visit invitations (host can invite friends read-only, opt-in per invite)
- Screen-reader narration with slow cadence prose
- Reduced-motion mode with cross-faded transitions
- Call captions with runtime descriptions
- Keyboard navigation and focus management
- Single horizontal scene with three perch zones (front/middle/back)
- Day/night cycle based on user's local timezone
- Ambient weather (rain, wind) with mood effects
- Ambient micro-motion (leaf/fellow/drift)

**Explicit out-of-scope (from non_goals.md):**
- Native mobile apps
- Gamification (no achievements, streaks, levels, scores, badges)
- Tamagotchi-style mechanics (no death, hunger, happiness meters)
- Social network surfaces (no profiles, follows, public feeds, discovery)
- Notification surfaces (no push/email about aviary activity)
- Streak counters or visit-frequency tracking
- UI chrome inside aviary view (only top bar available)
- Customizable scenes or geography features
- Multi-aviary accounts
- Payments or monetization features

**In scope but deliberately limited:**
- Web-only deployment only
- 2 birds at start, gradual species availability at aviary age thresholds
- Procedural bird naming with user renaming capability
- Identity continuity with stable bird IDs across account changes
- Accessibility surfaces designed for charm, not checklist parity
- Performance: <2MB JS bundle, first bird visible <500ms, 60fps on 5-year-old laptop, no memory growth over 30 minutes

## Architecture

**Service shape:** Monolithic simulation service with separate API layer
- Single server responsible for all simulation logic
- PostgreSQL + event store for canonical state
- Redis for session tokens and temporary state
- CDN for static assets and initial state snapshots

**Client/server split:**
- **Server (simulation service):** Owns all mutable state, runs simulation tick, computes drift, generates bird calls, writes interaction events, provides all state snapshots
- **Client (browser):** Renders snapshots, interpolates between states, synthesizes procedural audio, handles user input, manages WebAudio context, observes presence events

**Render pipeline boundary:** Canvas 2D/WebGL on client, server-side simulation separate from rendering

**Data flows:**
1. Client → Server: Interaction events (offer, listen-in, settle, presence pings)
2. Server → Client: State snapshots (bird positions, moods, call timing, active animations)
3. Server → Server (tick): Personality updates, mood transitions, drift calculations

## Data Model

**Bird entity:**
- Internal UUID (stable across account lifecycle)
- Personality vector: 5 scalar traits (boldness, social warmth, vocal frequency, plumage saturation, curiosity) - server-side, never exposed to user
- Mood enum: wary/content/curious/drowsy/alert - persists across sessions via server tick
- Name: user-assigned, mutable
- Species: from v1 pool of 6, assigned at adoption
- Perch position: derived from mood + personality
- Call signature: procedural grammar with personality-shaped timing and pitch

**Aviary entity:**
- Single per account
- Collection of bird objects
- Visit relationship (one-to-many read-only visitors)
- Notebook entries (read-only log)
- Account linkage (one-to-one)

**Account entity:**
- Email (encrypted storage)
- Synthetic UUID (never email-derived)
- Session tokens per device
- Visit invitations (email-based)

**Event types:**
- InteractionEvent (offer, listen-in start/end, settle)
- PresenceEvent (when user sits and watches)
- TickEvent (server simulation step)

**Field notebook entries:** Naturalist prose strings, lowercase, present-tense, sparse (one entry every few days)

## API Surface

**Client pull endpoints:**
- GET /api/v1/state: Returns current state snapshot (birds, moods, call timing, animations)
- GET /api/v1/notebook: Returns user's field notebook entries (paginated)
- GET /api/v1/invites: Returns user's pending visit invitations

**Client submit endpoints:**
- POST /api/v1/events: Submits interaction events (offer, listen-in, settle, presence pings)
- POST /api/v1/auth/magic-link: Requests magic link email
- POST /api/v1/auth/verify: Verifies magic link and returns session token

**Visit invitation flow:**
1. Host goes to settings → invites → enter visitor email
2. System emails visitor one-time link
3. Visitor clicks link, sees read-only view
4. Invite expires after 30 days if unused, revocable anytime by host

**Session flow:**
1. User signs in via magic link
2. Client requests initial state snapshot
3. Client connects WebAudio context, starts listening to procedural calls
4. User interacts via offer, listen-in, settle, or passive presence
5. Client sends interaction events to server
6. Server simulation tick updates personality vectors, moods
7. Client pulls updated snapshots and interpolates state changes
8. Session ends on tab close or user settle gesture

## Simulation Engine Design

**Server-side tick (~1 minute cadence):**
- Consumes event log in order
- Applies drift from recent presence-time and interactions
- Transitions moods based on time of day, personality vector, recent interactions
- Advances mood timers
- Writes new canonical state (bird positions, moods, call timers)
- Runs regardless of client connection

**Drift function:**
- Low-pass filter over presence-time + interaction signals
- Monotonic toward expressive (traits only increase, never decrease)
- Dominant input: presence-time (3-condition conjunction: visible, focus, pointer/key activity)
- Secondary inputs: listen-in (strong signal for targeted bird), offers (small curiosity boost), settle (mood-quieting signal)
- Calibration: measurable drift after ~1 week, visible to user after ~3 weeks
- Implementation: additive server-authored deltas, no client-submitted absolute values

**Mood transitions:**
- Set by: recent interactions (offer → content), time of day (drowsy near dusk), ambient events (rain dampens frequency), personality vector (high boldness reduces wary risk)
- Persists across sessions (ends in mood carries to next start, modulated by tick)
- Mood shapes: idle motion, call patterns, perch position

**Call-grammar runtime:**
- Procedural synthesis client-side using WebAudio
- Each bird has signature motif library shaped by vocal-frequency trait
- Calls recognizable across drift and mood changes
- Chorus mechanic: multiple birds calling create real-time mix, not stacked loops
- High vocal frequency → calls more often, joins chorus readily
- Calls audible from front-to-back perch zones

**Event ordering integrity:**
- Append-only event log on server
- No last-write-wins for personality state
- Simulation tick consumes in order, prevents race conditions

## Sync Model

**Canonical state sovereignty:** Server is only source of truth for personality vectors, moods, and drift. Clients own only interaction events and rendering state.

**Multi-device propagation:**
- Any device signs in with valid session token
- Server returns current snapshot (calendar state, drift history, mood persistence)
- Both devices render identical state because they read from canonical source
- Users don't sync state between devices - they share the same account/aviary

**Conflict prevention:**
- No client-submitted personality deltas
- Additive drift from event-log order
- Server-side tick only writer of personality vectors
- Event replay safety via replay detection on client
- Synthetic UUID separation prevents email-based conflicts

**Offline handling:**
- Simulation tick runs server-side regardless of connection
- Client resumes with latest snapshot on reconnect
- Pending events batched and sent on reconnection

## Frontend Rendering Pipeline

**Scene composition:**
- Canvas-based rendering with layered scene (background/sky, middle/birds, occasional foreground)
- Three fixed perch zones (front/middle/back) with parallax
- Bird sprites drawn per species silhouette with plumage saturation effects
- Ambient leaf/fellow drift (client-side, no simulation state)

**Idle micro-motion:**
- Birds never still, continuous animations
- Mood-shapes: wary birds scan back and forth, content birds preen, curious birds tilt toward sounds, drowsy birds fluff, alert birds scan edge of scene
- Motion independent of user attention (continues when tab hidden)

**Transitions:**
- Settle: soft lighting shift to evening over few seconds
- Day/night: gradual palette shifts based on local time
- Cross-fades for reduced-motion mode (instead of frame-by-frame animation)

**Reduced-motion mode:**
- Cross-fades between still poses instead of frame-by-frame
- Flight transitions become cross-fades between perches
- Ambient leaf drift removed, color shifts slowed
- Birds still drift, calls still play, notebook still updates

**Performance optimization:**
- Render only visible birds, cull offscreen
- WebAudio reused buffers, no per-call allocation that isn't freed
- Notebook entries scrolled into view don't retain references after scroll-out
- 60fps idle motion on five-year-old mid-range laptop

## Audio Pipeline

**Procedural synthesis:**
- WebAudio nodes for each bird's call synthesis
- Motif libraries per species, combined and varied at runtime
- Personality-shaped timing: high vocal frequency → more frequent calls
- Mood-shapes: wary birds quieter/higher-pitched, content birds softer, curious birds more frequent

**Listen-in mix:**
- Focus bird call rises gradually in mix
- Other birds quiet to ambient (but never silent)
- Smooth ramp up/down for gradual transition
- Disengages on bird ref focus, click empty space, or keyboard focus away

**Call-grammar recognizability:**
- Each bird maintains unique signature across drift
- 7-bird cap because above this chorus blurs into ambient
- Procedural variation prevents canned feel (no identical calls)
- Chorus created by mixing multiple real-time calls

**WebAudio fallback:**
- If WebAudio unavailable, graceful silence with captions on by default
- No recorded-audio fallback path
- Render-only experience continues with visual-only presentation

## Accessibility Surfaces

**Screen-reader narration:**
- Server/client generates naturalist prose updates (~every 30-60s at idle)
- Voice matches field notebook (lowercase, present-tense, specific)
- User-initiated events get priority bump in queue
- Examples: "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."

**Call captions:**
- Runtime descriptions of what each call sounds like
- Generated from procedural grammar: "a soft three-note rise", "a low trill, paused, low trill again"
- Fade in/out with call timing
- Same naturalist voice as rest of product

**Reduced-motion mode:**
- Cross-faded transitions instead of frame-by-frame
- Simpler visual register but same affective core
- Alternative rendering, not fallback

**Keyboard navigation:**
- Tab moves through top bar, then into aviary
- Arrow keys move focus between birds
- Enter triggers listen-in on focused bird
- Escape exits listen-in
- Offer affordance reachable via top-bar shortcut
- Focus indicators visible against aviary background

**Contrast:**
- User-copy text passes WCAG AA
- Aviary scene contains no user copy except top bar
- Design system specifies exact contrast ratios

## Performance Budgets and Observability

**Bundle size:** <2MB initial JS bundle (gzipped)
- Code-splitting for less-frequent surfaces (account settings, accessibility, invites)
- Procedural audio client-side saves recorded audio from bundle
- Small bird visual assets (SVG/bitmaps with tight optimization)
- Bundle measured via CI automated browser tests

**Time-to-first-bird:** <500ms on mid-tier mobile over 4G
- Fast initial state-snapshot delivery from CDN edge
- Render path skips non-critical assets for first bird
- Measured via automated browser tests with common device profiles

**Runtime performance:**
- 60fps idle motion on 5-year-old mid-range laptop
- No memory growth over 30 minutes
- Memory measured via browser DevTools heap snapshots

**Observability:**
- Aggregate-only telemetry (no per-bird state)
- Request counts, latencies, error rates
- Session-duration histograms (anonymized)
- Render-frame timing, audio-context errors
- Simulation-tick p99 latency alarm at 5 seconds
- Synthetic performance checks from automated browsers in common geographies

**Performance boundaries:**
- Per-bird telemetry: deliberately excluded from all observability
- Interaction history: never part of aggregate telemetry
- Privacy boundary separates operational health from user relationship data

## Rollout

**v1 shipping approach:**
- All accessibility surfaces ship with product (reduced-motion, captions, narration)
- Web-only launch as specified in non-goals
- No feature flags - all in-scope features enabled by default

**Birds-per-aviary ramp:**
- Starts with 2 birds per new account
- Third bird available when aviary reaches ~3 months age
- Fourth/fifth/sixth/seventh birds gradually unlock as aviary ages
- Pacing matches relationship deepening, not attention metrics

**Instrumentation from day one:**
- Bundle size, first-bird timing, render FPS
- Memory growth detection
- Audio-context errors, simulation-tick latency
- Retention measured by revisit rate (no streaks)
- Drift calibration monitored via automated instruments (as specified in PRD)

**Monitoring priorities:**
- Drift calibration alerts (deviation from design targets)
- Sync correctness detection (conflicts, desync)
- Audio uncanniness detection (recognizability tests)
- Accessibility regression detection (screen reader usability)
- Performance alerts (bundle size, render bottlenecks)

## Risks

**Drift calibration:**
- Risk: Drift function too fast turns product into Tamagotchi
- Risk: Drift function too slow turns into screensaver
- Mitigation: Automated instrument monitoring, user-visible drift targets after 3 weeks
- Red flag: Birds changing visibly between sessions

**Sync correctness:**
- Risk: Multi-device desync from client-side simulation attempts
- Risk: Event ordering corruption from network replay
- Mitigation: Server-only personality writing, additive deltas, synthetic UUID
- Red flag: Different birds on laptop vs phone showing different personalities

**Audio uncanniness:**
- Risk: Procedural calls feel canned if not varying enough
- Risk: Chorus blurs into ambient too early (more than 7 birds)
- Risk: Recognizability drops over drift
- Mitigation: Continuous audio quality testing, real user test for bird recognition, band 7-bird cap
- Red flag: Users cannot identify their bird by ear after weeks of use

**Accessibility regressions:**
- Risk: Screen reader narration feels like system announcement rather than observation
- Risk: Reduced-motion mode feels like stripped fallback rather than alternative
- Risk: Captions out of sync with calls
- Mitigation: Designed-in accessibility from day one, not retrofitted, continuous user testing
- Red flag: Screen reader user feels the product is dumbed down for them

**Performance failures:**
- Risk: Bundle exceeds 2MB, causing load notices
- Risk: First bird renders >500ms, breaking product opening promise
- Risk: Memory leaks after 30 minutes
- Mitigation: Automated browser testing on target devices, memory profiling, CI enforcement
- Red flag: User notices load state on first page open

**Relationship safety:**
- Risk: Gamification creep after successful engagement
- Risk: Visitor notifications push user back to counter instead of birds
- Mitigation: Hard rule against streak counters, explicit refusal of social features beyond visits
- Red flag: User asks "why aren't there achievements for my birds?"