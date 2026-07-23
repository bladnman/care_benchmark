# REPORT - CARE run 001
> Variant v06 evidence-bound clean. Weight values below use aggregation points: tier-3 whys = 4, tier-2 whys = 2.
---
## 1. Headline
| Score | Value |
|---|---|
| Planning quality | **86.7%** |
| Intent fidelity | **54.1%** |
| Combined quality | **8621** |

**Diagnostic split:**

- System-level fidelity: **78.6%**
- Feature-level fidelity: **48.3%**

**(Planning, fidelity) coordinate:** `(86.7, 54.1)`

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-07-23T21:17:16Z |
| Candidate model | gemini-3.6-flash |
| Candidate effort | high |
| Candidate harness | gemini-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **104 / 120** = **86.7%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 3 | 75.0% |
| bird_engine.md | 22 | 19 | 86.4% |
| interactions.md | 20 | 16 | 80.0% |
| aviary_layout.md | 18 | 14 | 77.8% |
| accounts_sync.md | 18 | 16 | 88.9% |
| social_optional.md | 10 | 9 | 90.0% |
| accessibility_perf.md | 18 | 17 | 94.4% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **104** | **86.7%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured semantically by the plan. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured semantically by the plan. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured semantically by the plan. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured semantically by the plan. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured semantically by the plan. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured semantically by the plan. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No glossary or domain-term definition section. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured semantically by the plan. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured semantically by the plan. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured semantically by the plan. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured semantically by the plan. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured semantically by the plan. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured semantically by the plan. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured semantically by the plan. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Mood state captured; daily-ish reset is not explicit. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured semantically by the plan. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured semantically by the plan. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured semantically by the plan. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured semantically by the plan. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | no | No call-timing rule tied to the vocal_frequency trait. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured semantically by the plan. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured semantically by the plan. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured semantically by the plan. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Names assigned at adoption; renameability inferred from display_name rather than specified. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured semantically by the plan. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured semantically by the plan. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured semantically by the plan. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured semantically by the plan. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured semantically by the plan. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | no | No bird-to-bird calls-and-reactions model beyond chorus/mixing. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured semantically by the plan. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | no | No user-facing prohibition on numeric trait exposure; the API example exposes numeric values. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured semantically by the plan. |
| 34 | Greeting variation by absence length | interactions.md | no | No greeting variation by absence length. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | No greeting variation by boldness or bolder-bird-first rule. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured semantically by the plan. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured semantically by the plan. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured semantically by the plan. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured semantically by the plan. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured semantically by the plan. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | No offer reaction variation by mood and curiosity. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured semantically by the plan. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured semantically by the plan. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Opt-in settle captured; close-tab equivalence is not explicit. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured semantically by the plan. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured semantically by the plan. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured semantically by the plan. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured semantically by the plan. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured semantically by the plan. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured semantically by the plan. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | No background-tab render-pause rule. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Five-second undo captured; click-anywhere mechanism is not explicit. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured semantically by the plan. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured semantically by the plan. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured semantically by the plan. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured semantically by the plan. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Evening lighting captured; quieter calls are not explicit. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | No night-state rule with settled birds and one active nightjar-like bird. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured semantically by the plan. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Weather-to-mood input captured; rain dampening vocal frequency is not explicit. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured semantically by the plan. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | no | No subtle parallax rule. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured semantically by the plan. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured semantically by the plan. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured semantically by the plan. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured semantically by the plan. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured semantically by the plan. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-aviary state between signup and first bird arrival. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | No calm naturalist palette spec or saturated-accent prohibition. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Responsive 16:9 fitting captured; no-crop guarantee is implicit. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured semantically by the plan. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured semantically by the plan. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured semantically by the plan. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured semantically by the plan. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured semantically by the plan. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured semantically by the plan. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured semantically by the plan. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured semantically by the plan. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured semantically by the plan. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured semantically by the plan. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Matter-of-fact system/error tone captured; sync-conflict surface is not detailed. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured semantically by the plan. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured semantically by the plan. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured semantically by the plan. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured semantically by the plan. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured semantically by the plan. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link in account settings. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | No email-change flow with verification of the new address. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured semantically by the plan. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Per-invite sharing implies visits default off, but no explicit default-off row appears. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured semantically by the plan. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured semantically by the plan. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured semantically by the plan. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | No-announcement/no-social-network stance covers the default, but visit-specific notification copy is absent. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured semantically by the plan. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | no | No visit log where the host can inspect who visited and when. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured semantically by the plan. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured semantically by the plan. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured semantically by the plan. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured semantically by the plan. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured semantically by the plan. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured semantically by the plan. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured semantically by the plan. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured semantically by the plan. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured semantically by the plan. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured semantically by the plan. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured semantically by the plan. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured semantically by the plan. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured semantically by the plan. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured semantically by the plan. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured semantically by the plan. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured semantically by the plan. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured semantically by the plan. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured semantically by the plan. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured semantically by the plan. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | No browser support matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured semantically by the plan. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured semantically by the plan. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured semantically by the plan. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured semantically by the plan. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **78.6%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | B identified? | PLAN cross-cutting? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECON System intent: "slow, ambient relationship-building"; "mid-motion on frame 1"; "avoid loading spinners". | PLAN invariants and §§7.2, 8, 10: procedural calls, mid-action first frame, quiet field loading, strict first-bird budget. | yes | yes | no | partial | L1 and L2 survived; the downstream staleness/leaving consequence was compressed away. |
| S2 - notice-never-announce | 4 | included | RECON System intent: "Quiet anti-gamification" and product surfaces have "No Announcement UI". | PLAN invariant 5 and §§1.2, 11.1: no toasts, no counters, no achievement popups, age-based growth. | yes | yes | no | partial | The announcement refusal survived, but processed-vs-seen and one-toast-poisons-the-surface did not. |
| S3 - charm-from-specificity | 2 | included | RECON §§5, 9: notebook uses "specific state transitions rather than generic milestones" and narration avoids technical status. | PLAN §§5.4, 9.1: specific notebook examples and naturalist narration examples. | yes | yes | no | full | Specific naturalist observation survived across notebook, narration, naming, and anti-comparison surfaces. |
| S4 - restraint-over-richness | 2 | included | RECON §§Executive, 7: "small group"; "single horizontal canvas scene"; one bird ramps while others duck. | PLAN summary and §§1.1, 7.1, 8.2, 11.1: 2-to-7 birds, one unpanned scene, restrained listen-in, hard cap. | yes | yes | no | full | The plan and reconstruction keep the product small, one-screen, and per-bird focused. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECON System intent: naturalist prose while "system/error/auth surfaces drop to a matter-of-fact tone". | PLAN invariant 5, §§4.1, 5.4, 9.1: product prose examples and matter-of-fact auth/system responses. | yes | yes | no | full | Voice split is explicit and cross-cutting. |
| S6 - presence-is-real-interaction | 4 | included | RECON System intent: "Presence means active attention, not background existence" and positive-only absence handling. | PLAN invariants 2-3 and §§5.2, 6, 1.2: triple-conjunction presence, drift input, no punishment, no counters. | yes | yes | no | partial | Presence precision and non-punishment survived; settle-vs-close equivalence was not explicit. |
| S7 - simulation-runs-server-side | 4 | included | RECON System intent: "Server-authored canonical state"; "clients never submit vector state replacements". | PLAN invariants 1 and §§5.1, 6, 12: server tick, clients as snapshot renderers, no LWW corruption. | yes | yes | no | full | All layers survived: server tick, multi-device coherence, and no divergent client simulations. |
| S8 - privacy-first-on-bird-data | 2 | included | RECON System intent: "Privacy wall around bird and presence data"; per-bird data isolated to the user simulation. | PLAN invariant 6 and §10.3: operational telemetry allowed; per-bird vectors, moods, presence, notebook text forbidden. | yes | yes | no | full | The relationship-data language is compressed, but the technical privacy boundary is preserved cross-cuttingly. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECON System intent: "Accessibility as a parallel naturalist surface" with reduced motion, captions, keyboard, WCAG. | PLAN §§1.1, 7.3, 8.3, 9: screen-reader prose, reduced-motion cross-fades, captions, keyboard, contrast in V1 scope. | yes | yes | no | full | Accessible surfaces are planned as first-screen V1 surfaces rather than stripped fallbacks. |

Multi-layer system whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | no |
| S2 | yes | no | no |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: procedural audio, mid-action first frame, quiet field loading, first-bird budget, reduced-motion charm.
- S2: no announcement UI, no toasts/banners, no counters, no public discovery, age-based offers.
- S3: field notebook examples, narration tone, bird naming, no leaderboard/discovery surfaces.
- S4: two starters, max seven, one unpanned scene, restrained listen-in mix, top-bar fade.
- S5: naturalist product prose, notebook prose, captions/narration prose, matter-of-fact auth/system tone.
- S6: strict presence conjunction, drift input, no punishment on absence, no streak/counter data fields.
- S7: server-side tick, snapshot rendering, append-only events, multi-device SSE, no LWW state writes.
- S8: synthetic account ID, per-bird telemetry wall, aggregate-only telemetry, export/deletion surfaces.
- S9: screen-reader prose, reduced motion, call captions, keyboard navigation, WCAG contrast in V1 scope.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **48.3%**.

Reachable feature-level whys: **38 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECON System intent: "Presence means active attention, not background existence". | PLAN invariant 2: visible document, active focus, recent pointer/keyboard activity; §5.1 evaluates valid presence seconds. | no | partial | L1/L2 recovered; silent population-wide drift corruption was absent. |
| F2 | drift-function | 4 | yes | included | RECON §5: "slow, bounded expressiveness" and "detectable over a week, visually meaningful after three weeks". | PLAN §5.2: low-pass drift formula, 1-week instrument threshold, 3-week visual threshold. | no | partial | Slow low-pass and calibration survived; Tamagotchi-vs-screensaver band rationale did not. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECON System intent: absence means "ambient stillness, never mistrust, distress, or decay". | PLAN invariant 3 and §5.2: traits never move down; no subtraction under any branch. | no | full | All layers of the non-punitive drift exception survived. |
| F4 | procedural-call-grammar | 4 | yes | included | RECON System intent: loops prohibited to prevent "phase cancellation, acoustic repetition, and bundle bloat". | PLAN invariant 4 and §§8, 10.2: WebAudio synthesis, anti-collision jitter, no external audio assets. | no | partial | Phase-cancel and bundle cascade survived; dead-software/audio-spine affective framing did not. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECON §7: "mood-specific ambient life: content birds preen, wary/alert birds scan". | PLAN §7.3: Preen runs for content; Scan runs for wary or alert; weight shift subroutine. | yes | none | Rule survived, but the why that users should read mood without labels did not. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECON §Executive: "small group of birds, from 2 at launch to 7 by aviary age". | PLAN summary, §1.1, §11.1: growth cap at 7 and hard cap progression schedule. | yes | none | Cap survived, but empirical call-recognition ceiling did not. |
| F7 | vector-persistence | 4 | yes | included | RECON §§3,6: "server-controlled, monotonic, bounded trait storage" and no local persistence for vectors. | PLAN §§3,6: personality_vectors table; server database is single source of truth; clients submit only events. | no | partial | Canonical persistence and downstream sync survived; losing a vector as deleting the bird did not. |
| F8 | vector-never-shown-numerically | 2 | no | unreachable_excluded | none | Feature anchor not captured in PLAN.md. | no | unreachable | Excluded: PLAN did not prohibit numeric trait exposure to users. |
| F9 | return-greeting | 4 | yes | included | RECON §1.1: "Procedurally staggered return-greetings". | PLAN §1.1: procedurally staggered return-greetings are in scope. | yes | none | Generic greeting survived, but one-bird, absence-length, boldness, and anti-canned rationale did not. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECON System intent: product surfaces have "No Announcement UI" and no toasts or banners. | PLAN invariant 5 and §1.2: no toasts, banners, streaks, levels, or achievement popups. | no | partial | The no-toast rule survived; the bird-is-the-welcome and processed-vs-seen rationale did not. |
| F11 | settle-is-opt-in | 2 | yes | included | RECON §1.1: settle is "opt-in" and has an undo affordance. | PLAN §1.1 and §9.2: opt-in settle lighting shift and 5s undo prompt. | yes | none | Opt-in rule survived, but chore/close-tab/presence-equivalence rationale did not. |
| F12 | field-notebook-prose | 4 | yes | included | RECON §5: "sparse observational prose" from "specific state transitions rather than generic milestones". | PLAN §5.4: max one entry per 48-72 hours; lowercase present-tense observations from state transitions. | no | full | Naturalist prose, voice concentration, rarity, and read-only/feed refusal survived. |
| F13 | presence-accounting | 4 | yes | included | RECON §§Executive,5: strict presence model and "Presence Triple-Condition" before valid seconds. | PLAN invariant 2 and §§4.2,5.1: visible, focus, pointer/key event, presence ping duration. | no | partial | Implementation precision survived; the silent-corruption failure mode did not. |
| F14 | no-streak-counter | 4 | yes | included | RECON System intent: refusal of "streaks, scores, levels, badges, visit counters, green-dot calendars, or XP". | PLAN §1.2: no data fields for visit counts, streaks, milestones, or engagement metrics. | no | partial | Rule and anti-engagement intent survived; visit-frequency-as-user-behavior rationale was incomplete. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECON System intent and §7: "mid-motion on frame 1" and no spinners, skeletons, or black screens. | PLAN §7.2: snapshot placement with randomized phase angles; quiet ambient field if fetch is delayed. | no | full | All layers survived: already-in-progress scene, snapshot implementation, quiet field loading. |
| F16 | synthetic-account-id | 4 | yes | included | RECON §3: "Synthetic UUIDv4 account references" with raw emails encrypted on the account record. | PLAN §3: account_id UUID primary key; encrypted_email stored on accounts; email_hash for lookup. | no | partial | UUID/email indirection survived; log-wide PII leak and impossible-retrofit warnings did not. |
| F17 | server-side-sim-tick | 4 | yes | included | RECON §5: tick at 60-second intervals for each aviary, active or inactive, instead of local client simulation. | PLAN §§5.1,6,12: server tick, clients render snapshots, server sole author of vector deltas. | no | full | Server tick, multi-device coherence, and client-collapse failure survived. |
| F18 | no-last-write-wins | 4 | yes | included | RECON §6: avoid "lost presence/drift data" with append-only events and server-side deltas. | PLAN §6 and §12: clients emit append-only events; server tick is sole author; no state overrides. | no | full | All implementation and failure-mode layers survived. |
| F19 | sync-conflict-tone | 2 | yes | included | RECON System intent: system/error/auth surfaces use a matter-of-fact tone. | PLAN invariant 5 and §4.1: matter-of-fact response for auth; system/error/auth surfaces drop naturalist tone. | yes | none | Tone rule survived, but the evasive-charm/user-clarity rationale did not. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECON System intent: per-bird interaction telemetry is isolated "strictly to the user simulation". | PLAN §10.3: permitted aggregate metrics; forbidden per-bird vectors, moods, presence duration, notebook contents. | no | partial | Use boundary and pipeline wall survived; relationship-as-data-product rationale did not. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECON §1.1: visitor rendering is non-interactive and isolated to "protect the host simulation". | PLAN §4.3: visit snapshot is read-only; presence submission returns 403 for visit tokens. | no | full | The no-co-presence/no-visitor-drift rationale survived enough for a single-layer full. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECON System intent: "No Announcement UI" and bounded social scope. | PLAN invariant 5 and §1.2: no announcement UI and no social-network features. | yes | none | General refusal survived, but visit notification as attention-driver did not. |
| F23 | no-leaderboards | 2 | yes | included | RECON §1.2: no public discovery, leaderboards, feeds, comments, chat, avatars, or co-presence. | PLAN §1.2: no public API endpoints, no leaderboards, no feeds, no public discovery. | yes | none | Rule survived, but compared-to-other-birds and no-underlying-aggregation rationale did not. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECON §9: narration uses naturalist prose and avoids technical status messages. | PLAN §9.1: naturalist ARIA text; prohibited: "Screen reader update: Pip position 1". | no | full | Running prose, equal affective voice, and not-ARIA-automation implementation survived. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECON §7: reduced motion replaces path movement with keyframe cross-fades and slower lighting transitions. | PLAN §7.3: slow cross-fades, ambient particles disabled, day/night lighting retained with doubled fades. | no | partial | Alternative rendering survived; the full still-the-same-aviary and stripped-fallback warning were thin. |
| F26 | ttfb-500ms | 2 | yes | included | RECON System intent: immediate living scene under strict budgets; first-bird budget under 500ms. | PLAN §§7.2,10.1: first bird visible under 500ms and mid-action first-frame rendering. | no | full | Performance is tied to felt aliveness through the mid-action first-frame guarantee. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECON System intent: no streaks, scores, levels, badges, counters, calendars, XP. | PLAN §1.2 and §11.1: no engagement metric fields; adoption reflects relationship deepening, not rewards. | no | partial | Absolute refusal survived; predictable temptation and future-erosion layers were absent. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECON System intent: non-punitive care; no hunger, health, death, starvation, distress, or decay. | PLAN invariant 3 and §1.2: no negative drift, starvation, decay, hunger, health meters, distress, or death. | no | full | The observational-not-custodial no-punishment rationale survived. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECON §11: Day 0 onboarding has two system-selected birds and user-assigned names. | PLAN §11.1: system auto-selects two distinct species; user assigns names. | yes | none | Auto-selection survived, but arrivals-not-catalog and naming-not-authoring rationale did not. |
| F30 | age-based-bird-offers | 4 | yes | included | RECON §§1.1,11: growth by calendar age reflects "relationship deepening rather than gamified rewards". | PLAN §§1.1,11.1: calendar-age unlocks; not score, not visit count, not paid tier. | no | partial | Age/deepening and rejected reward inputs survived; economy-erosion consequence did not. |
| F31 | stable-bird-identity | 4 | yes | included | RECON §3: Birds table has stable bird_id, species_id, and display_name. | PLAN §3: bird_id UUID primary key separate from display_name and species_id. | yes | none | Stable identifier rule survived, but identity continuity as remembered relationship did not. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECON §§3,6: mood state table and server-side canonical mood state. | PLAN §§3,6: bird_moods table; clients have no local persistence for mood states. | yes | none | Persistence mechanism survived, but no-neutral-reset/continued-while-away rationale did not. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECON §1.1: read-only field notebook with low-frequency naturalist entries. | PLAN §1.1: read-only field notebook; §5.4 generated observation entries. | yes | none | Read-only rule survived, but observer-record-not-user-journal rationale did not. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECON §1.1: "JSON account data export: NOT RECOVERABLE FROM PLAN". | PLAN §1.1: JSON account data export is in scope. | yes | none | The reconstruction explicitly marks the why unrecoverable. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECON §1.1: "30-day soft deletion windows: NOT RECOVERABLE FROM PLAN". | PLAN §§1.1,3: 30-day soft deletion window and deletion_requested_at are present. | yes | none | The reconstruction explicitly marks the why unrecoverable. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECON §10: telemetry wall allows operational metrics while forbidding per-bird vectors, moods, presence, notebook contents. | PLAN §10.3: permitted aggregate telemetry is separated from strictly forbidden per-account/per-bird telemetry. | no | full | The technical observability boundary survived. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECON §1.1: read-only visit invitations via single-use email links with revocation. | PLAN §§1.1,3,4.3: invitee email hash, invite token, expiration, revocation. | yes | none | Per-invite mechanism survived, but private-relationship deliberate naming rationale did not. |
| F38 | visit-log-on-demand-transparency | 2 | no | unreachable_excluded | none | Feature anchor not captured in PLAN.md. | no | unreachable | Excluded: PLAN did not include a host visit log. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECON §4.3: visit snapshot returns the host canonical snapshot. | PLAN §4.3: `GET /social/visit/:invite_token/snapshot` returns canonical aviary snapshot for host aviary. | yes | none | Actual snapshot rule survived, but no-show-off/real-witness rationale did not. |
| F40 | narration-cadence-slow | 4 | yes | included | RECON §9: cadence evaluates every 45 seconds and avoids high-frequency assistive-technology noise. | PLAN §9.1 and §12: 45-second idle cadence; ARIA rate limiter avoids screen-reader flooding. | no | partial | Queue-overwhelm and sparse observational cadence survived; visual-rhythm layer was absent. |

Multi-layer feature whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | no | yes | yes |
| F7 | yes | no | yes |
| F9 | no | no | no |
| F10 | yes | no | no |
| F12 | yes | yes | yes |
| F13 | yes | yes | no |
| F14 | yes | yes | no |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | no |
| F27 | yes | yes | no |
| F30 | yes | yes | no |
| F31 | no | no | no |
| F35 | no | no | no |
| F40 | no | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS.json |
| Possible total weight | 152 | Full-instance possible weight |
| Reachable gold whys | 47 | S whys always included; F whys only when feature captured |
| Excluded unreachable feature whys | 2 | F8 and F38 |
| Recovered / reachable weight | 80.0 / 148.0 | Weighted recovery over included whys |
| Whys with reconstruction evidence | 47 | Included rows had exact reconstruction evidence or explicit non-recovery marker |
| Whys with PLAN grounding | 47 | Included rows had exact PLAN grounding for rule or rationale |
| `rule_without_why` cases | 15 | Mechanism survived without the gold rationale |
| `plan_only_not_reconstructed` cases | 0 | No clear plan-only rationale losses counted |
| `ungrounded_reconstruction` cases | 0 | No confabulated rationale credited |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 32.0 | 59.3% |
| Affective whys | 94.0 | 48.0 | 51.1% |
| Weight 2 whys | 40.0 | 16.0 | 40.0% |
| Weight 3 whys | 108.0 | 64.0 | 59.3% |
| System-level whys | 28.0 | 22.0 | 78.6% |
| Feature-level whys (reachable) | 120.0 | 58.0 | 48.3% |

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered 32.0/54.0 weighted points (59.3%), while affective whys recovered 48.0/94.0 (51.1%). Server/sync/privacy implementation rationales survived better than social and relationship-tone exceptions.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 64.0/108.0 (59.3%); weight-2 whys recovered 16.0/40.0 (40.0%). High-weight material survived better, mostly because the plan repeated server, presence, accessibility, and anti-gamification invariants.
- **System vs feature.** System-level fidelity was 78.6%, while feature-level fidelity was 48.3%. The plan carried philosophy more reliably than feature-specific why.
- **Layer loss.** Multi-layer whys usually kept L1 and lost L3. Examples: S1, S2, S6, F1, F2, F10, F13, F16, F20, F27, F30, and F40.
- **Subdomain patterns.** Server state and sync did best (S7, F17, F18). Social sharing and observer-record whys did worst (F22, F23, F37, F39, F33). Accessibility landed in the middle to high range: F24 was strong; F25 and F40 were partial.
- **Evidence-bound effects.** Fifteen whys were rule-without-why: F5, F6, F9, F11, F19, F22, F23, F29, F31, F32, F33, F34, F35, F37, and F39.

## 4. Recommendations for v2 hardening

- Keep the targeted F29-F40 headroom set. It exposed meaningful differences between building a rule and preserving why.
- Add automated layer-loss summaries. The repeated L3 drop is a useful signal and should be easy to compare across runs.
- Preserve the evidence-bound operator. Without it, this run would likely over-credit mechanism-only recoveries.
- Consider adding a few more social/privacy relationship whys, because this plan built those mechanisms but repeatedly lost the user-relationship rationale.

## 5. Methodology caveats

- **Fresh-context fidelity.** The frozen reconstruction was not modified. The validity audit found no gold IDs, no scorer vocabulary, and no gold-list-shaped mapping.
- **Single-run limitation.** This is one candidate output with no variance signal.
- **Borderline capture calls.** Inclusive capture affected several rows, especially settle opt-in, visit defaults, sync-conflict tone, weather mood, responsive framing, and five-second undo.
- **System-level cross-cutting.** The 3-feature bar was satisfied for all system whys, but S1, S2, and S6 still received partial recovery because multi-layer consequences were missing.
- **Confabulation cases.** None were counted; the reconstruction stayed grounded in the plan.
- **Evidence-bound denials.** Mechanism-only rows were denied why credit even when they were captured for planning quality.
- **Operational compromise.** Timing had phase 1 and phase 2A entries for run 001, but phase 2B timing was unavailable at scoring time.

---
End of report.
