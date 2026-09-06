# Pocket Aviary — v1 Implementation Plan

Phase-1 planning deliverable for the CARE benchmark, run 001 / wave_001 / slot 001.
Planner: qwen3.8-max (extra-high effort), opencode harness.
Source spec: `prd/` (product_brief, concepts, bird_engine, interactions, aviary_layout, accounts_sync, social_optional, accessibility_perf, non_goals).

This plan is written for a frontier engineering team to execute without further clarification. Where the PRD leaves a decision open, this plan makes a defensible call and flags it; all such calls are collected in **Appendix A — Resolved ambiguities**. The plan does not implement the product.

---

## 0. Executive summary

Pocket Aviary is a browser-only, single-user virtual aviary whose entire value is affective: birds that feel alive because a **server-side simulation** advances their moods and slowly drifts their hidden personalities in response to honestly-measured **presence**, and a **client render/audio pipeline** that expresses that state procedurally — never canned, never looping, never announcing.

The architecture follows one rule from which everything else falls out: **the server is the only writer of canonical state; clients emit events and render snapshots.** Personality vectors never leave the simulation service (clients receive a derived *behavior projection*). Drift is additive, non-negative, and computed from an append-only event log consumed in order — which makes multi-device sync a property of the architecture rather than a feature, and makes last-write-wins data loss structurally unreachable.

Key engineering pillars:

1. **Simulation service** with a Δt-parameterized continuous-time model: a ~60s tick materializes state, but the same pure `advance(state, events, Δt)` function lets any snapshot be evaluated as-of-now, so scheduler hiccups never corrupt the fiction that "the aviary has been running."
2. **Two-clock expression model**: slow personality drift (weeks, monotonic up, never punished by absence) × fast mood/recency dynamics (hours–days). The "quieter, not mistrustful" return-after-absence behavior is produced by a *recency envelope* on expression — not by negative drift.
3. **Edge-inlined bootstrap state** so the first painted frame is the aviary mid-motion (<500 ms to first bird), with a quiet-field fallback and no spinner anywhere in the product.
4. **Fully procedural audio** (WebAudio, per-bird signature seeds, server-planned call windows, client-realized variation) with graceful-silence-plus-captions as the only fallback.
5. **Accessibility as a designed surface**: server-generated naturalist narration prose, reduced-motion as its own cross-fade rendering register, runtime-generated call captions — all shipping in v1, not retrofitted.
6. **Privacy as architecture**: synthetic UUIDs everywhere, email encrypted at rest in exactly one column, per-account interaction data firewalled from all telemetry pipelines, calibration done on synthetic accounts in staging rather than population analysis in production.

---

## 1. Scope

### 1.1 In scope for v1

| Area | Ships |
|---|---|
| Accounts | Email magic-link auth (15-min expiry, single-use), per-device revocable sessions, email change with verification, account export (JSON via emailed link), soft-delete 30 days → hard delete |
| Aviary | One aviary per account; starts with 2 system-chosen starter birds; cap 7; age-gated adoption offers; user-assigned renameable names; stable bird identity forever |
| Simulation | Server-side tick (~60s), 5-trait personality vector with monotonic-up drift, 6-state mood system with cross-session persistence, recency envelope, bird-to-bird coupling, ambient weather scheduler, day/night on user-local time |
| Interactions | Return-greeting (procedural, absence-shaped, staggered), sit-and-watch presence accounting (3-signal conjunction), listen-in (slow mix re-balance), offers (seed / song fragment / still pool, per-bird cooldown, server-decided reactions), settle (5s undo), field notebook (sparse, read-only, unbounded scrollback) |
| Sync | Snapshot pull on open/visibility/keepalive/frame-gap; append-only event log; additive server-authored deltas; no client state ownership |
| Social | Visit invitations: per-invite opt-in, email one-time link, read-only ambient visitor session, immediate revocation, 30-day expiry, silent visit log, visit-notification toggle (off by default) |
| Accessibility | Screen-reader narration (naturalist prose, 30–60s idle cadence, event priority), reduced-motion mode (cross-fade register; `prefers-reduced-motion` + settings toggle), call captions (opt-in; auto-on when audio unavailable), full keyboard navigation, WCAG AA contrast on all user copy |
| Performance | <2 MB gz initial JS, <500 ms time-to-first-bird (mid-tier mobile / 4G), 60 fps idle on 5-year-old laptop sustained 30 min, zero client memory growth over 30 min (CI-enforced) |
| Observability | Synthetic browser fleet, aggregate-only RUM, tick-latency alarm (p99 > 5 s), privacy-boundary-enforced metric definitions |

### 1.2 Out of scope for v1 (hard refusals, per `non_goals.md` and the brief)

- Native apps of any kind; the data model and protocols are **not** designed around native-client constraints.
- All gamification: achievements, badges, levels, scores, streaks, XP, ranks, tiers, "birds adopted: N" counters, green-dot calendars, "you've been here every day this week" surfaces — including disguised variants (quiet settings calendar, exportable visit log, user-behavior notebook entries). The line: **observations of the aviary are allowed; observations of the user's behavior are never surfaced.**
- Tamagotchi mechanics: no death, no hunger, no distress, no decaying happiness meter, no punishment of absence anywhere in the engine.
- Social-network surfaces: profiles, follows, feeds, discovery, comments, co-presence, leaderboards (we do not even compute the underlying cross-account stats), show-off rendering for visitors.
- Notifications of any kind about the aviary (push, email, in-product toasts). The single exception is the opt-in visit-notification toggle in settings, off by default, which is an account-transparency surface, not an engagement loop.
- "Welcome back" toasts/banners/modals; any textual welcome surface. The bird greeting is the entire welcome.
- Payments, shared aviaries, multi-aviary accounts, customizable scenes, user-arranged perch placement, panning/scrolling/zoom of the scene.
- Numeric exposure of personality vectors on any product surface, debug view, or tier, ever. (The account export includes them because the PRD explicitly lists them there; export is a data-portability artifact, not a product surface — see Appendix A-9.)
- Recorded audio, anywhere, including as a fallback. No last-write-wins on personality state. No client-side ticking.

### 1.3 v1 success criteria (testable)

1. First painted frame shows birds mid-action; no spinner or entry animation exists in the codebase (charm-guard CI test, §14.4).
2. Two identical greeting sequences never occur (procedural-variation test over 10k simulated greetings).
3. Drift calibration targets hit on synthetic cohorts: instrument-measurable trait change after ~1 week of regular visits; user-visible behavioral change after ~3 weeks; **zero** negative trait deltas under any neglect scenario.
4. A background-tab-open-for-48h scenario records ~0 presence-time (presence-honesty test suite).
5. Laptop-morning and phone-evening show the same aviary state (same snapshot version semantics; no merge code exists).
6. Screen-reader session delivers naturalist prose at 30–60s idle cadence with prompt narration of greeting/offer/settle events.
7. All performance budgets green in CI on the defined reference profiles.

---

## 2. Architecture

### 2.1 Service shape

```
                       ┌────────────────────────────────────────────┐
                       │                Edge (CDN)                  │
                       │  static shell + edge bootstrap function    │
                       │  (session check → inline first-frame state │
                       │   from edge snapshot cache)                │
                       └───────┬──────────────────────┬─────────────┘
                               │                      │
                     HTML + inlined state      REST (JSON, small)
                               │                      │
                       ┌───────▼──────────────────────▼─────────────┐
                       │              API service (BFF)             │
                       │  authN/Z, rate limits, event ingest,       │
                       │  snapshot serving, offers (sync reaction), │
                       │  greeting directive, notebook pagination,  │
                       │  account/visit endpoints                   │
                       └──┬──────────┬──────────┬──────────┬────────┘
                          │          │          │          │
                 ┌────────▼───┐ ┌────▼─────┐ ┌──▼───────┐ ┌▼──────────┐
                 │ Simulation │ │ Identity │ │  Email   │ │ Prose     │
                 │ service    │ │ service  │ │ service  │ │ engine    │
                 │ tick sched │ │ magic    │ │ (txn     │ │ (library  │
                 │ advance()  │ │ links,   │ │ provider)│ │ inside    │
                 │ drift/mood │ │ sessions │ │          │ │ sim svc)  │
                 │ call plans │ │          │ │          │ │           │
                 └───┬────────┘ └────┬─────┘ └────┬─────┘ └───────────┘
                     │               │            │
              ┌──────▼───────────────▼────────────▼──────┐
              │  Postgres (primary)   │  Edge snapshot    │
              │  accounts, aviaries,  │  cache (KV, per-  │
              │  birds, events (appen-│  account compact  │
              │  only), notebook,     │  snapshots, pub-  │
              │  invitations, ticks   │  lished per tick) │
              └───────────────────────────────────────────┘
```

Five deployable units + one datastore cluster + one edge cache:

1. **API service (BFF)** — stateless HTTP; owns authN/Z enforcement, rate limiting, input validation, event ingest (assigns per-aviary monotonic `seq` and authoritative `ingested_at`), snapshot reads (from edge cache or simulation service), synchronous offer-reaction decisions (delegated to simulation service RPC), greeting-directive requests, notebook pagination, account/visit flows.
2. **Simulation service** — the only component that writes canonical bird/aviary state. Contains: the tick scheduler and worker pool; the pure `advance()` dynamics kernel (drift, mood, recency, coupling, weather, activity, call planning); the prose engine (notebook + narration generation, shared voice grammar); the offer-reaction and greeting-directive decision functions. The kernel is a versioned, deterministic, Δt-parameterized pure function library — unit-testable without infrastructure and the single source of truth for calibration.
3. **Identity service** — magic links, session tokens, device list/revocation, email change verification, account deletion/restore state machine.
4. **Email service** — thin wrapper over a transactional provider (magic links, visit invites, export links, opt-in visit notifications). DMARC/SPF/DKIM configured; delivery metrics are aggregate-only.
5. **Web client (SPA)** — render pipeline, audio engine, presence detector, event emitter, a11y surfaces. Detailed in §8–§11.
6. **Postgres** — one logical database with strict schema-level separation: `identity` schema (PII: encrypted emails, hashes), `sim` schema (aviaries, birds, events, ticks, notebook), `social` schema (invitations, visit sessions). Append-only enforcement on `interaction_events` via revoked UPDATE/DELETE grants + trigger guard.
7. **Edge snapshot cache (KV)** — compact per-account snapshot (~5–15 KB) published by the tick worker after each materialization; read by the edge bootstrap function and by `GET /v1/aviary/state` at the nearest POP.

Deliberate choices:

- **No message broker in v1.** The PRD's Kafka reference is a PII-leak cautionary example, not a requirement. At v1 scale, an append-only Postgres table with a per-aviary cursor gives strict in-order tick consumption (which the additive-delta correctness argument depends on) with far less machinery. The event-ingest path is designed so a broker can be inserted later without client-visible change (Appendix A-4).
- **No client-to-client channel, no realtime push in v1.** Sync is pull-based per the PRD; a keepalive poll (~45 s while visible) plus visibility-change pulls is sufficient for a simulation whose canonical cadence is ~60 s. WebSockets would add a merge/consistency surface the architecture is explicitly designed not to have.
- **Simulation DB is network-isolated from analytics.** The metrics pipeline has no credential, no route, and no ETL job touching `sim` schema rows (see §13.4).

### 2.2 Client/server split

| Concern | Owner | Notes |
|---|---|---|
| Personality vectors | Server only | Never serialized to any client. Clients receive a **behavior projection** (§7.6). |
| Mood state + transitions | Server only | Snapshot carries current mood; client maps mood → pose/motion parameters. |
| Recency envelope | Server only | Part of projection inputs. |
| Canonical positions / perch zone / current activity | Server (tick) | Snapshot carries perch zone, activity, and motion phase; client interpolates and adds micro-jitter. |
| Call plan (windows, intensities, response chains) | Server (tick) | Sparse schedule for the next ~180 s. |
| Call realization (motif choice, pitch/timing jitter, synthesis) | Client | Ephemeral; never canonical; two devices may hear different renderings of the same plan — acceptable and on-brand (calls are never identical twice anyway). |
| Idle micro-motion, leaves/feathers, parallax, lighting render | Client | Pure ornaments; leaves/feathers explicitly not tick-driven per PRD. |
| Presence detection (3-signal conjunction) | Client detects, server adjudicates | Client sends pings; server computes presence windows, dedupes across devices, clamps. |
| Interaction events | Client emits, server logs | Append-only; server-assigned ordering. |
| Offer reaction decision | Server (synchronous RPC at ingest) | Client renders the reaction; server decides it from mood/personality (§7.7). |
| Greeting directive | Server (computed on session-open snapshot request) | Which bird, what form, stagger offsets, variation seed. |
| Notebook entries + narration prose | Server (prose engine) | Client only queues/paginates/displays. |
| Day/night visuals | Client from device-local time | Server uses account timezone for mood/time-of-day dynamics. |
| Weather | Server scheduler; client renders | Snapshot carries current weather + horizon. |

### 2.3 Render pipeline boundary

The boundary is the **snapshot contract** (§6.2): a versioned JSON document containing everything the client needs to render and behave, and nothing it doesn't. The client is a pure function of (snapshot stream, event stream acknowledgments, local entropy, device-local time, viewport, a11y settings). It holds no canonical state; its local state is (a) interpolation buffers, (b) ephemeral audio/render seeds, (c) optimistic-render flags with bounded lifetime. If local state and the next snapshot disagree, the snapshot wins and the client eases toward it (never snaps — see §8.5 reconciliation).

---

## 3. Technology selections (defaults; substitutable without plan changes)

- **Client**: TypeScript; WebGL2 renderer (PixiJS-class or a thin custom layer — decision gate at M0 spike, §16); DOM overlay for top bar, notebook panel, settings, captions; no SSR framework (static shell + edge function); system font stack for chrome, no webfont on critical path.
- **API/simulation/identity/email**: TypeScript (Node) or Go — team's choice; the dynamics kernel must be a pure library with golden-vector tests either way. Postgres 16. Edge function on the CDN's worker runtime (Cloudflare Workers-class) + edge KV.
- **Auth primitives**: 256-bit random tokens, SHA-256-hashed at rest; session cookies HttpOnly/Secure/SameSite=Lax; AES-256-GCM app-layer encryption for email columns with KMS-managed keys; HMAC-SHA256 (peppered) email hashes for lookup.
- **CI/CD**: trunk-based; deploy gates in §14 (bundle size, memory soak, a11y, charm-guard, calibration harness, contrast).

---

## 4. Data model

All identifiers are synthetic UUIDv7 (time-ordered). **Email appears in exactly two encrypted columns in the entire system** (`accounts.email_enc`, `invitations.invitee_email_enc`) plus their lookup hashes. No log line, metric label, URL path, queue key, cache key, or foreign key anywhere contains an email. CI enforces via structured-logging lint + secret-pattern scans of log output in integration tests (§14.5).

### 4.1 `identity` schema

```
accounts
  id                uuid PK
  email_enc         bytea        -- AES-256-GCM, KMS key
  email_hash        bytea UNIQUE -- HMAC-SHA256(pepper, lower(email))
  created_at        timestamptz
  timezone          text         -- IANA, last-known, updated on session open
  status            enum(active, pending_deletion, deleted)
  deletion_at       timestamptz NULL  -- hard-delete due date (soft + 30d)
  settings          jsonb        -- {reduced_motion: bool|null, captions: bool|null,
                                 --  visit_notifications: bool (default false),
                                 --  audio_enabled_pref: bool|null}
                                 -- null = follow OS/device preference
  data_key_id       uuid         -- per-account crypto-shred key ref (§12.4)

magic_links
  id                uuid PK
  email_hash        bytea        -- pre-account sign-in uses hash, not plaintext
  account_id        uuid NULL FK
  token_hash        bytea UNIQUE
  created_at        timestamptz
  expires_at        timestamptz  -- created_at + 15 min
  consumed_at       timestamptz NULL

sessions
  id                uuid PK
  account_id        uuid FK
  token_hash        bytea UNIQUE
  device_label      text         -- user-agent-derived, matter-of-fact ("Chrome on macOS")
  created_at / last_seen_at / revoked_at / expires_at
```

### 4.2 `sim` schema

```
aviaries
  id                uuid PK
  account_id        uuid UNIQUE FK   -- 1:1 in v1; column (not join) keeps it explicit
  created_at        timestamptz      -- aviary AGE drives adoption offers (§7.9)
  weather_seed      bigint           -- deterministic ambient-weather schedule seed
  starter_seed      bigint           -- species/personality seeding for the 2 starters

birds
  id                uuid PK          -- STABLE FOREVER; never regenerated, never reused
  aviary_id         uuid FK
  species_id        smallint         -- pool of 6 (§7.8)
  name              text             -- user-assigned; default suggested at adoption
  adopted_at        timestamptz
  -- slow clock (server-only; NEVER serialized to clients):
  p_boldness        numeric(5,4)     -- all traits in [0,1], seeded ~N(0.35,0.08)
  p_social_warmth   numeric(5,4)     --   clipped to [0.15,0.60] at adoption
  p_vocal_freq      numeric(5,4)
  p_plumage         numeric(5,4)
  p_curiosity       numeric(5,4)
  call_sig_seed     bigint           -- stable per-bird audio/visual signature seed
  -- fast clock:
  mood              enum(alert, content, curious, wary, drowsy, resting)
  mood_params       jsonb            -- decay rates, coupling susceptibilities
  mood_since        timestamptz
  recency           numeric(5,4)     -- recency envelope value in [0,1] (§7.4)
  last_presence_at  timestamptz NULL
  -- canonical render state (written by tick):
  perch_zone        enum(front, middle, back)
  activity          enum(preen, scan, doze, call, watch, idle, drink, bathe, fly)
  activity_until    timestamptz
  motion_phase      numeric          -- phase offset so first frame is mid-motion
  pos_x, pos_y      numeric          -- scene-normalized [0,1] coordinates

tick_state
  aviary_id         uuid PK FK
  tick_seq          bigint           -- monotonic per aviary
  event_cursor      bigint           -- last-consumed interaction_events.seq
  last_tick_at      timestamptz
  state_version     bigint           -- snapshot version; bumped on every write
  weather_now       jsonb            -- {kind: none|rain|wind, intensity, until}
  call_plan         jsonb            -- planned call windows, next ~180s (§7.5)
  narration         jsonb            -- {current_line, queued_lines[], generated_at}
  projection_cache  jsonb            -- behavior projection (§7.6), rebuilt per tick

interaction_events          -- APPEND-ONLY: UPDATE/DELETE grants revoked; guard trigger
  seq               bigserial        -- global insert order
  aviary_id         uuid
  local_seq         bigint           -- per-aviary monotonic, assigned at ingest
  idempotency_key   uuid UNIQUE      -- client-generated; dedupes retries/replays
  session_id        uuid NULL        -- null for visitor-context = rejected, not logged
  device_id         uuid
  type              enum(presence_ping, presence_end, listen_in_start, listen_in_end,
                         offer, offer_reaction, settle, settle_undo, greeting,
                         rename, adopt, weather_marker, tick_marker)
  bird_id           uuid NULL
  payload           jsonb            -- type-specific facts ONLY (durations, offer kind,
                                     -- client timestamps). Schema-validated: any payload
                                     -- containing personality/mood/state fields is
                                     -- rejected at the API layer (422). Clients cannot
                                     -- express "set boldness" in any code path.
  occurred_at       timestamptz      -- client clock, informational only
  ingested_at       timestamptz      -- SERVER clock; authoritative for all ordering,
                                     -- durations, and presence-window computation

presence_windows               -- materialized by the tick from presence_ping/end events
  aviary_id, started_at, ended_at, source_devices uuid[]
  -- merged across devices: overlapping windows from laptop+phone count ONCE

notebook_entries
  id                uuid PK
  aviary_id         uuid FK
  created_at        timestamptz      -- in-world timestamp shown as "tuesday —" etc.
  body              text             -- naturalist prose, lowercase, present-tense
  source_tick_seq   bigint
  dedupe_sig        bytea            -- similarity signature vs recent entries (§7.10)
  -- read-only: no update/delete grants to API role; no user-facing mutation endpoint
```

### 4.3 `social` schema

```
invitations
  id                 uuid PK
  host_aviary_id     uuid FK
  invitee_email_enc  bytea           -- encrypted; needed to send the invite email
  invitee_email_hash bytea
  token_hash         bytea UNIQUE    -- one-time link token, hashed at rest
  created_at         timestamptz
  expires_at         timestamptz     -- created_at + 30 days
  first_used_at      timestamptz NULL
  revoked_at         timestamptz NULL
  status             enum(outstanding, active, expired, revoked)  -- derived, materialized

visit_sessions
  id                 uuid PK
  invitation_id      uuid FK
  visitor_token_hash bytea           -- visitor session cookie token (post link-use)
  started_at / ended_at timestamptz  -- approximate duration for the visit log
  -- NO presence_windows, NO interaction_events rows are ever created from a
  -- visitor session; visitor-scoped tokens carry zero write scope at the API
  -- layer (§6.5), so visitor attention cannot drift the host's birds.
```

### 4.4 Retention and deletion

- Soft delete (30 days): `accounts.status = pending_deletion`; sign-in during the window offers "I changed my mind" restore (matter-of-fact copy). All state frozen but intact.
- Hard delete: purge job deletes every row keyed by account/aviary across all three schemas, invalidates edge-cache snapshots, revokes sessions, and **crypto-shreds** the per-account `data_key_id` (so backups age out unreadable without a bespoke restore). Export artifacts already emailed are the user's copy.
- Event log rows are deleted with the account (no "anonymized retention" — the privacy commitment says interaction history is the user's, not ours).

---

## 5. Presence accounting (client detector + server adjudication)

Presence is the dominant drift input, so its honesty is engineered at both ends.

### 5.1 Client detector

A presence interval is open only while **all three** hold, evaluated continuously:

1. `document.visibilityState === 'visible'` (visibilitychange listener),
2. document has window focus (`focus`/`blur` listeners; `document.hasFocus()` polled at 5 s as a backstop for browsers that miss events),
3. at least one `pointermove` or `keydown` within the **activity window** — default **180 s**, config-flagged (`PRESENCE_ACTIVITY_WINDOW_S`), calibrated during beta leaning longer (watching birds without moving is the actual product; the PRD explicitly licenses a longer window).

Listeners are passive and throttled (pointermove coalesced to ≥1/s timestamp updates — no per-event work). While the interval is open, the client emits `presence_ping` every **30 s** (jittered ±3 s). On interval close — any condition failing, settle, `pagehide`, `visibilitychange→hidden` — it emits `presence_end` via `navigator.sendBeacon` (fire-and-forget; loss is safe, see server adjudication).

Hidden tab: rendering stops (rAF cancelled), pings stop, audio graph suspended. Simulation continues server-side; nothing client-side models it.

### 5.2 Server adjudication (in the tick)

- Presence windows are computed **from server-received ping times**, not client timestamps: a window extends across consecutive pings with gap ≤ 2× ping interval + 15 s slack; larger gaps close the window (a lost beacon costs at most one interval — biased toward under-counting, which is the honest direction).
- **Cross-device dedupe**: windows from all sessions of one account are unioned before crediting; two devices open simultaneously yield one presence stream, never double drift. (This is a named failure mode the PRD's inflation warning implies; we make it a test.)
- **Clamps**: credited presence ≤ 20 h/day; ping rate-limited at ingest (≤3/min/session, excess dropped with 429); events from revoked sessions rejected.
- Settle and tab-close both simply end the window; no penalty, no distinction in drift, a `settle` event exists only for mood-quieting and notebook/acknowledgment purposes.

### 5.3 Presence-honesty test suite (CI, runs against the kernel + a browser harness)

Scenarios with asserted credited-presence: laptop open in background tab 48 h → ~0; visible but unfocused window 8 h → ~0; visible+focused, one mouse move per 2.5 min for 1 h → ~1 h; visible+focused, no input for 10 min → window closes at activity-window boundary; two devices co-present 1 h → 1 h credited, not 2; beacon loss mid-session → ≤1 interval lost.

---

## 6. API surface

HTTPS + JSON. Auth: session cookie (HttpOnly, Secure, SameSite=Lax) + Origin check on all state-changing verbs. Visitor sessions use a distinct cookie with a `visit` scope claim. All error bodies: `{ "error": { "code": "...", "message": "..." } }` where `message` is **matter-of-fact voice** copy (§11.6 owns the string catalog). Rate limits per session and per IP; per-email magic-link throttling (5/hour, then soft-backoff copy).

### 6.1 Endpoint map

**Auth & account (identity service via API)**

| Method & path | Purpose | Notes |
|---|---|---|
| `POST /v1/auth/link` `{email}` | Request magic link | Always 200 (no account enumeration); rate-limited per email hash |
| `GET /v1/auth/verify?token=` | Consume link → session | Single-use in a txn; expiry 15 min; failure copy: "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| `POST /v1/auth/logout` | Revoke current session | |
| `GET /v1/sessions` | Device list | Matter-of-fact labels + last-seen |
| `DELETE /v1/sessions/{id}` | Revoke a device session | |
| `GET /v1/account` | Profile + settings | |
| `PATCH /v1/account/settings` | a11y prefs, visit-notification toggle | Syncs across devices via snapshot's settings echo |
| `POST /v1/account/email-change` `{new_email}` | Begin verification | Old email works until new verifies |
| `POST /v1/account/export` | Generate JSON export → emailed link | Includes birds, names, personality vectors, moods, notebook, settings (Appendix A-9); link single-use, 24 h |
| `POST /v1/account/delete` / `POST /v1/account/restore` | Soft delete / "I changed my mind" | |

**Aviary state & events (API + simulation service)**

| Method & path | Purpose | Notes |
|---|---|---|
| `GET /` (edge) | HTML shell; authenticated requests get **first-frame state inlined** (§8.2) | Personalized via edge function reading snapshot KV; anonymous/cold → quiet field |
| `GET /v1/aviary/state?open=1` | Full snapshot (§6.2) | `open=1` marks session-open: response includes `greeting` directive; server computes absence from `last_presence_at`. ETag = `state_version` |
| `POST /v1/events` | Batch append interaction events | ≤20 events/batch; idempotency keys dedupe; payload schema per type rejects any state-bearing field (422) |
| `POST /v1/offers` `{kind, bird_id?}` | Offer with **synchronous reaction decision** | Response: `{reaction, bird_id, animation_params, cooldown_remaining_s}` or 409 `offer_cooldown` (client renders a quiet not-taken; no toast). `bird_id` optional — untargeted offers land on the aviary and the server picks responders per mood/curiosity |
| `GET /v1/notebook?before=&limit=50` | Paginated entries, newest first | Unbounded scrollback; read-only |
| `PUT /v1/birds/{id}` `{name}` | Rename | Validates length/charset; logs `rename` event |
| `POST /v1/aviary/adopt` | Accept an age-gated adoption offer | 409 `no_offer_available` if age gate unmet; server chooses species, seeds personality, returns bird; client renders soft fly-in |

**Visits (social)**

| Method & path | Purpose | Notes |
|---|---|---|
| `POST /v1/visits/invites` `{email}` | Create + email one-time link | Host-scoped; per-host outstanding cap 10 |
| `GET /v1/visits/invites` | Outstanding/active invites | Settings surface |
| `DELETE /v1/visits/invites/{id}` | Revoke (immediate) | Active visitor sessions die at next snapshot pull |
| `GET /v1/visits/log` | Who visited, when, ~duration | On-demand only; never badged, never notified (unless toggle on) |
| `GET /v1/visit/{token}` (HTML) | Visitor entry | Token consumed → visitor session cookie; revoked/expired → matter-of-fact "This visit is no longer available." |
| `GET /v1/visit/state` | Read-only snapshot for visitor scope | Same projection contract, **no greeting directive, no event endpoints**; visitor pulls on the same keepalive so revocation lands ≤45 s |

### 6.2 Snapshot contract (`GET /v1/aviary/state`)

Target size 5–15 KB. Shape (versioned `v` field; additive evolution only):

```jsonc
{
  "v": 7, "state_version": 10428, "server_now": "...",
  "account": { "timezone": "Europe/Lisbon", "settings": { /* echo */ } },
  "aviary": {
    "age_days": 97,
    "lighting": { "phase": "evening", "params": {/* palette LUT keys */} },
    "weather": { "kind": "rain", "intensity": 0.4, "until": "...",
                 "horizon": [/* next scheduled hints, coarse */] },
    "settled": false,
    "adoption_offer": null | { "available": true }   // age gate met; no gamified copy
  },
  "birds": [
    {
      "id": "...", "name": "pip", "species": 3,
      "mood": "content",
      "perch_zone": "front", "pos": [0.31, 0.62],
      "activity": "preen", "activity_until": "...", "motion_phase": 0.37,
      "projection": {            // behavior projection (§7.6) — NOT personality
        "greet_latency_ms": [400, 1200], "approach_bias": 0.62,
        "call_rate": 0.8, "call_energy": 0.55, "motion_energy": 0.6,
        "startle_susceptibility": 0.2, "plumage_render": 0.48,
        "curiosity_render": 0.51
      },
      "call_plan": [ { "t": 42.5, "intensity": 0.6, "responds_to": null },
                     { "t": 44.0, "intensity": 0.4, "responds_to": "birdB:42.5" } ]
    }
  ],
  "narration": { "current": "a warbler perches on the high branch, calling softly.",
                 "queued": [] },
  "greeting": null | {           // present only on ?open=1 responses
    "absent_bucket": "overnight",
    "primary": { "bird_id": "...", "form": "head_tilt_step", "at_ms": 900, "seed": 8123 },
    "secondary": [ { "bird_id": "...", "form": "two_note_call", "delay_ms": 2600, "seed": 114 } ]
  }
}
```

Pull triggers on the client: session open; `visibilitychange→visible`; keepalive every 45 s (jittered) while visible; detected long render-frame gap (>90 s between rAFs — laptop suspend); after any `POST /v1/offers` or `settle` (to re-anchor canonical state). `If-None-Match` on `state_version` → 304 (client keeps interpolating).

### 6.3 Event payloads (exhaustive v1 set)

- `presence_ping` `{}` (facts live in `ingested_at`), `presence_end` `{reason: visibility|focus|activity_timeout|settle|pagehide}`
- `listen_in_start` `{bird_id}`, `listen_in_end` `{bird_id, duration_ms}` (duration also server-derivable; used as drift weight)
- `offer` `{kind: seed|song|pool, bird_id?}` → server appends the paired `offer_reaction` `{kind, bird_id, reaction, accepted: bool}` from its own decision (the client never reports the reaction)
- `settle` `{}`, `settle_undo` `{}` (undo valid ≤5 s after settle; tick treats the pair as a no-op for drift, a small mood-quieting blip only if not undone)
- `greeting` `{bird_id, form, absent_bucket}` — appended **server-side** when the greeting directive is issued, so the notebook can truthfully write "pip greeted before wren today"
- `rename` `{bird_id, from, to}`, `adopt` `{bird_id, species}`, `weather_marker` `{kind}` (tick-internal), `tick_marker` (audit)

Ordering authority is `ingested_at` + `local_seq`; `occurred_at` (client clock) is stored for diagnostics only and never used in dynamics — immune to clock skew.

### 6.4 Idempotency and replay safety

Every client event carries a UUID idempotency key generated at emit time; retries reuse it; the UNIQUE constraint makes replays no-ops. Magic-link and visit tokens are consumed inside transactions. Duplicate `settle_undo` after window → ignored silently.

### 6.5 Authorization scopes

Three token scopes: `owner` (full), `visitor` (read `visit/state` only; every write endpoint returns 403 — enforced in middleware, not UI), `pre-auth` (auth endpoints only). The visitor scope has **no event-ingest permission at all**, which is the architectural form of "a visitor's attention does not drift the host's birds."

---

## 7. Simulation engine design

### 7.1 The dynamics kernel

All simulation math lives in one pure, versioned library:

```
advance(state, events, Δt, wall_clock, timezone, config) → state'
```

- **Δt-parameterized**: every rate, decay, and probability is expressed per-unit-time (exponential decays `e^(−λΔt)`, Poisson processes `1−e^(−rΔt)`), so running the kernel at 60 s, 15 min, or a single 48-hour jump yields the same distribution of outcomes. This is what makes cadence tiering (§7.2) and lazy as-of-now evaluation safe.
- **Deterministic given seeds**: all stochasticity flows from `hash(aviary_seed, bird_id, tick_seq, salt)` — reproducible golden-vector tests, debuggable drift histories, no `Math.random()` in the kernel.
- **Versioned**: `kernel_version` stamped into `tick_state`; upgrades ship with migration replay tests (re-tick a corpus of synthetic aviaries old→new and diff behavior projections within tolerance).

### 7.2 Tick scheduling

- **Active tier** (presence or any event within 48 h): tick every **60 s**.
- **Dormant tier**: tick every **15 min** (same kernel, larger Δt — mathematically equivalent per above).
- **Lazy evaluation**: `GET /v1/aviary/state` always evaluates `advance(stored_state, events_since_cursor, now − last_tick_at)` **read-only** to serve as-of-now state, so a snapshot is never stale even if the tick job lags; writes happen only in tick workers, offer decisions, and greeting issuance. (Greeting/offer paths take a short row lock, apply their single-event mutation transactionally, and bump `state_version`.)
- Scheduler implementation: a durable job queue (Postgres-backed time wheel or pg_cron-class scheduler + worker pool with `SKIP LOCKED` claiming). Tick job = `(aviary_id, tick_seq)`; the worker re-checks `tick_seq` under row lock → at-most-once application, crash-safe (cursor + state written in one txn).
- Budget: per-account tick p50 < 25 ms, p99 < 250 ms kernel+IO; fleet p99 alarm at **5 s** per the PRD error budget. Batched fan-out; dormant-tier accounts ticked in cheap bulk passes.
- After each committed tick, the worker publishes the compact snapshot to edge KV (async, retried; staleness of KV is masked by the lazy-evaluation path above).

### 7.3 Tick step order (single transaction)

1. Load state + events `seq > event_cursor` in `local_seq` order.
2. Build/extend presence windows (§5.2); update per-bird `recency` (§7.4).
3. Compute drift deltas (§7.5) — additive, non-negative, applied in log order.
4. Mood transitions (§7.6) over Δt, including bird-to-bird coupling.
5. Weather scheduler: deterministic sparse schedule from `weather_seed` (target: rain ~2–3×/week, short; wind occasional; never assertive). Emit mood/vocal effects while active.
6. Perch + activity assignment: mood/personality/recency-weighted distributions (wary→back, bold→front; drowsy→low doze; etc.); set `activity_until`, `motion_phase`.
7. Call planning for the next ~180 s (§7.7).
8. Notebook candidate evaluation (§7.10) + narration line regeneration (§11.1).
9. Write state, bump `state_version`/`tick_seq`, advance cursor; publish snapshot to KV.

### 7.4 The two clocks, concretely

**Slow clock — personality drift (the only writer: step 3).**

Traits in [0,1]. Per tick, for each bird *b* and trait *t*:

```
Δp[b][t] = η_t · softcap(p[b][t]) · Σ_i w[t][i] · signal_i(b, Δt)      (Δp ≥ 0 always)
softcap(p) = (1 − p/0.90)⁺          -- diminishing returns; asymptote 0.90
```

Signals (per Δt, normalized):

| Signal | Derivation | Traits fed (weights w) |
|---|---|---|
| Presence-time | credited window overlap with Δt (dominant) | all five, small; plumage largest share (plumage "drifts up with sustained attention") |
| Listen-in | `listen_in_end.duration_ms` for bird *b* | social_warmth 0.6, vocal_freq 0.4 (that bird specifically) |
| Offer accepted (by *b*) | `offer_reaction.accepted` | curiosity (small) |
| Offer made near *b* | any offer while *b* is target/responder/front-perch | boldness (smaller) |
| Settle | — | **no directional drift**; ends the presence window cleanly; small mood-quieting only |

- `η_t` global per-trait learning rates, config-owned, calibrated by the harness (§7.11) so that a "regular visit" profile (~10 min presence/day + occasional interactions) yields: **Σ|Δp| ≥ 0.015 on the most-fed trait after 7 days** (instrument-measurable), and **≥ 0.05–0.08 after 21 days** (crosses the projection-visible threshold where greeting latency, front-perch dwell, and call rate observably change).
- **Monotonicity is a kernel invariant**: the code path that could produce Δp < 0 does not exist; a property test asserts `p'[t] ≥ p[t]` over randomized event/Δt fuzzing, including 90-day total-neglect runs (outcome: Δp = 0 exactly).
- **No single-session visibility**: per-tick clamps (Δp ≤ 0.002/tick/trait) plus the softcap make session-scale movement numerically invisible even under burst interaction; offer cooldowns (§7.7) prevent curiosity saturation within a session.

**Fast clock — recency envelope (expression, not personality).**

```
recency' = recency · e^(−Δt/τ_decay) + presence_credit · (1 − e^(−Δt/τ_rise))
τ_decay ≈ 5.5 days half-life; τ_rise rebuilds within ~1–2 regular sessions
```

Recency modulates the **behavior projection** (greeting propensity/latency, approach bias, call energy) but never the traits. This is the engine-level implementation of "a bird that gets ignored becomes ambient — quieter, not mistrustful": after two weeks away, `recency ≈ 0.15` → fewer/softer greetings and less front-perch dwell on day one back, recovering over the next visits, while personality is exactly where it was and **no wary-mood penalty is triggered by absence** (absence is simply not a mood input).

### 7.5 Mood system

Final v1 state set (PRD delegates finalization): **alert, content, curious, wary, drowsy, resting** (`resting` = night settled, eyes closed, low on perch; the nightjar-like species is exempt and may stay `alert`/calling late).

Per-tick transition model: categorical distribution over states with logits

```
logit(m') = base(m' | species, personality)          -- e.g. high boldness ↓ wary logit
          + tod(m' | local hour, timezone)           -- dawn ↑alert, dusk ↑drowsy, night ↑resting
          + weather(m' | kind, intensity)            -- rain ↓vocal energy (via projection), wind ↑alert/↑wary split
          + events(m' | recent offers/listen-ins)    -- accepted offer ↑content; startle-class events ↑wary
          + coupling(m' | other birds' moods)        -- wary spreads: ↑wary logit ∝ Σ wary neighbors × susceptibility
          + persistence(m_current)                   -- mood inertia: current state gets +β logit
          + noise(seeded)
```

- Moods **persist across sessions** by construction — state is stored, never reset on client open; the only "daily-ish reset" is the time-of-day baseline pull (overnight → `resting`/`drowsy`, dawn → `alert`), which is exactly the PRD's "modulo whatever the tick has done in the interim."
- Mood half-life without reinforcement ≈ 6–10 h (config), so yesterday's `wary` softens toward baseline by morning if time-of-day pushes that way — no snapping, always via tick-computed transitions.
- Mood→render mapping is client-side and rich (poses, energies, call coloring) — §8.4, §9.3.

### 7.6 Behavior projection (the client-safe shadow of personality)

Computed each tick: `projection = f(personality, mood, recency, tod, weather, species)`. Fields (see §6.2): `greet_latency_ms` range, `approach_bias`, `call_rate`, `call_energy`, `motion_energy`, `startle_susceptibility`, `plumage_render`, `curiosity_render`. Rules:

- Projection fields are **render/expression parameters**, bounded, quantized to 2 decimals, and deliberately non-invertible (many personality×mood×recency combinations map to the same projection).
- The raw vector exists only in `sim.birds` and the export artifact. No API serializer has access to `p_*` columns (separate DB role; ORM-level column deny-list + serializer test asserting no `p_`/`personality` keys can appear in any response body).
- This makes "never exposed numerically" an architectural property, not a policy: even a compromised client yields only projections.

### 7.7 Call planning + offer/greeting decisions

**Call plan (tick step 7).** For each bird, a thinned Poisson process over the next 180 s with rate `r = base_rate(species) · proj.call_rate · tod_factor · weather_factor · mood_factor`; planned calls get `{t, intensity}`. Coupling: for each planned call, with probability ∝ social_warmth of others, schedule a `responds_to` reply 0.8–3 s later (this is the server-planned skeleton of bird-to-bird conversation and emergent chorus: ≥2 high-vocal birds with overlapping windows). Night: only the nightjar-like species keeps a nonzero rate. The client realizes each planned call with procedural variation (§9); plans are suggestions in time, not audio — two devices hear the same *shape* of afternoon, differently rendered. If a snapshot is overdue, the client extrapolates locally from `proj.call_rate` (ephemeral, never logged).

**Offer reaction (synchronous at ingest).** Decision function over current canonical mood + personality + curiosity projection + per-bird cooldown state:

| Receiver state | Seed reaction |
|---|---|
| curious/content | approach (seed); drink/bathe/watch (pool); join/counter-call (song, by vocal_freq) |
| content, low curiosity | watch, maybe approach late |
| wary | wait, then eventually come near (longer animation latency) |
| drowsy/resting | may not approach at all |

Response includes `animation_params` (latency, path target, dwell) so the client renders a decision it did not make. **Cooldown**: 180 s per bird (config), server-enforced; the offer affordance reflects cooldown as a quiet dimmed state (no timer numerals, no gamified "ready!" flash). The paired `offer_reaction` event is appended server-side → next tick's curiosity/boldness drift input.

**Greeting (on `?open=1`).** Server computes `absent_bucket` from `last_presence_at`: `<15 min` (glance), `15 min–6 h` (quiet call / head-tilt), `6–48 h` (approach step, longer call), `>48 h` (re-orientation: approach + long call, higher chance of a second bird responding). Greeter selection: weighted by `boldness · greet-propensity(recency, mood)` — the bolder bird greets first; the warier bird greets later or not at all that day. Forms are parameterized (which preen to glance up from, call length, whether to step toward front perch) with a per-issuance `seed`; secondary greeters get randomized stagger offsets (1.5–4 s) — never unison. The issued greeting is logged as a `greeting` event (feeds notebook "greeted before" observations). A small per-aviary greeting-history window biases against repeating the identical form+greeter pair back-to-back.

### 7.8 Species pool (v1 = 6)

Design owns final art/names; engineering contract per species: silhouette rig, default plumage palette (calm/naturalist), motif library (call grammar), behavioral priors (base call rate, diurnal/nocturnal flag — exactly one nightjar-like nocturnal species), greeting-form affordances (some species glance, some call). Rarity is not a feature: adoption draws uniformly from species not yet in the aviary (soft preference for variety). Starter pair: two species selected from `starter_seed` with contrasting greeting styles (one higher-boldness prior) so the first session already demonstrates "one bird notices you."

### 7.9 Adoption pacing (age-gated, config-owned initial schedule)

New-bird offers keyed **only** to aviary age: bird 3 @ ≥90 d, 4 @ ≥180 d, 5 @ ≥270 d, 6 @ ≥365 d, 7 @ ≥540 d. (Matches the PRD's "a few months → third bird; a year old → five or six.") The offer appears as a quiet affordance in the natural flow (top-bar account surface + a scene-level soft cue the next session — no confetti, no modal, no "unlocked" language; naturalist copy: "a new bird has been seen near the aviary."). Not visit-count, not interaction score, not paid. Cap 7 hard-enforced in the kernel and the API.

### 7.10 Notebook generation (prose engine, tick step 8)

- **Truthfulness rule**: entries may reference only server-canonical facts (greetings logged, activities, moods, perches, weather, offers/reactions, call activity, adoption, renames). Client-only ornaments (leaves) may appear as ambient phrasing but never as specific claims.
- **Noteworthy-pattern detectors** (each a small predicate over recent canonical history): first-of-week greeting order ("pip greeted before wren today, first time this week"); long single-activity stretches ("pip preened for several minutes without looking up"); unusual quiet stretches; first rain/wind in a while; a shy bird dwelling on the front perch; the nocturnal species calling late; an offer accepted after wary waiting; a new bird arriving; a rename.
- **Sparsity governor**: base per-tick emission probability ≈ 0.0002 (≈ one entry per ~3 days of continuous ticking), ×20–50 boost when a pattern fires, hard cap 4/week, floor guarantee ≥1 per 10 days for regularly-visited aviaries (so the notebook never reads as dead). Tuned in beta; the PRD's "preserve sparsity even for very active users" is the governor's acceptance test (synthetic hyper-active profile → ≤4/week).
- **Dedupe**: `dedupe_sig` = simhash of entry body; candidates within Hamming distance 3 of any of the last 30 entries are dropped (specificity requires non-repetition).
- **Voice**: generated from a template grammar with slot fillers bound to real state (species descriptor, name, perch, mood-colored verb phrases), lowercase, present-tense, no "you," no exclamation, no numbers-as-stats. The grammar library is shared with narration (§11.1) so voice continuity is structural.
- **Hard content filter (CI-asserted)**: no entry may match user-behavior patterns — regex/LLM-lint suite over generated corpora rejects "you visited", "every day this week", presence-time references, any second-person address. The notebook observes the aviary, never the user.

### 7.11 Calibration harness (staging-only; production drift analytics are forbidden by the privacy rule)

- **Ghost-watcher simulator**: scripted synthetic users (profiles: `regular_10min_daily`, `weekend_only`, `binge_then_absent`, `hyper_active`, `two_devices`, `night_owl_tz`) driving the real event-ingest + kernel in staging against synthetic accounts.
- **Assertions (ship gates)**: 7-day instrument drift ≥ 0.015 on fed traits; 21-day projection-visible drift (greeting latency, front-perch dwell, call rate shift beyond render-notice thresholds); zero negative Δp under all profiles; neglect-90d → Δp = 0 and recency decay curve matches design; presence-honesty suite (§5.3) green; sparsity governor within bounds; mood diurnal curves match design tables (dawn alertness peak, dusk drowsiness onset).
- Kernel constants live in a config file with the harness as its regression suite — recalibration is a config change + harness run, never a code change.

---

## 8. Frontend rendering pipeline

### 8.1 Scene composition (WebGL2 canvas + DOM overlay)

Layers, back to front:

1. **Sky/lighting** — time-of-day palette LUT (dawn/day/golden/evening/night keys from design system), continuous interpolation on device-local time; settle applies the evening LUT shift over ~4 s regardless of local hour.
2. **Background foliage** — parallax factor 0.2×; soft, low-detail.
3. **Mid plane** — three perch zones (front/middle/back) mapped by a layout solver to viewport-relative bands; birds render here.
4. **Foreground passers** — occasional branch/leaf silhouettes, parallax 1.3×, subtle (the product is not parallax-heavy).
5. **Weather overlay** — rain streaks / wind leaf-ripple, driven by snapshot weather state; never assertive.
6. **DOM overlay** — captions (near calling bird, §11.2), focus rings, top bar, notebook panel, settings surfaces.

Responsive rules: layout solver scales the scene to viewport; on narrow viewports perch zones compress horizontally; on wide viewports they spread; invariant asserted in render tests: **no bird is ever cropped or offscreen at any viewport in the support matrix** (property test across 320 px–3840 px widths).

### 8.2 First frame and loading states (the central conceit)

- **Authenticated navigation**: edge function inlines the compact first-frame state (positions, moods, activities, `motion_phase`, lighting, weather) into the HTML. The renderer's very first paint places birds **mid-action** — `motion_phase` guarantees a bird is mid-preen / mid-call-posture / mid-scan at t=0. There is no entry animation, no fade-from-static, no wake-up sequence in the client; the codebase contains no such transition (charm-guard test §14.4). The only sanctioned fly-in is the one-time adoption entrance (§7.9) and the empty-aviary→first-bird moment after signup, per the PRD.
- **Snapshot pending (cold cache / slow link)**: the quiet field — soft sky color, one or two faint ambient cues (a slow light shift, a distant leaf) — cross-fading to the full scene when state lands. **No spinner exists in the component library.**
- **Stale-then-fresh**: if inlined state is older than the live snapshot, the client eases from stale to fresh over ~1.5 s (interpolation, never teleport).
- Budget path to <500 ms first bird: inlined state (no blocking API round-trip), renderer + bird rigs in the initial chunk, audio engine and panels lazy (§13.1), no webfonts on critical path, `preconnect` to API origin, first-bird paint not gated on audio-context init or non-critical chunks.

### 8.3 Render loop and interpolation

- Single rAF loop; fixed-order passes: input/a11y → state reconciliation → pose solving → layer draws → overlay DOM sync. Frame CPU budget ≤ 8 ms (headroom under 16.7 ms for 60 fps on the 5-year-old-laptop reference profile).
- **Snapshot interpolation**: positions/perch transitions tween with eased paths (flight arcs) over the inter-snapshot interval; mood/projection parameters tween over ~2–5 s (no visual snapping on snapshot arrival); activities blend via pose cross-fades (~400 ms).
- **Reconciliation rule**: server state always wins, but *visually* only through easing; if a bird's canonical perch changed while hidden, the returning user sees it fly/cross-fade there within the first seconds, not a jump-cut.
- Hidden tab: rAF stopped entirely (battery); resumed on visibility with a fresh pull. Long-frame-gap detector (>90 s) triggers a pull (laptop-suspend case).
- Deterministic procedural noise (per-bird seeded Perlin/simplex) drives micro-jitter so motion is varied but stable per bird across sessions.

### 8.4 Idle micro-motion (mood-shaped, continuous)

Per-activity pose rigs parameterized by `motion_energy`, mood, and species:

- **preen** — head-to-wing cycles, occasional pause; content-biased.
- **scan** — head sweeps with dwell; wary-biased (wary birds also render further back per perch assignment).
- **head-tilt** — toward sound events (calls, offers); curious-biased.
- **shuffle** — small weight-reset; random low-rate across moods.
- **doze/rest** — low on perch, fluffed plumage render, eyes closed; drowsy/resting.
- **fly** — perch transitions.

Mood is readable from motion alone — no labels, tooltips, status icons, or hover-chrome anywhere in the scene (no UI chrome inside the aviary; hover on a bird changes only the cursor and the focus affordance, never spawns a tooltip). Motion runs continuously while visible; it is never paused by inactivity (it pauses only when the tab is hidden, where nothing is observable anyway).

### 8.5 Top bar and chrome behavior

DOM top bar: account/settings, accessibility, notebook, offer affordance — four icons, nothing else (audio enablement is *not* an icon; §9.5). Fades to ~8% opacity after 3 s of cursor stillness; restores on pointermove/keydown/focus-within. Icons have accessible names (matter-of-fact register — they are system chrome: "Account", "Accessibility", "Field Notebook", "Offers"). Notebook panel: slide-over, paginated infinite scroll, read-only, entry timestamps in naturalist style ("tuesday —").

### 8.6 Reduced-motion rendering register (a designed surface, not a kill-switch)

Active when `prefers-reduced-motion: reduce` OR account setting on (setting syncs across devices; OS preference is the default, user override wins):

- Micro-motion → **slow cross-fades between still pose atlas frames** (preen = sequence of preen poses cross-fading over ~2–3 s each).
- Flight/perch transitions → cross-fade between perches (bird fades at A, fades in at B), no animated path.
- Ambient leaf/feather drift → removed. Day/evening color shifts → retained, slowed ~3×.
- Weather overlays → static-tinted variants (rain = soft dimming + subtle texture, no animated streaks).
- Audio, captions, drift, mood, notebook → unchanged; the aviary is still the aviary.
- First-frame rule adapts: birds appear in still poses (no mid-motion phase), which is the reduced-motion equivalent of "already there."
- Implemented as a first-class renderer mode sharing the same scene graph (pose-atlas path), not a CSS `animation: none` blanket — snapshot render tests cover both registers.

---

## 9. Audio pipeline

### 9.1 Graph

```
per bird (≤7 voices + 1 ambient bed):
  motif scheduler → synth voice (FM/additive oscillators + shaped noise)
    → per-bird GainNode (listen-in mix)
    → StereoPannerNode (x from scene position)
    → master bus → gentle compressor/limiter → destination
```

- One `AudioContext`, created lazily; bounded polyphony (max 8 concurrent synth voices; planned calls beyond that thin out gracefully — never audible clipping).
- **Zero recorded audio** in the product or the fallback path (unconditional PRD rule).

### 9.2 Call grammar runtime

- **Per-species motif library**: motifs = parameterized note sequences `{contour[], note_durations[], timbre_params, ornament_slots}`. 
- **Per-bird signature**: `call_sig_seed` selects a stable motif subset + fixed timbre offsets (formant-ish filter tuning, vibrato character, brightness). The signature is stable across moods and drift — a two-week user knows Pip by ear (recognizability is the load-bearing affordance; it is also why 7 is the cap).
- **Mood/personality coloring at realization**: wary → shorter, lower, sparser ornaments; content → relaxed contours; curious → rising terminal inflections; drowsy → rare, soft, slow-attack; alert → sharper onsets. `proj.call_rate`/`call_energy` scale rate and amplitude.
- **Never identical twice**: realization applies seeded local entropy — transposition ±, interval stretch, note drop/add within ornament slots, timing swing, micro-detune. A CI variation test synthesizes 10k realizations per species and asserts pairwise feature-distance above threshold (no two calls acoustically identical) while signature-identity metrics (timbre distance to the bird's centroid) stay within the recognizability band. This operationalizes both halves of the PRD's audio contract.

### 9.3 Chorus and bird-to-bird

Call plans carry `responds_to` chains (§7.7); the client schedules responses inside the planned windows with jitter. Because every call is synthesized in real time (no loops, no pre-rendered buffers of whole calls), stacked-loop phase-cancellation artifacts are structurally impossible; gentle per-voice ducking (−2 dB) on overlap keeps the chorus legible. Chorus emergence = two high-vocal-frequency birds' planned windows overlapping — no special code path.

### 9.4 Listen-in mix

- Engage (click/tap/Enter on a bird): focused bird's gain ramps **up** toward +4 dB relative; all others ramp **down** toward an ambient floor of −14 dB relative — **never to silence** (re-balance, not mute; the aviary stays a place where multiple things happen).
- Ramps: exponential approach via `setTargetAtTime`, time constant ≈ 0.7 s → perceptually "slow rise/slow drop" over ~2 s. Hard cuts are a banned pattern (charm-guard audio test asserts ramp presence on every gain change).
- Disengage triggers: re-click focused bird, click empty scene space, focus a different bird (cross-fade: new focus up while old returns to ambient), keyboard focus leaves the scene, Escape. Return to ambient with the same slow ramp.
- Listen-in start/end events emitted (§6.3); duration is a drift input for that bird's social warmth / vocal frequency.

### 9.5 Autoplay policy and the "calls already audible" conceit (resolved)

Browser autoplay policy forbids audible audio before a user gesture — a hard platform law the PRD's "calls already audible on first frame" cannot override. Resolution (Appendix A-6):

1. Attempt `AudioContext.resume()` immediately on load (succeeds for returning users under Chrome's MEI / prior-interaction allowances, and in some Safari states).
2. If blocked, resume on the **first natural gesture anywhere in the document** (pointerdown/keydown/scroll-touch) — typically within seconds; no interstitial, no modal, no "enable sound" surface, no top-bar audio icon (the top bar is four icons, per the PRD).
3. Until audio is live, the aviary is in the same state as the graceful-silence path: captions (if enabled) still render for planned calls, so the moment is covered.
4. The audio-settings toggle lives in accessibility settings (system surface), where a user who wants silence keeps it.

### 9.6 WebAudio fallback

If `AudioContext` is unavailable (old browser within support matrix — rare, permission denied, hardware fault): the aviary runs in **graceful silence with captions defaulted ON**. No recorded-audio fallback exists in any code path. Detection at boot; the failure is logged as an aggregate audio-pipeline error count (no per-account dimension).

### 9.7 Audio memory discipline (feeds the 30-min no-growth CI test)

Oscillator/FM voices are pooled and reused; noise buffers pre-computed once per context; every scheduled node `stop()`ed and dereferenced after playback; no per-call `AudioBuffer` allocation for oscillator-based motifs; caption DOM nodes recycled from a bounded pool; audio graph node count asserted bounded in soak tests.

---

## 10. Sync model

The PRD's claim — "multi-device sync is a property of the architecture, not a feature" — is realized as follows; there is **no sync code** in the client beyond snapshot pulling:

1. **Single canonical record**: one `sim` row-set per aviary; one writer (simulation service). There is nothing to merge because there is nothing duplicated.
2. **Clients never write state**: the event API's payload schemas have no fields for personality, mood, position, or any canonical value; a client that tried to send "set boldness to 0.62" gets a 422. Drift is additive server-authored deltas computed from the log — the PRD's laptop-morning/phone-lunch last-write-wins data-loss scenario is unreachable by construction.
3. **In-order consumption**: server-assigned `local_seq` at ingest; tick consumes strictly in cursor order; idempotency keys make retries/replays no-ops.
4. **Concurrent devices**: both pull the same snapshots (same `state_version` semantics); both emit events; events interleave in the log; presence windows union (§5.2) so co-present devices don't double-feed drift; simultaneous listen-ins from two devices are simply two attention signals (bounded by tick-level diminishing returns).
5. **Optimistic renders are cosmetic-only**: settle lighting, offer-reaction animations, and listen-in mix are rendered immediately from local intent + server decision responses; canonical state arrives in the next snapshot; disagreements resolve by easing (§8.5), never by user-visible correction surfaces. A server-rejected offer (cooldown race) renders as "not taken" — the bird simply doesn't react; no error toast (toasts are banned surfaces).
6. **Reconnection after suspend/absence**: pull on visibility + frame-gap detection; the lazy as-of-now evaluation (§7.2) guarantees the returning user meets the aviary that has been running, in one request, with mood continuity (no snapping to defaults — mood is stored state).
7. **Conflict surface**: true "conflicts" cannot occur in state; the only user-facing sync errors are session/auth-level (expired session mid-write, outage during load) and use the matter-of-fact catalog: "Your session timed out. Sign in again to keep watching." / "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."

---

## 11. Accessibility surfaces

### 11.1 Screen-reader narration

- **Source**: server prose engine (same grammar library as the notebook) generates `narration.current` + event lines each tick and on event-driven moments; text is naturalist voice — running prose, lowercase, present-tense, specific ("a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."). Never state-list format ("Pip is at perch 2" / "Wren mood: content" are banned patterns in the prose-engine test corpus).
- **Delivery**: a visually-hidden `aria-live="polite"` region. Client narration queue: idle cadence **30–60 s** (one line per window, jittered); user-initiated events (return-greeting on open, offer reaction as it happens, settle acknowledgment) get a priority bump — inserted at queue head with a min 2 s spacing so the screen reader's queue is never flooded. If a new idle line arrives while the previous is likely still being read, it **replaces** rather than appends (queue depth capped at 2). High-frequency narration is treated as a defect (it pushes the a11y surface aside).
- **Bird focusables**: each bird is a focusable element in the scene with `aria-label` = its name only (e.g., "pip"); descriptive prose lives in the narration stream, not in per-element state labels (labeling every visual state is the cheap version the PRD rejects). `role="group"` on the scene with an accessible naturalist description ("the aviary").
- Narration is generated from the same canonical state the visuals read — voice continuity between narration and notebook is structural (one grammar library), so a screen-reader user moving between surfaces hears one product.

### 11.2 Call captions

- Opt-in via accessibility settings; **auto-enabled** when WebAudio is unavailable (§9.6).
- Caption text is generated **at realization time from the actual synthesized call** (motif + mood + ornament choices → prose): "a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch". Never a stored per-call string — the caption matches what played (the generator consumes the same realization parameters the synth does).
- Rendering: small text near the calling bird (DOM overlay anchored to scene position), fading in/out with the call envelope; recycled node pool; WCAG AA contrast guaranteed via a semi-opaque scrim behind caption text (contrast-checked against all four lighting keys in CI).
- Captions are product-voice surfaces (naturalist), not system surfaces.

### 11.3 Keyboard navigation (full map)

| Key | Action |
|---|---|
| Tab | Cycles top-bar items (account, accessibility, notebook, offers); from the bar, Tab enters the scene → focuses the first bird; continues cycling birds; then exits to the bar |
| Arrow keys (scene focus) | Move focus between birds (spatial order: left→right, front→back) |
| Enter (bird focused) | Listen-in on that bird |
| Escape | Exit listen-in (from scene); close panel (from notebook/settings/offer sheet) |
| Offer flow | Opens from top-bar button (keyboard-reachable); the offer sheet is a focus-trapped list (seed / song / pool); Enter offers; focus returns to the scene on close |
| Settle | Top-bar-reachable action inside the account/settings surface group per design; single keystroke path documented in the a11y test |

- **Focus indicators**: soft double-stroke outline (light inner + dark outer) legible against both bright midday and dim night scene states — contrast-asserted in CI across lighting keys; exact treatment from the design system.
- All interactive surfaces reachable without a pointer; full-screen scene is operable one-handed; skip-link not applicable (single scene) but DOM order is bar → scene → panels.

### 11.4 Reduced motion — see §8.6 (a designed register, shipping in v1, not a v1.1 fix)

### 11.5 Contrast and legibility

All user-copy text (top-bar labels/icons, settings, account surfaces, errors, captions, displayed narration) passes **WCAG AA minimum**; the design system owns per-surface ratios (floor, not ceiling). The scene itself carries no user copy except chrome/captions. CI: automated contrast checks of every text style against every lighting-key background + caption scrim; axe-core suite on all system surfaces.

### 11.6 Voice register map (copy engineering)

| Surface | Register |
|---|---|
| Aviary scene, notebook, narration, captions, offer prompts, adoption cue, greeting (non-textual) | **Naturalist** — lowercase, present-tense, specific, bird-verbs, no "you," no exclamation |
| Sign-in/link errors, session timeout, load failure, account settings, accessibility settings, export, deletion/restore, device list, visit invites/log/revocation, "visit no longer available", unsupported-browser surface | **Matter-of-fact** — normal capitalization, direct, states what happened + what to do, zero naturalist phrasing |

The rule for future surfaces (encoded in the copy lint): any surface where the user engages the system *as a system* (money, identity, errors, settings) is matter-of-fact; everything else is naturalist. Exact v1 string catalog (from the PRD, verbatim where given):

- "We couldn't sign you in. The link may have expired. Try requesting a new link."
- "Your session timed out. Sign in again to keep watching."
- "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."
- "This visit is no longer available." (revoked/expired visitor link)
- Unsupported browser: "This browser isn't supported. Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge."

---

## 12. Security and privacy engineering

### 12.1 PII firewall (the synthetic-UUID rule as architecture)

- Account references are UUIDs in every table, message, log field, metric label, cache key, and URL. Email lives in `accounts.email_enc` / `invitations.invitee_email_enc` (AES-256-GCM, KMS keys) + HMAC hashes for equality lookup. One identity-service module can decrypt; no other service holds the key reference.
- CI enforcement: structured-logging schema forbids free-text fields in request logs; integration tests scan all emitted logs/metrics/traces for email-shaped patterns and fail the build; magic-link and visit URLs contain only hashed tokens.

### 12.2 Telemetry boundary (policy → pipeline)

- Allowed aggregate telemetry: request counts, latencies (incl. tick computation), error rates, anonymized session-duration histograms, client render-frame timings, first-bird-render timings, audio-context error counts. **No per-account dimension on any of these** (metric definitions exclude account id at the schema level; the metrics store rejects labeled series carrying account/bird identifiers).
- Forbidden and structurally unreachable: analytics warehouse reads of `sim` schema (no credential, no network route, no ETL job), cross-account interaction aggregates, per-bird fields in any pipeline, leaderboards-or-equivalent statistics ("we don't even compute the underlying stats"). Drift calibration runs on **synthetic staging accounts only** (§7.11) — production population drift analysis is exactly the "average drift across all accounts dashboard" the PRD prohibits.
- Privacy policy link in account settings, plain text, naming the aggregate categories and explicitly excluding per-bird interaction state.

### 12.3 Auth hardening

Magic links: 256-bit tokens, hashed at rest, 15-min TTL, single-use (consumed in txn), per-email rate limit; no account enumeration (uniform 200). Sessions: rotating refresh, revocable per device, absolute lifetime 90 d with silent renewal on activity. CSRF: SameSite=Lax + Origin header check on all mutating verbs. Visit tokens: one-time link exchange → scoped visitor session; revocation kills sessions at next pull (≤45 s). Export links: single-use, 24 h TTL, bound to the verified address.

### 12.4 Deletion

Per §4.4: 30-day soft window with one-click restore on any signed-in page ("I changed my mind", matter-of-fact), then hard purge + per-account crypto-shred key destruction covering backups. Deletion job verified by an integration test asserting zero rows and zero KV entries for a deleted account across all stores.

---

## 13. Performance budgets and observability

### 13.1 Budget table (CI gates — build fails on regression)

| Budget | Value | Enforcement |
|---|---|---|
| Initial JS bundle (gz) | **< 2 MB** hard cap; internal target ≤ 700 KB (renderer ~250, audio ~100, app shell ~200, vendor ~150) | size-diff CI on every PR; Lighthouse-CI budget assertions |
| Time to first bird visible | **< 500 ms** on mid-tier mobile / 4G (Moto-G4-class CPU throttle + Fast-4G network profile) | synthetic lab runs per commit on the reference profile; edge-inlined bootstrap keeps the critical path = HTML + initial chunk + first paint |
| Idle motion frame rate | **60 fps sustained 30 min** on 5-year-old mid-range laptop profile | soak harness: frame-time p95 < 16.7 ms, no degradation trend |
| Client memory over 30 min | **zero growth** (heap-snapshot slope ≈ 0) | CI soak with heap diffing; bounded pools (§9.7); notebook scroll virtualization releases offscreen entry references |
| Snapshot payload | 5–15 KB | contract test |
| Tick latency | p50 < 25 ms, p99 < 250 ms per account; **alarm at fleet p99 > 5 s** (PRD error budget) | server metrics + alert |

Code-splitting: initial chunk = shell + renderer + bird rigs + audio core; lazy chunks = notebook panel, settings surfaces, accessibility settings, visit-invitation flow, export, adoption flow (surfaces the user reaches less often, per the PRD's bundle rationale). Bird visuals: procedural rigs + small SVG-derived textures, procedurally tinted by `plumage_render` — no bitmap flock.

### 13.2 What we measure (day one)

- **Synthetic fleet**: scheduled automated browsers from common geographies running the real product: page-load timings, first-bird-render timing, frame-time distributions, audio-context error counts, snapshot delivery latency, edge-cache freshness, greeting-within-2s success rate.
- **Aggregate RUM**: same metric families from real users, anonymized histograms, no per-account dimension (§12.2).
- **Server**: tick latency/throughput/backlog, event-ingest latency + 422/429 rates, presence-ping rates, offer-decision latency, snapshot RPC latency, magic-link/invite email delivery rates, KV publish lag, deletion-job completeness.
- **Alarms**: tick p99 > 5 s; ingest error-rate; KV publish lag > 3 min (stale first frames); audio-error rate spike; email delivery drop; frame-time p95 regression in synthetic fleet.

### 13.3 What we deliberately do NOT measure

Per-account session counts or visit frequency in any dashboard (no streak-adjacent data exists to leak into a feature); per-bird interaction aggregates; population drift analytics (calibration is staging-synthetic only); engagement funnels / notification-y A/B loops (there are no announcement surfaces to optimize); any metric whose labels could reconstruct a user's relationship with their aviary. The metric-definition schema is the enforcement point, not team discipline.

### 13.4 Browser support gate

Last two major versions of Chrome, Safari, Firefox, Edge. Feature detection at boot (WebGL2, WebAudio, ES2020): unsupported → matter-of-fact surface (§11.6), no compatibility shims, no polyfill bloat.

---

## 14. Test strategy

### 14.1 Kernel (simulation)

Golden-vector tests (fixed seeds → exact expected state trajectories); property tests (monotonic drift Δp ≥ 0 under adversarial fuzz; Δt-invariance: 60 s×N ticks ≈ one N×60 s tick within distributional tolerance; presence clamps); calibration-harness gates (§7.11); kernel-version migration replays.

### 14.2 API/integration

Contract tests for every endpoint (schema, scope enforcement — visitor tokens rejected on all writes; 422 on any state-bearing event payload); idempotency/replay suites; magic-link lifecycle (expiry, double-consume, rate limit); deletion completeness; offer cooldown races; two-device concurrency scenario asserting single canonical outcome.

### 14.3 Client

Render snapshot tests in **both motion registers** (normal + reduced) across lighting keys and viewport extremes (bird-never-cropped property); first-frame test asserting mid-action poses at t=0 and absence of entry animations; audio variation + recognizability suites (§9.2); listen-in ramp tests (no hard cuts; floor never silent); keyboard-map E2E (full §11.3 table); narration-queue timing tests (30–60 s idle cadence, priority bump, cap-2); caption contrast across lighting keys; 30-min soak (memory + frame time) in CI.

### 14.4 Charm-guard suite (the product's immune system — CI-enforced absence tests)

Static + runtime assertions that banned surfaces **do not exist**: no toast/banner/modal components matching announcement patterns; no spinner in the component library; no "welcome back"/"you've been gone"/streak/calendar/badge/level/score/achievement strings anywhere in code or copy catalogs; no numeric personality values in any serialized API response (serializer fuzz); no recorded-audio assets in the bundle; no per-leaf server state; notebook corpus passes the user-behavior content filter (§7.10); narration corpus passes the state-list-format ban. Every rule traces to a PRD clause in a manifest file, so the suite is auditable against the spec.

### 14.5 Privacy/security suites

Log/metric/trace email-pattern scans; PII-firewall integration tests; telemetry-store schema lint (no account-dimensioned series); auth abuse tests (enumeration, replay, CSRF); export/deletion flows.

### 14.6 Accessibility suites

axe-core on all system surfaces; screen-reader smoke tests (VoiceOver/NVDA scripted flows: open → hear narration within cadence → tab to bird → Enter listen-in → Escape → open notebook); contrast matrix tests; reduced-motion parity tests (same content/state reachable in both registers).

---

## 15. Rollout plan

### 15.1 Stages

1. **Internal alpha** (weeks 1–2 post-M4): team accounts on production infra; focus: presence honesty, tick correctness, first-frame budget on real networks.
2. **Closed beta** (invite-only, few hundred accounts, 4–6 weeks): the calibration window — activity-window constant (`PRESENCE_ACTIVITY_WINDOW_S`), drift η constants, notebook sparsity governor, narration cadence, greeting-form distributions tuned against real (consented, aggregate-only operational) usage + synthetic harness. Beta cohort seeded across timezones (day/night correctness) and device mixes (phone/laptop sync reality).
3. **Public v1** (GA): open signup; staged traffic ramp via edge (10% → 50% → 100% over ~2 weeks) watching tick backlog, KV publish lag, first-bird p95, audio-error rates.

### 15.2 Birds-per-aviary ramp

The cap-7 engine ships at GA, but the **age gate** means no account can exceed 2 birds until 90 days post-creation — a natural, risk-staged ramp of birds-per-aviary across the fleet (chorus load, call-plan density, projection cost grow gradually). Additionally: the adoption schedule constants are config-owned; if 7-bird choruses show recognizability or perf issues in beta/staging (synthetic 7-bird aviaries are tested from M2), gates can be pushed later without code change. There is no engagement-based ramp — age only, per the PRD.

### 15.3 Instrumented from day one

Everything in §13.2 (synthetic fleet + aggregate RUM + server metrics + alarms), plus: kernel-version stamp on all state (for migration replay), edge-cache freshness, greeting-within-2s rate, offer-decision latency, narration-queue health, audio-context resume success rate (autoplay gap sizing), caption-render counts (a11y usage, aggregate). Calibration dashboards run **only** against staging synthetic accounts (§7.11, §13.3).

### 15.4 Operational runbooks (ship with GA)

Tick-backlog recovery (dormant-tier shedding; lazy evaluation masks user impact); KV publish outage (snapshots serve via lazy path, first-frame inlining degrades to quiet-field + API pull); email provider degradation (matter-of-fact retry copy, provider failover); kernel rollback policy (version-pinned; replay tests before any rollback); incident copy rules (system surfaces stay matter-of-fact even in incidents — no naturalist error pages, ever).

---

## 16. Workstreams, sequencing, and milestones

| ID | Workstream | Contents |
|---|---|---|
| W1 | Platform & identity | Postgres schemas, edge shell + KV, API service skeleton, magic-link auth, sessions, account lifecycle, export, deletion, email service |
| W2 | Simulation kernel | dynamics library (drift, mood, recency, coupling, weather, activity, call plans), tick scheduler/workers, lazy as-of-now evaluation, projection, offer/greeting decisions, calibration harness |
| W3 | Client render | WebGL2 scene, layout solver, bird rigs + pose system, interpolation/reconciliation, lighting/weather, ornaments, first-frame/quiet-field, top bar + fade, reduced-motion register, responsive invariants |
| W4 | Audio | WebAudio graph, motif libraries + signature system, realization/variation, chorus/response scheduling, listen-in mix, autoplay strategy, silence+captions fallback, memory discipline |
| W5 | Prose engine | shared grammar library, notebook detectors + sparsity governor + dedupe, narration lines, caption text generator, voice lint corpora |
| W6 | Accessibility | narration queue + live regions, captions UI, keyboard map, focus system, contrast matrix, settings surfaces, a11y test suites |
| W7 | Visits | invitations, one-time links, visitor scope + read-only session, revocation, visit log, notification toggle |
| W8 | Perf & observability | budgets in CI, synthetic fleet, RUM, alarms, soak harnesses, charm-guard + privacy suites |

**Milestones (each is a demoable gate, not a date):**

- **M0 — Bones** (W1 + W3 spike): auth works end-to-end; static shell + edge function; renderer spike decision (PixiJS-class vs custom) locked by 2-week spike with the 60 fps/8 ms frame budget on the reference laptop profile as the pass criterion.
- **M1 — Living aviary**: tick loop materializing mood/activity/positions; snapshot contract frozen v1; client interpolates; first-frame mid-action from inlined state; day/night on local time; procedural calls playing from call plans; listen-in mix; presence detector + pings + windows. *Gate: open the tab → a bird is mid-preen, calls audibly within policy (§9.5), greeting fires within 2 s.*
- **M2 — Drift & prose**: drift kernel + recency + monotonicity property tests green; offers with server reactions + cooldowns; settle + undo; notebook generation with sparsity/dedupe/filters; narration lines; adoption age gates; 7-bird synthetic load tests. *Gate: calibration harness passes 7-day/21-day targets on ghost-watcher profiles.*
- **M3 — Accessibility complete**: reduced-motion register, captions, keyboard map, narration queue, contrast matrix — all suites green. *Gate: scripted screen-reader session + reduced-motion parity pass; a11y ships with the product, not after.*
- **M4 — Accounts & social**: export, deletion/restore, email change, device revocation, visits end-to-end (invite → read-only session → revocation surface → log), notification toggle. *Gate: privacy suites green (visitor zero-write, PII firewall, telemetry boundary).*
- **M5 — Hardening**: perf budgets green on reference profiles; 30-min soaks (memory, frame time); charm-guard suite complete against the PRD-clause manifest; synthetic fleet live; runbooks written. *Gate: internal alpha exit criteria (§15.1).*
- **GA** after closed-beta calibration window and staged ramp.

Critical path: W2 kernel ↔ W3/W4 snapshot contract (freeze at M1); W5 shares the projection/state vocabulary with W2; W6 depends on W5's prose and W3's focus system. The snapshot contract (§6.2) is the single interface document — changes after M1 require explicit versioning.

Team shape assumption (adjust freely): 2× simulation/backend, 2× client/render, 1× audio, 1× a11y/frontend, 1× platform/infra, 1× design-adjacent prose/content engineering, shared QA automation. The plan's gates are team-size-independent.

---

## 17. Risks and mitigations

| # | Risk | Severity | Mitigations (owners: W-ids) |
|---|---|---|---|
| R1 | **Drift miscalibration — too fast** (Tamagotchi feel: users move numbers by clicking) | High — kills the core promise | Softcap + per-tick clamps + offer cooldowns; calibration harness gates with explicit 7d/21d targets; η constants config-owned for beta tuning; property tests bound single-session Δp (W2) |
| R2 | **Drift miscalibration — too slow** (screensaver feel: nothing matters) | High | Same harness asserts the floor (≥0.015/7d); recency envelope gives fast visible expression changes so week-1 users still feel responsiveness while traits move slowly (W2) |
| R3 | **Presence inflation bug** (silent, population-wide drift corruption — the PRD's named nightmare) | High — invisible until users feel it | Three-signal detector unit tests; server-side window adjudication (never client timestamps); cross-device dedupe; daily clamp; presence-honesty suite in CI (§5.3); beacon-loss bias toward under-count; session-duration histogram anomaly alarm (aggregate-only) (W2, W3, W8) |
| R4 | **Sync correctness regression** (someone adds a client-state write path later) | Medium-High | API schemas have no state fields (422 fuzz test); append-only DB grants; charm-guard serializer test (no `p_*` in any response); architecture decision record pinned to the PRD clause; code-owner review on event schemas (W1, W2, W8) |
| R5 | **Audio uncanniness** — calls feel synthetic/dead, or signature drifts so birds stop being recognizable by ear | High — audio is the affective spine | Motif-library quality bar with listening reviews at M1/M2/M5; variation-vs-recognizability dual metrics (§9.2 CI suite); stable `call_sig_seed`; mood coloring bounded to timing/ornament axes, not timbre; beta feedback channel (qualitative, non-telemetry) (W4) |
| R6 | **Chorus blur near cap 7** (recognizability ceiling) | Medium | 7-bird synthetic mix tests from M2; per-voice ducking; panning by scene position; if the ceiling proves real in beta, age gates push bird 6/7 availability (config, no code) (W4, W2) |
| R7 | **Autoplay gap** (first seconds silent, denting "calls already audible") | Medium | §9.5 strategy (immediate resume attempt + first-gesture resume + caption cover); instrument resume-success rate; no interstitial ever (charm-guard) (W4) |
| R8 | **Accessibility regression** — narration becomes state-list spam, reduced-motion degrades into "animations off," captions desync from realized calls | High — the PRD prices a11y regression as a launch failure | Narration corpora lint (banned formats); queue-timing tests; reduced-motion as a first-class renderer mode with parity snapshot tests; caption generator consumes realization parameters (same object the synth consumes); a11y suites block release (W5, W6, W8) |
| R9 | **Prose repetition** (notebook/narration say the same specific thing twice — specificity charm collapses) | Medium | simhash dedupe vs last 30 entries; greeting-history anti-repeat; sparsity governor; human review of grammar-library additions; corpus tests for near-duplicate rate (W5) |
| R10 | **First-frame budget miss on real networks** (>500 ms → user notices loading) | Medium-High | Edge-inlined state (no round-trip on critical path); stale-then-fresh easing; quiet field (never spinner); synthetic-fleet p95 tracking from alpha; budget gate in CI (W1, W3, W8) |
| R11 | **Tick scheduler backlog at scale** (aviaries stop "continuing") | Medium | Δt-invariant kernel makes cadence tiering + lazy as-of-now evaluation lossless; dormant-tier shedding; backlog alarms; runbook (W2, W8) |
| R12 | **PII leak** (email creeps into a log/key/URL during a deadline) | High (compliance) | Single-module decryption; structured-logging schemas; CI email-pattern scans on all emitted telemetry; tokenized URLs; review checklist item on identity-touching PRs (W1, W8) |
| R13 | **Gamification creep** ("just one harmless streak surface" from a future contributor) | Medium — product-identity risk | Charm-guard suite as executable policy with a PRD-clause manifest; copy lint; notebook user-behavior filter; the non-goals doc checked into the repo root of the client for contributor visibility (W8) |
| R14 | **Email deliverability** (magic links/invites land in spam → sign-in failure) | Medium | Reputable transactional provider, SPF/DKIM/DMARC from day one, delivery-rate alarms, matter-of-fact retry copy, per-email rate limits that don't lock users out (W1) |
| R15 | **Visitor-scope write leak** (visitor attention drifts host birds) | Medium | Scope middleware rejects all writes for visitor tokens (integration-tested); no presence rows creatable without an owner session; revocation ≤45 s via pull cadence (W7, W1) |
| R16 | **Mood snapping perceived on return** (birds feel reset after absence) | Medium | Mood is stored state, never defaulted on open; reconciliation eases all parameter changes; E2E test: 24 h simulated absence → return snapshot mood consistent with tick trajectory (W2, W3) |
| R17 | **Renderer perf on integrated-GPU laptops** (frame budget blown by overdraw/weather) | Medium | M0 spike gates the renderer choice on the reference profile; layer budget caps (draw calls, fill rate); weather overlays cheap by design ("never assertive" helps perf too); soak tests catch regressions (W3, W8) |

---

## Appendix A — Resolved ambiguities (defensible calls, per START_HERE's "make a call and note it")

| # | Ambiguity | Call | Rationale |
|---|---|---|---|
| A-1 | Exact mood set ("finalized in implementation") | {alert, content, curious, wary, drowsy, resting} | PRD's five examples + `resting` for the night-settled state the layout file describes (eyes closed, low on perch); nightjar species exempt |
| A-2 | Where call scheduling lives (tick is ~60 s; calls are finer-grained) | Server plans sparse call **windows/chains** per tick (~180 s horizon); client realizes each with procedural variation | Keeps canonical state server-owned and sync-coherent while allowing sub-tick, never-identical audio; two devices hearing different renderings of the same plan is consistent with "calls vary every time" |
| A-3 | Does the client receive personality vectors? | No — clients receive a bounded, non-invertible **behavior projection** | "Never exposed numerically" enforced architecturally; also keeps snapshots small and removes the temptation surface |
| A-4 | Event transport (broker vs DB log) | Append-only Postgres log with per-aviary cursor; no broker in v1 | Strict in-order consumption is the correctness requirement; broker insertable later without client change; PRD's Kafka mention is a PII caution, not a mandate |
| A-5 | Presence activity window ("a few minutes, calibrate during build") | Default 180 s, config flag, calibrate longer in beta | PRD says lean longer ("watching birds without moving is the actual product") |
| A-6 | "Calls already audible" on first frame vs browser autoplay policy | Resume on load attempt + first natural gesture anywhere; no interstitial/icon; captions cover the gap | Platform law cannot be overridden; the PRD's top bar is exactly four icons, so no audio toggle there; audio pref lives in accessibility settings |
| A-7 | Offer reaction timing (tick is too slow for a felt reaction) | Synchronous server decision at offer ingest; reaction event logged; drift applied at next tick | Server stays single author of outcomes; client gets immediate, canonical reaction parameters |
| A-8 | "Delivered from a CDN edge with the HTML" for a personalized snapshot | Edge function validates session cookie and inlines compact state from per-account edge KV refreshed each tick; lazy as-of-now evaluation covers KV staleness | Honors the PRD's latency intent without a per-user HTML cache invalidation nightmare |
| A-9 | Export includes personality vectors vs never-exposed rule | Export includes them (PRD explicitly lists "current personality vectors" in the export) | The never-exposed rule governs product surfaces; export is a private data-portability artifact the user triggers on themselves |
| A-10 | Adoption pacing specifics ("a few months → third; a year → five or six") | 3rd @90 d, 4th @180 d, 5th @270 d, 6th @365 d, 7th @540 d; config-owned | Matches the PRD's two data points; age-only, never engagement-gated |
| A-11 | "One-time" visit link semantics | Link token consumed on first use → visitor session cookie valid until 30-day expiry/revocation | "One-time" naturally attaches to the emailed link, not the whole visit; revocation still lands ≤45 s via pull cadence |
| A-12 | Timezone source for server-side day/night mood dynamics | Client reports IANA tz at session open; account stores last-known; visuals always use device-local time | PRD anchors mood to "the user's local timezone"; travelers get correct visuals immediately and correct dynamics from next session |
| A-13 | Simultaneous listen-in from two devices | Both logged; drift inputs are additive with diminishing returns; no arbitration needed | Canonical-state model makes this a non-conflict by construction |
| A-14 | Rejected offer (cooldown race) UX | Bird simply doesn't react; affordance shows quiet dimmed cooldown state beforehand; no toast/error | Toasts are banned announcement surfaces; a non-reaction is in-world and honest |
| A-15 | Renderer technology | WebGL2 (PixiJS-class or thin custom), decided by M0 spike against the 60 fps/8 ms budget | Canvas2D is marginal for 7 rigged birds + weather + parallax on a 5-year-old laptop; WebGL2 is universal across the support matrix |
| A-16 | Notebook "indefinite scrollback" vs unbounded storage | Keep all entries (they're sparse: ≤4/week ≈ ≤200/year — trivially small); paginate 50/page | Sparsity governor makes unbounded retention cheap; archiving would violate "old entries do not get archived or hidden" |
| A-17 | Greeting on every visibility change? | Greeting directive issued when absence ≥ ~10 min (below that, plain resume — a glance at most); bucketing per §7.7 | Prevents greeting-spam on tab flutters while honoring "return from another tab" as a real return |

---

*End of plan. Deliverables in this slot: `PLAN.md` (this file) and `CANDIDATE_METADATA.json`. No product code was written; no files outside the assigned slot were touched.*
