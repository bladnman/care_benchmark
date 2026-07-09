# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. This report scores the frozen run 001 reconstruction against the gold whys and preserves denominator status separately from recovery.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **95.0%** |
| Intent fidelity | **85.5%** |
| Combined quality | **9486** |

**Diagnostic split:**

- System-level fidelity: **92.9%**
- Feature-level fidelity: **83.9%**

**(Planning, fidelity) coordinate:** `95.0, 85.5`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-07-09T18:44:54Z |
| Candidate model | gpt-5.6-terra |
| Candidate effort | extra-high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **114 / 120** = **95.0%**.

| File | Total | Captured | Rate |
|---|---|---|---|
| product_brief.md | 6 | 6 | 100.0 |
| concepts.md | 4 | 3 | 75.0 |
| bird_engine.md | 22 | 21 | 95.5 |
| interactions.md | 20 | 20 | 100.0 |
| aviary_layout.md | 18 | 15 | 83.3 |
| accounts_sync.md | 18 | 17 | 94.4 |
| social_optional.md | 10 | 10 | 100.0 |
| accessibility_perf.md | 18 | 18 | 100.0 |
| non_goals.md | 4 | 4 | 100.0 |
| Total | 120 | 114 | 95.0 |


Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured by the implementation plan. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured by the implementation plan. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured by the implementation plan. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured by the implementation plan. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured by the implementation plan. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured by the implementation plan. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No implementation surface for the glossary/domain-term feature was planned. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured; gold why anchor F1 is reachable. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured by the implementation plan. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured by the implementation plan. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured by the implementation plan. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured; gold why anchor F2 is reachable. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured by the implementation plan. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured; gold why anchor F3 is reachable. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured by the implementation plan. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured by the implementation plan. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured; gold why anchor F4 is reachable. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured by the implementation plan. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured by the implementation plan. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured by the implementation plan. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured by the implementation plan. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured; gold why anchor F5 is reachable. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | no | Species IDs and species arrivals appear, but no v1 species-pool size was specified. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured by the implementation plan. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured; gold why anchor F29 is reachable. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured; gold why anchor F6 is reachable. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured; gold why anchor F30 is reachable. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured; gold why anchor F7 is reachable. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured; gold why anchor F32 is reachable. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured by the implementation plan. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured; gold why anchor F31 is reachable. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured; gold why anchor F8 is reachable. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured; gold why anchor F9 is reachable. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured by the implementation plan. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured by the implementation plan. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured by the implementation plan. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured; gold why anchor F10 is reachable. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured by the implementation plan. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured by the implementation plan. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured by the implementation plan. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured by the implementation plan. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured by the implementation plan. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured by the implementation plan. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured; gold why anchor F11 is reachable. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured; gold why anchor F12 is reachable. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured by the implementation plan. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured; gold why anchor F33 is reachable. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured; gold why anchor F13 is reachable. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured by the implementation plan. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured; gold why anchor F14 is reachable. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | borderline: Borderline inclusive: visibility/suspension behavior plus server simulation imply background-tab handling. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured by the implementation plan. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured by the implementation plan. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured by the implementation plan. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured by the implementation plan. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured by the implementation plan. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured by the implementation plan. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured by the implementation plan. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured by the implementation plan. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured by the implementation plan. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured by the implementation plan. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | no | Layered scene is planned, but subtle foreground/background parallax is not specified. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured by the implementation plan. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured by the implementation plan. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured by the implementation plan. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured; gold why anchor F15 is reachable. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured by the implementation plan. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | Empty-aviary state is not planned; v1 starts with two birds. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | Palettes are mentioned for states/contrast, but no calm naturalist palette spec or accent-color rule is planned. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured by the implementation plan. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured by the implementation plan. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured by the implementation plan. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured by the implementation plan. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured; gold why anchor F16 is reachable. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured; gold why anchor F17 is reachable. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured by the implementation plan. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured by the implementation plan. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured by the implementation plan. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured; gold why anchor F18 is reachable. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured by the implementation plan. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured; gold why anchor F19 is reachable. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured by the implementation plan. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured; gold why anchor F34 is reachable. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured; gold why anchor F35 is reachable. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured; gold why anchor F20 is reachable. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured; gold why anchor F36 is reachable. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | Privacy settings/data boundaries are planned, but no privacy-policy link surface is specified. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured by the implementation plan. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured; gold why anchor F37 is reachable. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured by the implementation plan. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured; gold why anchor F21 is reachable. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured by the implementation plan. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured by the implementation plan. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured; gold why anchor F22 is reachable. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured by the implementation plan. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured; gold why anchor F38 is reachable. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured; gold why anchor F39 is reachable. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured; gold why anchor F23 is reachable. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured; gold why anchor F24 is reachable. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured; gold why anchor F40 is reachable. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured by the implementation plan. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured; gold why anchor F25 is reachable. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured by the implementation plan. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured by the implementation plan. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured by the implementation plan. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured by the implementation plan. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured by the implementation plan. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured by the implementation plan. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured; gold why anchor F26 is reachable. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured by the implementation plan. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured by the implementation plan. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured by the implementation plan. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured by the implementation plan. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured by the implementation plan. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured by the implementation plan. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured by the implementation plan. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured by the implementation plan. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured; gold why anchor F27 is reachable. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured; gold why anchor F28 is reachable. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured by the implementation plan. |


### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **92.9%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md line 5: "already alive on first render"; line 303: loading states break the continuing conceit. | PLAN.md lines 51-53: birds render immediately; slow snapshots show a quiet field, never a spinner. | yes | yes | no | full | The reconstruction preserved the living-place conceit across first paint, motion, audio, and fallback behavior. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md lines 7 and 57: no urgency surfaces; visit notifications are opt-in and not badge/onboarding surfaces. | PLAN.md lines 16, 178, and 184: no engagement notifications, bird-led greeting, age-based adoption. | yes | yes | no | partial | The no-announcement rules survived, but the processed-vs-seen rationale and cumulative-toast danger were not fully reconstructed. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md line 11: aviary copy is lower-case, present-tense, specific naturalist observation. | PLAN.md lines 18-21 and 182: specific naturalist copy for greetings, notebooks, narration, offers, captions. | yes | yes | no | full | Specific naturalist voice survived as a cross-surface product rule. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md lines 29, 55, 221, and 228: two-bird/one-screen scope, no scene controls, seven-bird recognizability. | PLAN.md lines 5, 16, 184, 200, and 228: two birds, max seven, one screen, no pan/zoom, recognizability ceiling. | yes | yes | no | full | The restraint principle survived through bird count, layout, top-bar, listen-in, and negative-space constraints. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md line 11: two copy domains split aviary naturalist copy from direct system copy. | PLAN.md lines 18-21, 95, and 105: naturalist aviary surfaces; system-domain auth, settings, export, deletion, errors. | yes | yes | no | full | The copy split was explicitly reconstructed and cross-cutting in the plan. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md lines 187-195: visible/focused/recent activity, fail-closed validation, settle equivalence, nonnegative drift. | PLAN.md lines 142-146 and 150-159: qualified presence, server validation, settle equivalence, calibrated drift. | yes | yes | no | full | Presence survived as precise attention accounting, dominant drift input, and non-punitive session ending. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md lines 3, 181-183, and 299: canonical server-owned state, cursor/lock transaction, no last-write-wins patches. | PLAN.md lines 45, 130-138, and 317: simulation worker owns state, serialized ticks, no LWW/API patches. | yes | yes | no | full | Server-side canonical simulation and multi-device conflict avoidance were fully preserved. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md lines 13, 65, 267, and 315: telemetry separation, aggregate counters, no analytics/ML path. | PLAN.md lines 27, 57, 260-267, and 283: no warehouse path, email single-home rule, aggregate-only telemetry. | yes | yes | no | full | The privacy boundary survived technically, not merely as policy language. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md lines 17, 51, 253-259, and 307-309: parallel primary experience, living aviary, naturalist narration, cross-fades. | PLAN.md lines 13, 238-256, and 331: accessibility ships with narration, captions, keyboard, contrast, reduced motion. | yes | yes | no | full | Accessibility survived as an affective first-class surface, not a checklist fallback. |


For multi-layer system-level whys:

| Why ID | L1 primary | L2 secondary | L3 downstream |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | no |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |


**Cross-cutting evidence appendix:**

- S1: PLAN lines 51-53 first-render/quiet-field; 172-174 scene projection/call seeds; 226-230 procedural audio; 238-256 accessibility surfaces.
- S2: PLAN lines 16 no engagement notifications/streaks; 178 bird-led greeting; 182 notebook cannot report visit frequency; 184 age-only adoption; 216 fading top bar.
- S3: PLAN lines 18-21 copy domains; 182 notebook naturalist facts; 238 shared formatter; 253 naturalist narration; 333 no raw trait UI.
- S4: PLAN lines 5 two-bird one-screen place; 16 no scene customization/panning; 184 max seven; 200 top bar above scene; 228 seven-bird recognizability.
- S5: PLAN lines 18-21 copy domains; 95 matter-of-fact auth/sync errors; 105 deletion/export system-domain; 250 system errors use direct copy.
- S6: PLAN lines 142-146 presence qualification and settle equivalence; 150-159 presence-driven drift; 182 no attendance/streak reporting; 315-316 drift/presence risks.
- S7: PLAN lines 45 server owns durable simulation; 130-138 serialized tick/cursor; 177-185 server tick and catch-up; 317 no LWW/API patches.
- S8: PLAN lines 27 no warehouse query path; 57 email single-home; 260-267 service boundaries; 283 aggregate-only RUM.
- S9: PLAN lines 13 designed accessibility; 238 shared prose formatter; 242-250 narration; 254 reduced-motion cross-fades; 331 accessibility users get a designed living aviary.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **83.9%**.

Reachable feature-level whys: **40 / 40**. No feature-level why anchors were denominator-excluded.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md lines 187, 189, and 297: visible + focused + recent activity; overcount corrupts personality/trust. | PLAN.md lines 142-144: all required signals must qualify; frozen tabs fail closed. | no | partial | Precise conjunction and overcount risk survived; the silent load-bearing failure over weeks was thinner. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md lines 157, 197, and 295: low-pass additive drift, seven-day/three-week calibration, optimization/guilt failure. | PLAN.md lines 157-159 and 315: low-pass bounded update, one-week/three-week tests, failure modes. | no | full | All calibration layers survived. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md lines 7, 195, and 295: no absence punishment; drift cannot be negative; absence-tied drift creates guilt. | PLAN.md lines 23, 157, and 315: no urgency, no negative neglect drift, no absence-tied optimization/guilt. | no | full | The no-punishment exception survived clearly. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md lines 241, 245, and 305: stable procedural signatures, no recorded clips, looped/leaky audio harms aliveness. | PLAN.md lines 226-230 and 234: grammar/signature tests, no recorded fallback, bounded audio profiling. | no | partial | No-repeat/procedural and fallback cascade survived; the chorus phase-artifact rationale was not reconstructed. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md line 229: micro-actions combine so behavior expresses mood. | PLAN.md line 212: wary/content/curious/drowsy birds express mood through motion. | no | full | Recovered as mood shown behaviorally rather than labeled. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md line 228: seven-bird recognizability ceiling. | PLAN.md line 228: listen-in preserves a seven-bird recognizability ceiling. | no | full | The cap rationale survived as audio/identity recognizability. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md lines 113, 291, and 299: server-only personality values, no retroactive rebuild, no LWW patches. | PLAN.md lines 68, 157, and 309: server-only vector updates, calibration version, never recompute historical vectors. | no | partial | Canonical persistence and downstream sync survived; the user-felt deletion rationale was mostly lost. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md lines 9 and 145: personality is hidden and values stay out of host/visitor responses. | PLAN.md lines 16, 95, and 238: no trait UI, hidden values omitted, narration never exposes numeric traits. | no | full | Recovered as protecting behavioral expression from stat-dashboard treatment. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md line 207: one ephemeral bird-led greeting derives from absence, mood, traits, and scene. | PLAN.md line 178: primary bird selected from absence duration, mood, boldness/social warmth, and scene. | no | partial | Selection and variation survived; the anti-canned-cue downstream rationale was thin. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md lines 57 and 207: no text welcome; notification-like behavior must not become badge/onboarding. | PLAN.md lines 16 and 178: no push/email engagement notifications; greeting is visual/audio/narration, never text welcome. | no | partial | The rule survived, but the announced-at-vs-noticed rationale did not. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md line 191: settle ends presence like loss of attention and grants neither penalty nor missing-step state. | PLAN.md line 146: closing without settle is semantically equivalent and earns no penalty. | no | full | Recovered as optional ritual rather than required goodbye. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md lines 182, 211, and 311: naturalist templates, spaced observations, anti-log controls. | PLAN.md lines 74 and 182: sparse naturalist prose from noteworthy facts, not attendance/feed/raw traits. | no | full | Voice, rarity, and anti-feed layers survived. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md lines 187, 189, and 297: visible/focused/recent state machine, fail-closed validation, overcount corrupts trust. | PLAN.md lines 142-144 and 150-159: qualified pings, server validation, calibrated drift. | no | full | Recovered with precise implementation and population-drift risk. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md lines 7, 211, and 319: no urgency/optimization, no attendance/streak reporting, stat guardrails. | PLAN.md lines 16, 182, and 333: no streaks/scores/adoption counters; notebook cannot report visit-frequency behavior. | no | full | Recovered as a broad refusal of engagement-number surfaces. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md lines 5, 95, 227, and 303: already alive, quiet field, first paint mid-action, no loading state. | PLAN.md lines 51-53 and 208: immediate pose render; quiet field, no spinner/modal/entry transition. | no | full | All first-frame and quiet-field rationale survived. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md line 99: email is never identifier, event attribute, trace key, queue key, partition key, or telemetry label. | PLAN.md line 57: email is encrypted once and never an identifier or telemetry label. | no | full | Recovered as a technical PII containment rule. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md lines 177, 181, and 183: minute scheduler, row lock, cursor, deterministic retries. | PLAN.md lines 130-138: minute scheduler, per-aviary lock, cursor, retry-safe deterministic intervals. | no | full | Server-side tick and conflict-avoidance rationale survived. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md line 299: server-only writes, append-only ordered events, per-aviary cursor/lock, no last-write-wins patches. | PLAN.md lines 45, 111, and 317: simulation worker writes personality; host endpoint appends; no LWW/API patches. | no | full | Recovered as the implementation that makes server-side state coherent. |
| F19 | sync-conflict-tone | 2 | yes | included | none | none | yes | none | Rule survived in REC line 143 and PLAN line 95, but the evasive-naturalist-error rationale was absent. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md lines 13, 267, and 315: aggregate-only operations and no analytics/ML path to simulation data. | PLAN.md lines 27, 260-267, and 283: telemetry has no per-account simulation/query path. | no | full | Recovered as relationship-data privacy enforced at the pipeline boundary. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md lines 49, 171, and 313: visitor sessions cannot affect host state; snapshots exclude interaction controls. | PLAN.md lines 122-124: visitor snapshot excludes greetings/interactions and checks revocation. | no | full | Recovered as observation, not co-presence. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md lines 57, 173, and 289: host notification is explicit opt-in, silent log by default. | PLAN.md lines 16 and 124: no push/email engagement notifications; host notification only explicit opt-in. | no | full | Recovered as silent transparency rather than an attention loop. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | No discovery/public/social surfaces survived, but the comparison-surface rationale was not reconstructed. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md lines 17, 253, and 309: naturalist narration, same public snapshot data, no state dumps. | PLAN.md lines 238-250: shared formatter, naturalist prose, useful narration, direct system errors. | no | full | Recovered as a designed prose surface rather than ARIA automation. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md lines 17, 259, and 307: cross-fade renderer preserves birds, audio, captions, mood/drift, notebook. | PLAN.md line 254: reduced-motion keeps all features and uses slow cross-fades, not a static illustration. | no | full | Recovered as feature parity with a distinct quiet aesthetic. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md lines 273 and 303: first-bird budgets protect living-scene feel; loading breaks the conceit. | PLAN.md lines 277 and 319: first bird visible under 500 ms; compact bootstrap and mobile tests. | no | full | Recovered as affective performance, not a raw metric only. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md lines 7, 55, and 319: no urgency/optimization; exclusions prevent a game with streaks/scores/ranks. | PLAN.md lines 16, 327, and 333: forbid gamification data/UI and acceptance guardrails. | no | full | Recovered as a loud product-boundary refusal. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md lines 7, 195, and 295: no visible distress/recovery task; no absence-tied negative drift. | PLAN.md lines 23, 157, and 315: no urgency around absence; no negative neglect drift. | no | full | Recovered as observational care without guilt or custodial punishment. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | RECONSTRUCTION.md line 37 says NOT RECOVERABLE FROM PLAN; plan had the no-catalog rule but not the meeting-animals rationale. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md lines 213 and 287: age-only eligibility, not clicks/presence/subscription/social, no progress framing. | PLAN.md lines 184 and 301: aviary-age cadence, not attention score/paid tier/social activity. | no | full | Recovered as growth over time, not reward economy. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md lines 39, 111, 137, and 323: rename preserves ID; same identities; identities persist across sessions. | PLAN.md lines 67, 89, and 331: identity never changes; recovery reinstates same records; birds retain identity. | no | full | Recovered as relationship continuity, not only database consistency. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md lines 115, 199, and 323: mood carries through sessions and does not reset on open. | PLAN.md lines 69, 163, and 331: durable mood projection carries through sessions. | no | full | Recovered as continuity of the aviary while away. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Read-only/immutable notebook survived in REC line 125 and PLAN line 74, but the anti-journal/curation rationale did not. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION.md lines 41, 89, and 139: export is a private account artifact, not analytics or exposed state. | PLAN.md lines 91 and 104: export current aviary privately via short-lived link. | no | full | Recovered as a private copy of the user-owned aviary state. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md lines 43, 137, and 315: recovery restores same identities; hard-delete purges linked rows; relationship data privacy. | PLAN.md lines 89 and 105: 30-day recovery; hard-delete all linked rows/objects; system-domain deletion routes. | no | partial | Hard-delete/privacy and whole-record purge survived; accidental-regret rationale was not explicit. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md lines 13, 267, and 275: aggregate-only telemetry and no path to simulation/event/notebook data. | PLAN.md lines 27, 260-267, and 283: aggregate RUM only; no per-account/bird/relationship dimensions. | no | full | Recovered as a technical observability boundary. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md lines 129, 169, and 313: explicit named invitations, no global graph, per-invite controls. | PLAN.md lines 119-124: one named email invitation, no account existence leak, immediate revocation. | no | full | Recovered as deliberate access to a private relationship. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md lines 131, 173, and 289: transparency audit in settings; silent log by default. | PLAN.md line 124: visit log from authorized access; host notification only explicit opt-in. | no | full | Recovered as on-demand transparency without notification pressure. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md lines 171 and 323: visitor snapshot returns rendered host state; visitors observe only. | PLAN.md line 122: visit snapshot returns the same rendered state surface as host snapshot. | no | full | Recovered as witnessing the real host aviary, not a staged social surface. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md lines 255 and 309: 30-60 second meaningful-change updates, dedupe, bounded queue avoid flooding/status dumps. | PLAN.md lines 242-248: idle updates every 30-60 seconds, priority only for user events, no frame/API chatter. | no | full | Recovered as slow naturalist presence rather than announcement-style monitoring. |


For multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | yes |
| F3 | yes | yes | yes |
| F4 | yes | no | yes |
| F7 | yes | no | yes |
| F9 | yes | yes | no |
| F10 | yes | no | no |
| F12 | yes | yes | yes |
| F13 | yes | yes | yes |
| F14 | yes | yes | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | yes |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | yes | yes |
| F30 | yes | yes | yes |
| F31 | yes | yes | yes |
| F35 | no | yes | yes |
| F40 | yes | yes | yes |


### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---|---|
| Possible gold whys | 49 | From gold_why_totals |
| Possible total weight | 152 | From intent_recovery.total_possible_weight |
| Reachable gold whys | 49 | S whys always included; all F why anchors captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 130 / 152 | Sum of weight x recovery-score over included whys |
| Whys with reconstruction evidence | 45 | Rationale evidence present in frozen reconstruction |
| Whys with PLAN grounding | 45 | Rationale evidence present in PLAN |
| rule_without_why cases | 4 | Mechanism survived without gold rationale |
| plan_only_not_reconstructed cases | 0 | No additional plan-only rationale cases counted |
| ungrounded_reconstruction cases | 0 | No confabulated rationale credited |


### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weighted | Recovery rate |
|---|---|---|---|
| Functional whys | 54.0 | 48.0 | 88.9% |
| Affective whys | 98.0 | 82.0 | 83.7% |
| Weight-2 whys | 44.0 | 36.0 | 81.8% |
| Weight-3 whys | 108.0 | 94.0 | 87.0% |
| System-level whys | 28.0 | 26.0 | 92.9% |
| Feature-level whys (reachable) | 124.0 | 104.0 | 83.9% |


---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered better (48.0/54.0, 88.9%) than affective whys (80.0/98.0, 81.6%). Server ownership, presence precision, privacy, and telemetry boundaries remained crisp; social/copy exceptions were more likely to compress into rules.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 92.0/108.0 (85.2%), slightly above weight-2 at 36.0/44.0 (81.8%). The high-weight mechanisms were repeated across architecture, risks, and done criteria, which helped reconstruction.
- **System-level vs feature-level.** System-level fidelity was higher (26.0/28.0, 92.9%) than feature-level fidelity (102.0/124.0, 82.3%). The planner carried philosophy well, but some feature-level exception rationales were compressed away.
- **Multi-layer recovery patterns.** Lost layers were usually secondary/downstream consequence layers: S2 lost the processed-vs-seen and cumulative-toast rationale; F4 lost the chorus phase-artifact detail; F7 lost the user-felt deletion rationale; F35 lost accidental-regret motivation.
- **Subdomain patterns.** Accounts/sync, simulation, accessibility, privacy, and performance were strongest. Social and product-voice exceptions were weaker: F19, F23, F29, and F33 became mechanism-only survivals.
- **Evidence-bound effects.** Four rows were denied by rule-without-why: F19, F23, F29, and F33. F29 was explicit because the frozen reconstruction says `NOT RECOVERABLE FROM PLAN`.

What this suggests: this candidate plan is excellent at implementation coverage and architectural intent, but some affective exception whys require more explicit rationale-carrying language to survive a fresh reconstruction.

---

## 4. Recommendations for v2 hardening

- Keep the v06 evidence-bound operator. In this run it separated a very high feature-capture score from feature-level rationale losses that would be easy to miss under a looser semantic read.
- Add more targeted affective exception whys like F29 and F33. Strong planners often name the rule while dropping the relationship-shaping reason.
- Preserve the multi-layer format for weight-3 whys. It exposed where the model kept primary mechanisms but lost downstream product consequences.
- For future rubrics, add one more worked example for system-level partials where many operational rules survive but the named affective contrast does not. S2 was the main subjective call here.
- Keep the full 120-feature planning denominator. It detected a small set of non-why feature omissions without distorting feature-fidelity reachability.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** This phase-2B scorer used only the allowed phase-two materials and run 001 artifacts. The frozen reconstruction was not modified.
- **Single-run-at-temperature limitation.** This is one run; there is no variance estimate inside this score.
- **Borderline capture calls.** Feature 51 was counted captured by inclusive convention because the plan covers visibility/suspension behavior plus server-side simulation, though it does not literally say "background-tab pause." The call did not affect F1-F40 reachability.
- **System-level cross-cutting.** The cross-cutting bar is most subjective. S2 was scored partial because the operational refusal of announcements survived but the processed-vs-seen rationale did not.
- **Confabulation cases.** No ungrounded reconstruction claims were credited.
- **Evidence-bound denials.** F19, F23, F29, and F33 had mechanism/rule evidence but no gold-rationale evidence.
- **Operational compromise.** TIMING.json provided phase1 and phase2a timing for run 001; phase2b timing was not present at scoring time, so the score JSON timing object includes only available phase fields.

End of report.
