# Pocket Aviary — V1 Implementation Plan

## 1. Scope

### 1.1 In scope (v1)

- Browser-only SPA (last two major versions of Chrome, Safari, Firefox, Edge)
- Single-user accounts via email magic link; one canonical aviary per account
- Multi-device sync via server-canonical state (no client authorship of personality)
- Bird engine: personality vectors, monotonic drift, mood, procedural calls, idle motion, bird-to-bird interaction
- Two starter birds at account creation; species system selected (no catalog); user naming/renaming
- Aviary growth to max **7** birds by **aviary age** (not engagement metrics)
- Interactions: return-greeting, presence accounting, listen-in, offer (seed / song fragment / still pool), settle, field notebook (read-only, sparse naturalist entries)
- Layout: single horizontal scene, three perch zones, local-time day/night, rare ambient weather, top-bar chrome that fades on idle
- Optional quiet visits: per-invite email invitations, read-only render, no co-presence, revocable, OFF by default
- Accessibility as designed surfaces: screen-reader naturalist narration, reduced-motion cross-fade mode, call captions, WCAG AA chrome, full keyboard nav
- Performance budgets: JS bundle ≤2MB gzipped, time-to-first-bird <500ms (mid-tier 4G), 60fps idle on 5-year-old laptop, no memory growth over 30 min
- Account export (JSON), soft-delete 30 days then hard-delete, session list/revoke, privacy-bounded telemetry

### 1.2 Explicit non-goals (do not plan, scaffold, or leave hooks that encourage)

- Native iOS/Android apps
- Gamification of any kind (streaks, badges, levels, XP, visit calendars, counters)
- Tamagotchi mechanics (hunger, death, distress, decay meters, negative drift on neglect)
- Social network surfaces (profiles, follows, discovery, comments, leaderboards, public aviaries)
- Shared/multi-user aviaries, multi-aviary accounts, customizable scenes
- Payments / tiers
- Push/email notifications about aviary state (visit notifications only as opt-in settings toggle, off by default)
- Exposing personality vector numbers anywhere
- Last-write-wins personality sync; client-owned simulation
- Recorded-audio fallback for calls

### 1.3 Defensible product calls (ambiguities resolved)

| Ambiguity | Decision |
|-----------|----------|
| Presence activity window | **180s** without pointermove/keypress ends presence; calibrate 120–240s in dogfood |
| Simulation tick cadence | **60s** nominal; allow 45–90s under load with jitter |
| Personality trait range | Continuous **[0.0, 1.0]**; starters seeded ~U(0.35, 0.55) with species biases |
| Mood enum | `wary`, `content`, `curious`, `drowsy`, `alert`, `settled` (settled = post-settle / deep night soft state) |
| Offer cooldown | **4 minutes** per bird per offer type |
| Notebook sparsity | Target **~2–4 entries/week** active aviary; hard cap 1/day unless anomaly event |
| Third bird unlock | Aviary age **≥ 45 days**; 4th ~120d, 5th ~210d, 6th ~300d, 7th ~400d (tunable; age-only) |
| Species pool size | **6** species with distinct silhouette + motif library |
| Client stack | TypeScript, Vite, Canvas 2D primary render (SVG assets where useful), WebAudio, no heavy 3D engine |
| Backend | Single deployable service initially (API + tick worker + mail) with clear module boundaries; Postgres primary store |
| Auth session TTL | Refreshable session **90 days** device token; magic link **15 min**, single-use |
| Visit invite expiry | **30 days** unused |

---

## 2. Architecture

### 2.1 High-level shape

```
┌─────────────────────────────────────────────────────────────┐
│ Browser client (SPA)                                        │
│  Render loop │ WebAudio chorus │ Presence probe │ UI chrome │
│  Event outbox → HTTPS  │  Snapshot pull / SSE or long-poll  │
└────────────────────────────┬────────────────────────────────┘
                             │ TLS
┌────────────────────────────▼────────────────────────────────┐
│ Edge / CDN: static assets + bootstrap HTML                   │
│ Optional: cacheable bootstrap stub (no PII)                  │
└────────────────────────────┬────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────┐
│ API service                                                 │
│  Auth │ Aviary snapshot │ Event ingest │ Visits │ Accounts  │
│  Narration helpers │ Export │ Settings                      │
└──────────────┬───────────────────────────┬──────────────────┘
               │                           │
        ┌──────▼──────┐             ┌──────▼──────────────┐
        │ PostgreSQL  │             │ Tick worker         │
        │ accounts,   │◄────────────│ every ~60s per      │
        │ birds,      │  exclusive  │ active partition    │
        │ events,     │  writer of  │ reads event log,    │
        │ snapshots   │  personality│ updates vectors/mood│
        └─────────────┘             │ notebook candidates │
                                    └─────────────────────┘
               │
        ┌──────▼──────┐
        │ Mail (magic │
        │ links,      │
        │ invites,    │
        │ export URL) │
        └─────────────┘
```

**Invariant:** Only the tick worker mutates `personality_vector` and authoritative mood transitions. Clients append **interaction events** only. Snapshots are read models.

### 2.2 Client/server split

| Concern | Owner |
|---------|--------|
| Personality vector storage & drift | Server tick |
| Mood authoritative state | Server tick (+ light client prediction for snappy UX, discarded on snapshot) |
| Presence trust boundary | Client measures; server accepts pings but **rate-limits and de-dupes**; drift uses bounded correctness not perfect honesty |
| Call synthesis & mix | Client |
| Idle micro-motion drawings | Client (seeded by snapshot constraints) |
| Ambient leaf/feather ornaments | Client-only, no server state |
| Field notebook text | Server-generated, stored, client displays |
| Visit view | Same snapshot API with visitor token; interaction endpoints rejected |
| Day/night palette | Client from user local timezone (server stores `timezone` on account, used for night mood transitions in tick) |

### 2.3 Render pipeline boundary

1. **Network snapshot** → normalized `AviaryViewModel`
2. **Simulation interpolator** (client) advances pose parameters between server pose anchors
3. **Scene compositor** layers: sky → background foliage → perches/birds midplane → foreground accents → optional captions → top bar overlay
4. **Audio graph** consumes the same attention/focus state as compositor (listen-in IDs)
5. **A11y narrator** consumes throttled prose from server and local event hints

Rendering **must pause** (or throttle to 1fps) when `document.visibilityState !== 'visible'` to save battery; simulation **never** depends on client frames.

### 2.4 Service modules (logical packages)

Even if one binary:

- `auth` — magic link, sessions, device revoke
- `aviary` — snapshot assembly
- `events` — append-only ingest + validation
- `sim` — tick, drift, mood, bird-to-bird, weather scheduling
- `notebook` — sparse naturalist generation
- `visits` — invite lifecycle
- `accounts` — export, delete, settings, timezone
- `telemetry` — aggregate-only metrics emission
- `mail`

### 2.5 Data store strategy

- **Postgres** as system of record
- Account ID = UUID v4ALWAYS; email never a FK or log key
- Append-only `interaction_events` (partition by month optional later)
- `birds` row holds current personality JSONB + mood + perch zone + pose seed
- `aviary_state` denormalized snapshot fields for fast GET
- Advisory locks or `FOR UPDATE SKIP LOCKED` per aviary during tick
- Soft-delete flag on accounts with purge job at day 31

---

## 3. Data model

### 3.1 Core entities

**Account**

```
id: UUID (synthetic, PK)
email_ciphertext / email_hash_for_lookup
email_verified_at
timezone: IANA string
created_at
marked_for_deletion_at: nullable
settings: JSONB {
  visit_notifications: false,
  captions_default: false,
  reduced_motion_override: null | true | false,
  audio_muted: false
}
```

**Session**

```
id: UUID
account_id
token_hash
device_label
created_at, last_seen_at, revoked_at
```

**Aviary**

```
id: UUID
account_id (1:1)
created_at          -- drives age-based bird unlocks
bird_cap: 7
weather_state: { type, started_at, ends_at } | null
settled_until: timestamptz | null  -- client settle is session-local; server optional soft flag
last_tick_at
version: bigint     -- monotonic snapshot version
```

**Bird**

```
id: UUID (stable identity forever)
aviary_id
species_id
name: string (user-facing, mutable)
display_order: int
personality: {
  boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity
}  -- each float 0..1
mood: enum
mood_updated_at
perch_zone: front | middle | back
pose: { kind, phase, seed }  -- for mid-action first frame
call_grammar_seed: int
adopted_at
```

**InteractionEvent** (append-only)

```
id: bigserial / UUID
aviary_id
account_id
bird_id: nullable
type: enum [
  presence_ping,
  listen_in_start, listen_in_end,
  offer_seed, offer_song, offer_pool,
  settle, settle_undo,
  session_open, session_close,
  rename_bird,  -- administrative, not drift
  ...
]
payload: JSONB   -- durations, offer target, client timestamps
client_event_id: UUID  -- idempotency
received_at
processed_at: nullable
```

**NotebookEntry**

```
id: UUID
aviary_id
created_at
prose: text   -- naturalist lowercase
salience: float
source_refs: JSONB  -- internal event/bird ids, never user-visible as stats
```

**VisitInvite**

```
id: UUID
host_account_id
aviary_id
visitor_email_ciphertext
token_hash
created_at, expires_at, revoked_at, accepted_at
```

**VisitSession / VisitLog**

```
id, invite_id, started_at, ended_at, approx_duration_s
-- no presence events into host drift
```

**MagicLink**

```
id, account_id or email_pending, token_hash, expires_at, consumed_at
```

### 3.2 Personality & mood semantics

- Personality: five scalars, server-only mutation via additive deltas.
- **Monotonic expressive drift:** positive presence/interaction may increase traits toward 1.0; neglect does **not** decrease traits. Ambient quietness emerges from **behavior policy** (greeting rate, approach probability) conditioned on recent presence history and trait levels—not trait decay.
- Mood: fast timescale; transitions on tick; persists across sessions; never snap to neutral on open.
- Dual clock: design tests assert week-scale instrument drift and multi-week user-visible behavior deltas.

### 3.3 Presence model (data)

Store rolling windows:

```
presence_windows(aviary_id, started_at, ended_at, quality_score)
```

Built from presence_ping events with server-side max deficit (e.g. cannot claim > wall-clock, max 45s credit per ping interval).

### 3.4 Never exposed fields

No API for clients returns raw personality numbers except **account export** JSON (user owns their data for portability; v1 export includes vectors because PRD requires export of personality vectors—export is settings-only, downloaded privately, not live UI).

---

## 4. API surface

All authenticated routes use session cookie/`Authorization: Bearer` device token. IDs in URLs are UUIDs. Matter-of-fact error bodies for auth/system; no naturalist prose on errors.

### 4.1 Auth

| Method | Path | Notes |
|--------|------|-------|
| POST | `/v1/auth/magic-link` | `{ email }` → 202 always (anti-enum); rate limit |
| GET | `/v1/auth/callback?token=` | consume link → set session → redirect SPA |
| POST | `/v1/auth/logout` | revoke current |
| GET | `/v1/account/sessions` | list devices |
| DELETE | `/v1/account/sessions/:id` | revoke |
| POST | `/v1/account/email-change` | start verify |
| POST | `/v1/account/delete` | soft delete |
| POST | `/v1/account/undelete` | within 30d |
| GET | `/v1/account/export` | enqueue export email |

### 4.2 Aviary state

| Method | Path | Notes |
|--------|------|-------|
| GET | `/v1/aviary` | canonical snapshot for host |
| GET | `/v1/aviary/stream` | optional SSE of version bumps + light diffs |
| GET | `/v1/bootstrap` | minimal snapshot for first paint (edge-friendly) |

**Snapshot payload (illustrative):**

```json
{
  "version": 184422,
  "server_time": "ISO",
  "aviary": {
    "created_at": "...",
    "weather": null,
    "unlock": { "next_bird_at": "...", "bird_count": 2, "cap": 7 }
  },
  "birds": [
    {
      "id": "...",
      "species_id": "warbler_a",
      "name": "pip",
      "mood": "content",
      "perch_zone": "front",
      "pose": { "kind": "preen", "phase": 0.42, "seed": 91 },
      "visual": { "plumage_hint": "rich" },
      "call": { "signature_id": "...", "vocal_bias": "med" }
    }
  ],
  "ambient": { "day_phase": "morning" },
  "greeting": {
    "primary_bird_id": "...",
    "absence_bucket": "short|medium|long",
    "style_seed": 17
  }
}
```

Notes:

- No personality floats in live snapshot (map follow-on visuals via discrete hints if needed: plumage_hint tiers only).
- `greeting` computed server-side on session_open / first pull after absence using boldness, mood, `last_presence_end`.

### 4.3 Interaction events

| Method | Path | Notes |
|--------|------|-------|
| POST | `/v1/events` | batch append `{ events: [...] }` idempotent by `client_event_id` |

Validated types only. Reject visitor tokens. Reject events for foreign birds.

### 4.4 Notebook

| Method | Path | Notes |
|--------|------|-------|
| GET | `/v1/notebook?cursor=` | infinite scroll reverse chrono |
| — | no write/delete/annotate |

### 4.5 Offers / settle (also expressible purely as events)

Prefer events API; convenience endpoints optional:

- `POST /v1/interact/offer` `{ type, near_bird_id? }`
- `POST /v1/interact/settle`
- `POST /v1/interact/settle/undo` within 5s client-enforced; server accepts short window

Server enforces offer cooldowns.

### 4.6 Visits

| Method | Path | Notes |
|--------|------|-------|
| POST | `/v1/visits/invites` | host: `{ email }` |
| GET | `/v1/visits/invites` | outstanding |
| DELETE | `/v1/visits/invites/:id` | revoke immediate |
| GET | `/v1/visits/log` | who/when/duration |
| GET | `/v1/visit/:token` | visitor snapshot (read-only) |
| POST | `/v1/visit/:token/heartbeat` | duration only; **no** presence into host drift |

Visitor GET returns snapshot without checklist of host PII beyond aviary. On revoke: `410` matter-of-fact body.

### 4.7 Accessibility helpers

| Method | Path | Notes |
|--------|------|-------|
| GET | `/v1/narration` | current prose block + next cadence hint |
| GET | `/v1/captions/lexicon` | optional motif→phrase templates versioned |

Narration preferably assembled server-side from same state as snapshot to keep voice consistency; client may ask for refresh on offer/settle/greeting.

### 4.8 Versioning & errors

- Prefix `/v1`
- `409` / `401` / `410` with `{ "message": "..." }` matter-of-fact
- Snapshot `ETag` / `If-None-Match` for cheap keepalives

---

## 5. Simulation engine design

### 5.1 Tick loop

Cadence ~ **once per minute** per aviary (partition workers; skip aviaries with no unprocessed events **and** no time-driven work due—still need periodic mood/time-of-day transitions, so schedule **minimum tick** every N minutes or on time bucket change).

**Per tick steps (ordered):**

1. Load aviary + birds `FOR UPDATE`
2. Read unprocessed events since `last_processed_event_id` **in order**
3. Collapse presence pings → presence-seconds (clamped)
4. Apply interaction effects to **pending drift deltas** and **mood attractors**
5. Advance ambient weather schedule (rare rain/wind RNG with weekly expected rates)
6. Time-of-day attractors using account timezone
7. Bird-to-bird contagion (wary spread, chorus windows)
8. Apply **clamped additive** personality deltas (only ≥0)
9. Commit mood transitions (Markov-like with continuous hazards)
10. Update perch zone probabilities → snap/hold zones (not continuous world coords server-side)
11. Update pose seeds if long-idle server-side
12. Possibly enqueue notebook candidate if rarity gates pass
13. Bump `version`, `last_tick_at`, mark events processed
14. Publish version for SSE/pollers

### 5.2 Drift function

Implementation model: **leaky integrator of attention signals**, slow time constant.

Inputs (weights initial, tune against metrics):

| Signal | Relative weight | Primary traits |
|--------|-----------------|----------------|
| Presence-seconds | 1.0 | all toward expressive, mild |
| Listen-in duration on bird | 0.35 | social_warmth, vocal_frequency |
| Offer nearby accepted approach | 0.15 | curiosity |
| Offer near bird (attempt) | 0.08 | boldness |
| Settle | 0 | mood quiet only |

**Form (per trait):**

```
delta = saturation(gain * signal * trait_openness(trait))
trait = min(1.0, trait + delta)
```

`trait_openness = (1 - trait)^k` so late progress slows (avoids instant max).

**Calibration targets:**

- Instrumented: after **7 days regular visits** (~30–45 min presence total / week), detect trait Δ ≥ ε (~0.01–0.03) in CI sims
- User-visible: **~3 weeks** posture/greeting/chorus differences without numbers
- Hard rule: batch tests assert **no negative Δ** from pure absence ticks

**Ambient quiet on neglect (behavior, not trait decay):**

Maintain `recent_presence_score` decay on absence. Greeting probability & approach speed multiply by `f(score, social_warmth, boldness)`. User returns after 2 weeks → quieter greets, not greyer plumage down-drift.

### 5.3 Mood transitions

States: wary, content, curious, drowsy, alert, settled.

Drivers:

- Session interaction residuals (offer accept → content/curious)
- Local hour (morning → alert; dusk → drowsy; night → settled for most)
- Weather short pulses
- Personality modifiers (high boldness reduces wary entry rate)
- Contagion from neighbors' alarm

Mood is **authoritative on server**; client may show smoothed animation of transition.

Persist through tab close.

### 5.4 Call-grammar runtime

**Server** stores species motif library IDs + per-bird `call_grammar_seed` + vocal_frequency.

**Client synthesizer:**

- Motif graphs: notes (freq envelopes, FM/AM, noise bursts, silences)
- Variation: seed + timestamp → never bit-identical twice
- Timing: poisson-like waits scaled by vocal_frequency and mood
- Recognizability: species template fixed intervals + signature pitch ratio; mood scales tempo/intensity not identity
- Bird-to-bird: client listens to coarse chorus windows from snapshot (`chorus_window_open`); if bird A packages a call event, neighbors roll response chance

Audio **never** required buckets of recorded samples.

### 5.5 Idle motion & pose

Server: timezone + mood → preferred perch zone + pose family.

Client: continuous micro-motion FSM (preen, scan, shuffle, headtilt, fluff) with reduced-motion alternate (cross-fade keyposes).

First frame: use snapshot `pose.phase` so mid-action—**no entry animation**.

### 5.6 Adoption & species

- 6 species, coherent biome set
- First two auto-assigned with complementary motif families (diversity without choice UI)
- Additional birds age-gated offers (ambient card in top bar naturalist voice, not gamified "unlock!")
- Stable bird UUID immutable across renames

### 5.7 Notebook generator

Rules engine + templates → optional light LLM deferred; **v1 template compositional** for reliability:

- Facts: which bird greeted first today, long quiet stretches, weather, perch patterns
- Filters: no user streak language; no numeric traits; no "you visited"
- Sparsity gate: max frequency + novelty score
- Voice: lowercase, present-tense naturalist

---

## 6. Sync model

### 6.1 Canonical state

One aviary record + birds + version. Multiple devices issue GET snapshot independently.

### 6.2 Client write path

1. Local UI instant feedback (listen-in mix, offer fly animation)
2. Event enqueued offline-capable outbox (IndexedDB)
3. POST `/v1/events` with idempotency keys
4. Tick incorporates; next snapshot reflects

### 6.3 Conflict prevention

- **No LWW on personality** — impossible path; clients cannot PATCH vectors
- Compost concurrent listen-ins from two devices: both events append; tick applies both with diminishing returns
- Settle is session soft state; two devices don't fight hard global settle—prefer **per-client visual settle** with optional signaling event (mood quiet) rate-limited
- Magic link replay: token single-use; session list for revoke
- Clock skew: server `received_at` orders processing; client timestamps advisory only

### 6.4 Pull cadence

- On open
- On `visibilitychange` → visible
- On long rAF gap (resume from sleep)
- Keepalive every ~30–60s while present
- Optional SSE on version change for near-live multi-device pose convergence

### 6.5 Offline / degraded

- If snapshot fails: quiet field loading (soft sky), never spinner-as-brand
- Matter-of-fact error after timeout with reload CTA
- Outbox flushes when online; no local personality simulation inventing drift

### 6.6 Multi-device coherence story for QA

Test matrix: laptop morning presence → phone night open must show drifted traits continuous (instrument export/debug channel internal only) and mood advanced by night hours.

---

## 7. Frontend rendering pipeline

### 7.1 Stack recommendation

- TypeScript + Vite
- Canvas 2D scene (predictable 60fps path) with SVG sprites for birds/perches rasterized to atlas
- CSS for top bar / settings / notebook panel only
- Route shell: `/` aviary, `/settings`, `/auth`, `/visit/:token`
- Code-split settings, visits, account heavy paths

### 7.2 Scene composition

Layer order:

1. Sky gradient (time-based palette)
2. Far foliage parallax (subtle)
3. Perches: back, mid, front zones with fixed layout geometry responsive to viewport width
4. Birds z-sorted by zone + subtle y offsets
5. Weather particles (rain few lines; wind leaf sway)
6. Foreground branch ornament
7. Client-only leaf/feather drift sprites
8. Call captions near bird (if enabled)
9. Top bar (opacity fade)

**No** in-scene buttons, badges, tooltips, drag handles.

### 7.3 Responsive rules

- Always keep all birds on-screen; compress spacing on narrow viewports
- No pan/zoom/scroll of scene
- Safe areas for mobile notches; top bar height constant

### 7.4 First paint protocol ("already in motion")

1. Inline critical CSS sky color in HTML
2. Show quiet field immediately
3. Parallel: bootstrap JS + auth cookie + early `/v1/bootstrap` snapshot (small)
4. Draw birds at pose.phase without fade-from-zero alpha pop if possible; soft opacity only if assets late
5. Start rAF loop + audio context resume on first gesture (browser autoplay policy)—ambient may wait for gesture; visual must not

**Forbidden:** logo splash, gamified loader, "Welcome", confetti.

### 7.5 Idle micro-motion

- Continuous FSM per bird
- Mood → weight table of behaviors
- Personality → rates (curiosity headtilt frequency, boldness front travel)
- Random walk in micro-parameters better than looped GIFs

### 7.6 Transitions

- Perch change: short hop arc (full motion) or cross-fade (reduced motion)
- Listen-in focus: gentle visual weight (slight scale/contrast)—not selection chrome boxes except a11y focus ring when keyboard

### 7.7 Reduced-motion mode

Triggers: `prefers-reduced-motion: reduce` OR settings override.

- Replace frame animation with **slow cross-fades between authored still poses**
- Remove leaf drift / rain shimmer intensity
- Keep day/night slow color morph
- Keep audio or captions
- Mode is equal product quality, not muted broken checkbox

### 7.8 Top bar

Icons only: offer, notebook, accessibility, account.

- After **~3s** pointer idle: opacity → ~0.15
- Pointer/keyboard activity: restore
- System dialogs use matter-of-fact copy and normal casing

### 7.9 Empty aviary (once)

After naming starters: quiet field → soft fly-in once → never empty again.

---

## 8. Audio pipeline

### 8.1 Graph structure

```
MotifVoices[ bird ] → BirdGain → ListenInBus
                              ↘ ChorusBus → MasterGain → destination
SharedAmbientBus (very low)
```

- Listen-in: ramp focused bird gain up (~800–1500ms), others down (~same) but **floor > 0**
- Disengage: reverse ramps
- Mute setting: silence master; auto-enable captions when muted if user hasn't disabled permanently

### 8.2 Procedural synthesis

- WebAudio oscillators + noise buffers + filters + envelopes
- Motif DSL (JSON): sequenced nodes with jitter ranges
- Parameterization by mood (amplitude, interval scale) & vocal_frequency (event rate)
- Buffer pooling to meet no-memory-growth budget
- Worker optional for motif scheduling; rendering remains main/audio thread carefully

### 8.3 Chorus

- Independent schedulers per bird; soft phase avoidance rather than forced quantization
- Response calls scheduled with delay distributions based on social_warmth

### 8.4 Captions

- Generate short naturalist phrase from motif features at play time (`"a soft three-note rise"`)
- Position near bird; fade with amplitude envelope
- Match actually synthesized call

### 8.5 WebAudio fallback

If AudioContext unavailable/failed:

- Full quiet visuals
- Captions **on by default**
- No recorded MP3 fallback ever

Browser gesture gate: after first user gesture, resume context; before that rely on visual aliveness.

### 8.6 Audio uncanniness QA

- Checklist: no identical repeats within session window; two-bird mix not phase-canceled drones; species recognizable blind test

---

## 9. Accessibility surfaces

### 9.1 Screen-reader narration

- Live region (`aria-live="polite"`) updated **every 30–60s** idle
- Faster on greeting, offer outcome, settle
- Prose naturalist example style; never "mood: content"
- Bird focus: accessible names = user bird names + short species gloss if needed
- Separate notebook list semantics as log of observations

### 9.2 Keyboard

- Tab: top bar controls → into scene first bird
- Arrows: cycle birds
- Enter: listen-in toggle
- Escape: exit listen-in / close panels
- Offer & settle fully operable without pointer
- Visible focus ring designed for bright day and dim night palettes

### 9.3 Captions & mute

As above; dual channel with reduced motion.

### 9.4 Contrast

- WCAG AA minimum on all chrome/text/captions
- Scene itself may be low-contrast nearest nature; don't place essential UI on busy foliage without scrim

### 9.5 Accessibility ships day one

No deferred a11y milestone—same release train as visual aviary.

---

## 10. Performance budgets and observability

### 10.1 Budgets (gates in CI)

| Metric | Budget |
|--------|--------|
| Initial JS gzipped | **< 2MB** (target ≤1.2MB ambitious) |
| Time to first bird visible | **< 500ms** mid-tier 4G synthetic |
| Idle FPS | **60fps** sustained, 5-year-old laptop profile |
| Memory | **flat RSS** over 30 min scripted session |
| Snapshot size | few KB |
| Tick p99 | alarm **> 5s** |

### 10.2 Engineering tactics

- Aggressive code-split non-aviary routes
- Procedural audio/visuals beat asset packs
- Texture atlases; no multi-MB PNGs
- Pause render when hidden
- Object pools for particles and audio nodes
- Avoid retaining notebook DOM nodes for whole history (virtualize list)

### 10.3 Observability (aggregate only)

Allowed:

- request rates, latencies, error rates
- tick latency histograms
- RUM: TTFB, first-bird paint, long tasks, audio context errors
- anonymized session duration histograms **without account digests that re-identify**

Forbidden in telemetry pipelines:

- per-bird personality
- offers/listen-ins joinable to accounts for analytics warehouse
- presence timelines into ML training

Architecture: simulation DB **not** CDC'd into warehouse. Metrics emitters only counters/histograms from API/tick.

Synthetic fleet: scripted browsers from multiple geos drive first-bird and FPS checks.

### 10.4 Deliberately not measured as product KPIs

- DAU streaks
- conversion on offers spam
- "engagement score" leaderboards

Instrument health, not exploitation.

---

## 11. Interactions implementation notes

### 11.1 Return-greeting

Server bundles invitation on session_open:

- Choose bird ranking by boldness * mood approachability * RNG
- Absence bucket from last presence end
- Stagger secondary greets with random 400–1800ms offsets if multi
- Client plays motion+call variation from `style_seed`
- **Zero** text welcome surfaces

### 11.2 Presence probe (client)

All three must hold:

1. `visibilityState === 'visible'`
2. `document.hasFocus()`
3. pointermove or keypress within **180s**

Emit presence_ping every ~15–20s while conditions hold. End on settle or tab close/hide/blur/timeout.

### 11.3 Listen-in

Pointer click/tap or keyboard Enter. Second activate or empty click or blur exits. Audio + optional visual weight only—no "selected" badge language.

### 11.4 Offer

Top-bar menu: seed / song fragment / still pool.

Cooldowns per bird per type. Reactions by mood+curiosity; animations server-authorized via resulting event/mood.

### 11.5 Settle

Top-bar. 2–4s lighting → evening, calls quiet. 5s undo click-anywhere. Presence ends. Tab close without settle OK.

### 11.6 Field notebook

Sparse prose entries only; infinite history scroll; read-only.

---

## 12. Social (visits) implementation

- Default OFF (no invites until host acts)
- Host emails visitor; 30d unused expire
- Visitor token scopes read-only snapshot + heartbeat duration log
- Host log in settings: email, time, duration; no badge pings
- Visit notification email toggle **off by default**
- No chat, avatars, comments, discovery, leaderboards, show-off render path
- Revoke → next visitor pull `410`
- Visitor presence **must not** write host presence events

---

## 13. Accounts, privacy, compliance engineering

- Synthetic UUID everywhere except encrypted email column + blind index hash for login
- Magic link 15m, single consume
- Export JSON: birds, names, vectors, moods, notebook, settings—emailed download link
- Soft delete 30d → hard purge including events
- Privacy policy link in settings: names aggregates; excludes per-bird interaction reuse
- Per-bird events sole purpose: that user's tick

---

## 14. Rollout

### 14.1 Phased delivery (still one quality bar; a11y not deferred)

**M0 — Foundations**

- Auth magic link, account UUID model, empty quiet field shell
- Snapshot + event log scaffolding, tick skeleton
- CI perf harness shell

**M1 — One living bird**

- Species v1 visuals, procedural call MVP, presence + drift unit sims
- Greeting + idle motion + day/night palette
- Reduced-motion + focus ring + first live region

**M2 — Companion pair**

- Two starters, bird-to-bird calls, listen-in mix, offers + cooldown
- Notebook generator v1 templates
- Memory/FPS soak tests green

**M3 — Product completeness**

- Settle, weather, notebook polish, account export/delete, sessions revoke
- Captions default paths, narration cadence polish
- Visit invites end-to-end
- Bundle/TTFB budgets locked in CI

**M4 — Hardening**

- Drift calibration against 3-week sims
- Multi-device soak, chaos on tick lag
- Accessibility UX dogfood with keyboard + VoiceOver/NVDA
- Soft launch cohort

### 14.2 Birds-per-aviary ramp

- Launch: 2 starters only
- Unlock timeline by aviary age only (section 1.3)
- Never unlock via pay/engagement score
- Hold hard cap 7

### 14.3 Day-one instrumentation

- Tick p99, event ingest errors
- first_bird_paint RUM
- audio_context_failures
- auth magic-link delivery success rate (ops)
- visit revoke correctness sample

No dashboards of average boldness across users.

### 14.4 Content ops

- 6 species motif libraries + pose still sets for reduced motion
- Naturalist string banks reviewed for voice + non-gamification language

---

## 15. Testing strategy

### 15.1 Simulation

- Deterministic seed harness reruns weeks of time compression
- Asserts: monotone personality; calibration envelopes; no residual presence while hidden
- Mood continuity across “sessions”

### 15.2 Client

- Visual regression on palette phases (not every feather)
- Reduced-motion goldens
- Keyboard path e2e
- Audio: mock AudioContext; ensure caps captions fallback
- Memory soak playwright 30 min

### 15.3 Sync

- Two clients event race money tests prove no vector clobber
- Visitor cannot POST host events (authz)

### 15.4 Voice lint

- Script fails CI if UI strings match banned patterns: `Welcome back`, `achievement`, `streak`, `level up`, `daily reward`, etc.

---

## 16. Risks and mitigations

| Risk | Why it matters | Mitigation |
|------|----------------|------------|
| Drift too fast | Tamagotchi feel | Whole-week time simulations; gate boosts; diminishing returns near 1.0 |
| Drift too slow | Screensaver feel | Instrument ε after 7d regular; tune presence gain safely low |
| Presence definition too loose | Background tabs inflate drift | Prevent pings without focus+activity; server clamps |
| Presence too strict | Still watchers under-credited | 180s activity window; dogfood longer |
| Sync bugs / dual simulation temptation | Lost relationship history | Hard ban client vector writes; codeowners on sim package |
| LWW regressions | Silent drift loss | Only additive tick patches; no bird PUT of personality |
| Audio uncanny / looped feel | Product fails headline aliveness | Procedural only; peer review blind tests; no sample packs |
| Bundle/TTFB slip | Loading destroys "already in motion" | CI size gates; bootstrap path; quiet field only |
| A11y as checklist | SR users get inferior product | Narration & reduced-motion as features in M1+; a11y dogfood release gate |
| Gamification PR creep | Concept poison | Shared banned-pattern lint; non_goals as eng return rubric |
| Visit scope creep | Social network | Visits package owner; reject discovery tickets by default |
| Notebook generic logs | Voice collapse | Template review; sparsity metrics |
| Weather/assertive FX | Attention steal | Cap frequency; low amplitude |
| Autoplay audio blocked | Silent first seconds | Visual greets first; captions; resume audio on gesture without toast spam |
| Tick backlog | Stale multi-hour catchup spikes | Chunked catchup; p99 alarm 5s; prioritize time-of-day over micro events |
| Email as ID leakage | PII sprawl | UUID only; log scrubbing tests |
| Personality in UI debug | Numbers ruin relationship | No debug panel in prod builds; export-only vectors |

---

## 17. Suggested repository layout (implementation guide)

```
/apps/web          # SPA
/apps/api          # HTTP + worker entry
/packages/sim      # pure TS/Go drift & mood (shared tests)
/packages/callgram # motif schemas + caption phrases
/packages/voice    # naturalist string helpers + banlist
/infra             # terraform, CI
```

Shared sim package runs in worker and in CI accelerated clocks.

---

## 18. Definition of done (v1)

- Two living birds, age path to seven, stable identity
- Server tick anytime continuity across devices
- Presence-honest monotonic drift meeting 1-week instrument / ~3-week feel targets
- Listen-in, offer, settle, notebook, greeting with **no** announcement chrome
- Procedural chorus + WebAudio silence+captions fallback
- Visits optional quiet complete
- A11y: narration, reduced-motion, captions, keyboard, AA contrast
- Budgets held under synthetic mid-tier conditions
- Privacy isolation of sim data from analytics
- Explicit rejection of gamification / Tamagotchi / native / social-network scopes

---

## 19. What this plan deliberately does not do

- Implement application code
- Specify every hex color (design system companion)
- Choose final hosting vendor
- Introduce ML training on user birds
- Add achievement systems “behind a flag”

This document is the executable engineering blueprint for a frontier team to build Pocket Aviary v1 without diluting the PRD’s affective core.
