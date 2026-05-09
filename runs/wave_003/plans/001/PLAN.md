# Pocket Aviary — v1 Implementation Plan

This plan turns the PRD into an executable build for a frontier engineering team. It is organized so each section maps to one PRD concern; sections cite the PRD constraints they inherit and then commit to specific implementation choices, defaults, and calibration targets. Where the PRD leaves something open, a defensible call is made and flagged as a calibration target. Nothing in this plan compromises the five design principles (feels alive, notice-never-announce, charm-from-specificity, restraint-over-richness, naturalist-vs-matter-of-fact voice split) or the explicit non-goals.

This plan is a plan, not an implementation. Code shapes shown below are illustrative type sketches — they exist to pin shape, not to be copy-pasted as v1 code.

---

## 1. Scope

### In v1

- Single-user accounts, magic-link sign-in, per-device session tokens, account export, soft-then-hard delete.
- One canonical aviary per account, single horizontal scene, two starter birds, age-gated ramp toward seven max.
- Six-species pool with stable bird identities, user-assigned names, server-canonical personality vectors.
- Server-side simulation tick (~60s cadence) that advances mood, drift, ambient state, and notebook generation independent of client connectivity.
- Procedural call synthesis client-side via WebAudio with chorus mixing, listen-in mix, per-call captions.
- Procedural visual scene with three perch zones, day/night cycle anchored to user local time, ambient weather (rain, wind), foreground/background parallax, ambient leaf/feather drift.
- Five user-facing interactions: greeting (system-initiated on session start), listen-in, offer (seed, song fragment, still pool), settle, browse field notebook.
- Offer affordance reachable from a thin top bar; field notebook reachable from top bar; settle reachable from top bar.
- Visit invitations (per-invite opt-in, revocable, 30-day expiry, host-only revoke surface, silent default with opt-in notification).
- Multi-device sync as a property of the canonical-server architecture.
- Accessibility: screen-reader narration in naturalist voice, captioned calls, keyboard navigation, focus indicators, WCAG AA contrast, designed reduced-motion mode (not stripped fallback).
- Performance: <2MB initial gzipped JS, <500ms time to first bird on mid-tier mobile / 4G, 60fps idle on a five-year-old laptop, no-memory-growth invariant in CI for 30-minute sessions.
- Browser support: last two major versions of Chrome, Safari, Firefox, Edge.

### Out of v1 (non-goals respected at the architectural level, not just the surface)

- Native mobile apps. The protocol does not include native-only fields; we will not pre-shape the API for a future native client at the cost of clarity now.
- Gamification of any flavor. **Architectural enforcement**: the data warehouse is forbidden from receiving per-account dimensions; aggregate analytics cannot reconstruct streak-style metrics because the columns to do so do not exist. Refusing to compute the underlying counters makes adding a streak counter later require new infrastructure, not just a UI change. This is the cost we pay to keep the rule durable.
- Tamagotchi mechanics. **Engine-level enforcement**: the drift function clamps deltas to non-negative; there is no negative-drift code path to flip on later.
- Social network surfaces beyond visit invitations. The visitor session is render-only, with no event emission; there is no presence-time accounting for visitors and no field where a visitor identity could be displayed inside the host's aviary.
- Push notifications, email digests, friend-visited notifications by default. Email is reserved for transactional auth (magic link, export, deletion confirmation), visit-notification opt-in if enabled, and account-state errors. Marketing email is not a v1 surface.
- Stat panels, debug views, or any numeric exposure of personality vectors to the user. The values are not in any UI surface, including hidden ones; the API does not return them as numbers in client-bound payloads — the client receives only **rendered visual signals** derived from the vector (perch bias, plumage saturation level, idle animation parameters), never the vector itself.

### Defensible calls (made because the PRD leaves them open)

- **Stack**: TypeScript everywhere. Frontend chrome (top bar, settings, notebook, settle, offer affordance, accessibility surfaces) in Preact for bundle size; aviary scene rendered to a single `<canvas>` element via a custom Canvas2D renderer; audio in WebAudio with a custom DSP scheduler. Backend in Node 22 (TypeScript) for shared types with the client and for simpler model parity at engineering velocity. Postgres 16 for canonical state; Redis 7 for ephemeral hot caches and SSE fanout. Object storage (S3-compatible) for export bundles.
- **Edge**: Cloudflare Workers (or equivalent) host the HTML shell and the small initial state snapshot, served from edge cache. The same edge handles magic-link emission and short-lived signed token verification so the first paint can occur without a round-trip to origin.
- **Origin**: a small set of services behind one HTTP API gateway. Services are colocated in one repository (monorepo) and share a common types package.
- **Database**: Postgres for everything personality- and account-related. We do not introduce a separate document store; the data shape is well-suited to relational integrity and the velocity gain of a single store outweighs theoretical scaling wins of a polyglot persistence layer at this size.

---

## 2. Architecture

### 2.1 High-level service shape

Five logical services. They run as separate processes (containers) but share a deployment pipeline:

1. **edge** — HTML shell, static asset serving, initial-snapshot delivery, magic-link issuance/verification redirects.
2. **api** — REST + SSE endpoints for clients: state snapshots (read), event ingestion (write), visit invitation flow, account settings, narration stream.
3. **sim** — the simulation tick engine. A scheduler enqueues per-account tick jobs at cadence; a worker pool consumes and executes ticks. Sole writer of personality state.
4. **narrator** — generates field-notebook entries and screen-reader narration prose. Reads canonical state and the recent event window; writes notebook entries (rare) and narration phrases (per-aviary stream). No personality writes.
5. **mailer** — outbound transactional email (magic link, export ready, deletion confirmation, visit invitation, account export).

`sim` and `narrator` are the only services with write access to canonical state tables that hold personality vectors; even `narrator`'s writes are constrained to its own tables (notebook entries, narration phrases). `api` writes only to the event log and account-mutation tables (names, settings, visit invitations, sessions). The split is enforced at the database role level (separate Postgres roles per service with column-level grants).

```
[browser] ──(HTML+CSS+JS+initial snapshot inline)── [edge / CDN]
       │
       ├─ GET  /api/aviary/snapshot          ──► [api]  ─reads─► [postgres canonical]
       ├─ POST /api/aviary/events            ──► [api]  ─appends──► [postgres event log]
       ├─ GET  /api/aviary/stream  (SSE)     ──► [api]  ─subscribes─► [redis pubsub]
       ├─ GET  /api/notebook                 ──► [api]  ─reads─► [postgres notebook]
       ├─ GET  /api/narration/stream (SSE)   ──► [api]  ─subscribes─► [redis pubsub]
       ├─ POST /api/auth/magic-link          ──► [api]  ─enqueues─► [mailer]
       ├─ GET  /api/auth/verify              ──► [api]  ─sets cookie──► browser
       ├─ POST /api/visits/invite            ──► [api]  ─enqueues─► [mailer]
       ├─ GET  /api/visit/:token             ──► [api]  ─visit-only snapshot
       └─ DELETE /api/account                ──► [api]  ─soft-delete

[scheduler tick] ──► [sim worker] ─reads events, writes personality+mood──► [postgres]
                                                  │
                                                  └──► [redis pubsub: snapshot updated]
[narrator scheduler] ──► [narrator worker] ─reads state, writes notebook+narration──► [postgres+redis]
```

### 2.2 Client/server split (load-bearing)

The split is the architectural realization of "the aviary continues without the viewer."

- **The server is the sole writer of canonical state.** Personality vectors, mood, position, weather, day/night phase. Mutations only via `sim`.
- **The client is the sole renderer.** The client owns interpolation between snapshots, the audio synthesis, the visual rendering, the accessibility surface presentation, the listen-in mix, and the local-time mapping for any UI moments the server doesn't author.
- **Events flow client → server append-only, never as state mutations.** The client emits "user listened in to bird X for Y ms," "user offered seed near bird X at time T," "presence ping." It never emits "set boldness to 0.62" or anything that names a personality attribute by value.
- **State flows server → client as immutable snapshots.** Snapshots include rendered visual signals derived from personality (e.g., a perch-bias parameter in [0,1], a plumage-saturation level, a mood enum) but never the raw personality vector.

The render-pipeline boundary lives at the snapshot. Above it, the server thinks in personality and mood; below it, the client thinks in positions, opacities, and audio events. The client never sees a number called "boldness" — it sees a perch-bias, which is computed server-side from boldness plus mood.

### 2.3 Why this is correct (and what it forecloses)

Putting the simulation tick on the server is the difference between "the aviary continues without the viewer" being a real property and being a label. With a server tick, two devices reading the same canonical state see the same aviary; a user closing the laptop and opening the phone sees an aviary that has been continuing, not one that has been frozen. Without it, every multi-device session becomes a merge problem the client can't solve correctly.

Forbidding clients from writing personality directly is what forecloses last-write-wins corruption. A user with two devices can have two simultaneous sessions; both emit events; the server processes them in event-log order and applies additive deltas; nothing is lost. The architecture does not let last-write-wins exist as a code path, so it cannot become a regression.

---

## 3. Data model

All identifiers are synthetic UUIDv4. Email lives only on the `accounts.email_encrypted` column, encrypted with a column-level key from KMS. No other table references email; foreign keys are on `account_id` (UUID).

### 3.1 Tables

#### `accounts`
| column | type | notes |
|---|---|---|
| `id` | UUID | primary key, synthetic |
| `email_encrypted` | bytea | KMS-encrypted email; only column referencing the user's address |
| `email_lookup_hash` | bytea | HMAC-SHA256(email, server-key); indexed; used only for sign-in lookup |
| `created_at` | timestamptz | |
| `state` | enum(`active`, `pending_delete`, `deleted`) | |
| `deletion_scheduled_at` | timestamptz null | set when state→`pending_delete`, used by the daily hard-delete job |
| `settings` | jsonb | accessibility preferences, audio prefs, visit-notification opt-in |
| `local_timezone` | text | IANA tz; updated on session token issuance from client header |

#### `aviaries`
| column | type | notes |
|---|---|---|
| `id` | UUID | primary key |
| `account_id` | UUID | unique, foreign key to `accounts` |
| `created_at` | timestamptz | drives age-gated bird offer schedule |
| `current_weather` | enum(`clear`, `rain`, `wind`) | |
| `weather_until` | timestamptz | when the current weather state expires |
| `last_tick_at` | timestamptz | the server's record of the last tick processed for this aviary |

#### `birds`
| column | type | notes |
|---|---|---|
| `id` | UUID | primary key, stable across renames and migrations (engine-layer invariant) |
| `aviary_id` | UUID | foreign key |
| `species` | enum (six v1 species) | |
| `name` | text | user-assigned, renameable |
| `adopted_at` | timestamptz | |
| `personality_vector` | jsonb | `{ boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity }`, scalars in [0,1] |
| `mood` | enum(`wary`, `content`, `curious`, `drowsy`, `alert`) | |
| `mood_changed_at` | timestamptz | for cooldowns and transition pacing |
| `current_perch_zone` | enum(`front`, `middle`, `back`) | |
| `last_call_at` | timestamptz | for chorus pacing |

#### `events` (append-only)
| column | type | notes |
|---|---|---|
| `id` | UUID | primary key |
| `aviary_id` | UUID | indexed |
| `kind` | enum (see §3.3) | |
| `payload` | jsonb | shape varies per kind |
| `client_at` | timestamptz | client-reported event time |
| `server_at` | timestamptz | server-side receipt time; used for ordering |
| `processed_at` | timestamptz null | nulled until the next tick ingests it |

There is **one and only one** event log. The `sim` reads it in `server_at` order with a "tick window" cursor. No event is ever updated or deleted by anything other than the 30-day-old retention job (events older than 30 days are deleted to bound storage; tick-relevant aggregation has already been applied to canonical state).

#### `notebook_entries`
| column | type | notes |
|---|---|---|
| `id` | UUID | primary key |
| `aviary_id` | UUID | indexed |
| `written_at` | timestamptz | |
| `local_date` | date | the user's local date when the observation occurred; drives "tuesday — …" prefixes |
| `prose` | text | naturalist voice; lowercase; specific |

Entries are append-only from the user's perspective (no edit, no delete by the user). Operations may delete entries that violate voice (rare; see §11 for content audit).

#### `visit_invitations`
| column | type | notes |
|---|---|---|
| `id` | UUID | primary key |
| `host_account_id` | UUID | |
| `visitor_email_encrypted` | bytea | KMS-encrypted; never logged |
| `visitor_email_lookup_hash` | bytea | for revocation lookup if needed |
| `token_hash` | bytea | HMAC of the link token; the link is single-use until activated |
| `created_at`, `activated_at`, `revoked_at`, `expires_at` | timestamptz | |
| `state` | enum(`pending`, `active`, `expired`, `revoked`) | |

#### `visit_sessions`
| column | type | notes |
|---|---|---|
| `id` | UUID | |
| `invitation_id` | UUID | |
| `started_at`, `ended_at` | timestamptz | |
| `approximate_duration_seconds` | int | rounded to nearest 30s for the host's visit log |

A visit session **does not** write to the events table. It records its own session row only for the host's visit log. The simulation never reads `visit_sessions` for any drift input.

#### `session_tokens`
| column | type | notes |
|---|---|---|
| `id` | UUID | |
| `account_id` | UUID | |
| `device_label` | text | derived from User-Agent on issuance |
| `created_at`, `last_seen_at` | timestamptz | |
| `revoked_at` | timestamptz null | |

#### `magic_links`
| column | type | notes |
|---|---|---|
| `id` | UUID | |
| `email_lookup_hash` | bytea | |
| `token_hash` | bytea | |
| `expires_at` | timestamptz | 15 minutes from issuance |
| `consumed_at` | timestamptz null | one-time use |

### 3.2 Personality vector — exact ranges and seed

Five scalars, each in [0, 1]:

- `boldness`
- `social_warmth`
- `vocal_frequency`
- `plumage_saturation`
- `curiosity`

Seeds at adoption are drawn from a per-species distribution centered on a species-typical baseline ± a small jitter. The two starter birds in a new account are seeded with **complementary** vectors — one bird closer to bold/warm, one bird closer to wary/quiet — so the first encounter has visible per-bird difference even before drift starts. This is a defensible call: it makes the "two birds" surface read as two different birds rather than two neutral instances of the engine.

### 3.3 Event kinds

| kind | payload | drift weight (relative) |
|---|---|---|
| `presence_ping` | `{ idle_focused: true }` (sent every ~10s while presence holds) | 1.0 (dominant via accumulated time) |
| `listen_in_start` | `{ bird_id }` | — (paired with end) |
| `listen_in_end` | `{ bird_id, duration_ms }` | 1.5 per minute of focused listen-in (per bird) |
| `offer_initiated` | `{ kind: 'seed' \| 'song_fragment' \| 'still_pool', near_bird_id?: UUID }` | small toward boldness for the targeted bird |
| `offer_accepted` | `{ bird_id, kind }` | small toward curiosity for that bird |
| `offer_ignored` | `{ kind, bird_ids[] }` | none |
| `settle` | `{}` | none for drift; ends the presence window |
| `name_changed` | `{ bird_id, new_name }` | none for drift |
| `tab_visibility` | `{ visible: bool }` | client-side heartbeat the server uses to bound presence runs |
| `tab_focus` | `{ focused: bool }` | server uses to bound presence |
| `pointer_or_key_activity` | `{}` (rate-limited to once/30s) | server uses to bound presence |

The server determines presence time by intersecting these three streams (`tab_visibility`, `tab_focus`, `pointer_or_key_activity`) and integrating the conjunction into a presence-seconds quantity per tick. This is the engine-level realization of the PRD's three-condition presence definition. Keeping the conjunction on the server (rather than letting the client emit a single "I'm present" boolean) is what makes the signal honest — a malfunctioning or modified client cannot inflate presence by emitting that boolean prematurely.

### 3.4 Calibration constants (named so they can be tuned)

| name | initial value | rationale |
|---|---|---|
| `TICK_INTERVAL_SECONDS` | 60 | PRD says ~once per minute |
| `TICK_INTERVAL_IDLE_SECONDS` | 600 | for aviaries with no events in 24h |
| `PRESENCE_ACTIVITY_WINDOW_SECONDS` | 300 | five minutes is forgiving enough that "watching without moving" is the actual product, short enough that an abandoned tab is not counted indefinitely |
| `DRIFT_ALPHA_PER_HOUR_PRESENCE` | 0.0035 | ~30 hours of presence → ~0.10 absolute on a low-pass-filtered trait, which the design system maps to a perceptible plumage step |
| `DRIFT_ALPHA_PER_MINUTE_LISTEN_IN` | 0.0025 | listen-in is more concentrated attention than ambient presence |
| `DRIFT_ALPHA_PER_OFFER_ACCEPTED` | 0.005 | small per-event nudge |
| `DRIFT_TARGET_INSTRUMENT_HOURS` | 10 | "measurable drift in instruments after about a week" — a regular visitor at ~1.5h/day reaches ~10h/week |
| `DRIFT_TARGET_VISIBLE_HOURS` | 30 | "visible drift to the user after about three weeks" |
| `MOOD_DAILY_RESET_LOCAL_TIME` | 04:00 user-local | mood-resets-on-a-daily-ish-cadence; chosen as low-activity hour |
| `OFFER_COOLDOWN_PER_BIRD_SECONDS` | 240 | "a few minutes" |
| `LISTEN_IN_RAMP_MS` | 1500 | feels like listening, not switching |
| `SETTLE_UNDO_WINDOW_MS` | 5000 | per PRD |
| `BIRD_OFFER_AGE_GATE_DAYS` | `[30, 60, 120, 240, 365]` | third bird offered at 30 days; subsequent offers at compounding intervals so a year-old aviary can have ~five-six birds |

These are calibration targets, not literal constants in production code; each lives in a service config that can be tuned per environment, with version history. Calibration changes ship behind a feature gate so a tuning experiment can run on a cohort.

---

## 4. API surface

The API is HTTP/JSON for CRUD and command paths, SSE for state and narration streams. WebSockets are not used — there is no client-to-server push at high rate, and SSE is simpler to operate behind the CDN. All endpoints require a session token cookie (HttpOnly, Secure, SameSite=Lax) except auth, visit token, and unauthenticated marketing/landing paths.

### 4.1 Endpoint catalog

#### Auth

- `POST /api/auth/magic-link`
  Body: `{ email }`. Response: `204` always (no PII leak via response timing differences; rate-limited per `email_lookup_hash` and per source IP).
- `GET /api/auth/verify?token=...`
  Single-use. Sets session cookie on success. Redirects to `/` on success or `/sign-in?error=expired` on failure (matter-of-fact surface).
- `POST /api/auth/sign-out` — clears cookie, marks token revoked.
- `GET /api/account` — returns sanitized profile: timezone, settings, list of devices, list of outstanding visit invitations, list of recent visits.
- `POST /api/account/email/change` — body `{ new_email }`. Sends verification to new address; old address continues to work until verified.
- `POST /api/account/delete` — sets `state='pending_delete'`, schedules hard delete +30 days; returns confirmation copy in matter-of-fact voice.
- `POST /api/account/recover` — only valid when state is `pending_delete`; returns to `active`.

#### Aviary read path

- `GET /api/aviary/snapshot`
  Returns the current canonical aviary snapshot (see §4.2). Cached at the edge for 1s to absorb burst loads from a single device opening multiple tabs.
- `GET /api/aviary/stream` (SSE)
  Streams snapshot deltas at the simulation tick cadence and on event-driven updates (e.g., a bird transitions mood mid-tick because of a recent event). Heartbeats every 15s.
- `GET /api/notebook?cursor=...` — paginated notebook entries, newest first.

#### Aviary write path

- `POST /api/aviary/events`
  Body: `{ events: [...] }` — batched. Each event has `kind`, `payload`, and a client-generated `client_at` timestamp. Server stamps `server_at`. Accepts up to 50 events per request; rejects over-quota with 429.
- `POST /api/aviary/birds/:id/name` — body `{ name }`. Audited; logged for the user's own ability to recall renaming history.

#### Offer

- `POST /api/aviary/offer` — body `{ kind, near_bird_id? }`. Server checks per-bird cooldown; returns `204` on success. Reaction is computed by the next tick and reflected in the next snapshot.

#### Visit invitations

- `POST /api/visits/invite` — body `{ visitor_email }`. Issues invitation, enqueues mailer.
- `GET /api/visits` — host's invitation and visit log.
- `POST /api/visits/:id/revoke` — revoke; immediate effect on next snapshot pull on visitor's side.
- `GET /api/visit/:token` — visitor entry. Validates token, opens an unauthenticated visitor session (separate session model), redirects to a host-aviary read-only render.
- `GET /api/visit/:token/snapshot` — visitor-only snapshot endpoint. Returns the same shape as `/api/aviary/snapshot` but filtered through a visitor-render boundary (no notebook, no settings). Refused if invitation is `revoked` or `expired`.

#### Narration (accessibility)

- `GET /api/narration/stream` (SSE) — naturalist prose phrases at the configured cadence; for screen readers and for the captions surface when audio is off.

#### Export

- `POST /api/account/export` — enqueues an export job; emails a signed download link to the verified address when ready.

### 4.2 Snapshot shape

The snapshot is the contract between server and renderer. Keep it small (kilobytes) and free of personality vector values.

```ts
type Snapshot = {
  schema_version: number;          // for forward-compat; clients refuse unknown major versions
  taken_at: string;                // ISO 8601 server timestamp
  aviary: {
    age_days: number;
    weather: { kind: 'clear' | 'rain' | 'wind'; intensity: number /* 0..1 */; until: string };
    daypart: 'pre_dawn' | 'dawn' | 'morning' | 'midday' | 'afternoon' | 'dusk' | 'night';
    palette_phase: number;         // 0..1, drives day/night palette interpolation client-side
    settled: boolean;              // true when the user has invoked settle (server-tracked because cross-device)
  };
  birds: BirdRender[];
};

type BirdRender = {
  id: string;
  species: SpeciesId;              // enumerated, drives sprite/silhouette
  name: string;
  mood: 'wary' | 'content' | 'curious' | 'drowsy' | 'alert';
  current_perch_zone: 'front' | 'middle' | 'back';
  perch_index: number;             // 0..2 within the zone — fine-grained position
  visual_signals: {
    plumage_saturation_level: 1 | 2 | 3 | 4 | 5; // discrete steps; never the raw scalar
    perch_bias: number;            // 0..1, how strongly this bird is pulled to front; client uses for idle micro-motion
    head_tilt_propensity: number;  // 0..1
    preen_propensity: number;      // 0..1
  };
  call_schedule: CallSchedule;     // upcoming call windows, see §8
  last_call_at: string;            // for chorus pacing client-side
  greeted_this_session: boolean;   // for return-greeting narration cadence
};

type CallSchedule = {
  next_window: { earliest: string; latest: string };
  motif_id: string;                // selects a motif from the bird's call grammar
  pitch_shape: number[];           // sequence of relative pitches the client renders
  duration_envelope: number[];     // matching duration scalars
};
```

The snapshot deliberately exposes **levels** (1–5 for plumage saturation) rather than the underlying scalar. This is so a future debug build that prints the snapshot to a log file cannot resurrect numeric personality state. Plumage level transitions are visible to the user only as palette steps, not as smooth animations of a number.

### 4.3 Visit-invitation flow

```
host: POST /api/visits/invite { visitor_email }
   server: store invitation row (state=pending), generate token, sign HMAC,
           enqueue mailer (subject and body in matter-of-fact voice)
   server: respond 204
mailer: send email with signed link; link is single-use until activated, expires at +30d

visitor: GET /api/visit/:token
   server: verify HMAC, look up invitation, transition state to active if pending,
           record activated_at, mint visitor session cookie (separate cookie name, scoped to visit paths)
   server: redirect to /visit/:token (a separate frontend bundle entrypoint)

visitor: GET /api/visit/:token/snapshot
   server: confirm state is active; return host's aviary snapshot, filtered (no notebook, no offers, no settle)

host: POST /api/visits/:id/revoke
   server: set state=revoked, revoked_at=now
   server: publish redis event "visit_revoked" so any active visitor session terminates on next snapshot poll
```

The visitor surface is a separate frontend route (`/visit/:token`) with its own bundle entry, smaller than the host bundle (no offer, no settle, no notebook, no settings). The visitor's WebAudio context plays the host's calls; the visitor's render is the host's render path with input handlers stripped out.

The visitor session cookie has no overlap with the host session cookie. A user who is signed in as host and follows their own visit token receives the visitor view; their host cookie is not used for the visit path. This avoids accidental host-state writes from a "visitor" frame that turned out to be the same browser.

---

## 5. Simulation engine design

The simulation engine is the core of the product. It is the difference between "feels alive over weeks" and "screensaver."

### 5.1 Tick loop structure

A scheduler runs every `TICK_INTERVAL_SECONDS` and enqueues per-aviary tick jobs into a work queue partitioned by `aviary_id`. Workers consume jobs and execute the tick for one aviary at a time. Workers are stateless; the queue partition guarantees no two workers tick the same aviary simultaneously.

Each tick is the following pure function applied transactionally:

```
tick(aviary_state, recent_events, ambient_inputs, now) -> aviary_state'
```

Where:
- `aviary_state` is read from Postgres at the start of the tick (with `SELECT … FOR UPDATE` on the aviary row).
- `recent_events` is the events with `server_at > last_tick_at AND server_at <= now`.
- `ambient_inputs` is the time of day in the user's local timezone, the active weather, and any cross-bird signals from the same tick (one bird's alarm influences another bird's mood within the same tick window).
- The new state is written back; events are stamped `processed_at = now`; `aviaries.last_tick_at = now`.

Idempotency: if the worker crashes mid-tick, the transaction rolls back. A retry runs the tick from the same `last_tick_at`, processing the same events, producing the same new state (deterministic given the seed-derived RNG).

Determinism: a tick's RNG is seeded from `(aviary_id, last_tick_at)`. Two separate executions of the same tick produce identical output. This is required for testing and for safe retries.

### 5.2 Drift function

Personality drift is a low-pass filter integrating presence, listen-in, and offers. For each trait:

```
delta = sum_over_events_in_window(weight(kind) * trait_kernel(kind, trait))
value' = clamp(value + max(0, delta * DRIFT_ALPHA_KIND), 0, 1)
```

The `max(0, …)` clamp on the per-event delta is the **monotonic-toward-expressive** rule. There is no code path that produces a negative delta; the function is intrinsically asymmetric. We do not gate this with a config flag, because a flag is a code path that can be flipped on later. A trait that has reached 1.0 (the ceiling) does not move further; a bird saturates and stays saturated.

The `trait_kernel` is a small per-trait coefficient table:

| event kind | boldness | social_warmth | vocal_frequency | plumage_saturation | curiosity |
|---|---|---|---|---|---|
| presence-second | 0.05 | 0.05 | 0.03 | 0.10 | 0.03 |
| listen-in-second (focused bird) | 0.10 | 0.30 | 0.20 | 0.10 | 0.05 |
| listen-in-second (other birds) | 0.02 | 0.05 | 0.02 | 0.04 | 0.02 |
| offer-accepted (target bird) | 0.10 | 0.10 | 0.05 | 0.05 | 0.40 |
| offer-initiated near (target bird) | 0.20 | 0.05 | 0.02 | 0.02 | 0.05 |

The ratios are seeded at the values above and tuned during private beta against the calibration targets in §3.4. Tuning is logged with version history; we don't re-tune in prod without an experiment plan.

### 5.3 Mood transitions

Mood is a finite state machine with weighted transitions per tick. Inputs:

- Time of day (drowsy bias near dusk, alert bias in early morning).
- Active weather (rain → vocal frequency dampened, no mood shift directly; wind → small wary bias for low-boldness birds, alert bias for high-boldness).
- Recent interactions (offer accepted in this session → content; alarm call from another bird → wary).
- The bird's personality (high boldness reduces probability of entering wary on the same input).

Transitions are stochastic; for each non-current mood, compute a transition probability, sum, normalize, draw with the seeded RNG. Mood persists across ticks unless the draw rolls over the threshold.

Cross-bird coupling: each bird's mood transition reads other birds' moods from the start-of-tick state (not within-tick mutations). This makes the tick deterministic and prevents order-of-evaluation effects. A wary mood "spreads" through the aviary across multiple ticks, which matches the slow felt-pace of the product.

### 5.4 Call grammar runtime

The call grammar is the procedural specification of bird vocalizations. It runs on the server only to schedule call windows; the synthesis itself is client-side (see §8).

Per species, we author a small motif library: 8–12 motif templates per species. Each motif is a tuple `(pitch_sequence, duration_envelope, ornament_bias)`. The runtime selects motifs and varies them.

For each tick:

1. For each bird, decide whether a call is scheduled in the next tick interval, with probability shaped by `vocal_frequency`, mood (drowsy → low; alert → high), time of day, and weather (rain → low).
2. If a call is scheduled, pick a motif weighted by mood (drowsy birds favor short low motifs; alert birds favor longer rises).
3. Apply per-call variation: pitch jitter ±1–2 semitones, duration jitter ±10%, ornament insertion (a grace note before or after a sustained pitch) with probability shaped by curiosity.
4. Emit a `CallSchedule` for the bird in the next snapshot, with `pitch_shape` and `duration_envelope` arrays and a window for when the client should play.

Recognizability: the motif library is the per-species "fingerprint." Variation is applied to the realization, not to the motif identity. Pip's calls always come from Pip's species' motif set, which always sits within the recognizable melodic envelope of that species. Users learn the species; variation keeps the call alive within the species.

Chorus: if two birds have overlapping call windows, the renderer mixes them in real time. The schedule generation does **not** force chorus events; it lets them happen statistically. Forcing chorus on a cue would announce, which violates the principle.

### 5.5 Notebook entry generation

The narrator service runs once per ~hour per aviary (it's not on the simulation tick) and decides whether to write a notebook entry. Entry probability is shaped by:

- Time since last entry (longer → more likely; we want sparsity).
- Whether something noteworthy happened (a first-of-week event, a mood transition unusual for a bird, an offer accepted that the bird usually ignores, a chorus event between two specific birds, a settled-aviary moment).
- A jitter to avoid mechanical pacing.

When an entry is generated, the prose is composed from a small grammar of phrase fragments rather than rendered from a template. Phrases reference birds by name, by perch position, by mood verbs ("fluffed against the cool air," "watching the back perch"), by ambient cues ("morning light," "after the rain"). The phrasing must stay in naturalist voice; we do not allow generative content from a foundation model in v1 (see §11.4 for the rationale).

Sparse cadence: target one entry every 2–4 days for a regularly visited aviary, capped at one per local-day except when something rare happens (chorus event, first-of-week, weather + mood interaction).

### 5.6 Day/night and weather

Day/night is computed per-aviary from the user's local timezone (`accounts.local_timezone`) and the current server UTC time. We compute `daypart` and `palette_phase`. Sunset and sunrise vary slightly across the year via a simple solar approximation; we do not need precise astronomical computation, only a slow yearly drift in dawn/dusk times to keep the aviary feeling tied to the user's actual day.

Weather is simulated as a hidden Markov chain per aviary: most ticks remain `clear`; rare transitions to `rain` (median duration 3–8 minutes) or `wind` (median 6–12 minutes). Transition rates are seeded from `(aviary_id, day-of-year)` so weather across aviaries is uncorrelated but each aviary is reproducible. Active weather is in the snapshot; the renderer shows the visual cue.

### 5.7 Tick performance budget

- p50 tick latency: <100ms per aviary.
- p99 tick latency: <500ms per aviary.
- Alarm at `simulation-tick latency p99 > 5s` (per PRD).

A worker pool sized so even a 100x active-aviary spike does not breach the p99. The scheduler's cadence (60s) is the time budget; if ticks run longer than 60s, the queue grows, and a separate alarm fires on queue depth.

Cold aviaries (no events for 24h) skip the regular tick interval and run `TICK_INTERVAL_IDLE_SECONDS=600`. This still updates day/night and mood-time-of-day, but at lower compute cost.

### 5.8 Personality vector persistence and migration

Personality vectors live in `birds.personality_vector` as JSON. The format is versioned (`{ schema: 1, ... }`). If we add a new trait in the future, we run a one-time backfill that derives the new trait from existing traits + species defaults; we never delete existing values. The bird the user adopted on day one is *that* bird, with that personality.

There is no "regenerate bird" endpoint, no admin tool to reset a personality vector. The only personality-mutation paths are the simulation tick and a one-time schema migration job. Both are audit-logged.

---

## 6. Sync model

### 6.1 The architecture *is* the sync

There is no client-to-client sync, no CRDT, no eventual consistency layer. Two devices reading from the same canonical Postgres row see the same state. A user's laptop and phone are both clients of the same `api`; both pull snapshots; both render the same aviary.

### 6.2 Snapshot freshness across devices

- On `visibilitychange` (tab becoming visible) → pull snapshot.
- On long render-frame gap (>2s, indicating laptop suspend or background throttling) → pull snapshot.
- On SSE reconnect → pull snapshot.
- Heartbeat snapshot pull every 60s while the tab is visible (matches tick cadence).
- SSE pushes snapshot deltas at tick cadence and on event-driven transitions.

A device that has been idle for hours does not preserve a stale snapshot; the heartbeat refreshes it, and the SSE reconnect after suspend forces a snapshot.

### 6.3 Conflict prevention

- Personality state: only `sim` writes. Two devices can't conflict because they don't write.
- Mood: only `sim` writes (in response to event-log inputs).
- Bird names: writes go through `api`. The last write wins for names — this is a benign field, not personality. We log a notification in the user's account if a name change happened on another device "to be informed of an unfamiliar device" via the session list, not by an in-product surface.
- Settle: settle is a server-side aviary flag. Setting it cross-device is intended (a user settling on the phone should result in the laptop showing a settled aviary). The undo window is server-tracked too, so an undo on either device works.
- Offer cooldown: server-tracked per bird, so a phone offer prevents a laptop offer during the cooldown.

### 6.4 No last-write-wins on personality — enforced

We enforce this with a Postgres role that has UPDATE privilege on `birds.personality_vector` only for `sim`'s database role. The `api` role does not have that grant. This is an architectural backstop: even a bug in `api` cannot accidentally write personality; the database refuses the update.

### 6.5 Snapshot ordering and dedup

Each snapshot has a monotonically increasing `taken_at`. The client refuses to apply a snapshot older than the last applied. SSE delta events carry a sequence number; out-of-order deliveries are dropped, and the client resyncs with a full snapshot if it sees a gap.

---

## 7. Frontend rendering pipeline

### 7.1 Composition

- The DOM has a small chrome layer (Preact) and a single `<canvas>` for the aviary scene.
- The chrome holds top-bar icons (account, accessibility settings, notebook, offer, settle), the offer modal, the notebook drawer, account settings pages (separate route bundles), and accessibility surfaces.
- The canvas holds the aviary scene: sky, foreground/background, perches, birds, ambient leaves, weather effects.

The chrome and the canvas are composited via z-stacking; the chrome's top bar fades to near-transparent on cursor stillness. The canvas never has chrome drawn into it — no buttons, no icons. This makes the canvas pure scene and the chrome trivially testable in isolation.

### 7.2 Render loop

A single requestAnimationFrame loop drives the canvas. The loop is responsible for:

1. **Snapshot interpolation.** Birds have `current` and `target` perch positions and visual states; the renderer linearly (or eased) interpolates over the snapshot interval. Mood-shaped easing curves: drowsy birds move slower, alert birds snap.
2. **Idle micro-motion.** Per bird, a small set of pose offsets driven by a per-bird local oscillator phase plus mood-shaped amplitude. Preening, head-tilt, body-shuffle. Not animated frame-by-frame; computed from a couple of harmonic oscillators that produce smooth offsets cheaply.
3. **Ambient ornaments.** A small particle system for leaves and feathers. ~3–6 active particles at any time; lifecycle handled with a free-list to avoid allocations.
4. **Day/night palette interpolation.** Background sky gradient and foliage tint interpolate with `palette_phase`. Updated once per second; palette steps are not rendered every frame.
5. **Weather effects.** Rain is a thin streak particle pass; wind is a foreground leaf-rate increase plus a ripple in foliage parallax. Both are subtle; rain intensity is capped low.
6. **Listen-in highlight.** When a bird is focused, a soft halo around it (subtle, not an outline). Mix change is handled by audio, not visuals; the halo is the visual cue that the listen-in is engaged.

The loop targets 60fps. Animations time out gracefully if the device is slow: the renderer lowers the particle count and increases the palette-update interval before sacrificing bird animation, because the birds are the product.

### 7.3 Idle-while-hidden

When `document.visibilityState !== 'visible'`, the renderer **stops rendering** (cancels the rAF loop) and the audio context suspends. This is per PRD: "the client may stop rendering when the tab is hidden, but the simulation continues server-side." On `visibilitychange` back to visible, the renderer pulls a fresh snapshot, primes positions, and restarts the loop.

Critically: the renderer does **not** play a "wake-up" animation on resume. It places birds at the snapshot positions and renders from there as if it had been running all along. The first frame on resume is the aviary mid-action — same as the first paint on a cold load.

### 7.4 First paint and the load state

The HTML shell is served from the edge. It includes:
- Inline critical CSS for the chrome and the canvas frame.
- Inline initial state snapshot (~2KB of JSON for an unauthenticated visitor; for a signed-in user, the snapshot is fetched on the same request that serves the HTML, server-rendered into the page).
- A `<canvas>` element sized to the viewport.
- A small inline bootstrap script that paints the initial sky color and one or two faint motion cues (the "quiet field") immediately, before the JS bundle has finished loading.

This means the first frame the user sees on a cold cache is **not** a spinner. It is a quiet field that resolves into the aviary as the bundle finishes loading. The bundle's final step is to render the first bird at its snapshot position. Time-to-first-bird is measured from navigation to first non-blank canvas paint that includes a bird.

### 7.5 Reduced-motion mode (its own designed surface)

When `prefers-reduced-motion: reduce` is set, or when the user opts in, the renderer switches to a different render path:

- **Idle micro-motion → cross-fades between still poses.** Each bird has a small set of named poses (perched-alert, preening, head-tilt-left, head-tilt-right, fluffed). The reduced-motion renderer cross-fades between these with a long ease. A pose change happens roughly every 8–12 seconds per bird, mood-shaped.
- **Flight transitions → cross-fades between perches.** The bird at perch A fades out as the bird at perch B fades in. No animated path.
- **Ambient leaves and feathers → removed.** No particle system.
- **Day/night palette interpolation → preserved but slowed** to a multi-minute interpolation per palette step.
- **Calls → preserved at full quality.** Audio is independent of the visual mode.
- **Weather → simplified.** Rain is a soft tint shift instead of streaks; wind is removed visually but still drives mood.

This is its own aesthetic, not a stripped fallback. The design system specifies the cross-fade timings and pose set with the same care as the standard mode. A reduced-motion user sees a calmer, slower aviary; they do not see a degraded one.

### 7.6 Bird rendering — sprite vs vector

Birds are rendered as compact SVG paths converted at build time to small render commands (a custom intermediate format). For each species we author 5–8 pose templates (preen, head-tilt, perched, fluffed, alert, calling). Plumage saturation level is rendered as a palette swap on the path commands; saturation step changes when the level integer changes server-side, never per-frame.

The total visual asset budget per species is ~40KB on disk, ~15KB after build-time packing. Six species → ~90KB. The rest of the visual assets (sky gradients, foliage shapes, perches) are ~50KB. Visual asset total: ~140KB out of the 2MB bundle budget.

### 7.7 Greeting render

Return-greeting is server-driven. The snapshot arrives with `greeted_this_session: false` and a `greeting_intent` field describing which bird greets, with what motif, at what time offset. The renderer shows the greeting visually (head turn, glance up, step toward front) and the audio scheduler plays the greeting call. The client-side timing of the visual greeting is keyed to the audio onset to keep them in sync.

The greeting variation:
- **Bird selection** is server-decided based on boldness, mood, and a small randomization. The boldest awake bird greets first today; the second-boldest greets second after a stagger.
- **Form** depends on absence length: <30min → glance only; 30min-12h → glance + soft call; 12h-2d → call + step toward front; >2d → longer call followed by another bird's response.
- **Variation** is procedural: pitch jitter, motif choice from a small set, stagger offset.

A staggered chorus is permitted only by emergent statistical overlap, not by the server explicitly scheduling a unison cue.

### 7.8 Empty-aviary state

Between adoption and first bird placement, the canvas renders the same quiet field as the load state. The first bird then arrives with a fly-in to its starting perch (a soft alpha-fade-in plus a motion path from offscreen-back to its perch). Subsequent first-paint loads place birds already at their perches; the fly-in is a one-time onboarding moment.

---

## 8. Audio pipeline

### 8.1 Synthesis architecture

- One AudioContext per session. Suspended on `visibilitychange→hidden`, resumed on visible.
- A master bus → per-bird buses → per-call voices.
- Each call voice is built from: a primary oscillator (or a pair for harmony), a noise generator (filtered), an amplitude envelope, a pitch envelope, and a small effect chain (subtle reverb, slight randomized pan).
- Voices are pooled (24 voices total). When a call ends, its nodes are returned to the pool; we never allocate-on-call. This is the no-memory-growth invariant in CI.

### 8.2 Call rendering

When a call schedule arrives in the snapshot:
1. The audio scheduler reserves a voice from the pool.
2. At the scheduled time (driven by `audioContext.currentTime` math, not setTimeout, for sample-accurate timing), the scheduler plays `pitch_shape` over `duration_envelope`.
3. Per-pitch jitter and ornament insertions are applied at synthesis time using the seeded RNG from the snapshot, so the call sounds the same on the same tick across two devices but different tick-to-tick.

### 8.3 Listen-in mix

When the user engages listen-in on bird X:
- The bus gain for X's voice ramps up to `1.0` over `LISTEN_IN_RAMP_MS` (1.5s).
- The bus gains for all other birds ramp down to `0.4` (a re-balance, never silent — per PRD).
- Disengage reverses the ramp.

The ramps use `linearRampToValueAtTime` on `GainNode`. The change is gradual; a hard cut would convert the aviary into a UI of soloable tracks.

### 8.4 Chorus mixing

Two birds calling at once go through their own buses, summed at the master. Per-call pitch jitter ensures real-time variation, which is what makes a chorus a chorus and not stacked loops. We instrument a small "phase coherence" check in dev builds to verify that two simultaneous calls don't accidentally produce the phase-cancelling artifact PRD warns about; the check fires if pitch differences are below a threshold for a sustained window across multiple birds.

### 8.5 Captions

When captions are enabled, every call emits a caption phrase generated from the call's actual pitch shape and duration envelope (e.g., "a soft three-note rise" if pitches ascend over short durations; "a low trill, paused, low trill again" if the same low motif repeats with a pause). The caption is a small DOM element positioned near the calling bird's canvas position, fading in as the call starts and out as it ends. Multiple simultaneous captions are stacked vertically with small offsets.

The caption generator is a small grammar with phrase templates keyed to motif shapes (rise, fall, trill, single-note, repeated-pair, ornamented). The output is naturalist voice: lowercase, specific.

### 8.6 WebAudio fallback

If WebAudio is unavailable (no AudioContext, audio permission denied, hardware error):
- The aviary plays in graceful silence.
- Captions are turned **on by default** so the user can read the calls.
- The matter-of-fact accessibility settings page surfaces a one-line note: "Audio is unavailable on this device. Captions are on so you can read what the birds are saying."

We do **not** fall back to recorded audio. The "no recorded audio" rule is unconditional (per PRD §accessibility_perf).

### 8.7 Volume and mute

- Volume slider in accessibility settings (matter-of-fact UI). Slider value persists per-account in `accounts.settings`.
- Mute toggle is also in accessibility settings. Muted aviaries get captions on by default (the user can disable captions if they want a fully silent surface, but the default protects access).

### 8.8 Greeting audio timing

The return-greeting's audio onset is keyed to the visual greeting via the snapshot's `greeting_intent.audio_onset_offset_ms`. The renderer and the audio scheduler share the snapshot reference so they fire together. A drift check confirms ≤50ms onset skew across the duration of a session.

### 8.9 Audio test rig

A headless test rig captures the rendered audio buffer for a given motif schedule and runs:
- **Spectral fingerprint check**: each species' calls cluster in a specific spectral region; outliers fail.
- **Same-call-twice detector**: if two calls within a 60s window are within a tight similarity threshold, fail. This is the "no looped audio" invariant in test form.
- **Recognizability test**: a small panel of test motifs is reviewed periodically by a designer; the designer rates whether the species is identifiable across mood/personality variation.

---

## 9. Accessibility surfaces

### 9.1 Screen-reader narration (separate stream, not ARIA labels)

A separate accessible region (`role="status"`, `aria-live="polite"`) hosts running prose narration. The narration is delivered via the `narration` SSE stream. Phrases appear at a slow cadence (one per 30–60s at idle, faster on user-initiated events).

The phrases are written in naturalist voice:

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

The phrase generator is server-side (in `narrator`) and uses the same naturalist-prose grammar as the notebook. The grammar emits prose, not state lists. We refuse the ARIA-label-automation approach as a matter of design.

### 9.2 ARIA labels (small and matter-of-fact for UI affordances only)

Top-bar icons and chrome controls have ARIA labels in matter-of-fact voice (matching the system-surface convention): "Field notebook," "Offer," "Settle the aviary," "Account settings," "Accessibility settings." These are control labels, not narration of the aviary.

The canvas itself has `role="img"` with `aria-label` set to a one-line summary and points at the live narration region for the full prose.

### 9.3 Keyboard navigation

- `Tab` moves through top bar items in order (account, accessibility settings, notebook, offer, settle).
- `Tab` from the last top-bar item enters the aviary scene; focus is on the first bird (front-perch first, then middle, then back; ties broken by left-to-right canvas position).
- `Arrow keys` move focus between birds.
- `Enter` triggers listen-in on the focused bird.
- `Escape` exits listen-in and returns focus to the bird.
- `Tab` from a bird exits the aviary scene to the next focusable element.
- The offer affordance opens with `O` as a global shortcut and is fully keyboard-navigable (item list with arrows, Enter to confirm).
- Settle is reachable from the top bar; activation has a confirm-via-undo, no extra modal.

Focus indicators are a soft outline rendered into the canvas for birds (with high-contrast color computed against the local pixel area for visibility against bright and dim aviary states) and the standard browser focus indicators (overridden with accessible alternatives) for chrome.

### 9.4 Captioning

Per §8.5. The captions toggle is in accessibility settings and is on by default when WebAudio is unavailable.

### 9.5 Reduced motion

Per §7.5. Honoring `prefers-reduced-motion` is automatic; the user can also opt in/out manually in accessibility settings.

### 9.6 Color contrast

All chrome text passes WCAG AA. Settings, sign-in, error surfaces, captions, and any narration text rendered visually pass AA. The aviary canvas itself does not contain user-readable text.

The design system defines the exact contrast ratios per surface; we do not author the design system here, but we name the constraint: every text style in the chrome ships with an automated contrast assertion against the surfaces it appears on, and CI fails on regression.

### 9.7 Accessibility regression CI

- Every PR runs `axe-core` against the chrome routes (`/`, `/settings`, `/accessibility`, `/notebook`, `/sign-in`, error pages, `/visit/:token`).
- Lighthouse accessibility check runs nightly on synthetic browsers; failing scores file a ticket automatically.
- Manual screen-reader review (NVDA, VoiceOver) before any v1 launch milestone, repeated at any significant chrome change.

### 9.8 Signaling that accessibility is real

The settings UI surfaces accessibility prominently. The matter-of-fact voice on the settings page explains what each toggle does in plain language, without softening or charm:

> Reduced motion: the aviary uses slow cross-fades instead of animation. Calls and birds still happen. The visual style is calmer.
>
> Captions: short text appears near each bird as it calls.
>
> Larger text: increases the size of all readable text in settings, the field notebook, and captions.

This page does not perform the naturalist voice and does not pretend the user is being noticed. They are configuring the system.

---

## 10. Performance budgets and observability

### 10.1 Bundle budget

- Initial JS bundle (before first paint): ≤2MB gzipped.
- Initial bundle composition target:
  - Render core (Preact + canvas renderer + audio scheduler + WebAudio): ~250KB
  - Visual assets (six species + scene): ~150KB
  - Audio motif library + DSP: ~120KB
  - Initial state + bootstrap: ~30KB
  - Misc (styles, fonts subsetted to characters used): ~80KB
  - Headroom: ~1.4MB (intentional; we lose some to tree-shaking failures and library overhead in practice).
- Code-split routes (loaded on demand): account settings, accessibility settings, notebook drawer, visit-invitation flow, account export, sign-in. Each ~50–150KB.

We enforce the budget in CI with `bundlewatch` (or equivalent). PRs that push the initial bundle over a threshold (e.g., 1.8MB warning, 2MB fail) cannot merge without explicit budget reset.

### 10.2 Time-to-first-bird

- Target: ≤500ms on mid-tier mobile / 4G.
- Method:
  - HTML shell served from edge cache.
  - Inline initial state snapshot (small).
  - Inline critical CSS.
  - Bootstrap script paints quiet field immediately.
  - Render bundle parallel-loads; on `bundle ready`, the bundle reads the inline snapshot and renders the first bird.
- Measurement: synthetic Lighthouse runs from common geographies; real-user metric (page-render-timing API), aggregated only.

### 10.3 60fps idle motion

- Target: 60fps on a 5-year-old mid-range laptop.
- Method:
  - Canvas2D with all rendering work on the main thread; expensive work (audio scheduling, narration generation if client-side) on Web Workers.
  - Idle micro-motion is computed from harmonic oscillators (cheap math) rather than per-frame physics.
  - Particle systems use object pools.
  - Snapshot interpolation is straight-line lerp; no per-frame allocation.
- Measurement: `requestAnimationFrame` timestamps, p99 frame time tracked via aggregate-only RUM.

### 10.4 No memory growth over 30 minutes

- Target: stable RSS over a 30-minute session.
- Method:
  - Voice pools for audio.
  - Particle pools for ambient ornaments.
  - Notebook entries scrolled out of view are unmounted (virtualized list).
  - Every WebWorker is bounded; no per-event worker spawn.
  - Snapshot history is kept to the last 3 snapshots; older are dropped.
- Measurement: a CI test runs a 30-minute simulated session and asserts heap growth <5% (chrome-headless with `performance.memory`).

### 10.5 Server-side budgets

- Tick latency p50 <100ms, p99 <500ms, alarm at 5s p99.
- API latency p99 <300ms for snapshot endpoints.
- Event ingestion p99 <100ms for batches up to 50 events.
- Magic link issuance p99 <2s end-to-end (including mailer).

### 10.6 What we measure

- **Aggregate operational telemetry**: request counts, latencies, error rates, anonymized session-duration histograms, render-frame timing, audio-context errors, simulation-tick latencies.
- **No per-bird state in any telemetry record.** The data pipeline strips per-account dimensions at the event boundary; the analytics warehouse never sees `aviary_id` or `bird_id` (it sees coarse session bucket only).
- **No per-account interaction history** in any aggregated form.
- **No streak-style metrics computed at the data-warehouse level**, ever. The aggregation that would compute "average days visited per user this week" is not part of the analytics schema; the schema does not contain a per-user dimension.

The privacy boundary is enforced at the analytics ETL: a job validates that no aggregate metric contains a per-account dimension. The job fails the pipeline if it sees one.

### 10.7 What we deliberately don't measure

- We do not measure individual user retention curves to drive product decisions about engagement features, because the engagement features that would result are exactly the ones the product refuses.
- We do not collect per-bird interaction frequency, per-call event histories, per-drift-tick deltas in any aggregate dashboard.
- We do not run experiments on drift calibration on live users without explicit cohort consent (alpha/beta enrollment).

### 10.8 Observability stack

- Logs: structured JSON, shipped to a log aggregator. PII redaction at the logger boundary. The redaction layer is unit-tested.
- Metrics: Prometheus-compatible. Aggregated counters and histograms. No per-account labels.
- Traces: OpenTelemetry on the API and sim services. Trace IDs are not joined to account IDs in any persistent store; they live in logs only and expire with log retention.
- Synthetic browsers: nightly + every 5 minutes during business hours. Hit the major flows: cold load, sign-in, snapshot pull, listen-in, offer, settle, notebook open, settings open, sign-out.

### 10.9 Error budgets and alarms

- Tick latency p99 5s → page on-call.
- API error rate >1% over 5 minutes → page on-call.
- Email issuance failure rate >5% → page on-call (auth depends on this).
- WebAudio error rate spike → ticket only (silence-with-captions is graceful; not page-worthy).

---

## 11. Privacy, security, and content audit

### 11.1 Data classification

- **Tier 0 (highly sensitive)**: email, magic link tokens, session tokens.
- **Tier 1 (sensitive)**: per-bird personality vectors, mood, position, notebook entries, event log.
- **Tier 2 (operational)**: aggregate metrics, error counts, latencies.

Tier 0 is encrypted at rest with per-column KMS keys. Tier 1 is encrypted at rest with database-level encryption and is **never** exfiltrated to analytics. Tier 2 is the only data that can flow to analytics infrastructure.

### 11.2 Email handling

- Email is stored once, on `accounts.email_encrypted`, KMS-encrypted.
- All other references are by `email_lookup_hash` (HMAC-SHA256 with a server-side key) for sign-in lookup.
- Logs never contain email or hashes; redacted at the logger.
- Email is decrypted only at three points: outbound email send, account export to verified address, account email change verification.

### 11.3 Auth

- Magic links: 15-minute expiry, single-use, HMAC-signed, validated server-side. Replay protection by `consumed_at` set atomically on use.
- Session tokens: HMAC-signed, HttpOnly + Secure + SameSite=Lax cookies. 90-day expiry, sliding refresh on use. Revocable from settings.
- Rate limits: per-email and per-IP on magic-link issuance. Sign-in attempts logged with hashed identifiers; abuse triggers backoff.

### 11.4 Generative content

We do **not** use a foundation model in v1 to generate notebook prose or narration phrases. The reasons:

- **Determinism**: notebook prose for a given event window must be reproducible for testing.
- **Voice consistency**: the naturalist voice is load-bearing; a foundation model with its own affective register would drift even with prompt engineering.
- **Privacy**: per-bird interaction state cannot leave our system, including to model APIs.
- **Performance**: a model call per notebook entry adds latency and operational complexity.

The narration grammar is a small, hand-authored generator. We may revisit this post-v1 if we can run a private model with constrained output, but v1 is grammar-driven.

### 11.5 Notebook content audit

A daily job samples a small fraction of notebook entries and runs voice-quality checks (lowercase, present-tense, specific to bird/perch/mood, no announcement framing, no engagement language). Failures file an internal ticket. Entries are not modified post-hoc; the audit informs grammar tuning.

### 11.6 Visitor isolation

- Visitor session cookies are scoped to `/visit/*` routes only.
- Visitor sessions cannot read host notebook, settings, account info, or events.
- Visitor activity does not write to `events`; only to `visit_sessions`.
- The visitor's render bundle has no offer or settle code paths.

### 11.7 Account export

Export bundle JSON includes:
- Account email (the verified address; no decryption needed by the recipient — they own the address).
- Bird records: id, species, name, current personality vector, current mood, adopted_at.
- Notebook entries.
- Settings.
- A schema version.

Exports are generated as a one-shot job, written to object storage with a 7-day expiry, signed download URL emailed to the verified address. The export does not include event log entries (these are operational) or any tier-0 secrets.

### 11.8 Account deletion

- Soft-delete sets `accounts.state = 'pending_delete'`, `deletion_scheduled_at = now() + 30d`. The account is not signable into via magic link during this state; sign-in attempts return a matter-of-fact "your account is pending deletion" surface with a recovery option.
- Hard-delete (a daily job) removes the account row, all `aviaries`, all `birds`, all `events`, all `notebook_entries`, all `visit_invitations`, all `visit_sessions` for that account. Telemetry rows tied to the account are not stored (we don't store per-account telemetry); aggregate counters are unaffected.
- Cross-service: the `narrator` and `sim` workers check account state at tick start; if pending_delete or deleted, they skip the aviary.

### 11.9 Threat model

Plausible threats and our position:

- **Stolen magic link**: 15-minute window + single-use limits the blast radius. Session token can be revoked from any other signed-in device.
- **Stolen session cookie**: the user revokes the device from settings. Cookies are HttpOnly so XSS can't trivially steal them; Secure prevents non-HTTPS exposure; SameSite=Lax limits CSRF.
- **CSRF**: All write endpoints require a CSRF token in addition to the session cookie, validated on every POST/DELETE.
- **Compromised employee account**: Database access is via per-service roles with column-level grants. No employee or service account has direct read access to `accounts.email_encrypted`; access is mediated by an audit-logged decryption helper. Production access is logged.
- **Mass scraping of public aviaries**: Aviaries are not public. There is no "explore" endpoint; visitor links are individually tokenized.
- **Visit-link sharing**: A single visit token can technically be shared to a third party by the original visitor. Mitigation: token is single-use until activated; once activated, additional access requires a fresh link from the host. We document the property; we do not require browser fingerprinting (which would require new tracking infrastructure).

---

## 12. Rollout

### 12.1 Phases

- **Phase 0 — Internal alpha** (2 weeks): team accounts only. Tick correctness, snapshot accuracy, drift-calibration smoke tests. Synthetic browsers established.
- **Phase 1 — Closed beta** (4–6 weeks): invite-only accounts (via direct invitation, not the visit feature). Two birds per aviary, age-gated bird three available at 14 days as a drift-test compression. Drift calibration tuned against this cohort. Voice quality reviewed daily.
- **Phase 2 — Open beta** (4 weeks): public sign-up, two birds per aviary, age-gated bird three at the production cadence (30 days). All v1 features behind feature gates; gradual rollout: 10% → 25% → 50% → 100% over 4 weeks.
- **Phase 3 — General availability**: feature gates removed. Bird cap of seven; subsequent birds follow the age-gate schedule.

### 12.2 Bird-per-aviary ramp

The age-gate values in §3.4 (`BIRD_OFFER_AGE_GATE_DAYS = [30, 60, 120, 240, 365]`) are chosen so:
- Day 30: third bird offered (~one month is "an aviary that has settled in").
- Day 60: fourth bird offered (~two months).
- Day 120: fifth bird offered.
- Day 240: sixth bird offered.
- Day 365: seventh bird offered, capping the aviary.

These are calibration targets; we tune in beta against observed adoption rates. The shape (compounding intervals) is the stable property; specific days flex.

### 12.3 Feature gates

- Magic-link sign-in: required from Phase 0.
- Two-bird adoption: required from Phase 0.
- Listen-in, offer, settle, notebook: required from Phase 0 (these are the core surfaces).
- Visit invitations: gated; available in beta cohorts.
- Reduced motion: required from Phase 0 (no shipping without this).
- Captions: required from Phase 0.
- Account export: gated; available in beta cohorts.

Each gate is a config in the API; client respects gate flags in the snapshot response.

### 12.4 Deployment model

- Infrastructure-as-code (Terraform or equivalent).
- Blue/green deploys for `api`, `narrator`, `mailer`. Single-step deploys for `edge` (Workers).
- Sim workers deployed independently with rolling restart; the queue absorbs in-flight ticks.
- Migrations: versioned, applied via a deploy-step migration runner. Backwards-compatible in both directions for at least one deploy cycle (because the previous version of the app may still be running).

### 12.5 Day-one instrumentation

- Aggregate operational telemetry on day one (per §10.6).
- Synthetic browser tests on day one.
- Voice-quality audit job (§11.5) on day one.
- Drift instruments (§13.2) on day one.

What we **don't** instrument on day one (or ever):

- Streak counters or per-account engagement metrics.
- Per-bird interaction histories in any analytics surface.
- A/B testing framework that splits on per-account dimensions for any feature touching the bird engine without explicit cohort consent.

### 12.6 Onboarding

- New user signs up with email → receives magic link → signs in.
- First-time sign-in flow:
  1. Account is created; aviary is created.
  2. Two starter birds are seeded; species are chosen by the system, not by the user, with complementary personality seeds.
  3. The user is presented with the empty-aviary quiet field, then the first bird flies in to its perch.
  4. After both birds are perched and called once, a small chrome surface (matter-of-fact voice) lets the user name them. Default suggestions are species-appropriate (e.g., "Pip," "Wren"); the user can change them, including later.
  5. The user is in the aviary.

There is no "tutorial" sequence. There are no tooltips explaining listen-in, offer, settle. Discoverability comes from the top bar's existence and the user's natural exploration. A short matter-of-fact "how this works" is reachable from accessibility settings; it is not pushed.

### 12.7 Release-day risks

- WebAudio support in the wild: test against browsers in the support matrix; have the silence-with-captions path well-rehearsed.
- Edge initial-snapshot delivery: have a fallback that pulls the snapshot from the API if the inline path fails.
- Sim worker capacity: pre-provision 3x peak expected load.
- Email deliverability: warmed mailer setup, SPF/DKIM/DMARC configured; magic-link issuance has a dedicated subdomain.

---

## 13. Risks and mitigations

### 13.1 Drift miscalibration

**Risk**: drift is too fast (Tamagotchi-feeling, "click for change") or too slow (screensaver, "nothing I do matters").

**Mitigations**:
- Calibration constants live in service config, tunable per environment.
- Internal alpha and beta cohorts run with denser drift to verify the curves. Production cohort runs at the calibrated pace.
- **Drift instruments**: aggregate (no per-account dim) histograms of trait deltas per week, per cohort. We can see if a cohort's bird population is drifting in expected ranges.
- A/B-able alpha values gated behind cohort enrollment with explicit consent.
- Hard ceiling: traits clamped to [0, 1]; saturation is a graceful end-state rather than a runaway.

**Why this is high-priority**: the PRD calls out "feels alive over weeks" as the headline; drift calibration is the named implementation of it.

### 13.2 Sync correctness

**Risk**: a bug in `api` writes personality directly; a tick processes the same event twice; a snapshot is rendered against stale state.

**Mitigations**:
- Postgres role-level grants prevent `api` from writing `birds.personality_vector`. Backstop, not first line.
- Events are idempotent on `processed_at`; tick reads only `processed_at IS NULL` events.
- Snapshot `taken_at` ordering on the client.
- A consistency-check job runs hourly: for a sample of aviaries, recomputes expected drift from the event log replay and compares to canonical state. Drift exceeds a small tolerance → alert.
- Chaos test in CI: two clients submit overlapping events; assert canonical state is correct under both event orderings.

### 13.3 Audio uncanniness

**Risk**: procedural calls sound mechanical, "samey," or off-pitch in ways that break the spell.

**Mitigations**:
- Per-call jitter, ornament insertion, motif variation.
- Same-call-twice detector in tests (§8.9).
- Periodic listening-test reviews by a designer or audio engineer.
- Beta cohort feedback specifically on audio (qualitative feedback channel, not telemetry).
- WebAudio-fallback path is silence-with-captions, which is a clean fallback rather than a degraded one.

### 13.4 Accessibility regressions

**Risk**: a feature ships with a regression in screen-reader narration, keyboard navigation, or reduced-motion rendering; a regression goes unnoticed because the regression test panel is too small.

**Mitigations**:
- axe-core in CI on every PR for chrome routes.
- Lighthouse nightly accessibility check.
- Manual NVDA + VoiceOver review pre-launch and at any chrome change.
- Reduced-motion mode is its own designed surface, owned by the same team as the standard mode (not "an a11y feature" handed off to a smaller group).
- Captions and narration use the same naturalist-voice grammar as the rest of the product, so a charm regression there fires the same alarm as a charm regression in the notebook.

### 13.5 Voice drift (charm regression)

**Risk**: a contributor adds a "Welcome back!" toast, a streak counter, a tooltip, a celebration on a milestone — any "harmless" announcement-style surface.

**Mitigations**:
- A linter rule that flags PRs containing strings matching announcement patterns (`"Welcome back"`, `"streak"`, `"days visited"`, `"achievement"`, `"unlocked"`, `"badge"`, `"level up"`, `"score"`). The linter fails the PR; an explicit comment from a designer is required to override (and the override should not happen).
- A code review checklist: every new user-visible string is annotated with which voice (naturalist or matter-of-fact) it belongs to and which surface it appears on.
- Voice-quality audit on notebook entries (§11.5).

### 13.6 Privacy regression

**Risk**: per-bird state leaks into aggregate analytics; a logger captures an email; a shared analytics dashboard accidentally has a per-account dimension.

**Mitigations**:
- Logger PII redaction layer with unit tests that fail on regression.
- Analytics ETL validation (§10.6).
- Audit access to `accounts.email_encrypted`; production access is logged and reviewed monthly.
- Threat-model exercise pre-launch and on any major architectural change.

### 13.7 Sim worker capacity

**Risk**: a population spike exhausts sim workers; ticks run late; aviaries feel laggy.

**Mitigations**:
- Auto-scaling worker pool tied to queue depth.
- Cold-aviary tick interval (10 minutes) for accounts with no recent events; this is the long tail of the population by count and is cheap to defer.
- Tick latency alarms (§10.5).

### 13.8 Magic-link delivery failures

**Risk**: emails get marked as spam; users can't sign in.

**Mitigations**:
- Dedicated mailer subdomain with SPF/DKIM/DMARC.
- Use a reputable transactional email provider.
- Include a matter-of-fact help link on the sign-in page if the user reports a delivery problem.
- Monitor email-acceptance rates by domain; alert on regression.

### 13.9 Personality vector loss

**Risk**: a migration corrupts personality vectors; a bug causes a vector to be reset to defaults.

**Mitigations**:
- Personality is in versioned JSON; migrations are forward-only with backfill from existing values.
- Tested migration runs against a copy of production data before production deploy.
- Backups of `birds` table at a high cadence (point-in-time recovery via the database provider).
- Alarm on personality-vector deltas exceeding a per-tick maximum (a vector that snaps to a new value within one tick is a bug; alarm fires).

### 13.10 Visitor abuse of revoked links

**Risk**: a malicious visitor finds a way to keep a session alive past revocation.

**Mitigations**:
- Visitor sessions check invitation state on every snapshot pull (snapshot pulls happen at tick cadence).
- Visitor sessions have short cookie expiry (1 hour) and require re-validation against the invitation row on every API call.
- The host's revocation is immediate at the database level; no caching window allows continued access.

---

## 14. Cross-cutting concerns

### 14.1 Repo and team structure

- Single monorepo: `apps/edge`, `apps/api`, `apps/sim`, `apps/narrator`, `apps/mailer`, `apps/web` (frontend), `packages/types`, `packages/audio-dsp`, `packages/render`, `packages/narration-grammar`, `packages/calibration`.
- Two product teams: **engine** (sim, narrator, calibration, drift) and **surface** (web, edge, audio, accessibility).
- One platform engineer covering edge, observability, deploy.
- One audio specialist (full-time during phases 0–2; possibly part-time post-launch).
- One designer (visual + voice).
- Embedded accessibility reviewer.

The split between engine and surface is along the snapshot boundary (§2.2). Engine owns everything above the snapshot; surface owns everything below.

### 14.2 Testing strategy

- **Unit tests**: drift function, mood transition probabilities, call-grammar runtime, narration grammar. All deterministic given a seed.
- **Integration tests**: full tick from event log to canonical state to snapshot. Round-trip with a simulated event sequence over a multi-week timespan; assert calibration targets hit.
- **End-to-end browser tests**: Playwright running headless Chrome across critical user flows. Includes the listen-in audio mix verification (audio buffer captured and analyzed).
- **Performance tests**: 30-minute session memory test, 60fps idle test on a CI-provisioned mid-tier machine.
- **Accessibility tests**: axe-core on all chrome routes; screen-reader behavior tests with a recorded NVDA/VoiceOver harness on a staging environment.
- **Audio tests**: motif fingerprinting, same-call-twice detection (§8.9).
- **Voice tests**: notebook prose validity (lowercase, present-tense, no announcement framing, specific reference to bird/perch/mood), narration prose validity.

### 14.3 Configuration

- All calibration constants live in a config package versioned alongside the code.
- Per-environment overrides for development/staging/production.
- Config changes ship via deploy; no runtime hot-config for calibration values.
- Voice-quality thresholds, drift alphas, tick intervals, weather rates: all in config, all reviewed before change.

### 14.4 Internationalization

- v1 ships in English. Naturalist voice is hand-authored; localization is a v2 problem because the voice is load-bearing and machine translation will not preserve it.
- The chrome and matter-of-fact surfaces use a translation system (i18n keys), so they can be localized in v2 without architectural change.
- Day/night uses the user's local timezone (not locale-derived).

### 14.5 Browsers

- Last two majors of Chrome, Safari, Firefox, Edge.
- Detect on the edge; older browsers see a matter-of-fact unsupported-browser surface explaining which browsers are supported and why.
- We do not maintain compatibility paths or polyfills for older browsers.

### 14.6 Mobile web

- Responsive layout per `aviary_layout.md`: no cropping any bird out of frame, scene compresses horizontally on narrow viewports.
- Touch input: tap on bird = focus + listen-in; tap empty space = exit listen-in.
- iOS WebAudio quirk: AudioContext requires a user gesture to start. Handled by deferring AudioContext creation to first interaction; the silence-with-captions path covers the very first frame before any tap.

### 14.7 Data retention

- Events: 30 days (operational only; the personality drift they drive is already in canonical state).
- Sessions: 90 days after last seen; revoked sessions retained 30 days for audit.
- Magic links: 30 days (consumed or expired); we retain the row for replay-protection forensics.
- Notebook entries: indefinite (the user's record).
- Visit logs: indefinite.
- Aggregate telemetry: 13 months rolling.

### 14.8 Backups and disaster recovery

- Postgres point-in-time recovery via managed provider.
- Object storage backups for export bundles handled by the provider's redundancy.
- Restore drill quarterly: restore a snapshot to a staging cluster, verify a sample of accounts can sign in and their birds are intact.

### 14.9 Internal admin

- A small internal admin app (separate auth, internal-only network) lets engineering inspect operational state of an aviary by `aviary_id` (after the user has filed a support request).
- The admin view shows current snapshot, last 100 events, last tick output, and any error logs.
- The admin view does **not** show personality vector values numerically; it shows the same "rendered visual signals" the client sees, plus the tick's most recent computed deltas (for diagnosing calibration). Showing the raw vector in the internal tool would risk it leaking back into the user-visible surface; we don't.
- Admin actions are audit-logged.

### 14.10 Support surface

- An "Get in touch" link on error surfaces and in account settings, in matter-of-fact voice.
- Inbound support channel reads tickets and uses the internal admin to debug.
- No in-product chat, no AI support agent.

---

## 15. Open questions and defensible calls

These are points where the PRD allows multiple defensible answers; we make calls and flag them so a future reader can revisit if calibration suggests otherwise.

| Question | Call | Why |
|---|---|---|
| Frontend framework | Preact for chrome, Canvas2D for scene | Bundle budget. Preact gives most of React's shape at ~3KB; canvas keeps the scene as a single composited surface. |
| Personality-vector storage | JSONB column | Trait set is small, evolution is slow, schema versioning is cheaper than a separate table. |
| Tick scheduling | Postgres-row partition queue | Can be replaced with a dedicated queue later; starting with Postgres avoids early infra. |
| Real-time client comms | SSE | Simpler than WebSockets through CDN; bandwidth is small. |
| First two species pair | Complementary seed (one bold/warm, one wary/quiet) | Makes "two birds" feel like two birds from the start. |
| Mood-reset cadence anchor | 04:00 user-local time | Daily-ish cadence, low-activity hour. |
| Notebook entry frequency | One entry every 2–4 days for a regular visitor | "Rare" enough to feel observational. |
| Drift alpha values | Initial table in §5.2 | Will be tuned in beta; recorded in config with version history. |
| Visitor cookie scope | `/visit/*` only | Prevents accidental host-state writes. |
| Audio voice pool size | 24 voices | Six birds × four overlap-windows worst-case; pool prevents allocation. |
| Inline initial snapshot for unauthenticated visitors | Yes | Time-to-first-bird budget. |
| Bird-offer schedule | Compounding gates: 30, 60, 120, 240, 365 days | Year-old aviary reaches near-cap. |
| Event log retention | 30 days | Operational sufficient; personality already in canonical state. |
| WebAudio fallback | Silence with captions, no recorded-audio fallback | PRD constraint; preserves bundle budget. |
| Internal admin numeric view | No raw personality numbers | Avoid resurfacing risk. |
| First-time naming UI | Matter-of-fact (the user is configuring, not noticing) | Voice rule. |
| Reduced-motion default | Honors `prefers-reduced-motion` automatically; user can opt in/out manually | Standard practice + user agency. |

The spec lives. The engine is the thing that has to be true, and the surface is the thing that has to feel true. Build the engine to be true and the surface to feel true. Refuse the engagement features even when the meeting wants you to add them. Ship the accessibility surfaces with the rest of the product, not after. Calibrate the drift function in beta against the named targets. Watch the voice. The aviary is the welcome.
