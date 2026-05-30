# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Every S1-S9 and F1-F40 row below records denominator status, reconstruction evidence, PLAN grounding, rule-without-why status, and recovery.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **82.5%** |
| Intent fidelity | **52.1%** |
| Combined quality | **8202** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **44.8%**

**(Planning, fidelity) coordinate:** `(82.5, 52.1)`

No low-confidence banner: planning quality is above 30%.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-30T12:40:29Z |
| Candidate model | gpt-5.4 |
| Candidate effort | medium |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---
## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **99 / 120** = **82.5%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 2 | 50.0% |
| bird_engine.md | 22 | 20 | 90.9% |
| interactions.md | 20 | 13 | 65.0% |
| aviary_layout.md | 18 | 12 | 66.7% |
| accounts_sync.md | 18 | 14 | 77.8% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **99** | **82.5%** |

Per-feature detail:

| # | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Scope states the browser product and core promise. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Core promise says the aviary continues without the viewer. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Core promise says it notices without announcing. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Naturalist notebook/narration and matter-of-fact controls are specified. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Out-of-scope list blocks gamification, Tamagotchi, and social-network surfaces. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Two starter birds, gradual expansion, and seven-bird cap appear. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No glossary-like artifact is planned. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | no | Presence exists, but the precise visibility/focus/recent-activity conjunction is not specified. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Personality vector, drift, mood state, and persistence are modeled. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Settle event, endpoint, state, and transition are planned. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Vector fields are enumerated. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Simulation computes additive personality drift using calibrated low-pass filters. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | One-week instrument and three-week user-perceptible targets are named. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Plan requires monotonic upward drift and no negative neglect. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Mood enum, timers, modifiers, and persistence are planned. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Daypart, weather, recent interactions, and personality shape mood. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | WebAudio runtime synthesis from species motif libraries is planned. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Call signature seeds and recognizability testing are planned. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Chorus emergence and mixing to preserve per-bird recognizability are planned. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Call timing and spacing are parameterized by vocal frequency and seeds. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Continuous idle micro-motion is included. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Motion strategy says continuous idle micro-motion is mood-shaped. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | no | Species exist, but no v1 pool size is planned. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | no | Bird records have names, but user assignment and renaming are not planned. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | borderline: Two starter birds are clear, but auto-selection/catalog refusal is only implicit. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Seven-bird cap and mix cap are planned. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Additional birds unlock by aviary age, not score. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Server is only writer; vectors are canonical server state. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Mood persists and never resets on tab open. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Local signal propagation, chorus, and social warmth are planned. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Bird has stable UUID and snapshots expose stable ids. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Personality values are hidden from product UI and accessible surfaces. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | borderline: The plan says birds notice the user and models who greets first, but lacks arrival timing. |
| 34 | Greeting variation by absence length | interactions.md | no | No absence-length variation rule appears. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | Greeting order is tied to social warmth, not boldness. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | no | No greeting-stagger rule appears. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | borderline: Captured from the explicit notice-without-announcing product promise. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Listen-in events and mix ramps are planned. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Listen-in attenuates others to ambient, not silence. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Offer endpoint, reactions, and cooldown are planned, though offer types are compressed. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Offers bias curiosity and boldness when the bird engages. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Server-side per-bird cooldown is planned. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Settle endpoint/state and lighting/audio transition are planned. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | Plan has settle but not close-tab equivalence or opt-in framing. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Naturalist lowercase sparse notebook entries are planned. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Sparsity gate targets one entry every few days. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Notebook entries are immutable after creation. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Presence accounting and presence-time drift inputs are planned. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | no | The three-signal qualification rule is absent. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Streaks, counters, and user-behavior dashboards are blocked. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | Visibility refresh exists, but explicit background-render pause is absent. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | Five-second undo exists, but click-anywhere behavior is absent. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | no | Responsive in-frame layout is planned, but no single-horizontal/no-panning rule. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Three perch zones with depth cues are planned. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Perch state is canonical and scene customization is out of scope. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Aviary timezone and snapshot daypart drive transitions. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | no | Daypart color/audio shifts are generic; evening-specific warm/quiet behavior is absent. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | No night-specific behavior is planned. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Weather windows and overlays are planned. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Weather provides baseline mood pressure. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Leaf/feather ornaments and ornamental drift are planned. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Subtle parallax is explicitly non-canonical ornament. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Top bar is sparse overlay with fade behavior. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | no | Top bar exists, but its full contents are not enumerated. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Top bar fade behavior is planned. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | First frame renders birds mid-action. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Initial shell delivers quiet field without spinner-first framing. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-aviary state is planned. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | Palette behavior is daypart-based, not a calm color spec. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Responsive layout preserves all birds in frame. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Magic-link auth APIs and email integration are planned. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | No expiry duration is specified. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Single-user account and one-to-one aviary are specified. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Synthetic UUID primary key and encrypted email are specified. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Once-per-minute server tick is planned. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Clients fetch snapshots on open and visibility regain. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Interpolation and smoothing are planned. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Server-canonical multi-device semantics are planned. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Client bird-state writes and merge UI are forbidden. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Only tick transaction updates personality and mood. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | no | Matter-of-fact voice exists generally, but no sync-conflict surface is planned. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Per-device revocable session metadata and revoke API are planned. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Account export API and scope are planned. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | borderline: Soft-delete and recovery are planned; 30-day/hard-delete specifics are absent. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Aggregate observability excludes per-bird and per-account relationship data. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Aggregate operational metrics and exclusions are explicit. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link is planned. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | No email-change flow is planned. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Email-based invitations and invite APIs are planned. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Launch plan keeps visits default-off. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Visit sessions are read-only render sessions. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Visitor surfaces cannot post interactions. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Shared/social profile and comment surfaces are explicitly out of scope. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | borderline: Captured from quiet-visit framing, but not as a named notification rule. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Invitation revoke API and tests are planned. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Visit log API is planned. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Visit uses the same snapshot read path; public showcase variants are blocked. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Discovery feeds, profiles, rankings, and public showcase variants are blocked. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Dedicated live-region naturalist narration is planned. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Slow cadence, rate limits, and queue protection are planned. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Narration is observational and naturalist, never stat-list based. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Reduced motion uses cross-fades between still poses. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Reduced motion keeps interaction affordances and notebook behavior identical. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Call captions are planned and persist per account. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | WCAG AA minimum is planned. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Full keyboard traversal is planned. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | High-contrast focus treatment is planned. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Bundle budget is specified. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | First-bird budget is specified. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | 60fps idle budget is specified. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Long-session memory budget and soak tests are planned. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | WebAudio synthesis and compact motifs are planned. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Silent mode with captions and no recorded fallback is planned. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Aggregate performance observability is planned. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Tick p99 budget and alarms are planned. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Browser matrix tests are planned, but exact last-two-majors scope is compressed. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Native apps are explicitly out of scope. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Gamification, streaks, counters, and achievements are blocked. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Tamagotchi mechanics, hunger, distress, and death are blocked. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Discovery, follows, comments, profiles, rankings are blocked. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **82.1%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md:3 "already alive"; "quiet field"; "spinner-first framing"; "already alive illusion" | PLAN.md:5 core promise; 204-205 quiet field/no spinner; 218 first frame mid-action; 235-237 procedural calls; 367 already-alive risk | yes | yes | no | full | Continuity, non-static loading, motion, audio variation, and perf all inherit the same aliveness principle. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md:3 "notices the user without announcing them"; 13 blocks "gamification, streaks, counters"; 17 social access is "quiet" | PLAN.md:5 notices without announcing; 19 no gamification/streaks/counters; 21 no social rankings/profiles; 206 sparse top bar | yes | yes | no | partial | The top-level rule survived, but the processed-vs-seen rationale and cumulative-toast failure mode did not. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md:13 "observational, sparse, and naturalist"; 101 "observed specifics" rather than generic session logging | PLAN.md:173-177 noteworthy state changes, naturalist lowercase prose, observed specifics; 258 narration not stat-list based | yes | yes | no | full | Specific naturalist observation, not generic stat/event language, is preserved. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md:15 "individually recognizable inside a bounded small flock"; 35 tune "drift and recognizability" before expansion | PLAN.md:5 two starter birds, up to seven; 19-22 blocks gamification/social/customization; 206 sparse top bar; 241-243 ambient mix and recognizability | yes | yes | no | partial | The small-flock restraint survived, but one-screen/calm-palette/no-chrome restraint was only partially carried. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md:13 naturalist/non-stat voice and "matter-of-fact system voice where appropriate" | PLAN.md:176 naturalist lowercase prose; 258 narration observational/naturalist; 265 matter-of-fact system voice | yes | yes | no | full | The voice split between product charm and system clarity survived. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md:93 presence is the "dominant driver"; 95 no-punishment drift; 85 validated presence-time increments | PLAN.md:134 validates presence-time increments; 145 presence-time dominant driver; 148 neglect does not decrement; 198 owner sessions only | yes | yes | no | partial | Presence as a drift input survived, but the precise signal conjunction and settle/tab-close equivalence did not. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md:5 canonical state on server; 27 server-authored truth for sync; 105 divergence structurally impossible | PLAN.md:32 simulation service owns minute-scale tick; 43 server only writer; 189-192 no client bird-state writes; 196 event log serializes devices | yes | yes | no | full | Server authority, multi-device coherence, and conflict-prevention consequences were all recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md:9 privacy/non-analytics boundaries are "part of the product shape" and exclude relationship data/dashboards | PLAN.md:14 excludes per-account relationship data; 74 bounded retention; 293-305 aggregate-only metrics and exclusions; 371-375 privacy mitigation | yes | yes | no | full | The relationship-data boundary is treated as architecture, not policy prose. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md:11 accessibility is first-class and keeps charm; 47 not a stripped fallback; launch-level requirements | PLAN.md:13 accessibility surfaces in scope; 224-229 reduced-motion parity; 256-259 narration; 270-271 captions/audio-off continuity; 361-363 launch-blocking deliverables | yes | yes | no | full | Accessible variants keep the affective product and ship as launch-blocking work. |

Multi-layer system-level whys:

| Why ID | L1 primary | L2 secondary | L3 downstream |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | no |
| S6 | yes | no | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: PLAN.md:5 core promise; 204-205 quiet field/no spinner; 218 mid-action first frame; 235-237 procedural calls; 367 already-alive perf risk.
- S2: PLAN.md:5 notice without announcing; 19 no streaks/counters; 21 no public social/rankings; 206 sparse top bar. Preserved broadly but with thin deeper rationale.
- S3: PLAN.md:176 naturalist specific notebook prose; 258 naturalist narration; 72 hidden vector values; 21 no public ranking/profile surfaces.
- S4: PLAN.md:5 two starter/seven cap; 206 sparse top bar; 241-243 ambient non-solo mix; 357 mix cap at seven. Broader one-screen/palette details are incomplete.
- S5: PLAN.md:176 notebook naturalist voice; 258 narration naturalist/non-stat; 265 matter-of-fact control/system voice.
- S6: PLAN.md:134 validates presence-time; 145 presence-time dominant drift driver; 148 neglect not punitive; 19 no counters. Missing three-signal precision and settle-close equivalence.
- S7: PLAN.md:32 simulation service tick; 43 server only writer; 183-192 snapshot/client-disposable/no-client-writes; 196 event-log serialization.
- S8: PLAN.md:14 relationship data excluded; 74 bounded retention; 293-305 aggregate metrics only; 371-375 PII/bird-state review checks.
- S9: PLAN.md:13 accessibility surfaces; 226-229 reduced-motion parity; 256-259 narration cadence/voice; 270-271 captions/audio-off continuity; 361-363 launch-blocking accessibility.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **44.8%**.

Reachable feature-level whys: **37 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured: no visibility/focus/recent-input conjunction. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md:93 drift slowly shapes expressive traits; presence dominant; diminishing returns prevent jumps | PLAN.md:136 calibrated low-pass filters; 150-154 one-week/three-week/no-jump targets | no | partial | Recovered slow drift, but not the one-week vs three-week calibration rationale or Tamagotchi/screensaver failure band. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md:95 no negative neglect decrement preserves no-punishment contract | PLAN.md:143 monotonic upward adjustments; 148 neglect does not decrement; 331 tests no-negative-neglect | no | partial | Recovered positive-only/no-punishment drift, but not the two-weeks-away/mistrust consequence. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md:41 runtime synthesis, bird-specific seeds, and rejection of recorded substitutes | PLAN.md:235-237 motif libraries and WebAudio synthesis; 250 no recorded fallback; 357 fallback over recorded substitutes | no | partial | Recovered procedural-vs-recorded and fallback cascade, but not the chorus phase-canceling rationale. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md:97 mood creates persistent, varied behavior; 117 reduced motion keeps product behavior | PLAN.md:219 continuous idle micro-motion is mood-shaped | yes | none | The motion rule survived, but the why that users read mood without labels did not. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md:15 individually recognizable bounded small flock; 35 mix caps at seven birds | PLAN.md:5 expansion up to seven; 241-243 chorus preserves recognizability; 357 mix caps at seven birds | no | full | Recovered the recognizability ceiling rationale. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION.md:27 one server-authored truth; 61 server-only authority; 73 history-safe deltas | PLAN.md:43 server only writer of personality vectors; 61 vector on bird record; 189-192 no client writes, tick transaction | no | partial | Recovered canonical server persistence and sync consequences, but not the relationship loss of deleting the known bird. |
| F8 | personality-vector-never-numerical | 2 | yes | included | RECONSTRUCTION.md:71 personality values stay out of product UI and accessible surfaces | PLAN.md:72 personality values are hidden from product UI and accessible surfaces | yes | none | Rule survived; stat-management/relationship-collapse why did not. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md:99 social warmth affects who greets first; 101 first greeter changes | PLAN.md:5 notices the user; 169 social warmth influences who greets first; 175 first greeter changes | yes | none | Greeting mechanics are thinly present, but arrival timing, absence-length variation, and notice-not-announce payoff are absent. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md:3 notices the user without announcing them | PLAN.md:5 notices the user without announcing them | yes | none | The global principle exists, but no feature-level no-toast rationale or variants were reconstructed. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured: close-tab equivalence/optional settle is absent. |
| F12 | field-notebook-auto-entries | 4 | yes | included | RECONSTRUCTION.md:43 noteworthy rare entries, naturalist lowercase prose, one every few days; 101 observed specifics rather than generic session logging | PLAN.md:173-177 noteworthy changes, naturalist prose, sparsity gate; 75 notebook entries immutable | no | full | Notebook voice, anti-event-log rationale, and sparse cadence survived. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md:85 coarse-grained heartbeats and validated presence-time increments | PLAN.md:114 coarse heartbeats; 134 validated presence-time increments; 145 presence-time dominant driver | yes | none | Presence accounting survived only generically; the three-signal precision and silent drift-corruption failure did not. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md:13 blocks gamification, streaks, counters, achievements | PLAN.md:19 gamification, streaks, counters, achievements blocked | yes | none | Rule survived; presence-for-the-counter and adjacent-disguise rationale did not. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md:67 birds placed mid-action; 109 quiet field/no spinner into live scene; 19 first bird timing supports already-alive illusion | PLAN.md:50 mid-action first paint; 204-205 quiet field and no spinner; 218 first frame mid-action; 367 already-alive illusion | no | full | First-frame motion, snapshot implementation, and quiet-field/no-spinner consequence all survived. |
| F16 | synthetic-account-id | 4 | yes | included | none | PLAN.md:58 synthetic UUID primary key and encrypted email; 371-375 email/per-bird leakage mitigation and synthetic UUID enforcement | yes | none | plan_only_not_reconstructed: the PLAN grounds the PII rationale, but the reconstruction never addresses synthetic IDs. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md:37 tick advances canonical mood/drift/notebook/weather/calls; 39 sync coherence; 105 divergence structurally impossible | PLAN.md:131-139 once-per-minute tick flow; 183-192 snapshots, no client writes, tick transaction; 196 event-log serialization | no | full | Tick mechanics, sync coherence, and divergence-prevention rationale survived. |
| F18 | no-last-write-wins-personality | 4 | yes | included | RECONSTRUCTION.md:73 history-safe deltas, not replacement values; 105 append-only events and simulation tick transactions | PLAN.md:73 additive history-safe deltas; 189-192 no client writes and tick transaction; 196 event log serializes devices | no | partial | Recovered additive/server-authored implementation, but not the invisible lost-drift last-write-wins failure story. |
| F19 | sync-conflict-matter-of-fact | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured: no sync-conflict/error surface with matter-of-fact copy. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md:9 excludes per-account bird relationship data and relationship-dynamics dashboards; 49 excludes per-bird state and histories | PLAN.md:14 aggregate observability excludes relationship data; 293-305 aggregate metrics only and explicit exclusions; 371-375 privacy boundary mitigation | no | full | Recovered own-simulation/privacy-boundary rationale and technical telemetry separation. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md:45 quiet sharing without changing host aviary; visitor activity never writes host drift inputs | PLAN.md:46 read-only render sessions; 76 visitor activity never writes host drift inputs; 125 no notebook mutation or interaction posting | no | full | Recovered read-only observation and no accidental host drift. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md:17 social access is optional, quiet, read-only; 45 quiet sharing | PLAN.md:5 optional quiet visit invitations; 313 visits default-off | yes | none | Quiet social framing survived, but no-notification-by-default and attention-loop rationale did not. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | RECONSTRUCTION.md:13 blocks gamification; 49 excludes relationship-dynamics dashboards | PLAN.md:21 no discovery feeds, profiles, rankings; 301-305 no relationship-dynamics dashboards | yes | none | Rule survived, but the comparison-surface/different-product rationale did not. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md:47 coherent aviary experience, not stripped fallback; 125 narration from same state graph, observational and naturalist, slow cadence | PLAN.md:256-259 live-region narration, same state graph, observational/naturalist, rate limits; 361-363 accessibility not stripped fallback | no | full | Running naturalist prose, same-product accessibility, and event-priority behavior survived. |
| F25 | reduced-motion-mode | 4 | yes | included | RECONSTRUCTION.md:47 accessibility must not be a stripped fallback; 117 still poses, slowed transitions, identical affordances/notebook behavior | PLAN.md:226-229 cross-fades/still poses, removed ornaments, slowed daypart transitions, identical behavior; 270-271 captions/narration continuity | no | partial | Recovered cross-fade rendering and not-stripped fallback; omitted full calls/drift/mood continuity layer. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECONSTRUCTION.md:19 first bird visible within 500ms supports already-alive illusion; 131 budgets protect live-scene experience | PLAN.md:278 first bird within 500ms; 289 defers ornaments; 367 performance failure breaks already-alive illusion | no | full | Recovered affective-performance bridge. |
| F27 | no-gamification | 4 | yes | included | RECONSTRUCTION.md:13 product voice is observational/naturalist rather than gamified or stat-based | PLAN.md:19 gamification, streaks, counters, achievements blocked; 21 rankings blocked | no | partial | Recovered the anti-stat/gamification stance, but not the cheap-engagement temptation or foothold rationale. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | RECONSTRUCTION.md:7 blocks Tamagotchi mechanics and preserves slow non-punitive relationship change; 95 no-punishment contract | PLAN.md:20 no Tamagotchi mechanics; 148 neglect does not decrement; 331 tests no-negative-neglect | no | full | Recovered observational/non-punitive refusal of custodial mechanics. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION.md:35 two starter birds and gradual expansion tied to tuning and recognizability | PLAN.md:5 two starter birds; 313 public launch with two starter birds | yes | none | Starter birds survived, but arrival-not-catalog and naming-preserves-intimacy why did not. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECONSTRUCTION.md:135 public launch uses staged bird unlocks; 137 tune recognizability before unlocks | PLAN.md:313 staged unlock scheduling by aviary age; 317-318 tune recognizability before increasing bird count | yes | none | Age-based unlock survived, but the anti-reward-loop/economy rationale did not. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md:67 snapshot includes stable bird ids; 71 hidden personality values | PLAN.md:61 bird has stable UUID; 102 snapshot bird records include stable ids | yes | none | Stable ids survived, but same-individual/remembered-relationship rationale did not. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md:97 mood creates persistent, varied behavior and is never reset on tab open | PLAN.md:163 mood persists across sessions and is never reset on tab open; 50 canonical state includes mood for first paint | no | full | Recovered no-neutral-reset persistence and continuity. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | none | PLAN.md:75 notebook entries immutable after creation | yes | none | RECONSTRUCTION.md:77 says NOT RECOVERABLE FROM PLAN; observer-record-not-journal why is absent. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | PLAN.md:10 account export in scope; 87 POST /account/export; 285 code-split export surface | yes | none | RECONSTRUCTION.md:33 says NOT RECOVERABLE FROM PLAN; quiet relationship-copy why is absent. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | none | PLAN.md:10 soft-delete and recovery; 58 deletion window; 88-89 delete/cancel APIs | yes | none | RECONSTRUCTION.md:33 says NOT RECOVERABLE FROM PLAN; 30-day/hard-delete rationale was not recovered. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md:49 monitor operational metrics while excluding per-bird state and per-account interaction histories | PLAN.md:293-305 aggregate operational metrics and explicit relationship-data exclusions; 327 aggregate accessibility usage only | no | full | Recovered technical telemetry boundary. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md:45 quiet sharing; 91 constrained ongoing read-only access | PLAN.md:67 invitee email; 120 invitation API; 21 discovery/social surfaces blocked | yes | none | Per-invite mechanism survived, but named private-relationship/control rationale did not. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md:45 host metadata limited to transparency | PLAN.md:96 GET /aviary/visit-log; 125 host metadata only as needed for transparency | yes | none | Transparency survived, but on-demand log vs notification-surface rationale did not. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md:45 quiet sharing without changing the host aviary; 65 same snapshot read path | PLAN.md:46 visit sessions use same snapshot read path; 125 visitor snapshot restrictions; 22 public showcase variants blocked | no | full | Recovered actual-host-aviary, not show-off variant. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md:125 slow cadence, rate limits, user-triggered preemption, avoid queue flooding | PLAN.md:256 slow cadence and prioritization; 259 rate limits/user preemption; 335 narration cadence acceptance tests | no | full | Recovered slow rhythm, queue protection, and sparse user-event priority. |

Multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | no | no | no |
| F2 | yes | no | no |
| F3 | yes | yes | no |
| F4 | yes | no | yes |
| F7 | yes | no | yes |
| F9 | no | no | no |
| F10 | no | no | no |
| F12 | yes | yes | yes |
| F13 | no | no | no |
| F14 | no | no | no |
| F15 | yes | yes | yes |
| F16 | no | no | no |
| F17 | yes | yes | yes |
| F18 | yes | no | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | no | yes |
| F27 | yes | no | no |
| F30 | no | no | no |
| F31 | no | no | no |
| F35 | no | no | no |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From gold_why_totals |
| Possible total weight | 152 | From intent_recovery.total_possible_weight |
| Reachable gold whys | 46 | System whys plus reachable feature whys |
| Excluded unreachable feature whys | 3 | F1, F11, F19 |
| Recovered / reachable weight | 75.0 / 144.0 | Weighted recovery over included whys |
| Whys with reconstruction evidence | 42 | Rows with non-none frozen reconstruction evidence |
| Whys with PLAN grounding | 46 | Rows with non-none PLAN grounding |
| rule_without_why cases | 17 | Mechanism or rule survived without gold rationale |
| plan_only_not_reconstructed cases | 1 | PLAN carried rationale but reconstruction did not |
| ungrounded_reconstruction cases | 0 | No confabulated rationale counted |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 50.0 | 28.0 | 56.0% |
| Affective whys | 94.0 | 47.0 | 50.0% |
| Weight-2 whys | 40.0 | 21.0 | 52.5% |
| Weight-3 whys | 104.0 | 54.0 | 51.9% |
| System-level whys | 28.0 | 23.0 | 82.1% |
| Feature-level whys (reachable) | 116.0 | 52.0 | 44.8% |

---
## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered 28.0/50.0 (56.0%), while affective whys recovered 47.0/94.0 (50.0%). Architecture and privacy mechanisms survived better than relationship-tone exceptions.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 54.0/104.0 (51.9%), while weight-2 whys recovered 21.0/40.0 (52.5%). Higher weight did not produce a meaningful advantage because many multi-layer downstream consequences were compressed away.
- **System-level vs feature-level.** System-level fidelity was strong at 82.1%; feature-level fidelity was 44.8%. The planner preserved broad philosophy, but many feature-level whys were rule-only.
- **Multi-layer recovery patterns.** Primary mechanisms often survived (drift slow, calls procedural, server tick canonical), while secondary temptation/failure-mode layers were frequently absent.
- **Subdomain patterns.** Accessibility and server-sync were strongest. Social quietness, account lifecycle, presence precision, and no-gamification exceptions leaked the most.
- **Evidence-bound effects.** F16 was denied as plan-only-not-reconstructed. F8, F14, F29, F30, F31, F33, F37, and F38 are representative rule-without-why rows.

The failure shape suggests a plan that is excellent at enumerating buildable subsystems, but less consistent at carrying the deeper affective arguments behind refusal features.

---
## 4. Recommendations for v2 hardening

- Keep v06 evidence-bound scoring. It clearly distinguishes mechanism preservation from why preservation.
- Add or retain targeted headroom whys around quiet social/account lifecycle surfaces; this run implemented those features while losing much of the rationale.
- Consider splitting presence precision into an explicit high-salience plan requirement, because generic presence accounting is too easy to preserve while losing the load-bearing signal definition.
- Preserve the system-level cross-cutting appendix in reports. S2 and S4 show why the >=3-feature bar needs an audit trail, not just a yes/no call.
- In future instances, include more feature-level whys whose rationale is not immediately derivable from architecture; these discriminate strong implementation plans from strong intent-carrying plans.

---
## 5. Methodology caveats

- **Fresh-context fidelity.** The scoring prompt states fresh context, and the validity audit found no significant contamination signatures in the frozen reconstruction.
- **Single-run-at-temperature limitation.** This is one run with no variance signal.
- **Borderline capture calls.** Inclusive calls were made for starter-bird auto-selection, return greeting, no welcome toast, friend-visit notification, and account deletion. These lifted planning quality slightly but did not materially inflate fidelity because why recovery remained evidence-gated.
- **System-level cross-cutting.** S2 and S4 were the finest calls: both were cross-cutting in the PLAN but only partially recovered because deeper affective layers were missing.
- **Confabulation cases.** No ungrounded reconstruction cases were counted. Assertions in the reconstruction were broadly traceable to the PLAN, even when they were too thin for gold-why recovery.
- **Evidence-bound denials.** F16 is plan-only-not-reconstructed. Many no-credit rows are rule-without-why rather than missing-feature rows.
- **Operational compromise.** TIMING.json supplied phase 1 and phase 2A timing only; phase 2B timing was not fabricated in the score JSON.

---
End of report.
