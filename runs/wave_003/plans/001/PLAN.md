# Pocket Aviary — v1 implementation plan

This plan is the executable translation of the PRD. It does not restate the product; it names services, schemas, APIs, algorithms, render/audio contracts, tests, and rollout gates a separate team can build against. Where the PRD is underspecified, this plan makes a defensible call and marks it as a **decision**.

---

## 1. Scope

### 1.1 In v1

- Browser-only SPA + API. Last two major versions of Chrome, Safari, Firefox, Edge.
- Single-user accounts. Email magic-link auth. One canonical aviary per account.
- Two starter birds at account creation; hard cap of seven birds. Additional birds offered by aviary age, not engagement.
- Server-side simulation tick (~60s). Client renders snapshots and interpolates; never ticks.
- Personality vectors (boldness, social warmth, vocal frequency, plumage saturation, curiosity), mood, procedural calls, mood-shaped idle motion, bird-to-bird chorus/spread.
- Presence accounting (visibility ∧ focus ∧ recent pointer/key). Presence is the dominant drift input.
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle (with 5s undo), field notebook (read-only, sparse).
- Visit invitations: off by default, per-invite email, read-only ambient, revocable, 30-day unused expiry. Silent visit log. Optional visit-email toggle off by default.
- Accessibility ships with v1: naturalist screen-reader narration, designed reduced-motion mode, runtime call captions, WCAG AA chrome, full keyboard path.
- Account export (JSON emailed as download link), session list + revoke, email change with verify-before-commit, soft-delete 30 days then hard-delete.
- Aggregate operational telemetry only. No per-bird / per-account interaction analytics.

### 1.2 Out of v1 (non-goals, enforced)

- Native apps. Do not design protocols or the data model around native-client constraints.
- Gamification of any flavor: streaks, scores, badges, levels, visit calendars, “birds adopted” counters, XP, ranks, milestone celebrations. Not even as opt-in settings or notebook copy that observes the user’s visit frequency.
- Tamagotchi mechanics: death, hunger, distress, decaying happiness meters, negative personality drift on neglect.
- Social network surfaces: profiles, follows, public discovery, comments, chat, avatars, co-presence, leaderboards, mutual visits.
- Payments, multi-aviary accounts, shared/household aviaries, customizable scenes, push/SMS, SSO/passwords, recorded-audio fallback.
- Personality numbers anywhere a user can see them (no debug overlay in production, no stats panel, no ARIA exposure of vectors).

### 1.3 Decisions on underspecified items

| ID | Decision | Rationale |
|---|---|---|
| D1 | Simulation tick cadence = 60s, with catch-up ticks on snapshot pull if the last tick is >90s stale. | Matches “~once per minute”; catch-up keeps long-absence returns current without a visible snap. |
| D2 | Presence activity window = 4 minutes. | PRD says “few minutes, lean longer”; watching without moving is the product. |
| D3 | Presence pings every 20s while the conjunction holds; each ping is a 20s credit, not a heartbeat that implies continuous presence. | Prevents “tab open overnight” inflation if one condition flickers. |
| D4 | Personality scalars stored as `f32` in `[0.0, 1.0]`. Starter birds seeded in `[0.28, 0.52]` with species-biased offsets. | Hidden from users; range is small enough that three-week visible drift is ~0.06–0.10 on attended traits. |
| D5 | Mood enum: `wary`, `content`, `curious`, `drowsy`, `alert`. Night settled pose is `drowsy` plus a `settled` lighting flag on the aviary, not a sixth mood. | Matches PRD examples; keeps mood and lighting orthogonal. |
| D6 | Offer cooldown = 4 minutes per (bird, offer-kind). | “A few minutes”; long enough to stop curiosity saturation in one sitting. |
| D7 | New-bird offers at aviary age 45 / 120 / 210 / 320 / 450 days (third through seventh). | Age-only; months-to-year cadence; not visit-gated. |
| D8 | Species pool = 6: warbler, wren, sparrow, finch, dove, nightjar-analogue. Nightjar remains active at night. | Coherent backyard set; one nocturnal signature as specified. |
| D9 | Default name suggestions are species-keyed (e.g. Pip, Wren, Moss) but stored as user strings; identity is `bird_id` UUID. | Names never replace identity. |
| D10 | Quiet-field first paint is a static CSS sky + 1–2 CSS leaf ornaments; first bird draws as soon as snapshot + species silhouette arrive. | Protects “already in motion” without a spinner. |
| D11 | Visit notifications, if opted in, are email only (no push). | Product is not a notification surface; email is the only existing outbound channel. |
| D12 | Notebook generation runs inside the tick, target ~1 entry / 3–5 days of regular host presence, plus noteworthy-event overrides. | Sparse by construction. |
| D13 | Timezone is captured at signup from the browser and updated on each signed-in session start if it changed. Day/night and mood-of-day use this stored zone, not server UTC. | Local-time aviary. |
| D14 | Auth sessions last 90 days idle; listed and revocable. Magic link TTL 15 minutes, single use. | Calm, low-friction, revocable. |

---

## 2. Architecture

### 2.1 Service shape

Four deployable units. Keep them small; the load-bearing rule is **one writer of personality**.

```
┌─────────────┐     magic link      ┌──────────────┐
│   Web SPA   │◄───────────────────►│  edge / CDN  │  HTML + hashed JS/CSS
│  (browser)  │                     └──────────────┘
└──────┬──────┘
       │ HTTPS JSON + SSE (optional keepalive)
       ▼
┌──────────────────────────────────────────────┐
│  api-gateway                                 │
│  auth, rate limits, session cookies,         │
│  visit-token validation, request IDs         │
└──────┬───────────────────────────┬───────────┘
       │                           │
       ▼                           ▼
┌──────────────┐            ┌──────────────────┐
│  account-svc │            │  sim-svc         │
│  users,      │            │  tick worker,    │
│  sessions,   │            │  snapshot read,  │
│  invites,    │            │  event ingest,   │
│  export/del  │            │  notebook writer │
└──────────────┘            └──────────────────┘
       │                           │
       ▼                           ▼
┌──────────────┐            ┌──────────────────┐
│  account-db  │            │  sim-db          │
│  (PII vault) │            │  (no email)      │
└──────────────┘            └──────────────────┘
       │
       ▼
┌──────────────┐     ┌──────────────┐
│  mailer      │     │  ops-metrics │  Prometheus/OTel
│  (magic,     │     │  no sim-db   │  histograms only
│   export,    │     │  access      │
│   visit)     │     └──────────────┘
└──────────────┘
```

**Hard pipeline split:** `ops-metrics` and any warehouse **cannot** read `sim-db`. `account-db` email column is the only email storage. Every other table, log field, Kafka/queue key, and metric label uses `account_id` UUID.

### 2.2 Client / server split

| Concern | Owner | Client may |
|---|---|---|
| Personality vector | sim-svc tick only | Read derived *effects* (perch preference, call rate), never the numbers |
| Mood, perch zone, weather, settled flag, notebook | sim-svc | Read snapshot; interpolate motion |
| Interaction events, presence pings | client writes append-only | Never compute drift |
| Call audio | client WebAudio from snapshot + grammar seed | Never persist audio |
| Ambient leaves/feathers | client-only ornaments | No server state |
| Day/night palette | client, from snapshot `local_minutes` + timezone | — |
| Narration prose | generated from snapshot (see §9); cached ~30–60s | Not a second simulation |
| Visit presence | not recorded | Render-only client build flag |

### 2.3 Render pipeline boundary

```
snapshot (JSON, ~2–8 KB)
    → SceneGraph (perches, birds, weather, lighting)
        → MotionDirector (idle cycles, perch interpolation, greeting)
            → Renderer (Canvas2D or SVG+CSS; see §7)
            → AudioEngine (WebAudio graph)
            → A11yNarrator (live region)
            → CaptionLayer (optional)
```

The renderer never asks the server for a frame. It asks for state. If the tab is `hidden`, the renderer and audio context suspend; the server keeps ticking.

### 2.4 Suggested stack (decision D15)

- **SPA:** TypeScript, Vite, small custom scene (no game engine). Route-level code split for settings / visit / sign-in.
- **API:** TypeScript (or Go) HTTP service; same language as tick worker if it keeps one team.
- **DBs:** Postgres. Two logical databases (or two schemas with separate roles): `account` and `sim`.
- **Queue:** Postgres `LISTEN/NOTIFY` or a small job table is enough at v1 scale; no Kafka.
- **Mail:** transactional provider; templates are matter-of-fact.
- **CDN:** HTML document includes a signed, cacheable *bootstrap snapshot* cookie-or-header path for signed-in users when possible; otherwise first XHR after HTML.

---

## 3. Data model

All primary keys are UUIDv7 (time-sortable) except where noted. Email never appears in `sim-db`.

### 3.1 Account database

```
accounts
  id              uuid pk
  email_ciphertext bytea not null unique   -- encrypted at rest
  email_hash      bytea not null unique    -- keyed hash for lookup
  timezone        text not null            -- IANA
  created_at      timestamptz
  marked_for_deletion_at timestamptz null
  visit_notify_email boolean not null default false

sessions
  id              uuid pk
  account_id      uuid fk
  created_at      timestamptz
  last_seen_at    timestamptz
  user_agent      text
  revoked_at      timestamptz null

magic_links
  id              uuid pk
  account_id      uuid null               -- null until email resolved
  email_hash      bytea
  token_hash      bytea unique
  expires_at      timestamptz             -- created_at + 15m
  consumed_at     timestamptz null

email_change_requests
  id, account_id, new_email_ciphertext, new_email_hash,
  token_hash, expires_at, consumed_at

invites
  id              uuid pk
  host_account_id uuid fk
  visitor_email_ciphertext bytea
  visitor_email_hash bytea
  token_hash      bytea unique
  created_at      timestamptz
  expires_at      timestamptz             -- +30d if unused
  accepted_at     timestamptz null
  revoked_at      timestamptz null
  visitor_account_id uuid null            -- if they later sign up

visit_sessions
  id, invite_id, started_at, ended_at, approx_duration_s
  -- no presence pings, no interaction events

account_settings
  account_id pk
  reduced_motion_override  enum('system','on','off') default 'system'
  captions_on              boolean default false
  audio_muted              boolean default false
```

### 3.2 Simulation database

```
aviaries
  account_id      uuid pk                 -- 1:1 with account
  created_at      timestamptz             -- age clock for new-bird offers
  settled         boolean default false
  weather         enum('clear','rain','wind') default 'clear'
  weather_until   timestamptz null
  last_ticked_at  timestamptz
  tick_seq        bigint not null default 0
  pending_adoption_slot  int null         -- 3..7 when an age gate opens
  adoption_presented_at  timestamptz null

birds
  id              uuid pk                 -- stable identity, never recycled
  aviary_id       uuid                    -- = account_id
  species         enum(...)
  name            text
  adopted_at      timestamptz
  sort_index      smallint                -- 0..6, adoption order
  -- personality (server-only; never sent as named numbers)
  boldness        real not null
  social_warmth   real not null
  vocal_frequency real not null
  plumage_sat     real not null
  curiosity       real not null
  mood            enum('wary','content','curious','drowsy','alert')
  perch_zone      enum('front','middle','back')
  pose            text                    -- current idle pose id
  pose_t          real                    -- 0..1 phase
  call_phase      real                    -- grammar clock
  last_offer_at   jsonb                   -- {seed,song,pool} -> timestamptz

events                         -- append-only
  id              bigserial pk
  account_id      uuid not null
  bird_id         uuid null
  kind            text not null
  payload         jsonb not null
  client_event_id uuid not null           -- idempotency
  occurred_at     timestamptz not null    -- client clock, clamped
  ingested_at     timestamptz not null default now()
  consumed_tick   bigint null
  unique (account_id, client_event_id)

notebook_entries
  id              uuid pk
  account_id      uuid
  written_at      timestamptz
  body            text                    -- naturalist prose, stored
  trigger         text                    -- internal; not shown

adoption_log
  account_id, slot, offered_at, accepted_at, bird_id
```

**Event kinds:** `presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `unsettle`, `greeting_ack` (client reports which bird it rendered as greeter — informational, not a drift writer), `rename`, `adopt`, `tab_close` (best-effort `sendBeacon`).

### 3.3 Snapshot DTO (what the client is allowed to see)

The snapshot **must not** include raw personality scalars. It includes *derived, non-numeric* cues the renderer needs:

```ts
type Snapshot = {
  tick_seq: number
  server_now: string
  local_minutes: number          // 0..1439 in account timezone
  settled: boolean
  weather: 'clear' | 'rain' | 'wind'
  birds: Array<{
    id: string
    species: Species
    name: string
    mood: Mood
    perch_zone: 'front' | 'middle' | 'back'
    perch_slot: number           // which perch inside the zone
    pose: string
    pose_t: number
    plumage: { hue: number; sat_tier: 0|1|2|3 }  // quantized, not the trait
    call: {
      grammar_id: string
      seed: number               // per-call variation seed
      next_onset_ms: number      // relative
      vocal_tier: 0|1|2|3        // quantized rate, not the trait
    }
    greeting?: GreetingHint      // only on session-open snapshots
  }>
  ambient: { leaf_hint: boolean }
  notebook_latest_id?: string
  pending_adoption?: { slot: number }   // no “you earned this”
  visit?: { read_only: true }
}
```

`sat_tier` / `vocal_tier` are 4-bucket quantizations so the client can render without reconstructing the vector. **Do not** send the floats.

### 3.4 Presence event payload

```json
{
  "visible": true,
  "focused": true,
  "last_input_at": "2026-08-12T18:04:11Z",
  "window_ms": 240000
}
```

The tick **re-validates** the conjunction using the payload fields; a client that lies still cannot invent more than one ping per 20s per session because of server-side rate limits (`presence_ping` accepted at most every 15s per session).

---

## 4. API surface

All authenticated routes take an httpOnly session cookie. IDs in paths are UUIDs. Matter-of-fact error bodies.

### 4.1 Auth / account

| Method | Path | Notes |
|---|---|---|
| POST | `/v1/auth/magic-link` | `{email}`. Always 202. Rate-limit per email_hash + IP. |
| GET | `/v1/auth/consume?token=` | Single use. Sets session cookie. 15m expiry. Replay → matter-of-fact expired page. |
| POST | `/v1/auth/sign-out` | Revokes this session. |
| GET | `/v1/account` | Settings, timezone, deletion state. No personality. |
| PATCH | `/v1/account` | timezone, a11y prefs, `visit_notify_email`. |
| GET | `/v1/account/sessions` | Device list. |
| DELETE | `/v1/account/sessions/:id` | Revoke. |
| POST | `/v1/account/email-change` | Sends verify to new address; old email still works. |
| GET | `/v1/account/email-change/consume?token=` | Commits switch. |
| POST | `/v1/account/export` | Enqueues JSON snapshot email. |
| POST | `/v1/account/delete` | Soft-delete now. |
| POST | `/v1/account/undelete` | Only if within 30 days. |

Export JSON includes birds (including personality vectors — this is the user’s copy), moods, notebook, settings. It is emailed as a short-lived download link, not streamed in-browser, so we do not accidentally render numbers in the UI.

### 4.2 Aviary state and events

| Method | Path | Notes |
|---|---|---|
| GET | `/v1/aviary/snapshot` | Canonical snapshot. Triggers catch-up tick if stale. |
| GET | `/v1/aviary/stream` | Optional SSE: snapshot every 60s while visible. Fallback: 45s poll. |
| POST | `/v1/aviary/events` | Batch of events. Idempotent on `client_event_id`. 202. |
| POST | `/v1/aviary/events/beacon` | `text/plain` sendBeacon variant. |
| GET | `/v1/notebook` | Cursor pagination, oldest-first or newest-first; full history. |
| PATCH | `/v1/birds/:id` | `{name}` only. |
| POST | `/v1/aviary/adopt` | Accepts pending age-gated slot; server picks species; returns new bird id + suggested names. |

**Event POST body:**

```json
{
  "session_id": "…",
  "events": [
    {
      "client_event_id": "…",
      "kind": "listen_in_start",
      "bird_id": "…",
      "occurred_at": "…",
      "payload": {}
    }
  ]
}
```

Reject unknown kinds. Reject events with `occurred_at` > 5m in the future or > 24h in the past. Do not 4xx the whole batch for one bad event; return per-item results.

### 4.3 Visit invitation flow

| Method | Path | Notes |
|---|---|---|
| POST | `/v1/invites` | Host `{email}`. Creates invite, emails one-time link. Off-by-default means this is the only enablement. |
| GET | `/v1/invites` | Outstanding + recent, for settings visit log. |
| POST | `/v1/invites/:id/revoke` | Immediate. Next visitor snapshot → 410. |
| GET | `/v1/visit/:token` | Unauthenticated. Validates token, returns read-only snapshot + short-lived visit cookie. |
| GET | `/v1/visit/snapshot` | Visit-cookie snapshot. Identical rendering fields; `visit.read_only=true`. No events endpoint. |
| GET | `/v1/account/visit-log` | Host: visitor email, date, approx duration, outstanding invites. No badge, no push. |

Visitor client is the same SPA with `mode=visit`. It **does not mount** listen-in, offer, settle, notebook-as-interactive, or presence pinger. Audio plays; captions follow visitor a11y settings stored locally only.

Revoked/expired token response (matter-of-fact):

> This visit is no longer available.

If `visit_notify_email` is on, mailer sends a single email after a visit session ends (≥60s duration to ignore accidental opens). Default off. Never in-product toast.

### 4.4 System copy

Sign-in, expiry, timeout, load failure use the PRD sentences verbatim. Naturalist voice is forbidden on these routes’ HTML.

---

## 5. Simulation engine design

### 5.1 Tick worker

- One logical worker; shard by `account_id` hash if needed later.
- Schedule: every 60s, claim aviaries where `last_ticked_at < now()-55s` OR any unconsumed events exist.
- On snapshot GET, if `now - last_ticked_at > 90s`, run `N = min(1440, floor(delta/60))` catch-up ticks **in-process** before returning. Cap 1440 (= 24h of minute ticks) per request; leftover time is marked and a background job finishes the rest so a 2-week absence does not block TTFB. **Decision D16:** long catch-up is backgrounded after the first 30 ticks (30 minutes of sim) so first-bird stays <500ms; remaining ticks complete within a few seconds and the next snapshot/SSE pushes the rest. Mood/lighting for the *current* local time is applied on the first catch-up so the scene is already in the right part of day.

Tick steps, in order:

1. Load aviary + birds + unconsumed events (`consumed_tick IS NULL`) ordered by `ingested_at`, `id`.
2. Fold events into a `SessionSignals` struct (presence-seconds, listen-in seconds per bird, offers per bird, settle/unsettle).
3. Advance weather (see 5.5).
4. Advance mood per bird (5.3).
5. Apply personality **deltas** (5.2). Clamp `[0,1]`. Never decrement.
6. Choose perch zones from mood × boldness (5.6).
7. Advance call-grammar clocks; emit next-onset times into snapshot fields.
8. Maybe write a notebook entry (5.7).
9. Maybe open an adoption slot from aviary age (5.8).
10. `tick_seq++`, set `last_ticked_at`, mark events consumed.

Clients never run this function.

### 5.2 Drift function

Treat each trait as a leaky integrator that only accepts non-negative increments.

Let `P` be daily presence-hours (sum of accepted ping credits / 3600), typically 0–1.

Per tick (1 minute), for each bird:

```
presence_w = clamp(presence_seconds_this_tick / 60, 0, 1)

d_bold   = 1.6e-5 * presence_w
         + 2.0e-5 * offer_near_bird   # offer in this tick, any kind
d_warm   = 1.4e-5 * presence_w
         + 3.5e-5 * listen_in_frac    # seconds listening / 60
d_vocal  = 1.2e-5 * presence_w
         + 3.0e-5 * listen_in_frac
d_plum   = 1.8e-5 * presence_w        # attention → saturation
d_curi   = 1.0e-5 * presence_w
         + 4.0e-5 * offer_accepted    # bird approached / engaged

trait = min(1.0, trait + d_*)
```

**Calibration check (must be tested, not tuned by feel in prod):**

Assume “regular visits” = 20 min presence, 4 days/week.

- Per week ≈ 80 presence-minutes = 80 ticks with `presence_w≈1` if they sit continuously, more realistically ~60 effective ticks.
- `boldness` weekly Δ ≈ `60 * 1.6e-5 ≈ 0.0010` from presence alone, plus small offer terms.
- After 1 week: Δ ≈ 0.001–0.003 — **measurable in fixtures**, invisible in motion.
- After 3 weeks: Δ ≈ 0.004–0.010; perch-zone softmax starts to flip a borderline bird toward front more often. **Visible in retrospect.**

If fixtures miss the 1-week measurable / 3-week visible band, change the `1.x e-5` constants — not the monotonic rule.

**Ambient-not-negative:** if `presence_w == 0` for many days, deltas are 0. Greeting *probability* uses a recency window (5.4) so quiet birds greet less often without lowering traits. Neglect never decreases saturation or raises wariness via the vector.

Settle: zero drift terms; it only ends the presence window and sets `settled=true` (mood nudge in 5.3).

### 5.3 Mood transitions

Mood is a discrete Markov step each tick.

Inputs:

- `tod` = local hour: morning 5–9 `alert` bias, midday `content`, dusk 17–21 `drowsy`, night 21–5 `drowsy` (nightjar species: `alert` allowed).
- `weather`: rain → lower vocal activity, shift toward `content`/`drowsy`; wind → `alert` or `wary` by boldness.
- Recent events: accepted offer → `content`/`curious`; listen-in → slight `curious`; other bird `wary` → contagion.
- Personality as *transition bias*, not a displayed stat: high boldness halves `→ wary` probability; high curiosity doubles `→ curious` on novel weather/offer.

Persistence: store mood on the bird row. Do **not** reset on snapshot. On long catch-up, walk mood through time-of-day so a bird that was `drowsy` at dusk is not still `drowsy` at 10:00 unless rain/personality holds it.

Never snap to `content` on tab open.

### 5.4 Return-greeting (session-open, not a tick feature)

Computed when building a snapshot if `since_last_host_presence > 20s` (covers tab return and fresh nav).

1. Score each bird: `boldness + warmth + mood_mod - 0.15 if wary`.
2. Pick the top bird as primary greeter; with small noise so it is not deterministic every day.
3. Absence length buckets: `<5m` glance; `5m–6h` short call + head tilt; `>6h` longer call, possible step to front if boldness high.
4. If a second bird’s score is within 0.08, attach a staggered secondary greeting at +400–1200ms.
5. Never fire all birds at t=0.

The snapshot carries `greeting` hints; the client performs the motion/audio. Procedural variation comes from `seed = hash(tick_seq, bird_id, session_id)` so two opens are never identical.

**No welcome toast, no “you’ve been gone N days,” no calendar.** If a PR sneak-adds one, reject it.

### 5.5 Weather and ambient events

- Target: rain ~3×/week, wind ~2×/week, duration 8–25 minutes.
- Sample a geometric process each tick; never stack thunderstorms; never snow.
- Weather is a field on `aviaries`, not user-facing chrome.
- Rain: multiply call onset rate by 0.55 for duration + 5 minutes after.
- Wind: +alert for high-curiosity birds, +wary for low-boldness birds, short-lived.

### 5.6 Perch selection

Three zones × N slots. Birds are not user-placed.

Each tick, desired zone:

```
score_front  = 0.55*boldness + 0.20*(mood in {alert,curious,content}) - 0.35*(mood==wary)
score_back   = 1 - score_front
```

Hysteresis: do not change zone unless the new score beats the current by 0.12, so birds do not pace. Interpolation on the client is 4–10s ease.

Night: most species desired zone `middle`/`back`, pose `sleep`; nightjar may stay `front`/`middle`.

### 5.7 Call-grammar runtime (server clock, client synth)

Each species has a motif library: 4–7 motifs (pitch envelope + rhythm cells). A call is:

```
call = pick(motif_a, seed)
     + optional(motif_b if warmth>tier and mood!=drowsy)
     + timing_jitter(vocal_tier, mood)
     + pitch_shift(mood, species_base)
```

Server stores `grammar_id`, `seed`, `next_onset_ms`. Client synthesizes that exact call so captions match audio (same seed → same caption string).

Chorus: if bird B’s next onset falls within 1.2s of bird A’s onset and B’s `vocal_tier` ≥ 1 and mood ≠ `wary`, B joins with a response motif. This is computed in the tick (or in a lightweight “audio plan” attached to the snapshot covering the next 60s of onsets). **Decision D17:** snapshot includes an `onset_plan` of up to 12 timed calls for the coming minute so chorus alignment does not require a second protocol.

Recognizability: species `grammar_id` is fixed for the bird’s life. Mood only changes timing/intensity, not the motif family.

### 5.8 Bird-to-bird

- Call response as above.
- Wary contagion: if one bird is `wary`, neighbors have +0.15 transition bias for 3 ticks.
- Chorus when ≥2 high `vocal_tier` birds share a window.

These are social-system signals, not user-facing stats.

### 5.9 Adoption / naming / identity

- New account: insert two birds, species sampled without replacement from the pool, names suggested, `bird_id`s minted once.
- User does not catalog-pick starters. Empty aviary → quiet field → first bird soft fly-in (the **only** allowed fly-in after signup). After that, birds are always already in scene.
- Age gates (D7) set `pending_adoption_slot`. UI: a small top-bar affordance in naturalist voice (“a new bird is near”) — not a reward modal. Accept → server assigns species (prefer unused in this aviary, then allow repeats), user names.
- Rename is a string update. Zero effect on vector, mood, grammar.
- Never recycle `bird_id`. Never “reset” a bird. Migrations must copy vectors in place.

### 5.10 Offer resolution

On `offer` event `{kind: seed|song|pool}`:

- If `now - last_offer_at[kind] < 4m` for that bird, accept the event but mark `cooldown: true`; no drift; client already should have disabled the action.
- Else pick a receiving bird: nearest to front among those not drowsy, else any; curiosity + mood decide approach vs ignore vs delay (wary waits 8–20s then may approach).
- Seed: approach / peck / ignore.
- Song: play library motif id from payload; bird join / quiet / counter-call from vocal_tier × mood.
- Pool: spawn a 45s scene prop; drink / bathe / watch.

Reactions are encoded as short `action_plan` items on the next snapshot (or immediately if we run a micro-tick on offer ingest — **Decision D18:** offer and settle trigger an out-of-band *partial* tick so reaction is in the next 1s, not the next minute). Partial ticks may write mood and a small drift delta; they still only run on the server.

---

## 6. Sync model

### 6.1 Single canonical record

There is no client-to-client sync and no CRDT. Laptop and phone both `GET /snapshot`. Personality exists in one row. Conflict on personality is **structurally impossible** if the rule “only the tick writes vectors” is enforced by DB grants (`sim_app` role: insert `events`; `sim_tick` role: update `birds` personality columns).

### 6.2 How devices stay aligned

- Visible tab: SSE or 45s poll + pull on `visibilitychange` to `visible` + pull if `rAF` gap > 2s (laptop wake).
- Hidden tab: no render, no audio, no presence pings. Simulation continues.
- Settle and tab-close both terminate presence. `sendBeacon` on `pagehide`. Missing beacon is fine; pings just stop.

### 6.3 Preventing the lunch-overwrite bug

Clients **cannot** `PATCH` personality. Events are append-only and consumed in ingest order. Two devices sending presence/listen-in concurrently produce two event streams; the tick adds both deltas. Last-write-wins is not used.

Idempotency: `unique (account_id, client_event_id)` so retries do not double-count presence.

### 6.4 Visit vs host

Visitor snapshots are reads of the same rows. Visitor traffic must not write `events`. Enforce at the gateway: visit cookie has scope `visit:read`.

### 6.5 Rare conflict / error surfaces

Magic-link replay, expired session, sim-db blip → matter-of-fact copy from the PRD. No naturalist errors. No toast that a friend visited.

Soft-deleted accounts: sign-in allowed; aviary hidden behind a single restore screen (system voice). After 30 days, hard-delete job wipes account-db, sim-db, mail logs’ PII, export objects.

---

## 7. Frontend rendering pipeline

### 7.1 Scene composition

One horizontal scene, no pan/zoom/scroll of the aviary. Three depth planes:

- Background: sky gradient (local time), distant foliage.
- Mid: three perch zones, birds, still pool when offered.
- Foreground: occasional branch/leaf, captions, focus ring.

Top bar is **outside** the scene. Icons only: account/settings, accessibility, notebook, offer. After ~3s cursor/keyboard idle, bar opacity → ~0.08; any pointer/key restores it.

Palette: soft blues/greens/warm browns/muted ochres. No saturated UI accents in the scene. Chrome contrast AA.

Responsive: scale the scene to fit the viewport; keep aspect such that **every bird stays on-screen**. Narrow: compress inter-perch spacing. Wide: add air. Never crop.

### 7.2 First frame / loading

- No spinner. No fade-from-static. No “ready” pop.
- HTML + critical CSS paint the quiet field immediately (sky color for estimated local time if we have a cached timezone; else a neutral dawn).
- When snapshot arrives, instantiate birds at `pose_t` already mid-cycle and start the motion clock at `performance.now()` with phase offset from snapshot — visually continuous.
- Cold snapshot >~300ms: stay on quiet field (maybe one CSS leaf). Then birds appear mid-action, not via a logo sting.
- Signup empty state: quiet field, then one-time soft fly-in for bird 1 and 2. Never again.

### 7.3 Idle micro-motion

Continuous, personality/mood-shaped, never paused-looking:

| Mood | Motion |
|---|---|
| wary | back perch bias, more scan saccades, less preen |
| content | preen cycles, slow weight-shift |
| curious | head-tilt toward onsets and falling leaves |
| drowsy | low sit, fluffed silhouette, long holds |
| alert | more scan, quicker hops between slots in-zone |

Birds never read as paused. Even drowsy has breathing-scale scale/squash (~0.3Hz, amplitude tiny).

Ambient leaves/feathers: client-only Poisson process, not in sim-db. Subtle parallax only.

### 7.4 Transitions

- Perch changes: 4–10s eased path along a shallow arc, not teleport.
- Day/night: continuous palette mix from `local_minutes`. Evening warm; night dim; not a cut.
- Settle: 3–5s lighting ease to evening, call gain down. `settled=true`.
- Unsettle: any pointer/key in the scene within 5s of settle cancels (client sends `unsettle` if still inside the window; after 5s, a later click is “re-engage” and also clears settled).
- Weather: rain as sparse falling strokes; wind as leaf-rate bump. Never assertive.

### 7.5 Reduced-motion mode

Trigger: `prefers-reduced-motion: reduce` OR settings override `on`.

This is a **designed renderer**, not `animation: none`:

- Idle: still poses with 1.2–2.0s crossfades on pose change.
- Flight: crossfade between perch stills, no arcs.
- No leaf/feather drift.
- Day/evening still shifts, slower (2× duration).
- Audio and drift unchanged.
- Focus rings and captions unchanged.

Ship on day one. Feature-flagging it after launch is a v1 fail.

### 7.6 Interaction targeting

- Click/tap/keyboard-focus a bird → listen-in (not offer).
- Offer only from top bar → small chooser (seed / song / pool) in naturalist labels, then a placement in the scene, not on a bird’s hit-target.
- Empty-scene click ends listen-in.
- No tooltips, badges, or inline labels in the scene except captions and focus outline.

### 7.7 Notebook UI

Top-bar icon → panel over (not inside) the scene. Read-only scroll of stored prose. No edit/delete/annotate. Infinite history. Voice: lowercase, present, specific. Reject any entry template that mentions visit streaks or user behavior.

---

## 8. Audio pipeline

### 8.1 Graph

```
per-bird Voice (oscillators + noise + formant filters)
    → birdGain          (listen-in target)
        → chorusBus
            → masterGain
                → destination
```

No sample files in the bundle. Motif libraries are tiny parameter tables (tens of KB).

### 8.2 Procedural synthesis

- Each motif: sequenced exponential-ramp frequencies + filtered noise bursts (trill) + amplitude envelope.
- `seed` from snapshot makes the call deterministic for caption alignment, still unique across calls.
- Species base pitch + interval set is the recognizability fingerprint; mood scales rate and brightness only.

### 8.3 Chorus mixing

Real simultaneous voices, not stacked loops. Slight detune per bird. Master compressor with low ratio so two calls bloom instead of pumping.

### 8.4 Listen-in mix

- Engage/disengage ramps **1.6–2.2s** (not a cut).
- Focused bird → birdGain toward 1.0.
- Others → toward 0.22–0.30 (never 0).
- Disengage: all return to ambient staging derived from perch zone (front slightly louder than back).

Keyboard: Tab into scene → first bird; arrows move; Enter listen-in; Esc exit. Second Enter on same bird toggles off.

### 8.5 Settle / night

Settle and evening multiply master or per-bird rate. Nightjar species keeps a low night motif.

### 8.6 WebAudio fallback

If `AudioContext` missing, `resume()` rejected, or hardware error:

- Stay silent.
- Force captions on for the session (user can turn off).
- **No recorded-audio path. Unconditional.**

Mute toggle in a11y settings is user-initiated silence with captions optional (not forced). Distinct from fallback.

### 8.7 Autoplay

First audio after a user gesture if the browser requires it. Visual aviary does not wait on audio. A first call may be delayed until gesture; greeting motion still runs. Do not show a “click to enable sound” toast; a quiet top-bar speaker state in settings is enough. **Decision D19:** if blocked, treat as temporary fallback (captions on) until the first pointer/key, then resume audio and leave captions as the user set them.

---

## 9. Accessibility surfaces

Accessibility is a designed product surface and ships with v1.

### 9.1 Screen-reader narration

- Single `aria-live="polite"` region, visually hidden, updated with **prose**, not state dumps.
- Idle cadence 30–60s. Faster on greeting, offer reaction, settle.
- Same naturalist voice as the notebook.
- **Never** expose personality numbers, perch indexes as “perch 2”, or “mood: content”.
- Generation: client-side template composer over snapshot + last action_plan, with a small phrase bank. Server may attach a `narration` string on noteworthy ticks to keep voice quality high; if attached, client prefers it.
- Queue: replace pending idle narration if a priority event arrives; do not stack three sentences.

Example idle line:

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

### 9.2 Captions

- Opt-in in a11y settings (on by default only in WebAudio fallback).
- Generated from the same seed/grammar as the sounding call: “a soft three-note rise”, “a low trill, paused, low trill again”.
- Small text near the calling bird, fade with the call. AA contrast. Reduced-motion: snap in/out rather than fade if needed.

### 9.3 Keyboard and focus

- Tab: top-bar icons left-to-right, then first bird.
- Arrows: birds.
- Enter: listen-in.
- Esc: exit listen-in; second Esc returns focus to top bar.
- Offer and settle fully keyboard-reachable from the bar.
- Focus ring: soft but high-contrast, specified for both day and night palettes.

### 9.4 Contrast and settings

WCAG AA on all user copy (bar, settings, errors, captions, notebook). Aviary art is not user copy.

A11y settings use **matter-of-fact** voice (named exception). Reduced-motion override, captions, mute.

### 9.5 Visit mode a11y

Visitor gets the same narration/captions/reduced-motion. No interactive bird actions in the tab order beyond a single “leave visit” control.

---

## 10. Performance budgets and observability

### 10.1 Budgets (CI-enforced)

| Budget | Gate |
|---|---|
| Initial JS gzipped | < 2.0 MB (fail CI above 1.8 MB warn / 2.0 MB fail) |
| Time to first bird | < 500 ms p75 synthetic mid-tier 4G |
| Idle FPS | 60 fps on reference 5-year laptop, 30-minute session |
| Memory | no growth over 30 minutes (heap snapshots ±2% after GC) |
| Snapshot size | < 16 KB typical, < 32 KB hard |
| Tick p99 | alarm at 5s; SLO target p99 < 200ms at v1 scale |

How to hit TTFB: inline critical CSS; defer settings/notebook/visit modules; snapshot from an edge-adjacent API; draw silhouettes before plumage detail; do not wait on AudioContext.

### 10.2 Implementation tactics

- Procedural audio (no sample bank).
- Species art: SVG silhouettes + compact texture atlases; plumage sat_tier swaps CSS/SVG filters rather than new bitmaps.
- Reuse audio nodes; ring-buffer grains; no per-call `new Oscillator` leak — connect from a pool.
- Notebook virtualized list; drop DOM nodes on scroll-out.
- Suspend renderer on `document.hidden`.
- Code-split: `/settings`, `/visit`, `/sign-in`.

### 10.3 What we measure

Synthetic fleet: load, first-bird paint, rAF long-tasks, audio-context errors, tick latency. Geographies = primary user regions.

RUM (aggregate only): navigation timing, first-bird custom mark, frame-time histogram, audio errors. **No `account_id`, no bird ids, no event kinds, no session-duration per account.** Session-duration **histogram without dimensions** is allowed.

### 10.4 What we deliberately do not measure

- Per-bird interaction rates, offer funnels, listen-in heatmaps, drift distributions, “average boldness”, visit-frequency per user, anything that could reconstruct a relationship.
- No warehouse sync from `sim-db`.
- No ML features on bird fields.

Privacy policy in settings lists operational categories and explicitly excludes per-bird state.

### 10.5 Browser support

Unsupported UA → matter-of-fact page. No polyfill mountain.

---

## 11. Rollout

### 11.1 Build sequence (suggested workstreams)

1. **Foundations:** two DBs, synthetic `account_id`, magic-link, session revoke, privacy firewall in CI (grep + SQL grants).
2. **Sim core:** birds schema, tick, event ingest, snapshot DTO without numbers, drift fixtures for 1-week / 3-week targets.
3. **Scene v0:** quiet field, two silhouettes, mid-action first frame, day/night, three zones, reduced-motion crossfades.
4. **Audio v0:** one species grammar, chorus of two, listen-in ramps, fallback silence+captions.
5. **Interactions:** greeting variation, offer+cooldown+partial tick, settle+5s undo, presence pinger with three-signal conjunction.
6. **Notebook + narration** from the same phrase bank.
7. **Adoption age gates** (clock injectable in tests).
8. **Visits** last among product features — after read-only snapshot is cheap and event writes are locked down.
9. **Export / delete / email-change.**
10. **Perf CI + synthetic fleet** before public ramp.

### 11.2 Birds-per-aviary ramp

- Launch: all new accounts get **two** birds. Cap remains seven in the engine.
- Do not start anyone at 3+ “to show richness.”
- Age gates (D7) are the only ramp. No staff override that emails “you’ve unlocked a bird” announcements. If ops need a support tool, it is an internal admin action, logged, not a user-facing reward.

### 11.3 Feature flags

Keep flags internal (audio engine on, SSE on). Do **not** flag reduced-motion, captions, or narration for a later train — they are launch blockers.

### 11.4 Instrumentation from day one

- Tick p99 + claim lag.
- Snapshot latency + size.
- First-bird mark.
- Audio-context error count.
- Event ingest 4xx/409 rates.
- Magic-link issue/consume ratio (abuse).
- Soft-delete counts.
- **No** drift dashboards that average traits across accounts.

### 11.5 Launch bar

- Drift fixtures green.
- Presence conjunction tests green (including “background tab for 2h adds ~0 presence”).
- A11y pass: keyboard path, live-region prose review, reduced-motion visual QA, caption/seed alignment test.
- Bundle + TTFB + 30-min memory jobs green.
- Privacy CI: no email in sim-db dumps; no bird fields in metrics labels.

---

## 12. Risks

### 12.1 Drift calibration

**Risk:** constants too fast → Tamagotchi; too slow → screensaver. Silent presence bugs inflate the whole population.

**Mitigations:** fixture-based weekly/three-week targets; presence re-validated server-side; ping rate limit; never decrement traits; no last-write-wins; do not add “catch-up presence” for offline time.

### 12.2 Sync correctness

**Risk:** a convenience `PATCH /birds/:id/state` appears during a crunch; two devices diverge; lunch session erases morning drift.

**Mitigations:** DB role split; snapshot contract tests that fail if personality floats leak; code review checklist item in sim-svc; idempotent events.

### 12.3 Audio uncanniness

**Risk:** loops, phase-cancelled stacked samples, identical greetings, or a canned MP3 fallback when WebAudio fails.

**Mitigations:** no sample assets in the repo (lint); seed-varied grammars; chorus as independent voices; fallback = silence + captions; listen-in never mutes others fully; greeting seed includes session id.

### 12.4 Accessibility regressions

**Risk:** ARIA state dumps, reduced-motion as “static screenshot,” captions as fixed strings that disagree with the call, late a11y.

**Mitigations:** launch blockers; narration snapshot tests (forbidden substrings: `mood:`, `boldness`, `perch 2`, `welcome back`); reduced-motion visual golden tests; caption == grammar caption function(seed); keyboard e2e.

### 12.5 Affective leaks (process risk)

**Risk:** a “harmless” toast, streak, badge, or “your friend visited.”

**Mitigations:** PR template checkbox against `non_goals.md`; no toast component in the aviary route; visit log has no badge API; notebook linter rejects second-person visit-frequency.

### 12.6 First-frame failure

**Risk:** spinner then fade-in destroys “already running.”

**Mitigations:** quiet-field HTML; budget on TTFB; catch-up tick cap so snapshot GET cannot stall; no entry animation component.

### 12.7 PII spray

**Risk:** email used as partition key or log field.

**Mitigations:** synthetic UUID rule; email only in account-db; CI denylist; keyed email hash for lookup.

### 12.8 Offer/curiosity saturation

**Risk:** session button-mash maxes curiosity.

**Mitigations:** 4-minute per-kind cooldown server-side; small offer deltas vs presence.

### 12.9 Long absence catch-up vs TTFB

**Risk:** 20k ticks on return block first bird.

**Mitigations:** D16 — apply “now” lighting/mood immediately, background remaining ticks, SSE the rest.

### 12.10 Visit accidentally driving drift

**Risk:** shared client sends presence pings.

**Mitigations:** visit cookie cannot POST `/events` (gateway 403); visit bundle omits pinger.

---

## 13. Testing strategy (for the implementing team)

- **Unit:** drift monotonicity, clamp, 1-week/3-week fixtures, mood TOD walk, greeting stagger, cooldown, invite expiry/revoke.
- **Contract:** snapshot schema forbids `boldness` et al.; metrics exporter forbids bird labels.
- **Presence:** jsdom/browser tests for the three-signal conjunction; hidden tab; unfocused window; no input for 5 minutes → pings stop.
- **Render:** first-frame has non-zero pose phase; reduced-motion uses crossfades; no bird off-viewport at breakpoints.
- **Audio:** two-voice mix never zeros non-focused; fallback path sets captions; no `*.mp3` in bundle.
- **A11y:** axe on chrome surfaces; live-region content snapshots; keyboard map e2e.
- **Perf:** CI bundle; 30-min memory; synthetic 4G first-bird.
- **Privacy:** integration test that warehouse user cannot `SELECT` sim-db.

---

## 14. Voice implementation note

Ship a `copy/` module with two functions: `naturalist(key, ctx)` and `system(key, ctx)`. Aviary, notebook, narration, captions, offer labels → `naturalist`. Auth, errors, settings, visit-unavailable → `system`. New surfaces that handle money, identity, errors, or settings default to `system`.

---

This is the v1 build plan. Do not implement beyond this document in phase 1. Do not add native clients, gamification, Tamagotchi pressure, or a social graph. The product is a window that notices the user, not an app that announces them.
