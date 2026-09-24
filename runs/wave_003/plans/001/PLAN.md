# Pocket Aviary — v1 Implementation Plan

*CARE wave_003 · plan 001 · phase-1 planning deliverable.*

This plan turns the Pocket Aviary PRD into something a separate engineering team can build. It does not restate the spec; it makes the calls the spec leaves open, pins numbers where the spec says "calibrate during build," and says how each product principle is enforced in code, schema, CI, or process. Where the PRD is ambiguous or pulls in two directions, the call made is logged in §3 and referenced elsewhere as **D-n**.

---

## 0. Reading guide and conventions

- **§1 (invariants)** is the contract. Every later section traces back to it. A change that violates an invariant is out of scope however small it looks.
- Values marked **(cal)** are starting constants. A named harness or study (§16) tunes them before launch. They're given so engineering can build and test against real numbers from day one.
- **Product surface** means the naturalist voice (aviary, narration, captions, notebook, offer/gesture menu, naming). **System surface** means the matter-of-fact voice (sign-in, account settings, sessions, visits management, sync/errors, accessibility settings, unsupported browser, visit-unavailable).
- Code identifiers follow `concepts.md` vocabulary: `listenIn` (never `solo`/`select`/`pin`), `call` (never `song`/`chirp`, except the user-offered `songFragment` item), `bird` (never `pet`/`creature`), `settle`, `offer`, `presence`. Naming drift in code leaks into copy, so the lexicon lint (§11.4) covers identifiers as well as strings.

---

## 1. Load-bearing invariants

| ID | Invariant | Primary enforcement | Verification |
|---|---|---|---|
| **INV-1** | Only the server simulation tick writes personality vectors, drift-filter state, attunement, and mood. No client and no API handler can. | Postgres role `sim_tick` is the only role with `UPDATE` on `sim.bird_personality` and `sim.bird_mood`. The API role has `SELECT` only. | CI grants test. An integration test tries the write through the API role and expects `permission denied`. |
| **INV-2** | Personality drift is additive and non-negative, i.e. monotonic toward expressive. Neglect never lowers a trait. | Deltas are computed as `max(0, …)`. SQL applies `SET v = LEAST(ceiling, v + Δ)`. A row trigger rejects any decrease. | Property tests on the drift function. A trigger test. The prod counter `sim_monotonic_block_total` must stay at 0 (it alarms otherwise). |
| **INV-3** | The personality vector is never shown to the user in any product or system surface. It leaves the server only inside the export file the user asks for (D-2). | Snapshot and visit serializers are allowlist-based, and trait fields don't exist in the `contracts` schema. DOM and ARIA never carry trait values. | A contract test fails if any trait key appears. A DOM audit test runs over e2e sessions. |
| **INV-4** | A bird's identity (UUID) lasts as long as the account. No code path replaces, regenerates, re-seeds, or "resets" a bird. | Only the hard-delete job role has `DELETE` on `sim.birds`. FKs use `ON DELETE RESTRICT`. Species assets are pinned per bird by `species_rev`. | A migration checklist. Row-count and checksum assertions in every migration that touches the `sim` schema. |
| **INV-5** | Presence is `document.visibilityState === 'visible'` AND `document.hasFocus()` AND pointer/key activity within the last W minutes, all at once. Visitors never generate presence or interaction events. | Client `PresenceTracker`. The server rejects events from visitor passes (no `events:write` scope) and caps credited presence at the wall-clock union across devices. | A unit test per signal. The e2e test "tab open all night" produces zero drift. |
| **INV-6** | Interaction events are applied exactly once, in per-account order. | Client idempotency keys, a gap-free per-account sequence, and a tick cursor, with the cursor advanced in the same transaction as the state write, plus lease fencing. | Chaos tests: kill a worker mid-tick, deliver duplicates, run zombie leases. |
| **INV-7** | An aviary starts with exactly two birds and caps at seven. It gains birds only by aviary age. | Engine constant `MAX_BIRDS = 7` plus a DB insert trigger. The unlock scheduler reads only `aviary.created_at`. | Tests. A lint rule forbids the unlock module from importing event, presence, or interaction modules. |
| **INV-8** | No recorded audio is shipped or played, under any fallback. | The build asset scanner fails on audio MIME types. Lint bans `<audio>`, `HTMLAudioElement`, and `decodeAudioData` outside test code. | CI. |
| **INV-9** | Nothing announces: no welcome toasts or banners, badges, counters, confetti, spinners, push, or emails about the aviary. | The design system has no `Toast`, `Badge`, or `Spinner` components. Lint bans `Notification`, `PushManager`, and `showNotification`. Outbound email templates are a closed allowlist (§13.9). | CI lint and a copy review gate. |
| **INV-10** | Email is stored once, encrypted, on the account record. Every other reference uses the synthetic account UUID. | Envelope encryption. Lookup uses a keyed-HMAC column on the account row only. A log scrubber. | A PII scanner over all logs from integration runs, plus a schema lint (no `email*` columns outside an allowlist of two tables). |
| **INV-11** | Telemetry is aggregate-only, with no account, bird, session, or email dimension. Telemetry systems have no route to the simulation database. | A metric and label allowlist in the OpenTelemetry collector. Network policy and separate credentials. | A CI label lint and a quarterly boundary audit. |
| **INV-12** | The voice split holds: product surfaces naturalist, system surfaces matter-of-fact. The notebook and narration describe the aviary, never the user's behavior. | The `@aviary/voice` grammar and its lints, plus the system-string catalogue lint. | A generated-corpus lint in CI (100k samples per template). |
| **INV-13** | The first frame is the aviary mid-motion, or the quiet field while state loads. No spinner, entry animation, "ready" pop, or fade-from-static. The only exception is the one-time arrival of starter and newcomer birds. | The boot path has no loader component. The render core starts every bird at its current action phase. | Visual e2e: frame 1 shows birds with a non-zero action phase, and no element matches loader patterns. |
| **INV-14** | Accessibility surfaces (narration, captions, reduced-motion rendering, keyboard) ship in v1 at full product quality, not as fallbacks. | A launch gate. | Screen-reader, keyboard, deaf/HoH, and vestibular user testing, plus automated checks (§16.6). |

---

## 2. Scope

### 2.1 In v1

- **Scene:** one horizontal scene with no panning, scrolling, or zooming. Three perch zones. A day/night cycle on the aviary's local time. Rare ambient weather. Ambient leaf and feather ornaments. A four-icon top bar that fades. Responsive layout from a 320 px phone to an ultrawide desktop.
- **Birds:** a six-species pool. The system picks two starter birds; the user names them and can rename them any time. New birds arrive by aviary age, up to seven. Each bird has a stable identity, a hidden personality vector, mood, a procedural voice, and a greeting signature.
- **Simulation (server):** a roughly one-minute tick; drift; mood; the behavior plan; bird-to-bird interaction; weather; the return-greeting resolver; the offer resolver; the newcomer lifecycle; the notebook observer.
- **Interactions:** return-greeting, listen-in, offer (seed, song fragment, still pool), settle (with a 5-second undo), the field notebook, and presence accounting.
- **Audio:** procedural synthesis in an AudioWorklet, chorus mixing, the listen-in mix, and a fallback of graceful silence with captions on.
- **Accounts:** magic-link sign-in. Per-device sessions with revocation. Email change with verification. A JSON export delivered as an emailed link. Deletion that is soft for 30 days, then hard.
- **Sync:** multi-device through canonical server state, and tolerance for going offline.
- **Visits:** email invitations per invite, a read-only ambient view, revocation, expiry of unused invites after 30 days, a visit log, and visit notifications that are opt-in and off by default.
- **Accessibility:** naturalist narration, call captions, a designed reduced-motion mode, full keyboard support, visible focus, WCAG AA contrast, and accessibility settings.
- **Performance and observability:** the budgets in §14, aggregate-only RUM and synthetic checks, and tick SLOs.
- **Browsers:** the last two major versions of Chrome, Safari (macOS and iOS), Firefox, and Edge. Anything older gets a matter-of-fact unsupported-browser page.

### 2.2 Refused, and what in the architecture makes each refusal hard to undo

| Refusal | Structural consequence in v1 |
|---|---|
| Gamification: achievements, streaks, levels, scores, badges, "birds adopted: N", visit calendars | No column anywhere counts visits, days, or streaks (schema lint). `TopBarIcon` has no badge prop. The lexicon lint (§11.4) covers copy and identifiers. The newcomer unlock depends only on aviary age (INV-7). No product surface shows a date-count such as "days together". |
| Tamagotchi mechanics: death, hunger, distress, decaying meters | Monotonic drift (INV-2). Absence feeds no negative-valence input into mood (§7.6). The pose library has no distress, sick, or starving poses. Offers carry no sustenance state. Birds cannot be removed (D-26). |
| Notifications about the aviary | No Push API and no Notification API. The email allowlist (§13.9) contains no re-engagement template type, so adding one takes a reviewed schema change. |
| Social network, public discovery, leaderboards, show-off mode | No endpoint lists or looks up aviaries. Aviary IDs never appear in URLs. Visit tokens are opaque. No pipeline reads the `sim` schema across accounts, so leaderboard metrics can't be computed. Visitors get the same serializer as the host, minus the per-arrival greeting (§6.6). |
| Native apps | No native-driven protocol compromises: cookie-based web auth, no deep-link schemes, no push tokens. |
| Numeric personality exposure | INV-3. No support or admin tool renders trait values either (§13.8). There is no import path. |
| Payments, shared or multi-aviary accounts, customizable scenes | `sim.aviaries.account_id` is `UNIQUE`. No fields exist for customizing the scene or placing birds. |

### 2.3 Deferred (not refused; scoped out of v1 on purpose)

Other languages (the naturalist grammar is English-only in v1, D-17). SSO and passwords. Listening in the background with the tab hidden (D-12). Seasonal and latitude-aware light curves. A WebGL renderer. Server push over SSE/WebSocket (D-23). Data residency beyond the primary region. Letting the user release a bird (D-26).

---

## 3. Interpretations and decisions

| ID | PRD tension or gap | Decision | Rationale |
|---|---|---|---|
| **D-1** | The top bar is "account/settings, accessibility, notebook, offer. Nothing else", but settle is "triggered from the top bar." | The offer icon opens a **gestures menu**: three offer items, a separator, then **settle the aviary**. The top bar keeps exactly four icons. | Honors both clauses. Settle and offer are both gestures toward the aviary; settings and accessibility are system surfaces. |
| **D-2** | The export includes "current personality vectors", yet the user "never sees personality vector values." | The export includes the vectors under `birds[].personality`, as the PRD says. It's an out-of-band file the user explicitly asks for, needed for data portability and legal access requests. The product never previews, renders, or parses it, and there is no import. | The exposure ban targets product surfaces that turn birds into stats. A raw portability file isn't a surface. An import path would give clients a way to write personality (violating INV-1), so none exists. |
| **D-3** | Mood "resets on a daily-ish cadence", yet mood "does not reset when the user opens the tab." | The "daily reset" is a circadian relaxation on the server: interaction and contagion biases decay within hours, so each morning's mood is set mostly by time of day and personality. Opening a tab never touches mood. | Both clauses hold. No snapping. |
| **D-4** | A neglected bird "becomes ambient … greeting less often", yet "traits do not move down on neglect." | A separate, non-personality **attunement** state (§7.5) rises with presence and decays toward a floor with absence. It only lowers how readily a bird greets and approaches on arrival. It never raises wariness, lowers call rate, dims plumage, or touches the vector. | Models "less often is what's been observed" without punishing absence. A floor means birds never stop greeting altogether. |
| **D-5** | The brief says drift responds to "whether you mute the calls or let them play", but the engine's input list leaves mute out. | Presence feeds vocal-frequency drift only while calls are **perceivable**: audio on, or captions on. Muting with captions off simply withholds that one input. It never produces a negative. | Honors the brief. The captions clause keeps deaf and hard-of-hearing users on equal footing. |
| **D-6** | "Focus it … by keyboard-focusing it" vs. "Enter triggers listen-in on the focused bird." | Keyboard focus alone does **not** start listen-in; Enter or Space does. Moving focus to a different bird ends the current listen-in, and the new bird isn't engaged until Enter. | Arrowing through birds shouldn't churn the mix. Follows the more specific keyboard spec. |
| **D-7** | Which "pointer or key" signals count toward presence. | `pointermove` (covers mouse, pen, and touch-drag) and `keydown` (the modern stand-in for the deprecated `keypress`). A pure tap with no pointermove doesn't count. **W = 4 min (cal, range 3–5).** | The PRD's failure mode is over-counting. The strict reading can only under-count on phones, which is the safe direction. Flagged for product review in §20. |
| **D-8** | Who receives an offer, given that "offers are not reached by clicking on a bird." | Offers go to the aviary. The server offer resolver (§7.11) decides which birds react, from mood, curiosity, boldness, distance, and cooldown. The cooldown is invisible: no timers, and the menu is never disabled because of it. | Keeps the offer a gesture rather than a button aimed at a bird. Nothing gamey in the UI. |
| **D-9** | "A new species offer appears in the user's flow", with no toast or badge allowed. | The newcomer shows up **in the scene** as a wild bird on the back edge perch for up to 21 days. While it's around, the gestures menu gains *make room for the thrush*. A notebook entry may mention it. Welcoming it opens the naming card; ignoring it has no consequence. | The aviary itself does the noticing. There's no announcement surface, and declining costs nothing. |
| **D-10** | How long a visit link stays valid after it's opened. | The link is single-use. Redeeming it creates a **visitor pass** bound to that browser for **14 days**, revocable at any time and not renewable. Visitors get the ambient view plus accessibility settings only, with no notebook. | "One-time link" plus "no permanent visitor list". The notebook is the host's record; §20 notes it for revisit. |
| **D-11** | "Local time" when a user's devices report different timezones. | One canonical **aviary timezone**. It's updated when a host session reports a different IANA zone and builds at least 5 minutes of presence there. Both mood circadian rhythm and lighting use it; on a zone change the lighting transitions over 20 minutes. | Mood and light stay coherent across devices and for visitors, and a one-off phone check abroad doesn't flip the aviary. |
| **D-12** | Audio while the tab is hidden. The PRD only says rendering stops. | Audio fades out over 2 seconds and the `AudioContext` suspends. On return it resumes and fades in. | Background timer throttling would break call scheduling, it saves battery, and it matches "nothing to see". Background listening is deferred. |
| **D-13** | "Calls already audible" at first frame vs. browser autoplay policies. | The `AudioContext` is created at boot. If the browser requires a user gesture, audio starts at the first `pointerdown`/`keydown`/`touchend`, fading in over 1.5 seconds from wherever the plan is. There is no "tap for sound" prompt or icon. Where the browser allows it (engagement-based autoplay), calls are audible from the first frame. | A prompt would be announcement chrome. This is a platform constraint, stated honestly. |
| **D-14** | Is the "settled" state canonical or per device? | The settled **lighting** is local to that device's session. The **mood effect** is canonical: the settle event reaches the tick. The settled state ends on active re-engagement (a click, tap, or key in the aviary, a listen-in, or an offer) or on returning after an absence of at least 30 minutes. | Settle is a goodbye from one person at one screen. A later phone session is a new arrival and gets a greeting. |
| **D-15** | Greetings must fire within 1–2 seconds, and the boot path must not wait on a write to the primary. | The regional API resolves the greeting from replica state and returns it in the boot payload. The client posts an `arrival` event, and the server writes the greeting's actions into the canonical plan asynchronously. | Meets both the 500 ms and 1–2 s targets. Other devices see the same bird movements within about a second. |
| **D-16** | How notebook, narration, and caption prose get generated. | An authored, typed phrase grammar (`@aviary/voice`), written by a naturalist writer. No LLM at runtime. | Per-bird state can't go to a third party (privacy). The voice needs to be controlled and lint-checked. Outputs should be deterministic and testable. |
| **D-17** | Localization. | English only in v1. | The naturalist grammar is language-specific craft. |
| **D-18** | Where weather comes from. | Simulated per aviary on the server, seeded. Real-world weather is never used. | Real weather would need location (a privacy cost) and wouldn't hit the "rare, never assertive" calibration. |
| **D-19** | Name casing in the lowercase voice. | Bird names are lowercased on product surfaces ("pip greeted before wren") and shown as typed on system surfaces ("Pip"). The grammar avoids gendered pronouns for birds, using the name or the species noun. | Voice consistency. The user never assigns a sex to a bird. |
| **D-20** | Keyboard equivalent of "click anywhere to undo settle." | Escape, Enter, or Space within 5 seconds undoes it. The settle menu item's accessible description says so. | Keyboard parity (INV-14). |
| **D-21** | Expiry for the email-change verification link. | 24 hours, single-use. The old email keeps working until the new one is verified, and the old address gets a matter-of-fact notice. | The new address may be read on another device. There's no lockout risk because the old email still works. |
| **D-22** | Rendering technology. | A custom Canvas 2D renderer drawing from pre-rasterized part atlases, not WebGL. | About 200 draw calls per frame at seven birds is easy at 60 fps on old laptops. Small bundle, no context-loss handling, one code path. |
| **D-23** | Transport. | HTTP pull (snapshot plus keepalive) and HTTP event posts. No WebSocket or SSE in v1. | The PRD's pull model is enough because plans (§7.7) cover the gaps between pulls. Far less operational surface. |
| **D-24** | Retention of the event log. | Raw interaction events are kept 14 days after consumption, then dropped by partition. | The vector is never rebuilt from logs, so short retention minimizes the private history kept while still covering outage catch-up and debugging. |
| **D-25** | Rapid tab-flicking and "never identical twice." | Absences under 20 seconds produce at most a glance, with probability 0.5. Two arrivals within 2 minutes get at most one non-glance greeting. At most one "re-orientation" greeting per 12 hours. | Stops greetings becoming a tic that reads as canned. |
| **D-26** | Removing a bird. | Not offered in v1. | Bird identity is permanent (INV-4), and removal is custodial-flavored. Revisit after launch (§20). |
| **D-27** | Starter species. | Starter pairs come from a curated contrast table that excludes the nightjar-like species, which can only arrive later as a newcomer. | A starter that sleeps all day would spoil the first encounter for daytime users. |

---

## 4. Architecture

### 4.1 System shape

```
┌──────────────────────────── Browser (host or visitor mode) ────────────────────────────┐
│  inline boot (≤6 KB): quiet-field sky from device clock, feature check                 │
│  render core: scene layout · choreographer (plan evaluator) · rig/micro-motion · canvas│
│  audio engine: AudioContext → AudioWorklet synth → per-bird buses → mixer              │
│  voice runtime: narration composer (aria-live) · caption composer                      │
│  PresenceTracker · event outbox (IndexedDB) · clock-offset estimator  [host mode only] │
│  chrome (Preact, lazy): top bar · gestures menu · notebook · settings · visits         │
│  service worker: immutable asset cache, offline quiet field                            │
└───────────────┬────────────────────────────────────────────────────────────────────────┘
                │ HTTPS: HTML with inline snapshot · JSON API · aggregate metrics beacon
┌───────────────▼────────────────┐        ┌──────────────────────────────────────────────┐
│ Edge (CDN + edge worker)       │        │ Telemetry collector (separate host + project)│
│ static assets (immutable)      │        │ allowlisted aggregate schema only → metrics  │
│ streams HTML, inlines snapshot │        │ store. No credentials to any app database.   │
└───────────────┬────────────────┘        └──────────────────────────────────────────────┘
                │
┌───────────────▼────────────────┐   ┌───────────────────────────┐
│ Regional API (3 regions, read) │──▶│ Postgres read replica      │
│ boot/snapshot, notebook read,  │   └───────────────────────────┘
│ greeting resolution            │
└───────────────┬────────────────┘
                │ writes (events, offers, auth, settings, visits) routed to primary region
┌───────────────▼────────────────┐   ┌──────────────────────────────────────────────────┐
│ Primary API (modular monolith) │──▶│ Postgres primary                                  │
│ auth · events · offers ·       │   │   schema `acct`  (accounts, sessions, links)      │
│ settings · visits · export ·   │   │   schema `sim`   (aviaries, birds, personality,   │
│ deletion                       │   │                   mood, events, plans, notebook)  │
└───────┬────────────────────────┘   │   schema `social` (invites, passes, visits)       │
        │ jobs (pg-boss)             └───────▲──────────────────────────▲───────────────┘
┌───────▼──────────────┐   ┌─────────────────┴───────────┐   ┌──────────┴──────────────┐
│ Jobs: mail · export ·│   │ Tick workers (partition-     │   │ Redis: rate limits,     │
│ deletion · retention │   │ leased) + notebook observer  │   │ session cache (30 s)    │
└──────────────────────┘   └──────────────────────────────┘   └─────────────────────────┘
      │                              KMS (email envelope keys) · S3 (exports, 7-day lifecycle)
      └──▶ transactional email provider (no open/click tracking)
```

**Service shape.** Four deployables share one TypeScript monorepo:

- **edge:** CDN plus edge worker.
- **api:** a modular monolith with `auth`, `aviary`, `events`, `offers`, `notebook`, `settings`, `visits`, and `lifecycle` modules, deployed as a primary, and read-only regional replicas.
- **tick:** simulation workers and the notebook observer.
- **jobs:** mail, export, deletion, retention.

A modular monolith is right for a team of about 12. The tick is separate because it has its own scaling profile and is the sole writer (INV-1).

### 4.2 Client/server split

| Concern | Server (canonical) | Client (presentation) |
|---|---|---|
| Personality vector, drift filter, attunement | Owns, computes, persists | Never receives |
| Mood | Owns and transitions | Receives the enum and shapes idle motion from it. Never labels it. |
| Perch positions and macro actions (moves, calls, preen bouts, offer reactions, greetings) | Plans them in a rolling committed action queue | Evaluates the plan at the current server time and animates between states |
| Idle micro-motion (breathing, head saccades, blinks, shuffles, feather fluff) | Sends only a quantized `demeanor` hint | Generates it procedurally, seeded by bird ID and time |
| Call timing and context | Schedules call descriptors (time, bird, context, seed, intensity) | Realizes the grammar, synthesizes audio, and writes captions from what was actually played |
| Weather and light phase | Owns the weather schedule. Light is a function of the canonical timezone. | Renders both |
| Leaves, feathers, parallax breathing | None | Pure client ornaments (per the PRD) |
| Narration prose | None | Composed client-side from the same state the renderer reads |
| Notebook prose | The observer writes and stores rendered text | Displays it |
| Presence | Credits and merges intervals across devices | Detects the three-signal conjunction and reports spans |
| Offer outcome | Resolves it synchronously | Plays the item placement immediately and the bird reactions when they arrive |

### 4.3 The render pipeline boundary

The contract between simulation and rendering is the **snapshot plus action plan**. It is semantic, not visual:

- **The server says what a bird is doing and where:** "wren moves to front slot 0 at t, style hop", "pip calls, context response, seed s, intensity 0.4, at t", "pip is preening since t₀".
- **The client decides how it looks and sounds:** poses, arcs, beak sync, micro-motion, synthesis, captions, narration.

The boundary rules:

1. Plan times are server epoch milliseconds. The client maps them to its clock through an estimated offset (§8.6) and to `AudioContext.currentTime` for audio.
2. The plan never contains pixels, coordinates, or trait values. Perches are `{zone, slot}`, and the client's layout engine turns them into positions for the current viewport.
3. The client may *add* motion (micro-motion, ornaments, look-at-caller head turns). It may never *invent* macro actions, with one exception: the bounded offline continuation (§8.7), which is idle-only and flagged provisional.
4. Both host and visitor clients consume the identical plan.

### 4.4 Technology choices

| Layer | Choice | Why |
|---|---|---|
| Language | TypeScript everywhere (Node 22 LTS for services) | Shared contracts, PRNG, plan evaluator, and voice grammar across server and client |
| Client build | Vite with manual chunking and `size-limit` in CI | Aggressive code-splitting under the budget |
| Scene rendering | Custom Canvas 2D engine (D-22) | Small, predictable, runs 60 fps on old hardware |
| Chrome UI | Preact with signals, lazy-loaded after first frame | About 5 KB of runtime, and fine for accessible widgets |
| Audio | WebAudio with one custom AudioWorklet synth | Precise scheduling and no per-call node allocation (the memory rule) |
| Database | PostgreSQL 16 (managed; Aurora PostgreSQL or equivalent) with a primary and cross-region read replicas | Transactions give exactly-once tick application, and column grants enforce INV-1 |
| Jobs | pg-boss, a Postgres-backed queue | Low volume, and one less system |
| Cache and rate limits | Managed Redis | Ephemeral keys only |
| Edge | Cloudflare CDN plus Workers | Streams HTML and inlines the snapshot close to the user |
| Objects | S3 with SSE-KMS | Exports, deleted after 7 days by lifecycle rule |
| Email | Transactional provider under a DPA, with **open and click tracking disabled** | Tracking pixels are behavioral telemetry, and click-rewriting breaks single-use links |
| Observability | OpenTelemetry → Prometheus-compatible store and Grafana. Logs kept 14 days. Self-hosted error tracking with scrubbing. | Aggregate-only (INV-11), and no third-party SDKs in the page |

### 4.5 Repository layout

```
apps/web        client (boot, render, audio, chrome, service worker)
apps/edge       edge worker
apps/api        HTTP API modules
apps/tick       tick scheduler/workers, notebook observer
apps/jobs       mail, export, deletion, retention
packages/contracts     zod schemas → JSON Schema for snapshot, events, API (single source of truth)
packages/sim-core      shared: PRNG (PCG32, integer ops), time, plan types, plan evaluator
packages/sim-server    server-only: drift, attunement, mood, planner, resolvers, weather, unlocks
packages/voice         phrase grammar engine + corpora (notebook, narration, captions) + lints
packages/species       versioned species revs: rigs, pose library, palettes, motif libraries
packages/design-tokens palette (OKLCH), type, focus ring, contrast-verified pairs
tools/calibration      drift harness & reports        tools/audio-lab   listening-panel app, distinctness metrics
tools/rig-editor       species rig & pose authoring   tools/lints       voice/lexicon/telemetry/PII/asset lints
```

Drift and mood code lives in `sim-server` and is never bundled into the client: a CI check fails if `apps/web` imports it.

### 4.6 Deployment topology and capacity

- **Regions.** The primary (all writes and the tick) is in us-east. Read replicas plus regional API sit in eu-west and ap-southeast. The edge routes each request to the nearest region.
- **Sizing target: 100k accounts** twelve months after launch.
  - The tick evaluates each aviary once a minute, about 1.7k aviary evaluations per second, and writes only rows that changed (§7.1).
  - Roughly 5k concurrently active aviaries means about 170 presence inserts per second and about 800 sim row writes per second. One primary handles that comfortably.
  - Four tick workers (2 vCPU each) lease 1,024 partitions.
- **Scaling path to about 1M accounts.** Hash-shard `sim` by `account_id` (Citus or application-level shards). The partition leases already assume this, so tick throughput scales linearly with workers.

---

## 5. Data model

### 5.1 Schemas and roles

- **Schemas.** `acct` holds identity and PII. `sim` holds simulation state and private interaction data. `social` holds visits. `sim` and `social` reference accounts only by UUID and hold **no** email.
- **Roles:**
  - `api_rw` has `INSERT` on `sim.interaction_events`. It has `SELECT` on `sim.*`, and `UPDATE` only on the user-editable columns: `sim.birds.name`, `sim.aviary_settings`, and `sim.offer_state`.
  - `sim_tick` has `UPDATE` on the personality, mood, plan, weather, and notebook tables.
  - `lifecycle` has `DELETE` for hard delete and retention.
  - `readonly_replica` is used by the regional APIs.
  - No role available to analytics or telemetry has any `sim` access at all.

### 5.2 `acct` schema

| Table | Key columns | Notes |
|---|---|---|
| `accounts` | `id uuid pk`, `email_ct bytea`, `email_dek_id`, `email_lookup bytea unique` (HMAC-SHA256 of the normalized email, secret pepper held in KMS), `status (active \| pending_deletion)`, `created_at`, `deletion_requested_at`, `hard_delete_after`, `settings jsonb` | The only table holding email besides `email_change_requests`. |
| `magic_links` | `token_hash pk`, `email_lookup`, `account_id null`, `purpose (sign_in)`, `expires_at (+15 min)`, `consumed_at` | Consumed with `UPDATE … WHERE consumed_at IS NULL AND expires_at > now() RETURNING`. |
| `email_change_requests` | `id`, `account_id`, `new_email_ct`, `new_email_lookup`, `token_hash`, `expires_at (+24 h, D-21)`, `consumed_at` | The account email changes only when this is consumed. |
| `sessions` | `id uuid`, `account_id`, `token_hash`, `device_label` (e.g. "Safari on Mac", derived from UA at creation), `created_at`, `last_active_day date`, `revoked_at` | `last_active_day` is day-granular, just enough to spot an unfamiliar device, never a visit history. |

### 5.3 `sim` schema

| Table | Key columns | Writer |
|---|---|---|
| `aviaries` | `id uuid pk`, `account_id uuid unique`, `created_at` (drives unlocks), `timezone`, `sim_version`, `last_applied_tick bigint`, `event_cursor bigint`, `next_tick_at`, `rng_key`, `weather jsonb`, `presence_day jsonb` (the local date plus raw presence minutes credited today, used for saturation and overwritten daily), `last_presence_end_at`, `newcomer jsonb`, `unlock_schedule jsonb` | tick |
| `aviary_counters` | `aviary_id pk`, `next_event_seq bigint` | API ingest (row-locked) |
| `birds` | `id uuid pk` (the permanent identity), `aviary_id`, `species_id`, `species_rev`, `name text`, `name_version int`, `voice_print jsonb` (immutable), `look_seed int`, `greeting_signature jsonb` (immutable), `adopted_at`, `arrival_order` | Created by adoption. `name` is written by the API. Everything else is immutable. |
| `bird_personality` | `bird_id pk`, `boldness`, `warmth`, `vocal`, `plumage`, `curiosity` (float8 in [0,1]), `ceilings jsonb`, `filter jsonb` (per-trait EMA signals `s_i` and their timestamp), `attunement float8`, `held_delta jsonb` (used during a drift freeze, §18.4), `updated_tick` | tick only (INV-1). A trigger rejects decreases (INV-2). |
| `bird_mood` | `bird_id pk`, `mood enum(alert, curious, content, wary, drowsy, roosting)`, `mood_since`, `biases jsonb` (decaying interaction, contagion, and weather terms), `perch_zone`, `perch_slot`, `activity jsonb` | tick (plus server resolvers through plan amendments) |
| `bird_affinity` | `(bird_a, bird_b) pk`, `affinity float8`, `updated_tick` | tick. Bird-to-bird only, never user-derived. |
| `offer_state` | `aviary_id pk`, `active_item jsonb`, `per_bird_last_offer jsonb` | Offer resolver (row lock) |
| `action_queue` | `aviary_id pk`, `committed_until`, `actions bytea` (compact CBOR ring, horizon 180 s) | tick. Amended by the offer and arrival resolvers under the aviary lock. |
| `plan_overlays` | `id`, `aviary_id`, `created_at`, `effective_until`, `actions bytea` | Offer and arrival resolvers. Merged into snapshots on read. |
| `interaction_events` | `(account_id, seq) pk`, `client_event_id uuid` (UNIQUE with `account_id`), `device_session_id`, `kind enum(presence, listen_in_start, listen_in_end, offer, settle, settle_undo, arrival, audio_state)`, `bird_id null`, `payload jsonb` (schema-validated, small), `client_ts`, `server_ts` | Insert-only for the API. Partitioned by `server_ts` day. Dropped 14 days after consumption (D-24). |
| `snapshots` | `aviary_id pk`, `tick`, `blob bytea` (compressed) | tick. Materialized only for aviaries that have had a viewer in the last 10 minutes; otherwise built on demand. |
| `notebook_entries` | `id uuid`, `aviary_id`, `local_date`, `created_at`, `text` (stored rendered, so later template changes never rewrite history), `detector`, `template_id`, `bird_ids uuid[]` | Notebook observer (append-only; no UPDATE grant) |
| `notebook_state` | `aviary_id pk`, `tokens float8`, `last_entry_at`, `recent_templates text[]`, `facts jsonb` (a 14-day rolling window of aviary-internal facts, e.g. which bird greeted first each day and front-perch firsts) | Notebook observer |
| `aviary_settings` | `aviary_id pk`, `captions bool`, `reduced_motion enum(system, on, off)`, `keep_top_bar bool`, `single_key_shortcuts bool`, `narration_visible bool`, `visit_notifications bool DEFAULT false` | API |

**Personality seeding.** At adoption, each trait is drawn from a species baseline plus a small per-bird jitter (N(0, 0.05)), clipped to [0.20, 0.45]. That leaves real room for expressive growth. Each trait gets a per-bird ceiling cᵢ drawn from [0.75, 0.95], so long-lived birds don't all converge on the same maximum (R-3).

### 5.4 `social` schema

| Table | Key columns | Notes |
|---|---|---|
| `invites` | `id`, `host_account_id`, `visitor_email_ct`, `visitor_email_lookup`, `sender_name null`, `token_hash`, `status (outstanding \| active \| revoked \| expired \| used_up)`, `created_at`, `expires_at (created + 30 d, if unused)`, `redeemed_at`, `revoked_at` | Visitor email is a non-user's PII and gets the same encryption discipline. |
| `visitor_passes` | `id`, `invite_id`, `token_hash`, `created_at`, `expires_at (+14 d)`, `revoked_at` | Scope: read the host snapshot only. |
| `visits` | `id`, `invite_id`, `started_at`, `last_seen_at` | Duration is approximated from keepalive pulls. Kept while the invite exists plus 180 days (§20). |

### 5.5 Write-ownership matrix (conflict prevention by construction)

| Data | Sole writer | Concurrency mechanism |
|---|---|---|
| Personality, filter, attunement | Tick | Partition lease, row lock, cursor in the same transaction, monotonic trigger |
| Mood, perch, activity | Tick | Same |
| Action queue | Tick; offer and arrival resolvers (server code only) | Aviary row lock. Amendments only touch actions at least 5 seconds in the future. |
| Weather, notebook | Tick and observer | Same transaction as the tick, or the observer's own cursor |
| Offer cooldowns and active item | Offer resolver | `SELECT … FOR UPDATE` on `offer_state` |
| Interaction events | API ingest (append-only) | Per-account sequence under the `aviary_counters` row lock, plus `client_event_id` uniqueness |
| Bird names | API (the user) | `If-Match: name_version`. A conflict returns 412 with matter-of-fact copy (§6.7). |
| Settings | API (the user) | Per-field PATCH with a version |
| Account email | API, only through verification | Token consumption |

No field has two writers of different kinds, so there's never a last-write-wins race on personality state.

### 5.6 Retention

| Data | Retention |
|---|---|
| Interaction events | 14 days after consumption (partition drop) |
| Presence day counter | Overwritten each local day, with no history |
| Notebook facts window | Rolling 14 days |
| Application logs | 14 days. Account UUIDs appear only on error lines. Never emails or event payloads. |
| Backups (PITR) | 35 days. The privacy policy states that deleted data leaves backups within 35 days of hard deletion. |
| Exports | Object deleted after 7 days; link valid 72 hours |
| Magic links and email-change tokens | Purged 24 hours after expiry |
| Visitor passes | Purged 30 days after expiry or revocation |
| Aggregate metrics | 13 months. They have no account dimension. |

---

## 6. API surface

### 6.1 Conventions

- JSON over HTTPS, versioned at `/v1`.
- Host auth uses a `__Host-session` cookie (HttpOnly, Secure, SameSite=Lax). Visitor auth uses a `__Host-visit` cookie. They're distinct scopes.
- CSRF protection: SameSite plus a required `X-Aviary-Client` header (which forces a CORS preflight) plus an `Origin` check.
- Idempotency: every mutating event carries a client UUIDv7, and every other POST accepts an `Idempotency-Key`.
- Errors come as `{ code, retryable }`. The client maps each code to copy from the system-string catalogue, so the server never sends user-facing prose for system errors.
- Rate limits are keyed by account UUID or by an ephemeral HMAC of the email (TTL 24 h, never logged) for unauthenticated auth requests.

### 6.2 Endpoints

| Method and path | Auth | Purpose |
|---|---|---|
| `POST /v1/auth/magic-link` `{email}` | none | Always returns 202 (no account enumeration). Rate-limited per email-HMAC (5 per 15 min, 20 per day) and per IP. |
| `POST /v1/auth/verify` `{token}` | none | Consumes the link and sets the session. Returns `{ firstRun: bool }`. The first verify for a new email creates the account and aviary and seeds the starters (§7.13). |
| `POST /v1/auth/sign-out` | host | Revokes the current session |
| `GET /v1/account/sessions` · `DELETE /v1/account/sessions/{id}` | host | List and revoke sessions. Revocation takes effect on the next request (the session cache TTL is at most 30 s). |
| `POST /v1/account/email-change` `{newEmail}` · `POST /v1/account/email-change/verify` `{token}` | host / none | Two-step email change |
| `GET/PATCH /v1/account/settings` | host | Accessibility preferences and the visit-notification toggle |
| `POST /v1/account/export` | host | 202. A job builds the export and emails the link. |
| `GET /v1/account/exports/{id}` | host | Download. Needs a signed-in session plus the link's single-use token. |
| `POST /v1/account/deletion` · `DELETE /v1/account/deletion` | host | Schedule deletion, or restore ("I changed my mind") |
| `GET /v1/aviary/boot` (internal, edge → regional) | host | Snapshot plus greeting plus species list for the inline boot |
| `GET /v1/aviary/snapshot?reason=visible\|resume\|keepalive` | host | ETag = `tick:overlayVersion`, so an unchanged keepalive returns 304. `visible` and `resume` include a greeting; `keepalive` never does. |
| `POST /v1/aviary/events` | host | Batched non-offer events (§6.4). Returns 202. |
| `POST /v1/aviary/offers` | host | Synchronous offer resolution (§6.5) |
| `POST /v1/aviary/adoption` `{names[2]}` | host | First run: confirm the starter names, which triggers the arrival fly-in |
| `POST /v1/aviary/newcomer` `{action: welcome, name}` | host | Welcome the newcomer (D-9) |
| `PATCH /v1/birds/{id}` `{name}` + `If-Match` | host | Rename |
| `GET /v1/notebook?before=<cursor>&limit=20` | host | Reverse-chronological and paginated without end. Nothing is ever archived. |
| `POST /v1/visits/invites` `{visitorEmail, senderName?}` · `GET /v1/visits/invites` · `DELETE /v1/visits/invites/{id}` | host | Create, list, and revoke invites |
| `GET /v1/visits/log` | host | Visit log |
| `POST /v1/visits/redeem` `{token}` | none | Exchanges the single-use token for a visitor pass |
| `GET /v1/visits/snapshot` | visitor | Host snapshot without the greeting, or 410 `visit_unavailable` |
| `POST https://t.<domain>/m` | none | Aggregate metric buckets only (§14.4). A separate host and pipeline, with no cookies. |

### 6.3 Boot and snapshot contract

`GET /` for a signed-in host goes to the edge worker. It streams the HTML head straight away: critical CSS, the inline boot script, the statically rendered top bar, and preload hints. At the same time it calls `/v1/aviary/boot` on the nearest regional API, which reads the replica. It then streams the snapshot as `<script type="application/json" id="snap">`. If the regional call takes longer than 250 ms, the edge streams without the snapshot and the client fetches it itself (the quiet field covers the gap).

Snapshot shape (abridged; about 5–8 KB raw and under 3 KB gzipped with seven birds):

```json
{
  "v": 1,
  "serverTime": 1790000000000,
  "tick": 29833333,
  "aviary": {
    "tz": "America/Chicago",
    "weather": { "kind": "after_rain", "intensity": 0.3, "since": 1789999400000, "until": 1790001800000 },
    "newcomer": null,
    "settledHint": false
  },
  "birds": [{
    "id": "0190c3e2-7a1b-7c3e-9d2f-5b1e0a6c4f11",
    "name": "Pip", "species": "finch", "speciesRev": 3,
    "voice": { "seed": 219385 },
    "look": { "seed": 77120, "chroma": 0.83, "detail": 0.42 },
    "demeanor": "restless",
    "mood": "content",
    "perch": { "zone": "middle", "slot": 1 },
    "activity": { "kind": "preen", "since": 1789999997200 }
  }],
  "plan": {
    "until": 1790000180000,
    "actions": [
      { "t": 1790000002100, "b": 0, "k": "call", "ctx": "ambient", "seed": 55120, "i": 0.5 },
      { "t": 1790000003400, "b": 1, "k": "call", "ctx": "response", "re": 0, "seed": 9911, "i": 0.35 },
      { "t": 1790000012000, "b": 0, "k": "move", "to": { "zone": "front", "slot": 0 }, "style": "hop" }
    ]
  },
  "greeting": { "id": "gr_01J…", "actions": [ … ], "primary": 0 }
}
```

- `look.chroma` and `look.detail` are **derived, quantized render parameters** (1/64 steps) that mix species palette and plumage. They are not traits.
- `demeanor` is a 3-level quantized hint for micro-motion.
- The personality vector and attunement are absent (INV-3). The contract test fails the build if a field named after a trait appears anywhere in `contracts`.

### 6.4 Event submission

```json
POST /v1/aviary/events
{ "deviceSessionId": "ds_…", "clockOffsetMs": -132, "events": [
  { "id": "018f…", "kind": "presence", "clientTs": 1790000030000, "presentMs": 30000, "sustainedMs": 30000, "callsPerceivable": true },
  { "id": "018f…", "kind": "listen_in_start", "bird": "0190…", "clientTs": 1790000031200 },
  { "id": "018f…", "kind": "listen_in_end",   "bird": "0190…", "clientTs": 1790000211900 },
  { "id": "018f…", "kind": "settle", "clientTs": 1790000400000 },
  { "id": "018f…", "kind": "arrival", "greetingId": "gr_01J…", "clientTs": 1789999999000 },
  { "id": "018f…", "kind": "audio_state", "audible": false, "captions": true, "clientTs": 1790000100000 }
]}
→ 202 { "accepted": 5, "duplicates": 1 }
```

Server rules:

- The schema is validated with zod. Bird IDs must belong to the account's aviary.
- `presentMs` is at most the time elapsed since the same device session's previous ping plus 5 seconds of slack.
- Buffered offline events are accepted up to 24 hours old. Older ones are dropped. Presence can't be credited after the fact without limit.
- Sequence numbers are assigned under the `aviary_counters` row lock, so commit order equals sequence order (§8.2).
- `settle` is only sent once the 5-second undo window has closed. An undone settle is never sent.
- Visitor-pass requests get 403 (INV-5).

### 6.5 Offer resolution

```json
POST /v1/aviary/offers
{ "id": "018f…", "item": { "kind": "seed" } }           // or {kind:"song", fragment:"falling-three"} or {kind:"pool"}
→ 200 { "offerId": "…", "placement": { "zone": "front", "at": 1790000001400 },
        "reactions": [ { "b": 0, "response": "approach", "actions": [ … ] },
                       { "b": 1, "response": "wait_then_approach", "actions": [ … ] },
                       { "b": 2, "response": "watch", "actions": [ … ] } ],
        "overlayVersion": 17 }
```

- The client starts the item's placement animation locally the moment the user chooses it. Seeds scatter onto the front rail over about 800 ms; the pool fades in over about 1.2 s; the song fragment starts playing at once.
- Bird reactions are timed to start no earlier than 1.2 seconds after the gesture, which leaves time for a p95 offer round trip of 250 ms or less.
- Only one offered item can be in the scene at a time: seeds for about 90 s, the pool for about 150 s, a song fragment for its own length of about 3–6 s. While one is present, the matching menu item reads in naturalist voice ("seeds are on the front rail") and is inert. Nothing ever shows a cooldown.
- If the network fails, the client plays an offline resolution with a conservative outcome (birds watch but don't take the item) and queues the offer with `offline: true`. The server then credits only proximity (boldness drift), never acceptance.

### 6.6 Visit invitation flow

```
Host browser             Primary API                          Mail job          Visitor browser
 │ POST /visits/invites ─▶│ create invite: email enc + HMAC,        │                 │
 │   {visitorEmail}       │ token_hash, expires_at = now+30d,       │                 │
 │                        │ status=outstanding; rate limits         │                 │
 │                        │─ enqueue invite mail ──────────────────▶│ send https://…/visit#t=<token>
 │◀─ 201 {id, status}     │                                         │                 │
 │                        │                                         │   GET /visit (static page;
 │                        │                                         │   token stays in the fragment)
 │                        │◀── POST /visits/redeem {token} ─────────────────────────────│
 │                        │ atomically: outstanding ∧ unexpired → active; token consumed│
 │                        │ create visitor pass (14 d); Set-Cookie __Host-visit ───────▶│
 │                        │◀── GET /visits/snapshot (boot, then keepalive ~60 s) ───────│
 │                        │ visits row: started_at / last_seen_at                       │
 │                        │ if host opted in and no notice for this invite in 24 h →    │
 │                        │   enqueue matter-of-fact visit-notice email to host         │
 │ DELETE /invites/{id} ─▶│ revoke invite + pass immediately                            │
 │                        │◀── next snapshot pull ──────────────────────────────────────│
 │                        │── 410 visit_unavailable ───────────────────────────────────▶│ "This visit is no longer available."
```

- **Tokens live in the URL fragment.** That keeps them out of server logs and Referer headers, and link scanners that don't run JavaScript can't consume them. The same pattern is used for magic links.
- **Invite rules:**
  - At most 10 outstanding invites per host and 20 new invites per day.
  - At most 3 outstanding invites to the same address.
  - An expired or revoked invite can't be revived.
  - If an already-redeemed link is opened again, it shows "This visit link has already been used. Ask for a new invitation."
- **Visitor client** (same bundle, `mode=visitor`):
  - It never loads `PresenceTracker`, the event outbox, the gestures menu, the notebook, or account modules.
  - The top bar shows only the accessibility settings icon. Visitor accessibility preferences are stored locally.
  - It gets no greeting.
  - Narration and captions work exactly as they do for the host.
- **Host revocation** has no success toast. The invite simply moves to "Revoked" in the list, and that is the confirmation.
- **Visit log** (account settings → Visits): visitor email, date, approximate duration (from first pull to last pull), and outstanding invites with their expiry dates. No badge or indicator appears anywhere when a new visit happens.

### 6.7 Error surfaces and copy mapping (system voice)

| Code | Copy |
|---|---|
| `link_expired`, `link_used` | "We couldn't sign you in. The link may have expired. Try requesting a new link." |
| `session_expired` | "Your session timed out. Sign in again to keep watching." (Shown in the top-bar area. The aviary keeps rendering its last plan and winds down to idle-only continuation.) |
| `load_failed` (snapshot unavailable after 15 s) | "Something went wrong loading your aviary. Try reloading; if it keeps happening, get in touch." plus a Reload button |
| `offline` (more than 2 minutes) | "You're offline. The aviary will catch up when you reconnect." (Top-bar area, dismissible.) |
| `name_conflict` (412) | "This name was changed on another device. Here's the current name." |
| `visit_unavailable` | "This visit is no longer available." |
| `visit_link_used` | "This visit link has already been used. Ask for a new invitation." |
| `unsupported_browser` | "Pocket Aviary needs a recent version of Chrome, Safari, Firefox, or Edge." |
| `pending_deletion` | "Your account is scheduled for deletion on 24 October." plus an **I changed my mind** button, on every signed-in page, in the top-bar area. Never inside the scene. |

System notices appear only in the top-bar region, and the top bar won't fade while one is showing. None of them use the naturalist voice. None of them are toasts: they persist until resolved or dismissed, and they are never triggered by arrival.

---

## 7. Simulation engine

### 7.1 Tick scheduling and execution

- **Cadence.** Each aviary is evaluated every **60 s (cal)**. The phase is offset per aviary by `hash(aviary_id) mod 60 s` to spread load, so ticks don't bunch up at :00.
- **Partitions.** 1,024 partitions, assigned by `hash(account_id)`. Workers hold leases in `sim.partition_leases (partition, holder, fencing_token, expires_at)`: 30-second TTL, renewed every 10 seconds. Every tick transaction checks its fencing token, so a zombie worker whose lease has expired can't commit.
- **Logical tick number:** `tick = floor((now − epoch − phase) / 60 s)`. It's derived from time, not counted in a column. The CAS guard is `WHERE last_applied_tick < $tick`.
- **Write-on-change.** Every aviary is evaluated every minute, as the PRD requires ("runs whether or not any client is connected"). Rows are written only when something changed. The filter decay uses closed-form exponentials over elapsed time, so a quiet bird's filter state can be evaluated lazily and written on the next real change without any loss of accuracy.
- **dt-invariance.** All dynamics take the real elapsed dt and sub-step it (at most 15 minutes for drift, at most 5 minutes for mood). Running at 30 s, 60 s, or 120 s, or catching up after a 3-hour outage, gives the same trajectories within tolerance. A property test enforces this (§16.1). The cadence is therefore a cost knob, not a behavior knob.

### 7.2 Tick pipeline (one transaction per aviary, batched 50 per round trip)

1. **Lock.** `SELECT … FROM sim.aviaries WHERE id = $1 AND last_applied_tick < $tick FOR UPDATE`, and verify the lease fencing token.
2. **Consume events.** Read `interaction_events WHERE account_id = $a AND seq > event_cursor ORDER BY seq LIMIT 5000`.
3. **Presence credit.** Build the union of presence intervals across devices and intersect listen-in intervals with presence (§7.3).
4. **Drift.** Update filter state, compute Δ ≥ 0 for each trait, and apply it additively (§7.4).
5. **Attunement** (§7.5).
6. **Weather.** Advance the schedule and emit ambient events (§7.9).
7. **Mood.** Recompute logits and transition, with sub-stepping (§7.6).
8. **Plan.** Extend the action queue for aviaries with an active viewer: perches, calls, responses, chorus windows (§7.7, §7.8). Fold in overlays that have expired.
9. **Newcomer lifecycle** (§7.13).
10. **Commit.** Set `event_cursor = max(seq)`, `last_applied_tick = $tick`, and `next_tick_at`. Materialize the snapshot if a viewer is active.
11. **Post-commit.** The notebook observer takes the tick's domain facts, idempotently by `(aviary, tick)` (§7.14).

A crash before commit applies nothing, and the next tick redoes the work. A commit advances the cursor atomically, so no event is ever applied twice (INV-6).

### 7.3 Presence credit

- The client reports spans every 30 s: `presentMs`, plus `sustainedMs`, the portion that falls more than 3 minutes into a continuous presence run.
- The server turns each span into a server-time interval `[recv − presentMs, recv]`, adjusted by the reported clock offset, and caps it at the device session's elapsed time since its previous ping.
- **Account-level presence is the measure of the union of intervals across all device sessions.** Laptop and phone open together are worth what one device is worth, never double.
- Listen-in seconds for bird *b* only count where they overlap presence. A listen-in left running while the user walks away credits nothing once presence lapses, and the client auto-disengages it anyway (§10.5).
- Visitor traffic can't reach this path (INV-5).
- We don't guard against a user faking presence for their own aviary beyond these caps. It only affects their own birds, and with no leaderboard there's no reason to.

### 7.4 Drift function

Drift is a three-stage low-pass pipeline. It's monotonic by construction, and it's calibrated against named targets.

**Stage A: daily saturation.** Each unit of input is weighted by a marginal factor that falls as the local day's total grows:

```
Pe  = presence minutes · exp(−P_today / T_sat)          T_sat = 30 min (cal)
Pse = sustained presence minutes · exp(−P_today / T_sat)
Le_b = listen-in minutes on b (∩ presence) · exp(−L_b_today / 20 min)   (cal)
A_b = accepted offers by b, counted up to 5 per local day
N_b = offers placed within near-radius of b, counted up to 6 per local day
aud = fraction of presence during which calls were perceivable (D-5)
```

Diminishing returns hold a four-hour-a-day watcher to about 2.5× the drive of a 15-minute-a-day visitor. This is the main guard against Tamagotchi-speed drift for heavy users.

**Stage B: drive per trait**, in attention-minutes (am). The weights are ordered as the PRD orders them: presence dominant, then listen-in, then offers.

| Trait | Drive xᵢ |
|---|---|
| Boldness | `1.0·Pe + 2.0·N_b + 0.3·Le_b` |
| Social warmth | `0.8·Pe + 1.5·Le_b` |
| Vocal frequency | `0.6·Pe·aud + 1.5·Le_b` |
| Plumage saturation | `1.0·Pse + 0.5·Le_b` (sustained attention only) |
| Curiosity | `0.5·Pe + 3.0·A_b` |

Settle adds no drive. It only ends the presence window and adds a mood bias (§7.6).

**Stage C: low-pass filter, then trait update.** This runs per bird and per trait, with dt in days:

```
sᵢ ← sᵢ · e^(−dt/τ) + xᵢ/τ                         τ = 4 days (cal)     # EMA of daily drive, am/day
h(s) = s / (s + s_half)                            s_half = 20 am/day (cal)
Δvᵢ = k · h(sᵢ) · (cᵢ − vᵢ)^γ · dt                 k = 0.028 /day (cal), γ = 1.5
Δvᵢ = clamp(Δvᵢ, 0, cap_day · dt)                  cap_day = 0.008 /day
vᵢ ← LEAST(cᵢ, vᵢ + Δvᵢ)                           (applied as an additive SQL update)
```

Properties that follow from this structure:

- **Monotonic.** sᵢ ≥ 0, h ≥ 0, and (cᵢ − vᵢ) ≥ 0, so Δvᵢ ≥ 0 always. Neglect makes sᵢ decay toward 0 and drift stops; nothing reverses it. The DB trigger is a backstop.
- **It continues through absence from earlier inputs.** The filter keeps delivering drift for a few days after the user leaves, and then it stops. That matches the PRD: drift during absence is based on inputs from before the user left, never invented ones.
- **Slow and saturating.** The per-day cap guarantees no profile can move a trait by one JND in under 7.5 days. The (c − v)^γ term slows growth as a bird matures and keeps individuals distinct.
- **Worked check with the initial constants.** Reference visitor, boldness, v₀ = 0.35, c = 0.9, steady drive ≈ 14.7 am/day:
  - Day 7: Δ ≈ 0.022, detectable by instruments; the harness threshold is 0.01.
  - Day 21: Δ ≈ 0.085, above the 0.06 visibility JND.
  - Heavy profile: reaches one JND around day 9–10.
  - A single isolated session of any length contributes at most about 0.017, under 0.3 JND.

**Defining "visible."** Each trait's behavioral mapping gets a just-noticeable difference, set by a perception study (§16.2): participants compare rendered aviaries at trait deltas in a 2-alternative forced-choice test, and the JND is the delta at 75% correct.

| Trait | Behavioral mapping (server planner and client render) | Initial JND (cal) |
|---|---|---|
| Boldness | Front-zone preference logit `+2.5·(b − 0.5)`. How close a bird comes on greetings. How fast it approaches offers. | 0.06 (about +10 pp front-zone occupancy) |
| Social warmth | Weight for greeting first. Probability of call-back. Tendency to perch near others. | 0.06 |
| Vocal frequency | Ambient call-rate multiplier from 0.6× to 1.6×. Probability of joining a chorus. | 0.07 |
| Plumage saturation | OKLCH chroma multiplier from 0.78× to 1.12×. Feather-detail opacity from 0.3 to 0.9. Sheen. | 0.05 (ΔE₀₀ of about 2.5 on the primary color) |
| Curiosity | Probability of approaching an offer. Head-tilt rate toward new sounds and leaves. | 0.07 |

**Calibration targets.** The harness asserts these for every change to `sim-server`:

| Profile (synthetic) | Behavior | Day 7 | Day 21 | Guard |
|---|---|---|---|---|
| Reference regular | 6 visits a week, 15 min of presence, 3 min of listen-in on one bird, 1 offer | Δ ≥ 0.01 on at least 3 traits; every trait under 1 JND | At least 2 traits at 1 JND or more | — |
| Light | 2 visits a week, 10 min | Δ > 0 measurable on at least 2 traits | Every trait under 1 JND | At least one trait reaches 1 JND by day 60 |
| Heavy | 4 h a day of active presence, maximum offers | Every trait under 1 JND | — | No trait reaches 1 JND before day 9 |
| Tab open all night | Visible and focused, no activity | Δ = 0 | Δ = 0 | Credit is exactly zero once the activity window passes |
| Multi-device overlap | Laptop and phone active for the same 30 min | The same as one device for 30 min | — | Credit is the union, not the sum |
| Returner | 3 weeks regular, then 14 days absent | — | — | No Δ is ever negative. Drift stops within about a week of leaving. Mood never biased toward wary. |
| Single session | One session of any length, 0–8 h | — | — | Total Δ under 0.3 JND on every trait |

### 7.5 Attunement (D-4)

For each bird, αᵦ ∈ [0.30, 1.0]:

- **Rise with presence:** `α ← α + (1 − α)·(1 − e^(−Pe/40))`. Two or three regular sessions bring it near full.
- **Decay with absence:** `α ← 0.30 + (α − 0.30)·e^(−dt/10 d)`, a half-life of about 7 days.
- **Used only for:** the chance a bird *joins* a return-greeting as a secondary greeter, and how far forward it comes on arrival.
- **Explicitly not used for:** mood valence, call rate, call identity, plumage, perch choice outside the greeting, or anything in the vector.
- **The effect:** after two weeks away, one bird still notices the user, with a longer re-orientation greeting (§7.10). The others are quieter toward the user, never wary. A few sessions later, the aviary has eased back.

### 7.6 Mood model

- **States:** `alert, curious, content, wary, drowsy, roosting`. `roosting` is the eyes-closed night state. It's kept distinct from the aviary-level "settled" lighting to avoid overloading the word.
- **Target distribution** per bird: πₘ ∝ exp(Σ components).

| Component | Terms (cal) |
|---|---|
| Circadian prior Cₘ(local time, species) | Alert peaks 05:30–08:00 (the dawn chorus). Curious and content through midday. Drowsy from dusk. Roosting across the night, around 0.85 at 23:00–04:30. The nightjar-like species runs inverted: active at dusk and through the night, drowsy or roosting by day. |
| Personality Pₘ | wary `−2.0·(boldness − 0.5)`; curious `+1.5·(curiosity − 0.5)`; content `+1.0·(warmth − 0.5)`; alert `+0.8·(vocal − 0.5)` |
| Recent interaction Iₘ | Accepted offer: content +0.8 and curious +0.4, decaying with a half-life of 2.5 h. Listen-in: content +0.5 on that bird. Settle: drowsy +0.6 and alert −0.6, half-life 90 min. |
| Ambient Wₘ | Rain: drowsy +0.4, alert −0.3, and calls damped to 0.5× for the rain plus 10 minutes. Wind: alert `+0.6·b` and wary `+0.6·(1 − b)`. |
| Contagion Kₘ | Wary gets `+0.9 · Σⱼ proximityᵢⱼ · [mⱼ = wary] · (1 − bᵢ)`. An alarm call adds wary +1.5, decaying over 20 minutes. |

- **Transitions** are hazard-based: rate 1/τ_mood, with τ = 35 min by day and 90 min at night (cal), and a minimum dwell of 8 minutes. Startles (an alarm call, a wind gust) bypass the dwell. Each transition samples from π. The RNG seed is `(aviary.rng_key, bird, tick, substep)`, so ticks are reproducible (§7.15).
- **No absence term exists.** No input for time since last visit enters any mood logit. This is how "no Tamagotchi" is expressed in the mood layer.
- **The daily reset (D-3).** I and K decay within hours, so by morning each bird's mood is set by circadian rhythm and personality, having carried forward smoothly from the night before. Opening a tab changes nothing.
- **Alarm sources** are ambient only: a wind gust startling a timid bird, or a rare "passing shadow" event (around 0.3 per day, daytime only). Never the user.

### 7.7 Behavior planner and the committed action queue

- **Action kinds:**
  - `move{zone,slot,style: hop|flight}`
  - `call{ctx: ambient|response|greeting|alarm|chorus|offer_response|night}`
  - `preen{duration}`
  - `forage{target}`, `drink`, `bathe`
  - `lookAt{target: viewer|bird|item}`
  - `roost`, `wake`, `flyIn`, `depart` (newcomer only)
- **Rolling committed queue.** The horizon is 180 s, and each tick commits one more minute (the window from +120 s to +180 s). Actions already committed are immutable. Only server-side amendments (offer reactions, greetings) may change them, and only for actions starting at least 5 seconds in the future. Two devices on neighboring snapshots therefore always agree about the present, and nothing ever visibly jumps.
- **Bootstrapping dormant aviaries.** When nobody has viewed an aviary for 10 minutes, its queue isn't extended; canonical perch and mood still advance at tick granularity. On the next arrival, the queue is rebuilt deterministically from `(state, tick)`, so every replica produces the same bootstrap, and the tick adopts it.
- **Calls** follow a non-homogeneous Poisson process. `λ = base_species × vocal_mult(v) × mood_mult × circadian × weather_damp`. An onset-collision rule delays a call by 150–600 ms if another bird started one less than 150 ms earlier, except inside chorus windows. Real birds avoid acoustic overlap too, and it preserves recognizability.
- **Perch choice** is a softmax over free slots. Inputs: zone preference (boldness, mood: wary pushes back, curious and alert pull forward), crowding, pairwise affinity, and a small inertia term. Move rates vary by mood: alert about one every 3 min, content one every 8 min, drowsy one every 20 min, roosting none. There's one bird per slot.
- **Slots:** front 3, middle 3, back 3. Nine slots comfortably hold seven birds plus a newcomer on the back-edge perch.

### 7.8 Bird-to-bird interaction

- **Call-response.** When bird *i* calls, bird *j* responds with probability `0.25 × warmth_mult(wⱼ) × affinityᵢⱼ × mood_factor(mⱼ)` (roosting gives 0), after a delay drawn from U(0.4, 2.5) s. A response may echo the caller's contour. Exchanges are capped at 3.
- **Chorus windows** last 20–60 s. They happen when at least two awake birds have high relative vocal frequency. The daily high point is the dawn chorus (05:30–07:30 local). At other times the chance per 10 minutes is `0.08 × (eligible − 1)`. Inside a window, call rates rise and overlap is allowed.
- **Wary spread** runs through the contagion term (§7.6).
- **Affinity** is a slow EMA of time spent perched together (τ = 14 days). It lives on the server, only affects bird-to-bird behavior, and is never exposed.
- **Head turns:** the client makes birds look toward other birds' calls, so the server doesn't need to send that.

### 7.9 Weather and light

- **Rain.** About 3–5 a week (Poisson, cal), each 10–25 minutes long, scheduled only in waking hours (07:00–22:00 local). Each rain is followed by an **after-rain** phase of 30–60 minutes: droplets on leaves, birds shaking out wet feathers, slightly damped calls. The aftermath raises the chance a daily 15-minute visitor sees weather at all to roughly once every three weeks, without anything happening for the viewer's benefit.
- **Wind.** Soft gusts of 1–4 minutes, about 1–2 a day, rippling the leaves.
- **Never:** thunder, snow, storms, or anything the user must notice.
- **Light** is a pure function of the canonical aviary timezone (D-11). Keyframes: night (21:30–04:45), pre-dawn, sunrise (06:00–07:30), morning, midday (brightest, 11:00–15:00), afternoon, evening (warmer, 17:30–19:30), dusk. Palettes are interpolated continuously in OKLCH. The client computes light itself; the server uses the same curve for circadian mood.

### 7.10 Return-greeting resolver

Inputs:
- Absence **A** = now − `last_presence_end_at`, account-wide and so correct across devices.
- Each bird's boldness, warmth, mood, attunement, and perch.
- Local time.
- `seed = hash(aviary, arrival_id)`.
- Each bird's immutable **greeting signature**, drawn at adoption. For example, pip favors a head-tilt and two notes, while wren steps forward in silence. This is why the same bird greets the same way across visits while different birds greet differently.

Steps:

1. **Absence shaping** is continuous, not a set of fixed scripts. Its anchors:

   | Absence | Greeting |
   |---|---|
   | Under 10 min | A glance |
   | 10 min to 3 h | A notice |
   | 3 h to 36 h | An approach |
   | Over 36 h | A re-orientation |

   Vocal length, how far the bird moves, and the chance of a follow-up all grow with `log(A)`.
2. **Primary greeter.** Sample the bird (don't just take the top one) with weight `exp(1.6·b + 1.0·w + readiness(mood) + 0.5·α)`. Roosting birds count ×0.15, and wary birds ×0.4. The bolder bird usually greets first, but not always; the warier bird greets later or not at all. **Exactly one bird always notices.** If every bird is roosting, it's a sleepy notice: one eye opens, a shift on the perch, perhaps a soft night call.
3. **Form composition** is a grammar over four parts:
   - **Attention cue:** head turns toward the viewer, preening stops, a bird lifts out of its fluff, an eye opens.
   - **Vocal cue:** none, one soft note, two notes, a phrase, or a phrase that repeats.
   - **Movement:** none, a shuffle, a hop forward one slot, or a flight to the front zone.
   - **Social follow-up:** another bird calls back, another bird looks, or nothing.

   The bird's signature biases which parts are chosen. The continuous parameters (timing, head angle, pitch contour, hop arc) are drawn from the seed. A wary primary uses a vocal-only cue from where it sits, or a look only.
4. **Secondary greeters.** Each other bird joins with probability `f(α, warmth, mood, absence)`. The first onset lands 350–1500 ms after the first frame or visibility return, which keeps it inside the PRD's "first second or two". Each later onset adds U(0.6, 2.4) s. Two onsets are never less than 400 ms apart, and nothing fires in unison.
5. **Refractory** (D-25).
6. **Output.** Greeting actions, a narration hint, and the notebook fact `greeted_first = bird`. The fact describes the bird, not the user.

**Nothing canned.** There are no stored greeting variants. The CI test (§16.5) generates 10k arrivals per fixture aviary and asserts that the parameter hashes are all unique and that a perceptual feature distance exceeds ε between any two greetings.

### 7.11 Offer resolver

- **Placement.** A seed scatters on the front rail. A still pool appears at the front of the ground plane. A song fragment is played from the front center of the scene, from a library of 6 short melodic motifs with naturalist names ("a falling three-note phrase", "a slow rise", "two soft notes, repeated", "a long low line", "a quick bright figure", "a wandering figure").
- **For each bird**, the chance of each response is a function of mood, curiosity, boldness, distance to the item, and cooldown:
  - **Seed:** `approach` (curious or content bird) · `wait_then_approach` (wary; comes near after 10–60 s) · `watch` · `ignore` (drowsy or roosting)
  - **Pool:** `drink` · `bathe` · `watch`
  - **Song:** `join` (imitates the fragment's contour in its own voice) · `quiet` (stops calling for 20–60 s) · `counter_call`. Vocal frequency and mood weigh most here.
- **Acceptance.** `approach`, `wait_then_approach` (once it arrives), `drink`, `bathe`, and `join` count as **accepted**, which credits curiosity drift. Every bird inside the near-radius gets proximity credit, which credits boldness drift. At most two birds can accept one item.
- **Cooldown.** 4 minutes per bird (cal), enforced by the server. A bird in cooldown watches or glances but doesn't engage, and gets no credit.
- **Consistency.** The resolver reads canonical state from the primary and writes the offer event (with its outcomes), the cooldowns, and a plan overlay, all in one transaction. The next tick folds the outcome into perch and mood.

### 7.12 Call-grammar runtime (server half)

The server schedules *descriptors*: `{t, bird, ctx, seed, intensity}`. It never schedules audio. Realization happens client-side (§10.3). Because the same descriptor plus the pinned `species_rev` plus the bird's `voice_print` always produces the same call, every device and visitor hears the same Pip at the same moment, and variation comes from the fresh seed on each call.

### 7.13 Adoption, newcomers, cap

- **First run.**
  1. On the first magic-link verify for an email, create the account, aviary, and two starter birds. The pair comes from the curated contrast table (D-27), which maximizes difference in silhouette and call register.
  2. Draw the voice prints, rejection-sampling until the two are distinct enough (§10.3).
  3. Draw greeting signatures, look seeds, trait seeds, and ceilings.
  4. Mark the birds `arriving`.
  5. The naming card appears over the quiet field (§9.7). Its default names are suggestions from a species-appropriate list.
  6. When the user confirms (`POST /adoption`), the birds fly in one at a time, 2–4 seconds apart. This is the only entry animation in the product (INV-13).
  7. The observer writes the first notebook entry, e.g. "two birds came to the aviary this morning. pip took the front perch first."
- **Unlock schedule** (INV-7). It's computed from `aviary.created_at` alone:

  | Bird | Unlocks around day (cal) |
  |---|---|
  | 3rd | 90 |
  | 4th | 160 |
  | 5th | 240 |
  | 6th | 330 |
  | 7th | 450 |

  Each threshold carries a seeded ±10% jitter so it doesn't feel scheduled, and all of them are gated by the global config `max_birds_enabled` (§18.3).
- **Newcomer lifecycle** (D-9).
  - **Arriving.** For up to 21 days, a wild bird of the chosen species perches on the back-edge perch in 20–40-minute daytime stretches, several times a day. It calls occasionally with its own voice print.
  - **Welcome.** The gestures menu shows *make room for the thrush*, which opens the naming card. Confirming turns it into a full bird with fresh trait seeds (no inherited stats) and writes a notebook entry.
  - **Not welcomed.** At the end of the window it quietly departs, and the next newcomer becomes possible after 30 days. There's no penalty and no copy about it.
  - **Species choice.** Prefer species not yet in the aviary, and pick the voice print that maximizes distinctness from the birds already there. Visible birds, including the newcomer, never exceed 7.

### 7.14 Notebook observer

- **Inputs** are aviary-internal domain facts from each tick and arrival, never user-behavior facts:
  - greet-first order and whether it changed
  - a front-perch first, or a first in N days
  - chorus events (especially dawn choruses with 3 or more birds)
  - rain and wind and what the birds did after
  - long preen or long quiet stretches
  - plumage crossing a visibility JND
  - two birds perching together more often lately
  - the nightjar calling late
  - newcomer arrival and adoption
  - offer outcomes, described from the bird's side only ("a seed on the front rail; wren took it, then watched for a while")
- **Scoring.** Each candidate gets a noteworthiness score and a novelty penalty (for the same detector or template skeleton in recent entries).
- **Sparsity governor.**
  - A token bucket with capacity 2, refilling 1 token every 72 hours (cal), so roughly one entry every few days for a well-visited aviary.
  - Exceptional facts (score ≥ 0.9: arrival, adoption, a first) may borrow one token.
  - Hard maximum: one entry per local day, except adoption day.
  - A very active user gets no more entries than a regular one; a neglected aviary gets fewer, because fewer greetings happen.
- **Prose** comes from `@aviary/voice` (§11). It's stored as rendered text with the local weekday or date header ("tuesday — pip greeted before wren today, first time this week.").
- **Forbidden detector types, enforced by lint in code review:** anything about visit frequency, absence length, session counts, streaks, or "you." The observer's input type has no field for them, so they can't be expressed.

### 7.15 Determinism, versioning, shadow mode

- A tick is a pure function of `(state, events, dt, seed(aviary, tick))`, and every commit records `sim_version`.
- **Shadow mode.** New `sim-server` versions run on 5% of partitions, computing results without committing them. They're diffed against production output as aggregate distributions of Δ with no account dimension, computed in memory and exported only as aggregate histograms.
- **Promotion** requires the calibration harness to pass, the shadow diff to be within tolerance, and product sign-off.
- **Species revs are immutable once any bird is pinned to them.** Moving a bird to a new rev requires its voice-print distance to its own old voice to stay under a recognizability threshold, and a visual identity snapshot diff check.

---

## 8. Sync model

### 8.1 Principle

There's one canonical aviary per account on the server. Clients are views: they render snapshots and submit events, and they never own simulation state. Multi-device sync isn't a feature to build; it comes from the write-ownership matrix (§5.5).

### 8.2 Ordering and exactly-once

- **Sequence allocation.** `UPDATE aviary_counters SET next_event_seq = next_event_seq + $n WHERE aviary_id = $1 RETURNING` runs inside the ingest transaction, and the row lock is held until commit. That makes per-account commit order equal sequence order. The tick's `seq > cursor` read therefore never skips a late-committing lower sequence. (The classic failure with a global `bigserial` is explicitly avoided.)
- **Duplicates.** Retried events hit the `(account_id, client_event_id)` unique constraint and count as `duplicates`.
- **The tick** applies events strictly in sequence order, and advances the cursor in the same transaction as the state change.

### 8.3 Multi-device scenarios

| Scenario | Behavior |
|---|---|
| Laptop in the morning, phone at lunch (the PRD's last-write-wins example) | Each device only posts events. The tick adds both deltas in sequence order. Nothing is overwritten, because no device ever writes the vector. |
| Both devices open and active at once | Presence is the union (§7.3). Listen-in is a local mix on each device, and both listen-ins earn drift credit. Plans are identical, so both devices show the same bird on the same perch doing the same thing. |
| An offer on the laptop while the phone watches | The laptop gets the reaction synchronously. The phone sees it in the next snapshot (keepalive at most 60 s, or sooner after a visibility change) through the overlay merge, so the bird walks up to the seeds in the same way. |
| Settle on the laptop | The laptop goes to settled lighting. The phone keeps normal lighting, and the birds' mood quiets canonically on the next tick (D-14). |
| Renames from two devices at once | `If-Match` on `name_version`. The second write gets 412 and the matter-of-fact conflict copy. |
| A session revoked from another device | The next request returns 401, and the device shows the session-timed-out copy. Buffered events for that session are discarded. |

### 8.4 Snapshot freshness and replica lag

- **Never go backwards.** Every snapshot carries `tick` and `overlayVersion`, and a client ignores any snapshot older than the one it holds. A lagging replica can't make birds jump back.
- **Read-your-writes for offers and greetings.** The client keeps a local overlay until a snapshot with an `overlayVersion` at least as high arrives.
- **When the client pulls:**
  - on boot
  - on `visibilitychange` to visible
  - on rAF gaps over 2 s (suspend or resume; treated as an arrival if presence had lapsed)
  - on a keepalive every 60 s ± 10 s while visible, sent with `If-None-Match` so it's usually a 304

### 8.5 Plan continuity on the client

- **Choreographer merge.** When a new snapshot arrives, actions that start within the next 5 seconds come from the plan the client already holds. Everything after that comes from the new plan. The server's committed-prefix rule makes a conflict rare.
- **Reconciliation.** If a bird's rendered state disagrees with where the new plan says it is now, the client schedules a natural hop, flight, or turn to get there within 2 seconds. **Visible birds never teleport.** In reduced-motion mode, the correction is a cross-fade.

### 8.6 Clock

- **Offset.** NTP-style: `offset = serverTime + rtt/2 − clientNow`. It's re-estimated on every pull with a median filter over the last 5 samples.
- **Audio time:** `audioT = ctx.currentTime + (serverT − (Date.now() + offset))/1000`, with a lookahead of at least 100 ms.

### 8.7 Offline

- **Buffered events.** Events go into an IndexedDB outbox (at most 500 events or 24 hours) and flush with backoff on reconnect. The final flush on `pagehide` uses `fetch(…, {keepalive: true})`.
- **Visual continuation.** When the plan runs past its 180-second horizon, the client continues on a local provisional plan: idle only, no perch changes, calls at the same rate, no greetings. When fresh state arrives, the birds reconcile naturally.
- **Offline offers** follow §6.5. The offline notice follows §6.7.

---

## 9. Frontend rendering pipeline

### 9.1 Boot sequence and time to first bird

Targets are for the reference mid-tier phone on 4G with a warm asset cache, which is the normal case, because the sign-in flow warms the cache before a user first sees the aviary.

| t (ms) | Event |
|---|---|
| 0 | Navigation |
| ~150–250 | The HTML head arrives. The inline boot script (≤ 6 KB) runs the feature check and paints the **quiet field**: a sky gradient for the device's local time, plus one faint CSS-animated cloud drift (none in reduced motion). The top bar markup is static server HTML. |
| ~250–350 | The inline snapshot arrives. `boot.js` (≤ 45 KB gz, from the service worker or HTTP cache) runs, lays out the scene, and evaluates the plan at the current server time. |
| **≤ 450** (budget 500) | **First bird frame.** Every bird is drawn mid-activity with its phase set to `now − activity.since`. Ambient leaves are pre-warmed mid-flight at random positions. Micro-motion noise phases are seeded, so nothing starts from rest. `performance.mark('first-bird')`. |
| +0–400 | Deferred modules load: audio engine, voice runtime, event outbox, PresenceTracker, then chrome hydration (top bar, gestures menu). |
| 350–1500 after the first bird frame | The greeting begins with its first onset (§7.10). |

- **Cold cache.** The boot chunk plus species rigs add about 100–150 ms on 4G, still inside 500 ms at p75. Species rig files are immutable and preloaded from the snapshot's species list.
- **No snapshot yet.** The quiet field holds. There's no spinner anywhere, ever (INV-13). After 15 seconds, the `load_failed` system notice appears in the top-bar area.
- **Rejected alternative:** a server-rendered SVG first frame. The hand-off to canvas would risk a visible discontinuity, which is worse than 100 ms of quiet field.

### 9.2 Scene composition

The canvas layers, back to front:

1. **Sky gradient:** re-rendered offscreen only when the palette moves past a threshold, about every 10 s during transitions.
2. **Far foliage:** a pre-rendered bitmap for each palette keyframe, cross-blended, with gentle sway.
3. **Perch structures** for the back, middle, and front zones. Back-zone birds are drawn at scale 0.62 with slight haze and desaturation, middle at 0.8, front at 1.0.
4. **Birds**, z-ordered by zone and then by y.
5. **Offer items:** seeds, and the pool with a blurred, low-opacity reflection of any bird within reach.
6. **Foreground ornaments:** occasional passing branch or leaf silhouettes, and the leaf and feather particle pool (at most 12, from an object pool).
7. **Weather:** rain streaks, and leaf bursts on wind.
8. **Light grade:** interpolated per-layer palette LUTs, plus the settle grade.

On top of the canvas sit the DOM layers: the accessibility overlay (bird focus targets), captions, the top bar, and panels.

- **Parallax** is an ambient "breathing" drift of ±4 px over about 20 s per layer, not tied to the pointer. Pointer-driven parallax reads as a UI trick. It's off in reduced motion.
- **The render loop.** One `requestAnimationFrame`, evaluated in server time. The render loop allocates nothing per frame (enforced by an allocation-sampling test). Part atlases are baked per bird from the `look` parameters, and re-baked only when those change, which is rare.
- **Backing store.** CSS pixels × min(DPR, 2), capped at 4.0 MP. An adaptive render scale (0.6–1.0) steps down by 10% if frame-time p90 stays above 14 ms for 5 seconds, and steps back up after 30 seconds of headroom. Only an aggregate counter records these changes.

### 9.3 Birds: rigs and idle micro-motion

- **Rigs.** Each species is a 2D rig: body, head (profile, three-quarter, and front yaw poses, so a bird can turn to *look at the viewer*), beak (upper and lower), eye and lid, wings (folded and spread), tail, legs, and a feather-fluff scale. Rigs and pose libraries are authored in `tools/rig-editor` and compiled to compact JSON, 6 KB gz or less per species.
- **Individuality.** Each bird's `look.seed` adds small markings (a pale wing bar, a slight asymmetry). That keeps two birds of the same species visually distinct, which matters because seven birds from six species means at least one species repeats.
- **Micro-motion channels** layer additively on the rig:

| Channel | Behavior | Mood shaping |
|---|---|---|
| Breathing | Continuous, each cycle jittered ±15% | Drowsy slower and deeper. Alert quicker. |
| Head saccades | Quick turns and holds (lognormal fixation times) toward points of interest: calling birds, leaves, offer items, the viewer | Wary: scans more, with shorter holds. Curious: tilts toward sounds and watches leaves. |
| Weight shuffle | Occasional (Poisson) | Restless demeanor more often |
| Preen bouts | Head to wing or breast, 5–40 s | Content preens |
| Tail flicks, wing settles | Occasional | Alert more |
| Blinks and lids | Poisson blinks | Drowsy: slow blinks, half-closed eyes. Roosting: closed. |
| Fluff | Scale 1.0–1.25 | Morning cool, drowsy, and after rain fluff up ("fluffed against the cool air") |
| Beak and throat | Driven by the audio scheduler's note events (sync within ±30 ms) | Follows each call's amplitude envelope |

- **No fixed-period loops.** All timing comes from seeded distributions and 1D simplex noise. There are no fixed-period cycles, which avoids the "strobing micro-animation" failure. Every luminance change stays under 3 flashes per second (WCAG 2.3.1).
- **Mood changes** blend channel weights over 20–60 s; they never snap.

### 9.4 Macro transitions

- **Hop within a zone:** 250–450 ms, with a crouch, a hop arc, and a landing wobble.
- **Flight between zones:** a 700–1400 ms Bézier arc with a wingbeat cycle and takeoff and landing poses. Arcs are clamped to the viewport, and adopted birds never leave the frame.
- **Newcomer and starter arrival:** a soft fly-in from the edge of the scene.
- **Day and night:** continuous.
- **Weather:** fades in and out over about 30 s.
- **Settle:** a lighting grade toward the evening palette over 4 s (6 s in reduced motion). The call bus drops 6 dB and call rates fall on the next plan.

### 9.5 Layout engine (responsive, never crops a bird)

- **Space.** The scene fills the viewport. The top 12% (plus safe-area insets) is sky, reserved for the top bar, and birds never occupy it. Perch zones sit in the bands between 40% and 88% of height.
- **Slot positions.** Slots are spread across the width with side margins of at least max(16 px, 4% of width). Bird scale is `min(heightScale, widthScale)`, chosen so that three birds per zone, plus a gap of at least 0.25 × bird width, always fit.
- **Portrait phones.** Extra height goes to sky and foliage; the scene is never letterboxed with bars. **Ultrawide screens** clamp the maximum bird scale and spread the perches.
- **Invariant test.** Every viewport from 320×480 to 3840×2160 is tested, in 5% steps and in both orientations. Every adopted bird's bounding box, including its flight arcs and fluffed scale, must stay inside the viewport and outside the top-bar band.
- **Resize** re-lays out immediately while the user is dragging, debounced. On orientation change the birds hop to their new positions over 600 ms.

### 9.6 Top bar

- **Contents.** Four icon buttons, each at least 44×44 px, drawn as calm custom glyphs: account/settings, accessibility settings, field notebook, gestures (offer and settle, D-1). They sit on a soft gradient backdrop, not a solid bar. `TopBarIcon` has no badge API.
- **Fade.** After 4 s with no pointer movement, the bar fades over 1.2 s to 10% opacity. It returns within 150 ms on pointer movement, any key, or a tap in the top band.
- **It never fades** while focus is inside it, while a menu or panel is open, while a system notice is showing, or when **Keep the top bar visible** is on in accessibility settings.
- **Contrast** of glyphs against the backdrop is at least 3:1 in every light phase; at night the glyphs switch to a light treatment.

### 9.7 Panels and other screens

- **Gestures menu.** A popover anchored to its icon: `offer a seed`, `offer a song fragment ▸` (the six fragments), `offer a still pool`, then a separator, then `settle the aviary`. While a newcomer is present, `make room for the <species>` also appears. It uses the naturalist voice.
- **Field notebook.** A drawer (side sheet on desktop, bottom sheet on phones) with a paper-like surface and the proper label "Field Notebook".
  - Entries are grouped under lowercase date headers and virtualized, keeping ±10 entries in the DOM with recycled nodes, which satisfies the "no retained references after scroll-out" rule.
  - The aviary keeps rendering behind it. When a panel covers more than 80% of a phone screen, the aviary drops to 30 fps rather than pausing.
  - There is no "new entry" indicator of any kind.
- **Account settings** (a matter-of-fact, lazy chunk): Email, Sessions, Birds (a rename list showing names and species only; no dates, counts, or stats), Visits (invite, outstanding invites, visit log, notification toggle), Export, Delete account, Privacy policy link.
- **Accessibility settings:** Motion (Follow system, Reduced, Full), Captions, Sound, Keep the top bar visible, Single-key shortcuts, Show narration text.
- **Naming card** (first run and newcomer), shown over the quiet field in the naturalist voice: "two birds have come to the aviary. they'll need names." It has pre-filled suggested names and one button, **let them in**.
- **Signed-out root:** the quiet field with a matter-of-fact sign-in card. Magic-link landing: "Signing you in…" in text. This page warms the asset cache.
- **No spinner component exists in the design system.** Waiting states are text ("Preparing your export. We'll email you a link.").

### 9.8 Pointer and touch interaction

- **Birds.** Hit areas are expanded to at least 44×44 CSS px and resolve to the nearest bird. Click or tap toggles listen-in. Clicking empty scene space ends listen-in.
- **Settle undo.** A click within 5 s of settle undoes it, and that click does nothing else (it doesn't also start a listen-in).
- **Listen-in feedback** is audible, not chrome. The focused bird turns its head briefly toward the viewer. Keyboard focus shows the focus ring (`:focus-visible` only); there's no highlight on mouse click.

### 9.9 Reduced-motion mode: a designed surface

This is the same aviary through a different renderer strategy on the same plan. It follows `prefers-reduced-motion` by default, with an override in settings, and switches live without a reload.

| Normal | Reduced motion |
|---|---|
| Frame-by-frame micro-motion | A **still-pose sequence**: a new pose every 4–12 s, cross-faded over 1.2–2.0 s. Poses are authored to read well as stills (preen-1…4, scan-left, scan-right, tilt, fluffed, low-perch, drink, bathe, look-at-viewer). |
| Hop or flight arcs | A dissolve out at the origin (1.2 s) that overlaps by 0.4 s a dissolve in at the destination |
| Leaf and feather drift, parallax, breathing | Removed |
| Rain streaks | A cool tint plus droplet glints on leaves that cross-fade in and out slowly. Nothing falls. |
| Wind leaf ripples | Foliage pose cross-fades |
| Light transitions | Kept, at half speed. Settle takes 6 s. |
| Top-bar fade | Kept (opacity only), slower |
| Greeting | A cross-fade to a look-at-viewer pose, with the normal call |
| Calls, captions, narration, drift, mood, notebook | Identical |

The pose library is a named art deliverable in milestone M2 (§17), not an afterthought. It gets its own review for "calm, not broken."

### 9.10 Unsupported browsers

The inline boot checks for ES2022 modules, Canvas 2D, `fetch`, and `IntersectionObserver`. If any is missing, it shows the matter-of-fact unsupported page. A missing WebAudio or AudioWorklet isn't unsupported; it means silence plus captions (§10.7).

---

## 10. Audio pipeline

### 10.1 Graph

```
AudioWorkletNode "aviary-synth"  (≤10 simultaneous voices, preallocated state)
  ├─ out[0..6] per-bird buses ─▶ Gain(listen-in) ─▶ StereoPanner(x) ─▶ distance EQ (lowpass, gain) ─┬─▶ call bus
  ├─ out[7] newcomer/offer-song bus ──────────────────────────────────────────────────────────────┤
  └─ out[8] ambient bed (procedural air/leaves, rain noise, wind swells) ─────────────────────────┤
call bus ─▶ soft-knee compressor (2:1, −18 dB) ─┬─▶ master ─▶ limiter (−3 dBFS) ─▶ destination
                                               └─▶ send ─▶ Convolver (procedurally generated IR, built once) ─▶ master
```

- There is one `AudioContext` for the page's lifetime and one worklet node. Calls are messages to the worklet: note events with audio-time stamps, a few per second. The worklet's `process()` allocates nothing. Its sorted event queue has a fixed capacity of 256, so no per-call nodes are created and the memory rule holds.
- **Spatial cues.** Pan follows x position (±0.6 at most). Back-perch birds are 4 dB quieter, low-passed at 6 kHz, and get more reverb. Front birds are dry and full. This lets the ear read perch zones.
- **Level.** The ambient mix targets about −23 LUFS short-term, which is quiet. The ambient bed sits well below the calls. Rain adds procedural droplet transients over filtered noise.

### 10.2 Synthesis

Each voice is two oscillators (carrier and modulator, for FM trills and buzzes), a noise source with a resonant band-pass (for chips, churrs, and breathiness), per-note frequency and amplitude breakpoint envelopes, and a harmonic-weight table. Bird vocalizations are well served by sinusoidal and FM models. The sound designer authors motif templates in `tools/audio-lab` and auditions them against real-bird reference *descriptions*, never against shipped recordings (INV-8).

### 10.3 Call grammar realization (client)

- **The species motif library** has 8–14 parametric motifs: whistle, slur, trill, chip, buzz, churr, coo. Each specifies ranges for duration, pitch contour (relative semitones), envelope shape, FM rate and depth, and noise mix.
- **Voice print** (immutable, drawn at adoption from the bird's seed):
  - **Base pitch** inside the species register. Same-species birds are at least 2 semitones apart.
  - **Timbre offsets:** harmonics and noise.
  - **Repertoire:** 5–8 of the species' motifs.
  - **Syntax:** a first-order Markov chain over the repertoire.
  - **A signature motif** that appears in at least 60% of calls. This is the anchor for recognizing a bird by ear.
  - **Rhythm template:** inter-motif gap ratios.
  - **Interval signature:** a characteristic step between motifs, e.g. a falling minor third.
- **Species registers** are spread to reduce masking:

  | Species | Register |
  |---|---|
  | Dove | 0.3–0.7 kHz coos |
  | Nightjar | 0.6–1.4 kHz broadband churr |
  | Thrush | 1.5–4 kHz fluty phrases |
  | Finch | 2.5–5 kHz bright two-note calls |
  | Wren | 3–7 kHz bubbling trills |
  | Warbler | 4–8 kHz thin trills |

  The species list is a working set; final design is owned by the visual and sound designers.
- **Realizing a call** from `(voicePrint, mood, ctx, intensity, seed)`:
  1. Choose the phrase length from a mood-modulated distribution.
  2. Walk the syntax chain, starting at the signature motif with probability 0.6 or more.
  3. Sample each motif's parameters within the template ranges, with jitter: pitch ±30 cents, tempo ±8%, amplitude and envelope variation, repetition count.
  4. Apply **bounded** mood transforms, keeping identity features fixed:
     - wary: shorter, sharper, slightly higher
     - content: fuller phrases
     - drowsy: lower, slower, softer
     - alert: brighter, more repetitions
     - curious: rising, question-like endings
- **Identity bounds.** Mood and drift may move pitch by at most ±1.5 semitones, tempo by at most ±15%, level by at most ±6 dB, and phrase length within 0.5–1.5×. Vocal-frequency drift changes *how often* a bird calls (on the server), never its identity. Pip stays recognizable across mood and years.
- **Contexts:**
  - `response` may echo the partner's contour.
  - `greeting` is soft and oriented toward the viewer, with the head turned.
  - `alarm` is a short, sharp universal motif for each species.
  - `chorus` uses longer phrases.
  - `offer_response/join` maps the song fragment's contour into the bird's pitch space and motif types, which is imitation in its own voice.
  - `night` is the nightjar's churr.
- **Never twice.** A per-bird ring buffer holds hashes of the last 200 quantized call parameters, and a collision triggers a resample. Continuous jitter makes collisions vanishingly rare; the guard is there to prove it.
- **The caption** is produced from the *realized* structure in the same step (§12.3), so it always matches what was played.

### 10.4 Chorus mixing

Each bird's output is summed onto its own bus, so every call stays individually shaped. The call-bus compressor keeps a seven-bird dawn chorus from clipping without flattening it. Onset-collision avoidance (§7.7) keeps ordinary calls apart. Inside chorus windows, overlap is intended, and pitch spacing between the voice prints keeps the birds separable. The server exposes no "chorus loop"; a chorus is only coincident procedural calls.

### 10.5 Listen-in mix

| Phase | Focused bird | Other birds | Curve |
|---|---|---|---|
| Engage | +4 dB, reverb send −30%, low-pass opened | −10 dB relative, floor −14 dB; **never muted** | `setTargetAtTime`, τ = 0.7 s (settles in about 2 s) |
| Hold | as engaged | as engaged | — |
| Disengage | back to ambient | back to ambient | τ = 1.0 s (about 3 s) |

- **Disengage triggers:** clicking the bird again, focusing another bird, clicking empty space, keyboard focus leaving the birds, Escape, or presence lapsing. When presence lapses, listen-in decays back to ambient on the same slow curve, which is how the listen-in mix decays. There are never hard cuts.
- **Events.** The client sends a `listen_in_start` and `listen_in_end` pair. Credit is intersected with presence on the server.

### 10.6 Autoplay, visibility, and device quirks

- **Autoplay** (D-13): the context is created at boot, and audio resumes on the first user activation with a 1.5 s fade-in. There's no prompt.
- **Hidden tab** (D-12): a 2 s fade-out, then `ctx.suspend()`. On visible: `resume()` and a fade-in. Pending plan calls are rescheduled from the current plan position.
- **iOS "interrupted" state** (phone calls, other apps) is handled like a suspended context.
- **Sound off** in settings mutes the master bus. The synth keeps generating so captions stay accurate, and an `audio_state` event records perceivability (D-5).

### 10.7 WebAudio fallback

If creating the `AudioContext` fails, `addModule` fails, or the context never leaves the "closed" state:

- The aviary runs in **graceful silence with captions turned on by default** for that device. The call grammar still realizes calls so captions stay accurate.
- The only mention is a line in accessibility settings: "Sound isn't available in this browser, so captions are on."
- There is no recorded-audio path under any condition (INV-8), and no toast. An aggregate counter tracks the failure by browser family.

---

## 11. Voice system

### 11.1 `@aviary/voice`

A typed phrase-grammar engine with three corpora: **notebook** (server), **narration** (client), and **captions** (client). A naturalist writer authors all three with a writer-facing preview tool.

- **Grammar.** Rules produce clauses with typed slots: bird name (lowercased, D-19), species noun, perch ("the front rail", "the high branch", "the back perch"), activity, time-of-day phrase, weather phrase, and call description. Clauses combine through connective rules, so sentence shapes vary.
- **Present tense** is the default, and past tense is used for completed events ("pip greeted before wren today").
- **Repetition control.** Per-consumer memory blocks reuse of the same skeleton within the last N outputs (N = 12 for the notebook, N = 6 for narration).
- **Determinism.** Output comes from `(facts, seed)`, so it's testable and reproducible.
- The corpus JSON is lazy-loaded after the first frame (narration and captions total about 40 KB gz).

### 11.2 Samples (targets for the writer, not final copy)

- **Notebook:**
  > tuesday — pip greeted before wren today, first time this week.

  > a short rain passed through in the afternoon. wren shook out wet feathers on the back perch afterward.

  > the dawn chorus was fuller than usual this morning; all three birds joined before the light came up.
- **Narration:**
  > it is early morning in the aviary; the light is still cool. pip sits on the front rail, preening, and wren calls softly from the back perch.
- **Captions:**
  > a soft three-note rise

  > a low trill, paused, low trill again

  > a single sharp call from the back perch

### 11.3 System string catalogue

A separate catalogue holds every system-surface string: sign-in, errors, settings, sessions, visits management, export, deletion, and accessibility settings. It's written in plain English with normal capitalization. The error copy is in §6.7.

### 11.4 Lints (run in CI)

- **Naturalist corpora** are sampled 100k times per template. Each output must:
  - be all lowercase
  - contain no `!`
  - have no second person (`you`, `your`)
  - include no digits (dates in headers are exempt)
  - stay within length bounds
  - avoid trait names as numbers
  - avoid "welcome", "back", "missed", "again soon", "every day", "in a row", "days", or "visit" used about the user
- **Lexicon lint** covers all product strings *and code identifiers*. It bans achievement, unlock (in user-facing text), streak, level, badge, reward, points, xp, rank, score, "great job", "welcome back", and toast, plus solo, pet, creature, and chirp in product copy.
- **System catalogue lint** requires normal sentence case, bans naturalist-register markers (lowercase-only sentences, bird verbs), and requires every error to state what happened and what to do.
- **Notebook detector lint:** the observer's input type can't reference sessions, absences, or visit counts (§7.14).
- **Copy review gate.** Every new string gets the writer's approval in code review (CODEOWNERS on the corpus and catalogue directories).

---

## 12. Accessibility surfaces

### 12.1 Structure

- **Landmarks.** A `<header>` holds the top bar (a toolbar of four buttons). `<main>` holds the aviary as `role="region"` labeled "the aviary", containing one focusable element per bird.
- **The narration live region** is visually hidden, with `aria-live="polite"` and `aria-atomic="true"`. It is never `assertive`. Only system errors use `role="alert"`, and only in the top-bar notice area.
- **Panels** are dialogs with managed focus. When a panel closes, focus returns to the element that opened it.
- **The accessibility overlay.** The canvas isn't accessible by itself, so a transparent `<button>` per bird is kept positioned over its bird with transforms updated every frame (cheap at seven elements).
  - **Accessible name:** "pip, on the front rail". It's naturalist and short, and never states mood or traits (INV-3). It's refreshed at most every 20 s, and only on a meaningful change, so screen readers don't re-announce it.
  - **State:** `aria-pressed` reflects listen-in.
  - **Roving tabindex:** exactly one bird is in the tab order.

### 12.2 Screen-reader narration

- **Composition.** The narration composer builds prose from the same snapshot, plan, and realized-call stream the renderer uses:
  - light and time of day
  - weather
  - the 2–3 most salient birds by recent action and front-zone presence (rotating, so each update doesn't list every bird)
  - notable happenings since the last update
- **Cadence:**
  - At idle, one update every 30–60 s, with random spacing.
  - **Priority events** (return-greeting, offer reaction, settle and undo, newcomer arrival, listen-in engage) are narrated within 1–2 s of happening. They're still written as observations ("pip looks up and calls twice"), never as state changes.
  - At least 8 s between any two updates, and a queue of at most 2 (idle updates that have been overtaken are dropped).
  - Idle narration pauses while a dialog is open, so it doesn't talk over the notebook or settings.
- **Listen-in narration:** "pip's call comes closer; the others soften." Disengage: "the aviary's calls even out again."
- **Show narration text** (setting, off by default): displays the same prose as a line under the top bar, at AA contrast.
- **Visitors** get the same narration.

### 12.3 Captions

- **Opt-in** from accessibility settings. They're **on by default** when WebAudio is unavailable (§10.7).
- **Generation.** Captions come from the realized call structure: note count, contour (rise, fall, flat, trill), register relative to *that bird*, loudness, internal pauses, perch zone, and context. The mapping goes through the caption corpus: "a soft three-note rise", "a low trill, paused, low trill again", "a single sharp call from the back perch". Overlapping chorus calls produce one combined caption ("pip and wren call back and forth"), which still matches what was played.
- **Placement.** Small text near the calling bird, clamped to the viewport. Captions stack instead of overlapping, and at most 4 show at once; the rest fold into a combined chorus caption.
- **Timing.** They fade in and out with the call (opacity only, so reduced motion is fine) and linger at least 1.5 s.
- **Contrast.** Each sits on a soft translucent plate whose opacity is tuned per light phase to hold 4.5:1 or better.
- **Screen readers.** Captions are `aria-hidden`, because narration covers the screen-reader experience and captions would flood the queue.

### 12.4 Keyboard

| Key | Context | Action |
|---|---|---|
| Tab / Shift+Tab | Page | Top bar items → aviary (the first bird, in the current left-to-right order) → out |
| ← → ↑ ↓ | Aviary | Move focus between birds in the current left-to-right order. Home and End go to the first and last. Moving focus ends any listen-in (D-6). |
| Enter / Space | Bird focused | Toggle listen-in |
| Escape | Listen-in active | End listen-in |
| Escape / Enter / Space | Within 5 s of settle | Undo settle (D-20) |
| Escape | Menu or dialog | Close and return focus |
| O | Not in a text field | Open the gestures (offer) menu. This is the "top-bar shortcut"; `aria-keyshortcuts` is set. |
| N | Not in a text field | Open the field notebook |
| Arrow keys, Enter | Gestures menu | WAI-ARIA menu pattern, including the song-fragment submenu |

- **Single-key shortcuts** can be turned off in settings (WCAG 2.1.4).
- **Settle** is a menu item. Its accessible description reads "Undo within 5 seconds with Escape, or by clicking the aviary." It has no single-key shortcut, to avoid accidental goodbyes.
- **Focus follows the bird.** If the focused bird flies to another perch, focus stays on it. A newcomer joins the roving order when it's welcomed.

### 12.5 Focus and contrast

- **Focus ring.** A double ring (2 px dark inner, 2 px light outer) reads at 3:1 or better against every light phase and weather state, following WCAG 2.2 focus appearance. The visual designer sets the final treatment within that constraint.
- **Text contrast.** All user-copy text passes WCAG AA (4.5:1 for body, 3:1 for large text and UI glyphs). That covers the top bar, settings, account, errors, captions, and displayed narration.
- **Automated check.** Every text and plate pair is checked across the 8 light keyframes × 3 weather states × the settled grade (§16.6).
- **Forced colors.** `@media (forced-colors: active)` styles the chrome. The canvas is unaffected, and the focus targets use system colors.
- **Target size** is at least 44×44 px for the top bar and birds.

### 12.6 Reduced motion

See §9.9. It ships in v1, and a launch gate requires testers with vestibular disorders to sign off that the mode feels designed, not broken (INV-14).

---

## 13. Accounts, auth, privacy, and data lifecycle

### 13.1 Magic-link sign-in

- **The link:** `https://<domain>/auth#t=<token>`. The 256-bit random token is stored only as a SHA-256 hash. It expires after 15 minutes and is invalidated the moment it's consumed, atomically (§5.2).
- **Consumption.** The landing page reads the fragment and POSTs it, so mail scanners that prefetch GET links can't burn it.
- **Replay** of a used or expired link shows the matter-of-fact error copy.
- **Rate limits and enumeration.** Rate limits per §6.2. The response is identical whether or not the account exists. Sign-up and sign-in are one flow; the first verify creates the account.
- **Email.** It's normalized (lowercase, trimmed; no provider-specific dot rules) before the HMAC lookup.

### 13.2 Sessions

- **Token.** An opaque 256-bit token, stored as a hash, in a `__Host-session` cookie.
- **Lifetime.** Sliding 60-day idle expiry, 1-year absolute. Long enough that "leave for two weeks and come back" never ends in a sign-in wall.
- **Session list** in settings: device label, created date, last-active day. Any session can be revoked. Regional APIs validate sessions against the replica, with a 30-second cache; revocation reaches the primary immediately and replicas within seconds.

### 13.3 Email change

A verification link goes to the new address (24 hours, single use, D-21). The old address keeps working until that link is consumed. After the switch, the old address gets a matter-of-fact notice ("The email for your Pocket Aviary account was changed.").

### 13.4 Synthetic IDs and PII containment

- **Encryption.** The email exists only as ciphertext in `acct.accounts` (and in pending `email_change_requests`), under envelope encryption with a KMS-managed key and AES-256-GCM. Lookup goes through the keyed HMAC on the same row.
- **Every other reference is the account UUID:** every other table, queue payload, log line, metric, partition key, and error report. Queue payloads for mail jobs carry the account UUID, and the mail job decrypts at send time.
- **Log scrubber.** The logger blocks fields named `email*`, redacts values matching email patterns, and the CI PII scanner fails any integration run whose logs contain one.
- **Visitor email** gets the same treatment in `social.invites`.

### 13.5 Export

1. The job assembles the export JSON (`schema_version`, `exported_at`), containing:
   - the account's creation date and settings
   - the aviary timezone
   - `birds[]`: id, name, species, adopted date, `personality` (D-2), current mood
   - all notebook entries (date and text)
   - invites and the visit log
2. The file **excludes** interaction events and presence history. An exportable visit log is one of the PRD's named streak disguises, and the event log isn't part of the aviary's state.
3. The job stores the file in S3 (SSE-KMS) and emails a download link, valid 72 hours, that also requires a signed-in session.
4. The product never opens, previews, or imports the file.

### 13.6 Deletion

- **Soft.** Deletion sets `status = pending_deletion` and `hard_delete_after = now + 30 d`, immediately revokes all visitor passes and outstanding invites, and sends a matter-of-fact confirmation email.
- **During the window** the user can sign in, and every signed-in page shows the pending-deletion notice with **I changed my mind**, which restores the account. The tick keeps running, so a restored aviary has simply carried on.
- **Hard.** A lifecycle job deletes everything in one ordered cascade: `sim` rows (events, personality, mood, queue, snapshots, notebook), `social` rows, `acct` rows, S3 objects, Redis keys, and edge-cached fragments.
- **Verification.** A verifier job asserts that zero rows reference the UUID. No tombstone is kept. The job's own logs age out in 14 days, and backups in 35.
- **Aggregate metrics** carry no account dimension, so there's nothing to delete there by construction.

### 13.7 Privacy boundary at the pipeline level

- **Nothing leaves.** Per-bird interaction events feed only that account's tick. They aren't exported to any warehouse, model, recommender, or third party.
- **No route in.** The telemetry stack runs in a separate cloud project and account, with separate credentials. Network policy blocks it from the database subnet. No ETL job holds `sim` credentials.
- **The page loads no third-party scripts.** The CSP is `script-src 'self' 'nonce-…'`: no analytics, ads, or session-replay SDKs, and no tag manager.
- **Error reports** (self-hosted) contain the stack, build, browser family, and route. They have no breadcrumbs of user actions, no state, and no URLs with tokens or fragments.
- **Shadow-mode diffs** (§7.15) produce only aggregate histograms, computed in memory.
- **Privacy policy.** It's linked in account settings, in plain text. It names the aggregate categories and explicitly excludes per-bird interaction state.

### 13.8 Internal tooling

- **Support tooling** shows account status, the session list, recent error codes, and deletion state. It never shows personality values, mood history, notebook text, or events.
- **Break-glass access** to `sim` data for incident debugging needs two-person approval and the user's consent. It is audited and time-boxed.

### 13.9 Outbound email allowlist

1. Sign-in link
2. Email-change verification
3. Email-changed notice (to the old address)
4. Export ready
5. Deletion scheduled
6. Deletion cancelled
7. Visit invitation (to the visitor)
8. Visit notice (to the host, only when opted in; at most 1 per invite per day)

All of them are matter-of-fact, with no open or click tracking. **No template exists for re-engagement, digests, "your bird misses you", or anything about the aviary's state.**

---

## 14. Performance budgets and observability

### 14.1 Budgets

| Budget | Target | Hard limit | Enforcement |
|---|---|---|---|
| Initial JS at first paint (gz) | Boot ≤ 45 KB. Everything loaded in the first 2 s ≤ 450 KB. | **2 MB** (PRD) | `size-limit` in CI fails the PR |
| HTML plus inline snapshot (gz) | ≤ 14 KB, one initial congestion window | 32 KB | Contract test |
| Snapshot | ≤ 8 KB raw with 7 birds | 32 KB | Contract test |
| Time to first bird visible (mid-tier phone, 4G) | p75 ≤ 450 ms warm, ≤ 500 ms cold | **500 ms** (PRD) | Synthetic fleet, plus RUM `first-bird` mark |
| Idle frame rate (5-year-old laptop) | 60 fps: p95 frame ≤ 16.7 ms, under 1% dropped over **30 minutes** with 7 birds, rain, and chorus | 60 fps | Perf lab, nightly |
| Main-thread work per frame | ≤ 6 ms p95 on the reference laptop | 10 ms | Perf lab |
| Memory over 30 minutes | JS heap slope ≤ 0.5 MB per 30 min after a 2-min warm-up. DOM node, listener, and AudioNode counts constant. Worklet queue bounded. | **No growth** (PRD) | CI (§16.4) |
| Audio CPU | ≤ 10% of one core for a 7-bird chorus | 20% | Perf lab |
| Tick per-aviary compute plus commit | p99 ≤ 1 s | **Alarm at p99 > 5 s** (PRD) | Prod metrics |
| Tick lag (scheduled vs. applied) | p99 ≤ 75 s | Alarm at > 120 s | Prod metrics |
| API | Regional snapshot p95 ≤ 120 ms. Events p95 ≤ 150 ms. Offer round trip p95 ≤ 250 ms. | — | Prod metrics |
| Magic-link delivery | p95 ≤ 30 s | — | Aggregate provider webhook metrics |

- **Reference devices:**
  - Mid-tier Android (Moto G Power / Galaxy A-class, or Chrome with 4× CPU throttling), plus an iPhone SE-class device for Safari.
  - A five-year-old laptop at launch: 2021 mid-range Intel (i5-1135G7, Iris Xe). A stricter floor on a ThinkPad T490 (i5-8265U, UHD 620).
- **Network profile:** 4G at 9 Mbps down, 1.5 Mbps up, 170 ms RTT.

### 14.2 How the budgets are met

- **Code-splitting:** boot → scene core → audio, voice, and outbox → chrome → lazy panels (notebook, settings, visits, export).
- **Procedural bird visuals.** Compact rigs and no bitmaps beyond small baked atlases.
- **The edge inlines the snapshot** so there's no extra round trip. The service worker keeps immutable assets warm.
- **Offscreen caching** for slow-changing layers. No per-frame allocation. Pooled particles and caption nodes.
- **One AudioContext and one worklet.** Pooled voices.
- **A bounded outbox,** and notebook virtualization.

### 14.3 Synthetic performance checks

- **Scope.** A fleet of automated browsers (WebPageTest-class) runs the aviary every 15 minutes from 6 geographies: us-east, us-west, eu-west, south america, india, ap-southeast. It covers the reference phone and network profile, warm and cold cache.
- **Test accounts.** Dedicated synthetic accounts that belong to the test system and carry no real user data.
- **Metrics:** first-bird, HTML TTFB, whether the snapshot was inlined, quiet-field duration, and 60-second frame stats.

### 14.4 RUM (aggregate-only by construction)

- **Client-side aggregation.** The client folds measurements into fixed histogram buckets and flushes counts with `sendBeacon` to the telemetry host, without cookies or identifiers of any kind: no account, no session, no device ID, no page-view ID.
- **Allowed dimensions** are coarse only: `device_class (mobile|desktop)`, `browser_family`, `edge_region (continent)`, `build`.
- **The collector enforces** the schema allowlist and drops anything else. A CI lint checks every metric definition.

### 14.5 What we measure

- **Load:** first-bird-visible histogram, HTML TTFB, whether the snapshot arrived inline, quiet-field shown and for how long, cold vs. warm cache.
- **Runtime:** frame-time histograms (10-second windows), long-task counts, adaptive-scale step-downs, JS heap samples (Chrome only, coarse buckets).
- **Audio:** context-creation failures, worklet load failures, the share of sessions waiting on a gesture, underrun counts, all by browser family.
- **API:** rate, latency, and errors by route and status. Snapshot size histogram. Event-ingest volume and rejections by reason.
- **Tick:** per-aviary latency p50/p95/p99 (**alarm at p99 > 5 s**), tick lag, batch sizes, CAS conflicts, lease churn, age of the oldest unconsumed event, `sim_monotonic_block_total` (must be 0), personality-row integrity (the count of birds with no personality row; must be 0; checked daily).
- **Notebook:** entries generated (global count), grammar fallback count (should be about 0).
- **Email:** send latency, bounces, expired-link error counts.
- **Sessions:** anonymized session-duration histogram, bucketed on the client with no identifier.

### 14.6 What we deliberately don't measure

- **Anything with an account, bird, or session dimension:** DAU or retention cohorts by account, visit frequency, streaks, return intervals, interaction counts per account.
- **Engine behavior:** personality, mood, or attunement distributions; production drift rates; offer acceptance rates; listen-in durations; notebook open rates.
- **Accessibility-preference adoption.** Disability-adjacent data is sensitive, and we don't need it to ship the features properly.
- **Engagement analytics:** funnels, heatmaps, session replay, affect A/B tests (which would need per-account assignment and analysis).
- **Visitor behavior** beyond the host's own visit log.
- **Any cross-account aggregate over the `sim` schema.** No such pipeline exists, which also makes a leaderboard costly to add later.

**Calibration without production data.** Calibration uses the synthetic harness and lab perception studies (§16.2). Its behavior profiles are grounded in design research and the permitted anonymized session-duration histogram, never in per-account telemetry. This is a deliberate blind spot, and R-2 covers how it's managed.

---

## 15. Security (summary)

- **Headers:** strict CSP with nonces (no third-party origins), HSTS preload, `frame-ancestors 'none'`, COOP `same-origin`.
- **Tokens.** All tokens are 256-bit random and stored as hashes. Tokens for links live in URL fragments. Cookie prefixes are `__Host-`.
- **Rate limits** on auth, invites, events (at most 20 req/min per session), and offers (the cooldown plus at most 30 per hour per account).
- **Visitor passes** can only read the host snapshot. They can't read the notebook, settings, events, or anything else.
- **Dependencies.** Audited with a lockfile. The client runtime keeps dependencies minimal: Preact, zod (tree-shaken), idb-keyval.
- **Secrets and keys** live in KMS, and the HMAC pepper and encryption keys rotate yearly, dual-read during rotation.
- **Security review** is a launch gate: auth flows, visit flows, CSRF, and deletion completeness.

---

## 16. Verification strategy

### 16.1 Simulation core

- **Property tests** (fast-check):
  - Δ ≥ 0 for every input (INV-2).
  - Values stay in [0, cᵢ].
  - Zero input gives zero drift once the filter decays below ε.
  - dt-invariance: trajectories at 30, 60, and 120 s ticks and a 3-hour catch-up agree within 1%.
  - Presence credit is the union across devices.
  - Visitor events are rejected.
  - Absence never enters the mood logits (a static check on the logit builder's inputs).
- **Replay tests:** recorded synthetic event streams (golden files) reproduce the exact state for a given `sim_version`.
- **Chaos tests:** kill workers mid-transaction, let a lease expire mid-batch while a zombie commits (the fencing must reject it), deliver duplicate events, deliver out of order, commit a late lower sequence, and let a replica lag 30 s. Assertions: exactly-once application and a monotonic snapshot `tick`.

### 16.2 Drift calibration

- **The harness** (`tools/calibration`) runs every profile in §7.4 over 120 simulated days across 1,000 seeded birds per profile. It checks the day-7 and day-21 targets and the guards, and produces a report that `sim-server` changes must include.
- **Perception study.** Run it before alpha, then again before beta. About 30 participants view rendered aviary clips at controlled trait deltas (2-alternative forced choice) to set the per-trait JNDs. A second arm has participants watch a simulated "week 1 vs. week 3" of the same aviary and rate "has anything changed?". The targets: week 1 near chance, week 3 detected at 75% or more.

### 16.3 Sync and API

- **Contract tests:** zod and JSON Schema, with snapshots validated against the schema. A no-trait-field check.
- **Two-browser e2e** (Playwright): laptop and phone contexts do overlapping sessions, offers, renames, settles, and revocations, and every scenario in §8.3 is asserted.
- **Offline e2e:** a network cut for 10 minutes, then reconnect. No teleport (checked by a position-continuity assertion on bird bounding boxes), events delivered once.

### 16.4 Client performance and memory

- **Memory CI.**
  - Nightly: a real-time 30-minute run on Chrome (CDP heap snapshots and `performance.measureUserAgentSpecificMemory`) and Firefox, with scripted listen-ins, offers, 500-entry notebook scrolls, visibility toggles, and weather.
  - Per PR: a 3-minute run with time accelerated 10×.
  - Assertions: heap slope, DOM, listener, and AudioNode counts constant, worklet queue bounded.
- **Perf lab:** real reference laptops and phones in a lab, 30-minute 7-bird stress runs, frame and audio CPU budgets.
- **Bundle:** `size-limit` per chunk. The asset scanner rejects audio files (INV-8).

### 16.5 Audio and greeting quality

- **Distinctness metric.** Per-call MFCC and contour embeddings. Inter-bird distance must exceed intra-bird distance (across moods and drift levels) by a margin, for every seeded 7-bird fixture aviary. This is also the rejection-sampling criterion at adoption.
- **Listening panels** (`tools/audio-lab`), run at alpha, beta, and every cap ramp step:
  - **Recognizability:** after 10 minutes of familiarization, identify the bird from new calls across moods. Target at least 80% top-1 accuracy with 5 birds, and at least 80% with 7 before the cap is raised (§18.3).
  - **Naturalness MOS:** at least 3.5 out of 5, and a "sounds like a toy or beep" rating of 15% or less.
  - **30-minute repetition test:** "did you hear the same call twice?" must come back at chance.
- **Uniqueness tests.** 10k calls per bird and 10k greetings per fixture must have unique parameter hashes with minimum perceptual distances.

### 16.6 Accessibility

- **Automated:**
  - axe-core on all chrome surfaces.
  - Narration cadence: over a simulated 30 minutes, idle updates are 30–60 s apart and never less than 8 s.
  - Narration and caption corpus lints.
  - Focus-order tests.
  - The contrast matrix (§12.5).
  - Reduced motion: no transform animation on birds in RM mode (instrumented renderer assertion), and no particles.
- **Manual matrix:** VoiceOver (macOS Safari, iOS Safari), NVDA with Firefox and Chrome, JAWS with Chrome and Edge, TalkBack with Chrome.
- **Paid research sessions** in alpha and beta with blind and low-vision users, deaf and hard-of-hearing users (captions), people with vestibular disorders (reduced motion), and keyboard-only and switch users. **Launch gate:** each group signs off that the experience "feels like the aviary", not a fallback.

### 16.7 Principle guards (CI)

- A no-Toast/Badge/Spinner component lint.
- Push and Notification API bans.
- The outbound email allowlist test.
- The lexicon, voice, and system-catalogue lints.
- The telemetry label lint.
- The PII log scanner.
- DB grants and monotonic trigger tests.
- The seven-bird trigger test.
- The unlock-module import lint.
- A check that visitor mode never loads the host-only modules.
- The first-frame e2e (INV-13).
- A check that `sim-server` is never bundled into `apps/web`.

---

## 17. Delivery plan

### 17.1 Team (about 13)

- **Sim and backend:** 3 engineers (tick, drift and mood, resolvers, API, sync).
- **Client rendering:** 2 engineers, plus 1 technical animator (rigs, poses, micro-motion, reduced-motion pose library).
- **Audio:** 1 DSP engineer, plus 1 sound designer (motif libraries, voice prints).
- **Accessibility lead:** 1 engineer (narration, captions, keyboard, testing program).
- **Naturalist writer:** 1 (voice corpora, system catalogue review).
- **Infra, SRE, and security:** 1.5.
- **Visual designer:** 1 (species, palette, design system, focus treatment).
- **QA and test automation:** 1.
- **Product and design lead:** 1.

### 17.2 Milestones

| Milestone | Weeks | Scope | Exit criteria |
|---|---|---|---|
| **M0 Foundations** | 1–4 | Monorepo, contracts, CI with all guard lints from day one, Postgres schemas and roles and grants, auth skeleton, tick scheduler with leases and fencing, the calibration harness running against the first drift draft | Grants and monotonic trigger tests green. Harness runs every profile. |
| **M1 Living slice** | 3–10 | Edge boot with inline snapshot, canvas scene with 2 placeholder rigs, choreographer and action queue, micro-motion v1, AudioWorklet synth with one species, presence tracker and events, greeting v1, keepalive and reconciliation | First bird ≤ 500 ms in synthetic. A two-browser e2e shows identical plans. Tab-open-all-night gives zero drift. No teleports. |
| **M2 Engine and audio depth** | 8–16 | Full drift, attunement, mood, bird-to-bird, weather. Six species (rigs, poses, the reduced-motion pose library, motif libraries). Voice prints with distinctness. Chorus mixing and listen-in mix. Offer resolver and offer visuals. | Calibration targets pass. First audio panel (recognizability at 5 birds ≥ 80%, MOS ≥ 3.5). Perception study sets the JNDs. |
| **M3 Voice and accessibility surfaces** | 12–20 | `@aviary/voice` with the three corpora, the notebook observer and governor, narration, captions, the keyboard model, the reduced-motion renderer, accessibility settings, contrast matrix | Corpus lints pass. First accessibility research round. Reduced motion reviewed by vestibular testers. |
| **M4 Accounts, visits, lifecycle** | 16–24 | Sessions UI, email change, export, deletion (soft, hard, verifier), visits end to end, visit log, notification opt-in, system catalogue, error surfaces | Security review. Deletion verifier green. The visitor-can't-write test passes. |
| **M5 Hardening** | 20–26 | Perf lab passes, memory CI green for 2 consecutive weeks, chaos suite, observability dashboards and alarms, shadow-mode pipeline, feature flags and kill switches, runbooks | All budgets in §14.1 met on the reference devices. |
| **Alpha (internal)** | 14–26 | Staff aviaries run continuously from M1+ (§18.1) | Staff aviaries are at least 10 weeks old going into beta. Qualitative drift-feel reviews. |
| **Beta (closed)** | 26–34 | §18.1 | Beta exit criteria (§18.1) |
| **GA** | ~35 | §18.2 | Launch checklist (§18.2) |

---

## 18. Rollout

### 18.1 Stages

1. **Internal alpha** (from about week 14).
   - Staff sign up as normal users. Their aviaries get no special analytics. We learn from staff *perceiving* their own birds over weeks, which tests the "visible at about three weeks" promise in real life, alongside the harness.
   - Accessibility and listening research rounds run here.
2. **Closed beta** (weeks 26–34, 2k growing to 10k accounts).
   - Sign-up is limited by an allowlist of email HMACs.
   - Beta runs at least 8 weeks, so participants live through the one-week and three-week horizons.
   - Feedback comes only through channels the user starts: the "get in touch" link, plus an opt-in research panel with separate consent for interviews. There are **no in-product surveys or feedback emails**; those would be the product reaching for the user.
   - **Beta exit criteria:**
     - tick SLOs met for 4 weeks
     - zero monotonic blocks and zero personality integrity failures
     - no Sev-1 sync incidents
     - first-bird p75 under 500 ms from 5 of 6 geographies
     - memory CI green
     - accessibility sign-offs
     - qualitative drift feel "noticeable when looking back, never session-to-session" in at least 70% of panel interviews
3. **GA.** Open sign-up, with capacity headroom of 5× the peak beta load. **Launch checklist:**
   - all invariant tests green
   - launch gates passed
   - privacy policy published
   - backup restore drill done (restore the personality table to a staging clone and verify checksums)
   - on-call runbooks ready: tick lag, replica lag, email deliverability, deletion failures

### 18.2 Deployment mechanics

- **API and edge:** blue/green, with a 5% canary for 1 hour and automatic rollback on error-rate or latency alarms.
- **Tick workers:** new `sim_version`s go through shadow mode (§7.15), then partition-by-partition promotion (5%, 25%, 100% over 72 hours). Rolling back the code never rolls back state: drift already applied stays applied (INV-2), and later ticks just use the old code.
- **Client:** versioned assets. When a snapshot schema is incompatible, the client reloads quietly on the next visibility change, never mid-session, and never with a "new version" toast.
- **Species revs:** additive only (§7.15).

### 18.3 Ramping birds per aviary

Every aviary starts with two birds from launch. The *upper* cap is ramped by a server config, `max_birds_enabled`, which can't exceed the engine constant of 7:

| Step | Cap | Gate |
|---|---|---|
| GA | 3 | 3-bird perf lab and listening panel passed. The earliest third-bird unlocks for beta aviaries fall about 90 days after beta starts, so around GA. |
| GA + ~2 months | 5 | Recognizability of 80% or more at 5 birds in the real mix, and 5-bird perf lab on the reference devices |
| GA + ~4 months | 7 | Recognizability of 80% or more at 7 birds, 7-bird chorus perf lab, and layout verified at 7 on 320 px viewports |

- **Timing.** The age-based schedule means no real aviary gets its 4th bird before about day 160, so these gates run well ahead of demand.
- **Staging.** 7-bird aviaries are exercised with synthetic, backdated aviaries built by admin tooling that only exists in staging. Production aviaries are never aged artificially, because that would break the age rule.
- **Lowering the cap** only stops new unlocks. It never removes a bird (INV-4).

### 18.4 Feature flags and kill switches

- **`visits_enabled`:** turning it off returns `visit_unavailable` to every visitor and hides the Visits settings.
- **`newcomers_enabled`** and **`weather_enabled`.**
- **`notebook_observer_enabled`:** when off, facts still accumulate in the 14-day window but no entries are written.
- **`drift_rate_multiplier`,** in [0, 1]. It can only slow drift, never reverse it.
- **Drift freeze.** The tick keeps consuming events and running mood, but trait Δs accumulate in `held_delta` instead of being applied. After a fix, held deltas are applied, optionally scaled down (never below 0). This is the safety valve for a calibration bug, so no event is lost and no trait moves backwards.

### 18.5 Instrumented from day one

Everything in §14.5 is live in alpha, before any real user: tick latency and lag alarms, the monotonic-block and personality-integrity alarms, first-bird RUM and synthetic checks, audio failure counters, email delivery metrics, and the log PII scanner (run in production daily over samples, alerting on hits). Dashboards and alarms are an M5 exit criterion, and alpha begins on the M1 subset.

---

## 19. Risks

| # | Risk | Likelihood / impact | Mitigation | Early signal |
|---|---|---|---|---|
| **R-1** | **Drift too fast.** Birds change session to session: a Tamagotchi feel. | M / H | Daily saturation, a 4-day EMA, the per-day cap, and single-session and heavy-profile guards in the harness. `drift_rate_multiplier` and drift freeze. Shadow mode before any change. | Beta interviews mention "it changed since yesterday". A perception-study arm. |
| **R-2** | **Drift too slow or invisible,** which reads as a screensaver. Worsened by our refusal to look at production drift data. | M / H | JNDs set by perception studies. Staff aviaries lived with for 10+ weeks. Visible mappings chosen for legibility (perch zone, plumage chroma, greet-first). Notebook entries surface the plumage and front-perch "firsts" naturally. | Alpha and beta interviews: "nothing I do seems to matter". |
| **R-3** | **Long-term convergence.** After a year every bird is at maximum and they all look alike. | M / M | Per-bird ceilings, (c − v)^γ slowing, mappings relative to species, and immutable voice and look individuality. The harness runs over 2 simulated years. | The 2-year harness report shows inter-bird behavior variance collapsing. |
| **R-4** | **Presence over-counting** through a laxer implementation, or a browser quirk (e.g. `hasFocus` true in a background window on some OS). | M / H | Strict three-signal conjunction, a union across devices, a cap on elapsed time, and per-browser e2e tests of every signal (minimized windows, background windows, locked screens, OS sleep). | Aggregate presence-minutes-per-session histogram shifts after a browser release. |
| **R-5** | **Presence under-counting on phones,** because pure taps don't count (D-7). | M / M | W at the long end (4 min). Revisit with product in beta: adding `pointerdown` is a one-line change, reviewed against the harness. | Beta phone users report their birds "don't change". |
| **R-6** | **Sync correctness:** double-applied or lost events, zombie ticks, skipped sequences. | L / H | Row-locked sequence allocation, cursor in the same transaction, fencing tokens, and the chaos suite. | Growing age of the oldest unconsumed event. CAS conflict spikes. |
| **R-7** | **Personality loss** through a bad migration, restore, or species change. The worst possible failure, and invisible. | L / Catastrophic | INV-4 controls, migration checksums, a daily integrity job, PITR, restore drills, and species-rev pinning. No code path recomputes vectors from logs. | Integrity alarm. A migration checksum mismatch blocks the deploy. |
| **R-8** | **Audio uncanniness:** beepy, toy-like, repetitive, a muddy chorus. | H / H | A dedicated sound designer, sinusoidal and FM synthesis with noise components, bounded jitter, listening panels with MOS and repetition gates, and the compressor, spatial cues, and register spacing. | MOS under 3.5, or the "toy" rating over 15%, in any panel. |
| **R-9** | **Recognizability collapse** at 5–7 birds, or after mood and drift variation. | M / H | Signature motifs, identity bounds, rejection-sampled voice prints, and the cap ramp gated on panels (§18.3). | Panel accuracy under 80%. The distinctness metric shrinking. |
| **R-10** | **Autoplay policy** means calls aren't audible at first frame on first visits. | H / M | D-13: silent-but-alive visuals, audio on first activation with a fade-in, and no prompt. Engagement-based autoplay covers many return visits. Tracked in aggregate. | A high "awaiting gesture" ratio in Safari. |
| **R-11** | **Accessibility regression:** new features add motion in reduced-motion mode, narration floods, focus gets lost on moving birds, or contrast breaks in the night palette. | M / H | Automated reduced-motion assertion, narration cadence test, contrast matrix, focus tests, an accessibility lead with veto on the release checklist, and research rounds each stage. | CI failures. Research-session findings. |
| **R-12** | **The narration reads like a state list** because engineers write templates. | M / H | The writer owns the corpora (CODEOWNERS), plus lints and SR user sessions. | Reviewer or tester feedback like "it sounds like a log". |
| **R-13** | **Announcement creep:** a "harmless" toast, badge, or re-engagement email. | H / H | Missing components, API lint bans, the email allowlist, and a "refusal register" section in the PR template that asks "does this announce, count, or rank?" | A PR adding UI state near arrival or visits. |
| **R-14** | **Notebook voice fatigue:** repetitive templates, or entries that are too frequent. | M / M | The governor, novelty penalties, skeleton memory, and a large authored corpus (≥ 400 clause variants at launch). | The writer's review of generated 90-day notebooks for fixture aviaries. |
| **R-15** | **Time to first bird misses 500 ms** in far regions or on cold caches. | M / H | Edge inlining, regional replicas, a 250 ms edge fallback, the service worker, and a small boot chunk. | Synthetic p75 per geography. |
| **R-16** | **Memory leaks** over long sessions (audio, captions, notebook). | M / M | The worklet design, pools, virtualization, and nightly 30-minute CI. | A rising heap-slope trend in CI. |
| **R-17** | **PII leak** into logs, metrics, or error reports. | M / H | Scrubbers, the label allowlist, the CI PII scanner, the production sample scanner, and no third-party SDKs. | Scanner hits. |
| **R-18** | **Magic-link deliverability:** spam foldering, scanners burning links, slow delivery. | M / H | A reputable provider with SPF, DKIM, and DMARC; tracking off; fragment tokens consumed by POST; a fast resend. | Aggregate expired-link and consumption-latency metrics. |
| **R-19** | **Tick cost at scale.** Evaluating every aviary every minute is expensive. | L / M | Write-on-change, closed-form lazy decay, batching, partition sharding, and dt-invariance so cadence can be tuned without changing behavior. | Tick CPU per 1k aviaries. |
| **R-20** | **Visitor privacy or abuse:** invite spam, forwarded links, lingering access. | L / M | Rate limits, single-use redemption bound to one browser, 14-day passes, immediate revocation, and a visible visit log. | Invite bounce rate. |
| **R-21** | **The newcomer flow reads as a reward** or a nag. | L / M | Unlocks depend on age only, the newcomer appears in the scene, declining costs nothing, and there's no copy about "unlocking". | Beta interview language ("I earned a bird"). |

---

## 20. Open questions (non-blocking; defaults above apply until changed)

1. **Presence on touch (D-7).** Should `pointerdown` count as pointer activity on phones? Default: no. Decide from beta interview data and a harness run.
2. **Visitor access to the notebook (D-10).** Default: no. The PRD's "no notebook scrolling that affects the host" could be read as allowing read-only access.
3. **Visit-log retention.** Default: kept while the invite exists plus 180 days. It needs a privacy review.
4. **Background listening** (the tab hidden, audio on) as a later, opt-in accessibility-adjacent feature.
5. **Releasing a bird (D-26).** Whether a user who genuinely wants fewer birds should ever be able to, and how that would square with permanent identity.
6. **Seasonal light.** A latitude-free seasonal curve keyed to the timezone's hemisphere, for v1.1.
7. **Business metrics.** Leadership should explicitly accept that v1 reports only total account count and aggregate load, with no DAU or retention cohorts, as a direct result of the privacy boundary.
8. **Species working set.** Confirm the six species (finch, warbler, wren, thrush, dove, nightjar-like) with the visual and sound designers by the end of M1.
