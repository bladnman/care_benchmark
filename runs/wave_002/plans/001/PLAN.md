# Pocket Aviary — V1 Implementation Plan

Comprehensive phase-1 plan for a frontier engineering team. Interprets the PRD into an executable architecture and delivery sequence. Does not implement the product.

---

## 1. Scope

### 1.1 In scope (v1)

- Browser-only SPA (last two major versions of Chrome, Safari, Firefox, Edge)
- Single-user accounts; email magic-link auth; synthetic account UUID primary key
- One canonical aviary per account; 2 starter birds; hard cap of 7
- Server-side simulation tick (~1/min) as sole writer of personality, mood, and bird placement state
- Client pull of snapshots + local interpolation; append-only interaction/presence event log
- Multi-device sync as architectural property (no client-owned personality state)
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook (read-only)
- Presence accounting (visibility ∧ focus ∧ recent pointer/key activity)
- Day/night from user local timezone; rare ambient weather; three perch zones
- Procedural call synthesis (WebAudio); listen-in mix rebalance; captions; silence+captions fallback
- Accessibility: naturalist screen-reader narration, reduced-motion designed surface, WCAG AA chrome, full keyboard nav
- Quiet visit invitations (opt-in per invite, read-only ambient, revocable, default off)
- Account settings: sessions revoke, email change, export JSON, soft-delete 30d then hard, privacy policy link, visit log, visit-notification toggle (off by default)
- Performance budgets: initial JS ≤2MB gzipped; first bird visible <500ms on mid-tier mobile/4G; 60fps idle on 5-year-old laptop; no memory growth over 30 min

### 1.2 Explicitly out of scope (respect non-goals)

- Native iOS/Android apps; design protocols for web only
- Gamification of any kind (streaks, badges, levels, scores, calendars, XP, visit-frequency surfaces)
- Tamagotchi mechanics (death, hunger, distress, decaying happiness meters, negative personality drift on neglect)
- Social network surfaces beyond optional visits (no profiles, follows, discovery, comments, chat, avatars, leaderboards, co-presence)
- Shared/multi-user aviaries; multi-aviary accounts; payments; push/email about the aviary (except magic links and user-requested export/recover flows)
- Personality vector numeric exposure anywhere
- Recorded-audio fallback path
- Welcome toasts, banners, “you’ve been gone N days,” or any announce-on-return chrome

### 1.3 Defensible calls on ambiguity

| Ambiguity | Decision |
|-----------|----------|
| Exact presence activity window | Start at **4 minutes** idle; instrument; lean longer if users lose presence while still watching |
| Tick cadence | Start at **60s**; allow 45–90s under load; never faster than 30s |
| Personality scalar range | Floats in **[0.0, 1.0]**; starter seeds in **[0.25, 0.55]** with species bias |
| Mood enum | `{ wary, content, curious, drowsy, alert, settled, sleeping }` |
| Species pool size | **6** species with distinct silhouettes + motif libraries |
| Bird unlock pacing | Age gates only: 3rd ~day 45; 4th ~day 120; 5th ~day 210; 6th ~day 300; 7th ~day 400 (tunable; never engagement-gated) |
| Notebook rate | Target **~2–4 entries/week** for regular presence; hard cap ~1/day even for power users |
| Offer cooldown | **3 minutes per bird per offer type** |
| Visit invite TTL | **30 days** unused as specified |
| Quiet-field load | Solid soft sky + optional 1–2 CSS-only ambient drifts; no spinner |
| Backend language | **TypeScript (Node 22)** monorepo with shared types to clients |
| Primary store | **Postgres** for canonical state + event log; **Redis** for session tokens and tick locks |
| HTML/edge snapshot | Signed short-TTL cookie session + edge-cacheable anonymized empty shell; authenticated snapshot via API after session check (first-paint path below) |

---

## 2. Architecture

### 2.1 Service shape

```
┌─────────────────────────────────────────────────────────────┐
│  Browser client (SPA)                                      │
│  - Scene renderer (Canvas 2D primary; CSS/SVG accents)     │
│  - WebAudio call engine + mixer                            │
│  - Presence sensor                                         │
│  - A11y narration / captions / reduced-motion              │
│  - Event outbound queue                                    │
└──────────────▲──────────────────────────┬──────────────────┘
               │ snapshots / visit RO      │ events POST
               │                              │
┌──────────────┴──────────────────────────────▼──────────────┐
│  API gateway (HTTPS, edge TLS)                             │
│  - Auth (magic link)                                       │
│  - Snapshot read                                           │
│  - Event ingest                                            │
│  - Account / export / delete / visits                      │
└──────────────▲──────────────────────────┬──────────────────┘
               │                              │
     ┌─────────┴────────┐          ┌─────────▼─────────┐
     │  Simulation       │          │  Event log store  │
     │  worker (tick)    │◄─────────│  (append-only)    │
     └─────────┬─────────┘          └───────────────────┘
               │ writes canonical
     ┌─────────▼─────────┐
     │  Canonical state   │  birds, moods, vectors, notebook
     │  Postgres          │  invitations, sessions
     └───────────────────┘
```

**Services (logical; can start as one deployable with internal modules):**

1. **auth-api** — magic-link issue/consume, session tokens, revoke  
2. **aviary-api** — snapshots, events, notebook read, offers/settle/listen-in validation  
3. **sim-worker** — per-aviary tick; single-writer personality/mood/placement  
4. **visit-api** — invite create/revoke/accept; visitor RO snapshots  
5. **account-api** — settings, export job, soft/hard delete  
6. **notify-mailer** — transactional email only (magic links, export links, optional visit-notifs if enabled)

No analytics warehouse read path into per-bird tables. Ops telemetry ships to a separate pipeline with no join keys to bird personality.

### 2.2 Client / server split

| Concern | Owner |
|---------|--------|
| Personality vector | Server only |
| Mood, perch zone, animation phase seeds | Server tick + snapshot |
| Drift application | Server tick from event log |
| Interaction events, presence pings | Client → append log |
| Visual interpolation, micro-idle within seed constraints | Client |
| Procedural audio synthesis | Client (from snapshot call-grammar params) |
| Ambient leaf/feather ornaments | Client-only, no sim state |
| Notebook entry generation | Server tick (or dedicated notebook writer on tick) |
| Screen-reader prose | Client from snapshot + local event priorities (schema shared); optional server-generated booked lines in notebook |

### 2.3 Render pipeline boundary

- **Canonical state snapshot** is pure data (JSON). No HTML frames from server.  
- Client map: font → silhouette art → pose graph → screen space.  
- When `document.visibilityState !== 'visible'`, stop rAF and audio (or suspend AudioContext); **do not** stop presence sensor evaluation for the open-but-hidden case (presence fails visibility anyway). Simulation continues on server.  
- Resume: force snapshot fetch on visibility + long frame gap (>2s) + soft keepalive every ~45–90s while visible.

### 2.4 Monorepo layout (proposed)

```
apps/web/                 # SPA
apps/api/                 # HTTP APIs
apps/sim-worker/          # Tick consumer
packages/shared/          # Types, call-grammar IR, mood tables, onboard copy rules
packages/sim-core/        # Pure functions: drift, mood transitions, greet selection
packages/audio-grammar/   # Motif defs + caption templates (shared with client)
infra/                    # Terraform/CDK, migrations
```

`sim-core` must be pure and unit-testable without DB; worker wraps it with persistence and locking.

---

## 3. Data model

### 3.1 Accounts

```text
accounts
  id              uuid PK          -- synthetic; never email-derived
  email_enc       bytea            -- encrypted at rest
  email_hash      bytea unique     -- keyed hash for lookup only
  timezone        text             -- IANA; client-updated
  created_at      timestamptz
  deleted_at      timestamptz null -- soft delete
  hard_delete_at  timestamptz null
  settings_json   jsonb            -- a11y prefs, visit-notif opt-in, etc.
```

```text
sessions
  id              uuid PK
  account_id      uuid FK
  token_hash      bytea unique
  device_label    text
  created_at, last_seen_at, revoked_at
```

```text
magic_links
  id, account_or_email_hash, token_hash, expires_at, consumed_at
```

### 3.2 Aviary

```text
aviaries
  id              uuid PK
  account_id      uuid UNIQUE      -- one aviary per account v1
  created_at      timestamptz      -- age gate source
  settled_until   timestamptz null -- client settle cosmetic; server may clear
  weather_state   jsonb            -- or null if clear
  last_ticked_at  timestamptz
  sim_version     int              -- schema/migration of sim engines
```

### 3.3 Birds

```text
birds
  id              uuid PK          -- stable identity forever
  aviary_id       uuid FK
  species_id      text             -- from pool
  display_name    text
  sort_index      smallint
  adopted_at      timestamptz
  -- Personality (canonical; server-only writers)
  boldness        real
  social_warmth   real
  vocal_frequency real
  plumage_saturation real
  curiosity       real
  -- Fast state
  mood            text
  mood_updated_at timestamptz
  perch_zone      text             -- front | middle | back
  animation_seed  bigint           -- for mid-action first frame
  call_phase      real             -- 0..1 rhythm offset
  last_offer_at   jsonb            -- per offer type timestamps
```

Never expose personality fields on any client-facing DTO meant for UI display panels. Snapshots may include **derived display fields** (e.g. effective saturation for renderer) without naming trait engines numerically in UI.

### 3.4 Events (append-only)

```text
interaction_events
  id              bigserial / uuid
  aviary_id       uuid
  account_id      uuid             -- actor; visitors write nothing
  bird_id         uuid null
  type            text             -- presence_ping | listen_in_start | listen_in_end
                                   -- offer_seed | offer_song | offer_pool
                                   -- settle | unsettle | focus_change | client_hello
  payload         jsonb            -- duration_ms, song_id, client_ts, etc.
  client_ts       timestamptz
  server_ts       timestamptz default now()
  session_id      uuid
  processed_at    timestamptz null -- set by tick
```

Partition by `aviary_id` time if volume grows. Tick consumes `processed_at IS NULL` in `server_ts` order.

### 3.5 Notebook

```text
notebook_entries
  id              uuid
  aviary_id       uuid
  observed_at     timestamptz      -- in-world observation time
  prose           text             -- naturalist lowercase
  kind            text             -- greeting_oddity | weather | quiet | bird_pair | etc.
  source_event_ids uuid[] null
  created_at      timestamptz
```

Immutable; no user edit/delete.

### 3.6 Visits

```text
visit_invites
  id, host_account_id, visitor_email_hash, email_enc,
  token_hash, created_at, expires_at, revoked_at, accepted_at

visit_sessions
  id, invite_id, started_at, ended_at, approx_duration_s
  -- no presence contribution to sim
```

### 3.7 Snapshot DTO (client-facing)

Omit raw personality unless needed for renderer math under opaque names (`render_traits` with salt/hashed importance). Prefer:

```json
{
  "aviary_id": "...",
  "server_time": "...",
  "local_day_phase": 0.0,
  "weather": null,
  "settled": false,
  "birds": [{
    "id": "...",
    "name": "pip",
    "species": "warbler_a",
    "mood": "content",
    "perch_zone": "front",
    "pose": "preen_mid",
    "pose_t": 0.42,
    "plumage": { "sat": 0.61, "palette": "..." },
    "call": {
      "grammar_id": "warbler_a_v1",
      "vocal_rate": 0.55,
      "pitch_center": 1.02,
      "motif_weights": [0.2, 0.5, 0.3],
      "next_call_eta_ms": 4200
    },
    "listen_priority": 0
  }],
  "ambient": { "leaf_seed": 12, "parallax": 0.15 },
  "greet": {
    "bird_id": "...",
    "variant": "head_tilt_step",
    "absence_bucket": "short|medium|long",
    "stagger_ms": [0]
  },
  "notebook_latest_id": "...",
  "sim_tick": 184422
}
```

Visitor snapshots: same scene fields; strip account chrome affordances; flag `mode: "visit_ro"`.

---

## 4. API surface

All authenticated routes take session cookie or `Authorization: Bearer` (prefer httpOnly secure cookie for web). Account IDs never in logs as email.

### 4.1 Auth

| Method | Path | Notes |
|--------|------|-------|
| POST | `/auth/magic-link` | body `{ email }`; rate limit per email+IP; always generic OK |
| GET | `/auth/magic-link/consume?token=` | single use; 15m expiry; set session |
| POST | `/auth/sign-out` | |
| GET | `/auth/sessions` | list devices |
| DELETE | `/auth/sessions/:id` | revoke |
| POST | `/auth/email-change/request` | |
| POST | `/auth/email-change/confirm` | |

Matter-of-fact error bodies only.

### 4.2 Aviary state & events

| Method | Path | Notes |
|--------|------|-------|
| GET | `/v1/aviary/snapshot` | host full snapshot; supports `If-None-Match` / ETag on `sim_tick` |
| POST | `/v1/aviary/events` | batch append; schema-validated; max N/sec |
| GET | `/v1/notebook` | cursor pagination, chronological |
| POST | `/v1/account/timezone` | `{ timezone }` |

Event batch example:

```json
{
  "session_id": "...",
  "events": [
    { "type": "presence_ping", "client_ts": "...", "payload": { "window_ms": 60000 } },
    { "type": "listen_in_start", "bird_id": "...", "client_ts": "..." }
  ]
}
```

Server accepts events; never returns new personality numbers.

### 4.3 Offers / settle (optional specialized endpoints)

Prefer events-only for settle/offers/listen-in for one write path. If UX needs acknowledged reactions before next tick:

| Method | Path | Notes |
|--------|------|-------|
| POST | `/v1/aviary/offer` | validates cooldown; appends event; may return **immediate reaction hint** computed by pure function (non-persisted personality write); tick remains sole vector writer |
| POST | `/v1/aviary/settle` | event + snapshot flag `settled` |

Immediate reaction hints are soft UI; authoritative mood still lands on next tick (or special fast-path tick item — see sim).

### 4.4 Adoption & naming

| Method | Path | Notes |
|--------|------|-------|
| POST | `/v1/onboarding/complete` | if no birds: assign 2 species server-side; set names |
| PATCH | `/v1/birds/:id` | `{ display_name }` only |
| POST | `/v1/birds/adopt` | if age gate open and count < 7; system chooses species |

### 4.5 Visits

| Method | Path | Notes |
|--------|------|-------|
| POST | `/v1/visits/invites` | host; `{ email }` |
| GET | `/v1/visits/invites` | host list + outstanding |
| POST | `/v1/visits/invites/:id/revoke` | immediate |
| GET | `/v1/visits/log` | host; durations |
| GET | `/v1/visits/view?token=` | visitor session establish |
| GET | `/v1/visits/snapshot` | visitor RO snapshot |

Visitor cannot POST events. Snapshots on revoke return 410 with matter-of-fact copy.

### 4.6 Account lifecycle

| Method | Path | Notes |
|--------|------|-------|
| GET/PATCH | `/v1/account/settings` | a11y, visit notif toggle |
| POST | `/v1/account/export` | enqueue; email download link |
| POST | `/v1/account/delete` | soft |
| POST | `/v1/account/delete/cancel` | within 30d |

Export JSON includes birds, names, vectors, moods, notebook, settings — user-owned dump.

### 4.7 Realtime?

V1: **HTTP pull only** (snapshot + events). Do not require WebSocket. Optional later SSE for tick push is out of critical path.

---

## 5. Simulation engine design

### 5.1 Tick loop

Cadence: every ~60s per aviary that exists (including deleted? no — skip soft-deleted).

Worker pattern:

1. Claim aviary row with `SELECT … FOR UPDATE SKIP LOCKED` or Redis lock `sim:{aviary_id}` TTL 50s.  
2. Load birds + unprocessed events since `last_ticked_at` (and any still-null processed).  
3. Compute wall-clock / local day phase from account timezone.  
4. Apply ambient weather generator (seeded by day+aviary).  
5. Aggregate presence-time and interactions into **deltas**.  
6. Apply drift (monotonic upward only).  
7. Transition moods.  
8. Update perch preferences from mood × boldness.  
9. Advance animation seeds / call phases.  
10. Maybe emit notebook entry (sparsity gate).  
11. Maybe unlock adoption eligibility (age only; no auto-spam UI — soft flag).  
12. Write state; mark events processed; `last_ticked_at = now()`.

Tick must be **idempotent enough**: marking processed and state write in one transaction.

### 5.2 Drift function

Traits \( t \in [0,1] \). For each tick:

\[
\Delta t = \mathrm{clip}\big( w_p \cdot P + w_l \cdot L + w_o \cdot O,\; 0,\; \Delta_{\max} \big)
\]

\[
t \leftarrow \min(1,\, t + \Delta t)
\]

Where:

- \(P\) = presence seconds in window, normalized by expected weekly attention  
- \(L\) = listen-in seconds **on this bird**, stronger on warmth & vocal_frequency  
- \(O\) = offer proximity/accept signals (curiosity↑, boldness↑ small)  
- Settle contributes \(P\) end cleanliness only — **no directed trait push**  
- Neglect → \(\Delta t = 0\) (no decrease)

**Calibration targets** (instrumentation against synthetic agent profiles):

| Profile | 7 days | ~21 days |
|---------|--------|----------|
| Regular visitor (~20 min presence/day) | measurable mean \(\Delta \geq 0.01\)–0.03 on primary traits | user-visible behavior shifts (front perch rate, greet rate) |
| Idle-negligent | \(\Delta \approx 0\) | ambient expressing unchanged |

\(\Delta_{\max}\) per tick tiny (e.g. 0.001–0.003) so single sessions are invisible.

**Expression without negative drift:** greet probability and unsolicited call rate also scale with a separate **ambient attention response** (not a trait decrease): if recent multi-day presence is low, **behavior modulated by a presence-harmony term** that reduces greeting assertiveness without decreasing stored traits. This matches “quieter, not mistrusting” without violating monotonic storage.

### 5.3 Mood transitions

Markov-like rates modulated by:

| Input | Effect |
|-------|--------|
| Local hour | morning → alert/curious; dusk → drowsy; night → settled/sleeping (except nightjar species) |
| Offer accept | → content |
| Alarm/call cascade | nearby → wary brief |
| Rain | dampen vocal → drowsy/content quieter |
| Personality | high boldness suppresses wary transitions; high curiosity boosts curious |

Mood persists across sessions; tick evolves it offline.

**Fast path:** on offer endpoint, optional optimistic mood nudge written by API only if same pure function as tick would apply, still versioned; safer default: reaction animation client-side from kind+current mood without mood write until tick.

### 5.4 Call-grammar runtime

Shared IR in `packages/audio-grammar`:

```text
Grammar {
  species_id,
  motifs: [{ id, partials[], rhythm[], duration_ms_range }],
  combine_rules: sequence | call-and-response | ornament,
  mood_mod: { pitch_semitones, rate, density },
  caption_templates: [...]
}
```

**Server** stores parameters and schedules `next_call_eta`; may simulate abstract “calls happened” for notebook without audio.  
**Client** synthesizes actual samples:

1. Pull motif weights + mood mod.  
2. Compose unique contour (RNG seeded by `bird_id + call_index + seed`).  
3. WebAudio: oscillators + noise buffers + formant filters; never looping WAV.  
4. Mix into chorus bus; listen-in gain automation.

Recognizability: keep spectral fingerprint (formant centers + motif vocabulary) per species+bird instance sticky across mood.

### 5.5 Return-greeting selection (server-assisted)

On snapshot when client sends `client_hello` with `last_presence_end_at` / absence duration (or server stores last presence host session end):

1. Score birds by boldness × warmth × mood != sleeping.  
2. Pick primary greeter; optional second with stagger 400–1800ms.  
3. Map absence bucket:  
   - <15m: glance / head tilt  
   - 15m–12h: short call + step  
   - >12h: longer call, forward perch bias  

Never synchronized full-chorus greet.

### 5.6 Bird-to-bird

Tick micro-rules:

- Call response chance ∝ social_warmth × other recent virtual call  
- Wary contagion short half-life radius in perch graph  
- Chorus windows when ≥2 high vocal birds in “active” day phase  

Client mirrors with mixer ducking; source of truth for “who called” is client for audio, server for mood spill modeling from simplified event densities if needed.

### 5.7 Adoption age gates

Stored as config; eligibility computed from `aviaries.created_at`. Present as soft “a new bird may join when the aviary is ready” UI in naturalist voice when eligible — never meta progress bar.

---

## 6. Sync model

### 6.1 Canonical single writer

- Only **sim-worker** updates personality columns.  
- Clients **never** PATCH vectors.  
- Devices A and B both GET the same snapshot; no merge of personality.

### 6.2 Event ordering

- Events stamped `server_ts` on ingest.  
- Tick processes strictly ordered by `server_ts`, then `id`.  
- Duplicate client delivery: client sends `event_id` uuid; unique constraint ignores dups.

### 6.3 Conflict prevention = avoid shareable mutable client state

- No last-write-wins on birds.  
- Presence pings from two devices both count **only if** both pass honest presence (rare true dual watching). Accept additive presence (attention is attention) OR cap double-presence with max concurrent sessions contributing (prefer **cap**: only one active host presence stream per account via last-heartbeat lease).  

**Lease decision:** `presence_lease` on aviary: session_id with 90s TTL renewed by presence pings; only lessee’s presence counts. Avoids accidental dual-device drift inflation.

### 6.4 Resume correctness

On wake:rAF gap or visibility:

1. POST any queued events  
2. GET snapshot  
3. Hard-reconcile pose targets (interpolate from current rendered poses → new targets over 300–800ms; no teleport unless delta large)  

### 6.5 Error surfaces (matter-of-fact)

Copy bank for auth timeouts, snapshot failures, visit revoked — never naturalist.

---

## 7. Frontend rendering pipeline

### 7.1 Stack choices

- **React** (or Preact for bytes) for chrome/settings only.  
- **Canvas 2D** scene core for birds + parallax layers (predictable perf; easy reduced-motion stills).  
- SVG bird parts optional prerendered to bitmap atlas at load.  
- Top bar: DOM for a11y.  
- No WebGL requirement for v1 (optional later for polish without blocking TTFP).

### 7.2 Scene composition

Layers back→front:

1. Sky gradient (time-of-day zoned colors)  
2. Far foliage silhouette  
3. Back perch zone birds (scaled down, desaturated slightly)  
4. Mid zone  
5. Front zone  
6. Foreground occasional branch/leaf  
7. Captions (DOM or canvas text with AA contrast)  

Zones fixed percentage width; birds snap to soft anchors within zone with idle offsets.

### 7.3 First paint path (alive, not loading theater)

1. Shell HTML with soft sky CSS immediately.  
2. Critical JS chunk + bird atlas critical path.  
3. Parallel auth session check + snapshot.  
4. If snapshot ready < budget: first bird drawn mid-pose from `pose` + `pose_t` — **no fade-from-static, no entry animation**.  
5. If slow: quiet field only (same sky), never spinner.  
6. Bootstrap audio after user gesture if browser requires; calls start when allowed without “press to start” modal walling the scene — use first pointer/key as unlock silently.

### 7.4 Idle micro-motion

Pose graph per species: `idle_scan`, `preen`, `fluff`, `weight_shift`, `sleep`, `tilt_listen`.  
Mood selects weight table.  
Animator: continuous small sine/noise offsets + state machine blends.  
Never freeze unless tab hidden.

### 7.5 Transitions

- Perch changes: short hop/flight path BEZIER ~0.8–1.5s; reduced-motion → cross-fade still poses.  
- Settle: 3–5s warm evening grade + gain reduce.  
- Unsettle undo within 5s click: reverse grade.

### 7.6 Reduced-motion mode

Triggered by `prefers-reduced-motion: reduce` OR settings toggle.  

- Replace looping micro-anim with slow crossfades (1.5–3s) between key stills.  
- Remove leaf drift.  
- Keep day/night color drifts slowed ×2.  
- Keep full sim, notebook, audio/captions.  
Treat as designed mode, QA as first-class.

### 7.7 Responsive

Single vertical composition; horizontal squash preserves all birds in frame; maintain min bird hit-target ≥44px when possible; never crop birds.

### 7.8 Top bar

Icons: account, a11y, notebook, offer (+ settle).  
Fade to ~10–15% opacity after ~3s cursor idle; restore on pointer/key.  
No chrome inside scene.

### 7.9 Empty aviary (onboarding only)

Quiet field → first bird soft fly-in once; never again.

---

## 8. Audio pipeline

### 8.1 Graph

```
[CallSynth voice nodes per bird] → birdGain → chorusBus → masterGain → destination
                                      ↑
                              listenIn automation
ambientBus (very low wind/leaf optional, non-bird) → master
```

### 8.2 Procedural synthesis

- Base: band-limited noise bursts + FM/AM partials aligned to motif rhythm.  
- Per-bird sticky instances: pitch center disharmonic offset.  
- Mood: drowsy lowers rate & brightness; alert shortens attacks.  
- Never identical double: always reseed ornaments.

### 8.3 Chorus mixing

- Soft limiter on chorusBus.  
- Density cap: probabilistic defer if >N concurrent voices.  
- Spatial: front birds slightly louder / less LPF.

### 8.4 Listen-in

On engage focused bird:

- Focused birdGain → ramp 1.2–1.5 over **1.2s**  
- Others → ramp 0.15–0.25 floor over **1.2s** (never 0)  

Disengage (re-click, other bird, empty space, Escape): reverse ramps ~1.2s.  
Record listen_in_start/end events with bird_id.

### 8.5 Captions

Generate from grammar caption templates + actual chosen motif ids.  
DOM near bird, fade with call envelope.  
Default **on** if WebAudio unavailable.

### 8.6 WebAudio fallback

If `AudioContext` missing/denied: permanent silent mode + captions on + matter-of-fact one-line in a11y settings (“Calls can’t play in this browser; captions are on.”). **No MP3 pack.**

### 8.7 Memory

- Reuse OfflineAudioContext / buffer pools.  
- Disconnect and null nodes after call; pool oscillators.  
- CI leak test: 30min synthetic call storm → heap delta ≈0.

---

## 9. Accessibility surfaces

### 9.1 Screen-reader narration

- Live region (`aria-live="polite"`) updated every **30–60s** with naturalist paragraph from snapshot.  
- Priority queue for greeting, offer reaction, settle — still observational prose, not “event:”.  
- Avoid dumping pose telemetry.  
- Example: “a small grey bird is perched on the front rail, calling softly…”

Implementation: client templates fed by structured snapshot facts + species names; writers share style rules with notebook generator.

### 9.2 Keyboard

- Tab: top bar controls  
- Tab into scene: birds in left-to-right (or front-to-back) order  
- ←/→ or ↑/↓: move bird focus  
- Enter/Space: listen-in toggle  
- Escape: exit listen-in / close panels  
- Offer & settle: buttons in bar, full dialogs keyboardable  

Focus ring: soft high-contrast outline OK on light and night skies.

### 9.3 Contrast

WCAG AA for all chrome, settings, errors, captions, notebook text. Scene art exempt except text overlays.

### 9.4 Settings

A11y panel matter-of-fact voice: reduced motion, captions, narration verbosity (if needed: normal/minimal), mute calls (captions auto-on when mute).

Mute is user audio preference, not “silence the product’s life” — captions keep life.

---

## 10. Presence accounting

Client `PresenceSensor`:

```
condition = visibilityState==='visible'
         && document.hasFocus()
         && (now - lastPointerOrKey) < ACTIVITY_MS  // default 4 min
```

Emit `presence_ping` every 30–60s while true; stop on settle/tab close.  

Do **not**: count hidden tabs, unfocused windows, pure open-idle-overnight.

Calibration hooks: remote-config ACTIVITY_MS without redeploy.

Settle and unload both end presence; no scolding UI.

---

## 11. Field notebook generation

Server-side generator during tick when:

- Sparse RNG / token bucket allows entry  
- Or noteworthy: first greeter swap week, weather, long quiet stretch, new bird adopt  

Prose rules:

- lowercase naturalist  
- present tense  
- bird names lowercase unless UI label “Field Notebook”  
- **never** user behavior scoring (“you visited daily”)  
- **never** raw trait deltas  

Store forever; infinite scroll client with virtualization; no retain after unmount of offscreen trees (perf).

---

## 12. Performance budgets and observability

### 12.1 Budgets

| Metric | Budget |
|--------|--------|
| Initial JS gzipped | **< 2MB** (aim ≤1.2MB critical path) |
| Time to first bird | **< 500ms** mid-tier mobile 4G (lab + field) |
| Idle FPS | **60** on 5-year mid laptop, 30-min soak |
| Memory | no monotonic growth 30 min |
| Snapshot size | few KB; target < 16KB gzipped |
| Tick p99 | alarm **> 5s** |

### 12.2 How to hit TTFP

- Code-split settings, visits, export  
- Inline critical CSS sky  
- Compress atlas; procedural body tints  
- Edge CDN static assets  
- Snapshot ASAP after cookie; consider HTTP/2 push of critical JS where safe  
- Defer nonbird wallpaper detail  

### 12.3 Observability (aggregate only)

Synthetic browsers from major geos: TTFP, FPS, audio errors.  

RUM: navigation timing, first-bird paint, long tasks, AudioContext errors, API latencies — **no bird ids, no traits, no notebook prose, no emails**.  

Sim: tick duration histogram, queue lag, lock contention.  

Hard privacy line: sim DB credentials not available to BI warehouse roles.

### 12.4 Deliberately not measured for product dashboards

- Average boldness, offer rates per user (population productization)  
- Visit leaderboards inputs  
- Streak-like DAU gamification funnels as success metrics for relationship quality (ops health is fine)

---

## 13. Privacy implementation

- Email only on account row encrypted; all else UUID  
- Interaction events drive sim only; retention tied to account lifetime + delete  
- Visitors emit no sim events  
- Optional visit email notifications: double opt-in host-side toggle off by default  
- Privacy policy link plain text listing aggregates allowed  

Hard delete (day 30+): cascade birds, events, notebook, invites, sessions; scrub backups per policy schedule.

---

## 14. Frontend modules map

| Module | Responsibility |
|--------|----------------|
| `shell` | routing between scene / settings / onboarding |
| `scene/renderer` | canvas draw loop |
| `scene/animator` | pose SM + reduced-motion |
| `audio/engine` | grammar synth + mixes |
| `presence` | sensors + pings |
| `sync/client` | snapshot + event queue + backoff |
| `a11y/narration` | live region writer |
| `a11y/captions` | call captions |
| `notebook/ui` | read-only list |
| `offers/ui` | seed/song/pool picker |
| `visits/ui` | host invite manage; visitor RO shell |
| `auth/ui` | magic link matter-of-fact |

Voice lint: product copy files tagged `naturalist` vs `system`.

---

## 15. Rollout

### 15.1 Delivery phases (engineering)

1. **Foundations** — monorepo, auth magic-link, account UUID, empty aviary snapshot, quiet field  
2. **Sim-core pure** — drift/mood unit tests + calibration harness with fake timers  
3. **Tick worker + event log** — idle evolution offline proved  
4. **Two birds visual + mid-action bootstrap** — TTFP path  
5. **Audio grammar + mixer + listen-in**  
6. **Offers + settle + presence** — drift fingerprints signed off  
7. **Notebook generator + a11y narration + reduced motion + captions + keyboard** (ship same milestone as “scene complete”)  
8. **Multi-device lease + soak tests**  
9. **Visits RO**  
10. **Export/delete + finds** 

### 15.2 Bird count ramp

- Launch: max 2 live; unlocks behind age flags disabled until post-calibration; then enable 3rd+  
- Never sell unlocks  

### 15.3 Day-one instrumentation (ops)

- Auth success/fail rates  
- Snapshot latency  
- Tick p50/p99  
- TTFP RUM  
- AudioContext fail rate  
- Presence false-positive canaries (synthetic alternate tab tests)  
- JS error budget  

### 15.4 Content/art

- 6 species packs: silhouette, pose atlas, motif IR, default personality seeds  
- Calm palette design tokens with AA pairs for chrome  

### 15.5 Launch criteria (affective QA becomes release gate)

- No welcome toast regressions  
- First frame always mid-motion when data ready  
- Drift instruments meet weekly target on dogfood  
- Reduced-motion + SR story tested by accessibility reviewers as **charm**, not bare labels  
- Silence+captions path polish  

---

## 16. Risks and mitigations

| Risk | Why it hurts | Mitigation |
|------|--------------|------------|
| Drift too fast | Tamagotchi feel; area spoils attention idea | Slow \(\Delta_{\max}\); weekly harness; remote config caps; monotonic + presence-harmony split |
| Drift too slow | Feels inert | Amplify listen-in weights carefully; dogfood midday; instrument 7/21 day gates |
| Lax presence | Population over-drifts silently | Triple AND; lease; e2e “hidden tab” tests that assert zero presence events |
| Dual-device inflation | Same | Presence lease |
| LWW personality bug | Silent character loss | Code-search CI forbid client personality writes; only worker role DB grants UPDATE on trait cols |
| Audio uncanniness / loops | Spell breaks | Procedural only; listening QA checklist; no sample fallback; chorus density limits |
| Recognizability collapse at N→7 | Cap fails product | Call fingerprint tests; gradual unlock; keep 7 hard |
| Tick backlog after outage | Time skip weirdness | Catch-up capped steps per tick (e.g. max 30 virtual minutes applied per pass → multi-pass) |
| Notebook genericity | Voice fails product-wide | Style guide + snapshot snapshot-to-prose golden tests; human edit samples pre-ship |
| A11y as afterthought | Affective product gated | Same milestone; no “v1.1 a11y” |
| Bundle bloat | Miss 500ms | Budget in CI; fail PR >2MB; Preact/careful deps |
| Memory leaks in audio/canvas | Multi-hour gentle sessions die | Soak CI; pool audit |
| Contributor adds streaks/toasts | Principle break | PR checklist explicit; grep CI for banned copies (“achievement”, “streak”, “welcome back”) |
| Visit feature creep | Social network pivot | Feature flag + maintain RO only; no visitor write APIs ever |
| Magic-link email deliverability | Auth death | ESP warmups; clear retry copy |
| Soft-delete misuse | Privacy | Hard delete job audited |

---

## 17. Testing strategy

- **Unit:** sim-core drift monotonic, mood tables, greet selection, grammar deterministic given seed  
- **Property:** presence requires all three signals  
- **Contract:** snapshot schema, event validation  
- **Visual:** screenshot mid-pose load (percy/chromium)  
- **Perf:** Lighthouse CI + custom first-bird mark; FPS bench  
- **A11y:** axe on chrome; keyboard path e2e; narration cadence e2e  
- **Privacy:** static analysis no email in logs; warehouse role cannot SELECT birds  
- **Chaos:** kill worker mid-tick; ensure lock release & no partial trait corrupt  

---

## 18. Security notes (thin)

- Magic links single-use, 15m, rate limits  
- Session revoke  
- Visit tokens unguessable; revoke immediate  
- CSRF on cookie sessions  
- Export links expiring signed URLs  
- Standard OWASP API hygiene  

---

## 19. Document collapse of “what not to build” into eng checklist

Banned in codebase & copy:

- streak, achievement, badge, XP, level-up, leaderboard, feed, follow  
- hunger, health bar, die/death for birds  
- welcome back toast components  
- stats panels listing boldness etc.  
- recorded bird call assets as product path  

---

## 20. Success definition (v1)

A user opens a tab; somewhere mid-preen a bird notices them without a banner; they sit; presence quietly accrues; next week instruments see drift; in three weeks they feel Pip is bolder without a number; phone at night matches laptop morning; muted users still get captions and prose that feel like the same place; reduced-motion users get a calmer aviary that is still alive; neglect is quiet ambient, never guilt. Engineering is correct when those sentences are properties of the system, not marketing.

---

*End of plan. No product implementation in this phase.*
