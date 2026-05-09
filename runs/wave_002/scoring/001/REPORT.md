# REPORT -- CARE run 001

> Phase 2B scoring report for wave_002 slot 001. Companion artifacts: frozen `RECONSTRUCTION.md`, strict score JSON at `runs/wave_002/scores/run_001.json`, and the interactive HTML twin `REPORT.html`.
>
> Variant v06 evidence-bound clean + targeted gold headroom: every S1-S9 and F1-F40 row below separates denominator status from recovery and records frozen-reconstruction evidence plus PLAN grounding where available. Rule-only evidence is called out separately and does not count as why recovery.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **63.3%** |
| Intent fidelity | **40.6%** |
| Combined quality | **6274** |

**Diagnostic split:**

- System-level fidelity: **50.0%**
- Feature-level fidelity: **38.0%**

**(Planning, fidelity) coordinate:** `(63.3, 40.6)` -- plot on a 2D scatter with both axes 0-100; upper-right is best.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-09T13:55:09Z |
| Candidate model | gemini-3.1-pro-preview |
| Candidate effort | low |
| Candidate harness | gemini-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **76 / 120** = **63.3%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 3 | 50.0% |
| concepts.md | 4 | 2 | 50.0% |
| bird_engine.md | 22 | 18 | 81.8% |
| interactions.md | 20 | 8 | 40.0% |
| aviary_layout.md | 18 | 6 | 33.3% |
| accounts_sync.md | 18 | 12 | 66.7% |
| social_optional.md | 10 | 7 | 70.0% |
| accessibility_perf.md | 18 | 16 | 88.9% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **76** | **63.3%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Scope describes a web-only virtual aviary. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | no | Implementation hints exist, but the design philosophy is not stated. |
| 3 | "Notice, never announce" principle callout | product_brief.md | no | No explicit notice-vs-announce principle or welcome-surface rule. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | no | Naturalist prose appears, but the matter-of-fact system exception is absent. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Out-of-scope section names gamification, Tamagotchi mechanics, and social network surfaces. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Scope and rollout specify two starters and a maximum of seven. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No glossary or concept-definition surface beyond data-model bullets. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Presence is defined as visible + focused + recent input. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Data model separates slow personality vector and fast mood. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | no | Settle appears only as an event type. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Vector traits are enumerated. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Drift function is named as a low-pass filter. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | no | No one-week/three-week calibration target. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Drift is monotonic upward; negative drift is excluded. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Mood is defined as fast-timescale enumerated state. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Mood updates use local time, weather, and recent interactions. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Client WebAudio uses species-specific motifs. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Rollout caps birds to preserve per-bird audio recognizability. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Real-time mixing prevents phase cancellation. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Timing and pitch are modulated by vocal frequency and mood. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Preening and scanning micro-motion are specified. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Idle micro-motion is mood-shaped. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Rollout starts from a pool of about six species. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | borderline: User-assigned name is captured; renameability is not explicit. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | no | Two starters are captured, but system auto-selection/no catalog is not. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Maximum seven birds is explicit. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Additional birds are based purely on aviary age. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Server owns personality state; clients never own absolute values. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | borderline: Server-owned mood state implies persistence, though no-neutral-reset is not explicit. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | no | Chorus is present, but bird-to-bird reactions are not specified. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Bird has a stable internal ID. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | no | No rule hides trait values from users. |
| 33 | Return-greeting on viewer arrival | interactions.md | no | No arrival greeting feature. |
| 34 | Greeting variation by absence length | interactions.md | no | No absence-length greeting variation. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | No greeting behavior tied to boldness. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | no | No greeting stagger rule. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No explicit textual welcome-surface refusal. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Listen-in focuses a bird with a gradual volume ramp. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Other birds fade to ambient levels. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | no | Offer is only an event type, not the feature contents. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | No offer reaction logic. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | no | No cooldown rule. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | no | Settle appears only as an event type. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | No close-tab equivalence or opt-in wording. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Tick occasionally generates naturalist field notebook entries. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | borderline: Occasional generation captures rarity; noteworthy criteria are absent. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Read-only field notebook is in scope. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Presence accounting is in scope and drives drift. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | The strict conjunction is explicit. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | borderline: No streaks is explicit; days-visited variants are not. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | Visibility is used for presence, not render-pausing. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | No undo window. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Single responsive horizontal scene is specified. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Three depth planes/front-middle-back perches are specified. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | no | Perch zones exist, but bird choice/user non-placement is absent. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | no | Local time affects mood; no day/night scene cycle. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | no | No evening palette/call quieting rule. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | No night state. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | no | Weather affects mood, but ambient weather rendering is not specified. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Mood updates use weather. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | borderline: Ambient particle drift is captured, though leaf/feather identity is generic. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Subtle parallax is specified. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | no | No chrome/top-bar placement rule. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | no | No top-bar contents. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | no | No auto-fade rule. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Idle animations are active immediately on load. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | no | No quiet-field loading state. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-aviary state. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | No palette spec. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | no | Responsive scene is captured; no never-crop rule. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Magic-link sign-in endpoints are specified. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | No expiry duration. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Single-user accounts are in scope. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Account is indexed by synthetic UUID; email encrypted separately. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Server cron runs roughly once per minute. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | borderline: Client pulls snapshots, but visibility-trigger specificity is absent. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Client handles interpolation. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Devices pull the same canonical snapshot. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Clients never write absolute values, preventing last-write-wins conflicts. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Server owns personality and processes event logs sequentially. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | no | No sync-conflict UI copy/tone rule. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | no | Per-device tokens are mentioned, but revocation is absent. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | borderline: Export generation is present; JSON contents are not detailed. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | no | Only soft-deletion endpoints are named. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | borderline: No per-bird state/interaction logs in aggregate telemetry; ML is not named. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Aggregate telemetry and privacy boundary are explicit. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | No email-change flow. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Opt-in invitation system stores visitor email and invite token. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | borderline: Opt-in visits imply default off. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Read-only visit invitation system and no co-presence are in scope. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | borderline: Read-only/no co-presence implies no visitor interactions. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | no | Generic social exclusions do not name these surfaces. |
| 94 | No "your friend visited!" notification by default | social_optional.md | no | No notification rule. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | DELETE invite endpoint revokes. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | GET invites is labeled as visit log. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | no | No show-off-mode refusal. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | No public directories or leaderboards are specified. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Narration is naturalist prose sourced from semantic state. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Slow cadence and queue-overwhelm risk are specified. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Naturalist prose updates are explicit. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Cross-fades replace frame-by-frame motion and particles. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | A specifically designed aesthetic is specified. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | no | Captions exist, but no toggle/mood text rule. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | WCAG AA contrast is specified. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Keyboard navigation keys are specified. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | High-contrast focus indicators are specified. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Initial JS budget is specified. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | 500ms target on mid-tier mobile 4G is specified. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | 60fps target is specified. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Zero memory growth budget is specified. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Client-side WebAudio procedural synthesis is specified. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Fallback is graceful silence with call captions. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | RUM, synthetic checks, and aggregate telemetry are specified. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | p99 tick-latency alarm is specified. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | No browser matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Native mobile apps are out of scope. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Gamification is explicitly out of scope. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Tamagotchi mechanics are explicitly out of scope. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Social network surfaces are out of scope. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **50.0%**. Weighted recovery: **14.0 / 28**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 -- feels-alive-not-robotic | 4 | included | System intent: "Semantic state drives rich local experience"; client handles "procedural variation"; slow cadence appears across surfaces. | PLAN Render Pipeline: "procedural variation"; Frontend: "active immediately upon load"; Audio: "Procedural Call Synthesis". | yes | yes | no | partial | Aliveness survives as procedural/motion/audio architecture, but the downstream staleness/fail-one-place rationale is absent. |
| S2 -- notice-never-announce | 4 | included | none | Rule fragments only: Scope excludes "gamification" and "social network surfaces"; no welcome-toast or notification rationale. | no | no | yes | none | The plan refuses some engagement surfaces but does not encode notice-vs-announce. |
| S3 -- charm-from-specificity | 2 | included | System intent: "Naturalist field-observation product voice" and "observational rather than game-like." | PLAN includes "naturalist prose string," "naturalist field notebook entries," and naturalist screen-reader prose. | yes | yes | no | full | Specific observational voice and anti-game language are preserved across notebook and narration surfaces. |
| S4 -- restraint-over-richness | 2 | included | System intent: maximum of 7 "to preserve per-bird audio recognizability." | Scope/Rollout: start with 2, cap at 7; Frontend: "single responsive horizontal scene"; Audio cap preserves recognizability. | yes | yes | no | partial | The bird-count/audio-recognizability part survives, but one-screen/calm/no-chrome restraint is only partly encoded. |
| S5 -- naturalist-voice-with-system-exception | 2 | included | System intent: "Naturalist field-observation product voice." | PLAN naturalist notebook and screen-reader prose; account/error/sync matter-of-fact exception is absent. | yes | no | yes | partial | Naturalist voice survived; the system/error exception did not. |
| S6 -- presence-is-real-interaction | 4 | included | Per-feature: presence uses "presence-time and interactions"; over-counting makes drift "artificially accelerate." | Simulation: strict conjunction of visible/focus/input; Drift based on "presence-time and interactions"; Risk: over-counting accelerates drift. | yes | yes | no | partial | Precise presence and overcount risk survive; settle/tab-close equivalence and relationship-shape consequence do not. |
| S7 -- simulation-runs-server-side | 4 | included | System intent: "single canonical server state, not client-owned state"; avoids "last-write-wins conflicts." | Architecture: server "authoritative source of truth"; Sync: clients never write absolute state; devices pull same snapshot. | yes | yes | no | partial | Server canonicality and sync coherence survive; the divergent-client failure mode is compressed away. |
| S8 -- privacy-first-on-bird-data | 2 | included | System intent: "Privacy boundary" with "Aggregate telemetry only" and "Absolutely no per-bird state or interaction logs." | Performance/Observability: aggregate telemetry only; Privacy Boundary excludes per-bird state and interaction logs. | yes | yes | no | full | The technical telemetry boundary is clearly preserved. |
| S9 -- accessibility-as-first-class-surface | 4 | included | System intent: "Accessibility as a designed surface, not only a fallback." | Accessibility includes SR narration, captions, keyboard/focus/contrast; Reduced Motion is a "specifically designed aesthetic." | yes | yes | no | partial | Designed accessible surfaces survive; ship-with-v1/downstream exclusion consequence is not explicit. |

For multi-layer system-level whys:

| Why ID | L1 / L2 / L3 recovered? | Note |
|---|---|---|
| S1 | yes / yes / no | Aliveness survives as procedural/motion/audio architecture, but the downstream staleness/fail-one-place rationale is absent. |
| S2 | no / no / no | The plan refuses some engagement surfaces but does not encode notice-vs-announce. |
| S6 | yes / yes / no | Precise presence and overcount risk survive; settle/tab-close equivalence and relationship-shape consequence do not. |
| S7 | yes / yes / no | Server canonicality and sync coherence survive; the divergent-client failure mode is compressed away. |
| S9 | yes / yes / no | Designed accessible surfaces survive; ship-with-v1/downstream exclusion consequence is not explicit. |

**Cross-cutting evidence appendix:**

- S1: Render Pipeline procedural variation; Frontend motion active immediately on load; Audio procedural synthesis; Reduced Motion designed cross-fades. Count >=3, but downstream staleness layer absent.
- S2: No gamification and no public/social surfaces only. No welcome-toast, notification, or announcement-surface pattern. Count <3 for the actual principle.
- S3: Naturalist notebook prose; naturalist screen-reader prose; caption/audio specificity; named birds/species motifs; anti-gamification. Count >=3.
- S4: Start with two/max seven; single horizontal scene; maximum seven tied to audio recognizability; limited species pool. Count >=3, but calm/no-chrome parts missing.
- S5: Naturalist notebook and screen-reader prose are present; account/error/sync matter-of-fact exception is missing. Count <3 for the full split.
- S6: Presence conjunction; drift input; over-count risk; no gamification/no streaks as a subordinate refusal. Count >=3, but settle-vs-close-tab equivalence missing.
- S7: Server tick; canonical server state; clients pull snapshots; append-only event log; no absolute client writes. Count >=3.
- S8: Encrypted email separation; aggregate telemetry only; no per-bird state/interaction logs in telemetry; read-only/private social constraints. Count >=3.
- S9: Screen-reader prose; reduced-motion designed aesthetic; captions; keyboard/focus/contrast; WCAG AA. Count >=3, but ship-with-v1 rationale is implicit only.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **38.0%**. Reachable feature-level whys: **31 / 40**; the other 9 had unreachable anchors because the feature was not captured. Weighted recovery: **38.0 / 100.0**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | Per-feature: strict conjunction protects integrity because over-counting makes drift "artificially accelerate." | PLAN: strict conjunction of visible/focus/recent input; risk says over-counting accelerates drift. | no | partial | L1 recovered; shortcut-signal examples and silent failure/load-bearing consequence are absent. |
| F2 | drift-function | 4 | yes | included | Per-feature: "low-pass filter" and risk that too aggressive feels "Tamagotchi" while too slow feels "unresponsive." | PLAN Drift Function and Risks use the same low-pass/Tamagotchi/unresponsive framing. | no | partial | Slow filter and failure endpoints survive; one-week/three-week calibration is absent. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | System intent: "no punitive drift"; per-feature drift is "monotonically upwards." | PLAN: monotonic upward drift; Scope excludes "hunger, death, or negative drift." | no | partial | Positive-only drift and no-Tamagotchi rationale survive; the two-week-return relationship consequence is absent. |
| F4 | procedural-call-grammar | 4 | yes | included | Per-feature: calls use "species-specific motifs" and chorus mixing prevents "phase-cancellation." | PLAN Audio: WebAudio motifs; real-time mixing prevents phase-cancellation; fallback is silence with captions. | no | partial | Procedural calls and chorus rationale survive; audio-as-affective-spine cascade is not stated. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | Rule only: "mood-shaped" continuous animations such as "preening" and "scanning." | Rule only: Frontend says idle micro-motion is "mood-shaped"; no label/tooltip rationale. | yes | none | Mechanism survives, but not the why that users should read mood without labels. |
| F6 | bird-count-cap-7 | 2 | yes | included | Per-feature: cap at 7 "to preserve per-bird audio recognizability." | PLAN Rollout: maximum 7 "to preserve per-bird audio recognizability." | no | full | The recognizability rationale is directly present. |
| F7 | personality-vector-persistence | 4 | yes | included | Per-feature: server owns personality/mood; clients never write absolute values, preventing LWW conflicts. | PLAN Sync: server owns personality/mood; clients never write absolute state values. | no | partial | Canonical server persistence and sync consequences survive; losing-vector-as-deleting-bird does not. |
| F8 | personality-vector-never-numerical | 2 | no | unreachable_excluded | none | Anchor feature not captured in PLAN. | no | unreachable | Unreachable because the plan never forbids numeric trait exposure. |
| F9 | return-greeting | 4 | no | unreachable_excluded | none | Anchor feature not captured in PLAN. | no | unreachable | Unreachable because return greeting is absent. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | Anchor feature not captured in PLAN. | no | unreachable | Unreachable because no welcome-toast/banner rule is absent. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | Anchor feature not captured in PLAN. | no | unreachable | Unreachable because settle opt-in/close-tab equivalence is absent. |
| F12 | field-notebook-prose | 4 | yes | included | Per-feature: procedural notebook generation creates "naturalist field notebook entries" from templates. | PLAN: Notebook Entry is a "Naturalist prose string"; tick "occasionally generates" entries. | no | partial | Naturalist prose layer survives; concentrated-voice and rare/read-only rationale are absent. |
| F13 | presence-accounting | 4 | yes | included | Per-feature: presence uses strict conjunction and over-counting accelerates population drift. | PLAN: visible + focus + recent input; presence-time dominates drift; over-counting accelerates drift. | no | partial | Core precision survives; individual-signal misses and silent failure narrative are absent. |
| F14 | no-streak-counter | 4 | yes | included | Rule only: Scope says "gamification (no streaks, levels, or achievements)." | Rule only: PLAN excludes streaks; no days-visited/calendar/behavior-observation rationale. | yes | none | The no-streak rule is present without the counter-vs-birds rationale. |
| F15 | scene-loads-with-motion | 4 | yes | included | none; reconstruction says "No entry/loading animations: NOT RECOVERABLE FROM PLAN." | Rule only: PLAN says animations are "active immediately upon load (no entry/loading animations)." | yes | none | Captured feature, but B marked the rationale unrecoverable. |
| F16 | synthetic-account-id | 4 | yes | included | Per-feature: account uses synthetic UUID and encrypted email "strictly for authentication and account exports." | PLAN Account: indexed by synthetic UUID; email stored encrypted for authentication/export. | no | partial | Identifier rule survives; PII leak/compliance and impossible-retrofit layers are absent. |
| F17 | server-side-sim-tick | 4 | yes | included | Per-feature: tick runs server-side at "slow cadence" and keeps canonical state. | PLAN: server cron ~once/minute processes event log; server owns canonical state; devices pull snapshots. | no | partial | Tick and sync coherence survive; client-tick collapse consequence is absent. |
| F18 | no-last-write-wins | 4 | yes | included | Per-feature: clients never write absolute state, preventing "last-write-wins conflicts." | PLAN: append-only log; clients never write absolute state; server processes event logs sequentially. | no | partial | Implementation rule survives; concrete cross-device data-loss example is absent. |
| F19 | sync-conflict-matter-of-fact | 2 | no | unreachable_excluded | none | Anchor feature not captured in PLAN. | no | unreachable | Unreachable because sync conflict/error tone is absent. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | Per-feature: aggregate telemetry only; "Privacy Boundary" excludes per-bird state or logs. | PLAN: aggregate telemetry only; no per-bird state or interaction logs in aggregate telemetry. | no | partial | Technical boundary survives; private-relationship/data-product and ML-specific rationale are absent. |
| F21 | visit-read-only-ambient | 2 | yes | included | Per-feature: read-only opt-in visit system excludes "co-presence." | PLAN Scope: read-only opt-in visit invitations; out of scope includes "co-presence." | no | full | Read-only/no-co-presence rationale is present enough for the single-layer why. |
| F22 | no-friend-visited-notification | 2 | no | unreachable_excluded | none | Anchor feature not captured in PLAN. | no | unreachable | Unreachable because no friend-visited notification rule is absent. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | Rule only: no "public directories, leaderboards, or co-presence." | Rule only: PLAN excludes leaderboards/public directories; comparison/data-pipeline rationale absent. | yes | none | The refusal survives, not the why. |
| F24 | sr-narration-running-prose | 4 | yes | included | Per-feature: narration is "slow-cadence, naturalist prose updates" from semantic state. | PLAN Accessibility: naturalist prose narration sourced from semantic aviary state. | no | partial | Running prose survives; same-right-to-feel and ARIA-automation failure are absent. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | System intent: accessibility is "a designed surface, not only a fallback"; reduced motion uses "slow, calming cross-fades." | PLAN: reduced motion is a "specifically designed aesthetic" using cross-fades. | no | partial | Different-rendering and not-fallback rationale survive; continuing calls/drift/notebook layer is incomplete. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | Rule only: instrumentation monitors "500ms Time-to-first-bird." | Rule only: PLAN budget is first bird <500ms; no affective-perf threshold rationale. | yes | none | Metric is captured without the why that 500ms preserves already-running aliveness. |
| F27 | no-gamification | 4 | yes | included | Rule only: Scope excludes "gamification (no streaks, levels, or achievements)." | Rule only: PLAN excludes gamification; no adjacent-product temptation or erosion rationale. | yes | none | The prohibition survives without the relationship rationale. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | System intent: "no punitive drift"; Scope excludes hunger/death/negative drift. | PLAN excludes "hunger, death, or negative drift" and flags Tamagotchi-feeling drift as a risk. | no | full | The plan and reconstruction recover the punishment-of-absence rationale. |
| F29 | starter-birds-not-catalog | 2 | no | unreachable_excluded | none | Anchor feature not captured in PLAN. | no | unreachable | Unreachable because auto-selected arrivals/no catalog is absent. |
| F30 | age-based-new-bird-offers | 4 | yes | included | Rule only: additional birds offered "based purely on aviary age." | Rule only: PLAN says based purely on aviary age; reward/economy rationale absent. | yes | none | Age-based unlock survives without the anti-reward-loop why. |
| F31 | stable-bird-identity | 4 | yes | included | none; reconstruction says "Bird with stable internal ID... NOT RECOVERABLE FROM PLAN." | Rule only: PLAN data model has "Stable internal ID"; identity-continuity rationale absent. | yes | none | B marked this unrecoverable, so the why scores none. |
| F32 | mood-persists-across-sessions | 2 | yes | included | Rule only: server owns "all personality and mood state." | Rule only: PLAN server owns mood state; no no-neutral-reset/continued-while-gone rationale. | yes | none | Mood persistence is only implicit and lacks the why. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | none; reconstruction says "Read-only field notebook: NOT RECOVERABLE FROM PLAN." | Rule only: Scope includes a "read-only field notebook." | yes | none | Read-only rule survives, observer-record rationale does not. |
| F34 | account-export-relationship-copy | 2 | yes | included | Rule only: export generation is one reason email is retained. | Rule only: PLAN has account export generation; relationship-copy/quiet-QoL rationale absent. | yes | none | Export feature survives without its relationship-right rationale. |
| F35 | account-deletion-grace-then-hard-delete | 4 | no | unreachable_excluded | none | Anchor feature not captured in PLAN. | no | unreachable | Unreachable because 30-day recovery and hard-delete are absent. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | Per-feature: "Aggregate telemetry only" and no per-bird state or interaction logs. | PLAN Observability/Privacy Boundary: aggregate request counts/latency histograms; no per-bird state/logs. | no | full | The technical telemetry boundary is clear and grounded. |
| F37 | per-invite-named-sharing | 2 | yes | included | Rule only: read-only opt-in visit invitation system with visitor email/token. | Rule only: PLAN has visitor email and invite token; private-relationship/named-control rationale absent. | yes | none | The invite mechanism survives without the why. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | Rule only: API includes "GET /api/invites" as visit log. | Rule only: PLAN exposes a visit log; transparency-vs-notification rationale absent. | yes | none | Visit log is present without notification-surface rationale. |
| F39 | visitor-sees-actual-aviary | 2 | no | unreachable_excluded | none | Anchor feature not captured in PLAN. | no | unreachable | Unreachable because no show-off-mode refusal is absent. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | Per-feature: narration is "slow-cadence"; risks say queues must be throttled so users do not mute narration. | PLAN: slow-cadence naturalist updates; user events receive priority; risk says overwhelming queues force users to mute. | no | full | All three cadence layers are recovered and grounded. |

For multi-layer feature-level whys:

| Why ID | L1 / L2 / L3 recovered? | Note |
|---|---|---|
| F1 | yes / no / no | L1 recovered; shortcut-signal examples and silent failure/load-bearing consequence are absent. |
| F2 | yes / no / yes | Slow filter and failure endpoints survive; one-week/three-week calibration is absent. |
| F3 | yes / yes / no | Positive-only drift and no-Tamagotchi rationale survive; the two-week-return relationship consequence is absent. |
| F4 | yes / yes / no | Procedural calls and chorus rationale survive; audio-as-affective-spine cascade is not stated. |
| F7 | yes / no / yes | Canonical server persistence and sync consequences survive; losing-vector-as-deleting-bird does not. |
| F9 | n/a / n/a / n/a | Unreachable because return greeting is absent. |
| F10 | n/a / n/a / n/a | Unreachable because no welcome-toast/banner rule is absent. |
| F12 | yes / no / no | Naturalist prose layer survives; concentrated-voice and rare/read-only rationale are absent. |
| F13 | yes / no / no | Core precision survives; individual-signal misses and silent failure narrative are absent. |
| F14 | no / no / no | The no-streak rule is present without the counter-vs-birds rationale. |
| F15 | no / no / no | Captured feature, but B marked the rationale unrecoverable. |
| F16 | yes / no / no | Identifier rule survives; PII leak/compliance and impossible-retrofit layers are absent. |
| F17 | yes / yes / no | Tick and sync coherence survive; client-tick collapse consequence is absent. |
| F18 | yes / no / yes | Implementation rule survives; concrete cross-device data-loss example is absent. |
| F20 | yes / no / yes | Technical boundary survives; private-relationship/data-product and ML-specific rationale are absent. |
| F24 | yes / no / no | Running prose survives; same-right-to-feel and ARIA-automation failure are absent. |
| F25 | yes / no / yes | Different-rendering and not-fallback rationale survive; continuing calls/drift/notebook layer is incomplete. |
| F27 | no / no / no | The prohibition survives without the relationship rationale. |
| F30 | no / no / no | Age-based unlock survives without the anti-reward-loop why. |
| F31 | no / no / no | B marked this unrecoverable, so the why scores none. |
| F35 | n/a / n/a / n/a | Unreachable because 30-day recovery and hard-delete are absent. |
| F40 | yes / yes / yes | All three cadence layers are recovered and grounded. |

### 2.4. Evidence-bound scoring audit

Counts here treat rationale-bearing evidence as evidence. Rule-only quotes are shown in the ledgers but counted separately as `rule_without_why`.

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From benchmark constants |
| Possible total weight | 152 | Full-instance possible weight |
| Reachable gold whys | 40 | S whys always included; 31 F whys reachable |
| Excluded unreachable feature whys | 9 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 52.0 / 128 | Sum of weight x recovery-score over included whys |
| Whys with reconstruction evidence | 26 | Rationale-bearing evidence, not rule-only mentions |
| Whys with PLAN grounding | 26 | Rationale-bearing grounding, not rule-only mentions |
| `rule_without_why` cases | 14 | What survived without why |
| `plan_only_not_reconstructed` cases | 0 | No clear rationale-only-in-plan cases after evidence gate |
| `ungrounded_reconstruction` cases | 0 | No clear ungrounded reconstruction claims |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 50 | 26.0 | 52.0% |
| Affective whys | 78 | 26.0 | 33.3% |
| Weight-2 whys | 32 | 14.0 | 43.8% |
| Weight-3 whys | 96 | 38.0 | 39.6% |
| System-level whys | 28 | 14.0 | 50.0% |
| Feature-level whys (reachable) | 100 | 38.0 | 38.0% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered **26.0 / 50.0** weighted points, while affective whys recovered **26.0 / 78.0**. The plan is strongest when architecture itself carries the why: server canonical state, presence precision, aggregate telemetry, and narration queue throttling. It is weaker on relational/affective refusals like S2, F14, F23, F30, F31, F33, and F38.
- **Weight-3 vs weight-2.** Weight-3 whys recovered **38.0 / 96.0**, slightly worse than weight-2 whys at **14.0 / 32.0**. Multi-layer rows often recovered primary mechanisms but lost calibration and downstream consequence: F2 lost the one-week/three-week gap; S7/F17 lost the divergent-client failure story; F27 lost the erosion/foothold rationale.
- **System-level vs feature-level.** System-level fidelity (**50.0%**) beat feature-level fidelity (**38.0%**). The plan preserved enough cross-cutting architecture to reconstruct principles like S7/S8/S9, but feature-level rationales were compressed into labels or rules.
- **Subdomain pattern.** Accessibility performed best among feature whys, with F40 fully recovered and F24/F25 partially recovered. Accounts/sync preserved mechanisms well but often lost why layers. Social and interaction restraint had the largest affective losses.
- **Evidence-bound effects.** Fourteen reachable whys were rule-without-why cases. Under a looser v1-style scoring pass, many would look semantically close; v06 correctly denied credit where PLAN and RECONSTRUCTION only showed the operational rule.

What this suggests: the candidate planner is competent at naming system pieces and implementation constraints, but it compresses the product's relationship contract. The reconstruction then faithfully reproduces that compression instead of hallucinating missing whys, which is good validity behavior but lowers intent fidelity.

---

## 4. Recommendations for v2 hardening

- Keep the targeted headroom additions. F29-F40 exposed meaningful losses that the original feature set might not catch, especially stable identity, account export/deletion, visit-log intent, and sharing privacy.
- Preserve multi-layer scoring. It separated primary mechanism recovery from calibration and consequence recovery in F2, F17, F18, F20, S6, and S7.
- Add more small affective exception traps. This run shows that models easily state "no gamification" or "read-only" without carrying why the exception exists.
- Clarify in future schemas whether `whys_with_reconstruction_evidence` counts only rationale-bearing evidence or any rule quote. This report uses the stricter rationale-bearing interpretation and counts rule-only cases separately.
- Keep the system-level cross-cutting bar, but ask scorers to document the inherited decisions. S4/S5/S9 show that otherwise a broad principle can be over-credited from a few adjacent implementation bullets.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** From this scoring context, the safeguard held: I used only the allowlisted phase-two materials, assigned PLAN/metadata/timing, and frozen RECONSTRUCTION. I did not modify the frozen reconstruction.
- **Single-run limitation.** This is one candidate run with no variance estimate.
- **Borderline capture calls.** I leaned inclusive on several feature-capture decisions: bird naming despite missing renameability, mood persistence by server-owned mood state, client snapshot pulls without the exact visibility trigger, account export without JSON contents, no per-bird telemetry without ML wording, and visitor interaction limitations inferred from read-only/no co-presence. These raised planning quality but did not award why recovery without evidence.
- **System-level subjectivity.** S3, S4, S5, and S9 required judgment about partial cross-cutting preservation. The appendix lists the concrete plan decisions used for each.
- **Confabulation cases.** I found no clear ungrounded reconstruction claims. The reconstruction often chose NOT RECOVERABLE rather than inventing rationale.
- **Evidence-bound denials.** The main denials were rule-only: F14/F27 no gamification, F23 no leaderboards, F30 age-based bird offers, F31 stable identity, F33 notebook read-only, F34 export, F37 per-invite sharing, and F38 visit log.

---

End of report.
