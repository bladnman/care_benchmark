# Pocket Aviary — v1 Implementation Plan

This plan turns the Pocket Aviary PRD into an executable build for a frontier engineering team. It is organized so a backend, frontend, audio, and accessibility specialist can each find their lane and the contract that connects them. The deliverable here is the plan; no product code is written.

Throughout, two design rules from the PRD are treated as architectural invariants rather than guidelines, because the rest of the plan is shaped to make violating them *hard*:

1. **The server is the only writer of personality state.** Clients emit interaction events; they never mutate vectors. (`accounts_sync.md`, `bird_engine.md`)
2. **Presence is the conjunction of three independently-checkable signals**, and is the dominant drift input, so it must be measured honestly. (`concepts.md`, `interactions.md`)

Where the PRD leaves a value to "calibration during build," this plan picks a defensible default, marks it **[CALIBRATION]**, and routes it through config rather than hard-coding it, so the calibration loop is a config + test exercise, not a code change.

---

## 1. Scope

### In scope for v1

- **Aviary core**: single horizontal scene, 2 starter birds, cap of 7, three perch zones, day/night on user-local time, rare ambient weather, ambient leaf/feather drift.
- **Bird engine (server-authoritative)**: per-bird hidden personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity); fast-timescale mood enum; slow-timescale drift via low-pass filter; bird-to-bird interaction; species pool (~6 species).
- **Server-side simulation tick** (~1/min) advancing canonical state from the append-only event log.
- **Interactions**: return-greeting, listen-in, offer (seed / song-fragment / still-pool) with per-bird cooldown, settle (with 5s undo), field notebook (auto-generated, read-only, sparse), presence accounting.
- **Rendering**: scene composition, mood-shaped idle micro-motion, snapshot interpolation, loads-already-in-motion, top-bar chrome with fade, reduced-motion mode as a designed surface.
- **Audio**: procedural per-bird call synthesis (WebAudio), real-time chorus mixing, listen-in mix re-balance with slow ramps, silence+captions fallback.
- **Accounts/sync**: magic-link auth, synthetic account UUID, per-device revocable sessions, email change with verification, account export, soft-then-hard deletion, multi-device sync as an emergent property of server-canonical state.
- **Social (single affordance)**: per-invite, opt-in, revocable, read-only ambient visit; visit log; invite expiry; default-off visit notifications.
- **Accessibility (first-class, ships with v1)**: naturalist screen-reader narration, reduced-motion mode, call captions, keyboard navigation, WCAG AA contrast on chrome/copy.
- **Performance**: <2MB gzipped initial bundle, <500ms time-to-first-bird on mid-tier mobile/4G, 60fps idle on a 5-year-old laptop, no memory growth over 30 min.
- **Observability/privacy**: aggregate-only telemetry, synthetic perf checks, hard telemetry/simulation data boundary.

### Explicitly out of scope (non-goals respected)

Native apps; any gamification (achievements, streaks, levels, scores, badges, counters, green-dot calendars, "visited X days"); Tamagotchi mechanics (death, hunger, distress, decay, happiness meters, negative drift on neglect); social-network surfaces (profiles, follows, feeds, discovery, leaderboards, comments, co-presence, chat, avatars, friend-of-friend); push/email engagement notifications; payments; shared/multi-aviary accounts; customizable scenes; user-controlled perch placement; any numeric exposure of the personality vector; any per-account aggregation of interaction data.

### Standing scope guards (engineering checklist used at PR review)

These exist because the PRD names them as the temptations a well-meaning contributor will reach for. They are review gates, not docs:

- No textual "welcome back" on return, in any form (toast, banner, modal, "gone X days").
- No surface that reports the user's own visit behavior (only observations of the aviary).
- No code path where a client writes a personality vector value.
- No code path where the simulation DB is read by analytics, or per-bird fields enter telemetry.
- No recorded-audio asset in the audio pipeline.
- No spinner / fade-from-static on aviary load.
- No reduced-motion path that merely disables animation.

---

## 2. Architecture

### Service shape

A small set of services with a hard data boundary between simulation and analytics.

- **Edge/web tier**: serves the SPA shell + a small inlined initial state snapshot (for the <500ms budget). CDN-fronted. Stateless.
- **API service** (`api`): REST/JSON + a thin realtime channel. Handles auth, snapshot reads, event-log writes, account settings, export, social invites. Validates and authorizes; never computes drift.
- **Simulation service** (`sim`): owns the tick. Reads the event log + current canonical state, computes drift deltas + mood transitions, writes canonical state. The **only** writer of personality vectors. Runs as a scheduled worker fleet, sharded by account UUID.
- **Auth service** (`auth`): magic-link issuance/consumption, session token lifecycle, email change/verify. Owns the single encrypted email field.
- **Notebook generator**: a sub-component of `sim` (runs on the tick) that decides—sparsely—whether to emit a notebook entry and renders its prose.
- **Narration generator**: produces the screen-reader prose stream from the same canonical state the visual reads; runs server-side (preferred) with a client-side renderer for assembling the live narration. (See §9.)

```
                +-------------------+
   browser <--->|  edge / CDN (SPA  |
                |  shell + inlined  |
                |  state snapshot)  |
                +---------+---------+
                          |
        snapshot reads /   |   event writes / auth / settings
        social / export    v
                +-------------------+        +------------------+
                |       api         |<------>|      auth        |
                +----+----------+---+        +------------------+
   reads canonical    |          | append-only writes
   state (RO)         v          v
        +-------------------+   +-------------------------+
        |  canonical state  |   |   interaction event log |
        |  store (RW: sim   |   |   (append-only)         |
        |   only)           |   +------------+------------+
        +---------+---------+                |
                  ^   consumes log in order  |
                  |   writes canonical state |
            +-----+--------------------------+
            |        sim (tick worker fleet) |
            +--------------------------------+

   analytics warehouse  <---  aggregate telemetry only
   (NEVER reads canonical state / event log per-bird fields)
```

### Client/server split

- **Server owns**: personality vectors, mood, drift history, canonical positions/perch choices, mood timers, weather state, day/night anchor, notebook entries, narration prose, accounts, sessions, invites.
- **Client owns**: rendering, interpolation between snapshots, procedural audio synthesis, ambient ornaments (leaves/feathers — no server state), input capture, presence-signal detection, top-bar fade, reduced-motion rendering register, caption rendering.

### The render pipeline boundary (the contract that protects "feels alive")

The server emits **intent + current phase**, not frames. A snapshot says "Pip is on the front perch, mood content, mid-preen, call grammar seed S, next-call window W." The client is responsible for turning that into continuous motion and sound. This split is deliberate: it lets the server tick slowly (~1/min) while the client renders 60fps, and it makes "loads already in motion" trivial — the client receives a state that is *already mid-action* and renders from there, never from a zero/idle state. The boundary rule: **the server never sends "start" events for ambient life; it sends the current state of ongoing life.**

---

## 3. Data model

All IDs are synthetic UUIDs. Email appears exactly once (encrypted, on the account record). Personality values are server-internal and never serialized to any client-facing payload.

### Account

```
Account {
  account_id: UUID (PK, used everywhere: shards, logs, telemetry, messages)
  email_encrypted: bytes            // the ONLY place email lives
  email_verified: bool
  created_at, updated_at
  status: enum { active, pending_deletion }
  deletion_requested_at: timestamp?  // soft-delete window anchor
  settings: AccountSettings
}

AccountSettings {
  reduced_motion: enum { system, on, off }   // 'system' honors prefers-reduced-motion
  captions: enum { on, off }                 // forced on when WebAudio unavailable
  visit_notifications: bool (default false)
  audio_enabled: bool
}

Session {
  session_id: UUID
  account_id: UUID
  device_label: string              // best-effort UA summary, no PII
  created_at, last_seen_at
  revoked_at: timestamp?
}
```

### Aviary & Bird

```
Aviary {
  aviary_id: UUID
  account_id: UUID (1:1 at v1)
  created_at                        // drives "add a third bird" pacing (age, not visits)
  weather_state: WeatherState
  // day/night is derived from client-reported local time, not stored as a clock
}

Bird {
  bird_id: UUID (STABLE FOREVER; never reassigned on rename/sync/migration)
  aviary_id: UUID
  species_id: enum (from ~6-species pool)
  name: string (user-assigned, renameable)
  personality: PersonalityVector    // SERVER-INTERNAL, never exposed numerically
  mood: MoodState
  perch_zone: enum { front, middle, back }
  call_grammar_seed: bytes          // stable per bird; shapes recognizable signature
  adopted_at
  drift_filter_state: LowPassState  // internal filter accumulators (see §5)
}

PersonalityVector {
  boldness: float (normalized small range, e.g. [0,1])
  social_warmth: float
  vocal_frequency: float
  plumage_saturation: float         // monotonic non-decreasing
  curiosity: float
}

MoodState {
  mood: enum { wary, content, curious, drowsy, alert }
  entered_at: timestamp
  // mood persists across sessions; never reset to neutral on tab open
}

WeatherState {
  kind: enum { clear, rain, wind }
  started_at, ends_at
}
```

### Events, notebook, narration, social

```
InteractionEvent {                  // APPEND-ONLY; clients write these
  event_id: UUID
  account_id: UUID
  bird_id: UUID?                     // null for aviary-wide events
  type: enum { presence_ping, listen_in_start, listen_in_end,
               offer_seed, offer_song, offer_pool, settle, settle_undo }
  client_ts, server_ts              // server_ts is authoritative ordering key
  payload: jsonb                     // e.g. presence dwell ms, song-fragment id
}

NotebookEntry {
  entry_id: UUID
  account_id: UUID
  created_at
  local_date_label: string          // "tuesday" (user-local, naturalist)
  prose: string                     // lowercase present-tense naturalist
}

NarrationSnapshot {                 // ephemeral / short-retention
  account_id, generated_at
  prose: string
  priority: enum { idle, event }
}

VisitInvite {
  invite_id: UUID
  host_account_id: UUID
  visitor_email_encrypted: bytes
  token_hash: bytes                 // one-time link
  created_at, expires_at (created_at + 30d)
  state: enum { outstanding, active, used, revoked, expired }
}

VisitLogEntry {
  visit_id, host_account_id, invite_id
  visitor_email_encrypted: bytes
  started_at, approx_duration_s
}
```

### Modeling notes / defensible calls

- **Personality range** [CALIBRATION]: normalized [0,1], seed values mid-low so there's room to drift up. Plumage saturation has a non-decreasing constraint enforced at the write layer in `sim` (a guard, not just convention) — this is the load-bearing "monotonic toward expressive."
- **Event log is the source of truth for *inputs*; canonical state is the source of truth for *outputs*.** Personality is never recomputed from the full log at runtime (PRD: "never derived from session history"). The log is consumed incrementally by the tick with a per-account cursor; old events can be retention-trimmed once consumed (subject to the 30-day deletion window) without affecting state.
- **Day/night is not stored as server clock.** The client reports its local time/timezone with snapshot requests; the server stamps mood/lighting intent against that. This honors "the user's morning is the aviary's morning" without a stored per-account clock that could drift.

---

## 4. API surface

JSON over HTTPS. Auth via session token (httpOnly cookie + CSRF token, or bearer for the SPA — pick cookie for magic-link simplicity). All account references in URLs/bodies are the resolved session's `account_id`; never email.

### Auth
- `POST /auth/request-link` `{email}` → 202 always (no account enumeration). Rate-limited per email.
- `POST /auth/consume` `{token}` → sets session; one-time, invalidated on use; 15-min expiry. Errors use matter-of-fact copy.
- `POST /auth/email-change/request`, `POST /auth/email-change/verify` — old email valid until new verifies.
- `GET /sessions`, `POST /sessions/{id}/revoke`.

### Aviary state (read path)
- `GET /aviary/snapshot?local_time=<iso>&tz=<iana>` → **Snapshot** (kilobytes): per-bird `{bird_id, species_id, name, perch_zone, mood, motion_phase, call_window, plumage_render_tier}`, aviary `{lighting_phase, weather_state}`, server time. **Never includes raw personality floats** — only render-facing derived fields (e.g. `plumage_render_tier` is a quantized visual bucket, not the saturation scalar). This is the enforcement point for "never exposed numerically."
- Snapshot is also inlined into the initial HTML from the edge for the <500ms budget.
- Pulled on: visibility change, long render-frame gap (suspend recovery), low-frequency keepalive while visible.

### Interaction events (write path)
- `POST /events` `{type, bird_id?, payload, client_ts}` → 202, appended to log. Batched: client may POST an array (presence pings batch with other events to limit requests). Server stamps `server_ts` for ordering. **No endpoint accepts a personality value.**
- Offer cooldown enforced server-side (a few minutes per bird per offer type [CALIBRATION ~3min]); client also greys the affordance, but server is authoritative.

### Notebook
- `GET /notebook?cursor=...` → paginated entries, newest first, read-only. No write/delete/annotate endpoints exist (absence is the design).

### Narration (accessibility)
- `GET /narration/stream` (SSE or poll) → prose updates at idle cadence (1 per 30–60s [CALIBRATION]), priority-bumped on user events. Generated from the same canonical state as the snapshot.

### Account
- `GET/PATCH /account/settings`
- `POST /account/export` → generates JSON snapshot, emails verified address a download link.
- `POST /account/delete` (soft), `POST /account/restore` (within 30d).

### Social (visit invitation flow)
- `POST /invites` `{visitor_email}` → creates `VisitInvite`, emails one-time link. Opt-in per invite; visits default off (no invite → no sharing).
- `GET /invites` → outstanding + active.
- `POST /invites/{id}/revoke` → state=revoked; next visitor snapshot pull returns "visit no longer available" (matter-of-fact). No host notification of success.
- `GET /visit/{token}` (unauthenticated visitor path) → resolves to a **read-only** snapshot stream of the host aviary. Visitor events are **never written** to the host event log (enforced: the visit path has no `/events` access). Visitor sees exactly the host's current state (no show-off rendering).
- `GET /visit-log`, settings toggle for `visit_notifications` (default false).

### Error/voice contract on the API
- Any response the user reads as "talking to the system" (auth, settings, sync, export, unsupported browser, visit-unavailable) carries matter-of-fact copy. Everything narrative (notebook, narration) carries naturalist prose. This is encoded as a `voice` discriminator on error/copy payloads so the split is mechanical, not per-developer judgment.

---

## 5. Simulation engine design

The engine is the product. It runs entirely in `sim`, on the tick.

### The tick

- Cadence ~1/min [CALIBRATION]. Sharded by `account_id`. Idempotent and resumable: each account has a `log_cursor` (last consumed `server_ts`/event_id) and a `last_tick_at`.
- Per account per tick:
  1. Read canonical state + new events since `log_cursor`.
  2. Compute **drift deltas** (slow timescale) from presence-time + interaction signals.
  3. Compute **mood transitions** (fast timescale) from recent interactions, time-of-day (from last client local-time report, extrapolated), ambient/weather events, and personality.
  4. Advance **weather** (rare, scheduled) and **bird-to-bird** propagation.
  5. Possibly emit a **notebook entry** (sparse gate).
  6. Possibly refresh **narration** prose (if a narration consumer is active).
  7. Write canonical state; advance `log_cursor` and `last_tick_at`.
- The tick runs whether or not a client is connected. A long-idle account still ticks (mood through dusk→night), but with no presence events it accrues no drift.
- **Catch-up**: if `sim` was down or an account wasn't ticked, the next tick processes elapsed wall-time in bounded sub-steps (capped to avoid unbounded loops after long outages) so mood/day-night land correctly without replaying per-minute for months.

### Drift function (slow timescale, monotonic-up)

Drift is a **low-pass filter** over a per-tick "expressive signal." For each trait `t`:

```
signal_t = w_presence * presence_time_in_window
         + w_listen   * listen_in_attention(bird, window)   // social_warmth, vocal_freq
         + w_offer     * offer_signal(bird, window)          // curiosity (+accept), boldness (+offer-near)
delta_t  = alpha * max(0, signal_t)        // alpha small → slow; max(0,...) → monotonic up
trait_t  = clamp(trait_t + delta_t, 0, 1)  // additive, server-authored
```

- **Weights, rough order** (from PRD): presence-time dominant; listen-in strong; offers small; settle ~0 directional (only ends presence window cleanly).
- **Monotonic toward expressive** is enforced two ways: `max(0, signal)` so neglect contributes nothing, and plumage's hard non-decreasing write guard. Neglect → **ambient**, never wary/sad/faded.
- **Calibration targets are tests, not vibes**: a harness simulates "regular visits" and asserts (a) *measurable* delta after ~1 week of simulated regular presence, (b) *visible* delta (crossing a render-tier or greeting-order threshold) after ~3 weeks, (c) no single session moves a trait into a visibly different state. `alpha` and the weights are tuned against these assertions. **[CALIBRATION]** lives in a versioned config (`drift_params.vN`) so re-tuning is a config bump + test run.
- **Presence-time is the input, and it must be honest** (see §6 for the client signal; the engine consumes already-validated presence dwell from the event payload and additionally sanity-caps per-window dwell so a malformed client can't inflate drift).

### Mood transitions (fast timescale)

- A small enum FSM (`wary, content, curious, drowsy, alert`) with transition propensities, not hard edges. Inputs: recent-session interactions (offer accepted → toward content; alarm/wind → toward wary/alert), time-of-day (dusk→drowsy, early-morning→alert), weather (rain dampens vocal frequency briefly), and the bird's own personality (high boldness resists `wary`).
- **Persists across sessions**: mood at session-end is mood at next session-start, modulo tick evolution. Never snap-to-neutral on tab open. (Stored on `MoodState.entered_at`; transitions are time-driven on the tick, so an absent user's bird still moves dusk→night naturally.)
- **Daily-ish reset** is a soft re-baseline, not a hard wipe — implemented as time-of-day pull dominating after long idle, not as an explicit "reset to content."

### Bird-to-bird interaction

- On the tick, a bird's call/mood can influence neighbors: a wary mood spreads with a decaying probability to nearby (same/adjacent perch) birds; high-vocal-frequency birds calling in the same window can form a **chorus event** flag in the snapshot (which the client renders as overlapping calls). This is what makes the aviary a small social system, not independent NPCs.

### Call-grammar runtime (server side of it)

- The server does **not** synthesize audio. It owns each bird's **call grammar seed** (stable, per bird) and emits, in the snapshot, the *current call intent*: which motifs are active, timing window shaped by `vocal_frequency`, pitch/timing modifiers shaped by mood. The client synthesizes from this (see §7). Recognizability-across-drift is guaranteed by keeping the seed/motif library stable per bird while only timing/pitch modulate.
- **Caption text** is derived from the same call-intent so a caption always matches what was actually played (the caption is generated, not a stored string).

### Notebook generation (sparse, naturalist, observations-of-the-aviary-only)

- Runs on the tick. A **sparsity gate** ensures ~1 entry per few days even for very active users [CALIBRATION]: a cooldown since last entry + a "noteworthiness" score (e.g., a first-this-week greeting-order flip, a long quiet stretch, a weather moment) must clear a threshold.
- Prose is templated against a naturalist grammar (lowercase, present-tense, bird-named, specific), with procedural variation so entries don't repeat phrasing.
- **Hard content rule, enforced in the generator**: entries describe the *aviary* (birds, moments, weather), never the *user's behavior* (no "you visited", no frequency, no streak). This is the notebook's expression of "observations of the aviary, never observations of the user."

---

## 6. Sync model & presence

### Sync = server-canonical, not a feature

- One canonical record per aviary, in the simulation store. `sim` is the only writer of personality/mood. Clients read snapshots and write events. Therefore two devices are reading the **same** record — multi-device sync is emergent, with no client-to-client sync, no merge, no eventual consistency.
- **No last-write-wins on personality**: clients submit events ("listened in to Pip 3 min"), never absolute vectors. The tick applies **additive deltas in event-log order**. The morning-laptop/lunch-phone overwrite failure is unreachable because no path lets a client write a vector. Code-level guard: the canonical store's personality columns are writable only by the `sim` service role; `api` has read-only grants on them. This is enforced at the DB permission layer, not just in code review.
- **Conflict surfaces** (magic-link replay, mid-write session timeout, outage) resolve to matter-of-fact error copy; they never ask the user to arbitrate personality state because there is nothing to arbitrate.

### Presence accounting (the honest signal)

Client-side presence detector records a presence-event only when **all three** hold simultaneously:
1. `document.visibilityState === 'visible'`,
2. document has window focus (`document.hasFocus()`),
3. a `pointermove` or `keypress` occurred within the activity window **[CALIBRATION: lean long, e.g. ~3–5 min]**, because watching birds without moving is the actual product.

- The detector emits `presence_ping` events carrying *validated dwell time* (it only counts intervals where the conjunction held). It stops counting when the tab hides/blurs or activity lapses. The server additionally caps per-window dwell so a buggy/hostile client cannot inflate drift across the population.
- **Settle and tab-close are both terminal and equivalent** at the engine level: both end the presence window; neither is penalized; no "you didn't settle" surface. Settle additionally emits a `settle` event (mood-quieting, ~0 drift direction).
- **Background tab**: client stops *rendering* (battery), but presence simply isn't accruing (condition 1 fails) — and the *server* keeps ticking. The aviary on return is the one that has been running.

---

## 7. Frontend rendering pipeline

### Scene composition

- Single horizontal scene, no pan/scroll/zoom. Three perch zones map to depth planes: front (near/large), middle, back (far/small). Subtle foreground/background parallax; **not** parallax-heavy.
- Rendering target: Canvas/WebGL for the scene (birds + ambient motion) to hit 60fps; DOM for the top bar and accessibility surfaces (so focus/ARIA are real elements). Birds are compact SVG/procedural sprites assembled per species; plumage saturation maps to a quantized **render tier** (from the snapshot), never a raw float.
- **Responsive**: scene compresses horizontally on narrow viewports and widens on desktop **without ever cropping a bird out of frame**. Aspect handling lives in a rendering-spec module; the invariant ("all birds always visible") is a render-time assertion in dev builds.

### Loads-already-in-motion

- Boot sequence: read inlined initial snapshot → place each bird at its current `perch_zone` and `motion_phase` → start the render loop *mid-motion*. **No** spinner, fade-from-static, wake-up animation, or entry sequence. First frame is birds mid-action + a drifting leaf.
- Slow-snapshot / cold-cache fallback is a **quiet field** (soft sky, 1–2 faint motion cues), never a spinner. Empty-aviary (post-adoption, pre-first-bird) reuses the quiet field; first bird enters with a soft fly-in, then the empty state is never seen again.

### Idle micro-motion (mood-shaped)

- Continuous procedural micro-motion: preening, scanning, head-tilt toward sounds, weight-shuffle. **Mood-shaped** so mood is read from motion without a label: wary → further back + more scanning; content → preening; curious → tilts toward sounds/leaves; drowsy → low + fluffed. No status icon, tooltip, or label ever states mood.
- Motion is driven by client-side procedural generators seeded per bird (so it varies, never loops identically), parameterized by the snapshot's mood + personality-derived render hints.

### Transitions / interpolation

- Between snapshots the client interpolates: a bird moving perch A→B is animated along a path (full-motion mode) and never teleports. Chorus flags from the snapshot drive overlapping call rendering.

### Reduced-motion mode (designed surface, not a kill switch)

- Honors `prefers-reduced-motion` or explicit setting. Micro-motion → slow cross-fades between still poses; flight → cross-fade between perches (no animated path); ambient leaf/feather drift removed; day→evening color shifts **remain**, slowed. Calls still play (or caption); birds still drift; mood still changes; notebook still notices. It is a calmer register of the same aviary, with its own quiet aesthetic — explicitly not "animations off."

### Top bar & chrome

- Thin top bar above the scene: account/settings, accessibility settings, notebook, offer affordance. Nothing else. **No UI chrome inside the scene** (no in-scene buttons/badges/tooltips/overlay icons/inline labels).
- Top bar fades nearly transparent after a few seconds of cursor stillness; returns to full opacity on cursor/keyboard activity. (Fade is a render-only concern; it does not affect keyboard reachability — focus still surfaces it.)
- Ambient leaf/feather drift is **pure client ornament** (no server/per-leaf state), generated at idle cadence.

---

## 8. Audio pipeline

### Procedural synthesis (non-negotiable, no recorded audio)

- WebAudio graph per aviary. Each bird has a **call generator** built from its species motif library + per-bird seed, with timing/pitch modulated by `vocal_frequency` and mood from the snapshot. Output is synthesized at runtime — never a sample/loop. Recognizable-across-drift because the motif set + seed are stable while only modulation moves.
- **Chorus** is real-time mixing of independent procedural voices (not stacked loops — which would phase-cancel audibly). Multiple high-vocal-frequency birds calling in the same window produce an emergent chorus.

### Listen-in mix

- Focusing a bird (click/tap/keyboard) **slowly ramps** that bird's mix level up while others **slowly drop to ambient — never to silence** (a re-balance, not a mute/solo). Disengage (re-click, focus another, click empty space, move keyboard focus away, Escape) ramps back to ambient with the same slow curve. No hard cuts (a hard cut makes it a soloable-tracks UI — wrong product).

### Decay / settle

- Settle quiets calls generally over the slow evening shift. Night dims most calls; the **nightjar-like species remains active** and may call into late hours (night is not a dead state).

### Performance & fallback

- **Reuse audio buffers/nodes**; bound the WebAudio node graph; no per-call allocation that isn't freed (feeds the "no memory growth over 30 min" CI test). Audio contexts are bounded and reused.
- **WebAudio unavailable** (old browser, permission denied, hardware): graceful **silence + captions on by default**. No recorded-audio fallback path exists (unconditional rule). Silence-with-captions beats canned audio.

---

## 9. Accessibility surfaces

Accessibility ships **with** v1, designed for charm, not parity-by-checklist. A reduced-motion / screen-reader / audio-off user gets an aviary that *feels alive*, not a stripped variant.

### Screen-reader narration

- Running **naturalist prose** (lowercase, present-tense, bird-named, specific) generated from the **same canonical state** the visual reads — not ARIA-label automation, not a state list. Delivered via `/narration/stream` and rendered into a polite `aria-live` region.
- Cadence slow: ~1 update / 30–60s at idle [CALIBRATION], faster only on user events (return-greeting, successful offer, settle) which get a priority bump in the queue but are still written as observations.
- Voice continuity: narration and notebook read as the same product.

### Captions for calls

- Opt-in (forced on when WebAudio unavailable). Short prose descriptions of the actual call in current mood ("a soft three-note rise"), **generated from the call grammar at runtime** so each caption matches what played. Rendered as small text near the calling bird, fading with the call, naturalist voice.

### Keyboard navigation

- Tab cycles top-bar items; Tab into the scene focuses the first bird; arrow keys move focus between birds; Enter triggers listen-in on the focused bird; Escape exits listen-in. Offer affordance opens via top-bar shortcut and is fully keyboard-navigable. Settle reachable from the top bar.
- Visible focus indicator: soft high-contrast outline legible against both bright and dim aviary states (exact treatment from the design system).

### Contrast

- All user-copy (top-bar labels, settings, account/error surfaces, captions, any visually-displayed narration) passes **WCAG AA** minimum; design system specifies per-surface ratios. The scene itself carries no copy except the top bar, so the constraint targets the chrome.

---

## 10. Performance budgets & observability

### Budgets (treated as gates, with CI tests)

- **Initial JS bundle <2MB gzipped** at first paint. Drives: procedural audio (can't carry recorded audio at needed variation), procedural/compact bird assets, **aggressive code-splitting** of less-frequent surfaces (account settings, accessibility settings, visit-invite flow). Enforced by a bundle-size CI check.
- **Time-to-first-bird <500ms** on mid-tier mobile / 4G. Achieved via: inlined initial snapshot delivered with the HTML from a CDN edge; a render path that draws the first bird before non-critical assets; no blocking spinner. Measured by synthetic checks + RUM first-bird-render timing.
- **60fps idle on a 5-year-old mid-range laptop**, sustained over a 30-min session (runtime budget, not just launch). Measured via RUM render-frame timing + synthetic long-session runs.
- **No memory growth over 30 min** — a real CI test (headless long session asserting bounded heap): reused audio buffers, no leaked per-call allocations, notebook entries scrolled out drop references, bounded workers/audio contexts.

### Observability (aggregate-only; privacy boundary enforced at metric definition)

- Collect: request counts, latencies (incl. **simulation-tick latency**), error rates, anonymized session-duration histograms (no per-account dimension), client render-frame timing, audio-context error counts, first-bird-render timing.
- **Deliberately do NOT collect / aggregate**: per-bird state, per-account interaction history, anything reconstructing a user's relationship with their aviary.
- **Error budget**: simulation-tick latency **p99 alarms at >5s** (tick should be far faster; early-warning before users feel the aviary "running slow").
- Synthetic perf fleet: automated browsers running the aviary on a schedule from common geographies (first-bird, frame timing, audio errors).

---

## 11. Privacy & data boundary (architectural, not policy)

- **Synthetic UUID everywhere**; email encrypted in one place. No service derives identifiers from email. This is enforced by review + by the schema (no email column outside `Account`).
- **Hard pipeline boundary**: telemetry pipelines never touch the simulation DB; the simulation DB / event log is never read by the analytics warehouse; per-bird fields never enter any aggregate event or ML path. Enforced by separate datastores + DB role grants (analytics role has no access to simulation tables), not by policy alone.
- **Visitor isolation**: the visit path is render-only and has no event-write capability, so visitor attention can never drift the host's birds.
- **Deletion**: soft for 30 days (sign-in + "I changed my mind" restores), then hard — birds, vectors, notebook, telemetry tied to the account, all gone.
- **Export**: on-demand JSON (birds, names, current vectors, moods, notebook, settings) emailed to the verified address as a download link. (Note: export is the *only* place vector values leave the system, and they go to the owning user, not to any UI surface — this preserves "never exposed numerically" in-product while honoring "the relationship is theirs.")

---

## 12. Rollout

1. **Foundation**: accounts/auth (magic-link, synthetic UUID, sessions), schema with `sim`-only write grants on personality columns, event-log ingestion, the tick skeleton (mood + day/night, no drift yet). Validates the architecture before the engine.
2. **Engine**: personality vector + drift low-pass + mood FSM + bird-to-bird; calibration harness with the 1-week/3-week assertions. Gate engine work behind the calibration tests passing.
3. **Render + audio**: scene, mood-shaped idle, interpolation, loads-already-in-motion; procedural calls, chorus, listen-in ramp, silence/caption fallback.
4. **Interactions**: return-greeting (absence-length + boldness + mood + real procedural variation — explicitly *not* a canned arrival clip), offers + cooldown, settle + 5s undo, notebook sparsity gate.
5. **Accessibility surfaces**: narration stream, captions, reduced-motion register, keyboard nav, contrast — shipped *with* the above, not after.
6. **Social**: invite/visit read-only path, visit log, revocation, expiry, default-off notifications.
7. **Hardening**: perf budgets (bundle, first-bird, 60fps, memory CI), synthetic fleet + RUM, error budgets, privacy-boundary audit.

- **Birds-per-aviary ramp**: start at 2; the "add a third bird" offer is driven by **aviary age** (config-driven pacing curve — a few months → third, ~a year → five/six), never visit count/score/paid tier. The pacing curve is config so it can be tuned without code changes; cap hard-coded at 7 in the engine.
- **Instrument from day one**: tick latency, first-bird-render, frame timing, audio errors, bundle size — all aggregate-only.

---

## 13. Risks

- **Drift miscalibration** (the central risk). Too fast → Tamagotchi-by-clicking; too slow → screensaver. *Mitigation*: the calibration harness with explicit 1-week-measurable / 3-week-visible / no-single-session-visible assertions; `alpha`/weights in versioned config; ship behind those tests; monitor *aggregate* drift-rate sanity (without per-account data) as a population guardrail.
- **Presence dishonesty inflating drift** across the population (the "tab open all night" failure — silent, no test catches it by default). *Mitigation*: strict three-signal conjunction client-side; server-side per-window dwell cap; an explicit test that "tab open, unfocused, no activity" yields zero drift.
- **Sync correctness / personality loss** (worst failure: deleting the bird the user knows). *Mitigation*: server-only vector writes enforced by DB grants; additive ordered deltas (no LWW); event-log replay-safety (idempotent cursor); never recompute vector from log at runtime; backups of canonical state; identity continuity guard (stable `bird_id`, never reassigned on rename/sync/migration).
- **Audio uncanniness** (looped/canned audio breaks the spell irrecoverably). *Mitigation*: procedural-only rule enforced (no sample assets in the audio pipeline — a build check); chorus via independent voices (no stacked loops); per-bird recognizability test (signature stable across mood + simulated vector drift); listen-in slow ramp (no hard cut).
- **Accessibility regression to a stripped fallback** (telling reduced-motion/SR users the product isn't for them). *Mitigation*: narration from canonical state (not ARIA automation) with a review gate forbidding state-list narration; reduced-motion as a designed register with its own visual QA; a11y ships in the same release, with the same naturalist-voice review as the notebook.
- **"Notice never announce" leakage** (a "harmless" welcome toast / streak / notification slips in). *Mitigation*: the standing scope guards in §1 as PR-review gates; the voice discriminator on copy payloads; a periodic surface audit for any user-behavior-reporting UI.
- **Perf budget erosion over time** (bundle creep, slow memory leak). *Mitigation*: bundle-size + memory-growth as failing CI gates, not advisory; synthetic fleet + p99 tick-latency alarm; first-bird-render RUM watch.
- **Notebook drift toward feed/log** (entries become frequent or report user behavior). *Mitigation*: sparsity gate tuned + tested for active users; generator content rule forbidding user-behavior observations; naturalist-prose review.

---

## 14. Open calibration items (all routed through versioned config, none blocking architecture)

Tick cadence (~1 min); presence activity window (~3–5 min, lean long); offer cooldown (~3 min); drift `alpha` + per-input weights (against the 1wk/3wk targets); narration idle cadence (30–60s); notebook sparsity threshold + cooldown; personality seed values + ranges; add-a-bird age-pacing curve. Each has a defensible default above and a test or harness that makes re-tuning a config change rather than a code change.
