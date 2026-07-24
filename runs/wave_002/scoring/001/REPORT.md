# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. This report scores the frozen reconstruction for run 001 against the gold why list using exact reconstruction evidence plus PLAN grounding.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **95.0%** |
| Intent fidelity | **50.7%** |
| Combined quality | **9451** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **43.5%**

**(Planning, fidelity) coordinate:** `(95.0, 50.7)`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-07-24T12:15:09Z |
| Candidate model | kimi-k3 |
| Candidate effort | low |
| Candidate harness | opencode |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **114 / 120** = **95.0%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 3 | 75.0% |
| bird_engine.md | 22 | 21 | 95.5% |
| interactions.md | 20 | 19 | 95.0% |
| aviary_layout.md | 18 | 16 | 88.9% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **114** | **95.0%** |

Missed features were narrow and mostly peripheral: glossary, species-pool count, offer-reaction variation, empty-aviary state, palette spec, and privacy-policy link. Borderline captures were counted inclusively per RUBRIC section 4.7.

| Feature ID | Feature title | File | Captured | Note |
|---|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Scope describes a web-only one-scene aviary with birds, accounts, visits, accessibility, and export/deletion. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured through procedural variation, server-side tick, mid-action first frame, and no-spinner loading. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured through return-greeting as bird behavior, no gamification, no pushed notifications, and rejected toast proposals. |
| 4 | Voice-and-tone guide for product surface | product_brief.md | yes | Captured via naturalist notebook/narration/captions and matter-of-fact auth/session error copy. |
| 5 | "What this is not" callout | product_brief.md | yes | Explicit non-goals cover gamification, Tamagotchi mechanics, social-network surfaces, notifications, recorded audio, and numeric vectors. |
| 6 | Restraint-over-richness scope statement | product_brief.md | yes | Captured by two starter birds, seven-bird cap, single scene, no UI inside scene, and no chrome-heavy aviary. |
| 7 | Glossary of domain terms | concepts.md | no | The plan uses domain terms but does not include or plan a glossary-like domain reference. |
| 8 | Definition of "presence" | concepts.md | yes | Presence is defined as the 3-signal conjunction and dominant drift input. |
| 9 | Definition of personality vector vs mood | concepts.md | yes | Personality drift is slow; mood is a faster state machine over recent events, time, weather, and personality. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Settle is a user event with no directional drift, evening transition, and undo. |
| 11 | Personality vector traits | bird_engine.md | yes | The Bird model enumerates boldness, social warmth, vocal frequency, plumage saturation, and curiosity. |
| 12 | Personality drift function | bird_engine.md | yes | Tick applies a low-pass filter to presence and interaction signals. |
| 13 | Drift rate calibration | bird_engine.md | yes | Plan pins one-week instrument-visible and three-week user-visible drift. |
| 14 | Personality drift is monotonic toward expressive | bird_engine.md | yes | Signals push up only; absence applies zero, never negative, delta. |
| 15 | Mood state | bird_engine.md | yes | Mood enum and state-machine behavior are specified. |
| 16 | Mood inputs | bird_engine.md | yes | Inputs include recent interactions, time of day, ambient weather, and personality modulation. |
| 17 | Procedural call grammar | bird_engine.md | yes | Per-species motif library, jitter, mood weighting, and WebAudio synthesis are specified. |
| 18 | Per-bird call signature | bird_engine.md | yes | Fixed seed-derived timbre offsets preserve call signature stability. |
| 19 | Chorus mixing | bird_engine.md | yes | Bird-to-bird coupling and per-bird gain nodes produce chorus behavior. |
| 20 | Call timing shaped by personality | bird_engine.md | yes | Vocal frequency affects call rate and warmth affects response likelihood. |
| 21 | Idle micro-motion | bird_engine.md | yes | Preen, scan, head-tilt, and weight-shuffle are mood/personality-modulated. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Idle micro-motion is explicitly mood-keyed. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | no | Species are referenced, but no v1 pool size or roughly-six species target appears. |
| 24 | Bird naming | bird_engine.md | yes | borderline; Name field and rename endpoint are present; adoption-time naming is implied rather than spelled out. |
| 25 | Adoption flow | bird_engine.md | yes | Two starter birds are auto-selected for new accounts. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | The plan states the cap of seven. |
| 27 | Adding a third+ bird | bird_engine.md | yes | Additional birds unlock only by aviary age. |
| 28 | Personality vector persistence | bird_engine.md | yes | Server-only persisted personality vectors are canonical. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Mood persists across sessions and advances during absence. |
| 30 | Bird-to-bird interaction | bird_engine.md | yes | Call response scheduling, wary spread, and chorus emergence are included. |
| 31 | Bird identity stability | bird_engine.md | yes | Birds have stable UUIDs that are never regenerated. |
| 32 | Personality vector exposure | bird_engine.md | yes | Raw vectors are server-only and never exposed numerically on the wire or UI. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Greeter selection varies by boldness, mood, absence length, and procedural execution. |
| 34 | Greeting variation by absence length | interactions.md | yes | Absence buckets drive greeting intensity. |
| 35 | Greeting variation by bird boldness | interactions.md | yes | Boldness participates in greeter weighting. |
| 36 | Greeting stagger | interactions.md | yes | Qualified greeters are staggered by random offsets. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | borderline; No pushed notifications and rejected toast proposals capture the refusal, though no dedicated welcome-toast line appears. |
| 38 | Listen-in interaction | interactions.md | yes | Listen-in focuses a bird and raises its call in the mix. |
| 39 | Listen-in mix decay | interactions.md | yes | Other birds ramp down to an ambient floor, never silence. |
| 40 | Offer interaction | interactions.md | yes | Seed, song-fragment, and still-pool offers are included. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | Offer events affect drift inputs, but visible reaction variation by mood/curiosity is not specified. |
| 42 | Offer cooldown | interactions.md | yes | Per-bird offer cooldown is fixed at 5 minutes. |
| 43 | Settle gesture | interactions.md | yes | Settle triggers a slow evening shift. |
| 44 | Settle is opt-in | interactions.md | yes | borderline; Captured inclusively through no penalty/zero-drift absence semantics, but close-tab equivalence is not explicit. |
| 45 | Field notebook auto-entries | interactions.md | yes | Event-driven notebook generation and naturalist sparse prose are specified. |
| 46 | Field notebook entry frequency | interactions.md | yes | Sparsity governor limits entries to roughly every 2-3 days with a weekly cap. |
| 47 | Field notebook is read-only | interactions.md | yes | Scope states the notebook is read-only. |
| 48 | Presence accounting | interactions.md | yes | Presence pings fold into presence-time as dominant drift input. |
| 49 | Presence accounting requires focus + cursor + visibility | interactions.md | yes | The 3-signal conjunction is explicit. |
| 50 | No streak counter, no days-visited display | interactions.md | yes | No streaks, counters, badges, visit calendars, or user-behavior notebook entries. |
| 51 | Background-tab pause | interactions.md | yes | Render loop halts when hidden while simulation continues server-side. |
| 52 | Click-anywhere-to-undo settle | interactions.md | yes | Settle has a 5s any-click undo. |
| 53 | Single horizontal scene | aviary_layout.md | yes | Single horizontal canvas scene is specified. |
| 54 | Three perch zones | aviary_layout.md | yes | Snapshot includes perch zone, and scene has foreground/middle/background perch zones. |
| 55 | Bird-chosen perch | aviary_layout.md | yes | Tick chooses perch positions from mood and personality; user placement is absent. |
| 56 | Day/night cycle | aviary_layout.md | yes | Palette keyframes are driven by local time. |
| 57 | Evening palette shift | aviary_layout.md | yes | Settle and day/night transitions include evening/color shifts. |
| 58 | Night state | aviary_layout.md | yes | Night state includes one nocturnal species active. |
| 59 | Ambient weather | aviary_layout.md | yes | Rare rain and occasional wind are included. |
| 60 | Weather affects mood | aviary_layout.md | yes | Rain dampens vocal frequency; wind drives alert/wary split. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Leaves and feathers drift client-side. |
| 62 | Foreground/background parallax | aviary_layout.md | yes | Three parallax planes are included. |
| 63 | No UI chrome inside aviary view | aviary_layout.md | yes | Top bar is above scene and no UI lives inside the scene. |
| 64 | Top bar contents | aviary_layout.md | yes | Account, accessibility, notebook, and offer controls are listed. |
| 65 | Top bar auto-fades | aviary_layout.md | yes | Top bar fades after cursor stillness and returns on movement/keyboard. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Birds are placed mid-action with phase-offset idle loops. |
| 67 | Loading state is a quiet field | aviary_layout.md | yes | Slow connection shows a quiet field, not a spinner. |
| 68 | Empty-aviary state | aviary_layout.md | no | No between-adoption empty aviary state is planned. |
| 69 | Color palette spec | aviary_layout.md | no | Soft sky and day/night color appear, but no calm naturalist palette spec or accent-color refusal is given. |
| 70 | Responsive but never crops birds | aviary_layout.md | yes | All birds remain in frame with responsive perch spacing. |
| 71 | Email + magic-link sign-in | accounts_sync.md | yes | Magic-link auth is specified. |
| 72 | Magic link expiry | accounts_sync.md | yes | Links expire in 15 minutes. |
| 73 | Single-user accounts | accounts_sync.md | yes | One aviary per account in v1. |
| 74 | Synthetic account ID | accounts_sync.md | yes | Synthetic UUID is the only identifier used in logs, telemetry, and sharding. |
| 75 | Server-side simulation tick | accounts_sync.md | yes | Server tick runs roughly once per minute. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Snapshots pull on load and visibility changes. |
| 77 | Client interpolates between snapshots | accounts_sync.md | yes | Interpolation prevents teleporting. |
| 78 | Multi-device sync | accounts_sync.md | yes | All devices read the same canonical server record. |
| 79 | Last-write-wins forbidden | accounts_sync.md | yes | No last-write-wins anywhere; personality deltas are server-authored. |
| 80 | Conflict resolution tick-only writer | accounts_sync.md | yes | Only the server tick writes personality vectors. |
| 81 | Sync conflict matter-of-fact tone | accounts_sync.md | yes | Auth/session conflicts use matter-of-fact error copy and re-auth. |
| 82 | Per-device session token | accounts_sync.md | yes | Per-device sessions are revocable and listed in account settings. |
| 83 | Account export | accounts_sync.md | yes | Account export emails a JSON link. |
| 84 | Account deletion | accounts_sync.md | yes | Soft-delete for 30 days, then hard-delete. |
| 85 | No telemetry on per-bird interactions for ML | accounts_sync.md | yes | Per-bird state and per-account dimensions are excluded from telemetry. |
| 86 | Aggregate-only telemetry | accounts_sync.md | yes | RUM uses aggregate timings, errors, and anonymized histograms only. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | Privacy policy link is not mentioned. |
| 88 | Email change flow | accounts_sync.md | yes | Email change endpoint verifies the new address before switching. |
| 89 | Visit invitations | social_optional.md | yes | Email-based per-invite visit invitations are present. |
| 90 | Visits default OFF | social_optional.md | yes | Visits are opt-in and off by default. |
| 91 | Visit is read-only ambient view | social_optional.md | yes | Visitor snapshots are render-only and reject event ingest. |
| 92 | Visitor cannot trigger greetings/listen-in/offers | social_optional.md | yes | Visitor endpoint rejects events, so visitors cannot interact. |
| 93 | No chat/comments/avatars during visits | social_optional.md | yes | Social-network surfaces including chat, avatars, and comments are explicitly out. |
| 94 | No friend visited notification by default | social_optional.md | yes | Visit notifications are off by default and visits log silently. |
| 95 | Visit revocation | social_optional.md | yes | Invites can be revoked and revocation is enforced at snapshot pull. |
| 96 | Visit log | social_optional.md | yes | Visit log endpoint and log entries are included. |
| 97 | Visitor sees host aviary as it is | social_optional.md | yes | borderline; Captured by rendering the host snapshot, though "no show-off mode" is not named. |
| 98 | No leaderboards/discovery/public aviaries | social_optional.md | yes | No leaderboards, discovery, public feeds, or leaderboard substrate. |
| 99 | Screen-reader narration of aviary state | accessibility_perf.md | yes | ARIA live naturalist prose narration is included. |
| 100 | Narration cadence is slow | accessibility_perf.md | yes | Narration updates every 30-60 seconds at idle. |
| 101 | Narration prose is naturalist | accessibility_perf.md | yes | Narration uses the same naturalist voice as notebook prose. |
| 102 | Reduced-motion mode | accessibility_perf.md | yes | Reduced motion uses cross-fades and slower transitions. |
| 103 | Reduced-motion preserves charm | accessibility_perf.md | yes | Reduced motion is a designed surface, not a stripped fallback. |
| 104 | Captioning toggle for procedural calls | accessibility_perf.md | yes | Captions derive from the same motif and parameters as the calls. |
| 105 | WCAG AA contrast | accessibility_perf.md | yes | WCAG AA checks are part of CI. |
| 106 | Keyboard-only navigation | accessibility_perf.md | yes | Keyboard path covers top bar, scene, listen-in, offer, settle, and disengage. |
| 107 | Focus indicators visible | accessibility_perf.md | yes | High-contrast focus indicators are required against bright and dim scenes. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Initial JS budget is under 2MB gzipped. |
| 109 | Time to first bird <500ms | accessibility_perf.md | yes | First bird target is under 500ms on mid-tier mobile/4G. |
| 110 | 60fps idle motion target | accessibility_perf.md | yes | 60fps idle is CI-enforced for a 30-minute session on reference laptop. |
| 111 | No memory growth | accessibility_perf.md | yes | Heap growth tests are a hard gate. |
| 112 | Procedural audio client-side | accessibility_perf.md | yes | WebAudio synthesis ships no audio files. |
| 113 | WebAudio fallback | accessibility_perf.md | yes | Fallback is graceful silence plus captions. |
| 114 | Performance observability | accessibility_perf.md | yes | Synthetic fleet and aggregate RUM are included. |
| 115 | Simulation tick latency budget | accessibility_perf.md | yes | Tick p99 alarm fires above 5s. |
| 116 | Browser support matrix | accessibility_perf.md | yes | Last 2 major versions of Chrome, Safari, Firefox, and Edge are specified. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Native apps are explicitly out. |
| 118 | Out of scope: gamification | non_goals.md | yes | Achievements, streaks, scores, badges, XP, ranks, and tiers are explicitly out. |
| 119 | Out of scope: Tamagotchi mechanics | non_goals.md | yes | Death, hunger, distress, and decaying meters are explicitly out. |
| 120 | Out of scope: social network surfaces | non_goals.md | yes | Profiles, follows, feeds, discovery, co-presence, chat, avatars, and comments are explicitly out. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **82.1%** (23/28 weighted).

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | System intent: "Procedural variation with recognizable individuals"; "no spinner, no entry animation"; audio repetition would "break the spell". | PLAN secs. 5, 7, 8: procedural calls; birds mid-action; quiet field; first bird <500ms; no recorded audio. | yes | yes | no | full | Recovered as a cross-cutting aliveness principle spanning animation, audio, loading, tick state, and performance. |
| S2 - notice-never-announce | 4 | included | System intent: "Ambient, non-gamified, non-social product shape"; risks reject every "harmless" streak, toast, or notification proposal. | PLAN secs. 1, 5, 12: return greeting, no notifications, no gamification, no streaks, rejected toast proposals. | yes | yes | no | partial | The refusal of announcement surfaces survived, but the processed-vs-seen affective register was not articulated. |
| S3 - charm-from-specificity | 2 | included | System intent: "Naturalist, observation-voiced prose"; notebook entries "observe the aviary, never the user's behavior". | PLAN secs. 5, 8, 9: notebook phrase variation, naturalist captions, naturalist narration, no generic entries. | yes | yes | no | full | Specific, observational voice survived across notebook, captions, and narration. |
| S4 - restraint-over-richness | 2 | included | none | PLAN secs. 1, 7, 8: two starter birds, cap of seven, single scene, no UI inside scene, listen-in not soloing. | no | yes | no | partial | The plan preserves restraint, but the reconstruction did not identify it as a system-level philosophy. |
| S5 - naturalist-voice-with-system-exception | 2 | included | System intent: "Naturalist, observation-voiced prose"; sync row: conflicts handled with "matter-of-fact error copy and re-auth". | PLAN secs. 6, 9, 12: naturalist narration/notebook/captions plus matter-of-fact auth/session errors. | yes | yes | no | full | Both naturalist product voice and system-copy exception are present. |
| S6 - presence-is-real-interaction | 4 | included | Presence row: any "tab open" shortcut would "corrupt drift population-wide"; drift row says absence applies "zero (never negative) delta". | PLAN secs. 5, 7, 12: 3-signal conjunction, presence-time dominant, no negative absence drift, render loop halts hidden. | yes | yes | no | partial | Presence precision and population-drift risk survived; settle-vs-close-tab equivalence was not explicit. |
| S7 - simulation-runs-server-side | 4 | included | System intent: "Server-authored life, client-rendered presence"; sync has "Single canonical state per account" and "nothing to merge". | PLAN secs. 2, 5, 6: tick is only writer, clients append events, no last-write-wins, multi-device reads same record. | yes | yes | no | full | Server-authored canonical state, sync coherence, and merge/LWW avoidance are all recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | System intent: "Privacy boundary by architecture, not just policy"; simulation DB "never read by analytics pipelines". | PLAN secs. 2, 3, 10: no warehouse connection, separate credentials, no per-bird state or per-account dimensions. | yes | yes | no | full | Privacy is recovered as an architectural/data-pipeline boundary. |
| S9 - accessibility-as-first-class-surface | 4 | included | System intent: "Accessibility is part of v1, not a fallback"; reduced motion is "a designed surface, not a stripped fallback". | PLAN secs. 1, 7, 9, 12: screen-reader prose, reduced motion, captions, keyboard, ship gate launch-blocking. | yes | yes | no | full | The reconstruction preserves charm, designed alternatives, and launch-blocking accessibility. |

Multi-layer system-level whys:

| Why ID | L1 primary | L2 secondary | L3 downstream |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | yes |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: PLAN secs. 5/7/8/10: low-pass drift, procedural calls, birds mid-action, no spinner, first-bird budget, WebAudio fallback.
- S2: PLAN secs. 1/5/10/12: no pushed notifications, return greeting, no streaks, no user-behavior notebook, rejected toast proposals.
- S3: PLAN secs. 5/8/9/12: notebook phrase banks, captions, narration, no generic entries, named birds and no comparison surfaces.
- S4: PLAN secs. 1/7/8: two starter birds, cap of seven, one horizontal scene, no scene chrome, listen-in not solo tracks.
- S5: PLAN secs. 5/6/9: naturalist notebook/narration/captions, matter-of-fact auth/session conflict copy, system settings surfaces.
- S6: PLAN secs. 5/7/12: 3-signal presence, dominant presence drift, absence zero delta, hidden render pause, no streaks/Tamagotchi.
- S7: PLAN secs. 2/5/6/10: server tick, canonical state, clients append events only, no LWW, tick latency observability.
- S8: PLAN secs. 2/3/10: no analytics access to sim DB, no per-bird aggregation, aggregate-only RUM, export/deletion.
- S9: PLAN secs. 1/7/9/12: screen-reader prose, designed reduced motion, captions, keyboard, focus, launch-blocking ship gate.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity: **43.5%** (54/124 weighted).

Reachable feature-level whys: **40 / 40**. No feature-level why was denominator-excluded.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | Presence row: any shortcut such as "tab open" would "corrupt drift population-wide". | PLAN secs. 1, 5, 12: 3-signal conjunction, presence-time dominant, server sanity checks. | no | partial | Recovered precision/drift risk, but not the per-signal failure analysis. |
| F2 | drift-function | 4 | yes | included | Low-pass drift row names one-week instruments, three-week users; risk: too fast -> Tamagotchi, too slow -> screensaver. | PLAN secs. 5, 11, 12: low-pass filter, one-week/three-week targets, failure-mode calibration. | no | full | All three calibration layers survived. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | Monotonic row: signals only push up and absence applies "zero (never negative) delta," avoiding Tamagotchi decay. | PLAN secs. 5, 12: absence applies zero delta; no death, hunger, distress, or decaying meters. | no | partial | Recovered no-negative-drift and no-Tamagotchi intent; did not recover the return-after-absence affective consequence. |
| F4 | procedural-call-grammar | 4 | yes | included | Procedural calls row: varied and "not a canned clip"; audio risk says repetition would "break the spell". | PLAN secs. 5, 8, 12: motif library, seeded jitter, WebAudio synthesis, no recorded audio path. | no | partial | Recovered no loops and WebAudio cascade; missed the recorded-loop chorus phase artifact layer. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none | PLAN sec. 7: mood-keyed animation state machines are specified. | yes | none | Rule survived, but the why that users read mood from motion without labels did not. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | PLAN secs. 1, 11: cap of seven and age ramp are specified. | yes | none | Seven-bird cap survived without the empirical recognizability rationale. |
| F7 | vector-persistence | 4 | yes | included | Bird personality vector row: protects hidden personality state and supports tick-only write path. | PLAN secs. 2, 3, 6: server-only write access, canonical vectors, no client personality writes. | no | partial | Recovered canonical/server-side persistence, but not the deletion-of-the-known-bird rationale. |
| F8 | vector-never-shown-numerically | 2 | yes | included | none | PLAN sec. 2: raw vectors are never exposed; render hints are coarsened into buckets. | yes | none | Mechanism and privacy survived, not the stat-management relationship rationale. |
| F9 | return-greeting | 4 | yes | included | Return-greeting row: absence buckets, boldness, mood availability, and procedurally varied execution. | PLAN sec. 5: greeter weighted by boldness/mood; absence buckets choose intensity; staggered offsets. | no | partial | Recovered variation and one-bird greeting shape; not the notice-never-announce consequence. |
| F10 | no-welcome-back-toast | 4 | yes | included | none | PLAN secs. 1, 12: no notifications and rejected toast proposals. | yes | none | No-toast rule is inferable, but the bird-greeting-as-entire-welcome why was not reconstructed. |
| F11 | settle-is-opt-in | 2 | yes | included | none | PLAN secs. 5, 7: settle has no directional drift and closes presence cleanly. | yes | none | Settle mechanics survived; optionality/chore/close-tab equivalence did not. |
| F12 | field-notebook-prose | 4 | yes | included | Field notebook row: sparse aviary observations; entries observe the aviary rather than user behavior. | PLAN sec. 5: phrase banks, day-part/weather lexicon, sparsity governor, never user-behavior phrasing. | no | partial | Naturalist/sparse observer record survived, but the stock-event-log-breaks-spell layer was thin. |
| F13 | presence-accounting | 4 | yes | included | Folding presence row: presence-time is dominant; shortcut risk corrupts drift population-wide. | PLAN secs. 5, 12: validated pings, 3-signal conjunction, rejection of implausible durations. | no | partial | Recovered conjunction and drift corruption; not the individual-signal miss cases. |
| F14 | no-streak-counter | 4 | yes | included | System intent excludes gamification; notebook entries observe aviary, never user behavior. | PLAN secs. 1, 5, 10, 12: no streaks/counters/visit calendars; no user-behavior notebook or metrics. | no | partial | Rule and adjacent-disguise boundary survived; the intention-rotation rationale did not. |
| F15 | scene-loads-with-motion | 4 | yes | included | Immediate experience row: "no spinner, no entry animation," quiet field, first bird visible <500ms. | PLAN sec. 7: snapshot inlined, birds mid-action, phase-offset loops, quiet field loading state. | no | partial | Recovered mid-action and quiet-field layers; server-snapshot central-conceit explanation was partial. |
| F16 | synthetic-account-id | 4 | yes | included | Synthetic UUID row: avoid email or another direct identifier in logs, telemetry, and sharding. | PLAN sec. 3: synthetic UUID is only identifier; email encrypted once; used for logs, telemetry, sharding. | no | partial | Recovered PII/leakage prevention; not the easy-now/impossible-retrofit consequence. |
| F17 | server-side-sim-tick | 4 | yes | included | Tick row: server pass reads events, advances moods/drift, writes checkpoints; sync row has one canonical record. | PLAN secs. 5, 6: tick every ~60s, server only writer, clients render snapshots, devices read same state. | no | partial | Recovered server tick and sync coherence; not the full client-tick-collapse scenario. |
| F18 | no-last-write-wins | 4 | yes | included | No-LWW row: additive server-authored deltas and ordered event consumption avoid merge conflicts. | PLAN secs. 5, 6: clients append events only; no last-write-wins anywhere; tick consumes logs in order. | no | partial | Recovered additive/tick-only implementation, but not the silent overwrite example. |
| F19 | sync-conflict-tone | 2 | yes | included | none | PLAN sec. 6: conflicts use matter-of-fact error copy and re-auth. | yes | none | Matter-of-fact rule survived without the evasive-naturalist-error rationale. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | Privacy rows: simulation DB blocked from analytics; no per-bird/per-account data; no relationship reconstruction. | PLAN secs. 2, 3, 10: no warehouse connection, no per-bird state, no per-account dimensions, no population interaction analysis. | no | full | Privacy, relationship-data rationale, and technical pipeline boundary all survived. |
| F21 | visit-read-only-ambient | 2 | yes | included | Visit row: visitor snapshots reject event ingest so visitors cannot affect canonical state. | PLAN secs. 1, 4: read-only ambient visits, rejects event ingest, logs visit duration silently. | no | full | Read-only observation and no visitor drift survived. |
| F22 | no-friend-visited-notification | 2 | yes | included | none | PLAN secs. 1, 4: visit notifications off by default and visits are logged silently. | yes | none | Notification rule survived, not the attention-driver loop rationale. |
| F23 | no-leaderboards | 2 | yes | included | none | PLAN secs. 1, 10: no discovery/feed/leaderboards and no leaderboard substrate. | yes | none | Rule survived, but the compared-to-other-people relationship shift was not reconstructed. |
| F24 | sr-narration-running-prose | 4 | yes | included | Narration row: naturalist observations from snapshot state, not a state list; priority bumps remain observation-voiced. | PLAN sec. 9: aria-live naturalist prose, 30-60s idle cadence, priority bump for user events. | no | partial | Recovered prose and implementation shape; not the same-right-to-feel/voice-continuity layer. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | Reduced-motion row: cross-fades, removed ambient drift, slowed color transitions, "designed rather than stripped." | PLAN secs. 7, 9: cross-fade pose sequences, calls/captions settings, designed surface, ship gate. | no | partial | Recovered designed alternative and not-stripped consequence; not all preserved-aviary elements were reconstructed. |
| F26 | ttfb-500ms | 2 | yes | included | Immediate experience row: first bird visible <500ms as part of a quiet, continuously framed experience. | PLAN secs. 7, 10: snapshot inlined at CDN edge; first bird <500ms on mid-tier mobile over 4G. | no | full | Affective performance bridge is sufficiently recovered. |
| F27 | no-gamification-non-goal | 4 | yes | included | System intent excludes gamification; risks reject every "harmless" streak, toast, or notification proposal. | PLAN secs. 1, 10, 12: no achievements, streaks, scores, badges, XP, ranks, tiers, or leaderboard substrate. | no | partial | Recovered absolute refusal and scope-creep foothold; missed the engagement-metric temptation layer. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | Monotonic row: absence applies zero delta, avoiding Tamagotchi-style decay. | PLAN secs. 1, 5, 12: no death/hunger/distress/decay; absence is never punished. | no | full | No-punishment/non-custodial mechanics recovered. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | PLAN secs. 1, 11: two starter birds are selected by the system and the user can name/rename birds. | yes | none | Starter-bird rule survived, not the arrived-animals-not-avatar-catalog rationale. |
| F30 | age-based-bird-offers | 4 | yes | included | Bird ramp row: starts at two and uses age-based offers; growth is time, not game mechanics. | PLAN secs. 1, 11: additional birds unlock by aviary age only; ramp is time, not feature flags. | no | partial | Recovered age/time growth; missed visit-score/paid-tier rejection and economy erosion. |
| F31 | stable-bird-identity | 4 | yes | included | none | PLAN sec. 3: Bird id is a stable UUID that is never regenerated. | yes | none | RECONSTRUCTION marks this exact feature NOT RECOVERABLE FROM PLAN. |
| F32 | mood-persists-across-sessions | 2 | yes | included | Mood row: mood persists across sessions and advances during absence. | PLAN sec. 5: mood persists across sessions; tick advances mood timers during absence. | no | full | Mood continuity rationale recovered through continued-while-absent framing. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | Field notebook row: entries observe the aviary rather than the user; scope says read-only. | PLAN secs. 1, 5: field notebook is read-only and entries observe aviary events only. | no | full | Observer-record purpose is recovered. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | PLAN secs. 1, 4: account export emails a JSON link. | yes | none | RECONSTRUCTION marks account export NOT RECOVERABLE FROM PLAN. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | none | PLAN secs. 1, 4: account delete and recover endpoints; soft-delete 30 days then hard-delete. | yes | none | RECONSTRUCTION marks deletion rationale NOT RECOVERABLE FROM PLAN. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | Aggregate-only RUM row: useful observability without per-bird state or per-account dimensions. | PLAN sec. 10: aggregate load/frame/error/tick metrics; no per-bird state or per-account dimensions. | no | full | Technical telemetry boundary recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | none | PLAN secs. 1, 4: per-invite opt-in visit invitations with encrypted visitor email and revocation. | yes | none | Invite mechanism survived without the private-relationship/not-publishing rationale. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | PLAN secs. 1, 4: silent visit log and visit-log endpoint are present. | yes | none | Visit-log mechanism survived without transparency-not-social-loop rationale. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | PLAN sec. 4: visitor snapshot endpoint returns render-only host snapshot. | yes | none | Actual-snapshot mechanism survived, not the no-show-off/marketing-rendering rationale. |
| F40 | narration-cadence-slow | 4 | yes | included | Narration cadence row: about 30-60s at idle keeps narration sparse; priority bumps remain observation-voiced. | PLAN sec. 9: narration updates every 30-60s, with priority bump for user-initiated events. | no | partial | Recovered slow sparse rhythm and priority-bound exception; missed screen-reader queue overwhelm. |

Multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | no | yes |
| F2 | yes | yes | yes |
| F3 | yes | yes | no |
| F4 | yes | no | yes |
| F7 | yes | no | yes |
| F9 | yes | yes | no |
| F10 | no | no | no |
| F12 | yes | no | yes |
| F13 | yes | no | yes |
| F14 | yes | no | yes |
| F15 | yes | no | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | no |
| F18 | yes | no | yes |
| F20 | yes | yes | yes |
| F24 | yes | no | yes |
| F25 | yes | no | yes |
| F27 | yes | no | yes |
| F30 | yes | no | no |
| F31 | no | no | no |
| F35 | no | no | no |
| F40 | yes | no | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS.json |
| Possible total weight | 152 | Full benchmark denominator |
| Reachable gold whys | 49 | S whys always included; all F anchors captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 77 / 152 | Sum of weight times recovery over included whys |
| Whys with reconstruction evidence | 34 | Rows with rationale evidence in frozen reconstruction |
| Whys with PLAN grounding | 34 | Rows with same-rationale PLAN grounding |
| `rule_without_why` cases | 15 | Mechanism survived without rationale |
| `plan_only_not_reconstructed` cases | 20 | PLAN carried more rationale/layers than reconstruction |
| `ungrounded_reconstruction` cases | 0 | Reconstruction did not appear to assert scorer-side unsupported whys |

### 2.5. Failure groupings

| Grouping | Total reachable | Recovered (full + partial-weighted) | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 weighted denominator | 32.0.0 | 59.3% |
| Affective whys | 98 weighted denominator | 45.0.0 | 45.9% |
| Weight-2 whys | 44 weighted denominator | 19.0.0 | 43.2% |
| Weight-3 whys | 108 weighted denominator | 58.0.0 | 53.7% |
| System-level whys | 28 weighted denominator | 23.0.0 | 82.1% |
| Feature-level whys (reachable) | 124 weighted denominator | 54.0.0 | 43.5% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered better: 32/54 weighted (59.3%) versus affective whys at 45/98 (45.9%). The plan and reconstruction both preserved architecture and privacy more crisply than social/relationship tone.
- **Weight-3 vs weight-2.** Weight-3 whys recovered slightly better (58/108, 53.7%) than weight-2 whys (19/44, 43.2%) because high-weight engine/sync/accessibility structures were explicit.
- **System-level vs feature-level.** System-level fidelity was strong at 82.1%, but feature-level fidelity fell to 43.5%. The plan's worldview survived better than the feature-specific rationales.
- **Multi-layer recovery.** Primary mechanisms usually survived. Secondary temptation analysis and downstream affective consequences were the most common missing layers.
- **Subdomain patterns.** Simulation, sync, privacy, and performance were strongest. Social optionality, no-announcement surfaces, account export/deletion, and starter-bird identity/intimacy rationales were weakest.
- **Evidence-bound effects.** Several v1-style apparent recoveries were denied: F5, F6, F8, F10, F19, F22, F23, F29, F31, F34, F35, F37, F38, and F39 kept rules without the gold why.

The failure shape suggests that the candidate plan is excellent as an implementation handoff but less reliable as a rationale-preserving artifact when the reason is affective, exception-shaped, or social-boundary-specific.

---

## 4. Recommendations for v2 hardening

- Keep the evidence-bound ledger. It cleanly separated feature capture from rationale recovery in a run where implementation coverage was high.
- Add or retain targeted exception whys around social surfaces, notifications, export/deletion, and identity continuity. These were the easiest to implement while compressing away why.
- Continue using multi-layer high-weight whys. The layer split showed whether the candidate preserved mechanism, temptation, and consequence separately.
- Clarify scoring guidance for document/support features such as glossary and privacy-policy link. They can be missed by implementation-heavy plans even when product behavior is otherwise covered.
- Keep the system-level cross-cutting appendix. It is the best guard against over-crediting a design-principle sentence that does not shape the plan.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** This scorer used only the allowed phase_two package, assigned plan, assigned metadata, runs.001 timing, and frozen reconstruction. The reconstruction was not modified.
- **Single-run limitation.** This is one sampled run; no variance estimate is available from this report alone.
- **Borderline capture calls.** Bird naming, no welcome-back toast, settle optionality, and visitor actual-aviary rendering were counted inclusively; none materially changes the main diagnosis.
- **System-level cross-cutting.** S2, S4, and S6 required the most judgment. S2 and S6 were partial because important affective/downstream layers did not survive; S4 was partial because the plan preserved restraint but the reconstruction did not identify it as system-level intent.
- **Confabulation cases.** I found no major ungrounded reconstruction assertions. The validity audit verdict is PASS.
- **Evidence-bound denials.** Fifteen included whys were marked rule_without_why. These mostly explain why feature-level fidelity is much lower than planning quality.
- **Timing.** TIMING.json exposed phase1 and phase2a for run 001 only. Phase2b timing was unavailable at scoring time, so only bounded available timing fields are included in run_001.json.

---

End of report.
