# PLAN.md — Pocket Aviary v1 implementation plan

This is a phase-1 plan: an executable interpretation of the PRD into architecture, data model, API surface, simulation engine, sync model, rendering/audio pipelines, accessibility surfaces, performance budgets, rollout, and risks. It is written for a frontier engineering team that can take it from here without further clarification. It does not restate the spec; it interprets it. Where the PRD is ambiguous, the plan makes a defensible call and names it as a decision (callout: `Decision:`).

The plan honors every constraint in the PRD's design philosophy, non-goals, and load-bearing rules. The single most important property of the system is the server-side simulation tick, which is the implementation rule that makes "feels alive, not robotic," "the aviary continues without the viewer," and "multi-device sync coherent" all literally true rather than aspirational. Most of this plan is downstream of that one decision.

---

## 1. Scope

### In v1

- Single-user accounts, email magic-link sign-in, per-device session tokens (revocable).
- One canonical aviary per account. Two starter birds at adoption, hard cap of seven.
- Six-species pool at launch; new birds offered based on aviary age (not activity, not paid tier).
- Server-side simulation tick at ~1-minute cadence; client renders snapshots and interpolates.
- Personality vector (5 traits) persisted server-side; drift is monotonic toward expressive; the user never sees numeric values.
- Mood (small enumerated state set) persisted across sessions; mood transitions driven by time-of-day, recent interactions, ambient events, and personality.
- Procedural call grammar per species, synthesized client-side via WebAudio; per-bird recognizability preserved across mood and drift.
- Front/middle/back three-perch zone layout; birds choose perch by mood+personality; no user placement.
- Day/night cycle anchored to the user's local timezone; rare ambient weather (rain, wind); ambient micro-motion (leaves, feathers, parallax).
- Interactions: return-greeting (procedural, varied by absence length, boldness, mood), listen-in, offer (seed / song fragment / still pool, per-bird cooldown), settle (with 5s undo), field notebook (auto-generated, read-only, sparse naturalist prose).
- Presence accounting: precise three-signal conjunction (visibilityState=visible AND window focus AND pointermove-or-keypress in last few minutes); presence-time is the dominant drift input.
- Field notebook: rare naturalist observations; no per-session feed; no visit-frequency observations of the user.
- Visit-invitation social feature: host emails a one-time link to a visitor; visit is read-only ambient, no co-presence, no chat, no avatars, no comments, no discovery, no leaderboards; revocable; expires in 30 days; defaults OFF; host visit log; opt-in visit notifications (off by default).
- Accessibility surfaces ship with v1: screen-reader running narration (naturalist prose, slow cadence), reduced-motion mode (designed cross-fade surface, not stripped fallback), call captions (procedurally generated from the call grammar at runtime), WCAG AA contrast on all user-copy, full keyboard navigation, visible focus indicators.
- Performance: <2MB gzipped initial JS bundle, <500ms time-to-first-bird on mid-tier mobile over 4G, 60fps idle motion on a 5-year-old laptop, no memory growth over 30 minutes (CI-enforced), WebAudio fallback to silence+captions-on-by-default (no recorded audio fallback), last-two-major-versions of Chrome/Safari/Firefox/Edge.
- Account export (JSON snapshot emailed to verified address), soft-delete (30-day recovery) then hard-delete.
- Aggregate operational telemetry only; per-bird / per-account interaction state never aggregated, never used for ML training, never shared.

### Not in v1 (respects `non_goals.md`)

- No native iOS/Android app. The data model and protocols are not designed with native-client constraints in mind.
- No gamification of any flavor: no achievements, streaks, levels, scores, badges, XP, ranks, tiers, "birds adopted" counter, green-dot calendar. Not as a setting, not as an opt-in, not on a milestone.
- No Tamagotchi mechanics: birds do not die, get hungry, show distress, or carry a decaying happiness meter. Drift is monotonic toward expressive; neglect produces ambient quietness only.
- No social network surfaces: no profiles, follows, public feed, shared discovery, mutual visits, comments on visits. The visit-invitation is the only social affordance.
- No shared/multi-aviary accounts, no household model, no multi-user aviary.
- No payments/billing tier, no paid species, no paid bird slots.
- No push/ping/email about the aviary. The aviary lives where the user visits it.
- No customization of the scene (no palette picker, no perch editor, no scene themes).
- No co-presence in visits, no shared cursor, no "your friend is here too" overlay.
- No notification to the host when a friend visits (default); opt-in toggle only, never surfaced during onboarding.
- No recorded-audio fallback path; no audio loops; no Spotify-of-birds.

### Decisions made here, named

- `Decision:` Presence activity window = **120 seconds** for the pointermove/keypress check, leaning long because watching birds without moving is the actual product. Calibrated during build; surfaced in account settings as "session activity" only as a debug hook for staff, not user-facing.
- `Decision:` Simulation tick cadence = **60 seconds** nominal, with a jittered ±5s to avoid herding all accounts onto the same wall-clock second. Personality-vector writes happen once per tick; mood writes can happen once per tick or on user-initiated event ingestion.
- `Decision:` Personality trait scalar range = normalized to **[0.0, 1.0]** with a per-trait "soft cap" at 0.95 LPF asymptote, so traits never pin to 1.0 and the math stays smooth.
- `Decision:` Mood enumerated set for v1 = **{wary, content, curious, drowsy, alert, settled}** (settled added to support the post-settle lighting state without conflating it with drowsy).
- `Decision:` Notebook entry generation target = **~1 entry per 3 days** for a regularly-visited aviary, with up to ~3 entries/day during notable weather or first-of-week events. Sparsity is enforced server-side by a "noteworthy" classifier over simulation deltas, not by per-session counting.
- `Decision:` Offer per-bird cooldown = **120 seconds** of wall-clock time per (bird × offer-type) pair. Cross-bird cooldowns are not modeled; only per-bird.
- `Decision:` Visit link TTL = **30 days** outstanding, **24 hours** once first opened, single-use per open (the link re-validates the visitor's email on each open via a fresh magic-link-style challenge).
- `Decision:` Magic-link TTL = **15 minutes**, single-use, rate-limited per email at **5 requests / 15 minutes** and **20 requests / 24 hours**.
- `Decision:` Field notebook retention = indefinite, no archival, no truncation, paginated read API with cursor.
- `Decision:` Top-bar fade threshold = **3 seconds** of cursor-stillness; returns to full opacity on cursormove or any keydown.

---

## 2. Architecture

### Service shape

Four services, plus a CDN edge and an auth mailer:

1. **edge** — CDN-served static HTML+JS+SVG bundle. The HTML carries a small inline JSON state bootstrap (the most recent canonical snapshot for the account, encrypted to the session token) so the first bird can render before the API round-trip. Cold loads fall back to a quiet-field color while the snapshot is fetched.
2. **api** — the public API the client talks to. Stateless; horizontally scalable. Routes: session/auth, snapshot pull, event submit (offer/listen-in/settle/presence-ping), notebook read, visit link claim, account settings/export/delete. Writes go to `sim`'s event log; reads come from `sim`'s snapshot cache.
3. **sim** — the simulation service. Owns the canonical aviary state, the per-account event log, the personality vectors, the mood state, the call-grammar runtime config, and the tick scheduler. The only writer of personality state. Runs the tick at 60s ± 5s per account. Reads come from a snapshot cache populated by the tick; clients never read the live in-memory state directly through a different path than the cache.
4. **notifier** — sends magic-link emails, visit-invitation emails, account-export download links, and (off by default) visit notifications if the host opted in. Does not push to the client; does not ping the user about the aviary.

A fifth internal service, **notebook-writer**, is a worker that consumes notable simulation deltas from `sim`'s tick output and writes notebook entries. It is the only writer of the notebook table. It is separated from `sim` so that notebook prose generation (which is heavier, slower, and more LLM-shaped) can run on a different cadence and resource pool than the tick.

### Client/server split

- Client owns: rendering, audio synthesis, presence detection, event submission, optimistic UI for in-flight events (e.g., a settle lighting change is shown locally before the server confirms), keyboard/focus, accessibility narration cadence (client-side throttle on top of server-side narration text), reduced-motion cross-fade, ambient leaf/feather ornament generation.
- Client does not own: personality vectors, mood state, perch assignments, call-grammar timing (the server-side tick emits per-bird call-timing plans; the client renders them through its synth), notebook entries, visit invitations, account state, drift math of any kind.
- The client never writes personality state. The client never derives personality from event history. The client never recomputes mood from a local replay. The client is a renderer and an event-ingestor, never a writer of canonical state.

### Render pipeline boundary

- The server emits a **snapshot** (~kilobytes) per tick: per-bird `{id, name, species, perch_zone, mood, call_plan_seed, micro_pose_seed, animation_phase, drift_visible_marker}` plus aviary `{local_time_offset, weather_state, settled_state, day_phase}` plus notebook cursor. The client renders from the snapshot and interpolates between consecutive snapshots using a 60fps client-side tween that knows about perch-to-perch transitions, mood-to-pose mappings, and call-timing plans.
- The client pulls a fresh snapshot on: (a) tab visibility regaining focus, (b) render-frame gap > 2 seconds (laptop suspend), (c) a low-frequency keepalive at 30s while visible. Each pull is conditional on a server `etag`-style snapshot version to skip redundant transfers.
- The first frame the user sees is the bootstrap snapshot inline in the HTML; there is no entry animation, no fade-from-static, no spinner. If the bootstrap snapshot is stale (cold cache, slow edge), the client renders a quiet-field color and the first bird appears in-place from the first real snapshot — never as a fly-in from offscreen (with the single deliberate exception of the empty-aviary → first-bird fly-in after adoption).

---

## 3. Data model

### Birds

```
bird:
  id            uuid, stable, never reused, never renamed-by-id
  account_id    uuid (the synthetic account UUID; never the email)
  species_id    enum (the six species at v1)
  name          text, user-assigned, renameable, no effect on id/personality/mood/call
  personality   personality_vector (5 traits, server-only, never sent numerically to client)
  mood          mood_state enum + mood_timer + mood_drivers (recent interactions, ambient events)
  call_grammar  reference into species call-motif library + per-bird variation seed
  perch_zone    enum {front, middle, back}  (server-authoritative; updated by tick)
  adopted_at    timestamp
  drift_history pointer to per-bird drift event log (append-only, for audit & export only)
```

### Personality vector

```
personality_vector:
  boldness            float [0,1], LPF, asymptote 0.95
  social_warmth        float [0,1]
  vocal_frequency     float [0,1]
  plumage_saturation  float [0,1]  (visual rendering reads this; client gets a derived render-hint not the raw value)
  curiosity            float [0,1]
  updated_at          timestamp (server tick only)
```

The client never receives these numbers. For rendering, the client receives per-bird **derived render hints** — e.g., `plumage_render_richness: low|med|high`, `boldness_pose_hint: forward|mid|back`, `vocal_cadence_hint: rare|regular|frequent`. These hints are coarse buckets derived server-side from the underlying scalars and are the only thing the renderer sees. This is the load-bearing implementation of the "user never sees personality numerically" rule — the constraint lives at the API boundary, not at the UI layer.

### Mood

```
mood_state:
  state           enum {wary, content, curious, drowsy, alert, settled}
  entered_at      timestamp
  timer_minutes   integer  (counts time in state; some transitions are timer-gated)
  drivers         struct (recent_interaction, time_of_day, ambient_event, personality_baseline)
```

Mood is persisted across sessions; the tick can transition it on each pass. A session-open does not reset mood to neutral.

### Presence events

```
presence_event:
  account_id       uuid
  started_at       timestamp (when the three-signal conjunction first became true)
  ended_at         timestamp (when any signal dropped, OR tab hidden, OR session timed out)
  duration_s       integer
  device_id        uuid (per-device session token id; presence is per-device, summed server-side)
```

The client submits presence-events as `{started_at, ended_at, duration_s}` pairs when a presence window closes. The server sums across devices into `presence_time_total` for the account, which feeds the drift LPF. The client never submits a raw "presence-time" number; it submits event boundaries and the server computes the duration from server-confirmed timestamps (the client's clock is not trusted for drift math).

### Interaction events (append-only event log)

```
event:
  id              uuid
  account_id      uuid
  bird_id_target  uuid (nullable for aviary-wide events like settle)
  type            enum {offer, listen_in_start, listen_in_end, settle, presence_ping, presence_window}
  payload         json (offer_type, song_fragment_id, etc., per type)
  client_ts       timestamp (client wall-clock, for ordering hints only)
  server_ts       timestamp (server wall-clock, the canonical order)
  device_id       uuid
  written_to_log_at timestamp
```

The event log is the only thing the client writes to. The tick consumes it in `server_ts` order. Personality deltas are computed from event-log aggregates per tick; mood transitions can be triggered immediately on event ingestion (a "soft mood" path) but the canonical mood state is reconciled by the next tick.

### Notebook entries

```
notebook_entry:
  id              uuid
  account_id      uuid
  written_at      timestamp (server)
  prose            text  (lowercase, present-tense, naturalist, specific)
  observed_at_sky timestamp (the in-aviary moment the entry describes; not the write time)
  source_event_id  uuid (nullable; some entries are pure observation, some are tied to an event)
  read_cursor_use  integer (sequence number for paginated reads)
```

The notebook is read-only at the API: there is no update/delete route for entries.

### Accounts

```
account:
  id                 uuid (synthetic; the only identifier used in logs, telemetry, shard keys, kafka, anywhere)
  email_encrypted    bytes (the only place the email lives; encryption at rest with KMS-managed key)
  email_verified_at  timestamp
  created_at         timestamp
  soft_deleted_at    timestamp nullable
  hard_delete_at     timestamp nullable  (soft_deleted_at + 30 days)
  settings           json  (visit notifications on/off, default off; any per-account feature flag)
  visit_log_visibility enum (host-only; not exposed to visitors)
```

```
session:
  id              uuid
  account_id       uuid
  device_id        uuid
  issued_at       timestamp
  revoked_at       timestamp nullable
  last_seen_at     timestamp
  kind             enum {host, visitor}
```

```
visit_invitation:
  id              uuid
  host_account_id  uuid
  visitor_email_encrypted bytes (until first open; then converted to a visitor account-id-equivalent for the visit log)
  issued_at        timestamp
  expires_at       timestamp (issued_at + 30 days)
  first_opened_at  timestamp nullable
  revoked_at_impl  timestamp nullable
  status_current    enum {outstanding, active, expired, revoked, consumed}
```

---

## 4. API surface

All routes are HTTPS JSON. The client authenticates with a bearer session token; the edge HTML bootstrap includes the token in a cookie. Errors return matter-of-fact English (per the named voice exception); never naturalist; never stack traces.

### Auth

- `POST /auth/magic-link/request` — body `{email}`. Rate-limited per-email. Sends a magic link via `notifier`. Always returns 202 (even for unknown emails — no enumeration).
- `GET /auth/magic-link/consume?token=…` — validates, invalidates, issues a per-device session token, sets cookie, returns `{account_id, session_id, device_id}`. 410 if expired/used; 400 if malformed.
- `POST /auth/signout` — revokes the current session token.
- `GET /account/sessions` — list of active device sessions; matter-of-fact tone.
- `POST /account/sessions/{id}/revoke`.

### Snapshot

- `GET /aviary/snapshot?if_none_match=ETag` — returns the current canonical snapshot for the account, or 304 if unchanged. Body is a small binary-friendly JSON; the ETag is the tick version. This is the only route the renderer needs at steady state.

### Event submit

- `POST /aviary/events` — body is a batch of events `{type, payload, bird_id_target?, client_ts}`. Returns `{accepted: [event_id], server_ts}`. The server never echoes back personality or mood changes here; the client learns of those via the next snapshot pull.
- `POST /aviary/presence` — body `{started_at, ended_at, device_id}`. Server computes duration from server-received timestamps; rejects any duration > 30 minutes per single window (forces the client to re-open a window if a session is that long — a defensive cap against drift inflation from a stuck signal).

### Notebook

- `GET /notebook?cursor=…&limit=N` — paginated read; default limit 20; max 100. Returns entries newest-first or oldest-first per the cursor direction. No mutation routes.

### Visit flow

- `POST /visits/invite` — body `{visitor_email}`. Creates an outstanding invitation; emails the visitor a one-time link via `notifier`. Rate-limited per-host.
- `GET /visits/invitations` — list of outstanding/active/revoked invitations for the host.
- `POST /visits/invitations/{id}/revoke` — immediately revokes; the visitor's next snapshot pull gets a 410 with a matter-of-fact "visit no longer available" body.
- `GET /visit/snapshot?token=…` — the visitor's snapshot pull. Returns a read-only snapshot of the host's aviary; same shape as `/aviary/snapshot` but no event-submit route is exposed to the visitor. The visitor cannot pull notebook entries (the notebook is the host's; the visitor sees only the live scene).
- `POST /visit/notifications/opt-in` — toggles the host's per-account visit-notification setting. Default off.

### Account

- `POST /account/export` — schedules a JSON snapshot email to the verified address.
- `POST /account/delete` — soft-deletes immediately, hard-deletes after 30 days; recoverable by sign-in during the window.
- `GET /account/settings` / `PATCH /account/settings`.

---

## 5. Simulation engine design

### The tick

The tick is the single most important component in the system. It is run by `sim` per account at 60s ± 5s, scheduled by a per-account timer wheel that survives `sim` restarts (timers are persisted, not held in memory only). On each tick:

1. **Read** the recent event log: all events with `server_ts` since the last tick's `consumed_event_id`.
2. **Presence aggregation**: sum `presence_event.duration_s` across all devices for this tick window; update the per-account `presence_time_total` accumulator.
3. **Drift computation**: for each bird, compute a per-trait delta from (a) the account's presence_time_total this tick, (b) listen-in events targeting this bird, (c) offer events targeting this bird, (d) settle events (mood-quiet only, no drift). Apply each delta as an additive update to the personality vector; clamp at the asymptote 0.95. Never subtract. See drift function below.
4. **Mood transitions**: for each bird, evaluate transition rules from current mood + drivers (recent interactions, time-of-day in account's tz, ambient weather, personality baseline). Apply any transition; update `mood_timer_minutes`. Some transitions are timer-gated (e.g., a bird must be in `wary` for at least 5 minutes before transitioning to `content` absent a strong positive input).
5. **Perch re-evaluation**: each bird's perch zone is re-evaluated as a function of (boldness, mood, current weather). The perch zone is server-authoritative; the client never picks a perch.
6. **Call-grammar plan**: for each bird, the tick emits a per-bird `call_plan` for the next 60 seconds — a sequence of `{t_offset_ms, motif_id, pitch_offset, duration_ms}` entries, parameterized by the bird's `vocal_frequency`, `mood`, and species motif library. The client's WebAudio synth renders this plan; it does not invent calls.
7. **Ambient event evaluation**: rare weather events (rain, wind) are scheduled by a separate low-frequency ambient scheduler that writes into the aviary's `weather_state`; the tick reads `weather_state` and propagates mood effects to birds in the affected window.
8. **Notebook-worthy detection**: the tick writes notable deltas to an internal queue consumed by `notebook-writer`. Notability is a classifier over (rarity, first-of-week, weather correlation, drift threshold crossings) — not per-session counting.
9. **Write** the new canonical snapshot to the snapshot cache; bump the ETag. The cache is the only thing the API reads.

### Drift function

Low-pass filter per trait, additive, monotonic-up. For a trait `T` and per-tick input `i` (normalized to [0,1] from presence/listen-in/offer signals):

```
T_new = T_old + α * i * (asymptote - T_old)
where:
  α = per-trait, per-input-signal learning rate (small; calibrated so 1 week of regular visits = measurable in instruments; 3 weeks = visible to user)
  asymptote = 0.95 (never pinned to 1.0)
  i = per-tick aggregate of presence-time / listen-in / offer signals, normalized
  T_old = current value
```

If `i = 0` (no presence, no interactions this tick), `T_new = T_old` — the trait does not move down. Neglect is not punished. A bird that has been left alone for two weeks has the same personality vector it had two weeks ago; what changes is its mood (ambient quietness, more drowsy/settled states, fewer choruses). The bird is "quieter than it was, not sadder than it was."

The asymmetry is the load-bearing rule. The drift function never subtracts. This is enforced by a unit test that fuzzes the function with adversarial inputs (long absence, all-zero inputs, malformed events) and asserts `T_new >= T_old - epsilon` for all traits, all inputs.

### Mood transition rules

A small rules table per (current_state, driver) → (next_state, min_timer). Example rows:

- `wary` + `content_input` (offer accepted by this bird) → `content`, min-timer 30s
- `wary` + `passage_of_time + early_morning` → `alert`, min-timer 5min
- `content` + `dusk_in_account_tz` → `drowsy`, min-timer 5min
- `drowsy` + `early_morning` → `content`, min-timer 10min
- `curious` + `offer_nearby` → `content`, min-timer 30s
- `alert` + `passing_rain` → `wary`, min-timer 2min
- `*` + `settle_gesture` → `settled`, min-timer 30s (the settled state is the post-settle lighting state)
- `settled` + `tab_reopen / any user_event` → `content` (cleared on next session-open)

Personality baseline modulates the rules: a high-boldness bird is less likely to enter `wary` on the same input; a high-curiosity bird is more likely to enter `curious` on offer_nearby. The modulation is a small per-rule bias on the transition probability, not a rule replacement.

### Call-grammar runtime

Per-species motif library: 6 species × ~12 motifs each = ~72 motifs in v1. Each motif is a short parameterized fragment `{note_intervals, base_pitch, amplitude_envelope, duration}`. The tick's per-bird call plan picks motifs, varies pitch by ±a semitone, varies duration by ±15%, and sequences them with personality-shaped inter-call gaps (a high-vocal-frequency bird calls more often; a drowsy bird lengthens the gaps). The client synth renders the plan through WebAudio oscillators + envelopes + a small per-species filter chain to give each species a recognizable timbre.

Per-bird recognizability is a function of (a) species timbre (the filter chain), (b) the bird's stable variation seed (set at adoption, never changes), (c) the bird's call_plan parameter ranges (personality-shaped but bounded). The recognizability test is: a user who has spent two weeks with Pip can identify Pip's call from a chorus of three birds, blind. This is a playtest-level target, not a unit test; we instrument the call-plan parameters so we can audit that variation is bounded (a unit test that asserts no two birds in the same aviary produce overlapping call-plan parameter envelopes for >X% of windows).

---

## 6. Sync model

There is no client-to-client sync. There is no client-side state to merge. There is no eventual consistency to reconcile. Both of the user's devices read the same canonical record on `sim`.

### How a single canonical state propagates

- Each device pulls `/aviary/snapshot` on its own cadence (visibility change, suspend-resume, 30s keepalive). Each pull is conditional on the snapshot ETag. The server returns 304 if the device already has the current version.
- Both devices see the same ETag-stream over time; there is no per-device fork of the aviary.
- The renderer's interpolation is local-only and is overwritten without comment on the next snapshot pull — a bird that the laptop animated moving from perch A to perch B is snapped to its actual server-determined perch on the next pull, with the tween hiding the snap if the pull is recent enough (within 2s) and a visible "catch up" tween if the pull is older (laptop suspended for 10 minutes — the bird tween-snapths to its actual current position, which is the central conceit made concrete: the aviary continued without the viewer).

### How conflicts are prevented

- Only the server writes personality. Clients never send absolute values; clients send events. The tick consumes events in `server_ts` order. There is no last-write-wins on personality because there is no write at all from the client.
- Mood has a "soft" client-side path (a settle gesture immediately shifts the local render to evening lighting before the server confirms) — but the canonical mood is reconciled by the next tick. The local soft state is purely visual; it never feeds back into drift. The client's local mood rendering is a preview, not a state.
- Perch assignments are server-authoritative. The client never picks a perch. A bird the user sees on the front perch on their laptop is on the front perch on their phone because both renderers read the same snapshot.
- Device clock skew is handled by: the server timestamps every event on receipt; the client's `client_ts` is metadata for ordering hints only and is never used for drift math. Presence durations are computed from server-received timestamps, not from the client's claimed `duration_s`.

### Conflict surface (rare)

The only sync conflicts the user sees are auth/session-level: magic-link replay, session timeout mid-write, server outage. These surface in matter-of-fact tone (`accounts_sync.md`'s named exception). There is no per-bird "we couldn't reconcile this bird's state" surface; the architecture precludes it.

---

## 7. Frontend rendering pipeline

### Scene composition

The aviary is one horizontal scene. Three planes:

- **background**: sky color (a slow gradient driven by account-tz time-of-day), soft foliage shapes (procedural SVG with low-frequency wind ripple), weather overlay (rain = soft falling lines, wind = leaf ripple).
- **middle**: three perch zones (front / middle / back), birds rendered as procedural SVG-with-inlined-bitmaps (the species' silhouette is a small SVG; the plumage richness is a server-derived render-hint that modulates saturation and feather-detail density; the mood is rendered through pose and idle-motion choice).
- **foreground**: occasional leaf/feather drift (client-side ornament, no per-item state), thin top bar above the scene.

The scene is responsive: viewport width drives perch spacing; aspect ratio is preserved such that all birds remain onscreen at all viewports; narrow phone viewports compress horizontally without cropping; wide desktop viewports widen. The minimum supported viewport is 320px width; the maximum is unbounded (the scene widens, it does not letterbox).

### Idle micro-motion

Each bird has a continuously-running idle-motion state machine on the client: preening, scanning, head-tilting toward the most recent call from another bird, body-shuffle. The state choice is mood-shaped (wary → scan; content → preen; curious → tilt; drowsy → fluffed-low; settled → eyes-closed-low; alert → upright-still). The animation phase is seeded by the snapshot's `micro_pose_seed` so two devices render the same bird in the same pose at the same moment — important for the "the laptop and the phone show the same aviary" property to feel literal.

### Transitions

- Perch-to-perch: a smooth tween (350–700ms, mood-shaped — a wary bird's move is slower and more hesitant; a bold bird's move is direct).
- Mood-to-pose: a cross-fade of idle-motion state over ~400ms when the snapshot's mood changes for a bird.
- Day-to-evening: a slow palette tween (the sky-color gradient lerps over the day; settle shortens the path to the evening palette over ~3s).
- Settle: lighting shifts to evening over ~3s; calls quiet (a server-side mood shift to `settled` for all birds drives the call-plan gaps longer); the 5s undo affordance is purely client-side (a click anywhere within 5s of settle locally reverts the lighting tween and submits a `settle_undo` event, which the next tick ignores for mood purposes — settle_undo does not produce drift).

### Reduced-motion mode

A separate rendering path, not "animations off." Triggers: `prefers-reduced-motion` media query OR an opt-in toggle in accessibility settings (which the client persists locally and also writes to account settings so it syncs across devices).

In reduced-motion:

- Idle micro-motion is replaced by slow cross-fades between still poses (a wary bird cycles through {scan-from-back, watchful-still, look-up-briefly} as cross-faded stills at ~3s per pose).
- Perch-to-perch transitions become cross-fades between the two perch poses (~800ms cross-fade, no flight path).
- Ambient leaf/feather drift is removed.
- Day-to-evening palette tween remains, slowed (the full day palette tween takes ~2x as long; settle takes ~6s instead of ~3s).
- Calls still play at full quality; captions still available; the audio surface is unchanged.
- The notebook still writes; the simulation still drifts; the user still gets the actual product. Reduced-motion is a different aesthetic, not a degraded one.

### Aviary appears in motion on first paint

The HTML bootstrap carries the most recent snapshot inline. The first paint composes birds at their snapshot-determined perches, mid-pose (the `micro_pose_seed` is the phase), with ambient motion already running, the day-phase palette already at the account-tz-correct color, and (if WebAudio is up) a call already in progress. There is no fade-from-static. There is no spinner. If the bootstrap snapshot is missing (cold cache, slow edge), the first paint is a quiet-field color (the soft sky color for the account's current time-of-day) and the first bird appears in-place from the first real snapshot — never a fly-in from offscreen, with the deliberate exception of post-adoption first-bird appearance (which is a fly-in from off-screen to the starting perch, by spec).

---

## 8. Audio pipeline

### Procedural call synthesis

Each bird's call plan (from the snapshot, computed server-side by the tick) is a 60-second window of `{t_offset_ms, motif_id, pitch_offset, duration_ms}` entries. The client's WebAudio synth renders each entry:

- A small bank of oscillators (per-species: e.g., a warbler uses two detuned triangle oscillators + a soft highpass filter; a nightjar uses a low-passed sawtooth + tremolo LFO).
- An amplitude envelope per motif (attack/decay/sustain/release shaped to the species' characteristic call shape).
- A small reverb send (a single ConvolverNode with a short, smooth impulse response shared across all birds; this is the glue that makes the chorus sound like birds in the same place rather than birds in separate studios).
- Pitch and duration variation from the plan.

Two birds calling at once share the same reverb send and master bus; this is the chorus. No layering of recorded loops. No phase-canceling artifacts.

### Chorus mixing

The master bus has a per-bird gain that is set by:

- The bird's `vocal_frequency`-derived render-hint (a frequent-calling bird sits at -6dB; a rare-calling bird sits at -12dB on the master).
- The listen-in state (see below).
- The mood (a drowsy bird is at -9dB; an alert bird is at -3dB).

The chorus is the sum of these per-bird gains over the master. No "chorus mode" exists as a separate code path; chorus emerges from per-bird call-timing plans overlapping in time.

### Listen-in mix decay

When the user listen-ins on a bird (click/tap/keyboard-focus):

- A ramped gain change over ~700ms: the focused bird's master gain rises by +6dB toward a cap; all other birds' master gains drop by -6dB toward a floor of -18dB (audible-ambient, never silent).
- The ramp is implemented as a `GainNode.linearRampToValueAtTime` on each bird's gain, scheduled from the current value to the target over 700ms.
- On disengage (click focused bird again, focus another bird, click empty aviary, Esc, focus moves out), the ramp reverses over the same 700ms.

Other birds never go silent. The listen-in mix is a re-balance, not a mute.

### WebAudio fallback

On WebAudio unavailable (older browser, audio context permission denied, hardware issue), the synth does not instantiate. The aviary renders in graceful silence; call captions turn on by default (the captions setting has three states: off / on / on-if-no-audio; the fallback sets it to on-if-no-audio automatically; the user can override to off in settings but the default in this state is captions-on).

There is no recorded-audio fallback path. The "no recorded audio" rule is unconditional.

### Audio context lifecycle

The AudioContext is created on first user-gesture (a click or keydown anywhere in the document, per browser autoplay policies). Before that, the aviary renders silently with captions on. The first user interaction (even a scroll or a focus move) unlocks the context and the calls begin at their next scheduled `t_offset_ms`. There is no "click to enable audio" modal — the first gesture unlocks it without announcement, which is consistent with "notice, never announce."

---

## 9. Accessibility surfaces

### Screen-reader narration

- A live region (`aria-live="polite"`) carries running narration prose. The narration is generated from the same snapshot the visual reads from; it is updated on a cadence of ~30–60s at idle and on a faster cadence (within 2s) on user-initiated events (offer accepted, settle, return-greeting).
- The narration text is server-authored in naturalist voice (lowercase, present-tense, specific), the same generator that writes the notebook, restricted to current-state observations (not past-tense notebook entries). This unifies the voice across the aviary surface and the notebook surface; a screen-reader user moving between them hears the same product.
- The narration is generated server-side so the prose generator can run on the same resource pool as the notebook-writer; the client receives prose strings in the snapshot. The client's only job is to push the prose into the live region at the right cadence (it does not generate prose).
- Cadence throttle: idle narration every 30–60s; event narration within 2s of the event; both queues coexist with the polite live region (idle prose does not pre-empt event prose). High-frequency narration is explicitly avoided because it overwhelms the screen reader's queue.

### Captions for calls

- An opt-in setting (off by default; auto-on in the no-audio fallback path).
- For each call the synth plays, the caption text is generated from the call-plan entry (motif + pitch + duration mapped to naturalist prose: "a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch"). The caption appears as small text near the calling bird's perch, fading in over ~200ms with the call's onset and fading out over ~400ms after the call's end.
- The caption text is generated client-side from the call-plan entry (the call plan is already on the client; the caption is a pure render of it). This keeps the caption in sync with what was actually played — no pre-stored strings, no mismatch between audio and caption.

### Reduced-motion mode

See §7. The designed cross-fade surface, not a stripped fallback. Ships with v1.

### Keyboard navigation

- Tab moves through the top bar items: account/settings → accessibility settings → notebook → offer → settle (left-to-right order; the visual designer picks the icon order).
- Tab into the aviary scene focuses the first bird (leftmost on-screen).
- Arrow keys move focus between birds (Left/Right; Up/Down reserved for future perch-zone navigation but not mapped in v1).
- Enter triggers listen-in on the focused bird.
- Escape exits listen-in (focus stays on the bird; the mix returns to ambient).
- The offer affordance opens with a top-bar shortcut; the offer menu is itself fully keyboard-navigable (arrow keys move through seed / song fragment / still pool; Enter offers; Escape closes the menu without offering).
- The settle gesture is reachable from the top bar (a single key press; the undo affordance works on any click or any keydown within 5s of settle).

### Focus indicators

A soft, high-contrast outline (visual-designer-specified) that reads against both bright (midday) and dim (night, settled) aviary states. The outline color is computed against the snapshot's current day-phase palette, so it adapts rather than being a fixed color.

### WCAG AA contrast

All user-copy text — top bar labels, settings surfaces, account surfaces, error surfaces, captions, narration when displayed visually — passes AA at minimum. The aviary scene itself contains no user copy except in the top bar, so the contrast constraint applies primarily to the chrome. The design system spec (separate document) specifies per-surface ratios; this plan honors them as a floor.

---

## 10. Performance budgets and observability

### Bundle size: <2MB gzipped initial JS at first paint

- The initial bundle is the renderer + WebAudio synth + presence detector + top bar + accessibility narration client. Account settings, accessibility settings, the visit-invitation flow, the notebook full-reader (the inline preview is in the initial bundle; the deep paginated reader is not), and the offer menu's song-fragment library are code-split.
- The species pool's SVG silhouettes and motif libraries are lazy-loaded per species on first appearance (a bird appears → its species' motif library is fetched in the background; the bird calls from the synth's default motif until its species library arrives, ~50ms gap that the user does not perceive because the first call is short).
- Procedural call synthesis is partly how we hit this budget — recorded audio at the variation we need would not fit. Bird visual assets are procedural SVG + small compact bitmaps; no large textures.

### Time to first bird visible: <500ms on mid-tier mobile over 4G

- The HTML bootstrap carries the inline snapshot so the first bird renders before the API round-trip.
- The render path does not wait for non-critical assets (species motif library, full notebook) before drawing the first bird. The first bird is drawn from the inline snapshot, with the synth's default motif until the species library arrives.
- The CDN edge serves the HTML with the inline snapshot encrypted to the session token; the edge has the snapshot cache warm for active accounts (the snapshot cache is replicated to edge on write, so an active account's snapshot is hot at the edge closest to them).

### 60fps idle motion on a 5-year-old laptop

- The render loop is `requestAnimationFrame`-driven; the snapshot is the source of truth but is only consulted on pull, not every frame. The 60fps tween is purely client-side; it does not round-trip.
- Procedural SVG is the render substrate (not canvas, not WebGL) — modern browsers composite SVG efficiently and the bird shapes are simple enough that SVG is the right choice. (Canvas/WebGL is an option if profiling shows SVG is the bottleneck; the plan starts with SVG for portability and falls back only if needed.)
- The audio synth runs on its own worker thread (AudioWorklet) so it does not contend with the render loop on the main thread.

### No memory growth over 30 minutes

- A CI test runs a 30-minute synthetic session and asserts the JS heap does not grow. Procedural audio buffers are reused (a single ring buffer per bird's synth voice); no per-call allocation. Notebook entries scrolled into view do not retain references after scroll-out (a small LRU on the notebook reader's rendered entries; the cursor is server-side so re-reading is cheap). AudioWorklet threads and AudioContexts are bounded (one per aviary; never spawned per call).

### Performance observability

- Synthetic checks: a fleet of automated browsers (Playwright) runs the aviary on a schedule from common geographies (US-East, US-West, EU-West, APAC-Southeast) and records: page-load timing, first-bird-render timing, render-frame timing over a 5-minute session, audio-context creation success/failure, simulation-tick latency (the synthetic client observes snapshot ETag changes and infers tick cadence).
- Real User Monitoring: page-load timings, first-bird-render timings, render-frame timings (sampled client-side, batched to the API on a slow cadence to avoid per-frame network), audio-context errors, simulation-tick latencies. All aggregate-only; no per-bird state, no per-account dimension. The privacy boundary is honored at the metric definition: a metric that cannot be defined without per-bird state is not collected.
- Error budget: simulation-tick latency p99 alarms if it exceeds 5 seconds. The tick is supposed to take well under 1 second; an alarm at p99 5s catches degradation early before users notice the aviary "running slow."
- We deliberately do not measure: per-bird drift rates across accounts, per-account visit frequency, per-account session duration. These would require per-account state in the telemetry pipeline, which the privacy rule forbids.

---

## 11. Rollout

### How v1 ships

- Single staged rollout: 100% of new sign-ups on day one; existing waitlist invites ramped over the first week at 10%/25%/50%/100% daily to surface drift calibration issues at small N before big N.
- The simulation service is the rollout risk — it is the only service that holds canonical state and the only writer of personality. We deploy `sim` with a read-only shadow period of 48 hours where the new tick runs in parallel with the old and the diffs are audited per-account before we cut over. (For v1 launch there is no "old" to compare to; this is a v1.1+ practice but built into the deployment process from day one.)

### Birds-per-aviary ramp

- v1 starts at two starter birds and caps at seven. The cap is enforced at `sim`'s adoption logic; a third-bird offer appears at aviary age 30 days; fourth at 90 days; fifth at 180 days; sixth at 365 days; seventh at 730 days. The pacing is tied to aviary age (created_at), not to visit count or interaction total, which is the load-bearing refusal of the gamification trap.
- The age thresholds are configurable in `sim`'s settings (a staff-only surface) so we can tune the pacing from real adoption data without code changes. The defaults above are the v1 ship values.

### What we instrument from day one

- Aggregate operational telemetry: request counts, latency histograms (p50/p90/p99), error rates, simulation-tick latency p99, audio-context creation success rate, first-bird-render timing, render-frame timing, JS bundle size at first paint (reported by the client), memory-growth check results from the synthetic fleet.
- Drift calibration signals: per-account (private to the user, never aggregated) the user can request an "aviary audit" from account settings which shows them their own bird's drift-over-time as a small naturalist-prose summary ("pip is bolder than she was three weeks ago"). This is the only surface where drift is made visible to the user, and it is in voice, not numbers. For staff calibration, an internal-only dashboard (gated to staff accounts, never exposed to users, never aggregated across accounts) shows per-account drift trajectories for a small opted-in alpha cohort — this is the calibration instrument for the drift LPF, and the alpha cohort is recruited explicitly for this purpose.
- We deliberately do not instrument from day one: per-account visit frequency, per-account session duration, per-account drift comparisons. Adding any of these would require per-account state in the telemetry pipeline and would violate the privacy rule.

---

## 12. Risks

### Drift calibration

The drift LPF learning rates (`α` per trait per input signal) are the most sensitive parameter in the system. Too fast and the product is a Tamagotchi (the user can move a number by clicking); too slow and it is a screensaver (nothing the user does matters). The 1-week-instrument / 3-week-user-visible target is a calibration band, not a formula.

**Mitigation:** Alpha-opted-in cohort (above) with per-account drift trajectory instrumentation; weekly review during the first month; tuning of `α` via the `sim` settings surface without code change; automated tests that assert monotonicity and asymptote-clamp behavior but not calibration rate (rate is calibrated from real use, not from a test).

**Risk shape:** The failure is silent — no test catches it. The signal is user reports of "my birds feel different than I expected" (too fast) or "I've been visiting for a month and nothing has changed" (too slow). The instrumentation has to be in place from day one to catch this before it becomes a press of user reports.

### Sync correctness

The architecture precludes last-write-wins on personality by precluding client writes of personality at all. The residual risk is at the event-log-consumption layer: if the tick consumes events out of `server_ts` order (a clock skew between two `api` instances, a kafka partition mis-ordering), the drift math is computed on a wrong-ordered event stream.

**Mitigation:** `server_ts` is assigned by a single monotonic sequencer per account (the `api` instance that receives the event writes the `server_ts` from a per-account sequence number held in a small persistent counter — a Redis-coordinated counter or a Postgres sequence per account; not a wall clock). The tick consumes events in `server_ts` order; an out-of-order event is held until its predecessor arrives (with a timeout — if a predecessor never arrives within 5s, the event is processed and the gap is logged as a metrics anomaly).

**Risk shape:** The failure is silent and corrupts drift slowly. The mitigation is at the ingestion layer, not the tick layer. CI test that fuzzes event arrival order and asserts the tick's personality output is order-independent for order-equivalent event streams.

### Audio uncanniness

The procedural call synth is the affective spine. Failure modes: (a) the synth's timbre is too "computer-y" (sine-wave-ish, buzzy) and the call reads as synthetic; (b) two birds' calls phase-cancel audibly when their motifs overlap; (c) the recognizability target fails — a user can't tell Pip from Wren by ear after two weeks.

**Mitigation:** The per-species filter chain is tuned by ear during build, not by a target spec. The shared reverb send is the glue that masks per-oscillator artifacts. The call-plan parameter envelope audit (above) is the recognizability test. A weekly playtest during the build phase where a listener identifies birds blind from a chorus is the qualitative test; we ship when a non-engineer listener can pass it.

**Risk shape:** Uncanniness is felt, not measured. The synthetic fleet cannot catch it. The build-phase playtest is the only instrument; missing it means shipping an aviary that feels like a synth demo.

### Accessibility regressions

The accessibility surfaces are designed (not stripped fallbacks). Failure modes: (a) the narration cadence drifts too high and overwhelms the screen reader; (b) the reduced-motion cross-fade is too slow and reads as broken; (c) the captions are out of sync with the played call (a pre-stored string instead of a runtime render); (d) the focus indicator is invisible against the night palette.

**Mitigation:** The narration cadence is throttled at the client (a 30s minimum gap between idle narration prose pushes); the caption is rendered from the call-plan entry on the client (no pre-stored strings, sync is structural); the focus indicator color is computed against the snapshot's day-phase palette (adapts, not fixed). A CI test asserts the narration prose generator does not produce more than N prose strings per minute on a synthetic idle aviary; another asserts the caption string matches the call-plan entry that was played.

**Risk shape:** A contributor adds a "harmless" pre-stored caption for a frequently-played motif to save CPU; the caption drifts out of sync; the accessibility surface quietly degrades. The structural mitigation (caption is rendered from the call-plan, not stored) has to be enforced at code review.

### Drift inflation from presence signal

The presence definition's three-signal conjunction is the load-bearing precision. Failure modes: (a) a client bug counts "tab is open" as presence (one of the three signals is checked too loosely); (b) the activity window is too long (a 30-minute window counts a user who walked away at minute 1 as present for 30 minutes); (c) the per-window 30-minute server cap is bypassed by a client that opens a new window before the old one closes.

**Mitigation:** The three-signal conjunction is asserted at every presence-event submit — the server re-checks (the client sends the `started_at` and `ended_at` but the server only accepts the event if the duration is plausible and within the cap). The activity window is calibrated at 120s with a documented tradeoff (longer = more permissive, shorter = more strict); leaning long because watching-without-moving is the product. The per-window cap is enforced server-side; a client that opens a new window while an old one is still open gets both events rejected with a sync-conflict matter-of-fact error.

**Risk shape:** The failure is silent and inflates drift across the entire user base. The mitigation is at the server boundary, not the client. CI test that fuzzes presence-event boundaries and asserts the server's `presence_time_total` accumulator does not exceed the wall-clock elapsed time by more than a small epsilon.

### Visit-link replay

A visit invitation link is one-time-per-open but the visitor can re-open the link within the 24-hour window. Failure mode: a visitor bookmarks the link and uses it past the 24-hour window; or the visitor's email is leaked and a third party opens the link.

**Mitigation:** Each open re-validates the visitor's email via a fresh magic-link-style challenge (a fresh 15-minute link emailed to the visitor's address on each open; the visitor must click that fresh link to actually open the visit). This is heavier friction than a single one-time link but is the right surface for the privacy commitment (the host invited a specific email, not a URL).

**Risk shape:** Friction at the visitor's open path. The tradeoff is privacy-correct vs. low-friction; the plan picks privacy-correct because the visit feature is opt-in and the host has 30 days to re-issue if the visitor finds the friction too high.

### Simulation service availability

`sim` is the only writer of canonical state. If `sim` is down, the aviary the user sees is the last snapshot they pulled — it continues to render (the client's tween keeps animating from the last snapshot, the synth keeps playing from the last call-plan, the renderer keeps drawing) but the state stops advancing. The user does not see an error in the aviary surface (the aviary is in motion); the failure is operational, not user-facing, until the user notices that "the birds have been in the same mood for an hour."

**Mitigation:** `sim` is multi-AZ, hot-replicated, with a recovery SLO of <5 minutes. The synthetic fleet's tick-latency p99 alarm at 5s catches `sim` degradation before the user notices. The client continues rendering during `sim` downtime for up to 30 minutes (the tween degrades gracefully — birds keep their last snapshot's mood and idle-motion, calls continue at the last call-plan's cadence, ambient motion continues; after 30 minutes the client shows a quiet matter-of-fact "the aviary is taking a moment; we'll have it back" surface, which is the rare sync-conflict surface).

**Risk shape:** The failure is invisible to the user until it's been going on for a while. The operational alarm is the only early-warning instrument. The 30-minute client-side degrade-gracefully window is the user-facing buffer.

---

## 13. Notes on the load-bearing rules, for the executing team

A few rules are repeated here because they are the ones a well-meaning contributor will most reliably violate:

- **No "Welcome back" toast.** The bird greeting is the entire welcome surface. A toast in this position is the single most damaging violation of "notice, never announce." Code-review-checklist item #1.
- **No streak counter, no green-dot calendar, no visit-frequency surface.** Not as a setting, not as an opt-in, not on a milestone. The rule is absolute. Code-review-checklist item #2.
- **Drift is monotonic toward expressive.** Never subtract. The unit test that asserts this is the only line of defense; it must run on every PR. Code-review-checklist item #3.
- **Personality vector is never exposed numerically.** The API returns render-hints (buckets), not scalars. A debug view that shows scalars is a regression even if it's staff-only. Code-review-checklist item #4.
- **The client never writes personality state.** The API has no `PUT /aviary/personality` route. A contributor who adds one has broken the sync model. Code-review-checklist item #5.
- **Calls are procedural.** No recorded audio, no audio loops, no fallback to a recorded call. The WebAudio fallback is silence+captions, never canned audio. Code-review-checklist item #6.
- **The simulation is server-side.** A client-side tick would collapse multi-device sync. The client renders snapshots; it does not tick. Code-review-checklist item #7.
- **Email is stored once, encrypted; synthetic UUID is the identifier everywhere else.** A log line that contains an email is a PII leak. Code-review-checklist item #8.
- **The notebook writes observations of the aviary, not observations of the user.** "Pip greeted first today" is in scope. "You visited every day this week" is out of scope. Code-review-checklist item #9.
- **Accessibility ships with v1, not after.** Reduced-motion landing two months later is a v1 that told reduced-motion users the product wasn't for them. Code-review-checklist item #10.

---

## 14. Out-of-scope confirmations

The plan does not implement: native apps, gamification, Tamagotchi mechanics, social network surfaces, payments, shared aviaries, multi-aviary accounts, public discovery, leaderboards, push notifications, recorded audio, co-presence in visits, scene customization, bird hunger meters, bird death, bird distress, achievement surfaces, streak surfaces, calendar surfaces, "birds adopted" counters, profiles, follows, comments on visits, mutual visits, friend-of-friend chains. Each refusal is per `non_goals.md` and per the relevant PRD file; the plan treats them as load-bearing refusals, not as missing features.

---

End of plan.
