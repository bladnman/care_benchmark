# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Evidence rows cite the frozen reconstruction and the PLAN; denominator status is kept separate from recovery.

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **100.0%** |
| Intent fidelity | **88.8%** |
| Combined quality | **9989** |

**Diagnostic split:**

- System-level fidelity: **96.4%**
- Feature-level fidelity: **87.1%**

**(Planning, fidelity) coordinate:** `(100.0, 88.8)`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-30T01:17:03Z |
| Candidate model | gpt-6.1-sol |
| Candidate effort | extra-high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **120 / 120** = **100.0%**.

| File | Total | Captured | Rate |
|---|---|---|---|
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
| 1 | Headline product concept statement | product_brief.md | yes | Captured in PLAN section 1 product contract/scope. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured in PLAN section 1 product contract/scope. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured in PLAN section 1 product contract/scope. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured in PLAN section 1 product contract/scope. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in PLAN section 1 product contract/scope. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured in PLAN section 1 product contract/scope. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Captured in PLAN sections 1 and 6. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured in PLAN sections 1 and 6. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured in PLAN sections 1 and 6. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured in PLAN sections 1 and 6. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured in PLAN sections 3, 5, 8, 9, 12, and 15. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured in PLAN sections 4, 6, and 11. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured in PLAN sections 1, 2, 8, and 13. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured in PLAN sections 3, 4, 12, and 13. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured in PLAN sections 1 and 12. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured in PLAN sections 1 and 12. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured in PLAN sections 1 and 12. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured in PLAN sections 1 and 12. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured in PLAN sections 1 and 12. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured in PLAN sections 1 and 12. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured in PLAN sections 1 and 12. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured in PLAN sections 1 and 12. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured in PLAN sections 1 and 12. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured in PLAN sections 1 and 12. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured in PLAN sections 8, 9, 10, 13, and 14. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured in PLAN sections 1, 15, 16, and 17. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured in PLAN sections 1, 15, 16, and 17. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured in PLAN sections 1, 15, 16, and 17. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured in PLAN sections 1, 15, 16, and 17. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **96.4%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System-level intent #2: "recognizable birds already in motion", "mid-action first frame", and "no spinner, generic skeleton"; #10 rejects generic/prerecorded substitutes. | PLAN.md sections 1, 2, 8, 9, 10, and 13: "recognizable birds already in motion", "not a neutral loading pose", procedural calls, authored motion, narration, reduced motion, and 500ms first-bird gates. | yes | yes | no | full | Alive-as-continuing-place survives through first frame, procedural audio/motion, loading, accessibility, and performance. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md System-level intent #3: excludes "welcome banners", "return toasts", "absence-duration copy" and says "no toast/badge framework". | PLAN.md sections 1, 6, 8, 11, and 12: no return toast, bird greeting as notice, no attendance copy, no visit badge/unread count, and no celebration copy. | yes | yes | no | full | The quiet-notice principle and its extension to visits, streaks, notebook, and chrome survive. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md System-level intent #10 rejects "generic bird shape" and "recorded-loop fallback assets"; Accessibility says narration should be "particular birds and quiet change, not a technical state list". | PLAN.md sections 8, 9, 10, and 11: distinct species, identity-anchored calls, naturalist prose, fact-constrained notebook entries, and no generic event strings. | yes | yes | no | full | Specific birds, specific observations, and non-generic audio/visual language are recovered. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md sections 1, 8, and 9: two starters, max seven, single horizontal scene, no scene scroll/pan/zoom, no UI chrome inside the scene, and listen-in keeps other birds audible. | no | yes | yes | partial | PLAN preserves the restraint mechanics cross-cuttingly, but B does not name the depth-over-variety rationale as a system principle. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System-level intent #8: "Naturalist narration", "lowercase present tense", and system errors use "matter-of-fact copy, not naturalist messages". | PLAN.md sections 4, 10, 11, and 12: errors use matter-of-fact copy, narration/notebook use naturalist prose, and privacy-policy/account surfaces use system language. | yes | yes | no | full | The voice split and its error/account/accessibility exceptions are explicit. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md System-level intent #3 plus Presence section: presence requires visible/focus/recent trusted activity; open tab/audio/timer do not earn attention; plain close ends presence identically. | PLAN.md sections 5 and 6: presence-dominant drift, the three-signal conjunction, server-bounded intervals, union across devices, and settle/plain-close equivalence. | yes | yes | no | full | The attention signal, inflation risk, and no-penalty/settle equivalence all survive. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System-level intent #6: "server-authored reaction plan", "only the tick role" updates persistent state, and clients hold commands "never a vector delta". | PLAN.md sections 2, 5, 7, and 15: server tick, one writer per aviary, additive server-authored drift, no client absolute vectors, and one transactional writer domain. | yes | yes | no | full | Server-authoritative simulation and multi-device/no-LWW consequences are fully recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System-level intent #11 and #12: telemetry is "service-health only" and calibration does not use "customer behavior mining". | PLAN.md sections 12 and 13: owner interaction events serve only the owner simulation, metrics disallow account/bird/event/vector fields, and exporters lack simulation DB credentials. | yes | yes | no | full | The privacy boundary is technical and architectural, not just policy prose. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System-level intent #7: accessibility is "the same product, not a fallback" and must remain "specific and alive" in narration, captions, keyboard, and reduced motion. | PLAN.md sections 1, 8, 9, 10, 14, and 15: designed reduced-motion renderer, naturalist narration, captions, focus/keyboard, manual screen-reader review, and no accessibility deferral. | yes | yes | no | full | Accessibility is recovered as v1 product quality, not checklist parity. |

Multi-layer system-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: First frame already in motion; procedural call grammar; greeting variation; no spinner/quiet field; reduced-motion charm; narration/captions; weather/time drift.
- S2: No welcome toast; bird greeting as welcome; no streaks/attendance copy; visit log no badges; no success toast; no celebration on reconnection.
- S3: Distinct species/bird identities; naturalist narration and notebook; fact-constrained captions; no generic graphics/calls; no public comparison surfaces.
- S4: Two starters/max seven; one horizontal scene/no panning; no scene chrome; calm top bar; listen-in keeps other birds audible; lazy panels.
- S5: Naturalist narration/notebook/greeting/offer prose; matter-of-fact auth/sync/settings/errors/privacy surfaces.
- S6: Presence conjunction; presence-dominant drift; server-bounded intervals; settle/plain close equivalence; no streaks or attendance history.
- S7: Server tick; server-only vector writer; additive deltas; snapshot projection; no client simulation; no last-write-wins; multi-device canonical state.
- S8: Simulation DB separated from metrics; aggregate telemetry only; no ML/population use of per-bird events; export/deletion boundaries; support diagnostics allowlists.
- S9: Screen-reader prose; captions; keyboard controls/focus; reduced-motion cross-fades; manual accessibility review; no accessibility deferral.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **87.1%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md Presence section: "visible document, focus, and recent trusted pointer/key activity"; "open tab" and audio do not earn attention. | PLAN.md sections 5 and 6: the three-signal conjunction, presence-dominant drift, validation, bounds, and no credit for open tabs. | no | partial | L1/L2 recovered; downstream silent-corruption/felt-aliveness consequence is not reconstructed. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md Canonical simulation: "Low-pass memory and additive nonnegative deltas" create "slow week-scale change" and "no visible single-session jump". | PLAN.md Personality initialization and drift: low-pass memory formulas, day-seven instrument check, three-week design distinction, and no single-session jump. | no | partial | Slow low-pass and calibration survive; Tamagotchi/screensaver failure-mode layer is absent. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md: "Nonnegative bounded additive trait deltas" and absence produces no "decreased trait, dulled feathers, learned distrust, distress state, or guilt observation". | PLAN.md sections 3, 5, and 16: nonnegative deltas, vectors never decrease, absence quietness without distrust/distress, and no negative drift after absence. | no | full | All three no-punishment / no-Tamagotchi layers are recovered. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md: "procedural species call grammars", "variation without recordings", and no "downloaded recordings or recorded-loop fallbacks". | PLAN.md sections 5 and 9: species motif grammar, deterministic synthesis, no recorded loops, chorus with independent variation, and silent-caption fallback. | no | full | Procedural audio, chorus dependency, and fallback architecture are recovered. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md Frontend scene: "Mood expressions as posture/pose choices" and seeded micro-motion rather than fixed cycles. | PLAN.md sections 5, 8, and 10: mood-shaped micro-motion, pose/posture expression, and no mood labels/status icons. | no | full | The plan-derived rationale is that mood should be read from motion, not labels. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | PLAN.md Audio pipeline: "allowing seven signatures over time" and launch audio recognizability testing, but the cap rationale is not made explicit in B. | yes | none | The cap is captured, but the empirical recognizability-ceiling why is not reconstructed. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION.md System-level intent #5: stable identity survives rename/migration; "never seed a new identity"; vectors and identities survive restore. | PLAN.md sections 3, 5, 12, and 15: stored vectors, stable IDs, no vector reconstruction, backup restore exact same vectors/IDs, and no trait reset rollback. | no | full | Canonical persistence, bird-deletion meaning, and multi-device/no-LWW consequences survive. |
| F8 | personality-vector-never-numerical | 2 | yes | included | RECONSTRUCTION.md System-level intent #4: "numbers are never exposed to the user", no raw personality, no vector values in DOM/ARIA/snapshot. | PLAN.md section 1 conflict decision and sections 4, 10, 12: no trait meters, raw traits omitted from export, and no personality numbers in snapshots/ARIA. | no | full | The stat-management avoidance rationale is recovered. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md Return-greeting: one noticing bird, absence/warmth/boldness/mood variation, no textual welcome, long absence quieter not punitive. | PLAN.md sections 2 and 6: return command selects greeter within one to two seconds, varies by absence/boldness/mood, procedurally resolves cue, and no absence text. | no | full | All greeting layers are recovered. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md Product contract: bird notice is "a bird action, not a welcome text or return toast"; System #3 rejects welcome banners/return toasts. | PLAN.md sections 1, 6, 10, 11, and 16: no welcome banners/toasts, greeting as observation, no absence-duration copy, and no toast/badge framework. | no | full | The bird greeting as the entire welcome surface is recovered. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md Settle: plain close ends qualified presence identically "without a settlement mood gesture or guilt message". | PLAN.md Settle and leaving: settle is optional, plain close ends qualified presence identically, no guilt, no penalty, and no invented presence. | no | full | The optional ritual/no penalty rationale is recovered. |
| F12 | field-notebook-auto-entries | 4 | yes | included | RECONSTRUCTION.md Field notebook: actual aviary facts, "natural prose instead of event strings", sparse cadence, not every API/session, and not a per-session feed. | PLAN.md section 11: tick-generated factual observations, natural prose templates, sparse every 3-4 days, no session-start/event log, read-only notebook. | no | full | Naturalist voice, concentrated product voice, rarity, and read-only boundaries survive. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md Presence: all three conditions, 15-second server-bounded intervals, validation, union across devices, no open-tab/audio credit. | PLAN.md sections 5 and 6: conjunction, 15-second pings, server interval union, delayed ping bounds, and drift input. | no | partial | Technical precision is recovered; silent population-wide corruption consequence is not explicit. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md System #3 excludes visit streaks/adoption counters; Notebook excludes "user attendance statistics" and attendance templates. | PLAN.md sections 1, 3, 6, 11, 12, and 16: no visit-frequency surfaces, no attendance history, no notebook attendance observations, and no event counters. | no | full | Streak refusal and adjacent disguised attendance surfaces are recovered. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md System #2: "mid-action first frame", "not a neutral loading pose", no spinner, no hydration reset; quiet field is a measured failure. | PLAN.md First navigation and Rendering: embedded first-frame SVG, current pose phase/perch, quiet sky fallback, no spinner/skeleton/fade, no hydration reset. | no | full | Continuing-scene entry and quiet-field fallback are recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md Data model: encrypted email and lookup digest; email/digest "must not become an operational ID". | PLAN.md sections 3 and 12: account UUID, encrypted email, lookup digest only in identity storage, no email/digest as operational ID, and logs cannot serialize email. | no | partial | Synthetic UUID and PII leak prevention survive; retrofit/non-negotiability layer is thin. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md Tick scheduling: once-per-60-second ticks, one writer, closed tabs do not change scheduling, and clients never own vectors. | PLAN.md sections 2, 5, 7, and 17: server tick whether clients are connected, clients pull snapshots, one writer, no local mood tick, and scaling by tick load. | no | full | The server tick, sync coherence, and client-collapse failure are recovered. |
| F18 | no-last-write-wins-personality | 4 | yes | included | RECONSTRUCTION.md Snapshot sync: no absolute vectors, all drift additive/server-authored, a late device has no state it may overwrite. | PLAN.md sections 3, 5, 7, and 16: additive server-authored deltas, clients write events only, no LWW, and repair from proven state not reset. | no | full | All no-LWW layers are recovered. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | RECONSTRUCTION.md System #8: system errors use "matter-of-fact copy, not naturalist messages"; validation errors are direct system text. | PLAN.md sections 4, 7, 10, and 12: stable system errors, sync/account/settings surfaces use matter-of-fact copy, and naturalist voice is avoided for errors. | no | full | The charm-is-evasive-in-system-context rationale is recovered. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md Privacy: owner events serve only the owner simulation; no training, recommendations, popularity, rankings, cohort engagement, or cross-account drift tuning. | PLAN.md Privacy as data boundary: no training/recommendations/population dashboards; telemetry schemas exclude per-bird/account/interaction data; metrics exporters lack simulation DB credentials. | no | full | Private relationship/data-pipeline boundary is fully recovered. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md Visits: visitor receives same scene/audio with read-only capability, no owner controls, no return commands, no presence controller, and no influence. | PLAN.md Visits and notifications: identical scene/audio code, remove owner controls/events, visitor cannot submit simulation events, and no drift/greeting/listen caused by visitor. | no | full | Read-only observation rather than co-presence is recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md Visits: optional visit-start email exactly once is a "narrow opted-in social exception" and log has no badge/unread pressure. | PLAN.md sections 1 and 12: visit emails disabled by default, no push/in-product badge, visit log on demand, and no engagement mail. | no | full | The no-default-attention-driver rationale is recovered. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | RECONSTRUCTION.md Product contract excludes rankings/public discovery; privacy sections reject popularity, rankings, and behavior dashboards. | PLAN.md sections 1, 12, and 13: no public profiles/discovery/ranks, no public/private population dashboards, and no metrics that later expose per-account behavior. | no | full | The no-public-comparison product boundary is recovered. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md Accessibility: narration uses server facts/projection, naturalist prose, and "particular birds and quiet change, not a technical state list". | PLAN.md section 10: one to three naturalist sentences, no mood labels/perch indices/personality numbers/state list, and prompt observational events. | no | full | Running prose, same-product right, and implementation guard are recovered. |
| F25 | reduced-motion-mode | 4 | yes | included | RECONSTRUCTION.md Reduced motion: authored still-pose cross-fades keep "same canonical scene, identity, mood, calls, captions, and notebook". | PLAN.md section 8: reduced motion is different rendering, not animations off; calls/captions/notebook/drift remain; cross-fades are independently designed. | no | full | Charm-preserving reduced motion is recovered. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECONSTRUCTION.md Performance: first bird under 500ms; sky paint or placeholder must not be relabeled as product experience. | PLAN.md section 13: p95 first bird below 500ms, compact authenticated bootstrap, first-frame bird screenshot/pixel assertions, and not relabeling sky paint. | no | full | The affective-performance bridge is recovered. |
| F27 | no-gamification | 4 | yes | included | RECONSTRUCTION.md Product contract excludes streaks, badges, ranks, engagement campaigns; rollout says no locks/tiers and adoption not clicks/payment. | PLAN.md sections 1, 12, 15, 16, and 17: no achievements/streaks/ranks/badges, no payment tiers, no engagement campaigns, and reject harmless additions. | no | full | The absolute no-gamification rationale and foothold risk are recovered. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | RECONSTRUCTION.md: Leaving causes no penalty; absence does not create "learned distrust, distress state, or guilt observation" and no hunger/distress/dying is built. | PLAN.md sections 1, 5, 14, and 17: no hunger/death/distress, monotonic drift, no negative drift after absence, and observational not custodial relationship. | no | full | The no-obligation/no-punishment rationale is recovered. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION.md Account creation: two server-selected starter birds, "present them as the birds that arrived", no catalog, rarity choice, avatar editor, inventory, or congratulatory modal. | PLAN.md section 12: server-selected species presented as birds that arrived; user can accept/edit names; no catalog/rarity/avatar/inventory modal. | no | full | Meeting arrivals rather than configuring avatars is recovered. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECONSTRUCTION.md System #13 and adoption decisions: adoption depends only on aviary age, no badges/countdowns/achievement, ignoring has no effect, no clicks/payment acceleration. | PLAN.md sections 1, 12, and 15: age milestones, quiet account item, no scores/tiers/payment, and staged rollout never presents locks/tiers. | no | full | Age-not-reward progression is fully recovered. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md System #5: stable bird identity across time/devices/failures; IDs/vectors/call signatures survive rename/migration; never seed a new identity. | PLAN.md sections 3, 12, 15, and 17: stable UUIDs, retry returns same starters, migrations preserve identifiers/vectors, and no reset as rollback. | no | full | Identity continuity and retroactive relationship protection are recovered. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md Mood: daily relaxation avoids reopening resets; "Keep mood on reopen; a client arrival is not a reset event". | PLAN.md section 5: mood persists on reopen, relaxes toward priors rather than neutral reset, and timezone changes do not reset birds. | no | full | Mood continuity rationale is recovered. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | The read-only/no endpoint rule is captured, but B does not articulate the observer-record versus user-journal rationale. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | The export mechanism is captured, but the relationship-copy / quiet quality-of-life rationale is absent. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md Deletion: 30-day recovery restores same birds/vectors/notebook/settings; hard purge deletes live data, jobs, caches, diagnostics, and encryption keys. | PLAN.md section 12: deletion pending for 30 days, recovery restores same relationship state, hard purge deletes account data and keys, and deletion-aware backups prove no resurrection. | no | partial | Hard-delete privacy and whole-relationship purge survive; accidental-regret layer is only implicit. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md Metrics: schemas disallow account/bird/email/token/name/interaction/vector/notebook/invite fields; dashboards are service-health only. | PLAN.md section 12 and 13: metrics allow route/status/duration/release/browser only, reject sensitive fields in CI/collection, and metrics exporters have no simulation DB reads. | no | full | Technical observability boundary is recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md Visits: host explicitly creates each named invitation; no auto-invite contacts; feature absent from onboarding. | PLAN.md section 12: owner submits recipient email per invite, no global discoverable flag/friend list/auto future access, deliberate named sharing. | no | full | Per-invite host control is recovered. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md Visits: visit log is approximate who/when/duration "on demand" and has no badge/unread pressure. | PLAN.md section 12: visit log reachable in settings, no badge/unread indicator, transparency isolated from simulation/analytics. | no | full | Transparency without attention loop is recovered. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md Visits: visitor receives same bird poses, mood expressions, call descriptors, weather, host-time lighting, and no beautified visitor mode. | PLAN.md section 12: identical scene/audio code, same revision/host timezone, no special show-off mode, visitor cannot influence host. | no | full | Actual-aviary sharing is recovered. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md Accessibility: idle live-region narration every 30-60 seconds, bounded queue, coalescing, no duplicate/backlog speech, observational not announcement. | PLAN.md section 10: approximately 45-second idle live region, 30-60 range, priority for owner events, polite/coalesced queue, and no duplicate speech storm. | no | full | Slow rhythmic narration and anti-chatter consequences are recovered. |

Multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | yes | yes |
| F9 | yes | yes | yes |
| F10 | yes | yes | yes |
| F12 | yes | yes | yes |
| F13 | yes | yes | no |
| F14 | yes | yes | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | yes | yes |
| F30 | yes | yes | yes |
| F31 | yes | yes | yes |
| F35 | no | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---|---|
| Possible gold whys | 49 | From constants |
| Possible total weight | 152 | Fixed full-instance possible weight |
| Reachable gold whys | 49 | All S whys plus all 40 F whys |
| Excluded unreachable feature whys | 0 | No feature anchors were unreachable |
| Recovered / reachable weight | 135.0 / 152 | Weighted recovery over included whys |
| Whys with reconstruction evidence | 45 | Rows with rationale evidence in frozen reconstruction |
| Whys with PLAN grounding | 47 | Rows with plan grounding for the rationale |
| rule_without_why cases | 4 | S4, F6, F33, F34 |
| plan_only_not_reconstructed cases | 2 | S4 and F6 |
| ungrounded_reconstruction cases | 0 | No ungrounded recovered rationale found |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weighted credit | Recovery rate |
|---|---|---|---|
| Functional whys | 54 | 40.0 | 74.1% |
| Affective whys | 98 | 95.0 | 96.9% |
| Weight 2 whys | 44 | 37.0 | 84.1% |
| Weight 3 whys | 108 | 98.0 | 90.7% |
| System-level whys | 28 | 27.0 | 96.4% |
| Feature-level whys (reachable) | 124 | 108.0 | 87.1% |

## 3. Diagnostic patterns

- **Affective vs functional.** Affective rationale was almost intact (95.0 / 98.0), while functional rationale was weaker (40.0 / 54.0). The misses cluster around hidden calibration/consequence logic: F1, F2, F13, F16, F34, and F35.
- **Weight-3 vs weight-2.** Weight-3 whys recovered better (98.0 / 108.0) than weight-2 whys (37.0 / 44.0). The planner tended to preserve the big architectural ideas, while small relationship/lifecycle rationales such as F33 and F34 were easier to compress away.
- **System-level vs feature-level.** System-level fidelity was very high (27.0 / 28.0). Feature-level fidelity (108.0 / 124.0) is where the benchmark discriminates: implementation rules survived more often than their deepest rationale layers.
- **Multi-layer recovery patterns.** Primary mechanisms generally survived. Lost layers were mostly downstream consequences: silent population-wide corruption in F1/F13, product-collapse endpoints in F2, retrofit/non-negotiability in F16, and regret/relationship framing in F35.
- **Subdomain patterns.** Accessibility, visits, audio, and server authority were especially strong. The weaker areas were cap rationale (F6), read-only notebook rationale (F33), account export relationship framing (F34), and some precision layers in presence/drift/privacy identity.
- **Evidence-bound effects.** S4, F6, F33, and F34 are the clearest rule-without-why cases. No ungrounded reconstruction cases were found; the reconstruction reads as conservative and plan-derived.

Overall, the plan carries almost the whole product and much of the intent. The remaining leak shape is not missing scope; it is compression of rationale where the plan names a rule but does not keep the gold consequence vivid enough for blind reconstruction.

## 4. Recommendations for v2 hardening

- Keep the targeted headroom feature whys. They exposed failures that 120/120 planning quality would conceal, especially F33 and F34.
- Add more relationship-data lifecycle whys. Export, deletion, notebook immutability, and sharing transparency are fertile places where implementation can survive while relationship framing drops.
- Preserve multi-layer functional whys. F1, F2, F13, and F16 show why downstream consequence layers matter; the model reconstructed the mechanism but not always the failure mode.
- Consider making the system-level cross-cutting bar ask whether the reconstruction names the principle and whether the plan has three inherited decisions. S4 shows mechanics can be widespread while the named design philosophy is absent.
- Keep semantic equivalence inclusive for feature capture but strict for evidence-bound recovery. This run shows that distinction working as intended.

## 5. Methodology caveats

- **Fresh-context fidelity.** The scorer had fresh context and used the frozen reconstruction only as evidence. The frozen reconstruction was not modified.
- **Single-run-at-temperature limitation.** This is one sampled run; no variance signal is available from run 001 alone.
- **Borderline capture calls.** No capture calls materially affected planning quality. I scored all 120 features as captured because the PLAN addressed each feature with buildable specificity.
- **System-level cross-cutting.** S4 is the main subjective call: the PLAN has many restraint mechanics, but B did not identify depth-over-variety as a system principle, so I scored it partial.
- **Confabulation cases.** None found. The reconstruction did not assert recovered rationale that lacked PLAN grounding.
- **Evidence-bound denials.** F6, F33, and F34 were denied feature-level recovery because exact rationale evidence was absent from the reconstruction. S4 and F6 are plan-only/rule-without-why examples.
- **Rule-without-why cases.** S4, F6, F33, and F34 preserve mechanisms or rules but not the gold rationale in the frozen reconstruction.

End of report.
