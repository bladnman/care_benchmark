# Pocket Aviary — Implementation Plan

This plan turns the PRD (`product_brief.md`, `concepts.md`, `bird_engine.md`, `interactions.md`, `aviary_layout.md`, `accounts_sync.md`, `social_optional.md`, `accessibility_perf.md`, `non_goals.md`) into an executable build plan for v1. It assumes a frontier engineering team with full context on the spec; it interprets ambiguity into defensible decisions rather than asking questions back.

---

## 1. Scope

### In v1

- Single-user accounts, magic-link sign-in, single canonical aviary per account.
- Two starter birds at signup, server-controlled growth to a cap of seven, drawn from a ~6-species pool.
- Personality vector (5 traits) per bird, server-owned, monotonic-toward-expressive drift, never exposed numerically.
- Mood system: small enumerated state set, daily-ish reset, modulated by interactions/time-of-day/ambient events/personality.
- Procedural call synthesis (WebAudio), bird-to-bird call interaction, chorus.
- Return-greeting, listen-in, offer (seed / song fragment / still pool), settle (with 5s undo), field notebook (sparse, naturalist, read-only).
- Presence accounting (visibility + focus + recent pointer/key activity, all three required).
- Server-side simulation tick (~1/min) that runs independent of client connection.
- Multi-device sync via canonical server state (no client-side merge logic).
- Visit-invitation social feature: email invite, read-only ambient visitor session, revocable, 30-day expiry, opt-in visit notifications, visit log.
- Screen-reader narration (naturalist prose, not state-list), reduced-motion mode (its own designed render path, not "animations off"), call captioning, WCAG AA contrast, full keyboard navigation.
- Performance budgets: ≤2MB gz initial JS, <500ms time-to-first-bird on mid-tier mobile/4G, 60fps idle motion on a 5-year-old laptop, no client memory growth over 30 minutes.
- Account export (JSON snapshot, emailed), account deletion (30-day soft, then hard), aggregate-only operational telemetry.

### Explicitly not in v1 (`non_goals.md`, `product_brief.md`)

Native apps; any gamification surface (streaks, achievements, badges, levels, scores, visit-frequency surfaces of any kind); Tamagotchi mechanics (death, hunger, decaying happiness, visible distress); social-network surfaces beyond the single visit affordance (profiles, follows, public feed, discovery, leaderboards); payments; shared/multi-profile aviaries; customizable scenes; multi-aviary accounts; push notifications of any kind (including no "friend visited" push — that path is a quiet visit-log entry plus an opt-in settings toggle only).

These are architectural constraints, not just UI omissions — see §4 (no client-writable personality state) and §13 (no aggregated per-account telemetry) for where the refusal is enforced below the UI layer, where a "harmless" feature could otherwise sneak back in.

---

## 2. Architecture

### 2.1 Service shape

Four backend services plus a CDN-served client. Kept deliberately small — this is not a microservice-per-noun system, it's a small number of services with hard ownership boundaries that map directly to the PRD's load-bearing rules.

1. **Auth service** — magic-link issuance/verification, session token issuance/revocation, account lifecycle (creation, soft/hard deletion, email change verification). Owns no bird/personality data.
2. **Aviary service** — owns accounts (synthetic UUID), birds, personality vectors, mood state, the append-only interaction-event log, and the simulation tick. This is the only writer of personality and mood. Exposes the state-snapshot read API and the event-write API.
3. **Notebook service** (or a module inside aviary-service — see §2.3) — generates and stores field-notebook entries from the same event/state stream the tick consumes. Read-only to clients.
4. **Visit service** — invitation lifecycle (issue, revoke, expire), visitor session tokens scoped to read-only snapshot access on the host's aviary. No write path to the aviary at all, by construction (the visitor's token is never accepted by the event-write API).

A separate **telemetry/observability pipeline** (synthetic checks, aggregate RUM, operational metrics) is architecturally isolated from the aviary service's database — see §13. It is fed by emitted operational events only, never by reads against the simulation database.

### 2.2 Client/server split

The client is a thin renderer and an event emitter. It:

- Pulls state snapshots (never computes personality, mood-transition logic, or drift — see §5).
- Synthesizes audio from a motif library shipped in the bundle, driven by snapshot data (vocal-frequency trait, current mood, call-grammar selection) — synthesis itself is a pure client-side rendering concern, not simulation.
- Renders idle motion, day/night palette, weather visuals, and listen-in mix changes locally, all derived from snapshot + local clock interpolation.
- Writes interaction events (offer, listen-in start/end, settle, presence pings) to the event-write API. Never writes personality or mood directly.

This split is the direct implementation of "the client never owns state" (`accounts_sync.md`). It also gives multi-device sync for free: two clients reading the same canonical record need no client-to-client protocol at all.

### 2.3 Render pipeline boundary

The render pipeline boundary sits exactly at the state snapshot. Everything below the line (personality, mood, drift, tick scheduling, notebook generation, presence-to-drift mapping) is server-owned and the client never sees its internals. Everything above the line (perch placement interpolation, idle micro-motion frame generation, audio buffer synthesis, weather particle rendering, screen-reader narration *string assembly* if done client-side, reduced-motion cross-fade sequencing) is client-owned and stateless across reloads — a fresh page load reconstructs the full visual state from one snapshot fetch with no client-side cache that could drift from the server.

Open implementation question, resolved here rather than left ambiguous: **narration prose (screen-reader and notebook) is generated server-side**, not client-side. Both consume the same internal state representation (the tick's bird/mood/event log), and generating them in the same service that owns that state avoids duplicating the "what changed and is it notebook-worthy" logic in two places with two voices that could drift apart. The client receives narration as a string field in the snapshot/poll response, not as raw state it interprets into prose itself. The notebook service and narration generator can be the same internal module behind two endpoints (entry log vs. live narration stream) since both are "naturalist prose from the same state," differing only in cadence and triggers — sparsity-tuned for the notebook, 30–60s cadence for narration.

### 2.4 Tech stack (defensible defaults, not mandated by the PRD)

- **Backend**: a typed server language (Go or Node/TypeScript — pick the team's existing stack; nothing in the PRD requires a specific one). Postgres for account/bird/personality state (strong consistency matters for the no-last-write-wins rule in §5). A lightweight append-only log table (or a narrow Kafka-less queue — volumes are low, ~1 event per user interaction, no need for a streaming platform at v1 scale) for the interaction-event log that the tick consumes.
- **Tick scheduler**: a cron-style worker process per-account-shard, or a single scheduler that sweeps all accounts needing a tick (accounts with new events since last tick, or accounts simply due for their ~60s cadence regardless of activity — ticks must run even with zero client connections, see §5.1). At v1 scale (single-user accounts, ~1/min cadence), a simple polling scheduler over an accounts-due index is sufficient; no need for per-account always-on processes.
- **Client**: a component framework with fine-grained reactive updates well-suited to continuous animation (React is fine if render-loop-sensitive code — the actual bird motion, audio scheduling — is kept outside the framework's render cycle and driven by `requestAnimationFrame`/WebAudio clocks directly). Canvas or WebGL for the aviary scene (DOM-based animation of 7 independently-animating birds plus ambient ornaments at 60fps is the kind of workload Canvas/WebGL handles more predictably than DOM transforms at scale).
- **CDN**: snapshot delivery from edge per `accessibility_perf.md`'s <500ms time-to-first-bird target — the initial snapshot should be inlined with the HTML response where the CDN/edge supports it, avoiding a second round-trip before first render.

---

## 3. Data model

All IDs are synthetic UUIDs (`accounts_sync.md` — email is never a key anywhere outside the account record itself).

### Account
```
account_id: uuid (pk)
email_encrypted: bytes
email_verified: bool
created_at, deletion_requested_at: timestamp | null
deletion_status: enum(active, soft_deleted, hard_deleted)
settings: { visit_notifications_enabled: bool, reduced_motion_opt_in: bool, captions_enabled: bool, audio_enabled: bool }
aviary_created_at: timestamp   // anchors the "aviary age" clock for new-bird unlocks
```

### Session (per-device)
```
session_id: uuid (pk)
account_id: uuid (fk)
device_label: string            // user-visible, for the revocation list
created_at, last_seen_at: timestamp
revoked_at: timestamp | null
```

### MagicLink
```
token_hash: string (pk)         // never store the raw token
account_id: uuid (fk)
issued_at, expires_at: timestamp // expires_at = issued_at + 15min
consumed_at: timestamp | null
```

### Bird
```
bird_id: uuid (pk)
account_id: uuid (fk)
species_id: string               // references the static species pool, not a row the simulation mutates
name: string
adopted_at: timestamp
personality: {
  boldness: float[0,1],
  social_warmth: float[0,1],
  vocal_frequency: float[0,1],
  plumage_saturation: float[0,1],
  curiosity: float[0,1]
}
mood: enum(wary, content, curious, drowsy, alert, settled)  // 'settled' added for night/settle-gesture state
mood_set_at: timestamp            // for daily-ish reset logic
perch_zone: enum(front, middle, back)   // server-computed signal, not user-set
last_offer_at: { seed: ts|null, song: ts|null, pool: ts|null }   // per-offer-type cooldown
```
Personality and mood are **never** present in any client-writable API payload — enforced at the API schema layer (the write endpoints physically have no field for them), not just by convention.

### InteractionEvent (append-only)
```
event_id: uuid (pk)
account_id: uuid (fk)
bird_id: uuid | null            // null for aviary-wide events (settle, presence ping)
event_type: enum(presence_ping, listen_in_start, listen_in_end, offer_seed, offer_song, offer_pool, settle, settle_undo)
occurred_at: timestamp
metadata: jsonb                  // e.g. listen-in duration on _end, offer target bird
client_session_id: uuid          // which device emitted it, for tick ordering & idempotency
sequence_no: bigint              // monotonic per-account, assigned at write time — gives the tick a strict event order regardless of which device/clock produced it
```
This table is the **only** way client behavior reaches personality/mood. It is read, never mutated, by the tick (§5).

### NotebookEntry
```
entry_id: uuid (pk)
account_id: uuid (fk)
written_at: timestamp
prose: string                    // final naturalist text, generated server-side
referenced_bird_ids: uuid[]      // for any future internal querying; never exposed as structured data to the client
```

### Invitation (Visit service)
```
invite_id: uuid (pk)
host_account_id: uuid (fk)
visitor_email_encrypted: bytes
token_hash: string
issued_at, expires_at: timestamp  // expires_at = issued_at + 30 days
revoked_at: timestamp | null
first_used_at: timestamp | null
```

### VisitSession
```
visit_session_id: uuid (pk)
invite_id: uuid (fk)
started_at: timestamp
last_seen_at: timestamp
duration_accum_seconds: int       // for the visit log's "approximate duration"
```

### VisitLogEntry (derived/materialized view over VisitSession, scoped to host)
```
host_account_id, visitor_email_encrypted (display only, decrypted at read time for the host), visited_at, approx_duration
```

Notably absent from the data model: anything resembling a streak counter, a visit-frequency aggregate, a "days active" field, or a personality-as-derived-from-event-log-at-read-time computation. Personality is a stored column updated by the tick, full stop (`bird_engine.md` — "never derived from session history at runtime").

---

## 4. API surface

Three logical surfaces: read (snapshot), write (events), and account/social management. All endpoints require a valid session token except magic-link request/verify.

### Read

- `GET /v1/aviary/snapshot` → current rendering snapshot: per-bird `{id, name, species, mood, perch_zone, position/animation_state, plumage_saturation_visual_tier}`, weather state, time-of-day-derived light state, narration string (most recent), settled/active aviary-level state. **Personality scalar values are never included** — only the visual/behavioral projections the client needs to render (e.g., a `plumage_saturation` value 0–1 is fine to send since it directly drives a visual, but boldness/warmth/curiosity/vocal-frequency raw scalars are not sent as named fields to the client; their effects are pre-baked into `mood`, `perch_zone`, and call-timing parameters the snapshot includes). This is the API-layer enforcement of "personality is never exposed numerically" — it's not a client-side hiding rule, the values simply aren't transmitted.
- `GET /v1/aviary/snapshot?since=<snapshot_version>` — supports the low-frequency keepalive and visibility-change refetch with a cheap diff/etag path.
- `GET /v1/notebook?cursor=` — paginated, reverse-chronological notebook entries, infinite scroll-back.
- `GET /v1/narration/stream` (SSE or long-poll) — pushes new narration strings at the 30–60s cadence for screen-reader consumption, with priority bump events (greeting, offer reaction, settle) pushed immediately out of band.

### Write (event log only)

- `POST /v1/events/presence-ping` — body: `{client_session_id, visible: bool, focused: bool, recent_input: bool}`. The client computes the three-condition conjunction locally (it has access to `visibilityState`, focus, and input events) and only pings when all three currently hold; server still timestamps and stores rather than trusting a client-asserted "presence=true" boolean blindly used for drift — see §5.2 for anti-gaming notes.
- `POST /v1/events/listen-in-start` — `{bird_id}`
- `POST /v1/events/listen-in-end` — `{bird_id}` (server computes duration from the matching start event; client doesn't self-report duration)
- `POST /v1/events/offer` — `{type: seed|song|pool, target_bird_id}`. Server validates the per-bird cooldown (§ bird_engine offer cooldown) and rejects with a no-op if still cooling down — the cooldown is enforced server-side, not just hidden client-side, since it's load-bearing for drift-saturation prevention.
- `POST /v1/events/settle` and `POST /v1/events/settle-undo` (undo only valid within 5s server-side window of the settle event; server, not client timer, is authoritative so a client clock skew can't extend the undo window).

No endpoint anywhere accepts a personality or mood value. This is enforced by the schema, not by server-side validation logic that could be bypassed by a future change — the request types for these endpoints simply have no such field defined.

### Account / management

- `POST /v1/auth/magic-link/request` — `{email}`, rate-limited per email.
- `POST /v1/auth/magic-link/verify` — `{token}` → session token; invalidates the link immediately.
- `GET/POST /v1/account/sessions` — list/revoke device sessions.
- `POST /v1/account/email-change/request`, `POST /v1/account/email-change/verify`.
- `POST /v1/account/export` → triggers async export job, emails download link.
- `POST /v1/account/delete`, `POST /v1/account/delete/undo`.
- `PATCH /v1/birds/:id/name`.
- `PATCH /v1/account/settings` — visit-notification toggle, reduced-motion opt-in, captions, audio.

### Visit-invitation flow

- `POST /v1/visits/invite` — `{visitor_email}` (host-authenticated) → emails one-time link.
- `POST /v1/visits/revoke` — `{invite_id}` (host-authenticated).
- `GET /v1/visits/log` (host-authenticated) → visit log entries + outstanding invites.
- `GET /v1/visit-session/:token` → (no host auth; token-authenticated visitor) issues a `VisitSession`, then the visitor client calls a **separate, read-only snapshot endpoint** `GET /v1/visit-session/:token/snapshot` that returns the same shape as `/v1/aviary/snapshot` for the host's aviary, scoped strictly to read. This endpoint is structurally incapable of accepting writes — it's a different route entirely from the event-write API, not a permission check on the same route, so there's no code path where a visitor token could be replayed against the write endpoints.
- The visitor snapshot endpoint returns `410 Gone` (rendered client-side as the matter-of-fact "visit no longer available" surface) once `revoked_at` is set or `expires_at` has passed.

The visitor session explicitly does not feed `InteractionEvent` — the visit-session snapshot read path has no associated event-write capability, which is what makes "visitor presence never drives host drift" (`social_optional.md`) true by construction rather than by a filter that could be forgotten.

---

## 5. Simulation engine design

### 5.1 The server-side tick

A scheduler sweeps accounts due for a tick (every account, every ~60s, regardless of connected clients — `accounts_sync.md`: "the tick runs whether or not any client is connected"). Per account, the tick:

1. Reads all `InteractionEvent` rows with `sequence_no` greater than the account's last-processed watermark.
2. Computes elapsed real-world time since the last tick (handles the case where an account has had zero ticks for days — e.g., infra downtime — by replaying the correct elapsed-time-based mood/time-of-day progression rather than assuming exactly 60s passed).
3. Updates **mood** per bird: applies time-of-day (computed from the account's last-known timezone offset, refreshed on each client snapshot request so DST/travel are handled), ambient weather rolls, bird-to-bird propagation (a wary mood in one bird raises wary-transition probability in nearby birds this tick), and the bird's own personality biasing transition probabilities (high-boldness birds resist wary transitions).
4. Computes **personality deltas**: aggregates the new events into per-trait deltas per the weighting in `bird_engine.md` (presence-time dominant; listen-in strong; offers small; settle non-directional). Deltas are added through a low-pass filter (e.g., exponential moving average toward a "target" implied by recent behavior, with a small time-constant tuned to the 1-week-instrument / 3-week-visible calibration target — see §5.3) and clamped so a delta is never negative regardless of input (monotonic-toward-expressive is enforced at the clamp, not just by the filter's natural behavior, so a future bug in the weighting math can't accidentally produce negative drift).
5. Advances `perch_zone` as a function of current mood + boldness (deterministic mapping, e.g., wary/drowsy bias back, content/curious/alert bias front, smoothed so a bird doesn't teleport perch-to-perch every tick — see §7 for client-side interpolation).
6. Updates call-timing parameters (next-call-eligible window, motif weighting) from vocal_frequency and mood.
7. Evaluates notebook-worthiness (§5.4) and writes a `NotebookEntry` if the tick's computed state change clears the sparsity bar.
8. Persists the new `Bird` rows and advances the account's processed watermark, all in one transaction per account (so a crash mid-tick can't partially apply a personality update — re-running from the watermark is idempotent).

### 5.2 Presence accounting and anti-gaming

Presence pings are client-asserted but timestamp-bounded server-side: a ping is only credited toward presence-time if it arrives within a tight server-side window of "now" (rejecting stale or replayed pings), and presence-time for drift purposes is computed as **the tick's reconciliation of ping density**, not a raw client-reported duration. A client can't claim "I had 3 hours of presence" in one event; presence accrues only as a rate of recent pings the server itself timestamps. This matters because presence is the dominant drift input (`concepts.md`) — the server must be the timekeeper, not the client, even though the client is the one detecting the visibility/focus/input conjunction.

### 5.3 Drift calibration target as a testable contract

The "measurable in instruments after ~1 week, visible to users after ~3 weeks" target (`bird_engine.md`) is encoded as an actual calibration parameter (the low-pass filter's time constant) with a CI-level test: a simulated event stream representing "typical regular use" (a defined synthetic profile — e.g., 20 min/day presence, occasional listen-in, occasional offers) run through the tick logic for a simulated week must produce a numerically detectable trait change above a defined epsilon, and for a simulated three weeks must produce a change above a second, larger, user-perceptible threshold (calibrated against the visual/behavioral tiers that `plumage_saturation` etc. map to — see §7). This test is the operational definition of "calibrated correctly" and should be the gate before the time-constant ships, not a value picked once and never verified against the stated target.

### 5.4 Notebook generation as part of the tick

The notebook generator runs as a tick-adjacent process (same transaction boundary or immediately following, reading the tick's output) that evaluates whether *this* tick's state change is notebook-worthy: a "first this week" event (e.g., bird A greeted before bird B for the first time in N days), a sustained-quiet streak worth narrating, a meaningfully large single-tick mood or perch shift, a notable weather-coincidence. The sparsity target (roughly one entry per few days for a regularly-visited aviary) is enforced by a per-account cooldown on entry generation plus a "noteworthiness score" threshold — most ticks produce no entry, by design. Templates are intentionally avoided in favor of a small generation system that composes prose from a constrained vocabulary keyed to the specific bird/event/time (e.g., a structured-but-varied template grammar, not literal `f"{bird} greeted before {other_bird}"` string formatting that would read identically every time — see voice notes in §11).

### 5.5 Call-grammar runtime

Lives client-side (per `accessibility_perf.md` — synthesized client-side via WebAudio), driven by snapshot-delivered parameters: each bird's `vocal_frequency`-derived call-eligibility timer, a small per-species motif library (shipped in the bundle, not fetched per-call), and runtime parameter randomization (pitch/timing jitter within the bird's signature envelope) so no two calls from the same bird are byte-identical while remaining recognizably "that bird's call." Chorus emerges client-side when two birds' independently-scheduled call timers land in the same window — this is **not** server-orchestrated; the server only provides the timing propensities (vocal_frequency, mood), and the actual moment-to-moment scheduling and overlap is a client rendering concern, consistent with calls being a rendering/audio concept layered on server-provided behavioral parameters rather than simulation state itself.

---

## 6. Sync model

Per `accounts_sync.md`, sync is an emergent property of the architecture, not a separate subsystem:

- **Single writer**: only the tick writes `Bird.personality` and `Bird.mood`. Clients write only to the append-only event log.
- **No last-write-wins**: because clients never submit absolute personality values, there is no write-write conflict on personality to resolve. Two devices both writing `InteractionEvent` rows concurrently is fine — both rows are kept, ordered by server-assigned `sequence_no`, and the next tick consumes both. This is explicitly safe even if the laptop's morning session and the phone's lunch session overlap in wall-clock time, because neither writes a "final" value — they each contribute events the tick later aggregates in event order.
- **Snapshot delivery**: clients pull a fresh snapshot on `visibilitychange` (tab foregrounded), on detecting a large `requestAnimationFrame` gap (laptop resume from suspend), and on a low-frequency keepalive (every ~30-60s while visible) — matching the tick cadence so a connected client never sees state more than one tick stale. Snapshot payloads are small (kilobytes) — no per-bird raw history, just current renderable state, per the perf budget.
- **No client-side personality cache used as source of truth.** A client may cache the last snapshot for instant re-render on reconnect (avoiding the "quiet field" loading state when the cache is fresh enough), but it always treats a freshly pulled snapshot as authoritative and replaces the cache wholesale — never merges.

---

## 7. Frontend rendering pipeline

### 7.1 Scene composition

Canvas/WebGL-rendered single horizontal scene, three perch zones (front/middle/back) mapped to fixed regions of the canvas that scale with viewport (responsive resize per `aviary_layout.md`, preserving "never crop a bird, never let one drift offscreen" — perch-zone anchor points are defined as proportions of viewport width/height, not fixed pixels). Layered render: background (sky/foliage, subtle parallax) → perch/bird middle layer → foreground ornament layer (occasional branch/leaf) → top-bar UI layer (DOM, not canvas, for accessibility/focus reasons — see §9).

### 7.2 Idle micro-motion

A small state machine per bird (preening, scanning, head-tilt, weight-shuffle) driven by the current `mood` from the snapshot, randomized timing within mood-appropriate envelopes (a drowsy bird's idle cycle is slow and low-amplitude; an alert bird's is faster and more head-movement-heavy). Perch-to-perch transitions (when `perch_zone` changes between snapshots) are smooth flight/hop animations interpolated over the gap between the old and new snapshot's perch zone, not a teleport — directly implementing "interpolates between snapshots for smooth motion" (`accounts_sync.md`).

### 7.3 Loading and first frame

No spinner, no fade-from-static, ever (`aviary_layout.md`). Implementation: the server-rendered (or edge-cached) initial HTML response includes the first snapshot inlined (avoiding a client-side fetch round-trip before first paint), and the renderer's first frame places each bird already mid-animation-cycle (a randomized phase offset into its idle state machine, not frame 0 of a cycle) so the very first paint looks like a continuation. If the snapshot fetch is slow (cold cache, slow connection), the interim state is a quiet soft-sky-color field with at most one or two faint ambient motion cues (e.g., a slow color gradient breathing) — explicitly not a spinner, not a progress bar, not a skeleton-UI pattern.

### 7.4 Empty-aviary and adoption-flow transition

Between account creation and the first bird's arrival, the canvas renders the same quiet field as the loading state. The first starter bird then flies in with a soft, slow entrance to its starting perch; the second follows similarly. After that, the empty state is never shown again for that account (enforced by checking `bird_count > 0` before ever rendering the empty-field branch).

### 7.5 Transitions: day/night, weather

Day/night palette is computed client-side from the user's local time (the client owns the local-time-to-palette mapping; the server doesn't need to know the user's wall-clock time for rendering purposes, only for mood transitions where it does need a timezone — see §5.1) and animates continuously (a slow palette interpolation function of local time-of-day, recomputed each frame or on a coarse timer, not stepped at hour boundaries). Weather events (rain, wind) are server-flagged in the snapshot (so all of a user's devices show the same weather moment, and weather's mood effects are tick-computed) but rendered client-side as a lightweight particle/shader effect.

### 7.6 Reduced-motion mode

A genuinely separate render path selected by `prefers-reduced-motion` or the accessibility setting, not a flag that disables animation calls in the default path. Concretely: the idle micro-motion state machine in reduced-motion mode renders a small number of discrete "pose" keyframes per mood (not a continuous procedural animation) and cross-fades between them on a slow timer; perch transitions are a cross-fade between the old-perch pose and new-perch pose rather than a flight path animation; ambient leaf/feather ornaments are switched off entirely; the day/night palette shift remains but is slowed further. This is built and QA'd as its own designed surface from day one (per `accessibility_perf.md`'s explicit instruction that it cannot ship as a v1.1 follow-up), with its own visual review pass distinct from the default mode's.

### 7.7 Listen-in mix-adjacent visual cue

While listen-in is purely an audio-mix interaction at spec level, the rendering pipeline should give the focused bird a very subtle visual emphasis (e.g., a soft focus-ring consistent with the keyboard focus indicator in §9, not a spotlight effect that would read as "switching channels") so the interaction reads coherently across audio and visual without becoming UI chrome inside the scene — note this stays consistent with "no UI chrome inside the aviary" (`aviary_layout.md`) by reusing the keyboard-focus treatment rather than inventing a new visual affordance.

---

## 8. Audio pipeline

### 8.1 Procedural call synthesis

Each species ships a compact **motif library** in the client bundle: a small set of parametrized waveform/envelope generators (not samples) — e.g., a handful of base motifs per species defined as oscillator + envelope + filter parameter sets, combinable and re-orderable at runtime. A bird's call at any moment is assembled by selecting motifs (weighted by personality/mood) and applying runtime jitter to pitch, timing, and amplitude within a per-species "signature envelope" that keeps the result recognizable as that species (and, via small per-bird-seeded parameter offsets, recognizable as that *specific* bird — e.g., a bird-specific pitch-center offset that persists for the bird's lifetime, layered under the mood/personality-driven variation) while never repeating identically.

### 8.2 Chorus mixing

When multiple birds' call timers land in overlapping windows, their independently-synthesized buffers are mixed live through the WebAudio graph (each bird has its own oscillator/gain chain feeding a shared mix bus) rather than pre-rendered and layered — this is what avoids the phase-cancellation artifact the PRD calls out for stacked recorded loops (`bird_engine.md`), since each call is generated fresh into the graph rather than two fixed waveforms being summed.

### 8.3 Listen-in mix decay

Listen-in engage/disengage drives a gain-automation ramp (WebAudio `GainNode.linearRampToValueAtTime` or an equivalent eased ramp, on the order of ~1-2 seconds) on the focused bird's gain node (up) and all other birds' gain nodes (down to a reduced-but-nonzero ambient floor — never to zero, per `interactions.md`'s "never go silent" rule). Disengage (re-click, click elsewhere, focus-away, focus a different bird) reverses the ramp symmetrically back to the ambient mix.

### 8.4 WebAudio fallback

Feature-detect `AudioContext` availability and permission state at startup. If unavailable or denied, the aviary renders and behaves identically except no audio graph is constructed; captions default to **on** in this fallback (per `accessibility_perf.md`) regardless of the user's stored caption preference, since silence with no captions would be a strictly worse experience than the user's stored preference anticipated. No recorded-audio fallback path exists anywhere in the codebase — this is enforced by simply not building one, not by a runtime check that could be bypassed.

### 8.5 Buffer/resource bounds

Audio buffers and oscillator nodes are pooled and reused rather than allocated per call (supports the "no per-call allocation that isn't freed" / no-memory-growth requirement in §10). A bounded number of concurrent voice chains (sized to the 7-bird cap plus a small headroom for overlapping chorus) are pre-allocated at audio-context init.

---

## 9. Accessibility surfaces

### 9.1 Screen-reader narration

Server-generated naturalist prose (§2.3), delivered via a live-region-friendly mechanism: an `aria-live="polite"` region updated on the narration cadence (30-60s idle, immediate for prioritized events — greeting, offer reaction, settle). The narration text is the *same* generation system that produces notebook entries (shared voice, shared underlying state), differing in cadence/scope, so a screen-reader user and a sighted user are experiencing the same product's voice rather than a parallel accessibility-only feature. Implementation must explicitly avoid the "ARIA-label automation" trap called out in the PRD — there should be no code path that walks visual DOM/canvas state and auto-generates labels like "Bird 2: mood content"; the narration is authored prose output from the same generator, full stop.

### 9.2 Reduced-motion

Covered in §7.6 — its own render path, shipped with v1, not deferred.

### 9.3 Captions

Generated at runtime from the actual call-grammar parameters used for that specific call instance (§8.1) — e.g., a caption generator that inspects the motif/pitch/timing selections just made for this call and produces a short naturalist description ("a soft three-note rise") matching what was actually synthesized, not a fixed string keyed to call "type." Rendered as small fading text near the calling bird (DOM overlay positioned relative to the canvas bird position, or a canvas-rendered text layer — DOM overlay is simpler for contrast/AA compliance and font handling).

### 9.4 Keyboard navigation

Top bar (DOM elements) is in normal tab order. Entering the canvas scene via Tab moves focus to a logical (non-visual, but visually indicated via a focus ring) "first bird" element; arrow keys move focus among birds (an off-screen/ARIA-described focus model layered over the canvas, e.g., a hidden but tab-reachable element per bird synced to canvas position for the focus ring's visual placement). Enter triggers listen-in on the focused bird; Escape exits listen-in. Offer affordance opens via top-bar shortcut and is itself keyboard-navigable (a small DOM popover/menu, not a canvas-drawn picker, for straightforward focus trapping and AA contrast). Settle is a top-bar button, naturally keyboard-reachable.

### 9.5 Contrast

All DOM-rendered user copy (top bar labels, settings, account/error surfaces, captions, any visually-displayed narration) is checked against WCAG AA in the design system and verified in CI via an automated contrast-checking pass over the rendered chrome (not the aviary scene itself, which carries no required-contrast user copy per `accessibility_perf.md`).

---

## 10. Performance budgets and observability

| Budget | Target | Primary levers |
|---|---|---|
| Initial JS bundle | ≤2MB gzipped | Procedural audio (no recorded files), code-splitting account/accessibility-settings/visit-flow behind route-level chunks, procedurally-generated or compact-SVG bird visuals |
| Time to first bird visible | <500ms, mid-tier mobile, 4G | Edge-delivered inlined initial snapshot, critical-path render with no non-critical asset blocking, bundle budget above |
| Idle motion frame rate | 60fps sustained, 5-year-old laptop, 30-min session | Canvas/WebGL render path, bounded idle-state-machine complexity, capped concurrent ambient ornament count |
| Memory growth | None detectable over 30 min | Audio buffer/node pooling (§8.5), notebook entries scrolled out of view release references (virtualized list), bounded worker/audio-context lifecycle |
| Simulation tick latency | p99 < 5s (alarm threshold) | Per-account tick transaction kept small (bounded event-log read window since last watermark), indexed accounts-due query |

**Observability**: synthetic browser checks on a schedule from multiple geographies (page load, first-bird-render timing); aggregate RUM (render-frame timing histograms, audio-context error counts, simulation-tick latency percentiles, anonymized session-duration histograms with no per-account dimension). CI gates: a memory-growth regression test (automated 30-min simulated session, heap snapshot diffing) and the drift-calibration test from §5.3 are both release-blocking, not advisory.

**Browser support**: last two major versions of Chrome/Safari/Firefox/Edge; older browsers get a matter-of-fact unsupported-browser page, no compatibility shims maintained.

---

## 11. Rollout

### 11.1 Launch shape

V1 ships complete per the scope in §1 — accessibility and core engine are not phased apart (`accessibility_perf.md` is explicit that reduced-motion/narration/captions cannot land as a post-launch fix). The launch sequence:

1. **Internal dogfood** (small cohort, team + early testers): validate the drift-calibration test against real usage patterns over a real 1-3 week window before opening signups — this is the one piece of the system that can't be fully validated by synthetic tests alone, since "feels alive over weeks" is ultimately a felt judgment, not just a numeric one.
2. **Soft launch with capped signups**: validate the simulation tick's scheduler holds up at the target latency budget (§10) with a real account population running ticks every ~60s regardless of activity, and validate multi-device sync behavior with real overlapping-session event logs (not just the synthetic conflict scenario from `accounts_sync.md`).
3. **Open signup**: once tick latency, time-to-first-bird, and the drift-calibration target are all holding in production telemetry.

### 11.2 Birds-per-aviary ramp

New-species offers are gated on aviary age (`bird_engine.md` — age, not interaction count or tier), computed from `Account.aviary_created_at`. The exact age thresholds (e.g., "a few months" for bird three, up to "five or six" by a year) are a tunable config table rather than hardcoded constants, since they're explicitly named in the PRD as a pacing choice the team may want to adjust post-launch without a code change — but the *mechanism* (age-gated, never attention/visit-count-gated) is fixed at the architecture level so a future "just make it activity-based" request can't be satisfied without contradicting this plan's data model (there is no stored "interaction count" or "visit count" aggregate to gate on — see §13, this is also a privacy/anti-gamification choice, not just a product one).

### 11.3 Day-one instrumentation

From day one: aggregate RUM and synthetic perf checks (§10), simulation-tick latency percentiles, audio-context error rates, the drift-calibration CI test results as an ongoing canary (re-run periodically against a fresh synthetic profile, not just at the time the time-constant was first chosen), and account-lifecycle funnel metrics (signup → first bird seen → first return session) computed in aggregate only, with no per-account interaction-pattern dimension reaching any analytics warehouse (§13).

---

## 12. Risks

**Drift calibration is a single point of product failure.** If the low-pass filter's time constant is wrong in either direction, the product either becomes a Tamagotchi-by-accident (drift visible session-to-session, teaching users they can "earn" a bolder bird by clicking) or a screensaver (no perceptible change ever, undermining the entire "feels alive over weeks" claim). Mitigation: the calibration test in §5.3 as a release gate, plus a real dogfood window (§11.1) before any external users see the system, plus treating the time constant as a server-side config value that can be tuned without a client release if early production data shows the target is missed.

**Sync correctness under concurrent multi-device sessions.** The no-last-write-wins design (§6) is sound on paper, but the actual tick's event-aggregation-in-order logic is exactly the kind of code that can silently misbehave under real concurrent writes (clock skew between devices affecting `occurred_at` even though `sequence_no` is server-assigned, a tick crash mid-transaction leaving a partial watermark advance, etc.). Mitigation: `sequence_no` (not client timestamp) is the only ordering authority the tick trusts; per-account tick transactions are atomic (watermark advance and personality write commit together or not at all); a dedicated integration test suite simulates overlapping-device event streams against the tick and asserts no event is double-counted or dropped across a crash-and-retry.

**Audio uncanniness.** Procedural synthesis that doesn't clear a believability bar is worse than the looped-audio problem it's meant to solve — a chorus that sounds obviously synthetic in a *new* way (e.g., audibly "randomized" rather than "varied," motifs that don't actually sound bird-like) breaks the spell just as hard as a detected loop. Mitigation: this is fundamentally a sound-design problem requiring iteration with a real audio/sound designer against the actual motif-library implementation, not something resolvable by engineering alone — budget explicit design-review cycles on the call-grammar output before launch, not just a functional "does it play" check.

**Accessibility regressions creeping in post-launch.** Because reduced-motion and narration are full designed surfaces (not flags), every future feature addition (a new offer type, a new species, a new top-bar affordance) has two render paths and a narration/caption obligation, not one. Mitigation: a checklist gate in the PR/release process requiring any new bird-state-affecting feature to ship its reduced-motion rendering and narration/caption text in the same change, mirroring the "ships with the rest of the product, not after" rule from the PRD itself, enforced procedurally rather than left to reviewer memory.

**Gamification creep via "harmless" additions.** The PRD names this risk explicitly and at length (`non_goals.md`) because it is a near-certain future pressure (a stakeholder asking for "just a small streak indicator," a growth-focused contributor proposing a visit-frequency surface). Mitigation: the data model itself doesn't store the aggregates that would make such a feature cheap to build (no visit-count field, no streak field, no per-account interaction-frequency rollup anywhere) — see §11.2 and §13. This is a structural defense, not just a documented rule: building the feature later would require *adding* a new aggregate pipeline, which is a visible, reviewable architectural change rather than a quiet UI addition over existing data.

**Visitor-isolation bugs in the social feature.** A bug that lets a visitor session's snapshot reads feed back into host presence/drift (even accidentally, e.g., via a shared snapshot-fetch code path that also logs a presence ping) would silently violate the "visitor attention never drives host drift" guarantee with no visible symptom to either party. Mitigation: the visitor snapshot endpoint is a structurally separate route (§4) with no code path that touches the `InteractionEvent` writer at all — not a shared handler with a role check — making the isolation a property of the routing table, verified by an integration test that asserts no `InteractionEvent` rows are ever created by requests authenticated with a visit-session token.

---

## 13. Privacy and telemetry boundary (cross-cutting, enforced at the architecture layer)

Restated here because it constrains §3, §4, §11.2, and §12 simultaneously: per-bird interaction events and personality/mood state exist only to drive that account's own simulation. The aviary/simulation database is never read by the analytics/telemetry warehouse; telemetry pipelines emit only operational metrics (request counts, latencies, error rates, anonymized session-duration histograms with no per-account dimension) from a separate emission path that never touches per-bird fields. This is the same architectural boundary that makes the no-streak-counter, no-visit-frequency-surface rule durable (§11.2, §12) — there is no aggregate pipeline that *could* be repurposed into a gamification feature, because none exists in the first place.
