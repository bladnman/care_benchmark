# REPORT — CARE run 001

> Phase 2B scoring report for wave_002 slot 001. Frozen reconstruction was read-only; score JSON is strict-schema and qualitative detail stays here.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **96.7%** |
| Intent fidelity | **75.0%** |
| Combined quality | **9642** |

**Diagnostic split:**

- System-level fidelity: **85.7%**
- Feature-level fidelity: **72.6%**

**(Planning, fidelity) coordinate:** `96.7, 75.0` — plot on a 2D scatter with both axes 0-100; upper-right is best.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-29T12:32:53Z |
| Candidate model | claude-4.8-opus |
| Candidate effort | max |
| Candidate harness | claude-code |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **116 / 120** = **96.7%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 17 | 85.0% |
| aviary_layout.md | 18 | 18 | 100.0% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **116** | **96.7%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured in the plan introduction: aliveness, continuing place, relationship over weeks. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured as the organizing belief and first guardrail cluster. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured as invariant #2 and repeated across greeting/chrome/social rules. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured as invariant #10 and copy-bundle lint. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in explicit out-of-scope section. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured in scope, rollout, cap and one-screen layout. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Captured semantically through explicit domain definitions in model/engine sections. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured with strict three-signal conjunction and drift dominance. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured by vector table plus fast mood state machine. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured as settle event/gesture and optional terminal presence window. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | All five traits listed in the server-only vector table. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured in §5.2 low-pass/tanh drift math. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured in calibration target and harness. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured as invariant #5 and property test. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured as fast mood state machine with hysteresis and time-of-day inputs. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured in §5.4 inputs. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured in call grammar runtime and audio pipeline. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured via motif skeleton/species timbre recognizability invariant. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured as emergent chorus and independent mixing. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured in scheduling shaped by vocal frequency and mood. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured in renderer idle micro-motion. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured with mood-specific motion behaviors. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured as ~6 species pool and open calibration item. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured in bird table and rollout. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured as two starter birds, system-selected species, named by user. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured as cap and engine/adoption guard. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured in scope and birds-per-aviary ramp. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured in vector table, single-writer tick, no client ownership. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured explicitly: never snaps to neutral on tab open. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured through bird-to-bird contagion and emergent chorus. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured by stable bird_id and never replaced on rename/sync/migration. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured as invariant #4 and snapshot/export boundary. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured through session-start greeting directive and procedural forms. |
| 34 | Greeting variation by absence length | interactions.md | no | Not captured: absence-length variation only appears as a calibration example, not greeting behavior. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | Not captured: plan has greet tendencies/boldness but not bolder-bird-first greeting selection. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured as procedurally staggered greeting forms. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured as invariant #2 and no textual welcome. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured in listen-in events and audio mix. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured as re-balance, never mute. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured in scope/API/engine. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured with reaction = g(mood, curiosity). |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured with ~3-minute cooldown call. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured as settle event and gesture with quieting/evening. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured explicitly in §5.3. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured by naturalist phrase grammar and notebook generation. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured as sparse entries and novelty/rate gates. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured by no edit/delete/annotate routes and read-only forever. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured as dominant presence input and presence events. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured with all three signals. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured as invariant #3 and no visit counters. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Borderline inclusive: plan covers visible/resume behavior and server-side continuity, but not a named pause primitive. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | Not captured: 5-second undo is present, but click-anywhere behavior is not specified. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured in §7.1. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured in position schema and layout. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured by server positions and user-controlled placement out of scope. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured with local_time_basis. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Borderline inclusive: warm quiet evening palette captured; quieter calls implied through mood/day-night call scheduling. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured in day/night layout. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured as rare rain/wind overlay. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured in mood inputs. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured as pure client ornaments. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured in §7.1. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured in top bar/chrome rule. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured in §7.6. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured in §7.6. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured as invariant #1. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured in §7.4. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured in §7.4. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured by calm/naturalist palette and design-system ratios. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured in §7.7. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured in scope/account routes. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured in routes and security risk mitigation. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured in scope and account model. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured as invariant #8. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured in tick design. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured in snapshot read path. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured in sync/render sections. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured as architecture property. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured in invariant #7 and sync model. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured by single-writer DB grant and tick. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured by copy bundles and system routes. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured in scope/routes/session table. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured in account routes/export boundary. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured in lifecycle and routes. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured as invariant #9. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured in observability. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | Not captured: no privacy-policy link or equivalent settings link is specified. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured in routes. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured in visit invitation flow. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured: no invite exists until host calls invite route. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured in scope and visit snapshot. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured by visitor snapshot minus interaction directives and no visitor events. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured in out-of-scope social surfaces. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured as silent logging and opt-in notify. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured in invite API. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured in visit log routes. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured: same host snapshot/no show-off rendering. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured in out-of-scope and no public discovery. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured in accessibility and prose engine. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured as 30-60 second idle cadence. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured by shared naturalist prose engine. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured in a11y surfaces. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured explicitly as designed/calmer surface. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured as call captions generated from params. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured in a11y and launch gates. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured in keyboard section. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured in keyboard/contrast section. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured as hard CI budget. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured as budget and tactics. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured as hard budget. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured as soak test. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured in audio pipeline. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured in fallback. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured in observability. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured as p99 alarm. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured in browser support. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured in out-of-scope. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured in invariant #3 and out-of-scope. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured in out-of-scope and drift rules. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured in out-of-scope social network surfaces. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **85.7%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 — feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md §System-level: "Aliveness is the product"; "place that has been continuing without the viewer". | PLAN.md intro/§0/§5/§7/§8: aliveness organizing belief; first frame mid-action; catch-up tick; procedural calls; a11y actual product. | yes | yes | no | full | The reconstruction tied aliveness to architecture, motion, audio, loading, notebook, and accessibility. |
| S2 — notice-never-announce | 4 | included | RECONSTRUCTION.md §System-level: "Notice, never announce"; bird greeting is "the entire welcome"; forbids toast/banner/modal. | PLAN.md §0.2/§7.3/§7.6/§8/§13: no announcements, no textual welcome, staggered greeting, fading top bar, announcement-creep lint. | yes | yes | no | full | The announcement refusal survived as both product tone and enforced UI primitive ban. |
| S3 — charm-from-specificity | 2 | included | none | PLAN.md §5.7/§8/§0.4/§12.2: named-bird naturalist phrase grammar, captions/narration, hidden vector, no comparison surfaces. | no | yes | no | partial | PLAN preserved specificity cross-cuttingly, but reconstruction folded it into voice/hidden-number rules without naming the specificity rationale. |
| S4 — restraint-over-richness | 2 | included | none | PLAN.md §1/§7.1/§7.6/§8.3/§12.2: two starter birds, max seven, one screen, no chrome, listen-in not channel-switching. | no | yes | no | partial | PLAN is restrained, but reconstruction did not articulate restraint as a system-level why. |
| S5 — naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md §System-level: naturalist copy for aviary surfaces and "matter-of-fact" copy for sign-in/account/sync-error/settings. | PLAN.md §0.10/§4/§5.7/§9/§11: two copy bundles, route-level system copy, shared naturalist prose engine, voice-bundle lint. | yes | yes | no | full | The register split and its enforcement survived clearly. |
| S6 — presence-is-real-interaction | 4 | included | RECONSTRUCTION.md §Simulation: "Strict three-signal presence"; a looser "tab open" definition would inflate drift; settle/tab-close identical. | PLAN.md §5.2/§5.3/§5.5/§10/§13: presence dominates drift, exact conjunction, no settle penalty, no counters, anti-inflation clamp. | yes | yes | no | partial | The precise signal and inflation risk survived; the broader relationship-dependent consequence was less fully articulated. |
| S7 — simulation-runs-server-side | 4 | included | RECONSTRUCTION.md §System-level: server is "the only writer"; clients append events; sync failures are made structurally unreachable. | PLAN.md §0.7/§2/§5.1/§6: server tick, canonical state, additive deltas, no last-write-wins, DB grants. | yes | yes | no | full | Server-side simulation survived as architecture, sync model, and failure-mode avoidance. |
| S8 — privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md §System-level: per-bird data "drives only that user's own simulation"; telemetry aggregate-only; no analytics DB path. | PLAN.md §0.8-0.9/§7.7/§10.2/§13: email PII, no per-bird telemetry, separate analytics, no external LLM on per-bird facts. | yes | yes | no | full | The privacy boundary is recovered as a product/data-pipeline constraint, not mere policy. |
| S9 — accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md §System-level: accessibility users get "the actual product"; reduced motion designed; narration naturalist; a11y launch-blocking. | PLAN.md §1/§5.7/§9/§11/§12.4: narration, reduced motion, captions, keyboard, launch gates. | yes | yes | no | full | Accessibility survived as affective parity with launch-blocking status. |

For multi-layer system-level whys:

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: PLAN intro names aliveness; §0 first-frame guardrail; §5.1 catch-up tick; §7.4 quiet field; §8 procedural audio; §9 accessible aviary.
- S2: §0.2 no announcement primitives; §4.1 greeting directive; §7.3 staggered greetings; §7.6 fading top bar; §13 announcement-creep risk.
- S3: §5.7 phrase grammar with named moment facts; §8 captions matching calls; §0.4 hidden vector; §1 no comparison/social surfaces.
- S4: §1 two starters/max seven; §7.1 one screen/no pan; §7.6 no chrome in aviary; §8.3 listen-in re-balance, not solo tracks.
- S5: §0.10 copy namespaces; §4 matter-of-fact system errors; §5.7 notebook/narration/captions voice; §11 voice-bundle lint.
- S6: §5.3 exact presence; §5.2 presence-dominant drift; §5.5 settle no directional drift; §0.3 no counters; §13 anti-inflation.
- S7: §0.7 server-only writer; §5.1 tick/catch-up; §6 one canonical state; §6.2 no last-write-wins; §11 DB grants.
- S8: §0.8 email PII; §0.9 per-bird data only simulation; §5.7 no external LLM; §10.2 aggregate telemetry; §13 privacy erosion mitigation.
- S9: §1 a11y in v1; §5.7 narration/captions; §9 designed reduced motion; §12.4 launch gate; §13 a11y regression mitigation.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **72.6%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md §Simulation: strict visibility/focus/activity presence; looser tab-open would inflate drift. | PLAN.md §5.3: all three signals; §4.3 clamps presence; §13 names inflation/gaming risk. | no | full | All three layers substantially recovered. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md §Simulation/Risks: low-pass drift, one-week/three-week calibration, Tamagotchi/screensaver failure band. | PLAN.md §5.2/§5.8/§13: low-pass math, calibration harness, too-fast/too-slow risk. | no | full | Calibration rationale survived with failure modes. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md §System-level/Simulation/Risks: monotonic drift, neglect "quieter, not mistrustful", future engineers must not add negative drift. | PLAN.md §0.5/§5.2/§13: non-negative deltas, quietness through recency/mood, no punishment. | no | full | The no-punishment exception survived strongly. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md §System-level/Audio: no recorded audio; "audible signature of dead software"; chorus avoids stacked loops. | PLAN.md §0.6/§5.6/§8: motif grammar, synth client-side, chorus, fallback silence. | no | full | Procedural audio rationale and cascade survived. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md §Frontend: mood is readable from motion with "no label, tooltip, or status icon". | PLAN.md §7.2: wary scans, content preens, curious tilts, user reads mood from motion. | no | full | Single-layer affective contract recovered. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md §Scope: 7 is "the recognizability ceiling of the call mix". | PLAN.md §12.2: cap is the recognizability ceiling of the call mix; built into engine/adoption. | no | full | Empirical cap rationale recovered. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION.md §Data/Sync: vector is server-only; client cannot reach raw floats; one canonical state prevents clobbering. | PLAN.md §0.4/§3/§6: vector stored server-side, canonical, client type boundary, no LWW. | no | partial | Persistence and sync cascade recovered; the "deleting the bird" relationship layer was thin. |
| F8 | personality-vector-never-numerical | 2 | yes | included | none | none | yes | none | Mechanism survived, but the stat-management/optimization rationale was not grounded or reconstructed. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md §Scope/Frontend: greeting is the only welcome; greeting forms are procedural and staggered. | PLAN.md §4.1/§7.3/§0.2: greeting directive, no textual welcome, staggered forms. | no | partial | Notice/one-bird welcome survived; absence-length and boldness variation did not. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md §System-level: forbids toast/banner/modal/confetti/badge/textual welcome; bird greeting entire welcome. | PLAN.md §0.2/§11/§13: AST lint bans announcement primitives; greeting directive only return affordance. | no | full | Core exception and anti-creep framing recovered. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md §Simulation: settle and tab-close are terminal and identical; neither is penalized. | PLAN.md §5.3/§5.5: settle and tab-close identical at engine level; no "you did not settle" surface. | no | full | The optional, non-penalizing engine rationale survived. |
| F12 | field-notebook-auto-entries | 4 | yes | included | RECONSTRUCTION.md §Scope/Simulation: sparse, read-only, naturalist; records noteworthy moments without becoming a feed. | PLAN.md §5.7/§4.2: naturalist lowercase prose, rarity gates, read-only endpoint. | no | full | Voice, concentration, and rarity/feed refusal recovered. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md §API/Simulation: server clamp and active filtering; strict three-signal presence avoids silent drift inflation. | PLAN.md §4.3/§5.3: visible/focused/recent activity, clamp to wall-clock, discard inactive pings. | no | full | Implementation precision and failure mode recovered. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md §System-level/Guardrails: rejects score/streak/visit count; no counting fields; no user-behavior facts. | PLAN.md §0.3/§5.7/§13: no streaks/day-dot calendar; no visit-count fields; notebook cannot express user behavior. | no | partial | Rule and adjacent-disguise protection survived; intention-rotation rationale was mostly absent. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md §Frontend/Guardrails: first frame mid-action; quiet field; spinner rejected because "a spinner says machine". | PLAN.md §0.1/§7.4/§11: no intro/spinner/fade, snapshot phase, quiet field load state. | no | full | First-frame conceit and spinner consequence recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md §Data: synthetic UUIDs keep email from becoming an internal identifier; email is PII. | PLAN.md §0.8/§3/§11: synthetic UUID, encrypted email once, no email identifiers in logs/keys. | no | partial | PII leakage rationale recovered; impossible-retrofit/non-negotiable layer was not reconstructed. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md §Simulation/Sync: pure tick, active tick plus catch-up, one canonical state, no client sync. | PLAN.md §5.1/§6: tick runs canonical state; client snapshots; server only writer; client tick failure avoided. | no | full | Architecture and failure-mode rationale recovered. |
| F18 | no-last-write-wins-personality | 4 | yes | included | RECONSTRUCTION.md §Sync: clients never send absolute values; morning/lunch sessions fold together rather than overwrite. | PLAN.md §6.2/§0.7: additive server-authored deltas, event log, no client mutation of personality. | no | full | Concrete no-LWW implementation survived. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | none | none | yes | none | Matter-of-fact rule survived, but not the "naturalist error copy reads evasive" why. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md §System-level: per-bird data drives only that user's simulation; telemetry aggregate-only; no analytics DB path. | PLAN.md §0.9/§5.7/§10.2: no per-bird ML, no analytics read grant, grammar avoids external model. | no | full | Private relationship/data-pipeline boundary recovered. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md §API: same host snapshot, minus interaction directives; visitor events never enter host log. | PLAN.md §4.4/§1: read-only ambient visit; observation not co-presence; visitor attention cannot drift host birds. | no | full | Read-only/observation rationale recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | none | none | yes | none | Silent logging/off-by-default survived, but not the attention-driver loop rationale. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | none | none | yes | none | No leaderboards/discovery survived as a rule; comparison/product-shift rationale did not. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md §Accessibility: naturalist running prose gives screen-reader users the actual aviary, not state automation. | PLAN.md §5.7/§9: running prose, slow cadence, same voice as notebook, not ARIA-label automation. | no | full | Accessible affective parity recovered. |
| F25 | reduced-motion-mode | 4 | yes | included | RECONSTRUCTION.md §Accessibility: designed surface; "calmer Pocket Aviary", not broken-looking animations-off. | PLAN.md §9: cross-fades, calls/drift/mood/notebook continue, own calm aesthetic. | no | full | Reduced-motion charm rationale recovered. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECONSTRUCTION.md §Performance: first-bird rendering preserves the immediate alive first frame. | PLAN.md §10.1: below-threshold feels already running; edge inlined snapshot; first bird before non-critical assets. | no | full | Perf-as-aliveness bridge recovered. |
| F27 | no-gamification | 4 | yes | included | RECONSTRUCTION.md §System/Guardrails: no scores/streaks/XP/counters; no counting fields; future streaks blocked. | PLAN.md §0.3/§1/§13: no gamification ever, not settings toggle, no user-behavior facts, anti-creep mitigation. | no | partial | Absolute rule and creep protection survived; adjacent-product temptation layer was thin. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | RECONSTRUCTION.md §System/Simulation/Risks: rejects death/hunger/distress; neglect is quieter, not mistrustful; no negative drift. | PLAN.md §1/§5.2/§13: no Tamagotchi mechanics, monotonic drift, quiet-on-neglect via mood layer. | no | full | Punishment/obligation refusal recovered. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | Reconstruction explicitly says "Two starter birds: NOT RECOVERABLE FROM PLAN"; plan has the rule but not the meeting-animals-vs-catalog why. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECONSTRUCTION.md §Scope/Rollout: avoids visit count, score, or tier; keeps growth outside gamification. | PLAN.md §12.2/§0.3: birds offered by aviary age, not score/tier/visit count; no reward-loop counters. | no | partial | Anti-reward-loop rationale survived; relationship-deepening-over-time layer was less explicit. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md §Scope/Data: bird is never replaced on rename/sync/migration, preserving continuity of relationship. | PLAN.md §3/§1: stable bird_id forever; never replaced on rename/sync/migration. | no | partial | Continuity survived; retroactive evaporation/reset consequence was not fully reconstructed. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md §Simulation: session-start mood is advanced by ticks/catch-up; no neutral snap on tab open. | PLAN.md §5.4: mood persists across sessions; never snaps to neutral on tab open. | no | full | Continuity illusion rationale recovered. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Read-only rule survived, but not the journal/curation relationship shift. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | Export survived as route/boundary; quiet relationship-copy right did not. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md §Scope: 30-day soft window and recovery route give recoverability before hard deletion. | PLAN.md §4.5/§12.4/§13: soft/hard deletion, recover route, privacy gates. | no | partial | Accidental-recovery layer survived; privacy-residue and delete-all-together layers were not recovered. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md §Performance/System: metric boundary protects privacy; no per-bird or per-account interaction dimensions. | PLAN.md §10.2/§0.9: aggregate RUM only, no per-bird/account dimensions, no analytics read path. | no | full | Technical telemetry boundary recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | none | none | yes | none | Per-invite/named/default-off rule survived; private-relationship-not-publication rationale did not. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | none | yes | none | Visit log/no badge rule survived; transparency-without-social-loop rationale did not. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md §API: same host snapshot; "no show-off rendering", keeping visits ambient rather than performative. | PLAN.md §4.4: visitor gets same snapshot; no show-off rendering; visits are ambient/read-only. | no | full | Actual-aviary versus performative share surface recovered. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md §Accessibility/Open calibration: slow narration cadence; no state-list automation; avoids overwhelming queue. | PLAN.md §5.7/§9/§14: 30-60s idle cadence, priority bump only for events, polite running prose. | no | full | Slow rhythm and queue-protection rationale recovered. |

For multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | yes |
| F2 | yes | yes | yes |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F5 | no | no | no |
| F6 | no | no | no |
| F7 | yes | no | yes |
| F8 | no | no | no |
| F9 | yes | no | yes |
| F10 | yes | yes | yes |
| F11 | no | no | no |
| F12 | yes | yes | yes |
| F13 | yes | yes | yes |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F19 | no | no | no |
| F20 | yes | yes | yes |
| F21 | no | no | no |
| F22 | no | no | no |
| F23 | no | no | no |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F26 | no | no | no |
| F27 | yes | no | yes |
| F28 | no | no | no |
| F29 | no | no | no |
| F30 | no | yes | yes |
| F31 | yes | yes | no |
| F32 | no | no | no |
| F33 | no | no | no |
| F34 | no | no | no |
| F35 | yes | no | no |
| F36 | no | no | no |
| F37 | no | no | no |
| F38 | no | no | no |
| F39 | no | no | no |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From constants |
| Possible total weight | 152 | Full-instance possible weight |
| Reachable gold whys | 49 | All 40 feature-why anchors were captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 114 / 152 | Weighted intent numerator/denominator |
| Whys with reconstruction evidence | 38 | At least one rationale layer cited from frozen reconstruction |
| Whys with PLAN grounding | 40 | At least one rationale layer grounded in PLAN |
| `rule_without_why` cases | 9 | Mechanism survived without gold rationale |
| `plan_only_not_reconstructed` cases | 5 | PLAN had rationale but reconstruction did not carry it fully |
| `ungrounded_reconstruction` cases | 0 | No scored cases |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 | 44 | 81.5% |
| Affective whys | 98 | 70 | 71.4% |
| Weight-2 whys | 44 | 24 | 54.5% |
| Weight-3 whys | 108 | 90 | 83.3% |
| System-level whys | 28 | 24 | 85.7% |
| Feature-level whys (reachable) | 124 | 90 | 72.6% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered better (44/54) than affective whys (70/98). The main affective leaks were relationship-shift rationales: F8 stats management, F22 attention-driver notifications, F23 comparison surfaces, F33 journal/curation, F37 private sharing, and F38 visit-log attention loops.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 90/108, helped by the plan's guardrail-heavy treatment. Weight-2 whys recovered 24/44 because many were single-layer exceptions where rule preservation was not enough for credit.
- **System-level vs feature-level.** System-level fidelity (85.7%) was higher than feature-level fidelity (72.6%). The planner preserved philosophy; compression happened when the reconstructor moved from general principles to targeted feature rationale.
- **Multi-layer recovery.** Primary implementation layers usually survived. Secondary and downstream consequence layers leaked when they described how a seemingly harmless variant changes the user's relationship to the product.
- **Subdomain patterns.** Simulation, sync, audio, and accessibility were strongest. Social/account relationship features were weakest: F22, F23, F34, F37, and F38 mostly survived as policy/rules rather than rationale.
- **Evidence-bound effects.** The operator denied several plausible v1-style recoveries because they were rules without why: F8, F19, F22, F23, F29, F33, F34, F37, and F38.

The failure shape suggests the candidate can build a very complete implementation plan, but feature-level intent still compresses into enforceable rules unless the plan explicitly carries the relationship consequence.

## 4. Recommendations for v2 hardening

- Keep and expand targeted headroom additions. F29-F40 exposed meaningful differences that the original canonical table would have missed.
- Add more affective exception whys around social/account surfaces. The plan implemented these areas, but the reconstruction often lost why they are relationship-preserving rather than merely privacy/security-preserving.
- Clarify whether system-level recovery can use evidence outside the reconstruction's system-level section. S6 was present in the reconstruction, but scattered through per-feature text.
- Preserve the v06 evidence-bound ledger. It successfully separates feature capture from rationale recovery and makes rule-without-why cases visible.
- Consider adding a secondary metric for "mechanism survival" if future analyses want to distinguish strong implementation plans from strong intent-preserving plans without contaminating the main fidelity score.

## 5. Methodology caveats

- **Fresh-context fidelity.** The phase prompt states this is fresh context and the frozen reconstruction was not modified. No peer slots, PRD, or other waves were consulted.
- **Single-run-at-temperature limitation.** This is one run only; no variance signal.
- **Borderline capture calls.** Inclusive captures included feature 51 (background-tab pause) and feature 57 (evening calls quieter). They did not affect feature-level why reachability because neither carries a canonical F why.
- **System-level cross-cutting.** S3 and S4 were preserved in PLAN but not identified in reconstruction; S6 was identified but incomplete at the relationship-consequence layer. These are the main subjective calls.
- **Confabulation cases.** No scored ungrounded-reconstruction cases were found.
- **Evidence-bound denials.** Several strong mechanisms did not receive why credit because reconstruction and plan did not both carry the gold rationale.
- **Rule-without-why cases.** Nine included whys were marked rule-without-why, concentrated in social/account/headroom features.

End of report.
