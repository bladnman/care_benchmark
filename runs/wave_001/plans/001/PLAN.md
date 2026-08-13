# Pocket Aviary — v1 implementation plan

This plan turns the PRD into an executable system. It does not restate the product; it makes the engineering calls a second team needs to ship. Where the spec is intentionally open, a defensible default is named and marked **[decision]**.

---

## 1. Scope

### 1.1 In v1

- Browser-only SPA (last two major Chrome, Safari, Firefox, Edge).
- Single-user account, one canonical aviary per account.
- Email magic-link auth; per-device revocable sessions.
- Two starter birds at signup; hard cap of seven; additional birds gated by aviary age, not engagement.
- Server-owned personality vectors, mood, perch intent, weather, and notebook generation.
- Client-owned interpolation, idle micro-motion, procedural WebAudio calls, and first-frame “already running” scene.
- Interactions: presence (idle attention), listen-in, offer (seed / song fragment / still pool), settle (+ 5s undo), field notebook (read-only), rename, age-gated adoption.
- Multi-device sync as a property of “server is the only writer of canonical state.”
- Visit invitations: per-invite opt-in, read-only ambient view, revocable, 30-day unused expiry, silent visit log, notifications off by default.
- Accessibility shipping on day one: naturalist screen-reader narration, designed reduced-motion renderer, runtime call captions, WCAG AA chrome, full keyboard path.
- Account export (JSON emailed), soft-delete 30 days then hard-delete.

### 1.2 Out of v1 (non-goals, enforced in review)

- Native apps; no native-shaped protocols or SDKs.
- Any gamification: scores, streaks, badges, levels, visit calendars, “birds adopted: N,” XP, ranks, milestone celebrations.
- Tamagotchi mechanics: death, hunger, distress, decaying happiness, punishment for absence.
- Social network surfaces: profiles, follows, discovery, comments, chat, avatars, co-presence, leaderboards, public aviaries.
- Payments, multi-aviary accounts, customizable scenes, shared/household aviaries.
- Push notifications of any kind. The only optional outbound email besides auth/invite/export is host-opt-in visit mail (**[decision]** in §4.6).
- Personality numbers anywhere a user can see them, including debug UI in production.
- Recorded-audio fallback. WebAudio missing → silence + captions on.

### 1.3 Product invariants (treat as CI-level rules)

1. First painted frame is mid-action or a quiet field — never a spinner, never fade-from-static.
2. No announcement toasts/banners/modals on return. Bird greeting is the welcome.
3. Clients never write personality or mood. They append events.
4. Drift is monotonic toward expressive. Neglect does not decrement traits.
5. Presence requires `visibilityState === 'visible'` AND window focus AND recent pointer/key activity.
6. Internal IDs are synthetic UUIDs. Email lives in one encrypted column and nowhere else (logs, keys, telemetry dimensions).
7. Per-bird interaction data never enters aggregate telemetry, warehouse, or any training path.
8. Naturalist voice on product surfaces; matter-of-fact voice on auth, errors, sync, account, and a11y-settings.

---

## 2. Architecture

### 2.1 Service shape

Three deployable units, one data plane:

| Unit | Responsibility |
|---|---|
| **api** | Auth, sessions, snapshot reads, event ingest, notebook reads, settings, visits, export, deletion |
| **sim-worker** | Per-aviary tick (~60s), drift, mood, perch intent, weather, notebook authoring, adoption eligibility |
| **mailer** | Magic links, visit invites, export links, optional visit-notice email, deletion confirmations |

Shared:

- **Postgres** — canonical state + append-only event log.
- **Redis** — session token lookup, magic-link single-use, rate limits, per-aviary tick lock, short-lived snapshot cache.
- **Object store** — export JSON blobs (signed, short TTL).
- **CDN / edge** — static assets + HTML shell. Authenticated HTML may include a last-known snapshot bootstrap (see §8.3).

No Kafka, no analytics warehouse reader on the simulation DB, no client-to-client channel.

**[decision]** Monorepo, TypeScript on api + worker + client. api is a single stateless HTTP service. sim-worker is a horizontal pool that claims aviaries via `SELECT … FOR UPDATE SKIP LOCKED` (or Redis lock keyed by `aviary_id`). One tick owner at a time.

### 2.2 Client / server split

**Server owns (canonical):**

- Account, sessions, aviary age, bird identity, names, species, personality vector, mood, perch *zone intent*, weather phase, day/night phase (from stored timezone), offer cooldowns, notebook entries, visit invites/log, adoption eligibility.
- Greeting *plan* for the next host visibility (which bird, absence bucket, seed).
- Narration source text for the current slow cadence (or enough structured facts for the client narrator — see §10.1).

**Client owns (ephemeral, never authoritative):**

- Scene-graph interpolation, idle micro-motion, leaf/feather ornaments.
- Procedural call synthesis and listen-in mix.
- Presence detector and event batching.
- Reduced-motion pose crossfades, captions, focus overlay.
- Settle lighting (session-local) after posting the settle event.

**Clients never tick.** A hidden or background tab stops rendering; the worker keeps ticking.

### 2.3 Render pipeline boundary

```
snapshot (JSON)
    → SceneState (typed, versioned)
        → SceneGraph (birds, perches, weather, lighting)
            → VisualRenderer (canvas 2D, two strategies: full-motion / reduced-motion)
            → FocusOverlay (DOM, hit + keyboard)
            → AudioEngine (WebAudio graph)
            → NarrationController (live region)
            → CaptionLayer (DOM)
```

VisualRenderer must be swappable without changing SceneGraph. AudioEngine reads the same SceneState. No renderer writes back into canonical fields.

### 2.4 Trust and privacy boundary

- Simulation DB and operational metrics are separate credentials and separate schemas.
- Telemetry exporters are allowlisted to operational tables/metrics only.
- Email encrypt-at-rest; decrypt only in auth/mailer/visit-log/export paths.
- Visit snapshot endpoint is a distinct capability: read-only projection, no event ingest, no presence.

---

## 3. Data model

All primary keys are UUIDv7 (time-sortable) except where noted. Timestamps are `timestamptz`.

### 3.1 `accounts`

| Column | Notes |
|---|---|
| `id` | Synthetic UUID. The only identifier used in logs, locks, partitions. |
| `email_ciphertext` | AES-GCM; unique hash index on blind index (`email_blind_index`) for lookup |
| `email_pending_ciphertext` | In-flight email change |
| `timezone` | IANA tz; updated from client on session start **[decision]** |
| `created_at` | Aviary age clock starts here (same instant as aviary row) |
| `deleted_at` | Soft-delete marker; null if active |
| `hard_delete_after` | `deleted_at + 30d` |
| `visit_notify_email` | bool, default false |
| `captions_default` | bool, default false |
| `reduced_motion_override` | `null \| true \| false` — null means follow `prefers-reduced-motion` |

### 3.2 `sessions`

`id`, `account_id`, `token_hash`, `device_label` (UA-derived, editable later if needed), `created_at`, `last_seen_at`, `revoked_at`.

### 3.3 `aviaries`

One row per account.

| Column | Notes |
|---|---|
| `id`, `account_id` unique | |
| `created_at` | Age-gate source |
| `weather` | `clear \| rain \| wind` |
| `weather_until` | |
| `last_ticked_at` | |
| `tick_version` | Monotonic; snapshot etag |
| `next_bird_eligible_at` | Precomputed by tick |
| `last_host_presence_end` | For absence-length greeting |
| `notebook_last_written_at` | Sparsity control |

No “settled” column. Settle is a session gesture + event; lighting is client-local.

### 3.4 `birds`

| Column | Notes |
|---|---|
| `id` | Stable forever. Never recycled. |
| `aviary_id` | |
| `species_id` | FK into code-defined species catalog (not user data) |
| `name` | User string, length 1–24, no empty |
| `adopted_at` | |
| `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity` | `numeric(6,5)` in **[0, 1]** |
| `mood` | `wary \| content \| curious \| drowsy \| alert` **[decision: lock this set]** |
| `mood_entered_at` | |
| `perch_zone` | `front \| middle \| back` |
| `perch_slot` | 0–2 within zone; tick assigns without overlap |
| `pose_seed` | uint32; client uses to start mid-cycle |
| `call_phase_ms` | For chorus timing continuity across snapshots |
| `offer_cooldown_until` | Per-bird, all offer types share one cooldown **[decision]** |
| `expressiveness` | **[0, 1]** fast-medium gain from recent presence; *not* a personality trait. Implements “ambient quietness” without negative drift. **[decision]** |

`expressiveness` decays slowly toward a floor (~0.25) when presence is absent, and rises toward 1.0 with recent honest presence. Greeting rate, chorus join probability, and how often the bird looks at the viewer are `f(personality, expressiveness, mood)`. Traits themselves never decrease.

### 3.5 `interaction_events` (append-only)

| Column | Notes |
|---|---|
| `id` | |
| `account_id` | |
| `aviary_id` | |
| `bird_id` | nullable |
| `session_id` | host session; null for system |
| `type` | see §4.2 |
| `payload` | jsonb, small, schema-validated |
| `client_occurred_at` | |
| `received_at` | server clock |
| `consumed_at` | set by tick |

Unique `(session_id, client_event_id)` for idempotency.

**Never update event rows except `consumed_at`.** No deletes except hard-delete of the account.

### 3.6 `notebook_entries`

`id`, `aviary_id`, `written_at`, `prose` (lowercase naturalist), `trigger` (internal enum: `greeting_order`, `long_quiet`, `weather`, `mood_texture`, `first_listen_in_week`, …). `trigger` is never exported to the product surface; include it in account export under an `internal` key or omit **[decision: omit from user-facing export, include in ops-only dump]**. User export includes `written_at` + `prose` only.

No user edit/delete. Indefinite retention until account hard-delete.

### 3.7 `visit_invites` / `visit_sessions`

- Invite: `id`, `host_account_id`, `visitor_email_ciphertext`, `visitor_email_blind_index`, `token_hash`, `created_at`, `expires_at` (created+30d), `revoked_at`, `accepted_at`.
- Session: `id`, `invite_id`, `started_at`, `last_snapshot_at`, `ended_at`. Duration ≈ last_snapshot − start. No presence events.

### 3.8 `magic_links` / `email_change_tokens`

Single-use, 15-minute TTL, hashed token, consume-once in a transaction.

### 3.9 Species catalog (code, versioned)

Six species **[decision]**, coherent temperate woodland set:

1. **warbler** — small, high perch bias, bright motifs
2. **wren-like** — compact, talkative motifs
3. **finch** — mid perch, seed-curious
4. **thrush** — fuller body, lower calls
5. **tit** — social, chorus-prone
6. **nightjar-like** — nocturnal active; the one species that remains vocal at night

Each entry: silhouette SVG parts, default plumage palette + saturation mapping, call-grammar motif library, perch-zone prior, whether `nocturnal`.

Starter pair: worker picks two species with **dissimilar motif families and silhouettes** so the first chorus is distinguishable. User does not choose.

### 3.10 Personality seed

**[decision]** On adopt: sample each trait from `truncated_normal(μ_species, σ=0.08)` clipped to `[0.20, 0.55]`. New birds start mid-low so weeks of presence have room to move toward expressive. Never persist client-side seeds as truth.

### 3.11 What the user never sees

No API field for raw personality or `expressiveness` on any client-consumed resource except the private account export (export *does* include vectors, per PRD). Visual/audio/narration are the only live expressions.

---

## 4. API surface

Version prefix: `/v1`. JSON. Session cookie (`HttpOnly`, `Secure`, `SameSite=Lax`) holding an opaque token. Visit view uses a separate `visit` cookie scoped to `/v1/visit/*`.

All error bodies use matter-of-fact copy. No naturalist errors.

### 4.1 Auth and account

| Method | Path | Notes |
|---|---|---|
| `POST` | `/v1/auth/magic-link` | `{ email }`. Always 202. Rate limit **5 / hour / email blind-index** and **20 / hour / IP**. |
| `GET` | `/v1/auth/consume` | `?token=`. One-time. Issues session. Used link → 410. Expired → 410. Copy: link may have expired. |
| `POST` | `/v1/auth/logout` | Revoke this session. |
| `GET` | `/v1/me` | Account settings, device list (no email in logs). |
| `PATCH` | `/v1/me` | timezone hint, a11y prefs, `visit_notify_email`, name-less profile (there is no profile). |
| `GET` | `/v1/me/sessions` | |
| `DELETE` | `/v1/me/sessions/:id` | |
| `POST` | `/v1/me/email/request` | Sends verify to new address; old remains live. |
| `GET` | `/v1/me/email/confirm` | Commits swap. |
| `POST` | `/v1/me/export` | Enqueues export; email signed URL (15 min). |
| `POST` | `/v1/me/delete` | Soft-delete. |
| `POST` | `/v1/me/undelete` | Allowed while `now < hard_delete_after`. Signed-in page also offers this. |

Unsupported browser: static HTML, matter-of-fact, no app boot.

### 4.2 Host snapshot and events

`GET /v1/aviary/snapshot`

Returns:

```json
{
  "tick_version": 18421,
  "server_time": "…",
  "local_phase": "morning|midday|evening|night",
  "weather": { "kind": "clear|rain|wind", "until": "…" },
  "birds": [
    {
      "id": "…",
      "species": "warbler",
      "name": "pip",
      "mood": "content",
      "perch": { "zone": "front", "slot": 0 },
      "plumage": { "palette": "…", "saturation": 0.41 },
      "pose_seed": 8821,
      "call": { "phase_ms": 1204, "propensity": 0.33 },
      "offer_ready": true
    }
  ],
  "greeting": {
    "bird_id": "…",
    "absence_bucket": "brief|hours|days",
    "seed": 44,
    "stagger_ms": [0, 380]
  },
  "adoption": { "eligible": false, "species_hint": null },
  "narration": "a small grey bird is perched on the front rail, calling softly. …",
  "settle_suggested": false
}
```

Notes:

- `plumage.saturation` is a *display mapping* the renderer needs; do not label it. Alternatively send only a baked palette hex set so the number never appears **[decision: send baked palette colors, not the raw 0–1]**.
- `call.propensity` is a 0–1 hint for the audio scheduler, not a user-facing trait.
- `greeting` is present only when this snapshot is a *host visibility edge* (first snapshot after hidden/new session). Subsequent keepalives omit it so the greeting cannot replay.

`POST /v1/aviary/events`

```json
{
  "events": [
    {
      "client_event_id": "uuid",
      "type": "presence_ping|listen_in_start|listen_in_end|offer|settle|settle_undo|rename|focus_idle",
      "bird_id": "…",
      "occurred_at": "…",
      "payload": {}
    }
  ]
}
```

Validation:

- `offer.payload.kind ∈ { seed, song, pool }`; `song` includes `fragment_id` from the small library.
- Cooldown enforced server-side; extra offers ack but no-op (`accepted: false`).
- `rename.payload.name` only; personality untouched.
- Max 50 events / POST, 1 POST / 2s / session.
- **No personality fields accepted. Reject unknown keys strictly.**

`GET /v1/aviary/notebook?before=&limit=40` — chronological reverse, infinite scroll. Prose only.

`PATCH /v1/birds/:id` — `{ name }` only.

`POST /v1/aviary/adopt` — 409 if not eligible or at cap 7; 200 returns new bird id + snapshot. Species chosen server-side.

### 4.3 Presence pings

Client sends `presence_ping` every **30s** while all three conditions hold, plus on falling edge (`presence_end` as payload flag on the last ping or a dedicated `presence_end` type **[decision: dedicated type]**).

Ping payload: `{ visibility, focused, last_input_at }` — server re-checks nothing about the DOM; it trusts the client for those booleans but **drops pings that fail an internal consistency check** (e.g. `last_input_at` older than the activity window). Activity window **[decision]: 4 minutes**. Lean long so still watching counts.

### 4.4 Visit invitation flow

| Method | Path | Who |
|---|---|---|
| `POST` | `/v1/visits/invites` | Host. `{ email }`. Sends one-time link. Off-by-default means this is the only way in. |
| `GET` | `/v1/visits/invites` | Outstanding + recent. |
| `DELETE` | `/v1/visits/invites/:id` | Immediate revoke. |
| `GET` | `/v1/visits/log` | Email (decrypted here only), date, approx duration, outstanding invites. No badge, no unread. |
| `GET` | `/v1/visit/consume?token=` | Sets visit cookie or 410 matter-of-fact “visit no longer available.” |
| `GET` | `/v1/visit/snapshot` | Same visual fields as host snapshot **minus** greeting, adoption, notebook, offer_ready, and with `read_only: true`. |
| `GET` | `/v1/visit/ended` | Used after revoke/expiry on next pull. |

Visitor client:

- Renders ambient view only.
- No event POST (endpoint 403).
- No listen-in, offer, settle, notebook, top-bar host actions.
- Top bar: none, or a single matter-of-fact “viewing a friend’s aviary” is **rejected** — that would announce. **[decision]** Visitor chrome is empty except an accessibility-settings icon (captions / reduced-motion for *this* viewer) and a leave control that just navigates away.
- Snapshot pull on the same keepalive; revoke is visible within one pull (~20s). Optionally shorten visit pull to 10s.

Host is not notified unless `visit_notify_email` is true. Email copy is matter-of-fact, one line, no “they loved your birds.”

### 4.5 Snapshot pull triggers (client)

1. First navigation (bootstrap snapshot in HTML if present).
2. `visibilitychange` → visible.
3. `pageshow` after bfcache / resume.
4. Render-frame gap > **2s** (laptop sleep).
5. Keepalive every **20s** while visible.
6. After posting settle / offer / adopt (read-your-writes).

ETag / `tick_version`: if unchanged, `304` and client keeps interpolating.

### 4.6 Ambiguity: visit email vs “no email about the aviary”

PRD brief forbids emailing about the aviary; social file allows opt-in visit notifications. **[decision]** Opt-in visit email is the sole exception, off by default, never shown in onboarding, never a push. Auth, invite, and export emails are transactional, not aviary pings.

---

## 5. Simulation engine design

### 5.1 Tick loop

Cadence **[decision]: 60 seconds**. Catch-up: if `now - last_ticked_at` is N minutes (device-off accounts), run **min(N, 180)** virtual ticks in one worker job, but apply *time-of-day and weather* from the actual timeline, not 180 weather rolls. Cap 180 minutes of catch-up compute; beyond that, jump mood/phase to “now” and apply drift using *integrated presence already in the log only* (never invent presence).

Per claimed aviary:

1. Lock aviary.
2. Read unconsumed events in `(received_at, id)` order.
3. Fold events into a `SessionSignals` window (presence seconds, listen-in seconds per bird, offers per bird, settle flag).
4. Update `expressiveness` (fast-medium).
5. Apply personality **deltas** (slow).
6. Advance weather (rare).
7. Transition moods (personality-gated).
8. Recompute perch zone intents.
9. Maybe write a notebook entry (sparse).
10. Recompute adoption eligibility from `aviary.created_at`.
11. Bump `tick_version`, set `last_ticked_at`, mark events consumed.
12. Unlock.

Tick p99 budget: **≪ 5s**; alarm at 5s. Target p99 < 200ms per aviary excluding catch-up.

### 5.2 Presence integration

`presence_seconds` in a tick = overlap of ping coverage with the tick interval, only for pings that passed the three-signal rule. Isolated pings do not count a full minute; interpolate only between consecutive valid pings ≤ 45s apart. A single ping = 0s. This stops “one accidental mouse move” from minting a minute of drift.

Settle and tab-close: client sends `presence_end`. Missing end is fine — coverage simply stops.

### 5.3 Drift function

Traits `x ∈ [0,1]`. Per tick, for each trait:

```
delta = α_trait * (w_presence * P + w_listen * L_bird + w_offer_c * C + w_offer_b * B)
x := min(1, x + delta)
```

All weights ≥ 0. **No negative terms. No neglect term.**

**[decision] Calibration constants** (tune in staging against the instrument target; do not ship a “faster for demo” flag in prod):

| Input | Weight role |
|---|---|
| `P` | `presence_seconds / 3600` in this tick, shared across birds |
| `L_bird` | listen-in hours on that bird |
| `C` | 1 if this tick contains an *accepted* offer near this bird (curiosity) |
| `B` | 1 if an offer occurred with this bird as nearest (boldness), accept or not |

Per-hour coefficients so that **~20 min honest presence / day × 7 days** moves at least one trait by **≥ 0.015** (instrument) and **21 such days** moves perch/greeting/plumage enough to notice (~0.04–0.07 on the dominant axes).

**Per-session cap:** sum of deltas for a trait in any UTC day ≤ **0.008**. Stops a 6-hour sit from skipping the week.

Trait targeting:

- Presence → small + on all five, largest on plumage saturation and social warmth.
- Listen-in → social warmth + vocal frequency on the focused bird only.
- Offer accepted → curiosity.
- Offer near bird → boldness (smaller).
- Settle → **zero personality delta**; may nudge mood toward drowsy.

Low-pass feel comes from tiny α and the daily cap, not from a separate EMA of the vector. The stored vector *is* the filter state.

### 5.4 Expressiveness (ambient quietness)

```
e := e + k_up * P - k_down * dt_absent
e := clamp(e, 0.25, 1.0)
```

`k_down` is slow: two weeks away → near floor. Floor is still alive (calls continue). Greeting probability scales with `e * social_warmth * boldness`. This is how neglect is *quiet*, not *punished*.

### 5.5 Mood transitions

States: `wary, content, curious, drowsy, alert`.

Each tick, sample a transition matrix modulated by:

- Time of day in **account timezone**: morning → alert prior; dusk → drowsy; night → drowsy unless species.nocturnal.
- Weather: rain → lower vocal behavior, slight wary/content mix; wind → alert or wary by boldness.
- Recent offer accept → content / curious.
- Nearby bird in `wary` → raise wary prior (spread).
- High boldness → suppress wary.
- High curiosity → raise curious when an offer or weather change exists.

Mood persists across sessions. No snap-to-content on snapshot. Daily-ish reset = priors pull toward a time-of-day baseline, not a hard midnight wipe.

**[decision]** Do not expose mood names in UI. Renderer and narrator infer.

### 5.6 Perch intent

Zone chosen from mood × boldness:

- wary / low boldness → back
- content / high boldness → front
- drowsy → middle or back, low pose
- curious → middle, bias toward last offer location (front if pool/seed is out)

Slots: greedy assign to avoid stacking; if a zone is full, spill one zone back. Client interpolates path 4–8s. Tick may keep the same perch for many minutes; birds are not restless.

### 5.7 Bird-to-bird

Worker maintains a short `social_buffer` in aviary state (not user-visible):

- If bird A called in the last tick (inferred from propensity roll) and bird B has high warmth/vocal, mark `response_bias` for B.
- If A is wary, add wary pressure to others for 2–4 ticks.
- Chorus window: if ≥2 birds have high vocal × expressiveness and phase alignment < 800ms, set `chorus: true` in snapshot so the audio engine actually overlaps calls.

The worker can roll “did call this minute” with a seeded RNG from `(bird_id, tick_version)` so clients reconstruct the same chorus intent.

### 5.8 Call-grammar runtime (split)

**Server:** does not synthesize audio. It outputs `propensity`, `call_phase_ms`, `motif_family`, `mood`, `chorus`, and a `call_seed` per bird per tick.

**Client audio engine:** expands seed → motif sequence (see §9). Recognizability = species motif family + per-bird sticky timbre knobs derived from `bird.id` (not from drifting traits except mild pitch/timing from vocal frequency). Drift changes *how often* and *how readily they join*, not the identity of the voice.

### 5.9 Greeting plan

On host visibility edge, if no greeting was emitted in the last 90s:

- Rank birds by `boldness * expressiveness * greeting_mood_factor`.
- Pick top bird; if two are close, pick one, stagger a second by 200–700ms (never unison).
- `absence_bucket`: `< 20 min` glance; `20 min–12 h` short call; `> 12 h` re-orient (step forward or longer call).
- Seed must be unique per occurrence (`hash(tick_version, bird_id, last_presence_end)`).

Visitor snapshots never include a greeting plan. Visitors do not trigger greetings.

### 5.10 Offers

Three kinds. Nearest bird in front/middle with cooldown elapsed is the receiver; if none, the boldest eligible bird.

| Kind | Reaction (client, mood-shaped) | Drift |
|---|---|---|
| seed | approach / wait / ignore | curiosity if approach; boldness if near |
| song fragment | join / quiet / call-against | vocal + curiosity if join |
| still pool | drink / bathe / watch | curiosity if engage |

Cooldown **[decision]: 4 minutes per bird**, shared across kinds. Functional, not punitive. Song library: **6 fragments**, species-neutral, short motifs, shipped as synthesis scores not audio files.

### 5.11 Weather

**[decision]** Target ~3 short rains / week / aviary and ~2 wind passages. Poisson process in the tick, duration 8–20 minutes. Never storm, never snow. Effects expire with `weather_until`. Client ornaments (drops, leaf ripple) are local.

### 5.12 Notebook authoring

Generator runs inside the tick when:

- `now - notebook_last_written_at ≥ 2.5 days`, **or**
- noteworthy: first greeting-order swap this week, weather + hush, long quiet (≥ 40 min presence with low call rolls), adoption.

Hard sparsity: **max 3 entries / 7 days** even for heavy users. Templates are compositional (time-of-week, bird names, perch, weather, hush) in lowercase present-tense. Ban list: “you”, “session”, numbers-as-stats, “visited”, “streak”, “unlocked”, mood/trait labels.

Human review corpus of 40 gold examples in repo; snapshot tests for banned tokens.

### 5.13 Adoption pacing

**[decision]** Age gates from `aviary.created_at`:

| Bird count after adopt | Min aviary age |
|---|---|
| 2 | day 0 (starters) |
| 3 | 60 days |
| 4 | 150 days |
| 5 | 240 days |
| 6 | 360 days |
| 7 | 540 days |

Offer appears as a quiet top-bar affordance (not a modal, not a toast). Copy is naturalist (“another bird is near the aviary”) not “unlock.” System picks species not yet in the aviary if possible.

---

## 6. Sync model

### 6.1 Single writer

Only `sim-worker` updates personality, mood, perch intent, expressiveness, weather, notebook, adoption clocks. api may update names, settings, invites, and **insert** events.

There is no merge. There is no LWW. There is no client vector clock on birds.

### 6.2 How devices stay aligned

Laptop and phone each pull the same snapshot. Both append events into the same log. Tick applies events in receive order. Clock skew: prefer `received_at` for ordering; `client_occurred_at` is used only for presence coverage heuristics within a small skew bound (±2 min), else ignored.

If two devices are concurrently present (rare, allowed):

- Both may emit presence pings. Tick unions coverage (OR, not double-count overlapping seconds) **[decision]**.
- Listen-in on two devices: two events; drift can apply twice at listen weights — accept this; daily cap bounds it. Do not try to mutex listen-in across devices (that would be co-presence machinery).

### 6.3 Preventing the classic overwrite

Forbidden code paths (lint + review checklist + API schema):

- `UPDATE birds SET boldness = $client`
- Putting vectors in PATCH bodies
- Caching vectors in localStorage as source of truth
- “Reconcile” jobs that average two snapshots

Allowed: `UPDATE birds SET boldness = boldness + $delta` inside the worker only.

### 6.4 Snapshot cache

Redis `snapshot:{aviary_id}` invalidated on tick and on name/adopt. api may serve cache for keepalive 304s. After event POST that should be visible immediately (offer cooldown, name), api patches the cached projection’s non-canonical fields only, or busts cache.

### 6.5 Conflict / error surfaces

No personality conflicts by construction. User-visible failures:

- Magic-link expired / reused
- Session timeout
- Snapshot load failure
- Visit revoked

Copy stays matter-of-fact (PRD samples). Reload is the recovery. No “syncing your flock…” charm.

### 6.6 Offline

No offline mutation queue for offers/listen-in beyond a few seconds of send retry. If the tab cannot reach api, presence pings drop (correct: we cannot prove presence to the server). Renderer may continue from last snapshot locally; on reconnect, pull and interpolate — do not replay a local sim.

---

## 7. Frontend rendering pipeline

### 7.1 Stack **[decision]**

- TypeScript + Vite, small runtime (no React if it threatens the 2MB / 500ms budgets; if a UI library is used, confine it to settings/notebook routes that are code-split).
- **Recommended:** Preact or vanilla for chrome; **vanilla scene loop** for the aviary.
- Canvas 2D visual renderer (60fps control, no layout thrash).
- Transparent DOM overlay matching bird hit-rects for click/tap/focus.
- CSS variables for day/night palette driven by `local_phase` + local clock interpolation between ticks.

### 7.2 Scene composition

Three depth planes: background foliage/sky, perch+bird middle, occasional foreground branch. Parallax is a few pixels, not a showpiece.

Three perch zones with fixed anchors that **reflow with viewport width** (narrow: compress spacing; wide: more air). Constraint: every bird’s hit-rect stays fully on-screen at all supported viewports. Never crop a bird.

Color: calm naturalist palette from design system. No saturated UI accents in-scene. Chrome contrast AA.

### 7.3 First frame / load

Boot sequence:

1. HTML shell + critical CSS (sky wash) ships immediately.
2. If bootstrap snapshot present, construct SceneGraph and paint **before** waiting on fonts/audio/notebook. First bird ≤ 500ms budget.
3. If snapshot missing, show **quiet field** (soft sky, optional one faint leaf). No spinner. No logo pulse.
4. Place birds at snapshot perches with `pose_seed` so cycles start mid-preen / mid-scan.
5. Start audio context on first user gesture if the browser requires it; ambient calls wait, visuals do not. If autoplay audio is blocked, captions-on is *not* forced unless WebAudio is actually unavailable — **[decision]** autoplay-block is not “WebAudio unavailable.” Show no toast; calls begin after first key/pointer.

Empty-aviary (post-signup only): quiet field, then each starter fly-in once. Never again.

### 7.4 Idle micro-motion

Always-on while tab visible: preen, scan, head-tilt toward call sources, weight-shift. Mood maps:

| Mood | Motion |
|---|---|
| wary | back bias, more scan, less preen |
| content | preen, slower |
| curious | tilts to leaves/offers/calls |
| drowsy | low sit, fluffed, rare motion |
| alert | quicker head snaps, forward bias |

Motion is personality-keyed in *timing*, not in a named stat overlay. Never fully still.

Hidden tab: cancel rAF, suspend canvas, keep audio context suspended. Do not simulate locally.

### 7.5 Transitions

- Perch change: ease-in-out path along a short arc, 4–8s. Reduced-motion: crossfade poses between zones.
- Day/night: continuous palette lerp from local clock, not a stepped theme swap. Settle: force evening lerp over ~3s; undo within 5s of settle click reverses the lerp.
- Weather: rain/wind ornaments fade in/out; no flash.

### 7.6 Reduced-motion renderer

Triggered by `prefers-reduced-motion` or settings override. Same SceneGraph.

- Replace looping motion with 2–4 still poses per action, crossfade 1.5–3s.
- Flight → dissolve between perch stills.
- No leaf/feather drift.
- Keep slow lighting shifts (slower).
- Audio and drift unchanged.

This is a first-class aesthetic, shipped in v1, tested in CI with the media query forced.

### 7.7 Chrome

Thin top bar: account, accessibility, notebook, offer. Settle lives here too **[decision: settle icon in the same sparse set; PRD listed four then described settle from the top bar — include settle as a fifth icon]**.

Fade to near-transparent after **3s** cursor stillness; restore on pointer/key. No in-scene buttons, badges, tooltips, name labels.

Notebook: slide-over or full panel *above* the scene, not overlaid chrome on birds. Read-only list, infinite past.

Offer: small chooser (seed / song / pool); song opens the 6-fragment list. Not opened by clicking a bird (click = listen-in).

### 7.8 Responsive

Single screen, no pan/scroll/zoom of the scene. Phone: horizontal compress. Desktop: widen gaps. Aspect handling in renderer spec: letterbox the sky, never the birds.

---

## 8. Time-to-first-bird tactics

Tied to §7.3 and §11:

- HTML includes inlined sky CSS + optional `window.__SNAPSHOT__`.
- Edge/api injects last snapshot for authenticated HTML GET `/` (cookie). Snapshot ≤ a few KB.
- Split: `aviary-core` (scene + audio + events) vs `settings`, `notebook`, `visit-admin`, `account`.
- SVG parts for six species, atlas’d; no photo textures.
- No webfonts required for first bird; system fonts until chrome loads.
- Do not wait on WebAudio resume for first paint.

---

## 9. Audio pipeline

### 9.1 Procedural calls

WebAudio graph per visible session:

```
motifOsc/noise → formant filters → birdGain → chorusBus
ambientBus (other birds) ↘
listenInBus (focused)     → masterGain → destination
```

Each species has a motif library: 4–8 atoms (short FM/noise grains with pitch envelopes). A call is a grammar:

```
Call := Motif+ with gaps ~ U(species)
Motif := atom sequence with jittered duration/pitch
```

Per-bird sticky identity from `hash(bird.id)`: base pitch offset, vibrato depth, filter Q. Mood selects atom subsets (drowsy = fewer, lower; alert = sharper). Vocal frequency (via snapshot propensity) sets inter-call interval when unobserved and chorus join chance.

**Never loop a buffer as “the call.”** Grain buffers may be reused as wavetables; the *sequence* is unique per seed.

Recognizability test (QA): listeners identify bird A vs B at ≥ 80% after 10 minutes exposure, across two moods.

### 9.2 Chorus mixing

Independent schedulers per bird, phase from snapshot `call_phase_ms` so a late-joining client does not start everyone at t=0. Two birds calling = two live graphs, not two `<audio src>` tags.

### 9.3 Listen-in mix

On engage: focused bird gain eases **up** over **1.6s**; others ease **down** to an ambient floor (**[decision] floor = 0.28 linear**), never zero. Disengage: reverse, same ramp. Hard cuts are a bug.

Disengage: second click on same bird, click another bird (transfer), click empty scene, Escape / focus leaving the bird.

### 9.4 Caption generation

Grammar emits a parallel descriptor (`soft`, `three-note`, `rise`, `trill`, `sharp`, perch zone). Caption composer prints naturalist fragments:

> a soft three-note rise

Not “call_type=A3”. Generated at the same time as the audio schedule so caption matches what played. Fade with the call, near the bird, AA contrast.

### 9.5 WebAudio fallback

If `AudioContext` missing or create fails: master stays silent, **captions force-on**, no recorded files. If context exists but resume() is denied until gesture: wait silently without forcing captions.

Mute control: **[decision]** do not add a mute *announcement*; OS/browser mute and a11y captions cover it. If we need an in-app mute, put it in accessibility settings, not a toast.

### 9.6 Memory

Preallocate grain buffers. No per-call `new Float32Array` retained. Disconnect finished nodes. CI heap test: 30 min idle + periodic calls, heap delta ≈ 0 within noise.

---

## 10. Accessibility surfaces

Ship with v1. Not a follow-up.

### 10.1 Screen-reader narration

A single `aria-live="polite"` region (visually hidden) receives **prose**, not state lists.

Cadence:

- Idle: every **45s** **[decision]**, replace text (don’t stack).
- Immediate-ish: return-greeting, offer reaction, settle. Still observational sentences.
- Never high-frequency perch telemetry.

**[decision]** Generate narration **server-side** into `snapshot.narration` so voice stays consistent with the notebook author and visitors/hosts share the same sentence. Client may emit a one-shot local sentence for offer/settle if the next snapshot is too far; those use the same template pack.

Banned: “mood: content”, perch indices, trait numbers, “selected”, “button”.

Aviary birds in the focus overlay: accessible names are the bird’s given name plus a short species phrase, not a control announcement dump. Role: treat birds as a roving tabindex list, not a grid of buttons with “click to solo.”

### 10.2 Keyboard

- Tab: top-bar icons, then first bird.
- Arrows: move among birds.
- Enter: listen-in toggle on focused bird.
- Escape: exit listen-in; if offer panel open, close it; if settle pending undo window, do not auto-undo.
- Offer and settle fully keyboardable from top bar.

Focus ring: soft high-contrast outline specified by design, tested on midday and night palettes.

### 10.3 Captions

Settings + force-on when WebAudio unavailable. Runtime grammar, not a fixed map.

### 10.4 Contrast and settings

All user copy AA. Settings surface is matter-of-fact: reduced motion, captions, (optional) narration verbosity **[decision: no verbosity slider in v1 — one cadence]**. Link to privacy text.

### 10.5 Reduced-motion

See §7.6. Honor OS + override. Test with both.

---

## 11. Performance budgets and observability

### 11.1 Budgets

| Budget | Gate |
|---|---|
| Initial JS gzipped at first paint | **< 2MB** (aim < 800KB core) |
| Time to first bird visible | **< 500ms** mid-tier 4G |
| Idle motion | **60fps** on a 5-year-old mid laptop, including minute 30 |
| Heap | **no growth** over 30 min (CI) |
| Snapshot size | **low single-digit KB** |
| Tick p99 | alarm **> 5s**; target **< 200ms** |

### 11.2 What we measure

Synthetic browsers on a schedule (common geos): TTFB, first-bird mark, rAF long-task rate, audio context errors, snapshot 304 ratio, tick latency.

RUM (aggregate only): navigation timing, first-bird, frame timing histograms, audio errors, HTTP error rates, session-duration histogram **with no account or bird dimension**.

### 11.3 What we deliberately do not measure

- Per-bird offers, listen-in, presence minutes, trait values, mood distributions across users.
- “Average drift.”
- Visit funnels that identify hosts.
- Any metric that reconstructs a relationship.

Pipeline rule: sim DB credentials are not in the telemetry agent’s secret set.

### 11.4 Logging

`account_id` UUID only. Never email, never bird names in info logs (names can be PII-adjacent). Debug sampling off by default.

---

## 12. Rollout

### 12.1 Build sequence

1. **Foundations** — account UUID model, magic link, encrypted email, session revoke, soft-delete.
2. **Event log + snapshot schema** — no UI charm yet; golden tests for “client cannot write vectors.”
3. **sim-worker v0** — mood + perch + expressiveness; drift coefficients behind config; instrument harness for 7-day compressed time.
4. **Client scene v0** — quiet field, first-frame mid-pose, 60fps idle, reduced-motion sibling renderer.
5. **Audio v0** — two species grammars, chorus, listen-in ramps, caption descriptors, silent fallback.
6. **Interactions** — presence detector (three signals), offer, settle+undo, greeting planner.
7. **Notebook** — sparse author + banned-token tests + a11y live region sharing voice.
8. **Visits** — invite/revoke/read-only snapshot; no host ping.
9. **A11y pass** — keyboard, AA, narration cadence, captions, reduced-motion visual QA.
10. **Perf gates** — bundle, 500ms, 30-min heap, tick alarm.
11. **Dogfood** — internal accounts only, real-time (no accelerated drift in prod).

### 12.2 Bird-count ramp

Do not start dogfood at seven. Prod age gates stay as in §5.13. Internally, a **staging-only** clock offset may exist to test adopt; it must be impossible to set on prod accounts.

v1 launch: everyone starts at two. No marketing of “collect all six.”

### 12.3 Day-one instrumentation

Ship: tick latency, event ingest errors, magic-link consume rates (not emails), snapshot size, first-bird RUM, audio error counts, 404/410 on visit tokens, CI heap + fps.

Do not ship: engagement dashboards, DAU-as-success, streak-like funnels.

### 12.4 Drift calibration process

Before public launch, run a **time-compressed sim** in CI (virtual 21 days, scripted presence 20 min/day) asserting:

- Day 7: max trait delta ∈ [0.015, 0.04]
- Day 21: max trait delta ∈ [0.04, 0.12]
- Zero-presence 21 days: all traits **exactly unchanged**; expressiveness near floor; mood still cycles with TOD/weather
- Dual-device overlapping events: no trait decrease, no lost listen-in seconds beyond union rules

Human playtest: “do birds feel different after three weeks” — qualitative, no showing numbers to testers.

### 12.5 Feature flags

Allowed: visit feature kill switch, weather intensity, notebook writer on/off. **Not allowed:** “show personality debug,” “faster drift,” “welcome toast,” “streak experiment.”

---

## 13. Risks

### 13.1 Drift calibration

**Risk:** Too fast → Tamagotchi numbers; too slow → screensaver. Presence bugs silently accelerate the whole population.

**Mitigations:** three-signal presence; 4-minute activity window; ping interpolation rules; daily delta cap; CI compressed-time tests; no “tab open” shortcut; union not sum for multi-device presence; expressiveness separate from traits so quietness doesn’t require negative drift.

### 13.2 Sync correctness

**Risk:** A well-meaning client cache or LWW settings merge corrupts vectors. Dual-tab event double-count.

**Mitigations:** schema denylist; worker-only updates; idempotent `client_event_id`; daily caps; never persist vectors in localStorage; codeowners on `birds` update paths.

### 13.3 Audio uncanniness

**Risk:** Loopy grains, phasey stacked samplers, seven birds becoming mush, identical greetings.

**Mitigations:** grammar + sticky per-id timbre; no file loops; chorus as independent voices; cap 7; recognizability listening tests; greeting seeds unique; listen-in floor so the place doesn’t become a DAW.

### 13.4 Accessibility regressions

**Risk:** Live region spam; reduced-motion as “animations: none”; captions that don’t match audio; focus rings invisible at night; treating ARIA as a state dump.

**Mitigations:** designed RM renderer in the same PR as full-motion; narration cadence tests; caption-from-grammar unit tests; night-palette focus screenshots; a11y launch criterion equal to visual launch.

### 13.5 First-frame failure

**Risk:** Spinner culture, font blocking, audio blocking paint, 2MB of framework.

**Mitigations:** quiet field, snapshot-in-HTML, code-split settings, canvas scene without UI framework, budget CI.

### 13.6 Voice leakage

**Risk:** “Welcome back,” notebook-as-event-log, error copy in warbler-speak, visit badges.

**Mitigations:** copy lint for product vs system surfaces; banned strings in CI (`achievement`, `welcome back`, `streak`, `you visited`); no toast component in the aviary route.

### 13.7 Privacy / PII spray

**Risk:** Email as key in Redis, logs, invite URLs.

**Mitigations:** synthetic UUID everywhere; blind index for email lookup; hashed tokens in URLs only; telemetry credential split; export/delete actually drops events and vectors.

### 13.8 Social creep

**Risk:** Co-presence, “friend is here,” discovery.

**Mitigations:** visit endpoint cannot POST events; no visitor presence; host notify default off; no public list tables in the schema at all.

---

## 14. Testing strategy (executable)

- **Unit:** drift monotonicity, daily cap, presence coverage math, mood priors, notebook banned tokens, call grammar uniqueness, caption alignment.
- **Contract:** API rejects personality writes; visit snapshot shape omits greeting/events.
- **Property:** tick catch-up never invents presence.
- **E2E:** magic link, first-frame no spinner, listen-in ramp, settle undo 5s, revoke visit mid-session.
- **Perf CI:** bundle, heap 30m, fps budget on a throttled profile.
- **A11y CI:** axe on chrome/settings; keyboard path; live-region rate limiter.

---

## 15. Open decisions already closed in this plan

| Topic | Call |
|---|---|
| Presence activity window | 4 minutes |
| Tick | 60s |
| Mood set | wary, content, curious, drowsy, alert |
| Trait range | [0, 1] |
| Ambient quietness | `expressiveness` gain, not negative drift |
| Offer cooldown | 4 min / bird, all kinds |
| Species count | 6 including nocturnal |
| Age gates | 60 / 150 / 240 / 360 / 540 days |
| Renderer | Canvas 2D + DOM focus overlay |
| Narration | Server-authored in snapshot |
| Visit notify | Optional email only, default off |
| Settle in chrome | Fifth top-bar icon |
| Multi-device presence | Union of seconds |
| Snapshot keepalive | 20s |
| Listen-in ambient floor | 0.28 |
| No recorded audio | Unconditional |

These can be retuned in staging against calibration tests, not by product-surface experiments that violate non-goals.

---

## 16. Explicit non-implementation note

This document is the v1 plan only. It is not the product. No application code, assets, or infra are created as part of this phase.
