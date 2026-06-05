# Pocket Aviary — v1 Implementation Plan

This plan interprets the PRD into an executable design for a small engineering team. The deliverable is the plan; the product itself is not built in phase 1. Where the PRD is silent, a defensible call is made and flagged.

---

## 1. Scope

### In scope (v1)

- Web-only single-page application served from a CDN edge.
- Single-user accounts authenticated via email magic link; one account, one aviary, one user.
- Two starter birds at adoption, with a species-pool-driven ramp up to a hard cap of seven.
- Server-authoritative simulation tick that runs at a slow cadence (~once per minute, calibrated in build).
- Personality vectors (boldness, social warmth, vocal frequency, plumage saturation, curiosity) drifting monotonically toward expressive on positive presence; never drifting down on neglect.
- Per-bird mood (wary, content, curious, drowsy, alert — finalized in implementation) with mood-shaped idle motion and time-of-day / ambient-event modulation.
- Procedural call grammar, synthesized client-side via WebAudio; per-bird call signatures stable enough to recognize across mood and drift.
- Interactions: sit/watch (passive presence), listen-in, offer (seed / song fragment / still pool), settle. Return-greeting procedural variation.
- Field notebook — read-only, naturalist prose, ~one entry every few days for an active aviary.
- Multi-device sync via the server-canonical-state model; no client-to-client sync, no last-write-wins on personality.
- Optional visit invitations (off by default, per-invite opt-in, revocable, read-only ambient view).
- Accessibility: screen-reader narration, reduced-motion mode as a designed surface, call captioning, keyboard navigation, WCAG AA contrast on all user copy.
- Performance: initial JS <2MB gzipped, time-to-first-bird <500ms on mid-tier mobile / 4G, 60fps idle on a 5-year-old laptop, no client memory growth over 30 minutes.
- Aggregate-only RUM and synthetic perf checks; no per-bird telemetry in any pipeline.
- Soft-then-hard account deletion (30 day window), per-account JSON export.

### Out of scope (v1) — explicit refusals

These are non-negotiable absent in v1:

- Native iOS or Android app.
- Gamification of any kind: no achievements, streaks, levels, scores, badges, counters, calendars of green dots, XP, tiers.
- Tamagotchi-style decay: no hunger, no health, no visible distress, no negative drift on neglect, no death.
- Social-network surfaces: no profiles, follows, public feed, discovery, leaderboards, comments, chat, avatars in-aviary, or mutual visits.
- Push / email / in-product notifications about the aviary (a per-account, off-by-default toggle for visit notifications is the only exception, and it is hidden from onboarding).
- "Welcome back!" toasts, banners, or any textual welcome surface.
- Customizable scenes, multi-aviary accounts, shared aviaries, household profiles.
- Payments, billing, premium tiers.
- Recorded audio assets, canned greetings, or any audio surface that is not procedurally synthesized client-side.
- Public APIs for third parties; no per-account fields ever exported to analytics/ML/training.

The shape of v1 is what's left after those subtractions. Any feature pitch that lands adjacent to a refusal needs to be rejected at the design step, not the implementation step.

---

## 2. Architecture

### Service shape

Three services plus shared infra:

- **Web client** (SPA) — served from a CDN. Renders the scene, handles user input, synthesizes audio, posts interaction events, polls/pull state snapshots. Owns no canonical state.
- **Simulation service** — owns the aviary database (per-account), runs the tick loop, computes mood transitions, applies personality deltas, generates notebook entries, handles invite / visit tokens, and serves snapshot reads. This is the only writer to the per-account simulation tables.
- **Account / auth service** — magic-link issuance, session tokens, email verification, account settings, account export, soft-delete, session revocation, per-account privacy preferences (visit notification toggle, accessibility settings, reduced-motion preference). Reads `account_id` only; never queries the simulation database.

A small **static asset CDN** in front of the web bundle. A **transactional mail provider** for magic links, invites, and account-export download links. A **privacy-clean telemetry pipeline** (see §10) — strictly aggregate; never touches the per-account simulation database.

### Client / server split

- **Server is the only writer of personality state, mood, notebook entries, and visit tokens.** The client is read-only on those tables.
- **Client is the only writer of interaction events** (offer, listen-in start/end, settle, presence pings) — appended to an event log on the simulation service, consumed by the tick.
- **No client-to-client sync.** Two devices are both reading the same server record. There is no merge problem because there is no client state to merge.
- **No realtime push in v1.** Clients pull snapshots on a low-frequency keepalive and on visibility change. The simulation tick is server-side; latency from event to reflected-state-change is at most one tick.

### Render pipeline boundary

- The client has a thin **state cache** populated by snapshot pulls and event acknowledgements. It never persists state across reloads (the server is canonical).
- The renderer (a single canvas / WebGL surface plus a SVG-overlay for the top bar) is fed by the state cache via a reactive layer. Mood, perch position, and idle motion are interpolated between snapshots; procedural call events drive audio.
- The **narration generator** runs in the same process as the renderer and reads from the same state cache, so screen-reader output stays voice-continuous with the visual surface.
- The **audio graph** (WebAudio nodes) is created lazily on first user gesture; before that, calls are silent or caption-only.

---

## 3. Data model

The persistent state is held in a relational store. The shapes below are the columns the engine actually reads and writes — names in `snake_case` for SQL friendliness; the application uses camelCase.

### Account

| Field | Type | Notes |
|---|---|---|
| `account_id` | UUID | Synthetic. The only identifier used outside this row. |
| `email_encrypted` | bytea | Stored once, AES-GCM at rest with a key held by the account service. Never used as an identifier. |
| `email_verified` | bool | |
| `created_at` | timestamptz | |
| `soft_delete_at` | timestamptz, nullable | Set on user-initiated delete. Hard delete runs 30 days later. |
| `preferences` | jsonb | Reduced-motion, captions, visit-notification toggle, audio-on default. |

Sessions, magic-link tokens, email-change verifications, and export-download tokens live in side tables keyed by `account_id` only.

### Aviary

One row per account, created at signup. Stores the cross-bird state:

| Field | Type | Notes |
|---|---|---|
| `account_id` | UUID | PK, FK. |
| `created_at` | timestamptz | Drives the "new bird available by aviary age" pacing. |
| `local_timezone` | string | Best-effort guess from the browser at signup; user-correctable. Drives day/night. |
| `current_weather` | enum | none, light_rain, soft_wind. Short-lived server state. |
| `weather_until` | timestamptz | |
| `last_tick_at` | timestamptz | |

### Bird

One row per bird; up to 7 rows per aviary. The `bird_id` is a stable synthetic UUID generated at adoption and never reused or replaced — the identity rule from `bird_engine.md` is enforced by never having a code path that updates `bird_id` or replaces a row.

| Field | Type | Notes |
|---|---|---|
| `bird_id` | UUID | Stable identity. Never mutated. |
| `account_id` | UUID | FK. |
| `species_id` | string | From the small species pool (~6 species at v1). |
| `name` | string | User-assigned, default suggested, renameable at any time. |
| `personality` | jsonb | `{boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity}` — normalized scalars, server-only; never returned to client as raw numbers. |
| `mood` | enum | wary, content, curious, drowsy, alert (final set in implementation). |
| `mood_set_at` | timestamptz | |
| `perch_zone` | enum | front, middle, back. |
| `adopted_at` | timestamptz | |
| `call_signature_seed` | uint64 | Used to seed the procedural call grammar client-side. Same seed → same recognizable call across devices and mood states. |

### Interaction event log

Append-only. The only writer is the client; the only reader is the tick. Rows are not deleted by the client, but old rows are eligible for archival by a periodic job that keeps the last 30 days online and the rest in cold storage (used only for drift debugging, not for runtime).

| Field | Type | Notes |
|---|---|---|
| `event_id` | UUID | |
| `account_id` | UUID | Partition key. |
| `bird_id` | UUID, nullable | Null for aviary-level events (settle, presence ping). |
| `event_type` | enum | presence_ping, listen_in_start, listen_in_end, offer_made, offer_accepted, offer_ignored, settle, return_greeting_rendered. |
| `occurred_at` | timestamptz | |
| `payload` | jsonb | E.g. offer type, listen-in duration, presence-window length. |

`personality` is **never** in this log. Clients never write absolute personality values.

### Presence window

Aggregated by the tick from `presence_ping` events into a per-account daily rollup. The tick stores, per day:

| Field | Type | Notes |
|---|---|---|
| `account_id` | UUID | |
| `day` | date | In account's local timezone. |
| `presence_seconds` | int | Sum of qualified presence windows that day. |
| `listen_in_seconds` | jsonb | Per-bird listen-in totals. |
| `offers` | jsonb | Per-bird offer counts (made / accepted / ignored). |

This is what the drift function reads. The raw event log is also available for re-derivation if the rollup is ever wrong.

### Notebook

| Field | Type | Notes |
|---|---|---|
| `entry_id` | UUID | |
| `account_id` | UUID | Partition key. |
| `written_at` | timestamptz | |
| `prose` | string | Naturalist prose, lowercase, present-tense. Server-generated. |

Entries are not user-editable. The generator (§6) decides when an entry is warranted — roughly one per few days for an active aviary, more for noteworthy events, never per session.

### Visit / invite

| Field | Type | Notes |
|---|---|---|
| `invite_id` | UUID | |
| `host_account_id` | UUID | |
| `visitor_email_encrypted` | bytea | Encrypted at rest; surfaced as cleartext only in the host's visit log. |
| `token_hash` | bytea | One-time-use; bcrypt-hashed. |
| `created_at` | timestamptz | |
| `expires_at` | timestamptz | 30 days. |
| `revoked_at` | timestamptz, nullable | |
| `used_at` | timestamptz, nullable | |

Visit sessions are short-lived signed tokens; revocation takes effect on the next snapshot pull.

### Privacy boundary

- `personality`, `mood`, `notebook`, `interaction_event_log`, `visit`, and `bird` are **never** read by the analytics warehouse, never included in telemetry events, never logged with identifiers beyond `account_id` in error traces, and never part of any aggregate that could be re-per-account'd.
- Aggregate telemetry uses a separate ingestion path that only sees hashed counters and timing histograms.

---

## 4. API surface

All endpoints are over HTTPS, authenticated by short-lived session tokens issued by the account service. The simulation service trusts an `account_id` claim issued by the account service; it never re-validates emails.

### Snapshot read

`GET /v1/aviary/snapshot`

Returns: list of birds (id, name, species_id, mood, perch_zone, call_signature_seed — **never** personality numbers), current day/night phase, current weather, pending notebook entries, last tick timestamp, and any active server-side animations (settle in progress, etc.).

The client polls this:
- On tab visibility becoming `visible`.
- On a long render-frame gap (laptop wake).
- On a low-frequency keepalive (every ~15s when the tab is visible and idle; throttled to ~60s during active interaction so as not to flood).
- On a listen-in start, on an offer, on a settle (to pick up the state transition).

Snapshots are small — target <2KB JSON for a 2-bird aviary, scaling linearly to ~6KB at 7 birds.

### Interaction event write

`POST /v1/aviary/events`

Body: array of `{event_type, bird_id?, payload?, client_ts}`. Idempotent on `(client_event_id)`; the client generates a UUID per event and the server returns ack/no-op. The server tags `server_received_at` for ordering.

The tick consumes the log in arrival order. The client never writes personality deltas, mood, perch position, or notebook entries — those are server-originated.

### Magic-link auth

`POST /v1/auth/request-link` — accepts an email, returns ack. Rate-limited per email.

`GET /v1/auth/consume?token=…` — exchanges a valid, unexpired, unused token for a session cookie + bearer token.

`POST /v1/auth/revoke-session` — for the account-settings "sign out this device" surface.

### Account settings

`GET / PATCH /v1/account` — read/update display name, local timezone, preferences (reduced-motion, captions, visit-notification toggle).

`POST /v1/account/request-export` — kicks off an async export job; the download link is emailed to the verified address.

`POST /v1/account/request-delete` — sets `soft_delete_at = now()`. The user can sign in during the 30-day window to recover.

### Visit / invite flow

`POST /v1/visits/invites` — host provides visitor email; service generates a one-time token, stores its hash, emails a signed magic link to the visitor.

`GET /v1/visits/invites` — host's list of outstanding and past invites.

`DELETE /v1/visits/invites/{invite_id}` — host revokes; takes effect on visitor's next snapshot pull.

`POST /v1/visits/consume` — visitor follows the email link; exchanges a valid invite token for a short-lived read-only visit session.

`GET /v1/visits/aviary-snapshot` — visit-session-authenticated variant of the snapshot endpoint. Returns the same payload the host would see, plus a `visit_session_expires_at`.

`GET /v1/visits/log` — host's visit log.

### Error envelope

A single, predictable error envelope. Matter-of-fact copy (never naturalist):

```
{ "error": { "code": "string", "message": "string" } }
```

Status codes are HTTP-standard; messages are written for the user reading them in the surface where they appear, not for engineers grepping logs.

---

## 5. Simulation engine design

### Tick loop

A single, durable worker process per simulation-service instance (replicated for HA) runs the tick loop. Each tick:

1. Lock the set of accounts due for an update (those whose `last_tick_at` is older than `tick_cadence`).
2. For each, in a single transaction:
   - Read recent events since `last_tick_at` from the interaction event log.
   - Update the daily presence rollup.
   - Compute mood transitions from current mood, mood age, time of day, weather, and recent events.
   - Apply drift deltas to personality vectors (see §5.2).
   - Advance perch zones per bird based on mood + personality + time of day.
   - Decide whether a notebook entry is warranted (see §5.4).
   - Update `last_tick_at`.
   - Commit.

The default cadence is 60 seconds per account, tuned down or up in calibration. The lock is per-account, so the global tick is parallel across accounts.

A second, slower loop runs the day/night phase transitions and the ambient weather roll.

### Drift function

Personality drift is a low-pass filter over presence-and-interaction signals:

```
Δtrait = base_responsiveness[trait] × signal_magnitude
trait_new = trait_old + Δtrait
```

Key properties:

- **Monotonic toward expressive.** `Δtrait ≥ 0` for every drift input. There is no negative branch.
- **Bounded.** Each trait has an upper cap (e.g. 1.0). A trait at the cap stays at the cap. The cap is not exposed to the user.
- **Slow.** Total possible movement per trait per day is small (on the order of 1–3% of the trait range at peak signal). The user should not see trait movement in a single session; they should see it after weeks.
- **Personality-shaped.** A high-boldness bird has its boldness trait near its cap, so the same input produces a smaller absolute delta (and visually no movement) — the bird is already at the ceiling for that trait.

Inputs and approximate weights (final calibration in build):

| Input | Trait movement |
|---|---|
| Qualified presence-time (any session that meets the 3-signal rule) | small upward push on boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity (all five, weighted by per-trait responsiveness) |
| Listen-in on a specific bird | that bird's social_warmth and vocal_frequency get a stronger push |
| Offer accepted | that bird's curiosity gets a small push |
| Offer made (regardless of outcome) | that bird's boldness gets a small push |
| Settle | no direct drift effect; ends the presence window cleanly |

The drift function is **additive and server-authored**. The tick computes `Δtrait` and applies it; no client ever writes a personality value. This is the architectural rule that makes last-write-wins impossible.

Calibration target: a typical bird shows measurable drift in instruments after ~1 week of regular visits and visible drift to the user after ~3 weeks. The test harness runs synthetic daily-visit profiles against a fixture aviary and asserts that trait values move within a target band by day 7 and day 21.

### Mood transitions

Mood is an enumerated state with explicit transition rules:

```
mood' = transition(mood, time_of_day, weather, recent_events, personality)
```

- **Time of day** has a soft pull: late evening pulls toward drowsy; early morning pulls toward alert; mid-afternoon pulls toward content. The pull is small and personality-shaped (a high-boldness bird resists drowsy in late evening).
- **Weather**: light_rain dampens vocal_frequency (mood-level) briefly; soft_wind nudges some birds toward alert, others toward wary based on personality.
- **Recent events**: an accepted offer nudges the receiving bird toward content; an ignored offer nudges toward wary (but only at the mood level — it does not affect personality). A nearby bird's alarm call shifts the listener toward wary for a few ticks.
- **Personality ceiling**: a high-boldness bird cannot be in wary for very long; the transition function shortens the dwell time of wary proportionally to boldness.

Mood persists across sessions; the user never sees a bird "snap to neutral" on tab open.

### Call grammar runtime

The runtime is a client-side component (synthesis happens client-side per the audio constraint). The server hands the client:

- `call_signature_seed` per bird — a stable seed that, combined with `species_id`, deterministically produces a motif library and a per-bird variation envelope.
- The current `mood` — modulates tempo, pitch range, and motif selection at synthesis time.
- A `call_timing_seed` from the tick — controls when the next call is likely to fire (gated by vocal_frequency and mood).

At synthesis time, the audio module:
1. Picks a motif from the species's library.
2. Applies the bird's per-bird variation envelope (small pitch / timing / formant shifts).
3. Modulates by mood (drowsy → slower, lower; alert → tighter, sharper).
4. Schedules the call into the WebAudio graph at a randomized small offset for naturalness.

Call signatures are stable across mood and drift, so the user recognizes Pip by ear after weeks. Drift in `vocal_frequency` changes how often Pip calls, not what Pip sounds like.

### Notebook generation

A separate function in the tick, gated by both a time-since-last-entry check and a noteworthy-event check. The generator:

- Reads the recent state (mood, perch, weather, time of day, presence pattern).
- Picks a template family (greeting-order, weather-response, mood-dwell, presence-quiet, offer-acknowledged).
- Fills the template with bird names, perch references, and time-of-day wording in the same voice the rest of the product uses.
- Writes the entry.

Hard rules enforced in the generator:
- Lowercase, present-tense, no second person, no exclamation, no "you," no visit-frequency observations, no "you've been here every day."
- No entry is generated more than once per ~36 hours for an active aviary; the gap is bigger for quiet aviaries.
- The generator never reads per-account interaction history as observations of the user; it reads the aviary's behavior and writes about the aviary.

The generator's output is reviewed against a style-spec test corpus in CI to catch voice regressions — generic phrasing must fail the build.

---

## 6. Sync model

Multi-device sync is a property of the architecture, not a feature.

### Canonical state

- The simulation service holds the only authoritative copy of `bird.personality`, `bird.mood`, `bird.perch_zone`, `aviary.*`, and `notebook.*`.
- The account service holds the only authoritative copy of `account.*`, sessions, magic-link tokens, and visit tokens.
- The client holds an ephemeral cache of the latest snapshot plus a small interaction-event outbox. The cache is rebuilt on every page load.

### Conflict prevention

There is no merge problem because:

1. **Personality is server-only.** No client writes it. There is no two-writer scenario to reconcile.
2. **Mood is server-only.** Same.
3. **Perch is server-only.** Same.
4. **The interaction event log is append-only**, ordered by `server_received_at`. The tick consumes in order. A client never re-submits an event once it's been ack'd.
5. **Notebook entries are server-only**, generated by the tick.

What *can* go wrong, and how we handle it:

- **Two devices interact simultaneously.** Each posts its own events; the tick processes them in arrival order. Drift deltas are additive, so the order only matters for the magnitude of the day's drift, not for correctness.
- **A device posts an event for a state that has since moved on.** The event is still valid (it represents a real user action), and the server's mood / drift logic accounts for "stale" events. The system doesn't reject events based on state freshness.
- **A device posts an event while offline.** Events are queued in IndexedDB and replayed on reconnect. The replay is idempotent on `client_event_id`. If the queue is too old (>1 hour), the client surfaces a matter-of-fact "some of your actions weren't recorded" notice — never a naturalist apology.
- **A session expires mid-action.** The action is rejected with a clear "sign in again" surface; no partial state is written.

The visible conflict surfaces (sync error, session timeout, unsupported browser) all use matter-of-fact copy per the named exception.

### Visit revocation

A revocation flag on the invite row takes effect on the visitor's next snapshot pull. The visit-session-authenticated endpoint returns a 410 Gone with the matter-of-fact "visit no longer available" payload, and the client renders that surface.

---

## 7. Frontend rendering pipeline

### Scene composition

A single full-viewport canvas, plus a SVG overlay for the top bar. Three rendering layers, drawn back to front:

1. **Background** — soft sky / foliage, anchored to the day/night phase. Mostly static; subtle slow color drift over minutes.
2. **Middle plane** — the three perch zones (back, middle, front) and the perches themselves. Birds render into this plane.
3. **Foreground** — occasional leaves, a feather, a foreground branch passing through. Pure client-side ambient — generated at idle cadence, no server state, no per-leaf tracking.

### State interpolation

Birds are rendered as a procedural composition (a small set of SVG body parts composed in a `transform` tree) rather than a sprite sheet. This keeps the bundle small and lets the renderer compose the per-bird variation (plumage tint, posture, beak angle) from `species_id` + `personality` + `mood` + per-bird seed.

Between snapshots, the renderer interpolates:

- **Perch position** — linearly over the expected tick interval.
- **Body pose** — driven by a per-bird procedural pose function that runs continuously, never frame-stepped.
- **Idle micro-motion** — preen, scan, head-tilt, weight-shuffle. Each micro-motion has a personality-keyed probability per tick.
- **Color** — small plumage saturation drift, interpolated over seconds.

The first frame uses the most recent snapshot's positions and the bird's procedural pose at that time. There is no entry animation; the first frame is the aviary, mid-motion.

### Reduced-motion mode

In reduced-motion (gated by `prefers-reduced-motion` or user preference), micro-motion is replaced by a sequence of pre-composed poses cross-faded over ~600ms. The pose graph is the same one that drives the default mode; what changes is the inter-frame interpolation. Flight transitions cross-fade between perch poses rather than animating paths. Ambient leaf drift is removed; the day/night color shifts remain, slowed.

Calls continue at full quality (or with captions if audio is off). The notebook still writes. The aviary is still the aviary — just rendered in a different visual register.

### Empty / loading state

- **Initial load before the first snapshot arrives**: render a quiet field — soft sky color, a single faint leaf passing through. No spinner.
- **Between adoption flow and the first bird arriving**: the same quiet field; the first bird enters with a soft fly-in to its starting perch. This is the only place a "fly-in" animation exists, and it happens exactly once per bird, ever.
- **Snapshot pull taking longer than ~400ms**: extend the quiet field, do not show a spinner. The user perceives this as the aviary catching up, not the app loading.

### Top bar

A thin fixed top bar containing (in order): account/settings icon, accessibility settings, field notebook icon, offer affordance. Nothing else. The bar fades to ~10% opacity after ~5 seconds of cursor stillness, returns to 100% on pointer or keyboard activity. The icons are reach via keyboard tab; the offer affordance has a top-bar shortcut key.

### Rendering performance

- All scene state changes flow through a single reactive dispatcher; the render loop pulls dirty regions rather than redrawing the full frame.
- Per-bird SVG composition is cached per (species_id, mood, plumage_level_bucket) and reused across birds of the same kind.
- The render loop is throttled to 60fps by default; below 60fps in a given frame, the dispatcher drops pose micro-updates before it drops perch-position updates.

---

## 8. Audio pipeline

### WebAudio graph

A single `AudioContext` per page, created on the first user gesture (per browser autoplay policies). The graph:

```
[ Call source A ]─┐
[ Call source B ]─┼─→ [ Per-bird gain ] ─→ [ Listen-in mix ] ─→ [ Master gain ] ─→ [ Destination ]
[ Call source C ]─┘                                            [ Ambient bed ] ─↗
```

- **Per-bird gain** sets the bird's base level in the chorus.
- **Listen-in mix** re-balances per-bird levels: focused bird ramps up over ~1.5s; non-focused birds ramp down to a quiet ambient level (never to silence) over the same window. On disengage, the inverse.
- **Master gain** is user-controllable from accessibility settings; default respects the system volume.
- **Ambient bed** is a very quiet layer of soft natural sound (a faint breeze, distant leaves) generated procedurally, present whenever calls are muted or very quiet.

### Procedural call synthesis

Each call is a small graph of `OscillatorNode`s routed through a `BiquadFilterNode` and a `GainNode`, with a short attack-sustain-release envelope. Synthesis is per-call, not from a sample library:

- Species contributes a motif library (e.g. two-note rise, low trill, single sharp call, descending pair). Each motif is a small parameter set: base pitch, interval, vibrato depth, envelope shape.
- Per-bird variation envelope (derived from `call_signature_seed`) shifts pitch and timing slightly so no two birds of the same species sound identical.
- Mood modulation: drowsy → slower attack, lower pitch, longer release; alert → tighter, sharper; content → relaxed vibrato; curious → ornamented.

Synthesis is performed in an `AudioWorklet` so the main thread is not blocked. The worklet is initialized once and reused across calls. Per-call allocation is minimal — the worklet owns its node pool and reuses nodes.

### Listen-in

- Engage: clicking or keyboard-activating a bird sets that bird as the focus. Its `Per-bird gain` ramps to 1.0 over ~1.5s; others ramp to 0.25 over the same window.
- Disengage: clicking the focused bird, clicking empty aviary space, pressing Escape, or moving keyboard focus away from any bird triggers the inverse ramp.
- A focused bird's call frequency is unchanged — only its mix level. The user is hearing the same aviary, rebalanced.
- Caption generation runs in parallel with synthesis. A short prose caption is generated from the call's parameters (`mood`, motif, variation) at the moment the call is scheduled and is shown in the caption overlay for the duration of the call.

### WebAudio fallback

If the `AudioContext` cannot be created (older browser, permission denied, no audio hardware), the aviary plays in graceful silence with captions on by default. The caption overlay shows the same per-call captions; the user reads the aviary.

The product does not ship a recorded-audio fallback. A canned fallback would feel canned, and a procedural fallback at recorded quality would blow the bundle budget.

### Audio observability

- `AudioContext` state changes are logged (started, suspended, closed).
- Audio-pipeline errors (worklet exceptions, context failures) are reported as aggregate counters only — no per-account audio state.
- We do not record or transcribe the audio. There is no mic input anywhere in the product.

---

## 9. Accessibility surfaces

Accessibility is designed in, not retrofitted. The accessibility work is owned by the same engineers as the visual / audio surface, and it ships with v1, not after.

### Screen-reader narration

A narration generator runs in the client and reads from the same state cache as the visual renderer. It writes a short prose update every 30–60 seconds at idle, faster on user-initiated events (return-greeting, offer reaction, settle). Updates are pushed to a polite live region (`aria-live="polite"`).

Narration voice is the same naturalist field-notebook voice as the rest of the product. Lowercase, present-tense, specific. Examples in the prose are reused directly from the PRD.

The generator's behavior is exercised in CI by a fixture-state-driven test that asserts the output style stays within the voice-spec (lowercase, no second person, no announcement framing, no visit-frequency observations). Voice regressions fail the build.

### Reduced-motion mode

A first-class rendering mode, not a fallback. See §7 for the rendering details. The mode is reached via `prefers-reduced-motion` (system) or via accessibility settings (user opt-in), and the choice is persisted in account preferences.

### Call captioning

Opt-in via accessibility settings. When on, each procedurally synthesized call is paired with a short prose caption generated from the call's parameters. The caption appears as small text near the calling bird, fades in with the call, fades out at the call's release. Caption text is generated at runtime, not stored as fixed strings per call.

The user can also enable "audio off, captions on" as a single toggle in accessibility settings — this is the canonical state when WebAudio is unavailable.

### Keyboard navigation

All interactive surfaces are keyboard-reachable. The keymap:

- `Tab` / `Shift+Tab` — top bar items, then into the aviary scene.
- `Arrow keys` — move focus between birds.
- `Enter` / `Space` — engage listen-in on the focused bird.
- `Escape` — disengage listen-in; close the offer panel; close the field notebook.
- `o` — open the offer affordance.
- `n` — open the field notebook.
- `s` — trigger the settle gesture.
- `?` — open keyboard help.

Focus indicators are a soft, high-contrast outline visible against both bright and dim aviary states. The design system specifies the exact treatment.

### Contrast

All user-copy text passes WCAG AA at minimum. The aviary scene itself contains no user copy except in the top bar and the caption overlay; both are checked against the calm palette at every time-of-day phase to ensure AA across the day.

### Accessibility settings

A dedicated settings surface with the toggles grouped together: reduced-motion, captions, audio on/off. Each toggle is described in matter-of-fact voice in the setting label and the help text. There is no "we recommend" copy that would frame one choice as default in naturalist voice.

---

## 10. Performance budgets and observability

### Budgets

| Budget | Target | Failure mode |
|---|---|---|
| Initial JS bundle (gzipped) | < 2MB | Above this, time-to-first-bird becomes unrecoverable. |
| Time-to-first-bird (mid-tier mobile, 4G) | < 500ms | Above this, the aviary feels like it's loading. |
| Idle motion framerate (5-year-old laptop) | 60fps sustained | A 30-minute session must hold this. |
| Client memory growth (30 min session) | flat | Procedural audio buffers reused; no per-call allocation. Verified in CI. |
| Simulation-tick latency p99 | < 5s | Alarms before users notice the aviary "running slow." |
| Listen-in mix ramp | ~1.5s engage / disengage | Hard cuts feel like a UI of soloable tracks. |
| Snapshot payload | < 2KB at 2 birds, < 6KB at 7 birds | Drives pull frequency and battery cost. |

### What we measure

Aggregate-only real user monitoring:

- Page load timings (TTFB, FCP, LCP, custom first-bird-render time).
- Render-frame timing distribution.
- Audio-context errors and `AudioContext` state changes.
- Simulation-tick latency (server-side).
- Snapshot pull latency (client-side histogram).
- Magic-link and visit-link success rates.
- Anonymized session-duration histogram.

Synthetic perf checks: a fleet of automated browsers in 3–4 geographies running the aviary on a schedule, asserting the budgets above on common device profiles.

### What we deliberately don't measure

- Per-bird state, per-account interaction history, per-bird personality values.
- Notebook entry contents.
- Visit log contents.
- Anything that could reconstruct a user's relationship with their aviary.

The telemetry pipeline is a separate ingestion path. It does not have read access to the per-account simulation database. Any metric definition that would require reading a bird, a notebook entry, or an interaction event is rejected at the metric definition step, not the policy step.

### Bundle composition

The 2MB cap drives several downstream choices:

- Procedural audio synthesis client-side (no recorded assets).
- Bird visuals procedurally composed from small SVG parts, not sprite sheets.
- Aggressive code-splitting for low-traffic surfaces (account settings, accessibility settings, the visit-invitation flow, the field notebook detail).
- The top bar icons are inline SVG, not an icon font.
- The species pool and motif library are loaded with the initial bundle; the per-bird variation is computed at runtime from a small seed.

---

## 11. Rollout

### Ship plan

- **Internal alpha (engineers + design)**: bird engine and tick in isolation, fixture-driven tests for drift calibration, mood transitions, and notebook generation style. Accessibility and audio built in from the start.
- **Closed beta (small invited cohort, ~50 accounts)**: full end-to-end, with synthetic perf checks running against beta traffic. Calibration pass on the drift function, mood dwell times, and notebook entry rate.
- **Limited public launch (a few hundred accounts)**: the new-bird-by-age pacing is held to the slow end during this phase; we instrument drift distribution across the cohort to confirm the calibration target.
- **General availability**: opens to anyone. The two-starter-bird flow, magic-link auth, and visit invitations are live. The new-bird-by-age pacing ramps to its nominal cadence.

### What we instrument from day one

- Drift distribution across accounts (aggregate histogram; per-bird values never leave the simulation database).
- Mood dwell-time distribution per mood per personality bucket.
- Notebook entry rate per account (we want roughly one per few days, not one per session).
- Return-greeting variation (sanity check that the procedural variation is producing real variation, not three rotated variants).
- Accessibility-surface engagement: reduced-motion opt-in rate, captions opt-in rate, narration engagement.
- Audio-pipeline success rate across browsers and devices.
- WebAudio fallback rate (we expect this to be a few percent; a spike means something is broken).
- Snapshot pull latency and snapshot payload size at each bird count.
- First-bird-render time on common device profiles.

### What we don't ship with a toggle

- Streaks, achievements, badges, levels, scores, visit counters, calendars, milestones.
- Notifications about the aviary.
- Public surfaces, discovery, leaderboards.
- A "show-off" rendering mode for visitors.
- A personality-vector debug view, even for staff.

A toggle implies the feature can come back. The refusals in `non_goals.md` are not toggles.

### Post-launch evolution (not v1)

- A possible native app, if/when the team has the time and the case is clear.
- A possible new-bird-by-age refinement, if the pacing calibration needs it.
- A possible audio-mix enhancement that raises the recognizability ceiling above seven birds — not assumed, not planned, just a possible future lever.

---

## 12. Risks

### Drift calibration

**Risk**: the drift function is too fast, the user sees trait movement session-by-session, the aviary starts to feel like a Tamagotchi with longer timescales. **Or** the drift function is too slow, the user feels that nothing they do matters, the aviary starts to feel like a screensaver. **Mitigation**: the calibration target in `bird_engine.md` is named and testable. CI runs a fixture aviary through a synthetic 30-day presence profile and asserts drift values land in the target band at day 7 and day 21. The closed beta measures the actual drift distribution and lets us adjust `base_responsiveness[trait]` coefficients before public launch.

### Sync correctness

**Risk**: a subtle last-write-wins path sneaks in. A future feature lets a client mutate state directly, the simulation tick becomes one writer among many, and drift history is silently lost. **Mitigation**: enforce the architectural rule in the simulation service. The service has a single writer per personality column; the only code path that updates personality is the tick. A test in CI asserts that no API endpoint accepts a `personality` write from a client session. Code review checklist includes "does this touch a server-authoritative column from a client path?".

### Audio uncanniness

**Risk**: the procedural call grammar produces calls that are technically varied but emotionally flat. The user hears variation but not life. The aviary starts to feel like a synthesizer demo. **Mitigation**: a curated test corpus of "sounds alive" reference calls is built up during calibration and the synthesis output is reviewed against it. The audio work is its own review surface; the engineers who build it listen to a 30-minute session a day during calibration. A/B tests on motif selection weights, but only with synthesized output reviewed by humans, not blind.

### Accessibility regressions

**Risk**: a visual or audio change ships that breaks the reduced-motion surface, the caption voice, or the keyboard navigation. Reduced-motion users suddenly get a stripped fallback; screen-reader users suddenly get announcement-style prose. **Mitigation**: accessibility is owned by the same engineers as the visual / audio surface. A fixture-state-driven test exercises the narration generator, the reduced-motion renderer, the caption generator, and the keyboard map against a known input and asserts the output stays within the voice-spec and the surface-spec. Voice regressions fail the build. Accessibility settings have a "preview" affordance so the user can verify the surface is what they expect.

### Performance regressions

**Risk**: a feature adds bundle weight, pushes past the 2MB cap, and the time-to-first-bird budget becomes unrecoverable on a mid-tier mobile. **Mitigation**: a CI budget gate asserts the initial bundle stays under 2MB gzipped on every PR. A separate synthetic perf job runs the aviary against a fixture on a throttled-network profile and asserts the first-bird-render time stays under 500ms. Both gates block merge.

### Voice drift

**Risk**: a well-meaning contributor adds a "small" announcement surface — a "welcome back" tag, a "you've been here X days" tooltip, a "Pip is in a happy mood!" status chip. The voice becomes mixed; the aviary starts to read as a system showing states. **Mitigation**: the PRD names the rule, the design philosophy file (`product_brief.md`) names the rule, and the code review checklist calls out any text that addresses the user in the aviary surface. A style-spec test asserts that no UI string in the aviary surface starts with "you", "your", or "welcome". A separate review surface reviews every UI string for voice continuity.

### Privacy boundary erosion

**Risk**: an engineer reaches for `account_id` (or, worse, `email`) as a partition key for a new analytics pipeline, and PII ends up in places it shouldn't. **Mitigation**: the synthetic-UUID rule is enforced at the data-pipeline level. Email is stored encrypted in exactly one place, and only the account service can read it. The analytics warehouse has no read access to the simulation database or the account table. The metric definition step rejects any metric that would require reading per-bird state. A periodic audit (quarterly) checks that no log line, error trace, or telemetry event contains a PII-looking field.

### "Just one streak counter"

**Risk**: a streak counter or a visit counter slips in as a "harmless" feature. The user starts visiting for the counter, and the rest of the product becomes furniture. **Mitigation**: the refusal is named in `non_goals.md` and reiterated in this plan. There is no setting toggle for it. There is no A/B test that can introduce it. The code review checklist includes "does this surface a count of user behavior?" and the answer must be no.

### Browser fragmentation

**Risk**: an old browser (or a private-mode quirk, or a content blocker) breaks WebAudio, the rendering pipeline, or the auth flow in a way that affects a non-trivial percentage of users. **Mitigation**: the supported-browser list is the last two major versions of Chrome, Safari, Firefox, and Edge. Older browsers receive a matter-of-fact unsupported-browser surface. We do not maintain compatibility paths for very old browsers.

---

## 13. Defensible calls where the PRD is silent

These are decisions I made by reading the PRD. They are flagged so a reviewer can overturn them in build.

- **Tick cadence**: 60 seconds per account. The PRD says "slow, ~once per minute" and the calibration target is named. This is the starting value.
- **Mood set**: wary, content, curious, drowsy, alert. The PRD gives these as examples and says the exact set is finalized in implementation. This is the working set.
- **Species pool size**: ~6 species at v1. The PRD says "about six." This is the working number.
- **New-bird pacing**: one new species offer at intervals tied to aviary age — weeks to months between offers, calibrated against the drift calibration target. The PRD names "an aviary a few months old offers a third bird" as the rhythm; the exact intervals are tuned in build.
- **Field notebook entry gap**: minimum ~36 hours between entries for an active aviary; longer for quiet aviaries. The PRD says "roughly one entry every few days for a regularly-visited aviary." This is the working floor.
- **Presence activity window**: 3 minutes for the pointer-or-key check, leaning toward the longer side. The PRD says "a few minutes" and explicitly says to lean longer. This is the working value.
- **Magic-link expiration**: 15 minutes. The PRD is explicit.
- **Visit-link expiration**: 30 days. The PRD is explicit.
- **Account soft-delete window**: 30 days. The PRD is explicit.
- **Reduced-motion cross-fade duration**: ~600ms. The PRD doesn't specify; this is the working value.
- **Listen-in mix ramp duration**: ~1.5s engage and disengage. The PRD says "gradual" and "the interaction must feel like listening, not like switching channels." This is the working value.
- **Screenshot per snapshot pull**: keep-alive at ~15s during idle, ~60s during active interaction. The PRD says "low-frequency keepalive" and "on visibility change" and "on long render-frame gaps." This is the working schedule.

Each of these can be tuned in build against the calibration tests. None of them are load-bearing for the architecture — they are knobs, not foundations.

---

## 14. What this plan is not

- It is not a UI design. The visual specification (palette, perch-zone positions, bird proportions, top-bar treatment) is in the design system spec, a separate document with the visual designer.
- It is not a content policy. The notebook generator's voice is specified here, but the per-template prose is built and reviewed during build.
- It is not a security model. The auth flow is described, but the threat model, the key management, and the rate-limiting values are in the security design, a separate document.
- It is not a data-retention policy. The 30-day online / cold-storage archival of the interaction event log is a starting position; the retention rules and the right-to-be-forgotten mechanics are in the data-retention spec.

What it *is*: the architectural and engineering plan that turns the PRD into a buildable product, detailed enough that a separate team could execute it against the constraints in the PRD without further clarification on the load-bearing decisions.
