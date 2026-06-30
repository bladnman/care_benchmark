# Pocket Aviary — v1 Implementation Plan

This plan turns the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`) into an executable engineering plan. It does not restate the spec; it interprets it into services, data, APIs, and sequencing decisions a team can build from without further clarification.

---

## 1. Scope

### In v1
- Single-user accounts, magic-link auth, one canonical aviary per account.
- Two starter birds at adoption, species drawn from a ~6-species pool, growth to a cap of 7 birds gated by aviary age.
- Server-authoritative bird engine: personality vector (5 traits), mood (5-state enum), monotonic-toward-expressive drift, presence accounting, bird-to-bird interaction (call propagation, wary-mood spread, chorus events).
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook (read-only, sparse, naturalist prose).
- Aviary scene: single horizontal scene, three perch zones, day/night cycle on local time, rare ambient weather, idle micro-motion, top-bar chrome with fade.
- Multi-device sync via server-side simulation tick (~1/min); clients never write personality state directly.
- Visit-invitation social feature: per-invite opt-in, read-only ambient visits, revocable, 30-day expiry, off by default.
- Accessibility: naturalist screen-reader narration, reduced-motion mode (designed surface, not animations-off), call captioning, WCAG AA contrast, full keyboard navigation — shipped with v1, not retrofitted.
- Performance budgets: ≤2MB gzipped initial JS, <500ms time-to-first-bird on mid-tier mobile/4G, 60fps idle motion on a 5-year-old laptop, no client memory growth over 30 minutes.
- Account export (JSON snapshot, emailed), soft deletion (30-day undo, then hard delete).
- Aggregate-only operational telemetry; per-bird/per-account interaction data is never aggregated, never used for ML, never shared.

### Explicitly not in v1 (per `non_goals.md`)
No native apps. No gamification of any kind (streaks, achievements, levels, badges, counters, calendars of activity) — this is treated as an absolute constraint on every surface designed below, not just a missing feature. No Tamagotchi mechanics (no death, hunger, decay, visible distress) — drift is asymmetric (up-only) by construction. No social-network surfaces beyond the single visit-invitation affordance (no profiles, follows, public discovery, leaderboards, comments). No multi-aviary accounts, no shared/team aviaries, no payments, no customizable scenes.

These non-goals are enforced architecturally, not just by omission from the UI: the event log and telemetry schemas described below have no fields that could back a streak counter, visit-frequency surface, or leaderboard, so re-adding one later requires a deliberate schema change, not a flag flip.

---

## 2. Architecture

### Service shape

```
┌─────────────┐      HTTPS/JSON      ┌───────────────────┐
│   Client     │ ───────────────────▶│   API Gateway      │
│ (web app)    │◀─────────────────── │ (auth, rate limit) │
└─────────────┘   snapshots, events   └─────────┬──────────┘
                                                  │
                        ┌─────────────────────────┼─────────────────────────┐
                        ▼                         ▼                         ▼
              ┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
              │ Account Service   │     │ Aviary Service    │     │ Visit Service     │
              │ (auth, sessions,  │     │ (snapshot reads,   │     │ (invites, visitor  │
              │ export, deletion) │     │ event writes)      │     │ session tokens)    │
              └──────────────────┘     └─────────┬──────────┘     └──────────────────┘
                                                  │
                                                  ▼
                                        ┌──────────────────┐
                                        │ Event Log (append- │
                                        │ only, per-account)  │
                                        └─────────┬──────────┘
                                                  │ consumed by
                                                  ▼
                                        ┌──────────────────┐
                                        │ Simulation Tick    │
                                        │ Worker (cron-like,  │
                                        │ ~60s cadence)       │
                                        └─────────┬──────────┘
                                                  │ writes
                                                  ▼
                                        ┌──────────────────┐
                                        │ Canonical Aviary   │
                                        │ State Store         │
                                        │ (birds, moods,       │
                                        │ personality vectors)  │
                                        └──────────────────┘
```

Four services, deliberately small:

1. **Account Service** — magic-link issuance/verification, session token lifecycle, email change, export generation, soft/hard deletion. Owns the account record and the encrypted email.
2. **Aviary Service** — the read/write surface clients talk to during a session: serves state snapshots, accepts interaction events (writes to the event log only), serves the field notebook, serves narration/caption text.
3. **Simulation Tick Worker** — a scheduled worker (not a request-driven service) that runs independently of any connected client, reads each account's unconsumed event-log entries, computes mood transitions and personality deltas, advances the day/night/weather state, and writes the new canonical state. This is the only writer of personality vectors and mood, full stop.
4. **Visit Service** — invite issuance/revocation, visitor one-time-link redemption, visitor session tokens (scoped read-only, no event-log write capability at all — not just policy-blocked but capability-absent at the token-scope level).

### Client/server split

The client is a thin renderer and event emitter. It never computes personality, mood, or drift. It:
- Pulls state snapshots (REST GET, small JSON payload).
- Renders/interpolates between snapshots.
- Emits interaction events (POST to event log) for offer, listen-in start/end, settle, presence pings.
- Synthesizes audio client-side via WebAudio from a call-grammar description embedded in the snapshot (see §9).
- Generates purely cosmetic, non-simulated ambient ornaments (leaf/feather drift) with no server round-trip, per `aviary_layout.md`'s explicit carve-out that these aren't simulation state.

This split is what makes multi-device sync a property of the architecture rather than a feature: two clients reading the same canonical store via the same snapshot endpoint cannot diverge, because neither one is a source of truth for anything except its own local interpolation state.

### Render pipeline boundary

The render pipeline boundary sits exactly at "snapshot in, pixels/audio out." The renderer is a pure function of (current snapshot, previous snapshot, elapsed time, accessibility settings) → frame. This boundary is enforced by structuring the client as:
- `SimulationClient` — polling/snapshot-fetch + event POST, no rendering knowledge.
- `SceneRenderer` — canvas/WebGL or DOM+CSS layer (see §8 for the recommendation), takes snapshots, has no network knowledge.
- `AudioEngine` — WebAudio synthesis from call-grammar parameters, no network knowledge.
- `AccessibilityLayer` — narration text generation + ARIA live region + caption rendering, reads the same snapshot the `SceneRenderer` does, so narration and visuals can never drift into two different "products" as `accessibility_perf.md` warns against.

---

## 3. Data model

All IDs are synthetic UUIDs. Email is stored exactly once, encrypted, on the account record (per `accounts_sync.md`'s non-negotiable rule). No other table, log line, partition key, or telemetry event ever contains email or any other PII.

### `Account`
| field | type | notes |
|---|---|---|
| `account_id` | UUID, PK | synthetic identifier used everywhere else |
| `email_encrypted` | bytes | encrypted at rest; decrypted only for sending mail |
| `created_at` | timestamp | |
| `deletion_state` | enum: `active`, `pending_deletion`, `deleted` | |
| `deletion_initiated_at` | timestamp, nullable | start of 30-day soft-delete window |
| `aviary_id` | UUID, FK | one aviary per account at v1, but modeled as a separate entity so the schema doesn't have to be reshaped if multi-aviary ever ships |
| `notification_prefs` | jsonb | currently just `visit_notifications_enabled: bool`, default false |

### `Session` (per-device)
| field | type | notes |
|---|---|---|
| `session_id` | UUID, PK | |
| `account_id` | UUID, FK | |
| `device_label` | text | user-visible in session list ("Chrome on Mac", best-effort UA parse) |
| `created_at`, `last_seen_at` | timestamp | |
| `revoked_at` | timestamp, nullable | |

### `Aviary`
| field | type | notes |
|---|---|---|
| `aviary_id` | UUID, PK | |
| `account_id` | UUID, FK | |
| `created_at` | timestamp | drives "aviary age" for the third-bird-and-beyond pacing in `bird_engine.md` |
| `settled` | bool | current settle state |
| `settled_at` | timestamp, nullable | for the 5-second undo window |
| `last_tick_at` | timestamp | watermark for the tick worker |
| `weather_state` | jsonb | current/last weather event, expiry time |

### `Bird`
| field | type | notes |
|---|---|---|
| `bird_id` | UUID, PK | stable forever — see `bird_engine.md`'s identity-continuity rule; never regenerated, never replaced |
| `aviary_id` | UUID, FK | |
| `species_id` | text | references the static species pool (visual silhouette, palette, call-motif library) |
| `name` | text | user-assigned, renameable, no effect on simulation |
| `adopted_at` | timestamp | |
| `personality_boldness` | float, normalized range | server-only field; **never serialized to any client response** |
| `personality_social_warmth` | float | same |
| `personality_vocal_frequency` | float | same |
| `personality_plumage_saturation` | float | same |
| `personality_curiosity` | float | same |
| `mood` | enum: `wary`,`content`,`curious`,`drowsy`,`alert` | |
| `mood_set_at` | timestamp | for daily-ish reset/transition logic |
| `current_perch_zone` | enum: `front`,`middle`,`back` | derived/cached from personality+mood by the tick, not computed per-request |
| `last_offer_at` (per bird, per offer-type) | timestamp | cooldown enforcement |

The five raw `personality_*` fields are deliberately **not** exposed by any API response schema (see §4) — this is enforced at the DTO layer (a `BirdPublicView` projection that the Aviary Service constructs has no path to these columns), not just by client-side discipline, so a future contributor adding a debug panel cannot accidentally leak them.

### `InteractionEvent` (append-only event log)
| field | type | notes |
|---|---|---|
| `event_id` | UUID, PK | |
| `aviary_id` | UUID, FK | |
| `bird_id` | UUID, FK, nullable | null for aviary-wide events like settle |
| `event_type` | enum: `presence_ping`,`listen_in_start`,`listen_in_end`,`offer`,`settle`,`undo_settle` | |
| `offer_kind` | enum: `seed`,`song`,`pool`, nullable | only for `offer` events |
| `client_session_id` | UUID | which device emitted it, for ordering/debugging only |
| `occurred_at` | timestamp | client-reported, server validates against skew tolerance |
| `received_at` | timestamp | server receipt time, authoritative for tick ordering |
| `consumed_by_tick_id` | UUID, nullable | set once the tick worker has folded this event into a personality delta — makes the log idempotent-safe and gives the tick a natural watermark |

This is the only table clients ever write to for simulation-relevant data. It is intentionally a pure log — no mutable aggregate fields — so "no last-write-wins" is structurally true: there is nothing to overwrite.

### `PersonalityDelta` (audit trail, append-only)
| field | type | notes |
|---|---|---|
| `delta_id` | UUID, PK | |
| `bird_id` | UUID, FK | |
| `tick_id` | UUID | which tick run produced this |
| `trait` | enum of the 5 traits | |
| `delta_value` | float, always ≥ 0 | enforces monotonicity at the schema level — a negative value is a bug, not a valid state, and a CI/runtime assertion rejects it |
| `applied_at` | timestamp | |

Storing deltas rather than only the running total gives us the "measurable in instruments after ~1 week" calibration target a concrete thing to query (sum of deltas over a 7-day window per bird) and gives support/debugging a way to answer "why did this trait move" without ever exposing the number to the user.

### `NotebookEntry`
| field | type | notes |
|---|---|---|
| `entry_id` | UUID, PK | |
| `aviary_id` | UUID, FK | |
| `text` | text | the naturalist prose, generated once and stored verbatim (never regenerated on read, so historical entries don't change voice if the generator changes later) |
| `generated_at` | timestamp | |
| `trigger_kind` | enum | internal classification used to throttle entry frequency (see §5), never exposed to the user as a label |

### `VisitInvite`
| field | type | notes |
|---|---|---|
| `invite_id` | UUID, PK | |
| `aviary_id` | UUID, FK | host's aviary |
| `visitor_email_encrypted` | bytes | encrypted, same posture as account email |
| `token_hash` | text | one-time link token, hashed at rest |
| `state` | enum: `pending`,`active`,`revoked`,`expired` | |
| `created_at`, `expires_at` | timestamp | 30-day expiry |
| `last_visited_at` | timestamp, nullable | for the visit log |
| `visit_count` | int | aggregate count only, no per-visit duration history beyond what's needed for the log display (approximate duration bucketed, not precise timestamps, to avoid building a behavioral-tracking surface for visitors) |

### Indexing/partitioning notes
- All per-account tables are partitioned/sharded by `account_id` (the synthetic UUID), never by any derived-from-email value.
- `InteractionEvent` is the highest-write-volume table (presence pings); index on `(aviary_id, received_at)` for the tick worker's scan, and a separate sparse index on `consumed_by_tick_id IS NULL` to make "give me unconsumed events" cheap as the table grows. Old consumed events older than ~90 days can be cold-archived since the tick has already folded them into `PersonalityDelta`/mood state — the event log doesn't need to be queried indefinitely once consumed.

---

## 4. API surface

REST over HTTPS, JSON bodies. No GraphQL — the access patterns are narrow and don't benefit from a query language; a fixed set of endpoints is easier to keep the PII/exposure boundaries auditable on.

### Auth
- `POST /auth/magic-link` — `{email}` → triggers email send, no response body leaks whether the email exists (avoid account enumeration). Rate-limited per-email.
- `POST /auth/magic-link/verify` — `{token}` → issues session token + sets device session cookie. 15-minute token expiry, single-use, enforced via the `token_hash` consumed-flag pattern (same approach as visit invites).
- `DELETE /auth/sessions/{session_id}` — revoke a device session (account settings session list).
- `GET /auth/sessions` — list active sessions for the account settings surface.

### Aviary state
- `GET /aviary/snapshot` — returns the current canonical snapshot: per-bird `{bird_id, name, species_id, mood, perch_zone, call_grammar_params, last_greeting_state}`, aviary-level `{settled, time_of_day_phase, weather_state}`. **No personality fields.** This is the endpoint the client polls on visibility-change, on long frame gaps, and on a low-frequency keepalive (~every 20–30s while visible), per `accounts_sync.md`.
- `GET /aviary/narration` — returns the current naturalist narration prose string + a monotonically increasing `narration_version` so the client/screen-reader layer only re-announces on change. Polled at the same slow cadence as narration updates (30–60s), independent of the visual snapshot poll so a screen-reader-only user doesn't need the visual poll path at all.
- `POST /aviary/events` — `{event_type, bird_id?, offer_kind?, occurred_at}` → appends to `InteractionEvent`. This is the **only** write path for interaction data. Returns 202 (accepted, not yet simulated) — the client does not get back a "new mood" synchronously; it sees the effect on the next snapshot poll, which is correct given the tick is async and matches "no single session moves things visibly."
- `GET /aviary/notebook?cursor=` — paginated, reverse-chronological notebook entries, cursor-based for indefinite scrollback per `interactions.md`.

### Account
- `GET /account` — account settings surface data (email, created_at, notification prefs).
- `PATCH /account/email` — initiates email-change verification flow.
- `POST /account/export` — triggers export job, emails a download link to the verified address.
- `POST /account/delete` — soft delete, 30-day window.
- `POST /account/delete/undo` — "I changed my mind," available any time pre-hard-delete.

### Visits
- `POST /visits/invite` — `{visitor_email}` (host-only, authenticated) → creates `VisitInvite`, sends one-time link.
- `DELETE /visits/invite/{invite_id}` — revoke (host-only).
- `GET /visits/log` — host's visit log (who, when, approximate duration, outstanding invites).
- `GET /visits/redeem?token=` — visitor redemption, issues a scoped visitor session token (read-only capability, no `POST /aviary/events` access — enforced by the token's scope claim, checked at the gateway before requests reach the Aviary Service).
- `GET /aviary/{aviary_id}/snapshot` (visitor-scoped variant, same response shape as the host's `GET /aviary/snapshot`, gated by the visitor token's scope) — visitors get the actual canonical state, no special "visitor rendering," per `social_optional.md`.

### Why no client-submitted absolute state anywhere
Every write endpoint above is either an auth/account-lifecycle action or an append to the event log. There is no endpoint, at any version, that accepts a mood value, a personality value, or a perch position from a client. This isn't a convention the client happens to follow — it's that no such endpoint exists in the service, so there's no code path to accidentally call.

---

## 5. Simulation engine design

### The tick

The Simulation Tick Worker runs on a fixed schedule (~60s; exact cadence is a tunable constant, calibrated during build per the PRD's own hedge). On each run, for each aviary with unconsumed events or with mood/time-of-day state that needs advancing (i.e., effectively all active aviaries, since time-of-day always advances):

1. **Load** the aviary's current canonical state and all `InteractionEvent` rows with `consumed_by_tick_id IS NULL` and `received_at <= tick_start`.
2. **Compute time-of-day/weather** advance: local-time phase is derived from the account's last-known timezone offset (captured at snapshot-request time from the client, not assumed from IP) — see note below.
3. **Fold events into per-bird signal accumulators**: presence-time (sum of presence-ping intervals), listen-in duration per bird, offer acceptances/attempts per bird, settle/undo-settle.
4. **Compute personality deltas** via the low-pass-filter drift function (§5.1).
5. **Compute mood transitions** per bird (§5.2).
6. **Compute perch-zone assignment** per bird from updated personality+mood (boldness/mood → front/middle/back).
7. **Compute bird-to-bird interaction effects**: call-response propagation, wary-mood spread, chorus-eligibility (§5.3).
8. **Evaluate notebook-entry triggers** (§5.4) and write any new `NotebookEntry`.
9. **Write** updated `Bird` rows, new `PersonalityDelta` rows, updated `Aviary` row (weather/settled state), mark consumed events with this `tick_id`.

Steps 1–9 run inside a single transaction per aviary so a crash mid-tick never leaves an aviary half-updated; the next tick simply re-reads the same unconsumed events (idempotency is safe because deltas are append-only and consumption is a flag set in the same transaction as the writes that depend on it).

**Timezone note**: "the user's local time" requires the server to know the user's timezone without ever needing the user to be present. Solution: the client sends its IANA timezone string with every snapshot request and event POST; the Aviary Service stores the most-recently-seen timezone on the `Aviary` row. The tick uses that cached value when computing day/night phase even if no client is currently connected (which is the normal case — the aviary should keep advancing through the night while everyone's asleep). This is a deliberate, named interpretation: the PRD says "local time" without specifying how the server learns it absent a connected client, and caching the last-observed value is the only option consistent with "the simulation continues server-side whether or not anyone is watching."

### 5.1 Drift function (low-pass filter, monotonic)

For each trait, the tick computes a non-negative delta and adds it to the existing value (clamped at the trait's max). No subtraction path exists anywhere in the tick code — this is enforced by the `PersonalityDelta.delta_value >= 0` constraint at the schema level plus a unit test that asserts the drift function's output is never negative for any input combination.

Weighted inputs, in the order given by `bird_engine.md`:
- **Presence-time** (dominant): `delta_boldness += k1 * presence_seconds`, `delta_social_warmth += k1' * presence_seconds`, spread across traits at a low base rate. Presence-time is global to the aviary (not per-bird) since presence is defined as attention to the aviary as a whole.
- **Listen-in**: `delta_social_warmth[bird] += k2 * listen_in_seconds[bird]`, `delta_vocal_frequency[bird] += k2' * listen_in_seconds[bird]` — attributed to the specific bird being listened to.
- **Offers**: accepting an offer → `delta_curiosity[bird] += k3`; offering near a bird at all (regardless of acceptance) → `delta_boldness[bird] += k3' ` (smaller than k3).
- **Settle**: no drift contribution; only closes the presence window cleanly (sets an explicit `presence_ended_at` so the worker doesn't keep accruing presence-time into a tick after the user has left).

The filter is literally a low-pass filter in the signal-processing sense: each tick's delta is `coefficient * raw_signal_this_tick`, and the coefficient is tuned so that the cumulative sum crosses a "measurable in instruments" threshold (defined as: a 7-day rolling sum of `PersonalityDelta` for a trait exceeds a small fixed epsilon, queryable for test/instrumentation purposes) after about a week of regular visits (defined for calibration purposes as ~20–30 minutes/day of presence), and crosses a larger "visible to user" threshold (a perceptible step in the rendering — see §8 mapping of personality → render parameters) after about three weeks of the same usage pattern. These two thresholds and the `k` coefficients are tunable constants seeded from this target and adjusted via the instrumented 7-day rolling-sum query during a beta/calibration period before launch; they are not hardcoded guesses shipped without a way to verify them.

Plumage saturation drifts from sustained attention specifically (presence-time + listen-in), per the spec's explicit note that plumage "drifts up with sustained attention."

### 5.2 Mood transitions

Mood is a small state machine, not a continuous value, re-evaluated every tick per bird:

Inputs → weighted score per candidate mood state → highest-scoring state wins (with hysteresis: a mood only changes if the winning candidate's score exceeds the current mood's score by a threshold margin, so mood doesn't flicker tick-to-tick on noise).

- Recent interaction signal (this session): an accepted offer biases toward `content`; a long listen-in biases toward `content`/`curious` for that bird.
- Time-of-day: dusk/night biases toward `drowsy`; early morning biases toward `alert`.
- Ambient events: active rain biases the whole aviary slightly toward `wary`/`drowsy` (dampened vocal frequency); wind biases some birds (by personality) toward `alert`, others toward `wary`.
- Personality modulation: high-boldness birds have a damping multiplier applied against any `wary`-biasing input, per the spec's "less likely to enter wary even on the same input."
- Daily-ish reset: once per local day (tracked via `mood_set_at` crossing a local-midnight boundary), the accumulated session-interaction bias is cleared, so mood re-derives mostly from time-of-day/personality at the start of a new day rather than carrying forward yesterday's specific interactions indefinitely — this is the implementation of "resets on a daily-ish cadence" without snapping to a hardcoded default mood.

Mood persists across sessions by construction: it's a stored column updated only by the tick, so a client opening the tab simply reads whatever the last tick wrote — there's no session-start mood-initialization code path to accidentally add.

### 5.3 Bird-to-bird interaction

Implemented as a same-tick, in-aviary pass after individual mood transitions are computed:
- **Call-response propagation**: a bird with `mood == alert` or recent listen-in attention has an elevated chance (weighted by `social_warmth`) of "calling," which is recorded as an ephemeral per-tick flag (not persisted state) consumed by the call-grammar runtime client-side; a high-`social_warmth` bird nearby has an elevated chance of responding within the same or next tick.
- **Wary-mood spread**: if a bird transitions into `wary` this tick, nearby birds (same perch zone or adjacent) get a small additive bias toward `wary` in *their* mood-scoring pass, applied before the hysteresis check so it can tip a close call but won't override a strongly-scored alternative.
- **Chorus eligibility**: when ≥2 birds with high `vocal_frequency` are both in a calling-eligible state in the same tick window, a `chorus_active` flag is set on the snapshot for the client's audio engine to mix accordingly; this is a snapshot-level ephemeral field, not stored history.

### 5.4 Notebook entry generation

A rule-based trigger evaluator (not a free-form LLM call — see note below) runs each tick and checks a small set of named conditions against the just-computed state and recent history:
- First-of-week / first-of-kind events (e.g., "bird X greeted before bird Y today, first time this week" requires tracking each day's first-greeter and comparing against the prior 7 days — a small derived table or rolling cache keyed by `aviary_id, date`).
- Sustained-quiet detection (no calls/interactions logged for an unusually long stretch within a day).
- Notable mood persistence (a bird stays in the same mood for an unusually long stretch).
- Weather-adjacent observations (a passing rain plus a specific bird reaction).

Each trigger has a cooldown/rarity budget (a minimum spacing in days, tuned per trigger type) so that even a highly active aviary doesn't generate an entry every session — directly implementing the "roughly one entry every few days" target. When a trigger fires, the system selects from a library of naturalist-voice templates parameterized by the specific bird names/states involved (not generic event-log strings), satisfying the specificity requirement without requiring a live generative-model call in the hot path (lower latency, lower cost, and fully reviewable/auditable copy before ship — a real concern for a voice this load-bearing). The template library is content the writing/design team owns and iterates on, not something engineering free-types at implementation time.

**Explicit non-implementation**: no trigger, ever, reads or reports visit-frequency, streak length, or "days since last visit" — the trigger evaluator's input set is restricted by construction to aviary/bird state (mood, drift, weather, bird-to-bird events), never to account-level visit-cadence metrics, which structurally enforces the "no streak counter, no disguised version of one" rule from `interactions.md`.

---

## 6. Sync model

Already covered architecturally in §2–§4, but stated as a model:

- **Single writer**: the Simulation Tick Worker is the only process that ever writes `Bird.personality_*` or `Bird.mood`. Clients write only to the append-only `InteractionEvent` log.
- **Conflict-free by construction**: because personality/mood updates are *additive deltas computed from an ordered log*, not *absolute values submitted by clients*, there is no last-write-wins race to resolve. Two devices submitting events concurrently both land in the log; the tick consumes both, in `received_at` order, and the deltas from both apply. Nothing is overwritten.
- **Snapshot consistency**: `GET /aviary/snapshot` always reads the latest committed `Bird`/`Aviary` rows — no client-side caching of personality-affecting state beyond the current snapshot, and no merge logic on the client at all.
- **Propagation latency**: a device's own action (e.g., an offer) is visible to that same device, and to any other connected device, only after the next tick has run and the device's next snapshot poll picks it up — worst case ~tick-cadence + poll-cadence (roughly 60–90s). This is acceptable and arguably correct for the product's pacing (no interaction should feel instantaneous-and-causal in a way that resembles a button-press game), but it should be explicitly communicated to the team so nobody "fixes" it with an optimistic-UI shortcut that would imply client-side state ownership.
- **Re-sync triggers**: client re-pulls snapshot on `visibilitychange` → visible, on detecting a large `requestAnimationFrame` gap (laptop resume), and on the low-frequency keepalive — covering the three ways a client can become stale (backgrounded, suspended, just-idle-too-long) without needing a push channel (no WebSocket/SSE infrastructure needed for v1 given the slow tick cadence; polling is simpler to operate and sufch enough latency is fine).

---

## 7. Frontend rendering pipeline

### Scene composition
Layered composition, foreground-to-background: top-bar chrome (DOM, fades on idle) → birds + perches (the primary interactive/animated layer) → foreground ornament pass (occasional branch/leaf) → background (foliage/sky, subtle parallax) → ambient color overlay (day/night/weather tint).

**Recommendation**: render birds and the scene via a single `<canvas>` (2D context, not WebGL — the visual complexity described doesn't need a GPU shader pipeline, and 2D canvas keeps the bundle smaller and the team's iteration speed faster) with DOM used only for the top-bar chrome and any text overlays (captions, narration-if-visible, settings panels). This keeps accessibility tree concerns (focus, ARIA) on real DOM elements layered transparently over the canvas for the interactive bird hit-targets (each bird gets an invisible, properly-labeled, keyboard-focusable DOM element positioned over its canvas-rendered position, updated each frame — this is how "Tab focuses the first bird, arrow keys move focus between birds" works without reimplementing focus management inside canvas).

### Idle micro-motion
Each bird has a small per-mood motion-state machine (preen, scan, head-tilt, weight-shift) driving a sprite/skeletal animation (procedurally generated or hand-authored small SVG/sprite-sheet assets per species, kept small per the bundle budget). The motion-state machine picks the next idle action with mood-weighted randomness (a `content` bird weights toward preen; a `wary` bird weights toward scan) so behavior is recognizably mood-shaped without being scripted to a fixed loop length — directly serving the "user reads mood from motion without a label" requirement.

### Transitions
Snapshot-to-snapshot interpolation (perch-to-perch movement, mood-state visual changes) uses eased tweening over the tick interval, not teleporting. Flight transitions between perches are a short flight animation (curved path, brief), distinct from idle motion.

### First-frame requirement
The render pipeline must be able to draw a fully "mid-motion" first frame with zero blank/loading frames once the snapshot is available: each bird's initial animation-state-machine state is seeded directly from the snapshot's current mood/action hints (the snapshot includes a `current_action_hint` per bird, e.g. `preening`, `calling`, `idle_scan`, set by the tick) rather than starting every bird at a neutral pose and animating toward state. Before the snapshot arrives (cold cache / slow connection), the quiet-field loading state (soft sky color, one or two faint motion cues — explicitly not a spinner) renders immediately and is replaced in-place by the live scene with no transition animation between them, per `aviary_layout.md`.

### Reduced-motion mode
A structurally separate render path inside the same `SceneRenderer`, not a CSS `animation: none` override: `prefers-reduced-motion` (or the in-app toggle) switches the bird-rendering strategy from frame-by-frame procedural animation to a small set of discrete cross-fadeable pose keyframes per mood/action (preen-pose-1 → preen-pose-2 cross-fade over ~1.5–2s instead of continuous procedural motion). Flight becomes a perch-to-perch cross-fade instead of a flight-path animation. Ambient leaf/feather ornaments are disabled entirely; ambient color (day/night/weather) shifts remain but interpolate more slowly. This mode is built and reviewed as its own designed visual register from day one (assigned to the same visual designer who owns the primary mode), not implemented as a fallback flag late in the project — sequencing detail in §11.

### Top-bar fade
Pure client-side UI state: a few seconds of no pointermove/keydown fades the top bar's opacity via CSS transition; any pointermove/keydown restores it. Independent of the presence-accounting system (§5) — this is purely cosmetic UI state, not a presence signal, even though it uses similar input events.

---

## 8. Audio pipeline

### Procedural call synthesis
Each species has a call-grammar definition: a small library of motifs (short pitch/rhythm/timbre fragments) plus combination rules (which motifs can follow which, typical call lengths, typical pause patterns) authored as data (JSON/small DSL), not as audio files. At runtime, the `AudioEngine` takes a bird's `species_id` + current `mood` + relevant personality-derived timing parameters (vocal_frequency is **not** sent to the client as a raw number — instead the snapshot includes a derived, already-scaled `call_interval_hint` and `call_eagerness_hint` so the client never needs the raw personality value to drive timing, preserving the "never exposed numerically" rule even informally/indirectly) and generates a call via WebAudio oscillators/noise generators + envelope shaping, picking a motif sequence fresh each time so no two calls are byte-identical.

This is implemented as a small client-side synthesis library (oscillator graphs + simple additive/FM synthesis, not sample playback) — the bundle-budget math in `accessibility_perf.md` explicitly rules out recorded audio, so this library is core, not optional.

### Chorus mixing
When the snapshot/ephemeral state indicates `chorus_active` (§5.3) or simply when ≥2 birds independently decide to call in overlapping windows, each bird's call is synthesized through its own independent WebAudio node graph and summed at the `AudioContext` destination (via a mix bus with per-source gain), which is what produces a real chorus rather than the phase-cancellation artifact the PRD calls out for stacked recorded loops — because each graph is generating live signal, not replaying a fixed waveform, there's no fixed-phase relationship to cancel against.

### Listen-in mix decay
A per-bird gain node in the mix bus. Engaging listen-in ramps the focused bird's gain toward 1.0 and all others toward a quieter-but-nonzero ambient floor (e.g. 0.25–0.35, tunable) over ~1–2 seconds using `AudioParam.linearRampToValueAtTime` (or exponential ramp for a more natural perceptual curve) — never a hard `gain.value =` step, which would be the "channel switch" the spec explicitly warns against. Disengaging ramps back to the default balanced mix over the same kind of curve.

### WebAudio fallback
Feature-detect `AudioContext` availability and permission state at startup. If unavailable or denied, the `AudioEngine` simply never initializes its synthesis graph; the rest of the app proceeds normally, captions default to **on** (per the spec), and no recorded-audio path exists anywhere in the codebase to fall back to — this is enforced by there being no audio asset pipeline at all (no `/audio/*.mp3` files shipped), not just a runtime check, so the "no recorded audio, unconditionally" rule can't be quietly violated by someone adding a "just one fallback file."

### Captioning
Caption text is generated by the same call-grammar runtime that drives synthesis, from the same motif-sequence decision, so the caption always describes what was actually played (e.g., motif sequence `[rise, pause, rise]` at `mood=content` maps deterministically to a caption template like "a soft three-note rise"). This is implemented as a caption-template table keyed by `(motif_sequence_shape, mood)`, evaluated at the moment of synthesis, not as a separate guess about what the audio sounds like.

---

## 9. Accessibility surfaces

### Screen-reader narration
A dedicated `GET /aviary/narration` endpoint (see §4) returns server-generated naturalist prose, built from the same canonical snapshot data via a template system structurally similar to the notebook generator (§5.4) but tuned for present-moment description rather than retrospective observation, and on a faster (30–60s idle, faster on user-initiated events) cadence. Delivered to an ARIA live region (`aria-live="polite"` for idle updates, with the queue depth capped at 1 — a new narration replaces rather than queues behind a pending one, since at this slow cadence backlog should never occur, but the cap guards against a burst of user-initiated events flooding the screen reader, which the spec explicitly calls out as the failure mode to avoid).

User-initiated events (return-greeting, offer reaction, settle) get a priority path: the client emits the event and, instead of waiting for the next idle narration cycle, requests an immediate single narration update scoped to that event (a lightweight templated sentence specific to the action, generated client-side from the action type + bird involved, not a full re-fetch of `/aviary/narration`) — this keeps the prompt-response feel responsive without making the idle cadence itself faster (which the spec warns would overwhelm the queue).

### Reduced-motion mode
Covered in §7 as a render-path decision; from the accessibility-requirements side, the key implementation note is that `prefers-reduced-motion` is detected via the CSS media query / matchMedia API at startup and is also independently settable in-app (some users want it without it being their OS-wide default), with the in-app setting taking precedence when explicitly set and persisted per-account (not just per-browser) so it follows the user across devices via the account settings record, consistent with the multi-device-sync model.

### Captioning
Per §8 — generated alongside synthesis, rendered as small fading DOM text near the calling bird's screen position, naturalist voice, opt-in via accessibility settings (with the WebAudio-unavailable case defaulting it on).

### Contrast
Top-bar icons/labels, settings surfaces, account surfaces, error surfaces, and any visible caption/narration text are implemented with the design system's WCAG-AA-passing tokens (not ad hoc colors), reviewed against both the brightest (midday) and darkest (night) aviary background states since contrast needs differ across the day/night cycle for any overlaid text.

### Keyboard navigation
Top bar items are real, tab-order DOM elements. Each bird gets an invisible, ARIA-labeled (`aria-label` built from the same narration vocabulary, e.g. "Pip, perched on the front rail") DOM hit-target positioned over its canvas-rendered location each frame (per §7), reachable via Tab and then arrow-key roving focus among birds (`role="group"` with `aria-activedescendant` or a roving-`tabindex` pattern — implementation detail for the frontend team, either is acceptable). Enter triggers listen-in on the focused bird; Escape exits listen-in; the offer affordance opens via a top-bar button reachable by Tab and is itself a fully keyboard-operable small panel (radio-group-style selection among seed/song/pool, Enter to confirm). Focus indicators are a high-contrast outline defined once in the design system and applied uniformly, tested visually against both bright and dim backgrounds.

---

## 10. Performance budgets and observability

### Budgets (restated as engineering constraints, not just targets)
- **Bundle ≤2MB gzipped**: enforced via a CI bundle-size check that fails the build on regression past budget, with code-splitting applied to account settings, accessibility settings, and the visit-invitation flow (none of which are needed for the critical first-paint path) so the initial chunk contains only: scene renderer, audio engine bootstrap, snapshot client, top-bar shell.
- **<500ms time-to-first-bird on mid-tier mobile/4G**: requires the initial HTML response to carry (or immediately follow with) the first snapshot — implemented via a server-rendered or edge-cached snapshot embedded in the initial HTML payload (avoiding a second network round-trip before the first bird can render) plus the render path drawing directly from that embedded snapshot without waiting on non-critical assets (species sprite assets for off-screen/not-yet-relevant elements can lazy-load; the visible birds' assets are part of the critical path and must be small).
- **60fps idle on a 5-year-old laptop**: enforced via a synthetic performance check (part of the observability fleet, §10.3) running the idle scene on throttled-CPU profiles in CI/staging and alarming on dropped-frame regressions, not just spot-checked manually pre-launch.
- **No memory growth over 30 minutes**: implemented via object pooling for audio buffers (the `AudioEngine` reuses a bounded pool of oscillator/gain nodes rather than allocating fresh per call) and a hard rule that scrolled-out notebook entries release their DOM nodes/listeners (virtualized list rendering for the notebook, not an ever-growing DOM). This is enforced as an actual CI test: a synthetic 30-minute (accelerated/simulated-time) session run in a headless browser with heap-snapshot diffing at intervals, failing the build on sustained growth — per the spec's explicit "real test in CI, not a guideline."

### Observability
- Synthetic checks: scheduled headless-browser runs from multiple geographies hitting the real production path, measuring time-to-first-bird, bundle delivery time, and idle-frame timing.
- Aggregate RUM: page load timings, first-bird-render timings, render-frame timings, audio-context error counts, simulation-tick latency — all aggregate-only, no per-account dimension in the metric labels (enforced by the metrics-emission layer simply not accepting an `account_id` parameter at all, so there's no accidental high-cardinality leak).
- Alarm: simulation-tick p99 latency > 5s pages on-call — this is the single most important alarm in the system, since a stalled tick worker silently breaks "the aviary continues without the viewer" for every account at once.

### Browser support
Last two major versions of Chrome/Safari/Firefox/Edge as the supported matrix; an explicit unsupported-browser detection (feature-detect rather than UA-sniff where possible, e.g. checking for required Canvas/WebAudio/ES features) renders the matter-of-fact unsupported-browser surface instead of attempting a degraded experience.

---

## 11. Rollout

### Sequencing (recommended build order, not a rigid phase gate)
1. **Foundation**: Account Service (magic-link auth, sessions) + Aviary/Bird/Aviary data model + the Simulation Tick Worker skeleton (even with a trivial drift function) — this de-risks the architecturally load-bearing piece (server-as-sole-writer) earliest, since every other surface depends on it being correct.
2. **Core engine**: full drift function, mood transitions, bird-to-bird interaction, calibration-instrumentation queries (the 7-day/3-week rolling-sum checks) — built and tunable before any rendering exists, validated against synthetic event-log fixtures in tests.
3. **Rendering + audio, in parallel with each other but both gated on #2**: scene renderer (full-motion mode first), procedural audio engine, snapshot polling/interpolation client.
4. **Reduced-motion mode**: built as its own visual register immediately after full-motion rendering stabilizes, not deferred — per the explicit "ship with v1, not v1.1" instruction, this is scheduled inside the same milestone as primary rendering, with its own design review pass, not bolted on afterward.
5. **Accessibility narration + captions + keyboard nav**: built alongside rendering (they share the snapshot/template infrastructure with the notebook generator), reviewed by accessibility-focused QA before any beta.
6. **Field notebook + offer/listen-in/settle interactions**: layered on top of the now-stable engine + rendering.
7. **Visit-invitation feature**: built last among in-scope features since it's additive and doesn't gate the core single-user experience, but still ships in v1.
8. **Performance hardening pass**: bundle-size audit, CI perf gates, memory-growth test, synthetic monitoring setup — done as a dedicated pass before GA, informed by real measurements against the budgets in §10, not assumed correct from architecture alone.

### Birds-per-aviary ramp
Two starter birds at adoption is fixed at launch (not ramped — it's a day-one product decision, not a rollout lever). What ramps is the **age-gated third-bird-and-beyond offer** (`bird_engine.md`): this is implemented as a simple aviary-age-threshold table (e.g., third bird available at N weeks, fourth at N+M weeks, etc.) that the team can tune post-launch by adjusting thresholds (a config value, not a code change) once real usage data shows whether the "few months → third bird, year → five or six" pacing feels right. This is explicitly flagged as a parameter the team should expect to revisit based on early-cohort behavior, with the threshold table designed to be hot-tunable (a config row, read by the offer-eligibility check) rather than hardcoded into the tick worker.

### What we instrument from day one
- The calibration queries from §5.1 (7-day/3-week drift-visibility thresholds) — needed to verify the core promise is true before/at launch, not discovered after users complain.
- Aggregate performance RUM (§10) — from the first deployed environment, not added post-GA.
- Notebook-entry generation rate per aviary (aggregate, to verify the "roughly one entry every few days" target is actually being hit by the trigger-cooldown tuning, not just intended).
- Visit-invitation funnel (aggregate: invites sent, redeemed, revoked) — operational health only, never which specific accounts are involved beyond what's needed for the host's own visit log.

### Launch gating
Do not launch without: reduced-motion mode complete and reviewed, screen-reader narration complete and reviewed, the memory-growth CI test green, the bundle-size budget green, and the calibration queries showing the drift function lands within the targeted instrumented/visible windows on test cohorts. These are explicit go/no-go gates, not nice-to-haves, because retrofitting any of them post-launch contradicts the PRD's own stated reasoning for why each must ship with v1.

---

## 12. Risks

### Drift calibration
**Risk**: the low-pass filter coefficients are guesses until real usage data exists; shipping with miscalibrated `k` values either makes drift invisible (the "screensaver" failure named in `bird_engine.md`) or too fast (the "Tamagotchi" failure). **Mitigation**: the `PersonalityDelta` audit table and the 7-day/3-week instrumented queries (§5.1, §11) exist specifically so this is measurable pre-launch against synthetic/beta cohorts and tunable post-launch without a schema or architecture change — coefficients live in a config table, not embedded in tick-worker code.

### Sync correctness
**Risk**: the additive-delta/append-only-log model is conceptually conflict-free, but a bug in tick-worker idempotency (e.g., a crash between writing `Bird` updates and marking events consumed) could double-apply a delta on retry. **Mitigation**: the single-transaction-per-aviary design (§5) makes the write and the consumption-marking atomic; additionally, a periodic reconciliation job can recompute a bird's personality from the full `PersonalityDelta` history and compare against the stored running value, alarming on drift between the two (a cheap correctness check given deltas are already an audit log, not an extra system).

### Audio uncanniness
**Risk**: procedural synthesis that sounds obviously synthetic (a "MIDI bird") is arguably worse than the looped-audio failure mode it's meant to avoid — the PRD sets a high bar ("real chorus, not stacked loops") without specifying acceptable audio quality. **Mitigation**: this is the single highest-uncertainty, least spec-determined piece of the build; recommend an early audio-prototype spike (before full engine integration) specifically to validate that the oscillator/noise-based synthesis approach clears a believability bar with real listeners, with an explicit fallback design conversation (richer synthesis techniques, e.g. granular synthesis or physical-modeling-style synthesis of a vocal tract, vs. the simpler additive/FM approach assumed in §8) budgeted into the schedule rather than discovered as a crisis late.

### Accessibility regressions
**Risk**: because narration, captions, and reduced-motion are designed surfaces built in parallel with the primary visual/audio path (not generated mechanically from it), they can drift out of sync with engine changes over time — e.g., a new mood-transition rule ships and the narration template library isn't updated to reflect it, leaving screen-reader users with stale-feeling descriptions. **Mitigation**: narration/caption template coverage should be part of the engine's own test suite (a test asserting every mood/action/event combination the engine can produce has a corresponding template), not a separately-owned accessibility checklist that can silently fall behind; this is a process risk as much as a technical one and should be named explicitly in engineering review norms, not just in this plan.

### Gamification creep
**Risk**: `non_goals.md` names this explicitly as the most likely failure mode — a well-intentioned future feature (a "harmless" milestone celebration, a visible visit-frequency widget) reintroducing exactly what the architecture was built to refuse. **Mitigation**: structural, not just cultural — the data model and API surface (§3–§4) have no fields or endpoints that expose visit-frequency, streak length, or counts to the user, and the notebook/narration trigger evaluators are restricted by construction to aviary-state inputs (§5.4). Any future feature in this direction requires a deliberate schema/API change and is not reachable by toggling a flag or adding a UI element against existing data — this plan treats that friction as intentional and load-bearing, not a gap to be smoothed over later.
