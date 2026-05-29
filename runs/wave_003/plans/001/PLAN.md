# Pocket Aviary — v1 Implementation Plan

This plan interprets the PRD into an executable build for a frontier engineering team. It is organized so each section makes engineering decisions (stack, schemas, algorithms, endpoints, invariants, tests) rather than re-narrating the spec. Where the PRD's affective rules are load-bearing, they are translated into **invariants with enforcement** — because in this product the "feel" constraints are the architecture, not decoration on top of it.

A short reading note: most products treat "no gamification," "notice never announce," and "presence is honest" as policy. Here we treat them as structural properties — features that are *not built* so they cannot later leak in, and invariants enforced at the database/role/CI layer rather than at code review. That stance drives several otherwise-surprising decisions below; each is flagged.

---

## 0. Engineering invariants (the load-bearing rules)

These are extracted to the top because they cut across every section. Each has a named enforcement mechanism so it survives future contributors.

| # | Invariant | Enforcement |
|---|---|---|
| I1 | **Server is the sole writer of personality vectors and canonical mood/state.** Clients never write personality. | The simulation-tick DB role has `UPDATE` on personality columns; the API DB role does **not** (column-level grant). A client write path that touches personality fails at the DB, not at review. |
| I2 | **Personality updates are additive, server-authored deltas applied in event-log order.** No last-write-wins, ever. | Tick consumes the append-only log by monotonic per-account sequence; applies `v += Δ`; never `v = absolute`. Event processing is idempotent (dedup on event id + tick watermark). |
| I3 | **Drift is monotonic toward expressive.** Traits increase on positive presence/interaction; they never decrease on neglect. | Drift function returns only non-negative deltas (clamped `Δ ≥ 0`). Neglect → no delta. Property test asserts no code path produces a negative personality delta. |
| I4 | **Presence is the 3-condition conjunction.** `visibilityState === 'visible'` AND `document.hasFocus()` AND pointer/key activity within the activity window. All three, simultaneously. | A single `PresenceDetector` module is the only emitter of `presence_ping`. "tab open" alone never emits. Server rate-limits/sanity-checks ping cadence. |
| I5 | **Personality vector is never exposed numerically** anywhere, any tier, any debug view. | No API field, DTO, ARIA attribute, or log line carries raw trait values to the client. Lint rule forbids serializing the `personality_vector` type into any client-facing schema. |
| I6 | **Email lives in exactly one place, encrypted; every other reference is the synthetic account UUID.** | UUID generated at account creation is the only key in FKs, partitions, logs, telemetry, inter-service messages. Email is an encrypted column on `account` plus a keyed-HMAC lookup index (see §4.1). Log scrubber + schema check reject email-shaped or raw-email fields in any non-account table. |
| I7 | **Per-bird / per-account interaction data never enters analytics, training, or population-level computation.** | Telemetry pipeline has no network path/credential to the simulation DB; simulation DB is not a source in the analytics warehouse; ML jobs cannot reference per-bird fields. Boundary enforced by infra (network policy + IAM), verified in CI infra tests. |
| I8 | **No announcement surface exists.** No toast/banner/welcome/notification/streak/visit-frequency component is in the system. | The design system ships **no** toast/snackbar/badge primitive. The metric that would back a streak is **not computed**. Absence is structural, not a guideline. |
| I9 | **Audio is procedural-only.** No recorded audio is shipped, including as a fallback. | Build has no audio-asset pipeline. WebAudio-unavailable → silence + captions. CI asset check fails the build if any `.mp3/.ogg/.wav` lands in the bundle. |
| I10 | **Stable bird identity.** A bird's internal id is never reused, regenerated, swapped, or "reset" across rename, sync, species-pool changes, or migration. | `bird.id` is immutable; migrations carry it forward; no "regenerate bird" code path exists. |
| I11 | **Naturalist voice everywhere except system surfaces.** Sign-in, account, sync errors, accessibility settings, visit-unavailable, unsupported-browser use matter-of-fact voice; everything else naturalist. | Copy lives in two namespaced catalogs (`voice.naturalist.*`, `voice.system.*`). System surfaces may only pull from `voice.system.*`; lint enforces the namespace per surface. |
| I12 | **Accessibility is a designed surface shipped with v1**, not a fallback retrofitted later. | Reduced-motion, narration, captions, keyboard nav are launch-blocking acceptance criteria, in the same milestone as the default visual path. |

If a future change appears to require violating one of these, that is a product decision escalated to the PRD owner — not an engineering judgment call.

---

## 1. Scope

### In v1
- Single-user accounts; magic-link sign-in; per-device revocable sessions; email-change with verify; account export (JSON, emailed link); soft-delete (30 days) → hard-delete.
- One canonical aviary per account. 2 starter birds (system-selected species, user-named), cap 7. Third+ birds offered by **aviary age**.
- Bird engine: hidden personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity); fast-timescale mood; monotonic-up drift; bird-to-bird interaction; procedural call grammar.
- Server-side simulation tick (~1/min) as the only writer of canonical state.
- Client: single horizontal scene (3 perch zones, day/night by local time, rare ambient weather, parallax, ambient leaf/feather drift); first frame mid-action; quiet-field load/empty state; thin auto-fading top bar (account/settings, accessibility, notebook, offer).
- Interactions: return-greeting (absence/boldness/mood-shaped, procedurally varied, staggered), listen-in (slow mix re-balance, never mute), offer (seed / song-fragment / still-pool, per-bird cooldown), settle (with 5s undo), field notebook (sparse, read-only, naturalist).
- Multi-device sync (a property of server-authoritative state, not a feature).
- Visit invitations: per-invite opt-in (email), read-only ambient, revocable, 30-day expiry, silent visit log, off-by-default visit notifications.
- Accessibility: screen-reader naturalist narration, reduced-motion mode (designed), call captions, full keyboard nav, WCAG AA on all user copy.
- Performance: bundle <2MB gzip; time-to-first-bird <500ms (mid-tier mobile/4G); 60fps idle on 5-yr-old laptop; no memory growth over 30 min; procedural WebAudio with silence+captions fallback.
- Aggregate-only operational telemetry + synthetic perf checks.

### Explicitly out (non-goals respected)
- No native app; data model/protocols are **not** shaped for a future native client.
- No gamification of any flavor (achievements, streaks, levels, scores, badges, "birds adopted: N", visit calendars, XP) — **not even computed** (I8).
- No Tamagotchi mechanics (no death, hunger, distress, decaying happiness meter); neglect → ambient quietness via mood/recent-activity, never negative personality drift.
- No social-network surfaces (profiles, follows, feeds, discovery, leaderboards, comments, co-presence, friend-of-friend); the single visit affordance is the entire social surface.
- No notification/push/email-about-the-aviary surface; no "welcome back" text of any kind.
- Personality numbers never surfaced (I5).

### Defensible calls made under ambiguity (noted, per PRD instruction)
1. **Email HMAC lookup index** to reconcile "email never an identifier" (I6) with "sign in by email." See §4.1.
2. **Client-local call scheduler driven by server-provided parameters** (audio runs locally for latency; server still owns personality/mood). See §5.5 / §7.
3. **Canvas2D renderer + Preact chrome** as the bundle-safe default; PixiJS/WebGL only if it fits the budget. See §6.
4. **Drift calibration via a synthetic-agent harness, never prod per-bird data** (forced by I7). See §5.3 / §10.
5. **Postgres for canonical state and the append-only event log** at v1 scale (not Kafka). See §2 / §3.
6. **Activity window initial value ≈ 3–4 min, leaning long**, pending calibration. See §5.2.
7. **Edge-inlined initial snapshot** to make the <500ms first-bird budget reachable. See §2.3 / §6.1.

---

## 2. Architecture

### 2.1 Service shape
Three deployables plus datastores and an isolated telemetry plane.

- **Edge/CDN layer.** Serves the HTML shell, critical CSS, the tiny first-bird boot script, and an **inlined initial snapshot** for the signed-in account (read from a short-TTL snapshot cache, keyed by account UUID, gated by the session cookie at the edge). Static assets + code-split chunks served from CDN.
- **API service** (stateless, horizontally scalable). Auth (magic link), snapshot reads, event-log writes, settings, visits, export/deletion initiation, narration text endpoint. Holds the **API DB role** (no personality `UPDATE`, per I1).
- **Simulation tick worker** (the only writer of canonical personality/mood/state, per I1). Runs the tick loop, partitioned by account UUID. Holds the **tick DB role** (the only role with personality `UPDATE`).
- **Mailer** (magic links, export links, visit invites, optional visit notifications). Templated copy from `voice.system.*`.
- **Telemetry plane** (separate account/VPC/credentials). Receives only aggregate metrics. No route to the simulation DB (I7).

Rationale: a strict writer/reader split is what makes "server-authoritative personality" a real property rather than a label (PRD `accounts_sync.md`). Splitting the tick into its own deployable with its own DB grants is the cheapest way to make I1/I2 unfalsifiable.

### 2.2 Client/server split
- **Server owns**: canonical personality vectors, mood, perch intent, weather, day/night derivation, drift, mood transitions, notebook generation, presence accounting, call *parameters* (cadence, pitch center, ornamentation budget), greeting *decisions* (which bird, absence bucket). The tick is the single mutator.
- **Client owns**: rendering, interpolation between snapshots, the local **call scheduler** + WebAudio synthesis (procedural, latency-sensitive), idle micro-motion, ambient ornaments (leaves/feathers — no sim state), caption generation, narration display, presence detection + event emission.

The boundary rule: anything that must be canonical and survive across devices/absence lives server-side; anything latency-sensitive and ephemeral (the exact audio waveform, this frame's wing position, a drifting leaf) is client-local but **driven by** server parameters so two devices feel like the same aviary without per-frame chatter.

### 2.3 Render pipeline boundary (the <500ms conceit)
The first bird must be visible from the inlined snapshot **before** the full bundle or audio initializes. Two-stage hydration:

1. **Critical path (~tens of KB):** HTML shell + critical CSS + boot script that reads the inlined snapshot, draws the first frame (birds mid-action at their current perches/poses) to the canvas, and starts a minimal rAF loop. No spinner; if the snapshot is absent/slow, render the **quiet field** (soft sky + one or two faint motion cues).
2. **Full hydration (lazy):** load the full renderer, interpolation, call scheduler/WebAudio, top-bar chrome, and (code-split) settings/accessibility/visit flows. Audio starts on first user gesture or autoplay-allowed context; until then, the scene is alive visually and captions can stand in.

This split is the architectural expression of "the aviary has been continuing without the viewer" (`aviary_layout.md`): the user never sees a load state, because the first paint *is* the aviary.

---

## 3. Data model

Postgres. All FKs and partition keys are the synthetic `account.id` UUID (I6). Personality columns are writable only by the tick role (I1).

### 3.1 Core tables

**account**
- `id UUID PK` — synthetic, generated at creation; the only identifier used anywhere else.
- `email_ciphertext BYTEA` — AES-GCM encrypted; the **only** place email is stored.
- `email_hmac BYTEA UNIQUE` — HMAC-SHA256(normalized_email, server_key); the lookup index for sign-in only (not a partition/shard key, never logged). Defensible call (§4.1).
- `timezone TEXT` — for day/night + mood time-of-day.
- `status TEXT` — `active | pending_deletion`.
- `deletion_requested_at TIMESTAMPTZ NULL`.
- `created_at TIMESTAMPTZ`.
- `settings JSONB` — reduced_motion override, captions on/off, audio on/off, visit_notifications (default false), privacy ack.

**session**
- `id UUID PK`, `account_id UUID FK`, `token_hash BYTEA`, `device_label TEXT` (coarse, no fingerprinting), `created_at`, `last_seen_at`, `revoked_at TIMESTAMPTZ NULL`.

**magic_link**
- `id UUID PK`, `account_id UUID NULL` (null for new-signup pending), `pending_email_hmac BYTEA NULL`, `token_hash BYTEA`, `purpose TEXT` (`signin | email_change`), `created_at`, `expires_at` (created_at + 15 min), `consumed_at TIMESTAMPTZ NULL`.

**bird**
- `id UUID PK` — stable forever (I10).
- `account_id UUID FK`.
- `species_id SMALLINT` — from the ~6-species pool.
- `name TEXT` — user-assigned, renameable.
- `adopted_at TIMESTAMPTZ`.
- **personality_vector** (tick-write-only): `boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity` as `REAL` in `[0,1]` (saturation may have a floor > 0 so new birds aren't colorless).
- **drift accumulators** (tick-write-only): per-trait slow EMA accumulators (`acc_*`) used by the low-pass filter (§5.3).
- **mood**: `mood TEXT` enum, `mood_since TIMESTAMPTZ`, `mood_dwell_until TIMESTAMPTZ` (hysteresis floor).
- **perch**: `perch_zone TEXT` (`front|middle|back`), `perch_sub REAL` (render hint).
- `call_seed BIGINT` — per-bird RNG seed for procedural call continuity (advanced, never reset).
- `last_seen_recent_presence REAL` — recent-activity term feeding behavior (greeting frequency), distinct from personality (so neglect quiets behavior without moving personality; see §5.3).

**aviary** (1:1 with account; could be columns on account, kept separate for clarity)
- `account_id UUID PK/FK`, `age_days` (derived from `account.created_at`), `weather_state JSONB` (`type, started_at, ends_at` — short-lived), `settled BOOLEAN`, `settled_since TIMESTAMPTZ NULL`, `next_species_offer_at TIMESTAMPTZ` (age-tied scheduler), `tick_seq BIGINT` (canonical version), `last_tick_at`.

**event_log** (append-only; the spine of I2)
- `account_id UUID`, `seq BIGINT` — per-account monotonic sequence (`(account_id, seq)` PK).
- `bird_id UUID NULL`.
- `type TEXT` — `presence_ping | listen_in_start | listen_in_end | offer | settle | settle_undo | adopt | rename | greeting_ack`.
- `payload JSONB`, `client_ts`, `server_ts`.
- `event_uuid UUID UNIQUE` — client-generated, for idempotent dedup.
- No `UPDATE`/`DELETE` grant for anyone except retention jobs. Tick reads forward from a per-account watermark.

**notebook_entry**
- `id UUID PK`, `account_id UUID FK`, `created_at`, `text TEXT` (naturalist prose), `topic TEXT` (internal only, never shown), `window_ref` (which observation window produced it). Read-only to users (no edit/delete API).

**visit_invitation**
- `id UUID PK`, `host_account_id UUID FK`, `visitor_email_ciphertext BYTEA`, `visitor_email_hmac BYTEA`, `token_hash BYTEA`, `created_at`, `expires_at` (+30 days), `revoked_at NULL`, `first_used_at NULL`, `last_used_at NULL`, `status TEXT` (`outstanding|active|revoked|expired`).

**visit_session** (for the visit log only; never feeds drift)
- `id UUID PK`, `invitation_id UUID FK`, `started_at`, `ended_at`, `approx_duration_s INT`. Render-only; no interaction/presence events recorded for visitors (I7-adjacent: protects host drift from visitor attention).

### 3.2 Notes
- Presence-time is **not** a stored per-event blob to be mined; the tick folds `presence_ping`s into the drift accumulators and a bounded recent-presence term, then the raw pings age out under retention. This keeps the privacy surface small.
- Backups + PITR are configured specifically to protect personality vectors (losing one = deleting the bird the user knows; PRD `bird_engine.md`).

---

## 4. API surface

REST + JSON over HTTPS; session cookie auth (httpOnly, SameSite=Lax). All identifiers in payloads are UUIDs. Snapshots are kilobytes.

### 4.1 Auth (matter-of-fact voice; I11)
- `POST /auth/request-link {email}` → always 200 (no account enumeration); mailer sends a link if the email maps to an account or a new signup. Per-email rate limit.
  - Lookup: normalize email → `HMAC(email)` → match `email_hmac`. **Defensible call:** sign-in inherently needs to find an account by email, which requires *some* index. A keyed HMAC is a one-way, non-reversible index that (a) is not derived-from-email in the sense the PRD forbids (it's not an FK/partition/shard/log key and never leaves the account table), and (b) keeps the plaintext email in exactly one encrypted column. This satisfies the spirit of I6 — "email lives in one place; every other reference is the UUID" — while making magic-link auth possible. Flagged for PRD-owner confirmation.
- `POST /auth/consume {token}` → validates token_hash, not expired (15 min), not consumed; issues a session; marks link consumed (single-use). Replay → matter-of-fact "link may have expired."
- `POST /auth/email-change {new_email}` (authed) → verify-new-before-switch; old email works until new verifies.
- `GET /account/sessions` / `POST /account/sessions/{id}/revoke`.

### 4.2 Aviary state (naturalist surfaces consume this; the data itself is neutral)
- `GET /aviary/snapshot` → `{ tick_seq, server_ts, day_night_phase, weather, settled, birds: [{ id, species_id, name, mood, perch_zone, perch_sub, plumage_saturation, vocal_params, call_seed, call_phase, pending_greeting?: {absence_bucket} }] }`. **No raw personality traits** (I5) — only the render/audio-derived params the client needs. Served inlined at the edge on first load (§2.3) and via this endpoint on subsequent pulls.
- Pull triggers (client): on load (inlined), on `visibilitychange→visible`, on long render-frame gap (suspend/resume), and a low-frequency keepalive while visible.

### 4.3 Interaction events (append-only; clients never write personality, I1)
- `POST /events` (batched) → array of `{event_uuid, type, bird_id?, payload, client_ts}`; server stamps `server_ts`, assigns per-account `seq`, dedups on `event_uuid`. Accepts `presence_ping`, `listen_in_start/end`, `offer`, `settle`, `settle_undo`, `rename`, `greeting_ack`. The tick consumes them next pass. Server sanity-checks ping cadence/rate (I4) and rejects implausible flooding.
- `rename` updates `bird.name` directly (not personality) — names have no engine effect.

### 4.4 Notebook / narration
- `GET /notebook?before=cursor` → paginated entries (indefinite scrollback; old entries never archived). Read-only (no write endpoints).
- `GET /narration` → current running naturalist narration string (same state, same voice as notebook) for screen-reader polling, OR pushed via the snapshot/SSE channel; cadence 1 per 30–60s idle, prompt on user-initiated events.

### 4.5 Offers / settle
- Offers and settle are submitted as events (§4.3). Per-bird offer cooldown (a few minutes) enforced server-side; client also reflects it. Settle sets `aviary.settled=true`; `settle_undo` within 5s reverses (client-side optimistic reversal + event).

### 4.6 Visits
- `POST /visits {visitor_email}` (authed host) → create invitation, mailer sends one-time link. Off by default (host must act).
- `GET /visit/{token}` (unauthed visitor) → returns a **read-only** snapshot stream of the host's current aviary; no event endpoints exposed to this token; visitor cannot greet/listen-in/offer/settle/scroll-the-host's-notebook. A `visit_session` row is opened for the log.
- `POST /visits/{id}/revoke` → immediate; visitor's next snapshot pull returns matter-of-fact "visit no longer available."
- `GET /account/visits` → visit log (visitor email, date, approx duration, outstanding invites); reachable on demand, no badge/notification (I8). Optional per-account `visit_notifications` toggle (default off).
- Invitations expire after 30 days unused; revoked/expired links return the matter-of-fact surface.

### 4.7 Export / deletion
- `POST /account/export` → generates JSON snapshot (birds, names, current vectors *as the user's own data export* — this is the one place vector values leave the system, to the **owner only**, as data portability; still never rendered as a stat UI, preserving I5's intent which is about in-product surfacing) + moods + notebook + settings; emailed download link to the verified address.
  - **Defensible call:** I5 forbids *surfacing personality numbers in the product UI*. An owner-only data export is data portability, not a stats panel. If the PRD owner reads I5 as "the user must never obtain the numbers at all," the export omits raw traits and includes only names/species/moods/notebook. Flagged.
- `POST /account/delete` → soft-delete (status=pending_deletion, timestamp). Any signed-in page shows a quiet "I changed my mind" restore for 30 days. A scheduled job hard-deletes after 30 days (birds, vectors, notebook, events, telemetry linkage — everything).

---

## 5. Simulation engine design

### 5.1 The tick
A worker loops at ~1/min (configurable). For each account due for a tick (sharded by UUID):
1. Read new `event_log` rows forward from the account's watermark, **in `seq` order**.
2. Derive day/night phase + time-of-day signals from `account.timezone`.
3. Maybe start/advance/end an ambient weather event (rare; a few times/week rain, occasional wind).
4. **Mood transition** per bird (§5.4), with hysteresis and bird-to-bird coupling.
5. **Drift** per bird (§5.3): fold presence/interaction signals into accumulators; apply additive, non-negative deltas (I3) to personality vectors.
6. Update perch intent, call parameters, greeting decisions.
7. Maybe emit a **notebook entry** (§5.6) — sparse.
8. Advance `tick_seq`, write canonical state, advance watermark. Idempotent (re-running a tick with no new events is a no-op on personality).

The tick runs whether or not a client is connected (PRD: "continues without the viewer"). Latency p99 alarm at 5s (`accessibility_perf.md`).

### 5.2 Presence accounting (I4)
- Client `PresenceDetector`: maintains `lastActivityAt` (updated on `pointermove`/`keydown`); on a cadence, emits `presence_ping` **iff** `document.visibilityState==='visible' && document.hasFocus() && (now - lastActivityAt) < ACTIVITY_WINDOW`.
- `ACTIVITY_WINDOW` initial ≈ 3–4 min, **leaning long** because watching without moving is the actual product (`interactions.md`); final value set by calibration. Pings carry the asserted condition flags; the server rate-limits and discards implausible bursts.
- When hidden, the client stops rAF (battery) and stops pinging; the server keeps ticking. On return, a fresh snapshot is pulled (the aviary that *kept running*, not a frozen-and-resumed one).
- The tick converts ping density into presence-time minutes feeding drift; "tab open" can never inflate it (I4) because a backgrounded/unfocused/idle tab emits no pings.

### 5.3 Drift function (the calibration the PRD pins on us)
Per trait `t`, per tick:
```
signal_t       = weighted accumulation of relevant inputs since last tick
acc_t          = acc_t * (1 - α) + α * signal_t          # slow low-pass (EMA)
Δv_t           = k_t * acc_t * (1 - v_t)                 # diminishing returns toward ceiling
Δv_t           = max(0, Δv_t)                            # MONOTONIC UP (I3)
v_t            = clamp(v_t + Δv_t, floor_t, 1)
```
- **Inputs by weight** (`bird_engine.md`): presence-time (dominant) → all traits, especially plumage saturation; listen-in minutes (strong) → social_warmth, vocal_frequency on the focused bird; offer accepted (small) → curiosity; offering near a bird (small) → boldness; settle → no directional drift (clean window end only).
- **α small** so no single session moves a trait visibly; weeks accumulate. **k_t** calibrated to the named targets: instrument-detectable Δ after ~1 week of regular visits, user-visible Δ after ~3 weeks (`bird_engine.md`). "User-visible" means crossing a behavior threshold (greeting-first probability, front-perch tendency, plumage saturation) — not a number.
- **Plumage saturation** rises with sustained presence and never falls (a direct render parameter).
- **Neglect ≠ negative drift.** A bird that's ignored keeps its personality; what drops is the **recent-presence behavior term** `last_seen_recent_presence`, which gates *greeting frequency / liveliness* in behavior (§5.4). So a two-week absence yields birds that are **quieter, not warier** — the asymmetry that is the engine-level implementation of "no Tamagotchi" (`bird_engine.md`, `non_goals.md`). Property test: no input combination yields `Δv_t < 0`.
- **Calibration harness** (forced by I7): a synthetic-agent simulator drives compressed-time presence/interaction patterns against the real drift code and asserts the ~1wk/~3wk targets. Calibration uses **synthetic agents and a consented internal cohort only — never mined prod per-bird data** (I7). This is flagged because the obvious "tune from the population's drift curve" is exactly the privacy violation the PRD forbids.

### 5.4 Mood transitions
- Enumerated states: `wary, content, curious, drowsy, alert`, plus night `settled/sleeping`.
- Each tick computes mood "pressures" from: recent interactions (offer accepted → content; nearby alarm/wary → wary), time-of-day (drowsy near dusk, alert early morning, settled at night), ambient weather (rain → dampen vocal; wind → some alert, some wary), and the bird's personality (high boldness resists wary; high vocal resists drowsy quieting).
- **Hysteresis:** a `mood_dwell_until` floor prevents per-tick flapping; transitions require pressure to exceed a margin and dwell to elapse → mood reads as persistent, not snapping.
- **Persistence across sessions:** mood at session-start = stored mood advanced by the tick during the interim (drowsy-at-dusk → settled/sleeping by morning → waking content). The client never resets mood on tab open; it only reads the stored value (so no "snap to neutral").
- **Bird-to-bird coupling:** a wary mood spreads to neighbors with a damped factor; co-calling high-vocal birds raise each other's chorus-join probability — the aviary behaves like a small social system, not independent NPCs (`bird_engine.md`).
- **Idle motion is mood-shaped** and consumed by the renderer (§6): wary = further back + scanning, content = preening, curious = head-tilt toward sounds/leaves, drowsy = low + fluffed. The user reads mood from motion with **no label/tooltip/icon** (`bird_engine.md`).

### 5.5 Call-grammar runtime (server params + client synthesis)
- Per species: a **motif library** (note-shapes: rise, two-note, trill, sharp), a default pitch range, and a timbre profile — the species **signature**.
- Per bird: a stable pitch center + timbre offset (recognizable signature) + a `call_seed` advanced per call.
- The **server** sets call *parameters* in the snapshot: cadence (from vocal_frequency), chorus-join propensity, mood-shaping (drowsy = slower/lower, alert = sharper), pitch center. The **client** runs a local scheduler that, between snapshots, decides exact call onsets and synthesizes them procedurally (§7), drawing parameter jitter from `call_seed` so calls are **never identical twice yet always recognizably this bird** (`bird_engine.md`). **Defensible call:** putting the exact-onset scheduler client-side keeps audio latency-free and avoids per-call server chatter; the server still owns vocal_frequency/mood (personality stays canonical). Two devices feel like the same aviary because they share parameters + seed.
- **Recognizability is a test target:** a listener (and an automated classifier) should ID a bird across mood and drift. This is what makes 7 the cap.

### 5.6 Notebook generation (sparse, naturalist, observations-of-the-aviary-only)
- A generator runs in the tick, gated to **rarity**: ~1 entry per few days for a regular aviary, more on noteworthy moments, **never per-session** (`interactions.md`). A sparsity governor caps frequency even for very active users.
- Entries are templated naturalist prose parameterized by real moments ("pip greeted before wren today, first time this week"; "wren is fluffed against the cool air, watching the back perch. low calls only").
- **Hard rule (I8 / `interactions.md`):** the notebook writes **observations of the aviary**, never **observations of the user's behavior**. No "you visited every day this week," no session timestamps, no event-log dumps. The generator has no access to visit-frequency aggregates because that metric is **not computed**.

---

## 6. Frontend rendering pipeline

### 6.1 Composition
- **Renderer:** Canvas2D primary (defensible call: smallest bundle, ample for ≤7 birds + subtle parallax + cross-fades; meets 60fps on old hardware). Birds drawn as **parametric vector shapes** so `plumage_saturation` is a continuous render input (drift shows up as richer color over weeks) and so reduced-motion pose cross-fades are cheap. PixiJS/WebGL is an upgrade path **only if** it fits the 2MB budget.
- **Layers:** background (day/night sky gradient, soft foliage) → mid (3 perch zones, birds) → foreground (occasional branch/leaf, subtle parallax — not parallax-heavy).
- **Top bar:** thin, sparse — account/settings, accessibility, notebook, offer. **No UI chrome inside the scene.** Fades to near-transparent after a few seconds of cursor stillness; returns on cursor/keyboard. DOM/HTML (for a11y + WCAG AA contrast + keyboard nav), not canvas.

### 6.2 Loop, interpolation, micro-motion
- rAF loop; interpolate bird positions between snapshots (perch A→B is an eased flight path, never a teleport). Suspend rAF when hidden; resume + fresh snapshot on visible.
- **Idle micro-motion** is continuous and mood-shaped (§5.4): preen, scan, head-tilt, weight-shuffle — procedurally driven so it never reads as a paused loop. This is the visible surface of mood; if a bird ever looks "paused," that's a bug against the headline principle.
- **Ambient ornaments** (leaves/feathers) are pure client-side, idle-cadence, **no sim state** (`aviary_layout.md`).
- **Transitions:** return-greeting rendered per server decision (which bird, absence bucket) shaped by boldness/mood — glance / two-note call / step-forward / longer call+response; multiple greeters **stagger** by a small randomized offset (never unison — unison would "announce"). Day/night palette interpolates gradually from local time; weather is short and never assertive.

### 6.3 No load state; first frame is the aviary
- First paint = birds mid-action from the inlined snapshot (§2.3). **No spinner, no fade-from-static, no entry animation.** Slow snapshot → **quiet field** (soft sky + faint motion cues), not a spinner. Empty-aviary (post-adoption, pre-first-bird) uses the same quiet field; the first bird then **fly-ins** softly to its perch; the user never sees an empty aviary again.

### 6.4 Reduced-motion mode (a designed surface, not "animations off"; I12)
- Trigger: `prefers-reduced-motion` OR settings opt-in.
- Micro-motion → slow cross-fades between still poses; flight → cross-fade between perches; ambient leaf drift removed; day→evening color shift **remains, slowed**. Calls still play/caption; **drift, mood, notebook all still function.** It has its own calm aesthetic — a reduced-motion user gets a *calmer* Pocket Aviary, not a broken one. Built and QA'd in the same milestone as the default path.

### 6.5 Responsive scene
- One horizontal scene at any viewport; **no pan/scroll/zoom.** Narrow phone compresses horizontally **without cropping any bird out**; wide desktop spreads perches. Aspect handling keeps all birds visible at all times.

---

## 7. Audio pipeline

- **Graph:** per-bird synth chain (oscillators / light FM + a touch of filtered noise for breathiness + ADSR gain) → **per-bird mix gain** (listen-in control) → master bus → destination. Bounded node count; **buffers/nodes reused** (no per-call allocation) → supports the no-memory-growth budget (§8).
- **Scheduler:** lookahead pattern on the WebAudio clock (schedule motif notes slightly ahead; never block on the main thread). Driven by server call params + `call_seed` (§5.5).
- **Chorus:** emerges from overlapping scheduled calls of high-vocal birds. Because each call is **procedural with real per-call variation**, mixing two calls is a genuine chorus — not two recorded loops phase-cancelling (`bird_engine.md`).
- **Listen-in:** on engage, `linearRampToValueAtTime` the focused bird's mix gain **up** and others **down** over ~1–2s (slow ramp = "listening," not "switching channels"); others **drop but never go silent** (a re-balance, not a mute). Disengage (click focused bird again / focus another / click empty space / keyboard focus away / Escape) ramps back to ambient with the same slow curve.
- **Captions** (opt-in): generated from the **same call-grammar params at synthesis time** so the caption matches what actually played ("a soft three-note rise"; "a low trill, paused, low trill again"). Small text near the calling bird, fading with the call, naturalist voice.
- **Fallback (I9):** no AudioContext (old browser / permission denied / hardware) → **graceful silence with captions on by default.** **No recorded-audio fallback path is ever built** — silence+captions beats canned audio, and recorded audio at the needed variation breaks the bundle anyway.

---

## 8. Accessibility surfaces (shipped with v1; I12)

- **Screen-reader narration:** running **naturalist prose** (not a state list), same voice as the notebook, from the same state the visual reads. Cadence ~1 update / 30–60s at idle; **prompt** on user-initiated events (return-greeting, successful offer, settle). Implemented as **two `aria-live` regions**: a `polite` ambient-narration region (slow) and a separate priority region for user-initiated events (so they announce promptly without flooding the SR queue). "a small grey bird is perched on the front rail, calling softly. another sits further back, fluffed. it is morning in the aviary; the light is gentle." — never "Pip mood: content."
- **Captions:** §7; opt-in; runtime-generated; naturalist.
- **Keyboard navigation:** Tab cycles top-bar items; Tab into the scene focuses the first bird; arrow keys move focus between birds; **Enter = listen-in**, **Escape = exit listen-in**; offer opens via a top-bar shortcut and is fully keyboard-navigable; settle reachable from the top bar. **Focus indicators** are a soft high-contrast outline legible against both bright and dim aviary states.
- **WCAG AA contrast** on **all user copy** (top-bar labels, settings, account/error surfaces, captions, narration-when-shown visually). The scene itself carries no copy except the top bar, so contrast effort concentrates on chrome.
- **Voice continuity:** narration and notebook share the naturalist catalog (`voice.naturalist.*`); a SR user moving between surfaces hears one product, not two.

---

## 9. Performance budgets & observability

### Budgets (all enforced in CI)
- **Bundle <2MB gzip at first paint** → critical-path/full-hydration split (§2.3); aggressive code-splitting of settings/accessibility/visit flows; Preact-class tiny chrome; procedural assets + compact SVG/bitmaps; **no recorded audio** (I9). `bundlesize`-style gate fails the build over budget.
- **Time-to-first-bird <500ms** (mid-tier mobile / 4G) → edge-inlined snapshot + minimal critical render path that draws the first bird before the full bundle/audio; verified by scheduled synthetic runs.
- **60fps idle on a 5-year-old laptop**, sustained over a 30-min session → bounded per-frame work, layer compositing, no layout thrash.
- **No memory growth over 30 min** → reused audio buffers/nodes; virtualized notebook list releasing DOM refs on scroll-out; bounded workers/audio contexts. **A real CI test**, not a guideline.

### Observability (aggregate-only; I7)
- **Synthetic** perf fleet (scheduled browsers from several geographies running the aviary) + **aggregate RUM**: page-load, first-bird-render, frame timings, audio-context error counts, tick latency. **No per-bird / per-account dimension** in any metric; the privacy boundary is honored **at metric definition**, and the telemetry plane has no route to the simulation DB.
- **Alarms:** simulation-tick latency **p99 > 5s**.
- Deliberately **not** measured: anything per-bird/per-account for population analysis; visit-frequency; "engagement" loops. These metrics are not defined, so they cannot leak.

---

## 10. Rollout

1. **Internal dogfood** behind a flag; calibrate `ACTIVITY_WINDOW`, drift `k_t`/`α`, tick cadence via the synthetic-agent harness + consented internal cohort (never prod per-bird mining; I7).
2. **Closed beta** (small invite list) — validate first-bird budget on real mid-tier devices, recognizability of calls, narration voice, reduced-motion feel.
3. **GA** — accessibility (narration, reduced-motion, captions, keyboard) is **launch-blocking** and ships **with** the default path (I12), not after.
- **Birds-per-aviary ramp:** start 2, cap 7; third+ offered by **aviary age** via the `next_species_offer_at` scheduler (config'd pacing curve — e.g., ~3rd within a couple of months, growth toward 5–6 by ~a year), **tunable server-side**. Offers appear in the flow; the user names new arrivals; the mechanic **never** ties new birds to visit count / interaction score / payment (`bird_engine.md`).
- **Instrument from day one:** aggregate ops health, synthetic checks, and the **drift-calibration instruments on synthetic accounts** (not prod users). Feature-flag the ramp pacing, tick cadence, and activity window for safe tuning.

---

## 11. Testing strategy (mapped to the invariants)

- **I1/I2 (server-sole-writer, no LWW):** integration test proving the API DB role **cannot** UPDATE personality columns; ordered-replay test showing two interleaved device sessions both resolve to the same additive result with no lost drift; idempotency test (replayed `event_uuid` is a no-op).
- **I3 (monotonic drift):** property/fuzz test — no input sequence produces a negative personality delta; neglect leaves personality flat while the recent-presence behavior term decays (birds quieter, not warier).
- **I4 (presence):** unit tests that `presence_ping` fires only on the 3-condition conjunction; "tab open but unfocused/idle" emits nothing; server discards implausible bursts.
- **I5 (no numeric exposure):** schema/lint test that no client-facing DTO or ARIA attribute carries raw trait values; the only egress is the owner-only export (and behind the conservative flag if the PRD owner chooses).
- **I6 (PII):** test that no table other than `account` stores email-shaped data; log-scrubber test; partition/shard keys are UUIDs.
- **I7 (privacy boundary):** infra test that the telemetry plane has no credential/route to the simulation DB; metric definitions carry no per-account dimension.
- **I8 (no announcement/gamification):** design-system test that no toast/badge/streak primitive exists; notebook generator has no access to visit-frequency (it isn't computed); copy audit for any "welcome back"/"you visited" string.
- **I9 (procedural audio):** build asset check fails on any audio file; chorus test confirms per-call variation (no identical buffers); fallback path = silence + captions.
- **Drift calibration:** synthetic-agent harness asserts the ~1-week (instrument) / ~3-week (user-visible behavior threshold) targets.
- **Call recognizability:** automated classifier + listening test that a bird is identifiable across mood and drift.
- **Performance:** CI gates for bundle size, synthetic first-bird <500ms, 30-min memory-growth, frame-rate on a throttled profile.
- **Accessibility:** narration-voice tests (prose, not state-list), aria-live cadence, full keyboard path (Tab→bird→Enter listen-in→Escape), reduced-motion rendering, WCAG AA contrast on all copy.
- **Voice split (I11):** lint that system surfaces pull only from `voice.system.*`.

---

## 12. Risks & mitigations

- **Drift miscalibration** (too fast → Tamagotchi-by-clicking; too slow → screensaver). The product lives in a narrow band the PRD makes us own. *Mitigation:* synthetic-agent calibration over compressed time; tunable `k_t`/`α` behind config; monotonic-up property tests; calibrate on synthetic/consented data only (I7).
- **Personality loss / sync corruption** — the worst, often-invisible failure (a silently slow-drifting or reset bird). *Mitigation:* server-sole-writer enforced at DB-role level (I1); additive deltas in event-log order, idempotent (I2); PITR/backups tuned to protect vectors; stable bird id across all sync/migration (I10); an anomaly-detection job that *flags* (never silently reconstructs) suspicious vector jumps.
- **Audio uncanniness / canned feel** — once a user hears the same call twice the spell breaks. *Mitigation:* procedural-only with real per-call variation (I9); recognizability tests; slow listen-in ramps; chorus via runtime mixing; **no recorded fallback ever**.
- **Accessibility regressions** flattening the affective core. *Mitigation:* a11y as a designed, launch-blocking surface (I12); narration tested as prose not state-list; reduced-motion as its own QA target; captions match the played call.
- **"Notice never announce" / gamification creep** — the most predictable contributor instinct. *Mitigation:* make the absence **structural** — no toast/badge primitive exists, the streak metric is **not computed** (I8); PR checklist; copy audit; notebook restricted to aviary-observations.
- **Presence laxity** silently inflating drift across the population. *Mitigation:* encode the 3-condition conjunction (I4); server cadence sanity-checks; activity window calibrated (lean long).
- **PII leakage** via email-as-identifier. *Mitigation:* synthetic UUID everywhere, email encrypted in one place + HMAC index, scrubbers, UUID partition keys (I6).
- **Performance breach** (first-bird >500ms, bundle >2MB, memory growth). *Mitigation:* budgets as CI gates; critical-path split; lazy chunks; reused audio buffers; virtualized notebook.
- **Visit feature scope-creep** toward a social network. *Mitigation:* render-only token (no event endpoints); no co-presence; no cross-account metrics computed; the architectural absence makes leaderboards/discovery *harder* to add later, by design.

---

## 13. Suggested milestone sequence

1. **Foundations:** account model (synthetic UUID, encrypted email + HMAC), magic-link auth, session management, DB roles (tick vs API) and the personality-write grant split. Establishes I1/I6 before any engine code.
2. **Canonical state + tick skeleton:** bird/aviary tables, append-only event log with per-account `seq`, tick worker reading the log in order, snapshot endpoint. Establishes I2.
3. **Engine:** personality vector + monotonic drift (I3), mood transitions + hysteresis + bird-to-bird coupling, presence accounting (I4), call parameters + greeting decisions.
4. **Client core:** Canvas2D renderer, two-stage hydration + edge-inlined snapshot (first-bird budget), interpolation, mood-shaped idle micro-motion, top bar with fade, no-load-state/quiet-field.
5. **Audio:** WebAudio procedural synth, scheduler, chorus, listen-in ramps, captions, silence fallback (I9).
6. **Interactions:** return-greeting (absence/boldness/mood-shaped, staggered), offers + cooldown, settle + 5s undo, notebook generation (sparse, aviary-only).
7. **Accessibility (parallel, not after):** narration (dual aria-live), reduced-motion designed rendering, keyboard nav, WCAG AA — launch-blocking (I12).
8. **Sync hardening:** multi-device snapshot consistency, idempotent replay, conflict surfaces in matter-of-fact voice.
9. **Visits:** invite/revoke/expire, read-only token, visit log, off-by-default notifications.
10. **Account lifecycle:** export, soft/hard deletion, email-change-with-verify, session revocation.
11. **Perf + observability + privacy boundary:** CI budget gates, synthetic fleet, aggregate-only RUM, telemetry-plane isolation (I7), calibration harness.
12. **Beta → GA** per §10.

Accessibility (7) and performance gates (11) are written here as their own milestones for clarity, but both are **continuous constraints** applied from milestone 4 onward, not end-stage add-ons.

---

## 14. Open calls flagged for the PRD owner
1. Email **HMAC lookup index** as the I6-compatant way to support magic-link sign-in (§4.1).
2. Account **export** including raw personality values to the **owner only** vs. omitting them to read I5 maximally (§4.7).
3. Final **`ACTIVITY_WINDOW`**, drift **`k_t`/`α`**, and **tick cadence** — calibration outputs, with the named ~1wk/~3wk drift targets as acceptance criteria.
4. Whether the **call scheduler** lives client-side (latency) with server-owned parameters, as proposed (§5.5).
5. Renderer choice: **Canvas2D** default vs. WebGL/PixiJS if the bundle budget allows (§6.1).

Each is a defensible default chosen to honor the invariants; flagged so the owner can confirm rather than discover.
