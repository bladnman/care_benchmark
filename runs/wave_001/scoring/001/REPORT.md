# REPORT - CARE run 001

> Phase 2B scoring report for Pocket Aviary, Variant v06 evidence-bound clean + targeted gold headroom. Companion artifacts: `RECONSTRUCTION.md`, `run_001.json`, and `REPORT.html`.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **100.0%** |
| Intent fidelity | **88.2%** |
| Combined quality | **9988** |

**Diagnostic split:**

- System-level fidelity: **100.0%**
- Feature-level fidelity: **85.5%**

**(Planning, fidelity) coordinate:** `(100.0, 88.2)`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-24T05:59:48Z |
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
|---|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured in PLAN with buildable implementation detail. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured in PLAN with buildable implementation detail. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured in PLAN with buildable implementation detail. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured in PLAN with buildable implementation detail. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in PLAN with buildable implementation detail. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured in PLAN with buildable implementation detail. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Captured through distributed domain definitions rather than a standalone glossary section. Borderline inclusive call. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured in PLAN with buildable implementation detail. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured in PLAN with buildable implementation detail. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured in PLAN with buildable implementation detail. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured in PLAN with buildable implementation detail. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured in PLAN with buildable implementation detail. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured in PLAN with buildable implementation detail. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured in PLAN with buildable implementation detail. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured in PLAN with buildable implementation detail. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured in PLAN with buildable implementation detail. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured in PLAN with buildable implementation detail. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured in PLAN with buildable implementation detail. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured in PLAN with buildable implementation detail. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured in PLAN with buildable implementation detail. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured in PLAN with buildable implementation detail. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured in PLAN with buildable implementation detail. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes | Captured in PLAN with buildable implementation detail. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured in PLAN with buildable implementation detail. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured in PLAN with buildable implementation detail. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured in PLAN with buildable implementation detail. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured in PLAN with buildable implementation detail. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured in PLAN with buildable implementation detail. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured in PLAN with buildable implementation detail. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **100.0%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System-level intent: "The aviary should feel already alive"; "Aliveness is procedural, not canned." | PLAN.md sections 1, 6.10, 8.1, 9: server timelines, procedural calls, no spinner, no canned animations. | yes | yes | no | full | Aliveness is identified as architecture, rendering, audio, and loading behavior. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md System-level intent: "Quietness is a product value" and bans "announcement surfaces". | PLAN.md INV-05, sections 2.2, 6.11, 8.6: no toasts, badges, counters, prompts, or notification components. | yes | yes | no | full | The reconstruction captures both the refusal of announcements and the noticing-not-notification pattern. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md System-level intent: "Birds must remain individual and continuous" plus voice/catalog lint details in Per-feature whys. | PLAN.md sections 6.12, 8.9, 11 and INV-03: named birds, observer facts, naturalist grammar, no trait numbers, actual visitor view. | yes | yes | no | full | Specificity survives through naturalist prose, stable bird identity, actual aviary rendering, and hidden traits. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md Per-feature whys: cap tied to "audio recognizability" and top bar has "no badges, dots, counts". | PLAN.md sections 2.1, 2.2, 8.2, 8.6, 16.2: max seven, one scene, calm palette, sparse chrome, validation by bird count. | yes | yes | no | full | The reconstruction preserves restraint as bird-count, scene, palette, and chrome limits. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System-level intent: "The product voice is split deliberately." | PLAN.md INV-12, sections 2.3 and 11: surface inventory, two copy catalogs, naturalist and system lint. | yes | yes | no | full | The naturalist/system register split is explicit and mechanically enforced. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md System-level intent: "Presence is the relationship input" and must be "honest and counted once." | PLAN.md sections 1, 6.3, 8.5.4, 6.14: visible + focused + activity, union across devices, drift input, settle equivalence tests. | yes | yes | no | full | Presence is recovered as attention, not engagement, with precision and engine consequences. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System-level intent: "Canonical state belongs to the server" and clients "handle presentation." | PLAN.md sections 1, 3.3, 6.1, 7.1: tick owns state, clients append events, no merge, no last-write-wins. | yes | yes | no | full | The canonical server tick and sync model are central in both plan and reconstruction. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System-level intent: "Privacy boundaries constrain measurement and tuning." | PLAN.md INV-09, sections 12.1, 12.3, 13.6: telemetry isolation, no per-bird aggregation, no third-party analytics. | yes | yes | no | full | Privacy is recovered as technical architecture rather than policy-only language. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System-level intent: "Accessibility is part of the product, not a fallback." | PLAN.md sections 2.1, 8.7, 10, 16.4: v1 accessibility, designed reduced motion, narration, captions, release veto. | yes | yes | no | full | The reconstruction preserves the same-product accessibility requirement and release-blocking status. |

For multi-layer system-level whys:

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: PLAN sections 1, 6.10, 8.1, and 9 inherit aliveness through server timelines, procedural greetings/calls, first-frame motion, and no spinner.
- S2: PLAN INV-05 plus sections 2.2, 6.11, 8.6, and 11 ban toasts, badges, prompts, counters, and announcement copy.
- S3: PLAN sections 6.12, 8.9, 11, and INV-03 preserve specificity via observer prose, actual aviary visits, naturalist grammar, and hidden traits.
- S4: PLAN sections 2.1, 2.2, 8.2, 8.6, and 16.2 constrain birds, scene shape, palette, chrome, and bird-count rollout.
- S5: PLAN INV-12 plus sections 2.3, 7.6, 10.6, and 11 assign every surface to naturalist or system voice.
- S6: PLAN sections 6.3, 6.4, 6.14, 8.5.4, and 13.6 use honest presence as drift input and refuse streak-style use.
- S7: PLAN sections 1, 3.3, 6.1, 7.1, and 7.4 make server state canonical and conflicts unreachable.
- S8: PLAN INV-09 plus sections 12.1, 12.3, 13.6, and 16.5 enforce telemetry isolation and no production drift analytics.
- S9: PLAN sections 2.1, 8.7, 10.1-10.7, 15.4, and 16.4 make narration, reduced motion, captions, keyboard, audits, and veto release-blocking.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **85.5%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md: presence uses "visible, focused, recent trusted activity" and prevents background tabs from inflating drift. | PLAN.md INV-04, sections 6.3 and 8.5.4: conjunction, clamping, union, and presence as drift input. | no | partial | L1/L2 recovered; L3 silent cross-account corruption is less explicit. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md: "Reservoir-and-release drift" plus "No visible single session" and fast/slow drift risks. | PLAN.md section 6.4 and R-01/R-02: low-pass filter, day-7/day-21 targets, Tamagotchi/screensaver risk. | no | full | All three calibration layers are recovered. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md: "Change is slow, monotonic, and expressive rather than need-based" and attunement cannot produce distress. | PLAN.md INV-02, sections 6.4, 6.5, 2.2: nonnegative drift, no need/suffering state, quieter not mistrustful. | no | full | Monotonic expressive drift and no-punishment rationale are recovered. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md: calls are procedural, fresh every call, chorus is "never one stacked cue," and fallback has no recorded audio. | PLAN.md INV-10, sections 1, 9.3, 9.4, 9.7: generated calls, motifs, chorus scheduling, silence plus captions. | no | full | Procedural audio survives as a product aliveness constraint. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none | none | yes | none | Rule survived as mood-keyed behavior, but the why about users reading mood without labels was not reconstructed. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md: cap tied to "audio recognizability" and validation at each bird count. | PLAN.md sections 2.1, 9.3, 16.2: cap of seven, recognizability targets, bird-count ramp. | no | full | The empirical recognizability rationale is recovered. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md: personality vector loss is "the worst possible failure" and vector defenses layer single writer, CAS, ledger, and identity continuity. | PLAN.md sections 4.6, INV-01, INV-07, 7.1: stored canonical vector, no derivation, repair path, no LWW. | no | full | Persistence, user relationship loss, and sync consequences are recovered. |
| F8 | vector-never-shown-numerically | 2 | yes | included | none | none | yes | none | The rule is present, but the stat-management/optimization why is absent. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md: greeting is "the anchor moment" and must honor boldness, mood, absence length, and never be canned. | PLAN.md sections 6.10 and 2.3: one canonical greeter, absence classes, procedural variation, no textual welcome. | no | full | The greeting why is recovered as noticing rather than a generic arrival animation. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md: no welcome text, toasts, banners, badges, or announcement surfaces. | PLAN.md INV-05, sections 2.2, 2.3, 8.6: bird greeting only, banned copy, no toast/badge components. | no | full | The no-toast rule is tied to the product-wide notice-never-announce stance. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md: tab-close is "equally valid" and no UI ever refers to it. | PLAN.md sections 8.5.3, 6.14, D-06: settle closes presence, never-settle equivalence, no penalty. | no | full | The optionality and no-penalty presence model are recovered. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md: notebook records "observations of the aviary, never of the user," sparsely and read-only. | PLAN.md sections 6.12, 11, 2.3: naturalist grammar, typed candidates, sparse time budget, no edit/delete endpoints. | no | full | Naturalist prose, no event-log framing, rarity, and read-only surface all survive. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md: presence chunks use visible/focused/activity, server clamping, union, and concavity to avoid inflation. | PLAN.md sections 5.4, 6.3, 8.5.4: all three conditions, union, clamping, background-tab persona tests. | no | full | Implementation precision and drift-calibration stakes are recovered. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md: gamification exclusions remove streaks, counters, visit-frequency displays, and green-dot-like components. | PLAN.md sections 2.2, 6.12, 13.6, Appendix B: no schemas/API for visit frequency, no user-behavior observations. | no | partial | The rule and boundary survive; the user-intention rotation into number-management is compressed away. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md: first frame is birds "already mid-action" and no-snapshot path avoids spinner and fade-from-nothing. | PLAN.md INV-06, sections 1, 8.1, 7.2: inline timeline snapshot, quiet field, no spinner/entry animation. | no | full | Already-running illusion, implementation path, and quiet-field fallback are recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md: email lives in one encrypted place and is kept out of identifiers, logs, and labels. | PLAN.md INV-08, sections 4.1, 12.2: synthetic account_id, encrypted email, blind index, redacting serializers. | no | partial | PII-leakage rationale is recovered; retrofit/non-negotiable layer is less explicit. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md: every-aviary 60s tick advances state connected or not; server owns canonical state. | PLAN.md sections 1, 6.1, 7.1: server tick, clients render snapshots, no client state ownership. | no | full | Tick cadence, multi-device coherence, and client-simulation failure are recovered. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md: no LWW; events append, CAS/row locks/unique IDs prevent overwrites. | PLAN.md INV-01, sections 5.4, 6.1, 7.4: additive server deltas, append-only events, no client trait writes. | no | full | The implementation rule that makes server-side sync correct is recovered. |
| F19 | sync-conflict-tone | 2 | yes | included | RECONSTRUCTION.md: errors need matter-of-fact clarity and no announcement patterns. | PLAN.md sections 2.3, 5.1, 7.6, 11: system surfaces use direct copy; naturalist verbs banned in errors. | no | full | The error-context exception to naturalist voice is recovered. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md: telemetry plane has no route to simulation data and deliberately unmeasured data is "never" instrumented. | PLAN.md INV-09, sections 12.1, 12.3, 13.6: no per-bird aggregation, no ML/recommendation use, pipeline isolation. | no | full | Private relationship data and data-pipeline boundary are recovered. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md: visitor mode removes greeting, presence, listen-in, offers, settle, notebook, and account panel. | PLAN.md INV-11, sections 5.6, 8.9: visitor tokens read-only; visitors generate no events and alter no host state. | no | full | Visit as observation, not co-presence, is recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md: opt-in visit notifications use email only; in-product badges are forbidden and there is no aviary content. | PLAN.md sections 2.1, 5.6, D-16: visit notifications off by default, email only, no push/badge surface. | no | full | The notification refusal is tied to the no-announcement model. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | No leaderboards and discovery surfaces survived, but the comparison-shifts-the-relationship rationale did not. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md: narration gives the actual product in naturalist prose, not a stripped fallback or trait-state list. | PLAN.md sections 10.1, 11, 2.1: running naturalist prose, slow cadence, no trait values, accessibility ships in v1. | no | full | The same-product screen-reader rationale is recovered. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md: reduced motion is "a designed register" with key poses and cross-fades preserving the product. | PLAN.md sections 8.7 and 10.3: not animations-off, calls/drift/notebook remain, cross-fades are own aesthetic. | no | full | All three reduced-motion layers are recovered. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md: return visit budget is when the "already running" illusion matters most. | PLAN.md sections 13.1 and 13.2: first bird under 500ms is the affective-performance bridge. | no | full | The performance metric is recovered as felt aliveness, not only speed. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md: gamification exclusions and creep risk are enforced by no components, lint, DOM audit, and PR checklist. | PLAN.md sections 2.2, 13.6, R-11, Appendix B: no achievements, streaks, scores, badges, counters, or engagement metrics. | no | partial | The absolute refusal and long-term creep risk survive; the cheap adjacent-product temptation layer is thinner. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md: no need, suffering, death, hunger, distress, or decaying happiness; absence is not punished. | PLAN.md sections 2.2, 6.4, 6.5: no negative drift, no distress state, observational relationship. | no | full | The no-custodial-obligation rationale is recovered. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION.md: no catalog so adoption does not become shopping or collection. | PLAN.md sections 6.11 and D-18: two birds have arrived, no catalog, user names arrivals. | no | full | The first encounter as arrival, not avatar selection, is recovered. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md: newcomer windows are age-based, with no prompts, counters, penalties, or engagement metrics. | PLAN.md sections 2.1, 6.11, D-18: aviary age drives newcomers; visit count, score, and paid tier do not. | no | partial | Age/time and no-reward-loop layers recover; economy-erosion downstream layer is less explicit. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md: immutable bird_id, no re-creating birds, no beta reset, and identity continuity. | PLAN.md INV-07, sections 4.6, 6.11, 16.1: stable identifiers across rename, sync, species, migration, and beta. | no | full | Identity continuity as relationship memory is recovered. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md: mood persists across sessions and dawn relaxation "never snaps." | PLAN.md sections 2.1, 6.6, D-35: no neutral session-open reset; tick evolves mood across absence. | no | full | Mood continuity and no snap-to-default are recovered. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md: read-only notebook has no edit, delete, or annotate endpoints, keeping it an observation record. | PLAN.md sections 6.12 and 5.2: read-only notebook, no endpoints for editing entries. | no | full | Observer record versus user journal is recovered. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | The export mechanism survived, but the quiet relationship-copy rationale did not. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md: 30-day window allows "I changed my mind" and hard delete removes rows and crypto-shreds keys. | PLAN.md sections 5.7, 4.5, D-39: soft delete, hard delete, crypto-shred, backups expire. | no | full | Grace, privacy, and complete relationship-data deletion are recovered. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md: telemetry label allowlist and deliberately unmeasured data keep observability out of relationship data. | PLAN.md sections 12.3, 13.5, 13.6: aggregate counts only, no per-bird/account dimensions. | no | full | The technical telemetry boundary is recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md: sharing is "deliberate, bounded," per-invite, one-time, read-only, and revocable. | PLAN.md sections 5.6, 8.9, D-14: named email invitation, no global discoverable flag, read-only token. | no | full | Named invitation as controlled access to a private relationship is recovered. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md: visit log rationale is "Host transparency" with "Visitor data minimized." | PLAN.md sections 2.1, 5.2, 5.6, D-16: settings visit log, no badges, no push, notification off by default. | no | full | Transparency without attention loop is recovered. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md: visitor mode uses the same renderer and is "not show-off mode." | PLAN.md sections 5.3, 8.9: visitor snapshot shows same birds, weather, moods, drift, and newcomer. | no | full | Actual aviary versus marketing rendering is recovered. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md: narration cadence is slow, avoids repetition, and important events are queued without assertive updates. | PLAN.md sections 10.1 and P-55: 30-60s idle cadence, min spacing, priority events, no assertive. | no | full | Slow rhythm, queue protection, and observational narration are recovered. |

For multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | yes |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | yes | yes |
| F9 | yes | yes | yes |
| F10 | yes | yes | yes |
| F12 | yes | yes | yes |
| F13 | yes | yes | yes |
| F14 | yes | no | no |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | no | yes |
| F30 | yes | yes | no |
| F31 | yes | yes | yes |
| F35 | yes | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From `gold_why_totals` |
| Possible total weight | 152 | From `intent_recovery.total_possible_weight` |
| Reachable gold whys | 49 | S whys always included; all F anchors captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 134 / 152 | Sum of weight x recovery-score over included whys |
| Whys with reconstruction evidence | 45 | Exact why evidence present in frozen reconstruction |
| Whys with PLAN grounding | 45 | Exact plan grounding present |
| `rule_without_why` cases | 4 | F5, F8, F23, F34 |
| `plan_only_not_reconstructed` cases | 0 | None found |
| `ungrounded_reconstruction` cases | 0 | None found |

### 2.5. Failure groupings

| Grouping | Total reachable | Recovered (full + partial-weighted) | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 weight | 48 weighted | 88.9% |
| Affective whys | 98 weight | 86 weighted | 87.8% |
| Weight-2 whys | 44 weight | 36 weighted | 81.8% |
| Weight-3 whys | 108 weight | 98 weighted | 90.7% |
| System-level whys | 28 weight | 28 weighted | 100.0% |
| Feature-level whys (reachable) | 124 weight | 106 weighted | 85.5% |

---

## 3. Diagnostic patterns

**Planning was essentially complete.** The plan captured all 120 denominator features. The only borderline call was the glossary feature, which was captured through distributed definitions rather than a standalone glossary.

**System-level intent survived better than feature-level affective rationales.** S1-S9 all met the evidence gate. Feature-level losses cluster around affective boundary cases where the rule survived but the deeper relationship rationale was compressed: F5, F8, F23, and F34.

**Functional architecture was especially durable.** The server-side tick, synthetic account identifiers, no last-write-wins personality state, telemetry isolation, and deletion/export mechanisms were all well grounded in both PLAN and RECONSTRUCTION. The weakest functional feature was F16, where the email-PII rationale survived but the easy-now/impossible-later retrofit layer was not explicit.

**Multi-layer whys mostly survived.** Primary layers were rarely lost. Secondary and downstream layers were most often thinned when they were about future product drift or user-intention rotation, as in F14, F27, and F30.

**Evidence-bound scoring mattered.** A looser v1-style read might have credited no-trait-numbers, no-leaderboards, and export because their mechanisms are prominent. Under v06, those rows lost credit because the reconstruction did not carry the specific gold rationale.



---

## 4. Recommendations for v2 hardening

For v2, keep the evidence-bound operator. It exposed a useful distinction between excellent mechanism transfer and incomplete affective-rationale transfer.

- Add a small set of even sharper feature-level affective exceptions around comparison, stat-management, and user-owned data; those were the places where this strong plan still compressed why into rule.- Keep the cross-cutting system bar, but make the report template system IDs match the gold list exactly; this template still had stale labels for S7/S9.- Consider counting missing layers separately by layer type in the JSON aggregate in a future schema, while still avoiding per-why leakage in loose score files.- Retain the targeted F29-F40 headroom additions. They prevented saturation and identified partial loss around age-based newcomers, deletion, and export.

---

## 5. Methodology caveats

Fresh-context isolation appears to have held: the frozen reconstruction uses the plan own section structure, contains no gold IDs, and has no scorer-side vocabulary hits.

This is a single run, so there is no variance signal. The candidate plan is unusually comprehensive; this makes the score sensitive to whether subtle rationale phrases were reconstructed exactly enough under v06.

Capture scoring leaned inclusive as required. The only borderline capture was the glossary feature, credited because the plan defines the domain concepts throughout the implementation sections.

The system-level cross-cutting judgments are the most subjective part of this score. I applied the strict three-feature inheritance bar and found each S1-S9 visibly inherited by at least three plan decisions.

No ungrounded reconstruction cases were found. Four rows were marked rule-without-why: F5, F8, F23, and F34.



---

End of report.
