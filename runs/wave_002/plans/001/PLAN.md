# Pocket Aviary — v1 Implementation Plan

**Status:** phase-1 plan, ready for engineering execution
**Source:** `prd/` (product_brief, concepts, bird_engine, interactions, aviary_layout, accounts_sync, social_optional, accessibility_perf, non_goals)
**Deliverable of this document:** the plan. Not the product.

---

## 0. How to read this plan

The PRD is unusual in that most of its hard requirements are *affective* — "feels alive," "notice never announce," "charm comes from specificity." Affective requirements rot silently: nothing fails, nothing alarms, the product just gradually becomes a different product. A plan that restates them as prose and hopes engineers remember has already lost.

So the organizing principle here is: **every affective rule in the PRD is translated into a structural or automated constraint.** Not a guideline in a wiki. A missing design-system primitive, a revoked database grant, a serializer field allowlist, a CI-failing property test. Section 12 collects these as a single suite; they are also called out inline where they belong.

Three examples of what that means concretely, to set expectations:

- "The personality vector is never exposed numerically" becomes: personality columns are never included in any API serializer, enforced by a field allowlist with a contract test; traits reach the client only pre-resolved into rendering decisions (a perch zone, a palette step, a call plan).
- "Personality drift is monotonic toward expressive" becomes: a drift function whose increment is non-negative by construction, *plus* a Postgres trigger rejecting any decrease, *plus* a property test over randomized event streams. Three layers, because losing this is a product-level failure that no user would report as a bug.
- "No 'Welcome back!' toast" becomes: **the design system contains no toast, banner, badge, spinner, or confetti primitive at all.** You cannot add one in a Friday-afternoon PR because there is nothing to import.

Section 15 lists every judgment call this plan makes where the PRD was silent or ambiguous, so a reviewer can overturn them individually without re-deriving the plan.

---

## 1. Scope

### 1.1 In scope for v1

**Accounts and identity**
- Single-user accounts, one aviary per account
- Email + magic-link sign-in (15-minute expiry, single-use)
- Per-device session tokens, listed and revocable from account settings
- Email change with verification of the new address before cutover
- JSON account export, generated on demand, delivered as an emailed download link
- Soft account deletion with a 30-day recovery window, then hard deletion

**The aviary**
- Two starter birds at adoption (system-selected species, user-named), cap of seven
- Single horizontal scene, one screen, no pan/scroll/zoom
- Three perch zones (front / middle / back), bird-chosen, never user-arranged
- Local-time day/night cycle, continuous
- Rare ambient weather (short rain, soft wind) with small mood effects
- Ambient micro-motion: leaf and feather drift, subtle parallax
- Thin top bar with exactly four affordances: account/settings, accessibility settings, field notebook, offer

**The bird engine**
- Five-trait hidden personality vector per bird, server-persisted, server-authored
- Monotonic drift toward expressive, driven primarily by presence-time
- Five-state mood system on a fast timescale, persisting across sessions
- Procedural call grammar with per-bird stable signatures, synthesized client-side
- Mood-shaped idle micro-motion (noise-driven pose walk, not animation clips)
- Bird-to-bird interaction: call-and-response, mood contagion, emergent chorus
- Six-species pool, one nocturnal
- Stable per-bird identity, invariant across rename / sync / migration
- Age-gated third-bird-and-beyond offers

**Interactions**
- Return-greeting: one bird, varying by absence length, boldness, and mood; procedurally generated
- Listen-in: focus a bird, gradual mix rebalance, others quiet but never silent
- Offer: seed, song fragment, still pool — from the top bar, with per-bird cooldown
- Settle: soft evening shift with a five-second undo
- Field notebook: auto-generated, sparse, read-only, naturalist prose, infinite scrollback
- Presence accounting on the strict three-signal conjunction

**Sync**
- Server-side simulation tick (~60s), running independent of client connections
- Snapshot-pull clients with interpolation; append-only client event log
- Multi-device coherence as an architectural property, not a feature

**Social**
- Visit invitations: per-invite, email-addressed, read-only, revocable, 30-day expiry, off by default
- Visit log in account settings
- Optional per-account visit-notification toggle, default off

**Accessibility (ships with v1, not after)**
- Screen-reader narration in naturalist prose, slow cadence
- Reduced-motion mode as a designed cross-fade register
- Call captions generated from the live call grammar
- Full keyboard navigation, visible focus against all scene states
- WCAG AA contrast on all user copy

**Performance**
- <2MB gzipped initial JS bundle
- <500ms to first bird visible on mid-tier mobile over 4G
- 60fps idle motion on a five-year-old mid-range laptop, sustained over 30 minutes
- Zero memory growth over a 30-minute session, verified in CI
- Aggregate-only observability

### 1.2 Explicitly out of scope for v1

From `non_goals.md` and `product_brief.md`, treated as build-time prohibitions rather than backlog items:

- **Native apps.** No iOS, no Android. We also do not shape the data model or protocols around hypothetical native-client constraints; the API is designed for the web client we are building.
- **Gamification, in every form.** No achievements, badges, levels, scores, XP, ranks, tiers, streaks, green-dot calendars, "birds adopted: N" counters, milestone celebrations, or visit-frequency surfacing — not as a setting, not as an opt-in, not as a "harmless" one-off. The notebook may observe the aviary; it may never observe the user's behavior.
- **Tamagotchi mechanics.** No death, hunger, distress, decay meters, or negative drift. Neglect produces ambient quietness only.
- **Social-network surfaces.** No profiles, follows, feeds, discovery, chat, avatars, comments, mutual visits, co-presence, leaderboards, or show-off rendering. We additionally do not compute the cross-account metrics that a leaderboard would need, so the feature cannot be "just exposed" later.
- **Notifications.** No push, no email about the aviary, no re-engagement mail. The single exception is the opt-in, default-off visit notification, which is user-requested and per-account.
- **Payments, shared aviaries, multi-aviary accounts, customizable scenes, species catalogs, rarity.**
- **Recorded audio, at any quality, under any fallback path.**
- **Any user-facing surface that displays a personality trait value**, including internal debug builds shipped to users. (Staff-only tooling is covered in §12.4.)

### 1.3 Scope boundaries worth naming

Three things sit near the line and are decided here:

- **The visual design system** (exact palette hex values, per-surface contrast ratios, focus-ring treatment, silhouette artwork) lives with the visual designer in a separate spec, per `aviary_layout.md`. This plan specifies the *slots* that spec fills and the automated checks that validate whatever it produces.
- **Copy** for system surfaces (auth, errors, settings) is written by engineering against the matter-of-fact voice sample and reviewed by the voice owner. Naturalist copy (notebook templates, narration templates, caption phrases) is authored by the voice owner as a versioned content package, because it is the product's charm engine and cannot be a side effect of implementation.
- **Bird artwork** is procedurally generated geometry with a designer-authored parameter set per species, not hand-drawn sprite sheets. This is forced by the 2MB budget and is called out in §8.3.

---

## 2. Architecture

### 2.1 Shape

Five deployable units, deliberately boring:

```
                      ┌──────────────────────────────┐
   browser ──HTML──►  │  edge (CDN + SSR shell)      │
                      │  · HTML + inline critical CSS│
                      │  · inlined snapshot (≤30s TTL)│
                      └──────────────┬───────────────┘
                                     │
   browser ──JSON──►  ┌──────────────▼───────────────┐      ┌───────────────┐
                      │  aviary-api  (stateless)     │◄────►│  Redis        │
                      │  · auth / sessions           │      │  · snapshot   │
                      │  · snapshot read             │      │    cache      │
                      │  · event ingest (append-only)│      │  · rate limit │
                      │  · notebook, settings, visits│      └───────────────┘
                      └──────────────┬───────────────┘
                                     │  (READ personality; NO WRITE grant)
                      ┌──────────────▼───────────────┐
                      │  Postgres (canonical)        │
                      │  · accounts, birds, vectors  │
                      │  · event log (partitioned)   │
                      │  · notebook, invites         │
                      └──────────────▲───────────────┘
                                     │  (SOLE WRITER of personality)
                      ┌──────────────┴───────────────┐
                      │  sim-tick  (sharded workers) │
                      │  · 60s cadence per aviary    │
                      │  · drift, mood, perch, calls │
                      │  · notebook generation       │
                      └──────────────────────────────┘

                      ┌──────────────────────────────┐
                      │  mailer (magic links,        │
                      │  invites, export links)      │
                      └──────────────────────────────┘
```

**Stack:** TypeScript end to end. Node (Fastify) for `aviary-api` and `sim-tick`; Postgres 16; Redis 7; the edge tier on any CDN with an SSR-at-edge runtime. No Kafka, no event-streaming platform — the "event log" is a partitioned Postgres table, which is sufficient at our write rate (a presence ping per 20s per active session) and avoids introducing a system whose partition keys are exactly the place PII leaks in the failure mode `accounts_sync.md` warns about.

**The one architecturally load-bearing decision** is the separation of `aviary-api` from `sim-tick` at the *database grant* level, not just the code level. See §6.1.

### 2.2 Client/server split

The split is stated once and holds everywhere:

| Concern | Owner |
|---|---|
| Personality vectors | Server, exclusively |
| Mood state and mood timers | Server, exclusively |
| Perch assignment | Server, exclusively |
| Call plan (who calls, when, which motifs, expressive params) | Server generates plan; client synthesizes audio |
| Weather state and day-phase | Server (deterministic from seed + local time) |
| Notebook entries | Server |
| Interaction events (offer, listen-in, settle, presence) | Client emits; server is the only interpreter |
| Interpolation between snapshots | Client |
| Idle micro-motion pose sampling | Client (unsimulated ornament) |
| Ambient leaves and feathers | Client (unsimulated ornament) |
| Narration prose composition | Client, from a shared grammar package |
| Caption text | Client, from the same call plan the synth reads |

The rule that generates this table: **anything that must survive a device switch or an absence is server state; anything that is a rendering of that state at 60fps is client state.** A leaf drifting through frame does not need to be the same leaf on the phone as on the laptop, and making it so would cost a per-leaf simulation record for zero user-visible benefit — `aviary_layout.md` says exactly this, and it generalizes.

### 2.3 Render pipeline boundary

The boundary between "simulation" and "rendering" sits at the **call plan and the pose target**, not at the pixel.

The server does not say "draw Pip at x=340, frame 12 of preen_cycle." It says: *Pip is on the front perch, mood content, her next three calls are at t+4.2s / t+31.8s / t+66.1s with these motif sequences and these expressive parameters, and her motion phase offset is 0.63.* The client turns that into continuous motion.

This boundary is what makes the "first frame is mid-action" rule mechanically true rather than a stylistic aspiration. The snapshot carries a **motion phase offset** per bird — a scalar into the client's pose-noise function — so the client's very first drawn frame samples the pose walk at a non-zero phase. There is no pose index 0, no clip start, no "begin animation" call anywhere in the renderer. Section 8.4 details this.

---

## 3. Data model

Postgres. All identifiers are UUIDv7 (time-sortable, no natural-key leakage). Every table that references an account does so by `account_id UUID`, never by email — see §3.6.

### 3.1 Accounts and auth

```sql
CREATE TABLE account (
  id                 UUID PRIMARY KEY,           -- synthetic; the ONLY account identifier
  email_ciphertext   BYTEA NOT NULL,             -- AES-GCM, envelope-encrypted via KMS
  email_blind_index  BYTEA NOT NULL UNIQUE,      -- HMAC-SHA256(email_normalized, server pepper)
  email_verified_at  TIMESTAMPTZ,
  pending_email_ct   BYTEA,                      -- set during email change, until verified
  tz_iana            TEXT NOT NULL DEFAULT 'UTC',
  created_at         TIMESTAMPTZ NOT NULL,
  settings           JSONB NOT NULL DEFAULT '{}',-- a11y prefs, visit-notify toggle, audio prefs
  soft_deleted_at    TIMESTAMPTZ,
  hard_delete_after  TIMESTAMPTZ                 -- soft_deleted_at + 30d
);

CREATE TABLE device_session (
  id             UUID PRIMARY KEY,
  account_id     UUID NOT NULL REFERENCES account(id),
  created_at     TIMESTAMPTZ NOT NULL,
  last_seen_at   TIMESTAMPTZ NOT NULL,
  revoked_at     TIMESTAMPTZ,
  ua_family      TEXT,       -- "Safari on iPhone" — for the user's session list, not fingerprinting
  approx_region  TEXT        -- country-level only
);

CREATE TABLE magic_link (
  token_hash   BYTEA PRIMARY KEY,                -- SHA-256 of the token; raw token never stored
  account_id   UUID NOT NULL REFERENCES account(id),
  purpose      TEXT NOT NULL,                    -- 'signin' | 'email_change' | 'export'
  expires_at   TIMESTAMPTZ NOT NULL,             -- issued_at + 15 minutes
  consumed_at  TIMESTAMPTZ
);
```

`email_blind_index` deserves a note, because it is the one place this design lets email influence an identifier. It exists so sign-in can find an account without decrypting every row, and so per-email rate limiting works. It is deliberately *not* a general-purpose key: it appears in exactly two queries (sign-in lookup, rate-limit bucket), it is never logged, never emitted in telemetry, never used as a partition or shard key, and it is HMAC'd with a pepper held only by `aviary-api`. A lint rule (§12.3) fails the build if `email_blind_index` appears in any file outside `auth/`.

### 3.2 The aviary and its birds

```sql
CREATE TABLE aviary (
  id                UUID PRIMARY KEY,
  account_id        UUID NOT NULL UNIQUE REFERENCES account(id),
  created_at        TIMESTAMPTZ NOT NULL,   -- drives bird-offer pacing; NOT a "days active" counter
  weather_seed      BIGINT NOT NULL,
  last_tick_index   BIGINT NOT NULL DEFAULT 0,
  last_tick_at      TIMESTAMPTZ NOT NULL,
  settled_until     TIMESTAMPTZ,            -- non-null while the aviary is in the settled state
  next_offer_at     TIMESTAMPTZ             -- next species-offer eligibility (age-derived)
);

CREATE TABLE bird (
  id                 UUID PRIMARY KEY,      -- STABLE FOR THE LIFE OF THE ACCOUNT
  aviary_id          UUID NOT NULL REFERENCES aviary(id),
  species_id         TEXT NOT NULL REFERENCES species(id),
  name               TEXT NOT NULL,
  adopted_at         TIMESTAMPTZ NOT NULL,

  -- personality vector: slow timescale. Server-written only. Never serialized to any client.
  boldness           REAL NOT NULL CHECK (boldness           BETWEEN 0 AND 1),
  social_warmth      REAL NOT NULL CHECK (social_warmth      BETWEEN 0 AND 1),
  vocal_frequency    REAL NOT NULL CHECK (vocal_frequency    BETWEEN 0 AND 1),
  plumage_saturation REAL NOT NULL CHECK (plumage_saturation BETWEEN 0 AND 1),
  curiosity          REAL NOT NULL CHECK (curiosity          BETWEEN 0 AND 1),

  -- mood: fast timescale
  mood               TEXT NOT NULL,         -- wary|content|curious|drowsy|alert
  mood_since         TIMESTAMPTZ NOT NULL,
  mood_floor_until   TIMESTAMPTZ NOT NULL,  -- minimum dwell; prevents flicker

  perch_zone         SMALLINT NOT NULL,     -- 0 front, 1 middle, 2 back
  perch_slot         SMALLINT NOT NULL,     -- horizontal slot within the zone

  call_seed          BIGINT NOT NULL,       -- fixes the immutable call signature at adoption
  motion_phase       REAL NOT NULL,         -- phase offset into the client pose-noise walk
  last_greeted_at    TIMESTAMPTZ,
  offer_cooldowns    JSONB NOT NULL DEFAULT '{}'  -- {seed: ts, song: ts, pool: ts}
);
```

Two structural guarantees on this table:

**Monotonicity trigger.** A `BEFORE UPDATE` trigger rejects any statement where a trait column decreases. It is not conditional on an application flag; it is a database-level invariant.

```sql
CREATE FUNCTION assert_monotonic_drift() RETURNS trigger AS $$
BEGIN
  IF NEW.boldness           < OLD.boldness
  OR NEW.social_warmth      < OLD.social_warmth
  OR NEW.vocal_frequency    < OLD.vocal_frequency
  OR NEW.plumage_saturation < OLD.plumage_saturation
  OR NEW.curiosity          < OLD.curiosity THEN
    RAISE EXCEPTION 'drift monotonicity violation on bird %', NEW.id;
  END IF;
  RETURN NEW;
END; $$ LANGUAGE plpgsql;
```

Any exception here pages, immediately, as a product-integrity alarm (§11.4) — it means a code path exists that would have quietly taken something away from a user's bird.

**Identity permanence.** `bird.id` has no update path in any migration, service, or admin tool. Species-pool changes alter `species` rows and rendering parameters; they never reassign a bird. Renaming writes `name` only. A migration-review checklist item requires any PR touching `bird` to state explicitly why `id` is untouched.

### 3.3 Drift audit trail

```sql
CREATE TABLE personality_delta (
  id           BIGSERIAL PRIMARY KEY,
  bird_id      UUID NOT NULL REFERENCES bird(id),
  tick_index   BIGINT NOT NULL,
  d_boldness   REAL NOT NULL,  d_social_warmth   REAL NOT NULL,
  d_vocal      REAL NOT NULL,  d_plumage         REAL NOT NULL,
  d_curiosity  REAL NOT NULL,
  presence_s   REAL NOT NULL,  listen_s          REAL NOT NULL,
  offers       SMALLINT NOT NULL,
  applied_at   TIMESTAMPTZ NOT NULL,
  UNIQUE (bird_id, tick_index)
);
```

This is an **audit record, not a source of truth.** `bird_engine.md` is explicit that the vector is stored and never recomputed from logs, and that rule holds: nothing reads `personality_delta` to derive current state. It exists so that (a) a drift-loss incident is detectable and forensically reconstructable, and (b) a one-time corrective transform is possible if launch calibration proves wrong (§13.1). The `UNIQUE (bird_id, tick_index)` constraint is also the exactly-once guard for the tick.

### 3.4 Events and presence

```sql
CREATE TABLE interaction_event (
  id                 BIGSERIAL,
  account_id         UUID NOT NULL,
  aviary_id          UUID NOT NULL,
  bird_id            UUID,
  kind               TEXT NOT NULL,   -- presence_ping | listen_in_start | listen_in_end
                                      -- | offer | settle | settle_undo | session_start
  payload            JSONB NOT NULL DEFAULT '{}',
  client_event_id    UUID NOT NULL,   -- idempotency key, client-generated
  client_ts          TIMESTAMPTZ,     -- ADVISORY ONLY
  server_ts          TIMESTAMPTZ NOT NULL DEFAULT now(),
  consumed_by_tick   BIGINT,
  PRIMARY KEY (server_ts, id)
) PARTITION BY RANGE (server_ts);
-- daily partitions; raw partitions dropped at 90 days (deltas are already folded in)

CREATE UNIQUE INDEX ON interaction_event (account_id, client_event_id);

CREATE TABLE presence_window (
  id                 UUID PRIMARY KEY,
  account_id         UUID NOT NULL,
  device_session_id  UUID NOT NULL,
  started_at         TIMESTAMPTZ NOT NULL,
  ended_at           TIMESTAMPTZ,
  terminal_reason    TEXT           -- settle | tab_close | ping_timeout | signout
);
```

`terminal_reason` is recorded for operational debugging only. It never influences drift: `interactions.md` is explicit that settle and tab-close are equivalent at the engine level, and §5.5 encodes that as a test.

Raw-event retention at 90 days is a privacy-positive default — the folded deltas and the canonical vector carry everything the product needs, and holding a year of per-offer records serves nothing the user asked for.

### 3.5 Notebook, species, visits

```sql
CREATE TABLE notebook_entry (
  id            UUID PRIMARY KEY,
  aviary_id     UUID NOT NULL REFERENCES aviary(id),
  occurred_on   DATE NOT NULL,        -- the user's local date
  created_at    TIMESTAMPTZ NOT NULL,
  text          TEXT NOT NULL,        -- rendered naturalist prose, stored rendered
  template_id   TEXT NOT NULL,        -- for sparsity/repetition control, never shown
  subject_ids   UUID[] NOT NULL       -- birds referenced
);

CREATE TABLE species (
  id              TEXT PRIMARY KEY,
  display_name    TEXT NOT NULL,      -- naturalist descriptor: "a small grey bird"
  silhouette_id   TEXT NOT NULL,
  palette_id      TEXT NOT NULL,
  motif_library   JSONB NOT NULL,
  nocturnal       BOOLEAN NOT NULL DEFAULT FALSE
);

CREATE TABLE visit_invite (
  id                    UUID PRIMARY KEY,
  host_account_id       UUID NOT NULL REFERENCES account(id),
  visitor_email_ct      BYTEA NOT NULL,
  visitor_email_bidx    BYTEA NOT NULL,
  token_hash            BYTEA NOT NULL UNIQUE,
  created_at            TIMESTAMPTZ NOT NULL,
  expires_at            TIMESTAMPTZ NOT NULL,   -- created_at + 30 days
  first_used_at         TIMESTAMPTZ,
  revoked_at            TIMESTAMPTZ
);

CREATE TABLE visit_session (
  id             UUID PRIMARY KEY,
  invite_id      UUID NOT NULL REFERENCES visit_invite(id),
  started_at     TIMESTAMPTZ NOT NULL,
  last_seen_at   TIMESTAMPTZ NOT NULL,
  approx_seconds INTEGER            -- rounded to the nearest minute for the host's log
);
```

Note the absence: there is no `visit_event` table, no visitor presence table, no visitor interaction path. `social_optional.md` requires that visitor attention never drifts the host's birds; the strongest way to guarantee that is to have no schema in which visitor attention could be recorded as a drift input. Section 7.3 completes this at the API layer.

Notebook text is stored **rendered**, not as a template reference. If we later revise the notebook grammar, existing entries must not silently rewrite themselves — the notebook is an observer's record, and a record that changes retroactively is not a record.

### 3.6 The synthetic-ID rule, operationalized

`accounts_sync.md` calls this the single most important boring detail. It is enforced at four points, not one:

1. **Schema**: no table has an email column except `account` (ciphertext + blind index) and `visit_invite` (same pair). Enforced by a schema test that scans `information_schema.columns` for names matching `email` and fails on any table outside that allowlist.
2. **Logging**: the structured logger's serializer has a redaction pass; a CI test emits a log line for every log call site reachable from a fixture request and greps the output for `@` in any value. Any hit fails the build.
3. **Telemetry**: metric definitions are declared in a typed registry; the registry type forbids any dimension not in an enumerated safe set (§11.3).
4. **Keys**: shard, partition, cache, and rate-limit keys are constructed only through a `keys.ts` module whose functions accept `AccountId` (a branded type). An email string cannot be passed without an explicit, reviewable cast.

Retrofitting any of these is the failure mode the PRD names. All four land in the first infrastructure milestone, before any product surface is built.

---

## 4. API surface

REST/JSON over HTTP/2. Session cookie (`HttpOnly`, `Secure`, `SameSite=Lax`), CSRF token on mutating routes. Polling, not WebSockets — the tick cadence is 60s and the client's pull triggers are event-driven and sparse (§6.3); a persistent socket per open tab would cost connection state for no latency the user can perceive.

### 4.1 Auth

```
POST /api/auth/request-link      { email }
  → 202 always, with an identical body and timing profile regardless of whether the
    account exists. No account enumeration. Rate limited per blind-index and per IP.

POST /api/auth/consume           { token }
  → 200 { redirect } + Set-Cookie   | 410 { code: "link_expired" | "link_used" }
    Consumption is a single transaction: SELECT ... FOR UPDATE on magic_link,
    reject if consumed_at IS NOT NULL or expires_at < now(), then set consumed_at.
    Replay is therefore impossible, not merely unlikely.

POST /api/auth/signout           → 204   (revokes this device_session)
GET  /api/account/sessions       → 200 [{ id, ua_family, approx_region, last_seen_at, current }]
DELETE /api/account/sessions/:id → 204
```

Error bodies carry a machine `code`; the client maps codes to matter-of-fact copy (§10.6). The server never sends user-facing prose — a server that emits display strings will eventually emit one in the wrong voice.

### 4.2 Snapshot

```
GET /api/aviary/snapshot?reason=session_start|visibility|keepalive|resume
  → 200 AviarySnapshot   (ETag; 304 on unchanged tick_index)
```

`reason` is functional, not analytics: `session_start` triggers return-greeting computation and absence-length evaluation; `resume` (long render-frame gap) forces a cache bypass because the client's assumptions may be stale by hours.

```jsonc
{
  "tick_index": 184213,
  "server_time": "2026-07-26T14:02:11Z",
  "day_phase": "afternoon",          // dawn|morning|midday|afternoon|evening|dusk|night
  "light": { "warmth": 0.42, "level": 0.86 },   // resolved palette drivers, not raw time
  "weather": { "kind": "none" },     // none|rain|wind, with intensity + ends_at
  "settled": false,
  "birds": [
    {
      "id": "01919e...",
      "name": "pip",
      "species": "warbler",
      "descriptor": "a small grey bird",   // for narration + accessible name
      "perch": { "zone": 0, "slot": 1 },
      "motion": {
        "register": "content",       // mood, expressed as a MOTION REGISTER name
        "phase": 0.63,               // ← the "already in motion" seed
        "energy": 0.55, "restlessness": 0.2
      },
      "plumage_step": 3,             // quantized 0..4, NOT a saturation scalar
      "call_plan": [
        { "at": 4.21, "motifs": ["rise3","hold"], "expr": { "tempo": 1.06, "bright": 0.4, "reps": 1 } },
        { "at": 31.8, "motifs": ["trill","gap","trill"], "expr": { "tempo": 0.92, "bright": 0.55, "reps": 2 } }
      ]
    }
  ],
  "greeting": {                       // present only when reason=session_start and a bird greets
    "bird_id": "01919e...",
    "form": "step_forward_call",      // glance | glance_call | step_forward_call | reorient
    "at": 1.1,
    "call": { "motifs": ["rise2","fall"], "expr": { "tempo": 0.98, "bright": 0.5, "reps": 1 } }
  },
  "horizon_s": 180
}
```

**The serializer is a field allowlist, not an object mapper.** `AviarySnapshot` is constructed by an explicit builder; there is no `toJSON()` on the `Bird` domain object and no spread of a database row into a response. A contract test asserts that the JSON schema of every response, recursively, contains none of the strings `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity`, and that the numeric fields present are exactly the enumerated rendering drivers.

This is the concrete implementation of "the personality vector is never exposed numerically." The client cannot build a stats panel, a debug view, or a third-party inspector view of trait values, because the values are not in the payload in any form. What crosses the wire is *the decision the trait already made*:

| Trait | How it reaches the client |
|---|---|
| Boldness | `perch.zone` — already chosen, plus greeting selection |
| Social warmth | Whether this bird greets, and whether it answers another's call in the plan |
| Vocal frequency | The density of `call_plan` entries |
| Plumage saturation | `plumage_step`, quantized to 5 palette steps |
| Curiosity | The resolved reaction in an offer response |

The quantization of plumage to five steps is deliberate: a continuous value would be a trait scalar wearing a different name, and a determined user could read it off the wire. Five steps over the trait's range means a step change is exactly the granularity of "visible drift the user notices by looking back," which is what §5.2's calibration targets anyway.

### 4.3 Events

```
POST /api/aviary/events
  { events: [ { client_event_id, kind, bird_id?, client_ts, payload } ] }   // ≤ 32 per batch
  → 202 { accepted: [client_event_id...], duplicates: [...] }
```

Append-only. Idempotent on `(account_id, client_event_id)`; a duplicate is acknowledged, not re-inserted. `client_ts` is stored but never trusted for ordering or duration — `server_ts` is authoritative (§6.4).

Rejected at this boundary, with a 400 and no partial write:
- any `kind` not in the enum
- any payload containing a key matching a trait name
- `bird_id` not belonging to the caller's aviary
- an offer whose bird is inside its cooldown (returns `202` with a `cooldown` note, not an error — the client suppresses the affordance anyway, and a hard error here would produce a system-voice message in a naturalist context)

There is no endpoint, under any authentication, by which a client sets a personality value. This is not a validation rule that could be relaxed; there is no route.

### 4.4 Notebook, settings, account

```
GET  /api/notebook?before=<cursor>&limit=30   → 200 { entries: [...], next_cursor }
     Read-only. No POST, PATCH, or DELETE route exists on notebook_entry.
     Cursor pagination, indefinite scrollback, no archiving.

GET  /api/settings                            → 200 { accessibility, audio, visits }
PATCH /api/settings                           → 200

POST /api/account/export                      → 202  (emailed link, 24h expiry, single-use)
POST /api/account/email-change                → 202  (verification to the NEW address)
POST /api/account/delete                      → 202  { recoverable_until }
POST /api/account/restore                     → 200  (any signed-in page, during the 30d window)
```

The export payload contains birds, names, adoption dates, current personality vectors, current moods, notebook entries, and settings — per `accounts_sync.md`. This is the *only* place trait values leave the server, and it is a user-initiated export of their own data to a file, not a product surface. It is a JSON download, not a rendered view; nothing in the product displays its contents. This is a deliberate, narrow exception, and §12.4 records why it does not violate the never-exposed rule: the rule protects the *relationship* from becoming a stat-management exercise, and a JSON file the user asked for is data portability, not a dashboard.

### 4.5 Visits

```
POST   /api/visits/invites        { visitor_email }  → 201 { id, expires_at }
GET    /api/visits/invites                           → 200 [...]
DELETE /api/visits/invites/:id                       → 204   (immediate revocation)
GET    /api/visits/log                               → 200 [{ visitor_email, started_at, approx_minutes }]

GET    /api/visit/:token/snapshot                    → 200 VisitSnapshot
                                                     | 410 { code: "visit_unavailable" }
```

The visitor path is served by a **separate router mounted with a distinct middleware chain** that has no event-ingest routes registered at all, and a database role with `SELECT`-only grants. Revocation and expiry are checked on every snapshot pull, so revocation takes effect at the visitor's next poll (≤60s), matching "terminated at the next state-snapshot pull."

`VisitSnapshot` is produced by the *same* builder as `AviarySnapshot`, minus the `greeting` field. `social_optional.md` forbids show-off rendering, and reusing one builder means there is no code path in which a visitor view could diverge — a second builder would eventually acquire a prettifying tweak.

---

## 5. Simulation engine design

### 5.1 Tick structure

One tick per aviary per 60 seconds. `tick_index = floor((now − aviary.created_at) / 60s)`.

Exactly-once execution is enforced by a compare-and-set inside the same transaction as the state write:

```
BEGIN
  SELECT ... FROM aviary WHERE id = $1 FOR UPDATE
  IF last_tick_index >= target THEN ROLLBACK; RETURN  -- another worker got here
  ... compute ...
  UPDATE bird SET ... ;  INSERT INTO personality_delta ... ;  -- UNIQUE(bird_id,tick_index) backstop
  UPDATE aviary SET last_tick_index = target, last_tick_at = now()
  UPDATE interaction_event SET consumed_by_tick = target WHERE ...
COMMIT
```

Stages, in order:

1. **Load** state and un-consumed events, ordered by `(server_ts, id)`.
2. **Exogenous signals** — local day-phase from `account.tz_iana`; weather from `PRNG(weather_seed, tick_index)`.
3. **Presence fold** — qualified presence-seconds attributable to this tick, deduplicated across devices (§6.5).
4. **Drift** — slow, monotonic (§5.2).
5. **Mood transitions** — fast, hazard-based (§5.3).
6. **Perch selection** (§5.4).
7. **Call plan generation** for the next `horizon_s` (§5.6).
8. **Bird-to-bird propagation** — contagion and call-response scheduling.
9. **Notebook candidate evaluation** (§5.7).
10. **Write** and invalidate the snapshot cache.

**Determinism is a hard requirement of the tick.** All randomness comes from a splittable PRNG seeded by `(aviary.weather_seed, tick_index, stage_id, bird_id)`. No `Math.random()`, no wall-clock reads outside stage 2. This is what makes §5.9 possible and what makes tick behavior reproducible in tests and incident replay.

### 5.2 Drift function

Per trait *k*, per tick:

```
v_k ← v_k + α_k · g_k(inputs) · (1 − v_k)
```

Three properties, each doing work:

- **`g_k ≥ 0` always.** Monotonicity by construction. No input, including a two-week absence, can produce a negative term — an absence contributes `g_k = 0`, which means *no change*, which is exactly `bird_engine.md`'s "becomes ambient, not diminished."
- **`(1 − v_k)` factor.** A one-sided leaky integrator with the leak removed: traits approach 1 asymptotically and never overshoot or run away. It also means early drift is faster than late drift, which matches the felt shape of a relationship deepening.
- **`α_k` is small and configuration-driven**, derived from the calibration targets rather than tuned by feel.

**Inputs per tick**, with weights in the order `bird_engine.md` specifies:

| Input | Symbol | Feeds |
|---|---|---|
| Qualified presence-seconds this tick (aviary-wide) | `P` | all traits, dominant |
| Listen-in seconds on *this* bird | `L` | social_warmth, vocal_frequency (strong) |
| Offer accepted by this bird | `A` | curiosity (small) |
| Offer made while this bird is present | `O` | boldness (small) |
| Settle | — | **nothing**; ends the presence window only |

```
g_boldness   = w1·P̂ + w2·Ô
g_warmth     = w3·P̂ + w4·L̂
g_vocal      = w5·P̂ + w6·L̂
g_plumage    = w7·P̂
g_curiosity  = w8·P̂ + w9·Â
```

**Daily saturation.** Raw presence is passed through a soft cap before use:

```
P̂ = P_cap · (1 − exp(−P_day / P_cap)),  P_cap = 45 minutes of qualified presence per local day
```

This is not a punishment or a rate limit; it is the mechanical form of "a single session never moves a personality value visibly." Without it, a user who leaves a focused, mouse-jiggling tab open for a nine-hour workday out-drifts three weeks of honest watching, and the calibration collapses in exactly the way `concepts.md` warns about.

**Calibration.** The PRD names the target, so we solve for `α` rather than guessing:

- *Regular visits* is defined as 5 sessions/week × 8 minutes qualified presence ≈ **2400 presence-seconds/week**.
- *Measurable in instruments after ~1 week*: Δ on the dominant trait ≥ **0.010** after 7 days, against a harness resolution of 0.002.
- *Visible to the user after ~3 weeks*: cumulative Δ ≥ **0.08**, which must be large enough to (a) cross at least one `plumage_step` boundary and (b) shift the bird's front-perch occupancy probability by ≥ 15 percentage points.

From the first target, at a mid-range starting value `v ≈ 0.40`:

```
α_presence ≈ 0.010 / (2400 · 0.60) ≈ 7.0 × 10⁻⁶  per presence-second
```

Seed values for new birds are drawn per-species from a modest band (roughly 0.30–0.50, species-shaped) so that two starter birds are audibly and visibly distinct from day one without either being at a boundary.

These three numbers — 0.010 at one week, 0.08 at three weeks, `P_cap = 45 min/day` — are **the spec**. `α` is a derived constant in a config file. If product changes the targets, `α` is re-derived; nobody hand-tunes `α` toward a feeling. The synthetic-cohort harness (§11.5) asserts all three continuously.

### 5.3 Mood

Five states: `wary`, `content`, `curious`, `drowsy`, `alert`.

Mood is **hazard-based, not a state machine with hard triggers.** Each tick, for each bird, we compute a transition propensity toward each other state and sample:

```
h(s → s′) = base(s, s′, day_phase)
          · personality_mod(s′, vector)
          · event_bump(s′, recent_events)
          · weather_mod(s′, weather)
          · contagion(s′, neighbours)
```

- `base` encodes the diurnal shape: `drowsy` propensity rises through dusk, `alert` peaks in early morning, `content` is the broad daytime attractor.
- `personality_mod` is where the vector shapes what the user sees. Concretely, `h(→wary) *= (1 − 0.6·boldness)`; a bold bird is materially harder to spook on identical input, which is `bird_engine.md`'s stated requirement.
- `event_bump` gives a short-lived multiplier after an interaction — an accepted offer raises `→content` and `→curious` for ~3 ticks with exponential decay.
- `weather_mod`: rain suppresses vocal output aviary-wide for its duration and briefly after; wind raises `→alert` for high-boldness birds and `→wary` for low.
- `contagion`: an alarm call raises `→wary` in other birds for ~2 ticks, scaled by perch-zone proximity and damped by the receiver's boldness.

**Dwell floor.** `mood_floor_until` prevents transitions for 4–10 minutes (mood-dependent) after a change. Without it, hazard sampling at 60s produces visible flicker and the user reads noise instead of a mood.

**Persistence.** Mood lives in `bird.mood` and is written *only* by the tick. Session start does not touch it. This is asserted directly: an integration test opens a session, pulls a snapshot, and verifies zero writes to `bird` occurred (via a statement-level audit hook). "The user should never notice mood snapping to a default on tab open" is a property we can actually check.

**Daily-ish reset, without a snap.** At local ~04:00 the tick re-draws a per-bird *baseline bias* for the day. It does not set mood; it shifts the hazard field, and the bird's mood migrates over the next hour or so. A bird that ended yesterday drowsy is likely settled by 4am anyway, so the change is invisible; a bird that ended wary softens gradually, which is what `bird_engine.md` describes.

**Night.** Under `day_phase = night`, non-nocturnal birds sit low with eyes closed (a mood-adjacent `settled` motion register, not a sixth mood — it is `drowsy` at low energy). The single nocturnal species retains normal call-plan density with a night-shifted motif palette. Night is not a dead state.

### 5.4 Perch selection

Each tick, for each bird, a target zone is sampled from a distribution over `{front, middle, back}`:

```
P(front)  ∝ exp( 2.2·boldness + 0.9·mood_forwardness(mood) + 0.6·recent_listen_in )
P(middle) ∝ exp( 0.8 )
P(back)   ∝ exp( 2.0·(1 − boldness) + 1.1·mood_backwardness(mood) )
```

with a **stickiness** term that heavily favours staying put — a bird changes perch on the order of once every several minutes, not every tick. Slot assignment within a zone avoids overlap and gently attracts high-warmth birds toward occupied slots (they perch near others) and repels low-warmth ones.

There is no user-facing perch control anywhere: no endpoint, no drag handler, no keyboard command. Perch is a signal the user reads.

### 5.5 Presence accounting

The three-signal conjunction from `concepts.md`, implemented exactly:

**Client side.** A presence ping is emitted every **20 seconds** while and only while all three hold simultaneously:
1. `document.visibilityState === 'visible'`
2. the document has window focus (`document.hasFocus()`, maintained via `focus`/`blur`)
3. a `pointermove`, `pointerdown`, `keydown`, `touchstart`, or `wheel` occurred within the last **5 minutes**

Each ping carries the client's assertion of how long condition (3) has held. The activity window is 5 minutes, at the long end of "a few minutes," because — as `interactions.md` says outright — watching birds without moving is the actual product. A user sitting perfectly still for four minutes is doing the thing the product is for.

**Server side.** The server does not trust the client's duration claim. It credits, per ping, `min(now − last_ping_server_ts, 25s)`. A client that stops pinging for more than 50 seconds has its presence window closed with `terminal_reason = ping_timeout` and gets **no credit for the gap**. This handles suspended laptops, killed tabs, network drops, and clock skew with one rule.

**Explicitly not presence:** a visible-but-unfocused window; a focused-but-minimized window; a focused, visible window with no input for six minutes; a background tab of any kind. Each of these has a dedicated unit test asserting zero credited seconds, because the failure this guards against is silent and population-wide.

**Settle and tab-close are equivalent.** Both close the presence window; `terminal_reason` differs and drift does not. Test: two identical scripted sessions, one ending in settle and one in tab-close, must produce byte-identical `personality_delta` rows.

### 5.6 Call grammar runtime

**Motif library.** Each species has 8–14 motifs. A motif is a parametric primitive, not a recording: `{ kind: rise|fall|trill|buzz|click|hold|warble, duration, pitch_contour, harmonic_profile, envelope }`.

**The signature/expressive split** is the central invariant of this subsystem, and it is what makes `bird_engine.md`'s "recognizable across mood and personality drift" achievable rather than aspirational:

| **Signature parameters** — drawn once from `bird.call_seed` at adoption, **immutable for the life of the bird** | **Expressive parameters** — varied every call by mood, drift, weather, chorus context |
|---|---|
| The bird's motif subset (4–6 of its species' library) | Tempo (±25%) |
| Base pitch centre and interval set | Repetition count (1–4) |
| Harmonic profile / timbre | Inter-motif gap lengths |
| Characteristic 2–3 motif opening phrase | Brightness (filter cutoff) |
| Attack character | Amplitude, micro-jitter in onset timing |

Drift and mood may touch only the right-hand column. This is enforced by types: the synthesis function takes `(Signature, Expression)` where `Signature` is `readonly` and derived purely from `call_seed`, and there is no code path from a personality trait into a `Signature` field. A user who has spent two weeks with Pip knows Pip because Pip's timbre, intervals, and opening phrase have literally not changed.

**Call plan generation.** Each tick, for each bird, the server schedules calls over the next 180 seconds:

```
λ_bird = base_rate(species) · (0.35 + 1.3·vocal_frequency) · mood_vocal_mod(mood)
       · weather_vocal_mod · day_phase_mod · listen_in_boost
```

Calls are sampled as an inhomogeneous Poisson process with a refractory minimum, then post-processed for social structure:

- **Call-and-response**: with probability `∝ social_warmth` of a listener, a call from bird A schedules a response from bird B at +0.8 to +2.5s using B's own signature.
- **Chorus emergence**: chorus is not a scripted event. It falls out of two or more high-vocal-frequency birds' independent schedules overlapping, with a mild mutual-excitation term (`λ` bumps ~30% for 20s after hearing a neighbour). This is what makes chorus feel like something that *happened* rather than something that fired.

The plan is regenerated each tick, and the client always holds ≥120s of plan, so a late snapshot never produces silence.

### 5.7 Notebook generation

Entries are **rare**: the target is roughly one entry every 2–4 days for a regularly-visited aviary, and the sparsity is a hard constraint, not a tuning preference.

Mechanism, evaluated once per local day at a jittered time in the user's evening:

1. **Candidate detection.** The tick maintains a small rolling set of *noteworthy* facts: a first-in-a-while ordering change ("pip greeted before wren today, first time this week"), an unusual quiet stretch, a weather event and the birds' response, a first bath in the still pool, a bird holding an unusual perch, a long preen. Each candidate carries a *notability score*.
2. **Sparsity gate.** An entry is written only if `days_since_last_entry ≥ 2` **and** `notability ≥ threshold`, where the threshold rises with recent entry density. Active users therefore get *fewer* entries per session, not more — which is the correct direction, since `interactions.md` warns that an observation per session dilutes the entries that matter into noise.
3. **Rendering.** The chosen candidate is rendered through the naturalist grammar package into prose and stored rendered.

**The hard content rule**, from `interactions.md`: the notebook writes observations of *the aviary*, never observations of *the user's behaviour*. A candidate detector may reference the aviary's state and the birds' actions. It may not reference visit counts, session counts, day streaks, elapsed time since the user's last visit, or any aggregate of user presence. There is no candidate type that takes a user-behaviour statistic as input, and the candidate-type registry is a closed enum reviewed by the voice owner. "pip greeted before wren today" is legal. "you've been here every day this week" is not expressible.

**Voice enforcement.** All notebook templates live in `content/naturalist/` and are validated by the copy linter (§12.3): lowercase, present-tense, no second person, no exclamation, no gamification lexicon, at least one bird name or specific descriptor.

### 5.8 Return-greeting

The most-specified interaction in the PRD, and the easiest to flatten into "play arrival animation." It is computed server-side, inside the `reason=session_start` snapshot, so that it is consistent, narratable, and testable.

**Absence length** = `now − max(presence_window.ended_at)`, bucketed:

| Bucket | Absence | Greeting form |
|---|---|---|
| `brief` | < 10 min | a glance up from whatever the bird was doing; no call |
| `short` | 10 min – 6 h | glance, and a single short call with ~50% probability |
| `day` | 6 h – 3 d | a step toward the front perch and a short call |
| `long` | > 3 d | re-orientation: move to the front perch, a longer call, elevated chance a second bird answers |

**Greeter selection** is a weighted draw, not a fixed rule:

```
weight(bird) = (0.4 + 1.6·boldness)
             · mood_availability(mood)        -- drowsy ≈ 0.15, wary ≈ 0.3, curious ≈ 1.2
             · (0.5 + social_warmth)
             · recency_damp(last_greeted_at)  -- the same bird greeting every single time reads scripted
```

Exactly one bird is the primary greeter. A warier bird may not greet at all on a given day — that is a correct outcome, and the absence of a greeting from a wary bird is itself a signal the user reads. If a second bird qualifies (weight above a threshold, and `long` bucket raises the threshold pass rate), it greets **staggered by a uniform random 700–2200 ms**. Never in unison: a synchronized chorus on cue announces the user's arrival, which is precisely the affective register `interactions.md` forbids.

**Variation is generative, not a variant table.** The greeting call is assembled from the bird's grammar with freshly sampled expressive parameters; the motion is assembled from pose targets with fresh timing. There is no `GREETING_VARIANTS` array anywhere. Test: 500 consecutive `session_start` snapshots for a fixed bird produce zero exact duplicates on the tuple `(form, motif sequence, expressive params rounded to 2dp, stagger offset)`.

**And there is no textual welcome.** No toast, no banner, no modal, no "you've been gone N days," no sentence anywhere on the surface. This is enforced structurally: the design system has no toast, banner, or snackbar component (§12.2), and a test asserts that the DOM produced by a `session_start` render contains no text node that was not present in the steady-state render.

### 5.9 Scaling the tick without breaking the conceit

`accounts_sync.md` requires the tick to run whether or not a client is connected — this is what makes "the aviary has been continuing" true. Literally executing 60-second ticks forever for every account that ever signed up is unnecessary work, but *removing* the property is not acceptable. The resolution is equivalence, not omission.

**Active ticking** (the real 60s loop) runs for any aviary with: a connected client, any interaction event in the last 6 hours, a pending mood/offer/weather timer, or an unconsumed event.

**Dormant catch-up.** For an aviary with a provably empty event log across `[t₀, t₁]`, the state at `t₁` is computed in one pass by `catchUp(state, t₀, t₁)`. Because the tick is fully deterministic (§5.1) and the only inputs across a dormant interval are time and the seeded PRNG, the result is *identical* to folding the individual ticks — including which notebook entries were generated and at which tick indices, and including mood evolution through however many dawns and dusks passed.

This is guarded by a property test that is one of the most important in the suite:

```
∀ (state, t₀, t₁ with no events):  catchUp(state, t₀, t₁) ≡ fold(tick, state, t₀…t₁)
```

byte-for-byte over the entire canonical state and all emitted notebook entries. If that test ever fails, catch-up is disabled by config and the fleet falls back to full ticking while we fix it.

Catch-up runs on first access *and* on a slow background sweep (every aviary at least once daily), so notebook entries for a dormant aviary exist before the user opens the tab rather than materializing suspiciously at the moment of arrival.

**Cost shape.** At 60s cadence with ~2ms of compute per aviary-tick, one worker core handles ~30k active aviaries. Sharding is by `account_id` hash across N workers, with shard ownership in Redis and a lease. Dormant accounts cost one catch-up per day.

---

## 6. Sync model

### 6.1 The server is the only writer — enforced by grants

`accounts_sync.md` says clients never write personality state. Code conventions decay; grants do not:

```sql
-- aviary-api's role
GRANT SELECT ON bird TO api_role;
GRANT UPDATE (name) ON bird TO api_role;              -- rename only
REVOKE UPDATE ON bird FROM api_role;                  -- (then re-grant the single column)
GRANT INSERT ON interaction_event TO api_role;
REVOKE UPDATE, DELETE ON interaction_event FROM api_role;   -- append-only, at the database

-- sim-tick's role
GRANT SELECT, UPDATE ON bird TO tick_role;
GRANT INSERT ON personality_delta, notebook_entry TO tick_role;

-- visitor path
GRANT SELECT ON bird, aviary, species TO visitor_role;   -- nothing else, ever
```

If someone writes `UPDATE bird SET boldness = ...` in the API service, it does not fail code review; it fails at runtime in the first integration test, with a permission error naming the column. A migration test asserts the grant matrix on every deploy.

### 6.2 Additive deltas, never absolute values

The tick computes `Δv` from the event log and applies it to the stored vector. No message anywhere in the system carries an absolute trait value except the account export.

The failure `accounts_sync.md` describes — a phone session writing a vector derived from an older read, silently deleting the laptop's morning drift — is unreachable here, because there is no code path that writes a client-derived vector, and because deltas commute: applying the same set of deltas in any order over a monotonic accumulator yields the same result. Combined with the `UNIQUE (bird_id, tick_index)` constraint, a delta cannot be double-applied either.

### 6.3 How clients pull

Snapshot pull triggers, per `accounts_sync.md`:

| Trigger | Behaviour |
|---|---|
| Initial navigation | Snapshot inlined in the HTML by the edge (≤30s TTL), then reconciled |
| `visibilitychange` → visible | Immediate pull, `reason=visibility` |
| Long render-frame gap (>5s between rAF callbacks) | Immediate pull, `reason=resume`, cache-bypass |
| Keepalive while visible | Every 60s, aligned to the tick, jittered ±5s to avoid a thundering herd |
| Post-interaction | ~2s after an offer or settle, to pick up the resolved reaction |

Snapshots are 3–8 KB. `ETag` on `tick_index` means a keepalive during an unchanged tick is a 304.

**Reconciliation never pops.** When a fresh snapshot disagrees with the rendered state, the client eases to the new state over 400–900ms — birds move along a path, light ramps, plumage steps cross-fade. The one exception is a `resume` after a gap longer than ~2 minutes, where continuity is a lie anyway; there the client cross-fades the whole scene over 700ms. It never cuts, never re-mounts, never shows a load state.

### 6.4 Ordering, idempotency, and clock skew

- Ordering is by `(server_ts, id)`. `client_ts` is stored for debugging and never used to order or to compute durations.
- Idempotency is by `(account_id, client_event_id)`, a UUID the client generates before the first send attempt, so a retry after a timeout cannot double-count.
- Presence duration is derived server-side from inter-ping arrival (§5.5), so a device with a wrong clock, a throttled background timer, or a malicious payload cannot inflate drift.
- Events arriving for a tick that has already been consumed are attributed to the next tick. They are never dropped and never retro-applied.

### 6.5 Multi-device presence: union, not sum

The PRD does not address this case directly, and it matters: a user with the aviary open and focused on both laptop and phone would, under naive accounting, generate double presence and drift twice as fast.

That is the same corruption as counting "tab open" as presence — a laxer definition inflating the drift signal — so we resolve it the same way the PRD resolves the parent case. **Presence is the union of qualified intervals across devices, not their sum.** The tick collapses overlapping `presence_window` rows for an account before crediting seconds. One user watching from two devices is one user watching.

This is called out as a judgment call in §15 (A3), but the direction follows directly from `concepts.md`: the engine measures whether the user was there, and there is one user.

### 6.6 Conflicts

There is no state to merge, so the only conflicts are operational:

| Situation | Handling | Surface |
|---|---|---|
| Magic-link replay | `FOR UPDATE` + `consumed_at` check makes the second consume fail | "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| Session expired mid-visit | 401 on next pull; the client pauses the scene and shows a system surface | "Your session timed out. Sign in again to keep watching." |
| Snapshot fetch fails repeatedly | Client keeps rendering the last snapshot's motion (it is a continuous simulation, so it degrades gracefully for a minute), then a system surface | "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." |
| Two devices, one settles | `settled_until` is canonical; the other device eases into evening on its next pull | none — this is just the aviary being one aviary |
| Revoked visit invite, active visitor | 410 at the visitor's next pull | "This visit is no longer available." |

All of these are **matter-of-fact voice**, per the named exception in `product_brief.md`. Rendering path: system surfaces are DOM components in `surfaces/system/`, styled distinctly from the aviary, sentence-cased, direct. The client maps error `code` → copy; the server never sends prose.

### 6.7 Timezone changes

Day/night follows the user's local time. The client reports its IANA timezone on each snapshot request; the tick uses `account.tz_iana`. When the reported zone differs from the stored one for **more than 6 hours of continuous reporting** (hysteresis, so a VPN or a single odd reading doesn't flap the aviary), the stored zone updates.

A zone change must never move the aviary *backwards* through the day — a user flying east would otherwise watch evening un-happen. The day-phase driver is clamped to be non-decreasing within a local day; a backwards jump holds the current phase until the new local clock catches up.

---

## 7. Frontend rendering pipeline

### 7.1 Technology

- **Canvas 2D**, single canvas, `devicePixelRatio`-aware. Not WebGL: seven birds, four parallax layers, and a handful of ornament particles do not need a GPU pipeline, and WebGL costs context-loss handling, shader compilation on the critical path, and a larger core chunk — all three of which fight the 500ms budget.
- **Chrome in DOM** (top bar, notebook, settings, offers): accessible, keyboard-navigable, and contrast-checkable for free. Preact for the chrome (~4KB) — a full framework in the critical path is unaffordable and unnecessary.
- **No CSS or Web Animations API driving bird motion.** All bird motion is sampled per-frame from the pose model so that reduced-motion and standard registers share one implementation (§7.6).

### 7.2 Boot path (the 500ms budget)

```
0ms     navigation
~90ms   edge returns HTML: inline critical CSS, inline snapshot JSON (signed, ≤30s TTL),
        modulepreload for core chunk. Sky gradient painted from CSS alone, using the
        device clock's local hour — correct light before any JS runs.
~180ms  core chunk (≤120KB gz) parsed: canvas bootstrap, silhouette geometry,
        pose sampler, interpolator, snapshot hydration
~260ms  FIRST BIRD DRAWN — mid-pose, at motion_phase from the snapshot
        performance.mark('first-bird')
~400ms  ornament layer, parallax foliage
~700ms  audio chunk loads; ambient bed fades in over 1.2s; call plan scheduling begins
~1.2s   chrome chunk (top bar), then idle-time prefetch of notebook/settings/offer chunks
```

The inline snapshot is the key move: the first render needs no network round trip beyond the HTML itself. Its ≤30s TTL means the day-phase and bird positions are current; the client reconciles against a live pull as soon as the core chunk is up, easing rather than popping (§6.3).

**There is no load state on this path.** When the inline snapshot is absent (cold edge, uncacheable) or the network is slow, the fallback is the **quiet field**: the sky gradient at the correct local phase plus one or two slow foliage sways, rendered by the core chunk with no bird data. It reads as the aviary before you have focused on it. It is not a spinner, and it cannot become one — there is no spinner component in the design system (§12.2), and `aviary_layout.md` is unambiguous that a spinner says "machine."

### 7.3 Bundle budget allocation

Hard cap 2MB gzipped at first paint; the working allocation, enforced per-chunk in CI:

| Chunk | Budget (gz) | Contents |
|---|---|---|
| `core` (critical) | 120 KB | canvas bootstrap, geometry, pose sampler, interpolation, snapshot hydration |
| `scene` | 380 KB | full parallax layers, species geometry parameter sets, ornament emitters, weather |
| `audio` | 260 KB | WebAudio graph, motif synthesis, scheduler, mixer, caption renderer |
| `chrome` | 180 KB | top bar, focus management, narration composer, naturalist grammar package |
| `runtime` | 120 KB | Preact, router, fetch layer, error surfaces |
| Deferred (`notebook`, `settings`, `visits`, `onboarding`) | 340 KB | loaded on demand; not counted against first paint |
| **Headroom** | **~600 KB** | deliberately unspent |

Bird visuals are **procedural vector geometry with per-species designer-authored parameters** (silhouette control points, feather-tract definitions, palette ramps), rasterized at runtime. No sprite sheets. This is what makes six species plus five plumage steps plus mood-shaped poses fit in the scene budget, and it is the same forcing function that makes audio procedural.

CI gate: `size-limit` per chunk; a PR that grows any chunk by >2% fails and must state why.

### 7.4 Scene composition

Four layers, back to front:

1. **Sky** — vertical gradient driven by `light.warmth` / `light.level`, plus a slow cloud-density field. Parallax 0.
2. **Background foliage** — soft silhouetted masses, slow sway. Parallax 0.02.
3. **Perch plane** — three perch zones and all birds. Parallax 0. *(The birds do not parallax. Depth is read from perch zone, scale, and atmospheric fade, not from motion offset — parallaxing the subject makes it feel like a diorama.)*
4. **Foreground ornament** — occasional passing branch, drifting leaves and feathers. Parallax 0.06.

`aviary_layout.md` says explicitly the product is not parallax-heavy; 0.02/0.06 are deliberately just past the threshold of conscious notice.

**Layout.** Perch zones are anchored at fractions of a safe area, with front lower and larger, back higher and smaller (scale 1.0 / 0.86 / 0.74, with a slight desaturation and lift toward the back). Slots within a zone are spread across the width. On narrow viewports the horizontal spread compresses and the zones' vertical separation increases slightly to preserve legibility; **nothing is ever cropped.** Every bird's bounding box is clamped into a safe inset each frame.

Test: at 320×568, 390×844, 768×1024, 1440×900, and 2560×1440, with 2 and with 7 birds, every bird's bounding box is fully within the viewport with ≥8px margin.

### 7.5 Idle micro-motion — the pose walk

Birds are never still in a way that reads as paused, and they are never playing a clip.

Each bird has a **pose space**: a small set of parameterized targets (`perch_neutral`, `preen_breast`, `preen_wing`, `scan_left`, `scan_right`, `tilt`, `fluff`, `shuffle`, `wing_flick`, `call_open`, `settle_low`). A pose is a vector of joint/shape parameters; any two poses blend continuously.

Motion is generated by a **noise-driven walk through pose space**:

```
w(t) = softmax( A(mood) · valueNoise( t · rate(mood) + motion_phase, octaves=3 ) + bias(mood) )
pose(t) = Σ  w_i(t) · pose_i
```

- `motion_phase` comes from the snapshot, so **the first frame is already somewhere in the middle of the walk.** There is no `t = 0`.
- `A(mood)` and `bias(mood)` are what make idle motion mood-shaped: a wary bird's weights favour `scan_*` and hold higher `rate`; a content bird's favour `preen_*`; a curious bird's favour `tilt` and respond to call events with a head turn toward the source; a drowsy bird's collapse toward `settle_low` and `fluff` with a low rate.
- Layered on top: an always-on breathing oscillation (~0.35 Hz, amplitude mood-scaled) and a small weight-reset shuffle at Poisson intervals.

The user reads mood from motion. There is no mood label, tooltip, icon, or badge anywhere in the product — the whole point of mood-shaped idle is that being told would be a failure.

Test: capture 10 minutes of pose parameter output and assert no window of ≥3 seconds repeats within tolerance. A looping clip would fail this immediately.

### 7.6 Reduced-motion register

`accessibility_perf.md` is emphatic: reduced-motion is a designed rendering of the same aviary, not animations-off.

Implementation: **the same scene graph, the same state, the same pose space — one swapped sampler.**

| | Continuous register | Reduced register |
|---|---|---|
| Pose | noise walk at 60fps | hold a sampled pose 6–10s, cross-fade 1.2–1.8s to the next pose drawn *from the same distribution* |
| Flight between perches | eased arc path | cross-fade out at origin, cross-fade in at destination |
| Breathing | ~0.35 Hz oscillation | omitted |
| Ambient leaves/feathers | active emitter | disabled |
| Day/night colour ramp | continuous | retained, slowed ~2× |
| Top-bar fade | 900ms opacity | retained (opacity is not vestibular) |
| Calls, drift, mood, notebook, narration | full | **identical — full** |

Because the two registers are one code path with a sampler flag, a new bird behaviour cannot ship to one register and not the other. That is the structural answer to "the reduced-motion mode rots into a fallback."

Activation: `prefers-reduced-motion: reduce` (live-updating via `matchMedia`), or the accessibility settings toggle, which can also *override in either direction* — a user who wants full motion despite the OS setting can have it.

### 7.7 Interpolation and the hidden tab

- **Snapshot → snapshot**: perch changes ease over 700–1400ms along a flight arc; light and plumage steps cross-fade.
- **Late snapshot**: the client extrapolates conservatively — pose walk continues, calls continue from the buffered plan (≥120s held), no perch changes invented. It degrades into "the aviary is calm right now," which is a legal aviary state.
- **Hidden tab**: `rAF` stops, the audio context suspends, the presence window closes, ornament emitters stop. Nothing is rendered because nothing is visible and rendering costs battery. The simulation continues server-side.
- **Return to visible**: fresh snapshot, then render — and the first frame is again mid-action at the new `motion_phase`. No fade-in, no re-entry sequence. The aviary was running; the client just started drawing it again.

### 7.8 The scene carries no chrome

No buttons, badges, hover tooltips, overlay icons, inline labels, or hover states on birds inside the aviary. Clicking a bird engages listen-in; that is the entire in-scene interaction vocabulary, plus keyboard focus.

Captions (§10.4) and focus rings are the only things drawn over the scene, and both are accessibility surfaces that the user has enabled or invoked.

**Top bar**: exactly four items — account/settings, accessibility settings, field notebook, offer. It fades to 8% opacity after **4 seconds** of cursor stillness (900ms ease) and returns to full in 150ms on any pointer move, key press, or focus event. It never fades while a menu is open, while focus is inside it, or while a screen reader is active (detected via focus-visible interaction patterns; when in doubt, do not fade).

### 7.9 Onboarding and the empty aviary

1. Sign-in (system voice, DOM, matter-of-fact).
2. The system selects two species from the pool. **There is no catalog and no picker** — `bird_engine.md` is explicit that the first encounter should be meeting an animal, not configuring an avatar. The two are presented as the birds that arrived.
3. Naming: two fields with naturalist default suggestions already filled in, so a user who just presses continue gets named birds rather than a blank.
4. **The empty-aviary state** — the same quiet field as the loading state: sky at the correct local phase, foliage, no birds.
5. Each bird enters with a soft fly-in to its starting perch, staggered ~1.5s. From this moment the user never sees an empty aviary again.

Step 4 is a real state with a real design, not a gap between screens.

---

## 8. Audio pipeline

### 8.1 Graph

```
per bird:  [voice pool] → envelope gain → per-bird filter → per-bird pan → mix gain ─┐
ambient bed (filtered noise: wind, foliage, rain) ───────────────────────────────────┤
                                                                                     ├→ aviary bus → limiter → destination
offer sounds (song fragment: procedural motif) ──────────────────────────────────────┘
```

Everything is synthesized. There are no audio file assets in the repository, and a CI check fails the build if any `.mp3`, `.wav`, `.ogg`, `.m4a`, `.opus`, or `.flac` file appears in the bundle graph. `accessibility_perf.md` makes the no-recorded-audio rule unconditional; a dependency check makes it unbreakable by accident.

### 8.2 Voice synthesis

Per motif: 2–3 detuned oscillators (or a small wavetable) through a resonant bandpass whose centre frequency is automated along the motif's pitch contour, plus an amplitude envelope, plus an optional FM operator for trills and buzzes. Warble is contour LFO; click is a short filtered noise burst.

All of it is parameter automation on a fixed node graph.

**Voice pooling is a hard requirement**, both for the 30-minute no-memory-growth rule and for scheduling latency. WebAudio oscillator nodes are single-use by spec, so the pool holds persistent oscillators started once at context creation and kept silent by their envelope gains; a "call" schedules parameter automation and opens a gain envelope. Nothing is constructed per call in steady state.

Pool size: 9 voices (7 birds + 2 for overlap during response pairs). Allocation is least-recently-finished; if the pool is exhausted, the quietest scheduled call is dropped rather than clipped — a dropped call is invisible, a stolen voice is audible.

### 8.3 Scheduling

A lookahead scheduler on a 25ms interval with a 150ms horizon, reading the server call plan and converting `at` offsets into `AudioContext.currentTime` values. Standard WebAudio practice, and it survives main-thread jank because the automation is committed to the audio thread ahead of time.

The client holds ≥120s of call plan, so a delayed snapshot never produces a silent aviary.

### 8.4 Chorus mixing

Real chorus is the point, and real chorus is where naive mixers turn to mush. Three rules:

- **Spatial spread.** Pan is derived from perch slot (±0.35) with a small per-call jitter; back-zone birds get a touch of high-shelf rolloff and a hair of reverb send. Distinguishable positions keep signatures legible when two birds overlap.
- **Crowd ducking.** At most 3 voices sound at full level. The 4th and beyond attenuate 4–6dB, oldest-onset first. This preserves per-bird recognizability at the seven-bird cap, which `bird_engine.md` identifies as the load-bearing affordance that sets the cap at all.
- **No shared bus compression that pumps.** The limiter is a transparent brick-wall for safety only, at a threshold the normal mix never approaches.

Because calls are procedural, two simultaneous calls mix as two independent signals — none of the phase-cancellation artifact `bird_engine.md` warns about from layered recordings.

### 8.5 Listen-in mix

- **Engage**: focused bird's mix gain `+4 dB` via `setTargetAtTime` with a ~1.6s time constant. Others ramp to `−9 dB` over the same period. Its pan centres slightly; a small reverb-send reduction brings it forward.
- **Others never go silent.** `−9 dB` is a floor with a hard clamp; there is no code path to `−∞` or to `gain = 0` for a non-focused bird. `interactions.md` is explicit: this is a rebalance, not a mute, and silencing the others would teach the user the aviary is a set of soloable tracks.
- **Disengage**: same 1.6s ramp back to ambient, triggered by clicking the focused bird again, focusing a different bird, clicking empty aviary space, `Escape`, or keyboard focus moving away.
- Visual accompaniment is minimal and in-scene: a very slight atmospheric lift on the focused bird. No ring, no highlight, no badge, no label — the aviary carries no chrome.

Listen-in start/end events are posted to the event log; duration feeds drift on that bird's social warmth and vocal frequency.

### 8.6 Autoplay policy — a real conflict, resolved

`aviary_layout.md` requires calls to be audible in the first frame. Every modern browser requires a user gesture before an `AudioContext` may produce sound. These cannot both be fully satisfied on a cold navigation. This is the sharpest spec/platform tension in the plan, and the resolution has to not violate "notice, never announce."

**Resolution.** On load, attempt `context.resume()` immediately — on a returning visit with prior engagement, browsers frequently permit it, and the ideal case works. If it is blocked:

- The aviary runs in **silence with captions automatically enabled** — exactly the WebAudio-unavailable path, which the PRD already establishes as the correct graceful degradation.
- **No "click to enable sound" banner, overlay, or prompt.** That is an announcement, and it would be the first thing a new user sees.
- The muted state is legible as a small speaker glyph in the top bar (the top bar is where chrome lives), which the user can also click.
- The *first* pointer or key event anywhere — including the pointer movement that reveals the top bar, or the click that engages listen-in — resumes the context, and the ambient bed and calls fade in over 1.2s. In practice audio arrives within the first few seconds for any user who moves their mouse.

The cost is that a completely motionless first-time visitor experiences a silent aviary with captions. That is a strictly better failure than a modal gate, and it is consistent with how the PRD already handles missing WebAudio. Flagged as judgment call A5 (§15).

### 8.7 Fallback and captions

No WebAudio, denied context, or hardware failure → **silence with captions on by default**. No recorded-audio fallback path exists.

Captions are generated by the same module that drives synthesis, from the same call plan: motif sequence + expressive parameters → naturalist phrase.

```
["rise","rise","rise"] tempo 1.0 amp .5  →  "a soft three-note rise"
["trill","gap","trill"] tempo .9         →  "a low trill, paused, low trill again"
["click"] amp .9 zone=back               →  "a single sharp call from the back perch"
```

Because both the synthesizer and the caption renderer consume one plan object, a caption can never describe a call that did not play. Caption phrasing passes the naturalist copy linter like every other product-surface string.

### 8.8 Ambient bed and offer sounds

The ambient bed is filtered pink noise with slow band modulation — wind through foliage, intensified during a wind event; rain adds a denser filtered band plus sparse droplet transients. Procedural, so it never loops audibly.

The song-fragment offer plays a short procedural motif from a small designer-authored library in the aviary's ambient register, softly. Bird responses (joining, going quiet, calling against) are scheduled by the tick from vocal frequency and mood, and arrive in the next call plan.

---

## 9. Accessibility surfaces

The framing from `accessibility_perf.md` governs everything in this section: the user gets the actual product, not a stripped-down variant. Accessibility work ships **with** v1; a launch candidate without §9 complete is not a launch candidate (§11.6 gate).

### 9.1 Screen-reader narration

**Composition.** A shared `naturalist-grammar` package composes prose from snapshot state. The same package renders notebook entries server-side. One package means the narration and the notebook cannot drift into two different products with two different personalities — a concern `accessibility_perf.md` raises directly.

Narration runs client-side from the snapshot: low latency for user-initiated events, and no round trip for an idle update.

**Structure.** An update is assembled from a subset of clause types, varied so it never reads as a template:

```
[subject clause]   "a small grey bird is perched on the front rail"
[action clause]    "calling softly"
[second subject]   "another bird sits further back with feathers fluffed"
[ambient clause]   "it is morning in the aviary; the light is gentle"
```

→ *"a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle."*

**Cadence.** One update per **30–60 seconds** at idle, jittered so it does not feel metronomic. User-initiated events (return-greeting, offer reaction, settle) get a prompt update with a priority bump. Never more than one queued update: a new one supersedes a pending one rather than queueing behind it. High-frequency narration would force the user to silence it, which is the system pushing the accessibility surface aside.

**Delivery.** A single `aria-live="polite"` region, `aria-atomic="true"`. User-initiated events use the same polite region with an immediate flush — `assertive` interrupts the user's own reading and is the screen-reader equivalent of a toast.

**Repetition control.** A rolling window of the last 6 updates suppresses clause-template reuse; the composer prefers whatever has changed since the last update (a perch move, a mood shift, weather starting) so successive updates read as observation rather than restatement.

**Prohibited in narration**, enforced by the same copy linter that governs the notebook: trait numbers, bare mood labels ("mood: content"), perch indices ("perch 2"), state-transition phrasing ("Pip moved to front perch"), any second-person construction, any announcement of the user's arrival.

### 9.2 Keyboard navigation

Exactly the model `accessibility_perf.md` specifies:

| Key | Behaviour |
|---|---|
| `Tab` | through top-bar items, then into the aviary scene |
| `Tab` into the scene | focuses the first bird (front zone, leftmost) |
| `←` `→` | move focus between birds by horizontal position |
| `↑` `↓` | move focus between perch zones |
| `Enter` / `Space` | engage listen-in on the focused bird |
| `Escape` | disengage listen-in; second `Escape` leaves the scene |
| `Tab` from a bird | leaves the scene (birds are a single tab stop with roving focus) |

**Implementation.** Each bird gets an absolutely-positioned, visually-transparent `<button>` in an overlay layer, kept aligned to the canvas position at 10Hz (not per-frame — layout thrash at 60fps would cost the frame budget). Roving `tabindex` keeps the bird group a single tab stop. Accessible name is the bird's given name plus its naturalist descriptor: *"pip, a small grey bird on the front rail"* — not "bird 1."

The offer affordance and settle are top-bar items and are fully keyboard-navigable; the offer panel is a standard focus-trapped dialog with `Escape` to close and focus restoration on close.

### 9.3 Focus indication

A double outline — 2px light inner, 2px dark outer, 3px offset — so it reads against both a bright midday sky and a dim night scene. Validated automatically: the focus ring is composited against sampled scene pixels at 12 points across the day/night palette ramp and in every weather state, asserting ≥3:1 contrast on at least one of the two strokes at every sample.

### 9.4 Captions

Rendered as DOM text near the calling bird's canvas position, fading in with the call and out ~1.2s after it ends. They use naturalist voice and are validated by the copy linter.

Captions carry `aria-hidden="true"`. Screen-reader users already receive call information through narration; duplicating it into the live region would flood the queue — the precise failure §9.1 is designed to avoid. Captions serve audio-off, hearing-difference, and noisy-environment users.

Contrast against a dynamic scene is solved with a soft radial scrim behind caption text (a low-opacity darkening or lightening chosen from the local scene luminance), guaranteeing AA regardless of what is behind it. Tested at the same 12 palette-ramp sample points.

Enabled from accessibility settings, and **on by default** whenever audio is unavailable or suspended.

### 9.5 Contrast

All user copy passes WCAG AA (4.5:1 body, 3:1 large text and UI boundaries). Since the aviary scene contains no user copy except the top bar, captions, and focus rings, the surface area is small and fully testable:

- Static chrome: automated token-pair contrast check over the design-token file, in CI.
- Dynamic overlays (captions, focus ring, top bar over the scene): the 12-point palette-ramp sampling described above.

### 9.6 Voice split enforcement

`product_brief.md` names the exception and says it should be obvious where the line falls. It is drawn in the directory structure:

- `content/naturalist/` — notebook templates, narration clauses, caption phrases, offer prompts, bird descriptors. **Lint**: lowercase, present tense, no second person, no exclamation marks, no gamification lexicon, at least one specific noun.
- `surfaces/system/` — sign-in, sync errors, account settings, accessibility settings, session list, export, deletion, visit management, unsupported browser. **Lint**: normal sentence capitalization, no naturalist lexicon (`perch`, `flutter`, `settle`, `notice` as product verbs), imperative next step present for every error string.

Both linters run in CI. A string in the wrong directory fails on the other directory's rules, so misplacement is caught mechanically. The general rule for any future surface, from `product_brief.md`: **any surface where the user is engaging with the system as a system — identity, money, errors, settings — drops out of the naturalist register.**

---

## 10. Performance budgets and observability

### 10.1 Budgets and how each is enforced

| Budget | Target | Enforcement |
|---|---|---|
| Initial JS bundle | <2MB gz | `size-limit` per chunk in CI; >2% growth fails |
| Time to first bird | <500ms on mid-tier mobile / 4G | Scripted run on a throttled profile (4× CPU slowdown, 70ms RTT / 9Mbps), asserting `performance.mark('first-bird')` ≤ 500ms at p75 over 20 runs |
| Idle frame rate | 60fps, sustained 30 min | Headless run on a pinned CPU-throttled profile with 7 birds: p95 frame ≤ 16.7ms, and no more than 3 frames >50ms across the run |
| Memory | no growth over 30 min | 30-min soak sampling heap every 60s: regression slope ≤ 0.5MB/30min **and** post-forced-GC delta ≤ 3MB |
| Tick latency | p99 < 5s | Production alarm; pages |

The frame-rate and memory checks run nightly (they are too slow for every PR) plus on any PR touching the renderer or audio. The bundle and first-bird checks run on every PR. A **real five-year-old laptop** (2021 mid-range, pinned OS and browser versions) runs the full suite weekly in the device lab — the throttled CI profile is a proxy, and proxies drift.

`accessibility_perf.md` says the memory rule is "a real test in CI, not a guideline." The two-part assertion above is what makes it real: slope catches leaks, post-GC delta catches retention.

### 10.2 The usual memory traps, pre-empted

- **Audio**: voice pooling (§8.2); no node construction in steady state; `AudioContext` closed on unload; automation events cancelled on disengage.
- **Notebook**: virtualized list; entries outside the viewport are released; a scroll-to-2000-entries test asserts bounded DOM node count.
- **Renderer**: bird raster caches are LRU-bounded (≤ 7 birds × 8 cached poses × DPR variants); ornament particles come from a fixed pool; no per-frame closure allocation in the hot path (verified by an allocation-profile assertion in the soak test).
- **Snapshots**: only the current and previous snapshot are retained; the call-plan buffer is trimmed to the horizon.

### 10.3 What we measure

Aggregate RUM, no per-account dimension:

- Page-load timings, `first-bird` mark distribution
- Render-frame timing histograms (p50/p95/p99), sampled at 1% of sessions
- Audio-context availability rate, audio error counts by class
- API latency and error rate by route
- Simulation-tick latency, tick lag (aviaries more than 3 ticks behind), catch-up invocation rate
- Auth funnel counts (link requested / consumed / expired), aggregate
- Reduced-motion and caption adoption as aggregate counts

Plus synthetic monitoring: a fleet of automated browsers loading the aviary on a schedule from several geographies, asserting first-bird timing and audio initialization.

### 10.4 What we deliberately do not measure

This list is part of the plan, not an omission from it. The privacy commitment in `accounts_sync.md` is an architectural rule, and the non-goals make some metrics product-hostile regardless of privacy:

- No per-account or per-bird dimension on any metric, ever
- No per-bird interaction telemetry of any kind leaving the simulation database
- No population-level drift analysis from production data (§10.5 covers the legitimate need)
- No engagement metrics as goals: no DAU/WAU dashboards, no session-count-per-account, no retention cohorts keyed to bird state, no "days since last visit" aggregate
- No cross-account aggregates that a leaderboard or discovery feed would need — `social_optional.md` notes that the architectural absence is what makes those features harder to add later, and that is intentional

**Enforcement.** Metric definitions live in a typed registry whose dimension type is a closed union of safe keys (`route`, `status_class`, `region`, `browser_family`, `device_class`). `AccountId` and `BirdId` are branded types that do not satisfy it. A metric with an account dimension is a compile error, not a review catch.

**Pipeline separation.** The analytics warehouse has no credentials to the simulation database. The simulation database is not in any ETL source list. There is no ML training pipeline; if one ever exists, per-bird fields are not in a schema it can reach.

### 10.5 Measuring drift calibration without touching user data

We need to know whether drift calibration is right. Aggregating production drift would violate the privacy commitment even for an anodyne "average drift across accounts" dashboard — `accounts_sync.md` names that exact example as forbidden.

**Resolution: a synthetic cohort.** A staging tick fleet runs several hundred seeded aviaries driven by scripted presence patterns — daily-8-minutes, weekend-only, twice-daily, three-weeks-then-absent, marathon-single-session, two-devices-simultaneously. The harness asserts the §5.2 calibration targets continuously, and every drift-related change must keep them green.

Real-world calibration signal comes from **opt-in qualitative interviews** during beta ("does Pip seem different than three weeks ago?"), not from mining beta users' bird data. Slower, and correct.

### 10.6 Alarms

| Alarm | Threshold | Severity |
|---|---|---|
| Simulation-tick latency p99 | > 5s | page |
| Drift monotonicity violation | any occurrence | **page** — product-integrity |
| Tick lag | > 0.5% of active aviaries more than 3 ticks behind | page |
| Catch-up equivalence test failure | any | page + auto-disable catch-up |
| First-bird p75 | > 500ms for 15 min | page |
| Audio-context error rate | > 2% of sessions | ticket |
| Auth consume failure rate | > 5% | page |
| Email-in-logs scanner | any hit | page |
| Personality write from a non-tick role | any (grant denial) | **page** |

### 10.7 Browser support

Last two major versions of Chrome, Safari, Firefox, and Edge. Older browsers get a matter-of-fact unsupported-browser surface naming what is needed. No compatibility shims, no polyfill bundles for old engines — `accessibility_perf.md` is explicit that the cost-benefit does not justify the bundle bloat, and the bundle is the binding constraint on the whole client.

---

## 11. Rollout

### 11.1 Milestones

Assumes a team of ~7: 2 backend, 2 frontend/rendering, 1 audio/DSP, 1 design (visual + voice), 1 QA/perf-accessibility. Durations are engineering estimates, not commitments.

**M0 — Audio and engine spike (4 weeks). Gated.**
Two procedurally-synthesized birds with distinct signatures, a working drift function on synthetic input, a running tick.
*Gate:* an ABX listening test with a 12-person panel achieves ≥80% correct identification between two birds' calls, across three moods. If procedural calls cannot clear this at two birds, they will not clear it at seven, and the product's central affordance needs rethinking before another line of client code is written. This gate is first because it is the only genuinely novel technical risk in the plan.

**M1 — Canonical spine (4 weeks).**
Postgres schema, grant matrix, synthetic-ID enforcement (all four points in §3.6), magic-link auth, sessions, the tick loop with exactly-once semantics, snapshot API with the serializer allowlist, event ingest, presence accounting with the three-signal conjunction. Single-device, no rendering beyond a debug view.

**M2 — The aviary renders (5 weeks).**
Canvas pipeline, four layers, pose walk, perch zones, day/night ramp, weather, the boot path meeting the 500ms budget, quiet-field loading state, responsive layout, top-bar fade. Audio engine integrated: call plan consumption, scheduling, chorus mixing, listen-in.

**M3 — Accessibility, in full (3 weeks). Gated.**
Narration, reduced-motion register, captions, keyboard navigation, focus indication, contrast validation, copy linters. *Gate:* all of §9 complete, all a11y CI checks green. `accessibility_perf.md` states that a reduced-motion mode shipping as a v1.1 fix is a v1 that told those users the product was not for them; making this a gate ahead of feature completion is how that is prevented in practice.

**M4 — The full session (4 weeks).**
Return-greeting with absence bucketing and generative variation, offers with cooldowns and mood-shaped reactions, settle with 5s undo, the field notebook (candidate detection, sparsity gate, naturalist rendering), adoption/onboarding flow, account settings, export, deletion.

**M5 — Multi-device and visits (3 weeks).**
Multi-device verification (union presence, reconciliation easing, chaos tests), the visit flow end to end — invite, read-only snapshot on the separate router, revocation, expiry, visit log, opt-in notification toggle.

**M6 — Private beta (≥6 weeks, overlapping hardening).**
~100 invited accounts. Six weeks minimum is a hard floor, not a schedule preference: the drift calibration's headline claim is "visible after about three weeks," and a two-week beta cannot observe it at all. We need at least two three-week windows to see whether the third week actually lands.

**M7 — Launch.**

### 11.2 Launch gates

Every one of these is binary:

1. M0 ABX gate passed at 2 birds, re-run and passed at 7 birds.
2. All §10.1 budgets green on the real five-year-old laptop, not only in CI.
3. All of §9 shipped; a11y checks green; a manual screen-reader pass on NVDA/Windows, VoiceOver/macOS, and VoiceOver/iOS.
4. Grant matrix verified in production; a penetration attempt to write personality via the API returns a permission error.
5. Drift monotonicity property test green over 10,000 randomized event streams.
6. Catch-up equivalence property test green.
7. Copy linters green; a manual voice review of every user-visible string by the voice owner.
8. Charm-invariant suite (§12) green.
9. Privacy: metric registry compiles with no account dimension; log scanner clean; warehouse has no credential path to the simulation database.

### 11.3 Ramping birds per aviary

The engine supports seven from day one and the audio mix is validated at seven before launch. What ramps is the **age-gated offer pacing**, which is the user-visible part.

Offers are triggered by `aviary.created_at` age, per `bird_engine.md`: not visit count, not interaction score, not tier. The schedule:

| Bird | Aviary age |
|---|---|
| 3rd | 90 days |
| 4th | 210 days |
| 5th | 370 days |
| 6th | 580 days |
| 7th | 840 days |

This pacing means essentially every launch-cohort account sits at two birds for the first quarter, which is both correct for the product ("two is companionship without overload") and operationally convenient: it gives us a long runway to validate seven-bird chorus behaviour in the wild before any real account reaches it. The schedule is a config value, adjustable after beta observation — but only in the direction the PRD describes, as *relationship pacing*, never as a reward curve.

The offer itself appears in the aviary as a bird arriving at the edge of the scene, not as a notification, badge, or modal. The user names it or declines.

### 11.4 Instrumented from day one

Everything in §10.3, live at launch. Explicitly *not* added later "once we need it": the synthetic-cohort harness (§10.5) and the charm-invariant suite (§12), both of which are load-bearing for keeping the product the product.

### 11.5 Post-launch posture

The first three months after launch are spent on calibration and hardening, not features. The two questions that matter: does drift feel right at three weeks, and does the seven-bird chorus hold up. Both need real time to answer, and neither is answerable by shipping something else in the meantime.

---

## 12. The charm-invariant suite

The PRD's affective rules become a named CI suite. This section exists because these rules are the product, and because every one of them fails silently.

### 12.1 Behavioural invariants (automated tests)

| # | Invariant | Test |
|---|---|---|
| 1 | Personality never crosses the API boundary | Recursive schema scan of every response type for trait field names; snapshot serializer field allowlist |
| 2 | Drift is monotonic | Property test over 10k randomized event streams incl. long absences; DB trigger; `g_k ≥ 0` by type |
| 3 | Neglect changes nothing | 14 simulated days with zero events → zero non-zero `personality_delta` rows |
| 4 | Settle ≡ tab-close | Two identical scripted sessions differing only in terminal gesture → byte-identical deltas |
| 5 | Presence requires all three signals | 8 unit tests, one per combination; only the full conjunction credits seconds |
| 6 | Presence does not double-count devices | Two overlapping device sessions → union seconds, not sum |
| 7 | No client writes personality | Grant matrix assertion; integration test attempts a write and expects a permission error |
| 8 | Mood does not reset on session start | Session-start integration test asserts zero writes to `bird` |
| 9 | Greetings never repeat | 500 consecutive greetings → zero duplicate `(form, motifs, expr, stagger)` tuples |
| 10 | Idle motion never loops | 10-min pose capture → no ≥3s window repeats within tolerance |
| 11 | Call signature is immutable | Drift a bird to trait ceiling; assert signature parameters bit-identical to adoption |
| 12 | Non-focused birds never silence | Fuzz listen-in engage/disengage; assert every bird's gain stays above the floor |
| 13 | Bird identity is permanent | Rename, sync, species-pool migration, export/import → `bird.id` unchanged |
| 14 | Notebook is sparse | 90 simulated days of heavy use → entry count within the 1-per-2-to-4-days band |
| 15 | Notebook never observes the user | Candidate-type registry is a closed enum; no candidate accepts a user-behaviour statistic |
| 16 | Visitors never drift the host | Simulated 1-hour visit with zero host presence → zero deltas |
| 17 | No visitor write path exists | Route-table assertion: the visitor router registers only `GET` |
| 18 | Catch-up ≡ real ticking | Byte-for-byte equivalence over randomized dormant intervals |
| 19 | No recorded audio | Bundle-graph scan for audio file extensions |
| 20 | Reduced motion has full parity | Both registers run the same scenario; assert identical drift, mood, calls, notebook, narration |
| 21 | Session start adds no text | DOM diff between session-start render and steady-state render → no new text nodes |
| 22 | No email outside auth | Schema scan; log scanner; blind-index import restriction |
| 23 | No account dimension in telemetry | Metric registry type check (compile-time) |
| 24 | No bird is ever cropped | Viewport matrix × bird-count matrix bounding-box assertion |

### 12.2 Structural absences (things that do not exist)

The strongest guards are the ones that make the wrong thing unbuildable. The design system contains **no**:

- toast, snackbar, banner, or notification component
- badge, pill-counter, or numeric indicator component
- spinner, progress bar, or skeleton loader
- confetti, celebration, or milestone animation
- streak, calendar-heatmap, or visit-log-for-the-user component
- modal that fires on session start
- chart or stat-display primitive

A contributor who wants to add a welcome toast has to first build a toast system, which is a conversation, which is the point.

### 12.3 Copy linters

- `content/naturalist/**` — lowercase; present tense; no second person; no exclamation; no gamification lexicon (`streak`, `achievement`, `level`, `unlock`, `badge`, `score`, `progress`, `earn`, `milestone`, `daily`, `welcome back`); at least one specific noun or bird name per template.
- `surfaces/system/**` — sentence capitalization; no naturalist lexicon; every error string contains an actionable next step.
- Both run in CI; misplacement is caught because each string fails the other directory's rules.

### 12.4 Staff tooling — the one place traits are visible

Engineering needs to debug drift. The rules:

- Trait values are visible only in an internal admin tool, behind staff SSO, on an aviary the operator owns or has an approved support ticket for.
- The tool is a separate deployable, on a separate host, not code-split out of the product bundle. It cannot be reached by flipping a flag in the user-facing client, because its code is not in the user-facing client.
- Every access writes an audit record.
- Support responses never quote a trait value to a user. The support macro for "is my bird changing?" answers in naturalist terms.

The account export (§4.4) also contains trait values, and that is fine and deliberate: `accounts_sync.md` specifies it, it is the user's own data, it is delivered as a JSON file rather than a rendered surface, and nothing in the product displays it. The never-exposed rule protects the relationship from becoming stat management; a file the user requested is portability.

---

## 13. Risks

### 13.1 Drift calibration is wrong at launch

**The risk.** Too fast and the product becomes a Tamagotchi the user can move by clicking; too slow and it is a screensaver. `bird_engine.md` says the product lives in a narrow band, and we cannot fully validate the three-week claim before a beta that runs longer than three weeks.

**Compounding factor.** Because personality is stored and never recomputed (correctly — §3.3), changing `α` after launch does *not* retroactively fix the launch cohort. Those birds keep the drift they got. Identity permanence and calibration correction are in genuine tension here.

**Mitigation.**
- Calibration targets are the spec and `α` is derived (§5.2), so nobody hand-tunes toward a feeling.
- The synthetic-cohort harness (§10.5) asserts the targets continuously and in CI.
- The six-week beta floor exists specifically to observe the three-week claim twice.
- If launch calibration proves wrong, `personality_delta` supports a **one-time corrective transform**, computed from the recorded history and applied **monotonically only** — it can add drift a bird should have had; it can never take drift away. Gated on explicit product review, applied once, recorded.
- `α` is a config value with an audited change log, not a code constant.

**Residual.** A too-*fast* launch calibration cannot be corrected for the launch cohort, since correction is monotonic-only. This argues for erring slow. **We will deliberately launch ~15% under the derived `α` and raise it after beta if the three-week signal reads weak** — an under-drifted bird can be corrected upward; an over-drifted one cannot be corrected at all.

### 13.2 Sync correctness

**The risk.** Silent drift loss. `accounts_sync.md` describes the failure precisely: no error, no log line, just a bird that drifts more slowly than it should, and a user who feels something is wrong without being able to name it.

**Mitigation.** Grant-level write exclusivity (§6.1); additive commutative deltas (§6.2); `UNIQUE (bird_id, tick_index)` against double-application; `client_event_id` idempotency; server-derived presence durations; the monotonicity trigger paging on any violation; chaos tests covering duplicate tick execution, out-of-order events, concurrent two-device sessions, worker crash mid-tick, and partial network failure during event submission.

**Residual.** A bug in the drift *function* is not caught by any of these — they guarantee correct plumbing, not correct arithmetic. That is what the synthetic cohort covers.

### 13.3 Audio uncanniness and recognizability collapse

**The risk.** Procedural calls sound synthetic rather than avian; or they sound fine individually but per-bird recognizability collapses before seven birds, which would unravel the bird-count cap and, with it, the per-bird relationship.

**Mitigation.**
- M0 is a hard gate before client work, precisely so this risk is discharged first.
- The signature/expressive split (§5.6) is the structural answer to recognizability under drift and mood: the identifying parameters cannot change, by type.
- ABX testing at 2, 4, and 7 birds, re-run before launch.
- Chorus ducking and spatial spread (§8.4) preserve legibility at density.
- If recognizability degrades between four and seven, the lever is the mix (raise ducking aggressiveness, widen pan spread, increase signature separation in species assignment) — not lowering the cap. The cap is engine-level and user-visible; the mix is not.

**Residual.** "Sounds like a real bird" is a subjective target that a 12-person panel approximates imperfectly. Beta qualitative feedback is the backstop.

### 13.4 Accessibility regression after launch

**The risk.** Reduced-motion mode and narration decay into afterthoughts as features land — the standard fate of accessibility surfaces built as parallel implementations.

**Mitigation.** The single-sampler architecture (§7.6) means reduced-motion is not a parallel implementation and cannot fall behind. Invariant 20 asserts full parity mechanically. Narration and the notebook share one grammar package, so voice cannot fork. The copy linters gate every new string. Standing rule: **any new surface ships with narration copy and keyboard access, or it does not ship.**

**Residual.** Screen-reader *quality* — whether the prose actually feels like the aviary — is not automatable. A manual pass with a screen-reader user is on every release checklist, not just at launch.

### 13.5 Charm erosion

**The risk.** The PRD says outright that the announcement-style UI temptation is what developer instinct most reliably reaches for, and that "just one" is the foothold. Six months of reasonable-looking PRs and the product is a different product with birds in it.

**Mitigation.** §12.2 — the primitives do not exist. §12.3 — the linters. Invariant 21 — session start cannot add text. A named voice owner reviewing every user-visible string. A PR template question: *does this announce, count, or rank anything?*

**Residual.** No test detects a feature that is individually defensible and cumulatively wrong. This one needs a human with standing to say no, which is why the voice owner is a named role and not a rotating duty.

### 13.6 The 500ms budget versus snapshot freshness

**The risk.** The inline snapshot is what makes 500ms achievable, and a cached inline snapshot can be stale — showing an evening palette to a user whose local clock says morning would be a visible, immediate lie about the aviary's continuity.

**Mitigation.** Inline snapshot TTL ≤30s at the edge. Independently, the sky gradient is painted from **CSS using the device's own clock** before any JS runs, so light phase is correct even if the snapshot is stale. The client reconciles by easing (§6.3), never by popping. If the inline snapshot is missing entirely, the quiet field covers the gap.

### 13.7 Autoplay policy

Covered in §8.6. Residual risk: a motionless first-time visitor gets a silent (captioned) aviary. Accepted; the alternative is a modal gate, which is worse in this product than in almost any other.

### 13.8 Tick cost at scale

**The risk.** A literal 60s tick for every account that ever existed is unbounded cost growth against a mostly-dormant tail.

**Mitigation.** The catch-up equivalence in §5.9, guarded by a byte-for-byte property test and auto-disabled on failure. **The risk here is not cost — it is that a future engineer, under cost pressure, "optimizes" catch-up into an approximation.** The equivalence test is the guard, and its failure mode is to fall back to full ticking rather than to accept divergence.

### 13.9 PII leakage

**The risk.** The exact failure `accounts_sync.md` describes: an engineer reaches for email as a convenient unique identifier and six months later PII is sprayed across observability tooling nobody can fully audit.

**Mitigation.** All four enforcement points in §3.6, landing in M1 before any product surface exists. The log scanner pages on any hit. `email_blind_index` is import-restricted to `auth/`.

**Residual.** Third-party services (the email provider) necessarily receive addresses. Vendor scope is documented in the privacy policy, and no vendor receives an account ID alongside an address, so no vendor can correlate a user to aviary state.

### 13.10 Visit feature scope creep

**The risk.** Visits are the natural seed crystal for a social network: co-presence, then "your friend visited" notifications, then a friends list, then discovery.

**Mitigation.** The structural absences do most of the work — no visitor event table (§3.5), no visitor write routes (§4.5), no cross-account aggregates (§10.4). Each addition would require building new infrastructure, which surfaces as a decision instead of a diff. Invariants 16 and 17 hold the line mechanically.

---

## 14. Testing strategy (summary)

| Layer | Coverage |
|---|---|
| Unit | Drift function, mood hazards, perch sampling, call grammar, presence conjunction, caption rendering, narration composition |
| Property | Monotonicity over randomized streams; catch-up equivalence; delta commutativity; greeting non-repetition; pose-walk non-repetition |
| Contract | Snapshot serializer field allowlist; API schema; grant matrix; metric registry dimension check |
| Integration | Auth lifecycle, tick exactly-once, event idempotency, two-device coherence, visit revocation timing |
| Chaos | Duplicate tick execution, out-of-order events, worker crash mid-tick, network partition during event submit, clock skew, suspended-device resume |
| Performance | Bundle size per chunk; first-bird on throttled profile; 30-min frame soak; 30-min memory soak; weekly real-laptop run |
| Accessibility | axe on all DOM surfaces; narration snapshot tests; contrast sampling across the palette ramp; keyboard-path integration tests; manual screen-reader pass per release |
| Charm invariants | §12.1, all 24, in a named suite that must be green to deploy |
| Listening | ABX panel at 2, 4, and 7 birds, at M0 and pre-launch |

---

## 15. Judgment calls and open items

The PRD instructs us to make defensible calls where it is silent and to note them. These are the calls; each is independently overturnable without re-deriving the plan.

| # | Call | Rationale | Reversibility |
|---|---|---|---|
| A1 | Presence activity window **5 minutes**; ping cadence **20s**; server credits ≤25s per ping | `interactions.md` says lean long because watching without moving is the product; 20s pings make the credit granularity fine enough that a closing window loses ≤25s | Config |
| A2 | Tick **60s**, with catch-up equivalence for dormant aviaries | PRD says "~once per minute"; equivalence preserves the property exactly (§5.9) | Config + feature flag |
| A3 | Concurrent multi-device presence counted as **union, not sum** | Same corruption class as "tab open" = presence; there is one user | Code, one function |
| A4 | Third-bird pacing **90 / 210 / 370 / 580 / 840 days** | `bird_engine.md`: "a few months" → third, "a year-old aviary may have grown to five or six" | Config |
| A5 | Autoplay block → **silence + captions**, no enable-sound prompt | A prompt is an announcement; the PRD already establishes silence+captions as the correct degradation | Design decision |
| A6 | **Canvas 2D**, not WebGL | 7 birds + 4 layers does not need a GPU pipeline; WebGL costs core-chunk size and context-loss handling against a 500ms budget | Significant rework |
| A7 | Mood set finalized as **wary, content, curious, drowsy, alert** | PRD lists exactly these and says the set is finalized in implementation | Additive |
| A8 | **Polling**, not WebSockets | Pull triggers are sparse and tick cadence is 60s; sockets add state for imperceptible latency gain | Additive |
| A9 | Raw event retention **90 days** | Deltas are folded; longer retention serves nothing the user asked for | Config |
| A10 | Plumage transported as **5 quantized steps** | A continuous value is a trait scalar in disguise; 5 steps matches "visible drift" granularity | Config |
| A11 | Narration composed **client-side** from the shared grammar package | PRD permits either; client-side gives low latency and guarantees voice parity with the notebook | Moderate |
| A12 | Species pool of **6**, one nocturnal | PRD: "about six species," one nightjar-like signature | Additive |
| A13 | Notebook target **1 entry per 2–4 days**, threshold rises with recent density | "Roughly one entry every few days"; density-adaptive threshold implements "preserve sparsity even for very active users" | Config |
| A14 | Launch `α` **15% under** the derived value | Under-drift is monotonically correctable; over-drift is not (§13.1) | Config |
| A15 | Timezone updates require **6h** of consistent reporting; day-phase clamped non-decreasing | Avoids VPN flapping and evening un-happening | Config |
| A16 | Visit revocation effective at the visitor's **next poll (≤60s)** | PRD says "next state-snapshot pull"; this is that, literally | — |

**Open items requiring a decision before or during M4**, none blocking earlier work:

1. **Exact species set and their motif libraries** — needs the audio designer, informed by M0 ABX results.
2. **Design-system palette values and focus-ring treatment** — visual designer; this plan specifies the slots and the automated validation.
3. **Naturalist content package v1** — the notebook candidate types and their template families are the product's charm engine and are authored by the voice owner, not implemented ad hoc.
4. **Email vendor selection** — must support per-recipient suppression and not require account IDs alongside addresses (§13.9).
5. **Whether the offer library's song fragments are designer-authored motifs or drawn from species grammars** — leaning designer-authored, so the offer is legibly *from the user* rather than sounding like another bird.

---

## 16. What success looks like

Three checks that the plan, if executed, produced the product the PRD describes:

1. **A user opens the tab on a slow phone and does not perceive a load.** The first frame has birds mid-action, a bird notices them within a second or two, and nothing tells them they arrived.
2. **A user comes back after two weeks away and finds birds that are quieter, not birds that are hurt.** Nothing in the product mentions the absence. The notebook has a few entries from days they weren't there.
3. **A user three weeks in says, unprompted, that Pip comes closer than she used to** — and there is no number anywhere in the product they could have read that from.
