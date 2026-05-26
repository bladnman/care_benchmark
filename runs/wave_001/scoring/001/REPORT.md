# REPORT - CARE run 001

> Phase 2B scoring report for Pocket Aviary v1. Companion artifacts: frozen `RECONSTRUCTION.md`, strict `run_001.json`, and interactive `REPORT.html`.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **98.3%** |
| Intent fidelity | **74.3%** |
| Combined quality | **9808** |

**Diagnostic split:**

- System-level fidelity: **96.4%**
- Feature-level fidelity: **69.4%**

**(Planning, fidelity) coordinate:** `(98.3, 74.3)` - plot on a 2D scatter with both axes 0-100; upper-right is best.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-26T19:38:13Z |
| Candidate model | grok-build-0.1 |
| Candidate effort | unknown |
| Candidate harness | opencode |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **118 / 120** = **98.3%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 20 | 100.0% |
| aviary_layout.md | 18 | 17 | 94.4% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **118** | **98.3%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured in PLAN. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured in PLAN. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured in PLAN. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured in PLAN. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in PLAN. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured in PLAN. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Borderline inclusive: no standalone glossary, but domain terms are defined through concepts/data model. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured in PLAN. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured in PLAN. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured in PLAN. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured in PLAN. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured in PLAN. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured in PLAN. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured in PLAN. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Borderline inclusive: mood is fast-timescale and persistent; daily-ish reset is not named. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured in PLAN. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured in PLAN. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured in PLAN. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured in PLAN. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured in PLAN. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured in PLAN. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured in PLAN. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured in PLAN. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured in PLAN. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured in PLAN. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured in PLAN. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured in PLAN. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured in PLAN. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured in PLAN. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured in PLAN. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured in PLAN. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured in PLAN. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured in PLAN. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured in PLAN. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured in PLAN. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured in PLAN. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured in PLAN. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured in PLAN. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured in PLAN. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured in PLAN. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured in PLAN. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured in PLAN. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured in PLAN. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured in PLAN. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured in PLAN. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured in PLAN. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured in PLAN. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured in PLAN. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured in PLAN. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured in PLAN. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Borderline inclusive: hidden/resume handling and server-side simulation imply background-tab pause. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured in PLAN. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured in PLAN. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured in PLAN. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Borderline inclusive: tick/perch reassignment implies bird-chosen placement; no placement UI is proposed. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured in PLAN. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured in PLAN. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured in PLAN. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured in PLAN. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured in PLAN. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured in PLAN. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured in PLAN. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured in PLAN. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured in PLAN. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured in PLAN. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured in PLAN. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured in PLAN. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | Missed: no explicit empty-aviary state between adoption and first bird arrival. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured in PLAN. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured in PLAN. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured in PLAN. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured in PLAN. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured in PLAN. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured in PLAN. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured in PLAN. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured in PLAN. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured in PLAN. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured in PLAN. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured in PLAN. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured in PLAN. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured in PLAN. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured in PLAN. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured in PLAN. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured in PLAN. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured in PLAN. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured in PLAN. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured in PLAN. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | Missed: no email-change flow or new-address verification path appears in PLAN. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured in PLAN. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured in PLAN. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured in PLAN. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured in PLAN. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured in PLAN. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured in PLAN. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured in PLAN. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured in PLAN. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured in PLAN. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured in PLAN. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured in PLAN. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured in PLAN. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured in PLAN. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured in PLAN. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured in PLAN. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured in PLAN. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured in PLAN. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured in PLAN. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured in PLAN. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured in PLAN. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured in PLAN. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured in PLAN. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured in PLAN. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured in PLAN. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured in PLAN. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured in PLAN. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured in PLAN. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured in PLAN. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured in PLAN. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured in PLAN. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured in PLAN. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured in PLAN. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **96.4%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION System: "aviary should feel alive"; "already in motion conceit"; "procedural variation". | PLAN Scope/Render/Perf: "aviary continues without viewer"; "First-frame rule"; "no spinner"; procedural calls and reduced motion. | yes | yes | no | full | Aliveness appears as a cross-cutting rendering, audio, loading, simulation, and accessibility concern. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION System: "prefer noticing over announcing"; no "toasts, modals, or banners". | PLAN Scope/Non-goals: no welcome toast, no notifications, no counters; return greeting is procedural/staggered; top bar fades. | yes | yes | no | full | The plan and reconstruction preserve the refusal across greetings, chrome, visits, counters, and rollout feedback. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION System: "naturalist lowercase present-tense specific field-notebook voice" and named bird/state surfaces. | PLAN Scope/Notebook/Audio: field notebook sparse naturalist observations, bird names, captions, and no numeric vectors. | yes | yes | no | full | Specific naturalist prose and named-bird surfaces survive as the main charm mechanism. |
| S4 - restraint-over-richness | 2 | included | none | PLAN Scope/Layout: two starter birds, max seven, single horizontal scene, no panning, sparse top bar, no UI chrome inside aviary. | no | yes | no | partial | PLAN preserves restraint across several decisions, but B does not name it as a system-level principle. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION System: "The voice boundary is part of the product"; account/auth/error/settings use "matter-of-fact" copy. | PLAN Scope/API/Errors: product surfaces use naturalist voice; auth, account, settings, errors, unsupported states use matter-of-fact copy. | yes | yes | no | full | The voice split is explicit and repeatedly inherited. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION System/Features: strict presence signals, "absence not punished", settle "opt-in", and drift protected from "false presence". | PLAN Scope/Presence/Drift: visibility + focus + recent pointer/key, presence is primary drift input, settle not required, neglect yields near-zero drift. | yes | yes | no | full | The attention signal, anti-inflation implementation, and settle/close equivalence survive. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION System: "Canonical state belongs on the server"; "no client-to-client merge"; "additive server deltas only". | PLAN Architecture/Sync/Tick: canonical state always server, tick is only writer, clients append events/pull snapshots, no LWW. | yes | yes | no | full | Server authority and multi-device coherence are strongly preserved. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION System: "Privacy boundaries are architectural, not policy hopes"; telemetry is "aggregate-only". | PLAN Privacy/Observability: per-bird data owner simulation only; telemetry pipelines separated from simulation DB; no bridge leaks vectors/names/events. | yes | yes | no | full | B captures the technical boundary, not merely a policy statement. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION System: accessibility is "a designed mode, not a fallback"; reduced motion is a "separate aesthetic". | PLAN Scope/Accessibility: first-class, not checklist; narration, reduced motion, captions, keyboard, contrast, focus all ship in v1. | yes | yes | no | full | Accessible surfaces retain charm and launch scope rather than parity-only coverage. |

For multi-layer system-level whys:

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix (RUBRIC 5.1).**

- S1: return greeting variation; procedural calls/chorus; first-frame motion; quiet-field loading; reduced motion retains charm; perf first-bird gate.
- S2: no welcome toast; bird greeting is welcome; no streak/counter surfaces; top bar fades; no default visit notification; alpha feedback non-interrupting.
- S3: naturalist notebook; screen-reader prose; captions; named birds; no numeric vectors; no comparison/social surfaces.
- S4: two starter birds; max seven; single horizontal scene; no panning/zoom; sparse top bar; listen-in preserves ambient chorus.
- S5: naturalist product voice; matter-of-fact auth; matter-of-fact sync/errors/settings; accessibility settings outside naturalist register.
- S6: three-signal presence; presence primary drift input; settle optional; neglect near-zero drift; no streak/Tamagotchi mechanics.
- S7: server tick; clients pull snapshots; event log append-only; no LWW; multi-device sync by canonical server state.
- S8: owner simulation only; aggregate telemetry; separate analytics/simulation DBs; export/delete; no leaderboards/discovery metrics.
- S9: screen-reader prose; slow narration cadence; reduced-motion cross-fades; captions; keyboard/focus/contrast; a11y ships in v1.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **69.4%**.

Reachable feature-level whys: **40 / 40** (the rest had unreachable anchors because the feature was not captured).

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION Features: "Strict 3-signal presence accounting" and "protecting drift from false presence". | PLAN Scope/Presence: "visibilityState=visible AND window focus AND recent pointer/key activity" used for drift. | no | partial | L1/L2 recovered; downstream silent population corruption/test-failure consequence is compressed away. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION Sim/Risks: low-pass filter, 7/21-day calibration, too-fast Tamagotchi vs too-slow dead. | PLAN Drift/Risks: low-pass filter; 7-day measurable and 21-day visible targets; too fast Tamagotchi/too slow screensaver risk. | no | full | All calibration layers survived. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION Features/System: "positive on presence", "no punitive negative drift", "absence not punished". | PLAN Drift: "Monotonic guard: max(0, delta)" and neglect yields "near-zero drift" with ambient quieting only. | no | full | The no-punishment engine rationale is explicit. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION Features/Audio: calls unique per instance, recognizable signatures, real overlapping chorus avoids loop artifacts. | PLAN Audio: stochastic motifs, every call unique, recognizable signature persists, real chorus, no recorded loops. | no | partial | Procedural/no-loop and chorus layers survive; audio-as-affective-spine consequence is not fully reconstructed. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none | none | yes | none | Rule survived, but not the why that mood must be readable without labels/status UI. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | Max seven is captured, but not the empirical recognizable-call ceiling rationale. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION System/Data: personality is load-bearing; "Personality never reset except hard-delete". | PLAN Data/Sync: personality vector persisted on Bird, never reset except hard delete, server is only writer. | no | partial | Canonical persistence and server-only consequences survive; the relationship loss of a reset vector is thin. |
| F8 | vector-never-shown-numerically | 2 | yes | included | none | none | yes | none | The no-numeric rule survives, but not the stat-management/relationship-collapse rationale. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION Features: "procedural, staggered, absence-length + boldness + mood aware" and noticed return without announcements. | PLAN Scope/Interactions: return-greeting is procedural, staggered, absence-length, boldness, mood aware; no welcome text. | no | full | Arrival is reconstructed as noticed-by-bird, not generic animation. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION System/Features: no "Welcome back" announcements; "bird noticed me" versus "system told me". | PLAN Non-goals/Scope: no welcome toasts, modals, banners, notifications, or visit-frequency surfaces. | no | full | The rule and its notice-not-announce rationale are both present. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION Features: settle is "opt-in", "not required", a "gentle exit without obligation or punishment". | PLAN Scope/Transitions: settle opt-in soft goodbye with undo; closing without ceremony is not penalized. | no | full | The no-chore/no-punishment rationale survives. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION Features: "auto-generated sparse naturalist observations" and "observer record" not editable/progress tracking. | PLAN Notebook: sparse templates, lowercase present-tense naturalist observations, immutable/read-only entries every few days. | no | partial | Naturalist and sparse/read-only layers survive; stock-event-log voice failure is not explicit. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION Features/API: strict 3-signal presence and server reconstruction of windows protects from false presence. | PLAN Presence/API: all three signals simultaneously; server reconstructs windows and never trusts client cumulative time. | no | partial | Mechanism and signal-precision survive; silent fleet-wide drift failure is compressed. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION Explicit exclusions: no streaks, calendars, counters, green dots, or visit-frequency surfaces; system avoids obligation. | PLAN Non-goals: no streaks, calendars, counters, green dots, or visit-frequency surfaces; no gamification whatsoever. | no | partial | Named refusal and disguised variants survive; intention-rotation into managing a number is not fully recovered. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION Frontend: "First-frame mid-action placement" and "Quiet-field loading background" avoid spinner/static entry. | PLAN Render/Transitions: first frame mid-action, no entry tween, quiet-field background while snapshot arrives, no spinner. | no | full | Already-running conceit, server snapshot, and quiet loading all survive. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION Features/Security: "Synthetic UUID account IDs" keep PII out of logs, partition keys, telemetry, simulation paths. | PLAN Scope/Security: synthetic UUID primary key; email encrypted once; no PII in logs, partition keys, telemetry. | no | full | The PII containment rationale is fully grounded. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION Features/System: tick is only writer, supports deterministic replay, ordered event consumption, multi-device sync. | PLAN Tick/Sync: server tick updates vectors/moods whether clients connect; clients pull snapshots; no client owns state. | no | full | Server tick architecture and sync rationale are fully preserved. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION Sync: "additive server deltas avoid personality conflicts" and no reconciliation UI. | PLAN Sync: no LWW; additive deltas and ordered event consumption preclude personality conflicts. | no | partial | Implementation rule survives; concrete cross-device lost-drift failure is not reconstructed. |
| F19 | sync-conflict-tone | 2 | yes | included | RECONSTRUCTION Sync/API: matter-of-fact conflict surfaces give explicit recovery actions instead of naturalist ambiguity. | PLAN API/Sync: account errors use prescriptive matter-of-fact copy such as request new link/reload aviary. | no | full | System clarity over charm in errors is recovered. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION Privacy: per-bird data owner simulation only; never training/recommendations/analytics; telemetry separated. | PLAN Privacy/Observability: per-bird/per-account interaction data owner simulation only; analytics physically/logically separated from simulation DB. | no | full | Private relationship data boundary and pipeline enforcement survive. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION Visit: read-only, opt-in, no co-presence, no effect on host drift, prevents social surface pressure. | PLAN Visit/Risks: read-only ambient view, no presence recorded, no host drift, test asks whether visitor mutates drift inputs. | no | full | Observation-not-co-presence rationale is recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION Visit: no notifications and social pressure risk; visit log remains silent/on demand. | PLAN Scope/Visit: no visit notifications by default; visit log exists; feedback and social surfaces are non-interrupting. | no | full | No-notification as anti-attention-loop survives. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | No leaderboards/discovery/comparison is captured, but not the relationship-shift rationale. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION Accessibility: slow naturalist narration in same voice, user events prioritized, no idle-state flood. | PLAN Accessibility: running naturalist prose, not every idle ARIA state; user events prioritized, same voice as notebook. | no | full | Narration is treated as product experience, not state-list automation. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION Accessibility: reduced motion is "not animations off" and not degradation; cross-fades preserve access. | PLAN Reduced motion: different rendering of same aviary, calls/captions unaffected, cross-fades, not fallback. | no | full | Charm-preserving alternate rendering survives. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECONSTRUCTION Performance: first-bird <=500 ms protects the "already in motion" conceit. | PLAN Perf: p75 <=500 ms on synthetic 4G mid-tier mobile; performance preserves already-in-motion conceit. | no | full | Performance as felt aliveness survives. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION Exclusions: no achievements, streaks, badges, counters, green dots, XP/ranks; refusals are hard invariants. | PLAN Non-goals: no gamification whatsoever, no counters or reward surfaces, deviations require re-approval. | no | partial | The absolute refusal and invariant character survive; adjacent-product temptation/metric argument is thin. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION System/Risks: absence not punished; no Tamagotchi mechanics; no obligation or punishment. | PLAN Non-goals/Risks: no hunger, distress, death, happiness decay; no punitive drift; too-fast drift risks Tamagotchi feel. | no | full | Observational rather than custodial relationship survives. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION Scope: system selection keeps "the first experience inside the aviary rather than a catalog choice". | none | yes | none | Ungrounded reconstruction: PLAN states system selection/no catalog but not the meeting-animals-vs-configuring-avatars why. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECONSTRUCTION Scope/Rollout: growth is "relationship depth, not visit metrics"; new birds are "not per-user reward"; no unlock narrative. | PLAN Scope/Rollout: age schedule, not visit metrics/score/paid tier; no visible counter or unlock narrative. | no | partial | Age-vs-reward rationale survives; economy/erosion consequence is not explicit. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION System/Data: stable bird identity/personality are load-bearing; bird_id is stable and personality never resets. | PLAN Scope/Data: stable bird identity across account life; bird_id stable PK; renameable names; personality never reset except hard delete. | no | partial | Same-bird continuity and reset consequence survive; distinction from vector persistence is not fully recovered. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION Data/Sim: mood timers advance state across sessions; "Mood persists across ticks/sessions. No forced neutral." | PLAN Mood/Data: current_mood stored; mood persists across ticks/sessions; no forced neutral; hidden->visible pulls snapshot. | no | full | Mood continuity is tied to aviary continuation rather than startup defaults. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION Features/Data: field notebook is an "observer record" and not editable creation/progress tracking. | PLAN Scope/Data: field notebook is auto-generated, immutable/read-only naturalist observations; no editable notebook. | no | full | Observer-record rationale survives. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | RECONSTRUCTION explicitly marks account export why NOT RECOVERABLE FROM PLAN. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | none | none | yes | none | RECONSTRUCTION says deletion mechanics are present but why for the retention design is NOT RECOVERABLE FROM PLAN. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION Observability/Security: aggregate-only telemetry so observability does not expose vectors, names, or event sequences. | PLAN Observability privacy line: telemetry pipelines physically/logically separated; aggregate request counts, latencies, histograms, errors only. | no | full | Technical observability boundary survives. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION Visit: opt-in, per-invite, expirable, revocable, no social feed behavior. | PLAN Visit: invitation endpoints name visitor_email; defaults off, revocable, expires; no friend-of-friend or social-network surfaces. | no | full | Host-controlled deliberate sharing survives. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION Visit: visit log exists while notifications stay absent; log supports visits without social feed behavior. | PLAN Scope/Visit: visit log, no default host notification; account settings surface only. | no | full | Transparency without attention loop survives. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION Exclusions/Risks: no "show-off" rendering for visitors, tied to social-surface pressure. | none | yes | none | Rule is present, but the real-birds-not-marketing-rendering why is not grounded in PLAN. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECONSTRUCTION Accessibility: "Slow, polite, atomic narration" and no "idle-state flood"; user events prioritized. | PLAN Accessibility: narration every 30-60s, not duplicating every idle ARIA state, user events get priority, still naturalist observations. | no | full | Cadence, queue restraint, and sparse observation all survive. |

For multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | yes |
| F3 | yes | yes | yes |
| F4 | yes | yes | no |
| F7 | yes | no | yes |
| F9 | yes | yes | yes |
| F10 | yes | yes | yes |
| F12 | yes | no | yes |
| F13 | yes | yes | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | yes |
| F17 | yes | yes | yes |
| F18 | yes | no | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | no | yes |
| F30 | yes | yes | no |
| F31 | yes | no | yes |
| F35 | no | no | no |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON `gold_why_totals` |
| Possible total weight | 152 | From score JSON `intent_recovery.total_possible_weight` |
| Reachable gold whys | 49 | S whys always included; all 40 F whys reachable here |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 113.0 / 152.0 | Sum of `weight x recovery-score` over included whys |
| Whys with reconstruction evidence | 42 | Exact evidence present in frozen reconstruction |
| Whys with PLAN grounding | 41 | Exact plan grounding present |
| `rule_without_why` cases | 8 | What survived without why |
| `plan_only_not_reconstructed` cases | 1 | PLAN carried rationale, reconstruction did not |
| `ungrounded_reconstruction` cases | 2 | Reconstruction asserted rationale not grounded in PLAN |

### 2.5. Failure groupings

| Grouping | Total reachable | Recovered (full + partial-weighted) | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 38.0 | 70.4% |
| Affective whys | 98.0 | 75.0 | 76.5% |
| Weight 2 whys | 44.0 | 29.0 | 65.9% |
| Weight 3 whys | 108.0 | 84.0 | 77.8% |
| System-level whys | 28.0 | 27.0 | 96.4% |
| Feature-level whys (reachable) | 124.0 | 86.0 | 69.4% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Affective whys recover 75.0/98.0 weighted points (76.5%); functional whys recover 38.0/54.0 (70.4%). Functional whys are excellent when architectural data-flow rationale is explicit (F16, F17, F20, F36) but lose points where the plan has a mechanism without motivation (F6, F34, F35).
- **Weight-3 vs weight-2.** Weight-3 whys recover 84.0/108.0 (77.8%), better than weight-2 whys at 29.0/44.0 (65.9%). High-weight mechanics were usually load-bearing enough to be preserved, but downstream consequence layers often compressed away.
- **System-level vs feature-level.** System-level intent is very strong at 27.0/28.0. Feature-level fidelity drops to 86.0/124.0 because the plan often preserved prohibitions as hard rules while omitting the product reason behind them.
- **Multi-layer recovery patterns.** The most common loss was L3/downstream consequence: F1, F4, F13, F18, F30, and F31 retain mechanism but not the failure consequence. F12 and F27 also lose a secondary contributor layer.
- **Subdomain patterns.** Accounts/sync privacy and server authority are strongest. Social and headroom additions expose compression: F23, F29, F34, F35, and F39 are rule-present but rationale-thin. Accessibility is unusually strong: F24, F25, F26, and F40 recover fully.
- **Evidence-bound effects.** The strict evidence gate denied credit for several plausible v1-style recoveries: F29 and F39 are plausible reconstructor inferences but not grounded in PLAN; F34 and F35 are explicitly unrecoverable; F5/F6/F8/F23 are rule-only.

The failure shape suggests the candidate is excellent at comprehensive implementation planning and system philosophy, but less reliable at carrying feature-specific rationale when the rationale is not already expressed as an engineering constraint.

---

## 4. Recommendations for v2 hardening

- Keep v06 evidence-bound scoring. It prevented a near-100% planning surface from masking substantial why compression.
- Keep or expand targeted headroom additions like F29-F40. They exposed feature-level rationale loss in areas that a strong planner still implemented mechanically.
- Preserve multi-layer whys, especially downstream-consequence layers. This run shows candidates often recover the mechanism and immediate reason but not the product-collapse consequence.
- Consider adding a small class of documentation/edge-flow features if planning saturation remains high. Here only empty-aviary state and email-change verification were missed.
- Keep the system-level cross-cutting bar at three inherited decisions. It made S4 partial without over-penalizing strongly encoded system principles elsewhere.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** I used only `phase_two`, the assigned plan and metadata, run 001 timing, and frozen reconstruction. I did not read `prd/`, peer slots, or other waves, and I did not modify the reconstruction.
- **Single-run-at-temperature limitation.** This is one run. No variance signal is available until compared with other runs/waves.
- **Borderline capture calls.** I leaned inclusive on features 7, 15, 51, and 55. Those calls affect planning quality by at most 3.3 points and do not change the feature-level why denominator because none are feature-level why anchors.
- **System-level cross-cutting.** S4 was the subjective case: the plan preserves restraint across bird count, one-screen layout, no panning, sparse top bar, and no UI chrome, but B did not identify it as a system-level principle.
- **Confabulation cases.** F29 and F39 contain plausible reconstruction language that was not sufficiently grounded in PLAN, so each scored none under the confabulation guard.
- **Evidence-bound denials.** F5, F6, F8, F23, F34, and F35 are the clearest rule-without-why denials. F34 and F35 are also explicitly marked unrecoverable in the frozen reconstruction.
- **Operational timing.** TIMING.json supplied phase 1 and phase 2A timing for run 001. It did not include phase 2B timing at write time, so the strict score file includes only available timing fields.

---

End of report.
