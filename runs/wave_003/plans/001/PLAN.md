# Pocket Aviary — Implementation Plan (v1)

This plan interprets the PRD into an executable build. It is organized so a
separate engineering team can pick up any section and implement it without
further clarification. Where the PRD leaves an implementation detail open,
this plan makes a defensible call and flags it as **[DECISION]**.

---

## 1. Scope

### 1.1 In scope for v1

- Web-only product, modern browsers (last two major versions of Chrome,
  Safari, Firefox, Edge).
- Single-user accounts, email magic-link auth, per-device revocable sessions.
- One canonical aviary per account, 2 starter birds, cap of 7, additional
  birds offered by aviary age (not activity, not payment).
- Server-side simulation tick (~1/min) as the only writer of canonical state.
- Personality vector (5 traits), monotonic-toward-expressive drift, mood
  system, procedural call grammar, bird-to-bird interaction.
- Interactions: return-greeting, listen-in, offer (seed / song fragment /
  still pool, per-bird cooldown), settle (with 5s undo), presence accounting.
- Field notebook: auto-generated, read-only, sparse naturalist entries.
- Multi-device sync as a property of the architecture (no client-side state
  to merge).
- Visit invitations: read-only ambient view, per-invite opt-in, revocable,
  30-day expiration, visit log, opt-in visit notifications (default off).
- Accessibility: screen-reader narration (naturalist prose), reduced-motion
  mode (designed surface, cross-fade rendering), call captioning, keyboard
  navigation, WCAG AA contrast on all chrome text.
- Performance budgets: <2MB gzipped initial JS, <500ms time-to-first-bird,
  60fps idle on 5-year-old laptop, no memory growth over 30 min (CI-tested).
- Account export (JSON, emailed download link) and 30-day-soft-then-hard
  account deletion.

### 1.2 Out of scope (respected non-goals)

- No native apps (v1 is web-only; data model and protocols are not designed
  for native-client constraints).
- No gamification of any kind: no streaks, achievements, levels, scores,
  badges, counters, calendars, XP, tiers — not even as opt-in toggles. No
  surface anywhere reports the user's own visit frequency to them.
- No Tamagotchi mechanics: birds never die, hunger, distress, or decay.
  Neglect produces ambient quietness only; drift never moves down.
- No social-network surfaces: no profiles, follows, feeds, discovery,
  leaderboards, comments, mutual visits, friend-of-friend chains. The single
  social feature is the read-only visit invite.
- No push notifications of any kind. No "welcome back" toasts, banners, or
  modals anywhere. The bird greeting is the entire welcome surface.
- No payments, shared aviaries, customizable scenes, multi-aviary accounts,
  species rarity, drag-to-place perches, editable notebook.

### 1.3 Hard rules carried into every subsystem

1. The user never sees personality vector values — no stats panel, debug
   view, export is exempt (export is the user's own data; **[DECISION]**:
   export includes vectors per the PRD's explicit list, but no in-product UI
   ever displays them).
2. Drift is monotonic toward expressive; neglect never moves a trait down.
3. The server is the only writer of personality state. Clients write events
   only.
4. Email is stored once, encrypted, on the account record. Every other
   reference — DB keys, logs, telemetry, shards — uses a synthetic UUID.
5. Per-bird interaction data never enters aggregate telemetry or any
   analytics pipeline. This is enforced at the pipeline level, not by policy.
6. Voice split: naturalist (lowercase, present-tense, specific) for the
   aviary, notebook, narration, captions, offer prompts; matter-of-fact for
   sign-in, account, sync errors, accessibility settings.

---

## 2. Architecture

### 2.1 Service shape

Three deployable units plus static hosting:

1. **Edge/static tier** — HTML shell + initial JS bundle from a CDN. The
   HTML response embeds a small inline bootstrap that can render the "quiet
   field" loading state immediately and kicks off the first snapshot fetch.
2. **API service** (stateless, horizontally scaled) — auth, session
   management, snapshot reads, interaction-event ingestion, notebook reads,
   account settings, export/deletion, visit invites. Talks to the primary
   datastore and publishes ingested events to the event log.
3. **Simulation service** (the tick engine) — consumes the interaction-event
   log, advances canonical aviary state for every active account on the ~1
   minute cadence, writes snapshots and notebook entries. Single-writer per
   account by construction (see §5).

### 2.2 Client/server split

- **Server owns**: accounts, sessions, personality vectors, moods, canonical
  positions/perches, call-schedule state, notebook entries, visit invites,
  the interaction-event log. The server is the only writer of all aviary
  state.
- **Client owns**: rendering, interpolation between snapshots, WebAudio
  synthesis of calls from motif parameters shipped in the snapshot, presence
  detection, idle ornament generation (leaves/feathers — pure client-side,
  no server state), top-bar chrome, accessibility surfaces. The client never
  computes drift, never mutates personality, never derives state from event
  history.
- **Render pipeline boundary**: the server sends *semantic* state (perch
  zone, mood, current action, call schedule, personality-influenced
  parameters expressed as non-numeric hints like palette saturation level
  and call-timing parameters). The client turns semantic state into pixels
  and audio. Personality trait values themselves are never serialized to the
  client — only their *renderable consequences* (perch preference, call
  timing, plumage saturation level as a bucketed integer, greeting style
  class). **[DECISION]** We bucket rather than send raw floats so the vector
  stays genuinely hidden even from DevTools inspection.

### 2.3 Datastores

- **Primary store**: relational (Postgres-class) for accounts, sessions,
  invites, notebook entries; JSONB column or adjacent table for per-bird
  canonical state (vector, mood, action, position). One row per account's
  aviary state document, versioned.
- **Event log**: append-only, ordered per account (Kafka-class or a Postgres
  append-only table with per-account sequence numbers at v1 scale —
  **[DECISION]** start with Postgres append-only + per-account sequence;
  the abstraction boundary allows swapping to a log system later without
  touching the tick).
- **Snapshot cache**: small (kilobytes) per-account snapshot served from the
  API tier; cache-aside with the simulation service as the writer.
- **Telemetry pipeline**: physically separate from the simulation store.
  Different credentials, different network path, schema allowlist at
  ingestion that contains no per-bird/per-account interaction fields. The
  privacy boundary is a build artifact, not a review checklist.

---

## 3. Data model

All IDs are UUIDs. `account_id` below is always the synthetic UUID, never
email.

### 3.1 Account

```
account {
  id: uuid (synthetic, generated at creation)
  email_ciphertext: encrypted string   // stored exactly once, here
  created_at, deleted_at (nullable), deletion_hard_at (nullable)
  settings: {
    visit_notifications_enabled: bool = false,
    captions_enabled: bool,
    reduced_motion_override: null | "on" | "off",  // null = follow OS
    audio_enabled: bool
  }
}
session { id, account_id, device_label, created_at, revoked_at }
```

### 3.2 Aviary & birds

```
aviary {
  account_id (1:1), created_at,           // age drives new-bird offers
  state_version: bigint,                  // incremented by tick
  last_tick_at, weather_state, lighting_phase
}
bird {
  id: uuid,                               // stable internal identity — never
                                          // replaced by rename/species change
  aviary_id, species_id, name,            // name user-renameable freely
  personality_vector: {                   // SERVER ONLY — never serialized
    boldness, social_warmth,              // to clients in raw form
    vocal_frequency, plumage_saturation,
    curiosity
  },                                      // each normalized [0,1]
  mood: enum(wary, content, curious, drowsy, alert),  // set finalized in build
  mood_entered_at,
  action: { kind, started_at, params },   // e.g. preen, scan, perch-move
  perch_zone: enum(front, middle, back),
  offer_cooldown_until
}
species { id, silhouette_asset_ref, palette, motif_library_id, nightjar_like: bool }
```

### 3.3 Interaction events (append-only)

```
event {
  account_id, seq: bigint (per-account monotonic), client_event_id (idempotency),
  kind: presence_ping | listen_in_start | listen_in_end | offer |
        settle | settle_undo | session_open | session_close,
  payload: { bird_id?, offer_kind?, duration_ms?, ... },
  client_ts, server_ts
}
```

Presence pings are the raw material; the tick folds them into presence-time
per §7. Visitors write **no** events — the visit endpoint rejects event
ingestion for visit tokens.

### 3.4 Notebook entries

```
notebook_entry {
  id, account_id, created_at (day-granularity display),
  body: text (naturalist prose, pre-rendered server-side)
}
```

Entries are generated by the simulation service (§5.5), stored read-only,
never edited or deleted except by account deletion.

### 3.5 Visits

```
visit_invite {
  id, host_account_id, visitor_email_ciphertext,
  token_hash, created_at, expires_at (30d), revoked_at, first_used_at
}
visit_log_entry { id, host_account_id, invite_id, started_at, approx_duration_s }
```

Visitor identity is never represented in aviary rendering.

---

## 4. API surface

REST-ish JSON over HTTPS; WebSocket is explicitly **not** used at v1 —
snapshots are kilobytes and the tick cadence is a minute, so polling on
well-chosen triggers is simpler and cheaper. **[DECISION]** All endpoints
versioned under `/v1/`.

### 4.1 Auth

- `POST /v1/auth/magic-link` `{email}` — issues 15-minute, single-use link;
  per-email rate limited.
- `GET /v1/auth/consume?token=...` — validates, invalidates the token
  immediately, issues a per-device session cookie + CSRF token.
- `POST /v1/auth/sign-out`, `GET /v1/sessions`, `DELETE /v1/sessions/{id}`
  (revoke).
- `POST /v1/account/email-change` — verify-new-before-commit; old email
  works until verification.

### 4.2 State pull (how clients read)

- `GET /v1/aviary/snapshot?since_version=N` →
  `{ state_version, server_time, lighting_phase, weather,
     birds: [{ id, name, species_id, perch_zone, mood, action,
               plumage_level (bucketed), call_timing_params, ... }] }`
  Delta form when `since_version` is current-ish; full snapshot otherwise.
  Response is a few KB. Also embeddable in the initial HTML payload for
  first-paint speed (§9).
- Clients pull on: initial load, `visibilitychange` to visible, render-frame
  gap detection (>30s), and a low-frequency keepalive (~2–5 min) while
  visible. **[DECISION]** No long-polling at v1; the interpolation layer
  hides the pull cadence.

### 4.3 Interaction events (how clients write)

- `POST /v1/events` — batched array of events with `client_event_id` for
  idempotent retry. Server assigns per-account `seq`. Validates session,
  rejects unknown kinds, never accepts any field that resembles absolute
  personality state. Rate limits generous but bounded.
- Semantics: `listen_in_start/end`, `offer {kind, bird_id?}`,
  `settle` / `settle_undo`, `presence_ping` (sent every ~30s while the
  three-signal presence conjunction holds, plus on conjunction loss).

### 4.4 Notebook

- `GET /v1/notebook?before=<cursor>&limit=50` — reverse-chronological,
  cursor-paginated, read-only. Entries older than any age remain reachable.

### 4.5 Visit flow

- `POST /v1/visits/invites` `{visitor_email}` — creates invite, emails a
  one-time-link-style URL (link usable repeatedly until revoked/expired —
  **[DECISION]** the PRD says "one-time link" for the email but describes a
  revocable ambient session; we implement: the emailed URL carries a token
  that mints a short-lived read-only visit session on each visit, renewable
  while the invite is live. "One-time" is honored in that the token cannot
  be forwarded to mint host sessions and grants no account).
- `GET /v1/visits/invites`, `DELETE /v1/visits/invites/{id}` (revoke —
  effective at the visitor's next snapshot pull).
- `GET /v1/visit/{token}/snapshot` — same snapshot shape minus anything
  identifying the host beyond the aviary itself; served only while invite is
  live; each response carries `visit_status: active | revoked | expired` so
  revocation surfaces immediately.
- `GET /v1/visits/log` — host's visit log (visitor email, date, approx
  duration, outstanding invites).

### 4.6 Account lifecycle

- `POST /v1/account/export` — queues export, emails download link to the
  verified address (birds, names, vectors, moods, notebook, settings).
- `POST /v1/account/delete` — soft-delete; any sign-in within 30 days
  surfaces the "I changed my mind" restore. A nightly job hard-deletes after
  the window: birds, vectors, notebook, events, telemetry joins — every
  record keyed by the account UUID.

---

## 5. Simulation engine design

### 5.1 Tick structure

Cadence: ~60s per tick, jittered ±5s per account to avoid thundering herd.
**[DECISION]** Accounts are sharded across tick workers by `account_id`;
each account's state is advanced by exactly one worker at a time (advisory
lock or single-partition assignment), which gives the single-writer
guarantee without a global lock.

Each tick for an account:

1. Load aviary state + all events with `seq > last_consumed_seq`, in order.
2. Fold `presence_ping` events into presence-time for the tick window
   (gaps > the calibrated activity window are not counted).
3. Apply **mood transitions** (§5.3).
4. Apply **drift deltas** (§5.2) — additive only, clamped to [0,1],
   monotonic (deltas are never negative; see §5.2).
5. Advance per-bird action/position state machines (perch moves, preen
   bouts, scan events) using personality + mood + time-of-day + weather.
6. Schedule calls: per-bird next-call times shaped by vocal_frequency, mood,
   weather, chorus emergence rules (§5.4).
7. Possibly generate a notebook entry (§5.5).
8. Possibly emit a new-bird offer if aviary age crossed a threshold.
9. Write new state, bump `state_version`, write snapshot cache, mark events
   consumed.

Tick work per account is small (a handful of birds); p99 tick latency budget
is well under the 5s alarm.

### 5.2 Drift function

Per trait, per tick:

```
delta_t = gain_t * lowpass(signal_t)      // gain_t per-trait, small
trait  = clamp01(trait + max(0, delta_t)) // monotonic: floor at 0 delta
```

- `signal_t` aggregates, weighted: presence-time (dominant), listen-in
  minutes (→ social_warmth, vocal_frequency of the focused bird), accepted
  offers (→ curiosity), offers near a bird (→ boldness).
- The low-pass filter is a decaying accumulator (e.g. EMA with a multi-day
  time constant) so no single session moves a trait visibly.
- **Calibration target** (testable, per PRD): a synthetic "regular visitor"
  harness profile (daily ~10-min presence sessions) must produce
  instrument-measurable drift within 7 simulated days and
  perceptible-to-user drift (a perceptibility rubric: perch-zone
  distribution shift, greeting-first probability shift, plumage bucket
  change) within 21 simulated days. A "two weeks absent" profile must
  produce zero negative drift and only decay-toward-ambient behavior in
  greeting frequency. These run as a fast-forward simulation test in CI
  (tick clock injected).
- Neglect handling: with no inputs, deltas are exactly zero — the *rendered
  expression* of traits (e.g. greeting probability) is additionally modulated
  by a fast-decaying "recently attended" factor that relaxes toward ambient
  over days, which is how birds read as "quieter, not warier." The stored
  vector never moves down. **[DECISION]** This two-layer expression
  (permanent vector + transient expressiveness factor) is how we honor both
  "monotonic drift" and "quieter after absence" without contradiction.

### 5.3 Mood transitions

Mood is a small enum (`wary, content, curious, drowsy, alert` — final set
locked during build with the design team). Transitions are computed per tick
from a scoring function:

```
score(m) = base_prior(m, local_time_of_day)
         + personality_bias(m, vector)     // high boldness ↓ wary entry
         + recent_events(m, last-hours events)
         + ambient(m, weather, other birds' moods)   // wary spreads; rain
                                                      // dampens vocal freq
```

Highest score wins with hysteresis (must exceed current mood's score by a
margin) to prevent flicker. Mood persists in stored state across sessions —
there is no reset-to-neutral anywhere. Daily-ish reset emerges from the
time-of-day prior rather than a hard reset.

### 5.4 Call-grammar runtime

- Each species has a **motif library**: parameterized note primitives
  (pitch contour, duration, timbre params, trill structure).
- A bird's **call signature** = a fixed-per-bird selection and parameter
  seed drawn at adoption from the species library, so Pip is always
  recognizably Pip.
- At runtime the server schedules *when* calls happen (from vocal_frequency,
  mood, weather, chorus rules, greeting logic); the snapshot carries
  `{ bird_id, scheduled_at, motif_id, variation_seed, gain }`.
- The client synthesizes audio from motif params + variation seed via
  WebAudio (§8). Variation seeds guarantee no two renders of the "same" call
  are identical while the signature stays recognizable. The same seed also
  generates the caption text, so caption matches audio exactly.
- **Chorus**: when two+ birds' scheduled calls overlap within a window,
  timing offsets are nudged so the mix reads as a chorus; never stacked
  identical loops.
- **Bird-to-bird**: a call event can schedule a response call on another
  bird with probability weighted by that bird's social_warmth; an alarm-call
  action shifts nearby birds' mood scores toward wary next tick.

### 5.5 Notebook generation

The tick evaluates a set of *noteworthiness detectors* (first-greeter
changed today, unusual quiet stretch, first accepted offer of a kind,
weather reactions, perch-habit changes over weeks). Each detector emits a
candidate observation; a sparsity gate (~one entry per few days for a
regularly visited aviary; hard minimum spacing, priority to rarer event
classes) selects at most one. Selected observations are rendered into
naturalist prose by a template-and-slots prose engine with a curated phrase
bank (lowercase, present-tense, bird-named, no exclamations, no
"you"-sentences, never about the user's behavior). **Hard rule enforced in
the detector layer**: no detector may reference visit frequency or streaks;
code review checklist item.

### 5.6 Return-greeting selection

On `session_open`, the tick (or an on-demand greeting computation using the
same functions — **[DECISION]** computed on-demand at first snapshot after
open, since greetings need sub-tick latency) picks **one** greeter:
weighted by boldness, mood (drowsy birds rarely greet), absence length
buckets (<1h glance; 1–24h call + look; >24h re-orientation: approach +
longer call). If multiple birds qualify, the bolder fires and others may
respond, staggered by randomized 0.3–1.2s offsets — never in unison. The
greeting is expressed as action + call instructions in the snapshot; the
client renders it procedurally with variation seeds. No canned animations,
no textual welcome, ever.

---

## 6. Sync model

- **One canonical record.** All state lives server-side; every client is a
  renderer + event emitter. Multi-device sync requires no merge logic
  because there is nothing to merge.
- **Additive deltas only.** Drift applies `trait += f(events)` computed
  from the ordered per-account event log. No code path accepts absolute
  personality values from clients; the event API schema makes it
  unrepresentable. A phone session concurrent with a laptop session simply
  interleaves events in one ordered log — both sessions' attention counts,
  neither overwrites the other. This makes the last-write-wins failure mode
  structurally unreachable.
- **Ordering**: per-account `seq` assigned at ingestion; the tick consumes
  strictly in order; `client_event_id` dedupes retries.
- **Conflict surface**: genuine conflicts are limited to auth/session edges
  (replay, timeout, outage). These use matter-of-fact copy per the PRD
  examples. There is no aviary-state conflict UI because aviary state cannot
  conflict.
- **Visitor isolation**: visit tokens authenticate to a read-only scope that
  cannot reach the event endpoint; visitor presence generates no events and
  no drift.

---

## 7. Presence accounting (client)

- The client evaluates the three-signal conjunction continuously:
  `document.visibilityState === "visible"` AND `document.hasFocus()` AND
  (pointermove or keypress within the last T minutes). **[DECISION]**
  T = 4 minutes initially, biased long because motionless watching is the
  product; calibrated against session recordings in beta.
- While the conjunction holds, the client sends `presence_ping` every 30s.
  On conjunction loss (any signal), it sends a final ping marked
  `window_end`. Both settle and tab-close end presence identically at the
  engine; `settle` additionally carries the gesture so the tick can apply
  the small mood-quieting signal and the client renders the lighting shift.
- Battery: when hidden, the client stops rendering entirely (RAF cancelled,
  audio context suspended after fade), but sends nothing extra — the server
  keeps ticking regardless.
- Anti-inflation tests: headless-browser CI scenarios (tab open backgrounded
  overnight; focused-but-unattended; visible-unfocused window) assert zero
  presence-time recorded in each.

---

## 8. Audio pipeline

- **Synthesis**: WebAudio graph per calling bird: oscillator/NoiseNode
  sources shaped by motif parameters (pitch envelopes, AM/FM for trills,
  formant-ish filtering) → per-bird gain node → master bus with a soft
  ambient reverb (ConvolverNode with a tiny generated impulse response, no
  audio asset).
- **Chorus mixing**: all birds feed the master bus at ambient levels; the
  mix engine applies the listen-in state.
- **Listen-in mix**: engaging listen-in ramps the focused bird's gain up and
  others down over ~1.5–2.5s (and reverse on disengage). Other birds floor
  at an ambient minimum — never silent. Disengage triggers: re-click the
  focused bird, focus another bird, click empty scene, move keyboard focus
  away.
- **Scheduling**: the snapshot's call schedule drives an
  `AudioContext`-clock scheduler with lookahead (~200ms); variation seeds
  perturbe pitch/timing within the signature's tolerance band so no call
  repeats exactly.
- **Captions**: when enabled (or in fallback mode), each synthesized call
  emits a caption generated from the same motif+seed — a template over
  motif structure ("a soft three-note rise", "a low trill, paused, low
  trill again") — rendered near the calling bird, fading with the call.
- **Fallback**: if WebAudio is unavailable/denied — graceful silence,
  captions default ON. No recorded-audio fallback path exists anywhere.
- **Memory**: all buffers and impulse responses allocated once at init;
  per-call nodes are created and garbage-collected but never accumulate
  buffers; verified by the 30-minute no-growth CI test.

---

## 9. Frontend rendering pipeline

- **Stack**: canvas-based scene renderer (WebGL with 2D fallback —
  **[DECISION]** WebGL via a thin custom sprite/SDF layer; birds are
  procedurally generated vector-ish sprite parts with bone-less procedural
  micro-motion, plumage saturation applied as a shader-level palette
  parameter). No heavy framework for the scene; a small framework (or
  vanilla) for chrome. Aggressive code-splitting: account settings,
  accessibility settings, visit flow, and notebook UI load on demand.
- **First paint**: the HTML payload inlines the bootstrap + the initial
  snapshot (edge-joined at the CDN where possible). The renderer's first
  frame places birds at snapshot positions *mid-action* — the action state
  includes phase offsets so preens and scans resume mid-cycle. There is no
  entry animation, no fade-from-static, no spinner. The loading state
  (snapshot delayed) is the quiet field: soft sky color, one or two faint
  motion cues.
- **Empty aviary**: same quiet field; the first bird enters with a single
  soft fly-in to its starting perch; never shown again after adoption.
- **Interpolation**: between snapshots, positions and action phases lerp;
  perch changes render as short flight paths. Motion never teleports.
- **Idle micro-motion**: procedural — preen cycles, scans, head-tilts,
  weight-shuffles — parameterized by mood (wary: back perch + scan-heavy;
  content: preen-heavy; curious: tilt-toward-sound; drowsy: low + fluffed).
  Motion *is* the mood readout; there are no mood labels or icons.
- **Ambient ornaments**: leaves/feathers drift client-side at random slow
  intervals; subtle two-plane parallax. Zero server state.
- **Day/night**: lighting phase from the user's local time (computed
  client-side, cross-checked against server phase) warms/cools the palette
  gradually; at night most birds render settled; the nightjar-like species
  stays active.
- **Weather**: rare (few times a week) rain/wind events from the snapshot;
  short-lived palette/audio effects only.
- **Top bar**: sparse (account, accessibility, notebook, offer), fades to
  near-transparent after a few seconds of cursor stillness, returns on
  movement/keyboard. No chrome inside the scene. No hover tooltips, badges,
  or inline labels.
- **Responsive**: scene scales to viewport; all birds always in frame at any
  aspect; narrow viewports compress perch spacing, never crop.
- **Reduced-motion mode**: a parallel render path, not a stripped one.
  Micro-motion becomes slow cross-fades between still poses; flights become
  cross-fades between perches; leaf drift removed; day/night color shifts
  remain, slowed. Audio, drift, notebook, mood all unchanged. Triggered by
  `prefers-reduced-motion` or the accessibility setting override.

---

## 10. Accessibility surfaces

- **Narration**: an `aria-live="polite"` region fed by a narration
  generator reading the same snapshot state as the renderer, producing
  naturalist prose ("a small grey bird is perched on the front rail,
  calling softly."). Cadence: one update per 30–60s at idle; user-initiated
  events (return-greeting, offer reaction, settle) get a priority bump but
  are still written as observations. A rate limiter guarantees the queue
  never floods. Same voice as the notebook — the two surfaces are
  generated by the same prose engine where possible.
- **Keyboard**: Tab through top bar → into scene → first bird focused;
  arrows move between birds; Enter = listen-in; Escape = disengage; offer
  palette and settle reachable from the top bar; all dialogs keyboard-trapped
  and escapable. Focus indicator: soft high-contrast outline legible against
  bright and dim aviary states (design system specifies treatment).
- **Contrast**: all chrome/copy (top bar, settings, errors, captions,
  visual narration) meets WCAG AA minimum; checked in CI via automated
  contrast audit on rendered states including night palette.
- **Audio-off / captions**: per §8.
- **Ship gate**: accessibility surfaces ship *with* v1, not after; the
  reduced-motion render path and narration generator are in the critical
  path of the rendering/epic schedule, not a follow-up.

---

## 11. Performance budgets & observability

| Budget | Value | Enforcement |
|---|---|---|
| Initial JS bundle | <2MB gzipped at first paint | CI bundle-size check, hard fail |
| Time to first bird | <500ms on mid-tier mobile / 4G | Synthetic fleet + RUM p50/p90 |
| Idle motion | 60fps on 5-year-old mid-range laptop | Synthetic long-session runs |
| Memory | No growth over 30-min session | CI headless heap-delta test, hard fail |
| Tick latency | p99 < 5s alarm threshold | Server metrics + paging alarm |

- **What we measure**: synthetic checks from common geographies; aggregate
  RUM (load timings, first-bird timings, frame timings, audio-context
  errors); server metrics (tick latency, event-ingest rate, snapshot
  latency, error rates); anonymized session-duration histograms.
- **What we deliberately don't measure**: anything per-bird or per-account
  interaction; no per-account dimension in any aggregate; no analytics
  warehouse reads of the simulation store; no "average drift across
  accounts" dashboards — not computed, so they can't leak. Metric schemas
  are allowlisted in code; adding a field touching account or bird state
  fails review-by-lint.
- **Unsupported browsers**: matter-of-fact surface, no compatibility shims.

---

## 12. Rollout

1. **Phase 0 — foundations**: auth, account model with synthetic-UUID rule,
   event log, tick skeleton with trivial state machine, snapshot API,
   quiet-field client shell. Privacy boundary (telemetry allowlist) landed
   before any interaction data exists.
2. **Phase 1 — the living aviary**: bird engine (vector, mood, drift with
   calibration harness), procedural motion, procedural audio, day/night,
   return-greeting. Internal dogfood behind a flag.
3. **Phase 2 — the relationship**: presence accounting, listen-in, offers,
   settle, notebook prose engine, drift calibration locked against the
   7-day/21-day targets using fast-forward simulation.
4. **Phase 3 — access & polish**: narration, reduced-motion path, captions,
   keyboard nav, contrast audit, perf hardening to budgets.
5. **Phase 4 — the quiet edges**: visit invites, export, deletion, sync
   conflict surfaces, unsupported-browser surface.
6. **Launch**: invite-only beta (a few hundred accounts) → drift calibration
   re-checked against real (aggregate-free, per-account-internal-only)
   instrument readings → general availability.

- **Birds-per-aviary ramp**: 2 at adoption for everyone. New-bird offers
  unlock purely on aviary age (third bird ~3 months; up to ~5–6 by a year;
  hard cap 7). Offer pacing constants are server-configurable without
  deploy. The cap is enforced in the engine, not just the UI.
- **Instrumented from day one**: tick latency, first-bird timing, bundle
  size, audio errors, presence-signal sanity counters (aggregate only),
  notebook sparsity rate. Also a **drift-calibration dashboard over our own
  synthetic test accounts** (never real accounts) so calibration drift is
  visible without touching user data.

---

## 13. Risks

| Risk | Failure mode | Mitigation |
|---|---|---|
| **Drift calibration** | Too fast → Tamagotchi feel; too slow → screensaver feel. Silent — no unit test catches "feels wrong." | Named calibration targets (7d instrument / 21d visible); fast-forward simulation harness in CI; synthetic-account dashboards; beta re-calibration before GA; per-trait gains are config, not constants. |
| **Presence inflation** | Lax presence detection silently accelerates drift population-wide; invisible failure. | Three-signal conjunction implemented once, shared by all surfaces; headless anti-inflation CI scenarios; bias the activity window long. |
| **Sync correctness** | Any client-side personality write reintroduces last-write-wins and silently deletes drift. | Event API schema cannot express absolute state; server-only writer enforced by code ownership + review; additive deltas; per-account ordered log; CI chaos test with interleaved concurrent sessions asserting no drift loss. |
| **Audio uncanniness** | Repeated-sounding calls, phase-y chorus, or canned feel breaks the entire product. | Procedural synthesis only; variation seeds with recognizability tolerance bands; chorus scheduling with timing nudges; listening-panel reviews during beta; the fallback is silence+captions, never loops. |
| **Accessibility regressions** | Narration floods SR queues; reduced-motion ships as "animations off"; a11y deferred to v1.1. | Narration rate limiter with tests; reduced-motion as a parallel designed render path reviewed by reduced-motion users in beta; a11y epics in the critical path; contrast audit in CI including night palette. |
| **Personality-vector loss/exposure** | Losing a vector deletes a bird the user knows; exposing values collapses the relationship into stats. | Vectors persisted server-side only, included in backups and export; bucketed-only consequences serialized to clients; schema lint forbids vector fields in client DTOs. |
| **Voice drift in copy** | A single toast, streak, or "welcome back" re-frames the product. | Copy review checklist naming the two voices; lint-ish rule: no user-facing strings containing "welcome back", "streak", counts of visits; the announcement ban restated in the repo's copy guidelines. |
| **Notebook voice collapse** | Entries degrade into event logs ("session started 7:43") and break the spell. | Curated phrase bank + detector whitelist; sparsity gate with tests; human prose review of generated samples each release. |
| **Tick cost at scale** | Per-minute ticks over many accounts get expensive. | Jittered sharding, tiny per-account work, stateless workers, snapshot cache; load-tested at 100× launch target. |
| **Magic-link abuse / replay** | Account takeover via forwarded links. | 15-min expiry, single-use consumption, per-device sessions, session list + revoke, rate limiting. |
| **Scope creep toward social/gamification** | The predictable "just one harmless counter" pitch. | Non-goals restated in engineering docs; no underlying cross-account metrics exist to expose; architectural absence makes reappearance a build, not a toggle. |

---

## Appendix A — Key decisions deferred to build (with defaults)

- Tick cadence: 60s ±5s jitter (calibrate).
- Presence activity window: 4 minutes (calibrate long).
- Mood enum final set: wary, content, curious, drowsy, alert.
- Event log: Postgres append-only with per-account seq (swap-ready boundary).
- Scene renderer: WebGL thin custom layer; 2D canvas fallback.
- Visit token model: revocable token minting short-lived read-only sessions.
- New-bird age thresholds: ~3mo third bird, ~5–6 by one year (config).
