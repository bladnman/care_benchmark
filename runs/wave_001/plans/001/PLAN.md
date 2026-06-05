# Plan: Pocket Aviary v1 — Comprehensive Implementation Plan

**Run:** 001  
**Wave:** wave_001  
**Model:** qwen3-coder  
**Effort Level:** unknown  
**Harness:** opencode  
**Provider:** qwen  

---

## 1. Scope

### v1 In Scope

- **Birds:** 2 starter birds (selected by system), max 7 per aviary. Birds drawn from a pool of ~6 species. Each bird has stable internal identity, user-assigned name, and persistent personality vector.
- **Interactions:** Listen-in (audio focus), Offer (seed/song/pool), Settle (evening transition), Field Notebook (auto-generated observations), Return-greeting (procedural bird notice).
- **Account Model:** Single-user, magic-link sign-in, email+UUID synthetic ID, per-device session tokens, soft-delete (30 days) then hard-delete.
- **Multi-Device Sync:** Server-side canonical state, client pulls snapshots, additive server-authored personality deltas, no client-to-client sync.
- **Simulation:** Server-side tick (~once per minute), client-side interpolation between snapshots.
- **Presence:** Precisely defined (visibilityState + window focus + pointer/keypress in last few minutes), used as primary drift input.
- **Audio:** Procedural call synthesis via WebAudio, chorus mixing, listen-in mix decay, WebAudio fallback to silent captions.
- **Accessibility:** Screen-reader narration (naturalist prose), reduced-motion mode (cross-fade rendering), call captions, WCAG AA contrast, full keyboard navigation.
- **Social (Optional):** Read-only visit invitations (email-based, one-time links), revocable, logged silently, no co-presence, no chat, no friend notifications by default.
- **Visual Scene:** Single horizontal screen, three perch zones (front/middle/back), day/night cycle (local time), ambient weather (rain/wind), ambient micro-motion (leaf/feather drift), calm naturalist color palette.

### v1 Out of Scope (per `non_goals.md`)

- **Native apps:** Web-only at v1.
- **Gamification:** No achievements, no streaks, no levels, no scores, no badges, no XP, no calendar of green dots, no visit-frequency surface.
- **Tamagotchi mechanics:** Birds do not die, do not get hungry, do not show distress, no happiness meter decay.
- **Social network surfaces:** No profiles, no follows, no public feed, no shared discovery, no comments on visits.

### Explicit Out-of-Scope Interactions

- Streak counter (any form)
- "Welcome back!" toast or banner
- Bird catalog selection (starter birds selected by system)
- Customizable scene (colors, layout, panning)
- Multi-aviary accounts
- Public discovery or leaderboards
- Push notifications

---

## 2. Architecture

### Service Shape

**Backend:**
- **State Service (REST/JSON):** Single canonical aviary state per account. Reads snapshots, writes interaction events.
- **Simulation Service (Server-side tick):** Processes event log, updates personality vectors, transitions moods, advances ambient state. Runs ~once per minute.
- **Auth Service:** Magic-link generation, expiration, validation. Email encrypted at rest.
- **Event Log Service (Append-only):** Interaction events (offer, listen-in start/end, settle, presence pings). Append-only, consumed by simulation tick.
- **Notification Service (Quiet):** Visit invitation emails, account export emails, account deletion reminders. No push, no in-product alerts.

**Frontend:**
- **Single-page application (SPA):** React or Preact (lightweight, mature ecosystem).
- **Client-side rendering:** Pulls state snapshots, interpolates motion, synthesizes audio, handles presence detection, writes interaction events.
- **Worker Threads:** Separate thread for procedural audio synthesis to avoid main-thread blocking.
- **Service Worker (optional):** For caching static assets and enabling offline state snapshots (read-only, no interaction writes).

### Client/Server Split

- **State Ownership:** Server is sole writer of personality vectors and canonical aviary state. Client is read-only consumer.
- **Rendering Boundary:** Client handles all visual rendering, audio synthesis, and interaction event submission. Server handles simulation tick and state persistence.
- **Event Flow:** Client → Event Log (append-only) → Simulation Tick (server) → Canonical State Update → Client Snapshot Pull.
- **No Client-Side Simulation:** Clients never compute drift or mood transitions; they render the server-provided state.

### Render Pipeline

1. **Initial Load:**
   - HTML with embedded initial state snapshot (server-rendered for speed)
   - JS bundle loads, hydrates state
   - Birds placed at current positions, motion already in progress
   - No spinner, no fade-in, no "ready" animation

2. **Runtime Rendering:**
   - Client pulls state snapshots on visibility change, frame gaps, keepalive
   - Interpolates between snapshots for smooth motion (bird positions, perch transitions)
   - Ambient animations (leaf drift, feather fall) run client-side at idle cadence
   - Reduced-motion mode: cross-fade between still poses instead of frame-by-frame animation

3. **Performance Guardrails:**
   - Bundle size <2MB gzipped
   - Time to first bird <500ms on mid-tier mobile over 4G
   - 60fps idle motion on 5-year-old laptop
   - No memory growth over 30-minute session

---

## 3. Data Model

### Account
- `account_id` (UUID, synthetic)
- `email` (encrypted, stored once)
- `created_at` (timestamp)
- `aviary_id` (UUID, one per account)
- `session_tokens` (array of {token, device_info, created_at, last_used})
- `soft_delete_at` (nullable timestamp, 30-day window)
- `visit_notifications_enabled` (boolean, default false)

### Aviary
- `aviary_id` (UUID)
- `owner_id` (UUID → account)
- `age_days` (integer, computed from created_at)
- `current_time_of_day` (local time zone: 0.0–1.0, 0.0 = midnight, 0.5 = noon, 1.0 = next midnight)
- `current_weather` (enum: none, light_rain, light_wind, etc.)
- `weather_end_time` (nullable timestamp)
- `settled` (boolean, true if user triggered settle)
- `settled_until` (nullable timestamp)
- `notebook_entries` (array of {id, timestamp, text})

### Bird
- `bird_id` (UUID, stable internal identifier)
- `aviary_id` (UUID)
- `species` (enum: 1 of ~6 species)
- `name` (user-assigned string)
- `personality` (object: boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity)
- `mood` (enum: wary, content, curious, drowsy, alert)
- `perch_zone` (enum: front, middle, back)
- `idle_animation_state` (object: current pose, progress, target pose)
- `last_call_time` (timestamp)
- `call_timing_offset` (float, procedural variation)
- `offer_cooldown_until` (nullable timestamp)
- `created_at` (timestamp)
- `adopted_at` (timestamp)

### Interaction Events (Append-only)
- `event_id` (UUID)
- `aviary_id` (UUID)
- `bird_id` (nullable, many events are aviary-level)
- `event_type` (enum: offer, listen_in_start, listen_in_end, settle, presence_ping, etc.)
- `timestamp` (UTC)
- `payload` (JSON, type-specific data)

### Visit Invitations
- `invitation_id` (UUID)
- `host_account_id` (UUID)
- `visitor_email` (string)
- `token` (UUID, one-time use)
- `created_at` (timestamp)
- `expires_at` (timestamp, 30 days)
- `used_at` (nullable timestamp)
- `revoked_at` (nullable timestamp)

### Field Notebook Entry
- `entry_id` (UUID)
- `aviary_id` (UUID)
- `timestamp` (UTC)
- `text` (naturalist prose)
- `source` (enum: daily_summary, notable_event, session_boundary)

---

## 4. API Surface

### State Endpoints

**GET `/api/v1/state?aviary_id={id}`**
- Returns: Full state snapshot (aviary, birds, recent notebook entries, weather, time of day)
- Response size: kilobytes, not megabytes
- Cache: CDN edge with short TTL (1–2 seconds)
- Authentication: Bearer token (session token)

**GET `/api/v1/state?aviary_id={id}&since={timestamp}`**
- Returns: Incremental update (only changed fields since timestamp)
- Optimization for long-running sessions

### Event Endpoints

**POST `/api/v1/events`**
- Body: `{ aviary_id, events: [ { type, bird_id?, payload? } ] }`
- Writes to append-only event log
- Returns: 202 Accepted (no immediate state update)
- Rate limit: per-session token

**Event Types:**
- `offer`: `{ bird_id, offer_type: seed | song_fragment | pool }`
- `listen_in_start`: `{ bird_id }`
- `listen_in_end`: `{ bird_id }`
- `settle`: `{}` (aviary-level)
- `presence_ping`: `{ timestamp, duration_ms }`
- `notebook_scroll`: `{ entry_id }` (read-only, no simulation impact)

### Account Endpoints

**POST `/api/v1/auth/magic-link`**
- Body: `{ email }`
- Returns: 202 Accepted (email queued)
- Rate limit: per-email

**POST `/api/v1/auth/verify`
- Body: `{ token }`
- Returns: `{ account_id, session_token, aviary_id }`
- Validates token, issues session token, invalidates token

**POST `/api/v1/auth/revoke-session`
- Body: `{ session_token }`
- Revokes session token

**GET `/api/v1/account/export`
- Returns: JSON snapshot of aviary state (birds, names, personality vectors, moods, notebook entries)
- Auth: Bearer token
- Response: Download link via email (large payload)

**DELETE `/api/v1/account`
- Soft-deletes account, sets `soft_delete_at`
- 30-day window to recover
- Hard-delete after window

### Visit Invitation Endpoints

**POST `/api/v1/visits/invite`
- Body: `{ aviary_id, visitor_email }`
- Returns: Invitation record
- Auth: Bearer token (host)

**GET `/api/v1/visits/invite/{token}`
- Returns: Aviary state (read-only, no interaction allowed)
- No auth required (one-time token)

**POST `/api/v1/visits/invite/{token}/revoke`
- Body: `{}` (aviary_id optional, inferred from token)
- Revokes invitation
- Auth: Bearer token (host)

**GET `/api/v1/visits/log`
- Returns: List of visits (visitor_email, visited_at, duration)
- Auth: Bearer token (host)

---

## 5. Simulation Engine Design

### Server-Side Tick

**Cadence:** ~once per minute (configurable, calibrated during build)

**Tick Sequence:**
1. Read event log since last tick
2. Process presence events → accumulate presence-time
3. Update mood transitions (time of day, recent interactions, ambient events)
4. Compute personality drift deltas (presence-time, listen-in duration, offers accepted)
5. Apply drift (additive, monotonic toward expressive)
6. Advance weather timers
7. Update ambient micro-motion state (leaf/feather drift)
8. Write canonical state snapshot
9. Generate notebook entries (if conditions met)

### Drift Function

**Inputs (in rough order of weight):**
1. **Presence-time** (dominant): user sitting and watching
2. **Listen-in duration** (strong): focusing a bird
3. **Offers accepted** (small): curiosity drift
4. **Offers near bird** (small): boldness drift
5. **Settle gesture** (quieting): mood, not drift

**Drift Direction:** Monotonic toward expressive. Traits move up on positive presence; never move down on neglect.

**Calibration Target:**
- Measurable drift in instruments after ~1 week of regular visits
- Visible drift to user after ~3 weeks

**Implementation:**
- Low-pass filter over presence-and-interaction signals
- Per-bird, per-trait deltas stored in event log
- Server computes cumulative drift from event log, never client-submitted absolute values

### Mood Transitions

**Mood States:** wary, content, curious, drowsy, alert

**Transition Triggers:**
- Time of day (drowsy near dusk, alert early morning)
- Recent interactions (offer accepted → content, offer ignored → wary)
- Ambient events (rain → dampened vocal frequency, wind → alert/wary)
- Personality vector (high boldness → less likely wary)
- Mood persists across sessions (server-side tick modulates)

**Transition Rules:**
- Mood does not reset on tab open (mood at session-end carries forward)
- Mood transitions are gradual (not instant)
- Mood affects idle motion (wary bird scans, content bird preens, curious bird tilts)

### Call Grammar Runtime

**Procedural Generation:**
- Each bird has a motif library (species-specific)
- Personality-shaped timing (vocal frequency → call rate)
- Mood-shaped pitch and variation
- Real-time chorus mixing (no pre-recorded loops)

**Recognizability:**
- Call signature remains recognizable across mood and personality drift
- Core motifs unchanged; timing and pitch vary
- 7-bird cap: chorus must remain individually recognizable

**WebAudio Synthesis:**
- Client-side via WebAudio API
- Reusable buffers (no memory growth)
- Graceful fallback: silent with captions if WebAudio unavailable

### Idle Motion

**Micro-Motion Types:**
- Preening (content)
- Scanning (wary)
- Head-tilting (curious)
- Body-shuffle (reset weight, all moods)
- Eye state (alert/drowsy)

**Mood-Shaped:**
- Wary bird: further back perch, more scanning
- Content bird: closer perch, preening
- Curious bird: tilts toward sounds, watches leaves
- Drowsy bird: low perch, feathers fluffed, eyes closed

**Implementation:**
- Server tracks animation state (pose, progress, target)
- Client interpolates between states
- Reduced-motion mode: cross-fade between poses instead of frame-by-frame

---

## 6. Sync Model

### Canonical State

**Server is Sole Writer:**
- Personality vectors: server-only writes via simulation tick
- Mood: server-only writes (transition logic)
- Aviary state: server-only writes (weather, time of day)

**Client is Read-Only:**
- Pulls state snapshots
- Interpolates motion
- Writes interaction events (append-only event log)

### Multi-Device Sync

**Architecture:**
- Single canonical aviary per account
- Clients pull same snapshots
- No client-to-client sync
- No eventual consistency to reconcile

**Conflict Prevention:**
- Personality drift: additive server-authored deltas, never client-submitted absolute values
- Event log: append-only, consumed in order by simulation tick
- No last-write-wins on personality state

**Client Behavior:**
- Pull snapshot on visibility change
- Pull snapshot on long render-frame gaps (laptop resume)
- Pull snapshot on keepalive while visible
- Interpolate between snapshots for smooth motion

### Conflict Resolution Surfaces

**Sync Conflict (rare cases):**
- Magic-link replay
- In-flight session timing out mid-write
- Server-side outage

**Surface:**
- Matter-of-fact tone (not naturalist prose)
- Example: "We couldn't sign you in. The link may have expired. Try requesting a new link."
- Account settings: session list with revocation affordance

---

## 7. Frontend Rendering Pipeline

### Initial Load

1. **Server-rendered HTML** with embedded initial state snapshot
2. **JS bundle loads** (2MB gzipped cap)
3. **Hydrates state**, places birds at current positions
4. **Motion already in progress** (no fade-in, no spinner)
5. **First frame** shows birds mid-action, ambient drift already running

### Rendering Sequence

**Main Thread:**
- Scene composition (birds, perches, foliage)
- Bird positioning (interpolated between snapshots)
- Ambient micro-motion (leaf/feather drift, parallax)
- Reduced-motion mode (cross-fade rendering if user preference set)

**Worker Thread:**
- Procedural audio synthesis (WebAudio)
- Call grammar runtime (motif selection, timing, pitch)
- Chorus mixing (listen-in mix decay)

**Performance Guardrails:**
- Bundle size <2MB gzipped
- Time to first bird <500ms on mid-tier mobile over 4G
- 60fps idle motion on 5-year-old laptop
- No memory growth over 30-minute session

### Reduced-Motion Mode

**Implementation:**
- `prefers-reduced-motion` detection (system)
- Accessibility settings toggle (user opt-in)
- Cross-fade between still poses instead of frame-by-frame animation
- Ambient animations removed (leaf drift)
- Color shifts remain, slowed

**Aesthetic:**
- Not "animations off" (degraded variant)
- Its own designed surface (quiet, calm)
- Same product, different visual register

### Ambient Motion

**Client-Side Only (not simulation):**
- Leaf drift (random intervals, slow speed)
- Feather fall (random intervals, slow speed)
- Foreground/background parallax (gentle, not parallax-heavy)
- Ambient color shifts (day → evening, slow)

**Purpose:**
- Visual cue that aviary continues without viewer
- No per-leaf state (client-side generation only)

---

## 8. Audio Pipeline

### Procedural Call Synthesis

**WebAudio Implementation:**
- Client-side synthesis (no downloaded audio files)
- Motif library per species
- Personality-shaped timing (vocal frequency → call rate)
- Mood-shaped pitch and variation
- Real-time chorus mixing

**Buffer Reuse:**
- Reusable audio buffers (no memory growth)
- Pool allocation for repeated motifs
- Worker thread for synthesis (main thread not blocked)

**Fallback:**
- If WebAudio unavailable: silent with captions on by default
- No recorded audio fallback (bundle budget collapse, canned audio break spell)

### Listen-in Mix

**Behavior:**
- User clicks/taps/focuses bird → call rises in mix
- Other birds quiet to ambient (not silence)
- Gradual ramp on engage and disengage (no hard cut)
- Disengage: click same bird, click different bird, click empty space, keyboard focus away

**Mix Curve:**
- Focused bird: +6dB (typical)
- Ambient birds: -12dB (typical)
- Smooth transition (200–500ms ramp)

### Call Captions

**User Opt-In:**
- Accessibility settings toggle
- Captions appear as small text near calling bird
- Fade in/out with call

**Caption Content:**
- Naturalist prose (same voice as field notebook)
- Generated at runtime from procedural call grammar
- Example: "a soft three-note rise", "a low trill, paused, low trill again"

---

## 9. Accessibility Surfaces

### Screen-Reader Narration

**Content:**
- Naturalist prose (same voice as field notebook)
- Running narration updated slowly (30–60 seconds at idle)
- User-initiated events get priority bump (return greeting, offer reaction)
- Not state lists ("Pip is at perch 2")
- Not announcement style ("bird greeted you")

**Example:**
> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

**Cadence:**
- Slow: 30–60 seconds at idle
- Faster only on user-initiated events
- Queue management: high-frequency narration would overwhelm screen reader

### Reduced-Motion Mode

**Implementation:**
- `prefers-reduced-motion` detection
- Accessibility settings toggle
- Cross-fade between still poses (not "animations off")
- Ambient animations removed (leaf drift)
- Color shifts remain, slowed

**Aesthetic:**
- Designed surface, not fallback
- Quiet, calm aesthetic
- Same product, different visual register

### Call Captions

**User Opt-In:**
- Accessibility settings toggle
- Captions appear near calling bird
- Fade in/out with call
- Naturalist prose (same voice as field notebook)

### WCAG AA Contrast

**User-Copy Text:**
- All text in top bar, settings, account surfaces, error surfaces, captions, narration (when displayed visually)
- Passes WCAG AA (minimum)
- Specific ratios per surface in design system

### Keyboard Navigation

**Tab Order:**
- Account/settings
- Accessibility settings
- Field notebook
- Offer affordance
- Aviary scene (first bird)
- Arrow keys move focus between birds
- Enter triggers listen-in
- Escape exits listen-in
- Offer affordance opens with top-bar shortcut

**Focus Indicators:**
- Visible against aviary background
- Soft, high-contrast outline
- Reads against bright and dim aviary states

---

## 10. Performance Budgets and Observability

### Budgets

**Bundle Size:** <2MB gzipped at first paint  
**Time to First Bird:** <500ms on mid-tier mobile over 4G  
**Idle Motion:** 60fps on 5-year-old laptop  
**Memory Growth:** 0 over 30-minute session  
**Simulation Tick Latency:** p99 <5 seconds (alarm threshold)

### What We Measure

**Synthetic Monitoring:**
- Automated browsers (schedule, geographies)
- Page load timings
- First-bird-render timings
- Render-frame timings
- Audio-context errors
- Simulation-tick latencies

**Real User Monitoring (Aggregate Only):**
- Page load timings
- First-bird-render timings
- Render-frame timings
- Audio-context errors
- Simulation-tick latencies
- Session-duration histograms (anonymized, no per-account dimension)

**What We Don't Measure:**
- Per-bird state
- Per-account interaction history
- Any telemetry that could reconstruct user's relationship with their aviary

### Error Budget

**Simulation-Tick Latency:**
- p99 alarm if >5 seconds
- Expected: much less (sub-second)
- Alarm catches degradation early, before users notice

### Browser Support

- Last two major versions of Chrome, Safari, Firefox, Edge
- Older browsers: matter-of-fact unsupported-browser surface
- No compatibility paths for very old browsers (bundle bloat not justified)

---

## 11. Rollout

### v1 Shipping

**Single Account Per Deployment:**
- One canonical aviary per account
- Server-side simulation tick
- Client pulls snapshots
- Multi-device sync via canonical state

**Birds-Per-Aviary Ramp:**
- V1 starts with 2 birds (minimum for small social system)
- Cap at 7 birds (per-bird call signatures must remain individually recognizable)
- No ramp needed (cap is empirical, not arbitrary)

### Instrumentation

**Day 1:**
- Bundle size (gzipped)
- Time to first bird (per device/network)
- Render-frame timing (60fps on 5-year-old laptop)
- Simulation-tick latency (p99)
- Audio-context errors
- Session duration (anonymized)
- Error rates (4xx, 5xx)

**No Day 1:**
- Per-bird state
- Per-account interaction history
- Any telemetry that could reconstruct user's relationship

### Deployment Strategy

**Initial:**
- Single region, CDN for static assets
- Edge caching for state snapshots (short TTL)
- Server-side simulation in app region

**Later:**
- Multi-region if needed
- Regional simulation clusters (one per region)
- Cross-region sync only for failover (rare)

---

## 12. Risks

### Drift Calibration Risk

**Risk:** Drift too fast or too slow, breaking "feels alive over weeks" promise  
**Mitigation:**
- Calibration target named: measurable drift in ~1 week, visible in ~3 weeks
- Instrumentation: per-account drift tracking in test harness
- A/B test: different decay rates, measure user retention

### Sync Correctness Risk

**Risk:** Personality drift corrupted, birds "reset"  
**Mitigation:**
- Server is sole writer of personality vectors
- Additive server-authored deltas, never client-submitted absolute values
- Event log consumed in order by simulation tick
- Audit log: all personality updates (who, when, delta)

### Audio Uncanniness Risk

**Risk:** Procedural calls sound canned, breaking spell  
**Mitigation:**
- Real-time variation (not 3 variants in rotation)
- Personality-shaped timing and pitch
- Mood-shaped variation
- User testing: "Do these calls sound like they're happening now, or playing back?"

### Accessibility Regression Risk

**Risk:** Accessibility features degrade product quality  
**Mitigation:**
- Screen-reader narration: naturalist prose, not state lists
- Reduced-motion: designed surface, not fallback
- Captions: same voice as field notebook
- WCAG AA contrast: design system enforced
- Test harness: automated accessibility checks

### Performance Budget Risk

**Risk:** Bundle size >2MB, time to first bird >500ms  
**Mitigation:**
- Bundle budget enforced in CI
- Code-splitting for low-traffic surfaces (account settings, accessibility settings, visit invitations)
- Lazy loading: only ship what's needed for first render
- Performance tests: synthetic monitoring on schedule

### Presence Detection Risk

**Risk:** Presence signal too lax, corrupting drift  
**Mitigation:**
- Precise definition: visibilityState + window focus + pointer/keypress in last few minutes
- All three conditions required simultaneously
- Test harness: presence detection unit tests
- A/B test: different activity windows, measure drift calibration

### Mood Persistence Risk

**Risk:** Mood resets on tab open, losing continuity  
**Mitigation:**
- Server persists mood across sessions
- Client reads mood from state snapshot
- Test harness: mood continuity test (session-end → session-start)
- User testing: "Did the birds feel like they were still there while you were away?"

---

## 13. Implementation Milestones (High-Level)

### Phase 1: Core Engine
- [ ] Database schema (accounts, aviaries, birds, events)
- [ ] Server-side simulation tick
- [ ] Drift function implementation
- [ ] Mood transition logic
- [ ] Event log append-only service
- [ ] State snapshot endpoint

### Phase 2: Client Rendering
- [ ] SPA foundation (React/Preact)
- [ ] State snapshot hydration
- [ ] Bird rendering (SVG/procedural)
- [ ] Idle motion interpolation
- [ ] Reduced-motion mode
- [ ] Ambient micro-motion (leaf/feather drift)

### Phase 3: Audio
- [ ] WebAudio call synthesis
- [ ] Procedural call grammar runtime
- [ ] Chorus mixing
- [ ] Listen-in mix decay
- [ ] Call captions

### Phase 4: Interactions
- [ ] Listen-in (focus, mix change)
- [ ] Offer (seed/song/pool, cooldown)
- [ ] Settle (evening transition, undo)
- [ ] Return-greeting (procedural, procedural variation)
- [ ] Field notebook (auto-generated entries)

### Phase 5: Accounts & Sync
- [ ] Magic-link auth
- [ ] Session token management
- [ ] Multi-device sync (canonical state)
- [ ] Account export
- [ ] Soft-delete (30-day window)

### Phase 6: Accessibility
- [ ] Screen-reader narration
- [ ] Reduced-motion mode
- [ ] Call captions
- [ ] WCAG AA contrast
- [ ] Keyboard navigation

### Phase 7: Social (Optional)
- [ ] Visit invitation flow
- [ ] Read-only visit endpoint
- [ ] Visit log
- [ ] Visit revocation
- [ ] Visit notification toggle

### Phase 8: Polish & Testing
- [ ] Performance budget validation
- [ ] Accessibility audit
- [ ] User testing (feels alive, not robotic)
- [ ] Drift calibration testing
- [ ] Audio uncanniness testing

---

## 14. Success Criteria

### Affective Success
- Users report "the birds feel like they have their own life"
- Users return because they want to see what the birds are doing, not because of a streak counter
- Users notice personality drift over weeks ("Pip is bolder than she used to be")
- Users feel the aviary continues without them (not frozen on tab close)

### Technical Success
- Bundle size <2MB gzipped
- Time to first bird <500ms on mid-tier mobile over 4G
- 60fps idle motion on 5-year-old laptop
- No memory growth over 30-minute session
- Simulation-tick latency p99 <5 seconds

### Accessibility Success
- Screen-reader users report same affective experience as sighted users
- Reduced-motion users report same affective experience as default users
- Keyboard-only users can complete all interactions
- WCAG AA contrast on all user-copy text

---

## 15. Open Questions (For Engineering Team)

1. **Exact tick cadence:** ~1 minute is the target; should it be configurable per deployment? Should it vary by load?
2. **Presence activity window:** "A few minutes" is the spec; what exact value (e.g., 90 seconds, 120 seconds)?
3. **Drift calibration:** How to measure "measurable drift in instruments after ~1 week"? What instrumentation is needed?
4. **Call caption generation:** Should captions be pre-generated per call type, or generated at runtime? Runtime is more flexible but costs CPU.
5. **Reduced-motion cross-fade duration:** How long should cross-fades be? (Too long feels sluggish, too short defeats the purpose.)
6. **Notebook entry generation logic:** What triggers an entry? Daily summary? Notable events? Both? What's the exact rule?
7. **Ambient weather frequency:** "A few times a week" — what's the exact probability per day?
8. **Bird adoption pacing:** "An aviary a few months old offers a third bird" — what's the exact timeline (30 days? 60 days? 90 days?)?

---

## 16. Design System Notes

### Color Palette
- Soft blues, greens, warm browns, muted ochres
- Saturated UI accent colors (bright reds, electric blues) deliberately absent
- Calm, naturalist aesthetic

### Typography
- System fonts (no webfont load)
- High contrast on user-copy text
- WCAG AA minimum

### Animation
- Idle motion: mood-shaped, personality-keyed
- Transitions: gradual (no hard cuts)
- Reduced-motion: cross-fade between poses

### Voice
- Naturalist (product surface): lowercase, present-tense, specific
- Matter-of-fact (system surface): capitalized, direct, no naturalist phrasing
- Named exception: account, error, sync, accessibility-settings surfaces use matter-of-fact

---

## 17. Dependencies

### Internal
- Auth service (magic-link)
- Database (PostgreSQL or similar)
- CDN (for static assets, short-TTL state snapshots)
- Observability stack (metrics, logs, traces)

### External
- WebAudio API (browser-native)
- `prefers-reduced-motion` (browser-native)
- `visibilityState` (browser-native)

---

## 18. Conclusion

This plan describes a single-user, browser-based virtual aviary where birds live in a small horizontal scene, notice the user, and drift in personality over weeks in response to idle attention and small interactions. The product is closer to a low-key relationship with a window than to a Tamagotchi — observational, not custodial; quiet, not announced; specific, not gamified.

The engine is server-side, client-side rendering pulls snapshots and interpolates motion, and the simulation tick advances canonical state whether or not anyone is watching. The birds feel alive because they have their own continuity, their own mood, their own personality — and the user's attention, measured precisely as presence, shapes that personality over weeks.

The product is what's left after the explicit subtractions: no games, no Tamagotchi mechanics, no social network, no native apps, no gamification of any flavor. It's a small social system the user gets to know intimately — two birds at start, up to seven, each with a name, a personality, a call signature, and a mood that the user reads from idle motion and greeting behavior.

The success metric is not engagement time or session count; it's whether the user feels accompanied rather than entertained, noticed rather than announced at, and whether they return because they want to see what the birds are doing, not because of a streak counter. The product is designed to feel like a place that has been continuing without the viewer, and the implementation plan is the map to that destination.
