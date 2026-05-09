# Pocket Aviary — v1 Implementation Plan

This plan turns the PRD into an executable build. It is written so that a separate engineering team — frontend, backend, audio/graphics, SRE, accessibility — can pick up specific sections and start work without further clarification. It interprets, rather than restates, the spec; where the PRD names a calibration target, the plan pins down how to land it; where the PRD names a refused feature, the plan pins down how the architecture refuses to grow it back.

Throughout, the load-bearing affective rules (`feels alive, not robotic`, `notice never announce`, `charm comes from specificity`, `restraint over richness`, naturalist vs. matter-of-fact voice split, presence as central idea, drift monotonic toward expressive) are treated as engineering constraints with concrete enforcement points.

---

## 1. Scope

### 1.1 In scope (v1)

- Single-user account, single canonical aviary, magic-link auth, multi-device sync.
- Bird engine: 6 species pool, 2 starter birds at adoption, cap of 7 per aviary, age-gated new-bird offers, stable internal bird ID, per-bird personality vector + mood, slow drift, mood persistence across sessions, bird-to-bird interaction.
- Aviary scene: single horizontal scene with three perch zones, day/night cycle anchored to user local time, ambient weather, ambient micro-motion, no in-aviary chrome, fading top bar with notebook / offer / settle / accessibility / account icons.
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle (with 5s undo), field notebook (read-only, sparse, naturalist).
- Server-side simulation tick at ~60s cadence (calibration target — see §6), continuing whether the client is connected or not.
- Procedural call synthesis client-side via WebAudio. Procedural-only, no recorded fallback.
- Accessibility surfaces as designed surface (not parity-by-checklist): screen-reader narration in naturalist prose, reduced-motion mode (cross-fade rendering), call captions, full keyboard navigation, WCAG AA contrast on user copy.
- Visit-invitation feature: per-invite opt-in, off by default, read-only ambient view, revocable, 30-day invite expiry, visit log in settings, opt-in visit notification toggle (off).
- Account hygiene: email-change verification, soft-delete + 30-day window, on-demand JSON export emailed to verified address, per-device session list and revocation.
- Aggregate-only operational telemetry; per-bird interaction state never aggregated.

### 1.2 Out of scope (v1, refused at the architectural level)

- Native iOS/Android. No native code paths, no protocol changes for native clients, no abstractions left "in case."
- Gamification of every form: streaks, badges, scores, levels, XP, "days visited," green-dot calendars, milestone confetti, "you visited every day this week" notebook entries, exportable visit logs that read as activity logs. The architectural defense is that we never compute the underlying counters or store them in a queryable shape (see §4.5).
- Tamagotchi mechanics: no hunger, no death, no distress, no negative drift. The drift function (§6.3) physically cannot decrement personality traits.
- Social-network surfaces: profiles, follows, public discovery, leaderboards, comments, co-presence, shared cursors, mutual-friend chains, "show-off" rendering for visitors, "your friend visited" notifications by default. Visit data model is intentionally a small isolated table that cannot be joined into a social graph (see §4.4).
- Push notifications, transactional re-engagement email, marketing email beyond magic-link / export / account events, recorded-audio fallback.
- Native features that would require schema concessions: device-local personality state, client-authoritative writes, multi-aviary accounts.

### 1.3 Calibration commitments

- Drift: instrument-detectable change after ~1 week of regular visits; user-perceptible after ~3 weeks; never decrements (§6.3).
- Bird cap: hard 7. Starter count: hard 2. Cap is enforced both in service code and at the database constraint level.
- Tick cadence: 60s nominal, configurable in [30s, 120s] without code changes (§6.1).
- Time-to-first-bird: <500ms p95 on mid-tier mobile / 4G synthetic harness.
- JS bundle (initial, gzipped): <2MB enforced by CI.
- Idle motion: 60fps sustained on a 5-yr-old laptop reference rig.
- 30-minute session: zero net heap growth (CI test).

---

## 2. Architecture

### 2.1 Service shape

Three boundaries. Each is independently deployable and has its own failure mode.

1. **Edge / web tier** — static SPA bundle, server-rendered HTML shell with inline initial-state snapshot, served from a CDN edge. No business logic.
2. **API tier** — stateless HTTP service ("aviary-api"). Handles auth, snapshot reads, event-log writes, notebook reads, visit invites, account ops. Talks to the simulation database and the auth database.
3. **Simulation tier** — stateful tick worker pool ("aviary-sim"). Sharded by account UUID. Owns canonical personality state. Reads the event log, writes new aviary state and notebook entries.

Auxiliary services:

4. **Mailer** — sends magic-link, account-export, deletion-confirmation, revocation, and (only if user opted in) visit-notification emails.
5. **Narrator job** — out-of-band per-account job that decides whether to write a notebook entry, at low cadence (every few hours; minimum interval enforced).
6. **Synthetic perf monitor** — fleet of automated browsers running the aviary on schedule from common geographies.
7. **Telemetry collector** — aggregate-only metrics ingest. Hard separation from simulation database (§9.4).

### 2.2 Client/server split

The simulation engine lives entirely on the server. The client is a render+capture surface only.

- The client never computes drift, never derives mood, never authors personality state. It pulls snapshots and submits interaction events.
- The client owns: rendering the scene, synthesizing call audio, capturing presence events (as defined), capturing user gestures, generating call captions (because captions must match what was actually played, which only the audio path knows), maintaining the keyboard focus model, animating the top bar fade.
- The server owns: personality vectors, mood transitions, aviary timeline, narration prose for the screen reader, notebook entries, visit-invite state, account state.

This split is not a stylistic preference. It is the rule that makes "the aviary continues without the viewer" true and that makes multi-device sync trivial (§7).

### 2.3 Render pipeline boundary

One sentence: the server delivers an authoritative, time-stamped snapshot; the client renders a continuous interpolation between snapshots and never invents persistent state.

- The snapshot includes everything needed to render a frame in any session: per-bird perch / sub-perch position, idle-pose phase, current-call state, mood, plumage saturation; aviary-level day phase, weather; queued bird events with their timestamps (e.g. "Pip will call between t+2.0s and t+3.4s using motif M with parameters P"); narration prose for the next idle window.
- Between snapshots, the client interpolates motion and runs the call schedule the server queued. It does not invent new birds, new calls, new moods.
- The first frame the user sees uses the inline initial snapshot from the SSR HTML, bypassing any network round-trip on a warm cache.

### 2.4 Stack choices

- **Backend**: Go. Single binary, mature concurrency primitives, predictable latency for the tick loop, low ops cost. The same language for the API tier and the sim tier reduces engineering surface area; we are not large enough to justify polyglot.
- **Database**: PostgreSQL 16. One logical database per environment, one schema per concern (auth, simulation, telemetry tag), partitioned tables where the volume warrants (event log). Postgres handles v1 scale comfortably and matches the team's ops experience. We will not premature-introduce Kafka, NATS, or a CRDT store.
- **Cache / queue**: Redis for short-lived session and rate-limit state, and as the dispatch layer for narrator/tick triggers. Do not store canonical personality state in Redis.
- **Object storage**: S3-compatible bucket for outbound JSON exports.
- **CDN**: CloudFront / Fastly with edge HTML compute for inline-snapshot SSR.
- **Frontend**: TypeScript, React for chrome (top bar, settings, notebook, account, visit, error surfaces) and an internal `Aviary` Canvas2D component for the scene. Canvas2D is sufficient for the PRD's motion budget; WebGL is reserved for a later optimization if profiling shows we cannot hit 60fps on the reference laptop.
- **Audio**: WebAudio API directly. No audio library in the bundle; the synthesis nodes we need are small (oscillators, biquad filters, noise sources, gain stages, a tiny convolution reverb).
- **Build**: Vite + esbuild with a strict bundle-size budget gate (see §10.2). Service worker for snapshot caching and offline-first cold-load.
- **Auth crypto**: standard library plus a vetted JOSE implementation; magic-link tokens are opaque random IDs, not JWTs (§5.1).

### 2.5 Environments

Three: `local`, `staging`, `prod`. Staging mirrors prod's data model and tick worker shape but runs on smaller instances. Synthetic perf monitor runs against staging on every deploy and against prod on a continuous schedule.

---

## 3. Data model

All identifiers are synthetic UUIDs, never derived from email or user input.

### 3.1 Account

```
accounts
  id                uuid primary key
  email_encrypted   bytea            -- AES-GCM with KMS-managed key
  email_hash        bytea            -- HMAC-SHA256, single index for lookup
  status            enum {active, soft_deleted, hard_deleted}
  soft_deleted_at   timestamptz null
  created_at        timestamptz
  updated_at        timestamptz
  visit_notify_opt_in  boolean default false
```

`email_hash` is the lookup index; the plaintext is decrypted only inside the auth path. The encryption key rotates; data is re-encrypted lazily on next write or by a background re-key job. **Email never appears as an identifier elsewhere — not in logs, partitions, telemetry tags, queue keys, error messages, or filenames.** This rule is enforced by a logging filter and a code-review checklist (§10.4).

### 3.2 Bird

```
birds
  id                  uuid primary key            -- stable internal ID
  account_id          uuid references accounts(id)
  species             smallint                    -- enum index into species pool
  name                text                        -- user-assigned, renameable
  personality_vector  jsonb                       -- {boldness, social_warmth,
                                                  --  vocal_frequency, plumage_saturation,
                                                  --  curiosity}, each in [0.0, 1.0]
  current_mood        enum {wary, content, curious, drowsy, alert, settled}
  current_perch       enum {front, middle, back}
  perch_offset        real                        -- 0..1 along the perch
  mood_until          timestamptz                 -- earliest time this mood may transition
  adopted_at          timestamptz
  archived            boolean default false       -- never used in v1; reserved
  -- deliberately no last_drift_at, no streak_*, no level, no xp, no score
```

The stable `id` is the load-bearing invariant from `bird_engine.md` §"Bird identity." Renaming, syncing, or species-pool migrations never replace this row. There is no "regenerate bird" operation in any path.

The columns `streak_*`, `xp`, `level`, `score`, `visit_count`, `last_visited`, `days_active` etc. are explicitly absent. Their absence is enforced by the schema review process (§13.5) — if a future migration adds one, it must pass an explicit gamification review and the rule-of-the-product review.

### 3.3 Personality drift history (audit only)

```
personality_history
  id              bigserial
  bird_id         uuid
  applied_at      timestamptz
  delta           jsonb                          -- per-trait delta
  cause           text                           -- summary; e.g. "tick: presence_60s+listen_in_2"
```

This is for engineering inspection during calibration and incident response. **It is not exposed to clients, never aggregated across accounts, and never joined into product features.** A user-facing "your bird's drift over time" surface is explicitly out of scope; surfacing this would inflate personality into a number to optimize.

### 3.4 Interaction event log

```
interaction_events
  account_id      uuid
  seq             bigint                          -- monotonic per account
  event_type      enum {presence_ping, listen_in_start, listen_in_end,
                       offer_submitted, offer_accepted_observed,
                       settle_started, settle_undo, return_greeting_ack,
                       visit_session_open, visit_session_close}
  bird_id         uuid null
  payload         jsonb
  occurred_at     timestamptz
  received_at     timestamptz
  client_session  text                            -- per-tab UUID, not joined to user identity
  primary key     (account_id, seq)
  partitioned by  hash(account_id) into 64 partitions
```

Append-only. The server inserts with `seq = last_seq + 1` per account. The simulation tick reads `WHERE seq > last_consumed_seq ORDER BY seq` and updates `last_consumed_seq` in `aviary_runtime`. **Visit-session events (open/close) exist for the visit log only and are never used as drift inputs** — see §6.5.

### 3.5 Aviary runtime state

```
aviary_runtime
  account_id            uuid primary key
  last_tick_at          timestamptz
  last_consumed_seq     bigint
  current_weather       jsonb
  day_phase             enum {dawn, morning, midday, afternoon, dusk, night}
  timezone              text                       -- IANA tz, set at signup, editable in settings
  next_bird_offer_at    timestamptz                -- aviary-age-based, see §6.7
```

The `next_bird_offer_at` field is the only counter we maintain that touches "growth." It is computed from aviary age, not from interaction count. It is never surfaced to the user as a count or progress indicator; it is consumed once when the offer becomes available.

### 3.6 Notebook entries

```
notebook_entries
  id           uuid primary key
  account_id   uuid
  body         text                                -- naturalist prose, lowercase
  written_at   timestamptz                         -- when the narrator job wrote it
  observed_at  timestamptz                         -- the in-aviary moment it observes
  source       jsonb                               -- internal: which signals triggered
  index on (account_id, written_at desc)
```

Read-only to the user. `source` is for internal review of the narrator's voice (§8); it is never sent to the client.

### 3.7 Magic links and sessions

```
magic_links
  token           bytea primary key                -- 32 random bytes
  email_hash      bytea                            -- looked up at issuance
  created_at      timestamptz
  expires_at      timestamptz                      -- created_at + 15 minutes
  consumed_at     timestamptz null
  consumed_from   inet null

sessions
  id              uuid primary key
  account_id      uuid
  device_label    text                             -- best-effort UA fingerprint, user-editable
  issued_at       timestamptz
  last_seen_at    timestamptz
  revoked_at      timestamptz null
```

Magic-link tokens are opaque, single-use, indexed by their primary token. Per-email rate limit is enforced via Redis: 5 issuances per email per 30 minutes; 20 per IP per 30 minutes; soft pushback after.

### 3.8 Visit invites and sessions

```
visit_invites
  id                   uuid primary key
  host_account_id      uuid
  visitor_email_hash   bytea                       -- hash only; we do not store visitor email
                                                   -- in plaintext anywhere
  visitor_email_label  text                        -- the host's typed-in string, for the visit log;
                                                   -- shown only to the host, encrypted at rest
  token                bytea                       -- visit URL token, 32 random bytes
  created_at           timestamptz
  expires_at           timestamptz                 -- created_at + 30 days
  revoked_at           timestamptz null

visit_sessions
  id              uuid primary key
  invite_id       uuid
  started_at      timestamptz
  ended_at        timestamptz null
  ip_hash         bytea                            -- coarse abuse signal, not exposed
```

Visit traffic is never joined into the host's interaction event log. Visitor presence does not drift the host's birds. The architectural separation makes the "no co-presence, no aggregated visitor influence" rule (§social_optional, §6.5) enforced by topology, not by code review.

### 3.9 Telemetry tables (aggregate only)

In a separate analytics database, never reachable from `aviary-api` or `aviary-sim`:

```
metrics_events       -- request counts, latencies, error rates; tags are bounded
session_durations    -- anonymized histogram buckets only
audio_errors         -- error code, browser family, no account or bird identifier
render_frame_timings -- buckets only
sim_tick_latencies   -- per-shard, never per-account
```

The schema deliberately omits `account_id`, `bird_id`, and any column that could carry per-user state. The CI schema-lint check (§10.4) blocks PRs that try to add one.

---

## 4. API surface

All endpoints are HTTPS-only, JSON, versionless at the URL level (we will use header-based versioning if it is ever needed). Authenticated endpoints carry a session cookie.

### 4.1 Auth

```
POST /auth/magic-link         { "email": "..." }              -> 202
GET  /auth/consume?token=...                                    -> 302 to /aviary
POST /auth/sign-out                                             -> 200
GET  /auth/sessions                                             -> [ {id, label, last_seen} ]
DELETE /auth/sessions/:id                                       -> 200
```

`POST /auth/magic-link` always returns 202 regardless of whether the email is on file (response shape is identical for known and unknown emails — no email enumeration). Rate-limited per email and per IP. Magic-link consumption is one-shot; `consumed_at` is set inside the same transaction as session creation.

Errors on this surface use matter-of-fact voice (§voice split):

> We couldn't sign you in. The link may have expired. Try requesting a new link.

### 4.2 Aviary state

```
GET  /aviary/state            -> { snapshot_id, server_time, day_phase, weather,
                                   timezone, birds: [...], queued_events: [...],
                                   narration_prose: "...", reduced_motion_hint: bool }
POST /aviary/events           { events: [ {client_seq, type, payload, occurred_at}... ] }
                              -> { accepted: [...], rejected: [...] }
GET  /aviary/state/stream     -- SSE keep-alive stream emitting snapshot diffs while visible
```

The SSE stream is optional; clients fall back to a 30-second visibility-aware poll if SSE is unavailable. The diff payloads are tiny — kilobytes. The full snapshot is also reachable on demand.

`POST /aviary/events` accepts batches. The server assigns the canonical `seq`. The client's `client_seq` is included in the response only for the client's own dedup. The server is free to drop a batch entirely on overload, in which case the client retries with backoff; events are idempotent because the simulation tick reads them in order and an offer at t and an offer at t+50ms are allowed to be modeled either way.

### 4.3 Notebook

```
GET /notebook?cursor=...&limit=20   -> { entries: [...], next_cursor: ... }
```

Read-only. No POST, PATCH, DELETE. The lack of a write surface is the architectural enforcement of "the notebook is an observer's record, not a journal" (§interactions).

### 4.4 Visits

```
POST   /visits/invites               { visitor_email: "..." }      -> { invite_id, expires_at }
DELETE /visits/invites/:id                                          -> 200
GET    /visits/invites                                              -> [...]
GET    /visits/log                                                  -> [...]
GET    /visit/state?token=...                                       -> snapshot or matter-of-fact end surface
PATCH  /account/visit-notify         { opt_in: true|false }
```

The visitor surface — `GET /visit/state?token=...` — is on a cookie-less code path. It cannot read or write any other account's data. It cannot append to the host's event log. It returns the same snapshot shape as the host endpoint, minus narrator prose authored for the host.

When the host revokes, the next pull on the visitor side returns:

> This visit is no longer available.

(Matter-of-fact voice.)

### 4.5 Account

```
GET    /account                                                   -> { email, timezone, prefs }
PATCH  /account                  { display_name?, timezone?, ... }
POST   /account/email-change     { new_email: "..." }             -> 202 (verification email)
POST   /account/email-confirm    { token: "..." }                 -> 200
POST   /account/export                                              -> 202 (export emailed)
POST   /account/delete                                              -> 200 (soft-delete starts)
POST   /account/restore                                             -> 200 (within 30-day window)
GET    /account/prefs                                               -> { reduced_motion, captions, ... }
PATCH  /account/prefs            { reduced_motion?, captions?, ... }
DELETE /birds/:id/name           -- not exposed; renaming is PATCH /birds/:id { name }
PATCH  /birds/:id                { name?: "..." }
```

Note the deliberate absence: there is no `GET /account/streak`, no `GET /account/days-visited`, no `GET /account/stats`. The product does not offer these endpoints because the underlying counters are not computed (§3 schema). A future PR proposing to add such an endpoint must pass the rule-of-the-product review.

The account-export payload includes: bird records (with current personality vectors — this is the one place the user can see their numbers, and only in the export blob, not in any UI), notebook entries, visit log, account preferences, account creation date. The numbers are present in the export because the user's relationship is theirs to take, but the UI never renders them as a stat panel.

### 4.6 Errors

All client-facing errors use matter-of-fact voice on system surfaces. The error envelope is:

```
{ "error": { "code": "magic_link_expired", "message": "The link may have expired. Try requesting a new link." } }
```

Codes are stable across versions; messages are hand-written, plain English, capitalized normally. No naturalist phrasing here. No "oh dear" or "the aviary is napping" charm in error states; the user trying to sign in needs information, not flavor.

---

## 5. Auth

### 5.1 Magic-link details

- Token: 32 random bytes from `crypto/rand`, base64url-encoded in URLs.
- Storage: hashed token (SHA-256) in `magic_links.token`. The plaintext is never stored.
- Expiry: 15 minutes. Single-use. Race-safe consumption via `UPDATE ... WHERE consumed_at IS NULL RETURNING ...`.
- Issuance is rate-limited; issuance always returns 202 to prevent enumeration.
- Email body: matter-of-fact voice, single sentence + the link, no marketing copy.

### 5.2 Sessions

- Session cookie: HttpOnly, Secure, SameSite=Lax, 30-day rolling expiry.
- Per-device session list; user can revoke a session from settings; revocation is immediate (sessions are looked up on each request via a Redis cache backed by the `sessions` table; revocation invalidates both).
- Suspicious-session signal (different IP geolocation than recent activity) is *not* surfaced to the user as a notification — that would be an announcement surface in the wrong product. It is surfaced inside the device list page, when the user opens it.

### 5.3 Email change

Old email continues to work until the new email confirms. If the user never confirms, the old email remains canonical. The confirm token follows the same shape as a magic link.

### 5.4 Soft and hard delete

- Soft-delete sets `accounts.status = soft_deleted` and `soft_deleted_at = now()`. The user can sign in for 30 days and click restore on any signed-in page, which sets `status = active`.
- A scheduled job at 30-day boundaries marks accounts hard-deleted: drops birds, personality_history, notebook_entries, interaction_events, visit_invites, visit_sessions, sessions, magic_links, and finally the `accounts` row. The hard-delete job is single-tenant per account, idempotent, and emits an aggregate "hard delete completed" metric (no account identifier).

---

## 6. Simulation engine

The simulation engine is the spine of `feels alive over weeks`. It is the most carefully calibrated subsystem in the product.

### 6.1 Tick loop

- The simulation tier is a pool of stateless workers. Account work is sharded by `hash(account_id) mod N`.
- A scheduler maintains a per-account `next_tick_at` and pushes due accounts onto a Redis-backed work queue. Workers claim a per-account lock (Redis `SETNX` with TTL), run a tick, and release.
- The lock prevents two workers from ticking the same account concurrently and is the operational backstop on the "server is the only writer" rule.
- Nominal cadence: 60 seconds. Tunable in config without redeploy. Per-account cadence may slow for accounts with no active session and no recent events (down to 5–10 minutes) to reduce idle work.
- Each tick's transaction reads `last_consumed_seq`, the slice of new events, and the bird records, computes deltas, writes new bird records, advances `last_consumed_seq`, advances day/weather, queues call schedule for the upcoming snapshot window, and may emit a "narrator candidate" signal to the narrator job.

The `aviary continues without the viewer` claim is now a real property: the tick runs even on accounts with no clients connected, advancing day phase and applying any drift from prior events still in the log.

### 6.2 Personality vector

Per bird, five scalar traits in [0.0, 1.0]:

- `boldness`
- `social_warmth`
- `vocal_frequency`
- `plumage_saturation`
- `curiosity`

Stored as `jsonb`, validated on read and write. The vector seed at adoption is drawn from a per-species distribution centered around the species' "default" personality, with mild per-bird variance, so two robins do not feel identical. The seed is persisted and never re-rolled.

The vector is **never exposed numerically** to the client. The client receives `plumage_saturation` only because it is a render parameter, and only as the saturation factor applied to the sprite's color ramp — the number itself is not labeled, displayed, or made readable from the DOM.

### 6.3 Drift function (the calibration core)

Drift is a bounded, monotonic-up, low-pass filter over presence-and-interaction signals.

For each tick, per bird, per trait:

```
delta_trait = sum over events e in this tick window of:
                gain(trait, e.type) * weight(e, bird)
              + presence_drift(bird, presence_seconds_in_window)

new_trait = clamp(old_trait + max(delta_trait, 0), 0.0, 1.0)
```

Notes:

- `max(delta_trait, 0)` is the structural enforcement of "drift never decrements." There is no code path that subtracts from a trait. (If a future change wanted to subtract, it would have to remove the `max`, which is a deliberate one-line change that triggers code-review attention.)
- `gain(trait, event_type)` is a small constant table: e.g., `(social_warmth, listen_in_observed_seconds)` has gain 1e-4 per second; `(boldness, offer_submitted)` has gain 5e-4; `(curiosity, offer_accepted_observed)` has gain 1e-3. The exact numbers are owned by the calibration test harness (§6.4) and can be tuned without code changes via a server-side config table.
- `presence_drift` is the dominant input — see `concepts.md`. It is shaped by the strict presence definition (§7.4): a tick window's presence_seconds is the sum of presence-validated seconds reported by the client, never a "tab open" approximation.
- `plumage_saturation` drifts up only on sustained presence with the bird visible on the front or middle perch. A bird on the back perch does not gather plumage drift from a passing user — this preserves the link between attention and visible richness.

Drift is **monotonic-up by construction.** This is the single most important design rule in the engine and it is named in code with the exact comment:

> // Drift is monotonic toward expressive. A bird that is ignored becomes
> // ambient; it does not become wary, silent, or dull. Do not change this
> // without rule-of-the-product review.

The clamp is also intentional: drift saturates at 1.0 per trait so a heavily-attended bird does not exceed the species range. We accept the saturation tradeoff because the user noticing a bird "stop changing" after months is a much smaller failure mode than runaway drift breaking species recognizability (§audio).

### 6.4 Calibration test harness

The drift function is calibrated against named targets. The test harness runs simulated user traces against a temp-database with the production tick code and verifies:

- After 7 simulated days of "regular visits" (defined: 15 minutes presence per day, one offer per day, two listen-in events per day), at least one trait per bird has changed by >= 0.02 (instrument-detectable).
- After 21 simulated days of regular visits, at least one trait per bird has changed by >= 0.10 (user-perceptible).
- After 30 simulated days of zero presence and zero events, every trait is exactly equal to its day-0 value (no decay).
- After 30 simulated days where the user is *only* in `listen_in` mode on Pip, Pip's `social_warmth` has drifted >= 0.08 and no other bird has drifted by more than 0.03.

These thresholds are chosen so the calibration is named, defensible, and changeable. They are checked in CI on every PR that touches the drift function or the gain table.

### 6.5 Mood

Mood is a small enum: `wary, content, curious, drowsy, alert, settled`.

Per tick, per bird:

1. Each mood has a `cooldown` (e.g., the bird stays in `wary` for at least 90 seconds before another transition is considered). This is `mood_until` on the bird row.
2. Inputs to the mood transition function:
   - personality vector (a `boldness`-high bird is far less likely to transition to `wary`),
   - last few interactions (an offer_accepted in the last 30s pushes toward `content`/`curious`),
   - time of day in the user's local timezone (drowsy near dusk, alert in early morning, settled at night),
   - ambient events (rain → vocal frequency dampened across the aviary, slight push toward `wary` for low-boldness birds),
   - neighbor mood (an alarm call from one bird raises P(wary) for nearby birds; a sustained `content` neighbor mildly pulls a bird out of `wary`).
3. The transition is computed as a probability distribution; one option is sampled with a deterministic per-bird, per-tick PRNG seed (so reproducible across replay). If the sampled mood differs from current and the cooldown has expired, transition.

Mood persists across sessions. When the client opens the tab, the snapshot includes the bird's current mood; the client never resets to a "default" mood. The "no snap" experience is enforced because the client has no logic to set mood — it only renders what it receives.

**Settle gesture:** a client-emitted `settle_started` event causes the tick to transition all birds to `settled` (or `drowsy` for high-`alert` birds), shift `aviary_runtime.day_phase` toward dusk over a short interpolation window, and quiet the call schedule. A `settle_undo` event within 5 seconds reverses these. Settle is not a drift input beyond cleanly closing the presence window.

**Visit-session events** (`visit_session_open`, `visit_session_close`) are stored in the visit log table only (§3.8) and **never injected into the drift or mood pipelines.** They are filtered out at the simulation tier's event reader. The host's drift comes from the host's presence only.

### 6.6 Call grammar runtime

Each species ships with a small motif library: 4–8 motifs, each motif being a sequence of (note, duration, pitch_envelope, timbre_params). Motifs combine and vary at runtime.

The server's tick produces a "call schedule" for the next snapshot window: per-bird, ordered list of `(start_time, motif_id, parameter_seed, duration)`. The `parameter_seed` is a per-emission random seed the client uses to vary pitch, timing micro-jitter, and timbre. The seed makes calls non-identical even between the same bird's same motif.

Call frequency is shaped by `vocal_frequency` and current mood:

- `drowsy` and `settled`: very low frequency, low-pitch motifs only.
- `wary`: short, sparse, low-amplitude motifs.
- `content`: idle motif library, medium frequency.
- `curious`: more variation, more upward-pitched motifs.
- `alert`: short, bright motifs, sometimes alarm motifs.

A bird's motif library is *signature*: even at high drift, even across moods, the underlying motifs are identifiably the same family. A user who has spent two weeks with Pip should be able to identify Pip's call from Wren's by ear at any drift value within the species range. This is what makes the cap of seven possible (§audio).

Two birds in `content` or `curious` calling in overlapping windows produce a **chorus** — the call schedule places them with offsets so the calls fold into one another rather than colliding. Chorus emergence is detected by the tick (two birds with high `vocal_frequency`, both not in `wary`, in the same tick window) and lightly nudges their schedules toward overlap rather than scheduling independently.

### 6.7 Bird-to-bird interactions

Modelled as small influences during the tick:

- A bird in `wary` mildly raises P(wary) for adjacent birds the next tick.
- A bird with high `social_warmth` whose neighbor is in `wary` slightly accelerates the neighbor's recovery toward `content`.
- An alarm-call event (rare; emerges from `alert` mood + low-`boldness` bird seeing a sudden ambient shift) pushes nearby birds toward `wary` and produces a short cluster of responses.

The chosen weights are small. The aim is for these effects to be visible over a few minutes, not a single tick.

### 6.8 New-bird offers

`aviary_runtime.next_bird_offer_at` is set at adoption to a future timestamp shaped by aviary age curve (e.g., first additional bird offered at ~8 weeks, second at ~16 weeks, then slower). It is **strictly age-based** — interaction count, presence-time, and notebook entries do not factor in. The pacing curve is configurable; we expect to tune it in the first months of operation.

When the offer time is reached, the next snapshot includes a small `pending_bird_offer` flag. The UI surfaces it as an unobtrusive prompt in the top bar (no toast, no modal-on-load): "a new species has been spotted nearby." The user can accept and name the bird, or dismiss; dismissed offers re-surface gently after a long delay (weeks, not days).

The cap of 7 is enforced both at the offer-generation step (no offer is generated for an aviary already at 7 birds) and at the database layer (`CHECK COUNT(birds WHERE account_id = X AND archived = false) <= 7`).

### 6.9 Day, night, and weather

Per tick, the simulation reads the user's timezone from `aviary_runtime.timezone` and computes `day_phase`. Sunrise / sunset times are approximated to the user's latitude only if we have it (we do not at v1; we'll use a fixed sunrise/sunset by month and a single configurable latitude per account — defaulting to the user's signup IP geolocation as a coarse hint, which we drop after first use).

Weather is a small Markov chain per aviary, advanced once per tick. The chain is heavily biased toward "clear"; transitions to "light rain" or "soft wind" are rare (a few times a week, per the spec). Weather state is part of the snapshot.

Night is not a dead state: one species in the pool — a nightjar-like — has a `nocturnal` flag that flips its mood probabilities so it stays alert and may call into the late hours.

### 6.10 The narrator job

A separate worker per account at much lower cadence (every few hours). For each account it considers:

- Are there any noteworthy moments in the recent tick history? (First-greet ordering changes; sustained mood unusual for this bird; first chorus event of the week; first offer accepted by a long-wary bird; weather event with response.)
- Has it been at least 36 hours since the last entry? (Unless a strongly noteworthy event triggers an exception.)
- If yes, generate one entry using a deterministic prose composer (§8) drawing from a curated phrase library and the current state. *No LLM in v1.* The prose is hand-shaped so the voice is invariant across millions of entries.

Entries are written to `notebook_entries`. A user with very active visits gets entries no more frequently than ~once every two days, rising slightly when the aviary is doing something notable.

The narrator never writes about the user. It writes about the aviary. The narrator is forbidden — by the phrase library's structure, by its inputs, and by code review — from generating any sentence that contains "you," "your," "today you," "every day," "every week," "since last," "X days," or any variant. This is the architectural enforcement of "the line is between observations of the aviary and observations of the user's behavior" (§interactions).

---

## 7. Sync model

### 7.1 Single canonical aviary

There is one source of truth per account: the rows in the simulation database. Every client renders that state.

### 7.2 Snapshot pull

Clients pull a snapshot:

- on tab visibility change to visible,
- on long render-frame gap (covers laptop suspend/resume),
- on a slow keepalive while the tab is visible (every 30s nominal),
- on receiving a server-sent event that the snapshot has changed (SSE, optional).

Snapshots are small (typically a few KB). The endpoint sets cache-control to `no-store` because the snapshot is account-specific.

### 7.3 Event submission

Clients buffer interaction events locally (with `client_seq` and `occurred_at`) and POST them to `/aviary/events` in batches with backoff on failure. The server assigns `seq` atomically in the tick database and returns the assigned `seq` for the client's bookkeeping.

Loss handling: if the network drops, events buffered locally during the loss window are re-sent on reconnect; the server is order-tolerant within a window (events arriving slightly out-of-order are sorted by `occurred_at` before tick consumption). Events older than 5 minutes are accepted but logged as `late_event` for telemetry.

### 7.4 Presence accounting

Presence is the dominant drift input and its definition is precise (per `concepts.md`):

> A presence-event is recorded when, simultaneously, the document's `visibilityState` is `visible`, the document has window focus, and a `pointermove` or `keypress` has occurred in the last few minutes.

The client emits a `presence_ping` event every 60 seconds while all three conditions hold. The activity-window threshold for "recent pointer/keypress" is **3 minutes** (we will tune in [2, 5] minutes during build, leaning long because watching birds without moving is the actual product). The threshold value lives in client config and is shipped in the SSR HTML so it can be changed without a deploy.

Each `presence_ping` carries the number of seconds in the window during which all three conditions held (typically 60). The simulation tick uses the sum of these seconds, not the count of pings.

Two consequences:

- Tabs in background windows stop emitting presence pings within a few seconds of losing visibility/focus. The simulation never sees them.
- A laptop left open with the aviary tab in front but the user away stops emitting pings within the activity-window threshold.

We do not depend on the client's honesty here for fairness reasons (there are no rankings to fake), but we do server-side sanity checks: presence_seconds in a tick window cannot exceed the wall-clock window. Bursts above are clipped and logged.

### 7.5 Multi-device coherence

Because the server is the only writer of personality and the only authoritative source of mood and call schedule, two devices signed into the same account see the same aviary at the same moments.

The two device clients each:

- pull their own snapshots,
- emit their own presence pings (the simulation receives both streams),
- emit their own interaction events (also both streams).

If the user offers a seed on the laptop and immediately performs a listen-in on the phone, both events land in the event log and the next tick processes them in `seq` order. There is no merge to do because the server is reconstructing the next state from a serial event log; concurrency does not produce divergence.

### 7.6 No last-write-wins

The personality vector is **never** written by the client. Only the simulation tick writes it, only via additive deltas (§6.3). This rule is enforced at three layers:

1. The schema: `birds.personality_vector` is updated only by the simulation service; its database role is the only role with `UPDATE` permission on this column. The API service has only `SELECT`.
2. The API: `PATCH /birds/:id` accepts only `name`. There is no endpoint to set personality.
3. The simulation: the tick computes deltas from events, never from a client-supplied vector.

If a future endpoint accidentally exposes a vector write, it will fail at the database role boundary. This is the structural defense for the failure mode named in `accounts_sync.md` ("a last-write-wins model on personality state would let one device overwrite drift recorded from a previous session on another device").

### 7.7 Conflict surfaces

Genuine conflicts at v1 are rare and bounded to auth surfaces (replayed magic link, expired session, dual sign-in collisions). The user sees matter-of-fact errors:

> Your session timed out. Sign in again to keep watching.

> Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch.

There is no aviary-state conflict surface, by design — the architecture does not produce them.

---

## 8. Frontend rendering pipeline

### 8.1 Composition

The frontend is one SPA. Rendering is split:

- **React** owns: the top bar, settings, notebook, account, visit, sign-in, error surfaces, captions overlay, narration live region.
- **Aviary canvas** owns: the scene — sky, foliage, perches, birds, weather, ambient micro-motion, leaf drift, focus indicators on birds.

The Aviary canvas runs its own animation frame loop (`requestAnimationFrame`) bound to the snapshot. React never re-renders the canvas; it only signals transitions via props (e.g., reduced-motion mode switching).

### 8.2 First-frame strategy

The HTML shell rendered at the edge contains an inline `<script type="application/json">` with the initial snapshot for this account. The Aviary canvas reads it on init and renders the first frame as the aviary in motion at the snapshot's time, with no intervening blank state. This is what hits the <500ms time-to-first-bird budget.

If the inline snapshot is missing (cold-start failure, stale CDN edge), the client falls back to a **quiet field** — the soft sky color, slow ambient cloud, no spinner, no skeleton, no progress bar — and pulls a fresh snapshot in the background. The quiet field reads as the aviary catching up; a spinner here would compromise the central conceit and is forbidden in code review.

### 8.3 Scene composition

- Layers (back → front): sky gradient, distant foliage, mid foliage with parallax factor 0.3, perches, birds, foreground branch (subtle parallax 1.2), ambient leaf drift overlay, weather overlay (raindrops, wind gusts).
- Birds render as composed sprites: the species-shaped silhouette + plumage layers + eye + beak + procedural feather detail. Plumage saturation modulates the color ramp applied to feather layers.
- Idle micro-motion: each bird has a small set of named loops (`preen`, `scan`, `head_tilt`, `weight_shift`, `feather_fluff`) that the renderer interpolates between. The active loop is mood-keyed (a `wary` bird uses `scan` and `weight_shift`; a `content` bird uses `preen`). Loop phases are time-driven so two loops running simultaneously don't sync up like a metronome.

### 8.4 Transitions

- Bird perch-to-perch transitions use a fly-arc. The arc duration is shaped by personality (`boldness`-high birds fly more directly, `boldness`-low birds detour and pause).
- Mood transitions are smooth across a few seconds: a bird going `wary → content` slowly relaxes posture, scan rate drops, perch may shift forward. There is no "mood snap."
- Day-phase transitions are continuous color ramps over minutes, not stepped fades.
- Settle: a 6–8 second slow ramp toward warm evening light, calls quiet over the same window.

### 8.5 Top bar

A thin React component with five icons: account, accessibility, notebook, offer, settle. After 3 seconds of cursor stillness (and no keyboard activity), opacity ramps to ~10%. On any pointer movement or keyboard activity, opacity returns to 100% over 200ms.

The top bar is never fully invisible; it remains discoverable. The fade is the small affordance cost of having controls on the surface.

### 8.6 Reduced-motion mode

If `prefers-reduced-motion: reduce` is set, **or** the user opts in via accessibility settings, the renderer switches to the reduced-motion pipeline:

- Idle micro-motion is replaced by slow cross-fades between still poses (`preen-frame-1 ↔ preen-frame-2 ↔ scan-frame-1 …`), interval ~8 seconds per pose.
- Flight transitions become cross-fades between perch positions over 1.5 seconds — no animated path.
- Ambient leaf drift is removed.
- Day-phase color ramps remain, slowed to ~2× duration.
- Weather is rendered as gentle ambient color shift (no rain particles, no wind animation), but still listed in narration / captions.
- Calls play at full quality (or caption, per user's audio settings).
- Drift, mood, notebook, and all engine behavior remain unchanged.

This is not a fallback. It is its own designed surface. The cross-fade pose set is curated by the visual designer; we do not auto-derive it from the animation rigging.

### 8.7 Visit (read-only) rendering

The visitor's client is the same SPA but in a constrained mode:

- No interaction surfaces (offer, listen-in, settle, notebook are hidden or disabled with matter-of-fact tooltips).
- No event submission (the API path is blocked by the visit token).
- No narration prose is requested (visitor accessibility surface is a separate consideration; v1 ships visit with same captions and same screen-reader narration of the *aviary* as the host, which is fine because the aviary is what is being shared).

When the visit invitation is revoked or expired, the next snapshot pull returns a `visit_terminated` envelope and the client navigates to the matter-of-fact "this visit is no longer available" surface.

### 8.8 Empty / loading states

- **Empty aviary** between adoption and first bird arriving: the quiet field, plus the soft fly-in of bird #1 to its starting perch.
- **Unsupported browser**: the matter-of-fact unsupported-browser page lists supported browsers ("the last two major versions of Chrome, Safari, Firefox, and Edge") and a link to a fuller compatibility note.
- **Audio context unavailable**: the aviary plays in graceful silence with captions enabled by default. A small one-time matter-of-fact note explains.

### 8.9 Forbidden patterns

The following are explicit code-review failures:

- Spinners, loaders, skeleton screens, "loading…" text anywhere in the aviary view.
- Toasts or banners welcoming the user.
- Pop-ups celebrating any milestone.
- Any text containing "Welcome back," "you've been here," "you visited," "X days," "your streak," "achievement," "level up."
- Any UI surface that displays personality-vector numbers.
- Counters of any kind in the UI: bird count badge, days-active, visits-today.

These are encoded as a lint check on JSX strings in CI (§10.4).

---

## 9. Audio pipeline

### 9.1 Synthesis

A single shared `AudioContext` is created on first user interaction (browser autoplay policies; the magic-link click qualifies, otherwise a quiet "tap to enter the aviary" matter-of-fact prompt is shown). Every per-bird voice is a node graph rooted in this context.

Per-bird voice graph (typical):

```
[Oscillator(s)] ─┐
                ├─> [BiquadFilter (bandpass, mood-shifted)]
[Noise burst] ──┘
                       │
                       v
                [Gain envelope, motif-driven]
                       │
                       v
                [Per-bird gain (listen-in mix)]
                       │
                       v
                [Aviary-level reverb send] -> [ConvolverNode (small impulse)]
                       │
                       v
                [Master gain] -> destination
```

The motif is parameterized by the call schedule from the server. The client's audio engine reads the schedule, schedules envelope triggers in the `AudioContext`'s timeline, and lets the Web Audio scheduler do the timing.

### 9.2 Procedural-only

There is no recorded audio in the bundle. There is no recorded fallback path, even for offline mode. If WebAudio is unavailable, the aviary plays in graceful silence with captions on.

### 9.3 Listen-in mix

When the user focuses a bird (click / tap / keyboard Enter):

- The focused bird's per-bird gain ramps from 1.0 to 1.6 over 800ms.
- All other birds' per-bird gains ramp from 1.0 to 0.4 over 800ms.
- Aviary-level reverb send is slightly increased on the focused bird, slightly reduced on others.
- The narration live region announces the bird being focused (matter-of-fact, in-context: "listening in on pip" — naturalist voice still fits because we are still in the aviary surface).

Other birds **do not go silent.** This is enforced by the minimum gain floor (0.4) and is named in the audio engine code.

Disengage on: clicking the focused bird again, focusing a different bird, clicking empty aviary space, moving keyboard focus away. Same 800ms ramp back to ambient.

### 9.4 Chorus mixing

Two or more birds calling in overlapping windows produce a chorus naturally because each is procedural; phase-cancel artifacts characteristic of looped audio do not appear. The aviary-level reverb send unifies the calls into a shared acoustic space. A soft master limiter prevents transient peaks when many birds call at once.

### 9.5 Caption generation

Captions are derived from the same motif spec the audio is synthesizing from. As each call is scheduled, the client pulls a caption phrase from a per-motif phrase template:

- motif "two-note rise" → "a soft two-note rise"
- motif "low trill, paused, low trill" → "a low trill, paused, low trill again"
- motif "single sharp call" → "a single sharp call from the back perch"

Captions render as small text near the calling bird, fade in over the call's start, fade out at the call's end. Captions are also written to the narration live region for screen-reader users (lower priority than user-initiated event narration; deduped to avoid overwhelming).

Captions use the same naturalist voice as the rest of the surface. The phrase templates are written by the same editor who writes the notebook prose, ensuring voice continuity.

### 9.6 Memory discipline

- AudioBuffers are reused; a small pool of envelope nodes is recycled rather than allocated per call.
- The convolver IR is loaded once; never per-call.
- Every scheduled node has a `node.onended = () => recycle(node)` lifecycle hook, verified by a leak test.

The "no memory growth over 30 minutes" rule (§10) requires this discipline.

### 9.7 Edge cases

- **Audio context suspended** (browser tab loses audibility, OS-level mute): we don't resume on tab-hidden — there's nothing to play to. We resume on tab-visible + user interaction.
- **Permission denied**: graceful silence with captions on.
- **Hardware audio failure**: graceful silence; we log an `audio_error` aggregate metric.

---

## 10. Accessibility surfaces

### 10.1 Screen-reader narration

The narration live region is a `aria-live="polite"` div in the React tree, updated from a server-derived `narration_prose` field on the snapshot. The server produces this prose on each tick using the same naturalist phrase composer used for the notebook (§6.10), at a much higher cadence (every 30–60 seconds) but with much shorter outputs.

Example narration:

> a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

User-initiated events get a priority bump:

- Return-greeting on session start: narrated within the first 1.5 seconds. ("pip glances up from preening and calls once. wren watches from the back perch.")
- Offer: narrated as the bird approaches or doesn't ("pip steps forward to the seed and tilts her head.").
- Settle: narrated quietly ("the light begins to soften. the calls quiet to ambient.").

The narration is **not state-list** (`Pip mood: content`). It is naturalist prose. This is enforced by the prose composer (§13.3) and reviewed by the editor before any new template ships.

### 10.2 Captions

(Detailed in §9.5.)

Captions are user-toggleable in accessibility settings and are **on by default** when the audio context is unavailable. We may also default-on captions for users whose browser exposes a "captions preferred" media query if such a thing standardizes; for v1, opt-in.

### 10.3 Keyboard navigation

- `Tab` / `Shift+Tab` cycles top bar → enters aviary scene → cycles birds left-to-right → wraps.
- `Arrow Left/Right` while in the aviary scene cycles between birds.
- `Enter` on a focused bird engages listen-in. `Enter` again or `Escape` disengages.
- Top-bar shortcuts:
  - `O` opens the offer affordance (when top bar is visible or accessibility "always show top bar" is on).
  - `S` triggers settle.
  - `N` opens the notebook.
  - `,` opens settings.
- Focus indicator: a soft outline ring with ~4:1 contrast against any aviary background. The visual designer specifies the exact treatment; the renderer composites the ring above the bird sprite layer.

### 10.4 WCAG AA contrast

- All user copy in the top bar, settings, notebook, account surfaces, captions, error surfaces meets WCAG AA contrast (4.5:1 for body, 3:1 for large) on every aviary background state (dawn, midday, dusk, night, weather).
- The contrast is verified per-state in CI by rendering the aviary at known states and checking computed contrast ratios on top-bar text against the pixel directly under it (or a designed background plate behind chrome — the designer's call).
- The aviary scene itself does not contain user copy except in the top bar, captions, and narration overlay. The rest of the scene is visual.

### 10.5 Settings surface

`/settings/accessibility` exposes:

- Reduced motion (auto / on / off)
- Captions (on / off)
- Always show top bar (off / on)
- Narration cadence (default / faster / slower) — stored in account prefs and respected by the snapshot's narration window.

Accessibility settings use matter-of-fact voice. The setting labels are direct ("Show captions for bird calls"), not naturalist ("notice the calls in writing").

---

## 11. Performance budgets and observability

### 11.1 Bundle budget

- Initial JS bundle ≤ 2MB gzipped, including: React, Aviary canvas engine, audio engine, top bar, caption renderer, narration manager.
- Code-split: account settings, accessibility settings, visit invitation flow, notebook scrolling, account-export confirmation, sign-in flow.
- CI gate: bundle-size budget enforced per chunk; any PR that grows the initial bundle past 1.8MB requires explicit sign-off (we leave 200KB headroom for hot-fixes).
- No `lodash`, `moment`, `chart.js`, or other large general-purpose libraries. `date-fns` (tree-shaken) only if needed.

### 11.2 Time-to-first-bird budget

- Target: <500ms p95 from navigation to first bird visible on a mid-tier mobile device over 4G.
- Inputs:
  - Edge-rendered HTML with inline initial snapshot (no extra round-trip).
  - Critical CSS inlined; non-critical CSS deferred.
  - Aviary canvas init runs synchronously on first paint; bird sprites are inlined as compact procedural data (paths + parameters), not network-fetched assets.
  - WebAudio context is created on first interaction, not on page load — audio latency does not block first bird visible.

### 11.3 Idle frame rate

- Target: 60fps sustained during a 30-minute session on a 5-yr-old laptop reference rig (Intel HD 620 class GPU).
- Enforcement:
  - Aviary canvas uses pre-rendered glyph atlases for static foliage.
  - Particle effects (rain, leaf drift) capped at small counts; pooled.
  - Layout reflows are forbidden in the inner animation loop.
  - Synthetic perf monitor (§10.6) runs 30-minute soak tests against the reference rig profile.

### 11.4 Memory budget

- "No memory growth" CI test: a Puppeteer harness runs the aviary for 30 minutes (compressed wall-clock via simulation override), measures heap before and after, fails the build if delta > 5MB after GC.
- Audio nodes pooled (§9.6).
- Notebook entries scrolled out of view detach their DOM nodes (virtualized list).

### 11.5 Tick budget

- Tick latency p99 < 200ms per account on the production tick worker.
- Alert if p99 exceeds 5 seconds (per `accessibility_perf.md` error budget).
- Backpressure: if the work queue depth exceeds N, individual account cadences slacken (§6.1) before any user-facing degradation.

### 11.6 Observability

- Aggregate metrics (RED + USE per service): request rate, error rate, latencies, CPU, memory, DB pool usage.
- Synthetic monitor: time-to-first-bird and first-call latency from common geographies, every few minutes.
- Aggregate session-duration histograms (no per-account dimension).
- Audio error counts by browser family (no account or bird identifier).
- Tick latency, tick queue depth, tick re-attempt rate per shard.
- Hard rule: **no per-bird, per-account, or per-interaction state** flows into the telemetry pipeline. Telemetry tags are bounded enums; code-review checklists reject high-cardinality tags. The analytics database has no foreign-key path back to accounts or birds.

### 11.7 Browser support

Last two major versions of Chrome, Safari, Firefox, Edge. Older browsers receive a matter-of-fact unsupported-browser page. We do not maintain compatibility shims. We do test on the four supported browsers in CI on every release candidate.

---

## 12. Voice and content discipline

The PRD's voice split is load-bearing and pervasive. It is enforced as engineering discipline:

### 12.1 Two voice surfaces

- **Naturalist** (lowercase, present-tense, specific, observational): aviary, notebook entries, narration, caption text, offer prompts, post-offer reactions, return-greeting flavor (narration only, never as a banner).
- **Matter-of-fact** (capitalized, direct, plain): sign-in, sign-out, account settings, accessibility settings, error surfaces, sync error surfaces, billing (none in v1), email-change, account-export, account-delete, browser-unsupported, audio-unavailable, visit-revocation.

### 12.2 Phrase library

A central phrase library is owned by the editor (single human owner). All naturalist prose — narration templates, notebook templates, caption templates, offer-reaction templates — is in this library. Engineering does not write product prose ad-hoc.

### 12.3 Forbidden surfaces

The following are forbidden everywhere in the product:

- Welcome banners or toasts.
- Achievement / badge / level / streak surfaces.
- "You've been here X days" / "You haven't been here in a while" / any second-person observation about the user's behavior.
- Per-bird stat panels.
- Any UI element that displays a number labeled as a personality trait.
- Marketing email beyond auth and account ops.

These are enforced by:

- A JSX content lint that forbids the trigger phrases in source strings.
- A schema lint that forbids streak-shaped columns.
- A pre-launch and pre-feature rule-of-the-product review (§13.5).

### 12.4 Naturalist voice as discipline

Lowercase by default in product surfaces. Present tense. Specific. The narration says "a warbler perches on the high branch, calling softly" — not "warbler perched at high branch." A draft that reaches for state-transition phrasing is rewritten before merge.

---

## 13. Privacy, compliance, and engineering boundaries

### 13.1 Email isolation

Email lives encrypted in `accounts.email_encrypted` and as a hash in `accounts.email_hash` for lookup. Email never appears as an identifier elsewhere — not in URLs, partition keys, queue keys, log lines, telemetry, error messages. This rule is enforced by:

- A logging filter that scrubs anything that looks like an email pattern, with alerts on triggers.
- A code-review checklist line item: "does this code path touch email outside the auth boundary? no."
- Quarterly audits of telemetry dashboards (manual; cheap insurance).

### 13.2 Data isolation between simulation and analytics

The simulation database and the analytics database are two separate Postgres clusters. They have no replication relationship. The analytics ingest pipeline reads from a small, schema-locked allowlist of metrics topics emitted by the API and sim services. The pipeline does not have credentials to read from the simulation database directly. This is the topology that enforces "per-bird state never appears in telemetry."

### 13.3 Account export

`POST /account/export` enqueues a job; the job composes a JSON blob of the user's data, writes to S3 with a 24-hour signed URL, emails the link to the verified address. The blob includes personality vectors (numerically — this is the only place numbers are exposed; see §6.2). The S3 object is auto-deleted at expiry.

### 13.4 Hard delete

After 30 days in soft-delete, a scheduled job hard-deletes the account: birds, personality_history, notebook_entries, interaction_events, visit_invites, visit_sessions, sessions, magic_links, accounts. The job is idempotent and emits an aggregate "hard delete completed" metric (no identifier). Backups follow the privacy commitment: hard-deleted records are dropped from incremental backups within 90 days; full snapshots roll over the same way.

### 13.5 Rule-of-the-product review

A small named review process — engineering lead + product + editor + designer — gates any feature, schema, or copy change that touches:

- A new counter or aggregate over user behavior.
- A new notification, push, or email surface.
- A new social surface.
- Drift function changes.
- Anything that exposes personality numerics in the UI.
- Anything that introduces a recorded-audio path.

This is the explicit defense against the "just one streak counter" foothold (`non_goals.md`).

---

## 14. Rollout plan

### 14.1 Build phase (months 0–6)

Milestones, in roughly this order:

1. **M1 (week 4)**: scaffolding — repo, CI, environments, auth, magic-link flow, sessions, account creation, single bird record. No simulation engine yet. Nothing user-visible.
2. **M2 (week 8)**: simulation engine v0 — one tick worker, drift function, mood transitions, snapshot endpoint. No audio. Renderer shows a static silhouette per bird that updates perch on snapshot.
3. **M3 (week 14)**: audio engine v1 — procedural call synthesis, motif library for 3 species, listen-in mix, captions. Renderer adds idle micro-motion for those species.
4. **M4 (week 18)**: full species pool (6 species), aviary scene assets (foliage, perches, day/night, weather), top bar with notebook + offer + settle.
5. **M5 (week 22)**: notebook narrator, screen-reader narration, reduced-motion mode, full keyboard nav.
6. **M6 (week 26)**: visit invitation feature, account export, account delete, settings surfaces.
7. **M7 (week 28)**: synthetic perf monitor, calibration test harness wired into CI, full perf-budget pass.

We do not ship to anyone until M7 is green. We do *not* ship a v0 with broken audio and call it a public beta — the audio is the affective spine and shipping it broken risks the "feels alive" core.

### 14.2 Closed beta (months 6–8)

- Invite ~50 people. Personal invitations, not a public sign-up.
- All accounts start at 2 birds; the new-bird offer pacing is artificially accelerated for beta accounts (a third bird offered after ~2 weeks instead of ~8) to surface the offer experience for review.
- Calibration band check: the calibration test harness is rerun nightly against beta data (with PII isolation respected — the test extracts only aggregate drift distributions, never per-account state).
- Editor reviews a sample of narrator-generated notebook entries weekly for voice consistency.

### 14.3 Public launch (month 9+)

- Open sign-up; new-bird pacing on the natural curve.
- Synthetic perf monitor running continuously from common geographies.
- Privacy review and accessibility review pre-launch (gate).
- Announcement is small; we are not building for a hype cycle.

### 14.4 Feature flags

- Visit feature: feature-flagged off in code initially; turned on once the visit-invite flow and revocation are robust and the visitor read-only path is confirmed not to leak host event-log writes.
- Calibration knobs: server-side config only, not surfaced as feature flags. Editable without redeploy.

### 14.5 What we instrument from day one

- Aggregate request, error, latency metrics on `aviary-api` and `aviary-sim`.
- Synthetic time-to-first-bird from a few geographies.
- Tick latency and queue depth.
- Audio context error counts by browser family.
- Email send rates (magic-link, export, delete, change-confirmation, visit-notify).
- Calibration drift bands (aggregate distributions, no per-account dimension).

What we explicitly do **not** instrument:

- Per-account presence-time, even in aggregate as a histogram tagged by account.
- Per-bird drift values.
- Per-account visit frequency.
- Per-account session counts or "days active."
- Anything that could be later "just exposed" as a streak.

---

## 15. Risks

The risks worth naming are the ones that fail invisibly. Each has a mitigation.

### 15.1 Drift calibration drifts (over a year of operation)

**Risk:** the drift function gains, set early in build, turn out to be too fast or too slow at scale once real users are spending real time. The failure mode is silent — users don't report "my bird is drifting at the wrong rate"; they just report something feels off, or they leave.

**Mitigation:**
- Calibration test harness as a CI gate (§6.4).
- Aggregate drift-distribution dashboards (no per-account data) reviewed monthly.
- Tunable gains via server-side config; no redeploy needed to adjust.
- A "drift band" alerting threshold: if the aggregate distribution of "trait change after 7 days of regular visits" moves outside the design band, alert.

### 15.2 Sync correctness regressions (overwriting drift)

**Risk:** a future code path inadvertently allows the client or a non-tick service to write personality vectors. The user sees their bird's drift silently flatten.

**Mitigation:**
- Database role isolation (§7.6): only the tick worker has UPDATE on `birds.personality_vector`.
- A CI test that opens connections as the API service role and verifies it cannot write the column.
- The `personality_history` audit table flags discontinuities (a delta whose source is not a recognized tick run).

### 15.3 Audio uncanniness

**Risk:** procedural calls sound mechanical, repetitive, or "early-2000s synthesizer." The product's affective spine breaks; the user disables audio.

**Mitigation:**
- Voice-design review with audio designers before any species' motif library ships.
- Per-call randomization on pitch micro-jitter, timing micro-jitter, and timbre parameters; no exact-repeat possible.
- A synthetic listening test: a small panel of human listeners compares a 5-minute aviary recording against a control of looped audio; the procedural recording must be preferred at >2:1.
- Editor reviews voice (motif phrase descriptions, captions) for naturalness.

### 15.4 Accessibility regressions

**Risk:** an early v1 ships with strong accessibility surfaces; a later "small visual change" breaks them silently — narration falls behind, captions misalign, keyboard focus loses an indicator.

**Mitigation:**
- Accessibility review on every PR that changes the renderer or the narration manager (named reviewer).
- Automated tests for screen-reader live region updates, keyboard navigation, focus indicator contrast, captions text presence.
- Reduced-motion regression test as a screenshot diff: render a 60-second sequence in reduced-motion mode and diff against a stable baseline.
- The accessibility QA pass blocks releases.

### 15.5 Bundle bloat

**Risk:** dependencies creep, the bundle grows past 2MB, time-to-first-bird falls behind 500ms, the central conceit fades into a load state.

**Mitigation:**
- Per-PR bundle size diff in CI; flagged at +50KB, blocked at the 1.8MB hard line.
- Quarterly "what's in the bundle" audit (`source-map-explorer`).
- Code-split aggressively (§11.1).

### 15.6 Telemetry leakage of per-bird state

**Risk:** a future engineer adds an `account_id` tag to a metric "for debugging," and per-account behavior data starts flowing into the analytics pipeline.

**Mitigation:**
- Database topology separation (§13.2) makes the join physically impossible.
- Telemetry schema lint allows only enumerated, low-cardinality tags.
- Quarterly audit of metric tags.

### 15.7 The "harmless" gamification feature

**Risk:** a contributor proposes a small streak counter, a "you visited every day this week" notebook entry, or an exportable visit log shaped as activity log. The proposal is well-intentioned and would pass code review on technical grounds.

**Mitigation:**
- Rule-of-the-product review (§13.5) — required gate for any feature that introduces a counter, notification, or aggregate over user behavior.
- This PLAN.md is the durable artifact the review cites.
- The narrator's phrase library structurally cannot generate user-behavior observations (§6.10).

### 15.8 Personality numerics leaking via the export

**Risk:** the account export contains personality vectors. A future "in-product preview of your export" feature surfaces the numbers, breaking §6.2.

**Mitigation:**
- The export is generated as a JSON blob and emailed as a link. There is no in-product preview surface, by design.
- Adding one would require rule-of-the-product review.

### 15.9 Visit feature growing into a social surface

**Risk:** the visit feature is the foothold from which leaderboards, public discovery, friend feeds eventually sprout.

**Mitigation:**
- The visit data model is intentionally small and not joinable into a social graph (§3.8).
- No aggregation across visits exists in the analytics warehouse.
- "No leaderboards, no public discovery" is named in `non_goals.md` and in this plan.
- Rule-of-the-product review for any visit-feature extension.

### 15.10 Tick worker outage

**Risk:** the tick tier goes down for hours; when it returns, drift catches up correctly (events are still in the log, ordered) but the user sees a "frozen" aviary during the outage.

**Mitigation:**
- Multi-AZ tick deployment.
- The API tier remains up during a tick outage; it serves the last canonical snapshot and accepts events into the log. The user sees a slightly stale aviary, not an error.
- Tick re-engagement on recovery is automatic (the tick reads from `last_consumed_seq` and catches up).
- Aggregate alarm if tick latency p99 > 5s (per error budget).

### 15.11 Adoption-flow first-encounter feels processed

**Risk:** the first encounter — the user signs up, names two birds, and lands in the aviary — is the moment the affective contract is made. If it feels like configuring an avatar, the rest of the product reads as decoration.

**Mitigation:**
- The two starter birds are presented as "the birds that arrived," not as a catalog selection (§bird_engine §adoption flow). The text is naturalist: "two birds have arrived in your aviary. they need names — would you like to name them now or watch first?"
- The aviary transitions from empty quiet field to the first bird flying in — there is no "your aviary is being prepared" loading screen.
- Voice review on every word of the adoption flow.

---

## 16. Open questions and explicit calls

The PRD names some calibration as "implementation detail." Where the call is mine, I record it here so a future review can find it.

1. **Tick cadence default**: 60 seconds. Tunable by config.
2. **Presence activity-window threshold**: 3 minutes. Tunable; we will lengthen toward 4–5 if early data shows users sitting still for longer than expected.
3. **Drift gain table**: starts at the values that hit the calibration test harness targets. Owned by the calibration owner role; tunable in config.
4. **New-bird offer pacing**: first offer at 8 weeks, second at 16, third at 28, fourth at 44, fifth at 64 weeks. Reaches cap of 7 around 18 months. Tunable.
5. **Magic-link rate limit**: 5/email/30min, 20/IP/30min.
6. **Notebook entry minimum interval**: 36 hours (extendable to 48 if the narrator candidate signal is not "noteworthy").
7. **Sunrise/sunset model**: month-of-year + per-account latitude (default from signup IP, then dropped). We do not refine to lat/long over time in v1.
8. **Visit invite expiry**: 30 days; non-revivable.
9. **Soft-delete window**: 30 days.
10. **Account export expiry**: 24 hours after generation.
11. **Backend stack**: Go for both API and simulation tiers. Postgres 16 for primary store. Redis for ephemeral (rate limits, locks, work queue). S3-compatible for exports.
12. **Frontend stack**: React + TypeScript, Vite/esbuild, Canvas2D for the aviary scene, WebAudio for synthesis, no animation library.
13. **Adoption flow**: 2 birds presented as "the birds that arrived." The user names them; the names default to a small hand-picked list per species silhouette. The flow is one screen.
14. **Visit notification opt-in**: a single toggle in account settings, off by default. Email only; no push.

---

## 17. Definition of done for v1

The v1 launch is shippable when, simultaneously:

- The calibration test harness (§6.4) passes the four targets on every PR.
- Time-to-first-bird is <500ms p95 in the synthetic monitor over 4G from at least three geographies for one week running.
- Bundle size is ≤2MB gzipped.
- 30-minute memory test passes on the reference rig.
- 60fps idle motion is sustained on the reference rig for 30 minutes.
- Accessibility QA pass complete: screen-reader narration, captions, reduced-motion, keyboard navigation all verified by an accessibility reviewer.
- Privacy review pass complete: email isolation, telemetry topology, hard-delete pipeline verified.
- Rule-of-the-product review complete: the surface contains nothing that violates "notice, never announce" or the gamification refusals.
- Multi-device sync verified: laptop + phone manual test session, with simultaneous interactions, produces identical aviary states across both clients within one tick cycle.
- Visit feature: invite, view, revoke, expiry, log all verified end-to-end. Visitor cannot inject events into the host's log (verified in test).
- Account export, account soft-delete, account hard-delete all verified end-to-end.
- WebAudio fallback path (graceful silence + captions) verified on a denied-audio-permission browser.

When all of those are simultaneously true, we ship — quietly. Not with a launch event, not with a marketing push. The product is meant to be found, not announced.

---

*End of plan.*
