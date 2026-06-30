# REPORT — CARE run 001

> Phase 2B scoring report for the frozen run 001 reconstruction. Exact evidence quotes are short and come only from the assigned PLAN and RECONSTRUCTION.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **90.0%** |
| Intent fidelity | **54.2%** |
| Combined quality | **8954** |

**Diagnostic split:**

- System-level fidelity: **71.4%**
- Feature-level fidelity: **50.0%**

**(Planning, fidelity) coordinate:** `(90.0, 54.2)` — plot on a 2D scatter with both axes 0-100; upper-right is best.


### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-06-30T21:24:01Z |
| Candidate model | claude-5-sonnet |
| Candidate effort | medium |
| Candidate harness | claude-code |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **108 / 120** = **90.0%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 14 | 70.0% |
| aviary_layout.md | 18 | 14 | 77.8% |
| accounts_sync.md | 18 | 16 | 88.9% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **108** | **90.0%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Overall app concept is operationalized across scope, architecture, and surfaces. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Server tick, mid-motion first frame, procedural calls, quiet field, and slow pacing preserve the principle. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured through non-goal constraints, no gamification surfaces, and quiet social defaults, though the named wording is absent. |
| 4 | Voice-and-tone guide for product surface | product_brief.md | yes | Naturalist notebook/narration/captions and matter-of-fact system surfaces are planned. |
| 5 | "What this is not" callout | product_brief.md | yes | Non-goals are explicit and architecturally enforced. |
| 6 | Restraint-over-richness scope statement | product_brief.md | yes | Two starters, cap of seven, one screen, and no customizable scenes are covered. |
| 7 | Glossary of domain terms | concepts.md | yes | borderline: No standalone glossary, but the domain terms are operationalized in data/API/engine sections. |
| 8 | Definition of "presence" | concepts.md | yes | borderline: Presence is treated as attention and a drift input, but the exact three-signal conjunction is missing. |
| 9 | Definition of personality vector vs mood | concepts.md | yes | Slow personality drift and faster mood state are distinct in data model and tick logic. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Settle is an event/state with undo and presence-window closure. |
| 11 | Personality vector traits | bird_engine.md | yes | The five traits are modeled as server-only fields. |
| 12 | Personality drift function | bird_engine.md | yes | Low-pass filter, calibration, and signal inputs are detailed. |
| 13 | Drift rate calibration | bird_engine.md | yes | One-week measurable and three-week visible calibration targets are explicit. |
| 14 | Personality drift is monotonic toward expressive | bird_engine.md | yes | Non-negative deltas and no subtraction path are enforced. |
| 15 | Mood state | bird_engine.md | yes | Five-state mood enum and daily-ish reset are planned. |
| 16 | Mood inputs | bird_engine.md | yes | Interactions, time of day, ambient events, and personality modulation are modeled. |
| 17 | Procedural call grammar | bird_engine.md | yes | Motif grammar and WebAudio synthesis are planned. |
| 18 | Per-bird call signature | bird_engine.md | yes | borderline: Call grammar parameters and personality-derived hints imply per-bird sound, but recognizability is not foregrounded. |
| 19 | Chorus mixing | bird_engine.md | yes | Independent node graphs and mix bus are planned. |
| 20 | Call timing shaped by personality | bird_engine.md | yes | Derived call interval/eagerness hints preserve this without raw trait exposure. |
| 21 | Idle micro-motion | bird_engine.md | yes | Mood-weighted idle motion state machines are planned. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | The plan explicitly says the user reads mood from motion without a label. |
| 23 | Bird species pool for v1 | bird_engine.md | yes | A roughly six-species pool is included. |
| 24 | Bird naming | bird_engine.md | yes | Names are user-assigned and renameable with no simulation effect. |
| 25 | Adoption flow | bird_engine.md | yes | Two starter birds at adoption are fixed at launch. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | The cap is included. |
| 27 | Adding a third+ bird | bird_engine.md | yes | Age-gated growth and hot-tunable thresholds are planned. |
| 28 | Personality vector persistence | bird_engine.md | yes | Vectors are canonical server-side state. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Stored mood and no session-start initialization path are planned. |
| 30 | Bird-to-bird interaction | bird_engine.md | yes | Call-response, wary spread, and chorus eligibility are covered. |
| 31 | Bird identity stability | bird_engine.md | yes | Stable bird IDs are permanent and never regenerated. |
| 32 | Personality vector exposure | bird_engine.md | yes | Raw personality values are never serialized or exposed. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Return-greeting is in scope and in narration priority events. |
| 34 | Greeting variation by absence length | interactions.md | no | The plan does not specify absence-length variation. |
| 35 | Greeting variation by bird boldness | interactions.md | no | The plan does not specify boldness-based greeting order. |
| 36 | Greeting stagger | interactions.md | no | No stagger rule appears. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No textual welcome prohibition appears. |
| 38 | Listen-in interaction | interactions.md | yes | Listen-in events and focused audio gain are planned. |
| 39 | Listen-in mix decay | interactions.md | yes | Other birds ramp to a quieter nonzero floor. |
| 40 | Offer interaction | interactions.md | yes | Seed/song/pool offers are event types and UI affordances. |
| 41 | Offer reaction varies by mood and curiosity | interactions.md | yes | Offer acceptance biases mood and curiosity. |
| 42 | Offer cooldown | interactions.md | yes | Per-bird offer cooldown fields are modeled. |
| 43 | Settle gesture | interactions.md | yes | borderline: Settle is modeled as a session-end event, though the evening lighting shift is not specified. |
| 44 | Settle is opt-in | interactions.md | no | The plan lacks explicit close-tab equivalence or optionality. |
| 45 | Field notebook auto-entries | interactions.md | yes | Sparse naturalist notebook generation is planned. |
| 46 | Field notebook entry frequency | interactions.md | yes | Cooldown/rarity budgets target every-few-days entries. |
| 47 | Field notebook is read-only | interactions.md | yes | Read-only notebook is stated in scope. |
| 48 | Presence accounting | interactions.md | yes | borderline: Presence pings and presence-time are planned, but precision is weak. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | no | The three-signal conjunction is absent. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | No streaks/counters/calendars and no backing fields are explicit. |
| 51 | Background-tab pause | interactions.md | yes | Visibility re-sync and server-side continuation cover the behavior. |
| 52 | Click-anywhere-to-undo settle | interactions.md | yes | A 5-second undo window is modeled. |
| 53 | Single horizontal scene | aviary_layout.md | yes | The scene is single horizontal. |
| 54 | Three perch zones | aviary_layout.md | yes | Front/middle/back perch zones are explicit. |
| 55 | Bird-chosen perch | aviary_layout.md | yes | Perch is derived from personality/mood, not user placement. |
| 56 | Day/night cycle tied to local time | aviary_layout.md | yes | Server caches IANA timezone for local phase. |
| 57 | Evening palette shift | aviary_layout.md | yes | borderline: Day/night color tint is captured; warmer hue and quieter-calls detail is thin. |
| 58 | Night state | aviary_layout.md | no | No nightjar-like active-bird state is planned. |
| 59 | Ambient weather | aviary_layout.md | yes | Rare rain/wind and weather state are modeled. |
| 60 | Weather affects mood | aviary_layout.md | yes | Rain and wind bias mood/vocal frequency. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Client-side cosmetic leaf/feather drift is planned. |
| 62 | Foreground/background parallax | aviary_layout.md | yes | Subtle parallax appears in scene composition. |
| 63 | No UI chrome inside aviary view | aviary_layout.md | yes | Chrome is a thin DOM top bar over the scene. |
| 64 | Top bar contents | aviary_layout.md | yes | borderline: Top-bar shell and controls are planned, though contents are distributed rather than enumerated. |
| 65 | Top bar auto-fades | aviary_layout.md | yes | Pointer/keydown restore and idle fade are covered. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Mid-motion first frame is explicit. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Quiet field and no spinner are explicit. |
| 68 | Empty-aviary state | aviary_layout.md | no | No between-adoption empty state appears. |
| 69 | Color palette spec | aviary_layout.md | no | No calm naturalist palette or saturated-accent rule appears. |
| 70 | Responsive but never crops a bird out of frame | aviary_layout.md | no | No no-crop responsive rule appears. |
| 71 | Email + magic-link sign-in | accounts_sync.md | yes | Magic-link auth endpoints are planned. |
| 72 | Magic link expiry | accounts_sync.md | yes | 15-minute single-use tokens are planned. |
| 73 | Single-user accounts | accounts_sync.md | yes | Single-user, one aviary per account is in scope. |
| 74 | Synthetic account ID | accounts_sync.md | yes | Synthetic UUIDs and encrypted email are explicit. |
| 75 | Server-side simulation tick | accounts_sync.md | yes | Scheduled tick worker is central. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Snapshot pulls on visibility change and keepalive are planned. |
| 77 | Client interpolates between snapshots | accounts_sync.md | yes | Snapshot-to-snapshot interpolation is planned. |
| 78 | Multi-device sync | accounts_sync.md | yes | Canonical server store and no client merge are planned. |
| 79 | Last-write-wins forbidden for personality state | accounts_sync.md | yes | Append-only events and no mutable aggregate overwrite are explicit. |
| 80 | Conflict resolution via server tick only writer | accounts_sync.md | yes | The tick is the sole writer of personality/mood. |
| 81 | Sync conflict surface matter-of-fact tone | accounts_sync.md | no | Matter-of-fact tone appears for unsupported browser, not sync conflict/account error surfaces. |
| 82 | Per-device session token | accounts_sync.md | yes | Session tokens, device labels, and revocation are planned. |
| 83 | Account export | accounts_sync.md | yes | JSON export job/email link is planned. |
| 84 | Account deletion | accounts_sync.md | yes | 30-day soft delete and undo before hard delete are planned. |
| 85 | No telemetry on per-bird interactions for ML | accounts_sync.md | yes | Per-bird/per-account interaction data is never aggregated or used for ML. |
| 86 | Aggregate-only telemetry | accounts_sync.md | yes | Aggregate metrics with no account_id dimension are planned. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy policy link is planned. |
| 88 | Email change flow | accounts_sync.md | yes | PATCH email initiates verification flow. |
| 89 | Visit invitations | social_optional.md | yes | Email-based per-invite sharing is planned. |
| 90 | Visits default OFF | social_optional.md | yes | Visit feature is opt-in and off by default. |
| 91 | Visit is read-only ambient view | social_optional.md | yes | Visitor sessions are read-only and cannot write events. |
| 92 | Visitor cannot trigger greetings/listen-in/offers | social_optional.md | yes | No POST /aviary/events access for visitors. |
| 93 | No chat/comments/avatars during visits | social_optional.md | yes | No profiles, follows, discovery, leaderboards, or comments are explicit; avatars are implied by no social surfaces. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | borderline: Visit notifications default false, though the exact notification surface is not elaborated. |
| 95 | Visit revocation | social_optional.md | yes | Invite revocation endpoint is planned. |
| 96 | Visit log | social_optional.md | yes | Host visit log endpoint and data are planned. |
| 97 | Visitor sees host aviary as-is | social_optional.md | yes | Visitor snapshot is the actual canonical state with no special rendering. |
| 98 | No leaderboards/discovery/public aviaries | social_optional.md | yes | No discovery, public aviaries, or leaderboards are explicit and structurally unsupported. |
| 99 | Screen-reader narration of aviary state | accessibility_perf.md | yes | Dedicated naturalist narration endpoint and ARIA live region are planned. |
| 100 | Narration cadence is slow | accessibility_perf.md | yes | 30-60s idle cadence and priority event path are planned. |
| 101 | Narration prose is naturalist | accessibility_perf.md | yes | Narration vocabulary and naturalist prose are explicit. |
| 102 | Reduced-motion mode | accessibility_perf.md | yes | Cross-fade render path is planned. |
| 103 | Reduced-motion preserves charm | accessibility_perf.md | yes | Designed visual register, not fallback, is explicit. |
| 104 | Captioning toggle for calls | accessibility_perf.md | yes | Captioning is generated with call grammar and defaults on without WebAudio. |
| 105 | WCAG AA contrast | accessibility_perf.md | yes | AA tokens reviewed against bright and dim states. |
| 106 | Keyboard-only navigation | accessibility_perf.md | yes | DOM hit targets, roving focus, Enter/Escape, and offer selection are planned. |
| 107 | Focus indicators | accessibility_perf.md | yes | High-contrast focus indicators are planned. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Bundle-size CI gate is planned. |
| 109 | Time to first bird <500ms | accessibility_perf.md | yes | Embedded first snapshot and <500ms budget are planned. |
| 110 | 60fps idle motion target | accessibility_perf.md | yes | Synthetic throttled checks are planned. |
| 111 | No memory growth over 30 minutes | accessibility_perf.md | yes | Heap-snapshot CI test and object pooling are planned. |
| 112 | Procedural audio client-side | accessibility_perf.md | yes | WebAudio synthesis and no audio assets are planned. |
| 113 | Audio fallback without WebAudio | accessibility_perf.md | yes | Graceful silence plus captions-on fallback is planned. |
| 114 | Performance observability | accessibility_perf.md | yes | Synthetic checks and aggregate RUM are planned. |
| 115 | Simulation tick latency alarm | accessibility_perf.md | yes | p99 >5s alarm is planned. |
| 116 | Browser support matrix | accessibility_perf.md | yes | Last-two-majors support and unsupported-browser surface are planned. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Native app is explicitly out of scope. |
| 118 | Out of scope: gamification | non_goals.md | yes | Gamification is explicitly and structurally out of scope. |
| 119 | Out of scope: Tamagotchi mechanics | non_goals.md | yes | Death, hunger, decay, distress, and negative drift are excluded. |
| 120 | Out of scope: social network surfaces | non_goals.md | yes | Profiles, follows, public discovery, comments, and leaderboards are excluded. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **71.4%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 — feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System-level intent: "slow, ambient, non-game pacing"; "procedural media over static loops"; "the aviary continues without the viewer" | PLAN.md §§2,7,8: "thin renderer and event emitter"; "mid-motion first frame"; "no two calls are byte-identical"; quiet field, server tick, procedural audio | yes | yes | no | partial | L1/L2 survive; the downstream staleness/leak-across-the-product consequence is not reconstructed. |
| S2 — notice-never-announce | 4 | included | RECONSTRUCTION.md System-level intent: "No gamification of any kind"; "no fields" for streak counters; visit feature has "no profiles, follows, discovery, leaderboards, or comments" | PLAN.md §§1,5.4,12: no gamification; no visit-cadence triggers; future gamified surfaces require deliberate schema/API changes | yes | yes | yes | partial | Announcement refusals survive as constraints, but the noticed-vs-announced affective rationale is mostly absent. |
| S3 — charm-from-specificity | 2 | included | RECONSTRUCTION.md System-level intent: "Naturalist voice and specificity"; templates are "parameterized by the specific bird names/states involved" | PLAN.md §§5.4,9: notebook/narration/captions use naturalist templates and specific bird state; raw personality values are hidden | yes | yes | no | full | Specific, naturalist, bird-state prose is explicitly identified and grounded. |
| S4 — restraint-over-richness | 2 | included | RECONSTRUCTION.md Per-feature whys: two starter birds are "fixed at launch"; single horizontal scene is marked not recoverable; system notes small services and bounded surfaces | PLAN.md §§1,7,10: two starters, cap seven, one screen, top-bar chrome, 2D canvas, bundle budget, no customizable scenes | no | yes | yes | partial | The plan preserves restrained scope, but B does not articulate the depth-over-variety rationale. |
| S5 — naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System-level intent: "Naturalist voice and specificity"; unsupported browser uses a "matter-of-fact unsupported-browser surface" | PLAN.md §§5.4,8,9,10: naturalist templates/captions/narration; unsupported-browser surface is matter-of-fact | yes | no | no | partial | Naturalist voice survives strongly; the account/error/sync exception is only weakly grounded. |
| S6 — presence-is-real-interaction | 4 | included | RECONSTRUCTION.md System-level intent: "Presence accounting" is "attention to the aviary as a whole"; settle "closes the presence window" | PLAN.md §§5.1,5.4,6: presence-time is dominant drift input; settle closes window; no visit-frequency/streak surfaces | yes | yes | yes | partial | The attention-as-input principle survives, but precise three-signal accounting and settle/tab-close equivalence are missing. |
| S7 — simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System-level intent: "Server-as-sole-writer authority"; client is a "thin renderer and event emitter"; sync is "conflict-free by construction" | PLAN.md §§2,4,6: tick runs independently; server is sole writer; clients write append-only events; no client absolute state endpoint | yes | yes | no | full | Server-side canonical state, multi-device coherence, and LWW avoidance are all reconstructed. |
| S8 — privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System-level intent: "Privacy, PII minimization, and aggregate-only observation"; metrics layer does not accept `account_id` | PLAN.md §§1,3,10: email stored once encrypted; per-bird/per-account data never aggregated or used for ML; aggregate-only telemetry | yes | yes | no | full | Privacy is recovered as a technical data-pipeline boundary. |
| S9 — accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System-level intent: "Accessibility as a designed v1 surface"; reduced motion is "not animations-off"; launch gating requires reduced motion and narration | PLAN.md §§1,7,9,11: reduced motion and narration ship with v1; designed visual register; accessibility QA before beta; launch go/no-go gates | yes | yes | no | full | Charm, designed alternatives, and v1 launch timing all survive. |

Multi-layer system-level recovery:

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | yes | yes | no |
| S2 | yes | no | yes |
| S6 | yes | no | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: server tick continues without clients; mid-motion first frame; quiet-field loading; procedural calls; reduced-motion charm; narration/captions.
- S2: non-gamification schema friction; no streak/visit fields; no visit-cadence notebook triggers; visit notifications default false; no discovery/leaderboards.
- S3: specific notebook templates; naturalist narration/captions; bird names; hidden trait numbers; no comparison surfaces.
- S4: two starters and cap seven; single horizontal scene; no customizable scenes; top-bar chrome; 2D canvas/bundle restraint.
- S5: naturalist notebook/narration/captions; field-notebook templates; matter-of-fact unsupported browser/account-adjacent surfaces, but sync-error exception is thin.
- S6: presence-time drift input; settle closes presence window; no streak/cadence surfaces; monotonic no-punishment drift, but exact presence precision is missing.
- S7: scheduled tick; canonical state store; client event log only; no absolute state endpoint; append-only deltas; snapshot polling across devices.
- S8: synthetic IDs; email stored once; aggregate-only telemetry; no account_id metrics; per-bird data excluded from ML; hard deletion/export surfaces.
- S9: naturalist narration; reduced-motion designed path; captions; keyboard/focus/contrast; accessibility QA and launch gates.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **50.0%**.

Reachable feature-level whys: **37 / 40**. Excluded unreachable whys: **3 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md: presence is "attention to the aviary as a whole" | PLAN.md §5.1: "Presence-time (dominant)"; no visibility/focus/recent-input conjunction | yes | none | Captured borderline, but the precision rationale is absent. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md: "Low-pass drift calibration"; one-week and three-week targets; "screensaver" versus "Tamagotchi" failure | PLAN.md §5.1 and §12: low-pass filter, 7-day/3-week thresholds, screensaver/Tamagotchi calibration risk | no | full | All three calibration layers are recovered. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md: "Monotonic expressive growth, not decay"; "No subtraction path"; tied to "No Tamagotchi mechanics" | PLAN.md §§1,5.1: monotonic up-only deltas; negative values rejected; no Tamagotchi mechanics | no | partial | Up-only and no-Tamagotchi survive; the quieter-not-mistrust downstream consequence is absent. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md: "Procedural media over static loops"; "no two calls are byte-identical"; "real chorus" not phase artifact | PLAN.md §8: motif grammar, WebAudio synthesis, independent node graphs, no audio asset pipeline | no | full | Procedural variation, chorus dependency, and no-recorded-fallback cascade survive. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md: idle motion lets the "user read mood from motion without a label" | PLAN.md §7: mood-weighted idle actions make behavior recognizably mood-shaped | no | full | The affective reason is stated nearly directly. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md: "Cap of 7 birds: NOT RECOVERABLE FROM PLAN" | PLAN.md §§1,11: cap of 7 and birds-per-aviary ramp are specified without recognizability rationale | yes | none | Rule captured, empirical recognizability why not recovered. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md: server is sole writer; "Stable bird IDs" and no client ownership of personality state | PLAN.md §§2,3,6: personality fields are canonical server state; tick only writer; append-only deltas | no | partial | Canonical persistence and sync consequences survive; losing-vector-as-deleting-bird rationale is absent. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md: public projection has "no path to raw personality columns"; hints preserve "never exposed numerically" | PLAN.md §§3,8: `BirdPublicView` excludes raw traits; snapshot sends derived hints | yes | none | The hiding rule survives, but not the stat-management relationship why. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md: "Return-greeting: NOT RECOVERABLE FROM PLAN" | PLAN.md §§1,9: return-greeting is in scope and a user-initiated narration event | yes | none | Feature named, but one-bird/procedural/notice-not-announce why is not recovered. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | The plan does not capture the no-textual-welcome rule. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Close-tab equivalence and optionality are not captured. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md: notebook is sparse, specific "naturalist prose"; templates avoid "generic event-log strings"; cooldowns every few days | PLAN.md §5.4: naturalist templates, named triggers, cooldown/rarity budgets, read-only sparse notebook | no | full | Naturalist voice, event-log refusal, and rarity survive. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md: "Presence accounting" uses "attention to the aviary as a whole" | PLAN.md §§4,5.1: presence pings and presence-time exist, but exact signal conjunction does not | yes | none | The plan/reconstruction preserve presence pings but not the precision/failure-mode why. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md: non-goals have "no fields" for streak counters; notebook triggers never read visit-frequency | PLAN.md §§1,5.4,12: no streaks/counters/calendars; no visit-cadence triggers; deliberate schema/API friction | yes | partial | Rule and disguised-surface boundary survive; user-intention rotation into number-management is not reconstructed. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md: "Mid-motion first frame and quiet-field loading"; avoids blank/loading frames and spinner | PLAN.md §7: seeded current action hints, zero blank frames, quiet field with faint cues, no transition animation | no | full | Continuing scene, snapshot implementation, and quiet-field fallback are recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md: email stored "exactly once, encrypted" and kept out of table keys, logs, partitions, telemetry | PLAN.md §3: all IDs synthetic UUIDs; email stored exactly once encrypted; no PII elsewhere | no | partial | PII containment survives; impossible-to-retrofit rationale is absent. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md: scheduled tick runs independently; server is sole writer; sync is conflict-free by construction | PLAN.md §§2,5,6: tick runs every ~60s, whether connected or not; clients render snapshots; server writes canonical state | no | full | Tick authority and multi-device rationale are fully recovered. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md: append-only log has "no mutable aggregate fields"; "there is nothing to overwrite"; no absolute state endpoint | PLAN.md §§3,4,6: client writes only events; tick consumes ordered log; no mood/personality endpoint | no | full | Additive event model, overwrite failure, and implementation rule are all present. |
| F19 | sync-conflict-tone | 2 | no | unreachable_excluded | none | none | no | unreachable | The plan does not capture sync-conflict/account-error tone specifically. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md: per-bird/per-account interaction data is "never aggregated, never used for ML, never shared"; metrics do not accept `account_id` | PLAN.md §§1,10: aggregate-only telemetry; no per-account dimensions; telemetry never touches per-bird interaction data | no | partial | Storage/use boundary and pipeline enforcement survive; relationship-as-private-data rationale is not reconstructed. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md: visitor tokens are read-only with "no event-log write access"; visitors see "actual canonical state" | PLAN.md §§2,4: visitor token has no POST /aviary/events capability; visitor snapshot uses actual canonical state | no | full | Observation-not-co-presence is recovered through capability absence. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md: visit feature is "per-invite opt-in" and "off by default" | PLAN.md §3: `visit_notifications_enabled: bool`, default false | yes | none | Default-off rule appears, but attention-driver rationale is not reconstructed. |
| F23 | no-leaderboards | 2 | yes | included | RECONSTRUCTION.md: no "profiles, follows, discovery, leaderboards, or comments"; no fields for leaderboards | PLAN.md §§1,12: no public discovery/leaderboards; no backing fields or telemetry pipeline | yes | none | Rule and architecture survive, not the theirs-vs-compared relationship why. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md: naturalist narration reads same snapshot so narration and visuals cannot become "two different products"; ARIA queue cap avoids flooding | PLAN.md §§2,9: narration endpoint, naturalist prose, same snapshot, priority event path | no | full | Naturalist prose, equal product feel, and implementation guardrails are recovered. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md: reduced motion is a "structurally separate render path" and "designed visual register," not bolted on | PLAN.md §§7,9,11: cross-fade render path, slower color shifts, designed from day one and launch-gated | no | partial | Designed rendering and non-stripped fallback survive; calls/drift/notebook continuity layer is not explicit. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md: embedded first snapshot avoids a second round trip before first bird can render | PLAN.md §10: <500ms budget, embedded/edge-cached snapshot, draw without non-critical assets | yes | none | Performance rule survives, but not the affective threshold rationale. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md: no gamification is an "absolute constraint"; future gamified surfaces require deliberate schema/API changes | PLAN.md §§1,12: no achievements/streaks/levels/badges/counters; schema/API friction is intentional | no | partial | Explicit refusal and future-creep guard survive; cheap-engagement temptation layer is absent. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md: monotonic expressive growth rejects "No Tamagotchi mechanics" and any negative delta | PLAN.md §§1,5.1: no death, hunger, decay, distress; drift is asymmetric up-only | no | full | The no-punishing-absence principle is recovered through monotonic/no-decay mechanics. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION.md: two starter birds are "fixed at launch" and a "day-one product decision" | PLAN.md §§1,11: two starter birds at adoption; species drawn from pool; names user-assigned | yes | none | Starter rule survives, but arrivals-not-catalog rationale does not. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md: age-gated growth rationale is "pacing" and threshold tuning | PLAN.md §§1,11: new birds gated by aviary age, not score; hot-tunable age thresholds | yes | none | The age rule is present, but the anti-reward-loop why is not reconstructed. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md: stable `bird_id` values support the "identity-continuity rule"; birds are "never regenerated, never replaced" | PLAN.md §3: `bird_id` stable forever; never regenerated/replaced; renaming has no simulation effect | yes | partial | Identity continuity rule survives; vector-vs-identity and retroactive-relationship consequences are absent. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md: stored mood avoids any "session-start reinitialization path" | PLAN.md §5.2: mood persists as a stored column and does not snap to a hardcoded default | yes | none | Persistence rule survives, but the continued-while-away illusion why is not reconstructed. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md: field notebook is sparse, specific naturalist prose | PLAN.md §1: field notebook is read-only; §5.4: generated observer-style entries | yes | none | Read-only observer-record rationale is not reconstructed. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION.md: "Account export: NOT RECOVERABLE FROM PLAN" | PLAN.md §§1,4: account export triggers a JSON snapshot job emailed to the user | yes | none | Feature present; relationship-copy/quiet-QOL why absent. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md: undo endpoint labelled "I changed my mind" gives soft-delete rationale | PLAN.md §§1,4: soft deletion, 30-day undo, then hard delete are planned | no | partial | Accidental-regret layer survives; privacy hard-delete and delete-all-together layers do not. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md: aggregate RUM has no `account_id`, preventing per-account telemetry leaks | PLAN.md §10: metrics-emission layer does not accept `account_id`; aggregate request/latency/error metrics only | no | full | Technical telemetry boundary and observability-backdoor risk are recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md: visits are "per-invite opt-in" and a narrowly bounded social affordance | PLAN.md §§1,4: POST /visits/invite takes visitor_email; no public discovery/social surfaces | yes | none | Per-invite rule survives, but private-relationship-not-publishing rationale is absent. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md: approximate visit duration avoids "behavioral-tracking surface" | PLAN.md §§3,4: host visit log endpoint with invite/visit data and default false notifications | yes | none | Visit-log mechanics survive, but transparency-without-attention-loop why is absent. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md: visitors see the "actual canonical state" and no special visitor rendering | PLAN.md §4: visitor-scoped snapshot uses same response shape and actual canonical state | yes | none | Actual-aviary rule survives, but no-show-off-mode rationale is absent. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md: ARIA live narration has 30-60s cadence, queue depth cap, and priority events without faster idle cadence | PLAN.md §9: idle narration 30-60s, `aria-live=polite`, queue depth capped at 1, user events get priority path | no | full | Slow rhythm, queue-overwhelm risk, and sparse event exception are all recovered. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | no | no | no |
| F2 | yes | yes | yes |
| F3 | yes | yes | no |
| F4 | yes | yes | yes |
| F7 | yes | no | yes |
| F9 | no | no | no |
| F10 | unreachable | unreachable | unreachable |
| F12 | yes | yes | yes |
| F13 | no | no | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | yes |
| F25 | yes | no | yes |
| F27 | yes | no | yes |
| F30 | no | no | no |
| F31 | yes | no | no |
| F35 | yes | no | no |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From benchmark constants |
| Possible total weight | 152 | Full-instance possible weight |
| Reachable gold whys | 46 | S whys always included; F whys included only when captured |
| Excluded unreachable feature whys | 3 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 78 / 144 | Sum of weight x recovery-score over included whys |
| Whys with reconstruction evidence | 46 | Rows with exact frozen-reconstruction evidence, including honest non-recovery evidence |
| Whys with PLAN grounding | 46 | Rows with exact plan grounding for the rule or rationale |
| `rule_without_why` cases | 21 | Mechanism survived without the gold rationale |
| `plan_only_not_reconstructed` cases | 10 | Notable missing layers where PLAN had more rationale than B carried forward |
| `ungrounded_reconstruction` cases | 0 | No material confabulation found |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 | 30 | 55.6% |
| Affective whys | 90 | 48 | 53.3% |
| Weight-2 whys | 40 | 14 | 35.0% |
| Weight-3 whys | 104 | 64 | 61.5% |
| System-level whys | 28 | 20 | 71.4% |
| Feature-level whys (reachable) | 116 | 58 | 50.0% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered at 55.6% and affective whys at 53.3%, so the gap is small. The sharpest affective losses were relationship-frame exceptions: F8, F23, F29, F32, F33, F37, F38, and F39 kept the rule but not the human reason.
- **Weight-3 vs weight-2.** Weight-3 whys recovered much better (61.5%) than weight-2 whys (35.0%). The plan and reconstruction preserved load-bearing architecture and calibration better than smaller exception rationales.
- **System-level vs feature-level.** System-level fidelity (71.4%) substantially exceeded feature-level fidelity (50.0%). The planner encoded principles like server authority, accessibility, privacy, and procedural media, but many feature-specific whys collapsed into mechanisms.
- **Multi-layer recovery patterns.** Primary implementation causes survived most often. Secondary "why this tempting shortcut fails" and downstream relationship consequences were the common drops, especially F1, F3, F7, F14, F16, F20, F25, F27, F31, and F35.
- **Subdomain patterns.** Audio, sync, accessibility narration, and core drift calibration were strong. Return greeting, presence precision, account/sync tone, social sharing nuance, and account export/deletion rationale were weaker.
- **Evidence-bound effects.** Several v1-style apparent recoveries were denied because only the rule survived: F8 hid trait numbers, F23 banned leaderboards, F29 fixed starter birds, F30 used age gates, F37/F39 bounded sharing, but the reconstruction did not carry the gold rationale.

The failure shape suggests a strong implementation planner: architecture and testable mechanisms were durable, but affective exception rationales compressed away unless the plan itself repeatedly named them as engineering constraints.

## 4. Recommendations for v2 hardening

- Add more targeted single-layer exceptions like F29-F40; they exposed meaningful headroom where the plan captured the rule but not the reason.
- Keep multi-layer whys for calibration, sync, accessibility, and social exceptions. Layer scoring showed exactly which downstream consequences were lost.
- Consider adding a specific capture convention for "named but underspecified" features such as return-greeting and presence; these were reachable under the inclusive rule but low-recovery, which is useful but subjective.
- Preserve the system-level cross-cutting bar, but require scorers to document whether the plan preserved the principle or merely repeated a scoped rule. S4 and S5 show the ambiguity.
- Include more feature-level whys around tone surfaces and explicit absences. The sync-error tone and no-welcome-toast misses were high-signal despite being small features.

## 5. Methodology caveats

- **Fresh-context fidelity.** I treated the phase-2A reconstruction as frozen and plan-derived. The validity audit found no clear contamination signs.
- **Single-run limitation.** This is one candidate and one frozen reconstruction; no variance signal is available inside this scoring slot.
- **Borderline capture calls.** I leaned inclusive on glossary/domain vocabulary, presence definition, settle gesture, evening palette, top-bar contents, and visit notifications. The largest downstream effect was keeping F1 reachable despite the missing three-signal presence definition.
- **System-level cross-cutting.** The (b) bar was most subjective for S2, S4, S5, and S6. I awarded partial rather than full where mechanisms repeated but the principle was not fully reconstructed.
- **Confabulation cases.** I did not find material ungrounded reconstruction. Weak rows were generally honest rule-only recovery rather than invented rationale.
- **Evidence-bound denials.** F8, F13, F23, F26, F29, F30, F32-F34, and F37-F39 are the clearest cases where the rule appeared but the gold why did not.
- **Rule-without-why cases.** There were 21 included cases where operational behavior survived without the full gold rationale; this is the main diagnostic for this run.

End of report.
