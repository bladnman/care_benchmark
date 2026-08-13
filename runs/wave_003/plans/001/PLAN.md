# Pocket Aviary — v1 implementation plan

This plan turns the PRD into an executable build. It interprets, it does not restate. Where the spec is silent, a named decision is recorded under **Call**. Those calls are binding for v1 unless a later calibration note explicitly revises them.

The product is a relationship with a window. Architecture exists to keep that relationship honest: the aviary continues without the viewer, presence is measured rather than inferred, personality is server-owned, and nothing on the surface announces, scores, or punishes.

---

## 0. Binding decisions (read first)

These decisions are load-bearing. Teams should treat them as invariants, not preferences.

1. **Clients never tick and never write personality.** They append interaction events and render snapshots.
2. **Personality is additive server-authored deltas only.** No last-write-wins, no client-submitted absolute trait values, no rebuild-from-logs at read time.
3. **Drift is monotonic toward expressive.** Neglect does not lower traits. Ambient quietness is a separate, recency-weighted expression layer.
4. **Presence is the conjunction of visibility + window focus + recent pointer/key activity.** Any laxer definition is a product bug, not a telemetry shortcut.
5. **No announcement UI.** No welcome toast, streak, badge, visit ping (default), or “you’ve been gone” surface.
6. **Accessibility ships in v1 as designed surfaces**, not fallbacks. Reduced-motion, narration, and captions are first-class render/audio modes.
7. **No recorded call audio, ever.** WebAudio procedural synthesis or silence-plus-captions.
8. **Email is PII and lives in one encrypted column.** Synthetic account UUID everywhere else.
9. **Per-bird interaction history never enters aggregate telemetry, warehouses, or training sets.**
10. **Voice split is mechanical:** naturalist on product surfaces; matter-of-fact on auth, errors, account, sync, and accessibility settings.

---

## 1. Scope

### 1.1 In v1

- Browser-only SPA. Last two major versions of Chrome, Safari, Firefox, Edge.
- Single-user accounts. Magic-link auth. Per-device revocable sessions.
- One canonical aviary per account. Two starter birds. Cap of seven. New birds unlock by aviary age, not attention.
- Server-side simulation tick (~60s). Personality, mood, perch choice, weather, bird-to-bird prompting, sparse notebook authorship.
- Client render of a single non-scrolling horizontal scene with three perch zones, local-time day/night, rare weather, ambient ornaments.
- Interactions: return-greeting, idle presence, listen-in, offer (seed / song fragment / still pool), settle (with 5s undo), field notebook (read-only), rename.
- Multi-device sync as a property of server-canonical state.
- Optional visit invitations: email link, read-only ambient view, revocable, off by default.
- Account export (JSON emailed as download link), soft-delete 30 days then hard-delete.
- Accessibility: screen-reader narration, reduced-motion mode, call captions, WCAG AA chrome, full keyboard path.
- Performance: gzipped initial JS < 2MB; first bird visible < 500ms on mid-tier 4G; 60fps idle on a 5-year-old laptop; no client memory growth over 30 minutes.

### 1.2 Explicitly out of v1

Native apps. Payments. Shared / multi-profile / multi-aviary accounts. Customizable scenes. Catalog-picked starters. Public discovery, profiles, follows, comments, chat, avatars, leaderboards, show-off visitor rendering. Achievements, streaks, levels, scores, badges, visit calendars, XP. Tamagotchi hunger/death/distress/happiness meters. Push/email about the aviary (visit-notification email is the single opt-in exception, off by default). Recorded-audio fallback. Personality numbers anywhere in UI, including debug in production. SSO/passwords. Client-side simulation. Last-write-wins personality merge.

### 1.3 v1 quality bar that is in scope even if it feels like “polish”

First frame already in motion. Quiet-field loading (never a spinner). Procedurally unique greetings. Listen-in as a slow rebalance, never a mute. Notebook sparsity and naturalist prose. Identity continuity of birds across rename, sync, and species-catalog changes.

### 1.4 Call — product name and URL

Ship as **Pocket Aviary**. Public origin `https://aviary.example` is a placeholder; marketing domain is not a v1 blocker. Internal service names use `aviary-*`, never `pet-*` or `game-*`.

---

## 2. Architecture

### 2.1 Shape

Three deployable units, one database of record, one cache, one email provider.

```
┌─────────────────────────────────────────────────────────────┐
│ Browser                                                      │
│  chrome (Preact) │ scene renderer │ audio runtime │ a11y    │
│  presence probe  │ event outbox   │ snapshot interpolator   │
└─────────────┬───────────────────────────┬───────────────────┘
              │ HTTPS / JSON              │ (visitor token path)
┌─────────────▼───────────────────────────▼───────────────────┐
│ api-gateway  (TypeScript, Hono on Node 22 LTS)              │
│  auth, sessions, snapshots, event ingest, visits, account   │
└───────┬──────────────────┬───────────────────┬──────────────┘
        │                  │                   │
┌───────▼──────┐   ┌───────▼────────┐   ┌──────▼─────────────┐
│ Postgres 16  │   │ Redis 7        │   │ sim-worker         │
│ canonical    │   │ sessions,      │   │ Go 1.23            │
│ state +      │   │ rate limits,   │   │ per-account tick   │
│ event log    │   │ snapshot cache,│   │ every ~60s         │
│              │   │ tick lease     │   │                    │
└──────────────┘   └────────────────┘   └────────────────────┘
        │
        ▼
 email (SES or Postmark) — magic links, visit invites, export links
```

**Call — language split.** Shared domain types are specified in an OpenAPI + JSON Schema contract (`/contracts`). TypeScript owns the HTTP edge because the client already speaks it. Go owns the tick because tick latency, lease correctness, and “never skip an account silently” are easier to keep honest in a small dedicated worker than in request-scoped Node. The worker is the only process allowed to `UPDATE birds` personality/mood/perch columns.

**Call — no Kafka, no warehouse, no ML feature store in v1.** The event log is a Postgres table. Operational metrics go to the existing APM/metrics stack as aggregates. Simulation rows are not replicated into analytics.

### 2.2 Client / server split

| Concern | Owner | Why |
|---|---|---|
| Personality vector | sim-worker only | Continuity; no LWW |
| Mood, perch, weather, settled flag | sim-worker | Continues while clients are gone |
| Notebook prose | sim-worker | Sparse, server-authored, same voice as narration seed |
| Interaction events | client writes, server appends | Clients report what happened, not what it means |
| Presence qualification | client measures, server accepts pings only if well-formed | Three-signal conjunction cannot be reconstructed later |
| Greeting selection | sim-worker precomputes `next_greeting` into the snapshot | Avoids canned client-side “play arrival #2” |
| Scene ornaments (leaves, feathers) | client only | Not simulation state |
| Call synthesis | client WebAudio | Bundle + chorus |
| Narration prose | client composer from snapshot + event acknowledgements, using a shared phrase grammar | Must stay in lockstep with visuals; server may attach optional seed lines |
| Interpolation / idle micro-motion | client | Snapshots are sparse |
| Auth, export, deletion, visits | api-gateway | System surfaces |

### 2.3 Render pipeline boundary

The scene is **not** a React tree of birds. Chrome (top bar, settings, notebook sheet, auth, errors) is Preact. The aviary is a custom 2D scene graph drawn to a single full-bleed `<canvas>` (OffscreenCanvas + worker when available). Preact mounts the canvas and a parallel accessibility tree; it does not own per-frame bird state.

Snapshot → `WorldState` (immutable) → interpolator produces `FrameState` at display refresh → renderer draws → audio scheduler consumes the same `FrameState` for call timing. Reduced-motion is a renderer swap, not a “skip rAF” flag.

### 2.4 Why this split exists

If the laptop and the phone each simulated, return would require merging two climates. The PRD forbids that. If React owned bird poses, first-frame-mid-action and 60fps idle become accidental. If the client wrote traits, multi-device presence would silently delete drift. The split is the product.

### 2.5 Edge bootstrap for time-to-first-bird

`GET /` for an authenticated session is an HTML shell plus an inline `<script type="application/json" id="boot-snapshot">` containing the latest cached snapshot (Redis, 15s TTL, stamped by sim-worker). The renderer’s first paint uses that blob. A keepalive pull immediately refreshes it. Unauthenticated and cold-cache paths show the quiet field (soft sky, one faint leaf) — never a spinner — until the snapshot arrives, then the first bird frame is mid-pose with no fade-from-static.

**Call.** Boot snapshot is allowed to be up to 90s stale. Mood/perch will be close enough; the keepalive corrects within one RTT. Do not block first paint on a live tick.

---

## 3. Data model

Postgres is the source of truth. UUIDv7 for all public IDs (time-ordered, no email material). All timestamps `timestamptz`.

### 3.1 `accounts`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | Synthetic. The only identifier used in logs, keys, traces. |
| `email_ciphertext` | bytea | AES-GCM; key in KMS. The only email storage. |
| `email_lookup_hash` | bytea unique | HMAC-SHA256 of normalized email with a server pepper. Used only for login lookup. |
| `created_at` | timestamptz | Aviary age clock starts here (see 3.2). |
| `timezone` | text | IANA tz from client; updated on snapshot pull if changed. |
| `locale` | text | For future copy; v1 prose is English. |
| `captions_opt_in` | bool default false | |
| `reduced_motion_opt_in` | bool default false | ORs with `prefers-reduced-motion`. |
| `visit_notify_opt_in` | bool default false | The only notification toggle. |
| `deletion_started_at` | timestamptz null | Soft-delete marker. |
| `recovered_at` | timestamptz null | |
| `pending_email_ciphertext` | bytea null | Email-change flow. |
| `pending_email_hash` | bytea null | |
| `pending_email_token_hash` | bytea null | |
| `pending_email_expires_at` | timestamptz null | |

Never log `email_*` columns. Never use email as a partition key.

### 3.2 `aviaries`

One row per account.

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | |
| `account_id` | uuid unique fk | |
| `founded_at` | timestamptz | Age for new-bird offers. Frozen at creation; not reset on recovery. |
| `bird_slots_unlocked` | int | Starts at 2; sim-worker increments on age thresholds. |
| `weather` | enum | `clear`, `rain`, `wind` |
| `weather_until` | timestamptz | |
| `settled` | bool | True after settle until re-engage or tab-end processed. |
| `settled_at` | timestamptz null | |
| `last_presence_at` | timestamptz null | Last qualified presence ping. |
| `presence_minutes_7d` | numeric | Rolling window used by the expression layer, not by personality. |
| `notebook_cooldown_until` | timestamptz | Sparsity throttle. |
| `sim_version` | int | Tick schema / calibration epoch. |
| `tick_cursor` | bigint | Last consumed `events.id`. |
| `updated_at` | timestamptz | Snapshot etag source. |

### 3.3 `birds`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | Stable identity. Never recycled. |
| `aviary_id` | uuid fk | |
| `species_id` | text | From the v1 pool. |
| `name` | text | User-facing; renameable. |
| `adopted_at` | timestamptz | |
| `sort_index` | int | Adoption order for focus cycling. |
| `boldness` | numeric(6,5) | 0.00000–1.00000 |
| `social_warmth` | numeric(6,5) | |
| `vocal_frequency` | numeric(6,5) | |
| `plumage_saturation` | numeric(6,5) | |
| `curiosity` | numeric(6,5) | |
| `mood` | enum | `wary`, `content`, `curious`, `drowsy`, `alert` |
| `mood_since` | timestamptz | |
| `perch_zone` | enum | `front`, `middle`, `back` |
| `perch_slot` | int | 0–2 within zone; collision-avoided by tick. |
| `pose` | text | Current idle cycle id, e.g. `preen`, `scan`, `fluff`, `tilt`, `sleep`. |
| `pose_phase` | numeric | 0–1, so clients resume mid-action. |
| `heading` | numeric | Facing; small variance. |
| `offer_cooldown_until` | timestamptz | Per-bird. |
| `call_seed` | int | Stable PRNG seed for this bird’s grammar. Never changes. |
| `retired_at` | timestamptz null | Unused in v1; reserved so identity is never deleted except on hard account delete. |

**Hard rule:** no API response includes the five trait numbers except the account-export JSON, which is a user-owned copy, not a UI surface.

### 3.4 Expression layer (not personality)

Store on `aviaries` / derive at tick:

- `greeting_readiness` per bird: recency-weighted presence, range 0–1, **may fall** when the user is away. This is how neglect becomes ambient quietness without lowering traits.
- `chorus_heat`: short-lived, minutes, from recent calls.

These are allowed to decay. Personality is not.

### 3.5 `events` (append-only)

| Column | Type | Notes |
|---|---|---|
| `id` | bigserial | Tick cursor. |
| `account_id` | uuid | |
| `bird_id` | uuid null | |
| `type` | text | See §5.3. |
| `payload` | jsonb | Strict schema per type. |
| `client_event_id` | uuid | Idempotency. Unique per account. |
| `client_occurred_at` | timestamptz | |
| `received_at` | timestamptz | |
| `processed_at` | timestamptz null | |

No updates except `processed_at`. No deletes except hard account deletion.

### 3.6 `notebook_entries`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `aviary_id` | uuid | |
| `prose` | text | Naturalist, lowercase, present-tense. |
| `observed_on` | date | User-local date, used for “tuesday —” leads. |
| `created_at` | timestamptz | |
| `trigger` | text | Internal only (`greet_order`, `long_quiet`, `weather`, `new_bird`, `first_listen_in_week`, …). Never exposed. |
| `subjects` | uuid[] | Bird ids mentioned; for future, not UI. |

Read-only to clients. No edit/delete/annotate. No archival.

### 3.7 Auth and devices

- `magic_links`: `token_hash`, `email_lookup_hash`, `expires_at` (created_at + 15m), `consumed_at`, `account_id` null until consume-or-create.
- `sessions`: `id`, `account_id`, `token_hash`, `device_label` (UA-derived, user-renameable later if cheap), `created_at`, `last_seen_at`, `revoked_at`.
- Session cookie: `HttpOnly; Secure; SameSite=Lax; Path=/`. Token is 256-bit random. Store only hash.

### 3.8 Visits

- `visit_invites`: `id`, `host_account_id`, `visitor_email_ciphertext`, `visitor_email_hash`, `token_hash`, `created_at`, `expires_at` (30 days), `revoked_at`, `first_used_at`.
- `visit_sessions`: `id`, `invite_id`, `started_at`, `ended_at`, `approx_duration_s`. No presence events. No bird events.

Visitor auth is the invite token only. Visitors are not accounts unless they already have one; having an account does not grant extra powers on a visit.

### 3.9 Species catalog (code, not DB)

Six species, one coherent temperate-woodland set. Rarity is not a feature.

| `species_id` | Silhouette | Default palette | Call family | Night behavior |
|---|---|---|---|---|
| `warbler` | slim, tail-flick | olive / grey | three-note rise | sleeps |
| `wren` | small, cocked tail | warm brown | hurried trill | sleeps |
| `sparrow` | compact, round head | muted ochre | two-note chip | sleeps |
| `finch` | stout beak | dusty rose-brown | rolling phrase | sleeps |
| `dove` | fuller body | soft grey | low coo pair | sleeps |
| `nightjar` | longer wing, crouch | bark-brown | hollow churr | **active at night** |

Starter pair: sim-worker draws two **different** species with contrasting call families (never two trillers). Seed traits: each trait ~ `clip(N(μ_species, 0.08), 0.18, 0.62)` so starters are distinct but not extreme.

### 3.10 Personality ranges and seeds

All traits ∈ [0, 1]. Soft ceiling 0.97 so drift never “completes.” New birds after starters use the same draw. `call_seed` is independent of traits so a rename or saturation drift never changes identity of the voice.

### 3.11 Snapshot document (what clients actually consume)

A snapshot is a versioned JSON blob, typically 2–8 KB:

```
{
  schema: 1,
  etag: "...",
  server_time: iso,
  aviary: { founded_at, weather, weather_until, settled, lighting_phase, local_tod },
  birds: [{
    id, species_id, name, mood, perch_zone, perch_slot,
    pose, pose_phase, heading, plumage_saturation,
    greeting: null | { kind, delay_ms, motif_hint },
    call: { next_window_s, grammar_params },
    offer_cooldown_until,
    expression: { greeting_readiness, alertness }  // qualitative bins only if needed; prefer raw 0–1 for client motion, never labeled in UI
  }],
  ambient: { leaf_intensity, wind, nightjar_active },
  notebook_latest_id,
  a11y_seed: { idle_paragraph }  // optional server-authored idle line
}
```

**Call.** `plumage_saturation` is the one numeric trait that must reach the renderer (color). It is never labeled. Other traits influence snapshot fields only through derived pose/perch/call params computed by the tick.

### 3.12 Export document

The account-export JSON includes birds (with vectors), moods, names, notebook entries, settings, aviary founded_at, and invite metadata (emails of invitees the user already knows). It excludes session tokens, magic-link hashes, and raw event logs. Generated on demand, stored 24h in object storage, emailed as a signed download link.

---

## 4. API surface

All authenticated routes take the session cookie. JSON in/out. Matter-of-fact error bodies. No HATEOAS. Idempotency on writes via `Idempotency-Key` or `client_event_id`.

Rate limits (Redis): magic-link 5 / email / 15 min; event ingest 120 / min / session; snapshot 30 / min / session; visit-invite 10 / day / account.

### 4.1 Auth and account

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/auth/magic-link` | Body `{ email }`. Always 202 with the same body copy whether or not the email exists. Sends link. |
| `GET` | `/v1/auth/consume` | Query `token`. One-time. Issues session, 302 to `/`. Expired/used → matter-of-fact page. |
| `POST` | `/v1/auth/logout` | Revoke this session. |
| `GET` | `/v1/me` | Account settings DTO. No traits. |
| `PATCH` | `/v1/me` | Timezone (usually automatic), a11y flags, visit_notify, device label. |
| `POST` | `/v1/me/email` | Start email change; verify new address. Old email works until verify. |
| `GET` | `/v1/sessions` | Device list. |
| `DELETE` | `/v1/sessions/:id` | Revoke. |
| `POST` | `/v1/me/export` | Enqueue export email. |
| `POST` | `/v1/me/delete` | Soft-delete. |
| `POST` | `/v1/me/recover` | Clear `deletion_started_at` if within 30 days. Any signed-in page also offers this. |

Magic-link email copy is matter-of-fact. No “your birds miss you.”

### 4.2 Aviary state and events

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/aviary/snapshot` | Canonical snapshot. `If-None-Match` supported. On visibility return and keepalive. |
| `POST` | `/v1/events` | Batch append. Body `{ events: [{ client_event_id, type, bird_id?, occurred_at, payload }] }`. Max 50. |
| `GET` | `/v1/notebook` | Cursor pagination, oldest-allowed, never hidden. |
| `PATCH` | `/v1/birds/:id` | `{ name }` only. |
| `POST` | `/v1/aviary/adopt` | Adopt next unlocked bird if `bird_slots_unlocked > count`. Names optional; suggestions provided. No catalog. |

Clients do **not** `POST /personality`. Offers, listen-in, settle, presence are events.

### 4.3 Event types (client → log)

| `type` | Payload | Tick effect |
|---|---|---|
| `presence_ping` | `{ dt_s, visibility, focused, last_input_age_s }` | Accrue presence-time **only if** all three signals qualify. Reject/ignore otherwise (still 204). |
| `listen_in_start` | `{ bird_id }` | Open attention window on that bird. |
| `listen_in_end` | `{ bird_id, duration_s }` | Close window; drift social_warmth + vocal_frequency. |
| `offer` | `{ kind: seed\|song\|pool, near_bird_id? }` | Reaction + curiosity/boldness deltas if accepted / near. |
| `settle` | `{}` | End presence; set `settled`; mood quieting; no trait direction. |
| `settle_undo` | `{}` | If within 5s of settle, clear settled; do not invent presence. |
| `reengage` | `{}` | Click after settle (post-undo window) or any intentional return-to-day. Clears settled. |
| `focus_visible` | `{ bird_id }` | Keyboard focus; does not by itself listen-in until Enter. |
| `session_hello` | `{ last_hidden_at?, viewport }` | Used to compute absence length for greeting. |
| `name_change` | `{ bird_id }` | Audit only; name write is the PATCH. |

**Call — activity window.** Start at **240 seconds** without pointermove/keypress before presence fails. Lean long: watching without moving is the product. Calibrate in §12; do not ship below 120s.

**Call — presence ping cadence.** Every 30s while qualified; one immediate ping on becoming qualified; one terminal ping on disqualify/settle/pagehide with `dt_s` since last.

### 4.4 Visit flow

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/visits` | Host body `{ email }`. Creates invite, emails one-time link. |
| `GET` | `/v1/visits` | Outstanding invites + visit log (email, date, approx duration). No badge counts. |
| `DELETE` | `/v1/visits/:id` | Revoke immediately. |
| `GET` | `/v1/visit/:token` | Visitor HTML/app shell. |
| `GET` | `/v1/visit/:token/snapshot` | Same visual snapshot as host, **stripped** of greeting directives, offer cooldowns-as-actions, notebook, and any write cursors. If revoked/expired: 410 with matter-of-fact body. |

Visitor client is a render-only build: no event POST, no listen-in, no offer, no settle, no notebook, no presence probe. Audio plays ambient mix only. If WebAudio blocked, captions on.

**Call.** Visit-notification email (only if `visit_notify_opt_in`) fires at most once per visitor per 24h, matter-of-fact: “Someone visited your aviary.” No bird names, no duration in the email. The log is the detail surface.

### 4.5 Error voice

```
{ "error": "magic_link_expired", "message": "We couldn't sign you in. The link may have expired. Try requesting a new link." }
{ "error": "session_timeout", "message": "Your session timed out. Sign in again to keep watching." }
{ "error": "snapshot_failed", "message": "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." }
{ "error": "visit_unavailable", "message": "This visit is no longer available." }
{ "error": "unsupported_browser", "message": "This browser isn't supported. Use a recent Chrome, Safari, Firefox, or Edge." }
```

No naturalist phrasing. No “the flock could not find you.”

---

## 5. Simulation engine

### 5.1 Tick loop

`sim-worker` every 5s looks for aviaries due for a tick (`updated_at + interval` or a due-queue in Redis). Target cadence **60s ± jitter 0–10s** so a fleet does not align. Lease via Redis `SET nx ex 45` per `aviary_id`.

Per tick, in one DB transaction:

1. Lock aviary row.
2. Read birds + unprocessed events with `id > tick_cursor` ordered by `id`.
3. Qualify presence events (server re-checks payload flags; does not trust `dt_s` if flags fail).
4. Apply interaction deltas to a scratch vector (never below current trait).
5. Apply presence-weighted drift.
6. Decay expression layer (`greeting_readiness`, `chorus_heat`).
7. Advance weather (rare).
8. Transition moods (time of day, weather, interactions, personality as bias).
9. Choose perches and poses; advance `pose_phase` as if 60s of motion occurred.
10. Schedule call windows and optional greeting directive if a `session_hello` arrived since last tick **or** if a snapshot pull with `visible=1` is happening — see §5.6.
11. Maybe write one notebook entry.
12. Maybe unlock a bird slot by age.
13. Write birds + aviary; set `tick_cursor`; mark events processed; bump `etag`.
14. Refresh Redis snapshot cache.

If no events and no weather/mood timer due, still advance pose_phase, lighting, and expression decay. The aviary is not frozen between visits.

**Call — catch-up.** If an aviary was not ticked for N minutes (worker outage), run `min(N, 180)` virtual minutes at the same per-minute rates, chunked in 15-minute steps, rather than one giant step. Cap 3 hours of catch-up per wake to bound tx time; leftover remains due. Mood follows local sun even during catch-up.

### 5.2 Drift function

Let each trait `x ∈ [0, 0.97]`.

```
Δx = (1 - x) * ( w_p * P + w_i * I_x ) * k
x  ← min(0.97, x + Δx)
```

`(1 - x)` is the low-pass: early movement is easier, late movement is slow. `k` is the global calibration constant.

**Inputs (per bird, per tick, then scaled to actual minutes represented):**

- `P` — qualified presence-minutes on this tick / 60. Dominant.
- `I_social` — listen-in minutes on this bird.
- `I_vocal` — same listen-in minutes.
- `I_curiosity` — 1 if this bird accepted an offer this tick else 0.
- `I_boldness` — 1 if an offer landed near this bird (same or adjacent perch zone) else 0.
- `I_plumage` — `P` only (attention saturates color). Settle contributes 0 to all `I_*` and 0 to `P` after the settle event.

**Call — initial weights** (to be locked by the harness in §12):

| Source | Weight |
|---|---|
| Presence → all traits, extra on plumage | `w_p = 1.0` for plumage, `0.55` others |
| Listen-in → warmth, vocal | `w_i = 2.2` |
| Offer accepted → curiosity | `w_i = 0.8` per accept |
| Offer near → boldness | `w_i = 0.45` per near-offer |
| `k` | `0.0040` per presence-hour equivalent |

**Calibration target, made numeric.**

Assume a “regular visitor”: 5 days/week, 18 qualified minutes/day.

- Week 1: ~1.5 presence-hours → typical trait Δ ≈ 0.006–0.012. **Measurable in fixtures.** Not user-visible.
- Week 3: ~4.5 hours → Δ ≈ 0.02–0.04, plus listen-in on a favorite. **User-visible** as “Pip comes forward more” / richer color / greets first more often.
- A single 20-minute session: Δ < 0.002. Must fail a “visible step” screenshot test.

Neglect: `Δx = 0`. Traits hold. `greeting_readiness *= exp(-λ * absent_hours)` with `λ ≈ 0.04` (half-life ~17h of absence). After two weeks away, greetings are rare glances; color and identity are intact.

### 5.3 Mood

Enum: `wary | content | curious | drowsy | alert`.

Daily-ish reset is not a midnight snap to `content`. It is a soft attractor toward a time-of-day prior, mixed with recent events:

| Local time | Prior |
|---|---|
| 05:00–09:00 | `alert` (nightjar: `content` if night-active window) |
| 09:00–16:00 | `content` / `curious` |
| 16:00–20:00 | `drowsy` rising |
| 20:00–05:00 | `drowsy` or sleep-pose; nightjar `alert`/`content` |

Transitions (priority order):

1. Another bird’s alarm (rare; weather-start or sudden offer to a wary bird) → nearby birds `wary` for 2–8 minutes.
2. Rain → vocal dampen; mood bias `wary` or `content` (not `alert`).
3. Wind → `alert` or `wary` by boldness.
4. Offer accepted → `content` (curious birds may stay `curious`).
5. Listen-in on a bird → slight `curious` if not drowsy.
6. Settle → all birds toward `drowsy`.
7. Else drift toward time-of-day prior, speed inverse to trait stability (high boldness resists `wary`).

Mood persists across sessions. Tick writes it; clients do not reset on load.

### 5.4 Perch and idle pose

Perch choice is a softmax over zones:

```
score(front)  = boldness + 0.25*(mood==alert|curious) - 0.45*(mood==wary) + greeting_boost
score(back)   = (1-boldness) + 0.35*(mood==wary|drowsy)
score(middle) = 0.4 + 0.2*social_warmth
```

High social_warmth birds prefer a slot adjacent to another bird. Tick resolves collisions. Users never place birds.

Pose from mood: wary→scan, content→preen, curious→tilt, drowsy→fluff/sleep, alert→scan. Pose_phase increments by a species-specific rate so the client can start mid-preen.

### 5.5 Bird-to-bird

On each tick, if bird A scheduled a call in the last window and bird B has high vocal_frequency and is not drowsy, B gets a response-call window offset 0.4–1.8s. Wary spreads: if ≥1 bird is wary and weather is wind/rain, each neighbor rolls `p = 0.25 * (1 - boldness)` to become wary. Chorus: if ≥2 birds have overlapping call windows and vocal_frequency > 0.5, snapshot marks `chorus: true` so the client mix breathes rather than stacking peaks.

### 5.6 Return-greeting

Computed when a `session_hello` arrives (or the first snapshot after a visibility gap ≥ 20s). **One** primary greeter.

Selection: highest `boldness * greeting_readiness * mood_factor`, with `mood_factor` low for drowsy, high for alert/curious. If the winner’s score is below a floor, **no full greeting** — only a glance pose. That is valid.

Absence length:

| Hidden / away | Greeting kind |
|---|---|
| < 3 min | glance, no call |
| 3–90 min | head-tilt or one two-note call |
| 90 min–36 h | step toward front + short call |
| > 36 h | longer call; possible staggered second bird at +400–1200ms |

Stagger is mandatory if a second bird also clears a lower threshold. Never unison.

`greeting` in the snapshot is a directive with `kind`, `delay_ms`, and `motif_hint`. Client realizes it procedurally (timing, pitch, which notes) so two sessions never match. After realization, client does not need to ACK; the next tick clears the directive.

### 5.7 Offers

Kinds: `seed`, `song`, `pool`.

Cooldown: **4 minutes per bird** after that bird is the receiver. Offering “to the aviary” still picks a receiver: nearest bird to the front-center that is not on cooldown; if all on cooldown, the offer still appears visually but produces no drift and a muted reaction (watch, not approach).

Reactions:

- Seed: approach if `curiosity + mood_bonus > 0.45`; wary waits 8–20s then maybe approaches; drowsy often ignores.
- Song: play a library motif (6 fragments, not bird calls — human/whistle-like, quiet). Response: join / quiet / call-against from vocal_frequency × mood.
- Pool: a soft reflective ellipse in the front plane for ~90s. Drink / bathe / watch from curiosity and species (dove bathes more; nightjar watches).

Accepted offer: `curiosity` delta. Near offer: `boldness` delta. Neither is visible in-session as a number; the reaction *is* the feedback.

### 5.8 New birds by age

| Aviary age | Unlocked slots |
|---|---|
| 0 | 2 (starters; system-chosen; named in a short adopt overlay, not a catalog) |
| 42 days | 3 |
| 120 days | 4 |
| 240 days | 5 |
| 365 days | 6 |
| 540 days | 7 |

Unlock writes `bird_slots_unlocked` and a notebook line. The adopt affordance appears in the top bar as a small extra mark on the offer cluster — **not** a modal on login, **not** “you earned a bird.” User opts in, names the arrived bird, soft fly-in once. Empty aviary only happens on first-ever adopt.

### 5.9 Call-grammar runtime (server half)

Server does not render audio. It emits `grammar_params`: species motif ids, tempo, pitch center, density (from vocal_frequency), mood filter (drowsy = longer gaps, lower amp), and the next 60s of call-window suggestions (Poisson process). Client is free to micro-vary within windows so two devices are not phase-locked in a disturbing way, but identity (`call_seed`, motif family) is identical.

### 5.10 Notebook author

A small deterministic composer in the worker. Inputs: greet order, long quiet (>20 min presence with no calls — rare), weather begin, new bird, first time this week bird A greeted before bird B, a drowsy fluff lasting many minutes.

Sparsity: if `now < notebook_cooldown_until`, skip. After a write, cooldown = 36–72h plus noise, shortened to 12h only for `new_bird` or first rain of the month. Active users do not get a feed.

Prose templates are slot-filled with bird names, local weekday, perch, weather. Lowercase. Present tense. No “you.” No visit counts. No trait numbers. Example generator tests must reject strings matching `achievement|streak|visited every|vocal frequency|boldness`.

---

## 6. Sync model

### 6.1 Single writer

```
device A ─┐
          ├─► POST /events ─► events table ─► sim-worker ─► birds/aviaries
device B ─┘                                              │
                                                         ▼
                                              GET /snapshot (both devices)
```

There is no client-to-client channel and no CRDT. Devices are projectors.

### 6.2 Conflict prevention (not resolution)

Personality cannot conflict because clients cannot write it. Concurrent offers from two devices: both events land, tick applies both with cooldowns (second accept within cooldown is visual-only). Concurrent settles: settled is boolean; two settles are one. Concurrent renames: last `PATCH` wins on `name` only — names are not drift. That is the only LWW field, and it is labeled as such.

Presence from two devices at once: **Call — do not double-count.** Presence pings carry `session_id`. Tick unions overlapping qualified intervals per account rather than summing. Watching on phone and laptop simultaneously is one presence, not two. This protects calibration.

### 6.3 Client pull triggers

1. Boot (inline + GET).
2. `visibilitychange` → `visible`.
3. `pageshow` after `persisted` (bfcache) or long hidden.
4. `rAF` gap > 5s (laptop sleep).
5. Keepalive every 45s while visible.
6. After posting events that should change the scene quickly (offer, settle): optimistic local visual, then snapshot within 1s. Tick may not have run; optimistic layer handles offer props and settle lighting immediately. Personality still waits for the tick (and must not appear to jump).

### 6.4 Interpolation

Snapshot N says Pip at back/slot1; N+1 says front/slot0. Client eases over 8–14s (reduced-motion: cross-fade poses at the two perches, no path). Never teleport. If a snapshot is older than 3 minutes (tab backgrounded, render stopped), skip interpolation and place mid-pose at the new perch — still no fade-from-black.

### 6.5 Offline / flaky network

Event outbox in `IndexedDB`: persist unsent events, replay with same `client_event_id`. Snapshot failure: keep last world, quiet field only if we have never had a snapshot. Do not invent ticks offline. If offline > 10 minutes, freeze ornaments at low energy; on reconnect, pull snapshot (server has been ticking).

### 6.6 Session and magic-link races

Used magic link invalidates immediately. Replay shows expired copy. Mid-write timeout: events are idempotent; user may see the generic snapshot error, not a merge UI. There is **no** “pick a version of your birds” dialog. If we cannot load, we say so matter-of-factly.

### 6.7 Deletion vs sync

Soft-deleted accounts: sessions still authenticate but every page shows recover. Tick **pauses** (no drift during deletion limbo — absence should not be “time served”). Hard delete: cascade birds, events, notebook, invites, sessions, encrypted email; object-storage exports expired. Telemetry has only `account_id` which now points at nothing; do not retain a mapping table.

---

## 7. Frontend rendering pipeline

### 7.1 Scene graph

Layers back to front, all in one canvas:

1. Sky gradient (local time + settle).
2. Far foliage (slow tint, almost no parallax).
3. Back perch zone + birds in that zone.
4. Mid foliage / branches.
5. Middle perch + birds.
6. Front perch + birds + still pool when active.
7. Foreground leaf/feather ornaments.
8. Caption layer (DOM overlay, not canvas, for a11y and contrast control).

Parallax: max 4px shift over full viewport — present, not showy. No zoom, pan, or scroll. Viewport resize recomputes a letterbox-free layout that **never crops a bird**. Narrow phones compress inter-perch spacing; wide desktops add air. Min supported CSS width 320px.

### 7.2 Bird drawing

Each species is a small rig: body, tail, head, beak, far/near wing, eye. Drawn from parameterized paths (not large bitmaps). Plumage saturation eases fill chroma. Eye open/closed for sleep. No sprite sheet of recorded frames for idle — poses are procedural blends of rig parameters. Bundle stays small.

### 7.3 Idle micro-motion

`rAF` loop, 60fps target. Each bird has a phase clock started from snapshot `pose_phase` + `performance.now()`. Motions: weight-shift, preen arc, scan saccades, breath scale (~0.7% body, ~3s period), head tilt toward the last call location. **Never zero velocity** unless reduced-motion (then still poses with slow cross-fades). Hidden tab: cancel rAF, suspend audio, keep outbox. Simulation continues on server.

### 7.4 First frame

On boot snapshot: instantiate birds at `pose_phase`, start clocks as if they had been running (`phase0 = pose_phase - ε` so the first frame is mid-gesture). Play ambient mix immediately if audio unlocked; otherwise wait for first pointer/key (browser policy) without a big “click to enter” splash — a first pointer anywhere unmutes. Quiet field is only for no-snapshot-yet.

Empty aviary (first session only): quiet field, then each starter fly-in once after naming. Never again.

### 7.5 Lighting and weather

Local time from `Intl` + account timezone (server lighting_phase is authoritative to keep devices aligned). Sunrise/sunset via a compact sun-times function. Evening warm shift; night dim; settle forces evening palette over **3.5s** regardless of clock, undone in 3.5s on undo.

Rain: sparse streaks, 4–12 minutes, a few times a week (tick-scheduled). Wind: leaf rate up, birds alert/wary. No thunder, no snow, no lightning.

Ornaments: client Poisson process, no per-leaf network state.

### 7.6 Top bar

Icons only: settings, accessibility, notebook, offer (and adopt when a slot is open). After **4s** cursor/keyboard stillness, opacity → 0.12. Any pointermove/key → 1.0. No labels inside the scene. No badges. No unread-notebook dot that functions as a notification (if we need a hint that a rare entry exists, it is a one-pixel notebook-icon material change, not a count).

Offer flow: top-bar button opens a three-choice popover (seed, song, pool) in naturalist microcopy, then the item appears in-scene. Not a click-on-bird.

Settle: top-bar. 5s undo: any click in the scene. After 5s, clicks are reengage.

### 7.7 Reduced-motion renderer

Swap the motion system, not the world:

- Idle: 2–3 still poses per mood, cross-fade 1.2–2.0s.
- Perch change: dissolve A→B, no flight arc.
- No leaf/feather drift.
- Day/evening color still shifts, slower (8s).
- Greetings: pose cross-fade + call (or caption), no hop.
- First frame: still a mid-pose, not a blank fade-in.

Honor `prefers-reduced-motion` immediately, even before settings fetch; then OR with account flag.

### 7.8 Color

Calm naturalist palette. No electric accents. Focus ring is a soft high-contrast outline specified to pass against noon and night skies (design system owns hex; engineering owns the ring not being clipped by canvas). User-copy in chrome/captions/settings: WCAG AA floor.

### 7.9 Chrome framework

Preact + signals for settings/notebook/auth. Route table: `/` aviary, `/enter` magic-link request, `/visit/:token`, `/unsupported`. No marketing site in this repo.

---

## 8. Audio pipeline

### 8.1 Graph

```
per-bird Voice  → gain_bird  ┐
                             ├→ chorusBus (mild saturation, never limiter-pump)
ambient air    → gain_air    ┤
song-offer     → gain_offer  ┤
                             └→ masterGain → destination
```

Each `Voice` is an `AudioWorklet` (fallback: main-thread `Oscillator` + `Biquad` + noise buffer if worklet fails but WebAudio exists).

### 8.2 Procedural voice

Motif = sequence of grains. A grain: damped sine or FM pair + very short noise burst, pitched from `call_seed` and species template. Timing jitter ±8–15%. Amplitude envelope personality- and mood-shaped. **No sample files for calls.**

Recognizability: species template defines interval set and rhythm skeleton; `call_seed` picks a stable tonic offset and timbre bias. Mood changes tempo/space, not the skeleton. A listener two weeks in should name Pip without a label.

Vocal frequency maps to Poisson rate of windows, and to p(join chorus).

### 8.3 Chorus

When two voices overlap, they are separate sources with independent jitter. Do not start the same motif on a grid. A tiny allpass difference per bird avoids phase-cancel combing. Cap simultaneous voices at 7 by product rule; mixer is built for 7.

### 8.4 Listen-in mix

Engage/disengage: **2.4s** equal-power ramp.

- Focused bird target gain: 1.0
- Others: 0.22 (never 0)
- Ambient air: 0.7

This is a rebalance. Disengage by second click/tap on the bird, click empty scene, focus another bird, or Escape / focus leaving the scene.

### 8.5 Unlock and fallback

Autoplay policy: construct `AudioContext` on load, resume on first user gesture. Until then, if captions are off, still render visually; do not throw a modal. If WebAudio is missing or `resume` permanently fails: **silence + captions forced on** for the session. No MP3 pack.

### 8.6 Caption generation

The same motif expander that schedules grains emits a caption token stream: e.g. `soft three-note rise`, `low trill, paused, low trill again`. DOM text near the bird, fade with the call (~call duration + 400ms). Naturalist voice. Generated, not a fixed map of three strings.

### 8.7 Memory

Worklet uses a preallocated grain pool. No `new AudioBuffer` per call. Disconnect/reconnect is forbidden in the hot path; gains go to 0. CI heap snapshot after 30 min simulated calling must be flat within noise.

### 8.8 Settle and night

Settle: master call density ×0.25 over 3.5s. Night: most species rate ×0.1; nightjar unconstrained (still personality-shaped). Rain: global rate ×0.6.

---

## 9. Accessibility surfaces

### 9.1 Accessibility tree

A visually-hidden (not `display:none`) live region and a roving-tabindex list of birds sit beside the canvas.

- Tab: top bar icons, then first bird.
- Arrows: birds by `sort_index` / spatial left-to-right.
- Enter: listen-in toggle.
- Escape: exit listen-in; if a popover is open, close it first.
- Offer and settle: top-bar buttons, fully keyboardable.

Focus ring visible on canvas via a synced overlay element.

### 9.2 Narration

`aria-live="polite"` region. Cadence **45s** idle (jitter 30–60). Immediate (but still observational) lines for: return-greeting, offer reaction, settle, new bird fly-in.

Wrong: `Pip is at perch 2. Mood content.`
Right: `a small grey bird is perched on the front rail, calling softly.`

Composer lives in the client, shared phrase tables with the notebook (same repo package `naturalist-copy`). Uses names once they are known; species vernacular otherwise. No trait numbers. No “welcome back.” No visit-frequency.

Priority: user-initiated observation > greeting > idle. Queue max 2; drop idle if a user event is pending. Never spam.

### 9.3 Captions and reduced-motion

See §8.6 and §7.7. Settings page is matter-of-fact: “Call captions”, “Reduce motion”, “Visit emails”. System `prefers-reduced-motion` is explained as already honored.

### 9.4 Contrast and names

Chrome, settings, errors, captions, notebook text: AA minimum. Notebook sheet is a real document, scrollable, not infinite-virtualized in a way that drops text from the a11y tree — virtualize only beyond 200 entries, keeping focus stable.

### 9.5 Visit a11y

Visitor view: same narration and reduced-motion; no interactive birds (tab skips the bird list or presents it as non-interactive text). 410 page is matter-of-fact and focusable.

---

## 10. Performance budgets and observability

### 10.1 Budgets (CI-enforced)

| Budget | Gate |
|---|---|
| Initial JS gzipped (critical path) | < 2.0 MB; alarm at 1.6 MB |
| Time to first bird pixels | < 500 ms p75 synthetic mid-tier 4G |
| Idle FPS | 60 on reference 5-year laptop profile, 30-min run |
| Heap | no monotonic growth over 30 min (regression test) |
| Snapshot size | < 16 KB gzipped |
| Tick p99 | < 5 s (alarm); expected p50 < 40 ms / aviary |
| Magic-link email | p95 < 10 s enqueue-to-accept |

Code-split: settings, visit-host UI, notebook sheet, adopt overlay. Critical path: renderer + audio worklet + snapshot hydrate.

Assets: species rigs as path data in TS modules, not PNG atlases. No call samples.

### 10.2 What we measure

Synthetic browsers in a few geos: boot, first-bird mark (`performance.mark('first-bird')`), rAF long-task rate, audio-context errors, snapshot RTT.

RUM (aggregate only): TTFB, first-bird, JS errors, audio unlock failure rate, tick latency histograms. Dimensions: browser family, country, connection bucket. **Not** account_id, bird_id, species, mood, presence minutes, offer kinds.

Server: request rates, 4xx/5xx, tick lease failures, queue lag, email bounces. Logs keyed by `account_id` UUID only.

### 10.3 What we deliberately do not measure

Per-account session duration as an exported product metric. Funnel “engagement.” Average boldness. Offer CTR. Visit conversion. Anything that would make a weekly meeting about making birds needier. Internal debug can inspect a **single** account in an admin break-glass tool with audit log; that tool is not a dashboard of the flock.

### 10.4 Privacy = pipeline shape

- Simulation DB security group is not peered to the warehouse.
- ETL jobs listed in repo; none select from `events`, `birds` traits, or notebook prose.
- APM scrubbers drop query strings with tokens.
- Feature flags are account-UUID targeted, never email.

---

## 11. Rollout

### 11.1 Build sequence (one team, sequential where noted)

**M0 — contract and skeleton (week 1–2)**  
OpenAPI, event schemas, Postgres migrations, magic-link loop, empty quiet-field page, matter-of-fact errors.

**M1 — snapshot + mid-action render (week 2–5)**  
Two hardcoded birds, three perches, day/night, top-bar fade, first-frame-mid-pose, reduced-motion cross-fades, keyboard focus. No drift yet. This is the aliveness spike; if this fails the product, stop.

**M2 — audio (week 4–7, overlaps M1)**  
Worklet voices, two species grammars, chorus, listen-in ramps, caption generator, silence fallback. Recognizability listening tests.

**M3 — sim-worker (week 5–8)**  
Tick, drift, mood, weather, greeting directives, presence qualification, expression decay. Fixture harness for week-1 / week-3 targets.

**M4 — interactions + notebook (week 7–10)**  
Offers, settle+undo, notebook composer + sheet, adopt-by-age (time-accelerated in staging), rename.

**M5 — sync and account (week 8–11)**  
Multi-device presence union, export, soft/hard delete, email change, session revoke.

**M6 — visits + a11y hardening (week 10–13)**  
Invites, visitor render-only client, revoke 410, narration polish, AA pass, screen-reader dogfood.

**M7 — perf, dogfood, ramp (week 12–16)**  
Budgets in CI, 30-min heap, synthetic fleet, internal dogfood with real weeks of drift.

Do not launch M4+ if M1/M2 feel canned. Aliveness is the release gate, not feature count.

### 11.2 Bird-count ramp

Production starts with the two-bird lock. Age unlocks are live but the first third-bird wave is ~6 weeks post-launch by construction. Staging uses a `sim_version` flag to accelerate age for QA only; that flag is impossible to set on production accounts.

### 11.3 Launch instrumentation (day one)

- first-bird timing
- audio unlock / fallback rate
- tick p99
- presence-qualify ratio (qualified pings / all pings) as an **ops** health signal for probe bugs — stored without account dimension
- JS error rate
- magic-link consume success

Do not instrument “DAU of offers” as a success metric. Success is qualitative dogfood: do people sit, and do birds feel like the same birds in week 3?

### 11.4 Feature flags

Coarse: `visits_enabled` (can disable the whole social surface), `adopt_enabled`, `weather_enabled`. No flag for streaks, because streaks do not exist in the code.

### 11.5 Support and legal surfaces

Privacy policy link in settings: names operational aggregates; explicitly excludes per-bird history. Export and delete documented in matter-of-fact language.

---

## 12. Risks

### 12.1 Drift calibration

**Failure:** too fast → Tamagotchi numbers; too slow → screensaver; presence too loose → population-wide over-drift.

**Mitigations:** three-signal presence; union across devices; `(1-x)` low-pass; week-1/week-3 fixture tests committed in `sim-worker`; staging soak with recorded event tapes; `k` is a single constant in one file. If users report session-to-session change, cut `k` by 2× before adding features. If week-4 dogfooders notice nothing, raise `k` by 1.4× once — not per-trait knobs in a meeting.

### 12.2 Sync correctness

**Failure:** double presence; unprocessed event cursor skip; optimistic offer that the tick never saw; rename LWW confusing someone.

**Mitigations:** transactional tick + cursor; idempotent event ids; presence union; no client trait writes; snapshot etags; property tests: shuffled event batches commute on traits (adds only). Chaos: kill worker mid-tick, assert lease expiry reruns cleanly.

### 12.3 Audio uncanniness

**Failure:** looping feel, phase-cancel chorus, motif set too small, nightjar scary, listen-in sounding like a DAW solo.

**Mitigations:** no samples; jitter; never mute others; listening QA checklist per species × mood; caption/audio pairing review; if a motif is recognized as “the same file,” it is a bug. Budget time for a composer/engineer pass in M2, not a plugin pack.

### 12.4 Accessibility regressions

**Failure:** live-region flood; canvas-only birds; reduced-motion as `animation: none` blankness; captions as `chirp`.

**Mitigations:** a11y in M1/M2, not M6-only; cadence caps; designed reduced-motion renderer; copy lints; weekly VoiceOver/NVDA pass as a release blocker equal to FPS.

### 12.5 First-frame / perf miss

**Failure:** spinner culture creeps in; 2MB blown by a dependency; 500ms missed on 4G; heap leak in grains.

**Mitigations:** quiet-field only; bundle size CI; no Three.js/no heavy UI kit; worklet pool; first-bird mark in synthetics. Dependency additions require a size note.

### 12.6 Voice / announcement creep

**Failure:** a “harmless” welcome toast, notebook that praises attendance, visit badge.

**Mitigations:** copy lints; PR template checkbox for the five principles; no notification infrastructure except the one opt-in email. Reviewers reject engagement copy on sight.

### 12.7 Privacy leakage

**Failure:** email in logs; events in warehouse; “average curiosity” dashboard.

**Mitigations:** HMAC lookup only; CI grep for `email` in log formatters; network policy between DBs; design-doc rule that new metrics must cite §10.3.

### 12.8 Identity continuity

**Failure:** species catalog update regenerates birds; migration re-seeds `call_seed`; export/import used as reset.

**Mitigations:** `birds.id` immutable; `call_seed` immutable; migrations are additive; there is no “regenerate aviary” admin action.

### 12.9 Offer / listen-in saturation

**Failure:** user mashes offers, curiosity hits 0.97 in a weekend.

**Mitigations:** 4-minute per-bird cooldown; small `w_i`; `(1-x)` term. Load test a hostile clicker; traits must still be < 0.05 Δ over one day.

### 12.10 Visit feature social pressure

**Failure:** hosts prettify for guests; notify-opt-in becomes growth.

**Mitigations:** visitor sees exact host snapshot; no show-off render; notify off by default and unmentioned in onboarding; no public lists.

---

## 13. Testing strategy (so the plan is executable)

- **Unit:** presence qualifier, drift monotonicity, mood priors, notebook lint, caption expander, magic-link expiry.
- **Property:** trait updates are monotone and commutative over event order for presence-union correctness.
- **Golden audio:** motif fingerprints (spectral, not samples) stay within identity bands across mood.
- **Visual:** first-frame is not pose 0; reduced-motion has no leaf particles; settle undo window.
- **e2e:** magic-link (mail catcher), offer cooldown, visit revoke → 410, multi-tab presence union.
- **Perf CI:** bundle, heap 30m, tick p99 on a 10k-account fixture (synthetic, no real PII).
- **A11y CI:** axe on chrome/settings; plus manual AT notes in the release template.

---

## 14. Repo and team shape

Monorepo:

```
/apps/web          client
/apps/api          Hono gateway
/apps/sim          Go worker
/packages/contracts OpenAPI + event JSON Schema
/packages/naturalist-copy  shared prose tables + lints
/packages/species  rigs + grammar templates
```

Engineers can own vertical slices (render, audio, sim, accounts) but contracts are reviewed by everyone. Do not stand up a “growth” workstream.

---

## 15. Open points resolved by this plan

| Ambiguity | Call |
|---|---|
| Exact presence idle window | 240s |
| Tick cadence | 60s + 0–10s jitter |
| Trait interval | [0, 0.97] |
| Offer cooldown | 4 minutes / bird |
| Listen-in other-bird gain | 0.22 over 2.4s |
| Age unlock schedule | 0 / 42 / 120 / 240 / 365 / 540 days |
| Species count and nightjar | 6, one nocturnal |
| Dual-device presence | union, not sum |
| Narration authorship | client composer + optional server idle seed |
| Boot snapshot staleness | ≤ 90s OK |
| Deletion limbo ticks | paused |
| Visit notify | opt-in email, 1/visitor/24h, no in-product badge |
| Backend languages | TS gateway, Go worker, Postgres, Redis |
| Renderer | custom canvas scene graph, Preact chrome |
| Activity of ornaments | client-only |

If a later calibration note revises a number, it revises the number, not the architecture.

---

## 16. What not to build while building this

Do not add a debug overlay of trait floats in production. Do not add a staging-only overlay that then ships. Do not add service workers that fake ticks offline. Do not add push. Do not add a public changelog of “your bird grew.” Do not implement the product as a game loop with scores underneath a skin.

Ship a window that was already going. Everything in this plan is in service of that sentence.
