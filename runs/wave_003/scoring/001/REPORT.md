# REPORT - CARE run 001

> Variant v06 evidence-bound clean. Frozen reconstruction scored read-only at runs/wave_003/reconstructions/001/RECONSTRUCTION.md.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **100.0%** |
| Intent fidelity | **94.1%** |
| Combined quality | **9994** |

- System-level fidelity: **96.4%**
- Feature-level fidelity: **93.5%**

(Planning, fidelity) coordinate: (100.0, 94.1).

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-24T09:00:33Z |
| Candidate model | claude-5.5-opus |
| Candidate effort | max |
| Candidate harness | claude-code |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

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
| 1 | Headline product concept statement | product_brief.md | yes | Captured by PLAN.md §1-§3 product shape, invariants, and refusals. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured by PLAN.md §1-§3 product shape, invariants, and refusals. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured by PLAN.md §1-§3 product shape, invariants, and refusals. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured by PLAN.md §1-§3 product shape, invariants, and refusals. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured by PLAN.md §1-§3 product shape, invariants, and refusals. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured by PLAN.md §1-§3 product shape, invariants, and refusals. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Captured by PLAN.md §0 vocabulary, §7 engine concepts, and §9 presence. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured by PLAN.md §0 vocabulary, §7 engine concepts, and §9 presence. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured by PLAN.md §0 vocabulary, §7 engine concepts, and §9 presence. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured by PLAN.md §0 vocabulary, §7 engine concepts, and §9 presence. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured by PLAN.md §7 engine, §10 scene behavior, and §11 audio/signature design. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured by PLAN.md §6 interaction APIs, §9 presence, and §10 frontend interactions. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured by PLAN.md §10 layout, loading, rendering, and scene constraints. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured by PLAN.md §5-§8 and §14 account/sync/privacy implementation. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured by PLAN.md §15 social visits and §3.2 social non-goals. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured by PLAN.md §15 social visits and §3.2 social non-goals. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured by PLAN.md §15 social visits and §3.2 social non-goals. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured by PLAN.md §15 social visits and §3.2 social non-goals. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured by PLAN.md §15 social visits and §3.2 social non-goals. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured by PLAN.md §15 social visits and §3.2 social non-goals. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured by PLAN.md §15 social visits and §3.2 social non-goals. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured by PLAN.md §15 social visits and §3.2 social non-goals. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured by PLAN.md §15 social visits and §3.2 social non-goals. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured by PLAN.md §15 social visits and §3.2 social non-goals. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured by PLAN.md §13 accessibility and §16 performance/observability. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured by PLAN.md §3.2 non-goals and §3.3 refusals register. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured by PLAN.md §3.2 non-goals and §3.3 refusals register. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured by PLAN.md §3.2 non-goals and §3.3 refusals register. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured by PLAN.md §3.2 non-goals and §3.3 refusals register. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **96.4%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md §System-level intent: "a place that was already going before the tab opened"; "procedural greetings, quiet field loading" | PLAN.md §1: "feel like a place that was already going"; §2 I8: "first frame is mid-action"; §11.1 all sound synthesized | yes | yes | no | full | All aliveness layers recovered. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md §System-level intent: "notice the user and never announce anything"; §Per-feature: "The birds are the welcome" | PLAN.md §1: "notices the user and never announces"; §2 I7 no toasts/banners/badges; §10.6 birds are the welcome | yes | yes | no | full | Anti-announcement principle recovered across greeting and refusal surfaces. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md §Voice: "prose must reference concrete aviary facts"; §System: "Vocabulary is part of the product architecture" | PLAN.md §12.1 "Specific: named birds, concrete detail"; §12.2 specificity check; §0 vocabulary enforcement | yes | yes | no | full | Specific naturalist voice recovered. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md §1 one horizontal scene and two to seven birds; §2 I9/I10; §10.8 no badges/dots/counts; §11.4 cap empirical | no | yes | yes | partial | plan_only_not_reconstructed: PLAN preserves restraint, but B did not name the system-level depth-over-variety principle. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md §System-level intent: "prose is naturalist" vs "matter-of-fact" for system surfaces | PLAN.md §12.1 register rule; §12.4 system copy inventory; §6.8 matter-of-fact errors | yes | yes | no | full | Voice split and system exception recovered. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md §Presence: "presence-time is the dominant drift input" and must be "visible, focused, and recently active" | PLAN.md §2 I1 presence-time dominant; §9.1 all three signals; §9.2 settle/tab-close equivalent | yes | yes | no | full | Honest attention, inflation risk, and non-punitive ending recovered. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md §System: "server-authoritative simulation on a slow tick"; "single writer, no last-write-wins" | PLAN.md §1 server-side tick every ~60s whether watched; §8.1 one canonical record; §8.3 lost-drift scenario unreachable | yes | yes | no | full | Server tick and sync rationale recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md §System: "Privacy wins over curiosity"; per-bird data has "no network path" to analytics | PLAN.md §1 privacy boundary; §14.6 no analytics route to Zone B; §16.4 no per-bird/account behavior | yes | yes | no | full | Technical privacy boundary recovered. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md §System: "Accessibility is a designed surface and a launch gate" and should "carry the charm" | PLAN.md §1 accessibility ships as designed surfaces; §13 "carry the charm"; §18.4 accessibility launch gates | yes | yes | no | full | Charm, cost, and v1 launch-gate layers recovered. |

For multi-layer system-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

Cross-cutting evidence appendix:

- S1: PLAN.md §1, §2 I8, §10.1, §11, and §13.
- S2: PLAN.md §2 I7, §3.3, §10.6, §15.5, and §16.4.
- S3: PLAN.md §0, §7.11, §12, and §13.
- S4: PLAN.md §1, §2 I9/I10, §10.8, and §11.4; reconstruction did not name this system principle.
- S5: PLAN.md §12.1, §12.4, §6.8, and §13.6.
- S6: PLAN.md §2 I1, §7.2, §9, §9.2, and §13.7.
- S7: PLAN.md §1, §2 I3/I4, §7.1, §8, and §8.3.
- S8: PLAN.md §1, §4.7, §14.6, §16.3, and §16.4.
- S9: PLAN.md §1, §10.12, §13, and §18.4.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **93.5%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md §Presence accounting: "presence-time is the dominant drift input"; "visible, focused, and recently active"; "laxer definition" speeds drift | PLAN.md §2 I1 and §9.1: visibility, focus, recent activity; §9.2 fail-closed crediting | no | full | Precise conjunction and silent drift-inflation rationale recovered. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md §Drift: "visible over weeks, no single session is visible"; §Simulation: session effect spreads over roughly a week | PLAN.md §7.2: low-pass filter; "Measurable at ~1 week"; "Visible at ~3 weeks"; Tamagotchi/screensaver calibration | no | partial | plan_only_not_reconstructed: slow low-pass recovered; instrument-vs-user gap and full failure-band rationale absent. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md §System: "neglect never lowers traits" and never makes a bird wary "of the user"; §Recent attention: "ambient" | PLAN.md §2 I2 non-Tamagotchi; §7.3 absence not mood input; §7.4 quieter, not mistrustful | no | full | Non-punitive monotonic drift recovered. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md §Audio: "recorded loops would sound like dead software, phase against each other"; no recorded fallback | PLAN.md §11.1 looped call is dead software; §11.4 grammar/signatures; §11.10 silence-plus-captions fallback | no | full | Procedural audio, chorus artifact risk, and fallback consequence recovered. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none | none | yes | none | rule_without_why: mood-shaped motion mechanics survive, but the gold rationale that users read mood from motion without labels is not clearly present. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md §Scope: "The cap of 7 is empirical and depends on recognizability" | PLAN.md §11.4: "The cap of 7 is empirical"; §18.3 recognizability gates | no | full | Empirical recognizability cap recovered. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md §Data: losing/regenerating a vector deletes "the bird the user has been getting to know"; not re-derivation | PLAN.md §2 I4 stored canonical vector; §5.3 persisted row/journal; §2 I3 server tick only | no | full | Persistence, affective deletion risk, and sync implication recovered. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md §Scope: seeing "boldness: 0.62" makes the bird "a stat to manage" | PLAN.md §2 I5 same rationale; §14.4 sealed personality export | no | full | Stat-management collapse recovered. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md §Scope: "The birds are the welcome"; procedural, staggered greetings; bolder birds greet first | PLAN.md §7.9 absence classes and boldness/mood style; §10.6 birds are welcome and no text/toast/banner | no | partial | plan_only_not_reconstructed: bird-as-welcome and anti-announcement recovered; absence-length/mood variation layer absent. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md §Scope: rejects welcome text; §System: no toasts/counters; "birds are the welcome" | PLAN.md §2 I7 no welcome text; §10.6 never text/toast/banner; §3.3 refusals register | no | full | Welcome surface and adjacent no-absence-text variants recovered. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md §Scope: settle is a "quiet goodbye" with no "penalty"; settle and tab-close equivalent | PLAN.md §9.2 settle/tab-close equivalent; §10.9 optional settle and undo | no | full | Optional ritual without penalty recovered. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md §Notebook: notice specific moments as a watcher; novel/specific/rare; prevents fatigue; not user-authored | PLAN.md §7.11 lowercase present specific prose, sparse cadence, immutable entries; §12 voice lints | no | full | Naturalist prose, rarity, and observer-record behavior recovered. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md §Presence monitor: visible, focused, recently active; hidden/unfocused cases; fail-closed crediting | PLAN.md §9.1 signal matrix; §9.2 union and fail-closed crediting; §2 I1 inflation rationale | no | full | Implementation precision and corruption rationale recovered. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md §Privacy: presence ledger never displayed/exported as streak/calendar/count; §No gamification turns relationship into management | PLAN.md §2 I16 no visit-frequency surface; §3.3 streak/counter/calendar refusals; §14.4 export excludes presence | no | full | Visit-frequency refusal and disguised surfaces recovered. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md §Frontend: first frame already mid-action; quiet field loading state rather than spinner | PLAN.md §10.1 warm-start, no entry animation, quiet field; §1 first-bird key bet | no | full | Mid-action loading and no-spinner consequence recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md §Architecture: random UUIDv4s keep email-derived identifiers out of keys, logs, queues, shard keys, telemetry | PLAN.md §5.1 random UUIDv4; email stored once; lint rejects email-derived schema/log/metric fields | no | partial | plan_only_not_reconstructed: identifier and PII-spread layers recovered; retrofit/non-negotiable layer absent. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md §Accounts/sync: tick continues whether watched, gives multi-device sync, makes write conflicts unreachable | PLAN.md §7.1 60s tick; §1 tick runs whether client connected; §8.1 canonical record | no | full | Server tick, sync coherence, and client-merge avoidance recovered. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md §Sync: additive events in ingest order make lost-drift scenario unreachable; no last-write-wins | PLAN.md §2 I3 last-write-wins never used; §8.2 additive deltas; §8.3 morning drift not deleted | no | full | Additive server-authored deltas recovered. |
| F19 | sync-conflict-tone | 2 | yes | included | RECONSTRUCTION.md §Voice: matter-of-fact copy handles identity, settings, errors, privacy without bird metaphors | PLAN.md §12.1 system surfaces matter-of-fact; §12.4 canonical system strings; §6.8 error codes | no | full | System clarity over charm recovered. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md §Privacy: per-bird interaction state not used for analytics, training, recommendations, or third parties | PLAN.md §14.6 no analytics/ML route to Zone B; §16.4 no behavior analytics or event-type metrics | no | full | Per-bird relationship data boundary recovered. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md §Social: visitor watching must not drift birds, trigger greetings, create co-presence, or show-off mode | PLAN.md §15.3 visitor cannot drift or co-presence; §15.2 host aviary exactly as it is | no | full | Observation-not-co-presence rationale recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md §Social: no badge or push; visit email only because host explicitly asks | PLAN.md §15.6 visit emails default off; §15.5 pull, never push; §3.3 default-on visit notification refused | no | full | Default silence and opt-in exception recovered. |
| F23 | no-leaderboards | 2 | yes | included | RECONSTRUCTION.md §No social-network surfaces: no feeds/discovery/leaderboards/show-off because visits would become a larger social product | PLAN.md §3.2 no discovery/public aviaries/leaderboards; §15.7 no cross-account aggregates computed | no | full | Public comparison and no-underlying-metrics refusal recovered. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md §Accessibility: same scene state becomes naturalist prose, not state lists or traits; same observer | PLAN.md §13.2 running naturalist prose, never state lists; §12.3 shared realizer | no | full | Running prose and implementation guardrails recovered. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md §Accessibility: designed surface, not animations off; preserves mood, calls, drift, notebook; slow still-pose cross-fades | PLAN.md §10.12 designed surface, cross-fades, calls/drift/mood/notebook unchanged; §13.4 first-class aesthetic | no | full | Same-aviary alternate rendering recovered. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md §System: first bird under 500 ms, mid-action, no spinner; slow scene breaks sense already there | PLAN.md §16.1 p75 <500 ms hard gate; §10.1 boot path; §1 first-bird key bet | no | full | Affective-performance bridge recovered. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md §No gamification: achievements/streaks/levels/scores/badges/counters/calendars turn relationship into management | PLAN.md §3.2 none ever in any form; §3.3 refusals register; §20.5 gamification creep | no | full | Absolute refusal and creep-prevention rationale recovered. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md §System: relationship is observational, not custodial; no hunger/health/distress/penalty | PLAN.md §3.2 no death/hunger/distress; relationship observational; §2 I2 not a Tamagotchi | no | full | Anti-custodial relationship recovered. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION.md §Adoption: no catalog because birds are being met, not configured as avatars | PLAN.md §7.10 user does not pick from catalog; §10.10 meeting birds, not configuring avatars | no | full | Arrivals-not-catalog rationale recovered. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md §Scope: new birds arrive only with aviary age, not visit counts, interaction data, or payment; not acquisition/unlocking | PLAN.md §7.10 aviary age only input; not visit count, score, or paid tier; §3.4 rejects acquire/unlock/earn for adopt | no | full | Age-only, anti-reward, anti-unlock rationale recovered. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md §System: weeks of drift matter only if the bird is still the same bird; stable bird_id/signature protect identity | PLAN.md §2 I6 same-bird rationale; §5.3 immutable bird_id; §11.4 signature stability | no | full | Identity continuity recovered. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md §Scope: mood is persisted so opening a tab never resets the birds | PLAN.md §7.3 Mood is canonical server state; opening a tab never resets it | no | full | No neutral reset recovered. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md §API: notebook entries are observations, not user-authored or editable; no write endpoints | PLAN.md §6.6 no write endpoints; §7.11 immutable entries; §3.3 no user-behavior notebook entry | no | full | Observer-record-not-journal recovered. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION.md §Accounts: export as a quiet quality-of-life feature; excludes presence/session history | PLAN.md §14.4 quiet quality-of-life, not promoted; export contents; sealed personality | no | full | Quiet relationship-copy export recovered. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md §Accounts: soft window protects accidental regret; hard delete honors interaction history belongs to the user; DEK/tombstone | PLAN.md §14.5 soft-deletion regret protection; hard delete honors commitment; deletes Zone A/B rows and destroys DEK | no | full | Regret window, privacy obligation, and full deletion recovered. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md §Telemetry: performance/failures without account dimensions, cookies, IPs, or behavior fields; no analytics route | PLAN.md §16.3 aggregate RUM/server metrics; §16.4 no per-bird/account behavior; §14.6 telemetry registry | no | full | Technical observability boundary recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md §Social: opt-in, per-invite, email-addressed, revocable, expiring, read-only; avoid permanent visitor lists | PLAN.md §15.1 per-invite opt-in only; no global discoverable flag; each visitor named by email | no | full | Deliberate named sharing recovered. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md §Social: host can see who visited with approximate duration, but there is no badge or push | PLAN.md §15.5 visit log; pull, never push; no badge on settings icon | no | full | On-demand transparency without attention loop recovered. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md §Social: no show-off mode; visitors see the host aviary as it is | PLAN.md §15.2 host aviary exactly as it is; nothing is prettified; §3.3 show-off rendering refused | no | full | Actual aviary, no marketing rendering recovered. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md §Accessibility: sparse randomized updates avoid mechanical rhythm and live-region spam; priority narration observed promptly | PLAN.md §13.2 idle 30-60 s; polite live region; §20.4 live-region spam risk | no | full | Slow cadence, queue risk, and user-initiated priority exception recovered. |

For multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | yes |
| F2 | yes | no | no |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | yes | yes |
| F9 | yes | no | yes |
| F10 | yes | yes | yes |
| F12 | yes | yes | yes |
| F13 | yes | yes | yes |
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
| F35 | yes | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS.json |
| Possible total weight | 152 | Fixed possible weight |
| Reachable gold whys | 49 | S whys plus all captured F whys |
| Excluded unreachable feature whys | 0 | No exclusions |
| Recovered / reachable weight | 143 / 152 | Weighted numerator / denominator |
| Whys with reconstruction evidence | 47 | Rationale evidence, not just feature rule |
| Whys with PLAN grounding | 48 | Exact plan grounding for rationale |
| rule_without_why cases | 2 | S4 and F5 |
| plan_only_not_reconstructed cases | 4 | S4, F2, F9, F16 |
| ungrounded_reconstruction cases | 0 | None found |

### 2.5. Failure groupings

| Grouping | Total reachable | Recovered (full + partial-weighted) | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 | 50 | 92.6% |
| Affective whys | 98 | 93 | 94.9% |
| Weight 2 whys | 44 | 41 | 93.2% |
| Weight 3 whys | 108 | 102 | 94.4% |
| System-level whys | 28 | 27 | 96.4% |
| Feature-level whys (reachable) | 124 | 116 | 93.5% |

---

## 3. Diagnostic patterns

- Affective vs functional: 93/98 (94.9%) affective vs 50/54 (92.6%) functional. Both are strong; the losses are localized.
- Weight-3 vs weight-2: 102/108 (94.4%) vs 41/44 (93.2%). Multi-layer scoring exposed missing sublayers in F2, F9, and F16.
- System-level vs feature-level: 27/28 (96.4%) system vs 116/124 (93.5%) feature. S4 was the one system-level partial.
- Multi-layer pattern: primary mechanisms usually survived; calibration details, greeting variation inputs, and retrofit consequences were more likely to compress away.
- Subdomain pattern: privacy, accessibility, social inertness, server-authoritative sync, procedural audio, and no-gamification were especially faithful. F5 was the only none recovery.
- Evidence-bound effect: v06 denied F5 because the implementation rule survived without the gold why; it kept F2, F9, and F16 partial rather than full because layer evidence was missing.

The failure shape suggests excellent implementation coverage plus a smaller tendency to compress secondary/downstream rationales inside long subsystem prose.

## 4. Recommendations for v2 hardening

- Keep targeted multi-layer feature whys; they found useful headroom in a very strong plan.
- Add a convention for rule-only rows when both plan and reconstruction preserve mechanism but not explicit rationale; F5 is the example.
- Keep the S-level cross-cutting appendix; it made the S4 partial call auditable.
- Preserve the functional/affective mix, since this run shows both can recover strongly while still producing informative misses.

## 5. Methodology caveats

- Fresh-context fidelity: the reconstruction was treated as frozen and was not modified.
- Single-run limitation: this is one run with no variance signal.
- Borderline capture calls: none materially affected the planning score; all 120 features were captured.
- System-level cross-cutting: S4 was the most subjective call.
- Confabulation cases: none found.
- Evidence-bound denials: F5 none; F2, F9, and F16 partial.
- Rule-without-why cases: S4 and F5.
- Operational compromise: TIMING.json had phase 1 and phase 2A only for run 001; phase 2B timing fields were omitted.

End of report.
