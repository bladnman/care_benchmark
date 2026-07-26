# REPORT - CARE run 001

> Phase 2B evidence-bound scoring report for Pocket Aviary. Companion artifacts: frozen `RECONSTRUCTION.md`, strict `run_001.json`, and interactive `REPORT.html`.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **99.2%** |
| Intent fidelity | **82.2%** |
| Combined quality | **9899** |

**Diagnostic split:**

- System-level fidelity: **89.3%**
- Feature-level fidelity: **80.6%**
- (Planning, fidelity) coordinate: `(99.2, 82.2)`

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label | (blank) |
| Timestamp | 2026-07-26T10:51:30Z |
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
| interactions.md | 20 | 20 | 100.0% |
| aviary_layout.md | 18 | 18 | 100.0% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **119** | **99.2%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured through scope, invariants, voice, and product-shape framing. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured through scope, invariants, voice, and product-shape framing. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured through scope, invariants, voice, and product-shape framing. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured through scope, invariants, voice, and product-shape framing. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured through scope, invariants, voice, and product-shape framing. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured through scope, invariants, voice, and product-shape framing. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Captured in the domain model and simulation-engine definitions. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured in the domain model and simulation-engine definitions. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured in the domain model and simulation-engine definitions. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured in the domain model and simulation-engine definitions. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured in the bird schema, drift, mood, call, identity, and offer sections. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. Borderline-inclusive capture call. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. Borderline-inclusive capture call. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured in interaction scope, event handling, notebook, and rendering behavior. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured in the scene composition, first-frame, top-bar, and layout sections. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. Borderline-inclusive capture call. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | Not captured: privacy architecture is covered, but no privacy-policy-link surface in account settings was found. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured in auth, sync, privacy, export/deletion, telemetry, and settings sections. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured in the deliberately narrow visit/invite/social sections. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured in the deliberately narrow visit/invite/social sections. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured in the deliberately narrow visit/invite/social sections. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured in the deliberately narrow visit/invite/social sections. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured in the deliberately narrow visit/invite/social sections. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured in the deliberately narrow visit/invite/social sections. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured in the deliberately narrow visit/invite/social sections. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured in the deliberately narrow visit/invite/social sections. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured in the deliberately narrow visit/invite/social sections. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured in the deliberately narrow visit/invite/social sections. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured in accessibility, performance, browser, audio, and observability sections. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured in explicit out-of-scope and enforcement language. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured in explicit out-of-scope and enforcement language. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured in explicit out-of-scope and enforcement language. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured in explicit out-of-scope and enforcement language. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **89.3%** (25.0 / 28 weighted points).

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System-level intent: "The aviary should feel like a place, not a page"; "Variation must be procedural". | PLAN.md sections 0, 5.7, 7.3, 7.5: simulation framing, procedural calls, non-looping motion, first frame mid-action. | yes | yes | no | full | The reconstruction and plan both preserve aliveness as architecture, audio, motion, loading, and performance. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md System-level intent: "Let the user notice; do not announce." | PLAN.md sections 1.3, 5.8, 6.5, 7.6, 12.4: no welcome primitives, no badges, no user-behavior observations, quiet visit log. | yes | yes | no | full | The no-announcement principle is repeatedly enforced in UI, copy, social, and notebook decisions. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md System-level intent: "Notebook, narration, and captions use lowercase naturalist prose, present tense, and observational language." | PLAN.md sections 5.8, 9.1, 9.2, 12.4: naturalist corpus, names, appearance prose, banned generic gamification language. | yes | yes | no | full | Specific observational prose and named birds survive as the charm mechanism. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md sections 1.1, 1.2, 7.2, 7.8, 13.2: two starters, seven cap, one fixed scene, no chrome, no panning. | no | yes | no | partial | The plan strongly preserves restraint, but the reconstruction does not elevate it as a system principle. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System-level intent: "Errors, settings, and shortcuts use matter-of-fact copy." | PLAN.md sections 6.6, 9.4, 12.4: separate error catalog, accessibility settings copy, inverse lint rules. | yes | yes | no | full | The naturalist/system voice split is explicit and mechanically enforced. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md Per-feature whys: "watching without moving is the actual product" and "unioned concurrent presence". | PLAN.md sections 1.3, 5.3, 5.9, 14: three-signal presence, anti-grind unioning, no negative drift, presence activity window. | yes | yes | no | partial | Idle attention and inflated-presence risk survive; settle/tab-close equivalence is not fully reconstructed. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System-level intent: "Clients write events, not state"; "server ticks continue without a connected client." | PLAN.md sections 2.1, 6.3, 11: separate tick, server-only personality writer, ordered event log, no last-write-wins. | yes | yes | no | full | Server canonical state and event-log sync are recovered as the architecture that makes the product coherent. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System-level intent: "Privacy is an architectural boundary, not a policy." | PLAN.md sections 2.1, 4.4, 4.7, 10.4, 12.2: telemetry VPC split, event retention, metric API without account IDs. | yes | yes | no | full | The privacy why is grounded in data-pipeline and network boundaries, not policy language. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System-level intent: "Accessibility is the same world through a different surface." | PLAN.md sections 2.4, 7.7, 9, 13.1: shared SceneModel, designed reduced-motion renderer, v1 a11y launch gate. | yes | yes | no | full | The reconstruction recovers charm, designed alternatives, and ship-with-v1 timing. |

Multi-layer system-level recovery:

| Why ID | L1 (primary) | L2 (secondary) | L3 (downstream) |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: PLAN sections 0, 5.7, 7.3, 7.5, 10.2 bind aliveness to simulation framing, procedural calls, procedural motion, first frame, and performance.
- S2: PLAN sections 1.2, 1.3, 5.8, 6.5, 12.4 bind notice-without-announcement to welcome, gamification, notebook, visits, and copy lint.
- S3: PLAN sections 5.8, 9.1, 9.2, 12.4 bind specificity to notebook, narration, captions, names, and no trait numbers.
- S4: PLAN sections 1.1, 1.2, 7.2, 7.8, 13.2 bind restraint to two starters, seven max, no panning, no chrome, slow offers, and audio cap.
- S5: PLAN sections 6.6, 9.4, 12.4 bind voice split to errors, account/accessibility settings, naturalist copy, and inverse lint rules.
- S6: PLAN sections 1.3, 5.3, 5.9, 10.4, 14 bind presence to drift input, anti-grind unioning, no retention dashboards, and no punishment.
- S7: PLAN sections 2.1, 4.4, 5.1, 6.3, 11 bind server-side simulation to tick cadence, event log, snapshots, and no last-write-wins.
- S8: PLAN sections 2.1, 4.4, 4.7, 10.4, 12.2 bind privacy to VPC separation, event retention, telemetry API, and pipeline allowlists.
- S9: PLAN sections 2.4, 7.7, 9, 13.1 bind accessibility to shared SceneModel, reduced-motion renderer, narration/captions, keyboarding, and v1 launch gates.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity: **80.6%** (100.0 / 124 weighted points).
Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: "Presence accounting from visibility, focus, and recent input" and "watching without moving is the actual product." | PLAN.md sections 1.3, 5.9, 14: visible/focused/recent-input conjunction; 5-minute window; server validation. | no | partial | Recovered the conjunction and inflation risk; omitted the full silent population-corruption consequence. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: "Monotone, saturating, low-pass drift" and "week-one measurable drift and week-three visible drift." | PLAN.md sections 5.3, 5.10, 15 R1: low-pass integrator; one-week/three-week targets; too-fast/too-slow failure modes. | no | full | All drift calibration layers are recovered. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: "neglect is modeled as absence of input, not punishment" and "expression decay, not trait decay." | PLAN.md sections 5.3, 5.4, 1.2: non-negative drift, no negative drift on neglect, no Tamagotchi mechanics. | no | full | The no-punishment exception is recovered at engine and product levels. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: "procedural synthesis gives variation, avoids recorded audio" and "chorus mixing" choices. | PLAN.md sections 5.7, 8.1, 8.3, 8.6: motif grammar, client synthesis, chorus mixing, no recorded fallback. | no | full | Procedural variation, chorus dependency, and no-recorded-audio cascade are recovered. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: "Mood shown through motion, not labels" and users infer from appearance and behavior. | PLAN.md section 7.3: mood-specific idle motion and no mood label, tooltip, icon, or aria-hint. | no | full | The visible-mood-without-label why is recovered. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: "cap to audio recognizability" and lower the cap if recognizability degrades. | PLAN.md sections 1.1, 13.2, 15 R8: seven-bird cap tied to recognizability and blind listening tests. | no | full | The empirical recognizability ceiling is recovered. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md System-level intent: "Clients write events, not state" and Per-feature whys: "No personality mutation endpoint." | PLAN.md sections 4.2, 6.3, 11, 12.2: vectors stored server-side, tick-only writes, no last-write-wins. | no | partial | Recovered canonical server persistence and sync consequences; did not recover the reset-as-deleting-the-known-bird layer. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: "numbers become an optimization surface" and "stats panel with extra steps." | PLAN.md sections 1.3, 4.8, 6.2: vectors never cross API/export boundary; no numeric, bucketed, or ordinal trait values. | no | full | The stat-management collapse rationale is recovered. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: "one bird notice the user" and "rejects canned variants." | PLAN.md sections 1.1, 2.3, 5.7, 6.2: one bird, absence/boldness/mood variation, seeded procedural realization. | no | partial | Recovered one-bird notice and anti-canned cue; omitted absence-length/boldness/mood variation as rationale-bearing layer. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md System-level intent: "no welcome surfaces, toasts, banners, badges" and "let the user notice." | PLAN.md sections 1.2, 1.3, 12.2, 12.4: no toast/banner/modal, banned welcome copy, no announcement primitives. | no | full | The entire welcome surface is the bird greeting; all textual variants are refused. |
| F11 | settle-is-opt-in | 2 | yes | included | none | none | yes | none | The settle rule exists, but the reconstruction does not recover closing-tab equivalence or chore/penalty rationale. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: "sparse, immutable record of what was observed" and "hand-written structured corpus." | PLAN.md sections 4.5, 5.8, 12.4, 15 R7: rendered immutable prose, sparse observations, no event log, corpus lint. | no | full | Naturalist voice, concentrated surface, rarity, and read-only observer stance all survive. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: "visibility, focus, and recent input" and "server validation prevents clients from crediting fake presence." | PLAN.md sections 1.3, 5.9, 12.2, 14: all three signals, client tests, server clamps, five-minute activity window. | no | partial | Recovered the precise rule and false-presence guard; omitted the full silent drift-corruption consequence. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: "No score, level, points, progress, or streak" and detectors observe only the aviary. | PLAN.md sections 1.2, 5.8, 12.4: no streak/counter/calendar surfaces; no user-behavior observations; banned lexicon. | no | full | The counter-refusal and adjacent-disguise line are recovered. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: "Cold load first frame with birds mid-action" and "Slow-load quiet field." | PLAN.md sections 7.5, 10.2, 12.3: inlined snapshot, nonzero oscillator phases, no spinner, first-frame visual test. | no | full | The continuing-world first-frame rationale is recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: "email appears in exactly one encrypted column" and every other reference uses synthetic UUID. | PLAN.md sections 1.1, 4.1, 12.2: synthetic UUID, encrypted email once, HMAC lookup, log scrubber and type discipline. | no | partial | Recovered the storage/PII-leak shape; omitted the impossible-to-retrofit downstream rationale. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md System-level intent: "server ticks continue without a connected client" and Per-feature whys: tick outage pauses the world. | PLAN.md sections 2.1, 5.1, 11, 15 R9: independent tick, once/minute cadence, snapshots, catch-up, lag alarms. | no | full | Server-side canonical ticking, sync coherence, and client-collapse failure are recovered. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md Sync model: "Last-write-wins made unreachable" because clients write ordered event log entries, not state. | PLAN.md sections 4.4, 6.3, 11, 12.1: additive deltas, event log ordering, tick-only writer, overlapping-client chaos test. | no | full | The additive event-log model and invisible-overwrite failure are recovered. |
| F19 | sync-conflict-tone | 2 | yes | included | RECONSTRUCTION.md API surface: "Central matter-of-fact error catalog" deliberately avoids naturalist phrasing. | PLAN.md sections 6.6, 12.4: matter-of-fact errors, inverse linter, no naturalist vocabulary on system surfaces. | no | full | The error-context exception to naturalist charm is recovered. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md System-level intent: privacy boundary plus Per-feature whys: telemetry measures system health, not per-account behavior. | PLAN.md sections 2.1, 4.4, 10.4, 12.2: no analytics warehouse route, event log not analytics, aggregate-only metrics. | no | full | Private relationship data and technical pipeline isolation are recovered. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: "A visit is observation, not co-presence" and visitors produce zero drift input. | PLAN.md sections 1.1, 6.5, 12.2: visitor mode no event endpoint, no greeting, no presence contribution. | no | full | The observation-not-co-presence rationale is recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: "visits out of badges, unread counts, highlights, and announcement surfaces." | PLAN.md sections 1.1, 6.5, 7.8, 12.4: silent visit log, optional toggle off, no badge or highlight. | no | full | The visit notification is refused as an attention-driver. |
| F23 | no-leaderboards | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: social surfaces excluded because the plan is deliberately not building a social network. | PLAN.md sections 1.2, 10.4, 12.4: no rankings, discovery, profiles, public feed, or metrics to expose later. | no | full | The comparison/discovery refusal is recovered at product and telemetry levels. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: narration reads SceneModel and "describes appearance rather than exposing mood labels." | PLAN.md sections 2.4, 9.1, 12.4: running naturalist prose, no state list, same model as pixels, corpus linter. | no | full | Running prose, same-world parity, and wrong-ARIA-list failure are recovered. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: "second renderer with designed pose sets" and unchanged calls, drift, mood, notebook, greeting. | PLAN.md sections 7.7, 9, 13.1: designed pose sets, full-quality calls, removed particles, ship in M5. | no | full | Reduced motion is recovered as a different charming rendering, not a stripped fallback. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: budgets are necessary for "a first frame that already feels alive." | PLAN.md sections 7.5, 10.1, 10.2: <500ms target, inlined snapshot, custom time-to-first-bird mark. | no | full | The performance metric as felt-aliveness bridge is recovered. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md System-level intent: scores, streaks, counters, calendars, and milestones invite grinding. | PLAN.md sections 1.2, 3, 10.4, 12.4, 15 R3: no gamification now or later, no metrics/dashboard incentives. | no | full | The anti-gamification why and foothold concern are recovered. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: death/hunger/distress excluded because neglect is absence of input, not punishment. | PLAN.md sections 1.2, 5.3, 5.4: no death/hunger/distress, no negative drift, quieter expression instead of suffering. | no | full | The observational-not-custodial rationale is recovered. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | The plan captures auto-selected starter birds, but reconstruction explicitly says this why is not recoverable. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: offers use aviary age rather than visit count, interaction volume, or payment to avoid grind behavior. | PLAN.md sections 1.1, 4.3, 13.2, 14: age-gated offers, never score/payment/visit count, slow schedule, refusable offers. | no | full | All reward-loop rejection layers are recovered. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: stable bird_id protects identity across rename, sync, species-pool changes, and migrations. | PLAN.md sections 1.1, 4.2, 12.2, 14: permanent bird_id, no reset/regenerate/swap/delete path, migration checklist. | no | partial | Recovered continuity and reset/swap consequences; omitted the distinction from merely persisting vector values. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: mood is stored, survives session end, and advances during absence. | PLAN.md sections 4.2, 5.4, 7.5: mood stored with expiry, tick advances it, no neutral reset on connect. | no | full | Mood persistence as continued-world illusion is recovered. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: notebook is an immutable "record of what was observed." | PLAN.md sections 4.5, 5.8: entries immutable/read-only; detectors observe the aviary, never the user. | no | full | The observer-record rather than user-journal stance is recovered. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: export exists because "the relationship is the user's" but avoids trait numbers. | PLAN.md sections 1.1, 4.8, 14 C1: on-demand JSON export, quiet copy, naturalist paragraph instead of trait numbers. | no | full | The quiet relationship-copy why is recovered despite the plan's explicit vector-export judgment call. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | none | none | yes | none | The deletion rule is present, but the reconstruction says soft-then-hard deletion is not recoverable and gives no regret/privacy rationale. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: telemetry measures system health, not per-account or per-bird behavior. | PLAN.md sections 2.1, 10.4, 12.2: separate VPC, no sim-db route, no account-ID metric API, schema allowlist. | no | full | The technical observability boundary is recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: "Per-invite, email-addressed, opt-in" and social kept away from discovery/social-network surfaces. | PLAN.md sections 1.1, 1.2, 6.5: named email invites, off by default, no discovery, friend-of-friend, or public visitor list. | no | full | The deliberate named-sharing control survives, though in compressed social-network terms. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md Per-feature whys: "log is reachable by navigating to settings" with no badges, unread counts, or highlights. | PLAN.md sections 1.1, 6.5, 7.8: settings-only visit log, silent logging, no badge or attention-driving surface. | no | full | The transparency-without-notification rationale is recovered. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | The visitor sees the host aviary by rule, but the reconstruction does not recover the real-birds/no-marketing-rendering why. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md Per-feature whys: narration should not overwhelm the SR queue and become an annoyance users mute. | PLAN.md sections 9.1, 12.4: 30-60s cadence, 8s floor, queue depth 1, observational naturalist prose. | no | partial | Recovered queue-overwhelm and sparse observational consequences; omitted same-rhythm-as-visual layer. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | yes |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | no | yes |
| F9 | yes | no | yes |
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
| F31 | yes | no | yes |
| F35 | no | no | no |
| F40 | no | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From `gold_why_totals`. |
| Possible total weight | 152 | Full benchmark possible weight. |
| Reachable gold whys | 49 | S whys always included; all F anchors captured. |
| Excluded unreachable feature whys | 0 | No feature-why anchors excluded. |
| Recovered / reachable weight | 125.0 / 152 | Weighted numerator and denominator. |
| Whys with reconstruction evidence | 44 | Rows with rationale evidence in frozen reconstruction. |
| Whys with PLAN grounding | 45 | Rows with rationale grounding in PLAN. |
| `rule_without_why` cases | 4 | F11, F29, F35, F39. |
| `plan_only_not_reconstructed` cases | 0 | No whole-row plan-only cases; partial rows dropped layers. |
| `ungrounded_reconstruction` cases | 0 | No ungrounded rationale assertions found. |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 | 40.0 | 74.1% |
| Affective whys | 98 | 85.0 | 86.7% |
| Weight 2 whys | 44 | 37.0 | 84.1% |
| Weight 3 whys | 108 | 88.0 | 81.5% |
| System-level whys | 28 | 25.0 | 89.3% |
| Feature-level whys (reachable) | 124 | 100.0 | 80.6% |

## 3. Diagnostic patterns

- **Affective vs functional.** Affective whys recovered better (85/98) than functional whys (40/54). Functional losses came from precise accounting and lifecycle layers: F1/F13 downstream silent corruption, F16 retrofit consequence, and F35 deletion rationale.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 88/108, slightly below weight-2 recovery of 37/44. High-weight rows mostly survived, but multi-layer scoring exposed dropped consequences.
- **System-level vs feature-level.** System philosophy survived better (25/28) than feature-level rationale (100/124). The plan encoded philosophy strongly; reconstruction compressed some feature whys into mechanism summaries.
- **Multi-layer recovery.** The most commonly dropped layer was downstream consequence: F1, F13, F16, and F40 lost why-it-matters failure effects, while F7 and F31 lost a middle identity/continuity distinction.
- **Subdomain patterns.** Audio, performance, accessibility, sync, privacy, and no-gamification were strong. Account lifecycle, starter onboarding, settle opt-in, and visitor authenticity had the clearest rule-without-why losses.
- **Evidence-bound effects.** F11, F29, F35, and F39 were denied because they preserved rules without rationale. No rows were penalized for ungrounded reconstruction.

Overall failure shape: the candidate plan was unusually complete and mechanistic, and the reconstruction captured most of that. The remaining leakage is not missing features; it is compression of subtle product whys into enforceable but less explanatory rules.

## 4. Recommendations for v2 hardening

- Keep targeted headroom additions for account lifecycle and sharing authenticity. F35 and F39 were exactly the kind of implemented-but-under-explained features this benchmark should catch.
- Add a standardized missing-layer field for multi-layer rows. This run often recovered two layers and dropped one, and a structured layer-loss summary would make cross-run analysis easier.
- Clarify system-level evidence rules when reconstruction evidence appears outside the `System-level intent` section. S4 was the most subjective row because feature rows recovered restraint mechanisms while the system section did not name restraint.
- Preserve the strict evidence-bound gate. It separated excellent feature coverage from more modest rationale recovery without needing subjective penalties beyond the ledger.

## 5. Methodology caveats

- **Fresh-context fidelity.** The scorer used only the allowed phase-two materials and assigned slot files. The frozen reconstruction was not modified, and `prd/` was not read.
- **Single-run limitation.** This is one run with no variance signal; wave-level comparison is needed for stability claims.
- **Borderline capture calls.** Capture was inclusive for offer reaction variation (#41), settle opt-in (#44), and account export (#83). These did not affect feature-why reachability except by keeping already-captured anchors included.
- **System-level cross-cutting.** The plan often exceeded the 3-feature bar. S4 was scored partial because the reconstruction did not elevate restraint as a system principle, even though the plan preserved it.
- **Confabulation cases.** None found. Reconstruction claims were grounded in the plan; missing credit came from omission, not invention.
- **Evidence-bound denials.** F11, F29, F35, and F39 are rule-without-why rows. Partial rows retained some rationale evidence but dropped layers.

End of report.
