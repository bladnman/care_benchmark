# Pocket Aviary — v1 Implementation Plan

This is an executable engineering plan for Pocket Aviary v1: a browser-based virtual aviary of 2–7 procedurally-animated birds whose hidden personalities drift over weeks in response to the user's presence. It is written so a separate frontier engineering team can build the product without further clarification. It interprets the PRD into systems, contracts, data shapes, and sequencing; it does not restate the PRD and it does not implement the product.

The plan is organized around one load-bearing architectural fact that every section inherits from: **the server is the only writer of canonical aviary state, the simulation advances on a slow server-side tick whether or not a client is connected, and clients are render-only consumers of snapshots plus append-only event producers.** Everything downstream — sync correctness, drift integrity, multi-device coherence, the "feels alive without the viewer" conceit — falls out of that single rule, and most of the risks in this plan are failures to honor it.

A second fact shapes the affective layer: **the product earns attention by being noticed, never by announcing.** That is not a copy guideline; it is an architectural constraint that forbids a class of UI (toasts, banners, streak counters, level-ups, friend-visited pings) at the framework level so it cannot leak in feature-by-feature. The plan names where that constraint lives in code so it survives well-meaning contributors.

---

## 1. Scope

### 1.1 In scope for v1

**Core relationship engine**
- Hidden per-bird personality vectors (boldness, social warmth, vocal frequency, plumage saturation, curiosity) — slow timescale, server-persisted, never exposed numerically anywhere.
- Monotonic-toward-expressive drift driven primarily by presence-time, secondarily by interactions.
- Fast-timescale mood (enumerated states) persisted across sessions and advanced by the server tick.
- Server-side simulation tick (~once/minute, calibrated) that consumes the interaction event log, drifts personality, transitions mood, advances day/night and weather, and writes canonical state.
- Procedural call grammar: per-species motif libraries, personality-shaped timing/pitch, synthesized client-side via WebAudio. Real-time chorus, no recorded audio, no loops.
- Bird-to-bird interaction (call/response, mood contagion, emergent chorus).
- Stable internal bird identity invariant across rename, sync, and any future species-pool change.

**Account & sync**
- Single-user accounts, one canonical aviary per account.
- Magic-link email auth (15-min link expiry, single-use, per-email rate limit), per-device revocable session tokens, email-change with new-address verification.
- Synthetic UUID as the only identifier used anywhere except the single encrypted email field.
- Multi-device sync as an emergent property of server-canonical state (no client-to-client sync).
- Additive, server-authored personality deltas processed in event-log order (no last-write-wins on personality).
- Account export (on-demand JSON, emailed download link), soft-delete (30 days) → hard-delete.

**Session surface**
- Single horizontal aviary scene, three perch zones (front/middle/back), no pan/scroll/zoom, responsive without ever cropping a bird.
- Loads with motion already in progress (no entry animation, no spinner); quiet-field loading state and empty-aviary state.
- Local-time day/night cycle; rare, non-assertive ambient weather; ambient leaf/feather drift (client-only ornaments).
- Return-greeting (one bird, varied by boldness/mood/absence-length, staggered when multiple, procedurally non-repeating).
- Listen-in (gradual mix re-balance, others quiet to ambient but never silent).
- Offer (seed / song-fragment / still-pool, reaction shaped by mood + curiosity, per-bird few-minute cooldown).
- Settle (soft evening session-end gesture with 5-second undo; equivalent to tab-close at the engine level).
- Field notebook (auto-generated naturalist prose, sparse ~once/few-days, read-only, infinite scrollback).
- Adoption flow: two system-selected starter birds, user-assigned names, renameable any time.
- Third-bird-and-beyond offers paced by **aviary age**, capped at 7.

**Accessibility (first-class, ships with v1)**
- Screen-reader running naturalist narration (slow cadence, prioritized on user-initiated events).
- Reduced-motion mode as a designed cross-fade rendering, not animations-off.
- Call captioning generated from the live call grammar.
- WCAG AA contrast floor on all user copy; full keyboard navigation with visible focus.

**Social (one quiet affordance)**
- Per-invite, opt-in, revocable, read-only ambient visits (email one-time link, 30-day unused-invite expiry).
- Silent visit logging; opt-in (off by default) visit notifications; visit log in settings.
- Visitor sessions never record presence or events and never drift the host's birds.

**Performance & observability (v1 hard targets)**
- Initial JS bundle < 2MB gzipped at first paint.
- Time-to-first-bird < 500ms on mid-tier mobile over 4G.
- 60fps idle motion on a 5-year-old mid-range laptop across a 30-minute session.
- No client memory growth over 30 minutes (enforced in CI).
- Aggregate-only telemetry; per-bird/per-account interaction state never enters telemetry, analytics, or training.

### 1.2 Explicitly out of scope (non-goals enforced, not just noted)

Native apps; gamification of every flavor (achievements, streaks, levels, scores, badges, "birds adopted: N", green-dot calendars, XP, ranks, milestone celebrations); Tamagotchi mechanics (death, hunger, distress, decaying happiness meters, negative drift on neglect); social-network surfaces (profiles, follows, public feed, discovery, leaderboards, comments, co-presence, friend-of-friend, mutual visits, show-off rendering); push/email notifications about the aviary; payments; shared/multi-profile aviaries; customizable/multi-aviary scenes; user-controlled bird placement; any numeric exposure of personality.

These are not "later" — several are **architecturally foreclosed** (see §13.2) so they are harder to add later, not easier.

### 1.3 Defensible calls made where the PRD left room

- **Presence activity window = 3 minutes** (PRD says "a few minutes, lean longer"). Watching without moving is the product, so we lean long; revisited via the drift-calibration harness (§5.6).
- **Tick cadence = 60s.** Snapshot keepalive while visible = 30s. These are config constants (§5.1), tuned against the p99-5s tick-latency error budget.
- **Mood set (final): `wary, content, curious, drowsy, alert, settled`.** PRD lists the first five as examples plus a `settled` night/post-settle state. We finalize six.
- **Drift becomes user-visible at ~3 weeks, instrument-measurable at ~1 week** — encoded as explicit calibration assertions, not vibes (§5.6).
- **Bird-age third-bird cadence (initial, tunable):** offer #3 at aviary age ~8 weeks, then a new offer roughly every ~10–12 weeks, capped at 7. Driven by `aviary.created_at`, never by visit/interaction counts.

---

## 2. Architecture

### 2.1 Service shape

Five backend services plus a static client, behind an edge/CDN. Service boundaries are drawn around the **privacy and write-ownership rules**, not around convenience.

1. **Edge / BFF (Backend-for-Frontend).** Terminates TLS, serves the static client and the HTML+inline-snapshot document (critical for time-to-first-bird, §9), proxies API calls, holds no business logic. Co-located with CDN edge for snapshot delivery.
2. **Auth service.** Magic-link issuance/consumption, session-token issuance/revocation, email-change verification. Owns the one encrypted email field. Issues the synthetic account UUID at creation.
3. **Aviary API service (stateless).** Read path (state snapshots) and write path (append-only event log). The *only* externally reachable door to simulation state. Never writes personality directly; only appends events and reads canonical snapshots.
4. **Simulation service (the tick).** A scheduled, horizontally-shardable worker (sharded by account UUID) that owns canonical aviary state. **The sole writer of personality vectors, mood, perch, and notebook entries.** Reads the event log, applies additive deltas in log order, writes canonical state. No client ever reaches it.
5. **Social/visit service.** Invite issuance, one-time visit-link consumption, revocation, visit logging, visit notifications (opt-in). Visitor sessions are routed through a read-only snapshot path that is gated to *never* append events.

A separate **notebook generator** runs inside the simulation service (it has the state and the cadence) rather than as its own service — entries are a function of canonical state transitions and must share the tick's ordering guarantees.

### 2.2 Client/server split — the bright line

```
            writes interaction events  ──►  append-only event log  ──►  [SIM TICK reads]
 CLIENT  ──┤                                                                    │
 (render) └──  pulls state snapshots   ◄──  canonical aviary state  ◄──  [SIM TICK writes]
```

- The client **renders snapshots and interpolates**; it never computes personality, never ticks, never owns canonical state.
- The client **emits events** (offer, listen-in start/end, settle, presence pings, greeting-shown) to an append-only log.
- The simulation tick is the **only** code path that mutates personality/mood/perch/notebook.
- Visit sessions get snapshots through a read-only door with event-append physically disabled.

This split is the implementation of "no last-write-wins" and "the aviary continues without the viewer." It is enforced by giving the Aviary API service no write credential to personality columns at all — the database role used by the read/write API can `INSERT` into the event log and `SELECT` snapshots, but has no `UPDATE`/`INSERT` grant on personality/mood tables. The simulation service uses a distinct role that *can* write those, and *cannot* be reached from the client network path. The constraint is enforced at the DB-grant layer so a future code change can't quietly violate it.

### 2.3 Render pipeline boundary

The render boundary is: **simulation produces a discrete snapshot at perch/mood/call-intent granularity; the client owns all continuous motion, audio synthesis, and ornamentation.** Concretely:

- Server sends *intents and states* ("Pip: front perch, mood content, next call window opens at T, call signature seed S"), never frames, never audio, never per-leaf data.
- Client turns intents into continuous motion (interpolated flight paths, idle micro-motion, audio synthesis, leaf/feather drift). Leaves and feathers have **no server state** — they are pure client ornaments at idle cadence.

This boundary is what keeps snapshots in the kilobytes (§7.4) and what lets the same canonical state drive the visual surface, the reduced-motion surface, and the screen-reader narration from one source of truth (§8).

### 2.4 Technology choices (recommended, with rationale)

- **Client:** TypeScript. Rendering on a single `<canvas>` (2D) or WebGL via a thin scene layer — recommend **2D canvas with a sprite/SVG-pose atlas** as the default because the scene is one plane with subtle parallax, not a 3D world, and 2D canvas hits the 60fps/no-growth budget more predictably on 5-year-old hardware. Audio via **WebAudio API** with an `AudioWorklet` for synthesis. State/UI chrome in a lightweight framework (Preact or Svelte) to protect the bundle budget; the aviary scene itself is **not** a component tree — it is an imperative render loop.
- **Backend:** language is team's choice; the plan assumes a typed backend (TypeScript/Go/Rust all fine). Requirements that matter: cheap horizontal sharding of the tick by account UUID, and strong ordered processing of the per-account event log.
- **Datastore:** a primary transactional store (Postgres) for accounts, birds, personality vectors, mood, notebook, sessions, invites; the append-only event log can be a partitioned table keyed by account UUID (Postgres) or a log store — **recommend a partitioned Postgres table** at v1 scale to keep ordering and retention simple, revisit if tick throughput demands a dedicated log.
- **Scheduler:** the tick is a sharded scheduled worker; recommend a leased-shard model (each shard worker holds a lease on a range of account UUIDs) so ticks are exactly-once per account per tick window.
- **Telemetry:** a metrics pipeline (counts/latencies/histograms) that is **physically separate** from the simulation database (§11). No shared connection, no shared reader.

---

## 3. Data model

All identifiers are synthetic UUIDs. Email appears exactly once, encrypted, on the account record. Personality vector values never leave the server in numeric form.

### 3.1 Account

```
account
  id                UUID  (synthetic, primary key, the ONLY id used everywhere else)
  email_encrypted   bytes (encrypted at rest; the single PII location)
  email_verified    bool
  created_at        timestamptz
  status            enum { active, pending_deletion, deleted }
  deletion_marked_at timestamptz null   (soft-delete window start)
  settings          jsonb  (reduced_motion_pref, captions_on, audio_on,
                            visit_notifications_on=false, ...)
```

`pending_deletion` accounts still sign in and can recover during the 30-day window (§6.4). A nightly job hard-deletes accounts past the window (cascades to all aviary/bird/event/notebook/visit rows).

### 3.2 Aviary

```
aviary
  id                UUID
  account_id        UUID -> account.id   (1:1 at v1)
  created_at        timestamptz          (drives third-bird-and-beyond pacing — AGE, not counts)
  canonical_state_version  bigint        (monotonic; bumped each tick that writes)
  day_phase         enum { dawn, day, dusk, night }  (derived from user local tz at tick)
  weather           enum { clear, rain, wind } + weather_until timestamptz
  last_tick_at      timestamptz
```

Day-phase is recomputed each tick from the account's local timezone. The aviary stores the user's IANA timezone (captured at sign-in, updatable) so the tick can advance day/night without a client connected.

### 3.3 Bird

```
bird
  id                UUID   (STABLE internal identity — never reassigned, survives rename/sync/migration)
  aviary_id         UUID
  species_id        ref species_pool
  display_name      text   (user-assigned; renaming never touches id/personality/mood/call)
  adopted_at        timestamptz
  -- personality vector (slow timescale; server-only; NEVER serialized to any client numerically)
  trait_boldness         real
  trait_social_warmth    real
  trait_vocal_frequency  real
  trait_plumage_saturation real
  trait_curiosity        real
  -- fast timescale
  mood              enum { wary, content, curious, drowsy, alert, settled }
  mood_entered_at   timestamptz
  perch_zone        enum { front, middle, back }
  -- call runtime seed (varies output; recognizable signature preserved)
  call_grammar_seed bigint
```

Traits are normalized to a small bounded range (e.g. `[0,1]`), seeded per-species with small randomization at adoption. **Persistence rule:** traits are read-modify-written only by the tick, only as additive deltas, never recomputed from event history, never rebuilt by the client. Losing a vector = deleting the bird the user knows (§13.1 makes this a backup/restore invariant).

### 3.4 Personality is never exposed numerically — enforced in the type system

There are two bird DTOs and they are different types, not the same type with fields omitted:

- `BirdCanonical` (server-internal): includes trait floats. Lives only in the simulation service and the DB.
- `BirdSnapshot` (wire format to client): includes id, name, species, mood, perch, call-intent, animation-state. **Has no trait fields and no field from which a trait value could be reconstructed.** There is no debug flag, no admin view, no tier that adds them.

A serialization test asserts `BirdSnapshot` JSON contains none of the trait keys, run in CI, so the rule can't regress.

### 3.5 Interaction event log (append-only)

```
event
  id            UUID
  account_id    UUID   (partition key)
  bird_id       UUID null   (null for aviary-wide events)
  type          enum { presence_ping, listen_in_start, listen_in_end,
                       offer_seed, offer_song, offer_pool, settle, greeting_shown }
  payload       jsonb (e.g. presence duration window, offer target, song motif id)
  client_ts     timestamptz   (advisory)
  server_ts     timestamptz   (authoritative ordering)
  processed_by_tick_at timestamptz null
```

Append-only. Clients can only `INSERT`. The tick consumes unprocessed events **in `server_ts` order per account**, computes deltas, and stamps `processed_by_tick_at`. This ordered, additive consumption is the whole defense against the last-write-wins failure (§6.3). Event log is retained per privacy policy and purged on hard-delete.

### 3.6 Presence accounting record

Presence is not a single event; it is reconstructed by the tick from `presence_ping` events. The client emits a `presence_ping` only while the **conjunction** holds: `visibilityState === 'visible'` AND `document.hasFocus()` AND a pointermove/keypress occurred within the activity window (3 min default). Pings carry the covered interval; the tick sums non-overlapping covered intervals into presence-time. (Detailed in §5.3.)

### 3.7 Notebook entry

```
notebook_entry
  id            UUID
  aviary_id     UUID
  created_at    timestamptz
  prose         text   (naturalist, lowercase, present-tense, specific — generated, never templated to gamification)
  source_signal jsonb  (internal: which state transition prompted it — NOT shown to user, used for sparsity dedup)
```

Read-only to the user. `source_signal` is internal only and is one of the **observations-of-the-aviary** kinds — never an observation of the user's behavior (no "you visited every day"; see §5.7 generation rules).

### 3.8 Session & auth

```
magic_link    { token_hash, account_id, issued_at, expires_at (issued+15m), consumed_at }
session       { id UUID, account_id, device_label, created_at, last_seen_at, revoked_at null }
email_change  { account_id, new_email_encrypted, verify_token_hash, expires_at, committed_at null }
```

### 3.9 Visit / invite

```
invite
  id            UUID
  host_account_id UUID
  visitor_email_encrypted bytes
  token_hash    text   (one-time visit link)
  created_at    timestamptz
  expires_at    timestamptz   (created + 30d if unused)
  revoked_at    timestamptz null
  consumed_first_at timestamptz null
visit_log
  id, host_account_id, invite_id, visitor_email_encrypted (for host display), started_at, approx_duration_s
```

Visitor sessions are read-only: the visit door issues a snapshot stream bound to the host aviary and **has no event-append capability**. Revoking flips `revoked_at`; the next snapshot pull on the visitor returns the matter-of-fact "visit no longer available" surface.

### 3.10 Species pool

```
species   (≈6 entries, content-defined, shipped with build)
  id, silhouette_asset, default_plumage_palette, call_motif_library_id, behavior_profile
```

One species has a nightjar-like signature that stays active at night. Species rarity is not modeled. New birds draw from the same pool.

---

## 4. API surface

REST/JSON over HTTPS for control-plane and event writes; a lightweight pull model for snapshots (no client tick, so no realtime socket is *required* — see §6.5 for the keepalive/visibility-change pull triggers). All authenticated routes require a session token; the synthetic account UUID is resolved server-side from the token, never sent by the client.

### 4.1 Auth

- `POST /auth/magic-link` `{ email }` → 202. Always 202 regardless of whether the email exists (no account enumeration). Rate-limited per email.
- `GET /auth/consume?token=…` → sets session, redirects into aviary; single-use, 15-min expiry. On failure renders matter-of-fact surface: *"We couldn't sign you in. The link may have expired. Try requesting a new link."*
- `POST /auth/email-change` `{ new_email }` → sends verification to new address; old email works until verify.
- `GET /auth/email-change/verify?token=…` → commits switch.
- `GET /account/sessions` → list of devices; `POST /account/sessions/{id}/revoke`.

### 4.2 State read (the snapshot)

- `GET /aviary/snapshot` → `AviarySnapshot`:

```jsonc
{
  "version": 184123,                 // canonical_state_version, monotonic
  "server_time": "…Z",
  "day_phase": "day",                // derived from user local tz
  "weather": { "kind": "rain", "until": "…Z" },
  "birds": [
    {
      "id": "uuid",                  // stable identity
      "name": "pip",
      "species": "warbler",
      "mood": "content",             // enum only — no numeric trait anywhere
      "perch": "front",
      "call": { "seed": 99213, "next_window": "…Z", "signature_id": "warbler-A" },
      "anim": { "state": "preening", "since": "…Z" },
      "greeting": null               // or a greeting directive on session start (see 4.3)
    }
  ]
}
```

`AviarySnapshot` is the `BirdSnapshot` wire type (§3.4): **no trait values, ever.** Payload is kilobytes. Delivered inline in the initial HTML document from the CDN edge for the first paint (§9), then pulled.

- Pull triggers (client): on `visibilitychange`→visible, on a detected long render-frame gap (laptop resume), and on a 30s keepalive while visible. (§6.5)

### 4.3 Return-greeting directive

The greeting is **selected server-side** (it depends on boldness/mood/absence-length, which the client must never see numerically). On the first snapshot of a session the server includes, on exactly one bird, a `greeting` directive:

```jsonc
"greeting": { "form": "step-forward-and-call", "stagger_ms": 0, "variation_seed": 71113 }
```

- Server selects *which* bird greets (honoring boldness/mood) and the greeting *form* (varied by absence length: glance for short absence, re-orientation/approach for long).
- When more than one bird would greet, the server assigns ascending `stagger_ms` offsets so they never fire in unison.
- `variation_seed` lets the client render real procedural variation (not 1-of-3 canned variants).
- Absence length is computed server-side from `session.last_seen_at` / last presence; the client never receives a "you've been gone X days" value and there is no surface that renders one.

### 4.4 Event write (append-only)

- `POST /aviary/events` `{ events: [ {type, bird_id?, payload, client_ts} ] }` → 202.
  - Accepts batches (presence pings are batched to keep request count low).
  - Server stamps `server_ts`, appends, returns nothing about personality. Idempotency key per event to dedupe magic-link replay / double-send.
  - **There is no endpoint that sets personality, mood, or perch.** Mutation of canonical state is exclusively the tick's job. This absence is the API-level enforcement of §2.2.

### 4.5 Interactions map to events (not to state writes)

| User action | Event(s) emitted | Server effect (via tick only) |
|---|---|---|
| Sit and watch | `presence_ping` (while conjunction holds) | presence-time → dominant drift |
| Listen in on a bird | `listen_in_start` / `listen_in_end` | that bird's social-warmth + vocal-freq drift; **audio mix is client-side**, server only records the attention signal |
| Offer seed/song/pool | `offer_*` (rejected client-side if within per-bird cooldown) | small curiosity drift on accept, small boldness drift for offering-near; mood nudge |
| Settle | `settle` | ends presence window cleanly; small mood-quieting; **no drift direction** |
| Tab close | (no event needed) | end of presence pings = end of window; treated identically to settle at engine level |

Listen-in mix change and the settle lighting shift are **client-rendered immediately** for responsiveness; the event is the durable record the tick reads. This keeps interactions feeling instant while preserving server-authoritative drift.

### 4.6 Notebook

- `GET /notebook?before=cursor&limit=n` → page of entries, newest first, infinite scrollback. Read-only; no create/edit/delete routes exist.

### 4.7 Offers / library / settle / cooldown

- `GET /offers/song-library` → small list of song-fragment motif ids (code-split, fetched when offer surface opens).
- Per-bird offer cooldown (few minutes) is enforced **both** client-side (disable the affordance, no error noise) and server-side (tick ignores offer events inside the cooldown window) so cooldown can't be bypassed by a crafted request and curiosity-drift can't saturate in one session.

### 4.8 Social / visits

- `POST /invites` `{ visitor_email }` → issues one-time link, emails visitor. Visits default OFF: this is the only way to enable sharing; there is no global discoverable flag.
- `GET /visit/consume?token=…` → starts a **read-only** visit session bound to host aviary.
- `GET /visit/snapshot` (visitor session) → same `AviarySnapshot` as the host sees, **no special/prettified rendering**, no event-append capability. Returns "visit no longer available" (matter-of-fact) if revoked/expired.
- `GET /invites` (host) → outstanding + active; `POST /invites/{id}/revoke` → immediate effect at next visitor pull.
- `GET /visit-log` (host) → who visited, when, approx duration. Pulled on demand; **no badge, no push** unless the host opted into visit notifications.

### 4.9 Account export / deletion

- `POST /account/export` → generates JSON snapshot (birds, names, **current** vectors, current moods, notebook, settings), emails a download link to the verified address.
  - Note: export is the **only** place vector numbers exist in user-reachable output, and it is a private data-portability artifact emailed to the account owner — not a UI surface, not a stats panel. It does not violate "never exposed numerically in the product" because it is the user taking a copy of their own data, off-surface. (Defensible call; flagged for privacy review.)
- `POST /account/delete` → mark `pending_deletion`; recoverable for 30 days via any signed-in "I changed my mind"; hard-delete job after window.

---

## 5. Simulation engine design

The engine is the product. This section specifies the tick, the drift function, mood transitions, and the call-grammar runtime with enough precision to test against.

### 5.1 The tick

A sharded scheduled worker, sharded by account UUID, leased per shard for exactly-once execution. Cadence config constant `TICK_INTERVAL = 60s` (calibrated against the p99-5s tick-latency error budget, §11). Per account, each tick:

1. **Load** `BirdCanonical[]` + aviary row for the account.
2. **Read** unprocessed events in `server_ts` order.
3. **Recompute day_phase** from the account's local timezone; **expire/spawn weather** per the weather scheduler (§5.5).
4. **Compute presence-time** for this window from `presence_ping` intervals (§5.3).
5. **Apply additive personality deltas** (§5.2) — read-modify-write traits, clamped to range, **monotonic up only**.
6. **Transition moods** (§5.4) per bird from interactions + time-of-day + weather + ambient (bird-to-bird) + the bird's own personality.
7. **Recompute perch** from mood+boldness (front/middle/back is a *signal*, never user-set).
8. **Advance call intent**: set each bird's next call window + signature seed from vocal-frequency + mood + chorus state (§5.8).
9. **Possibly emit a notebook entry** subject to sparsity rules (§5.7).
10. **Write** canonical state, bump `canonical_state_version`, stamp events `processed_by_tick_at`.

The tick is **idempotent-safe under retry**: it only advances events it marks processed, and delta application is additive over the *unprocessed* set, so a crashed-and-retried tick that didn't commit re-reads the same unprocessed events and produces the same delta. (No double-application because marking-processed and writing state commit in one transaction.)

The tick runs for every account on cadence **whether or not a client is connected** — this is what makes the returned-to aviary the one that has been running, and what makes mood transition through the day during absence. Drift during absence comes only from events recorded *before* the user left (presence-time already banked), never from inputs invented at return.

### 5.2 Drift function

Drift is a **slow low-pass filter** over presence-and-interaction signals, monotonic toward expressive.

Per trait, per tick:
```
delta = GAIN[trait] * weighted_signal(window)
trait_new = clamp(trait_old + max(0, delta), TRAIT_MIN, TRAIT_MAX)   // max(0,·): never decreases
```

- `weighted_signal` is dominated by **presence-time**, then **listen-in** (boosts the focused bird's social-warmth + vocal-frequency), then **offers** (accept → small curiosity; offering-near → small boldness). Settle contributes **no** directional drift.
- `GAIN[trait]` constants are tuned so the calibration assertions in §5.6 hold. They live in one config module with a comment block tying each constant to its calibration target.
- **Monotonic-up only is enforced in the function** (`max(0, delta)`), not as a downstream guard — neglect can never decrease a trait. A bird that is ignored doesn't get warier; it gets *ambient* (it simply hasn't banked the presence that would have raised its greeting frequency). This is the engine-level implementation of "no Tamagotchi" and is a unit-test invariant: feed an event stream with long gaps and assert no trait ever decreases.

Plumage saturation drifts up with sustained attention and never down — same rule, no special case.

### 5.3 Presence-time computation

A `presence_ping` is emitted by the client only while the **three-way conjunction** holds (visible AND focused AND recent pointer/key within the 3-min window). Each ping carries the `[start,end]` interval it covers. The tick:
- Merges overlapping/adjacent intervals across pings (dedupes multi-tab and resend).
- Sums covered seconds → `presence_seconds(window)` → primary drift input.
- **Never** counts "tab open." A background tab emits no pings (visibility false → no ping). A focused-but-idle laptop stops emitting pings after the activity window lapses. This honesty is the whole point (§concepts): laxer presence silently inflates drift across the population.

Visitor sessions emit **no** presence pings and write **no** events — a visitor watching for an hour drifts nothing.

### 5.4 Mood transitions

Mood is a small FSM per bird (`wary, content, curious, drowsy, alert, settled`). Each tick computes a transition from weighted inputs:
- **Recent interaction:** accepted offer → toward content/curious; listen-in → toward alert/curious.
- **Time of day (local):** dusk → drowsy; early morning → alert; full night → settled (eyes closed, low on perch) — except the nightjar-species, which may stay active/calling.
- **Weather:** rain dampens vocal frequency briefly and nudges toward content/drowsy; wind nudges some toward alert, others toward wary (per species behavior_profile). Short-lived.
- **Ambient / bird-to-bird:** one bird's alarm/wary tends to spread to nearby birds; a chorus pulls participants toward alert/content. (§5.9)
- **Personality bias:** high-boldness birds resist `wary` on the same input; high-social-warmth birds settle into `content` more readily.

**Mood persists across sessions:** the mood at session-end is the mood at next session-start, *modulated by what the tick did in between* (a bird that ended `drowsy` at dusk is likely `settled` by morning's tick chain). The client never snaps mood to a default on tab open — it renders whatever the snapshot says, and the snapshot is the continuously-ticked canonical mood.

### 5.5 Day/night and weather scheduler

- Day-phase derived each tick from the account's stored IANA timezone. Palette/calls/mood biases follow phase. Night is not dead: most birds `settled`, nightjar active.
- Weather is rare and non-assertive: a scheduler gives each aviary a low per-day probability of a short `rain` and occasional `wind`, bounded so it reads as "the aviary has its own moments," never a weather feature. No thunderstorm/snow. Effects are short and surface mainly as small mood shifts.

### 5.6 Drift calibration harness (a real test, not a guideline)

A deterministic simulation harness drives synthetic event streams through the real tick code and asserts:
- **Instrument-measurable at ~1 week:** a "regular visitor" profile (e.g. ~20 min presence/day) produces a *measurable* trait delta after 7 simulated days (delta > instrument epsilon).
- **User-visible at ~3 weeks:** the same profile crosses a "visible" threshold (the magnitude that maps to a perceptible behavior change — greets-first frequency, comes-to-front frequency) at ~21 days, and **not before** (no single session moves a trait visibly).
- **Monotonic invariant:** no event stream, including long neglect gaps, ever decreases any trait.
- **No-saturation invariant:** offer cooldown prevents curiosity from maxing within one session.
- **Honesty invariant:** a "tab left open, no activity" stream produces ~zero drift (no presence pings).

These assertions pin the `GAIN`/`TICK_INTERVAL`/window constants. If a constant change breaks a calibration assertion, CI fails. This is the only place these numbers are pinned down, so the harness is a v1 deliverable, not a follow-up.

### 5.7 Notebook generation (sparse, specific, never about the user)

The tick may emit a notebook entry when a **noteworthy aviary transition** occurs (first-greeter-of-the-week changed; a long quiet stretch; a chorus event; a bird came to the front for the first time in a while; weather passed). Rules:
- **Sparsity:** target ~1 entry per few days for a regular aviary; a rate limiter + `source_signal` dedup prevents an entry-per-session even for very active users. Noteworthy events can raise the rate, ordinary sessions don't.
- **Voice:** naturalist, lowercase, present-tense, specific to bird+moment, generated from state (not a fixed template, not gamification language). *"pip greeted before wren today, first time this week."* never *"Achievement unlocked."* never *"session started 7:43."*
- **Observations of the aviary, never of the user:** the generator can write that Pip greeted first; it **cannot** write that the user visited every day, cannot count visits, cannot reference streaks. This line is enforced by restricting `source_signal` kinds to aviary-state transitions; there is no user-behavior signal in the generator's input set. (This is the notebook-level enforcement of "no streak counter in disguise," §13.2.)
- **Read-only / immutable:** no edit/delete/annotate; infinite scrollback; old entries never archived.

### 5.8 Call-grammar runtime

Calls are **procedural**, synthesized client-side, but the *intent and timing* are server-driven so the chorus and recognizability survive sync.

- Each species has a **motif library** (small set of pitch/rhythm motifs) and a **call signature** — the invariant character that makes Pip identifiable by ear across mood and drift. Signature is keyed to `species` + `bird.call_grammar_seed` (stable per bird).
- The tick sets each bird's **next call window** and a per-call **variation seed** from vocal-frequency (rate), mood (timbre/energy), and chorus state. The client synthesizes the actual call from motif + seed at that window, varying timing/pitch every time (never identical twice — looped audio is the audible signature of dead software).
- **Recognizability across drift:** the signature parameters (which motifs, characteristic interval shape) are invariant; only rate/timbre/variation move. A user who knows Pip after two weeks still knows Pip when her vocal-frequency has drifted up.
- The runtime also produces the **caption text** (§8.3) from the same call parameters, so a caption always matches what was actually played.

### 5.9 Bird-to-bird interaction

The tick models the aviary as a small social system, not independent NPCs:
- A call from one high-vocal-frequency bird raises the probability the next tick that a nearby high-social-warmth bird answers (call/response).
- A `wary` mood tends to spread to nearby birds (with personality resistance).
- A **chorus** emerges when ≥2 high-vocal-frequency birds have overlapping call windows; chorus participants bias toward alert/content and the client mixes real procedural calls (not stacked loops — the phase-cancel artifact of layered recordings is exactly what we avoid, §7).

---

## 6. Sync model

Multi-device sync is **not a feature**; it is the absence of client-owned state. This section states the guarantees and the failure modes they foreclose.

### 6.1 One canonical record, many readers

The server is the sole writer of personality/mood/perch/notebook. Laptop and phone both `GET /aviary/snapshot` and render the same canonical state in the same mood with the same drift. There is **no client-to-client sync, no client state to merge, no eventual-consistency reconciliation** — both clients read one record.

### 6.2 Clients write events, never state

Every device writes interaction events to the append-only log. No device ever sends "set boldness = 0.62." It sends "user listened in to Pip for 3 minutes" and the server decides what that means. Enforced at the DB-grant layer (§2.2): the API role cannot write trait columns.

### 6.3 No last-write-wins on personality

Personality deltas are **additive, server-authored, processed in event-log order.** The morning laptop session's drift and the lunchtime phone session's drift are both *events* the tick consumes in `server_ts` order and *adds*; neither overwrites the other. The classic failure — a phone session that started on a stale read overwrites the laptop's just-banked drift, silently and unlogged — is **unreachable** because no client write carries an absolute trait value and the tick never takes a client's view of the vector. This is the implementation rule that makes server-canonical correct rather than a label, and it's covered by a concurrency test (two overlapping sessions; assert both sessions' presence-time both contribute to drift).

### 6.4 Sync conflict & account-error surfaces (matter-of-fact voice)

Rare cases — magic-link replay, in-flight session timeout mid-write, server outage — surface in matter-of-fact tone, never naturalist:
- *"We couldn't sign you in. The link may have expired. Try requesting a new link."*
- *"Your session timed out. Sign in again to keep watching."*
- *"Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."*

These surfaces (sign-in, account settings, sync errors, accessibility settings) are the **named exception** to the naturalist voice and are gated by a `SystemSurface` component family distinct from the naturalist surfaces (§8.5), so the register can't drift.

### 6.5 Snapshot pull cadence & interpolation

- Client pulls a fresh snapshot on `visibilitychange`→visible, on a detected long render-frame gap (laptop suspend/resume), and on a 30s keepalive while visible.
- Between snapshots the client **interpolates** (a bird at perch A in snapshot N, perch B in N+1 is rendered moving smoothly, never teleporting).
- `version` is monotonic; the client ignores out-of-order/stale snapshots (keeps the highest version) to avoid visual rubber-banding.
- When hidden, the client **stops rendering** (battery) while the server keeps ticking; on return the client pulls and resumes from the now-current state — the aviary that has been running, not the frozen one.

---

## 7. Audio pipeline

Audio is the affective spine. The rules here are unconditional.

### 7.1 Procedural synthesis, no recorded audio, ever

Calls are synthesized client-side via WebAudio (`AudioWorklet`) from the per-species motif library. **No recorded audio path at any quality** — recorded loops are the audible signature of dead software, and layered recorded loops produce the phase-cancel artifact the ear catches. Procedural is also forced by the 2MB bundle: we can't carry recorded audio at the variation the chorus needs.

### 7.2 Per-call variation

Each call is generated fresh from motif + the tick-provided variation seed + mood-shaped timbre/energy. The same bird never produces the identical waveform twice. Vocal-frequency drives rate and chorus-join readiness.

### 7.3 Chorus mixing

Two+ birds with overlapping call windows mix as **real-time independent voices** through a small mixer graph, not stacked loops. This is what makes a chorus a chorus. The mixer is a bounded graph (fixed node count ≤ 7 voices + ambient bus) to honor the no-memory-growth rule (§9.4).

### 7.4 Listen-in mix

Listen-in is a **gradual re-balance**, not a mute and not a channel switch:
- On engage: slow ramp up the focused bird's bus gain, slow ramp down the others **to ambient (never to silence)**. A hard cut would turn the aviary into soloable tracks — wrong product.
- On disengage (click focused bird again, focus another, click empty space, move keyboard focus away, Escape): slow ramp back to ambient with the same time constant.
- The mix is client-side and immediate; the `listen_in_start/end` events are the durable attention signal the tick reads for drift.

### 7.5 Listen-in mix decay & ambient bed

Other birds drop in the mix but keep calling on an ambient bus — multiple things are still happening at once. The decay is a time-constant ramp, not a step.

### 7.6 WebAudio fallback = graceful silence + captions on

If WebAudio is unavailable (old browser, audio-context permission denied, hardware fault), the aviary plays in **graceful silence with captions on by default** (§8.3). There is **no recorded-audio fallback**. Silence-with-captions is a better fallback than canned audio. The audio subsystem detects unavailability at init and flips the captions-default + a quiet, matter-of-fact note in accessibility settings (not a toast on the scene).

### 7.7 Audio resource management

Audio buffers/nodes are pooled and reused; no per-call allocation that isn't freed; one bounded `AudioContext`; worklet processors reused. This is a CI-tested invariant (§9.4).

---

## 8. Accessibility surfaces (first-class, ships with v1)

Accessibility is a **designed surface**, not a checklist fallback. All of the below ships with v1 — a reduced-motion mode that lands as a "v1.1 fix" is a v1 that told reduced-motion users the product isn't for them.

### 8.1 Single source of truth → three renderings

The same canonical state drives the **visual** surface, the **reduced-motion** surface, and the **screen-reader narration**. They are three renderings of one state, so they can't drift apart. A "narration view-model" derives from the snapshot exactly as the visual scene does.

### 8.2 Screen-reader narration (naturalist prose, not a state list)

- Running prose in the **same naturalist voice** as the notebook: *"a small grey bird is perched on the front rail, calling softly. another sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."* — never *"Pip at perch 2, mood content."*
- Generated from canonical state (server- or client-side) into an ARIA live region.
- **Cadence:** ~1 update per 30–60s at idle (high-frequency narration would flood the SR queue and force the user to silence it). User-initiated events (successful offer, settle, return-greeting) get a **priority bump** and are narrated promptly — still as observations, not state transitions.
- Voice continuity is mandatory: a user moving between aviary and notebook hears one product, not two. A planner who treats narration as ARIA-label automation has built the wrong feature — this is called out so it isn't.

### 8.3 Call captioning (from the live grammar)

- Opt-in (and **on by default** when audio is unavailable, §7.6).
- Short naturalist prose of what each call sounds like in the bird's current mood: *"a soft three-note rise"*, *"a low trill, paused, low trill again"*, *"a single sharp call from the back perch."*
- Generated from the **same call parameters actually played** (§5.8), not a fixed per-call string — each caption matches the real call.
- Rendered as small text near the calling bird, fading in/out with the call. Passes WCAG AA contrast against the scene (§8.6).

### 8.4 Reduced-motion mode (designed, not stripped)

Triggered by `prefers-reduced-motion` or an accessibility-settings opt-in. **Not "animations off"** — a different rendering of the same aviary:
- Micro-motion → slow cross-fades between still poses (a preening bird becomes cross-faded preen-poses).
- Flight transitions → slow cross-fades between perches, not animated paths.
- Ambient leaf/feather drift removed; ambient day→evening color shifts **remain, slowed**.
- **Calls still play (or caption), birds still drift, mood still changes, notebook still notices.** It's the aviary in a calmer visual register, with its own quiet charm — not a broken-looking static scene. A vestibular user gets a calmer Pocket Aviary, not less of one.

### 8.5 Keyboard navigation & focus

- Tab cycles top-bar items (account/settings, accessibility, notebook, offer). Tab into the scene focuses the first bird; arrow keys move focus between birds; **Enter** triggers listen-in on the focused bird; **Escape** exits listen-in. Offer surface opens via top-bar shortcut and is fully keyboard-navigable; settle reachable from the top bar.
- Visible focus indicator: a soft high-contrast outline legible against both bright and dim aviary states (exact treatment from the design system).

### 8.6 Contrast & the voice split in code

- All user copy (top-bar labels, settings, account, error surfaces, captions, displayed narration) passes **WCAG AA** as a floor. The scene itself carries no user copy except the top bar, so contrast applies to chrome.
- The **naturalist vs. matter-of-fact** split is enforced structurally: a `SystemSurface` component family (sign-in, account, sync errors, accessibility settings) uses matter-of-fact copy; everything else uses naturalist copy. Copy lint/snapshot tests assert no naturalist phrasing in `SystemSurface` and no announcement phrasing in naturalist surfaces. The split is load-bearing and named once so no future feature re-litigates it.

---

## 9. Frontend rendering pipeline & performance

### 9.1 Scene composition

- One horizontal scene, one canvas (2D recommended), three perch zones, subtle 2-plane parallax (foreground branch/leaf occasionally passes; background foliage/sky behind; birds+perches on the middle plane). **No pan/scroll/zoom.**
- Responsive: scene compresses on narrow viewports and widens on wide ones, **always keeping every bird in frame** — never crop, never let a bird drift offscreen.
- **No UI chrome inside the scene** — birds and place only. Chrome lives in a thin top bar (account/settings, accessibility, notebook, offer; nothing else) that **fades to near-transparent after a few seconds of cursor stillness** and returns on cursor/keyboard activity.

### 9.2 Loads with motion already in progress (the central conceit)

- First frame has birds mid-action (mid-preen, soft call from the high perch, a leaf drifting). **No entry animation, no fade-from-static, no spinner, no "wake up" sequence.**
- Implementation: the initial HTML document from the CDN edge carries the **inline state snapshot**; the render path places birds at their current positions/motions from that snapshot and starts the loop as if it had been running. The client pulls a fresh snapshot immediately after to reconcile.
- **Slow-load state = a quiet field** (soft sky, one or two faint motion cues), **never a spinner.** A spinner says "machine"; the quiet field reads as the aviary catching up.
- **Empty-aviary state** (between adoption and first bird) = the same quiet field; the first bird then **flies in softly** to its starting perch. After that the user never sees an empty aviary again.

### 9.3 Idle micro-motion & transitions

- Continuous, **mood-shaped** idle motion (wary perches back and scans; content preens; curious tilts toward sounds/watches leaves; drowsy sits low, fluffed). The user reads mood from motion — **no label, tooltip, or status icon** ever tells them what a bird feels.
- Birds are never still in a way that reads as paused. Idle runs continuously (client stops rendering only when the tab is hidden; the *simulation* never pauses).
- Ambient leaf/feather drift: **client-only ornaments** at idle cadence, no server/per-leaf state.
- Interpolation between snapshots (§6.5) makes perch changes smooth.

### 9.4 Performance budgets (v1 hard targets, CI-enforced)

- **Bundle < 2MB gzipped at first paint.** Enforced by a CI bundle-size gate. Aggressive code-splitting for less-frequent surfaces (account settings, accessibility settings, visit-invite flow, song-fragment library). Bird visuals are procedural where possible, otherwise small SVGs/compact bitmaps. Audio is synthesized (no recorded files) partly to fit this budget.
- **Time-to-first-bird < 500ms** on mid-tier mobile / 4G. Achieved via: bundle budget + inline snapshot from CDN edge with the HTML + a render path that draws the first bird without waiting on non-critical assets. Measured by synthetic checks (§11) and gated.
- **60fps idle on a 5-year-old mid-range laptop**, sustained over a 30-minute session (runtime budget, not just launch). Single render loop, pooled sprites, no per-frame allocation.
- **No memory growth over 30 minutes** — a **real CI test**, not a guideline: pooled/reused audio buffers, bounded audio graph, notebook entries scrolled out drop references, bounded worker/audio contexts. A 30-min headless session asserts flat heap.
- **Browser support:** last two major versions of Chrome, Safari, Firefox, Edge. Older browsers get a matter-of-fact unsupported-browser surface; **no compatibility paths for very old browsers** (bundle bloat not justified).

---

## 10. Affective-constraint enforcement (notice-never-announce as architecture)

This is the section that keeps the product from sliding into the thing it's adjacent to. The constraints are enforced in code so they survive feature work and well-meaning contributors.

- **No announcement UI primitive exists.** There is no `Toast`, `Banner`, `WelcomeBack`, `LevelUp`, or notification-on-scene component in the component library. The return-greeting (a bird directive, §4.3) is the *entire* welcome surface. Adding a textual welcome is the single most damaging "notice, never announce" violation, and the way it's foreclosed is that the primitive to build it isn't in the kit and a lint rule flags any new always-on overlay added to the scene.
- **No gamification data exists to surface.** No visit counter, no streak field, no "days visited," no green-dot calendar, no level/score/badge/XP/rank/tier, no "birds adopted: N." The system **doesn't compute** these, so there's nothing to "just expose later." The notebook generator's input set excludes user-behavior signals (§5.7) so it can't write a streak in disguise. Visit-frequency is not surfaced anywhere.
- **No Tamagotchi affordance exists.** Monotonic-up-only drift (§5.2) means there is no decay path, no distress state, no hunger/happiness meter, no death. Neglect → ambient quietness (banked-less-presence), never suffering. The return after two weeks shows quieter birds to ease back into, never a guilt surface.
- **Personality is never numeric on any surface** (§3.4) — two distinct DTO types, CI serialization test, no debug/admin/tier exception.
- **Social is one read-only affordance** (§4.8) — no co-presence, chat, avatars, comments, discovery, leaderboards, show-off rendering, or friend-visited push (opt-in only, off by default). The underlying cross-account metrics are **not computed**, so a leaderboard can't "just be exposed."

These are tracked as explicit acceptance criteria, not vibes (§14).

---

## 11. Observability & privacy boundary

### 11.1 Aggregate-only telemetry, physically separated

- Collected: request counts, latencies, **simulation-tick latency**, error rates, anonymized session-duration histograms (no per-account dimension), client render-frame timing, audio-context error counts, page-load and first-bird-render timings (synthetic + aggregate RUM).
- **Never** collected into telemetry/analytics/training: per-bird state, per-account interaction history, anything reconstructing a user's relationship with their aviary.
- **Architectural enforcement:** the telemetry pipeline has **no connection to the simulation database**; the simulation DB is **never read by the analytics warehouse**; ML training (if it ever exists) **never receives per-bird fields.** This is a privacy claim that behaves as an infra rule — the data simply doesn't flow across the boundary because the network/credential paths don't exist. Per-bird events are stored **only** to drive that user's own simulation, never aggregated for any purpose (not even an "average drift" dashboard).

### 11.2 Error budgets & alarms

- **Simulation-tick latency p99 > 5s → alarm** (catches degradation before users feel "running slow"; the tick should take far less).
- First-bird-render regression and bundle-size regression are CI gates (block merge), not just dashboards.
- Synthetic performance fleet: automated browsers run the aviary on a schedule from common geographies, reporting first-bird and frame timings.

### 11.3 Privacy policy surface

Plain-text privacy policy in account settings, naming the aggregate categories and **explicitly excluding** per-bird interaction state. Matter-of-fact voice.

---

## 12. Rollout

### 12.1 Build sequencing (phased, each phase shippable-internally)

1. **Foundations & the bright line.** Account/auth (magic link, sessions, synthetic UUID, encrypted email), DB schema with the **personality-write grant split** (§2.2), append-only event log, snapshot read API. Prove a client can render a static snapshot and append an event but *cannot* write a trait.
2. **The tick + drift.** Simulation worker, additive monotonic drift, mood FSM, day/night, weather, **the calibration harness (§5.6)** — built alongside, gating the constants. Prove "feels alive over weeks" in the harness before any polish.
3. **The session surface.** Canvas render loop, three perches, loads-with-motion, mood-shaped idle, interpolation, top-bar + fade, listen-in (client mix), offer (+ cooldown), settle (+ undo), return-greeting directive, notebook read surface, adoption flow.
4. **Audio.** Procedural synthesis worklet, per-call variation, chorus mixer, listen-in ramp, WebAudio fallback to silence+captions.
5. **Accessibility (in parallel from phase 3, not after).** Narration live region, reduced-motion designed rendering, captions from grammar, keyboard nav + focus, contrast pass, the `SystemSurface`/naturalist split.
6. **Social.** Invite issuance, read-only visit door (no event-append), revocation, visit log, opt-in notifications.
7. **Account lifecycle & privacy.** Export, soft/hard delete, sessions revocation, privacy policy surface, telemetry pipeline with the enforced separation.
8. **Perf hardening & rollout gates.** Bundle gate, time-to-first-bird, 60fps, no-memory-growth CI, synthetic fleet, tick-latency alarm.

Accessibility (5) and audio (4) are not tail phases — they ship with v1 by being built alongside the session surface.

### 12.2 Birds-per-aviary ramp

- Every new account starts with **two system-selected starter birds** (not a catalog — first encounter is meeting an animal, not configuring an avatar). User names them at adoption, renameable any time.
- **Third bird and beyond is paced by aviary age** (§1.3), never by visits/interactions/payment. A new-species offer appears at age-tied intervals; cap **7**. The mechanic deliberately refuses to teach "more attention earns more stuff."
- The 7-cap is built into the engine (recognizability ceiling for call signatures), revisited only if future audio-mix work raises the ceiling.

### 12.3 Day-one instrumentation

Aggregate operational telemetry + synthetic fleet + the tick-latency alarm + CI perf gates are live from launch. No per-account behavioral instrumentation — by design.

### 12.4 Launch gates (must pass to ship v1)

- Calibration harness assertions green (§5.6).
- Bundle < 2MB, first-bird < 500ms, 60fps/30-min, no-memory-growth — all green in CI.
- Personality-never-numeric serialization test green; DB write-grant split verified.
- Accessibility: narration, reduced-motion, captions, keyboard, AA contrast all functional (not stubbed).
- Affective-constraint acceptance criteria (§10) all met.

---

## 13. Risks & mitigations

### 13.1 Personality-vector loss (worst possible failure, likely invisible)

Losing a vector = deleting the bird the user knows; a "reset" bird passes every unit test and only un-reveals itself to a user who can't name what's wrong.
- **Mitigations:** vectors are server-only, single-writer, additive (never recomputed from logs, never rebuilt by client). Point-in-time DB backups + restore drills. A **stable-bird-id invariant test**: rename, sync, simulated species-pool migration — assert `bird.id` and traits unchanged. No code path "regenerates" or "swaps" a bird. Restore procedure preserves drift history, not just existence.

### 13.2 Engagement features leaking in (the slow slide)

"Just one streak counter" / "just a small welcome toast" is the foothold.
- **Mitigations:** the primitives don't exist (§10); the metrics aren't computed; the notebook generator can't see user-behavior signals; a lint rule flags new always-on scene overlays and new cross-account aggregations of interaction data. Acceptance criteria (§14) make these explicit, reviewable refusals.

### 13.3 Drift mis-calibration (Tamagotchi on one side, screensaver on the other)

- **Mitigations:** the calibration harness (§5.6) pins the band with testable assertions (~1wk instrument, ~3wk visible, monotonic-up, no-saturation, presence-honesty). Constants live in one config tied to those assertions; a constant change that breaks calibration fails CI. Presence is the honest three-way conjunction so the population-wide drift signal isn't silently inflated.

### 13.4 Sync correctness / last-write-wins

- **Mitigations:** additive server-authored deltas in event-log order (§6.3); no client API to write traits (DB grant split); concurrency test with two overlapping sessions asserting both contribute. The failure mode is *unreachable*, not *handled*.

### 13.5 Audio uncanniness (looped/canned audio breaks the spell irrecoverably)

- **Mitigations:** procedural-only, no recorded path at any quality; per-call variation seed so no waveform repeats; real-time chorus mixing (no stacked loops / phase-cancel); recognizable signature invariants tested by ear in QA + a spectral-similarity guard that flags if two consecutive calls from a bird are too identical. Fallback is silence+captions, never canned audio.

### 13.6 Accessibility regressing to a stripped fallback

- **Mitigations:** one canonical state → three renderings (§8.1) so surfaces can't diverge; reduced-motion is a designed surface with its own QA, not "animations off"; narration/caption voice continuity asserted by copy tests; accessibility built alongside the session surface (phase 5 in parallel), gating launch. A reduced-motion or narration stub blocks ship.

### 13.7 Time-to-first-bird / bundle regressions

- **Mitigations:** CI bundle-size and first-bird gates block merge; inline-snapshot-from-edge render path; aggressive code-splitting; procedural assets. Synthetic fleet catches field regressions; spinner is forbidden so a slow load degrades to the quiet field, not a machine cue.

### 13.8 PII leakage via email-as-identifier

- **Mitigations:** synthetic UUID everywhere; email encrypted in one place; a lint/schema check that no new partition key, log field, telemetry dimension, or inter-service message carries email. Impossible to retrofit, so enforced from day one.

### 13.9 Visitor sessions corrupting host drift / leaking write access

- **Mitigations:** the visit door issues read-only snapshot streams with **no event-append capability** (separate credential/route); visitor sessions emit no presence pings; revocation takes effect at next pull; show-off/prettified rendering doesn't exist (visitor sees exactly the host's state).

### 13.10 Tick reliability (missed/duplicated ticks, shard rebalance)

- **Mitigations:** leased-shard exactly-once execution; tick transaction commits state-write + event-mark-processed atomically (idempotent under retry, §5.1); p99-5s latency alarm; backfill logic that, on a missed window, advances from `last_tick_at` so day/night and mood don't jump incorrectly after an outage.

---

## 14. Acceptance criteria (testable, the affective rules included)

A v1 is acceptable when, in addition to the §12.4 launch gates:

1. **Aliveness:** first frame shows birds mid-action; no spinner/entry-animation/fade-from-static anywhere; calls never repeat identically (spectral guard); idle motion is mood-shaped and continuous.
2. **Notice-never-announce:** no toast/banner/modal/welcome/notification on the scene; the bird greeting is the only welcome; no "you've been gone X days" surface exists.
3. **No gamification:** no streak/score/level/badge/XP/calendar/"birds adopted: N" anywhere; the underlying metrics are not computed; notebook never writes user-behavior observations.
4. **No Tamagotchi:** no decay/death/distress/hunger/happiness meter; drift is monotonic-up (invariant test); neglect → ambient quiet, never suffering.
5. **Personality never numeric:** serialization test green; no debug/admin/tier exposes traits.
6. **Drift calibration:** harness assertions green (~1wk instrument, ~3wk visible, monotonic, no-saturation, presence-honesty).
7. **Sync:** no client trait-write path (grant verified); overlapping-session concurrency test shows additive drift; no last-write-wins reachable.
8. **Audio:** procedural-only; real chorus; listen-in is a gradual re-balance (others to ambient, never silent); fallback is silence+captions.
9. **Accessibility (ships with v1):** naturalist narration at slow cadence with event priority bump; reduced-motion as designed cross-fade rendering; captions from live grammar; full keyboard nav + visible focus; AA contrast; voice split enforced (`SystemSurface` vs naturalist).
10. **Social:** per-invite opt-in, read-only, revocable; default off; no co-presence/chat/avatars/comments/discovery/leaderboards; visitor never drifts host; friend-visited notification opt-in and off by default.
11. **Privacy/infra:** synthetic-UUID-only (no email outside the one encrypted field); telemetry physically separated from simulation DB; aggregate-only metrics; export + soft/hard delete work.
12. **Performance:** bundle < 2MB, first-bird < 500ms (mid-tier/4G), 60fps/30-min on 5-yr laptop, no memory growth over 30 min — all CI-gated.

---

## 15. Open calibration items (named, not deferred — resolved during build with the harness)

- Exact presence activity window (default 3 min; harness + field-tuning leaning long).
- Exact tick cadence (default 60s) vs. tick-latency budget.
- `GAIN[trait]` constants pinned by the calibration assertions.
- Third-bird age cadence (initial ~8wk then ~10–12wk, cap 7) — tunable, age-driven only.
- Trait ranges and per-species seed values (simulation-service implementation detail; never user-visible).
- Notebook sparsity rate limiter thresholds (target ~1 / few days; never per-session).

These are the only numbers the PRD leaves to build-time, and each is owned by a test or a config module rather than left implicit, so calibration is a measured activity, not a guess.
