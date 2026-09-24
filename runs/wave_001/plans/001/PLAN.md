# Pocket Aviary — v1 Implementation Plan

Status: execution plan for engineering, design, audio, content, and accessibility. It interprets the PRD (`prd/*.md`) into buildable work. Where the PRD is ambiguous or self-contradictory, this plan makes a call and records it in §18 (Decision log). Nothing here is a proposal to change the product.

---

## 0. How to read this plan

- **INV-nn** — invariants. Hard product rules turned into engineering rules. Each has an enforcement mechanism (code, schema, database grant, CI gate, or review gate). If an invariant can only be kept by people being careful, the plan treats that as a gap and adds a mechanism.
- **D-nn** — decisions made where the PRD leaves room or contradicts itself (§18).
- **R-nn** — risks (§17).
- **P-nn** — tunable parameters with initial values (Appendix A). Parameters live in server-side config, are versioned, and change only through a reviewed change process (§16.5).
- "Canonical" means server-authored state. "Presentation" means state the client derives locally, never persists, and never sends back as truth.

---

## 1. Executive summary

Pocket Aviary is a web-only aviary with 2–7 procedurally animated, procedurally voiced birds. Their hidden personalities drift slowly and only upward in response to the user's *presence*. The engineering problem is not scale or feature count. It is **keeping the product's affective promises true at the architecture level**, so they don't depend on every contributor remembering them.

The plan rests on five structural choices:

1. **The server is the only writer of bird state.** A slow (~60 s) server-side tick owns personality, mood, perch placement, weather, and the notebook. Clients render snapshots and append interaction events. They never write state. Multi-device sync falls out of this: there is nothing to merge. Last-write-wins on personality can't happen because no code path accepts a client-written trait value.
2. **Snapshots carry a short timeline, not just a state.** Each tick publishes where each bird *is and will be* over the next ~3 minutes: perch occupancy, flights, and scheduled social calls. That lets any client, at any moment, compute "now" and paint a first frame with birds already mid-action. It is how "no entry animation" is built rather than just promised.
3. **Everything the user perceives as alive is generated, not stored.** Calls come from an AudioWorklet synthesizer driven by per-species call grammars and per-bird voice signatures. Idle motion is a procedural rig driven by mood-keyed behavior. Greetings are realized with fresh per-session randomness. Captions, narration, and notebook prose come from one authored generative grammar in the naturalist voice. There are no audio files and no canned animations.
4. **Presence is measured honestly and counted once.** The client measures the three-signal conjunction (visible ∧ focused ∧ recent trusted activity). The server clamps intervals and unions them across devices, so two open devices never double-count. Presence feeds a two-stage low-pass drift filter that is monotonic by construction and backed by a database trigger that rejects any trait decrease.
5. **Privacy and anti-gamification are architecture, not policy.** Email lives in exactly one encrypted table. Everything else uses a synthetic UUID. The simulation database is unreachable from telemetry infrastructure. Metric schemas are allowlisted in CI. The design system ships no toast, badge, counter, or notification component, so those patterns have nowhere to plug in.

### 1.1 Load-bearing invariants

| ID | Invariant | Primary enforcement |
|---|---|---|
| INV-01 | Only the simulation tick writes personality vectors; no client-submitted absolute values exist in any API. | DB grants (only `sim_tick` role can UPDATE `bird_personality`); API schemas contain no trait fields; code-owner review on the tick module. |
| INV-02 | Personality traits never decrease. | Pure drift function produces non-negative deltas (property-tested); DB trigger rejects `NEW.x < OLD.x`; nightly integrity job compares against checkpoints. |
| INV-03 | Personality values are never shown to the user in any product surface, and never sent to the client. | Snapshot compiler emits only quantized presentation parameters; snapshot JSON schema forbids trait keys; CI test greps built bundles and fixture snapshots for trait names. (Export exception handled in D-02.) |
| INV-04 | Presence = visible ∧ focused ∧ trusted pointer/key activity within window W, evaluated continuously; presence-time is the union across devices. | Client presence module with unit and browser tests; server-side clamping and cross-device interval union; persona tests in the drift harness (e.g. "tab open in background 48 h ⇒ zero presence"). |
| INV-05 | No announcement surfaces: no welcome text, toasts, banners, badges, unread counts, streaks, visit-frequency displays, or push/email about the aviary. | Design system has no such components; DOM-audit E2E tests on return flows; banned-copy lint; no Notifications/Badging API usage (lint). |
| INV-06 | The first aviary frame shows motion already in progress; no spinner, fade-from-static, or entry sequence. | Boot renderer paints from the inline snapshot's timeline; visual-regression test on the first frame; lint bans spinner components in aviary code. |
| INV-07 | Bird identity is stable forever: `bird_id` is immutable and never reassigned, re-created, or swapped. | Append-only identity table (no DELETE grant except the hard-delete job); migration review rule; species-pool changes are additive config versions. |
| INV-08 | Email appears in exactly one place (encrypted), never as an identifier, key, log field, or metric label. | Separate identity schema and role; `Email` value type with redacting serializers; log-redaction tests; metric-label allowlist. |
| INV-09 | Per-bird and per-account interaction state is never aggregated, never used across accounts, and never reaches telemetry or third parties. | Network and credential isolation between the simulation DB and the analytics stack; telemetry schema allowlist in CI; no third-party analytics or session replay SDKs (CSP + dependency lint). |
| INV-10 | Calls are always procedural; no recorded-audio path exists, including fallback. | No audio assets in the repo (CI check on file types); WebAudio-unavailable path is silence + captions. |
| INV-11 | Visitors never generate presence or interaction events and never alter host state. | Visitor credentials are read-only by construction (separate token type, rejected by all write endpoints); E2E privilege tests. |
| INV-12 | Two voice registers: naturalist for product surfaces, matter-of-fact for system surfaces. | Two copy catalogs with register-specific lint rules; surface inventory (§2.3) assigns each surface a register; content review gate. |

---

## 2. Scope

### 2.1 In v1

**Aviary and birds**
- One aviary per account. Two starter birds, system-selected from a pool of 6 species. The cap is 7 birds, engine-enforced.
- New birds ("newcomers") become available purely by aviary age (D-18). No catalog. The user names birds at adoption and can rename them any time.
- Server-side tick (~60 s, P-01) that advances personality drift, mood, perch placement, bird-to-bird social events, weather, and day/night whether or not any client is connected.
- Personality vector with 5 hidden traits (boldness, social warmth, vocal frequency, plumage saturation, curiosity). Drift is slow and monotonic.
- Mood: wary, content, curious, drowsy, alert, plus `roosting` (asleep/settled at night; D-20). Mood persists across sessions.
- Procedural calls (client-side WebAudio synthesis), chorus, call-and-response, and mood contagion.
- Single horizontal scene with three perch zones, local-time day/night, rare weather, ambient leaves and feathers, subtle parallax, and responsive layout that never crops a bird.

**Interactions**
- Return-greeting, idle presence, listen-in, offer (seed, song fragment, still pool) with a per-bird cooldown, settle with a 5 s undo, and the field notebook (auto-generated, read-only, sparse, infinitely scrollable).

**Accounts and platform**
- Email magic-link sign-in (15-minute expiry, single use, rate limited). Unified sign-up/sign-in (D-37).
- Per-device sessions with a session list and revocation. Verified email change. JSON export delivered by emailed download link. Soft delete for 30 days, then hard delete.
- Multi-device sync as a property of the architecture.

**Social**
- Visit invitations: per-invite, email-addressed, one-time link, read-only ambient view. Revocable, and invitations expire after 30 days unused.
- Visit log in account settings. Opt-in visit notifications, off by default.

**Accessibility (ships at v1, not after)**
- Screen-reader narration in naturalist prose with slow cadence.
- Reduced-motion rendering as its own designed register.
- Call captions generated from the same call score that was synthesized.
- Full keyboard navigation, visible focus, and WCAG AA contrast on all user copy.

**Performance and operations**
- Initial JS ≤ 2 MB gz (hard cap; internal target far lower).
- First bird visible < 500 ms on the reference profile (§13.1).
- 60 fps idle on a 5-year-old mid-range laptop. No memory growth over 30 minutes, tested in CI.
- Aggregate-only RUM and synthetic monitoring. Tick-latency p99 alarm at 5 s.
- Supported browsers: last two majors of Chrome, Safari, Firefox, and Edge. Older browsers get a matter-of-fact unsupported surface.

### 2.2 Explicitly out of v1, and how each exclusion is enforced

| Excluded | Enforcement beyond "don't build it" |
|---|---|
| Native apps | No native-specific protocol concessions. The API is designed for browsers only (cookies, edge HTML); we do not version for store review cycles. |
| Gamification: achievements, streaks, levels, scores, badges, XP, counters such as "birds adopted: 2", green-dot calendars, milestone celebrations | No schema fields for visit counts or streaks. No API returns the host's own visit history or frequency. Banned-copy lint. The design system has no badge, counter, or progress component. PR checklist item (Appendix B). |
| Tamagotchi mechanics: death, hunger, distress, decaying happiness | No state variable can represent need or suffering. The mood set has no distress state. Property test: absence-only input never raises wary occupancy above baseline. |
| Social network surfaces: profiles, follows, feeds, discovery, comments, chat, avatars, mutual visits, leaderboards, show-off mode | Visitor token type can only read one host's snapshot. No display-name or profile fields exist. No cross-account queries exist in the simulation service. Leaderboard-style stats are never computed (INV-09). |
| Push notifications and aviary-related email | No Push API, service-worker push handler, Notifications API, or Badging API. Email templates are limited to the system set in §5.7, and none contain aviary content. |
| Payments, shared aviaries, multi-aviary accounts, customizable scenes | `aviaries.account_id` is UNIQUE. No scene-configuration fields exist. |
| Showing personality numbers anywhere, including debug views | INV-03. Internal debug tooling reads canonical state only in staging. Production support access is consent-gated and audited (§12.6) and never presents trait values in the product UI. |
| Recorded audio | INV-10. |
| SSO and passwords | The auth module supports only the magic-link factor. |
| User-controlled bird placement | No placement API. Perch position is tick-owned. |
| Localization | English only in v1 (D-23). The naturalist grammar is authored in English, but copy is externalized so localization remains possible later. |

### 2.3 Surface inventory and voice register

Every user-visible surface is listed here. Any new surface must be added to this table in the same PR that introduces it, and CI checks that each copy key belongs to a registered surface.

| Surface | Register | Notes |
|---|---|---|
| Aviary scene | none (no text) | No labels, tooltips, buttons, or badges inside the scene. |
| Return-greeting | naturalist (narration only) | Visual and audio only. The narration describes the bird's action. |
| Offer tray | naturalist | "a seed", "a song fragment", "a still pool", "welcome the newcomer" (D-18). Song fragments have naturalist names ("three falling notes"). |
| Settle control | naturalist label, system-style accessible hint | Label "settle". The accessible description states the undo affordance plainly. |
| Field Notebook panel | naturalist | Panel title "Field Notebook" (proper UI label). Entries are lowercase prose. |
| Narration (screen reader / optional visible text) | naturalist | |
| Captions | naturalist | |
| Adoption (starters, newcomer naming) | naturalist | "two birds have arrived." Name fields come prefilled with suggestions. |
| Loading / quiet field | none | No text for at least the first 8 s. A matter-of-fact line appears only on real failure (§7.6). |
| Sign-in, magic-link landing, session timeout | matter-of-fact | PRD examples used verbatim where given. |
| Account settings (email, devices, birds/rename, export, privacy link, delete) | matter-of-fact | Renaming is a settings action, so it uses system voice. |
| Accessibility settings | matter-of-fact | |
| Visits (invite form, invitations, visit log, notification toggle) | matter-of-fact | |
| Visitor view: scene | none | Identical renderer. |
| Visitor view: "visit no longer available" | matter-of-fact | |
| Error / sync / offline lines | matter-of-fact | Inline quiet line near the top bar, not a toast (D-34). |
| Unsupported-browser page | matter-of-fact | |
| All emails | matter-of-fact | No aviary content in any email. |

---

## 3. Architecture

### 3.1 System shape

```
                         ┌───────────────────────────── Browser ─────────────────────────────┐
                         │ boot renderer ─► scene renderer (Canvas2D) ◄─ projection/timeline │
                         │ audio engine (AudioWorklet voices) ◄─ call scheduler ─► captions   │
                         │ presence tracker ─► event queue ─┐     narration (ARIA live)       │
                         │ top bar / panels (lazy chunks)   │     snapshot poller             │
                         └──────────────┬───────────────────┼──────────────▲────────────────┘
                     HTML + inline      │ POST /events,     │ GET snapshot │
                     snapshot (edge)    │ POST /offers      │ (ETag)       │
                         ┌──────────────▼───────────────────▼──────────────┴───────┐
                         │ Edge worker (auth via signed edge token, reads snapshot │
                         │ KV, streams HTML; static assets immutable at CDN)       │
                         └──────────────┬───────────────────────────────────────────┘
                                        │
          ┌─────────────────────────────▼──────────────────────────────┐
          │ API service (stateless, TypeScript/Node)                   │
          │  auth · events ingest · offers (sync resolution) · notebook│
          │  birds/rename · adoption · visits · export/deletion        │
          └───┬───────────────┬──────────────────┬──────────────┬──────┘
              │               │                  │              │
   ┌──────────▼───┐  ┌────────▼─────────┐  ┌─────▼──────┐ ┌─────▼────────┐
   │ Identity DB  │  │ Simulation DB    │  │ Snapshot   │ │ Mailer       │
   │ (PII schema, │  │ (aviaries, birds,│  │ store      │ │ (only service│
   │ sessions,    │  │ personality, mood│  │ (Redis +   │ │ that decrypts│
   │ magic links) │  │ events, notebook)│  │ edge KV)   │ │ email)       │
   └──────────────┘  └────────▲─────────┘  └─────▲──────┘ └──────────────┘
                              │                  │
                   ┌──────────┴──────────────────┴───────┐
                   │ Tick workers (bucketed, leased)      │
                   │ step() → state + ledger + candidates │
                   │ → snapshot compile → publish         │
                   ├──────────────────────────────────────┤
                   │ Notebook writer · Adoption scheduler │
                   │ Export worker · Deletion worker      │
                   └──────────────────────────────────────┘

   Telemetry plane (separate network segment and credentials):
   RUM collector · metrics (OTel/Prometheus) · logs (redacted, 14-day retention) · synthetic probes
   ── no route or credentials into Simulation DB or Identity DB ──
```

### 3.2 Components and responsibilities

| Component | Responsibility | Writes |
|---|---|---|
| **Edge worker** | Validates a short-lived signed *edge token* cookie (no DB call). Fetches the aviary snapshot from edge KV. Streams HTML with critical CSS and the inline snapshot. Serves visitor pages the same way with a visitor token. Checks the revoked-session KV. | nothing canonical |
| **API service** | Authentication, session management, event ingestion (validation, idempotency, clamping), synchronous offer resolution, notebook reads, rename, adoption acceptance, visits, export/deletion requests. Stateless, horizontally scaled. | events, offer ledger, names, invitations, identity records |
| **Tick workers** | Run `step()` for leased buckets of aviaries every tick. Persist state with compare-and-set on `last_tick_index`. Append the personality delta ledger and emit observation candidates. Compile and publish snapshots. | personality, mood, perch, weather, attunement, observation candidates, snapshots |
| **Notebook writer** | Hourly pass per aviary (host-local). Selects candidates under the sparsity budget and renders entries through the grammar. | notebook entries |
| **Adoption scheduler** | Daily pass. Creates newcomer availability windows from aviary age (§6.11). | adoption offers |
| **Mailer** | Only component with KMS decrypt rights for email ciphertext. Renders the system-voice templates. Sends through a transactional provider. | send log (no content, no plaintext address) |
| **Export worker** | Builds the JSON export, stores it encrypted in object storage with a 7-day lifecycle, and asks the mailer to send the link. | export records |
| **Deletion worker** | Daily. Hard-deletes accounts past 30 days of soft delete and crypto-shreds per-account keys. | deletions |
| **Telemetry plane** | RUM collector (first-party endpoint), metrics, logs, synthetic probes. Physically and credential-isolated from both databases. | telemetry stores only |

### 3.3 Writer-ownership matrix (the sync model in one table)

| State | Sole writer | Clients may… |
|---|---|---|
| Personality vector, drift reservoirs, attunement | Tick | never read raw values |
| Mood, perch, activity timeline, weather, social-call schedule | Tick | read via snapshot |
| Offer reaction outcomes | API offer resolver (writes an *event*; the tick applies consequences) | request an offer |
| Interaction events | API ingest (append-only) | append via `/events` |
| Bird name | API rename (optimistic concurrency) | request rename |
| Notebook entries | Notebook writer | read |
| Account settings, a11y prefs | API | update via PATCH |
| Greeting realization, micro-motion, mix levels, settle lighting, captions, narration text | the client (presentation only, never persisted) | own locally |

### 3.4 Technology choices (recommendations, fixed unless a spike disproves them)

- **TypeScript end to end.** This is the single most important stack choice, because server and client must share deterministic modules: the greeting planner, caption generator, grammar engine, timeline evaluator, PRNG, and species catalog. Node 22 LTS on the server; ES2022 target in the browser (last-two-majors support makes this safe).
- **PostgreSQL 16** as the system of record. Separate databases (or at minimum separate clusters' schemas with separate roles and network policies) for identity and simulation. Synchronous replica plus PITR.
- **Redis** (regional) for the snapshot store, rate limits, and hot cooldown cache. **Edge KV** (the CDN provider's) for globally replicated snapshots and the revoked-session list.
- **Edge compute** (Cloudflare Workers / Fastly Compute class) for HTML streaming and edge-token validation.
- **Object storage** with a KMS-encrypted bucket for exports.
- **Client runtime:** no framework in the scene path (vanilla TS, a small signal-based store). **Preact** for the top bar and lazily loaded panels (settings, visits, notebook). Canvas2D for the scene. AudioWorklet for synthesis. Vite/Rollup build with enforced size budgets.
- **Infra:** containers on a managed orchestrator, Terraform, OpenTelemetry, self-hosted error collection with scrubbing (no SaaS error tracker that receives user payloads). One primary region at launch, multi-AZ.

### 3.5 Shared deterministic core (`aviary-core`)

`aviary-core` is a pure TypeScript package with no DOM or Node APIs. It is imported by tick workers, the API (offer resolution), the notebook writer, and the client.

- **Determinism rules.** Every decision that the server and client must agree on, or that must be reproducible in tests, uses an integer PRNG (PCG32 via `Math.imul`) and fixed-point arithmetic. It never uses `Math.random` or transcendental functions (`Math.sin`, `exp`, `pow`), because their last-ulp results differ across JS engines. Transcendental math is allowed only in rendering and audio synthesis, which are presentation-only.
- **Seed derivation.** `seed = hash64(aviary_seed, bird_id, tick_index, stream_tag)`. Streams are named: `mood`, `perch`, `weather`, `social`, `greeting`, `offer`, `notebook`. Adding a stream never perturbs existing streams.
- **Server-only subpath.** `aviary-core/server/*` contains the drift model, behavior compiler, and attunement. A lint rule prevents client code from importing it, and a bundle check greps the build for its module IDs. This keeps drift math out of the browser entirely.
- **Versioning.** The snapshot schema carries `schema_version`. Clients accept N and N−1 so rolling deploys are safe. The tick's `step()` carries `engine_version`, recorded on every ledger row.

### 3.6 Environments

- **dev**: local stack (docker compose) with a fake mailer that writes to a local inbox UI.
- **staging**: production-shaped. Supports a **simulated clock per aviary** (staging-only config) so aviaries can be aged at 60× or more. This is needed to test newcomers, saturation, and 7-bird scenes months before real aviaries get there. It is also where the staff dogfood calibration cohort runs (§6.14).
- **production**: no simulated clock (the code path is compiled out behind a build flag), and no debug surfaces.
---

## 4. Data model

All primary keys are UUIDs: v7 where time ordering helps, v4 for anything exposed in URLs. The only link between the identity plane and the other planes is `account_id`. Timestamps are UTC `timestamptz`. Trait and continuous engine values are stored as **fixed-point integers** (micro-units, 0–1,000,000) so drift arithmetic is exact, deterministic, and monotonic-checkable.

### 4.1 Identity plane (identity DB, role `identity_svc`; only the mailer role may decrypt)

```sql
accounts (
  account_id            uuid PK,               -- synthetic; the only account reference anywhere
  status                enum('active','pending_deletion') NOT NULL,
  created_at            timestamptz NOT NULL,
  deletion_requested_at timestamptz NULL,
  timezone              text NOT NULL,         -- IANA; canonical aviary tz (D-22)
  prefs                 jsonb NOT NULL,        -- a11y + visit-notification toggle (schema-validated)
  prefs_version         int NOT NULL
)
account_pii (
  account_id            uuid PK FK,
  email_ct              bytea NOT NULL,        -- AES-256-GCM under per-account DEK (envelope, KMS)
  email_bidx            bytea UNIQUE NOT NULL, -- HMAC-SHA256(normalized email, bidx key) for lookup
  dek_wrapped           bytea NOT NULL,        -- crypto-shredded at hard delete
  pending_email_ct      bytea NULL, pending_email_bidx bytea NULL,
  pending_token_hash    bytea NULL, pending_expires_at timestamptz NULL
)
magic_links (
  link_id uuid PK, token_hash bytea UNIQUE,      -- SHA-256 of 256-bit random token; raw token only in email
  purpose enum('sign_in','email_change'), email_bidx bytea, account_id uuid NULL,
  created_at, expires_at (created_at + 15 min), consumed_at NULL
)
sessions (
  session_id uuid PK, account_id uuid, token_hash bytea UNIQUE,
  device_label text,                             -- coarse, from UA: "Safari on iPhone"
  created_at, last_seen_at, revoked_at NULL, idle_expires_at
)
```

- Email normalization for the blind index: trim, lowercase the domain, Unicode NFC. Do **not** strip plus-tags or dots; that is provider-specific and error-prone.
- `accounts.prefs` holds: `captions`, `reduced_motion` (`system|on|off`), `narration_visible`, `keep_top_bar_visible`, `char_shortcuts_enabled`, `sound_enabled`, `volume`, `visit_notifications`. Nothing in prefs is usage history.

### 4.2 Simulation plane (simulation DB; roles `sim_tick`, `api_rw`, `notebook_writer`)

```sql
aviaries (
  aviary_id uuid PK, account_id uuid UNIQUE NOT NULL,
  created_at timestamptz NOT NULL,               -- aviary age drives adoption (never visit count)
  seed bigint NOT NULL,                          -- procedural root seed
  tick_phase smallint NOT NULL,                  -- 0..59, spreads load across the minute
  last_tick_index bigint NOT NULL,               -- CAS guard
  event_cursor bigint NOT NULL,                  -- last consumed event seq
  weather jsonb NOT NULL,                        -- current/next weather episode
  attunement_umicro int NOT NULL,                -- aviary-level recency signal (§6.5)
  last_presence_end_at timestamptz NULL,         -- for greeting absence class; never displayed
  engine_version int NOT NULL
)
birds (                                          -- identity; INSERT-only except name
  bird_id uuid PK, aviary_id uuid NOT NULL,
  species_id text NOT NULL,                      -- references versioned species catalog
  voice_seed bigint NOT NULL,                    -- fixed at adoption: the recognizable call signature
  plumage_seed bigint NOT NULL,                  -- fixed individual markings
  name text NOT NULL, name_version int NOT NULL,
  adopted_at timestamptz NOT NULL,
  ceilings jsonb NOT NULL                        -- per-trait asymptotes, fixed at adoption (§6.4)
)
bird_personality (                               -- UPDATE granted ONLY to sim_tick
  bird_id uuid PK,
  boldness int, warmth int, vocal int, plumage int, curiosity int,       -- micro-units
  res_boldness int, res_warmth int, res_vocal int, res_plumage int, res_curiosity int, -- reservoirs
  day_released jsonb,                             -- per-trait release so far today (daily cap)
  bird_attunement_umicro int,                     -- per-bird component from listen-in (§6.5)
  version bigint, updated_tick_index bigint
)
-- trigger: RAISE if any NEW.trait < OLD.trait (INV-02); DELETE revoked from every role but deletion_worker
personality_ledger (bird_id, tick_index, engine_version, d_boldness, …, created_at) -- append-only audit
personality_checkpoints (bird_id, taken_at, vector, reservoirs)                    -- daily, 35-day retention
bird_state (                                     -- written by sim_tick
  bird_id uuid PK,
  mood enum('wary','content','curious','drowsy','alert','roosting'),
  mood_since timestamptz, mood_intensity int, mood_min_dwell_until timestamptz,
  perch_zone enum('front','middle','back'), perch_slot smallint,
  activity text, timeline jsonb,                 -- next ~3 min of segments (§6.7)
  vocal_damp_until timestamptz NULL,             -- e.g. rain
  recent_greeting_forms smallint[]               -- ring buffer of last 5 greeting skeleton ids
)
offer_ledger (bird_id uuid PK, cooldown_until timestamptz, last_offer_id uuid)       -- API-written
interaction_events (                             -- append-only, partitioned by day
  seq bigserial, event_id uuid UNIQUE,           -- client-generated UUIDv7 => idempotency
  aviary_id uuid, session_id uuid, type text, bird_id uuid NULL,
  payload jsonb,                                 -- per-type JSON schema; no free text
  client_ts timestamptz, received_at timestamptz, consumed_tick bigint NULL
)
presence_intervals (aviary_id, session_id, start_at, end_at, present_ms, audible_ms, event_id) -- normalized from events
greeting_history (aviary_id, local_date, first_bird_id, order bird_id[])                    -- 60-day retention
observation_candidates (aviary_id, tick_index, kind, salience int, local_date, slots jsonb, consumed bool)
notebook_entries (
  entry_id uuid PK, aviary_id uuid, written_at timestamptz, local_date date,
  template_id text, slots jsonb,                 -- structured; bird refs by bird_id
  rendered_seed bigint                           -- deterministic re-render
)
adoption_offers (offer_id uuid PK, aviary_id, ordinal smallint, species_id, voice_seed, plumage_seed,
  window_start, window_end, status enum('lingering','welcomed','departed'))
aviary_snapshots_meta (aviary_id PK, tick_index, etag, published_at)   -- bookkeeping only
```

### 4.3 Social plane (simulation DB, role `api_rw`; email columns encrypted as in §4.1)

```sql
visit_invitations (
  invite_id uuid PK, host_account_id uuid,
  visitor_email_ct bytea, visitor_email_bidx bytea, dek_ref,  -- visitor is a data subject too
  token_hash bytea UNIQUE,
  created_at, expires_at (created_at + 30 d),   -- if unused
  bound_at NULL,                                 -- first use (one-time link consumed)
  access_expires_at NULL,                        -- bound_at + 30 d (D-14)
  revoked_at NULL
)
visit_sessions (visit_session_id uuid PK, invite_id, token_hash, started_at, last_seen_at, ended_at NULL)
```

The approximate visit duration is `last_seen_at − started_at` plus one poll interval. A new `visit_session` row starts when a visitor pulls a snapshot more than 30 minutes after that session's last pull.

### 4.4 Content and configuration (versioned files, shipped to server and client)

- **Species catalog** (`species@vN.json`): id, naturalist common name used in prose ("warbler", "wren", "finch", "thrush", "dove", "nightjar"), silhouette rig parameters, palette ramp (muted to full chroma), perch preferences, circadian profile (diurnal or nocturnal), call grammar reference, and trait base values and ceilings. Changes are additive only. A bird's `species_id` is never remapped (INV-07).
- **Call grammars** (`grammar/calls/<species>.json`): motif definitions (§9.3).
- **Prose grammar** (`grammar/prose/*.grammar`): notebook, narration, captions, greeting narration, offer-reaction narration (§11).
- **Offer library**: 3 offer kinds, plus 6–8 song fragments defined as motif scores (not audio).
- **Engine parameters** (P-nn): server config, versioned, with audit history.

### 4.5 Retention schedule

| Data | Retention | Rationale |
|---|---|---|
| Personality vector, mood, birds, notebook | Life of account; hard delete at day 30 after deletion request | Canonical relationship state. |
| Interaction events, presence intervals | 35 days after consumption, then deleted | Needed only to drive the user's own simulation: the notebook's "this week" lookbacks and a replay window that matches backup PITR. |
| Personality ledger | 400 days | Integrity audit and repair (§4.6). Never read at runtime. |
| Personality checkpoints | 35 days (daily) | Point-in-time repair. |
| Greeting history | 60 days | "first time this week" style observations. |
| Magic links | 24 h after expiry | Abuse forensics. |
| Sessions | 30 days after revocation or expiry | Device list accuracy. |
| Visit invitations and log | 12 months, or until host account deletion | Host transparency. Visitor data minimized. |
| Exports | 7 days in object storage | Download window. |
| Operational logs | 14 days | Ensures logs that mention a UUID age out before hard delete. |
| Backups (simulation DB) | 14-day PITR | Residual copies expire within 14 days after hard delete. Disclosed in the privacy policy. PII is crypto-shredded immediately. |

### 4.6 Protecting the personality vector (the worst possible failure)

The PRD calls losing a personality vector "the worst possible failure of this product", and notes that it would pass every unit test. The plan therefore layers defenses:

1. **Single writer.** Database grants: only `sim_tick` may UPDATE `bird_personality`. No role (including migrations run by humans) may DELETE from it except `deletion_worker`, which deletes only for hard-deleted accounts.
2. **Monotonic trigger** rejects decreases (INV-02). A bug that would "reset" a bird fails loudly instead.
3. **Compare-and-set ticks.** `UPDATE … WHERE last_tick_index = :k-1` makes double-application impossible.
4. **Append-only delta ledger and daily checkpoints.** These exist *only* for audit and repair after an engineering fault. The runtime never derives personality from them or from events, and the ledger is not an event log. Recovery procedure: restore the checkpoint, then replay ledger deltas to the target tick. It is rehearsed in staging every quarter.
5. **Nightly integrity job.** For every bird, verify: vector ≥ yesterday's checkpoint element-wise; vector present for every bird row; ledger sum ≈ vector − checkpoint. It emits a count of violations only (an operational metric with no account dimension). Any nonzero count pages on-call.
6. **Migration discipline.** Any migration touching `birds`, `bird_personality`, or `bird_state` requires two-person review. It must pass a dry run against a masked production-shaped dataset in staging with before/after vector hashes compared.
7. **Identity continuity.** Species-pool changes, rig changes, and grammar changes are rendering concerns keyed by `species_id` and `voice_seed`. They never re-create birds. A species retired from the adoption pool is still rendered for birds that already have it.
---

## 5. API surface

### 5.1 Conventions

- JSON over HTTPS under `/v1`. Host authentication uses the `__Host-pa_session` cookie (HttpOnly, Secure, SameSite=Lax, opaque token, hashed server-side). The edge also receives `__Host-pa_edge`, a signed token with 10-minute TTL containing `aviary_id` and `session_id` and nothing else. Visitors use `__Host-pa_visit`, a distinct token type accepted only by visitor read endpoints (INV-11).
- **CSRF:** SameSite plus a required `X-PA-Client: 1` header plus an Origin check on every state-changing request.
- **Idempotency:** every client write carries a client-generated UUIDv7 (`client_event_id`). Replays return the original result.
- **Errors:** `{ "error": { "code": "link_expired", "retryable": false } }`. The client maps codes to matter-of-fact copy. No server-provided prose reaches the product surface.
- **Rate limits:** per session and per account on writes. Per email blind index and per IP on magic links. Per host on invitations (P-40..P-43).
- **Caching:** snapshots are `Cache-Control: private, no-store` at shared caches. Freshness uses an ETag of `tick_index:schema_version`.

### 5.2 Endpoints

| Method & path | Purpose | Notes |
|---|---|---|
| `POST /v1/auth/magic-links` | Request sign-in link `{email}` | Always 202 with identical body (no account enumeration). |
| `POST /v1/auth/magic-links/consume` | `{token}`, sets session cookie | Creates the account on first use (D-37). Returns `{account_state: new/existing/pending_deletion}`. 410 `link_expired` / `link_used`. |
| `GET /v1/auth/sessions` | Device session list | `{session_id, device_label, created_at, last_seen_at, current}`. |
| `DELETE /v1/auth/sessions/{id}` | Revoke a device | Immediate at the API. Also pushed to the edge revocation KV (≤60 s). |
| `POST /v1/auth/sign-out` | Revoke the current session | |
| `POST /v1/account/email-change` | `{new_email}`, sends verification to the new address | The old email keeps working until confirmation. |
| `POST /v1/account/email-change/confirm` | `{token}` | Swaps ciphertext and blind index atomically. |
| `GET/PATCH /v1/account` | Timezone, prefs | PATCH uses `If-Match: prefs_version`. |
| `POST /v1/account/export` | Request export | 202. Email with link when ready. |
| `GET /v1/exports/{id}?t=` | Download | Requires the token *and* a signed-in session for the same account (D-02). |
| `POST /v1/account/deletion` | Soft delete | Status becomes `pending_deletion`. Other sessions are revoked. |
| `DELETE /v1/account/deletion` | "I changed my mind" | Available from any signed-in page during the window. |
| `GET /v1/aviary/snapshot` | Current snapshot (§5.3) | 304 on ETag match. Stale-tolerant. |
| `POST /v1/aviary/events` | Batch of ≤50 events, ≤16 KB (§5.4) | 202 with `{accepted[], rejected[{id,code}]}`. |
| `POST /v1/aviary/offers` | Offer with server-resolved reactions (§5.5) | 200 `{offer_id, item, reactions[]}`. |
| `GET /v1/aviary/notebook?cursor=&limit=20` | Notebook page, newest first | Rendered prose with *current* bird names (D-17). |
| `PATCH /v1/birds/{bird_id}` | Rename `{name}` | `If-Match: name_version`. 409 on a concurrent rename. |
| `GET /v1/birds` | Names and species for the settings list | No dates, no stats. |
| `GET /v1/aviary/adoption` | Current newcomer or starter state | Starters: species and suggested names. |
| `POST /v1/aviary/adoption/{offer_id}/welcome` | `{name}`, adopts the newcomer | Rejected if the cap of 7 is reached or the window is closed. |
| `POST /v1/aviary/starters/confirm` | `{names:[a,b]}` | First-run only. |
| `POST /v1/visits/invitations` | `{visitor_email}` | 201. Mailer sends the one-time link. |
| `GET /v1/visits/invitations` | Outstanding and active invitations | Visitor email decrypted for the host only. |
| `DELETE /v1/visits/invitations/{id}` | Revoke | Immediate: token invalidated, visit sessions ended. |
| `GET /v1/visits/log` | Visits, most recent first | `{visitor_email, date, approx_duration}`. |
| `POST /v1/visit/consume` | Visitor exchanges the one-time token for a visit cookie | Binds the invitation to that browser. |
| `GET /v1/visit/snapshot` | Visitor-scoped snapshot | Same visual state. Host-private fields stripped. 410 `visit_unavailable`. |
| `POST /v1/rum` | Aggregate RUM beacon | Schema allowlisted. No identifiers (§13.5). |

The API has **no** endpoint that accepts trait, mood, or perch values. It has no endpoint that returns the host's own visit history, session counts, or durations. The visit log is strictly about *visitors*.

### 5.3 Snapshot schema (v1)

Target size is ≤ 6 KB gzipped at 7 birds. It is compiled by the tick, not assembled per request.

```jsonc
{
  "schema_version": 1, "engine_version": 12,
  "aviary_id": "…", "tick_index": 29012345,
  "generated_at": "2026-…Z", "server_now": "2026-…Z",   // server_now rewritten at serve time
  "timeline_until": "…Z",                              // generated_at + 180 s
  "tz": "Europe/Lisbon",
  "sky": { "phase": "morning", "sun": 0.34, "season": 0.62 }, // stylized, from local clock (D-31)
  "weather": { "kind": "rain", "intensity": 3, "since": "…", "until": "…" } | null,
  "last_presence_end_at": "…Z",                        // greeting absence class; host-only field
  "greeting_plan": {                                   // host-only field
    "plan_id": "…", "order": [ { "bird_id": "…", "eagerness": 5 } ],
    "skip": ["…"], "recent_forms": { "<bird_id>": [3, 1, 4] }
  },
  "birds": [ {
    "bird_id": "…", "name": "pip", "species_id": "warbler",
    "voice_seed": "…", "plumage_seed": "…",
    "look": { "chroma": 19 },                          // 0..31 quantized plumage level
    "mood": "content", "mood_since": "…Z",
    "profile": {                                       // quantized 0..15 presentation levels (INV-03)
      "front_affinity": 9, "greet": 7, "call_rate": 6, "chorus_join": 5,
      "approach": 8, "tilt": 10, "fluff": 3
    },
    "timeline": [                                       // segments, absolute times
      { "k": "perch", "zone": "middle", "slot": 2, "from": "…", "to": "…", "act": "preen" },
      { "k": "flight", "from_slot": ["middle",2], "to_slot": ["front",1], "at": "…", "dur_ms": 1800, "seed": "…" },
      { "k": "social_call", "at": "…", "kind": "call", "reply_to": null, "seed": "…" }
    ]
  } ],
  "active_offer": { "offer_id": "…", "kind": "pool", "x": 0.42, "until": "…Z",
                    "reactions": [ /* as §5.5 */ ] } | null,
  "newcomer": { "offer_id": "…", "species_id": "finch", "voice_seed": "…",
                "timeline": [ /* as birds */ ] } | null
}
```

- The edge rewrites `server_now` at serve time so the client can compute its clock offset.
- The visitor snapshot is the same document minus `last_presence_end_at`, `greeting_plan`, and the ability to act on `newcomer`. The newcomer still renders, because visitors see exactly what the host sees.
- The quantization of `profile` and `look` is deliberately coarse. Its purpose is to render behavior, not to leak traits. There is no invertible mapping from a single level back to a trait value, because each level mixes trait, mood, attunement, and time of day.

### 5.4 Event schema

| `type` | Payload | Server treatment |
|---|---|---|
| `session_start` | `{entry: navigation/visibility/reengage, greeting: {plan_id, form, primary_bird_id} \| null, absence_class}` | Records the greeting outcome (the canonical greeter comes from `plan_id`, not the client claim). Updates `greeting_history` on the first session of the host-local day. |
| `presence_interval` | `{start, end, present_ms, audible_ms}` | Clamped to `[max(start, received_at − 60 min), received_at]` and to `present_ms ≤ end − start`. Normalized into `presence_intervals`. The tick computes the cross-device union (§6.3). |
| `listen_in_start` / `listen_in_end` | `{bird_id, at}` | The tick pairs them per session. Unpaired starts close at the session's last presence end. Duration counts only the time intersected with presence. |
| `settle` | `{at}` | Emitted only after the 5 s undo window elapses, or on `pagehide` during the window. Closes the session's presence window. Small mood-quieting input. |
| `reengage` | `{at}` | Opens a new presence window after settle. |

Every event is rejected if the credential is a visitor token (INV-11), the session is revoked, or the payload fails schema validation. Payloads contain no free text.

### 5.5 Offer resolution (synchronous, server-authoritative)

An offer needs a visible reaction within a second, but the tick runs once a minute. The API therefore resolves the *reaction* synchronously, and the tick applies the *consequences*: mood nudge, drift inputs, and perch changes in later timelines.

1. The client plays the offered item's appearance immediately: a seed dropping, a pool settling, or the song fragment playing softly. This masks the round trip.
2. `POST /v1/aviary/offers {client_event_id, kind, fragment_id?, drop_x, near_bird_id?}`. `drop_x` sits in the front zone. If a bird is being listened in on, the item lands at the front-zone point nearest it (D-10).
3. The API loads canonical `bird_state` and the presentation profile from the latest compiled snapshot, plus the cooldowns from `offer_ledger`. It runs `resolveOffer()` from `aviary-core` with seed stream `offer`:
   - For each bird not on cooldown, weighted choice of reaction class. Weights come from mood, the `approach` level, distance to the drop point, and time of day.
   - Seed: `approach_now`, `approach_later` (wary: wait 20–90 s, then come near), `watch`, `ignore` (drowsy/roosting).
   - Song fragment: `join_in`, `go_quiet`, `call_against`, weighted by vocal-driven `chorus_join` and mood.
   - Pool: `drink`, `bathe`, `watch`.
   - Birds on cooldown get `glance`, a small acknowledgement with no approach, and no drift input. The user never sees a timer (D-09).
4. In one transaction: append an `offer` event (reactions included; `bird_id`s of acceptors and "near" birds) and set `cooldown_until = now + P-20` for each bird that approached or engaged.
5. The response carries reactions with `{bird_id, reaction, start_delay_ms, seed}`. The client choreographs them. The next compiled snapshot includes `active_offer`, so a second device shows the same item and the birds at it.
6. If the request fails, the item remains and fades after its normal lifetime, and no bird reacts beyond a glance. After 3 consecutive failures, the quiet system line from §7.6 appears.

### 5.6 Visit-invitation flow

```
Host                          API / Mailer                           Visitor browser
 │ POST /visits/invitations ─►│ rate-limit; create invite (token_hash, │
 │  {visitor_email}           │ 30 d unused expiry); mailer sends link │
 │                            │ ───────────── email: /visit#t=… ──────►│
 │                            │                                        │ GET /visit (static page; token in
 │                            │                                        │ URL fragment → never hits logs/edge)
 │                            │◄──── POST /v1/visit/consume {token} ───│ (JS POST; link scanners that only
 │                            │ bind invite to this browser; set       │  GET cannot consume — D-36)
 │                            │ __Host-pa_visit; access 30 d (D-14)    │
 │                            │───────── 200 → navigate /visit ───────►│ edge streams HTML + host snapshot
 │                            │◄──── GET /v1/visit/snapshot (60 s) ────│ polls double as the visit heartbeat
 │ (opt-in only) email:       │ visit_session start/last_seen updated  │
 │ "<visitor> visited…"       │                                        │
 │ DELETE /invitations/{id} ─►│ revoke → next poll returns 410 ───────►│ "This visit is no longer available."
```

- Tokens are carried in the URL **fragment** so they never reach server logs, the edge, or `Referer` headers. The landing page POSTs the token.
- The invitation email identifies the host by their account email address (the only identity that exists, since there are no profiles). The invite form discloses this before sending (D-15). The email has no free-text message field, so invitations can't carry spam or phishing text.
- Host limits: 10 new invitations per day and 30 outstanding or active at once (P-42/43).
- Visitors do not need an account. If a visitor is also a Pocket Aviary user, their host session and visitor session are independent cookies.
- Visit notification (opt-in): at most one email per visitor per day, sent at the start of a visit session, matter-of-fact, no aviary content (D-16).

### 5.7 Auth, account, and email flows

- **Magic link:** a 256-bit token in the URL fragment. The landing page auto-POSTs it to `/consume`, with a visible "Continue" button as a noscript and scanner-safe fallback. Links expire in 15 minutes and are invalidated at consumption. Requesting a new link does not invalidate outstanding ones; each is independently single-use and expiring. Rate limit: 5 per 15 minutes and 20 per day per email blind index, 30 per hour per IP (hashed with a daily-rotating salt for rate limiting only).
- **Session lifetime:** 90-day sliding idle expiry (D-26). An API 401 produces "Your session timed out. Sign in again to keep watching." Events queued at that moment are discarded (drift is slow; losing minutes is harmless and avoids any cross-session attribution).
- **Email change:** the verification link goes to the new address. The old address keeps working until confirmed. A notice goes to the old address after the swap (security hygiene, system voice).
- **Deletion:** soft delete revokes other sessions and schedules the hard delete at +30 days. Signing in during the window shows a matter-of-fact panel on every signed-in page: "Your account is scheduled for deletion on 12 October. Keep my account." Hard delete removes all rows keyed by `account_id` or `aviary_id` in both databases, crypto-shreds the DEK, deletes exports, and relies on the 14-day log and backup expiry for residue.
- **Export JSON:** `{account: {created_at, timezone, prefs}, birds: [{bird_id, name, species, adopted_at, mood, personality_state: "pa1:<base64url>"}], notebook: [{local_date, text}], visits: {...}}`. Personality is included as an opaque, versioned, checksummed encoding for portability, not as a human-readable number panel (D-02).
- **Email template set** (complete for v1): sign-in link, email-change verification, email-changed notice, export ready, deletion scheduled, visit invitation, opt-in visit notification. No other templates exist, and a CI test fails if a template outside this list is registered.
---

## 6. Simulation engine

### 6.1 Tick scheduling and execution

- **Cadence.** Every aviary ticks every 60 s (P-01), offset by its `tick_phase` second. v1 ticks *every* aviary every minute, connected or not (D-27). Because `step()` is pure and deterministic, a future "dormant aviary" optimization (advancing N ticks in one batch for aviaries with no pending events and no connected clients) is a scheduling change with provably identical results, not a semantic change. The equivalence property test (§14) exists from day one so the optimization stays safe.
- **Work units.** Aviaries hash into `S` shards (start at 64). A work unit is `(shard, phase_second)`. Once per minute, the scheduler enqueues 64 × 60 units into a Postgres work queue. Workers claim them with `SELECT … FOR UPDATE SKIP LOCKED` and a 30 s lease.
- **Batch execution** (≤500 aviaries per batch):
  1. Load aviary rows, birds, personality, state, and cooldowns in one round trip.
  2. Load unconsumed events (`seq > event_cursor AND received_at ≤ tick_time`) for the batch.
  3. For each aviary, run `step()` (§6.2) for every missed tick index up to now. Normally this is 1. After an outage it can be up to 1,440 inline; beyond that, a dedicated catch-up worker takes over.
  4. In one transaction: bulk-UPDATE state with `WHERE last_tick_index = k-1` (CAS), append ledger rows and observation candidates, and advance `event_cursor`.
  5. After commit: compile snapshots (§5.3) and publish to Redis and edge KV. Publishing is idempotent and retryable, because the snapshot is a pure function of committed state and the tick index.
- **Capacity.** Budget ≈ 0.3 ms CPU per aviary-tick, plus amortized DB cost. At 250k aviaries that is ~4.2k aviary-ticks/s: ~9 batch transactions/s and ~1.5 vCPU of step compute. The DB is the constraint, so bulk statements and HOT-updatable rows (no indexes on frequently updated columns) are mandatory. Load tests target 10× the forecast.
- **Latency SLO.** Tick latency is the time from scheduled tick time to snapshot published. p99 < 5 s alarms (PRD). The internal target is p99 < 1.5 s. The per-batch compute time is also recorded.

### 6.2 `step()` contract

```ts
step(prev: AviaryState, events: NormalizedEvents, tick: TickContext): {
  next: AviaryState,              // personality (+reservoirs), attunement, mood, perches, weather, timelines
  ledger: PersonalityDelta[],     // non-negative by construction
  candidates: ObservationCandidate[],
  metrics: { invariantViolations: number }  // aggregate counter only
}
// TickContext: { tickIndex, tickTime, localTime (canonical tz), params: EngineParams, engineVersion }
```

Rules:

- `step()` has no I/O. Randomness comes only from `seed(aviary_seed, bird_id, tickIndex, stream)`.
- Absence is the normal case. With no events, `step()` still advances release of drift reservoirs, attunement decay, the mood hazards, weather, and the perch/social timeline.
- Every sub-model is a separate pure module with its own property tests: presence, drift, attunement, mood, perch, social, weather, greeting planner, adoption, and candidates.

### 6.3 Presence ingestion and presence-time

- The client measures presence per §8.5.4 and sends `presence_interval` chunks: every 30 s during continuous presence, plus a final chunk when presence ends.
- The server clamps each interval (§5.4) and stores it per session.
- At each tick, presence seconds for the tick window are the **measure of the union of all sessions' intervals intersected with the window**. Two devices both "present" at the same moment count once (D-05). The result, `P_tick ∈ [0, 60]`, is the only presence input to drift.
- `audible_ms` is unioned the same way and is used only as the vocal-drift modifier (D-08).
- Visitor sessions produce no events at all (INV-11).
- **Daily concavity.** Presence has diminishing returns within a host-local day: `P_eff(D) = C·(1 − e^(−D/C))`, with `C = 3600 s` (P-10), where `D` is the day's raw presence so far. The per-tick contribution is `P_eff(D + P_tick) − P_eff(D)`. This bounds what an all-day open-and-active session, a mouse jiggler, or automation can contribute, so the drift calibration holds for every usage pattern. A 1 h session yields ~0.63 effective hours. Twelve hours yields ~1.0.

### 6.4 Personality drift: a two-stage, monotonic low-pass filter

The PRD calls for a low-pass filter over presence and interaction signals. It must be slow (no session moves a trait visibly), monotonic toward expressive, and continue during absence "based on inputs from before they left". The plan implements that as a **reservoir-and-release** filter per bird, per trait.

**Stage 1 — inflow into the reservoir** (per tick):

| Input | Traits affected (weight per effective hour, P-11..P-16) |
|---|---|
| Presence `ΔP_eff` (aviary-wide; every bird present in the scene) | boldness 1.0, warmth 0.8, vocal 0.6 × (0.5 + 0.5·audible_share), plumage 1.0, curiosity 0.5 |
| Listen-in on bird b (effective seconds, daily concave with `C_L = 900 s`) | warmth_b 1.6, vocal_b 1.6 (listen-in is "a strong signal of attention") |
| Offer accepted by b (approach / engage reaction) | curiosity_b +0.08 credit per accepted offer, ≤ 4 per day |
| Offer landed near b (within the proximity radius) | boldness_b +0.04 credit, ≤ 4 per day |
| Settle | no drift input. It only closes the presence window cleanly. |
| Absence, mute, no offers | **zero inflow**. Nothing is ever subtracted. |

**Stage 2 — release into the trait** (every tick, present or not):

```
r      = R · (1 − e^(−Δ/τ))                          τ = 2 days (P-17); computed via fixed-point LUT
R     -= r
room   = max(0, (c − x) / c) ^ γ                     γ = 1.5 (P-18); c = per-bird ceiling
Δx     = min( g · r · room,  dailyCap − releasedToday )   g = 0.08 (P-19); dailyCap = 0.012 (P-21)
x     += Δx                                          Δx ≥ 0 always (INV-02)
```

Why this shape:

- **Monotonic by construction.** `r ≥ 0` and `room ≥ 0`, so `Δx ≥ 0`. The DB trigger is a second line of defense, not the design.
- **Continues during absence.** The reservoir drains over days, so the bird keeps changing for a while after the user leaves, based only on what happened before they left. Nothing is invented at return.
- **No visible single session.** Inflow is concave per day, release is spread by τ, and the daily cap (0.012) sits well below the perceptibility threshold (≈0.06, see calibration).
- **Individuality survives years.** Per-bird ceilings `c ∈ [0.70, 0.95]` are drawn at adoption from species ceiling ± individual variation. Two heavily watched birds converge to *different* expressive characters rather than all saturating at 1.0. Seeds start at species base ± N(0, 0.05) in the 0.15–0.45 range, which leaves headroom.
- **Err slow.** Drift can't be undone (INV-02 plus the no-reset rule). Too-fast gains would permanently over-drift every bird, while too-slow gains can be corrected upward later. Launch parameters sit at the slow edge of the calibration band (R-01).

**Calibration targets as testable bands**, using the "regular visitor" persona of §6.14 (5 sessions/week, ~10 min presence each, 1–2 listen-ins, ~1 offer per session):

| Horizon | Target mean Δ over the five traits | Rationale |
|---|---|---|
| After 1 session | ≤ 0.008 on every trait | Never visible. |
| Day 7 | 0.015 – 0.035 | "Measurable in instruments after about one week". |
| Day 21 | 0.06 – 0.10, with ≥ 1 trait ≥ 0.07 | "Visible to the user after about three weeks". 0.06 is roughly one plumage chroma step and a meaningful front-perch occupancy shift (§6.7). |
| Day 90 | 0.15 – 0.30 | Relationship deepening, not saturating. |
| "Tab open in background 48 h" persona | exactly 0 | INV-04. |
| "Visible and focused, no activity, 8 h" persona | ≤ W minutes of presence per idle span | Kiosk/away case. |
| Heavy (60 min/day, daily) | Visible by ~day 8–10 | Capped by the daily cap; never session-to-session visible. |

With g = 0.08, τ = 2 d, and the concave inflow, first-order estimates put the regular persona at ≈0.025 at day 7 and ≈0.085 at day 21. The harness (§6.14) is the authority, and parameters are tuned there before launch.

### 6.5 Attunement: "quieter after absence" without negative drift

The PRD needs two things to hold together. Traits never move down on neglect. Yet a user returning after two weeks finds birds "quieter than they were", "greeting less often because less often is what's been observed". Personality (monotonic, slow) and mood (daily) can't express that alone. The plan adds one bounded internal signal, **attunement**, and fences it in (D-06):

- `A ∈ [A_min, 1]` with `A_min = 0.30` (P-25), held aviary-wide, plus a per-bird component raised by listen-in.
- **Rises** with presence: `A += (1 − A) · k_up · ΔP_eff`, with `k_up` chosen so ~30–40 minutes of presence over a couple of sessions takes A from floor to ~0.8.
- **Relaxes** during absence toward `A_min` with a 5-day half-life (P-26).
- **Only affects expression**: greeting eagerness, front-perch affinity, and call rate *when observed* (via the compiled `profile` levels). It never feeds the mood engine's wary hazard, never changes traits, and never produces a posture or behavior that reads as distress.
- **At the floor, birds are ambient.** They are still perched, preening, and calling at their personality's unobserved rate, and one bird always greets (§6.10). The PRD's "quieter, not mistrustful" holds by construction. A property test asserts that for any absence length, wary occupancy stays ≤ its no-absence baseline.
- Attunement is not personality: it is not in the export's personality encoding and not in the drift ledger. It is never displayed.

### 6.6 Mood model

- **States:** `wary`, `content`, `curious`, `drowsy`, `alert`, `roosting` (D-20). The internal name `roosting` avoids confusion with the settle gesture; prose renders it as "settled" or "asleep".
- **Dynamics:** a per-bird continuous-time Markov process evaluated each tick. The transition probability is `1 − e^(−λΔ)`. The destination is chosen by weighted draw over `baseline(personality) × context multipliers`, with a minimum dwell of 8 minutes (P-30) except for strong impulses.
- **Personality baseline:** `wary ∝ 0.25(1 − boldness)`, `content ∝ 0.35 + 0.3·warmth`, `curious ∝ 0.15 + 0.4·curiosity`, `alert ∝ 0.1 + 0.2·boldness`. As boldness drifts upward, wary becomes rarer: expressive drift shows through mood without mood ever being set directly.
- **Circadian** (host-local stylized sun, §6.9):
  - Diurnal species: drowsy pressure rises from ~60 min before sunset. Roosting dominates from ~45 min after sunset until ~30 min before sunrise. Alert peaks at dawn, and dawn chorus probability peaks.
  - Nocturnal species (the nightjar-like one): the inverse. It is active and calls into the late hours, and roosts in daylight on a sheltered back perch.
- **Recent interactions** (from consumed events): an accepted offer multiplies content ×2.0 for 20 minutes. Listen-in multiplies content/alert ×1.4. Presence alone gives a mild curious ×1.2.
- **Ambient events:**
  - Rain: vocal damp ×0.4 for its duration plus 10 minutes. It is a call-rate multiplier, not a mood.
  - Wind: alert ×1.5 for high-boldness birds, wary ×1.5 for low-boldness birds.
  - Alarm call from another bird: wary impulse on birds in the same or adjacent zone with probability `0.5·(1 − boldness)`. Duration 3–10 minutes, with no minimum dwell, so wary lifts naturally.
- **Daily-ish reset:** around host-local dawn, the pull toward the personality baseline is multiplied ×3 for 90 minutes. Mood relaxes, but never snaps (D-35). There is no reset on session open. The mood a bird ended with yesterday is the mood the tick has evolved it into today.
- **Guardrails (property tests):**
  - No transition path depends on absence length.
  - Wary occupancy under absence-only input ≤ baseline wary occupancy.
  - No mood persists >24 h except roosting across the night and nocturnal day-roosting.
### 6.7 Perch placement and timeline generation

- **Slots.** Front (3 slots), middle (4), back (4), plus a ground/pool strip in the front zone used only for offers. Slots are defined in normalized scene coordinates (§8.2). High branches sit in the middle and back zones.
- **Target zone** per bird is a weighted draw each time the bird decides to relocate:
  - toward front: `front_affinity` (boldness, attunement, curious/content mood)
  - toward back: wary and roosting
  - drowsy: low, sheltered perches
  - warmth pulls birds toward slots adjacent to other birds
- **Relocation rate:** a Poisson process with mean dwell 4–12 minutes, depending on mood and species (alert birds relocate more often).
- **Timeline.** Each tick writes the next 180 s for each bird: perch segments with activity labels (`preen`, `scan`, `rest`, `watch`, `forage`, `drink`, `bathe`, `roost`), flights (`from`, `to`, `at`, `dur`, `seed`), and scheduled social calls (§6.8). Timelines overlap across ticks, and a newer tick overrides older segments from its `generated_at` forward. Segments are coarse. Micro-motion inside a segment is client-procedural (§8.3).
- **Conflict-free slots.** Two birds never target the same slot at the same time (reservation table in `step()`).
- **Newcomer:** appears on back-edge slots only, and only during its visit windows (§6.11).

### 6.8 Bird-to-bird interaction, chorus, and alarm contagion

- **Call-and-response.** When a bird's social call is scheduled, each other bird may respond with delay 0.4–2.5 s. Probability is `0.15 + 0.5·warmth_responder`, scaled by mood (drowsy ×0.3, roosting ×0.05).
- **Chorus events.** Emerge when ≥2 birds with high `chorus_join` are in calling moods in the same 60 s window. Probability peaks at dawn and softens at dusk. Rain suppresses them. A chorus is a scheduled cluster of overlapping social calls, never a single stacked cue.
- **Contagion.** Wary spreads by proximity (§6.6). Content spreads weakly, ×1.1 to neighbors of a content bird. Contagion is capped per tick to prevent cascades.
- **Alarm calls.** Rare (P-33: expected 0–2 per aviary-day), triggered by wind gusts or random startles for low-boldness birds. They are never tied to user behavior or absence.
- Idle calls outside the social schedule are client-generated (§9.4). The tick schedules only the calls that involve more than one bird, because those must be canonical and notebook-visible.

### 6.9 Day/night and weather

- **Local time.** Everything is anchored to the account's canonical IANA timezone (D-22), so every device shows the same light.
- **Stylized sun (D-31).** Sunrise and sunset come from local clock time with gentle seasonal variation. Latitude is approximated from a bundled table mapping each IANA zone to its reference city (hemisphere included). There is no geolocation. Near the poles the curve is clamped so there is always a night and a morning.
- **DST transitions** are handled by computing local time from UTC each tick. A skipped or repeated hour eases lighting over 30 minutes rather than jumping.
- **Timezone change** (travel, or a device reporting a new zone for more than 30 minutes of presence): the canonical tz updates, and the sun curve eases to the new local time over ~45 minutes. Moods follow the circadian hazards naturally.
- **Weather** is synthetic and seeded per aviary per day (D-30). Rain happens ~3 times a week (Poisson), 10–40 minutes each, at modest intensity. Soft wind happens ~4 times a week, 5–20 minutes. There are no storms, snow, or anything assertive. Weather episodes are in the snapshot and are canonical across devices and visitors.

### 6.10 Greeting planner (server) and realization (client)

The greeting is the anchor moment. It has to honor boldness, mood, and absence length, and it must never be canned.

- **Server, per tick:** compile `greeting_plan`:
  - `order` = birds sorted by a weighted draw on `greet` level (warmth, boldness, attunement, mood; drowsy and roosting heavily down-weighted).
  - `skip` = birds whose greet probability draw fails today. Wary birds often skip.
  - `recent_forms` = the last 5 greeting skeletons per bird.
  - The plan is seeded by `(aviary, tick, greeting)`. It is canonical, so the notebook can truthfully say "pip greeted before wren today".
  - **At least one bird always greets.** The first entry of `order` is never skipped.
- **Client, at session start** (fresh navigation, return to visible, or re-engage after settle):
  1. **Absence class** = `now − max(last_presence_end_at, locally known hidden-since)`. Classes (P-35):
     - A: < 2 min
     - B: 2 min – 2 h
     - C: 2 h – 2 days
     - D: > 2 days
  2. **Form selection** from a skeleton library by class × the bird's `greet` level × mood, excluding `recent_forms`:
     - A: a glance up, or a pause in preening.
     - B: head-tilt toward the viewer, with a quiet 1–2 note call.
     - C: step toward the front of its perch, or hop to a nearer slot, with a call.
     - D: re-orientation. The bird flies to a front slot if bold, or calls longer; another bird may answer.
  3. **Continuous variation** inside the form, from a fresh `crypto.getRandomValues` session seed: onset 350–1,400 ms after the first painted frame, head angles, gaze targets, call motif and contour (§9.3), and body shuffle.
  4. **Stagger.** If a second bird participates (class C/D, or a warmth-driven reply), it follows after a randomized 0.6–2.5 s offset. Never unison, never all birds.
  5. Send `session_start {plan_id, form, primary_bird_id, absence_class}`.
- **Exclusions:** no greeting for visitors (INV-11), and no greeting when the page opens in a hidden tab. The greeting runs on the first transition to visible.
- **Reduced motion:** the greeting becomes a cross-fade to an attentive pose plus the call. Narration delivers the greeting promptly (§10.1).
- **Test:** 10,000 simulated greetings per class produce no identical `(skeleton, quantized parameter vector)` pair across any 20 consecutive greetings for the same bird.

### 6.11 Adoption: starters and newcomers

- **Starters.** At account creation the server picks two distinct *diurnal* species, maximizing a precomputed acoustic-contrast matrix between species grammars so the two birds are easy to tell apart by ear. The nocturnal species is never a starter: a first encounter with a sleeping bird is wrong. Voice and plumage seeds are drawn, ceilings and seeds set, and suggested names drawn from a curated list (short, gentle, no species names).
  - The user sees the adoption surface (naturalist voice): "two birds have arrived." Each bird has a small procedurally drawn portrait and a prefilled name field. There is no catalog.
  - After confirming, the aviary opens on the quiet field. The first bird flies in to its starting perch, and the second follows after a randomized 3–8 s. This is the only fly-in entrance in the product, and it never happens again.
- **Newcomers (third bird onward; D-18).** Availability is a function of aviary age only:
  - Bird 3 window opens at age 75 ± 10 days (seeded jitter). Later windows open every 60–80 days, so a year-old aviary can reach 5–6 birds.
  - No newcomer when the aviary holds 7 birds, or when a global ceiling flag (P-45, §16.2) is reached. Such a window is deferred, not lost.
  - **Arrival as a noticing:** during an open window (up to 21 days), a bird of a not-yet-present species (or, if all 6 are present, a same-species bird with a distinct voice seed) lingers on back-edge perches during daylight. It is intermittently present, calls, and never enters the front zone. It renders for visitors too.
  - **Discovery:** the offer tray gains "welcome the newcomer" while a newcomer is present. Narration mentions it ("a young finch lingers near the back perch."). A notebook observation is eligible ("a finch has been lingering at the edge of the aviary these last few days."). There is no toast, badge, or prompt.
  - **Welcome:** a naming sheet (naturalist voice, suggested name prefilled). On confirmation the bird is adopted with a new immutable `bird_id` and moves into the regular perch model.
  - **Not welcomed:** the newcomer "departs" at window end and the next window is scheduled normally. There is no penalty, no counter, and no "missed" state.

### 6.12 Notebook generation

- **Candidates** come from `step()`: typed facts about the aviary only. Examples:

  | Kind | Example fact |
  |---|---|
  | `greet_order_change` | pip greeted before wren, first time in 7 days |
  | `long_quiet` | a long quiet stretch this morning |
  | `extended_preen` | pip preened for several minutes |
  | `weather_passage` | a short rain passed |
  | `dawn_chorus` | dawn chorus |
  | `night_caller` | the nightjar called late |
  | `perch_shift` | wren now often on the front perch, first time in weeks |
  | `newcomer_lingering` | a newcomer has been lingering |
  | `arrival` | a bird was adopted |
  | `plumage_glint` | pip's wing-bars look brighter in the low light (qualitative, only after a level change) |

  Each candidate carries a salience score (P-50).
- **Forbidden candidate kinds** are enforced by an allowlist of kinds; anything else fails CI. That rules out anything about user behavior: visits, session counts or durations, return frequency, days away, "you". The line is the PRD's: observations of the aviary, never of the user.
- **Sparsity budget:** a token bucket per aviary. Capacity 2, refill 1 token every 3 days (≈ one entry every few days). A separate "noteworthy" bucket (capacity 1, refill 1 per 2 days) serves only candidates with salience ≥ 80, such as `arrival`, `greet_order_change` "first time", and `newcomer_lingering`. The budget is time-based, not activity-based, so heavy users do not get more entries.
- **Writer pass:** hourly in host-local time, skipping 23:00–05:00 local. It picks the highest-salience unconsumed candidate that fits the budget, renders it with the prose grammar (§11), stores `template_id`, `slots`, and the render seed, and marks candidates consumed. Duplicate suppression: the same kind won't repeat within 10 days, and templates track recent use per aviary.
- **Absence:** the writer keeps running during absence (the aviary kept going), but at the same sparse cadence. Entries never frame the absence ("while you were away" is banned copy).
- **Rendering:** names resolve at read time to the bird's current name, lowercased in prose (D-17). Entries read like `tuesday — pip greeted before wren today, first time this week.` The day name comes from `local_date` in the canonical tz.
- The notebook is read-only for users: no edit, delete, or annotate endpoints exist.

### 6.13 Call-grammar runtime: division of labor

The server never synthesizes audio. It owns *when* multi-bird calls happen (§6.8) and each bird's `call_rate` and `chorus_join` levels, via the compiled profile. The client owns *what* each call sounds like (§9). Each bird's voice signature is fixed by `voice_seed` and species. Mood and drift change the delivery (tempo, register spread, phrase length, loudness), never the signature (§9.3).

### 6.14 Drift and behavior calibration harness

Production drift or mood data can't be aggregated (INV-09), so calibration must be done before launch, offline and in staging. The harness is a first-class deliverable, owned by the engine team, and runs in CI.

- **Offline harness:** runs the real `step()` over simulated time, typically 120 days at 1-minute ticks, fed by **synthetic personas** that generate client-realistic event streams, including presence produced by a simulated browser-signal model.
- **Personas:**
  - regular (5×/week, 10 min)
  - light (2×/week, 5 min)
  - heavy (daily, 60 min)
  - binge-then-absent (daily for 3 weeks, then 14 days away, then return)
  - weekend-only
  - two-device overlap (laptop and phone active simultaneously)
  - tab-in-background 48 h
  - focused-idle kiosk 8 h
  - mouse-jiggler 12 h/day
  - listen-in enthusiast
  - offer spammer (offers every 10 s)
  - settle-every-time vs never-settle
  - timezone traveler
  - visitor-heavy host (10 visitors, zero drift contribution expected)
- **Assertions:**
  - the calibration bands of §6.4
  - monotonicity
  - absence invariants
  - settle vs tab-close equivalence (identical drift for identical presence)
  - offer-cooldown saturation guard (the spammer's curiosity at day 7 ≤ 1.3× the regular persona's)
  - two-device union (drift identical to single-device with the same union)
  - batch-advance equivalence
  - mood occupancy distributions per persona within design bands (e.g. wary ≤ 15% of daylight time for median-boldness birds)
  - greeting-first frequency shifts toward bolder birds
- **Outputs:** trajectory plots per trait, mood occupancy, greeting orders, notebook-entry cadence, and a single pass/fail summary. Designers review plots before each parameter change.
- **Staging dogfood cohort:** 30–60 staff and consenting friends use staging aviaries for ≥ 6 weeks before GA (§15). Weekly diary prompts ask whether the birds feel different, same, or weird. Designers may inspect these staging aviaries' internals because participants consented to a research environment separate from the production privacy promise. None of this data ever migrates to production.
---

## 7. Sync model

### 7.1 Principles

1. **One canonical record per aviary.** The simulation DB is authoritative, and the tick is the only writer of bird state (§3.3).
2. **Clients read snapshots and append events.** There is no client-to-client channel, no client-side persistence of bird state, and nothing to merge.
3. **Conflicts are made unreachable rather than resolved.** Every write is either an append (events), a server-side function of canonical state (offers, ticks), or a user-authored scalar protected by optimistic concurrency (names, prefs).

### 7.2 Read path

| Trigger | Action |
|---|---|
| Navigation | Edge-inlined snapshot (≤ 1 tick old). No fetch needed for the first frame. |
| Keepalive while visible | `GET /snapshot` every 60 s ± 10 s jitter with `If-None-Match`, phase-aligned ~5 s after the aviary's tick phase so most polls hit a fresh tick. |
| `visibilitychange → visible` | Immediate fetch, then the greeting (§6.10). |
| Long frame gap (> 5 s between rAF callbacks: laptop suspend, OS sleep) | Immediate fetch. Timeline reconciliation (§8.4) animates any change as natural movement. |
| After `POST /offers` | A fetch at the next tick boundary picks up `active_offer` and new perches. |
| Hidden tab | No polling. Rendering stops. The server keeps ticking. |

The client's timeline covers 180 s, so a single missed poll is invisible. If no fresh snapshot arrives by `timeline_until`, the client **holds**: birds stay on their perches with full idle micro-motion and calls at the last known rates. The aviary never freezes.

### 7.3 Write path

- An in-memory event queue with bounded size (500; presence chunks coalesce, so it never grows in practice).
- **Flush** every 30 s. Also flush immediately on `listen_in_end`, `settle`, and `session_start`, on `visibilitychange → hidden` (via `fetch(…, {keepalive: true})`), and on `pagehide` (`navigator.sendBeacon`).
- **Retry** with exponential backoff and jitter. `client_event_id` makes retries safe. Events older than 60 minutes are dropped client-side; the server would clamp them anyway.
- The queue is never persisted to disk. Presence lost to a crash is minutes, which is harmless under slow drift. Persisting would add storage and privacy surface.

### 7.4 Conflict-prevention catalogue

| Scenario | Why it can't corrupt state |
|---|---|
| Laptop and phone open at the same time, both present | Presence is unioned server-side (§6.3). Listen-ins on both devices each count, but only within the unioned presence and the daily concave caps. |
| Morning laptop session and a lunch phone session that started earlier | No device submits personality. Both sessions' events are appended and consumed in `seq` order. Drift deltas are additive and commutative. Nothing is overwritten. |
| Retry after a timeout duplicates an event | `event_id` UNIQUE, so it is a no-op. |
| Tick worker dies mid-batch; another worker re-runs | CAS on `last_tick_index`: the whole batch transaction either committed or didn't. Re-running is exact. |
| Zombie worker commits late | CAS fails, the batch is discarded, and a stale-lease counter increments. |
| Beacon arrives after the tick consumed later events | It gets a higher `seq` and is consumed next tick. Additive effects make order irrelevant. Mood effects use clamped event times within the last hour. |
| Offer on two devices at once | Each resolves against canonical cooldowns in its own transaction. `offer_ledger` rows are locked `FOR UPDATE`, so a bird can't accept two offers inside a cooldown. |
| Rename on two devices | `If-Match: name_version`. The loser gets 409, and the settings panel shows the current name with a matter-of-fact note: "This name was changed on another device." |
| Prefs on two devices | Same pattern with `prefs_version`. |
| Session revoked mid-write | The API rejects with 401. Queued events are discarded (§5.7). |
| Magic-link replay | The token is single-use and consumption is atomic (`UPDATE … WHERE consumed_at IS NULL`). A replay gets 410 and the PRD's copy. |
| Server outage for hours | The tick catches up deterministically on recovery (§6.1). Clients hold, then reconcile. |

### 7.5 Clock, timezone, and suspend/resume

- **Clock offset.** `server_now` on each snapshot gives the offset, smoothed with a median of the last 5 samples. All timeline evaluation uses server time. Presence intervals are sent as server-time estimates and clamped server-side.
- **Timezone.** Lighting and circadian behavior use the canonical account tz, so the laptop in the morning and the phone at night show the same aviary in the same mood. Device timezone only feeds the tz-change detector (D-22).
- **Suspend/resume.** A long rAF gap triggers a fetch. Birds whose canonical perch changed while the machine slept *fly* there, or cross-fade in reduced motion, over natural durations. They never teleport.

### 7.6 Offline and failure behavior

| Condition | Behavior |
|---|---|
| Snapshot fetch fails, timeline still valid | Nothing visible changes. Retry. |
| No fresh snapshot for > 3 min | Hold mode continues. After 2 more minutes of failure, a quiet one-line message appears next to the top bar (system voice): "Can't reach Pocket Aviary. Trying again." It clears itself on success. |
| Initial load with no inline snapshot (edge miss) and slow network | Quiet field (§8.1). Fetch with retry. After 8 s of failure: "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." |
| 401 | "Your session timed out. Sign in again to keep watching." with a sign-in action. |
| Offer POST fails | §5.5 step 6. |
| Visit revoked | "This visit is no longer available." The scene fades to the quiet field. This is a system transition, not an aviary behavior. |

There are no toasts, and no message is ever shown for success.

---

## 8. Frontend rendering pipeline

### 8.1 Boot sequence and the first frame (INV-06)

1. **HTML** from the edge contains:
   - inline critical CSS (sky gradient as the page background, so even pre-JS pixels are the quiet field)
   - the inline snapshot as `<script type="application/json" id="snap">`
   - `<link rel="modulepreload">` for `boot.js`, also sent as 103 Early Hints
   - the top-bar skeleton in static HTML, faded state
2. **`boot.js`** (≤ 45 KB gz; served immutable, cached by the service worker) contains the projection evaluator, the layout solver, the rig renderer for the snapshot's species, and the background painter. It:
   - reads the snapshot and computes the clock offset
   - evaluates every bird's timeline at `now`: current segment, flight progress, activity phase (seeded so a preening bird is *mid-preen*, a calling bird mid-call with its beak moving)
   - paints background layers and birds in one frame
   - calls `performance.mark('first-bird')` in the rAF after the draw
3. **Runtime chunk** (`aviary.js`, ≤ 250 KB gz) loads right after the first paint and hydrates the rest: continuous rendering, audio engine and worklet, interaction layer, presence tracker, event queue, poller, narration, captions.
4. **Audio** starts at the soonest moment the browser allows (§9.6). If it can start immediately, calls fade in over ~400 ms, which reads as already-audible ambient sound, not an entrance.
5. **Lazy chunks:** notebook panel, settings, accessibility settings, visits, adoption sheet, error surfaces. Prefetched during idle after 10 s.

**No-snapshot path** (edge miss, first load after an idle-token expiry, visitor cold path): the quiet field shows soft sky, perches, and one or two faint cues such as a drifting mote of light or a single leaf. When the snapshot arrives, birds *emerge from the foliage edges* with short, staggered flights to their current perches, as if they had been just out of view (D-33). It is never a fade from nothing, and never simultaneous.

**First-run (empty aviary):** the same quiet field, then the one-time starter fly-in (§6.11).

### 8.2 Scene composition and responsive layout

- **Logical scene.** 1600 × 900 units. Perch slots live in a *safe band*, and background and foreground extend beyond it.
- **Fit rule.** Scale to fit width, then height. Birds' rendered size is `clamp(minBirdPx, scale·base, maxBirdPx)`.
  - **Narrow viewports:** perch x-positions are remapped by a monotonic piecewise function that compresses spacing but preserves zone order and a minimum gap. Birds are never cropped (PRD).
  - **Tall portrait viewports:** the band sits in the vertical middle, with extra sky above and foliage and ground below as a filler that can't hold birds.
  - **Wide viewports:** perches spread apart with more space between them.
  - The minimum supported viewport is 320 × 480. A layout solver test runs all 11 slots, the newcomer strip, and offer positions across a viewport matrix and asserts that every bird's bounding box, including its flight arcs, stays in frame.
- **Layers** (back to front):
  1. sky gradient, re-rendered every 10 s or on palette change
  2. far foliage, cached, gentle sway
  3. mid foliage and perches, cached
  4. birds, per frame
  5. offer items, per frame
  6. foreground occasional branch and particles (leaves, feathers, rain)
  7. DOM overlays: bird focus targets, captions, top bar

  Parallax is subtle: ≤ 6 px at 1600 units, driven by slow ambient drift only, never by pointer position. That avoids a parallax-heavy feel and pointer-motion coupling.
- **Palette.** Calm naturalist ramps (soft blues, greens, warm browns, muted ochre). Time-of-day color grading is applied when layers are cached, not per pixel per frame. Bird colors come from species palette ramps modulated by `look.chroma` and the time-of-day grade. The design system (owned by the visual designer) supplies the exact colors; engineering supplies the ramp mechanism and the contrast tests (§10.5).
- **Canvas.** Canvas2D at `devicePixelRatio`, capped at 2. `ResizeObserver` is debounced to 150 ms. On resize, perches re-solve and birds glide to their new positions within 400 ms, or cross-fade in reduced motion.

### 8.3 Bird rig and idle micro-motion

- **Procedural 2D rig per species:** body ellipse with breathing, head (rotation, tilt, eye and blink), beak (open amount driven by the audio scheduler), wing (fold, shuffle, preen reach, flight wingbeat), tail (flick), legs (hop, shuffle), and a feather "fluff" scalar that scales the body outline with noise. Plumage pattern comes from `plumage_seed` (bars, patches, speckle) and is baked into small cached sprite atlases at boot per bird × chroma level. Individual birds of the same species look distinct.
- **Idle behavior:** a stochastic state machine per bird, keyed by mood and the current timeline activity:
  - wary: scan more, crouch, face away from the front
  - content: preen, rest, slow blinks
  - curious: head-tilts toward sound sources (other birds' calls, offers, drifting leaves)
  - drowsy: low posture, fluffed, long blinks
  - alert: upright, quick head turns
  - roosting: head tucked, eyes closed, breathing only
- **Aliveness rules:**
  - Breathing never stops.
  - There are no perfectly periodic loops. Motion blends incommensurate frequencies and value noise.
  - Every behavior's duration and amplitude are randomized.
  - A bird is never motionless for more than 1.5 s except for breathing and blinks.
- **Allocation discipline:** all per-frame math writes into preallocated typed arrays. There are no closures per frame and no per-frame object creation, which is the foundation for the 30-minute memory test.

### 8.4 Transitions and snapshot reconciliation

- Evaluating timeline segments by absolute server time yields smooth motion across snapshot boundaries: a flight in snapshot N+1 that started in N's window simply continues.
- **Divergence** (a new snapshot places a bird somewhere else than the client shows, for example after a suspend or a stale edge snapshot): the reconciler schedules a *catch-up flight* from the current rendered position to the canonical slot, starting within 0.5–3 s (randomized), using ordinary flight animation. In reduced motion it is a cross-fade between perches.
- **Mood change:** idle behavior weights blend over 5–20 s. Posture shifts (fluff, height) ease; nothing snaps.
- **Day/night:** continuous, from the canonical local time. The settled lighting override (§8.5.3) is layered on top.

### 8.5 Interactions

#### 8.5.1 Listen-in

- **Engage:**
  - click or tap a bird (hit area expanded to ≥ 44 × 44 CSS px)
  - keyboard: focus a bird, then Enter or Space
  - while listening in, clicking or arrowing to another bird transfers listen-in (D-24)
- **Disengage:**
  - click the focused bird again
  - click empty scene space
  - press Escape
  - move keyboard focus out of the scene
  - also on `visibilitychange → hidden` and settle
- **Mix:** a slow, equal-power re-balance (§9.5). There is no visual chrome. The focused bird may glance toward the viewer once, drawn from its behavior library. For keyboard users, the focus ring stays visible (§10.4).
- **Events:** `listen_in_start` and `listen_in_end`.

#### 8.5.2 Offer

- **Open the tray:** top-bar offer icon, or the `o` key (disableable, D-25). The tray is a small popover under the top bar, keyboard navigable with arrows, Enter, and Escape. Items: "a seed", "a song fragment" (opens 6–8 named fragments), "a still pool", and "welcome the newcomer" only while a newcomer is present.
- **Item presentation** (client, immediate):
  - Seed: a few seeds fall onto the ground strip near the drop point.
  - Song fragment: plays softly through the offer voice (§9.1).
  - Pool: a soft reflective surface settles into the front ground strip. It lasts ~6 minutes, then fades.
- **Reaction choreography** comes from the server's reaction list (§5.5). The client generates the approach path, hop sequence, and drink or bathe animations procedurally, and the song-fragment response calls through the voice synth. Wary `approach_later` birds wait, watch, then come near after the delay if the session is still open.
- There is no disabled state and no countdown. Cooldown birds glance.

#### 8.5.3 Settle

- The top-bar settle control (D-01) runs a light ramp to the evening palette over ~6 s, a master audio duck of −12 dB over 4 s, and a bias toward drowsy postures, all local. One bird, the highest `greet`, gives a soft low acknowledgement call. That is the goodbye.
- **Undo window, 5 s.** Any `pointerdown` in the aviary, or Escape, reverses the ramp over ~2 s and restores the mix. No event is sent.
- **After 5 s:** send `settle`. The presence tracker is forced off until re-engage. The aviary stays settled until the tab closes or the user re-engages.
- **Re-engage** is an explicit action: a click in the scene after the window, keyboard activation of a bird, or opening the offer tray or notebook. Pointer movement alone doesn't count, because the user may be reaching for the close button. Re-engage runs a slow lighting return to the true time of day, sends `reengage` and `session_start` (greeting class A/B), and resumes presence.
- Tab-close without settling is equally valid. The server ends presence at the last interval, and no UI ever refers to it.

#### 8.5.4 Presence tracker (client half of INV-04)

- **State:** `present = document.visibilityState === 'visible' && document.hasFocus() && (now − lastActivity) ≤ W`, where W = 4 min (P-02, calibrate within 3–5 min, leaning long).
- **Activity events** (only `event.isTrusted`): `pointermove`, `pointerdown`, `keydown`. `keypress` in the PRD maps to `keydown`, because `keypress` is deprecated and does not fire for arrows or Escape, which our keyboard navigation relies on. `pointerdown` is included so touch devices, where `pointermove` fires only during drags, are not systematically under-counted (D-04).
- **Evaluation:** on `visibilitychange`, `focus`, and `blur`, plus a 1 s check (a timer, not rAF, so evaluation doesn't depend on rendering). Presence is forced false while settled or in visitor mode.
- **Output:** present intervals with `audible_ms` (AudioContext running, not muted, volume > 0), chunked every 30 s.
- **Tests** run in real browsers (§14): background tab, minimized window, another window focused on a second display (visible but unfocused, so the aviary keeps rendering and playing but records no presence; D-12), idle beyond W, and touch-only tapping.
### 8.6 Top bar and panels

- **Controls** (exactly five; D-01), in tab order: field notebook, offer, settle, accessibility settings, account/settings. Icons only, with accessible names. There are no badges, dots, counts, avatars, or status indicators, ever.
- **Fade.** After 4 s of pointer stillness (P-05), the bar fades to ~10% opacity over 1.2 s. It returns to full opacity on `pointermove`, `keydown`, focus entering the bar, or a touch in the top region.
  - It never fades while keyboard focus or an open panel is inside it.
  - "Keep top bar visible" in accessibility settings disables the fade.
  - On touch devices the fade timer runs from the last touch.
- **Panels:** the notebook side sheet, the offer popover, and the accessibility and account settings sheets. Each panel is a lazy chunk that overlays the scene while the scene keeps running underneath: birds and calls continue, and presence continues if the conditions hold.
  - Panels trap focus while open. Escape returns focus to the invoking control.
  - Panel surfaces are the only places with UI density. The aviary scene has none.
- **Notebook panel.** A virtualized list with a pool of about 30 DOM nodes. Pages of 20 load via cursor on scroll. There is no "new" marker. The list opens at the newest entry. Scrollback is unlimited.
- **Account settings** sections: Email, Devices (session list with revoke), Birds (name and species, rename), Visits (invite, invitations, log, notification toggle), Export, Privacy policy (link), Delete account. All in system voice (§2.3).

### 8.7 Reduced-motion rendering

This is a designed register, not motion turned off.

- **Trigger:** `prefers-reduced-motion: reduce`, or the accessibility setting (system / on / off), live-switchable without reload.
- **Key poses.** Each species × activity gets a set of 3–6 authored key poses rendered by the same rig (preen-A/B/C, scan-left/right, tilt, rest, fluffed, roost). Idle behavior becomes slow **cross-fades** between key poses (1.2–2.0 s blends, holds of 4–12 s).
- **Flights** become cross-fades between perches: fade out at the source while fading in at the destination, with a 0.8 s overlap.
- **Removed:** leaf and feather drift, parallax, rain streaks. Rain becomes a gentle cross-faded wet tint and sheen on perches. Wind becomes a slow cross-fade of foliage poses.
- **Kept, slowed:** day/evening color shifts (grade interpolation lengthened 2×), the settle lighting (10 s), and the top-bar fade (opacity only).
- **Unchanged:** calls, captions, drift, mood, notebook, and narration.
- **Greetings and offer reactions** use pose cross-fades. The greeting still lands within 1–2 s via an attentive-pose cross-fade plus a call.
- **Flash safety:** no content flashes more than 3 times per second in any mode (WCAG 2.3.1), enforced by a frame-difference test in CI.

### 8.8 Frame-budget governor

- Frame times are measured continuously. If the p90 over a 5 s window exceeds 14 ms, the governor degrades in this order: DPR 2 → 1.5 → 1.25, then ambient particle budget halved, then parallax off, then background re-grade interval doubled.
- Bird rig fidelity and call timing are the last things ever degraded.
- The governor restores quality after 30 s of headroom.
- Only aggregate histograms reach RUM.

### 8.9 Visitor mode

- `/visit` uses the same boot and renderer against the visitor snapshot. There is no greeting, no presence, no listen-in, no offers, no settle, no notebook, and no account panel.
- The top bar contains only accessibility settings, stored in `localStorage` because a visitor has no account.
- Lighting follows the **host's** timezone, so the visitor sees what the host would see now. Weather, perches, moods, drift, the newcomer, and any active offer items render identically. There is no special rendering (no show-off mode).
- Audio plays identically, subject to the autoplay rules (§9.6). Captions and narration are available.
- The visitor bundle excludes interaction code, and the server rejects writes regardless.

---

## 9. Audio pipeline

### 9.1 Graph

```
per bird (≤7) + newcomer (≤1) + offer voice (1):
  AudioWorkletNode "voice" ─► StereoPanner (x position) ─► BiquadFilter (lowpass by zone depth)
       ─► Gain "listen-in" ─► chorus bus
ambient bed: noise-based wind/leaf/rain generators (AudioWorklet "ambience") ─► ambient bus
chorus bus ─► ConvolverNode (procedural IR, generated at boot) ─► wet/dry mix ─┐
ambient bus ──────────────────────────────────────────────────────────────────┤
                                            master: DynamicsCompressor (gentle) ─► Gain (user volume,
                                            settle duck, visibility fade) ─► destination
```

- The node count is fixed and bounded: 9 voices plus a fixed utility set. Calls are **not** new nodes. A call is a message to an existing voice (§9.2). This is how "no per-call allocation" is enforced.
- The reverb impulse response is generated procedurally at boot: ~0.9 s of exponentially decaying filtered noise giving a small open-air space. No audio files exist (INV-10).
- `latencyHint: 'playback'` on mobile saves power. Desktop uses `'balanced'`. Mixes change over seconds, so latency is irrelevant.

### 9.2 Synthesis voice (AudioWorklet)

- Each voice is a sinusoidal-modeling synthesizer, well suited to birdsong, which is dominated by near-pure tones with rapid frequency and amplitude modulation. It has:
  - 1–4 partials with per-partial amplitude
  - a frequency contour per syllable (start, peak, end, curve)
  - FM for trills (rate 15–60 Hz, depth)
  - AM tremolo
  - an optional band-passed noise component for harsh or "sharp" calls and for the nightjar's churr
  - ADSR envelopes per syllable
- **Scores.** The main thread sends a *score* (≤ 24 syllables, each ~10 numbers) via `port.postMessage`. The worklet copies it into a preallocated ring of syllable slots, so `process()` does no allocation. Where cross-origin isolation is enabled (we have no third-party content, so COOP/COEP is feasible), scores go through a `SharedArrayBuffer` ring instead of messages.
- **Safety.** Output is soft-clipped, with a per-voice loudness ceiling. A DC-block filter guards against NaN and runaway values, and on detection the voice resets and increments an error counter.

### 9.3 Call grammar and signature recognizability

- **Grammar per species** (6 files): a motif library (syllable templates with parameter ranges) and phrase rules (motif sequences, repetition, pauses). Starting sketches:
  - wren-like: loud rapid trill bursts with a terminal buzz
  - warbler-like: thin high "seet" series, rising
  - finch-like: twittering chatter with nasal notes
  - thrush-like: fluting phrases with pauses between them
  - dove-like: low soft coos in 3–5 note patterns
  - nightjar-like: long, even churr with occasional wing-clap clicks (noise transients)
- **Per-bird signature** from `voice_seed`, fixed for life:
  - base register offset within the species range (±3 semitones)
  - one **signature motif**: a distinctive syllable pattern that opens or recurs in most of the bird's phrases
  - a rhythm ratio between syllables
  - a timbre fingerprint (partial balance, FM depth tendency)
- **Variation on every call.** A fresh per-call seed perturbs pitch (±40 cents), timing (±8%), and syllable count (±1). Motif ordering varies within grammar rules. The same call never plays exactly twice.
- **What mood and drift change:** tempo (±15%), loudness, phrase length, register spread (±2 semitones), and call density. Specifically:
  - wary: short and sharp
  - content: full phrases
  - drowsy: slow and soft
  - alert: crisp and repeated
  - Vocal-frequency drift raises call density and chorus participation, **never** the signature motif or timbre fingerprint. This is what keeps Pip recognizable across moods and weeks.
- **Recognizability targets** (verified in §9.9): at 7 birds with 2 same-species birds, trained listeners identify a target bird from a 3-call excerpt ≥ 80% of the time across moods. An automated proxy supports this: the MFCC and pitch-contour embedding distance between birds exceeds within-bird variation by a margin (P-60). Same-species pairs get a *signature separation* constraint when seeds are drawn at adoption, and seeds are re-drawn until it passes.

### 9.4 Scheduling and chorus

- **Idle calls:** a per-bird Poisson process on the client. The rate is set by `call_rate` level × circadian curve × weather damp × settle duck. There is a refractory period of ≥ 2 s per bird and a soft density cap of about 3 simultaneous calls. At night, diurnal birds are nearly silent and the nightjar is active.
- **Social calls** (call-and-response, chorus, alarm) come from the snapshot timeline at canonical times, realized with per-device variation. When the client's own idle call happens, a warm neighbor may add a local reply (presentation only).
- **Beak sync.** The scheduler looks ahead 150 ms, sends the score to the voice, and passes syllable onsets to the renderer so beak and throat movement match the sound within ±30 ms.
- **Listen-in effect on scheduling:** none. Listen-in changes the mix, not the bird's behavior.

### 9.5 Listen-in mix

- **Engage:** the focused bird's gain rises to +6 dB over 3.0 s. Every other bird's gain falls to −10 dB over the same 3.0 s. Both follow equal-power curves via `setValueCurveAtTime`, using precomputed, reused `Float32Array` curves.
- **Floor:** non-focused birds never go below −14 dB relative to the ambient mix. They never go silent (PRD).
- **Disengage:** everyone returns to the ambient mix over 3.5 s.
- **Transfer:** the old bird ramps down while the new bird ramps up, both over 3.0 s.
- **Panning is untouched.** The focused bird stays where it is in space, because this is listening, not switching channels.
- **Captions** during listen-in prioritize the focused bird (§10.2).

### 9.6 Autoplay policy and startup

Browsers block audible AudioContexts until a user activation, unless site engagement allows autoplay. The PRD's "calls already audible" is therefore achievable only when the browser permits it. The plan maximizes the permitted cases and degrades without announcing (D-13).

1. Create the AudioContext at hydration and call `resume()`. If the state becomes `running` (common for engaged return visitors in Chromium), fade the mix in over 400 ms, so it arrives as ambient sound already in progress.
2. If suspended, the aviary runs visually. A one-shot capture listener on the first `pointerdown` or `keydown` anywhere calls `resume()` synchronously (required by Safari). The ambient mix then fades in over ~2.5 s, like stepping outside. It is not a sudden start.
3. There is no "tap to enable sound" overlay or prompt. The accessibility settings panel offers a "Sound" control for users looking for one. If captions are enabled, they show during the silent period. Narration mentions calls either way.
4. Aggregate RUM counts the share of loads that start audible and the share that are blocked. That informs future UX work but carries no user identity.

### 9.7 Fallback (WebAudio unavailable)

- **Triggers:** no `AudioContext`, no `audioWorklet`, context creation throws, the worklet module fails to load, or `resume()` rejects after a user activation.
- **Behavior:** graceful silence, with **captions on by default** unless the user explicitly turned captions off in settings. Beak and throat animation continues.
- There is no recorded-audio path (INV-10). An aggregate error counter increments.

### 9.8 Caption generation

Captions are produced from the same score object the voice played, so they match what was actually heard:

- note count (one, two, three, "a run of")
- contour (rise, fall, rise-and-fall, level)
- rhythm (quick, paused, spaced)
- timbre (trill, whistle, coo, churr, sharp call, chatter)
- loudness (soft, clear, sharp)
- location (from the back perch, on the high branch)

The grammar composes these into phrases like "a soft three-note rise" or "a low trill, paused, low trill again", following the naturalist rules (§11). Chorus windows collapse into one caption, e.g. "pip and wren call back and forth."

### 9.9 Audio QA

- **Offline rendering tests:** `OfflineAudioContext` in headless Chromium renders 10,000 calls per species. They must have no NaN or clipping, and loudness must stay within −30 to −18 LUFS short-term at the master.
- **Recognizability proxy** as a CI gate (P-60).
- **Human listening panels** at M2 and M3: identification tests at 2, 5, and 7 birds; annoyance and "canned-ness" ratings after 30 minutes of listening; and a fatigue test with the aviary as background for 2 hours.
- **Sound designer sign-off** on every species grammar before inclusion.
- **Cross-browser worklet tests** on Safari, Firefox, Chrome, and Edge (latest two).
---

## 10. Accessibility surfaces

Accessibility ships in v1 and is a release blocker. The bar is the PRD's: every user gets the *actual product* in their register, not a stripped fallback.

### 10.1 Screen-reader narration

- **DOM:** the aviary region is `<section aria-label="aviary">`. It contains:
  - one visually hidden `<div aria-live="polite" aria-atomic="true">` (the narration region)
  - the bird focus targets (§10.4)
  - an `aria-hidden="true"` canvas
- **Generation (client-side).** `aviary-core`'s prose grammar reads the same rendered state the canvas uses: timeline segment, activity, mood-keyed posture, zone, time of day, weather, the most recent calls, offers, and the newcomer. It never reads trait values; the client doesn't have them.
- **Idle cadence:** one update every 30–60 s (randomized), 1–2 sentences, rotating focus across birds and ambient details. An update is skipped rather than repeated if nothing new is true, and near-duplicates of the last 5 updates are suppressed.
- **Priority events** (return-greeting, offer reaction, settle, re-engage, newcomer welcome) go to the front of the queue and are delivered within ~1.5 s. They reset the idle timer. There is never more than one update per 5 s. `assertive` is never used.
- **Voice:** naturalist, lowercase, present tense, specific. For example: "pip looks up from preening and gives a quiet two-note call." Birds are referred to by name, with an occasional descriptive mention ("wren, the small brown one, is fluffed on the back perch"). Settle: "the light softens toward evening. the aviary quiets." There is no "you".
- **Visible narration** (optional, "Show narration text"): the same text shown in a slim band beneath the top bar with AA contrast, fading between updates.
- **Tests:**
  - scripted NVDA+Firefox, JAWS+Chrome, VoiceOver+Safari (macOS, iOS), TalkBack+Chrome
  - queue cadence test (no more than 2 idle updates per minute)
  - grammar lint (§11)

### 10.2 Captions

- Captions are opt-in (default off), and default on when WebAudio is unavailable (§9.7).
- **Placement:** near the calling bird, auto-placed to avoid overlapping birds and the top bar and clamped inside the viewport. They fade in with the call onset and out ~1.5 s after it ends. At most 2 visible at once. During listen-in the focused bird's captions take priority and chorus captions merge.
- **Legibility:** text on a soft translucent backing sized to the text, so AA contrast holds against any time-of-day background. Caption DOM nodes come from a fixed pool of 3 and are `aria-hidden`, because narration serves screen readers and double announcement is avoided.
- **Reduced motion:** captions keep opacity fades, which is compatible with reduced motion.

### 10.3 Reduced motion

Specified in §8.7. The acceptance review includes vestibular-sensitive participants in the usability study (§10.7).

### 10.4 Keyboard and focus

- **Tab order:** the five top-bar controls, then the aviary (one tab stop via roving tabindex), then open panels when present.
- **Bird focus targets:** transparent `<button>` elements positioned over each bird and moved with it every frame via a transform, not layout. Each has:
  - accessible name = bird's name
  - `aria-describedby` = a short naturalist description refreshed at most every 30 s (e.g. "preening on the front perch")
  - `aria-pressed` = listen-in state
- **Entering the scene with Tab** focuses the first bird in left-to-right order. Focus stays with the *bird identity* as it moves.
- **Keys:**
  - ←/→ (and ↑/↓): move to the spatially adjacent bird
  - Home/End: first or last bird
  - Enter/Space: listen-in
  - Escape: exit listen-in, or undo within the settle window
  - `o`: offer tray (disableable; D-25)
- **Focus ring:** a double ring (light inner 2 px, dark outer 2 px) drawn as a DOM outline that follows the bird's ellipse. It meets 3:1 non-text contrast against bright, dim, and settled scenes (tested). It shows only on `:focus-visible`, so there is no chrome for mouse users.
- **Panels:** focus trap, Escape to close, focus returns to the invoking control. The top bar never fades while it contains focus.
- **Hit targets:** at least 44 × 44 CSS px on touch, which exceeds the WCAG 2.2 AA minimum of 24 px.

### 10.5 Contrast

- **Scope:** all user copy (top-bar accessible labels when visible, panels, settings, errors, captions, visible narration) meets WCAG AA (4.5:1, or 3:1 for large text). Icons and focus indicators meet 3:1 (WCAG 1.4.11).
- **Automated gate:** Playwright renders the top bar, captions, visible narration, and the focus ring against scene states (dawn, noon, dusk, night, settled, rain) at 3 viewports. It samples actual pixels behind text bounding boxes and computes worst-case contrast. Any failure blocks the merge.

### 10.6 Accessibility settings (system voice)

- Captions: On/Off
- Reduced motion: Use system setting / On / Off
- Show narration text: On/Off
- Keep top bar visible: On/Off
- Sound: On/Off, plus Volume
- Keyboard shortcuts: On/Off

Settings sync per account, except that "Use system setting" for reduced motion resolves per device (D-21). Visitors store them locally.

### 10.7 Audit and research plan

- **Every PR:** axe-core in CI, the keyboard-only E2E suite, and the contrast gate.
- **M3:** external WCAG 2.2 AA audit, with remediation before the beta.
- **M3–M4:** paid usability sessions with screen-reader users (≥ 5), reduced-motion and vestibular users (≥ 5), and Deaf and hard-of-hearing users (≥ 5). The success criterion is qualitative parity of "does it feel alive", not task completion alone.
- The accessibility lead has veto on release (§16.4).

---

## 11. Voice and content system

- **Two catalogs:** `copy/naturalist/*` and `copy/system/*`. Every component declares its register, and the surface inventory (§2.3) maps surfaces to registers.
- **Naturalist lint** (catalog strings and 100k sampled grammar outputs per build):
  - lowercase except proper UI labels
  - present tense in narration and notebook (heuristic tagger plus curated exceptions)
  - no `!`, no second person (`you`, `your`)
  - no digits (numbers spelled as observation: "three-note", "several minutes")
  - no trait names (boldness, warmth, vocal frequency, saturation, curiosity, level, stat)
- **System lint:** sentence case, direct, no naturalist verbs used as jargon in error text.
- **Global banned list** (both registers, plus emails):
  - welcome back, great to see you, you've been, while you were away, days in a row, streak
  - achievement, unlocked, badge, level up, points, score, XP, reward, congrat, milestone
  - miss(ed) you, hungry, sad, lonely, sick
- **Prose grammar engine:** a typed, weighted generative grammar (Tracery-like) with slots bound to state (bird, zone, activity, light, weather, call description). Templates carry salience and context constraints. Per aviary, recent-template memory prevents reuse within N renders. Output is deterministic given a seed, for testability and for the notebook's stored render seeds.
- **No LLM generation in v1** (D-28). Sending per-bird state to a third-party model would breach INV-09. A self-hosted model would add nondeterminism, voice risk, and cost that the product doesn't need. The writer-authored grammar is the voice.
- **Authoring volume** (to avoid repetition across months): notebook ≥ 400 templates across ≥ 25 candidate kinds; narration ≥ 600 fragments; captions ≥ 150 descriptors; greeting narration ≥ 80. Content grows each release.
- **Review gate:** any change to either catalog or a grammar needs content-designer approval (CODEOWNERS). Each release gets a sample review of 300 random notebook and narration renders.

---

## 12. Privacy and security engineering

### 12.1 Data planes and isolation (INV-08, INV-09)

- Three planes: identity DB, simulation DB, telemetry. Each has separate credentials and separate network segments. Neither DB has a replication or ETL connector to any analytics store.
- A CI guard rejects any IaC or connector config that references the simulation DB from telemetry projects.
- Service roles hold least-privilege grants. The analytics warehouse, if one ever exists for business operations, sees aggregate operational metrics only.

### 12.2 PII handling

- Email is encrypted with a per-account DEK wrapped by KMS. Lookup uses the blind index. Only the mailer decrypts to send, and the host's visits panel decrypts only for display to the host.
- An `Email` type redacts itself in `toString`, JSON, and log serializers. A log-scanning test injects canary emails through every flow and fails if any appears in logs or traces.
- UUIDs never derive from email or any user data (v7 or v4 only).

### 12.3 Telemetry boundary

- Metric labels come from an allowlist (endpoint, status class, region, browser family and major version, device class, build). `account_id`, `aviary_id`, `bird_id`, `session_id`, and email are rejected at the SDK wrapper and by a CI schema check.
- RUM beacons carry a random per-page-view ID only. Nothing links them to an account.
- Logs may include `account_id` for operational debugging ("is this account having errors" is allowed) with 14-day retention. They never include event payloads, trait or mood values, notebook text, bird names, or email.

### 12.4 Third parties

- The only processors are the cloud host, the CDN/edge (under DPA), and the transactional email provider (receives recipient address and system-voice content only).
- There are no analytics SDKs, session replay, ad pixels, fonts from third-party CDNs, or external scripts. This is enforced by a strict CSP (`script-src 'self'` plus hashes) and a dependency allowlist.

### 12.5 Security controls

- Magic-link hardening (§5.7): fragment tokens, POST consumption, single use, 15-minute expiry, rate limits.
- Cookies use the `__Host-` prefix. CSRF protection per §5.1.
- Content security: Trusted Types and SRI on static assets.
- Visitor tokens are read-only and bound to one browser, and revocation is effective on the next poll.
- Session revocation is immediate at the API and ≤ 60 s at the edge.
- Invitation abuse controls (§5.6).
- **Account enumeration resistance:** identical responses and timing-equalized handlers.
- **Pen test** before the beta (M3). Threat model reviewed at each milestone.

### 12.6 Support access

- Per-account support debugging (e.g. "my bird feels reset") requires a **user-initiated consent grant** from account settings. The grant creates a time-boxed (72 h) support token.
- Every access is audited, and the user can see that the grant exists.
- Support tooling shows canonical state in staff-only tools. It never feeds any aggregate view, and no bulk-export capability exists.

### 12.7 Privacy policy

- Linked from account settings, plain text.
- Names the aggregate telemetry categories (§13.5), explicitly excludes per-bird interaction state, and states the retention and backup expiry (§4.5) and the processor list.
---

## 13. Performance budgets and observability

### 13.1 Budgets and exact measurement definitions

| Budget | PRD limit | Internal target | Measured how |
|---|---|---|---|
| Initial JS at first paint | ≤ 2 MB gz | `boot.js` ≤ 45 KB gz; everything before first bird ≤ 60 KB gz; `aviary.js` ≤ 250 KB gz; all JS ever loaded ≤ 900 KB gz | CI bundle analyzer. Hard fail at the PRD cap; fail at internal targets unless a budget-owner override is recorded. |
| Time to first bird visible | < 500 ms, mid-tier mobile over 4G | p75 < 500 ms on the reference profile; p75 < 1.2 s cold first visit (tracked, not gated) | `performance.mark('first-bird')` after the first frame containing a bird is presented, relative to navigation start. Synthetic plus RUM. |
| Idle motion | 60 fps on a 5-year-old laptop | p95 frame interval ≤ 17.5 ms and < 0.5% frames > 33 ms over 30 min | rAF interval histograms in CI device-lab runs and RUM. |
| Memory over 30 min | no growth | JS heap slope ≤ 25 KB/min after warmup; DOM nodes, AudioNodes, and listeners constant | CI soak (§13.4). |
| Snapshot payload | "kilobytes" | ≤ 6 KB gz at 7 birds | Compile-time assertion in the tick. |
| Tick latency | p99 alarm > 5 s | p99 < 1.5 s | Scheduled time → snapshot published. |

**Reference profile (D-41).** Mid-tier Android device, 2023-class. 4G conditions: 9 Mbps down, 1.5 Mbps up, 70 ms RTT. A *return visit*: warm DNS, HTTP/3 with TLS resumption, and static assets in the service worker or HTTP cache. That is the moment the "already running" illusion matters most.

A cold first-ever visit needs at least 2 extra round trips of connection setup. It is also preceded by sign-in and adoption, so its first aviary view is the empty-aviary quiet field by design. It is tracked separately, and budget reports never blend the two.

### 13.2 How the 500 ms is achieved

| Step | Budget |
|---|---|
| Navigation → HTML first byte at the edge (HTTP/3 resumption, edge-token check, KV read ≤ 10 ms p95, early flush of the head) | ≤ 150 ms |
| HTML parse; `boot.js` from cache (modulepreload or Early Hints); snapshot parse | ≤ 90 ms |
| Projection, layout, cached-sprite synthesis for ≤ 8 birds (precomputed at small resolution first, refined in idle), first draw | ≤ 120 ms |
| Frame presentation | ≤ 50 ms |
| **Total** | **~410 ms**, leaving ~90 ms of headroom |

Nothing on the critical path waits on audio, fonts (system font stack; no web fonts in the scene path), panels, or `aviary.js`.

### 13.3 Runtime budget, per frame at 60 fps, 5-year-old laptop

| Work | Budget |
|---|---|
| Timeline evaluation and behavior updates | ≤ 1.5 ms |
| Bird drawing (≤ 8 rigs from cached sprite parts plus path strokes) | ≤ 4 ms |
| Particles | ≤ 1 ms |
| Composition of cached layers | ≤ 2 ms |
| DOM overlay transforms | ≤ 0.5 ms |
| **Total main-thread work** | **≤ 9 ms**, leaving headroom for GC and the browser |

Audio synthesis runs on the audio rendering thread (worklet). The budget is ≤ 25% of one core for 9 voices at 48 kHz, verified on the reference laptop. A laptop reference device lab covers 2021 Intel i5 with integrated graphics on Windows/Chrome and Edge, 2020 MacBook Air (Intel) on Safari and Firefox, and a 2021 Chromebook-class device.

### 13.4 Memory discipline and the 30-minute CI test

- **Rules:**
  - preallocated typed arrays for motion
  - fixed voice nodes (no per-call nodes)
  - pooled caption and notebook DOM nodes
  - bounded event queue
  - timeline segments pruned as they pass
  - sprite atlases regenerated only on chroma change, with the old atlas explicitly released (`canvas.width = 0`)
  - every `addEventListener` paired with an `AbortController` signal per component lifecycle
- **CI soak:**
  - Nightly: 30-minute real-time runs in Chromium via CDP. The script opens the aviary, cycles listen-in every 2 min, makes offers every 5 min, opens and scrolls the notebook through 200 fixture entries, toggles reduced motion, settles and re-engages, and runs a synthetic weather episode.
  - Samples: forced-GC heap every 60 s, DOM node count, JS event listener count, and AudioNode count (via an instrumented factory).
  - Assertion: regression slope and endpoint limits.
  - Firefox and WebKit: nightly 30-minute runs with process RSS sampling and a coarser threshold.
  - Per PR: a 5-minute variant.

### 13.5 What we measure

- **Server (metrics, no per-account labels):**
  - request rate, latency, and errors per endpoint
  - tick latency, compute time, catch-up backlog, CAS conflicts, and invariant-violation count
  - event ingest rate, lag, and rejects by code
  - snapshot publish lag and edge KV propagation
  - offer resolution latency
  - mailer send, bounce, and complaint rates by template
  - magic-link issued/consumed/expired counts (aggregate)
  - DB, Redis, and queue health
  - notebook writer run duration
  - nightly integrity job result
- **Client RUM (aggregate, ~20% sampled, per-page-view random ID only):**
  - navigation timings and first-bird time
  - frame-interval histograms and governor degradations
  - audio start state (running, blocked, unavailable) and audio error counts
  - snapshot fetch latency and failure counts
  - event POST failures
  - JS error type counts (scrubbed stack frames, no messages containing user data)
  - session-duration histogram: a bucketed duration sent once at `pagehide` with no identifier, which is the PRD's anonymized histogram
  - browser family/major and device class
- **Synthetic monitoring:** headless Chromium, Firefox, and WebKit from 6 regions every 5 minutes against dedicated synthetic accounts. They measure first-bird, frame timing over 60 s, audio startup (with autoplay flags), snapshot freshness (`tick_index` age), and the sign-in flow (with test mail sinks).

### 13.6 What we deliberately do not measure

The engineering boundary itself is the privacy promise, so these are not "not yet" items. They are "never":

- per-account or cohort engagement: DAU/WAU/retention by account, visit frequency, session counts per account, streak-like metrics (D-29)
- any distribution of personality, drift, mood, attunement, greeting order, offers, or listen-ins across accounts, including "average drift across all accounts"
- funnels, click maps, heatmaps, session replay, A/B tests on engagement
- accessibility-setting adoption rates (disability-adjacent data; we learn from research sessions instead)
- notebook open rates, or which notebook entries were read
- visitor behavior beyond the host-visible visit log

### 13.7 Alarms and SLOs

| Signal | Alarm |
|---|---|
| Tick latency p99 | > 5 s for 5 min (page) |
| Tick backlog | > 2 ticks behind for any shard (page) |
| Snapshot publish lag p95 | > 10 s (ticket) |
| Integrity violations | > 0 (page) |
| API 5xx | > 0.5% for 10 min (page) |
| Magic-link consumption success | drops > 20% day-over-day (deliverability signal) |
| RUM first-bird p75 | regresses > 15% week-over-week (ticket) |
| Audio-unavailable share | exceeds baseline by 2× after a browser release (ticket) |

SLOs: snapshot availability 99.9%; event ingest 99.9%; tick freshness (snapshot ≤ 2 ticks old) 99.9%.

---

## 14. Testing strategy (cross-cutting)

| Layer | What | Gate |
|---|---|---|
| `aviary-core` unit + property tests | Drift monotonicity; daily-cap bound; reservoir conservation; absence invariants (no wary increase, no trait decrease); determinism (same seed ⇒ same output across Node and 3 browser engines for all canonical decisions); batch-advance equivalence; presence-union math; greeting plan (≥ 1 greeter, boldness ordering tendency); offer resolution (cooldown ⇒ glance and no drift) | PR |
| Calibration harness (§6.14) | Persona bands, mood occupancy, notebook cadence | PR (short run), nightly (full 120-day) |
| API contract tests | Schemas; no trait fields anywhere; visitor tokens rejected on all writes; idempotency | PR |
| DB tests | Monotonic trigger; grants (only `sim_tick` updates personality; telemetry roles can't connect); CAS ticks under concurrency | PR |
| Chaos / failure | Kill workers mid-batch; zombie commits; DB failover; Redis loss (snapshot recompiled from DB); edge KV lag; 6 h outage catch-up | Weekly in staging |
| Browser E2E (Playwright across Chromium, Firefox, WebKit, and a real-Safari device cloud for presence and audio) | Sign-in (incl. scanner-prefetch simulation), adoption, first frame mid-action (visual diff with a deterministic seed hook in test builds only), greeting within 2 s, listen-in ramps (audio graph param inspection), offers, settle + undo, notebook scroll, visits and revocation, deletion and recovery | PR (Chromium), nightly (all) |
| Presence tests | Every combination of visible/hidden × focused/unfocused × active/idle, plus touch-only, on real OS windows | Nightly |
| Anti-announcement DOM audit | After return with each absence class, no text nodes appear outside the top bar for 10 s; no `role=alert` or `status` except the error line and narration; no forbidden components in the bundle | PR |
| Voice lint | Catalogs + 100k grammar samples | PR |
| Accessibility | axe, keyboard suite, contrast gate, SR scripts | PR / nightly / milestone audits |
| Performance | Bundle budgets (PR); first-bird on the reference profile in the device lab (nightly); frame budget on reference laptops (nightly); 30-minute memory soak (nightly) | as listed |
| Audio | Offline render safety, loudness, recognizability proxy; human panels at milestones | PR / milestone |
| Privacy | Canary-email log scan; telemetry label allowlist; CSP and dependency allowlist; IaC data-plane guard | PR |
| Load | 10× forecast ticks, snapshot reads, event ingest | Pre-beta, pre-GA |
| Security | Pen test; magic-link and visit-token abuse cases | M3 |
---

## 15. Delivery plan

### 15.1 Team shape (about 14 people)

| Workstream | People | Owns |
|---|---|---|
| Engine | 2 engineers | `aviary-core` sim modules, tick service, drift/mood/perch/social/weather, greeting planner, adoption, notebook candidates and writer, calibration harness |
| Client scene | 2 engineers + 1 technical artist | Boot and projection, renderer, rig, species visuals, key poses, layout solver, reconciliation, governor |
| Audio | 1 audio engineer + 1 sound designer (contract OK) | Worklet synth, grammars, signatures, mixer, captions generator, listening panels |
| Platform | 2 engineers | API, auth, sessions, edge worker, snapshot pipeline, events, offers, visits, export/deletion, infra, observability, privacy plumbing |
| Product UI | 1 engineer | Top bar, panels, settings, adoption sheets, notebook panel, visitor shell |
| Accessibility | 1 lead | Narration integration, keyboard model, audits, research sessions; release veto |
| Content | 1 content designer / writer | Both copy catalogs, prose grammar, banned list, sample reviews |
| Design | 1 product/visual designer | Design system, palette, top bar, panels, reduced-motion aesthetics |
| Quality | 1 SDET | E2E, device lab, soak, presence tests, performance harness |

### 15.2 Milestones

Calendar weeks assume the team starts together.

**M0 — Foundations (weeks 0–2)**
- Deliverables: monorepo, CI with bundle-budget and lint scaffolding, IaC for three planes, `aviary-core` skeleton with PRNG and fixed-point, snapshot schema v0, identity schema and magic-link spike, design tokens v0, voice guide v1, species shortlist.
- Exit criteria: CI enforces the INV lint rules (banned components/copy, server-only import boundary, metric allowlist).

**M1 — Vertical slice (weeks 2–8)**
- Deliverables: tick service with one species and CAS batches; snapshot compile and edge inline; boot renderer painting birds mid-action; presence tracker plus event ingest; drift v1 plus the harness running the regular and background-tab personas; one-species voice worklet with idle calls; staging simulated clock.
- Exit criteria: a staff member signs in, sees two birds already in motion under 500 ms on the reference profile, hears procedural calls, and presence produces measurable harness drift.

**M2 — Feature complete (weeks 8–15)**
- Deliverables: 6 species (rigs and grammars); full mood, social, weather, and day/night; greeting planner and realization; listen-in; offers with server resolution; settle with undo; notebook end to end; captions; narration v1; reduced-motion v1; accounts complete (sessions, email change, export, deletion); starter adoption.
- Exit criteria: all §14 PR gates green; listening panel #1; harness bands met for all personas.

**M3 — Hardening (weeks 15–19)**
- Deliverables: visits (invite, bind, revoke, log, opt-in email); newcomer mechanic (tested via the staging clock at 7 birds); governor; 30-minute soak green on 3 engines; external accessibility audit and fixes; pen test; privacy review with data-flow diagram sign-off; load test at 10×; DR drill (restore plus integrity).
- Exit criteria: go/no-go gates for alpha (§16.4) satisfied.

**M4 — Alpha / dogfood (weeks 19–25, overlapping M3)**
- Deliverables: staging dogfood cohort running ≥ 6 weeks (it starts in week 12 on the M2 build, so real-time 3-week drift has been felt before beta); research sessions with disabled participants; listening panel #2; content expansion.
- Exit criteria: diary evidence that drift is "noticed on looking back" and not session-to-session; no accessibility blockers.

**M5 — Private beta (weeks 25–31)**
- Deliverables: production, invite-list access, account cap 2k → 10k.
- Exit criteria: SLOs met for 3 weeks; RUM budgets met; zero integrity violations.

**GA (week ~32)**
- Deliverables: open sign-up, capacity at 10× forecast.

### 15.3 Critical path and dependencies

- The snapshot schema (platform + engine) gates the boot renderer. Freeze v1 at week 6, after which changes are additive only.
- The **dogfood clock** is the true critical path. Visible drift is a three-week, real-time phenomenon, so the staging cohort must start by week 12, or the beta slips.
- Species grammars (audio) and species rigs (client/art) must be co-designed per species. Species are delivered in pairs, and the starter-eligible pairs come first.
- The prose grammar volume (content) gates narration quality and notebook non-repetition. The writer starts in M0.

### 15.4 Cut lines if the schedule slips

**May slip to v1.1 without violating the PRD:**
- convolution reverb (use a dry mix)
- SharedArrayBuffer transport (messages suffice)
- governor refinements beyond DPR scaling
- 6th species (ship 5, still "about six"; the nocturnal species is the one kept, because the PRD names it)
- more song fragments beyond 4
- visible-narration band (screen-reader narration is not cuttable)
- tz-traveler smoothing polish

**Never cut:**
- server tick
- presence precision
- monotonic drift and its protections
- procedural calls
- the no-entry-animation first frame
- greeting realization
- narration, captions, reduced motion, keyboard
- privacy planes
- the announcement and gamification guardrails
- the performance CI gates

---

## 16. Rollout

### 16.1 Stages

| Stage | Who | Environment | Purpose |
|---|---|---|---|
| Internal alpha | Engineering team | staging | Functional validation, harness vs felt behavior |
| Dogfood cohort | 30–60 consenting staff and friends | staging (research environment) | Real-time drift calibration, notebook sparsity, audio fatigue, accessibility diaries |
| Private beta | Waitlist invitees, capped | production | Real networks, devices, and deliverability; SLO burn-in |
| GA | Open sign-up | production | — |

Production accounts created in beta are real and permanent. No beta reset ever happens, because it would violate identity and persistence. Engine changes after beta start must respect INV-02 and INV-07 exactly as in GA.

### 16.2 Ramping birds per aviary

- Per-aviary growth is dictated by aviary age (§6.11): no production aviary can reach bird 3 before about day 65. Launch therefore serves only two-bird aviaries for months, which gives the team a natural window to validate larger aviaries against real conditions.
- A global **adoption ceiling** flag (P-45) starts at **3** at beta. It rises one step at a time (4 → 5 → 6 → 7) only after that bird count passes, on the staging clock with aged aviaries:
  - the frame budget on reference laptops and phones with N birds plus newcomer
  - the recognizability listening panel and proxy at N birds
  - narration cadence and verbosity review at N birds
  - layout solver coverage at N on the narrowest viewport
  - snapshot size ≤ 6 KB
- If the ceiling is below an aviary's next newcomer, the newcomer window is **deferred**. It is not skipped, and nothing is shown. Nothing is ever taken away.
- The engine hard cap of 7 is a constant (not a flag) and a DB check constraint on bird count per aviary.

### 16.3 Instrumented from day one

- Everything in §13.5 is live before the first beta account. Nothing in §13.6 is ever instrumented.
- Nightly integrity job and invariant-violation counters.
- Mail deliverability dashboards (SPF, DKIM, DMARC alignment; dedicated sending subdomain; warm-up plan for the sending IP or pool before beta).
- RUM audio-start-state shares, to learn how often autoplay allows audible starts.
- Edge KV propagation and snapshot freshness.

### 16.4 Launch gates (go/no-go for beta and GA)

1. Harness: all persona bands pass on the release's `engine_version`. Dogfood diaries show no "birds changed during a session" reports and confirm "looking back" noticing by weeks 3–4.
2. Performance: reference-profile first-bird p75 < 500 ms (synthetic and beta RUM); frame budget met; the 30-minute soak green on 3 engines for 7 consecutive nights.
3. Accessibility: external audit closed with no AA failures; research-session parity findings addressed; accessibility lead sign-off.
4. Privacy: data-flow diagram audit, telemetry allowlist review, processor DPAs, privacy policy published, deletion and export verified end to end.
5. Security: pen-test criticals and highs fixed; magic-link scanner test passes.
6. Integrity: DR drill passed (restore, checkpoint plus ledger repair, zero vector loss); zero integrity violations in the last 14 days.
7. Content: sample review of 300 renders signed off; banned-lint clean.
8. Anti-announcement audit: DOM audit green, and a design review of every surface against §2.3.

### 16.5 Operations: parameter changes, kill switches, runbooks

- **Engine parameter changes** (drift gains, τ, caps, W, cooldowns, notebook budget) require:
  - a harness run on the new values with a diff of trajectories
  - design and engine sign-off
  - a staged rollout by aviary-hash cohort (1% → 10% → 100%) with only operational metrics watched (there are no per-bird metrics to watch, by design)
  - a recorded `engine_version`

  Increases to drift speed are discouraged after launch, and decreases can't reverse past drift. That is why launch values are conservative (§6.4).
- **Kill switches:**
  - offers (tray hides seed/pool/fragment items and shows nothing in their place)
  - visits (new invitations disabled; existing visits keep working unless there is a security incident)
  - notebook writer (pauses; the budget carries over)
  - newcomer scheduler
  - tick cadence degrade to 120 s (`step()` handles any Δ)
  - edge inline snapshot off (falls back to the client fetch path)
- **Runbooks:** tick backlog, integrity violation (freeze parameter changes, snapshot the DB, investigate, then run the checkpoint+ledger repair tool with two-person approval), magic-link deliverability incident, edge KV outage, audio regression after a browser release, and session-token compromise (mass revocation).
---

## 17. Risks

Likelihood (L) and impact (I) are rated H/M/L. Each risk names the earliest signal we can observe *without* violating the telemetry boundary.

| ID | Risk | L / I | Early signal | Mitigation |
|---|---|---|---|---|
| R-01 | **Drift calibrated too fast.** Birds change visibly between sessions, the product reads as a Tamagotchi, and because drift is monotonic and persistent, over-drift can't be rolled back. | M / H | Harness bands; dogfood diaries ("she's different since yesterday") | Launch at the slow edge of the band; daily release cap far below perceptibility; reservoir τ spreads effects; changes only via §16.5. The asymmetry is deliberate: slow is fixable, fast is not. |
| R-02 | **Drift too slow.** Nothing the user does seems to matter, and the product reads as a screensaver. | M / H | Dogfood diaries at weeks 3–5 ("nothing changes") | Harness proves day-21 visibility; the perceptibility mapping (plumage chroma steps, front-affinity shifts) is designed so a 0.06–0.1 change is actually visible; gains can be raised going forward safely. |
| R-03 | **Can't tune with production data.** The privacy boundary forbids population drift analytics, so a mis-calibration may persist unseen. | H / M | — (by design) | Pre-launch harness plus a ≥ 6-week dogfood; qualitative channels (support contact, opt-in research interviews); the consent-gated per-account support view for individual reports. This constraint is accepted and not worked around. |
| R-04 | **Presence inflation or deflation.** Jigglers, kiosks, and multi-device double counting inflate; touch devices without pointermove deflate. | M / M | Harness personas; device-lab presence tests | Three-signal conjunction; `isTrusted`; `pointerdown` counted for touch (D-04); cross-device union; daily concavity. |
| R-05 | **Personality vector loss or corruption** through bad migrations, a bug, or operator error. This is the worst failure and is invisible to users until felt. | L / Critical | Nightly integrity job; CAS conflict counters | §4.6's seven-layer defense; DR drills; two-person migrations; monotonic trigger. |
| R-06 | **Sync or tick incorrectness:** double application, lost events, ordering bugs, tz edge cases (DST, travel), catch-up errors after outages. | M / H | CAS conflicts, backlog, property tests | Pure `step()`; CAS; idempotent events; batch-equivalence tests; DST and travel personas; catch-up chaos tests. |
| R-07 | **Audio uncanniness.** Synthesized calls sound beepy, electronic, grating, or repetitive; chorus mush at high bird counts; fatigue over long sessions. | H / H | Listening panels; dogfood fatigue diaries | Sinusoidal modeling fits birdsong; a dedicated sound designer; per-call variation; loudness ceilings; density caps; recognizability gates per bird count; adoption ceiling ramp (§16.2); night quietness; gentle master compression. |
| R-08 | **Autoplay restrictions** break "calls already audible" for many loads. | H / M | RUM audio-start-state share | Resume on the first activation with a slow fade-in; no nagging prompt (D-13); captions available; engaged returning users often get audible starts. |
| R-09 | **Accessibility regressions.** Narration too chatty or stale; the reduced-motion register looks broken; the canvas is hostile to assistive tech; the focus ring disappears at night. | M / H | CI gates; research sessions | A11y lead with veto; DOM overlay targets; cadence tests; the contrast gate across all lighting states; milestone audits; disabled participants in research. |
| R-10 | **Performance:** the 500 ms budget fails in real networks, or on cold edge KV, or high-DPI old laptops miss 60 fps, or long sessions leak. | M / H | Synthetic and RUM dashboards; device lab | Edge-inline snapshot; tiny boot; SW caching; governor; soak tests; the precise measurement definition (D-41) reported honestly alongside cold numbers. |
| R-11 | **Announcement and gamification creep** as well-meaning future contributions ("just a small toast"). | H / H (long-term) | PR review checklist hits | No such components in the design system; banned copy lint; DOM audit; CODEOWNERS on copy and top bar; this plan's invariants in the repo's contributor guide. |
| R-12 | **Privacy leakage** through logs, error reports, support tooling, or future analytics asks. | M / H | Canary scans; allowlist violations | Data-plane isolation; the `Email` type; label allowlist; no third-party SDKs; consent-gated support access; 14-day logs. |
| R-13 | **Magic-link deliverability and link scanners.** Users can't sign in, or scanners consume tokens. | M / H | Consumption-rate alarm | Reputable provider; SPF/DKIM/DMARC; a warmed dedicated subdomain; fragment tokens plus POST consumption (D-36); a quick re-request path in the PRD's matter-of-fact copy. |
| R-14 | **Tick cost at scale.** Per-minute ticks for every aviary, including dormant ones. | M / M | Tick compute metrics; DB load | Bucketed bulk batches; HOT updates; sharding plan; the provably equivalent dormant-batching optimization held in reserve (§6.1). |
| R-15 | **Export vs. "never exposed" conflict** draws legal or product dispute. | M / M | Legal review | D-02: opaque encoding; escalate to counsel before beta. |
| R-16 | **Newcomer mechanic under-discovered**, so users never adopt a third bird. | M / L | Dogfood (staging clock) | Narration, a notebook observation, and the offer-tray item; windows recur every 60–80 days; no pressure by design. |
| R-17 | **Content repetition** over months: notebook and narration start to feel templated. | M / M | Sample reviews; dogfood diaries | Large template volume; recent-use memory; ongoing writer investment. |
| R-18 | **Visit abuse:** spam invitations, forwarded links. | L / M | Invite rate metrics | Per-host limits; no free-text message; single-browser binding; revocation; 30-day expiries. |
| R-19 | **Schedule risk** from many bespoke systems (synth, rig, grammar, engine). | M / M | Milestone exits | Vertical slice by week 8; explicit cut lines (§15.4); species delivered in pairs. |
| R-20 | **Cross-engine determinism drift** makes client and server disagree on greeting or caption decisions. | L / M | Determinism tests across engines | Integer PRNG and fixed-point only in canonical decision paths (§3.5). |
---

## 18. Decision log (ambiguities resolved)

| ID | Ambiguity or conflict in the PRD | Decision | Why |
|---|---|---|---|
| D-01 | The top bar lists four icons and says "nothing else", but settle is "triggered from the top bar". | Settle is a fifth top-bar control. "Nothing else" is read as no other *kinds* of chrome (badges, counters, status). | Two PRD files require settle in the top bar. A fifth quiet control keeps the bar sparse. |
| D-02 | The export includes "current personality vectors", but the user "never sees personality vector values… not at any version". | The export includes personality as an opaque, versioned, checksummed encoding (`personality_state`) for portability. There is no readable number panel anywhere. Legal review before beta. | Satisfies portability without making a numeric surface. The export is a system artifact, not a product surface. |
| D-03 | The snapshot "includes per-bird … current moods, current call timing", and clients need behavior inputs. | Snapshots carry quantized, mixed presentation levels. Traits never leave the server. | Keeps INV-03 true even for curious users reading network payloads. |
| D-04 | Presence activity is specified as "pointermove or keypress". | `pointermove`, `pointerdown`, `keydown`, all `isTrusted`. | `keypress` is deprecated and misses navigation keys. Touch devices rarely emit `pointermove`, and ignoring taps would systematically bias phone users. |
| D-05 | Multi-device presence isn't specified. | Presence-time is the union of intervals across devices. | Attention is per person, not per device. Summing would inflate drift. |
| D-06 | "Quieter after absence" vs. "traits never move down". | A bounded, non-trait attunement signal affects expression only. It never feeds wary, and there is always one greeter. | The only way to satisfy both statements. The fences are property-tested. |
| D-07 | "Low-pass filter" and "drift during absence based on inputs from before they left". | Reservoir-and-release filter (§6.4). | Directly expresses both statements and is monotonic by construction. |
| D-08 | The brief says "whether you mute the calls" shapes drift. The engine's input list omits it. | Audibility scales only the vocal component of presence inflow (0.5–1.0×). Never negative. | Honors the brief without punishing mute. |
| D-09 | What an offer does during cooldown. | The offer always works visually. Cooled-down birds glance, get no drift, and no timer UI exists. | "Functional, not punitive", and no game-like cooldown display. |
| D-10 | The offer isn't aimed at a bird, but "offering near a bird" matters. | The drop point is front-center, or nearest the listened-in bird if there is one. | Gives "near" a concrete, user-intelligible meaning. |
| D-11 | Who decides the greeter. | The server plans the order per tick. The client chooses the form from the absence class and realizes the variation. | Canonical for the notebook, and still never canned. |
| D-12 | Audio and rendering when unfocused or hidden. | Hidden: stop rendering, fade audio out. Visible but unfocused: render and play normally, with no presence recorded. | Honors battery and the PRD's hidden-tab rule. A second-monitor aviary stays alive without inflating presence. |
| D-13 | Autoplay policies vs. "calls already audible". | Audible immediately where allowed; otherwise resume silently on the first activation with a slow fade. No enable-sound prompt. | A prompt would be an announcement. |
| D-14 | What "one-time link" means, and how long visitor access lasts. | The link binds once to one browser. Access lasts 30 days from first use unless revoked. Visitors get no notebook, greeting, or interaction. Host timezone. | Deliberate, bounded sharing; "no permanent visitor list". |
| D-15 | How the visitor knows who invited them (no profiles). | The invite email names the host's account email, disclosed to the host at invite time. No free-text message. | No display names exist. The email is verified (anti-phishing) and there's no spam vector. |
| D-16 | Channel for opt-in visit notifications. | Email only, system voice, ≤ 1 per visitor per day, no aviary content. | No push exists in v1. In-product badges are forbidden. |
| D-17 | Notebook names after a rename. | Render with the current name. | Names are the user's labels. Identity is the `bird_id`. Avoids "who's pip?". |
| D-18 | How "a new species offer appears in the user's flow" without announcing. | A newcomer lingers in the scene and can be welcomed from the offer tray. Windows are age-based. No prompts. | Makes the offer a noticing, not a notification. |
| D-19 | Bird removal. | No release or removal in v1. | Conflicts with identity continuity and "birds do not die"; not in the PRD. |
| D-20 | The mood set is "finalized in implementation", and the layout needs night sleep. | Add `roosting`. Prose calls it settled or asleep. | The layout requires sleeping birds, and the internal name avoids collision with the settle gesture. |
| D-21 | Scope of accessibility preferences. | Synced per account. Reduced-motion "system" resolves per device. | A captions user wants captions everywhere. OS motion settings are device facts. |
| D-22 | Timezone with multiple devices in different zones. | One canonical account timezone. Auto-updates after sustained presence in a new zone, with eased lighting. | Both devices must show "the same aviary in the same mood". |
| D-23 | Language. | English only in v1, with externalized copy. | The naturalist grammar is authored per language. |
| D-24 | Keyboard focus vs. Enter to listen in. | Enter or Space engages. Arrows transfer an active listen-in. Escape exits. | Browsing birds by keyboard shouldn't thrash the mix. Transfer mirrors click-switching. |
| D-25 | Offer shortcut. | Single key `o`, active only outside text inputs, disableable in settings. | Satisfies WCAG 2.1.4 while keeping the PRD's shortcut. |
| D-26 | Session lifetime. | 90-day sliding idle expiry. | A calm, low-friction product; the PRD's timeout copy still applies. |
| D-27 | Tick cadence for dormant aviaries. | Every aviary every minute in v1. A pure `step()` enables provably equivalent batching later. | The PRD states the tick runs regardless of connection. |
| D-28 | Prose generation technology. | Authored generative grammar. No LLM. | Privacy (INV-09), determinism, voice control. |
| D-29 | Business metrics (DAU, retention). | Not collected, beyond request, sign-in, and account create/delete counts. | The PRD's aggregate categories and anti-engagement stance. Legal and product to confirm; engineering defaults to conservative. |
| D-30 | Real vs. synthetic weather. | Synthetic, seeded, rare. | No location data; weather "is never assertive". |
| D-31 | Sunrise and sunset without location. | Stylized curve from local clock plus a tz reference latitude for season and hemisphere. | Local-time anchoring without geolocation. |
| D-32 | Which calls are canonical. | Multi-bird social calls are tick-scheduled. Solo idle calls are client-generated. | Notebook-visible events must be canonical; the rest is presentation. |
| D-33 | Birds appearing after a slow snapshot load. | Birds emerge from the foliage edges in staggered short flights. | Avoids "fade from nothing" and keeps the aviary continuing. |
| D-34 | Where errors appear, given "no toasts". | A quiet inline system line near the top bar, only on real failure, self-clearing. | Errors need clarity (matter-of-fact) without announcement patterns. |
| D-35 | "Mood resets daily-ish" vs. "never snaps". | A strong relaxation toward the personality baseline around local dawn. | Honors both. |
| D-36 | Magic-link prefetch by mail scanners. | Token in the URL fragment, consumed by POST, with a button fallback. | Prevents silent token burn and log leakage. |
| D-37 | Sign-up flow. | Unified with sign-in. The first consumption creates the account and runs starter adoption. | Magic-link only; no separate registration. |
| D-38 | Calibration data source. | Harness plus consenting staging cohort. Never production aggregates. | INV-09. |
| D-39 | Backups vs. "every record… gone" after 30 days. | PII crypto-shredded at hard delete. Simulation backups expire within 14 days. Disclosed. | Selective deletion from immutable backups isn't possible. Bounded, disclosed residue. |
| D-40 | Rename concurrency. | Optimistic concurrency with 409 and a matter-of-fact note. | User-authored scalar; the no-LWW rule targets personality. |
| D-41 | Measurement definition for time-to-first-bird. | Return visit on the reference profile (gated). Cold first visit tracked separately. | A cold first visit is physically bounded by connection setup and preceded by sign-in and adoption. Both numbers are reported honestly. |

## 19. Open items for specialist input (non-blocking; defaults above apply)

1. **Legal:**
   - D-02 export encoding
   - D-15 disclosure of the host email to the visitor
   - D-29 metric set
   - D-39 backup residue language
   - visitor-data retention (12 months)
2. **Design:** the exact top-bar iconography, including the settle glyph; focus-ring colors; the caption backing treatment; key-pose art direction for reduced motion.
3. **Sound design:** final species grammars and the signature-separation constraint thresholds (P-60).
4. **Content:** the suggested-name list and song-fragment names; review of the grammar starter corpus.
5. **Engine and design jointly:** the final W (P-02) after dogfood; drift gains after the harness plus dogfood.

---

## Appendix A — Parameters (initial values; server config, versioned)

| ID | Parameter | Initial | Notes |
|---|---|---|---|
| P-01 | Tick interval | 60 s | Degradable to 120 s. |
| P-02 | Presence activity window W | 4 min | Calibrate 3–5 min, lean long. |
| P-05 | Top-bar fade delay / target opacity | 4 s / 10% | |
| P-10 | Daily presence concavity C | 3600 s | |
| P-11..16 | Presence and listen-in inflow weights | §6.4 table | Listen-in C_L = 900 s. |
| P-17 | Reservoir release τ | 2 days | |
| P-18 | Ceiling exponent γ | 1.5 | |
| P-19 | Drift gain g | 0.08 | Slow edge of the band. |
| P-20 | Offer cooldown per bird | 4 min | "A few minutes." |
| P-21 | Daily release cap per trait | 0.012 | |
| P-22 | Seeds / ceilings | base 0.15–0.45 ± N(0, 0.05); ceilings 0.70–0.95 | |
| P-25/26 | Attunement floor / half-life | 0.30 / 5 days | |
| P-30 | Mood minimum dwell | 8 min | Impulses exempt. |
| P-33 | Alarm call rate | 0–2 per aviary-day | |
| P-35 | Greeting absence classes | 2 min / 2 h / 2 days | |
| P-40..43 | Rate limits | Magic link 5/15 min and 20/day per email, 30/h per IP; invites 10/day and 30 outstanding per host | |
| P-45 | Adoption ceiling (global) | 3 at beta | Raised per §16.2. |
| P-50 | Notebook buckets | Regular cap 2, refill 1 per 3 days; noteworthy cap 1, refill 1 per 2 days, salience ≥ 80 | |
| P-55 | Narration idle cadence | 30–60 s; priority ≤ 1.5 s; min spacing 5 s | |
| P-60 | Recognizability margin | Between-bird distance ≥ 2.5× the within-bird 95th percentile | Tuned by the sound designer. |
| P-65 | Listen-in ramps | Engage 3.0 s to +6/−10 dB; disengage 3.5 s; floor −14 dB | |
| P-70 | Settle | Light ramp 6 s (10 s reduced motion); duck −12 dB over 4 s; undo 5 s | |

## Appendix B — PR guardrail checklist (in the PR template)

- [ ] Does this add or change any user-visible text? The surface is registered in §2.3, it uses the right register, and it passes the voice lint.
- [ ] Does this add any UI inside the aviary scene? It shouldn't (focus ring and captions excepted).
- [ ] Does this show, count, or imply visit frequency, streaks, progress, levels, achievements, or trait values? It must not.
- [ ] Does this notify, ping, badge, or email the user about the aviary? It must not.
- [ ] Does this write bird state outside the tick, accept a client-provided trait, mood, or perch, or make any trait decrease possible? It must not.
- [ ] Does this touch `birds`, `bird_personality`, or `bird_state` schemas? Two-person review and a staging dry run are required.
- [ ] Does this add a metric, log field, or RUM field? It is on the allowlist, with no account, bird, or aviary identifiers in aggregates and no email anywhere.
- [ ] Does this add a dependency or external request? It is on the allowlist, with no third-party analytics or scripts.
- [ ] Does this add motion? The reduced-motion register is specified, and flash safety holds.
- [ ] Does this add audio? It is procedural, adds no new AudioNodes per call, and has a caption path.
- [ ] Does it stay within the performance budgets (bundle, frame, memory)? CI must be green.
