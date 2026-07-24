# Pocket Aviary — Phase 1 Implementation Plan

Executable plan for v1. Interprets the PRD into architecture, data model, APIs, simulation, sync, rendering, audio, a11y, performance, rollout, and risks. Does not implement the product.

---

## 1. Scope

### In v1

- Browser-only SPA (last two majors of Chrome, Safari, Firefox, Edge)
- Single-user accounts: email magic link, one synthetic UUID account ID, one aviary per account
- Two starter birds at signup; age-gated offers up to seven birds
- Server-side simulation tick (~1/min), personality drift, mood, bird-to-bird behavior
- Client: single horizontal scene, day/night (user local TZ), ambient weather, procedural calls (WebAudio)
- Interactions: return-greeting, presence accounting, listen-in, offer (seed / song fragment / still pool), settle, field notebook (read-only)
- Multi-device sync via canonical server state + append-only interaction event log
- Optional visit invitations (off by default, email invite, read-only ambient, revocable)
- Accessibility: screen-reader narration (naturalist prose), reduced-motion designed surface, call captions, WCAG AA chrome, full keyboard nav
- Account export (JSON), soft-then-hard delete (30 days), session revoke, visit log in settings

### Out of v1 (non-goals enforced)

- Native apps
- Gamification (streaks, badges, levels, visit calendars, XP, counters of “birds adopted”)
- Tamagotchi mechanics (death, hunger, distress, decaying happiness)
- Social network surfaces (profiles, follows, discovery, comments, chat, co-presence, leaderboards)
- Shared aviaries, multi-aviary accounts, payments, customizable scenes, push/email about aviary state
- Numerical personality exposure anywhere
- Recorded-audio fallback path
- Per-bird interaction data in aggregate analytics / ML

**Decision when ambiguous:** prefer notice-over-announce, sparsity over feature density, server ownership of personality, silence+captions over canned audio.

---

## 2. Architecture

### Service shape

| Service | Responsibility |
|--------|----------------|
| **Auth** | Magic-link issue/consume, session tokens (per device), revoke, email change verify |
| **API gateway** | Session auth, rate limits, routes to snapshot / events / account / invite |
| **Simulation worker** | Tick loop (~60s), consumes event log, writes bird + aviary canonical state, ambient weather scheduling, notebook generation triggers |
| **State store** | Postgres (or equivalent) for accounts, birds, vectors, moods, notebook, invites, visits; append-only `interaction_events` |
| **Realtime / snapshot** | Snapshot GET (+ optional short-lived SSE/WS for multi-tab wake only; not required for correctness). Edge-cached empty-shell HTML; auth’d snapshots uncached or short TTL per account |
| **Mailer** | Magic links, visit invites, export download links |
| **Telemetry** | Aggregate ops only; separate from simulation DB |

Clients never tick and never write personality. Clients pull snapshots and append interaction events.

### Client/server split

- **Server owns:** personality vectors, mood, perch targets (logical zone + depth), offer cooldowns ends, weather phase, settle flag windows, drift, bird-to-bird response scheduling seeds, notebook entries, visit ACL
- **Client owns:** interpolation/rendering, WebAudio synthesis & mix, presence detection & heartbeats, local time palette, reduced-motion visual path, caption layout, quiet-field boot until first snapshot

### Render pipeline boundary

```
snapshot JSON → SceneGraph (perches, birds, ambient layers)
             → Animator (pose pools / cross-fade if reduced motion)
             → Canvas or SVG compositor (prefer Canvas 2D or WebGL-lite for 60fps idle; SVG only if budget holds at 7 birds)
AudioGrammar(snapshot + listen-in focus) → WebAudio graph
NarrationBuilder(snapshot + priority events) → live region / caption strip
```

No game loop that advances canonical simulation. Client clock only interpolates between server-known waypoints and runs cosmetic leaf/feather ornaments.

---

## 3. Data model

### Account

- `id` UUID (synthetic; only internal key)
- `email_enc`, `email_hash` for lookup
- `created_at`, `timezone` (preferred; fallback to client-reported offset each session)
- `deletion_scheduled_at` nullable
- Graph settings: visit notify opt-in (default false), a11y prefs (captions, reduced motion override)

### Session

- `id`, `account_id`, `device_label`, `token_hash`, `created_at`, `last_seen_at`, `revoked_at`

### Aviary

- `account_id` PK/FK
- `started_at` (age for adoption offers)
- `bird_count_cap` = 7
- `lighting_mode` normal | settled
- `weather` none | rain | wind + `weather_until`
- `version` (monotonic for snapshot races)
- `last_tick_at`

### Bird

- `id` UUID stable identity forever
- `aviary_id`, `species_id`, `display_name`
- `adopted_at`, `order_index`
- Personality vector (server-only stored, never derivable-only):
  - `boldness`, `social_warmth`, `vocal_frequency`, `plumage_saturation`, `curiosity` — scalars in fixed range e.g. [0, 1]
- Mood: enum `wary | content | curious | drowsy | alert` + `mood_updated_at`
- Placement: `perch_zone` front|middle|back, `perch_slot`, `pose_id`, `motion_phase` (for mid-action hydrate)
- Cooldowns: `offer_available_at` per bird or global-per-bird map
- Call grammar seed: `call_seed` immutable random for motif identity

### Species (static catalog ~6)

- silhouette id, default palette, motif library id, night-active flag (nightjar-like)

### InteractionEvent (append-only)

- `id`, `account_id`, `bird_id` nullable, `type`, `payload` JSON, `client_ts`, `server_ts`, `device_session_id`
- Types: `presence_ping`, `listen_in_start`, `listen_in_end`, `offer`, `settle`, `settle_undo`, `return_session`, `focus_idle` (optional)

### PresenceEvent (optional table or derived)

- Aggregated presence-seconds buckets for tick; or tick folds raw pings with anti-inflation rules

### NotebookEntry

- `id`, `aviary_id`, `observed_at`, `prose` (lowercase naturalist), `source_signals` internal only

### VisitInvite

- `id`, `host_account_id`, `visitor_email_enc`, `token_hash`, `created_at`, `expires_at` (+30d unused), `revoked_at`, `accepted_at`

### VisitSession / VisitLog

- visitor token, start/end, duration_approx; no presence contribution to host drift

### MagicLink

- token hash, account or pending email, expires 15m, consumed_at

**Hard rules:** personality never last-write-wins; email never used as partition/key outside account table; identified birds immutable identity across rename.

---

## 4. API surface

Auth header: `Authorization: Bearer <session>`. All account paths keyed by synthetic UUID from session.

### Auth

- `POST /auth/magic-link` `{ email }` → 202 (always generic success message)
- `GET /auth/magic-link/consume?token=` → set session cookie or return token + redirect SPA
- `POST /auth/sign-out`
- `GET /account/sessions`, `DELETE /account/sessions/:id`
- `POST /account/email-change`, `POST /account/email-change/confirm`

### Aviary state

- `GET /aviary/snapshot` → full render snapshot  
  Include: birds (id, name, species, mood, zone, pose, motion_phase, plumage display params derived not raw vector numbers as user stats — display params ok as colors), weather, lighting, time_of_day_phase, listen_in permission true, version, server_now  
  **Omit raw personality vector numbers from client forever** (derive only presentation fields server-side if needed).
- `GET /aviary/snapshot?since_version=` optional delta
- Pull also: on `visibilitychange` visible, after long rAF gap, keepalive ~30–60s while focused

### Events

- `POST /aviary/events` batch append  
  `{ events: [{ type, bird_id?, payload, client_ts, idempotency_key }] }`  
  Server validates, stamps `server_ts`, never applies personality in this path.

### Interactions convenience (optional thin wrappers still writing events)

- Offer: payload `{ kind: seed|song|pool, target_bird_id? }` — recipients resolved server-side by curiosity/mood near gesture zone
- Settle / settle_undo
- Listen-in start/end with bird_id

### Presence

- `POST /aviary/presence` or presence events in batch: server accepts only if flags in payload claim visible+focused+recent_input; **server also trusts client but clamps rate** (e.g. max N seconds credit per wall minute). Client must enforce local triple-condition before send.

### Notebook

- `GET /notebook?cursor=` reverse chrono; entries immutable

### Adoption

- `GET /aviary/adoption-offer` if age unlock ready  
- `POST /aviary/adopt` accept next bird (system picks species; user supplies optional name)

### Account lifecycle

- `POST /account/export` → email download link  
- `POST /account/delete`, `POST /account/delete/cancel`  
- Settings CRUD: a11y, visit notify

### Visits

- Host: `POST /visits/invites` `{ email }`, `GET /visits/invites`, `DELETE /visits/invites/:id`, `GET /visits/log`
- Visitor: `GET /visits/v/:token/snapshot` read-only; no event POST except optional telemetry count; revoke → matter-of-fact 410 body

### Errors (matter-of-fact copy)

Sign-in, session timeout, load failure strings exactly in PRD spirit — no naturalist voice.

---

## 5. Simulation engine design

### Tick cadence

- Target ~60s; jittered per shard to avoid thundering herd
- Idempotent: process events with `server_ts ∈ (last_tick_at, now]`
- p99 compute alarm > 5s

### Tick steps (ordered)

1. Ingest interaction events (order by server_ts, tie-break id)
2. Presence buckets → presence-seconds since last tick (clamp)
3. Weather: rare scheduler (few rains/week; soft wind); set flags and mood nudges
4. Time-of-day from account TZ → base mood biases (morning alert, dusk drowsy, night settle; night-active species exception)
5. Bird-to-bird: threshold wariness contagion; chorus windows if high vocal_frequency overlap
6. Mood transitions (Markov-ish with personality modifiers): recent offer, listen-in, rain dampen vocal expression
7. Perch targets: boldness → front bias; wary → back; social_warmth → cluster tendency
8. **Drift** (slow LPF):  
   - Dominant: presence-time → all traits gently toward expressive ceiling  
   - Listen-in duration on bird → social_warmth, vocal_frequency  
   - Offer near bird / accept → curiosity, boldness  
   - Settle: end presence cleanly; no special drift direction  
   - **Monotonic toward expressive only** — neglect does not decrease traits; expression of greets may reduce via “observed rate” smoothing separate from vector if needed (ambient quieter presence of greeting frequency can be a soft derived rate without punishing vectors downward)
9. Pose/motion_phase advance for snapshot mid-action
10. Notebook candidate: rare spark if noteworthy (first tee greeting order of week, long quiet, weather) — enforce sparsity (e.g. max ~2–3/week baseline)
11. Write new state + bump version; set `last_tick_at`

### Drift calibration targets

- Measurable instrument delta ~1 week regular presence  
- User-visible feel ~3 weeks  
- Single session invisible  
- CI property tests: 0 presence → no positive trait move beyond noise = 0; heavy presence accelerates vs control within bounds

### Call-grammar runtime (spec for client; server sends seeds + mood + vocal params)

- Motif library per species; combine with personality pitch/timing jitter  
- Recognizability invariant across mood (same motif family)  
- Chorus: independent generators mixed, not loop stack  

### Greeting selection (session open)

- Client or server: pick greeter by boldness × mood × absence_length  
- Absence from last presence end  
- Stagger if secondaries; procedural variety (no 3-clip rotation)

### Adoption unlock

- Pure aviary age thresholds (e.g. ~weeks/months cascaded) — never engagement score  
- Capac 7 hard stop  

---

## 6. Sync model

### Canonical source

One aviary row + birds per account. All devices: session → same `GET snapshot`.

### Write path

```
Client events ──append──► interaction_events
                              ▲
Simulation tick ──read──┘──write──► birds/aviary
Client snapshot ◄──read─────────────┘
```

No client personality mutation. No LWW on vectors. Multi-device concurrent offers: events ordered; cooldown enforced server-side (second offer may no-op politely).

### Conflict prevention

- Personality: only tick  
- Names/settings: version or row lock; rare conflict → matter-of-fact retry  
- Snapshots include `version`; client discards stale optimistic UI if any (prefer zero optimistic personality)

### Presence multi-device

Each device may presence-ping; tick unions with diminishing returns / clip so dual devices don’t double-speed drift (e.g. max(wall presence windows) not sum walls with concurrent credit — **decision: credit unique wall-clock minutes with at least one qualifying device**, not sum device minutes).

### Background tab

Stop render; stop presence credit; server keeps ticking. On return: pull snapshot → mid-action hydrate + return-greeting rules.

---

## 7. Frontend rendering pipeline

### Boot

1. Paint quiet field (soft sky; faint motion cues only) — **no spinner**  
2. Fetch snapshot ASAP (preload critical path with HTML if possible)  
3. Place birds mid pose/phase; start ambient + audio  
4. Trigger return-greeting within 1–2s  

Never spin-then-fade “product start” animation.

### Scene layout

- One viewport, no pan/zoom/scroll of world  
- Three zones front/mid/back; slots auto-assigned  
- Responsive: compress gaps; never crop birds  
- Top bar: account, a11y, notebook, offer, settle — fades toward transparent after idle; restore on pointer/key  
- No chrome inside scene  

### Motion

- Continuous idle: preen, scan, head-tilt, weight shuffle — mood-shaped  
- Reduced-motion: still pose sequences + slow cross-fades; no leaf drift; slow day palette  
- Cosmetics client-side leaf/feather at low rate  

### Day/night

- Local TZ driven palette; settle forces evening wash over few seconds; undo within 5s on any scene click  

### Transitions

- Perch moves interpolated between snapshots  
- Empty only first adopt fly-in once  

### Asset strategy

Procedural plumage where possible; compact SVG/sprites; code-split settings/visits/notebook drawer.

---

## 8. Audio pipeline

### Synthesis

WebAudio: motif oscillators / noise / envelope / filters parametrized by grammar; buffer reuse pools; no per-call permanent alloc.

### Mix

- Ambient default gains by “distance” (zone)  
- Listen-in: slow ramp focus up, others down but never mute (~seconds-long curves; not channel switch)  
- Weather / settle dampen gains  

### Captions

From same grammar instance as audio; short naturalist phrases near bird.

### Fallback

No WebAudio → silence + captions default on; no sample packs.

### Bundle

No multi-MB sample library; motif params in compact JSON in main or lazy bird pack under total <2MB gzip JS budget.

---

## 9. Accessibility surfaces

### Screen reader

- Polite live region; naturalist running prose  
- Cadence 30–60s idle; faster on greeting, offer, settle  
- Not ARIA “mood: content” dumps  
- Same voice as notebook  

### Keyboard

- Tab: top bar → enter scene birds  
- Arrows between birds; Enter listen-in; Esc exit  
- Offer/settle via bar shortcuts  
- Focus ring high-contrast on bright and night  

### Contrast

WCAG AA on all chrome, captions, system UIs  

### Reduced motion

First-class path shipping day one with feature (detect `prefers-reduced-motion` + settings override)

### Captions toggle

A11y settings; auto on silent path  

**Stance:** accessible paths stay charming, not checklist shells.

---

## 10. Performance budgets and observability

| Budget | Target |
|--------|--------|
| Initial JS gzip | < 2MB |
| First bird visible (mid mobile / 4G) | < 500ms |
| Idle | 60fps on ~5yo mid laptop; 30+ min sessions |
| Memory | No growth over 30 min (CI soak) |
| Tick p99 | Alarm > 5s |

### Measure (aggregate only)

- TTFB, first-bird paint, rAF hitch rate, audio context errors, tick latency, HTTP error rates  
- Anonymized session duration histograms without account dimension for product-sensitive joins  
- Synthetic geo browser fleet  

### Deliberately do not measure / store in warehouse

- Per-bird vectors, offer contents tied to identity for dropout learning, visit patterns as social graphs for ranking, “engagement optimization” funnels that imply streak directions  

### Privacy plane

Simulation DB not joined to analytics warehouse for bird fields; pipelines separate.

---

## 11. Rollout

### Phases

1. **Internal dogfood:** 2 birds, core tick + presence + contents of greetings, audio, quiet field load  
2. **Closed beta:** multi-device, notebook, settle, a11y triad  
3. **v1 public:** visits opt-in, export/delete, age unlock of bird 3+ ramp  
4. **Ramp birds-per-aviary:** unlock thresholds by real-world weeks; monitor call recognizability usability comments not LMS; hold hard at 7  

### Day-one instrumentation

- Tick health, snapshot latency, first-bird timing, audio fail counts  
- Drift canaries: synthetic accounts with scripted presence always-on vs never — trend trait deltas  
- Privacy: confirm no bird fields in telemetry sampling tests  

### Feature flags

Server-side for weather density, notebook sparsity, unlock ages — not gamification toggles.

### Browser gate

Unsupported UA → matter-of-fact upgrade message  

---

## 12. Risks and mitigations

| Risk | Why it hurts | Mitigation |
|------|--------------|------------|
| Drift too fast | Tamagotchi feel | LPF constants; weekly instrument gates; dual-device presence max not sum |
| Drift too slow | Screensaver | Presence weight tuning; dogfood 3-week longitudinal |
| Drift on neglect downward “by accident” | Breaks no-Tamagotchi | Unit property: neglect paths don’t decrease vectors |
| Client LWW / optimistic personality | Silent identity loss | No client personality writes; review code paths |
| Presence inflated by open tabs | Population overdrift | Triple signal + server rate clip + activity window long enough for still watching |
| Canned audio / loops sneaks in | Kills aliveness | Ban samples in CI lint/bundle; grammar-only |
| WebAudio uncanny metallic birds | User flinches | Design review on motifs; mute-friendly captions |
| Accessible path as state dump | Second-class product | Narrative tests in CI (prose quality samples); ship RM/captions day one |
| Spinner / welcome toast prince creep | Violates notice | PR checklist + no-toast lint on session mount |
| Visit slowly becomes social network | Gravity shift | Code ownership: refuse feeds; no aggregate visit rankings |
| Tick backlog after outage | Mood snaps | Catch-up ticks with capped steps + smooth day phase from real clock not N° skipped emotions incorrectly |
| Export exposes raw vectors | Stat-management temptation | Export is for ownership/portability — acceptable for user export of own vectors; **UI still never shows**; optionally label as technical |
| Email as ID | PII sprawl | UUID only; static analysis on logs |
| Bundle >2MB | Misses mid-action first frame | Disc budgets in CI; aggressive split |

---

## 13. Work breakdown (execution order for eng team)

1. Auth + synthetic account + empty quiet shell SPA  
2. Schema birds/events + snapshot endpoint  
3. Tick worker skeleton + mood/time baseline  
4. Client hydrate mid-motion + day palette  
5. Presence client+server credit  
6. Drift + calibration harness  
7. Call grammar + mix + listen-in  
8. Offer/settle/cooldowns  
9. Greeting selections  
10. Notebook generator + UI  
11. A11y narration/RM/captions/keyboard  
12. Adoption age unlock  
13. Visits invite/read-only/log/revoke  
14. Export/delete/sessions  
15. Perf soak + synthetic monitors + privacy boundary tests  

---

## 14. Explicit product decisions (ambiguity calls)

1. **Presence multi-device:** unique wall-clock minutes, not summed devices.  
2. **Snapshot omits raw personality numbers;** derived colors/pose only. User JSON export may include vectors for portability.  
3. **Greeting orchestration** computed on snapshot + `return_session` event so all devices agree on who greeted when.  
4. **Renderer:** Canvas 2D default with SVG escape hatch if art prefers and fps holds.  
5. **Realtime push optional;** correctness never requires WS.  
6. **Song library:** small fixed set of motif abstractions, not licensed music tracks.  
7. **Visit email that isn’t a user:** still link works as guest visitor token until expiry; guest no account required for visit.  

---

## 15. Success criteria (v1 quality bar)

- User never sees spinner-of-machine or “Welcome back” on return  
- Birds feel continuous across absence and across phone/laptop  
- Presence-only weeks change birds instrumentably; neglect does not punish  
- Seven-distinct call recognizability remains design constraint  
- A11y users get naturalist aliveness, not inventory UI  
- Ops can see health without reading anyone’s relationship  

---

*End of plan. Stop before product implementation and before evaluation phase.*
