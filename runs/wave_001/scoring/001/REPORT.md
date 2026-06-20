# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Frozen reconstruction was not modified.

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **100.0%** |
| Intent fidelity | **62.5%** |
| Combined quality | **9963** |

**Diagnostic split:**

- System-level fidelity: **89.3%**
- Feature-level fidelity: **56.5%**

**(Planning, fidelity) coordinate:** `(100.0, 62.5)`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label | (blank) |
| Timestamp | 2026-06-20T15:00:18Z |
| Candidate model | openrouter-z-ai-glm-5.2-z-ai-glm-5.2-glm-5.2-glm-z-ai-glm-5.2 |
| Candidate effort | unknown |
| Candidate harness | opencode |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **120 / 120** = **100.0%**.

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
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured in intro, scope, non-goals, and load-bearing rule notes. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured in intro, scope, non-goals, and load-bearing rule notes. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured in intro, scope, non-goals, and load-bearing rule notes. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured in intro, scope, non-goals, and load-bearing rule notes. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in intro, scope, non-goals, and load-bearing rule notes. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured in intro, scope, non-goals, and load-bearing rule notes. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | borderline: Captured inclusively: terms are defined operationally across data model/engine sections, not as a standalone glossary. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured across scope, decisions, data model, and simulation semantics. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured across scope, decisions, data model, and simulation semantics. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured across scope, decisions, data model, and simulation semantics. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured in scope, bird data model, tick, drift, mood, audio, and rollout. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | borderline: Captured inclusively: settle is implemented as a gesture while presence-end/tab-hidden semantics make ordinary close valid; rationale is thin. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured in scope, event model, API, rendering/audio, and load-bearing refusals. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured in render pipeline, first-paint path, top bar, responsive scene, and palette rules. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured in auth/API, account data model, sync model, privacy, export/delete, and telemetry. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured in visit flow, invitation model, revocation, read-only visitor snapshot, and non-goals. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured in visit flow, invitation model, revocation, read-only visitor snapshot, and non-goals. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured in visit flow, invitation model, revocation, read-only visitor snapshot, and non-goals. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured in visit flow, invitation model, revocation, read-only visitor snapshot, and non-goals. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured in visit flow, invitation model, revocation, read-only visitor snapshot, and non-goals. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured in visit flow, invitation model, revocation, read-only visitor snapshot, and non-goals. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured in visit flow, invitation model, revocation, read-only visitor snapshot, and non-goals. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured in visit flow, invitation model, revocation, read-only visitor snapshot, and non-goals. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes | Captured in visit flow, invitation model, revocation, read-only visitor snapshot, and non-goals. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured in visit flow, invitation model, revocation, read-only visitor snapshot, and non-goals. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured in accessibility surfaces, performance budgets, observability, and risk tests. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured explicitly in Not in v1 and out-of-scope confirmations. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured explicitly in Not in v1 and out-of-scope confirmations. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured explicitly in Not in v1 and out-of-scope confirmations. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured explicitly in Not in v1 and out-of-scope confirmations. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **89.3%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System-level intent: "server-side simulation is the load-bearing way" and makes "feels alive, not robotic" literally true. | PLAN.md intro: server tick makes "feels alive" and "continues without the viewer" literal; PLAN §§2,7,8,10 also carry first-paint, procedural audio, and perf aliveness. | yes | yes | no | full | All three aliveness layers are present across continuing simulation, procedural/non-looped calls, and no-spinner first paint. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md System-level intent: "The aviary should notice, never announce"; the bird greeting is "the entire welcome surface." | PLAN.md §§1,8,13: no welcome toast, no audio-enable modal, top-bar fade, no default visit notification, and no streak/calendar surfaces. | yes | yes | no | full | Recovered as a cross-surface refusal of toasts, pings, and engagement announcements. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md System-level intent: "The voice is sparse naturalist prose" and the notebook "observes the aviary, not the user." | PLAN.md §§1,3,9,13: notebook/narration/captions use specific naturalist prose; numeric trait exposure, leaderboards, and user-frequency observations are refused. | yes | yes | no | full | The reconstruction preserved specificity through prose voice, observations, and non-stat surfaces. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md §§1,7,8,13,14: two starter birds, max seven, one horizontal scene, no user placement, no scene customization, and listen-in rebalances rather than becoming a tracks UI. | no | yes | no | partial | PLAN clearly carries restraint, but the reconstruction never elevates it as a system-level principle. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System-level intent: "The voice is sparse naturalist prose, except for errors." | PLAN.md §§4,9: notebook/narration are naturalist; API/account/sync errors are matter-of-fact and never naturalist. | yes | yes | no | full | Recovered the named voice split and its surface inheritance. |
| S6 - presence-is-real-interaction | 4 | included | none | PLAN.md §§1,3,5,12,13: presence is visibility + focus + recent activity, dominant drift input, settle/close equivalent at engine level, no streak surfaces, and server validation against drift inflation. | no | yes | no | partial | PLAN preserves all layers, but the reconstruction only treats presence at feature level, not as a system-level principle. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System-level intent: "Canonical state belongs on the server" and the client is "a renderer and an event-ingestor." | PLAN.md §§2,5,6,13: sim owns canonical state, clients render snapshots, server is only personality writer, and no client-side tick is allowed. | yes | yes | no | full | Recovered the server-authoritative architecture, sync consequence, and no-last-write-wins implication. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System-level intent: "Privacy is structural, not cosmetic." | PLAN.md §§1,3,10,13: synthetic account IDs, aggregate-only telemetry, no per-bird/per-account aggregation, and no email in logs. | yes | yes | no | full | Recovered privacy as an enforced data-pipeline and identifier boundary, not just policy language. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System-level intent: accessibility "ships as part of v1" and is "a designed surface." | PLAN.md §§1,7,9,12,13: running prose narration, designed reduced motion, captions, keyboard/focus, and v1 accessibility rule. | yes | yes | no | full | Recovered accessibility as product-quality parity, not a checklist or delayed fallback. |

Multi-layer system-level recovery:

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: intro server tick; client/server split; first paint from bootstrap snapshot; procedural audio; no-spinner quiet field; performance targets.
- S2: no welcome toast; bird greeting entire welcome; top-bar fade; no audio-enable modal; no default visit notifications; no streak or calendar surfaces.
- S3: naturalist notebook prose; screen-reader prose; captions generated from calls; no numeric personality exposure; no leaderboards/discovery.
- S4: two starters/max seven; one horizontal scene; no user placement; no scene customization; listen-in rebalances rather than exposing tracks.
- S5: notebook/narration/captions use naturalist voice; auth/account/sync errors use matter-of-fact voice; accessibility settings remain system-clear.
- S6: three-signal presence; presence drives drift; settle/no close-tab penalty implicit through presence end; no streaks; server-side validation prevents inflation.
- S7: sim owns canonical state; server tick runs without clients; clients render snapshots; no client personality writes; no client-to-client sync.
- S8: synthetic IDs; encrypted email only once; telemetry excludes per-bird/per-account data; no per-account visit/session metrics; export/delete boundaries.
- S9: running prose narration; reduced-motion cross-fade surface; captions; keyboard/focus; accessibility ships with v1.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **56.5%**.
Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md Scope: presence accounting is "precision against drift inflation" while "watching birds without moving is the actual product." | PLAN.md Decision + §§3,12: 120s activity window, visibility/focus/activity conjunction, server validation, and drift-inflation risk. | no | partial | Recovered precision and watching-without-moving, but not the full silent-failure/account-wide corruption consequence. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md Simulation/Risks: low-pass additive drift is "smooth, monotonic, asymptotic" with one-week/three-week calibration and Tamagotchi/screensaver failure modes. | PLAN.md §§5,12: LPF formula, one-week instrument/three-week user target, and too-fast/too-slow risk shape. | no | full | All calibration layers survived. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md System-level intent: "Drift is care without punishment"; unattended birds are "quieter than it was, not sadder than it was." | PLAN.md §§5,12,13: drift never subtracts, neglect not punished, fuzz test, no Tamagotchi mechanics. | no | full | Recovered the asymmetric no-punishment rule and its product consequence. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md Audio/System: procedural calls are recognizable, not recorded loops, and the synth is the "affective spine." | PLAN.md §§5,8,10,13: WebAudio motif plans, no recorded loops/fallback, recognizability tests, and bundle implications. | no | partial | Recovered no-loops and audio-spine implications; missed the phase-canceling chorus layer. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md Frontend: mood-shaped idle micro-motion makes state "visible through pose and idle choice." | PLAN.md §7: wary scans, content preens, curious tilts, drowsy fluffs; mood is read from motion. | no | full | Recovered mood as visible behavior rather than a label. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | The cap appears, but the reconstruction ties it to age pacing/gamification rather than empirical call recognizability. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION.md System/Data: canonical state is server-owned; the client never writes personality and never derives it locally. | PLAN.md §§2,3,6,13: personality is server-only, updated by tick, never client-owned, and protected from last-write-wins. | no | partial | Recovered canonical/server persistence and sync consequence; missed the relationship-deletion rationale. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md System-level intent: personality must be "felt, not inspected numerically" and API uses derived render hints. | PLAN.md §3 and §13: client receives coarse render hints, not scalars; debug scalar views are regressions. | no | full | Recovered the relationship-protecting reason, not merely the UI rule. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md Scope/System: the return greeting is "notice, never announce," replaces the toast, and varies by absence length, boldness, and mood. | PLAN.md §§1,7,13: procedural return greeting, no welcome toast, absence/boldness/mood variation, first-second bird notice. | no | full | Recovered greeting as the session anchor and anti-toast surface. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md Load-bearing refusals: "No welcome toast" because it violates "notice, never announce." | PLAN.md §§1,13: bird greeting is entire welcome; no toast, banner, greeting modal, or welcome text. | no | partial | Recovered the primary anti-toast rationale, but not the well-meaning-contributor or every-variant downstream layers. |
| F11 | settle-is-opt-in | 2 | yes | included | none | none | yes | none | Settle is implemented, but neither plan nor reconstruction recovers the chore/penalty rationale for opt-in close-tab equivalence. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md Scope/System: notebook is sparse naturalist observation, "not a feed," with notability from simulation deltas. | PLAN.md §§1,3,5,9,13: lowercase naturalist prose, rare entries, read-only API, no per-session or user-behavior observations. | no | partial | Recovered naturalist prose and rarity/read-only separation; missed the stock-event-log spell-breaking layer. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md Scope/Risks: presence accounting is precision against drift inflation and server-side validation prevents population-wide inflation. | PLAN.md §§1,3,12: simultaneous three-signal requirement, server duration computation, 30-minute cap, and silent inflation risk. | no | partial | Recovered precision and population drift risk, but not the per-signal failure analysis. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md Load-bearing refusals/System: no streaks, green-dot calendars, visit-frequency surfaces, and notebook observes aviary not user. | PLAN.md §§1,13,14: no streak/counter/calendar and no visit-frequency observations; user-behavior observations are out of scope. | no | partial | Recovered the absolute surface refusal and adjacent disguises; missed the managing-a-number intention shift. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md Architecture/Frontend: first frame from bootstrap snapshot, no entry animation, no fade-from-static, no spinner, quiet-field fallback. | PLAN.md §§2,7,10: inline snapshot, birds mid-pose, quiet field if missing, first bird before API, no spinner/fly-in. | no | full | Recovered first-paint aliveness, server-snapshot implementation, and no-spinner loading consequence. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md Privacy/Data: synthetic account UUID is used anywhere except encrypted email storage; email in a log is a PII leak. | PLAN.md §§1,3,13: synthetic UUID is the only identifier; email encrypted once; no email in logs/telemetry/shards. | no | partial | Recovered identifier and PII-leak layers; retrofit/non-negotiable consequence is not articulated. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md System/Simulation: the server-side tick makes the aviary continue, keeps sync coherent, and client-side tick would collapse sync. | PLAN.md intro + §§2,5,6,13: ~60s tick, reads event log, writes canonical state, runs without clients, clients render snapshots. | no | full | Recovered architecture, multi-device coherence, and client-tick failure consequence. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md Sync/System: clients never write personality; there is no last-write-wins because there is no client personality write. | PLAN.md §§3,5,6,12,13: additive server-authored deltas, event log, server tick only, no client absolute values. | no | partial | Recovered the implementation rule; missed the invisible deletion example of one device overwriting another. |
| F19 | sync-conflict-tone | 2 | yes | included | none | none | yes | none | Matter-of-fact errors are present, but the evasive-naturalist/blocked-user clarity rationale is not recovered. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md Privacy/Performance: aggregate-only telemetry; metrics requiring per-bird or per-account state are not collected. | PLAN.md §§1,10,11,13: per-bird/per-account interaction state is never aggregated, used for ML, or included in telemetry pipelines. | no | partial | Recovered the storage/use rule and telemetry boundary; missed the private-relationship/data-product rationale. |
| F21 | visit-read-only-ambient | 2 | yes | included | none | none | yes | none | Read-only visits are present, but the co-presence-would-be-a-larger-multi-user-product why is not recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md Scope: host visit notifications are off by default because the aviary "does not ping the user." | PLAN.md §§1,4,14: no push/ping/email about the aviary; visit notifications default off and are never surfaced during onboarding. | no | full | Recovered the attention-driver refusal in simpler language. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | The rule appears, but the comparison-surface relationship shift and aggregation cascade are absent. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md Accessibility: screen-reader narration is current-state naturalist observation from the same snapshot, with unified voice and slow cadence. | PLAN.md §§1,9,12,13: live-region prose, same generator as notebook, naturalist current-state observations, event prose priority. | no | partial | Recovered running prose and voice continuity; missed the explicit ARIA/state-list wrong-feature warning. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md System/Frontend: reduced motion is "not a stripped fallback" and the user still gets the actual product, a different aesthetic. | PLAN.md §§1,7,9,13: cross-fades replace micro-motion, calls/notebook/drift remain, and accessibility ships with v1. | no | full | Recovered the designed alternate rendering and non-degraded accessibility stance. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECONSTRUCTION.md System/Performance: performance is part of the product feeling; <500ms first bird uses inline bootstrap before API round-trip. | PLAN.md §§2,7,10: inline snapshot, first bird drawn before non-critical assets/API, edge snapshot replication. | no | full | Recovered first-bird speed as felt aliveness, not just a mechanical target. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md System/Load-bearing refusals: no achievements, streaks, levels, scores, badges, XP, and age pacing refuses the gamification trap. | PLAN.md §§1,11,13,14: no gamification in any version/surface; age-based unlocks; no counters/calendars/milestones. | no | partial | Recovered the loud refusal; missed the predictable-temptation and foothold-to-different-product layers. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md System: non-Tamagotchi, no hunger/death/distress, neglect is not punished, birds become quieter not sadder. | PLAN.md §§1,5,13,14: no death/hunger/distress/decaying happiness; monotonic drift; absence is not punished. | no | full | Recovered observational relationship and no-punishment engine. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | Reconstruction explicitly marks two starter birds as NOT RECOVERABLE FROM PLAN; no arrivals-not-catalog why is recovered. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md Scope/Rollout: new birds are by aviary age and "load-bearing refusal of the gamification trap," not visit count or interaction total. | PLAN.md §§1,11: third through seventh bird offers are tied to created_at/aviary age, not visit count, interaction total, or paid tier. | no | partial | Recovered age-not-attention reward refusal; missed the downstream economy/erosion consequence. |
| F31 | stable-bird-identity | 4 | yes | included | none | none | yes | none | The reconstruction marks stable bird UUID as NOT RECOVERABLE FROM PLAN; rename independence is only a rule-level fragment. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md Scope/Data: mood persisted across sessions is for continuity; session-open does not reset mood to neutral. | PLAN.md §§1,3,5: mood persists, tick can transition it, and opening a tab never resets to neutral. | no | full | Recovered mood continuity and no neutral startup reset. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Read-only notebook is present, but the observer-record-not-user-journal rationale is not recovered. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | Reconstruction explicitly says the account-export product rationale is NOT RECOVERABLE FROM PLAN. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | none | none | yes | none | Reconstruction explicitly says the deletion product rationale is NOT RECOVERABLE FROM PLAN. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md Privacy/Performance: aggregate-only telemetry; metrics requiring per-bird or per-account state are not collected. | PLAN.md §§1,10,11,13: request counts/latencies/errors only; no per-bird state or relationship-reconstructing dimensions. | no | full | Recovered observability as a technical privacy boundary. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md Risks: visitor re-validation is "privacy-correct" because the host invited a specific email, not a URL. | PLAN.md §§1,4,12: email-based invite, no global discoverable flag, no implicit sharing, revalidate visitor email on each open. | no | full | Recovered named, deliberate sharing rather than ambient discoverability. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | none | yes | none | Host visit log is marked NOT RECOVERABLE; transparency-without-attention-loop rationale is absent. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Visitor snapshot is implemented, but the no-show-off-mode/marketing-rendering why is not recovered. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md Accessibility: narration uses slow cadence, event prose coexists, and high-frequency narration is explicitly avoided. | PLAN.md §§1,9,12: idle narration every 30-60s, event narration within 2s, polite live region, queue-overwhelm risk. | no | full | Recovered slow rhythm, queue protection, and event-vs-idle cadence distinction. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | yes |
| F3 | yes | yes | yes |
| F4 | yes | no | yes |
| F7 | yes | no | yes |
| F9 | yes | yes | yes |
| F10 | yes | no | no |
| F12 | yes | no | yes |
| F13 | yes | no | yes |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | no | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | no |
| F25 | yes | yes | yes |
| F27 | yes | no | no |
| F30 | yes | yes | no |
| F31 | no | no | no |
| F35 | no | no | no |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON gold_why_totals |
| Possible total weight | 152 | From score JSON intent_recovery.total_possible_weight |
| Reachable gold whys | 49 | S whys included; all 40 F anchors captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 95.0 / 152 | Sum of weight times recovery score over included whys |
| Whys with reconstruction evidence | 35 | Rows with exact rationale evidence in frozen reconstruction |
| Whys with PLAN grounding | 37 | Rows with exact rationale grounding in PLAN |
| rule_without_why cases | 12 | Mechanism survived without the gold rationale |
| plan_only_not_reconstructed cases | 2 | PLAN carried rationale but system-level reconstruction missed it |
| ungrounded_reconstruction cases | 0 | No scored recovery depended on ungrounded claims |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 32.0 | 59.3% |
| Affective whys | 98.0 | 63.0 | 64.3% |
| Weight 2 whys | 44.0 | 23.0 | 52.3% |
| Weight 3 whys | 108.0 | 72.0 | 66.7% |
| System-level whys | 28.0 | 25.0 | 89.3% |
| Feature-level whys (reachable) | 124.0 | 70.0 | 56.5% |

## 3. Diagnostic patterns

Affective whys recovered slightly better than functional whys (64.3% vs. 59.3%), but both leaked in the same way: primary rules survived, while secondary and downstream rationales were compressed away. Functional leakage was visible in F16 and F18, where the architecture survived but retrofit/invisible-loss layers did not.

Weight-3 whys recovered better than weight-2 whys (66.7% vs. 52.3%). The strongest full recoveries were the concepts the plan repeatedly named as load-bearing: F2 drift calibration, F3 monotonic drift, F15 first paint, F17 server tick, F25 reduced motion, and F40 narration cadence.

System-level fidelity was much stronger than feature-level fidelity. The reconstruction recovered the big architecture and philosophy (S1, S2, S5, S7, S8, S9), but missed S4 as a named principle and treated S6 mostly as implementation detail. Feature-level misses cluster around social/privacy relationship whys and quiet product-quality surfaces: F21, F23, F29, F31, F33, F34, F35, F38, and F39.

The evidence-bound operator mattered. Twelve rows were rule_without_why: the plan/reconstruction preserved what to build but not why it mattered. In a looser semantic pass those rows might look acceptable; v06 correctly keeps them at none recovery.

## 4. Recommendations for v2 hardening

Keep targeted headroom whys like account export, deletion, visit log, actual visitor rendering, and starter-bird arrival framing. This run demonstrates that a strong planner can implement those surfaces while losing the relationship rationale.

Keep multi-layer whys and the exact-evidence gate. The common pattern was L1 recovery with L2/L3 loss; binary scoring would hide that useful diagnostic.

Preserve the system-level cross-cutting appendix. S4 and S6 show why it matters: the plan can preserve a principle cross-cuttingly even if the reconstruction does not name it at the system level.

## 5. Methodology caveats

Fresh-context fidelity appears to hold. The validity audit found no gold IDs, no scoring vocabulary, and headings that mirror the plan structure rather than GOLD_WHYS.md.

This is one run, so there is no variance signal. Planning quality was 100.0%, so low-confidence degeneracy does not apply.

Borderline capture calls were handled inclusively per RUBRIC §4.7: feature #7 was captured operationally rather than as a standalone glossary, and feature #44 was captured through settle plus presence-end semantics even though the opt-in rationale was thin.

The most subjective calls were S4 and S6. I scored both partial because PLAN preserved the principle across 3+ places, but the reconstruction did not identify the principle in its System-level intent section.

Timing note: TIMING.json contained phase1 and phase2a for run 001, but no phase2b entry yet. The score JSON includes only the available bounded timing fields.

End of report.
