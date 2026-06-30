# Pocket Aviary — V1 Implementation Plan

This plan turns the PRD into an executable build for a frontier engineering team. It assumes the design philosophy in `product_brief.md` is non-negotiable: every architectural choice below is justified against "feels alive, not robotic," "notice, never announce," and the no-Tamagotchi / no-gamification non-goals. Where the PRD leaves an implementation detail open, this plan makes a defensible call and flags it as `[CALL]`.

---

## 1. Scope

### In v1
- Single-user accounts, magic-link auth, one canonical aviary per account.
- Two starter birds at adoption, server-driven offers of new species as the aviary ages, cap of seven.
- Server-side simulation tick driving personality drift (slow timescale) and mood (fast timescale).
- Procedural, client-synthesized calls (WebAudio), bird-to-bird call interaction, chorus mixing.
- Client rendering: single horizontal scene, three perch zones, day/night cycle anchored to local time, ambient weather, idle micro-motion, reduced-motion mode.
- Interactions: return-greeting, listen-in, offer (seed / song fragment / still pool), settle (with 5s undo).
- Field notebook: sparse, auto-generated, naturalist-voice, read-only.
- Presence accounting (visibility + focus + recent pointer/key activity, conjunctively).
- Multi-device sync via canonical server state (no client-side merge).
- Visit-invitation: per-invite opt-in, read-only ambient visits, revocable, visit log, off-by-default friend-visited notification toggle.
- Accessibility: screen-reader narration (naturalist prose, not state-list), reduced-motion as its own designed render mode, call captions generated from the call grammar at runtime, WCAG AA contrast on chrome, full keyboard navigation.
- Performance: <2MB gzipped initial bundle, <500ms time-to-first-bird on mid-tier mobile/4G, 60fps idle motion on a 5-year-old laptop, no client memory growth over 30 minutes.
- Account export (JSON snapshot, emailed link) and soft-then-hard account deletion (30-day window).

### Out of v1 (respecting `non_goals.md`)
- No native apps; no app-shell abstraction layer that anticipates one. The web client is built as a web client, not as a cross-platform-ready core with a thin web shell — that abstraction would cost render-pipeline quality we can't spend.
- No gamification surfaces of any kind: no achievement tables, no streak-counter service, no "visits this week" aggregation — not even an internal admin-only one, because an internal one has a way of becoming a user-facing one under product pressure six months from now.
- No Tamagotchi mechanics: no hunger/decay timers, no distress states, no death/reset path for a bird under any operational condition (including data-recovery tooling — see §11 on recovery semantics).
- No social-network surfaces beyond the single visit affordance: no profiles, follows, feeds, discovery, comments, leaderboards, or the underlying aggregate metrics that would make a leaderboard cheap to bolt on later. We do not compute cross-account ranking statistics anywhere, including internally.
- No shared/multi-aviary accounts, no payments.

---

## 2. Architecture

### 2.1 Service shape

Four backend services plus a CDN-fronted static client, deliberately small:

1. **Auth service** — magic-link issuance/consumption, session token issuance/revocation, account lifecycle (create, soft-delete, recover, hard-delete), email-change verification. Owns the only table that stores raw email.
2. **Simulation service** — owns the canonical aviary record (birds, personality vectors, mood state, perch/position state, weather/time-of-day derived state) and runs the tick. The only writer of personality vectors and mood. Exposes a read API (state snapshot) and an event-ingest API (interaction events). This is the load-bearing service; it is built and tested before the others.
3. **Social service** — visit invitations, visit sessions, visit log, revocation. Reads aviary snapshots from the simulation service through the same read API a host's own client would use (a visitor is just a scoped, read-only client), so there is structurally no path for a visitor to gain write access — the visitor session token issued by this service carries a `read_only=true, account_id=<host>` claim that the simulation service's read API enforces.
4. **Notebook service** — consumes the simulation event stream (tick outputs + significant interaction events) and generates sparse naturalist-voice entries. Decoupled from the simulation service so notebook-generation logic (an LLM-assisted or template-driven prose generator, see §4.5) can iterate independently of the tick's correctness-critical code.

A thin **edge/BFF layer** sits in front of all four for the client: issues the small state-snapshot payload from a CDN edge alongside the HTML shell (this is what makes the <500ms time-to-first-bird budget reachable — see §10), aggregates the top-bar data (notebook unread? no — there is no unread state, see `interactions.md` "no streak counter" reasoning extended to notebook badges in §4.5), and proxies interaction-event writes.

`[CALL]` Telemetry/observability is a fifth, fully isolated pipeline (see §9) that never reads from the simulation database directly — it consumes only the aggregate-emission path described in §11, enforced by network policy (the analytics warehouse has no credential that can reach the simulation DB).

### 2.2 Client/server split

- **Server owns**: personality vectors, mood state and transitions, the event log, drift computation, presence-time aggregation, notebook entry generation, visit/invite state, account state.
- **Client owns**: rendering (scene composition, idle micro-motion interpolation between snapshots, day/night palette computation from local time, ambient leaf/feather ornament generation — explicitly *not* simulation-tracked per `aviary_layout.md`), procedural audio synthesis, presence *detection* (the three-signal check) and presence-event submission, reduced-motion rendering mode, narration-to-speech-or-visual-text rendering, local interaction-event buffering before submission.

The line is exact: anything that determines what a bird's personality or mood *is* lives server-side; anything that determines how that state is *rendered* lives client-side. This is what makes "clients never tick; they pull snapshots and interpolate" (`concepts.md`) actually true in code, not just in the data model.

### 2.3 Render pipeline boundary

The client maintains two distinct state layers:
1. **Authoritative layer** — the last-pulled snapshot from the simulation service (bird positions/perch assignments, moods, active animation/transition hints, weather state, server timestamp).
2. **Interpolation/render layer** — a continuously-advancing local clock that tweens between authoritative snapshots (position A → position B over the snapshot interval) and layers in client-only ambient ornaments (leaf drift, feather fall) that have no server representation.

This split is what lets the first frame render "already in motion": on cold load, the client receives a snapshot whose `last_tick_at` may be up to ~60s old (tick cadence), computes how much *render*-layer motion should have elapsed since then (idle-motion phase offset, not personality state), and starts the interpolation layer already mid-cycle rather than at a neutral pose. See §7.1 for the exact load sequence.

---

## 3. Data model

All IDs are server-generated synthetic UUIDs. No email-derived identifiers anywhere outside the auth service's account table (per `accounts_sync.md`'s "single most important boring detail").

### 3.1 Account
```
Account {
  id: uuid (pk, synthetic)
  email_encrypted: bytes          // auth service only; never replicated to other services
  status: enum(active, pending_deletion, deleted)
  pending_deletion_at: timestamp | null
  hard_delete_at: timestamp | null   // pending_deletion_at + 30 days
  created_at: timestamp
  settings: {
    reduced_motion_opt_in: bool,        // independent of prefers-reduced-motion; explicit user override
    captions_enabled: bool,
    visit_notifications_enabled: bool,  // default false, per social_optional.md
    audio_enabled: bool
  }
}

Session {
  token: uuid (pk)
  account_id: uuid (fk)
  device_label: string             // user-visible, e.g. "Safari on iPhone" derived from UA at issuance
  created_at: timestamp
  last_seen_at: timestamp
  revoked_at: timestamp | null
}
```

### 3.2 Aviary and birds
```
Aviary {
  account_id: uuid (pk, 1:1 with Account)
  created_at: timestamp             // "aviary age" anchor for the third-bird-and-beyond pacing
  last_tick_at: timestamp
  weather_state: { kind: enum(clear, rain, wind), started_at, expires_at } | null
}

Bird {
  id: uuid (pk)                     // stable identity, never reassigned, never reused — see concepts.md "Bird identity"
  aviary_account_id: uuid (fk)
  species_id: string (fk -> species pool, static reference data)
  name: string                      // user-assigned, renameable any time, no effect on personality/mood/call
  adopted_at: timestamp
  personality: {
    boldness: float [0,1],
    social_warmth: float [0,1],
    vocal_frequency: float [0,1],
    plumage_saturation: float [0,1],
    curiosity: float [0,1]
  }                                  // server-authored only; see §6.2 for the additive-delta write path
  mood: {
    state: enum(wary, content, curious, drowsy, alert),
    entered_at: timestamp,
    daily_reset_at: timestamp        // next daily-ish reset boundary
  }
  perch_zone: enum(front, middle, back)
  current_animation_hint: string     // e.g. "preening", "calling", "scanning" — server-computed at tick time, consumed by client renderer
  last_offer_accepted_at: timestamp | null   // cooldown tracking, per bird
}
```
`[CALL]` Personality and mood live as JSON-typed columns on the `Bird` row rather than separate tables, since they are always read/written together at tick time and never queried independently across birds. Species pool (~6 species) is static reference data shipped with the simulation service, not a DB table that changes at runtime.

### 3.3 Event log (append-only, client-submitted)
```
InteractionEvent {
  id: uuid (pk)
  account_id: uuid (fk)
  bird_id: uuid | null              // null for account-level events (settle, presence pings not yet attributed)
  type: enum(presence_ping, listen_in_start, listen_in_end, offer_seed, offer_song, offer_pool, settle, settle_undo)
  client_event_at: timestamp        // client-reported, used for ordering within a session
  received_at: timestamp            // server receipt time, authoritative for tick ordering
  payload: jsonb                    // e.g. listen-in duration on listen_in_end, offer target bird on offer_*
  device_session_token: uuid (fk)
}
```
This table is append-only and is the *only* path by which client behavior reaches personality state — enforced at the API layer (§4) and re-stated as a hard rule in §6.2.

### 3.4 Notebook
```
NotebookEntry {
  id: uuid (pk)
  account_id: uuid (fk)
  written_at: timestamp
  prose: string                      // naturalist voice, lowercase, present-tense
  // no "kind"/"category" enum exposed to client — entries are prose, not typed events,
  // matching interactions.md's "not generic event logs" requirement
}
```
Read-only to all clients including the owning account's. No edit/delete/annotate path exists anywhere in the API surface (not just hidden in the UI).

### 3.5 Visits
```
VisitInvite {
  id: uuid (pk)
  host_account_id: uuid (fk)
  visitor_email_encrypted: bytes
  token: uuid                        // the one-time link token
  status: enum(pending, active, revoked, expired)
  created_at: timestamp
  expires_at: timestamp              // created_at + 30 days
  first_used_at: timestamp | null
  revoked_at: timestamp | null
}

VisitSession {
  id: uuid (pk)
  invite_id: uuid (fk)
  started_at: timestamp
  last_seen_at: timestamp
  ended_at: timestamp | null
}
```
`VisitLog` is a read projection over `VisitInvite`/`VisitSession`, not a separate write path.

### 3.6 What is deliberately absent from the data model
- No streak/visit-frequency table, no "consecutive days" counter, no aggregate-per-account engagement score — per `non_goals.md`, these are refused at the schema level so they cannot be "just exposed" later by a UI-only change.
- No "happiness"/decay/hunger field on `Bird`.
- No personality-history table exposed to any client API (an internal time-series may exist purely for the instruments described in §6.4's calibration testing, but it is not part of the export schema or any user-facing read path).

---

## 4. API surface

All endpoints sit behind session-token auth (except magic-link issuance/consumption). Naturalist-vs-matter-of-fact voice split (per `product_brief.md`) is enforced at the API error-message layer too: error payloads carry a `message` field already in matter-of-fact voice, generated server-side, so clients never construct error copy themselves and can't accidentally drift into naturalist phrasing for a system surface.

### 4.1 Auth
- `POST /auth/magic-link` `{email}` → 202 (always 202 regardless of whether the email exists, to avoid account enumeration)
- `POST /auth/magic-link/consume` `{token}` → `{session_token}` or matter-of-fact error (expired/used)
- `POST /auth/sessions/:id/revoke` (authenticated)
- `GET /auth/sessions` (authenticated) → list for the device-revocation UI
- `POST /auth/email-change` `{new_email}` → triggers verification email to new address; old email remains active until verified
- `POST /account/delete` → soft-delete, sets `pending_deletion_at`/`hard_delete_at`
- `POST /account/restore` → "I changed my mind," available any time before `hard_delete_at`
- `POST /account/export` → enqueues export job, emails download link on completion

### 4.2 Aviary state (read)
- `GET /aviary/snapshot` → the core read endpoint. Returns:
  ```
  {
    server_time: timestamp,
    last_tick_at: timestamp,
    weather: {...} | null,
    birds: [{id, name, species_id, perch_zone, mood_state, animation_hint, personality_render_hints}],
    notebook_unread: false   // literally always false — there is no unread state; field omitted entirely, not just hardcoded (see 4.5)
  }
  ```
  `personality_render_hints` is the *only* personality-derived data ever sent to a client: a small set of derived, non-numeric rendering parameters (e.g. plumage color stops, idle-motion pacing tier) computed server-side from the vector. The raw vector never serializes to any client payload, satisfying "personality vector is never exposed numerically" as a wire-format guarantee, not just a UI omission.
  
  Polled via: visibility-change pull, long-render-frame-gap pull (suspend/resume detection), and a low-frequency keepalive (`[CALL]` every 20s while visible) — per `accounts_sync.md` §"How clients consume state."

- `GET /aviary/snapshot` also serves the **visitor** read path, scoped by a visit session token instead of an account session token; same response shape, with interaction-submission endpoints (§4.3) returning 403 for visitor tokens.

### 4.3 Interaction events (write)
- `POST /events` `{type, bird_id?, client_event_at, payload}` → 202, appends to `InteractionEvent` log. This is the *only* client write path that can influence personality/mood. No endpoint anywhere accepts a personality or mood value from a client.
  - `presence_ping`: submitted by the client's presence detector (§5) at a steady interval while all three presence conditions hold; payload is empty, server derives presence-time from inter-ping gaps.
  - `listen_in_start` / `listen_in_end`: payload includes `bird_id`; duration computed server-side from the start/end pair (or from `received_at` gap if `listen_in_end` is missing due to tab close — see §6.2 ordering note).
  - `offer_seed` / `offer_song` / `offer_pool`: payload includes target `bird_id`; server enforces the per-bird cooldown (rejects with a quiet no-op, not an error — an offer during cooldown is absorbed silently rather than surfaced as a failure, since "you can't offer yet" would be a system-voice intrusion into a naturalist interaction).
  - `settle` / `settle_undo`: account-level, `bird_id` null.

### 4.4 Offers reference data
- `GET /offers/catalog` → the small, static song-fragment library and seed/pool metadata (icons, labels) for the top-bar offer affordance. Static, cacheable, ships in the initial bundle rather than fetched if it stays small (`[CALL]`: bundle this at build time, it's well under the 2MB budget and avoids a render-blocking fetch).

### 4.5 Notebook
- `GET /notebook/entries?before=<entry_id>&limit=N` → cursor-paginated, newest first, for infinite scroll-back. No "unread count," no "new entries since last visit" marker — per `interactions.md`'s no-streak-counter reasoning extended here: an unread badge is structurally the same "system tracking your visit cadence and surfacing it back at you" pattern the PRD names as the most-reachable forbidden feature, even though it's nominally about notebook content rather than visit frequency. `[CALL]` This is an interpretation beyond the letter of `non_goals.md` (which is specifically about visit-frequency surfaces) but consistent with its stated spirit ("the rotation is small... but its cumulative effect... is total"); flagged here so a reviewer can override if they read the scope more narrowly.

### 4.6 Visits (social)
- `POST /visits/invite` `{visitor_email}` → host-only, creates `VisitInvite`, sends one-time-link email
- `POST /visits/:invite_id/revoke` → host-only, immediate
- `GET /visits/log` → host-only, the visit log read
- `POST /visits/consume` `{token}` → visitor-facing, exchanges one-time token for a `read_only` session token (single use; subsequent hits to the same link after first consumption re-validate against the still-active `VisitSession` rather than re-issuing, so a visitor can refresh the page during a visit without the link "using itself up" twice)
- `GET /visits/snapshot` → equivalent to `/aviary/snapshot` but visitor-scoped (or this can simply *be* `/aviary/snapshot` with token-type dispatch server-side — `[CALL]`: implement as the same endpoint, dispatching on token claims, to guarantee visitor and host literally cannot see different data, which is a stronger guarantee than two endpoints kept consistent by convention)

---

## 5. Presence accounting (client implementation)

Implements `concepts.md`'s precise three-signal conjunction. Client-side detector:

```
presenceActive = document.visibilityState === 'visible'
              && document.hasFocus()
              && (now - lastPointerOrKeyEventAt) < ACTIVITY_WINDOW   // [CALL] start at 3 minutes, tunable server-side via config
```

- Listeners: `visibilitychange`, `focus`/`blur`, `pointermove` (throttled to ~1/sec to avoid event-volume issues), `keydown`.
- While `presenceActive` is true, the client submits a `presence_ping` event on a steady interval (`[CALL]` 30s) rather than continuously, to bound event-log write volume; the server derives presence-time as the sum of inter-ping gaps capped at the ping interval (so a missed ping due to network blip doesn't inflate presence-time beyond what was actually observed).
- The activity window is a *server-distributed config value*, not a client constant, so it can be recalibrated without a client release (per `interactions.md`: "we will calibrate the exact window during build").
- Presence detection runs identically regardless of which page/route is active within the app (account settings, notebook) as long as the tab itself is visible+focused+active — `[CALL]`: this is an interpretation call; the PRD doesn't explicitly say whether presence accrues while the user is in account settings vs. the aviary scene. Given "idle attention is itself an interaction" and presence is about *the tab*, not *the scene*, this plan treats any in-app surface as presence-eligible, since gating it to the aviary-scene route specifically would require the user to keep a specific route open to "count," which reads like a disguised engagement mechanic.

---

## 6. Simulation engine design

### 6.1 Tick loop

A scheduled job per account (`[CALL]`: implemented as a queue-driven worker pool, one job per account per tick window, rather than a single global loop over all accounts, so tick latency doesn't degrade as the account base grows — this is what the p99 5s alarm in `accessibility_perf.md` is actually measuring per-account). Cadence ~60s per account, jittered slightly per-account to avoid thundering-herd load.

Each tick, per account:
1. Read all `InteractionEvent` rows since `last_tick_at`, ordered by `received_at` (server receipt order, not client-reported order — client clocks aren't trusted for ordering, only for intra-session UX).
2. Compute presence-time accrued in the window (sum of ping gaps, per §5).
3. Compute mood transitions per bird (§6.3).
4. Compute personality deltas per bird (§6.2) and apply additively.
5. Recompute perch-zone assignment per bird from updated mood+personality (boldness pulls toward front, wary mood pulls toward back).
6. Recompute/expire weather state (roll a small chance of starting rain/wind per tick if none active; `[CALL]` target frequency "a few times a week" → roughly a 1-2% chance per tick of starting a weather event when none is active, tuned during build to hit the target frequency empirically).
7. Advance `current_animation_hint` per bird (a lightweight state machine: idle → occasional preen/scan/call hint, mood-weighted).
8. Write updated `Bird` and `Aviary` rows, advance `last_tick_at`.
9. Emit a tick-summary event to the notebook service's stream (not raw event-log access — the notebook service never reads `InteractionEvent` directly, only the simulation service's emitted summaries, keeping the notebook generator decoupled from event-log schema changes).

### 6.2 Personality drift (additive, server-only)

Per bird, per tick, compute a small delta vector from the window's signals, in the PRD's stated weight order:
- `presence-time` (dominant): scaled contribution to all traits, weighted toward `plumage_saturation` and general "expressiveness."
- `listen-in` duration on this bird: contributes to `social_warmth` and `vocal_frequency`.
- `offer accepted` near this bird: small `curiosity` delta; any offer near the bird (accepted or not) gives a smaller `boldness` delta.
- `settle`: no directional drift; only closes the presence window for this tick's accounting.

**Monotonicity enforcement**: deltas are clamped to `>= 0` before being applied — i.e., the delta computation can produce zero (no presence/interaction signal this window → no movement) but never negative. This is enforced in code, not just by the delta formula's natural sign, so a future contributor adding a new signal can't accidentally introduce a negative-drift path without explicitly bypassing a guard. `[CALL]`: implement as `bird.personality[trait] = min(1.0, bird.personality[trait] + max(0, computed_delta))` with the `max(0, ...)` as a named, commented invariant, not an incidental clamp.

**Write path**: only the tick job writes `Bird.personality`. The event-ingest API (`POST /events`) has no code path that touches the `personality` column — enforced by giving the ingest API's DB role no `UPDATE` grant on that column at the database level, not just by application-layer convention. This makes "no last-write-wins for personality state" a database-enforced guarantee, directly implementing `accounts_sync.md`'s additive-server-authored-deltas rule.

**Concurrency**: per-account tick jobs are serialized per account (a per-account advisory lock or queue partition key) so two tick runs for the same account never race, even if a worker retries after a timeout. This is what prevents the exact "lunch write wins, morning's drift silently deleted" failure the PRD calls out, applied to the tick itself rather than to a hypothetical client write.

### 6.3 Mood

Mood is a small state machine per bird (`wary, content, curious, drowsy, alert`), transitioned each tick based on:
- Recent interaction signals this window (offer accepted → bias toward `content`; absence of any signal for several ticks → gentle drift toward `drowsy`/ambient, not toward `wary` — neglect must not produce wary, per the drift-asymmetry rule extended to mood).
- Time-of-day (computed from the account's last-known client timezone offset, stored at snapshot-request time — `[CALL]`: store timezone offset, not IANA tz name, refreshed on every snapshot pull, since the simulation only needs "what hour is it locally," and storing just the offset avoids a tz-database dependency in the simulation service).
- Active weather (`rain` dampens `vocal_frequency`-driven call hints; `wind` biases toward `alert` for some birds, `wary` for others, weighted by `boldness` — bold birds get alert, less-bold birds get wary, per `aviary_layout.md`).
- The bird's own personality vector damps/amplifies transition likelihood (high boldness reduces wary-transition probability, as specified).

Mood persists across ticks and across sessions (it's just a `Bird` row field, read fresh on every snapshot) — there is no client-side mood reset, structurally guaranteeing `bird_engine.md`'s "mood does not reset on tab open."

Daily-ish reset: each bird has a `daily_reset_at` (`[CALL]`: midnight in the account's last-known local timezone, jittered ±30min per bird so all birds in an aviary don't reset in unison, which would read as a synchronized "system event" rather than independent animals). At reset, mood is *nudged* toward a personality-weighted baseline rather than hard-reset, so the transition is gradual across the next few ticks rather than an instantaneous snap a user could catch mid-session.

### 6.4 Calibration targets and how we test them

Per `bird_engine.md`: measurable drift in instruments after ~1 week of regular visits, visible-to-user drift after ~3 weeks. This is implemented as:
- An internal-only (not user-facing, not exported) time-series snapshot of personality vectors taken at fixed intervals, used exclusively by a calibration test harness that simulates synthetic "regular visit" event streams and asserts the 1-week/3-week deltas fall in designed bands.
- This harness is part of CI for the simulation service, not a production feature, and is explicitly excluded from the export schema (§3.6) and from any per-account API surface — it exists to let engineers tune the delta-scaling constants in §6.2 without shipping a debug view that would violate "personality vector is never exposed numerically."

### 6.5 Calls (procedural, client-side)

Each species ships a small **call-grammar motif library** (a set of pitch/rhythm/timbre motif fragments, authored as data, not audio files) bundled with the client. At runtime, the client synthesizes a bird's call via WebAudio by:
1. Selecting a motif sequence from the bird's species library, weighted by mood (e.g. `content` favors gentler motifs, `alert` favors sharper ones).
2. Shaping timing/pitch by the bird's `personality_render_hints` (vocal-frequency-derived pacing, boldness-derived... `[CALL]` — pitch/timbre variance scaled by personality so two birds of the same species remain distinguishable even before considering mood).
3. Adding per-call jitter (micro-variations in timing/pitch within the motif's allowed range) so no two calls are bit-identical — this is the concrete mechanism behind "calls vary every time."
4. Mixing: the **listen-in mix** is implemented as a per-bird WebAudio gain node; engaging listen-in ramps the focused bird's gain up and all others down over ~1-2s (`[CALL]`), disengaging reverses the ramp — never a hard cut, never full silence on the de-emphasized birds (floor gain > 0).
5. **Chorus**: when 2+ birds with high `vocal_frequency` happen to have call events scheduled in overlapping windows, their independently-synthesized calls play simultaneously through the mixer — true chorus, not pre-mixed audio, avoiding the phase-cancellation artifact the PRD calls out for stacked recorded loops.

Bird-to-bird interaction (one bird's call prompting another's response, wary mood spreading) is computed server-side at tick granularity for the *probability* a bird responds (so it's consistent with personality/mood state), but the actual call *timing* within a tick window is randomized client-side within a server-given envelope, so simultaneous sessions on two devices for the same account (or a host + a visitor) don't hear bit-identical call timing, which would read as canned.

### 6.6 Idle motion

Mood-shaped idle-motion state machine, computed as `current_animation_hint` server-side (§6.1 step 7) but *rendered* client-side as a looping-but-varied animation keyed to that hint (preen-cycle, scan-cycle, head-tilt-cycle, low-fluffed-drowsy-pose), with per-instance timing jitter so two birds in the same hint state don't visibly sync. Idle motion continues advancing in the render layer even when the tab is hidden-but-the-process-isn't-suspended is moot — per `interactions.md`, the client stops rendering entirely when hidden; only the *simulation* continues server-side. On return to visible, the client re-syncs the render layer to a fresh snapshot rather than trying to "catch up" a paused animation loop.

---

## 7. Sync model

### 7.1 Cold load and the "already in motion" guarantee

1. HTML shell + a small inlined initial state-snapshot are served together from a CDN edge (the inlining avoids a second round-trip for the data needed to render the first frame — this is the specific mechanism behind the <500ms budget in §10).
2. Client computes `elapsed = now - snapshot.last_tick_at` and `[CALL]` derives a render-layer phase offset for each bird's idle-motion cycle from `elapsed` (e.g., a preen cycle is `elapsed mod cycle_length` frames into its loop) so the first paint shows birds mid-cycle, not at a neutral start pose.
3. If the snapshot fetch is slow (cold cache, slow connection), the client shows the quiet-field loading state (soft sky, one or two faint motion cues — explicitly not a spinner) until the snapshot resolves, per `aviary_layout.md`.
4. First bird then either (a) is already rendered mid-action if this is a normal load, or (b) does the soft fly-in-to-perch entrance if this is the genuine empty-aviary-to-first-bird transition immediately after adoption (the *only* case where an entrance animation is correct, because the bird genuinely has just arrived).

### 7.2 Steady-state sync

Per §4.2: visibility-change pull, long-frame-gap pull, low-frequency keepalive pull. Between pulls, the client interpolates positions/poses using the render layer (§2.3). No client ever computes a *new* authoritative state — interpolation is purely visual tweening between two server-given points, never a source of truth.

### 7.3 Multi-device and conflict handling

Because personality/mood are server-owned and additively updated (§6.2), there is no merge step: laptop and phone both pull the same `Bird` rows from the same account. The "conflict" surface that exists is narrower than data conflict — it's session/auth conflict (expired magic link, timed-out session), handled via the matter-of-fact error surfaces specified in `accounts_sync.md` and implemented as typed error responses (§4) the client renders verbatim rather than restyling.

Race case worth naming explicitly: a `listen_in_start` submitted from the laptop and a `listen_in_end` for the same bird arriving late from the phone (e.g. phone was backgrounded and flushed its buffered event on resume) — the tick job resolves this by event-log order (`received_at`), pairing starts/ends by `bird_id` and `device_session_token` rather than globally per bird, so interleaved sessions from two devices on the same bird don't cross-pair a start from one device with an end from another. `[CALL]` This pairing-key choice isn't specified in the PRD and is a defensible call: pairing by device session prevents a degenerate cross-device pairing bug at the cost of (rare) double-counted overlapping listen-in windows, which is the safer failure direction given drift is monotonic and small over-counting just means a slightly larger (still bounded, still positive) delta.

---

## 8. Frontend rendering pipeline

- **Scene composition**: a single Canvas/WebGL-or-DOM-layered scene (`[CALL]`: Canvas2D with a sprite/sprite-sheet-light approach for birds, DOM/CSS for the top bar only — full WebGL is unnecessary scope for a static-camera, no-pan scene and would cost bundle budget; Canvas2D keeps the renderer small and is more than sufficient for 60fps at this visual complexity). Three perch-zone lanes (front/middle/back) with parallax depth via scale + slight desaturation/blur on background-plane elements, not true 3D.
- **Idle micro-motion**: per-bird animation state machine driven by `current_animation_hint` + mood (§6.6), sprite/pose interpolation for normal mode.
- **Transitions**: perch-to-perch movement is a tweened path (not teleport) over the snapshot interval; mood transitions cross-fade pose sets over a few seconds so a bird doesn't visibly "snap" from wary to content.
- **Reduced-motion mode**: a parallel rendering path (not a CSS `prefers-reduced-motion` media-query toggle layered on the same animation code) — per `accessibility_perf.md`'s "different rendering of the same aviary," this plan treats it as a genuinely separate render strategy: a pose-cross-fade renderer that consumes the same `current_animation_hint`/mood state but outputs slow cross-fades between a small set of authored still poses per hint, with ambient leaf/feather ornaments disabled and day/night color transitions slowed rather than removed. Triggered by `prefers-reduced-motion` media query OR the explicit account setting (§3.1), with the explicit setting taking precedence if set (so a user can opt in even without the OS-level preference, and opt out is *not* offered if the OS preference is set — `[CALL]`: PRD doesn't specify whether OS-preference users can override back to full motion; this plan respects OS `prefers-reduced-motion` as authoritative unless the user has an explicit account-level override, consistent with standard web accessibility practice).
- **Day/night palette**: computed client-side from local time (a simple time-of-day → color-stop interpolation function), independent of server tick cadence — this is purely a render concern, the server's mood-transition use of time-of-day (§6.3) is a separate, coarser-grained consumer of the same underlying "what hour is it" fact.
- **Ambient ornaments** (leaves, feathers, weather rain/wind visuals): generated client-side at idle cadence, no per-instance server state, per `aviary_layout.md`'s explicit "not driven by the simulation tick" note — weather *presence* (is it raining) is server state (so it's consistent across devices/visitors), but the specific leaf positions/timings are pure client ornament.
- **Empty-aviary state**: same quiet-field component as the loading state, reused rather than a separate implementation, with the first-bird fly-in triggered once at the adoption-completion event.

---

## 9. Audio pipeline

Covered substantively in §6.5 (call grammar, mixing, chorus). Implementation notes:
- **WebAudio graph**: one `AudioContext`, per-bird `GainNode` feeding a shared bus, with a master gain respecting the account's `audio_enabled` setting. Listen-in implemented as gain automation (`AudioParam.linearRampToValueAtTime`) on the focused bird's node and all others, never abrupt `gain.value =` assignment, to guarantee the "slow rise/slow drop" feel is a property of the audio graph, not just convention.
- **Buffer reuse**: per `accessibility_perf.md`'s memory-growth budget, oscillator/buffer nodes created per call are explicitly disconnected and dereferenced after each call completes (WebAudio nodes are not GC'd just by going out of scope while connected); a small pool of reusable nodes is preferred over per-call allocation where the Web Audio API allows it.
- **WebAudio fallback**: feature-detect `AudioContext` availability and permission state at load; if unavailable, the client sets `captions_enabled = true` for the session (without overwriting the account's persisted setting) and renders in graceful silence — no recorded-audio fallback path exists in the codebase at all, not just unused, so there's no temptation to "just enable it" later for this case.
- **Captions**: generated client-side from the same motif-selection step that drives synthesis (§6.5 step 1-3) — the caption string is derived from which motif/mood combination was actually selected for that call instance, not a separate lookup, guaranteeing caption-matches-what-played.

---

## 10. Accessibility surfaces

- **Screen-reader narration**: a server-or-client-generated (`[CALL]`: client-generated, from the same snapshot + render-layer state the visual surface uses, templated through a small naturalist-phrase-construction module shared with the notebook generator's vocabulary/style rules — sharing the phrase-construction module is what guarantees voice continuity between narration and notebook per `accessibility_perf.md`) prose narration, pushed to an ARIA live region (`aria-live="polite"`, `[CALL]` upgraded to a brief `assertive` nudge only for the user-initiated priority events named in the PRD: return-greeting, offer reaction, settle). Cadence: idle narration update every 30-60s (`[CALL]` 45s baseline), generated from current mood/animation-hint state rather than literal event-log entries, so the narration reads as observation rather than as a transcript.
- **Keyboard navigation**: top-bar items in normal tab order; Tab from the last top-bar item moves focus into the aviary scene onto the first bird (DOM order following perch zone front-to-back, left-to-right within zone); Arrow keys move focus between birds; Enter triggers listen-in on focused bird; Escape exits listen-in; offer affordance has both a top-bar focus target and, once open, full keyboard navigation of its small option set; settle reachable via the top bar's normal tab order. Focus ring: a high-contrast outline drawn with sufficient contrast against both bright and dim aviary backgrounds (`[CALL]`: a two-layer outline — a dark inner ring + light outer ring, or vice versa depending on local brightness sampling — so it remains visible regardless of the underlying scene color at that moment, since the aviary background is dynamic, unlike a typical static-UI focus ring).
- **Captions**: per §9, rendered as small fading text anchored near the calling bird's screen position, naturalist phrasing, same generation source as the audio.
- **Contrast**: WCAG AA enforced via the component library / design tokens for all top-bar, settings, and error-surface text; verified in CI via an automated contrast-check pass over the design-token palette (`[CALL]`: not a runtime check, a build-time lint against the token set, since the aviary scene itself doesn't carry arbitrary user copy).

---

## 11. Performance budgets and observability

### Budgets (restated as engineering targets with the implementation lever that hits them)
| Budget | Target | Primary lever |
|---|---|---|
| Initial JS bundle | <2MB gzipped | Procedural audio (no recorded files), code-splitting for settings/accessibility-settings/visit-invitation routes, SVG/compact-bitmap bird assets, no WebGL engine dependency |
| Time to first bird visible | <500ms, mid-tier mobile/4G | CDN-edge-delivered inlined initial snapshot, render path that doesn't block on non-critical assets (audio motif library can lazy-load fractionally after first paint) |
| Idle-motion frame rate | 60fps, 5-year-old laptop | Canvas2D sprite approach over heavier rendering tech, bounded per-frame work in the idle-motion state machine |
| Memory growth | none over 30 min | WebAudio node pooling/disconnection (§9), notebook entries virtualized/derefed on scroll-out, bounded worker/audio-context lifecycle |

### Observability
- **Synthetic checks**: scheduled automated-browser runs from common geographies measuring time-to-first-bird and render-frame timing against the budgets above, alerting on regression.
- **Aggregate RUM**: page load timings, first-bird-render timings, render-frame timings, audio-context error counts, simulation-tick latency (server-side instrumentation, but reported through the same aggregate-telemetry pipeline) — all explicitly excluding any per-bird state or per-account interaction history, enforced by the telemetry pipeline's schema (the emission layer only has access to scalar timing/error fields, never a reference to the account's `Bird` rows) rather than by a downstream filtering policy that could be misconfigured.
- **Tick latency alarm**: p99 simulation-tick latency alarms above 5s.
- **Privacy boundary as an architectural property, not a policy**: the analytics warehouse's database credentials have no grant on the simulation service's database; the only path data takes from simulation → analytics is the explicit aggregate-emission calls in §6.1/§9, which by construction cannot carry a personality vector or event payload (the emission function's type signature only accepts the scalar metric fields listed above) — this is the concrete implementation of `accounts_sync.md`'s "this reads as a privacy claim and behaves as an architectural rule."

---

## 12. Rollout

- **Birds-per-aviary ramp**: two starter birds at signup (server-selected species, not user-chosen, per `bird_engine.md`'s "meeting an animal, not configuring an avatar"). Third-and-beyond species offers triggered by `Aviary.created_at` age thresholds (`[CALL]`: starting target — third bird offered around the 2-3 month mark, continuing roughly through year one toward five-to-six birds, exact thresholds tuned post-launch from real retention/engagement data *that does not itself become a user-facing counter* — the tuning data is operational, not a feature). This is implemented as a simple age-threshold check at tick time, not a separate "milestone" subsystem, to avoid building infrastructure that looks suspiciously like the gamification-milestone machinery the PRD refuses.
- **Launch sequencing**: simulation service + tick correctness (with the calibration harness from §6.4) is the critical path and ships first internally, validated against synthetic event streams before any client work depends on it. Accessibility surfaces (narration, reduced-motion, captions) ship *with* v1, not as a follow-up — per `accessibility_perf.md`'s explicit instruction that a post-launch reduced-motion mode "quietly told reduced-motion users that the product wasn't for them." This is a hard gate on launch readiness, not a stretch goal.
- **Instrumentation from day one**: the aggregate-telemetry pipeline (§11) and the calibration harness (§6.4) both ship before public launch, since drift calibration and performance regressions are both effectively unfixable-after-the-fact in their effect on early users' trust (a user who experiences mis-calibrated drift in week one forms an impression of the product that's hard to undo, and the PRD's "feels alive" claim depends on exactly this not happening).
- **Visit feature**: ships in v1 per `social_optional.md`, but defaults off and requires no separate rollout ramp beyond the rest of the product — it's structurally a read-only consumer of the same snapshot API already built for the host's own client (§4.2), so there's no separate "social infrastructure" rollout phase.

---

## 13. Risks

- **Drift calibration is the single highest-risk surface.** Too fast and the product becomes a Tamagotchi-with-extra-steps where users learn to click for visible reward; too slow and it reads as a screensaver where nothing the user does matters — and there is no UI affordance to course-correct a user's perception once formed, since the product structurally refuses to ever tell the user a number. Mitigation: the calibration harness (§6.4) gates the delta-scaling constants in CI against the explicit 1-week-instruments / 3-week-visible targets, and post-launch the only safe lever is tuning the *scale* of existing additive deltas (never their sign, never adding negative-drift paths), reviewed before any change ships.
- **Sync correctness under concurrent multi-device sessions** is the second-highest risk because failures are silent by construction (per `accounts_sync.md`: "the user never sees the failure — they just have a bird that's drifting more slowly than it should"). Mitigation: per-account tick serialization (§6.2), database-level write restriction on `personality` columns to the tick job's role only, and the device-session-keyed event pairing (§7.3) for the listen-in race case — all three are structural guarantees, not test coverage alone, because a silent failure here may never surface in test data.
- **Audio uncanniness** — procedural calls that sound "almost real" but land in an uncanny-valley zone are arguably worse than honestly synthetic-sounding calls, and this is a perceptual/design risk that engineering correctness alone won't catch. Mitigation: this plan treats the motif library and personality-to-timbre mapping (§6.5) as requiring iterative listening-test passes before launch, not a one-shot implementation — flagged here because it's the one area of this plan where "build it per spec" is necessary but not sufficient; it needs design/audio review cycles the rest of the engine doesn't.
- **Accessibility regression risk concentrates at the reduced-motion/narration boundary**, because it's the surface most likely to silently degrade under future feature work (a contributor adds a new idle-motion state for sighted users and forgets the corresponding still-pose set for reduced-motion, or a new mood-driven narration phrase). Mitigation: §6.6/§8's shared `current_animation_hint` contract means every new hint *must* have both a full-motion and a reduced-motion rendering registered, enforced by a build-time check that the reduced-motion pose-set registry has an entry for every hint the normal-motion state machine can emit — making the omission a build failure rather than a silent visual gap.
- **The gamification-foothold risk is organizational, not technical** — per `non_goals.md`'s own framing, the most likely failure mode is an incremental, individually-reasonable feature (an unread badge, a "visits this week" admin dashboard that later gets exposed, a "your friend visited!" default flipped on by a well-intentioned growth experiment). Mitigation: this plan deliberately refuses to build the *infrastructure* for these features (no engagement-aggregation table, no notebook-unread state, no cross-account ranking computation anywhere — §3.6, §4.5) specifically so that adding them later requires new schema and new computation, not a config flip, raising the bar for the "just enable it" failure mode the PRD repeatedly names as the actual threat.

---

## 14. Open implementation calls summary

For a reviewer scanning quickly, every `[CALL]` above in one place:
1. Personality/mood stored as JSON columns on `Bird`, not normalized tables (§3.2).
2. Presence activity window starts at 3 minutes, server-configurable (§5).
3. Presence accrues anywhere in-app, not just the aviary route (§5).
4. Tick implemented as per-account queue-driven worker pool, jittered cadence (§6.1).
5. Weather start probability ~1-2%/tick when none active, tuned to "a few times a week" (§6.1).
6. Mood daily reset at local midnight ±30min jitter per bird, nudge not snap (§6.3).
7. Listen-in event pairing keyed by `(bird_id, device_session_token)` (§7.3).
8. Canvas2D rendering approach over WebGL (§8).
9. Reduced-motion: OS preference authoritative unless explicit account override set (§8).
10. Screen-reader narration cadence baseline 45s, client-generated (§10).
11. Notebook has no unread/badge state at all, interpreted as in-scope of the no-streak-counter spirit (§4.5).
12. Third-bird-and-beyond age thresholds: illustrative starting points only, tuned post-launch from non-user-facing operational data (§12).

None of these calls touch a non-goal or a named hard rule (monotonic drift, no personality exposure, no last-write-wins, naturalist/matter-of-fact voice split, no gamification surfaces) — they are scoped to genuinely unspecified tuning constants and minor architectural choices the PRD leaves to implementation.
