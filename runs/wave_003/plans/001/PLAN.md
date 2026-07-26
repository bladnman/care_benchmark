# Pocket Aviary — v1 Implementation Plan

**Status:** Phase-1 plan. Implementation-ready. No product code is written by this document.
**Audience:** A frontier engineering team (≈6–8 engineers + 1 visual designer + 1 audio-capable generalist) executing v1 without further clarification from the PRD author.

---

## 0. How to read this plan

The PRD is unusually opinionated about *feel*, and most of its hard requirements are affective rather than functional. That means a normal engineering plan — one that decomposes features into services and endpoints — will pass every acceptance test and still ship the wrong product. So this plan does three things at once:

1. **Decomposes the system** (architecture, data model, API, engine, sync, render, audio, a11y, perf, rollout, risk).
2. **Names the affective invariants** that must survive implementation, and binds each to a mechanism (a lint rule, a CI test, a code-review gate) rather than to good intentions. Anything defended only by a paragraph in a doc will be violated within two quarters.
3. **Records the judgment calls** the PRD left open, with the reasoning, so the team doesn't relitigate them. These are collected in §14 and flagged inline as **[CALL]**.

The single most useful framing for the team: *this is a simulation product with a rendering client, not a web app with animations.* The server owns a continuously-running world; the browser is a viewport onto it. Every time a decision is ambiguous, resolve it in the direction that makes the world more independent of the viewer.

---

## 1. Scope

### 1.1 In scope for v1

**Accounts & identity**
- Email + magic-link sign-in. No passwords, no SSO.
- 15-minute link expiry; single-use consumption; per-email rate limiting.
- Synthetic UUID account identifier; email encrypted at rest, stored exactly once.
- Per-device revocable session tokens; session list in account settings.
- Email change with verify-before-commit (old address remains valid until the new one verifies).
- On-demand JSON account export, delivered as an emailed download link.
- Soft delete (30 days, recoverable by signing in) → hard delete.

**The aviary**
- One canonical aviary per account. Two starter birds at adoption; hard cap of seven.
- Server-side simulation tick (~60s) advancing personality drift, mood, position, call scheduling, weather, and day/night, running whether or not a client is connected.
- Six-species pool with distinct silhouettes, palettes, and call-motif libraries.
- Stable per-bird internal identity, invariant across rename, sync, species-pool changes, and internal migrations.
- Age-gated new-bird offers (aviary age only — never visit count, interaction volume, or payment).

**Interactions**
- Return-greeting: one bird notices the user within ~1–2s of the aviary becoming visible; varied by boldness, mood, and absence length; procedurally generated, never a canned variant.
- Listen-in: focus one bird, its call rises in the mix, others fall to ambient but never mute. Slow ramps both ways.
- Offer: seed, song fragment, still pool. Reachable from the top bar only. Per-bird cooldown (a few minutes).
- Settle: soft evening lighting shift, calls quiet, 5-second undo on any aviary click.
- Presence accounting: the three-signal conjunction (visible ∧ focused ∧ recent pointer/key activity).
- Field notebook: auto-generated, read-only, sparse naturalist observations, scrollable indefinitely.

**Social (one feature, deliberately)**
- Per-invite, email-addressed, opt-in visit invitations. Off by default for new accounts.
- Read-only ambient visitor view; no interaction, no presence contribution, no co-presence.
- Immediate revocation, effective at the visitor's next snapshot pull.
- 30-day invite expiry, non-revivable.
- Silent visit logging; host-visible visit log in settings; optional per-account visit notification toggle, off by default.

**Accessibility (ships with v1, not after)**
- Screen-reader narration as running naturalist prose on a 30–60s idle cadence, with priority bumps for user-initiated events.
- Reduced-motion mode as a *designed alternate rendering* (slow cross-fades between poses), not "animations off."
- Runtime-generated call captions in naturalist prose, matching what was actually synthesized.
- Full keyboard navigation with focus indicators legible against both bright and dim aviary states.
- WCAG AA contrast on all user copy.

**Performance**
- Initial JS bundle ≤ 2MB gzipped at first paint.
- Time-to-first-bird < 500ms on mid-tier mobile over 4G.
- 60fps idle motion on a 5-year-old mid-range laptop, sustained across a 30-minute session.
- Zero memory growth across a 30-minute session, enforced in CI.
- Client-side WebAudio procedural synthesis; graceful silence + captions-on when WebAudio is unavailable.
- Browser support: last two major versions of Chrome, Safari, Firefox, Edge.

**Observability**
- Aggregate-only operational telemetry, architecturally separated from the simulation database.
- Synthetic performance monitoring fleet.
- Simulation-tick latency p99 alarm at 5s.

### 1.2 Explicitly out of scope

Native mobile apps (and we do not shape the data model or protocols around a future native client). All gamification — achievements, streaks, levels, scores, badges, counters, green-dot calendars, XP, ranks, tiers, milestone celebrations, in any tier or toggle. Tamagotchi mechanics — death, hunger, distress, decaying happiness meters, negative drift. Social-network surfaces — profiles, follows, feeds, discovery, friend-of-friend, mutual visits, comments, leaderboards, chat, avatars, show-off rendering. Push notifications and marketing email about the aviary. Payments. Shared or multi-aviary accounts. Customizable scenes. Panning, scrolling, or zooming the aviary. Any UI that exposes personality vector numbers.

### 1.3 Non-negotiable invariants (the "teeth" list)

These are elevated out of prose because each one is a load-bearing property that fails silently. Each gets an enforcement mechanism in §12.

| # | Invariant | Enforced by |
|---|---|---|
| I1 | Only the simulation tick writes personality vectors. No client code path, no admin path, no migration path mutates them outside the tick. | DB grants + repository-layer type split + CI grep + integration test |
| I2 | Drift is monotonic non-decreasing per trait. | Property test in the tick; runtime assertion on delta sign; alarm on violation |
| I3 | Personality vector values never cross the API boundary in any form (numeric, bucketed, or ordinally labeled). | Response-schema allowlist test; snapshot contract test |
| I4 | Presence requires the conjunction of visibility ∧ focus ∧ recent input. | Client unit tests + server-side plausibility validation + tick-level rejection |
| I5 | No recorded audio ships. Zero audio binaries in the bundle. | Bundle-composition CI check (fails on any audio MIME asset) |
| I6 | No announcement surfaces: no toast/banner/modal/badge welcoming, congratulating, or counting the user. | Component-inventory lint (banned primitives) + a11y-tree snapshot test + design review gate |
| I7 | No visit-frequency, streak, count, or calendar surface anywhere, including settings and export. | Copy lint on a banned-lexicon list + export-schema test |
| I8 | Email appears in exactly one column of one table, encrypted. Every other reference is the synthetic UUID. | Static analysis on identifier types + log-scrubber test + schema review |
| I9 | Per-account/per-bird interaction state never reaches the analytics warehouse or any training pipeline. | Network-policy isolation + metric-definition review + pipeline schema allowlist |
| I10 | Visitor sessions contribute zero drift inputs and zero presence. | Session-type flag threaded to the event writer; test that visitor events are dropped at ingest |
| I11 | Bird internal IDs are stable forever. No reset, regenerate, or swap code path exists. | Migration review checklist + FK constraints + no-delete policy on bird rows |
| I12 | The aviary never presents a spinner, entry animation, or fade-from-static. | Loading-state component allowlist + visual regression test on cold-load first frame |

---

## 2. Architecture

### 2.1 Shape

Five deployable units. The split is driven by the privacy boundary and by the tick's very different runtime profile from request serving — not by microservice fashion.

```
                         ┌──────────────────────────┐
   browser  ──HTTPS──►   │  edge (CDN + SSR shell)  │  HTML + inlined
   (client)              │                          │  bootstrap snapshot
                         └────────────┬─────────────┘
                                      │
                         ┌────────────▼─────────────┐
                         │      aviary-api          │  read snapshots,
                         │  (stateless, autoscaled) │  write events,
                         │                          │  auth, settings,
                         └───┬───────────────┬──────┘  visits, notebook
                             │               │
              ┌──────────────▼───┐      ┌────▼──────────────┐
              │  sim-db          │      │  event-log        │
              │  (Postgres)      │      │  (Postgres,       │
              │  canonical state │      │   append-only)    │
              └──────────▲───────┘      └────┬──────────────┘
                         │                   │
              ┌──────────┴───────────────────▼──┐
              │        sim-tick worker           │  the only writer
              │  (partitioned, ~60s cadence)     │  of personality
              └──────────────────────────────────┘

              ┌──────────────────────────────────┐
              │  notify (transactional email)    │  magic links, invites,
              └──────────────────────────────────┘  export links only

   ══════════ hard network boundary ══════════
              ┌──────────────────────────────────┐
              │  telemetry / ops (separate VPC)  │  aggregate only;
              └──────────────────────────────────┘  no route to sim-db
```

**Why these boundaries:**

- **`sim-tick` is a separate deployable** because it must keep running at cadence during an API deploy, an API incident, or an API scale-down at 3am. The PRD's central claim — the aviary continues without the viewer — is falsified the first time a routine API rollout skips ninety seconds of ticks for the whole population. It also has an opposite scaling signal (steady CPU work proportional to *account count*, not to *request rate*).
- **`event-log` is physically separate from `sim-db`** so that the write path (high-frequency, append-only, tolerant of brief unavailability) cannot lock or bloat the canonical state store, and so that retention policies differ (see §4.7).
- **Telemetry lives in a different VPC with no network route to `sim-db`.** I9 is an architectural fact, not a policy. If a future engineer wants per-bird analytics, they have to file a networking change, which is exactly the friction we want.
- **The edge tier is not just a CDN.** It serves the HTML shell *with the bootstrap snapshot inlined* (§8.2) — this is the single highest-leverage decision for the 500ms budget, because it removes a full round-trip from the critical path.

### 2.2 Technology choices **[CALL]**

The PRD doesn't specify a stack. These are chosen for the budgets, not for team preference; substitutions are fine if they hold the same properties.

- **Client:** TypeScript, no UI framework in the critical path. The aviary scene is a hand-written render loop over a canvas/WebGL surface (§7). Chrome surfaces (top bar, settings, notebook, offer sheet) use Preact — small, code-split out of the critical bundle, and never blocking first bird. A React-in-the-critical-path decision would spend 40–50KB gz on a surface that has two buttons in it and would tempt the team into rendering birds through a virtual DOM, which cannot hold 60fps for seven animated entities plus ambient ornaments.
- **Server:** Go for `aviary-api` and `sim-tick`. Predictable latency, cheap goroutine fan-out for tick partitions, tight p99 control (we have a hard 5s p99 alarm). Node would work; the tick's CPU profile argues against it.
- **Datastores:** Postgres for both `sim-db` and `event-log`. The dataset is small (kilobytes per account), relational, and needs strong ordering guarantees on the event log — which is exactly Postgres's strength and exactly what a document store makes harder. Redis for presence-window scratch state, per-bird offer cooldowns, magic-link nonces, and rate limiting.
- **Email:** one transactional provider, used only for magic links, invites, export links, and (opt-in) visit notifications. The provider integration is deliberately incapable of sending anything else — no template registry, no campaign API surface, no marketing list sync. This is I-list adjacent: the cheapest way to guarantee we never become a notification surface is to not build the capability.

### 2.3 Client/server split — the rule

> **The server owns everything that persists or that another device must agree about. The client owns everything that is purely how the current frame looks.**

Concretely:

| Server-owned | Client-owned |
|---|---|
| Personality vectors, drift | Interpolation between snapshot positions |
| Mood and mood timers | Idle micro-motion phase and pose selection |
| Perch assignment and target positions | The animated path between perches |
| Call scheduling (which bird calls, when, in what mood) | Call *synthesis* (the actual waveform) |
| Weather events and their windows | Leaf/feather ornaments (pure decoration, no state) |
| Day/night phase (computed from the account's IANA timezone) | Palette interpolation between phase keyframes |
| Notebook entry generation | Notebook scroll/virtualization |
| Greeting decision (which bird, what form, when) | Greeting's exact motion and waveform realization |

The dividing question for any new behavior: *if two devices are open at once, must they agree?* If yes, server. If no, client. The greeting is the interesting case — both devices must agree that Pip greeted (it's in the notebook, it feeds drift, it's narrated), but they need not agree on the exact waveform. So the server emits a greeting *decision* with a seed; the client realizes it. This pattern — **decision on the server, realization on the client, seeded for determinism** — recurs throughout §5, §7, and §8, and it is the core architectural idea of the product.

### 2.4 Render pipeline boundary

The client's render loop never reads network state directly. There is one intermediate structure:

```
snapshot(s) from server ──► SceneModel ──► Renderer ──► frame
                              (interpolating,          (visual mode:
                               authoritative-ish)       full | reduced)
                                    │
                                    ├──► AudioDirector ──► WebAudio graph
                                    ├──► NarrationComposer ──► aria-live
                                    └──► CaptionComposer ──► caption layer
```

`SceneModel` is the single source of truth for the frame: it holds the last two snapshots, an interpolation clock, and the local realization state (motion phases, ornament particles, active ramps). Renderer, AudioDirector, NarrationComposer, and CaptionComposer are all **pure consumers of `SceneModel`** and never talk to each other.

This matters more than it looks. It is the mechanism that makes accessibility structurally impossible to skew: narration and captions are generated from the same model the pixels are, so they cannot drift out of sync with what's actually happening, and a reduced-motion user gets the same *world* through a different Renderer. If narration were generated from a separate code path, the two would diverge within a release or two and screen-reader users would be told about an aviary that isn't the one running.

---

## 3. Domain model (conceptual)

Before schemas, the object graph the whole team should share:

- An **Account** has exactly one **Aviary**.
- An **Aviary** has 2–7 **Birds**, an **AviaryState** (day phase, weather, settled flag, ambient mood bias), a **Notebook** (list of **NotebookEntry**), a set of **Invites**, and a **VisitLog**.
- A **Bird** has: a permanent `bird_id`, a `species`, a user-assigned `name`, a **PersonalityVector** (5 traits), a **MoodState**, a **PerchState**, a **CallProfile** (derived from species + personality, cached), and a `drift_accumulator`.
- An **InteractionEvent** is an immutable, append-only record with an `account_id`, optional `bird_id`, a type, a timestamp, and a small typed payload. Events are the *only* thing clients produce that affects state.
- A **Snapshot** is a derived, cacheable projection of AviaryState + Birds at a moment, plus a short horizon of *scheduled* future events (calls, motions) so the client can render ahead smoothly.

The word "score," "level," "points," "progress," and "streak" appear nowhere in this model, and any PR that introduces one of them to a type name should be rejected on sight.

---

## 4. Data model

All tables keyed by synthetic UUIDs. Timestamps are `timestamptz` in UTC; the account's IANA timezone is stored separately and used only for day/night and notebook date phrasing.

### 4.1 `accounts`

```sql
account_id            uuid primary key default gen_random_uuid(),
email_encrypted       bytea not null,              -- envelope-encrypted, KMS data key
email_hash            bytea not null unique,       -- HMAC-SHA256(email, pepper) for lookup
email_pending_enc     bytea,                       -- during email change
email_pending_hash    bytea,
timezone              text not null default 'UTC', -- IANA
created_at            timestamptz not null,
deleted_soft_at       timestamptz,                 -- null = active
hard_delete_after     timestamptz,                 -- deleted_soft_at + 30d
settings              jsonb not null default '{}'  -- see 4.9
```

**Notes.** `email_hash` exists because magic-link sign-in must look an account up by email without decrypting anything, and without an index on plaintext. It is an HMAC with a pepper held in KMS, not a bare hash — a bare SHA of an email is trivially reversible against any email list. This is the only column that is functionally derived from the email, and it is not usable as an identifier anywhere downstream because it's a `bytea` that no other table references (I8).

There is no `display_name`, no `avatar_url`, no `bio`. Not because we don't need them yet, but because their absence is the schema-level expression of "not a social network."

### 4.2 `birds`

```sql
bird_id          uuid primary key default gen_random_uuid(),  -- permanent, never reissued
account_id       uuid not null references accounts,
species_id       text not null,           -- 'warbler' | 'wren' | ... (6 in v1)
name             text not null,
adopted_at       timestamptz not null,
-- personality vector, 0..1, monotonic non-decreasing
p_boldness       real not null,
p_social_warmth  real not null,
p_vocal_freq     real not null,
p_plumage_sat    real not null,
p_curiosity      real not null,
-- fractional drift carry (see 5.3)
drift_carry      jsonb not null default '{}',
-- mood
mood             text not null,                    -- see 5.4
mood_entered_at  timestamptz not null,
mood_expires_at  timestamptz not null,
-- position
perch_zone       smallint not null,                -- 0=front, 1=middle, 2=back
perch_slot       smallint not null,                -- index within zone
position_seed    bigint not null,                  -- stable per-bird motion seed
-- call scheduling
next_call_at     timestamptz,
last_call_at     timestamptz,
call_seed        bigint not null,
version          bigint not null default 0         -- optimistic concurrency for the tick
```

**Deliberately absent:** any `is_deleted`, `is_active`, or `replaced_by` column. Birds are not deletable in v1. I11 is enforced partly by the absence of the mechanism.

**On monotonicity storage:** traits are stored as `real` in [0,1]. Because drift steps are far smaller than float precision at these magnitudes over a single tick, per-trait fractional remainder accumulates in `drift_carry` and is only folded into the trait when it exceeds a quantum (§5.3). Without this, a week of sub-epsilon deltas rounds to zero and drift silently never happens — a failure mode that would pass every unit test and only surface as "my birds never changed" months after launch.

### 4.3 `aviary_state`

```sql
account_id        uuid primary key references accounts,
created_at        timestamptz not null,        -- aviary age → new-bird offers
day_phase         text not null,               -- night|dawn|morning|midday|afternoon|evening
settled_until     timestamptz,                 -- non-null while settled
weather           text,                        -- null | 'rain' | 'wind'
weather_until     timestamptz,
next_weather_eligible_at timestamptz not null,
ambient_bias      jsonb not null default '{}', -- short-lived cross-aviary mood nudges
next_bird_offer_at timestamptz,                -- age-gated; null once at cap
last_tick_at      timestamptz not null,
version           bigint not null default 0
```

### 4.4 `interaction_events` (append-only)

```sql
event_id      bigserial primary key,           -- ordering is the point
account_id    uuid not null,
bird_id       uuid,                            -- null for aviary-scope events
type          text not null,                   -- see below
occurred_at   timestamptz not null,            -- client clock, validated
received_at   timestamptz not null default now(),
payload       jsonb not null default '{}',
consumed_at   timestamptz,                     -- set by the tick
client_event_id uuid not null,                 -- idempotency
unique (account_id, client_event_id)
```

Event types: `presence_window` (payload: `{start, end, ms_credited}`), `listen_in` (`{start, end, ms}`), `offer` (`{kind, accepted_by_bird, reaction}`), `settle`, `settle_undo`, `session_start`, `session_end`, `greeting_ack`.

**No `page_view`, no `click`, no `feature_used`.** The event log is a simulation input, not an analytics stream. If it ever becomes both, I9 has already been lost — the temptation to "just join the analytics warehouse to the event log" is the exact failure the PRD's privacy section is describing, and the way to prevent it is for the event log to contain nothing an analyst would want.

`bigserial` (not UUID) because the tick consumes strictly in order, and monotonic integer ordering within an account is the guarantee that makes the no-last-write-wins argument in §6.3 actually hold.

### 4.5 `notebook_entries`

```sql
entry_id       uuid primary key,
account_id     uuid not null references accounts,
written_at     timestamptz not null,
local_date     date not null,          -- in the account's tz, for "tuesday —" phrasing
prose          text not null,          -- final rendered prose; immutable
template_id    text not null,          -- for sparsity bookkeeping and QA sampling
subject_bird_ids uuid[] not null default '{}',
novelty_score  real not null           -- see 5.8; used for sparsity, never shown
```

Entries are immutable and never deleted (except at hard account deletion). `prose` is stored rendered rather than as a template + params because the notebook is a *record of what was observed* — re-rendering an old entry through a newer template version would retroactively change the user's history, which is a small version of the identity violation I11 protects against.

### 4.6 Auth, sessions, invites, visits

```sql
magic_links(token_hash pk, account_id, created_at, expires_at, consumed_at, requested_ip_hash)
sessions(session_id pk, account_id, created_at, last_seen_at, revoked_at,
         device_label, ua_family, approx_region)
invites(invite_id pk, host_account_id, visitor_email_encrypted, visitor_email_hash,
        token_hash, created_at, expires_at, revoked_at, first_used_at)
visits(visit_id pk, invite_id, started_at, ended_at)
```

`device_label` is derived (`"Safari on iPhone"`), never a raw UA string, and `approx_region` is country-level. The session list needs to be recognizable to a user hunting an unfamiliar device; it does not need to be a fingerprint archive.

Visitor emails are encrypted with the same envelope scheme as account emails. A visitor who is not a user still has PII in our system, and the fact that they never signed up doesn't lower the bar.

### 4.7 Retention

| Data | Retention |
|---|---|
| `interaction_events`, consumed | 30 days, then deleted |
| `interaction_events`, unconsumed | until consumed (alarm if > 10 min old) |
| Personality vectors, moods, birds | life of account |
| Notebook entries | life of account |
| `visits` | 1 year rolling |
| `magic_links` | 24h post-expiry |
| Aggregate telemetry | 13 months |

Consumed events are deleted at 30 days because they are the raw material of the user's relationship and we have committed that it is theirs. Keeping them "just in case we need to recompute" would also quietly create a reconstruct-the-vector-from-history capability, which the PRD explicitly forbids (`bird_engine.md`: vectors are stored, never derived). The 30-day window exists solely for incident replay and drift-calibration debugging, and even that is done on synthetic accounts by preference.

### 4.8 Account export payload

Birds (id, name, species, adopted date, **current mood label**, current perch), notebook entries in full, account settings, invite/visit log, aviary created-at.

**Personality vectors are excluded from the export.** The PRD's export section lists "current personality vectors" among exported fields, and `bird_engine.md` states the user never sees the values, "not in any tier," with no toggle. These conflict directly. **[CALL]** We resolve toward the stronger, more-argued rule: the export contains no trait numbers. A JSON file is a stats panel with extra steps — the first thing a motivated user does is diff two exports a week apart and post the deltas, which manufactures exactly the optimization surface `bird_engine.md` is written to prevent, and does it in a form we can never claw back.

What the export *does* include instead is a per-bird naturalist paragraph in the notebook voice, generated at export time, describing how the bird is now — "pip comes to the front rail more often than she used to, and greets before wren most mornings." That honors the export's stated rationale (the relationship is the user's; they can take a copy) without handing over the numbers. This decision needs a one-line sign-off from the PRD author; if overruled, the fix is a one-field schema change, so it does not block the build.

### 4.9 Settings shape

```jsonc
{
  "a11y": {
    "reduced_motion": "system" | "on" | "off",
    "captions": bool,
    "narration": bool,
    "audio_enabled": bool
  },
  "social": { "visit_notifications": false },   // default false, always
  "privacy": { "export_requested_at": null }
}
```

No `notifications` object beyond the visit toggle. No `engagement`, no `reminders`, no `digest`. The settings schema is a place where features get smuggled in; keeping it small and typed makes additions visible in code review.

---

## 5. Simulation engine

This is the heart of the product and gets the most specification. The engine is a **discrete-time, per-account, deterministic-given-inputs simulation**.

### 5.1 Tick architecture

- **Cadence:** nominal 60s per account.
- **Partitioning:** accounts hash into N partitions (start at 64). One worker leases a partition via an advisory lock with a TTL; each worker sweeps its partition once per cadence window. Partition count is a config value; scale by adding workers, not by changing N (changing N is a rebalance, and rebalancing mid-flight risks double-ticking an account — see below).
- **Tick contract:** exactly-once *per account per window*, at-least-once delivery with idempotency. Each account row carries `last_tick_at` and `version`; a tick reads state, computes, and writes with `WHERE version = $read_version`. A losing write is dropped, not retried into a second application — a double-applied drift step is a silent correctness bug of exactly the kind §6.3 exists to prevent.
- **Catch-up:** if a partition falls behind (deploy, incident), the tick does not replay N individual steps. It computes an *elapsed-time-aware* step: drift integrates over the gap (with a cap, §5.3), mood timers advance to the correct phase, weather rolls are made for at most one event in the gap. Replaying 500 skipped ticks would burn CPU proportional to the outage and can produce non-linear artifacts in the mood chain; integrating the gap is both cheaper and closer to what "the world kept running" means.
- **Sleeping accounts:** an account with no events in 7 days and no connected client ticks at 1/10th cadence (every ~10 min). Nothing observable changes — drift with zero presence input is ~zero, mood transitions are time-driven and computed from elapsed time, and the next client connection forces an immediate on-demand tick before serving a snapshot. This is a pure cost optimization and it must be invisible; it is called out here so that no one later "optimizes" it into skipping ticks for connected accounts.

### 5.2 Tick pipeline

For one account, in this order (order matters and is part of the contract):

1. **Load** aviary state + birds + unconsumed events (ordered by `event_id`).
2. **Validate events** — drop implausible ones (§5.9).
3. **Fold presence** — sum credited presence-ms; clamp to wall-clock elapsed since last tick (a client cannot credit more presence than time has passed).
4. **Compute drift deltas** per bird per trait; apply monotonically (§5.3).
5. **Advance day phase** from account timezone + wall clock.
6. **Roll weather** (§5.6).
7. **Transition moods** (§5.4).
8. **Assign perches** from mood + boldness (§5.5).
9. **Schedule calls** for the next horizon (§5.7).
10. **Consider a notebook entry** (§5.8).
11. **Consider a new-bird offer** (age-gated).
12. **Write** state with optimistic concurrency; mark events consumed; invalidate snapshot cache.

Steps 4–11 are pure functions of (previous state, folded inputs, wall clock, per-account seed). This purity is what makes the engine testable at all, and every one of them is covered by property tests in §12.

### 5.3 Drift function

Drift is a **monotone, saturating, low-pass integrator**.

For trait `t` on bird `b` over a tick of elapsed `Δ` seconds:

```
raw_input(t)   = Σ_signals w(signal, t) · magnitude(signal)
gated_input(t) = raw_input(t) · saturation(t) · novelty(b, signal)
carry(t)      += gated_input(t) · (Δ / 3600)          // per-hour normalized
if carry(t) >= QUANTUM:
    steps      = floor(carry(t) / QUANTUM)
    value(t)   = min(1.0, value(t) + steps · QUANTUM)
    carry(t)  -= steps · QUANTUM
```

with:

- `saturation(t) = (1 - value(t))^1.5` — a bird already high in a trait moves less. This gives an asymptotic approach to 1.0, which is what makes "monotonic toward expressive" not degenerate into "everyone maxes out in six months." Without saturation, a two-year user has five birds all pinned at 1.0 and indistinguishable — the drift mechanic would eat its own product.
- `novelty(b, signal)` — a per-session diminishing factor on *interaction* signals (not presence). The 20th listen-in in one session contributes far less than the first. This is the anti-grind term; it is what makes clicking not work.
- `QUANTUM = 0.001`.

**Signal weights** (relative, tuned against §5.10 targets):

| Signal | boldness | social warmth | vocal freq | plumage sat | curiosity |
|---|---|---|---|---|---|
| Presence-time (per hour, all birds) | 0.30 | 0.30 | 0.25 | 0.40 | 0.25 |
| Listen-in (per minute, focused bird) | 0.20 | 1.00 | 0.80 | 0.10 | 0.15 |
| Offer made near bird (per offer) | 0.50 | 0.10 | — | — | 0.20 |
| Offer accepted by bird (per accept) | 0.30 | 0.20 | — | 0.05 | 1.00 |
| Settle | — | — | — | — | — |

Presence is the dominant input by construction: it's the only signal that accrues continuously and applies to every bird, so an hour of just watching outweighs a burst of clicking on every trait. Plumage saturation is weighted almost entirely to presence, which is deliberate — the most visible long-term change should be the one the user cannot grind for.

**Caps.** Per-hour credited presence is capped at 60 minutes (tautologically) and per-day at **4 hours**. The daily cap exists because presence-time is honest but not unbounded — a user with the aviary on a second monitor while working genuinely satisfies all three presence conditions for eight hours, and we do not want that user's birds to drift twice as fast as a user who watches attentively for an hour. The cap is high enough that no ordinary session touches it.

**Monotonicity** is enforced structurally: the apply step is `value = min(1.0, value + non_negative)`, and `gated_input` is clamped at zero before it's added. There is no code path that subtracts. Neglect isn't modeled as negative drift; it's modeled as *the absence of input*, which is the whole point — and its visible consequence (a quieter aviary) comes from mood and call scheduling (§5.4, §5.7), never from the vector going down.

### 5.4 Mood

**States:** `wary`, `content`, `curious`, `drowsy`, `alert`, `settled`. (`settled` is added to the PRD's example set because full night and the settle gesture both need a distinct terminal-ish state — birds low on the perch, eyes closed — that isn't `drowsy`. **[CALL]**, consistent with `aviary_layout.md`'s "at full night, most birds are settled.")

Mood is a **weighted stochastic transition** evaluated per bird per tick, not a state machine with hard rules. Each candidate mood gets a score:

```
score(m) = base(m)
         + timeOfDay(m, day_phase)
         + recentInteraction(m, events_this_window)
         + ambient(m, weather, neighbor_moods)
         + personalityBias(m, bird.personality)
         + inertia(m == current_mood, time_in_mood)
```

then a softmax sample with temperature 0.6. Sampling rather than thresholding is the mechanism behind "never identical twice" at the mood layer: two birds with the same inputs on the same morning shouldn't reliably land in the same mood.

Key term definitions:

- **`inertia`** decays with `time_in_mood`, so moods are sticky for a while and then loosen. Minimum dwell time 8 minutes; without it, birds flicker between moods on adjacent ticks and read as glitchy rather than alive.
- **`timeOfDay`**: dawn → `alert` +1.2; morning → `curious`/`content`; midday → `content`; afternoon → `content`/`drowsy`; evening → `drowsy` +1.0; night → `settled` +2.0 (except the nightjar species, which gets `alert` +0.8 at night instead).
- **`personalityBias`**: `wary` is suppressed by boldness (`-1.8 · boldness`) — a high-boldness bird is *less likely to enter wary on the same input*, exactly as specified. `curious` is boosted by curiosity, `content` by social warmth.
- **`ambient` / contagion**: a bird entering `wary` writes a short-lived `ambient_bias` on the aviary that boosts `wary` for other birds for ~3 ticks, scaled by *inverse* boldness so bold birds resist the spread. Rain adds `drowsy`, suppresses `alert`. Wind adds both `alert` and `wary`, split by boldness — bold birds get alert, wary birds get warier, which is the specified "makes some birds more alert and others more wary" falling out of one term rather than being special-cased.
- **`recentInteraction`**: an accepted offer adds `content` +1.5 decaying over ~15 minutes; a listen-in adds `curious` +0.8 to the focused bird.

**Persistence:** mood is a stored column with an expiry, so it survives session end trivially. The tick advances it during absence. There is no "reset to neutral on connect" code path anywhere, and a test asserts that connecting a client does not change any bird's mood.

**Long-absence behavior:** after weeks away, moods are whatever the time-driven terms produced — typically `content`/`drowsy`, never `wary`-as-punishment. The "quieter aviary" the PRD promises comes from call scheduling: greeting frequency is a function of `social_warmth` and *recent* greeting-acknowledgment, so a long-absent user's birds greet a little less readily and call a little less often. This is decay of *expression*, not decay of *trait* — the vector is untouched (I2) and the birds warm back up within a session or two.

### 5.5 Perch assignment

```
zone_pull = 0.6·boldness + moodZoneBias(mood) + 0.15·socialWarmth·(neighbors_in_zone)
```
mapped to front/middle/back with hysteresis (a bird doesn't switch zones unless the pull crosses the boundary by a margin, and not more than once per ~4 ticks). Slot within a zone is assigned to keep birds from overlapping and to keep high-social-warmth birds adjacent. Moves emit a `motion` entry in the snapshot's schedule so the client can animate the flight rather than teleport.

There is no API to set a perch. `PATCH /birds/:id` accepts `name` and nothing else.

### 5.6 Weather

Per tick, if `now > next_weather_eligible_at`, roll: 3% chance rain (duration 4–12 min), 5% chance wind (2–6 min), else nothing; then set `next_weather_eligible_at = now + 90min` on a miss and `now + 6h` on a hit. Calibrated to land "a few times a week" for rain across a typical account's wall-clock, not per-session — weather happens whether or not anyone is watching, which means most weather is never seen, which is correct.

No thunder, no snow, no severity levels, no forecast surface, no weather icon in the top bar.

### 5.7 Call grammar and scheduling

**Split:** the server decides *that* a call happens, *which* bird, and *what motif structure*; the client synthesizes the waveform. The server emits, in the snapshot's schedule horizon (next ~90s):

```jsonc
{ "t": "2026-07-26T14:03:12.400Z", "bird_id": "...", "seed": 8823710,
  "grammar": { "motif": "rise3", "repeats": 2, "gap_ms": 340,
               "pitch_center": 0.62, "pitch_spread": 0.11,
               "tempo": 0.9, "brightness": 0.55, "intensity": 0.7 },
  "role": "spontaneous" | "greeting" | "response" | "chorus" | "alarm" }
```

**Grammar model.** Each species owns a **motif library** (4–7 motifs: `rise3`, `trill_low`, `two_note`, `chatter`, `descend`, …). A call is a *phrase*: a sequence of 1–4 motif instances with gaps, where each instance is parameterized by pitch, tempo, and timbre. Species fixes the motif *set* and the timbral envelope — this is the recognizable part. Personality and mood modulate the parameters — this is the varying part.

**Recognizability across drift** is the hard requirement (seven birds must stay individually distinguishable by ear). The mechanism:

- **Species identity** = timbre (harmonic profile, formant-like filter shape, attack/decay envelope) + motif set. Never modulated by personality or mood. This is the invariant fingerprint.
- **Individual identity** = a per-bird permanent `voice_offset` derived from `bird_id` at adoption: a fixed pitch offset (±2 semitones), a fixed tempo scalar (0.9–1.1), and a preferred motif weighting. Permanent, never drifts. Two wrens in one aviary are distinguishable because their voice offsets are.
- **Mood and drift** modulate only *rate, intensity, and pitch spread* — the things that vary in a real bird between a wary morning and a content afternoon — within bounded ranges (±15% pitch, ±25% tempo). Vocal-frequency drift changes *how often* a bird calls, not what it sounds like.

Locking timbre and per-bird offset out of the drift path is what lets a user who has known Pip for two weeks still know Pip after her vocal frequency has drifted up. If drift touched timbre, the product's own core mechanic would erode its own core affordance.

**Scheduling.** Per bird per tick, expected calls in the window ≈ `base_rate(species) · (0.4 + 1.2·vocal_freq) · moodRate(mood) · dayPhaseRate(phase) · weatherRate(weather)`. Times are jittered within the window, never on a grid. **Chorus** emerges rather than being scheduled: if two birds' scheduled calls land within 1.5s, both get `role: "chorus"` and a small intensity boost; a third bird with high vocal frequency may have a call *pulled forward* into the window with probability `0.5·vocal_freq`. That last rule is the only place we nudge emergence, and it exists because pure independence produces chorus events too rarely to read as a social system.

**Response:** a call with `social_warmth > 0.5` in a nearby bird can generate a `response` call 0.6–2.0s later, probability `0.35·social_warmth`.

**Determinism.** Every call carries a seed. The same seed on two devices produces the same waveform, so a user with a laptop and phone open hears the same aviary. This falls out of the decision/realization split and is worth a test.

### 5.8 Notebook entry generation

**Target sparsity:** ~1 entry per 2–4 days for a regularly-visited aviary. This is enforced by a hard budget, not by hoping the scoring is tuned right: **at most 1 entry per 24h, at most 3 per rolling 7 days.**

**Candidate generation.** Each tick, the engine evaluates a set of *observation detectors* against recent state. Each detector that fires produces a candidate with a `novelty_score`. Examples:

- `first_greeter_change` — a different bird greeted first than usual this week.
- `long_quiet` — an unusually long gap with no calls.
- `sustained_preen` — a bird stayed in one idle behavior a long while.
- `weather_reaction` — bird behavior during/after rain.
- `perch_shift` — a bird has been sitting further forward than its recent baseline.
- `chorus_event` — an unusually dense chorus.
- `first_of_season_ish` — first rain in a while; first time a bird used a rare motif.
- `drift_milestone_implicit` — a bird's behavior has crossed a threshold the user might notice. **Never phrased as a milestone.** "pip is on the front rail again this morning; she used to keep to the middle" is allowed. "pip's boldness increased" is not, and neither is anything that reads as an achievement.

**Selection.** Highest novelty wins, with a penalty for any detector used in the last 14 days (so the notebook doesn't repeat itself) and a hard suppression of any detector used in the last 3 days.

**Prose generation.** **[CALL]** No LLM at runtime. Entries are rendered from a large, hand-written, structured template corpus: per detector, 12–20 phrasings, each with slots (bird name, perch, weather, time-of-day, a small adjective pool keyed to mood). Combinatorially this yields thousands of distinct entries, is deterministic, costs nothing, has zero latency, cannot hallucinate a bird that doesn't exist, and — most importantly — cannot drift out of voice. A runtime LLM would produce better prose on its best day and off-voice or factually wrong prose on its worst, in the one surface where the PRD says the voice is most concentrated and most visible. It would also create a per-bird-state egress to a third party, straining the privacy commitment. The corpus is written by someone who can actually write; this is a content-authoring task on the critical path, not an engineering task, and it should be staffed in week 2.

**Voice enforcement:** the corpus is linted in CI — lowercase-first, no exclamation marks, no second person ("you", "your"), no banned lexicon (§12.4), present tense, no numerals for counts of user behavior. The date prefix ("tuesday — ") uses the account's local weekday.

**Hard rule:** detectors may only observe *the aviary*. There is no detector with access to session counts, visit frequency, or day-streaks. The line the PRD draws — observations of the aviary, never of the user — is enforced by the detector interface simply not receiving that data.

### 5.9 Event validation

Clients are not trusted with presence. Server-side checks before an event is folded:

- `occurred_at` within [now − 10 min, now + 30s]; else drop.
- Presence windows: clamp `ms_credited` to the wall-clock span; reject overlapping windows from the same session; cap per-session and per-day (§5.3).
- Multiple concurrent sessions on one account: presence is **unioned, not summed**. Two devices open and attended for the same hour is one hour of presence. Summing would let a user double their drift by opening a second tab, which is a grind mechanic we'd be shipping by accident.
- Offer cooldowns are authoritative server-side (Redis + a check in the tick). A client that fires offers faster than cooldown gets them dropped silently — no error surface; the offer simply doesn't produce a reaction.
- Visitor-session events are dropped at ingest, before the log (I10).

### 5.10 Calibration targets and how we hit them

The PRD gives two numbers: **measurable drift in instruments after ~1 week**, **visible drift to the user after ~3 weeks**. We operationalize:

- "Regular visits" = 5 sessions/week × 12 minutes = ~1 hour presence/week.
- **Week 1 (~1h presence):** aggregate trait movement of **0.02–0.04** — detectable by the harness, ~2–4% of range, below perceptual threshold.
- **Week 3 (~3h presence):** cumulative **0.06–0.12** on the most-driven traits. Crossing this must produce at least one *behaviorally visible* change: a perch-zone boundary crossing, a noticeable greeting-frequency increase, or a visible plumage step.

**Perceptual quantization.** Plumage saturation renders in **6 discrete visual steps**, not continuously. Continuous saturation drift is invisible — no user perceives a 3% saturation change against a day/night-varying palette, so the trait would be doing nothing for the user regardless of how well the math worked. Discrete steps mean a user occasionally notices that Pip *is* brighter than she was, which is the entire point. The steps are placed at trait values {0.15, 0.30, 0.45, 0.62, 0.80} so that a typical user crosses their first around week 3.

**Calibration harness** (built in week 3, before the engine is tuned): a simulation runner that executes the tick against synthetic user-behavior profiles (daily-attentive, weekly-visitor, binge-then-absent, background-tab-only, grinder) at 1000× wall clock, and reports trait trajectories, perch-crossing dates, and visual-step dates. Tuning happens against this harness, not against production. Its assertions become CI regression tests: a code change that moves week-3 drift outside the target band fails the build. This harness is the single most valuable piece of internal tooling in the project — without it, the drift constants are guesses that we find out about three weeks after launch, one cohort at a time.

---

## 6. API surface

REST + JSON over HTTPS/2. Session cookie: `HttpOnly`, `Secure`, `SameSite=Lax`. CSRF via double-submit token on mutations.

### 6.1 Auth

```
POST /auth/request-link   { email }              → 202 (always; never reveals existence)
GET  /auth/consume?token=…                       → 302 to /aviary, sets session cookie
POST /auth/signout                               → 204
GET  /account/sessions                           → list (device_label, last_seen, region, current)
DELETE /account/sessions/:id                     → 204
```

`request-link` always returns 202 regardless of whether the account exists — an account-existence oracle on a product with no passwords is still an email-enumeration vector. Rate limits: 5/hour per email, 20/hour per IP.

### 6.2 Reading state

```
GET /aviary/snapshot?since=<snapshot_id>
```

Response:

```jsonc
{
  "snapshot_id": "s_18f2c9",
  "server_time": "2026-07-26T14:02:58.120Z",
  "aviary": {
    "day_phase": "afternoon", "phase_progress": 0.42,
    "weather": null, "settled": false, "bird_cap": 7
  },
  "birds": [{
    "bird_id": "b_9f3a", "name": "pip", "species": "warbler",
    "mood": "curious",
    "plumage_step": 3,                       // 0..5, NOT the trait value
    "perch": { "zone": 0, "slot": 1 },
    "position_seed": 88123,
    "expressiveness": "high",                // coarse render hint, see below
    "idle": { "behavior": "scan", "phase": 0.31 }
  }],
  "schedule": [ /* calls + motions, next ~90s, each seeded */ ],
  "greeting": { "bird_id": "b_9f3a", "form": "call_and_step",
                "at": "…", "seed": 4471, "absence_bucket": "medium" },
  "next_poll_after_ms": 45000
}
```

**On `expressiveness`:** the client needs *some* personality signal to shape idle motion amplitude and greeting animation. Shipping the raw traits would violate I3 outright — "the user never sees the numbers" is not satisfied by putting them in a JSON payload the devtools network tab displays on request. So the server sends a **coarse 3-bucket render hint** (`low|medium|high`), derived from a blend of boldness and vocal frequency, plus the already-quantized `plumage_step`. Buckets are wide enough that no meaningful reverse-engineering of the vector is possible, and there is a contract test asserting the snapshot schema contains no field matching `/bold|warmth|vocal|curio|satur|trait|personality|drift/`.

**Snapshot delivery.** Snapshots are computed at tick time and cached in Redis keyed by `(account_id, tick_version)`. A `GET` is a cache read plus a re-projection of the schedule horizon against current time. Cold read forces an on-demand tick first. Payload target: **< 6KB gzipped** for seven birds.

**Polling, not sockets. [CALL]** The PRD says "low-frequency keepalive." At a 60s tick and kilobyte payloads, WebSockets would cost us a stateful connection tier, sticky-session complexity, reconnect/backoff logic, and a load-balancer configuration — to deliver one small message per minute. Polling with `Cache-Control` and conditional requests is dramatically simpler and cheaper, and the client's interpolation + 90s schedule horizon means it renders ahead smoothly regardless. Client pulls: on load (inlined at edge), on `visibilitychange → visible`, after a render-frame gap > 5s (laptop resume), and every 45s while visible. Jittered ±20% to avoid a thundering herd at the top of each minute.

### 6.3 Writing events

```
POST /aviary/events
{ "events": [ { "client_event_id": "uuid", "type": "presence_window",
                "occurred_at": "…", "payload": { … } } ] }
→ 202 { "accepted": ["uuid", …], "rejected": [{ "id": "uuid", "reason": "stale" }] }
```

Batched (up to 20), idempotent by `client_event_id`, sent on a 30s flush timer, on `visibilitychange → hidden`, and via `navigator.sendBeacon` on `pagehide`. Events queue in IndexedDB when offline and flush on reconnect (with staleness dropping applied server-side).

**There is no endpoint that writes personality state.** Not `PUT /birds/:id/personality`, not an admin one, not a debug one. The Go type for a personality vector has no exported setter outside the `sim` package, and the DB role used by `aviary-api` has `SELECT` but not `UPDATE` on the five `p_*` columns (enforced via a view + column-level grants). This is I1 implemented three ways, because the argument in `accounts_sync.md` is that the failure is *silent* — and a silent failure needs redundant, mechanical prevention, not a code-review norm.

### 6.4 Mutations the client may make

```
PATCH /birds/:bird_id            { name }                    → 200
POST  /aviary/offer              { kind }                    → 202 (event)
POST  /aviary/settle             { }                         → 202
POST  /aviary/settle/undo        { }                         → 202
GET   /notebook?before=&limit=   → cursor-paginated entries
GET   /account                   → email (masked), tz, settings, aviary age
PATCH /account                   { timezone?, settings? }    → 200
POST  /account/email-change      { new_email }               → 202
POST  /account/export            { }                         → 202 (emailed link)
POST  /account/delete            { }                         → 202 (soft)
POST  /account/undelete          { }                         → 200
POST  /aviary/adopt              { name }                    → 201 (only when offered)
```

Offer and settle return `202` and produce events; their *effects* appear in the next snapshot, plus an immediate optimistic local realization on the client (§7.6). The client does not wait for the server to show the seed appearing — a 200ms round-trip between "user offers a seed" and "anything happens" reads as lag in a product whose whole claim is immediacy of response.

### 6.5 Visits

**Host side:**
```
POST   /invites          { visitor_email }   → 201 { invite_id, expires_at }
GET    /invites                              → outstanding invites
DELETE /invites/:id                          → 204 (immediate revocation)
GET    /visits                               → visit log (email, date, approx duration)
```

**Visitor side:**
```
GET /visit/:token          → HTML shell, visitor mode
GET /visit/:token/snapshot → same snapshot schema, minus greeting, minus notebook
```

Visitor sessions are a distinct session type with no cookie-bearing account, no event endpoint (the client doesn't even instantiate the event queue in visitor mode — the module is code-split out), and no write capability. Every revoked/expired/unknown token returns the same matter-of-fact surface, and the visitor client checks on each snapshot pull; a revoked visit terminates within one poll interval (≤45s), which satisfies "immediately" at human timescale.

The visitor snapshot **omits `greeting`** — no bird notices the visitor, because the birds don't know the visitor. This isn't in the PRD explicitly but follows directly from "the visit is observation, not co-presence" and from greeting being a drift-relevant, host-specific event. **[CALL]**

The visit log is reachable only by navigating to settings. There is no badge, no unread count, no highlight on new visits.

### 6.6 Errors

All error responses carry matter-of-fact copy from a central catalog, capitalized normally, direct, no naturalist phrasing. The catalog is a single file so the register is reviewable in one place, and it has a lint rule inverse to the notebook's: error copy must *not* be lowercase-first and must *not* use naturalist vocabulary.

---

## 7. Frontend rendering pipeline

### 7.1 Renderer choice **[CALL]**

**WebGL2 via a thin custom layer, with a Canvas2D fallback.** Seven birds with per-feather-group articulation, layered parallax backgrounds, ambient particles, and continuous palette interpolation, at 60fps on a five-year-old laptop, with no memory growth over 30 minutes. Canvas2D can *probably* hold that; WebGL2 holds it with margin, and the palette/day-night work (a continuous LUT blend across the whole scene) is nearly free in a fragment shader and expensive on the CPU. We write ~600 lines of GL rather than take a 120KB engine dependency against a 2MB budget. Canvas2D fallback exists for WebGL-unavailable contexts and drops ambient particles and parallax; it is not a designed alternate surface (unlike reduced-motion), just a compatibility path.

DOM/SVG for the scene is rejected: seven birds × multiple articulated parts × 60fps means thousands of style recalculations per second, and Safari's compositor will not hold it.

### 7.2 Scene composition

Five draw layers, back to front:

1. **Sky** — a full-screen gradient quad driven by the day-phase LUT.
2. **Far foliage** — soft shapes, parallax factor 0.15, gentle wind displacement in the vertex shader.
3. **Bird plane** — perches (front/middle/back), birds, still-pool offer surface.
4. **Ambient ornaments** — leaves, feathers, drifting particles, parallax 1.0–1.3.
5. **Near foreground** — an occasional branch, parallax 1.4, heavily blurred.

**Responsive layout.** The scene is laid out in a virtual coordinate space with a **fixed vertical extent** and a **variable horizontal extent** clamped to [1.0, 2.4] aspect. Perch anchor points are defined as fractions of the horizontal extent, so a wide viewport spreads them and a narrow one compresses them. Bird sprites scale with vertical extent only, so a bird is never smaller on mobile than legibility allows. There is a per-bird `visibleBounds` assertion each frame in dev builds: if any bird's bounding box leaves the viewport, it throws. "Never crop a bird out" is a testable property and we test it (§12.3) rather than trusting the layout math.

No panning, no zoom, no scroll on the aviary route. `overscroll-behavior: none`, no wheel handler, no pinch handler on the scene.

### 7.3 Bird rendering and idle micro-motion

Each bird is a small skeleton (body, head, beak, tail, two wings, feet) driven by **layered procedural oscillators**, not keyframe animation clips:

- **Breath** — 0.2–0.4Hz body scale, always on, amplitude by mood.
- **Weight shift** — a slow aperiodic drift in stance, driven by 1D value noise seeded per bird.
- **Head scan** — saccade-like discrete head reorientations at Poisson intervals; rate by mood (wary highest, drowsy lowest).
- **Preen** — an episodic behavior that occupies head+wing for 3–12s.
- **Blink** — Poisson, suppressed in `settled`.
- **Fluff** — feather-spread parameter, high in `drowsy`/cold-morning, low in `alert`.
- **Tilt-toward-sound** — a reactive term triggered by another bird's call within earshot, gated by the curiosity hint.

Composition rather than clips is what makes "never identical twice" true at the visual layer: the same bird in the same mood produces a different-looking minute every minute, because the oscillators are aperiodic and seeded on a continuous clock rather than looping. A clip library would be visibly a clip library within one session — the human eye catches an 8-second animation loop almost immediately, and once caught, it cannot be un-caught.

**Mood shapes idle**, per the PRD: `wary` → back perch, high scan rate, low preen, upright stance; `content` → frequent preen, mid stance, slow scan; `curious` → high tilt-toward-sound, forward lean, frequent short hops in place; `drowsy` → low stance, high fluff, rare slow blinks, minimal scan; `alert` → upright, still-but-tense, fast infrequent saccades; `settled` → lowest stance, eyes closed, breath only.

There is no mood label, tooltip, icon, or aria-hint that names the mood on the visual surface. The user reads it from the motion. (Narration describes the *appearance*, not the state name — §9.1.)

### 7.4 Interpolation and the schedule horizon

`SceneModel` holds snapshot N and the schedule horizon. Position changes interpolate along a Bezier flight path with species-appropriate wingbeat over 0.8–1.6s. Scheduled calls fire on the client's clock, offset-corrected against `server_time` at each snapshot. If a snapshot arrives late, the client extends the last one — birds keep breathing, scanning, and preening, because none of that requires the server. **The client never freezes waiting for state.** A hitch in the network must not become a hitch in the aviary; that's the difference between a place and a page.

If a snapshot reveals a discontinuity (a bird moved while the client was catching up), the client *fills the gap plausibly*: it plays the flight now rather than teleporting. Slightly-late is invisible; teleporting is not.

### 7.5 Cold load — the first frame

The critical path (§8.2) is engineered so that **the first painted frame already contains birds mid-action.** Concretely:

1. The edge returns HTML with the snapshot inlined in a `<script type="application/json">` tag and the critical JS (renderer + bird geometry + scene) inlined or preloaded.
2. On parse, the client seeds `SceneModel` with the snapshot, sets the animation clock to `server_time`, and **advances every procedural oscillator to a non-zero phase derived from `position_seed` and elapsed time** before the first draw.
3. First draw shows birds mid-breath, mid-preen, one leaf already partway across frame, palette at the correct time of day.

Step 2 is the whole trick and must not be dropped in optimization: if oscillators start at phase 0, every bird begins in an identical neutral pose and the scene visibly "starts," which is precisely the entry animation the PRD forbids. There is a visual regression test on the first frame (§12.3).

**Slow-load state:** a quiet field — sky gradient at the correct local time-of-day (computable from the client clock with zero network), plus one or two faint ambient motion cues. No spinner, no progress bar, no skeleton shimmer, no logo. The component library has no spinner primitive, and adding one fails the bundle-composition check.

**Empty-aviary state** (post-adoption, pre-first-bird): the same quiet field. The first bird enters with a soft fly-in from offscreen to its starting perch, the second a beat later. This is the *only* fly-in-from-nothing in the product, and it is legitimate because the birds genuinely are arriving.

### 7.6 Interaction realization

- **Listen-in:** click/tap/keyboard-focus a bird → the audio ramp begins immediately client-side (§8.4); the visual treatment is a subtle deepening of focus (a slight vignette on the rest of the scene, ~4% — not a highlight ring, not a spotlight, not a border). An event is sent. Disengage on: re-click, click empty space, focus another bird, `Escape`, focus leaving the scene.
- **Offer:** top-bar affordance opens a small sheet with three choices in naturalist copy ("a seed", "a song", "a still pool"). On choose: the sheet closes, the object appears in the scene immediately, and bird reactions begin locally using the client's mood knowledge, then reconcile with the server's authoritative reaction on the next snapshot. Cooldown is reflected by the option quietly not being offered again for that bird — not by a disabled button with a timer, which would be a countdown UI in a product with no counters.
- **Settle:** the evening LUT blends over ~6s, call rate drops, birds bias toward `settled`. A 5s undo window: any click in the aviary reverses the blend. The undo affordance is not labeled or announced — it's a mercy, not a feature; announcing it would be an announcement.

### 7.7 Reduced-motion mode

A **second Renderer implementation** against the same `SceneModel`, selected by `prefers-reduced-motion` or the explicit setting.

- Continuous oscillators → a **pose set** per bird per mood (4–6 poses), cross-fading over 1.2–2.0s at behavior-change boundaries. A preening bird cross-fades through preen poses; it does not animate.
- Flight → a 1.5s cross-fade between perch positions with a faint intermediate ghost pose, not an animated path.
- Ambient leaf/feather particles → removed entirely.
- Parallax → removed (layers static).
- Day/night palette transitions → retained, slowed (the LUT blend runs at 0.5× rate).
- Calls, drift, mood, notebook, greeting: **unchanged**. Full quality.

The pose sets are drawn by the visual designer as a deliverable, not derived by freezing the animation at arbitrary frames — a frozen frame from a procedural rig looks like a bug, and the PRD is explicit that this mode must have its own charm rather than looking broken. Reduced-motion is reviewed in design review as its own surface, with its own sign-off, in the same milestone as the full renderer. It does not ship a week later.

### 7.8 Top bar

Four icons: account/settings, accessibility settings, notebook, offer. Fades to ~8% opacity after 4s of cursor stillness; returns instantly on pointer move, key press, or focus. Never fades while a menu is open, while focus is inside it, or when the user is keyboard-navigating (a fading control the keyboard user can't see is an accessibility bug wearing an aesthetic).

The top bar is the only chrome. Nothing renders inside the aviary scene: no hover tooltips on birds, no name labels, no mood icons, no focus badges, no offer targets. Bird names appear only in the notebook, in narration, in captions, and in settings.

---

## 8. Audio pipeline

### 8.1 Synthesis architecture

WebAudio, all synthesis client-side, zero audio files (I5).

```
per-call voice ──► [oscillator bank + noise] ──► [formant filter chain]
                                  │                        │
                             [envelope]              [species timbre]
                                  └──────► [bird gain] ──► [pan] ──┐
                                                                    ▼
   ambient bed (filtered noise, wind, rain) ─────────────► [master bus] ──► out
                                                                    ▲
                              listen-in mix automation ─────────────┘
```

**Voice model per call:** 2–3 detuned oscillators (sine/triangle) with a per-motif pitch contour applied as `setValueCurveAtTime`, a shaped noise component for breathiness, a 2–3 band biquad chain approximating species formants, and an amplitude envelope per motif instance. A phrase is a sequence of these with gaps. This is small, cheap, and — critically — *parameterized*, so variation is free.

**Cost control:** a **pre-allocated voice pool of 12** `AudioBufferSourceNode`-free graph slices (oscillators are created per call — they're single-use by spec — but filters, gains, and panners are pooled and reused). At most 7 birds × ~1.5 concurrent phrases fits comfortably. A hard cap of 12 simultaneous voices; a 13th request steals the quietest. Per-call node allocation without pooling is the standard WebAudio memory leak and would fail the 30-minute no-growth test on day one.

**Recognizability** is implemented exactly as §5.7 specifies: timbre + motif set = species (never modulated); `voice_offset` = individual (permanent); mood/drift modulate rate/intensity/spread only, within ±15% pitch and ±25% tempo.

### 8.2 Critical-path note (belongs here and in perf)

The audio graph is constructed lazily on first user gesture (browser autoplay policy requires it) but the *scene* never waits for audio. First bird visible does not depend on `AudioContext` construction. If the user never gestures, the aviary plays silently and captions turn on automatically after ~10s of a suspended context — the same treatment as the WebAudio-unavailable path, because from the user's side it's the same experience.

### 8.3 Chorus mixing

Simultaneous calls mix on a shared bus with:
- **Per-voice stereo pan** from perch position (front = center + wide, back = narrower + slightly attenuated).
- **A shared reverb** (a short procedurally-generated impulse, ~0.9s, generated once at init — not a downloaded IR file) so the birds sound like they're in one place rather than in seven separate mixes. This is a surprisingly large part of "chorus, not stacked tracks."
- **Light spectral ducking**: overlapping voices in the same band get 1.5–3dB of mutual attenuation, so two simultaneous calls stay individually audible rather than summing to mush.
- **No compressor on the master bus** beyond a gentle limiter. Compression pumps, and pumping is an audible signature of "mixed audio" rather than "a place."

### 8.4 Listen-in mix

On engage: focused bird's gain ramps `+4dB` over **1.8s** (exponential ramp); all other birds ramp to `-9dB` over 2.4s; ambient bed drops `-4dB`. On disengage, the reverse over the same durations. Others **never go below -9dB and never reach silence** — the PRD is explicit that this is a rebalance, not a mute, and the floor is a constant with a test asserting no gain node in the graph reaches zero during listen-in.

The asymmetric ramp lengths (rise faster than others fall) make the engage feel like leaning in rather than like a crossfader. A hard cut, or matched 100ms ramps, produces the "soloable tracks" feel the PRD rejects; the slowness is the affordance.

Also on engage: the focused bird's call *rate* increases modestly on the next snapshot (the server sees the listen-in event), so listening in eventually rewards itself with more to listen to.

### 8.5 Ambient bed

Filtered noise (wind), a slow low-level tonal bed keyed to day phase, and rain when weather is active (procedural: filtered noise with a slow amplitude contour, plus sparse impulse "drops"). Very quiet — the bed's job is to prevent the silence between calls from reading as "audio is off."

### 8.6 Fallback

No WebAudio (unsupported, blocked, hardware failure, context creation throws, or context stays suspended): the aviary runs in silence and **captions turn on by default**. No recorded-audio path exists in the codebase. An `audio_unavailable` counter goes to aggregate telemetry with a reason dimension, no account dimension.

---

## 9. Accessibility

Accessibility is a first-class surface, planned into every milestone, with its own design sign-off. It ships with v1 (§13) — there is no "a11y hardening" milestone after launch, because that phrasing is how a11y slips.

### 9.1 Screen-reader narration

**Structure.** A visually-hidden `aria-live="polite"` region containing the current narration paragraph, plus an `aria-live="assertive"`-adjacent (still polite, but priority-queued internally) path for user-initiated events. `NarrationComposer` reads `SceneModel` — the same state the pixels come from.

**Cadence.** One prose update per **30–60s** at idle (jittered). Priority events (return-greeting, offer reaction, settle, a bird arriving) narrate promptly and reset the idle timer. A hard floor of 8s between any two utterances, and a queue depth of 1 — a new utterance replaces a pending one rather than stacking. Overwhelming the SR queue is how a narration feature gets muted by the user, which converts a designed surface into an annoyance.

**Voice.** Naturalist prose from the same corpus discipline as the notebook (§5.8) — a separate template corpus, same linter, same rules. Lowercase, present tense, specific, observational.

> a small grey bird is perched on the front rail, calling softly. another sits further back with feathers fluffed. it is morning in the aviary; the light is gentle.

Never: `"Pip is at perch 2"`, `"Wren mood: content"`, `"3 birds present"`, `"Bird 1 of 2"`.

**Describing mood without naming it.** The composer maps mood → *appearance* phrases, exactly as the visual surface maps mood → motion. `drowsy` becomes "sitting low with feathers fluffed," not "drowsy." A screen-reader user infers mood the same way a sighted user does — from described behavior — which is the point of the whole stance. This is the single most important implementation detail in the a11y section and the easiest to lose to a well-meaning "but it'd be clearer to just say the mood."

**Naming.** Narration uses bird names once the user has named them ("pip is on the front rail"), falling back to species-descriptive phrasing for variety ("a small grey bird") so it doesn't read as a roll call.

**Structure for navigation.** Beyond the live region, the aviary exposes a semantic structure so an SR user can explore rather than only listen: a `role="group"` per bird with an accessible name that is a short naturalist phrase, focusable and arrow-navigable. Focusing a bird narrates that bird specifically. This gives SR users an *exploratory* affordance sighted users get for free by looking at whichever bird they want.

### 9.2 Captions

Opt-in (and automatic when audio is unavailable). Generated at synthesis time from the actual grammar parameters used, so the caption always matches what played:

- `rise3 × 1, narrow spread` → "a soft three-note rise"
- `trill_low × 2, gap 340ms` → "a low trill, paused, low trill again"
- `two_note × 1, high intensity, back perch` → "a single sharp call from the back perch"

Composed from a parameter→phrase mapping (contour word × repetition word × intensity adjective × location clause), not a fixed string table. Rendered as small text near the calling bird, fading with the call. Same naturalist voice, same linter. When multiple birds call at once, captions stack vertically with the most recent lowest, max 3 visible, each fading after ~2.5s.

Captions are the one text that renders inside the aviary scene. That's an intentional exception to "no UI chrome in the aviary": a caption is a transcription of the aviary's own sound, not chrome about the aviary. It carries no affordance, no border, no background chip — just text with a soft shadow for contrast.

### 9.3 Keyboard navigation

`Tab` cycles the four top-bar items, then enters the aviary and focuses the first bird. Arrow keys move focus among birds (left/right by horizontal position, up/down by perch zone). `Enter`/`Space` toggles listen-in on the focused bird. `Escape` exits listen-in; a second `Escape` releases focus from the scene. The offer sheet is a proper focus-trapped dialog with `Escape` to close. Settle is a top-bar button. All shortcuts are discoverable from the accessibility settings panel (matter-of-fact voice).

**Focus indicator:** a two-tone outline (a light inner stroke + a dark outer stroke) so it reads against both the bright midday sky and the dark night scene, at ≥3:1 against both. Never `outline: none`. Focus is visible in reduced-motion mode identically, and its appearance does not animate.

### 9.4 Contrast and visual settings

All user copy (top bar, settings, errors, captions, notebook, visible narration) passes WCAG AA (4.5:1 body, 3:1 large). Captions over a variable scene use a soft text shadow plus a per-frame contrast check in dev builds that samples the backdrop behind caption text and warns on failure — the aviary's palette changes all day, so a design-time contrast check is insufficient for this one surface.

The accessibility settings panel (matter-of-fact voice) offers: reduced motion (system/on/off), captions, narration, audio on/off. Four toggles. Nothing else.

### 9.5 Accessibility acceptance

A11y is not "done" when axe passes. Acceptance for v1 includes: a moderated session with at least two screen-reader users (NVDA/Windows and VoiceOver/macOS+iOS) and one reduced-motion user with vestibular sensitivity, where the acceptance question is not "could you use it" but **"did it feel like a place?"** If the answer is "it was usable," we have shipped the degraded variant the PRD is explicitly refusing, and the milestone is not complete.

---

## 10. Performance and observability

### 10.1 Budgets and enforcement

| Budget | Target | Enforcement |
|---|---|---|
| Initial JS (gzipped, first paint) | ≤ 2MB — **internal target 400KB** | `size-limit` in CI, hard fail |
| Time to first bird (mid-tier mobile, 4G) | < 500ms — **internal target 350ms** | Lighthouse CI + WebPageTest on throttled Moto-G-class device, per PR on main |
| Idle motion FPS (5-yr-old laptop) | 60fps sustained 30 min | Nightly headless run with frame-timing capture; fail on >1% frames over 16.7ms |
| Memory growth over 30 min | 0 | Nightly CDP heap-snapshot diff; fail on >2MB retained growth |
| Snapshot payload | < 6KB gz | Contract test |
| Tick latency | p99 < 5s (alarm), p50 < 200ms | Production alarm + load test in CI |

The internal targets are deliberately far below the stated caps. A budget you're at 95% of is a budget you'll breach in the next sprint, and the 2MB figure in the PRD is a ceiling derived from the 500ms goal, not a target to grow into.

### 10.2 Hitting time-to-first-bird

The critical path, in order:

1. **Edge-rendered HTML** with (a) inline critical CSS, (b) the **bootstrap snapshot inlined as JSON**, (c) inline JS for the renderer bootstrap. Single round trip after DNS/TLS.
2. **Bird geometry is procedural, not asset-loaded.** Silhouettes are parametric curves per species evaluated at init (a few KB of coefficients), so there is no image request in the critical path at all.
3. **First draw happens before hydration of any chrome.** The top bar, notebook, settings, offer sheet, and visit flow are all code-split and loaded after first paint.
4. **Fonts are not in the critical path.** The scene has no text. Chrome text uses a system font stack; if a webfont is used at all, it's `font-display: optional`.
5. **Audio init is off the critical path** (§8.2).

Snapshot inlining requires the edge to fetch it — so `aviary-api` serves a snapshot-only endpoint from a Redis read at the same region, and the edge caches per-account for the remainder of the current tick window with a private cache key. Cold-cache case is one origin round trip, still one RTT before HTML flush.

**Measurement:** "time to first bird" is a custom mark emitted in the same frame as the first draw containing a bird, reported through RUM. Not FCP, not LCP — those measure the sky gradient. Measuring the wrong proxy is how a team hits the number and misses the goal.

### 10.3 Runtime performance

- Single `requestAnimationFrame` loop; all systems tick from one clock. No per-bird timers, no `setInterval` anywhere in the render path.
- Zero allocation in the hot loop: preallocated typed arrays for transforms, object pools for particles, no closures created per frame, no array literals, no string concatenation. Enforced by a dev-mode allocation counter and by the nightly heap test.
- Ambient particles capped at 12 concurrent, pooled.
- On `visibilitychange → hidden`, the RAF loop stops entirely, the audio context suspends, and a `session_pause` marker is recorded. On visible, pull a snapshot, resume, and advance oscillator phases by the elapsed time (so the aviary looks like it kept going, because it did).
- Notebook list is virtualized with explicit reference release on scroll-out.

### 10.4 Observability — and the boundary

**Collected (aggregate only, no account dimension):** request counts/latencies/error rates by endpoint; tick latency and lag distributions; tick failure counts; page-load and time-to-first-bird histograms; frame-timing percentiles; audio-context error counts by reason; WebGL-unavailable counts; anonymized session-duration histogram; email-delivery success rates; bundle size over time.

**Never collected, at metric-definition level:** anything per-account or per-bird — trait values, moods, drift rates, presence-time, offer counts, listen-in durations, notebook entry counts, bird names, species distribution *per account*. No "average drift across all accounts" dashboard; the PRD names that exact example as forbidden and it would be the first thing someone builds.

**Architectural enforcement of I9:**
- Telemetry runs in a separate VPC with no route to `sim-db` (security-group deny, not just absence of a connection string).
- The metrics client library **has no API that accepts an account ID** — the function signature makes it impossible, so violations are compile errors rather than review catches.
- Log scrubbing at the collector drops any field matching UUID patterns in metric dimensions.
- A quarterly review of every metric definition against the allowlist, with the exclusion list as the checklist.

**Alarms:** tick p99 > 5s (PRD-specified); tick lag > 5 min for any partition; unconsumed events older than 10 min; magic-link delivery failure rate > 2%; time-to-first-bird p75 > 500ms; error rate > 1%; drift-monotonicity violation (any occurrence — page immediately, it means I2 is broken).

**Deliberately not measured:** DAU, retention curves, session frequency per user, engagement funnels. Not because they're technically hard, but because a team that has a retention dashboard will eventually ship something to move it, and the thing that moves it is a streak counter. The absence of the measurement is part of the defense of the product. We measure whether the system is healthy; we do not measure whether users are coming back often enough.

---

## 11. Sync model (consolidated)

Restated compactly because it's the property most likely to be violated by a well-meaning optimization:

1. **One canonical record.** Personality, mood, perch, and schedule live in `sim-db`, per account. There is no client copy that is authoritative for anything.
2. **The tick is the only writer** of personality (I1), enforced by column grants, package boundaries, and CI.
3. **Clients write events, not state.** Append-only, ordered by `bigserial`, idempotent by `client_event_id`.
4. **Deltas, never absolutes.** The tick computes `Δ` from the event log and applies it. No message in the system carries a personality value in either direction.
5. **Therefore last-write-wins is unreachable.** Two devices producing events concurrently produce two log entries that both get folded in order. Neither overwrites the other, because neither writes state. The morning-laptop / lunch-phone scenario from the PRD cannot occur — there is no read-modify-write cycle for a client to lose.
6. **Multi-device is not a feature.** Both devices poll the same snapshot cache and render the same world. There is no reconciliation code because there is nothing to reconcile.
7. **Concurrent presence unions, not sums** (§5.9).
8. **Optimistic concurrency on the tick's own write** prevents a double-tick from double-applying drift.
9. **Real conflicts are auth-shaped, not state-shaped** — magic-link replay, expired session, revoked session, server outage. These get matter-of-fact error surfaces (§6.6), and they are the *only* conflict surfaces in the product.

**Failure behavior.** If `sim-db` is unavailable, clients keep rendering from the last snapshot and queue events locally (IndexedDB); the aviary looks fine for minutes. If the event log is unavailable, clients queue and retry with backoff; nothing is lost unless the outage exceeds the client's local retention (24h), and even then only presence granularity degrades. If `sim-tick` is down, the world pauses — moods and drift stall — and the client keeps rendering plausibly from the last snapshot. Tick downtime is the only outage the user can eventually feel, which is why the tick is the most operationally-protected component and why its lag alarm is aggressive.

---

## 12. Testing and enforcement

### 12.1 Engine correctness

- **Property tests** on drift: monotonicity (∀ trait, ∀ input sequence: `after ≥ before`); saturation (values stay in [0,1]); zero-input produces zero change; determinism (same seed + inputs → identical output).
- **Property tests** on mood: dwell-time minimum respected; no transition to `wary` from `personalityBias` alone at high boldness; contagion decays to zero.
- **Idempotency test:** replaying the same event batch twice produces identical state.
- **Ordering test:** interleaved events from two sessions fold deterministically by `event_id`.
- **Calibration regression** (§5.10): synthetic profiles must land in the target bands. This test fails the build if drift constants are changed without a corresponding intentional band update.
- **Long-run simulation:** two simulated years of the daily-attentive profile must not saturate all traits to 1.0, must not crash, and must produce a notebook with no repeated entry within 14 days.

### 12.2 Invariant enforcement (the I-list)

| Invariant | Mechanism |
|---|---|
| I1 | Column-level DB grants; `sim` package owns the only mutator; CI grep for `p_boldness` etc. outside `sim/`; integration test asserting API cannot mutate |
| I2 | Property test + runtime assertion + production alarm |
| I3 | Snapshot-schema contract test with a forbidden-field-name regex; export-schema test |
| I4 | Client unit tests on the presence state machine (all 8 combinations of the three signals); server clamp tests |
| I5 | Bundle-composition CI check: any asset with an audio MIME type fails the build |
| I6 | Component allowlist lint (no `Toast`, `Banner`, `Modal`-welcome, `Badge` primitives exist in the design system); a11y-tree snapshot on session start asserts no live-region announcement other than narration |
| I7 | Copy lint over all user-facing strings against the banned lexicon (§12.4); export-schema test |
| I8 | Type-level: `AccountID` is a distinct type from `Email`; the email type has no `String()` method that logs; log-scrubber test asserting no `@` reaches logs |
| I9 | Network policy test in infra CI; metrics-client API shape (no account-ID parameter exists); pipeline schema allowlist |
| I10 | Ingest test: visitor-session events produce zero rows |
| I11 | Migration checklist requiring explicit sign-off on any change touching `birds.bird_id`; no `DELETE` grant on `birds` for the API role |
| I12 | Loading-component allowlist; first-frame visual regression test |

### 12.3 Visual and behavioral

- **First-frame test:** cold-load in a headless browser, capture frame 1, assert (a) ≥2 birds present, (b) their pose parameters are not at phase 0, (c) no spinner element in the DOM, (d) palette matches the mocked local time.
- **No-crop test:** render at 20 viewport sizes from 320×568 to 3440×1440; assert every bird's bounding box is inside the viewport with ≥8px margin.
- **Loop-detection test:** capture 90s of pose parameters for one bird; assert no exact repeat and autocorrelation below threshold. This is the mechanical version of "never identical twice," and it catches the accidental introduction of a clip-based animation.
- **Greeting variation test:** simulate 50 session starts with identical state; assert ≥20 distinct greeting realizations and no realization occurring more than 3 times.
- **Listen-in mix test:** assert no bird's gain node reaches ≤ -60dB during listen-in; assert ramp durations within spec.

### 12.4 Copy linting

A single lint pass over the notebook corpus, narration corpus, caption corpus, and all UI strings.

**Naturalist surfaces must:** start lowercase; use present tense; contain no `!`; contain no second person (`you`, `your`, `you've`); contain no numerals describing user behavior.

**Banned lexicon everywhere (I7):** `streak`, `achievement`, `badge`, `level`, `score`, `points`, `XP`, `rank`, `tier`, `milestone`, `unlocked`, `progress`, `days in a row`, `visits`, `welcome back`, `you've been`, `keep it up`, `don't lose`, `congratulations`, `leaderboard`, `share your`, `invite friends` (as a prompt — the invite *flow* copy is scoped separately and reviewed).

**Matter-of-fact surfaces must:** be sentence-cased; avoid naturalist vocabulary (`perch`, `settle`, `notice`, `flutter`); state what happened and what to do.

The linter runs in CI and is the mechanical form of the voice discipline. Voice rules that live only in a doc are voice rules that erode.

---

## 13. Rollout

### 13.1 Milestones

**M0 — Foundations (weeks 1–2).** Repo, CI, infra-as-code, Postgres schemas, auth (magic link end-to-end), synthetic-UUID discipline and its tests, telemetry VPC separation, bundle-budget CI. *Exit:* a user can sign in and see an empty page; I8 and I9 have passing tests.

**M1 — Engine v0 + calibration harness (weeks 3–5).** Tick worker, drift, mood, perch, weather, call scheduling, event ingest and validation. The calibration harness (§5.10) is built here, *before* tuning. *Exit:* the harness runs 2 simulated years across 5 behavior profiles; drift lands in target bands; I1/I2 enforced.

**M2 — Renderer + first frame (weeks 4–7, overlapping).** WebGL layer, bird rig, idle oscillators, interpolation, responsive layout, edge-inlined bootstrap. *Exit:* first-frame test passes; 60fps on the reference laptop; TTFB-bird < 500ms on the throttled device.

**M3 — Audio (weeks 6–8).** Synthesis, motif libraries for six species, chorus mixing, listen-in, ambient bed, fallback path. *Exit:* a blind listening test — a team member who has "known" a bird for a week identifies it from among seven by call alone, ≥80% accuracy. If this fails, the cap of seven is wrong or the voice model is, and we fix it before shipping rather than discovering it from users.

**M4 — Interactions + notebook (weeks 8–10).** Greeting, listen-in, offers, settle, presence accounting, notebook corpus + detectors. The corpus authoring starts in week 2 and lands here. *Exit:* copy lint passes; sparsity budget holds over a simulated month; greeting variation test passes.

**M5 — Accessibility (weeks 7–11, overlapping and continuous).** Narration composer + corpus, reduced-motion renderer + designed pose sets, captions, keyboard nav, focus treatment, contrast. *Exit:* the moderated sessions in §9.5 return "it felt like a place," not "it was usable."

**M6 — Accounts, social, settings (weeks 10–12).** Session management, email change, export, soft/hard delete, invites, visitor mode, visit log. *Exit:* revocation effective within one poll; visitor events produce zero drift (I10).

**M7 — Hardening + private beta (weeks 12–15).** Load testing the tick at 10× projected accounts, 30-minute memory soak, cross-browser matrix, incident runbooks, the full I-list audit. Private beta with 50–100 invited users for **at least four weeks** — the drift calibration cannot be validated in less than three, because three weeks is the number the PRD gives for user-visible change. A two-week beta would ship a drift function nobody has ever seen work.

**M8 — Public v1.**

### 13.2 Ramping birds per aviary

The cap is seven; the mechanic is age-gated. Ship v1 with the offer schedule at: **third bird at 8 weeks, fourth at 20 weeks, fifth at 40 weeks, sixth at 70 weeks, seventh at 110 weeks** — roughly matching the PRD's "a few months old offers a third; a year-old may have grown to five or six."

But the *audio* cap is what makes seven safe, and no beta user will reach even four birds before launch. So:

- **Server-side config, not code:** the schedule is a config value, changeable without deploy.
- **Pre-launch validation:** internal accounts are seeded at 4, 5, 6, and 7 birds from M3 onward, and the blind listening test runs at each count. If recognizability degrades at six, we lower the cap in config and say so.
- **Post-launch monitoring:** aggregate audio-mix metrics (voice-steal rate, concurrent-voice histogram) tell us if the mix is saturating. There is no per-account bird-count metric — the histogram is population-level and dimensionless.
- **The offer is refusable and not repeated aggressively:** a declined new-bird offer re-appears after a long interval. A user who wants two birds forever should be able to have two birds forever without being nudged.

### 13.3 Instrumented from day one

Bundle size, TTFB-bird histogram, frame-timing percentiles, tick latency/lag, tick failure rate, event-ingest lag, audio-context failure rate by reason, WebGL-unavailable rate, magic-link delivery rate, error rates by endpoint, synthetic checks from four geographies. All aggregate. All defined against the exclusion list before the first metric is emitted, so that the boundary is established by the first PR rather than retrofitted onto twenty.

### 13.4 Launch gates

Ship only when: all I-list tests green; calibration harness in band; the blind listening test passes at the shipping cap; the a11y sessions return "felt like a place"; TTFB-bird p75 < 500ms in the field from beta RUM; the 30-minute memory soak is flat; four weeks of beta with no drift anomalies.

---

## 14. Judgment calls made **[CALL] index**

| # | Ambiguity | Call | Reasoning |
|---|---|---|---|
| C1 | Export includes "current personality vectors" (`accounts_sync.md`) vs. vectors never visible in any tier (`bird_engine.md`) | Exclude numbers; include a generated naturalist paragraph per bird | The never-expose rule is argued at length and is load-bearing; an export is a stats panel with extra steps, and diffed exports manufacture the optimization surface. Reversible in one field if overruled. |
| C2 | Mood set "finalized in implementation" | `wary, content, curious, drowsy, alert, settled` | `settled` is needed by both full night and the settle gesture and is named as a state in `aviary_layout.md`. |
| C3 | Transport for state | Polling (45s, jittered) + 90s schedule horizon, not WebSockets | One small message per minute doesn't justify a stateful tier; interpolation + horizon covers smoothness. |
| C4 | Presence activity window ("a few minutes", lean long) | **5 minutes** | Watching without moving is the actual product; 5 min is long enough for genuine stillness, short enough to exclude an abandoned desk. |
| C5 | Concurrent multi-device presence | Union, not sum | Summing is a grind mechanic shipped by accident. |
| C6 | Unbounded daily presence | Cap at 4h/day | A second-monitor user shouldn't drift 8× an attentive user; the cap is above any ordinary session. |
| C7 | Notebook/narration prose generation | Hand-written structured corpus, no runtime LLM | Determinism, zero latency, no hallucination, no voice drift, no per-bird egress — in the surface where voice matters most. |
| C8 | Renderer | WebGL2 + Canvas2D fallback | 60fps × 7 articulated birds × 30 min on a 5-year-old laptop with margin; palette LUT is free in a shader. |
| C9 | Personality signal the client needs for rendering | 3-bucket `expressiveness` hint + 6-step `plumage_step` | Clients need *some* signal; buckets are coarse enough to be non-reversible and satisfy I3. |
| C10 | Plumage drift visibility | 6 discrete steps, not continuous | Continuous saturation drift is imperceptible; discrete steps make the trait actually do something for the user. |
| C11 | Does a visitor get a greeting? | No | Visits are observation, not co-presence; greeting is host-specific and drift-relevant. |
| C12 | Drift saturation | `(1 - value)^1.5` asymptotic | Pure monotonic drift pins every long-term bird at 1.0 and destroys per-bird distinctness. |
| C13 | Offer cooldown surfacing | The option quietly isn't offered; no timer UI | A countdown is a counter, and counters are the thing. |
| C14 | Tick catch-up after outage | Integrate the gap, don't replay N ticks | Cheaper, and avoids non-linear artifacts in the mood chain. |
| C15 | Species pool composition | 6: warbler, wren, finch, tit-like, thrush-like, nightjar-like | Six per PRD; nightjar-like is required by the night-activity rule. |
| C16 | Consumed-event retention | 30 days | Enough for incident replay; short enough to honor the privacy claim and to avoid building a vector-reconstruction capability. |

---

## 15. Risks

Ordered by expected damage × likelihood, each with a mitigation that is a *mechanism*, not an intention.

### R1 — Drift calibration is wrong, and we find out from users (high impact, high likelihood)

The failure is silent by construction: too-fast drift reads as "my birds changed overnight," too-slow reads as "nothing I do matters," and neither generates a bug report — they generate quiet churn. Three weeks is the feedback loop, which means a wrong constant costs a month per iteration.

*Mitigation:* the calibration harness (§5.10) built **before** tuning, running 1000× wall clock across five behavior profiles, with its assertions wired into CI as regression tests. A ≥4-week private beta so at least one cohort crosses the three-week visibility threshold before launch. Drift constants held in server config so a correction is a config change, not a release. Post-launch, an aggregate (account-dimensionless) histogram of *time-to-first-visual-step* — the one drift-adjacent metric that can be collected without violating I9, because it's a population distribution with no per-account dimension.

### R2 — Audio uncanniness: procedural calls sound synthetic (high impact, medium likelihood)

This is the risk most likely to be discovered too late, because the team will habituate to the calls over months of development and lose the ability to hear them fresh. A synthetic-sounding call fails the product's affective spine, and there is no fallback — recorded audio is forbidden.

*Mitigation:* a dedicated audio-capable engineer from M3, not a generalist doing audio on the side. **Fresh-ears reviews**: every two weeks, three people who haven't heard the calls recently rate naturalness blind, and the ratings are tracked over time so habituation is visible. The blind recognizability test (§13.1 M3) as a hard gate. A/B against real field recordings *internally only* (as a target, never shipped) to keep the reference honest. If naturalness stalls, the fallback is *fewer, sparser, quieter* calls with more ambient bed — a restrained aviary beats an uncanny one — and, if necessary, lowering the bird cap.

### R3 — Announcement creep (medium impact, very high likelihood)

Over a year, someone will ship a toast. It will be small, reasonable, and well-argued — an offer confirmation, an "invite sent" chip, a settle acknowledgment, a "your export is ready" banner. Each is individually defensible and collectively fatal.

*Mitigation:* the design system **has no toast, banner, or badge primitive** — adding one is a visible PR to shared infrastructure, not a component import. The component allowlist lint (I6). The copy lint (§12.4). A standing item in design review: "does this surface announce, or does it let the user notice?" And an a11y-tree snapshot test asserting that session start produces no live-region announcement other than narration — which catches the sneakiest version, where a toast is added for "accessibility."

### R4 — Sync correctness regression (high impact, low likelihood)

The architecture makes last-write-wins unreachable *today*. The risk is a future feature — offline mode, an optimistic UI shortcut, a "faster feedback" optimization — that introduces a client-side personality write and reopens the hole. The failure is invisible: birds drift slightly slower, no error is logged, no test fails.

*Mitigation:* I1 enforced at three layers (DB grants, package boundaries, CI grep) so a violation cannot compile or deploy. A documented ADR that any PR touching the sync model must cite. A chaos test in CI: two simulated clients producing overlapping events across a tick boundary, asserting the folded result equals the sequential fold.

### R5 — Accessibility ships as a fallback despite intent (high impact, medium likelihood)

Reduced-motion and narration are the surfaces most likely to be descoped under schedule pressure, precisely because they're "done enough to be usable" long before they're good. And "usable" is the exact bar the PRD rejects.

*Mitigation:* M5 runs *parallel to* M2–M4, not after. Reduced-motion pose sets are a **designer deliverable** in the same sprint as the full renderer, so descoping them is visibly descoping design work rather than quietly deprioritizing engineering polish. The moderated-user acceptance criterion (§9.5) is a launch gate with a qualitative bar. `NarrationComposer` reads the same `SceneModel` as the renderer (§2.4), so narration cannot structurally fall behind the visual surface.

### R6 — Performance budget erosion (medium impact, high likelihood)

2MB feels enormous until settings, notebook, visit flow, and a date library are in the bundle. Then TTFB-bird crosses 500ms and the first frame becomes a load state.

*Mitigation:* internal targets at 400KB / 350ms, well under the caps. Hard CI failure, not a warning. Bundle composition tracked per-PR with a visible delta. Aggressive code-splitting as an architectural rule (§10.2), with everything outside the aviary route lazily loaded. Nightly throttled-device measurement so regression is caught in a day, not a quarter.

### R7 — Notebook prose goes stale or off-voice (medium impact, medium likelihood)

A user reading their tenth "pip greeted before wren today" learns the notebook is a template, and the discovery retroactively cheapens every entry. This is the specific failure the PRD warns about most vividly.

*Mitigation:* corpus size floor (12–20 phrasings per detector, ≥15 detectors) validated by a test that simulates a year and asserts no repeated `(template_id, phrasing_index)` within 14 days and no `template_id` within 3 days. The copy linter. A quarterly corpus-expansion task budgeted as ongoing content work, not a one-time deliverable — the notebook is content, and content that never grows goes stale by definition.

### R8 — The bird cap of seven is wrong (medium impact, medium likelihood)

Seven is asserted as empirical but has not been measured with *our* synthesis. If recognizability actually collapses at five, we discover it a year after launch when users reach five birds.

*Mitigation:* the blind listening test at every count 4–7 during M3, before launch, on internal seeded accounts. Cap held in server config. The offer schedule is slow enough (third bird at 8 weeks) that we have months of runway after launch to lower the cap before anyone is affected.

### R9 — Tick outage is user-visible (medium impact, low likelihood)

The one outage users can feel: moods stall, drift stalls, the world stops. The client hides it for a while by rendering plausibly, which is good, but a multi-hour tick outage means a lost afternoon of everyone's drift.

*Mitigation:* the tick is deployed independently and never taken down by an API rollout. Partition-level lag alarms at 5 minutes. Catch-up integrates elapsed time (§5.1), so a recovered outage restores the correct state rather than losing it — events are still in the log and get folded when the tick returns. Unconsumed-event-age alarm at 10 minutes as an independent detector of the same failure.

### R10 — Privacy boundary erosion (high impact, low likelihood, unrecoverable)

Someone joins the analytics warehouse to the simulation database for one reasonable question. Once done, it's done — the data has been copied and the boundary is now a policy, not an architecture.

*Mitigation:* separate VPC with explicit deny rules. A metrics client whose function signatures cannot accept an account identifier. Quarterly metric-definition review against the exclusion list. And the deliberate absence of engagement metrics (§10.4) — the strongest defense is that nobody has a dashboard that would motivate the join.

---

## 16. Open items for the PRD author (non-blocking)

1. **C1 (export contents)** — confirm the exclusion of trait numbers from the export. One-field change if overruled; we build to the exclusion in the meantime.
2. **Species names and visual direction** — the six species need naming and silhouette design from the visual designer; the engine treats species as config, so this is not on the critical path.
3. **Design-system spec** — palette values, contrast ratios per surface, focus-indicator treatment, and reduced-motion pose sets are referenced by the PRD as living in a separate document with the visual designer. They are required by M2/M5 and should be scheduled accordingly.
4. **Notebook corpus authorship** — needs a writer, starting week 2. This is the single largest non-engineering deliverable and the one most likely to be discovered late.

---

*End of plan.*
