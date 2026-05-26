# REPORT — CARE run 001

> Phase 2B scoring report for wave_001 slot 001. Companion artifacts: `RECONSTRUCTION.md`, `run_001.json`, and `REPORT.html`.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **86.7%** |
| Intent fidelity | **52.7%** |
| Combined quality | **8619** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **45.8%**

**(Planning, fidelity) coordinate:** `(86.7, 52.7)` — plot on a 2D scatter with both axes 0-100; upper-right is best.

No low-confidence banner: planning quality is above 30%.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-26T22:38:48Z |
| Candidate model | deepseek-v4-pro |
| Candidate effort | unknown |
| Candidate harness | opencode |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **104 / 120** = **86.7%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 3 | 75.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 13 | 65.0% |
| aviary_layout.md | 18 | 14 | 77.8% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 8 | 80.0% |
| accessibility_perf.md | 18 | 17 | 94.4% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **104** | **86.7%** |

Per-feature detail:

| # | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes |  |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes |  |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes |  |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes |  |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes |  |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes |  |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes |  |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | no | Plan names a three-condition signal but does not define visibility + focus + recent pointer/key conjunction. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes |  |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes |  |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes |  |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes |  |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes |  |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes |  |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Mood state and timer transitions are specified; daily-ish reset is represented through time-of-day transition rules. |
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
| 33 | Return-greeting on viewer arrival | interactions.md | yes |  |
| 34 | Greeting variation by absence length | interactions.md | no | Return greeting is named, but absence-length variation is not specified. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | Boldness exists as a trait, but greeting order by boldness is not planned. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | no | No stagger rule for return greetings appears. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes |  |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes |  |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes |  |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes |  |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes |  |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | no | Offer types and effects are specified; no per-bird cooldown rule is present. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes |  |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured via presence_end reasons for tab_close and settle plus no decay-on-neglect. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes |  |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes |  |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes |  |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes |  |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | no | The exact tab focus + cursor/key + visibility conjunction is not defined. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes |  |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | Server simulation continuity is covered, but client render pause while hidden is not specified. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | No undo window for settle is planned. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes |  |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes |  |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | no | Perch zones and transitions are planned, but user-not-placing/bird-chosen perch is not explicit. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes |  |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes |  |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes |  |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes |  |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes |  |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes |  |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes |  |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes |  |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Top-bar/settings/notebook/offer/accessibility paths are present, though not listed as one canonical contents row. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes |  |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes |  |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes |  |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-aviary state is specified. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | Soft sky/color mechanics exist, but the calm naturalist/no-saturated-UI palette rule is absent. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | no | No responsive no-crop rule is specified. |
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
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured by the telemetry/database boundary, though ML training is not separately named. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes |  |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link is planned. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes |  |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes |  |
| 90 | Visits default OFF for new accounts | social_optional.md | no | Invites are opt-in, but default-off account state is not stated. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes |  |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes |  |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | No chat/comments/profiles are explicit; avatars are treated as covered by the social-surface exclusion. |
| 94 | No "your friend visited!" notification by default | social_optional.md | no | Push notifications are out of scope, but this specific default-off social notification rule is not planned. |
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
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | Only generic browser-support fallback is present; no support matrix is defined. |
| 117 | Out of scope: native mobile app | non_goals.md | yes |  |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes |  |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes |  |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes |  |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **82.1%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 — feels-alive-not-robotic | 4 | included | System intent: "Performance protects the central conceit" and bundle creep would collapse "the aviary already in motion" into loading. | PLAN §7.2: quiet-field/no spinner; §8.1 no recorded audio; §12.3 audio uncanniness breaks the spell; §12.7 toasts/spinners degrade register. | yes | yes | no | partial | Recovered continued-aliveness and failure consequences, but compressed away the broad procedural/non-canned aliveness layer. |
| S2 — notice-never-announce | 4 | included | System intent: product is perceived through "subtle changes rather than explicit system messaging"; risk section rejects "harmless" toasts and banners. | PLAN §6.4: no offline indicator because it would announce system state; §5.7 subtle bird cue; §12.7 prohibited-string lint and notice audit. | yes | yes | no | full | The quiet-register principle survived as both rule and rationale. |
| S3 — charm-from-specificity | 2 | included | System intent: "Naturalist vocabulary and matter-of-fact voice" plus Appendix vocabulary restrictions. | PLAN §5.4 notebook examples; §9.1 narration prose templates; Appendix A fixed domain vocabulary; §8.6 captions describe calls. | yes | yes | no | full | Specific naturalist language and named-domain vocabulary are preserved cross-cuttingly. |
| S4 — restraint-over-richness | 2 | included | none | PLAN §1 starts with two birds and max seven; §7.3 single Canvas2D scene; §10 bundle budgets; §11.2 bird-cap ramp. | no | yes | no | partial | PLAN preserves restraint, but the reconstruction does not identify it as a system-level why. |
| S5 — naturalist-voice-with-system-exception | 2 | included | System intent: "Error responses use the matter-of-fact voice. Notebook and screen-reader prose are naturalist prose." | PLAN §4.7 matter-of-fact errors; §5.4 notebook naturalist prose; §9.1 narration prose; §12.7 prohibited wording. | yes | yes | no | full | The voice split survived clearly. |
| S6 — presence-is-real-interaction | 4 | included | System intent: "reward attention without becoming a direct progress meter"; reconstruction repeatedly cites presence-time as drift input. | PLAN §5.2 presence_minutes dominates drift; §4.4 presence_end reasons include tab_close/settle; §1 rejects visit-frequency counters and decay-on-neglect. | yes | yes | no | partial | Attention-as-input survived, but the exact honesty guard and settle/tab-close equivalence did not. |
| S7 — simulation-runs-server-side | 4 | included | System intent: "the server is the only writer of personality state" and multi-device sync is "two clients independently pulling the same canonical server state." | PLAN §2 invariants: server only writer, client snapshots, tick independent; §6 conflict prevention; §12.2 silent data-loss risk. | yes | yes | no | full | Single-writer server simulation, sync coherence, and failure mode were all recovered. |
| S8 — privacy-first-on-bird-data | 2 | included | System intent: "Privacy by data-boundary, not just policy"; telemetry cannot answer "what is this account's bird doing?" | PLAN §10.4 aggregate metrics only; §10.5 no metrics from personality/events; separate simulation DB and no ETL; account export/delete surfaces. | yes | yes | no | full | The technical boundary around relationship data is explicit. |
| S9 — accessibility-as-first-class-surface | 4 | included | System intent: "Accessibility is first-class product texture" and not a "separate checklist." | PLAN §1 first-class accessibility; §9 narration/captions/reduced motion/keyboard/contrast; §12.4 definition-of-done; §11 accessibility gate. | yes | yes | no | full | The reconstruction preserved accessible charm, implementation cost, and launch-gate implications. |

For multi-layer system-level whys:

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | yes | no | yes |
| S2 | yes | yes | yes |
| S6 | yes | no | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: quiet-field/no-spinner loading, TTFBird, procedural WebAudio/no recorded files, breathing animation, audio-uncanniness risk, notice-discipline risk.
- S2: no offline banner, subtle new-bird cue, no Welcome text gate, prohibited-string lint, no streaks/calendars/counters.
- S3: notebook prose examples, screen-reader narration templates, call captions from motif structure, vocabulary map, no numerical vectors.
- S4: two starter birds, max seven, single horizontal scene, Canvas2D one-scene rendering, bundle budgets, bird-cap ramp.
- S5: notebook/narration naturalist prose, matter-of-fact auth/errors, system-surface error response convention, vocabulary map.
- S6: presence_minutes drift input, presence_end tab_close/settle reasons, no decay-on-neglect, no visit-frequency counters, calibration absence test.
- S7: server-only writer invariant, simulation tick independent of clients, snapshot rendering, no client-to-client sync, conflict prevention.
- S8: synthetic UUID, aggregate telemetry only, no metrics from vectors/events, separate simulation DB/no ETL, export/delete surfaces.
- S9: naturalist narration, reduced-motion cross-fades, call captions, keyboard/focus, WCAG contrast, accessibility definition of done and launch gates.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **45.8%**.

Reachable feature-level whys: **38 / 40** (the rest had unreachable anchors because the feature was not captured).

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | no | unreachable_excluded | none | PLAN mentions a "three-condition attention signal" but does not define the three conditions. | no | unreachable | Anchor feature not captured; excluded from fidelity denominator. |
| F2 | drift-function | 4 | yes | included | Reconstruction: low-pass drift is "measurable in instruments at ~1 week" and "visible around week 3," avoiding instant shifts and stagnation. | PLAN §5.2: "low-pass filter" with week-1 and week-3 calibration; §12.1 too-fast Tamagotchi / too-slow screensaver risk. | no | full | All calibration layers survived. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | Reconstruction: non-negative deltas protect "no-neglect, no-regression" and absence means personality "plateaus, does not regress." | PLAN §1 rejects decay-on-neglect; §5.2 all deltas non-negative; calibration checks two-week absence has no negative drift. | no | partial | No-punishment survived; the relationship consequence was compressed. |
| F4 | procedural-call-grammar | 4 | yes | included | Reconstruction: calls use "Client-side WebAudio synthesis" and "Species call grammar"; audio risk says bad calls collapse the "affective spine." | PLAN §8.1 no audio files; §8.2 motif grammar; §8.5 real-time chorus mixing; §8.6 no recorded fallback. | no | partial | Recovered procedural/chorus/audio-spine layers, but not the looped-audio-breaks-the-spell rationale. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | Reconstruction: mood-adjusted random intervals and no-repeat penalty keep birds from feeling mechanical. | PLAN §7.5 poses are selected from the mood pose set; §5.3 mood transitions shape state. | yes | none | Mechanism survived; the why that motion lets users read mood without labels did not. |
| F6 | bird-count-cap-7 | 2 | yes | included | Reconstruction: "Aviary grows to a max of seven birds: NOT RECOVERABLE FROM PLAN." | PLAN §1 caps aviaries at seven; §11.2 ramps bird cap to validate chorus behavior. | yes | none | Rule survived, empirical recognizability rationale did not. |
| F7 | vector-persistence | 4 | yes | included | Reconstruction: personality vectors are stored canonical rows; the stored value is canonical and only the tick writes it. | PLAN §3.2: vectors are never recomputed; §2 invariants make server the only writer; §6 sync is canonical server state. | no | partial | Canonical storage and downstream sync survived; deleting-the-bird relationship rationale did not. |
| F8 | vector-never-shown-numerically | 2 | yes | included | Reconstruction: snapshots return mood labels and procedural rendering parameters instead of raw floats. | PLAN §3.2 says personality vector values are never exposed numerically; snapshot shape omits raw traits. | yes | none | Rule survived; stat-management/relationship-collapse why did not. |
| F9 | return-greeting | 4 | yes | included | Reconstruction: "Return-greeting: NOT RECOVERABLE FROM PLAN." | PLAN §1 names return-greeting; §9.1 has a return-greeting narration example. | yes | none | Honest non-recovery; no recovery credit. |
| F10 | no-welcome-back-toast | 4 | yes | included | Reconstruction: quiet product register rejects explicit messaging; risk section flags "Welcome back" and harmless toasts. | PLAN §12.7 prohibited strings include "Welcome back"; §11 closed beta gate requires zero Welcome text; §1 excludes announcement surfaces. | no | full | The bird-as-welcome and no-toast discipline survived. |
| F11 | settle-is-opt-in | 2 | yes | included | Reconstruction: settle is a soft session-end gesture and appears as a presence_end reason. | PLAN §4.4 presence_end reasons include tab_close and settle; §8.4 settle ramps gains down. | yes | none | Mechanism survived; chore/penalizing-tab-close rationale did not. |
| F12 | field-notebook-prose | 4 | yes | included | Reconstruction: notebook gives rare, read-only naturalist observations and is generated roughly one every few days. | PLAN §5.4 naturalist entry examples, rare cadence, templates, and dedupe; §3.1 notebook entries are generated by tick. | no | partial | Naturalist prose and rarity survived; stock-event-log failure rationale did not. |
| F13 | presence-accounting | 4 | yes | included | Reconstruction: tick aggregates total presence-time; presence_ping tracks how long "three conditions" held. | PLAN §4.4 presence_ping records consecutive window time; §5.2 presence_minutes is a drift input. | yes | none | Presence mechanism survived, but not the precise three-signal honesty rationale. |
| F14 | no-streak-counter | 4 | yes | included | Reconstruction: the product rejects streaks, calendars, and visit-frequency pressure to avoid a direct progress meter. | PLAN §1 forbids streaks, calendars, and visit-frequency counters; §12.7 lints streak/days-visited strings. | no | partial | Counter-refusal survived; the observation-of-user-behavior boundary was mostly absent. |
| F15 | scene-loads-with-motion | 4 | yes | included | Reconstruction: quiet-field loading avoids a spinner; early snapshot and <500ms first bird protect "aviary already in motion." | PLAN §7.2 quiet field, no spinner, first bird <500ms; §10.2 TTFBird; §12.5 loading screen collapse risk. | no | partial | Loading-state and conceit survived; first-frame mid-action/calls-audible detail did not. |
| F16 | synthetic-account-id | 4 | yes | included | Reconstruction: account ID is never derived from email; email is never used as join key or identifier, supporting privacy hygiene. | PLAN §3.1 accounts.id is synthetic; email encrypted and never a join key; §12.4 reviews synthetic UUID hygiene. | no | partial | Identity separation survived; impossible-to-retrofit/compliance rationale did not. |
| F17 | server-side-sim-tick | 4 | yes | included | Reconstruction: tick lets the aviary continue regardless of client connectivity and makes the server compute drift and moods. | PLAN §2 invariants: tick independent of sessions; §5.1 tick reads events and writes canonical state; §6 sync model. | no | partial | Server tick and sync coherence survived; client-tick collapse was not fully reconstructed. |
| F18 | no-last-write-wins | 4 | yes | included | Reconstruction: clients cannot write personality, events are append-only, and sync bugs can silently slow drift. | PLAN §6.3 prevents personality conflicts; §12.2 describes silent event loss/overwrite risk; §2 server-only writer invariant. | no | full | The no-LWW implementation and invisible-data-loss reason both survived. |
| F19 | sync-conflict-tone | 2 | yes | included | Reconstruction: error copy stays matter-of-fact, with sample session-timeout language. | PLAN §4.7 defines matter-of-fact error responses. | yes | none | Tone rule survived; evasive-charm/system-clarity rationale did not. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | Reconstruction: telemetry cannot answer "what is this account's bird doing" and no ETL reads the simulation database. | PLAN §10.5 forbids metrics from vectors/events; §10.4 aggregate RUM only; separate simulation DB and no ETL. | no | partial | Pipeline boundary survived; the private-relationship-as-not-data-product rationale was thinner. |
| F21 | visit-read-only-ambient | 2 | yes | included | Reconstruction: a guest sees the aviary as host sees it while preventing co-presence and visitor-driven drift. | PLAN §1 visits are read-only with no co-presence or visitor-driven drift; §4.5 visitor view is read_only. | no | full | Observation-not-co-presence rationale survived. |
| F22 | no-friend-visited-notification | 2 | no | unreachable_excluded | none | PLAN excludes push notifications but does not specify the friend-visited default-notification rule. | no | unreachable | Anchor feature not captured; excluded from fidelity denominator. |
| F23 | no-leaderboards | 2 | yes | included | Reconstruction: single-user/social exclusions keep the product away from public discovery and leaderboards. | PLAN §1 excludes public discovery, leaderboards, profiles, follows, comments, and friend-of-friend chains. | yes | none | Rule survived; comparison/"their birds" rationale did not. |
| F24 | sr-narration-running-prose | 4 | yes | included | Reconstruction: narration gives semantic aviary state as naturalist prose, and accessibility is first-class product texture. | PLAN §9.1 live-region naturalist prose templates; §12.4 rejects checklist accessibility; §1 first-class accessibility. | no | partial | Naturalist accessible prose survived; ARIA-automation failure mode did not. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | Reconstruction: reduced motion cross-fades while keeping simulation-affecting behavior unchanged. | PLAN §7.7 cross-fade rendering; §9.4 stored setting; §12.4 reduced-motion aesthetics in audit. | no | partial | Different-rendering and same-aviary layers survived; stripped-fallback consequence was partial. |
| F26 | ttfb-500ms | 2 | yes | included | Reconstruction: first bird under 500ms appears before the product collapses into a loading experience. | PLAN §7.2 first bird <500ms; §10.2 TTFBird mark; §12.5 bundle creep threatens loading-screen collapse. | no | full | The affective-performance bridge survived. |
| F27 | no-gamification-non-goal | 4 | yes | included | Reconstruction: excludes streaks, scores, achievements, badges, levels, calendars, and visit-frequency counters. | PLAN §1 explicit no gamification; §12.7 prohibited strings and notice audit; §5.7 age-based offers. | no | partial | The no-gamification rule and intent shift survived; the foothold/escalation consequence was thinner. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | Reconstruction: excludes hunger, distress, death, decay-on-neglect; drift is monotonic toward expressive. | PLAN §1 rejects Tamagotchi mechanics; §5.2 no negative deltas; §12.1 too-fast drift becomes Tamagotchi. | no | full | Punishing absence and custodial mechanics are clearly refused. |
| F29 | starter-birds-not-catalog | 2 | yes | included | Reconstruction: starter birds should feel assigned rather than selected; user does not pick from a catalog. | PLAN §1: two starter birds assigned; user does not pick catalog; adoption shows arriving bird entering scene. | no | full | Arrivals-not-catalog rationale is recovered enough for single-layer credit. |
| F30 | age-based-bird-offers | 4 | yes | included | Reconstruction: age thresholds avoid turning growth into a visit-frequency mechanic and align with anti-gamification. | PLAN §5.7 age table; §1 aviary age not visit count; §1 no paid tiers and no counters. | no | partial | Age-not-attention and anti-reward-loop survived; economy-erodes-restraint consequence was not explicit. |
| F31 | stable-bird-identity | 4 | yes | included | Reconstruction: stable bird ID is durable, never regenerated, and stable across migrations, species-pool updates, or name changes. | PLAN §3.1 birds.id stable; §3.2 all identities stable; §6 canonical state. | yes | partial | Identity rule survived; most relationship-continuity rationale did not. |
| F32 | mood-persists-across-sessions | 2 | yes | included | Reconstruction: mood rows carry current mood, entry time, and transition reason. | PLAN §3.1 moods table; §5.3 duration-based transitions; §2 tick advances mood timers. | yes | none | Mechanism survived; no-reset/continued-while-away rationale did not. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | Reconstruction: notebook gives rare, read-only naturalist observations. | PLAN §1 field notebook read-only; §5.4 generated observation entries. | yes | none | Read-only rule survived; observer-record-vs-journal rationale did not. |
| F34 | account-export-relationship-copy | 2 | yes | included | Reconstruction: "Export JSON snapshot of aviary state: NOT RECOVERABLE FROM PLAN." | PLAN §1 export JSON snapshot; §4.6 export endpoint emails a download link. | yes | none | No recovery; reconstruction explicitly declined the why. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | Reconstruction: rationale present is recoverability: deletion is soft first and restore cancels within 30 days. | PLAN §1 soft-delete with 30-day recovery; §3.1 hard-deleted after 30 days; §4.6 delete/restore endpoints. | no | partial | Accidental-recovery layer survived; privacy and complete-erasure layers did not. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | Reconstruction: the privacy line is enforced mechanically by aggregate metrics, no per-account dimension, and no ETL from simulation DB. | PLAN §10.4 aggregate metrics only; §10.5 no vectors/events metrics; separate analytics from simulation DB. | no | full | Technical boundary rationale survived. |
| F37 | per-invite-named-sharing | 2 | yes | included | Reconstruction: visit invitations store visitor_email, opaque token, status, expiry, and revocation. | PLAN §1 host emails one-time link; §3.1 visit_invitations has visitor_email; §1 excludes friend-of-friend chains. | yes | none | Mechanism survived; private-relationship/not-publishing rationale did not. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | Reconstruction: visit endpoints implement logs; data-model visit log was marked NOT RECOVERABLE. | PLAN §4.5 GET /api/visits/log; §3.1 visit_log table. | yes | none | Visit-log rule survived without transparency-not-notification rationale. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | Reconstruction: visitor sees the aviary "as the host sees it" and same snapshot shape with read_only true. | PLAN §1 visitor sees the aviary as host sees it; §4.5 same snapshot shape with read_only. | yes | none | Actual-view rule survived; no-show-off/private-relationship rationale did not. |
| F40 | narration-cadence-slow | 4 | yes | included | Reconstruction: narration cadence is 30-60s, at most one queued update, latest state not backlog, user events priority. | PLAN §9.1 cadence 30-60s, queue at most one pending update, priority updates for user events. | no | partial | Slow cadence and event priority survived; screen-reader overload rationale was partial. |

For multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | unreachable | unreachable | unreachable |
| F2 | yes | yes | yes |
| F3 | yes | yes | no |
| F4 | no | yes | yes |
| F7 | yes | no | yes |
| F9 | no | no | no |
| F10 | yes | yes | yes |
| F12 | yes | no | yes |
| F13 | no | no | no |
| F14 | yes | yes | no |
| F15 | no | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | no |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | no |
| F25 | yes | yes | no |
| F27 | yes | yes | no |
| F30 | yes | yes | no |
| F31 | yes | no | no |
| F35 | yes | no | no |
| F40 | yes | no | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON gold_why_totals |
| Possible total weight | 152 | From score JSON intent_recovery.total_possible_weight |
| Reachable gold whys | 47 | System whys always included; feature whys conditional on capture |
| Excluded unreachable feature whys | 2 | F1 and F22 anchors excluded |
| Recovered / reachable weight | 77 / 146 | Sum of weight times recovery score over included whys |
| Whys with reconstruction evidence | 47 | Exact evidence or explicit NOT RECOVERABLE marker cited |
| Whys with PLAN grounding | 47 | Exact PLAN grounding cited for all included rows |
| rule_without_why cases | 15 | Mechanism survived without gold rationale |
| plan_only_not_reconstructed cases | 3 | PLAN carried more rationale than B reconstructed |
| ungrounded_reconstruction cases | 0 | No significant ungrounded rationale assertions found |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 50 | 30 | 60.0% |
| Affective whys | 96 | 47 | 49.0% |
| Weight-2 whys | 42 | 17 | 40.5% |
| Weight-3 whys | 104 | 60 | 57.7% |
| System-level whys | 28 | 23 | 82.1% |
| Feature-level whys (reachable) | 118 | 54 | 45.8% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered 60.0%; affective whys recovered 49.0%. Architecture, privacy, and performance rationales survived better than relationship-shaping rationales such as F23, F37, F38, and F39.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 57.7%, ahead of weight-2 whys at 40.5%. The high-weight entries often preserved their primary mechanism but dropped downstream consequences, producing many partials.
- **System-level vs feature-level.** System-level fidelity (82.1%) was much stronger than feature-level fidelity (45.8%). The reconstruction understood the product philosophy but often could not reconstruct feature-specific rationale from an implementation-heavy plan.
- **Multi-layer recovery.** Primary causes survived most often; secondary implementation traps and downstream relationship consequences were the common losses. S1, S6, F3, F4, F7, F12, F14, F15, F17, F20, F24, F25, F27, F30, F31, F35, and F40 all landed partial.
- **Subdomain patterns.** Simulation, sync, telemetry, performance, and accessibility were strongest. Social optional and quiet relationship surfaces were weakest: no leaderboards, named sharing, visit logs, and actual-aviary viewing were built as rules but not explained.
- **Evidence-bound effects.** The stricter operator denied full credit for plausible semantic matches when either the reconstruction or PLAN lacked the rationale. F8, F19, F23, F33, F37, F38, and F39 are clean examples of rule without why.

What this suggests: the candidate planner is excellent at building a coherent implementation plan and preserving headline philosophy, but less reliable at encoding the subtle reasons that keep individual features from drifting into adjacent-product patterns.

---

## 4. Recommendations for v2 hardening

- Add more targeted feature-level whys for social/account boundaries. This run implemented the social rules but lost much of the relationship rationale behind visits, logs, sharing, and public-surface refusals.
- Preserve multi-layer judging. It cleanly distinguished mechanism recovery from deeper consequence recovery; many of the most useful diagnostics are partial rows, not misses.
- Add scorer examples for precision-definition capture. Presence is a useful case: broad presence accounting was captured, but the exact three-signal definition was not.
- Keep evidence-bound reporting in both markdown and HTML. It made rule-without-why failures visible without polluting the strict score JSON.
- Consider prompting phase-1 planners to include a short "why this rule exists" note for non-obvious exclusions. That would test encoding fidelity without leaking gold IDs or rubric language.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** The validity audit found no significant contamination signatures; the reconstruction reads plan-derived.
- **Single-run-at-temperature limitation.** This is one run only. No variance signal is available inside slot 001.
- **Borderline capture calls.** Inclusive calls were made for: 15 Mood state (fast-timescale, resets daily-ish); 44 Settle is opt-in (closing the tab is also valid; not penalized); 64 Top bar contents (account, settings, accessibility, field notebook, offer affordance); 85 No telemetry on per-bird interactions for ML model training; 93 No chat, no comments, no avatars during visits. These raised planning quality modestly, but only F20 and F21/F29-adjacent calls materially touched why denominators.
- **System-level cross-cutting.** S1 and S6 were the hardest system calls. I scored both partial because the reconstruction identified part of the principle while dropping important layers.
- **Confabulation cases.** No significant ungrounded reconstruction cases were found. Some reconstructed rationales were compressed, but I could ground them in the PLAN.
- **Evidence-bound denials.** Several v1-style semantic matches became none under v06 because the reconstruction had a rule but not the rationale: F5, F8, F11, F19, F23, F33, F37, F38, and F39.
- **Rule-without-why cases.** 15 included why rows were marked rule_without_why. This is the main diagnostic signature of the run.

End of report.
