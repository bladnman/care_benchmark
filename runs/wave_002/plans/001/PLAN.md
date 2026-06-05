# Pocket Aviary — Implementation Plan
## Run 001, Wave 002 — Phase 1 Comprehensive Plan

---

## 1. Scope

### v1 In Scope
- Single-user accounts with email magic-link sign-in
- One canonical aviary per account (2 starter birds, max 7)
- Server-side simulation tick (~once per minute)
- Multi-device sync via canonical server state
- Field notebook (auto-generated, naturalist prose)
- Presence accounting (visibility + focus + pointer/key activity)
- Bird-to-bird interaction and chorus mechanics
- Personality drift (slow, monotonic toward expressive)
- Mood system (fast-timescale, daily-ish reset)
- Procedural call synthesis via WebAudio
- Listen-in (focus one bird, raise its mix)
- Offer system (seed, song fragment, still pool)
- Settle gesture (soft session-end)
- Visit-invitation feature (read-only, opt-in, per-invite)
- Screen-reader narration (naturalist prose)
- Reduced-motion mode (cross-fade rendering)
- Call captioning (procedural, naturalist voice)
- Accessibility settings (captions, reduced motion, keyboard nav)
- Top-bar chrome (account, accessibility, notebook, offer, settle)
- Ambient micro-motion (leaf/feather drift, parallax)
- Day/night cycle (local time, mood modulation)

### v1 Out of Scope (per `non_goals.md`)
- Native mobile apps (web-only)
- Gamification (no achievements, no streaks, no levels, no scores, no badges)
- Tamagotchi mechanics (no death, no hunger, no distress, no happiness decay)
- Social network surfaces (no profiles, no follows, no public feed, no comments on visits)
- Payments, billing, or premium tiers
- Shared aviaries, household accounts, or multi-user accounts
- Customizable scenes, bird selection, or species catalog
- Multi-aviary accounts or aviary switching
- Public discovery, leaderboards, or rankings
- Push notifications, emails, or pings about the aviary

### Explicit Boundaries
- Bird count cap: 7 (empirical, not arbitrary — audio recognizability ceiling)
- Personality drift: monotonic toward expressive, no negative drift on neglect
- Presence: visibility + focus + pointer/key activity in last few minutes (all three)
- Drift calibration: measurable in instruments after ~1 week, visible to user after ~3 weeks
- Top-bar chrome only in top bar; no UI inside the aviary scene proper

---

## 2. Architecture

### Service Shape
- Single monolithic service for v1 (Node.js/TypeScript on a cloud provider)
- Database: PostgreSQL (canonical state, birds, accounts, events, notebook entries)
- Cache: Redis (session tokens, recent presence pings, short-lived tokens)
- Queue: Simple append-only event log per account (interaction events for tick consumption)
- CDN: Static assets (HTML, JS, CSS, SVGs) served from edge

### Client/Server Split
- **Server responsibilities:**
  - Account management (magic-link auth, session tokens, email change)
  - Canonical state storage (birds, personality vectors, moods, presence-time)
  - Simulation tick (drift, mood transitions, event-log consumption)
  - Event log append (interaction events from clients)
  - State snapshot generation (read-only, all birds, all moods, all positions)
  - Notebook entry generation (naturalist prose, low cadence)
  - Visit-invitation management (create, revoke, log visits)
  - Aggregate telemetry collection (latencies, errors, session durations)

- **Client responsibilities:**
  - Rendering (scene composition, bird animations, ambient motion)
  - Audio synthesis (procedural calls, WebAudio mixer, chorus)
  - State interpolation (between snapshots for smooth motion)
  - Presence detection (visibilityState, focus, pointer/key activity)
  - Interaction event submission (offer, listen-in, settle, presence pings)
  - Keyboard navigation (focus management, shortcuts)
  - Screen-reader narration queue (slow cadence, naturalist prose)
  - Reduced-motion mode (cross-fade rendering, ambient motion reduction)
  - Call captioning (procedural text near calling bird)

### Render Pipeline Boundary
- **Server sends:** State snapshot (JSON, ~kilobytes), containing per-bird positions, moods, call timing, active animations, scene metadata (time-of-day, weather, lighting).
- **Client renders:** Bird positions interpolated between snapshots, idle micro-motion (mood-shaped), ambient leaf/feather drift, day/night lighting shifts, call captions, screen-reader narration.
- **No client-side simulation:** The client never computes drift, mood transitions, or personality updates. It only renders what the server sends.

---

## 3. Data Model

### Account
```json
{
  "id": "uuid",              // synthetic UUID, never email
  "email": "encrypted",      // encrypted, never used as identifier
  "created_at": "timestamp",
  "deleted_at": "null or timestamp", // soft-deletion window
  "settings": {
    "visit_notifications": "boolean", // off by default
    "captions_enabled": "boolean", // off by default
    "reduced_motion": "boolean", // off by default
    "audio_muted": "boolean" // off by default
  }
}
```

### Bird
```json
{
  "id": "uuid",              // stable internal identifier
  "account_id": "uuid",
  "name": "string",          // user-assigned, renameable
  "species": "string",       // from pool (6 species, v1)
  "personality": {
    "boldness": "float 0-1",
    "social_warmth": "float 0-1",
    "vocal_frequency": "float 0-1",
    "plumage_saturation": "float 0-1",
    "curiosity": "float 0-1"
  },
  "mood": "wary|content|curious|drowsy|alert",
  "last_mood_reset": "timestamp", // daily-ish cadence
  "current_perch": "front|middle|back",
  "idle_animation_state": "enum", // preening, scanning, head-tilt, etc.
  "call_timing": {
    "next_call_in_ms": "integer",
    "current_call_motif": "string",
    "call_count_since_session_start": "integer"
  },
  "last_offered_at": "null or timestamp", // per-bird offer cooldown
  "created_at": "timestamp"
}
```

### Presence Event
```json
{
  "id": "uuid",
  "account_id": "uuid",
  "session_id": "uuid",
  "started_at": "timestamp",
  "ended_at": "null or timestamp",
  "total_duration_ms": "integer", // computed at end
  "last_activity_at": "timestamp" // last pointer/key event
}
```

### Interaction Event (append-only per account)
```json
{
  "id": "uuid",
  "account_id": "uuid",
  "bird_id": "uuid",         // null for settle, presence ping
  "type": "offer|listen_in_start|listen_in_end|settle|presence_ping",
  "timestamp": "timestamp",
  "metadata": {              // type-specific
    "offer": {"item": "seed|song|pool", "response": "bird_reaction"},
    "listen_in": {"bird_id": "uuid", "duration_ms": "integer"},
    "settle": {},
    "presence_ping": {"duration_ms": "integer"}
  }
}
```

### Notebook Entry
```json
{
  "id": "uuid",
  "account_id": "uuid",
  "created_at": "timestamp",
  "prose": "string",         // naturalist prose, lowercase, present-tense
  "context": {
    "day_of_week": "string",
    "time_of_day": "string",
    "weather": "string",
    "notable_events": ["array of short strings"]
  }
}
```

### Visit Invitation
```json
{
  "id": "uuid",
  "host_account_id": "uuid",
  "visitor_email": "string",
  "created_at": "timestamp",
  "expires_at": "timestamp",
  "revoked_at": "null or timestamp",
  "used_at": "null or timestamp",
  "visitor_account_id": "null or uuid" // null until first use
}
```

### Visit Log Entry (read-only, host-only)
```json
{
  "id": "uuid",
  "invitation_id": "uuid",
  "visitor_email": "string",
  "started_at": "timestamp",
  "ended_at": "null or timestamp",
  "duration_ms": "integer"
}
```

### Session Token
```json
{
  "token": "uuid",           // stored in DB, hashed
  "account_id": "uuid",
  "created_at": "timestamp",
  "expires_at": "timestamp",
  "revoked_at": "null or timestamp",
  "device_info": "string"    // user-agent + platform
}
```

---

## 4. API Surface

### Client → Server

#### Authentication
- `POST /api/auth/magic-link-request` — email → magic link sent
- `POST /api/auth/magic-link-exchange` — token from link → session token

#### State and Rendering
- `GET /api/state` — current snapshot (birds, moods, positions, call timing, scene metadata)
- `GET /api/notebook` — notebook entries for this account (paginated, newest first)
- `GET /api/birds` — list of birds (names, species, IDs, current perch)

#### Interaction Events (append-only)
- `POST /api/events/offer` — { bird_id, item }
- `POST /api/events/listen_in_start` — { bird_id }
- `POST /api/events/listen_in_end` — { bird_id }
- `POST /api/events/settle`
- `POST /api/events/presence_ping` — { duration_ms }

#### Account Settings
- `GET /api/account/export` — JSON snapshot (birds, vectors, moods, notebook, settings)
- `POST /api/account/delete` — soft-delete (30-day window)
- `POST /api/account/restore` — restore during soft-deletion window
- `PATCH /api/account/settings` — update settings (visit_notifications, captions, reduced_motion, audio_muted)

#### Visit Invitations
- `POST /api/visits/invite` — { visitor_email }
- `GET /api/visits/list` — host's invitation list and visit log
- `POST /api/visits/revoke` — { invitation_id }
- `POST /api/visits/accept` — visitor's one-time link consumption

#### Accessibility
- `GET /api/captions/config` — caption style and positioning

### Server → Client (Server-Sent Events or WebSocket)
- `state_update` — push when server-side tick produces a meaningful change (optional, clients poll)
- `notebook_entry_created` — new entry generated (optional, clients poll)

### Client → Client (Not Supported)
- No client-to-client communication. All sync happens via canonical server state.

---

## 5. Simulation Engine Design

### Server-Side Tick Cadence
- **Cadence:** ~once per minute (calibrated during build)
- **Responsibilities per tick:**
  1. Read interaction events since last tick (append-only log)
  2. Update presence-time (sum of presence pings)
  3. Compute personality drift deltas from presence-time and interaction events
  4. Apply drift deltas to personality vectors (additive, monotonic)
  5. Update mood transitions (recent interactions, time-of-day, ambient events, personality)
  6. Advance idle animation states (mood-shaped)
  7. Generate notebook entries if conditions met (low cadence, ~one every few days)
  8. Write updated canonical state to database
  9. Clear interaction event log (consume and discard)

### Drift Function
- **Inputs (in rough order of weight):**
  1. Presence-time (dominant — user sitting and watching)
  2. Listen-in (strong — focusing a bird)
  3. Offers (small — accepting offers drifts curiosity; offering near birds drifts boldness)
  4. Settle (quiet — ends presence window cleanly, no drift direction)

- **Drift rules:**
  - Traits move up on positive presence; never move down on neglect
  - No single session shifts a trait visibly (slow filter)
  - Calibration target: measurable in instruments after ~1 week, visible to user after ~3 weeks
  - Additive server-authored deltas only — clients never submit absolute values

- **Drift implementation:**
  ```typescript
  // Pseudocode — actual implementation in simulation service
  const driftDelta = {
    boldness: presenceTimeWeight * presenceTime + offerNearWeight * offerNearCount,
    social_warmth: listenInWeight * listenInDuration + greetingFirstWeight * greetingFirstCount,
    vocal_frequency: listenInWeight * listenInDuration + vocalFrequencyBonus * callCount,
    plumage_saturation: presenceTimeWeight * presenceTime,
    curiosity: offerAcceptedWeight * offerAcceptedCount
  };

  // Apply to existing vector (additive, never subtractive)
  const newPersonality = {
    boldness: Math.min(1.0, bird.personality.boldness + driftDelta.boldness),
    social_warmth: Math.min(1.0, bird.personality.social_warmth + driftDelta.social_warmth),
    vocal_frequency: Math.min(1.0, bird.personality.vocal_frequency + driftDelta.vocal_frequency),
    plumage_saturation: Math.min(1.0, bird.personality.plumage_saturation + driftDelta.plumage_saturation),
    curiosity: Math.min(1.0, bird.personality.curiosity + driftDelta.curiosity)
  };
  ```

### Mood Transitions
- **Mood states:** wary, content, curious, drowsy, alert
- **Transition inputs:**
  1. Recent interactions in current session (offer accepted → content)
  2. Time of day in user's local timezone (drowsy near dusk, alert early morning)
  3. Ambient events (rain dampens vocal frequency, alarm call → wary)
  4. Bird's personality vector (high boldness → less likely wary on same input)

- **Mood persistence:** Mood at session-end carries to session-start (modulated by tick in between)
- **No snap to default:** Birds don't reset to neutral on tab open

### Call Grammar Runtime
- **Procedural call synthesis:** Client-side via WebAudio (see `bird_engine.md`)
- **Per-bird motif library:** Species-specific motifs, personality-shaped timing and pitch
- **Call timing:** Vocal frequency trait controls how often bird calls when unobserved
- **Chorus mechanics:** Real-time mixing of procedural calls, not stacked loops
- **Call captioning:** Procedural text generated from motif at runtime

### Bird-to-Bird Interaction
- **Calls from one bird can prompt responses from another**
- **Mood contagion:** Wary mood in one bird tends to spread to nearby birds
- **Chorus events:** Two or more birds with high vocal frequency calling in same window
- **Implementation:** Server-side state includes per-bird call state; client renders responses

---

## 6. Sync Model

### Canonical State
- **Server is the only writer** of personality vectors and moods
- **Clients are read-only** — they pull snapshots, never write state
- **No client-side state ownership** — no merging, no last-write-wins

### Multi-Device Sync
- **All clients read from one canonical record**
- User's laptop and phone show the same aviary, same moods, same drift
- No client-to-client sync, no client-side state to merge

### Conflict Prevention
- **Additive server-authored deltas only** — clients send events, server computes drift
- **Event log order preserved** — tick consumes events in order
- **No client mutates personality directly** — under any code path

### Sync Conflict Surface
- **Rare cases only** — magic-link replay, in-flight session timing out mid-write, server outage
- **Matter-of-fact voice only** — naturalist phrasing here reads as evasive
- **Examples:**
  - "We couldn't sign you in. The link may have expired. Try requesting a new link."
  - "Your session timed out. Sign in again to keep watching."
  - "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."

---

## 7. Frontend Rendering Pipeline

### Scene Composition
- **Single horizontal scene** — fits on one screen, no panning/scrolling/zooming
- **Three perch zones** — front, middle, back (signal for user, not user-controlled)
- **Responsive to viewport** — compresses horizontally on narrow phones, widens on desktops
- **Aspect ratio preserved** — all birds always visible, never cropped or offscreen

### Rendering Sequence
1. **First frame:** Birds already in motion (no "wake up" animation)
   - Client pulls state snapshot
   - Places birds at current positions in their current motions
   - Starts rendering as if it's been rendering all along
2. **Idle micro-motion:** Preening, scanning, head-tilt, body-shuffle (mood-shaped)
3. **Ambient motion:** Leaf/feather drift, parallax (client-side, idle cadence)
4. **Day/night cycle:** Local time, gradual palette shifts, mood modulation
5. **Weather:** Rare ambient rain/wind (mood effects, short-lived)

### Animation Strategy
- **Server sends state snapshots** (positions, moods, call timing)
- **Client interpolates between snapshots** for smooth motion (not teleport)
- **Idle micro-motion runs continuously** regardless of user attention
- **Client stops rendering when tab hidden** (battery saving), but simulation continues server-side

### Reduced-Motion Mode
- **Not "animations off"** — it is a different rendering of the same aviary
- **Micro-motion replaced by cross-fades** between still poses
- **Flight transitions become cross-fades** between perches
- **Ambient leaf drift removed** — ambient color shifts remain, slowed
- **Calls still play** at full quality (or caption, per user's audio settings)

### Top-Bar Chrome
- **Icons only:** Account/settings, accessibility settings, field notebook, offer affordance
- **No UI inside the aviary** — birds and place only
- **Fade after cursor stillness** — nearly transparent after a few seconds, returns on activity
- **Quiet loading state** — quiet field (soft sky color, faint motion cues) if snapshot takes a beat

### Accessibility Surfaces
- **Screen-reader narration:** Naturalist prose, low cadence (one update per 30–60 seconds at idle)
- **Keyboard navigation:** Tab through top bar; Enter/Arrow keys for birds; Escape to exit listen-in
- **Focus indicators:** Visible against aviary background (soft, high-contrast outline)
- **Call captioning:** Procedural text near calling bird, fading in/out with call
- **Reduced-motion mode:** Cross-fade rendering, ambient motion reduction

---

## 8. Audio Pipeline

### Procedural Call Synthesis
- **WebAudio only** — no recorded audio (bundle budget, chorus mechanics)
- **Per-bird motif library** — species-specific motifs, personality-shaped timing and pitch
- **Call timing:** Vocal frequency trait controls call frequency when unobserved
- **Call captioning:** Procedural text generated from motif at runtime

### Chorus Mixing
- **Real-time mixing** of procedural calls, not stacked loops
- **Listen-in mix:** Focus one bird, gradual rise in its mix level, gradual drop in others
- **Ambient quieting:** Other birds drop in mix but never go silent
- **No hard cuts** — mix changes are gradual (feels like listening, not switching channels)

### WebAudio Fallback
- **If WebAudio unavailable:** Play in graceful silence with captions on by default
- **No recorded-audio fallback path** — bundle budget collapses at quality needed
- **Silence with captions is better fallback** than canned audio

### Audio Budget
- **No memory growth over 30 minutes** — buffers reused, no per-call allocation that isn't freed
- **Procedural audio synthesized client-side** — WebAudio, not downloaded files

---

## 9. Accessibility Surfaces

### Screen-Reader Narration
- **Naturalist prose only** — not state lists, not ARIA-label automation
- **Running prose example:** "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
- **Cadence:** Slow — roughly one update per 30–60 seconds at idle, faster only on user-initiated events
- **Priority bump for events:** Return-greeting, offer reaction, settle narrated promptly, but as observations, not state transitions
- **Voice continuity:** Same naturalist field-notebook voice as the rest of the product

### Reduced-Motion Mode
- **Not a fallback** — it is a designed surface with its own charm
- **Cross-fade rendering** replaces micro-motion (idling birds cross-fade between preen-poses)
- **Ambient motion reduced** — leaf/feather drift removed, ambient color shifts slowed
- **Calls still play** at full quality (or caption, per user's audio settings)
- **Birds still drift** — mood still changes — the aviary is still the aviary

### Call Captioning
- **Short prose descriptions** — "a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch"
- **Procedural generation** — matches what was actually played, not fixed strings
- **Naturalist voice** — same voice as field notebook
- **Visual placement:** Small text near calling bird, fading in/out with call
- **Opt-in from accessibility settings** — captions off by default

### Keyboard Navigation
- **Tab through top bar items** — account, accessibility, notebook, offer, settle
- **Enter/Arrow keys for birds** — focus first bird with Tab, arrow keys move between birds
- **Enter triggers listen-in** on focused bird
- **Escape exits listen-in**
- **Offer affordance** opens with top-bar shortcut, fully keyboard-navigable
- **Focus indicators visible** against aviary background (soft, high-contrast outline)

### WCAG AA Contrast
- **All user-copy text passes WCAG AA** — top bar labels, settings, account surfaces, error surfaces, captions, narration when displayed visually
- **AViary scene itself has no user copy** except top bar — contrast constraint applies primarily to chrome

---

## 10. Performance Budgets and Observability

### Budgets
- **Initial JS bundle <2MB** (gzipped) — at first paint, before any code-splitting
- **Time to first bird visible <500ms** — on mid-tier mobile over 4G
- **60fps idle motion on 5-year-old laptop** — runtime budget, not just launch
- **No memory growth over 30 minutes** — procedural audio buffers reused, no per-call allocation that isn't freed

### Observability
- **Synthetic performance checks** — fleet of automated browsers running the aviary on a schedule from common geographies
- **Aggregate-only Real User Monitoring** — page load timings, first-bird-render timings, render-frame timings, audio-context errors, simulation-tick latencies
- **No per-bird state in telemetry** — privacy boundary in `accounts_sync.md` honored at metric definition

### Error Budgets
- **Simulation-tick latency p99 alarms if >5 seconds** — tick supposed to take much less; alarm catches degradation early
- **No memory growth over 30 minutes** — real test in CI, not a guideline

### Browser Support
- **Last two major versions** of Chrome, Safari, Firefox, and Edge
- **Older browsers receive matter-of-fact unsupported-browser surface** — explains what's needed
- **Deliberately do not maintain compatibility paths for very old browsers** — cost-benefit doesn't justify bundle bloat

---

## 11. Rollout

### v1 Shipping
- **Single deploy** — all features ship together (bird engine, simulation, rendering, audio, accessibility, sync)
- **No feature flags for v1** — all features are load-bearing; releasing piecemeal would ship an incomplete product
- **Gradual ramp on birds-per-aviary** — not needed for v1 (2 starter birds, max 7, but 7 is the cap from day one)

### Instrumentation from Day One
- **Simulation-tick latency** — p50, p95, p99
- **State-snapshot size and latency** — from CDN edge to client
- **Render-frame timing** — idle motion at 60fps on target hardware
- **Audio-context errors** — WebAudio availability and errors
- **Session duration** — anonymized, no per-account dimension
- **Error rates** — per-endpoint, per-feature
- **Accessibility feature usage** — captions enabled, reduced motion enabled, audio muted (aggregate only)

### Post-v1
- **Bird count cap may increase** if future audio-mix work raises the recognizability ceiling (7 is v1 limit)
- **Visit-invitation feature may be turned on by default** after v1 (currently off by default)
- **No plan for native apps at v1** — may revisit later; not designed for native-client constraints

---

## 12. Risks

### Drift Calibration Risk
- **Risk:** Drift is too fast or too slow, breaking the "feels alive over weeks" promise
- **Mitigation:** Calibration target named explicitly (measurable in instruments after ~1 week, visible to user after ~3 weeks); test harness measures numerical changes; user feedback loop for visible drift
- **Fallback:** If drift is too fast, increase the filter time constant; if too slow, decrease it

### Sync Correctness Risk
- **Risk:** Personality drift corrupted by client-side writes or last-write-wins
- **Mitigation:** Server is the only writer of personality vectors; additive server-authored deltas only; event log order preserved; no client mutates personality directly
- **Fallback:** Audit logs for personality updates; if corruption detected, roll back to last known-good state

### Audio Uncanniness Risk
- **Risk:** Procedural calls sound canned or phase-canceling artifacts from stacked loops
- **Mitigation:** Real procedural call grammar (not 3 variants in rotation); WebAudio chorus mixing (not stacked loops); procedural call captioning (matches what was actually played)
- **Fallback:** If uncanny, increase procedural variation; if phase-canceling, switch to real chorus mixing

### Accessibility Regression Risk
- **Risk:** Accessibility surfaces degrade or break, teaching users that access to the product's quality is rationed by sensory ability
- **Mitigation:** Accessibility surfaces designed as first-class, not a checklist; reduced-motion mode is its own designed surface; screen-reader narration is naturalist prose, not state lists; WCAG AA contrast enforced
- **Fallback:** Audit accessibility surfaces in CI; if regression detected, revert and ship accessibility-first

### Loading State Risk
- **Risk:** First frame shows "loading" or "wake up" animation, breaking the central conceit that the aviary has been continuing without the viewer
- **Mitigation:** Server-side simulation tick (aviary continues on server); client pulls state snapshot and renders as if it's been rendering all along; quiet loading state (soft sky color, faint motion cues) if snapshot takes a beat; no spinner, no fade-from-static
- **Fallback:** If first frame shows loading, reduce snapshot size or improve CDN edge latency

### Presence Detection Risk
- **Risk:** Presence signal too lax (tab open = presence) corrupts drift across entire user base
- **Mitigation:** Strict definition: visibilityState visible + window focus + pointer/key activity in last few minutes (all three); test harness measures presence-time; user feedback loop for drift calibration
- **Fallback:** If presence too lax, narrow the activity window or add more conditions

---

## 13. Implementation Notes

### Key Decisions
- **Server-side simulation tick** — makes "the aviary continues without the viewer" actually true
- **Additive server-authored deltas** — prevents last-write-wins corruption of personality drift
- **Procedural calls via WebAudio** — bundle budget + chorus mechanics + no canned audio
- **Naturalist voice everywhere** — product surface, field notebook, screen-reader narration, call captions
- **Matter-of-fact voice for system surfaces** — sign-in, errors, sync conflicts, accessibility settings
- **No gamification** — no achievements, no streaks, no levels, no scores, no badges (cumulative effect of "just one" is total failure)
- **Drift monotonic toward expressive** — no negative drift on neglect (avoids Tamagotchi model)

### Calibration Points
- **Drift calibration:** Measurable in instruments after ~1 week, visible to user after ~3 weeks
- **Mood reset cadence:** Daily-ish (calibrated during build)
- **Presence activity window:** A few minutes (calibrated during build — leaning toward longer side)
- **Simulation tick cadence:** ~once per minute (calibrated during build)
- **Notebook entry cadence:** ~one every few days for regularly-visited aviary, more often when something noteworthy happens

### Known Ambiguities and Defensible Calls
- **Activity window for presence:** PRD says "a few minutes"; implementation will calibrate during build, leaning toward the longer side (watching birds without moving is the actual product)
- **Mood reset cadence:** PRD says "daily-ish"; implementation will calibrate during build
- **Simulation tick cadence:** PRD says "~once per minute"; implementation will calibrate during build
- **Idle micro-motion animation rates:** Not specified; implementation will design per-bird idle states and animation curves

---

## 14. Out-of-Scope Clarifications

### Not in Plan (per `non_goals.md`)
- **Native mobile apps** — web-only at v1
- **Gamification** — no achievements, no streaks, no levels, no scores, no badges
- **Tamagotchi mechanics** — no death, no hunger, no distress, no happiness decay
- **Social network surfaces** — no profiles, no follows, no public feed, no comments on visits
- **Payments, billing, or premium tiers** — single-tier product
- **Shared aviaries, household accounts, or multi-user accounts** — single-user accounts only
- **Customizable scenes, bird selection, or species catalog** — 6 species in pool, 2 starter birds selected by system
- **Multi-aviary accounts or aviary switching** — one aviary per account
- **Public discovery, leaderboards, or rankings** — private to host and named invitees only
- **Push notifications, emails, or pings about the aviary** — aviary lives where user visits it

### Not in Plan (per `social_optional.md`)
- **Chat during visits** — read-only ambient view only
- **Avatars for visitors** — visitor identity not represented in aviary
- **Comments on aviaries or birds** — notebook is host's, not annotatable
- **Public discovery of aviaries** — no directory of public aviaries
- **Leaderboards** — no ranking of any kind
- **"Show-off" mode for visitors** — visitors see host's aviary exactly as it is
- **Friend-visited notifications** — by default, host gets no push, no email, no in-product notification

---

## 15. Success Criteria

### Functional
- Birds appear already in motion on first frame (no "wake up" animation)
- Personality drift measurable in instruments after ~1 week of regular visits
- Mood transitions visible to user after ~3 weeks of regular visits
- Procedural calls sound unique per bird (not 3 variants in rotation)
- Chorus mechanics work (two birds calling at once produce real chorus, not stacked loops)
- Presence accounting precise (visibility + focus + pointer/key activity, all three)
- Multi-device sync coherent (laptop and phone show same aviary, same moods, same drift)
- Accessibility surfaces work (screen-reader narration, reduced-motion mode, call captions, keyboard nav)
- No gamification anywhere in the product (no achievements, no streaks, no levels, no scores, no badges)

### Affective
- User feels accompanied, not entertained
- User feels noticed, not announced at
- User feels the aviary has been continuing without them
- User feels the birds have personality (not numbers)
- User feels the birds have mood (not states)
- User feels the relationship is observational, not custodial

### Performance
- Initial JS bundle <2MB (gzipped)
- Time to first bird visible <500ms (mid-tier mobile over 4G)
- 60fps idle motion on 5-year-old laptop
- No memory growth over 30 minutes
- Simulation-tick latency p99 <5 seconds

### Accessibility
- WCAG AA contrast on all user-copy text
- Screen-reader narration naturalist prose, slow cadence
- Reduced-motion mode is designed surface, not fallback
- Call captioning procedural, matching what was actually played
- Keyboard navigation complete (top bar, birds, offer, settle)

---

## 16. Appendix

### PRD Files Read
- `product_brief.md` — headline concept, design philosophy, voice and tone, what the product is and isn't
- `concepts.md` — domain vocabulary (presence, mood, personality vector, drift, tick, etc.)
- `bird_engine.md` — personality vector, drift, mood, calls, idle motion, bird species pool
- `interactions.md` — return-greeting, listen-in, offer, settle, field notebook, presence accounting
- `aviary_layout.md` — visual scene, perch zones, day/night cycle, ambient weather, top-bar chrome
- `accounts_sync.md` — auth, server-side simulation tick, multi-device sync, conflict handling, privacy
- `social_optional.md` — visit invitations and what they deliberately are not
- `accessibility_perf.md` — accessibility surfaces and performance budgets
- `non_goals.md` — explicit out-of-scope statements

### Design Philosophy Inheritance
- **Feels alive, not robotic** — procedural calls, mood-shaped idle motion, server-side simulation
- **Notice, never announce** — no "Welcome back!" toast, no level-up confetti, no badge popups
- **Charm comes from specificity** — naturalist prose, specific to bird and moment, no gamification language
- **Restraint over richness** — 2-7 birds, one screen, calm color palette, no UI chrome inside aviary
- **Naturalist voice for product, matter-of-fact for system** — naturalist for aviary, notebook, narration; matter-of-fact for sign-in, errors, sync conflicts, accessibility settings

### Constraints with Teeth
- **Server is the only writer of personality vectors** — no client mutates personality directly
- **Additive server-authored deltas only** — no last-write-wins, no client-submitted absolute values
- **Procedural calls via WebAudio only** — no recorded audio (bundle budget + chorus mechanics)
- **No gamification anywhere** — no achievements, no streaks, no levels, no scores, no badges
- **Drift monotonic toward expressive** — no negative drift on neglect (avoids Tamagotchi model)
- **Presence precise definition** — visibility + focus + pointer/key activity, all three
- **Naturalist voice everywhere** — product surface, field notebook, screen-reader narration, call captions
- **Matter-of-fact voice for system surfaces** — sign-in, errors, sync conflicts, accessibility settings

---

**End of Plan.**
