# Pocket Aviary — V1 Implementation Plan

This plan translates the PRD into an executable engineering roadmap for a frontier team. Defensible calls on underspecified details are noted inline.

---

## 1. Scope

### 1.1 In scope (v1)

| Area | V1 deliverable |
|------|----------------|
| Platform | Modern web browsers only (Chrome, Safari, Firefox, Edge — last two major versions) |
| Accounts | Single-user, email magic-link auth, one aviary per account |
| Birds | Start with 2 starter birds; cap at 7; ~6 species pool; age-gated adoption of additional birds |
| Simulation | Server-side tick (~1/min), personality drift, mood system, bird-to-bird interaction |
| Interactions | Return-greeting, presence accounting, listen-in, offer (seed/song/pool), settle |
| Notebook | Auto-generated naturalist field notebook (read-only, sparse entries) |
| Sync | Multi-device via canonical server state; clients pull snapshots, append events |
| Social | Opt-in visit invitations (read-only ambient view); off by default |
| Accessibility | Screen-reader narration, call captions, reduced-motion mode, keyboard nav, WCAG AA |
| Audio | Client-side procedural WebAudio synthesis; chorus mixing; listen-in mix decay |
| Rendering | Single horizontal scene, day/night (local TZ), ambient weather, no in-scene chrome |

### 1.2 Explicitly out of scope (non-negotiable)

- Native mobile apps
- Gamification of any kind (achievements, streaks, levels, badges, visit calendars, XP)
- Tamagotchi mechanics (death, hunger, distress meters, decay-on-neglect)
- Social network surfaces (profiles, follows, discovery, leaderboards, comments, co-presence)
- Push/email notifications about the aviary (visit notifications opt-in only, off by default)
- Payments, shared aviaries, customizable scenes, multi-aviary accounts
- Recorded-audio fallback path
- Exposing personality vector numerically to users (no debug toggle, ever)
- "Welcome back" toasts, banners, or any announcement-style return UI

### 1.3 Defensible calls on ambiguity

| Topic | Decision | Rationale |
|-------|----------|-----------|
| Presence activity window | 4 minutes | Leans long per PRD; watching without moving is the product |
| Personality trait range | `[0.0, 1.0]` normalized floats | Standard; seed values species-keyed |
| Tick cadence | 60 seconds | PRD "~once per minute" |
| Offer cooldown | 3 minutes per bird | "Few minutes"; prevents drift saturation |
| Notebook entry rate | Target 1 entry / 3–5 days for regular users; burst on noteworthy events | PRD sparsity requirement |
| Third bird unlock | Aviary age ≥ 21 days | "Few months" pacing starts earlier for v1 testing; tune in beta |
| Tech stack | TypeScript monorepo: React 19 + Vite (client), Node/Fastify (API), Postgres (state), Redis (tick queue) | Web-first, team velocity; not prescribed by PRD |

---

## 2. Architecture

### 2.1 Service shape

```
┌─────────────────────────────────────────────────────────────────┐
│                         CDN / Edge                               │
│  static assets (JS/CSS/SVG) + edge-cached initial snapshot stub │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────┐
│                     Web Client (SPA)                             │
│  Render │ Audio │ Presence │ A11y │ Event emitter               │
└────────────────────────────┬────────────────────────────────────┘
                             │ HTTPS (REST + SSE optional)
┌────────────────────────────▼────────────────────────────────────┐
│                     API Gateway / BFF                            │
│  auth │ snapshots │ events │ notebook │ visits │ settings       │
└──────┬─────────────────────┬──────────────────────┬─────────────┘
       │                     │                      │
┌──────▼──────┐    ┌─────────▼─────────┐   ┌───────▼────────┐
│ Auth Service │    │ Simulation Service │   │ Notebook Gen  │
│ magic link   │    │ tick worker        │   │ prose writer  │
└─────────────┘    └─────────┬─────────┘   └────────────────┘
                             │
                    ┌────────▼────────┐
                    │   PostgreSQL     │
                    │ accounts, birds, │
                    │ vectors, moods,  │
                    │ events, notebook │
                    └──────────────────┘
```

### 2.2 Client/server split

| Responsibility | Owner |
|----------------|-------|
| Canonical aviary state (positions, moods, personality vectors, drift history) | Server |
| Personality vector writes | Server tick only |
| Mood transitions between sessions | Server tick |
| Event log (append-only) | Server stores; client appends |
| Rendering, interpolation, idle micro-motion | Client |
| Procedural call synthesis & mix | Client |
| Presence detection (visibility + focus + activity) | Client detects; server records pings |
| Screen-reader narration prose assembly | Client (from snapshot); same voice rules as notebook |
| Visit read-only rendering | Client (visitor mode flag disables interaction emitters) |

**Hard rule:** Clients never tick, never write personality state, never send absolute trait values.

### 2.3 Render pipeline boundary

1. Client requests `GET /aviary/snapshot` on load, visibility resume, keepalive (~30s visible), and after long frame gaps (>5s).
2. Snapshot includes: per-bird `{id, species, name, mood, perchZone, poseKey, callScheduleHint, plumageSaturationVisual}`, scene `{timeOfDay, weather, settled}`, `serverTime`, `tickVersion`.
3. Client `SimulationInterpolator` holds last two snapshots; renders at 60fps between them.
4. Client `MotionDirector` layers procedural idle micro-motion (mood-shaped) on interpolated base pose.
5. Client `AmbientOrnamentRenderer` adds non-canonical leaf/feather drift (not in server state).
6. On tab hidden: stop `requestAnimationFrame`; keep presence ended; server continues ticking.

---

## 3. Data Model

### 3.1 Core entities

#### `accounts`
```sql
id              UUID PRIMARY KEY  -- synthetic; never email
email_encrypted BYTEA
email_hash      TEXT UNIQUE       -- for lookup only
created_at      TIMESTAMPTZ
deletion_scheduled_at TIMESTAMPTZ NULL
settings_json   JSONB             -- a11y prefs, visit notification toggle
```

#### `sessions`
```sql
id, account_id, device_label, created_at, revoked_at
```

#### `aviaries`
```sql
id              UUID PRIMARY KEY
account_id      UUID UNIQUE       -- 1:1 at v1
created_at      TIMESTAMPTZ       -- drives bird-unlock age
settled         BOOLEAN
local_tz        TEXT              -- IANA, from client on first load
```

#### `birds`
```sql
id              UUID PRIMARY KEY  -- stable forever; never regenerated
aviary_id       UUID
species_id      TEXT
display_name    TEXT
adopted_at      TIMESTAMPTZ
personality     JSONB             -- {boldness, socialWarmth, vocalFrequency, plumageSaturation, curiosity}
mood            TEXT              -- enum: wary|content|curious|drowsy|alert
mood_since      TIMESTAMPTZ
perch_zone      TEXT              -- front|middle|back
pose_key        TEXT              -- server hint for interpolation
```

**Invariant:** `personality` updated only by tick worker via additive deltas.

#### `interaction_events` (append-only)
```sql
id              BIGSERIAL
aviary_id       UUID
account_id      UUID
event_type      TEXT              -- presence_ping|listen_in_start|listen_in_end|offer|settle|tab_close
bird_id         UUID NULL
payload         JSONB
client_ts       TIMESTAMPTZ
server_ts       TIMESTAMPTZ DEFAULT now()
processed       BOOLEAN DEFAULT false
```

#### `notebook_entries`
```sql
id, aviary_id, body TEXT, created_at, source_event_ids BIGINT[]
```

#### `visit_invitations`
```sql
id, host_account_id, visitor_email_hash, token_hash, created_at, expires_at, revoked_at
```

#### `visit_log`
```sql
id, invitation_id, started_at, ended_at, duration_approx
```

### 3.2 Personality vector schema

```typescript
interface PersonalityVector {
  boldness: number;        // 0..1
  socialWarmth: number;
  vocalFrequency: number;
  plumageSaturation: number;
  curiosity: number;
}
```

- Seed per species at adoption (e.g., warbler: higher vocalFrequency, lower boldness).
- Never exposed via API to client in numeric form; client receives only visual proxies (plumage saturation affects shader/SVG fill).

### 3.3 Mood enum

`wary | content | curious | drowsy | alert`

Persist across sessions. Server tick applies time-of-day and ambient-event transitions when client offline.

### 3.4 Presence events

Client emits `presence_ping` every 60s while conjunction holds:
- `document.visibilityState === 'visible'`
- `document.hasFocus()`
- `lastPointerOrKey < 4 minutes ago`

Payload: `{ activityWindowSec: 240 }`. No presence recorded for visitors.

---

## 4. API Surface

### 4.1 Auth

| Method | Path | Description |
|--------|------|-------------|
| POST | `/auth/magic-link` | Body: `{email}` → sends link, rate-limited |
| GET | `/auth/verify?token=` | Consumes token, sets session cookie, 15min expiry |
| POST | `/auth/logout` | Revokes session |
| GET | `/auth/sessions` | List devices |
| DELETE | `/auth/sessions/:id` | Revoke device |

### 4.2 Aviary state

| Method | Path | Description |
|--------|------|-------------|
| GET | `/aviary/snapshot` | Current canonical state + `tickVersion` |
| POST | `/aviary/events` | Append interaction event(s); batch up to 10 |
| POST | `/aviary/timezone` | Set/update `local_tz` once |

### 4.3 Birds

| Method | Path | Description |
|--------|------|-------------|
| POST | `/birds/adopt-starters` | Onboarding: system picks 2 species; user supplies names |
| PATCH | `/birds/:id/name` | Rename |
| POST | `/birds/adopt-next` | Age-gated; returns 403 if too young or at cap |

### 4.4 Notebook

| Method | Path | Description |
|--------|------|-------------|
| GET | `/notebook/entries?cursor=` | Paginated, newest first |

### 4.5 Account

| Method | Path | Description |
|--------|------|-------------|
| GET/PATCH | `/account/settings` | A11y, notifications |
| POST | `/account/export` | Async JSON export emailed |
| POST | `/account/delete` | Soft delete; 30-day recovery |

### 4.6 Visits

| Method | Path | Description |
|--------|------|-------------|
| POST | `/visits/invite` | Body: `{email}` |
| GET | `/visits/invitations` | Host list |
| DELETE | `/visits/invitations/:id` | Revoke |
| GET | `/visits/log` | Host visit history |
| GET | `/visit/view?token=` | Visitor snapshot (read-only token) |

Visitor snapshot endpoint returns same shape as host snapshot but sets `mode: 'visit'` and omits notebook write paths.

### 4.7 Error voice

All auth/sync/load errors use matter-of-fact copy per PRD. HTTP 4xx/5xx bodies: `{ code, message, action? }`.

---

## 5. Simulation Engine Design

### 5.1 Tick worker

- **Cadence:** every 60s per active aviary (aviaries with events in last 7 days OR ticked in last 24h; cold aviaries tick every 5 min).
- **Input:** unprocessed `interaction_events` since last tick, ordered by `server_ts`.
- **Output:** updated `birds.personality`, `birds.mood`, `birds.perch_zone`, `birds.pose_key`, optional `notebook_entries`, mark events processed.

### 5.2 Drift function

Low-pass filter over weighted signals per trait:

| Signal | Weight (relative) | Trait targets |
|--------|-------------------|---------------|
| presence_time (sec) | 1.0 (dominant) | all traits ↑ slightly; plumageSaturation, socialWarmth strongest |
| listen_in_duration per bird | 0.6 | that bird's socialWarmth, vocalFrequency |
| offer_accepted | 0.3 | curiosity |
| offer_near_bird | 0.2 | boldness |
| settle | 0.05 | mood quieting only; neutral drift |

**Monotonic rule:** deltas are `max(0, computed_delta)` for all traits. Neglect produces no negative delta; birds become ambient (fewer greetings via mood/perch behavior, not trait punishment).

**Calibration:**
- Weekly regular use (~30 min/day, 5 days): instrument detects ≥0.02 total trait movement per bird.
- ~3 weeks: user-visible behavior change (bolder perch choice, richer plumage, more frequent calls) without UI stating it.

```typescript
function applyDrift(vector: PersonalityVector, signals: DriftSignals): PersonalityVector {
  const rate = 0.00015; // tuned in staging
  return {
    boldness: clamp(vector.boldness + rate * signals.presenceSec * 0.3 + rate * signals.offerNear * 0.5),
    socialWarmth: clamp(vector.socialWarmth + rate * signals.presenceSec * 0.5 + rate * signals.listenInSec * 0.8),
    vocalFrequency: clamp(vector.vocalFrequency + rate * signals.presenceSec * 0.4 + rate * signals.listenInSec * 0.6),
    plumageSaturation: clamp(vector.plumageSaturation + rate * signals.presenceSec * 0.6),
    curiosity: clamp(vector.curiosity + rate * signals.offerAccepted * 0.7),
  };
}
```

### 5.3 Mood transitions

State machine per bird, evaluated each tick:

**Inputs:** recent session events (offer → content), local time-of-day (dusk → drowsy), weather (rain → dampen vocal behavior via mood modifier), neighbor mood (wary spreads softly), personality (high boldness resists wary).

**Persistence:** mood at session end is start mood next session, modulated by offline ticks (overnight → drowsy/settled).

### 5.4 Call-grammar runtime (server side)

Server does not synthesize audio. Server maintains:
- `callScheduleHint`: next call window, motif seed, mood modifier
- Client grammar expands motif library per species

Bird-to-bird: when two birds' call windows overlap and vocalFrequency sufficient → chorus event flag in snapshot.

### 5.5 Return-greeting (server-authored trigger)

On first `presence_ping` after absence:
1. Compute `absenceDuration` since last host presence end.
2. Select greeter: highest `boldness * socialWarmth`, modulated by mood (wary may defer).
3. Emit `greetingIntent` in snapshot: `{ birdId, style: glance|call|approach, staggerMs }`.
4. Client procedural layer executes; never identical twice (seed from tickVersion + birdId + timestamp hash).

No textual welcome anywhere.

### 5.6 Bird-to-bird interaction

- Call response: bird A calls → bird B may respond if socialWarmth > threshold; stagger 200–800ms.
- Mood contagion: if ≥2 birds wary, small nudge to nearby birds.
- Chorus: snapshot flags simultaneous call slots.

### 5.7 Notebook generation (tick subprocess)

Triggered when noteworthy event detected (first greeter of day, long quiet stretch, weather, adoption). Template + NLG:

- Input: structured facts `{pipGreetedFirst, absenceNote, weather, moodSnapshot}`
- Output: lowercase present-tense prose; never numeric traits; never user-behavior stats ("you visited every day").
- Rate limiter: max 1 entry per 48h unless high-salience event.

---

## 6. Sync Model

### 6.1 Canonical state flow

```
Device A ──POST event──► Event Log ──► Tick ──► Canonical DB
Device B ──GET snapshot───────────────────────────────┘
```

No client-to-client sync. No CRDT. No LWW on personality.

### 6.2 Conflict prevention

| Failure mode | Prevention |
|--------------|------------|
| Dual-device personality overwrite | Only tick writes vectors; additive deltas from ordered events |
| Stale client snapshot | `tickVersion` monotonic; client discards older snapshots |
| Duplicate magic-link use | Token single-use |
| Event replay | Idempotency key `clientEventId` UUID per POST |
| Visitor affecting host | Visitor tokens cannot POST to `/aviary/events` |

### 6.3 Offline / suspended laptop

On resume: client pulls fresh snapshot; interpolator snaps smoothly from last rendered state to new canonical state over 500ms (no teleport if version jump small; hard snap if >10 ticks).

### 6.4 Account deletion sync

Soft delete blocks snapshot; 30-day recovery restores. Hard delete cascades all bird data.

---

## 7. Frontend Rendering Pipeline

### 7.1 Scene composition (layers, back to front)

1. Sky gradient (time-of-day keyed to user local TZ)
2. Background foliage (parallax 0.2x)
3. Back perch zone
4. Middle perch zone
5. Birds (SVG/canvas hybrid per species)
6. Front perch zone + offer props (pool, seed)
7. Foreground branch/leaf ornaments
8. Weather overlay (light rain particles)
9. Top bar chrome (outside scene hit area)
10. Captions overlay (a11y)

### 7.2 Bird rendering

- Species: compact SVG rig with bone-less transform tree (head, body, wing, tail).
- `plumageSaturation` modulates fill saturation in shader/SVG filter.
- Mood shapes: perch zone selection, scan frequency, preen vs fluff poses.

### 7.3 Idle micro-motion

`MotionDirector` per bird:
- Perlin-offset head scan (wary: faster; drowsy: slow)
- Preen cycle (content)
- Weight shift shuffle (all)
- Runs continuously at 60fps; mood-keyed parameters from snapshot

### 7.4 Transitions

- Perch change: 1.2s ease path (flight) or 0.8s hop (short distance)
- Settle: 4s lighting lerp to evening palette; call volume duck via audio bus
- Settle undo: click within 5s reverses lighting lerp

### 7.5 First paint / no entry animation

1. Inline critical CSS + skeleton sky in HTML shell.
2. Parallel: fetch snapshot + hydrate JS chunk.
3. If snapshot delayed: render quiet field (sky + faint leaf) — **no spinner**.
4. First bird visible ≤500ms: render first bird from partial snapshot or optimistic default pose from edge-cached stub.

### 7.6 Reduced-motion mode

When `prefers-reduced-motion` or user setting:
- Replace frame animation with 3–5s cross-fade between still poses
- Remove ambient leaf drift
- Keep day/night color shifts (slowed 2x)
- Audio, drift, notebook unchanged

### 7.7 Top bar

Icons: account, a11y, notebook, offer. Fade to 10% opacity after 3s pointer idle; restore on move/key.

### 7.8 Visitor mode

Same renderer; `interactionController` disabled; no event POSTs; banner none (matter-of-fact only on revoke/expiry).

---

## 8. Audio Pipeline

### 8.1 Architecture

```
CallScheduler (per bird, from snapshot hints)
    ↓
MotifLibrary[species] → PhraseBuilder (pitch, timing, mood)
    ↓
BirdVoiceNode (WebAudio AudioBufferSource or Oscillator+envelope)
    ↓
ChorusBus → MasterBus → destination
    ↑
ListenInMixer (focus bird +3dB, others -6dB, 1.5s ramp)
```

### 8.2 Procedural synthesis

- Each species: 4–8 motifs (frequency contours, duration ranges).
- Runtime variation: ±5% pitch, ±10% timing, mood modifies attack/decay.
- No looped samples. Buffer pool reused (no per-call alloc — perf budget).

### 8.3 Chorus mixing

Real-time mix of 2+ voices with slight stereo offset; avoid phase-cancel artifact of stacked identical loops by per-instance detune.

### 8.4 Listen-in

- Engage: 1.5s linear ramp focused bird 0→+3dB, others 0→-6dB (never silent).
- Disengage: same ramp reverse on unfocus/blur/empty click/Escape.

### 8.5 WebAudio fallback

If `AudioContext` fails: silence + force captions on; matter-of-fact toast once: "Audio isn't available in this browser. Call captions are on."

No recorded fallback.

### 8.6 Caption generation

At call play time, grammar emits prose caption from motif metadata: "a soft three-note rise". Shown near bird, 2s fade.

---

## 9. Accessibility Surfaces

### 9.1 Screen-reader narration

- `aria-live="polite"` region off-screen.
- `NarrationComposer` builds naturalist prose from snapshot every 30–60s idle; faster on greeting/offer/settle.
- Never state-list ("mood: content"). Prose example: "a small grey bird is perched on the front rail, calling softly."
- Priority queue for user-initiated events.

### 9.2 Captions

Opt-in setting; runtime from call grammar (§8.6).

### 9.3 Keyboard navigation

| Key | Action |
|-----|--------|
| Tab | Top bar → first bird |
| Arrow keys | Move bird focus |
| Enter | Toggle listen-in on focused bird |
| Escape | Exit listen-in |
| Shortcut (documented in a11y settings) | Open offer palette |

Focus ring: 2px high-contrast outline, works on dawn/dusk backgrounds.

### 9.4 Contrast

All chrome/copy WCAG AA minimum; design tokens enforced in CI (`axe-core` on settings surfaces).

### 9.5 Testing

- Automated: `jest-axe`, keyboard-only e2e paths.
- Manual: VoiceOver, NVDA, reduced-motion visual review each release.

---

## 10. Performance Budgets and Observability

### 10.1 Budgets

| Metric | Budget |
|--------|--------|
| Initial JS (gzip) | < 2 MB |
| Time to first bird | < 500 ms (4G, mid-tier mobile) |
| Idle render | 60 fps on 5-year-old laptop |
| Memory @ 30 min | Flat (±5 MB noise) |
| Snapshot payload | < 8 KB typical |
| Tick compute | p50 < 200ms, p99 < 2s (alarm at 5s) |

### 10.2 Client optimizations

- Code-split: settings, visits, onboarding.
- Procedural SVG birds; no sprite atlases > 200KB.
- Audio buffer pool (max 16 buffers recycled).
- `requestAnimationFrame` delta cap; pause when hidden.
- Virtualized notebook list (window ±20 entries).

### 10.3 Observability (aggregate only)

**Collect:** RUM first-bird timing, frame time histograms, audio context errors, API latency, tick duration, error rates.

**Never collect:** per-bird state, per-account interaction content, personality values.

**Synthetic checks:** scheduled Playwright from 3 regions; alert on first-bird > 800ms or tick p99 > 5s.

**Deliberately not measured:** engagement scores, streak proxies, cross-account bird behavior aggregates.

---

## 11. Rollout

### 11.1 Phases

| Phase | Milestone |
|-------|-----------|
| P0 | Auth, snapshot, 2 birds render, server tick, presence |
| P1 | Drift + mood + return-greeting + notebook |
| P2 | Listen-in, offer, settle, day/night, audio |
| P3 | Multi-device, export/delete, a11y complete |
| P4 | Visits, weather, age-gated adoption to 7 |
| Beta | 50 accounts; drift calibration soak 3 weeks |

### 11.2 Birds-per-aviary ramp

- Launch: 2 starters only.
- Enable `adopt-next` API at week 3 post-launch for beta; week 6 for GA.
- Monitor audio mix quality at 5–7 birds in beta before raising cap visibility.

### 11.3 Day-one instrumentation

- First-bird RUM, tick latency, audio init failures, snapshot errors, magic-link funnel (counts only).
- Drift instrumentation in staging: per-tick delta magnitudes (staging only, not prod per-account).

### 11.4 Feature flags

- `visits_enabled` (default off)
- `weather_enabled`
- `max_birds` (2→7 ramp)

---

## 12. Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| Drift too fast/slow | Breaks "weeks not sessions" promise | Staging soak with simulated presence; instrument deltas; weekly tuning |
| Presence definition too loose/tight | Corrupts drift signal | Strict 3-way AND; integration tests; 4-min window A/B in beta |
| Sync / tick ordering bugs | Silent personality loss | Event log ordering; only tick writes vectors; property tests on monotonic drift |
| Audio uncanniness | Breaks aliveness | Procedural variation QA; user ear tests; caption parity |
| WebAudio failures on mobile | Silent aviary | Captions default on; clear matter-of-fact notice |
| "Welcome back" creep | Violates core principle | PR checklist; design review gate; lint copy for banned phrases |
| Gamification creep | Product identity collapse | Explicit non-goals in PR template; reject notebook entries about user habits |
| Accessibility retrofit | Reduced-motion users get broken product | Ship a11y in P3 gate, not post-launch |
| Visit co-presence scope creep | Engine rewrite pressure | Read-only token architecture; no host presence merge |
| Bundle bloat | Miss 500ms first bird | CI bundle budget fail; no recorded audio temptation |
| Notebook too chatty | Dilutes charm | Hard rate limits; salience scorer |
| Personality vector exposure | Relationship becomes optimization | API schema strips numbers; no debug endpoint in prod |

---

## 13. Implementation Workstreams (suggested team parallelization)

1. **Platform & Auth** — magic link, sessions, synthetic UUID, account lifecycle
2. **Simulation** — tick worker, drift, mood FSM, event processor
3. **Client Render** — scene, birds, interpolation, day/night, weather
4. **Audio** — grammar, synthesis, listen-in, captions
5. **Interactions** — presence, greeting, offer, settle controllers
6. **Notebook & Voice** — NLG templates, naturalist lint
7. **A11y** — narration, reduced-motion, keyboard
8. **Visits** — invite flow, read-only mode, revocation
9. **Infra** — CDN, Postgres, tick queue, observability, CI perf gates

---

## 14. Acceptance Criteria (v1 ship gate)

- [ ] Two starter birds adopted without catalog picker; names user-chosen
- [ ] First frame shows mid-motion aviary; no spinner entry
- [ ] Return-greeting within 2s; procedurally varied; no welcome toast
- [ ] Presence requires visibility + focus + recent activity; drives drift in staging harness
- [ ] 3-week beta shows user-noticeable behavioral change without stat UI
- [ ] Listen-in ramps 1.5s; others never silent
- [ ] Settle optional; tab-close equivalent; 5s undo works
- [ ] Notebook sparse, naturalist, read-only; no gamification language
- [ ] Multi-device: same mood/drift on laptop + phone
- [ ] Visitor read-only; no host drift from visitor; revoke works
- [ ] Reduced-motion is cross-fade aesthetic, not static broken scene
- [ ] Bundle < 2MB gzip; first bird < 500ms on reference device
- [ ] No personality numbers anywhere in client network payloads or UI

---

*End of plan. Product implementation is explicitly out of scope for this deliverable.*
