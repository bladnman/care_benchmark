# Pocket Aviary — v1 Implementation Plan

**Status:** proposed for engineering execution
**Audience:** the team building v1 (backend/simulation, frontend/render, audio, design, writing, QA/accessibility)
**Source of truth:** the PRD (`prd/`). Where this plan and the PRD disagree, the PRD wins and this plan is wrong.

## How to read this document

This is an interpretation of the PRD into buildable decisions, not a restatement of it. Where the PRD names a value ("about a week," "a few minutes," "roughly seven"), this plan picks a concrete starting number and names the harness that will calibrate it. Where the PRD leaves a mechanism open, this plan picks one and says why. Where the PRD contains an internal tension, this plan resolves it, marks the resolution, and lists it in §17 for product sign-off.

Two things are worth stating before the technical content, because they change how a reader should weight the rest of it.

First, **this product's failure modes are mostly invisible to tests.** A bird whose personality vector silently reset, a drift function running 3× fast, a notebook entry that reads like an event log, a spinner that slipped into the loading path — none of these fail a unit test, and all of them destroy the thing the user came for. A disproportionate amount of the engineering budget below goes to making those failures *mechanically impossible* rather than *caught in review*: schema grants that make client-written personality unreachable, a CI voice lint, an observation taxonomy with no user-behavior member, a render-mode parameter instead of a reduced-motion code path. Where you see a rule in the PRD stated with unusual force, you will find a mechanism here rather than a convention.

Second, **the three-week claim is on the critical path in wall-clock time.** "Visible drift after about three weeks" cannot be validated by an accelerated harness alone; it needs real accounts drifting in real time. That forces a scheduling constraint (§15) that is easy to discover too late: the drift engine has to freeze roughly eight weeks before GA.

---

## Table of contents

1. [Scope](#1-scope)
2. [Principles, mechanisms, and what defends them](#2-principles-mechanisms-and-what-defends-them)
3. [Architecture](#3-architecture)
4. [Data model](#4-data-model)
5. [API surface](#5-api-surface)
6. [Simulation engine](#6-simulation-engine)
7. [Sync model](#7-sync-model)
8. [Frontend rendering pipeline](#8-frontend-rendering-pipeline)
9. [Audio pipeline](#9-audio-pipeline)
10. [The voice system: notebook, narration, captions](#10-the-voice-system-notebook-narration-captions)
11. [Accessibility surfaces](#11-accessibility-surfaces)
12. [Performance budgets and observability](#12-performance-budgets-and-observability)
13. [Security and privacy engineering](#13-security-and-privacy-engineering)
14. [Testing and calibration](#14-testing-and-calibration)
15. [Rollout and milestones](#15-rollout-and-milestones)
16. [Risks](#16-risks)
17. [Decisions needing product sign-off](#17-decisions-needing-product-sign-off)
18. [Appendices](#18-appendices)

---

## 0. Judgment calls register

Every place this plan decides something the PRD left open or ambiguous. Each is expanded where it lives; this is the index so a reviewer can audit the interpretation layer in one pass.

| # | Question | Call | Where |
|---|---|---|---|
| J1 | Is settle a top-bar item? `aviary_layout.md` enumerates four icons and says "nothing else"; `interactions.md` and `accessibility_perf.md` both say settle is triggered from the top bar. | Settle is a fifth top-bar glyph. Two files specify the behavior; one file's enumeration predates or overlooks it. Total bar inventory freezes at five. | §8.7, §17 |
| J2 | Does the account export contain personality vectors? `accounts_sync.md` lists them; `bird_engine.md` says the user never sees the numbers "at any version, in any tier." | Yes, include them — the export is a data-portability artifact, not a product surface. Hard-banned from ever being rendered in-product or re-imported into a UI. | §5.6, §13.4, §17 |
| J3 | Presence requires "pointermove or keypress." Touch devices emit neither in normal use. | Qualifying activity = `pointermove`, `pointerdown`, `keydown`, `touchstart`, `wheel`. Without this, presence is near-zero on phones and the product's mobile half stops drifting. | §6.2, §17 |
| J4 | Autoplay policy. Safari/iOS and Chrome block audio before a user gesture, which collides with "calls already audible" on the first frame. | The aviary opens visually alive in silence; audio fades in mid-phrase at the first genuine gesture. No modal, no "enable audio" prompt, no toast. | §9.7, §16 |
| J5 | Mood name collision: `bird_engine.md` lists moods and separately describes birds "settled" at night; "settled" is already the aviary's lighting state. | Night bird mood is named `roosting`. `settled` stays reserved for the aviary. | §6.4 |
| J6 | Who receives an offer, given offers are top-bar-triggered and not bird-targeted? | Offers are placed into the scene, not aimed at a bird. Every bird evaluates its own reaction against its own cooldown. | §6.7 |
| J7 | Notebook generation: templates or a language model? | Hand-written frame corpus with slot filling. A hosted LLM would put per-bird interaction data outside the simulation boundary, which the privacy commitment forbids. | §10.2, §13.3 |
| J8 | Drift instrumentation vs. the privacy rule. "Measurable in instruments after a week" cannot be measured across the production population without aggregating per-bird state. | Calibration runs on synthetic cohorts plus consented staff accounts flagged in the DB. Never on the user population. | §14.3, §13.3 |
| J9 | Transport for state: polling or push? | HTTP polling. The tick is 60s; sockets buy nothing and cost connection state, and the PRD already describes a pull model. | §7.1 |
| J10 | Renderer: Canvas2D or WebGL2? | WebGL2 with a small hand-rolled renderer; Canvas2D fallback. Decision gate with measurements on reference hardware in week 6 — if Canvas2D holds 60fps with the full grade+parallax load, take it for the smaller bundle. | §8.2 |
| J11 | Soft-deleted accounts: keep ticking? | Pause the tick. On recovery, catch-up integration reproduces the identical state (the tick is deterministic and replayable), so continuity is preserved without processing data for a user who asked for deletion. | §6.9 |
| J12 | How does a third bird arrive? | It arrives on its own and a quiet in-voice acceptance appears when the user focuses it. Declining is free and re-offered later. No badge, no modal, no notification. | §6.10 |

---

## 1. Scope

### 1.1 In scope for v1

**Account and identity.** Email + magic-link sign-in, 15-minute link expiry, single-use links, per-device revocable sessions, verified email change, on-demand JSON export, 30-day soft delete then hard delete.

**One aviary per account.** Two starter birds at adoption, engine cap of seven, age-based arrival of birds three through seven.

**The bird engine.** Five-trait hidden personality vector per bird; monotonic-toward-expressive drift driven primarily by presence; six-state mood with continuous-time transitions; per-bird procedural call grammar with a lifelong identity signature; mood-shaped idle motion; bird-to-bird response and mood contagion.

**Server-side simulation.** ~60s canonical tick, running independently of client connection; append-only client event log; server as the only writer of personality state.

**The scene.** One horizontal non-scrolling responsive scene; three perch zones; local-time day/night cycle; rare passing weather; ambient leaf and feather drift; five-item top bar that fades on stillness; no chrome inside the scene.

**Interactions.** Return-greeting (varied by absence length, boldness, mood; staggered across birds; never repeated identically); listen-in with slow bidirectional mix ramps; three offer types with per-bird cooldowns; settle with a 5-second undo; presence accounting to the three-signal conjunction.

**The field notebook.** Sparse auto-generated naturalist observations, read-only, infinitely scrollable back.

**Visits.** Per-invite opt-in, one-time email link, 30-day expiry, immediate revocation, read-only ambient rendering, no visitor drift contribution, silent visit log, off-by-default visit notifications.

**Accessibility.** Prose screen-reader narration on a slow cadence, reduced-motion as a designed render mode, runtime-generated call captions, full keyboard navigation with focus indicators legible against every aviary state, WCAG AA on all user copy.

**Performance.** ≤2MB gzipped initial JS, <500ms to first bird on mid-tier mobile over 4G, 60fps idle on five-year-old hardware sustained over 30 minutes, zero memory growth over 30 minutes, client-side procedural audio, graceful silence + captions when WebAudio is unavailable.

### 1.2 Explicitly out of scope

Native apps of any kind (and the data model is not shaped for them). Payments and tiers. Shared, team, or household aviaries. Multiple aviaries per account. Customizable or purchasable scenes. Public discovery, directories, feeds, profiles, follows, comments, ratings. Leaderboards and any cross-account metric that could become one. Push notifications and marketing email about the aviary. Recorded audio at any quality, in any fallback. Co-presence during visits. Chat, avatars, or visitor representation of any kind.

And the whole gamification cluster, in every disguise: achievements, badges, levels, XP, ranks, tiers, scores, streaks, day counters, visit calendars, green dots, "birds adopted: 2," milestone celebrations, exportable visit logs, or a notebook entry that observes the user's attendance. Not as a setting, not opt-in, not "quiet." §2 lists the mechanisms that keep these out rather than trusting review to catch them.

### 1.3 Non-goals that constrain the build, not just the roadmap

Three of the refusals have direct engineering consequences and are worth pulling out, because a team that treats them as roadmap items will build architecture that quietly readmits them.

**No negative drift.** Traits move up or hold. There is no decay term, no neglect penalty, no "wariness" accumulator. This is enforced at the storage layer (§6.3), not in the drift function alone, because a decay term is exactly the kind of thing a well-meaning future change adds for "balance."

**No cross-account aggregation of bird state.** Refusing leaderboards means refusing the pipeline that would compute them. The simulation database is never replicated into the analytics warehouse, and the analytics role holds no grant on simulation tables (§13.3). The architectural absence is what makes the feature hard to add later, which is the point.

**No notification surface.** There is no push infrastructure, no notification service, no email template for anything except: magic link, email-change verification, export ready, visit invitation, and — only if the host explicitly enabled it — visit notification. No scheduled job may send email. This is enforced by keeping outbound email behind a single service with a closed template allowlist.

---

## 2. Principles, mechanisms, and what defends them

The PRD states five principles with unusual force. Each row below names the mechanism that implements the principle and the automated check that keeps it implemented after the people who read the PRD have moved on.

| Principle | Mechanism | Automated defense |
|---|---|---|
| **Feels alive, not robotic** — the aviary has been continuing without the viewer | Server tick runs on absent accounts; snapshot carries a `motion_epoch` so the client seeds micro-motion mid-action; idle motion from continuous noise fields, not animation cycles; calls composed at runtime | Frame-zero test: first painted frame must show ≥1 bird with a pose-phase ≠ 0 and ≥1 ambient element in flight. Loop-detection test: 500 consecutive calls from one bird must produce 500 distinct parameter vectors. Asset lint: zero audio files in the bundle. |
| **Notice, never announce** | No toast/banner/modal primitives exist in the design system. The greeting is the only session-start surface. Errors render as inline surfaces on the affected control | Component lint: importing or defining a component named `Toast`/`Banner`/`Snackbar`/`Notification` fails the build. Route test: session start renders no element containing user-directed second person. |
| **Charm comes from specificity** | All product copy is generated by one `voice-kit` package shared by notebook, narration, and captions; slot-filled from realized aviary facts | Voice lint over the frame corpus and all product strings (§10.4): rejects second person, capitalized sentence starts, exclamation marks, gamification vocabulary, and generic-state phrasing. |
| **Restraint over richness** | Engine-level cap of 7 birds; single non-scrolling scene; five-item top bar; palette constrained to the design-system token set | Schema constraint on bird count. Token lint: any color literal outside the palette tokens fails. Top-bar inventory is a frozen enum with a test asserting length 5. |
| **Naturalist product voice, matter-of-fact system voice** | Two string namespaces, `voice/naturalist/*` and `voice/system/*`, with a compile-time rule that account, auth, error, sync, and settings surfaces may only import from `voice/system` | Import-boundary lint. Copy review gate for any new string in either namespace. |

Two additional refusals get the same treatment because they are the most reachable regressions:

| Refusal | Mechanism | Automated defense |
|---|---|---|
| **No streak / visit-frequency surface** | The notebook's observation taxonomy is a closed enum containing only aviary-fact types. There is no observation type whose subject is the user. Visit-frequency is never computed outside of rate-limiting counters that are not readable by any product surface | Test asserts the taxonomy enum has no member whose subject is the account holder. API contract test asserts no response body contains a visit count, day count, or date-set field. |
| **Personality is never exposed numerically** | Snapshots carry quantized *expression* parameters derived from multiple traits at once, never trait values (§7.3). The API schema is closed | Contract test: snapshot response validated against a closed JSON schema; any additional property fails. Inversion test: given 10⁵ synthetic states, the mapping from traits to the exposed expression vector must be non-injective across the whole trait space. |

---

## 3. Architecture

### 3.1 Shape

Five deployable units. The split is drawn along the two boundaries that carry real weight — *who may write personality state*, and *what may touch per-account interaction data* — rather than along feature lines.

```
                    ┌───────────────────────────────────────┐
   browser ────────►│ edge (CDN)                            │
                    │  static bundle, HTML + inline snapshot│
                    └──────────────┬────────────────────────┘
                                   │
                    ┌──────────────▼────────────────────────┐
                    │ api  (stateless, autoscaled)          │
                    │  auth, snapshot read, event ingest,   │
                    │  notebook read, settings, invites     │
                    │  DB role: aviary_api                  │
                    │   – SELECT on all simulation tables   │
                    │   – INSERT only on interaction_event  │
                    │   – NO write grant on personality     │
                    └──────┬───────────────────────┬────────┘
                           │                       │
              ┌────────────▼─────────┐   ┌─────────▼───────────┐
              │ postgres (simulation)│   │ redis               │
              │  accounts, birds,    │   │  hot snapshots,     │
              │  personality, mood,  │   │  presence buckets,  │
              │  events, notebook    │   │  rate limits,       │
              └────────────▲─────────┘   │  tick lease         │
                           │             └─────────────────────┘
              ┌────────────┴─────────┐
              │ sim  (tick workers)  │
              │  drift, mood, calls, │
              │  weather, notebook   │
              │  DB role: aviary_sim │
              │   – sole writer of   │
              │     personality/mood │
              └──────────────────────┘

              ┌──────────────────────┐   ┌─────────────────────┐
              │ mail (closed template│   │ ops telemetry       │
              │  allowlist)          │   │  aggregate-only,    │
              └──────────────────────┘   │  separate store     │
                                         └─────────────────────┘
```

**`edge`** — CDN serving the static bundle and the HTML document. The document carries an inlined bootstrap snapshot (§8.4) so the first bird can be drawn without a round trip.

**`api`** — stateless HTTP service. Handles auth, snapshot reads, event ingest, notebook reads, account settings, invites, and the visitor path. Its database role has `SELECT` on simulation tables and `INSERT` on `interaction_event` and nothing else. It is structurally incapable of writing a personality vector.

**`sim`** — the tick workers. Sole holder of write grants on `bird_personality` and `bird_mood`. Sharded by `account_id`, leased through Redis so exactly one worker owns a shard at a time.

**`mail`** — outbound email with a closed template allowlist (§1.3).

**`ops`** — aggregate telemetry, in a separate datastore with no network path to the simulation database (§13.3).

The `api`/`sim` split is not about scaling. It is the "no last-write-wins for personality" rule expressed as an infrastructure boundary: the service that talks to clients cannot write the state clients must not own. A single-service design would rely on developer discipline for the most consequential invariant in the product.

### 3.2 Client/server split

| Owned by server | Owned by client |
|---|---|
| Personality vectors and all drift | Frame-rate rendering |
| Mood state and transitions | Idle micro-motion (noise-field driven) |
| Perch assignment | Interpolation and flight paths between snapshot perches |
| Call scheduling (when a bird calls) | Call synthesis (what it sounds like), from a seeded, deterministic phrase realization |
| Weather state | Ambient leaves and feathers (pure ornament, no state) |
| Notebook entry generation | Narration text, captions (from `voice-kit`, same corpus) |
| Presence crediting and validation | Presence detection (the three-signal conjunction) |
| Time-of-day phase | Palette interpolation between phases |

The rule underneath: **the server owns discrete facts, the client owns continuous ornament.** A snapshot changes a bird's perch from `back` to `front`; the client decides what the flight looks like. A snapshot says a call fires at T+3.2s with phrase seed `0x8f21`; the client decides the exact partial amplitudes. Because ornament is never authoritative, reconciliation is smooth by construction and no client state ever needs merging.

### 3.3 Render pipeline boundary

Nothing crosses from `api` into the renderer except a validated snapshot object. The renderer takes `(snapshot, localTime, motionProfile, audioState)` and is otherwise pure — no network, no storage, no direct access to settings. This makes reduced-motion a parameter rather than a branch (§11.3), makes the renderer testable headlessly against fixture snapshots, and makes visitor rendering (§5.7) literally the same code with a snapshot that has no interaction affordances attached.

### 3.4 Technology choices

| Layer | Choice | Reason |
|---|---|---|
| Client framework | Preact + signals, ~5KB gz | The interactive surface is a five-item top bar and a few panels. React's weight buys nothing against a 180KB critical-path budget. Panels are lazy chunks. |
| Renderer | WebGL2, hand-rolled (~20KB), Canvas2D fallback | Day/night grading, weather, and parallax are per-pixel work that is near-free on GPU. Decision gate in week 6 (J10). No scene-graph library. |
| Audio | WebAudio + one `AudioWorklet` DSP kernel per voice | Worklets give allocation-free steady state and the f0 curves the call grammar needs. Pooled `OscillatorNode` path as secondary (§9.6). |
| Server language | TypeScript (api) / Rust (sim) | `api` shares `voice-kit` and schema types with the client. `sim` is a tight numeric loop with a hard p99 budget; Rust removes GC tail latency from the alarm path. |
| Database | Postgres 16 | Row-level grants, transactional tick batches, `jsonb` for event payloads. No scale requirement here justifies anything exotic. |
| Cache/coordination | Redis | Hot snapshots, presence buckets, tick shard leases, rate limits. |
| Hosting | Single region + global CDN | Latency budget is met by edge-delivering HTML and the bootstrap snapshot; the tick is 60s and does not need regional replicas. |

---

## 4. Data model

All times UTC. All identifiers are synthetic UUIDv7 (time-ordered, so they index well and carry no PII). Email appears in exactly two columns in the entire schema — `account.email_ciphertext` and `invite.visitor_email_ciphertext` — both envelope-encrypted, each paired with a keyed HMAC blind index for lookup.

### 4.1 Accounts and auth

```sql
CREATE TABLE account (
  id                 uuid PRIMARY KEY,              -- synthetic; the only id used anywhere else
  email_ciphertext   bytea NOT NULL,                -- envelope-encrypted, KMS data key
  email_hmac         bytea NOT NULL UNIQUE,         -- blind index for lookup; keyed, not a plain hash
  email_pending_ct   bytea,                         -- pending email change, until verified
  email_pending_hmac bytea,
  timezone           text NOT NULL DEFAULT 'UTC',   -- IANA, refreshed from client on each session
  created_at         timestamptz NOT NULL,
  status             account_status NOT NULL,       -- 'active' | 'pending_delete' | 'deleted'
  delete_requested_at timestamptz,
  settings           jsonb NOT NULL DEFAULT '{}',
  is_calibration     boolean NOT NULL DEFAULT false -- consented staff account (§14.3)
);

CREATE TABLE device_session (
  id            uuid PRIMARY KEY,
  account_id    uuid NOT NULL REFERENCES account(id) ON DELETE CASCADE,
  token_hash    bytea NOT NULL UNIQUE,              -- SHA-256 of the bearer token; token never stored
  label         text,                               -- "Firefox on macOS" — UA-derived, coarse
  created_at    timestamptz NOT NULL,
  last_seen_at  timestamptz NOT NULL,
  revoked_at    timestamptz
);

CREATE TABLE magic_link (
  id           uuid PRIMARY KEY,
  email_hmac   bytea NOT NULL,
  token_hash   bytea NOT NULL UNIQUE,
  created_at   timestamptz NOT NULL,
  expires_at   timestamptz NOT NULL,                -- created_at + 15 min
  consumed_at  timestamptz,
  request_ip_hash bytea                             -- truncated + salted, for rate limiting only
);
CREATE INDEX ON magic_link (email_hmac, created_at DESC);
```

`account.settings` is a closed, versioned document — not a bag. Adding a key requires a schema-version bump and a migration, which is deliberate friction against "just add a toggle."

```jsonc
{
  "v": 1,
  "captions": "auto",            // "on" | "off" | "auto" (auto = on iff audio unavailable)
  "motion": "system",            // "system" | "reduced" | "full"
  "audio": "on",                 // "on" | "off"
  "narration": "auto",           // "auto" (on iff AT detected/opted) | "on" | "off"
  "visit_notifications": false   // off by default, never surfaced in onboarding
}
```

### 4.2 Aviary and birds

```sql
CREATE TABLE aviary (
  account_id      uuid PRIMARY KEY REFERENCES account(id) ON DELETE CASCADE,
  created_at      timestamptz NOT NULL,             -- drives age-based bird arrival
  tick_index      bigint NOT NULL DEFAULT 0,        -- monotonic; also the snapshot version
  last_tick_at    timestamptz NOT NULL,
  event_watermark bigint NOT NULL DEFAULT 0,        -- last interaction_event.seq consumed
  weather         jsonb NOT NULL DEFAULT '{"kind":"none"}',
  settled_until   timestamptz,                      -- set by the settle gesture
  next_arrival_at timestamptz                       -- when bird N+1 becomes available
);

CREATE TABLE bird (
  id              uuid PRIMARY KEY,                 -- stable for the life of the account
  account_id      uuid NOT NULL REFERENCES account(id) ON DELETE CASCADE,
  name            text NOT NULL,
  species_id      text NOT NULL,                    -- 'warbler' | 'wren' | ... (§18.1)
  species_version int  NOT NULL,                    -- frozen at adoption; pool updates never re-skin
  seed            bigint NOT NULL,                  -- drives call signature + motion identity, forever
  adopted_at      timestamptz NOT NULL,
  slot            smallint NOT NULL                 -- 0..6, stable ordering
);
CREATE UNIQUE INDEX ON bird (account_id, slot);

-- Cap enforced in the database, not only in application code.
CREATE OR REPLACE FUNCTION enforce_bird_cap() RETURNS trigger AS $$
BEGIN
  IF (SELECT count(*) FROM bird WHERE account_id = NEW.account_id) >= 7 THEN
    RAISE EXCEPTION 'aviary bird cap (7) exceeded for account %', NEW.account_id;
  END IF;
  RETURN NEW;
END $$ LANGUAGE plpgsql;
```

`bird.id` and `bird.seed` are immutable. Renaming writes `bird.name` and nothing else. A species-pool revision writes a new `species_version` row in config and leaves existing birds on their frozen version; there is no migration path that swaps a live bird's species, and none will be added. This is the storage-level form of the identity-continuity rule: there is no code path, including internal tooling, that can replace one bird with another.

### 4.3 Personality and mood

```sql
CREATE TABLE bird_personality (
  bird_id            uuid PRIMARY KEY REFERENCES bird(id) ON DELETE CASCADE,
  boldness           real NOT NULL,
  social_warmth      real NOT NULL,
  vocal_frequency    real NOT NULL,
  plumage_saturation real NOT NULL,
  curiosity          real NOT NULL,
  revision           bigint NOT NULL DEFAULT 0,
  updated_at         timestamptz NOT NULL,
  CONSTRAINT traits_in_range CHECK (
    boldness BETWEEN 0 AND 1 AND social_warmth BETWEEN 0 AND 1 AND
    vocal_frequency BETWEEN 0 AND 1 AND plumage_saturation BETWEEN 0 AND 1 AND
    curiosity BETWEEN 0 AND 1)
);

-- Monotonicity is a database invariant, not a convention in the drift function.
CREATE OR REPLACE FUNCTION personality_monotonic() RETURNS trigger AS $$
BEGIN
  IF NEW.boldness < OLD.boldness - 1e-6 OR NEW.social_warmth < OLD.social_warmth - 1e-6
     OR NEW.vocal_frequency < OLD.vocal_frequency - 1e-6
     OR NEW.plumage_saturation < OLD.plumage_saturation - 1e-6
     OR NEW.curiosity < OLD.curiosity - 1e-6 THEN
    RAISE EXCEPTION 'personality drift is monotonic; attempted decrease on bird %', NEW.bird_id;
  END IF;
  RETURN NEW;
END $$ LANGUAGE plpgsql;
CREATE TRIGGER trg_personality_monotonic BEFORE UPDATE ON bird_personality
  FOR EACH ROW EXECUTE FUNCTION personality_monotonic();

REVOKE ALL ON bird_personality, bird_mood FROM aviary_api;
GRANT SELECT ON bird_personality, bird_mood TO aviary_api;
GRANT SELECT, INSERT, UPDATE ON bird_personality, bird_mood TO aviary_sim;
```

The trigger is the important line in this file. A negative-drift regression cannot ship: it raises in production and in every test that touches the tick. The `1e-6` tolerance absorbs float round-trip noise without admitting a decay term.

```sql
CREATE TABLE bird_mood (
  bird_id        uuid PRIMARY KEY REFERENCES bird(id) ON DELETE CASCADE,
  mood           mood_state NOT NULL,   -- 'alert'|'curious'|'content'|'wary'|'drowsy'|'roosting'
  mood_since     timestamptz NOT NULL,
  perch          perch_zone NOT NULL,   -- 'front'|'middle'|'back'
  perch_since    timestamptz NOT NULL,
  next_call_at   timestamptz,
  call_seed      bigint NOT NULL,       -- reseeded per call; makes phrases reproducible client-side
  offer_cooldowns jsonb NOT NULL DEFAULT '{}',   -- {"seed":"2026-07-26T10:04:00Z", ...}
  greeting_debt  jsonb                  -- pending greeting, style + scheduled offset
);
```

`bird_mood` has no "reset" column and no default-on-connect path. Mood is written only by the tick. A session-start handler that set mood would need a grant it does not have.

### 4.4 Interaction events

```sql
CREATE TABLE interaction_event (
  id          uuid PRIMARY KEY,                     -- client-generated; the idempotency key
  seq         bigserial NOT NULL,                   -- server order; the tick consumes by this
  account_id  uuid NOT NULL REFERENCES account(id) ON DELETE CASCADE,
  bird_id     uuid REFERENCES bird(id) ON DELETE CASCADE,
  kind        event_kind NOT NULL,
  payload     jsonb NOT NULL DEFAULT '{}',
  client_ts   timestamptz NOT NULL,
  server_ts   timestamptz NOT NULL DEFAULT now(),
  session_id  uuid REFERENCES device_session(id)
);
CREATE INDEX ON interaction_event (account_id, seq);
REVOKE UPDATE, DELETE ON interaction_event FROM aviary_api, aviary_sim;
```

`event_kind`: `presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `settle_undo`, `session_open`, `session_close`, `bird_focus`.

Append-only is enforced by revoking `UPDATE`/`DELETE`. Duplicate delivery is a primary-key conflict, discarded on `ON CONFLICT DO NOTHING` — which is what makes the client's at-least-once retry queue safe (§7.2).

Presence is stored in a compact aggregate rather than one row per ping, because pings arrive every 15s per active session and the tick only ever needs sums:

```sql
CREATE TABLE presence_minute (
  account_id   uuid NOT NULL REFERENCES account(id) ON DELETE CASCADE,
  minute       timestamptz NOT NULL,     -- truncated to the minute
  seconds      smallint NOT NULL,        -- 0..60, server-clamped
  focused_bird uuid,                     -- bird under listen-in for the majority of the minute
  PRIMARY KEY (account_id, minute)
);
```

### 4.5 Notebook, invites, visits, exports

```sql
CREATE TABLE notebook_entry (
  id               uuid PRIMARY KEY,
  account_id       uuid NOT NULL REFERENCES account(id) ON DELETE CASCADE,
  created_at       timestamptz NOT NULL,
  local_date       date NOT NULL,           -- the aviary's date, in the account timezone
  observation_type observation_type NOT NULL,  -- closed enum; no user-behavior member (§10.3)
  frame_id         text NOT NULL,           -- corpus frame used, for anti-repetition
  text             text NOT NULL,           -- realized prose, stored (never regenerated)
  facts            jsonb NOT NULL           -- the slot values, for audit and QA
);
CREATE INDEX ON notebook_entry (account_id, created_at DESC);

CREATE TABLE invite (
  id                      uuid PRIMARY KEY,
  host_account_id         uuid NOT NULL REFERENCES account(id) ON DELETE CASCADE,
  visitor_email_ciphertext bytea NOT NULL,
  visitor_email_hmac      bytea NOT NULL,
  token_hash              bytea NOT NULL UNIQUE,
  created_at              timestamptz NOT NULL,
  expires_at              timestamptz NOT NULL,     -- created_at + 30 days
  first_used_at           timestamptz,
  revoked_at              timestamptz
);

CREATE TABLE visit (
  id            uuid PRIMARY KEY,
  invite_id     uuid NOT NULL REFERENCES invite(id) ON DELETE CASCADE,
  host_account_id uuid NOT NULL REFERENCES account(id) ON DELETE CASCADE,
  started_at    timestamptz NOT NULL,
  last_pull_at  timestamptz NOT NULL,
  duration_bucket_s int NOT NULL DEFAULT 0          -- rounded to the minute; "approximate" by design
);

CREATE TABLE export_job (
  id                 uuid PRIMARY KEY,
  account_id         uuid NOT NULL REFERENCES account(id) ON DELETE CASCADE,
  requested_at       timestamptz NOT NULL,
  completed_at       timestamptz,
  download_token_hash bytea,
  expires_at         timestamptz                    -- requested_at + 24h
);
```

The notebook stores realized text rather than regenerating it from `facts`. An entry the user read three months ago must read the same way tomorrow; regenerating would let a corpus revision silently rewrite the user's history, which is a quieter version of resetting a bird.

`visit.duration_bucket_s` is rounded to the minute deliberately — the visit log is a transparency surface, not a measurement surface, and minute precision is all the transparency purpose needs.

### 4.6 Species pool

The six-species pool lives in versioned code, not the database: silhouette geometry, default palette, motif library, and behavioral biases per species. It is loaded by both `sim` (call scheduling biases) and the client (rendering, synthesis) from a shared package, so the two can never disagree about what a wren is. See §18.1 for the pool.

---

## 5. API surface

REST over HTTPS, JSON, cookie-bearer sessions (`SameSite=Lax`, `HttpOnly`, `Secure`). Every response validated against a published JSON Schema; every schema closed (`additionalProperties: false`), which is what makes the "no trait numbers" contract test possible.

### 5.1 Auth

```
POST /v1/auth/link            { "email": "…" }                  → 202 (always)
POST /v1/auth/consume         { "token": "…" }                  → 200 + Set-Cookie
POST /v1/auth/signout                                           → 204
GET  /v1/account/sessions                                       → 200 [{id,label,created_at,last_seen_at,current}]
DELETE /v1/account/sessions/{id}                                → 204
```

`POST /v1/auth/link` returns 202 whether or not the email has an account. Any other behavior is an account-enumeration oracle, and the enumeration set here is "people who use Pocket Aviary," which is exactly the kind of thing a user assumes is private.

Consumption is a **POST**, not a GET on the emailed URL. The email links to `GET /signin?t=…`, which renders a page that immediately `POST`s the token via JS, with a plain `<form method=post>` button as the no-JS path. Corporate mail scanners and link-preview bots follow GET links; if the magic link were consumed by GET, a meaningful fraction of users would find their link already used before they clicked it, and the failure would look like a flaky auth system. The same treatment applies to visit invitations (§5.7).

Rate limits: 5 link requests per email per hour, 20 per IP per hour, with an exponentially increasing delay rather than a hard block, so a user who mistypes their address twice is not locked out.

### 5.2 Snapshot read

```
GET /v1/aviary/snapshot            → 200 snapshot  (ETag: "<tick_index>")
                                   → 304 if If-None-Match matches
```

Called on: initial load (though the first one is inlined into the HTML, §8.4), `visibilitychange` to visible, a render-frame gap over 2s (laptop resume), and a 25s keepalive while visible. Never called while hidden.

The snapshot is the entire read surface of the aviary. Shape:

```jsonc
{
  "v": 184213,                          // = aviary.tick_index; monotonic, also the ETag
  "server_time": "2026-07-26T14:02:11Z",
  "motion_epoch": "2026-07-26T13:44:00Z",  // phase anchor for client micro-motion (§8.5)
  "day": { "phase": "morning", "progress": 0.42, "tz": "America/New_York" },
  "weather": { "kind": "rain", "intensity": 0.35, "ends_at": "2026-07-26T14:09:00Z" },
  "aviary": { "settled": false },
  "birds": [
    {
      "id": "01931f…",
      "name": "pip",
      "species": "finch",
      "species_version": 1,
      "seed": "7f3a9c21",                // stable; drives call signature + motion identity
      "perch": "front",
      "perch_since": "2026-07-26T13:58:02Z",
      "mood": "content",
      "mood_since": "2026-07-26T13:31:00Z",
      "render":  { "plumage_step": 7, "detail_step": 5 },        // quantized, 0..11 / 0..7
      "voice":   { "rate_step": 6, "tempo_step": 4, "ornament_step": 3 },  // quantized 0..7
      "posture": { "openness_step": 5, "vigilance_step": 2 },    // quantized 0..7
      "calls": [
        { "at": "2026-07-26T14:02:19Z", "phrase_seed": "3c81b0", "kind": "spontaneous" },
        { "at": "2026-07-26T14:03:04Z", "phrase_seed": "91ee42", "kind": "answer", "to": "01931e…" }
      ],
      "greeting": { "style": "call_and_step", "at": "2026-07-26T14:02:13Z" }  // present only when owed
    }
  ]
}
```

Three properties of this payload are load-bearing.

**No trait values appear anywhere.** `render`, `voice`, and `posture` are *expression* vectors: small integers, each derived from two or more traits plus mood plus time of day, quantized to 8–12 steps. The mapping is deliberately non-injective — a `rate_step` of 6 is reachable from many (vocal_frequency, mood, phase) combinations — so a determined client cannot invert it into "boldness: 0.62." A contract test asserts non-injectivity over a 10⁵-point sample of the trait space. This is what lets the client render personality faithfully while keeping the numbers genuinely unavailable, including to the user's own devtools.

**Calls are scheduled ahead, not pushed.** The server publishes the next ~90 seconds of call events with their seeds. The client synthesizes each phrase from `(bird.seed, phrase_seed, voice.*, mood)`, deterministically. This means calls fire on time regardless of network jitter, two devices watching the same aviary hear the same chorus, and the audio never waits on a request.

**`greeting` is present only when a greeting is owed.** It is not a flag the client interprets; it is the server having decided which bird notices, in what style, at what offset (§6.6).

Snapshots are ~2–5KB uncompressed for a seven-bird aviary, well inside the PRD's "kilobytes, not megabytes."

### 5.3 Event ingest

```
POST /v1/aviary/events   { "events": [ … up to 64 … ] }   → 202 { "accepted": [...ids] }
```

Batched, idempotent on `event.id`, at-least-once from the client. Events:

```jsonc
{ "id":"…", "kind":"presence_ping",   "client_ts":"…", "payload":{ "window_s":15, "visible":true, "focused":true, "last_activity_ms":8200 } }
{ "id":"…", "kind":"listen_in_start", "bird_id":"…", "client_ts":"…" }
{ "id":"…", "kind":"listen_in_end",   "bird_id":"…", "client_ts":"…", "payload":{"duration_s":214} }
{ "id":"…", "kind":"offer",           "client_ts":"…", "payload":{ "offer":"seed|song|pool", "motif_id":"m_07" } }
{ "id":"…", "kind":"settle",          "client_ts":"…" }
{ "id":"…", "kind":"settle_undo",     "client_ts":"…" }
```

The server never trusts `client_ts` for crediting. Presence is credited from server receipt time, clamped (§6.2). A client that claims a 15-minute presence window in one ping is credited at most one ping-interval.

`offer` carries no `bird_id`: offers are placed in the scene, not aimed (J6).

Rejections are silent at the protocol level (202 with the id omitted from `accepted`) except for auth failures. A failed presence ping must never produce a user-visible surface — that would be the system announcing its own plumbing into a product whose entire premise is quiet.

### 5.4 Notebook

```
GET /v1/notebook?before={cursor}&limit={n≤50}   → 200 { entries: [...], next_cursor }
```

Read-only. There is no POST, PATCH, or DELETE on this resource and there will not be one; the notebook is an observer's record, and an edit endpoint is the affordance that would turn it into a journal the user curates. Entries paginate backward indefinitely; nothing is archived or hidden.

### 5.5 Birds

```
PATCH /v1/birds/{id}          { "name": "pippa" }   → 200
GET   /v1/aviary/arrival                            → 200 { "pending": {...} | null }
POST  /v1/aviary/arrival/accept                     → 200
POST  /v1/aviary/arrival/decline                    → 204
```

`PATCH` writes `bird.name` only. There is no endpoint that writes personality, mood, or perch; the API service could not honor one if it existed (§4.3 grants). Name validation is length and control-character stripping — no profanity filter, no uniqueness requirement. The user naming their bird is the first thing the product asks of them and it should not be an argument.

### 5.6 Account

```
GET   /v1/account                                   → 200 { email_masked, created_at, timezone, settings }
PATCH /v1/account/settings                          → 200
POST  /v1/account/email-change  { "email":"…" }     → 202  (old address works until new verifies)
POST  /v1/account/export                            → 202  (emailed download link, 24h, session-gated)
POST  /v1/account/delete                            → 202  (soft; 30-day window)
POST  /v1/account/delete/cancel                     → 200
```

Export contents: account (email, created, timezone, settings), birds (id, name, species, adopted_at, **personality vector**, current mood), notebook entries in full, invites and visit log. Per J2 this includes the raw trait values, resolving the tension between `accounts_sync.md` (which lists them) and `bird_engine.md` (which forbids showing them) in favor of data portability — with three guards: the export is never rendered in-product, no import path exists, and the file carries a short matter-of-fact note explaining that these values are internal and are not meaningful to compare. §17 flags this for product sign-off; if product prefers, the alternative is to omit the vector and the export loses nothing a user could act on.

The download link requires an authenticated session — a link alone is not sufficient — because the email inbox is not the security boundary we want for a file containing the user's full interaction history.

### 5.7 Invitations and visits

```
POST   /v1/invites            { "email":"…" }   → 201 { id, expires_at }
GET    /v1/invites                             → 200 [{id, email_masked, created_at, expires_at, state}]
DELETE /v1/invites/{id}                        → 204   (revocation; immediate)
GET    /v1/visits                              → 200 [{visitor_email_masked, started_at, duration_min}]

GET    /v1/visit/{token}/snapshot              → 200 snapshot | 410 gone
```

The visitor path is a separate route tree with its own handler namespace and its own DB role (`aviary_visit`, `SELECT`-only, no grant on `interaction_event` at all). A visitor cannot generate an interaction event because there is no code path and no permission to write one — not a filter applied afterward. This is the mechanism behind "a visitor sitting and watching for an hour does not drift the host's birds."

The visitor snapshot is the host's snapshot with `greeting` stripped (a greeting is for the host) and no listen-in affordance attached. Captions and narration are available to visitors; accessibility is not a host-only feature.

Revocation takes effect on the next pull: the handler checks `revoked_at`/`expires_at` before serving and returns `410` with a matter-of-fact body. There is no push channel to terminate a visit faster, and none is needed — the visitor pulls every 25s.

### 5.8 Errors

All error bodies use `voice/system` strings (§2). The complete v1 set is in §18.4. The API never returns naturalist copy, and the aviary never returns system copy; the import-boundary lint enforces the split at build time so this cannot drift.

Transport errors during a session are **not** surfaced. If the snapshot request fails, the client keeps rendering from the snapshot it has and keeps queueing events; the aviary continues, which is both truthful (it is continuing, server-side) and correct for the product. An error surface appears only when a user-initiated action fails — an offer that could not be recorded, a settle that did not register — and it appears inline on that control, never as a banner. An offline indicator would be the product announcing its own plumbing at the exact moment the user is quietly watching birds.

---

## 6. Simulation engine

### 6.1 The tick

One tick = 60 seconds of aviary time, identified by `aviary.tick_index`. The tick is a **pure, deterministic function**:

```
tick(state, events_in_window, tick_index, account_tz, wall_clock_window) → state'
```

All randomness comes from a counter-based PRNG seeded by `hash(bird.seed, tick_index, purpose)`. There is no `rand()`, no wall-clock entropy, no ambient state. This single property earns three things at once: replayability for debugging, exact equality between eagerly-ticked and caught-up accounts (§6.9), and a test harness that can run a simulated year in seconds and get precisely the state a real year would have produced.

```
for each due account (leased shard, batch of 256):
  events   ← interaction_event WHERE account_id=? AND seq > watermark ORDER BY seq
  presence ← presence_minute rows in the tick window
  for tick_index in (last_tick_index+1 .. now_index):
      s ← advance_weather(s, rng(tick_index,'weather'))
      s ← advance_daylight(s, account_tz)
      for bird in s.birds:
          Δ ← drift_delta(bird, presence, events, tick_index)      # ≥ 0, always
          bird.personality ← clamp01(bird.personality + Δ)
          bird.mood        ← mood_step(bird, s, events, rng(...))
          bird.perch       ← perch_step(bird, s, rng(...))
          bird.calls       ← schedule_calls(bird, s, rng(...))
      s ← bird_to_bird(s, rng(tick_index,'social'))
      s ← maybe_write_notebook(s, rng(tick_index,'notebook'))
  persist(s, tick_index=now_index, watermark=max(seq))
```

The whole batch is one transaction. Persistence is a compare-and-set on `aviary.tick_index`: if another worker advanced it, this batch's writes are discarded and re-derived. Double-processing a tick would double-apply drift, which is a silent data-quality failure of exactly the kind §2 exists to prevent, so it is made unrepresentable rather than unlikely.

**Cost.** Seven birds × ~40 float ops × 60 ticks of catch-up is microseconds. At 100k active accounts the load is ~1.7k account-updates/sec, batched 256 at a time into ~7 transactions/sec of multi-row `UPDATE … FROM (VALUES …)`. This is unremarkable for Postgres. Rows are written only when something changed materially: mood, perch, call schedule, or accumulated drift exceeding 1e-5. A dormant aviary at 3am writes nothing.

**Tiering.** Hot accounts (presence in the last 30 min, or a client polling) tick every 60s. Warm accounts (activity in the last 7 days) tick every 60s as well — they are cheap. Cold accounts tick every 15 wall-clock minutes, replaying the 15 skipped indices in-process. Because the tick is deterministic and index-driven, the coarse path computes *the same sequence of tick functions*, not an approximation of them; the result is bit-identical. A property test asserts this over randomized event traces, so the optimization can never become a semantic difference.

Any snapshot read forces catch-up to the current index before responding. A user who returns after two weeks gets an aviary that has genuinely advanced two weeks, computed from the events that existed before they left.

### 6.2 Presence accounting

Client-side detector (`presence.ts`), evaluated on an animation-frame-throttled 1s timer:

```ts
const ACTIVITY_WINDOW_MS = 240_000;   // 4 min, initial; §14.4 calibrates
const PING_INTERVAL_MS   =  15_000;

function isPresent(): boolean {
  return document.visibilityState === 'visible'
      && document.hasFocus()
      && (now() - lastActivityAt) < ACTIVITY_WINDOW_MS;
}
```

`lastActivityAt` is updated by `pointermove`, `pointerdown`, `keydown`, `touchstart`, and `wheel` — passive listeners, coalesced to at most one update per 500ms so the listener itself costs nothing. The PRD names pointermove and keypress; adding pointerdown/touchstart is J3, and it is not a loosening: a tap is at least as strong a signal of a person being there as a mouse twitch, and without it presence on phones is structurally near-zero. Scroll is included via `wheel` but the aviary does not scroll, so it will rarely fire.

The window is 4 minutes, on the long side of "a few," because *watching birds without moving is the product*. Presence should end when a person has stopped showing any sign of being there, not when they have stopped fidgeting. Calibration (§14.4) will move this between 3 and 6 minutes based on the distribution of inter-activity gaps in real sessions — measured, per the privacy boundary, as an aggregate histogram with no per-account dimension.

Server-side crediting, in `POST /v1/aviary/events`:

```
credited = min( now − last_ping_at_for_session , PING_INTERVAL × 1.5 )
if !(payload.visible && payload.focused && payload.last_activity_ms < ACTIVITY_WINDOW):
    credited = 0
if session is a visitor session: reject (no code path exists)
presence_minute[account, trunc_minute(now)] += credited, clamped to 60
```

Two clamps matter. The per-ping clamp means a client cannot inflate presence by claiming a long window or by replaying pings. The per-minute clamp to 60 means two devices open simultaneously cannot double-count a single human's attention — a user watching on laptop and phone at once is one person watching, and crediting 120 seconds per minute would put that account's drift on a different curve from everyone else's. That second clamp is easy to miss and would corrupt calibration exactly the way the PRD warns a lax presence definition does.

No backfill, ever. When the tab is hidden the client stops pinging, and on return it does not reconstruct the gap. Presence that was not observed is not credited.

### 6.3 Drift

Traits are floats in [0,1], hidden, server-owned. Per tick, for each trait `t`:

```
Δ_t = α · w_t · input_t · (1 − v_t)          with Δ_t ≥ 0 enforced structurally
```

- `α = 9.0e-6` per second of presence-equivalent input — the master calibration constant.
- `w_t` — per-trait weight: social_warmth 1.0, plumage_saturation 0.9, boldness 0.8, vocal_frequency 0.7, curiosity 0.6.
- `input_t` — presence-equivalent seconds accruing to this trait this tick (below).
- `(1 − v_t)` — asymptotic approach. Diminishing returns fall out for free: a bird that has drifted a long way drifts more slowly, so long-tenured aviaries do not saturate into identical maximally-expressive birds, and the shape of change stays interesting after a year.

**Input sources per tick:**

| Source | Contribution | Traits |
|---|---|---|
| Presence | credited seconds in the tick, ×1.0 | all five, via `w_t` |
| Listen-in on this bird | overlapping seconds, ×2.5 | social_warmth, vocal_frequency (only) |
| Offer accepted by this bird | +60 presence-equivalent seconds, once | curiosity |
| Offer placed while bird is present | +30 presence-equivalent seconds, once | boldness |
| Settle | 0 | none — it ends the presence window and quiets mood, nothing more |

**Calibration derivation.** Define "regular visits" as 5 sessions/week × 12 minutes of credited presence = 3,600 presence-seconds/week. With `v₀ = 0.35` (seed midpoint) and `w_t = 1.0`:

```
v(T) = 1 − (1 − v₀)·e^(−αT)
week 1  (T=3.6e3):  Δv = 0.65·(1 − e^−0.0324) = 0.021
week 3  (T=1.08e4): Δv = 0.65·(1 − e^−0.0972) = 0.060
one 12-min session:  Δv = 0.65·(1 − e^−0.0065) = 0.0042
```

Read against the PRD's three requirements: 0.021 after a week is an order of magnitude above the 0.002 instrument noise floor, so it is *measurable in instruments*. 0.0042 in a session is 5% of one perceptual quantization step, so *no single session moves anything visibly*. 0.060 after three weeks crosses at least one step in every behavioral mapping below, so it is *visible when the user looks back*. The narrow band the PRD describes is exactly the band between the second and third rows, and `α` is the knob that sets it.

**Making drift visible.** Hidden numbers only matter if they change what the user sees. The mappings, with the slope chosen so that Δv = 0.06 produces a change a person would notice on reflection but not in the moment:

| Trait | Surface | Effect of Δv = 0.06 over three weeks |
|---|---|---|
| boldness | perch-choice weights; approach distance to offers; greeting proximity | P(front perch) +8pp; approaches a seed from one zone nearer |
| social_warmth | P(greets first today); P(answering another bird within 6s) | +10pp greet-first; +12pp answer rate |
| vocal_frequency | base inter-call interval; P(joining a chorus) | interval −9%; chorus join +7pp |
| plumage_saturation | shader saturation + feather-detail alpha, 12 quantized steps with hysteresis | +1 step — invisible frame to frame, evident against a three-week-old screenshot |
| curiosity | P(investigating an offer); head-tilt-to-sound rate; watching passing leaves | +9pp investigate; noticeably more head-tilts |

Quantization with hysteresis on the visual mappings prevents shimmer at a boundary; behavioral mappings are continuous because probability changes cannot shimmer.

**Monotonicity** is enforced three ways: `Δ_t ≥ 0` by construction (every input term is non-negative), a `clamp01` that cannot decrease, and the database trigger in §4.3 that raises on any decrease. Nothing in the engine models neglect, wariness accumulation, or decay. A bird that is ignored simply stops receiving input; its traits hold, and it reads as quieter because its call scheduling reflects a vocal_frequency that stopped rising, not one that fell.

### 6.4 Mood

Six states (J5): `alert`, `curious`, `content`, `wary`, `drowsy`, `roosting`. `roosting` is the night state — eyes closed, low on the perch. The aviary's evening lighting state remains `settled`; keeping the two words apart avoids a class of bug where a bird's state and the scene's state are confused in a conditional.

Mood is a continuous-time Markov chain sampled per tick. For each ordered pair, an intensity:

```
λ(from→to) = λ_base[from][to]
           · f_time(phase, to)          # drowsy/roosting rise near dusk; alert peaks at dawn
           · f_weather(weather, to)     # rain → +wary, −vocal; wind → +alert and +wary, split by boldness
           · f_recent(events, to)       # accepted offer → ×3 toward content/curious for ~10 min
           · f_neighbors(aviary, to)    # a wary neighbor raises wary; contagion, decaying with perch distance
           · f_personality(bird, to)    # high boldness damps entry into wary; high curiosity favors curious
P(transition to `to` this tick) = 1 − e^(−λ·60)
```

`f_personality` is where the PRD's "a high-boldness bird is less likely to enter wary even on the same input" lives: the boldness term multiplies λ into `wary` by `(1 − 0.6·boldness)`, so a drifted-bold bird visibly weathers a startle its neighbor does not. That is another channel by which three weeks of presence becomes something the user can see without being told.

**Daily-ish reset** is a dawn re-draw rather than a hard reset: at local sunrise, mood re-samples from a personality-weighted prior (a high-warmth bird wakes toward `content`/`curious`; a low-boldness bird wakes toward `alert`). This satisfies the daily cadence while never producing a snap on tab open, because it is anchored to the aviary's clock, not the session's.

**Persistence.** The mood a bird has at session end is the mood it has at session start, modulo the tick. There is no session-start mood code path (§4.3). A test opens a session, records mood, closes, reopens 30s later, and asserts mood is unchanged; a second test does the same across a simulated 14-hour gap and asserts the change is exactly what an unattended tick would have produced.

### 6.5 Perch selection

Perch is chosen per tick from a softmax over `front/middle/back` weighted by boldness, mood, weather, and neighbor positions, with a strong stickiness term (a bird that just moved is unlikely to move again for several minutes) so the scene reads as calm rather than twitchy. Mood dominates on short timescales — a `wary` bird sits back regardless of boldness — and boldness biases the resting distribution over weeks, which is what makes "Pip comes closer than she used to" true.

Nothing in the API lets a client set a perch. Perch is a signal the user reads, and a placement affordance would erase the signal; there is no drag handler, no perch command, and no field in any request body that names a perch.

### 6.6 Return-greeting

The greeting is the anchor moment of a session and is computed server-side, because the inputs (absence length, canonical mood, canonical boldness) live there.

Absence is `now − last_presence_credit`. Bird selection: each bird gets a greet-weight `boldness · (1 + social_warmth) · mood_factor(mood)`, where `mood_factor` is ~0 for `roosting` and low for `wary`; one bird is sampled by weight. Style is drawn from a table indexed by absence band and the selected bird's boldness:

| Absence | Low boldness | Mid | High boldness |
|---|---|---|---|
| < 10 min (stepped away) | glance up from preening | glance + head-tilt | short two-note call |
| 10 min – 6 h | head-tilt, holds position | two-note call | call + step toward front |
| 6 h – 2 d | slow scan toward viewer | call, then a step | approach to front perch + call |
| > 2 d | scan, then a single low call | approach one zone | approach to front + longer call, often answered |

Every style is a *parameterized* behavior, not a clip: the call is realized by the same procedural grammar as every other call (with a longer phrase budget), the approach is the same flight system, the head-tilt is the same pose blend at a mood-shaped amplitude. There are no greeting animations in the asset pipeline, so there is nothing that *could* be played identically twice. This matters because "three pre-recorded variants in rotation" is the failure the PRD explicitly names, and the way to not build it is to have no variants to rotate.

When a second bird would greet (high social_warmth answering the first), it fires at `+U(0.9s, 2.6s)`, never in unison. Unison would announce the user's arrival to the aviary; staggering makes it the aviary noticing, one bird at a time.

The greeting is the entire welcome. There is no toast, banner, modal, absence-duration copy, or return-related text anywhere in the client; the component lint in §2 makes adding one a build failure rather than a code-review conversation.

### 6.7 Offers

Offers are placed into the scene, not aimed at a bird (J6). A `seed` lands on the front rail; a `pool` fades in at the front of the scene; a `song` motif plays softly into the aviary. Each bird then evaluates independently:

```
P(investigate) = σ( a·curiosity + b·mood_openness(mood) − c·distance(perch, offer) − d·weather_damp )
```

`drowsy` and `roosting` birds have `mood_openness ≈ 0` and generally do not approach; `wary` birds have a long latency term and may arrive a minute later, which is the "waits and eventually comes near" the PRD describes. The `song` offer resolves against vocal_frequency and mood instead: join in, go quiet, or call against it.

**Cooldowns.** Per `(bird, offer_type)`: 4 minutes. Global per aviary: 45 seconds between placements. Both are functional, not punitive — without them, curiosity input saturates within one session and the drift model collapses into a clicker. Neither is surfaced as a timer, a countdown, or a disabled-with-tooltip state; the affordance simply doesn't re-arm, and the birds' non-reaction is legible on its own. A cooldown timer would be a game surface.

### 6.8 Settle and weather

**Settle.** Sets `aviary.settled_until = now + 8h`. Lighting crossfades to evening over 6 seconds, call scheduling stretches by ~2.5×, birds bias toward `drowsy`. It contributes **no drift** in any direction. Any click, tap, or keypress within 5 seconds reverses the lighting (a `settle_undo` event; the tick discards the paired `settle`). After the undo window, re-engagement also lifts it — the aviary is settled until the user actively comes back, and closing the tab is equally valid. Nothing anywhere records or reacts to whether a session ended with a settle.

**Weather.** A per-account Poisson process, ~3 events/week, drawn from `{rain: 0.6, wind: 0.4}`, duration 4–11 minutes, intensity capped at 0.5. No thunder, no snow, no event the user must notice. Effects are short: rain multiplies call rate by ~0.6 and biases `wary`/`drowsy`; wind raises `alert` in bold birds and `wary` in timid ones. Effects decay over ~10 minutes after the event ends, so the aviary carries a small aftermath rather than snapping back.

### 6.9 Lifecycle: dormancy, deletion, recovery

Dormant accounts tick coarsely (§6.1) and write nothing when nothing changes.

Soft-deleted accounts **stop ticking** (J11). This honors the user's instruction — we should not be running a simulation over the data of someone who asked us to delete it — without costing continuity, because on recovery the catch-up path replays every skipped index and produces bit-identical state to an account that never paused. The determinism property earns this directly; without it we would have to choose between honoring the deletion and preserving the aviary.

Hard deletion at 30 days is a cascade from `account`, plus a job that purges the Redis snapshot cache, the export objects, and the mail service's delivery log for that address. A monthly audit job asserts that no row anywhere references an `account_id` absent from `account`.

### 6.10 Bird arrival (third bird and beyond)

Availability is a function of `aviary.created_at` and nothing else — not visits, not interaction volume, not presence-time. Ladder: **day 90, 180, 300, 450, 630**, capping at seven. A one-year-old aviary has five birds, occasionally six; that matches the PRD's pacing and, more importantly, is uncorrelated with how much attention the user has paid. Any coupling to activity would teach the user that attention earns stuff, which is the mechanism the PRD is refusing.

Arrival is quiet (J12). At the appointed tick a bird of an unheld species arrives on the back perch with a default name, and the notebook is likely to remark on it in the ordinary way it remarks on things:

> a titmouse has been at the back perch since morning. pip has not decided about it yet.

When the user focuses the newcomer, a single in-voice line offers the choice — `let it stay` / `not now` — and the affordance is bounded to that focus state. Declining is free, non-final, and re-offered at the next rung. There is no badge on the settings icon, no email, no modal on load, and no counter anywhere that says how many birds the user has. Discovery happens by looking at the aviary, which is what the product is for.

---

## 7. Sync model

### 7.1 Why there is nothing to sync

Multi-device sync is not a feature in this system; it is the absence of a problem. The server holds one canonical aviary per account and is its only writer. Two devices signed into the same account are two readers of one row. There is no client-side state to merge, no vector clock, no CRDT, no last-writer, and no conflict resolution — because there is no second writer to conflict with.

Everything in §3.1 and §4.3 exists to keep that true as the codebase grows. The `api` service holds `SELECT` on personality and mood and no more. A future engineer who writes `UPDATE bird_personality` in the API service gets a permission error in local development on the first run, which is a far better teacher than a comment.

Transport is HTTP polling (J9): pull on load, on `visibilitychange` → visible, on a render-frame gap > 2s, and on a 25s keepalive while visible. With a 60s tick, a socket would deliver one meaningful message per minute in exchange for connection state, reconnect logic, and a scaling dimension. SSE is a reasonable later optimization if we ever want sub-second visitor revocation; it is not needed for v1.

### 7.2 Writes: the event log

Clients write only interaction events, append-only, at-least-once:

1. The event is written to an IndexedDB outbox with a client-generated UUIDv7 (`id`) and its `client_ts`.
2. A flusher posts batches of ≤64 every 5s, or immediately for user-initiated events (offer, settle, listen-in).
3. On 2xx, matching ids are deleted from the outbox. On failure, exponential backoff to 60s.
4. Duplicates collide on the primary key and are discarded server-side.

Outbox bounds: 500 events or 24 hours, whichever comes first, then oldest-first eviction. Presence pings are evicted preferentially and are **never** replayed after 2 minutes — a stale ping represents attention that has already ended, and crediting it would silently inflate drift for every user with a flaky connection. This is the same failure the PRD warns about, arriving through the retry path instead of the definition.

### 7.3 Reads: snapshot reconciliation

The client keeps a `PresentationState` that renders every frame and reconciles to each new snapshot:

| Snapshot field changed | Client response |
|---|---|
| `perch` | Play a flight from the current rendered position to the new zone. Never teleport, never cut. |
| `mood` | Cross-blend pose weights and motion parameters over 2–4s. Never snap. |
| `render.plumage_step` | Ramp saturation over ~8s with hysteresis. |
| `calls[]` | Merge into the audio scheduler by `phrase_seed`; already-played entries are idempotent. |
| `weather` | Cross-fade the weather layer over 20–40s. |
| `day.phase/progress` | Continuous palette interpolation; the client advances the day locally between snapshots. |
| `greeting` | Arm the greeting; fire at the specified offset; clear. |

Because the client never *owns* any of these, reconciliation is always a transition, never a merge. If the client's local extrapolation drifted from the server (a suspended laptop, a long stall), the correction is a visible-but-natural movement — a bird flies to where it actually is — rather than a pop.

### 7.4 Clock and timezone

Clock skew is handled by trusting the server's `server_time` and computing an offset at each snapshot, smoothed over the last five samples. All scheduling (calls, greeting offsets) is expressed in server time and converted through that offset.

The account's IANA timezone is written on each session open (and only if it changed). The day/night cycle interpolates on absolute time against locally-computed sunrise/sunset for the account's timezone, so DST transitions shift the curve smoothly instead of jumping the palette an hour. A user who flies across the Atlantic finds an aviary on their new local morning at their next session — the aviary shares their day, which is the point of anchoring locally at all.

---

## 8. Frontend rendering pipeline

### 8.1 Layers

Five composited layers, back to front:

1. **Sky** — full-viewport gradient, day/night driven, plus a slow cloud-luminance field.
2. **Background foliage** — soft, low-contrast, parallax factor 0.15.
3. **Mid plane** — the three perch zones and the birds. Parallax 1.0.
4. **Ambient ornaments** — leaves and feathers in flight. Parallax 1.0–1.3.
5. **Foreground** — an occasional branch or leaf sweeping through, parallax 1.4, heavily blurred.

Parallax responds to viewport width only (there is no camera and no pointer-driven parallax — pointer-parallax would make the scene respond to the mouse, converting a window into a toy). The whole scene is graded per-frame by a single day/night + weather color transform, which is one shader pass on the GPU path.

### 8.2 Renderer

WebGL2, hand-rolled, roughly 20KB (J10). Everything is textured quads from one build-time atlas generated from SVG sources; birds are small rigs (body, head, beak, near wing, far wing, tail, two legs) whose parts are posed by a bone transform per frame. No scene-graph library, no physics, no general-purpose engine.

The case for GPU: day/night grading, weather overlays, plumage saturation per bird, and reduced-motion cross-dissolves are all per-pixel blends that are nearly free in a fragment shader and meaningfully expensive in Canvas2D at 2560×1440 on integrated graphics. The five-year-old-laptop budget (§12) has to hold for 30 minutes, not 30 seconds, and CPU compositing is where that budget goes.

**Decision gate, week 6.** Build both paths for the mid plane and measure on the reference devices (§14.5). If Canvas2D sustains p95 frame ≤ 16.7ms with the full grade and parallax load on the 2019-class laptop, take Canvas2D and bank the bundle. The Canvas2D path ships regardless as the no-WebGL fallback, so this is a choice about which one is primary, not additional work.

There is no `<canvas>`-less DOM fallback and no static-image fallback. A static aviary is not this product.

### 8.3 Bird animation: noise fields, not cycles

Each bird's pose is a weighted blend of a small pose basis (perched-neutral, preen-back, preen-breast, scan-left, scan-right, tilt, fluff, crouch, wing-stretch). Blend weights come from three sources:

- **Mood targets** — `wary` biases scan + upright + back-leaning; `content` biases preen; `curious` biases tilt; `drowsy` biases fluff + crouch; `roosting` holds fluff + closed eyes; `alert` biases upright + high vigilance.
- **Continuous noise** — three octaves of simplex noise sampled at 0.05–0.4 Hz, seeded by `(bird.seed, motion_epoch)`. This is what makes micro-motion non-repeating: there is no cycle length, so there is nothing to notice looping.
- **Event impulses** — a call raises the beak and opens the throat; a nearby call triggers a tilt; a landing triggers a settle-shuffle.

Amplitude and rate are scaled by `posture.openness_step` and `posture.vigilance_step` from the snapshot, which is how drift reaches the body: a bolder bird over three weeks holds a fractionally more open posture and scans slightly less.

Because pose is a continuous function of `(seed, time, mood, posture)`, "a bird mid-preen on the first frame" needs no special case — evaluate the function at `now`, and it is wherever it has been.

Flight between perches is a short arc with wing-beat modulation, 400–900ms depending on distance and boldness. Landing carries a small weight-reset shuffle. Birds never slide, never fade between perches (except in reduced motion, §11.3), and never overlap a perch already occupied by a bird that has not moved.

### 8.4 First frame

The budget is <500ms to first bird on mid-tier mobile over 4G. The path:

| Step | Budget |
|---|---|
| DNS + TLS + TTFB from CDN edge | 150ms |
| HTML + inline critical CSS + **inline bootstrap snapshot** (≈20KB) | 40ms |
| Critical JS (renderer + presentation state + bootstrap), ≤60KB gz | 120ms parse + exec |
| First draw | 60ms |
| **Total** | **~370ms**, with 130ms of headroom |

The bootstrap snapshot is inlined into the HTML document by the edge worker, which fetches it from Redis on the document request. This removes the round trip that would otherwise sit between "page loaded" and "aviary appeared" — the exact gap a spinner would be invented to fill. If the snapshot fetch at the edge exceeds 80ms, the worker ships the document without it and the client pulls normally; the quiet field covers the difference.

Everything else is lazy: audio worklet and motif library, notebook, settings panels, invite flow, offer panel, narration. The 2MB budget in the PRD is the ceiling for the whole application; our working target for the first-bird path is 180KB gz, and the remaining surfaces together are budgeted to ~700KB gz, leaving real headroom rather than spending the allowance.

**The loading state is a quiet field**, not a spinner: the sky gradient at the current local phase, painted from inline CSS before any JS runs, plus one or two faint drift cues once the renderer is up. There is no spinner primitive in the codebase, no skeleton component, and no fade-from-static transition. A build-time check fails on any element with a rotation-based CSS animation on the critical path. The same quiet field is the empty-aviary state after adoption (§8.8), so there is one visual for "the aviary is catching up," and it never reads as machinery.

### 8.5 Motion epoch

The snapshot's `motion_epoch` is the anchor the client seeds noise fields with. Two consequences: the first frame after a cold load shows each bird at the phase of motion it would have been at had the client been rendering all along, and two devices watching the same aviary show closely-matched motion. Micro-motion is not synchronized frame-for-frame across devices (it does not need to be), but it is anchored, so the two do not read as different aviaries.

### 8.6 Frame loop and lifecycle

```
rAF → dt (clamped to 50ms) → advance presentation state → cull → draw
```

Hidden tab: cancel rAF, suspend the AudioContext, stop presence pings. Nothing renders and nothing is credited; the simulation continues server-side, which is where it belongs. On `visibilitychange` → visible: pull a snapshot, re-anchor to the new `motion_epoch`, resume audio, resume the presence detector. A frame gap > 2s (laptop resume, heavy tab throttling) triggers the same reconciliation path.

Adaptive quality: if p95 frame time exceeds 14ms over a 10s window, step down — ornament density first, then foreground blur radius, then parallax layer count. Never step down bird fidelity; the birds are the product. The renderer exposes the current quality tier to RUM (§12.3) as an aggregate distribution.

### 8.7 Top bar

Five items, frozen (J1): **account/settings, accessibility, notebook, offer, settle.** Nothing else, at any point, without a PRD change. The inventory is a TypeScript union with a test asserting its length, so a sixth item is a failing build.

It fades to 8% opacity after 4 seconds of cursor stillness (or, on touch, 4 seconds after the last touch) and returns to full on pointer movement, touch, or any keyboard activity. Three rules on the fade:

- It is an opacity transition only. Never `display:none`, never `visibility:hidden`, never `aria-hidden` — a faded control must remain in the accessibility tree and in the tab order, or the fade has silently made the product keyboard- and screen-reader-hostile in exchange for an aesthetic.
- Focus forces full opacity. If anything in the bar has focus, it does not fade.
- It respects reduced motion by shortening rather than removing the transition (an instant 8% jump is its own kind of motion event).

Inside the aviary there is no chrome: no buttons, no badges, no hover tooltips, no overlay icons, no inline labels. Two bounded exceptions, both accessibility surfaces rather than UI: keyboard focus indicators (visible only during keyboard navigation) and call captions when enabled (§11.4). Both are conditional, quiet, and named here so they are not later cited as precedent for anything else.

### 8.8 Adoption and the empty aviary

Adoption: the system picks two species from the pool; the user is shown two suggested names, editable, presented as birds that arrived rather than a catalog to choose from. There is no species picker, no rarity display, no re-roll.

The scene then shows the quiet field — the same visual as loading — and the first bird enters with a soft fly-in to its starting perch, followed by the second a beat later. This is the only entry animation in the product, and it is scoped to genuinely-new birds (adoption, and later arrivals per §6.10). It never plays at session start. A test asserts that the fly-in path is unreachable from the session-open code path.

### 8.9 Responsive behavior

The scene is fluid between 320px and 2560px wide. Perch zones are placed proportionally with a minimum horizontal separation; below ~480px the three zones compress vertically as well as horizontally to preserve readable depth. Bird scale is clamped so a bird is never smaller than 36px tall (recognizability) or larger than 12% of viewport height.

The invariant: **no bird is ever cropped or offscreen, at any viewport, at any time.** This is a rendering test that runs across 12 viewport sizes × 7 bird counts × 3 perch zones and asserts every bird's bounding box is fully inside the frame with a 8px margin. Orientation changes re-lay out without moving birds discontinuously — they fly to their new zone positions.

---

## 9. Audio pipeline

Audio is the affective spine of the product, and it is the subsystem where a shortcut is most audible. Every call is synthesized at runtime. There are zero audio assets in the bundle, and an asset lint fails the build if one appears — including in a fallback path, including "temporarily."

### 9.1 Call grammar

Three layers, from species down to the individual utterance.

**Species motif library.** Each of the six species carries 4–7 motifs. A motif is a short note sequence, each note described by:

```ts
type Note = {
  f0: { shape: 'rise'|'fall'|'flat'|'arch'|'dip'|'trill', start_hz: number, end_hz: number, curve: number },
  dur_ms: number,
  amp: { attack_ms, decay_ms, sustain, release_ms },
  timbre: { partials: number[], fm_index: number, noise_mix: number, formant_hz: number },
};
type Motif = { id: string, notes: Note[], gap_ms: number[] };
```

**Phrase grammar.** A small stochastic production system per species composes motifs into a phrase:

```
phrase   → intro? body tail?
body     → motif | motif rest body | motif motif       (weights by species + mood)
rest     → silence(80–600ms, mood-scaled)
```

Phrase length, repetition depth, rest lengths, ornamentation probability, and tempo are all mood- and expression-scaled. A `drowsy` bird produces short bodies with long rests and little ornament; a `curious` bird produces longer bodies with more repetition.

**Per-bird signature.** This is the layer that makes "you know Pip by ear" true. From `bird.seed`, deterministically and *for the life of the bird*:

- a pitch centre offset (±3 semitones, quantized),
- a timbre fingerprint (fixed partial-amplitude ratios and `noise_mix` bias),
- a rhythm signature (a characteristic inter-onset ratio, e.g. long-short-short),
- a fixed subset of the species motif library — the bird's own vocabulary.

**The invariant that makes recognizability survive drift:** mood and drift modulate *tempo, dynamics, phrase length, repetition, ornamentation, and call frequency*. They never modulate pitch centre, timbre fingerprint, rhythm signature, or motif vocabulary. Recognizability is therefore a structural property of the synthesizer rather than something we hope survives calibration. A CI test enforces it directly: render every bird across all 6 moods × 5 drift levels, extract MFCC embeddings, and assert that within-bird spread stays below between-bird separation with a margin — for aviaries of 7 birds drawn from the same species pool, which is the hardest case and the one that sets the cap.

### 9.2 Synthesis

One `AudioWorkletNode` per voice, running a small DSP kernel:

```
f0 curve → 2 oscillators (carrier + FM modulator, index from timbre)
         → additive partials (up to 6)
         ⊕ bandpass-filtered noise (breath component)
         → resonant formant filter
         → ADSR gain
         → per-bird pan (from perch zone + slot)
         → dry + reverb send
```

Voices are **pooled and pre-allocated**: 8 worklet nodes (7 birds + 1 for the song-fragment offer), created at audio init and never recreated. Notes are delivered as messages, not as node graphs. This is the direct answer to "no memory growth over 30 minutes" — the steady state allocates nothing, because there is nothing to allocate. `OscillatorNode` is one-shot by spec and a pooled-node design built on it inevitably churns nodes; the worklet avoids that class of leak entirely.

Reverb is a feedback-delay network (4 delay lines, ~1KB of code), not a convolution with an impulse response, because an IR would be a recorded audio asset with a weight problem and a rule problem.

### 9.3 Chorus

Calls are scheduled by the server (§5.2) and realized on a client-side lookahead scheduler (120ms lookahead, 25ms tick) using `AudioContext.currentTime` — never `setTimeout` for note onsets.

Three rules keep a chorus from becoming a pile-up:

- **Onset jitter.** If two scheduled onsets fall within 150ms, the later one is nudged by 100–400ms. Birds answer each other; they do not fire in unison. This is the same instinct as the staggered greeting.
- **Bounded loudness.** A gentle soft-knee compressor (ratio 2:1, threshold −18dBFS, 30ms attack, 250ms release) on the master bus keeps a five-bird chorus from being louder than one bird by five times, without audible pumping.
- **Bed.** A quiet, continuously-varying ambient bed (filtered noise shaped by weather and time of day, ~−32dBFS) sits under everything so silence between calls is a place rather than a gap.

Because each voice is independently synthesized with its own f0 curves and its own signature, two birds calling together produce genuine spectral interaction. The phase-cancellation artifact the PRD warns about is specific to layering identical recordings and cannot arise here.

### 9.4 Listen-in mix

```
engage:    focused gain → +6dB  over 1.2s (exponential ramp, τ≈0.6s)
           others  gain → −9dB  over 1.6s, hard floor at −12dB
           bed     gain → +2dB
disengage: all return to ambient over 1.6s, same curve
```

Ramps use `setTargetAtTime`, never `setValueAtTime`. The interaction must feel like leaning in, not like switching channels; a hard cut would turn the aviary into a mixer with soloable tracks, which is a different relationship with the sound.

**Other birds never reach silence.** The −12dB floor is a constant in one place with a test asserting no code path can drive a non-focused bird's gain below it. Silencing the others would teach the user that the aviary is a set of things to switch between rather than a place where several things are happening at once.

Disengage triggers: clicking the focused bird again, focusing a different bird, clicking empty aviary space, moving keyboard focus away, or pressing Escape. All produce the same ramp; there is no "cancel" affordance and no state indicator beyond the mix itself and the focus ring.

### 9.5 Offer audio

The `song` offer plays a short melodic motif from a library of ~12 fragments on the spare voice, at −12dB relative to the bed, with a gentle low-pass. Birds respond per §6.7. The fragment is procedurally realized like everything else — same synthesizer, different grammar — so it belongs to the same sound world rather than sounding like a sample dropped into it.

### 9.6 Degradation ladder

| Condition | Behavior |
|---|---|
| WebAudio + AudioWorklet | Full pipeline. |
| WebAudio, no AudioWorklet | `OscillatorNode`/`GainNode` path with a strict pool budget and an explicit node-count ceiling of 48. Slightly simplified timbre (no FM, fewer partials). Memory soak test runs against this path too. |
| No WebAudio, or context creation denied | **Graceful silence with captions on by default.** The aviary is otherwise unchanged: birds still call (the calls are still scheduled and still narrated/captioned), still drift, still change mood. |

There is no recorded-audio fallback at any tier. Silence with captions is a better product than canned audio, and shipping a recorded path would also mean shipping the assets we just spent the bundle budget avoiding.

### 9.7 Autoplay policy (J4)

Browsers block audio before a user gesture. This collides directly with the PRD's first-frame requirement that calls are already audible, and it is not mentioned in the PRD — so it needs a designed answer rather than a default one.

The answer: **the aviary opens visually alive, in silence, with no prompt of any kind.** No "enable audio" modal, no unmute banner, no toast — all of which would be announcements at the precise moment the product is meant to be noticing the user. The AudioContext is created suspended. At the first genuine user gesture anywhere on the page (`pointerdown`, `keydown`, `touchend`) the context resumes and the bed fades in over 800ms; if a call is already scheduled in flight, we join it mid-phrase rather than starting a phrase at the moment audio arrives, so the sound reads as having been there rather than as having been switched on.

The top bar's accessibility glyph reflects audio state as a quiet visual, with no copy. If the user never gestures, they get the silent aviary, which is a legitimate way to experience it. Captions are **not** auto-enabled in this case — autoplay-blocked is not the same as audio-unavailable, and forcing captions on a user whose audio is about to work would be the system guessing loudly. After a successful resume, Chrome's media-engagement heuristics generally permit autoplay on subsequent visits, so this is mostly a first-session condition.

Safari specifics: the context must be resumed inside the gesture handler synchronously, and the page must re-resume on `visibilitychange` because iOS suspends aggressively. Both are covered by device-lab tests (§14.5).

---

## 10. The voice system: notebook, narration, captions

The PRD treats voice as substance, not decoration: specificity is named as the product's only real charm engine, and the notebook is called the surface where the voice is most concentrated. That makes voice an engineering subsystem with its own package, its own corpus, its own tests, and its own owner.

### 10.1 `voice-kit`

One package, used by three consumers: the notebook generator (server, in `sim`), the screen-reader narrator (client), and the caption generator (client). Sharing the implementation is the mechanism behind the PRD's requirement that a screen-reader user moving between the aviary and the notebook hears one product rather than two glued together — the two surfaces are not "written in the same style," they are literally the same generator with different observation inputs.

`voice-kit` exports:

```ts
realize(frame: Frame, slots: Slots, rng: Rng): string
selectFrame(observationType, facts, history): Frame
describeCall(realizedPhrase): string      // captions
narrate(snapshot, localTime, recentEvents): string   // narration
```

### 10.2 Notebook generation (J7)

Generation is a **hand-written frame corpus with slot filling**, not a language model. Three reasons, in order of weight:

1. **Privacy.** Per-bird interaction state may not leave the simulation boundary (§13.3). A hosted model would put exactly that data in a third party's inference path. A self-hosted model would satisfy the letter of the rule but adds an inference tier, a GPU cost line, and an eval burden for a surface that produces roughly two sentences per user per week.
2. **Voice stability.** The corpus is reviewed once by the writer and is then invariant. A model drifts across versions, and the drift would be invisible until a user noticed the notebook had started sounding like something else.
3. **Determinism.** Entries are auditable and reproducible in tests, which is what lets the voice lint (§10.4) be meaningful.

The corpus is ~300 frames written by the product writer, tagged by observation type, with slot constraints. Frames look like:

```
{ id: "greet_order_first_this_week",
  type: "greeting_order",
  text: "{bird_a} greeted before {bird_b} today, first time this week.",
  requires: ["greeting_order_changed", "streak_within_week"] }

{ id: "fluffed_cool_air",
  type: "posture_weather",
  text: "{bird} is fluffed against the cool air, watching the {perch} perch. low calls only.",
  requires: ["mood:drowsy|content", "temp_phase:cool"] }
```

**Sparsity.** Target ~2 entries/week for a regularly-visited aviary, with a hard floor of 36 hours between entries. Each candidate observation gets a noteworthiness score (rarity of the fact against this aviary's own history, magnitude of the change, whether the frame has been used recently); an adaptive threshold holds the rate near target even for a very active user. The notebook is not a feed — an entry per session would dilute the entries that matter into noise, and sparsity is what makes the user trust that an entry means something.

**Anti-repetition.** A frame is not reused within 60 days per account, and no `(frame, bird)` pair is reused within 120 days. Realization also varies within a frame (optional clauses, ordering) so even a repeat is not identical.

**Entries are immutable.** Written once, stored as realized text, never regenerated (§4.5).

### 10.3 What the notebook may observe

`observation_type` is a closed enum. Every member's subject is the aviary:

```
greeting_order · perch_change · mood_read · weather_moment · chorus_event ·
bird_to_bird · offer_reaction · plumage_note · quiet_stretch · arrival · time_of_day
```

There is no member whose subject is the user, and a test asserts that. This is the structural form of the no-streak rule: the notebook can write *"pip greeted before wren today, first time this week"* — a fact about birds, compared across days — and it has no vocabulary at all for *"you have visited every day this week."* The line is between observations of the aviary and observations of the user's behavior, and it is drawn in the type system rather than in a style guide, because a style guide would not survive the first well-meaning contributor with a good idea.

`quiet_stretch` deserves a note: it exists so an aviary that has not been watched can still be observed truthfully — *"a long stretch of quiet this morning. pip preened for several minutes without looking up."* That is an observation about birds. It is not a reproach, it does not reference absence, and frames of this type are reviewed specifically for that.

### 10.4 Voice lint

Runs in CI over the frame corpus, all product strings, and generated fixtures. Fails on:

| Rule | Applies to |
|---|---|
| Second person ("you," "your," "you've") | naturalist namespace |
| Capitalized sentence start | naturalist namespace |
| `!` anywhere | naturalist namespace |
| Gamification vocabulary (streak, level, badge, achievement, score, unlock, earned, milestone, progress, XP, rank) | **both** namespaces |
| Announcement verbs directed at the user (welcome, congratulations, great job, way to go) | **both** namespaces |
| Generic-state phrasing ("is happier," "mood improved," "is doing well") | naturalist namespace |
| Numeric trait references, or any digit in a trait context | **both** namespaces |
| Naturalist phrasing in `voice/system` (lowercase-start, bird nouns as subjects) | system namespace |
| Import of `voice/naturalist` from an auth, account, error, settings, or sync module | build-wide |

The lint is a floor, not a substitute for the writer. Every new frame goes through copy review; the lint exists so that the failure mode "someone added a string at 6pm on a Friday" is caught by the build rather than by a user.

### 10.5 Narration

Client-side, from the same snapshot the renderer reads, via `voice-kit.narrate()`. Client-side rather than server-side because narration must describe what is actually rendered — including reduced-motion presentation and realized call events — and because a 30–60s server round trip for text would couple an accessibility surface to network health for no benefit.

Cadence: one prose update every 30–60s at idle, jittered. User-initiated events (return-greeting, offer reaction, settle, listen-in engage) get a priority bump and narrate promptly. Nothing narrates more often than every 8 seconds under any circumstances; a queue that outruns the reader forces them to silence it, which is the product pushing its own accessibility surface aside.

Output is running prose in the same voice, never a state list:

> a small grey bird is perched on the front rail, calling softly. another sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Not:

> Pip: perch 2, mood content. Wren: perch 3, mood drowsy.

Priority events are still written as observations:

> wren has come to the front rail and is looking this way.

Not "greeting event fired."

### 10.6 Captions

Generated from the **realized phrase**, after synthesis parameters are resolved, not from a per-motif string. `describeCall()` maps contour, note count, rhythm, and register to prose:

| Realized shape | Caption |
|---|---|
| 3 notes, ascending, soft | `a soft three-note rise` |
| trill, rest, trill, low register | `a low trill, paused, low trill again` |
| 1 note, sharp attack, back perch | `a single sharp call from the back perch` |

Because captions derive from the actual phrase, they vary exactly as much as the calls do — which means a caption user experiences the same non-repetition as a listening user. A fixed caption per motif would have quietly reintroduced the looped-audio problem in text.

---

## 11. Accessibility surfaces

The stance from the PRD is that accessible users get the actual product, not a stripped variant, and that this work ships with v1 rather than after it. The engineering consequence is that accessibility is not a workstream that runs alongside the renderer — it is a set of parameters *inside* the renderer and a consumer of the same voice system. There is no "accessible mode" build.

### 11.1 Screen-reader surface structure

```html
<header>  <!-- top bar: 5 items, real buttons, never aria-hidden even when faded -->
<main aria-label="the aviary">
  <div id="aviary-narration" aria-live="polite" aria-atomic="true"></div>   <!-- idle prose -->
  <div id="aviary-events"    aria-live="polite" aria-atomic="true"></div>   <!-- priority prose -->
  <ul role="list" aria-label="birds">
    <li>
      <button aria-label="listen in on pip" aria-describedby="d-pip" aria-pressed="false"></button>
      <span id="d-pip" hidden>pip is on the front rail, calling softly.</span>
    </li>
  </ul>
</main>
```

Decisions worth stating:

- **Two polite regions, no assertive region.** Priority events go into a second polite region so they jump the idle queue without interrupting the reader mid-sentence. `aria-live="assertive"` on an ambient product is rude by construction.
- **Bird descriptions are prose, refreshed on the narration cadence.** Not "perch: front, mood: content." A screen-reader user reading a bird's description gets the same specificity a sighted user gets from watching it.
- **Mood is never exposed as a label.** No `aria-label="mood: wary"`, no state text. Mood reaches the screen-reader user the same way it reaches everyone else — through description of what the bird is doing. "wren sits low with her feathers fluffed" is the drowsy read; naming the state would be the tooltip the PRD forbids, relocated into the accessibility tree.
- **Personality values are never in the accessibility tree.** The client does not have them (§5.2), so this is guaranteed rather than remembered.
- **`aria-pressed`** carries listen-in state, because that state is genuinely a toggle and a screen-reader user needs to know whether they are currently listening in.

### 11.2 Keyboard navigation

| Key | Action |
|---|---|
| `Tab` | Through the five top-bar items, then into the aviary |
| `Tab` (into aviary) | Focuses the first bird (roving tabindex over the bird list) |
| `←` `→` | Move focus between birds by scene position |
| `Enter` / `Space` | Listen in on the focused bird; again to disengage |
| `Esc` | Exit listen-in; if not listening in, return focus to the top bar |
| `Shift+Tab` | Back to the top bar |

The offer panel, notebook, settings, and invite flow are ordinary focus-trapped dialogs with `Esc` to close and focus restoration on exit. Settle is a top-bar button; its 5-second undo is reachable by any key, matching the "any click anywhere" behavior.

**Focus indicators** are a dual-stroke ring: a 2px dark inner stroke and a 2px light outer stroke, so at least one edge holds ≥3:1 contrast against every aviary state from midday sky to full night. A single-color ring cannot do this across a scene whose luminance changes by design over the day. The ring appears only for keyboard focus (`:focus-visible`), so it is not chrome sitting in the scene during pointer use.

### 11.3 Reduced-motion mode

Triggered by `prefers-reduced-motion: reduce` or the explicit setting, and re-evaluated live if the OS preference changes mid-session.

It is implemented as a `motionProfile` **parameter to the renderer**, not a separate path:

| Element | Full | Reduced |
|---|---|---|
| Idle micro-motion | continuous noise-driven pose | slow cross-fades between held poses, 600–900ms, same pose basis |
| Perch change | flight arc with wing-beats | cross-dissolve between positions, 900ms |
| Leaf/feather drift | present | removed |
| Parallax | active | fixed layers |
| Day/night palette shift | continuous | continuous, at half rate |
| Weather | animated overlay | slow opacity fade only |
| Top bar fade | 400ms | 150ms (shortened, not removed) |
| Calls, drift, mood, notebook | unchanged | **unchanged** |

A parameter rather than a branch is the difference between reduced-motion staying correct and rotting after three sprints of feature work on the primary path. It also makes the mode testable in the same screenshot suite as everything else.

The design intent from the PRD is that this is its own quiet aesthetic — a Pocket Aviary that is calmer and slower, not one that looks broken. The cross-fade poses are drawn deliberately (a held preen pose is composed, not a frame plucked from an animation), and the mode gets its own design review and its own QA pass, not a smoke test.

### 11.4 Captions

Opt-in from accessibility settings; default `auto`, which resolves to on when WebAudio is unavailable and off otherwise (§9.7 covers why autoplay-blocked does not count as unavailable).

Rendering: small text near the calling bird, offset so it never overlaps the bird's silhouette, on a subtle scrim that guarantees ≥4.5:1 against every aviary state. Fades in with the call and out ~1.2s after it ends. Multiple simultaneous captions stack vertically by perch depth and cap at three; beyond that the newest replaces the oldest. In reduced motion, captions fade only (no movement).

Captions are the one text element that appears inside the aviary scene. It is a bounded exception, conditional on a user setting, in the naturalist voice, and it exists because the alternative is rationing the product's core sensory content by hearing.

### 11.5 Contrast and other surfaces

All user copy — top bar, settings, account, error surfaces, captions, visible narration — passes WCAG AA (4.5:1 body, 3:1 large). Contrast is asserted in CI by computing ratios from the design tokens against the extreme aviary background states (brightest midday, darkest night), not against a mid-tone mock. Focus rings are checked at 3:1 against the same extremes.

Also covered: `prefers-contrast: more` raises the scrim opacity and the focus-ring weight; `prefers-reduced-transparency` makes the top bar's faded state 40% rather than 8%; all interactive targets are ≥44×44 CSS px (bird hit targets are generous ellipses larger than the bird's visual silhouette, which also makes listen-in pleasant with a trackpad).

The visitor experience includes captions, narration, keyboard navigation, and reduced motion. Accessibility does not stop at the account boundary.

---

## 12. Performance budgets and observability

### 12.1 Budgets

| Budget | Value | Enforcement |
|---|---|---|
| Initial JS, first paint | ≤2MB gz (PRD ceiling); **180KB gz working target** for the first-bird path | `size-limit` per entry chunk in CI; PR fails on regression >2% |
| Lazy surfaces combined | ≤700KB gz | Same |
| Time to first bird | <500ms, mid-tier Android over throttled 4G | Synthetic run per merge on a real device in the lab; p75 across 20 runs |
| Idle frame rate | 60fps on a 5-year-old mid-range laptop, **sustained 30 min** | 30-minute soak, p95 frame ≤16.7ms, p99 ≤25ms |
| Memory growth | none over 30 min | Heap sampled every 60s; fail if retained heap grows >3MB or >5% after settling |
| Simulation tick latency | p99 < 5s (alarm threshold) | Production alarm; the tick's actual target is <50ms/account-batch |
| Snapshot response | p99 < 120ms | Production alarm |

Frame-rate and memory soaks run nightly on real hardware, not in a container. A 30-minute soak in CI on every PR is impractical; the PR gate runs a 3-minute version and the nightly runs the full one, with a hard revert policy if the nightly fails.

### 12.2 Where the memory rule bites

"No memory growth over 30 minutes" is stated in the PRD as a real CI test, and it constrains four specific places:

- **Audio.** Pooled worklet voices, notes as messages (§9.2). No node allocation in steady state.
- **Notebook.** Virtualized list; entries scrolled out of view drop their DOM and their parsed representation. The infinite backscroll must not accumulate.
- **Renderer.** Ornament particles (leaves, feathers) come from a fixed-size ring buffer. Pose blends write into pre-allocated typed arrays. No per-frame object allocation in the hot path — a rule the frame-loop code is reviewed against, since GC pauses are also how the 60fps p99 is lost.
- **Event outbox.** Bounded at 500 entries / 24h (§7.2).

### 12.3 What we measure

**Synthetic:** a fleet of automated browsers running the aviary hourly from four geographies, measuring load timings, first-bird time, frame timing over a 5-minute session, and audio-context init success. Synthetic monitoring carries no privacy load and is the primary signal.

**RUM, aggregate only:** page load timings, first-bird render time, frame-time histograms, quality-tier distribution, audio-context error counts, WebAudio availability rates, snapshot request latency and error rates, client version distribution, browser/OS distribution.

**Server:** request counts, latencies, error rates by endpoint, tick latency, tick backlog depth, event ingest rate, mail delivery outcomes, DB connection saturation.

### 12.4 What we deliberately do not measure

This list is as much a part of the observability design as the one above, and it is enforced at the metric-definition layer rather than by convention.

- No per-account dimension on any metric. Not a hashed account id, not a bucketed cohort, not a "power user" flag.
- No per-bird state in any metric: no mood distribution, no trait values, no drift rates, no perch distribution.
- No interaction-content telemetry: no offer counts, listen-in durations, greeting rates, or notebook read events.
- No session-duration metric with any account dimension — the histogram is population-wide and unkeyed.
- No visit counts, retention cohorts, day-N curves, or any metric whose shape is "how often does this person come back." That metric is one product decision away from becoming a streak, and we do not want to be the team that has it in a dashboard when someone asks.
- No funnel instrumentation inside the aviary.

Enforcement: metrics are declared in a schema file with an allowlist of permitted dimension names. A metric with an undeclared dimension fails the build. The telemetry client has no API that accepts an account id.

Alerting: tick latency p99 > 5s (PRD-specified), snapshot p99 > 120ms, event ingest error rate > 1%, tick backlog > 3 minutes of accounts, mail delivery failure > 2%, audio-context failure rate deviating >3σ from baseline (an early signal of a browser update breaking synthesis), and a daily invariant job that asserts no personality vector decreased anywhere.

---

## 13. Security and privacy engineering

### 13.1 Auth

Session tokens are 256-bit random, stored as SHA-256 hashes, in `HttpOnly; Secure; SameSite=Lax` cookies with a 90-day sliding expiry. CSRF protection is `SameSite=Lax` plus a required custom header on all mutating requests. Magic-link and invite tokens are 256-bit, single-use, hashed at rest, consumed by POST (§5.1). Rate limits are per-email and per-IP with graduated delay.

Sessions are listed in account settings with a coarse device label and last-seen time, individually revocable. Revocation is immediate — the token hash is deleted, and there is no cached authorization anywhere with a longer life than the request.

### 13.2 PII containment

Email exists in two ciphertext columns (§4). Everything else — logs, metrics, traces, error reports, Redis keys, message payloads, shard keys, export filenames — uses the synthetic account UUID. This is enforced by:

- a log formatter that redacts anything matching an email pattern before emission,
- a CI grep over the schema asserting no column named `email` outside the two allowed tables,
- a review rule that any new table taking an account reference takes `account_id uuid`.

The PRD is right that this is trivially cheap at design time and effectively impossible to retrofit; the cost of getting it right is one afternoon, and the cost of getting it wrong is a compliance finding against observability tooling nobody can fully audit.

### 13.3 The telemetry boundary

The privacy commitment — per-bird interaction data drives only that user's own simulation, and is never aggregated for anything — is implemented as an architectural boundary, not a policy:

- The simulation Postgres cluster has no replication, CDC, export, or ETL path into the analytics store. This is a network-level absence, not a disabled job.
- The analytics role holds no grant on any simulation table.
- The telemetry client library has no API accepting an account id, bird id, mood, trait, or event kind (§12.4).
- Any future ML work is prohibited from receiving per-bird fields; there is no dataset to receive.

**Calibration is the one place this needs care (J8).** "Measurable drift after a week" is a claim about instruments, and instrumenting it across the user population would be exactly the aggregation the PRD forbids. So calibration draws on two sources only: synthetic cohorts driven by generated presence traces (§14.3), and a small set of staff accounts flagged `is_calibration = true`, whose owners have explicitly consented, and which are the only accounts from which per-bird state may be read for analysis. A query against `bird_personality` that does not filter on `is_calibration` is blocked outside of `sim` by role grants; the analysis role can read only calibration accounts. This is worth flagging because "we'll just check the drift dashboard" is a natural thing for a team to want in month two, and the answer needs to already exist.

### 13.4 Export and deletion

Export is generated asynchronously into object storage with a 24-hour signed URL that additionally requires an authenticated session (§5.6). Contents per J2 include personality vectors, with a matter-of-fact note; nothing in the product reads an export back.

Deletion: soft immediately (status change, tick paused, snapshots evicted, invites revoked, visitor links dead), recoverable by signing in and confirming. Hard at 30 days, cascading across every table plus object storage and mail logs. A monthly orphan audit asserts no row anywhere references a missing account.

### 13.5 Visitor isolation

Visitor sessions run through a separate route tree, a separate DB role with `SELECT` only and no grant on `interaction_event`, and a snapshot serializer that strips `greeting`. A visitor cannot write an event, cannot be credited presence, and cannot influence the host's drift — not because a filter removes their events but because there is no path that creates one. The host's aviary is rendered exactly as the host would see it; there is no visitor-specific rendering, no prettification, no "show-off" variant, and no code that takes a `for_visitor` flag into the renderer.

### 13.6 Abuse considerations

Invite abuse (using the invite system to email strangers) is limited to 10 outstanding invites per account and 20 invites per account per week, with the invite email clearly identifying the host's email address so a recipient knows who sent it. Presence forgery is bounded by the server-side clamps (§6.2) and buys the forger nothing anyway — there is no score to inflate, which is a pleasant side effect of having refused scores.

---

## 14. Testing and calibration

### 14.1 The general problem

Most of this product's failure modes do not throw. A drift constant 3× too high, a notebook that reads like a log, a greeting that repeats, a bird whose vector reset — all pass a conventional suite. The test strategy below is organized around making those specific failures detectable.

### 14.2 Layers

| Layer | Scope |
|---|---|
| Unit | Drift math, mood chain, perch softmax, call grammar productions, caption mapping, presence clamps, voice lint rules |
| Property | Tick determinism (same inputs → identical state, 10⁴ random traces); eager-vs-catch-up equality; monotonicity under adversarial event sequences; snapshot schema closure |
| Contract | Every response validated against its closed schema; trait-absence assertion; expression-vector non-injectivity |
| Integration | Auth flows including expiry/replay/revocation; event idempotency under duplicate and out-of-order delivery; multi-device coherence; visitor isolation (assert a visitor session cannot produce a row in `interaction_event` under any request) |
| Rendering | Screenshot tests across 12 viewports × 3 perch zones × 6 moods × 2 motion profiles; the no-bird-cropped invariant; first-frame-in-motion assertion |
| Audio | Recognizability separation (§9.1); non-repetition (500 calls → 500 distinct parameter vectors); listen-in floor assertion; chorus loudness bound |
| Accessibility | axe-core on every surface; keyboard traversal of the full map; narration cadence bounds; live-region flooding test; contrast computed against extreme aviary states; manual screen-reader passes on NVDA/JAWS/VoiceOver each milestone |
| Performance | Bundle budgets per PR; nightly 30-minute frame and memory soaks on reference hardware; synthetic first-bird timing on a real mid-tier device |
| Voice | Lint (§10.4); a weekly generated-entry sample reviewed by the writer through beta |

### 14.3 Drift calibration harness

`sim-lab` drives the real tick function — not a reimplementation — over synthetic cohorts:

- **Presence trace generator** producing archetypes: daily-15-minute, weekend-only, twice-daily-5-minute, three-weeks-then-absent, absent-then-returning, one-marathon-then-nothing, background-tab-only (must yield ~zero drift).
- **Simulated years in seconds**, exact because the tick is deterministic and index-driven.
- **Assertions** against the PRD's calibration targets: at one week of regular visits every trait has moved >10× the instrument noise floor; no single session moves any trait more than 15% of a perceptual step; at three weeks at least one behavioral mapping has crossed a threshold in every archetype classed as "regular"; a background-tab-only trace produces < 2% of the drift of an equal-wall-clock attentive trace.
- **Long-tenure checks**: a two-year daily user does not saturate to all-ones (the `(1−v)` term is what prevents this, and the test is what proves it).

The harness produces the drift curve chart the team calibrates `α` against. Any change to `α`, `w_t`, or the input weights requires the chart to be regenerated and reviewed — those five numbers are the product's temperament, and they should not move in a routine PR.

### 14.4 Presence-window calibration

The 4-minute activity window is initial. Calibration uses an aggregate, unkeyed histogram of inter-activity gaps within sessions (permitted: no account dimension, no per-bird content). The target is a window at roughly the 85th percentile of within-session gaps — long enough that watching without moving does not end presence, short enough that an unattended open laptop stops counting within minutes. Expected landing zone: 3–6 minutes. This runs in beta and locks before GA, because changing it after launch changes everyone's drift rate.

### 14.5 Device lab

Reference hardware, all physical:

- **Laptop floor:** 2019 13" MacBook Air (Intel, integrated graphics) and a 2019 Windows laptop with UHD 620. These define the 60fps budget.
- **Mobile floor:** a mid-tier Android (Pixel 6a class) on throttled 4G for the first-bird budget; iPhone SE (2nd gen) for Safari audio behavior.
- **Browsers:** last two majors of Chrome, Safari, Firefox, Edge, per the PRD. Older browsers get a matter-of-fact unsupported surface (§18.4); there is no compatibility path and no polyfill budget.

iOS Safari gets specific attention on audio: autoplay gating, context suspension on backgrounding, and worklet availability. Those three have historically been where a WebAudio product's assumptions break.

### 14.6 Human evaluation

Three things cannot be automated and are scheduled as recurring sessions:

1. **Call recognizability.** Every two weeks during build, listeners spend time with a 7-bird aviary and are then asked to identify birds by call alone. Target ≥80% at 7 birds after two weeks of exposure. If we cannot reach it, the cap comes down — the cap exists to protect recognizability, not the other way around, and shipping 7 birds that blur would be shipping the thing the cap was invented to prevent.
2. **Aliveness review.** A weekly 20-minute session where the team watches a live aviary and writes down anything that read as canned, mechanical, or repeated. This is the only reliable detector for a whole class of regressions.
3. **Voice review.** The writer reviews sampled notebook entries, narration transcripts, and captions weekly. Voice erosion is gradual and invisible to the people writing the code.

---

## 15. Rollout and milestones

### 15.1 The scheduling constraint that shapes everything

The PRD claims visible drift at about three weeks. That claim cannot be validated by an accelerated harness alone — acceleration proves the math, not the felt experience, and the felt experience is the deliverable. It requires real accounts drifting in real time on the shipping engine.

Therefore: **the drift engine, mood model, and expression mappings freeze at the start of M4 and long-soak accounts start the same day.** Everything downstream of that date is renderer, audio, accessibility, and polish work. A team that discovers this constraint in month four ships either an uncalibrated drift function or a delayed launch.

### 15.2 Milestones

| M | Weeks | Deliverable | Exit criteria |
|---|---|---|---|
| **M0** | 1–2 | Foundations: repo, CI, schema + grants, `voice-kit` skeleton, voice lint, telemetry schema linter, device lab standing | Grants provably prevent API-side personality writes; lint fails a deliberately bad string |
| **M1** | 3–6 | Simulation engine: tick, drift, mood, perch, presence crediting, `sim-lab`. Renderer prototype both paths | Determinism + monotonicity + eager/catch-up-equality property tests green; renderer decision gate (J10) resolved with numbers |
| **M2** | 5–9 | Audio: call grammar, worklet synthesis, chorus, listen-in mix. Recognizability harness | Recognizability ≥80% at 7 birds in the first human eval; non-repetition test green; zero audio assets |
| **M3** | 7–11 | Client: scene, birds, idle motion, day/night, weather, top bar, first-frame path, snapshot reconciliation | First-bird <500ms on the mobile floor; first-frame-in-motion test green; no spinner exists in the codebase |
| **M4** | 11 | **Engine freeze. Long-soak accounts begin.** ~30 staff aviaries with varied presence patterns, on the shipping engine | Freeze declared; soak accounts created and instrumented (calibration-flagged) |
| **M5** | 11–15 | Auth, accounts, sync, settings, notebook generation + corpus, export/delete | Multi-device coherence tests green; corpus at 300 frames through copy review; magic-link scanner-safety verified against real corporate mail filters |
| **M6** | 14–18 | Accessibility in full: narration, reduced-motion mode, captions, keyboard map, contrast. Visits end-to-end | axe clean; manual AT passes on three readers; reduced-motion design review signed off; visitor isolation test green |
| **M7** | 18–21 | Closed beta, ~200 invited users. Presence-window calibration. Perf soaks. Voice review at volume | Budgets met on reference hardware; presence window locked; no P1 open |
| **M8** | 21–24 | GA ramp | Soak accounts at ≥10 weeks confirm the three-week claim; alarms wired; runbooks written |

Weeks overlap deliberately; the engine, audio, and client tracks run in parallel after M0.

### 15.3 Team

Six engineers plus part-time specialists: one simulation engineer (Rust, `sim`, calibration), one platform engineer (auth, sync, infra, privacy boundary), two client engineers (one renderer/motion, one interaction/accessibility), one audio engineer (DSP, WebAudio, recognizability), one full-stack (notebook, settings, visits, export). Part-time: visual/motion designer, product writer (owns `voice-kit` corpus and every string), accessibility specialist (embedded from M0, not consulted at M6), QA.

The writer and the accessibility specialist are on from week one on purpose. Both surfaces are called out in the PRD as things that fail when retrofitted, and both are cheap to build in and expensive to bolt on.

### 15.4 Ramp

GA opens by invitation waves: 200 (beta), 2k, 10k, open. The tick is the only capacity question and it is not close to a limit at these numbers; the ramp exists to give the aliveness and voice reviews time to catch things at increasing population diversity, particularly across timezones (day/night correctness), locales, and low-end hardware.

Birds-per-aviary ramps by the age ladder in §6.10, and no launch user will reach day 90 during the ramp — so the arrival flow must be validated on time-shifted accounts before GA rather than discovered live three months later. This is on the M5 test plan explicitly.

### 15.5 Day-one instrumentation

Live from the first GA account: all §12.3 metrics and all §12.4 alarms, plus the daily monotonicity invariant job, plus a synthetic aviary in each supported browser running continuously with a screenshot diff, plus the tick backlog alarm. Nothing user-dimensioned, ever, per §13.3.

### 15.6 Post-launch operating rules

Two standing rules that keep the product from eroding after the PRD's authors are on other things:

1. **Any new user-facing string** goes through the writer and the voice lint. No exceptions for error copy, empty states, or emails.
2. **Any PR touching `α`, trait weights, mood intensities, or the expression mappings** regenerates the calibration chart and requires the simulation owner's review. These constants are the product's temperament, not tuning knobs.

---

## 16. Risks

Ordered by expected damage, not likelihood.

**R1 — Drift calibration lands wrong, and we find out in month three.** Too fast and the product becomes a Tamagotchi where clicking moves a number; too slow and it is a screensaver. The narrow band is real. *Mitigation:* `sim-lab` proves the math; long-soak accounts from M4 prove the feel; the perceptual-step mapping (§6.3) converts an abstract target into something testable. *Residual:* the felt threshold is a human judgment and we have one shot at it before users have weeks of history. If we get it wrong post-launch, we can raise `α` (drift is monotonic, so accelerating is safe and lowering it is not — an important asymmetry to know before we need it).

**R2 — A personality vector is lost or reset.** The worst failure available to this product, and the one most likely to be invisible: no test fails, the user just gradually senses something is off. *Mitigation:* server-only writes enforced by grants; additive deltas; append-only event log; monotonicity trigger; daily invariant job; point-in-time recovery on the simulation cluster with a documented, *rehearsed* restore. *Rehearsed* is the operative word — an untested restore procedure is not a mitigation for the failure we care most about, so the runbook drill is on the M8 exit criteria.

**R3 — Audio uncanniness.** Procedural calls can land in a valley where they sound synthetic rather than alive — and there is no fallback, since recorded audio is unconditionally refused. *Mitigation:* the audio engineer starts at M2 with real time to iterate; the aliveness review listens weekly; the recognizability harness catches the blur case. *Residual:* if calls are recognizable but unpleasant, the fix is craft time, which is why audio starts early and gets the longest runway of any subsystem.

**R4 — Accessibility regresses after M6.** Reduced motion and narration are the classic things that work at launch and rot. *Mitigation:* reduced motion is a renderer parameter, not a path; narration shares `voice-kit` with the notebook, so voice work maintains both; axe and the keyboard map are PR gates; screenshot tests cover both motion profiles. The specialist stays through GA rather than being released at M6.

**R5 — Autoplay policy degrades the first session.** A first-time user on iOS gets a silent aviary until they touch the screen, and audio is a large fraction of the affective payload. *Mitigation:* §9.7 — visual aliveness carries the first moments, audio joins mid-phrase at the first gesture, and no prompt is shown. *Residual:* a user who opens the tab, watches for 40 seconds without touching anything, and leaves has experienced a silent product. We accept this over a modal, because a modal would break the more important thing. Worth measuring in beta as an aggregate "time to first gesture" statistic.

**R6 — The notebook drifts toward event-log prose.** Under time pressure, frames get written quickly and generically, and the surface where the voice is most concentrated becomes the surface that breaks the spell. *Mitigation:* voice lint, closed observation taxonomy, weekly writer review, corpus complete at M5 rather than filled in during beta.

**R7 — Magic links and invite links consumed by scanners.** A meaningful fraction of users would see "this link has been used" on their first attempt, and it would look like a broken product. *Mitigation:* POST-only consumption (§5.1), tested against real corporate mail filters in M5.

**R8 — 60fps on five-year-old hardware over 30 minutes.** The sustained requirement is harder than the peak one; GC pressure and shader-state churn accumulate. *Mitigation:* allocation-free frame loop, ring-buffered ornaments, adaptive quality that never degrades birds, nightly soaks on real hardware.

**R9 — Timezone and DST correctness.** The day/night cycle is anchored to the user's local time; a DST bug shifts the palette an hour or makes a bird roost at noon. *Mitigation:* interpolate on absolute time against computed sunrise/sunset; test across DST transitions in both hemispheres, across a device that changes timezone mid-session, and across `UTC+13`/`UTC−11`.

**R10 — Someone adds a "harmless" engagement surface.** The PRD predicts this precisely, and it is a people problem with a partial engineering answer. *Mitigation:* the mechanisms in §2 — component lint, closed taxonomy, closed schemas, no notification infrastructure, no cross-account aggregation. The point of building the refusals into the type system and the grants is that the next contributor does not need to have read the PRD to be stopped.

**R11 — Simulation cost at scale.** Per-minute ticks across a large population is a recurring cost that grows linearly with signups, unlike most of the stack. *Mitigation:* tiering, change-only writes, batched transactions (§6.1). At 1M accounts this is roughly 5 cores and ~17k row-updates/sec, which is still ordinary; the tier boundaries are the knob if it stops being.

**R12 — Bundle creep.** 2MB is generous now and gone quickly once settings, notebook, invites, and accessibility surfaces land. *Mitigation:* per-chunk budgets in CI, aggressive code splitting, a working target well under the ceiling so the ceiling is never the thing we are negotiating with.

---

## 17. Decisions needing product sign-off

These are resolved in this plan so that engineering is not blocked, but each is a product call and should be confirmed before the milestone that ships it.

| # | Decision | This plan's call | Needed by |
|---|---|---|---|
| J1 | Settle in the top bar makes it five items, against `aviary_layout.md`'s "nothing else" | Ship five | M3 |
| J2 | Personality vectors in the account export | Include, with the three guards in §5.6; the alternative costs the user nothing | M5 |
| J3 | Touch events count as presence activity | Yes — without it, mobile presence is structurally near-zero | M1 |
| J4 | Silent first session under autoplay blocking, with no prompt | Accept the silence; refuse the prompt | M3 |
| J12 | Bird arrival is quiet, discovered by looking, with an in-voice acceptance on focus | Ship as described | M5 |
| — | Presence window final value | Locks at 3–6 min from beta data (§14.4) | M7 |
| — | Bird cap if recognizability testing fails at 7 | Lower the cap; do not ship a blurred chorus | M2 |

One external dependency, not a decision: the **design system spec** (palette values, contrast ratios per surface, focus-ring treatment, reduced-motion pose art) lives with the visual designer and is needed by **start of M3**. The renderer can be built against placeholder tokens, but the contrast CI gate and the reduced-motion pose set cannot be finished without it.

---

## 18. Appendices

### 18.1 Species pool

Six species, a coherent temperate garden/woodland set. Each carries a silhouette, a default palette, a motif library, and behavioral biases. Biases shift the *seed* distribution of the personality vector at adoption; they do not constrain where drift can go, so a timid species can become a bold individual over months.

| Species | Silhouette | Call character | Seed bias |
|---|---|---|---|
| finch | small, round, short conical beak | bright two-note "pip," quick | +boldness, +vocal |
| wren | tiny, upright tail, fine beak | rapid low trill, bursty | +curiosity, −boldness |
| warbler | slim, fine beak, long wing | soft three-note rise | +vocal, +warmth |
| thrush | mid-size, upright, spotted breast | fluting, spaced phrases | +warmth, −vocal |
| titmouse | small, crested, round | two-note descending, head-tilty | +curiosity |
| nightjar | long-winged, flat-headed, cryptic | churring, low, sustained | −vocal by day, **active at night** |

The nightjar satisfies the PRD's requirement that night not be a dead state: at full night most birds are `roosting`, and the nightjar may call into the late hours. Starter pairs are drawn to avoid two of the same species and to avoid pairing two low-vocal species (which would make a first session too quiet to read).

### 18.2 Mood reference

| Mood | Idle read | Perch bias | Call rate | Enters from | Notes |
|---|---|---|---|---|---|
| `alert` | upright, frequent scans, still body | any | ×1.1 | dawn, wind, alarm nearby | peaks in early morning |
| `curious` | head-tilts toward sound, tracks leaves | forward | ×1.2 | offers, new sounds, high curiosity | most responsive to offers |
| `content` | preens, relaxed posture, small shuffles | forward/middle | ×1.0 | accepted offers, calm weather | the baseline daytime state |
| `wary` | back-leaning, high vigilance, few calls | back | ×0.5 | alarm calls, wind, neighbor contagion | damped by boldness |
| `drowsy` | low on perch, feathers fluffed, slow blinks | middle/back | ×0.4 | dusk, settle, rain | precedes roosting |
| `roosting` | eyes closed, low, minimal motion | back | ×0.05 | full night | nightjar exempt |

### 18.3 Naturalist copy samples (product surface)

Notebook:
> tuesday — pip greeted before wren today, first time this week.

> a long stretch of quiet this morning. pip preened for several minutes without looking up.

> rain came through around midday. the calls thinned out and did not pick back up for a while.

Narration:
> a small grey bird is perched on the front rail, calling softly. another sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Captions:
> a soft three-note rise

> a low trill, paused, low trill again

Arrival acceptance:
> a titmouse has been coming to the back perch.  · let it stay · not now ·

### 18.4 System copy (matter-of-fact surface)

Complete v1 set. Normal capitalization, direct, no naturalist phrasing, no warmth standing in for information.

| Context | Copy |
|---|---|
| Expired/invalid magic link | We couldn't sign you in. The link may have expired. Try requesting a new link. |
| Link already used | This link has already been used. Request a new one to sign in. |
| Session expired | Your session timed out. Sign in again to keep watching. |
| Snapshot load failure (user-initiated retry) | Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch. |
| Offer failed to record | We couldn't record that. Try again in a moment. |
| Visit revoked or expired | This visit is no longer available. |
| Invite send failure | We couldn't send that invitation. Check the address and try again. |
| Email change pending | We sent a verification link to the new address. Your current address keeps working until it's confirmed. |
| Export ready (email) | Your aviary export is ready. The download link works for 24 hours and requires you to be signed in. |
| Deletion pending | Your account is scheduled for deletion in 30 days. Sign in and choose "I changed my mind" to keep it. |
| Unsupported browser | Pocket Aviary needs a current version of Chrome, Safari, Firefox, or Edge. |
| Rate-limited sign-in | Too many sign-in requests. Try again in a few minutes. |

### 18.5 Constant reference

Every tunable in one place, with its owner and its calibration source.

| Constant | Value | Source |
|---|---|---|
| `TICK_SECONDS` | 60 | PRD (~once per minute) |
| `COLD_TICK_SECONDS` | 900 (replays skipped indices) | §6.1 |
| `ALPHA` (drift master) | 9.0e-6 per presence-second | §6.3 derivation, `sim-lab` |
| `W_TRAIT` | warmth 1.0, plumage 0.9, boldness 0.8, vocal 0.7, curiosity 0.6 | §6.3 |
| `LISTEN_IN_MULT` | 2.5 | §6.3 |
| `OFFER_ACCEPT_EQUIV_S` / `OFFER_PLACE_EQUIV_S` | 60 / 30 | §6.3 |
| `TRAIT_SEED_MID` | 0.35 (±0.08 by species bias) | §6.3 |
| `ACTIVITY_WINDOW_MS` | 240,000 (locks 3–6 min at M7) | §6.2, §14.4 |
| `PING_INTERVAL_MS` | 15,000 | §6.2 |
| `PRESENCE_MINUTE_CAP_S` | 60 | §6.2 |
| `OFFER_COOLDOWN_S` (per bird, per type) | 240 | PRD ("a few minutes"), §6.7 |
| `OFFER_GLOBAL_COOLDOWN_S` | 45 | §6.7 |
| `SETTLE_UNDO_MS` | 5,000 | PRD |
| `SETTLE_DURATION_H` | 8 | §6.8 |
| `GREETING_STAGGER_MS` | U(900, 2600) | PRD (staggered, never unison) |
| `BIRD_CAP` | 7 | PRD; DB-enforced |
| `ARRIVAL_DAYS` | 90, 180, 300, 450, 630 | §6.10 |
| `NOTEBOOK_TARGET_PER_WEEK` / `MIN_GAP_H` | ~2 / 36 | §10.2 |
| `FRAME_REUSE_DAYS` / `FRAME_BIRD_REUSE_DAYS` | 60 / 120 | §10.2 |
| `NARRATION_IDLE_S` | U(30, 60), min 8s between any two | PRD, §10.5 |
| `LISTEN_IN_RAMP_MS` | 1,200 engage / 1,600 release | §9.4 |
| `LISTEN_IN_OTHERS_FLOOR_DB` | −12 (hard floor; never silence) | §9.4 |
| `VOICE_POOL_SIZE` | 8 | §9.2 |
| `WEATHER_EVENTS_PER_WEEK` | ~3, intensity ≤0.5 | PRD ("a few times a week"), §6.8 |
| `MAGIC_LINK_TTL_MIN` | 15 | PRD |
| `INVITE_TTL_DAYS` | 30 | PRD |
| `SOFT_DELETE_DAYS` | 30 | PRD |
| `SNAPSHOT_KEEPALIVE_S` | 25 | §5.2 |
| `OUTBOX_MAX` / `OUTBOX_TTL_H` | 500 / 24 | §7.2 |
| `PRESENCE_REPLAY_MAX_S` | 120 (older pings discarded, never credited) | §7.2 |

---

*End of plan.*

