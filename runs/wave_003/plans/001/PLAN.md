# Pocket Aviary — v1 Implementation Plan

## 1. Scope

### In scope (v1)

- Browser-only SPA: modern Chrome, Safari, Firefox, Edge (last two major versions).
- Single account → single canonical aviary; magic-link email auth; multi-device sync via server canonical state.
- Two starter birds at account creation; adopt up to seven birds, paced by aviary age (not engagement metrics).
- Server-side simulation tick (~1/min): personality drift, mood transitions, ambient weather, call-timing state.
- Client: single-screen horizontal scene, top-bar chrome, listen-in, offer (seed / song fragment / still pool), settle, field notebook, presence accounting.
- Optional quiet visits (invite-by-email, read-only, off by default).
- Accessibility as first-class surfaces: SR narration, reduced-motion design, call captions, keyboard nav, WCAG AA on chrome/copy.
- Aggregate operational telemetry only; no per-bird analytics warehouse.

### Out of scope (non-goals honored)

- Native apps; passwords/SSO (magic-link only); payments; multi-aviary / shared aviaries.
- Gamification: streaks, scores, badges, levels, calendars of visits, XP, Counters of “birds adopted.”
- Tamagotchi: death, hunger, distress, decaying happiness.
- Social network: profiles, follows, discovery, leaderboards, chat, comments, co-presence, avatars in scene.
- Push/email about the aviary (except magic-link and optional visit-notification toggle off by default).
- User-visible personality numbers; client-owned personality state; last-write-wins personality merge.
- Recorded-audio call libraries; entry spinner / “welcome back” toasts.

### Defensible calls where PRD is open

| Topic | Decision |
|--------|----------|
| Tick cadence | 60s nominal; allow 45–90s under load; tick must be idempotent and order events by server receipt time. |
| Presence activity window | 3 minutes without pointer/key → presence ends; calibrate later, prefer long side. |
| Trait range | Each trait ∈ [0.0, 1.0]; starters sampled mid-band with species-biased offsets. |
| Mood enum | `wary`, `content`, `curious`, `drowsy`, `alert`, `settled` (night/settle destination). |
| Offer cooldown | 4 minutes per bird per offer-type; server-enforced. |
| Notebook sparsity | Target ~1 entry / 2–4 calendar days of genuine presence; spikes on rarity events (first greet order flip, weather+call hush). |
| New-bird pacing | 3rd ~assoc age 6–8 weeks presence-capable age; further birds on widening age intervals; never pay-to-unlock. |
| Species pool | 6 species; internal codes stable; one nocturnally active species. |

---

## 2. Architecture

### High-level shape

```
┌─────────────────────────────────────────────────────────────┐
│  Browser client (SPA)                                         │
│  Render loop · WebAudio · presence · event emit · a11y       │
└───────────────┬───────────────────────────▲───────────────────┘
                │ HTTPS REST + SSE/WS       │ snapshots / auth
┌───────────────▼───────────────────────────┴───────────────────┐
│  Edge (CDN + API gateway)                                     │
│  static assets · boot HTML with optional snapshot bootstrap   │
└───────────────┬─────────────────────────────────────────────┘
┌───────────────▼─────────────────────────────────────────────┐
│  API service                                                  │
│  auth · event ingest · snapshot read · visits · account ops   │
└───────┬─────────────────┬─────────────────┬─────────────────┘
        │                 │                 │
┌───────▼──────┐  ┌───────▼──────┐  ┌───────▼──────────────┐
│ Auth store   │  │ Event log    │  │ Simulation workers   │
│ sessions     │  │ append-only  │  │ tick per account     │
│ magic links  │  │              │  │ personality / mood   │
└──────────────┘  └──────▲───────┘  └──────────┬───────────┘
                         │                     │
                  ┌──────┴─────────────────────▼───────────┐
                  │ Canonical aviary store (Postgres)        │
                  │ accounts · birds · vectors · notebook    │
                  └──────────────────────────────────────────┘
```

### Service boundaries

1. **API service** — request/response: auth, snapshot GET, event POST, notebook GET, settings, visits, export/delete.
2. **Simulation workers** — pull accounts due for tick (lease row / outbox), consume unread events, write bird/aviary rows, emit notebook candidates.
3. **Mailer** — magic links, export links, visit invites (transactional only).
4. **Static** — SPA bundle via CDN; chunked routes for settings/visits/export.

No microservice explosion: one deployable API + one worker pool is enough for v1 if tick is O(seconds) per account and presence execs are small.

### Client/server split (hard rules)

| State | Owner |
|--------|--------|
| Personality vector | Server only (tick writes) |
| Mood, perch intent, weather, settle flag | Server tick + ephemeral session modifiers |
| Interaction/presence events | Client append; server disulfide durable log |
| Visual interpolation, leaf drift, audio buffers | Client only |
| Narration string cache | Client render of server-or-client prose generator fed by snapshot |

### Render pipeline boundary

- **Simulation truth**: server snapshot (positions targets, mood, call phase seeds, weather, lighting phase, offer cooldowns).
- **Presentation**: client animation system interpolates poses; never invents personality; may synthesize call phases from seeds so audio stays procedural offline between snapshots.
- When tab hidden: stop rAF and WebAudio (or suspend context); do not stop draining presence on hide (presence ends); simulation continues on server.

---

## 3. Data model

### Account

```
Account {
  id: UUID                 // synthetic; ONLY internal key
  email_enc: bytes         // single store of email PII
  email_hash: bytes        // for lookup / rate-limit; keyed hash
  created_at: timestamptz
  aviary_born_at: timestamptz
  locale_tz: string        // IANA; used for day cycle + mood TOD
  deleted_at: timestamptz?
  settings: jsonb          // a11y, visit notify opt-in, caption default
  bird_cap: int            // 7
}
```

### Session

```
Session {
  id: UUID
  account_id: UUID
  device_label: string?
  token_hash: bytes
  created_at, last_seen_at, revoked_at?
}
```

### Bird

```
Bird {
  id: UUID                 // stable lifelong identity
  account_id: UUID
  species_id: string       // pool code
  display_name: string
  adopted_at: timestamptz
  sort_index: int
  // personality — canonical
  boldness, social_warmth, vocal_frequency,
  plumage_saturation, curiosity: float4  // [0,1]
  personality_updated_at: timestamptz
  // mood — canonical
  mood: enum
  mood_updated_at: timestamptz
  // placement intent (server)
  perch_zone: enum front|middle|back
  perch_slot: int
  // cooldowns
  last_offer_at: jsonb     // by offer type
}
```

### Aviary ambient

```
AviaryState {
  account_id: UUID PK
  weather: enum clear|rain|wind
  weather_until: timestamptz?
  settled: bool
  settled_at: timestamptz?
  lighting_phase: float    // 0–1 day cycle fraction derived at tick from tz
  version: bigint          // monotonic snapshot version
  last_tick_at: timestamptz
}
```

### Event log (append-only)

```
InteractionEvent {
  id: bigserial
  account_id: UUID
  bird_id: UUID?
  type: enum
    presence_ping | listen_in_start | listen_in_end |
    offer_seed | offer_song | offer_pool |
    settle | unsettle | tab_focus_return |
    rename_bird | visit_view  // visitor events never affect host drift
  payload: jsonb           // durations, song_id, client_ts
  server_received_at: timestamptz
  processed_at: timestamptz?
  actor: enum host | visitor
  session_id: UUID?
}
```

### Notebook

```
NotebookEntry {
  id: UUID
  account_id: UUID
  created_at: timestamptz
  prose: text              // naturalist lowercase
  trigger: string          // internal reason code; not shown
}
```

### Visits

```
VisitInvite {
  id: UUID
  host_account_id: UUID
  visitor_email_enc / hash
  token_hash: bytes
  created_at, expires_at   // 30d unused
  revoked_at?
  accepted_at?
}
VisitSession {
  id: UUID
  invite_id: UUID
  started_at, ended_at?
  approx_duration_s
}
```

### Magic link

```
MagicLink {
  id: UUID
  email_hash
  token_hash
  expires_at               // 15 min
  consumed_at?
}
```

Indexes: events `(account_id, processed_at NULLS FIRST, server_received_at)`; birds `(account_id)`; version on AviaryState for cache/ETag.

---

## 4. API surface

Base: `/api/v1`. Auth: `Authorization: Bearer <session>` except magic-link and visit token routes. All account-scoped routes key by session → account UUID never from client-supplied email.

### Auth

| Method | Path | Notes |
|--------|------|--------|
| POST | `/auth/magic-link` | body `{email}`; rate-limit; matter-of-fact errors |
| GET | `/auth/magic-link/consume?token=` | set session cookie/token; invalidate link |
| POST | `/auth/sign-out` | |
| GET | `/account/sessions` | list devices |
| DELETE | `/account/sessions/:id` | revoke |
| POST | `/account/email-change` | verify new email |
| GET | `/account/export` | enqueue export email |
| POST | `/account/delete` | soft-delete |
| POST | `/account/undelete` | within 30d |

### Aviary state

| Method | Path | Notes |
|--------|------|--------|
| GET | `/aviary/snapshot` | full canonical+render drunk state; `ETag: version`; optional `?since=` |
| GET | `/aviary/snapshot/stream` | SSE: version bumps ~tick / settle; not per-frame |
| POST | `/aviary/events` | batch append events; 202; validate rates |
| GET | `/notebook` | paginated oldest/newest; read-only |
| PATCH | `/birds/:id` | `{name}` only |
| POST | `/birds/adopt` | if age-gated offer available; server picks species |

### Snapshot payload (sketch)

```json
{
  "version": 1842,
  "server_time": "...",
  "local_tz": "America/Los_Angeles",
  "lighting": { "phase": 0.22, "settled": false },
  "weather": { "kind": "clear" },
  "birds": [{
    "id": "...",
    "name": "pip",
    "species": "warbler_a",
    "mood": "content",
    "perch": { "zone": "front", "slot": 0 },
    "motion_seed": 9012,
    "call": { "grammar_id": "...", "vocal": 0.55, "phase_seed": 12 },
    "visual": { "plumage": 0.48 },
    "cooldowns": { "seed": 0 }
  }],
  "greeting": {
    "bird_id": "...",
    "absence_s": 7200,
    "style": "soft_call",
    "stagger_ms": [0]
  },
  "adopt_available": false
}
```

**No personality numbers in client payload** except traits that must leak as continuous render params: plumage saturation and vocal-frequency-as-rate may stream as anonymous floats without naming them “personality.” Prefer deriving: server sends `call_rate`, `approach_bias`, `plumage` as presentation fields, never the vector table names in UI/debug panels.

### Presence / events body

```json
{
  "client_sent_at": "...",
  "events": [
    { "type": "presence_ping", "dt_s": 30, "seq": 8 },
    { "type": "listen_in_start", "bird_id": "..." },
    { "type": "offer_seed", "bird_id": "..." }
  ]
}
```

Server ignores visitor `actor` events for drift; visit viewer uses separate route:

| GET | `/visit/:token/snapshot` | read-only; no event write for host drift |
| POST | `/visit/invites` | host email list |
| GET | `/visit/invites` | outstanding |
| DELETE | `/visit/invites/:id` | revoke immediate |
| GET | `/visit/log` | host settings |

Errors use matter-of-fact English (expired link, session timeout, visit unavailable).

---

## 5. Simulation engine design

### Tick loop (worker)

1. Claim account lease (`FOR UPDATE SKIP LOCKED` or Redis lease).
2. Load `AviaryState`, birds, unprocessed events ordered by `server_received_at`, `id`.
3. **Time-of-day**: compute lighting phase from `locale_tz` + now.
4. **Weather**: rare Markov — few rains/week; short durations; set mood biases.
5. **Aggregate presence**: sum `presence_ping.dt_s` since last tick (host only).
6. **Apply interaction deltas** (see drift) without overwriting with absolute client values.
7. **Mood transitions**: Markov with personality multipliers + TOD + weather + recent offers/listen-ins.
8. **Perch choice**: sample zone from boldness×mood; avoid all birds collapsing front unless extreme.
9. **Bird-to-bird**: if call windows overlap high vocal birds → chorus flag; wary contagion short-lived.
10. **Notebook candidate**: if rarity heuristics fire and quota allows → enqueue prose job.
11. **Mark events processed**, bump `version`, `last_tick_at`, release lease.

Tick p99 alarm > 5s (ops metric).

### Drift function

Low-pass, monotonic **up toward expressive** only:

```
for each trait T:
  raw = w_p * presence_hours
      + w_l * listen_minutes_on_bird
      + w_o * offer_accept_signals
  // settle ends presence cleanly; ~0 drift
  delta = k * tanh(raw) * (1 - T)     // sensitive early, slows near 1
  T := clamp(T + delta, 0, 1)
  // never T := T - something on neglect
```

**Calibration targets**

- Rectified ~1h focused presence/day, ~5 days: instruments see Δ_mean in [0.005, 0.02] on at least one trait.
- ~3 weeks: perceptible (bolder front perch, greeter order changes, call rate).
- Single session Δ trait < user-perceptible threshold (~0.01 upper).

Trait-specific weights: presence → all mild; listen-in → social_warmth + vocal_frequency; offer accept → curiosity; offer near bird → boldness slight.

**Ambient quietness on neglect**: not trait decrease — reduce greeting rate and front-perch prior using *absence length* as a transient mood/intent modifier, leaving vector untouched.

### Mood transitions

Inputs ranked: recent session events > TOD > weather > social contagion > personality priors.

Examples:

- Dusk → +drowsy/+settled probability.
- Early local morning → +alert.
- Offer accepted → +content.
- Rain → temporary −vocal expression (mood/content, not permanent trait down).
- Alarm (optional ambient) → nearby +wary, decays.

Persist mood across sessions; tick advances offline.

### Call-grammar runtime (server seeds, client synth)

Server persists per species/bird:

- Motif library id, base pitch centroid, interval preferred, max notes, silence distribution shaped by `vocal_frequency`.
- Each snapshot: `phase_seed`, `next_call_window` bounds.

Client WebAudio:

- Builds note graphs (oscillators + noise bursts + envelopes) per call instance.
- Variation: timing jitter, micro pitch drift, motif permutation — **never identical twice**.
- Chorus: independent voices mixed; shared master bus; listen-in ducking slope ~800–1500ms.
- Recognizability: species filter + bird pitch offset locked for life of bird id.

### Greetings (return)

On snapshot after presence gap:

1. Rank birds by boldness × social_warmth × ¬wary.
2. Pick primary greeter; optional secondary with stagger 200–1200ms (never unison).
3. Style from absence buckets: <5min glance; <1day short call; multi-day approach + longer call.
4. Client plays motion/audio from server-chosen style enum + seeds — not three canned clips.

### Adoption / species

Account birth relies two birds: complementary species + trait seeds. Age-gated third+: server sets `adopt_available`; species random from pool unused-or-repeat allowed (no rarity economy). Names default suggestions; rename free.

### Notebook generation

Template+slot naturalist generator (not LLM required for v1):

- Facts: greeter order uniquely this week, long quiet presence, weather+preen, mood day.
- Output forced lowercase, present tense, bird names, no user-behavior scoring, no numbers of traits.
- Rate limiter rounded with dilution for hyperactive users.

---

## 6. Sync model

### Canonical single writer

- Only tick writes personality and authoritative mood/perch/weather/version.
- Clients only append events.
- Multi-device: both read same snapshot stream; no CRDT personality merge.

### Client refresh triggers

1. Initial boot (HTML-bootstrap snapshot if warm edge cache key by session eu irregular — careful with PII; prefer authenticated API fast).
2. `visibilitychange` → visible: immediate snapshot.
3. Long rAF gap (resume from sleep) → snapshot.
4. Keepalive poll 15–30s while visible OR SSE version bumps.
5. After local settle/offer optimistic UI, reconcile on next version.

### Conflict prevention

- Personality: additive deltas from ordered log → no LWW.
- Offers: server cooldown truth; client disable UI on reject.
- Dual-device simultaneous settle: last processed event wins settle flag; both valid.
- Magic-link replay: single consume.
- Soft-deleted account: API 410 matter-of-fact.

### Visit isolation

Visitor token → snapshot of host aviary **without** host session IDs; events from visitor never enter host drift aggregation; host visit log only.

---

## 7. Frontend rendering pipeline

### Stack guidance

- TypeScript SPA; WebGL or Canvas2D + SVG hybrid acceptable if 60fps idle budget holds.
- Prefer lightweight custom scene graph over heavy game engines to hit <2MB gzip JS.

### Scene composition

Layers back→front: sky/gradient (TOD), soft foliage parallax (subtle), perch planes (back/mid/front), birds, occasional FG leaf, no in-scene buttons.

Responsive: reflow perch x-spacing; never crop birds; phone compresses horizontal gaps.

### Boot / first frame

1. Quiet field placeholder (soft sky, faint motion) if snapshot slow — **no spinner**.
2. On snapshot: place birds at mid-animation phase from `motion_seed` + server time (phase = f(seed, t)); audio may start after gesture if browser autoplay policy — captions ready.
3. Return greeting within 1–2s once audio/motion ready.

Empty aviary only once at first adoption fly-in; never again.

### Idle micro-motion

Continuous loops keyed by mood:

- wary: back bias, scan
- content: preen
- curious: head-tilt to ambient sound loc
- drowsy/settled: low silhouette, fluff
- alert: frequent small...

Client may timewarp slightly for smoothness but snaps to server perch targets over few seconds.

### Top bar

Icons: account, a11y, notebook, offer (+ settle). Fade near-transparent after ~3s idle pointer/keyboard stillness; restore on activity. No aviary-internal chrome.

### Transitions

- Day/night: slow palette lerp continuous with local time (client interpolates between tick phases).
- Settle: 2–4s evening lean + call attenuation; 5s undo any click.
- Weather: rain particles light density few minutes.

### Reduced motion

Separate pose table + cross-fades only; leave color shifts slowed; remove leaf drift; keep full sim + audio/captions. Honor `prefers-reduced-motion` + settings toggle.

---

## 8. Audio pipeline

### Procedural synthesis

- Motif graphs compiled from bird grammar (species + individual offset).
- Per-call instance parameters from seed stream so captions map 1:1 to produced call.
- Buffer pools reused — no growth over 30m.

### Chorus & listen-in

- Per-bird gain nodes → bus.
- Listen-in: target bird → high gain ramp; others → ambient floor > 0 (never mute).
- Disengage: reverse ramp; empty-space click / Esc / second click bird / focus leave.

### Mix decay / settle / night

- Evening/night master duck + rate reduction except nightjar species.
- Tab hidden: suspend AudioContext.

### Fallback

WebAudio missing/denied → silence + captions forced default; no sample pack fallback.

### Autoplay

First gesture unlock; until then mild visual-only viable; greeting queues.

---

## 9. Accessibility surfaces

### Screen reader

- Live region (polite) updated 30–60s with naturalist paragraph of scene; faster on greeter/offer/settle.
- Not ARIA dump of coords/mood enums.
- Bird focusable; name + short observational description.
- Notebook entries accessible list.

### Captions

- Opt-in; prose from live grammar (“a soft three-note rise”); near bird; fade with call.

### Keyboard

- Tab: top bar → birds cycle.
- Arrows between birds; Enter listen-in; Esc exit.
- Offer/settle fully keyboard.
- Focus ring high-contrast on bright/dim skies.

### Contrast

WCAG AA all chrome/settings/errors/captions. Matter-of-fact voice on system surfaces; naturalist elsewhere.

---

## 10. Performance & observability

### Budgets

| Metric | Budget |
|--------|--------|
| Initial JS gzip | < 2MB |
| Time to first bird | < 500ms mid-tier 4G (synthetic + RUM) |
| Idle FPS | 60 on ~5yo laptop, 30m session |
| Memory | flat 30m (CI soak) |
| Tick p99 | alert > 5s |

### Choices driven by budgets

Code-split settings/visits/export; procedural audio/visual; small snapshots; CDN static; snapshot parallel to shell paint.

### Measure

- Synthetic geo browsers: TTFB, first-bird, FPS probes.
- RUM aggregates: load, first-bird, long tasks, audio errors — **no account/bird dimensions**.
- Worker: tick latency, event lag depth, lease failures.

### Deliberately do not measure

Per-bird interaction funnels for product analytics; engagement streaks; cross-account “average boldness.”

Privacy: simulation DB never ETL’d to warehouse; telemetry pipelines separate passwordless role.

---

## 11. Rollout

### Phases

1. **Internal dogfood** — 2 birds fixed, drift instruments dashboards (ops only, not user-facing numbers).
2. **Closed beta** — magic-link allowlist; validate presence honesty vs laptop-asleep false positive.
3. **GA v1** — 2 starters; bird 3+ age gates live; visits off default.

### Birds-per-aviary ramp

- Launch: hard cap 7 but adoption gates delayed so most users 2–3 early months.
- Audio mix emissions telemetry (errors only) if chorus clog; do not raise captoward>7 without listen tests.

### Day-one instrumentation

Tick health, auth fail rates, snapshot latency, first-bird RUM, audio context fail, CI memory, drift harness (synthetic presence scripts — offline lab accounts only).

### Content ops

Species art + motif libraries freeze checklist; first-boot quiet field assets tiny.

---

## 12. Risks & mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Drift too fast | Tamagotchi feel | Cap per-day delta; weekly offline calibration suite |
| Drift too slow | screensaver | Instrument weekly Δ; tint weights; not expose numbers |
| Lax presence | inflated population drift | Triple-gate; integration tests with hidden tab / no input |
| Dual-device “merge” bug | silent personality loss | Free ban on client vector write; log audits on vector mutations (who=worker only) |
| Audio uncanny / loops | breaks aliveness | Procedural only; entropy audits; human listen chrome |
| Identical greetings | theater | Seed from time+absence+bird; stagger multi |
| A11y as afterthought | excludes users | Ship narration + RMO + captions in same release |
| Welcome toast PR | principle break | Lint/UI review ban strings; intentional empty toast container |
| Bundle bloat | fails TTFB | budget CI gate 2MB; asset account |
| Notebook LLM-temptation | generic spam | Rule-based sparse templates v1; no session log |
| Visit feature creep | social network | No APIs for comments/discovery; invite-only schema |
| Soft-delete incomplete | privacy breach | Cascading hard delete job checked; email expunge policy |
| Autoplay blocked | silent helpless | Captions path; first-click cheer quietly |

---

## 13. Implementation workstreams (executable sequence)

### WS0 — Foundations

Postgres schemas, UUID accounts, email encryption, magic-link, sessions, soft-delete.

### WS1 — Simulation core

Bird + aviary tables, event log, worker tick, drift/mood unit tests, fake clock.

### WS2 — Snapshot API + boot client

Quiet field, mid-motion spawn, keepalive/SSE, settle/listen/offer event wiring skeleton.

### WS3 — Audio + motion polish

Grammars ×6 species, chorus, listen-in ramps, reduced-motion poses.

### WS4 — Notebook + naturalist copy pipeline

Generators, sparsity controller, UI drawer.

### WS5 — Presence honesty

Client multi-signal detector; server caps absurd dt; tests.

### WS6 — A11y

Narration live region, captions, keyboard matrix, contrast audit.

### WS7 — Visits (optional late)

Invites mail, read-only snapshot, revoke, host log, no notifications default.

### WS8 — Perf CI & synthetic monitors

Bundle gate, soak memory, first-bird synthetic, tick SLOs.

### WS9 — Account export/delete, privacy policy page, session revoke UI

Matter-of-fact voice QA pass on all system strings; naturalist pass on product strings.

---

## 14. Testing strategy (non-product polish)

- **Unit**: drift monotonicity; never-negative; mood TOD tables; cooldown.
- **Property**: event order reorder safety; tick idempotent if events reprocessed-guarded.
- **Client**: presence triple-condition matrix; reduced-motion; keyboard.
- **Soak**: 30m memory/audio node counts.
- **Golden notebook**: snapshot fixtures → prose snapshots (lint lowercase, ban streak language).
- **No** tests that assert user-visible “level up.”

---

## 15. Explicit product UX rules for implementers

1. No “Welcome back,” days-gone banners, streak UIs of any form.
2. Idle watching **is** interaction via presence — do not nudge “try offering.”
3. Neglect → quieter expressivity, never distress art.
4. Voice split enforced in copy review checklist.
5. First pixel is living scene or quiet field — never branded spinner.

---

## 16. Success criteria (v1)

- User can form multi-week “knowing” a bird by ear and perch habits without stats.
- Multi-device morning/night same mood continuum.
- A11y user receives living prose, not meter reading.
- Zero gamification surface ships; visit remains optional quiet.
- Perf budgets met on target devices; sim continues offline truthfully.

---

*End of plan. Do not implement product in this phase — engineering executes from this document.*
