# Pocket Aviary — v1 Implementation Plan

Run: `wave_003` / plan `001` · Planner: claude-5.5-opus (effort: max) · Harness: claude-code
Status: phase-1 planning deliverable. This document is the plan only; nothing in it has been implemented.

---

## 0. How to read this plan

- **§1** is the one-page shape of the system. **§2** is the invariants matrix: the product rules that carry enough weight that we enforce them through architecture rather than code review. Later sections point back to them as `I1`–`I17`.
- **§3–§16** are the executable plan, subsystem by subsystem. **§17–§19** cover testing, rollout, and staffing. **§20** lists risks. **§21** records every place where the PRD was ambiguous or contradicted itself, and the decision we made there.
- Numbers marked **(cal)** are starting values. The calibration harness or build-time measurement will tune them. Any number without that mark is a decision.
- Vocabulary follows the PRD glossary exactly: *bird* (never creature, pet, or character), *call* (never song, chirp, or noise), *listen-in*, *offer*, *settle*, *field notebook*, *visit*, *tick*, *adopt*. Code identifiers use the same words (`listenIn`, `settle`, `offer`, `Call`). That keeps the vocabulary intact in the codebase, in review comments, and in the heads of new contributors.

---

## 1. Summary

Pocket Aviary is a browser-only aviary: one horizontal scene holding two to seven birds, each animated and voiced procedurally. Every bird has a hidden personality that drifts slowly toward expressive in response to the user's honest attention. The product's value is affective. The aviary has to feel like a place that was already going before the tab opened. It notices the user and never announces anything. It changes over weeks, not within a session. This plan treats each of those feelings as a hard system property.

**The key bets:**

1. **A server-authoritative simulation on a slow tick.** Each account has one canonical aviary. A server-side tick advances it every ~60 s whether or not anyone is watching. The tick is the only writer of personality, mood, and perch state. Clients are renderers: they pull snapshots, realize them procedurally, and append interaction events to a log. This one decision delivers three things together: "the aviary continues without the viewer," multi-device sync, and freedom from write conflicts.
2. **Drift is a monotonic, lagged low-pass filter over honest presence.**
   - *Presence* is the strict conjunction of three signals: the page is visible, the window is focused, and there was recent pointer or key activity.
   - Presence-time is the dominant drift input.
   - Traits only rise, each toward that bird's own ceiling.
   - A multi-day release lag means drift keeps flowing from pre-departure inputs while the user is away, and no single session is ever visible on its own.
   - The calibration targets are drift measurable after about one week and visible after about three. We verify them with a simulation harness and consented dogfooding. We never verify them with production analytics, because the privacy commitment forbids that.
3. **The server decides what is true; the client decides how it looks and sounds.** The render boundary is a small snapshot: canonical state plus a short timeline of scheduled intents. The client owns all micro-motion, call synthesis, captions, and narration. All of it is procedural and freshly seeded every session.
4. **All audio is synthesized on the client.** An AudioWorklet voice pool plays calls generated from per-bird call grammars. Each bird's signature parameters are fixed at adoption, so the bird stays recognizable across mood and drift, yet no call is ever played the same way twice. No recorded audio exists anywhere in the product, including the fallback path.
5. **The first bird is visible in under 500 ms, already mid-action.** HTML is streamed from the edge with the snapshot inlined. The critical path is a small Canvas 2D renderer. Behavior controllers are warm-started, so the first frame catches a bird mid-preen. If the snapshot is late, the user sees a quiet field, never a spinner.
6. **Accessibility ships in v1 as designed surfaces.** Screen-reader narration is naturalist prose generated from the same scene state the renderer uses. Captions come from the same call descriptors that drive synthesis. Reduced-motion mode renders the scene as slow cross-fades between designed still poses.
7. **Privacy is an architectural boundary.**
   - Synthetic UUIDs are used everywhere; email is stored once, encrypted.
   - Per-bird interaction data has no network or credential path into telemetry or analytics.
   - Metrics use an allowlisted label schema with no account dimension.
8. **Refusals are enforced by absence.**
   - The design system ships no toast, badge, spinner, counter, or streak component.
   - The API has no endpoint that sets a trait or places a bird.
   - The notebook writer's input schema contains no data about the user's behavior.

   Features that would break the product are hard to add by construction.

---

## 2. Product invariants and how they are enforced

Each invariant lists three things: **Why** (the product intent it protects), **Enforced by** (the mechanism, preferably structural), and **Verified by** (the test or monitor).

**I1 — Presence is honest.** A presence-event exists only when three conditions hold at once: `document.visibilityState === 'visible'`, the document has window focus, and a pointer or key event occurred within the activity window (4 min **(cal)**, biased long).
- *Why:* presence-time is the dominant drift input. Any laxer definition, "tab open" above all, would silently speed up drift for every account. The failure would never show in a test; it would only surface as birds changing faster than designed.
- *Enforced by:*
  - The client `PresenceMonitor` emits intervals only while all three signals hold.
  - The server credits the per-account **union** of intervals, so two devices never double-count, and caps credit at wall-clock time.
  - Visitor tokens are rejected, and missing data fails closed: we under-count rather than inflate.
- *Verified by:* table-driven tests over signal sequences, server property tests, and harness personas. "Tab hidden for 48 h" must produce zero drift.

**I2 — Drift is monotonic toward expressive, and neglect never lowers a trait.**
- *Why:* this is the engine-level form of "not a Tamagotchi." A user who returns after two weeks should find birds that are quieter than before, not birds that have learned to mistrust them.
- *Enforced by:*
  - Every drift input is non-negative by type, and the release step only adds.
  - A database trigger rejects any update that would lower a trait column.
  - The quieting after absence comes from a separate, non-personality `recent_attention` term. That term can only lower expression amplitude, and only down to an "ambient" floor. It never feeds mood valence: a bird is never wary *of the user*.
- *Verified by:* property tests over 10⁵ random traces, absence personas in the harness, and a production invariant-violation counter that is expected to stay at zero.

**I3 — Only the server tick writes personality, mood, and perch state. Last-write-wins is never used.**
- *Why:* if a stale device could overwrite a vector, a morning's drift would disappear, silently and with no log entry.
- *Enforced by:*
  - Postgres role grants: only the `sim_tick` role can `UPDATE` `personality` or `bird_state`.
  - No API code path accepts trait values.
  - Clients append events; the tick consumes them in server-assigned order and applies additive, non-negative deltas.
  - Each tick commits with a compare-and-set on `tick_seq`, and writes the event cursor in the same transaction.
- *Verified by:* chaos tests (two workers holding one lease, killed transactions, replayed events), a lint rule, and API contract tests.

**I4 — The personality vector is persisted canonical state and is never recomputed from history.**
- *Why:* losing or regenerating a vector deletes the bird the user has been getting to know. That failure is invisible to unit tests and felt by the user.
- *Enforced by:*
  - A stored row per bird.
  - A journal of *applied outputs* (per-tick deltas), used only for audit and disaster recovery, never for re-derivation.
  - Drift-function versions that apply to future ticks only.
  - Synchronous replication plus point-in-time recovery (PITR).
  - A migration lint that blocks any migration writing to trait columns.
- *Verified by:* checksum audits before and after every deploy and migration, and quarterly DR drills.

**I5 — Personality numbers are never shown**, in any surface, version, tier, debug view, or ARIA attribute.
- *Why:* once a user sees "boldness: 0.62," the bird becomes a stat to manage.
- *Enforced by:*
  - Raw vectors never leave the simulation service. The snapshot carries only derived, mood-blended expression parameters, and none of them is named after a trait.
  - No stats, debug, or "how is my bird doing" surface exists.
  - The export carries personality only as a sealed blob (§21 D2).
- *Verified by:* snapshot and export schema tests, plus an accessibility-tree scan that fails on digits in bird descriptions.

**I6 — Bird identity is stable.**
- *Why:* weeks of drift only mean something if the bird is still the same bird.
- *Enforced by:*
  - `bird_id` is immutable.
  - Call signature and trait ceilings are immutable after adoption.
  - Renames touch only `name`.
  - Any species-art or synthesizer change must pass signature-stability regression (§11.4). Otherwise an audio refactor could silently "swap" Pip for a stranger.
- *Verified by:* a regression suite that re-renders every stored signature version and compares the results perceptually.

**I7 — Notice, never announce.** The product has no toasts, banners, badges, confetti, welcome text, "you've been gone" surfaces, or notifications. The one exception is the opt-in visit email.
- *Enforced by:*
  - The component library does not contain these primitives.
  - A lint forbids importing notification-style components into product surfaces.
  - The app never requests Notification permission.
- *Verified by:* an end-to-end test asserting that returning to the tab produces no new visible text, and a per-release UI audit.

**I8 — Aliveness.** The first frame is mid-action. There is no entry animation, no spinner, and no "ready" pop. Calls are procedural. Greetings are never identical.
- *Enforced by:* the quiet-field loading state, warm-started controllers, the absence of any spinner component, procedural-only audio, and a per-bird anti-repetition memory.
- *Verified by:* first-frame visual tests and audio repetition tests (§17).

**I9 — An aviary holds two to seven birds, and new birds arrive only with aviary age.**
- *Enforced by:* a database constraint plus an engine check on the count. The newcomer scheduler takes aviary age as its only input: it has no access to visit counts or interaction data, and there is no paid tier.

**I10 — One screen, no chrome inside the scene, and the user never places birds.**
- *Enforced by:* a layout solver that keeps every bird in frame at every viewport, no placement API, and a top bar limited to the controls specified in §10.8.

**I11 — Voice split.** Product surfaces use the naturalist voice; system surfaces (identity, errors, settings, and later money) use the matter-of-fact voice.
- *Enforced by:* a surface registry, two string catalogs, and a separate lint for each (§12).

**I12 — Privacy boundary.** Per-bird interaction data drives only that user's own simulation.
- *Enforced by:*
  - Network and IAM isolation of the simulation database; the analytics side has no route or credential to it.
  - A telemetry label allowlist; no metrics broken down by event type.
  - No third-party analytics, session replay, or ML.
- *Verified by:* CI schema checks, log PII scanners, and boundary penetration tests.

**I13 — A synthetic account UUID is used everywhere. Email is stored once, encrypted.**
- *Enforced by:* envelope encryption, a blind index confined to auth tables, and lint rules that forbid email-derived values in log fields, partition keys, or message keys.

**I14 — Visits are read-only and inert.**
- *Enforced by:*
  - Visitor tokens are scoped to `aviary:read`.
  - Ingestion rejects any event carrying a visitor token.
  - The visitor client is built with the presence and interaction modules compiled out.
  - There is no co-presence and no notification unless the host opts in.

**I15 — Accessibility ships with v1 as designed surfaces.** Narration, captions, reduced motion, and the keyboard model are launch gates, not follow-ups.

**I16 — No visit-frequency surface exists anywhere.** That covers streaks, calendars, counts, the export, and the notebook.
- *Enforced by:* the export excludes presence data, and the notebook writer has no access to session or presence data.

**I17 — Performance budgets are CI gates.** The initial bundle stays under 2 MB gzipped, the first bird appears in under 500 ms (reference device over 4G), idle motion holds 60 fps on a five-year-old laptop, and client memory does not grow across a 30-minute session.

---

## 3. Scope

### 3.1 In v1

| Area | What ships |
|---|---|
| Aviary | One scene per account, 2 starter birds, age-gated newcomers up to a hard cap of 7, three perch zones, local-time day/night, rare ambient weather, ambient leaf/feather drift, responsive layout that never crops a bird |
| Bird engine | 5-trait hidden personality, monotonic lagged drift, mood FSM with cross-session persistence, procedural call grammars, bird-to-bird interaction, ~6-species pool, naming/renaming, stable identity |
| Interactions | Return-greeting, listen-in, offer (seed / song fragment / still pool), settle (+5 s undo), field notebook, presence accounting |
| Accounts | Magic-link sign-in, per-device revocable sessions, verified email change, JSON export, 30-day soft delete → hard delete |
| Sync | Server tick, snapshot pull + interpolation, append-only event log, multi-device by construction |
| Social | Visit invitations: per-invite, email-addressed, one-time link, read-only, revocable, 30-day expiry, visit log, opt-in visit email (off by default) |
| Accessibility | Screen-reader narration, call captions, reduced-motion mode, full keyboard model, WCAG AA contrast, visible focus |
| Platform | Last two major versions of Chrome, Safari, Firefox, Edge; a matter-of-fact unsupported-browser page for everything else |

### 3.2 Explicitly out of v1

| Out of scope | Why (the design reason we are protecting) |
|---|---|
| Native iOS/Android apps | One team, one client; engine quality beats a second client. We do not shape protocols around native constraints. |
| Gamification | Achievements, streaks, levels, scores, badges, XP, bird counters, visit calendars — none, ever, in any form (not a toggle, not opt-in). |
| Tamagotchi mechanics | No death, hunger, distress, decaying happiness. The relationship is observational, not custodial. |
| Social-network surfaces | No profiles, follows, feeds, discovery, friend-of-friend, mutual visits, comments, chat, avatars, leaderboards, show-off mode. |
| Payments / tiers | Nothing in v1 is paid; no tier ever gates birds or features. |
| Shared / multi-aviary accounts, customizable scenes | One user, one aviary, one designed scene. |
| Push notifications, re-engagement email | Pocket Aviary never reaches for the user about the aviary. |
| SSO / passwords | Deferred; magic link is the v1 auth model. |
| Localization | v1 is English-only: the naturalist voice is hand-authored, and each language would need its own writer. Strings are externalized so localization stays possible later. |

### 3.3 Refusals register (checked into the repo as `REFUSALS.md`, referenced in the PR template)

These are features a well-meaning contributor will propose. Each one is pre-rejected, with the reason recorded so the argument never has to be re-run. The list: welcome toast; "you've been away N days"; any streak, visit counter, or calendar of green dots; a notebook entry about the *user's* behavior; a bird-count display; a trait stats or debug panel; a "how is my bird doing" view; hunger or health meters; visible distress; drag-to-place or "send Pip to the front"; a species catalog or picker; rarity; prettified show-off rendering for visitors; a default-on "your friend visited" notification; public aviaries; leaderboards (we do not even compute the underlying stats); push notifications; re-engagement email; an "install app" prompt; loading spinners; entry or "wake up" animations; recorded or looped audio, including as a fallback; a "click to enable sound" overlay; coach marks or tutorial overlays; per-account product analytics; A/B tests of affective mechanics.

### 3.4 Terminology enforcement

A CI word-list check runs over product-surface strings and spec docs. It rejects "creature/pet/animal/character" in place of *bird*, "song/chirp/noise" for a bird vocalization (the *song fragment* offer is the one named exception), "solo/select/highlight/pin" for *listen-in*, and "acquire/unlock/earn" for *adopt*.

---

## 4. Architecture

### 4.1 System context

```
 Browser (host or visitor)                          CDN edge (global)
 ┌──────────────────────────────┐   HTML+inline    ┌──────────────────────────────┐
 │ App shell + Scene renderer   │◀──snapshot───────│ Edge worker: session check,  │
 │ Audio engine (AudioWorklet)  │   static assets  │ stream HTML, inline snapshot │
 │ Presence monitor             │◀─────────────────│ (never caches per-user HTML) │
 │ Narration / captions         │                  └──────────────┬───────────────┘
 │ Chrome (top bar, panels)     │                                 │ regional fetch
 └──────┬───────────────▲───────┘                                 ▼
        │ events,       │ snapshot pulls          ┌───────────────────────────────┐
        │ offers,       │ (ETag = tick_seq)       │ aviary-api (stateless, modular│
        │ settings      │                         │ monolith): auth · ingest ·    │
        ▼               │                         │ offers · snapshot · notebook ·│
 ┌──────────────────────┴──────────────────────┐  │ birds · settings · visits ·   │
 │                 Regional API                 │──│ account lifecycle             │
 └──────────────────────────────────────────────┘  └──────┬───────────────┬────────┘
                                                          │               │
                       ┌──────────────────────────────────▼──┐   ┌────────▼─────────┐
                       │ Zone B: aviary DB (Postgres)         │   │ Zone A: identity │
                       │ birds · personality · bird_state ·   │   │ DB (Postgres)    │
                       │ events · journal · notebook · …      │   │ accounts (enc.   │
                       └──────────────▲───────────────────────┘   │ email) · sessions│
                                      │ only writer of personality │ · links · invites│
                       ┌──────────────┴───────────────┐           └──────────────────┘
                       │ sim-worker fleet (tick,       │   Redis: snapshot cache, leases,
                       │ snapshot build, notebook,     │   rate limits, revocation sets
                       │ weather, newcomers)           │   Object storage: exports, assets
                       └──────────────────────────────┘   jobs-worker: email, export, deletion

 Zone C (telemetry plane, separate accounts/network): RUM collector · metrics · logs · traces
   ← receives only allowlisted aggregate fields; has no route or credential to Zone A or Zone B
```

### 4.2 Services and processes

- **Edge worker** (CDN compute, e.g. Cloudflare Workers or Fastly Compute):
  - Validates the short-lived access token (signature, expiry, and a replicated revocation set).
  - Fetches the current snapshot from the nearest regional snapshot cache.
  - Streams the HTML: inline critical CSS, the sky color computed for the host's local time, a `modulepreload` for the renderer chunk, and the snapshot as an inline `application/json` block. It sends `103 Early Hints` for the renderer and the species art.
  - Marks per-user HTML `Cache-Control: private, no-store`. Static assets are content-hashed and immutable.
- **aviary-api**: one deployable, stateless, horizontally scaled, with enforced internal module boundaries (`auth`, `ingest`, `offers`, `snapshot`, `notebook`, `birds`, `settings`, `visits`, `lifecycle`). An import lint enforces those boundaries, and each module uses its own database role.
  - A modular monolith is the right shape for one team: the bird engine gets the attention, not service plumbing.
  - Only `ingest`, `offers`, and `snapshot` touch Zone B, and only with append or read privileges.
- **sim-worker fleet**: long-running workers that own aviary shards through leases and run the tick (§7.1). The same process materializes snapshots, runs the notebook writer, plans weather, and schedules newcomers. These are the only holders of the `sim_tick` database role.
- **jobs-worker**: a queue consumer (Postgres-backed queue or SQS). It sends transactional email (magic links, email-change confirmations, export links, invitations, opt-in visit emails), builds exports, runs the deletion saga, and expires invitations.
- **Datastores**:
  - **Postgres `identity`** (Zone A): accounts, sessions, links, settings, invitations, visit log.
  - **Postgres `aviary`** (Zone B): simulation state and the event log. Both clusters use synchronous replication, PITR, and 35-day backups.
  - **Redis**: snapshot cache, shard leases, rate limits, session revocation set.
  - **Object storage**: export files (72 h lifecycle) and static assets.
- **Telemetry plane** (Zone C): an unauthenticated RUM beacon collector (strips IP addresses, aggregates at ingest), metrics (OpenTelemetry → Prometheus-compatible), logs, and traces. It runs in separate cloud accounts with schema-enforced allowlists (§16).

### 4.3 Client/server split

| Concern | Server (authoritative) | Client (realization only) |
|---|---|---|
| Personality, drift, ceilings, reservoir | computes, stores, never exports raw | never receives |
| Mood (state, intensity, timers) | transitions each tick | blends expression over tens of seconds |
| Perch zone/slot, flights between zones | plans as timed intents | animates flights/hops; never invents zone changes |
| Social events (call-response, chorus windows, alarm calls) | schedules with timestamps | realizes them as actual calls and gestures |
| Ambient call rate, pitch/tempo ranges | derives parameters | runs the stochastic call scheduler and synthesis |
| Idle micro-motion (preen, scan, tilt, shuffle) | supplies mood/tempo parameters | fully procedural, seeded per session |
| Leaves, feathers, parallax, foreground passes | — (no per-leaf state) | pure ornaments |
| Weather events | schedules per aviary (all devices and visitors see the same rain) | renders rain/wind visuals and procedural audio |
| Day/night | computes light phase from host timezone | renders sky, grade, and light transitions |
| Greeting | supplies absence data and per-bird greeting propensity/style | picks greeter and order, realizes procedural variation |
| Offer reaction | decides who reacts and how (synchronous) | animates placement and reaction |
| Listen-in mix | records start/end events | performs the mix change immediately |
| Settle | records the event (mood quieting, presence end) | runs the lighting shift, undo window, quiet calls |
| Presence | credits the union of intervals, capped | detects the three signals, emits intervals |
| Notebook | detects, writes, stores (immutable) | displays, paginated and virtualized |
| Narration and captions | — | generates from local scene state and call descriptors |

### 4.4 The render pipeline boundary

The snapshot is the contract. It says **what is true now and what is scheduled to happen soon**: per-bird perch, mood and intensity, expression parameters, a 2–3 minute timeline of intents, active weather, active offers, light phase, and absence data for the greeting. It never carries raw trait values, reservoirs, ceilings, event history, or anything about other accounts.

The client turns the snapshot into a continuous scene. Everything it draws or plays is a *realization* of the snapshot, never a source of canonical state. When the client and the snapshot disagree, the snapshot wins, and the difference is resolved through natural motion rather than a snap (§8.4).

This boundary is what makes three rules true together: "clients never tick," "clients never own state," and "the first frame is the aviary, which has been there."

### 4.5 Technology choices

- **TypeScript end to end.** Shared packages cover the snapshot schema, the prose realizer used by both the notebook (server) and narration (client), and the call-descriptor types.
- **Server:** Node 22 LTS on the API and workers, Postgres 16, Redis 7, and a CDN edge runtime.
- **Client scene:** framework-free TypeScript on **Canvas 2D**, rendered as layered canvases plus CSS-composited sky and grade layers.
  - WebGL is not required. With at most 7 birds of ~12 sprite parts each plus sparse particles, Canvas 2D holds 60 fps with room to spare.
  - Skipping WebGL removes shader-compile stalls from the first-frame path and keeps the renderer small.
  - The render core has no DOM dependencies, so it can move into an `OffscreenCanvas` worker later if profiling calls for it.
- **Client chrome:** Preact (~4 KB), code-split per panel.
- **Audio:** WebAudio with an AudioWorklet voice pool. Native oscillator nodes are the secondary path.
- **Build:** Vite/Rollup, `size-limit` budgets in CI, ES2022 target.

### 4.6 Deployment topology

- **Primary region:** writes, the tick fleet, and the jobs worker.
- **Two read regions:** snapshot caches and read replicas. The edge worker uses whichever regional snapshot cache is closest.
- **Tick scheduling:** tick phases spread uniformly across each minute by `hash(aviary_id)`, so load stays flat.

A failure in the primary region degrades gracefully. Clients keep realizing their last timeline and then fall back to local idle behavior without any error surface. Ticks catch up on recovery (§7.1.4).

### 4.7 Privacy zones

- **Zone A (identity):** encrypted email, sessions, magic links, invitations, and visitor emails.
- **Zone B (aviary):** everything about the birds and every interaction event.
- **Zone C (telemetry):** aggregate operational data only.

Allowed flows:
- A → B: account UUID only.
- B → A: none.
- A or B → C: allowlisted aggregate metrics and scrubbed error events only. Error events may carry the account UUID, because "is this account having errors" is explicitly allowed. They never carry bird state, event payloads, names, or notebook text.

There is no warehouse connector, change-data-capture stream, or backup export from Zone B into any analytics system. This is enforced with separate cloud accounts, VPC isolation, and IAM deny policies, so it does not rest on policy documents alone.

---

## 5. Data model

### 5.1 Identifier policy

- **`account_id`** is a random UUIDv4, generated at account creation. It is deliberately not UUIDv7, so the ID leaks no creation time. It is the only reference to an account anywhere: foreign keys, inter-service messages, queue payloads, shard and partition keys, logs, traces, and error events.
- **Email** is stored once, in `accounts.email_ct`, envelope-encrypted with a per-account data encryption key (DEK) under a KMS key.
- **Sign-in lookup** uses `email_bidx = HMAC-SHA256(k_bidx, normalize(email))`. That keyed blind index lives only in the identity DB's `accounts` table, with a unique constraint. It never appears in logs, events, cache keys, or any other table.
- **Rate limiting** uses a *different* HMAC key, `k_rl`, in short-TTL Redis keys, so rate-limit keys cannot be joined against the accounts table.
- `aviary_id`, `bird_id`, `invite_id`, and `session_id` are random UUIDv4. `event_id` is a client-generated UUIDv7, which gives idempotency plus index locality.
- A CI lint rejects any schema, log field, metric label, or partition key whose name or derivation matches `email`. A log-shipper scrubber redacts email-shaped strings as a backstop, and every redaction increments an alerting counter.

### 5.2 Zone A — identity database

| Table | Key fields | Notes |
|---|---|---|
| `accounts` | `account_id` PK, `email_ct`, `email_bidx` UNIQUE, `dek_ref`, `timezone` (IANA), `created_at`, `status` (`active`/`pending_deletion`), `deletion_requested_at`, `hard_delete_at` | Timezone is reported by the most recently active host session (§7.8). |
| `email_changes` | `account_id`, `new_email_ct`, `new_email_bidx`, `token_hash`, `expires_at`, `verified_at` | The old email keeps working until `verified_at`; on verification the swap happens atomically. |
| `magic_links` | `token_hash` PK (SHA-256 of a 256-bit token), `account_id` NULL for first sign-in, `pending_email_ct`/`bidx` for sign-up, `purpose`, `expires_at` (+15 min), `consumed_at` | The account is created on *consumption*, never on request, so requesting links cannot create accounts for arbitrary emails. |
| `device_sessions` | `session_id` PK, `account_id`, `device_label` ("Safari on iPhone", parsed from UA), `created_at`, `last_active_day` (date only), `refresh_hash`, `revoked_at` | Day-granular activity only. This is a security list, not a visit history (I16). |
| `account_settings` | `account_id` PK, `captions` bool, `reduced_motion` (`system`/`on`/`off`), `narration_visible` bool, `shortcuts_enabled` bool, `visit_emails` bool **default false**, `updated_at` per field | Settings sync across devices with per-field LWW. That is acceptable because settings are not personality. |
| `invitations` | `invite_id` PK, `host_account_id`, `visitor_email_ct` (under the host's DEK), `visitor_email_bidx` (per-host HMAC, for dedupe and limits), `token_hash`, `created_at`, `expires_at` (+30 d), `accepted_at`, `access_until` (acceptance + 30 d), `revoked_at` | State machine in §15.1. |
| `visitor_sessions` | `visitor_session_id` PK, `invite_id`, `created_at`, `last_pull_at`, `ended_at` | Scope: `aviary:read` on one aviary. |
| `visit_log` | `visit_id`, `invite_id`, `host_account_id`, `started_at`, `approx_minutes` (derived from pull cadence) | Shown to the host only; retained 12 months **(cal)**. |
| `deletion_ledger` | `account_id`, `hard_deleted_at` | UUID-only tombstones. They are re-applied after any backup restore so deleted accounts can never resurrect, and they expire with backup retention (35 d). |
| `export_jobs` | `job_id`, `account_id`, `status`, `object_key`, `expires_at` (+72 h) | The file lives in object storage and is encrypted. |

### 5.3 Zone B — aviary database

| Table | Key fields | Write rights |
|---|---|---|
| `aviaries` | `aviary_id` PK, `account_id` UNIQUE (one aviary per account), `created_at` (**the only input to newcomer pacing**), `scene_seed`, `tz`, `lat_approx`, `weather_seed`, `tick_seq`, `last_tick_at`, `event_cursor`, `next_newcomer_at` | `sim_tick` (state); `lifecycle` (create) |
| `birds` | `bird_id` PK (immutable), `aviary_id`, `species_id`, `species_version`, `name`, `adopted_at`, `adoption_order`, `signature` JSONB (immutable after adoption), `signature_version` | `birds` module: `name` only; `sim_tick`: insert at adoption |
| `personality` | `bird_id` PK, `trait[5]` as int32 micro-units in [0, 1 000 000] (boldness, warmth, vocal, plumage, curiosity), `ceiling[5]` (immutable after adoption), `reservoir[5]` (pending drift, ≥ 0), `drift_fn_version`, `updated_tick_seq` | `sim_tick` only. The trigger rejects any lowered trait, a trait above its ceiling, a negative reservoir, or any change to a ceiling. |
| `personality_journal` | (`bird_id`, `tick_seq`) PK, `applied_delta[5]`, `post_checksum` | `sim_tick` insert-only. Retained 90 d plus monthly checkpoints; used for audit and DR, never re-derived. |
| `bird_state` | `bird_id` PK, `mood` enum, `mood_intensity`, `mood_since`, `mood_min_until`, `perch_zone`, `perch_slot`, `intents` JSONB (next ~3 min), `recent_attention` (0..1), `offer_cooldown_until`, `greeting_recent` (ring of 8 signature hashes), `updated_tick_seq` | `sim_tick` only |
| `interaction_events` | `ingest_seq` bigserial (**canonical order**), `event_id` (unique per aviary), `aviary_id`, `session_id`, `type`, `bird_id` NULL, `client_ts`, `received_at`, `payload` JSONB, `consumed_tick_seq` | `ingest`/`offers`: append only (no UPDATE/DELETE grant); `sim_tick`: marks consumed. Partitioned daily; raw rows dropped 14 d after consumption. |
| `presence_ledger` | `aviary_id`, `day_local`, `merged_intervals` (int ranges), `credited_minutes` | `sim_tick`. Kept 30 d, enough for the rolling saturation window and `recent_attention`. **Never exported or displayed** (I16). |
| `aviary_facts` | `aviary_id`, `kind`, `day_local`, `data` JSONB | `sim_tick`. Aviary observations only (greeting order, perch firsts, weather, chorus, nightjar activity). Kept 90 d. The notebook writer reads *only* this table plus bird names and species. |
| `notebook_entries` | `entry_id` PK, `aviary_id`, `written_at`, `day_local`, `template_id`@`version`, `tokens` JSONB (bird references by `bird_id`), `text_cache`, `salience` | `sim_tick` insert-only. No update or delete grant for anyone except the hard-delete saga. |
| `weather_events` | `aviary_id`, `kind` (`rain`/`wind`), `starts_at`, `ends_at`, `intensity` | `sim_tick` |
| `newcomers` | `newcomer_id` PK, `aviary_id`, `species_id`, `signature_seed`, `appeared_at`, `status` (`visiting`/`adopted`/`passed`) | `sim_tick`; `offers` marks adopt/pass via an event |

### 5.4 Static catalogs (shipped as versioned config, not user data)

- **Species pool (6):** four diurnal songbird silhouettes, one small ground-foraging bird, and one nightjar-like species that is crepuscular and nocturnal-leaning.
- **Each species defines:**
  - a rig (part list, joint limits, and a pose library, including the reduced-motion still poses);
  - a palette ramp;
  - a motif library of 8–12 motifs with a phrase grammar;
  - behavior tendencies;
  - trait baselines, seed distributions, and ceiling distributions;
  - perch preferences;
  - a `nocturnal` flag.
- **Song-fragment library:** 8 short melodic motifs, stored as synthesis parameters, never audio.
- **Default name suggestions:** a curated list of ~200 short, gentle names.

### 5.5 Retention summary

| Data | Retention |
|---|---|
| Personality and bird state, birds, notebook | Life of account |
| Raw interaction events | 14 d after tick consumption |
| Presence ledger | 30 d |
| Aviary facts | 90 d |
| Personality journal | 90 d rolling plus monthly checkpoints |
| Visit log | 12 months |
| Operational logs and traces containing `account_id` | 14 d |
| Error events | 30 d |
| Backups | 35 d |
| Aggregate metrics | 13 months (no account dimension) |
| Exports | 72 h |

On hard delete, every row for the account in Zones A and B is deleted and the DEK is destroyed (crypto-shredding the encrypted PII — account email and visitor emails — inside backups; everything else ages out of the 35-day backup window). Residual log and error records keyed by the UUID are purged by query. The tombstone goes into `deletion_ledger`.

---

## 6. API surface

### 6.1 Conventions

- **Transport:** JSON over HTTPS under `/v1`.
- **Browser auth:** two `__Host-` cookies, both `HttpOnly`, `Secure`, `SameSite=Lax`:
  - `aviary_rt`: a refresh token, 90-day sliding window, bound to a `device_sessions` row.
  - `aviary_at`: an access token, 10 min, signed, carrying the claims `account_id`, `session_id`, `aviary_id`, `tz`, and `scope`.
- **CSRF protection:** mutating requests must carry `X-Aviary-Client: 1`, which forces a CORS preflight on any cross-origin attempt. That, together with SameSite cookies, is the CSRF defense.
- **Idempotency:** a client-generated `event_id` or `offer_id`. Retries are always safe.
- **Errors:** every error returns a stable `code`. The client maps each code to matter-of-fact copy (§12.4); the server never sends product-voice prose in errors.
- **Contract-level omissions:** there is **no** endpoint that writes a trait, a mood, a perch, or a notebook entry; that deletes or edits a notebook entry; or that lists aviaries. Those absences are part of the contract (I3, I5, I10).

### 6.2 Authentication and account

| Method & path | Purpose | Notes |
|---|---|---|
| `POST /v1/auth/links` `{email}` | Request a magic link | Always `202`, identical for known and unknown emails (no enumeration). Limits: per-email 5 / 15 min and 20 / day; per IP 30 / hour **(cal)**. |
| `GET /auth/l/{token}` | Landing page for a link | Renders a one-button "Sign in" page. It does **not** consume the token, because corporate mail scanners prefetch GET links. |
| `POST /v1/auth/links/consume` `{token}` | Consume | Single use, 15-min TTL. On success: create a session and set cookies; if the account is new, create the account, the aviary, and 2 starters, then route to adoption. On failure: `LINK_INVALID`. |
| `POST /v1/auth/refresh` | Rotate the access token | Fails with `SESSION_ENDED` if the session was revoked or expired. |
| `POST /v1/auth/signout` | End this device's session | Sends `Clear-Site-Data: "cache", "storage"`. |
| `GET /v1/account/sessions` · `DELETE /v1/account/sessions/{id}` | List and revoke device sessions | Revocation writes to the replicated revocation set, so it takes effect on the next request, including at the edge. |
| `POST /v1/account/email-change` `{new_email}` → `POST /v1/account/email-change/confirm` `{token}` | Verified email change | The old address keeps working until confirmation. A matter-of-fact notice goes to the old address afterwards. |
| `POST /v1/account/export` | Request an export | `202`. A job builds the JSON (§14.4) and emails a signed link to the verified address. |
| `POST /v1/account/deletion` · `DELETE /v1/account/deletion` | Schedule deletion / "I changed my mind" | Soft delete is immediate. Restore works from any signed-in page for 30 days. |

### 6.3 Aviary state

**`GET /v1/aviary/snapshot`** (host), with `If-None-Match: "<tick_seq>:<overlay_rev>"`, returns `200` or `304`. The same payload is inlined into the edge-rendered HTML on navigation.

Clients pull:
- on load;
- on `visibilitychange` to visible;
- on `pageshow` with `persisted` (bfcache restore);
- after a render-frame gap longer than 2 s (suspend/resume);
- on the `online` event;
- as a keepalive every 60 s while visible, aligned to just after the aviary's tick phase plus 0–5 s jitter.

The snapshot is `canonical state at tick_seq` plus **display overlays** for events not yet consumed by the tick, such as an offer made 20 s ago. A second device therefore sees the same seed on the ground before the next tick makes it canonical.

```jsonc
{
  "v": 1, "aviary_id": "…", "tick_seq": 918273, "server_time": "2026-09-24T14:03:07.412Z",
  "valid_until": "2026-09-24T14:06:00Z",           // end of the intent timeline
  "light": { "tz": "America/Chicago", "lat_approx": 41.9, "phase": "morning", "sun_elev_deg": 18.2 },
  "weather": [{ "kind": "rain", "from": "…", "to": "…", "intensity": 0.35 }],
  "presence": { "last_present_at": "2026-09-23T21:40:00Z" },   // absence length for the greeting
  "birds": [{
    "id": "…", "name": "pip", "species": "tit_grey@3",
    "perch": { "zone": "front", "slot": 1 },
    "mood": "curious", "mood_intensity": 0.6, "mood_since": "…",
    "expr": {                                // derived, mood-blended, never named after traits
      "tempo": 0.62, "motion_amp": 0.55, "scan": 0.30, "preen": 0.40, "tilt": 0.70,
      "plumage": 0.4137,                     // continuous render level; client eases toward it over minutes
      "call": { "rate_pm": 1.8, "phrase": [2, 4], "loud": 0.6, "excursion_st": 1.5, "motif_w": [/*…*/] },
      "greet": { "propensity": 0.8, "style": 3 },
      "offer_ready": true                    // false during cooldown; the UI never shows timers
    },
    "signature": { "seed": "…", "v": 2 },   // immutable voice identity
    "intents": [{ "at": "…", "do": "fly", "to": { "zone": "middle", "slot": 2 } },
                { "at": "…", "do": "call_response", "with": "<bird_id>" }]
  }],
  "social": [{ "at": "…", "kind": "chorus", "birds": ["…", "…"], "dur_s": 14 }],
  "offers_active": [{ "offer_id": "…", "kind": "seed", "at": "…", "until": "…", "reactions": [/*…*/] }],
  "newcomer": null                           // or { "id": "…", "species": "…", "perch": {…} }
}
```

Size budget: 8 KB gzipped at 7 birds (§16).

### 6.4 Interaction events

**`POST /v1/aviary/events`** accepts a batch of up to 50 events and returns `202 {accepted: [event_id], rejected: [{event_id, code}]}`. **`POST /v1/aviary/events/beacon`** is the same contract with a `text/plain` body for `navigator.sendBeacon` on `visibilitychange→hidden` and `pagehide`.

| `type` | Payload | Server treatment |
|---|---|---|
| `presence` | `{from, to, seq, audible, captions}`, with interval ≤ 35 s | Kept only if non-overlapping for the session and within skew bounds. Credited as the per-aviary **union** across sessions, capped by wall clock. Intervals arriving more than 2 h late are dropped (honesty over completeness). |
| `listen_in_start` / `listen_in_end` | `{bird_id, at, reason}` | Duration credited only where it intersects credited presence. |
| `settle` | `{at}` | Sent only after the 5 s undo window passes. Closes the presence window and becomes a mood-quieting input. |
| `session_open` / `session_close` | `{kind: fresh/return, at}` | Maintains `last_present_at`. Never surfaced anywhere. |
| `greeting_rendered` | `{order: [bird_id], at}` | Becomes an aviary fact ("pip greeted first today"). |
| `newcomer_adopt` / `newcomer_pass` | `{newcomer_id, name?}` | Adoption happens in the tick (§7.10). |

Event types are **never** a metric dimension (§16.4).

### 6.5 Offers

**`POST /v1/aviary/offers`** takes `{offer_id, kind: "seed" | "song_fragment" | "still_pool", fragment_id?}` and returns `200 {placement, reactions: [{bird_id, reaction, start_ms, dur_ms}], until}`.

- Reactions are decided **synchronously on the server** from mood, curiosity, boldness, drowsiness, perch distance, and cooldown, using a seeded RNG. The call locks the relevant `bird_state` rows and is read-only on them: it records the decision in the event log, and the tick applies the consequences.
- Possible reactions:
  - Seed: `approach`, `wait_then_approach`, `watch`, `ignore`.
  - Song fragment: `join_in`, `go_quiet`, `call_against`.
  - Still pool: `drink`, `bathe`, `watch`.
- If an item of the same kind is still in the scene, the endpoint returns `409 ITEM_PRESENT`.
- The client's placement animation (0.6–1 s) masks the round trip. If there is no response within 1.5 s, the client plays a neutral `watch` and queues nothing. An offer that the server never decided produces no drift.

### 6.6 Birds, notebook, settings

| Method & path | Notes |
|---|---|
| `PATCH /v1/birds/{bird_id}` `{name}` | The only client-writable bird field. 1–24 characters, NFC-normalized, control characters stripped. Has no effect on personality, mood, or call. |
| `GET /v1/notebook?before=<cursor>&limit=20` | Newest first; scrolls back indefinitely. Each entry is rendered with the *current* bird names (§21 D9). No write endpoints exist. |
| `GET /v1/settings` · `PATCH /v1/settings` | Account-level preferences. Mute and volume are device-local and never sent. |

### 6.7 Visits

| Method & path | Notes |
|---|---|
| `POST /v1/visits/invitations` `{visitor_email}` | Sends a one-time link. Limits: 10 outstanding per host and 20 sends per day **(cal)**. |
| `GET /v1/visits/invitations` | Outstanding and active invitations. |
| `DELETE /v1/visits/invitations/{id}` | Revoke. Immediate on the server; the visitor client learns on its next pull. |
| `GET /v1/visits/log?before=` | Visitor email, date, approximate duration; most recent first. |
| `GET /visit/{token}` → `POST /v1/visit/accept` `{token}` | Scanner-safe consumption. Establishes a `__Host-aviary_visit` cookie with scope `aviary:read`. |
| `GET /v1/visit/snapshot` | Host snapshot minus `presence` and `newcomer` actions, with `offers_active` included. Returns `410 VISIT_UNAVAILABLE` when the invitation is revoked, expired, or suspended by host deletion. |

Every mutating endpoint returns `403` for visitor-scoped tokens, and ingestion additionally rejects any event whose session is not a host device session (I14).

### 6.8 Error codes (all rendered in the matter-of-fact register)

`LINK_INVALID`, `SESSION_ENDED`, `AVIARY_LOAD_FAILED`, `VISIT_UNAVAILABLE`, `RATE_LIMITED`, `ITEM_PRESENT` (rendered in the naturalist register inside the offer menu, because it concerns the scene: "a seed is already out"), `OFFLINE`, `UNSUPPORTED_BROWSER`.

---

## 7. Simulation engine

The engine is a pure function, `step(state, events, Δt, rng) → (state', journal, facts, intents)`, run by the tick workers. Purity gives us three things: deterministic replay for tests, idempotent retries, and a calibration harness that runs the exact production code on synthetic traces at thousands of simulated weeks per minute.

### 7.1 The tick

**7.1.1 Scheduling.**
- Aviaries are hashed by `aviary_id` into 4,096 virtual shards. Workers hold shard leases in Redis with a fencing token that increments on every handoff; lease TTL is 15 s, renewed every 5 s.
- Each aviary's tick phase is `hash(aviary_id) mod 60 s`, so load is flat across the minute.
- The cadence is 60 s **(cal)**. It is calibrated during the build against two things: mood-transition smoothness and database write load.

**7.1.2 Execution (one aviary):**
1. Load the aviary, birds, personality, bird state, active weather, and the pending-events cursor. Loading is batched per shard, 100 aviaries per query.
2. Read the events with `ingest_seq` in `(event_cursor, snapshot_of_max]`, in `ingest_seq` order.
3. Fold them into tick signals: credited presence minutes (the union across sessions), whether calls were perceivable, per-bird listen-in minutes intersected with presence, offer decisions, settles, greeting facts, newcomer decisions.
4. Run the sub-steps in fixed order:
   1. presence conditioning (§7.2);
   2. drift, meaning reservoir fill and release (§7.2);
   3. `recent_attention` (§7.4);
   4. mood (§7.3);
   5. social contagion and scheduling (§7.6);
   6. perch and intent planning (§7.5);
   7. weather (§7.7);
   8. newcomers, once per local day (§7.10);
   9. notebook detectors, hourly (§7.11).
5. Commit one transaction:
   - `UPDATE aviaries SET tick_seq = tick_seq + 1 … WHERE tick_seq = $expected AND lease_token = $fence`, plus the state rows, journal rows, facts, and `event_cursor`.
   - If zero rows match, abort. Either another worker has the lease, or this tick already ran. **Deltas can never be applied twice.**
6. Bump the snapshot ETag. The snapshot is built lazily on the next pull and cached per `tick_seq`.

**7.1.3 Determinism.** The RNG is counter-based (Philox), keyed by `(aviary.seed, tick_seq, substep)`. Re-running a tick yields identical output, so retries are safe and replay tests are exact.

**7.1.4 Catch-up after an outage.**
- All dynamics are defined in continuous time: hazards, first-order releases, exponential moving averages. A gap is integrated in steps of at most 5 min, so a 40-minute outage recovers correctly in 8 steps.
- Intents are planned only forward from "now," so recovery never produces a visible burst of flights.
- Presence that arrived during the outage is still credited, because it sits in the log.

**7.1.5 Latency target and alarm.** p50 compute under 20 ms per aviary. End-to-end tick lateness (scheduled → committed) p99 under 1 s; **the PRD alarm fires at p99 > 5 s.**

**7.1.6 Cost lever (not enabled at launch).** Because the math is continuous-time, an aviary with no connected client could tick every 5–15 minutes and produce statistically identical results. We keep the PRD's roughly-per-minute cadence for every aviary in v1. This lever is available only with product sign-off if database load demands it (§21 D11).

### 7.2 Personality drift

**Traits.** Each bird has five traits in [0, 1], stored as integer micro-units: **boldness** (B), **social warmth** (W), **vocal frequency** (V), **plumage saturation** (P), **curiosity** (C).
- *Seeds:* species baseline plus N(0, 0.05), clamped to [0.15, 0.45] **(cal)**. Every bird starts with headroom.
- *Ceilings:* each bird gets its own ceiling per trait at adoption, drawn from the species distribution and roughly U(0.70, 0.95) **(cal)**.
- *Why per-bird ceilings:* with drift that only rises, universal ceilings would make every long-lived aviary converge on identical, maximally expressive birds. Individuality, and with it recognizability, would erode over months. Per-bird ceilings mean each bird grows into its *own* expressive self.

**Structure: a two-stage, monotonic low-pass filter.**

```
input conditioning        reservoir (pending drift)            trait
u_i(t) ≥ 0  ──fill──▶   R_i += g_i · u_i          ──release──▶  x_i += q · h(x_i, c_i)
                          q = R_i · (1 − e^(−Δt/τ_R));  R_i −= q
```

- **Input conditioning (saturating per day).** Per tick, effective presence is `u_p = p_tick · exp(−m24 / S)`. Here `p_tick` is the credited presence minutes in the tick, `m24` is the rolling 24 h of credited presence, and `S = 20 min` **(cal)**. The first minutes of a day count most. An eight-hour "marathon" contributes only about 2.2× a typical 12-minute visit, so no user can move a bird by leaving the tab focused and wiggling the mouse. This is the PRD's "too fast becomes a Tamagotchi" guard.
- **Reservoir and lag.** Inputs fill a per-trait reservoir `R_i`, which drains into the trait with time constant `τ_R = 3 days` **(cal)**. This is what makes drift a true low-pass filter:
  - A session's effect is spread over roughly a week, so no session is ever visible on its own.
  - Drift *continues during absence* from inputs gathered before the user left. This is exactly the PRD's "personality drifts during the user's absence based on inputs from before they left, not on inputs invented at the moment they return."
- **Headroom.** `h(x, c) = max(0, (c − x) / (c − x_seed))`. Growth slows as a trait nears its ceiling and stops at it.
- **Monotonicity.** Every `u ≥ 0`, `g ≥ 0`, and `q ≥ 0`, so no path can lower a trait (I2). The database trigger is the backstop.

**Input weights `g` (per unit of conditioned input) (cal):**

| Input | B | W | V | P | C | Notes |
|---|---|---|---|---|---|---|
| Presence (aviary-wide, to every bird) | 1.0 | 1.0 | 0.6 → 1.0 | 1.0 | 0.6 | V uses 1.0 while calls are perceivable (audio on **or** captions on) and 0.6 while muted with captions off. Muting never subtracts; captions count as listening, so deaf and caption-only users are never disadvantaged (§21 D4). |
| Listen-in on bird *b* (minutes ∩ presence, saturating with `S_L = 10 min`) | — | 1.5 | 1.5 | — | — | The strong per-bird attention signal. |
| Offer accepted by *b* | — | — | — | — | +a_acc | A small curiosity step, `a_acc` ≈ 0.1 of a regular week's curiosity input. |
| Offer placed near *b* (same or adjacent zone) | +a_near | — | — | — | — | A small boldness step, `a_near` ≈ 0.05 of a regular week's boldness input. |
| Song fragment `join_in` / pool `bathe` | — | — | +small | +small | — | Secondary, calibration-level. |
| Settle | — | — | — | — | — | No drift. It closes the presence window and quiets mood. |

**Per-bird offer cooldown.** 4 min **(cal)** starting from the bird's reaction, plus a cap of 5 counted offers per bird per day. Without these, curiosity drift would saturate within one session and the engine would collapse. With them, an offer reads as a gesture.

**Calibration targets.** These are asserted by the harness in §7.12. A worked example with the starting constants: a regular user has 5 sessions a week of about 12 minutes each, headroom ≈ 0.55, and gain `k = 0.032`.
- **Measurable at ~1 week:** each presence-weighted trait moves 0.02–0.05. Instruments see it; users do not.
- **Visible at ~3 weeks:** the primary expressive traits cross the *visibility threshold* `V_vis ≈ 0.10–0.12`. The threshold is defined perceptually, not numerically, as any of:
  - front-perch occupancy shifts ≥ 15 percentage points;
  - unobserved call rate changes ≥ 20 %;
  - plumage rendered color shifts ΔE2000 ≥ 3;
  - greeting-first probability shifts ≥ 15 percentage points.

  The exact threshold is fixed by a perception study (§17.2).
- **No single session is visible:** the maximum 24 h contribution stays below `0.25 · V_vis`, even for a marathon session.
- **Devoted users** (two 30-minute visits a day) cross `V_vis` in about 10–14 days. That is faster, but still weeks.
- **Long run:** about 50 % of headroom is consumed after ~4 months of regular use, and it approaches the ceiling asymptotically over a year. Birds keep changing slowly and stay distinct.

**Versioning.** `drift_fn_version` is stored per bird. A new version applies only from the tick it ships and never re-processes history (I4). Every version change needs harness sign-off (§18.5).

### 7.3 Mood

**States (finalized for v1):** `alert`, `curious`, `content`, `wary`, `drowsy`, `settled`.
- `settled` is the night-rest state: eyes closed, low on the perch. It is distinct from the session-level *settled lighting* that the settle gesture produces (§10.9), and the code keeps those two names in separate enums.
- Mood is the enum plus an intensity in 0..1. Intensity drives how strongly the mood shapes idle motion and calls.

**Transitions.** A per-tick hazard model: for each candidate state *j*, `P(i→j) = 1 − exp(−λ_ij · Δt)`, with `λ_ij = base_ij × Π modifiers`. A bird must stay in a state for a minimum dwell of 10–25 min **(cal)**, and hysteresis on intensity prevents flicker. The modifiers are:

- **Time of day** (host-local; §7.8): pre-dawn hush, then early morning favoring `alert`; midday favoring `content`/`curious`; dusk favoring `drowsy`; night favoring `settled`. The nightjar-like species has its own curve and becomes more active at dusk and into the night.
- **Recent interactions:** an accepted offer or a listen-in nudges toward `content`; a song fragment nudges toward `curious`; a settle raises the `drowsy`/`content` hazards and lowers intensity.
- **Ambient events:**
  - Rain damps call rate and loudness for its duration plus ~15 min; it is an expression change, not a mood.
  - Wind raises `alert` for bolder or more curious birds and `wary` for less bold ones, briefly.
  - Another bird's alarm call raises `wary` for nearby birds.
- **Personality:** high boldness scales `→wary` hazards down; curiosity scales `→curious` up; warmth raises susceptibility to *positive* contagion.

**Daily-ish reset.** At host-local dawn, each bird's mood is re-anchored toward a morning distribution conditioned on its personality and on how it ended the night. The pull is soft, and waking is staggered across birds over 20–40 min. Birds are rarely watched at dawn; if they are, the user sees birds waking one by one, never a snap.

**Persistence.** Mood is canonical server state. Opening a tab never resets it (the client has no default mood), so a session starts in whatever mood the tick has carried forward.

**First-encounter exception.** For the first 60 minutes after adoption, starter birds are held in `curious`/`alert` regardless of the hour. A first session at night meets awake birds rather than a dark scene.

**Absence is not a mood input.** No term involving time since the last visit touches mood valence. Wary comes from wind, alarm calls, and personality, never from the user having been away (I2).

### 7.4 Recent attention: quieter, not mistrustful

Each bird has a non-personality scalar, `recent_attention ∈ [0.25, 1]`.
- It rises with credited presence (time constant about one typical session) and, for the focused bird, with listen-in.
- Without presence it decays toward the **ambient floor** of 0.25, with a half-life of 5 days **(cal)**.
- It scales only *expression amplitude*: greeting propensity, calls directed at the viewer, front-perch approach frequency, response to the viewer's cursor. It never touches traits, never touches mood valence, and never feeds drift.

This is how a bird that has been ignored "becomes ambient: still alive, still calling, but greeting less often because less often is what's been observed." After two weeks away, the birds are quieter than they were. Their traits are, if anything, slightly higher, because the reservoir kept releasing. Their greeting is a *re-orientation* (§7.9), and within a session or two, attention recovers and their full expressiveness returns. The returning user gets something to ease back into instead of a guilt surface.

### 7.5 Perch choice and intents

- **Layout of perches:** three zones, `front`, `middle`, and `back`, with 3 slots each (9 slots for at most 7 birds). Each zone also has low "roost" slots used by `settled` birds at night.
- **How a bird chooses a zone:** a utility score of the form `boldness·front_bias + mood_term + social_term + recent_attention·front_bias + noise`.
  - Wary birds sit back, bold birds come forward, and drowsy birds take low perches.
  - Warm birds perch near other birds.
- **How often birds move:** dwell times are 3–15 min **(cal)**, so the scene stays calm.
- **Intents published per tick:** flights and hops, plus call-response and alarm-call events, each with a server timestamp. Each tick publishes the next ~3 minutes, overlapping the previous tick's window, so a client that misses one pull still has a continuous timeline.
- **What the user cannot do:** there is no API or UI for placing birds (I10). The perch is a signal the user reads, not a layout the user controls.

### 7.6 Bird-to-bird interaction

The server models the social layer coarsely, and the client realizes it finely.
- **Mood contagion:** each tick, a bird's `wary` intensity spreads to birds in the same or an adjacent zone. Spread is weighted by the receiver's (1 − boldness), and calm birds damp it. That is how "a wary mood in one bird tends to spread."
- **Call-response:** when a bird is scheduled to call, a warm neighbor is scheduled to answer with probability proportional to its warmth and vocal frequency. The answer comes after a 0.4–1.5 s latency, which the client realizes.
- **Chorus windows:** when two or more birds with high vocal-frequency expression are in `alert`/`content`/`curious` at the same time, the tick opens a chorus window of 8–30 s. During it, the client raises call density and allows overlaps. Outside chorus windows, the client scheduler biases birds toward turn-taking, the way real birds avoid masking each other.

### 7.7 Weather

- **Frequency:** weather is a per-aviary Poisson process on `weather_seed`. Rain averages 3 per week, lasting 6–20 min each; soft wind 4–7 per week, lasting 3–10 min **(cal)**.
- **What never happens:** no thunderstorms, no snow, no user-facing weather state. Weather is not linked to real-world weather: that would need location, and it would turn weather into a feature.
- **Planning:** weather is planned a day ahead and published in snapshots, so the host's devices and any visitors see the same rain.
- **Effects are small and short-lived:** rain damps call rate and loudness; wind briefly shifts mood hazards (§7.3).

### 7.8 Day/night and the timezone model

- **Which clock:** the aviary follows the *host's* local time. The timezone is the IANA zone reported by the most recently active host session. When a report differs from the stored zone (travel), the new zone takes effect after it has been reported twice, 10 min apart. The light model then eases to the new local time over 30 min, and DST jumps are eased the same way.
- **Sunrise and sunset without location:** we take the latitude and longitude of the IANA zone's principal city from `zone.tab` and compute approximate solar elevation. That gives seasonal sunrise and sunset and correct hemispheres without ever collecting the user's location.
- **Phases:** pre-dawn, dawn, morning, midday, afternoon, dusk, evening, night, derived from solar elevation with smooth blends. Night is not a dead state:
  - The nightjar-like species stays active and may call late.
  - Settled birds still breathe, shift, and occasionally murmur.
  - Ambient motion continues.

### 7.9 Greeting planning (server side of the return-greeting)

The server supplies per-bird `greet.propensity` and `greet.style`. The client decides the moment (§10.6).
- **`propensity`** is a function of boldness, warmth, mood, and `recent_attention`. It sets both the order and whether a bird greets at all: the bolder bird greets first, and the warier bird greets later or not at all on a given day.
- **`style`** is a stable per-bird greeting character, derived from species, signature, and boldness. The same bird greets the same *way* across visits, while different birds greet differently. Procedural variation happens inside the style.
- **`last_present_at`** comes from `session_open`/`session_close` and the presence data across all devices. From it the client computes the **absence class**. The class thresholds are **(cal)**:

| Absence | Greeting shape |
|---|---|
| < 30 s (tab flick) | none, or at most an eye-flick from the most attentive bird |
| 30 s – 10 min | a glance up from whatever the bird is doing |
| 10 min – 6 h | a glance plus a quiet short call |
| 6 h – 2 days | a head-tilt and a hop or step toward the front, then a call; a second bird may follow |
| > 2 days | a re-orientation: the greeter comes closer and calls longer, and another bird answers |

- **Why a bird never flies off-screen:** every greeting motion happens within the scene. A greeting "step toward the front" is a local, temporary hop that returns to the canonical slot. It is realized per device and never written back.
- **Why the history is kept:** `greeting_recent` (the last 8 realized signature hashes) is returned in the snapshot. The client rejects candidate realizations too close to recent ones, across devices, so greetings are never identical twice.

### 7.10 Adoption and newcomers

**Starter selection.** When an account is created, the engine picks two species from the pool, chosen to be complementary: different silhouettes and separated call registers. The nightjar-like species is eligible as a starter with probability 1/3 **(cal)**. The engine also:
- seeds each bird's traits, ceilings, and **call signature**;
- chooses the signature to be maximally distinct from the aviary's other birds, with pitch centers at least 3 semitones apart and a different signature motif (§11.4).

The user does not pick from a catalog. The birds are presented as the ones that arrived (§10.10).

**Newcomers (a third bird and beyond).**
- **Eligibility:** eligible slots = `min(7, age_slots(aviary_age), max_birds_enabled) − current_birds`.
  - `age_slots` steps up at days 90, 165, 240, 330, and 450 **(cal)**, each with ±10 days of per-aviary jitter so arrival never feels scheduled.
  - `max_birds_enabled` is an operational safety ceiling for the rollout (§18.3). It is never per user.
  - **Aviary age is the only per-aviary input.** Visit counts, interaction scores, and payment are not inputs. The scheduler's input type does not contain them.
- **Arrival:** when a slot opens, a *newcomer* begins appearing on the back perch on some days. It is a species not currently present when possible, never framed as rare, and it has its own signature. It is not announced. The notebook will likely notice it ("a small brown bird with a pale eyebrow has been on the back perch two mornings running").
- **Adoption:** the user adopts the newcomer from the offer menu with **"offer the newcomer a perch"** and names it (default suggestion provided). "Let it pass" is available without penalty, and another newcomer comes after 45 days **(cal)**. Doing nothing has no consequence either; the newcomer keeps visiting occasionally.
- **Limits:** only one newcomer is pending at a time, and the cap of 7 is absolute.

### 7.11 Notebook writer

The notebook writer runs hourly per aviary inside the tick, reading `aviary_facts`, species, and names only.

1. **Detectors** turn facts into candidate observations. Examples:
   - greeting-order novelty ("pip greeted before wren today, first time this week");
   - first front-perch visit in a long while;
   - a rain moment;
   - a long quiet stretch or preening bout;
   - a dusk chorus;
   - the nightjar calling late;
   - a newcomer;
   - adoption day;
   - long-horizon change ("wren's breast looks warmer in the morning light than it did last month"). This one fires at most once per bird per 6 weeks, only after a visibility threshold is crossed, and is phrased tentatively. The notebook may *notice* change the way a watcher would, but never quantifies it.
2. **Salience** = novelty × specificity × rarity, relative to the aviary's own recent history.
3. **Sparsity governor:**
   - An entry is written only if salience ≥ a threshold, and the threshold rises with the number of entries in the last 7 days.
   - Target about one entry every 2–4 days for a regularly watched aviary.
   - Hard limits: one entry per local day and a minimum gap of 20 h. The only exceptions are the adoption-day entry and a newcomer's first appearance.
   - Very active users do not get more entries: detector inputs are aviary facts, not session counts.
4. **Realizer:** a grammar-based realizer built from writer-authored templates (§12.2). The output is lowercase, present tense, bird-named, and specific, and it goes through the voice lint before storage. Templates used for an aviary in the last 60 days are excluded, so the prose never becomes recognizable machinery.
5. **Storage:** entries are immutable, tokenized (bird references by id), dated with the host-local day, and never archived or hidden.

**What the writer cannot see:** presence, sessions, visits, or anything about the user's behavior. "The user visited every day this week" is unwritable because the data is not there (I16).

### 7.12 Calibration harness

- **What it is:** a CLI and CI job, `aviary-harness`, that runs the production `step()` over synthetic event traces with an accelerated clock and produces trajectories plus pass/fail assertions.
- **Personas:**

| Persona | Behavior |
|---|---|
| regular | 5×/week, 8–15 min |
| devoted | 2×/day, 20–40 min |
| weekend | 2×/week |
| marathon | 8 h focused, activity every 3 min |
| background-tab | 48 h hidden |
| minimized | visible but unfocused |
| two-week absence | after 4 weeks regular |
| offer-spammer | 50 offers per session |
| listener | 10 min listen-in on one bird daily |
| multi-device overlap | laptop and phone at the same time |
| muted-with-captions / muted-without | audio variants of regular |
| year-long regular | 52 weeks, 7 birds |

- **Assertions:**
  - **A1:** measurable drift at 7 d.
  - **A2:** `V_vis` crossed at 18–26 d for regular.
  - **A3:** no 24 h contribution ≥ 0.25 `V_vis`.
  - **A4:** no trait ever decreases (property test over 10⁵ random traces).
  - **A5:** absence produces zero trait decrease and no increase in wary occupancy beyond ±2 percentage points of baseline.
  - **A6:** background-tab and minimized produce zero drift.
  - **A7:** multi-device overlap equals single-device drift for the same wall-clock time.
  - **A8:** offer-spammer curiosity is bounded by cooldown and the daily cap.
  - **A9:** after one year, pairwise trait distance between birds is ≥ δ, and no bird sits at every ceiling.
  - **A10:** call signatures and ceilings never change.
  - **A11:** caption-only users are within 1 % of audible users on V drift.
- **Where it runs:** nightly in CI and on every change to `step()`. Plots are published to the engineering review page. This harness, together with consented dogfooding, is the **only** place drift behavior is measured (I12).

---

## 8. Sync model

### 8.1 The shape

Each account has **one canonical record**. The tick is its **single writer** for personality, mood, perch, and intents. **Every client is a reader plus an event appender.** There is no client-to-client sync, no client-side state to merge, and nothing to reconcile, because every device reads the same record. Multi-device sync is therefore not a feature we build; it is a property of this shape.

### 8.2 Write paths and their conflict semantics

| Write | Who | Semantics |
|---|---|---|
| Interaction events | Host device sessions | Append-only. The server assigns `ingest_seq`, which is the canonical order. Idempotent by `event_id`. |
| Offer decisions | `offers` module | Serialized per bird with `SELECT … FOR UPDATE` on `bird_state`. The cooldown decides races, so two devices offering at the same moment get one reaction and one "watch." |
| Personality, mood, perch, intents | Tick only | Additive, non-negative deltas computed from the log. A compare-and-set on `tick_seq` plus the lease fencing token make double application impossible. |
| Bird name | Any host device | Last write wins, which is fine: a name is not personality, and renaming changes nothing else. |
| Settings | Any host device | Per-field last write wins with `updated_at`. |
| Invitations | Host | Revocation always wins over acceptance, by a status check in the same transaction. |

### 8.3 Failure scenarios and why each one is harmless

- **Laptop and phone both open.** Presence is credited as the union of intervals, so drift equals one device's worth. Both devices render the same snapshot.
- **Morning laptop session, then a lunch phone session that started earlier and flushes its events late.** The events are additive and consumed in ingest order. Nothing overwrites anything. This is the PRD's lost-drift scenario, and it is unreachable here.
- **A tick worker dies mid-transaction.** The transaction rolls back and the next lease holder re-runs the tick deterministically.
- **Two workers believe they own a shard (split brain).** The stale fencing token fails the compare-and-set.
- **Duplicate or replayed events.** Rejected by the unique `(aviary_id, event_id)`.
- **Client clock skew.** All scheduling uses server time via an estimated offset. Presence intervals are bounded by server receipt time and by the elapsed monotonic time between pings.
- **Region failover.** Synchronous replication gives an RPO of about zero for personality. If a PITR restore is ever needed, the journal (the recorded deltas, not the events) re-applies ticks committed after the restore point. Then `deletion_ledger` is replayed.
- **Migration or refactor.** A migration lint blocks writes to trait columns. Checksum audits run before and after, and a failed audit blocks the deploy.

### 8.4 How clients consume state

1. **Clock.** Each snapshot response carries `server_time`. The client keeps an EWMA of the offset, taking samples only from low-RTT responses. All intents and scheduled events are in server time.
2. **Timeline.** The client builds a timeline from snapshot `N`: current state, then intents up to `valid_until`. Snapshot `N+1` extends and supersedes it. Because the windows overlap, one missed pull causes no discontinuity.
3. **Reconciliation, never teleporting.** When the rendered state disagrees with the snapshot at the current moment:
   - different zone → schedule a natural flight within a few seconds;
   - same zone, different slot → a hop;
   - different mood → blend the expression parameters over 20–60 s;
   - different plumage level → ease over minutes.
   This is how "a bird at perch A in snapshot N and perch B in N+1 is rendered moving smoothly between them."
4. **Long gaps** (suspend/resume longer than 60 s, bfcache restore). The frame the user last saw is stale by minutes or hours, so the client treats the moment as a fresh open: it rebuilds the scene from a fresh snapshot, with birds already where they are and mid-action, and the return-greeting fires. Flying every bird across the scene would read as a reset.
5. **Device-local by design (never synced):** settled lighting, the listen-in focus and mix, open panels, mute and volume, and the anti-repetition memory for greetings and calls. Each of these is a property of *this viewing*, not of the aviary.

### 8.5 Multiple tabs on one device

Tabs coordinate over `BroadcastChannel`. The visible, focused tab becomes the **audio leader** and the others fade their calls out, so two tabs never double the chorus. Presence is naturally exclusive, because only one tab can be focused, and the server-side union covers edge cases.

### 8.6 Offline and degraded modes

- **The snapshot pull fails but the session is valid.** Birds continue on the last timeline, then on local idle motion with no zone changes, and calls continue at the last parameters. There is no error surface while the scene is fine.
- **After 2 min of failed pulls,** a quiet matter-of-fact status line appears inside the top bar: "Offline. Your aviary will catch up when you reconnect." It fades with the bar.
- **While offline, offers are disabled** in the menu, with the note "Offers need a connection." Events queue in IndexedDB, bounded at 500, and flush on reconnect. Presence intervals queued more than 2 h ago are dropped.
- **The initial load fails.** The quiet field holds for up to 10 s, then the error appears in the top-bar region: "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."
- **The refresh token is rejected.** "Your session timed out. Sign in again to keep watching." The scene keeps rendering behind the sign-in panel.

---

## 9. Presence accounting

### 9.1 The client `PresenceMonitor` (host client only; compiled out of the visitor build)

It tracks three signals independently.

1. **Visible:** `document.visibilityState`, from the `visibilitychange` event.
2. **Focused:** `document.hasFocus()`, reconciled on `focus`/`blur` of the window and re-checked every 5 s. Some browsers drop blur events on OS-level switches, so the check does not rely on events alone.
3. **Recently active:** `lastActivity` is updated by:
   - `pointermove` from any pointer type;
   - `keydown`, the modern equivalent of `keypress`;
   - `pointerdown` on touch pens and fingers, because a tap with zero movement is still unmistakably someone at the device;
   - keyboard-driven `focusin`, because screen-reader navigation often never delivers raw key events to the page (§21 D3).

   The signal holds while `now − lastActivity ≤ W`, with **W = 4 min (cal, tested 3–6 min, biased long)**. Watching birds without moving is the product itself, so presence is lost only after a real stretch with no sign of anyone there.

**Presence holds iff all three signals hold.** No signal on its own can produce it.

**Additional rules:**
- Presence also requires the aviary scene to be at least partly on screen. A full-screen settings sheet on a phone pauses it.
- Presence stops at settle and resumes only on active re-engagement.
- While presence holds, the monitor emits `presence{from, to}` every 30 s. It closes the open interval immediately when any signal drops. On `visibilitychange→hidden` or `pagehide`, the final interval goes out by `sendBeacon`.
- If the final beacon is lost, the server closes the window at the last interval's end. **Presence is never extended from missing data.**

### 9.2 Server crediting

- **Validation per interval:** the duration is ≤ 35 s; the interval does not overlap the same session's previous interval; it sits within the skew bounds; the session is an active host device session.
- **Crediting per aviary:** merge the valid intervals into a union, then credit `min(union, wall-clock)` into `presence_ledger`. That credited time is the input to conditioning (§7.2) and to `recent_attention` (§7.4).
- **Fail closed:** malformed, overlapping, late (more than 2 h), or visitor-scoped intervals are dropped. Each drop increments an *error counter*, never a behavior metric.
- **Settle and tab-close are equivalent at the engine level.** Both end the window. Neither carries a penalty, and no "you didn't settle" state exists anywhere.

### 9.3 What presence is not

Presence is attention, not engagement. The simulation does not care about click counts, and clicks outside the three-signal conjunction earn nothing. Presence-time is never shown, exported, summarized, or aggregated across accounts. It exists only to drive that user's own birds.

---

## 10. Frontend rendering pipeline

### 10.1 Boot sequence: the first-bird path

Target: the first bird is visible in under 500 ms from navigation on the reference device and network (§16.1).

1. **Edge HTML, streamed.**
   - The first flush, about 6 KB, carries inline critical CSS, including the sky gradient computed at the edge for the host's local time, so even the first paint is the right sky. It also carries a `modulepreload` for `scene-core.js` (≤ 60 KB gz), which is also hinted by `103 Early Hints`.
   - The second flush adds the snapshot as `<script type="application/json" id="snap">`, which is non-executable and CSP-safe, plus the species art for this aviary: compact SVG part paths, 3–8 KB per species.
2. **`scene-core` boots.**
   - It parses the snapshot and computes the clock offset.
   - It builds the scene at *now*: birds at their perches in their current mood, with the timeline cued.
   - It **warm-starts** every behavior controller by simulating 3–6 s of virtual time in a tight loop (about 1 ms), so birds are mid-preen, mid-scan, or mid-shuffle and particles are mid-fall.
   - It then draws. **There is no entry animation, no fade from static, and no "ready" state.** The first frame is the aviary, which has been there.
3. **After the first bird, in priority order:**
   - the audio engine, target audible within about 1 s when autoplay allows (§11.7);
   - the greeting runner;
   - the presence monitor;
   - the narration composer;
   - the top-bar chrome.

   Settings, account, visits, and the notebook are lazy chunks, loaded on interaction or idle.
4. **Returning users.**
   - A Service Worker serves the shell and hashed assets from cache, stale-while-revalidate.
   - The snapshot is always fetched fresh. A cached snapshot is used only if it is younger than its `valid_until`, which is at most about 3 min, because the timeline covers exactly that span. Anything older would risk a visible correction.
   - The Service Worker is used only for caching. It never registers push and never prompts for install.
5. **The snapshot is late** (cold cache, slow link). The **quiet field** shows instead: the local-time sky, faint foliage, and one or two slow motion cues such as a drifting leaf or slow cloud movement. **Never a spinner.**
   - If the snapshot arrives within 150 ms, birds are simply drawn.
   - If it arrives later, the bird layer rises in opacity over 700 ms with the birds already mid-behavior, never animating *into* a pose. This reads as the aviary catching up.
6. **Unsupported browsers.** A 1 KB inline feature check covers ES2022 modules, `Intl` time zones, `structuredClone`, and `CSS.supports` for the features we use. Browsers that fail get a static, matter-of-fact page: "Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge. Update your browser or open this page in one of them." Missing WebAudio does **not** count as unsupported; it has its own fallback (§11.10).

### 10.2 Scene composition

The scene is layered back to front. Each layer redraws only as often as it needs to.

| # | Layer | Tech | Redraw |
|---|---|---|---|
| 1 | Sky (gradient, sun/moon glow) | CSS gradient `div` | Every 10 s via CSS transition |
| 2 | Far foliage | Canvas, procedurally generated from `scene_seed` at the viewport size | On resize; subtle drift via CSS transform |
| 3 | Perch plane (branches and rails for three zones, roosts) | Canvas | On resize |
| 4 | Birds and offered items (seed, still pool with a cheap flipped-alpha reflection) | Canvas, per frame | 60 fps |
| 5 | Ambient particles (leaves, feathers, rain streaks) | Canvas, per frame, pooled | 60 fps while active |
| 6 | Occasional foreground branch or leaf pass | Canvas | Only while a pass is active |
| 7 | Light grade (time of day, weather dim, settled evening) | CSS `div` with a blend mode | Transitions |
| 8 | Accessibility overlay (bird focus targets, focus ring, captions) | DOM | Positions synced at ≤ 30 Hz |
| 9 | Top bar | DOM (Preact) | Event-driven |

- **Depth:** parallax between layers 2, 4, and 6 comes from slow wind-driven drift of a few CSS pixels. It is never pointer-driven and never panning. The scene is not a layered illustration showing off.
- **Canvas size:** backing-store resolution is capped at DPR 2. Far layers render at 0.75× and are upscaled.
- **Palette:** calm naturalist design-system tokens (soft blues, greens, warm browns, muted ochres). The lint rejects saturated accent colors in scene code.

### 10.3 Layout solver: responsive, and no bird is ever cropped

- **Coordinate space:** the scene is authored in a normalized space with perch-slot anchors per zone.
- **Per-viewport solve:** for each viewport, the solver computes:
  - a horizontal compression for narrow viewports, which pulls slots inward non-uniformly and scales birds down to a minimum on-screen size, so the smallest bird stays recognizable and at least a 44×44 CSS px target;
  - wider perch spacing for wide viewports, with the extra room given to procedural foliage and sky;
  - a bird-free top band the height of the top bar;
  - respect for `env(safe-area-inset-*)`.
- **Invariant:** the bounding box of every bird at every slot, including greeting hops and bathing splash extents, stays inside the viewport. A viewport-matrix visual test asserts this from 320×568 to 3840×1600, in portrait and landscape.
- **What the solver never does:** scroll, pan, or zoom. The aviary always fits at a glance.

### 10.4 The motion system: idle micro-motion

Each bird is a 2D rig of roughly 12 parts (body, head, beak, eye and lid, two wings, tail, legs, fluff overlay) with joint limits. Motion is layered.

1. **Base layer.**
   - Breathing: a noise-modulated sinusoid, with rate and depth set by mood.
   - Balance micro-sway, blinks, and feather fluff (a scalar per mood: drowsy birds fluffed, wary birds sleek).
2. **Behavior layer.** Utility-based selection among parametric behaviors, not keyframed clips: preen (inverse kinematics to a randomly sampled wing or breast target, with a random stroke count), scan, head-tilt, body shuffle, beak-wipe, stretch, hop along the perch, look at a neighbor.
   - *Mood shapes the utility weights:*
     - a wary bird scans more and sits sleek and tall;
     - a content bird preens;
     - a curious bird tilts toward sounds and watches passing leaves;
     - a drowsy bird sits low with feathers fluffed and eyes half-lidded;
     - an alert bird makes quick head turns;
     - a settled bird has its head tucked and eyes closed.
   - *Expression parameters* (`tempo`, `motion_amp`, `scan`, `preen`, `tilt`) shape durations and amplitudes, so bold birds make bigger and nearer movements.
3. **Reactive layer.** Head-tilts toward sound sources: another bird's call, the song fragment, rain onset. Glances at leaves passing near the bird.
4. **Social and vocal layer.** The beak, throat, and body pulse are driven by the *actual amplitude envelope* of the synthesized call, using shared timestamps from the audio engine (§11.5). Birds visibly produce what the user hears.

**Nothing loops.** Every behavior samples its parameters fresh from a per-session seeded RNG. Continuous noise fields are sampled on a server-anchored time base, so a fresh open samples mid-cycle. No bird is ever still in a way that reads as paused. Flights are parametric arcs with procedural wingbeat counts, glide segments, and a landing flutter.

**Rendering stops when the tab is hidden.** Simulation continues on the server, and on return the scene resumes from the fresh snapshot (§8.4).

### 10.5 Input model in the scene

- **Clicking or tapping a bird** starts listen-in on it. Clicking the focused bird again disengages. Clicking another bird switches to that bird. Clicking empty space disengages.
- **Hit testing** uses the DOM overlay targets from layer 8, which serve both pointer and keyboard.
- **No hover affordances:** no tooltips, no cursor-follow effects. The scene carries no chrome.
- **Listen-in visuals are restrained.** The focused bird may glance toward the viewer, depending on its warmth. It gets no glow and no label.

### 10.6 Greeting realization (client)

- **Triggers:**
  - fresh navigation;
  - `visibilitychange→visible`;
  - bfcache `pageshow`;
  - resume after a long gap.

  Window `focus` alone produces at most the tab-flick response.
- **Choosing who greets:** sample the greeter by `greet.propensity` with randomness, so the bolder bird usually greets first but not always.
- **When:** the greeter begins **0.6–1.8 s** after the first visible frame, at a random point in that window, so it is never on cue.
- **How it looks:** compose the realization from the bird's `style` and the absence class (§7.9):
  - motion primitives: look-up, tilt, hop toward the front, wing flick;
  - a call built by the grammar with the greeting motif family;
  - randomized amplitude, timing, and interval choices.
- **Anti-repetition:** reject a candidate whose signature hash is too close to any in `greeting_recent` or the local history.
- **Following birds:** further birds greet only if their propensity clears a threshold. They are **staggered** by random offsets of 0.8–3 s and never fire in unison, because a synchronized chorus on cue would announce the user's arrival.
- **After the greeting:** report `greeting_rendered`. The first frame of the greeting also triggers the prioritized narration line (§13.2).
- **Never:** any text, toast, banner, "welcome back," or absence-length surface. The birds are the welcome.

### 10.7 Offers (presentation)

- **Entry points:** the top-bar offer affordance, or the `o` shortcut, opens a small menu with **"offer a seed," "offer a song fragment" ▸ (8-item library), "offer a still pool,"** and, when a newcomer is visiting, **"offer the newcomer a perch."**
- **What each offer looks like:**
  - *Seed:* falls softly to a front-perch spot.
  - *Song fragment:* plays softly in a distinct, non-bird "offered" timbre (§11.6).
  - *Still pool:* settles into the front of the scene as a soft reflective surface.

  The server's reactions (§6.5) then play out: a curious, content bird approaches; a wary bird waits, then comes; a drowsy bird may not come at all; for the song fragment, a bird joins in, goes quiet, or calls against it; at the pool, birds drink, bathe, or watch.
- **How long items stay:** they persist (seed about 3 min, pool about 5 min) and then fade.
- **What limits repeats:** while an item is present, its menu entry is disabled with naturalist text ("a seed is already out"). That natural limit replaces any visible cooldown. **Timers and counters are never shown.**
- **Seeds are gestures, not food.** There is no feeding state.

### 10.8 Top bar

- **Placement:** a thin bar above the aviary scene proper, overlaying the bird-free sky band.
- **Contents:** **account/settings, accessibility settings, Field Notebook, offer, and settle.** Settle is here because two PRD sections require it to be reachable from the top bar (§21 D1). Nothing else is allowed: no badges, dots, counts, or status indicators. The one exception is the matter-of-fact soft-delete strip (§14.5) and the offline line (§8.6), both system surfaces.
- **Fade:**
  - After 3.5 s **(cal)** of cursor stillness, the bar fades to 12 % opacity over 1.2 s.
  - It returns on pointer movement, keyboard activity, or a touch on empty scene space.
  - It never fades while focus is inside it or a menu is open.
  - Faded controls stay fully keyboard reachable and become fully opaque when focused.
- **Reduced-motion variant:** the same opacity change over a slower duration, with no movement.

### 10.9 Settle (presentation)

- **Triggering settle** from the top bar:
  - the grade layer shifts to evening over about 4 s;
  - call rate and loudness fall;
  - one bird acknowledges the goodbye with a glance and a soft low call, procedurally varied like the greeting.

  No text appears.
- **Undo:** any click or tap in the aviary within **5 s** reverses the shift. The `settle` event is sent only after the undo window passes.
- **After the window:** the tab stays *settled* until it closes or the user actively re-engages with a click or key press. A pointer twitch does not count, so a nudge of the mouse doesn't cancel a goodbye. Re-engagement brings the light back to local time over about 3 s and resumes presence.
- **Scope:** settled lighting is local to the device and session (§21 D7). Its canonical footprint is only the mood-quieting input and the end of the presence window.

### 10.10 Adoption and the empty-aviary state

1. **After the first magic-link consumption,** a single adoption step, in the naturalist register, reads "two birds have arrived." It shows each bird with a one-line naturalist description (for example, "a small grey bird with a pale eyebrow and a quick, rising call") and a prefilled suggested name the user can edit. This step is also the first user gesture, which unlocks audio.
   - There is no catalog and no species choice. The user is meeting the birds, not configuring avatars.
2. **"let them in"** shows the **empty aviary**, which is the same quiet field. The first bird then makes a **soft fly-in** to its starting perch, and the second follows after a 2–5 s stagger. From then on the user never sees an empty aviary again: no state removes all birds, not even at night.

### 10.11 Day/night and weather rendering

- **Light:** the sky and grade transition continuously from solar elevation (§7.8): sunrise warms the palette, midday is brightest, evening warms and quiets, and night dims most of the scene with moonlit accents.
- **Rain:** pooled streak particles plus a dimmer grade, a soft sheen on the perches, and birds' fluff and shelter behaviors.
- **Wind:** a foliage ripple through layer drift, and leaves released in bursts.

Weather never uses flashes, and nothing in the scene flashes more than three times a second (WCAG 2.3.1).

### 10.12 Reduced-motion rendering

The reduced-motion mode is a designed surface, not "animations off." It activates when `prefers-reduced-motion: reduce` is set, or when the user sets Accessibility → Motion to "Reduced." Users can also force "Standard."

- **Micro-motion** becomes slow cross-fades (1.5–3 s) between designed **still poses** from each species' pose library, one pose change every 6–15 s per bird, chosen by the same mood-shaped behavior selection. A preening bird becomes a sequence of preen poses.
- **Flights** become cross-fades between perches. Greeting hops and offer approaches become pose cross-fades plus a call.
- **Removed:** ambient leaf and feather drift, foreground passes, parallax drift, and the rain particle motion. Rain is shown as a still sheen plus dimming.
- **Retained, slowed:** color and light shifts from day to evening, and the settled-evening grade.
- **Unchanged:** calls at full quality (or captions), drift, mood, and notebook.

Every new animation must ship with its reduced-motion variant. The motion system refuses to register a behavior without one; a runtime assertion enforces this in development and CI.

### 10.13 Hidden tab, power, and visitor mode

- **Hidden tab:**
  - Stop the rAF loop, particle timers, the narration timer, and the call scheduler.
  - Fade the audio out over 2 s, then suspend the `AudioContext` (§21 D8).
  - Send the final presence interval.
- **Return:** a fresh snapshot, a scene rebuild if the gap was long (§8.4), a greeting, and audio fading in.
- **Visitor mode** is the same bundle with a `visitor` capability profile:
  - Compiled out: the presence monitor, event writer, greeting runner, offers, settle, notebook, and account chrome.
  - Top bar: accessibility settings plus a matter-of-fact label, "Visiting · view only."
  - Birds are keyboard-focusable so their narrated descriptions can be read, but Enter does nothing and there is no listen-in.
  - Light follows the host's local time, and weather is the host's weather: exactly what the host would see, with no prettified rendering.

### 10.14 Notebook, settings, and memory discipline

- **Notebook panel.** A side sheet on desktop and a bottom sheet on phones. The aviary keeps running behind it, and presence continues while any part of the scene is visible.
  - Heading: "Field Notebook."
  - Entries are grouped under lowercase day headers ("tuesday —" for the last week, "march 3 —" after that), with no counts or totals.
  - The list is virtualized and paginated by cursor. Rows that scroll out of view release their DOM and data references, per the PRD's "no retained references after scroll-out."
- **Settings panels.** Lazy chunks in the matter-of-fact register (§12.4).
- **Client memory rules:**
  - no per-frame allocations in the render loop (typed-array state, pooled particles);
  - one `AudioContext` for the life of the page;
  - one worklet node per bird bus;
  - at most one worker;
  - listeners are registered once;
  - the snapshot timeline is trimmed as it plays.

  A 30-minute soak test enforces all of this (§16.1).

---

## 11. Audio pipeline

### 11.1 Principles

- **All sound is synthesized in the browser. The product ships no audio files at all:** no calls, no ambience, no reverb impulse responses, no fallback.
  - *Why:* a looped call is the audible signature of dead software. Stacked loops also produce phase artifacts that the ear catches.
  - *Budget:* the 2 MB bundle ceiling could not carry recorded variation even if we wanted it.
- **Each bird's voice is recognizable across mood and drift, and no call ever repeats exactly.** Signature *invariants* and expressive *variables* are separated by construction (§11.4).
- **The chorus is real:** independently synthesized voices with independent micro-timing, mixed at runtime.

### 11.2 Engine graph

```
per bird (≤7):  VoiceWorklet(bird) → ChannelGain → StereoPanner(x-pos, ±0.6) → DistanceLP(zone) ─┬─▶ Dry bus ─┐
                                                                                                  └─▶ Reverb send │
ambient bed:   NoiseWorklet (wind/leaf rustle, rain) → AmbientGain ─────────────────────────────────────▶ │
reverb:        ConvolverNode (IR generated at boot: stereo, 1.4 s, filtered exponential noise) ────────▶ │
master:        Sum → DynamicsCompressor (gentle glue, −1 dBFS ceiling) → UserVolume → MasterMute → destination
```

- **One `AudioContext` per page**, with `latencyHint: 'balanced'`.
- **Zone as distance:** back-perch birds get lower gain, a lower low-pass cutoff, and a higher reverb send. Front-perch birds sound near.
- **Safari audio category:** where `navigator.audioSession` exists (Safari), set `type = 'ambient'`. The aviary then mixes politely with other audio and respects the silent switch.

### 11.3 Synthesis

**The AudioWorklet** is `VoiceWorklet`, one node per bird bus. Each node has a **preallocated pool of 3 voices**, so there is no allocation per call and memory stays flat. One voice consists of:
- 2–3 oscillators (sine or triangle partials, with FM for trills and buzzes);
- a pitch-contour engine (breakpoint segments with exponential sweeps);
- amplitude modulation (tremolo and warble);
- a state-variable band-pass formant filter;
- a noise component for breath and attack;
- a click-free ADSR with a minimum 3 ms ramp.

**Parameters arrive as compact call descriptors** over the node's `MessagePort`. They are written into a preallocated ring buffer, so there is no `SharedArrayBuffer` and no cross-origin isolation requirement.

**The ambient `NoiseWorklet`** generates shaped noise:
- wind is low-passed noise with a slow LFO;
- leaf rustle is band-limited bursts;
- rain is dense filtered micro-impulses plus a low bed.

### 11.4 Call grammar, signatures, and recognizability

**Per species:** a **motif library** of 8–12 motifs, each a parametric template (chip, rising two-note, descending whistle, trill, buzz, contact call, alarm chip, dawn phrase, nightjar churr), plus a **phrase grammar**, a weighted Markov chain over motifs with phrase-length rules.

**Per bird, fixed at adoption and stored in `birds.signature`:**
- **Signature invariants:** the species timbre recipe; the individual pitch center (the species range divided into non-overlapping sub-bands for birds sharing an aviary); a characteristic interval ratio; the vibrato rate and depth fingerprint; one **signature motif**, a bird-specific parameterization that appears in most phrases; and a preferred motif ordering.
- **Expressive variables** (driven by mood, drift, and time of day): call rate, phrase length, loudness, pitch excursion (within ±2 semitones of center), tempo, motif mix outside the signature motif, and micro-timing jitter.

Traits change only the variables, never the invariants. That is why Pip is recognizable by ear after two weeks even when her mood differs and her vocal frequency has drifted up.

**Distinctness within an aviary.** When a bird is adopted, its signature is chosen to maximize the minimum distance from the aviary's existing signatures. The requirements are pitch-center separation of at least 3 semitones, a different signature motif, and a different vibrato fingerprint. This matters most when two birds share a species, which must happen at 7 birds from a pool of 6.

**The cap of 7 is empirical.** Recognizability is tested at 3, 5, and 7 birds before each rollout step (§18.3):
- *Listening panels:* after a 10-minute familiarization, listeners must identify the calling bird at ≥ 80 % accuracy at 7 birds **(cal)**.
- *Automated proxy:* a classifier trained only on our own synthetic renders must reach ≥ 95 % accuracy across the full mood and drift range.

**Signature stability.** Any change to the synthesizer, grammar, or species library must re-render every stored `signature_version` and pass a perceptual-distance regression that compares old and new renders. The stored bird must sound like itself afterwards. Otherwise a refactor could silently swap a bird (I6).

### 11.5 Scheduling

- **Two-clock pattern.** A main-thread scheduler runs every 50 ms and schedules calls 150 ms ahead on `AudioContext.currentTime`. Every scheduled call's timestamps go to the renderer through `getOutputTimestamp()` mapping, so beaks move with the sound.
- **Call sources:**
  1. **Ambient calls.** A renewal process per bird with a refractory period. Its rate is `call.rate_pm` × time-of-day × weather damping × settle quieting × a `recent_attention` term for viewer-directed calls. Vocal-frequency expression is why some birds call more often and join choruses more readily.
  2. **Scheduled social events** from the snapshot: call-response pairs, alarm calls, and chorus windows.
  3. **Reactive calls:** greetings, settle acknowledgments, offer reactions, and head-tilt responses.
- **Anti-repetition.** Each bird keeps a history of its last 16 descriptors. A candidate within distance ε of any of them is resampled, where distance combines the motif sequence, contour, interval, and timing.
- **Turn-taking.** Outside chorus windows, a bird delays its call by 0.3–1.2 s if another bird is mid-call. Inside a chorus window, overlaps and call-and-answer density rise.

### 11.6 Song fragments (the offer)

The 8 fragments are pentatonic, 3–6 notes, and 2–6 s long. They are rendered through the same worklet with an **"offered" timbre**, a soft, breathy flute-like whistle distinct from every species timbre, so the fragment never sounds like a bird. Bird responses (`join_in`, `go_quiet`, `call_against`) are grammar-generated. A `join_in` borrows the fragment's contour into the bird's own signature voice. The PRD calls this a "song fragment"; bird vocalizations are always "calls."

### 11.7 Listen-in mix and its decay

- **Engage:**
  - the focused bird's channel ramps to +6 dB with `setTargetAtTime`, τ = 0.7 s (about 2.5 s to settle);
  - every other bird ramps to −11 dB, with its distance low-pass lowered to 3.5 kHz and its reverb send raised, which is how they "quiet to ambient."
- **The floor:** other birds never go below −18 dB relative to normal. Listen-in is a **re-balance, not a mute**, and never a hard cut. That keeps it from turning into switching between soloable tracks.
- **Disengage** uses the same slow ramps in reverse. Disengage triggers are: the same bird clicked again, another bird focused, empty space clicked, keyboard focus moved away, `Escape`, or the tab hidden.
- **Presence-lapse decay:** if presence lapses while a listen-in is active, the mix decays back to ambient over about 10 s and the listen-in ends. Credit had already stopped at the lapse (§21 D5).

### 11.8 Ambient bed, weather, time of day, settle

- **Ambient bed:** a very low wind and leaf bed at all times.
- **Rain:** adds a rain texture, and calls are damped per §7.7.
- **Night:** most birds are silent while `settled`, apart from rare sleepy murmurs. The nightjar-like species continues sparsely. The bed is quieter.
- **Pre-dawn hush:** the bed thins, then morning calling builds.
- **Settle:** the global call rate halves and the master level drops about 4 dB over 4 s. The acknowledgment call plays on top.

### 11.9 Autoplay policy: calls audible from the first frame whenever the browser allows

- **Creating and resuming the context.** The context is created at boot and `resume()` is attempted immediately.
  - On a returning user's browser, Chrome's media-engagement allowance and per-site settings often let audio run from the first frame. That is the PRD's "calls already audible."
  - Otherwise the context stays `suspended`, and we resume it on the **first activation-triggering input** anywhere on the page (`pointerdown`, `keydown`, `touchend`), fading the chorus up over 2.5 s. The effect is the window opening, not a start. Note that `pointermove` is *not* user activation, so a user who only moves the mouse will not unlock audio.
- **We never show a "tap to enable sound" overlay or button.** It would be an announcement and a "ready" pop.
- **While audio is suspended:**
  - captions are **on by default**, unless the user has explicitly turned them off. The PRD's "audio context permission denied" case is covered by the same rule as missing WebAudio;
  - the Accessibility panel shows a matter-of-fact line: "Sound starts after your first click or key press. This is a browser setting."
- **First-ever session:** the adoption step supplies the unlocking gesture, so a new user's first encounter with the aviary is audible.
- **iOS:** unlock happens on `touchend`, and the audio session category is ambient (§11.2).

### 11.10 Fallback ladder

1. **The AudioWorklet fails to load:** synthesize with native nodes. OscillatorNode and GainNode chains are created per call and released on `ended`. Memory is freed, and the soak test covers this path too.
2. **WebAudio is unavailable, context creation fails, or the device errors:** the aviary plays in **graceful silence with captions on by default**. There is no recorded-audio path under any condition, because silence with captions beats canned audio.
3. **Recovery:** on `statechange` to `interrupted` or `suspended` (for example, iOS phone calls), wait for the interruption to end, then resume with a fade.

Audio errors are counted by class and browser family only (§16.3).

### 11.11 Captions (generated from what was actually played)

**Generation.** Each scheduled call produces one **call descriptor** (motif sequence, note count, contour, loudness class, tempo, repetition and pause structure, perch zone). The **same object** drives both the synthesizer and the caption realizer, so a caption can never describe a different call than the one heard. The realizer maps descriptor features to naturalist prose using a writer-authored lexicon with synonym sets, for example:
- "a soft three-note rise"
- "a low trill, paused, low trill again"
- "a single sharp call from the back perch"

**Display:**
- Captions are small text anchored above the calling bird. They fade in with the call and stay for max(call duration, 1.2 s + 0.25 s per word).
- They have a translucent scrim tuned per light phase, so contrast stays ≥ 4.5:1 in every lighting state.
- At most 2 are visible at once. Overlapping calls merge into one line ("pip and wren, back and forth").
- They avoid covering other birds. In reduced motion they fade without moving.

**Mood appears through the words,** never as labels: "a thin, hurried call" rather than "wary."

---

## 12. Voice and content systems

### 12.1 Two registers, one rule

| Register | Surfaces | Rules |
|---|---|---|
| **Naturalist** | aviary, notebook, narration, captions, offer menu, adoption, newcomer naming | Lowercase by default, including bird names. Present tense. Specific: named birds, concrete detail, comparisons against the aviary's own history. Bird verbs: notice, perch, settle, listen in, offer. No exclamation marks, no second person ("you"), no announcement framing, no gamification lexicon, no numbers used as stats. |
| **Matter-of-fact** | sign-in, sessions, email change, export, deletion, visits management, privacy, accessibility settings, errors, unsupported browser, and any future money surface | Sentence case. Direct: what happened, then what to do. No bird metaphors and no warmth standing in for usefulness. |

**The rule for a new surface:** if the user is engaging with the system *as a system* (identity, money, errors, settings), it uses the matter-of-fact register. Everything else uses the naturalist register.

### 12.2 Enforcement machinery

- **Surface registry.** Every route, panel, and string key declares `register: 'naturalist' | 'system'`. Strings live in two catalogs.
- **Lints** (in CI and in the realizer at runtime):
  - *Naturalist:* no uppercase first letters except proper UI labels such as "Field Notebook"; no `!`; no `you`/`your`; a banned lexicon (achievement, unlock, streak, level, badge, points, score, welcome, congrats, "days in a row," "every day," "you visited," happier, sad, hungry, lonely, "missed you"); no digits except dates in notebook headers.
  - *System:* no naturalist bird-verb phrasing in error strings.
- **Template library.** Written and owned by the content designer. Every template has slot types, synonym sets, and test fixtures. Snapshot tests render 200 samples per template per release for human review.
- **Specificity check.** A template is rejected unless it references at least one concrete aviary fact: a bird name or description, a perch, the light, the weather, or a comparison against the aviary's own history. This blocks generic lines like "your bird is happier."

### 12.3 One realizer, three outputs

The package `@aviary/prose` holds the grammar engine and the lexicon. It is shared by three consumers:
- the server **notebook writer** (§7.11);
- the client **narration composer** (§13.2), about 25 KB gzipped, loaded right after the first bird;
- the client **caption realizer** (§11.11).

Sharing the package means the notebook, narration, and captions all sound like the same observer. A screen-reader user moving between the aviary and the notebook hears one product, not two.

### 12.4 System copy inventory (canonical strings)

- Sign-in failure: "We couldn't sign you in. The link may have expired. Try requesting a new link."
- Session ended: "Your session timed out. Sign in again to keep watching."
- Load failure: "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch."
- Visit unavailable: "This visit is no longer available."
- Rate limited: "Too many sign-in links were requested for this email. Try again in a few minutes."
- Offline: "Offline. Your aviary will catch up when you reconnect."
- Soft delete: "This account will be deleted on {date}. [I changed my mind]"
- Invitation sent (inline, not a toast): "Invitation sent."
- Magic-link email: "Sign in to Pocket Aviary. This link expires in 15 minutes and works once. If you didn't ask for it, you can ignore this email."

Emails carry no tracking pixels, no link tracking, and no UTM parameters.

---

## 13. Accessibility surfaces

Accessibility is designed to carry the charm, not to reach parity by checklist. It ships in v1 or v1 doesn't ship (I15). The cheap version is explicitly rejected: labeling every visual state, exposing traits to ARIA, and a static reduced-motion scene.

### 13.1 Semantic structure

```
<header role="banner">  top bar: buttons with accessible names; menus follow the ARIA menu pattern
<main>
  <canvas aria-hidden="true"> ×n
  <div role="group" aria-roledescription="aviary" aria-label="the aviary"
       aria-describedby="kbd-hint">            ← roving tabindex over birds
     <div role="button" aria-roledescription="bird" aria-label="pip"
          aria-describedby="pip-desc" aria-pressed="false"> … one per bird
  <p id="kbd-hint" hidden-visually>Use the arrow keys to move between birds. Press Enter to listen in, Escape to stop.</p>
  <div aria-live="polite" aria-atomic="true" class="visually-hidden" id="narration">
  <div id="narration-visible" …>   ← only when "Show narration as text" is on
```

- **Bird descriptions** (`pip-desc`) are prose that is re-authored only when focus arrives, for example "the small grey one, on the front rail, preening." They are never live-updated, and they never contain digits, trait names, or mood labels (I5).
- **Keyboard hint** (`kbd-hint`) is the one instructional string, and it exists for assistive technology only. It never appears visually and is never announced on its own.

### 13.2 Screen-reader narration

- **Source.** The narration composer reads the *same* scene state the renderer draws from: perches, current behaviors, mood expression, calling activity, light, weather, offered items, and recent reactions.
- **What it writes.** Short running prose in the naturalist voice. Each update covers the scene or one or two birds, and the focus rotates across birds and scene over successive updates. For example: "a small grey bird is perched on the front rail, calling softly. another bird sits further back with feathers fluffed. it is morning in the aviary; the light is gentle." It never writes state lists like "pip at perch 2" or "wren mood: content."
- **Cadence.**
  - Idle: one update every 30–60 s, randomized so there is no mechanical rhythm.
  - Repetition: a phrase history suppresses repeated lines, and nothing is re-announced if nothing notable changed.
  - **Priority bump** for user-initiated moments: the return-greeting (narrated within about 1 s of its first frame, as an observation such as "pip looks up from preening and gives a quiet two-note call"), offer reactions as they happen, settle, and listen-in engage and disengage ("pip's calls come closer; the others soften"). These bypass the idle interval but still use the polite live region, so they never interrupt the user's own reading.
  - Hidden tab: narration pauses.
- **Settings** (matter-of-fact register): narration on/off (default on; the live region is harmless for sighted users), "Show narration as text" (renders the same prose in a strip below the scene at WCAG AA contrast), and a sparser cadence (60–120 s).
- **Visitors** get the same narration, with no greeting lines, because birds do not greet visitors.

### 13.3 Captions

Captions are opt-in from Accessibility settings. They are on by default whenever audio cannot play (§11.9–11.10). Generation and display are covered in §11.11. Caption text also goes to the visible narration strip, when that is enabled, for users who prefer one text location.

### 13.4 Reduced motion

Reduced motion is its own designed surface (§10.12). It is design-reviewed as a first-class aesthetic, calmer and slower, rather than as a fallback. The trigger is either the system preference or an in-app setting ("Follow system" / "Reduced" / "Standard").

### 13.5 Keyboard model

| Key | Effect |
|---|---|
| `Tab` / `Shift+Tab` | Moves through top-bar items, then into the scene. |
| Entering the scene | Focuses the first bird. Order is spatial, left to right then front to back, **frozen while focus is inside the scene**, so birds moving don't reshuffle it. |
| `←` `→` `↑` `↓` | Move focus between birds. Focusing a different bird disengages any listen-in. |
| `Enter` / `Space` | Starts listen-in on the focused bird. Pressing again on the listened-in bird toggles it off. |
| `Escape` | Exits listen-in (focus stays on the bird), or closes an open menu or panel. |
| Tabbing out of the scene | Disengages listen-in. |
| `o` | Opens the offer menu; the menu is fully arrow-navigable. |
| `n` | Opens the Field Notebook. |
| `m` | Mutes or unmutes. |

- **Settle** is reached through its top-bar button and has no single-key shortcut, to avoid accidental goodbyes.
- **Single-key shortcuts** are active only when focus is not in a text field. They can be turned off in Accessibility settings (WCAG 2.1.4).
- **The focus ring** is a soft two-tone outline, a light inner stroke with a dark outer stroke, drawn on the overlay target that tracks the bird. It reads against both bright midday and dim night states (WCAG 2.4.7 and 2.4.11), and the visual designer owns the exact treatment.
- **Focus is never lost** when a bird flies: the target moves with the bird.

### 13.6 Contrast, audio control, motion: the WCAG mapping

- **1.4.3 AA contrast** for all user copy: top bar, settings, account, errors, captions (per-light-phase scrim), and visible narration. This is verified by automated contrast sampling across 8 light phases × 3 weather states.
- **1.4.2 Audio Control.** Mute and volume live in Accessibility settings, the first reachable top-bar group, and `m` toggles mute.
- **2.2.2 Pause, Stop, Hide.** The living scene is the essential content. Reduced motion is the provided alternative, and it has been reviewed with an external accessibility auditor.
- **2.3.1** No flashing.
- **2.5.8** Target sizes of at least 24×24 CSS px, with birds at 44×44 or larger.
- **Accessibility settings copy** uses the matter-of-fact register.

### 13.7 Fair presence for assistive-technology users

The strict presence definition could under-count users whose input never reaches the page as `pointermove`/`keydown`, such as screen-reader browse mode or switch access. Their birds would then drift more slowly, which would ration the product's quality by ability. We mitigate this in three ways:
- keyboard-driven `focusin` counts as activity (§9.1);
- W is biased long;
- the harness includes an AT persona, and moderated sessions with screen-reader users validate the result.

Any change to the activity signal set requires review by the calibration owner and the accessibility lead together (§21 D3).

### 13.8 Testing

- **Automated:** axe-core in CI on every surface; Playwright keyboard-only end-to-end journeys (sign in, adopt, listen in, offer, settle, notebook, settings, invite, revoke); contrast sampling.
- **Manual matrix every release:** NVDA+Firefox, JAWS+Chrome, VoiceOver+Safari on macOS and iOS, TalkBack+Chrome on Android.
- **Before launch:** moderated sessions with blind, low-vision, deaf or hard-of-hearing, and vestibular-sensitive participants.
- **Narration quality** is reviewed with screen-reader users as well as through the automated voice lints. Launch gates are in §18.4.

---

## 14. Accounts, authentication, and privacy implementation

### 14.1 Magic-link sign-in (sign-up and sign-in are the same flow)

1. **Request a link.** The user enters an email, and `POST /v1/auth/links` always returns `202` with the same matter-of-fact text: "Check your email for a sign-in link." A 256-bit token is generated and only its SHA-256 hash is stored. It expires in 15 min and can be requested again as often as the per-email rate limit allows.
2. **Open the link.** The link opens a one-button page. Consumption happens on `POST` (scanner-safe, §6.2) and is atomic: `UPDATE … SET consumed_at = now() WHERE token_hash = $1 AND consumed_at IS NULL AND expires_at > now()`.
   - Used, expired, or replayed links show "We couldn't sign you in. The link may have expired. Try requesting a new link."
   - Opening the link on a different device from the one that requested it is allowed, and signs in the device where it was opened.
3. **First consumption for a new email.** It creates the account (UUIDv4, encrypted email, blind index), the aviary, and the two starters in one transaction, then routes to adoption (§10.10).
4. **After sign-in there is no confirmation message.** The user lands directly in the aviary.

### 14.2 Sessions

- **What a session is:** each sign-in creates a `device_sessions` row. Refresh tokens rotate on every use, with reuse detection: presenting an already-rotated token revokes the whole session.
- **The device list:** the Account panel shows the device label, the sign-in date, and a day-granular "last active" value, each with a **Revoke** control.
- **Revocation** is immediate. It adds the session to the Redis revocation set, which the edge worker and the API check on every request. That device's next request returns `SESSION_ENDED`.

### 14.3 Email change

- The user enters a new address, and the system sends a verification link to it.
- **Until the new address is verified, the old address remains the sign-in address.** Verification swaps the ciphertext and blind index atomically, then sends a matter-of-fact notice to the old address.
- An unverified change expires after 24 h.

### 14.4 Export

**Contents.** `POST /v1/account/export` produces:

```jsonc
{
  "export_version": 1, "generated_at": "…",
  "account": { "email": "…", "created_at": "…", "timezone": "…", "settings": { … } },
  "aviary":  { "created_at": "…" },
  "birds": [{ "bird_id": "…", "name": "pip", "species": "grey tit (pocket aviary)", "adopted_at": "…",
              "current_mood": "content",
              "personality_sealed": "pa1.<base64url AES-256-GCM blob>" }],
  "notebook": [{ "entry_id": "…", "day": "2026-09-22", "text": "pip greeted before wren today, first time this week." }]
}
```

- **Personality appears only in sealed form.** It is present, as the PRD requires, but unreadable. The server holds the key. This honors "the user never sees these numbers, at any version, in any tier" (§21 D2). A plain-language `README.txt` explains: "personality is included in sealed form so it can be kept and restored, not read."
- **Deliberately excluded:** presence data and session history. An exportable visit log is one of the disguised streak surfaces the PRD rules out (I16).

**Delivery.** The job builds the file, encrypts it at rest, and emails the verified address a signed download link. The link expires in 72 h and allows at most 3 downloads.

This is a quiet quality-of-life feature and is not promoted anywhere.

### 14.5 Deletion

**Soft phase (days 0–30).**
- `POST /v1/account/deletion` sets `status = pending_deletion` and `hard_delete_at = now() + 30 d`.
- **Every** signed-in page shows the matter-of-fact strip "This account will be deleted on {date}. [I changed my mind]". Clicking restores the account immediately.
- During this window:
  - the aviary keeps ticking, so birds are never frozen and restoring loses nothing;
  - sign-in still works;
  - visits are suspended, and visitors get `VISIT_UNAVAILABLE`;
  - no new invitations can be sent.

**Hard phase (day 30).** An idempotent saga run by `jobs-worker`:
1. Revoke all sessions and visitor sessions.
2. Delete the account's rows in Zone B: birds, personality, journal, bird state, events, presence, facts, notebook, weather, newcomers.
3. Delete its rows in Zone A: invitations, visit log, settings, links, email changes, sessions, the account row.
4. Delete export objects.
5. Purge residual log and error records keyed by the UUID.
6. **Destroy the DEK.** This crypto-shreds the email ciphertexts that remain in backups.
7. Write the tombstone to `deletion_ledger`.

A verification job then asserts that zero rows reference the UUID in any store. Backups age out within 35 days, and any restore replays `deletion_ledger` first.

This split is deliberate. The soft window protects against the regret of an accident, and the hard delete honors the commitment that interaction history belongs to the user.

### 14.6 The privacy boundary as architecture (I12)

- **Separate stores and accounts.** Zone A, Zone B, and Zone C are separated at both the network and cloud-account level. No analytics warehouse, CDC stream, ETL job, or backup export reads Zone B. ML training does not exist. If it ever does, a schema-level ban keeps per-bird fields out of it.
- **Telemetry schema registry.** Every metric, RUM field, and log field is on an allowlist, and CI fails on anything new that isn't. Forbidden labels include `account_id` in metrics, `bird_id`, trait names, mood, event type, offer kind, and notebook text.
- **No third parties in the page.** No third-party scripts load: no analytics SDKs, no session replay, no heatmaps, no ads, no tag managers. The Content Security Policy enforces first-party-only script and connect sources.
- **Error tracking.** Error reports (self-hosted or a processor with a DPA) have DOM-text breadcrumbs disabled and request bodies stripped. Bird names and notebook text never leave as error context.
- **Support access** to an individual aviary is break-glass only. It requires the user's explicit request in a support ticket, is read-only, time-boxed, and audited. No dashboard of any aviary exists.
- **Privacy policy.** Account settings link to the plain-text privacy policy. It names the aggregate telemetry categories and explicitly states that per-bird interaction state is never used for analytics, training, recommendations, or third parties.

---

## 15. Social: visits

### 15.1 Invitation lifecycle

```
created ──(email sent)──▶ outstanding ──(visitor accepts one-time link)──▶ active ──(access_until)──▶ ended
    │                          │ (30 d unused)                               │ (host revokes)
    │                          ▼                                             ▼
    │                       expired (cannot be revived; host may issue a new invite)   revoked
    └────────────── host revokes while outstanding ──▶ revoked (link silently stops working)
```

- **Per-invite opt-in only.** There is no global discoverable flag, no share prompt during onboarding, no default visitor list, no automatic re-invitation, no friend-of-friend, and no "frequent visitor" status. Visits are off until the host explicitly sends an invitation.
- **Invitation email** (matter-of-fact): "{host email} invited you to visit their aviary on Pocket Aviary. You can watch and listen; you won't be able to change anything. The link works once and expires in 30 days." The host is told in the form that their email address will be shown.
- **Accepting.** The one-time token is consumed on `POST`. That establishes a visitor cookie scoped to this aviary, valid until `access_until` (30 days after acceptance, §21 D6), so the visitor can return from that browser. A forwarded email is useless once the link has been consumed.

### 15.2 What the visitor gets

- **The host's aviary exactly as it is, right now:** the same snapshot (§6.7), the host's local time and weather, calls and birds.
- **Their own accessibility settings,** stored locally: captions, reduced motion, narration display, and volume.
- **Nothing is prettified.** There is no show-off mode.

### 15.3 Read-only and inert: enforced on both sides

- **Visitor build.** The presence monitor, event writer, greeting runner, offers, settle, listen-in, and notebook are compiled out.
- **Server.** Visitor tokens carry only `aviary:read` and cannot reach mutating endpoints. Ingestion rejects any event that doesn't come from a host device session. The tick ignores anything without a host actor, as defense in depth.
- **What that guarantees:** a visitor watching for an hour does not drift the host's birds. They trigger no greetings. There is no co-presence: no shared cursor, no avatar, no marker, no "your friend is here" overlay. The host sees nothing new in their aviary.
- **Why co-presence is out:** it would need a multi-user simulation, where presence and drift inputs come from several people. That is a different and much larger product.

### 15.4 Revocation and expiry

- **Revoking.** Revocation takes effect on the server at once. The visitor's next snapshot pull, within about 60 s at keepalive cadence, returns `410`, and the client replaces the scene with "This visit is no longer available." The host gets no confirmation notice; the invitation simply leaves the list.
- **Revoked or expired links** show the same surface.
- **Expiry.** Outstanding invitations expire after 30 days unused. A nightly job marks them, and they cannot be revived.

### 15.5 Visit log

- **Where it lives:** Account settings → Visits. It shows each visitor's email, the date, and the approximate duration, most recent first, along with outstanding invitations.
- **How duration is measured:** from the first and last snapshot pulls of the visitor session, rounded to minutes. That is sufficient for "approximate" and requires no visitor telemetry.
- **Pull, never push:** there is no badge on the settings icon.

### 15.6 Visit emails (opt-in, off by default)

- **The setting:** a single per-account toggle, "Email me when someone visits." It defaults to off and never appears during onboarding.
- **The email:** when the toggle is on, a matter-of-fact email ("{visitor email} opened your aviary at {time}.") is sent at visit start, at most once per invitation per day.
- **Why it's the only exception:** it is the single case of the product emailing the host about anything aviary-adjacent, and it exists only because the host asked for it.

### 15.7 Abuse controls

- **Sending limits:** at most 10 outstanding invitations per host and 20 sends per day.
- **Recipient opt-out:** recipients can opt out of all future invitations. The opt-out is stored as a separate HMAC in the email provider's suppression list.
- **Bounce handling:** hard bounces automatically suppress the address.
- **Content:** invitation emails carry no host-supplied text, so no one can inject content into them.
- **Deliberately not built:** chat, comments, avatars, public discovery, leaderboards, mutual visits, and any stats that could feed them. No cross-account aggregates are computed, so none can "just be exposed" later.

---

## 16. Performance budgets and observability

### 16.1 Budgets

**Reference definitions:**
- *Mid-tier mobile:* a mid-tier Android phone from about two years back (Pixel 7a class) plus a two-generation-old iPhone SE.
- *4G profile:* 9 Mbps down, 1.5 Mbps up, 60 ms RTT, plus a stress profile of 1.6 Mbps and 150 ms RTT.
- *Five-year-old mid-range laptop:* a 2021 Intel i5 with Iris Xe integrated graphics at 1080p (DPR 1) and 1440p (DPR 1.5), plus a 2020 M1 MacBook Air at DPR 2.

| Budget | Target | Hard gate | How measured |
|---|---|---|---|
| Initial JS at first paint (PRD) | ≤ 180 KB gz for the critical path (HTML + inline + `scene-core`); ≤ 450 KB gz everything loaded before idle | **< 2 MB gz** (CI fails) | `size-limit` per entry and chunk graph |
| Time to first bird visible (PRD) | p75 < 400 ms, reference mobile / 4G | **p75 < 500 ms** (lab, per release); RUM p75 alarm at 500 ms | `performance.mark('first-bird')` in the rAF after the first frame that drew a bird, taking the next frame's timestamp as the painted bound; lab runs use real devices on WebPageTest |
| Idle motion (PRD) | 60 fps, p95 frame ≤ 16.7 ms, main-thread work ≤ 4 ms per frame, 0 long tasks > 50 ms in steady state | ≥ 99 % of frames on time over a **30-minute** session on the reference laptop | Chrome tracing in the device lab; RUM frame-time histograms |
| Memory (PRD) | Heap slope ≈ 0 after 2 min warm-up | **< 2 MB growth over 30 min**; no growth in detached nodes; exactly 1 `AudioContext`; bounded workers and nodes | CI soak: headless Chrome drives a 30-minute scripted session (listen-ins, offers, notebook scrolling, tab hide and show) while CDP samples the heap. **This is a real CI test, not a guideline.** |
| Snapshot payload | ≤ 6 KB gz | ≤ 8 KB gz at 7 birds | Contract test |
| Audio CPU | ≤ 5 % of one core, 7-bird chorus, reference laptop | No render-quantum underruns in a 30-minute soak | Worklet timing counters |
| Tick (PRD alarm) | p50 compute < 20 ms; lateness p99 < 1 s | **Alarm: p99 tick latency > 5 s** | Server histograms |
| API | Snapshot p95 < 120 ms regional; events p95 < 200 ms | Alarm at 2× target | Server histograms |

**What the budgets drive:**
- procedural audio instead of files;
- species art as compact vector parts rasterized on the client;
- Canvas 2D instead of WebGL;
- aggressive code-splitting for settings, account, visits, and the notebook;
- edge-inlined snapshots;
- warm-start rendering.

### 16.2 Synthetic monitoring

A fleet of scripted browsers runs every 5 min from 6 geographies (NA-E, NA-W, EU-W, EU-C, APAC-SE, SA-E) against dedicated synthetic accounts, which are flagged `synthetic` and excluded from nothing because nothing is analyzed. The checks:
- first bird;
- `AudioContext` reaches running after a scripted click;
- snapshot freshness, meaning `server_time − tick time`;
- event round-trip;
- the visitor path, including revocation;
- magic-link delivery, using a test inbox.

### 16.3 What we measure (aggregate only, no account dimension)

- **RUM:** first-bird time, TTFB, INP on chrome controls, frame-time histograms, long-task counts, heap buckets where the API exists, `AudioContext` failure and suspension counts by browser family, worklet load failures, snapshot fetch latency and errors, and unsupported-browser counts by browser family. RUM beacons are unauthenticated, sent without cookies, IP-stripped at the collector, bucketed at ingest, and carry timestamps coarsened to the hour.
- **Session-duration histogram:** the anonymized kind the PRD allows. Buckets are computed on the client and sent in the same unauthenticated beacon. Nothing joins it to anything.
- **Server:** request rate, latency, and errors per **endpoint**; tick compute latency and lateness; CAS aborts; lease handoffs; queue depths; database and Redis health; email send and bounce rates; magic-link consumption failures by reason; snapshot size histogram; and the **invariant-violation counters** (trait-lower attempts, presence over wall clock, ceiling breaches, visitor-event rejections, email-shaped strings redacted in logs). The invariant counters are error rates. The PRD explicitly allows asking "is this account having errors."
- **Account counts** from the accounts table, for capacity planning only.

### 16.4 What we deliberately do not measure

- Any per-bird or per-account behavior: trait or drift distributions, "average drift across all accounts," mood occupancy, offers or listen-ins by type, presence totals, greeting statistics, notebook entry counts or content, visits per account.
- Engagement analytics: DAU/retention cohorts keyed by account, funnels, streak-like frequency metrics, email open or click tracking.
- A/B tests of affective mechanics such as drift speed or greeting style. Measuring their outcome would require exactly the per-account interaction analytics the privacy commitment forbids. Feature flags exist for safety and staged enablement only.
- Session replay, heatmaps, any third-party analytics.
- Event type as a dimension on any metric, because a per-type count is already population-level analysis of how birds are interacted with.

These refusals also protect the future: with no cross-account aggregates in existence, a leaderboard or "average drift" dashboard cannot "just be exposed" later.

### 16.5 Alerts (paging)

- Tick latency p99 > 5 s for 5 min.
- Tick lateness p99 > 10 s.
- Any invariant-violation counter > 0.
- Snapshot 5xx > 0.5 %.
- Ingest 5xx > 0.5 %.
- RUM first-bird p75 > 500 ms for 30 min.
- `AudioContext` failure rate more than 2× the 7-day baseline.
- Magic-link delivery p95 > 60 s.
- Deletion saga overdue by more than 24 h.
- Personality checksum audit mismatch (this is a sev-1).

---

## 17. Testing strategy

### 17.1 Engine

- **Unit tests** for each sub-step.
- **Property-based tests** (fast-check) for I2 (monotonic), I3/I4 (no double-apply, no recompute), union crediting, cooldown, ceiling, and dwell. Each runs over 10⁵ random traces.
- **Golden deterministic replays:** recorded synthetic event streams must produce expected state hashes. Any engine change that alters a hash needs an explicit re-baseline, justified in its PR.
- **The calibration harness** (§7.12) runs nightly and on engine changes.

### 17.2 Perception and dogfood studies (wall-clock bound, so they start early)

- **Visibility-threshold study.** Participants compare side-by-side recordings of the same bird at different trait levels, and the study sets `V_vis` from the just-noticeable difference for each expression channel.
- **Consented staff dogfood,** from week 12 onward (§19). Participants use real aviaries in real time. Moderated check-ins at weeks 1, 3, and 6 ask whether the birds feel different, and when the participant first noticed. These studies use participants' own words, with consent. They never use production telemetry.
- **Aliveness studies:** first-session interviews testing "did it feel like it was already going," and whether any greeting felt canned.

### 17.3 Sync and chaos

A Jepsen-style harness covers:
- lease split-brain;
- killing workers mid-commit;
- duplicated, replayed, and reordered event batches;
- client clock skew of ±10 min;
- network partitions between API and database;
- a laptop and phone concurrency script (the PRD's lost-drift scenario).

Pass criteria: no trait ever decreases, no event is applied twice, no event is lost, and drift equals the single-writer oracle.

### 17.4 Client

- **Presence:** a table-driven matrix of visibility × focus × activity sequences, including minimized, background, side-by-side unfocused, idle for 5 min, touch-only, and screen-reader focus navigation.
- **Greeting rules:**
  - exactly one bird greets within 2 s;
  - further birds are staggered and never in unison;
  - the bolder bird comes first in ≥ 70 % of trials;
  - class selection follows absence length;
  - there are no duplicate signatures over 1,000 opens.
- **First frame:** the first frame with a snapshot contains birds in non-rest poses, and no opacity animation runs from 0 on the fast path.
- **Visual regression** across the viewport matrix, with the "no bird cropped" assertion.
- **Reduced motion:** instrumentation asserts no transforms except opacity cross-fades, and the registry refuses any behavior without a reduced-motion variant.
- **Soak and memory** (§16.1).

### 17.5 Audio

- **Offline renders** with `OfflineAudioContext`, checked for no clipping (peak < −1 dBFS), no DC offset, and no clicks (a discontinuity detector).
- **Spectral envelopes** per species stay inside their specification.
- **Repetition:** the MFCC and DTW distance between consecutive calls stays above ε across 10⁴ calls per bird.
- **Recognizability:** the classifier plus human panels (§11.4).
- **Signature stability regression** on every audio change.
- **Caption consistency:** a caption's note count, contour, and repetition must match its descriptor.

### 17.6 Voice, privacy, security, refusals

- **Voice:** voice lints plus human review of 200 samples per template.
- **Privacy:**
  - PII scanners over staging and production logs;
  - telemetry schema registry tests;
  - boundary tests (the analytics network cannot reach Zone B);
  - export schema tests (no plaintext traits, no presence);
  - deletion saga verification.
- **Security:** an external penetration test of the auth, visit-token, export-link, and CSRF surfaces before beta.
- **Refusal tests** (§3.3), automated:
  - an API surface snapshot must contain no trait, perch, or notebook write routes;
  - the component library must export no toast, badge, or spinner;
  - no `Notification.requestPermission` exists in the bundle;
  - no digits or trait words appear in the accessibility tree for birds;
  - a return to the tab adds no visible text.

---

## 18. Rollout

### 18.1 Phases

| Phase | When | Who | Purpose |
|---|---|---|---|
| **0. Internal dev** | Weeks 0–12 | Engineers | Non-production environments allow a per-aviary **accelerated clock** (a test-only flag that production code refuses). It lets us exercise weeks of drift, the 90-day newcomer, and 7-bird aviaries in hours. |
| **1. Consented dogfood** | From week 12 | ~60 staff, with written consent to moderated check-ins | Real aviaries in real time. This is the only way to validate "visible after ~3 weeks" as a feeling, so it starts as early as the vertical slice allows. |
| **2. Closed beta** | Week 24 | ~1,500 accounts via single-use invite codes (not an email allowlist, which would put emails somewhere else) | Real-world performance across devices and networks, accessibility sessions with external participants, content fatigue checks. |
| **3. General availability** | Week 30 | Open sign-up | Capacity ramp, sized from request rates and account counts only. |

### 18.2 Instrumented from day one (dogfood onwards)

- RUM performance: first bird, frames, memory buckets, audio errors.
- Tick latency and lateness, with the PRD p99 > 5 s alarm live from the first real account.
- Invariant-violation counters.
- Personality checksum audits on every deploy.
- Endpoint error rates.
- The synthetic monitoring fleet.
- Magic-link delivery.
- The deletion saga monitor.

We don't instrument engagement, drift distributions, or behavior. Their absence is a day-one design decision too (§16.4).

### 18.3 Ramping birds per aviary

- **Capacity from the start:** the engine supports the full cap of 7 from the first line of code. Rendering, audio CPU, and memory budgets are launch-gated **at 7 birds** using accelerated-clock aviaries, even though no real aviary will reach 7 for more than a year.
- **The operational ceiling:** `max_birds_enabled` is a global safety valve. It is never per user, never paid, and never tied to behavior.
  - **3** at beta, gated on a recognizability panel and chorus review at 3 birds.
  - **5** when the 5-bird gates pass, roughly around when the earliest dogfood aviaries reach day 165.
  - **7** when the 7-bird gates pass, before any aviary reaches day 330.
- **Ahead of demand:** the age thresholds (days 90, 165, 240, 330, 450) mean the ceiling will almost always be ahead of eligibility.
- **If a gate slips:** the newcomer simply hasn't arrived yet. Nothing is announced, so from the user's side nothing is late and nothing is broken.

### 18.4 Launch gates (beta and GA)

1. **Performance:** all hard gates in §16.1 on the reference hardware, including 7-bird aviaries.
2. **Accessibility:**
   - narration, captions, reduced motion, and the keyboard model are complete;
   - the external WCAG 2.2 AA audit has no open AA findings;
   - the moderated assistive-technology sessions have been run and their fixes landed;
   - reduced motion has been signed off by design as a first-class surface.
3. **Calibration:**
   - harness A1–A11 are green;
   - dogfood check-ins show week-1 comments like "nothing's changed," with changes noticed looking back somewhere in weeks 2–4;
   - no participant reports birds changing between sessions.
4. **Aliveness:** no spinner or entry animation anywhere; greeting variation passes; call repetition tests pass; first-session interviews show no "loading" perception.
5. **Voice:** a full copy audit against the registers; no announcement surfaces (a manual walkthrough of every return path); the refusal tests are green.
6. **Privacy and security:**
   - the data-flow review is signed;
   - the telemetry registry is frozen;
   - the penetration test's high and critical findings are fixed;
   - the deletion saga has been verified end to end;
   - the export has been verified (sealed personality, no presence).
7. **Sync:** the chaos suite is green, and a DR drill has restored from PITR plus the journal with checksums matching.

### 18.5 Change management for the load-bearing systems

- **Drift, mood, and presence changes** need calibration-owner sign-off with the harness diff attached. They are forward-only (new `drift_fn_version`). The owner must also answer in writing: "does this make any single session visible, punish absence, or count anything other than honest presence?"
- **Audio engine, grammar, or species changes** need signature-stability regression and sound-design sign-off.
- **New user-facing strings or surfaces** need register classification, a lint pass, and content-designer sign-off. New motion needs its reduced-motion variant. New surfaces need an accessibility review.
- **New telemetry** needs a schema-registry change reviewed by the privacy owner.

### 18.6 Flags and kill switches

The following can be turned off globally: the notebook writer, the newcomer scheduler, visits, visit emails, the AudioWorklet path (forcing native nodes), and edge inlining (falling back to client fetch).

Accessibility features cannot be killed. Flags are never used for per-user experiments on bird mechanics.

---

## 19. Team, workstreams, milestones

### 19.1 Staffing (~13 engineers plus specialist roles)

- **Engine:** 3 engineers. Tick, drift, mood, social, weather, newcomers, notebook writer, harness.
- **Client scene:** 3 engineers. Boot path, renderer, motion system, layout solver, reduced motion, greeting, offers, settle.
- **Audio:** 2 engineers plus a **sound designer**. Worklet synthesis, grammars, signatures, mix, caption realizer.
- **Platform, accounts, privacy:** 3 engineers. Edge, API, auth, data, deletion, telemetry plane, SRE.
- **Chrome and social:** 2 engineers. Top bar, panels, notebook UI, settings, visits.
- **Embedded specialists:** an **accessibility lead**; a **content designer/writer** who owns the naturalist voice and the templates; **visual and motion designers** for species rigs, pose libraries (including reduced-motion stills), palette, and focus and caption treatments; one QA engineer; and an engineering manager plus PM.

### 19.2 Milestones

| Weeks | Milestone | Exit criteria |
|---|---|---|
| 0–4 | **Risk spikes** | (a) A first-bird prototype under 500 ms on the reference phone over 4G. (b) An audio spike: 7 synthetic signatures, and a preliminary panel reaching ≥ 70 % identification. (c) The presence monitor verified on 4 browsers × 3 OSes. (d) The harness skeleton plus a first drift fit. (e) Privacy zones and the telemetry registry provisioned. |
| 5–12 | **Vertical slice** | Two species; the tick, snapshot, and edge paths; renderer with idle motion; calls and chorus; greeting; listen-in; presence; magic link; adoption. **Dogfood starts at week 12.** |
| 13–20 | **Feature complete** | Offers, settle, notebook writer and templates, narration, captions, reduced motion, weather and night, 6 species, newcomers, visits, export, deletion, settings. |
| 21–24 | **Hardening** | Soak and performance gates, chaos suite, external accessibility audit, penetration test, voice audit, calibration review with 8+ weeks of dogfood. **Closed beta at week 24.** |
| 25–30 | **Beta → GA** | Beta findings fixed, the `max_birds_enabled = 3` gate passed, capacity tests done. **GA at week 30.** |

**Critical-path notes:**
- Species art, rigs, pose libraries, and motif libraries have long lead times. Design and sound design start in week 0.
- The "visible at ~3 weeks" judgment is wall-clock bound: it cannot be compressed, only started earlier.
- The recognizability ceiling at 7 decides how far the rollout ramps.

---

## 20. Risks

Each risk states what could go wrong, how we would find out, and what we do about it. Severity (H/M/L) is our estimate of impact × likelihood.

### 20.1 Drift calibration

| Risk | Sev | Signal | Mitigation |
|---|---|---|---|
| **Drift too fast.** Users notice changes session to session, and it reads like a Tamagotchi. | H | Dogfood participants report visible changes in week 1; harness A2/A3 miss. | Daily saturation, the reservoir lag, and a per-24 h contribution ceiling at 0.25·`V_vis`. We tune `k`, `S`, and `τ_R` in the harness only. |
| **Drift too slow.** Nothing the user does seems to matter, and it reads like a screensaver. | H | "Nothing's changed" at week 4 in dogfood. | Perception study to set `V_vis` per channel. We raise gain on the most legible channels (perch occupancy, greeting-first) before less legible ones. |
| **Presence inflation.** Background tabs, unfocused windows, auto-mouse jigglers, or double counting across devices speed up drift silently. | H | Harness persona failures; invariant counter for presence exceeding wall clock. | The strict three-signal conjunction, union crediting, the daily saturation (which makes jigglers nearly worthless), and failing closed on missing data. |
| **We cannot observe calibration in production**, by design. | M | — (a structural limit) | Treated as a constraint rather than a bug. The harness, consented dogfood, and moderated research are the instruments. The privacy rule wins over our curiosity. |
| **Long-run convergence.** All birds end up maximally expressive and alike. | M | Harness A9 at 1 year. | Per-bird ceilings, headroom falloff, and signature invariants. |
| **Presence unfairness for assistive-technology users** (fewer raw input events). | M | AT persona; moderated SR sessions. | Keyboard-driven `focusin` counts as activity; W is biased long; the calibration owner and accessibility lead review jointly. |
| **Muted users' birds grow quieter** relative to audible users. | L | Harness A11. | Captions count as perceiving calls; muting only slows V, never lowers it. |

### 20.2 Sync correctness and data safety

| Risk | Sev | Signal | Mitigation |
|---|---|---|---|
| **A lost or reset vector** (the worst failure, and silent). | H | Checksum audit mismatch (sev-1); journal gaps. | Single writer, the DB trigger, synchronous replication, PITR, the journal, the migration lint, and pre/post-deploy audits. |
| **Double-applied deltas** from lease races or retries. | H | CAS aborts spike; chaos suite. | Compare-and-set on `tick_seq`, fencing tokens, and event cursors in the same transaction. |
| **Last-write-wins creeping in** through "convenience" admin tools or support scripts. | M | Code review; the role grants deny it. | Only `sim_tick` has UPDATE on personality and bird state, and no admin UI writes traits. |
| **Snap or teleport artifacts** when snapshots disagree with the rendered scene. | M | Visual QA; dogfood reports. | Reconciliation through natural motion, overlapping intent windows, and full rebuild only after long gaps. |
| **A backup restore resurrects deleted accounts.** | M | DR drill. | Replay `deletion_ledger` on every restore. |
| **Tick database write load at scale.** | M | DB CPU and WAL volume, tick lateness. | Batched upserts, writing only changed rows, lazy snapshot builds, shard-level horizontal partitioning of the `aviary` DB past ~1M aviaries, and the continuous-time cadence lever (§7.1.6), which needs sign-off. |

### 20.3 Audio uncanniness

| Risk | Sev | Signal | Mitigation |
|---|---|---|---|
| **Calls sound synthetic, "beepy," or toy-like.** | H | Aliveness interviews; the sound designer's review. | A dedicated sound designer from week 0; formant and noise components; micro-timing jitter; distance filtering and procedural reverb; species timbre recipes tuned against field-recording *references*, used for design only and never shipped. |
| **Audible repetition** makes the spell break. | H | The repetition test; dogfood. | Anti-repetition memory, a large motif grammar, variation driven by mood, drift, and time of day. |
| **The recognizability ceiling falls below 7.** | H | Panel results at 5 and 7. | Distinctness-maximizing signature assignment. If it fails, the rollout ceiling holds at the passing level (§18.3), and product revisits the cap with evidence. |
| **Chorus mud or clipping** at 7 birds. | M | Offline renders; soak tests. | Turn-taking bias, zone distance cues, the compressor and limiter. |
| **Autoplay leaves first sessions silent.** | M | RUM counts of suspended contexts (aggregate). | The adoption gesture unlocks audio on the first visit; unlock on the first activation anywhere; captions by default while suspended; never an overlay. |
| **Clicks and pops** from envelope discontinuities; **AudioWorklet CPU on weak devices.** | M | Discontinuity detector; underrun counters. | 3 ms minimum ramps, bounded voice pools, the native-node fallback, and automatic reduction of the voice count on underruns. |
| **A synthesizer refactor changes a bird's voice**, making it effectively a different bird. | M | Signature-stability regression. | The regression suite is a blocking gate. |

### 20.4 Accessibility regressions

| Risk | Sev | Signal | Mitigation |
|---|---|---|---|
| **Narration decays into state-list phrasing** as features are added. | H | Voice lint; SR-user review. | A shared realizer, a specificity check, and the banned patterns ("at perch", "mood:"). Narration copy goes through the content designer. |
| **Live-region spam** makes users silence narration. | M | SR sessions. | Idle cadence of 30–60 s, polite only, deduplication, a sparse option. |
| **New animations ship without a reduced-motion variant.** | M | Registry assertion. | The motion system refuses to register a behavior without one. |
| **Focus lost or disorienting** when birds move. | M | Keyboard end-to-end tests. | Overlay targets move with birds; order frozen while inside the scene. |
| **Caption contrast fails** at night or in rain. | M | Automated contrast sampling. | A scrim per light phase. |
| **Accessibility deferred to v1.1** under schedule pressure. | H | Milestone reviews. | Accessibility is a launch gate (§18.4), owned by an embedded lead from week 0. |

### 20.5 Other risks

- **Announcement and gamification creep** (H), e.g. "just a small toast," "just a streak in settings."
  - *Mitigation:* the refusals register, primitives absent from the component library, refusal tests, and the PR template checklist.
- **Too little onboarding feedback** (M): users don't discover listen-in, offers, or settle, because we refuse coach marks.
  - *Mitigation:* discoverable top-bar icons with accessible names, and birds that respond to clicks immediately. Dogfood and beta interviews check comprehension. Any fix must be *in-world*: a bird glancing at the cursor, never a tooltip.
- **Notebook fatigue or template recognizability** after months (M).
  - *Mitigation:* the 60-day template exclusion per aviary, a writer continuing to add templates after launch, and detectors that prefer rare, specific facts.
- **First-bird budget missed on cold loads** (M).
  - *Mitigation:* edge inlining, early hints, a small critical path, and the Service Worker for returns. As a last resort, the first frame is server-rendered as inline SVG from the edge worker, then handed off to the canvas.
- **PII leaks into observability** (H if it happens).
  - *Mitigation:* allowlists, scrubbers, PII scanners, the email lint, and third-party-free pages.
- **Visit invitations abused as a spam vector** (L/M).
  - *Mitigation:* send caps, suppression, fixed templates.
- **Timezone and DST oddities** (L).
  - *Mitigation:* confirm with two reports before switching zones, and ease every light transition.
- **Cost of per-minute ticks for dormant aviaries** (L/M).
  - *Mitigation:* efficient batching first. The cadence lever only with sign-off.

---

## 21. Decisions on ambiguities and conflicts in the PRD

The PRD asks us to make defensible calls instead of asking questions. Each decision below names the tension, the call, and the reason. The product owner should confirm the ones marked ★ before beta.

- **D1 ★ Settle lives in the top bar alongside the four listed icons.**
  - *Tension:* `aviary_layout.md` lists four top-bar icons and says "nothing else." `interactions.md` and `accessibility_perf.md` both require settle to be reachable from the top bar.
  - *Call:* settle is a fifth, equally quiet control. We read "nothing else" as forbidding badges, counters, status, and extra features, not a gesture the PRD itself specifies.
  - *Rejected:* hiding settle inside the offer menu (it would bury the goodbye) and putting it in account settings (that would move a naturalist gesture into a system surface).
- **D2 ★ The export carries personality in sealed form.**
  - *Tension:* `accounts_sync.md` says the export includes "current personality vectors." The glossary (which wins on conflicts) and `bird_engine.md` say the user never sees the numbers, in any surface, version, or tier.
  - *Call:* the export *includes* the vector, as an encrypted blob the user can keep but not read, plus a README explaining why.
  - *Legal:* if counsel decides a data-access request legally requires raw values, that goes through the support process, not the product.
- **D3 Presence activity signals.**
  - *Tension:* the definition names `pointermove` or `keypress`.
  - *Call:* we implement `pointermove` and `keydown` (`keypress` is deprecated). We add touch `pointerdown`, because a tap without movement is still a person at the device, and keyboard-driven `focusin`, because screen-reader navigation often never delivers key events to the page.
  - *Why it holds:* both additions are signs of a person being present, not of a tab being open, so the definition's honesty is preserved.
  - *Other settings:* W starts at 4 min (tested 3–6, biased long, as the PRD directs). Any change needs sign-off from both the calibration owner and the accessibility lead.
- **D4 Muting is a drift input, and it is never negative.**
  - *Tension:* the brief says drift responds to "whether you mute the calls or let them play." The engine's input list doesn't mention muting.
  - *Call:* presence contributes more to vocal frequency when calls are perceivable. Captions count as perceiving, so caption-only and deaf users are not disadvantaged, and accessibility is never rationed.
- **D5 Listen-in ends when presence lapses.**
  - *Tension:* the PRD lists the explicit disengage triggers but not what happens when the user walks away mid-listen-in.
  - *Call:* the mix decays back to ambient over about 10 s. Drift credit had already stopped, because listen-in only counts inside presence.
- **D6 ★ Visitor access after acceptance lasts 30 days.**
  - *Tension:* the PRD fixes unused-invite expiry at 30 days, but active invites have no stated end. It also rules out "permanent visitor lists."
  - *Call:* access ends 30 days after acceptance, and the host can re-invite.
- **D7 Settled lighting is local to the device and session.**
  - *Tension:* settle is a goodbye made on one screen, but the aviary is shared across devices.
  - *Call:* the canonical effects are mood quieting and the end of presence. Dimming a phone the user just opened elsewhere would be wrong.
- **D8 Audio fades out and suspends when the tab is hidden.**
  - *Tension:* the PRD stops rendering on hide but says nothing about audio.
  - *Call:* hearing calls from a background tab would disconnect hearing the birds from visiting them, and would drift toward the aviary being an ambient app or a notification-like surface. Suspending also saves battery.
- **D9 Notebook entries render the bird's current name.**
  - *Tension:* entries record a moment in time, but birds can be renamed.
  - *Call:* entries store `bird_id` tokens and render the current name. Identity continuity (Pippa is the same bird as Pip) outweighs a strict historical record, and a rename never makes old entries look like they're about another bird.
- **D10 ★ Newcomers are offered in the world, then adopted through the offer menu.**
  - *Tension:* the PRD says "a new species offer appears in the user's flow," which on its face conflicts with "notice, never announce."
  - *Call:* the newcomer appears on the back perch, and "offer the newcomer a perch" appears in the existing offer menu. There is no notification, badge, or modal. Declining, or doing nothing, carries no penalty.
- **D11 ★ Tick cadence is roughly one per minute for every aviary, including unobserved ones.**
  - *Call:* the PRD's "whether or not any client is connected" is followed literally at launch. The continuous-time engine *allows* a coarser cadence for dormant aviaries later, but only with sign-off.
- **D12 Visitors do not see the notebook.**
  - *Tension:* the PRD lists what visitors see (birds, calls, day/night, weather); the notebook is not on the list, and it is the host's private record.
- **D13 The opt-in visit notification is an email.**
  - *Tension:* the product has no push notifications.
  - *Call:* email is the only channel. It is sent only for hosts who flip the default-off toggle.
- **D14 Where settings live.**
  - *Call:* accessibility preferences are account-level and follow the user across devices; the OS reduced-motion preference is always honored in addition. Mute and volume are device-local, so muting a phone in a meeting doesn't mute the laptop at home.
- **D15 Mood set.**
  - *Call:* {alert, curious, content, wary, drowsy, settled}. The engine and the UI keep the night-rest *mood* `settled` and the settle gesture's *lighting* in separate code enums, to avoid a name collision in code.
- **D16 The server chooses which bird receives an offer.**
  - *Why:* offers are made to the aviary, not by clicking a bird, so the server picks the recipient from proximity, mood, curiosity, and cooldown.
- **D17 Autoplay handling.**
  - *Call:* audible from the first frame where the browser allows it; unlocked by the first real input otherwise; captions on by default while audio is suspended; never a "click for sound" prompt.
- **D18 v1 is English only.** The naturalist voice is hand-authored.
- **D19 Timezone and daylight come from the host's reported IANA zone,** with approximate latitude taken from `zone.tab`. We never collect location. Visitors see the host's time.
- **D20 Starters.**
  - *Call:* the nightjar-like species can be a starter (probability 1/3), so some aviaries have night activity from day one. Starters are held awake for the first hour after adoption, so a first session at night isn't a dark scene.
- **D21 Bird names are lowercased in naturalist surfaces and shown as typed in system surfaces.** This matches the PRD's samples ("pip", "wren").
- **D22 Canvas 2D, not WebGL.** It's sufficient for 60 fps at 7 birds, has no shader-compile stall on the first-bird path, and costs less bundle. The render core stays portable in case this ever needs revisiting.

---

## Appendix A — Starting calibration values (all **(cal)**)

| Parameter | Start | Tuned by |
|---|---|---|
| Tick cadence | 60 s | DB load + mood smoothness |
| Presence activity window W | 4 min (test 3–6) | Dogfood + AT sessions |
| Presence ping interval | 30 s | Server load |
| Daily presence saturation S | 20 min | Harness A2/A3 |
| Drift gain k | 0.032 | Harness A1/A2 + perception study |
| Reservoir release τ_R | 3 days | Harness A3 + dogfood |
| Seeds / ceilings | [0.15, 0.45] / [0.70, 0.95] | Harness A9 |
| `recent_attention` half-life / floor | 5 days / 0.25 | Dogfood return interviews |
| Offer cooldown / daily cap | 4 min / 5 per bird | Harness A8 |
| Mood min dwell | 10–25 min | Visual review |
| Perch dwell | 3–15 min | Visual review |
| Greeting window / stagger | 0.6–1.8 s / 0.8–3 s | Aliveness interviews |
| Newcomer days | 90, 165, 240, 330, 450 (±10) | Product |
| Notebook cadence | ~1 per 2–4 days; ≤ 1/day; ≥ 20 h gap | Dogfood reading |
| Narration idle cadence | 30–60 s | SR sessions |
| Top-bar fade | 3.5 s → 12 % over 1.2 s | Design review |
| Listen-in mix | +6 dB focus / −11 dB others / −18 dB floor / τ 0.7 s | Sound design |
| Weather | rain ~3/wk (6–20 min), wind 4–7/wk (3–10 min) | Design review |
| Visit access / invite expiry | 30 d / 30 d | Product (D6) |

## Appendix B — The session, end to end (what each system does)

1. **The user opens the tab.**
   - The edge validates the session, streams HTML with the local-time sky, and inlines the snapshot.
   - `scene-core` warm-starts and draws birds mid-preen and mid-scan, with a leaf mid-fall. **First bird in under 500 ms.**
   - Audio resumes if allowed; otherwise it waits for the first click or key press, with captions shown meanwhile.
2. **About 1 s later, Pip notices.** The greeting runner picks Pip (boldest, curious, high propensity) and composes a procedural glance and two-note call shaped by the absence class. Wren, warier, doesn't greet today. Narration: "pip looks up from preening and gives a quiet two-note call." No text appears on screen.
3. **The user sits and watches.** `PresenceMonitor` holds all three signals, and intervals flow every 30 s. The tick credits the union, fills reservoirs, and releases drift too small to see. Mood follows the morning.
4. **The user listens in on Wren.** The mix re-balances over about 2.5 s: others go to ambient, never silent. The event pair is credited within presence, feeding Wren's warmth and vocal frequency.
5. **The user offers a seed.** The server decides: Pip approaches, and Wren waits and then comes. The seed sits on the front rail. Pip's cooldown starts, and curiosity and boldness get small inputs. Narration covers the reaction.
6. **The user settles.** The light shifts to evening over about 4 s, calls quiet, and Wren gives a soft low call. After the 5 s undo window, `settle` is sent and presence ends.
7. **Overnight.** The tick keeps running: dusk makes the birds drowsy, night brings them to rest, and the nightjar, if present, calls late. Reservoirs keep releasing a little. At dawn the birds wake one by one.
8. **Some days later, a notebook entry appears:** "thursday — pip greeted before wren today, first time this week." Nothing tells the user it is there.
