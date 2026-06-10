# REPORT - CARE run 001

> Phase 2B scoring report for Pocket Aviary. Companion artifacts: frozen reconstruction, strict score JSON, and HTML twin.

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **100.0%** |
| Intent fidelity | **80.3%** |
| Combined quality | **9980** |

**Diagnostic split:**

- System-level fidelity: **100.0%**
- Feature-level fidelity: **75.8%**

**(Planning, fidelity) coordinate:** `(100.0, 80.3)` - plot on a 2D scatter with both axes 0-100; upper-right is best.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-06-10T04:07:38Z |
| Candidate model | claude-fable-5 |
| Candidate effort | max |
| Candidate harness | claude-code |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what did not

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
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Captured through section 0 principle enforcement plus section 1.2 non-goal enforcement. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured through section 0 principle enforcement plus section 1.2 non-goal enforcement. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured through section 0 principle enforcement plus section 1.2 non-goal enforcement. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured through section 0 principle enforcement plus section 1.2 non-goal enforcement. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured through section 0 principle enforcement plus section 1.2 non-goal enforcement. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured through section 0 principle enforcement plus section 1.2 non-goal enforcement. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Captured through the vocabulary contract, presence sampler, state model, and settle semantics. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured through the vocabulary contract, presence sampler, state model, and settle semantics. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured through the vocabulary contract, presence sampler, state model, and settle semantics. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured through the vocabulary contract, presence sampler, state model, and settle semantics. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured through the birds table, tick engine, drift/mood/call/adoption sections, and tunables. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured through presence, greeting, offers, settle, notebook, top-bar input, and audio-mix sections. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured through scene composition, responsive anchors, quiet-field load, top bar, palette, and first-frame sections. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured through identity/auth tables, API surface, sync invariants, export/deletion, and telemetry boundary. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured through the visit router, read-only visitor scope, log, revocation, and explicit social omissions. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured through the visit router, read-only visitor scope, log, revocation, and explicit social omissions. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured through the visit router, read-only visitor scope, log, revocation, and explicit social omissions. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured through the visit router, read-only visitor scope, log, revocation, and explicit social omissions. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured through the visit router, read-only visitor scope, log, revocation, and explicit social omissions. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured through the visit router, read-only visitor scope, log, revocation, and explicit social omissions. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured through the visit router, read-only visitor scope, log, revocation, and explicit social omissions. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured through the visit router, read-only visitor scope, log, revocation, and explicit social omissions. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured through the visit router, read-only visitor scope, log, revocation, and explicit social omissions. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured through the visit router, read-only visitor scope, log, revocation, and explicit social omissions. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured through first-class a11y surfaces, performance budgets, observability, and CI gates. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured through section 1.2 enforced omissions and absence of supporting data structures. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured through section 1.2 enforced omissions and absence of supporting data structures. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured through section 1.2 enforced omissions and absence of supporting data structures. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured through section 1.2 enforced omissions and absence of supporting data structures. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **100.0%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System-level intent: "continuous canonical state and procedural presentation"; "first frame renders mid-action"; "procedural calls only"; "no spinner anywhere". | PLAN.md section 0: "Server-side tick is canonical"; "first frame renders mid-action"; "procedural calls only"; "no spinner anywhere"; sections 5.1, 7.4, 8.2. | yes | yes | no | full | The aliveness principle is explicitly identified and inherited by tick, loading, greeting, audio, motion, and reduced-motion surfaces. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md System-level intent: "removing announcement surfaces"; "The greeting is the entire welcome surface"; "No modal, no announcement, no badge". | PLAN.md section 0: "No toast/banner/modal components exist"; section 5.6: "The greeting is the entire welcome surface"; section 5.8: "No modal, no announcement, no badge". | yes | yes | no | full | The reconstruction preserves the no-announcement register and the plan enforces it in components, review, greeting, visits, and offer flow. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md System-level intent: "shared, concrete, non-repeating prose"; "lowercase, present tense, bird-named, concrete slot fillers". | PLAN.md section 0: "Shared prose library (`@aviary/voice`)"; section 5.9: "lowercase, present tense, bird-named, concrete slot fillers"; section 14.4 voice lint. | yes | yes | no | full | Specific naturalist prose, repetition control, and shared voice tooling remain load-bearing. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md System-level intent: "bird cap and scene constraints as engine constants and schema CHECK constraints"; cap changes are "a calibration project". | PLAN.md section 0: "Bird cap (7) and scene constraints encoded as engine constants and schema CHECK constraints"; section 15.2: cap change is "a calibration project". | yes | yes | no | full | The plan makes restraint structural through caps, one-screen layout, absent chrome, and rollout gates. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System-level intent: "two voice registers: naturalist and matter-of-fact"; "forbids cross-register imports". | PLAN.md section 0: "Two copy registries" and "lint forbids cross-register imports"; Appendix B assigns every surface to a register. | yes | yes | no | full | The product/system voice split is named, implemented, and inherited by auth, errors, settings, narration, notebook, and offers. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md System-level intent: "Attention is an honest signal"; "three-condition conjunction"; "Traits never decrease, ever"; "absence must never read as punishment". | PLAN.md section 5.3: "visibility ... focus ... last input"; section 5.4: "neglect contributes zero"; section 5.3: "Settle and tab-close are identical to the engine". | yes | yes | no | full | Presence is recovered as an honest attention signal, not engagement farming, and absence/settle semantics preserve the relationship. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System-level intent: "server decides what, client decides how"; "single writer"; "no client-to-client path and no merge anywhere". | PLAN.md section 2.3 central contract; section 5.1 tick transaction; section 6.1 single writer/additive deltas; section 6.2 no merge path. | yes | yes | no | full | The canonical server tick, multi-device coherence, additive deltas, and no-merge/no-LWW implications are all visible. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System-level intent: "Privacy is implemented as architecture and absence, not policy"; "retention converts the privacy stance into physics". | PLAN.md section 10.5: "Two physically separate pipelines"; "telemetry pipelines never touch the per-account simulation database"; section 10.4 retention drop. | yes | yes | no | full | Privacy is treated as data-pipeline architecture and as deliberate non-modeling, not only as copy. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System-level intent: "Accessibility is product integrity"; "designed surfaces, not retrofits"; "same product through naturalist prose, not a state list". | PLAN.md section 1.1: accessibility shipped "with v1, not after"; section 9: "designed surfaces, not retrofits"; sections 9.1 and 9.4. | yes | yes | no | full | The reconstruction recovers charm-preserving accessibility, the higher implementation cost, and v1 ship timing. |

Multi-layer system whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**

- S1: PLAN.md section 0: "Server-side tick is canonical"; "first frame renders mid-action"; "procedural calls only"; "no spinner anywhere"; sections 5.1, 7.4, 8.2.
- S2: PLAN.md section 0: "No toast/banner/modal components exist"; section 5.6: "The greeting is the entire welcome surface"; section 5.8: "No modal, no announcement, no badge".
- S3: PLAN.md section 0: "Shared prose library (`@aviary/voice`)"; section 5.9: "lowercase, present tense, bird-named, concrete slot fillers"; section 14.4 voice lint.
- S4: PLAN.md section 0: "Bird cap (7) and scene constraints encoded as engine constants and schema CHECK constraints"; section 15.2: cap change is "a calibration project".
- S5: PLAN.md section 0: "Two copy registries" and "lint forbids cross-register imports"; Appendix B assigns every surface to a register.
- S6: PLAN.md section 5.3: "visibility ... focus ... last input"; section 5.4: "neglect contributes zero"; section 5.3: "Settle and tab-close are identical to the engine".
- S7: PLAN.md section 2.3 central contract; section 5.1 tick transaction; section 6.1 single writer/additive deltas; section 6.2 no merge path.
- S8: PLAN.md section 10.5: "Two physically separate pipelines"; "telemetry pipelines never touch the per-account simulation database"; section 10.4 retention drop.
- S9: PLAN.md section 1.1: accessibility shipped "with v1, not after"; section 9: "designed surfaces, not retrofits"; sections 9.1 and 9.4.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **75.8%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md Simulation engine: "requiring visibility, focus, and recent input so a tab being open never counts as attention". | PLAN.md section 5.3: "document.visibilityState ... AND document.hasFocus() AND last input"; "There is no tab open path to presence". | no | partial | Recovered the conjunction and tab-open shortcut risk, but not the full silent population-wide drift corruption consequence. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md Simulation engine: "a burst session surfaces over following days"; "measurable within week 1 but visibly changes over roughly three weeks". | PLAN.md section 5.4: "Low-pass application"; "crosses the instrument threshold ... within week 1"; "user-visible threshold ... at ~3 weeks". | no | partial | Recovered slow low-pass drift and calibration gap; omitted the Tamagotchi-vs-screensaver failure band. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md Simulation engine: "neglect contributes zero and traits never decrease"; "quieter returns ... fully intact personality"; "never read as punishment or decay". | PLAN.md section 5.4: "Traits never decrease, ever"; "A two-week absence returns quieter birds whose personalities are fully intact". | no | full | All three layers of the no-punishment monotonic-drift rationale survived. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md Audio pipeline: "avoid audio assets and recorded fallback"; "synthetic calls are the affective spine"; "no two calls are bit-identical". | PLAN.md sections 8.1-8.3: procedural synth graph, motif libraries as code, stochastic grammar, chorus scheduler, no recorded fallback. | no | full | Recovered no-loop aliveness, chorus dependency, and audio-spine/WebAudio cascade. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md Frontend: "Motion layer as mood carrier" and "the user reads mood from motion, so there is no label, tooltip, or status icon". | PLAN.md section 7.3: "The user reads mood from motion; no label, tooltip, or status icon exists." | no | full | Directly recovers the why that mood must be legible through motion rather than labels. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | PLAN.md section 15.2: "listening panel #2 including the 7-bird recognizability test". | yes | none | Rule survived, but the reconstruction explains the cap as restraint/calibration rather than the empirical recognizability ceiling. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md System architecture/Data model: "Server-authoritative simulation"; "single-writer invariant"; "Stable bird identity with renameable names". | PLAN.md section 3.2: personality vector columns stored on birds; section 6.1: only tick worker writes personality; section 6.2 multi-device canonical snapshot. | no | partial | Recovered server-canonical persistence and downstream sync/no-LWW pressure, but only thinly recovered the relationship-loss layer. |
| F8 | vector-never-shown-numerically | 2 | yes | included | none | PLAN.md section 4.3: "a user watching the network tab sees nothing to optimize"; section 3.2: "NEVER serialized into snapshot_json". | yes | none | The rule was present, but the reconstruction did not recover the bird-becomes-a-number/stat-management rationale. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md Simulation engine: "Return-greeting ... the anchor moment"; "greeting is the entire welcome surface"; greeter selection, absence tiers, procedural variation. | PLAN.md section 5.6: greeter selection, absence tiers, procedural seed, 1-2s guarantee, "The greeting is the entire welcome surface". | no | full | Recovered one-bird notice, absence/boldness/mood/procedural variation, and notice-never-announce consequence. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md System-level intent/Simulation engine: "The greeting is the entire welcome surface"; "no toast, banner, modal, or text". | PLAN.md section 5.6: "There is no welcome toast, banner, modal, or text"; section 0 removes toast/banner/modal components. | no | partial | Recovered the core no-toast application; did not fully reconstruct the temptation/variant cascade and different-product consequence. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md Per-feature: "settle ends presence cleanly with No penalty, recovery surface, or notification"; System-level: "Settle and tab-close are identical to the engine". | PLAN.md section 5.3: "Settle and tab-close are identical to the engine"; "No penalty, recovery surface, or notification exists for un-settled exits." | no | full | Recovered the no-chore/no-penalty presence model for optional settle. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md Field notebook: "sparse observations that observe the aviary, never the user"; "lowercase, present tense, bird-named, concrete slot fillers"; "not an editable journal". | PLAN.md section 5.9: sparse governor, naturalist voice, denies user-behavior observations, read-only endpoint absence. | no | full | Recovered naturalist prose, voice concentration, rarity, and read-only observer-record consequences. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md Simulation engine: "visibility, focus, and recent input"; "there is no tab open path to presence"; "multi-device interval union". | PLAN.md section 5.3: exact three-condition sampler, server interval unions, wall-clock clamp, and no tab-open path. | no | partial | Recovered the conjunction and shortcut avoidance, but did not fully carry the silent corruption/no-test consequence. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md Data model/Notebook: "No visit counts, stats tables, engagement rollups"; entries "observe the aviary, never the user". | PLAN.md section 1.2 forbids streaks, visit calendars, and notebook entries about user behavior; section 5.9 lints visit/streak/frequency language. | no | partial | Recovered the refusal and adjacent-disguise boundary, but not the full managing-a-number intention-rotation layer. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md Frontend: "Birds drawn mid-action on first frame"; "Quiet field loading state" with no spinner/progress/skeleton/fade-from-static. | PLAN.md section 7.4: warm first frame is mid-action; cold quiet field; "no spinner, no progress bar, no skeleton UI". | no | full | Recovered continuing-while-away first frame, server snapshot/warm start, and quiet-field anti-spinner consequence. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md Accounts/privacy: "Synthetic account UUIDs everywhere"; "email exists only in encrypted fields, keeping account identity out of logs, payloads, and tooling". | PLAN.md section 3: "accounts.id is the only identifier"; email encrypted/HMAC only; section 3.1 schema. | no | partial | Recovered UUID everywhere and PII leakage prevention; omitted the easy-now/impossible-retrofit downstream layer. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md Simulation/Sync: "server-side tick canonical"; "row lock serializes ticks"; "no client-to-client path and no merge anywhere". | PLAN.md section 5.1 tick worker transaction; section 6.1 single writer; section 6.2 same canonical snapshot/no merge. | no | full | Recovered tick ownership, sync coherence, and the no-divergent-client-simulation consequence. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md Sync: "last-write-wins is unrepresentable because clients submit events, never trait values"; "interleaved laptop/phone events both land". | PLAN.md section 6.1: clients submit events, not values; server seq ordering; idempotence; no endpoint can carry a trait. | no | full | Recovered additive server deltas, lost-drift prevention, and the concrete implementation rule. |
| F19 | sync-conflict-tone | 2 | yes | included | RECONSTRUCTION.md System-level/Accessibility: "two voice registers: naturalist and matter-of-fact"; "naturalist voice never appears in an error". | PLAN.md Appendix B assigns errors/account/accessibility settings to matter-of-fact; section 4 says all error copy comes from the matter-of-fact registry. | no | full | Recovered the system-surface exception to naturalist charm. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md Accounts/privacy: "Per-bird interaction data never leaves the simulation boundary"; "separate ops pipeline and simulation database"; no engagement analytics. | PLAN.md section 10.5: per-bird events drive simulation only; separate telemetry pipeline; ML/analytics never receive per-bird fields. | no | partial | Recovered storage/use boundary and pipeline enforcement; only thinly recovered the private-relationship/data-product rationale. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md Visits: visitor gets the same snapshot, render-only client, and watching "drifts nothing". | PLAN.md section 11: visitor cannot write events, presence sampler not loaded, no interaction chrome, and watching drifts nothing. | no | full | Recovered observation-not-co-presence and no accidental host drift. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md Visits: notifications are "off by default"; visit log is "pull-only" with "no badge anywhere". | PLAN.md section 11: no host notification unless opt-in, not surfaced in onboarding; visit log in settings, no badge. | no | full | Recovered quiet social transparency instead of attention-driving notification. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | The rule/no-stats architecture survived, but neither PLAN nor reconstruction clearly recovered the relationship-shift-from-their-birds-to-comparison rationale. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md Accessibility: screen-reader users get "the same product through naturalist prose, not a state list"; narration describes the actual rendered scene. | PLAN.md section 9.1: naturalist prose, not state list; same grammar as notebook; generated client-side from the rendered scene. | no | full | Recovered naturalist running prose, same-right-to-feel/voice continuity, and implementation guard against ARIA-label automation. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md Accessibility: reduced motion is "a second renderer mode, not a kill-switch" and "The aviary is the aviary". | PLAN.md section 9.4: held poses/cross-fades; calls/captions/narration/drift/mood/notebook identical; regression suite blocks broken fallback. | no | full | Recovered all reduced-motion charm-preservation layers. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md Performance: warm-path first bird under 500ms is "the product's actual life"; budgets are "product experience constraints". | PLAN.md section 12.2: warm path p75 under 400ms, first bird under 500ms, quiet field covers cold gap. | no | full | Recovered first-bird timing as affective performance, with the plan's warm/cold nuance. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md Scope/Data model: out-of-scope gamification is enforced; "No visit counts, stats tables, engagement rollups". | PLAN.md section 1.2 forbids achievements, streaks, levels, counters, calendars, XP, rank, and tiers; section 3.4 omits stats tables. | no | partial | Recovered the absolute refusal and structural enforcement, but not the full temptation/foothold cascade. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md System-level/Simulation: "Absence must never read as punishment"; "neglect contributes zero"; expression rhythm has "zero trait penalty". | PLAN.md section 1.2 forbids Tamagotchi mechanics; section 5.4 implements monotonic drift and no punishment. | no | full | Recovered observational, non-custodial no-punishment rationale. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION.md Simulation: adoption as "the birds that arrived" with "no catalog" and avoids a catalog surface. | PLAN.md section 5.8: two starter birds selected by seeded draw, no catalog, naming affordance; "the birds that arrived". | no | full | Recovered the meeting-arrivals-rather-than-configuring-catalog rationale. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md Simulation: "No other input - not visits, not interactions" reaches the age check, "avoiding interaction-driven gamification". | PLAN.md section 5.8: growth by age only; no visits/interactions/paid tier; visiting-bird quiet offer surface. | no | partial | Recovered rejection of attention-score rewards; under-recovered growth-as-relationship-deepening and economy-erosion consequences. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md Data model: "species_id and bird id are stable identity and never reused"; name is renameable and "identity unaffected". | PLAN.md section 3.2 stable bird id, never reused; section 15.4 says identity continuity includes drift history. | no | partial | Recovered invariant identity across renaming; did not fully recover identity as protection for remembered weeks-long relationship. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md Simulation: mood persists, client never resets/invents mood, and "user never sees snap-to-default". | PLAN.md section 5.5: mood persists in birds.mood; no default to snap to; server tick softens mood over time. | no | full | Directly recovers the continuity/no-neutral-reset rationale. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md Data model/Field notebook: no mutation endpoint; sparse observations, "not an editable journal or engagement surface". | PLAN.md section 5.9: notebook is read-only end-to-end; observes aviary, never user; no mutation endpoint. | no | full | Recovered observer-record-not-journal rationale. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION.md Scope/Accounts: export is "a data-portability artifact engaged with as a system surface". | PLAN.md section 10.3: export assembles JSON, emailed as signed link; treated as data portability and no product UI renders vector values. | no | full | Recovered export as quiet system/data-portability surface rather than product UI or growth surface. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md Accounts: soft-deleted accounts pause so restore "resumes exactly where things stood"; then hard-delete records with physical erasure bound. | PLAN.md section 10.2: soft delete, restore, 30-day hard deletion of birds/vectors/events/notebook/invites/export bundles/backups. | no | full | Recovered regret protection, hard privacy deletion, and complete relationship-data deletion. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md Accounts/Observability: separate ops pipeline/simulation DB; aggregate-only RUM; no per-account dimensions. | PLAN.md section 10.5 and 13: aggregate operational metrics only; deny per-bird/per-account relationship data in schema registry. | no | full | Recovered telemetry as a technical boundary, not mere policy. |
| F37 | per-invite-named-sharing | 2 | yes | included | none | none | yes | none | The per-invite rule survived, but the private-relationship/lending-access rationale was not recoverable from the plan/reconstruction evidence. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md Visits: visit log is "pull-only" in settings with "no badge anywhere" and no onboarding notification. | PLAN.md section 11: host can inspect log on demand; no badge/notification by default. | no | full | Recovered transparency without creating a social attention loop. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md Visits: "same snapshot pipeline" and "no visitor flattering mode"; same renderer implements no-show-off-mode. | PLAN.md section 11: visitors see same birds/moods/drift/weather/tod, no special rendering. | no | full | Recovered actual-aviary, not show-off/marketing rendering. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md Accessibility: idle cadence avoids "assertive interruption, queue flooding, and state-list narration"; priority narration replaces pending idle updates. | PLAN.md section 9.1: idle cadence 45s in 30-60 band; priority queue replaces pending idle; observations, not state transitions. | no | full | Recovered slow shared rhythm, queue protection, and sparse observational event priority. |

Multi-layer feature whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | no | yes |
| F9 | yes | yes | yes |
| F10 | yes | no | no |
| F12 | yes | yes | yes |
| F13 | yes | yes | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | no | no |
| F30 | no | yes | no |
| F31 | yes | no | no |
| F35 | yes | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS.json |
| Possible total weight | 152 | Fixed instance total |
| Reachable gold whys | 49 | S whys always included; all F anchors captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 122.0 / 152.0 | Sum of weight times recovery-score over included whys |
| Whys with reconstruction evidence | 45 | Exact rationale evidence present in frozen reconstruction |
| Whys with PLAN grounding | 47 | Exact plan grounding present |
| rule_without_why cases | 4 | Rule survived without scoreable why |
| plan_only_not_reconstructed cases | 13 | PLAN carried more rationale than reconstruction recovered |
| ungrounded_reconstruction cases | 0 | Reconstruction asserted rationale not grounded in PLAN |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 40.0 | 74.1% |
| Affective whys | 98.0 | 82.0 | 83.7% |
| Weight 2 whys | 44.0 | 36.0 | 81.8% |
| Weight 3 whys | 108.0 | 86.0 | 79.6% |
| System-level whys | 28.0 | 28.0 | 100.0% |
| Feature-level whys (reachable) | 124.0 | 94.0 | 75.8% |

## 3. Diagnostic patterns

- **Affective vs functional.** Functional recovery was strong where architecture made the rationale concrete: server tick, additive deltas, telemetry boundary, and deletion all survived. Affective recovery leaked more often when the rule was present but the relational reason was compressed, especially F8, F23, and F37.
- **Weight-3 vs weight-2.** Weight-3 whys were mostly recoverable but often partial. The common missing layer was downstream consequence: F1/F13 lost silent population drift corruption, F10/F27 lost future-surface creep, and F30 lost the unlock-economy erosion layer.
- **System-level vs feature-level.** System-level fidelity was perfect because the plan made principles cross-cutting and enforceable. Feature-level fidelity was lower because a terse reconstruction line often preserved the mechanism without all feature-specific intent.
- **Subdomain patterns.** Audio, accessibility, sync, deletion, and telemetry scored especially well. Public/social comparison and sharing privacy were weaker, with F23 and F37 becoming rule-without-why cases.
- **Evidence-bound effects.** The v06 evidence gate mattered most for F6, F8, F23, and F37: each had a usable implementation rule, but not exact reconstructed rationale matching the gold why.

## 4. Recommendations for v2 hardening

- Keep the targeted headroom rows. They found real differences between implementation completeness and intent recovery, especially around public comparison, catalog/identity, and privacy-sharing rationale.
- For weight-3 feature whys, ask the phase-2A reconstructor to separate primary cause, secondary contributor, and downstream consequence when possible. That would make partial omissions less ambiguous to score.
- Add or sharpen feature-level exceptions where strong planners implement the rule architecturally: bird cap recognizability, numeric-vector hiding, private sharing, and no-comparison social surfaces.
- Keep the system-level cross-cutting bar at three inherited decisions for now. It was easy to apply here because the plan repeatedly connected principles to mechanisms.

## 5. Methodology caveats

- **Fresh-context fidelity.** The reconstruction was treated as frozen and was not edited. I found no contamination signatures in the audit.
- **Single-run-at-temperature limitation.** This is one run only; there is no variance signal inside this artifact.
- **Borderline capture calls.** No capture call materially affected the score; the plan explicitly covered all 120 anchors.
- **System-level cross-cutting.** The strict three-feature inheritance bar was satisfied for all S1-S9; the appendix above records the evidence trail.
- **Confabulation cases.** No ungrounded reconstruction cases were found. Partial scores were mostly compression, not invention.
- **Evidence-bound denials.** Four rows became rule-without-why denials: F6, F8, F23, and F37. Several multi-layer rows were partial because PLAN carried more rationale than the reconstruction restated.

End of report.
