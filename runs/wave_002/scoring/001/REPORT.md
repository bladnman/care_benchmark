# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. This report scores the frozen reconstruction against the gold why list and keeps denominator reachability separate from why recovery.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **100.0%** |
| Intent fidelity | **75.0%** |
| Combined quality | **9975** |

**Diagnostic split:**

- System-level fidelity: **92.9%**
- Feature-level fidelity: **71.0%**

**(Planning, fidelity) coordinate:** `(100.0, 75.0)` - plot on a 2D scatter with both axes 0-100; upper-right is best.

No low-confidence banner: planning is above 30%.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-24T08:00:37Z |
| Candidate model | gpt-6-sol |
| Candidate effort | high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **120 / 120** = **100.0%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 20 | 100.0% |
| aviary_layout.md | 18 | 18 | 100.0% |
| accounts_sync.md | 18 | 18 | 100.0% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **120** | **100.0%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured in PLAN section 1 lines 5-7. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured in PLAN section 1 lines 5-7. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured in PLAN section 1 lines 5-7. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured in PLAN section 1 lines 5-7. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in PLAN section 1 lines 5-7. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured in PLAN section 1 lines 5-7. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | borderline: Borderline inclusive: no literal glossary, but the plan defines domain concepts in the contract and engine sections. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured through the product contract and presence/settle model in PLAN lines 9 and 66-70. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured through the product contract and presence/settle model in PLAN lines 9 and 66-70. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured through the product contract and presence/settle model in PLAN lines 9 and 66-70. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured across PLAN sections 3, 5, and 6, especially lines 33, 68-82. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured across PLAN interaction, API, tick, audio, and scene sections, lines 14, 50, 66, 72-74, 82, and 90. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured through the scene/chrome/rendering plan, lines 5, 25, and 88-94. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | borderline: Borderline inclusive: captured as privacy settings that describe categories plainly rather than a literal policy link. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured in account, API, sync, telemetry, and lifecycle planning, lines 31-37, 47-60, and 98-102. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured in invitation/visit scopes and social exclusions, lines 5, 36, 56-60, and 100. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured in invitation/visit scopes and social exclusions, lines 5, 36, 56-60, and 100. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured in invitation/visit scopes and social exclusions, lines 5, 36, 56-60, and 100. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured in invitation/visit scopes and social exclusions, lines 5, 36, 56-60, and 100. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured in invitation/visit scopes and social exclusions, lines 5, 36, 56-60, and 100. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured in invitation/visit scopes and social exclusions, lines 5, 36, 56-60, and 100. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured in invitation/visit scopes and social exclusions, lines 5, 36, 56-60, and 100. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured in invitation/visit scopes and social exclusions, lines 5, 36, 56-60, and 100. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured in invitation/visit scopes and social exclusions, lines 5, 36, 56-60, and 100. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured in invitation/visit scopes and social exclusions, lines 5, 36, 56-60, and 100. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | borderline: Borderline inclusive: captions and accessibility settings are specified; the exact toggle label is not singled out. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured in accessibility/performance gates and tests, lines 17, 92-94, 106-108, and 115. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | borderline: Borderline inclusive: supported browser/OS testing and fallback are specified, not the literal last-two-majors list. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured in explicit exclusions and sync/offline constraints, lines 7 and 98. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured in explicit exclusions and sync/offline constraints, lines 7 and 98. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured in explicit exclusions and sync/offline constraints, lines 7 and 98. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | borderline: Borderline inclusive: public feeds/discovery/comments are explicitly excluded; profiles/follows are covered semantically as social-network surfaces. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **92.9%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md lines 8, 11: "Aliveness comes from deterministic procedural variation"; "already-running world" | PLAN.md lines 13-14, 80, 90: first frame already-running, varied greeting/calls, non-fixed idle motion | yes | yes | no | full | Recovered as procedural variation plus an already-running world across greeting, audio, loading, and motion. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md lines 3, 12: "default is silence" and "avoids pressure loops" | PLAN.md lines 7, 58, 60, 74: no toast/banner, no badge/push, no default notifications, no user-behavior notebook prose | yes | yes | yes | partial | The no-announcement rule and cumulative pressure-loop refusal survived, but the processed-vs-seen rationale was not reconstructed. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md lines 4, 31: "naturalist observation, not a game dashboard"; entries use "specific detail" | PLAN.md lines 7, 74, 84: naturalist prose, specific notebook observations, call-derived captions | yes | yes | no | full | Specific naturalist observation, named/situated moments, and anti-dashboard framing are preserved. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md lines 3, 23, 54: sparse bar, one-screen scene, scene-only birds/place | PLAN.md lines 5, 25, 88, 116: one screen, seven-bird cap, no scene chrome, staged bird ramp | yes | yes | no | full | Restraint appears as capped birds, one screen, sparse top bar, no scene chrome, and controlled rollout. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md lines 13, 38: product prose is naturalist; failure/account surfaces are plain and direct | PLAN.md lines 7, 43, 100: naturalist product prose, typed errors with plain system copy, matter-of-fact revoked visits | yes | yes | no | full | The naturalist/product vs direct/system voice split is explicitly preserved across product and error/account surfaces. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md lines 93-96: visible/focused/recent activity, close/settle no penalty, no double-count, input evidence | PLAN.md lines 66, 68, 90: conjunction-based presence, dominant drift input, settle not a drift reward | yes | yes | no | full | Precise attention evidence, anti-inflation checks, and close/settle equivalence are all recovered. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md lines 6, 90-92, 136: canonical server-authored simulation, durable tick, no last-write-wins | PLAN.md lines 21, 64, 98: worker alone advances state, tick runs without clients, no LWW personality path | yes | yes | no | full | Server-side canonical state, multi-device coherence, and LWW avoidance all survive. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md lines 9, 141-143, 167: aggregate-only telemetry, no per-bird state, no analytics join | PLAN.md lines 31, 102, 118: encrypted identifiers, aggregate-only telemetry, no population analysis of private interactions | yes | yes | no | full | The privacy boundary is technical and cross-cutting, not just policy language. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md lines 10, 36, 128-130, 156: supported sensory paths, same aviary, no launch waiver | PLAN.md lines 17, 92-94, 108, 115, 126: narration/captions/cross-fades, queue pacing, keyboard/SR tests, no waiver | yes | yes | no | full | Accessible surfaces are recovered as designed product surfaces that ship with the core experience. |

Multi-layer system-level recovery:

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | yes |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: PLAN lines 13-14 (already-running greeting), 80 (procedural calls), 88-90 (ongoing scene/motion), 125 (load-state risk).
- S2: PLAN lines 7 (no toast/default silence), 58-60 (no visit notification by default), 74 (no user-behavior notebook prose), 100 (no badge).
- S3: PLAN lines 7 (naturalist prose), 74 (specific notebook observations), 84 (call-derived captions), 108/155 (felt aliveness review).
- S4: PLAN lines 5 (one screen and seven cap), 25 (scene/chrome split), 88 (no scroll/pan/zoom), 116 (staged bird-count ramp).
- S5: PLAN lines 7 (naturalist vs matter-of-fact), 43 (plain error copy), 60 (visitor notification mail), 100 (revoked visit state).
- S6: PLAN lines 66 (presence definition and close/settle equivalence), 68 (dominant drift input), 90 (settle not reward), 108 (presence tests).
- S7: PLAN lines 21 (worker-only simulation), 64 (server tick without clients), 98 (no local canonical drift), 108 (idempotent replay tests).
- S8: PLAN lines 31 (synthetic IDs/encrypted email), 34 (retention), 102 (telemetry isolation), 118 (no private per-bird population analysis).
- S9: PLAN lines 17 (supported sensory paths), 92-94 (narration/reduced motion/keyboard), 108 (accessibility tests), 126 (stripped-fallback risk).

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **71.0%**.

Reachable feature-level whys: **40 / 40** (no feature-level anchors were excluded).

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md lines 93, 95: visible/focused/recent activity; duplicate devices do not double-count | PLAN.md line 66: visibility, focus, activity-window conjunction with duplicate-device union | yes | partial | Recovered the precise rule and anti-inflation shape, but not the silent population-wide drift-corruption consequence. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md line 97: low-pass drift calibrated to one intensive session, one week, and three weeks | PLAN.md lines 15, 68, 122: week/three-week calibration and low-pass filter | yes | partial | Recovered slow low-pass calibration, but not the Tamagotchi-vs-screensaver failure endpoints. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md lines 7, 100: non-punitive growth and no distress, punishment, or lowered traits | PLAN.md lines 15, 68, 70: absence never decreases personality; no neglect subtraction; no distress | no | full | All three layers survive: upward-only drift, no punishment, and quieter-not-hurt return behavior. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md lines 26, 111: rejects recorded loops; local WebAudio and no recorded-loop fallback | PLAN.md lines 80, 82, 124: synthesized calls, no audio downloads, no recorded-loop fallback, canned-audio risk | yes | partial | Recovered no loops and WebAudio cascade, but not the chorus phase-cancel rationale. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md lines 27, 126: mood maps into behavior and makes mood visible through behavior | PLAN.md lines 90, 92: mood-keyed preen/scan/tilt and no raw state-list primary experience | no | full | Mood is recovered as something read through motion rather than labels. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md line 113: seven-bird listening tests preserve recognizability | PLAN.md lines 80, 116: seven-bird mixes and ramp only after seven-voice chorus passes | no | full | Seven-bird cap is grounded in recognizability, not an arbitrary limit. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION.md lines 6, 61, 69: canonical simulation, stable traits, halt instead of reseeding | PLAN.md lines 23, 33, 39, 98: canonical state, stable IDs/vectors, halt on corruption, no LWW | no | full | Vector persistence, identity stakes, and multi-device/no-LWW implications all survive. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md lines 5, 50: personality expressed through behavior, not raw vector statistics | PLAN.md lines 23, 102: no raw personality-vector numbers; private export is not product UI stats | no | full | The plan/reconstruction preserve the bird-as-behavior, not bird-as-number, rationale. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md lines 37, 102: varied bird greeting using absence, mood, boldness, and greeting history | PLAN.md lines 14, 72: one varied greeter weighted by absence, boldness, warmth, mood, and history | yes | partial | Recovered one-bird procedural greeting inputs, but not the explicit warning against generic arrival animation. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md line 37: no text welcome; procedurally varied bird greeting keeps surface quiet | PLAN.md lines 7, 14, 60: no arrival toast/banner, no text welcome, no default notification | yes | partial | Recovered the core no-toast application; deeper reframing and all variant surfaces are thinner. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md lines 30, 94: settle ends presence with no penalty and changes quietness, not personality | PLAN.md lines 66, 90: closing/settling end presence with no penalty; settle is not a drift reward | no | full | Optional settle is grounded in non-penalty and non-obligation. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md lines 31, 103: noteworthy aviary events, specific detail, rarity gates, not generic receipts | PLAN.md line 74: noteworthy aviary events, rare spacing, specific detail, no session starts/streaks/deltas | yes | partial | Recovered naturalist specificity and rarity/read-only boundary; the voice-as-concentrated-surface layer is weaker. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md lines 93-96: validated attention, bounded intervals, no double counting, input evidence | PLAN.md line 66: all three conditions, bounded segments, duplicate-device union, no engagement metric | yes | partial | Recovered precise presence accounting and missed-cases logic, but not the full silent-drift-failure consequence. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md lines 4, 31, 96: no streaks/counters, no user-behavior notebook prose, no metric-like care loops | PLAN.md lines 7, 66, 74: no streaks/counters, presence not an engagement metric, no streak prose | no | full | The counter refusal, intention-rotation risk, and adjacent notebook/metric disguises are recovered. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md lines 11, 43, 52: already-running world, 500 ms gate, quiet sky fallback | PLAN.md lines 13, 25, 88, 125: already-running frame, quiet non-spinner field, no entry animation, load-state risk | no | full | Already-running first frame, server snapshot implication, and quiet-field fallback all survive. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md lines 59, 127: email only encrypted, no logging, synthetic IDs everywhere | PLAN.md lines 31, 127: encrypted email, nonreversible lookup, account UUIDs in all references | yes | partial | Recovered PII/logging boundary; retrofit-impossibility rationale is not reconstructed. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md lines 6, 90-92: server-authored canonical tick even without browser, stable order, atomic commit | PLAN.md lines 21, 64, 98: worker-only simulation, tick with no clients, no local canonical drift | no | full | Canonical tick, sync coherence, and client-tick collapse risk are recovered. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md lines 48, 136, 152: API rejects client-authored traits; no LWW; two devices apply exactly once | PLAN.md lines 16, 21, 98, 108: devices converge, API never accepts trait values, no LWW, concurrency tests | no | full | Server-authored deltas, overwrite avoidance, and implementation rule all survive. |
| F19 | sync-conflict-tone | 2 | yes | included | none | none | yes | none | Rule appears in RECONSTRUCTION.md line 13 and PLAN.md lines 7/43, but the evasive-naturalist-error rationale is absent. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md lines 141-143, 167: no per-bird analytics, no warehouse join, no population analysis | PLAN.md lines 102, 118: aggregate operations only and no population-level analysis of private interactions | no | full | Own-simulation-only data use, private-relationship concern, and pipeline enforcement are recovered. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md lines 32, 83: read-only visits, no event-write route, zero host events/drift | PLAN.md lines 16, 57, 60: visitor activity produces zero drift; visitor token cannot use host endpoints | no | full | Visit as observation, not co-presence or host-drifting interaction, survives. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md lines 84, 86, 12: no badge/push; opt-in mail only; avoids pressure loops | PLAN.md lines 58, 60, 100: no badge/push, no default notification, log on demand | no | full | Default silence and refusal of attention-driving visit loops are recovered. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | The social/public surfaces are excluded in RECONSTRUCTION.md line 12 and PLAN.md line 7, but the comparison-to-other-birds rationale is not recovered. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md lines 10, 129, 156: first-class accessibility, specific prose, not noisy ARIA, no waiver | PLAN.md lines 17, 92, 94, 108: narration/captions/cross-fades, prose cadence, not raw/noisy ARIA | no | full | Running prose, equal affective experience, and not-ARIA-automation implementation all survive. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md lines 10, 36, 128: same aviary through cross-fades, not stripped fallback | PLAN.md lines 17, 92, 126: same snapshot/events, slow cross-fades, stripped-fallback risk gate | no | full | Reduced motion is recovered as a designed alternate rendering, not animations-off. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md lines 43, 147: first-bird timing as acceptance/performance gate | PLAN.md lines 13, 25, 106, 125: 500 ms first bird, bootstrap snapshot, first-frame risk | no | full | The performance metric is tied to already-running felt aliveness. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md lines 4, 12, 96: not a game dashboard, avoids pressure loops, presence not an engagement metric | PLAN.md lines 7, 66, 122: no achievements/streaks/scores; presence evidence not engagement; drift gameability risk | yes | partial | Recovered counter/pressure-loop refusal; future erosion cascade is not reconstructed. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md lines 7, 100: non-punitive growth; no distress, punishment, or lowered traits | PLAN.md lines 7, 15, 70: no care meters/distress/death, no decline after absence, no punishment | no | full | No-punishment observational relationship is recovered. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | System-chosen starters are present in RECONSTRUCTION.md line 21 and PLAN.md line 78, but the arrived-not-catalog relational why is absent. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md lines 22, 80, 165: aviary age, never usage/score, users gain birds slowly | PLAN.md lines 53, 78, 116: server checks age schedule, never visits/score/usage, age schedule stays slow | yes | partial | Recovered age-not-score refusal; bird-economy erosion consequence is not explicit. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md lines 51, 61, 166: stable identity, no regenerate IDs, no rewinding bird IDs | PLAN.md lines 23, 33, 39, 116: stable IDs/signatures, migrations preserve IDs, rollback without rewinding | yes | partial | Recovered stable identity and migration distinction; retroactive evaporation of presence-time is not fully reconstructed. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md lines 11, 101: already-running world and mood continuity with no snap to neutral | PLAN.md lines 70, 88: mood persists across sessions and no entry animation for returning accounts | no | full | Mood persistence is tied to the aviary having continued while away. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | The immutable notebook rule appears in RECONSTRUCTION.md lines 31/65 and PLAN.md lines 35/74, but the anti-curation/journal rationale is absent. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | Export is present in RECONSTRUCTION.md lines 34/143 and PLAN.md line 102, but the relationship-is-theirs quiet-QoL why is absent. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md lines 35, 168: 30-day recovery, hard deletion, privacy-pipeline release gate | PLAN.md lines 37, 55, 118: related rows deleted after 30 days; deletion/recovery and privacy gates | yes | partial | Recovered privacy/hard-delete and all-record deletion; accidental-regret layer is not reconstructed. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md lines 141-142: aggregate-only telemetry and no join to bird history | PLAN.md line 102: no account/bird/trait/action/per-bird fields and no analytics read path | no | full | The technical boundary preventing observability from becoming relationship data is recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md lines 32, 66, 138: explicit host action, invited email identity, separate scopes | PLAN.md lines 36, 56, 60: invitation by encrypted email, explicit host action, scoped visitor token | no | full | Sharing remains deliberate, named, and non-discoverable. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md lines 84, 139: host-only log with no badge/push and revocation visibility | PLAN.md lines 58, 100: visit log on demand with no badge; revocation checked each pull | no | full | Visit log is transparency without becoming an attention surface. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | The exact-snapshot rule appears in RECONSTRUCTION.md line 83 and PLAN.md line 57, but the no-show-off/real-birds rationale is absent. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md lines 117, 129: no flooding, 30-60 second cadence, faster only for user-initiated events | PLAN.md lines 92, 94, 108: slow narration, priority queue, no noisy ARIA stream, queue-pacing tests | no | full | Slow rhythm, screen-reader queue protection, and sparse observational pacing are recovered. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | no | yes |
| F7 | yes | yes | yes |
| F9 | yes | yes | no |
| F10 | yes | no | no |
| F12 | yes | no | yes |
| F13 | yes | yes | no |
| F14 | yes | yes | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | yes | no |
| F30 | yes | yes | no |
| F31 | yes | yes | no |
| F35 | no | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON `gold_why_totals` |
| Possible total weight | 152 | From score JSON `intent_recovery.total_possible_weight` |
| Reachable gold whys | 49 | S whys always included; all F whys reachable because all anchors were captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 114.0 / 152.0 | Sum of `weight x recovery-score` over included whys |
| Whys with reconstruction evidence | 43 | Rationale evidence for at least one full/partial layer |
| Whys with PLAN grounding | 43 | PLAN rationale grounding for at least one full/partial layer |
| `rule_without_why` cases | 19 | Mechanism survived while at least one rationale layer did not |
| `plan_only_not_reconstructed` cases | 13 | PLAN carried more rationale than the frozen reconstruction recovered |
| `ungrounded_reconstruction` cases | 0 | No scored recovery depended on ungrounded reconstruction |

### 2.5. Failure groupings

| Grouping | Total reachable | Recovered (full + partial-weighted) | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 42.0 | 77.8% |
| Affective whys | 98.0 | 72.0 | 73.5% |
| Weight 2 whys | 44.0 | 32.0 | 72.7% |
| Weight 3 whys | 108.0 | 82.0 | 75.9% |
| System-level whys | 28.0 | 26.0 | 92.9% |
| Feature-level whys (reachable) | 124.0 | 88.0 | 71.0% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered 40.0/54.0 (74.1%), while affective whys recovered 74.0/98.0 (75.5%). The bigger distinction was not type but whether the why was an exception to a tempting product pattern.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 82.0/108.0 (75.9%); weight-2 whys recovered 32.0/44.0 (72.7%). High-weight rationales survived only slightly better, mainly because system-level and architecture-heavy items were repeated throughout the plan.
- **System-level vs feature-level.** System-level fidelity was very high at 26.0/28.0 (92.9%). Feature-level fidelity was lower at 88.0/124.0 (71.0%), showing that the plan carried broad philosophy better than it carried every local why.
- **Multi-layer recovery patterns.** Downstream-consequence layers were the common miss: F1/F2/F9/F30/F31 lost the consequence even when they recovered the primary implementation rule.
- **Subdomain patterns.** Server-side simulation, privacy telemetry, and accessibility/performance held up strongly. Relational product whys around onboarding, notebook editability, export, and visitor presentation leaked most.
- **Evidence-bound effects.** Several rows looked implemented but not recovered as whys: F19, F23, F29, F33, F34, and F39 all had clear rules but no gold rationale in the frozen reconstruction.

The failure shape suggests a strong implementation plan that compresses local relational rationales when those rationales are not repeated as explicit design principles.

## 4. Recommendations for v2 hardening

- Add more targeted exception whys around features that are easy for strong planners to implement correctly while losing the relational reason: adoption/catalog refusal, export/deletion, and visitor presentation were useful discriminators here.
- Keep the system-level cross-cutting bar, but ask scorers to mark which layer was recovered for multi-layer system whys. S2 shows how a plan can preserve the no-announcement rule while losing the processed-vs-seen affective claim.
- Preserve the evidence-bound ledger. It prevented mechanism-only rows from inflating fidelity and made the system-vs-feature split diagnostic rather than ornamental.
- Consider recording layer-loss counts as bounded aggregate fields in a future schema. In this run, downstream-consequence loss was the dominant pattern.

## 5. Methodology caveats

- **Fresh-context fidelity.** The reconstruction reads plan-derived: it uses the plan sections and plan vocabulary rather than gold IDs or rubric terms. Validity audit verdict: PASS.
- **Single-run-at-temperature limitation.** This is one run with no variance estimate.
- **Borderline capture calls.** Five non-gold feature captures were inclusive: #7, #87, #104, #116, and #120. None affected feature-level why reachability because all 40 F anchors were directly captured.
- **System-level cross-cutting.** The 3-feature bar was easy to satisfy for most principles; only S2 lost a layer because the deeper processed-vs-seen rationale was not reconstructed.
- **Confabulation cases.** No scored recovery was based on ungrounded reconstruction. Some non-scored rule explanations, such as the starter-bird contrast rationale, were grounded in the plan but did not match the gold why.
- **Evidence-bound denials.** F19, F23, F29, F33, F34, and F39 were denied recovery because the frozen reconstruction preserved the mechanism without the gold rationale.
- **Rule-without-why cases.** Nineteen included rows were marked rule_without_why, mostly partial multi-layer rows or single-layer exceptions whose mechanisms survived without the relational reason.
- **Operational compromise.** TIMING.json had phase1 and phase2a timing for run 001 but no phase2b timing at scoring time, so the score JSON includes only available bounded timing fields.

---

End of report.
