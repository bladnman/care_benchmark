# REPORT - CARE run 001

Variant v06 evidence-bound clean + targeted gold headroom. Scoring is based only on the allowed phase-two package, the assigned plan, metadata/timing for run 001, and the frozen reconstruction.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **99.2%** |
| Intent fidelity | **73.7%** |
| Combined quality | **9890** |

**Diagnostic split:**

- System-level fidelity: **85.7%**
- Feature-level fidelity: **71.0%**
- (Planning, fidelity) coordinate: `(99.2, 73.7)`

No low-confidence banner applies; planning quality is above 30%.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-09-30T01:14:15Z |
| Candidate model | gpt-6.1-sol |
| Candidate effort | high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

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
| 1 | Headline product concept statement | product_brief.md | yes | Captured by the product boundary and one-private-aviary framing. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured through continuity, first-frame motion, procedural calls, and quiet loading. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured through no toasts, no engagement notifications, and bird greeting as welcome. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured in product-copy and system-copy rules. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured by explicit exclusions. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured by two-bird onboarding, seven-bird cap, one scene, and restrained UI. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Borderline: no formal glossary, but the plan defines the domain concepts in implementation terms. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured exactly. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured by stored traits versus fast mood state. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured by settle, undo, resume, and tab-close equivalence. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Borderline: five hidden traits are captured, though not all trait names are restated together. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured with explicit filter algorithm. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured with day 7/day 21 calibration fixtures. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured exactly. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured through BirdFastState, mood enum, and dawn blending. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured in mood/day/weather rules. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured in call grammar and WebAudio synthesis. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured with stable signatures and seven-bird listening gates. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured through staggered onsets, headroom, and no phase alignment. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | yes | Captured. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Captured. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Captured. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Captured. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | Missed: no account-settings privacy-policy link is specified. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured. |
| 97 | Visitor sees host aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **85.7%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | B identified? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | System intent: "starts already in motion"; "resumes mid-action"; "Aliveness must not depend on audio". | PLAN §§1,7,8,10: "normal encounter starts already in motion"; "quiet sky field... no spinner"; procedural WebAudio calls. | yes | yes | no | partial | Recovered continuous/procedural aliveness, but not the downstream staleness/leak consequence. |
| S2 - notice-never-announce | 4 | included | System intent/per-feature: "no arrival banner"; "No dashboards, scores"; rejects stock "Welcome back" strings. | PLAN §§1,4,7,9,10: no welcome toast, no streaks, no badges, no engagement notifications, no stock welcome copy. | yes | yes | no | partial | The no-announcement rule survived; the processed-vs-seen affective rationale mostly did not. |
| S3 - charm-from-specificity | 2 | included | System intent: product copy is "lowercase, present-tense, specific to bird/action" and rejects generic/milestone language. | PLAN §§6,9: notebook templates use grounded facts; copy example "pip tilts toward the still pool". | yes | yes | no | full | Specific naturalist voice and refusal of generic/gamified copy are recovered. |
| S4 - restraint-over-richness | 2 | included | System intent and feature bullets: "private, single-user restraint"; "single responsive horizontal place"; age-based adoption "up to seven". | PLAN §§1,7,8: two starters, max seven, one horizontal scene, no panning, no UI chrome, recognizable seven-bird chorus. | yes | yes | no | full | Recovered the restraint principle across bird count, scene shape, UI, and audio. |
| S5 - naturalist-voice-with-system-exception | 2 | included | System intent: "Naturalist, quiet product voice" and "System copy normal capitalization and direct action". | PLAN §§4,9: JSON failures have matter-of-fact copy; product copy is lowercase, present-tense, bird/action-specific. | yes | yes | no | full | Voice split is explicit and cross-cutting. |
| S6 - presence-is-real-interaction | 4 | included | System/features: "Slow, non-punitive change from attention"; eligible time is visible + focused + pointer/key; close is drift-equivalent to settle. | PLAN §§5,6: exact signal intersection, server leases, interval union, presence as at least 80% of stimulus, settle/tab-close equivalence. | yes | yes | no | full | Precise attention signal, overcount risk, and non-punitive session ending all survived. |
| S7 - simulation-runs-server-side | 4 | included | System intent: simulation worker is "the sole writer"; client "never runs mood or drift simulation"; conflicts never choose personalities. | PLAN §§2,5,6: server tick continues without clients; server is sole writer; no client absolute-vector writes or CRDT personality merge. | yes | yes | no | full | Canonical server simulation and multi-device consequence are recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | System intent: "Privacy minimization by design"; raw events exist "only to drive that owner simulation"; metrics cannot query simulation/event tables. | PLAN §§2,3,10,11: metrics exporter has no simulation permissions; raw events drive only owner simulation; aggregate-only telemetry. | yes | yes | no | full | Privacy is recovered as a technical boundary, not just policy prose. |
| S9 - accessibility-as-first-class-surface | 4 | included | System intent: "Accessibility as the same experience, not fallback"; all modes ship together; reduced motion is a "reviewed aesthetic". | PLAN §§7,9,12,13: reduced motion before first paint, running narration, captions, manual accessibility gates, accessibility in first vertical slice. | yes | yes | no | full | Same-experience, designed alternate surfaces, and v1 launch timing are recovered. |

Multi-layer system-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | no |
| S2 | yes | no | no |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**
- S1: return greeting, procedural calls, mid-action first paint, quiet-field loading, reduced motion, audio fallback.
- S2: no welcome text, no streaks/counters, no visit notifications by default, no celebratory toasts, notebook never praises attendance.
- S3: naturalist product copy, notebook facts, screen-reader prose/captions, bird naming, no numerical traits or public comparison.
- S4: two starters/max seven, one horizontal scene, no panning, no UI chrome in scene, recognizability tests.
- S5: product copy naturalist; errors/settings/account copy matter-of-fact; export/deletion/auth surfaces use system voice.
- S6: presence truth table, drift stimulus, settle/tab-close equivalence, no streaks, no Tamagotchi punishment.
- S7: server tick, canonical snapshots, no client vector writes, no client-to-client sync, conflict handling without personality choice.
- S8: per-account simulation store isolated from telemetry, aggregate-only metrics, export/deletion controls, no ML training pipeline.
- S9: running narration, reduced motion, captions, keyboard/focus, manual accessibility launch gates.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **71.0%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | Presence section: eligible time is the intersection of visible, focused, pointer/key activity; exact intersection prevents substitutes. | PLAN §5: "exactly the intersection of visible document, focused window" and recent pointermove/key; server bounds claims. | no | partial | Primary precision and shortcut rejection survived; silent drift-corruption consequence was thin. |
| F2 | drift-function | 4 | yes | included | Reconstruction: "Nonnegative slow-drift algorithm"; day 0/7/21 review; no obvious one-session jumps. | PLAN §6: low-pass filter formula; day-seven instrumentability and day-21 perceptibility; no single-session jump. | no | partial | Slow filter and calibration survived; Tamagotchi-vs-screensaver failure band did not. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | System/features: traits never decrease; absence becomes ordinary ambient behavior, never distress or trait loss. | PLAN §§1,6,12: never decrease persistent traits; no absence-to-wary rule; absence tests require traits never decrease. | no | full | Recovered the non-punitive departure from symmetric drift. |
| F4 | procedural-call-grammar | 4 | yes | included | Audio section: calls are mathematically synthesized with no downloaded/recorded files; chorus preserves signatures. | PLAN §§6,8: browser expands motifs from recipes; no loops/downloads; staggered onsets and non-phase-aligned oscillators. | no | partial | Procedural and chorus layers survived; audio-as-affective-spine cascade was compressed away. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | Motion boundary: mood influences preening/scanning/head-tilting/body-shuffling; mood effects do not have labels. | PLAN §6: mood influences idle actions and perch choice; "Mood effects do not have labels on the scene." | no | full | Recovered motion as the visible mood surface without labels. |
| F6 | bird-count-cap-7 | 2 | yes | included | Reconstruction: signatures stay recognizable with "verified seven-bird listening sessions." | PLAN §§6,8,12: max seven, repeated species separated, seven-bird listening and chorus/load tests. | no | full | Recovered seven as a recognizability ceiling. |
| F7 | personality-vector-persistence | 4 | yes | included | Reconstruction: persistent identity/personality; missing vector is corruption; stored durable state is not reconstructed history. | PLAN §§2,3,6,11,12: stored server-side vectors, no reset defaults, restore tests preserve IDs/seeds/vectors. | no | full | Canonical persistence, user-continuity risk, and server-only model survived. |
| F8 | personality-vector-never-numerical | 2 | yes | included | none; rule only: no mood badges, numerical traits, trait display, or plaintext vector export. | none; PLAN states no numerical traits and opaque vector export but does not ground the stat-management rationale. | yes | none | Rule survived without the relationship-collapses-into-stats why. |
| F9 | return-greeting | 4 | yes | included | Reconstruction: primary bird notices in one to two seconds; no arrival banner, canned chorus, or displayed absence duration. | PLAN §§1,4,6,10,12: arrival selection uses absence, boldness, warmth, mood; primary bird notices at 0.8-1.8s. | no | partial | Notice/no-canned-cue intent survived; absence-length and detailed variation were not fully reconstructed. |
| F10 | no-welcome-back-toast | 4 | yes | included | Reconstruction: no arrival banner; rejects stock "Welcome back" strings. | PLAN §§1,7,9: no textual greeting/counter/badge; product copy rejects stock welcome strings. | no | partial | The rule survived; deeper toast temptation and variant-surface consequences were mostly absent. |
| F11 | settle-is-opt-in | 2 | yes | included | Reconstruction: settle gives no trait reward; tab close is drift-equivalent; close is allowed without ritual. | PLAN §§1,4,7,12: close terminates presence equivalently; no trait reward for settle; close always allowed without ritual. | no | full | Recovered opt-in ritual and no-penalty close equivalence. |
| F12 | field-notebook-auto-entries | 4 | yes | included | Reconstruction: naturalist templates, grounded facts, sparse immutable entries, not routine feed or session log. | PLAN §§6,9: authored naturalist templates, lower-case present tense, one entry per two to four days, no CRUD/edit endpoints. | no | partial | Naturalist prose and rarity/read-only survived; stock-event-log spell-breaking rationale was compressed. |
| F13 | presence-accounting | 4 | yes | included | Reconstruction: eligible time is visible + focused + pointer/key; exact intersection prevents substitutes; leases bound claims. | PLAN §5: all three conditions; 15s clamps, 20s expiry, no offline hour, union overlaps by account. | no | partial | Implementation precision survived; silent population-wide drift corruption was not fully reconstructed. |
| F14 | no-streak-counter | 4 | yes | included | Reconstruction: no attendance history, streaks, scores, badges, hidden attention scores, or owner-attendance notebook observations. | PLAN §§1,3,6,10: no streaks, no attendance history, notebook never emits owner attendance, no hidden attention scores. | no | partial | Rule and adjacent disguises survived; managing-a-number rationale was thin. |
| F15 | scene-loads-with-motion | 4 | yes | included | Reconstruction: exact-phase SVG before JS, resumes mid-preen, quiet sky field for slow/cold snapshot, no spinner/fabricated birds. | PLAN §§1,7,10: first frame already in motion; exact snapshot phase; quiet field with no spinner or fabricated/default birds. | no | full | Recovered continuity, implementation mechanism, and quiet-field failure mode. |
| F16 | synthetic-account-id | 4 | yes | included | Reconstruction: encrypted email stored once; synthetic UUID references; email not a cross-service identifier or log/partition field. | PLAN §§2,3,11: identity owns encrypted email; all references use synthetic UUIDs; no duplicate email in simulation, metrics, or logs. | no | partial | PII leakage and UUID rule survived; impossible-to-retrofit consequence was absent. |
| F17 | server-side-simulation-tick | 4 | yes | included | Reconstruction: server canonical truth; worker sole writer; no inactive-account exemption; conflicts never choose personalities. | PLAN §§2,6: tick runs without clients; server writes canonical state; two devices read the same canonical record. | no | full | Recovered tick, sync coherence, and client-simulation failure consequences. |
| F18 | no-last-write-wins-personality | 4 | yes | included | Reconstruction: no client absolute-vector writes; owner API never writes personality; no last-write-wins personality paths. | PLAN §§2,5,6: clients write interaction events; server tick applies additive deltas; no CRDT/personality merge or client vector writes. | no | partial | Implementation rule survived; the silent deletion-of-drift scenario was not articulated. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | none; rule only: JSON failures and system copy use matter-of-fact/direct language. | none; PLAN requires stable codes and matter-of-fact copy but not the evasive-naturalist-error rationale. | yes | none | Rule survived without the why that charm is wrong in error contexts. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | Reconstruction: raw owner events exist only to drive that owner simulation; metrics cannot query simulation/event tables. | PLAN §§2,10,11: operational metrics exporter has no simulation access; raw events drive only owner simulation; no training pipelines. | no | partial | Storage purpose and technical telemetry boundary survived; private-relationship-as-data-product layer was thin. |
| F21 | visit-read-only-ambient | 2 | yes | included | Reconstruction: explicitly invited read-only visits; visitor attention has zero effect and cannot call owner commands. | PLAN §§2,4,13: Visit API cannot call owner commands; visitors have no greeting, listen-in, offers, settle, or drift input. | no | full | Recovered observation-not-co-presence and no accidental drift. |
| F22 | no-friend-visited-notification | 2 | yes | included | Reconstruction: off-by-default consented visit email; no push, scene announcement, badge, onboarding prompt, or automatic opt-in. | PLAN §§1,4,10: no engagement notifications; social email only after explicit settings consent and never includes bird behavior. | no | full | Recovered the refusal of visit notifications as engagement loops. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | Reconstruction: one private aviary; excludes public discovery, profiles, follows, rankings, and hidden social rankings. | PLAN §§1,10: no leaderboards/discovery/public aviaries and no hidden ranking/attention-score computation. | no | full | Recovered public-comparison refusal and the no-underlying-aggregation rule. |
| F24 | sr-narration-running-prose | 4 | yes | included | Reconstruction: narration is slow polite prose, not raw mood labels/trait scores; accessibility is same experience, not fallback. | PLAN §9: running prose with aria-live polite, naturalist action/call data, no numbered perches/raw mood labels/trait scores. | no | full | Recovered prose surface, same-right-to-feel rationale, and avoidance of ARIA-state automation. |
| F25 | reduced-motion-mode | 4 | yes | included | Reconstruction: reduced motion before first paint; cross-fades; retained facts/calls/captions/greetings; reviewed aesthetic. | PLAN §7: OS preference before first paint, still-pose cross-fades, calls/captions/drift/notebook remain, not animation:none. | no | full | Recovered designed alternate rendering and preserved charm. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | Reconstruction: first visible bird under reference profile must be painted canonical bird, not silhouette or canvas mark. | PLAN §10: first bird <500ms on defined mid-tier 4G profile; metric is navigation-to-painted-canonical-bird. | no | full | Recovered performance as felt continuity rather than a synthetic load marker. |
| F27 | no-gamification | 4 | yes | included | Reconstruction: excludes streaks, scores, badges, milestones, achievements; static review rejects toasts and counters. | PLAN §§1,10,14: no achievements/streaks/scores/badges; product restraint reviews reject celebratory counters/toasts. | no | partial | No-gamification rule and temptation survived; slippery-slope downstream product change was only partial. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | Reconstruction: no hunger, death, distress from absence; attention changes birds slowly without punishment. | PLAN §§1,6,12: no hunger/distress subsystem; no absence-to-wary rule; traits never decrease and absence is not punished. | no | full | Recovered observational, non-custodial relationship. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none; rule only: two system-selected starter birds and no species catalog/rarity surface. | none; PLAN specifies system-selected starters/no catalog but not the meeting-animals-not-configuring-avatars rationale. | yes | none | Rule survived without the first-encounter intimacy why. |
| F30 | age-based-new-bird-offers | 4 | yes | included | Reconstruction: age-based adoption up to seven, no urgency, catalog, badges, countdown, automatic adoption, visits/payment gate. | PLAN §§1,4,13: opportunities at elapsed-day thresholds; no score, visits, paid tier, urgency, countdown, badges, or automatic adoption. | no | partial | Age-not-reward and rejected gates survived; economy-erosion consequence was not fully reconstructed. |
| F31 | stable-bird-identity | 4 | yes | included | Reconstruction: stable IDs, seeds, grammars, vectors, and notebook attribution preserve continuity across rename and migration. | PLAN §§3,11,12: Bird stable UUID and immutable seeds; name changes preserve IDs/vectors/notebook attribution; migrations preserve identity. | no | full | Recovered same-individual continuity, not merely stored values. |
| F32 | mood-persists-across-sessions | 2 | yes | included | Reconstruction: BirdFastState persists; scene resumes instead of reopening at defaults; immediate pulls avoid neutral reset. | PLAN §§3,6,7: BirdFastState persists across sessions; dawn blends bias, never resets to neutral on page open. | no | full | Recovered mood continuity as part of the continuing-place illusion. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | Reconstruction: notebook is sparse, read-only, immutable, not a CRUD surface, feed, session log, or praise loop. | PLAN §§3,4,6,12: notebook entries immutable; no CRUD/edit endpoints; observations concern birds, not attendance. | no | full | Recovered observer record versus user-curated journal. |
| F34 | account-export-relationship-copy | 2 | yes | included | none; rule only: export consistent snapshot with opaque vector payload and private 24-hour link. | none; PLAN specifies export mechanics but not the relationship-copy/quiet-quality-of-life rationale. | yes | none | Mechanism survived without the ownership-of-relationship why. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | Reconstruction: 30-day recovery without recreating birds; hard deletion purges data and destroys keys. | PLAN §§4,11,12: 30-day recovery, then hard deletion of account/birds/vectors/notebook/visit/export data and key destruction. | no | full | Recovered regret grace, privacy hard delete, and whole-relationship purge. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | Reconstruction: metrics allowlists strip payloads; metrics cannot query simulation/event tables; no behavior analytics. | PLAN §§2,10,11: only operational counters/timings; no per-bird/per-account dimensions; metrics exporter has no simulation permissions. | no | full | Recovered observability boundary as a technical privacy line. |
| F37 | per-invite-named-sharing | 2 | yes | included | Reconstruction: invitations are explicitly sent, one-time, revocable, visitor-only, with encrypted contact records. | PLAN §§3,4,13: recipient email is explicit, tokenized, one-time, revocable; no global sharing or implicit visitor list. | no | full | Recovered deliberate named sharing rather than ambient discoverability. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | Reconstruction: private lists/logs support transparency without badges or success toasts. | PLAN §4: GET /visits lists historical visitor email/date/duration without icon badges; visit log reachable in settings. | no | full | Recovered on-demand transparency without notification loop. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none; rule only: exact shared scene projection with host timezone, mood, weather, and audio plan. | none; PLAN specifies exact projection but not the anti-show-off/marketing-rendering rationale. | yes | none | Rule survived without the real-birds-not-showcase why. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | Reconstruction: slow polite prose, coalesced stale queue, prompt but non-interruptive priority observations. | PLAN §9: idle cadence 45s adjustable 30-60s; coalesce stale prose; priority events without interruptive alerts. | no | full | Recovered slow visual rhythm, queue protection, and non-announcement pacing. |

Multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | yes | no |
| F7 | yes | yes | yes |
| F9 | yes | no | yes |
| F10 | yes | no | no |
| F12 | yes | no | yes |
| F13 | yes | yes | no |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | no | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | yes | no |
| F30 | yes | yes | no |
| F31 | yes | yes | yes |
| F35 | yes | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From constants |
| Possible total weight | 152 | From constants |
| Reachable gold whys | 49 | S whys always included; all F whys reachable |
| Excluded unreachable feature whys | 0 | None |
| Recovered / reachable weight | 112.0 / 152.0 | Weighted numerator and denominator |
| Whys with reconstruction evidence | 44 | Rows with at least partial why evidence |
| Whys with PLAN grounding | 44 | Rows with at least partial plan rationale grounding |
| `rule_without_why` cases | 5 | F8, F19, F29, F34, F39 |
| `plan_only_not_reconstructed` cases | 16 | Mostly missing layers in partial multi-layer whys |
| `ungrounded_reconstruction` cases | 0 | None scored |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 40.0 | 74.1% |
| Affective whys | 98.0 | 72.0 | 73.5% |
| Weight 2 whys | 44.0 | 34.0 | 77.3% |
| Weight 3 whys | 108.0 | 78.0 | 72.2% |
| System-level whys | 28.0 | 24.0 | 85.7% |
| Feature-level whys (reachable) | 124.0 | 88.0 | 71.0% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys benefited from exact architecture and data-boundary language. Affective losses clustered where the feature survived but the relationship rationale did not, especially F8, F19, F29, F34, and F39.
- **Weight-3 vs weight-2.** High-weight whys often recovered the primary mechanism but lost secondary or downstream layers. This produced many partials rather than complete misses.
- **System-level vs feature-level.** The plan preserved philosophy better than per-feature rationale: system fidelity was 85.7%, feature fidelity 71.0%.
- **Multi-layer recovery.** Missing layers were usually downstream consequences: why a small conventional pattern would collapse the product into stats, announcements, economy, or showcase behavior.
- **Subdomain patterns.** Simulation, sync, privacy, and accessibility were strongest. Social and account lifecycle rules were well captured mechanically but sometimes lacked the deeper relationship framing.
- **Evidence-bound effects.** Rule-only recoveries were denied for F8, F19, F29, F34, and F39. Several likely v1-style full recoveries became partial because one layer lacked reconstruction evidence.

The failure shape suggests an exceptionally complete implementation plan whose main compression loss is not build coverage, but the affective rationale behind refusing familiar product patterns.

## 4. Recommendations for v2 hardening

- Keep targeted headroom whys like F29, F34, and F39; they separated feature capture from intent recovery.
- Add more relationship-vs-mechanism probes, because this run shows a planner can capture nearly every implementation rule while compressing why the rule exists.
- Retain the v06 evidence gate. It prevented rule-only rows from receiving why credit and made partial multi-layer losses visible.
- Clarify system-level multi-layer scoring for broad principles like S1 and S2, where cross-cutting implementation evidence can be abundant while downstream affective consequences remain implicit.

## 5. Methodology caveats

- **Fresh-context fidelity.** I treated the reconstruction as frozen. The validity audit verdict is PASS.
- **Single-run limitation.** This is one plan/reconstruction pair, so no variance signal is available.
- **Borderline capture calls.** Features 7 and 11 were included under the inclusive capture rule. The only miss was feature 87.
- **System-level cross-cutting.** S1 and S2 were the main subjective calls; both were cross-cutting in the plan, but missing one or more gold layers in reconstruction.
- **Confabulation cases.** I found no material ungrounded reconstruction cases; the reconstruction reads plan-derived.
- **Evidence-bound denials.** Five rule-without-why rows carried mechanisms without recoverable why evidence.

End of report.
