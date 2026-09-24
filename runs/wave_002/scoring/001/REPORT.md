# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. This report scores the frozen reconstruction for run 001. The frozen reconstruction was not modified.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **100.0%** |
| Intent fidelity | **69.1%** |
| Combined quality | **9969** |

**Diagnostic split:**

- System-level fidelity: **75.0%**
- Feature-level fidelity: **67.7%**

**(Planning, fidelity) coordinate:** `(100.0, 69.1)`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-24T07:10:36Z |
| Candidate model | claude-5.5-opus |
| Candidate effort | max |
| Candidate harness | claude-code |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **120 / 120** = **100.0%**.

By PRD file:

| File | Total | Captured | Rate |
| --- | --- | --- | --- |
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 20 | 100.0% |
| aviary_layout.md | 18 | 18 | 100.0% |
| accounts_sync.md | 18 | 18 | 100.0% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| Total | 120 | 120 | 100.0% |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
| --- | --- | --- | --- | --- |
| 1 | Headline product concept statement | product_brief.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Borderline inclusive: no standalone glossary, but domain vocabulary is enforced by terminology lints and typed model sections. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Borderline inclusive: the plan captures procedural call captions, though it avoids explicit mood-label captions. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured by a specific invariant, implementation section, or API/model surface. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **75.0%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| S1 - feels-alive-not-robotic | 4 | included | B: 'Affective aliveness is an engineering invariant'; 'First-bird edge boot and no spinner' | PLAN §§1,2,9.1,11: 'First frame is the aviary'; accessibility charm gates | yes | yes | no | partial | Recovered aliveness as cross-cutting and first-frame/procedural expression; downstream staleness consequence was thin. |
| S2 - notice-never-announce | 4 | included | B: 'Quiet noticing replaces announcement'; 'The greeting is the welcome' | PLAN §§2,3.3,9.5,12.2: no toasts/badges/streaks; greeting as welcome | yes | yes | no | partial | Recovered no-toast/greeting-as-welcome rule; not the full one-toast-corrupts-the-surface consequence. |
| S3 - charm-from-specificity | 2 | included | B: 'Lowercase, present-tense, specific prose with bird verbs' | PLAN §§7.14,11.1,12: naturalist prose, named birds, terminology lints | yes | yes | no | full | Naturalist specificity, named birds, and avoidance of generic gamified copy survived. |
| S4 - restraint-over-richness | 2 | included | B: no chrome in scene; cap gates by recognizability/performance/layout | PLAN §§1,2,3,9.2,9.8,16.2: 2-7 birds, one scene, no chrome | no | yes | no | partial | The plan preserved restraint, but B did not identify it as a distinct cross-cutting principle. |
| S5 - naturalist-voice-with-system-exception | 2 | included | B: 'Voice is split, registered, and machine-checked' | PLAN §§6.9,12,13.6: naturalist/system registers and matter-of-fact errors/emails | yes | yes | no | full | The voice split and machine enforcement were cleanly recovered. |
| S6 - presence-is-real-interaction | 4 | included | B: 'Honest attention is valuable'; 'visible, focused, recent activity' | PLAN §§1,2,7.4,8.4,9.10: conjunction, union, saturation, hidden-tab behavior | yes | yes | no | partial | Recovered honest-attention precision and inflation controls; settle/tab-close equivalence was not fully reconstructed. |
| S7 - simulation-runs-server-side | 4 | included | B: 'Determinism is a product trust mechanism'; 'Server-authoritative deterministic simulation' | PLAN §§1,4,7.1-7.2,8.1: server tick, clients never tick, no lost drift | yes | yes | no | full | Server-authoritative state, multi-device coherence, and no last-write-wins implications survived. |
| S8 - privacy-first-on-bird-data | 2 | included | B: 'Privacy is architectural, not merely policy' | PLAN §§1,4.7,13.9,14.4: separate DBs/networks, aggregate telemetry | yes | yes | no | full | The architectural privacy boundary and per-bird data constraint were fully recovered. |
| S9 - accessibility-as-first-class-surface | 4 | included | B: 'Accessibility is a designed register of the same product' | PLAN §§1,9.7,11,16.3: reduced motion, narration, captions, launch gates | yes | yes | no | full | Same-product accessibility, implementation cost, and v1 gating survived. |

For multi-layer system-level whys:

| Why ID | L1 primary | L2 secondary | L3 downstream |
| --- | --- | --- | --- |
| S1 | yes | yes | no |
| S2 | yes | yes | no |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: PLAN §§1,2,7.2,9.1,9.4-9.5,10,11,14: first frame, procedural audio, idle motion, no spinner, accessibility charm.
- S2: PLAN §§2,3.3,7.14,9.5,12.2,13.6,18.5: no toasts/badges/streaks, greeting as welcome, no re-engagement emails.
- S3: PLAN §§7.14,11.1,12: field-notebook/narration/caption grammar, named birds, terminology lints, no numeric traits.
- S4: PLAN §§1,2,3.2,9.2,9.8,16.2: two starters, cap seven, single scene, no chrome, recognizability gates.
- S5: PLAN §§6.9,12,13.6: naturalist register for aviary surfaces and system register for auth/error/settings/emails.
- S6: PLAN §§1,2,7.4,7.6,8.4,9.10,18.1: presence conjunction, union, saturation, hidden-tab pause, no punishment.
- S7: PLAN §§1,4,5.3,7.1-7.2,8.1: server tick, single writer, append-only events, projection and convergence tests.
- S8: PLAN §§1,4.7,5.7,13.9,14.3-14.4: DB/network separation, telemetry allowlist, export/deletion boundaries.
- S9: PLAN §§1,9.7,10.8-10.9,11,16.3: designed reduced motion, naturalist narration, captions, keyboard, launch gates.


### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **67.7%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1 | presence-definition | 4 | yes | included | B: presence is 'visible, focused, recent activity'; 'honest attention' | PLAN §7.4: conjunction, clamping, union | no | partial | Missing silent long-run corruption layer. |
| F2 | drift-function | 4 | yes | included | B: 'Leaky pressure integrator'; Tamagotchi/screensaver risk | PLAN §7.5 and §18.1: low-pass, calibration bands, DC-1 risk | no | partial | Omitted one-week instrument / three-week user split. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | B: attention makes birds expressive without neglect lowering traits | PLAN §§1,7.5-7.6,18.1: no negative path; attunement only | no | full | No-punishment rationale survived. |
| F4 | procedural-call-grammar | 4 | yes | included | B: procedural calls avoid audio files; no exact repeat guard | PLAN §§1,10.3-10.10: procedural grammar, no recordings, fallback | no | partial | Missed stacked-loop/phase artifact layer. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | B: visible behavior matches what a viewer would have seen | PLAN §§7.7,9.4,11.1: mood-derived posture/action; narration avoids labels | no | full | Recovered mood read from motion, not labels. |
| F6 | bird-count-cap-7 | 2 | yes | included | B: cap rises only after recognizability/performance/layout gates | PLAN §16.2: 7-bird recognizability gate and DB cap | no | full | Recognizability ceiling survived. |
| F7 | personality-vector-persistence | 4 | yes | included | B: identity and accumulated state are sacred; no replay-to-rebuild path | PLAN §§7.1,13,18.2: stored canonical personality; personality loss worst failure | no | full | Persistence and relationship-loss rationale survived. |
| F8 | personality-vector-never-numerical | 2 | yes | included | B: hidden personality legible only through living behavior | PLAN §§1,2,11.1,13.5: no trait values in product snapshots/client | no | full | Recovered relationship-preserving reason for hiding numbers. |
| F9 | return-greeting | 4 | yes | included | B: arrival noticed through gaze/calls/steps/re-orientation | PLAN §9.5: absence buckets, greeter weights, fresh entropy | no | partial | Generic-arrival-animation failure layer was thin. |
| F10 | no-welcome-back-toast | 4 | yes | included | B: 'The greeting is the welcome'; no toasts/banners/badges | PLAN §§2,3.3,12.2: no toast/badge components; banned copy | no | full | No textual welcome and variants recovered. |
| F11 | settle-is-opt-in | 2 | yes | included | none | none | yes | none | Rule survived, but chore/penalty rationale for closing without settle was not reconstructed. |
| F12 | field-notebook-auto-entries | 4 | yes | included | B: notebook detectors from aviary facts only; sparse, noteworthy entries | PLAN §§7.14,12.3: naturalist grammar, sparse budget, not per-session | no | partial | Event-log-breaks-spell layer was not explicit. |
| F13 | presence-accounting | 4 | yes | included | B: coverage union, stale rejection, daily saturation curve | PLAN §7.4: all three conditions, union, staleness, saturation | no | partial | Silent population-wide failure layer under-specified. |
| F14 | no-streak-counter | 4 | yes | included | B: no gamification, no visit-frequency surfaces; notebook omits presence/session facts | PLAN §§2,3,7.14,14.4: no streaks/calendars; notebook excludes user behavior | no | partial | Less explicit on rotation from birds to number-management. |
| F15 | scene-loads-with-motion | 4 | yes | included | B: first frame must already be the aviary; quiet field fallback | PLAN §§1,9.1: birds mid-action, inline snapshot, no spinner | no | full | First-frame aliveness and no-spinner rationale survived. |
| F16 | synthetic-account-id | 4 | yes | included | B: synthetic account UUID plus encrypted email keeps references on UUIDs/blind indexes | PLAN §§2,5.1,13.1: UUID account IDs, encrypted email, blind index | no | partial | Retrofit-impossibility layer was not reconstructed. |
| F17 | server-side-simulation-tick | 4 | yes | included | B: server-authoritative deterministic simulation; clients never tick | PLAN §§1,7.2,8.1: server tick, clients render snapshots, convergence tests | no | partial | Divergent-client failure mode was not fully reconstructed. |
| F18 | no-last-write-wins-personality | 4 | yes | included | B: only sim_writer updates; no lost drift across devices | PLAN §§2,5.3,8.1: append-only events; server tick sole writer | no | full | Additive deltas and lost-drift risk survived. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | B: system register is direct/actionable, with no warmth standing in for information | PLAN §§6.9,12.1: account/error/sync surfaces matter-of-fact | no | full | Recovered error-context exception. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | B: per-bird interactions used only to run user's own aviary; telemetry boundary architectural | PLAN §§4.7,13.9,14.4: no per-bird analysis or ML training data | no | partial | Private-relationship-as-data-product layer was less explicit. |
| F21 | visit-read-only-ambient | 2 | yes | included | B: no visitor heartbeats/events; visitors see the same ambient aviary | PLAN §§6.7,8.6: visitor snapshot read-only; no event writer | no | full | Observation-not-co-presence and no accidental drift recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | B: quiet noticing replaces announcement; visit notification email opt-in and bounded | PLAN §§3,6.7,13.6: off by default; no push/in-product visit notification | no | full | Default no-notification rationale recovered. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | B: data not computed so leaderboards cannot just be exposed | PLAN §§3.2,14.4,18.5: no public discovery/ranking or visits-per-host stats | no | full | Relationship-not-comparison survived. |
| F24 | sr-narration-running-prose | 4 | yes | included | B: naturalist accessibility surfaces; narration describes posture, never labels/numbers | PLAN §§11.1,12.1: running naturalist prose, not state list | no | partial | ARIA-automation wrong-feature consequence not explicit. |
| F25 | reduced-motion-mode | 4 | yes | included | B: reduced-motion renderer separately designed so users still get the product | PLAN §§9.7,11.3,16.3: cross-fade renderer; calls/drift/notebook unchanged | no | full | Reduced-motion charm preservation survived fully. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | B: first-bird budgets keep first-bird path fast and avoid loading experience | PLAN §§9.1,14.1: first bird <=500 ms; first frame already aviary | no | full | Recovered performance as felt aliveness. |
| F27 | no-gamification | 4 | yes | included | B: no gamification, no pressure; no components exist for announcements/gamification | PLAN §§2,3,18.5: no achievements/streaks/scores; lints/audits | no | partial | Foothold cascade only partially represented. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | B: birds do not die, starve, or show distress; absence has no mood effect | PLAN §§2,7.5-7.6,18.1: no distress; no negative drift | no | full | Observational-not-custodial survived. |
| F29 | starter-birds-not-catalog | 2 | yes | included | B: adoption sheet introduces birds as arrivals; live vignettes before naming | PLAN §§12.4,13.2: 'two birds have come'; system-selected starters | no | full | Meeting arrivals, not catalog choices, survived. |
| F30 | age-based-new-bird-offers | 4 | yes | included | B: new birds by aviary age only avoids visit-frequency or achievement triggers | PLAN §§7.15,16.2: eligibility purely age-based; silent deferral | no | partial | Relationship-deepening-over-time layer was thin. |
| F31 | stable-bird-identity | 4 | yes | included | B: stable bird IDs and pinned species versions; identity and accumulated state are sacred | PLAN §§2,5.3,5.6,16.1: stable UUIDs, pinned species, beta data permanent | no | full | Identity continuity and retroactive loss consequence survived. |
| F32 | mood-persists-across-sessions | 2 | yes | included | B: birds persist mood through sessions and daily rhythms | PLAN §§7.7,8.3: no tab-open reset; mood blends and roosts by circadian cycle | no | full | No neutral reset recovered. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Read-only rule survived, but journal/curation rationale was not reconstructed. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | Export mechanism survived; quiet copy of relationship rationale did not. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | B: recover through I changed my mind page; hard delete with crypto-shred | PLAN §§13.7,5.7: 30-day soft delete, hard delete rows and DEK | no | partial | Partial-retention-as-company-residue layer was not explicit. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | B: telemetry without cookies/IDs/credentials; observability back door avoided | PLAN §§4.7,14.3-14.4: allowlisted aggregate telemetry, no relationship dimensions | no | full | Technical telemetry boundary survived. |
| F37 | per-invite-named-sharing | 2 | yes | included | none | none | yes | none | Per-invite rule survived, but private-relationship/control rationale was not recovered. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | none | yes | none | Visit-log mechanism survived, but transparency-not-social-loop rationale did not. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | B: visitors see the same ambient aviary | PLAN §6.7: same birds, moods, weather, timezone/day-night; no special rendering | no | full | Actual aviary vs show-off rendering recovered. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | B: cadence uses calm timing so narration is naturalist but not chatty | PLAN §11.1: idle every 30-60s, global gap, queue caps | no | partial | Monitoring-vs-presence consequence was partial. |

For multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
| --- | --- | --- | --- |
| F1 | yes | yes | no |
| F2 | yes | no | yes |
| F3 | yes | yes | yes |
| F4 | yes | no | yes |
| F7 | yes | yes | yes |
| F9 | yes | yes | no |
| F10 | yes | yes | yes |
| F12 | yes | no | yes |
| F13 | yes | yes | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | no |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | no |
| F25 | yes | yes | yes |
| F27 | yes | yes | no |
| F30 | no | yes | yes |
| F31 | yes | yes | yes |
| F35 | yes | yes | no |
| F40 | yes | yes | no |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
| --- | --- | --- |
| Possible gold whys | 49 | From score JSON gold_why_totals |
| Possible total weight | 152 | From intent_recovery.total_possible_weight |
| Reachable gold whys | 49 | All feature anchors captured |
| Excluded unreachable feature whys | 0 | No denominator exclusions |
| Recovered / reachable weight | 105.0 / 152 | Sum of weight x recovery |
| Whys with reconstruction evidence | 44 | Included rows with exact B-side rationale evidence |
| Whys with PLAN grounding | 44 | Included rows with exact plan-side rationale grounding |
| rule_without_why cases | 5 | Mechanism survived without gold rationale |
| plan_only_not_reconstructed cases | 18 | At least one rationale layer present in PLAN but missing in reconstruction |
| ungrounded_reconstruction cases | 0 | No confabulated rationale found |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weighted credit | Recovery rate |
| --- | --- | --- | --- |
| Functional whys | 54 | 36.0 | 66.7% |
| Affective whys | 98 | 69.0 | 70.4% |
| Weight 2 whys | 44 | 33.0 | 75.0% |
| Weight 3 whys | 108 | 72.0 | 66.7% |
| System-level whys | 28 | 21.0 | 75.0% |
| Feature-level whys (reachable) | 124 | 84.0 | 67.7% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered 36.0/54 weighted points (66.7%). Affective whys recovered 69.0/98 (70.4%). Functional losses were mostly calibration/privacy sublayers; affective losses concentrated in small social/account exceptions.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 72.0/108; weight-2 whys recovered 33.0/44. High-weight rows often got partial credit because B recovered mechanisms and primary causes but compressed downstream consequences.
- **System-level vs feature-level.** System-level fidelity (75.0%) exceeded feature-level fidelity (67.7%). The plan's invariants carried cross-cutting intent well; narrower feature whys such as F33, F34, F37, and F38 leaked.
- **Multi-layer recovery patterns.** The downstream-consequence layer was the most common drop. Examples: S1, S2, F1, F9, F13, F16, F17, F24, and F40.
- **Subdomain patterns.** Simulation, privacy architecture, accessibility, and audio were strong. Social optionality and account lifecycle surfaces had the most rule-without-why failures.
- **Evidence-bound effects.** Five rows were mechanism-only recoveries: F11, F33, F34, F37, F38. Eighteen rows had at least one plan-only layer that the reconstruction did not carry through.

What this suggests: the candidate plan is highly complete and richly engineered, but the reconstruction compressed many rationales into operational summaries.

---

## 4. Recommendations for v2 hardening

- Keep targeted headroom additions like F29-F40; they exposed meaningful fidelity loss even with 100% feature capture.
- Add scorer guidance for consequence-layer omissions. This run repeatedly recovered primary mechanisms while dropping why-it-matters-if-broken layers.
- Preserve the evidence-bound operator. Without it, rule-only rows such as F33/F34/F37/F38 would be easy to over-credit.
- Consider a slightly more structured system-level worksheet for the 3-feature cross-cutting bar, since S1/S2/S6 required fine judgment.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** The reconstruction showed no gold ID leakage and followed the PLAN structure. The same-context safeguard appears to hold.
- **Single-run limitation.** This is one run only; no variance signal is available.
- **Borderline capture calls.** Feature 7 and feature 104 were inclusive calls. Neither materially affects planning quality.
- **System-level cross-cutting.** S1, S2, S4, and S6 involved judgment about whether B identified the principle versus the PLAN merely preserving it.
- **Confabulation cases.** None found. Partial rows were scored conservatively for missing layers, not for invented content.
- **Evidence-bound denials.** F11, F33, F34, F37, and F38 were the clearest mechanism-only denials.
- **Rule-without-why cases.** These five cases counted for planning capture but not for why recovery.

---

End of report.
