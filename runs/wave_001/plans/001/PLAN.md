# Pocket Aviary — Implementation Plan (v1)

This is the executable plan for Pocket Aviary v1. It interprets the PRD (`prd/`) into concrete architecture, data models, APIs, and build order. A separate engineering team should be able to execute from this document without further clarification. Where the PRD is silent or ambiguous, a defensible call is made and marked **[call]**.

---

## 1. Scope

### 1.1 In scope (v1)

- **Accounts**: single-user accounts, email magic-link sign-in, per-device revocable sessions, verified email change, account export (JSON, emailed), soft-delete (30 days) → hard-delete.
- **Aviary**: one canonical aviary per account. Two starter birds at adoption (system-selected species from a pool of ~6), user-assigned names, renameable. Bird count cap of 7. New birds offered at aviary-age intervals.
- **Bird engine**: hidden 5-trait personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity), monotonic-toward-expressive drift driven by presence-time (dominant), listen-in, and offers; 5-state mood system (wary, content, curious, drowsy, alert) with daily-ish reset, persistence across sessions; procedural call grammar per species with per-bird recognizable signature; mood-shaped idle micro-motion; bird-to-bird interaction (call responses, mood contagion, emergent chorus).
- **Interactions**: return-greeting (one bird, varied by boldness/mood/absence length, staggered when multiple); listen-in (gradual mix re-balance, never mute); offers (seed, song fragment, still pool; per-bird cooldown of a few minutes); settle (opt-in soft session-end, 5-second undo window); field notebook (auto-generated, read-only, sparse naturalist entries); presence accounting (visibility AND focus AND recent input activity).
- **Scene**: single horizontal scene, three perch zones, no pan/scroll/zoom; local-time day/night cycle with one night-active species; rare ambient weather (short rain a few times a week, occasional wind) with small mood effects; continuous ambient micro-motion (leaves, feathers, subtle parallax); thin top bar (account/settings, accessibility settings, notebook, offers) that fades on cursor stillness; no UI chrome inside the scene; first frame renders mid-action (no entry animation, no spinner — quiet field as the load state).
- **Social**: visit invitations only — per-invite opt-in by email, one-time links, read-only ambient rendering, revocable with immediate effect, 30-day expiration for unused invites, silent visit log in settings, visit-notifications toggle default OFF. No chat, avatars, comments, discovery, leaderboards, or show-off rendering. Visitors generate zero presence-time and zero interaction events.
- **Accessibility**: screen-reader naturalist prose narration (30–60s cadence at idle, priority bump on user-initiated events); designed reduced-motion mode (slow cross-fades between still poses, no ambient leaf drift, full audio/captions preserved); runtime-generated call captions from the call grammar; WCAG AA contrast on all user copy; full keyboard navigation (top bar Tab, per-bird arrow-key focus, Enter = listen-in, Escape = exit, keyboard-reachable offer and settle).
- **Performance**: <2MB gzipped initial JS bundle; <500ms time-to-first-bird on mid-tier mobile over 4G; 60fps idle motion sustained over 30 minutes on a 5-year-old laptop; zero memory growth over 30 minutes (CI-tested); procedural WebAudio synthesis only (no recorded audio anywhere); graceful-silence + captions WebAudio fallback.

### 1.2 Out of scope (v1) — per `non_goals.md`, absolute

- Native mobile apps (and no native-client constraints in the data model or protocols).
- Gamification of any kind: no achievements, streaks, levels, scores, badges, counters, calendars of visits, XP, ranks, tiers — not as toggles, not opt-in, not ever.
- Tamagotchi mechanics: no death, hunger, distress, decaying meters. Neglect produces ambient quietness only.
- Social-network surfaces: no profiles, follows, feeds, discovery, friend-of-friend, mutual visits, visit comments.
- Also not shipping: payments, shared/multi-user aviaries, customizable scenes, multi-aviary accounts, push notifications of any kind, recorded-audio fallback paths, support for browsers older than the last two major versions of Chrome/Safari/Firefox/Edge.

### 1.3 Voice contract (enforced in code review)

Two registers, hard line between them:

- **Naturalist** (lowercase, present-tense, bird-named, specific): aviary surface, notebook, narration, captions, offer prompts.
- **Matter-of-fact** (normal capitalization, direct): sign-in, account settings, sync/conflict errors, accessibility settings, visit-revocation surfaces, unsupported-browser surface.

A lint-style copy check runs in CI: any user-facing string in the system-surface directory tree must not match naturalist patterns and vice versa **[call: a curated copy lint rule set; human review remains the backstop]**.

---

## 2. Architecture

### 2.1 Shape

Three deployable units plus edge:

1. **Web client** (SPA, TypeScript): renders the aviary, synthesizes audio, collects presence signals, emits interaction events. Owns no canonical state.
2. **API / realtime service** (stateless, horizontally scaled): auth, snapshot delivery, event-log ingestion, visit-invite flow, account management, notebook read API.
3. **Simulation service** (single-writer per aviary): the server-side tick. Reads the append-only interaction event log, advances moods, applies drift deltas, writes canonical state, generates notebook entries and return-greeting plans.

Datastores:

- **Canonical store** (Postgres): accounts, birds, personality vectors, moods, notebook entries, invites, sessions, event-log offsets. All internal references keyed by synthetic account UUID.
- **Append-only event log** (Kafka or equivalent ordered log, partitioned by account UUID): presence pings, listen-in start/stop, offers, settles. Retention: consumed-and-compacted after tick processing plus a short replay window (7 days) for disaster recovery.
- **Snapshot cache** (Redis or CDN-edge KV): latest rendered-state snapshot per aviary for fast first paint.
- **Object store**: account-export artifacts (short-lived signed URLs), email templates.

### 2.2 Client/server split and render-pipeline boundary

The **render-pipeline boundary** is the snapshot contract. The server is the only place time passes; the client is the only place pixels and sound happen. Concretely:

- The simulation owns: personality vectors, moods, perch assignments, call-schedule decisions, weather schedule, greeting plans, notebook prose.
- The client owns: interpolation between snapshots, all animation timing, idle micro-motion jitter, ambient ornaments (leaves/feathers — explicitly *not* simulated server-side), call synthesis from the motif parameters the snapshot provides, the listen-in mix, captions, narration delivery.

The snapshot is intentionally a *declarative scene-state + parameterized-intent* payload, not a frame stream: positions, moods, current call motifs with scheduled timestamps, weather, lighting phase, and per-bird animation seeds. The client plays it forward and interpolates. This keeps snapshots small (kilobytes) and keeps all high-frequency motion client-side, which is what makes 60fps at a ~1/min tick possible.

### 2.3 Why the tick is server-side (non-negotiable)

Per `accounts_sync.md`: the tick runs whether or not any client is connected. This is what makes "the aviary continues without the viewer" true, makes multi-device sync a non-feature (both clients read one canonical record), and makes personality corruption by concurrent clients structurally impossible. Clients never tick, never write personality state, never send absolute trait values. There is no client-side simulation fallback; a client with no connectivity shows the last snapshot it has and a quiet reconnecting state, and does not advance anything locally **[call: on reconnect after long offline, the client re-pulls and cross-fades into the new state rather than animating a catch-up]**.

### 2.4 Service topology

- Snapshot delivery: HTTP GET, cached at CDN edge with a short TTL (seconds) keyed by aviary ID + snapshot version; a version-tagged long-poll or SSE channel for low-frequency live updates while visible. WebSocket is *not* required at this cadence **[call: SSE chosen over WS for simplicity; cadence is ~1 update/min]**.
- Event ingestion: HTTP POST to an ingestion endpoint that validates session, stamps server receipt time, and appends to the log. Idempotency keys per event make client retries safe.
- Tick worker pool: each aviary is processed by exactly one worker at a time via a per-aviary advisory lock keyed on account UUID; workers claim due aviaries from a schedule table.

---

## 3. Data model

All internal IDs are UUIDs. Email appears only on the account record (encrypted at rest) and in the invite/email-sending path. Nothing else — no log key, no telemetry dimension, no shard key — may contain email.

### 3.1 `account`

| field | notes |
|---|---|
| `id` | synthetic UUID, generated at creation; the only cross-service account reference |
| `email_encrypted` | encrypted at rest; decrypted only for sending mail |
| `email_verified_at`, `pending_email_encrypted` | email-change flow |
| `created_at` | drives aviary-age → new-bird offers |
| `deletion_requested_at` | null normally; set = soft-deleted; hard-delete job purges at +30d |
| `settings_json` | reduced-motion opt-in, captions opt-in, visit-notifications toggle (default off), audio muted |
| `notification_prefs` | currently only the visit-notification toggle; designed so nothing else can land here without a schema review |

### 3.2 `session`

| field | notes |
|---|---|
| `id`, `account_id`, `device_label` | per-device, revocable from settings |
| `created_at`, `last_seen_at`, `revoked_at` | |

### 3.3 `bird`

| field | notes |
|---|---|
| `id` | **stable internal identifier, never regenerated, never swapped**. Renames, syncs, migrations, species-pool changes never touch it |
| `account_id`, `species_id` | species from the v1 pool of ~6 |
| `name` | user-assigned, renameable any time; no engine effect |
| `adopted_at`, `adoption_order` | starter birds vs. later offers |
| `personality_vector` | 5 floats, normalized ranges (stored as a typed struct, exact ranges internal to the sim): boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity. **Server-written only, never exposed in any API, never recomputed from logs** |
| `mood_state` | enum {wary, content, curious, drowsy, alert} + `mood_entered_at` + small per-bird mood-context fields (e.g., last-offer-accepted-at) needed by the transition function |
| `current_perch_zone` | {front, middle, back} + fine position seed |
| `seed_profile` | adoption-time seed values for the vector and the bird's call-signature parameters (motif variant weights, base pitch class) — the source of per-bird call recognizability, constant over drift |

### 3.4 `presence_event` / interaction events (append-only log payloads)

Event types: `presence_ping` (start/heartbeat/stop with client-computed conjunction evidence: visibility, focus, last-input timestamp), `listen_in_start`, `listen_in_end`, `offer` (kind: seed|song_fragment|still_pool, target bird if any), `settle`, `settle_undo`, `adoption` (naming). Each event carries: account UUID, client event UUID (idempotency), client timestamp, server receipt timestamp, session ID.

**Presence validation is re-checked server-side** **[call: the PRD defines presence as a client-observable conjunction; the server trusts client-reported fields but rejects pings whose claimed active-window exceeds wall-clock since the previous ping — anti-inflation guard against buggy or hostile clients corrupting drift]**.

### 3.5 `notebook_entry`

| field | notes |
|---|---|
| `id`, `account_id`, `created_at` | ordered, immutable, infinite scroll-back |
| `prose` | naturalist voice, lowercase, present-tense |
| `source_context_json` | internal-only provenance (which events/moods motivated the entry) for tuning generation sparsity; never served to the client |

### 3.6 `invite`

| field | notes |
|---|---|
| `id`, `host_account_id`, `visitor_email_encrypted` | visitor email stored encrypted; used only to send the link |
| `token_hash` | one-time link token; stored hashed |
| `created_at`, `expires_at` (creation + 30d), `consumed_at`, `revoked_at` | revocation effective at next snapshot pull |

### 3.7 `visit` (log rows)

`id, invite_id, started_at, ended_at, approx_duration_seconds` — displayed in the host's visit log. **No presence/drift side effects.** Visitors never write interaction events; the ingestion endpoint rejects event writes on visit-scoped tokens.

### 3.8 Derived/runtime (not persisted as canonical)

Snapshot cache entries; per-call audio buffers (client, reused); ambient ornament state (client-only); export artifacts (short-lived).

---

## 4. API surface

All endpoints under `/v1`. Authenticated via per-device session bearer token. Visit links mint visit-scoped read tokens. Error responses use matter-of-fact copy.

### 4.1 Auth & account

- `POST /auth/magic-link` — body: email. Sends 15-minute, single-use link. Per-email rate-limited. Response is identical whether or not the email exists (no account enumeration) **[call]**.
- `POST /auth/consume` — body: token. Validates, invalidates token immediately, issues session token + account bootstrap payload.
- `GET /account` — settings, sessions list, aviary age, invite list summary.
- `POST /account/email-change` — starts verification of new address; old email works until verify.
- `POST /account/email-change/confirm`.
- `POST /account/sessions/{id}/revoke`.
- `POST /account/export` — enqueues export generation; emails a short-lived signed download link to the verified address. Payload: birds (names, species, personality vectors, moods), notebook entries, settings. (Export is the *only* surface where the personality vector leaves the server — it is the user's own data per the PRD's export spec; it is never rendered in-product.)
- `POST /account/delete` — soft-delete with 30-day recovery. Signing in during the window surfaces the "I changed my mind" restore path (`POST /account/restore`).
- `PATCH /account/settings` — reduced motion, captions, audio muted, visit-notifications toggle.

### 4.2 State consumption

- `GET /aviary/snapshot?since_version=N` — returns the canonical snapshot: version, server time, per-bird {id, name, species, perch zone + fine position seed, mood, plumage-render params, call-signature params, scheduled call motifs with timestamps}, weather state, lighting phase (server-computed from the account's timezone), active greeting plan if any, current offer-cooldown states. Kilobyte-scale. CDN-cacheable with per-version keys; personalization is per-account so caching is private (Authorization-keyed, `Cache-Control: private`) **[call]**.
- `GET /aviary/stream` (SSE) — version-tagged snapshot diffs at tick cadence while visible; client falls back to polling on `visibilitychange`, after long frame gaps (suspend/resume), and on a low-frequency keepalive.
- `GET /notebook?cursor=…` — paginated entries, oldest-to-newest within pages; read-only; no write endpoint exists at all.

### 4.3 Event submission

- `POST /events` — batch of interaction events (presence pings, listen-in start/end, offers, settle, settle-undo, adoptions). Server validates session scope, dedupes on client event UUID, appends to the log in receipt order, returns accepted count. No synchronous simulation effect is promised; the client renders immediate *local* reaction affordances (e.g., offer animation starts) while the authoritative reaction arrives in the next snapshot **[call: optimistic local reaction rendering with server reconciliation; reactions are cosmetic, state is not]**.

### 4.4 Visits

- `POST /invites` — body: visitor email. Creates invite, emails one-time link. Per-account rate limit **[call: e.g., 5/day]**, cap on concurrent outstanding invites **[call: e.g., 20]**.
- `DELETE /invites/{id}` — revoke. Immediate: the visit token's next snapshot request returns the matter-of-fact "visit no longer available" surface.
- `POST /visits/consume` — body: link token. Marks invite consumed, issues a visit-scoped read token (valid until revoked; bound to the invite).
- `GET /visits/snapshot` — same shape as the owner snapshot, minus anything account-private (no settings, no offer cooldowns, no notebook access). Visitor client runs in a render-only mode that never calls `POST /events` and receives no greeting plan (no return-greeting for visitors — greetings are for the owner; the visitor sees the aviary as ambient).
- `GET /account/visit-log` — visitor email, date, approximate duration, outstanding invites. No badge, no push.

---

## 5. Simulation engine design

### 5.1 The tick

- **Cadence**: ~once per minute per aviary, calibrated during build; exact value is a config knob. The tick also runs opportunistically *before* serving a snapshot if the aviary is overdue (e.g., after a worker gap), so clients never see a stale-by-hours state as "current" **[call: lazy catch-up ticks computed deterministically from the event log and wall-clock, not by running 1440 sequential ticks — one catch-up pass over aggregated inputs]**.
- **Per-tick pipeline** (per aviary, single-writer under advisory lock):
  1. Read unprocessed events from the log (offset-tracked, in order).
  2. Validate and aggregate presence-time per session window.
  3. Update mood state machine (§5.3).
  4. Apply drift deltas (§5.2).
  5. Advance perch positions, call schedule, weather schedule, chorus emergence (§5.5).
  6. Maybe generate a notebook entry (§5.6).
  7. Write canonical state + new snapshot version; publish invalidation to the SSE channel; mark log offset.

### 5.2 Drift function

- Implemented as a **slow low-pass filter over presence-and-interaction signals**. Concretely: each trait holds an accumulator updated per tick as
  `trait ← trait + clamp(k_trait · input_signal · dt, 0, max_step)` — note the clamp floor of 0: **drift is monotonic toward expressive; there is no negative path**.
- **Input weights** (rough order per PRD): presence-time (dominant, applies gently across all traits toward expressive), listen-in (strong, targeted: social warmth + vocal frequency on the listened bird), offers (accepted offer → curiosity; offer near a bird → small boldness), settle (no directional drift; cleanly closes the presence window).
- **Offer cooldown** (per bird, a few minutes) is enforced at ingestion *and* in the tick's signal aggregation so mashing offers cannot saturate curiosity drift.
- **Calibration targets (testable)**: measurable drift in instruments after ~1 week of regular visits; user-visible drift after ~3 weeks. The calibration harness (§10.3) simulates synthetic presence patterns and asserts drift curves fall inside the target band. Absence produces **no negative drift** — only the absence of positive drift, which reads as "ambient quietness."
- Personality vectors are **persisted, never derived**: stored columns updated by the tick. The event log is an input, not a source of truth for recomputation. Backup/restore restores the vectors, full stop.

### 5.3 Mood state machine

- States: {wary, content, curious, drowsy, alert} (final set confirmed in implementation; these five are the v1 working set).
- Transition inputs per tick: recent-session interactions (accepted offer → content nudge), time-of-day in the account timezone (dusk → drowsy; early morning → alert), ambient events (rain → dampened vocal frequency; another bird's alarm call → nearby birds toward wary), and the bird's own personality (high-boldness birds resist wary on identical input).
- **Daily-ish reset**: a soft re-centering window once per local day rather than a hard midnight snap **[call: "daily-ish" = a decay back toward the bird's personality-anchored baseline over ~24h, so mood persists across sessions and never visibly snaps on tab open]**.
- Mood persists across sessions; the tick keeps moods moving while the user is away, so the returning user meets continuity, not a reset.

### 5.4 Return-greeting planner

- On detecting a session-start (first presence evidence after an absence window), the tick's next pass (or a fast-path greeting computation at snapshot time — **[call: greetings are computed at snapshot-request time from canonical state, not pre-planned, so they react to the true absence length]**) selects **one** greeter: weighted by boldness and current mood, with day-level stickiness so the same bird tends to greet first within a day but not forever.
- Greeting form is selected from absence length (minutes → glance up; hours → call and look; days → approach/longer call possibly answered by another bird), bird boldness, and mood, then **procedurally parameterized** (timing, pitch contour, motion variant) so it is never identical twice.
- If multiple birds qualify, their responses **stagger** by randomized small offsets — never a unison chorus on arrival.

### 5.5 Calls, chorus, bird-to-bird

- Each species ships a **call grammar**: a motif library (small set of motifs) plus combination/variation rules. Each bird has constant **signature parameters** (motif weights, base pitch class) drawn at adoption — this is what stays recognizable across mood and drift — while runtime parameters (tempo, loudness, elaboration) are shaped by mood and vocal_frequency.
- The tick maintains a **call schedule**: per-bird next-call windows influenced by vocal_frequency, time-of-day (evening quieting; the nightjar-like species stays active into late hours), weather (rain dampens), and bird-to-bird coupling (one bird's call raises nearby response probability; two+ high-vocal birds calling in overlapping windows produce emergent chorus events).
- The snapshot ships scheduled call *intents* (motif IDs + variation seeds + timestamps); the client synthesizes the actual audio (§8). Variation seeds mean the server and client agree on what was "said" without shipping audio — which is also what lets captions match the played call exactly.

### 5.6 Notebook generation

- A generator scans each tick's noteworthy deltas (greeting order changes, unusual quiet stretches, mood+weather coincidences, offer reactions) and scores them for note-worthiness with a **sparsity governor**: target ~1 entry per few days for a regularly-visited aviary, more on genuinely noteworthy moments, never one-per-session. Implemented as a rate-limited priority queue with a minimum inter-entry interval that tightens as entry rate rises.
- Entries are **prose templates + slot-filling over real state**, written in naturalist voice ("tuesday — pip greeted before wren today, first time this week."), never numeric, never about the user's behavior ("you visited every day" is a banned class — observations of the aviary only, never of the user).
- Read-only and immutable; infinite scroll-back; no archiving.

### 5.7 Weather and day/night

- Weather scheduler: rare, gentle events — short rain a few times a week, occasional wind. Per-aviary random schedule computed server-side (visitors see the same weather as the host). Effects are small, short-lived mood/vocal modulations.
- Lighting: server computes the lighting phase from the account's timezone (morning warming, bright midday, warm evening, dim night); clients render it. Night is not a dead state — most birds settle; the nightjar-signature species remains active.

---

## 6. Sync model

- **One canonical aviary per account, server-owned.** Multi-device sync is a property of the architecture: laptop and phone both pull the same snapshots from the same record. There is no client-to-client sync, no client-side canonical state, nothing to merge.
- **No last-write-wins, ever.** Clients submit events; the tick computes additive deltas in log order; only the tick writes personality vectors. No code path exists for a client to mutate personality. This makes the concurrent-session corruption case (laptop morning write overwritten by stale phone lunch write) structurally unreachable rather than handled.
- **Conflict prevention vs. resolution**: because there is a single writer, "conflicts" reduce to operational errors (expired magic link, timed-out session, transient outage). Those surfaces are matter-of-fact ("We couldn't sign you in. The link may have expired. Try requesting a new link.") — never naturalist.
- **Event ordering**: log receipt order is authoritative; client timestamps are advisory metadata. Late-arriving events (client retried after a network gap) are still valid inputs — drift doesn't care about minute-level skew.
- **Session revocation**: revoking a device session invalidates its token at the auth layer; its in-flight events already in the log remain processed (they were the user's real attention).
- **Visit revocation**: effective at the visitor's next snapshot pull; the visit token is checked against invite state on every request.

---

## 7. Frontend rendering pipeline

### 7.1 Stack and first paint

- TypeScript SPA. Rendering: **Canvas 2D or WebGL via a thin scene graph [call: WebGL with a Canvas2D fallback only if WebGL context creation fails; the budget math below assumes a single batched renderer]**. Bird art: procedurally assembled sprites from compact SVG/vector parts with personality-parameterized plumage saturation. Code-split aggressively: the aviary scene is the critical path; account settings, accessibility settings, invite flow, and notebook are lazy chunks.
- **First-frame rule**: the HTML shell carries an inline bootstrap fetch of the current snapshot from a CDN edge (small payload delivered with/behind the HTML). The renderer draws birds mid-action — mid-preen, mid-call, leaf mid-drift — from the snapshot's animation seeds and server timestamps (phase-aligned so motion looks continuous, not restarted). **No spinner, no fade-from-static, no entry animation.** The slow-connection load state is the *quiet field*: soft sky color, one or two faint motion cues. The empty-aviary state (post-adoption, pre-first-bird) is the same quiet field; the first bird enters with a soft fly-in; the user never sees an empty aviary again.
- **Budget enforcement**: bundle-size gate in CI (<2MB gzipped initial); first-bird-visible measured in the synthetic perf fleet (<500ms on the reference mid-tier-mobile-over-4G profile).

### 7.2 Scene composition

- Single horizontal scene, no pan/scroll/zoom. Three perch zones (front/middle/back) with fine positions from snapshot seeds. Responsive: narrow viewports compress horizontal spacing without ever cropping a bird; wide viewports add inter-perch space. All birds always in frame.
- Layering: soft background foliage/sky → middle plane (birds, perches) → occasional foreground branch/leaf. Subtle parallax only.
- Lighting: day/night palette interpolation from the snapshot's lighting phase; settle triggers a slow (few seconds) evening shift with a 5-second any-click undo.
- Top bar: account/settings, accessibility settings, notebook, offers — nothing else. Fades to near-transparent after a few seconds of cursor stillness; returns on cursor/keyboard activity. No chrome inside the scene: no tooltips, badges, overlays, or labels over the birds.
- Ambient ornaments: leaves/feathers drift at slow random intervals, **client-generated, not simulated** — pure rendering ornaments with no server state.

### 7.3 Idle micro-motion and transitions

- Continuous mood-shaped idle: preening, scanning, head-tilts, weight-shuffles — parameterized by mood (wary → back perch, more scanning; content → preening; curious → sound-oriented tilts; drowsy → low, fluffed). Motion runs from snapshot seeds + local jitter so it never loops identically.
- Snapshot-to-snapshot interpolation: positions, moods (as motion parameter blends), and lighting interpolate smoothly — a bird moving perch A → B glides, never teleports.
- **Pause discipline**: hidden/backgrounded tabs stop rendering (battery) while the server keeps ticking; on return, the client re-pulls and cross-fades into the current state.

### 7.4 Reduced-motion mode

A designed surface, not a fallback: micro-motion becomes slow cross-fades between still poses (preen pose → preen pose); perch transitions become cross-fades rather than flight paths; ambient leaf drift is removed; day/evening color shifts remain, slowed. Audio, captions, narration, drift, mood, notebook: all fully intact. Triggered by `prefers-reduced-motion` or the accessibility-settings toggle; ships with v1, not after.

---

## 8. Audio pipeline

### 8.1 Procedural synthesis (only path)

- Calls are synthesized client-side via **WebAudio** from the species motif library + per-bird signature parameters + per-call variation seeds delivered in the snapshot. No recorded audio ships anywhere in the product, at any quality, on any path.
- Synthesis design: each motif is a parameterized oscillator/envelope/filter recipe (pitch contour, formant-ish filtering, amplitude envelope, noise bed for breathiness). The runtime combines motifs per the grammar with the call's variation seed; mood shapes tempo/loudness/elaboration; the signature parameters keep each bird identifiable.
- **Chorus**: simultaneous calls are genuinely independent synthesized voices in one mix — this is why the whole pipeline is procedural; stacked loops phase-cancel and read as dead.
- Audio buffers are pre-allocated and reused; no per-call allocation (memory-growth CI test covers this).

### 8.2 The mix

- Ambient chorus mix by default: per-bird levels by perch zone (front louder) and mood.
- **Listen-in**: focused bird's level rises slowly; others drop slowly to ambient — a re-balance, never a mute, with matched slow ramps on engage and disengage. Disengage triggers: re-click focused bird, focus another bird, click empty scene, move keyboard focus away.
- **Settle**: calls quiet over the same few-seconds window as the lighting shift.
- Mute setting persists in account settings (and is itself a drift-neutral action — **[call: muting is not a drift input; the PRD lists presence, listen-in, offers, settle as the inputs and mute is not among them]**).

### 8.3 Fallback

WebAudio unavailable/denied → graceful silence with **captions on by default**. No recorded-audio fallback exists. This is unconditional.

### 8.4 Captions

Generated **at runtime from the same call-grammar parameters that produced the sound** ("a soft three-note rise", "a low trill, paused, low trill again"), rendered as small naturalist-voice text near the calling bird, fading with the call. Opt-in via accessibility settings (default-on only in the no-WebAudio fallback). Caption text is derived from motif shape (note count, contour, register, tempo), so the caption always matches what was actually played — or would have played.

---

## 9. Accessibility surfaces

Planned in from day one; ships with v1; treated as designed surfaces with their own charm, not parity checklists.

- **Screen-reader narration**: running naturalist prose generated from the same canonical state the visuals render ("a small grey bird is perched on the front rail, calling softly…"). Delivered via a visually-hidden live region (polite). Cadence: ~1 update per 30–60s at idle; user-initiated events (return-greeting, offer reaction, settle) get a priority bump but are still phrased as observations, never state transitions ("Pip mood: content" is a banned pattern). Same voice as the notebook — one product voice across surfaces.
- **Reduced-motion**: see §7.4.
- **Captions**: see §8.4.
- **Keyboard**: Tab through top bar; Tab into scene focuses first bird; arrow keys move between birds; Enter = listen-in; Escape = exit listen-in; offer affordance and settle fully keyboard-navigable from the top bar. Focus indicators: soft high-contrast outline visible against both bright and dim scene states (exact treatment from the design system).
- **Contrast**: all user copy (top bar, settings, errors, captions, visual narration) passes WCAG AA minimum; the scene itself carries no user copy outside the top bar.
- **Settings surface**: matter-of-fact voice; reduced-motion toggle, captions toggle, audio mute.

---

## 10. Performance budgets and observability

### 10.1 Budgets (CI-enforced gates, not guidelines)

| Budget | Gate |
|---|---|
| Initial JS bundle < 2MB gzipped | build-time size check, hard fail |
| Time-to-first-bird < 500ms (mid-tier mobile, 4G profile) | synthetic fleet, release gate |
| 60fps idle motion, sustained 30 min, 5-year-old laptop profile | synthetic fleet soak run |
| Zero memory growth over 30 min | CI soak test: heap snapshots at 5/15/30 min, flat-line assertion; covers audio buffer reuse, notebook scroll retention, worker/context bounds |
| Simulation tick p99 < 5s | production alarm |

### 10.2 What we measure

- **Synthetic checks**: automated browser fleet on a schedule from common geographies — load timings, first-bird-render timings, frame timings.
- **Aggregate RUM only**: page-load timings, first-bird timings, render-frame timing histograms, audio-context error counts, simulation-tick latencies, request counts/error rates, anonymized session-duration histograms (no per-account dimension).

### 10.3 What we deliberately don't measure

No per-bird state, no per-account interaction history, no drift curves over real users, nothing reconstructable into a user's relationship with their aviary. The telemetry pipeline is **architecturally separated**: it never reads the simulation database; the simulation database is never read by any analytics store. Per-bird interaction events exist only to drive that account's own simulation — never aggregated, never trained on, never shared, never population-analyzed. The one exception to "we don't see drift curves" is the **calibration harness**, which runs *synthetic* accounts with scripted presence patterns to verify drift lands in the 1-week-measurable / 3-weeks-visible band — synthetic data only.

### 10.4 Privacy mechanics (engineering rules, not policy text)

- Synthetic account UUID everywhere except the single encrypted email column.
- Telemetry schemas are reviewed against a denylist of per-account/per-bird fields at the metric-definition layer.
- Hard delete after the 30-day soft window purges birds, vectors, notebook, events, visit logs — everything keyed to the account UUID.
- Export path is the only bulk read of an account's own state and is delivered to the verified email via short-lived signed URL.

---

## 11. Rollout

### 11.1 Build order (dependency-sorted)

1. **Foundations**: account model + magic-link auth + sessions; canonical schema; event log + ingestion; snapshot contract + delivery (this is the spine — everything hangs off it).
2. **Simulation core**: tick worker, mood machine, drift function + calibration harness, call scheduler, weather scheduler, greeting planner, notebook generator.
3. **Client core**: renderer with first-frame-mid-motion boot, snapshot interpolation, idle motion system, top bar, presence collector, event emitter.
4. **Audio**: synthesis engine, mix, listen-in, captions, WebAudio fallback.
5. **Interactions end-to-end**: offers (+cooldowns), settle (+undo), adoption flow (system-selected starters, naming, empty-aviary → fly-in).
6. **Notebook surface** (read client).
7. **Accessibility surfaces**: narration, reduced-motion, keyboard map, contrast pass.
8. **Visits**: invites, visit tokens, render-only client mode, visit log, revocation surface.
9. **Account lifecycle**: export, soft/hard delete, email change, session revocation.
10. **Hardening**: perf soak, memory CI, privacy-schema audit, copy-voice lint, error-surface copy pass.

### 11.2 Ship strategy

- **Private alpha** (team + design partner accounts): full v1 surface. Instrument everything from day one: synthetic perf fleet, tick-latency alarms, bundle/memory CI gates, drift calibration harness on synthetic accounts. No feature flags that change the product's shape; flags only for operational kill-switches (e.g., disable SSE, disable weather) **[call]**.
- **Limited beta** (invited external users): primary goal is **drift calibration validation over real weeks** — since visible drift is a 3-week phenomenon, beta runs a minimum of 4–6 weeks before any launch decision. Audio uncanniness review (do calls read as alive on laptop speakers, phone speakers, headphones?) is a named beta exit criterion alongside perf budgets.
- **GA**: no waitlist mechanic, no launch-day marketing surface in-product; the product has no announcement surfaces and launch changes nothing about that.

### 11.3 Birds-per-aviary ramp

New-bird offers are tied to **aviary age**, not engagement: ~a third bird around a few months, growing toward five or six around a year, hard cap at seven. The offer appears quietly in the user's flow (an adoption moment, not a reward screen — no "you've earned a new bird!" framing; the bird simply arrives as an option). The age schedule lives in config so we can tune pacing without a migration. The seven-cap is enforced in the engine, and any future reconsideration requires audio-mix work proving recognizability above seven — the cap is empirical, not a plan tier.

---

## 12. Risks

### 12.1 Drift calibration (highest product risk)

- **Failure mode**: too fast → clickable Tamagotchi; too slow → screensaver; lax presence signal → population-wide drift inflation (silent — no test catches "tab open counted as presence" unless we test the conjunction explicitly).
- **Mitigations**: presence conjunction (visible AND focused AND recent-input) enforced client-side with server-side sanity checks; calibration harness asserting the 1-week-measurable/3-weeks-visible band across synthetic behavior profiles (daily-watcher, weekend-only, bursty, long-absence-return); monotonic-clamp unit tests (no code path can apply negative drift); drift-rate config knobs so tuning doesn't require redeploys; beta phase explicitly long enough to observe real drift.

### 12.2 Sync correctness

- **Failure mode**: any client-side authority over state reintroduces last-write-wins silently; stale-snapshot rendering after suspend/resume makes the aviary feel frozen-then-jumping.
- **Mitigations**: single-writer invariant enforced by schema (no personality write path in the API service — only the sim service holds write credentials to those tables); event idempotency; snapshot versioning with client-side gap detection (visibilitychange, long frame gaps → re-pull); lazy catch-up tick so snapshots are never served stale-as-current.

### 12.3 Audio uncanniness

- **Failure mode**: procedural calls that read as synthesized beeps kill the affective spine; chorus phase artifacts; listen-in ramps that feel like a mixer UI; per-bird signatures that blur above a few birds.
- **Mitigations**: dedicated audio-design iteration with motif-library listening reviews before beta; recognizability listening tests (can a listener pick Pip from Wren after exposure?) as a named beta gate; chorus stress-test at 7 simultaneous voices; ramp curves tuned against "switching channels" feel; strict no-recorded-audio rule held even if synthesis is hard — the fallback is silence+captions, never canned loops.

### 12.4 Accessibility regressions

- **Failure mode**: narration degenerating into state-list automation; reduced-motion shipping as "animations off"; captions diverging from played audio; keyboard focus lost after a scene re-render.
- **Mitigations**: narration copy reviewed against the voice contract like notebook prose; reduced-motion built as its own render path with its own visual review, in the v1 critical path (not post-launch); captions generated from grammar parameters by construction (they cannot diverge); focus-restoration tests across snapshot updates; axe-style automated checks in CI plus manual screen-reader passes per release.

### 12.5 Privacy boundary erosion

- **Failure mode**: per-account interaction data leaking into telemetry "for debugging"; email-as-identifier creeping into logs; a future aggregate dashboard built on the sim DB.
- **Mitigations**: architectural separation (telemetry can't reach the sim DB); UUID-only keys enforced by code review + log-scrubbing CI check for email patterns; metric-definition denylist; hard-delete job tested in CI (delete an account, assert zero residual rows across all stores).

### 12.6 Scope/spec-creep against the non-goals

- **Failure mode**: the well-meaning "small toast saying hi," the "harmless" streak counter, the engagement notification. Each is a one-line change that rotates the product into a different product.
- **Mitigations**: the banned-pattern list (welcome toasts, visit-frequency surfaces, user-behavior notebook entries, announcement-style UI, gamification vocabulary) encoded as a copy/UI review checklist and partially as lint rules; non-goals quoted in the PR template.

### 12.7 Tick reliability and cost

- **Failure mode**: tick backlog → aviaries that feel hours stale; hot accounts monopolizing workers; cost of ticking millions of idle aviaries every minute.
- **Mitigations**: p99 5s tick-latency alarm; per-aviary advisory locks with worker rebalancing; adaptive cadence (idle aviaries with no recent events tick less often, with deterministic catch-up on next snapshot request — **the user-visible state is identical; only compute scheduling changes [call]**); catch-up path load-tested for long-absence returns.

### 12.8 Greeting/notebook sameness over time

- **Failure mode**: procedural variation that is secretly "three variants in rotation"; notebook templates that repeat visibly within weeks.
- **Mitigations**: variation is parameterized (continuous timing/pitch/motion spaces), not variant-selected; template pools sized so naive repetition is rare, plus a per-account recent-entry dedupe check in the generator; sparsity governor doubles as a sameness guard (fewer entries, more noteworthy each).

---

## 13. Explicit defensible calls (index)

Decisions made where the PRD leaves latitude, collected for review:

1. SSE (not WebSocket) for live snapshot diffs at ~1/min cadence.
2. Offline client shows last snapshot + quiet reconnecting state; no local simulation; cross-fade catch-up on reconnect.
3. Server-side anti-inflation sanity checks on presence pings.
4. Optimistic local rendering of offer reactions, reconciled by next snapshot.
5. Magic-link request responses are enumeration-safe.
6. Owner snapshots are private-cached (not shared CDN-cached across users).
7. Greetings computed at snapshot-request time from canonical state.
8. "Daily-ish" mood reset implemented as ~24h decay toward personality-anchored baseline, not a hard snap.
9. Muting audio is not a drift input.
10. Invite rate limits (~5/day) and outstanding-invite cap (~20).
11. Adaptive tick cadence for idle aviaries with deterministic catch-up.
12. Feature flags restricted to operational kill-switches.
13. Copy-voice lint in CI as backstop to human review.
