# REPORT - CARE run 001

> Variant v06 evidence-bound clean. Every S1-S9 and F1-F40 row below separates denominator status from recovery and cites frozen reconstruction evidence plus PLAN grounding where recovered.

---

## 1. Headline

| Score | Value |
| --- | --- |
| Planning quality | **99.2%** |
| Intent fidelity | **78.3%** |
| Combined quality | **9895** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **77.4%**

**(Planning, fidelity) coordinate:** `99.2, 78.3`

### Run metadata

| Field | Value |
| --- | --- |
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-06T16:44:52Z |
| Candidate model | gpt-6-astra |
| Candidate effort | extra-high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **119 / 120** = **99.2%**.

By PRD file:

| File | Total | Captured | Rate |
| --- | --- | --- | --- |
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 20 | 100.0% |
| aviary_layout.md | 18 | 18 | 100.0% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **119** | **99.2%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
| --- | --- | --- | --- | --- |
| 1 | Headline product concept statement | product_brief.md | yes | Captured in PLAN with implementation-level specificity. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured in PLAN with implementation-level specificity. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured in PLAN with implementation-level specificity. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured in PLAN with implementation-level specificity. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in PLAN with implementation-level specificity. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured in PLAN with implementation-level specificity. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Captured semantically through the plan's domain definitions and typed contracts for bird, call, mood, presence, and settle. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured in PLAN with implementation-level specificity. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured in PLAN with implementation-level specificity. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured in PLAN with implementation-level specificity. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured in PLAN with implementation-level specificity. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured in PLAN with implementation-level specificity. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | borderline: Borderline inclusive: PLAN carries calmness, sky/foliage palette, plumage palettes, and contrast tokens, but not the exact saturated-accent exclusion. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured in PLAN with implementation-level specificity. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No explicit privacy-policy link in account settings found in PLAN; only privacy controls/evidence are present. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured in PLAN with implementation-level specificity. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured in PLAN with implementation-level specificity. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured in PLAN with implementation-level specificity. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured in PLAN with implementation-level specificity. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured in PLAN with implementation-level specificity. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured in PLAN with implementation-level specificity. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured in PLAN with implementation-level specificity. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured in PLAN with implementation-level specificity. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured in PLAN with implementation-level specificity. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured in PLAN with implementation-level specificity. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured in PLAN with implementation-level specificity. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured in PLAN with implementation-level specificity. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured in PLAN with implementation-level specificity. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured in PLAN with implementation-level specificity. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured in PLAN with implementation-level specificity. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured in PLAN with implementation-level specificity. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **82.1%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md:3,21: "already alive"; server "cannot be paused"; procedural specificity over canned assets. | PLAN.md:9,117,322,354: "already in motion"; normal ticks advance; no spinner; no recorded loops. | yes | yes | no | full | All three aliveness layers survive: continuing place, cross-layer procedural expression, and anti-canned failure risk. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md:13,41: no welcome toast, badge, rewards, streaks, counters; "no numbers to optimize". | PLAN.md:13,179,314,523: no welcome text, no notices/badges, bird response only, no reminders. | yes | yes | no | partial | The rule and cumulative refusal survive, but the processed-vs-seen affective register is not reconstructed. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md:13,21,97: "specific, lowercase, and observational"; procedural specificity; shared facts/vocabulary. | PLAN.md:52,199,384,523: shared semantic observations, named facts, naturalist prose, specific observational copy. | yes | yes | no | full | Specificity is preserved across notebook, narration, captions, procedural calls, and non-numeric presentation. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md:9,13,23,328,332,362: two-to-seven birds, one scene/no panning, no scene chrome, others remain audible. | no | yes | no | partial | PLAN preserves restraint cross-cuttingly, but the reconstruction does not identify depth-per-bird over variety as a system principle. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md:13,273,523: account/error copy matter-of-fact; product copy specific, lowercase, observational. | PLAN.md:230,280,368,523: API/account/accessibility errors matter-of-fact; product surfaces observational. | yes | yes | no | full | The product/system voice split is reconstructed and grounded across errors, auth, settings, captions, and product prose. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md:9,177,183,197: measured attention, exact predicate, not tab-open, no offline credit or penalty. | PLAN.md:131-141,149-157,187,445-447: presence predicate, drift dominance, settle/tab-close equivalence, unioned intervals. | yes | yes | no | partial | The attention and nonpunitive layers survive; the population-wide silent drift-corruption rationale is only lightly reconstructed. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md:3,7,77,89,287: one canonical aviary/history; sole worker; no second writer; one canonical sequence. | PLAN.md:25,47,100,117,123,250: server tick, sole simulation worker, client renders snapshots, no competing history. | yes | yes | no | full | Server authority, multi-device coherence, and avoidance of divergent client simulations are all recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md:15,95,433,515: private state never used for training/analysis; metrics collector lacks simulation access. | PLAN.md:63,414-420,489,512: private state boundary, forbidden telemetry labels, no population tuning from private events. | yes | yes | no | full | The technical data-pipeline boundary is reconstructed as a product boundary, not a policy-only promise. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md:17,369,403,411: accessibility is a complete presentation; narration/reduced motion preserve charm. | PLAN.md:11,15,346,378-390,461-462,523: access ships in first slice; designed narration, captions, keyboard, reduced motion. | yes | yes | no | full | Charm, non-degraded alternate presentations, and launch-time completeness are all preserved. |

For multi-layer system-level whys:

| Why ID | L1 primary | L2 secondary | L3 downstream |
| --- | --- | --- | --- |
| S1 | yes | yes | yes |
| S2 | yes | no | yes |
| S6 | yes | no | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: PLAN.md:9 continuing browser aviary; 117 normal ticks regardless of access; 322 current-phase SVG/no reset; 354 procedural WebAudio/no stored loops; 505 canned-aliveness risk.
- S2: PLAN.md:13 no welcome/counters/badges; 179 narration must not say welcome or announce session; 314 silent visit log; 491 no progress display; 523 no reminders.
- S3: PLAN.md:52 shared semantic observation facts; 199 notebook uses named concrete facts; 374 descriptor-specific captions; 384 narration describes coherent moments; 523 specific observational copy.
- S4: PLAN.md:9 two birds/seven max; 23 four top-bar icons with settle inside offer; 328 one fixed landscape/perch bands; 332 no panning/scene chrome; 362 nonfocused birds remain audible.
- S5: PLAN.md:199 notebook naturalist examples; 230 API errors matter-of-fact; 368 Enable sound in matter-of-fact voice; 280 system emails are transactional; 523 product/system copy split.
- S6: PLAN.md:131-141 exact presence predicate; 149-153 presence-dominant drift; 187 settle/tab-close end presence without penalty; 445-447 predicate and union tests; 523 no chores.
- S7: PLAN.md:25 no second simulation writer; 47 worker sole owner; 100 60-second cadence; 117 ticks regardless of access; 250 one history across devices.
- S8: PLAN.md:63 private state never used for training/analysis; 414-420 aggregate-only metrics; 467 telemetry privacy tests; 489 no dogfood private-event aggregation; 512 analytics boundary risk.
- S9: PLAN.md:11 access ships with first slice; 346 reduced motion before first frame; 380-386 semantic controls/authored narration; 390 human access QA; 523 access complete at launch.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **77.4%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md:177: "visible, focused, recent trusted activity" and "not a tab-open signal". | PLAN.md:131-141,143-157: exact conjunction, bounded intervals, and drift calibration. | no | partial | Predicate precision and tab-open avoidance survive; silent population-wide corruption is not fully reconstructed. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md:191,193,195,467: positive evidence filter, fractional precision, versioned constants, one/three-week calibration. | PLAN.md:145-157,437,501: low-pass state, seven/21-day bands, no single-session jump, tuning risk. | no | partial | Slow filter and calibration survive; Tamagotchi-vs-screensaver failure framing is absent. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md:9,41,197,451: nonpunitive direction, excluded distress/death, absence not penalty. | PLAN.md:145,155,161,436-438: nonnegative deltas, absence never lowers traits, no fabricated attendance. | no | partial | No-negative-drift and no-punishment survive; the quieter-not-mistrust relationship consequence is compressed away. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md:43,113,375,501: call identity from procedural grammar; no audio files or stored waveforms. | PLAN.md:13,352-358,370,455,487: procedural motifs, WebAudio synthesis, no recorded fallback, listening review. | no | partial | Procedural/no-loop rule and downstream no-recording cascade survive; chorus phase-cancel rationale is missing. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md:201: expressive ambient behavior through mood states, "not visible labels, meters". | PLAN.md:167-169,336: mood affects preening, scanning, wariness, drowsiness, and continuous motion. | no | full | The reconstruction preserves mood-as-visible-motion rather than a label/status system. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | The cap is present, but RECONSTRUCTION.md:35 explicitly says NOT RECOVERABLE FROM PLAN for its why. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md:5,109,123,443: stable recognizable individuals, canonical stored traits/drift, raw logs not backup. | PLAN.md:56-58,74,145,426,440: persisted vectors, server tick ownership, migration/restore continuity. | no | full | Persistence, relationship continuity, and multi-device/no-overwrite consequences are recovered. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md:11,59,275: "No numbers to optimize"; encrypted export; no trait vectors under alternate names. | PLAN.md:29,61,216,236,465,523: no numeric vectors in UI/export/plaintext/DOM/ARIA and no optimization. | no | full | The non-exposure rationale survives as protection against stat management. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md:31,209,211,213,215: first return, absence-derived greeting, one greeter, continuous variation. | PLAN.md:175-179,453: arrival event, one bird, absence length/boldness/mood variation, no canned clips. | no | full | One-bird noticing, absence/boldness variation, and anti-canned first-session stakes are recovered. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md:13,31,179,341: no welcome toast/text, no absence counter, no return prompt or badge. | PLAN.md:13,179,314,453,523: no welcome text/toast, bird response only, no reminders. | no | partial | The rule and adjacent variants survive; the exact "system announced me" vs bird-noticed-me rationale is thinner. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md:187,227: settle closes presence; tab-close and settle both end presence without penalty or ritual. | PLAN.md:27,187: settle is a viewing-session gesture; tab-close and settle both end presence without drift penalty. | no | full | Optional settle is recovered as a non-obligatory ritual rather than required ceremony. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md:13,239,241,243,245: specific observational voice, curated factual grammar, truth checks, sparse/read-only. | PLAN.md:199-205,239-245,455: naturalist examples, realized facts, rare budget, no edit/delete/attendance diary. | no | full | Naturalist prose, anti-event-log treatment, rarity, and read-only observer-record behavior are recovered. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md:177,183,453: exact predicate, not tab-open, prevents offline hours and double-counted devices. | PLAN.md:131-141,149-157,445-447: all three signals, server clipping/unioning, drift dominance, test matrix. | no | full | The implementation predicate, signal precision, and silent over-crediting failure mode are recovered. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md:41,197,245,455,481: no counters, no attendance export/language, no progress display, no chores. | PLAN.md:13,201,205,222,491,523: no streaks/counters/calendars, no attendance prose/export, no progress display. | no | full | No visit-frequency surfaces, number-management risk, and adjacent disguises all survive. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md:31,345,347,415,417: already moving first frame, no reset/spinner, quiet field on missing state. | PLAN.md:9,322-324,399-410,453: first-frame SVG in current phase, no spinner/fade, quiet field, real paint metric. | no | full | Continuing first frame, snapshot/hydration rationale, and quiet-field loading consequence are recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md:103,135,249: email encrypted once; never identifier, route, queue partition, metric, or log. | PLAN.md:71,268,418,464: synthetic account IDs, blind lookup, no email/token leaks, CSRF/auth tests. | no | partial | Identifier separation and PII-leak prevention survive; impossible-to-retrofit rationale is absent. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md:3,7,51,77,89,151,153,441: 60-second tick, sole worker, no second writer, one history. | PLAN.md:25,47,100-111,117,123,250: server tick, worker ownership, snapshots, event wakeups, no competing simulation. | no | full | The slow server tick, sync coherence, and client-divergence avoidance are recovered. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md:87,89,287,497: refusal of client absolute trait writes, sole worker, one canonical sequence. | PLAN.md:46-47,57-58,79,226,250,503: no absolute trait writes, append-only events, one history, atomic cursor/vector commit. | no | partial | Server-authored deltas and implementation boundary survive; the laptop/phone overwrite failure example is absent. |
| F19 | sync-conflict-tone | 2 | yes | included | none | none | yes | none | Matter-of-fact error copy is present, but the naturalist-tone-would-feel-evasive why is not reconstructed. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md:15,95,433,437,515: private state never used for training/analysis; metrics cannot touch simulation data. | PLAN.md:63,414-420,467,489,512: forbidden telemetry labels, schema rejection, no production relationship tuning. | no | full | Private relationship-data boundary, observability separation, and pipeline-level enforcement are recovered. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md:329,331,333: same canonical scene, no greeting/presence/offers/settle, no host-state effect. | PLAN.md:308-314,463,521: visitor reads current scene, cannot affect presence/drift, only local access preferences. | no | full | Visitor observation without co-presence or accidental drift is recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md:61,341,519: optional plain email only; off by default; no UI notice/badge; notification-loop response. | PLAN.md:30,314,463,514: silent log by default; optional notice confined to explicitly requested email; engagement loop risk. | no | full | The silent-default visit-notice rationale survives as refusal of a notification loop. |
| F23 | no-leaderboards | 2 | yes | included | RECONSTRUCTION.md:19,39,95: no public discovery/social surfaces; one owner relationship; no social-ranking statistics. | PLAN.md:13,19,414-416,523: no leaderboards/discovery/profiles and no aggregate social-ranking telemetry. | no | full | The reconstruction ties public-social exclusions to protecting the private owner-bird relationship. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md:17,395,403,411: complete presentation, intelligible scene region, authored prose, not raw state lists. | PLAN.md:380-386,390,461: naturalist running prose, polite live region, priority bumps, human access evaluation. | no | full | Narration as prose, equal affective surface, and rejection of ARIA/state-list automation are recovered. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md:17,369,411,462: reduced motion before first frame, semantics preserved, charm human-evaluated. | PLAN.md:346,390,462,523: cross-fades replace motion, calls/captions/moods/offers/drift/notebook remain. | no | full | Reduced motion survives as a designed alternate aviary, not an animations-off fallback. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md:31,417,429,431: first bird actually painted; honest cold mobile evidence; critical path controls. | PLAN.md:399-410,453,508: 500 ms measured to visible nontransparent bird, not placeholders or warm-only success. | no | full | The affective-performance bridge is recovered through real first-bird visibility, not skeleton metrics. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md:41,235,481,519: no rewards/streaks/counters, no attention-earned reward, no progress display. | PLAN.md:13,195,205,491,523: no achievements/streaks/scores/badges, no counters, no progress display, no chores. | no | partial | The no-gamification and number-management rationale survives; future-foothold/product-creep layer is weaker. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md:41,197,523: hunger/distress/death/decay excluded; absence not penalty; no chores to complete. | PLAN.md:13,161,519,523: no death/hunger/distress/happiness decay; continuing nonpunitive absence. | no | full | The product remains observational and non-custodial rather than absence-punishing. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION.md:233: no catalog, rarity, stat comparison, or avatar configuration; avoid optimizing/shopping. | PLAN.md:191-193: two system-chosen birds that arrived; no catalog, rarity, stat comparison, or avatar configuration. | no | full | The first encounter remains meeting particular arrivals, not optimizing a catalog choice. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md:107,235,477,481: age independent of attendance; growth without visit counts or rewards. | PLAN.md:33,73,195,222-224,491: adoption by aviary age, not visit count, score, paid tier, or progress gauge. | no | full | Age-based growth, rejection of reward loops, and economy/progress erosion are recovered. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md:5,109,141,145,499: same specific bird; identity survives rename, migration, restore; no regeneration. | PLAN.md:15,56-57,74,90-92,426,440,519: immutable IDs/signatures and continuity through rename/absence/devices/updates. | no | full | Stable identity is recovered as relationship continuity, not just database consistency. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md:3,201,438: continuing server behavior, no login neutralization/midnight reset, opening does not reset mood. | PLAN.md:9,117,167-169,438,519: mood/environment continue while gone, no neutral reset on open. | no | full | Mood persistence is recovered as part of the aviary continuing while absent. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Read-only notebook mechanics survive, but the observer-record-not-user-journal rationale is not recovered. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | Export mechanics and sealed vectors survive, but the quiet user-owned relationship-copy why is not recovered. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md:311,313,315,319: pending deletion lifecycle, recovery preserving state, hard deletion and key destruction. | PLAN.md:284-290,465-466: 30-day recovery, hard deletion of linked records, key erasure, restore drills. | no | partial | Hard-delete privacy and whole-record cleanup survive; soft-delete regret protection is not explicit. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md:433,437,515: operational health metrics without behavioral analytics; forbidden fields rejected at ingestion. | PLAN.md:414-420,467,512: aggregate counts only; no per-bird/account dimensions; analytics credentials cannot query protected storage. | no | full | The telemetry line is technical and enforced, not merely policy language. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md:19,323,327: deliberately invited read-only visits; named one-time invitations; no global discoverability. | PLAN.md:298,306,308,310: owner supplies recipient email, one-use link, no global flag or implicit sharing. | no | full | Sharing remains a deliberate named act, not an ambient/public setting. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md:63,304,339,341: host transparency, visit log, no badge/UI notice by default. | PLAN.md:31,304,314,319: visit log on demand, no badge/public route, transparency without notification surface. | no | full | The log is recovered as on-demand transparency rather than a social attention loop. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md:329,335: same canonical scene, not a flattering visitor-specific scene. | PLAN.md:308,310,521: visitors observe the real current scene, with no show-off/co-presence additions. | no | full | Visitor view remains the host's actual aviary rather than a staged/share-optimized rendering. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md:403,405: one polite live region, coherent quiet moment, no raw lists, no fighting a queue. | PLAN.md:384-386,390,456,461: approximately 45-second idle cadence, priority only for user events, queue-length QA. | no | partial | Queue and observational-sparsity rationale survives; exact slow cadence/rhythm layer is not reconstructed. |

For multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
| --- | --- | --- | --- |
| F1 | yes | yes | no |
| F2 | yes | yes | no |
| F3 | yes | yes | no |
| F4 | yes | no | yes |
| F7 | yes | yes | yes |
| F9 | yes | yes | yes |
| F10 | yes | no | yes |
| F12 | yes | yes | yes |
| F13 | yes | yes | yes |
| F14 | yes | yes | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | no | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | yes | no |
| F30 | yes | yes | yes |
| F31 | yes | yes | yes |
| F35 | no | yes | yes |
| F40 | no | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
| --- | --- | --- |
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS.json |
| Possible total weight | 152 | Full possible S+F weight |
| Reachable gold whys | 49 | S whys always included; all F anchors reachable here |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 119 / 152 | Weighted numerator over included whys |
| Whys with reconstruction evidence | 44 | Rows with exact reconstruction rationale evidence |
| Whys with PLAN grounding | 45 | Rows with exact PLAN rationale grounding |
| rule_without_why cases | 4 | Captured rule/mechanism without gold why |
| plan_only_not_reconstructed cases | 1 | PLAN preserved rationale but reconstruction did not |
| ungrounded_reconstruction cases | 0 | Reconstruction asserted unsupported rationale |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
| --- | --- | --- | --- |
| Functional whys | 54 | 38 | 70.4% |
| Affective whys | 98 | 81 | 82.7% |
| Weight 2 whys | 44 | 35 | 79.5% |
| Weight 3 whys | 108 | 84 | 77.8% |
| System-level whys | 28 | 23 | 82.1% |
| Feature-level whys (reachable) | 124 | 96 | 77.4% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Affective whys recovered better (82.7%) than functional whys (70.4%). Functional losses concentrate in exact engine/account rationales: F1, F2, F6, F16, F18, F34, and F35.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 77.8%, slightly below weight-2 whys at 79.5%. The larger whys often survived as a primary rule plus one layer, while downstream consequence layers leaked.
- **System-level vs feature-level.** System-level fidelity (82.1%) exceeded feature-level fidelity (77.4%). The plan carried philosophy strongly, but the reconstruction compressed several feature-specific rationales.
- **Multi-layer recovery patterns.** Primary causes survived most often. Secondary/downstream losses appear in S2, S6, F1, F2, F3, F4, F10, F16, F18, F27, F35, and F40.
- **Subdomain patterns.** Accessibility and visits were very strong (F21-F25, F36-F40 mostly full). The weakest areas were rationale behind empirical constants and product-copy exceptions: F6, F19, F33, and F34.
- **Evidence-bound effects.** v06 denied credit where the implementation rule survived without the why: F6, F19, F33, and F34. S4 is plan-only at system level: the plan preserves restraint, but the reconstruction does not name it as a system principle.

Overall, this run shows excellent planning capture and strong intent preservation, with leakage mainly at the level of non-obvious explanation rather than missing implementation requirements.

---

## 4. Recommendations for v2 hardening

- Add more targeted single-layer whys like F33 and F34. They found failures that raw feature capture and broad system principles would miss.
- Keep multi-layer judging, but add a structured layer-evidence field in the template. This would make partial calls for F1/F2/F16/F18/F40 easier to audit consistently.
- Split broad system principles where they invite scorer discretion. S4 restraint and S6 presence were both preserved in PLAN, but the reconstruction compressed them enough that partial/full calls were subjective.
- Continue separating privacy/observability whys from generic security whys. F20 and F36 recovered well because the plan made the technical boundary concrete.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** Accepted from the harness prompt: this scorer had fresh context and did not modify the frozen reconstruction. No subagents or background agents were spawned.
- **Single-run-at-temperature limitation.** This is one run only; no variance signal is available from this slot.
- **Borderline capture calls.** Feature 69 was counted captured under the inclusive rule because the plan specified calmness/palette/contrast, though not the exact saturated-accent exclusion. Feature 87 was missed: no explicit privacy policy link was found.
- **System-level cross-cutting.** S4 and S6 were the hardest calls. The plan preserved them across 3+ decisions, but the reconstruction compressed their system-level statement.
- **Confabulation cases.** No clear ungrounded reconstruction was found. The reconstruction mostly quotes or closely paraphrases PLAN language and uses NOT RECOVERABLE where appropriate.
- **Evidence-bound denials.** F6, F19, F33, and F34 are captured at the feature/rule level but lack reconstructed gold-rationale evidence.
- **Operational timing.** TIMING.json supplied phase1 and phase2a timing only for run 001; phase2b timing was unavailable inside this scorer context and was not fabricated.

End of report.
