# Pocket Aviary — v1 implementation plan

This plan turns the PRD into an executable build. It does not restatement-walk the spec. Where the PRD leaves a number, a stack choice, or a sequencing decision open, this document makes a defensible call and marks it as such. A separate engineering team should be able to implement from this file plus the PRD, without further product clarification.

**Planning stance.** Aliveness is the product. Every subsystem is judged by whether it preserves “the aviary was already going when you opened the tab,” “the bird noticed you,” and “three weeks later Pip is not the same bird, and nobody told you.” If a conventional product pattern (spinner, toast, last-write-wins, recorded loops, ARIA state dump, streak-adjacent telemetry) would be faster, it is still wrong.

---

## 1. Scope

### 1.1 In v1

- Web-only client, last two major versions of Chrome, Safari, Firefox, Edge.
- Single-user accounts, email magic-link auth, one canonical aviary per account.
- Two starter birds; hard cap of seven. New birds become available by aviary age, not attention.
- Server-authoritative simulation: personality, mood, perch choice, greeting selection, notebook facts, weather, day/night.
- Client rendering + procedural WebAudio + presence sensing + interaction event emission.
- Multi-device sync as a property of the architecture (not a merge protocol).
- Field notebook, listen-in, offers (seed / song fragment / still pool), settle.
- Visit invitations: off by default, per-invite opt-in, read-only, revocable.
- Accessibility as designed surfaces: screen-reader narration, reduced-motion mode, call captions, keyboard navigation, WCAG AA chrome.
- Account export, session revoke, email change, 30-day soft delete then hard delete.

### 1.2 Out of v1 (non-goals, enforced)

- Native apps. Do not design protocols or the data model around native-client constraints.
- Gamification of any flavor, including quiet settings-page disguises and notebook lines about the user’s visit frequency.
- Tamagotchi mechanics: death, hunger, distress, decaying happiness, punishment for absence.
- Social network surfaces beyond the single visit-invite affordance: no profiles, follows, discovery, comments, co-presence, leaderboards, “your friend visited” defaults.
- Payments, multi-aviary accounts, shared/household aviaries, customizable scenes, catalog-style species picking, recorded-audio fallback, personality numbers on any surface.

### 1.3 Locked product rules that become engineering invariants

These are not style notes. They are testable invariants.

1. Clients never write personality vectors, mood, or perch as absolute values.
2. Drift deltas are non-negative. Neglect never decrements a trait.
3. Presence requires `visibilityState === 'visible'` AND window focus AND recent pointer/key activity. Any one signal alone is insufficient.
4. No welcome toast, banner, modal, or “you’ve been gone N days” surface exists in the product tree.
5. No streak, visit-count, green-dot calendar, XP, badge, or “birds adopted: N” widget exists in the product tree.
6. Personality vector numbers are never in any user-facing surface, including settings and export-adjacent UI. Export JSON may include vectors because the user asked for their data; the live product must not display them.
7. First painted aviary frame is mid-motion or a quiet field — never a spinner, never fade-from-static.
8. Calls are procedural. There is no recorded-call asset path, including as a WebAudio fallback.
9. Visitor sessions write no presence and no interaction events that feed drift.
10. Telemetry pipelines never read the simulation database. Simulation rows never land in the analytics warehouse.
11. Internal identifiers are synthetic UUIDs. Email is stored once, encrypted, on the account row.

### 1.4 Defensible calls (ambiguities closed here)

| Topic | Call | Why |
|---|---|---|
| Trait range | Each trait is `numeric(6,5)` in `[0, 1]` | Small, comparable, easy to clamp; 5 decimal places is enough for week-1 instrument sensitivity |
| Starter seeds | Species prior in `[0.22, 0.42]` plus `N(0, 0.03)` noise, clamped | Room to drift up; starters feel distinct without being maxed |
| Mood enum | `wary`, `content`, `curious`, `drowsy`, `alert` | Matches the PRD’s working set; do not add more in v1 |
| Tick cadence | 60s logical tick | PRD’s “~once per minute”; even wall-clock for catch-up math |
| Presence activity window | Start at **180s**, calibrate in `[120, 240]` | PRD says lean long; watching without moving is the product |
| Presence ping | Every 15s while the conjunction holds; implicit end if a ping is missed for 45s | Survives brief focus blips; does not count a forgotten tab |
| Offer cooldown | 4 minutes per (bird, offer-kind) | Long enough to stop session-saturation of curiosity; short enough to try a second gesture |
| Notebook sparsity | Default ≥ 48h between entries; noteworthy events may write once per 12h | “Every few days,” with a valve for rain / first-greeter-this-week / new arrival |
| Third-bird pacing | Age gates in §6.9 | Matches “a few months → third; about a year → five or six” |
| Species pool | Six species in §6.8, including one nocturnal | Coherent place-set; night is not a dead state |
| Prose generation | Deterministic composition engine, no third-party LLM | Per-bird events must not leave our systems |
| Snapshot contents | Derived `renderProfile`, not named trait fields | Stops the client from becoming a stats console |
| Simulation advancement | Lazy catch-up on read/event + live sweeper for sessions with an open presence window | Observationally identical to “always ticking,” scalable |
| Frontend | TypeScript, Vite, Preact for chrome, custom Canvas 2D scene graph | Bundle budget; 60fps idle; no game-engine tax |
| Backend | TypeScript / Node 22 / Hono, Postgres 16, Redis snapshot cache, SES or Postmark | Shared schema package; boring ops |
| Timezone | IANA tz on the account, refreshed from the client at session start if it changed | Local morning = aviary morning |

---

## 2. Architecture

### 2.1 Shape

Four deployables, one database of record.

```
                    ┌──────────────────────────────────────────┐
   Browser          │  edge (HTML + critical CSS + optional    │
   (scene, audio,   │   inlined snapshot for warm sessions)    │
    presence)       └───────────────┬──────────────────────────┘
           │                        │
           ▼                        ▼
   ┌───────────────┐      ┌─────────────────────┐
   │ API (Hono)    │─────▶│ Postgres            │
   │  auth         │      │  accounts, birds,   │
   │  snapshots    │      │  event_log,         │
   │  events       │      │  notebook, visits,  │
   │  visits       │      │  sessions           │
   │  accounts     │      └──────────▲──────────┘
   └───────┬───────┘                 │
           │                  ┌──────┴──────────┐
           ▼                  │ Tick worker     │
   ┌───────────────┐          │  catch-up +     │
   │ Redis         │          │  live session   │
   │ snapshot cache│          │  sweeper        │
   │ magic-link jti│          └─────────────────┘
   │ rate limits   │
   └───────────────┘

   Email worker: magic links, visit invites, export download links, optional visit-notify.
```

No client-to-client channel. No websocket required in v1. Snapshots are pull. Events are POST. Live feel comes from interpolation plus a 20–30s keepalive while visible.

**Call:** skip WebSockets in v1. A 20–30s snapshot pull is well inside the slow simulation cadence, keeps the battery story simple, and avoids a second consistency path. Revisit only if listen-in-to-notebook latency or visit-revoke “next pull” feels too slow; revoke can additionally be bounded by a 10s visitor keepalive.

### 2.2 Client / server split

**Server owns (canonical, persisted):**

- Account, email (encrypted), timezone, settings, deletion state.
- Birds: stable id, species, name, personality vector, mood, mood-entered-at, perch zone, animation phase seed, adoption time.
- Aviary: created-at, weather state, settled flag (session-scoped, not durable across tab close), pending arrival.
- Append-only interaction event log.
- Derived-but-persisted: `recent_presence_ema`, last-presence-at, last-greeter, notebook entries, visit invites/log.
- Tick cursor: `last_tick_at`, `event_log_seq_applied`.

**Client owns (ephemeral, never source of truth):**

- Scene graph, interpolation, idle micro-motion, ambient leaf/feather ornaments.
- WebAudio graph, listen-in mix ramps, caption strings generated from the call that was actually synthesized.
- Presence conjunction monitor.
- Optimistic UI for listen-in and settle (events still go to the log).
- Chrome: top bar fade, settings, notebook scroller, offer sheet, auth screens.

**Client never owns:** personality, mood transitions, perch assignment, greeting selection, notebook prose, weather schedule, adoption eligibility, visit authorization.

### 2.3 Render-pipeline boundary

The renderer consumes a `AviarySnapshot` plus a local clock. It does not call the simulation. It may:

- Interpolate perch positions and body poses between snapshot `poseAt` values.
- Run client-only ornaments (leaves, feathers, parallax) from a seeded RNG that is *not* part of canonical state.
- Run idle micro-cycles (preen, scan, fluff) whose *style* is selected by `renderProfile.idleFamily`, but whose *phase* is continuous and local.

It may not:

- Decide that a bird should move zones because the user has been watching.
- Invent a greeting.
- Advance mood or personality.
- Persist anything except a small local cache of the last snapshot for quiet-field continuity on the next same-origin load (cache is a hint; server snapshot always wins).

### 2.4 Shared schema package

`packages/schema` is the contract: Zod (or equivalent) types for events, snapshots, account DTOs, visit DTOs. API, tick worker, and client all import it. A CI test fails if the client snapshot type gains a field named like a personality trait (`boldness`, `socialWarmth`, `vocalFrequency`, `plumageSaturation`, `curiosity`).

### 2.5 Privacy architecture (not a policy add-on)

Two data planes, no shared readers.

| Plane | Contents | Readers |
|---|---|---|
| Simulation | birds, vectors, events, notebook, presence intervals, visit log | API, tick worker, account-export job, hard-delete job |
| Operations | request counts, tick latency, error rates, anonymized session-duration histograms, TTFB, first-bird timing, fps, audio-context errors | metrics backend, synthetic probes |

Rules:

- Simulation DB credentials are not issued to the metrics pipeline.
- Operational events are keyed by synthetic account UUID only when needed for *this-account-is-erroring* debugging, and those keys are stripped before any warehouse export. Prefer request-id + anonymous session-duration buckets with no account dimension.
- No third-party LLM, analytics SDK that fingerprints users, or marketing pixel.
- Email appears in the account row and in the transactional-mail provider. Nowhere else: not logs, not Kafka keys (we are not introducing a bus that uses email), not error messages.

### 2.6 Service modules (logical, even if one API process)

1. **Auth** — magic-link issue/consume, session tokens, revoke, email change.
2. **Aviary read** — catch-up tick, build snapshot, cache.
3. **Events** — validate and append; never apply drift inline except where a synchronous reaction is required (offer outcome, greeting plan). Those reads use current state *after* catch-up; they still do not write vectors.
4. **Tick** — consume events, apply drift, mood, weather, circadian pull, notebook candidate, arrival eligibility.
5. **Notebook composer** — fact → sparse prose.
6. **Visits** — invite, redeem, revoke, visitor snapshot scope, host log.
7. **Accounts** — settings, export, delete/recover.
8. **Narration composer** — snapshot → naturalist prose block (can run server-side so SR and visual stay aligned; client may also compose from the same pure function shipped in `packages/prose` to avoid a round trip on local events).

`packages/prose` is a pure function library: facts in, lowercase present-tense strings out. Used by notebook, narration, and captions. No I/O.

---

## 3. Data model

Postgres 16, UUID primary keys (`gen_random_uuid()`), `timestamptz` everywhere.

### 3.1 `accounts`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | Synthetic. The only identifier used elsewhere |
| `email_ciphertext` | bytea | Encrypted at application layer (AES-GCM, key in KMS). Unique via `email_hash` |
| `email_hash` | bytea unique | HMAC of normalized email; used for lookup, not logged |
| `email_verified_at` | timestamptz | |
| `pending_email_ciphertext` | bytea null | Email-change flow |
| `pending_email_hash` | bytea null | |
| `timezone` | text | IANA, default `UTC` until first session |
| `created_at` | timestamptz | Aviary age clock starts here |
| `deleted_at` | timestamptz null | Soft delete |
| `hard_delete_after` | timestamptz null | `deleted_at + 30d` |
| `visit_notify_opt_in` | bool | Default `false` |
| `reduced_motion_opt_in` | bool null | `null` = follow `prefers-reduced-motion` |
| `captions_opt_in` | bool | Default `false`; forced `true` when WebAudio unavailable |
| `settings` | jsonb | Theme-less; a11y extras, settle-on-blur? no |

No `display_name`, no public handle, no avatar.

### 3.2 `sessions`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | Token id (hash of the cookie lives here; raw token only on the client) |
| `account_id` | uuid fk | |
| `created_at` | timestamptz | |
| `last_seen_at` | timestamptz | |
| `user_agent_hash` | bytea | For the device list; store a parsed label (`"Safari on iPhone"`) not the raw UA in UI |
| `revoked_at` | timestamptz null | |
| `kind` | enum(`owner`, `visitor`) | Visitor sessions are scoped to an invite |

Cookies: `HttpOnly`, `Secure`, `SameSite=Lax`, 30-day idle lifetime, rotated on magic-link consume.

### 3.3 `aviaries`

One row per account (v1). Keep it a table anyway so a future multi-aviary world is a migration, not a rewrite — but **do not expose** more than one.

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | |
| `account_id` | uuid unique fk | |
| `last_tick_at` | timestamptz | Catch-up cursor |
| `event_seq_applied` | bigint | |
| `weather` | enum(`clear`, `rain`, `wind`) | |
| `weather_until` | timestamptz | |
| `pending_arrival_species` | text null | Age-gate offer in flight |
| `recent_presence_ema` | numeric | Hours-scale EMA; **not** a personality trait |
| `last_presence_at` | timestamptz null | |
| `last_settle_at` | timestamptz null | |

`recent_presence_ema` is how ambient quietness is implemented without negative drift. It decays toward 0 with elapsed time and rises with presence-time. Greeting probability and unobserved call rate are multiplied by a function of this EMA. Personality stays put.

### 3.4 `birds`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid pk | Stable identity. Never recycled |
| `aviary_id` | uuid fk | |
| `species_id` | text | One of the six pool ids |
| `name` | text | User-assigned; default suggestion at adoption |
| `sort_index` | smallint | Adoption order, not a rank |
| `boldness` … `curiosity` | numeric(6,5) | Five trait columns, server-only |
| `mood` | enum | |
| `mood_entered_at` | timestamptz | |
| `perch_zone` | enum(`front`, `middle`, `back`) | |
| `perch_slot` | smallint | Slot within zone so two birds don’t occupy one pixel |
| `pose_seed` | int | Lets a snapshot resume mid-preen |
| `adopted_at` | timestamptz | |
| `call_signature_seed` | int | Stable timbre/pitch identity |

Indexes: `(aviary_id)`. Soft-deleted accounts cascade via account deletion job, not per-row bird deletes during life.

**Identity rule:** rename, species-pool edits, and migrations update columns. They never insert a replacement row for an existing bird. If a species motif library changes, `call_signature_seed` plus species id still resolve; ship motif versions keyed by species so old seeds remain valid.

### 3.5 `event_log` (append-only)

| Column | Type | Notes |
|---|---|---|
| `seq` | bigserial | Total order per… |
| `aviary_id` | uuid | …aviary (`unique (aviary_id, seq)` via serial + index) |
| `account_id` | uuid | Writer; visitor events that are *not* drift-eligible use a separate `visit_log` |
| `type` | text | See §4.2 |
| `bird_id` | uuid null | |
| `payload` | jsonb | Type-specific, no personality values |
| `client_event_id` | uuid | Idempotency |
| `device_session_id` | uuid | |
| `created_at` | timestamptz | Server receipt time |
| `client_occurred_at` | timestamptz | For ordering hints; tick uses receipt order as authority |

No updates, no deletes except hard-delete of the account.

Idempotency: unique `(aviary_id, client_event_id)`.

### 3.6 Presence intervals (optional materialized view of pings)

Store either raw `presence_ping` events or collapse them in the API to intervals. **Call:** collapse at ingest into `presence_intervals(aviary_id, started_at, ended_at, seconds)` so the tick does not re-parse ping storms. A ping stream with gaps > 45s closes the interval.

### 3.7 `notebook_entries`

| Column | Type | Notes |
|---|---|---|
| `id` | uuid | |
| `aviary_id` | uuid | |
| `observed_on` | date | Local-tz date, for “tuesday — …” |
| `body` | text | Already-composed prose, stored immutable |
| `fact_key` | text | Dedup: `first_greeter:2026-08-12` |
| `created_at` | timestamptz | |

Read-only to all clients. No edit/delete/annotate API.

### 3.8 Offers / cooldowns

`offer_cooldowns(bird_id, kind, available_at)`. Kinds: `seed`, `song`, `pool`.

### 3.9 Visits

`visit_invites(id, host_account_id, guest_email_hash, guest_email_ciphertext, token_hash, created_at, expires_at, redeemed_at, revoked_at, last_visit_at)`

`visit_log(id, invite_id, host_account_id, started_at, ended_at, duration_s)` — written from visitor session start/end. **Not** fed to the tick.

Outstanding invites expire 30 days after create if `redeemed_at is null`. Redeemed invites remain until revoked; a redeemed invite can be reused by that visitor until revoke or we treat the link as one-time.

**Call:** the email link is one-time for *establishing* a visitor session cookie (30-day). The invite row stays “active” until revoked. This matches “revoke outstanding or active” without forcing the friend to click email every visit. Unused invites expire in 30 days. Document this in settings in matter-of-fact voice.

### 3.10 Auth artifacts

`magic_links(id, account_id or pending email hash, jti, expires_at, consumed_at, purpose enum(sign_in, email_change, export_download))`. TTL 15 minutes. Consume is an atomic `UPDATE … WHERE consumed_at IS NULL AND expires_at > now()`.

Rate limit: 5 issued links per email-hash per 15 minutes, plus a 20/hour cap. Store counters in Redis.

### 3.11 What the export JSON contains

Generated on demand, not stored long-term (signed URL to a 24h object):

```json
{
  "exported_at": "…",
  "account_id": "…",
  "timezone": "America/New_York",
  "aviary_created_at": "…",
  "birds": [
    {
      "id": "…",
      "species_id": "wren",
      "name": "Pip",
      "personality": {
        "boldness": 0.41,
        "social_warmth": 0.37,
        "vocal_frequency": 0.33,
        "plumage_saturation": 0.29,
        "curiosity": 0.44
      },
      "mood": "content",
      "adopted_at": "…"
    }
  ],
  "notebook_entries": [{ "observed_on": "…", "body": "…" }],
  "settings": { "visit_notify_opt_in": false, "captions_opt_in": false }
}
```

Export is the one place personality numbers leave the server toward the user, because they asked. The in-app UI still never renders them.

### 3.12 What is deliberately not modeled

- Scores, streaks, visit-day sets, achievements.
- Hunger, health, death clocks.
- Friend graphs, follows, public handles.
- Per-leaf simulation state.
- Client-authoritative animation timelines.

---

## 4. API surface

Base: `https://api.pocket-aviary.example/v1`. JSON. Session cookie. All owner routes 401 if missing/revoked; 403 if `deleted_at` set unless the route is recover.

Voice: error bodies for these routes are matter-of-fact English, not naturalist.

### 4.1 Auth

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/auth/magic-link` | Body `{ email }`. Always 202 with a generic ack (do not reveal account existence). Issue or create-pending. |
| `GET` | `/auth/consume?token=` | One-time consume. Set session cookie. New email → create account + aviary + two birds, then redirect to `/`. |
| `POST` | `/auth/logout` | Revoke this session |
| `GET` | `/me` | Account settings DTO (no vectors, no email plaintext in logs) |
| `GET` | `/me/sessions` | Device list |
| `DELETE` | `/me/sessions/:id` | Revoke |
| `POST` | `/me/email-change` | Start verify-new-email |
| `POST` | `/me/export` | Enqueue export email |
| `POST` | `/me/delete` | Soft delete |
| `POST` | `/me/recover` | Clear `deleted_at` inside 30 days |
| `PATCH` | `/me/settings` | a11y + visit-notify only |

Magic-link HTML email is matter-of-fact. No “your birds miss you.”

Account creation side effects, transactional:

1. Insert `accounts` with synthetic id.
2. Insert `aviaries`.
3. Insert two `birds` (species pair from §6.8).
4. No notebook entry yet. No welcome event.

### 4.2 Owner aviary

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/aviary/snapshot` | Catch-up, return `AviarySnapshot`. Query: `reason=visible\|keepalive\|resume\|start` |
| `POST` | `/aviary/events` | Batch append. Body `{ events: Event[] }` |
| `POST` | `/aviary/offers` | Synchronous offer attempt. Returns reaction plan |
| `POST` | `/aviary/session-start` | Returns snapshot + `greetingPlan` |
| `POST` | `/aviary/rename` | `{ birdId, name }` |
| `POST` | `/aviary/accept-arrival` | Name a pending third+ bird; or auto-name if skipped |
| `GET` | `/aviary/notebook?before=&limit=` | Reverse chronological, default 30 |

#### Event types the client may submit

```
presence_ping        { }                         // only while conjunction holds
presence_end         { reason: blur|hidden|idle|settle|unload }
listen_in_start      { birdId }
listen_in_end        { birdId, durationMs }
settle_start         { }
settle_undo          { }
offer_intent         { kind, nearBirdId? }       // logged; reaction via /offers
pointer_or_key       // do not send raw inputs; presence_ping is enough
```

Reject unknown types. Reject events with personality fields. Reject visitor sessions on this route.

Batch max 50. Each event has `clientEventId`.

#### Snapshot DTO (client-visible)

```ts
type AviarySnapshot = {
  serverTime: string
  localHourHint: number          // 0–23 in account tz, so chrome can paint sky before tz js
  weather: 'clear' | 'rain' | 'wind'
  weatherIntensity: number       // 0–1, already decaying
  lighting: {
    phase: 'dawn' | 'morning' | 'midday' | 'evening' | 'night'
    warmth: number
    dim: number
    settled: boolean
  }
  birds: BirdSnapshot[]
  greetingPlan: GreetingPlan | null
  pendingArrival: { speciesId: string; defaultName: string } | null
  notebookUnreadHint: boolean    // not a badge count; just whether the icon should feel “there is a new page”
  activeOffer: OfferVisual | null
}

type BirdSnapshot = {
  id: string
  name: string
  speciesId: string
  perch: { zone: 'front' | 'middle' | 'back'; slot: number; x: number; y: number }
  pose: { family: IdleFamily; phase01: number }
  plumage: { paletteId: string; saturation: number; patternSeed: number }
  moodHint: Mood                 // for motion family only; never shown as text
  call: {
    signatureSeed: number
    nextWindowHintMs: number     // scheduler hint, not a countdown UI
    rate: number                 // 0–1, already includes EMA quietness
  }
  renderProfile: {
    approachBias: number         // derived from boldness + mood
    socialBias: number
    idleFamily: IdleFamily
    headTiltiness: number
    fluff: number
  }
}

type GreetingPlan = {
  steps: { birdId: string; delayMs: number; form: GreetingForm }[]
  absenceClass: 'moment' | 'hours' | 'days'
}

type GreetingForm =
  | 'glance_from_preen'
  | 'two_note_call'
  | 'head_tilt_step_forward'
  | 'longer_call'
  | 'call_and_answer'
```

No trait names. No “days since visit.” `absenceClass` is a coarse bucket used only to pick greeting form, not to render copy.

`notebookUnreadHint`: implement as “there is an entry newer than last notebook open,” stored as `accounts.notebook_seen_at`. The icon does not grow a numeric badge.

### 4.3 Offer endpoint

`POST /aviary/offers { kind: 'seed'|'song'|'pool', nearBirdId?: string, clientEventId }`

Server:

1. Catch-up tick.
2. Enforce 4-minute cooldown per bird for that kind. If cooling down, return `{ accepted: false, reason: 'cooldown', availableAt }` — client plays a small “already offered” still-life, no toast.
3. Choose receiving bird: `nearBirdId` if still in front/middle and not drowsy-asleep; else the bird with highest `curiosity * approachBias` that is not in cooldown.
4. Draw reaction from mood × curiosity × kind (table in §6.6).
5. Append `offer_resolved` **server** event (not client-authored) with `{ kind, birdId, reaction }`.
6. Return `{ accepted: true, birdId, reaction, motionPlan, callMotifHint, cooldownUntil }`.

Client starts motion immediately from `motionPlan`. Drift happens later in the tick from `offer_resolved`.

### 4.4 Visit flow

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/visits/invites` | `{ email }` — host only. Creates invite, emails one-time link |
| `GET` | `/visits/invites` | Outstanding + active, plus visit log |
| `DELETE` | `/visits/invites/:id` | Revoke immediately |
| `GET` | `/visits/redeem?token=` | Sets `kind=visitor` session cookie, redirects to `/visit` |
| `GET` | `/visits/snapshot` | Same visual snapshot as host, minus notebook, pending arrival controls, settled-as-host. If revoked/expired: `410` with matter-of-fact body |
| `POST` | `/visits/heartbeat` | Updates visit_log duration; **does not** write presence |

Visitor client: render-only build. No offer/listen-in/settle/notebook routes wired. Keyboard still moves a *visual* focus ring for SR users describing what they are looking at, but it must not change the mix or write events. **Call:** visitor listen-in is disallowed even locally. The mix stays ambient. Otherwise a visitor would have a private interactive surface that the PRD forbids, and we would be tempted to log it.

Revoke takes effect on next visitor snapshot/heartbeat (≤10s). Return:

```
The visit is no longer available.
```

### 4.5 Snapshot pull triggers (client)

Pull on:

- `session-start` (dedicated route, includes greeting).
- `document.visibilityState` → `visible`.
- `pageshow` after `persisted` (bfcache) or a `visibility` resume with `performance.now()` gap > 5s (laptop sleep).
- Keepalive every 25s while visible (owner) / 10s (visitor, for revoke).
- After settle undo, after offer return, after accept-arrival.

Do not pull on every pointermove.

### 4.6 Idempotency and auth failure copy

Use the PRD’s matter-of-fact strings verbatim for the three named cases; add only equally plain variants:

- Magic link expired / reused.
- Session timed out.
- Snapshot load failed.

Never: “the aviary could not find its way back to you.”

---

## 5. Simulation engine design

### 5.1 Tick contract

`tick(aviary, birds, eventsSince, from, to, tz) → { aviary, birds, notebookCandidates, serverEvents }`

Pure enough to replay. Same inputs → same outputs. Wall time is divided into 60s logical ticks. Catch-up of 24h = 1440 iterations, but **fast-forward** empty stretches:

- If no events in a span and weather is clear, collapse circadian/mood pulls into hourly steps rather than 1440 minute steps.
- Always apply at least hourly circadian + EMA decay during absence.

**Live sweeper:** every 30s, select aviaries with an open presence interval (`ended_at is null`) via `FOR UPDATE SKIP LOCKED`, run catch-up to now, refresh Redis snapshot. Connected tabs therefore see mood/weather change without waiting for their next action.

**Idle accounts:** no sweeper. Next `GET snapshot` or `session-start` catch-up is the continuity.

This satisfies “the aviary continues without the viewer” without a minute-cron across the whole user base.

### 5.2 Personality vector

```
traits = { boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity } ∈ [0,1]
```

Stored on the bird. Updated only inside `tick`. Never decremented.

#### Drift as a low-pass of positive signals

Per tick, for each bird:

```
presence_hours = overlap(presence_intervals, tick_window)   # 0–1/60 typically

listen_hours = overlap(listen_in windows on this bird, tick_window)

offers_near = count(offer_resolved this tick where near this bird)
offers_accepted = count(offer_resolved this tick where reaction in APPROACH_SET)

# base presence contribution (dominant)
Δ = presence_hours * α_presence * presence_weights

# listen-in (strong, targeted)
Δ.social_warmth      += listen_hours * α_listen
Δ.vocal_frequency    += listen_hours * α_listen

# offers
Δ.curiosity          += offers_accepted * α_offer_accept
Δ.boldness           += offers_near     * α_offer_near

# settle: no Δ

trait = min(1, trait + Δ)
```

**Calibration constants (starting point; lock behind a versioned `drift_v1` config):**

Assume a “regular” user: 4 sessions/week × 20 minutes real presence = 1.33 presence-hours/week.

Target: week-1 instrument Δ ≈ `0.015–0.030` on the most-touched trait (usually plumage + warmth if they listen in). Week-3 user-visible ≈ `0.05–0.09`.

```
α_presence        = 0.012 / hour     # split across traits by weights
presence_weights  = {
  boldness: 0.25,
  social_warmth: 0.25,
  vocal_frequency: 0.15,
  plumage_saturation: 0.25,
  curiosity: 0.10
}
α_listen          = 0.020 / hour     # on top of presence, targeted
α_offer_accept    = 0.004 / event
α_offer_near      = 0.002 / event
```

With presence only, 1.33h × 0.012 × 0.25 ≈ **0.004** per week per trait — too slow. Presence is dominant *relative to clicks*, but the absolute α needs to hit the week-1 instrument bar.

Revised:

```
α_presence = 0.040 / hour
```

1.33 × 0.040 × 0.25 ≈ **0.013**/week from presence alone. Add listen-in (say 10 min/week on one bird): 0.167 × 0.020 ≈ **0.003** extra on warmth/vocal. After one regular week, instruments see ~0.013–0.016; after three weeks ~0.04–0.06, plus offers. Visible perch/greeting change around 0.06–0.08 of boldness/warmth. Good.

**Cap per calendar day** per trait: `0.012`. A 4-hour single sitting cannot burn a week of drift. This is what keeps “no single session shifts a trait visibly.”

**No negative terms. No “decay to mean.”** A two-week absence applies `Δ = 0` to traits. EMA quietness handles the quieter greeting.

#### Ambient quietness (not a trait)

```
ema := ema * exp(-ln(2) * hours_absent / 72) + presence_hours * 0.15
ema := clamp(ema, 0, 1)
```

Half-life ~3 days of absence. Greeting chance and unobserved call rate multiply by `0.35 + 0.65 * smoothstep(ema)`. After two weeks away, birds still call and still live; they greet less. They do not dull in color.

### 5.3 Mood

Enum: `wary | content | curious | drowsy | alert`.

Mood is persisted. Tab open does not reset it.

Each tick, compute a **pull distribution** and sometimes transition:

```
prior = circadian_prior(local_hour, species)
  dawn/morning: alert, curious
  midday: content, curious
  evening: drowsy, content
  night: drowsy (nightjar: alert)

modulate:
  rain        → +wary +drowsy  −vocal behavior (see calls)
  wind        → +alert, some species +wary
  offer_accepted recently (≤20 min) → +content +curious
  other bird currently wary → +wary * contagion(0.25)
  high boldness → scale down wary probability (× (1 - 0.6*boldness))
  high curiosity → scale up curious
  settle lighting (host session only, ephemeral) → +drowsy

if rand() < p_transition (base 0.08 per minute, higher at hour boundaries):
  mood = sample(pull)
  mood_entered_at = now
```

Hourly catch-up during absence uses `p_transition` ≈ 0.4 per hour so a bird that ended drowsy at dusk can be content/alert by morning without a snap-to-default at tab open. The user should see continuity: last night’s wary bird is *softer*, not reset.

**Never** set all birds to `content` on session start.

### 5.4 Perch assignment

Zones: front, middle, back. Each zone has 3 slots. Client maps `(zone, slot)` to responsive coordinates.

Every 2–5 minutes of sim time (randomized per bird), consider a move:

```
desired_zone =
  wary     → back-weighted
  drowsy   → middle/back, low slot
  curious  → middle, sometimes front
  alert    → front/middle
  content  → anywhere, warmth pulls toward occupied zones

then mix with boldness: P(front) += 0.5 * boldness
```

User cannot place birds. Snapshot includes both zone and concrete `x,y` in a normalized scene space `[0,1]×[0,1]` so the client interpolates in scene space and then fits to viewport.

Moves take 4–10s of interpolated travel. No teleports unless the catch-up skipped across a move that is already finished (`pose.phase` already at rest on the new perch).

### 5.5 Call-grammar runtime (server side = scheduler; client side = synth)

Server snapshot gives each bird:

- `signatureSeed` (identity)
- `rate` (how often, already EMA-adjusted)
- `nextWindowHintMs` (so the client’s first seconds already have a call in progress — aliveness)

Server scheduler (inside tick / snapshot build):

```
mean_interval_s = lerp(28, 8, vocal_frequency) / (0.35 + 0.65*ema)
mood_mult = { wary: 1.4, drowsy: 1.8, content: 1.0, curious: 0.85, alert: 0.75 }[mood]
rain_mult = weather==rain ? 1.6 : 1.0
interval = Exp(mean_interval_s * mood_mult * rain_mult)
```

Chorus: if bird A called in the last 2s and bird B has high warmth or vocal_frequency, B’s next interval is multiplied by `0.4` (join). Alarm: a wary sharp call adds a one-shot `wary` pull to nearby birds next tick.

The server does **not** synthesize PCM. It may include `callInProgress: { motifId, elapsedMs, captionKey }` so a mid-call tab-open is mid-call.

Client grammar: §8.

### 5.6 Return-greeting

Computed only on `POST /aviary/session-start`, not on every keepalive.

Inputs: each bird’s boldness, warmth, mood, EMA, `now - last_presence_at`.

```
absenceClass =
  < 20 min  → moment   (glance)
  < 18 h    → hours    (two-note or tilt)
  else      → days     (longer call, possible step forward, possible answer)
```

Pick primary greeter: highest `boldness * warmth * greetMoodFit * emaBoost`, with `greetMoodFit` low for drowsy/wary. If the top score is weak (all drowsy at night, except nightjar), only the nightjar glances — or nobody, which is valid.

If a second bird’s warmth is high, append a staggered answer at `220–900ms` random offset. **Never** unison.

Procedural form: choose `GreetingForm` from absenceClass × mood × boldness, then pass `signatureSeed` so the actual notes differ every time.

Store `last_greeter_bird_id` and local date for notebook fact `pip greeted before wren`.

### 5.7 Bird-to-bird

- Call-answer (above).
- Wary contagion (mood pull).
- Chorus window when ≥2 birds with `rate` high overlap.
- Perch: high warmth prefers a slot adjacent to another bird; low warmth prefers an empty zone.

These are tick rules, not client easter eggs.

### 5.8 Weather

A few times a week, short rain; occasional wind.

**Call:** Poisson process on the aviary with λ_rain ≈ 3.5 / week, duration 8–20 minutes; λ_wind ≈ 2 / week, duration 4–12 minutes. No thunder, no snow. Weather is rolled during catch-up so a user returning mid-rain sees rain already happening.

Effects: vocal intervals lengthen in rain; wind increases alert/wary pulls for two species (sparrow, dove). Leaves (client ornaments) densify during wind — visual only.

### 5.9 Circadian lighting (server provides parameters; client paints)

Account timezone. Phases:

| Local hour | Phase | Birds |
|---|---|---|
| 5–7 | dawn | warming, alert pull |
| 7–11 | morning | |
| 11–16 | midday | brightest |
| 16–20 | evening | warmth up, calls quieter |
| 20–5 | night | dim; most drowsy; nightjar active |

Settle overrides lighting toward evening over ~3.5s on the client and sets `lighting.settled=true` on the next snapshot. Settle is session-ephemeral: tab close clears it. Engine treats settle as `presence_end` only.

### 5.10 Notebook composer

Runs at the end of a tick that produced a **fact**, then the sparsity gate decides whether to write.

Fact extractors (examples, closed set in v1):

- `first_greeter` — greeter ≠ yesterday’s, or first time this local week.
- `long_quiet` — no calls for ≥ 8 minutes of presence.
- `weather_passed` — rain just ended.
- `preen_long` — one bird stayed in preen family ≥ 3 minutes (from pose occupancy counters).
- `offer_reaction` — a wary bird eventually approached a seed.
- `night_call` — nightjar called after 22:00.
- `arrival` — a new bird flew in.
- `fluffed_cool` — morning + drowsy + back perch.

Sparsity gate:

```
if last_entry_at > now - 48h and fact not in NOTEWORTHY: drop
if last_entry_at > now - 12h: drop even if noteworthy
```

Composer: pick a template family, fill with **names, perch words, time-of-day words, weather words**. Lowercase, present tense, no “you,” no numbers, no trait names, no visit frequency.

Templates live in `packages/prose`. Example families already suggested by the PRD; add 20–30 variants per fact so the notebook does not loop.

Store the composed `body`. Do not store the personality dump that produced it.

### 5.11 Adoption and age gates

See §6.9. Tick sets `pending_arrival_species` when age crosses a gate and `bird_count < 7` and no pending. Snapshot exposes it. Client, after ~30s of presence in that session, plays a soft back-perch fly-in (same language as first-ever birds). A name field is available from the top-bar offer/arrival affordance — not a modal, not a toast. Default name applies if the user never opens it; the bird is already real.

Species for arrivals: draw from the pool excluding species already at count ≥2 unless the pool is exhausted; prefer unused species. No rarity.

### 5.12 Invariants the tick tests must lock

1. Replay of a week of synthetic “regular presence” produces trait Δ in the instrument band and never a decrease.
2. Replay of two weeks of zero presence: traits unchanged; EMA down; greeting probability down.
3. Two event streams from two devices, interleaved by `seq`, produce one vector; there is no “apply snapshot from device B.”
4. `offer_resolved` authored only by the server.
5. Visitor `visit_log` rows do not change any bird column.
6. Catch-up of 36 hours completes in < 50ms p95 for a 7-bird aviary (hourly collapse).
7. Tick p99 wall time for a live (non-catch-up) pass < 5s (alarm); budget is actually < 20ms.

---

## 6. Sync model

### 6.1 Single writer

Personality, mood, perch, weather, notebook: **tick worker / catch-up inside the API process**, same function, same DB row locks.

```
BEGIN
  SELECT aviary FOR UPDATE
  read birds, events where seq > event_seq_applied
  compute
  write birds, aviary cursors, optional notebook
  UPDATE event_seq_applied
COMMIT
invalidate Redis snapshot
```

Clients append events with `INSERT` only. No `UPDATE birds`.

### 6.2 Why last-write-wins is unreachable

There is no `PUT /birds/:id { boldness: 0.62 }`. A phone that opened an old snapshot cannot clobber a laptop morning. Both emit events; both are applied in `seq` order. If the phone emits `listen_in` against a bird id that still exists, the tick adds a small positive Δ. That is correct, not a conflict.

Rename is last-write-wins on the **name** column (user-facing string, not drift). Acceptable. Use `updated_at` only for name.

Settings patches are last-write-wins per key.

### 6.3 Multi-device felt consistency

Laptop and phone both `GET /aviary/snapshot` after catch-up. They receive the same moods, the same perch zones, the same notebook.

They will **not** share leaf positions or exact preen phase — those are local ornaments / local phase. This is correct: the place is the same; the camera isn’t locked.

Greeting: only the device that called `session-start` after a real absence should play a greeting. Implementation:

- Server records `last_session_start_at`.
- If another device session-starts within 10 minutes, return `greetingPlan: null` (the aviary already noticed someone).
- Presence from either device feeds the same intervals.

**Call:** two simultaneous open tabs *do* both count presence if both pass the conjunction. That is honest attention (user watching on a monitor and a phone is still watching). Drift daily cap still holds.

### 6.4 Sleep / hidden tab

Hidden: client stops rAF and suspends AudioContext (or lets the browser suspend it). Simulation continues via catch-up. On visible: snapshot pull with `reason=visible`, resume AudioContext on the next user gesture if the browser requires it — **except** we already need a gesture for audio on first visit. Keep a silent unlock on first pointer/key.

Long frame gap (`rAF` dt > 5s): treat as resume; pull snapshot; do not interpolate across the gap (that would smear a bird across the scene). Hard-sync pose, then continue.

### 6.5 Conflict / error surfaces

True state conflicts should not occur. Remaining user-visible failures are auth and load:

- Expired magic link.
- Revoked session.
- Snapshot 5xx / timeout → quiet field stays up, matter-of-fact inline in the top bar (not a toast): “Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch.”
- Visitor 410 as specified.

No merge UI. No “pick a version of Pip.”

### 6.6 Offer reactions (server table)

| Mood \ kind | seed | song | pool |
|---|---|---|---|
| curious | approach, peck | join motif | bathe |
| content | approach slower | soft answer | drink |
| alert | approach, then scan | listen, possible answer | watch |
| wary | wait 4–10s, maybe approach | go quiet or single note | watch from back |
| drowsy | ignore or eye-open only | ignore | ignore |

Curiosity scales wait time down and ignore chance down. Cooldown still applies on *attempt*, not only on accept, so mashers cannot farm Δ.

APPROACH_SET for curiosity drift: `approach`, `peck`, `bathe`, `drink`, `join motif`. Near-bird boldness drift: any offer with `nearBirdId` or the chosen receiver.

### 6.7 Starter pair algorithm

`pickStarterPair(accountId)`:

- Exclude pairing two nightjars.
- Prefer complementary silhouettes (one small high-perch, one fuller mid/front).
- Deterministic shuffle of allowed pairs from `accountId` so retries during a failed create don’t flicker species; once committed, stored rows win.

Default names: a short list per species (pip, wren, moss, pebble, soot, lark, …). Avoid cartoon trademark names. User can rename immediately on the first quiet field, still not a catalog.

### 6.8 Species pool (working ids)

| id | silhouette | default perch bias | call character | night |
|---|---|---|---|---|
| `warbler` | tiny, tail flick | high / back-middle | short bright motifs | sleeps |
| `sparrow` | round, social | front | chatter, warm | sleeps |
| `wren` | cocked tail | middle | complex fast motifs | sleeps |
| `dove` | full, low | middle-back | soft two-note | sleeps |
| `finch` | compact, bright face | front-middle | seed-bright chips | sleeps |
| `nightjar` | long wing, cryptic | back | low churr, sparse day | **active** |

Visual assets: SVG body parts (body, tail, eye, wing) composited; plumage saturation is a shader-like multiply in canvas, not a second sprite sheet.

### 6.9 Age gates for additional birds

Aviary age = `now - accounts.created_at`.

| Birds after accept | Unlock at age |
|---|---|
| 3 | 60 days |
| 4 | 150 days |
| 5 | 240 days |
| 6 | 330 days |
| 7 | 420 days |

A year-old aviary has had time to reach 5–6; 7 is a long relationship. Not visit-count, not paid.

---

## 7. Frontend rendering pipeline

### 7.1 Boot sequence (this is the product’s first impression)

1. HTML arrives with critical CSS: full-viewport quiet field (sky gradient from `localHourHint` cookie or a neutral dawn if unknown). **No spinner. No logo splash.**
2. If a warm session cookie exists, the edge may inline `window.__SNAPSHOT__`. If not, the quiet field holds.
3. Critical JS chunk (scene + snapshot apply + SVG sprites) parses. Target **< 80KB gz**.
4. First bird is drawn in its snapshot pose **mid-cycle** (`pose.phase01` ≠ 0). A leaf may already be mid-fall (ornament RNG seeded from `serverTime` so it’s not always the same leaf).
5. Audio chunk loads async; first call may start slightly after first paint. Better a late call than a blocked first bird.
6. Chrome (Preact) hydrates the top bar at opacity 0, fades to 1, then the stillness timer starts.
7. Remaining chunks: notebook, settings, visit, account, offer sheet.

Empty brand-new aviary: quiet field, then after adoption names (or defaults), each starter **soft fly-in** to its perch. That fly-in happens **once in the life of the aviary**. After that, birds are always already there.

Failed/slow snapshot: stay on quiet field. After 3s, matter-of-fact line in the top bar. Never a spinner.

### 7.2 Scene graph

Single horizontal world, no camera pan/zoom/scroll.

```
sky (gradient + sun/moon disc, very soft)
far foliage (few SVG silhouettes)
perch-back
birds in back zone
perch-mid
birds in mid
perch-front
birds in front
near leaves / occasional foreground branch
captions (if on)
focus ring (keyboard / listen-in, not a UI badge)
```

Subtle parallax: foliage shifts a few pixels vs birds over a very slow sine (period 20–40s, amplitude small). Reduced-motion: freeze parallax, keep color shifts.

Responsive: world is normalized; viewport fit is **letterboxed only if extreme**; normally the scene *compresses horizontally* on narrow phones so every bird remains in frame. Never crop a bird. Recompute perch `x` from zone/slot on resize without changing zone.

### 7.3 Idle micro-motion

Never paused-still. Families:

- `preen` — content
- `scan` — wary / alert
- `tilt` — curious
- `fluff_low` — drowsy
- `weight_shift` — all, as connective tissue
- `sleep_eye` — night non-nightjar

Each family is a looped pose cycle (6–12 keys) interpolated with ease-in-out. Personality/mood choose family via `idleFamily`; they do not choose a canned full-body clip from a library of three.

Motion continues conceptually when hidden; we just don’t draw it. On return, phase comes from snapshot + elapsed, not from 0.

### 7.4 Transitions

- Perch change: arc path, 4–10s, wing ticks only if crossing a long distance (front ↔ back). Short hops are steps.
- Offer objects: seed / pool / song are scene objects with their own fade, not DOM overlays in the aviary.
- Settle: 3.5s lighting lerp to evening, call gain master down to ~0.25, birds bias to `fluff_low`. Undo: reverse lerp if click within 5s.

### 7.5 Top bar

Icons only: account, accessibility, notebook, offer. Settle lives here too (fifth icon). Nothing else. After 4s without pointer/key, opacity → 0.12 over 1.2s. Any pointer/key → 1.0 over 180ms. No labels until hover/focus, and those labels are naturalist (`listen`, `offer`, `notebook`, `settle`) except account/a11y which are system (`Account`, `Accessibility`).

Zero chrome inside the scene: no names over birds, no mood icons, no hover tooltips. Names appear in notebook, arrival naming, and SR narration.

### 7.6 Reduced-motion mode

Trigger: `prefers-reduced-motion: reduce` OR settings opt-in. Settings can force full motion even if the OS prefers reduce (user choice).

This is a **second rig**, not `animation: none`.

- Idle: cross-fade between 2–3 still poses of the same family, 2.5–4s fades.
- Flight: cross-fade at old perch → new perch (hold 400ms empty, or a static mid-air pose at 40% opacity — designer call; prefer hold-empty to avoid a blinking bird).
- No leaf/feather drift.
- Day/evening color still shifts, slower (2× duration).
- Calls and captions unchanged.
- Drift, mood, notebook unchanged.

Ship this in v1, same release as full motion.

### 7.7 Listen-in visual

No spotlight, no nameplate, no dimming of other birds to zero. Allowed: a slightly stronger focus ring (same as keyboard focus) and the audio mix. If we over-mark the selected bird, listen-in becomes “select.”

### 7.8 Color

Calm naturalist palette. Implementation holds tokens in `design/palette.ts` (even if the visual designer later replaces hex). All chrome text tokens must pass WCAG AA on both day and night top-bar backgrounds. Captions sit on a 60% dark or light wash that itself meets AA against the caption text.

No saturated accent reds/electric blues.

### 7.9 Frame loop

`requestAnimationFrame`. When `document.hidden`, cancel the loop. Target 60fps idle on a 5-year-old laptop: budget

- 4ms scene update
- 6ms draw
- audio is on the audio thread

Bird count ≤ 7 keeps this honest. No per-frame GC: preallocate ornament pool (max 8 leaves).

---

## 8. Audio pipeline

### 8.1 Why procedural, operationally

- Bundle: we cannot ship enough recorded variation under 2MB gz.
- Chorus: two loops phase-cancel; two independently scheduled syntheses do not.
- Recognizability: identity is a seed, not a sample.

There is **no** `public/calls/*.mp3` in the repo. CI fails if one is added.

### 8.2 Motif library

Per species, 5–8 motifs. A motif is data:

```ts
type Motif = {
  id: string
  notes: { ratio: number; durMs: number; vel: number; ornament?: 'grace' | 'trill' }[]
  restMs: number
  moodBias: Partial<Record<Mood, number>>
}
```

Plus a timbre patch: 1–2 oscillators (sine/triangle), a narrow noise burst for consonants, two formant peaking filters, a simple AHR envelope. Nightjar: more noise, lower f0.

`signatureSeed` picks:

- base f0 within the species range (±2 semitones, fixed for that bird’s life)
- motif weights
- vibrato depth
- formant offset

Mood picks motif subset and tempo (±8%). Vocal frequency does **not** retune identity; it retimes the scheduler.

### 8.3 Synth graph (per voice, pooled)

```
oscA, oscB, noise → gainEnv → formant1 → formant2 → birdGain → listenInGain → master → destination
```

Pool of 7 voices. Start/stop with `AudioParam` ramps; do not create/destroy nodes per call.

### 8.4 Chorus mixing

Each bird has `birdGain`. Ambient: birds slightly different levels by depth (back quieter). Master bus gentle limiter.

When two calls overlap, they are just two voices. No sidechain ducking except listen-in.

### 8.5 Listen-in mix

On engage (click / tap / Enter on focused bird):

- Focused `listenInGain` → 1.0 over **1400ms**
- Others → 0.28 over **1400ms**
- Never 0. Back birds can go to 0.18, not silence.

On disengage (second click, other bird, empty scene click, Escape, focus leaving the scene): reverse, same 1400ms.

This must feel like leaning in, not like soloing a track.

Client emits `listen_in_start` / `listen_in_end` for drift. Mix itself is local and immediate.

### 8.6 Captions

Generated from the motif instance actually scheduled:

```
notes.length === 1 && sharp  → "a single sharp call from the back perch"
three rising ratios          → "a soft three-note rise"
repeated low trill           → "a low trill, paused, low trill again"
```

A small classifier over the instantiated motif (note count, interval direction, zone word) produces the string. Not a fixed map from `motif.id` only — two plays of the same motif with a dropped grace note should caption differently if they sound different.

Position: small text near the bird, fade with the call envelope. Naturalist voice. AA contrast wash.

Default off. On if user opts in **or** WebAudio is missing / context denied.

### 8.7 WebAudio fallback

If `window.AudioContext` is absent, or `resume()` fails after a gesture, or construction throws:

- Stay silent.
- Force captions on for the session (and persist the preference only if the user doesn’t turn them off).
- Do not fetch recordings.

### 8.8 Unlock and background

First pointer/key: `context.resume()`. Hidden: `suspend()`. Visible: wait for a gesture if the browser requires it; until then, captions if the user had audio before — **Call:** do not auto-enable captions on every suspend; only on hard unavailability. Brief tab switches should not suddenly cover the scene in words.

### 8.9 Uncanniness controls

- Minimum motif variation: ±12 cents, ±6% duration, 15% chance to drop/add a grace note.
- Forbid scheduling the exact same motif id + tempo twice in a row for one bird.
- Humanize chorus: no global quantized grid.
- Listening test gate (see §11): 5-minute chorus with 7 birds must not be identifiable as a loop by the internal listening panel.

---

## 9. Accessibility surfaces

Accessibility ships on day one as the same product in another register, not a bolt-on state dump.

### 9.1 Screen-reader narration

A live region (`aria-live="polite"`) receives composed prose, not DOM soup of seven birds.

Cadence:

- Idle: one paragraph every **45s** (range 30–60).
- Immediate (but still observational): session greeting, offer reaction, settle, arrival fly-in.
- Queue: max 2 pending. Drop idle updates if a priority observation is in flight. Never dump.

Example idle:

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Same `packages/prose` voice as the notebook. No “Pip mood content.” Names may appear when they aid recognition (“pip is on the front rail”) — specificity is charm — but not as a roster every cycle.

Implementation: a visually hidden region plus, in settings, an optional visible transcript panel for users who want both (not on by default; would add chrome).

Birds in the scene are in the tab order as a **flat list** after the top bar. Each bird’s accessible name is the user-assigned name; description is a short static species clause, not a live mood stat. Live changes go through the narration region so we don’t spam SR on every pose frame.

### 9.2 Keyboard

| Key | Action |
|---|---|
| Tab | Top bar icons, then first bird |
| Arrow L/R | Adjacent birds (spatial: left-to-right by current x) |
| Enter / Space | Listen-in toggle on focused bird |
| Escape | End listen-in; close offer/notebook sheets |
| Offer shortcut | Focus offer icon (documented in a11y settings) |

Focus ring: 2px soft outline, tokenized to pass contrast on dawn and night. Visible in reduced-motion too.

Offer sheet and settle are fully keyboarded. Naming a bird is a standard input in a sheet, matter-of-fact label “Name” with naturalist helper text.

### 9.3 Captions

§8.6. Also useful for SR users who keep audio on; do not duplicate a caption into the live region if a call was already described in the last narration sentence.

### 9.4 Contrast and motion

AA on all user copy. Scene itself is exempt except captions and focus ring.

Reduced-motion: §7.6. Respect OS, allow override in Accessibility settings (matter-of-fact copy).

### 9.5 Settings split (voice)

Accessibility and account settings: matter-of-fact.

Aviary, notebook, offer prompts, captions, narration: naturalist.

### 9.6 Visit + a11y

Visitor gets the same narration and reduced-motion and captions. No interactive listen-in. SR still describes the scene.

---

## 10. Performance budgets and observability

### 10.1 Budgets (CI + synthetic)

| Budget | Gate |
|---|---|
| Initial JS at first paint | **< 2MB gzipped** (hard). Internal target: critical ≤ 80KB gz, total first-load JS ≤ 350KB gz |
| Time to first bird | **< 500ms** on mid-tier mobile / 4G synthetic (Moto G / similar, throttled) |
| Idle fps | **60fps** median, 55fps p5, 30-minute session, 7 birds, 5-year-old mid laptop profile |
| Memory | Heap after 30 min ≤ heap at 2 min + 8MB slack; no monotonic growth. Audio buffers pooled |
| Tick live p99 | Alarm at **5s**; SLO **50ms** |
| Snapshot payload | **< 8KB** gz typical |
| Magic-link email | p95 issue < 2s |

### 10.2 How we hit TTFB / first bird

- Edge-cached shell HTML.
- Critical renderer split; no Preact on the first-bird path.
- Inline snapshot for warm cookies (worker fetch from Redis, 20ms budget; if miss, quiet field → one XHR).
- SVG sprites in the critical chunk, not a late image sprite atlas.
- No webfont on the scene; system UI for chrome, one small licensed text face for captions/notebook loaded async.
- No analytics SDK.

### 10.3 What we measure

Synthetic fleet (hourly, 4 geos): load, first-bird mark (`performance.mark('first-bird')` when the first bird draw call succeeds), fps sample, audio context errors, snapshot latency.

RUM, **aggregate only**:

- navigation timing, `first-bird`, long tasks
- fps histogram
- audio-context error counts
- API latency, 4xx/5xx
- session duration histogram **without account id**

### 10.4 What we deliberately do not measure

- Per-bird interaction counts in any warehouse.
- Presence-time per account in analytics.
- Drift distributions across the population.
- “Most listened-to species.”
- Visit funnels that could become leaderboards.
- Rage-clicks, heatmaps, session replay (replay would capture the relationship).

If a dashboard would let someone reconstruct how a person is with their birds, it does not exist.

### 10.5 Client marks

```
quiet-field-paint
snapshot-applied
first-bird
audio-ready
keepalive-rtt
```

These can go to RUM without bird ids.

### 10.6 Memory / audio hygiene

- Ornament pool, voice pool.
- Notebook virtualized list; detached entry nodes released.
- No `setInterval` audio schedulers that accumulate; use `currentTime` lookahead (120ms) on a single clock.
- CI: headless 30-minute soak with 7 birds, assert heap.

### 10.7 Browser support

Last two Chrome / Safari / Firefox / Edge. Others: matter-of-fact unsupported page. No polyfill soup that blows the bundle.

---

## 11. Rollout

### 11.1 Build sequence (teams can parallelize after the contract)

**Milestone A — contract and skeleton (week 1–2)**

- `packages/schema`, `packages/prose` stub, Postgres migrations, synthetic UUID + encrypted email.
- Magic-link auth, empty aviary quiet field, account delete/export stubs.
- Snapshot DTO frozen.

**Milestone B — engine without charm (week 2–5)**

- Tick function + catch-up + event log + presence intervals.
- Drift calibration harness (see tests).
- Mood, perch, weather, circadian, EMA quietness.
- Two starter birds, age-gate flags (can be time-accelerated in staging).

**Milestone C — visible aviary (week 3–7, parallel)**

- Canvas scene, three zones, idle families, fly-in once, responsive fit.
- Day/night palette, settle + 5s undo, top-bar fade.
- Reduced-motion rig in the same milestone, not later.

**Milestone D — audio (week 5–8)**

- Motif libraries for 6 species, voice pool, chorus, listen-in ramps, captions, silent fallback.

**Milestone E — session verbs (week 6–8)**

- Greeting planner, offers + cooldown, notebook composer + UI, presence monitor (the real conjunction).

**Milestone F — social + a11y + accounts polish (week 7–10)**

- Visits, visit log, revoke, narration live region, keyboard, settings voice split.

**Milestone G — perf, privacy audit, listening panel, closed beta (week 9–12)**

- Budgets in CI, telemetry boundary review, 3-week drift staging with staff aviaries.

### 11.2 Birds-per-aviary ramp

Production age gates stay as in §6.9. Staging clocks can accelerate 20× so QA can see bird 3–7.

Do **not** raise the 7 cap in v1. Do not A/B a higher cap. Recognizability is not a growth lever.

Staff flag: `max_birds` only in non-prod.

### 11.3 Ship shape

- Closed staff aviaries first (internal emails).
- Quiet beta: magic-link allowlist. No public discovery, no press kit that promises notifications.
- GA: same binary. No “engagement” experiment layer.

### 11.4 Instrumentation from day one

Ship with:

- Synthetic first-bird + fps + tick latency.
- Auth issue/consume rates (no email in logs).
- Event ingest counts **by type**, not by account.
- Privacy CI: grep warehouse schemas for `bird_id`, `boldness`, `presence_seconds`.
- Product CI: grep the client for `Welcome back`, `streak`, `achievement`, `days in a row`, `birds adopted`.

### 11.5 Drift calibration loop

A hidden staging harness (not a user-facing debug panel) replays:

- Regular: 4×20 min/week × 3 weeks.
- Heavy: 7×45 min + lots of listen-in.
- Absent: 0 presence × 14 days then return.

Assert numeric bands. Staff *look* at week-3 vs week-0 recordings for perch/greeting/plumage. If staff can see a difference on day 2, α is too high. If they cannot see a difference at day 21, α is too low.

There is no in-product “show vectors” for this. Staging admin only, behind VPN, matter-of-fact, never shipped to prod client bundles.

---

## 12. Risks

### 12.1 Drift calibration (highest product risk)

**Failure modes:** too fast (Tamagotchi / session-to-session “Pip leveled up”); too slow (screensaver); accidental negative drift from a well-meaning “mean reversion”; presence definition too loose so the whole population races.

**Mitigations:** versioned `drift_v1` constants; daily Δ cap; EMA separate from traits; presence conjunction tests; no client-authored vectors; calibration harness in CI; refuse any dashboard of population-average drift (it would tempt us to “tune for engagement”).

### 12.2 Sync correctness

**Failure modes:** two tabs writing absolute state; cache serving a pre-tick snapshot after an offer; visitor events leaking into `event_log`; catch-up that snaps mood to circadian prior on first tick.

**Mitigations:** single writer; Redis invalidated in the same transaction commit hook; visitor routes cannot insert drift-eligible types (DB constraint on `event_log.type` + API guard); catch-up tests that a drowsy dusk bird is not `content` at the first 60s of night.

### 12.3 Audio uncanniness

**Failure modes:** identifiable loops; chorus that sounds like ringtones; seven birds becoming mush; nightjar that reads as a soundboard joke.

**Mitigations:** motif variation rules; 7-bird listening panel as a release gate; cap remains 7; species motif ranges kept disjoint enough to recognize; no recorded fallback that would be *more* canned under stress.

### 12.4 Accessibility regressions

**Failure modes:** live region spam; reduced-motion as `display:none` animations; captions that say “call type 3”; focus ring invisible at night; shipping a11y in v1.1.

**Mitigations:** reduced-motion rig in milestone C; narration cadence tests; caption classifier review; contrast tokens in CI (axe + pixel contrast on captions); keyboard e2e for listen-in / offer / settle.

### 12.5 First-frame aliveness

**Failure modes:** spinner, logo, fade-from-grey, all birds in pose 0, greeting chorus in unison, “Welcome back.”

**Mitigations:** quiet-field boot; `phase01` from server; greeting stagger; lint for welcome strings; visual snapshot tests of first paint.

### 12.6 Presence honesty

**Failure modes:** counting `visibilityState` only; mobile Safari focus quirks; activity window too short so still watchers drop out; service worker keeping a “visible” lie.

**Mitigations:** all three signals; 180s window; integration tests per browser on the beta matrix; never take presence from the server clock or from “session open.”

### 12.7 Privacy leakage

**Failure modes:** email as a partition key; per-bird events in Sentry extras; export link logged; session replay vendor “just for a week.”

**Mitigations:** email HMAC lookup only; Sentry scrubbers that drop `payload` on events; no replay vendors; warehouse CI deny-list.

### 12.8 Notebook / narration quality

**Failure modes:** “session started at 7:43”; LLM drift or third-party send; one entry per session; entries about the user’s habits.

**Mitigations:** closed fact set + templates; sparsity gate; copy review as a release artifact; unit tests that reject bodies matching `/you visited|every day|streak|vocal frequency/`.

### 12.9 Scope creep (the predictable kind)

Someone will propose a streak, a public aviary, a hunger meter, a native wrapper, a recorded-audio pack, a “just this once” welcome toast, a stats drawer “for power users.”

Response is the non-goals file. Engineering’s job is to make those features *expensive to sneak in*: no hooks, no unused `score` columns, no toast component in the design system.

---

## 13. Testing strategy (so the next team knows what “done” means)

### 13.1 Engine (Node)

- Golden-file tick replays.
- Drift bands for the three personas.
- Monotonic traits.
- Visitor isolation.
- Idempotent event ingest.
- Magic-link consume-once.
- Soft delete recover / hard purge.

### 13.2 Client

- Presence conjunction matrix.
- Listen-in gain ramps (audio param values at t=0, 0.7s, 1.4s).
- Reduced-motion path selection.
- First paint: no spinner selector exists.
- Keyboard map e2e.
- Bundle size report on every PR.

### 13.3 Prose

- Snapshot tests for notebook + narration + captions.
- Forbidden-phrase linter.

### 13.4 Perf

- 30-minute soak.
- Synthetic 4G first-bird.

### 13.5 Privacy

- Static check: simulation ORM not imported from the metrics package.
- Log fixture: no `@` email strings.

---

## 14. Implementation notes the PRD does not say but the build needs

1. **No toast component.** If a library brings one, do not wrap it.
2. **No game loop library** (Phaser, Pixi, Three). Custom canvas keeps the budget and avoids stock loaders.
3. **Preact only for chrome.** Scene stays framework-free so first-bird JS stays small.
4. **Names:** 1–24 graphemes, no empty; rename does not touch traits.
5. **Song-fragment library:** 5–7 short motifs, distinct from bird grammars, played as a quiet extra voice. Birds respond via the offer table; we do not pitch-match in v1 beyond “join / quiet / against.”
6. **Still pool:** a translucent ellipse in the front plane, 45–75s lifetime, then fade. Not a permanent furniture unlock.
7. **Seed:** a small object near the chosen bird; removed after the reaction or 20s.
8. **Contact** for error copy: a single `hello@` support address on the matter-of-fact error; no chatbot.
9. **Legal:** privacy policy in account settings, plain text, names operational telemetry categories and explicitly excludes per-bird interaction state.
10. **i18n:** v1 English only; prose engine is English. Do not interpolate naturalist strings from a generic i18n framework in a way that title-cases them.

---

## 15. What success looks like

A user opens the tab. Within a second a bird notices them — not a banner. They sit. Presence ticks. They listen in on Pip; the others recede but do not vanish. They leave without settling; nothing scolds them. Two weeks later the aviary is quieter, not angry. Three weeks later they realize Pip is closer to the glass than she used to be, and the notebook says she greeted first, and nobody congratulates them.

If the implementation makes that story true — technically, in the tick, in the mix, in the first frame — v1 is done. Everything else is refusal.
