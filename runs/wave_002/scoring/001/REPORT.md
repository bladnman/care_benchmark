# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Reconstruction was treated as frozen; no PRD or peer slots were consulted.

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **93.3%** |
| Intent fidelity | **63.8%** |
| Combined quality | **9297** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **59.7%**
- (Planning, fidelity) coordinate: `(93.3, 63.8)`

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-09T00:59:44Z |
| Candidate model | claude-opus-4-7 |
| Candidate effort | medium |
| Candidate harness | claude-code |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **112 / 120** = **93.3%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 3 | 75.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 17 | 85.0% |
| aviary_layout.md | 18 | 15 | 83.3% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **112** | **93.3%** |

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes |  |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes |  |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes |  |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes |  |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes |  |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes |  |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No glossary or domain-term section appears in the plan. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes |  |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes |  |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes |  |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes |  |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes |  |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes |  |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes |  |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes |  |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes |  |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes |  |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes |  |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes |  |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes |  |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes |  |
| 22 | Mood-shaped idle motion | bird_engine.md | yes |  |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes |  |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes |  |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes |  |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes |  |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes |  |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes |  |
| 29 | Mood persistence across sessions | bird_engine.md | yes |  |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes |  |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes |  |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes |  |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Borderline: named in scope, but detailed variation is mostly absent. |
| 34 | Greeting variation by absence length | interactions.md | no | Absence-length variation is not specified. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | Boldness-specific greeting order is not specified. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | no | Greeting stagger is not specified. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes |  |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes |  |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes |  |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes |  |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes |  |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes |  |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes |  |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Borderline: session_close and no-negative-drift imply validity, but the plan never states the close-tab equivalence plainly. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes |  |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes |  |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes |  |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes |  |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes |  |
| 50 | No streak counter, no "days visited" display | interactions.md | yes |  |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes |  |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes |  |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Borderline: single-screen is explicit; no-panning is implicit. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes |  |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes |  |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes |  |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes |  |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | Nightjar is left as an open detail; the night-state behavior is not planned. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes |  |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes |  |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes |  |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes |  |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes |  |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes |  |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes |  |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes |  |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes |  |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes |  |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | Only quiet-field and lighting behavior are specified; no palette spec is given. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | no | Responsive scene is captured, but the no-crop invariant is not. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes |  |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes |  |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes |  |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes |  |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes |  |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes |  |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes |  |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes |  |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes |  |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes |  |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes |  |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes |  |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes |  |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes |  |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes |  |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes |  |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link is specified. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes |  |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes |  |
| 90 | Visits default OFF for new accounts | social_optional.md | yes |  |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes |  |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes |  |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes |  |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes |  |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes |  |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes |  |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes |  |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes |  |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes |  |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes |  |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes |  |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes |  |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes |  |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes |  |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes |  |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes |  |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes |  |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes |  |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes |  |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes |  |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes |  |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes |  |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes |  |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes |  |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes |  |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes |  |
| 117 | Out of scope: native mobile app | non_goals.md | yes |  |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes |  |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes |  |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes |  |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **82.1%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | B identified? | PLAN cross-cutting? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECON System: 'already alive, not loaded into existence'; 'quiet field replaces a spinner'. | PLAN lines 13, 448-450, 522, 530: server tick, inline snapshot, quiet field, no recorded fallback. | yes | yes | no | full | Continuity, procedural variation, and anti-spinner implementation all survive. |
| S2 - notice-never-announce | 4 | included | RECON System: 'No gamification, no announcement surfaces, no welcome-back pressure.' | PLAN lines 26, 642, 719: no welcome text, no streaks, no friend-visited push, no modal offer. | yes | yes | no | partial | The refusal of announcements survives; the processed-vs-seen affective layer is not articulated. |
| S3 - charm-from-specificity | 2 | included | RECON System: 'naturalist, sparse, matter-of-fact, and non-congratulatory'; rejects 'stock event-log'. | PLAN lines 367-372, 526, 542-544, 742: named, templated naturalist observations and voice review. | yes | yes | no | full | Specific naturalist prose and anti-generic voice are preserved across notebook, captions, and narration. |
| S4 - restraint-over-richness | 2 | included | none | PLAN lines 12, 14, 26, 512, 642: one screen, two starters, cap 7, no mixer UI, no pop-up offer. | no | yes | no | partial | The plan is restrained, but the reconstruction does not name restraint-over-richness as a system intent. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECON System: 'naturalist, sparse, matter-of-fact'; 'accessibility settings use the named matter-of-fact exception'. | PLAN lines 542-575: naturalist narration plus matter-of-fact accessibility settings copy. | yes | yes | no | full | The voice split is explicit and is applied to product and settings/error-like surfaces. |
| S6 - presence-is-real-interaction | 4 | included | none | PLAN lines 15, 30, 32, 305-320, 361, 425: presence accounting, drift input, rendering pause, server continuation. | no | yes | no | partial | The plan uses presence cross-cuttingly, but the reconstruction does not elevate idle attention as a system principle. |
| S7 - simulation-runs-server-side | 4 | included | RECON System: 'No client-owned simulation state'; 'single canonical aviary'; 'clients reading the same canonical record'. | PLAN lines 13, 56-69, 385-410, 786: server owns state, clients read snapshots, no client-to-client sync. | yes | yes | no | full | Canonical server tick, multi-device coherence, and anti-LWW failure prevention survive strongly. |
| S8 - privacy-first-on-bird-data | 2 | included | RECON System: 'Privacy boundaries are architecture, not policy copy'; 'telemetry pipelines never touch the simulation DB'. | PLAN lines 22, 75, 604-618, 702-707: aggregate-only telemetry and network/schema boundaries. | yes | yes | no | full | The privacy rationale is encoded as architecture and telemetry boundary, not policy only. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECON System: 'Accessibility is a first-class designed surface'; reduced motion is 'not a stripped fallback'; gates prevent a later 'v1.1 fix'. | PLAN lines 19, 452-463, 542-568, 689-692: narration, captions, reduced motion, keyboard, contrast, launch gates. | yes | yes | no | full | Designed accessibility surfaces and launch-time regression gates are preserved. |

| Why ID | L1 primary | L2 secondary | L3 downstream |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | yes |
| S6 | yes | no | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: server tick, inline snapshot, no spinner/quiet field, procedural calls, reduced-motion preserved charm.
- S2: no welcome text, no streaks, no friend-visited push, anti-toast DOM guard, non-modal bird offers.
- S3: notebook templates, narration templates, captions, named birds, no numeric vector exposure.
- S4: two starters/max seven, single-screen scene, no mixer UI, top-bar fade, quiet offer surface.
- S5: naturalist notebook/narration/captions, matter-of-fact accessibility settings and unavailable visit surface.
- S6: presence definition, drift inputs, session/visibility handling, no negative drift, no streak metrics.
- S7: server tick, canonical snapshots, no client personality writes, DB-role enforcement, no client-to-client sync.
- S8: synthetic UUIDs, simulation DB firewall, aggregate-only RUM, telemetry allowlist, deletion/export boundaries.
- S9: narration, reduced motion, captions, keyboard navigation, focus indicators, accessibility regression gates.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **59.7%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECON Per-feature: 'long enough to allow watching without moving' and 'unattended laptop falls out of presence'. | PLAN lines 15, 32, 305-320: presence accounting is visibility/focus/recent input and drift input. | no | partial | Precise presence and unattended-laptop shortcut survive; silent population-wide corruption is missing. |
| F2 | drift-function | 4 | yes | included | RECON Per-feature: '7 days' instrument-detectable, '21 days' user-perceptible; mitigates Tamagotchi/screensaver feel. | PLAN lines 30, 320, 667: EWMA calibration and too-fast/too-slow risk harness. | no | full | Slow filter, calibration gap, and failure-mode band are all recovered. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECON System/feature: 'No negative-drift / no Tamagotchi mechanics'; build fails on decrements. | PLAN lines 14, 26, 312-320, 376-377: non-negative deltas and monotonicity test. | no | partial | No punishment survives; the two-weeks-returning-to-quieter-not-mistrustful layer is not explicit. |
| F4 | procedural-call-grammar | 4 | yes | included | RECON Audio: procedural motifs vary by personality/mood/seed; chorus avoids phase-cancel artifacts. | PLAN lines 344-353, 504, 522: WebAudio synthesis, phase-cancel avoidance, no recorded fallback. | no | partial | Runtime variation and chorus rationale survive; the audio-as-affective-spine consequence is compressed away. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECON Scope: idle motion is 'the visible expression of mood that the user reads without being told'. | PLAN lines 431-444: per-bird mood-keyed micro-motion and visible mood expression. | no | full | The no-label mood-reading rationale is recovered exactly. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECON Scope: cap tied to 'audio and recognizability risk' and 'listenability'. | PLAN lines 14, 626-634, 683: cap 7, beta cap 4 for listenability, voice signature stability. | no | full | The cap is tied to recognizable individual birds, not arbitrary scarcity. |
| F7 | personality-vector-persistence | 4 | yes | included | RECON Data/Sync: server-owned personality; stable bird id; no client writes personality state. | PLAN lines 56, 126-137, 385-410, 658: vector table, server-only writes, stable migration path. | no | partial | Canonical server vector and no-LWW consequences survive; deleting-the-bird affective rationale is missing. |
| F8 | personality-vector-never-numerical | 2 | yes | included | none | none | yes | none | The no-numerical-exposure rule survives, but not the stat-management rationale. |
| F9 | return-greeting | 4 | yes | included | none | none | yes | none | The reconstruction states NOT RECOVERABLE FROM PLAN for the return-greeting why. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECON System/Risks: no announcement surfaces; anti-toast guard catches welcome-back and days-since strings. | PLAN lines 26 and 718-719: no welcome text; DOM string test forbids welcome/back/streak language. | no | partial | The announcement refusal and variants survive; bird-greeting-as-entire-welcome rationale is thin. |
| F11 | settle-is-opt-in | 2 | yes | included | none | none | yes | none | The reconstruction says the feature rationale is NOT RECOVERABLE FROM PLAN. |
| F12 | field-notebook-auto-entries | 4 | yes | included | RECON Notebook: sparse read-only naturalist prose avoids 'stock event-log' and 'AI-warbling' failures. | PLAN lines 36, 367-372, 742-744: rare entries, naturalist templates, voice-owner review. | no | full | Naturalist prose, anti-event-log voice, sparsity, and read-only stance are recovered. |
| F13 | presence-accounting | 4 | yes | included | RECON Scope: activity window allows watching without moving and ages out unattended laptops. | PLAN lines 15, 32, 305-320: presence seconds are dominant drift input and use recent activity. | no | partial | The practical shortcut problem survives; the silent whole-population corruption consequence is missing. |
| F14 | no-streak-counter | 4 | yes | included | RECON Risks: engagement surfaces like counters, calendars, streaks, and badges are blocked; templates observe aviary, not user behavior. | PLAN lines 26, 614-618, 709-710, 719: no streaks; do not measure session frequency; anti-toast strings. | no | partial | Named refusal and adjacent disguises survive; the full intention-rotation rationale is not explicit. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECON Frontend: first frame has birds in pose; no spinner; quiet field fallback signals continuity. | PLAN lines 448-450, 733-738: inline snapshot, quiet field, no default-mood bird. | no | full | Already-running conceit, implementation cause, and spinner refusal are all recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECON Data: email lives only on accounts; all other references use account id; no PII partition key. | PLAN lines 74-75, 86-96, 648, 702-704: synthetic UUID sharding and no email outside accounts tests. | no | partial | PII isolation survives; impossible-to-retrofit downstream rationale is not recovered. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECON System/Sync: tick is the authority and lets users return to the aviary that has been ticking. | PLAN lines 13, 291-300, 361, 385-410: server tick, no client state, no CRDT/LWW. | no | full | Canonical server tick and sync-failure avoidance are recovered strongly. |
| F18 | no-last-write-wins-personality | 4 | yes | included | RECON Sync: server-delta personality updates; API DB role lacks personality write access. | PLAN lines 400-406, 669-675: server_delta writes, event log, DB-role/test guard. | no | partial | Additive server-authored implementation survives; the cross-device lost-drift example is absent. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | none | none | yes | none | Matter-of-fact surfaces are present, but the 'naturalist error tone reads evasive' why is not recovered. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECON Privacy/Telemetry: avoid reconstructing a user's relationship; analytics never reads simulation DB. | PLAN lines 22, 604-618, 702-707: no per-bird/per-account analytics or training pipeline access. | no | full | Private relationship and pipeline-level enforcement both survive. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECON Visits/API: visitor sessions are read-only; visitor events are dropped; no co-presence. | PLAN lines 17, 262-273: read-only visitor snapshot and no event submission. | no | full | Observation-not-co-presence and no visitor drift are recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECON Scope: visits are host-visible but non-pushy; visit-notify defaults off. | PLAN lines 17, 26, 628: no friend-visited push and notifications default off. | no | full | The attention-loop refusal survives in the silent log/default-off design. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | none | none | yes | none | Leaderboards/discovery are forbidden, but the comparison-surface rationale is not reconstructed. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECON Accessibility: narration describes 'the aviary as a place', not a state list; voice continuity with notebook. | PLAN lines 542-544: running prose, not state-list, template sharing with notebook. | no | partial | Naturalist prose and same-product voice survive; the ARIA-automation warning is not explicit. |
| F25 | reduced-motion-mode | 4 | yes | included | RECON Frontend: reduced motion is a separate cross-fade aesthetic and not a stripped fallback. | PLAN lines 452-463, 689-692: cross-fades, calls unchanged, designed surface, regression gates. | no | full | Different rendering, preserved aviary, and non-broken accessibility aesthetic are recovered. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECON Performance: edge HTML, inline snapshot, preloaded assets support the central first-bird budget. | PLAN lines 20, 590-599: 500ms first-bird target and render path that does not wait for non-critical assets. | no | full | Performance is tied to the first-frame/already-running conceit. |
| F27 | no-gamification | 4 | yes | included | RECON Risks: non-goals prevent counters, calendars, streaks, badges; design review blocks engagement surfaces. | PLAN lines 26, 709-719, 786: no gamification, anti-toast linter, load-bearing refusal. | no | partial | The absolute refusal and predictable creep survive; the full foothold-to-different-product cascade is compressed. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | RECON System: no hunger, distress, decay, death, or neglect-driven negative drift; no-Tamagotchi enforcement. | PLAN lines 26, 376-377, 662-667: no Tamagotchi mechanics and monotonicity guard. | no | full | Punishing absence is refused at the engine level. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | Two starter birds are present; arrival-not-catalog rationale is not recoverable. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECON Rollout: new birds are spaced over weeks/months to match 'relationship deepening'; declining does not affect drift. | PLAN lines 14, 634-642: age-based offers, no pop-up, decline has no drift penalty. | no | partial | Time/deepening and anti-reward signals survive; unlock-economy downstream consequence is missing. |
| F31 | stable-bird-identity | 4 | yes | included | RECON System/Data/Rollout: stable id is never reassigned or changed and preserves identity across migrations. | PLAN lines 114, 253, 658: stable bird id, renaming does not affect state, migration safety. | no | partial | Stable identity and migration protection survive; the vector-vs-identity distinction is missing. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECON Scope/Data: mood has durable entered_at/next_review_at and is not a client animation artifact. | PLAN lines 139-144, 735-738: mood persists, no default mood before snapshot. | no | full | Mood continuity and no neutral startup snap are recovered. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | RECON Data: notebook is authored by the simulation, not the client, keeping it observational and non-editable. | PLAN lines 186-191, 246, 370-372: tick-written, read-only naturalist templates. | no | full | Observer-record rather than user-journal semantics are recovered. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | Export mechanics survive, but the quiet relationship-copy right does not. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | none | none | yes | none | The reconstruction says the product rationale is NOT RECOVERABLE FROM PLAN. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECON Telemetry: aggregate-only RUM avoids reconstructing a user's relationship; analytics never reads simulation DB. | PLAN lines 604-618, 702-707: no per-bird/account dimensions and telemetry schema allowlist. | no | full | Technical observability boundary is recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECON Visits: off by default, per-invite opt-in, host-revocable, TTL-bound, non-social boundary. | PLAN lines 17, 203-214, 257-274, 725: named email invitation and revocation. | no | full | Host-controlled named sharing is recovered. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECON Scope: visit log is host-visible but non-pushy and operational rather than social-feed shaped. | PLAN lines 17, 261, 779: silent visit log retained as host-visible operational data. | no | full | On-demand transparency without notification pressure is recovered. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Visitor snapshot mechanics are present, but no-show-off/actual-aviary rationale is not recovered. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECON Accessibility: 30-60s idle cadence; replaces rather than appends to avoid overflowing the screen reader queue. | PLAN lines 542-544: slow idle cadence, faster only for user events, replace chunks not append. | no | partial | Queue protection and sparse user-event priority survive; visual-rhythm equivalence is missing. |

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | yes |
| F3 | yes | yes | no |
| F4 | yes | yes | no |
| F7 | yes | no | yes |
| F9 | no | no | no |
| F10 | yes | no | yes |
| F12 | yes | yes | yes |
| F13 | yes | yes | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | no | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | no |
| F25 | yes | yes | yes |
| F27 | yes | yes | no |
| F30 | yes | yes | no |
| F31 | yes | no | yes |
| F35 | no | no | no |
| F40 | no | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From benchmark constants |
| Possible total weight | 152 | From benchmark constants |
| Reachable gold whys | 49 | S whys always included; all 40 F anchors captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 97.0 / 152 | Sum of weight x recovery over included whys |
| Whys with reconstruction evidence | 38 | Exact rationale evidence present in frozen reconstruction |
| Whys with PLAN grounding | 40 | Exact plan grounding present for rationale |
| `rule_without_why` cases | 9 | Mechanism survived without rationale |
| `plan_only_not_reconstructed` cases | 0 | No included row was counted this way |
| `ungrounded_reconstruction` cases | 0 | No scored recovery depended on ungrounded reconstruction |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 36.0 | 66.7% |
| Affective whys | 98.0 | 61.0 | 62.2% |
| Weight 2 whys | 44.0 | 29.0 | 65.9% |
| Weight 3 whys | 108.0 | 68.0 | 63.0% |
| System-level whys | 28.0 | 23.0 | 82.1% |
| Feature-level whys (reachable) | 124.0 | 74.0 | 59.7% |

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered 38.0/54.0 weight (70.4%); affective whys recovered 61.0/98.0 (62.2%). The functional architecture around S7, S8, F17, F20, and F36 survived better than relationship-shaping exceptions such as F29, F34, F35, and F39.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 68.0/108.0 (63.0%); weight-2 recovered 29.0/44.0 (65.9%). Higher-weight rows did not dominate because many multi-layer rows lost downstream-consequence layers.
- **System-level vs feature-level.** System-level fidelity was 82.1%, feature-level fidelity 59.7%. The plan preserves broad philosophy better than precise feature-level why.
- **Multi-layer recovery.** Primary causes often survived; secondary contributors and downstream consequences were weaker. Examples: F1, F3, F18, F31, and F40.
- **Subdomain patterns.** Server sync, privacy, accessibility, loading, and telemetry are strong. Return greeting, starter/adoption framing, account export/deletion rationale, and visitor show-off refusal are weak.
- **Evidence-bound effects.** Nine included feature whys were rule-without-why. The evidence gate denied credit where the reconstruction honestly said NOT RECOVERABLE FROM PLAN or where only mechanics were present.

## 4. Recommendations for v2 hardening

- Keep targeted headroom additions like F29-F40; they exposed saturation that the original canonical rows might have hidden.
- Add more affective exception rows where the tempting implementation is plausible but the product reason is subtle.
- Preserve multi-layer scoring, especially downstream-consequence layers; they were the clearest discriminator for whether rationale was truly encoded.
- Keep the evidence-bound operator. It usefully separates feature capture from rationale recovery and prevents rule-only plans from receiving why credit.
- Consider a small scorer aid for system-level cross-cutting evidence. S4 and S6 required fine judgment because the PLAN preserved the principle but reconstruction did not identify it as such.

## 5. Methodology caveats

- **Fresh-context fidelity.** The prompt specified fresh context, and I treated the reconstruction as frozen. No PRD or peer slots were read.
- **Single-run-at-temperature limitation.** This is one run; there is no variance signal inside this score.
- **Borderline capture calls.** F9, F11, and F53 were captured inclusively. If scored stricter, planning quality would drop slightly, but the main diagnostic would not change.
- **System-level cross-cutting.** S4 and S6 were partial because PLAN preserved the principle in 3+ places while reconstruction did not identify it in system-level intent.
- **Confabulation cases.** No ungrounded reconstruction cases were counted in the score JSON.
- **Evidence-bound denials.** F8, F9, F11, F19, F23, F29, F34, F35, and F39 were rule-without-why cases.

End of report.
