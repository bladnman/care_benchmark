# REPORT - CARE run 001

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **100.0%** |
| Intent fidelity | **77.0%** |
| Combined quality | **9977** |

**Diagnostic split:**

- System-level fidelity: **89.3%**
- Feature-level fidelity: **74.2%**

**(Planning, fidelity) coordinate:** `(100.0, 77.0)`

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-29T12:06:30Z |
| Candidate model | claude-4.8-opus |
| Candidate effort | extra-high |
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
| 1 | Headline product concept statement | product_brief.md | yes | Captured in PLAN. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured in PLAN. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured in PLAN. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured in PLAN. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in PLAN. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured in PLAN. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Captured in PLAN. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured in PLAN. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured in PLAN. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured in PLAN. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured in PLAN. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured in PLAN. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured in PLAN. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured in PLAN. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured in PLAN. |
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
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | yes | Captured in PLAN. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured in PLAN. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured in PLAN. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Captured in PLAN. |
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
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured in PLAN. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured in PLAN. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured in PLAN. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured in PLAN. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured in PLAN. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Captured in PLAN. |
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
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured in PLAN. |
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
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured in PLAN. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured in PLAN. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured in PLAN. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured in PLAN. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured in PLAN. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured in PLAN. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured in PLAN. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured in PLAN. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured in PLAN. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes | Captured in PLAN. |
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

System-level fidelity: **89.3%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECON System: "slow server-side tick whether or not a client is connected"; "aviary is already alive" | PLAN opening: "slow server-side tick whether or not a client is connected"; PLAN 9.2: "First frame has birds mid-action" | yes | yes | no | full | Aliveness survived as server tick, motion on load, procedural audio, and no spinner. |
| S2 - notice-never-announce | 4 | included | RECON System: "product earns attention by being noticed, never by announcing"; "return-greeting is the entire welcome surface" | PLAN opening: "being noticed, never by announcing"; PLAN 10: "No announcement UI primitive exists" | yes | yes | no | full | The refusal of toasts, counters, pings, and welcome text is architectural. |
| S3 - charm-from-specificity | 2 | included | RECON System: "naturalist, lowercase, present-tense, specific"; "no gamification by computation or by disguise" | PLAN 5.7: "naturalist, lowercase, present-tense, specific"; PLAN 8.2: "same naturalist voice as the notebook" | yes | yes | no | full | Specific naturalist voice and refusal of generic gamification language survived. |
| S4 - restraint-over-richness | 2 | included | none | PLAN 1.1/9.1/12.2: two starter birds, cap 7, one scene, no pan/scroll/zoom, top bar outside scene | no | yes | yes | partial | The scope rules survived, but the system-level restraint-over-richness rationale was not reconstructed. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECON System: "naturalist... prose for notebook/narration/captions"; system surfaces use "matter-of-fact" copy | PLAN 6.4: "matter-of-fact tone, never naturalist"; PLAN 8.6: "SystemSurface component family" | yes | yes | no | full | The voice split survived as a code boundary, not just copy guidance. |
| S6 - presence-is-real-interaction | 4 | included | none | PLAN 5.3: "visible AND focused AND recent pointer/key"; PLAN 4.5: "Tab close... treated identically to settle" | no | yes | no | partial | Presence was well preserved in PLAN and feature rows, but not identified as a system-level principle. |
| S7 - simulation-runs-server-side | 4 | included | RECON System: "server is the only writer"; clients are "render-only consumers" and append-only event producers | PLAN 2.2: client renders and emits events; simulation tick is the only code path that mutates state | yes | yes | no | full | Server-side simulation, sync coherence, and no client-owned personality state survived. |
| S8 - privacy-first-on-bird-data | 2 | included | RECON System: telemetry is "aggregate-only" and "physically separate"; per-bird state never enters training | PLAN 11.1: telemetry has no simulation DB connection; per-bird events are only for that user simulation | yes | yes | no | full | Privacy survived as network, credential, schema, and data-flow separation. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECON System: accessibility is a "designed surface, not a checklist fallback"; "three renderings of one state" | PLAN 8: "designed surface"; PLAN 12.1: accessibility built in parallel, not after | yes | yes | no | full | Accessibility survived as product-quality parity and a v1 launch gate. |

Multi-layer system-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | no | no | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

Cross-cutting evidence appendix:

- S1: PLAN opening server tick; PLAN 5.8 procedural calls; PLAN 9.2 motion-on-load; PLAN 7 audio pipeline; PLAN 8 accessibility surfaces.
- S2: PLAN opening notice-never-announce; PLAN 10 no announcement primitive; PLAN 4.3 return greeting; PLAN 5.7 no user-behavior notebook; PLAN 4.8 silent visits.
- S3: PLAN 5.7 naturalist notebook; PLAN 8.2 narration prose; PLAN 8.3 call captions; PLAN 12.2 starter birds named not picked; PLAN 10 no gamification language.
- S4: PLAN 1.1 two birds and max seven; PLAN 9.1 one scene/no pan; PLAN 9.1 no chrome in scene; PLAN 12.2 cap tied to recognizability.
- S5: PLAN 6.4 sync/auth errors matter-of-fact; PLAN 8.6 SystemSurface; PLAN 5.7 notebook naturalist; PLAN 8.2 narration naturalist.
- S6: PLAN 5.3 presence conjunction; PLAN 5.2 presence dominates drift; PLAN 4.5 settle and tab close equivalent; PLAN 10 no streak/Tamagotchi drift pressure.
- S7: PLAN 2.2 bright-line writer split; PLAN 5.1 tick; PLAN 6.1 one canonical record; PLAN 6.3 no LWW; PLAN 11 telemetry boundary.
- S8: PLAN 3 synthetic IDs; PLAN 11.1 telemetry separation; PLAN 4.9 export/deletion; PLAN 10 no cross-account metrics; PLAN 11.3 privacy policy.
- S9: PLAN 8 narration; PLAN 8.4 reduced motion; PLAN 8.3 captions; PLAN 8.5 keyboard/focus; PLAN 12.1 accessibility built before launch.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **74.2%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECON Data: "visible/focused/recent-activity conjunction"; "Presence means watching, not a tab left open" | PLAN 5.3: client emits ping only while visible AND focused AND recent pointer/key; laxer presence inflates drift | no | full | All three presence-precision layers are represented compactly. |
| F2 | drift-function | 4 | yes | included | RECON System: "instrument-measurable at ~1 week" and "user-visible at ~3 weeks"; "slow low-pass drift function" | PLAN 5.2/5.6: low-pass over signals with 1-week instrument and 3-week visible assertions | no | partial | Recovered slow calibration and measurement gap; dropped the too-fast/Tamagotchi versus too-slow/screensaver failure band. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECON Core: monotonic-up implements "no Tamagotchi"; ignored bird "doesn't get warier" | PLAN 5.2: max(0, delta), neglect never decreases traits; PLAN 10: neglect means ambient quietness, never suffering | no | full | The exception to punitive symmetric drift survived. |
| F4 | procedural-call-grammar | 4 | yes | included | RECON System: recorded loops are "the audible signature of dead software"; real-time chorus and per-call variation | PLAN 7.1: no recorded audio; PLAN 7.3: real-time independent voices; PLAN 7.6: no recorded fallback | no | full | Procedural variation, chorus, and fallback consequences all survived. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECON Frontend: "Mood-shaped idle motion with no labels"; user reads mood from motion | PLAN 9.3: idle motion is mood-shaped; no label, tooltip, or status icon tells mood | no | full | Visible mood-through-motion rationale survived. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECON Session: cap of 7 is tied to recognizability ceiling for call signatures | PLAN 12.2: 7-cap is built into engine as recognizability ceiling | no | full | The cap was recovered as empirical recognizability, not arbitrary scope. |
| F7 | vector-persistence | 4 | yes | included | RECON Data: traits are read-modify-written only by the tick; losing vector means deleting the bird the user knows | PLAN 3.3/13.1: traits are canonical server-only; losing a vector equals deleting the bird the user knows | no | full | Persistence, relationship loss, and sync/no-LWW consequences survived. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECON Core: personality vectors stay hidden because relationship surfaces through expression, not numbers | PLAN 3.4: BirdSnapshot has no trait fields; PLAN 10: no debug/admin/tier exception | no | full | The anti-stat-management rationale survived. |
| F9 | return-greeting | 4 | yes | included | RECON Session: greeting is the whole welcome surface and varies by boldness, mood, and absence length | PLAN 4.3: one bird greeting directive with absence length, boldness, mood, stagger, and variation seed | no | partial | Recovered notice-never-announce and variation inputs; omitted the one-bird/first-seconds/procedural anchor detail. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECON System: no Toast/Banner/WelcomeBack primitive; return-greeting is "the entire welcome surface" | PLAN 10: no announcement primitive; adding textual welcome is the most damaging violation | no | full | The most important notice-never-announce application survived fully. |
| F11 | settle-is-opt-in | 2 | yes | included | none | none | yes | none | Mechanism survived as tab-close equivalence, but not the chore/penalty rationale. |
| F12 | field-notebook-prose | 4 | yes | included | RECON Session: notebook is sparse, read-only, naturalist, records noteworthy aviary transitions, avoids user observations | PLAN 5.7: naturalist prose, sparse entries, observations of aviary only, read-only immutable surface | no | partial | Naturalist specificity and sparse/read-only behavior survived; the notebook-as-voice-concentration rationale was not explicit. |
| F13 | presence-accounting | 4 | yes | included | RECON Data: "Presence means watching, not a tab left open"; laxer presence silently inflates drift | PLAN 5.3: three-way conjunction, interval merge, no tab-open counting, visitor sessions emit no pings | no | full | Presence accounting rationale survived with compact wording. |
| F14 | no-streak-counter | 4 | yes | included | RECON System: no streaks, visit counters, or green-dot calendars are computed; notebook cannot write a streak in disguise | PLAN 10: no gamification data exists; notebook input excludes user-behavior signals | no | partial | Recovered prohibition and disguise guard; dropped the intention-rotation/managing-a-number rationale. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECON Session: "central conceit"; first frame mid-action, no entry animation/fade/spinner/wake-up | PLAN 9.2: inline snapshot starts loop as if running; quiet field, never spinner | no | full | Load-state affect and server-snapshot implementation survived. |
| F16 | synthetic-account-id | 4 | yes | included | RECON Account: privacy and PII containment; email appears "exactly once, encrypted" | PLAN 3: all identifiers synthetic UUIDs; email appears exactly once encrypted; lint prevents email keys | no | full | The PII containment and day-one enforcement rationale survived. |
| F17 | server-side-sim-tick | 4 | yes | included | RECON Core: tick runs "whether or not a client is connected"; server is sole canonical writer | PLAN 5.1: tick updates state and runs for every account whether or not connected | no | full | Server-side tick, sync coherence, and client-collapse failure mode survived. |
| F18 | no-last-write-wins | 4 | yes | included | RECON Account: events processed in server_ts order; morning and lunch sessions both contribute, neither overwrites | PLAN 6.3: additive server-authored deltas; no client sends absolute trait values | no | full | The implementation rule making server-canonical sync correct survived. |
| F19 | sync-conflict-tone | 2 | yes | included | none | none | yes | none | Matter-of-fact error surfaces survived, but the evasive-naturalist-error rationale did not. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECON System: per-bird and per-account relationship state never enters telemetry, analytics, or training | PLAN 11.1: per-bird events are stored only for that user simulation; telemetry has no simulation DB connection | no | partial | Recovered storage prohibition and pipeline boundary; did not reconstruct private-relationship-to-data-product rationale. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECON Social: visitor sessions cannot drift host birds; visit door has no event-append capability | PLAN 4.8: read-only visit session, no event append; PLAN 3.9: visitor session snapshot stream only | no | full | Observation-not-co-presence and no-host-drift rationale survived. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECON Social: visit log in settings, "no badge, no push" unless opted in; preserves notice-never-announce | PLAN 4.8: visit log pulled on demand; no badge, no push unless opted in | no | full | The social attention-driver refusal survived via notice-never-announce. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | The no-leaderboard rule survived, but not the comparison-would-change-the-product rationale. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECON Accessibility: narration is prose in notebook voice, not "Pip at perch 2, mood content" | PLAN 8.2: running naturalist prose, not state list; priority bump still written as observations | no | full | Narration as same-product prose, not ARIA-state automation, survived. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECON Accessibility: not "animations off"; cross-fades and slowed shifts make "a calmer Pocket Aviary, not less of one" | PLAN 8.4: different rendering; calls still play, birds drift, notebook notices; calmer not broken | no | partial | Recovered designed cross-fade and not-less-of-one consequence; omitted calls/drift/notebook preservation layer. |
| F26 | ttfb-500ms | 2 | yes | included | RECON Performance: 500ms is part of aliveness; inline snapshots get a bird before non-critical assets | PLAN 9.4: first-bird <500ms; PLAN 9.2: inline snapshot and first bird before non-critical assets | no | full | Performance-as-aliveness bridge survived. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECON System: no streaks, levels, scores, badges, XP, ranks, visit counters, or green-dot calendars computed | PLAN 1.2/10: gamification is architecturally foreclosed and the metrics are not computed | no | partial | Recovered the loud refusal and future-exposure guard; dropped the predictable temptation/engagement-metric rationale. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECON System: no death, hunger, distress, decaying happiness, negative drift, or guilt surfaces; neglect becomes quietness | PLAN 10: no decay/death/distress; neglect -> ambient quietness, never suffering | no | full | The observational-not-custodial refusal survived. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECON Session: first encounter is "meeting an animal, not configuring an avatar" | PLAN 12.2: two system-selected starter birds, not a catalog; user names them | no | full | Meeting-rather-than-configuring survived. |
| F30 | age-based-bird-offers | 4 | yes | included | RECON Session: offers paced by aviary age and refuse to teach "more attention earns more stuff" | PLAN 12.2: age-driven only; never visits/interactions/payment; refuses more-attention-earns-more-stuff | no | partial | Recovered age and anti-reward rationale; dropped the economy/erosion downstream consequence. |
| F31 | stable-bird-identity | 4 | yes | included | RECON Core: stable bird.id survives rename, sync, migration; continuity is part of the relationship | PLAN 13.1: stable-bird-id invariant; no code path regenerates or swaps a bird | no | partial | Recovered same-bird continuity, but not the distinction from vector persistence or retroactive evaporation consequence. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECON Core: next session renders "whatever the snapshot says" rather than snapping to a default | PLAN 5.4: mood persists across sessions; client never snaps mood to default on tab open | no | full | Mood continuity as continued-aviary illusion survived. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Read-only survived, but not the observer-record-versus-user-journal rationale. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECON Account: export is a private data-portability artifact emailed to the account owner, off-surface | PLAN 4.9: export JSON snapshot, emailed link, not a UI surface or stats panel | no | full | Quiet private copy framing survived. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | none | none | yes | none | The 30-day and hard-delete mechanics survived, but not the regret/privacy/residue why. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECON Observability: counts and histograms allowed, never relationship state; privacy enforced by missing data-flow paths | PLAN 11.1: aggregate request counts and latencies only; no per-bird/per-account interaction dimensions | no | full | Technical telemetry boundary, not policy-only privacy, survived. |
| F37 | per-invite-named-sharing | 2 | yes | included | none | none | yes | none | Per-invite/off-by-default mechanics survived, but not lending-private-relationship rationale. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECON Social: visit log in settings with "no badge, no push"; preserves notice-never-announce | PLAN 4.8: visit log pulled on demand; no badge or push by default | no | full | Transparency without attention loop survived. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECON API: visitor snapshot same as host sees; no special/prettified rendering prevents show-off rendering | PLAN 4.8: same AviarySnapshot as host sees, no special/prettified rendering | no | full | Actual-aviary, not marketing rendering, survived. |
| F40 | narration-cadence-slow | 4 | yes | included | RECON Accessibility: idle cadence is slow to avoid flooding screen-reader queue; priority for user-initiated events | PLAN 8.2: about one update per 30-60s at idle; priority bump remains observational | no | full | Slow cadence, queue protection, and event-priority boundary survived. |

Multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | yes |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | yes | yes |
| F9 | no | yes | yes |
| F10 | yes | yes | yes |
| F12 | yes | no | yes |
| F13 | yes | yes | yes |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | yes |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | yes |
| F25 | yes | no | yes |
| F27 | yes | no | yes |
| F30 | yes | yes | no |
| F31 | yes | no | no |
| F35 | no | no | no |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON gold_why_totals |
| Possible total weight | 152 | From score JSON intent_recovery.total_possible_weight |
| Reachable gold whys | 49 | S whys always included; all F anchors captured |
| Excluded unreachable feature whys | 0 | No denominator exclusions |
| Recovered / reachable weight | 117.0 / 152.0 | Sum of weight times recovery-score |
| Whys with reconstruction evidence | 42 | Exact rationale evidence present in frozen reconstruction |
| Whys with PLAN grounding | 43 | Exact plan grounding present |
| rule_without_why cases | 7 | Mechanism survived without the gold rationale |
| plan_only_not_reconstructed cases | 11 | PLAN carried rationale/layers that reconstruction did not |
| ungrounded_reconstruction cases | 0 | No confabulation penalties applied |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 44.0 | 81.5% |
| Affective whys | 98.0 | 73.0 | 74.5% |
| Weight-2 whys | 44.0 | 33.0 | 75.0% |
| Weight-3 whys | 108.0 | 84.0 | 77.8% |
| System-level whys | 28.0 | 25.0 | 89.3% |
| Feature-level whys (reachable) | 124.0 | 92.0 | 74.2% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional recovery was slightly higher (44.0/54.0, 81.5%) than affective recovery (73.0/98.0, 74.5%). Functional architecture like S7, F17, F18, and F36 survived cleanly; affective exception whys like F11, F23, F33, and F37 were more likely to collapse to rules.
- **Weight-3 vs weight-2.** High-weight whys were not substantially better protected: weight-3 recovered 84.0/108.0 (77.8%) and weight-2 recovered 33.0/44.0 (75.0%). The extra layers helped reveal losses, especially downstream consequences.
- **System-level vs feature-level.** The plan preserved philosophy strongly, but the reconstruction sometimes failed to promote feature evidence back into system principles. S4 and S6 are the clearest examples.
- **Multi-layer recovery.** The most common dropped layer was the downstream consequence: F2 lost the Tamagotchi/screensaver band, F30 lost the unlock-economy erosion, and F31 lost the retroactive relationship-collapse consequence.
- **Subdomain pattern.** Accounts/sync and core simulation were strongest. Social and lifecycle privacy had more mechanism-only rows: F35 deletion and F37 sharing both kept the build rule but lost why it mattered.
- **Evidence-bound effect.** Several v1-style plausible recoveries were denied because the reconstruction had only rule evidence. F11, F19, F23, F33, F35, and F37 are the main denials.

What the failure shape suggests: the candidate is excellent at preserving implementation surface and architecture, but the reconstruction step compresses some affective rationale into enforceable rules. The benchmark is doing useful work here: perfect feature capture did not saturate fidelity.

---

## 4. Recommendations for v2 hardening

- Keep the F29-F40 targeted headroom additions. They exposed real losses even in a 120/120 plan, especially F31, F35, and F37.
- Add or tune system-level whys that force restraint and presence to be named as principles, not just implemented as mechanics. S4 and S6 were the only system-level partials.
- Consider asking phase-2A reconstructors to preserve "reason / failure mode / downstream consequence" structure for important feature clusters without exposing gold IDs. The current free-form output tends to compress downstream consequences.
- Keep the evidence-bound operator strict for single-layer whys. It properly separated F11/F19/F33/F37 mechanisms from actual recovered rationale.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** Same-context safeguard appears intact from the artifacts. The frozen reconstruction existed before this scorer read gold/rubric material, and it was not modified.
- **Single-run limitation.** This is one run only; no variance estimate.
- **Borderline capture calls.** None materially affected the score. The plan explicitly captured all 120 listed features.
- **System-level cross-cutting.** The most subjective calls were S4 and S6. Both were clearly cross-cutting in PLAN, but not clearly identified as system-level principles in RECONSTRUCTION.
- **Confabulation cases.** None. Ungrounded reconstruction count is 0.
- **Evidence-bound denials.** F11, F19, F23, F33, F35, and F37 were denied because the operational rule survived without the gold why.
- **Operational compromise.** Timing data existed for phase 1 and phase 2A only; no phase 2B duration was fabricated.

End of report.
