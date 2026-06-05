# Pocket Aviary — Implementation Plan

## 1. Scope

### V1 In Scope
- Two starter birds per aviary, max seven
- Single-user accounts with email magic-link sign-in
- Multi-device sync via server-side canonical state
- Field notebook with auto-generated naturalist observations
- Presence accounting for personality drift
- Listen-in interaction (focus a single bird's audio)
- Offer interaction (seed, song fragment, still pool)
- Settle gesture (soft session end with evening lighting)
- Visit invitations (read-only, opt-in per invite)
- Screen-reader narration in naturalist prose
- Reduced-motion mode for accessibility
- Call captioning
- Day/night cycle tied to user's local time
- Ambient weather (rain, wind) with mood effects

### V1 Out of Scope (per `non_goals.md`)
- Native mobile apps (web-only)
- Gamification: no achievements, no streaks, no levels, no scores, no badges, no calendar of green dots
- Tamagotchi mechanics: birds do not die, no hunger, no distress, no happiness meter decay
- Social network surfaces: no profiles, no follows, no public feed, no comments on visits
- Notifications: no push/ping/email about aviary activity
- Shared aviaries, multi-aviary accounts, public discovery, leaderboards

### Named Exceptions to Naturalist Voice
- Account, error, sync, and accessibility-settings surfaces use matter-of-fact voice
- Visit log in settings is matter-of-fact
- Sync conflict surfaces are matter-of-fact

## 2. Architecture

### Service Shape
- **Frontend**: Single-page web application (React/Vue-like, but framework-agnostic)
- **Backend**: Stateful Node.js service with:
  - REST API for state snapshots and event submission
  - Server-side simulation tick (once per minute)
  - Event log append-only store
  - Session token management
  - Email service for magic links and account operations

### Client/Server Split
- **Client responsibilities**:
  - Render aviary scene with birds, perches, ambient motion
  - Procedural audio synthesis via WebAudio
  - Presence detection (visibilityState + focus + pointer/keypress)
  - Interaction event submission (listen-in, offer, settle, presence pings)
  - Interpolation between state snapshots for smooth motion
  - Reduced-motion rendering when requested
  - Screen-reader narration generation (or consumption from server)
  - Call captioning

- **Server responsibilities**:
  - Canonical state storage (birds, personality vectors, moods, notebook entries)
  - Simulation tick (personality drift, mood transitions, ambient events)
  - Event log processing (append-only, consumed by tick)
  - Session management (magic-link validation, token issuance/revocation)
  - Account CRUD (create, export, soft-deletion, email change)
  - Visit invitation management (issue, revoke, log)

### Render Pipeline Boundary
- Client pulls state snapshots from server; never owns canonical state
- Client interpolates between snapshots for smooth motion
- Client renders procedural audio in real-time, never downloads pre-recorded audio
- Client handles reduced-motion rendering as a designed surface, not a fallback

## 3. Data Model

### Account
- Synthetic UUID (primary identifier)
- Email (encrypted, stored once)
- Account creation timestamp
- Soft-delete flag and window end timestamp
- Settings (visit notifications opt-in, reduced-motion preference, caption preference)

### Bird
- Stable internal UUID (identity, never reused)
- User-assigned name
- Species (draws from small pool of ~6 species)
- Personality vector: boldness, social warmth, vocal frequency, plumage saturation, curiosity (normalized, server-stored)
- Current mood (wary, content, curious, drowsy, alert)
- Perch position (front, middle, back — derived from mood/personality)
- Call timing state (procedural grammar state)
- Adoption timestamp

### Presence
- User UUID
- Session start timestamp
- Last activity timestamp (pointermove/keypress)
- Document visibilityState
- Window focus state
- Presence events accumulated into presence-time for drift

### Interaction Events (append-only)
- Timestamp
- User UUID
- Bird UUID (where applicable)
- Event type: `listen_in_start`, `listen_in_end`, `offer`, `settle`, `presence_ping`
- Payload (e.g., offer type, listen-in duration)

### Notebook Entries
- Timestamp
- Entry prose (naturalist, lowercase, present-tense)
- Context (e.g., "morning", "evening", "after rain")

### Visit Invitations
- Host UUID
- Visitor email
- Token (one-time use, expires 30 days)
- Issued timestamp
- Revoked timestamp (nullable)
- Visit log entries (visitor email, start timestamp, end timestamp)

### Simulation Tick State
- Last tick timestamp
- Pending drift deltas per bird (computed from event log since last tick)
- Ambient weather state (current event, duration remaining)

## 4. API Surface

### State Retrieval
- `GET /api/state` — returns current canonical state snapshot
  - Birds with positions, moods, personality vectors, call timing state
  - Ambient weather state
  - Day/night phase (derived from user's timezone)
  - Notebook entries (last N, newest first)
  - Account settings

### Event Submission
- `POST /api/events` — append interaction events to event log
  - Body: array of events with timestamp, type, payload
  - Server confirms receipt; no per-event response

### Presence Pings
- `POST /api/presence` — lightweight endpoint for presence updates
  - Body: `visibilityState`, `hasFocus`, `lastActivityTimestamp`
  - Server records presence-event if all three conditions met

### Account Operations
- `POST /api/auth/magic-link` — request magic link for email
- `POST /api/auth/verify` — verify magic link, issue session token
- `GET /api/account/export` — trigger export generation, return download link
- `POST /api/account/delete` — initiate soft deletion
- `POST /api/account/restore` — restore soft-deleted account
- `POST /api/account/email/change` — request email change
- `POST /api/account/email/verify` — verify new email
- `GET /api/account/sessions` — list active sessions
- `POST /api/account/sessions/revoke` — revoke specific session

### Visit Invitations
- `POST /api/visits/invite` — issue new invitation
- `GET /api/visits/invitations` — list active invitations
- `POST /api/visits/revoke` — revoke specific invitation
- `GET /api/visits/log` — visit log (host only)
- `GET /api/visits/aviary/{token}` — visitor endpoint, returns read-only state snapshot

## 5. Simulation Engine Design

### Server-Side Tick (once per minute)
1. Read event log since last tick
2. Compute presence-time from presence pings
3. Apply drift deltas to personality vectors:
   - Presence-time → monotonic increase in expressive traits
   - Listen-in duration → social warmth, vocal frequency
   - Offer acceptance → curiosity
   - Offer attempt near bird → boldness
4. Update mood transitions:
   - Time-of-day signal (drowsy near dusk, alert morning)
   - Ambient weather (rain dampens vocal frequency, wind increases alertness)
   - Recent interactions (offer accepted → toward content)
   - Personality vector (high boldness → less likely wary)
5. Advance ambient weather (passing rain, wind events)
6. Generate notebook entries if conditions met (rare, ~one every few days for active users)
7. Write updated state to canonical store
8. Clear event log window (keep only recent for debugging)

### Drift Function
- Low-pass filter over presence-and-interaction signals
- Calibration target: measurable drift in instruments after ~1 week, visible to user after ~3 weeks
- Monotonic toward expressive: traits move up on positive presence, never down on neglect
- Personality vectors never exposed to user numerically

### Mood Transitions
- Fast-timescale, resets daily-ish
- Mood persists across sessions (bird at session-end is same mood at next session-start, modulated by tick)
- Mood shapes idle motion (wary → back perch, scanning; content → preening; curious → investigate sounds; drowsy → low perch, fluffed)

### Call Grammar Runtime
- Each bird has procedural call grammar with motifs
- Personality-shaped timing (vocal frequency → call frequency)
- Personality-shaped pitch (plumage saturation → pitch richness)
- Mood-shaped variation (content → fuller calls, wary → shorter calls)
- Calls synthesized client-side via WebAudio (procedural, never looped)

## 6. Sync Model

### Canonical State
- Server is sole writer of personality vectors and mood
- Clients never write personality state directly
- Clients write interaction events to append-only log
- Simulation tick consumes event log in order

### Multi-Device Sync
- All clients read from same canonical state
- No client-to-client sync
- No client-side state to merge
- No eventual consistency to reconcile

### Conflict Prevention
- Additive server-authored deltas, not client-submitted absolute values
- No last-write-wins on personality state
- Event log processed in order by server tick

### Client State Consumption
- Client pulls state snapshot on visibility change
- Client pulls fresh snapshot on long render-frame gaps (laptop resume)
- Client pulls on low-frequency keepalive while visible
- Client interpolates between snapshots for smooth motion

## 7. Frontend Rendering Pipeline

### Scene Composition
- Single horizontal scene, no panning/scrolling
- Three perch zones: front, middle, back
- Ambient background (sky, foliage)
- Foreground elements (branches, leaves)
- Parallax: subtle, not parallax-heavy

### Idle Motion
- Continuous micro-motion regardless of user attention
- Mood-shaped idle: wary bird scans, content bird preens, curious bird investigates, drowsy bird sits low
- Ambient leaf/feather drift (client-side, idle cadence)

### Transitions
- Day/night cycle (user's local time)
- Settle gesture (evening lighting over few seconds)
- Listen-in mix (gradual rise in focused bird, gradual drop in others)
- Ambient weather (soft transitions, no thunderstorms)

### Reduced-Motion Mode
- Cross-fade between still poses (not animated frame-by-frame)
- Ambient leaf drift removed
- Ambient color shifts remain, slowed
- Calls still play at full quality
- Screen-reader narration unchanged

### Loading Sequence
- No entry animation, no fade-from-static
- First frame has birds mid-action
- Server-side simulation tick ensures aviary continues without viewer
- If state snapshot delayed, show quiet field (not spinner)

### Performance Budgets
- Initial JS bundle <2MB (gzipped)
- Time to first bird visible <500ms (mid-tier mobile, 4G)
- 60fps idle motion on 5-year-old laptop
- No memory growth over 30-minute session

## 8. Audio Pipeline

### Procedural Call Synthesis
- WebAudio-based client-side synthesis
- Motif library per species
- Personality-shaped timing (vocal frequency → call frequency)
- Personality-shaped pitch (plumage saturation → pitch richness)
- Mood-shaped variation (content → fuller calls, wary → shorter calls)
- Procedural calls never looped; always varied at runtime

### Chorus Mixing
- Real-time mixing of multiple birds
- Listen-in: gradual mix change (not hard cut)
- Other birds drop in mix but never go silent
- Ambient mix level for un-focused birds

### WebAudio Fallback
- If WebAudio unavailable, play in graceful silence
- Captions on by default in fallback mode
- No recorded audio fallback (bundle budget and canned audio concerns)

### Accessibility
- Call captions (short prose descriptions, same naturalist voice)
- Screen-reader narration (running prose, naturalist voice, slow cadence)
- Reduced-motion mode (cross-fade rendering, not "animations off")

## 9. Accessibility Surfaces

### Screen-Reader Narration
- Naturalist prose, lowercase, present-tense
- Slow cadence: ~1 update per 30–60 seconds at idle
- Faster on user-initiated events (return-greeting, offer reaction, settle)
- Generated from same state as visual surface
- Same voice as field notebook

### Reduced-Motion Mode
- `prefers-reduced-motion` detection
- Opt-in via accessibility settings
- Cross-fade rendering (not "animations off")
- Ambient leaf drift removed
- Calls still play at full quality

### Call Captions
- Short prose descriptions of call sounds
- Naturalist voice
- Runtime-generated per call (not fixed strings)
- Appear near calling bird, fade with call

### WCAG AA Contrast
- All user-copy text passes WCAG AA
- Focus indicators visible against aviary background
- High-contrast outline for keyboard navigation

### Keyboard Navigation
- Tab through top bar items
- Enter focuses first bird
- Arrow keys move focus between birds
- Enter triggers listen-in on focused bird
- Escape exits listen-in
- Offer affordance reachable via top-bar shortcut
- Settle gesture reachable from top bar

## 10. Performance Budgets and Observability

### Budgets
- Initial JS bundle <2MB (gzipped)
- Time to first bird visible <500ms (mid-tier mobile, 4G)
- 60fps idle motion on 5-year-old laptop
- No memory growth over 30-minute session
- Procedural audio synthesized client-side (no recorded audio)
- WebAudio fallback: graceful silence with captions on

### Observability
- Synthetic performance checks (automated browsers, common geographies)
- Aggregate Real User Monitoring:
  - Page load timings
  - First-bird-render timings
  - Render-frame timings
  - Audio-context errors
  - Simulation-tick latencies
- Error budget: simulation-tick latency p99 alarms if >5 seconds

### Browser Support
- Last two major versions of Chrome, Safari, Firefox, Edge
- Older browsers receive matter-of-fact unsupported surface

## 11. Rollout

### V1 Shipping
- Single canonical aviary per account
- Two starter birds (user does not pick from catalog)
- Max seven birds per aviary
- Single-user accounts only
- Magic-link sign-in
- Multi-device sync via server-side canonical state
- Field notebook with auto-generated naturalist observations
- Presence accounting for personality drift
- Listen-in, offer, settle interactions
- Visit invitations (opt-in, read-only)
- Screen-reader narration, reduced-motion mode, call captions

### Bird Count Ramp
- Start with two birds per account
- New birds available based on aviary age (not visit count, not interaction score)
- A few months old → third bird offer
- A year old → may have five or six birds
- Cap at seven birds (empirical limit for call signature recognizability)

### Instrumentation from Day One
- Presence-time accumulation (for drift calibration)
- Personality drift measurements (instrumental, not visible to user)
- Session duration histograms (anonymized, no per-account dimension)
- Render-frame timing (client-side)
- Simulation-tick latency (server-side)
- Audio-context error counts
- Visit invitation usage (aggregate, no per-account PII)

### Rollout Constraints
- No push notifications
- No email about aviary activity
- No engagement metrics (streaks, visit counts)
- No public discovery or leaderboards

## 12. Risks

### Drift Calibration
- **Risk**: Drift too fast → birds change visibly between sessions; drift too slow → users feel nothing they do matters
- **Mitigation**: Calibration target named (measurable in ~1 week, visible in ~3 weeks); instruments to measure drift; A/B test of presence-time thresholds

### Sync Correctness
- **Risk**: Last-write-wins on personality state → drift lost silently; client-side state → divergent simulations
- **Mitigation**: Server-only writer of personality; additive deltas; event log processed in order; no client-to-client sync

### Audio Uncanniness
- **Risk**: Procedural calls sound canned; chorus sounds like stacked loops instead of real-time mixing
- **Mitigation**: Real procedural synthesis (no looped audio); personality-shaped variation; mood-shaped calls; chorus mixing with gradual transitions

### Accessibility Regressions
- **Risk**: Reduced-motion mode feels degraded; screen-reader narration feels like a checklist; captions feel tacked-on
- **Mitigation**: Accessibility as first-class design surface, not checklist; naturalist prose for all accessible content; reduced-motion as designed aesthetic, not fallback; captions in same voice as field notebook

### Mood Persistence Across Sessions
- **Risk**: Mood snaps to neutral on tab open → aviary feels reset, not continued
- **Mitigation**: Mood persisted at session-end; tick modulates mood based on time-of-day and ambient events; no "wake up" animation

### Presence Accounting
- **Risk**: Presence too loose → users who leave tab open generate false drift; presence too strict → users who watch without moving generate no drift
- **Mitigation**: Three-condition conjunction (visibility + focus + recent activity); calibrated activity window (few minutes); instruments to measure presence-time distribution

### Notebook Entry Frequency
- **Risk**: Entries too frequent → noise; entries too rare → user never sees them
- **Mitigation**: Target ~one entry every few days for active users; more often on noteworthy events; test with real usage data

### Visit Feature Misuse
- **Risk**: Users interpret read-only visit as co-presence; visitors expect to interact; hosts expect notifications
- **Mitigation**: Clear "read-only ambient" language; no co-presence affordances; visit notifications opt-in; matter-of-fact error surfaces for expired/revoked visits

### Personality Vector Exposure
- **Risk**: Users discover personality numbers → relationship becomes stat management
- **Mitigation**: Numbers never exposed; personality felt by watching bird; drift visible only by comparing over weeks

### Backward Compatibility
- **Risk**: Data model changes break existing accounts; simulation tick changes corrupt drift history
- **Mitigation**: Versioned data model; migration scripts; simulation tick changes tested against historical event logs; synthetic tests of drift calibration

### Browser Support Gaps
- **Risk**: WebAudio unavailable on older browsers; rendering pipeline fails on older devices
- **Mitigation**: Last two major versions only; graceful fallback (silence with captions); matter-of-fact unsupported surface

### Privacy Boundary
- **Risk**: Per-bird interaction state accidentally included in telemetry; PII leaked through identifiers
- **Mitigation**: Synthetic UUID as internal identifier; email encrypted, stored once; telemetry pipelines never touch simulation database; ML training never receives per-bird fields
