# Pocket Aviary — v1 implementation plan

This plan is an executable engineering brief for Pocket Aviary. It interprets the PRD into service boundaries, schemas, APIs, simulation math, render/audio pipelines, accessibility surfaces, rollout, and risks. It does not restate the spec; it decides how to build it.

**Product:** browser-only single-aviary companion. Two starter birds, cap of seven. Server is the only writer of personality. Clients pull snapshots and submit interaction events. Presence is the dominant drift input. Voice is naturalist on product surfaces and matter-of-fact on system surfaces.

**Decision rule used throughout:** when the PRD is silent, choose the option that (1) keeps the server as sole simulation writer, (2) refuses announcement/gamification surfaces, and (3) ships accessibility as a designed surface on day one.

---

## 1. Scope

### 1.1 In v1

- Web client for last two major versions of Chrome, Safari, Firefox, Edge.
- Magic-link auth, synthetic account UUID, one aviary per account.
- Server-side simulation tick (~60s), append-only interaction log, snapshot pull.
- Two starter birds selected by the system from a six-species pool; adoption by age of aviary up to seven.
- Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), mood enum, procedural call grammar, mood-shaped idle motion.
- Session interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle with 5s undo, field notebook (read-only, sparse).
- Presence accounting: `visibilityState === 'visible'` AND window focus AND pointer/key activity in the last N minutes.
- Multi-device sync as a property of server-canonical state (not a merge protocol).
- Visit invitations: opt-in per invite, read-only, revocable, 30-day unused expiry, silent visit log, notifications off by default.
- Account export (JSON emailed as download link), soft-delete 30 days then hard-delete.
- Accessibility: naturalist screen-reader narration, designed reduced-motion mode, runtime call captions, WCAG AA chrome, full keyboard path.
- Performance: gzipped first-paint JS < 2MB, first bird < 500ms on mid-tier 4G, 60fps idle on a 5-year-old laptop, no client memory growth over 30 minutes.
- Operational telemetry only: request/latency/error/session-duration histograms, render-frame and audio-context errors, tick latency. No per-bird or per-account interaction analytics.

### 1.2 Explicitly out of v1 (and not designed around)

Native apps; passwords/SSO; payments; multi-aviary or shared aviaries; customizable scenes; panning/zoom; catalog-style adoption; drag-to-place perches; personality numbers anywhere; streaks, badges, levels, scores, visit calendars, “days visited”; Tamagotchi hunger/death/distress meters; push/email about the aviary (except magic links, export links, and the optional visit-notification toggle); public discovery, profiles, follows, comments, chat, avatars, leaderboards, co-presence; recorded-audio fallback; last-write-wins personality writes; aggregating per-bird events for training or population analysis.

### 1.3 Calls made where the PRD is open

| Open item | Decision |
|---|---|
| Tick cadence | 60s nominal. Catch-up ticks may batch missed minutes after outage, but apply drift with time-dilated presence (presence only from recorded events, never invented). |
| Presence activity window | Start at **4 minutes** of last pointermove/keypress. Calibrate 3–6 min in dogfood. Prefer longer so still-watching is not dropped. |
| Mood set | `wary`, `content`, `curious`, `drowsy`, `alert`, plus `settled` as a lighting/session overlay that biases birds toward drowsy without replacing stored mood. |
| Trait range | Each trait is `float32` in `[0.05, 0.95]`. Seed from species prior + small per-bird noise. Floor 0.05 so no bird is visually “off.” |
| Starter pair | Server picks two *different* species with complementary priors (one higher boldness, one higher vocal) so the first chorus is readable. |
| Third+ bird pacing | Offer appears at aviary age **45 days**, then **120 days**, **240 days**, **400 days**, **600 days**. Age of aviary, not attention. Soft fly-in once accepted. |
| Species pool (6) | warbler, wren-like, finch, dove-like, thrush, nightjar-like (the night-active caller). Names are internal codes; UI uses naturalist common names. |
| Song-fragment library | 8 short motifs, ~2–4s each, synthesized from the same oscillator vocabulary as calls (not recorded files). |
| Offer cooldown | 4 minutes per bird per offer type. Cross-type independent. |
| Notebook sparsity | Target ~1 entry / 3 days of presence for a regular visitor; burst only on noteworthy events (first-greeter flip, first rain of the week, new bird fly-in). Hard cap 1 entry / 12 hours even for hyper-active users. |
| Snapshot keepalive | Every 20s while visible+focused; immediately on `visibilitychange` → visible and on rAF gap > 2s (suspend/resume). |
| Timezone | Stored on account, updated from client `Intl` on each signed-in session start. Tick uses last-known TZ. |
| Visit duration | Measured as visitor snapshot-session length on the server (first snapshot to last keepalive/disconnect). Approximate, shown as “about N minutes.” |

---

## 2. Architecture

### 2.1 Service shape

Four deployable units, one region at v1, Postgres as source of truth.

```
                    ┌─────────────┐
   magic-link email │  Mailer     │  (transactional only)
                    └──────▲──────┘
                           │
┌──────────┐   HTTPS    ┌──┴──────────┐  consume  ┌────────────────┐
│  Web SPA │◄──────────►│  API        │──────────►│  Sim worker    │
│  (edge)  │  snapshot  │  (stateless)│  events   │  (per-minute)  │
└──────────┘  + events  └──┬──────────┘           └───────┬────────┘
                           │                              │
                           ▼                              ▼
                    ┌──────────────┐              ┌──────────────┐
                    │  Postgres    │◄─────────────│  same DB     │
                    │  accounts,   │   exclusive  │  tick writes │
                    │  aviaries,   │   row lock   │  vectors,    │
                    │  events,     │   per aviary │  moods,      │
                    │  visits      │              │  notebook    │
                    └──────────────┘              └──────────────┘
                           ▲
                           │ operational metrics only
                    ┌──────┴───────┐
                    │  Telemetry   │  isolated; no sim-table replicas
                    └──────────────┘
```

- **Web SPA:** static assets on CDN. HTML document includes a tiny inline bootstrap and, when a session cookie is present, an **edge-injected state bootstrap** (see §8.3) so first bird can paint without waiting on a second RTT when cache is warm.
- **API:** auth, snapshot read, event append, account/settings, visit invite/revoke, export job enqueue, deletion. Horizontally scalable. **Never writes personality vectors or moods.**
- **Sim worker:** one logical loop. Claims aviaries due for tick (`FOR UPDATE SKIP LOCKED`), applies drift/mood/weather/notebook, writes canonical state. Runs whether or not a client is connected.
- **Mailer:** magic links, export download links, visit invites, optional visit notifications. No aviary “come back” mail. Ever.

No Kafka at v1. The interaction event log is a Postgres table. If tick lag grows, shard workers by `aviary_id` hash; do not introduce a second source of truth.

### 2.2 Client/server split

| Concern | Owner |
|---|---|
| Personality vector, mood, perch intent, weather phase, notebook entries, bird identity | Server (canonical) |
| Presence classification, listen-in, offer, settle | Client detects → event; server interprets |
| Idle micro-motion, leaf/feather ornaments, interpolation, call synthesis, captions | Client (derived from snapshot + local clock) |
| Narration prose | Generated from snapshot (client first; server can stamp a `narration_seed` so visit + host stay consistent) |
| Day/night palette | Client, from user local time + snapshot `settled_until` |

Clients never tick. If the tab is hidden, the client stops rAF and WebAudio (or suspends the context) and stops presence pings. Simulation continues on the server.

### 2.3 Render pipeline boundary

```
snapshot (authoritative poses, moods, weather, call schedule hints)
        │
        ▼
SceneDirector  ── interpolates perch targets, greeting plan, offer reactions
        │
        ├─ VisualRenderer (Canvas2D or WebGL2; see §8)
        │     poses, plumage, lighting, weather, ornaments
        ├─ AudioEngine (WebAudio)
        │     motif grammar → oscillators → chorus bus → listen-in mix
        ├─ CaptionLayer (DOM, optional)
        ├─ NarrationLayer (ARIA live, slow cadence)
        └─ ChromeLayer (top bar, settings, notebook, account) — separate React/Svelte tree
```

The aviary scene has **zero interactive DOM inside the canvas** except an invisible focus-proxy list of birds for keyboard/AT. Chrome never overlays badges on birds.

### 2.4 Privacy architecture (load-bearing)

- Account identifier everywhere except the account row: `account_id` UUID.
- Email encrypted at rest, decrypted only for mailer and visit-log display to the host.
- Simulation DB is **not** replicated to the analytics warehouse.
- Telemetry pipelines subscribe only to API/edge metrics and anonymized RUM. Metric definitions are code-reviewed against a denylist: `bird_id`, trait values, offer types, listen-in targets, notebook text, visitor emails.
- Per-bird events exist solely to drive that account’s next ticks.

---

## 3. Data model

Postgres, UUID PKs, `timestamptz` everywhere. Soft-deleted accounts remain until `purge_after`.

### 3.1 `accounts`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid PK | synthetic, generated at insert |
| `email_ciphertext` | bytea | unique via deterministic blind index `email_lookup` |
| `email_lookup` | bytea unique | HMAC of normalized email; not reversible |
| `timezone` | text | IANA, default `Etc/UTC` until first session |
| `created_at` | timestamptz | aviary age clock starts here (or `aviaries.created_at`; same instant) |
| `deleted_at` | timestamptz null | soft delete |
| `purge_after` | timestamptz null | `deleted_at + 30d` |
| `visit_notify` | bool | default false |
| `reduced_motion` | bool null | null = follow OS; true/false override |
| `captions` | bool | default false; forced true on WebAudio fallback |
| `locale` | text | for date words in notebook (“tuesday”) |

### 3.2 `sessions`

`id`, `account_id`, `created_at`, `last_seen_at`, `user_agent_hash`, `revoked_at`. Token is a random 32-byte value; only SHA-256 stored. Listed in settings as device rows; revoke sets `revoked_at`.

### 3.3 `magic_links`

`id`, `account_id` (nullable until first-time create), `email_lookup`, `token_hash`, `expires_at` (+15 min), `consumed_at`. Consume in a transaction: insert session, mark consumed. Replay of a consumed token is an error, not a new session.

First-time click creates account + aviary + two birds in the same transaction.

### 3.4 `aviaries`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid PK | |
| `account_id` | uuid unique | one aviary per account |
| `created_at` | timestamptz | age for next-bird offers |
| `weather` | jsonb | `{kind, started_at, ends_at}` or null |
| `settled_until` | timestamptz null | client settle is ephemeral; persist only if we need visit consistency — **decision:** persist `settled_at` for 15 minutes so a visitor/other device sees evening if the host just settled |
| `last_ticked_at` | timestamptz | |
| `tick_version` | bigint | monotonic; snapshot ETag |
| `next_bird_offered_at` | timestamptz null | when the age-gated offer became available |
| `next_bird_accepted_at` | timestamptz null | |

No “visit count,” no streak columns, no last-visit trophy fields.

### 3.5 `birds`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid PK | stable identity; never recycled |
| `aviary_id` | uuid | |
| `species_id` | text | one of six codes |
| `name` | text | user-assigned; default suggestion |
| `sort_adopted` | int | 0..6 |
| `boldness` … `curiosity` | real | five traits, `[0.05, 0.95]` |
| `mood` | text | enum |
| `mood_since` | timestamptz | |
| `perch_zone` | text | `front` \| `mid` \| `back` |
| `perch_slot` | smallint | discrete slot inside zone to avoid overlap |
| `call_seed` | bytea | 16 bytes; locks motif family for life |
| `adopted_at` | timestamptz | |

Indexes: `(aviary_id)`. Unique `(aviary_id, sort_adopted)`. Check `count(*) <= 7` enforced in application + trigger.

**Hard rule:** personality columns are updated only by the sim worker.

### 3.6 `interaction_events` (append-only)

| Column | Type | Notes |
|---|---|---|
| `id` | bigserial / uuid | |
| `aviary_id` | uuid | |
| `account_id` | uuid | host only; visitors never insert |
| `device_session_id` | uuid | |
| `type` | text | see below |
| `bird_id` | uuid null | |
| `payload` | jsonb | type-specific, no PII |
| `client_ts` | timestamptz | for ordering hints |
| `server_ts` | timestamptz | authoritative |
| `consumed_tick` | bigint null | set when tick processes the row |

Types:

- `presence_sample` — `{active_ms, window_s}` accumulated since last sample (client sends ~every 30s while all three presence conditions hold)
- `listen_in_start` / `listen_in_end` — `{bird_id, duration_ms}` on end
- `offer` — `{kind: seed|song|pool, target_bird_id?, motif_id?}`
- `settle` / `unsettle`
- `rename` — `{bird_id, name}` (also applied immediately by API to `birds.name`; not a personality write)
- `adopt_accept` — when user accepts an age-gated new bird
- `greeting_ack` — optional client note that greeting played (for notebook “greeted first”; server can also infer from session-open + snapshot)

Events are **facts**, never “set boldness to X.”

Retention: keep events for 90 days after consume for audit/debug of that account only; not warehouse-copied. After 90 days, delete consumed rows. Unconsumed rows are never deleted.

### 3.7 `notebook_entries`

`id`, `aviary_id`, `observed_on` (date in account TZ), `body` (text, lowercase naturalist), `created_at`. No edit, no delete (except account purge). Infinite scroll; paginate by `created_at desc`.

### 3.8 `visits` and `visit_invites`

`visit_invites`: `id`, `host_account_id`, `aviary_id`, `invitee_email_ciphertext`, `invitee_email_lookup`, `token_hash`, `created_at`, `expires_at` (+30d), `revoked_at`, `first_used_at`.

`visit_sessions`: `id`, `invite_id`, `started_at`, `last_seen_at`, `ended_at`. Used only for the host visit log. **No presence events written against the host aviary.**

Visitor auth is the invite token (separate cookie `visit_session`), not an account. A visitor who also has an account still cannot interact with the host aviary.

### 3.9 `species` (static config, versioned in repo)

Per species: silhouette SVG set, default plumage base hue/sat, motif library IDs, night-active flag, prior trait means, display common name.

### 3.10 Personality vector representation

```
Personality = {
  boldness: f32,
  social_warmth: f32,
  vocal_frequency: f32,
  plumage_saturation: f32,
  curiosity: f32
}
```

Never sent to analytics. Snapshot **does** include the five floats to the **owner client** so rendering (perch bias, plumage, call rate) can run offline between snapshots. The UI must not bind them to any text, ARIA attribute, or debug panel in production builds. Strip them from the **visitor** snapshot; visitor renderer uses server-provided derived fields only (`plumage_sat_render`, `call_rate_hint`, `perch_zone`) so we never leak a numeric “stat sheet” through a shared link.

### 3.11 Presence events vs presence-time

Client computes a running `active_ms` only while all three conditions hold. It flushes `presence_sample` every 30s and on pagehide. Server stores samples; tick sums `active_ms` since last tick into `presence_seconds`. A background tab produces **zero** samples.

Calibrate: a user who watches 20 minutes/day should accumulate ~15–20 minutes presence-time, not 24 hours.

---

## 4. API surface

All owner APIs require session cookie. JSON, small payloads. Matter-of-fact error bodies.

Idempotency: `POST /events` accepts `Idempotency-Key` (client UUID per event) unique per account.

### 4.1 Auth

- `POST /auth/magic-link` `{email}` → 202. Rate-limit per `email_lookup` (e.g. 5 / 15 min) and per IP. Always 202 to avoid account enumeration.
- `GET /auth/consume?token=` → set session cookie, 302 to `/`. Invalid/expired/used: HTML matter-of-fact page, not a toast in the aviary.
- `POST /auth/sign-out`
- `GET /account` / `PATCH /account` — timezone, notify toggle, a11y prefs, name-less profile.
- `GET /account/sessions` / `DELETE /account/sessions/:id`
- `POST /account/email/change` + verify link (old email works until new verifies)
- `POST /account/export` — enqueue job; email download link (15 min, single-use)
- `POST /account/delete` / `POST /account/restore`

No “welcome” payload. First-time consume returns the same snapshot shape as every other load.

### 4.2 State pull

`GET /aviary/snapshot`

Response (~2–8 KB):

```
{
  tick_version,
  server_time,
  timezone,
  weather,
  settled_until,
  lighting_hint,          // derived: morning|mid|evening|night|settled
  next_bird_available,    // bool; no “day 45!” copy
  birds: [{
    id, name, species_id,
    mood, perch_zone, perch_slot,
    pose_hint,            // preen|scan|tilt|shuffle|sleep|greet|approach
    motion_phase,         // 0..1 so client can start mid-action
    plumage,              // derived color params, not raw vector in visitor mode
    traits?,              // owner only, render use
    call: { seed, rate_hz_hint, last_motif_id },
    cooldown: { seed: ts, song: ts, pool: ts }
  }],
  greeting: {             // computed on first snapshot of a new owner session
    bird_id, kind, absence_bucket, stagger_ms[]
  } | null,
  notebook_unread_hint: false  // NEVER a badge count; always false. Notebook is not badged.
}
```

`ETag: tick_version`. `GET` with `If-None-Match` can 304.

Also: `GET /aviary/snapshot` on visit token → same visual fields, `greeting: null`, no traits, no cooldowns, `readonly: true`.

Triggers for client refetch: visibility visible, rAF gap > 2s, keepalive 20s, after posting an event that should be visible immediately (offer, settle).

### 4.3 Interaction events

`POST /events` `{ type, bird_id?, payload, client_ts, idempotency_key }`

- Validate type, bird belongs to aviary, offer cooldown server-side (client also gates).
- Reject visitor sessions with 403.
- Return `{ accepted, server_ts, snapshot_hint? }`. For `offer` / `settle` / `rename` / `adopt_accept`, include a **partial snapshot delta** so the client does not wait a full minute for the reaction. **Personality still unchanged** until tick; the delta is mood/perch/pose/cooldown/lighting only. Immediate mood nudge from an accepted offer is allowed as a *fast-path write by the API only for mood/pose*, **or** (cleaner) the API inserts the event and the sim worker exposes a `tick_now(aviary_id)` for interactive events.

**Decision:** interactive events enqueue a **priority tick** for that aviary (debounced 250ms). Priority tick may update mood, perch intent, offer reaction, settle lighting, and notebook-worthy flags. It applies only a **tiny** drift slice (see §5) so clicking cannot visibly move traits in-session.

`rename` updates `birds.name` in the API transaction (names are not simulation state).

### 4.4 Notebook

`GET /notebook?before=&limit=30` — chronological pages, oldest retained forever (until account purge). No POST.

### 4.5 Adoption / naming

- Starter names: `GET /aviary/snapshot` after first consume includes two birds with suggested names; `POST /events` `rename` or `PATCH /birds/:id {name}`.
- Age-gated bird: snapshot `next_bird_available`. Top-bar offer affordance gains a third item **only then**, copy in naturalist voice (“a new bird is at the edge of the scene”). `POST /events` `adopt_accept` → worker inserts bird, fly-in pose. No catalog.

### 4.6 Visit flow

- `POST /visits/invites` `{email}` — host only. Creates token, emails one-time link. Default off means this is the only way a visit exists.
- `GET /visits/invites` — outstanding + recent, for settings.
- `POST /visits/invites/:id/revoke` — immediate. Next visitor snapshot → 410 with matter-of-fact body: “This visit is no longer available.”
- `GET /visit/consume?token=` — sets visit cookie, 302 to `/visit`.
- `GET /visit/snapshot` — read-only snapshot; keepalive does not write presence.
- `GET /visits/log` — host settings: invitee email (decrypted for host), date, approximate duration, outstanding invites. No badge on the settings icon.

Invite unused 30 days → cron sets expired; consume shows the same 410 surface.

Optional notify: if `visit_notify`, mailer sends a single matter-of-fact email on first snapshot of a visit session. No push. Not mentioned in onboarding.

### 4.7 Errors (voice)

```
{ "error": "magic_link_expired", "message": "We couldn't sign you in. The link may have expired. Try requesting a new link." }
{ "error": "session_expired", "message": "Your session timed out. Sign in again to keep watching." }
{ "error": "snapshot_failed", "message": "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." }
{ "error": "visit_unavailable", "message": "This visit is no longer available." }
```

Unsupported browser: static HTML, same register.

No naturalist phrasing on these surfaces. No toasts inside the aviary canvas.

---

## 5. Simulation engine design

### 5.1 Tick loop

Every 60s, worker selects aviaries where `last_ticked_at < now() - 55s` OR a priority flag is set, `LIMIT N`, `SKIP LOCKED`.

Per aviary, in one transaction:

1. Read birds + unconsumed events with `server_ts` in `(last_ticked_at, now()]`, ordered by `server_ts, id`.
2. Sum presence seconds (clamp per-hour, see below).
3. Apply weather start/end.
4. Advance mood for each bird.
5. Apply drift deltas (monotonic, clamped).
6. Choose perch zones from mood × boldness + occupancy.
7. Advance call-schedule hints (next expected call window).
8. Maybe write one notebook entry.
9. Set `last_ticked_at`, increment `tick_version`, mark events consumed.
10. If catch-up (`now - last_ticked_at > 3 min`), run **virtual minutes** for mood/time-of-day/weather only. Drift uses only real presence samples — never “they would have been watching.”

### 5.2 Drift function

Traits move **only up** (toward expressive). Neglect does not decrement.

Let `P` = presence seconds in this tick (or summed over catch-up).  
Let `L_b` = listen-in seconds on bird `b`.  
Let `O_accept` = 1 if bird accepted an offer this window, else 0.  
Let `O_near` = 1 if an offer was placed while this bird was front/mid (approach opportunity).

Per-week calibration target: ~20 min presence/day × 7 ≈ 8400s/week should move a trait by **~0.02–0.04** (measurable in fixtures, not visible session-to-session). Three weeks ≈ **0.06–0.12**, enough to change perch bias and plumage richness.

```
ε = 1e-4
Δ_presence = k_p * log1p(P / 60)          # diminishing return inside a tick
Δ_listen   = k_l * log1p(L_b / 30)
Δ_offer_c  = k_c * O_accept
Δ_offer_b  = k_b * O_near

boldness           += clamp(Δ_presence * 0.5 + Δ_offer_b, 0, max_tick)
social_warmth      += clamp(Δ_presence * 0.4 + Δ_listen, 0, max_tick)
vocal_frequency    += clamp(Δ_presence * 0.3 + Δ_listen * 0.8, 0, max_tick)
plumage_saturation += clamp(Δ_presence * 0.6, 0, max_tick)
curiosity          += clamp(Δ_presence * 0.3 + Δ_offer_c, 0, max_tick)

trait = min(0.95, trait + Δ)
```

Initial constants (tune in sim harness, not in prod by gut):

- `k_p = 0.0018`, `k_l = 0.0025`, `k_c = 0.003`, `k_b = 0.002`
- `max_tick = 0.004` on a 60s tick; priority ticks use `max_tick = 0.0004`

**Ambient quietness (not negative drift):** greeting probability and unobserved call rate use an **expression gain**:

```
gain = 1 - exp(-presence_hours_28d / τ)    # τ ≈ 8 hours of presence over 28 days
unobserved_call_rate = base(species, vocal_frequency) * (0.35 + 0.65 * gain)
greet_first_weight   = boldness * social_warmth * (0.4 + 0.6 * gain)
```

A neglected bird keeps its traits but `gain` decays toward 0.35 floor with **time since last presence**, not by lowering traits. Return presence raises `gain` faster than traits move. This is how “quieter, not mistrustful” is implemented.

Store `presence_hours_28d` as a rolling server aggregate updated each tick (from events), not a client number.

Settle: ends presence window; **zero** trait delta; small mood push toward `drowsy`.

### 5.3 Mood transitions

Daily-ish reset is a **soft attractor**, not a midnight snap.

Each tick, compute `mood_logits` from:

- time-of-day in account TZ (alert dawn, content midday, drowsy dusk, settled/sleep night except nightjar → alert)
- weather (rain: −vocal expression, +wary; wind: split alert/wary by boldness)
- recent events (accepted offer → content; listen-in → curious; alarm-call from neighbor → wary)
- personality as prior (high boldness down-weights wary; high curiosity up-weights curious)

Transition only if the winning mood exceeds current by a margin **or** `mood_since` older than 2–6 hours (sampled). Persist across sessions. Never force `content` on tab open.

Bird-to-bird: if any bird enters `wary` via alarm, neighbors get a temporary wary bias for 2–3 ticks. Chorus: if two+ birds have high vocal and overlapping call windows, tag `chorus_active` for audio/narration.

### 5.4 Call-grammar runtime (server hints + client synth)

Server does not render audio. It maintains:

- `call_seed` (lifetime)
- `rate_hz_hint` from vocal_frequency × gain × mood × weather × time-of-day
- optional `next_call_at` (server time) so devices stay loosely aligned

Client grammar (per species motif library):

- Motifs: 4–7 short pitch envelopes + rhythm cells.
- Combine 1–3 motifs with jitter in inter-onset interval, pitch (±30 cents), and duration (±12%).
- Mood colors the grammar: wary = shorter, sharper; drowsy = longer, lower; curious = more interval leaps.
- **Recognizability:** species + `call_seed` pick a stable timbre (formant / filter cutoff / hop rate) that does not change with mood. Mood/personality change timing and intensity, not identity.

Chorus is simultaneous independent voices on one master bus, not loop stacking.

### 5.5 Perch and idle intent

Each tick assigns `perch_zone` via softmax over {front, mid, back} with weights from boldness, mood (wary→back, curious/alert→front), occupancy penalty, and night (most species → back/sleep slot).

Idle `pose_hint` distribution:

| Mood | Dominant poses |
|---|---|
| wary | scan, still-high |
| content | preen, shuffle |
| curious | tilt, scan |
| drowsy / night sleep | fluff, low-sit |
| alert | scan, hop-in-place |

Client runs continuous micro-motion around that hint; server does not send frame-by-frame animation.

### 5.6 Weather

Poisson process: expected **3 short rains / week / aviary**, **2 wind gusts / week**, seeded by `aviary_id` + date so host and visitor agree. Duration 4–12 minutes. Never storms/snow. Effects expire with `ends_at`.

### 5.7 Greeting plan (computed at session start, not every tick)

On first owner snapshot after a new `session` (or after absence > 90s hidden):

1. `absence_bucket`: `<10m` glance | `10m–6h` short call | `6h–48h` re-orient | `>48h` longer call ± second bird response.
2. Choose greeter by `greet_first_weight`; others may follow with `stagger_ms` in 400–1800ms, never unison.
3. Procedural `kind` from greeter mood × boldness × bucket. Store `greeting_id` on the session so refresh does not replay a second full greeting; a visibility-return after coffee uses a lighter glance variant.

### 5.8 Notebook writer

Separate scorer after mood/drift:

Candidates: first-greeter change vs last 7 days; long preen window; rain + low calls; new bird; first front-perch for a historically back bird (infer from perch history table, **not** from exposing numbers).

Emit at most one entry, naturalist template + filled specifics (names, weekday in account locale, perch language). Reject templates that mention the user, visit streaks, or trait deltas.

### 5.9 New bird insertion

When age threshold crossed, set `next_bird_available`. On `adopt_accept`, insert bird with new UUID, species ≠ existing if possible, seeded traits, `call_seed` random, perch back, pose fly-in. Identity is that row forever.

### 5.10 Test harness (required)

Deterministic sim fixture: fixed clock, scripted events, assert:

- 7 days regular presence → each trait Δ in `[0.015, 0.05]`
- 7 days zero presence → traits unchanged, `gain` down, call-rate hint down
- two devices’ events interleave → commutative trait result (additive deltas)
- client cannot submit absolute traits (API reject test)
- 8th bird rejected
- visitor events rejected
- notebook does not emit more than cap

---

## 6. Sync model

### 6.1 Single canonical record

Laptop and phone are two snapshot readers. There is no CRDT, no client personality cache that survives as authority, no LWW.

Local client cache (IndexedDB): last snapshot for **quiet-field → first paint** only. On load, paint cached birds mid-motion immediately, then reconcile with network snapshot. If `tick_version` jumped, interpolate to new perches; do not keep stale traits if server differs.

### 6.2 Conflict prevention

- Personality: only worker writes; events are additive facts; processed in `server_ts, id` order.
- Names: last API write wins (user intent, not drift).
- Settings: last API write wins.
- Settle: last event wins for `settled_until`.
- Offers: server cooldown is authoritative; duplicate idempotency keys collapse.

A phone session that started from an old snapshot cannot overwrite morning drift: it never sends vectors.

### 6.3 Multi-active devices

If two owner devices are visible at once, **both** may send presence samples. Cap: `min(sum(active_ms), wall_clock_ms * 1.15)` per tick so dual-device watching cannot 2× drift. Listen-in from two devices on different birds: both count (attention to two birds). Same bird: take max duration, not sum.

### 6.4 Visit isolation

Visitor keepalives update `visit_sessions.last_seen_at` only. Host sees no overlay. Host snapshot unchanged.

### 6.5 Failure surfaces

Magic-link replay, mid-write timeout, 5xx on snapshot → matter-of-fact copy in §4.7. Do not invent a “sync conflict merge UI” for personality; it cannot diverge. The only user-facing “conflict” is auth/session/load failure.

---

## 7. Frontend rendering pipeline

### 7.1 Stack

- **Chrome / settings / notebook / auth:** Svelte or Preact (small). Code-split settings, visit invite, account, notebook.
- **Scene:** WebGL2 if available, Canvas2D fallback. Prefer **Canvas2D + SVG silhouettes** for v1 if it hits 60fps and bundle; escalate to WebGL2 only if overdraw fails on the 5-year laptop. Decision gate in week 2 of render spike.
- **No** entry animation, spinner, or fade-from-static.

### 7.2 Scene composition

Three depth planes: background foliage/sky, perch+bird mid, occasional foreground leaf. Subtle parallax (max 4–6px at 16:9). Three perch zones with 2–3 slots each; slot assignment from snapshot.

Responsive: compute a letterboxed **safe stage** that always contains all slots. Narrow viewports compress inter-perch X; never crop a bird. No user zoom/pan.

Palette: calm naturalist; lighting multiply from local time curve (sunrise warm-up, midday high, evening warm, night dim). Settle forces evening curve over ~3s.

Weather: rain as low-contrast streaks; wind as leaf ripple. Never assertive.

### 7.3 Mid-action first frame

Bootstrap:

1. Paint quiet field (soft sky, 1–2 faint leaf shaders) from CSS/inline — **0 JS heavy**.
2. As soon as snapshot or valid cache exists, instantiate birds at `motion_phase` so preen/call is mid-cycle.
3. Start rAF and audio (after gesture if the browser requires it; until then captions-ready silence is OK — do not show a “click to start” modal; a first pointer/key anywhere unsuspends AudioContext).

Empty aviary only exists between first-time naming and first bird insert; then one soft fly-in. Never again.

### 7.4 Idle micro-motion

Continuous, personality- and mood-keyed, never paused while visible. Hidden tab: cancel rAF. On return: snapshot refresh + resume mid-phase (do not restart pose 0).

Reduced-motion: see §9.

### 7.5 Transitions

Perch changes: ease-in-out path along a short arc (~1.2–2.0s). Offers: seed/pool as scene props with bird approach or ignore per mood/curiosity. Song fragment: audio-only + head tilt / join / quiet.

Settle: 3s lighting + call gain down. Any pointer/key in 5s → `unsettle` event, reverse lighting. After 5s, click still un-settles (re-engage) but is a new event, not the undo window.

### 7.6 Top bar

Icons only: account, accessibility, notebook, offer. After ~3s cursor/keyboard stillness, fade to ~15% opacity; restore on activity. No badges, no unread dots, no bird labels in the scene.

Offer menu: seed, song (submenu of 8 fragments named naturally), still pool. Not opened by clicking a bird.

### 7.7 Quiet field vs spinner

Any wait (cold snapshot, slow 4G): keep quiet field. Never a circular spinner. If snapshot fails, matter-of-fact inline message in chrome, not over birds.

---

## 8. Audio pipeline

### 8.1 Synthesis

WebAudio graph per bird voice:

`motif scheduler → oscillator/noise source → formant filter (seed-stable) → gain → bird bus`

Master: `Σ bird buses → chorus compressor (gentle) → master gain`

No sample files for calls. Motif libraries are numeric tables in JS (<50 KB).

### 8.2 Listen-in mix

On engage (click/tap/Enter on focused bird):

- focused bird gain → 1.0 over **1800ms**
- others → 0.28–0.40 over **1800ms** (never 0)

Disengage (second activate, other bird, empty click, focus leave): reverse ramp **1800ms**. Switching birds: crossfade, no hard cut.

### 8.3 Chorus

Independent schedulers; slight humanizing delay. Avoid identical phase by seed-offset LFOs. Master bus ducking only when >3 simultaneous onsets.

### 8.4 Settle / night

Global call rate and master gain drop. Nightjar species exempt from sleep mute.

### 8.5 Fallback

If `AudioContext` missing or `resume()` denied after first gesture: stay silent, **force captions on**, no recorded MP3 path.

### 8.6 Buffer hygiene

Preallocate oscillator pools; never leak nodes. Stop and disconnect after each motif. CI heap snapshot after 30 min simulated session (see §10).

### 8.7 Caption generation

From the same motif sequence just scheduled: map intervals to phrases (“a soft three-note rise”). DOM label near bird, fade with call envelope. Naturalist voice, no “playing call_03.”

---

## 9. Accessibility surfaces

Ship on day one. Same product, different register — not a stripped mode.

### 9.1 Screen-reader narration

- One `aria-live="polite"` region off-canvas. Updates every **45s** idle (range 30–60).
- Immediate (still polite, not assertive) on greeting, offer reaction, settle.
- Prose from snapshot + names + lighting + weather. Example register from PRD. Never “mood: content”, never trait numbers, never ARIA on raw vectors.
- Focused bird: additional short observation, not “selected.”
- Cadence limiter: drop idle updates if live region still speaking.

Implementation: client-side templates + slot-fill; `narration_seed` in snapshot keeps visit/host wording aligned for the same tick.

### 9.2 Reduced-motion

If `prefers-reduced-motion: reduce` or account override:

- Replace looping micro-motion with **slow cross-fades between still poses** (1.5–3s).
- Perch travel = cross-fade, not flight path.
- No leaf/feather drift.
- Day/evening color shifts remain, duration ×2.
- Audio, drift, notebook, captions unchanged.

This is a second art pass, not `animation: none`.

### 9.3 Captions

Settings toggle; default on when audio unavailable. Generated at runtime from grammar (§8.7). Contrast AA against both day and night scenes (darken a pill behind text if needed).

### 9.4 Keyboard and focus

Tab: top-bar icons, then bird 1…n (DOM focus proxies positioned over birds). Arrows move between birds. Enter listen-in. Escape end listen-in. Offer and settle fully keyboardable.

Focus ring: soft high-contrast outline specified for light and night palettes (design system). Visible in reduced-motion too.

### 9.5 Contrast and AT chrome

All chrome, settings, errors, notebook text, captions: WCAG AA minimum. Notebook remains lowercase naturalist; that is content, not a contrast exemption.

### 9.6 Visit + AT

Visitor gets the same narration and captions; no listen-in control. Announce readonly once on load in matter-of-fact voice (“You’re visiting. The aviary is read-only.”), then switch to naturalist running narration.

---

## 10. Performance budgets and observability

### 10.1 Budgets

| Budget | Gate |
|---|---|
| Initial JS at first paint | < 2MB gzipped. Scene + audio grammar + chrome shell only. |
| Time to first bird | < 500ms mid-tier mobile / 4G. Cache snapshot + inline quiet field + no-block fonts. |
| Idle FPS | 60 on reference 5-year laptop, 30-minute soak. |
| Memory | Heap stable ± slack over 30 min; no per-call allocations retained. Notebook virtualize list. |
| Snapshot | < 16 KB typical. |
| Tick p99 | Alarm at 5s. Expected p99 << 500ms per aviary. |

### 10.2 How we hit TTFB-bird

- CDN HTML + critical CSS for quiet field.
- Sessioned edge worker injects `window.__SNAPSHOT__` when origin fetch < 80ms; else client uses IndexedDB cache.
- First bird draws from silhouette + solid plumage; feather detail progressive.
- Defer notebook, account, visit, settings chunks.
- No webfont blocking; system stack for chrome, one small display face optional after paint.

### 10.3 What we measure

Synthetic browsers on a schedule (common geos): load, first-bird mark, FPS, audio-context errors, tick lag probe.

RUM aggregate-only: navigation timing, `first_bird_ms`, long-task counts, audio errors, 30-min heap if available. **No account_id, no bird_id, no traits.** Session-duration histogram is anonymized and unlinked.

### 10.4 What we deliberately do not measure

Per-account presence totals in the warehouse; average boldness; offer conversion funnels; visit-to-signup; streak-like “D1/D7 return” **if** those dashboards are built from aviary interaction facts. Product analytics for growth, if any, stay at “signed in / snapshot 200 / error rate” — not relationship reconstruction.

Engagement features that would be “justified by the dashboard” are out of scope; do not build the dashboard that invites them.

### 10.5 CI performance tests

- Bundle size check on main.
- Headless first-bird timing against a mocked snapshot.
- 30-min simulated rAF + audio soak with heap diff.
- Tick microbench: 10k aviaries × 2–7 birds.

---

## 11. Rollout

### 11.1 Engineering phases (single team)

**P0 — spine (weeks 1–3).** Account + magic link + synthetic IDs. Aviary/bird schema. Snapshot + event APIs. Tick worker no-op then mood/time-of-day. Quiet-field client + two placeholder silhouettes mid-motion. No chrome except sign-in.

**P1 — aliveness (weeks 3–6).** Personality seed + drift + gain. Procedural calls + chorus. Listen-in ramps. Return-greeting planner. Day/night. Presence detector with 4-min window. Interpolation. Reduced-motion first implementation (cannot slip).

**P2 — gestures and notebook (weeks 6–8).** Offers + cooldowns + priority tick. Settle + undo. Notebook writer + UI. Captions. Keyboard path. AA pass.

**P3 — account completeness (weeks 8–10).** Multi-device soak. Export/delete. Visit invite/revoke/log. Age-gated third bird (can ship with flag; clock starts at account creation). Session revoke. Privacy policy link.

**P4 — harden (weeks 10–12).** Perf soak, tick alarm, dogfood drift calibration, a11y review with real AT, browser matrix, load quiet-field on throttled 4G.

### 11.2 Birds-per-aviary ramp

- Launch: **exactly two** birds, age offers **disabled** for the first 30 days of *calendar* public launch so support/calibration see a uniform population.
- Enable 45-day third-bird offer for accounts that have existed 45 days (includes dogfood accounts).
- Do not raise the cap above 7. Do not A/B more birds.
- Species pool ships all six at launch so later adoptees are not “content updates” that rewrite identity.

### 11.3 Instrumentation from day one

- Tick p99, queue depth, events lag (`now - oldest unconsumed`).
- Snapshot latency, 304 ratio.
- Auth consume success/expiry.
- Client: first-bird, FPS floor, audio fallback rate, JS errors.
- **Sim quality (in prod DB, not warehouse):** internal-only red-team queries on a **sampled anonymized scratch copy that strips account_id** used solely by the sim team during calibration, then destroyed. Prefer offline fixtures over live population stats. If live calibration is required, compute trait-delta histograms with no join back to email or events.

### 11.4 Launch posture

- No onboarding carousel. Magic link → quiet field → two birds already in motion → greeting.
- No “share with friends” prompt. Visit remains buried in settings.
- No emails except auth, export, invite, and optional visit notify.
- Feature flags: `visits_enabled`, `age_offers_enabled`, `weather_enabled` (weather can ship on). Flags are operational, not experiments on emotion.

---

## 12. Risks

### 12.1 Drift calibration

**Risk:** too fast → Tamagotchi; too slow → screensaver; leaky presence → population-wide over-drift.

**Mitigations:** three-signal presence; dual-device cap; `max_tick`; fixture tests with published weekly deltas; 4-minute activity window biased long; no “tab open” metric; priority ticks almost drift-free; 28-day gain separate from traits so quietness is reversible without punishing numbers.

**Watch:** if dogfood users describe session-to-session change, divide `k_*` by 2 immediately.

### 12.2 Sync correctness

**Risk:** a well-meaning client cache writes traits; LWW sneaks in; catch-up tick invents presence.

**Mitigations:** DB role / trigger rejecting personality updates outside worker role; API schema cannot accept trait fields; catch-up splits weather/mood time from drift; idempotent events; `tick_version` monotonic.

**Watch:** invariant test “sum of deltas == vector change” on every tick in staging.

### 12.3 Audio uncanniness

**Risk:** phasey chorus, repeating motif, modem-like oscillators, identical greetings.

**Mitigations:** seed-stable timbre + high timing jitter; no sample loops; greeting combinatorics logged in dogfood (hash of kind+motifs must not collide across a user’s first 20 sessions); listen-in ramps 1.8s; silence+captions rather than canned MP3.

**Watch:** “I keep hearing the same chirp” is a P0 affective bug.

### 12.4 Accessibility regressions

**Risk:** live region spam; reduced-motion as `display:none` animations; missing night contrast; personality leaked to AT.

**Mitigations:** cadence limiter; designed pose cross-fades in the same milestone as motion; contrast tokens for night; lint forbidding trait keys in ARIA; AT test in P4; captions forced on audio fail.

**Watch:** shipping a visual-only weather or greeting without narration is a launch blocker.

### 12.5 First-frame failure

**Risk:** spinner/fade “wakes up” the aviary and breaks the product thesis.

**Mitigations:** quiet-field CSS in HTML; cache paint; no ready-pop; budget 500ms/2MB as CI gates; forbid toast libraries in the scene bundle.

### 12.6 Offer/drift saturation

**Risk:** button-mashing curiosity.

**Mitigations:** 4-min per-type cooldown server-side; tiny offer deltas; diminishing `log1p`.

### 12.7 Social creep

**Risk:** visit log becomes a notification product; someone adds discovery.

**Mitigations:** notify off by default; no badge; no public routes; do not compute global visit ranks even internally.

### 12.8 Privacy leakage

**Risk:** email as ID in logs; warehouse join.

**Mitigations:** synthetic UUID rule in lint (deny email in log fields); separate mailer decrypt path; no sim replica to analytics; export is user-initiated only.

### 12.9 Identity continuity

**Risk:** species art “migration” replaces birds.

**Mitigations:** `birds.id` immutable; species_id additive; never DELETE+INSERT a bird for a content update; no “reset aviary” control.

---

## 13. Team workstreams and interfaces

Work in parallel after P0 contracts freeze:

1. **Sim** — tick, drift, mood, weather, greeting plan, notebook scorer, fixtures.
2. **API/auth** — magic link, snapshot, events, account lifecycle, visits.
3. **Render** — scene, lighting, idle, reduced-motion, first frame.
4. **Audio** — grammar, chorus, listen-in, captions, fallback.
5. **Chrome/a11y** — top bar, notebook UI, settings, keyboard, narration.
6. **Perf/privacy** — budgets, RUM allowlist, synthetic checks.

Frozen contracts: snapshot JSON, event types, trait ranges, mood enum, presence definition, voice split.

---

## 14. Definition of done (v1)

- Two birds mid-motion in <500ms on the reference mobile path; no spinner.
- One bird greets within 2s of owner session start; never identical twice in fixtures; no welcome toast.
- Presence requires all three signals; 24h background tab adds ~0 presence.
- Traits monotonic; 1-week fixture measurable; 3-week visible in perch/plumage/gain.
- Listen-in ramps; others never mute.
- Offers cooldown and mood-shaped reactions; settle undo 5s; tab close unpunished.
- Notebook sparse, naturalist, read-only, no user-behavior praise.
- Magic link 15m; sessions revocable; export and 30-day delete work.
- Phone and laptop show same moods/names/drift after isolated use.
- Visits read-only, off by default, revocable, no host ping by default.
- Narration, reduced-motion, captions, keyboard, AA chrome all on.
- Bundle, FPS, memory, tick p99 gates green.
- No achievements, streaks, native apps, recorded-call fallback, or personality numbers in any UI.

Stop here. Do not implement the product in this phase.
