# Pocket Aviary — V1 Implementation Plan

This plan translates the Pocket Aviary PRD into an executable engineering roadmap. It assumes a frontier team shipping a web-only product with a server-authoritative simulation, procedural client-side audio, and a single canonical aviary per account.

---

## 1. Scope

### 1.1 In scope for v1

| Area | V1 deliverable |
|------|----------------|
| Aviary scene | Single horizontal scene, three perch zones, day/night from user local time, rare ambient weather, no pan/zoom/scroll |
| Birds | 2 starters at adoption; up to 7 total; ~6 species pool; stable internal IDs; user-assigned renameable names |
| Personality & mood | Hidden 5-trait personality vector (server-only); enumerated mood states; monotonic drift toward expressive |
| Interactions | Return-greeting, idle presence, listen-in, offer (seed / song fragment / still pool), settle |
| Field notebook | Auto-generated, read-only, sparse naturalist prose entries |
| Accounts | Single-user; email magic-link auth; synthetic UUID account IDs; session tokens per device |
| Sync | Multi-device via server snapshots; append-only interaction event log; no client personality writes |
| Social | Opt-in visit invitations (off by default); read-only ambient visitor view; visit log; revocable invites |
| Accessibility | Screen-reader narration, call captions, reduced-motion mode, keyboard navigation, WCAG AA on chrome |
| Performance | <2MB gzipped initial JS; <500ms time-to-first-bird on mid-tier mobile/4G; 60fps idle on 5-year-old laptop |
| Export / deletion | JSON export on demand; soft delete 30 days then hard delete |

### 1.2 Explicitly out of scope (non-negotiable)

- Native mobile apps
- Gamification of any kind (streaks, achievements, levels, visit calendars, XP)
- Tamagotchi mechanics (death, hunger, distress, decaying happiness meters)
- Social network surfaces (profiles, follows, discovery, leaderboards, comments, co-presence)
- Push notifications, email nudges about the aviary, "welcome back" toasts
- Payments, shared aviaries, customizable scenes, multi-aviary accounts
- Recorded-audio fallback for calls
- Exposing personality vector numerically anywhere (including debug toggles)
- User-controlled perch placement or bird arrangement

### 1.3 Defensible implementation calls (noted ambiguities)

| Topic | Decision |
|-------|----------|
| Presence activity window | **4 minutes** for pointer/key recency; tunable via server config, default chosen to favor "watching without moving" |
| Simulation tick cadence | **60 seconds** nominal; jitter ±5s to avoid thundering herd |
| Offer cooldown | **3 minutes per bird per offer type** |
| Mood enum (v1) | `wary`, `content`, `curious`, `drowsy`, `alert`, `settled` |
| Personality trait range | `[0.0, 1.0]` floats; starters seeded per species with ±0.08 jitter |
| Third+ bird unlock | Aviary age gates: 3rd at 21 days, 4th at 60 days, 5th at 120 days, 6th at 240 days, 7th at 365 days (server-configurable) |
| Notebook entry rate | Target **0.25–0.4 entries/day** for regular visitors; burst cap 1/day unless rare event |
| Tech stack | **TypeScript monorepo**: React 19 + Canvas/WebGL renderer client; Node.js (Fastify) API + worker tick service; PostgreSQL primary store; Redis for tick locks and snapshot cache |

---

## 2. Architecture

### 2.1 Service shape

```
┌─────────────────────────────────────────────────────────────────┐
│                         CDN (static + edge snapshot cache)       │
└───────────────────────────────┬─────────────────────────────────┘
                                │
┌───────────────────────────────▼─────────────────────────────────┐
│  Web Client (React)                                              │
│  - Scene renderer (Canvas 2D primary; WebGL optional path)       │
│  - WebAudio call synthesizer                                     │
│  - Presence detector → event writer                              │
│  - Snapshot poller + interpolator                                │
│  - A11y narration + captions                                     │
└───────────────────────────────┬─────────────────────────────────┘
                                │ HTTPS (REST + SSE optional)
┌───────────────────────────────▼─────────────────────────────────┐
│  API Service (stateless)                                         │
│  - Auth (magic link, sessions)                                   │
│  - Snapshot read                                                 │
│  - Interaction event append                                      │
│  - Visit token validation                                        │
│  - Notebook read                                                 │
│  - Account settings / export / deletion                          │
└───────┬─────────────────────────────────────┬───────────────────┘
        │                                     │
        ▼                                     ▼
┌───────────────┐                    ┌────────────────────────────┐
│  PostgreSQL   │◄───────────────────│  Simulation Tick Worker     │
│  - accounts   │   reads/writes     │  - ~1/min per active aviary │
│  - birds      │                    │  - mood transitions         │
│  - events log │                    │  - drift deltas             │
│  - snapshots  │                    │  - weather/ambient rolls    │
│  - notebook   │                    │  - notebook candidate gen   │
└───────────────┘                    └────────────────────────────┘
        ▲
        │ email
┌───────┴───────┐
│  Mail provider │
└───────────────┘
```

### 2.2 Client/server split

| Responsibility | Owner |
|----------------|-------|
| Canonical personality vectors | Server only |
| Mood state transitions | Server tick |
| Perch selection logic | Server tick (client interpolates) |
| Call timing / chorus scheduling hints | Server snapshot; client executes synthesis |
| Procedural call audio | Client WebAudio |
| Visual rendering, idle micro-motion | Client (driven by snapshot + local interpolation) |
| Presence detection | Client emits pings; server accumulates presence-time |
| Listen-in mix | Client audio graph only (no server mix state) |
| Field notebook prose | Server generates and stores entries |
| Visit read-only rendering | Client uses visitor-scoped snapshot token |

### 2.3 Render pipeline boundary

The server snapshot is the **only** source of bird pose, perch zone, mood, and simulation-time. The client:

1. Pulls snapshot on load, visibility resume, keepalive (30s visible), or >10s render gap.
2. Maintains a **render clock** independent of network; interpolates position/pose between snapshots.
3. Runs **client-only ornaments**: leaf/feather drift, listen-in gain ramps, settle lighting overlay, reduced-motion cross-fades.
4. Never infers personality or mood changes locally except for sub-second visual anticipation (e.g., offer reaction animation starts on event ack, corrected on next snapshot).

---

## 3. Data Model

### 3.1 Core entities

#### `accounts`
- `id` UUID (synthetic, partition key everywhere)
- `email_encrypted` bytea
- `email_hash` for lookup (HMAC-Salted, not reversible)
- `created_at`, `timezone` (IANA string, default from first session)
- `settings_json` (a11y prefs, visit notifications off by default, reduced-motion override)
- `deletion_scheduled_at` nullable
- `aviary_id` FK (1:1 at v1)

#### `aviaries`
- `id` UUID
- `account_id` FK unique
- `created_at` (drives bird-unlock age gates)
- `simulation_cursor` (last processed event sequence)
- `last_tick_at`
- `settled_until` nullable timestamp (server-side settle state)
- `local_time_anchor` (for day/night; derived from account timezone + server UTC)

#### `birds`
- `id` UUID (immutable identity)
- `aviary_id` FK
- `species_id` FK
- `display_name` text
- `adopted_at`
- `personality` JSONB: `{ boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity }` — **never sent to client in API responses**
- `mood` enum
- `perch_zone` enum (`front` | `middle` | `back`)
- `pose_key` string (renderer lookup)
- `active_animation` nullable
- `last_greeted_at` nullable
- `offer_cooldowns` JSONB per offer type

#### `species` (seed table)
- `id`, `silhouette_asset_key`, `default_palette`, `motif_library_id`, `nocturnal` bool (one nightjar-like species)

#### `interaction_events` (append-only)
- `id` bigserial
- `aviary_id`, `sequence` monotonic per aviary
- `type` enum: `presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `settle_undo`, `session_end`
- `bird_id` nullable
- `payload` JSONB (offer type, duration seconds, etc.)
- `client_timestamp`, `server_received_at`
- `device_session_id`

#### `aviary_snapshots` (materialized, latest + optional history for debug)
- `aviary_id`, `version` incrementing
- `generated_at`
- `state` JSONB: per-bird public fields (id, name, species visual ref, mood, perch, pose, call_schedule hints, plumage_render_params derived from saturation trait without exposing number)
- `ambient` JSONB: time-of-day phase, weather event if active, settled flag
- `chorus_hints` JSONB: upcoming call windows per bird (timing only)

#### `notebook_entries`
- `id`, `aviary_id`, `written_at`, `body` text (lowercase naturalist prose), `trigger_event_id` nullable

#### `sessions`
- `id`, `account_id`, `device_label`, `created_at`, `last_seen_at`, `revoked_at` nullable

#### `visit_invitations`
- `id`, `host_account_id`, `visitor_email_hash`, `token_hash`, `created_at`, `expires_at` (30 days), `revoked_at`, `consumed_at`

#### `visit_log`
- `id`, `invitation_id`, `started_at`, `ended_at`, `approx_duration_seconds`

### 3.2 Personality vector handling

- Stored only in `birds.personality` and account export.
- API snapshot builder maps `plumage_saturation` → `plumage_render_params` (color multipliers) without revealing scalar.
- Tick worker applies **additive deltas** clamped to `[0, 1]`; no negative drift on neglect (traits freeze, not decay).

### 3.3 Event-sourcing boundary

Interaction events are the audit trail. Personality is **not** recomputed from full history at runtime; tick applies incremental deltas from events since `simulation_cursor`. Nightly job verifies drift integrity via checksum sampling (ops only).

---

## 4. API Surface

Base path `/api/v1`. All authenticated routes use `Authorization: Bearer <session_token>` except magic-link exchange and visitor tokens.

### 4.1 Auth

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/auth/magic-link` | Body: `{ email }`. Rate limit 5/hour/email. Sends link. |
| POST | `/auth/magic-link/consume` | Body: `{ token }`. Returns session token + account summary. Single-use, 15 min TTL. |
| POST | `/auth/logout` | Revoke current session |
| GET | `/auth/sessions` | List device sessions |
| DELETE | `/auth/sessions/:id` | Revoke device |

### 4.2 Aviary state

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/aviary/snapshot` | Current public snapshot + `snapshot_version`. ETag supported. |
| GET | `/aviary/snapshot/stream` | Optional SSE: push on new snapshot version (visible tab keepalive). |
| POST | `/aviary/events` | Append interaction event(s). Body: `{ events: [...] }`. Returns `{ accepted_sequence }`. |
| POST | `/aviary/settle` | Convenience wrapper → `settle` event |
| POST | `/aviary/settle/undo` | Within 5s of settle |

### 4.3 Birds & adoption

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/birds` | Names, species visuals, moods (no personality numbers) |
| PATCH | `/birds/:id` | Rename only |
| POST | `/birds/adopt` | Available when age gate open; server picks species; user names in same flow |

### 4.4 Notebook

| Method | Path | Purpose |
|--------|------|---------|
| GET | `/notebook` | Paginated entries, newest first |

### 4.5 Account

| Method | Path | Purpose |
|--------|------|---------|
| GET/PATCH | `/account/settings` | Timezone, a11y, visit notification toggle |
| POST | `/account/export` | Async JSON export emailed |
| POST | `/account/delete` | Schedule soft delete |
| POST | `/account/delete/cancel` | Within 30-day window |

### 4.6 Visits

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/visits/invite` | Body: `{ email }`. Sends one-time visitor link. |
| GET | `/visits/invitations` | Host: outstanding + history |
| DELETE | `/visits/invitations/:id` | Revoke |
| GET | `/visits/log` | Host visit log |
| GET | `/visits/view` | Visitor: query `?token=`. Returns read-only snapshot stream. No event write endpoints. |

### 4.7 Error voice

Auth, sync, and load failures return matter-of-fact copy per PRD. HTTP 401/403/410 for expired magic links and revoked visits.

---

## 5. Simulation Engine Design

### 5.1 Tick loop

For each aviary due for tick (active in last 7 days OR tick lag > 1 hour):

1. **Load** birds, personality (worker only), mood, cursor, recent events.
2. **Ingest events** in sequence order since cursor.
3. **Update presence accumulator** from `presence_ping` events (count only when pings include `visible+focused+active` flags from client).
4. **Compute drift deltas** (see §5.3) and apply to personality.
5. **Transition moods** based on time-of-day, weather, recent offers, bird-to-bird signals.
6. **Update perch zones** from mood + boldness (stochastic weighted choice, hysteresis to avoid flicker).
7. **Schedule call hints** for next window based on vocal_frequency, mood, chorus probability.
8. **Roll ambient weather** (~2–3×/week Poisson per aviary).
9. **Evaluate notebook triggers** (sparse).
10. **Build snapshot** vN+1, persist, bump version.

Tick target p50 < 200ms, p99 < 2s. Lock per aviary via Redis `SETNX` to prevent double tick.

### 5.2 Mood transitions

Finite state machine with weighted edges. Inputs:

- **Session-local**: recent offer accepted → nudge `content`; alarm call from bird A → nearby birds → `wary`.
- **Time-of-day**: morning → `alert`; late evening → `drowsy`; post-settle → `settled`.
- **Weather**: rain → temporary vocal dampening modifier (not a mood per se, but affects call hints and edge weights toward `content`/`drowsy`).
- **Personality bias**: high boldness reduces `wary` entry probability.

Mood persists across sessions; tick continues transitions while user away.

### 5.3 Drift function

Low-pass filter over **rolling 7-day window** of signals:

| Signal | Weight | Trait targets |
|--------|--------|---------------|
| presence-time (minutes) | 1.0 (dominant) | all traits slight uplift; plumage_saturation, social_warmth strongest |
| listen-in duration per bird | 0.35 | that bird's social_warmth, vocal_frequency |
| offer proximity / acceptance | 0.15 | curiosity (accept); boldness (offer near bird) |
| settle | 0.05 | none directional; closes presence window cleanly |

**Monotonic rule**: per trait, delta = `max(0, computed_delta)`. Neglect → delta = 0 (trait frozen), not decrease. Observable effect: fewer greetings, quieter chorus — not punishment visuals.

Calibration harness (CI):

- Simulated 30 min/day presence for 7 days → ≥0.03 trait movement (instrument).
- Same for 21 days → perch/greeting frequency shift perceptible in golden scenario tests.

### 5.4 Call-grammar runtime (server hints + client execution)

Server stores per-species **motif DAG**: motifs = { `rise3`, `trill_pair`, `sharp_single`, ... } with pitch/timing ranges.

Snapshot `chorus_hints` includes for each bird next 2–3 call windows with:

- `motif_id`, `pitch_scale`, `duration_ms`, `jitter_ms`

Client synthesizer:

- Oscillator + bandpass per motif segment
- Personality shapes base pitch and inter-call interval
- Mood shapes attack/decay and motif selection weights
- Two+ overlapping calls → chorus bus with gentle compression (no phase-locked loops)

Captions generated from the same motif metadata: "a soft three-note rise".

### 5.5 Return-greeting selection (server-side flag in snapshot)

On client session start, client sends `session_start` event with `absence_seconds`. Tick sets `greeting_bird_id` and `greeting_style` in snapshot:

- Select one bird weighted by `boldness × social_warmth`, excluding birds in `settled` sleep.
- Style scales with absence: <5 min → glance; 1–48h → call + step forward; >48h → longer call, possible second bird response after stagger 800–2000ms random.
- Procedural variation: pick motif subset + micro-timing jitter; **never** repeat exact parameters from previous greeting (store hash of last 10).

### 5.6 Bird-to-bird interaction

During tick, if multiple birds have call windows overlapping:

- Emit `chorus_event` internal flag → notebook may note it
- Wary mood contagion: if any bird in `wary`, 20% chance neighbors edge toward `wary` unless boldness > 0.7

### 5.7 Notebook generation

Template + NLG light (rules-based, not LLM in v1):

- Triggers: first-greeter-of-day, first-time-this-week pair ordering, long quiet stretch (>2h no calls), chorus, weather start/end, adoption milestone.
- Sparsity governor: min 48h between routine entries; max 1/day unless `rare_event`.
- Voice linter: enforce lowercase, present tense, bird names, no user-behavior stats, no numbers from personality.

---

## 6. Sync Model

### 6.1 Canonical state flow

1. Client A and B both authenticate → same `aviary_id`.
2. Each appends events to shared log (ordering by server sequence).
3. Tick consumes log → single personality truth.
4. Clients poll snapshot version; if `etag` unchanged, 304.

### 6.2 Conflict prevention

- **No LWW on personality**: clients cannot POST personality.
- **Events are facts**: "listened to Pip 180s" not "set warmth 0.8".
- **Idempotency**: client sends `event_id` UUID; server dedupes.
- **Clock skew**: server timestamps authoritative for tick; client timestamps advisory only.

### 6.3 Offline / suspended laptop

On resume after long gap:

- Client requests snapshot with `If-None-Match`.
- Full snapshot replaces local interpolator state.
- Missing presence pings while hidden → no presence credit (by design).

### 6.4 Visit sync isolation

Visitor token scoped read-only to host `aviary_id`. Separate rate limits. No events endpoint on visitor routes. Revocation checked each snapshot request.

---

## 7. Frontend Rendering Pipeline

### 7.1 Boot sequence (time-to-first-bird)

1. Inline **critical CSS** + **quiet field** shell in HTML (sky gradient, no spinner).
2. Load minimal entry chunk (<400KB gz target).
3. Parallel: auth session restore + snapshot fetch (edge-cached).
4. Draw first bird from snapshot pose within first animation frame after data — target **<500ms** from navigation.
5. Lazy-load settings, notebook UI, visit flows.

### 7.2 Scene composition (Canvas 2D)

Layers back → front:

1. Sky gradient (time-of-day keyed)
2. Background foliage (parallax 0.2)
3. Middle perch plane + birds
4. Foreground branch/leaves (parallax 0.5)
5. Offer props (pool, seed) when active
6. Caption text (a11y)
7. Top bar (DOM overlay, not canvas)

### 7.3 Bird rendering

- Species SVG rigged layers: body, wing, head, tail.
- Idle micro-motion: preen cycle, scan, fluff — **mood selects animation set**.
- Perch transitions: 1.2–2.5s ease paths (or cross-fade in reduced-motion).
- Plumage saturation trait → HSL multiplier on feather fills.

### 7.4 Interpolation

Between snapshots every ~60s + on demand:

- Position: spring-damper toward target perch coordinates.
- Pose: blend keys over 300ms on snapshot change.
- Avoid teleport: if delta > threshold, run fly-in path.

### 7.5 Day/night and settle

- Continuous gradient driven by user timezone + server `ambient.phase`.
- Settle: client overlay darkens/warms over 4s; dampens call gain −12dB; birds trend to drowsy poses.
- Undo settle: reverse over 2s if click within 5s.

### 7.6 Top bar behavior

Icons: account, a11y, notebook, offer. Fade to 15% opacity after 3s pointer idle; restore on move/key.

### 7.7 Loading / empty states

- Slow network: quiet field only (no spinner).
- Post-adoption empty: quiet field → single soft fly-in for first bird.

---

## 8. Audio Pipeline

### 8.1 Graph topology

```
Bird1 synth ─┐
Bird2 synth ─┼─► Chorus bus ─► Master gain ─► Destination
BirdN synth ─┘              ▲
                            │
                     Listen-in ducking
                     (non-focused −9dB, focused +3dB)
```

- Ramp times: 800ms on engage/disengage (exponential).
- Non-focused birds never muted fully (−9dB floor).

### 8.2 Procedural synthesis

- WebAudio `AudioContext`; resume on first user gesture if autoplay blocked.
- Motif player: scheduled oscillators with envelopes per segment.
- Per-call randomization: ±5% pitch, ±10% timing, motif variant pick.
- Buffer pool reused; no per-call allocations (perf budget).

### 8.3 Chorus

- Server hints prevent unnatural sync; client adds ±50ms jitter.
- Light dynamics compression on chorus bus.

### 8.4 Song-fragment offer

- Offer plays motif from library (synthesized, not MP3).
- Birds respond per mood/vocal_frequency: join, quiet, counter-call.

### 8.5 WebAudio fallback

If `AudioContext` fails:

- Enable captions by default (user can disable in settings).
- Silent audio path; no recorded fallback.
- Matter-of-fact one-line notice in a11y settings only (not a toast on aviary load).

---

## 9. Accessibility Surfaces

### 9.1 Screen-reader narration

- `aria-live="polite"` region off-screen.
- Prose generated from snapshot (shared template with notebook voice).
- Cadence: 45s idle default; 15s minimum gap.
- Priority queue: return-greeting, offer reaction, settle — still observational prose.

### 9.2 Captions

- Opt-in (or auto-on in audio fallback).
- Positioned near calling bird; 2s fade in/out.
- Generated from motif metadata at play time.

### 9.3 Reduced-motion mode

- Honor `prefers-reduced-motion` + settings override.
- Replace skeletal animation with 3–5 still poses cross-faded every 2–4s.
- Remove leaf/feather drift; keep slowed day/night shifts.
- Audio, drift, notebook unchanged.

### 9.4 Keyboard

- Tab order: top bar → birds left-to-right → offer menu.
- Arrow keys move bird focus ring.
- Enter: listen-in toggle; Escape: exit listen-in.
- Focus ring: 2px solid `#f5f0e6` with dark shadow, visible on all phases.

### 9.5 Contrast & copy

- All chrome/settings/errors WCAG AA.
- Naturalist voice on product surfaces; matter-of-fact on system surfaces.

---

## 10. Performance Budgets and Observability

### 10.1 Budgets

| Metric | Budget | Enforcement |
|--------|--------|-------------|
| Initial JS (gzip) | < 2 MB | Webpack bundle analyzer in CI; fail build |
| Time to first bird | < 500 ms (P75 mobile 4G) | Synthetic checks + RUM |
| Idle FPS | ≥ 60 on 2019 MacBook Pro | headless FPS test |
| Memory @ 30 min | ≤ +5 MB delta | Playwright heap snapshot CI |
| Snapshot payload | < 8 KB typical | Contract test |
| Tick duration p99 | < 5 s alarm | Worker metrics |

### 10.2 Code splitting

- `core`: scene + audio + presence
- `lazy`: settings, notebook panel, visits, export

### 10.3 Observability (aggregate only)

- RUM: TTFB, first bird paint, FPS samples, audio init failures.
- Server: request rates, tick latencies, event append errors.
- **Never** emit per-bird or per-account interaction fields to analytics warehouse.
- Simulation DB isolated from analytics pipeline.

### 10.4 Deliberately not measured

- Per-user engagement scores, streak eligibility, visit leaderboards, personality distributions across accounts.

---

## 11. Rollout

### 11.1 Phased delivery

| Phase | Weeks | Deliverable |
|-------|-------|-------------|
| 0 — Foundation | 1–2 | Monorepo, auth, data model, snapshot read, quiet field boot |
| 1 — Core loop | 3–5 | 2 birds, tick worker, mood/perch, presence, return-greeting |
| 2 — Audio | 6–7 | Procedural calls, listen-in, captions |
| 3 — Interactions | 8–9 | Offers, settle, notebook v1 |
| 4 — A11y & motion | 10 | Narration, reduced-motion, keyboard |
| 5 — Sync & accounts | 11 | Multi-device, export, deletion |
| 6 — Social | 12 | Visits, invite/revoke |
| 7 — Hardening | 13–14 | Perf gates, calibration harness, security review |

### 11.2 Bird ramp

- Internal dogfood: 2 birds only.
- Private beta: unlock 3rd bird age gate enabled; monitor audio chorus at 3–4 birds.
- GA: full age gate to 7; watch recognizability telemetry (see risks).

### 11.3 Day-one instrumentation

- Snapshot latency, tick lag, event accept rate, first-bird RUM, audio context success rate, a11y feature adoption (aggregate counts only).

### 11.4 Launch gates

- All CI perf budgets green for 7 consecutive days on `main`.
- Drift calibration scenario tests pass.
- Accessibility audit: keyboard-only path, VoiceOver walkthrough, reduced-motion review.
- Privacy review: confirm telemetry partition.

---

## 12. Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| Drift too fast/slow | Breaks "weeks not sessions" promise | Calibration harness with simulated presence; tunable weights in server config without deploy |
| Presence definition too loose/tight | Corrupts drift signal | Client sends explicit tri-state flags; server discards incomplete pings; A/B calibration on activity window |
| Sync / personality loss | Destroys trust in bird identity | Server-only writes; additive deltas; export checksum; nightly integrity sample |
| Audio uncanny / repetitive | Breaks aliveness | Procedural jitter; ban recorded loops; golden-ear review per species; caption coupling |
| Chorus mud at 6–7 birds | Per-bird recognizability collapses | Enforce cap; dynamic motif separation; user testing before enabling 7th bird |
| "Welcome back" creep | Violates core philosophy | PR checklist on UI PRs; lint against banned strings; design review gate |
| Gamification creep | Product identity erosion | Explicit non-goals in PR template; reject features that surface visit counts to user |
| Accessibility as afterthought | Excludes users from real experience | Ship narration/reduced-motion with core; same sprint ownership as renderer |
| WebAudio failures on mobile | Silent aviary feels broken | Auto captions; visible a11y indicator in settings; no silent failure mode without captions |
| Visit feature scope creep | Accidental social network | Read-only token; no host notifications by default; no discovery endpoints |
| Bundle bloat | Misses 500ms first bird | Strict CI budget; procedural assets only; lazy routes |
| Notebook too chatty | Dilutes charm | Sparsity governor; human review of template outputs in QA |

---

## 13. Team Workstreams (parallel)

1. **Simulation & data** — schema, tick worker, drift/mood, notebook generator.
2. **API & auth** — magic link, sessions, events, visits.
3. **Client renderer** — canvas scene, birds, day/night, settle.
4. **Audio** — motif libraries, synthesizer, listen-in, captions.
5. **A11y** — narration, keyboard, reduced-motion.
6. **Infra** — CDN, edge snapshot cache, Redis locks, CI perf gates.

---

## 14. Acceptance Criteria (v1 ship)

- [ ] User signs in via magic link; aviary appears mid-motion without spinner or welcome toast.
- [ ] One bird greets within 2s on return; greeting varies by absence and bird.
- [ ] Presence while watching without clicking accumulates; background tab does not.
- [ ] Listen-in ramps over ~800ms; other birds remain audible.
- [ ] Offers respect cooldown; bird reactions vary by mood/curiosity.
- [ ] Settle shifts lighting; undo within 5s works; tab close without settle is not penalized.
- [ ] Notebook entries are sparse, lowercase, specific; no user visit stats.
- [ ] Two devices show same mood/drift after sync.
- [ ] Visitor read-only link works; revocation ends session matter-of-factly.
- [ ] Screen-reader hears naturalist narration; reduced-motion is cross-fade not static.
- [ ] Initial bundle <2MB gzip; first bird <500ms P75; 60fps idle; no memory leak @30min.
- [ ] Personality numbers never appear in network payloads or UI.
- [ ] No streaks, achievements, notifications, or social discovery anywhere.

---

*End of plan. Implementation teams should treat PRD voice, non-goals, and monotonic drift as invariant constraints throughout build.*
