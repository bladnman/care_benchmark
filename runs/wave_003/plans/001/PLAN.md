# Pocket Aviary — Implementation Plan v1

## Scope

### In scope for v1
- Single canonical aviary per account, 2–7 birds
- Magic-link authentication, single-user accounts
- Server-side simulation tick (~1/min), personality drift, mood system
- Procedural call synthesis via WebAudio, per-bird call signatures
- Multi-device sync via canonical server state (no client-to-client sync)
- Field notebook: auto-generated naturalist prose entries, read-only
- Listen-in interaction: focused bird rises in mix, others quiet to ambient
- Offer interaction: seed, song fragment, still pool — per-bird cooldown
- Settle gesture: soft evening transition, opt-in session-end
- Day/night cycle anchored to user's local timezone
- Ambient weather (rare rain, occasional wind), ambient micro-motion
- Visit invitations: read-only ambient view, opt-in, revocable, email-based
- Screen-reader narration in naturalist prose, ~30–60s cadence
- Reduced-motion mode: cross-fade rendering, not stripped animation
- Call captions in naturalist voice, generated from procedural grammar
- WCAG AA contrast, full keyboard navigation
- Account export (JSON snapshot), soft-delete (30-day window)
- Aggregate operational telemetry only

### Out of scope for v1
- Native mobile apps (web-only)
- Gamification: no streaks, no achievements, no badges, no leaderboards
- Tamagotchi mechanics: no hunger, no death, no visible distress, no decay
- Social network surfaces beyond quiet visit invitations
- Shared aviaries, multi-profile accounts
- Customizable scenes, panning, scrolling

### Non-goals respected
- Personality vectors never exposed numerically to users
- No "welcome back" toasts or banners
- No visit-frequency surfaces surfaced to users
- No co-presence during visits
- No recorded audio fallback

---

## Architecture

### Service shape
Three primary services:

1. **Auth Service** — magic-link issuance/validation, session tokens, device management, email change flow. Stateless; session tokens are JWTs signed with a service key.

2. **Simulation Service** — the canonical aviary state writer. Owns the per-account bird personality vectors, mood states, presence accumulation, and drift computation. Runs the server-side tick. Is the only service that writes personality state. Clients never write personality directly.

3. **Client Application** — single-page web app served from CDN. Fetches state snapshots from the Simulation Service, renders the aviary, synthesizes audio client-side, writes interaction events to the append-only event log.

### Client/server split
- **Server owns state**: personality vectors, moods, presence-time accumulator, drift history, bird identity. Client is a render-only consumer.
- **Client owns rendering**: all visual animation, audio synthesis, user input handling, local interpolation between snapshots.
- **Boundary**: client pulls snapshots; client writes events. Client never writes personality. Server tick consumes events, updates personality, publishes new snapshot.

### Render pipeline boundary
The client receives a snapshot (kilobytes) and begins rendering immediately. Interpolation between snapshots smooths bird movement. The first frame shows birds mid-action; there is no entry animation or spinner.

---

## Data Model

### Bird
```
Bird {
  id: UUID (stable, never recycled)
  aviary_id: FK
  species: enum (from species pool)
  name: string (user-assigned, mutable)
  personality_vector: {
    boldness: float [0,1]
    social_warmth: float [0,1]
    vocal_frequency: float [0,1]
    plumage_saturation: float [0,1]
    curiosity: float [0,1]
  }
  mood: enum (wary | content | curious | drowsy | alert)
  mood_updated_at: timestamp
  created_at: timestamp
}
```

### Personality vector persistence
- Stored server-side in Simulation Service
- Never derived from session history at runtime
- Never recomputed from event logs
- Never sent to client as absolute values
- Drift applied as additive delta from previous tick

### Mood
- Small enumerated state per bird
- Resets on daily-ish cadence (tick-driven), modulated by interactions, time-of-day, ambient events
- Persists across sessions (not reset on tab open)
- Transitions shaped by: recent session interactions, user's local time-of-day, ambient weather events, bird's own personality

### Presence event
```
PresenceEvent {
  account_id: FK
  started_at: timestamp
  duration_seconds: int
  conditions_met: {
    visibility_visible: bool
    window_focused: bool
    recent_input: bool  // pointermove or keypress in last N minutes
  }
}
```
Presence is recorded only when all three conditions hold simultaneously.

### Interaction event (append-only log)
```
InteractionEvent {
  id: UUID
  account_id: FK
  bird_id: FK (nullable for general aviary events)
  event_type: enum (listen_in_start | listen_in_end | offer | settle | presence_ping)
  metadata: JSON
  occurred_at: timestamp
}
```
Clients write to this log. Simulation tick consumes it in order to compute drift.

### Field notebook entry
```
NotebookEntry {
  id: UUID
  account_id: FK
  prose: string (naturalist voice, lowercase, present-tense)
  authored_at: timestamp
}
```
Entries are auto-generated by the Simulation Service on notable moments (~every few days per active aviary).

### Visit invitation
```
VisitInvitation {
  id: UUID
  host_account_id: FK
  visitor_email: string (encrypted at rest)
  token: string (one-time link)
  status: enum (pending | accepted | revoked | expired)
  created_at: timestamp
  expires_at: timestamp
}
```

---

## API Surface

### Authentication
- `POST /auth/magic-link` — send magic link to email
- `GET /auth/magic-link/verify?token=` — validate link, issue session JWT
- `POST /auth/session/revoke` — revoke a device session
- `GET /auth/sessions` — list active sessions

### Aviary state
- `GET /aviary/snapshot` — current canonical state (all birds, moods, positions, active weather, time-of-day)
  - Response: `{ birds: [...], time_of_day: {...}, weather: {...}, updated_at: timestamp }`
  - Snapshot is small (kilobytes); client interpolates between pulls
- `GET /aviary/events?since=` — events since timestamp (for catch-up after offline)

### Interaction events
- `POST /events` — append interaction event (listen_in, offer, settle, presence_ping)
  - Body: `{ bird_id, event_type, metadata, occurred_at }`
  - Returns: `{ event_id }`
- These are write-only from client perspective; Simulation Service consumes them

### Notebook
- `GET /notebook/entries?since=` — fetch entries (read-only for client)

### Visits
- `POST /visits/invite` — create one-time visit link
- `GET /visits` — list outstanding invitations and recent visits
- `DELETE /visits/invitations/` — revoke invitation
- `GET /visits/aviary?token=` — visitor fetch of read-only snapshot

### Account
- `GET /account/export` — generate JSON snapshot of aviary state
- `DELETE /account` — initiate soft-delete
- `POST /account/recover` — cancel soft-delete within 30-day window

---

## Simulation Engine Design

### Server-side tick
- Runs at ~1/minute cadence per active account
- Continues regardless of client connection (aviary runs on server)
- Reads the append-only interaction event log since last tick
- Computes and applies personality drift as additive delta
- Transitions moods based on: time-of-day, recent events, ambient weather, personality
- Writes updated canonical snapshot to storage
- Generates notebook entries on notable triggers (not every tick)

### Drift function
- **Primary input**: presence-time accumulated since last tick
- **Secondary inputs**: listen-in duration per bird, offer events, settle gestures
- **Calibration target**:
  - Measurable drift in instruments after ~1 week of regular visits
  - User-visible drift after ~3 weeks
- **Monotonic toward expressive**: traits only increase; neglect results in ambient quietude, not regression
- **Low-pass filter**: no single session shifts traits visibly

### Mood transitions
- Daily-ish reset cadence via tick
- Modulated by: recent interactions (per event log), user local time-of-day, ambient weather events, bird personality vector
- Mood persists across sessions; not reset on tab open

### Call grammar runtime (server)
- Server holds per-species motif libraries (not audio files — motif descriptors)
- Motif descriptors describe: pitch contour shapes, rhythm patterns, frequency ranges, ornamentation rules
- On tick, server computes per-bird call timing and sends as part of snapshot
- Client synthesizes actual audio from motif descriptors via WebAudio
- Per-bird vocal_frequency trait shapes: call frequency when unobserved, chorus join probability

### Bird-to-bird interaction
- Calls from one bird can prompt responses from another (driven by event log)
- Wary mood can spread across aviary
- Chorus events emerge when multiple high-vocal-frequency birds call in same window

---

## Sync Model

### Single canonical source
- Simulation Service is the only writer of personality state
- All clients read the same snapshot
- No client-to-client sync; no state merging required
- "No last-write-wins" rule: personality is additive server-authored delta, never client-submitted absolute value

### Client snapshot pull strategy
- On visibility change (tab becomes visible)
- On long render-frame gap (laptop resuming from sleep)
- Low-frequency keepalive while tab is visible (~every 5 min)
- Client interpolates smoothly between snapshots

### Conflict prevention
- Only server writes personality vectors
- Clients write only interaction events to append-only log
- Tick processes events in order; no concurrent writes to personality

---

## Frontend Rendering Pipeline

### Scene composition
- Single horizontal scene, fits one viewport, no panning/scrolling
- Three perch zones: front, middle, back (proximity to viewer)
- Bird perch choice driven by mood and personality (not user-controlled)
- Foreground/background separation with subtle parallax

### Responsive behavior
- Scene compresses/widens with viewport while keeping all birds visible
- Aspect ratio preserved; birds never cropped or drift offscreen

### Idle micro-motion
- Continuous preening, scanning, head-tilting, body-shuffle
- Mood-shaped: wary = further back, scanning; content = preening; curious = tilts toward sounds; drowsy = low on perch, fluffed
- Runs regardless of user attention; does not pause on tab hidden

### Day/night cycle
- Anchored to user's local timezone
- Morning: warming palette; midday: brightest; evening: warmer hues, quieter calls; night: dim, most birds settled, nightjar-like species may remain active
- Full nightjar species is the exception to ambient quietness

### Ambient weather
- Rare rain (a few times/week), occasional wind
- Never assertive; short duration
- Affects mood in small ways (dampens vocal frequency briefly, wind makes some alert/wary)

### Ambient leaf/feather drift
- Pure rendering ornaments, generated client-side at idle cadence
- Not driven by simulation tick; no per-leaf state

### Loading state
- If snapshot is slow: quiet field (soft sky color, faint motion cues) — not a spinner
- First frame shows birds mid-action, ambient drift already running
- No entry animation, no fade-from-static, no "ready" pop

### Transitions
- Listen-in engage/disengage: slow mix-level ramp (not hard cut)
- Settle: slow lighting shift to evening over seconds, calls quiet
- Settle undo: click within 5 seconds reverses

### Top bar
- Contains: account/settings, accessibility settings, field notebook, offer affordance
- Fades nearly transparent after cursor stillness; returns on movement
- No chrome inside aviary scene

### Reduced-motion mode
- `prefers-reduced-motion` or opt-in via accessibility settings
- Cross-fades between still poses instead of frame animation
- Flight transitions: cross-fades between perches (not animated paths)
- Ambient color shifts (day/evening): remain, slowed
- Calls still play; birds still drift; notebook still notices
- Not a stripped fallback; its own designed surface

### Empty-aviary state
- Between adoption and first bird: quiet field
- First bird enters with soft fly-in to starting perch
- After adoption, aviary is never empty again

---

## Audio Pipeline

### Procedural call synthesis
- WebAudio client-side synthesis from motif descriptors
- Per-species motif library: pitch contour shapes, rhythm patterns, frequency ranges, ornamentation rules
- Personality-shaped timing and pitch variation
- Two birds produce real chorus, not stacked loops
- Recognizable per-bird signature across mood and drift (user learns Pip's call by ear)

### Chorus mixing
- Multiple birds calling simultaneously: real-time mixing with per-call variation
- Phase-canceling artifact from layered loops avoided by procedural synthesis
- Vocal frequency trait shapes: call rate when unobserved, chorus join probability

### Listen-in mix
- Focused bird rises gradually in mix
- Other birds drop to ambient (never silent)
- Gradual ramp on engage and disengage (not hard cut)

### WebAudio fallback
- If WebAudio unavailable (older browser, permission denied, hardware issue): graceful silence with captions on by default
- No recorded audio fallback under any circumstances

### Bundle budget alignment
- Procedural synthesis required because recorded audio at needed variation exceeds 2MB JS bundle
- Species pool ~6 species with distinct call signatures

---

## Accessibility Surfaces

### Screen-reader narration
- Running naturalist prose narration updated ~30–60s at idle
- Faster on user-initiated events (return-greeting, offer reaction)
- Generated server-side or client-side from same state as visual surface
- Naturalist voice: lowercase, present-tense, specific, observation-style
- Example: "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
- Never: "Pip is at perch 2" or "Wren mood: content"
- Voice continuity with field notebook

### Call captions
- Opt-in via accessibility settings
- Short prose descriptions: "a soft three-note rise" / "a low trill, paused, low trill again"
- Generated from procedural call grammar at runtime
- Appear near calling bird, fade in/out with call
- Same naturalist voice as rest of product

### Reduced-motion mode
- See Frontend Rendering Pipeline above
- Not a stripped fallback; cross-fade rendering is its own aesthetic

### WCAG AA contrast
- All user-copy text: top bar labels, settings, account surfaces, error surfaces, captions, narration (when visual)
- Design system specifies actual ratios per surface
- Aviary scene itself has no user copy (contrast applies to chrome)

### Keyboard navigation
- Tab: moves through top bar items
- Enter aviary: focuses first bird
- Arrow keys: move focus between birds
- Enter on focused bird: triggers listen-in
- Escape: exits listen-in
- Top bar shortcuts: offer affordance, settle gesture
- Focus indicators: soft high-contrast outline, visible against bright and dim aviary states

---

## Performance Budgets and Observability

### Initial JS bundle <2MB (gzipped)
- Drives: procedural audio synthesis, generated visual assets, aggressive code-splitting for secondary surfaces
- Code-split: account settings, accessibility settings, visit-invitation flow

### Time to first bird visible <500ms
- On mid-tier mobile, 4G connection
- Requires: bundle budget, fast CDN edge delivery of initial snapshot, non-blocking asset loading
- Above 500ms: user notices load; below: aviary feels already running

### 60fps idle motion on 5-year-old mid-range laptop
- Runtime budget for 30-minute session (not just first minute)

### No memory growth over 30 minutes
- Procedural audio buffers reused
- No per-call allocation without freeing
- Notebook entries: no reference retention after scroll-out
- Worker threads and audio contexts bounded
- CI test, not guideline

### Performance observability
- Synthetic checks: automated browsers on schedule from common geographies
- Aggregate-only RUM: page load timings, first-bird-render, render-frame timings, audio-context errors, tick latencies
- **Privacy boundary honored**: no per-bird state, no per-account interaction history in telemetry
- **Error budget**: tick latency p99 alarm at 5 seconds

### Browser support
- Last two major versions: Chrome, Safari, Firefox, Edge
- Unsupported browser: matter-of-fact surface explaining requirements

---

## Rollout

### v1 ship
- Two birds at adoption; user names them
- System selects two species from pool (not catalog pick)
- First encounter: birds that arrived, not birds user chose
- Field notebook, presence accounting, visit invitations (off by default)
- Screen-reader narration, reduced-motion mode, call captions
- Magic-link auth, multi-device sync

### Bird count ramp
- Third bird available based on aviary age (not visit count, not interaction score, not paid tier)
- Pacing: few months old → third bird; year-old → five or six birds
- Seven bird cap: empirical ceiling where call signatures remain individually recognizable

### Day-one instrumentation
- Aggregate operational telemetry (as above)
- Tick latency monitoring
- Synthetic performance checks
- Bundle size tracking
- First-bird-render timing

### Deliberately not instrumented
- Per-bird interaction state for any aggregate purpose
- Population-level drift analytics
- Visit-frequency metrics surfaced to users
- Any pipeline that could later expose per-account bird behavior

---

## Risks

### Drift calibration
- **Risk**: Drift function too fast → Tamagotchi feel; too slow → screensaver feel
- **Mitigation**: Calibration target is measurable in ~1 week, visible in ~3 weeks. Low-pass filter prevents single-session visibility. Test against target in CI.
- **Risk**: Presence signal inflation (tab-open counted as presence) → corrupted drift across all accounts
- **Mitigation**: Conjunction of three independently-checkable signals (visibility + focus + recent input). Strict server-side enforcement.

### Sync correctness
- **Risk**: Client submitting personality absolute values (last-write-wins) → silent data loss
- **Mitigation**: Server-only personality writes; additive delta model; append-only event log consumed in order. No code path allows client mutation.

### Audio uncanniness
- **Risk**: Looped audio detected → spell breaks; chorus phase-canceling artifact
- **Mitigation**: Procedural synthesis mandatory; no recorded audio fallback; chorus is real-time mixed procedural calls.

### Accessibility regression
- **Risk**: Accessible surfaces built as stripped fallback → reduced-motion users get degraded product
- **Mitigation**: Reduced-motion is its own designed surface; screen-reader narration in naturalist prose; accessibility features ship with product, not after.

### Personality vector exposure
- **Risk**: Numerical values surfaced to users → product collapses into stat management
- **Mitigation**: Hard rule — never exposed numerically, not even in debug views. Protects the affective contract.

### Server tick load
- **Risk**: Many accounts with infrequent tick → tick backlog
- **Mitigation**: Tick cadence is slow (~1/min); tick computation is lightweight; error budget alarm at p99 5s.

### Visit feature scope creep
- **Risk**: Pressure to add co-presence, chat, public discovery, leaderboards
- **Mitigation**: Explicit non-goals documented; visit is read-only ambient; refusal is structural, not policy.

### Magic-link security
- **Risk**: Token replay, email enumeration
- **Mitigation**: Used links invalidated immediately; rate-limiting per email; synthetic UUID for all internal identifiers.

### Synthetic performance regression
- **Risk**: Bundle growth, render regression, memory growth over time
- **Mitigation**: CI tests for bundle size, memory growth, 60fps idle; synthetic fleet monitoring; error budgets with alarms.
