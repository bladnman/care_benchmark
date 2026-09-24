# Pocket Aviary v1: Implementation Plan

**Status:** Phase-1 plan, ready for execution
**Audience:** The engineering, design, audio, and accessibility team building Pocket Aviary v1
**Source of truth:** The PRD set (`product_brief`, `concepts`, `bird_engine`, `interactions`, `aviary_layout`, `accounts_sync`, `social_optional`, `accessibility_perf`, `non_goals`). Where the PRD is ambiguous or contradicts itself, this plan makes the call and records it in §3.4. Every such decision has a stated rationale and a named owner who can reverse it.

---

## How to read this plan

- §1–§3: what we are building, the invariants that constrain every line of code, what's in and out of scope, and the decisions log.
- §4–§13: the executable design: architecture, data model, API, simulation engine, sync, rendering, audio, accessibility, voice system, and account/security/privacy mechanics.
- §14–§18: performance budgets and observability, test strategy, rollout, team and timeline, and risks.
- Appendices: snapshot example, event schemas, drift-calibration personas, and prose-grammar samples.

Numbers given as parameters (time constants, thresholds, weights) are **initial calibration values**. They live in versioned engine config, not in code constants, and are tuned by the process in §7.5 and §16.5. Numbers given as budgets (500 ms, 2 MB, 60 fps, p99 5 s) come from the PRD and are hard gates.

---

## 1. Executive summary

Pocket Aviary is a single-screen, browser-only aviary of 2–7 procedurally animated, procedurally voiced birds. Their hidden personalities drift slowly and only upward in response to the user's honest attention. The product's value is affective, so this plan treats "feels alive", "notice, never announce", and "no gamification" as **engineering invariants with enforcement**, not as design aspirations.

These twelve load-bearing decisions shape everything else:

1. **Server-authoritative, deterministic simulation.** The aviary advances on a server tick. The tick is a pure, deterministic step function: fixed-point math and a counter-based PRNG. Only the simulation worker writes personality or mood. Clients render and append events; they never tick.
2. **Two-tier tick scheduling with exact equivalence.** Aviaries with recent activity tick every 60 s. Dormant aviaries run the same 1-minute steps in batches every 15 minutes. Because the step function is deterministic, batched and real-time ticking produce identical state. A read-time projection serves up-to-the-second snapshots without waiting for a batch.
3. **Additive, monotonic, saturating drift.** Traits change only via server-computed, non-negative deltas that approach a per-bird ceiling asymptotically. A two-stage filter produces this: daily-saturating credit, then a leaky pressure integrator. Monotonicity is enforced three times: in code, by a database trigger, and by a nightly audit.
4. **"Quieter after absence" lives outside the personality vector.** A separate expression-layer state, *attunement*, decays during absence and recovers quickly with presence. It only changes how readily birds orient toward the viewer. It never produces wariness, distress, fading color, or any trait decrease.
5. **Presence is a client-evaluated three-signal conjunction, verified on the server.** Presence is `visible ∧ focused ∧ recent activity`, sent as monotonic-clock heartbeats. The server takes the union of presence across devices, so there is no double counting, and clamps it to wall time and a daily saturation curve.
6. **No trait value ever reaches a client.** Snapshots carry compiled presentation parameters (colors, call-rate parameters, perch assignments, reaction plans), never the personality vector. The only place raw vectors leave the server is the user-requested account export (decision D-01).
7. **First frame is the aviary.** An edge-rendered, streamed HTML response inlines the snapshot. A ≤110 KB gzipped "first-bird" module draws birds mid-action. There is no spinner anywhere in the codebase; a lint rule enforces this. The slow path is a quiet field, or the last-known snapshot cached on the device.
8. **Canvas 2D cut-out renderer behind a draw-list interface.** Rig parts are pre-rasterized per bird and composited per frame with transforms. This gives 60 fps on 2021-class integrated graphics, and a WebGL2 backend can replace it without touching scene code.
9. **All audio is procedural, synthesized in one AudioWorklet with a fixed voice pool.** Calls come from per-species grammars plus per-bird signature parameters that never drift. Captions come from the same grammar expansion that produced the sound. The repo contains no audio files, and CI enforces that.
10. **Accessibility is a designed register, not a fallback.** Screen-reader narration is naturalist prose from the shared prose grammar, rate-limited to about one update per 30–60 s. Reduced motion is its own cross-fade renderer. Captions, keyboard access, and contrast ship in v1 and are launch gates.
11. **The privacy boundary is architectural.** The identity DB and the aviary DB are separate. Telemetry runs on a separate network and credentials, with an allowlisted metric schema that has no account, bird, or interaction-type dimensions. Emails are encrypted, looked up via a blind index, and never logged. Hard deletion crypto-shreds per-account keys.
12. **The voice split is machine-checked.** Every user-visible string sits in a copy registry tagged as either `naturalist` or `system`. Lints check case, punctuation, banned vocabulary (gamification, announcement, and terminology), and second-person address in naturalist prose.

---

## 2. Product invariants → engineering guardrails

These rules are absolute. Each one has an enforcement point and a test, because "we'll be careful" is how they get broken.

| # | Invariant (PRD source) | Enforcement | Test / monitor |
|---|---|---|---|
| I-1 | Personality values are never shown to the user, at any version or tier (`bird_engine`). | The protocol package has no trait fields. The client codebase has no trait type. The snapshot compiler is the only path from traits to presentation. Support tooling cannot read trait columns (DB grants). | Schema test: snapshot/visitor/notebook payloads contain none of the trait keys and no field correlates 1:1 with a trait (compiler unit test on mixing/quantization). Lint rule bans trait identifiers in `apps/web`. |
| I-2 | Notice, never announce: no toasts, banners, badges, confetti, or "welcome back" (`product_brief`, `interactions`). | The design system ships no toast, snackbar, or badge component. A lint rule bans imports of toast/notification libraries and the `Notification` API. The copy registry bans phrases such as "welcome", "you've been", "days", and "streak". | Surface inventory review at every milestone. Copy lint in CI. A Playwright test asserts no element with role `alert`/`status` becomes visible on load, return, or offer, except the visually hidden narration region. |
| I-3 | No gamification or visit-frequency surfaces, including the notebook (`non_goals`, `interactions`). | The notebook detector input schema contains only bird and aviary facts. Presence/session records are not visible to the detectors (type-level separation). There are no visit-count or streak queries anywhere; a code-review checklist item enforces this. | Property test: random detector inputs never yield text with second person or user-behavior facts. Banned-vocabulary lint. |
| I-4 | The server is the only writer of personality; drift is additive server deltas; no last-write-wins (`accounts_sync`). | DB grants: only the `sim_writer` role may UPDATE `bird_personality`/`aviary_state`; the API role has INSERT-only on `interaction_events`. There is no API route that accepts a trait, mood, or position. | Grant test in CI (connect as each role, attempt forbidden writes). Chaos test: two devices, interleaved events, no lost credit. |
| I-5 | Drift is monotonic toward expressive and never goes down on neglect (`bird_engine`). | Non-negative credit by construction. In-code assertion before commit. A `BEFORE UPDATE` trigger rejects any decrease. The nightly audit compares against `bird_trait_daily`. | Property-based tests over random event streams. Metric `trait_decrease_rejected_total` must be 0 (page on >0). |
| I-6 | Presence = visible ∧ focused ∧ activity within the window, and visitors never generate presence (`concepts`, `social_optional`). | A single `AttentionMonitor` module is the only producer of presence heartbeats. The visitor bundle does not include the event writer. The server rejects events carrying visit credentials. | Unit tests for every combination of the three signals. E2E: background tab for 2 h yields 0 credited minutes. Visitor-token write attempt → 403. |
| I-7 | Stable bird identity: never reset, regenerated, or swapped (`bird_engine`). | UUIDv4 `bird_id` assigned at adoption. There is no delete path except account hard-delete. Species data is versioned and each bird pins its species version. Migrations touching birds require invariant checks (row counts, checksums, monotonic traits). | Migration CI harness runs pre/post invariant checks against a production-shaped fixture. Metric `suspected_reset_total` (traits equal to seed after ≥14 days of credited presence) must be 0. |
| I-8 | The synthetic account UUID is the only identifier; email is stored once, encrypted (`accounts_sync`). | Email is envelope-encrypted with a per-account DEK and looked up via an HMAC blind index. A log wrapper redacts email-like strings. Partition keys, metrics labels, and queue keys are typed as `AccountId`/`AviaryId` branded UUIDs. | CI log-scrubber test injects emails into every service path. A production log scanner alerts on any email-pattern match. |
| I-9 | Per-account interaction data never feeds aggregate analysis (`accounts_sync`). | Separate network, credentials, and pipeline for telemetry. The metric schema allowlist has no account, bird, species, mood, trait, or interaction-type labels. The analytics role has no route to the aviary DB. | Schema-allowlist test at the OTel collector (unknown labels are dropped and counted). Quarterly access review. |
| I-10 | Calls are procedural; no recorded audio, including in the fallback (`bird_engine`, `accessibility_perf`). | Build check fails if any audio MIME/extension or `decodeAudioData` call appears in the bundle. | Bundle scanner in CI. |
| I-11 | The first frame is mid-motion; no spinner, fade-from-static, or entry sequence (`aviary_layout`). | No spinner component exists (lint). The scene has no opacity/entry transitions on mount. The boot renderer starts every idle action at a random phase. | Visual test: the first committed frame has ≥1 bird with non-rest pose parameters. Lint for `spinner|loading-indicator` identifiers. |
| I-12 | The voice split: naturalist lowercase for product surfaces, matter-of-fact for system surfaces (`product_brief`). | Copy registry with a mandatory `register` tag; lints differ per register (§12). | CI lint. A writer reviews every new registry key. |
| I-13 | 2 birds to start, max 7; new birds by aviary age only (`bird_engine`). | DB trigger caps birds per aviary at 7. The newcomer schedule is a function of `aviaries.created_at` only. | Unit tests on the schedule. The DB rejects an 8th bird. |
| I-14 | Birds don't die, starve, or show distress (`non_goals`). | The mood model has no distress state. No input path runs from absence to `ease`. Attunement cannot affect mood. | "Absence invariance" property test: N days with no events yields mood statistics equal to the no-user baseline (§7.16). |

---

## 3. Scope

### 3.1 In v1

- Magic-link auth, per-device sessions with a revocation list, email change with verification, soft→hard deletion (30 days), and JSON export by emailed link.
- One aviary per account. Adoption of two system-selected starter birds with naming. Rename at any time. Newcomers by aviary age up to 7.
- A server-side tick with drift, mood, weather, perch selection, bird-to-bird events, attunement, the notebook observation pipeline, and offer reaction planning.
- A web client: the scene (three perch zones, day/night on account-local time, weather, ambient leaves/feathers, subtle parallax), idle micro-motion, return-greeting, listen-in, offers (seed, song fragment, still pool), settle with 5 s undo, field notebook, and the top bar with fade.
- The audio engine: procedural calls, chorus, listen-in mix, and captions generated from the grammar. The silent-plus-captions fallback.
- Accessibility: screen-reader narration, reduced-motion register, captions, full keyboard model, WCAG 2.2 AA contrast/focus/target sizes, and an opt-in visual narration display.
- Visits: invite by email, one-time link → browser-bound visit pass, read-only ambient view, revocation, 30-day expiry of unused invites, a visit log, and an opt-in visit-notification email (off by default).
- The unsupported-browser surface, the privacy policy link, and aggregate operational telemetry plus synthetic monitoring.

### 3.2 Explicitly out of v1

These come from the PRD non-goals, plus deferrals this plan adds.

- **Never, in any version:** achievements, streaks, levels, scores, XP, badges, visit calendars, "birds adopted" counters, leaderboards, public discovery, profiles, follows, comments, chat, avatars, a show-off mode, hunger or happiness meters, death, distress, push notifications, and aviary-content emails.
- **Not in v1:** native apps, payments, shared or household aviaries, multiple aviaries per account, customizable scenes, user-controlled perch placement, SSO or passwords, co-presence, and a species catalog/picker.
- **Deferred by this plan (decision-logged):** localization of naturalist prose (v1 is English; system strings are i18n-ready), real-world weather, location-based sun times beyond timezone coordinates (§9.2), background-tab audio (§10.7), a service-worker app shell (§9.1), and SSE/WebSocket push (§8.2).
- **No growth experiments on engagement.** There will be no A/B tests whose metric is return frequency, session length, or retention. Feature flags exist for safe infrastructure rollout only (§16).

### 3.3 Things an enthusiastic team will propose, and the pre-committed answer

| Proposal | Answer | Why |
|---|---|---|
| "A tiny 'welcome back' toast" | No | I-2. The greeting *is* the welcome. |
| "Show 'enable sound' button when autoplay is blocked" | No button in the scene. The top-bar audio glyph shows a muted state; audio starts on the first gesture. | Chrome inside the scene; announcing. |
| "Debug panel with trait values, just for us" | Only in non-production lab tools on synthetic aviaries | I-1. Production has no per-bird value surface. |
| "Average drift across users to validate calibration" | No. Calibrate in simulation and consented staging studies. | I-9 / `accounts_sync` privacy. |
| "Use an LLM to write notebook entries" | No. Use a hand-authored grammar. | A third-party data flow would violate the privacy commitment. A grammar also gives determinism and testability. |
| "Loading spinner while the snapshot loads" | No. Show the quiet field. | I-11. |
| "Retention dashboard" | No per-account cohorts. Aggregate operational health only. | §14.4 |

### 3.4 Ambiguities and decisions log

Each decision is the default the team builds. The owner can reverse it before the listed milestone without re-architecture.

| ID | PRD tension or gap | Decision | Owner / by |
|---|---|---|---|
| D-01 | `accounts_sync` export includes "current personality vectors", but `bird_engine` says values are never shown to the user. | Include raw vectors **only in the export file**. The file is user-requested, delivered by email link, never rendered in-product, and presented as stored simulation state. Every rendered surface, API response, support tool, and narration follows I-1. Server config `export.include_personality_vectors` (default `true`) allows reversal without code change. | PM + privacy counsel / M2 |
| D-02 | The top bar holds "nothing else" but the four icons, yet settle is "triggered from the top bar". | Settle lives in the **offer affordance's gesture popover**: *offer a seed / offer a song fragment / offer a still pool / settle for the evening*. The top bar stays at four icons. | Design / M1 |
| D-03 | The brief says drift responds to "whether you mute the calls", but `bird_engine` does not list mute as a drift input. | Mute is recorded as an `audio_state` event. Presence minutes while muted credit vocal frequency at 0.3× instead of 1×. Nothing is ever negative. The mute control lives in the accessibility panel and has a keyboard shortcut. | Engine lead / M1 |
| D-04 | Drift never decreases, yet a bird ignored for two weeks should be "quieter than they were". | Add **attunement** (§7.6), an expression-layer variable outside the personality vector. It modulates only viewer-directed behavior and has a floor that always preserves the guaranteed first greeting. | Engine lead / M1 |
| D-05 | The mood set is "finalized in implementation". | Six moods: `alert`, `curious`, `content`, `wary`, `drowsy`, `roosting`. `roosting` is the night state the PRD calls "settled". It is named differently in code to avoid collision with the user's *settle* gesture. Prose may still say "settled for the night". | Engine lead / M1 |
| D-06 | The presence activity window is "a few minutes, lean long". | Default **5 min**. Calibration range 3–8 min, chosen by the staging study (§7.5). The value is versioned with the drift model. | Engine lead / M3 |
| D-07 | Presence cites "pointermove or keypress". `keypress` is deprecated, and touch taps may not fire `pointermove`. | Activity = `pointermove`, `pointerdown` (covers touch), `keydown`, `wheel`. `wheel` counts because scrolling is deliberate attention and the aviary itself never scrolls. | Engine lead / M1 |
| D-08 | Lifetime of a used ("active") visit invitation is unspecified. | The one-time link is consumed on first open and converted into a browser-bound **visit pass**. The pass is valid until revoked, until the host enters pending deletion, or after 90 days without a visit ("lapsed" in the visit log). | PM / M3 |
| D-09 | Channel for opt-in visit notifications (there is no push in the product). | Email only, debounced to once per visitor per 24 h, matter-of-fact wording. | PM / M3 |
| D-10 | Do notebook entries show old or new names after a rename? | Entries store structured facts plus bird IDs and render **current names** at read time, keeping identity continuous. Wording and variation are frozen at write time. | Writer + PM / M2 |
| D-11 | Offers are made to the aviary, but cooldown is per bird. | Birds in cooldown may glance but don't approach and earn no drift. Offered items linger (seed for 10 min, pool for 10 min), so a bird whose cooldown ends may still come. A client-side global pace of one offer per 20 s prevents mashing. | Engine + design / M2 |
| D-12 | Is "settled" per device or canonical? | The evening lighting is **device-session local**. The settle *event* is canonical and applies a mood-quieting nudge at the next tick. Other devices see calmer birds, not evening light. | Engine + design / M1 |
| D-13 | Audio in hidden tabs is unspecified. | Fade out over 2 s and suspend the `AudioContext` when hidden. Resume with fade-in on return. This covers battery, background-timer throttling, and the rule that presence isn't counted there. Revisit after launch. | Audio lead / M2 |
| D-14 | Browser autoplay policies block "calls already audible" on many first loads. | Try to resume at boot (Chrome's media-engagement index often allows it for returning users). Otherwise stay silent, with no prompt in the scene, until the first user activation, then fade in over 2.5 s. The top-bar audio glyph shows a muted state. | Audio lead / M1 |
| D-15 | Species pool of "about six"; the nightjar-like species. | Six species, one of them nocturnal. Starters are never the nocturnal species, and the two starters come from distinct call registers (one high, one low). A seventh bird may repeat a species; same-species birds get maximally separated voice seeds. | Sound design + design / M1 |
| D-16 | "A few months → 3rd bird; a year → 5 or 6". | Eligibility at aviary age 90, 180, 270, 365, and 480 days, each with a per-aviary jitter of ±10 days. Minimum 45 days between adoptions. At most one newcomer visits at a time. See §7.15 for the flow. | PM / M3 |
| D-17 | Timezone source for "the user's local time" across devices. | The account timezone auto-follows the most recent session start, with a manual override in settings. On change, lighting ramps over 20 minutes and the mood engine re-anchors (birds are gently "jet-lagged" for a day). | Engine + design / M2 |
| D-18 | Pronouns for birds in generated prose. | Generated prose uses names, "the bird", or species epithets and never assigns gendered pronouns. Per-bird pronouns are deferred. | Writer / M2 |
| D-19 | Account state during the 30-day soft-delete window. | The aviary keeps ticking. Signed-in pages show a matter-of-fact page with **"I changed my mind"** instead of the aviary. Invites and passes are suspended and restored on recovery if not expired. A confirmation email is sent to the address on file. | PM / M2 |
| D-20 | Email-change link lifetime is unspecified. | 24 h. A matter-of-fact notice goes to the old address on completion (security). | Security / M2 |
| D-21 | Session lifetime is unspecified. | 60-day sliding idle expiry, 1-year absolute. Revocable per device. | Security / M2 |
| D-22 | Captions default. | Off by default. **On** by default when WebAudio is unavailable (PRD). Visitors choose captions locally. | A11y lead / M1 |
| D-23 | "Narration when displayed visually" is implied by the WCAG section. | Opt-in "show narration as text" setting renders narration as a quiet strip at the scene's bottom edge. | A11y lead / M2 |
| D-24 | Visit-log retention is unspecified. | Entries are retained 12 months, then deleted (visitor emails are PII). Outstanding and active invitations remain listed until they expire or are revoked. | Privacy / M3 |
| D-25 | Magic links and email security scanners. | GET never consumes a token. The landing page has a single matter-of-fact "Sign in" button that POSTs, so link-scanning proxies can't burn links. | Security / M2 |
| D-26 | Name casing in naturalist prose: the brief writes "Pip greeted before Wren", but the notebook samples use "pip … wren". | Naturalist-register prose (notebook, narration, captions) renders bird names lowercase, matching the voice samples. Settings and system surfaces show names exactly as typed. | Writer / M2 |
| D-27 | The minimum on-screen size for birds vs "never crop a bird" on 320 px viewports. | The layout solver shrinks birds to a floor of max(36 px, 7% of viewport height) before any other compromise. A 7-bird aviary at 320 px is a named design-review gate for raising the bird cap (§16.2). | Design / M3 |

---

## 4. Architecture

### 4.1 System overview

```
                         ┌───────────────────────── Browser ──────────────────────────┐
                         │  boot (inline) → first-bird module (scene-core, clock)     │
                         │  lazy: audio engine (AudioWorklet), a11y/narration,        │
                         │        interaction, top bar, notebook, settings, visits    │
                         │  AttentionMonitor → EventQueue (IndexedDB-backed)          │
                         └───────┬──────────────────────────────┬─────────────────────┘
                   HTML+inline snapshot                  JSON API (snapshot, events,
                   static assets (immutable)             offers, notebook, account)
                                 │                              │
                    ┌────────────▼────────────┐      ┌──────────▼───────────┐
                    │ Edge workers (CDN)      │─────▶│ Regional API (x3)    │
                    │ streamed HTML shell,    │ 150ms│ auth, snapshot read + │
                    │ session check, assets,  │ budget│ projection, events,  │
                    │ 103 Early Hints         │      │ notebook, visits     │
                    └─────────────────────────┘      └──┬───────┬───────────┘
                                                        │       │
                        regional Valkey (snapshot cache,│       │ read replica
                        auth cache, rate limits) ◀──────┘       │ (aviary-db, identity-db)
                                                                │
   ─────────────────────────────── home region ─────────────────┼──────────────────────────
     ┌──────────────┐   ┌──────────────────┐   ┌───────────────▼──┐   ┌─────────────────┐
     │ sim-scheduler│──▶│ sim-workers (N)  │──▶│ aviary-db (PG)   │   │ identity-db (PG)│
     │ (lease loop) │   │ step(), compile  │   │ primary + PITR   │   │ primary + PITR  │
     └──────────────┘   └──────┬───────────┘   └──────────────────┘   └───────▲─────────┘
                               │ publish snapshot (post-commit)               │
                               ▼                                              │
                        snapshot fan-out ──▶ regional Valkey          ┌───────┴─────────┐
                                                                      │ jobs: email,    │
     KMS (KEKs; per-account DEKs wrapped)                             │ export, delete, │
     Object storage (exports, lifecycle 7d)                           │ invite expiry,  │
     Transactional email provider                                     │ notebook render │
                                                                      └─────────────────┘
   ─────────────────────── separate telemetry account/VPC ────────────────────────────────
     OTel collector (schema allowlist) → metrics TSDB, traces (sampled, scrubbed), logs (14d)
     RUM beacon endpoint (no cookies, credentials: 'omit') → aggregate-only
     Synthetic browser fleet (5 geos) → same metrics store
```

### 4.2 Services and responsibilities

| Service | Runtime | Owns | Never does |
|---|---|---|---|
| **edge** | CDN workers (TypeScript) | Streams the HTML shell. Validates the session cookie against the regional auth cache. Fetches the inline snapshot from the regional API with a 150 ms budget. Serves immutable assets. Sends 103 Early Hints. Serves the unsupported-browser page. | Reads databases directly. Logs cookies or emails. |
| **api** | Node.js 24 LTS, Hono, stateless, 3 regions | Auth flows, sessions, account settings, snapshot serving (cache → replica → projection), event ingest (writes to the primary), offers, notebook reads, visits, export and deletion requests | Writes personality or aviary state. Holds any user state in memory beyond a request. |
| **sim-scheduler** | Node.js, home region, 2 replicas (leader-elected) | Maintains `aviary_schedule`, promotes aviaries warm/cold, detects lag, sheds load | Computes simulation |
| **sim-worker** | Node.js worker pool, home region, horizontally scaled | Leases due aviaries and runs `step()` k times. Commits state, personality, cursor, and observation candidates in one transaction. Compiles and publishes snapshots. | Serves user traffic. Sees emails (it has no identity-db credentials). |
| **jobs** | Node.js queue consumers (pg-boss on identity-db) | Magic-link, invite, export, and visit-notification emails. Export builds. Soft→hard deletion. Invite expiry. Notebook text pre-render cache. The nightly integrity audit. | Performs user-visible sends not listed in §13.6. |
| **telemetry** | OTel collector plus managed TSDB, in a separate cloud account | Aggregate metrics, sampled scrubbed traces, and 14-day logs | Has network reachability to either database |

**Monorepo layout (pnpm workspaces):**

```
packages/
  engine/        # pure deterministic step(), drift, mood, weather, perch, offers, attunement,
                 # newcomer schedule, snapshot compiler; zero I/O; fixed-point + PRNG
  prose/         # naturalist grammar runtime + authored grammars (captions, narration, notebook,
                 # adoption copy); shared client/server
  protocol/      # snapshot + event + API schemas (TypeScript types + valibot validators),
                 # schema version constants; NO trait fields (I-1)
  species/       # versioned species catalog: rig params, palettes (OKLCH), motif libraries
  voice/         # call-grammar expansion + parameter mapping (shared: server derives voice
                 # identity; client expands calls)
  copy/          # copy registry + lints
  rng/           # counter-based PRNG (Philox-4x32-10 via Math.imul), fixed-point helpers
apps/
  web/           # client (boot, scene, audio, a11y, ui-chrome via Preact)
  edge/          # CDN worker
  api/           # HTTP API
  sim/           # scheduler + workers
  jobs/          # async jobs
  lab/           # NON-PRODUCTION: persona simulator, calibration charts, grammar explorer
tools/
  lint-rules/    # eslint plugins: no-trait-in-client, no-spinner, no-toast, copy-register
  perf/          # bundle budget, soak tests, device-lab runners
```

### 4.3 Client/server split

| Concern | Server (canonical) | Client (presentation) | Shared code |
|---|---|---|---|
| Personality vector, drift | ✔ only writer | never sees it | — |
| Mood (affect coordinates + label) | ✔ computed each tick | blends toward snapshot values for idle motion | `engine` mapping fns |
| Perch zone/slot, flights between zones | ✔ decided each tick, emitted as timed transitions | interpolates. Synthesizes a natural hop if a change arrives without a transition. | layout slot table |
| Weather | ✔ per-aviary stochastic process | renders rain/wind and tapers stale weather | — |
| Ambient call timing | emits calling parameters + seed per tick window | deterministic scheduler from seed. Two devices at the same moment hear the same ambient calls. | `voice`, `rng` |
| Reactive calls (greeting, offer response, listen-in glance) | supplies dispositions/plans | generates with fresh entropy | `voice` |
| Idle micro-motion | — | procedural director from affect + presentation params | — |
| Greeting | supplies greeting dispositions and account-level absence | selects the greeter and choreographs | `engine/greeting` |
| Offer reactions | precomputes reaction plans per offer kind per snapshot. Validates the event and credits drift. | executes the plan instantly | `engine/offers` |
| Day/night lighting | supplies account timezone | computed locally every frame from local clock + tz | `engine/sun` |
| Captions | — | generated from the actual grammar expansion | `prose`, `voice` |
| Narration | — | composed from scene state + event stream | `prose` |
| Notebook | detects observations, stores facts | renders the list (names resolved at read) | `prose` |
| Presence | union, clamp, saturate, credit | evaluates the three-signal conjunction, sends heartbeats | — |

### 4.4 Render-pipeline boundary

The contract is **"canonical state in, pixels and sound out; attention events out."** The only inputs to the client scene are:

1. the latest snapshot (`protocol.Snapshot`, version-monotonic),
2. local clocks: `performance.now()`, the server-time offset, and the account timezone,
3. local interaction: pointer, keyboard, the attention state, and accessibility preferences.

Everything the client renders is a function of these three. The client writes only `protocol.ClientEvent`s. No client module can mutate canonical state. The snapshot is immutable in memory, frozen in dev builds. This boundary lets visitors run the same renderer with a narrower snapshot and no event writer.

### 4.5 Technology choices

| Area | Choice | Rationale | Escape hatch |
|---|---|---|---|
| Language | TypeScript everywhere | The engine, voice, and prose code is shared between server and client. Same-language determinism is testable. | The engine is pure and could be ported to Rust/WASM if tick cost grows. |
| Determinism | Integer PRNG (Philox via `Math.imul`), fixed-point Q16.16 for engine state, no transcendental functions on decision paths (lookup tables with linear interpolation) | `Math.exp`/`Math.sin` differ across JS engines. The server and client must agree on ambient schedules. | — |
| Server HTTP | Hono on Node 24 | Small, fast, edge-compatible | — |
| Databases | PostgreSQL 17 (two clusters: identity, aviary), multi-AZ, PITR | Transactional single-writer semantics, triggers for invariants, SKIP LOCKED leasing | Shard aviary-db by `aviary_id` hash once past ~1.5M aviaries (§16.6). |
| Cache | Valkey (regional) | Snapshot cache, auth cache, rate limits | — |
| Job queue | pg-boss | No new infrastructure. Transactional enqueue. | — |
| Client UI chrome | Preact + signals (~6 KB) | Tiny. The scene is framework-free. | — |
| Scene renderer | Canvas 2D, cut-out rig with per-bird atlases, behind a `DrawList` interface | Small code, robust across GPUs, no context-loss complexity. Meets 60 fps with part caching (§9.9). | A WebGL2 backend consumes the same `DrawList`. The M0 spike decides whether it is needed at launch. |
| Audio | WebAudio + one AudioWorklet synth with a fixed voice pool | Sample-accurate, allocation-free audio thread | — |
| Build | Vite, ES2022 target, Brotli, hashed immutable assets, `modulepreload` | Last-two-versions browser baseline | — |
| Email | Transactional ESP on a dedicated sending subdomain, SPF/DKIM/DMARC `p=reject` | Magic-link deliverability is on the sign-in critical path. | Keep a second provider warm for failover. |
| Observability | OpenTelemetry → managed metrics/traces/logs in a separate account | Enforces the privacy boundary (I-9) | — |

### 4.6 Deployment topology

- **Home region** (e.g., `us-east`): database primaries, sim-scheduler, sim-workers, jobs.
- **Serving regions** (`us-east`, `eu-west`, `ap-southeast`): API, Valkey, database read replicas (lag SLO < 1 s p99).
- **Edge:** a global CDN with workers. Each edge pins the nearest serving region for snapshot fetches.
- **Writes** (events, offers, settings) go from the regional API to the home primary. Their latency (up to ~200 ms cross-ocean) is off the critical path: events are asynchronous, and offers execute from precomputed plans (§7.12).
- **Data residency:** v1 stores all data in the home region under standard contractual transfer safeguards. Privacy counsel signs off at M2. If EU residency is required, the aviary-db shard key already supports an EU home shard.

### 4.7 The privacy boundary as architecture

1. **Two databases.** `identity-db` holds accounts, encrypted emails, sessions, links, invitations, and the visit log. `aviary-db` holds aviaries, birds, traits, moods, events, and notebook. The aviary DB contains no email, ever. Bird names are encrypted with the account DEK.
2. **Credentials.** `sim_writer` (aviary-db write), `api_rw` (identity-db rw, aviary-db INSERT on events plus SELECT on replicas), `jobs`, `support_readonly` (identity-db account and session metadata only; no aviary-db access), and `audit` (integrity checks that emit counts only). No role in the telemetry account holds credentials to either database.
3. **Network.** The telemetry VPC has no route to database subnets. Metrics are pushed out through the collector; nothing pulls in.
4. **Schema allowlist.** The collector drops any metric label not in `telemetry/allowlist.yaml` and increments `telemetry_label_dropped_total{metric}`. Allowed label keys: `route`, `method`, `status_class`, `region`, `service`, `build`, `browser_family`, `device_class`, `tier`, `result`, `reason`, `job`.
5. **Logs.** A structured logger with an allowlist of fields. `account_id` may appear (operational: "is this account having errors"). Request and response bodies, event payloads, bird names, trait/mood values, and emails may not. 14-day retention.
6. **Support.** Production has no admin UI that shows bird state. Incident repair uses break-glass access with two-person approval, audit logging, and time-boxed credentials, and it prefers integrity status (ok/violated) over values.

---

## 5. Data model

### 5.1 Conventions

- **Identifiers.** `account_id`, `aviary_id`, and `bird_id` are UUIDv4: random, with no creation time leaked. `event_id` is a client-generated UUIDv7, which gives ordering and idempotency. All are branded types in TypeScript (`AccountId`, `BirdId`, …), so an email or name can't be passed where an ID is expected.
- **Fixed point.** Trait values, ceilings, pressures, affect, and attunement are stored as `integer` micro-units (0–1,000,000 ≙ 0.0–1.0). The engine computes in Q16.16 and converts at the edges. No floats are persisted for engine state.
- **Encryption.** Each account has a random 256-bit DEK, wrapped by a KMS KEK and stored on the account row. Encrypted columns use AES-256-GCM with the DEK: email, bird names, and visitor emails in invitations. Blind indexes use HMAC-SHA-256 with a separate KMS-held key over normalized email (NFKC, lowercased, trimmed). DEKs are cached in memory for a short time (5 min) in the API and jobs only. Sim-workers never hold DEKs.
- **Time.** All timestamps are `timestamptz` in UTC. "Local date" is computed with the aviary's IANA timezone at write time and stored explicitly where it matters (daily buckets).

### 5.2 identity-db

```sql
accounts (
  account_id        uuid PRIMARY KEY,
  email_ct          bytea NOT NULL,            -- AES-GCM(DEK)
  email_bidx        bytea NOT NULL UNIQUE,     -- HMAC(normalized email)
  dek_wrapped       bytea NOT NULL,            -- KMS-wrapped per-account key
  kek_version       int   NOT NULL,
  status            text  NOT NULL CHECK (status IN ('active','pending_deletion')),
  deletion_requested_at timestamptz, hard_delete_at timestamptz,
  timezone          text  NOT NULL,            -- IANA, e.g. 'Europe/Lisbon'
  timezone_mode     text  NOT NULL DEFAULT 'auto' CHECK (timezone_mode IN ('auto','manual')),
  settings          jsonb NOT NULL DEFAULT '{}',-- validated: see §5.5
  created_at        timestamptz NOT NULL
);

sessions (
  session_id uuid PRIMARY KEY, account_id uuid NOT NULL REFERENCES accounts,
  token_hash bytea NOT NULL UNIQUE,            -- SHA-256 of 256-bit opaque token
  ua_label   text NOT NULL,                    -- coarse: 'Firefox on Windows'
  created_at timestamptz NOT NULL, last_seen_at timestamptz NOT NULL,
  absolute_expiry timestamptz NOT NULL, revoked_at timestamptz
);

auth_links (                                   -- sign-in and email-change links
  link_id uuid PRIMARY KEY, purpose text CHECK (purpose IN ('sign_in','email_change')),
  account_id uuid NULL,                        -- NULL for first sign-in (account created on consume)
  email_bidx bytea NOT NULL, email_ct bytea NOT NULL,   -- encrypted with a pending-DEK held in row
  pending_dek_wrapped bytea NULL,
  token_hash bytea NOT NULL UNIQUE,
  created_at timestamptz NOT NULL, expires_at timestamptz NOT NULL,  -- +15 min sign_in, +24 h email_change
  consumed_at timestamptz NULL
);

invitations (
  invite_id uuid PRIMARY KEY, public_id text UNIQUE NOT NULL,  -- non-secret, used in /visit/{public_id}
  host_account_id uuid NOT NULL REFERENCES accounts,
  visitor_email_ct bytea NOT NULL, visitor_email_bidx bytea NOT NULL,
  link_token_hash bytea UNIQUE NOT NULL,       -- one-time link
  pass_hash bytea UNIQUE NULL,                 -- set on first use (browser-bound pass)
  created_at timestamptz NOT NULL, expires_at timestamptz NOT NULL,   -- +30 d if unused
  first_used_at timestamptz, pass_last_used_at timestamptz,
  revoked_at timestamptz, suspended boolean NOT NULL DEFAULT false   -- D-19
);

visits (                                       -- the host's visit log (D-24: 12-month retention)
  visit_id uuid PRIMARY KEY, invite_id uuid NOT NULL REFERENCES invitations,
  host_account_id uuid NOT NULL, started_at timestamptz NOT NULL, last_seen_at timestamptz NOT NULL
);

export_jobs (job_id uuid PK, account_id uuid, requested_at, status, object_key text,
             download_token_hash bytea, expires_at, downloaded_at);
deletion_ledger (account_id uuid PRIMARY KEY, hard_deleted_at timestamptz);  -- UUIDs only; re-applied after any restore
```

### 5.3 aviary-db

```sql
aviaries (
  aviary_id uuid PRIMARY KEY, account_id uuid UNIQUE NOT NULL,   -- 1:1 at v1
  created_at timestamptz NOT NULL,             -- drives newcomer schedule (I-13)
  timezone text NOT NULL,                      -- mirrored from identity on change
  engine_version int NOT NULL,                 -- forward-only drift/mood model version
  rng_key bytea NOT NULL,                      -- 128-bit secret per aviary for Philox
  status text NOT NULL CHECK (status IN ('active','pending_deletion'))
);

aviary_state (                                 -- fast-changing canonical state, one row
  aviary_id uuid PRIMARY KEY REFERENCES aviaries,
  version bigint NOT NULL,                     -- +1 per committed tick batch (CAS)
  tick_index bigint NOT NULL, simulated_until timestamptz NOT NULL,
  event_cursor bigint NOT NULL,                -- last consumed interaction_events.server_seq
  state_blob bytea NOT NULL,                   -- msgpack, schema-versioned (§5.4)
  updated_at timestamptz NOT NULL
) WITH (fillfactor = 70);                      -- HOT updates

birds (
  bird_id uuid PRIMARY KEY, aviary_id uuid NOT NULL REFERENCES aviaries,
  species_id text NOT NULL, species_version int NOT NULL,        -- pinned (I-7)
  name_ct bytea NOT NULL,                      -- AES-GCM(account DEK)
  voice_seed bigint NOT NULL, appearance_seed bigint NOT NULL,   -- immutable identity seeds
  adoption_index smallint NOT NULL CHECK (adoption_index BETWEEN 1 AND 7),
  adopted_at timestamptz NOT NULL,
  arrival text NOT NULL CHECK (arrival IN ('awaiting_naming','arrived')),
  UNIQUE (aviary_id, adoption_index)
);
-- trigger: reject INSERT if count(birds WHERE aviary_id) >= 7; no DELETE grant to any app role

bird_personality (                             -- the guarded table
  bird_id uuid PRIMARY KEY REFERENCES birds,
  boldness int, social_warmth int, vocal_frequency int, plumage_saturation int, curiosity int,
  ceil_boldness int, ceil_social_warmth int, ceil_vocal_frequency int,
  ceil_plumage_saturation int, ceil_curiosity int,
  pressure jsonb NOT NULL,                     -- per-trait leaky-integrator state (fixed-point)
  version bigint NOT NULL, updated_tick bigint NOT NULL,
  CHECK (boldness BETWEEN 0 AND ceil_boldness) -- …and the same for each trait
);
-- BEFORE UPDATE trigger: RAISE if NEW.<trait> < OLD.<trait> for any trait, or if
-- current_user <> 'sim_writer', or if NEW.ceil_* <> OLD.ceil_*. Increments a counter table
-- read by the audit exporter.

bird_trait_daily (bird_id uuid, local_date date, boldness int, social_warmth int,
                  vocal_frequency int, plumage_saturation int, curiosity int,
                  PRIMARY KEY (bird_id, local_date));   -- integrity audit + DR second line

bird_daily_behavior (bird_id uuid, local_date date,
                     front_minutes smallint, back_minutes smallint, calls smallint,
                     greeted_first boolean, offers_approached smallint, chorus_joins smallint,
                     PRIMARY KEY (bird_id, local_date));  -- notebook long-horizon facts; 400-day retention

interaction_events (                           -- append-only, partitioned daily by received_at
  server_seq bigserial, event_id uuid NOT NULL, aviary_id uuid NOT NULL,
  session_id uuid NOT NULL, type smallint NOT NULL, bird_id uuid NULL,
  payload jsonb NOT NULL,                      -- validated per type (Appendix B)
  received_at timestamptz NOT NULL,
  PRIMARY KEY (received_at, server_seq),
  UNIQUE (aviary_id, event_id)                 -- idempotent ingest (ON CONFLICT DO NOTHING)
);
-- api_rw: INSERT only. sim_writer: SELECT. retention job: DROP PARTITION when
-- all rows consumed and partition age > 7 days.

notebook_entries (
  entry_id uuid PRIMARY KEY, aviary_id uuid NOT NULL,
  local_date date NOT NULL, observed_at timestamptz NOT NULL,
  template_key text NOT NULL, template_version int NOT NULL,
  facts jsonb NOT NULL,                        -- bird_ids + slot values; no user-behavior facts (I-3)
  variation_seed bigint NOT NULL, salience smallint NOT NULL
);  -- index (aviary_id, observed_at DESC); no UPDATE/DELETE grants (read-only notebook)

aviary_observations (aviary_id uuid, local_date date, facts jsonb,
                     PRIMARY KEY (aviary_id, local_date));  -- detector working set; 60-day retention

aviary_schedule (
  aviary_id uuid PRIMARY KEY, next_tick_at timestamptz NOT NULL,
  tier text NOT NULL CHECK (tier IN ('warm','cold')), warm_until timestamptz,
  lease_owner text, lease_expires_at timestamptz
);  -- index (next_tick_at) for SKIP LOCKED polling
```

### 5.4 `state_blob` contents

This is msgpack with a schema version and holds everything fast-changing, so a tick is one row read and one row write plus the personality rows.

```
StateBlob v1 {
  weather: { kind: none|rain|wind, started_at, ends_at, intensity_q, next_rain_at, next_wind_at },
  birds: [{
    bird_id, zone: front|middle|back, slot: u8, zone_since, transition?: {from, to, start, dur_ms},
    affect: { arousal_q, ease_q, novelty_q }, mood: enum, mood_since,
    attunement_q, last_greeted_first_at?, offer_cooldown_until?,
    daily_credit: { local_date, presence_eff_q, listen_eff_q, offers_credited: u8 },
    recent: { alarm_at?, joined_chorus_at?, accepted_offer_at? }
  }],
  presence: { coverage: [ [start,end] … last 30 min ], last_presence_end_at,
              daily: { local_date, raw_minutes_q, eff_minutes_q } },
  active_offers: [{ offer_id, kind, fragment_id?, placed_at, expires_at, slot }],
  newcomer: { state: none|visiting|paused, species_id?, window_ends_at?, next_eligible_at },
  notebook_budget: { tokens_q, last_refill_at },
  rng_counter: u64,
  settle_nudge_until?: ts
}
```

### 5.5 Account settings schema

`settings` JSON is validated server-side and synced across devices.

```
{ reduced_motion: 'system'|'on'|'off',   captions: boolean,  calls: 'play'|'mute',
  narration_text: boolean,                single_key_shortcuts: boolean (default true),
  visit_notifications: boolean (default false) }
```

Device-local overrides, such as a visitor's caption choice or "follow system" resolution, live in `localStorage` and never leave the device.

### 5.6 Species catalog (`packages/species`, static, versioned)

Six species. Each is an append-only versioned record with a stable `species_id`:

| Field | Content |
|---|---|
| `rig` | Parametric silhouette: body contour Béziers, head radius, beak shape, tail fan, wing shape, leg length, crest (optional) |
| `palette` | OKLCH base colors per plumage region: crown, nape, back, wing, wingbar, breast, belly, tail, eye-ring. Chroma range is `[c_min, c_max]`; plumage saturation maps into it. |
| `markings` | Procedural pattern families (streaks, bars, spots, cap) plus density ranges |
| `motifs` | Motif library: note primitives and grammar rules (§10.3), register (pitch band), and naturalist descriptors for captions |
| `idle_bias` | Action-weight biases (e.g., a flycatcher-like species scans more; a finch-like one preens more) |
| `trait_seed` | Per-trait baseline mean (0.25–0.45) |
| `nocturnal` | true for exactly one species |
| `epithets` | Naturalist referents for prose ("the small grey one", "the streaked bird") |

Birds pin `species_version`. A new species version never changes an existing bird's look or voice unless an explicit, reviewed migration passes listening and visual A/B review with the whole team (I-7).

### 5.7 Retention and deletion matrix

| Data | Retention | Hard-delete action |
|---|---|---|
| accounts, sessions, auth_links (consumed links purged after 24 h) | life of account | row delete + DEK destroyed (crypto-shred) |
| invitations / visits | until expiry or revocation + 12 months for the log | row delete |
| aviaries, aviary_state, birds, bird_personality, notebook | life of account | row delete (cascade) |
| bird_trait_daily | life of account | row delete |
| bird_daily_behavior | 400 days rolling | row delete |
| aviary_observations | 60 days rolling | row delete |
| interaction_events | ≤7 days after consumption | partition drop / row delete |
| exports (object storage) | 7 days (bucket lifecycle) | object delete |
| logs (may include `account_id`) | 14 days | age out (≤14 d after hard delete) |
| backups (PITR) | 35 days | age out. The `deletion_ledger` is re-applied after any restore. The destroyed DEK makes encrypted fields unreadable in backups. |
| telemetry | 13 months | contains no account dimension by construction |

---

## 6. API surface

### 6.1 Conventions

- **Transport.** HTTPS only (HSTS preload), HTTP/2 and HTTP/3. JSON bodies, Brotli/gzip. All `/api/*` responses are `Cache-Control: no-store` except snapshots, which use `private, no-cache` plus an ETag.
- **Auth.** A host session cookie `__Host-pa_s` (HttpOnly, Secure, SameSite=Lax, Path=/) holds an opaque 256-bit token. The server stores only its SHA-256 hash. A visitor pass cookie `__Host-pa_v` holds up to 8 `{public_id: pass_token}` entries and is accepted **only** on `/visit/*` and `/api/visit/*`. The host cookie is ignored on visitor routes and vice versa.
- **CSRF.** State-changing requests require `Content-Type: application/json` plus a custom header `X-PA-Client: web/<build>` (which forces a CORS preflight), an `Origin` check, and SameSite=Lax.
- **Versioning.** `X-PA-Protocol: 1` on requests. Snapshots carry `schema: 1`. The server supports N and N-1 simultaneously during deploys. A stale client receiving `426` does a silent reload at the next visibility change (never mid-view).
- **Idempotency.** Events carry client UUIDv7 `event_id` (unique per aviary). Mutating account endpoints accept an `Idempotency-Key` header.
- **Errors.** `{ "error": { "code": "session_expired", "ref": "E7K2-Q9" } }`. The client maps `code` to a matter-of-fact registry string (§6.9). `ref` is a random correlation ID that appears in logs for 14 days so support can trace a report. It is not derived from any identifier.

### 6.2 Page routes (edge-rendered)

| Route | Signed-in host | Signed-out | Notes |
|---|---|---|---|
| `GET /` | Streamed aviary shell + inline snapshot | Sign-in page (quiet field + email form, system voice) | Pending deletion → the "I changed my mind" page (D-19). |
| `GET /auth/link?t=…` | Link landing: "Sign in to Pocket Aviary" button (POST) | same | GET never consumes (D-25). |
| `GET /visit/i/{token}` | Invite landing: "{host email} invited you to visit their aviary." [Visit] | same | GET never consumes. |
| `GET /visit/{public_id}` | Read-only aviary shell + inline visitor snapshot | same (pass cookie decides) | Missing, revoked, or expired pass → the "no longer available" page. |
| `GET /export/{job_id}?t=…` | Download (requires a session for the same account) | Sign-in, then continue | |
| `GET /unsupported` | Static matter-of-fact page | | Served when the feature probe fails (§9.1). |

### 6.3 Auth and account

| Method & path | Body → Response | Server behavior |
|---|---|---|
| `POST /api/auth/link` | `{email}` → `202 {}` always | Normalizes the email, computes the blind index, and rate-limits (§6.8). If the account exists, creates a `sign_in` link bound to it. Otherwise creates one with `account_id NULL` plus a pending DEK. Sends the email. The identical response prevents enumeration. |
| `POST /api/auth/consume` | `{token}` → `200 {next: '/' }` + Set-Cookie | Atomic `UPDATE auth_links SET consumed_at=now() WHERE token_hash=$1 AND consumed_at IS NULL AND expires_at>now() RETURNING …`. First sign-in creates the account, aviary, two starter birds (`awaiting_naming`), and the schedule row, all in one saga (§13.2). Failure → `link_invalid` (one code for expired, used, or unknown). |
| `POST /api/auth/sign-out` | → `204` | Revokes the current session and clears the local snapshot cache (client). |
| `GET /api/account` | → `{email, timezone, timezone_mode, settings, status, created_at}` | Decrypts email with the DEK. The only endpoint that returns email. |
| `PATCH /api/account/settings` | partial settings → `200` | Validated. Timezone changes mirror to aviary-db. |
| `GET /api/account/sessions` | → `[{session_id, ua_label, created_at, last_seen_bucket, current}]` | `last_seen_bucket` ∈ now/today/this week/older. There is no IP or location. |
| `DELETE /api/account/sessions/{id}` | → `204` | Sets `revoked_at` and publishes `session_revoked` to regional auth caches (<5 s propagation SLO). |
| `POST /api/account/email-change` | `{new_email}` → `202` | Creates an `email_change` link to the new address. The old email stays active until it's consumed. On consume, swaps atomically and notifies the old address (D-20). |
| `POST /api/account/export` | → `202` | Enqueues `export_build`. At most one job per 24 h. |
| `POST /api/account/delete` | → `200 {hard_delete_at}` | Status becomes `pending_deletion`. Suspends invitations and emails a confirmation. |
| `POST /api/account/restore` | → `200` | ("I changed my mind"). Status becomes `active` and unexpired invitations are un-suspended. |
| `POST /api/aviary/adoption` | `{names: {bird_id: name}}` → `200` | Validates names (1–24 chars; letters, marks, spaces, `'`, `-`; NFC). Encrypts and sets `arrival='arrived'`. Also used for newcomers (§7.15). |
| `PATCH /api/birds/{bird_id}` | `{name}` → `200` | Rename only. Has no other mutable field (I-4/I-7). |

### 6.4 Aviary state

**`GET /api/aviary/snapshot`** (host) and **`GET /api/visit/{public_id}/snapshot`** (visitor)

- Request headers: `If-None-Match: "<aviary_version>.<presentation_version>"`.
- `200` returns a `Snapshot` (Appendix A), typically 2–6 KB before compression. `304` if unchanged.
- Serving algorithm, the same in every region:
  1. Resolve auth (regional auth cache → identity replica).
  2. Read the compiled presentation from regional Valkey `snap:{aviary_id}`.
  3. If `simulated_until < now − 60 s`, read `aviary_state` + personality from the replica. Run `engine.project(state, unconsumedEvents, now)`: the same `step()` for ≤15 steps, in memory, with no write. Compile. Asynchronously ask the scheduler to warm the aviary.
  4. Merge names, decrypting with the cached DEK. Strip fields by audience (visitor: no offer plans, no greeting dispositions, no `last_presence_end_at`, no newcomer adoption affordance).
  5. Return with `server_time` for clock-offset estimation.
- Projection is valid only if the API's `engine_version` equals the aviary's `engine_version`. Otherwise serve the cached snapshot (at most 15 min stale) with correct `server_time`, and let the client's continuity rules absorb it.
- **Version monotonicity.** Every snapshot carries `(aviary_version, projected_steps)`. The client never applies a snapshot older than the one it's showing (§8.2).

**`GET /api/notebook?before={cursor}&limit=20`** (host only) returns `{entries: [{entry_id, local_date, weekday, text}], next_cursor}`. Text is rendered at read time from `template_key` + `facts` + `variation_seed`, with current names (D-10). A per-aviary cache is invalidated on rename.

### 6.5 Interaction events

**`POST /api/aviary/events`** (host only; visitor credentials are rejected with `403 forbidden_for_visit`)

```
{ "events": [ ClientEvent, … up to 50 ] }   →   202 { "accepted": [event_id…], "rejected": [{event_id, reason}] }
```

| `type` | Payload (validated) | Server handling |
|---|---|---|
| `presence_heartbeat` | `{interval_ms, present_ms, activity_window_ms, muted: bool}` sent every 30 s while attended | `present_ms ≤ interval_ms ≤ 45 000`. Placed in server time as `[received_at − present_ms, received_at]`, then credited as the union with the aviary's coverage set (§7.4). |
| `presence_end` | `{reason: hidden|blur|idle|settle|pagehide}` | Closes coverage and sets `last_presence_end_at`. |
| `listen_in` | `{bird_id, duration_ms, ended_by: reclick|other_bird|empty_click|focus_move|escape|hidden}` | Sent at disengage. `duration_ms` is clamped to the wall time since the previous listen-in from this session. Credited to the bird's warmth and vocal frequency. |
| `offer` | `{offer_id, kind: seed|song|pool, fragment_id?, plan_ref}` | Validates `plan_ref` (§7.12) and applies cooldowns. Adds an `active_offers` entry, making the item visible to other devices and visitors at their next pull. |
| `settle` | `{}` (sent only after the 5 s undo window elapses) | Ends presence and applies `settle_nudge` for 20 min (mood quieting). |
| `greeting_observed` | `{first_bird_id, order: [bird_id…], absence_bucket}` | Feeds only `aviary_observations` (notebook facts). No drift effect. |
| `audio_state` | `{calls: play|mute}` | Sets the muted flag for presence weighting (D-03). |

Rejection reasons are `schema`, `unknown_bird`, `stale` (older than 15 min), `duplicate`, `rate_limited`, and `cooldown` (offer accepted visually but not credited). A rejection never surfaces to the user. The client drops rejected events silently, and the aggregate metric `events_rejected_total{reason}` records them without a type label (I-9).

### 6.6 Offers

Offers are **resolved client-side from server-computed plans**, so the reaction starts on the same frame as the gesture and works through brief network loss:

- Each host snapshot contains `offer_plans: {seed, song: {fragment_id → plan}, pool}`. Each plan is an ordered set of per-bird responses (§7.12) derived deterministically from `(rng_key, aviary_version, kind, fragment_id)`.
- The client plays the plan and posts `offer{plan_ref: "<aviary_version>:<kind>:<fragment>"}`. The server re-derives the plan from the same inputs to learn which birds approached (boldness credit) and accepted (curiosity credit). The client cannot claim a different outcome.
- Plans older than 3 minutes are expired. The client then disables the offer items in the popover and shows the matter-of-fact line "Offers need a connection." when offline (§8.6). After any offer the client pulls a fresh snapshot 2 s later, which refreshes plans and cooldowns.

### 6.7 Visits

**Host endpoints**

| Method & path | Behavior |
|---|---|
| `POST /api/visits/invitations {email}` | Rate-limited (10/day, 25 outstanding). Creates the invitation (30-day expiry) and emails the one-time link. Response: `{public_id, expires_at}`. |
| `GET /api/visits/invitations` | Outstanding (`sent`, `expires`) and active (`first_visit`, `last_visit`, `lapses`) lists with visitor emails, decrypted for the host. |
| `DELETE /api/visits/invitations/{public_id}` | Revokes immediately: sets `revoked_at` and evicts the pass from regional caches. The visitor's next snapshot pull returns `410`. |
| `GET /api/visits/log?before=` | `[{visitor_email, started_at, approx_duration_bucket}]`, most recent first |

**Visitor flow (sequence)**

```
Visitor                     Edge/API                                   identity-db
  │ GET /visit/i/{token}  ─▶ landing page (no consumption)
  │ POST /api/visit/redeem {token} ─▶ verify link_token_hash, unexpired, unrevoked,
  │                                   host active → set pass_hash, first_used_at ───────▶
  │ ◀── Set-Cookie __Host-pa_v (+pass) ; 303 → /visit/{public_id}
  │ GET /visit/{public_id} ─▶ edge: pass → regional API visitor snapshot (inline)
  │ …keepalive GET /api/visit/{public_id}/snapshot every 60 s
  │                         ─▶ checks pass; upserts visits(last_seen_at)  (log duration)
  │                         ─▶ 410 {code:'visit_unavailable'} if revoked/lapsed/suspended
```

- A second browser opening an already-consumed one-time link gets the same "no longer available" page. The host can re-invite.
- The visitor snapshot is the host snapshot compiled for audience `visitor` (same birds, moods, weather, host timezone and day/night), with the fields listed in §6.4 removed. There is no special rendering (`social_optional`: no show-off mode).
- The visit log's duration comes from snapshot pulls: `last_seen_at − started_at`, bucketed as <5 min, 5–30 min, 30–60 min, >1 h. A new `visits` row starts when the gap since `last_seen_at` exceeds 30 min. There are no visitor heartbeats and no visitor events.
- Opt-in notifications (D-09): on a new `visits` row, if `visit_notifications` is on and none was sent for this invite in 24 h, enqueue the email "A visitor started viewing your aviary: {visitor email}."

### 6.8 Rate limits (Valkey token buckets; keys are blind indexes or UUIDs, never raw email)

| Scope | Limit |
|---|---|
| `POST /api/auth/link` per email blind index | 5 / 15 min, 20 / day |
| `POST /api/auth/link` per /24 (IPv4) or /56 (IPv6) | 30 / hour (IP prefix is held only in Valkey with a 1 h TTL) |
| `POST /api/auth/consume` per IP prefix | 60 / hour |
| Snapshot reads per session | 30 / min |
| Events per session | 20 requests / min, 50 events / request |
| Invitations per host | 10 / day, 25 outstanding |
| Export per account | 1 / 24 h |

### 6.9 Error codes → matter-of-fact copy (system register)

| Code | Copy |
|---|---|
| `link_invalid` | We couldn't sign you in. The link may have expired. Try requesting a new link. |
| `session_expired` / `session_revoked` | Your session timed out. Sign in again to keep watching. |
| `aviary_load_failed` (initial load only) | Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch. |
| `visit_unavailable` | This visit is no longer available. |
| `offline_offers` | Offers need a connection. |
| `unsupported_browser` | Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge. |
| `rate_limited_link` | Too many sign-in links were requested. Wait a few minutes and try again. |
| `export_requested` | We'll email a download link to {email} when your export is ready. |

---

## 7. Simulation engine

### 7.1 Principles

1. **Pure step function.** `step(state: EngineState, events: Event[], t: TickTime, cfg: EngineConfig, rng: Rng) → {state', outputs}` does no I/O and reads no clock. Everything time-dependent derives from `t` and the aviary timezone.
2. **Deterministic.** Fixed-point Q16.16 arithmetic. Transcendentals (exp, sin, pow) come from shared lookup tables with linear interpolation. The PRNG is counter-based Philox-4x32-10, keyed by the aviary's `rng_key`, with counter `(tick_index, stream_id, bird_index, draw_index)`, so every random draw is addressable and replayable. Given the same inputs, the Node server and every browser produce bit-identical results. This is verified by a cross-engine golden test on V8, JavaScriptCore, and SpiderMonkey in CI.
3. **Versioned, forward-only.** `EngineConfig` (all tunables) and `engine_version` are pinned per aviary. A new version applies from the next tick onward. State is **never recomputed from history**: personality is stored and canonical (`bird_engine`), and there is no replay-to-rebuild path in production code.
4. **Monotonic-safe by construction.** Every personality delta is `κ·P·(C − T)·Δ` with `P ≥ 0`, `C ≥ T`, and `κ, Δ > 0`, so it is always ≥ 0. There is also an explicit assertion before commit (I-5).

### 7.2 Tick scheduling

| Tier | Entered when | Cadence | Steps per job |
|---|---|---|---|
| **warm** | Any event ingest or snapshot read. Sets `warm_until = max(warm_until, now + 30 min)`. | every 60 s | 1 (more if catching up) |
| **cold** | `warm_until < now` | every 15 min | 15 (one per simulated minute) |

- **Equivalence.** The cold tier runs the *same* 1-minute `step()` fifteen times, with no events (a cold aviary has none). So state is identical to per-minute ticking, and the aviary genuinely "continues without the viewer". A property test asserts `run15(state) == step^15(state)` byte-for-byte.
- **Read-time projection** (§6.4) makes cold aviaries look current to the reader without a synchronous write. The committed batch later produces exactly the projected state. In production we compare hashes on a sample: `projection_mismatch_total` must be 0.
- **Leasing.** Workers poll `SELECT aviary_id FROM aviary_schedule WHERE next_tick_at <= now() ORDER BY next_tick_at LIMIT 200 FOR UPDATE SKIP LOCKED`, stamp a 30 s lease, and process each aviary in its own transaction:
  1. `SELECT … FROM aviary_state WHERE aviary_id=$1` (version v)
  2. `SELECT … FROM interaction_events WHERE aviary_id=$1 AND server_seq > cursor ORDER BY server_seq`
  3. `step()` k times, then compile the presentation
  4. `UPDATE aviary_state SET … version=v+1 WHERE aviary_id=$1 AND version=v`. Zero rows means another writer won (should be impossible under leasing): abort, count `cas_conflict_total`, and retry once.
  5. Update `bird_personality` rows (trigger-guarded), insert observations and notebook entries, advance the cursor, and set `next_tick_at`, all in the same transaction.
  6. After commit, publish the compiled presentation to the regional Valkey instances (fire-and-forget; readers fall back to replicas).
- **Crash semantics.** A worker crash before commit leaves nothing applied, and the lease expires and is retaken. Consumption is exactly-once because cursor advance and state write share a transaction.
- **Load shedding.** If warm lag p99 exceeds 20 s, the scheduler stretches cold cadence to 30 min (unobserved aviaries lose nothing, since projection covers reads). If warm lag p99 exceeds 45 s, it pages.
- **Capacity.** At 200k aviaries with about 4% warm at any moment: 8k warm → 133 jobs/s; 192k cold → 213 jobs/s. That's about 350 transactions/s × ~6 row writes, well within one primary. Plan to shard aviary-db by `aviary_id` hash around 1.5M aviaries (§16.6). Measured tick compute: expect ~0.2 ms per step for 7 birds, so a 15-step batch is ~3 ms.

### 7.3 Step order of operations (one simulated minute ending at `t`)

1. **Events.** Take events with `server_seq > cursor` whose `received_at ≤ t`, in `server_seq` order. Late arrivals apply at the current step; credit is additive and order-insensitive.
2. **Presence** (§7.4) produces `presence_eff_delta` for the aviary.
3. **Listen-in / offers / audio_state** produce per-bird credits. Update cooldowns and expire `active_offers`.
4. **Drift** (§7.5) updates pressures and traits.
5. **Attunement** (§7.6).
6. **Weather** process (§7.9).
7. **Affect**: compute targets, apply an Ornstein–Uhlenbeck update, then map to mood with hysteresis (§7.7).
8. **Bird-to-bird** ambient events: alarm calls, call-and-response windows, chorus windows (§7.10).
9. **Perch** decisions and flight transitions (§7.8).
10. **Newcomer** state machine (§7.15).
11. **Observation detectors** update `aviary_observations` and emit notebook candidates (§7.14).
12. **Local-midnight rollover** (aviary timezone): write `bird_trait_daily` and `bird_daily_behavior`, and reset daily credit buckets.

After k steps the **snapshot compiler** turns engine state into presentation (Appendix A). This is the only place trait values influence anything outward-facing, and they are always mixed with mood and time, then quantized (I-1).

### 7.4 Presence accounting (server half)

The client's `AttentionMonitor` (§9.10) evaluates `visible ∧ focused ∧ (now − lastActivity ≤ W)`, with W = 5 min (D-06). While in the attended state it sends `presence_heartbeat` every 30 s, using `performance.now()` (monotonic) for `present_ms`.

On the server:

1. **Placement.** Each heartbeat becomes the server-time interval `[received_at − present_ms, received_at]`. `present_ms` is clamped to `min(present_ms, interval_ms, 45 s, received_at − previous heartbeat from this session + 5 s)`. Client wall clocks are never trusted.
2. **Union, not sum.** The aviary keeps a 30-minute coverage set of credited intervals in `state_blob.presence.coverage`, and credits only the uncovered portion of a new interval. A laptop and a phone both attended at the same time earn one minute per minute. Two tabs on one device already can't both be focused.
3. **Staleness.** Heartbeats older than 15 minutes (offline queues) are rejected as `stale`. Brief offline gaps still count; day-old backfills don't.
4. **Daily saturation.** Raw credited minutes `m` for the aviary's local day map to effective minutes via `E(m) = S·(1 − e^{−m/S})` with **S = 30 min**. The increment at each step is `E(m_after) − E(m_before)`. Effective minutes/day: 15 min → 11.8, 30 → 19.0, 60 → 25.9, 8 h → 30.0. This is what makes "no single session moves a trait visibly" true even for someone who leaves a mouse jiggler running. Honest-but-lax presence (tab open, user away) is already excluded by the conjunction. Saturation bounds the dishonest or pathological case.
5. **Visitors** have no event writer and their writes are rejected (I-6). Their snapshot pulls touch only `visits.last_seen_at` in identity-db and never the aviary.

### 7.5 Drift function

**State per bird per trait:** value `T ∈ [0, C]`, ceiling `C`, pressure `P ≥ 0`.

**Seeding at adoption:** `T₀ = clamp(species.trait_seed[i] + N(0, 0.05), 0.15, 0.55)` and `C = min(0.95, T₀ + U(0.35, 0.50))`. Per-bird ceilings keep birds distinct even after years of saturation: two long-loved birds do not converge to the same maxed-out character.

**Stage 1: credit (per step)**

| Input (per effective credit-minute unless noted) | boldness | social warmth | vocal freq. | plumage sat. | curiosity |
|---|---|---|---|---|---|
| Presence (all birds in the aviary) | 1.0 | 0.8 | 0.8 (0.3 while muted; D-03) | 1.0 | 0.6 |
| Listen-in on this bird (`E_l(l)` with S_l = 15 min/day/bird) | — | 2.0 | 2.0 | 0.5 | — |
| Offer: bird approached the item (per credited offer) | +3.0 | — | — | — | — |
| Offer: bird accepted (ate, drank or bathed, or joined the song) | — | — | — | — | +6.0 |
| Settle | — | — | — | — | — (mood only) |

Offer credit is limited by the per-bird **5-minute cooldown** and **3 credited offers per bird per local day**.

**Stage 2: low-pass (per 1-minute step Δ)**

```
P ← P + Σ weight·credit                    (impulses from stage 1)
T ← T + κ · P · (C − T) · Δ                (headroom-proportional growth; never negative)
P ← P − P · Δ / τ                          (leaky integrator, τ = 36 h)
```

The total eventual movement from one credit-minute is ≈ `G·(C − T)` with `G = κ·τ = 8.5·10⁻⁴`. The filter smears each session's effect over the following ~1.5 days: the change *during* a session is imperceptible, and drift continues gently after the user leaves, driven by inputs from before they left (`accounts_sync`).

**Calibration targets (CI-gated bands for the persona harness, Appendix C):**

| Persona | Pattern | Δ after 7 d | Δ after 21 d | Max Δ in any 24 h |
|---|---|---|---|---|
| Regular | 15 min/day, 6 days/week, 1 listen-in, 1 offer/session | 0.02–0.035 (instrument-measurable) | 0.065–0.10 (≈ visible threshold) | ≤ 0.012 |
| Light | 5 min/day, 3 days/week | 0.004–0.012 | 0.015–0.035 | ≤ 0.006 |
| Heavy | 60 min/day, daily | 0.05–0.08 | 0.13–0.19 | ≤ 0.012 |
| Tab-left-open | visible 12 h/day, no focus or activity | **0** | **0** | 0 |
| Jiggler | attended with synthetic activity, 12 h/day | ≤ heavy | ≤ heavy | ≤ 0.012 |
| Two devices | regular on both at once | = regular (±2%) | = regular (±2%) | ≤ 0.012 |
| Returner | regular 3 weeks, absent 14 days, regular again | monotone non-decreasing throughout | — | — |

**The "visible threshold"** is not a guess. In M1–M2, the design and research team maps each trait's behavioral expression (front-perch occupancy, greet-first probability, plumage chroma ΔE_OK, calls per minute, offer approach latency) and runs a moderated perception study on staging accounts. Participants compare the same bird at T and T+δ, in side-by-side and in "look back" recall formats. The JND per trait is set at 75% discrimination in look-back, and G is tuned so the regular persona crosses the JND at 18–24 days. The study uses consenting participants on staging, not production data (§14.4).

**No negative path exists.** There is no "decay" term on `T`, no punishment weight, no input from absence. Absence affects only attunement (§7.6).

### 7.6 Attunement (expression layer; D-04)

`A ∈ [0.20, 1.00]` per bird, stored in `state_blob`, never persisted in `bird_personality`, never exposed.

- **Rises fast with presence:** `A ← A + (1 − A)·(1 − e^{−m/15})` for `m` credited presence-minutes this step. A listen-in on a bird adds `m_listen` to that bird's term. One unhurried session restores most of the gap.
- **Decays slowly without presence:** `A ← max(0.20, A·2^{−Δ/5 d})` in steps with no presence. After two weeks away, A ≈ 0.20–0.25.
- **Effects (and only these):**
  - greeting: the probability that birds beyond the primary greeter join is ∝ A. The primary greeter is always guaranteed (`interactions`: one bird notices within 1–2 s);
  - greeting approach amplitude (steps toward the front) ∝ `A · boldness_expr`;
  - viewer-glance rate while attended: ×(0.5 + 0.5A);
  - a front-zone utility bonus of +0.1·A, only while the user is attended.
- **Explicitly not affected:** mood and affect, ambient call rate, bird-to-bird behavior, plumage, traits, offers. A neglected aviary is therefore *ambient*, per `bird_engine`: still alive, still calling to each other, preening, weathering rain, just less oriented toward the viewer. After a long absence, the return greeting is a larger re-orientation (§9.5), so coming back feels like being noticed, not like repairing damage.
- **Guardrail test:** for every persona, mood-label distributions with A = 0.2 and A = 1.0 are statistically indistinguishable (KS test p > 0.05 over 10⁵ simulated days).

### 7.7 Mood model

Mood is a **categorical label read from a continuous affect substrate**. The continuous part lets idle motion blend smoothly with no "snap". The label drives narration, captions, the grammar, and detectors.

**Affect per bird:** arousal `a`, ease `e`, novelty `n` ∈ [0, 1].

```
a ← a + (a* − a)·Δ/θa + σa·√Δ·ξ      θa = 20 min, σa = 0.020
e ← e + (e* − e)·Δ/θe + σe·√Δ·ξ      θe = 90 min, σe = 0.015
n ← n·2^(−Δ/10 min) + stimuli        (offer placed +0.5·curiosity_x, song +0.4, weather onset +0.2, newcomer +0.3)
```

*Expression values* (`bold_x`, `warm_x`, `vocal_x`, `curio_x`) are the traits after a fixed monotone mapping. They exist only inside the engine.

- **Arousal target `a*`.** A circadian curve driven by a sun-elevation proxy for the aviary timezone (§9.2). Diurnal species: night 0.08, a dawn rise to 0.80 (alert mornings, dawn chorus), midday 0.55, late afternoon 0.45, dusk 0.28 (drowsy), night again. The nocturnal species runs inverted: night 0.50 with calling peaks, midday 0.20. Modifiers: wind +0.10, rain −0.08, a nearby alarm +0.25 (half-life 10 min), an accepted offer +0.10, the settle nudge −0.15 for 20 min, and newly arrived birds +0.20 for their first hour (so a midnight sign-up still gets an awake first meeting).
- **Ease target `e*`.** `0.45 + 0.35·(0.6·bold_x + 0.4·warm_x)`. Plus an accepted-offer bonus of +0.15 (half-life 2 h) and calm contagion of +0.05 per content neighbor × `warm_x`. Minus alarm contagion `0.25·(1 − bold_x)·proximity` (same zone 1.0, adjacent 0.6, far 0.3) and wind `0.10·(1 − bold_x)`. **There is no absence term** (I-14).
- **Label regions (hysteresis margin 0.04, minimum dwell 3 min):**

| Mood | Region |
|---|---|
| `roosting` | a < 0.12 |
| `drowsy` | 0.12 ≤ a < 0.32 |
| `wary` | a ≥ 0.32 ∧ e < 0.35 |
| `curious` | a ≥ 0.32 ∧ e ≥ 0.45 ∧ n ≥ 0.45 − 0.2·curio_x |
| `alert` | a ≥ 0.68 ∧ e ≥ 0.35 (and not curious) |
| `content` | otherwise |

- **Daily-ish reset.** The circadian cycle drives every bird through `roosting` nightly. Ease relaxes toward baseline with θ = 90 min, so yesterday's wariness is gone by morning. Mood is never reset on tab open (`bird_engine`, mood persistence).
- **Personality shaping.** Boldness raises ease and damps alarm sensitivity, so a bold bird rarely enters `wary` from the same input. Curiosity lowers the novelty threshold for `curious`. Warmth scales contagion both ways.
- **Coarse activity hint.** Each tick also picks per-bird `activity ∈ {preening, scanning, resting, foraging, calling_bout, sheltering}` from mood and weather. The client idle director (§9.4) biases toward it, so notebook statements such as "pip preened for several minutes" match what a viewer would have seen.

### 7.8 Perch selection

- **Slots** (normalized scene coordinates; §9.2): front 3, middle 4, back 4. Eleven slots for at most seven birds, which leaves room for personal space.
- **Zone utility:** `U_front = 0.9·bold_x + 0.5·(e − 0.5) + 0.1·A·[attended] − 0.3·[wary]`, `U_middle = 0.3`, `U_back = 0.6·(1 − bold_x) + 0.4·[wary ∨ drowsy] + 0.3·[dusk roost-seeking]`. The zone is drawn by softmax at temperature 0.15.
- **Movement hazard:** `(1/20 min)·(0.5 + a)`. Minimum dwell 6 min. Roosting birds don't move. At dusk, birds move to sheltered roost slots before they roost.
- **Slot choice:** the nearest free slot in the chosen zone, avoiding adjacency unless both birds have `warm_x > 0.6` (warm birds perch together).
- **Transitions:** a flight gets `start = t + U(0, 60 s)` and a duration of 1.4–3.0 s by distance, and is emitted in the snapshot so every device animates the same flight.
- **User-caused moves stay canonical.** An offer approach or a long-absence greeting approach toward the front is reported (`offer` plan outcome, `greeting_observed.approached`). The next tick places the bird in a front slot for about 10 minutes if one is free, so other devices and visitors converge.

### 7.9 Weather

- **Rain:** a Poisson process at 3 per week, weighted toward afternoon and evening, 8–25 min long, light to medium intensity only, with at least 18 h between rains. **Wind:** 5 per week, 5–12 min. No storms, thunder, or snow (`aviary_layout`).
- **Effects:** rain multiplies call rate by 0.4 during and for 10 min after, lowers arousal by 0.08, and sends each bird to a sheltered slot with p = 0.5. Afterwards there is a small novelty bump and a preening bias. Wind raises arousal by 0.10 and lowers ease by 0.10·(1 − bold_x).
- The next event times are pre-drawn into `state_blob.weather`, so the process is deterministic and identical for projections, visitors, and all devices.

### 7.10 Bird-to-bird interaction

- **Alarm calls.** A bird with `e < 0.30 ∧ a > 0.60` emits an alarm call with per-minute hazard `0.05·(1 − bold_x)`. The call feeds alarm contagion (§7.7), which is how a wary mood spreads.
- **Chorus windows.** The dawn chorus runs about 20–40 min around local sunrise. Daytime choruses start with per-minute hazard 0.03 when two or more birds have `vocal_x > 0.55 ∧ a > 0.5`. The window multiplies call rates by (1.5 + vocal_x) and raises response propensity. It is emitted as `ambient.chorus_until`.
- **Call and response.** Per-bird `respond_p = 0.2 + 0.6·warm_x` is compiled into the snapshot. The client uses it to answer another bird's call (§10.4).

### 7.11 Call grammar: the server half

- **Voice identity** is fixed at adoption and never drifts (`bird_engine`: recognizability across mood and drift). It is derived from `(species motif library @ species_version, voice_seed)`:
  - pitch offset within the species register (±3 semitones),
  - signature motif and its variant parameters,
  - timbre fingerprint (FM ratio, modulation index, noise mix, spectral tilt),
  - rhythm template.
- **Distinctness at adoption.** Compute a weighted voice distance to every existing bird in the aviary. Redraw `voice_seed` deterministically until the minimum distance is at least 3 JND units, calibrated in the M1 listening study (§10.10). This matters most for same-species pairs (D-15).
- **Calling parameters** are compiled per tick; these are the *only* vocal-frequency-driven outputs:
  - `rate_per_min = species_base × (0.4 + 1.2·vocal_x) × mood_factor × weather × time_of_day × settle_nudge`. Mood factors: alert 1.3, content 1.0, curious 0.9, wary 0.6 (short calls), drowsy 0.3, roosting 0.02 (the nocturnal species is exempt at night).
  - `chorus_join_p`, `respond_p`, `loudness ∈ [0.6, 1.0]`, and production weights (§10.3).
  - All quantized to 1/32 and mixed with mood and time (I-1).

### 7.12 Offer reaction planning

For each offer kind, the compiler produces a plan from the current affect, expression values, and cooldowns, seeded by `Philox(rng_key, aviary_version, kind, fragment_id)`.

| Kind | Per-bird utility / response model |
|---|---|
| seed | `u = 0.5·curio_x + 0.3·bold_x + 0.4·(e − 0.5) − 0.6·[drowsy ∨ roosting]`, excluded if in cooldown. Response by `u + noise`: > 0.55 → `approach` (after 0.8–4 s). 0.30–0.55 → `wait_then_approach` (after 20–90 s; "a wary bird waits and eventually comes near"). 0.10–0.30 → `glance`. Otherwise → `ignore` (drowsy birds may not approach at all). At most two approachers. |
| song fragment | Response ∈ {`join_in`, `go_quiet`, `call_against`}. Join ∝ `vocal_x·e`. Call-against for wary birds with high `vocal_x`. Go-quiet for low `vocal_x` or drowsy birds. Join-ins answer with their *own* signature motif fitted to the fragment's key and tempo (§10.4). |
| still pool | Response ∈ {`bathe`, `drink`, `watch`}. Bathe for content, bold birds. Drink is the neutral response. Watch for wary or low-curiosity birds. Up to three responders. |

Approachers **accept** with p = 0.7 + 0.3·curio_x; acceptance drives the curiosity credit. Item placement sits in the front zone near the front-most non-cooldown bird. Plans expire 3 minutes after their snapshot.

### 7.13 Greeting inputs

The engine compiles, per bird:

- `greet_weight`: from `bold_x`, `warm_x`, mood suitability, and A. The bolder, warmer bird usually greets first; wary birds greet later or not at all.
- `greet_forms`: weights per absence bucket.
- `approach_amp = A·bold_x`.
- Night eligibility: roosting birds get only eye-open or shuffle forms, and the nocturnal species is fully eligible at night.

At aviary level it compiles `last_presence_end_at` (host only). Choreography happens on the client (§9.5).

### 7.14 Field-notebook observation pipeline

1. **Detectors** run inside the step over `aviary_observations` (daily facts), `bird_daily_behavior`, and engine outputs. Their input type contains **only aviary facts**: birds, places, weather, calls, greeting order. User presence is simply not in the type (I-3). Launch detectors: `ArrivalDay`, `FirstGreeterShift` ("pip greeted before wren today, first time this week"), `WeatherPassage`, `DawnChorus`, `LongPreen`, `QuietStretch`, `FrontPerchAfternoon`, `OfferFirst` ("wren came down to the seed for the first time"), `SongAnswered`, `NightCaller`, `NewcomerSighting`, `NewcomerArrival`, and `SeasonalLookBack`. The last one compares monthly front-perch share or greet-first share against three months earlier, and it is how drift becomes legible in prose rather than numbers ("pip perches nearer the front these days than in early spring").
2. **Salience** = novelty (how rarely this detector fired in the last 30 days) × specificity (named bird, place, particular) × rarity. Range 0–1.
3. **Sparsity budget:** a per-aviary token bucket with capacity 2, refilling one token per 60 h. Emit when salience ≥ 0.6 and a token is available. "Firsts" (salience ≥ 0.9) may borrow one token. At most one entry per local day, choosing the day's highest-salience candidate after a 24 h holding window. This yields about one entry every 2–3 days, more when something is noteworthy, and **independent of how often the user visits** (`interactions`: sparsity even for very active users).
4. **Storage:** `template_key` + `facts` + `variation_seed`. Rendering happens at read time via `packages/prose` (§12.3).
5. Entries can be written during absences, because the aviary continues. They are never framed as "while you were away".

### 7.15 Newcomers (third bird and beyond; D-16)

State machine per aviary: `none → visiting (21-day window) → adopted | paused (30 days) → visiting …`

- **Eligibility** is purely age-based: `created_at + {90, 180, 270, 365, 480} d ± 10 d`. It is also gated by the operational cap `ops.max_birds` (§16.2) and a 45-day minimum since the last adoption.
- **Visiting.** A wild bird of a species not present in the aviary (preferring an unused call register) appears on a far-back edge slot during about half of attended sessions, for 3–10 minutes. It has no `bird_id` and no personality yet; it does not greet, and resident birds may turn curious toward it. The `NewcomerSighting` detector may write "a streaked bird has been visiting the far branch in the mornings. it keeps to the edge."
- **Adoption affordance.** Only while the newcomer is in the scene, the gesture popover (D-02) shows one extra item: *make room for the streaked bird*. Choosing it opens the naming sheet (naturalist register), then `POST /api/aviary/adoption` with the newcomer token. The engine creates the bird (identity seeds, trait seeds, ceilings), and the newcomer flies from the edge to its starting perch.
- **If ignored:** no reminders, no expiry message. It stops visiting when the window ends and may return after the pause. At seven birds the schedule ends permanently.

### 7.16 Engine verification

| Test | Asserts |
|---|---|
| Property (fast-check, 10⁶ cases nightly) | Traits never decrease. `0 ≤ T ≤ C`. No NaN or overflow in fixed point. `run15 == step^15`. Union presence never exceeds wall time. 24 h Δ ≤ cap. |
| Absence invariance | Mood-label distribution under "no user" equals the distribution for "user absent for N days" (KS p > 0.05). Attunement has no mood effect (§7.6). |
| Cross-engine determinism | Golden hashes of 10k-step runs identical on V8, JSC, and SpiderMonkey (Playwright runs the engine in each browser). |
| Persona harness (CI gate) | Appendix C bands across 200 seeds × 6 species. Produces lab charts for design review (synthetic only). |
| Golden replays | Recorded synthetic streams → final state hash is stable unless `engine_version` changes. |
| Fuzz | Duplicates, reordering, late and stale events, gaps; exactly-once consumption. |
| Designer plausibility review | Lab renders a simulated fortnight as a mood/perch/weather timeline for each species; the design lead signs off per engine version. |

---

## 8. Sync model

### 8.1 Guarantees

Multi-device sync is a property of the architecture, not a separate feature. Correctness rests on these properties, each with an owner test:

| # | Property | Mechanism | Test |
|---|---|---|---|
| S-1 | Single writer of canonical state | DB grants plus the service split. There is no API route that writes state. | Grant test (I-4) |
| S-2 | Exactly-once event consumption | Idempotent ingest (`UNIQUE(aviary_id, event_id)`). The cursor advances in the same transaction as the state write. | Fuzz + crash-injection test (kill the worker between each statement) |
| S-3 | No lost drift across devices | Append-only events and additive credit; presence is a union, not last-write-wins | Two- and three-device chaos test: credited totals equal the single-device oracle |
| S-4 | Monotonic state versions | CAS on `version`. The client refuses snapshots older than the one it's showing. | Replica-lag injection test |
| S-5 | Projection consistency | Same `step()` and `engine_version` pinning | Staging shadow comparison, plus `projection_mismatch_total` in production |
| S-6 | Convergence | Every device shows the same canonical state within one keepalive (≤ 70 s) while visible | Multi-device E2E: after a perch change, all devices animate the same flight |

### 8.2 Client pull strategy

A snapshot is pulled:

1. **At navigation,** inline in the HTML (§9.1).
2. **On `visibilitychange` → visible,** immediately.
3. **On a long frame gap:** a rAF delta > 5 s while visible (laptop resumed from suspend).
4. **As keepalive while visible:** at `next_tick_at + U(2, 8) s`, which is about every 60 s aligned just after the aviary's tick, with jitter to avoid thundering herds.
5. **2 s after any offer** (fresh plans and cooldowns), and on the `online` event.
6. **On focus regained,** if more than 60 s have passed since the last pull.

There are no pulls while hidden. Requests use `If-None-Match`, so the typical keepalive is a `304` of a few hundred bytes. Settled tabs still pull on keepalive; the evening lighting is local, but birds keep living.

**Clock offset.** Each response carries `server_time`. The client computes `offset = server_time − (t_send + t_recv)/2` with `performance.timeOrigin`-based times, keeps the minimum-RTT sample of the last 8, and smooths it. All snapshot timestamps (transitions, chorus windows, plan expiry) are converted to local monotonic time through this offset. Device wall-clock errors therefore never affect animation timing. Day/night uses the offset-corrected time plus the account timezone.

**Why pull and not push (v1).** Canonical state changes at tick cadence (≤ 1/min). Host interactions that matter to other viewers (offer items, perch moves) become canonical within seconds and are visible at the next pull. Two simultaneously attended devices are rare, and visitors are observers. SSE would add connection state and a fan-out tier for little perceptible benefit. It is revisited in §16.6 if keepalive traffic cost exceeds budget.

### 8.3 Client reconciliation rules

When a newer snapshot arrives, the scene controller diffs it against what is on screen and applies these rules:

| Difference | Rule |
|---|---|
| Perch change, transition included | Animate the transition at its scheduled time. If the start time has passed, start the flight now with the remaining duration clamped to ≥ 1.2 s. |
| Perch change, no transition (happened between pulls) | **Never teleport a visible bird.** Synthesize a flight or hop now (reduced motion: cross-fade; §9.7). If several birds must move, stagger by U(0.3, 1.5) s. |
| Affect or mood change | The idle director retargets. Pose parameters blend over 2–4 s, and the current idle action completes before a new one is chosen. |
| Weather started or ended between pulls | Start with a 4–6 s intensity ramp. A stale rain tapers over 5 s, never cutting off. |
| Bird added (newcomer adopted on another device) | The bird flies in from the scene edge to its perch. |
| Name changed | Update silently. Names appear only in narration, the notebook, and settings. |
| Calling or voice parameters | Apply at the next scheduled call boundary; a call in progress is never retuned. |
| Offer items (`active_offers`) from another device | Items fade in; they are part of the canonical scene. |
| Older `aviary_version` | Ignore. |
| Lighting | Always computed locally from the corrected clock and account timezone, so it is correct on the first frame after return, independent of the network. A timezone change ramps over 20 min (D-17). |

On return after a long hidden period, the stale state stays on screen for at most one network round trip while motion continues. Reconciliation then plays alongside the return greeting (§9.5), so the movement reads as the aviary noticing the user rather than catching up.

### 8.4 Event write path

- **Queue.** An in-memory `EventQueue` mirrored to IndexedDB (`pa-events`, keyed by `event_id`). Maximum 500 entries; beyond that, the oldest *presence* heartbeats are dropped first.
- **Flush policy.** Every 30 s while attended. Immediately for `offer`, `presence_end`, `listen_in`, `settle`, and `greeting_observed`. On `visibilitychange → hidden` and `pagehide`, a `fetch(…, {keepalive: true})` flush.
- **Retry.** Exponential backoff (1 s → 60 s, full jitter). The server rejects anything older than 15 minutes as `stale` (§7.4), so offline backfill can't inflate presence.
- **Auth failures.** `401 session_expired/revoked` stops flushing, clears the queue (presence can't be attributed safely), wipes the local snapshot cache, and shows the matter-of-fact sign-in surface.
- **Idempotency** by `event_id` (UUIDv7) makes duplicate sends from retries or keepalive races harmless.

### 8.5 Multi-device and multi-tab

- **Devices** read the same canonical record and write to the same append-only log. Presence is unioned (§7.4). Greetings are per device: each device's user is noticed when they arrive. Settle lighting is per device (D-12).
- **Offers from two devices** in one tick window are both accepted. The server applies cooldown and credit in `server_seq` order, so a second, stale plan may still animate locally but earns no credit. This is harmless and invisible.
- **Tabs on one device** coordinate over `BroadcastChannel('pa')`:
  - *Audio leader election:* only the most recently attended tab plays audio. Others stay silent, which avoids doubled, phasey choruses.
  - Only the visible tab renders; background tabs are hidden anyway.
  - The last-known snapshot cache is shared, with the higher version winning.

### 8.6 Failure modes and user-facing surfaces

| Failure | Detection | Behavior | User sees |
|---|---|---|---|
| Inline snapshot missed the 150 ms edge budget | No inline JSON | Render the last-known cached snapshot (≤ 7 days old, same account) or the quiet field, and fetch | Birds already present, or a quiet field briefly. Never a spinner. |
| Snapshot fetch fails mid-session | Keepalive error | Keep rendering (the deterministic ambient layer keeps birds alive) and retry with backoff | Nothing, unless offers are attempted after plans expire: "Offers need a connection." inside the popover |
| Initial load: no inline snapshot, no cache, fetch fails ×3 | — | Quiet field plus the system-register error | "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." |
| Session revoked or expired | `401` | Stop writes, clear local caches | "Your session timed out. Sign in again to keep watching." |
| Magic-link replay, expired, or used | `link_invalid` | — | "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| Tick backlog | Scheduler lag metric | Projection keeps reads current. Cold cadence stretches. | Nothing |
| Replica lag | Version compare | Client ignores older versions | Nothing |
| Engine-version skew during deploy | API vs aviary version | Serve the cached (possibly ≤ 15 min old) snapshot; continuity rules smooth it | Nothing noticeable |
| Laptop suspend | rAF gap > 5 s | Pull and reconcile, greeting if the absence qualifies | Natural motion |
| Client clock wrong | Offset estimation | All scheduling uses corrected time | Nothing |
| Visit revoked or lapsed | `410` | Audio fades out, rendering stops | "This visit is no longer available." |
| Account pending deletion | `status` | Aviary replaced by the system page | "This account is scheduled for deletion on {date}. [I changed my mind] [Sign out]" |

### 8.7 Sync correctness test plan

- **Multi-device chaos harness** (runs nightly on staging with synthetic accounts). Simulated devices send heartbeats, listen-ins, and offers with random network partitions, duplicate sends, reordering, 0–20 s delays, clock skew of ±10 min, and worker crashes. The oracle is a single-device idealized model. Assert S-1…S-6 and the persona bands.
- **Crash-injection test** kills sim-workers at each statement boundary (10⁴ runs) and asserts exactly-once consumption and no personality decrease.
- **Client continuity test** (Playwright): feed snapshot sequences containing out-of-order versions, missing transitions, and multi-hour jumps. Instrument per-frame bird position deltas and assert no discontinuity above 2% of scene width per frame outside scheduled flights.
- **Deploy-skew test:** run N and N−1 engine versions side by side on staging and assert no mismatch alarms and no client errors.

---

## 9. Frontend rendering pipeline

### 9.1 Boot sequence and the first frame

**Critical-path artifacts:**

| Artifact | Budget (gzip) | Contents |
|---|---|---|
| HTML head (flushed immediately) | ≤ 6 KB | Critical CSS, CSP nonce, `modulepreload` for `first-bird.[hash].js`, inline feature probe (~1 KB) |
| Inline snapshot chunk (streamed next) | ≤ 6 KB | `<script type="application/json" id="snap" nonce>` |
| HTML total | ≤ 14 KB | Fits the initial congestion window, so it arrives in one flight |
| `first-bird` module | ≤ 110 KB | Scene core: layout solver, lighting (with a timezone-coordinates table of ~3 KB), rig renderer, idle director core, clock, snapshot decode |
| Early Hints | — | `103` with `Link: rel=modulepreload` so the module fetch starts before the HTML |

**Timeline target** (mid-tier Android over the 4G reference profile: 9 Mbps down, 85 ms RTT, cold visit):

```
  0 ms  navigation start
~170    QUIC/TLS done (1-RTT), request sent; 103 Early Hints → module fetch begins
~260    HTML head bytes arrive (edge flushes before snapshot fetch completes)
~300    inline snapshot bytes arrive (edge→regional API ≤150 ms budget, typical 30–60 ms)
~360    first-bird module arrives (110 KB ≈ 100 ms transfer, in parallel)
~420    module evaluated (V8 code cache on repeat visits cuts ~30 ms), layout + atlases built
~450    first frame with birds committed   ← budget 500 ms (p95 in synthetic)
```

Repeat visits (HTTP cache, code cache, 0-RTT) land around 200–300 ms.

**Boot steps:**

1. The **feature probe** (inline, synchronous) checks for ES2022 syntax, Canvas 2D, `Intl.DateTimeFormat().resolvedOptions().timeZone`, `BroadcastChannel`, and `IndexedDB`. On failure it redirects to `/unsupported`. WebAudio is **not** required (§10.9).
2. The **first-bird module** reads `#snap`. If there is no inline snapshot, it reads the IndexedDB last-known snapshot (same account, ≤ 7 days old). Failing both, it draws the **quiet field**: a sky gradient for the correct local time, far foliage, and one or two faint motion cues (a drifting mote, a slow grass sway). No text and no spinner. Then it fetches the snapshot.
3. It builds the layout and per-bird atlases, starts every idle action at a **random phase** (I-11: mid-preen, mid-scan), and draws the first frame. There is no opacity fade on the canvas.
4. After the first frame (`requestIdleCallback`-staged), it loads in this order: audio engine (target: calls audible ≤ 1.5 s where autoplay permits); a11y and narration; interaction and top bar; notebook, settings, and visits on demand.
5. `performance.mark('pa:first-bird')` fires inside the rAF callback that drew the first bird. RUM reports the bucketed delta (§14.3).

**Slow-path reveal.** If the birds come from a cached snapshot, the fresh snapshot reconciles by the §8.3 rules (flights, blends). If they come from the quiet field, birds are revealed at their perches by a 500 ms atmospheric clearing (a light mist lifting from the perch plane), the one sanctioned reveal. The target is < 3% of loads (the `snapshot_inline_hit` metric).

**The empty-aviary state** after sign-up is the same quiet field. The naming sheet (§12.4) sits over it. On confirmation the first starter flies in to its perch over about 2.5 s, and the second follows 3–6 s later. After that the aviary is never empty again.

**No service worker in v1.** Hashed immutable assets plus HTTP cache give nearly the same repeat-visit benefit. A service-worker shell would risk serving stale HTML without the fresh inline snapshot. Revisit after RUM data.

### 9.2 Scene composition and responsive layout

**Layers**, back to front:

1. Sky gradient, with weather tint.
2. Far foliage.
3. Mid foliage and perch structure (the three zones).
4. Birds and offer items.
5. Rain, leaves, and feathers.
6. Occasional foreground branch.
7. Color-grade overlay (time of day, settle).

Captions and focus rings are DOM overlays positioned from the scene transform.

- **Coordinates.** Scene units: a 1600 × 900 logical design space. Perch slots are defined in normalized x ∈ [0, 1] per zone, with zone depths and scales (front 1.0, middle 0.85, back 0.7).
- **Layout solver** (runs on resize and orientation change):
  - The *bird band* is the vertical region containing all perches. The top-bar band (48 px) is excluded, so birds never sit under chrome.
  - **Narrow viewports:** compress slot x-spacing down to a minimum inter-bird gap of 0.9 × bird width; if still too wide, scale the whole band down to a floor of bird height ≥ max(36 px, 7% of viewport height). No bird is ever cropped: the solver verifies every slot's bounding box is inside the viewport and scales down until it is (`aviary_layout`).
  - **Portrait phones:** the band is centered vertically, with sky extended above and ground or foliage extended below using procedural fill. No letterbox bars.
  - **Ultra-wide:** perch spacing is capped at 1.8× the design spacing, and extra width goes to extended foliage margins, so birds never feel scattered.
- **Backing store:** DPR is capped at 2, with a total backing-store budget of 4.5 MP; above that, render at reduced DPR and let the browser upscale. Adaptive quality can lower it further (§9.9).
- **Lighting:** a sun-elevation proxy comes from the aviary timezone's representative coordinates (compact table built from `zone1970.tab`) plus the day of year, using NOAA's approximate solar position via lookup tables. That gives seasonal day length and correct hemispheres without asking for location. Polar edge cases clamp to a minimum 4 h twilight band. Palette keyframes (night, dawn, morning, midday, afternoon, golden hour, dusk) in OKLCH are interpolated by elevation and weather. The grade is recomputed every 5 s, never per frame. Atlases re-rasterize when the grade moves more than ΔE_OK 0.01 (about every 20–60 s at dawn and dusk, rarely at midday).
- **Parallax:** a slow autonomous camera drift of ±3 px over about 40 s, applied at 0.3× to far foliage and 1.2× to the foreground. It is never tied to the pointer or device tilt (subtle, not showy). It is disabled in reduced motion.

### 9.3 Bird rig and plumage

- **Cut-out rig** per species: body, head, beak (upper and lower), eye with lid, wing (folded and spread variants), tail, legs, and optional crest. Parts are drawn with `Path2D` from species Béziers and rasterized **once per bird** into an `OffscreenCanvas` atlas at the current lighting grade and plumage. Each frame composes about 10–12 `drawImage` calls per bird with affine transforms. Seven birds cost about 80 draw calls, well within budget on integrated GPUs.
- **Plumage:** the snapshot carries per-region OKLCH colors (`plumage.regions`) and `detail_level ∈ {0…7}`. Both are compiled server-side from species palette × plumage-saturation expression and quantized. The trait never appears on the wire (I-1). A higher detail level adds wingbar crispness, feather-edge highlights, and subtle sheen passes. When the snapshot changes plumage, the atlas cross-blends over 10 s, invisible at the drift scale.
- **Pose parameters:** body pitch, crouch, fluff (contour scale), head yaw, pitch and tilt, beak gape, eyelid, wing fold/spread and flick, tail angle and flick, and weight shift. Beak gape is driven by the audio engine's per-voice envelope, delayed by `AudioContext.outputLatency` so it stays in sync over Bluetooth.

### 9.4 Idle micro-motion director

Animation runs in three layers per bird, blended additively:

1. **Base (always on):** breathing at 0.25–0.45 Hz with 1/f jitter on amplitude and period, weight micro-shifts, and blinks as a Poisson process (mean 4 s, never periodic).
2. **Attention layer (head and eyes):** orients toward the highest-salience stimulus on a shared **stimulus bus**: another bird's call (the "head-tilt toward sounds" behavior), a passing leaf (curious birds track it), an offer item, a newcomer, or the viewer (greeting glances, and ×(0.5 + 0.5A) glances while attended). Saccade-like head turns take 120–250 ms, holds 0.6–4 s.
3. **Action layer:** utility-based selection among `preen`, `scan`, `shuffle`, `fluff`, `wing-stretch`, `hop-in-place`, `forage-peck`, `rest`, and `tail-flick`. Weights come from continuous affect and the server's activity hint (§7.7): wary birds scan more and sit tight, content birds preen, curious birds tilt and track, drowsy birds sink low and fluffed. Each action is a parametric curve with seeded noise (duration 2–20 s, amplitude, and speed all drawn per instance). There are no canned clips, and no action instance repeats its parameters.

Motion is continuous and never reads as paused. All periodic components carry randomized period jitter (≥ 15%). No element flashes more than three times a second (WCAG 2.3.1), and no micro-animation runs above 3 Hz except wingbeats in flight.

### 9.5 Return-greeting choreography

1. **Return detection.** The `AttentionMonitor` enters `attended` (visible ∧ focused) after at least 20 s unattended, or it is a fresh navigation. At most one greeting per 2 minutes per device.
2. **Absence length:** `now − max(local_last_attended_end, snapshot.last_presence_end_at)`. This is account-level, so presence on the phone ten minutes ago makes the laptop return "short".
3. **Buckets and forms:**

| Absence | Primary greeter forms (weighted, then procedurally parameterized) |
|---|---|
| 20 s – 10 min | glance up (head turn toward viewer, hold 0.6–1.5 s), or a small tilt |
| 10 min – 6 h | glance + soft 1–2-note call (greeting production), or tilt + one step forward |
| 6 h – 3 d | a step or hop toward the front (amplitude = `approach_amp`), a longer call, sometimes answered by a second bird |
| > 3 d | re-orientation: hop to a front slot if free, a longer call, a second bird's response, and a brief attentive hold |

4. **Selection.** Weighted sampling over `greet_weight` using **fresh entropy** (`crypto.getRandomValues`), because greetings must never be identical twice (`interactions`). A roosting aviary at night yields an eye-opening and a small shuffle from one bird, or a call from the nocturnal species.
5. **Timing.** The primary greeter starts 600–1800 ms after the first frame or return. Additional greeters join with probability ∝ A, staggered by U(1.2, 5.0) s, at most two. They never fire in unison.
6. **Variation guarantee.** Every greeting is a composition of continuous parameters: gaze duration, tilt angle, step count and distance, call expansion, and whether a wing flick or fluff occurs. A hash of the parameter vector is checked against the last 50 greetings on the device; a collision re-draws.
7. **Report.** `greeting_observed{first_bird_id, order, absence_bucket, approached}` feeds notebook facts and the canonical front-slot move (§7.8).

### 9.6 Transitions

- **Flights:** a cubic Bézier arc with an apex above both perches. Wingbeat is 8–12 Hz while flying (the only fast motion), with a glide on descent and a landing flare plus settle-wobble. Duration comes from the snapshot or 1.4–3.0 s when synthesized.
- **Offer items:**
  - **Seed:** three to seven seeds fall onto the front ledge with small bounces.
  - **Pool:** a shallow reflective ellipse fades in over 1.5 s with a sky reflection and 0.2 Hz ripples. Bathing adds splash particles from a fixed pool.
  - **Song fragment:** a faint stir in the nearest leaves as it plays. Nothing text-like.

  Approachers follow the plan's delays.
- **Settle** (from the gesture popover):
  1. The grade ramps to the evening palette over 4 s, and the audio engine lowers the master level by 9 dB and call rates to 0.3× over the same 4 s.
  2. **Undo:** any click or tap in the aviary within 5 s reverses the ramp over 1.5 s. The `settle` event is sent only if the window elapses without undo.
  3. **Settled state:** the tab remains settled until close or active re-engagement (click, tap, or keydown; not pointermove). Re-engagement ramps lighting back to local time over 3 s, restarts the presence window, and qualifies for a short greeting.
- **Top bar fade:** after 3.5 s of cursor stillness, fades to 8% opacity over 1.2 s; restores over 250 ms on pointer movement or any key. Keyboard focus inside the bar pins it at full opacity. On touch devices, any touch restores it.

### 9.7 Reduced-motion register

This is a separately designed renderer mode, switched live via `matchMedia('(prefers-reduced-motion: reduce)')` or the account override.

| Normal | Reduced motion |
|---|---|
| Continuous idle actions | Each action is 2–4 authored **key poses**, cross-faded over 1.2–2.5 s, with holds of 6–20 s. The bird is drawn twice during a cross-fade (budgeted). |
| Breathing and blink micro-motion | Removed. Blinks become eyelid pose cross-fades at a slower rate. |
| Flights | Cross-fade out at perch A (1 s), cross-fade in at perch B (1 s), overlapping by 0.3 s |
| Greeting | Pose cross-fade to "looking at viewer" plus call. A step becomes a cross-fade to a nearer pose. |
| Leaves, feathers, parallax, rain streaks | Removed. Rain becomes a soft static veil and darker grade, cross-faded. |
| Day/night and settle grade | Kept, at 1.5× slower ramps |
| Offer items | Seed and pool fade in. Responses are pose cross-fades. |
| Top bar fade | Kept (opacity only), 1.5× slower |

Calls, drift, mood, the notebook, narration, and captions are unchanged. The designer owns a **reduced-motion pose library per species** as a v1 deliverable, not a later fix. **The motion registry** requires every animation to declare its reduced-motion counterpart. Code can't register an animation without one, a type-level requirement that stops future regressions.

### 9.8 Chrome: top bar, popovers, panels

- **Top bar** (DOM, Preact): exactly four icon buttons with accessible names: `Account and settings`, `Accessibility settings`, `Field Notebook`, `Offer`. When audio is autoplay-blocked or muted, the accessibility icon shows a subtle speaker-off glyph (D-14). There are no badges.
- **Gesture popover** (Offer icon, shortcut `O`): *offer a seed*, *offer a song fragment* (submenu of 8 fragments with naturalist names), *offer a still pool*, a separator, *settle for the evening*. While a newcomer is visiting it also shows *make room for the {epithet}*. After one choice it closes. The client paces offers at one per 20 s by quietly disabling the offer items with `aria-disabled` and no message (D-11).
- **Field Notebook panel:** a right sheet on desktop, a bottom sheet on mobile, and the aviary stays alive behind it. The list is windowed, with at most about 30 entry nodes in the DOM, and recycled nodes drop references on scroll-out (the memory rule). Entries are grouped by lowercase weekday and date. A humanist serif webfont (~25 KB) is lazy-loaded with the panel (`font-display: swap`).
- **Settings, accessibility, and visits:** code-split overlay sheets in the matter-of-fact register.
- **No chrome inside the scene:** no tooltips (a lint rule bans `title` attributes on scene elements), no hover states on birds, no labels.

### 9.9 Frame loop, adaptive quality, memory

- **Per frame:** time and offset → scheduled transitions → director update (≤ 7 birds) → pose → draw list → Canvas 2D backend. Budget per frame: ≤ 4 ms scripting, ≤ 6 ms raster on the reference laptop.
- **Adaptive quality ladder,** stepped when frame p95 > 16.7 ms over a 10 s window and relaxed after 60 s of headroom:
  1. drop the foreground branch blur;
  2. halve particle caps;
  3. reduce DPR by 0.25 steps to 1.0;
  4. go to a 30 fps cadence with motion-blur-free interpolation.

  Bird motion quality is never reduced before particles.
- **Allocation discipline:**
  - Particles live in struct-of-arrays `Float32Array` pools (leaves 24, feathers 6, rain 400, splash 64).
  - Draw lists are reused arrays and poses are preallocated.
  - There are no closures per frame.
  - Atlases are bounded: 7 birds × 1 atlas plus 1 newcomer, replaced in place.
  - An ESLint rule flags allocation syntax inside `@hot` functions.
- **Memory soak gate** (§14.2): 30 minutes, heap slope ≈ 0 after 5-minute warm-up, and constant DOM node count, `AudioNode` count, and canvas count.

### 9.10 AttentionMonitor, hidden tabs, frame gaps

- **States:** `hidden`, `visible-unfocused`, `attended-idle` (visible ∧ focused, no activity within W), and `attended-active` (presence).
- **Inputs:** `visibilitychange`, `focus`/`blur` plus `document.hasFocus()` polling at 1 Hz (some browsers miss blur on OS-level switches), and activity (`pointermove`, `pointerdown`, `keydown`, `wheel`; D-07), throttled to one timestamp update per second.
- **Outputs:** presence heartbeats (only in `attended-active`), `presence_end`, return events for greetings, and top-bar fade timing.
- **Hidden:** cancel rAF, since there is nothing to see and it wastes battery. Fade audio and suspend it (D-13). Flush events. The simulation continues on the server.
- **Visible:** draw immediately with locally correct lighting, fetch the snapshot, and reconcile (§8.3). A frame gap over 5 s is handled the same way.

---

## 10. Audio pipeline

### 10.1 Graph (built once; no per-call nodes)

```
AudioWorkletNode "aviary-synth"  (8 stereo outputs: 7 bird buses + 1 "offer/newcomer" bus;
│                                  plus 1 ambient-bed output for rain/wind noise)
├─ bus[i] → Gain(listen-in) → BiquadFilter(lowpass: distance + attention) → StereoPanner(x) ─┐
│                                                   └─ send → Convolver(procedural IR) ──────┤
└─ ambient bed → Gain(weather level) ─────────────────────────────────────────────────────┤
                                                                                           ▼
                                   Ambient bus Gain → DynamicsCompressor(2:1, −18 dB) → Master Gain
                                   (mute, settle, time-of-day) → limiter(WaveShaper soft clip @ −3 dBFS)
                                   → destination
```

- All nodes are created at audio-module init and persist for the session. The node count is constant, and the soak test asserts it (§14.2).
- **The reverb IR is generated procedurally at startup:** 0.9 s of exponentially decaying, band-filtered noise, synthesized into an `AudioBuffer` once. It is not a recording (I-10).
- `latencyHint: 'playback'` gives larger buffers, fewer glitches, and battery savings; interactivity needs are modest.

### 10.2 The synth (AudioWorklet processor)

- **Voice pool:** 16 preallocated voices. Stealing takes the oldest, quietest voice. `process()` never allocates: messages from the main thread are decoded into preallocated structs, and parameter curves live in fixed `Float32Array` rings.
- **Voice model (syrinx-inspired):** two independent sources per voice, because songbirds can produce two simultaneous pitches. Each source is:
  - a carrier (sine or 2-operator FM with a time-varying index),
  - optional band-limited noise through a state-variable filter (buzzes, chips, churrs),
  - an amplitude envelope with curved attack, decay and release, plus a slight pitch overshoot at onset,
  - a frequency contour from piecewise-cubic control points,
  - AM/FM for trills (12–25 Hz) and vibrato, and spectral tilt for timbre.
- **Pitch registers:** species registers are mapped for comfort on laptop and phone speakers. High species 1.2–5.5 kHz (avoiding piercing 7–9 kHz content), low species 0.5–2 kHz, and the nocturnal churr as low pulsed noise-FM.
- **Scheduling:** sample-accurate. Every note event carries `when` in `AudioContext.currentTime` seconds. The processor reports per-voice envelope levels at 60 Hz back to the main thread to drive beak gape.

### 10.3 Call-grammar expansion (client half)

Each species grammar (`packages/voice`, data from `packages/species`) is a small probabilistic grammar:

```
CALL    → INTRO? BODY CODA?
BODY    → SIGNATURE MOTIF{0..3}        # the bird's own signature appears in ≥70% of calls
MOTIF   → chip | whistle | rise | fall | two_note | trill | buzz | warble | churr (nocturnal)
GREETING→ soft_note{1,2} | SIGNATURE (long absence: SIGNATURE MOTIF{1,2})
ALARM   → sharp_chip{3..6}
```

- **Production weights** come from mood and the compiled `calling` parameters: wary birds use short, sharp, low-repetition calls; content birds use longer melodic calls; curious birds use rising inflections; drowsy birds use single low soft notes; alert birds use clear repeated phrases.
- **Identity parameters** are applied to every expansion and never vary: pitch offset, signature motif shape, timbre fingerprint, rhythm template.
- **Per-call variation:** ±30 cents per note, ±8% timing, ±2 dB per note, repetition count, and ornament probability.
- **Output:** a `CallPlan` of notes, duration, and caption descriptors, sent to the synth and to captions from the same object (I-10, `accessibility_perf`: "Each call's caption matches what was actually played").
- **No exact repeats.** Each call hashes its parameter vector against the last 200 calls of that bird and re-draws on collision. This is a guard; collisions are practically impossible given the continuous jitter.

### 10.4 Scheduling

- **Ambient calls:** per bird, inter-call intervals are exponential with mean `1/rate_per_min`, drawn from `Philox(hash(bird_id), window_index, call_index)`. Two devices attending at once therefore schedule the same calls at the same corrected times, and a visitor hears what the host hears.
- **Call and response:** when bird X calls, each other bird Y answers with probability `respond_p × proximity` after 0.4–2.5 s. Its sub-seed derives from X's call index, so devices agree. Responses to responses are damped (×0.4 per generation) so exchanges end naturally.
- **Chorus windows** multiply rates and allow overlap. Outside choruses, different birds' onsets are kept at least 120 ms apart, with at most three simultaneous calls. Two birds calling at once still produce a real mix of independently synthesized voices, never stacked loops.
- **Reactive calls** (greeting, offer response, the focused bird acknowledging listen-in with p = 0.5 within 2–5 s) use fresh entropy.
- **Song-fragment offer:** 8 authored fragments of 3–6 notes, played with a soft human-whistle timbre deliberately distinct from any bird. Join-in responders answer with their *signature motif* snapped to the fragment's pitch classes and tempo. Call-against responders overlap with alarm-like chips. Go-quiet responders have their rate suppressed for 3 minutes.
- **Look-ahead:** the main-thread scheduler runs every 50 ms while visible and schedules 200 ms ahead into the worklet. It is paused while hidden (D-13).

### 10.5 Listen-in mix

| Parameter | Engage | Disengage |
|---|---|---|
| Focused bus gain | +4 dB, `setTargetAtTime` τ = 0.8 s (~95% at 2.4 s) | → 0 dB, τ = 1.2 s |
| Other buses | −10 dB, τ = 0.8 s. **Floor −14 dB: never silent.** | → 0 dB, τ = 1.2 s |
| Other buses' lowpass | 12 kHz → 4.5 kHz, τ = 1.0 s ("attention narrowing") | → 12 kHz |
| Focused bird's proximity foley | Procedural feather rustles and beak clicks at −24 dB, synced to preen and shuffle actions | Off |

- Switching focus between birds cross-ramps both buses at once. There is never a hard cut, which would read as "switching channels".
- Disengage triggers: re-click on the focused bird, focusing another bird, a click on empty scene space, keyboard focus leaving the bird group, `Escape`, or the tab becoming hidden (`interactions`).

### 10.6 Chorus mixing and loudness

- **Per-call gain normalization:** each `CallPlan` has an estimated RMS, and the voice gain is scaled so every bird sits in a ±3 dB loudness window before mood-driven loudness is applied.
- **Depth:** back perch −4 dB, lowpass at 7 kHz, and a larger reverb send; front perch drier and brighter. Pan position = bird x mapped to [−0.7, 0.7], so the chorus has a real spatial layout that helps recognizability.
- **Bus compression** (2:1 above −18 dB, 10 ms attack, 250 ms release) and the soft-clip limiter prevent chorus build-up from clipping.
- **Master level** tracks time of day (evening −3 dB, night −6 dB) and settle (−9 dB).
- **The ambient bed** (rain or wind noise, procedural) plays only during weather events, at −30 to −22 dB.

### 10.7 Lifecycle

| Situation | Behavior |
|---|---|
| Boot | Create the context when the audio module loads and call `resume()`. If `running` (for example Chrome with high media engagement), calls are audible within about 1.5 s of first paint. |
| Autoplay-suspended | No prompt and no button in the scene. The top-bar accessibility icon shows the speaker-off glyph. On the first user activation (`pointerdown`, `keydown`, `touchend`), resume and fade in over 2.5 s, like the room coming into earshot. `pointermove` does not count as activation in browsers, so a watch-only user on Safari or Firefox hears calls from their first click (§18, risk A-3). |
| Hidden | Fade out over 2 s, then `suspend()` (D-13) |
| Visible again | `resume()` and fade in over 1 s |
| Mute (accessibility panel, shortcut `M`) | Fade master to 0 over 0.6 s, then suspend. Emit `audio_state` (D-03). Captions keep working if enabled. |
| Multiple tabs | Only the elected leader tab is audible (§8.5) |
| iOS | Keep the default audio session, which respects the hardware silent switch. The accessibility panel (system register) says "On iPhone, calls are silent while Silent Mode is on." Handle the `interrupted` state (phone calls) by resuming on return. |

### 10.8 Captions

- **Composition:** the caption grammar in `packages/prose` builds the text from `CallPlan` descriptors: dynamics (soft, low, sharp, clear), count words, motif nouns (rise, trill, whistle, chip, churr), structure (pauses, repeats), and a location phrase from the bird's zone. The results read like "a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch". It always matches the call that was actually played.
- **Display:** captions appear only when enabled (D-22), as DOM text anchored above the calling bird and clamped inside the viewport. They fade in over 200 ms at onset, hold for the call duration plus 1.2 s, and fade out over 400 ms. At most three are visible. A dense chorus collapses into one aggregated caption such as "several birds calling together, the high whistle loudest".
- **Accessibility:** captions are `aria-hidden="true"`. Screen-reader users get calls through narration (§11.1), which avoids double speech. Contrast comes from a dynamic scrim (§11.5).

### 10.9 WebAudio fallback

If `AudioContext` is missing, construction throws, `audioWorklet.addModule` rejects, or the context errors, the client enters **silent mode**:

- Captions switch on by default; the user can still turn them off.
- The scheduler keeps running and drives captions, beak motion, and narration of calls, so the aviary is still socially alive and visibly calling.
- There is no recorded-audio path of any kind (I-10).
- The aggregate metric `audio_unavailable_total{reason}` counts these cases.

### 10.10 Sound-design process and listening gates

Staffing: one sound designer and one audio engineer from M0. Tooling: a lab "grammar explorer" for auditioning species grammars, voice seeds, and moods, on synthetic birds only. Gates, run as remote listening panels of 30+ people on both headphones and laptop speakers:

| Gate | Threshold | When |
|---|---|---|
| Naturalness ("a real bird in a real place"), MOS 1–5 | ≥ 3.8 per species | M1 exit, re-run per species change |
| Recognizability: name the calling bird in a 20 s chorus after 10 min of familiarization | ≥ 85% at 2–4 birds. **≥ 80% at 7 birds (chance 14%).** | Gate for each `ops.max_birds` raise (§16.2) |
| Identity across moods and drift | Same bird judged "same" ≥ 85% | M2 |
| Non-repetition over 30 min | Detection of repeats no better than chance (ABX) | M2 |
| Comfort over 30 min | ≥ 4/5, with no complaints of piercing frequencies | M2 |

---

## 11. Accessibility surfaces

The bar is that **every user gets the actual product**. Accessibility work is budgeted into every milestone, and v1 cannot launch without it (§16.3).

### 11.1 Screen-reader narration

**Architecture (client):** `NarrationComposer → NarrationQueue → live region`.

- **Live region:** `<div id="narration" aria-live="polite" aria-atomic="true" class="visually-hidden">`. It is present in the initial HTML so assistive tech registers it before the first update. We never use `assertive`.
- **Composer inputs:** the same scene state the renderer draws from (bird zones, current idle action, affect-derived posture, weather, lighting phase, calls just played via `CallPlan` descriptors) plus the stimulus bus and interaction events. It scores candidate observations: change since the last utterance, rotating attention across birds, and a scene summary when nothing changed.
- **Cadence:**

| Utterance type | Rule |
|---|---|
| Idle | One every U(30, 60) s. The queue holds at most one pending idle item (newer replaces older). |
| Events: return greeting, offer reaction, settle, weather onset, newcomer arrival | Prompt. Items within a 1.5 s coalescing window become one sentence. Priority over idle. |
| Global | ≥ 10 s between any two utterances. Nothing while hidden. When settled, one settle sentence, then idle every 90–120 s. |
| User setting | Narration pace: *calm* (default) / *quieter* (90–180 s idle) / *events only*. System register. |

- **Content rules** (enforced by grammar constraints and a CI lint over 10⁴ sampled renders):
  - Naturalist, lowercase, present tense.
  - Names are the primary referent; a species epithet is used on first mention per session and occasionally after.
  - Describe **observable behavior and posture**, never mood labels ("wary"), numbers, perch indexes, or trait words ("boldness").
  - Calls are described consistently with captions.
  - Examples:
    - *"it is early morning in the aviary; the light is thin and gold. pip glances up from the front rail and calls twice, softly."*
    - *"wren keeps to the back perch, feathers fluffed, watching the far leaves."*
    - *"a light rain begins. both birds tuck in under the high branch."*
- **Recency memory:** no phrase atom repeats within the last 20 utterances. Sentence openings rotate.
- **Visual narration (D-23):** when enabled, the latest utterance also appears in a quiet text strip along the scene's bottom edge, with WCAG AA contrast via scrim (§11.5).

### 11.2 DOM semantics and the keyboard model

- **Structure:**

```
<header>  <div role="toolbar" aria-label="Aviary controls">  4 buttons  </div> </header>
<main>
  <div role="group" aria-label="the aviary" aria-describedby="scene-summary">
    <canvas aria-hidden="true">
    <button class="bird-proxy" aria-pressed="false">pip</button>   … one per bird, roving tabindex
  </div>
  <div id="narration" …>  <div id="scene-summary" class="visually-hidden">
</main>
```

- **Bird proxies** are transparent buttons tracked to each bird's hull each frame (transform only, no layout), with hit areas ≥ 44 × 44 CSS px. Accessible name: the bird's name. Accessible description: the naturalist posture sentence, refreshed only on focus and then at most every 10 s. `aria-pressed` reflects listen-in.
- **Keys:**

| Key | Action |
|---|---|
| `Tab` / `Shift+Tab` | Moves through the four top-bar buttons, then into the aviary group (one tab stop). Entering focuses the first bird (leftmost by x). |
| `←` `→` | Previous or next bird by x-order. Moving focus to another bird disengages listen-in (`interactions`). |
| `↑` `↓` | Nearest bird in the zone behind or in front |
| `Home` / `End` | First or last bird |
| `Enter` / `Space` | Toggle listen-in on the focused bird |
| `Escape` | Exit listen-in (focus stays). In popovers or sheets, close and return focus to the invoker. |
| `O` `N` `M` `A` `?` | Gesture popover, notebook, mute, accessibility settings, shortcut help. These are single-key shortcuts, active only when focus is not in a text field, and can be turned off in accessibility settings (WCAG 2.1.4). |

- **Popovers** use `role="menu"` with arrow navigation. **Sheets** are dialogs with focus trap, `aria-modal`, and focus restore. The notebook is an `<ol>` of `<article>`s with day headings, so screen-reader users can navigate by heading.
- **Pointer and touch:** clicking or tapping a bird (canvas hit test forwarded to its proxy) toggles listen-in. Clicking empty scene space disengages it and shows the top bar.

### 11.3 Captions and reduced motion

- **Captions** are covered in §10.8. Settings: captions on/off and caption size normal/large (1.25×).
- **Reduced motion** is covered in §9.7. Setting: *follow system* (default) / *on* / *off*.

### 11.4 Other WCAG 2.2 AA commitments

- **Target size** of at least 44 × 44 CSS px for top-bar buttons and bird proxies (exceeds 2.5.8).
- **Reflow** at 320 CSS px and 400% zoom for all system surfaces and the notebook (1.4.10). The scene itself scales by its layout solver.
- **Forced colors:** `@media (forced-colors: active)` maps chrome to system colors, and focus uses the proxy `outline` in `Highlight`. The canvas still renders as imagery.
- **No color-only signals** (1.4.1): listen-in is conveyed by `aria-pressed` and audio.
- **Visitors** get narration, captions, and reduced motion. Their top bar contains only the accessibility button.

### 11.5 Contrast and focus visibility

- **Top bar:** at full opacity, icons sit on a subtle translucent scrim that guarantees ≥ 3:1 non-text contrast (1.4.11) in every lighting and weather state. An automated test renders 12 lighting × 3 weather × 2 settle states, samples the 5th and 95th luminance percentiles under each icon, and asserts the ratio. When faded to 8% opacity the controls are visually dormant; any pointer or keyboard activity restores full opacity before interaction.
- **Captions and the narration strip:** a dynamic scrim. The client samples luminance percentiles behind the text box from the cached background layers, which is cheap, and picks light or dark text plus a scrim alpha that achieves ≥ 4.5:1 against the worst-case percentile.
- **Focus indicator:** a two-tone ring (light inner 2 px, dark outer 2 px) around the bird's hull via the proxy outline, plus a soft canvas glow. It keeps ≥ 3:1 against adjacent colors in every state and appears on keyboard focus only (`:focus-visible`), never as a mouse "highlight".

### 11.6 Accessibility QA

| Layer | What | When |
|---|---|---|
| Automated | axe-core on all DOM surfaces. Live-region rate test (≤ 2 idle utterances/min). Narration content lint (no mood labels, digits, or trait words). Keyboard traversal E2E. Contrast matrix. | Every PR |
| Manual | NVDA + Firefox, JAWS + Chrome, VoiceOver (macOS Safari, iOS), TalkBack (Android Chrome) scripts | Every milestone |
| External | Independent audit | M3 |
| Lived-experience panels | Paid sessions with screen-reader users, users with vestibular disorders (reduced motion), and Deaf and hard-of-hearing users (captions). Scored on a **charm rubric** ("did the aviary feel alive?", 1–5) as well as task success. | M2 and M3 |
| Launch gate | Zero open critical or serious issues. Charm rubric ≥ 4/5 median for screen-reader and reduced-motion cohorts. | GA |

---

## 12. Voice and copy system

### 12.1 Registers

| Register | Surfaces | Rules |
|---|---|---|
| **naturalist** | Aviary scene, gesture popover labels, notebook, narration, captions, adoption and newcomer naming sheets | Lowercase, including bird names in generated prose (as in the PRD samples: "pip greeted before wren"; D-26). Present tense. Specific. No `!`. No second person in notebook, narration, or captions. Numbers as words. Bird verbs preferred (notice, perch, settle, listen in, offer). |
| **system** | Sign-in, account, sessions, settings (including accessibility and bird rename), visits management, invite and landing pages, errors, unsupported browser, privacy, all emails | Normal sentence case, direct, and actionable. No naturalist phrasing and no warmth standing in for information. |

A new surface that engages the system *as a system* (identity, errors, settings, money) is `system` by default (`product_brief`).

### 12.2 Copy registry and lints

`packages/copy` holds every static string as `{key, register, text, owner, notes}`. System strings use ICU MessageFormat and are i18n-ready. Naturalist text comes from grammars (§12.3), never from ad-hoc strings. The CI lints are:

- **Register lints:** the rules in the table above.
- **Banned vocabulary** in all registers, with case-insensitive stems: *achievement, unlock, level, streak, badge, points, score, reward, xp, rank, congrat, welcome back, you've been, days in a row, keep it up, don't forget, miss(ed) you, come back*.
- **Terminology** (`concepts`): *creature, pet, animal, character* → bird. *song, chirp, noise* → call, with the single allowed compound "song fragment". *solo, select, highlight, pin* → listen in.
- **Announcement guard:** no string in the registry may be attached to toast, snackbar, or banner UI. Those components don't exist (I-2).

### 12.3 The prose grammar engine (shared client and server)

- **Format:** weighted alternatives, typed slots (bird, place, count, time-phrase, weather, call-descriptor), agreement helpers (a/an, plural, count words), constraints (e.g., `place` must match the bird's zone), and recency memory to avoid repeats.
- **Deterministic** given a seed. Notebook entries store `variation_seed`, so their wording is frozen while names resolve at read time (D-10).
- **Authoring:** a staff writer with a naturalist consultant. Volume targets: notebook ≥ 40 templates per detector with ≥ 10⁴ distinct surface forms overall; narration ≥ 600 phrase atoms; captions: full coverage of every motif descriptor combination.
- **Review:** every grammar PR attaches 200 random renders for writer sign-off, and the lints run on 10⁴ samples.
- **Place vocabulary** (fixed, scene-consistent): *the front rail*, *the low perch*, *the middle branch*, *the high branch*, *the back perch*, *the far branch*.

### 12.4 Key product copy (naturalist)

- **Adoption sheet:** "two birds have come to the aviary." Each bird is shown alive in a small vignette with its name field prefilled from species-appropriate suggestions (*pip, wren, sorrel, tamsin, moss, lark…*), plus a lowercase confirm button: "these are their names". A system-register helper line reads "You can rename them later in settings."
- **Gesture popover:** *offer a seed* · *offer a song fragment* · *offer a still pool* · *settle for the evening* · (visiting) *make room for the streaked bird*.
- **Song fragments:** *a rising three-note phrase*, *a slow falling whistle*, *two notes, then a pause*, …
- **First notebook entry (ArrivalDay):** "two birds arrived today. pip took the front rail first; wren stayed back, watching."

---

## 13. Accounts, security, and privacy mechanics

### 13.1 Sign-in (magic link)

- **Token:** 256-bit random, base64url, only in the email link. The server stores `SHA-256(token)`. Lifetimes are 15 min for sign-in and 24 h for email change. Consumption is a single atomic `UPDATE … WHERE consumed_at IS NULL AND expires_at > now()`, so a used link is invalidated immediately.
- **Email** (system register): "Sign in to Pocket Aviary. This link expires in 15 minutes and works once. Requested from Firefox on Windows at 7:42 AM. If you didn't request it, you can ignore this email." It is sent from a dedicated subdomain with SPF, DKIM, and DMARC `p=reject`. A second provider is kept warm for failover. We monitor delivery latency p95 ≤ 30 s from provider webhooks, aggregated only.
- **Scanner-safe:** the GET renders a landing page and the POST consumes (D-25).
- **Uniform responses** to link requests prevent account enumeration.

### 13.2 First sign-in saga (account → aviary → starters)

1. `consume` creates the `accounts` row (identity-db) with a fresh DEK. Idempotency key = `link_id`.
2. An internal RPC `aviary.create(account_id)` (idempotent on `account_id`) creates `aviaries`, `aviary_state`, `aviary_schedule`, and two birds:
   - **Species:** two distinct, non-nocturnal species with different call registers, chosen deterministically from `rng_key`.
   - **Seeds:** voice seeds with the distinctness check (§7.11), appearance seeds, trait seeds, and ceilings (§7.5).
   - Both birds start `arrival='awaiting_naming'`.
3. If step 2 fails, a job retries it. Meanwhile the client shows the quiet field and polls the snapshot.
4. The client shows the adoption sheet. `POST /api/aviary/adoption` stores names, sets `arrived`, and the fly-in plays (§9.1). A second device opened before naming shows the same sheet. After naming, other devices simply show the birds present.

### 13.3 Sessions

- Tokens rotate on use every 24 h (sliding), with 60-day idle expiry and a 1-year absolute limit (D-21).
- Revocation propagates to regional auth caches in under 5 s via pub/sub. Every API call validates the session.
- The session list shows `ua_label`, the "last active" bucket, and "this device". No IP or location is ever stored or shown.

### 13.4 Email change

A verification link goes to the new address. The old address keeps signing in until the link is consumed. On consume, `email_ct` and `email_bidx` swap atomically, all *other* sessions are revoked as a security precaution, and a notice goes to the old address (D-20).

### 13.5 Export

- A job builds the JSON, encrypts it with SSE-KMS in object storage (7-day lifecycle), and emails "Your Pocket Aviary export is ready" with a link. Downloading requires a signed-in session for the same account.
- The schema is `export_version 1`: `account {email, created_at, timezone, settings}`, `aviary {created_at}`, `birds [{bird_id, name, species, adopted_at, personality {boldness, social_warmth, vocal_frequency, plumage_saturation, curiosity}: decimals 0–1, mood}]`, `notebook [{date, text}]`, `visits {invitations, log}`.
- A top-level `about` string (system register) explains that the values are the simulation's stored state (D-01). The export is never rendered in-product.

### 13.6 The complete inventory of emails the product sends

All are transactional and system register. There is **nothing about aviary content, ever** (`product_brief`: not a notification surface).

1. Sign-in link
2. Email-change verification
3. Email-change notice to the old address
4. Deletion scheduled (with the date and "sign in to keep your account")
5. Export ready
6. Visit invitation (to the visitor)
7. Visit notification (opt-in, off by default)

There are no marketing, digest, re-engagement, or "we miss you" emails. Adding an email type requires a PM, privacy, and design sign-off recorded in the decisions log.

### 13.7 Deletion

- **Soft (day 0):** status becomes `pending_deletion` and invitations are suspended. Sessions remain, so the user can sign in and see the "I changed my mind" page. The aviary keeps ticking (D-19).
- **Restore:** status becomes `active` and unexpired invitations are un-suspended.
- **Hard (day 30):** the job deletes all aviary-db rows (birds, personality, state, events, notebook, observations, schedule), identity rows (account, sessions, links, invitations, visits, exports), and export objects. It destroys the DEK (crypto-shred) and records the UUID in `deletion_ledger`. A post-delete verification query must return zero rows; failures page. After any backup restore, the ledger is re-applied before traffic resumes (§5.7).
- **Visitor data-subject requests:** support can delete a visitor email from every host's invitations via the blind index, following a documented runbook.

### 13.8 Keys and application security

- **Keys:** KMS KEK rotated yearly, with DEKs re-wrapped lazily. Blind-index key rotation runs dual indexes during a migration window. TLS 1.3, HSTS preload, and encryption at rest on all stores.
- **Browser hardening:**
  - Strict nonce-based CSP: `script-src 'nonce-…' 'strict-dynamic'`, `worker-src 'self'`, `connect-src` limited to API and telemetry hosts, `frame-ancestors 'none'`.
  - Trusted Types enforced. Bird names and all user text render via `textContent` only.
  - COOP `same-origin`, CORP `same-site`, `Referrer-Policy: no-referrer` (visit and link URLs never leak via Referer).
- **Threat model** (reviewed at M1, pen test at M3):

| Threat | Mitigation |
|---|---|
| Link interception or replay | 15-min expiry, single use, scanner-safe GET |
| Session theft | HttpOnly, rotation, revocation list |
| Email enumeration | Uniform responses and timing |
| Invite spam via our domain | Fixed template with no custom message, rate limits, recipient complaint handling |
| Visit-pass leakage | The pass never appears in URLs after redemption, is browser-bound, and revocable |
| XSS through names | Text-only rendering |
| CSRF | Custom header plus Origin check plus SameSite |
| Supply chain | Lockfile, provenance checks, minimal dependencies, no third-party scripts at runtime |

### 13.9 Privacy policy (linked from account settings, plain text)

The policy names the aggregate categories we collect: request counts and latencies, error rates, anonymized session-duration histograms, render and first-bird timings, audio-pipeline error counts, and simulation-tick latencies. It states that per-bird interaction data (offers, listen-ins, presence) is used only to run the user's own aviary. That data is never aggregated, never used for training, never shared, and interaction events are deleted within about seven days of processing. The policy also describes export, deletion, visit-log retention, and the email inventory above.

---

## 14. Performance budgets and observability

### 14.1 Budgets

| Budget | PRD hard limit | Internal target | Gate |
|---|---|---|---|
| Initial JS at first paint (gzip) | < 2 MB | ≤ 350 KB total. First-bird critical path ≤ 110 KB. HTML ≤ 14 KB. | CI `size-limit`: fail over target +10%; hard fail at 2 MB |
| Time to first bird visible (mid-tier mobile, 4G) | < 500 ms | p75 ≤ 400 ms, p95 ≤ 500 ms (synthetic) | Synthetic fleet, alert on regression; launch gate |
| Idle motion frame rate (5-year-old mid-range laptop, 30-min session) | 60 fps | Frame p95 ≤ 16.7 ms, p99 ≤ 20 ms, ≤ 1 frame > 50 ms per 10 min | Nightly device lab; launch gate |
| Memory over 30 min | No growth | JS heap growth ≤ 1 MB after a 5-min warm-up (post-GC samples at 5, 15, 30 min). DOM nodes ±50. AudioNodes and canvases constant. | Nightly 30-min soak; 5-min PR variant |
| Snapshot size | "kilobytes" | ≤ 6 KB gzip at 7 birds | Protocol test |
| API | — | Snapshot p95 ≤ 80 ms server time. Events p95 ≤ 100 ms. | SLO |
| Simulation tick | p99 alarm at 5 s | p50 ≤ 50 ms end-to-end per aviary job | Alert |
| Audio thread | — | `process()` ≤ 25% of the render quantum at the p99 | Nightly lab |

**Reference hardware** (owned device lab):

- **Mid-tier Android:** Moto G Power (2022) class and Galaxy A14 class, Chrome.
- **"Five-year-old mid-range laptop":** Intel i5-1135G7 / Iris Xe / 8 GB (2021) on Windows with Chrome, Edge, and Firefox, plus a conservative 2019 i5-8265U / UHD 620 unit.
- **Macs and iPhones:** MacBook Air M1 (2020) with Safari; iPhone 12 with Safari.
- **4G reference network profile:** 9 Mbps down / 1.5 Mbps up / 85 ms RTT.

### 14.2 How each budget is measured

- **Bundle:** measured per build in CI. The report lists every chunk in the critical path.
- **First bird:** a synthetic fleet of scripted real browsers on the reference devices (or emulation where devices are unavailable) runs from 5 geographies (us-east, us-west, eu-west, ap-southeast, sa-east) every 15 minutes, covering cold, warm, signed-in, and visitor loads. RUM reports `first_bird_ms` as a histogram by `device_class` and `region`.
- **Frame rate:** scripted 30-minute sessions on the device lab cover idle, listen-in, all offers, rain, dawn transition, a 7-bird aviary, reduced motion, and settle. Frame timings come from tracing. RUM samples 1 in 10 sessions for a client-side frame-time histogram, plus `long-animation-frame` counts where supported.
- **Memory:** the CDP-driven soak forces GC and snapshots the heap. Instrumented wrappers count AudioNodes and canvases. A leak detector compares retained object types between the 15- and 30-minute snapshots.
- **Tick:** server histograms `tick_job_seconds{tier}` and `tick_lag_seconds{tier}`.

### 14.3 Telemetry catalog (allowlisted; I-9)

- **RUM beacon:** sent at most once per attended session plus on `pagehide`, to a separate telemetry origin with `credentials: 'omit'`. It has no cookies, no IDs, no session keys, and the collector strips the IP. Fields: `build`, `browser_family`, `device_class` (mobile/desktop × memory bucket), `region` (added by the edge), and histograms computed **client-side** into fixed buckets: `first_bird_ms`, `snapshot_inline_hit` (bool counts), `frame_ms`, `long_frames`, `audio_state_at_boot` (running/suspended/unavailable), `audio_errors`, `attended_session_minutes_bucket` (the anonymized session-duration histogram the PRD allows), `narration_live_region_errors`, and `js_errors_by_module`.
- **Server metrics:**
  - Requests: `http_requests_total{route,method,status_class,region}`, `http_request_seconds{route,region}`.
  - Events: `events_accepted_total`, `events_rejected_total{reason}`.
  - Engine: `tick_job_seconds{tier}`, `tick_lag_seconds{tier}`, `projection_total{result}`, `projection_mismatch_total`, `cas_conflict_total`.
  - Integrity: `trait_decrease_rejected_total`, `suspected_reset_total`, `integrity_audit_failures_total`.
  - Delivery: `snapshot_publish_failures_total`, `email_send_total{template,result}`, `email_delivery_seconds{template}`.
  - Jobs and capacity: `job_runs_total{job,result}`, `aviaries_total`, `warm_aviaries`.
  - Privacy: `log_pii_scanner_hits_total`, `telemetry_label_dropped_total{metric}`.
- **Traces:** sampled at 1%. Attributes are allowlisted and bodies are never captured.

### 14.4 What we deliberately do not measure

- Per-account engagement, visit frequency, streak-like series, retention cohorts, funnels, or DAU per account. Capacity uses the `warm_aviaries` gauge only.
- Distributions of traits, moods, perch behavior, or drift across accounts. This includes calibration: calibration uses simulation and consented staging studies (§7.5), never production data.
- Counts by interaction type (offers vs listen-ins), notebook reads, or feature usage per account.
- Visits per host, or any statistic that could rank or compare aviaries. These are not computed at all, so a leaderboard can never "just be exposed" (`social_optional`).
- Session replay, heatmaps, third-party analytics SDKs, or advertising pixels: banned by CSP and code review.
- A/B tests on engagement outcomes.

### 14.5 SLOs and alerts

| Signal | Objective / alert |
|---|---|
| Snapshot endpoint availability | 99.9% monthly |
| Synthetic first-bird p95 | Warn at 450 ms, page at 500 ms sustained 30 min |
| Tick latency p99 | **Alarm at 5 s** (PRD) |
| Warm tick lag p99 | Warn at 20 s, page at 45 s |
| `trait_decrease_rejected_total`, `suspected_reset_total`, `integrity_audit_failures_total` | Page on any non-zero value (sev-1: potential loss of the relationship) |
| `projection_mismatch_total` | Warn on any non-zero value |
| `log_pii_scanner_hits_total` | Page the privacy on-call |
| Magic-link delivery p95 | Warn at 30 s, page at 120 s |
| Sign-in consume error rate | Warn at > 5% over 1 h |

---

## 15. Test strategy (summary)

| Layer | Scope | Cadence / gate |
|---|---|---|
| Unit + property (fast-check) | Engine invariants (§7.16), prose grammar constraints, layout solver (no bird cropped for any viewport 320–5120 px wide), presence state machine truth table | Every PR; 10⁶-case nightly |
| Persona calibration harness | Appendix C bands | Every PR touching `packages/engine`; required for any `engine_version` bump |
| Cross-engine determinism | Golden hashes in V8, JSC, SpiderMonkey | Every PR touching `engine`/`rng`/`voice` |
| Contract | `packages/protocol` validators on both client and server. Snapshot has no trait keys (I-1). Visitor snapshot has no host-only fields. | Every PR |
| DB grants and triggers | Forbidden writes per role. Trait-decrease trigger. Bird cap trigger. No DELETE on notebook or birds. | Every PR (ephemeral Postgres) |
| E2E | Playwright on Chromium, Firefox, WebKit: sign-in, adoption, greeting, listen-in, offers, settle and undo, notebook, settings, visits, revocation, deletion and restore, export | Every PR (smoke), nightly (full) |
| Visual regression | Canvas renders at fixed seeds × 12 lighting × 3 weather × normal/reduced motion; pixel diff with perceptual tolerance | Nightly; designer approves baselines |
| Continuity | No teleports or snaps under adversarial snapshot sequences (§8.7) | Nightly |
| Performance | Bundle (PR), synthetic first-bird (every 15 min), device lab frame rate (nightly), soak (nightly 30 min, PR 5 min) | Gates in §14.1 |
| Accessibility | §11.6 | PR + milestone |
| Privacy | Log scrubber (emails injected into every code path), telemetry allowlist, PII scanner canary, network-path test (telemetry VPC cannot reach DBs) | PR + weekly |
| Sync chaos | Multi-device, crash injection, deploy skew (§8.7) | Nightly on staging |
| Migrations | Pre/post invariant harness on a production-shaped synthetic fixture: bird count, identity seeds, traits non-decreasing, checksums | Every migration PR |
| Copy | Register lints, banned vocabulary, terminology; writer review of grammar samples | Every PR |
| Human panels | Listening gates (§10.10), perception JND study (§7.5), lived-experience accessibility panels (§11.6) | Milestones |
| Surface audit | PM + design + writer walk every surface state against I-2 and I-3 (announcements, gamification, user-behavior facts) | Every milestone and before GA |

---

## 16. Rollout

### 16.1 Milestones and exit criteria (≈ 36 weeks to GA)

| Milestone | Weeks | Scope | Exit criteria |
|---|---|---|---|
| **M0 Foundations + spikes** | 0–4 | Monorepo, CI gates (bundle, lint rules, copy registry, grants test), both DB clusters, KMS, separate telemetry account and allowlist. Four spikes: (a) Canvas 2D cut-out renderer, 7 birds, reference laptop; (b) edge-streamed first frame on the reference 4G profile; (c) synth prototype for 2 species; (d) Philox and fixed-point cross-engine determinism. | (a) ≥ 60 fps p95, or WebGL2 backend promoted to the launch plan. (b) p95 < 500 ms. (c) MOS ≥ 3.3 with a credible path to 3.8. (d) Identical hashes. |
| **M1 Engine + vertical slice** | 4–14 | Engine v1 (drift, mood, weather, perch, attunement, offer plans, notebook detectors), persona harness, scheduler and workers, event ingest, snapshot compile/serve/projection. Client: first frame, scene, rigs and grammars for 2 species, idle director, greeting, listen-in, captions, narration v0, reduced motion v0, full keyboard model. Lab tools (synthetic only). | Staging demo with time acceleration. Budgets met at 2 birds. MOS ≥ 3.8 for 2 species. Perception study round 1 complete; G tuned. |
| **M2 Integrated alpha (dogfood)** | 14–22 | Auth, sessions, settings, export, deletion, offers and settle, notebook UI, all 6 species (rigs, palettes, grammars, **reduced-motion pose libraries**), newcomer flow (flagged), privacy review, threat model, a11y feature-complete, lived-experience panel 1. About 50 staff accounts (declared wipeable; the only wipeable data ever). | All §2 invariants covered by CI. Soak and device lab green at 7 birds. Charm rubric ≥ 4/5 for SR and reduced-motion cohorts. No open sev-1. |
| **M3 Closed beta** | 22–30 | **Data is permanent from here on**: beta aviaries carry into GA, never wiped, never reset (I-7). Allowlisted waves of 500 → 2,000 → 10,000 accounts. Visits enabled from week 26. External a11y audit. Pen test. DR drill (PITR restore to staging, ledger re-apply, integrity audit). Load test at 1M synthetic aviaries. Perception study round 2 plus forward-only recalibration. | SLOs met for 3 consecutive weeks. Zero integrity alarms. Beta feedback triaged for announcement and gamification leaks. Visits revocation E2E green. |
| **M4 GA** | 30–36 | 3× capacity headroom, runbooks, on-call rotation, open sign-up. Launch communication happens outside the product. There is no in-product launch announcement. | §16.3 checklist signed |

### 16.2 Ramping birds per aviary

- **Engine constants:** starters = 2, product cap = 7 (with a DB trigger). These never change via flags.
- **Operational cap `ops.max_birds`:** starts at **3** at GA. It is raised to **5** when 5-bird recognizability is ≥ 85%, the device lab passes at 5 birds, and a tick-cost review is done. It is raised to **7** when 7-bird recognizability is ≥ 80%, perf holds at 7 birds, and the 320 px layout review passes (D-27).
- **Calendar alignment:** the earliest beta aviaries (created around week 22) become eligible for a third bird around week 35, which is roughly GA. So three-bird support ships in GA. Fourth-bird eligibility arrives around week 48, fifth around 61, sixth around 74, and seventh around 90. **Scheduled gates: cap → 5 at week 44, cap → 7 at week 70**, each leaving about four months of buffer.
- **If a gate slips:** eligible aviaries are deferred silently (the newcomer simply doesn't visit yet). When the cap rises, the 45-day spacing still applies, so no aviary gets a burst of birds.
- **Continuous coverage before real aviaries get there:** staging "aged aviary" fixtures (backdated `created_at`, 3–7 birds) run in synthetic monitoring for render, audio, and tick paths. No production data is involved.

### 16.3 GA launch gates

1. Every §2 invariant has passing enforcement and tests. The surface audit is signed by PM, design, and the writer.
2. Performance budgets are met on the reference devices and the synthetic fleet (§14.1).
3. Accessibility: no critical or serious issues open, lived-experience charm rubric ≥ 4/5, and reduced-motion pose libraries complete for all species.
4. Audio: all listening gates pass at the GA cap (3 birds) and are verified at 7 birds in the lab.
5. Privacy: counsel sign-off (D-01, residency), a telemetry allowlist audit, a network-path test, and a clean PII-scanner week.
6. Security: pen-test findings at high severity or above are fixed.
7. Reliability: DR drill passed, 3 weeks of SLOs met, zero integrity alarms, and on-call staffed.

### 16.4 Instrumented from day one

Everything in §14.3 ships in M1 builds, so beta data is comparable. Dashboards:

- **Load:** first-bird and frame-time histograms by device class and region, inline-snapshot hit rate.
- **Audio boot state:** running, suspended, or unavailable.
- **Engine:** tick latency, lag, and tier mix; projection results.
- **Integrity:** the integrity panel (sev-1 signals).
- **Delivery and events:** email delivery; event accept/reject by reason.

Runbooks cover tick backlog, integrity alarm (freeze engine deploys, snapshot `bird_personality`, investigate), email provider failover, and a restore with ledger re-application.

### 16.5 Changing calibration after launch

- **Forward-only.** A new `engine_version` applies from the next tick. Vectors are never recomputed or back-filled.
- **Glide paths.** A parameter change that would noticeably alter current behavior (mood thresholds, call rates, perch utilities) interpolates from old to new over 14 days per aviary, so no bird "snaps" into a new personality.
- **Staged like infrastructure, not like an experiment.** Rollout goes 1% → 10% → 100% by aviary-hash cohort, watching operational metrics only. There is no outcome comparison between cohorts (§14.4).

### 16.6 Scaling triggers (pre-agreed)

| Trigger | Action |
|---|---|
| 1.5M aviaries, or aviary primary CPU > 60% sustained | Shard aviary-db by `aviary_id` hash. The engine and scheduler are already shard-agnostic. |
| Keepalive egress cost over budget, or multi-device attended share > 10% (aggregate gauge) | Add SSE for snapshot invalidation |
| RUM desktop frame p95 > 16.7 ms on > 10% of sessions | Ship the WebGL2 `DrawList` backend |
| Counsel requires EU residency | EU home shard with pinned accounts |
| Tick compute > 50% of worker capacity | Port `packages/engine` to Rust/WASM behind the same interface (determinism tests carry over) |

---

## 17. Team, ownership, and timeline

| Workstream | Staffing | Owns |
|---|---|---|
| **Engine** | 3 engineers + 1 simulation/calibration scientist | `packages/engine`, `apps/sim`, persona harness, perception-study analysis, I-4/I-5/I-7/I-13/I-14 |
| **Scene and motion** | 3 engineers (1 graphics-performance specialist) + 1 technical animator | Renderer, rigs, idle director, greeting, transitions, reduced-motion register, frame budgets, I-11 |
| **Audio** | 1 audio engineer + 1 sound designer | Synth, grammars, mix, captions data, listening gates, I-10 |
| **Platform and privacy** | 3 engineers + 1 SRE | Edge, API, identity, sync, data, telemetry boundary, security, I-6 (server half), I-8, I-9 |
| **Accessibility and voice** | 1 accessibility engineer + 1 staff writer + a naturalist consultant (part-time) + accessibility QA (contract) | Narration, keyboard model, copy registry and lints, prose grammars, lived-experience panels, I-12 |
| **Design / product / QA** | 1 visual designer (design-system spec: palette, contrast, focus treatment), 1 product designer, 1 PM, 1 QA automation engineer (device lab) | Surface inventory, I-2/I-3 audit, decisions log |

**Critical path:**

- Audio naturalness (M0–M1): if MOS lags, add a second sound designer before cutting species.
- First-bird performance (M0).
- Calibration studies (M1–M3).
- Reduced-motion pose libraries for six species (M2), started in M1 alongside the rigs.

**Cross-team rituals:**

- A weekly "aliveness review": a real-device session in which the whole team watches the aviary for 15 minutes.
- A decisions-log review at each milestone.
- The invariant owners sign each release.

---

## 18. Risks and mitigations

Likelihood (L) and impact (I) are rated H/M/L. "Signal" is the early warning we watch for.

### 18.1 Drift calibration

| ID | Risk | L / I | Signal | Mitigation |
|---|---|---|---|---|
| DC-1 | The perceived-change threshold is misestimated, so drift feels instant (Tamagotchi) or inert (screensaver). | M / H | Perception study round 2. Beta interview themes ("they changed overnight" / "nothing I do matters"). | Two perception rounds before GA. G is per-trait and tunable forward-only with 14-day glide paths. Calibration bands sit in CI. |
| DC-2 | Long-term convergence: after a year every bird is maximally bold, so individuality dies. | M / H | Lab 365-day persona runs: pairwise trait distance between birds | Per-bird ceilings (0.35–0.50 above seed) and headroom-proportional growth. The species' expression mappings keep species character. CI asserts a minimum pairwise distance after 365 days. |
| DC-3 | Presence under-counts legitimate watchers: a second-monitor watcher is visible but not focused, and phone users watch without touching. | H / M | Qualitative beta feedback ("my birds don't change") | This is the PRD's intended strictness (`concepts`, `interactions`), so we do not relax it unilaterally. The 5-minute activity window leans long, and touch `pointerdown` counts (D-07). Any change goes through the decisions log with the PRD owner. |
| DC-4 | Automation (mouse jigglers, kiosk tabs) inflates presence. | M / L | None needed (no incentive exists) | The daily saturation curve caps any day's contribution. Nothing is competitive, so there is no motive beyond the user's own aviary. |
| DC-5 | Daily buckets break at DST or timezone changes (double-counted hour, split day). | M / M | Property tests on DST boundaries in 40 zones | All buckets keyed by aviary-local date computed from UTC `t`. A timezone change closes the bucket at the change instant. |
| DC-6 | A parameter release "snaps" existing birds into new behavior. | L / H | Staged rollout watch; staging diff renders | Glide paths plus forward-only versions. A designer reviews a before/after fortnight render per release. |
| DC-7 | Attunement reads as a hidden happiness meter (a `non_goals` violation). | L / H | Surface audit; beta language ("they're mad at me") | Attunement affects only viewer-directed orientation, never mood, color, or calls between birds, and it has a floor. The absence-invariance test (§7.16). Long-absence greetings are *bigger* re-orientations, not colder ones. |

### 18.2 Sync correctness

| ID | Risk | L / I | Signal | Mitigation |
|---|---|---|---|---|
| SC-1 | **Personality loss:** a bad migration, operator error, or restore gone wrong. This is the worst failure the product can have. | L / **Catastrophic** | `trait_decrease_rejected_total`, `suspected_reset_total`, nightly audit | Trigger-enforced monotonicity and immutable ceilings. No app-role DELETE. A migration invariant harness. The two-person rule for production data access. PITR plus daily `bird_trait_daily` plus an encrypted daily logical dump of `bird_personality`. A sev-1 runbook: freeze deploys, restore affected rows to the last good daily snapshot + PITR. Traits only restore upward, never below a pre-incident value. |
| SC-2 | Lost interaction events (tab crash before flush, offline). | M / L | `events_rejected_total{stale}` trend | IndexedDB-mirrored queue plus keepalive flush. The loss is bounded to minutes of presence, which is invisible at the drift scale. |
| SC-3 | Double credit from retries or two devices. | M / M | Chaos harness | Idempotent `event_id`, and a presence union instead of a sum (§7.4). |
| SC-4 | Projection diverges from the committed tick. | L / M | `projection_mismatch_total` | A single `step()` implementation, `engine_version` pinning, cross-engine golden tests. |
| SC-5 | A stale replica or cache shows older state, so a bird "moves back". | M / L | Client version-rejection counter (aggregate) | Monotonic versions; the client ignores older snapshots. |
| SC-6 | The tick backlog makes the aviary "run slow". | M / M | Tick lag and p99 latency alerts | Warm/cold tiers, load shedding, read-time projection, autoscaling workers, sharding trigger. |

### 18.3 Audio uncanniness

| ID | Risk | L / I | Signal | Mitigation |
|---|---|---|---|---|
| AU-1 | Calls sound synthetic or toy-like, which breaks the affective spine. | **H** / H | MOS < 3.8 at M1 | A sound designer from M0. The syrinx-inspired two-source model with FM, noise, curved contours, onset overshoot, and room simulation. MOS gates. Contingency: extend M1 by 3 weeks and add a second sound designer. Launch with 5 species if one species misses (the pool is "about six"). |
| AU-2 | Repetition becomes detectable over weeks (grammar too small). | M / H | Dogfood listening diaries; ABX non-repetition | Grammar size targets, continuous per-call jitter, repeat-hash guard, a long-horizon listening diary during M2–M3. |
| AU-3 | Autoplay policy mutes first impressions on Safari and Firefox. | H / M | `audio_state_at_boot` (aggregate) | Accepted. Visuals carry the first seconds, and audio fades in on the first gesture. We never add an in-scene prompt; any additional cue goes through design review with I-2. |
| AU-4 | Chorus mud at 5–7 birds destroys recognizability. | M / H | Recognizability gate | Registers, stereo layout by perch, onset spacing, signature-motif prominence, voice-distance enforcement. The operational cap raises only on passing gates (§16.2). |
| AU-5 | Listening fatigue or annoyance in shared spaces. | M / M | Comfort gate; mute rate is *not* tracked (privacy), so rely on interviews | Comfortable pitch ceilings, evening and night levels, easy mute (`M`), captions. |
| AU-6 | Beak and sound desync on Bluetooth output. | M / L | QA on AirPods and other Bluetooth headsets | `outputLatency` compensation. |

### 18.4 Accessibility regressions

| ID | Risk | L / I | Signal | Mitigation |
|---|---|---|---|---|
| AR-1 | New animations ship without reduced-motion counterparts. | H / H | Type error (by design) | The motion registry requires a counterpart at compile time (§9.7). Visual regression runs in reduced motion. |
| AR-2 | Narration grows chatty as features add events. | M / H | Live-region rate test | Queue caps, a global minimum gap, a settings pace control. New event types need accessibility review. |
| AR-3 | Narration decays into state-list style ("pip: front perch"). | M / H | Content lint on 10⁴ samples | Grammar-only generation, writer review, banned patterns (digits, mood labels, colons). |
| AR-4 | Screen-reader quirks: VoiceOver skips identical repeated text; NVDA truncates long atomic updates. | M / M | Per-SR manual scripts | Recency memory prevents identical strings. Utterances are kept ≤ 30 words. |
| AR-5 | New palettes break contrast (top bar, captions). | M / M | Contrast matrix test | Dynamic scrims plus the matrix test on every palette change. |
| AR-6 | Keyboard focus lost during reconciliation (a bird moves or a newcomer is added). | M / M | Keyboard E2E | Proxies are stable DOM nodes keyed by `bird_id` and are never re-created on snapshot. |

### 18.5 Principle erosion, privacy, operations

| ID | Risk | L / I | Mitigation |
|---|---|---|---|
| PE-1 | Announcement or gamification creep ("just one toast", "a quiet visit counter") | H / H | No components exist to build them with. Lints. Milestone surface audits. Decisions-log sign-off for any new notification or email type. §3.3 pre-committed answers. |
| PE-2 | Pressure for engagement metrics | M / H | The data does not exist at aggregate level (§14.4). The architectural absence makes reappearance expensive, as `social_optional` intends. |
| PV-1 | PII leaks into logs or traces via libraries | M / H | Allowlisted logger, PII scanner, CI injection tests, 14-day retention |
| PV-2 | D-01 (vectors in export) judged inconsistent with I-1 | M / L | A config flag reverses it without code. Counsel and PM decide by M2. |
| OP-1 | Magic-link deliverability (spam folders, delays) blocks sign-in | M / H | Dedicated domain, warm-up, DMARC reject, second provider, delivery SLO. A system-register help line on the check-email screen. |
| OP-2 | Edge snapshot budget missed in far regions | M / M | Regional replicas and caches, last-known cache, quiet-field fallback, `snapshot_inline_hit` alerting |
| OP-3 | Browser behavior changes (autoplay, focus, throttling) | M / M | Beta and nightly browser channels in CI (Chrome Beta, Safari Technology Preview, Firefox Nightly) |
| OP-4 | Users never notice or adopt newcomers | M / L | Acceptable by design: no pressure, no reminders. The notebook sighting entry gives a gentle, specific hint. Qualitative beta interviews only. |
| OP-5 | Invites abused for spam | L / M | Fixed template, rate limits, complaint-driven host suspension |

---

## 19. Open questions (non-blocking; defaults already chosen)

1. D-01: should exports include raw vectors? Default yes, pending counsel and PM confirmation.
2. D-08: is a 90-day lapse right for unused visit passes? Default yes.
3. D-13: should background-tab audio become an opt-in setting after launch? Default no in v1.
4. Seasonal scene variation (leaf color by hemisphere and season) is not in the PRD and is deferred. The timezone-coordinates table already supports it.
5. Additional languages for naturalist prose: deferred. The grammar engine is language-parameterized.
6. Per-bird pronoun preference (D-18): deferred.

---

## Appendix A: Snapshot example (host audience, abbreviated)

Compiled presentation only. There are no trait fields anywhere (I-1). Continuous values are mixed with mood and time and quantized to 1/32.

```json
{
  "schema": 1,
  "aviary_version": 48213,
  "projected_steps": 0,
  "server_time": "2026-09-23T14:02:11.412Z",
  "next_tick_at": "2026-09-23T14:03:00.000Z",
  "audience": "host",
  "aviary": {
    "timezone": "Europe/Lisbon",
    "weather": { "kind": "rain", "started_at": "2026-09-23T13:55:40Z", "ends_at": "2026-09-23T14:14:10Z", "intensity": 0.5 },
    "ambient": { "window_index": 29314441, "window_seed": "9f2c41d0", "chorus_until": null },
    "last_presence_end_at": "2026-09-23T07:48:03Z",
    "active_offers": [ { "offer_id": "0199…", "kind": "seed", "slot": "front-1", "expires_at": "2026-09-23T14:09:30Z" } ],
    "newcomer": null
  },
  "birds": [
    {
      "id": "8d1e5c2a-…",
      "name": "pip",
      "species": "grey-warbler", "species_version": 3,
      "appearance": { "seed": 184467, "detail_level": 3,
        "plumage": { "crown": "oklch(0.52 0.045 250)", "back": "oklch(0.58 0.030 240)", "breast": "oklch(0.83 0.020 90)", "wingbar": "oklch(0.90 0.010 95)" } },
      "voice": { "seed": 99812 },
      "perch": { "zone": "middle", "slot": 2, "since": "2026-09-23T13:56:02Z" },
      "transition": null,
      "affect": { "arousal": 0.594, "ease": 0.719, "novelty": 0.125 },
      "mood": "content",
      "activity": "sheltering",
      "calling": { "rate_per_min": 0.344, "respond_p": 0.563, "chorus_join_p": 0.469, "loudness": 0.719,
                   "productions": { "melodic": 0.5, "short": 0.188, "rising": 0.313 } },
      "greet": { "weight": 0.656, "forms": { "glance": 0.375, "soft_call": 0.344, "step": 0.281 }, "approach_amp": 0.406 },
      "offer_cooldown_until": null
    }
  ],
  "offer_plans": {
    "ref": "48213", "expires_at": "2026-09-23T14:05:11Z",
    "seed": [ { "bird": "8d1e…", "response": "approach", "delay_ms": 2300, "accept": true },
              { "bird": "31aa…", "response": "wait_then_approach", "delay_ms": 41000, "accept": true } ],
    "song": { "f3": [ { "bird": "8d1e…", "response": "join_in", "delay_ms": 1800 } ] },
    "pool": [ { "bird": "31aa…", "response": "watch", "delay_ms": 900 } ]
  }
}
```

**The visitor audience** is the same, minus `last_presence_end_at`, `greet`, `offer_plans`, and `offer_cooldown_until`, and plus `"audience": "visitor"`.

## Appendix B: Client event batch example

```json
{ "events": [
  { "event_id": "0199a2c4-7e10-7c21-9d0e-2f6a1b9c0e11", "type": "presence_heartbeat",
    "payload": { "interval_ms": 30012, "present_ms": 30012, "activity_window_ms": 300000, "muted": false } },
  { "event_id": "0199a2c4-8a2b-7f03-b1c4-7d2e9a0f4c52", "type": "listen_in", "bird_id": "8d1e5c2a-…",
    "payload": { "duration_ms": 94210, "ended_by": "escape" } },
  { "event_id": "0199a2c4-9b77-7a55-8e21-0c4d6f2b8a93", "type": "offer",
    "payload": { "offer_id": "0199…", "kind": "seed", "plan_ref": "48213:seed:-" } },
  { "event_id": "0199a2c4-a0f1-7d8e-9c3b-5e1f7a2d6b04", "type": "greeting_observed",
    "payload": { "first_bird_id": "8d1e5c2a-…", "order": ["8d1e5c2a-…", "31aa…"], "absence_bucket": "6h_3d", "approached": true } },
  { "event_id": "0199a2c4-b3c9-7e12-a7d4-1b8c3e5f9a75", "type": "settle", "payload": {} }
] }
```

## Appendix C: Persona definitions for the calibration harness

Every persona runs 180 simulated days (DC-2 variants run 365) × 200 seeds × all 6 species. Assertions are the §7.5 bands plus the invariants in §7.16.

| Persona | Presence | Listen-in | Offers | Other |
|---|---|---|---|---|
| Regular | 15 ± 5 min/day, 6 days/week, one session at a random time 07:00–22:00 local | 1 × 1–3 min on a random bird | 1 per session | 10% of sessions muted |
| Light | 5 ± 2 min, 3 days/week | 0–1 | 0–1 | — |
| Heavy | 60 ± 15 min daily, two sessions | 3 × 2–5 min | 3 | — |
| Tab-left-open | visible 12 h/day, focus false or no activity | 0 | 0 | Assert zero credit |
| Jiggler | attended with activity, 12 h/day | 0 | 0 | Assert ≤ heavy |
| Two-device | the regular pattern duplicated on two devices, overlapping 80% | same | same | Assert = regular ±2% |
| Returner | regular 21 d → absent 14 d → regular 21 d | regular | regular | Assert non-decreasing; attunement recovers to > 0.7 within 2 sessions |
| Traveler | regular, with the timezone shifting +7 h every 10 days | regular | regular | Assert no double buckets; lighting ramps |
| Night owl | regular pattern at 23:00–01:00 local | regular | regular | Greeting eligibility at night; nocturnal species behavior |
| Long-haul (365 d) | regular | regular | regular | Pairwise trait distance between birds ≥ 0.15 (DC-2) |

## Appendix D: Prose-grammar samples (naturalist register)

**Notebook** (one entry per qualifying day, after the sparsity budget):

- `FirstGreeterShift`: "tuesday — pip greeted before wren today, first time this week."
- `WeatherPassage`: "a short rain came through after lunch. both birds tucked in under the high branch and went quiet until it passed."
- `LongPreen`: "a long stretch of quiet this morning. pip preened for several minutes without looking up."
- `OfferFirst`: "wren came down to the seed for the first time — slowly, after a long look from the back perch."
- `SeasonalLookBack`: "pip spends more of the afternoons on the front rail than in early summer."
- `NightCaller`: "the nightjar called twice after dark, low and far off."

**Narration:**

- Return: "it is late morning in the aviary. pip looks up from the middle branch and calls once, softly."
- Idle: "wren sits further back with feathers fluffed, watching the far leaves."
- Offer: "a few seeds fall onto the front rail. pip drops down to them almost at once; wren waits, then follows."
- Settle: "the light turns to evening. the calls grow few and quiet."

**Captions:** "a soft three-note rise" · "a low trill, paused, low trill again" · "a single sharp call from the back perch" · "several birds calling together, the high whistle loudest".

**Never generated** (lint-rejected): "Welcome back!" · "you visited every day this week" · "pip is happier!" · "pip's boldness increased" · "Achievement unlocked" · "wren mood: wary" · "session started at 7:43".
