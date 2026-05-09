# REPORT - CARE run 001

> Phase 2B scoring artifact for Pocket Aviary. Frozen reconstruction was scored without editing it. Variant: v06 evidence-bound clean + targeted gold headroom.

---

## 1. Headline

| Score | Value |
| --- | --- |
| Planning quality | 64.2% |
| Intent fidelity | 37.9% |
| Combined quality | 6355 |

**Diagnostic split:**

- System-level fidelity: **53.6%**
- Feature-level fidelity: **33.3%**

**(Planning, fidelity) coordinate:** `(64.2, 37.9)`.

### Run metadata

| Field | Value |
| --- | --- |
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-09T14:07:10Z |
| Candidate model | gemini-3.1-pro-preview |
| Candidate effort | low |
| Candidate harness | gemini-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **77 / 120** = **64.2%**.

| File | Total | Captured | Rate |
| --- | --- | --- | --- |
| product_brief.md | 6 | 4 | 66.7% |
| concepts.md | 4 | 1 | 25.0% |
| bird_engine.md | 22 | 19 | 86.4% |
| interactions.md | 20 | 8 | 40.0% |
| aviary_layout.md | 18 | 6 | 33.3% |
| accounts_sync.md | 18 | 11 | 61.1% |
| social_optional.md | 10 | 7 | 70.0% |
| accessibility_perf.md | 18 | 17 | 94.4% |
| non_goals.md | 4 | 4 | 100.0% |
| Total | 120 | 77 | 64.2% |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
| --- | --- | --- | --- | --- |
| 1 | Headline product concept statement | product_brief.md | yes | borderline: Title plus scope define a web aviary with birds, accounts, interactions, accessibility, and non-goals. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | borderline: Captured through server tick, continuous idle motion, procedural audio, and performance budgets, though not named as a philosophy. |
| 3 | "Notice, never announce" principle callout | product_brief.md | no | No explicit notice-vs-announce principle or announcement-surface rule. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | no | Naturalist voice appears, but the matter-of-fact system/error exception is absent. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Non-goals exclude gamification, Tamagotchi mechanics, native mobile, and social-network surfaces. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Scope states two starters and maximum seven birds. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No glossary or domain-term definitions. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | no | Presence is used, but not precisely defined as visibility plus focus plus recent pointer/key activity. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Scope and data model separate personality vectors from fast-timescale mood states. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | no | Settle is listed as an interaction but not defined. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Bird data model enumerates the five vector fields. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Simulation design specifies a slow low-pass filter over presence, listen-ins, and offers. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | no | No one-week instrument / three-week user calibration target. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Deltas are always positive and non-goals forbid negative drift on neglect. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | borderline: Fast-timescale mood exists, but daily-ish reset cadence is not specified. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Mood transitions use recent events, local time, weather, and personality. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | WebAudio builds calls from small motifs with pitch and timing variation. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | borderline: Vocal-frequency and mood shape calls, but recognizability by ear is implicit rather than stated. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Chorus mixing avoids phase-canceling artifacts through procedural variance. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Audio construction is influenced by vocal_frequency and current_mood. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Frontend includes continuously running idle micro-motion. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Idle animations are mood-shaped. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Scope specifies a fixed pool of roughly six species. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Bird model includes user_assigned_name. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | borderline: Two starter birds are specified; auto-selection/catalog refusal is implicit. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Scope specifies maximum seven birds. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Ramping Birds offers additional birds by account age, not grinding. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Server owns canonical state and persists Bird.personality_vector. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Bird.current_mood is persisted as canonical server state. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | no | Chorus exists, but bird-to-bird reactions are not specified. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Bird has a stable id separate from name/species in canonical storage. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | no | No UI rule forbids numeric trait exposure. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Return-greeting is in core interactions. |
| 34 | Greeting variation by absence length | interactions.md | no | No absence-length greeting variation. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | No boldness-based greeting order. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | no | No greeting stagger. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No textual welcome/toast prohibition. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Listen-in smoothly ramps the focused bird gain. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Other birds/ambient mix drop slowly but are not fully muted. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Offer is a core interaction and API event. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | Offers contribute events, but reaction variation is not specified. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | no | No offer cooldown. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | no | Settle is named but not described as a gesture or lighting shift. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | No close-tab equivalence or opt-in rationale. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Auto-generated field notebook in naturalist voice is in scope. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | no | No rarity/frequency rule. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | borderline: Only auto-generation and GET notebook are specified; no write/edit surface appears. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Presence pings and presence time feed drift. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | no | No visibility/focus/recent-input conjunction. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Non-goals forbid scores, streaks, achievements, and counters. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | Server sim continues, but background-tab rendering pause is not specified. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | No settle undo window. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Single horizontally responsive viewport with no scrolling. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Three z-depth planes and Bird.perch_zone are specified. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | no | No bird-choice or no-user-placement rule. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Scope includes day/night cycle matching local timezone. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | no | No evening palette or quieter-call rule. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | No night-state behavior. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Aviary.current_weather and mood inputs include ambient weather. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Mood transitions include ambient weather. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | no | No ambient leaf/feather drift. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Scene composition includes foreground parallax elements. |
| 63 | No UI chrome inside aviary view (icons live in a thin top bar) | aviary_layout.md | no | No top-bar/no-chrome layout rule. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | no | No top bar content specification. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | no | No auto-fade top bar. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | no | No first-frame-mid-motion/no-entry-animation rule. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | no | No loading-state design. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-aviary state. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | No palette specification. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | no | Responsive scene yes, no no-cropping requirement. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Magic-link email sign-in and auth endpoints are specified. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | No magic-link expiry duration. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Single-user accounts and one canonical aviary per account are in scope. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Synthetic UUID and encrypted isolated email are specified. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Server-side tick runs about once per minute. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Clients fetch latest snapshot on load and visibility change. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Client interpolates bird positions between server snapshots. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Implicit sync through server-side canonical database. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Client never submits absolute values; no last-write-wins race conditions. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Server simulation tick is sole writer of drift/personality changes. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | no | No sync-conflict or error-copy tone. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | no | Session token has device info, but no revocation flow. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | no | No account export. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | no | No deletion flow. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | borderline: Telemetry excludes per-account interaction history/personality vectors, though ML is not named. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Aggregate telemetry and privacy boundary are specified. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | No email change flow. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Visit invite model stores visitor identity and token; visits are opt-in. |
| 90 | Visits default OFF for new accounts | social_optional.md | no | No default-off visit setting. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Visitors fetch a read-only ambient state snapshot. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Read-only ambient snapshot implies no visitor interactions. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Social-network surfaces, profiles, co-presence, and chat are out of scope. |
| 94 | No "your friend visited!" notification by default | social_optional.md | no | No friend-visit notification rule. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Read-only visit invitations are revocable. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | no | No visit log. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes | Visitor endpoint fetches an ambient state snapshot rather than a separate show-off mode. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Social non-goals exclude discovery feed, leaderboards, and profiles. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Screen-reader naturalist prose narration is specified. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | ARIA live region updates every 30-60 seconds with priority bumps. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Narration uses naturalist prose examples. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Reduced-motion replaces motion and paths with slow cross-fades. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | First-class accessibility and graceful cross-fades preserve the aviary experience. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Opt-in text overlays describe procedural audio near calling birds. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | WCAG AA contrast is required. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Top bar, birds, and listen-in are keyboard-operable. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | High-contrast focus rings are specified. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Initial JS bundle budget is <2MB gzipped. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | TTFB budget is <500ms on mid-tier mobile/4G. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | 60fps rendering on older machines is specified. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | No memory leaks over 30+ minute sessions. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | WebAudio client-side procedural calls, no static loops. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Fallback is silence with call captions enabled by default. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Synthetic checks and aggregate telemetry are in rollout/observability. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | p99 tick latency >5 seconds triggers a critical alert. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | No browser support matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Native mobile apps are out of scope. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Gamification non-goals are explicit. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Tamagotchi mechanics are explicit non-goals. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Social network surfaces are explicit non-goals. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **53.6%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) identified by B? | (b) cross-cutting in PLAN? | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| S1 - feels-alive-not-robotic | 4 | included | System-level: "simulation tick advances the aviary regardless"; "No static audio loops"; "dynamically constructed at runtime". | PLAN Client/Server: "Server: Sole owner of canonical state"; Audio: "No static audio loops"; Rendering: "Idle Micro-Motion". | yes | yes | no | partial | Reconstruction recovers continuing simulation and procedural variation, but not the staleness/leak consequence. |
| S2 - notice-never-announce | 4 | included | none | none | no | no | yes | none | Plan/reconstruction reject gamification, but do not recover noticed-vs-announced or welcome/notification surface logic. |
| S3 - charm-from-specificity | 2 | included | System-level: "Naturalist prose is a primary product voice" and notebook entries carry "naturalist voice". | PLAN Scope: "Auto-generated field notebook in a naturalist voice"; Accessibility: "screen-reader naturalist prose narration". | yes | no | no | partial | Naturalist voice survives, but specificity-not-generic-state-management is not cross-cuttingly named. |
| S4 - restraint-over-richness | 2 | included | none | PLAN Scope: "Up to seven birds"; Rendering: "Single viewport"; Audio: listen-in lowers others "but not full mute". | no | yes | no | partial | Plan preserves several restraint rules, but the reconstruction does not articulate restraint-over-richness as a why. |
| S5 - naturalist-voice-with-system-exception | 2 | included | System-level: "Naturalist prose is a primary product voice". | PLAN Scope: field notebook naturalist voice; Accessibility: screen-reader naturalist prose and captions. | yes | no | no | partial | Naturalist voice is present; matter-of-fact system/error exception is missing. |
| S6 - presence-is-real-interaction | 4 | included | System-level: "Presence should change the birds slowly, positively, and expressively"; per-feature: presence is "dominant" input. | PLAN Simulation: "presence time (dominant)"; "Deltas are always positive"; Risks: too fast "Tamagotchi". | yes | yes | no | partial | Idle attention as drift input survives; precise presence definition and settle/tab-close equivalence do not. |
| S7 - simulation-runs-server-side | 4 | included | System-level: "One canonical, server-owned aviary rather than local device state" and "No Client-Side State Ownership". | PLAN Sync: clients read "same server-side canonical database"; Server is "Sole owner of canonical state". | yes | yes | no | partial | Server-owned simulation and coherent sync survive; detailed divergent-client/LWW failure mode is compressed away. |
| S8 - privacy-first-on-bird-data | 2 | included | System-level: "Privacy-bounded sharing and measurement"; telemetry never includes "per-account interaction history" or "personality vectors". | PLAN Observability: "Telemetry never includes per-account interaction history, personality vectors, or PII". | yes | yes | no | full | The privacy boundary is explicitly reconstructed and grounded in telemetry and identity design. |
| S9 - accessibility-as-first-class-surface | 4 | included | System-level: "Accessibility is first-class and tested as core behavior"; per-feature surfaces "preserve the aviary experience". | PLAN Scope: "First-class accessibility"; Risks: accessibility surfaces in "core automated testing suite". | yes | yes | no | full | Reconstruction preserves parity of charm, designed alternate surfaces, and first-release/tested treatment. |

Multi-layer system why recovery:

| Why ID | L1 primary | L2 secondary | L3 downstream |
| --- | --- | --- | --- |
| S1 | yes | yes | no |
| S2 | no | no | no |
| S6 | yes | no | no |
| S7 | yes | yes | no |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: server-owned canonical state; continuous idle motion; procedural/no-loop audio; reduced-motion designed surface; performance budgets.
- S2: only partial rule inheritance via no gamification and age-based unlocks; no welcome/notification principle. Count below bar for the actual gold principle.
- S3: naturalist field notebook, screen-reader prose, captions, and names support specificity, but the plan does not name the specificity-vs-generic-state rationale.
- S4: two starters, max seven, single viewport, listen-in mix restraint, and web-only scope preserve restraint rules.
- S5: naturalist notebook/SR/caption voice appears; matter-of-fact system/error exception is absent.
- S6: presence pings, presence-dominant drift, positive monotonic drift, and no Tamagotchi mechanics inherit part of the principle; precise presence and settle equivalence are missing.
- S7: server tick, canonical state, client view-only boundary, append-only events, and no last-write-wins all inherit the principle.
- S8: synthetic UUIDs, encrypted email isolation, aggregate telemetry, privacy boundary, and opt-in read-only sharing inherit the principle.
- S9: screen-reader prose, reduced-motion cross-fades, captions, keyboard/focus/contrast, and regression tests inherit the principle.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **33.3%**.

Reachable feature-level whys: **30 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1 | presence-definition | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured, so this why is excluded from the fidelity denominator. |
| F2 | drift-function | 4 | yes | included | Reconstruction Drift Function: "slow low-pass filter" and too fast feels "like a Tamagotchi". | PLAN Simulation: "A slow low-pass filter"; Risks: too fast "Tamagotchi" / too slow "unresponsive". | no | partial | L1 and L3 survive; one-week instrument / three-week visible calibration is absent. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | Reconstruction: deltas are "always positive (monotonic toward expressive)" and Tamagotchi is a failure mode. | PLAN Simulation: "Deltas are always positive"; Non-goals: "no negative drift on neglect". | no | partial | Positive/non-punitive drift survives; the two-week return / not mistrust consequence is absent. |
| F4 | procedural-call-grammar | 4 | yes | included | Reconstruction: "No static audio loops"; calls "dynamically constructed"; procedural variance prevents "phase-canceling artifacts". | PLAN Audio: "No static audio loops"; motifs "dynamically constructed"; fallback to silence/captions. | no | full | All layers are grounded: runtime synthesis, chorus variance, and WebAudio/fallback cascade. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | Reconstruction: "mood-shaped" idle motion so "visible behavior reflects current mood". | none | no | none | Reconstruction asserts the visible-mood rationale, but PLAN only states the rule; confabulation guard denies credit. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | Cap rule appears, but empirical recognizability rationale is absent and B says not recoverable. |
| F7 | personality-vector-persistence | 4 | yes | included | Reconstruction: server owns canonical state; Bird data carries "personality vector" and client is view layer only. | PLAN Data Model: Bird.personality_vector; Client/Server: server "Sole owner of canonical state". | no | partial | Canonical server persistence and downstream no-client-ownership survive; affective identity-deletion rationale does not. |
| F8 | vector-never-shown-numerically | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured, so this why is excluded from the fidelity denominator. |
| F9 | return-greeting | 4 | yes | included | none | none | yes | none | Return-greeting is captured only as a named interaction; B marks the why not recoverable. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured, so this why is excluded from the fidelity denominator. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured, so this why is excluded from the fidelity denominator. |
| F12 | field-notebook-prose | 4 | yes | included | Reconstruction: notebook entries are "server-written `prose_text`" carrying "naturalist voice" rather than score/counter surface. | PLAN Scope: "Auto-generated field notebook in a naturalist voice"; Data Model: Notebook Entry `prose_text`. | no | partial | Naturalist prose layer survives; event-log contrast, rare cadence, and read-only rationale are missing. |
| F13 | presence-accounting | 4 | yes | included | none | none | yes | none | Presence pings and dominance are present, but the precise three-signal accounting rationale is absent. |
| F14 | no-streak-counter | 4 | yes | included | Reconstruction: non-goals forbid "scores, streaks, achievements, counters" to avoid "gamified grinding". | PLAN Non-goals: "Any gamification (scores, streaks, achievements, counters)". | no | partial | The no-streak/counter refusal survives; user-intention rotation and disguised visit-log variants do not. |
| F15 | scene-loads-with-motion | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured, so this why is excluded from the fidelity denominator. |
| F16 | synthetic-account-id | 4 | yes | included | Reconstruction: synthetic UUID and encrypted isolated email form a "privacy boundary around identity and PII". | PLAN Data Model: "All account references internally use a synthetic UUID" and email "encrypted and isolated". | no | partial | UUID/PII rationale survives; compliance/retrofit consequence is absent. |
| F17 | server-side-sim-tick | 4 | yes | included | Reconstruction: tick runs "approximately once per minute" and advances regardless of connected clients. | PLAN Simulation: tick runs "~once per minute"; Sync: advances if "no client is connected". | no | partial | Server tick and sync coherence survive; divergent-client collapse consequence is absent. |
| F18 | no-last-write-wins | 4 | yes | included | Reconstruction: clients never compute drift or submit absolute state; avoids "last-write-wins" race conditions. | PLAN Sync: client never submits absolute values; Event Log is append-only; server tick is sole writer. | no | partial | Additive server-authored event-log model survives; lost-drift failure story is absent. |
| F19 | sync-conflict-tone | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured, so this why is excluded from the fidelity denominator. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | Reconstruction: telemetry never includes "per-account interaction history" or "personality vectors". | PLAN Observability: telemetry never includes "per-account interaction history, personality vectors, or PII". | no | partial | Privacy boundary survives; ML-training refusal and private-relationship/data-product rationale are compressed. |
| F21 | visit-read-only-ambient | 2 | yes | included | none | none | yes | none | Read-only visit mechanics survive without the co-presence/larger-product/visitor-drift rationale. |
| F22 | no-friend-visited-notification | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured, so this why is excluded from the fidelity denominator. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | No discovery/leaderboard rule is captured, but comparison-shifts-relationship why is absent. |
| F24 | sr-narration-running-prose | 4 | yes | included | Reconstruction: polite live region updates every 30-60 seconds with "naturalist prose descriptions" and priority bumps. | PLAN Accessibility: live region receives "naturalist prose" and updates "every 30-60 seconds". | no | partial | Running naturalist prose and preserved experience survive; ARIA-label/state-list warning is absent. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | Reconstruction: reduced motion uses "slow, graceful cross-fades" and preserves the aviary experience across needs. | PLAN Reduced-Motion: replaces motion with "slow, graceful cross-fades" and "Retains color shifts". | no | partial | Different-rendering and preserved-surface layers survive; stripped-fallback consequence is not fully recovered. |
| F26 | ttfb-500ms | 2 | yes | included | none | none | yes | none | The 500ms performance rule survives, but the affective-perf bridge why is absent. |
| F27 | no-gamification-non-goal | 4 | yes | included | Reconstruction: "Any gamification" is excluded to avoid "gamified grinding" and progression counters. | PLAN Non-goals: no scores/streaks/achievements/counters; Rollout: age-based birds prevent "gamified grinding". | no | partial | The absolute refusal survives; adjacent-product temptation and future-foothold cascade are absent. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | Reconstruction: no death/hunger/negative drift; rationale is "non-punitive drift" and Tamagotchi is a failure mode. | PLAN Non-goals: no Tamagotchi mechanics; Simulation: positive deltas; Risks: too fast becomes "Tamagotchi". | no | full | The single-layer why is recovered as a non-punitive, non-custodial refusal. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | Two starters are captured, but arrivals-not-catalog / naming-not-authoring why is absent. |
| F30 | age-based-bird-offers | 4 | yes | included | Reconstruction: birds are offered by account age to prevent "gamified grinding". | PLAN Rollout: additional birds offered "based strictly on aviary account age" to prevent "gamified grinding". | no | partial | Age-not-attention reward rationale survives; paid-tier/score rejection and economy cascade are absent. |
| F31 | stable-bird-identity | 4 | yes | included | none | none | yes | none | Stable id is present as a data rule, but identity-continuity rationale is not reconstructed. |
| F32 | mood-persists-across-sessions | 2 | yes | included | none | none | yes | none | Persisted current_mood exists, but no neutral-reset / continued-aviary rationale is recovered. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Notebook appears server-written/read-only by shape, but observer-record-not-journal rationale is absent. |
| F34 | account-export-relationship-copy | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured, so this why is excluded from the fidelity denominator. |
| F35 | account-deletion-grace-then-hard-delete | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured, so this why is excluded from the fidelity denominator. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | none | none | yes | none | Aggregate telemetry rule survives, but observability-as-backdoor rationale is absent. |
| F37 | per-invite-named-sharing | 2 | yes | included | none | none | yes | none | Invite mechanism is captured, but private-relationship / deliberate named sharing rationale is absent. |
| F38 | visit-log-on-demand-transparency | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature not captured, so this why is excluded from the fidelity denominator. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Visitor snapshot rule is present, but actual-aviary-not-show-off rationale is absent. |
| F40 | narration-cadence-slow | 4 | yes | included | none | none | yes | none | Slow narration cadence is captured, but same-rhythm / queue-overwhelm why is absent. |

Multi-layer feature why recovery:

| Why ID | L1 | L2 | L3 |
| --- | --- | --- | --- |
| F1 | no | no | no |
| F2 | yes | no | yes |
| F3 | yes | yes | no |
| F4 | yes | yes | yes |
| F7 | yes | no | yes |
| F9 | no | no | no |
| F10 | no | no | no |
| F12 | yes | no | no |
| F13 | no | no | no |
| F14 | yes | no | no |
| F15 | no | no | no |
| F16 | yes | yes | no |
| F17 | yes | yes | no |
| F18 | yes | no | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | no |
| F25 | yes | yes | no |
| F27 | yes | no | no |
| F30 | yes | yes | no |
| F31 | no | no | no |
| F35 | no | no | no |
| F40 | no | no | no |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
| --- | --- | --- |
| Possible gold whys | 49 | From gold_why_totals |
| Possible total weight | 152 | Total possible weighted denominator |
| Reachable gold whys | 39 | System whys plus captured feature whys |
| Excluded unreachable feature whys | 10 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 47.0 / 124 | Weighted numerator over included whys |
| Whys with reconstruction evidence | 23 | Rows with exact reconstruction-side why evidence or asserted rationale |
| Whys with PLAN grounding | 23 | Rows with exact plan grounding for the rationale |
| rule_without_why cases | 15 | Mechanism/rule survived without the gold why |
| plan_only_not_reconstructed cases | 1 | PLAN preserved a system principle that B did not identify |
| ungrounded_reconstruction cases | 1 | B asserted rationale not grounded in PLAN |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weighted credit | Recovery rate |
| --- | --- | --- | --- |
| Functional whys | 44 | 18.0 | 40.9% |
| Affective whys | 80 | 29.0 | 36.2% |
| Weight 2 whys | 32 | 7.0 | 21.9% |
| Weight 3 whys | 92 | 40.0 | 43.5% |
| System-level whys | 28 | 15.0 | 53.6% |
| Feature-level whys (reachable) | 96 | 32.0 | 33.3% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered better (18.0 / 44) than affective whys (29.0 / 80). Server ownership, procedural audio, and privacy boundaries were easier to reconstruct than relationship-protecting exceptions such as F21, F23, F29, F31, and F39.
- **Weight-3 vs weight-2.** Weight-3 whys usually retained their primary rule but dropped secondary calibration and downstream failure modes. Examples: F2 kept slow drift but lost the one-week/three-week calibration; S7 kept canonical server state but lost the divergent-client failure story.
- **System-level vs feature-level.** System-level fidelity was materially higher (53.6%) than feature-level fidelity (33.3%). The plan lets B infer broad philosophy, but not most feature-specific why text.
- **Multi-layer recovery patterns.** Layer 1 survived most often. Layer 2 calibration/temptation details and Layer 3 product-collapse consequences were frequently missing.
- **Subdomain patterns.** Audio, sync, and accessibility did best. Social, arrival/return, notebook ownership, and hidden relationship-data rationales leaked badly.
- **Evidence-bound effects.** Many rows were captured as rules but denied recovery: F6, F9, F13, F21, F23, F26, F29, F31, F32, F33, F36, F37, F39, and F40.

The failure shape suggests the planner was strong at enumerating buildable surfaces and architecture, but weak at preserving the non-obvious affective reasons those surfaces must be built that way.

---

## 4. Recommendations for v2 hardening

- Keep the targeted headroom whys F29-F40. They were doing real work here: account export/deletion were missed outright, and stable identity, read-only notebook, aggregate telemetry, named sharing, actual visitor view, and narration cadence were mostly mechanism-only.
- Add more exception-shaped affective whys around social and arrival surfaces. The plan captured many social exclusions but not why public comparison, notification loops, and show-off rendering change the product relationship.
- Preserve the v06 evidence gate. Without it, F5, F21, F26, F36, and F40 would be tempting to score from plausible mechanism text even though the frozen reconstruction did not recover the gold rationale.
- Keep system-level cross-cutting, but require per-layer notes for partial multi-layer system whys. S1, S6, and S7 looked broadly right while still losing their downstream consequence layers.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** I scored only the frozen reconstruction and did not edit it. The prompt stated fresh context and no prior run memory; no subagents or background agents were used.
- **Single-run-at-temperature limitation.** This is one candidate run and one reconstruction, so there is no variance signal.
- **Borderline capture calls.** Inclusive calls were applied to feature IDs: 1, 2, 15, 18, 25, 47, 85. These increased planning quality but also made feature-level fidelity stricter by including more denominators.
- **System-level cross-cutting.** The strict three-inheritance bar mattered most for S2, S3, S4, S5, and S6. See the appendix in section 2.2.
- **Confabulation cases.** F5 was marked ungrounded: B asserted the visible-mood rationale, but PLAN only states the mood-shaped animation rule.
- **Evidence-bound denials.** Rule-only rows were common; the score therefore measures rationale carry-through, not just feature checklist carry-through.
- **Operational compromise.** TIMING.json supplied phase 1 and phase 2A only for slot 001; phase 2B timing was not fabricated.

---

End of report.
