# Pocket Aviary v1 — Implementation Plan

This plan is the executable interpretation of the Pocket Aviary PRD. It is written for a frontier engineering team and is detailed enough to start work without further clarification. PRD calibration targets and refusals are honored throughout. Where the PRD is silent or ambiguous, a defensible choice is made and called out under "Decisions made on PRD ambiguity."

The plan is organized so that earlier sections constrain later ones. Read in order; do not reorder during execution.

---

## 0. Three load-bearing rules

These rules govern every other section. If a downstream decision contradicts one of them, the downstream decision is wrong.

1. **Server is the only writer of personality state.** Personality vectors are never set by clients. Clients append interaction events; the server simulation tick consumes them in order and emits monotonic, additive deltas to the personality vector. There is no last-write-wins on personality, ever.
2. **Aliveness is the product.** Procedural calls, mood-shaped idle motion, server-side ticks that run during absence, "first frame is mid-action" rendering — these compose one felt property. Cheap-feeling shortcuts (recorded-audio fallback, spinner-then-fade, ARIA state-list narration, "animations off" reduced motion) are forbidden because they collapse this property even when the rest of the product is correct.
3. **Privacy is architectural, not policy.** Per-bird interaction events never reach aggregate telemetry, never train models, never appear as identifiers in logs or partitions. The line is enforced at the pipeline boundary (separate stores, no cross-reads), not by code review.

A smaller fourth rule, derived from the others: **email is PII; identifiers are synthetic UUIDs**. Email lives in exactly one encrypted column on the account record. Every other reference is the UUID. Telemetry and Kafka partition keys never see email.

---

## 1. Scope

### 1.1 In scope, v1

Functional surfaces:

- Single-user accounts: email plus magic-link auth (15-min expiry; one-shot consume), per-device session tokens revocable from settings, email-change with verification, account export (JSON snapshot, emailed download link), 30-day soft-delete then hard-delete.
- One canonical aviary per account.
- Aviary scene: one horizontal screen, three perch zones (front/middle/back), no panning/scrolling/zoom, day/night cycle anchored to user local time, rare ambient rain/wind, ambient leaf/feather drift, top-bar chrome that fades on cursor stillness, no UI chrome inside the aviary scene proper.
- 2 starter birds at adoption; cap of 7. Adoption assigns 2 species from the pool — no catalog. Species pool: ~6 species with distinct silhouettes, default palettes, and call-grammar motif libraries; one species is a nightjar-like late-active singer.
- New-bird-availability gated by aviary age: ~30d, ~75d, ~150d, ~250d, ~365d. Not interaction-gated.
- Bird-rename anytime; stable internal `bird_id` invariant across renames, syncs, species-pool migrations.
- Personality vector per bird (boldness, social warmth, vocal frequency, plumage saturation, curiosity), 5 scalars, normalized to [0,1]. Stored server-side. Never exposed numerically anywhere.
- Mood state per bird (`wary`, `content`, `curious`, `drowsy`, `alert`, `settled`, `sleeping`), persisted across sessions.
- Personality drift: monotonic toward expressive (no negative drift on absence). Slow filter; calibrated to instrument-detect at ~1 wk and user-perceive at ~3 wk for a "regular" 10-min/day baseline.
- Mood transitions on each tick, modulated by recent events, time-of-day in user local timezone, weather, neighbor moods, and personality.
- Procedural calls synthesized client-side via WebAudio. Per-bird call grammar: small motif sets combined and varied at runtime. Recognizable across mood/drift. Real chorus mixer (not stacked loops).
- Idle micro-motion: continuous, mood-shaped, never reads as paused.
- Bird-to-bird interaction: calls prompt responses, mood spreads, chorus events emerge from co-firing high-vocal-frequency birds.
- Return-greeting: one bird notices on each open, varied by absence length, bird boldness, and mood; staggered when multiple birds would greet.
- Listen-in: focusing a bird raises its mix and quiets others to ambient with slow ramps; never silences others; disengages on second-click / focus-other / empty-click / focus-out.
- Offer: top-bar affordance opens a small sheet for {seed, song fragment, still pool}; per-bird cooldown of a few minutes.
- Settle: top-bar gesture; lighting shifts to evening; 5s undo on any aviary click; tab-close equivalent at engine level.
- Field notebook: auto-generated, naturalist prose, lowercase present-tense, sparse (~1 entry / 3 days for active accounts; soft cap 1/day; bumped for noteworthy events). Read-only, infinitely scrollable, never reflects user-behavior observations (no streak language).
- Visits: per-invite opt-in, off by default, host emails the visitor, one-time link, 30-day unused expiry, host-revocable, no chat, no avatars, no co-presence, no host-side notification by default (per-account opt-in toggle), visit log in settings.
- Multi-device sync: implicit, because canonical state lives on the server and clients pull snapshots.
- Reduced-motion mode: cross-fade rendering, not "animations off"; honors `prefers-reduced-motion` and explicit setting.
- Screen-reader narration in naturalist prose, paced (1 update per 30–60s idle, prioritized for user-initiated events).
- Call captioning, generated from the same grammar at runtime; available as accessibility setting.
- Keyboard navigation across all interactive surfaces, with visible focus indicators that read against bright and dim aviary states.
- WCAG AA on all user copy.

Non-functional surfaces:

- Browser support: last 2 major versions of Chrome, Safari, Firefox, Edge.
- Performance: bundle ≤ 2MB gz, time-to-first-bird ≤ 500ms (mid-tier mobile, 4G), 60fps idle on 5-yo mid-range laptop, no memory growth in 30-min session.
- Telemetry: aggregate operational only — request counts, latencies, error rates, anonymized session-duration histograms, render-frame timing, audio-context error counts. No per-bird, no per-account interaction state.

### 1.2 Out of scope, v1 (and the reasons that flow into design)

- **Native apps.** Web only. Data model and protocols are not designed around native-client constraints.
- **Gamification of any flavor.** No achievements, streaks, levels, scores, badges, "birds adopted: N" counters, green-dot calendars, XP/rank/tier, "you've been here every day" surfaces, exportable visit logs, or quietly-toggleable engagement metrics. The notebook generator is forbidden from emitting user-behavior observations even if the underlying signal is technically available.
- **Tamagotchi mechanics.** Birds do not die, do not show distress, do not have a happiness meter that decays. Drift is monotonic toward expressive. Absence reads as quiet, never as suffering.
- **Social network surfaces.** No profiles, follows, public feed, "explore," friend-of-friend, comments, visitor-to-visitor anything. The single visit affordance is the entire social surface.
- **Push notifications.** No web push. The only emails are: magic link, account export delivery, visit invitation, optional visit-arrival (per-account opt-in), and account-deletion confirmations.
- **Multi-aviary accounts; shared aviaries; household profiles; payments; customizable scenes; player-arranged perches; species catalog at adoption.**
- **Recorded-audio fallback for calls.** Silence + captions on by default is the WebAudio fallback.
- **Last-write-wins on personality state.** The architecture forbids it; not a runtime check.
- **Personality vector exposure.** No stats panel, no debug toggle, no dev mode, no admin-only "see your numbers" UI, ever.
- **Welcome-back / arrival announcements.** No "Welcome back!" toast, no banner, no friendly-text greeting on return. The bird greeting is the entire welcome surface.

### 1.3 Decisions made on PRD ambiguity

These are calls made because the PRD asks for "calibrated during build" or leaves a gap. They are defensible defaults; QA can re-tune.

| Item | Decision |
|---|---|
| Presence activity window (last pointermove/keypress) | 4 minutes |
| Server tick cadence | 60s base, jittered ±5s, per-account-shard scheduling |
| Snapshot keepalive when tab visible | 20s; paused when tab hidden |
| Listen-in mix levels | focused +6dB, others -10dB ambient floor; 1.5s in, 2.0s out, equal-power crossfade |
| Settle undo window | 5s after gesture; any aviary click reverses lighting shift |
| Mood enumeration | {wary, content, curious, drowsy, alert, settled, sleeping} |
| Notebook cadence | ~1 entry per 3 days for active accounts; soft cap 1/day; noteworthy-event bypass with a per-account daily ceiling of 2 |
| Drift filter | EMA with τ ≈ 14 days at 10-min/day baseline; instrument-detectable ≈ 7d, user-perceptible ≈ 21d |
| Bird-availability cadence | 30d → 3rd, 75d → 4th, 150d → 5th, 250d → 6th, 365d → 7th. Aviary age only, never interaction-gated |
| Late-night species | one of the six is the nightjar-like late singer; quieter palette, slower call cadence, distinct call-grammar motif library |
| Magic-link rate limit | 5 requests / hour / email, exponential backoff after burst |
| Account export delivery | emailed download link to verified address; link valid 24h, one-shot consume |
| Visit-link expiry | 30 days unused; one-shot consume; revocable any time |
| Snapshot transport | server-rendered HTML with inline first-snapshot JSON for first paint; WebSocket for streaming snapshots; long-poll fallback over fetch |
| Database | Postgres for simulation state + event log; S3-compatible blob for export artifacts |
| Repo shape | Single monorepo at v1; service-shaped module boundaries for later split (auth, sim, visits, telemetry) |

---

## 2. Architecture

### 2.1 Service shape

V1 ships as one logical product, decomposed into seven services in one monorepo. Boundaries are service-shaped so that a later split (e.g., extracting `sim` to its own deploy) does not require schema changes.

```
┌─────────────────────────────────────────────────────────────────────────┐
│                              Edge (CDN + SSR)                           │
│  - serves the SPA shell with first-snapshot JSON inlined                │
│  - caches static assets (immutable hash filenames)                      │
└───────────────────────────────┬─────────────────────────────────────────┘
                                │ HTTPS
┌───────────────────────────────▼─────────────────────────────────────────┐
│                              API Gateway                                │
│  - terminates HTTPS, JWT verification on session token                  │
│  - dispatches REST + WebSocket to internal services                     │
└──────┬──────────┬────────────┬──────────┬──────────┬───────────┬────────┘
       │          │            │          │          │           │
   ┌───▼────┐ ┌───▼─────┐ ┌────▼───┐ ┌────▼────┐ ┌───▼──────┐ ┌──▼───────┐
   │ auth   │ │ aviary  │ │ events │ │ visits  │ │ notebook │ │ settings │
   │ svc    │ │ snapshot│ │ ingest │ │ svc     │ │ svc      │ │ svc      │
   └───┬────┘ │ svc     │ └────┬───┘ └────┬────┘ └───┬──────┘ └──┬───────┘
       │      └───┬─────┘      │          │          │           │
       │          │            │          │          │           │
       │          │            ▼          │          │           │
       │          │      ┌─────────────┐  │          │           │
       │          │      │ event log   │  │          │           │
       │          │      │ (append-only│  │          │           │
       │          │      │ Postgres)   │  │          │           │
       │          │      └──────┬──────┘  │          │           │
       │          │             │         │          │           │
       │          │             ▼         │          │           │
       │          │      ┌──────────────┐ │          │           │
       │          │      │  sim ticker  │ │          │           │
       │          │      │  (worker pool│ │          │           │
       │          │      │  per shard)  │ │          │           │
       │          │      └──────┬───────┘ │          │           │
       │          │             │         │          │           │
       └──────────┴─────────────▼─────────▼──────────┴───────────┘
                          ┌──────────────┐
                          │ canonical    │
                          │ aviary state │
                          │ (Postgres)   │
                          └──────────────┘
```

Services and their single responsibilities:

- **auth**: magic-link issuance/consumption, session token issuance/revocation, email-change verification, account-deletion soft/hard transitions, account export coordination.
- **aviary snapshot**: read-only snapshot delivery (REST `GET /aviary/snapshot`, WS `aviary.snapshot.stream`).
- **events ingest**: append-only writer for client-submitted interaction events. Validates event schema and source token. Never writes personality vectors.
- **visits**: invitation issuance, visit-link validation, host-side visit log, revocation. Owns its own tables but never writes personality state.
- **notebook**: notebook-entry generation worker (consumes simulation deltas + recent events) and read API.
- **settings**: account settings (notification preferences, accessibility prefs that need server persistence, session management UI back-end).
- **sim ticker**: the simulation worker pool. Consumes events from the log on each tick, computes drift and mood transitions per shard, writes canonical state. Single writer of personality vectors per shard.

### 2.2 Client/server split

- **Server owns:** personality vectors, mood, mood timers, current call timing, perch position transitions, bird identity, weather state, day/night state, notebook entries, visit invitations, visit logs, account state, session tokens.
- **Client owns:** rendering. The client interpolates between snapshots, synthesizes audio from grammar, runs ambient-only ornaments (leaf/feather drift), and emits interaction events. The client never persists personality, mood, drift, or any field that drives the simulation.
- **Shared:** call-grammar definitions (shipped in the bundle), species visual assets (shipped in the bundle), the snapshot schema, and the event schema.

### 2.3 Render pipeline boundary

The boundary lives at the snapshot. A snapshot is a small JSON payload (kilobytes) describing per-bird positions, mood, call-timing windows, transitions in flight, weather state, and time-of-day. Everything visible — including the first frame the user sees — is derived from a snapshot. The first snapshot is delivered inline in the SSR HTML to keep TTFB ≤ 500ms; subsequent snapshots arrive over a WebSocket stream.

The client never derives personality from snapshots; personality is opaque to the client. The snapshot exposes only what the client needs to render. This is enforced at the snapshot serializer: there is one allowed-fields list, and personality fields are not on it.

### 2.4 Repository layout

```
/                 root
  apps/
    web/          SPA (TypeScript, React or equivalent — see §7)
    edge/         SSR worker
  services/
    auth/
    snapshot/
    events/
    visits/
    notebook/
    settings/
    sim/
  packages/
    schema/       snapshot + event schemas, shared types
    grammar/      call-grammar definitions and motif libraries
    species/      species silhouettes, palettes, behavioral defaults
    naturalist/   naturalist-voice prose generator (notebook + narration + captions)
  ops/
    migrations/   Postgres migrations (one DB at v1, one schema per service)
    deploy/       infra-as-code
    obs/          dashboards, alerts, synthetic checks
  tests/
    e2e/
    perf/
    drift-calibration/
```

`packages/naturalist/` is one shared module because the notebook, narration, and captions all use the same voice and grammar. Centralizing the voice generator keeps the line between "naturalist" and "matter-of-fact" enforceable: any string that crosses out of `packages/naturalist/` and into a system-error surface gets caught by an eslint rule.

### 2.5 Identifier discipline

- Account: `account_id` UUIDv7, generated server-side at signup.
- Bird: `bird_id` UUIDv7, generated server-side at adoption. Stable across renames, species-pool migrations, syncs.
- Session: `session_id` UUIDv7, one per device.
- Visit invite: `visit_id` UUIDv7. The link the visitor receives carries an opaque random token, never the `visit_id`.
- Email: stored once on the `accounts` table, AES-GCM encrypted with a per-environment master key in KMS. Never logged, never used as a partition key, never used as a join key. A SHA-256 of the lowercase email is stored alongside for "does this email already have an account" checks; that hash is also never used as a foreign-key or partition key.

---

## 3. Data model

All schemas are Postgres unless noted.

### 3.1 `accounts`

```
account_id          UUID PRIMARY KEY
email_ciphertext    BYTEA NOT NULL              -- AES-GCM
email_nonce         BYTEA NOT NULL
email_hash          BYTEA NOT NULL UNIQUE       -- SHA-256(lower(email))
created_at          TIMESTAMPTZ NOT NULL
last_signed_in_at   TIMESTAMPTZ
deletion_requested_at TIMESTAMPTZ                -- soft-delete marker
hard_delete_at      TIMESTAMPTZ                  -- = deletion_requested_at + 30d
notification_visit_arrival BOOLEAN NOT NULL DEFAULT false
accessibility_prefs JSONB NOT NULL DEFAULT '{}'
```

Index: `(email_hash)`, `(hard_delete_at) WHERE hard_delete_at IS NOT NULL`.

### 3.2 `aviaries`

```
aviary_id     UUID PRIMARY KEY
account_id    UUID NOT NULL UNIQUE REFERENCES accounts
created_at    TIMESTAMPTZ NOT NULL              -- aviary age = now() - created_at
weather_state JSONB NOT NULL                    -- current weather; small
day_night     JSONB NOT NULL                    -- derived but cached for fast snapshot
last_tick_at  TIMESTAMPTZ NOT NULL
```

One aviary per account at v1. The 1:1 is enforced with `UNIQUE`.

### 3.3 `birds`

```
bird_id              UUID PRIMARY KEY
aviary_id            UUID NOT NULL REFERENCES aviaries
species              TEXT NOT NULL              -- enum: nightjar-like, warbler, finch, sparrow, wren, thrush
display_name         TEXT NOT NULL
adopted_at           TIMESTAMPTZ NOT NULL
boldness             REAL NOT NULL              -- [0,1]
social_warmth        REAL NOT NULL
vocal_frequency      REAL NOT NULL
plumage_saturation   REAL NOT NULL
curiosity            REAL NOT NULL
mood                 TEXT NOT NULL              -- enum
mood_entered_at      TIMESTAMPTZ NOT NULL
perch                SMALLINT NOT NULL          -- 0=front,1=middle,2=back
last_called_at       TIMESTAMPTZ
order_in_aviary      SMALLINT NOT NULL          -- presentation order (left to right)
```

Constraint: `COUNT(*) <= 7` per aviary, enforced via CHECK on a denormalized aviary count or trigger.

The personality fields live here because there is no separate "personality vector" table — the cleanest schema for "five scalars per bird" is "five columns per bird." Splitting them into a long-form table ("trait per row") would invite ad-hoc SELECTs that join across, expand the surface area for accidental leakage to telemetry, and make the "no client write to personality" rule harder to grep.

### 3.4 `events`

The append-only interaction event log.

```
event_id      UUID PRIMARY KEY
account_id    UUID NOT NULL                   -- partition key in shard map
aviary_id     UUID NOT NULL
bird_id       UUID                            -- nullable; presence pings have none
emitted_at    TIMESTAMPTZ NOT NULL            -- client clock; UTC
received_at   TIMESTAMPTZ NOT NULL DEFAULT now()
session_id    UUID NOT NULL
kind          TEXT NOT NULL                   -- enum: presence_ping, listen_in_start,
                                              --       listen_in_end, offer_seed,
                                              --       offer_song, offer_pool, settle,
                                              --       tab_visible, tab_hidden, focus_bird
payload       JSONB NOT NULL                  -- shape per kind
applied_in_tick TIMESTAMPTZ                   -- nullable until consumed
```

Indexes:
- `(account_id, emitted_at)` for tick consumption.
- `(applied_in_tick) WHERE applied_in_tick IS NULL` for next-tick selection.

The log is *append-only*. There is no UPDATE path that mutates an existing event. Replays are detected by `event_id` uniqueness; duplicates are dropped silently.

### 3.5 `notebook_entries`

```
entry_id      UUID PRIMARY KEY
aviary_id     UUID NOT NULL REFERENCES aviaries
written_at    TIMESTAMPTZ NOT NULL
prose         TEXT NOT NULL
subject_birds UUID[] NOT NULL                  -- bird_ids referenced in the prose
generator_version TEXT NOT NULL                -- so we can reason about format drift
```

Index: `(aviary_id, written_at DESC)`.

### 3.6 `sessions`

```
session_id      UUID PRIMARY KEY
account_id      UUID NOT NULL REFERENCES accounts
created_at      TIMESTAMPTZ NOT NULL
last_seen_at    TIMESTAMPTZ NOT NULL
revoked_at      TIMESTAMPTZ
user_agent      TEXT NOT NULL                 -- truncated; for the visible session list
ip_country      TEXT                          -- coarse geolocation; never finer than country
```

The user_agent and ip_country fields exist so the session list is recognizable to the user ("Chrome on macOS, US"). They are intentionally not finer-grained.

### 3.7 `magic_links`

```
link_id        UUID PRIMARY KEY
account_id     UUID                           -- nullable: nullable until first signup completes
email_hash     BYTEA NOT NULL
token_hash     BYTEA NOT NULL UNIQUE          -- SHA-256 of opaque token
issued_at      TIMESTAMPTZ NOT NULL
expires_at     TIMESTAMPTZ NOT NULL           -- = issued_at + 15 min
consumed_at    TIMESTAMPTZ                    -- non-null = used; one-shot
ip_country     TEXT
```

Index: `(token_hash)`, `(account_id, issued_at DESC)`.

### 3.8 `visits`

```
visit_id        UUID PRIMARY KEY
host_account_id UUID NOT NULL REFERENCES accounts
visitor_email_ciphertext BYTEA NOT NULL
visitor_email_nonce      BYTEA NOT NULL
visitor_email_hash       BYTEA NOT NULL
issued_at                TIMESTAMPTZ NOT NULL
expires_at               TIMESTAMPTZ NOT NULL  -- = issued_at + 30d
revoked_at               TIMESTAMPTZ
first_used_at            TIMESTAMPTZ
last_seen_at             TIMESTAMPTZ
token_hash               BYTEA NOT NULL UNIQUE
```

The visitor's email is stored once, encrypted, exactly the same way as the account email. The token is one-shot per session — a visitor who follows the link and stays gets a visitor session token; subsequent loads use that token rather than the original.

### 3.9 `visit_session_tokens`

```
visit_session_id UUID PRIMARY KEY
visit_id         UUID NOT NULL REFERENCES visits
created_at       TIMESTAMPTZ NOT NULL
last_seen_at     TIMESTAMPTZ NOT NULL
revoked_at       TIMESTAMPTZ
```

Visitor sessions are revoked when the host revokes the parent `visit_id`.

### 3.10 `audit_log`

A small append-only log for security-relevant actions: sign-in, session revocation, account-deletion request, account-deletion cancellation, hard-delete, email change. This is *not* the interaction event log; it lives in a different table and is consumed by an entirely separate pipeline. It has no per-bird information.

### 3.11 What is NOT in the data model

Listed because the absences are deliberate:

- No `streaks` table, `visit_count` column, `visited_today` flag, `days_in_a_row` field, or any per-account-visit aggregate. The schema must not be able to answer "how many days in a row has this user visited," because the existence of the answer is a foothold for surfacing it.
- No `bird_health`, `bird_hunger`, `last_fed_at`, `distress_level`. These are not allowed even as columns ignored by the application; the schema is the spec.
- No `discovery_index`, `public_aviary` flag, `featured_at`, `like_count`. The architectural absence makes the social-network surfaces' reappearance harder, not easier.
- No `personality_vector_history` table. Drift is implicit in the current vector; we do not log the trajectory, because (a) we don't need it, and (b) keeping it creates a tempting "show the user how their bird has changed" surface that would expose what is supposed to be felt rather than read.

---

## 4. API surface

Two transports: REST for one-shot operations, WebSocket for streaming snapshots.

### 4.1 Authentication

```
POST /auth/magic-link
  body: { email }
  202 Accepted (always; do not leak whether the email exists)

GET  /auth/consume?token=...
  302 → /aviary on success with HttpOnly cookie session token
  302 → /auth/expired on expiry/replay

POST /auth/sign-out
  204; revokes current session

GET  /auth/sessions
  200; list of visible session metadata (no IPs, country only)

POST /auth/sessions/:session_id/revoke
  204

POST /auth/email-change
  body: { new_email }
  202; sends verification to new_email; old email continues to work

GET  /auth/email-change/confirm?token=...
  302 → /account on success

POST /auth/account/delete
  204; sets deletion_requested_at and hard_delete_at = +30d

POST /auth/account/delete/cancel
  204; clears the deletion timestamps if within window
```

### 4.2 Aviary state

```
GET  /aviary/snapshot
  200 → SnapshotV1 (see §4.5)
  Cache-Control: no-store
  Used by:
    - SSR worker for first paint
    - client on visibility change
    - client on long render-frame gap
    - client when WebSocket reconnects

WS   /aviary/stream
  → server-pushed SnapshotV1 deltas
  → client-pushed events (Event schema, §4.6)
  Frames: { kind: "snapshot.full" | "snapshot.delta" | "event.ack" | "control.ping" }
```

### 4.3 Events

Clients append events via the WebSocket frame with `kind: "event"`, or via REST as a fallback.

```
POST /events
  body: Event[]
  202; idempotent on event_id
```

The events ingest path validates schema, attaches `received_at`, and writes to the `events` table. It does *not* compute anything based on the event. The simulation tick is the sole consumer.

### 4.4 Notebook

```
GET  /notebook?cursor=...
  200 → { entries: NotebookEntry[], cursor: ?string }
  Used for paginated infinite-scroll. Entries are immutable.
```

There is no `POST /notebook` — entries are server-generated only.

### 4.5 Visits

```
POST /visits/invite
  body: { visitor_email }
  201 → { visit_id, expires_at, link_url }
  Sends an email to the visitor with the link.

POST /visits/:visit_id/revoke
  204

GET  /visits
  200 → list of host's visits with status

GET  /visit/:token  (visitor entry point, not authenticated)
  302 → /aviary?visit=... on success with visitor session cookie
  302 → /visit/expired on expiry/revocation
```

The visitor's `/aviary` request sets the host's `account_id` in the snapshot route, but with a visitor token that strips write capability and visitor presence-recording. The events ingest service rejects all events from visitor sessions — visitors cannot drift the host's birds.

### 4.6 Schemas

`SnapshotV1` (allowed fields only):

```
{
  schema_version: "1",
  aviary_id, server_time, day_night: { phase, color_temp, brightness },
  weather: { kind: null | "rain" | "wind", strength: 0..1, started_at, ends_at },
  birds: [
    {
      bird_id, species, display_name, perch, mood,
      // motion frame: client interpolates between server snapshots
      pose: { x, y, facing, micro_motion_phase },
      // call timing: when next call expected and grammar seed
      next_call_window: { start, end },
      call_grammar_seed: integer,        // seeds procedural generation
      // listen-in mix: server-controlled so all clients agree
      mix_level: 0..1
    }
  ],
  // Things the client needs but should not read in any other way:
  greeter_bird_id: bird_id | null,       // for return-greeting; cleared after consumption
  notebook_unread_count: integer
}
```

`Event` (from client to server):

```
{
  event_id: UUID,
  emitted_at: ISO8601,
  kind: enum,
  payload: shape per kind
}
```

Concretely, payload shapes:

- `presence_ping`: `{ visibility: "visible", focus: true, last_input_at: ISO8601 }` — server validates the conjunction.
- `listen_in_start`: `{ bird_id }`
- `listen_in_end`: `{ bird_id }`
- `offer_seed | offer_song | offer_pool`: `{ bird_id?: optional, song_motif?: id }`
- `settle`: `{ }` (no payload)
- `tab_visible | tab_hidden`: `{ }` (transport-only signals)
- `focus_bird`: `{ bird_id }` — keyboard or click focus, distinct from listen-in start

Every event carries `event_id`; replays are dropped on `UNIQUE` violation.

### 4.7 Visit-invitation flow

1. Host signs in, opens settings, invokes `POST /visits/invite { visitor_email }`.
2. Server creates a `visits` row, generates an opaque token, returns the `link_url`, and sends an email to the visitor: subject like "Pip and Wren can be visited" (lowercase per naturalist voice for product-side surfaces — see §10.6 — *but visit emails use matter-of-fact subject because they are system-side*; subject is "[Pocket Aviary] {host} has invited you to visit"). Body: matter-of-fact, naming the host, the link, and the expiry.
3. Visitor clicks the link.
4. `GET /visit/:token` validates the token, issues a visitor session cookie, redirects to `/aviary?visit=...`.
5. The aviary snapshot route, with a visitor session, serves the host's snapshot but stripped of any host-private fields (currently none — but the route is the chokepoint). The snapshot transport is read-only; visitor events are rejected with 401.
6. Host can revoke at any time via `POST /visits/:visit_id/revoke`. The next snapshot pull on the visitor's client returns a 410 with a matter-of-fact "visit no longer available" body.

---

## 5. Simulation engine

### 5.1 Tick architecture

The simulation tick runs on a worker pool sharded by `account_id` hash. Each shard is owned by one worker at a time; ownership is leased via Postgres advisory locks so a crashed worker frees its shard within the lease TTL (default 90s).

A single tick for an aviary executes in a transaction:

1. Read the current `birds` rows and the `aviaries` row.
2. Read all events for the account with `applied_in_tick IS NULL`, in `(emitted_at, event_id)` order.
3. Compute drift deltas for each bird, mood transitions, weather/day-night progression.
4. Write updated bird rows (boldness, social_warmth, etc.), mood transitions, weather, and `last_tick_at`.
5. Mark consumed events with `applied_in_tick = now()`.
6. Emit a snapshot for any open WebSocket sessions for this account, via a fanout queue.
7. Hand off to notebook generator: pass the deltas and the consumed events to the notebook scoring function (§5.6).

The transaction guarantees that no partial tick is observable; clients always see a consistent canonical state.

The tick cadence is 60s base ± 5s jitter per shard. Shards are rebalanced on add/remove of workers via consistent hashing.

### 5.2 Drift function

Drift is computed as low-pass-filtered presence and interaction signals. The filter is implemented as an exponential moving average with time constant τ ≈ 14 days at the design baseline of 10 minutes/day of presence.

Per tick, for each bird:

```
For trait t in {boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity}:
    raw_signal_t = sum over events since last tick of (
        weight(event.kind, bird, t) * normalize(event.payload, t)
    )
    delta_t = max(0, alpha_t * raw_signal_t)   // monotonic-toward-expressive: never negative
    new_value_t = clamp(old_value_t + delta_t, [0, 1])
```

Where:

- `alpha_t` is the per-trait gain. Design target: 14-day EMA equivalent for a 10-min/day presence-time signal hitting a measurable change at the instrument level around day 7.
- `weight(event.kind, bird, t)` encodes which interaction kinds drift which traits:
  - `presence_ping` (eligible only when conjunction holds): weight ≈ 1.0 to plumage_saturation; ≈ 0.6 to social_warmth; ≈ 0.4 to others.
  - `listen_in_start/end` summed: weight ≈ 1.5 to social_warmth and vocal_frequency for the focused bird.
  - `offer_*` accepted: weight ≈ 0.8 to curiosity for the receiving bird; weight ≈ 0.4 to boldness for any nearby bird.
  - `settle`: small mood-quieting only; no drift weight (per PRD).
- `normalize` converts payload to a [0, 1] amplitude — for presence_ping, the elapsed time since last presence_ping bounded by activity window; for listen_in, the duration bounded.

Critically, the drift function never decreases trait values. A bird that goes uninvolved for two weeks loses no boldness, no warmth, no plumage saturation. The visible effect is that the bird is *quieter* — mood gravitates to ambient states, vocal frequency expression is lower because vocal_frequency is multiplied at expression time by recent interaction salience — but the personality has not moved backward.

#### Calibration

A calibration test fixture in `tests/drift-calibration/` plays back synthetic presence patterns and asserts:

- Daily 10-min presence for 7 days produces a measurable trait change (sum of all five > 0.05).
- Daily 10-min presence for 21 days produces a user-perceptible trait change (any individual trait moves ≥ 0.10).
- 10 minutes of presence in a single session does not move any trait by more than 0.005.
- 14 days of zero presence does not decrease any trait.

These assertions are run in CI; calibration failures block the build.

### 5.3 Mood transitions

Mood transitions per tick per bird are computed as a weighted Markov decision over the enumerated mood states. Inputs:

- Recent events in the current session window (±10 min): an offer just accepted nudges toward `content`; an alarm-call from another bird nudges toward `wary`.
- Local time-of-day (computed from the user's timezone, derived from the most recent presence_ping or session creation): early morning → `alert`, midday → `content`, late afternoon → `curious`, dusk → `drowsy`, night → `settled`/`sleeping` (the nightjar species stays `alert` longer).
- Weather: rain → vocal_frequency expression dampens, mood unchanged unless severe; wind → `alert` for some, `wary` for others depending on personality.
- Neighbor moods: a bird perched next to a `wary` bird is more likely to enter `wary`; a chorus event nudges nearby high-vocal-frequency birds toward `content`.
- Personality: high-boldness birds resist `wary`; high-curiosity birds are biased toward `curious`; high-vocal-frequency birds resist `drowsy`.

The transition function is implemented as a small table (`packages/sim/mood_transitions.ts`) mapping `(current_mood, signal_pressure_vector) → next_mood_distribution`. Transitions are sampled deterministically per bird from `(account_id, bird_id, tick_number)` so the same canonical state replays the same way for any client at any time.

Mood is *persisted across sessions*. The mood_entered_at timestamp drives idle expression on the client (e.g., a freshly-entered `wary` looks alert and scanning; a long-held `wary` looks watchful and still).

### 5.4 Call grammar runtime

Calls are *generated*, not stored. Each species has a motif library:

```
species: warbler
motifs:
  - { id: "rise2", shape: [+0, +200, +400], ms: [0, 80, 220] }
  - { id: "trill", shape: [+0, +50, -50, +50, -50], ms: [0, 30, 60, 90, 120] }
  - { id: "soft", shape: [+0], ms: [0, 250] }
combinators:
  - { rule: "rise2 then trill", weight: 0.3 }
  - { rule: "soft then rise2", weight: 0.5 }
  - { rule: "trill alone", weight: 0.2 }
modulators_by_mood:
  content:  { pitch_jitter: 0.05, tempo_jitter: 0.10, partial_drop: 0.0 }
  wary:     { pitch_jitter: 0.02, tempo_jitter: 0.02, partial_drop: 0.5 }
  drowsy:   { pitch_jitter: 0.03, tempo_jitter: 0.20, partial_drop: 0.3 }
  ...
modulators_by_personality:
  vocal_frequency: → call rate per minute
  social_warmth:   → response delay to other birds' calls
  boldness:        → amplitude
```

The server *decides when* a bird calls (next_call_window in the snapshot). The client *synthesizes how* the call sounds (motif selection, jitter, partial drop, modulators) using `call_grammar_seed` from the snapshot — so two clients viewing the same aviary hear the same calls.

The chorus mixer is a runtime client-side audio-graph node that mixes per-bird synthesized voices. It is *not* stacked loops; each voice is a live oscillator+filter chain. Per-bird voices are summed into a master bus, with listen-in mixing applied as per-voice gain envelopes (§7.5). The chorus mechanic falls out naturally: when two birds are calling within overlapping windows, the mixer sums them; the per-call procedural variation prevents phase-cancellation artifacts.

### 5.5 Bird-to-bird interactions

Bird-to-bird is computed in the sim tick:

- A bird that just called has a probability of triggering a "response" call from neighbors with high `social_warmth`.
- A bird in `wary` has a probability of pushing neighbors' next mood transitions toward `wary` (small effect; capped per tick to prevent runaway propagation).
- A "chorus event" is detected when two or more birds with vocal_frequency > 0.6 have overlapping next_call_window. The chorus event is recorded as a noteworthy event that can prompt a notebook entry (§5.6).

These interactions are part of the canonical state, computed by the sim, not the client. The client renders them — it sees in the snapshot that bird Pip's next_call_window has shifted because Wren just called — but it does not invent them.

### 5.6 Notebook generator

The notebook generator runs after each tick. Inputs: the consumed events for this account, the deltas computed by drift, mood transitions in this tick, and a tick-level "noteworthy salience score."

Generation:

1. **Salience scoring:** each candidate observable (e.g., "first-greeter changed today," "two-bird chorus," "long quiet stretch," "first time bird X drank from the pool") is scored 0..1.
2. **Cadence enforcement:** if no entry has been written in ~3 days for an active account, raise the salience floor; if an entry has been written today, raise the floor sharply.
3. **Naturalist composer:** the highest-salience candidate above the floor is composed into prose by `packages/naturalist/`. The composer uses a small templated grammar of phrasings, with bird names interpolated and time-of-day color injected. Templates are *patterns*, not strings — randomized so two entries about "first greeter" don't read identically.

Anti-patterns guarded by tests:

- The composer cannot emit any phrase referencing visit count, "every day this week," "you," or any second-person address. A unit test asserts these strings never appear in any composed entry.
- The composer cannot compose entries about the user's behavior; only the aviary's. A test asserts no template has a subject of "you" or "the user."
- The composer cannot reference personality vector values numerically.

### 5.7 Weather and day/night

Weather and day/night are computed at tick time:

- Day/night is a deterministic function of the user's local time, derived from the most recent presence_ping's reported timezone offset (or session-creation timezone if no recent ping). Phases: `dawn`, `morning`, `midday`, `afternoon`, `evening`, `dusk`, `night`. Each phase has a color-temperature and brightness range that the client interpolates between for rendering.
- Weather is sampled stochastically: each tick, a small probability of starting a rain event (∼1 per few days per aviary), a small probability of starting a wind event. Events have durations of 2–8 minutes. While active, weather influences the mood transition function and dampens vocal_frequency expression.

### 5.8 Adoption and bird-availability

A new account starts with two birds, picked deterministically from the species pool by hashing `account_id` so the choice is stable across an unlikely re-creation. Names are user-supplied at signup; defaults are species-shaped suggestions ("Pip," "Wren," etc.).

Bird availability beyond two is gated by aviary age (now() - aviaries.created_at):

```
3rd: 30 days
4th: 75 days
5th: 150 days
6th: 250 days
7th: 365 days
```

When eligible, a "new species offer" surface appears in the top bar — a small unlabeled dot on the account icon. The user can accept, defer, or ignore; ignoring is honored, and the offer remains available indefinitely.

The mechanic *deliberately refuses* to reward interaction. Users with high presence-time get new birds at the same cadence as users with low presence-time. This is the load-bearing rule for "no gamification."

### 5.9 Tick edge cases

- **Worker crash mid-tick:** the transaction is uncommitted; the events remain marked `applied_in_tick = NULL` and will be consumed by the next tick. No drift is lost; no event is double-counted because consumption is idempotent on `(account_id, applied_in_tick)`.
- **Long absence:** when an account hasn't been seen in 30+ days, the tick frequency for that account drops to once per hour (still server-driven, still authored by the server). On the next user visit, a single tick catches up. Drift across 30 days of zero presence is zero (correctly).
- **Account marked for deletion:** ticks continue until hard-delete; this preserves the soft-delete recovery path. The user who clicks "I changed my mind" finds the aviary as it was, plus 30 days of zero-drift quietude.
- **Snapshot fanout failure:** if the WebSocket fanout queue fails to deliver, the client's keepalive will pull a fresh snapshot at the next interval, and the client's interpolation buffer absorbs the gap.

---

## 6. Sync model

The sync model is the architectural property of "server is the only writer," not a separate feature.

### 6.1 The single canonical record

Every personality vector, mood, and per-bird state lives in one `birds` row. There is no client-side personality cache, no offline queue that mutates personality, no eventual consistency to reconcile.

### 6.2 Multi-device flow

1. User signs in on laptop. Laptop opens a WebSocket; receives snapshots; emits events.
2. User signs in on phone. Phone opens a WebSocket to the same `account_id`; receives snapshots from the same canonical state.
3. Both clients' `presence_ping` events go to the same `events` table. The simulation tick consumes them in order; double-counting is prevented by event-ID idempotency. Drift from both sessions accumulates to the same canonical bird record.
4. Both clients see the same snapshots from the next tick onward.

The user experiences this as "I left Pip in a wary mood on my laptop, and when I opened my phone an hour later, Pip had drifted to content" — because the server tick handled the time passage exactly as it would have without the multi-device split.

### 6.3 What sync does *not* do

- It does not do client-to-client direct messaging.
- It does not do client-side merge of personality state.
- It does not do "device X is the master."
- It does not do "last-write-wins" anywhere; clients have nothing to write that could conflict.

This is the implementation of the PRD's "no last-write-wins" rule. The rule is not enforced at runtime by a check; it is enforced by the absence of any code path that would let a client mutate personality. The events ingest service does not have a "set personality" handler. There is no PATCH/PUT on personality. Adding one would require touching the schema and the sim and would be loudly visible in code review.

### 6.4 Conflict scenarios and resolution

- **Concurrent listen-in on the same bird from two devices:** both events land in the log; the sim treats them as separate listen_in episodes. Drift is computed from both. There is no "winner."
- **Concurrent settle from two devices:** both events land; the sim treats them as the most recent settle, which sets the lighting to evening on both clients via the next snapshot.
- **Magic-link consumed twice (replay):** the second consumption fails with a matter-of-fact "this link has expired" message. Auth tokens are one-shot.
- **Stale snapshot at client:** the client uses snapshot timestamp ordering; out-of-order arrivals are dropped at the WebSocket layer. The keepalive ensures recovery within 20s.

### 6.5 Account-deletion sync behavior

When an account is soft-deleted, every device session is signed out at the next `auth/sessions` poll; clients see the matter-of-fact "your account has been marked for deletion. Sign in to recover." surface. Recovery flow re-authenticates and clears the deletion.

---

## 7. Frontend rendering pipeline

### 7.1 Stack

- TypeScript.
- React for the chrome (top bar, settings, notebook list, accessibility settings, account UI). React was chosen because the chrome is small and traditional; the SPA shell and chrome contribute < 80KB gzipped.
- The aviary scene itself is rendered on a `<canvas>` element using a tiny custom WebGL renderer. *Not React for the scene.* React's reconciliation overhead is incompatible with 60fps idle motion at the bird counts we're targeting; the scene runs in an imperative render loop driven by `requestAnimationFrame`.
- WebAudio for procedural audio.
- WebSockets for snapshot streaming.

### 7.2 Bundle layout and code-splitting

Core bundle (≤ 2MB gzipped target, with comfortable headroom):

- `apps/web/main.ts` — boot sequence, snapshot decoder, scene renderer, audio engine. < 600KB target.
- `packages/grammar/` — call-grammar definitions, ~6 species. ~80KB.
- `packages/species/` — species silhouettes (compact SVGs hand-tuned to render efficiently in WebGL via path tessellation). ~150KB total for 6 species.
- React + chrome surfaces. ~250KB.
- WebGL + audio runtime. ~200KB.

Code-split:

- Account settings (the visit-list UI, session-list UI, accessibility settings, export flow) — ~80KB, loaded on settings open.
- The visit-invitation flow — ~30KB, loaded on first invite open.
- Notebook viewer — ~40KB, loaded on first notebook open.
- Visitor mode — ~30KB, loaded only on `/aviary?visit=...`.

Total v1 budget: comfortably under 1.5MB gzipped on first paint. Headroom is reserved for design polish.

### 7.3 First-paint sequence (TTFB ≤ 500ms)

The CDN edge serves the SPA shell HTML with the first snapshot inlined as `<script type="application/json" id="aviary-snapshot">…</script>`. The HTML is generated by an SSR worker that calls `GET /aviary/snapshot` against the snapshot service and embeds the response.

Boot sequence:

1. HTML parse begins; snapshot JSON is parsed synchronously.
2. The hand-written WebGL renderer initializes — it knows how to draw a snapshot-shaped state without any framework.
3. The first snapshot is composed: birds at their snapshot-described perches, in their snapshot-described mood-shaped poses, with idle micro-motion already in progress (the snapshot includes `micro_motion_phase` so the bird's pose is mid-cycle, not at frame 0).
4. The first frame is drawn as the user sees the page. There is no entry animation. There is no fade-from-black. The first frame is the aviary.
5. The audio engine boots; the first bird's `next_call_window` may already be in the past relative to the snapshot's `server_time`, in which case the call begins playing immediately, mid-call.

The PRD's "first frame the user sees has birds mid-action" is enforced in tests: a synthetic perf check captures the first paint and asserts the bird's pose phase is non-zero.

### 7.4 Idle micro-motion

Idle motion runs continuously at 60fps. Each bird has a per-mood motion-program — a small set of pose-keyframes blended at runtime by a phase variable. The phase is owned by the client and persisted across snapshots: the snapshot tells the client "this bird is in mood X, at phase 0.42 at server_time T," and the client extrapolates from there.

Mood-shaped motion:

- `wary`: scan motion (head turns), slow body shifts, stays at back perch.
- `content`: preen cycle (head down, feathers lift, beak tucks, head up), slow steady.
- `curious`: head-tilts toward sound events, leans forward, occasional perch hops.
- `drowsy`: low body, fluffed feathers, slow breath cycle.
- `alert`: upright posture, sharp head turns, wing flicks.
- `settled`: low on perch, head tucked, no scan.
- `sleeping`: still, eyes closed, slow breath.

Transitions between moods are blended over ~2s using pose interpolation; no hard pose snap.

### 7.5 Listen-in and audio mix

The audio graph:

```
[ per-bird voice node ] -- gain[bird] --+
[ per-bird voice node ] -- gain[bird] --+--> [ master bus ] --> output
[ per-bird voice node ] -- gain[bird] --+
                                         |
                          [ ambient bed ] +
```

Listen-in:

- On `listen_in_start` (server-acked via snapshot mix_level updates): focused bird's gain ramps to +6dB over 1.5s; other birds' gains ramp to -10dB over 1.5s. Equal-power fade.
- On `listen_in_end`: gains ramp back to ambient over 2.0s.
- Other birds' gains *never reach -∞*. Floor is -10dB; calls remain audible.

The mix levels are *server-controlled* fields on the snapshot so all clients on the same account hear the same mix. This makes the multi-device behavior coherent: a phone listen-in is reflected in the laptop's mix.

### 7.6 Reduced-motion mode

`prefers-reduced-motion: reduce` (or the explicit accessibility-settings opt-in) switches the scene renderer to cross-fade mode:

- Per-bird motion is replaced by a sequence of pose snapshots that cross-fade over 4–6s each.
- Flight transitions become 2s cross-fades between perches.
- Day/night color shifts continue, slowed by 50%.
- Ambient leaf/feather drift is removed entirely.
- Calls play at full quality, on the same procedural timing.
- Captions are auto-enabled.

Reduced-motion mode is *its own designed surface*. The cross-fade rendering uses the same pose-keyframes the standard renderer uses — the motion is a sequence of those poses interpolated slowly rather than an "animations off" stub. A user opening the aviary in reduced-motion mode sees the aviary, calmer.

Test: a snapshot-rendering harness verifies that a reduced-motion render shows the same birds at the same perches with the same moods, and that the only difference is the motion register.

### 7.7 Responsive layout

The scene `<canvas>` fills the viewport (minus the top bar). Aspect ratio is preserved by horizontal compression: birds shift toward each other on narrow viewports, never out of frame. Perch positions are computed in scene-relative coordinates (0..1 horizontal) and mapped to pixel coordinates per frame. No bird is ever cropped.

Top bar height is fixed (~48px on desktop, ~56px on mobile to accommodate touch targets). The bar fades to 0.1 opacity after 4s of cursor stillness, returns to 1.0 on cursor movement / keyboard activity / touch.

### 7.8 Loading state (slow connection)

If the snapshot is not yet available (cold cache, slow connection), the loading surface is the soft sky color of the aviary palette plus one or two faint motion cues (a leaf drifting, a soft brightness pulse). It is *not* a spinner. The surface is rendered by the SPA shell HTML alone; no framework needs to load to display it.

### 7.9 Empty-aviary state

Between adoption and the first bird appearing, the aviary is empty: same soft sky color, same ambient drift. The first bird enters with a soft fly-in to its starting perch over ~3 seconds. From that moment, the user is never shown an empty aviary again.

### 7.10 Render budget enforcement

A render-frame budget of 16.6ms is enforced via a CI perf test that runs the aviary on a fingerprinted browser environment and asserts:

- 95th-percentile frame time < 16.6ms over a 5-minute idle session.
- 99th-percentile frame time < 33ms (no dropped frames over two adjacent intervals).
- No GC pause > 5ms over a 30-min session.
- Memory growth < 1MB over 30 minutes.

Failures block the deployment.

---

## 8. Audio pipeline

### 8.1 Core architecture

The audio engine is built around WebAudio's `AudioContext`. Per bird, a voice node graph:

```
[ oscillator(s) ] -> [ envelope ] -> [ filter ] -> [ per-bird gain ] -> [ master bus ]
```

Multiple oscillators per voice handle the harmonic structure of bird calls (a fundamental + harmonics for warblers; chirp-like glide for wrens; etc.). The filter shapes spectral content per species. Envelope and oscillator parameters come from the call-grammar combined with mood and personality modulators.

Voice nodes are pooled, not allocated per-call. The pool sizes to the bird-count cap (7) plus a small buffer for chorus overlap. This is the implementation of "no memory growth": no `new AudioBufferSourceNode` per call.

### 8.2 Per-call synthesis

When the server-decided `next_call_window` arrives:

1. The grammar combinator picks a motif sequence using `call_grammar_seed` so all clients agree.
2. The motif is rendered as a sequence of pitch+duration pairs.
3. Mood modulators apply pitch_jitter, tempo_jitter, partial_drop.
4. Personality modulators apply amplitude (boldness), response delay (social_warmth), call rate (vocal_frequency).
5. The voice node plays the sequence, with envelope-controlled per-pitch attack/decay/sustain/release shaping.

The result: two calls from the same bird in the same mood are similar in shape but different in detail — pitch jitter, micro-timing, partial harmonic content. This is what makes the calls feel alive rather than looped.

### 8.3 Chorus mixing

The chorus mechanic emerges from the sim's bird-to-bird interaction model (§5.5) producing overlapping `next_call_window`s. The audio engine doesn't need a special chorus mode: when overlapping voices play, the master bus sums them. Because each voice is procedurally varied, there is no phase-cancellation artifact.

A subtle masterbus compressor with a slow attack/release prevents transient peaks when many voices co-fire; the compressor is conservative enough that it does not compress dynamics in normal listening.

### 8.4 Listen-in mix

Listen-in mix levels are fields on the snapshot (§4.5). The audio engine reads `mix_level` per bird and ramps `per-bird gain` accordingly. Ramps are equal-power crossfades implemented via WebAudio's `setTargetAtTime` for smoothness. Because the mix is server-controlled, multiple devices on the same account hear the same ramps.

### 8.5 Decay back to ambient

When listen-in disengages, the focused bird's gain ramps back to ambient over 2.0s. The other birds' gains ramp back to 0dB over 2.0s. The user experiences this as "the aviary returning to its room sound" rather than a hard switch.

### 8.6 WebAudio fallback

If `AudioContext` cannot be created (older browsers, hardware issue, autoplay policy), the aviary plays in graceful silence, and call captions are auto-enabled. There is *no* recorded-audio fallback. Captions describe what the call would have sounded like, generated from the same grammar. The aviary remains the aviary; the user just experiences it as a quiet aviary with text descriptions of the calls.

A matter-of-fact accessibility-settings note explains the silent state if the user opens settings: "Audio is unavailable in this browser. Calls are shown as captions instead."

### 8.7 Audio asset budget

Total audio code: < 60KB gzipped for the synthesis engine. Grammar libraries: ~80KB total for all 6 species. *Zero recorded audio in the bundle.* No fallback samples, no "silence is unbearable" emergency loops. The 2MB bundle budget is partly enforced by this constraint.

### 8.8 Mute / volume controls

A volume control lives in accessibility settings (matter-of-fact UI). Master gain ramps with the slider. The user can mute entirely; muting does not stop the simulation, only the audio output. A muted user still gets calls played as captions if captions are on.

---

## 9. Accessibility surfaces

Accessibility is designed alongside the rest of the product, not as a v1.1 follow-up. The PRD is unambiguous: a screen-reader user, a reduced-motion user, a user with audio off all get the actual product, not a stripped variant.

### 9.1 Screen-reader narration

- Output: a `aria-live="polite"` region holds running naturalist prose. Updates are paced at 30–60s during idle, prioritized for user-initiated events (return-greeting on session start, offer reactions, settle gesture, chorus events).
- Content: prose composed by `packages/naturalist/`. Same voice as the notebook. Not "Pip is at perch 2" — "a small grey bird is perched on the front rail, calling softly."
- Generation: server-side or client-side, but both call into `packages/naturalist/` so the voice is one voice across surfaces.
- Cadence guard: a per-account-session token-bucket for narration ensures that even high-activity moments cannot exceed 1 update per 10s; the screen reader's queue does not fill up.

The narration is a top-level surface, not an ARIA-label scattering. Test: snapshot-driven narration generation passes a "voice" linter that rejects state-list phrasing and any second-person ("you") address.

### 9.2 Captions for calls

- Captions appear as small text overlays near the calling bird, fading in and out with the call. Same naturalist voice.
- Captions are generated by `packages/naturalist/` from the call-grammar at runtime — the same grammar that produced the audio, with a caption-mode renderer.
- Auto-enabled when audio is unavailable; manually opt-in otherwise.
- Captions respect WCAG AA contrast against any aviary background (a soft outline + readable foreground).
- A user with captions on but audio also on hears and reads the same call. A user with captions off and audio off gets a silent aviary with no caption overlays — a defensible choice; the assumption is that they have actively chosen no audio and no captions.

### 9.3 Keyboard navigation

- Tab cycles top-bar items: notebook, offers, accessibility, account, then aviary scene focus.
- Aviary scene focus: arrow keys move focus between birds (left-right by perch, up-down between perches).
- Focus indicator: a soft high-contrast outline around the focused bird, visible against bright and dim aviary states. Per the PRD, the visual designer specifies the exact treatment; the outline must pass a contrast test on every day/night phase.
- Enter on a focused bird triggers listen-in. Escape exits listen-in.
- Esc on a top-bar overlay closes it.

A keyboard-trap test asserts that focus can always escape any open surface.

### 9.4 Reduced-motion mode

Already detailed in §7.6. Repeated here: not "animations off"; cross-fade rendering of the same scene.

### 9.5 Contrast

All user copy passes WCAG AA. The design system spec owns the per-surface ratios. The aviary scene itself contains no copy except in the top bar; all top-bar text is on a (slightly opaque) background panel that maintains contrast even in night-phase aviaries.

### 9.6 Focus visibility while top bar is faded

When the top bar fades to 0.1 opacity, focused items remain at full opacity (and the focused state forces the bar back to full opacity). A keyboard user never has a focus indicator on an invisible bar.

### 9.7 Settings surface tone

Accessibility settings, account settings, and any error surfaces are matter-of-fact tone, capitalized as normal English. This is the named exception. An eslint rule on `apps/web/components/system/**` enforces the tone boundary by linting strings against a "no naturalist phrasing here" word list and asserting capitalization on first letters of sentences.

---

## 10. Performance budgets and observability

### 10.1 Bundle budget

- Initial gzipped JS ≤ 2MB. Target ≤ 1.5MB to reserve headroom.
- Code-split surfaces (settings, notebook, visit invite, visitor mode) load on demand.
- A CI gate on bundle size blocks PRs that exceed the budget. The gate fingerprints bundles by hash so a single binary asset addition cannot land without a deliberate review.

### 10.2 TTFB first bird visible ≤ 500ms

Measured on a synthetic mid-tier-mobile (Pixel-equivalent) over emulated 4G. Required path:

- Edge SSR: < 100ms render.
- HTML transfer: < 200ms over 4G.
- Parse + first paint: < 200ms.

If any of these grow, the budget is breached. A perf CI gate measures end-to-end TTFB on a fleet of synthetic browsers from common geographies and alerts on regressions.

### 10.3 60fps idle on 5-yo laptop

Measured by a perf test that runs the aviary at 7-bird capacity with active mood-shaped motion and ambient drift, on a fingerprinted runtime that simulates a 2020-era mid-range CPU. Asserts p95 frame time < 16.6ms over 5 minutes.

### 10.4 Memory growth = 0 over 30 minutes

A perf test runs the aviary at 7-bird capacity for 30 minutes, sampling heap size every 60s. Required: linear regression slope on heap size < 1MB / 30 min. Specifically tested:

- Audio voice nodes are pooled and reused.
- Notebook entries scrolled out of view release their references.
- Scene-rendered birds do not retain frame buffers.
- WebSocket message buffers are bounded.

### 10.5 Tick latency

p50 tick latency < 200ms; p99 < 5s. Alarm at p99 = 5s. A tick that takes longer is an early signal of degradation; the alarm fires before users feel "running slow."

### 10.6 Observability surfaces

- **Aggregate operational telemetry** (allowed): request counts per route, latencies, error rates, anonymized session-duration histograms, render-frame timings (without per-account dimension), audio-context error counts, simulation-tick latencies, queue depths.
- **Synthetic checks** (allowed): a fleet of automated browsers that exercise the aviary, measure first-paint and TTFB, and log results. No real-user data.
- **Real User Monitoring** (allowed at the *aggregate* level only): page-load and first-bird-render histograms, audio-context error counts. Each metric carries no per-account dimension and no per-bird dimension.
- **Per-account interaction state** (forbidden): never exported from the simulation database to telemetry. The simulation database has no read connection from the analytics warehouse. The boundary is enforced by network policy, not by query review.

The dashboards live in `ops/obs/dashboards/`. Alarms route to the on-call rotation; tick-latency alarms are P2 (page during business hours, ticket overnight); error-rate alarms above 1% on any auth/snapshot/events route are P1.

### 10.7 What we deliberately do not measure

- Per-account session duration with account_id attached.
- Per-bird interaction frequencies (offer counts, listen-in counts).
- Visit counts per account.
- "Days active" per account.
- Notebook open frequencies per account.
- Anything that, if surfaced to a stakeholder, would tempt the product into building a dashboard around it.

This is a deliberate choice. The metrics we don't collect can't leak into product decisions that pull the product toward gamification.

---

## 11. Rollout

### 11.1 v1 scope cuts

The launchable v1 is the entire scope in §1.1. The plan is *not* to ramp scope incrementally; it is to ramp population.

- Internal alpha (engineering team): 50 accounts, used daily, full feature set.
- Closed beta: ~500 invited accounts, drawn from a friends-and-family list and a small public-interest list. 4-week duration. Bird-availability cadence reduced to 7d/14d/30d/60d/120d for beta to gather drift-calibration data within a beta cycle. Beta accounts are flagged in the database but not in the user surface.
- Public launch: open signup. Bird-availability cadence returns to PRD defaults (30d / 75d / 150d / 250d / 365d). Population ramps as quickly as infrastructure can scale.

### 11.2 Day-one instrumentation

- Per-tick latency (aggregate).
- Snapshot fanout queue depth.
- WebSocket connection counts.
- Drift-calibration synthetic accounts: 100 fixed-pattern accounts (10-min/day, 20-min/day, 0-min/day baselines, sparse-visit, burst-visit) running continuously to validate that the calibration target holds in production.
- Audio-context error rates per browser version.
- Render-frame timings (aggregate) by browser version.

### 11.3 Day-one experiments we do not run

- A/B testing on the return-greeting variation. The return-greeting is a felt property; A/B testing it would compress its variation into an "optimal" version, which would defeat the procedural-variation premise.
- A/B testing on notebook prose patterns. Same reason.
- Engagement experiments. There is no engagement metric to optimize for.

### 11.4 The bird population ramp

The bird-availability cadence is per-aviary; population ramps as users sign up. There is no "bird supply" that needs to be increased. The population scaling concerns are infrastructure: number of WebSocket connections, tick worker fleet size, snapshot fanout queue throughput.

### 11.5 Soft-launch playbook

- Week 1 of beta: monitor tick latency, audio-context errors, render-frame distributions. Fix any P1.
- Week 2: turn on the visit-invitation feature. Monitor `visits` table growth; verify revocation works end-to-end.
- Week 3: ramp synthetic drift-calibration accounts to 1000. Verify drift calibration still holds at this fan-out.
- Week 4: open signup with rate limits; ramp limits over the next month.

### 11.6 Browser-support dropouts

If a user opens the aviary on an unsupported browser (more than 2 major versions old on Chrome/Safari/Firefox/Edge):

- They see a matter-of-fact unsupported-browser surface explaining what's needed: "Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge. Yours is older. Please upgrade and try again."
- They do not see a degraded aviary; they see no aviary, just the message.
- The reasoning: a degraded aviary on an old browser would probably look bad in a way that reflects on the product, and the user is well-served by clarity.

### 11.7 What we ship together vs. behind-flag

V1 surfaces ship as a single release. The accessibility surfaces are not behind flags. The reduced-motion mode is not behind a flag; it activates on `prefers-reduced-motion`. Captions are not behind a flag; they're an accessibility setting toggle. The visit-invitation flow is shipped on day one of public launch but defaults off per the PRD.

A small number of engineering-only flags exist for emergency rollback: tick-cadence override, snapshot-rate override, audio-engine kill-switch (forces silent + captions for everyone). These are operations levers, not product flags. They are not surfaced to users.

### 11.8 Documentation deliverables at launch

- Public privacy policy (matter-of-fact, naming the aggregate categories of telemetry, explicitly excluding per-bird interaction state, naming the architectural boundary).
- A short "what is Pocket Aviary" page outside the app for prospective users.
- Internal runbook for on-call: tick-latency alarm, snapshot fanout, audio-context errors, account-deletion flow.
- Internal data-pipeline documentation describing the simulation-database / analytics-warehouse boundary.

---

## 12. Risks

The risks worth naming are the ones the PRD cares about: drift calibration, sync correctness, audio uncanniness, and accessibility regressions. Each is paired with a mitigation that is part of the plan, not an afterthought.

### 12.1 Drift calibration drift

**Risk:** the drift function's per-trait gains are too high or too low, so users experience either "my bird changed visibly between sessions" (too fast) or "nothing about my bird ever changes" (too slow). Symptom is silent — no test fails — and shows up in user feedback months later.

**Mitigation:**
- A `tests/drift-calibration/` suite with fixture playback (§5.2) runs per-build and asserts the design targets.
- 100+ drift-calibration synthetic accounts run continuously in production with fixed presence patterns and assert per-tick that drift trajectories match expectations.
- Any change to drift constants requires a calibration-test review, treated as a load-bearing change.

### 12.2 Sync correctness

**Risk:** a code path lets a client write personality state, or a tick consumes events out of order, or a sim worker double-applies events.

**Mitigation:**
- The `birds` schema has no PATCH/PUT endpoint; the schema is the spec.
- A grep-based check in CI prevents new code from mutating bird trait fields outside `services/sim/`.
- Event consumption is idempotent on `(applied_in_tick, event_id)`; a replay is a no-op.
- Tick processing is transactional and ordered by `(emitted_at, event_id)` per account.
- Periodic invariant checks: bird traits never decrease; total drift across an account is monotonically non-decreasing per trait.

### 12.3 Audio uncanniness

**Risk:** the procedural call grammar lands in a register that sounds wrong — either too "synthy" (electronic instead of bird-like), or too repetitive (the variation isn't enough), or too uncanny (mood modulators distort calls into unnatural shapes).

**Mitigation:**
- Audio design review with an external sound designer at the end of week 4 of beta.
- A "two-call repetition test": the same bird, same mood, two calls within 30 seconds — assert MFCC spectral similarity is above 0.4 and below 0.8 (similar enough to be the same voice; varied enough to be alive).
- Ear-test sessions at each beta milestone with at least 5 listeners; failures are tracked as P1 against the 4-week beta budget.
- Audio kill-switch (silence + captions) shipped from day one, so a degraded audio launch is recoverable.

### 12.4 Accessibility regressions

**Risk:** a future feature lands without honoring screen-reader narration, captioning, or reduced-motion. The most likely regression is a new top-bar item that is keyboard-traversable but not properly focus-indicated against night phases.

**Mitigation:**
- `packages/naturalist/` is the single source of voice; new surfaces routed through it inherit narration automatically.
- A CI snapshot test takes the rendered aviary at noon, evening, and night phases and runs an automated contrast check against every focus-indicator pixel.
- A keyboard-trap E2E test asserts focus can always escape any open surface.
- The accessibility settings UI is the matter-of-fact register; an eslint rule on those files asserts no naturalist phrasing.

### 12.5 Privacy boundary erosion

**Risk:** an engineer adds a "convenient" log line that includes account_id-derived bird information, or wires the simulation database into the analytics warehouse for a "harmless" report.

**Mitigation:**
- Network policy denies analytics-warehouse reads of the simulation database.
- A periodic audit query (run by infrastructure security) asserts no log files contain bird_id or per-account interaction-event payloads.
- Telemetry fields are allowlisted at the metric definition; new fields require security review.
- Synthetic UUID-only identifiers in any cross-service message; an eslint rule rejects email patterns in logger calls.

### 12.6 Notebook voice drift

**Risk:** the notebook generator starts emitting bland, generic prose ("a bird greeted you"), or worse, user-behavior phrasing ("you've been watching every day").

**Mitigation:**
- A unit test asserts no notebook entry contains second-person address, "every day," "streak," "in a row," or any visit-frequency phrasing.
- Notebook entries are sampled and reviewed weekly during beta; the templates are tuned based on review.
- A notebook-quality dashboard (aggregate, no per-account) tracks entry length distribution and bird-name interpolation rate; dramatic changes are surfaced.

### 12.7 Empty-loading regressions

**Risk:** a future engineer adds a spinner to the loading state because it's the safe pattern.

**Mitigation:**
- The loading surface is rendered by SSR HTML, not by a component; there's no "spinner component" to add.
- A visual-regression test of the loading surface asserts no spinner glyphs are present.
- Code review on `apps/edge/` is an explicit "high care" zone in the engineering handbook.

### 12.8 Bird identity loss

**Risk:** a database migration, species-pool update, or sync bug effectively replaces a user's bird with a "new" version, losing drift history.

**Mitigation:**
- `bird_id` is a UUID generated at adoption; it is the *only* immutable identifier and is never reassigned.
- Every migration on `birds` is reviewed for "does this preserve `bird_id`?" — checklist item.
- A monthly invariant check: every bird's `bird_id` matches one extant bird, no `bird_id` is reused, no bird's adopted_at has shifted.

### 12.9 Subtle gamification creep

**Risk:** an analytics request, a stakeholder ask, or a "harmless" feature like "show your bird's mood history" introduces a gamified surface despite the PRD.

**Mitigation:**
- The non-goals section of the PRD is referenced in every product review.
- The schema's structural absence of streak/visit-count/mood-history columns means a "show mood history" feature requires a schema change, which would surface in review.
- Any feature pitch that adds a per-account aggregate dimension (visit count, days active, etc.) is subject to a "PRD non-goal" gate that requires a written justification.
- The notebook generator's anti-pattern tests are run on every build and would fail loudly if a "you've been here every day" template snuck in.

### 12.10 Operational risk: tick worker outage

**Risk:** the tick worker pool fails for an extended period; the aviary stops advancing.

**Mitigation:**
- Tick workers are leader-leased per shard with 90s TTL; a crashed worker frees its shard within 90s.
- An alarm on "shards with no completed tick in 5 minutes" pages the on-call.
- The events log is preserved across the outage; recovery replays the consumed events from `applied_in_tick IS NULL` once the worker fleet is back.
- Clients during the outage continue to receive snapshots based on the last completed tick; the aviary appears slightly stale but does not error. The user does not see an error surface during a brief outage.

---

## 13. Open questions and follow-ups

These are non-blocking items to track during build:

1. **Exact presence activity-window value.** PRD says "a few minutes, leaning longer." Plan defaults to 4 minutes. Confirm with first round of user testing.
2. **Notebook noteworthy-event ceiling.** Plan defaults to 1/day soft, 2/day hard cap on bursts. Tune in beta.
3. **Per-trait drift gains.** Calibration is in-test; production data may necessitate small adjustments.
4. **Visit notification text.** Plan: "[Pocket Aviary] {host} has invited you to visit." Confirm with the visual designer.
5. **Mood transition table tuning.** Plan ships with a starting table; beta-period observation will refine entries (especially the `wary` propagation rate, which is most sensitive to overshoot).
6. **Snapshot delta encoding.** The plan ships with full-snapshot WebSocket frames at first; deltas can be added in v1.1 if the bandwidth becomes a concern. Initial measurement at beta will tell us.
7. **Top-bar new-species offer surface.** Plan has it as a small dot on the account icon. Visual designer confirms the treatment.

---

## 14. Concluding read

The product is what's left after a long list of refusals. This plan honors every refusal as an architectural property rather than as a policy: the schema can't answer questions we said we wouldn't answer; the audio engine can't ship what we said it can't ship; the sync model can't be in the failure mode we said it can't be in.

The team building this should re-read `product_brief.md` and `non_goals.md` once a quarter for the lifetime of the product. The refusals compound, and so does the discipline.
