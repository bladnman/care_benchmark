# Pocket Aviary — v1 Implementation Plan

This plan interprets the PRD into an executable engineering plan. It is for a frontier engineering team that has read the PRD and needs the decisions, the architecture, the data model, and the build order to ship v1 without further clarification. The product is a browser-based virtual aviary where two-to-seven birds live in a single horizontal scene; the birds have hidden personality vectors that drift over weeks in response to presence-time; the simulation is server-authoritative; everything is shaped by five design principles — "feels alive," "notice, never announce," "charm from specificity," "restraint over richness," and a split voice (naturalist for the product surface, matter-of-fact for system surfaces).

The deliverable is the plan. No product code is written here.

---

## 1. Scope

### 1.1 In v1

- Single-user accounts; one aviary per account; magic-link sign-in.
- 2 starter birds at adoption; cap of 7 birds per aviary; species pool of ~6.
- Server-authoritative simulation tick (~1 minute cadence).
- Personality vector drift over weeks (monotonic toward expressive; no negative drift on neglect).
- Mood state per bird (wary, content, curious, drowsy, alert; final set decided in build).
- Procedural call synthesis (WebAudio); per-bird call grammars; chorus mixing.
- Idle motion that is mood-shaped and never reads as paused.
- Return-greeting, listen-in, offer (seed / song fragment / still pool), settle, field notebook.
- Presence accounting: conjunction of `visibilityState === 'visible'`, window focus, and pointer-or-key activity within a calibrated window.
- Multi-device sync (architectural property of server-authoritative state, not a separate feature).
- Optional visit invitations (per-invite opt-in, read-only ambient, revocable, default OFF).
- Accessibility: screen-reader narration in naturalist prose; reduced-motion mode as its own designed surface; call captions; WCAG AA contrast on user copy; full keyboard navigation.
- Performance budgets: <2MB initial JS (gzipped), <500ms time-to-first-bird on a mid-tier mobile over 4G, 60fps idle motion on a 5-year-old laptop, no memory growth over 30 minutes.
- Account export (JSON snapshot emailed as download link); soft-delete (30d) → hard-delete.
- Aggregate operational telemetry only; per-bird state never leaves the per-account simulation store.

### 1.2 Out of v1 (explicit non-goals)

The non-goals are not "we'll do them later." They are refusals that shape the rest of the plan.

- **No native app.** Web-only. The data model and protocols are not designed around native-client constraints.
- **No gamification.** No streaks, badges, levels, scores, visit counters, calendar of green dots, XP, tier, milestones, or "harmless" engagement celebrations — including in v1.x. The cumulative-effect argument in the PRD is final; the rule is absolute.
- **No Tamagotchi mechanics.** Birds do not die, get hungry, decay, or punish absence. The drift function is monotonic toward expressive.
- **No social-network surfaces.** No profiles, follows, feeds, public discovery, friend-of-friend chains, comments on visits, leaderboards, or rankings. The visit-invitation feature is the single social affordance and is read-only ambient.
- **No shared aviaries / multi-aviary accounts / household profiles.** One user, one aviary.
- **No payments / no paid tiers / no in-app purchases.** Aviary age — not visit count, not interaction score — gates new-bird offers.
- **No notifications about the aviary.** No push, no email about visits (host opt-in only, default off), no "welcome back" toast, no return banners. The aviary is the welcome.
- **No customizable scenes, no drag-to-place perches, no perch-selection panel.** Perch position is a signal the user reads, not a layout the user controls.
- **No recorded audio of any kind.** The bundle budget and the chorus mechanic both require procedural synthesis; the WebAudio-unavailable fallback is silence + captions, not canned audio.
- **No streak surface in any disguise** — no quiet calendar in settings, no exportable visit log, no notebook entry that says "you visited every day this week."

### 1.3 Ambiguity calls (defensible and named)

The PRD is tight but not exhaustive. A few calls that an engineering team would have to make:

- **Species pool is fixed, not user-extensible.** A future "import a species" feature would compromise the coherent-set feel; refuse.
- **Naming language is English-only at v1.** Bird names from the suggestion list are English short forms (Pip, Wren, etc.); user-entered names are accepted as Unicode strings but no transliteration or right-to-left layout work ships in v1.
- **Time-of-day resolution is the user's browser timezone.** The aviary reads `Intl.DateTimeFormat().resolvedOptions().timeZone` on connect and binds the day/night cycle to that zone for the session. A user who travels keeps the original zone until they sign out and back in; the aviary does not "follow" them across timezones in v1, which would make time-of-day mood signals incoherent.
- **Presence activity window is initially 4 minutes** — leaned long because watching birds without moving is the actual product. This is calibrated in build and re-tuned from drift telemetry after launch.
- **Simulation tick cadence is initially 60 seconds.** Calibrated in build; can stretch to 90–120s if server load warrants without affecting user-perceived behavior.
- **The mood-state enumeration is not strictly fixed** — wary / content / curious / drowsy / alert are the working set; final set is decided during the bird-engine build, with a hard cap of 6 to keep transition tables bounded.

---

## 2. Architecture

### 2.1 Service shape

Five services, each independently deployable, with the boundaries drawn so that the per-account simulation database is read by exactly one of them.

```
                 ┌────────────────────────────────┐
                 │  CDN edge (HTML, JS bundle,    │
                 │  initial state snapshot pre-  │
                 │  rendered per region)          │
                 └────────────────────────────────┘
                              │
                ┌─────────────┴─────────────┐
                ▼                           ▼
       ┌──────────────────┐       ┌──────────────────┐
       │  Web client      │       │  Visit client    │
       │  (host)          │       │  (visitor)       │
       └──────────────────┘       └──────────────────┘
                │                           │
                │ HTTPS (state pull,        │ HTTPS (state pull only;
                │ event append, auth)       │ no event writes)
                ▼                           ▼
       ┌──────────────────────────────────────────────┐
       │  Edge API gateway                             │
       │  (auth, rate limit, synthetic-UUID routing)  │
       └──────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
 ┌──────────────┐     ┌──────────────┐      ┌──────────────┐
 │  Account svc │     │  Aviary svc  │      │  Visit svc   │
 │  (auth,      │     │  (state      │      │  (invites,   │
 │  export,     │     │  pull, event │      │  revocation, │
 │  deletion)   │     │  append)     │      │  visit log)  │
 └──────────────┘     └──────────────┘      └──────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  Simulation engine    │
                  │  (tick, drift, mood,  │
                  │  notebook emitter,    │
                  │  narration emitter)   │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  Per-account sim DB   │
                  │  (canonical aviary    │
                  │  state)               │
                  └───────────────────────┘
                              │
                              ▼
                  ┌───────────────────────┐
                  │  Append-only event    │
                  │  log (offers, listen- │
                  │  in, presence pings,  │
                  │  settle, etc.)        │
                  └───────────────────────┘
```

- The **edge gateway** is the only place that resolves the synthetic UUID from a session token. Below the gateway, all routing and logs are by UUID.
- The **aviary service** owns the read path: it serves state snapshots, receives interaction events, and forwards events to the event log. It does not compute drift.
- The **simulation engine** is the only writer of personality state. It is a separate worker pool, scaled independently, with its own SLOs.
- The **visit service** handles invites, revocation, and the visitor's read-only pull. It shares the aviary service's read path; it does not have its own state.
- The **account service** owns auth, magic-link issuance, session tokens, export, and soft/hard deletion.
- **Aggregate telemetry** is collected by a sidecar that reads only the operational counters (request counts, tick latencies, render-frame timings, audio errors). The telemetry pipeline never touches the per-account sim DB; the per-account sim DB is never read by the analytics warehouse. This boundary is enforced at the IAM / network level, not by policy.

### 2.2 Client/server split — the cardinal rule

**The server is the only writer of personality state.** The client never sends a personality value, never recomputes a vector, never owns drift. The client writes interaction events into the append-only event log; the simulation tick consumes the log and applies deltas to the existing vector. This is the architectural property that makes multi-device sync, multi-day "the aviary continues without the viewer," and the "no last-write-wins" rule all fall out of the same decision.

If a planner or contributor is tempted to push drift into the client for performance reasons, that change is rejected. The performance cost of one extra snapshot per session-start is acceptable; the cost of breaking the architectural property is total.

### 2.3 Render pipeline boundary

The client has a clear pipeline, with a hot path and a cold path:

- **Hot path (per frame):** sprite/vector render of birds, ambient motion, mood-shaped idle micro-motion, day/night palette interpolation, call captioning layer, listen-in mix coefficients, focus indicator. Runs at the display's `requestAnimationFrame` cadence, throttled to 60fps.
- **Cold path (on event):** state-snapshot fetch, event-log append, animation transition setup, narration string update, notebook entry fetch, field-notebook panel render, settings panel render, visit panel render.
- **Reduced-motion path:** cross-fade pose sequencer, no `requestAnimationFrame` per-frame motion, no parallax; everything still updates, just in the slower cross-fade register.

The simulation engine never runs on the client. The day/night interpolation is client-side and purely presentational — it is bound to the user's local time, not to a server tick — but it is the only piece of "what is happening in the aviary" that the client computes locally, and it does not feed back to the engine in any way.

### 2.4 Build order

The build order is the dependency order. The team should not start a layer before its dependencies are in place, because later retrofits (especially to the append-only event log and the synthetic-UUID rule) are expensive.

1. **Account service + magic-link auth + synthetic-UUID.** No UI yet. Service-level tests for: link issuance, link expiration, link single-use, session token issuance, session revocation, email change with verification, export, soft-delete / recover / hard-delete.
2. **Per-account sim DB schema + append-only event log.** No simulation logic yet. Schema for birds, personality vectors, mood, perch state, ambient state, notebook entries, visits; event-log table with append-only enforcement at the DB role level (the simulation engine's role is the only role that can read + apply; the aviary service's role can only append).
3. **Simulation engine — tick + drift + mood.** This is the longest, most carefully calibrated part of the build. Calibration is verified against the named target: "measurable drift in instruments after ~1 week of regular visits; visible drift to the user after ~3 weeks."
4. **Call-grammar runtime + WebAudio synthesis.** Client-side; driven by server-side per-bird call-timing snapshots. The chorus mixer is built here.
5. **Rendering pipeline — scene composition, idle motion, day/night.** Bound to state snapshots; cross-fade reduced-motion path from day one.
6. **Interactions — return-greeting, listen-in, offer, settle, field notebook.**
7. **Visit invitations + read-only ambient visitor surface.**
8. **Accessibility surfaces — screen-reader narration, captions, keyboard navigation, contrast verification.**
9. **Telemetry + perf budgets + observability + synthetic perf fleet.**
10. **Hardening — drift calibration re-tuning, audio uncanniness sweeps, accessibility audit, browser matrix verification.**

---

## 3. Data model

The data model is server-side and per-account. The client never persists any of it locally beyond the in-memory render state. There is no localStorage cache of personality vectors; there is no IndexedDB shadow of the aviary; there is no offline mode in v1. (Offline would require defining offline drift semantics, which the PRD deliberately refuses to do.)

### 3.1 Account

```
Account {
  id:               UUID   // synthetic, generated at account creation
  email_encrypted:  bytes  // AES-GCM; key in KMS
  created_at:       time
  deletion_state:   enum { active, soft_deleted, hard_deleted }
  soft_delete_at:   time?  // set on soft-delete; null otherwise
  hard_delete_at:   time?  // set on transition to hard_deleted
  email_change:     { pending_new_email_encrypted, verify_token, expires_at }?
  settings: {
    visit_notifications_enabled: bool  // default false
    captions_enabled:             bool  // default false
    reduced_motion:               enum { off, system, on }
                                    // 'system' = matches prefers-reduced-motion; default
  }
  sessions: [
    {
      id:            UUID
      created_at:    time
      last_seen_at:  time
      user_agent:    string  // captured at session creation; not used as ID
      revoked:       bool
    }
  ]
}
```

The email is encrypted at rest; the encryption key lives in KMS. The UUID is the only identifier used in any other table, in any inter-service message, in any log line, in any partition or sharding key. The email is never logged.

### 3.2 Aviary

```
Aviary {
  account_id:      UUID
  created_at:      time
  aviary_age_days: int   // monotonically increasing; used by new-bird offer gating
  ambient: {
    weather:         enum { clear, light_rain, soft_wind }  // brief, server-tick-driven
    weather_started_at: time?
    weather_ends_at:    time?
    last_tick_at:       time
  }
  birds: [Bird]
  notebook: [NotebookEntry]   // append-only on the server; capped growth handled in §5
}

Bird {
  id:                  UUID   // stable internal id; survives name/species changes
  species_id:          string // from species pool
  name:                string // user-assigned; Unicode
  personality: {
    boldness:           float  // [0, 1]
    social_warmth:      float
    vocal_frequency:    float
    plumage_saturation: float
    curiosity:          float
  }
  mood: {
    state:        enum { wary, content, curious, drowsy, alert }
    entered_at:   time
    modifiers:    [string]   // transient, e.g. "rain_dampen"
  }
  perch: {
    zone:         enum { front, middle, back }
    height:       enum { low, high }
    entered_at:   time
  }
  call_timing: {
    next_call_after: time   // server-side
    chorus_lean:      float // per-bird, shapes chorus participation
  }
  identity: {
    adopted_at:   time
    adopter_session_id: UUID   // first session that adopted this bird
  }
}
```

The personality vector values are never exposed to the client in numeric form. The client receives a redacted snapshot (positions, moods, call timing, perch, name) for rendering, and may receive aggregate bands ("warm," "quiet," "vivid") for screen-reader narration if and only if the narration needs them — but not the underlying numbers. The numbers live on the server.

### 3.3 Presence events

```
PresencePing {
  account_id:    UUID
  session_id:    UUID
  bird_scope:    enum { aviary, bird_id }  // 'aviary' for ambient; specific bird for listen-in
  started_at:    time
  ended_at:      time?
  ended_reason:  enum { tab_hidden, focus_lost, idle_timeout, session_end, settle, tab_close }?
}
```

The client writes pings on state transitions (visibility change, focus change, idle timeout firing, listen-in start/end, settle, session end). The client never derives presence-time locally for any decision the server makes; the server is the only writer of presence accumulation.

### 3.4 Interaction events (append-only log)

```
Event {
  id:            UUID
  account_id:    UUID
  session_id:    UUID
  ts:            time
  kind:          enum {
                    offer_seed, offer_song, offer_pool,
                    listen_in_start, listen_in_end,
                    settle_start, settle_undo,
                    return_greeting,
                    notebook_viewed
                  }
  bird_id:       UUID?    // null for aviary-scoped events
  payload:       json     // kind-specific, small
}
```

The log is append-only at the DB role level. The simulation engine's role is the only role that reads the log; the aviary service's role appends and never reads back. A read-back of the log for debugging is gated to a separate audit role with its own access controls.

### 3.5 Notebook entries

```
NotebookEntry {
  id:           UUID
  account_id:   UUID
  bird_id:      UUID?
  created_at:   time
  prose:        string    // naturalist voice; generated server-side
  kind:         enum { observation, weather, milestone, structural }
  ttl:          time?     // entries can age out after N years (implementation detail)
}
```

The notebook is server-authored and read-only to the client. The user can scroll back indefinitely within the retention window; old entries are not archived, hidden, or paginated.

### 3.6 Visit invitations

```
VisitInvite {
  id:                UUID
  host_account_id:   UUID
  visitor_email_enc: bytes
  issued_at:         time
  expires_at:         time  // 30 days
  consumed_at:       time?
  revoked_at:        time?
  consumer_session:  UUID?  // session that consumed the invite, if any
}

VisitSession {
  id:                UUID
  invite_id:         UUID
  host_account_id:   UUID
  visitor_session:   UUID
  started_at:        time
  ended_at:          time?
  ended_reason:      enum { host_revoke, link_expired, tab_close, idle_timeout }?
}
```

The visit log on the host side is derived from `VisitSession` rows. The host sees email, date, approximate duration, and outstanding invites — exactly the PRD-stated shape.

---

## 4. API surface

The API is HTTPS+JSON over the edge gateway. All routes are by synthetic UUID after the gateway resolves the session token. Errors are matter-of-fact. The natural-language surface (return-greeting, narration, notebook) is not a separate API — it is generated server-side and delivered in the state snapshot or via a server-sent events stream for narration updates.

### 4.1 Auth

- `POST /v1/auth/magic-link` — body: `{ email }`. Rate-limited per email.
- `GET /v1/auth/consume?token=...` — consumes a magic link, issues a session token, redirects to the aviary.
- `POST /v1/auth/revoke-session` — body: `{ session_id }`. Matter-of-fact confirmation.
- `POST /v1/auth/email-change/start` — body: `{ new_email }`.
- `POST /v1/auth/email-change/confirm` — body: `{ token }`.
- `POST /v1/account/export` — generates JSON, emails download link.
- `POST /v1/account/delete` — soft-delete.
- `POST /v1/account/recover` — restore within 30d window.
- `GET /v1/account/me` — settings, sessions list.

All auth/account responses are in matter-of-fact voice in any user-visible error.

### 4.2 State pull

- `GET /v1/aviary/state?since={ts}` — returns:
  - `server_time`
  - `tick_at` (the canonical time the snapshot was last written by the tick)
  - `birds[]` — id, name, species_id, perch, mood (state only, not numerics), call_timing summary
  - `ambient` — weather, day_phase (computed from server timezone, not user timezone — the client rebinds to user timezone on render)
  - `narration` — current naturalist prose for screen readers
  - `notebook_recent` — last N entries (small; full notebook paginated separately)
  - `bird_count`, `can_adopt_third` (boolean; age-gated)

The snapshot is small (kilobytes) and CDN-cacheable per region, per account, with a short TTL (5–10s). The client pulls on `visibilitychange` to `visible`, on long render-frame gaps (suspend/resume handling), and on a low-frequency keepalive (~30s) while visible.

- `GET /v1/aviary/notebook?cursor=...` — paginated, read-only.

### 4.3 Event submission

- `POST /v1/aviary/events` — body: `{ events: [Event] }`. Append-only. The client batches events at most every 5 seconds (or on session end). Idempotency keys per event prevent double-writes across reconnects.

The client never sends a personality update, never sends a mood state, never sends a perch position. The server computes all of these from the event log on the next tick.

### 4.4 Visit invitations

- `POST /v1/visits/invites` — body: `{ visitor_email }`. Issues invite, emails link. Matter-of-fact confirmation.
- `GET /v1/visits/invites` — host lists outstanding + past.
- `POST /v1/visits/invites/{id}/revoke` — host revokes; takes effect on the visitor's next state pull.
- `GET /v1/visits/log` — host's visit log.
- Visitor consumption path: `GET /v1/visits/consume?token=...` — issues a visit-scoped session token that can only call the read-only ambient state endpoint.

### 4.5 Settings

- `GET /v1/settings`, `PATCH /v1/settings` — accessibility settings, visit-notifications toggle. The reduced-motion setting has three values: `off`, `system` (default — mirrors `prefers-reduced-motion`), `on`.

### 4.6 Server-sent narration stream (optional but recommended)

- `GET /v1/aviary/narration/stream` — server-sent events delivering prose updates on a slow cadence (one update per 30–60s at idle, faster on user-initiated events). This is the screen-reader narration channel. The same prose is also embedded in the state snapshot for clients that prefer polling.

---

## 5. Simulation engine design

The simulation engine is the only writer of personality state. It runs as a worker pool with a per-account tick at a slow cadence (~60s in v1). The engine reads the per-account event log, computes deltas, applies them in order, and writes a new canonical state. It is idempotent in the sense that re-running the same log produces the same vector — but the log is the source of truth, not the vector; the vector is the cache.

### 5.1 The tick

For each active account (any account with a non-soft-deleted aviary and recent event activity, or a rolling tail of recently-active accounts to keep the aviary "alive" even with no recent events), the tick does:

1. Read events since the last tick from the append-only log.
2. Compute presence-time accumulated in this window (sum of presence-ping durations with the conjunction enforced client-side, verified server-side against `visibilityState` and focus signals cross-checked against the session's last keepalive).
3. Apply drift deltas to each bird's personality vector (see §5.2).
4. Transition moods (see §5.3).
5. Advance mood timers; transition time-of-day modifiers (drowsy near dusk, alert near dawn).
6. Run bird-to-bird interaction (see §5.4).
7. Schedule next call times for each bird (procedural call grammar, not literal timings).
8. Possibly generate a notebook entry (sparsity rule, see §5.5).
9. Possibly generate a narration string update (same cadence as notebook, separate from user-event narration).
10. Commit the new state. The event log is not modified.

Tick latency SLO: p99 < 5 seconds (alarms on exceedance). Tick work is bounded per account (O(birds + events_in_window)); the per-account work cap is ~7 birds × ~100 events/window.

### 5.2 Drift function

Drift is a low-pass filter over presence-and-interaction signals. The filter is implemented as additive deltas, never as absolute value writes. The function is calibrated against the named target — measurable in instruments after ~1 week, visible to the user after ~3 weeks — and is the part of the engine that gets the most tuning post-launch.

A working shape (weights and rates are calibration parameters, not final numbers):

```
drift_delta_per_tick(bird, account) =
  w_presence * presence_minutes_in_window
    * lerp(bird.personality, expressive_target, drift_rate)
  + w_listen_in * listen_in_minutes_for_bird
    * social_warmth_step
    * vocal_frequency_step
  + w_offer_accepted * (if offer accepted in window)
    * curiosity_step
  + w_offer_near * (if any offer near bird in window)
    * boldness_step
  + w_settle * (if settle in window)
    * (no directional push; closes the presence window cleanly)
```

The `expressive_target` is a per-trait upper bound; drift is monotonic toward it and never below the current value on the same axis. A bird that gets ignored for two weeks does not become more wary — its boldness holds at whatever it had drifted to, and its vocal frequency may slowly relax toward a quiet floor (not zero; ambient). This is the implementation of "the user can leave for two weeks and come back to birds that are quieter than they were, not birds that have learned to mistrust them."

**Calibration verification.** The build ships a test harness that runs the drift function against simulated presence-and-interaction histories and asserts: (a) regular presence over 7 days produces a measurable delta on at least one trait; (b) zero presence over 30 days produces no negative delta on any trait; (c) presence-and-listen-in over 21 days produces deltas in social_warmth and vocal_frequency for the listened-in bird that are visible in the test harness's snapshots. These are the named tests the engine must pass.

### 5.3 Mood transitions

Mood is a small enumerated state per bird with a transition function over the same inputs as drift, plus time-of-day and ambient events. Transitions are computed on the tick. The mood transition table is a small Markov-like matrix, weighted by personality — a high-boldness bird has a lower transition probability into `wary`; a high-vocal-frequency bird has a higher transition probability into `alert` on a chorus event.

Mood persists across sessions; the mood the bird has at session-end is the mood it has at session-start, modulo whatever the tick has done in the interim. The mood state in the state snapshot is the same for the host and any visitor, which is what makes the visit feature render "exactly what the host would see at this moment" without special-casing.

### 5.4 Bird-to-bird interaction

Birds interact with each other, not just with the user. The engine runs a small graph per tick:

- If bird A calls, nearby birds with high vocal frequency and a mood compatible with joining have an elevated probability of calling within the next few seconds (modeled as a `chorus_lean` adjustment on call timing).
- If bird A enters `wary`, nearby birds with high social_warmth have a small probability of also transitioning to `wary` (wary spreads softly).
- The nightjar species in the species pool has a separate night behavior and is the only one that calls at full night.

Bird-to-bird is what makes the aviary read as a small social system rather than a row of independent NPCs.

### 5.5 Notebook entry generation

Notebook entries are generated server-side by a small template-and-slot system over the recent event log and current state. The voice is the naturalist field-notebook voice — lowercase, present-tense, specific. The sparsity rule is enforced: roughly one entry every few days for a regularly-visited aviary, more often when something noteworthy happens, never on every session. The implementation maintains a per-account "notebook-pressure" counter that decays slowly and only fires an entry generation when both the counter and a state condition (a notable mood transition, a first-of-week event, a weather passing) align.

The notebook generator is a closed system. There is no LLM, no template in the engineering sense of `String.format` — the entries are produced by a small rule-based selector over (state delta, time-of-day, weather, recent history) → template + slots. The voice is enforced by the template library, not by a model.

### 5.6 Narration string

The narration string (for screen readers) is generated the same way as notebook entries, with a separate cadence (one update per 30–60s at idle, faster on user-initiated events) and a slightly different slot set. The screen-reader user and the sighted user are reading the same aviary; the prose is the only difference.

---

## 6. Sync model

Multi-device sync is a property of the architecture, not a feature. The server is the only writer of personality state; the client never writes personality, never owns drift; both devices pull the same record. The shape of "sync" in this product is: the user signs in on a second device, the second device pulls the latest snapshot, and the aviary is the aviary the first device is showing.

### 6.1 Snapshot delivery

- The CDN edges the initial state snapshot with the HTML on the first navigation (small payload, per-region cache, short TTL). The 500ms time-to-first-bird budget depends on this.
- Subsequent snapshots are pulled on `visibilitychange`, on long render-frame gaps (suspend/resume), and on a low-frequency keepalive (~30s) while visible.
- The client does not request a snapshot on every event; events are appended client-side and reflected in the local render optimistically, with the next snapshot confirming or correcting.

### 6.2 Conflict prevention

The architecture prevents the conflict class entirely:

- Personality state is not shared between clients; it is owned by the server. There is no merge problem.
- Interaction events are append-only and idempotent. Two clients appending events concurrently produce a consistent log; the tick consumes them in order.
- The "laptop session in the morning writes a personality update, phone session at lunch overwrites it" failure mode from the PRD is unreachable because clients do not write personality updates under any code path.

The only sync-adjacent failures that can occur are:

- **Snapshot staleness during a long render frame.** Handled by the visibility/suspend-recovery pull.
- **Event-log double-write on reconnect.** Handled by per-event idempotency keys.
- **Magic-link replay.** Handled by single-use consumption.
- **In-flight session timing out mid-write.** Handled by the client's retry-with-backoff; the server is idempotent on the event-log side.

### 6.3 Sync error surface

When the rare conflict surface does appear, the voice drops to matter-of-fact per the PRD:

- `We couldn't sign you in. The link may have expired. Try requesting a new link.`
- `Your session timed out. Sign in again to keep watching.`
- `Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch.`

These strings live in the system-voice string table, not in the naturalist templates.

---

## 7. Frontend rendering pipeline

The renderer is a single horizontal scene, no panning, no scrolling, no zooming. All birds are always in frame. The scene is a vector/canvas composition, not a layered illustration trying to show off; the parallax is gentle.

### 7.1 Scene composition

Layers, back to front:

1. **Sky and day/night palette.** Interpolated continuously from the local time-of-day function, independent of the server tick. The user sees evening arrive over a few minutes; the palette never snaps.
2. **Background foliage.** Soft, slow parallax on cursor movement (subtler than the foreground).
3. **Middle plane — perches and birds.** Three perch zones (front, middle, back) × two heights (low, high). Birds are positioned by the engine's perch state; the client renders them at that position with the mood-shaped idle motion.
4. **Foreground ornaments.** Occasional foreground branch or leaf, gentle parallax.
5. **Ambient drift.** Leaves and feathers drifting through frame at slow random intervals, generated client-side at idle cadence. These are pure ornaments, not driven by the simulation tick.
6. **Captioning layer.** Optional, fades in with the call, fades out after.
7. **Focus indicator.** A soft, high-contrast outline around the focused bird, visible against both bright and dim palette states.
8. **Top bar.** Thin, fades nearly to transparent after a few seconds of cursor stillness; returns to full opacity on cursor movement or keyboard activity.

### 7.2 Idle micro-motion

Idle motion runs continuously and is mood-shaped. The motion vocabulary:

- Preening (content, drowsy)
- Scanning the scene, head-tilting toward sounds (curious, alert)
- Watchful body posture, slight retreat (wary)
- Sitting low on the perch with feathers fluffed (drowsy)
- Small body-shuffle, weight reset (universal)
- Idle motion is never paused at the engine level; the client may stop rendering when the tab is hidden, but the simulation continues server-side. The aviary the user comes back to is the aviary that has been ticking.

### 7.3 Transitions

- **Flight transitions** (bird moving between perches) — animated path, not teleport. Interpolation between snapshot positions.
- **Settle transition** — slow lighting shift to evening over a few seconds; calls quiet. The transition has a 5-second undo window: any click anywhere in the aviary within 5s of triggering settle reverses the lighting shift.
- **Return-greeting** — the greeting itself is a small bird action (glance up from preening, two-note call, head-tilt and step toward front perch, longer call) chosen by personality × absence-length. The visual is one of a small set of greeting animations, varied procedurally inside the rule.
- **First frame** — the aviary loads with motion already in progress. There is no fade-from-static, no entry animation, no spinner-resolves-into-aviary. The state snapshot is read, birds are placed at their current positions and current motions, and rendering starts. The loading state during a slow first-paint is a quiet field (soft sky color, perhaps one or two faint motion cues), not a spinner.

### 7.4 Reduced-motion mode

Reduced-motion mode is its own designed surface, not a stripped fallback.

- Idle micro-motion becomes slow cross-fades between still poses (a preen-pose cross-fading to another preen-pose).
- Flight transitions become cross-fades between perches.
- Ambient leaf drift is removed.
- Day/night color shifts remain, slowed.
- Calls still play at full quality (or caption, per the user's audio settings). Birds still drift. Mood still changes. The field notebook still notices things.

The user with `prefers-reduced-motion: reduce` gets a calmer, slower aviary, not a broken-looking one.

The reduced-motion path is built in parallel with the default path from day one; it is not a v1.x add-on. The cross-fade sequencer is its own rendering pipeline sharing the same state-snapshot input.

### 7.5 Bird visual assets

Bird visuals are procedurally generated where possible (silhouette, plumage palette, plumage-saturation-driven feather detail) and otherwise small SVGs or compact bitmaps. The bundle budget (<2MB gzipped) drives this; the audio system alone is incompatible with a heavy asset library.

---

## 8. Audio pipeline

The audio system is the affective spine of the product. It runs client-side via WebAudio. The bundle budget and the chorus mechanic both require procedural synthesis; there is no recorded audio of any kind in v1, including in the fallback.

### 8.1 Call grammar

Each bird has a procedural call grammar — a small set of motifs combined and varied at runtime, with personality-shaped timing and pitch. A motif is a short sequence of (pitch, duration, articulation) primitives. The grammar combines motifs with rules:

- **Variety:** no two consecutive calls from the same bird are drawn from the same motif template.
- **Pitch:** the per-bird base pitch drifts slowly with personality (vocal_frequency up = slightly higher pitch, but recognizably the same bird). The call signature must remain recognizable across mood and personality drift — a user who has spent two weeks with Pip knows Pip by ear.
- **Timing:** the inter-call interval is personality-shaped (vocal_frequency up = shorter interval, more calls). The mood shifts the interval too (drowsy = longer interval, content = baseline, alert = short).
- **Mood shaping:** the articulation and pitch curve shift with mood (drowsy calls are softer, alert calls are sharper, content calls have small ornamental inflections).

### 8.2 Synthesis

The synthesis path is WebAudio, with a small bank of oscillators and biquad filters per active call. Each call is generated as a short AudioBuffer in a worker thread, scheduled on the AudioContext, and disposed of after the call ends. No persistent buffer per bird; no shared buffer that could leak.

The synthesis worker pool is bounded (default: 2 workers). No per-call allocation that isn't freed; the "no memory growth over 30 minutes" rule is enforced in CI as a leak test.

### 8.3 Chorus mixer

The chorus mixer is the live mixer that combines the per-bird call outputs into a single scene mix:

- Each bird's call sits in its own gain node.
- The mixer applies a small per-bird spatialization (panning, gentle reverb).
- When listen-in is active on bird X, the mixer ramps X's gain up and the others' gain down on a slow ramp (engaging) and back to ambient (disengaging). The ramp is gradual — a hard cut would convert the aviary into a UI of soloable tracks. The other birds drop in mix but never go silent.

### 8.4 Listen-in mix decay

The listen-in mix is a re-balance, not a mute. The decay time on engage and disengage is calibrated in build (working value: 1.5–2.5 seconds) and the calibration target is "feels like leaning in to listen, not like switching channels." The mix returns to ambient on disengage with the same ramp.

### 8.5 WebAudio fallback

If WebAudio is unavailable (older browser, audio context permission denied, hardware issue), the aviary plays in graceful silence with captions on by default. The user's captions setting is unaffected; if captions were off, they default to on only for the WebAudio-unavailable case. The "no recorded audio" rule is unconditional.

### 8.6 Captions

Captions are short prose descriptions generated at runtime from the call grammar, not stored as fixed strings. The caption generator reads the motif + mood + pitch and emits:

> a soft three-note rise
> a low trill, paused, low trill again
> a single sharp call from the back perch

Captions appear near the calling bird, fade in with the call, fade out after. The voice is the same naturalist voice as the rest of the product.

---

## 9. Accessibility surfaces

Accessibility is a first-class, designed surface from day one. The accessibility work ships with the rest of the product, not after. The deliberate design stance: a screen-reader user, a reduced-motion user, a user with audio off all experience an aviary that feels alive, not a stripped variant of the product.

### 9.1 Screen-reader narration

The narration is a running prose stream, generated server-side or client-side from the same state the visual surface reads from, in the same naturalist voice as the field notebook. Cadence: roughly one prose update per 30–60 seconds at idle, faster on user-initiated events (a successful offer, a settle, a return-greeting).

The narration string is delivered via the state snapshot and the optional server-sent stream. It is an `aria-live="polite"` region. The narration language is consistent with the notebook — lowercase, present-tense, specific. A user who moves between the aviary surface and the notebook surface hears the same product.

### 9.2 Reduced-motion mode

Built in parallel with the default path from day one (see §7.4). The cross-fade pose sequencer is its own rendering pipeline. The setting has three values: `off`, `system` (default — mirrors `prefers-reduced-motion`), `on`.

### 9.3 Captions

Captions are opt-in via accessibility settings and are useful for users with audio off, hearing differences, noisy environments, or any situation where the audio isn't getting through. Captions are generated from the call grammar at runtime, not stored, and use the same voice as the rest of the product.

### 9.4 Keyboard navigation

All interactive surfaces are reachable by keyboard:

- Tab moves through the top bar items.
- Entering the aviary scene with Tab focuses the first bird.
- Arrow keys move focus between birds.
- Enter triggers listen-in on the focused bird.
- Escape exits listen-in.
- The offer affordance opens with a top-bar shortcut and is itself fully keyboard-navigable.
- The settle gesture is reachable from the top bar.

Focus indicators are visible against the aviary background — a soft, high-contrast outline that reads against both bright and dim aviary states.

### 9.5 Contrast

All user-copy text passes WCAG AA contrast. The aviary scene itself contains no user copy except the top bar and the captioning layer; the constraint applies primarily to chrome and to captions. The design system spec owns the exact ratios per surface.

### 9.6 Accessibility audit

A formal accessibility audit (external or internal-specialist) is part of the v1 launch gate, not a v1.x task. The audit covers screen-reader experience, reduced-motion experience, captions, keyboard, and contrast. The audit must be passed before the public launch, not after.

---

## 10. Performance budgets and observability

### 10.1 The four budgets

- **Initial JS bundle <2MB gzipped.** Drives procedural audio, procedural visuals, aggressive code-splitting. Non-critical surfaces (account settings, accessibility settings, visit-invitation flow) are code-split.
- **Time-to-first-bird visible <500ms** on a mid-tier mobile device over 4G. Achieved by CDN-edged initial state snapshot, render-before-non-critical-assets, and a small per-region cache.
- **60fps idle motion on a 5-year-old mid-range laptop.** Applies to a 30-minute session, not just the first minute.
- **No memory growth over 30 minutes.** A real CI test, not a guideline. Procedural audio buffers are reused; per-call allocations are freed; notebook entries do not retain references after scroll-out; workers and audio contexts are bounded.

### 10.2 WebAudio fallback budget

When WebAudio is unavailable, the aviary runs in silence with captions on; the render path is the same but the audio worker pool is not started. The bundle and time-to-first-bird budgets are unaffected.

### 10.3 Browser support

Last two major versions of Chrome, Safari, Firefox, and Edge. Older browsers receive a matter-of-fact unsupported-browser surface explaining what's needed. We do not maintain compatibility paths for very old browsers; the cost-benefit doesn't justify the bundle bloat.

### 10.4 What we measure

Aggregate-only:

- Page load timings (CDN, HTML, JS, first-bird-render).
- Render-frame timings (per-minute, with cohort dimensions for browser, viewport, device class).
- Audio-context errors (count, rate).
- Simulation-tick latencies (p50, p95, p99, with an alarm at p99 > 5s).
- Server-side request counts, latencies, error rates.
- Anonymized session-duration histograms (no per-account dimension).

### 10.5 What we deliberately do not measure

- Per-bird state, mood, perch, or personality vector values.
- Per-account interaction history or presence-time accumulation.
- Anything that could be used to reconstruct a user's relationship with their aviary.

This boundary is enforced at the metric definition in code, not by policy. The telemetry pipeline never reads the per-account sim DB. The sim DB is never read by the analytics warehouse. The two are separate IAM scopes.

### 10.6 Synthetic perf fleet

A small fleet of automated browsers runs the aviary on a schedule from common geographies, measuring first-bird-render, render-frame timings, audio errors, and tick latency from the client perspective. The fleet is the primary signal for performance regressions between releases.

---

## 11. Rollout

### 11.1 v1 ship

- Web-only. Last two major versions of Chrome, Safari, Firefox, Edge.
- Magic-link auth.
- 2 starter birds; cap of 7.
- All interactions (return-greeting, listen-in, offer, settle, field notebook).
- Multi-device sync (architectural property; not a separate feature).
- Visit invitations (default OFF).
- Accessibility surfaces built in from day one.
- Aggregate telemetry; per-account sim DB walled off from the telemetry pipeline.

### 11.2 Bird-count ramp

- A new aviary starts at 2 birds.
- New birds become available based on aviary age — not visit count, not interaction score, not paid tier. Working pacing: third bird offered around 3–6 months; fourth around 9–12 months; fifth and beyond at slower intervals, capped at 7. The pacing matches the rhythm of a relationship deepening; it deliberately refuses to teach the user that more attention earns more stuff.
- The new-bird offer is a quiet surface in the aviary (a small notice when the aviary reaches the age threshold), not a notification or a marketing email. The user can decline; declining does not re-offer.

### 11.3 Day-one instrumentation

- Aggregate telemetry from launch.
- Synthetic perf fleet running from launch.
- Drift calibration harness running in CI from launch.
- Memory-leak test running in CI from launch.
- Audio uncanniness review (manual listening sessions) before launch and at each release.
- Accessibility audit before launch.

### 11.4 Calibration loop post-launch

- Drift function is re-tuned based on aggregated, anonymized drift deltas in the first 4–8 weeks. The user-visible behavior should not change; the calibration moves the internal weights. The retune is governed by the test harness, not by hand-tuning against a user cohort.
- Audio calibration (call pitch range, inter-call interval, listen-in ramp) is tuned by manual listening, not by telemetry.

---

## 12. Risks

The risks below are the named failure modes the product is most likely to suffer. Each is paired with the mitigation, which is also a test or a check that the build must pass.

### 12.1 Drift calibration

**Risk.** Drift too fast and the product becomes a Tamagotchi where the user can move a number by clicking. Drift too slow and the product becomes a screensaver where nothing the user does seems to matter.

**Mitigation.** The drift function has a named calibration target (measurable in instruments after ~1 week, visible to the user after ~3 weeks). The test harness (§5.2) verifies the function against this target before each release. The post-launch calibration loop (§11.4) re-tunes weights under the same harness.

### 12.2 Sync correctness

**Risk.** A last-write-wins model on personality state would silently lose drift. A client-side simulation tick would produce divergent aviaries on different devices.

**Mitigation.** The architectural rule is enforced in code: the simulation engine is the only writer of personality state; the client never writes personality under any code path; the append-only event log is the source of truth. The drift function is additive deltas, never absolute values. Code review must reject any change that introduces a client-side write of personality state or a last-write-wins merge.

### 12.3 Audio uncanniness

**Risk.** The audio is the affective spine; a single canned-feeling call or a chorus phase-canceling artifact breaks the spell. Looped audio is the audible signature of dead software.

**Mitigation.** Procedural synthesis with real per-call variation; chorus built from procedural calls, not stacked loops. Manual listening sessions before launch and at each release. No recorded audio of any kind. The WebAudio fallback is silence + captions, not canned audio. The "no two consecutive calls from the same bird are drawn from the same motif template" rule is verified in tests.

### 12.4 Accessibility regressions

**Risk.** An accessibility surface that lands as a stripped fallback (animations off, semantic-markup narration, static scene) tells the user that their preference cost them the product.

**Mitigation.** Accessibility is built in from day one. The reduced-motion path is its own designed pipeline. The narration is in the same voice as the notebook. Captions are generated at runtime from the call grammar. A formal accessibility audit is part of the launch gate.

### 12.5 Tamagotchi regressions

**Risk.** A contributor adds a hunger meter, a happiness decay, a "your bird is sad" surface, or a positive-side equivalent that frames presence as earning the bird's good mood. Once any of these lands, the engine's monotonic-toward-expressive rule is contradicted and the "absence is fine" promise collapses.

**Mitigation.** Code review must reject any change that introduces a per-bird state with negative drift on neglect. The "no Tamagotchi" rule is the drift function's monotonicity; the design principle is enforced at the engine layer.

### 12.6 Gamification regressions

**Risk.** A "harmless" streak counter, a quiet calendar in settings, a "you've been here every day this week" notebook entry, a "first visit" badge. The cumulative effect is total; the first one is the foothold.

**Mitigation.** Code review and design review must reject any of these in any form. The notebook generator's closed template library does not include templates that observe the user's visit frequency. Settings does not include a visit-calendar surface. There is no streak counter, no "days visited" widget, no "achievement unlocked" surface anywhere in the product.

### 12.7 Privacy regressions

**Risk.** An engineer reaches for email as a "convenient unique identifier" and PII is sprayed across logs, partition keys, and aggregate analytics. Or an analytics dashboard is built on top of the per-account sim DB and the privacy boundary is silently crossed.

**Mitigation.** The synthetic UUID rule is enforced at the gateway and at the IAM level. The telemetry pipeline has read access to operational counters only; it has no read access to the per-account sim DB. The sim DB and the analytics warehouse are separate IAM scopes. Code review must reject any change that logs the email, that uses email as a routing key, or that adds an analytics read of per-account state.

### 12.8 First-frame regressions

**Risk.** A spinner-then-fade-in transition because it is the safe pattern; the aviary appears to be loading rather than continuing. The central conceit of the product is broken in the first 500ms.

**Mitigation.** The render path is built to draw the first bird from the initial state snapshot before non-critical assets are loaded. The loading state is a quiet field, not a spinner. Code review must reject any entry animation, fade-from-static, or "wake up" sequence.

### 12.9 "Welcome back" regression

**Risk.** A toast, a banner, a return modal, a "you've been gone X days" surface. The single most damaging violation of "notice, never announce."

**Mitigation.** No welcome surface exists in the codebase. The bird greeting is the entire welcome. Code review must reject any textual welcome on return, including "subtle" variants.

### 12.10 Visit-feature scope creep

**Risk.** A "harmless" chat overlay, a comment on the host's notebook, a visitor avatar in the scene, a public discovery feed. The social-network surfaces the PRD explicitly refuses.

**Mitigation.** The visit service has no write endpoints. Visitors read the state snapshot only. There is no chat, no comments, no avatar, no public discovery. The visit log on the host side is reachable on demand, not pushed.

---

## 13. Notes for the engineering team

A few decisions that fall out of the PRD but are worth stating explicitly so the build does not re-litigate them.

- **The simulation engine is server-authoritative and the only writer of personality state.** This is the architectural property the rest of the plan rests on. Do not propose client-side drift. Do not propose last-write-wins merges. Do not propose caching personality state locally in localStorage or IndexedDB.
- **The voice is split by surface, not by feature.** Naturalist for the aviary, the notebook, the narration, the offer prompts, the return-greeting narration. Matter-of-fact for sign-in, account settings, sync errors, accessibility settings, the unsupported-browser surface, the WebAudio-unavailable surface. The line is "is the user talking to the system as a system?"
- **The bundle budget drives the architecture.** Procedural audio, procedural visuals, code-split settings, no recorded audio, no heavy asset library. The 500ms time-to-first-bird is the affective bridge; the bundle is the lever.
- **Accessibility ships in v1, not in v1.x.** A reduced-motion mode that lands two months after launch as a "fix" is a launch that quietly told reduced-motion users the product wasn't for them.
- **The cumulative-effect argument governs every "harmless" feature request.** Streak counter, hunger meter, "your friend visited!" notification, public leaderboard, "explore other aviaries" feed, calendar of green dots in settings. Each is a foothold. Refuse the first one.
- **The species pool is fixed and small.** About six species, designed to feel like a coherent set of birds. No user-extensible species library, no rarity tiers, no paid species.
- **The notebook and the narration are closed template systems.** Not LLMs, not String.format, not freeform prose generation. A small rule-based selector over (state delta, time-of-day, weather, recent history) → template + slots, with the voice enforced by the template library. This is deliberate: it makes the voice auditable and keeps the narration in the same product as the rest of the surface.

---

## 14. Open questions deferred to build

These are not blocking; they are the calibrations that get made in the build and re-tuned from telemetry.

- Exact presence activity window (initial: 4 minutes).
- Exact simulation tick cadence (initial: 60 seconds; can stretch to 90–120s).
- Final mood-state enumeration (working set: wary, content, curious, drowsy, alert).
- Exact inter-call interval ranges per personality profile.
- Listen-in mix ramp time (initial: 1.5–2.5 seconds).
- Day/night color palette interpolation curve.
- Notebook sparsity threshold (working: roughly one entry every few days at regular use, more often on notable events, never per session).
- New-bird offer age thresholds (working: third at 3–6 months, fourth at 9–12 months, fifth and beyond at slower intervals, capped at 7).

These are tuning knobs, not architectural decisions, and the calibration harness in §5.2 plus the synthetic perf fleet in §10.6 are the primary signal for moving them.
