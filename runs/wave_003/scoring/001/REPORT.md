# REPORT - CARE run 001

> Phase 2B evidence-bound scoring report for Pocket Aviary. The frozen reconstruction was read-only and left unchanged.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **12.5%** |
| Intent fidelity | **10.0%** |
| Combined quality | **1160** |

**Diagnostic split:**

- System-level fidelity: **14.3%**
- Feature-level fidelity: **0.0%**

**(Planning, fidelity) coordinate:** `(12.5, 10.0)`.

**LOW CONFIDENCE:** planning < 30% - fidelity reading is degenerate. Treat the fidelity number as diagnostic for this run.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-09T15:33:59Z |
| Candidate model | gemini-3.1-flash-lite-preview |
| Candidate effort | medium |
| Candidate harness | gemini-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **15 / 120** = **12.5%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 1 | 16.7% |
| concepts.md | 4 | 0 | 0.0% |
| bird_engine.md | 22 | 5 | 22.7% |
| interactions.md | 20 | 1 | 5.0% |
| aviary_layout.md | 18 | 0 | 0.0% |
| accounts_sync.md | 18 | 3 | 16.7% |
| social_optional.md | 10 | 0 | 0.0% |
| accessibility_perf.md | 18 | 4 | 22.2% |
| non_goals.md | 4 | 1 | 25.0% |
| **Total** | **120** | **15** | **12.5%** |

Per-feature detail:

| # | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | borderline inclusive call - Broad aviary/bird-evolution concept present in Scope. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | no | not addressed in the compact plan |
| 3 | "Notice, never announce" principle callout | product_brief.md | no | not addressed in the compact plan |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | no | not addressed in the compact plan |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | no | not addressed in the compact plan |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | no | not addressed in the compact plan |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | not addressed in the compact plan |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | no | not addressed in the compact plan |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | no | not addressed in the compact plan |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | no | not addressed in the compact plan |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | no | not addressed in the compact plan |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Names personality drift and a drift function over environmental/interaction inputs; low-pass detail absent. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | no | not addressed in the compact plan |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | no | not addressed in the compact plan |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Bird data model includes mood; fast-timescale/reset detail absent. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Drift function changes mood from environmental and interaction inputs. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Procedural call synthesis via WebAudio appears. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | no | not addressed in the compact plan |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | no | not addressed in the compact plan |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | no | not addressed in the compact plan |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Rendering pipeline includes CSS idle micro-motion. |
| 22 | Mood-shaped idle motion | bird_engine.md | no | not addressed in the compact plan |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | no | not addressed in the compact plan |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | no | not addressed in the compact plan |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | no | not addressed in the compact plan |
| 26 | Maximum 7 birds per aviary | bird_engine.md | no | not addressed in the compact plan |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | no | not addressed in the compact plan |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | no | not addressed in the compact plan |
| 29 | Mood persistence across sessions | bird_engine.md | no | not addressed in the compact plan |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | no | not addressed in the compact plan |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | no | not addressed in the compact plan |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | no | not addressed in the compact plan |
| 33 | Return-greeting on viewer arrival | interactions.md | no | not addressed in the compact plan |
| 34 | Greeting variation by absence length | interactions.md | no | not addressed in the compact plan |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | not addressed in the compact plan |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | no | not addressed in the compact plan |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | not addressed in the compact plan |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Listen-in named as an initial interaction surface. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | no | not addressed in the compact plan |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | no | not addressed in the compact plan |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | not addressed in the compact plan |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | no | not addressed in the compact plan |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | no | not addressed in the compact plan |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | not addressed in the compact plan |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | no | not addressed in the compact plan |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | no | not addressed in the compact plan |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | no | not addressed in the compact plan |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | no | not addressed in the compact plan |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | no | not addressed in the compact plan |
| 50 | No streak counter, no "days visited" display | interactions.md | no | not addressed in the compact plan |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | not addressed in the compact plan |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | not addressed in the compact plan |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | no | not addressed in the compact plan |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | no | not addressed in the compact plan |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | no | not addressed in the compact plan |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | no | not addressed in the compact plan |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | no | not addressed in the compact plan |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | not addressed in the compact plan |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | no | not addressed in the compact plan |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | no | not addressed in the compact plan |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | no | not addressed in the compact plan |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | no | not addressed in the compact plan |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | no | not addressed in the compact plan |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | no | not addressed in the compact plan |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | no | not addressed in the compact plan |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | no | not addressed in the compact plan |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | no | not addressed in the compact plan |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | not addressed in the compact plan |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | not addressed in the compact plan |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | no | not addressed in the compact plan |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | no | not addressed in the compact plan |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | not addressed in the compact plan |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | no | not addressed in the compact plan |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | no | not addressed in the compact plan |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Periodic server-side tick appears. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | borderline inclusive call - Authenticated state fetch covers snapshot pull only broadly; visibility trigger absent. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | no | not addressed in the compact plan |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Multi-device layer and canonical server state are named. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | no | not addressed in the compact plan |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | no | not addressed in the compact plan |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | no | not addressed in the compact plan |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | no | not addressed in the compact plan |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | no | not addressed in the compact plan |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | no | not addressed in the compact plan |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | no | not addressed in the compact plan |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | no | not addressed in the compact plan |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | not addressed in the compact plan |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | not addressed in the compact plan |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | no | not addressed in the compact plan |
| 90 | Visits default OFF for new accounts | social_optional.md | no | not addressed in the compact plan |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | no | not addressed in the compact plan |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | no | not addressed in the compact plan |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | no | not addressed in the compact plan |
| 94 | No "your friend visited!" notification by default | social_optional.md | no | not addressed in the compact plan |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | no | not addressed in the compact plan |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | no | not addressed in the compact plan |
| 97 | Visitor sees host aviary as it is (no special "show-off" mode) | social_optional.md | no | not addressed in the compact plan |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | no | not addressed in the compact plan |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | no | not addressed in the compact plan |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | no | not addressed in the compact plan |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | no | not addressed in the compact plan |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | no | not addressed in the compact plan |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | no | not addressed in the compact plan |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | borderline inclusive call - Captions for procedural calls are named; toggle/mood text absent. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | no | not addressed in the compact plan |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | no | not addressed in the compact plan |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | no | not addressed in the compact plan |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Bundle limit is stricter than gold threshold. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | no | not addressed in the compact plan |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | 60fps UI target named. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | no | not addressed in the compact plan |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | WebAudio procedural synthesis implies client-side procedural audio. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | no | not addressed in the compact plan |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | no | not addressed in the compact plan |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | no | not addressed in the compact plan |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | not addressed in the compact plan |
| 117 | Out of scope: native mobile app | non_goals.md | no | not addressed in the compact plan |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | no | not addressed in the compact plan |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | no | not addressed in the compact plan |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | borderline inclusive call - Complex social features out of scope; specific profile/follow/feed surfaces absent. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **14.3%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | B identified? | PLAN cross-cutting? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md: "Responsive interaction and birdsong" and "Procedural call synthesis" (rule evidence only). | PLAN.md: "Procedural call synthesis using WebAudio API"; "CSS-based scene composition for idle micro-motion" (mechanisms only). | no | no | yes | none | Mechanisms that could support aliveness appear, but neither PLAN nor reconstruction articulates the continuing-place / non-stale rationale. |
| S2 - notice-never-announce | 4 | included | none | none | no | no | no | none | No bird greeting, no toast refusal, and no announcement-vs-noticing rationale survived. |
| S3 - charm-from-specificity | 2 | included | none | none | no | no | no | none | Notebook is reduced to interaction logs; naturalist specificity and named-bird charm are absent. |
| S4 - restraint-over-richness | 2 | included | none | none | no | no | no | none | The plan is scoped down, but not around the gold restraint rule of two birds, max seven, one calm screen, and per-bird recognizability. |
| S5 - naturalist-voice-with-system-exception | 2 | included | none | none | no | no | no | none | No naturalist voice, matter-of-fact system exception, or sync/error tone split appears. |
| S6 - presence-is-real-interaction | 4 | included | none | none | no | no | no | none | The label "presence events" appears, but not idle attention as interaction, the three-signal conjunction, or settle/close equivalence. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md: "Real-time, server-centered synchronization"; "Canonical server state pushed to all clients". | PLAN.md: "Periodic server-side tick"; "Canonical server state pushed to all clients via WebSockets". | yes | yes | no | partial | Server tick and canonical multi-device state survived; the no-divergent-client / no-last-write-wins failure rationale did not. |
| S8 - privacy-first-on-bird-data | 2 | included | none | none | no | no | no | none | No per-bird privacy boundary, data-pipeline separation, or aggregate-only telemetry claim appears. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md: "Accessible presence and calls are part of the product surface". | PLAN.md: "Full screen-reader support"; "Captions for all procedural calls". | yes | no | no | partial | The first-class-accessibility surface survived weakly; charm-preserving reduced-motion/captions and launch-timing rationale did not. |

Multi-layer system-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | no | no | no |
| S2 | no | no | no |
| S6 | no | no | no |
| S7 | yes | yes | no |
| S9 | yes | no | no |

**Cross-cutting evidence appendix:**

- S1: Procedural calls, idle micro-motion, and server tick are present, but the aliveness principle is not named or applied as a cross-cutting rationale; (b) no.
- S2: No greeting/no-toast/no-notification refusal surfaces appear; (b) no.
- S3: Notebook is only interaction logs and no naturalist/specificity voice is present; (b) no.
- S4: The plan is generally scoped down, but not around two birds, max seven, one screen, calm palette, or per-bird recognizability; (b) no.
- S5: No voice register split for product vs system surfaces; (b) no.
- S6: Presence events are named but not the idle-attention conjunction, drift dominance, or settle/close equivalence; (b) no.
- S7: Scope, Architecture, Communication, Simulation Engine Design, API Surface, and Sync Model all point to server-centered canonical state; (b) yes.
- S8: No privacy/data-pipeline boundary appears; (b) no.
- S9: Screen-reader support and captions are in the v1 plan, but only two inherited decisions and no charm-preserving reduced-motion surface; (b) no.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **0.0%**.

Reachable feature-level whys: **3 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md: "Drift function" calculates "mood change based on environmental and interaction inputs". | PLAN.md: "Drift function calculating mood change based on environmental and interaction inputs." | yes | none | Reachable as a rule, but no low-pass filter, one-week/three-week calibration, or Tamagotchi/screensaver rationale was reconstructed. |
| F3 | drift-monotonic-toward-expressive | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md: "Procedural call synthesis" uses WebAudio API "for responsive birdsong". | PLAN.md: "Procedural call synthesis using WebAudio API for responsive birdsong." | yes | none | Reachable as procedural WebAudio, but no no-loop/dead-software, chorus-artifact, or audio-spine rationale survived. |
| F5 | mood-shaped-idle-motion | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F6 | bird-count-cap-7 | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F7 | personality-vector-persistence | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F8 | personality-vector-never-numerical | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F9 | return-greeting | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F12 | field-notebook-auto-entries | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F13 | presence-accounting | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F14 | no-streak-counter | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F15 | scene-loads-with-motion | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F16 | synthetic-account-id | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md: "Periodic server-side tick" has frequency "configurable per performance budget". | PLAN.md: "Periodic server-side tick" and "Canonical server state pushed to all clients via WebSockets." | yes | none | Reachable as a server tick, but the feature-level why about canonical state, clients never owning state, and divergent-client failure was not recovered. |
| F18 | no-last-write-wins-personality | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F19 | sync-conflict-matter-of-fact | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F20 | no-per-bird-ml-telemetry | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F21 | visit-read-only-ambient | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F22 | no-friend-visited-notification | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F23 | no-leaderboards-no-discovery | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F24 | sr-narration-running-prose | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F25 | reduced-motion-mode | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F26 | time-to-first-bird-500ms | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F27 | no-gamification | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F28 | no-tamagotchi-mechanics | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F29 | starter-birds-not-catalog | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F30 | age-based-new-bird-offers | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F31 | stable-bird-identity | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F32 | mood-persists-across-sessions | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F33 | field-notebook-read-only-observer-record | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F34 | account-export-relationship-copy | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F35 | account-deletion-grace-then-hard-delete | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F36 | aggregate-telemetry-boundary | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F37 | per-invite-named-sharing | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F38 | visit-log-on-demand-transparency | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F39 | visitor-sees-actual-aviary | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |
| F40 | sr-narration-cadence-slow | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured by PLAN; excluded from fidelity denominator. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | unreachable | unreachable | unreachable |
| F2 | no | no | no |
| F3 | unreachable | unreachable | unreachable |
| F4 | no | no | no |
| F7 | unreachable | unreachable | unreachable |
| F9 | unreachable | unreachable | unreachable |
| F10 | unreachable | unreachable | unreachable |
| F12 | unreachable | unreachable | unreachable |
| F13 | unreachable | unreachable | unreachable |
| F14 | unreachable | unreachable | unreachable |
| F15 | unreachable | unreachable | unreachable |
| F16 | unreachable | unreachable | unreachable |
| F17 | no | no | no |
| F18 | unreachable | unreachable | unreachable |
| F20 | unreachable | unreachable | unreachable |
| F24 | unreachable | unreachable | unreachable |
| F25 | unreachable | unreachable | unreachable |
| F27 | unreachable | unreachable | unreachable |
| F30 | unreachable | unreachable | unreachable |
| F31 | unreachable | unreachable | unreachable |
| F35 | unreachable | unreachable | unreachable |
| F40 | unreachable | unreachable | unreachable |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS gold_why_totals |
| Possible total weight | 152 | Full-instance possible weight |
| Reachable gold whys | 12 | S whys always included; only 3 F whys reachable |
| Excluded unreachable feature whys | 37 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 4.0 / 40.0 | Weighted numerator / denominator |
| Whys with reconstruction evidence | 6 | Includes rule-only evidence where explicitly marked |
| Whys with PLAN grounding | 6 | Includes rule-only plan grounding where explicitly marked |
| rule_without_why cases | 4 | S1, F2, F4, F17 |
| plan_only_not_reconstructed cases | 0 | None counted |
| ungrounded_reconstruction cases | 0 | None counted |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 18.0 | 2.0 | 11.1% |
| Affective whys | 22.0 | 2.0 | 9.1% |
| Weight 2 whys | 8.0 | 0.0 | 0.0% |
| Weight 3 whys | 32.0 | 4.0 | 12.5% |
| System-level whys | 28.0 | 4.0 | 14.3% |
| Feature-level whys (reachable) | 12.0 | 0.0 | 0.0% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Both leaked badly. Functional whys recovered 2.0/18.0 weight via S7 only; affective whys recovered 2.0/22.0 via a weak S9 partial. Feature-level affective and functional whys both scored zero.
- **Weight-3 vs weight-2.** Weight-3 whys did slightly better only because S7/S9 partials are weight-3 system whys. All reachable feature-level weight-3 whys scored none.
- **System-level vs feature-level.** The plan preserved a small amount of system architecture intent, especially server-centered state, but did not preserve feature-level rationales. Feature-level fidelity is 0.0%.
- **Multi-layer recovery.** The first implementation layer was most likely to survive. Downstream failure-mode layers, such as S7 client divergence and F2 Tamagotchi/screensaver calibration, were consistently absent.
- **Subdomain patterns.** Simulation/audio/sync/performance were the only domains with material feature capture. Social, privacy, product voice, return greeting, layout, and most accessibility-charm surfaces were not captured.
- **Evidence-bound effects.** S1, F2, F4, and F17 show mechanism survival without why survival. V06 denied credit that a looser rubric might have granted for naming drift, procedural audio, and server ticks.

The failure shape suggests a planner that compressed a charm-heavy PRD into an engineering prototype skeleton. It remembered several nouns and mechanisms but lost most affective intent and nearly every exception that keeps Pocket Aviary from becoming a generic simulation app.

---

## 4. Recommendations for v2 hardening

- Preserve the evidence-bound rule-only distinction. This run is a clean example: mechanisms are present, but the whys that make them product-defining are absent.
- Keep the targeted F29-F40 headroom. The compact plan missed every targeted addition, so those rows continue to reveal planning compression.
- Add explicit reporting support for low-planning runs. With planning at 12.5%, feature-level denominator collapse makes the system/feature split useful but statistically fragile.
- Consider retaining a strict system-level cross-cutting bar. S9 shows why: two accessibility bullets should not become full credit for first-class, charm-preserving accessibility.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** This scoring pass used only the allowed phase-two files and assigned run artifacts; the frozen reconstruction was not modified.
- **Single-run limitation.** This is one run from wave_003, so it has no temperature or wave-level variance signal by itself.
- **Borderline capture calls.** I leaned inclusive on #1, #76, #104, and #120. Removing all four would lower planning from 12.5% to 9.2%, but would not change feature-level fidelity because none carry reachable recovered whys except denominator-neutral non-why rows.
- **System-level cross-cutting.** S7 is the only clear cross-cutting preservation. S9 was credited only partial because it has first-class accessibility hints but fails the three-decision charm-preserving bar.
- **Confabulation cases.** None were counted. The reconstruction mostly paraphrases the short PLAN and marks several items not recoverable.
- **Evidence-bound denials.** The main denials were F2, F4, and F17: all three features were reachable, but their gold rationales were absent in the frozen reconstruction.
- **Rule-without-why cases.** S1, F2, F4, and F17 were counted as rule/mechanism without why.

End of report.
