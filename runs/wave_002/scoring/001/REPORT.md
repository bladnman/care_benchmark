# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Every S1-S9 and F1-F40 row below separates denominator status from recovery and cites frozen reconstruction evidence plus PLAN grounding where available.

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **98.3%** |
| Intent fidelity | **53.3%** |
| Combined quality | **9787** |

**Diagnostic split:**

- System-level fidelity: **67.9%**
- Feature-level fidelity: **50.0%**

**(Planning, fidelity) coordinate:** `(98.3, 53.3)` - plot on a 2D scatter with both axes 0-100; upper-right is best.


### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-24T06:07:09Z |
| Candidate model | gpt-6-luna |
| Candidate effort | max |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **118 / 120** = **98.3%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 20 | 100.0% |
| aviary_layout.md | 18 | 17 | 94.4% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **118** | **98.3%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured in PLAN. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured in PLAN. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured in PLAN. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured in PLAN. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in PLAN. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured in PLAN. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Borderline captured: no glossary artifact, but domain concepts are defined throughout the plan. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured in PLAN. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured in PLAN. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured in PLAN. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured in PLAN. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured in PLAN. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured in PLAN. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured in PLAN. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured in PLAN. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured in PLAN. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured in PLAN. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured in PLAN. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured in PLAN. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured in PLAN. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Borderline captured: idle actions were not enumerated as preen/scan/head-tilt/shuffle, but pose/action and mood-shaped motion are planned. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured in PLAN. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured in PLAN. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured in PLAN. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured in PLAN. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured in PLAN. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured in PLAN. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured in PLAN. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured in PLAN. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured in PLAN. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured in PLAN. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured in PLAN. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured in PLAN. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured in PLAN. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured in PLAN. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured in PLAN. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured in PLAN. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured in PLAN. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured in PLAN. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured in PLAN. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured in PLAN. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured in PLAN. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured in PLAN. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured in PLAN. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured in PLAN. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured in PLAN. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured in PLAN. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured in PLAN. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured in PLAN. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured in PLAN. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured in PLAN. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured in PLAN. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured in PLAN. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured in PLAN. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured in PLAN. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured in PLAN. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured in PLAN. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | Not captured: plan did not specify the night-state/nightjar behavior. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured in PLAN. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured in PLAN. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured in PLAN. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured in PLAN. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured in PLAN. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured in PLAN. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured in PLAN. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured in PLAN. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured in PLAN. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured in PLAN. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Borderline captured: calm/naturalist palette is preserved, while UI accent exclusions are implicit. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured in PLAN. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured in PLAN. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured in PLAN. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured in PLAN. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured in PLAN. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured in PLAN. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured in PLAN. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured in PLAN. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured in PLAN. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured in PLAN. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured in PLAN. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured in PLAN. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured in PLAN. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured in PLAN. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured in PLAN. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured in PLAN. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured in PLAN. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | Not captured: privacy policy link in settings was not planned. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured in PLAN. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured in PLAN. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured in PLAN. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured in PLAN. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured in PLAN. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured in PLAN. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured in PLAN. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured in PLAN. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured in PLAN. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured in PLAN. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured in PLAN. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured in PLAN. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured in PLAN. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured in PLAN. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured in PLAN. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured in PLAN. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured in PLAN. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured in PLAN. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured in PLAN. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured in PLAN. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured in PLAN. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured in PLAN. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured in PLAN. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured in PLAN. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured in PLAN. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured in PLAN. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured in PLAN. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured in PLAN. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured in PLAN. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured in PLAN. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured in PLAN. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured in PLAN. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured in PLAN. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **67.9%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECON System 2: "feel continuing and alive between visits"; tick runs without clients; first frame has motion; avoids spinner/wake-up. | PLAN sections 1, 5, 6: "feel as though it continues between visits"; tick runs without clients; "ambient motion already in progress". | yes | yes | no | partial | Recovered the continuing-place and cross-layer aliveness mechanisms, but not the downstream staleness/accompaniment consequence. |
| S2 - notice-never-announce | 4 | included | RECON System 5/8: no welcome toast or banner; visit notifications default off; no badge or unsolicited message. | PLAN sections 1, 4, 10: no reward loop, no welcome toast/banner, default-off visit notifications, no badge, no engagement scores. | yes | yes | yes | partial | The no-announcement rule survived, but the processed-vs-seen rationale and cumulative-toast failure were not reconstructed. |
| S3 - charm-from-specificity | 2 | included | RECON System 5: product surfaces use "specific lowercase naturalist prose" and avoid generic rows, trait deltas, attendance, and streak phrasing. | PLAN sections 4, 5, 8: naturalist prose for product surfaces; notebook from actual facts; no generic event rows or trait deltas. | yes | yes | no | full | Specific naturalist observation survived as a cross-surface product voice principle. |
| S4 - restraint-over-richness | 2 | included | none | PLAN sections 1, 6, 7: up to seven birds in one horizontal scene; no in-scene chrome; compact 2D renderer and recognizability gates. | no | yes | no | partial | The plan preserved restraint, but the reconstruction reframed it mainly as compact performance rather than depth-of-character over richness. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECON System 5: product-facing copy uses naturalist prose; failures use plain system language. | PLAN sections 4, 8, 9: failures use plain system language; narration, reactions, captions, and notebook use naturalist prose. | yes | yes | no | full | The voice split between charm surfaces and system/error surfaces is clearly recovered. |
| S6 - presence-is-real-interaction | 4 | included | RECON System 1: presence is primary, not click volume; Simulation: visible, focused, recent activity; lease union prevents accelerated drift. | PLAN sections 1, 5: presence is primary; presence lease requires visible, focused, recent pointer/key; overlapping leases are unioned. | yes | yes | no | partial | Recovered the honest-presence signal and inflation risk, but not the full settle/tab-close equivalence consequence. |
| S7 - simulation-runs-server-side | 4 | included | RECON System 3: server owns canonical reality; clients render snapshots and submit bounded events; versions, watermarks, idempotency. | PLAN sections 2, 5: server authoritative state, tick workers, event watermark, compare-and-swap, no last-write-wins, two-device checks. | yes | yes | no | partial | Canonical-server architecture and sync coherence survived; the two-client divergent-simulation failure mode was mostly absent. |
| S8 - privacy-first-on-bird-data | 2 | included | RECON System 6: privacy and data minimization are cross-cutting; telemetry is apart from simulation; interaction records stay owner-simulation only. | PLAN sections 2, 9: telemetry physically/logically apart; collector has no read path to bird records; interactions never enter analytics warehouse. | yes | yes | no | full | Recovered both the privacy principle and the technical data-pipeline boundary. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECON System 7: accessibility ships in v1, uses the same snapshot/world model, and audio/accessibility acceptance happens before release. | PLAN sections 8, 10: ship accessibility in v1; narration/captions from same snapshot/grammar; reduced motion keeps calls and simulation; acceptance before release. | yes | yes | no | full | Recovered same-product accessibility, designed alternate surfaces, and the launch-with-v1 consequence. |

Multi-layer system-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | no |
| S2 | yes | no | no |
| S6 | yes | yes | no |
| S7 | yes | yes | no |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix (RUBRIC section 5.1).**

- S1: PLAN sections 1, 5, 6: "feel as though it continues between visits"; tick runs without clients; "ambient motion already in progress".
- S2: PLAN sections 1, 4, 10: no reward loop, no welcome toast/banner, default-off visit notifications, no badge, no engagement scores.
- S3: PLAN sections 4, 5, 8: naturalist prose for product surfaces; notebook from actual facts; no generic event rows or trait deltas.
- S4: PLAN sections 1, 6, 7: up to seven birds in one horizontal scene; no in-scene chrome; compact 2D renderer and recognizability gates.
- S5: PLAN sections 4, 8, 9: failures use plain system language; narration, reactions, captions, and notebook use naturalist prose.
- S6: PLAN sections 1, 5: presence is primary; presence lease requires visible, focused, recent pointer/key; overlapping leases are unioned.
- S7: PLAN sections 2, 5: server authoritative state, tick workers, event watermark, compare-and-swap, no last-write-wins, two-device checks.
- S8: PLAN sections 2, 9: telemetry physically/logically apart; collector has no read path to bird records; interactions never enter analytics warehouse.
- S9: PLAN sections 8, 10: ship accessibility in v1; narration/captions from same snapshot/grammar; reduced motion keeps calls and simulation; acceptance before release.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **50.0%**.

Reachable feature-level whys: **40 / 40** (no feature-level why anchors were excluded).

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECON Simulation: presence active only while visible, focused, and recently moved or typed; limits overstated/spoofed presence. | PLAN section 5: browser reports presence only when visible, focused, and recent pointer/key activity hold; no backfill. | no | partial | L1 recovered; individual-signal misses and silent population drift failure were not reconstructed. |
| F2 | drift-function | 4 | yes | included | RECON Simulation: low-pass drift makes personality gradual; one week measurable, three weeks visible. | PLAN section 5: drift is a low-pass filter; calibration targets one week measurable and three weeks visible. | no | partial | Recovered slow filter and calibration; missed Tamagotchi/screensaver failure-band rationale. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECON Simulation: bounded non-negative trait deltas; absence makes birds quieter/ambient, never hungry, distressed, angry, dead, or less colorful. | PLAN sections 5, 11: bounded non-negative deltas; absence never decreases traits; return after absence gives quieter birds, not punished birds. | no | partial | Recovered monotonic/non-punitive behavior and return-from-absence consequence; missed symmetric-drift/Tamagotchi temptation as the explicit exception. |
| F4 | procedural-call-grammar | 4 | yes | included | RECON Procedural audio: motif grammars keep calls fresh and recognizable; chorus mixer avoids identical loops. | PLAN section 7: per-species motif grammars, stable seeded signatures, constrained variation, client synthesis, no recorded-call download. | no | partial | Fresh procedural audio and chorus rationale survived; the audio-as-affective-spine cascade did not. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | rule only: RECON Simulation says mood drives visible action and perch choice. | PLAN section 5 says mood drives visible action and perch choice "rather than a meter or label". | yes | none | plan_only_not_reconstructed: plan carried the no-label rationale, but reconstruction kept only the mechanism. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECON Product/Adoption: cap supports recognizability and performance; do not expand beyond seven in v1. | PLAN sections 5, 10, 11: cap remains seven; validate recognizability/performance before more birds; calls can become indistinguishable. | no | full | Recovered the recognizable-per-bird ceiling behind the cap. |
| F7 | vector-persistence | 4 | yes | included | RECON Data model: stable UUIDs persist through drift/rename/export; hidden vector affects expression; calibration changes avoid resetting identity. | PLAN sections 2, 3, 5: server owns personality vectors; hidden vector is stored on Bird; versioned calibration avoids resetting bird identity. | no | partial | Recovered server-owned persistent state at a high level, but not vector loss as deleting the known bird or the no-LWW downstream. |
| F8 | vector-never-shown-numerically | 2 | yes | included | rule only: RECON Data model says raw numbers stay out of product UI and telemetry. | rule only: PLAN section 5 says keep numbers out of visual controls and product copy. | yes | none | Mechanism survived, but not the stat-management relationship rationale. |
| F9 | return-greeting | 4 | yes | included | RECON Simulation: one likely bird chosen by boldness, mood, absence; staggered others; no unison welcome. | PLAN section 5: return greeting chooses one likely bird using boldness, mood, absence; staggered greetings; procedural variation. | no | partial | Recovered selection/variation; missed the notice-never-announce payoff and generic-arrival-animation failure. |
| F10 | no-welcome-back-toast | 4 | yes | included | rule only: RECON API says no welcome toast or banner appears on host return. | rule only: PLAN section 4 says no welcome toast or banner appears on host return. | yes | none | The rule survived, but no layer of the bird-greeting-as-welcome rationale was reconstructed. |
| F11 | settle-is-opt-in | 2 | yes | included | rule only: RECON Frontend says settle is in the top bar with evening shift and undo click. | rule only: PLAN sections 5, 6 say tab close/pagehide ends presence; settle closes presence and has undo. | yes | none | Mechanics survived, but not the chore/punishment rationale for making settle optional. |
| F12 | field-notebook-prose | 4 | yes | included | RECON Simulation: notebook prose from approved templates and actual events; sparse, useful, naturalist, never attendance. | PLAN section 5: naturalist templates/facts tied to actual bird events; sparse; never generic rows, trait deltas, or attendance. | no | partial | Recovered naturalist prose and sparsity; missed notebook as the concentrated voice surface whose generic treatment breaks the spell. |
| F13 | presence-accounting | 4 | yes | included | RECON Simulation: visible, focused, recent activity limits overstated/spoofed presence; quiet watching counts. | PLAN sections 5, 11: all three conditions; no arbitrary backfill; prevent overstated/spoofed presence and accelerating drift. | no | partial | Recovered conjunction and approximation rationale; missed the tab-open silent-corruption population failure. |
| F14 | no-streak-counter | 4 | yes | included | RECON Product: no badge, visit calendar, dashboard, or visit-frequency copy; protects no reward loop and attendance reporting. | PLAN sections 1, 4, 5: no streaks, scores, visit-frequency observations, activity dashboard, or attendance praise. | no | partial | Recovered refusal and adjacent disguises; missed the cumulative user-intention rotation into managing a number. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECON Frontend: first frame uses current pose/animation phase; ambient motion in progress; no fade, spinner, or wake-up; quiet sky field. | PLAN section 6: first meaningful frame uses server pose and animation phase; no fade/spinner/wake-up; quiet sky field with faint motion. | no | full | Recovered continuing-first-frame, server snapshot implementation, and spinner-as-machine fallback avoidance. |
| F16 | synthetic-account-id | 4 | yes | included | RECON Data/Security: records are account-scoped by synthetic UUID; email encrypted and never partition/log identifier. | PLAN sections 3, 9: synthetic UUIDs for account scope; email encrypted; account identifiers in logs are UUIDs, never email. | no | partial | Recovered UUID-vs-email and PII leakage risk; missed retrofit impossibility as the downstream reason. |
| F17 | server-side-sim-tick | 4 | yes | included | RECON Simulation: one-minute tick for every aviary, including no connected clients; ordered events commit one canonical snapshot/version. | PLAN section 5: tick runs for every aviary; consumes ordered owner events; commits canonical snapshot/version; clients render snapshots. | no | partial | Recovered server tick and canonical sync shape; missed explicit two-client simulation-collapse consequence. |
| F18 | no-last-write-wins | 4 | yes | included | RECON Architecture/API: clients cannot mutate canonical state; client never sends trait values or full state; avoid last-write-wins replacement. | PLAN sections 2, 4, 5: client never sends trait values; only server tick writes vectors; append-only events; no last-write-wins state replacement. | no | partial | Recovered the implementation rule; missed the concrete laptop/phone overwrite failure. |
| F19 | sync-conflict-tone | 2 | yes | included | rule only: RECON API says plain system language for failures, distinct from product narration. | rule only: PLAN section 4 says magic-link, session, invite, and export failures use plain system language. | yes | none | Recovered voice split, but not the evasive-naturalist-error rationale. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECON Security: interaction records excluded from training, recommendations, population analysis, third-party sharing, and analytics warehouse. | PLAN sections 3, 9: events stored only for owner simulation; never model training, recommendation, population bird analysis, third-party sharing, or analytics. | no | partial | Recovered use prohibition and technical boundary; missed relationship-as-private-data-product rationale. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECON Simulation/Security: visitors render current state without greeting, event submission, presence credit, or host-state effects. | PLAN sections 4, 9: visitor sessions are read-only snapshot access; visitor data is not presence or drift input. | no | full | Recovered visit as observation, not host-affecting co-presence. |
| F22 | no-friend-visited-notification | 2 | yes | included | rule only: RECON Product says visit notifications default off; no visit badge or unsolicited default message. | rule only: PLAN section 1 says notification preference default off; no visit badge or unsolicited message by default. | yes | none | The default-off rule survived, but not the attention-driver/social-loop rationale. |
| F23 | no-leaderboards | 2 | yes | included | rule only: RECON Social says no public profiles or discovery indexes; not a social discovery surface. | rule only: PLAN sections 1, 3 say no social discovery/profiles/follows/comments, leaderboards, public profiles, or discovery indexes. | yes | none | Recovered public-surface refusal, but not the comparison-changes-relationship rationale. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECON Accessibility: screen-reader naturalist narration every 30-60s; no coordinates, mood codes, or personality numbers; same snapshot/world model. | PLAN section 8: running naturalist narration from same snapshot; no coordinate/mood-code/personality-number announcements. | no | partial | Recovered prose-not-state-list and same-world-model parity; missed ARIA-automation-is-wrong-feature consequence. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECON Accessibility: slow cross-fades; remove drifting leaves; calls and simulation continue. | PLAN section 8: reduced motion renders slow cross-fades, removes leaf drift, slows palette shifts, while calls and simulation continue. | no | partial | Recovered alternate rendering and preserved simulation/audio; missed stripped-fallback-as-exclusion consequence. |
| F26 | ttfb-500ms | 2 | yes | included | rule only: RECON Performance says first bird visible under 500 ms; paint first bird without non-critical assets. | rule only: PLAN section 8 says first bird visible under 500 ms; payload does not wait for non-critical assets. | yes | none | Performance target survived, but not the affective-performance bridge rationale. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECON System/Product: no reward loop; no streaks, scores, badges, visit calendars, activity dashboards, or engagement-score rollout. | PLAN sections 1, 10: no achievements, streaks, scores, badges, public comparison, or engagement loops; rollout uses aggregate health only. | no | partial | Recovered explicit refusal; missed temptation/engagement-metric and foothold-to-different-product layers. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECON Simulation: absence leaves birds quieter/ambient, never hungry, distressed, angry, dead, or less colorful; avoid guilt-inducing drift. | PLAN sections 5, 11: absence can leave birds quieter/ambient, never hungry/distressed/angry/dead; no-decrement absence invariant. | no | full | Recovered observational, non-custodial, non-punitive mechanics. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECON Data model: starter adoption uses server-selected species; small species pool has no rarity or catalog-selection mechanic. | PLAN sections 3, 5: starter adoption creates two server-selected species; species pool has no rarity or catalog-selection mechanic. | no | full | Recovered the refusal to turn first birds into a catalog/optimization choice. |
| F30 | age-based-bird-offers | 4 | yes | included | RECON Adoption: thresholds independent of sessions/attention; additions must not become reward loop or monetized collection path. | PLAN sections 3, 5, 10: opportunities depend only on aviary age, not attention/offers/paid tier; quiet optional flow; no reward loop. | no | partial | Recovered age-only and anti-reward rationale; missed engine-becomes-economy erosion consequence. |
| F31 | stable-bird-identity | 4 | yes | included | RECON Data model: stable bird UUIDs persist through drift, rename, export, and replay invariants. | PLAN sections 3, 5: Bird has stable UUID; rename updates only name; invariants cover stable IDs and calibration without resetting identity. | no | partial | Recovered invariant identity beyond name; missed retroactive evaporation of the remembered relationship if reset/swapped. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECON Product/Simulation: daily boundary reevaluates, not reset to neutral; mood transition persists across sessions. | PLAN sections 1, 5: daily boundary triggers reevaluation, not reset; mood persists across sessions and is not reset. | no | full | Recovered mood continuity as protection against visible neutral reset. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECON Data model: "No notebook edit/delete UI: NOT RECOVERABLE FROM PLAN". | PLAN section 5: notebook prose comes from approved observations/facts rather than editable feed rows. | yes | none | plan_only_not_reconstructed: B explicitly marked the why not recoverable, so this scores none. |
| F34 | account-export-relationship-copy | 2 | yes | included | rule only: RECON API says export is explicit and includes vectors, moods, notebook, and settings. | rule only: PLAN section 4 says export includes birds, names, vectors, moods, notebook, and settings. | yes | none | Recovered export contents/mechanics, but not relationship-ownership or quiet-quality-of-life rationale. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECON API/Security: recoverable deletion for 30 days, then hard-removes account-linked state; expirations match the deletion promise. | PLAN sections 4, 9: deletion recoverable for 30 days, then hard-deletes birds, vectors, events, notebook, invites, sessions, backups, and artifacts. | no | partial | Recovered hard-delete/privacy and whole-state deletion; missed soft-delete as regret protection for a relationship. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECON Architecture/Security: collector accepts aggregate operations data and has no read path to bird records or interaction logs. | PLAN sections 2, 9: telemetry only aggregate request counts, latencies, errors, timings, anonymous histograms; no bird/account event payloads. | no | full | Recovered technical telemetry boundary against observability back doors. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECON API/Social: named visit invitations are narrow, revocable, expiring, and not a social discovery surface. | PLAN sections 3, 4, 9: invite one named email; no public profiles/discovery; read-only visitor credentials and revocation checks. | no | full | Recovered deliberate per-invite sharing and anti-discoverability. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECON Data/Product: visit log supports visit transparency; no badge or unsolicited default message. | PLAN sections 3, 4: visit invitation/log stores minimal visit records; no visit badge or default unsolicited message. | no | full | Recovered log-as-transparency without attention-driving notification. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | rule only: RECON Simulation says visitor renders the host current state without host-arrival greeting or presence credit. | rule only: PLAN section 5 says visitors render the host current state without host-arrival greeting, event submission, or presence credit. | yes | none | Actual-state rendering survived, but not the witness-real-birds/no-show-off-mode rationale. |
| F40 | narration-cadence-slow | 4 | yes | included | RECON Accessibility: narration every 30-60 seconds at idle, prompt after actions; queued updates do not overlap or chatter. | PLAN section 8: narration roughly every 30-60 seconds at idle, promptly after actions, queue updates so they do not overlap or chatter. | no | full | Recovered slow rhythm, queue-overwhelm avoidance, and user-event priority without idle chatter. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | no | no |
| F2 | yes | yes | no |
| F3 | yes | no | yes |
| F4 | yes | yes | no |
| F7 | yes | no | no |
| F9 | yes | yes | no |
| F10 | no | no | no |
| F12 | yes | no | yes |
| F13 | yes | yes | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | no |
| F18 | yes | no | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | no |
| F25 | yes | yes | no |
| F27 | yes | no | no |
| F30 | yes | yes | no |
| F31 | yes | yes | no |
| F35 | no | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON gold_why_totals |
| Possible total weight | 152 | From intent_recovery.total_possible_weight |
| Reachable gold whys | 49 | S whys always included; all F whys reachable here |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 81.0 / 152.0 | Sum of weight x recovery-score over included whys |
| Whys with reconstruction evidence | 37 | Rationale evidence, excluding rule-only rows |
| Whys with PLAN grounding | 40 | Rationale grounding, excluding rule-only rows |
| rule_without_why cases | 12 | What survived without why |
| plan_only_not_reconstructed cases | 3 | PLAN carried rationale, reconstruction did not |
| ungrounded_reconstruction cases | 0 | Reconstruction asserted rationale not grounded in PLAN |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weighted | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 28.0 | 51.9% |
| Affective whys | 98.0 | 53.0 | 54.1% |
| Weight-2 whys | 44.0 | 23.0 | 52.3% |
| Weight-3 whys | 108.0 | 58.0 | 53.7% |
| System-level whys | 28.0 | 19.0 | 67.9% |
| Feature-level whys (reachable) | 124.0 | 62.0 | 50.0% |

---

## 3. Diagnostic patterns

**Headline pattern:** planning was near-complete, but fidelity was evidence-limited. The PLAN captured 118/120 features, yet many reconstructed rows preserved only the rule or mechanism. This is clearest in F8, F10, F11, F19, F22, F23, F26, F34, and F39.


**Affective vs functional:** affective recovery was only slightly stronger by weight (53.0/98.0) than functional recovery (28.0/54.0). The planner carried strong affective rules, but the reconstruction often lost second-order relationship rationales such as comparison pressure, announcement pressure, and show-off mode.


**Weight-3 vs weight-2:** higher-weight multi-layer whys recovered at 58.0/108.0, while weight-2 whys recovered at 23.0/44.0. Primary mechanisms usually survived; downstream failure modes were the most common loss.


**System vs feature:** system fidelity (19.0/28.0) exceeded feature fidelity (62.0/124.0). The reconstruction named the major principles, but per-feature why rows frequently compressed deep rationale into build instructions.


**Evidence-bound effect:** v06 denied several plausible v1-style recoveries because exact rationale evidence was absent. S4 was plan-only; F5 and F33 had plan rationale that B did not reconstruct; 12 rows were marked rule_without_why.

## 4. Recommendations for v2 hardening

For v2, keep the feature-level headroom additions: this run shows they discriminate well even when planning quality is saturated. F29-F40 produced a mix of full, partial, and none recoveries rather than all passing.


Add more tests for downstream-consequence layers. The most frequent multi-layer loss was L3: examples include S1, S2, S6, S7, F2, F9, F16, F17, F24, F25, F30, and F31.


Consider sharpening instructions around rule_without_why evidence. This run had many rows where the plan and reconstruction plainly preserved the product rule, but the scorer still had to deny recovery because the relationship rationale was absent.


The system-level cross-cutting bar was workable, but S4 shows a recurring ambiguity: compact performance and restraint can look similar in implementation while carrying different product intent.

## 5. Methodology caveats

**Fresh-context fidelity:** the scorer had fresh context for this turn, and the frozen reconstruction showed no clear contamination signs. The audit verdict is PASS.


**Single-run limitation:** this is one candidate plan and one reconstruction, so the score has no within-model variance estimate.


**Borderline capture calls:** features 7, 21, and 69 were counted as captured under the rubric's inclusive rule. The only missed features were 58 (nightjar-style night state) and 87 (privacy policy link in settings), so these calls did not materially change the conclusion.


**System-level subjectivity:** S4 was the finest call: the plan preserved restraint across multiple implementation decisions, but B did not identify restraint-over-richness as the principle, so it received partial credit.


**Operational issues:** TIMING.json had phase1 and phase2a entries for run 001 but no phase2b entry at scoring time; no run label was read because METADATA.json was outside the explicit allowlist.

---

End of report.
