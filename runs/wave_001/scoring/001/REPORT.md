# REPORT — CARE run 001

> Phase 2B scoring report for Pocket Aviary v1. Frozen reconstruction was read-only; scoring used the evidence-bound v06 rubric.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **49.2%** |
| Intent fidelity | **38.5%** |
| Combined quality | **4855** |

**Diagnostic split:**

- System-level fidelity: **57.1%**
- Feature-level fidelity: **31.6%**

**(Planning, fidelity) coordinate:** `49.2, 38.5`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-09T14:56:12Z |
| Candidate model | gemini-3.1-flash-lite-preview |
| Candidate effort | low |
| Candidate harness | gemini-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **59 / 120** = **49.2%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 3 | 50.0% |
| concepts.md | 4 | 2 | 50.0% |
| bird_engine.md | 22 | 14 | 63.6% |
| interactions.md | 20 | 7 | 35.0% |
| aviary_layout.md | 18 | 3 | 16.7% |
| accounts_sync.md | 18 | 9 | 50.0% |
| social_optional.md | 10 | 6 | 60.0% |
| accessibility_perf.md | 18 | 11 | 61.1% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **59** | **49.2%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured by scope: browser-based, single-user virtual aviary. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | no | Procedural mechanisms appear, but the design philosophy itself is not stated. |
| 3 | "Notice, never announce" principle callout | product_brief.md | no | No notice-vs-announce principle or welcome-surface rule. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | no | Naturalist prose appears, but the matter-of-fact system exception is missing. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Strict non-goals name native apps, gamification, Tamagotchi mechanics, public social discovery, chat, and comments. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Two initial birds and max seven are explicit. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | Terms are used, but no glossary/definition surface is planned. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Presence is visibilityState + focus + activity and feeds drift. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Personality vector and fast-timescale mood are separated. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | no | Settle appears only as an event name. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Vector traits are enumerated. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | no | borderline/thin: A drift function is named, but low-pass behavior and calibration are absent. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | no | No one-week/three-week calibration target. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Monotonic expressive and no penalty for neglect are explicit. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | borderline/thin: Mood is an enumerated fast-timescale state, though reset cadence is not specified. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | no | Mood advances, but inputs are not specified. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Client-side motif-based call grammar is explicit. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | no | Species motifs and traits appear, but not per-bird recognizable signatures. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Real-time mixing and no baked loops are explicit. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Call grammar is shaped by personality traits and vocal frequency is named. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Preening and head-tilting are named. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Mood-shaped idle motion is explicit. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | no | Species exists as a field, but no v1 pool size. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | borderline/thin: User-assigned name is explicit; renameability is absent but core naming is present. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | no | Two initial birds are present, but auto-selection/adoption flow is not. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Max seven is explicit. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Paced bird availability based on aviary age is explicit. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Persistent DB and server-side canonical state cover vector persistence. |
| 29 | Mood persistence across sessions | bird_engine.md | no | Moods are in state snapshots, but no no-reset/persistence rule. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | no | Chorus exists, but bird-to-bird reactions are not planned. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Stable UUID is explicit. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Personality vector is marked hidden. |
| 33 | Return-greeting on viewer arrival | interactions.md | no | No return greeting feature. |
| 34 | Greeting variation by absence length | interactions.md | no | No greeting feature or absence-length variation. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | No greeting feature. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | no | No greeting feature. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No explicit welcome-toast/banner refusal. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Listen-in event and rebalance are explicit. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Listen-in rebalance says not mute. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | no | Offer is only an event name; no interaction affordances are specified. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | No offer reaction behavior. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | no | No cooldown. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | no | borderline/thin: Settle is only an event name; gesture and lighting are absent. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | No close-tab equivalence or opt-in settle rule. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Append-only auto-generated naturalist observations are explicit. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | no | No frequency/rarity rule. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Append-only auto-generated observations imply no user edits. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Presence-event log and presence-time accumulator are explicit. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | VisibilityState + focus + activity are explicit. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | borderline/thin: Streaks are explicitly excluded under gamification; days-visited is implicit. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | Server sim and visibility are present, but client pause is not. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | No undo window. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Responsive horizontal layout with no scroll/pan is explicit. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Three perch zones are explicit. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | no | No perch choice/placement rule. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Day/night local-timezone anchoring is explicit. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | no | No evening palette/call quieting. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | No night-state behavior. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | no | No weather system. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | no | No weather/mood coupling. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | no | Ambient drift appears only as something removed in reduced motion. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | no | No parallax. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | no | No UI chrome/top-bar placement rule. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | no | No top bar contents. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | no | No auto-fade behavior. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | no | No already-in-progress first-frame rule. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | no | No loading-state rule. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-aviary state. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | No palette spec. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | no | borderline/thin: Responsive layout appears, but the no-crop rule is absent. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Magic-link authentication is explicit. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | No expiry. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Single-user virtual aviary and one canonical state imply this. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | no | No account ID privacy rule. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Slow server-side tick is explicit. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | borderline/thin: GET /state pulls a canonical state snapshot; visibility trigger is absent. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Client pulls snapshots and interpolates. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Multi-device sync through canonical server state is explicit. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Prevents last-write-wins and clients never write personality/drift directly. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Server tick processes append-only event log; clients never write personality/drift directly. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | no | No sync conflict UI/tone rule. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | no | No per-device token. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | no | No account export. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | no | No account deletion. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | no | Telemetry is operational/anonymized only, but no per-bird ML prohibition. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | borderline/thin: Operational latency/errors plus anonymized load metrics capture aggregate telemetry, though the boundary is thin. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | No email-change flow. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | borderline/thin: Opt-in visitor links and GET /invites are explicit; email naming is absent. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Opt-in visitor links imply visits are off until invited. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Read-only visitor links are explicit. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Read-only visitor links exclude visitor interactions. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | borderline/thin: Chat and comments are explicit non-goals; avatars are implicit under social-surface refusal. |
| 94 | No "your friend visited!" notification by default | social_optional.md | no | No visit-notification rule. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | no | No revocation. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | no | No visit log. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | no | borderline/thin: Read-only visits are present, but actual-state/no-show-off is absent. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Public social discovery is explicitly out of scope. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Naturalist, present-tense screen-reader prose is explicit. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | no | No cadence. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Naturalist, present-tense prose is explicit. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Cross-fades and no ambient drift are explicit. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Designed alternate render mode captures non-stripped intent. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | borderline/thin: Call captioning and prose descriptions are explicit, though toggle is absent. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | no | No contrast target. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | no | No keyboard navigation. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | no | No focus indicator rule. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Bundle <2MB gzipped is explicit. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | First bird <500ms on mid-tier mobile/4G is explicit. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | borderline/thin: 60fps idle motion is explicit; device age is absent. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | No memory growth over 30min is explicit. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Client-side WebAudio procedural calls and no baked loops are explicit. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | no | No WebAudio fallback. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Operational latency/error telemetry and anonymized load metrics capture observability, but not synthetic/RUM specifics. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | no | No simulation-tick error budget. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | No browser matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Native apps are explicitly out of scope. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Gamification, streaks, badges, and achievements are explicitly out of scope. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Tamagotchi mechanics, hunger, and death are explicitly out of scope. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Public social discovery, chat, and comments are out of scope. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **57.1%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 — feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md §System-level intent: "Procedural life rather than baked content"; "Procedural rendering for motion"; "WebAudio for procedural calls"; "No baked loops." | PLAN.md §Architecture/Rendering/Audio/Performance: "Procedural rendering for motion"; "No baked loops"; "Micro-motion"; "First bird <500ms". | yes | yes | no | partial | Procedural/non-baked aliveness survived, but the continuing-place and spinner/canned-staleness consequences were not explicit. |
| S2 — notice-never-announce | 4 | included | none | none | no | no | no | none | No notice-vs-announce principle, no return greeting, and no toast/banner refusal. |
| S3 — charm-from-specificity | 2 | included | RECONSTRUCTION.md §System-level intent: "Naturalist prose is part of the product voice." | PLAN.md §Data Model/Accessibility/Captions: "auto-generated naturalist observations"; "Naturalist, present-tense prose"; "Prose descriptions". | yes | yes | no | partial | Naturalist specificity partly survived, but the anti-generic/anti-stat-management rationale was compressed away. |
| S4 — restraint-over-richness | 2 | included | RECONSTRUCTION.md §System-level intent: "Bounded, calm, single-user web product" and non-goals excluding social/game expansion. | PLAN.md §Scope/Rendering: "Two initial birds per aviary (max seven)"; "Responsive horizontal layout"; "No scroll/pan"; strict non-goals. | yes | yes | no | full | The bounded/calm restraint principle is visible across scope, layout, and non-goals. |
| S5 — naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md §System-level intent: "Naturalist prose is part of the product voice." | PLAN.md §Data Model/Accessibility: "naturalist observations"; "Naturalist, present-tense prose"; "Prose descriptions of procedural calls." | yes | no | no | partial | The naturalist voice survived; the matter-of-fact system/error exception did not. |
| S6 — presence-is-real-interaction | 4 | included | RECONSTRUCTION.md §System-level intent: "Presence and interactions should move birds toward expression without punishment" and "Presence-logic integrity (ensuring attention-metric honesty)." | PLAN.md §Data Model/Simulation/Risks: "visibilityState + focus + activity"; "presence-time (primary)"; "Asymmetric (no penalty for neglect)". | yes | yes | no | partial | Presence as attention input survived; population-inflation precision and settle-vs-close equivalence did not. |
| S7 — simulation-runs-server-side | 4 | included | RECONSTRUCTION.md §System-level intent: "Canonical server state over client autonomy" and "Sync correctness and drift preservation matter." | PLAN.md §Architecture/Sync: "server is the canonical state holder and simulation engine"; "Clients never write personality/drift directly"; "prevents last-write-wins." | yes | yes | no | full | Server-canonical tick, multi-device coherence, and no-lost-drift consequences were all recovered. |
| S8 — privacy-first-on-bird-data | 2 | included | none | none | no | no | no | none | No per-bird relationship-data privacy boundary or pipeline separation is present. |
| S9 — accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md §System-level intent: "Accessibility is designed as a first-class alternate experience" and reduced motion is a "Designed alternate render mode." | PLAN.md §Scope/Accessibility: "screen-reader narration"; "reduced-motion mode"; "call captioning"; "Designed alternate render mode". | yes | yes | no | full | The reconstruction preserved accessibility as designed experience, not mere parity, and the features are in v1 scope. |

Multi-layer system-level whys:

| Why ID | L1 primary | L2 secondary | L3 downstream |
|---|---|---|---|
| S1 | yes | yes | no |
| S2 | no | no | no |
| S6 | yes | no | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: procedural rendering/no baked loops; mood-shaped idle motion; WebAudio motif calls; first-bird performance target. Count >= 3; (b) yes, but consequence layers thin.
- S2: no return greeting or announcement-surface rules; gamification/social non-goals are adjacent but not this principle. Count < 3; (b) no.
- S3: naturalist notebook, screen-reader prose, captions, user-assigned names. Count >= 3, but only the specificity half is visible.
- S4: two initial birds/max seven, single horizontal no-pan scene, strict non-goals, web-only scope. Count >= 3; (b) yes.
- S5: naturalist notebook/narration/captions; no matter-of-fact system surfaces. Count < 3 for the full split; (b) no.
- S6: presence log conjunction, presence-time primary drift input, no penalty for neglect, Tamagotchi non-goal. Count >= 3; (b) yes, but missing settle/tab-close equivalence.
- S7: server-side tick, canonical state snapshots, clients never write drift, append-only event log preventing last-write-wins. Count >= 3; (b) yes.
- S8: only anonymized operational telemetry is present; no per-bird database/telemetry separation. Count < 3; (b) no.
- S9: screen-reader prose, reduced-motion alternate rendering, call captions, v1 scope accessibility features. Count >= 3; (b) yes.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **31.6%**.

Reachable feature-level whys: **24 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md §Per-feature whys: "Presence accounting" uses "presence-time (primary)" and "attention-metric honesty." | PLAN.md §Data Model/Simulation: "visibilityState + focus + activity" and "presence-time (primary)." | no | partial | Only the primary presence-as-drift-input layer survived; laxer-tab-open failure modes did not. |
| F2 | drift-function | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured because low-pass behavior and calibration were absent. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md §System-level intent/Per-feature whys: "without punishment" and "no penalty for neglect." | PLAN.md §Simulation Engine: "Monotonic toward expressive" and "Asymmetric (no penalty for neglect)." | no | partial | No-punishment survived, but the two-week return/ambient-not-mistrust consequence was absent. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md §System-level intent/Audio: "Procedural life rather than baked content"; "No baked loops"; "Motif libraries per species." | PLAN.md §Architecture/Audio: "WebAudio for procedural calls"; "No baked loops"; "Motif libraries per species." | no | partial | Procedural calls and real-time mixing survived; the audio-spine/fallback/bundle consequence did not. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md §Rendering Pipeline: "Mood-shaped idle motion" such as "preening" and "head-tilting" to "express moods." | PLAN.md §Rendering Pipeline: "Mood-shaped idle motion (preening, head-tilting)." | yes | none | Mechanism survived, but not the no-label affective rationale. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md §Per-feature whys: "Two initial birds per aviary (max seven): NOT RECOVERABLE FROM PLAN." | PLAN.md §Scope: "Two initial birds per aviary (max seven)." | yes | none | The cap appears, but the empirical recognizability rationale was explicitly not recovered. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md §System/Sync: "canonical state"; "Clients never write personality/drift directly"; persistent "account/bird state." | PLAN.md §Architecture/Data/Sync: persistent DB for "account/bird state"; "Server-side only" canonical state. | no | partial | Canonical server persistence and sync consequences survived; losing-vector-as-deleting-bird did not. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md §Data Model: "Hidden personality vector on birds: NOT RECOVERABLE FROM PLAN." | PLAN.md §Data Model: "personality vector (hidden)." | yes | none | Hidden vector rule survived, but the stat-management rationale did not. |
| F9 | return-greeting | 4 | no | unreachable_excluded | none | none | no | unreachable | Return greeting was not captured. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | No welcome-toast/banner refusal was not captured. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Settle opt-in/close-tab equivalence was not captured. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md §Scope/Data Model: Field notebook uses "append-only auto-generated naturalist observations" and matches "naturalist prose product voice." | PLAN.md §Data Model: "Append-only auto-generated naturalist observations." | no | partial | Naturalist observer prose survived; rare cadence/feed refusal did not. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md §Per-feature whys: presence feeds drift via "presence-time (primary)" and protects "attention-metric honesty." | PLAN.md §Data Model/Simulation/Risks: "visibilityState + focus + activity"; "presence-time (primary)"; "attention-metric honesty." | no | partial | The conjunction/input layer survived; individual-signal misses and silent population corruption were absent. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md §System-level intent: non-goals exclude "gamification." | PLAN.md §Scope: non-goals exclude "gamification (streaks, badges, achievements)." | yes | none | The no-streak rule is present, but not the presence-for-birds rationale. |
| F15 | scene-loads-with-motion | 4 | no | unreachable_excluded | none | none | no | unreachable | Already-in-progress first-frame behavior was not captured. |
| F16 | synthetic-account-id | 4 | no | unreachable_excluded | none | none | no | unreachable | Synthetic account ID was not captured. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md §System/Simulation: server is "canonical state holder and simulation engine"; tick "updates personality vectors" and "advances moods." | PLAN.md §Scope/Simulation/Sync: "Server-side simulation with a slow (~1min) tick" and clients pull snapshots. | no | partial | Server tick and coherence survived; client-divergence collapse was only implicit. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md §System/Sync: "avoid last-write-wins"; "no lost drift data"; "Append-only event log processed sequentially by server tick." | PLAN.md §Sync Model: "Clients never write personality/drift directly"; append-only event log prevents last-write-wins. | no | full | The no-LWW implementation and lost-drift rationale were recovered. |
| F19 | sync-conflict-tone | 2 | no | unreachable_excluded | none | none | no | unreachable | Sync conflict tone was not captured. |
| F20 | no-per-bird-ml-telemetry | 4 | no | unreachable_excluded | none | none | no | unreachable | Per-bird ML telemetry prohibition was not captured. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md §Scope: "opt-in read-only visitor links." | PLAN.md §Scope/API: "opt-in read-only visitor links" and GET /invites. | yes | none | Read-only visits survived, but the observation-not-co-presence/drift rationale did not. |
| F22 | no-friend-visited-notification | 2 | no | unreachable_excluded | none | none | no | unreachable | No friend-visited notification was not captured. |
| F23 | no-leaderboards | 2 | yes | included | RECONSTRUCTION.md §System-level intent: non-goals exclude "public social discovery." | PLAN.md §Scope: strict non-goals include "public social discovery." | yes | none | Public discovery refusal survived, but not the comparison/private-birds rationale. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md §Scope/Accessibility: screen-reader narration uses "Naturalist, present-tense prose." | PLAN.md §Accessibility: "Naturalist, present-tense prose via screen reader." | no | partial | Running naturalist prose survived; same-right-to-feel and anti-ARIA rationale did not. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md §Accessibility: "Designed alternate render mode" using "cross-fades" and "no ambient drift." | PLAN.md §Accessibility: "Designed alternate render mode (cross-fades, no ambient drift)." | no | partial | Alternate rendering survived; full same-aviary details such as calls/notebook persistence were absent. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md §Performance: "First bird <500ms" grounded in "mid-tier mobile/4G." | PLAN.md §Performance: "First bird <500ms (mid-tier mobile/4G)." | yes | none | Metric survived, but not the affective-perf threshold rationale. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md §System-level intent: non-goals exclude "gamification." | PLAN.md §Scope: strict non-goals include "gamification (streaks, badges, achievements)." | yes | none | The ban survived, but not the foothold/relationship-rotation rationale. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md §System-level intent: presence/interactions move birds toward expression "without punishment"; non-goals exclude "Tamagotchi mechanics." | PLAN.md §Scope/Simulation: "Tamagotchi mechanics (hunger, death)" out of scope and "no penalty for neglect." | no | full | The punishment-of-absence rationale is directly preserved. |
| F29 | starter-birds-not-catalog | 2 | no | unreachable_excluded | none | none | no | unreachable | Starter birds as arrivals/not catalog choices was not captured. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md §Rollout: "Paced bird availability based on aviary age: NOT RECOVERABLE FROM PLAN." | PLAN.md §Rollout: "Paced bird availability based on aviary age." | yes | none | Age-based unlock rule survived, but the anti-reward-loop why was explicitly not recovered. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md §Data Model: "Birds as persistent records with stable UUID" and "stable bird identity." | PLAN.md §Data Model: "Persistent records with stable UUID." | yes | none | Stable ID survived as mechanism, but relationship-continuity rationale did not. |
| F32 | mood-persists-across-sessions | 2 | no | unreachable_excluded | none | none | no | unreachable | Mood no-reset persistence was not captured. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md §Data Model: "Notebook as append-only observations." | PLAN.md §Data Model: "Append-only auto-generated naturalist observations." | yes | none | Append-only/read-only survived, but not observer-record-not-journal rationale. |
| F34 | account-export-relationship-copy | 2 | no | unreachable_excluded | none | none | no | unreachable | Account export was not captured. |
| F35 | account-deletion-grace-then-hard-delete | 4 | no | unreachable_excluded | none | none | no | unreachable | Account deletion was not captured. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md §Rollout: telemetry gives operational visibility into "latency" and "errors" plus "anonymized load metrics." | PLAN.md §Rollout: "Operational (latency, errors) + anonymized load metrics." | yes | none | Operational telemetry survived, but the technical boundary against relationship-data reconstruction did not. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md §System-level intent: sharing is "controlled, not social discovery" and uses "opt-in read-only visitor links." | PLAN.md §Scope/API: "opt-in read-only visitor links" and GET /invites. | yes | none | Opt-in sharing survived, but named private-relationship sharing rationale did not. |
| F38 | visit-log-on-demand-transparency | 2 | no | unreachable_excluded | none | none | no | unreachable | Visit log was not captured. |
| F39 | visitor-sees-actual-aviary | 2 | no | unreachable_excluded | none | none | no | unreachable | No show-off mode/actual aviary view was not captured. |
| F40 | narration-cadence-slow | 4 | no | unreachable_excluded | none | none | no | unreachable | Slow narration cadence was not captured. |

Multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | no | no |
| F2 | no | no | no |
| F3 | yes | yes | no |
| F4 | yes | yes | no |
| F7 | yes | no | yes |
| F9 | no | no | no |
| F10 | no | no | no |
| F12 | yes | yes | no |
| F13 | yes | no | no |
| F14 | no | no | no |
| F15 | no | no | no |
| F16 | no | no | no |
| F17 | yes | yes | no |
| F18 | yes | yes | yes |
| F20 | no | no | no |
| F24 | yes | no | no |
| F25 | yes | no | yes |
| F27 | no | no | no |
| F30 | no | no | no |
| F31 | no | no | no |
| F35 | no | no | no |
| F40 | no | no | no |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS.json |
| Possible total weight | 152 | Fixed total possible weight |
| Reachable gold whys | 33 | System whys plus captured feature whys |
| Excluded unreachable feature whys | 16 | Feature anchors not captured |
| Recovered / reachable weight | 40 / 104 | Weighted numerator and denominator |
| Whys with reconstruction evidence | 31 | Rows with exact frozen-reconstruction evidence or explicit non-recovery quote |
| Whys with PLAN grounding | 31 | Rows with exact plan grounding |
| rule_without_why cases | 13 | Mechanism survived without rationale |
| plan_only_not_reconstructed cases | 0 | None counted |
| ungrounded_reconstruction cases | 0 | None counted |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 36 | 18 | 50.0% |
| Affective whys | 68 | 22 | 32.4% |
| Weight-2 whys | 28 | 6 | 21.4% |
| Weight-3 whys | 76 | 34 | 44.7% |
| System-level whys | 28 | 16 | 57.1% |
| Feature-level whys (reachable) | 76 | 24 | 31.6% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered **18 / 36** weighted points, while affective whys recovered **22 / 68**. Functional recovery was helped by S7/F18; affective recovery often collapsed to rule-only non-goals.
- **Weight-3 vs weight-2.** Weight-3 whys recovered **34 / 76**, better than weight-2 at **6 / 28**, but most weight-3 recoveries were partial: the first layer survived more often than calibration or downstream consequences.
- **System-level vs feature-level.** System-level fidelity (**57.1%**) was much stronger than feature-level fidelity (**31.6%**). The plan preserved broad architecture and accessibility principles better than per-feature rationale.
- **Subdomain patterns.** Sync/server architecture was the strongest area. Social, notification, visit-log, account privacy, and subtle affective layout rules were the weakest.
- **Evidence-bound effects.** F6, F8, F14, F21, F23, F26, F27, F30, F31, F33, F36, and F37 are representative rule-without-why cases.

The failure shape suggests a compact low-effort plan that preserved many product nouns and implementation choices but rarely carried the reasons those choices exist.

---

## 4. Recommendations for v2 hardening

- Keep the targeted F29-F40 headroom whys. This run captured some of them as mechanisms, especially F30, F31, F33, F36, and F37, while dropping the rationale.
- Add more exception-style affective whys around social, announcements, and engagement loops. Broad non-goal language is easy to plan; the relationship-protection rationale is harder.
- Preserve the evidence-bound operator. It prevented metric/rule mentions such as F26 and F36 from scoring as intent recovery without rationale evidence.
- Consider reporting layer-loss diagnostics automatically. In this run, downstream-consequence layers were the most commonly dropped layer.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** The frozen reconstruction showed no gold-ID leakage and primarily mirrors PLAN headings, so phase separation appears to hold.
- **Single-run limitation.** This is one sampled run; no variance signal is available inside this score.
- **Borderline capture calls.** Thin inclusive calls include feature 50 (streaks vs days-visited), 76 (snapshot on visibility), 86 (aggregate telemetry boundary), 89/90 (opt-in visits), 103/104 (accessibility details), and 114 (observability specifics). These increased planning quality, but fidelity still required evidence.
- **System-level cross-cutting.** S3/S5 were partial because naturalist prose survived without the broader specificity/system-exception principle. S4/S9 were full because the plan carried them across multiple decisions.
- **Confabulation cases.** No ungrounded reconstruction cases were counted. The reconstruction was often sparse, but it generally stayed inside the plan.
- **Rule-without-why cases.** Thirteen included whys preserved mechanisms without the gold rationale.

---

End of report.
