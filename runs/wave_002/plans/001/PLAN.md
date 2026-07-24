# Pocket Aviary — Phase 1 Implementation Plan

## 1. Scope

### 1.1 In scope (v1)

Pocket Aviary is a browser-only, single-account virtual aviary: one canonical aviary per user, two starter birds (cap seven), magic-link auth, multi-device sync via server-owned simulation, field notebook, presence-driven monotonic personality drift, listen-in / offer / settle interactions, optional read-only visit invites, and first-class accessibility (naturalist screen-reader narration, reduced-motion surface, call captions).

Concrete deliverables:

| Area | v1 commitment |
|------|----------------|
| Platform | Modern web (Chrome, Safari, Firefox, Edge — last two major) |
| Auth | Email magic link, per-device revocable sessions |
| Aviary | One horizontal scene, 3 perch zones, day/night local-time, rare weather |
| Birds | ~6 species pool; 2 at start; age-gated adoption up to 7; stable bird IDs |
| Engine | Server tick ~1/min; personality vector + mood; procedural calls client-side |
| Interactions | Return-greeting, presence, listen-in, offer (seed/song/pool), settle, notebook |
| Social | Per-invite email visits, read-only, off by default, revocable |
| A11y | Narration, reduced-motion, captions, keyboard, WCAG AA chrome |
| Perf | JS bundle <2MB gzip; TTFB bird <500ms; 60fps idle; no mem growth 30m |
| Data rights | JSON export; soft delete 30d then hard |

### 1.2 Out of scope (respect non-goals)

- Native iOS/Android apps; password/SSO deferred
- Gamification: streaks, badges, XP, calendars, counters, ranks
- Tamagotchi: death, hunger, distress, decaying meters, negative drift
- Social network: profiles, follows, public discovery, chat, comments, leaderboards, co-presence
- Payments, multi-aviary accounts, customizable scenes, push about aviary state
- Last-write-wins personality merges; client-owned sim; recorded-audio fallback
- Exposing personality numbers anywhere

**Ambiguity calls (defensible defaults):**

1. Presence activity window: **4 minutes** without pointer/key before presence ends (favors “watch without moving”).
2. Tick cadence: **60s** nominal; adaptive backlog drain if lag.
3. Trait range: **[0.0, 1.0]** floats; seed with species priors + small per-bird jitter.
4. Mood enum: `wary | content | curious | drowsy | alert | settled` (settled for night/settle).
5. Notebook: max ~**2 entries/week** baseline + event-triggered; LLM or template engine with strict naturalist lock — v1 prefers **template+slot assembler** with curated phrase banks for predictability and privacy.
6. Bird-3 timing: offer at aviary age **~45–60 days**; subsequent slots ~every 60–90 days; never visit-count gated.
7. Presence pings: client emits presence samples every **30s** while triple-condition holds; server aggregates duration.

---

## 2. Architecture

### 2.1 Service shape

```
┌─────────────────────────────────────────────────────────────┐
│ Browser SPA (Vite + TypeScript)                             │
│  Render (Canvas/WebGL2 or 2D) │ WebAudio │ A11y bus │ Events│
└───────────────┬───────────────────────────┬─────────────────┘
                │ HTTPS / WSS optional      │
                ▼                           ▼
┌──────────────────────────┐   ┌──────────────────────────────┐
│ API Gateway / BFF        │   │ Auth service                 │
│ REST + optional SSE      │   │ Magic link, sessions, revoke │
└──────────┬───────────────┘   └──────────────┬───────────────┘
           │                                  │
           ▼                                  ▼
┌──────────────────────────┐   ┌──────────────────────────────┐
│ Simulation service       │   │ Account / identity store     │
│ Tick worker, event log   │   │ UUID id, email encrypted     │
│ Snapshot builder         │   └──────────────────────────────┘
└──────────┬───────────────┘
           ▼
┌──────────────────────────┐   ┌──────────────────────────────┐
│ Primary DB (Postgres)    │   │ Object/CDN (static assets)   │
│ accounts, birds, events, │   │ SPA, edge bootstrap HTML     │
│ snapshots cache, visits  │   └──────────────────────────────┘
└──────────────────────────┘
┌──────────────────────────┐
│ Ops telemetry only       │
│ (no per-bird warehouse)  │
└──────────────────────────┘
```

**Principles:**

- **Single writer for personality/mood canonical state:** simulation tick only.
- **Clients are render + eventsources:** pull snapshots; append interaction events.
- **Synthetic account UUID** everywhere except one encrypted email column.
- **Simulation DB isolated** from analytics/warehouse connectors (network + IAM deny).

### 2.2 Client/server split

| Responsibility | Client | Server |
|----------------|--------|--------|
| Personality vector | read via snapshot only | store + drift apply |
| Mood | interpolate display | transition on tick + event effects |
| Positions / idle phase | interpolate, ornament leaves | base perch targets, intent flags |
| Calls | synthesize WebAudio from grammar params | emit call schedule seeds / motif params in snapshot |
| Presence | measure triple condition; send pings | accumulate presence-time for next tick |
| Notebook entries | display | generate & store |
| Visit | read-only snapshot with visit token | host invite CRUD; strip write APIs |

### 2.3 Render pipeline boundary

1. **Bootstrap HTML** (CDN edge) includes minimal CSS quiet-field + hydration stub.
2. **State snapshot** fetched ASAP (cookie/session or visit token).
3. **Scene graph** builds birds at current pose indices/phases from server motion seeds; starts requestAnimationFrame immediately — **no intro animation**.
4. **Interpolation layer** maps discrete perch/mood targets to continuous visual state.
5. **Ornament layer** client-only leaves/feathers (not sim).
6. **Chrome layer** top bar outside scene; fades on idle.
7. **Hidden tab:** cancel rAF, suspend AudioContext; keep presence off; re-pull snapshot on `visibilitychange` → visible.

---

## 3. Data model

### 3.1 Core tables (logical)

**Account**

- `id` UUID PK (synthetic)
- `email_ciphertext`, `email_hash` (lookup only)
- `created_at`, `deleted_at` (null | soft)
- `settings` JSONB (a11y prefs, visit-notify opt-in default false, locale/tz)
- `aviary_created_at` (age for adoption unlocks)

**Session**

- `id`, `account_id`, `device_label`, `token_hash`, `created_at`, `revoked_at`, `last_seen_at`

**Aviary** (1:1 account)

- `account_id` PK/FK
- `settled_until` nullable
- `weather_state` enum + ends_at
- `last_tick_at`, `tick_version` (monotonic)

**Bird**

- `id` UUID stable
- `aviary_id`
- `species_id`
- `display_name`
- `adopted_at`
- `personality` JSONB `{ boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity }` — never sent to non-owner export-only with export feature; **not in normal client API**
- `mood` enum
- `mood_updated_at`
- `perch_zone` `front|middle|back`
- `motion_seed` u64
- `call_signature_params` JSONB (species motif roots + unique jitter)

**Note:** Client snapshots include **derived display params** (effective call rate band, boldness-influenced perch preference, plumage saturation as rendered color factor) but **do not include raw trait dictionary** in debug or UI forms. Export API is the only place full vectors appear, by user request.

**InteractionEvent** (append-only)

- `id` ULID/time-sortable
- `account_id`, `bird_id` nullable
- `type` enum: `presence_sample | listen_in_start | listen_in_end | offer | settle | settle_undo | rename | session_open | session_close`
- `payload` JSONB (offer kind, duration_ms, client_ts, focus state)
- `server_ts`
- `device_session_id`

**PresenceAggregate** (optional rollup)

- daily/hourly presence seconds per account — inputs to drift; not user-visible

**NotebookEntry**

- `id`, `account_id`, `created_at`, `prose` text, `trigger` enum

**VisitInvite**

- `id`, `host_account_id`, `visitor_email_hash`, `token_hash`, `created_at`, `expires_at`, `revoked_at`, `accepted_at`

**VisitSession / VisitLog**

- visitor token sessions, duration estimates, for host log only

**MagicLink**

- token hash, account, expires 15m, consumed_at

### 3.2 Snapshot DTO (client-facing)

```ts
type AviarySnapshot = {
  version: number;
  serverTime: string;
  localDayPhase: { solarFraction: number; label: "morning"|"midday"|"evening"|"night" };
  weather: { kind: "clear"|"rain"|"wind"; intensity: number; endsAt?: string } | null;
  settled: boolean;
  birds: Array<{
    id: string;
    name: string;
    speciesId: string;
    mood: Mood;
    perch: "front"|"middle"|"back";
    visual: { saturation: number; poseHint: string; phase: number };
    call: { grammarId: string; params: CallParams; nextWindow?: number };
  }>;
  ambient: { chorusBias: number };
  notebookPreviewCount?: number; // not content
  activeOffers?: { birdId: string; kind: OfferKind; cooldownUntil: string }[];
};
```

Personality raw values excluded from snapshot.

---

## 4. API surface

Base: `/api/v1`, JSON, session cookie or `Authorization: Bearer`.

### 4.1 Auth

| Method | Path | Notes |
|--------|------|-------|
| POST | `/auth/magic-link` | body `{ email }`; always generic 200 |
| GET | `/auth/callback?token=` | consume link; set session; redirect SPA |
| POST | `/auth/logout` | revoke current |
| GET | `/me` | account meta, settings |
| PATCH | `/me/settings` | a11y, visit notify |
| GET | `/me/sessions` | list devices |
| DELETE | `/me/sessions/:id` | revoke |
| POST | `/me/email-change` | start verify |
| POST | `/me/export` | queue JSON email link |
| POST | `/me/delete` | soft-delete |
| POST | `/me/delete/cancel` | within 30d |

Matter-of-fact errors only.

### 4.2 Aviary state & events

| Method | Path | Notes |
|--------|------|-------|
| GET | `/aviary/snapshot` | canonical state |
| GET | `/aviary/snapshot/stream` | optional SSE/tick push every ~30–60s while connected |
| POST | `/aviary/events` | batch append events; validate schema; **no personality fields accepted** |
| POST | `/birds/:id/rename` | name only |
| GET | `/notebook` | paginated entries oldest→newest support |
| GET | `/adoption/status` | whether new bird available by age |
| POST | `/adoption/accept` | if unlocked; system assigns species; client sends names |

### 4.3 Visit flow

| Method | Path | Auth |
|--------|------|------|
| POST | `/visits/invites` | host; `{ email }` |
| GET | `/visits/invites` | host list outstanding |
| DELETE | `/visits/invites/:id` | revoke immediate |
| GET | `/visits/log` | host |
| GET | `/visit/:token/snapshot` | visitor; read-only; **events API 403** |

Visitor SPA route loads token; no offer/listen-in/settle/write notebook. Revoked → matter-of-fact “visit no longer available.”

### 4.4 Event payload rules

- Reject absolute personality/mood writes.
- Idempotency-Key header for retries.
- Server clamps offer cooldowns (~3–5 min per bird per offer kind).
- Presence samples must include client flags for visibility/focus/activity; server may still re-check nothing client-side-only — trust with rate limits + sane caps (e.g. max 12 presence hours/day contribution).

---

## 5. Simulation engine design

### 5.1 Tick worker

- Schedule: every **60s** per aviary (sharded by account UUID).
- Steps:
  1. Load aviary + birds + unreprocessed events since `last_event_cursor`.
  2. Fold presence samples → presence seconds since last tick.
  3. Apply **drift deltas** (section 5.2).
  4. Apply mood transitions (5.3).
  5. Update perch targets from mood × boldness softmax.
  6. Advance weather RNG (rare rain/wind).
  7. Bird-to-bird: alarm/wary spread, chorus window flags.
  8. Maybe generate notebook entry (sparsity controller).
  9. Write new state + `tick_version++`; advance cursor.
  10. Emit metrics: tick latency, queue lag (aggregate only).

Offline users: tick still runs using last presence/interaction only — **no invented presence**.

### 5.2 Drift function

Traits `t ∈ [0,1]`. Monotonic **up only** on positive signals; neglect → **no downward trait**, only lower greeting probability via **ambient expression curve** (separate from stored traits).

Weighted PPE (presence preference energy) per tick:

```
Δ = clamp(
  w_p * f(presence_sec) +
  w_l * listen_in_seconds_on_bird +
  w_o * offer_accept_signal +
  w_ob * offer_near_boldness_signal
, 0, Δ_max_per_tick)
```

Suggested weights (calibrate):

- Presence dominates (~60–70% of positive mass).
- Listen-in strong on **social_warmth** + **vocal_frequency** for focused bird.
- Offer accept → **curiosity**; offer proximity → **boldness**.
- Settle: ends presence cleanly; **Δ = 0** for traits.

Low-pass: `trait := trait + α * Δ * (1 - trait)` so asyptote at 1 without overshoot drama.

**Calibration targets:**

- Instrument: after **7 days** regular presence (~20–40 min/day), mean trait delta detectable (e.g. ≥0.02 on at least one trait) in offline replay harness.
- User-visible expression: ~**3 weeks** — greet/perch/plumage shifts.
- Single session: trait deltas below perceptual threshold.

**Ambient quietness without negative drift:** greeting rate = `g(boldness, social_warmth, presence_recency_hours)`. Long absence lowers short-term greeting intensity via recency, **not** by lowering stored traits.

### 5.3 Mood transitions

Markov-like with continuous biases:

Inputs: recent offers/listen-ins, local hour, weather, neighbors’ moods, personality.

Examples:

- Dust → bias **drowsy/settled**
- Early morning → **alert/curious**
- Rain → temp −vocal expression, slight **wary/content** mix
- High boldness resists **wary**
- Offer accept → **content**
- Alarm call nearby → **wary** window

Persist across sessions; tick continues transitions while user away.

### 5.4 Call-grammar runtime

- Per species: motif lexicon (pitch envelopes, rhythm cells, ornament probability).
- Per bird: stable jitter seed in `call_signature_params`.
- Mood gates tempo/amplitude; vocal_frequency gates inter-call intervals and chorus join probability.
- Server snapshot may include upcoming Poisson windows; **client synthesizes samples** each call.
- Recognizability > variety; limit simultaneous full-level callers; listen-in rebalance stays mix, not mute.

### 5.5 Return-greeting logic (session_open)

On first `session_open` after absence:

1. Compute absence length buckets: <5m glance; 5m–6h small call; 6h–48h reorient; >48h longer call / approach if bold.
2. Select greeter: argmax soft score(boldness, social_warmth, mood).
3. Stagger secondary birds random 200–1200ms if they also react — never unison fanfare.
4. Procedural variant seeds unique per online event id.

No toast, no “welcome back,” no days-gone text.

### 5.6 Offers

Kinds: `seed | song_fragment | still_pool`.

Reaction chooser uses mood × curiosity; animation intent flags in next snapshots. Cooldown 3–5 min/bird. Song fragment plays soft motif; bird join/quiet/counter from vocal_frequency.

### 5.7 Settle

Server records settle event; snapshot `settled=true`, lighting evening, quieter call schedule. Client undo within 5s sends `settle_undo`. Tab close ≡ presence end without penalty.

### 5.8 Adoption

Starters: server assigns 2 species on account create (diverse pair). Unlock third+ by `now - aviary_created_at` thresholds only. No catalog browse; “birds that arrived.”

### 5.9 Notebook generator

Sparsity: token bucket ~1 entry / 2–4 days + boosters for rare events (first reverse greeter order, long quiet morning, weather + behavior).

Templates slot bird names, relative facts from **derived observable facts** (greeter order, perch, fluffed, call density) — never raw trait numbers, never “you visited N days.”

---

## 6. Sync model

1. Server authoritative; tick applies ordered event log → additive trait deltas only.
2. All devices GET same snapshot version.
3. On visibility restore / long rAF gap / low-frequency poll (e.g. 45s): refetch snapshot; snap interpolate from new targets without teleports if possible.
4. Conflicts: no LWW personality. Concurrent sessions append events; tick serializes by event id time.
5. If snapshot version gap huge: hard re-base poses (science over jitters) quietly in field.
6. Magic-link curriculum / timeout: matter-of-fact copy only.
7. Visits: separate read path; **no presence_sample accepted** under visit token; visitor attention cannot drift host birds.

---

## 7. Frontend rendering pipeline

### 7.1 Stack recommendation

- TypeScript SPA, Vite, route code-split settings/visits.
- Scene: **Canvas 2D or WebGL** with light-weight sprite/mesh birds; prefer procedural + compact SVG atlases under bundle budget.
- State store: snapshot + local interpolation clock; event outbox with retry.

### 7.2 Scene composition

- Layers: sky, far foliage, mid perches/birds, near branch occluders, caption layer, no chrome inside.
- Three perch anchors responsive width; **never crop birds**.
- Day/night from client local tz applied to palette (server also uses account tz for mood).
- Loading: quiet field (soft sky); **not spinner**. Mid-session fetch failures: soft retry without announcing write toast spam.

### 7.3 Idle micro-motion

Mood-shaped loops: wary scan back, content preen, curious tilt, drowsy fluff, alert head-check. Continuous cage-feel without pause when simulated; rAF stops when hidden but engines server-side continue.

Ambient leaves client-only.

### 7.4 Top bar

Icons: account, a11y, notebook, offer (+ settle). Fade α→~0 after ~3s idle cursor/keys; restore on activity.

### 7.5 Reduced-motion

If `prefers-reduced-motion` or setting: replace continuous motion with **slow cross-fades between still poses**; keep color phase & calls/captions; remove leaf drift. Own aesthetic, not “broken static.”

### 7.6 First frame aliveness

Hydrate snapshot pose/phase so bird already mid-preen/call. Defeat default SPA blank-then-fade.

---

## 8. Audio pipeline

1. **WebAudio graph:** master → dry/wet light reverb → destination.
2. Per-bird gain nodes; ambient baseline gains sum <1.
3. **Listen-in:** 800–1500ms ramps: focus gain ↑, others ↓ to floor **>0**.
4. Call scheduler consume snapshot/Poisson; generate buffer from grammar (oscillators + noise + filtered envelopes); pool buffers; free after playback — **no unbounded alloc**.
5. Chorus: independent start times + motif variants.
6. Fallback: if AudioContext missing/denied → **silence + captions default on**; no recorded MP3 path.
7. Mute preference: user chrome; still counts presence; muting is interaction-ish but not negative drift.
8. Autoplay policies: unlock AudioContext on first pointer/key; until then visual-only + captions if enabled.

---

## 9. Accessibility surfaces

| Surface | Design |
|---------|--------|
| Screen reader | Live region with naturalist prose ~30–60s idle; faster for greeting/offer/settle; observation voice not ARIA state dumps |
| Captions | Runtime from same grammar (“a soft three-note rise”); near bird; fade with call |
| Keyboard | Tab chrome → birds; arrows between birds; Enter listen-in; Esc exit; offer/settle from bar |
| Focus ring | Soft high-contrast outline AA on dim/bright |
| Contrast | AA for all chrome/captions/settings |
| Settings | Matter-of-fact labels for reduced-motion, captions, narration rate |
| Visit / errors | Matter-of-fact only |

Ship a11y **with** v1 visual path, not as lagging retrofit.

Narration implementation I: client assembler from snapshot facts + same phrase banks as notebook for voice continuity; throttle SR queue.

---

## 10. Performance budgets and observability

### 10.1 Budgets

| Metric | Budget |
|--------|--------|
| Initial JS gzip | < 2MB |
| Time to first bird visible | < 500ms mid-tier 4G target |
| Idle | 60fps on ~5yo laptop |
| Session memory | flat over 30 minutes (CI soak) |
| Snapshot size | few KB |
| Tick p99 | alarm > 5s |

### 10.2 Tactics

- Aggressive code-split settings/social/export.
- Procedural audio & lean art.
- Edge HTML + early snapshot (HTTP/2 push or embedded bootstrap token path carefully cache-private).
- rAF pause when hidden; audio suspend.
- Buffer pools; notebook virtualized scroll without retaining all DOM.

### 10.3 Observability (aggregate only)

Synthetic browsers geo fleet: TTFB bird, FPS proxy, audio errors.

RUM: navigation timing, first-bird paint, long tasks, audio context errors, API latencies — **no bird ids, no traits, no per-user interaction content**.

Explicit dual storage: ops metrics store ≠ simulation DB.

Do **not** measure: engagement streaks, visit-frequency gamification, average drift dashboards from raw per-bird fields exported to warehouse.

---

## 11. Rollout

### 11.1 Phased ship

1. **Internal dogfood:** auth, snapshot, 2 birds, presence, drift dry-run metrics.
2. **Closed beta:** full interactions, notebook templates, a11y, audio.
3. **GA:** visit invites default off; adoption unlock clocks real.

### 11.2 Birds-per-aviary ramp

- all users: 2 starters
- unlock ladder age-based toward 7
- audio lab validates recognizability at 5–7 before enabling sixth/seventh globally (feature flag on max birdsER)

### 11.3 Day-one instrumentation

- Tick latency/lag, event ingest errors
- Snapshot latency, first-bird timing
- Audio context failure rate → caption default path
- Magic-link success/expiry rates
- CI: drift calibration replay, memory soak, no-personality-in-snapshot contract test, presence triple-condition unit tests

### 11.4 Content & design deps

Calm palette design-system symbol tokens; vocabulary docs for naturalist vs matter-of-fact; species art + motif libraries.

---

## 12. Risks and mitigations

| Risk | Why it hurts | Mitigation |
|------|--------------|------------|
| Drift too fast | Tamagotchi feel | Caps per tick/day; weekly calibration suite; monotonic but α small |
| Drift too slow | Screensaver feel | Presence weight floor; instrument 7-day SMART |
| Lax presence | Silent population over-drift | Triple condition + 4m activity; server daily caps; no tab-open-alone |
| LWW personality / dual client sim | Lost weeks of self | Server-only writers; contract tests; ban client trait fields |
| Personality leak to UI | Stat optimization | Snapshot schema strip; ESLint ban `boldness` in UI; no debug panel in prod |
| Audio uncanny / looped feel | Breaks aliveness | Procedural only; silence fallback; ear QA chorus |
| Simultaneous greeting | Announces arrival | Stagger greeters |
| Welcome toast regression | Destroys notice-never-announce | UI audit checklist; forbid toast lib on aviary route |
| A11y as checklist labels | Rations charm | Budget design for narration & reduced-motion as peer surfaces |
| Bundle bloat | Miss TTFB bird | Bundle CI gate 2MB; code-split |
| Visit presence bleed | Host relationship distorted | Token scopes; reject visitor events |
| Notebook generic logs | Voice collapse | Template review; sparsity; ban systemy phrases |
| Soft-delete / export bugs | Trust | Integration tests; encrypted email lifecycle |
| Tick backlog after outage | Mood snaps oddly | Catch-up tick with time-stepped multi-pass small Δ |
| Reduced-motion lag | Users excluded at launch | Same release train as GA |

---

## 13. Engineering work breakdown (execution-oriented)

1. **Foundations:** monorepo, Postgres schema, account UUID+email crypto, magic link, sessions revoke.
2. **Sim core:** tick worker, event log Appending, drift+mood pure functions, unit/property tests, calibration harness.
3. **Snapshot API + SPA shell:** quiet-field load, mid-motion first paint, perch layout responsive.
4. **Bird render + idle moods:** species pack, reduced-motion modes.
5. **WebAudio grammar:** motifs, mix, listen-in ramps, caption strings, silence path.
6. **Interactions:** greeting, offer, settle/undo, presence collector.
7. **Notebook:** generator + UI read-only naturalist.
8. **A11y:** keyboard, live region narration, contrast, settings matter-of-fact.
9. **Visits:** invite email, read-only SPA mode, log, revoke, expiry 30d.
10. **Export/delete, privacy policy link, ops RUM gates.**
11. **Perf CI soak + synthetic monitors; GA flags for bird caps.**

---

## 14. Explicit non-implementation

This document is the plan only. No application code, no `phase_two` consumption, no product implementation in this phase.

---

## 15. Traceability to product principles

| Principle | Plan expression |
|-----------|-----------------|
| Feels alive | Server tick continuity; mid-action first frame; procedural calls; idle motion |
| Notice never announce | Bird greeting only; no toasts/streaks |
| Charm from specificity | Notebook & narration templates; no generic achievements |
| Restraint | 7 bird cap, one scene, sparse chrome, calm palette |
| Dual voice | Naturalist product surfaces; matter-of-fact system |
| Presence as attention | Strict presence definition as drift spine |
| No Tamagotchi | Monotonic drift; ambient quiet on neglect |

---

*End of plan — batch ready for independent engineering execution without further PRD clarification.*
