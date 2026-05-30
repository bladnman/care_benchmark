# Pocket Aviary — Implementation Plan v1

## 1. Scope

### 1.1 In Scope (v1)

- **Two starter birds** per new aviary; cap of seven birds total. Species drawn from a pool of ~6 coherent species.
- **Single-user accounts** via email magic link, one canonical aviary per account.
- **Multi-device sync** via server-side canonical state; clients render snapshots only.
- **Procedural bird engine**: personality vectors (boldness, social warmth, vocal frequency, plumage saturation, curiosity), mood state machine (wary, content, curious, drowsy, alert), monotonic drift function.
- **Procedural call synthesis** via WebAudio motif library; chorus mixing; listen-in mix rebalance.
- **Interactions**: return-greeting, listen-in, offer (seed, song fragment, still pool), settle gesture, field notebook.
- **Visual scene**: single horizontal scene, three perch zones (front/middle/back), day/night cycle anchored to user's local time, ambient weather (rare rain, occasional wind), ambient micro-motion (leaves, feathers).
- **Top bar** with account/settings, accessibility settings, field notebook, offer affordance — fades nearly transparent after cursor stillness.
- **Visit invitations**: host emails invitee a one-time link; visitor sees read-only ambient view; visits off by default.
- **Accessibility**: screen-reader naturalist narration, reduced-motion designed surface (cross-fades not cut animations), call captions in naturalist voice, WCAG AA contrast, full keyboard navigation.
- **Performance**: <2MB initial JS bundle, <500ms time-to-first-bird, 60fps idle on 5-year-old laptop, no memory growth over 30 min.

### 1.2 Out of Scope (v1)

- Native mobile apps.
- Gamification: no streaks, achievements, levels, scores, badges, green-dot calendars.
- Tamagotchi mechanics: no hunger/decay meters, no visible distress, no death.
- Social network surfaces: no profiles, follows, public feeds, discovery, leaderboards.
- Shared aviaries, customizable scenes, multi-aviary accounts, push notifications, payments.

### 1.3 Non-Goals Honored

- Personality vectors **never exposed numerically** to users.
- Drift is **monotonic toward expressive only** — no negative drift on neglect.
- Presence is defined as the conjunction of three signals: `visibilityState === 'visible'` AND window focus AND pointer/key activity within a calibrated window. "Tab open" alone does not count.
- No streak counter, no visit-frequency surface, no "days visited" decoration.
- Settle and tab-close are equivalent at the engine level — neither is penalized.

---

## 2. Architecture

### 2.1 Service Shape

```
┌─────────────────────────────────────────────────────┐
│  Browser Client (SPA)                              │
│  ├── Rendering Engine (Canvas/SVG)                 │
│  ├── Audio Engine (WebAudio, procedural synthesis) │
│  ├── State Interpolation Layer                     │
│  └── Interaction Event Writer                      │
└──────────────────────┬──────────────────────────────┘
                       │ HTTPS JSON
┌──────────────────────▼──────────────────────────────┐
│  API Service (Node.js / Express or Fastify)         │
│  ├── Auth (magic link session tokens)              │
│  ├── State Snapshot API (GET /aviary/:id/snapshot) │
│  ├── Event Log API (POST /aviary/:id/events)       │
│  ├── Account API (settings, export, deletion)       │
│  └── Visit API (invite, revoke, visit log)          │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│  Simulation Service (server-side tick)              │
│  ├── Tick Loop (~1/min, calibrated)                 │
│  ├── Personality Vector Updater (low-pass filter)  │
│  ├── Mood Transition Engine                        │
│  ├── Drift Function (presence + interactions)       │
│  └── Canonical State Writer                        │
└──────────────────────┬──────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────┐
│  Database (PostgreSQL or equivalent)                │
│  ├── accounts (synthetic UUID, encrypted email)    │
│  ├── birds (id, aviary_id, species, name, ...)     │
│  ├── personality_vectors (bird_id, boldness, ...)  │
│  ├── moods (bird_id, state, updated_at)            │
│  ├── event_log (account_id, event_type, payload, ts)│
│  ├── notebook_entries (account_id, prose, ts)      │
│  ├── sessions (account_id, device_token, ...)       │
│  └── visit_invitations (host_id, visitor_email, ...)│
└─────────────────────────────────────────────────────┘
```

### 2.2 Client/Server Split

- **Server is the only writer of personality state.** Clients never mutate personality directly.
- **Clients own rendering and audio synthesis only.** They pull snapshots, interpolate between them, synthesize calls in real-time.
- **Clients write interaction events to an append-only event log.** The simulation tick consumes this log in order and computes personality deltas.
- **No client-side simulation tick.** If a client ticks locally, multi-device sync becomes divergent simulations, and the product's central conceit (aviary continues without viewer) fails.

### 2.3 Tick Cadence

- ~1 tick per minute, calibrated during build. Tick runs whether or not any client is connected.
- On tick: read event log since last tick, update personality vectors (low-pass filter), transition moods, advance mood timers, write canonical state.
- Clients pull snapshots on: tab visibility change, long render-gap (suspend/resume), low-frequency keepalive while tab is visible.

---

## 3. Data Model

### 3.1 Birds

```
Bird {
  id: UUID (stable, never recycled)
  aviary_id: FK -> Account
  species: string (from species pool)
  name: string (user-assigned, renameable)
  created_at: timestamp
}
```

### 3.2 Personality Vector (per bird)

```
PersonalityVector {
  bird_id: FK -> Bird
  boldness: float  (normalized, server-authoritative)
  social_warmth: float
  vocal_frequency: float
  plumage_saturation: float
  curiosity: float
  updated_at: timestamp
}
```

- Values are scalar, normalized to a small range (implementation detail).
- Seed values set at adoption; updated only by simulation tick.
- **Never sent to client in raw form.** Client receives only derived display signals (e.g., perch position choice, call frequency).

### 3.3 Mood (per bird)

```
Mood {
  bird_id: FK -> Bird
  state: enum (wary, content, curious, drowsy, alert — exact set finalized in impl)
  updated_at: timestamp
  timer_reset_at: timestamp (for daily-ish cadence)
}
```

- Mood persists across sessions. No reset on tab open.
- Transitions shaped by: recent interactions, time-of-day (local TZ), ambient events, personality vector.

### 3.4 Presence Events

```
PresenceEvent {
  account_id: FK -> Account
  started_at: timestamp
  ended_at: timestamp
  duration_seconds: float
}
```

- A presence-event is recorded only when all three conditions hold simultaneously:
  1. `document.visibilityState === 'visible'`
  2. Document has window focus
  3. Pointer-move or keypress occurred within the calibrated activity window (few minutes)
- Presence-time is the dominant drift input.

### 3.5 Interaction Events (append-only log)

```
EventLog {
  id: UUID
  account_id: FK -> Account
  event_type: enum (presence, listen_in, offer, settle, ...)
  payload: JSON (event-specific data)
  occurred_at: timestamp
}
```

- Clients write events; server tick reads and consumes.
- Events are immutable once written.

### 3.6 Field Notebook Entries

```
NotebookEntry {
  id: UUID
  account_id: FK -> Account
  prose: string (naturalist voice, lowercase, present-tense)
  created_at: timestamp
}
```

- Auto-generated by the simulation service or a dedicated narration service.
- Entries are rare (~one every few days for active aviaries), not per-session.
- Read-only to user.

### 3.7 Accounts

```
Account {
  id: UUID (synthetic, never email)
  email_encrypted: bytes (stored once, encrypted)
  created_at: timestamp
  deleted_at: timestamp (soft delete, 30-day window)
}
```

---

## 4. API Surface

### 4.1 State Snapshot

```
GET /api/aviary/:account_id/snapshot
Authorization: Bearer <session_token>

Response 200:
{
  "birds": [
    {
      "id": "...",
      "name": "Pip",
      "species": "small-grey",
      "perch_zone": "front",
      "mood": "content",
      "call_timing": { "next_call_in_ms": 3400, "avg_interval_ms": 12000 },
      "idle_animation": "preening"
    }
  ],
  "time_of_day": "morning",
  "weather": null,
  "settled": false,
  "server_time": "2026-03-18T07:30:00Z"
}
```

- Snapshot is small (kilobytes). CDN-edge cacheable with short TTL.
- Client interpolates between snapshots for smooth per-bird motion.

### 4.2 Interaction Events

```
POST /api/aviary/:account_id/events
Authorization: Bearer <session_token>
Content-Type: application/json

Body:
{
  "events": [
    {
      "type": "presence",
      "payload": { "duration_seconds": 240 },
      "occurred_at": "2026-03-18T07:30:00Z"
    },
    {
      "type": "listen_in",
      "payload": { "bird_id": "...", "duration_seconds": 180 },
      "occurred_at": "2026-03-18T07:32:00Z"
    },
    {
      "type": "offer",
      "payload": { "offer_type": "seed", "bird_id": "..." },
      "occurred_at": "2026-03-18T07:35:00Z"
    }
  ]
}

Response 200:
{ "received": 3 }
```

- Batched event submission; client buffers and flushes on visibility change / session end.
- Events are append-only; server tick consumes in order.

### 4.3 Visit Invitation Flow

```
POST /api/aviary/:account_id/invitations
Body: { "visitor_email": "friend@example.com" }

→ System emails one-time link to friend@example.com
→ Link is valid for 30 days
→ Visitor clicks link → receives read-only snapshot stream of host's aviary
→ Host can revoke from account settings
```

```
GET /api/visit/:invitation_token
→ Returns read-only snapshot stream (no interaction event submission)
```

### 4.4 Account Endpoints

```
POST /auth/magic-link         { "email": "..." }
GET  /auth/verify?token=...   → Sets session cookie, returns account
POST /auth/logout
GET  /api/account/export       → JSON snapshot of aviary state
POST /api/account/delete      → Soft delete, 30-day window
GET  /api/account/sessions    → List active sessions (for revocation)
DELETE /api/account/sessions/:session_id
```

---

## 5. Simulation Engine Design

### 5.1 Server-Side Tick

The tick is the heartbeat of the product's "feels alive" claim.

**Tick loop (once per minute):**
1. Read all events since last tick from event_log.
2. Compute presence-time from presence events.
3. For each bird, apply drift function:
   - **Presence-time** → dominant input; birds drift toward expressive.
   - **Listen-in** → strong signal for the listened-to bird (social warmth, vocal frequency).
   - **Offers** → small drift toward curiosity (offered item) and boldness (offering near a bird).
   - **Settle** → small mood-quieting signal; no directional drift.
4. Apply mood transitions:
   - Time-of-day modulation (drowsy near dusk, alert in early morning).
   - Ambient events (rain dampens vocal frequency; alarm call shifts nearby birds toward wary).
   - Recent interaction history.
5. Advance mood timers.
6. Generate notebook entry if warranted (sparsity: ~one per few days).
7. Write new canonical state to database.

### 5.2 Drift Function

Drift is a **low-pass filter** over presence-and-interaction signals.

- Calibration target: measurable drift in instruments after ~1 week of regular visits; visible drift to user after ~3 weeks.
- No single session shifts a trait visibly.
- Traits move **up** on positive presence; **never move down** on neglect.
- Neglect produces ambient quietness (birds call less often because less often is what's been observed), not negative trait movement.

**Drift inputs (rough weight order):**
1. Presence-time (dominant)
2. Listen-in (strong per-bird signal)
3. Offers (small, curiosity + boldness)
4. Settle (mood-quieting only)

### 5.3 Mood State Machine

Per-bird enumerated state: `wary | content | curious | drowsy | alert` (exact set finalized in impl).

Transitions triggered by:
- Recent interactions in current session (offer accepted → content nudge)
- Time of day in user's local timezone (drowsy near dusk; alert in early morning)
- Ambient events (rain → dampened vocal frequency; alarm call → wary spread)
- Bird's own personality vector (high-boldness bird less likely to enter wary)

Mood persists across sessions. At session start, bird resumes whatever mood it had at last tick.

### 5.4 Call Grammar Runtime

Each species has a **motif library**: small sets of melodic fragments combined and varied at runtime.

- Personality shapes timing and pitch (vocal_frequency trait → call interval and join-chorus likelihood).
- Two birds calling simultaneously produce a real chorus, not stacked loops.
- Client-side WebAudio synthesis from motif library.
- Call signature must remain individually recognizable up to 7 birds (the cap).

---

## 6. Sync Model

### 6.1 Single Canonical Aviary

- **Server is the only source of truth.** One canonical aviary state per account.
- Clients pull snapshots; they do not sync state between themselves.
- No client-to-client sync; no eventual consistency to reconcile.
- Both devices pulling the same record = same aviary on both devices.

### 6.2 No Last-Write-Wins for Personality

- Personality drift is **additive, server-authored delta** — never client-submitted absolute values.
- Tick computes delta from event log and applies to existing vector.
- Client never sends "set boldness to 0.62"; client sends "user listened in to Pip for 3 min."
- This makes divergent simulations between devices impossible.

### 6.3 Conflict Prevention

- Server tick is single-writer per account (serialized).
- Event log is append-only, consumed in order.
- No concurrent writers to personality state.

---

## 7. Frontend Rendering Pipeline

### 7.1 Scene Composition

- Single HTML page; aviary fills the viewport.
- Rendering: hybrid Canvas (birds, perches, ambient motion) + SVG/CSS (top bar, overlays).
- Three perch zones rendered as horizontal rails; birds positioned by perch zone.
- Background/foreground foliage as layered CSS or SVG.

### 7.2 Idle Micro-Motion

- Preening, scanning, head-tilting, body-shuffle — continuous, mood-shaped.
- Never pauses when tab is hidden from the user's perspective (client may stop rendering, but simulation continues server-side).
- A wary bird perches further back and scans more; a content bird preens; a curious bird tilts toward sounds; a drowsy bird sits low with feathers fluffed.

### 7.3 State Interpolation

- Bird at perch A in snapshot N and perch B in snapshot N+1 → rendered moving smoothly between them (not teleporting).
- Interpolation runs client-side between snapshot pulls.

### 7.4 Reduced-Motion Mode

- Triggered by `prefers-reduced-motion` or user toggle in accessibility settings.
- **Not "animations off."** Micro-motion replaced by slow cross-fades between still poses.
- Flight transitions → cross-fades between perches.
- Ambient leaf drift removed; day/evening color shifts remain, slowed.
- Calls still play at full quality (or caption per user settings).

### 7.5 Loading State

- If snapshot takes a beat to load: quiet field (soft sky color, faint motion cues) — not a spinner.
- First frame shows birds mid-action; no wake-up animation, no fade-from-static.
- Aviary appears with motion already in progress.

### 7.6 Top Bar

- Thin bar above scene: account/settings icon, accessibility settings icon, field notebook icon, offer affordance icon.
- Fades nearly transparent after a few seconds of cursor stillness.
- Returns to full opacity on cursor movement or keyboard activity.

### 7.7 Responsive Scene

- Aspect ratio preserved; all birds visible at all times.
- On narrow viewport: scene compresses horizontally without cropping birds.
- On wide viewport: scene widens with more space between perches.

---

## 8. Audio Pipeline

### 8.1 Procedural Call Synthesis

- WebAudio API: oscillator networks, noise sources, envelope shapers.
- Each species has a motif library of melodic fragments.
- Personality shapes timing and pitch (vocal_frequency trait).
- Calls are never looped audio; they are synthesized fresh each time.

### 8.2 Chorus Mixing

- Multiple birds calling simultaneously → real-time mix, not stacked loops.
- WebAudio GainNode per bird for individual mix control.
- Phase-canceling artifacts from stacked loops are avoided by construction.

### 8.3 Listen-In Mix

- On listen-in engage: focused bird's gain rises gradually; others drop to ambient level gradually.
- On disengage: mix returns to ambient with same gradual ramp.
- Other birds never go fully silent — re-balance, not mute.

### 8.4 WebAudio Fallback

- If WebAudio is unavailable (older browser, permission denied, hardware issue): graceful silence with captions on by default.
- No recorded-audio fallback path.

### 8.5 Call Captions

- Short naturalist-prose descriptions generated from procedural call grammar at runtime.
- Appear near the calling bird, fade in/out with the call.
- Captions use the same voice as the field notebook.

---

## 9. Accessibility Surfaces

### 9.1 Screen-Reader Narration

- Running naturalist prose narration of aviary state, updated on slow cadence (~30–60s at idle).
- Generated server-side or client-side from the same state as the visual surface.
- Voice: lowercase, present-tense, specific. Example: "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."
- User-initiated events (return-greeting, offer reaction, settle) get small priority bump but are still written as observations, not state transitions.
- High-frequency narration would overwhelm the screen reader queue — cadence matches the slow rhythm of the visual aviary.

### 9.2 Reduced-Motion Mode

- See Section 7.4. Its own designed surface, not a fallback.

### 9.3 Call Captions

- See Section 8.5.

### 9.4 Keyboard Navigation

- Tab → through top bar items.
- Enter aviary scene → focuses first bird.
- Arrow keys → move focus between birds.
- Enter → trigger listen-in on focused bird.
- Escape → exit listen-in.
- Offer affordance opens with top-bar shortcut; fully keyboard-navigable.
- Settle gesture reachable from top bar.

### 9.5 Focus Indicators

- Visible, soft, high-contrast outline against both bright and dim aviary states.
- Specified by visual designer.

### 9.6 WCAG AA Contrast

- All user-copy text passes WCAG AA contrast minimum.
- Applies to: top bar labels, settings, account surfaces, error surfaces, captions, narration when displayed visually.

---

## 10. Performance Budgets and Observability

### 10.1 Budgets

| Metric | Budget |
|--------|--------|
| Initial JS bundle (gzipped) | <2MB |
| Time to first bird visible | <500ms (mid-tier mobile, 4G) |
| Idle motion framerate | 60fps on 5-year-old mid-range laptop |
| Memory growth | None over 30-minute session |
| Simulation tick latency p99 | <5s alarm threshold |

### 10.2 What We Measure

- Synthetic performance checks: automated browsers running aviary on a schedule from common geographies.
- Aggregate-only Real User Monitoring: page load timings, first-bird-render timings, render-frame timings, audio-context errors, simulation-tick latencies.
- **None of this includes per-bird state or per-account interaction history.** Privacy boundary honored at metric definition level.

### 10.3 What We Deliberately Don't Measure

- Per-bird, per-account interaction state is never in telemetry.
- Telemetry pipeline never touches the per-account simulation database.
- Simulation database never read by analytics warehouse.

### 10.4 Browser Support

- Last two major versions of Chrome, Safari, Firefox, Edge.
- Older browsers → matter-of-fact unsupported-browser surface.

---

## 11. Rollout

### 11.1 How We Ship v1

- Soft launch to small cohort; instrument synthetic perf + aggregate RUM from day one.
-gradual ramp based on operational health metrics, not engagement metrics.
- No engagement dashboards; no DAU/MAU targets.

### 11.2 How We Ramp Birds Per Aviary

- New birds become available based on **aviary age** (not visit count, not interaction score, not paid tier).
- Species offer appears at intervals tied to how long the aviary has existed.
- Pacing: few months old → third bird offered; year-old aviary → five or six birds.
- The mechanic refuses to teach the user that more attention earns more birds.

### 11.3 What We Instrument From Day One

- Simulation tick latency p99.
- Time-to-first-bird on real devices.
- Audio-context error rate.
- Bundle size at first paint.
- Error rates on snapshot and event APIs.
- **Not instrumented**: per-bird drift, per-account visit frequency, any per-bird interaction metric.

---

## 12. Risks

### 12.1 Drift Calibration

- **Risk**: Drift function too fast → Tamagotchi model (user can move a number by clicking). Drift function too slow → screensaver (nothing the user does seems to matter).
- **Mitigation**: Calibration target is named and testable: ~1 week for measurable drift in instruments, ~3 weeks for visible drift to user. Tune during build; instrument against the named target.

### 12.2 Sync Correctness

- **Risk**: If a client ever writes personality state directly, divergent simulations become possible across devices.
- **Mitigation**: Server-side tick only; additive server-authored deltas only; no last-write-wins. Enforce at the architecture layer, not policy.

### 12.3 Audio Uncanniness

- **Risk**: Loop-based or low-variation audio is immediately identifiable as dead software; the chorus mechanic fails if phase-canceling artifacts are audible.
- **Mitigation**: Procedural synthesis only; motif library with sufficient variation; WebAudio chorus mixing without stacked loops.

### 12.4 Accessibility Regressions

- **Risk**: Accessibility surfaces are a designed experience, not a checklist. A retrofitted reduced-motion mode or generic screen-reader narration would tell users the product wasn't designed for them.
- **Mitigation**: Accessibility work ships with the rest of the product, not after. Narration is naturalist prose; reduced-motion is its own visual register. Both must be planned in from day one.

### 12.5 Presence Signal Inflation

- **Risk**: A laxer presence definition ("tab is open") silently inflates drift signal across the user base. Users who left their laptop open all night would drift as much as users who watched for two hours.
- **Mitigation**: Presence is the conjunction of three independently-checkable signals. Test the conjunction rigorously.

### 12.6 Field Notebook Voice Degradation

- **Risk**: Stock event-log entries ("session started at 7:43") would break the naturalist voice across the entire product.
- **Mitigation**: Notebook entry generation is a named feature with its own prose-quality bar. Entries are rare (~one every few days). Generator must pass a voice review.

### 12.7 Identity Continuity Failure

- **Risk**: If a bird could ever be "reset" or "regenerated" (e.g., during a migration), the entire premise of weeks-long drift evaporates retroactively.
- **Mitigation**: Stable internal bird ID; server-side persistence only; no client-side reconstruction. The no-last-write-wins rule and server-side tick together make identity continuity an architectural property.

---

## 13. Implementation Phases (Suggested)

### Phase 1 — Foundation
- Auth (magic link), database schema, API skeleton.
- Server-side tick loop with mood transitions.
- State snapshot API.
- Client rendering scaffold (empty aviary → quiet field → first bird fly-in).

### Phase 2 — Bird Engine
- Personality vector storage and drift function.
- Procedural call synthesis (WebAudio motif library).
- Idle micro-motion (mood-shaped).
- Three perch zones and perch-selection logic.

### Phase 3 — Interactions
- Return-greeting (procedurally varied, absence-length signal).
- Listen-in (mix rebalance with gradual ramp).
- Offer (seed, song fragment, still pool) with per-bird cooldown.
- Settle gesture with undo window.
- Presence accounting (conjunction of three signals).

### Phase 4 — Notebook and Social
- Field notebook auto-generation (naturalist prose, sparse).
- Visit invitation flow (opt-in, read-only).
- Visit log for host.

### Phase 5 — Accessibility
- Screen-reader narration (naturalist prose, slow cadence).
- Reduced-motion designed surface.
- Call captions.
- Full keyboard navigation.

### Phase 6 — Polish and Instrument
- Day/night cycle (local time anchored).
- Ambient weather (rare rain, occasional wind).
- Ambient leaf/feather drift.
- Performance observability (synthetic checks, aggregate RUM).
- Bundle optimization (<2MB, <500ms first bird).

---

*Plan produced by phase-1 planner, run 001, wave_001. Model: minimax-m2.7. This plan is the substantive deliverable; no implementation occurs in this phase.*
