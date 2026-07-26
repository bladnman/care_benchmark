# Pocket Aviary — v1 Implementation Plan

**Status:** phase-1 plan, ready for engineering execution.
**Audience:** a frontier engineering team (~6 engineers, 1 visual designer, 1 sound designer, 1 writer) executing without further clarification from the PRD authors.
**Deliverable of this document:** the plan. No product code is written here.

---

## 0. How to read this plan

The PRD has five principles that behave like invariants, not aspirations. Most of the engineering decisions below exist because a specific principle would otherwise be violated by the obvious implementation. Where that is the case, the invariant is named inline as **[INV-n]** so it can be traced into tests. The full invariant register is §14, and every invariant has at least one automated check.

| ID | Invariant | Primary enforcement |
|---|---|---|
| INV-1 | Aliveness: no load state, no canned cue, no looped audio, no cycle animation | §7 render, §8 audio, §12 CI gates |
| INV-2 | Notice, never announce: no toast/banner/notification/welcome copy | §6 API deny-list, §12 refusals suite |
| INV-3 | No gamification, ever: no counter derivable from any surface or endpoint | §5 data model, §6 API, §11 observability, §12 refusals suite |
| INV-4 | Monotonic drift toward expressive; neglect never reduces a trait | §5 DB trigger, §7 drift function |
| INV-5 | Server is the only writer of personality; no last-write-wins | §5 DB grants, §9 sync |
| INV-6 | Personality never exposed numerically to the user | §6 snapshot shape, §12 schema test |
| INV-7 | Bird identity is stable forever | §5 immutable `bird_id`, migration policy §5.9 |
| INV-8 | Presence = visible ∧ focused ∧ recent input, and is credited honestly | §7.2 presence accounting |
| INV-9 | Per-bird interaction data never leaves the simulation boundary | §11 telemetry boundary, §13 infra policy |
| INV-10 | Email is PII: synthetic UUID everywhere else | §5.1 accounts |
| INV-11 | Accessibility is a designed surface shipping with v1, not after | §10, §12.6 definition-of-done |
| INV-12 | Voice split: naturalist product surface, matter-of-fact system surface | §10.6 prose package, §12 lint |

Section §15 lists every judgment call this plan makes where the PRD is deliberately open, so reviewers can overrule a specific decision without re-deriving the plan.

---

## 1. Scope

### 1.1 In scope for v1

**Accounts and identity**
- Single-user accounts, one canonical aviary per account.
- Email + magic-link sign-in (15-minute expiry, single-use), per-device revocable sessions, verified email change.
- Account export (JSON, emailed link), soft delete with 30-day recovery then hard delete.

**Bird engine (server-authoritative)**
- Five-trait hidden personality vector per bird with monotonic slow drift.
- Six-state mood machine per bird on a fast timescale, persisting across sessions.
- Six-species pool with per-species silhouette, palette, and call-motif library.
- Two starter birds at adoption, age-gated offers up to a hard cap of seven.
- Bird-to-bird interaction: call-and-answer, mood contagion, emergent chorus.
- Server-side simulation tick (~60s) that runs whether or not a client is connected.

**Client experience**
- Single horizontal non-panning scene, three perch zones, responsive 320–2560 CSS px.
- First frame already in motion; no spinner, no entry animation, no fade-from-static.
- Day/night cycle anchored to the user's local time; rare ambient weather; ambient ornament drift; subtle parallax.
- Thin fading top bar: account/settings, accessibility settings, notebook, offer. Nothing else.
- Return-greeting (one bird, varied by boldness/mood/absence length, procedurally varied, staggered when multiple).
- Listen-in (gradual re-balance, never mute), offer (seed / song fragment / still pool, per-bird cooldown), settle (with 5s undo).
- Field notebook: sparse, auto-generated, read-only, infinite scrollback.

**Audio**
- Fully procedural client-side WebAudio call synthesis from a motif grammar; real-time chorus mixing; listen-in mix ramps; silence-plus-captions fallback when WebAudio is unavailable.

**Accessibility (ships with v1, not after)**
- Screen-reader naturalist prose narration on a slow cadence with a priority lane.
- Reduced-motion mode as a distinct designed render mode.
- Opt-in call captions generated from the actual synthesized call.
- Full keyboard navigation, visible focus treatment against all aviary lighting states, WCAG AA on all copy.

**Social (off by default)**
- Per-invite email invitations, read-only ambient visitor view, immediate revocation, 30-day invite expiry, on-demand visit log, opt-in visit notification toggle (default off).

**Operations**
- Aggregate-only telemetry and RUM, synthetic browser fleet, perf and a11y budgets as CI gates, privacy boundary enforced architecturally.

### 1.2 Explicitly out of scope for v1

Native iOS/Android apps or any native-driven protocol concession. Payments or tiers. Shared, team, or household aviaries. Multiple aviaries per account. Customizable or purchasable scenes. Public discovery, profiles, follows, feeds, ratings, or featuring. Leaderboards or any cross-account ranking — **including the underlying aggregates that would make one possible** (§11.3). Achievements, badges, levels, XP, scores, streaks, "days visited," green-dot calendars, "birds adopted: N," any visit-frequency surface, in any tier, setting, or debug view. Push notifications or marketing email about the aviary (transactional email only: magic link, invite, export link, account-deletion confirmation). Hunger, health, happiness meters, death, or any distress state. Co-presence, chat, comments, avatars, or visitor identity rendered in the scene. Recorded-audio playback paths of any kind. Password or SSO auth. Support for browsers older than the last two majors of Chrome/Safari/Firefox/Edge.

### 1.3 Non-goals treated as engineering constraints

Each PRD non-goal is converted into something a machine can check, because "we agreed not to" does not survive eighteen months of contributors:

- **No gamification** → no schema column, no API field, and no client string that expresses a count of user behavior. Presence-seconds exist in the database as drift input and are unreachable through every read path (§6.6). A CI test asserts the aviary bundle contains no string matching the gamification lexicon (§12.5).
- **No Tamagotchi** → the drift function is structurally incapable of negative movement (§7.3), enforced at the database layer by a trigger, not by the calling code.
- **No social network** → no cross-account read path exists. The visit feature is the only code path where account A's data reaches user B, it is a single read-only endpoint, and it is gated on an unexpired unrevoked invite token.
- **No native app** → the API is designed for a browser client (cookie sessions, HTML-inlined bootstrap snapshot, CDN-cacheable snapshot reads). We do not add token-based auth flows or offline-first mutation semantics "in case."
- **No notification surface** → the transactional mailer has a hard allow-list of four template IDs. Adding a fifth requires touching a file whose header says why.

### 1.4 Success criteria for v1

1. First bird visible < 500ms on a mid-tier Android over 4G, with no load state ever rendered (§11.1).
2. A listener who has spent two weeks with one bird identifies that bird's call blind at ≥ 80% at 5 birds and ≥ 70% at 7 birds (§8.7 listening study).
3. Instrumented drift is measurable after ~1 week of regular visits and rated "noticeably different" by ≥ 60% of dogfooders after ~3 weeks (§7.4 calibration).
4. A screen-reader-only session and a reduced-motion session are both rated by external auditors as delivering the product's affective core, not a fallback (§10.8).
5. 30-minute session: p95 frame ≤ 16.7ms on the reference 5-year-old laptop, heap growth ≤ 2MB.
6. Zero surfaces in the shipped product that announce, count, rank, or number.

---

## 2. Architecture

### 2.1 Service shape

Five deployable units. The split is driven by two hard boundaries — *only the simulation writes personality* (INV-5) and *per-bird data never reaches analytics* (INV-9) — plus one performance boundary, the edge-rendered first paint.

```
                    ┌───────────────────────────────┐
  browser ─────────▶│ edge (CDN worker)             │  HTML + inlined bootstrap snapshot
                    │  - static assets, cache       │  + immutable asset serving
                    └──────────────┬────────────────┘
                                   │ snapshot fetch (ETag=tick_seq)
                    ┌──────────────▼────────────────┐
                    │ aviary-api  (stateless)       │  auth, snapshot read, event ingest,
                    │  role: api_rw / sim_ro        │  notebook read, settings, visits, export
                    └───────┬───────────────┬───────┘
                            │               │
              write-only    │               │  read-only
            event log       │               │  bird_personality (SELECT only)
                    ┌───────▼───────────────▼───────┐      ┌──────────────────┐
                    │ Postgres (simulation primary) │◀────▶│ Redis            │
                    │  accounts, aviaries, birds,   │      │  snapshot cache  │
                    │  personality, mood, events,   │      │  rate limits     │
                    │  notebook, invites, sessions  │      └──────────────────┘
                    └───────────────▲───────────────┘
                                    │ SOLE WRITER of personality + mood
                    ┌───────────────┴───────────────┐
                    │ sim-worker  (tick engine)     │  leases aviaries, folds events,
                    │  role: sim_rw                 │  drifts, transitions, schedules calls,
                    │                               │  proposes notebook candidates
                    └───────────────┬───────────────┘
                                    │
                    ┌───────────────▼───────────────┐   ┌──────────────────────────────┐
                    │ notebook-writer (in-process   │   │ mailer (4 templates, allow-  │
                    │ module of sim-worker, own     │   │ listed): magic link, invite, │
                    │ cadence + sparsity budget)    │   │ export link, deletion notice │
                    └───────────────────────────────┘   └──────────────────────────────┘

  ╔══════════════════ hard boundary — no network route, no shared credential ══════════════╗
                    ┌───────────────────────────────┐
                    │ telemetry / RUM (separate     │  aggregate metrics only, schema-enforced
                    │ store, separate VPC subnet)   │  label deny-list, no account dimension
                    └───────────────────────────────┘
```

**Why `sim-worker` is a separate deployable rather than a cron inside `aviary-api`:** it holds the only database role with `UPDATE` on personality and mood. If it shared a process with the request path, that grant would be reachable from any handler, and INV-5 would be a code convention instead of a permission. It also lets us scale tick throughput independently of request traffic, which matters because tick load is a function of account count and request load is a function of active sessions.

**Why `notebook-writer` is a module inside `sim-worker` rather than its own service:** it needs the same transactional view of state and events that the tick already holds, and it runs at a far slower cadence (candidate evaluation once per hour per aviary). A separate service would re-read the same rows for no isolation benefit. It does get its own module boundary, its own sparsity budget, and its own prose review process.

### 2.2 Client/server split

The dividing line is **discrete versus continuous state**.

| Owned by server (canonical, persisted) | Owned by client (derived, disposable) |
|---|---|
| Personality vector, per-trait filter state | Sub-pixel bird position, easing curves |
| Mood, mood dwell timers | Micro-motion oscillator phase advance within a cycle |
| Perch zone + slot assignment, transition start/duration | Flight arc shape, wingbeat overlay |
| Call intents (who, when, which motif, which seed) | Audio synthesis of each intent, voice allocation, mixing |
| Weather events (kind, start, end) | Leaf/feather ornaments, parallax offsets |
| Aviary settled state, offer cooldowns | Top bar fade state, hover, local listen-in target |
| Notebook entries, greeting assignment | Narration composition, caption text, focus ring |

Two consequences worth stating because they will otherwise be re-litigated:

1. **Ornaments are not simulated.** Leaves and feathers have no server state (the PRD says so explicitly). They are seeded from `(aviary_id, wall-clock minute)` so two devices look *similar* without any synchronization protocol. We never try to make two devices frame-identical; nothing in the product depends on it, and attempting it would put a real-time channel in the critical path for decoration.
2. **Calls are scheduled server-side and voiced client-side.** The server writes call *intents* into a forward window (`now` → `now + 180s`) inside the snapshot. The client synthesizes them. This is what makes "calls already audible on the first frame" achievable without a round trip, makes chorus grouping a server decision (it needs both birds' vocal-frequency traits, which never leave the server), and keeps synthesis — the expensive, variation-rich part — on the client where the bundle-budget argument put it.

### 2.3 Render pipeline boundary

The snapshot is a **description of the aviary at a server timestamp, with enough forward information to keep rendering for three minutes without another byte.** The client is a pure function of `(latest snapshot, wall clock, local settings, viewport)`. It has no authoritative state of its own and no state that survives a reload except the event outbox and user settings cache.

That property is what buys us three things at once: the first frame can be drawn from an HTML-inlined snapshot with zero fetches; a stale or offline client degrades into *continuing* rather than *freezing* (INV-1); and reconciliation on a new snapshot is a retarget, never a teleport.

### 2.4 Technology choices

| Layer | Choice | Rationale |
|---|---|---|
| Client language | TypeScript, strict | Shared types with the API and the prose package |
| Aviary renderer | Canvas 2D, layered offscreen canvases | 7 birds + subtle parallax does not need WebGL; WebGL costs bundle bytes, context-loss handling, and older-GPU driver risk against a 2MB ceiling and a 5-year-old-laptop target. Revisit only if §12.4 soak fails. |
| Aviary UI framework | None on the critical path (custom elements + direct DOM for the top bar) | Every framework KB competes with the 500ms budget. Framework use is permitted in code-split non-critical routes (settings, notebook, visit management). |
| Audio | WebAudio, hand-built graph, pre-allocated voice pool | Procedural mandate (INV-1); pooling is how §11.1 "no memory growth" is met |
| Server language | TypeScript (Node) for `aviary-api`; TypeScript for `sim-worker` | One language, shared engine types, shared prose package between client narration and server notebook. The tick is arithmetic on small structs; Node is fast enough at the batch sizes in §7.9. |
| Primary store | Postgres 16 | Row-level grants give us INV-5 for free; partitioned append-only event log; `SKIP LOCKED` gives us tick leasing without a broker |
| Cache | Redis | Snapshot cache keyed by aviary, rate limits, invite-token lookups |
| Queue | None at v1 | The event log *is* the queue (tick reads it in `ingest_seq` order). Adding Kafka would add a second ordering authority and a partition key we would be tempted to derive from email (INV-10). |
| Email | Two providers behind one interface, automatic failover | A broken magic-link path is total account loss; this is the only single point of failure that locks users out of their own birds |
| Edge | CDN worker (Cloudflare-class) | HTML assembly with inlined bootstrap snapshot; asset immutability; geographic snapshot read caching |

**Deliberate omission:** no WebSocket, no SSE, no realtime channel at v1. The product's cadence is a 60s tick and a client that interpolates; polling on visibility change plus a 45s keepalive is sufficient, cheaper, and removes an entire class of connection-state bugs. The one place this is felt is a visitor watching a host's aviary — they see changes up to 45s late, which is invisible at this cadence.

---

## 3. Domain model, conceptually

Before the schema: the engine is three nested clocks.

- **Weeks (personality).** A five-dimensional vector per bird, monotonically non-decreasing, updated only by the tick, never displayed as a number. It changes the *distribution* of everything below it.
- **Hours-to-a-day (mood).** A six-state machine per bird, sampled by the tick from a score built out of time-of-day, recent interactions, weather, contagion, and personality bias. It changes *which* behaviors are likely right now.
- **Seconds (behavior).** Perch choice, call intents, micro-motion activity. Concrete, forward-scheduled, and the only layer the client renders.

Reading downward, personality biases mood biases behavior. Reading upward, behavior is what the user sees, mood is what they read from the motion, and personality is what they notice only by comparing to memory. Any implementation where the user can read a layer directly — a mood label, a trait number — collapses two layers into one and removes the noticing, which is the product.

---

## 4. Two vocabulary rules that are engineering rules

The PRD's vocabulary section is not decoration; the terms leak into identifiers, and identifiers leak into UI copy through templating.

1. **Code uses the PRD's nouns.** `bird` (never `pet`, `creature`, `entity`, `character`), `call` (never `song`, `sound`, `chirp`, `sfx`), `listen_in` (never `solo`, `select`, `focus_bird` as a mix concept), `offer`, `settle`, `presence`, `drift`, `tick`, `aviary`. Enforced by a lint rule with a banned-identifier list scoped to the domain packages. A `sfx/` directory would eventually produce a caption reading "sound effect played."
2. **No user-facing string is written inline.** All product-surface prose comes from the shared `@aviary/prose` package (§10.6), which is voice-split: `prose/naturalist/*` and `prose/system/*`. A lint rule forbids string literals in JSX/DOM-text positions in the client and in mailer templates. This is how INV-12 survives contributors — you cannot add a "Welcome back!" without adding it to a file that a writer owns and whose tests reject announcement patterns.

---

## 5. Data model

Postgres. All timestamps `timestamptz`, stored UTC. All IDs UUID v7 unless noted (time-ordered, so they index well and give us a natural sequence for the event log).

### 5.1 `accounts`

```sql
CREATE TABLE accounts (
  account_id        uuid PRIMARY KEY,              -- synthetic; the ONLY identifier used anywhere else [INV-10]
  email_ciphertext  bytea NOT NULL,                -- AES-256-GCM, key in KMS, rotated by key-id
  email_key_id      text  NOT NULL,
  email_blind_index bytea NOT NULL UNIQUE,         -- HMAC-SHA256(email_normalized, pepper) for sign-in lookup
  email_pending_ciphertext bytea,                  -- email change: old address works until new one verifies
  email_pending_blind_index bytea,
  tz_iana           text,                          -- last-observed client timezone; a hint, refreshed on session
  created_at        timestamptz NOT NULL,
  deleted_at        timestamptz,                   -- soft delete; hard purge job at +30d
  settings          jsonb NOT NULL DEFAULT '{}'    -- audio_on, captions_on, reduced_motion, visit_notify
);
```

**Email handling.** Email appears in exactly two places: `accounts.email_ciphertext` and `visit_invites.visitor_email_ciphertext`. Every other reference in every service, log line, metric label, trace attribute, cache key, and partition key is `account_id`. Sign-in needs to look an address up, which is why the blind index exists — an HMAC with a server-held pepper, not a bare hash, so an attacker with a database dump cannot dictionary the address space. The pepper lives in KMS and is not in the database.

This is the PRD's "single most important boring detail," and the reason it is a schema decision rather than a code-review guideline is that it is unretrofittable. A CI check (§12.5) fails the build if the string `email` appears as a metric label, log field, cache key prefix, or foreign-key column name outside these two tables.

**Timezone.** Day/night follows the user's *local* time. The client computes lighting from its own clock (authoritative for rendering, always correct, zero staleness). `tz_iana` exists so the *server* can evaluate time-of-day mood affinities while no client is connected, which the tick must do. When they disagree — user flies to Tokyo — the client is right for lighting, the server catches up on the next session's session-refresh, and mood re-anchors within a couple of ticks. Documented drift, bounded, invisible.

### 5.2 `sessions`, `magic_links`

```sql
CREATE TABLE sessions (
  session_id     uuid PRIMARY KEY,
  account_id     uuid NOT NULL REFERENCES accounts,
  token_hash     bytea NOT NULL UNIQUE,      -- SHA-256 of the cookie value; raw token never stored
  device_label   text,                       -- coarse UA-derived, e.g. "safari on iphone"; for the revoke list
  created_at     timestamptz NOT NULL,
  last_seen_at   timestamptz NOT NULL,
  revoked_at     timestamptz
);

CREATE TABLE magic_links (
  token_hash     bytea PRIMARY KEY,          -- SHA-256; 32 bytes of CSPRNG in the link
  account_id     uuid NOT NULL REFERENCES accounts,
  issued_at      timestamptz NOT NULL,
  expires_at     timestamptz NOT NULL,       -- issued_at + 15 min
  consumed_at    timestamptz                 -- single-use; set atomically via UPDATE ... WHERE consumed_at IS NULL
);
```

Consumption is `UPDATE magic_links SET consumed_at = now() WHERE token_hash = $1 AND consumed_at IS NULL AND expires_at > now() RETURNING account_id`. Zero rows returned is the replay/expiry case and produces the matter-of-fact sign-in error, not a naturalist one.

### 5.3 `aviaries`

```sql
CREATE TABLE aviaries (
  aviary_id            uuid PRIMARY KEY,
  account_id           uuid NOT NULL UNIQUE REFERENCES accounts,   -- one aviary per account at v1
  created_at           timestamptz NOT NULL,     -- aviary AGE: the only input to new-bird availability
  tick_seq             bigint NOT NULL DEFAULT 0,
  last_tick_at         timestamptz NOT NULL,
  next_tick_at         timestamptz NOT NULL,
  tick_lease_until     timestamptz,
  event_watermark      bigint NOT NULL DEFAULT 0, -- highest interaction_events.ingest_seq folded in
  settled_until        timestamptz,               -- non-null => aviary is in the settled lighting state
  weather_kind         text,                      -- null | 'rain' | 'wind'
  weather_started_at   timestamptz,
  weather_ends_at      timestamptz,
  next_weather_at      timestamptz NOT NULL,
  next_species_offer_at timestamptz,              -- null once at cap
  rng_stream           bigint NOT NULL             -- per-aviary PRNG salt, fixed at creation
);
```

`created_at` is load-bearing: new birds become available on aviary *age* and nothing else (INV-3). There is deliberately no `visit_count`, `session_count`, `total_presence_seconds`, or `last_seen_at` on this table. Presence lives in a derived table the read path cannot reach (§5.7).

### 5.4 `birds`

```sql
CREATE TABLE birds (
  bird_id      uuid PRIMARY KEY,          -- IMMUTABLE FOREVER [INV-7]
  aviary_id    uuid NOT NULL REFERENCES aviaries,
  species_id   text NOT NULL,             -- references the in-code species pool
  name         text NOT NULL,             -- user-assigned, renameable, no effect on anything else
  adopted_at   timestamptz NOT NULL,
  scene_order  smallint NOT NULL,         -- stable left-to-right ordering for keyboard nav
  timbre_seed  bigint NOT NULL            -- derived once from bird_id; fixes the call signature forever
);
CREATE INDEX ON birds (aviary_id);
```

`birds` rows are never deleted (except by account hard-delete) and `bird_id`, `species_id`, `adopted_at`, and `timbre_seed` are never updated. `name` and `scene_order` are the only mutable columns. **`timbre_seed` is the mechanism behind "you know Pip by ear"** (§8.4): it fixes the bird's harmonic profile, formant filter, base pitch band, and preferred syllable transitions at adoption, and drift never touches it.

Migration policy for INV-7: adding a species is additive. Changing a species' visual or motif library is permitted and does not create a new bird. There is no code path, migration, admin tool, or support runbook that reassigns a `bird_id`, changes a bird's `species_id`, or recreates a bird row. If a species must be retired, its existing birds keep it and its assets ship forever. This is written into the migration review checklist because it is the failure the PRD calls "the worst possible failure of this product."

### 5.5 `bird_personality`

```sql
CREATE TABLE bird_personality (
  bird_id            uuid PRIMARY KEY REFERENCES birds,
  boldness           numeric(6,5) NOT NULL,
  social_warmth      numeric(6,5) NOT NULL,
  vocal_frequency    numeric(6,5) NOT NULL,
  plumage_saturation numeric(6,5) NOT NULL,
  curiosity          numeric(6,5) NOT NULL,
  filter_state       jsonb NOT NULL,     -- per-trait low-pass accumulator + per-UTC-day saturation counters
  ceilings           jsonb NOT NULL,     -- per-trait species ceiling, fixed at adoption
  version            bigint NOT NULL DEFAULT 0,
  updated_at         timestamptz NOT NULL
);

-- [INV-5] only the simulation role may write
REVOKE INSERT, UPDATE, DELETE ON bird_personality FROM api_rw;
GRANT  SELECT                  ON bird_personality TO   api_rw;
GRANT  INSERT, UPDATE          ON bird_personality TO   sim_rw;

-- [INV-4] monotonicity is a database property, not a code convention
CREATE FUNCTION assert_monotonic_drift() RETURNS trigger AS $$
BEGIN
  IF NEW.boldness           < OLD.boldness
  OR NEW.social_warmth      < OLD.social_warmth
  OR NEW.vocal_frequency    < OLD.vocal_frequency
  OR NEW.plumage_saturation < OLD.plumage_saturation
  OR NEW.curiosity          < OLD.curiosity THEN
    RAISE EXCEPTION 'INV-4 violation: trait decrease on bird %', NEW.bird_id;
  END IF;
  IF NEW.version <> OLD.version + 1 THEN
    RAISE EXCEPTION 'INV-5 violation: non-sequential personality version on bird %', NEW.bird_id;
  END IF;
  RETURN NEW;
END $$ LANGUAGE plpgsql;
```

Traits are normalized to `[0, 1]`. Species-specific per-trait ceilings (a shy species caps boldness around 0.85) are frozen at adoption so that six weeks of presence does not homogenize the pool into six identical maximally-bold birds — which is the natural failure mode of a monotonic drift function and would quietly destroy per-bird distinctiveness, the thing the whole engine exists to produce.

`filter_state` holds the low-pass accumulator so drift is **never recomputed from the event log** (the PRD forbids deriving personality from session history). The event log is a source of *increments*, consumed once, watermarked. Trimming the log for retention can never affect a vector.

The trigger raises rather than clamps deliberately: a clamp would silently absorb the bug that INV-4 exists to prevent. A raise fails one tick for one aviary, pages us, and leaves the vector intact.

### 5.6 `bird_state` (fast timescale)

```sql
CREATE TABLE bird_state (
  bird_id           uuid PRIMARY KEY REFERENCES birds,
  mood              text NOT NULL,        -- wary|content|curious|drowsy|alert|settled
  mood_since        timestamptz NOT NULL,
  mood_min_until    timestamptz NOT NULL, -- hysteresis floor; prevents flicker
  perch_zone        text NOT NULL,        -- front|middle|back
  perch_slot        smallint NOT NULL,
  perch_from        jsonb,                -- {zone,slot} when a transition is in flight
  perch_move_at     timestamptz,
  perch_move_ms     integer,
  activity          text NOT NULL,        -- preen|scan|tilt|shuffle|rest|call|drink|bathe|watch
  activity_phase    real NOT NULL,        -- 0..1, so the client resumes mid-cycle [INV-1]
  activity_since    timestamptz NOT NULL,
  last_call_at      timestamptz,
  offer_cooldown_until timestamptz,
  greeted_at        timestamptz,          -- for "first greeter today" notebook candidates
  updated_at        timestamptz NOT NULL
);
-- same grants as bird_personality: sim_rw writes, api_rw reads
```

`activity_phase` is small and important. Without it the client has to start every micro-motion at phase 0, so every bird begins mid-preen-*start* on load — a subtle but real "the app just woke up" tell. With it, the first frame is genuinely mid-action (INV-1).

### 5.7 Event log and presence

```sql
CREATE TABLE interaction_events (
  ingest_seq   bigserial,                 -- server-assigned; THE canonical order [INV-5]
  event_id     uuid NOT NULL,             -- client-generated; idempotency key
  aviary_id    uuid NOT NULL,
  bird_id      uuid,
  session_id   uuid NOT NULL,             -- for multi-device presence de-duplication
  kind         text NOT NULL,             -- presence_ping|listen_in_start|listen_in_end|offer|
                                          -- offer_outcome|settle|settle_undo|greeting_seen
  client_ts    timestamptz NOT NULL,      -- advisory only, never used for credit
  server_ts    timestamptz NOT NULL,
  payload      jsonb NOT NULL,
  PRIMARY KEY (server_ts, ingest_seq)
) PARTITION BY RANGE (server_ts);         -- daily partitions, dropped after 45 days

CREATE UNIQUE INDEX ON interaction_events (event_id);   -- retry-safe ingest

CREATE TABLE presence_credit (            -- written by the tick, read by NOTHING on the API path
  aviary_id   uuid NOT NULL,
  day_utc     date NOT NULL,
  seconds     integer NOT NULL,
  PRIMARY KEY (aviary_id, day_utc)
);
REVOKE ALL ON presence_credit FROM api_rw;   -- [INV-3] unreachable from every read path
```

Three decisions embedded here:

- **`ingest_seq` (server-assigned) is the ordering authority, not `client_ts`.** Two devices with skewed clocks cannot reorder each other's events, and a client cannot backdate an event to win an ordering.
- **The event log has a 45-day retention.** It can, because personality is stored, not derived. Retention on interaction history is also a privacy posture: the PRD says this history is the user's, and we keep the minimum needed to run the simulation with a replay margin.
- **`presence_credit` is revoked from the API role.** Presence-time is real, it is the dominant drift input, and it is *exactly* the number a future contributor would surface as "days visited" or "hours watched." Making it unreadable by the request path means shipping that feature requires a migration, a grant change, and a conversation (INV-3).

### 5.8 Notebook, visits

```sql
CREATE TABLE notebook_entries (
  entry_id    uuid PRIMARY KEY,
  aviary_id   uuid NOT NULL REFERENCES aviaries,
  local_date  date NOT NULL,          -- the user's local day, for the "tuesday —" prefix
  body        text NOT NULL,          -- naturalist prose, final, immutable
  template_id text NOT NULL,          -- for anti-repetition scoring and corpus QA
  subjects    uuid[] NOT NULL,        -- bird_ids referenced
  created_at  timestamptz NOT NULL
);
CREATE INDEX ON notebook_entries (aviary_id, created_at DESC);
-- read-only to users: no UPDATE/DELETE endpoint exists; api_rw has SELECT only

CREATE TABLE visit_invites (
  invite_id                 uuid PRIMARY KEY,
  aviary_id                 uuid NOT NULL REFERENCES aviaries,
  visitor_email_ciphertext  bytea NOT NULL,
  visitor_email_blind_index bytea NOT NULL,
  token_hash                bytea NOT NULL UNIQUE,
  created_at                timestamptz NOT NULL,
  expires_at                timestamptz NOT NULL,   -- created_at + 30 days
  first_used_at             timestamptz,
  revoked_at                timestamptz
);

CREATE TABLE visit_sessions (
  visit_session_id uuid PRIMARY KEY,
  invite_id        uuid NOT NULL REFERENCES visit_invites,
  started_at       timestamptz NOT NULL,
  last_pull_at     timestamptz NOT NULL,            -- duration = last_pull - started, approximate by design
  ended_reason     text                             -- null|expired|revoked|idle
);
```

`visit_sessions` rows are never joined to `interaction_events` and the tick never reads them. That is the enforcement of "a visitor sitting and watching for an hour does not drift the host's birds" — not a filter in the drift function, but the absence of a join. Visitor pulls produce no event-log rows at all (§6.5).

### 5.9 Species pool: config, not data

The six species live in `packages/engine/species/*.ts` as versioned config: silhouette geometry references, default palette ramp per plumage tier, motif library, per-trait ceilings, base call rate, night-activity multiplier (the nightjar-like species has a non-zero one), and perch-zone bias. Config rather than rows because they are code-coupled (a motif references synthesis primitives) and because a DB-editable species pool invites production edits that would change an existing bird's voice without review.

### 5.10 Account deletion and export

**Soft delete** sets `accounts.deleted_at`. Effects: sign-in still works (recovery requires it), `sim-worker` stops ticking the aviary, snapshot reads return the matter-of-fact recovery surface, outstanding invites are treated as revoked. Any signed-in page offers "I changed my mind," which clears `deleted_at` and resumes ticking; the aviary resumes from its stored state — birds unchanged, drift intact (INV-7).

**Hard delete** at +30 days: a purge job deletes, in FK-safe order, `visit_sessions`, `visit_invites`, `notebook_entries`, `bird_state`, `bird_personality`, `birds`, `presence_credit`, `aviaries`, `sessions`, `magic_links`, the account row, and all `interaction_events` rows for the aviary across live partitions. Two things that make this actually complete:

- **Backups.** Point-in-time recovery retention is set to 35 days so that a hard delete at day 30 has fully aged out of every restorable snapshot by day 65, and this window is documented in the privacy policy rather than left implicit.
- **Telemetry.** There is nothing to delete, because no telemetry record has an account dimension (§11.3). The privacy boundary pays for itself here: deletion completeness is a property of the architecture instead of a cross-system erasure workflow that is always one forgotten pipeline from being wrong.

**Export** is a job that renders a JSON document (birds with `bird_id`/name/species/adopted_at, current personality vectors, current moods, all notebook entries, settings, visit log) to object storage behind a signed URL with a 7-day expiry, emailed to the verified address.

*Named tension:* the export contains personality vector numbers, and INV-6 says the user never sees them. The PRD explicitly lists vectors in the export, so we honor it, with the reading that INV-6 governs *product surfaces* — panels, debug views, tooltips, any rendered UI — while the export is a data-portability artifact. The engineering consequence is a hard line: no client code ever parses an export, there is no in-app viewer, no "import" path, and the export endpoint is not reachable from the aviary route bundle. The vectors are in a file the user can open if they choose; they are not in the product.

---

## 6. API surface

REST/JSON over HTTPS. `HttpOnly; Secure; SameSite=Lax` cookie sessions. Versioned under `/v1`. All error bodies carry a `code` the client maps to prose from `prose/system/*` — the server never sends user-facing English, so the voice split (INV-12) cannot be violated by a backend engineer writing an error message.

### 6.1 Auth

| Method | Path | Notes |
|---|---|---|
| `POST` | `/auth/magic-link` | Body `{email}`. **Always** 202, identical latency and body whether or not the account exists (no account enumeration). Rate-limited per-email (5/hour) and per-IP (20/hour) in Redis. Creates the account if new and enqueues the adoption flow. |
| `GET` | `/auth/consume?t=…` | Validates single-use + unexpired, creates a session, sets cookie, 303 → `/`. Failure renders the matter-of-fact sign-in error page, not a redirect loop. |
| `POST` | `/v1/auth/sign-out` | Revokes the current session. |
| `GET` | `/v1/sessions` | List of `{session_id, device_label, created_at, last_seen_at, current: bool}`. |
| `DELETE` | `/v1/sessions/:id` | Revokes. Revoked sessions fail their next request with `session_revoked`. |
| `POST` | `/v1/account/email` | Starts a verified email change; old address keeps working until the new one confirms. |

### 6.2 The aviary document (`GET /`)

Served by the edge worker, not by the SPA shell pattern. Response contains, in order: inline critical CSS with the sky gradient for the current local hour (best-effort from CDN geo, corrected by JS within a frame — a wrong-by-one-band sky for 30ms is invisible; a white flash is not), the bootstrap snapshot as `<script type="application/json" id="bootstrap">`, and the critical module preloaded.

**Why the snapshot is inlined:** it removes one full RTT from the first-bird path. On a 4G connection with ~120ms RTT, that single fetch is a quarter of the entire 500ms budget (§11.1). The edge worker fetches the snapshot from the API with a 10s stale-while-revalidate cache; a snapshot up to 10s stale is *fine* because the client extrapolates forward from `server_time` anyway — the birds are simply where they'd be now, which is what we want regardless.

If the snapshot fetch fails or exceeds 150ms at the edge, the worker ships the document without it, and the client renders the **quiet field** — soft sky, one or two faint motion cues — while it fetches. Never a spinner, never a progress indicator, never a skeleton (INV-1).

### 6.3 `GET /v1/aviary/snapshot`

`ETag: "<aviary_id>:<tick_seq>"`, honors `If-None-Match` → 304. `Cache-Control: private, max-age=0, must-revalidate`.

```json
{
  "server_time": 1785500460.412,
  "snapshot_seq": 20114,
  "forward_until": 1785500640.0,
  "next_pull_after_ms": 45000,
  "aviary": {
    "settled": false,
    "weather": { "kind": "rain", "started_at": 1785500390.0, "ends_at": 1785500505.0, "intensity": 0.35 },
    "ornament_seed": 8823741
  },
  "birds": [
    {
      "bird_id": "018f…c2",
      "name": "pip",
      "species": "warbler",
      "scene_order": 0,
      "perch": { "zone": "front", "slot": 1 },
      "perch_move": null,
      "motion": { "activity": "preen", "phase": 0.42, "rate": 0.94 },
      "render": { "plumage_tier": 5, "posture": "upright", "fluffed": false },
      "calls": [
        { "call_id": "018f…9a", "at": 1785500468.2, "motif": "w-rise-3",
          "seed": 918273645, "gain": 0.9, "chorus_group": null }
      ],
      "greeting": { "role": "first", "at": 1785500461.6, "form": "step_forward",
                    "stagger_ms": 0, "absence_band": "hours" }
    }
  ],
  "offer_cooldowns": { "018f…c2": 1785500640.0 },
  "listen_in_available": true
}
```

**What is deliberately absent (INV-6):** `boldness`, `social_warmth`, `vocal_frequency`, `curiosity`, and `plumage_saturation` do not appear. Four of the five never leave the server at all — they are consumed by the tick to produce `perch`, `mood`-derived `motion`, `calls`, and `greeting`, and the client receives only those consequences.

Plumage must reach the renderer somehow, so it crosses as `render.plumage_tier`, an integer 0–7 that indexes a palette/detail ramp. That is the minimum possible leak: it is quantized to eight steps (so it is not a readable trait value), it is a rendering instruction rather than a labeled trait, and it is not accompanied by anything to compare it against. Note also that **`mood` itself is absent** from the wire: the client receives `motion.activity`, `render.posture`, and `render.fluffed`, which are mood's *visible consequences*. A `mood` field would be one `console.log` away from a community-built status dashboard, and the PRD is explicit that the user reads mood from the motion or the product has failed.

A JSON-schema test (§12.5) asserts the snapshot response schema contains none of the five trait names and no `mood` field, so this cannot regress by accident.

### 6.4 `POST /v1/aviary/events`

```json
{ "events": [
  { "event_id": "018f…31", "kind": "presence_ping", "client_ts": 1785500445.0,
    "payload": { "visible": true, "focused": true, "last_input_ms_ago": 12400 } },
  { "event_id": "018f…32", "kind": "listen_in_start", "bird_id": "018f…c2", "client_ts": 1785500401.0 },
  { "event_id": "018f…33", "kind": "offer", "bird_id": null, "client_ts": 1785500410.0,
    "payload": { "offer_kind": "seed" } }
] }
```

`202 Accepted` with `{ "accepted": [...event_ids], "rejected": [{ "event_id": …, "code": "stale" }] }`. Semantics:

- Idempotent on `event_id` (unique index); retries are free, which is what makes the client outbox safe.
- **No event mutates personality or mood.** Handlers only insert log rows. Two exceptions that are aviary-scoped *presentation* state and are applied synchronously so the user sees an immediate response: `settle`/`settle_undo` write `aviaries.settled_until`, and `offer` writes `bird_state.offer_cooldown_until`. Neither touches personality; `bird_state`'s mood columns remain sim-only via column-level grants.
- **Presence pings older than 10 minutes of `server_ts` are rejected as `stale`.** This is the fix for the suspended-laptop case: without it, a machine that slept for eight hours wakes, flushes 480 queued pings, and the tick credits eight hours of "presence" (INV-8). The client is also required to drop them locally; the server rejection is the belt.
- Payload assertions are advisory. The server does not trust `visible/focused`; credit derives from server-side ping arrival spacing (§7.2).
- Rate limit: 60 events/minute/session, burst 200. A client exceeding it is buggy, and dropping is safe.

### 6.5 Notebook, settings, visits, export

| Method | Path | Notes |
|---|---|---|
| `GET` | `/v1/notebook?before=<cursor>&limit=30` | Reverse-chronological, cursor-paginated, unbounded scrollback. No unread count, no badge field, no view-tracking. |
| `PATCH` | `/v1/birds/:bird_id/name` | `{name}`, 1–24 grapheme clusters, trimmed, no effect on any other field. |
| `GET`/`PATCH` | `/v1/settings` | `audio_on`, `captions_on`, `reduced_motion` (`system`\|`on`\|`off`), `visit_notify` (default `false`). |
| `POST` | `/v1/visits/invites` | `{visitor_email}` → creates token, sends invite email. Max 10 outstanding. |
| `GET` | `/v1/visits/invites` | Outstanding invites with `{visitor_email, created_at, expires_at, used: bool}`. |
| `DELETE` | `/v1/visits/invites/:id` | Sets `revoked_at`. Effective immediately; no confirmation surface (the absence from the log is the confirmation). |
| `GET` | `/v1/visits/log` | `{visitor_email, started_at, approx_duration_minutes}` most-recent-first. |
| `POST` | `/v1/account/export` | Enqueues export; 202. Link emailed. |
| `POST` | `/v1/account/delete` / `/v1/account/undelete` | Soft delete / recover. |
| `GET` | `/visit/:token` | Visitor HTML document, same inlined-bootstrap shape as `GET /`, no session cookie required; sets a scoped visit cookie. |
| `GET` | `/v1/visit/snapshot` | Read-only snapshot for a visit session. **410 Gone** with code `visit_unavailable` when the invite is revoked, expired, or the account is deleted. |

**The visitor path is a different route, not a flag.** `GET /v1/visit/snapshot` is served by handlers that (a) have no access to the event-ingest module at all, (b) strip `greeting` (a greeting is *for* the host — a visitor triggering greetings would be interaction), (c) strip `offer_cooldowns` and `listen_in_available`, and (d) touch only `visit_sessions.last_pull_at`. There is no code path from a visitor request to `interaction_events`, which is how "visitor attention does not drift the host's birds" is guaranteed structurally rather than by a conditional someone can later invert. The visitor bundle is a separate entry point with the offer, settle, listen-in, notebook, and settings modules absent, so the read-only guarantee holds even against a tampered client.

### 6.6 Endpoints that will never exist

Written down because the discipline is the feature (INV-2, INV-3). No `/v1/stats`, `/v1/streak`, `/v1/activity`, `/v1/presence`, `/v1/insights`, `/v1/birds/:id/traits`, `/v1/leaderboard`, `/v1/discover`, `/v1/aviaries` (plural), `/v1/notifications`. No field named `days_visited`, `session_count`, `visit_count`, `streak`, `level`, `score`, `xp`, `achievement`, `badge`, `last_seen_days_ago`, `total_presence`. A contract test asserts the OpenAPI document contains no path or property name matching that deny-list, so adding one fails CI rather than shipping.

---

## 7. Simulation engine design

### 7.1 Tick scheduling

`sim-worker` runs N replicas. Each loop iteration:

```sql
UPDATE aviaries SET tick_lease_until = now() + interval '30 seconds'
WHERE aviary_id IN (
  SELECT aviary_id FROM aviaries
  WHERE next_tick_at <= now()
    AND (tick_lease_until IS NULL OR tick_lease_until < now())
    AND deleted_at IS NULL
  ORDER BY next_tick_at
  LIMIT 500
  FOR UPDATE SKIP LOCKED
) RETURNING *;
```

Batch of 500 aviaries: one read of their birds/state/personality, one read of new events, pure in-memory computation, one batched write, one lease release. `SKIP LOCKED` plus the lease means an aviary is never ticked concurrently — the precondition for INV-5 holding under horizontal scaling.

**Cadence.** 60s while *active* (any presence event in the last 24h). After 24h with no presence, the aviary **hibernates to a 300s cadence**. Justification and its limit:

- Drift during absence is exactly zero by design (INV-4: `u = 0` → no change), so the only thing a tick does for an unwatched aviary is advance mood, perches, weather, and call schedule. At 300s the mood machine still moves through the day, birds still change perch, weather still passes. The aviary continues; it continues more cheaply.
- **Hibernation must be unobservable.** On a snapshot request where `last_tick_at` is older than the *active* cadence, the API enqueues an immediate tick and waits up to 150ms for it. If the tick lands, the fresh snapshot is served. If it doesn't, the last state is served and the client extrapolates (which it does anyway). Either way, the returning user sees an aviary that has caught up to now, not one frozen five minutes ago.
- A returning user's first presence ping restores the 60s cadence within one tick.

**Determinism.** The tick is `f(state, events, now_bucket, prng) → state'` with `prng = xoshiro256(hash(aviary_id ^ rng_stream, tick_seq))`. No `Math.random()` and no direct clock reads inside the pure core (`now` is an injected parameter). This makes every tick replayable for tests, for calibration (§7.4), and for incident forensics. A lint rule bans `Date.now`, `new Date()`, and `Math.random` inside `packages/engine/`.

**Idempotency.** A tick that crashes after computing but before committing leaves `event_watermark` unadvanced; the retry recomputes identically from the same `tick_seq` and produces the same result. Events are consumed exactly once because the watermark advance and the state write are in one transaction.

### 7.2 Presence accounting

The PRD's three-part conjunction (visible ∧ focused ∧ recent input) is evaluated **on the client**, because only the client can observe it. That means the server must not trust it. The design makes honest reporting the only thing that produces credit:

**Client side.** A presence monitor tracks `document.visibilityState`, `window` focus/blur, and the timestamp of the last `pointermove`/`keydown`/`pointerdown`/`touchstart`. It emits a `presence_ping` every **15s** only while all three hold, with an **activity window of 4 minutes** for the input condition. Four minutes leans deliberately long, per the PRD: sitting and watching without moving the mouse *is* the product, so presence should survive a few still minutes and end only when there has been no sign of a person for a while. Any condition breaking stops the pings immediately — no drain, no trailing ping.

**Server side.** The tick folds pings per aviary:

1. Collect pings by `session_id`, ordered by `server_ts`.
2. For consecutive pings from a session, credit `min(gap, 20s)` — a gap larger than 20s means pings were missing, which means presence was not continuous, so it must not be credited. The 20s ceiling against a 15s cadence gives one ping of jitter tolerance and nothing more.
3. Credit the first ping in a run 15s (opening a window costs one interval).
4. **Merge intervals across sessions before crediting** (§9.3). A user with laptop and phone both open and both watching gets one aviary-second per wall-clock second, not two. Presence is the user's attention to *the aviary*, not the sum of their devices; without the union, a two-device user drifts their birds twice as fast, which is a silent calibration break of exactly the kind the PRD warns about.
5. Clamp daily credit at 4 hours. Above that, either something is wrong or the signal has stopped meaning what the drift function assumes.
6. Write to `presence_credit` and pass the interval's seconds into drift.

Combined with the 10-minute staleness rejection in §6.4, the failure modes are closed: a background tab produces no pings; a suspended machine's replay is rejected; a lax client that pings unconditionally still gets no more than wall-clock credit and gets caught by §12.3's fixture tests; and a multi-device user is credited once.

Settle and tab-close are both simply the absence of further pings. Neither is penalized, neither is required, and the engine does not distinguish them (the PRD is explicit). `settle` exists as an event only for the lighting state and for a small mood-quieting nudge.

### 7.3 Drift function

Per tick interval, for each bird, for each trait:

```
u_trait = clamp01( Σ_signals  w[signal][trait] · normalized_intensity[signal] )

x' = x + α · u_trait · (ceiling_trait − x)
```

Properties, each of which is load-bearing:

- **Monotonic** (INV-4). `α > 0`, `u ≥ 0`, `x ≤ ceiling` ⇒ `x' ≥ x`. Neglect means `u = 0`, which means `x' = x` exactly — not slow decay, not "approach a lower baseline." A bird ignored for a month has the identical vector it had a month ago. What changes on neglect is *behavior*, and it changes for a different reason: mood follows time-of-day and weather with no interaction nudges, so the bird sits further back and greets less. Quieter, not diminished. This is the whole "no Tamagotchi" claim implemented in one line.
- **Saturating.** The `(ceiling − x)` term makes early drift fast and late drift slow, which matches the felt shape of a relationship deepening and prevents every long-tenured bird from pinning at 1.0.
- **Low-pass, not integrating.** `intensity` is normalized against the interval, and `filter_state` carries a per-trait exponential moving average of recent signal so a single burst of activity moves the EMA a little rather than moving the trait a lot. A user cannot click their way to a bolder bird; they can only *be there* repeatedly.
- **Daily saturation cap.** Total per-trait delta per UTC day is capped at 3× the expected delta from a 20-minute presence session. Prevents an eight-hour session from being eight sessions and prevents offer-spam from concentrating curiosity drift (the PRD's stated reason for the offer cooldown, enforced a second time in the engine).

**Signal weights** (`w`, initial values, server config, hot-reloadable):

| Signal | Intensity definition | boldness | social warmth | vocal freq | plumage | curiosity |
|---|---|---|---|---|---|---|
| Presence-time | credited seconds / 1200s reference, capped 1.5 | 0.10 | 0.14 | 0.12 | 0.20 | 0.09 |
| Listen-in on this bird | credited focused seconds / 300s, capped 1.0 | 0.03 | 0.11 | 0.10 | 0.02 | 0.02 |
| Listen-in on another bird | 0.25 × that bird's intensity | 0 | 0.02 | 0.01 | 0 | 0 |
| Offer made (bird nearby) | 1 per offer, ≤ 3/day | 0.05 | 0.01 | 0 | 0 | 0.02 |
| Offer accepted (approached) | 1 per acceptance | 0.02 | 0.02 | 0 | 0 | 0.07 |
| Settle | — | 0 | 0 | 0 | 0 | 0 |

Presence dominates (≈65% of typical total drive), listen-in is second (≈22%), offers third (≈13%), settle contributes nothing — exactly the PRD's stated ordering. Settle's only effect is a mood nudge toward drowsy/settled and a clean close of the presence window.

`α` starts at **0.0022 per tick-hour of credited presence**, with the fitting procedure below.

### 7.4 Drift calibration

The PRD names the target so it can be tested; we make it a test.

**Definitions.** *Regular visits* = 5 sessions/week × 12 minutes credited presence = 60 min/week. *Measurable* = a trait delta the harness detects above per-tick numerical noise, ≥ 0.010 on at least three of five traits. *Visible* = at least one behavioral quantization boundary crossed: `plumage_tier` +1, front-perch probability +12 percentage points, unobserved call rate +20%, or greets-first probability +15pp.

**The calibration harness** (`packages/engine/calibrate`) replays synthetic user profiles through the real tick function against an injected clock, twelve simulated weeks in a few seconds:

| Profile | Behavior | Assertion at 1 week | Assertion at 3 weeks |
|---|---|---|---|
| `regular` | 5×/wk, 12 min, occasional listen-in and offer | measurable, not visible | ≥ 1 boundary crossed |
| `daily-long` | 7×/wk, 40 min | measurable | ≥ 2 boundaries, no trait at ceiling |
| `weekend-only` | 2×/wk, 25 min | measurable | ≥ 1 boundary |
| `single-session` | 1 session, 20 min, then gone | delta < 0.004 | **identical to week 1** |
| `lapsed-returner` | 3 weeks regular, 3 weeks absent, 3 weeks regular | — | week-6 vector **exactly equals** week-3 vector |
| `farmer` | 200 offers + listen-in cycling, 4h/day, one week | ≤ 2.2× the `regular` delta | — |
| `two-device` | `regular`, both devices open simultaneously | within 5% of `regular` | within 5% of `regular` |

The `lapsed-returner` and `single-session` rows are INV-4 as an executable statement. The `farmer` row is the anti-Tamagotchi guard from the other direction: effort must not be convertible into character. The `two-device` row is §9.3's presence union.

**Fitting.** Solve `α` so `regular` lands mid-band: measurable at 7 days, first boundary crossed between days 17 and 24. Then **ship at 0.6 × α_fit**, deliberately at the slow edge.

The reason is the sharpest operational consequence of monotonic drift, and it deserves to be stated plainly: **too-fast drift is unrecoverable and too-slow drift is a config change.** Because traits never decrease, an over-tuned `α` in production permanently over-advances every bird in the population, and the only "fixes" available are to rewrite vectors (destroying drift history, which is INV-7's failure) or to leave them. Too-slow drift, by contrast, is corrected by raising `α`; users' birds simply continue from where they are. The asymmetry in the cost of being wrong is the entire argument for launching conservative and raising after six weeks of dogfood signal. Every `α` change is a reviewed config release with a changelog entry, never a runtime experiment.

**Real-time validation.** The harness can only validate the math. That three weeks of drift is *felt* is a human question, which is why dogfood runs a minimum of four real weeks (§13.2) with a structured week-3 interview: "has anything about your birds changed?" asked open-ended before any prompted question, because a prompted yes is worthless here.

### 7.5 Mood

States: `wary`, `content`, `curious`, `drowsy`, `alert`, `settled`. `settled` is reachable only from night lighting or a settle gesture and exits on dawn or re-engagement.

Each tick, for each bird, score every candidate state:

```
score(m) = A[m][time_band]                     // time-of-day affinity: alert at dawn, drowsy at dusk
         + Σ nudge[recent_event][m] · decay(age)  // offer accepted → content; alarm heard → wary
         + weather[kind][m]                       // rain → drowsy/content up, alert down
         + contagion[m]                           // neighbours' wary/alert bleeds within one perch zone
         + bias(personality, m)                   // high boldness suppresses wary; high curiosity lifts curious
         + hysteresis(current == m)               // a constant favouring the status quo
```

Then sample `softmax(score / T)` with `T` low enough to average **2–5 mood changes per bird per day**, subject to `mood_min_until` (6–20 minutes depending on the state; `drowsy` and `settled` are stickier).

**Daily-ish reset without snapping.** At the local dawn band, `T` rises for ~40 minutes and time-affinity weights shift, so the bird is *likely* to re-roll toward a morning-appropriate mood but does so by sampling, in a transition the client can animate. There is no assignment to a neutral default anywhere in the code, at dawn or on session start. The PRD's "the user should never notice mood snapping to a default on tab open" holds trivially: **session start is not an input to the mood function at all.** Opening the tab does not appear in the score.

**Mood persistence** is therefore automatic — mood is a stored column advanced by a tick that has no notion of clients. A bird that ended yesterday drowsy at dusk has been ticking through night and dawn and is somewhere reasonable by morning, not resumed-from-frozen.

**Contagion** is what makes the aviary a small social system: one bird entering `wary` or `alert` adds a decaying term to same-and-adjacent-zone birds for ~10 minutes, weighted by their social warmth. A bird with high warmth catches its neighbour's alarm; a bold, low-warmth bird mostly doesn't.

### 7.6 Perch assignment

Zones `front`/`middle`/`back` with slot capacities scaled to viewport-independent normalized positions (front 2, middle 3, back 3 — eight slots for a maximum of seven birds, so there is always somewhere to go).

```
desire(bird, zone) = zone_bias[species][zone]
                   + boldness_pull[zone] · boldness        // front for bold
                   + mood_pull[mood][zone]                 // wary → back, curious → forward
                   + listen_in_pull[zone] · is_listened_to // gentle forward lean when attended to
                   + offer_pull[zone] · offer_active · curiosity
                   + stay_bonus(current zone)              // strong: birds don't fidget between zones
```

Resolved as a **stable matching** against slot capacity, seeded by the tick PRNG, so an unchanged desire ordering produces an unchanged assignment. Without the stay bonus and the stable resolution, a near-tie between two birds makes them swap perches every tick — 60-second oscillation that reads as twitchy and mechanical, precisely the "robotic" failure INV-1 forbids. Expected rate: 1–4 zone changes per bird per hour while attended, fewer at night.

A perch change emits `perch_from` + `perch_move_at` + `perch_move_ms` (700–1400ms) so the client draws a flight arc. The user has no affordance to move a bird — there is no request that accepts a perch, so "perch is a signal, not a layout" is enforced by the absence of an endpoint.

### 7.7 Call scheduling

Each tick extends a **forward window** to `now + 180s` (three tick-intervals of margin, so one lost tick never produces silence). For each bird:

```
rate = base_rate[species]
     · (0.45 + 1.10 · vocal_frequency)
     · mood_mult[mood]                   // alert 1.3, curious 1.2, content 1.0, wary 0.7, drowsy 0.35, settled 0.05
     · time_mult[local_band]             // dawn 1.4, midday 1.0, evening 0.7, night 0.05 (nightjar species: 0.9)
     · weather_mult[weather]             // rain 0.6 across the aviary, per the PRD
     · (unobserved ? 1.15 : 1.0)         // "calls more when unobserved" is a real term, not a metaphor
```

Sample inter-call intervals from a gamma distribution (shape ≈ 2, so calls cluster naturally rather than arriving like a metronome — a Poisson process sounds mechanical because it produces implausibly even spacing at these rates). For each scheduled call, draw a **motif** from the mood-weighted grammar and a 32-bit **variation seed**. The client's synthesis is a pure function of `(timbre_seed, motif, variation_seed, mood)`, so a repeated snapshot pull never re-voices a call differently and no call is ever identical to another (the seed space is 2³², and the anti-repetition rule in §8.3 additionally forbids reuse within a session).

**Call-and-answer.** When bird A has a scheduled call, each other bird B gets an answer probability `p = 0.15 + 0.45 · social_warmth_B` (× mood multiplier), scheduled 0.4–1.8s after A. Answers are marked `answer_to: call_id` so the client can voice them with a slight pitch lean toward A's call — the acoustic tell that they are answering rather than coinciding.

**Chorus is emergent, not an event.** If two or more scheduled calls overlap within 1.5s, the tick tags them with a shared `chorus_group`. The client then applies mild mutual timing attraction (≤ 120ms) and slight pitch spreading within the group. There is no "chorus event" state, no trigger, no cooldown; choruses happen because two high-vocal-frequency birds in compatible moods happened to be scheduled close together, which is exactly what the PRD describes.

### 7.8 Return-greeting

The PRD gives this the most prose in `interactions.md`, and it is the single most compressible feature in the product — "play an arrival animation" is right there. It gets its own explicit pipeline.

**Trigger.** The API detects a presence resumption: a `presence_ping` arriving for an aviary whose last ping is older than 90 seconds. It records `absence_ms` and enqueues an immediate tick with a `greeting_request`. The greeting is computed by the **tick**, not the client, because it needs boldness, mood, and warmth — none of which cross the wire (INV-6).

**Absence bands** (the PRD requires the signal to be wired through, so it is a first-class parameter, not a boolean):

| Band | Range | Greeting character |
|---|---|---|
| `moment` | 90s – 5 min | at most a glance up; often nothing at all |
| `minutes` | 5 – 45 min | a glance, sometimes a single quiet note |
| `hours` | 45 min – 10 h | a two-note call, or a head-tilt and a step toward front |
| `day` | 10 h – 3 d | a longer call, a step forward, second bird often answers |
| `long` | > 3 d | re-orientation: forward move, longer call, higher chance of a second greeter |

**Greeter selection.** `p_greet(bird) = base[absence_band] · (0.35 + 0.9 · boldness) · warmth_mult(social_warmth) · mood_mult(mood)`. Sample **one** primary greeter weighted by that score. Each other bird independently gets a much smaller secondary chance (scaled by warmth and by the primary's call). If no bird passes threshold — a plausible outcome for a short absence with two wary birds — **no greeting is emitted**, and that is correct: the PRD says the warier bird greets "later or not at all on a given day," and a guaranteed greeting is a canned cue with extra steps.

**Form selection.** From `{glance, single_note, two_note, head_tilt_step, call_and_answer, forward_move_long_call}`, weighted by band, boldness, and mood, with a **recency penalty**: the same `(bird, form)` pair is down-weighted 0.25× if used in the last three greetings. Stored in `bird_state` as a small ring buffer. Within a form, the call's variation seed and the motion's timing/amplitude are freshly drawn, so `two_note` is a *class* of greeting rather than a clip. This is the difference between "procedurally varied" and "three pre-recorded variants in rotation," and it is the thing the PRD says the user detects on the first session.

**Staggering.** When two birds greet, the secondary's `stagger_ms` is drawn from `U(400, 1600)`. Simultaneity is explicitly forbidden — a unison greeting *announces* the arrival (INV-2), and the whole affective point is the aviary noticing one bird at a time.

**What is not built:** no toast, no banner, no modal, no "welcome back," no "you've been gone 3 days" copy, no absence-length text anywhere on any surface. `absence_band` exists only inside the engine and is not present in the snapshot. The greeting is the entire welcome.

### 7.9 Tick cost model

Per aviary-tick: ~7 bird records, five trait updates, six mood scores, one matching, ~10 call schedules. Well under 1ms of CPU; cost is dominated by database IO, which is why batching is 500 aviaries per pass.

At 100k accounts with ~15% active (85% hibernating): `15k/60 + 85k/300 ≈ 250 + 283 ≈ 533 ticks/s`, or ~1.1 batches/s. Two `sim-worker` replicas with headroom; scale by replica count, and the `SKIP LOCKED` lease makes that linear with no coordination. Postgres write volume is the real constraint (~533 aviaries/s × ~7 bird rows), addressed by batched multi-row `UPDATE ... FROM (VALUES ...)` and by only writing rows whose state actually changed — typically 10–20% per tick, since mood and perch are sticky by design.

---

## 8. Audio pipeline

### 8.1 Call grammar

A **motif** is a sequence of syllable specifications, authored by the sound designer in a small declarative DSL, not written by engineers:

```ts
{ id: "w-rise-3", species: "warbler", moods: ["content","curious","alert"],
  syllables: [
    { shape: "rise", f0: [0.42, 0.78], dur_ms: [90, 130], vibrato: { hz: 0, depth: 0 },
      noise: 0.06, env: "soft-attack" },
    { shape: "rise", f0: [0.50, 0.86], dur_ms: [85, 120], noise: 0.05, env: "soft-attack" },
    { shape: "fall", f0: [0.80, 0.55], dur_ms: [120, 190], noise: 0.09, env: "round" }
  ],
  gaps_ms: [[35, 70], [40, 90]],
  repeat: { min: 1, max: 2, p: 0.3 } }
```

`f0` values are **normalized 0–1 within the bird's own pitch band**, so the same motif on two birds of the same species reads as the same phrase in two individual voices. Ranges (not values) are sampled per instance from the variation seed.

Grammar: `call → phrase (gap phrase){0..2}`, `phrase → syllable{1..4}`. Motif selection is mood-weighted; syllable count, gap lengths, pitch offsets, tempo, and vibrato depth are seed-sampled inside authored ranges. 4–7 motifs per species × sampling ranges × 2³² seeds means a user will not hear the same call twice, ever, which is the whole point (INV-1).

### 8.2 Synthesis graph

Per voice, pre-allocated:

```
  [oscillator ×2 (sine + shaped saw, detuned)]  ┐
                                                 ├─▶ [gain: syllable envelope] ─▶
  [noise buffer source, looped, shared]  ──────  ┘
      ─▶ [formant bank: 3× biquad bandpass, per-bird Q and centres]
      ─▶ [pitch/vibrato via oscillator.frequency automation]
      ─▶ [voice gain] ─▶ [bird bus: gain + lowpass (listen-in)] ─▶ [stereo panner: perch x]
      ─▶ [aviary bus] ─▶ [FDN reverb, 4 delays, ~0.9s tail] ─▶ [compressor, gentle] ─▶ destination
```

- **Voice pool of 8**, pre-allocated at context creation, allocated round-robin with oldest-release stealing. Zero `AudioNode` construction after warm-up. This is how §11.1's no-memory-growth budget is met: WebAudio node churn is the largest allocation source in a 30-minute session, and pooling eliminates it. One shared looped noise `AudioBufferSourceNode` (a node cannot be restarted, so the pool holds a small ring of them created at warm-up and recycled by gating gain, never by `start()`/`stop()` churn).
- **Bird buses are persistent**, one per bird (max 7), created on adoption-in-session.
- **Reverb is algorithmic (feedback delay network), not convolution.** A convolution impulse response is a downloaded audio asset, which the "no recorded audio" rule forbids in spirit and the 2MB budget forbids in fact. An FDN is ~40 lines and a handful of nodes, and it is what makes two calls sound like they are in *one place* rather than in two mixers — the actual affective purpose of the reverb.
- **Scheduling** uses `AudioContext.currentTime` with a 250ms lookahead pump on a 100ms interval. Call intents from the snapshot are converted from server time to context time via a smoothed offset estimate (§8.6). Drift of tens of milliseconds is inaudible; a hard resync would be audible, so the offset is low-pass filtered.

### 8.3 Ambient bed

Beneath the calls: a very low-level wind/foliage bed synthesized from filtered noise with slowly-modulated bandpass centres, plus occasional distant non-aviary bird calls at −24 dB (the sense of a wider world). Procedural, no assets. Intensity follows weather and time of day. This is what keeps "others quiet to ambient" during listen-in from meaning "near-silence."

Anti-repetition: a session-scoped ring buffer of the last 12 `(bird, motif, quantized-pitch-offset)` triples; a redraw that collides is re-rolled once. Cheap insurance against the seeded sampler producing an audible near-repeat.

### 8.4 Per-bird recognizability

The mechanism that makes seven a workable cap and "you know Pip by ear" true. From `timbre_seed`, fixed at adoption and **never modified by drift, mood, or anything else**:

- Base pitch band (a ~1.5-semitone window inside the species range)
- Formant bank centre frequencies and Q values (the bird's "throat")
- Harmonic profile (oscillator mix and detune)
- Noise ratio (breathiness)
- Two preferred syllable-shape bigrams (its habitual phrasing, e.g. tends to follow a rise with a trill)
- A characteristic tempo multiplier

What mood and drift are allowed to modulate: **rate, phrase length, pitch spread, motif selection weights, and amplitude.** Nothing else. This is the invariant/variant split that lets a bird be obviously in a different mood while being obviously the same bird — which is precisely what the PRD asks for and what a naive "mood changes the voice" implementation destroys.

Enforcement: a unit test asserts the synthesis parameter set is partitioned into `identity` (function of `timbre_seed` only) and `expression` (function of mood/drift/seed), and that no `identity` parameter has a mood or trait in its dependency graph.

### 8.5 Listen-in mix

On engage (target bird B):

| Bus | Target | Ramp |
|---|---|---|
| B's bird bus gain | 0 dB (+2 dB lift) | `setTargetAtTime`, τ = 0.55s |
| Other bird buses | −12 dB | τ = 0.7s |
| Other bird buses lowpass | 2.2 kHz (from 20 kHz) | τ = 0.7s |
| Ambient bed | −4 dB | τ = 0.9s |
| B's reverb send | −3 dB (drier = closer) | τ = 0.7s |

Perceptual settle ≈ 1.8–2.5s. Disengage uses identical constants in reverse. Rules with teeth: **no bus is ever set to zero or disconnected** (the PRD: "other birds drop in mix but never go silent"), and **no gain is ever assigned directly** — a lint rule bans `gain.value =` outside the initialization module, forcing every change through a ramp. A hard cut is what turns an aviary into a mixing desk with soloable tracks, and it is one careless assignment away at all times.

The lowpass is doing quiet work: level reduction alone reads as "turned down," while level plus high-frequency roll-off reads as "further away," which is what leaning in toward one bird actually sounds like.

Disengage triggers: clicking B again, clicking another bird (cross-fade, single transition, not out-then-in), clicking empty scene, `Escape`, or keyboard focus leaving the aviary.

### 8.6 Clock alignment and the audio-unlock problem

**Clock.** Each snapshot yields `offset = server_time − performance.now()/1000`, fed through an EMA (α = 0.15) with outlier rejection. Call intents are scheduled at `context.currentTime + (call.at − now_est)`. Intents whose time has already passed by < 250ms are voiced immediately at reduced attack (they are mid-call); intents older than that are dropped, since a call from twenty seconds ago is not a call.

**Autoplay policy is a real conflict with the PRD and needs a named resolution.** Every target browser blocks or suspends `AudioContext` before a user gesture, so "calls already audible on the first frame" is not always physically achievable on a cold navigation. The resolution:

1. Construct the `AudioContext` immediately and call `resume()`. On a return visit within an established origin engagement (common in Chrome/Edge), this succeeds and calls are audible in the first second, as designed.
2. If it remains `suspended`, the aviary renders and animates normally — visually complete, silent — and the top-bar audio icon renders in an inactive state. **No modal, no banner, no "click to enable sound" copy, no overlay.** The icon is the affordance (INV-2).
3. The first pointerdown/keydown anywhere resumes the context, and the ambient bed plus already-scheduled calls fade in over ~1.2s. Never a burst of queued calls.

The alternative — a "tap to begin" gate — would install exactly the entry state the PRD forbids, and it would do so on the first frame of the first session, which is the worst possible place to break the central conceit. Silence-until-touched with a passive icon is the lesser cost, and it is the same graceful-silence path the WebAudio fallback already requires (§8.8), so it is a code path we test regardless.

### 8.7 Recognizability listening study

The seven-bird cap is asserted as empirical in the PRD, so we verify rather than assume. Pre-launch, n ≥ 12 listeners on headphones: 15 minutes of familiarization with 2 named birds, then a blind identify-the-caller task at aviary sizes 2, 3, 5, and 7, 20 trials each, chorus conditions included.

Gates: ≥ 80% at 5 birds, ≥ 70% at 7 birds. If 7 fails, **ship the age-gated offer schedule capped at 5** (a config value, §13.3) and treat raising the cap as post-v1 audio-mix work. If 5 fails, the motif libraries are not distinct enough and the fix is authoring, not engineering. Also run an automated proxy in CI: MFCC-based spectral distance between calls, asserting within-bird variation exceeds a floor (not canned) while between-bird distance exceeds a larger floor (identifiable) — a cheap regression guard for when a motif library is edited.

### 8.8 WebAudio fallback

If `AudioContext` is unavailable or construction throws: the aviary plays in silence with **captions on by default**, the audio icon renders unavailable, and one matter-of-fact line appears in accessibility settings ("Audio isn't available in this browser. Call captions are on."). We ship no recorded-audio path at any quality — the rule is unconditional, and the code has no decoder, no asset pipeline, and no `<audio>` element to make it easy to violate. Aggregate telemetry counts audio-context failures by browser so we know the size of the affected population without knowing who is in it.

---

## 9. Sync model

### 9.1 What makes sync trivial

There is one canonical record, one writer, and no client-side state to reconcile. Multi-device sync is not a feature with a protocol; it is the absence of divergence. Both devices `GET /v1/aviary/snapshot`, both receive the same bytes (same ETag), both interpolate forward from the same `server_time`. There is no client-to-client channel, no CRDT, no merge, no vector clock, no offline mutation queue for state.

### 9.2 Pull schedule

The client pulls a fresh snapshot on:

1. **Bootstrap** — inlined in the HTML, zero fetch.
2. **`visibilitychange` → visible** — the tab was hidden and the world moved on.
3. **Long frame gap** — a rAF delta > 5s means the machine was suspended or throttled; pull before rendering another frame, so a bird never appears to teleport after a sleep.
4. **Keepalive every 45s while visible** — with jitter ±7s to avoid thundering herds against the tick boundary.
5. **Forward-window exhaustion** — if `forward_until − now < 45s`, pull early; we must never run out of scheduled calls.
6. **After an event that changes visible aviary state** (offer, settle) — a targeted pull ~400ms later to pick up the server's response.

304s are the common case and cost almost nothing. Backoff on failure: 1s, 2s, 4s … capped at 60s with jitter, and **the aviary keeps rendering the whole time** (§9.5).

### 9.3 Multi-device correctness

Two devices signed in simultaneously is the case that quietly breaks a naive implementation, in three places:

1. **Presence double-credit.** Both devices ping. Naive summing doubles the dominant drift input. Fixed by interval union per aviary before crediting (§7.2 step 4), verified by the `two-device` calibration profile.
2. **Listen-in.** Listen-in is treated as **device-local UI state that emits an aviary-scoped event**. It is not mirrored: a phone listening in on Pip does not change what the laptop renders. Rationale — listen-in is an act of attention by a person at a device, not a property of the aviary; mirroring it would make one device yank another's audio mix, which is worse in every scenario including the single-user-two-devices one. Server-side, overlapping listen-in intervals on the *same* bird are unioned (no double credit); on *different* birds, both credit, each duration-capped at 300s per session per bird.
3. **Settle.** Settle *is* aviary state (it changes lighting), so it does propagate: the second device sees the settled aviary within one pull. Settle and `settle_undo` are resolved by `ingest_seq` order — last write wins — which is safe precisely because it is presentation state with no history, not personality. This exception is documented next to INV-5 so nobody generalizes it.

### 9.4 Why last-write-wins is unreachable for personality

Structurally, not by convention:

- No API endpoint accepts a trait value. There is no request shape in the OpenAPI document containing `boldness` or any sibling; a client cannot express the write.
- `api_rw` has `SELECT` only on `bird_personality`, and column-level grants restrict it on `bird_state`'s mood columns. Even a compromised or buggy handler cannot write a vector.
- Every write is `x' = x + Δ` computed from the current row inside the tick's transaction, with `version` incremented and asserted sequential by trigger.
- Ordering authority is `ingest_seq`, server-assigned, so a stale client cannot influence order.
- Personality is never recomputed from history, so a partial or trimmed event log cannot regress a vector.

The PRD's specific nightmare — a lunch session on the phone, holding an older read, overwriting the morning's laptop drift, with no log line saying data was lost — requires a client-submitted absolute value. There is nowhere to submit one.

### 9.5 Degradation

| Condition | Behavior |
|---|---|
| Snapshot fetch fails, forward window intact | Keep rendering and voicing scheduled calls; retry with backoff. Nothing visible. |
| Forward window exhausted (> 3 min stale) | Continue micro-motion; synthesize calls locally at the last-known per-bird rate with fresh seeds; no perch changes. The aviary is quieter and stiller, never frozen. |
| Offline > 10 min | Same, plus drop queued presence pings older than 10 min (they were not honest presence and must not be credited late). |
| New snapshot after divergence | **Retarget, never teleport.** Position deltas ease over ≤ 220ms; a mood-driven activity change swaps at the next activity boundary rather than mid-motion; in-flight calls finish. |
| Session revoked / expired | Matter-of-fact surface: "Your session timed out. Sign in again to keep watching." |
| Server 5xx sustained | Matter-of-fact reload surface after 3 failed attempts *only if* the forward window is also exhausted — never interrupt a rendering aviary with an error for a transient failure it survived. |

The through-line: **the client's failure mode is "continues, quieter," which is the same shape as the product's neglect behavior.** Freezing or erroring is what a machine does; the aviary is not supposed to look like a machine even when it is broken.

---

## 10. Frontend rendering pipeline and accessibility

### 10.1 Scene coordinates and layers

Normalized scene space: `x ∈ [0,1]`, `y ∈ [0,1]`, mapped to the canvas each resize. Perch zones sit at fixed normalized `y` (back 0.34, middle 0.52, front 0.72) with slot `x` positions spread by a viewport-width-dependent spacing function — **wider viewports spread perches apart rather than scaling birds up**, per the PRD's "more space between perches" on desktop.

**No-crop guarantee:** every slot `x` is inset from the edges by the maximum bird half-width plus a margin, computed after layout. On narrow viewports, slot spacing compresses and slots gain a small vertical stagger so silhouettes don't collide. Test matrix: 320, 375, 414, 768, 1024, 1440, 1920, 2560 CSS px × 7 birds × all perch permutations, asserting every bird's bounding box lies fully inside the canvas. No panning, no scrolling, no zoom — the scene has no camera transform at all, so those features cannot creep in via a viewport parameter.

Four canvases:

| Layer | Redraw cadence |
|---|---|
| Sky + light | On light-band change (~1 Hz), plus a weather cross-fade. Cached gradient. |
| Background foliage (parallax) | On resize and on parallax offset change > 0.5px (pointer-driven, heavily damped) |
| Bird plane | Every frame, dirty-rect where feasible |
| Foreground ornaments | Every frame; small, cheap |

Parallax is driven by a damped pointer offset with a maximum excursion of ~10px on the background — enough to feel like depth, nowhere near a layered-illustration showpiece.

### 10.2 Frame loop

Single `requestAnimationFrame` driver. `dt` clamped to 50ms (a suspended tab must not integrate an hour of motion in one frame). When `document.hidden`, **rendering stops entirely and the audio context is suspended after a 2s grace** (battery; the PRD explicitly endorses this — the simulation continues server-side, which is the point). On becoming visible: pull a snapshot, then resume rendering *from the newly computed state*, so nothing rewinds.

Adaptive quality, in strict priority order, driven by a rolling p95 frame time:

1. Reduce ornament density (30 → 12 → 4 concurrent)
2. Halve parallax update rate, then disable parallax
3. Drop sky redraw to 0.25 Hz
4. Reduce bird micro-motion oscillator count from 5 to 3 layers

Bird motion is degraded **last and never disabled**, because bird motion is the product. There is no path where a slow device gets still birds.

### 10.3 Bird actors: procedural motion, not cycles

Each bird is a small actor: `(activity, phase, rate, mood-derived amplitudes)` → pose. The pose is composed from **layered continuous oscillators**, not keyframe cycles:

- Breath: 0.35–0.9 Hz, amplitude by mood (drowsy slow and deep, alert shallow and quick)
- Head scan: quasi-periodic with 1-D value noise, excursion by mood (wary scans wide and often)
- Weight shuffle: Poisson, ~1 per 25–60s, a small lateral hop-and-resettle
- Blink: Poisson, ~1 per 4–9s, suppressed while `settled`
- Preen: a sequence sampler over preen sub-poses with irrational-ratio timing so it never lines up the same way twice
- Tail/wing settle: damped spring responding to the other layers

Because every layer is continuous and driven by `activity_since + phase`, **two consecutive seconds of a bird are never identical, and the first frame after load is mid-motion** (INV-1). Sprite atlas art (baked at build from the designer's SVGs into one compact WebP, plumage applied as a runtime multiply-tint pass so we ship one atlas rather than eight tiers) supplies the drawn parts; the *motion* is entirely procedural. A lint rule bans a `frames[]`-indexed animation type in the bird module — the type system makes cycle animation awkward to express, which is the point.

Micro-motion runs at the same fidelity whether or not the user is interacting; there is no "attract mode" and no idle-timeout state change.

### 10.4 First frame

The critical path, budgeted against §11.1:

1. Edge HTML arrives with inline sky CSS → **the quiet field is painted before any JS runs.** This is the load state, and it is indistinguishable from the aviary's own sky.
2. Critical module (target ≤ 160KB gzip) parses: scene layout, bird actor, **vector-drawn bird silhouettes**, snapshot decode, rAF loop.
3. Birds are drawn from vector paths at their snapshot positions and phases. **First bird visible.**
4. The sprite atlas decodes asynchronously and upgrades detail in place (a cross-fade over ~250ms; the silhouette-to-detail transition is a plumage refinement, not an appearance).
5. Audio module, narration composer, caption layer, and top bar hydrate after first paint. Settings, notebook, and visit routes are separate chunks fetched on demand.

Vector-first birds are the specific trick that makes 500ms achievable without gambling on image decode inside the budget: the atlas is no longer on the critical path at all. The empty-aviary state (post-adoption, pre-first-bird) uses the same quiet field, then a soft fly-in to the starting perch — the only entry animation in the product, justified because a bird arriving *is* the event, and it happens exactly once per bird ever.

**Forbidden in the loading path,** by code review and by a CI check for the corresponding component names: spinner, progress bar, skeleton screen, percentage, "loading" copy, fade-from-white, logo splash, or any transition whose semantics are "the app is starting."

### 10.5 Reduced-motion mode

A **separate render mode**, selected by `prefers-reduced-motion` or the explicit setting, sharing the scene graph and state but substituting a different pose driver:

| Default mode | Reduced-motion mode |
|---|---|
| Continuous oscillator motion | Cross-fade between 3–5 baked poses per activity, 900–1400ms fade, pose changes at 0.5–1.2 Hz keyed to mood |
| Animated flight arc between perches | 1.2s cross-fade between perch positions, no intermediate path |
| Leaf/feather ornaments | Removed |
| Parallax | Removed |
| Day/night and weather colour shifts | Retained, at 1/3 rate |
| 60fps rAF | ~20fps throttled rAF (nothing moves fast; saves battery) |
| Top bar fade | Slower, longer dwell |
| Calls, drift, mood, notebook | **Fully retained, unchanged** |

The mode has its own visual acceptance criteria and its own screenshot suite. It is explicitly *not* "the default renderer with animations disabled" — it is a calmer register with its own designed charm, and the designer signs off on it as a deliverable. Definition-of-done for any visual feature includes its reduced-motion treatment (§12.6), which is the only durable defense against the mode rotting behind the default.

### 10.6 The prose package

`@aviary/prose` is a shared workspace package, used by the client narrator, the client caption generator, and the server notebook writer. Structure:

```
prose/
  naturalist/
    lexicon.ts        // bird descriptors, light words, weather words, posture words, motion verbs
    narration.ts      // idle + priority narration templates with slot constraints
    captions.ts       // call-shape → prose mapping
    notebook.ts       // notebook templates + noteworthiness rules
    forbidden.ts      // banned subjects and banned phrasings
  system/
    auth.ts  account.ts  sync.ts  a11y.ts  browser.ts   // matter-of-fact copy, keyed by error code
```

`forbidden.ts` is the enforcement point for the product's refusals in prose, and it is tested against the entire template corpus and against generated output in fuzz runs:

- **Banned subjects:** the user's behavior in any form — visit frequency, session counts, elapsed absence, "you," second person entirely, comparisons across time framed as the user's doing.
- **Banned phrasings:** `welcome`, `back`, `streak`, `achievement`, `unlocked`, `badge`, `level`, `score`, `points`, `days in a row`, `you've`, `great job`, `congratulations`, exclamation marks, title case, emoji.
- **Required form (naturalist):** lowercase, present tense, a named bird or a species descriptor, one concrete observable detail.

A property test generates 10,000 narration/caption/notebook outputs across randomized state and asserts every one satisfies the required form and violates no ban. This is how INV-2 and INV-12 survive eighteen months of contributors: violating them fails a test rather than requiring a reviewer to notice tone.

### 10.7 Screen-reader narration

**Generated client-side**, by `@aviary/prose/naturalist/narration.ts`, from the same snapshot the renderer uses. Client-side because it must reflect exactly what is being rendered right now, must not cost a round trip, and must respond immediately to user-initiated events; the notebook stays server-side because it needs multi-day history. Both use one lexicon and one grammar, so a screen-reader user moving between the aviary and the notebook hears one product.

Two live regions, both `aria-live="polite"` — **never `assertive`**, because interrupting a screen-reader user's speech is the auditory form of a toast (INV-2):

| Region | Content | Cadence |
|---|---|---|
| Idle | Running observation of the scene | One utterance per 30–60s (jittered) |
| Priority | Return-greeting, offer reaction, settle, listen-in engage/disengage | Prompt, ≤ 1 per 8s, suppresses idle for 10s after |

Composition: 1–3 clauses drawn from `{bird posture and place, a second bird, light/time, weather, a call just heard}`, with an anti-repetition ring buffer over the last 8 utterances at both template and content-word level. Example idle output:

> pip is low on the front rail, feathers loose. wren watches from further back. it is late afternoon; the light has gone warm.

Example priority output on a greeting:

> wren steps toward the front rail and calls twice.

Never state-lists ("pip: perch 2, mood content"), never trait numbers, never mood labels as labels. Mood is conveyed through its observable consequences — "feathers loose," "scanning the back of the scene," "sitting low" — exactly as the visual surface conveys it. Rate-limited and coalescing: if three things happen in two seconds, they become one sentence, not three utterances.

### 10.8 Captions

Generated from the **call spec object the synthesizer actually consumed**, by a pure function `captionFor(callSpec) → string`. Same object, therefore guaranteed to match what was heard; there is no lookup table and no stored per-call string. Syllable shapes and counts map to prose:

> a soft three-note rise · a low trill, paused, low trill again · a single sharp call from the back perch

Rendered as small text near the calling bird, fading in with the call and out ~2.5–4s after, with collision avoidance between simultaneous captions, and mirrored into a third polite live region for AT users who enable them. Captions get a subtle scrim so they hold AA against every sky state. On by default when WebAudio is unavailable (§8.8) or audio is off; otherwise opt-in.

Captions are the single sanctioned exception to "no UI chrome inside the aviary." They are text, they are opt-in, and they are the accessibility surface the PRD explicitly requires there.

### 10.9 Keyboard navigation and focus

Focus model: an absolutely-positioned transparent `<button>` per bird, over the canvas, inside a `role="group"` labelled "aviary". Positions sync at **10 Hz, not per frame** — assistive technology re-reads on DOM mutation, and 60Hz position updates would flood it. Each button's accessible name is the bird's name plus a short current observation ("pip, low on the front rail"), refreshed at the same 10 Hz with change detection so the name only mutates when the words actually differ.

| Key | Action |
|---|---|
| `Tab` | Through top bar items, then into the aviary group (first bird) |
| `←` / `→` | Previous/next bird in `scene_order` |
| `↑` / `↓` | Between perch zones (nearest bird in the target zone) |
| `Enter` / `Space` | Toggle listen-in on the focused bird (`aria-pressed`) |
| `Escape` | Exit listen-in; second press moves focus out of the aviary |
| `Tab` from last bird | Out of the aviary group |

Offer, notebook, settle, and settings are top-bar buttons, each fully keyboard-operable; the offer panel is a proper modal dialog with focus trap, labelled controls, and `Escape` to dismiss. Single-letter shortcuts are **not** implemented: they collide with screen-reader quick-nav keys, and the top bar is two `Tab` presses away.

Focus indicator: a dual-tone ring (2px dark inner + 2px light outer, 2px offset) so it clears 3:1 against bright midday sky and against night dim without needing per-state variants. The top bar **never fades while any descendant has focus, and always returns to full opacity on `keydown`** — the fade is a cursor-stillness affordance and must never hide the controls from someone navigating by keyboard.

### 10.10 Accessibility verification

- `axe-core` on every route in CI, zero violations, build-blocking.
- Canvas-aware contrast test: render each state (dawn/midday/evening/night × clear/rain), sample actual pixels behind every text element and focus ring, assert ratios. Standard tooling cannot see through a canvas, so this is bespoke and necessary.
- Reduced-motion screenshot suite paired with the default suite; a visual feature landing without its reduced-motion counterpart fails.
- Manual scripts per release: NVDA/Firefox, VoiceOver/Safari, VoiceOver/iOS, keyboard-only, 200% zoom, forced-colors mode.
- External audit at the end of Phase 3 and again pre-GA, with a brief that explicitly asks whether the accessible experience delivers the product's affective core — not whether it passes AA. A pass on AA with a flat, state-list feel is a **fail** by this plan's success criteria (§1.4.4).

---

## 11. Performance budgets and observability

### 11.1 Budgets, owners, gates

| Budget | Target | Measurement | Gate |
|---|---|---|---|
| Initial JS, aviary route | **≤ 2MB gzip (PRD ceiling); internal target ≤ 450KB**, critical module ≤ 160KB | `size-limit` per entry point | CI fail |
| Total initial transfer | ≤ 700KB | Lighthouse CI | CI fail |
| Time to first bird | **< 500ms** p75 on Moto-G-class / 4G | Lighthouse CI throttled, every PR; weekly real-device lab | CI fail on regression > 5% |
| First contentful paint (quiet field) | < 200ms | Same | CI warn |
| Idle frame time, 7 birds | p95 ≤ 16.7ms, p99 ≤ 25ms, no frame > 50ms, 30-min soak, reference 5-year-old laptop | Nightly CDP soak | Nightly fail |
| Heap growth, 30 min | ≤ 2MB slope over 3 samples | Nightly CDP heap snapshots | Nightly fail |
| WebAudio node count | Bounded; ≤ 8 voices + 7 buses + fixed graph, non-increasing after warm-up | Nightly assertion | Nightly fail |
| Sim tick latency | p50 < 50ms, **p99 < 5s alarms** (PRD) | Server histogram | Page on alarm |
| Tick lateness | p99 < 20s past `next_tick_at` | Server histogram | Page |
| Snapshot API | p99 < 120ms server-side; cache hit ≥ 85% | Server histogram | Alert |
| Snapshot payload | ≤ 12KB gzip at 7 birds | Contract test | CI fail |
| Availability | 99.9% snapshot + auth monthly | Synthetic + RUM | SLO review |

Note the deliberate gap between the PRD's 2MB ceiling and the 450KB internal target: 2MB is where the *affective* budget becomes unrecoverable, not a place to aim. Parsing 2MB of JS on a mid-tier phone alone consumes more than the entire 500ms first-bird budget, so the real constraint is the 500ms number and the bundle target is derived from it.

The 500ms breakdown we build against: 130ms edge HTML including inlined snapshot, 90ms critical module fetch, 80ms parse+execute, 60ms layout + first draw from vector silhouettes, 140ms slack. Atlas decode, audio init, narration, and the top bar are all outside it by construction.

### 11.2 What we measure

Aggregate only, no account dimension, ever:

- **Client RUM:** navigation timings, time-to-first-bird, p50/p95/p99 frame time bucketed by device class, ornament-degradation trigger rate, audio-context state outcomes, synthesis underruns, snapshot fetch latency and error rate, forward-window-exhaustion events, JS error rates by route and browser.
- **Server:** request rate/latency/error by route, tick latency and lateness histograms, batch sizes, events ingested per second, ingest rejection reasons (`stale`, `duplicate`, `rate_limited`), snapshot cache hit rate, notebook entries generated per 1000 aviary-days, magic-link send/consume success, invite send/consume/revoke counts, email provider failover events.
- **Counts of accessibility settings enabled** (reduced-motion: N, captions: N, audio-off: N) — population counts with no account dimension, so we can tell whether the surfaces are used without knowing who uses them.
- **Synthetic fleet:** automated browsers from 4 geographies every 15 minutes running a scripted session — load, assert first bird < 500ms, assert a greeting occurred, assert a call was synthesized, listen-in, offer, settle — with screenshots on failure.

### 11.3 What we deliberately do not measure

This list is a design deliverable, not an omission (INV-3, INV-9):

- No per-account or per-bird anything in telemetry: no presence-time, no session counts, no drift values, no mood distributions, no interaction counts, no per-account funnels.
- No retention, DAU/MAU, cohort, or engagement metrics of any kind. These are the metrics that generate pressure for a streak, and their absence is the point.
- No cross-account drift analysis, even anonymized ("average plumage after 30 days" is a data product built from private relationships).
- No A/B experimentation framework on the aviary surface. There is no infrastructure to assign accounts to variants, which forecloses experimenting our way into an engagement feature.
- No session recording, heatmaps, or click analytics.

**Enforcement is architectural, not policy:**

1. The metrics client takes a typed label set; `account_id`, `bird_id`, `email`, `session_id`, `invite_id`, and `visitor_email` are not assignable to it, and a CI check greps for them in metric/log/trace call sites.
2. The simulation Postgres lives in a subnet with no route to the analytics VPC. The warehouse has no credential for it. There is no logical-replication slot, no CDC connector, and an infra-policy test asserts the Terraform plan creates neither.
3. Application logs are structured with an allow-listed field set; `account_id` is permitted in error logs (we must be able to debug an account's failures — "is this account having errors" is allowed) while per-bird state fields are not, and the logger drops unknown keys rather than passing them through.
4. The privacy policy in account settings names the aggregate categories in plain text and explicitly excludes per-bird interaction state, so the commitment is externally legible.

**Accepted cost, stated plainly:** we are choosing to run this product without behavioral analytics. Product decisions come from synthetic testing, dogfood, qualitative research, and support contacts. When someone asks "are users engaging with the notebook," the honest answer is that we can see aggregate notebook route requests and nothing more, and we will not build the pipeline that would answer it better. That tradeoff is deliberate and should be re-stated in the first post-launch planning meeting rather than discovered.

---

## 12. Testing and quality strategy

### 12.1 Engine unit and property tests

Pure functions with an injected clock and seeded PRNG make the engine exhaustively testable. Property tests: drift monotonicity over random event streams (INV-4); tick determinism (same inputs → identical outputs, 10k random cases); tick idempotency (crash-and-retry equals single execution); perch matching never exceeds slot capacity and never leaves a bird unassigned; mood dwell minimums respected; call scheduling always covers the forward window; greeting selection never fires twice for one resumption.

### 12.2 Calibration tests

The §7.4 profile table, run in CI. These are the tests that would catch the PRD's named silent failures — over-fast drift, farmable drift, double-credited presence, drift-on-neglect — and they run in seconds because the tick is a pure function of an injected clock.

### 12.3 Presence fixture tests

Recorded ping sequences representing: continuous watching; watching with 3-minute still periods; tab hidden mid-session; window blurred but visible; laptop suspended 8 hours then resumed; two devices overlapping; a malicious client pinging every second unconditionally; clock skew ±5 minutes. Each asserts exact credited seconds. This is the highest-leverage test suite in the plan, because presence miscounting is the PRD's canonical example of a corruption no test would catch — so we write the test.

### 12.4 Client tests

Deterministic render tests (fixed snapshot + fixed clock + fixed seed → golden frame hash) across viewport × lighting × weather × bird-count × reduced-motion. Perf soak under CDP for frames and heap. Audio graph assertions: node count bounded, no gain ever set to zero on a non-focused bus, no direct `gain.value` assignment, every mix change is a ramp. Snapshot-stale simulation: assert the client keeps rendering and calling with no snapshot for 10 minutes.

### 12.5 The refusals suite

Automated checks for the product's absolute rules, because "we agreed" does not scale:

- Aviary route bundle contains no string matching the gamification/announcement lexicon (`streak`, `achievement`, `badge`, `level`, `score`, `xp`, `welcome back`, `days in a row`, `you've been`, `congratulations`, `unlocked`).
- No toast, banner, snackbar, or notification component is reachable from the aviary entry point (module-graph assertion).
- OpenAPI document contains no path or property matching the §6.6 deny-list.
- Snapshot response schema contains none of the five trait names and no `mood` field (INV-6).
- No `email` as a metric label, log field, cache-key prefix, or column outside `accounts` and `visit_invites` (INV-10).
- Terraform plan creates no replication slot, CDC connector, or cross-VPC route from the simulation subnet (INV-9).
- No user-facing string literal outside `@aviary/prose` (INV-12).
- Prose corpus and 10k generated outputs satisfy `forbidden.ts` (INV-2).
- No `Date.now`/`new Date()`/`Math.random` inside `packages/engine`.
- No `frames[]`-style keyframe animation type in the bird render module (INV-1).
- Mailer template registry has exactly four entries.

### 12.6 Definition of done

A visual or behavioral feature is not done until it has: reduced-motion treatment; narration coverage (or a documented reason it produces no utterance); caption coverage if it makes a sound; keyboard reachability and a focus treatment; AA contrast verification against all lighting states; a perf measurement showing budgets held; and prose reviewed by the writer if it produces any user-visible words. **Accessibility and voice are not separate workstreams to be swept up later — they are completion criteria** (INV-11).

### 12.7 Load and failure testing

Tick throughput at 10× projected accounts with the hibernation mix (§7.9). Event ingest at 20× peak. Snapshot read at 50× with cache. Chaos drills: kill a `sim-worker` mid-batch (assert lease expiry and identical replay); Redis down (assert snapshot reads degrade to database with acceptable latency, no correctness loss); primary email provider down (assert failover); Postgres failover during a tick (assert no partial state write, no personality corruption).

---

## 13. Rollout

### 13.1 Build phases

Timeline assumes ~6 engineers, 1 visual designer, 1 sound designer, 1 writer. Sound design and prose start in Phase 1, not Phase 4 — both are long-lead creative work on the critical path, and both are the sort of thing that gets compressed into "engineer writes placeholder" if it starts late, which is how procedural calls end up sounding like a test tone and notebook entries end up reading like a log.

| Phase | Weeks | Content | Exit criteria |
|---|---|---|---|
| **0 — Foundations** | 0–3 | Monorepo, CI with budgets-as-gates, Postgres schema + grants + monotonicity trigger, synthetic-UUID account model with encrypted email + blind index, magic-link auth, tick scheduler with leasing, event ingest, snapshot endpoint, edge HTML with inlined bootstrap, canvas harness with one bird and one motif | One bird visible < 500ms in the lab; tick advances mood; refusals suite green |
| **1 — Bird engine** | 3–8 | Full personality/drift/mood/perch/call scheduling; calibration harness and fitting; 2 species with sound-designer-authored motif libraries; synthesis graph and voice pool; listen-in mix; presence accounting + fixture tests; return-greeting pipeline | Calibration profiles pass; two birds recognizable by ear in an internal listen; presence fixtures exact |
| **2 — The aviary as a place** | 8–13 | Remaining 4 species; day/night; weather; ornaments; parallax; adaptive quality; top bar with fade; offers with cooldowns; settle + 5s undo; adoption flow with naming; responsive/no-crop matrix; first-frame-in-motion path complete | 7-bird scene at 60fps on reference laptop; no load state on any path; no chrome in the scene |
| **3 — Accessibility as a designed surface** | 11–16 (overlaps 2) | Prose package; narration composer + live regions; caption generator; reduced-motion render mode with its own designed poses; keyboard nav; focus treatment; canvas contrast tests; settings surfaces in matter-of-fact voice | axe clean; external audit round 1 passed **on affective grounds**, not just AA |
| **4 — Notebook, account, social** | 14–18 | Notebook writer + sparsity + corpus review; notebook UI (virtualized, code-split); session list and revoke; email change; export; soft/hard delete + purge job; visit invites, visitor route, revocation, log, expiry, opt-in notify toggle | Notebook corpus passes prose review; visitor route provably cannot write events; deletion verified complete including backup window |
| **5 — Hardening** | 18–21 | Perf soak; memory; synthetic fleet; listening study (§8.7); external a11y audit round 2; privacy boundary review; load and chaos tests; runbooks | All budgets green; listening study gates met; privacy review signed off |

### 13.2 Launch ramp

1. **Internal dogfood — minimum 4 real weeks, overlapping Phases 3–5.** ~30 accounts. Four weeks is not schedule padding: the three-week drift claim can only be validated in real elapsed time. The accelerated-clock harness validates the arithmetic; only a human who has watched the same two birds for three weeks can validate that the change is *felt*. Structured week-3 interview leads with an unprompted "has anything about your birds changed?" before any specific question, because a prompted yes tells us nothing.
2. **Closed beta — 500 accounts, ~4 weeks.** Watch tick capacity, first-bird timings across real devices and geographies, audio-context failure rates by browser, notebook sparsity in the wild, and support contacts. The time-dilation test mode exists only in test builds and is asserted absent from production bundles.
3. **Open with a waitlist drip.** Admit in batches sized so tick capacity leads demand. Ramp gate: tick lateness p99 and snapshot p99 both inside budget for 72 hours before each batch.

Deliberately absent from the launch plan: launch email campaigns to existing users, re-engagement email, "your aviary misses you" of any kind (INV-2, and the PRD's not-a-notification-surface rule).

### 13.3 Birds-per-aviary ramp

The offer schedule is **server config**, not code: 3rd bird at aviary age ~90 days, 4th ~180, 5th ~300, 6th ~450, 7th ~600. Gated on age alone — not visits, not interactions, not payment (INV-3).

Consequence worth planning around: no GA account reaches three birds for three months, so 3–7 bird behavior is validated pre-launch on **seeded aviaries** (fixtures with synthetic age and drift history — created only through a test-only fixture path that does not exist in production builds) plus the §8.7 listening study.

Contingency: if the study shows recognizability failing at 7, set the config cap to 5 and treat 6–7 as post-v1 audio-mix work. Since no account reaches 5 birds for ten months, this decision can be revisited with real listening data long before it binds. The seven-bird cap remains a hard engine limit in code regardless of the config value — perch slots, voice pool, and bus count are all sized to 7 and asserted.

### 13.4 Day-one instrumentation

Everything in §11.2, live from the first beta account, with dashboards and alerts for: first-bird p75 by device class, frame-time p95, tick latency p99 (5s page per PRD), tick lateness, snapshot error rate and cache hit rate, event ingest rejection reasons, audio-context failure rate by browser, magic-link consume success rate (a drop here is users locked out of their own birds — highest-severity page), invite delivery, and notebook entries per 1000 aviary-days (sparsity regression detector).

Nothing in §11.3, from day one either — the absence needs to be established at launch, because retrofitting a privacy boundary onto a live pipeline never happens.

### 13.5 Runbooks

Written in Phase 5: tick backlog (scale replicas, verify no double-tick); tick storm after outage (hibernation makes catch-up cheap; cap catch-up to one logical tick, never replay every missed interval, because 1,440 replayed ticks would compress a day of mood transitions into a minute of visible chaos); monotonicity trigger firing (a real bug — freeze the aviary's ticks, do not clamp, page the engine owner); email provider failure; `α` config change procedure (reviewed release, changelog, never a runtime toggle); suspected personality corruption (the vector is canonical and unrecoverable from logs, so the response is PITR of the affected rows, not recomputation — and this is exactly why the calibration conservatism in §7.4 matters).

---

## 14. Risks

Ordered by expected cost, each with detection and response. The four the PRD names — drift calibration, sync correctness, audio uncanniness, accessibility regression — are R1, R2, R3, R5.

**R1 — Drift calibration is wrong, and being wrong in one direction is permanent.**
Monotonic drift means an over-fast `α` cannot be undone: you cannot walk plumage back down without rewriting vectors, and rewriting vectors is the INV-7 failure the PRD calls the worst thing that can happen to this product. Too slow is a config change; too fast is permanent population-wide damage.
*Detection:* the §7.4 harness pre-launch; week-3 dogfood interviews; a low-cardinality **aggregate** drift-band histogram (percentage of birds by plumage tier at 30/60/90 days of aviary age, with no account dimension and no cross-account comparison surface — this is the one aggregate over vectors we allow, because launching an unrecoverable parameter completely blind is a worse risk than the narrow aggregate; it is documented in the privacy policy).
*Response:* launch at 0.6× fitted `α`, per-trait species ceilings, daily saturation caps, reviewed config releases only. Raise `α` after ≥ 6 weeks of dogfood signal, never lower it in a way that implies a rewrite.

**R2 — Sync correctness fails silently.**
Multi-device presence double-credit, late-arriving presence lumps, non-idempotent ticks, a well-meaning "just let the client set this" patch.
*Detection:* presence fixtures (§12.3); tick determinism and idempotency properties; an invariant alarm if credited presence exceeds wall-clock for any aviary-day; the monotonicity trigger; the DB grant that makes the client-write patch fail at runtime in development immediately.
*Response:* every mechanism is structural (grants, unique indexes, server-assigned ordering, interval union) rather than conditional, so the failure mode is a loud error rather than quiet data loss.

**R3 — Procedural calls sound synthetic, or the chorus turns to mush.**
The highest-variance risk in the plan, because the audio is the affective spine and "procedurally generated bird call that sounds like a bird" is genuinely hard.
*Detection:* internal listens from Phase 1 (not Phase 5 — we must know early); the §8.7 study; the CI spectral-distance proxy on every motif change.
*Response:* the motif libraries are authored by a sound designer in a DSL, not tuned by engineers; the synthesis graph (formant bank, FDN reverb, per-bird timbre) is built to be expressive enough for a designer to work in. If a species cannot be made to sound right, **cut it and ship five** — the PRD says "about six." If the chorus blurs, reduce the config cap (§13.3). What we do not do under any schedule pressure is ship a recorded fallback; there is no decoder in the codebase, which makes that decision hard to reverse under pressure, deliberately.

**R4 — The 500ms first-bird budget is missed on real mid-tier devices.**
Lighthouse throttling flatters real hardware.
*Detection:* weekly real-device lab runs from Phase 0, not Phase 5; RUM p75 by device class in beta.
*Response:* the vector-first bird (§10.4) removes atlas decode from the critical path; the inlined bootstrap snapshot removes an RTT; further headroom comes from splitting the critical module further (audio, narration, and captions are already out). If a device class still misses, the graceful answer is fewer initial ornaments and a simpler first-frame silhouette — never a load state.

**R5 — Accessibility regresses after launch.**
Narration slides toward state-lists as features are added; reduced-motion falls behind the default renderer; a new surface ships without keyboard support.
*Detection:* axe in CI; paired reduced-motion screenshots; the prose property tests; quarterly external audit.
*Response:* §12.6's definition of done, plus the prose package as the only source of user-facing words, plus the reduced-motion mode being a peer render mode rather than a flag — so "forgot to handle reduced motion" is a missing implementation that fails a screenshot test rather than a silently-degraded experience.

**R6 — Charm erosion by accumulation.**
Not one bad decision; forty reasonable ones. A toast for a sync error. A "3 birds" count in settings. A calendar in the export. A "welcome back" in an email. Each defensible, cumulatively fatal.
*Detection:* the refusals suite (§12.5) catches the mechanical forms; a PR template question ("does this add a surface that announces, counts, ranks, or numbers?") catches the rest.
*Response:* the invariant register (§0) with named owners, and the practice of encoding each refusal as a test at the moment it is decided rather than as a paragraph in a document nobody re-reads.

**R7 — Notebook prose reads canned, or drifts into observing the user.**
Templates repeat; entries get too frequent; someone adds "pip greeted you every day this week" because it is a lovely sentence and it is exactly the forbidden thing.
*Detection:* the sparsity metric (entries per 1000 aviary-days) in day-one instrumentation; anti-repetition over the last 30 entries per aviary; `forbidden.ts` property tests; writer review of the corpus pre-launch.
*Response:* sparsity is a hard budget (≤ 1 entry per 2 days, ≤ 3 per week, ≤ 1 per session) enforced in the writer; noteworthiness scoring (first-time-this-week orderings, unusual perch choices, chorus events, weather coincidences, long quiets) selects candidates rather than logging events; the banned-subject list makes "observations of the user" a test failure.

**R8 — Tick cost or backlog at scale.**
*Detection:* tick lateness p99, batch size, and queue depth from day one; the §12.7 load test at 10×.
*Response:* hibernation (§7.1), batching, only writing changed rows, horizontal `sim-worker` scaling via `SKIP LOCKED`. The catch-up cap in §13.5 prevents a recovery from producing visible chaos.

**R9 — Magic-link email delivery fails.**
A user who cannot receive the link has lost access to a relationship they have spent weeks building. This is the only total-loss failure in the product that is not our data's fault.
*Detection:* consume-rate-vs-send-rate alert per provider and per recipient domain; highest-severity page.
*Response:* two providers with automatic failover behind one interface, DKIM/SPF/DMARC correctly configured before beta, a human support path, and the account-export feature as a genuine mitigation (a user with an export at least holds a copy of their aviary).

**R10 — Privacy boundary erosion.**
Someone adds a CDC connector, a debug export, a "temporary" join between the simulation database and the warehouse.
*Detection:* the infra-policy CI check; a named data-boundary owner reviewing any change touching either subnet.
*Response:* network isolation plus separate credentials mean the erosion requires an infrastructure change, which is reviewable, rather than a query, which is not.

---

## 15. Decisions this plan makes where the PRD is open

Listed so a reviewer can overrule any single one without re-deriving the plan. Each is a defensible reading, not a gap.

| # | Decision | Rationale |
|---|---|---|
| 1 | Presence ping cadence **15s**; input-activity window **4 minutes**; credit `min(gap, 20s)` | PRD says "a few minutes, leaning toward the longer side because watching birds without moving is the actual product" |
| 2 | Tick **60s active / 300s hibernating**, with synchronous catch-up on snapshot read | Preserves "continues without the viewer" observably while cutting cost ~5×; hibernation is unobservable by construction |
| 3 | Mood set is **six** states: the PRD's five plus `settled` | Night and post-settle need a distinct state; the PRD says the exact set is finalized in implementation |
| 4 | `mood` is **not** sent to the client; only its visible consequences | PRD: the user reads mood from the motion. A wire field is one dev-tools panel away from a status dashboard |
| 5 | Only `plumage_tier` (quantized 0–7) crosses the wire; the other four traits never leave the server | Minimum possible leak consistent with rendering plumage at all (INV-6) |
| 6 | Narration generated **client-side**; notebook **server-side**; both from one shared prose package | Narration must match the current frame with no round trip; notebook needs multi-day history. One lexicon keeps the voice single |
| 7 | **Canvas 2D**, not WebGL | 7 birds and subtle parallax don't need it; WebGL costs bundle bytes, context-loss handling, and old-GPU risk against the 2MB/5-year-laptop constraints |
| 8 | **No realtime channel** (no WebSocket/SSE) at v1 | A 60s tick with client interpolation makes polling sufficient; removes a class of connection bugs. Visitor lag ≤ 45s is invisible at this cadence |
| 9 | Listen-in is **not mirrored** across a user's devices | Listen-in is a person's act of attention at a device, not aviary state; mirroring lets one device hijack another's mix |
| 10 | `settle` is the one **last-write-wins** state, resolved by `ingest_seq` | It is presentation state with no history; the exception is documented adjacent to INV-5 so it isn't generalized |
| 11 | Audio-unlock resolution: silent aviary + passive top-bar icon state, resume on first input | A "tap to begin" gate installs exactly the entry state the PRD forbids, on the first frame of the first session |
| 12 | Ship **α at 0.6× fitted**; raise only after ≥ 6 weeks of dogfood | Too-fast monotonic drift is unrecoverable; too-slow is a config change. Asymmetric cost ⇒ asymmetric caution |
| 13 | Per-trait **species ceilings** fixed at adoption | Without them, monotonic drift homogenizes every long-tenured aviary into identical maximal birds |
| 14 | Event log retention **45 days**; personality never recomputed from it | Legal under "personality is stored, not derived"; minimizes retained interaction history |
| 15 | Export contains vectors (PRD says so) but no in-app viewer, parser, or import exists | INV-6 governs product surfaces; the export is data portability. The line is enforced by the absence of client code |
| 16 | Offer schedule 90/180/300/450/600 days; hard code cap 7, **config cap adjustable down** | Age-gated per PRD; lets the §8.7 study move the config cap without touching engine limits |
| 17 | One narrow aggregate over vectors: plumage-tier histogram by aviary age, no account dimension | Launching an unrecoverable parameter completely blind is a worse risk; documented in the privacy policy |
| 18 | Species pool may ship as **five** if one cannot be made to sound right | PRD says "about six"; recognizability is the binding constraint, not count |
| 19 | Backup PITR retention **35 days** so a day-30 hard delete fully ages out | Makes deletion completeness a property of the architecture rather than a manual workflow |
| 20 | Timezone: client authoritative for lighting, `tz_iana` hint for server-side mood while no client is connected | Lighting must never be stale; the tick must still evaluate time-of-day for an unwatched aviary |

---

## 16. Team shape and sequencing note

Six engineers map cleanly onto the boundaries this plan already draws: **engine** (2 — tick, drift, mood, calls, calibration), **client render** (2 — scene, actors, reduced-motion mode, perf), **audio** (1, paired with the sound designer from Phase 1), **platform** (1 — auth, accounts, sync, visits, privacy boundary, infra). Accessibility is not a person; it is a completion criterion on every one of those seats (§12.6), with the external audit as the independent check.

The two things that will be under the most schedule pressure and must not be compressed are the **four real weeks of dogfood** (the only way to validate a three-week drift claim) and the **Phase 1 start for sound design and prose** (both are creative long-lead work that becomes engineer-placeholder work if it starts late, and placeholder calls and placeholder prose are precisely the "canned" failure that the PRD says the user detects on the first session and does not forgive).
