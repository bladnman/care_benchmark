# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Frozen reconstruction was scored without modification.

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **95.0%** |
| Intent fidelity | **72.4%** |
| Combined quality | **9472** |

**Diagnostic split:**

- System-level fidelity: **100.0%**
- Feature-level fidelity: **66.1%**
- (Planning, fidelity) coordinate: `(95.0, 72.4)`

### Run metadata

| Field | Value |
|---|---|
| Run number | 1 |
| Run label |  |
| Timestamp | 2026-05-09T01:50:41Z |
| Candidate model | gpt-5.5 |
| Candidate effort | high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **114 / 120** = **95.0%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 3 | 75.0% |
| bird_engine.md | 22 | 21 | 95.5% |
| interactions.md | 20 | 19 | 95.0% |
| aviary_layout.md | 18 | 17 | 94.4% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 17 | 94.4% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **114** | **95.0%** |

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes |  |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes |  |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes |  |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes |  |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes |  |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes |  |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No standalone glossary/domain-term deliverable appears in the plan. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes |  |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes |  |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes |  |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes |  |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes |  |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes |  |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes |  |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes |  |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes |  |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes |  |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes |  |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes |  |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes |  |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes |  |
| 22 | Mood-shaped idle motion | bird_engine.md | yes |  |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | no | Species pool is referenced, but the v1 target of about six species is not specified. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes |  |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes |  |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes |  |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes |  |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes |  |
| 29 | Mood persistence across sessions | bird_engine.md | yes |  |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes |  |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes |  |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes |  |
| 33 | Return-greeting on viewer arrival | interactions.md | yes |  |
| 34 | Greeting variation by absence length | interactions.md | yes |  |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes |  |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes |  |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes |  |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes |  |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes |  |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes |  |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes |  |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes |  |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes |  |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes |  |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes |  |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes |  |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes |  |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes |  |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes |  |
| 50 | No streak counter, no "days visited" display | interactions.md | yes |  |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes |  |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | A five-second undo exists, but the click-anywhere undo affordance is not specified. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes |  |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes |  |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes |  |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes |  |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes |  |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes |  |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes |  |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes |  |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes |  |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes |  |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes |  |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes |  |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes |  |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes |  |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes |  |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | The plan only says not to show empty after adoption; it does not define the between-adoption empty state. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes |  |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Borderline inclusive: stable aspect constraints are treated as enough to imply no bird-cropping. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes |  |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes |  |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes |  |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes |  |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes |  |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes |  |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes |  |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes |  |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes |  |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes |  |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes |  |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes |  |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes |  |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes |  |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes |  |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes |  |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | The data model has a privacy-policy acknowledgement field, but no account-settings privacy-policy link. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes |  |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes |  |
| 90 | Visits default OFF for new accounts | social_optional.md | yes |  |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes |  |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes |  |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes |  |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes |  |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes |  |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes |  |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes |  |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes |  |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes |  |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes |  |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes |  |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes |  |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes |  |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes |  |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes |  |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes |  |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes |  |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes |  |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes |  |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes |  |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes |  |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes |  |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes |  |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes |  |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes |  |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | The plan says modern browsers and tests several browsers, but does not define the last-two-majors support matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes |  |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes |  |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes |  |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes |  |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **100.0%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md:3 says continuity/procedural variation/restriction show up in living continuously, slow tick, sparse notebook, and quality-first v1. | PLAN.md:5, 559, 838, 1010: living continuously; birds mid-action; no spinner; aviary already there. | yes | yes | no | full | The reconstruction recovers aliveness as a cross-product principle, though in compressed terms. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md:9 and 35: non-announcing voice; bird notices through action rather than welcome copy. | PLAN.md:21, 449, 601, 983, 1010: no welcome surfaces, no badge pressure, and restraint against announcements. | yes | yes | no | full | The reconstructed principle is broad enough to cover welcome, visit, badge, and copy surfaces. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md:43, 95, 207 cite sparse naturalist observations, stable prose, and no event-log phrasing. | PLAN.md:493-500 and 694-700 specify lowercase, present-tense, specific naturalist prose and no raw states. | yes | yes | no | full | Specific naturalist prose survived as a product-wide voice constraint. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md:3 and 25: restraint over breadth, small set of birds, one-scene v1, and quality/audio recognizability. | PLAN.md:5, 10-11, 543, 549-550, 1008: small set, max seven, one scene, bundle guard, cap/ramp hold. | yes | yes | no | full | The plan and reconstruction both treat restraint as a governing design rule. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md:9 and 113 distinguish naturalist product prose from matter-of-fact account/error surfaces. | PLAN.md:246, 493-496, 706-709, 931-935 define product voice and system/error/settings exceptions. | yes | yes | no | full | The voice split is explicitly reconstructed and cross-cutting in the plan. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md:145, 157: presence-time is dominant drift input; visible/focused/recently active sessions; closing without settle penalty-free. | PLAN.md:374, 384, 470, 890-891 specify the three-signal rule, dominant drift input, neutral close, and rejection of hidden tabs. | yes | yes | no | full | The reconstruction recovers precision, stale-tab risk, and settle/close neutrality. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md:5, 47, 165: server tick authority, append-only events, no client-to-client merge, no LWW. | PLAN.md:45-49, 504-522, 951-955: server tick owns state and prevents stale concurrent overwrites. | yes | yes | no | full | Server authority and multi-device consequences are fully recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md:11, 219, 231: interaction events are private inputs; analytics cannot read simulation data. | PLAN.md:170, 242, 734-740, 975-979: per-bird events drive simulation only and telemetry excludes relationship data. | yes | yes | no | full | The data-pipeline privacy boundary survived. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md:15, 51, 207, 215: accessibility ships in v1 and designed modes are not static fallback. | PLAN.md:18, 690, 849, 963-967: first-class surfaces ship in v1 and are product work. | yes | yes | no | full | The reconstruction preserves accessibility as product quality, not compliance cleanup. |

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: PLAN.md:5 product intent; 28 seeded procedural choices; 559 birds mid-action/no spinner; 627 procedural calls; 690 accessibility with same product feel; 1010 final success condition.
- S2: PLAN.md:21 excluded toasts/counters/social surfaces; 449 no text welcome; 601 notebook badge restraint; 983 small additions risk; 1003 welcome-copy guardrail.
- S3: PLAN.md:493-500 notebook voice; 694-700 narration voice; 593 offer naturalist voice; 933 copy tests; 66 no numeric trait surfaces.
- S4: PLAN.md:10 two-to-seven birds; 11 one scene; 549-550 stable one-screen scene; 543 bundle restraint; 872-874 bird-ramp restraint.
- S5: PLAN.md:246 API/system voice; 493-496 naturalist notebook; 706-709 accessible/control names; 931-935 copy/voice tests.
- S6: PLAN.md:374 three-signal presence; 384 dominant drift input/no counter; 470 close without penalty; 890-891 simulation tests.
- S7: PLAN.md:45-49 simulation service; 355-370 tick algorithm; 504-522 sync model; 951-955 sync risk mitigation.
- S8: PLAN.md:70 synthetic ID; 170 private inputs; 242 allowed telemetry fields; 734-740 privacy rules; 975-979 telemetry erosion mitigation.
- S9: PLAN.md:18 first-class surfaces; 613-623 reduced-motion renderer; 690-702 narration; 849 designed aviary exit criterion; 963-967 accessibility risk mitigation.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **66.1%**.
Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md:91, 131, 145: bounded windows, visible/focused/recent activity, dominant drift input. | PLAN.md:183, 374, 384, 947-949: all three signals, dominant drift input, false-drift risk. | no | partial | Precision and stale-tab inflation survived; the silent population-wide failure layer was only implicit. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md:147: measurable after a week, visible after three, no single-session jump. | PLAN.md:388-409 and 941-943: slow low-pass drift with the same calibration targets. | no | partial | Calibration survived; Tamagotchi-versus-screensaver failure framing did not. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md:147 and 251: never negative deltas; absence making birds worse is a Tamagotchi violation. | PLAN.md:403, 827, 890, 1007: no negative neglect and no absence punishment. | no | full | The no-punishment rationale is recovered through drift and guardrail evidence. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md:45, 195, 199: procedural recognizable calls, no recorded fallback, chorus avoids loop artifacts. | PLAN.md:15, 627, 653-654, 957-961: procedural WebAudio, recognizable chorus, repetition/phase risk. | no | partial | Mechanics and chorus risk survived; audio-as-affective-spine cascade was compressed away. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md:33: mood becomes observable through pose, call probability, perch, and micro-motion without labels. | PLAN.md:553, 565-568, 421-436: mood modifiers drive pose, call, perch, and offer behavior. | no | full | Mood-as-readable-motion survived. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md:25: cap/ramp protects quality and audio recognizability. | PLAN.md:10, 872-874, 1008: seven-bird ramp holds until recognizability holds. | no | full | The recognizability rationale survived, though not the empirical wording. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION.md:85, 149, 165: only simulation updates vectors; no recomputation; server authority. | PLAN.md:141, 411-415, 521-522: persisted vectors, formula migration, no LWW. | no | partial | Canonical persistence and downstream sync survived; the affective deletion/reset layer did not. |
| F8 | personality-vector-never-numerical | 2 | yes | included | RECONSTRUCTION.md:7 and 31: traits should be felt, not shown as stats or numeric surfaces. | PLAN.md:13, 21, 66, 1001: hidden vectors and number surfaces violate the product. | no | full | The stat-management risk survived as a relationship-protecting rule. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md:35 and 153: one primary greeter, absence length/current state, no welcome text. | PLAN.md:442-449 and 895: absence, boldness/state weighting, one greeter, staggered response, no announcement. | no | partial | Greeting mechanics and non-announcement survived; the canned-cue failure mode was not explicit. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md:35 and 153: bird notice replaces text welcome and away copy. | PLAN.md:21, 449, 1003: no welcome/toast/copy because it replaces the bird greeting. | no | partial | The key refusal survived, but the full different-product consequence was compressed. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md:41 and 157: settle ends presence and closing without settle is neutral/penalty-free. | PLAN.md:461-470 and 827: settle is optional and close without settle is neutral. | no | full | The no-penalty rationale survived. |
| F12 | field-notebook-auto-entries | 4 | yes | included | RECONSTRUCTION.md:43, 95, 161: sparse naturalist observations, not event logs or feed, stored stable. | PLAN.md:493-500 and 482-491: rare meaningful observations in naturalist voice, no user summaries. | no | full | Notebook voice, sparsity, and anti-feed rationale survived. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md:131 and 145: visible/focused/recently active pings; stale tabs cannot create runaway presence. | PLAN.md:314, 374, 381, 949: three-signal rule with bounded heartbeats. | no | partial | The conjunction and stale-tab failure survived; individual-signal miss cases did not. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md:17, 145, 249: streaks excluded, presence not displayed as counter, copy tests ban streaks. | PLAN.md:21, 384, 493-497, 981-985: no streaks/visit frequency and no user-behavior summaries. | no | partial | The explicit refusal and adjacent surfaces survived; managing-a-number rationale was mostly implicit. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md:125, 179, 241: bootstrap for first bird, quiet field/no spinner, first frame already in motion. | PLAN.md:289-292, 559, 838, 908: bootstrap, mid-action first frame, no spinner. | no | full | First-frame continuity, server/bootstrap implementation, and quiet loading all survived. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md:77: no log line, partition, metric tag, or foreign key should use email as identity. | PLAN.md:70, 76-79, 736: synthetic UUIDs and encrypted email/lookup hash. | no | partial | PII leakage prevention survived; impossible-to-retrofit consequence did not. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md:5, 143, 165: server tick authority, per-aviary locks, no client-to-client merge. | PLAN.md:45-49, 355-370, 504-522: server-side tick owns canonical state and sync. | no | full | Server tick, multi-device coherence, and client-tick failure avoidance survived. |
| F18 | no-last-write-wins-personality | 4 | yes | included | RECONSTRUCTION.md:47, 129, 171: append-only events, no direct personality updates, no stale concurrent overwrites. | PLAN.md:504-522, 900-902, 951-955: append-only events, idempotency, no LWW personality overwrite. | no | full | All three LWW layers are recovered in sync and API evidence. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | RECONSTRUCTION.md:113 and 209: account/settings/error surfaces stay direct and matter-of-fact. | PLAN.md:246, 263-264, 528, 706-709: system/error/account surfaces use direct language. | no | full | The system-clarity exception survived. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md:11, 219, 231: private simulation inputs; no analytics read; no account/bird dimensions. | PLAN.md:170, 734-740, 975-979: per-bird events are simulation-only and never ML/analytics inputs. | no | partial | The technical boundary survived; private-relationship-as-not-data-product was not explicit. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md:49, 139, 245: visits allow sharing without visitor writes or host drift effects. | PLAN.md:344-349, 541, 860: visitors cannot mutate, interact, or generate presence. | no | full | Read-only observation and no accidental drift survived. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md:49 and 187: visits are off/default quiet; persistent badge pressure is avoided. | PLAN.md:21, 83, 261, 601, 983: visit notifications default false and badge pressure is resisted. | no | full | The attention-loop refusal survived. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | RECONSTRUCTION.md:17, 49, 249: public discovery, leaderboards, profiles, and comparative/status copy are excluded. | PLAN.md:21, 884, 935, 997: no public discovery/social-network leakage. | no | full | The comparison/public-surface refusal survived. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md:207, 215, 243: slow naturalist observations, no labels/numbers/event-log phrasing, designed aviary. | PLAN.md:694-702 and 849: slow naturalist prose, no state-list wording, designed accessible aviary. | no | partial | Voice and equal-feel layers survived; anti-ARIA-automation consequence was not explicit. |
| F25 | reduced-motion-mode | 4 | yes | included | RECONSTRUCTION.md:191 and 215: same snapshot state, different presentation, keeps calls/captions/notebook/mood/drift, designed mode. | PLAN.md:613-623, 849, 963-967: cross-fades preserve the aviary, not a static fallback. | no | full | Reduced-motion as designed alternate rendering survived fully. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECONSTRUCTION.md:61, 227, 229: time-to-first-bird, product depends on under-500ms, first-bird-before-effects. | PLAN.md:289-292, 762-765, 971-973: first bird within 500ms via bootstrap and budget gates. | no | full | The affective-performance bridge is recovered through first-bird dependency evidence. |
| F27 | no-gamification | 4 | yes | included | RECONSTRUCTION.md:17, 249, 251: achievements/scores/streaks excluded; numbers and engagement surfaces violate the product. | PLAN.md:21, 884, 981-985, 1001: no gamification and product-restraint erosion risk. | no | partial | The refusal and temptation/erosion risk survived; the full foothold cascade was compressed. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | RECONSTRUCTION.md:17, 147, 251: hunger/death/distress excluded; no negative deltas; absence punishment is Tamagotchi. | PLAN.md:21, 403, 1007: no distress mechanics and no absence punishment. | no | full | The anti-caretaking/anti-punishment rationale survived. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | PLAN.md:10, 326-328, 872: two starter birds; server selects species; no catalog browsing. | yes | none | The no-catalog rule survived, but not the meeting-animals-not-configuring-avatars rationale. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECONSTRUCTION.md:25 and 137: age-paced expansion avoids earn language and count-progress copy. | PLAN.md:10, 322-328, 870-874: age-paced bird offers, no earn/count-progress, quality-gated ramp. | no | partial | The anti-reward-loop rule partly survived; relationship-deepening and economy-collapse layers did not. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md:83: stable identity supports rendering identity, procedural calls, visual continuity, and canonical transitions. | PLAN.md:119-127, 578, 811: stable IDs and identity-bearing snapshot/render state. | no | partial | Continuity survived at the identity/mechanics layer; remembered-relationship consequences did not. |
| F32 | mood-persists-across-sessions | 2 | yes | included | none | PLAN.md:153, 438, 578-581: mood persists and tab open does not reset it. | yes | none | The rule survived, but the illusion-of-continuity/reset-default rationale was not reconstructed for mood. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md:43 and 95: sparse stable naturalist observations; entries are read-only and stored as final prose. | PLAN.md:206, 294-296, 493-498, 599-601: read-only notebook, no event-log/user-behavior summaries. | no | full | Observer-record stability survived. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | PLAN.md:19, 272-273, 751: account export exists with signed delivery. | yes | none | Export mechanics survived, but not the quiet relationship-copy rationale. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md:53, 123, 221: soft deletion/recover endpoints, recovery window, hard deletion worker. | PLAN.md:19, 275-279, 751-752, 857, 929: 30-day recovery then hard deletion. | no | partial | Grace-window mechanics survived; regret, privacy, and residue rationale were mostly absent. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md:99, 219, 231: aggregate fields only; no account/bird/event/notebook dimensions. | PLAN.md:242, 734-740, 780-785: operational metrics only and privacy-safe dashboards. | no | full | The technical telemetry boundary survived. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md:49 and 139: visit invitations share without public discovery or social-network mechanics. | PLAN.md:17, 337-339, 997: email visitor invites and no public identifiers beyond named email invites. | no | full | The deliberate named-sharing/no-discovery rationale survived. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | PLAN.md:334-335, 601, 855: recent visit log exists in settings, not as a badge surface. | yes | none | The visit-log rule is in the plan but the reconstruction did not recover the transparency-not-attention-loop why. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | PLAN.md:344-346, 541: visitor gets the same rendering snapshot as host. | yes | none | The same-snapshot rule survived, but not the no-show-off/real-birds rationale. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md:127 and 207: slow-cadence narration, naturalist observations, no flooding or event-log phrasing. | PLAN.md:694-702, 843, 849: 30-60s idle cadence, priority events, coalescing, designed surface. | no | partial | Slow rhythm and queue-protection survived; user-event priority/monitoring distinction was compressed. |

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | yes | no |
| F7 | yes | no | yes |
| F9 | yes | yes | no |
| F10 | yes | yes | no |
| F12 | yes | yes | yes |
| F13 | yes | no | yes |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | no |
| F25 | yes | yes | yes |
| F27 | yes | yes | no |
| F30 | no | yes | no |
| F31 | yes | no | no |
| F35 | yes | no | no |
| F40 | yes | yes | no |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From gold_why_totals |
| Possible total weight | 152 | From intent_recovery.total_possible_weight |
| Reachable gold whys | 49 | S whys always included; all F why anchors were captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 110.0 / 152 | Sum of weight times recovery over included whys |
| Whys with reconstruction evidence | 44 | Rationale evidence present in frozen reconstruction |
| Whys with PLAN grounding | 49 | Exact PLAN grounding present |
| rule_without_why cases | 5 | Rule survived without gold rationale |
| plan_only_not_reconstructed cases | 5 | PLAN/rule grounding present; reconstruction lacks why |
| ungrounded_reconstruction cases | 0 | No scored row required the confabulation guard |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 | 38.0 | 70.4% |
| Affective whys | 98 | 72.0 | 73.5% |
| Weight 2 whys | 44 | 34.0 | 77.3% |
| Weight 3 whys | 108 | 76.0 | 70.4% |
| System-level whys | 28 | 28.0 | 100.0% |
| Feature-level whys (reachable) | 124 | 82.0 | 66.1% |

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered 38/54 weighted points (70.4%); affective whys recovered 72/98 (73.5%). The difference is small, but functional losses were often privacy/account nuance (F16, F20, F34, F35), while affective losses clustered in relationship framing (F29, F32, F38, F39).
- **Weight-3 vs weight-2.** Weight-3 whys recovered 76/108 (70.4%); weight-2 whys recovered 34/44 (77.3%). High-weight whys were more demanding because L2/L3 consequence layers were often compressed.
- **System-level vs feature-level.** The plan preserved philosophy extremely well: S1-S9 recovered fully. Feature-level fidelity fell because the reconstruction often restated mechanisms, not the specific why behind them.
- **Multi-layer recovery patterns.** Primary implementation layers usually survived; downstream consequence layers were the most common loss. Examples: F2 lost the Tamagotchi/screensaver boundary, F7 lost the reset-bird affect, and F24 lost the anti-ARIA-automation consequence.
- **Subdomain patterns.** Core simulation/sync was strongest (F17, F18 full). Accessibility was good but compressed (F24, F40 partial; F25 full). Social/visits lost the most relationship nuance (F38, F39 none).
- **Evidence-bound effects.** Five mechanism-only rows were denied: F29, F32, F34, F38, and F39. No ungrounded reconstruction was awarded.

The failure shape suggests the phase-1 plan was strong as an implementation plan, but phase-2 reconstruction favored concise design-rule summaries over preserving feature-level rationale prose.

## 4. Recommendations for v2 hardening

- Keep F29-F40 style targeted headroom. Those rows exposed differences between building the right mechanism and carrying the right rationale.
- Add more single-layer feature whys where the rule is easy to preserve but the relationship reason is subtle. F34, F38, and F39 were useful examples.
- Preserve the evidence-bound ledger. It prevented rule-only language from inflating intent fidelity.
- Retain the system-level cross-cutting appendix. It made the S-level full scores auditable instead of impressionistic.
- Consider a secondary diagnostic for partial multi-layer rows that records which layer type was lost most often; here, downstream consequence layers drove much of the loss.

## 5. Methodology caveats

- **Fresh-context fidelity.** This phase-2B task was handled as fresh context. The frozen reconstruction was read-only and was not modified.
- **Single-run-at-temperature limitation.** This is one run in one slot; it gives no variance estimate by itself.
- **Borderline capture calls.** Feature 70 was counted inclusively because stable aspect constraints imply no-cropping behavior. Feature 52 was not counted because the plan had five-second undo but not the click-anywhere affordance.
- **System-level cross-cutting.** All nine S whys cleared the three-feature inheritance bar; see the appendix in section 2.2.
- **Confabulation cases.** No scored recovery depended on a reconstruction claim that lacked PLAN grounding.
- **Evidence-bound denials.** Several v1-style semantic matches became none under v06 because the reconstruction contained only the mechanism, not the rationale.
- **Rule-without-why cases.** F29, F32, F34, F38, and F39 are the main mechanism-only recoveries.

End of report.
