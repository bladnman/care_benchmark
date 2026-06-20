# Comprehensive Implementation Plan for Pocket Aviary v1.0

## Scope

**What's in v1:**
- Single-user accounts with email magic-link authentication
- Two starter birds per aviary (caps at seven birds total)
- Core bird engine with personality vectors, mood system, and procedural calls
- Multi-device sync through server-side simulation tick
- User interactions: return-greeting, listen-in, offer, settle, field notebook
- Accessibility surfaces: screen-reader narration, reduced-motion mode, call captioning
- Visit-invite feature (host can invite friends for read-only viewing)
- Account export functionality

**What's NOT in v1 (respected from non-goals):**
- Native mobile apps
- Game mechanics (no achievements, streaks, scores)
- Tamagotchi-style mechanics (birds don't die or get hungry)
- Social network surfaces (no profiles, follows, public feeds)
- Gamification elements of any kind
- Notifications or push communications

## Architecture

### Service Shape
- **Frontend**: Single-page webapp with WebAudio client-side synthesis
- **Backend**: Server-side simulation tick (~1 minute cadence)
- **Database**: Relational database with synthetic UUID account IDs
- **Communication**: REST APIs for state snapshots and event submission
- **Infrastructure**: CDN for static assets, service workers for offline capabilities

### Client/Server Split
- **Server owns**: Personality state, event log, canonical aviary state
- **Client owns**: Rendering pipeline, audio synthesis, user input handling
- **Sync model**: Server pushes state snapshots to clients on change

### Render Pipeline Boundary
- Critical rendering path optimized for <500ms first bird visibility
- Procedural bird animation generation client-side
- WebAudio API for real-time call synthesis (no recorded audio)
- Reduced-motion mode as a first-class designed surface

## Data Model

### Core Entities
- **Account**: Synthetic UUID, encrypted email, single aviary reference
- **Bird**: Stable ID, name, species reference, personality vector (5 traits), current mood
- **Personality Vector**: 5 traits (boldness, social warmth, vocal frequency, plumage saturation, curiosity)
- **Mood**: Enumerated state (wary, content, curious, drowsy, alert)
- **Field Notebook**: Auto-generated naturalist observations (read-only)
- **Event Log**: Append-only log of user interactions
- **Visit**: Host-to-visitor invitation relationships

### Relationships
- One Account → One Aviary
- One Aviary → Multiple Birds (2-7)
- Birds interact with each other through ambient calls
- Birds interact with user through personality drift and mood transitions

## API Surface

### State Operations
- `GET /api/aviary/{accountId}`: Pull current state snapshot
- `POST /api/events`: Submit user interaction events (offer, listen-in, settle)
- `GET /api/account/{accountId}/export`: Generate aviary state export
- `DELETE /api/account/{accountId}`: Initiate account deletion

### Visit Operations
- `POST /api/visits`: Create new visit invitation
- `GET /api/visits/{visitId}`: Check visit status (for visitor)
- `DELETE /api/visits/{visitId}`: Revoke invitation

### Account Operations
- `POST /api/auth/magic-link`: Request magic link
- `POST /api/auth/verify`: Verify magic link and establish session
- `GET /api/account/{accountId}/sessions`: List active sessions
- `DELETE /api/account/{accountId}/sessions/{sessionId}`: Revoke session

### Access Control
- All state operations require valid session token
- Account ownership enforced server-side
- Visit invitations use one-time links with email verification

## Simulation Engine Design

### Server-Side Tick
- Runs approximately once per minute regardless of client presence
- Processes event log in chronological order
- Updates personality vectors based on accumulated inputs
- Transitions moods based on time, interactions, and ambient factors
- Writes canonical aviary state to database

### Drift Function
- **Inputs**: Presence-time (weighted highest), listen-in, offers, settle gestures
- **Direction**: Monotonic toward expressive (traits only move up, never down)
- **Calibration**: Visible drift after ~3 weeks, measurable instruments after ~1 week
- **Mechanism**: Low-pass filter over accumulated event inputs

### Call Grammar Runtime
- Procedural motif combination based on bird species library
- Personality-shaped timing (vocal frequency trait)
- Mood-appropriate call variation
- Real-time chorus mixing with per-bird level control

### Mood Transitions
- **Triggers**: Recent interactions, time of day, ambient events, personality traits
- **Persistence**: Mood carries across sessions, updated by server tick
- **Visibility**: Expressed through idle motion and call characteristics

## Sync Model

### Canonical State
- Server maintains single source of truth for aviary state
- Personality vectors only written by server simulation tick
- Event log is append-only, never overwritten
- Multi-device sync achieved through shared server state

### Conflict Prevention
- **No last-write-wins**: Additive server-authored deltas only
- **Event ordering**: Events processed in chronological order from log
- **Personality ownership**: Clients never submit personality updates
- **Session handling**: Revocable session tokens prevent unauthorized writes

### Client Synchronization
- Pull-based model: Clients request state changes
- Efficient snapshot format (kilobytes, not megabytes)
- Interpolation for smooth bird motion between ticks
- Presence-aware state pull (visibility change, focus change, long idle)

## Frontend Rendering Pipeline

### Scene Composition
- Single horizontal scene optimized for one screen
- Three perch zones (front/mid/back) for proximity signaling
- Local timezone-anchored day/night cycles
- Ambient weather effects (rare, subtle rain/wind)

### Idle Micro-Motion
- Continuous bird preening, scanning, head-tilting
- Mood-shaped idle behavior (wary birds scan more, content birds preen)
- Ambient leaf and feather drift (client-side rendering ornaments)
- Subtle parallax between foreground and background

### Listen-In Mix
- Gradual audio level transition when focusing a bird
- Other birds drop to ambient but remain audible
- Smooth ramp up/down on listen-in engage/disengage
- No hard channel switching

### Reduced-Motion Mode
- Cross-fade between still poses instead of frame-by-frame animation
- Flight transitions become cross-fades between perches
- Ambient elements simplified (no leaf drift)
- Core simulation (birds, calls, drift) unchanged

### Performance Optimizations
- Initial JS bundle <2MB (gzipped)
- Time to first bird <500ms on mid-tier devices
- 60fps idle motion on 5-year-old laptops
- No memory growth over 30-minute sessions
- Procedural audio only (no recorded audio fallbacks)

## Audio Pipeline

### Procedural Call Synthesis
- WebAudio API for real-time call generation
- Species-specific motif libraries
- Personality-shaped timing and pitch modulation
- Mood-appropriate call variation

### Chorus Mixing
- Real-time per-bird level control
- Dynamic fade based on bird proximity to listener
- High vocal frequency birds join chorus more readily

### Listen-In Mix Decay
- When listen-in ends, focused bird gradually returns to chorus level
- Other birds gradually rise from ambient
- Complete smooth transition over a few seconds

### Accessibility Integration
- All call synthesis accessible through screen readers
- Text captions generated from procedural grammar
- Audio fallback: complete silence with captions if WebAudio unavailable

## Accessibility Surfaces

### Screen-Reader Narration
- Running naturalist prose updated every 30-60 seconds
- Generated from server-side state (not pre-recorded)
- User-initiated events get priority bumps
- Same voice as field notebook (continuous prose, not bullet points)

### Reduced-Motion Mode
- Cross-fade animation system
- Simplified ambient effects
- Slower color transitions for day/night
- Core simulation unchanged

### Call Captioning
- Real-time captions generated from call grammar
- Short prose descriptions synchronized with calls
- Uses naturalist voice consistent with rest of product
- Optional but enabled by default in accessibility settings

### Keyboard Navigation
- Full keyboard accessibility for all interactive surfaces
- Tab order: top bar → birds → offer affordance
- Arrow keys for bird selection, Enter for listen-in, Escape to exit
- Focus indicators with high contrast against aviary background

## Performance Budgets and Observability

### Bundle Size
- Initial JS: <2MB (gzipped)
- Critical path includes bird rendering, audio engine, state management
- Code-splitting for account settings, accessibility, visit flows
- Aggressive tree-shaking for unused features

### Rendering Performance
- First bird visible <500ms on mid-tier mobile over 4G
- 60fps idle motion requirement enforced in CI
- Render pipeline skips non-critical paths when tab is hidden
- Memory usage capped with buffer pool reuse

### Audio Performance
- WebAudio context creation on user gesture only
- Audio buffers reused across calls to prevent memory growth
- Simulation-tick timeout protection (alarm at p99 5s)
- Error rate monitoring for audio context failures

### Observability
- Aggregate-only Real User Monitoring (no per-bird data)
- Synthetic performance tests from multiple geographic locations
- Session duration histograms (anonymized)
- Error budgets for bundle size, render timing, audio stability
- No per-account interaction history in telemetry

### Calibration Targets
- Bird recognizability maintained up to 7 birds in chorus
- Drift visible after 3 weeks, measurable after 1 week
- Presence accuracy >95% on proper user engagement scenarios
- Accessibility compliance maintained throughout development

## Rollout Strategy

### v1 Launch
- Magic-link authentication only (no password options)
- Two starter birds per new account
- Visit invitations opt-in only (disabled by default)
- Limited user feedback collection (session metrics only)
- No marketing push or social promotion

### Birds-Per-Aviary Ramp
- Start with 2 birds per aviary (default)
- Third bird becomes available at 2 months aviary age
- Fourth+ birds available at 6+ months age
- Species variety increases with aviary age
- Automatic progression to maintain relationship depth

### Day One Instrumentation
- Session start/end tracking
- Presence event recording and accuracy monitoring
- Audio pipeline error rates
- Render frame timing histograms
- Bird interaction frequency (offer, listen-in usage patterns)

### Success Metrics
- User retention: % of sessions where birds show visible drift after 3 weeks
- Performance: Bundle size, time-to-first-bird, audio continuity
- Accessibility: Screen reader task completion rates
- Drift calibration: Personality vector changes in instrumented accounts

## Risks

### Drift Calibration Risk
- **Problem**: Birds drift too slowly (users notice nothing) or too fast (visible session-by-session changes)
- **Impact**: Product fails "feels alive over weeks" promise
- **Mitigation**: Target measurable after 1 week, visible after 3 weeks; A/B testing across user populations

### Sync Correctness Risk
- **Problem**: Multi-device sessions corrupt personality state
- **Impact**: User sees inconsistent birds across devices
- **Mitigation**: Server-only personality writing; additive deltas only; comprehensive conflict testing

### Audio Uncanniness Risk
- **Problem**: Procedural calls feel too robotic or unnatural
- **Impact**: User abandons product due to audio quality
- **Mitigation**: Continuous user testing; voice-cliff evaluation; iteration on motif libraries

### Accessibility Regression Risk
- **Problem**: Accessibility features compromise core product charm
- **Impact**: Reduced user base, legal compliance issues
- **Mitigation**: Accessibility included from day one; designed surfaces instead of fallbacks; regular accessibility testing

### Privacy Compliance Risk
- **Problem**: Per-bird interaction data leaks into telemetry/analytics
- **Impact**: User trust violation, regulatory penalties
- **Mitigation**: Strict telemetry boundaries; synthetic UUIDs for all internal references; regular privacy audits

### Performance Budget Risk
- **Problem**: Bundle size or render timing exceeds thresholds
- **Impact**: Poor user experience, slow adoption
- **Mitigation**: Automated performance budgets in CI; performance testing on target devices; continuous optimization
