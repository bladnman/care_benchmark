# REPORT - CARE run 001

> Phase 2B scoring report for `runs/wave_002/plans/001/PLAN.md` and frozen `runs/wave_002/reconstructions/001/RECONSTRUCTION.md`.
>
> Variant v06 evidence-bound clean + targeted gold headroom: every S1-S9 and F1-F40 row includes denominator status, reconstruction evidence, PLAN grounding, rule-without-why flag, and recovery.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **55.8%** |
| Intent fidelity | **38.4%** |
| Combined quality | **5522** |

**Diagnostic split:**

- System-level fidelity: **39.3%**
- Feature-level fidelity: **38.1%**

**(Planning, fidelity) coordinate:** `(55.8, 38.4)`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-30T13:24:10Z |
| Candidate model | gemma-4-26b-a4b-it |
| Candidate effort | unknown |
| Candidate harness | opencode |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **67 / 120** = **55.8%**.

| File | Total | Captured | Rate |
| --- | --- | --- | --- |
| product_brief.md | 6 | 2 | 33.3% |
| concepts.md | 4 | 2 | 50.0% |
| bird_engine.md | 22 | 16 | 72.7% |
| interactions.md | 20 | 5 | 25.0% |
| aviary_layout.md | 18 | 5 | 27.8% |
| accounts_sync.md | 18 | 12 | 66.7% |
| social_optional.md | 10 | 7 | 70.0% |
| accessibility_perf.md | 18 | 14 | 77.8% |
| non_goals.md | 4 | 4 | 100.0% |
| Total | 120 | 67 | 55.8% |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
| --- | --- | --- | --- | --- |
| 1 | Headline product concept statement | product_brief.md | yes | Captured by the browser-based single-user aviary concept focused on long-term observational relationships. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | no | Mechanisms imply aliveness, but the principle itself is not stated. |
| 3 | "Notice, never announce" principle callout | product_brief.md | no | No notice-vs-announce principle or return-surface rule appears. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | no | Naturalist prose appears, but the matter-of-fact system exception is absent. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in the non-goals. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | no | Two starting birds and one scene are scattered, but the max-seven/restraint statement is missing. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No glossary-level definitions are provided. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | no | Presence is named, but not precisely defined. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Personality vectors are slow drift and mood is a fast state. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Settle is listed as session end and an interaction event. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | The vector traits are enumerated. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | The server tick applies a low-pass filter. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | borderline; The risk table names 1-week/3-week drift targets. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | The drift formula clamps negative signals to zero. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Mood is a fast state with timestamped daily reset/modulation. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Mood transition inputs are listed. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Motifs are synthesized client-side with WebAudio. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Audio recognizability is explicitly a risk/alpha target. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | borderline; Ambient mix and listen-in floor are specified; loop avoidance is in risk mitigation. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Pitch and timing vary by vocal_frequency and mood. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Mood-driven idle animations include preening, scanning, and fluffing. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Mood-driven micro-motion is explicit. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | no | Only species_id is referenced; no v1 pool size. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | borderline; The bird name is user-assigned; renameability is not specified. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | no | Two birds at launch are mentioned, but not adoption or auto-selection. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | no | No seven-bird cap appears. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | no | No unlock rule appears. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Server authority and canonical state preserve personality/mood. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | borderline; Current mood and mood timestamp are persisted as server state. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | no | Ambient chorus exists, but bird-to-bird reactions are not specified. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | bird_id is a stable UUID. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | no | No UI-hiding rule for trait values appears. |
| 33 | Return-greeting on viewer arrival | interactions.md | no | No arrival greeting appears. |
| 34 | Greeting variation by absence length | interactions.md | no | Absent. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | Absent. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | no | Absent. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No textual welcome prohibition appears. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Listen-in raises the focused bird gain. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Other birds decrease toward an ambient floor. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | no | Offer is named, but the concrete offer types are absent. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | Absent. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | no | Absent. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | no | Settle is named as session end, but the lighting gesture is absent. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | Closing-tab equivalence is absent. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Auto-generated naturalist observation log and notebook generation are specified. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | no | Significant state shifts are mentioned, but rarity/frequency is not. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | no | Read-only is absent. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Presence history and presence-time drive drift. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | no | The three-signal conjunction is absent. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | borderline; No streaks is explicit; days-visited is inferred from the same non-goal. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | Snapshot pull/rendering exists, but no background-tab pause rule. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | Absent. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | borderline; Single horizontal scene is explicit; no-panning is implicit. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Three front/middle/back depth layers are specified. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | no | Absent. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Day/night cycle uses local time. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | no | Lighting shifts exist, but evening warmth and quieter calls are absent. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | Absent. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | borderline; Ambient weather is named, but not rain/wind details. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Ambient weather feeds mood transitions. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | no | Absent. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | no | Absent. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | no | Absent. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | no | Absent. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | no | Absent. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | no | No first-frame/mid-action load rule. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | no | Absent. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | Absent. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | Absent. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | no | Absent. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Magic-link email auth is in scope and has an API endpoint. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | Absent. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Single-user and single canonical aviary per account are explicit. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Synthetic UUID and encrypted one-copy email are specified. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | The server-side tick cadence is specified. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | borderline; Client snapshot pull is captured; on-visibility is not explicit. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Interpolation is specified. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Phone and laptop pull the same server record. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | No last-write-wins and additive deltas are explicit. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Server sole writer and event ordering are explicit. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | no | No sync conflict copy surface or tone rule. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | no | Absent. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | borderline; POST /account/export requests a JSON snapshot via email. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | no | Absent. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | borderline; Aggregate-only RUM forbids PII/per-bird data, though ML is not named. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Aggregate-only latency/error/session metrics and no per-bird data are explicit. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | Absent. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | Absent. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Email invitation and revocation endpoints are present. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | borderline; Visit is opt-in. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Visit is opt-in, read-only, ambient, and no co-presence. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | borderline; Read-only/no co-presence implies no visitor-triggered interactions. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | no | No social-network surfaces is broad, but chat/comments/avatars are not specified. |
| 94 | No "your friend visited!" notification by default | social_optional.md | no | Absent. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | DELETE invite endpoint is present. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | no | Absent. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes | borderline; Read-only ambient visit implies host aviary view; show-off mode is not explicit. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Discovery feeds and public profiles are excluded. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Screen-reader narration is naturalist prose. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Narration updates every 30-60s. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Narration uses naturalist prose. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Reduced motion uses slow alpha-blended cross-fades. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | no | Cross-fades are present, but preserved-charm rationale is absent. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | borderline; Call captioning and localized descriptions are present; toggle is not specified. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | no | Absent. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Keyboard focus management is specified. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | no | Absent. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Initial JS <2MB is explicit. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | First bird visible <500ms on 4G mid-tier mobile is explicit. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | borderline; 60fps idle motion is explicit; device class is not. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Zero growth over 30 minutes is explicit. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Client WebAudio procedural synthesis is explicit. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Graceful Silence with mandatory captions is explicit. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Synthetic monitoring and aggregate-only RUM are specified. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | P99 tick latency >5s alerting is specified. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | Absent. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Web-only for v1 is explicit. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Gamification non-goal is explicit. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Tamagotchi mechanics are explicitly out of scope. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Social network surfaces are explicitly out of scope. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **39.3%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | B identified? | PLAN cross-cutting? | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md:19 procedural, mood-shaped audiovisual expression across bird engine/audio. | PLAN.md:8,92,109,114,151 procedural calls, mood-shaped motion, WebAudio, no loops. | yes | yes | no | partial | Recovered the procedural/mood-shaped layer, but not the continuing-without-viewer or staleness consequence. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md:3,75 long-term relationships/no game progression; no notice-vs-announce rationale. | PLAN.md:17,19 no gamification and no public/social-network surfaces; no toast/notification rule. | no | no | yes | none | Mechanism-level refusals survived, but the notice/announcement affective principle did not. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md:13 naturalist prose as product voice across notebook and accessibility. | PLAN.md:12,121,122 naturalist observation log, narration, and specific call captions. | yes | yes | no | partial | Naturalist prose survived; specificity as the charm engine and anti-generic rationale did not. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md:17 performance budgets and lightweight web experience; no richness/restraint principle. | PLAN.md:9,14,142 single scene, performance budgets, two launch birds; max-seven/calm-palette rationale absent. | no | no | yes | none | Some restraint-shaped rules exist, but not the depth-over-variety why. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md:13 naturalist prose as part of the product voice. | PLAN.md:12,121 naturalist notebook and narration; no matter-of-fact system exception. | yes | no | no | partial | Recovered naturalist voice, but missed the account/error/settings exception. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md:27 presence-based idle watching drives presence_history/presence-time; 5 no punishment for neglect. | PLAN.md:7,54,82,89 presence interactions, presence_history, drift inputs, zero negative drift. | yes | yes | no | partial | Recovered idle attention as drift input, but not the three-signal precision or settle/close equivalence. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md:7 server-side canonical truth/no merging/no last-write-wins; 197 additive deltas. | PLAN.md:29-31,77-85,97-102 server authority, tick, snapshots, additive no-LWW sync. | yes | yes | no | partial | Recovered server canonical truth and multi-device coherence; missed the failure-mode consequence. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md:11 privacy mitigation with synthetic UUIDs and telemetry decoupling; 251 no PII or per-bird data. | PLAN.md:135,154 aggregate RUM with no PII/per-bird data and decoupled telemetry pipelines. | yes | yes | no | partial | Technical privacy boundary survived, but the relationship-as-private-data rationale was not reconstructed. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md:15 accessibility surfaces are core features; 61 naturalist prose narration and core accessibility. | PLAN.md:13,111,118,121-123,152 narration, reduced motion, captions, keyboard, and CI/CD as core. | yes | yes | no | partial | Recovered first-class/core accessibility and some designed surfaces; missed charm-not-checklist and launch exclusion rationale. |

Multi-layer system-level recovery:

| Why ID | L1 | L2 | L3 |
| --- | --- | --- | --- |
| S1 | N | Y | N |
| S2 | N | N | N |
| S6 | Y | N | N |
| S7 | Y | Y | N |
| S9 | N | N | Y |

**Cross-cutting evidence appendix:**

- S1: PLAN.md:8 bird engine primitives; 92 WebAudio procedural calls; 109 mood-driven motion; 114 synthesis; 151 procedural-over-loops risk mitigation.
- S2: PLAN.md:17 no streaks/scores/levels/achievements/badges; 19 no profiles/follows/discovery/public profiles. Fewer than three notice/announcement-specific inheritances.
- S3: PLAN.md:12 naturalist observation log; 121 naturalist prose narration; 122 specific call captions.
- S4: PLAN.md:9 single horizontal scene; 14 performance budgets; 142 two birds per account. The depth-over-variety/restraint rationale is not carried.
- S5: PLAN.md:12 and 121 naturalist prose surfaces. The system-error/account/settings exception is absent.
- S6: PLAN.md:7 presence-based idle watching; 54 presence_history; 82 drift input; 89 no negative drift; 17 no streaks.
- S7: PLAN.md:29 server authority; 30 client projector; 31 append-only interactions; 77-85 server tick; 97-102 no merging/no LWW.
- S8: PLAN.md:39 synthetic UUID; 40 encrypted email once; 135 aggregate RUM no PII/per-bird; 154 decoupled telemetry pipeline.
- S9: PLAN.md:13 accessibility surfaces in scope; 111 reduced-motion cross-fades; 118 captions fallback; 121-123 narration/captions/keyboard; 152 accessibility as core CI/CD feature.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **38.1%**.

Reachable feature-level whys: **26 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1 | presence-definition | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured: PLAN names presence but does not define the three-signal conjunction. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md:35 low-pass filter based on presence-time and interaction weights; 267 one-week/three-week targets. | PLAN.md:82 low-pass filter over presence-time/interactions; 149 one-week/three-week targets. | no | partial | Low-pass drift survived; instruments-vs-user calibration and Tamagotchi/screensaver failure modes did not. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md:5 monotonic toward expressive/no punishment; 179-181 clamp produces zero change. | PLAN.md:18 no punishment for neglect; 88-89 clamp negative signals to zero. | no | partial | Captured no-negative-drift and no-punishment; missed the quieter-not-mistrusting return consequence. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md:39 motif/WebAudio synthesis; 219 procedural synthesis over loops for recognizability, variety, uncanniness. | PLAN.md:91-92 motifs synthesized with WebAudio; 114 synthesis; 151 prioritize procedural synthesis over loops. | no | partial | Recovered procedural-not-looped audio and WebAudio cascade; missed chorus phase-canceling and audio-spine rationale. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md:41 rule only: visible behavior expresses current mood. | PLAN.md:109 continuous mood-driven idle animations. | yes | none | Mechanism survived, but the user-reads-mood-without-label rationale did not. |
| F6 | bird-count-cap-7 | 2 | no | unreachable_excluded | none | none | no | unreachable | Max-seven feature not captured. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md:89 server owns canonical state; 193 no merging; 197 additive deltas. | PLAN.md:29-31 server authority/projector/log; 97-101 same record and additive deltas. | no | partial | Canonical server ownership survived; deleting-the-known-bird rationale did not. |
| F8 | vector-never-shown-numerically | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured; no UI-hiding rule for trait values. |
| F9 | return-greeting | 4 | no | unreachable_excluded | none | none | no | unreachable | Return greeting feature not captured. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | No welcome-toast/banner feature not captured. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Settle close-tab equivalence not captured; reconstruction also marks settle rationale not recoverable. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md:59 generated from significant state shifts using naturalist prose observation voice; 173 significant state shifts. | PLAN.md:12 naturalist-voice observation log; 55 prose_text; 84 notebook generation. | no | partial | Naturalist observation prose survived; rarity/read-only/feed-not-journal rationale did not. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md:27 rule only: presence_history and presence-time feed drift calculations. | PLAN.md:54 aggregated presence-time; 82 presence-time as drift input. | yes | none | The accounting mechanism survived, but the three-signal precision and silent-corruption why did not. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md:75 no streaks/scores protect long-term observational relationships from game progression. | PLAN.md:4 long-term observational relationships; 17 no streaks/scores/levels/achievements/badges. | no | partial | Recovered anti-game-progress direction; missed counter-not-birds and adjacent-disguise layers. |
| F15 | scene-loads-with-motion | 4 | no | unreachable_excluded | none | none | no | unreachable | Scene-loads-mid-motion feature not captured. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md:101 synthetic UUID as privacy mitigation; 103 email encrypted and stored once. | PLAN.md:39 account_id synthetic UUID; 40 email encrypted; 154 privacy-leak mitigation. | no | partial | UUID/email separation survived; retrofit/compliance failure rationale did not. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md:89 server owns canonical state/tick; 165-175 tick process; 191-195 same account/no merge. | PLAN.md:77-85 tick cadence/process; 97-98 same record and no merging. | no | partial | Server tick and sync coherence survived; client-tick collapse consequence did not. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md:197 additive deltas instead of absolute client values; 199 prevents client overwrites. | PLAN.md:101 additive deltas and no absolute personality values; 102 ordered tick processing. | no | partial | Implementation rule survived; the silent deleted-drift failure story did not. |
| F19 | sync-conflict-tone | 2 | no | unreachable_excluded | none | none | no | unreachable | Sync conflict tone feature not captured. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md:251 aggregate-only metrics with no PII or per-bird data; 277 telemetry pipelines physically decoupled. | PLAN.md:135 aggregate-only RUM with no PII/per-bird data; 154 decoupled telemetry pipelines. | no | partial | Technical no-per-bird telemetry boundary survived; relationship/private-data-product rationale did not. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md:57 rule only: opt-in, read-only, ambient Visit with no co-presence. | PLAN.md:11 opt-in, read-only, ambient Visit feature with no co-presence. | yes | none | Read-only visit rule survived, but the no-multi-user-simulation/no-visitor-drift rationale did not. |
| F22 | no-friend-visited-notification | 2 | no | unreachable_excluded | none | none | no | unreachable | No friend-visited notification feature not captured. |
| F23 | no-leaderboards | 2 | yes | included | RECONSTRUCTION.md:79 rule only: social remains opt-in/read-only ambient rather than public network. | PLAN.md:19 no profiles, follows, discovery feeds, or public profiles. | yes | none | Public/discovery surfaces were refused, but comparison/metrics rationale was not recovered. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md:61 naturalist prose stream and core accessibility; 229 naturalist prose updates every 30-60s. | PLAN.md:121 separate audio/textual stream of naturalist prose updates. | no | partial | Naturalist prose narration survived; same-right-to-feel and ARIA-automation warning did not. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md:63 slow alpha-blended cross-fades; 215 replaces frame-by-frame animation with cross-fades. | PLAN.md:111 reduced motion replaces animation with slow alpha-blended cross-fades. | no | partial | Cross-fade rendering survived; still-the-same-aviary and stripped-fallback rationale did not. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md:69 rule only: first bird visible under 500ms on 4G mid-tier mobile. | PLAN.md:129 LCP/first bird visible <500ms on 4G mid-tier mobile. | yes | none | Performance budget survived, but the affective-perf threshold rationale did not. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md:75 no gamification protects long-term observational relationships from game progression. | PLAN.md:4 long-term observational relationships; 17 no streaks/scores/levels/achievements/badges. | no | partial | Recovered the anti-game-progression primary direction; missed temptation and future-erosion layers. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md:3 observational relationships; 77 no punishment for neglect and monotonic drift. | PLAN.md:4 observational relationships; 18 no mortality/hunger/distress and no punishment for neglect; 89 zero negative drift. | no | full | The reconstruction recovered observational-not-punitive mechanics closely enough for the single-layer why. |
| F29 | starter-birds-not-catalog | 2 | no | unreachable_excluded | none | none | no | unreachable | Starter-birds adoption flow not captured. |
| F30 | age-based-bird-offers | 4 | no | unreachable_excluded | none | none | no | unreachable | Age-based new-bird offer feature not captured. |
| F31 | stable-bird-identity | 4 | yes | included | none (RECONSTRUCTION.md:109 marks bird_id stable UUID NOT RECOVERABLE FROM PLAN). | PLAN.md:44 bird_id is a stable UUID. | yes | none | Critical rule applied: reconstruction marked the stable-ID rationale not recoverable. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md:117-119 mood is fast state and last_mood_timestamp supports daily reset/modulation. | PLAN.md:48 current_mood; 49 last_mood_timestamp for daily reset/modulation. | yes | none | Mood state survived, but no-neutral-reset/continuity rationale did not. |
| F33 | notebook-read-only-observer-record | 2 | no | unreachable_excluded | none | none | no | unreachable | Read-only notebook feature not captured. |
| F34 | account-export-relationship-copy | 2 | yes | included | none (RECONSTRUCTION.md:149 marks POST /account/export NOT RECOVERABLE FROM PLAN). | PLAN.md:68 POST /account/export requests JSON snapshot via email. | yes | none | Export endpoint was captured in PLAN, but the reconstruction explicitly marked the why not recoverable. |
| F35 | account-deletion-grace-then-hard-delete | 4 | no | unreachable_excluded | none | none | no | unreachable | Account deletion feature not captured. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md:251 aggregate-only metrics with no PII or per-bird data; 277 telemetry physically decoupled from simulation DB. | PLAN.md:135 aggregate-only metrics, no PII/per-bird data; 154 telemetry pipelines physically decoupled. | no | full | The technical aggregate-only boundary was recovered with plan grounding. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md:151 rule only: POST /social/invite supports opt-in email invitation; 153 revocation preserves opt-in control. | PLAN.md:69 send email invitation; 70 revoke invitation. | yes | none | Named invite mechanics survived, but private-relationship/control rationale did not. |
| F38 | visit-log-on-demand-transparency | 2 | no | unreachable_excluded | none | none | no | unreachable | Visit log feature not captured. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md:57 rule only: opt-in, read-only, ambient Visit with no co-presence. | PLAN.md:11 opt-in, read-only, ambient Visit feature. | yes | none | Actual-aviary/no-show-off rationale was not recovered. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md:229 naturalist prose updates every 30-60s. | PLAN.md:121 naturalist prose updates one per 30-60s. | no | partial | Slow cadence survived; screen-reader queue and announcement-monitoring failure modes did not. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
| --- | --- | --- | --- |
| F1 | N | N | N |
| F2 | Y | N | N |
| F3 | Y | Y | N |
| F4 | Y | N | Y |
| F7 | Y | N | Y |
| F9 | N | N | N |
| F10 | N | N | N |
| F12 | Y | N | N |
| F13 | N | N | N |
| F14 | Y | N | N |
| F15 | N | N | N |
| F16 | Y | Y | N |
| F17 | Y | Y | N |
| F18 | Y | N | Y |
| F20 | Y | N | Y |
| F24 | Y | N | N |
| F25 | Y | N | N |
| F27 | Y | N | N |
| F30 | N | N | N |
| F31 | N | N | N |
| F35 | N | N | N |
| F40 | Y | N | N |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
| --- | --- | --- |
| Possible gold whys | 49 | From gold_why_totals |
| Possible total weight | 152 | Full-instance possible weight |
| Reachable gold whys | 35 | S whys always included; F whys only when captured |
| Excluded unreachable feature whys | 14 | Denominator exclusions |
| Recovered / reachable weight | 43.0 / 112.0 | Weighted numerator / denominator |
| Whys with reconstruction evidence | 33 | Rows with non-none reconstruction evidence |
| Whys with PLAN grounding | 35 | Rows with non-none PLAN grounding |
| rule_without_why cases | 12 | Mechanism survived without why |
| plan_only_not_reconstructed cases | 0 | PLAN rationale absent from reconstruction |
| ungrounded_reconstruction cases | 0 | Rationale asserted without PLAN grounding |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
| --- | --- | --- | --- |
| Functional whys | 44.0 | 19.0 | 43.2% |
| Affective whys | 68.0 | 24.0 | 35.3% |
| Weight 2 whys | 28.0 | 7.0 | 25.0% |
| Weight 3 whys | 84.0 | 36.0 | 42.9% |
| System-level whys | 28.0 | 11.0 | 39.3% |
| Feature-level whys (reachable) | 84.0 | 32.0 | 38.1% |

---

## 3. Diagnostic patterns

- Affective vs functional: Functional whys recovered at 47.7% (21.0/44.0), while affective whys recovered at 32.4% (22.0/68.0). The plan was strongest where architecture carried the intent: S7, F17, F18, F36.
- Weight-3 vs weight-2: Weight-3 whys did better (36.0/84.0 = 42.9%) than weight-2 whys (7.0/28.0 = 25.0%), but mostly through partial primary-layer recovery.
- System vs feature: System-level fidelity was 39.3%; feature-level fidelity was 38.1%. The planner did not clearly preserve product philosophy more than feature rationale.
- Multi-layer patterns: Primary mechanisms often survived; secondary calibration and downstream consequences usually disappeared. F2, F4, F17, F18, F20, F24, F25, F27, and F40 show this shape.
- Subdomain patterns: Accounts/sync and accessibility/performance were captured best. Interactions and aviary layout lost many surface-specific details, especially greetings, settle nuance, loading state, and UI chrome.
- Evidence-bound effects: The stricter operator denied credit where only rules survived: F5, F13, F21, F23, F26, F31, F32, F34, F37, and F39.

---

## 4. Recommendations for v2 hardening

- Keep the evidence-bound operator for v2; it cleanly separated broad mechanism capture from why recovery in this run.
- Preserve targeted headroom whys like F29-F40. Stable identity, export, visit log, and per-invite sharing revealed meaningful compression.
- Add an optional diagnostic bucket for "named but underspecified" captures. Several inclusive calls were borderline and did not deserve why credit.
- Retain multi-layer whys, because most failures here were second/third-layer losses rather than total absence.
- Consider testing another product with less architecture-heavy scaffolding to see whether functional recovery remains consistently easier than affective recovery.

---

## 5. Methodology caveats

- Fresh-context fidelity: The reconstruction was treated as frozen. The validity audit found no significant contamination signs.
- Single-run limitation: This is one candidate run with no variance signal.
- Borderline capture calls: Inclusive capture affected #13, #19, #24, #29, #50, #53, #59, #76, #83, #85, #90, #92, #97, #104, and #110. These did not inflate fidelity unless the reconstruction and plan carried why evidence.
- System-level cross-cutting: The three-feature bar was most subjective for S3, S4, S8, and S9. I applied it strictly and documented the evidence appendix above.
- Confabulation cases: No major ungrounded reconstruction was counted for credit; generic inferences were either grounded in the plan or scored as rule-only/none.
- Evidence-bound denials: Several plausible v1-style recoveries were denied because the rationale was absent even when a rule existed.

---

End of report.
