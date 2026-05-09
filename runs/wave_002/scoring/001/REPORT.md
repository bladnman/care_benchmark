# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Frozen reconstruction scored against PLAN, GOLD_WHYS, and RUBRIC.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **95.8%** |
| Intent fidelity | **77.0%** |
| Combined quality | **9560** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **75.8%**
- (Planning, fidelity) coordinate: `(95.8, 77.0)`

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-09T03:05:22Z |
| Candidate model | claude-opus-4-7 |
| Candidate effort | extra-high |
| Candidate harness | claude-code |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **115 / 120** = **95.8%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 17 | 85.0% |
| aviary_layout.md | 18 | 16 | 88.9% |
| accounts_sync.md | 18 | 18 | 100.0% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **115** | **95.8%** |

Per-feature detail:

| # | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes |  |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes |  |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes |  |
| 4 | Voice-and-tone guide for product surface | product_brief.md | yes |  |
| 5 | "What this is not" callout | product_brief.md | yes |  |
| 6 | Restraint-over-richness scope statement | product_brief.md | yes |  |
| 7 | Glossary of domain terms | concepts.md | yes | Captured by distributed definitions rather than a standalone glossary. |
| 8 | Definition of "presence" | concepts.md | yes |  |
| 9 | Definition of personality vector vs mood | concepts.md | yes |  |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes |  |
| 11 | Personality vector traits | bird_engine.md | yes |  |
| 12 | Personality drift function | bird_engine.md | yes |  |
| 13 | Drift rate calibration | bird_engine.md | yes |  |
| 14 | Personality drift is monotonic toward expressive | bird_engine.md | yes |  |
| 15 | Mood state | bird_engine.md | yes |  |
| 16 | Mood inputs | bird_engine.md | yes |  |
| 17 | Procedural call grammar | bird_engine.md | yes |  |
| 18 | Per-bird call signature | bird_engine.md | yes |  |
| 19 | Chorus mixing | bird_engine.md | yes |  |
| 20 | Call timing shaped by personality | bird_engine.md | yes |  |
| 21 | Idle micro-motion | bird_engine.md | yes |  |
| 22 | Mood-shaped idle motion | bird_engine.md | yes |  |
| 23 | Bird species pool for v1 | bird_engine.md | yes |  |
| 24 | Bird naming | bird_engine.md | yes |  |
| 25 | Adoption flow | bird_engine.md | yes |  |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes |  |
| 27 | Adding a third+ bird | bird_engine.md | yes |  |
| 28 | Personality vector persistence | bird_engine.md | yes |  |
| 29 | Mood persistence across sessions | bird_engine.md | yes |  |
| 30 | Bird-to-bird interaction | bird_engine.md | yes |  |
| 31 | Bird identity stability | bird_engine.md | yes |  |
| 32 | Personality vector exposure | bird_engine.md | yes |  |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Return greeting is present mainly as session-start/a bird-notices framing. |
| 34 | Greeting variation by absence length | interactions.md | no | No absence-length greeting rule in PLAN. |
| 35 | Greeting variation by bird boldness | interactions.md | no | No boldness-first greeting rule in PLAN. |
| 36 | Greeting stagger | interactions.md | no | No greeting-stagger rule in PLAN. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes |  |
| 38 | Listen-in interaction | interactions.md | yes |  |
| 39 | Listen-in mix decay | interactions.md | yes |  |
| 40 | Offer interaction | interactions.md | yes |  |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured inclusively via offer reaction priority plus mood/curiosity shaping. |
| 42 | Offer cooldown | interactions.md | yes |  |
| 43 | Settle gesture | interactions.md | yes |  |
| 44 | Settle is opt-in | interactions.md | yes |  |
| 45 | Field notebook auto-entries | interactions.md | yes |  |
| 46 | Field notebook entry frequency | interactions.md | yes |  |
| 47 | Field notebook is read-only | interactions.md | yes |  |
| 48 | Presence accounting | interactions.md | yes |  |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes |  |
| 50 | No streak counter, no "days visited" display | interactions.md | yes |  |
| 51 | Background-tab pause | interactions.md | yes | Captured inclusively via visibility/presence gating and server-side continuation. |
| 52 | Click-anywhere-to-undo for settle gesture | interactions.md | yes |  |
| 53 | Single horizontal scene | aviary_layout.md | yes |  |
| 54 | Three perch zones | aviary_layout.md | yes |  |
| 55 | Bird-chosen perch | aviary_layout.md | yes |  |
| 56 | Day/night cycle tied to local time | aviary_layout.md | yes |  |
| 57 | Evening palette shift | aviary_layout.md | yes | Palette/day-night transition captured; quieter evening calls not explicit. |
| 58 | Night state | aviary_layout.md | no | Night phase appears, but no most-birds-settled/nightjar-like active rule. |
| 59 | Ambient weather | aviary_layout.md | yes |  |
| 60 | Weather affects mood | aviary_layout.md | yes |  |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes |  |
| 62 | Foreground/background parallax | aviary_layout.md | yes |  |
| 63 | No UI chrome inside aviary view | aviary_layout.md | yes |  |
| 64 | Top bar contents | aviary_layout.md | yes |  |
| 65 | Top bar auto-fades | aviary_layout.md | yes |  |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes |  |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes |  |
| 68 | Empty-aviary state | aviary_layout.md | yes |  |
| 69 | Color palette spec | aviary_layout.md | yes | Calm naturalist palette captured as design framing, not a full token spec. |
| 70 | Responsive scene never crops a bird | aviary_layout.md | no | Responsive caption placement appears, but no no-crop bird framing rule. |
| 71 | Email + magic-link sign-in | accounts_sync.md | yes |  |
| 72 | Magic link expiry | accounts_sync.md | yes |  |
| 73 | Single-user accounts | accounts_sync.md | yes |  |
| 74 | Synthetic account ID | accounts_sync.md | yes |  |
| 75 | Server-side simulation tick | accounts_sync.md | yes |  |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes |  |
| 77 | Client interpolates between snapshots | accounts_sync.md | yes |  |
| 78 | Multi-device sync | accounts_sync.md | yes |  |
| 79 | Last-write-wins forbidden for personality state | accounts_sync.md | yes |  |
| 80 | Conflict resolution: server tick only writer | accounts_sync.md | yes |  |
| 81 | Sync conflict surface matter-of-fact | accounts_sync.md | yes |  |
| 82 | Per-device session token | accounts_sync.md | yes |  |
| 83 | Account export | accounts_sync.md | yes |  |
| 84 | Account deletion | accounts_sync.md | yes |  |
| 85 | No telemetry on per-bird interactions for ML | accounts_sync.md | yes |  |
| 86 | Aggregate-only telemetry | accounts_sync.md | yes |  |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes |  |
| 88 | Email change flow | accounts_sync.md | yes |  |
| 89 | Visit invitations | social_optional.md | yes |  |
| 90 | Visits default OFF | social_optional.md | yes |  |
| 91 | Visit is read-only ambient view | social_optional.md | yes |  |
| 92 | Visitor cannot trigger greetings/listen-in/offers | social_optional.md | yes |  |
| 93 | No chat/comments/avatars during visits | social_optional.md | yes |  |
| 94 | No friend visited notification by default | social_optional.md | yes |  |
| 95 | Visit revocation | social_optional.md | yes |  |
| 96 | Visit log | social_optional.md | yes |  |
| 97 | Visitor sees host aviary as it is | social_optional.md | yes |  |
| 98 | No leaderboards/discovery/public aviaries | social_optional.md | yes |  |
| 99 | Screen-reader narration running prose | accessibility_perf.md | yes |  |
| 100 | Narration cadence is slow | accessibility_perf.md | yes |  |
| 101 | Narration prose naturalist | accessibility_perf.md | yes |  |
| 102 | Reduced-motion mode | accessibility_perf.md | yes |  |
| 103 | Reduced-motion preserves charm | accessibility_perf.md | yes |  |
| 104 | Captioning toggle for procedural calls | accessibility_perf.md | yes |  |
| 105 | WCAG AA contrast | accessibility_perf.md | yes |  |
| 106 | Keyboard-only navigation | accessibility_perf.md | yes |  |
| 107 | Focus indicators visible | accessibility_perf.md | yes |  |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes |  |
| 109 | Time to first bird visible <500ms | accessibility_perf.md | yes |  |
| 110 | 60fps idle motion target | accessibility_perf.md | yes |  |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes |  |
| 112 | Procedural audio synthesized client-side | accessibility_perf.md | yes |  |
| 113 | Audio fallback without WebAudio | accessibility_perf.md | yes |  |
| 114 | Performance observability | accessibility_perf.md | yes |  |
| 115 | Simulation tick latency error budget | accessibility_perf.md | yes |  |
| 116 | Browser support matrix | accessibility_perf.md | yes |  |
| 117 | Out of scope: native mobile app | non_goals.md | yes |  |
| 118 | Out of scope: gamification | non_goals.md | yes |  |
| 119 | Out of scope: Tamagotchi-style mechanics | non_goals.md | yes |  |
| 120 | Out of scope: social network surfaces | non_goals.md | yes |  |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **82.1%**.

| Why ID | Weight | Denom | Reconstruction evidence | PLAN grounding | B identified? | PLAN cross-cutting? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System: "felt-aliveness, naturalist voice, restraint"; "Procedural life rather than canned theater". | PLAN.md intro: "shortcuts ... destroy the product's center"; final: "aviary appears already in motion". | yes | yes | no | full | All three aliveness layers survived: continuing place, cross-layer procedural life, and shortcut-failure consequence. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md System: "Non-goals as architectural absences"; no engagement dashboard; no streak/social-feed mechanics. | PLAN.md scope/non-goals/final: no welcome toast, no notifications, no streaks, no engagement metrics. | yes | yes | no | partial | Broad refusal of announcement/engagement survived, but the noticed-vs-announced affective distinction was thinned. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md System: "Naturalist voice split from system voice" and notebook/caption/narration voice continuity. | PLAN.md notebook/narration sections: naturalist prose, grounded facts, no generic event-log treatment. | no | yes | no | partial | PLAN preserves specificity strongly; reconstruction mostly reduced it to voice-system separation. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md System: product center includes "restraint"; sparse top-bar and no in-scene labels. | PLAN.md scope/non-goals: two starters, cap seven, one scene, sparse chrome, no social/gamification expansion. | yes | yes | no | full | Restraint appears as a cross-cutting implementation constraint rather than just tone. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System: "naturalist voice" for aviary/notebook/narration/captions; "matter-of-fact voice" for system surfaces. | PLAN.md API/accessibility/cross-cutting: voice modules split naturalist from matter-of-fact copy. | yes | yes | no | full | The voice split and its structural enforcement are recovered cleanly. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md System: drift is "driven primarily by presence" and absence is not punished; sync says settle means "presence ends here". | PLAN.md sync/simulation: presence pings require visibility + focus + recent input; presence dominates drift; settle is per-session. | yes | yes | no | partial | Presence as drift input and settle equivalence survived; the laxer-signal corruption rationale was not reconstructed. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System: "server is the only writer"; clients emit events and "never send absolute state". | PLAN.md architecture/sync: server-only personality writes, canonical snapshots, no client-vs-client personality conflict. | yes | yes | no | full | Canonical server-side simulation, multi-device coherence, and no-last-write-wins consequences all survived. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System: per-bird data "never reaches analytics or training pipelines"; topology and lints block leakage. | PLAN.md telemetry/privacy: analytics warehouse isolated, telemetry cannot import simulation/events, aggregate-only metrics. | yes | yes | no | full | The private-relationship data boundary is recovered as a technical topology, not just policy. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System: accessibility is "a designed surface, not a checklist"; reduced motion is "not a stripped fallback". | PLAN.md accessibility/rollout: narration, captions, keyboard, reduced motion, and v1 launch inclusion are first-class. | yes | yes | no | full | Designed accessibility, quality parity, and launch-time inclusion all survived. |

Multi-layer system-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | yes |
| S6 | yes | no | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

Cross-cutting evidence appendix:

- S1: PLAN.md intro: "shortcuts ... destroy the product's center"; final: "aviary appears already in motion".
- S2: PLAN.md scope/non-goals/final: no welcome toast, no notifications, no streaks, no engagement metrics.
- S3: PLAN.md notebook/narration sections: naturalist prose, grounded facts, no generic event-log treatment.
- S4: PLAN.md scope/non-goals: two starters, cap seven, one scene, sparse chrome, no social/gamification expansion.
- S5: PLAN.md API/accessibility/cross-cutting: voice modules split naturalist from matter-of-fact copy.
- S6: PLAN.md sync/simulation: presence pings require visibility + focus + recent input; presence dominates drift; settle is per-session.
- S7: PLAN.md architecture/sync: server-only personality writes, canonical snapshots, no client-vs-client personality conflict.
- S8: PLAN.md telemetry/privacy: analytics warehouse isolated, telemetry cannot import simulation/events, aggregate-only metrics.
- S9: PLAN.md accessibility/rollout: narration, captions, keyboard, reduced motion, and v1 launch inclusion are first-class.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity: **75.8%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denom | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md: presence pings dominate drift; conditions hold visibility + focus + recent input. | PLAN.md sync: presence pings every ~15s while visibility + focus + recent input hold. | no | partial | Recovered the precision-as-drift-input layer only; laxer-signal failure and silent corruption were absent. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md: 7-day measurable / 21-day visible targets; calibration too fast/too slow risk. | PLAN.md drift/risk: low-pass filter; measurable at ~7 days, visible at ~21; Tamagotchi/screensaver risk. | no | full | All calibration layers survived. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md: one-sided clamping; birds do not decay, suffer, or punish neglect. | PLAN.md drift/final: max clamp; ignored bird becomes ambient, not distressed. | no | full | No-punishment exception recovered. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md: procedural-only audio avoids canned-software signal; audio is affective spine. | PLAN.md audio: procedural-only, no recorded fallback, chorus payoff, graceful silence if WebAudio fails. | no | full | Procedural audio why recovered across call, chorus, and fallback layers. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md: mood expression in motion makes mood visible without labels. | PLAN.md frontend: wary/content/curious/drowsy/alert map to posture and movement. | no | full | Recovered visible-mood-without-label rationale. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md: cap tied to observed call recognizability and chorus/species recognition. | PLAN.md rollout/audio: cap rises based on observed call recognizability; species signatures tuned. | no | full | Recovered the recognizability ceiling rationale. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md: personality and bird identity are precious; vector loss is catastrophic and silent. | PLAN.md data/sync/risks: server-side vector, no reset, migration tests preserve vector and bird_id. | no | full | Persistence, relationship loss, and sync consequences survived. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md: hidden vector supports felt behavior; relationship is not reduced to visible stats. | PLAN.md snapshot/final: snapshots omit personality vector; no debug surface exposes it. | no | full | Recovered stat-management refusal. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md: narration gives priority bump for "return-greeting on session start" only. | PLAN.md final/accessibility: a bird notices on return; return-greeting gets narration priority. | yes | none | Rule survived only thinly; no one-bird/absence/boldness/procedural why recovered. |
| F10 | no-welcome-back-toast | 4 | yes | included | none | PLAN.md final/cross-cutting: no "Welcome back" toast; forbidden phrase lint blocks welcome-back copy. | yes | none | PLAN carried the rule, but the frozen reconstruction did not recover this why. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md: settle means "presence ends here" and is per-session. | PLAN.md sync: settle is per-session; presence ends when presence ends. | yes | none | Mechanism survived, but not the opt-in/chore/ordinary-tab-close rationale. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md: notebook carries naturalist voice in sparse, grounded prose. | PLAN.md notebook: template grammar, rare entries, read-only, grounded facts, no LLM runtime. | no | full | Naturalist prose, concentrated voice, rarity, and read-only boundary recovered. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md: presence pings while visibility + focus + recent input hold; presence dominates drift. | PLAN.md sync/simulation: same conjunction and presence-weighted drift. | no | partial | Recovered conjunction and drift input; missed individual-signal/laptop-left-open failure rationale. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md: streaks excluded in disguise; metrics absent so reintroduction is costly. | PLAN.md non-goals/schema absence: no streaks, calendars, visit-count metrics, or user-behavior observations. | no | full | No-streak rationale and disguised-surface consequence recovered. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md: quiet field, no spinner/loading label; first bird budget and already-in-motion framing. | PLAN.md frontend/final: first frame mid-action; quiet field; no spinner or entry animation. | no | full | Already-running conceit and no-spinner consequence recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md: email decrypted only by auth/mailer and never used as identifier outside account table. | PLAN.md data model: account_id UUID; email encrypted once and never an identifier elsewhere. | no | partial | Recovered the identifier rule, but not the full PII-leak/retrofit rationale. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md: tick is architectural heart and single writer of canonical aliveness. | PLAN.md tick/sync: tick consumes ordered events, writes canonical state, runs whether connected or not. | no | full | Server tick, sync coherence, and client-collapse consequence recovered. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md: client emits events, never absolute state; lack of client-write code path prevents conflicts. | PLAN.md sync/conflict: clients never submit absolute personality; server tick consumes append-only events. | no | partial | Recovered no-absolute-state implementation, but not the concrete lost-morning-drift failure. |
| F19 | sync-conflict-tone | 2 | yes | included | RECONSTRUCTION.md: naturalist voice for aviary, matter-of-fact voice for errors/settings/system surfaces. | PLAN.md API/accessibility: system errors render matter-of-fact, never naturalist. | no | full | Recovered error-context voice exception. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md: per-bird simulation data never reaches analytics or training pipelines; topology/lints block leakage. | PLAN.md telemetry: event log has no telemetry path; analytics cannot read simulation DB. | no | full | Private-relationship, ML exclusion, and pipeline boundary recovered. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md: visits are read-only ambient view; visit snapshot rejects events. | PLAN.md visits API: visitor cannot submit events; read-only snapshot only. | no | full | Recovered observation-not-write/co-presence boundary. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md: visit notify default off; product does not pull users back or surface pressure. | PLAN.md visits/scope: no push/email/in-product visit notification by default; toggle off by default. | no | full | Recovered attention-driver refusal. |
| F23 | no-leaderboards | 2 | yes | included | RECONSTRUCTION.md: no profiles, feeds, leaderboards, friend chains, or chat so visits do not become a social network. | PLAN.md non-goals: no leaderboards, public feed, discovery, or comparison surfaces. | no | full | Recovered public-comparison/product-shift refusal. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md: narration gives same naturalist, lowercase, present-tense aviary; ARIA-style lists forbidden. | PLAN.md accessibility: running naturalist prose, slow cadence, no state-list narration. | no | full | Narration prose, parity, and implementation exception recovered. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md: designed surface keeps birds drifting, moods changing, and notebook accruing. | PLAN.md reduced motion: cross-fades, calls/captions unaffected, birds still drift, charm preserved. | no | full | All reduced-motion charm layers recovered. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md: edge initial state and tiny critical chunk support the <500ms first-bird target. | PLAN.md initial paint/perf: inline snapshot, critical chunk, synthetic first-bird test. | no | full | Recovered affective-performance bridge. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md: gamification shifts toward engagement mechanics; future PR checklist guards against streaks/achievements. | PLAN.md non-goals/risks: no achievements/streaks/counters; metrics absent; temptation explicitly mitigated. | no | full | No-gamification exception and future-foothold risk recovered. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md: birds do not die, hunger, decay, or suffer; absence is not punished. | PLAN.md non-goals/final: no distress/decay; ignored bird becomes ambient, not distressed. | no | full | Recovered observational-not-custodial refusal. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION.md: "NOT RECOVERABLE FROM PLAN" for two starter birds. | PLAN.md scope: two starter birds are system-chosen species and user-named. | yes | none | Feature captured, but the arrival-not-catalog why was absent from reconstruction. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md: "NOT RECOVERABLE FROM PLAN" for age-tied milestones. | PLAN.md scope/rollout: new species offered at age-tied milestones/intervals. | yes | none | Feature captured, but reward-loop rationale was not recovered. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md: bird_id is never reused/reset/regenerated/swapped; identity continuity protects relationship. | PLAN.md sync/migrations: preserve bird_id and vector; no reset-bird feature/tool/debug command. | no | full | Identity, distinction from data reset, and retroactive relationship loss recovered. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md: persisted mood lets mood survive sessions and multi-device sync without recomputation. | PLAN.md mood/sync: mood persisted on bird row; survives sessions; no default recomputation. | no | full | Recovered continuity rationale sufficiently. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md: read-only protects observed naturalist prose rather than user-authored log or game journal. | PLAN.md notebook: read-only, template-generated, fact-grounded observer record. | no | full | Recovered observer-record vs journal distinction. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION.md: export gives account control through on-demand JSON and short-TTL emailed link. | PLAN.md export API/scope: on-demand JSON export via emailed signed link. | yes | none | Mechanism survived; relationship-copy/quiet-quality rationale did not. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md: soft delete allows recovery; hard deletion gives final lifecycle. | PLAN.md account deletion: 30-day recovery and hard delete after 30 days; privacy boundary elsewhere. | no | partial | Recovered grace/recovery layer; privacy and residue-deletion layers were thin. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md: aggregate RUM only; static schema blocks forbidden fields and relationship tracking. | PLAN.md telemetry: no per-account dimensions, no per-bird state, analytics isolated from simulation DB. | no | full | Recovered technical observability boundary. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md: visits are email-bound, opt-in, read-only, revocable, and off by default. | PLAN.md visits/security: invites bind to a single visitor email; no friend chains or implicit sharing. | no | full | Recovered deliberate named sharing vs ambient discovery. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md: visit-notify is default off; product does not pull users back or create pressure. | PLAN.md visits: visit log exists in settings; no badge/push/email attention surface by default. | no | full | Recovered transparency without notification loop. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md: visit snapshot uses identical schema to host snapshot. | PLAN.md visits API: visitor gets read-only snapshot with identical schema. | yes | none | Rule survived, but not the no-show-off/marketing-rendering rationale. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md: slow naturalist updates, polite live region, priority bumps, and no overwhelming. | PLAN.md narration: updates every 30-60s, user events prioritized, state-list chatter blocked. | no | full | Cadence, overload, and observational-priority layers recovered. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | no | no |
| F2 | yes | yes | yes |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | yes | yes |
| F9 | no | no | no |
| F10 | no | no | no |
| F12 | yes | yes | yes |
| F13 | yes | no | no |
| F14 | yes | yes | yes |
| F15 | yes | yes | yes |
| F16 | yes | no | no |
| F17 | yes | yes | yes |
| F18 | yes | no | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | yes | yes |
| F30 | no | no | no |
| F31 | yes | yes | yes |
| F35 | yes | no | no |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From `gold_why_totals` |
| Possible total weight | 152 | From constants |
| Reachable gold whys | 49 | 9 system + reachable feature whys |
| Excluded unreachable feature whys | 0 | Denominator exclusions |
| Recovered / reachable weight | 117.0 / 152.0 | Weighted numerator / denominator |
| Whys with reconstruction evidence | 48 | Exact reconstruction evidence field not `none` |
| Whys with PLAN grounding | 49 | Exact plan grounding present |
| `rule_without_why` cases | 7 | Mechanism survived without rationale |
| `plan_only_not_reconstructed` cases | 2 | PLAN carried rationale but reconstruction did not |
| `ungrounded_reconstruction` cases | 0 | No clear ungrounded rationale assertions |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 40.0 | 74.1% |
| Affective whys | 98.0 | 77.0 | 78.6% |
| Weight 2 whys | 44.0 | 35.0 | 79.5% |
| Weight 3 whys | 108.0 | 82.0 | 75.9% |
| System-level whys | 28.0 | 23.0 | 82.1% |
| Feature-level whys (reachable) | 124.0 | 94.0 | 75.8% |

## 3. Diagnostic patterns

- **Affective vs functional.** Recovery was close: functional 40.0/54.0 and affective 77.0/98.0. Functional losses were concentrated in precision/privacy lifecycle details (F16, F18, F34, F35); affective losses clustered around return-greeting, welcome, new-bird rewards, and visitor show-off boundaries.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 82.0/108.0; weight-2 whys recovered 35.0/44.0. High-weight system and architecture whys did well, but several high-weight interaction exceptions (F9, F10, F30) failed completely.
- **System-level vs feature-level.** System-level fidelity (82.1%) beat feature-level fidelity (75.8%). The planner encoded philosophy broadly, but the blind reconstruction flattened some per-feature rationale into rules.
- **Multi-layer pattern.** Secondary-contributor layers were dropped most often: laxer-presence corruption, synthetic-ID PII blast radius, last-write-wins lost-drift example, and deletion/privacy residue were thinner than primary rules.
- **Subdomain pattern.** Audio, accessibility, server simulation, privacy topology, and telemetry were strong. Return greeting, growth/unlock framing, export/deletion, and visitor actualness were weaker.
- **Evidence-bound effects.** F29 and F30 explicitly said `NOT RECOVERABLE FROM PLAN`; F34 and F39 preserved mechanisms but not gold rationale. These are exactly the cases v06 is designed to expose.

## 4. Recommendations for v2 hardening

- Keep targeted headroom rows like F29-F40. They produced real separation in a plan that otherwise captured almost everything.
- Add or group greeting-return whys more deliberately. The PLAN lost several greeting micro-rules while preserving the broader no-toast surface.
- Clarify the boundary between naturalist voice and charm-from-specificity. S3 was a subjective partial because the reconstruction recovered voice continuity more than specificity.
- Preserve the strict `NOT RECOVERABLE FROM PLAN` handling. Honest non-recovery made F29/F30 cleaner to score and reduced confabulation risk.
- Consider a small checklist for plan-side rationale on data lifecycle features. Export and deletion mechanisms were captured, but their relationship/privacy whys were thin.

## 5. Methodology caveats

- **Fresh-context fidelity.** I treated the phase 2A reconstruction as frozen and did not modify it. Same-context safeguard appears to hold from the provided setup.
- **Single-run-at-temperature limitation.** This is one run only; no variance signal is available inside this slot.
- **Borderline capture calls.** I counted features 7, 33, 41, 51, 57, and 69 inclusively under RUBRIC §4.7. The missed features were 34, 35, 36, 58, and 70.
- **System-level cross-cutting.** The strict 3-feature bar was easy for S1, S5, S7, S8, and S9; S2, S3, and S6 required judgment because the PLAN preserved the principle better than the reconstruction named it.
- **Confabulation cases.** No material ungrounded reconstruction cases were found. Losses were mostly compression or explicit non-recovery.
- **Evidence-bound denials.** F9, F10, F11, F29, F30, F34, and F39 are the main rule-without-why cases.
- **Operational compromise.** `TIMING.json` had phase 1 and phase 2A timing for runs.001 only; no phase 2B timing was fabricated.

---
End of report.
