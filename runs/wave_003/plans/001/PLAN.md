# Pocket Aviary — v1 implementation plan

**Status.** Phase-1 plan derived from the nine PRD files under `prd/`. It interprets the spec into buildable decisions; it does not restate it. Where the PRD leaves a value or mechanism open, the choice is made here and logged in §13 with its rationale and the test that guards it.

**How to read.** Numbers labelled *initial* are launch defaults that the calibration harness (§5.12) is expected to tune. Numbers labelled *hard* fail CI or abort a tick. The invariants in §0 are the spine; every later section names the mechanism that enforces the ones it touches. The plan is written for a team that will build it without further clarification; open questions are resolved, not deferred.

---

## 0. Invariants with teeth

The PRD's principles are treated as invariants with enforcement, not as tone guidance.

| # | Invariant | Enforced by |
|---|---|---|
| I1 | Personality is written only by the server tick. Clients submit events, never values. | DB column grants (§3.6); API has no trait write path (§4); tick compare-and-set (§6.3) |
| I2 | Drift is monotonic non-decreasing per trait. | Drift math is non-negative by construction (§5.3); tick asserts `new ≥ old` and aborts; DB trigger rejects decreases |
| I3 | Presence = visible ∧ focused ∧ recent pointer/key activity; unioned across devices; never exceeds wall time. | Client tracker (§7.9); server union and clamp (§5.2); harness test T5 |
| I4 | Personality numbers never reach a user surface. | Snapshot schema carries only quantized render parameters (§4.2); client bird type has no trait fields; UI lint (§7.10); export is the single named exception |
| I5 | No announcement surfaces: no toasts, banners, welcome text, streaks, counters, badges, calendars. | Component registry allowlist and lint (§7.10); prose lint forbids user-behaviour observations (§5.9) |
| I6 | Procedural audio only. No recorded audio in bundle or on the wire. | Asset lint fails CI on any audio file; fallback is silence plus captions (§8.8) |
| I7 | Per-account interaction data never leaves the simulation database. | Network/IAM isolation; RUM and metrics schemas have no identity fields (§10.4); negative connectivity test in launch gates |
| I8 | Email is stored once, encrypted. Every other reference is the synthetic account UUID. | Schema (§3.1); log PII scanner at CI and at log ingest (§10.4) |
| I9 | Bird identity is a stable UUID. No code path replaces, resets, or regenerates a bird. | `birds` rows are never deleted or re-seeded except by account hard-delete (§3.2); migration policy (§6.6) |
| I10 | Accessibility surfaces ship in v1, in the product voice. | Launch gates (§11.3); narration, captions, reduced-motion are M1–M2 deliverables, not follow-ups |
| I11 | The first frame is the aviary, already moving. No spinner, no entry animation. | Boot path (§7.2); synthetic first-bird budget (§10.1) |

---

## 1. Scope

### 1.1 In v1

- **Birds.** Two starters chosen by the system from a six-species pool (one night-active). Age-based arrivals up to the hard cap of seven. Hidden five-trait personality vector with monotonic drift. Persisted mood. Bird-to-bird callbacks, alarm spread, chorus. Stable bird identity. User-assigned, renameable names.
- **Interactions.** Return-greeting (absence-aware, boldness-weighted, procedurally varied, staggered). Presence accounting. Listen-in with gradual mix re-balance. Offers: seed, song fragment, still pool, with per-bird cooldown. Settle with a five-second undo. Read-only, sparse field notebook.
- **Scene.** One horizontal screen, three perch zones, local-time day/night, rare rain and wind, ambient leaves and feathers, subtle parallax, thin fading top bar with exactly four icons, quiet-field loading state, empty-aviary fly-in after adoption.
- **Accounts.** Magic-link sign-in (15-minute, single-use links), per-device revocable sessions, verified email change, JSON export delivered as an emailed link, 30-day soft delete then hard purge, synthetic UUID identity, encrypted email at rest.
- **Sync.** Server-side tick as the sole writer of canonical state, snapshot pull, append-only event log, multi-device coherence by construction.
- **Visits.** Named email invites, one-time link, read-only ambient view with no presence or event capture, immediate revocation, 30-day expiry, visit log in settings, visit notification toggle off by default.
- **Accessibility.** Naturalist screen-reader narration on a slow cadence, reduced-motion as a designed cross-fade renderer, runtime-generated call captions, complete keyboard navigation, WCAG AA on all copy.
- **Performance.** Initial bundle ≤ 2 MB gzipped (hard), first bird visible < 500 ms on a mid-tier phone over 4G (hard), 60 fps idle motion on a five-year-old laptop for a 30-minute session, zero memory growth over 30 minutes (CI test), simulation-tick p99 alarm at 5 s.
- **Browsers.** Last two major versions of Chrome, Safari, Firefox, Edge. Others get a matter-of-fact unsupported surface.

### 1.2 Out of v1, and not designed for

Native apps; payments; shared, household, or multi-aviary accounts; customizable scenes; public discovery, profiles, follows, feeds; comments, chat, co-presence, avatars; leaderboards; achievements, streaks, levels, badges, counters, XP, calendars of any kind (including opt-in or hidden variants); hunger, death, distress, decaying happiness; push notifications or re-engagement email of any kind; user-arranged perches or drag-to-place; species catalog at adoption; any user, admin, or debug view of per-bird trait values; recorded audio; population analytics over interaction data; ML training on any per-bird field. The data model and protocols are not shaped for native clients.

### 1.3 Launch defaults

| Parameter | Value | Kind |
|---|---|---|
| Tick cadence | 60 s (idle coarsening to 300 s behind a flag, sub-stepped at 60 s, §5.1) | initial |
| Timeline horizon per tick | 90 s | initial |
| Snapshot keepalive pull | at `next_tick_at` + 2 s, jittered ±1 s, only while visible | initial |
| Long-frame-gap threshold triggering a pull | 5 s | initial |
| Presence activity window | 4 min (deliberately long) | initial |
| Presence heartbeat / flush | 20 s extend; flush interval events every 60 s | initial |
| Greeting absence buckets | < 2 min glance; 2 min–2 h short; 2 h–2 d medium; > 2 d re-orientation | initial |
| Greeting rate limit | 1 per 2 min per aviary | initial |
| Listen-in ramps and levels | engage 1.5 s, disengage 2.5 s; focused 1.0, others 0.25, floor 0.2, ambient 0.7 | initial |
| Offer cooldown | 4 min per bird; 45 s minimum between offers per aviary | initial |
| Settle | 4 s lighting fade; 5 s undo window | initial / hard (undo) |
| Top bar fade | after 3 s stillness, 600 ms fade to 8 % opacity; never while focused or a sheet is open | initial |
| Notebook sparsity | token bucket capacity 2, refill 1 per 72 h; ≤ 1 entry per local day | initial |
| Narration cadence | idle 45 s ± 15 s; minimum 8 s between utterances | initial |
| Reservoir time constant τ | 7 days | initial |
| Daily drift cap | 0.02 per trait per local day | hard (engine) |
| Magic link | 15 min TTL, single use; 5 requests/hour/email, 20/day | hard (TTL) |
| Session lifetime | 180 days sliding; revocable | initial |
| Invite expiry | 30 days unused; visitor session 30 days from first use | hard / initial |
| Soft delete window | 30 days | hard |
| Adoption ladder | 3rd bird at 75 d, 4th 150 d, 5th 240 d, 6th 365 d, 7th 540 d of aviary age | initial |
| Weather | rain 2–3/week, 3–8 min; wind 2–4/day, 1–3 min | initial |
| Event retention | consumed tick + 7 days, then purged | hard |
| Bird cap / starters / species | 7 / 2 / 6 | hard |

---

## 2. Architecture

### 2.1 Shape

```
 Browser (host or visitor)
 ┌──────────────────────────────────────────────────────────────┐
 │ chrome (DOM) │ scene (canvas) │ audio (WebAudio) │ a11y layer │
 │ presence tracker · timeline executor · local improviser      │
 │ @aviary/engine (derived use only) · @aviary/prose            │
 └──────▲───────────────▲──────────────────────▲────────────────┘
        │ HTML with     │ snapshots, events,   │ visitor
        │ inlined boot  │ offers, account,     │ snapshots
        │ snapshot      │ notebook, visits     │
 ┌──────┴────────┐ ┌────┴──────────────────────┴───────────────┐
 │ Edge worker   │ │ API service (stateless, TypeScript)       │
 │ shell, static,│ │ auth · snapshot · events · offers         │
 │ cookie verify │ │ account · notebook · visits · adoption    │
 └──────▲────────┘ └────▲──────────────────┬───────────────────┘
        │ snapshot cache│ read              │ append events
 ┌──────┴───────────────┴──────────────────▼───────────────────┐
 │ Simulation DB (Postgres) · Redis (leases, limits, cache)     │
 └──────────────────────▲──────────────────────────────────────┘
                        │ sole writer of bird/aviary state
 ┌──────────────────────┴──────────────────────────────────────┐
 │ Tick workers (leased partitions) running @aviary/engine      │
 │ presence · drift · mood · timeline · weather · notebook ·    │
 │ ladder · snapshot cache write                                │
 └─────────────────────────────────────────────────────────────┘
 side systems: email outbox → provider · export jobs → object storage · purge jobs
 observability account: metrics, logs, traces, RUM collector, synthetics
   — no network route and no credential to the simulation DB
```

Components:

- **Web client.** TypeScript SPA. Preact for chrome (top bar, sheets, settings, notebook, adoption, visits). Custom Canvas 2D scene renderer with a 2D bird rig. WebAudio synthesis engine. DOM accessibility layer. Service worker for the app shell.
- **Edge worker + CDN.** Serves the HTML shell with the boot snapshot and critical CSS inlined, static assets with immutable caching, and the visitor shell. Verifies the signed session cookie locally (no origin round trip) to select which cached snapshot to inline.
- **API service.** Stateless Node/TypeScript. Auth, snapshot assembly (including greeting plans), event ingestion, offer reaction plans, account, notebook, visits, adoption, export and deletion job enqueueing.
- **Tick service.** Node/TypeScript workers that lease partitions of aviaries and run the tick. The only writer of traits, reservoirs, mood, perch, timeline, weather, notebook entries, arrivals.
- **Shared packages.** `@aviary/engine` (pure, deterministic simulation and behaviour functions) and `@aviary/prose` (naturalist text generation and voice lint). Consumed by tick, API, and client.
- **Stores.** Postgres (canonical simulation state, accounts, events partitioned by day), Redis (rate limits, tick leases, snapshot cache), object storage (exports), KMS (email encryption keys, blind-index HMAC key).
- **Side systems.** Transactional email provider fed by an outbox table. Export and purge job runners. Synthetic browser fleet.

### 2.2 The render-pipeline boundary

The server decides *what happens*; the client decides *how it looks and sounds*. Nothing that affects canonical state is computed on the client.

| Layer | Owner | Contents |
|---|---|---|
| Canonical | tick (sole writer) | traits, reservoirs, mood and influences, perch slot, behaviour timeline, weather, notebook, ladder, bird identity and signature |
| Server-computed per request | API (reads state, appends events, never writes bird state) | greeting plan, offer reaction plan, snapshot assembly, arrival presentation |
| Derived on client | renderer, audio, a11y | micro-motion, flight paths, synthesized call audio, captions, narration text, leaves and feathers, lighting interpolation, listen-in mix |
| Device-local | client only | listen-in focus, settled lighting, audio-unlocked state, top-bar fade, effective motion mode, caption visibility |

Two devices reading the same snapshot render the same birds on the same perches in the same moods hearing the same phrases (same seeds); micro-motion phase may differ, which is acceptable and intentional.

### 2.3 Shared engine package

`@aviary/engine` is the reason the backend is TypeScript. It contains: trait and reservoir update, drift, mood scoring and transition, timeline generation, greeting selection, offer reaction, notebook detectors, ladder scheduling, the call grammar (symbolic phrase generation, not audio), and the PRNG. It has no I/O, no clocks, no globals: every function is `(state, inputs, seed, now) → output`. The tick runs it to write canonical state; the API runs it read-only for greeting and offer plans; the client runs it for the local improviser (§6.5) and to expand timeline seeds into phrases. Golden-vector tests pin server and client outputs to byte equality per engine version, and the engine version is carried in every snapshot so a client with a stale bundle falls back to server-provided expansions rather than diverging.

`@aviary/prose` contains the template grammar, slot fillers, anti-repetition memory, and the voice lint. It generates notebook entries (server), narration (client), captions (client), bird descriptors for `aria-label` (client), and adoption copy.

### 2.4 Deployment topology

Single primary region for Postgres (primary plus streaming replica), API, tick workers, Redis. Global CDN and edge workers for the shell and static assets. Object storage with signed URLs for exports. The observability stack lives in a separate cloud account with metric and log ingestion endpoints only; it has no VPC peering and no database credential. Analytics for product health is the aggregate metric stream, nothing else.

### 2.5 Trust and privacy boundaries

- The simulation database is reachable only from API and tick services through a role each. The `api` role cannot update bird trait, reservoir, mood, perch, or timeline columns. The `tick` role cannot read email ciphertext. The `analytics` role does not exist.
- Logs are structured, carry only synthetic UUIDs, and pass through a redaction filter that drops any field matching an email pattern; the same scanner runs in CI against fixture logs and against sampled production logs daily.
- RUM and metrics schemas are closed: a fixed set of metric names and coarse dimensions (browser family, device class, coarse region, app version). Adding a dimension requires a privacy review noted in the schema file.
- Visitors are unauthenticated browsers holding an invite-scoped token; they hit a read-only endpoint that cannot append events.

---

## 3. Data model

All identifiers are UUIDv7 (time-ordered, random). Timestamps are UTC `timestamptz`. Local-time reasoning uses the aviary's stored IANA timezone.

### 3.1 `accounts`

| Column | Notes |
|---|---|
| `id` | synthetic UUID; the only identifier used anywhere else |
| `email_ciphertext` | KMS envelope encryption; decrypted only to send mail or show the user their own address |
| `email_blind_index` | HMAC-SHA256(email, dedicated key), unique; used only for sign-in lookup and uniqueness. Not an identifier: never logged, never joined, never exported to another table |
| `pending_email_ciphertext`, `pending_email_token_hash`, `pending_email_expires_at` | email-change flow; old address keeps working until confirmation |
| `timezone` | IANA zone from the most recently active device (§6.4) |
| `status` | `active` or `pending_deletion`; `deletion_requested_at` |
| `settings` | jsonb: `captions` (off/on), `motion` (system/reduced/full), `narration_text` (off/on), `narration_pace` (normal/slower), `audio` (on/visible_only/off), `visit_notifications` (default false) |
| `created_at` | |

### 3.2 `birds`

| Column | Notes |
|---|---|
| `id` | stable identity; never reused, never rewritten |
| `aviary_id`, `species_id` | species is a code-level enum (§3.9) |
| `name`, `named_at` | ≤ 24 chars; `named_at` null means arrived but unnamed |
| `arrived_at` | adoption or arrival time |
| `bold`, `warm`, `vocal`, `plum`, `curio` | doubles in [0,1]; the personality vector |
| `r_listen`, `r_offer_curio`, `r_offer_bold` | per-bird reservoirs (§5.3) |
| `drift_today` (double[5]), `drift_day` (local date) | daily-cap accounting |
| `mood`, `mood_since`, `mood_influences` (jsonb list of decaying influences) | §5.5 |
| `perch_slot`, `pose_family` | current slot (0–8) and pose family |
| `signature` | jsonb: `pitch_center`, `tempo`, `brightness`, `motif_weights`, `ornament_p`, `rhythm` (§8.4); derived from `id` at arrival, stored, immutable |
| `cooldown_until` | offer cooldown |

Constraints: a `BEFORE UPDATE` trigger rejects any decrease of the five trait columns (belt and braces for I2). No `DELETE` grant exists for any runtime role; the hard-delete purge job uses a dedicated role.

### 3.3 `aviaries`

`id`, `account_id` (unique), `created_at` (the age source for the ladder), `seed` (32 random bytes), `tick_seq` (monotonic), `last_tick_at`, `next_tick_at`, `lease_owner`, `lease_until`, `timezone` (copied from account at each tick), `attention_reservoir` (A, §5.3), `last_presence_at`, `last_greeting_at`, `weather` (jsonb: `kind`, `started_at`, `ends_at`, `next_rain_at`, `next_wind_at`), `timeline` (jsonb, versioned by `tick_seq`), `notebook_tokens`, `notebook_last_local_date`, `ladder_next_at`, `ladder_paused`, `pool_until` (active still-pool offer).

### 3.4 `events` (append-only, partitioned by `received_at` day)

`id` (client-generated UUIDv7 for client events, server-generated for server events; primary key for idempotency), `aviary_id`, `source` (`client` | `server`), `session_id` (nullable), `type`, `bird_id` (nullable), `occurred_at` (client clock, clamped on read), `received_at`, `payload` (jsonb), `consumed_tick_seq` (nullable). Partial index on `(aviary_id, received_at)` where `consumed_tick_seq IS NULL`.

Event types and payloads:

| Type | Source | Payload |
|---|---|---|
| `presence.interval` | client | `from`, `to` (≤ 60 s span) |
| `presence.end` | client | `at` |
| `listen_in.start` / `listen_in.end` | client | `bird_id`, `at`, `duration_ms` (end only) |
| `offer` | client via offer endpoint | `kind` (seed/song/pool), `song_motif`, `seed`, `plan_hash` |
| `settle` | client | `at` (sent only after the undo window closes) |
| `greeting` | server (API) | `greeter_id`, `responders`, `form`, `absence_s` |
| `arrival` | server (tick) | `bird_id`, `species_id` |
| `rename` | server (API) | `bird_id` (for notebook awareness only; no payload text) |

Retention: rows are purged 7 days after `consumed_tick_seq` is set. This is the entire lifetime of raw interaction data; nothing downstream copies it.

### 3.5 `sessions`, `magic_links`

`sessions`: `id`, `account_id`, `token_hash` (SHA-256 of a 256-bit random token), `device_label` (UA family only, e.g. "Safari on iPhone"), `created_at`, `last_seen_at`, `expires_at`, `revoked_at`. The cookie carries `session_token` (HttpOnly, Secure, SameSite=Lax) plus a compact signed claim `{sid, aid, exp ≤ 24 h}` that the edge verifies with a shared key to choose the boot snapshot; revocation is enforced at the origin on the first API call.

`magic_links`: `id`, `email_blind_index`, `email_ciphertext` (needed to create the account on first consume), `token_hash`, `created_at`, `expires_at` (15 min), `consumed_at`, `request_ip_hash` (daily-salted, for rate limiting only).

### 3.6 Roles and grants

| Role | May |
|---|---|
| `api` | SELECT most tables; INSERT `events`, `sessions`, `magic_links`, `invites`, `exports`, `email_outbox`; UPDATE `accounts` (settings, email flow, status), `sessions`, `birds.name`/`named_at`, `invites.revoked_at`, `visitor_sessions`, `visits` |
| `tick` | SELECT `aviaries`, `birds`, `events`; UPDATE `aviaries`, `birds` (all simulation columns); INSERT `notebook_entries`, `birds` (arrivals), `events` (server-sourced); UPDATE `events.consumed_tick_seq` |
| `purge` | DELETE for hard-delete and retention jobs only |
| `export` | SELECT for the account being exported |

There is no role with `UPDATE` on trait columns other than `tick`. This is I1 as a grant, not a convention.

### 3.7 Notebook, invites, visits, exports, outbox

- `notebook_entries`: `id`, `aviary_id`, `created_at`, `local_date`, `text`, `template_id`, `bird_ids` (uuid[]). Index `(aviary_id, created_at DESC)` for cursor pagination.
- `invites`: `id`, `aviary_id`, `visitor_email_ciphertext`, `token_hash`, `created_at`, `expires_at` (+30 d), `consumed_at`, `revoked_at`.
- `visitor_sessions`: `id`, `invite_id`, `token_hash`, `started_at`, `last_seen_at`, `expires_at` (30 d from first use), `ended_at`.
- `visits`: `id`, `invite_id`, `aviary_id`, `started_at`, `last_seen_at`. A visit is a run of visitor snapshot pulls with gaps under 10 minutes; approximate duration is `last_seen_at − started_at`.
- `exports`: `id`, `account_id`, `requested_at`, `status`, `object_key`, `url_expires_at` (24 h).
- `email_outbox`: `id`, `account_id` (nullable), `kind` (magic_link, invite, export_ready, email_change, visit_notice, deletion_notice), `to_ciphertext`, `payload`, `created_at`, `sent_at`, `attempts`, `last_error`.

### 3.8 Export document

JSON with: account id, created_at, timezone, settings; birds with id, species, name, arrived_at, current personality vector (the PRD's explicit exception to I4: this is the user's own data, delivered as a file, never rendered in-product), current mood, signature; all notebook entries; visit log; outstanding invites. No event rows (they are transient) and no telemetry.

### 3.9 Species (code, not database)

Six species records in the engine package: `id`, display descriptors (for prose: "small grey bird", "the warbler"), silhouette part definitions (vector paths), base palette and a ten-step saturation ramp, `night_active` (true for the nightjar-like species only), idle style parameters, call grammar and motif library (§8.4), base call rate, seed ranges for traits.

Trait seed ranges (initial): bold 0.25–0.55, warm 0.30–0.60, vocal 0.30–0.60 (nightjar 0.40–0.70), plum 0.30–0.50, curio 0.30–0.60. Starters are sampled so |bold₁ − bold₂| ≥ 0.15, giving the first encounter a legible bolder-and-warier pair.

---

## 4. API surface

JSON over HTTPS, versioned by an `X-Aviary-Api: 1` header. All copy in error bodies comes from `copy/system.ts` (matter-of-fact voice). Cookies are HttpOnly, Secure, SameSite=Lax; state-changing requests also require an `Origin` check. Rate limits live in Redis keyed by account UUID or hashed IP, never by email.

### 4.1 Endpoints

| Method and path | Auth | Purpose |
|---|---|---|
| `POST /api/auth/magic-link` `{email}` | none | Always 200 (no account enumeration). Enqueues a link email. Limits: 5/h and 20/day per blind index, 30/h per IP hash |
| `GET /auth/consume?token=` | none | Validates hash, TTL, single use; creates account, aviary, and two unnamed starters if new; issues session; redirects to `/` (or `/adopt` for new accounts) |
| `POST /api/auth/signout` | session | Revokes the current session |
| `GET /api/aviary/snapshot?reason=open\|visible\|resume\|keepalive&tz=` | session | Canonical snapshot (§4.2). Includes a greeting plan when eligible (§5.7). Updates account timezone if `tz` differs |
| `POST /api/aviary/events` `{events:[...]}` | session | Batch append, ≤ 50 per call, idempotent on event id, 202 |
| `POST /api/aviary/offers` `{kind, song_motif?, event_id}` | session | Returns a reaction plan (§4.3); 429 with `available_at` when the aviary-level spacing is violated |
| `GET /api/aviary/adoption` | session | Pending starters and suggested names |
| `POST /api/aviary/adoption/name` `{names:{bird_id:name}}` | session | Names starters; schedules arrival fly-ins |
| `PATCH /api/birds/:id` `{name}` | session | Rename (also used to name a ladder arrival) |
| `GET /api/notebook?before=&limit=30` | session | Entries newest first, cursor by `(created_at, id)` |
| `GET` / `PATCH /api/account/settings` | session | Settings jsonb |
| `GET /api/account/sessions`, `DELETE /api/account/sessions/:id` | session | Device list with labels and last-seen; revoke |
| `POST /api/account/email-change` `{new_email}`; `GET /account/email-change/confirm?token=` | session / none | Verified change; old address works until confirm |
| `POST /api/account/export` | session | Enqueues an export job; email with 24 h signed link |
| `POST /api/account/delete`, `POST /api/account/restore` | session | Soft delete; restore within 30 days |
| `POST /api/visits/invites` `{email}` | session | Invite; 10/day per account, 3/day per distinct visitor blind index |
| `GET /api/visits/log` | session | Visits (email, date, approximate duration) and outstanding invites |
| `DELETE /api/visits/invites/:id` | session | Revoke; effective at the visitor's next pull |
| `GET /visit/:token` | none | Consumes on first use; sets visitor cookie; serves the shell in visitor mode |
| `GET /api/visit/snapshot` | visitor | Read-only snapshot; `410 {reason: revoked\|expired}` |
| `GET /api/health`, `GET /api/version` | none | Operations |

There is no endpoint that accepts a trait value, a mood value, a perch, or a timeline from any client.

### 4.2 Snapshot schema

```
{
  "engine_version": "1.3.0",
  "server_time": "2026-09-06T14:03:12.410Z",
  "tick_seq": 48122,
  "next_tick_at": "2026-09-06T14:04:00Z",
  "aviary": {
    "id": "…", "created_at": "…", "timezone": "Europe/Dublin",
    "lighting": { "phase": "day", "t": 0.62 },          // t: 0..1 within phase
    "weather": { "kind": "rain", "started_at": "…", "ends_at": "…" } | null,
    "pool_until": "…" | null
  },
  "birds": [
    {
      "id": "…", "name": "pip", "unnamed": false, "species": "warbler",
      "perch_slot": 1, "mood": "content", "mood_since": "…",
      "pose_family": "preen", "plumage_step": 4,
      "signature": { "pitch_center": 2210, "tempo": 1.05, "brightness": 0.55,
                     "motif_weights": [...], "ornament_p": 0.12, "rhythm": [...] }
    }
  ],
  "timeline": {
    "from": "…", "to": "…", "seed": "…",
    "items": [
      { "t": "…", "bird": "…", "kind": "call", "seed": "…", "phrase_kind": "unprompted" },
      { "t": "…", "bird": "…", "kind": "callback", "to": "…", "seed": "…" },
      { "t": "…", "bird": "…", "kind": "perch", "slot": 4 },
      { "t": "…", "kind": "chorus", "members": ["…","…"] },
      { "t": "…", "bird": "…", "kind": "preen" | "scan" | "stretch" | "drink" | "bathe" },
      { "t": "…", "kind": "weather", "state": "rain_start" | "rain_end" | "wind_start" | "wind_end" },
      { "t": "…", "bird": "…", "kind": "arrive" }
    ]
  },
  "greeting": { "greeter": "…", "form": "tilt_step", "responders": [{"bird":"…","offset_ms":1400}], "seed": "…" } | null,
  "offer_state": { "next_offer_at": "…", "cooldowns": { "<bird_id>": "…" } },
  "settings": { …account settings… }
}
```

The bird object has no trait fields. `plumage_step` is the only trait-derived value and is a quantized render parameter (0–9). The visitor variant omits `greeting`, `offer_state`, and `settings`, and carries `visitor: true`.

Snapshots are ~2–4 KB gzipped for seven birds. Responses carry `ETag: tick_seq` and return 304 on `If-None-Match` when nothing changed, which is the common keepalive case within a tick.

### 4.3 Offer reaction plan

Request: `{kind: "seed" | "song" | "pool", song_motif?: 0..5, event_id}`. The API:

1. Checks the aviary-level spacing (45 s) and returns 429 with `available_at` if violated. Per-bird cooldowns do not reject the offer; birds in cooldown simply respond with `ignore` and accrue no drift.
2. Draws a seed, runs `engine.offerReaction(state, kind, seed)` read-only, and appends an `offer` event with the seed and a hash of the plan.
3. Returns the plan: `{offer_id, placed_at, items: [{bird, reaction, t_offset_ms, seed}], pool_until?}` where `reaction ∈ approach_now | approach_slow | watch | ignore | join_song | quiet | call_against | drink | bathe`.

The tick recomputes the identical plan from the event (pure function, same seed) to apply mood influences and drift inputs. The client renders from the response immediately; the next snapshot agrees by construction.

### 4.4 Visit flow

1. Host enters an email in the visits sheet → `POST /api/visits/invites`. Server encrypts the address, stores a token hash, enqueues an invite email ("<host email> invited you to visit their aviary. This link works for 30 days."). Rate limits protect against using the product as a mail cannon.
2. Visitor opens `/visit/:token`. First use sets `consumed_at`, creates a `visitor_sessions` row and cookie (30 days). Later uses of the same link on the same browser reuse the cookie; on a different browser the link is rejected as used (matter-of-fact copy).
3. Visitor shell pulls `GET /api/visit/snapshot` on the same triggers as a host. Each pull updates the current `visits` row or opens a new one after a 10-minute gap. Visitor clients never send events and never receive greeting plans; the aviary is rendered exactly as the host would see it at that moment.
4. Revocation sets `revoked_at`; the next visitor pull returns 410 and the client shows "This visit is no longer available." No confirmation is sent to the host.
5. If `visit_notifications` is on, the first pull of a new visit enqueues one email to the host, at most one per visitor per 6 hours. Off by default and not mentioned in onboarding.

### 4.5 Auth and system copy

Every system string is reviewed against the matter-of-fact lint: normal capitalization, no bird verbs, states what happened and what to do. Examples shipped: "We couldn't sign you in. The link may have expired. Try requesting a new link." / "Your session timed out. Sign in again to keep watching." / "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." / "This browser isn't supported. Pocket Aviary needs a current version of Chrome, Safari, Firefox, or Edge."

---

## 5. Simulation engine

### 5.1 Tick loop and scheduling

Every aviary ticks on the server regardless of connected clients. Tick workers each lease a hash range of aviary ids (Redis lease, 30 s, renewed at 10 s) and every second scan their range for `next_tick_at ≤ now` (indexed). Per aviary:

1. Acquire the per-aviary lease row-lock: `UPDATE aviaries SET lease_owner=$w, lease_until=now()+20s WHERE id=$id AND lease_until < now() RETURNING tick_seq, …`. No row means another worker owns it; skip.
2. Load birds and unconsumed events for the aviary (bounded query; events are small).
3. Compute `dt = now − last_tick_at`. If `dt > 60 s` (coarsened idle tick, worker restart, backlog) sub-step the engine at 60 s increments so results are step-size independent. The tick never "invents" inputs for the gap: events carry their own timestamps and are attributed to the sub-step they fall in.
4. Run the engine: presence (§5.2), reservoirs and drift (§5.3), mood (§5.5), weather, timeline (§5.6), notebook (§5.9), ladder (§5.10).
5. Write in one transaction with compare-and-set: `UPDATE aviaries SET … , tick_seq = tick_seq + 1 WHERE id=$id AND tick_seq=$expected`; update birds; mark events consumed with the new `tick_seq`; insert notebook entries and arrivals. If the CAS fails, roll back and skip; nothing was applied twice.
6. Write the compact snapshot to the Redis/edge cache keyed by aviary id (TTL 10 min).
7. Set `next_tick_at = now + 60 s`, or `+ 300 s` when the idle-coarsening flag is on and no session has pulled a snapshot in the last 10 minutes and no unconsumed events exist. Any incoming snapshot request for a coarsened aviary triggers an immediate tick before responding (so the returning user gets fresh state and their greeting is computed against it).

Tick work per aviary is a few milliseconds; a worker handles thousands of aviaries per minute. Fleet size is sized from tick lag, not CPU.

### 5.2 Presence accounting (server side)

Per tick, gather `presence.interval` and `presence.end` events from every session of the account since the last tick. Clamp each interval's client timestamps to `[received_at − 5 min, received_at]` to defuse clock skew. Union the intervals across sessions (two devices present at once count once), clip to `(last_tick_at, now]`, and sum: `presence_minutes ≤ dt` always. Update `attention_reservoir` and `last_presence_at`. Listen-in minutes per bird are computed the same way from `listen_in.start/end` pairs, clipped to presence intervals (no listen-in credit without presence). Visitor sessions contribute nothing.

### 5.3 Drift

```
reservoirs (all ≥ 0; exponential decay with τ = 7 days; units are "minutes at rate", so R_ref = rate_ref × τ):
  A     (aviary)  ← A·e^(−dt/τ)     + presence_minutes(tick)
  L_b   (bird)    ← L_b·e^(−dt/τ)   + listen_in_minutes_b(tick)
  C_b   (bird)    ← C_b·e^(−dt/τ)   + offers_accepted_b(tick)      // approach, drink, bathe, join_song
  B_b   (bird)    ← B_b·e^(−dt/τ)   + offers_near_b(tick)          // any reaction other than ignore, or on front perch during the offer
saturation:   s(R) = min(R / R_ref, 1)
raw delta:    Δx = (1 − x) · dt_days · Σ_signal k[x][signal] · s(R_signal)
daily cap:    Δx ← min(Δx, 0.02 − drift_today[x])       // hard; reset at local midnight
apply:        x ← min(1, x + Δx);  assert x_new ≥ x_old   // hard; abort the tick and page if violated
```

Weights `k` (initial, per day at saturation):

| signal → trait | bold | warm | vocal | plum | curio |
|---|---|---|---|---|---|
| presence A (rate_ref 20 min/day) | 0.012 | 0.012 | 0.010 | 0.018 | 0.008 |
| listen-in L_b (5 min/day) | 0 | 0.010 | 0.008 | 0.004 | 0 |
| offers accepted C_b (2/day) | 0 | 0 | 0 | 0 | 0.010 |
| offers near B_b (2/day) | 0.006 | 0 | 0 | 0 | 0 |

Projection for the reference user (15 min/day, five days a week, one two-minute listen-in per session): boldness +0.014 by day 7 (instrument resolution is 0.001), +0.085 by day 21; plumage +0.02 / +0.13. Neglect: all reservoirs decay toward zero, every Δx → 0, no trait moves. Monotonicity is structural: reservoirs and weights are non-negative, so Δx ≥ 0 for every input; the assertion and DB trigger exist to catch bugs, not to implement the rule.

### 5.4 Expressiveness, and why neglect reads as quiet

The PRD wants a neglected aviary to greet less and call less without any trait moving down. The reservoir is the state that does that:

- `E = 0.5 + 0.5 · s(A)` — the aviary's expressiveness. Multiplies greeting probability and unprompted call rate. After two weeks away, `A` has decayed to ~14 % of its steady value and `E ≈ 0.57`; within a week of regular presence `E > 0.9` again. Traits are untouched throughout.

Perceptual mappings (how traits become behaviour; these are what "visible drift" means and what the harness tunes against):

| Behaviour | Mapping (initial) |
|---|---|
| Greet-first weight | `w_b = exp(2.5·bold + 1.5·warm) · m(mood) · E` |
| Front-perch propensity | `σ(6·(bold − 0.45))`, ×0.2 when wary, ×0.5 when drowsy |
| Unprompted call rate | `λ_b = base_species · (0.4 + 1.2·vocal) · mood_factor · E · vocal_damp · tod_factor` |
| Callback probability | `0.15 + 0.6·warm`, ×mood factor |
| Seed approach probability | `σ(5·(curio − 0.4))`, mood-adjusted |
| Head-tilt to sounds and leaves | probability ∝ curio |
| Plumage step | `floor(plum·10)`; feather-detail overlays appear at steps ≥ 6 |

### 5.5 Mood

Mood set (decision, §13): `alert`, `curious`, `content`, `wary`, `drowsy`, `roosting`. `roosting` is the night rest state (eyes closed, low on the perch); the aviary lighting state is called "settled" and the bird state is not, to keep the two words apart.

Each tick, per bird, score every mood: `S_m = prior_m(local phase, species) · personality_mod_m + Σ influences_m(t)`.

Priors (initial):

| Local phase | alert | curious | content | wary | drowsy | roosting |
|---|---|---|---|---|---|---|
| dawn 05–08 | .40 | .25 | .25 | .10 | 0 | 0 |
| day 08–17 | .15 | .30 | .40 | .10 | .05 | 0 |
| evening 17–21 | .10 | .15 | .35 | .05 | .35 | 0 |
| night 21–05 | 0 | 0 | .05 | 0 | .15 | .80 |
| night, night-active species | .35 | .25 | .30 | .05 | .05 | 0 |

Personality modifiers: wary × (1 − 0.6·bold); curious × (0.5 + curio); alert × (0.7 + 0.6·curio).

Influences are appended by events and decay exponentially (half-life in parentheses): offer accepted +0.5 content, +0.3 curious (30 min); listen-in active +0.2 content (30 min); neighbour alarm +0.6·(1 − bold) wary (10 min); rain +0.1 drowsy and `vocal_damp = 0.4` during the rain and 5 min after; wind +0.3 alert if curio > 0.5 else +0.3·(1 − bold) wary (15 min); settle +0.5 drowsy (20 min); song fragment +0.3 curious, +0.2 alert (20 min). Influences that survive a session end keep decaying with a 6-hour half-life, so by morning the prior dominates: that is the "daily-ish reset".

Transition: sample from softmax(S, temperature 0.35) with the bird's seeded stream. Switch only if the sample differs from the current mood, the score gap is ≥ 0.15, and dwell ≥ 10 min. Event-driven nudges (offer, alarm, settle) bypass the dwell rule but not the gap. Mood and `mood_since` persist; nothing about a snapshot request touches mood.

### 5.6 Behaviour timeline

For the window `[now, now + 90 s]` the tick emits an ordered item list (§4.2):

1. **Perch.** Move probability 0.08 per tick (×1.5 curious, ×0.5 drowsy, 0 roosting). Target zone from front-perch propensity and mood (wary → back; content/curious → middle or front by bold); pick a free slot in that zone (three slots per zone, nine total). Item `perch{t, slot}` at a random `t` in the window.
2. **Calls.** Poisson process with `λ_b` (§5.4); each `call{t, seed, phrase_kind}` where `phrase_kind` follows mood (§8.5). Roosting birds have λ ≈ 0; the night-active species keeps its normal rate at night.
3. **Callbacks.** For each call, every other awake bird responds with `p_cb` within 0.8–3 s: `callback{t, to, seed}`. Warm birds occasionally get `perch` items to a slot adjacent to another warm bird.
4. **Chorus.** When two or more birds with vocal ≥ 0.5 have calls within 2 s, emit `chorus{t, members}` so clients render deliberate overlap and the notebook detector fires.
5. **Alarm.** On wind onset, or for a wary bird with small probability, emit `alarm{t}`; the wary influence spreads to birds in the same or adjacent zone at the next tick.
6. **Weather.** If none is active and `next_rain_at`/`next_wind_at ≤ now`, start it; emit `weather` items with start and end timestamps so every device agrees. Rain inter-arrival is exponential with mean 60 h and duration 3–8 min; wind mean 8 h, 1–3 min.
7. **Activities.** Two to eight discrete pose events per window (`preen`, `scan`, `stretch`; `drink`/`bathe` when a pool is active) weighted by mood.

Seeds: `seed = H(aviary.seed, tick_seq, bird.id, item_index)`. Two devices expand the same seed into the same phrase and the same choreography.

### 5.7 Return-greeting (computed at snapshot time)

When `reason ∈ {open, visible, resume}` and `now − last_greeting_at ≥ 2 min`, the API computes a greeting plan against current state and appends a server-sourced `greeting` event (the notebook needs to know who greeted first):

- **Absence** `Δ = now − last_presence_at`. Form by bucket: `< 2 min` → `glance` (head lift toward the viewer, no call); `2 min–2 h` → `glance` or `two_note` (weighted by warm); `2 h–2 d` → `tilt_step` (head tilt, one step toward the front of the perch, short call) with one responder at `p_cb`; `> 2 d` → `reorient` (hop to a front slot if bold ≥ 0.45, longer phrase, one or two responders). Roosting birds do not greet; if every bird is roosting, the night-active bird calls once, or the plan is a single `shift` (a bird resettling its weight).
- **Greeter lottery.** Weighted sample with `w_b` (§5.4). The bolder, warmer bird usually greets first; "first time this week" happens naturally.
- **Stagger.** Responders offset by 0.8–2.5 s, randomized; never simultaneous.
- **Variation.** The form is a family, not a clip: the client expands `(form, seed, mood, signature)` into head angles, timing, phrase, and step distance. A 10 000-seed test asserts no two expansions are identical.

The greeting is delivered inside the first snapshot, so it is on screen within the first second. Short absences (a tab switch) produce only a glance, which is the intended texture.

### 5.8 Offer reactions

`engine.offerReaction(state, kind, seed)` maps each bird's mood, curio, bold, cooldown, and perch to a reaction and a choreography offset (§4.3): seed → `approach_now` (content/curious, curio high), `approach_slow` (wary; waits 6–15 s then comes near), `watch` (alert or drowsy with low curio), `ignore` (roosting, or in cooldown); song → `join_song` (vocal high, content/curious), `quiet` (wary/drowsy), `call_against` (alert, vocal high); pool → `drink` (content), `bathe` (curious/content and bold ≥ 0.5), `watch`. The tick applies the resulting influences (§5.5) and reservoir inputs (§5.3) and sets `cooldown_until = now + 4 min` for every bird that did not `ignore`. The pool stays for 6 minutes (`pool_until`) and appears in every device's snapshot.

### 5.9 Field notebook generation

Runs in the tick. Detectors emit candidates with a noteworthiness score:

| Detector | Score | Notes |
|---|---|---|
| newcomer arrived | 1.0 | bypasses the token bucket |
| greet-order change vs. the week's pattern | 0.8 | at most once per week |
| wary bird accepted an offer | 0.8 | |
| first front-perch for a bird in ≥ 5 days | 0.7 | |
| chorus event | 0.6 | |
| plumage step increased | 0.6 | phrased as light and colour, never as a number |
| rain passing, with what the birds did | 0.5 | |
| night call by the night-active bird | 0.5 | |
| two warm birds perched adjacent | 0.5 | |
| long daytime quiet (no calls for 2 h) | 0.4 | |
| preen or stretch moment | 0.3 | |
| leaf or feather moment | 0.2 | filler: only when nothing scored ≥ 0.5 for 5 days |

Selection per tick: candidates with score ≥ 0.5 compete; the winner is written only if `notebook_tokens ≥ 1` (capacity 2, refill 1 per 72 h) and no entry exists for today's local date. Filler keeps a regularly visited aviary from going more than a week without an entry; an absent user's aviary still produces a thin trickle from weather and night calls, because the aviary continues.

Composition (`@aviary/prose`): template families with slot fillers — lowercase bird names, species descriptors, perch names ("front rail", "low perch", "high branch", "back perch"), time words ("this morning", "just after the rain", "late, well after dark"), and a verb allowlist (perch, preen, fluff, scan, call, tilt, settle, drink, bathe, watch, notice, listen). The first entry of a day is prefixed with the weekday ("tuesday — …"). Anti-repetition: no template reused within the last 10 entries.

Voice lint (CI, on every template and on 10 000 generated samples): lowercase only; present tense (verb allowlist); no "you"/"your"; no digits; no exclamation marks; ≤ 180 characters; forbidden vocabulary (achievement, streak, level, score, unlock, congratulations, welcome, visited, days in a row, session, login). Templates can only reference aviary observations; there is no slot type for user actions, visit counts, or dates other than the weekday header.

### 5.10 Adoption ladder and arrivals

`ladder_next_at` is set at aviary creation to `created_at + 75 d` and advanced through 150, 240, 365, 540 days as birds arrive. At tick, if `ladder_next_at ≤ now`, the aviary has fewer than 7 birds, and `ladder_paused` is false: create a bird (species chosen to maximize signature distance and species variety; no species repeats until all six are present), `named_at = null`, trait seeds from species ranges, signature from id. Schedule its `arrive` item for the next local morning between 07:00 and 10:00 so the user finds it "at the back perch this morning"; the newcomer detector writes the day's entry. The bird is a full bird from arrival: it accrues drift, calls, and callbacks. Naming is quiet: the first listen-in on an unnamed bird opens a small naming sheet under the top bar ("a new bird has been at the back perch. what will you call it?"), and the settings sheet lists birds for naming or renaming at any time. Nothing nags; an unnamed bird stays unnamed indefinitely and is described by species in prose.

Starters at account creation follow the same path with `arrive` items scheduled immediately after naming, staggered 1.5 s apart.

### 5.11 Determinism and PRNG

xoshiro128\*\* seeded from SHA-256 of `(aviary.seed, tick_seq)`, split into per-bird and per-subsystem streams. All engine functions are pure in `(state, events, seeds, now)`. Golden vectors per engine version are checked in; a change to any output requires a version bump, and the tick and client both refuse to mix engine versions (the client falls back to server-expanded items for the tick in which versions differ, then reloads the bundle at the next idle window).

### 5.12 Calibration harness and targets

A headless runner in the engine package drives synthetic profiles through 90 simulated days at 60-second ticks in seconds of wall time. Profiles: reference (15 min/day, 5 of 7 days, one 2-minute listen-in per session), heavy (120 min/day, daily, five listen-ins, offers at cooldown), light (5 min, 2 of 7), absent-after-three-weeks, night-owl, multi-device (two overlapping devices), tab-left-open (visible and focused but no pointer or key activity), visitor-only (a visitor watching for hours).

| Test | Assertion |
|---|---|
| T1 measurable | reference: max trait Δ ∈ [0.010, 0.030] by day 7 |
| T2 visible | reference: at least two traits Δ ≥ 0.08 by day 21; none > 0.20 |
| T3 bounded | heavy: per-day Δ ≤ 0.02 every trait; day-21 total ≤ 0.25 |
| T4 monotone | property test over random event logs: no trait ever decreases |
| T5 honest presence | tab-left-open: ≤ 4 presence minutes per idle stretch; multi-device: presence ≤ wall time; visitor-only: zero presence, zero drift |
| T6 neglect reads as quiet | absent profile: E < 0.65 within 14 days, greeting probability falls, traits unchanged; E ≥ 0.9 within 7 days of return |
| T7 day cycle | over 30 days, roosting share at night ≥ 80 % for non-night-active species; wary share ≤ 15 % for birds with bold ≥ 0.5 |
| T8 notebook sparsity | reference: 0.2–0.5 entries/day; heavy: ≤ 0.5/day; no template repeated within 10 entries; zero lint failures |
| T9 calls | unprompted calls per hour within the species band; ≥ 1 chorus per day when two birds have vocal ≥ 0.5; zero identical phrases in 10 000 samples per bird |
| T10 greeting | 10 000 seeds: greeter distribution matches `w_b`; no identical expansions; responders never at offset 0 |

The felt half of calibration ("visible after three weeks") is checked in beta through opt-in research sessions and diary interviews, never through production interaction data (I7).

### 5.13 Engine test suite

Unit tests per function with golden vectors; property tests for monotonicity, presence clamping, and cap accounting; a tick integration test against a real Postgres with two workers racing on the same aviary (exactly one CAS succeeds, drift applied once); sub-step equivalence (one 300 s tick equals five 60 s ticks to within floating tolerance, given the same events); timezone tests across DST transitions and travel; ladder timing tests; notebook lint over generated corpora.

---

## 6. Sync model

### 6.1 One canonical record

The aviary row and its birds are the state. Every client renders the latest snapshot it has and interpolates toward the next. There is no client-to-client path, no merge, and no client-held state worth reconciling beyond the device-local items in §2.2.

### 6.2 Pull triggers and snapshot handling

Clients pull on: initial open (the inlined boot snapshot counts, followed by a fresh pull within 2 s), `visibilitychange` to visible, window focus after ≥ 60 s unfocused, a render-frame gap > 5 s (laptop suspend), timeline exhaustion, and the keepalive at `next_tick_at + 2 s` while visible. A snapshot is applied only if its `tick_seq` is greater than the current one. Reconciliation is always animated: a differing perch becomes a normal hop or flight (or a cross-fade in reduced motion); a differing mood cross-fades pose families over 3 s; timeline items already in the past are dropped and an in-flight call finishes. Nothing snaps.

### 6.3 Single writer, no last-write-wins

Only the tick writes simulation columns (§3.6). It consumes events in `received_at` order and applies additive deltas; the CAS on `tick_seq` makes double application impossible even when a lease expires mid-tick and a second worker starts. Event ids are client-generated UUIDv7 and the `events` primary key, so retried batches are idempotent. The only client-writable aviary data are bird names and account settings, which use last-write-wins with `updated_at`; the PRD's no-LWW rule is about personality state and those fields are not it.

### 6.4 Multi-device details

- Presence from several devices is unioned (§5.2); listen-in credit requires presence on the same device.
- Timezone: each snapshot request carries the device's IANA zone; the account adopts the most recently active device's zone, so the aviary shares the day of whichever device the user is on. Both devices open at once in different zones is resolved by recency.
- Greeting rate limit prevents a laptop and phone opened together from producing two greetings; the second device shows the glance form.
- Settled lighting is device-local; the `settle` event still quiets mood server-side.
- Offers and cooldowns are aviary-level, so a phone cannot bypass a laptop's cooldown.

### 6.5 Offline and stale behaviour

If a pull fails, the client keeps executing the current timeline, then hands off to the **local improviser**: `engine.generateTimeline` run client-side with a local seed and the last known state, producing visual and audio continuity (idle calls, small perch moves) with no canonical effect. Events queue in memory (bounded to 500, oldest presence intervals dropped first) and flush with backoff. On reconnect, the next real snapshot wins and reconciles as in §6.2. The user never sees a frozen aviary or an error for a transient network gap; a persistent failure (> 5 min) surfaces the matter-of-fact reload copy in a sheet, not over the scene.

### 6.6 Migrations and bird identity

Schema migrations never rewrite `birds.id`, never re-seed traits, and never regenerate signatures; a migration that needs to change a trait's semantics must be a monotone re-mapping applied once with an audit row per bird. Species pool changes add species; they never remove or remap existing birds. This policy is written into the migration checklist and reviewed like a security change.

### 6.7 Failure modes and recovery

| Failure | Effect | Recovery |
|---|---|---|
| Tick worker dies mid-tick | lease expires in 20 s; no partial write (single transaction) | another worker picks it up; events remain unconsumed |
| Backlog (fleet too small) | `next_tick_at` lag grows | alarm at p99 lag 120 s; autoscale; sub-stepping preserves correctness |
| Snapshot cache stale or missing | edge inlines nothing; client draws the quiet field and pulls | first-bird budget degrades for that session only |
| Client clock wildly off | clamped timestamps; server offset from `server_time` | none needed |
| Duplicate event batches | ignored by primary key | none needed |
| Magic link replay | rejected (consumed_at set) | matter-of-fact copy |

---

## 7. Frontend rendering pipeline

### 7.1 Stack and bundle plan

TypeScript, ES modules, Vite build with manual chunking. Preact for DOM chrome; no UI framework in the render loop. Canvas 2D for the scene (seven birds and a few dozen particles do not need WebGL, and Canvas 2D has the least driver variance on old laptops; the `SceneRenderer` interface allows a WebGL implementation later). No third-party scripts of any kind (privacy and bundle).

| Chunk | Estimated gzipped | Loaded |
|---|---|---|
| inline boot script + critical CSS (in HTML) | 8 KB | with the HTML |
| core: engine, prose, timeline executor, presence tracker | 60 KB | modulepreload |
| renderer: layers, rig, species parts, layout solver | 90 KB | modulepreload |
| audio engine | 40 KB | modulepreload |
| chrome: Preact, top bar, sheets, a11y layer, captions | 30 KB | modulepreload |
| notebook / settings and account / visits / adoption | 10 / 25 / 10 / 10 KB | on demand |
| **initial total** | **≈ 230 KB** | hard cap 2 MB; CI alert above 700 KB |

Fonts are the system stack; no web fonts on the critical path. Species art is vector part data, not bitmaps.

### 7.2 Boot path (first frame is the aviary)

1. **Edge.** The worker verifies the signed cookie claim, reads the cached compact snapshot for the aviary, and returns the shell with the snapshot, critical CSS, and the boot script inlined. `Cache-Control: private, no-store` for the HTML; assets immutable with content hashes.
2. **Inline boot** (runs before any external script): sizes the canvas, computes the layout from the viewport, and draws sky for the current local phase, perches, and every bird as a species silhouette in the pose family from the snapshot, at its slot. This is the first bird: no spinner, no fade, no "ready" state. Performance mark `aviary:first-bird`.
3. **Main bundle** (preloaded, deferred) mounts the full renderer on the same canvas, seeds micro-motion from bird ids so the first animated frame matches the static pose, starts the timeline executor at the current server time, and renders the greeting from the snapshot. There is no visual discontinuity by test: a pixel-diff between the boot frame and the renderer's first frame must be under a small threshold.
4. **Fresh pull** within 2 s reconciles anything the cached snapshot missed.
5. **Repeat visits.** A service worker caches the shell and the last snapshot; a repeat open draws the first bird from cache in well under 200 ms regardless of network, then pulls. The cache is cleared on sign-out and on session revocation errors.

If no snapshot can be inlined (cold cache, first-ever visit), the boot script draws the **quiet field**: the phase-correct sky gradient and one faint leaf drifting, and the renderer places birds as soon as the pull returns, each with its current pose — still no spinner, no fade-from-static.

### 7.3 Scene composition and layout

Layers, back to front: sky gradient (phase colour) → background foliage (parallax 0.2, cached offscreen canvas) → back perches → back-zone birds → middle perches and birds → front perches and birds → still pool (when active) → foreground branch and leaf particles (parallax 1.15) → weather (rain streaks, wind sway via a shear transform on the foliage layer) → lighting overlay (evening warmth, night dim). Captions and the focus ring are DOM, positioned from canvas coordinates.

Layout solver: scene box = viewport minus top bar; three depth rows with three slots each; slot x from width with minimum spacing 64 px at 320 px width; bird scale `clamp(width/1280, 0.55, 1.1)`; ultra-wide viewports cap the scene at 1920 px centred with foliage bleed; portrait phones increase vertical separation between zones. Tested from 320 to 2560 px: every slot's bird bounding box stays inside the box. No panning, scrolling, or zooming exists.

Parallax follows slow ambient drift only (not the cursor), amplitude a few pixels; it reads as depth, not as an effect.

### 7.4 Bird rig and procedural idle motion

Each bird is a small 2D rig: body, head, beak, eye, wing, tail, feet, drawn from per-species vector parts rasterized into a sprite atlas per `(species, plumage_step)` on demand (about ten parts, at most two DPR scales). Per frame the rig evaluates ~12 parameters: breath (0.25–0.4 Hz, amplitude by mood), weight shift (every 8–20 s), head yaw and pitch from layered noise (curious birds track leaf particles and recent call positions), fluff (drowsy, cool mornings), blink (Poisson), tail flick, preen sequence (procedural keyframes over 3–6 s, triggered by timeline `preen` items or idly when content), scan (wary), and roosting posture at night. Noise streams are seeded from the bird id, so a bird's manner is its own and persists across devices without being identical frame for frame.

Pose families by mood: `alert` upright and still with quick head turns; `curious` head tilts and forward lean; `content` relaxed, preening; `wary` further back, upright, scanning; `drowsy` low, fluffed, slow blinks; `roosting` eyes closed, lowest posture. Mood is read from posture; there is no label, tooltip, or icon.

### 7.5 Transitions and choreography

- **Perch move.** Quadratic Bézier between slots, 0.8–1.6 s by distance, anticipation crouch 150 ms, wing cycle at 8 Hz in flight, landing settle 300 ms.
- **Greeting forms.** `glance`: head lift toward the viewer over 1.2 s. `two_note`: glance plus a short phrase. `tilt_step`: head tilt, one step along the perch, short phrase. `reorient`: hop to a front slot, longer phrase, responders staggered. Expanded from `(form, seed, mood, signature)` so no two are alike.
- **Offers.** Seed placed at front centre; `approach_now` flies to the front and pecks in a loop for 4–8 s; `approach_slow` waits then comes; `watch` turns the head. Song fragment: birds `join_song` with a phrase timed to the fragment's end, `call_against` with a contrasting phrase. Pool: reflective ellipse, ripple rings on `drink`/`bathe`, bathe includes a shake.
- **Settle.** Lighting overlay interpolates to evening over 4 s; audio bus −8 dB over the same ramp; birds cross-fade toward drowsy posture. Any click, tap, Enter, or Space within 5 s reverses everything and cancels the event; after 5 s the `settle` event is sent and the aviary stays settled until tab close or a click, key, or interaction on the scene, which restores normal lighting over 3 s (the server's mood nudge stands; that is fine).
- **Arrival fly-in.** From off-canvas, 2.2 s, lands on a back slot; starters arrive 1.5 s apart into the quiet field.

### 7.6 Timeline executor and interpolation

A scheduler consumes timeline items by server time (offset computed from `server_time` on each snapshot, smoothed). Calls hand off to the audio engine and captions; perch, activity, greeting, and offer items drive the rig. Between snapshots the rig interpolates; when a snapshot arrives, positions and moods reconcile per §6.2. When a timeline runs out with no snapshot, the local improviser (§6.5) continues.

Frame loop: `requestAnimationFrame` with a fixed 60 Hz simulation step and accumulator; render every frame; device pixel ratio capped at 2; hidden tab stops the loop (rendering only; see §8.7 for audio); return to visible pulls a snapshot, re-syncs the clock, and resumes.

### 7.7 Reduced-motion renderer

`StillsRenderer` implements the same `SceneRenderer` interface as the motion renderer and is a designed surface, not a fallback. Rig state is quantized to pose keyframes; on change, two rasterized poses cross-fade over 1.5–3 s. A perch move is a 1 s fade-out at the old slot and a 1 s fade-in at the new one. No leaf or feather particles. Lighting shifts at half speed. Rain is a slow tint and a narration line, not streaks. Captions, calls, focus ring, greeting (as a pose cross-fade plus the call), and offers all remain. Active when `prefers-reduced-motion: reduce` is set or the account motion setting is "reduced"; switching re-mounts the renderer on the same canvas without a pull.

### 7.8 Top bar and chrome

The top bar is DOM above the canvas with exactly four icon buttons: account and settings, accessibility, field notebook, offer. It fades to 8 % opacity after 3 s of pointer stillness and restores in 150 ms on `pointermove` or `keydown`; it never fades while any child has focus, while a sheet is open, or when reduced motion is on (it stays at a steady 60 %). Sheets (notebook, settings, visits, adoption, offer, naming) open anchored under the top bar over the scene and are the only place chrome exists. The scene holds no DOM except the accessibility layer and captions.

The offer sheet lists the three offers and, visually separated below them, "settle the aviary for the evening" (§13). The account sheet holds settings, birds (naming, renaming), sessions, visits, export, email change, deletion, and the privacy policy link.

### 7.9 Presence tracker

```
present ⇔ document.visibilityState === "visible"
        ∧ document.hasFocus()
        ∧ (now − lastActivityAt) < 4 min
lastActivityAt ← pointermove (throttled to 1/s), pointerdown, touchstart, wheel, keydown
```

While present, the open interval extends every 20 s and flushes as `presence.interval` events every 60 s (each ≤ 60 s). On any condition failing, `presence.end` is sent (via `sendBeacon` on `pagehide`, blur, or hidden). Visitor mode never instantiates the tracker. Touch counts as pointer activity; on phones, watching without touching lapses after the four-minute window, which is the spec's definition and is noted as a known texture (§13).

### 7.10 UI lint and voice enforcement

- **Component registry allowlist.** The chrome package exports exactly: `TopBar`, `Sheet`, `IconButton`, `Toggle`, `TextField`, `ListRow`, `Caption`, `NarrationRegion`, `FocusLayer`, `QuietField`. No toast, snackbar, banner, modal, badge, or notification component exists; a lint rule fails CI on any new component name matching those words or on `alert()`.
- **Copy tables.** Every user-facing string lives in `copy/naturalist.ts` or `copy/system.ts`. Naturalist strings pass the voice lint (§5.9); system strings pass the matter-of-fact lint (sentence case, no bird verbs). Product surfaces import only from the naturalist table; account, error, sync, unsupported-browser, and accessibility-settings surfaces import only from the system table. The rule is enforced by import-boundary lint.
- **Forbidden identifiers** anywhere in client source: streak, badge, achievement, level, score, xp, welcome, toast, confetti.
- **No trait fields.** The client's `Bird` type is generated from the server snapshot schema, which has none; a test asserts the schema has no key matching the five trait names.

### 7.11 Memory and frame discipline

Pools: leaves 16, rain 200, ripples 8, captions 7 DOM nodes, voices 14, envelope `Float32Array`s allocated once per voice. Snapshot objects are replaced, never accumulated. Notebook is a windowed list keeping ≤ 60 rows mounted. Listeners are bound once and removed in `destroy()`. Per-frame allocation is zero by design (object reuse; no closures created in the loop), verified by a heap-allocation sampling test. Offscreen layer caches re-render only on lighting phase change or every 20 s.

### 7.12 States

Loading: quiet field. Empty aviary (post-adoption): quiet field, then fly-ins. Unsupported browser: static page with system copy (feature detection: ES2020 modules, Canvas 2D, `requestAnimationFrame`; WebAudio is optional). Errors: matter-of-fact sheet; the scene keeps showing the last known state or the quiet field. Audio-locked: see §8.8.

---

## 8. Audio pipeline

### 8.1 Graph

```
per voice (2 per bird, 14 pooled):
  osc(sine) + osc(triangle, −14 dB) → waveshaper(brightness) → biquad LP → voiceGain(envelope)
  shared looped noise buffer (2 s) → biquad BP → noiseGain(envelope)      // chips, burrs, churrs
  → birdGain(mix level) → StereoPanner(perch x) → dry → master
                                                 → send → reverb → master
ambient bed: pink noise → HP/LP → slow LFO gain (wind swell)
rain: white noise → HP + random droplet blips through a small resonant filter
master: gain (night and settle trims) → DynamicsCompressor(threshold −18 dB, ratio 2, knee 12) → destination
reverb: procedurally generated 0.9 s decaying-noise impulse response built at boot (no file)
```

All nodes are created once at engine start. Oscillators run continuously at zero gain; calls are envelopes, not node lifecycles, which removes per-call allocation and start/stop clicks.

### 8.2 Motif recipes

| Motif | Recipe |
|---|---|
| whistle | pure tone with a gentle pitch contour, 60–400 ms, attack 8 ms, release 40 ms |
| rise / fall | whistle with exponential glide up or down (ratio 1.1–1.6) |
| trill | whistle with 20–40 Hz frequency and amplitude modulation, 120–500 ms |
| chip | 20–60 ms band-passed noise burst plus a sine blip |
| burr | low-passed noise with 30–60 Hz amplitude modulation, 150–400 ms |
| purr | low sine with slow AM, 300–700 ms |
| double | two whistles 60–90 ms apart, second slightly higher |
| churr | sustained burr, 0.6–3 s (night-active species) |

### 8.3 Signature and grammar

Species grammars (initial): warbler-like `rise (trill | whistle){1,3} fall?`; finch-like `chip{2,5} whistle?`; thrush-like `whistle whistle+ pause whistle`; dove-like `purr purr fall`; wren-like `trill trill burr`; nightjar-like `churr{1,3}` at night, a single `chip` by day.

Per-bird signature (fixed at arrival, stored, immutable): `pitch_center` within the species range (±15 %), `tempo` 0.85–1.2, `brightness` 0.3–0.8, `motif_weights` (Dirichlet over the species motifs), `ornament_p` 0.05–0.3, `rhythm` (a fixed inter-motif gap pattern). The distance rule at arrival: normalized feature distance to every existing bird ≥ 0.35 and pitch centres ≥ 2 semitones apart; a species repeat (only after all six are present) must differ in at least three features. The signature dominates timbre and rhythm, so the bird stays recognisable while mood shapes phrase length, level, and pitch offset within narrow bands.

Phrase generation is symbolic and lives in the engine: `(species grammar, signature, mood, seed) → phrase = [motif, params, gap]…`. Variation per call: seeded motif choice, ±10 % timing, ±3 % pitch per motif, ornament insertion, small dynamics. The same seed yields the same phrase on every device; the no-duplicates test (T9) guards against the "same call twice" failure.

### 8.4 Mood and personality shaping

| Mood | Shaping |
|---|---|
| wary | 1–2 motifs, +8 % pitch, faster attack, −4 dB |
| content | full phrase, relaxed tempo |
| curious | rising endings, more ornaments |
| drowsy | 0.8× tempo, −6 dB, lower brightness, fewer motifs |
| alert | single sharp motif |
| roosting | none (rare soft rustle from the noise path) |

Personality reaches audio only through the server: vocal sets the call rate and chorus participation; warm sets callbacks. The client never needs the numbers.

### 8.5 Scheduling

Timeline `call` and `callback` items carry server times; the engine converts to `AudioContext.currentTime` using the clock offset. A lookahead scheduler (250 ms lookahead, 50 ms timer; 2 s lookahead when the tab is hidden and timers are throttled) writes envelopes with `setValueCurveAtTime` from pooled arrays. Items more than 2 s in the past when encountered are dropped, so a resumed laptop never plays a burst of catch-up calls. Greeting and offer phrases are scheduled the same way from their plans.

### 8.6 Mixing

Ambient per-bird level 0.7 scaled by depth (back 0.6, middle 0.8, front 1.0). Listen-in: focused bird → 1.0 over 1.5 s, others → 0.25 over 1.5 s; disengage returns everyone to ambient over 2.5 s; the floor is 0.2 and there is no mute path. Chorus: overlapping calls receive per-voice detune (±6 cents) and micro-timing (±30 ms) so partials never phase-lock; pitch-centre separation at arrival prevents comb-filter artefacts between birds. Night trims the master by 6 dB; settle by 8 dB; rain adds the rain bed and dampens birds through the server's `vocal_damp`; wind swells the ambient bed. Overall loudness targets a quiet room, not a media player.

### 8.7 Background tabs

Rendering stops when the tab is hidden; audio continues at ambient level using the improviser (a tab left open for the sound is a window left open). The account setting `audio` offers `on`, `visible_only`, `off`; default `on`. With `visible_only`, the master ramps to zero over 2 s on hide and back on show.

### 8.8 Autoplay lock and the WebAudio fallback

If `AudioContext` cannot be constructed, or its state stays `suspended` after a user gesture, or the hardware errors, the aviary plays in silence with captions forced on (the setting is unchanged; the forced state ends if audio recovers). There is no recorded-audio path anywhere; the asset lint fails the build on any audio file.

Autoplay policy is the common case of the same state: on open, the engine calls `resume()`; if the context is still suspended (no prior activation on this origin), the aviary runs silent with captions shown until the first pointer or key gesture, which resumes the context and ramps the bus in over 1.5 s while captions return to the user's setting. The first session unlocks naturally during adoption (typing names). Returning users on Chrome with a media-engagement history get sound immediately; others get it on the first click on empty scene space, which is otherwise a no-op. Nothing on screen asks for the click.

### 8.9 Song-fragment offer

Six motifs in a "user" timbre (sine-only, softer, a fifth below the species range) play once at −12 dB from a dedicated voice; the birds' responses come from the reaction plan and the normal call path.

### 8.10 Quality tests

Golden phrase generation per seed; the 10 000-phrase no-duplicate test; the signature distance test at seven birds; an `OfflineAudioContext` render of two overlapping calls with a spectral-flatness assertion against sustained comb-filter notches; a click-and-pop test (no discontinuity above threshold at envelope boundaries); CPU test: 14 voices plus rain under 3 % of one core on the reference laptop profile; node-count stability over a 30-minute run. In beta, a listening panel runs an identification study (target: participants name the calling bird ≥ 80 % of the time at four birds and ≥ 70 % at seven after two weeks of use); the seven-bird cap and ladder pacing (§11.4) depend on this passing before any cohort reaches those counts.

---

## 9. Accessibility surfaces

### 9.1 Screen-reader narration

An `aria-live="polite"`, `aria-atomic="true"` region outside the canvas receives prose composed on the client by `@aviary/prose` from the same snapshot and timeline the renderer uses. Each utterance is a scene line (phase, light, weather) plus up to two bird lines chosen by salience (most recent event, focused bird, changed mood or perch), with anti-repetition memory over the last 20 sentences. Cadence: idle every 45 s ± 15 s; event bumps for the greeting (within 1 s of the first frame), offer reactions, settle, arrivals, and weather onset; a single-slot queue where the latest wins; minimum 8 s between utterances; never `assertive`. A `narration_pace: slower` setting stretches idle cadence to 90–120 s. `narration_text: on` mirrors the same line as quiet text under the scene for sighted users who want it. Example utterance the composer produces:

> a small grey bird is perched on the front rail, calling softly. wren sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Narration sentences pass the same voice lint as notebook entries. Bird names are spoken as given; unnamed birds are described by species.

### 9.2 Focus and keyboard

`FocusLayer` renders a `role="group" aria-label="aviary"` container over the canvas with one `<button>` per bird using roving `tabindex`, positioned over each bird's bounding box (updated at ≤ 4 Hz) and visually hidden except for the focus ring. `aria-label` is a naturalist descriptor: "pip — a small grey bird on the front rail, preening". Keys: Tab into the group focuses the first bird; arrow keys move between birds; Enter or Space toggles listen-in; Escape ends listen-in and returns focus to the group; Tab leaves to the top bar (account, accessibility, notebook, offer, in that order). Sheets trap focus, close on Escape, and return focus to their opener. The offer sheet is a radio group (seed, song with a motif list, pool, settle). Listen-in disengages on focus leaving the bird, as the spec requires, with the same slow ramp.

### 9.3 Captions

Enabled from accessibility settings (and forced during audio-lock or fallback). Each call's caption is composed at schedule time from the phrase the engine generated, not from a stored string: motif sequence and shaping become prose ("a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch"). Rendered as a small DOM caption near the bird's head, fading in over 150 ms, held for the phrase duration plus 800 ms, fading out over 400 ms; at most three visible, stacked when birds are close; `aria-hidden` because narration already covers calls for screen readers. A fixed-lightness scrim behind caption text guarantees AA in every lighting phase.

### 9.4 Reduced motion

§7.7. The mode is selected by `prefers-reduced-motion` or the account setting, and it carries every feature: calls, captions, greeting, offers, notebook, narration, drift. Design signs off on it as a surface with its own screenshots in the design system.

### 9.5 Contrast and focus ring

All copy on chrome ≥ 4.5:1; icon glyphs ≥ 3:1 at full opacity; the fade only applies when the bar is not interactive. Focus ring: 2 px dark inner outline plus 2 px light outer outline, ≥ 3:1 against every scene state; the exact colours come from the design system tokens, and CI checks the token pairs against each lighting phase's background samples.

### 9.6 Settings surface

The accessibility sheet is a system surface in the matter-of-fact voice: Captions (Off / On), Motion (Match system / Reduced / Full), Narration text (Off / On), Narration pace (Normal / Slower), Audio (On / Only while visible / Off). Settings persist on the account and apply on every device.

### 9.7 Testing

Per release: manual passes with VoiceOver + Safari (macOS and iOS), NVDA + Firefox, NVDA + Chrome, JAWS + Chrome, TalkBack + Chrome; a scripted keyboard-only walkthrough (open, greet, listen in, offer, settle, notebook, settings, sign out); axe-core on every sheet; a focus-layer snapshot test; narration and caption corpora through the voice lint; reduced-motion visual regression; live-region cadence test (no more than one utterance per 8 s, none marked assertive). Beta includes research sessions with screen-reader and reduced-motion users; their findings are launch-gating (§11.3).

---

## 10. Performance budgets and observability

### 10.1 Budgets and CI gates

| Budget | Value | Gate |
|---|---|---|
| Initial JS, gzipped, at first paint | ≤ 2 MB hard; alert > 700 KB; target ≈ 230 KB | size check on every PR |
| Time to first bird | < 500 ms from `navigationStart` to `aviary:first-bird` on a mid-tier phone profile over 4G | synthetic in CI (Chrome, 4× CPU slowdown, 75 ms RTT / 9 Mbps edge-realistic profile, cold and warm-cache runs); RUM p75 in production |
| Idle motion | 60 fps for 30 minutes on a five-year-old mid-range laptop | lab test on a reference machine profile (4× CPU throttle): ≥ 99 % of frames < 16.7 ms over a 60 s window, sampled every 5 minutes for 30 minutes |
| Memory | no growth over 30 minutes | headless soak: JS heap after forced GC at minute 30 ≤ minute 5 + 2 MB; DOM node and audio node counts flat; nightly, plus a 10-minute variant on PRs touching renderer or audio |
| Audio CPU | 14 voices plus weather ≤ 3 % of one core | lab test |
| Tick latency | p50 ≤ 30 ms; p99 alarm at 5 s | production alert |
| Tick lag (`now − next_tick_at`) | p99 ≤ 30 s; alarm at 120 s | production alert |
| Snapshot API | p95 ≤ 120 ms at origin | production SLO |
| Events ingest / offers | p95 ≤ 80 ms / ≤ 150 ms | production SLO |
| Magic-link email | p95 ≤ 60 s from request to provider accept | production alert |
| Availability | 99.9 % monthly on snapshot and events | SLO |

### 10.2 How each budget is met

- **Bundle.** Vector part data instead of bitmaps; procedural audio and reverb; one small UI library for chrome only; on-demand chunks for everything behind the top bar; no third-party scripts; a size budget file that PRs must update deliberately.
- **First bird.** Edge-served shell with inlined snapshot and inline first-frame drawing (§7.2); HTTP/3 with TLS 1.3 and 0-RTT resumption; `modulepreload` for the four core chunks; service-worker shell for repeat visits; no font or CSS on the critical path beyond the inlined critical CSS; the tick keeps the snapshot cache warm so the edge never waits on the database.
- **60 fps.** Canvas 2D with cached static layers, ≤ 12 draw calls per bird, bounded particle pools, no per-frame allocation, DPR cap 2, reduced work at portrait phone sizes.
- **Memory.** §7.11 pools and reuse; envelope arrays and audio nodes created once; windowed notebook; explicit teardown verified by tests.
- **Tick.** Bounded per-aviary queries; sub-stepped coarsening for idle aviaries behind a flag; partition leases sized from tick lag.

### 10.3 Server observability

Metrics (Prometheus/OpenTelemetry): tick latency, tick lag, ticks per second, CAS conflicts, events ingested per second, unconsumed event backlog, snapshot and events request latency and error rates, cache hit ratio at the edge, magic-link request/consume counts and failure reasons, mail outbox depth and delivery latency, invite counts, export and purge job durations, DB replication lag. Traces on API requests and ticks with the aviary UUID as the only identity attribute. Structured logs with redaction (§2.5).

Alerts: tick p99 > 5 s; tick lag p99 > 120 s; CAS conflict rate > 1 %; unconsumed backlog age > 10 min; snapshot 5xx > 0.5 %; mail delivery p95 > 60 s or outbox age > 5 min; edge cache miss ratio > 20 %; purge or export job failure; replication lag > 30 s.

### 10.4 Client telemetry and the privacy boundary

Real user monitoring is aggregate-only and sampled at 10 %. Each beacon carries: metric name and value, app version, browser family, coarse device class, coarse region (from the edge, not from the client), and nothing else — no cookie, no session, no account id, no bird data. Metrics: page load timings, first-bird time, frame-time histogram buckets, long-task counts, snapshot pull latency, audio-context state and error counts, caption and narration enabled flags (aggregate counts of feature use only), reduced-motion active flag, service-worker hit flag. The schema is a checked-in closed list; adding a field requires a privacy review recorded in the schema file.

Synthetic checks: a browser fleet in six regions runs the aviary on a schedule every 10 minutes against a dedicated synthetic account, measuring first-bird, frame timing, audio start, and snapshot latency.

Enforcement: the observability account has no route to the simulation database; the RUM collector accepts only the closed schema and drops unknown fields; the log pipeline drops any email-shaped field; a launch-gate negative test confirms no analytics credential can connect to Postgres.

### 10.5 What we deliberately do not measure

Visit frequency per account, streak-like sequences, retention cohorts keyed to aviary state, drift distributions across accounts, offer or listen-in counts per account, per-bird anything, visitor identities in aggregate, and "engagement" of any flavour. Session-duration histograms are collected without an account dimension, as the PRD allows, and that is the extent of behavioural measurement. Calibration questions are answered by the harness and by opt-in research, not by production data.

---

## 11. Rollout

### 11.1 Workstreams and milestones

Team of seven: engine and backend (2), scene rendering (2), audio (1), accessibility and chrome (1), infrastructure and SRE (1), with a visual designer and a sound designer as partners. Sixteen weeks to launch.

| Milestone | Weeks | Deliverables | Exit criteria |
|---|---|---|---|
| M0 Foundations | 1–3 | `@aviary/engine` and `@aviary/prose` with voice lint; calibration harness; schema, roles, migrations; magic-link auth and sessions; tick workers with lease and CAS; snapshot and events API; CI with size, lint, and PII scanner | T1–T10 green on initial values; golden vectors pinned; two-worker race test passes |
| M1 Vertical slice | 3–7 | Canvas renderer with two species; rig and idle motion; timeline executor; boot path with inlined snapshot; presence tracker; listen-in; audio engine with two grammars; captions; narration v1; top bar; notebook UI; basic settings | first-bird and 60 fps budgets met in lab; 10-minute soak flat; keyboard walkthrough passes |
| M2 Complete | 7–10 | All six species (art, grammars); offers; settle; weather; ladder and arrivals; reduced-motion renderer; full focus and keyboard; visits; export; deletion and restore; email change; service-worker shell; unsupported-browser surface; RUM, synthetics, dashboards, alerts; runbooks | all invariants I1–I11 have a passing automated check; internal dogfood begins week 8 |
| M3 Private beta | 10–14 | ~200 invited accounts; screen-reader and reduced-motion research sessions; audio identification panel; tick-fleet load test at 50× beta load; deliverability check; security review | launch gates (§11.3) satisfied or explicitly waived by the product owner |
| M4 Launch | 14–16 | public sign-up; on-call rotation; weekly ops review | budgets green in production for 7 days |

### 11.2 Environments and flags

Development (local Postgres, mail to a sink), staging (production-shaped, synthetic accounts, synthetic fleet pointed at it), production. Server-side flags with instant effect: `visits.enabled`, `weather.enabled`, `ladder.paused`, `tick.coarsen_idle`, `notebook.token_refill_hours`, `greeting.rate_limit_s`, `audio.background_default`, `captions.during_audio_lock`, `offer.spacing_s`. No flag exposes or alters the invariants in §0.

### 11.3 Launch gates

1. Hard budgets green in CI and in synthetic runs from all six regions.
2. Harness T1–T10 green; engine golden vectors unchanged since beta start or re-pinned with review.
3. Screen-reader matrix (§9.7) passes; reduced-motion parity signed off by design; findings from beta research sessions resolved or explicitly accepted.
4. Voice lint at 100 % on notebook, narration, caption, and adoption corpora; system copy reviewed.
5. Privacy: seven consecutive days with a clean log scanner; negative connectivity test from the observability account; RUM schema review; retention purge verified on staging.
6. Security: auth, session, visitor-token, and export-link review; rate limits exercised; invite abuse test.
7. Deletion (soft, restore, hard) and export exercised end to end on staging with a real mail provider.
8. Kill switches tested: visits, weather, ladder, notebook, background audio.
9. On-call runbooks reviewed by someone who did not write them.

### 11.4 Ramping birds per aviary

The ladder is age-based, so birds per aviary ramps itself: at launch every public aviary has two birds; the earliest public aviaries reach the third-bird threshold about 75 days after launch, and the seventh bird is more than 18 months out. Internal dogfood aviaries (created week 8) cross each threshold roughly 11 weeks before public ones, and beta aviaries about 6 weeks before, which makes them the canaries for every count. Before any cohort crosses a threshold, the audio identification study must pass at that count in the lab; if it does not, `ladder.paused` holds arrivals for everyone with nothing announced (no promise was made), and the audio team revisits signature distance and mixing. The cap of seven is a constant in the engine, not a flag.

### 11.5 Instrumented from day one

Tick latency and lag; CAS conflicts; event backlog age; snapshot and events latency and errors; edge cache hit ratio; first-bird time (synthetic and RUM p50/p75/p95); frame-time histograms; long tasks; memory soak results; audio-context errors and locked-state rate; caption, narration, reduced-motion feature-use counts (aggregate); magic-link funnel (requested, delivered, consumed, expired, replayed); mail latency; invite, visit, revocation counts (aggregate); export and deletion job outcomes; log-scanner hit count (should be zero).

### 11.6 Operations

Runbooks: tick lag (scale workers, check DB), CAS conflict spike (lease clock skew), snapshot cache miss (tick cache writes), mail provider outage (the sign-in surface says "We couldn't send the link right now. Try again in a few minutes."), DB failover, edge key rotation for the cookie claim, HMAC and KMS key rotation (blind index re-keying by batch job), invite abuse, RUM regression triage, purge or export job failures, incident communication (matter-of-fact voice, no in-product announcement).

---

## 12. Risks

| Risk | Likelihood / impact | Mitigation | Early signal |
|---|---|---|---|
| Drift calibrated too fast (birds change between sessions) or too slow (nothing seems to matter) | medium / high | harness targets T1–T3 with initial weights; daily cap; beta diary research; weights are server constants adjustable without a client release | beta interviews report "changed overnight" or "no different after a month" |
| Presence inflation (tab-open counted as watching) | medium / high | three-signal tracker, server union and clamp, T5; tab-left-open profile in harness | presence minutes per session approaching wall time in the harness; never measured per account in production |
| Multi-device double counting | medium / medium | union across sessions at tick; multi-device profile in harness | CAS or union bugs surface as T5 failures |
| Tick applied twice or lost (double drift, lost events) | low / high | lease plus CAS, single transaction, idempotent event ids, race test in CI | CAS conflict metric; backlog age |
| Neglect accidentally reads as punishment (mood settling into wary, greetings vanishing) | medium / high | expressiveness floor 0.5; wary prior modulated by bold; T6 | harness T6; research sessions after a planned absence |
| Procedural calls sound synthetic or robotic | high / high | sound designer on the motif recipes; harmonic and noise blends; reverb "place"; per-call variation; beta listening panel | panel feedback; audio setting turned off in aggregate more than expected |
| Phase-lock or comb-filter artefacts in chorus | medium / medium | pitch-centre separation, detune and micro-timing, offline render test | spectral flatness test |
| Recognisability collapses at 5–7 birds | medium / high | signature distance rule; identification study gates each ladder count; `ladder.paused` | study results below target |
| Autoplay lock leaves first-time users silent and confused | high / medium | captions during lock; adoption flow unlocks naturally; empty-space click unlocks | aggregate locked-state rate; support contact |
| First-bird budget missed on real networks | medium / high | edge shell with inlined snapshot; service-worker shell for repeats; synthetic from six regions; RUM p75 | synthetic and RUM regressions per release |
| Frame drops or memory growth on old laptops over long sessions | medium / medium | pools, cached layers, soak test in CI, reference-profile lab test | soak and frame tests |
| Narration too chatty, or state-list in tone | medium / high | cadence caps and single-slot queue; voice lint; screen-reader research sessions | live-region cadence test; research |
| Reduced-motion mode shipped as a stripped fallback | medium / high | separate renderer with design sign-off; feature parity checklist | parity review |
| Contrast failures on evening and night palettes | medium / medium | fixed-lightness scrims; token checks per phase in CI | token CI |
| Announcement surfaces creep in ("just a small toast") | high / high | component allowlist, forbidden identifiers, copy import boundaries, review checklist | lint failures in PRs |
| PII leaks into logs or metrics | medium / high | synthetic UUID everywhere, redaction, scanner in CI and daily, closed RUM schema | scanner hits |
| Magic-link deliverability | medium / high | reputable provider, domain authentication (SPF, DKIM, DMARC), outbox retries, funnel metrics | consume rate drop |
| Invite emails used for spam | low / medium | per-account and per-recipient limits; revocation; provider complaint monitoring | complaint rate |
| Timezone edge cases (DST, travel, two devices in two zones) | medium / low | recency rule; DST tests; phase computed from local time each tick | harness timezone tests |
| Engine version skew between client and server | medium / low | version in snapshot; server-expanded fallback; bundle reload at idle | version-mismatch metric |

---

## 13. Decision log (ambiguities resolved)

| # | Decision | Rationale and guard |
|---|---|---|
| D1 | Backend in TypeScript sharing `@aviary/engine` with the client | one implementation of drift, mood, grammar, and choreography; golden vectors prove parity |
| D2 | Tick cadence 60 s for every aviary, with flag-gated idle coarsening to 300 s implemented by 60 s sub-steps | the aviary always continues server-side; coarsening is a cost lever that is provably equivalent (sub-step test) |
| D3 | Greeting computed at snapshot time by the API and recorded as a server event | absence length is known precisely server-side; the notebook needs the greeter; clients never choose behaviour |
| D4 | Offers return a server-computed reaction plan; the tick recomputes the same plan from the event seed | single writer preserved while reactions are immediate |
| D5 | Expressiveness `E` from the decaying attention reservoir explains "quieter after absence" | traits never decrease (I2); quietness comes from reservoir decay, which recovers |
| D6 | Mood set `alert, curious, content, wary, drowsy, roosting` | "settled" is reserved for the aviary lighting state; `roosting` is the bird's night state |
| D7 | Presence activity window 4 min; heartbeat 20 s; flush 60 s | lean long, as the PRD asks; T5 keeps it honest |
| D8 | Multi-device presence is unioned, not summed | presence can never exceed wall time |
| D9 | Personality never appears in the snapshot; `plumage_step` (0–9) is the only trait-derived client value | I4 as a schema property; export is the PRD's named exception |
| D10 | Settle is reached through the offer affordance's sheet as a fourth, visually separated gesture | the top bar must hold exactly four icons and settle must be reachable from the top bar; settle is a gesture the user gives the aviary, so it sits with the other gestures |
| D11 | Settled lighting is device-local; the `settle` event is sent only after the 5 s undo window | undo is free; the server sees a clean end of presence and a mood-quieting signal |
| D12 | Background tabs stop rendering but keep ambient audio by default; a setting offers `visible_only` and `off` | a window left open still sounds like a window; presence is unaffected because visibility fails |
| D13 | Autoplay-locked audio is treated as the fallback state: silence plus captions until the first gesture | consistent with the PRD's own fallback; no "enable sound" announcement |
| D14 | Canvas 2D renderer with a rig and vector parts, behind a `SceneRenderer` interface | seven birds do not need WebGL; least variance on old laptops; WebGL remains possible later |
| D15 | Ladder at 75 / 150 / 240 / 365 / 540 days, arrivals land the next local morning | matches "a few months" for the third bird and five or six by a year; morning arrival gives the notebook and the user the same moment |
| D16 | Arrivals are full birds from the moment they land; naming is optional and quiet (first listen-in or the settings bird list) | no nagging, no offer flow that reads as a reward; identity and drift begin at arrival |
| D17 | Starters are sampled with a boldness gap ≥ 0.15 | a legible bolder-and-warier pair makes greet-first order readable from day one |
| D18 | Visitor one-time link becomes a 30-day cookie session on the first browser that uses it | invites can be "active" and revocable per the PRD; a second browser cannot reuse the link |
| D19 | Visit notification, when enabled, is at most one email per visitor per 6 h | the toggle exists per the PRD; the cap prevents it from becoming a feed |
| D20 | Account timezone follows the most recently active device | the aviary shares the day of the device the user is on |
| D21 | Names and settings use last-write-wins; nothing else is client-writable | the no-LWW rule is about personality; these fields have no drift to lose |
| D22 | Event rows are purged 7 days after consumption | the PRD stores interaction events only to drive the simulation; a week covers tick replay and debugging |
| D23 | Idle coarsened aviaries tick immediately on the first snapshot request | the returning user's greeting and state are computed against fresh state |
| D24 | The blind index (HMAC of email) lives only on the account record for sign-in lookup | required for magic-link sign-in without decrypting every row; it is not an identifier and never leaves the table |
| D25 | Mobile presence lapses after four minutes without a touch | this is the PRD's definition; the long window is the mitigation, and the harness pins it |
| D26 | Notebook entry sparsity via a token bucket (capacity 2, refill 72 h) plus one-per-day and filler after five quiet days | matches "one entry every few days, more when something happens"; T8 guards it |
| D27 | Research on "felt" drift is opt-in and qualitative, never production interaction data | I7; calibration comes from the harness |

---

## 14. Definition of done for v1

Every invariant in §0 has an automated check that fails CI or pages on-call. Every launch gate in §11.3 is green or explicitly waived in writing. The first frame a user sees after sign-in is the aviary, already moving, in under half a second on the reference mobile profile. A screen-reader user, a reduced-motion user, and a user with audio locked each get the aviary in the product's own voice, not a description of it. Two devices show the same birds in the same moods. A user who leaves for two weeks comes back to birds that are quieter, not sadder. And nothing, anywhere, announces.
