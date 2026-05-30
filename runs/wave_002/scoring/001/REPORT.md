# REPORT - CARE run 001

> Phase 2B scoring report for Pocket Aviary. Companion artifacts: frozen `RECONSTRUCTION.md`, strict `run_001.json`, and interactive `REPORT.html`.
> Variant v06 evidence-bound clean + targeted gold headroom. PLAN evidence can establish capture and grounding; feature-level recovery requires frozen-reconstruction evidence as well.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **88.3%** |
| Intent fidelity | **61.6%** |
| Combined quality | **8795** |

**Diagnostic split:**

- System-level fidelity: **78.6%**
- Feature-level fidelity: **57.6%**

**(Planning, fidelity) coordinate:** `(88.3, 61.6)`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 1 |
| Run label |  |
| Timestamp | 2026-05-30T13:39:24Z |
| Candidate model | minimax-m2.7 |
| Candidate effort | unknown |
| Candidate harness | opencode |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **106 / 120** = **88.3%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 12 | 60.0% |
| aviary_layout.md | 18 | 15 | 83.3% |
| accounts_sync.md | 18 | 15 | 83.3% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **106** | **88.3%** |

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured in PLAN with enough implementation specificity. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured in PLAN with enough implementation specificity. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured narrowly through anti-counter/non-gamification language; welcome-toast specifics are missing. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured in PLAN with enough implementation specificity. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in PLAN with enough implementation specificity. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured in PLAN with enough implementation specificity. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | No glossary section, but core domain terms are defined throughout the plan. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured in PLAN with enough implementation specificity. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured in PLAN with enough implementation specificity. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured in PLAN with enough implementation specificity. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured in PLAN with enough implementation specificity. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Return-greeting is named as in scope, but timing and variation details are missing. |
| 34 | Greeting variation by absence length | interactions.md | no | No absence-length greeting behavior appears. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | No boldness-based greeting order appears. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | no | No multiple-bird greeting stagger is specified. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No textual return-welcome exclusion appears. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured in PLAN with enough implementation specificity. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured in PLAN with enough implementation specificity. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured in PLAN with enough implementation specificity. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | Offer effects exist, but reaction variation by current mood/curiosity is absent. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | no | No per-bird offer cooldown is specified. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Settle exists, but the evening-lighting affordance is not specified. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | The plan does not state close-tab equivalence or optional settle status. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured in PLAN with enough implementation specificity. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured in PLAN with enough implementation specificity. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured by generated entries and read-only API shape; no explicit edit/delete prohibition. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured in PLAN with enough implementation specificity. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured in PLAN with enough implementation specificity. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured in PLAN with enough implementation specificity. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured in PLAN with enough implementation specificity. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | No settle undo window is specified. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Evening color shift appears, but warmer hue/call-quieting detail is thin. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | No nightjar/most-birds-settled night state appears. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Top-bar scene treatment implies no in-aviary chrome; explicit wording is thin. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Top-bar surfaces appear, though notebook placement is less direct. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | no | No idle top-bar fade behavior is specified. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No between-adoption empty-aviary state appears. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured in PLAN with enough implementation specificity. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | No 15-minute magic-link expiry appears. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured through aggregate-only/no per-bird telemetry boundary; ML-specific wording is absent. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured in PLAN with enough implementation specificity. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link appears. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | No email-change flow appears. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured in PLAN with enough implementation specificity. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured in PLAN with enough implementation specificity. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured in PLAN with enough implementation specificity. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured in PLAN with enough implementation specificity. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Read-only visits/no profiles cover the intent; chat/comments are not named. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured in PLAN with enough implementation specificity. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured in PLAN with enough implementation specificity. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured in PLAN with enough implementation specificity. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured in PLAN with enough implementation specificity. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured in PLAN with enough implementation specificity. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured in PLAN with enough implementation specificity. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured in PLAN with enough implementation specificity. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured in PLAN with enough implementation specificity. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured in PLAN with enough implementation specificity. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured in PLAN with enough implementation specificity. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **78.6%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION System intent: "The aviary is a continuing place, not a session-bounded app"; "Procedural audio is part of the spell of liveness." | PLAN 7.3: "birds mid-action"; PLAN 8.1: "looped audio is the audible signature of dead software"; PLAN 10.2: "affective-perf bridge." | yes | yes | no | full | Continuing-place, procedural variation, quiet-field loading, and affective performance survive as one principle. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION System intent: "Presence is relational, not a counter" and refusal of "more attention earns more stuff." | PLAN 1.3: "presence is for a counter rather than the birds"; PLAN 11.2: "not visit count, not interaction score, not paid tier." | yes | yes | no | partial | Anti-counter/reward-loop survives; bird-as-welcome/no-toast affect is mostly absent. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION System intent: "Bird state should be felt through behavior, not read as numbers" and product voice is "naturalist and observational." | PLAN 3.4: "naturalist prose, lowercase, present-tense, specific"; PLAN 3.2: "felt through behavior, not read as numbers." | yes | yes | no | full | Specific naturalist observation and anti-stat framing are preserved. |
| S4 - restraint-over-richness | 2 | included | none | PLAN 1.1: two starters/cap seven; PLAN 7.1: single horizontal scene and no panning; PLAN 8.2: cap preserves call signatures. | no | yes | no | partial | PLAN is restrained, but B did not elevate restraint as a system-level why. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION System intent: product voice is "naturalist and observational, with one explicit exception" for sync conflicts. | PLAN 6.4: conflict surfaces use "matter-of-fact tone: direct, clear, no naturalist phrasing"; PLAN 9.1/9.2 use naturalist prose. | yes | yes | no | full | The voice split survives with the system-surface exception. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION System intent: "Presence is relational, not a counter"; regular presence and interactions shape birds slowly. | PLAN 5.2: presence-time is dominant; PLAN 12.5: three-condition conjunction prevents inflated presence-time; PLAN 5.2: settle ends presence cleanly. | yes | yes | no | partial | Presence-as-attention and signal honesty survive; settle-vs-close-tab equivalence does not. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION System intent: "Canonical server state protects coherence and history"; server is "only writer of personality state." | PLAN 2.1-2.2: server holds canonical state; PLAN 6.1: both devices pull same snapshot; PLAN 6.3: no last-write-wins. | yes | yes | no | full | Server tick, multi-device coherence, and LWW failure prevention are recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | none | PLAN 10.5: aggregate RUM excludes per-bird state and per-account interaction history; PLAN 4.7/10.5 keep export and telemetry bounded. | no | yes | no | partial | PLAN has a privacy boundary, but B did not make privacy a system-level principle. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION System intent: "Accessibility is first-class design, not a fallback" and surfaces "ship with v1." | PLAN 7.4: reduced-motion is "a designed surface"; PLAN 9.1: narration as running prose; PLAN 12.4: accessibility surfaces ship with v1. | yes | yes | no | full | Charm-preserving accessibility, designed reduced motion, and v1 shipping are recovered. |

Multi-layer system whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | no |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: server-side continuing simulation, first frame mid-action, quiet-field loading, procedural audio, idle motion, 500ms affective-perf bridge.
- S2: anti-counter review question, no gamification surfaces, age-based bird offers, silent visit log/default notifications off. Welcome-toast specificity is absent.
- S3: naturalist notebook/narration/captions, named birds, vector values hidden, no public ranking/discovery.
- S4: two starters and cap seven, single horizontal scene, no panning/zooming, calm palette, listen-in as rebalance not solo UI.
- S5: notebook/narration/caption naturalist voice, plus sync conflict/account-style matter-of-fact exception.
- S6: presence-time dominant drift input, three-signal presence honesty tests, no streak/counter surfaces, monotonic no-punishment drift.
- S7: server-side tick, server-only personality writes, canonical multi-device snapshots, no last-write-wins, additive event-log deltas.
- S8: aggregate-only RUM, no per-bird/per-account telemetry dimensions, JSON export, 30-day deletion flow. B did not elevate this systemically.
- S9: screen-reader prose, reduced-motion designed surface, captions, keyboard/focus/contrast, accessibility surfaces shipping with v1.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity: **57.6%**.

Reachable feature-level whys: **38 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | none | PLAN 12.5: "three-condition conjunction" and "silently inflating presence-time" corrupts drift. | yes | none | PLAN carries the rationale, but B compresses it to generic presence/presence-honesty. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION Simulation: "Drift as a low-pass filter" and calibration balances "too fast" vs "too slow." | PLAN 5.2: low-pass filter, one-week measurable/three-week visible, and too-fast/Tamagotchi vs too-slow/screensaver risk. | no | partial | B recovers slow filtering and failure endpoints but drops the instrument-vs-user calibration layer. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION Simulation: traits do not move down because "absence is fine, not penalized," implementing "no Tamagotchi." | PLAN 5.2: traits move up with positive presence; neglected bird becomes ambient, not distressed. | no | full | No-punishment drift is recovered as the engine-level anti-Tamagotchi rule. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION Audio: recorded loops break the spell; chorus needs "real-time per-call variation." | PLAN 8.1: looped audio is "dead software"; PLAN 8.2: independent streams; PLAN 8.4: no recorded fallback. | no | full | Procedural synthesis, chorus variation, and cascade into WebAudio/fallback survive. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION Rendering: mood-shaped movement lets the user read mood "without being told." | PLAN 7.2: wary scans, content preens, curious tilts, and user reads mood without being told. | no | full | The visible mood-through-motion rationale is recovered. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION Scope: cap exists because beyond seven the ear cannot separate signatures and relationship collapses. | PLAN 8.2: seven is where per-bird signatures remain recognizable; beyond that chorus blurs. | no | full | The empirical recognizability ceiling survives. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION System intent: canonical server state protects coherence/history; clients never own personality state. | PLAN 2.2/6.3: server owns vectors, tick is only writer, no client mutation. | no | partial | Canonical persistence and server-only writes survive; the "deleting the bird" relationship layer is absent. |
| F8 | personality-vector-never-numerical | 2 | yes | included | RECONSTRUCTION Data: "Vector values are felt through behavior, not read as numbers." | PLAN 3.2: no stats/debug surface; values are felt through behavior, not read as numbers. | no | full | The anti-stat-management rationale is recovered. |
| F9 | return-greeting | 4 | yes | included | none | none | yes | none | The feature is named, but B says NOT RECOVERABLE FROM PLAN and the PLAN lacks the detailed greeting why. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | The return-toast exclusion is not captured in the PLAN. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Close-tab equivalence/optional settle is not captured in the PLAN. |
| F12 | field-notebook-auto-entries | 4 | yes | included | RECONSTRUCTION Scope: notebook is naturalist observation, not generic event log; per-session entries dilute what matters. | PLAN 3.4/12.6: naturalist lowercase specific prose; sparse entries; not a generic event logger. | no | full | Naturalist voice, anti-event-log rationale, and sparsity survive. |
| F13 | presence-accounting | 4 | yes | included | none | PLAN 12.5: all three signals must be tested; incorrect implementation silently inflates presence-time and corrupts drift. | yes | none | The rule is present in PLAN, but B does not recover the implementation precision rationale. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION Non-goals: hard rule prevents teaching that "presence is for a counter rather than the birds." | PLAN 1.2/1.3: no streaks or days-visited surfaces; reject anything teaching presence is for a counter. | no | partial | The primary counter-vs-birds reason survives; intention-rotation and disguised surfaces are missing. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION Rendering: first frame mid-action because simulation "has been"; spinner says "machine." | PLAN 7.3: first frame mid-action, no wake-up; client pulls snapshot; quiet field not spinner. | no | full | Continuing-place load, snapshot implementation, and no-spinner consequence all survive. |
| F16 | synthetic-account-id | 4 | yes | included | none | none | yes | none | B marks it NOT RECOVERABLE; PLAN has the UUID rule but not the PII/compliance why. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION Simulation: tick is only writer; events become drift, moods, positions, notebook, weather, and visits. | PLAN 5.1/6.1/6.3: server tick runs, server only writer, devices read same canonical state. | no | full | The architectural why and sync coherence are recovered. |
| F18 | no-last-write-wins-personality | 4 | yes | included | RECONSTRUCTION Sync: clients send "user was present" never "set boldness"; prevents LWW corruption of drift history. | PLAN 6.3/12.2: additive server-authored deltas, event log in order, no direct client personality writes. | no | full | Additive deltas, lost-drift risk, and implementation invariant survive. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | RECONSTRUCTION Sync: rare errors use "direct, clear, matter-of-fact language" and no naturalist phrasing. | PLAN 6.4: sync conflicts drop into matter-of-fact tone: direct, clear, no naturalist phrasing. | no | full | The system-clarity exception is recovered, though without the exact "evasive" word. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION Observability: aggregate metrics exclude per-bird state and per-account interaction history so privacy boundary is honored. | PLAN 10.5: RUM is aggregate only and contains no per-bird state or per-account interaction history. | no | partial | Telemetry boundary survives; the private-relationship/data-product rationale is not explicit. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION Scope: friend sees read-only ambient view with no interaction events and no co-presence mechanism. | PLAN 4.6/6.4: visitor path returns read-only snapshot; no interaction events from visitor sessions. | no | full | Read-only observation and no accidental drift are recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | none | none | yes | none | The silent log rule appears, but the attention-driver/social-loop why is absent. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | none | none | yes | none | The no-public-social rule appears, but the comparison/"their birds" rationale is absent. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION Accessibility: same state and voice as notebook, naturalist experience, not a state-change list. | PLAN 9.1: running naturalist prose, not lists; same voice as field notebook; events remain observations. | no | full | Naturalist prose, equal affective experience, and anti-ARIA-list implementation survive. |
| F25 | reduced-motion-mode | 4 | yes | included | RECONSTRUCTION Accessibility: vestibular user gets "calmer and slower, not broken"; calls, drift, mood, notebook remain. | PLAN 7.4: designed surface, cross-fades, calls still play, birds drift, notebook notices. | no | full | Reduced motion as a designed equivalent surface is recovered. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECONSTRUCTION Performance: below 500ms the aviary feels "already running"; above it user notices load. | PLAN 10.2: 500ms threshold is the affective-perf bridge; bundle/snapshot/render path required. | no | full | The performance-as-liveness rationale survives. |
| F27 | no-gamification | 4 | yes | included | RECONSTRUCTION Non-goals: hard rule prevents teaching that "presence is for a counter rather than the birds." | PLAN 1.2/1.3: no achievements/streaks/scores; non-goals are load-bearing constraints. | no | partial | The core anti-counter reason survives; temptation and erosion layers are compressed away. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | RECONSTRUCTION Non-goals: rejects death/hunger/decay/distress because relationship is "observational, not custodial." | PLAN 1.2/5.2: no distress; neglected bird becomes ambient; absence is not penalized. | no | full | The no-obligation/no-punishment rationale is recovered. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION Scope: first encounter should be "meeting an animal, not configuring an avatar." | PLAN 11.1: user does not pick from catalog; first encounter is meeting an animal. | no | full | The arrival-not-catalog rationale is recovered. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECONSTRUCTION Rollout: time-based availability refuses the gamification trap and makes birds "a function of time, not effort." | PLAN 11.2: aviary age, not visit count/score/tier; relationship deepens over time. | no | partial | Age and anti-reward-loop layers survive; economy/erosion consequence is thin. |
| F31 | stable-bird-identity | 4 | yes | included | none | none | yes | none | B explicitly marks stable internal ID NOT RECOVERABLE FROM PLAN. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION Simulation: user should never see mood "snapping" to a default on tab open. | PLAN 5.3: mood persists across sessions; tick may advance it; no snapping to default. | no | full | The continuity/no-reset rationale is recovered. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Read-only is at most implicit; B does not recover the observer-record-vs-journal why. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | B marks account export NOT RECOVERABLE FROM PLAN; PLAN gives the rule but not the quiet relationship-copy why. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | none | none | yes | none | B marks account deletion NOT RECOVERABLE FROM PLAN; PLAN gives timing but not regret/privacy/residue layers. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION Observability: metrics exclude per-bird state and per-account history so privacy boundary is honored at metric definition level. | PLAN 10.5: aggregate request/timing/error metrics; no per-bird state or per-account interaction history. | no | full | The technical telemetry boundary is recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | none | none | yes | none | Per-invite sharing is present, but the private-relationship/not-publishing why is absent. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | none | yes | none | The silent on-demand log rule appears, but transparency-vs-attention-loop rationale is absent. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | The snapshot rule appears, but B does not recover actual-witnessing vs show-off-mode rationale. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECONSTRUCTION Accessibility: slow cadence; high-frequency narration overwhelms queue; events are still observations. | PLAN 9.1: 30-60s idle cadence, faster user events, high-frequency overwhelms queue, slow visual rhythm. | no | full | Slow rhythm, queue protection, and observational priority events are recovered. |

Multi-layer feature whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | no | no | no |
| F2 | yes | no | yes |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | no | yes |
| F9 | no | no | no |
| F10 | n/a | n/a | n/a |
| F12 | yes | yes | yes |
| F13 | no | no | no |
| F14 | yes | no | no |
| F15 | yes | yes | yes |
| F16 | no | no | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | no | no |
| F30 | yes | yes | no |
| F31 | no | no | no |
| F35 | no | no | no |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From benchmark constants |
| Possible total weight | 152 | From intent recovery constants |
| Reachable gold whys | 47 | S whys always included; F whys only when feature captured |
| Excluded unreachable feature whys | 2 | F10 and F11 |
| Recovered / reachable weight | 90.0 / 146.0 | Weighted sum over included whys |
| Whys with reconstruction evidence | 32 | Exact B-side why evidence present |
| Whys with PLAN grounding | 36 | Exact PLAN rationale grounding present |
| `rule_without_why` cases | 13 | Operational rule survived without the gold rationale |
| `plan_only_not_reconstructed` cases | 4 | Mainly S4, S8, F1, F13 |
| `ungrounded_reconstruction` cases | 0 | No clear ungrounded rationale assertions scored |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 27.0 | 50.0% |
| Affective whys | 92.0 | 63.0 | 68.5% |
| Weight-2 whys | 42.0 | 26.0 | 61.9% |
| Weight-3 whys | 104.0 | 64.0 | 61.5% |
| System-level whys | 28.0 | 22.0 | 78.6% |
| Feature-level whys (reachable) | 118.0 | 68.0 | 57.6% |

## 3. Diagnostic patterns

- **Affective vs functional.** Affective whys recovered better overall (63.0/92.0 = 68.5%) than functional whys (27.0/54.0 = 50.0%). The functional losses cluster around privacy/identity/account details: F16, F34, and F35 are rules without their deeper rationale, while F1/F13 lose the precision of the presence signal.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 64.0/104.0 (61.5%), close to weight-2 at 26.0/42.0 (61.9%). High-weight rows often preserved the primary cause but lost secondary/downstream layers, e.g. F2, F14, F27, and F30.
- **System-level vs feature-level.** The planner preserved philosophy better than feature-level rationale. System fidelity is 78.6%; feature fidelity is 57.6%. The reconstruction is good at summarizing global themes but often says NOT RECOVERABLE for specific account/identity features.
- **Multi-layer recovery.** Primary causes were most likely to survive. Secondary calibration layers and downstream consequence layers were the common drop points.
- **Subdomain patterns.** Audio and accessibility are strong; accounts/sync architecture is mixed. Server tick and no-LWW are strong, while synthetic ID, export, deletion, and stable identity rationale are missing.
- **Evidence-bound effects.** Several v1-style semantic matches were denied or reduced because B carried the rule but not the why: F16, F22, F23, F31, F33-F35, and F37-F39. F1 and F13 are plan-only cases.

## 4. Recommendations for v2 hardening

- Keep the evidence-bound operator. It cleanly distinguishes rule capture from why recovery; this run would look much stronger without that distinction.
- Add targeted checks around account/privacy rationale. F34-F39 expose places where a plan names operational surfaces but omits relationship/privacy reasons.
- Preserve multi-layer whys for high-risk calibration features. F2, F14, F27, and F30 show that a planner can recover the main rule while losing why the line must hold.
- Consider making the system-level cross-cutting bar report an explicit count. S4 and S8 were preserved in PLAN but not identified by B.
- Keep concrete exclusion whys like no welcome toast and settle opt-in. They caught genuine omissions that broad non-gamification language did not cover.

## 5. Methodology caveats

- **Fresh-context fidelity.** The scoring prompt states this was fresh context, and the frozen reconstruction stayed read-only. I saw no operational evidence of same-context contamination.
- **Single-run limitation.** This is one run in one wave; no variance estimate is available from this artifact alone.
- **Borderline capture calls.** I leaned inclusive on #3, #7, #33, #43, #47, #57, #63, #64, #85, and #93. These raise planning quality but do not grant why recovery without reconstruction evidence.
- **System-level cross-cutting.** S4 and S8 are subjective partials: the PLAN preserves them across multiple places, but B does not identify them in system-level intent.
- **Confabulation cases.** I did not mark any clear ungrounded-reconstruction recovery. Where B was unsure, it often wrote NOT RECOVERABLE FROM PLAN, which correctly scored as none.
- **Evidence-bound denials.** F1, F13, F16, F22, F23, F31, F33-F35, and F37-F39 are the main mechanism-without-why failures.
- **Operational compromise.** TIMING.json exposed phase1 and phase2a data for run 001 only; no phase2b timing was present in that ledger at score-write time.

---
End of report.
