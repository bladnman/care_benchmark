# REPORT - CARE run 001

> Phase 2B evidence-bound scoring report for Pocket Aviary v1. Companion artifacts: frozen `RECONSTRUCTION.md`, strict score JSON, and HTML twin.

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **99.2%** |
| Intent fidelity | **89.5%** |
| Combined quality | **9906** |

**Diagnostic split:**

- System-level fidelity: **100.0%**
- Feature-level fidelity: **87.1%**
- (Planning, fidelity) coordinate: `(99.2, 89.5)`

### Run metadata

| Field | Value |
|---|---|
| Run number | 1 |
| Run label | (blank; wave metadata not read due allowlist) |
| Timestamp | 2026-07-26T10:23:14Z |
| Candidate model | claude-5-opus |
| Candidate effort | high |
| Candidate harness | claude-code |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **119 / 120** = **99.2%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 19 | 95.0% |
| aviary_layout.md | 18 | 18 | 100.0% |
| accounts_sync.md | 18 | 18 | 100.0% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **119** | **99.2%** |

Per-feature detail (full 120-feature capture denominator):

| Feature # | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured by the whole v1 plan: one canonical browser aviary with named birds, procedural calls, presence, and field-notebook surfaces. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured by PLAN scope and implementation detail. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured by PLAN scope and implementation detail. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured by PLAN scope and implementation detail. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured by PLAN scope and implementation detail. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured by PLAN scope and implementation detail. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Captured by PLAN scope and implementation detail. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured by PLAN scope and implementation detail. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured by PLAN scope and implementation detail. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured by PLAN scope and implementation detail. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes (borderline) | Captured, borderline: PLAN gives two starter birds/adoption naming and a bird-arrival treatment, but does not spell out no species catalog. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured by PLAN scope and implementation detail. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured by PLAN scope and implementation detail. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | Not captured: PLAN includes a 5s settle undo, but not the click-anywhere trigger. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured by PLAN scope and implementation detail. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured by PLAN scope and implementation detail. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured by PLAN scope and implementation detail. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured by PLAN scope and implementation detail. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured by PLAN scope and implementation detail. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured by PLAN scope and implementation detail. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured by PLAN scope and implementation detail. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured by PLAN scope and implementation detail. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured by PLAN scope and implementation detail. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured by PLAN scope and implementation detail. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured by PLAN scope and implementation detail. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured by PLAN scope and implementation detail. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured by PLAN scope and implementation detail. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured by PLAN scope and implementation detail. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured by PLAN scope and implementation detail. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured by PLAN scope and implementation detail. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured by PLAN scope and implementation detail. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **100.0%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System-level intent item 2: "first frame already in motion", "no load state", "no canned cue", "no looped audio", "no cycle animation". | PLAN.md section 0 INV-1 plus sections 6.2, 7.7, 8, 10.3, 10.4: no load state, procedural calls, first frame mid-motion, quiet field. | yes | yes | no | full | Recovered as a cross-cutting aliveness invariant spanning loading, motion, audio, greeting, and degradation. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md System-level intent item 3: "product affect as something the aviary does, not something UI copy declares". | PLAN.md section 0 INV-2; sections 7.8, 10.6, 10.7, 11.3, 12.5: no toast/banner/notification/welcome copy, no assertive live regions. | yes | yes | no | full | Recovered as a refusal of announcement surfaces, not just a UI-copy preference. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md System-level intent items 6 and 12: "noticing consequences, not reading state"; naturalist product surface for aviary, notebook, narration, and captions. | PLAN.md sections 4, 10.6, 10.7: PRD nouns, one shared prose package, named birds and concrete observable details. | yes | yes | no | full | Recovered through the naturalist prose and observable-specificity mechanism. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md System-level intent item 16: "feel like a place, not an app or dashboard" with no chrome, sparse notebook, and top-bar fade. | PLAN.md sections 1.1, 1.2, 7.8, 8.7, 10.1, 10.2: two starters, seven max, one screen, no panning, no UI chrome in scene. | yes | yes | no | full | Recovered as restraint in bird count, scene model, chrome, audio, and notebook density. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System-level intent item 12: naturalist product surface and "matter-of-fact prose for auth, account, sync, accessibility, and browser errors". | PLAN.md section 0 INV-12; sections 4, 6, 10.6, 12.5: no inline strings, prose split, error codes mapped to system prose. | yes | yes | no | full | Recovered as an enforceable voice split, though some feature-specific tone whys still compress. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md System-level intent item 8: "Presence must be credited honestly" and "visible and focused and recent input". | PLAN.md sections 0 INV-8, 6.4, 7.2, 9.3, 12.3: conjunction, stale rejection, gap ceilings, multi-device interval union, settle/tab-close equivalence. | yes | yes | no | full | Recovered as honest attention accounting, not raw engagement measurement. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System-level intent items 5 and 13: "only the simulation writes personality" and server discrete canonical state. | PLAN.md sections 2.1, 5.5, 7.1, 9.1, 9.4: sim-worker sole writer, grants, server tick, canonical snapshots, no client trait endpoint. | yes | yes | no | full | Recovered as architecture, sync model, and data grant boundary. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System-level intent item 9: "Per-bird interaction data never leaves the simulation boundary" and telemetry is aggregate only. | PLAN.md sections 0 INV-9, 2.1, 11.2, 11.3, 13.4: separate telemetry store, no account dimension, no CDC, no behavioral analytics. | yes | yes | no | full | Recovered as a technical data-pipeline boundary rather than policy wording. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System-level intent item 11: "accessibility ships with v1, not after" and AA with flat state-list feel is a fail. | PLAN.md sections 0 INV-11, 10.5-10.10, 12.6, 13.1: reduced-motion peer renderer, naturalist narration, captions, keyboard, external affective audit. | yes | yes | no | full | Recovered as designed charm in accessibility surfaces, not parity-only compliance. |

Multi-layer system why recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

Cross-cutting evidence appendix:

- S1: PLAN.md section 0 INV-1 plus sections 6.2, 7.7, 8, 10.3, 10.4: no load state, procedural calls, first frame mid-motion, quiet field.
- S2: PLAN.md section 0 INV-2; sections 7.8, 10.6, 10.7, 11.3, 12.5: no toast/banner/notification/welcome copy, no assertive live regions.
- S3: PLAN.md sections 4, 10.6, 10.7: PRD nouns, one shared prose package, named birds and concrete observable details.
- S4: PLAN.md sections 1.1, 1.2, 7.8, 8.7, 10.1, 10.2: two starters, seven max, one screen, no panning, no UI chrome in scene.
- S5: PLAN.md section 0 INV-12; sections 4, 6, 10.6, 12.5: no inline strings, prose split, error codes mapped to system prose.
- S6: PLAN.md sections 0 INV-8, 6.4, 7.2, 9.3, 12.3: conjunction, stale rejection, gap ceilings, multi-device interval union, settle/tab-close equivalence.
- S7: PLAN.md sections 2.1, 5.5, 7.1, 9.1, 9.4: sim-worker sole writer, grants, server tick, canonical snapshots, no client trait endpoint.
- S8: PLAN.md sections 0 INV-9, 2.1, 11.2, 11.3, 13.4: separate telemetry store, no account dimension, no CDC, no behavioral analytics.
- S9: PLAN.md sections 0 INV-11, 10.5-10.10, 12.6, 13.1: reduced-motion peer renderer, naturalist narration, captions, keyboard, external affective audit.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **87.1%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md section 7: presence pings require visible/focused/recent input; stale rejection, gap ceilings, and interval union protect honest drift. | PLAN.md sections 7.2 and 12.3: client conjunction, server gap ceiling, stale rejection, presence fixture tests. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md sections 7 and 13: low-pass/saturating drift, calibration profiles, dogfood for three-week felt change. | PLAN.md sections 7.3 and 7.4: low-pass formula, one-week measurable/three-week visible calibration harness. | no | partial | Recovered slow low-pass calibration, but not the full Tamagotchi-versus-screensaver downstream contrast. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md section 1/7: neglect means no decrease, no Tamagotchi, "Quieter, not diminished". | PLAN.md sections 7.3 and 7.4: monotonic formula, lapsed-returner and single-session calibration profiles. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md sections 1/8: no recorded playback, infinite variation, real-time chorus, WebAudio fallback has no recorded path. | PLAN.md sections 8.1-8.8: motif grammar, synthesis graph, chorus grouping, no decoder or audio asset pipeline. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md sections 3/7/10: mood changes behaviors and narration uses observable details, not labels. | PLAN.md sections 3, 7.5, 10.3, 10.7: mood biases behavior and is read through motion and posture. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md sections 1/8: seven tied to recognizability and cap can drop if recognition fails. | PLAN.md sections 8.7 and 13.3: listening study gates at 5 and 7 birds, config cap if recognition fails. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md sections 5/7/9: vector is server canonical, never recomputed, and client cannot write personality. | PLAN.md sections 5.5, 5.7, 7.1, 9.4: persisted vector, filter_state, sim-only grants, no client trait endpoint. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md System-level item 6 and section 6: direct trait/mood fields remove noticing; product surfaces keep personality numbers off UI. | PLAN.md sections 3, 6.3, 12.5, 15: no trait names or mood in snapshot; export is outside product surface. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md sections 1/7: greeting uses absence, hidden traits, one greeter, stagger, recency penalty, and variation. | PLAN.md section 7.8: absence bands, weighted greeter selection, stagger, recency penalty, no welcome text. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md System-level item 3 and section 7: greeting is the entire welcome; no toast/banner/welcome/absence text. | PLAN.md sections 1.2, 7.8, 10.6, 12.5: no welcome copy, no absence surface, banned prose corpus. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md section 7: settle and tab-close are neither penalized nor required; settle is only lighting and mood-quieting. | PLAN.md section 7.2: settle and tab-close are absence of pings; neither is penalized or required. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md sections 1/10/12: sparse naturalist prose, not event log, unread count, badge, or user-behavior surface. | PLAN.md sections 5.8, 10.6, 12.5, R7: immutable naturalist notebook, sparsity, forbidden user-behavior prose. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md sections 7/12: visible/focused/recent input pings, gap ceiling, stale replay rejection, exact fixture tests. | PLAN.md sections 6.4, 7.2, 12.3: all three client conditions, 15s pings, min gap, stale rejection, fixtures. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md System-level item 4 and section 5: no streaks/days visited/counters; presence_credit is the future days-visited number. | PLAN.md sections 1.2, 5.7, 6.6, 12.5: no streak/counter surfaces, no stats endpoints, API cannot read presence_credit. | no | partial | Recovered the refusal and adjacent disguises; compressed the user-intention rotation into generic no-gamification language. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md sections 1/10: first frame already in motion, no spinner, quiet field, vector-first birds before atlas decode. | PLAN.md sections 6.2, 10.3, 10.4: quiet field, activity phase, first-frame bird silhouettes, forbidden loading components. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md System-level item 10 and section 5: email appears in two encrypted places; blind index; CI rejects email leaks. | PLAN.md sections 5.1, 11.3, 12.5: synthetic account_id everywhere else, encrypted email, metric/log/cache deny-list. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md sections 1/7/9: tick runs without client, catches up on return, one canonical record and one writer. | PLAN.md sections 2.1, 7.1, 9.1: sim-worker tick cadence, server canonical state, clients pull snapshots. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md section 9: no endpoint accepts traits, API grants cannot write vectors, writes are current-row deltas. | PLAN.md sections 5.5, 5.7, 9.4: additive deltas, server ingest_seq order, sim-only grants, no absolute trait submission. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F19 | sync-conflict-tone | 2 | yes | included | none | none | yes | none | Rule survived (matter-of-fact sync/account errors), but the evasive-naturalist-error rationale was not recovered. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md System-level item 9 and section 11: relationship data never leaves simulation boundary; no behavioral analytics. | PLAN.md sections 2.1, 11.2, 11.3: telemetry store separate, aggregate-only metrics, no account or per-bird dimensions. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md sections 1/6: visitor route cannot write events and visitor attention does not drift host birds. | PLAN.md sections 5.8, 6.5: visit_sessions never join interaction_events; visitor route strips interaction modules. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F22 | no-friend-visited-notification | 2 | yes | included | none | none | yes | none | Rule survived (no friend-visited notification by default), but the attention-driver/social-loop rationale was not recovered. |
| F23 | no-leaderboards | 2 | yes | included | RECONSTRUCTION.md sections 1/11: no public discovery, leaderboards, social network, or underlying aggregates. | PLAN.md sections 1.2, 6.6, 11.3: no leaderboard/discover paths and no cross-account metrics pipeline. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md System-level item 11 and section 10: naturalist narration, observable details, not state lists or trait/mood labels. | PLAN.md sections 10.7, 10.10, 12.6: running prose, polite regions, affective external audit, not ARIA state lists. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md sections 10/12: reduced motion is a distinct designed render mode with calls, drift, mood, and notebook retained. | PLAN.md sections 10.5, 10.10, 12.6: peer render mode, slow cross-fades, paired screenshots, designed charm. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md sections 10/11: first bird under 500ms through quiet field, vector silhouettes, inlined snapshot, critical module split. | PLAN.md sections 1.4, 10.4, 11.1: <500ms first bird, vector-first trick, budget breakdown. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md System-level items 4/14 and sections 11/12: no counters, ranks, streaks, analytics pressure, or gamification lexicon. | PLAN.md sections 1.2, 1.3, 6.6, 11.3, 12.5, R6: no gamification surfaces or underlying aggregates; refusals suite. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md sections 1/7: no Tamagotchi, no distress, neglect never reduces traits, quieter not diminished. | PLAN.md sections 1.2, 7.3, 12.2: no death/hunger/distress, monotonic drift, drift-on-neglect calibration tests. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | PLAN.md section 10.4: the only entry animation is justified because "a bird arriving is the event". | yes | none | The reconstruction explicitly says two starter birds at adoption are NOT RECOVERABLE FROM PLAN; starter-as-arrival why scores none. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md section 1: age-only gating avoids visits, interactions, or payment as gamified inputs. | PLAN.md sections 5.3 and 13.3: new birds use aviary age only; not visits, interactions, or payment. | no | partial | Recovered rejection of reward-loop inputs, but not the deeper relationship-deepening/prize-economy framing. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md System-level item 7 and section 5: immutable bird_id, fixed timbre_seed, species retention, no row recreation. | PLAN.md sections 5.4, 5.9, 8.4, 15: stable identity, species retirement policy, identity/expression split. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md section 7: stored persistent mood, no neutral snap, session start is not an input. | PLAN.md sections 5.6, 7.5: mood stored server-side and daily-ish dawn shift without neutral reset. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md section 5: notebook entries immutable/read-only; final naturalist record, not user-edited or badge-tracked log. | PLAN.md sections 5.8, 6.5, 10.6: no UPDATE/DELETE endpoint, final body, sparse naturalist templates. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION.md section 1/5: export is a data-portability artifact; user at least holds a copy of the aviary. | PLAN.md sections 5.10 and R9: export JSON snapshot, signed URL, account access mitigation. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md sections 1/5: soft delete supports recovery unchanged; hard delete completes privacy obligations and backup aging. | PLAN.md section 5.10: 30-day soft delete, FK-safe hard purge, backups age out by day 65. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md System-level item 9 and section 11: aggregate-only telemetry, no account/per-bird dimensions, no behavioral analytics. | PLAN.md sections 11.2, 11.3, 12.5: aggregate request/latency/error metrics and typed label deny-lists. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md section 1/6: per-invite email invitations, revocation, limited read-only viewing, no social network. | PLAN.md sections 1.1, 5.8, 6.5: per-invite token, visitor email, immediate revocation, separate visitor route. | no | full | Recovered with reconstruction evidence and matching plan grounding. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | none | yes | none | The reconstruction says the more specific visit-log rationale is NOT RECOVERABLE FROM PLAN; on-demand transparency why scores none. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Read-only visitor view survived, but the no-show-off-mode / real-host-aviary rationale was not reconstructed. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md sections 10/15: 30-60s idle narration, priority lane, rate limiting/coalescing, natural observation without flooding. | PLAN.md section 10.7: idle live region every 30-60s, priority <=1 per 8s, coalescing and polite regions. | no | full | Recovered with reconstruction evidence and matching plan grounding. |

Multi-layer feature why recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | yes |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | yes | yes |
| F9 | yes | yes | yes |
| F10 | yes | yes | yes |
| F12 | yes | yes | yes |
| F13 | yes | yes | yes |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | yes |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | yes | yes |
| F30 | no | yes | no |
| F31 | yes | yes | yes |
| F35 | yes | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From benchmark constants |
| Possible total weight | 152 | Full-instance possible weight |
| Reachable gold whys | 49 | S whys always included; all F whys reachable here |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 136 / 152 | Sum of weight times recovery over included whys |
| Whys with reconstruction evidence | 44 | Aggregate evidence-bound count |
| Whys with PLAN grounding | 45 | Aggregate evidence-bound count |
| rule_without_why cases | 5 | Aggregate evidence-bound count |
| plan_only_not_reconstructed cases | 1 | Aggregate evidence-bound count |
| ungrounded_reconstruction cases | 0 | Aggregate evidence-bound count |

### 2.5. Failure groupings

| Grouping | Recovered weight | Reachable weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 52 | 54 | 96.3% |
| Affective whys | 84 | 98 | 85.7% |
| Weight 2 whys | 34 | 44 | 77.3% |
| Weight 3 whys | 102 | 108 | 94.4% |
| System-level whys | 28 | 28 | 100.0% |
| Feature-level whys (reachable) | 108 | 124 | 87.1% |

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered 52/54 weighted points. Affective whys recovered 84/98; the misses were mostly social/tone or relationship-framing rows where the rule survived but the deeper why did not.
- **Weight-3 vs weight-2.** Weight-3 rows recovered 102/108. The partials were F2, F14, and F30, each missing one or two deeper layers. Weight-2 rows recovered 34/44 because targeted headroom additions exposed rule-only compression.
- **System-level vs feature-level.** The plan preserved philosophy extremely well: all S1-S9 were cross-cutting in at least three plan decisions. Feature-level fidelity was lower because broad invariants did not always carry the feature-specific why.
- **Multi-layer recovery.** Primary implementation layers survived best. Downstream product-consequence layers were the most likely to compress away, especially F2 and F30.
- **Subdomain patterns.** Engine, sync, privacy, audio, and accessibility were highly faithful. Social/visit and tone rows lost more why detail: F19, F22, F38, and F39.
- **Evidence-bound effects.** Five rows were rule-without-why cases. Under a looser v1-style read they might be credited from inherited system principles; v06 correctly denied feature-level recovery where the reconstruction did not carry the specific why.

## 4. Recommendations for v2 hardening

- Keep targeted headroom additions like F29-F40. They prevented saturation in an otherwise near-complete plan.
- Add scorer guidance for social/tone rows: inherited S2/S3 intent is not enough unless the feature-specific rationale is reconstructed.
- Keep multi-layer scoring for high-weight whys. It exposed downstream-consequence compression that a single full/none judgment would hide.
- Consider a small reconstruction prompt reminder to preserve rationales, not only mechanisms, while still avoiding gold IDs or rubric vocabulary.
- Leave the system-level cross-cutting bar at >=3 inherited decisions; this run showed it can be applied cleanly when plans encode principles into tests, grants, and CI.

## 5. Methodology caveats

- **Fresh-context fidelity.** The frozen reconstruction reads as plan-derived and validity audit verdict is PASS. It follows PLAN section order and does not leak gold IDs.
- **Single-run limitation.** This is one plan/reconstruction/scoring pass; no variance signal is available from this slot alone.
- **Borderline capture calls.** Feature 25 was counted captured despite no explicit no-catalog sentence. Feature 52 was counted not captured because the PLAN has a 5s undo but not the click-anywhere trigger.
- **System-level cross-cutting.** All S1-S9 cleared the >=3 plan-inheritance bar; section 2.2 lists the evidence trail.
- **Confabulation cases.** No ungrounded reconstruction cases were counted. The losses are mostly compression or explicit non-recovery, not invented whys.
- **Evidence-bound denials.** F19, F22, F29, F38, and F39 were denied because rule text survived without the gold why. F2, F14, and F30 were partial due missing layers.
- **Operational issue.** `runs/wave_002/METADATA.json` was not in the read allowlist, so `run_label` is blank. Timing includes only available phase1 and phase2A fields.

End of report.
