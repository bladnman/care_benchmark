# REPORT - CARE run 001

> Phase 2B scoring report for Pocket Aviary, wave_003/run 001. Frozen reconstruction was read-only; score JSON remains numeric/bounded only.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **96.7%** |
| Intent fidelity | **77.0%** |
| Combined quality | **9644** |

**Diagnostic split:**

- System-level fidelity: **89.3%**
- Feature-level fidelity: **74.2%**

**(Planning, fidelity) coordinate:** `96.7, 77.0`

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-09T02:40:26Z |
| Candidate model | claude-opus-4-7 |
| Candidate effort | high |
| Candidate harness | claude-code |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **116 / 120** = **96.7%**. Four features were missed: greeting stagger (#36), click-anywhere settle undo (#52), empty-aviary state (#68), and privacy-policy link (#87).

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 18 | 90.0% |
| aviary_layout.md | 18 | 17 | 94.4% |
| accounts_sync.md | 18 | 17 | 94.4% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 18 | 100.0% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **116** | **96.7%** |

| # | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes |  |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes |  |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes |  |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes |  |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes |  |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes |  |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Captured inclusively: domain definitions are embedded across scope/data/API sections rather than in a glossary block. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes |  |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes |  |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes |  |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes |  |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes |  |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes |  |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes |  |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes |  |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes |  |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes |  |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes |  |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes |  |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes |  |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes |  |
| 22 | Mood-shaped idle motion | bird_engine.md | yes |  |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes |  |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes |  |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Borderline captured: starter pair is specified, but the no-catalog rationale is only implicit. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes |  |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes |  |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes |  |
| 29 | Mood persistence across sessions | bird_engine.md | yes |  |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes |  |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes |  |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured for product UI/API; JSON export exception is treated separately under account export. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes |  |
| 34 | Greeting variation by absence length | interactions.md | yes |  |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes |  |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | no | Greeting variation is captured, but explicit multi-bird greeting stagger is absent. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes |  |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes |  |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes |  |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes |  |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes |  |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes |  |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes |  |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured inclusively: tab/session end is implied by presence termination and no-negative-drift handling; explicit opt-in copy is thin. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes |  |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes |  |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes |  |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes |  |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes |  |
| 50 | No streak counter, no "days visited" display | interactions.md | yes |  |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes |  |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | Undo window is captured, but the click-anywhere affordance is not specified. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes |  |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes |  |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes |  |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes |  |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes |  |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes |  |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes |  |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes |  |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes |  |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes |  |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes |  |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes |  |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes |  |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes |  |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes |  |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-aviary state between adoption and first bird arrival is specified. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes |  |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes |  |
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
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes |  |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes |  |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link in account settings is specified. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes |  |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes |  |
| 90 | Visits default OFF for new accounts | social_optional.md | yes |  |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes |  |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes |  |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured inclusively through the absolute refusal of social-network surfaces, profiles, and comments. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes |  |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes |  |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes |  |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes |  |
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
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes |  |
| 117 | Out of scope: native mobile app | non_goals.md | yes |  |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes |  |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes |  |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes |  |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **89.3%** (25/28 weighted).

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | B identified? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System-level intent: "Felt aliveness over time" and "aviary appears already in motion". | PLAN.md sections 5.1, 7.2, 14: server tick, no spinner, procedural calls, first frame already in motion. | yes | yes | no | full | Aliveness survives as cross-cutting architecture, loading, audio, motion, drift, and testing concern. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md System-level intent: "notice, never announce" plus refusals of toasts, notifications, visit-frequency widgets. | PLAN.md sections 1, 4, 12.7, 14: no Welcome back, no streaks, no visit notifications, no engagement surfaces. | yes | yes | no | full | The reconstruction preserves the anti-announcement stance and its spread across return, visits, gamification, and API shape. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md System-level intent: "specific particulars" and "naturalist prose" for notebook and narration. | PLAN.md sections 5.7, 9.1, 14: notebook templates prefer named birds and specific moments; narration uses the same voice. | yes | yes | no | full | Specific naturalist observation survives in notebook, narration, captions, and anti-gamification surfaces. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md sections 1, 7.8, 8.4: two starters, cap seven, single horizontal scene, restrained listen-in mix. | no | yes | no | partial | The plan preserves restraint, but the frozen system-level reconstruction does not identify the depth-over-variety rationale directly. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System-level intent: naturalist notebook/narration; settings, unsupported browser, audio failure, email recovery are "matter-of-fact". | PLAN.md sections 7.2, 9.5, 13: naturalist prose for product surfaces; system/error surfaces are matter-of-fact. | yes | yes | no | full | The voice split is explicitly preserved across product and system surfaces. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md Simulation engine: presence is the "primary input" and the 4-minute window honors "watching birds without moving". | PLAN.md sections 4, 5.3, 5.5, 14: validated presence pings, focus/visibility/input conjunction, no negative drift, no streaks. | yes | yes | no | partial | The core precision and anti-inflation rationale survive; settle/tab-close equivalence is only implied, not reconstructed. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System-level intent: server-authoritative continuity; clients "never write personality state". | PLAN.md sections 2, 6, 14: single canonical record, server tick, additive deltas, no client personality writes. | yes | yes | no | full | Server authorship, sync coherence, and no-LWW failure avoidance all survive. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System-level intent: telemetry pipeline has no simulation DB credentials; privacy is "architectural; not a policy". | PLAN.md sections 2, 10.6, 12.6, 14: aggregate-only telemetry, no per-bird analytics, data-pipeline separation. | yes | yes | no | full | The private-relationship data boundary is recovered as architecture, not policy language. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System-level intent: accessibility is "a designed surface, not a checklist" and not a stripped fallback. | PLAN.md sections 1, 9, 11.1, 14: narration, reduced motion, captions, keyboard, contrast, a11y CI ship with v1. | yes | yes | no | full | Designed charm, alternate reduced-motion rendering, and day-one shipment all survive. |

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | yes |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: first-frame draw (section 7.2), procedural audio (section 8), server tick (section 5.1), quiet-field loading (section 7.2), reduced motion (section 7.7).
- S2: no welcome surfaces (section 1), no visit notification (sections 1/4), no streaks (section 12.7), no engagement feature flags (section 11.5), API omits visit-frequency aggregates (section 4).
- S3: naturalist notebook (section 5.7), narration (section 9.1), captions (section 9.2), named birds (section 3), no comparison surfaces (section 1).
- S4: cap seven (sections 1/11.3), single horizontal scene (section 7.8), no UI chrome inside aviary (section 7.3), listen-in preserves other birds (section 8.4).
- S5: naturalist notebook/narration (section 9.1), matter-of-fact settings (section 9.5), audio failure copy (section 8.5), unsupported browser/sign-in copy (sections 10.7/12.10).
- S6: presence conjunction (section 5.3), drift input (section 5.5), no negative drift (section 5.5), no streaks (section 1), visitor presence rejected (section 4).
- S7: server tick (section 5.1), only worker writes personality (sections 2/6), snapshots read canonical state (section 5.8), no LWW (section 6.3), migrations preserve vectors (section 11.6).
- S8: synthetic UUID/email boundary (section 3), no per-bird analytics (section 10.6), telemetry credentials separation (section 12.6), export/deletion rights (sections 6.7/6.8).
- S9: narration (section 9.1), reduced-motion renderer (section 7.7), captions (section 9.2), keyboard/focus (section 9.4), a11y CI blocks deploy (section 9.8), v1 scope freeze (section 11.1).

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity: **74.2%** (92/124 weighted). Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md: raw presence signals and server validation; presence is the "primary input". | PLAN.md sections 4, 5.3: visibility, focus, and last-input conjunction drives drift. | no | partial | Precise conjunction recovered; individual failure cases and silent product-wide corruption are compressed away. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md: low-pass drift supports week-scale change; "measurable at week 1, visible at week 3". | PLAN.md section 5.5: low-pass state, one-week instrument target, three-week visible target, no negative drift. | no | partial | Calibration survives; the too-fast/Tamagotchi vs too-slow/screensaver failure band is incomplete. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md: asymmetry rule means negative delta becomes zero; neglect produces ambient quietness, not punishment. | PLAN.md sections 5.5, 12.1, 14: monotonic clamp, no negative drift, no Tamagotchi. | no | full | Monotonicity, anti-punishment, and ambient return state all survive. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md: recorded or looped audio would sound canned and fail real-time chorus variation. | PLAN.md sections 5.6, 8, 12.3, 14: procedural synthesis, real chorus, no recorded fallback. | no | full | Procedural audio is recovered as affective spine plus chorus and bundle constraint. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none | none | yes | none | The motion examples survive, but not the rationale that mood must be readable without labels. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | The cap survives as a rule; the empirical recognizability ceiling does not. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md: server-authoritative continuity and migrations never reset or rebuild personality. | PLAN.md sections 3, 6, 11.6, 14: persisted canonical vectors, server tick, no LWW, migration preservation. | no | partial | Canonical persistence and downstream sync survive; the deletion-of-the-known-bird rationale is not explicit. |
| F8 | vector-never-shown-numerically | 2 | yes | included | RECONSTRUCTION.md: numeric vectors stay out of product UI so the client cannot surface gamification. | PLAN.md sections 3, 6.8, 14: no product UI vector exposure; export exception only for explicit user data. | no | full | The stat-management risk is recovered, with the plan export exception kept narrow. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md: the bird greeting is the entire welcome and reads as "the bird noticed me". | PLAN.md sections 1, 7.6, 14: greeting varies by absence, boldness, mood; no Welcome back surface. | no | partial | Notice-vs-announcement survives; absence/boldness/procedural variation is mostly rule-level. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md: no Welcome Back because the bird greeting is the entire welcome. | PLAN.md sections 1, 4, 14: no toast/banner/modal/text; bird greeting is the entire welcome. | no | full | The textual-welcome refusal and its variants survive. |
| F11 | settle-is-opt-in | 2 | yes | included | none | none | yes | none | Settle exists, but the why for optionality and ordinary tab closing is not recovered. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md: read-only, server-generated, sparse naturalist prose about bird-and-aviary observations. | PLAN.md sections 3, 5.7, 9.1, 14: lowercase present-tense entries, rare cadence, read-only notebook. | no | partial | Voice and sparseness survive; the stock-event-log failure is not clearly reconstructed. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md: presence raw signals are sent so the server can validate the conjunction. | PLAN.md sections 4, 5.3: visibility, focus, pointer/key activity, ping cadence, server validation. | no | partial | Implementation precision survives; individual signal misses and silent population drift are mostly absent. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md: refuses streaks, visit-frequency widgets, engagement surfaces, and user-behavior notebook entries. | PLAN.md sections 1, 4, 10.6, 12.7, 14: no streaks, calendars, visit-frequency aggregates, or user-behavior observations. | no | full | The anti-counter rationale and adjacent disguises survive. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md: the first frame is "the aviary" and the quiet field is "never white, never a spinner". | PLAN.md sections 7.2, 10.3, 12.5, 14: snapshot first draw, no fade-in, quiet field, no spinner. | no | full | First-frame continuity and quiet-field loading are fully recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md: email is never a key, partition, log dimension, or telemetry attribute. | PLAN.md sections 3, 14: synthetic UUIDs, encrypted email, no email identifiers outside account lookup. | no | partial | PII containment survives; the impossible-to-retrofit rationale is not reconstructed. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md: tick keeps mood, drift, call scheduling fresh and keeps the aviary continuing without the viewer. | PLAN.md sections 2, 5.1, 6, 14: server-side tick, live/dormant queues, canonical snapshots, no client tick. | no | full | Tick authorship, sync coherence, and client-tick collapse all survive. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md: additive deltas prevent the morning drift deleted failure mode. | PLAN.md sections 6.2-6.4, 12.2, 14: one writer, append-only events, additive deltas, no absolute client values. | no | full | The LWW failure and one-writer implementation rule are fully recovered. |
| F19 | sync-conflict-tone | 2 | yes | included | none | none | yes | none | Matter-of-fact system tone survives, but not the why that charm in error contexts reads evasive. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md: aggregate-only telemetry is trust/privacy; the system cannot answer "which user is most engaged". | PLAN.md sections 2, 10.6, 12.6, 14: no per-bird analytics, no ML training fields, pipeline cannot read simulation DB. | no | full | The private-relationship telemetry boundary is fully recovered. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md: visits are read-only/render-only so visitor presence cannot drift host birds. | PLAN.md sections 4, 12.9, 14: visitor session is render-only, no pings, no write paths. | no | full | Observation-not-co-presence is recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md: refuses notifications by default and friend-visited notifications. | PLAN.md sections 1, 4, 14: no push/email/in-product visit notification by default; visit log only. | no | full | Notification-as-attention-driver is recovered through the anti-announcement/social loop framing. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | No leaderboards/discovery survives as a rule; the comparison-surface rationale does not. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md: narration gives "a sense of the morning, not a stream of events". | PLAN.md section 9.1: naturalist prose, polite live region, cadence hints, not a state-change list. | no | partial | Running prose and equal-feeling surface survive; the ARIA-automation failure is less explicit. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md: reduced motion is a designed path with cross-fades, preserved calls, mood, and day-night. | PLAN.md sections 7.7, 9.3, 14: alternate renderer, calls/captions, mood/day-night preserved, no stripped fallback. | no | full | Reduced-motion charm is fully recovered. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md: first-frame fidelity reads as product meaning; missed first-bird budget reads as product breaking. | PLAN.md sections 10.1-10.3, 12.5: p75 500ms, quiet-field first paint, snapshot preload and edge cache. | no | full | The affective-performance bridge survives. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md: no gamification, no engagement widgets, no affective/gamified experiments under flags. | PLAN.md sections 1, 10.6, 12.7, 14: no scores/streaks/badges/XP/ranks; code review rejects counters. | no | full | The absolute anti-gamification rule and slope-risk survive. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md: no Tamagotchi; neglect produces ambient quietness, not punishment. | PLAN.md sections 1, 5.5, 7.6, 14: no death/hunger/distress; no negative drift; no departure. | no | full | The observational-not-custodial rationale survives. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | Starter birds are included, but the meeting-animals-not-configuring-avatars why is not recovered. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md: aviary age, not engagement, drives growth by relationship time rather than optimization. | PLAN.md sections 5.7, 11.3, 14: age-paced offers, not visit count/score/tier; no feature-flagged gamified experiments. | no | full | Age-based growth and anti-reward-loop rationale are fully recovered. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md: bird IDs are never reissued and protect continuity across renames, sync events, and migrations. | PLAN.md sections 3, 11.6, 14: stable bird identity, renaming never affects identity, migrations preserve values. | no | partial | Continuity survives, but the retroactive-loss and identity-vs-data distinction are compressed away. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md: mood persistence means the user never sees a "mood reset" on tab open. | PLAN.md sections 1, 5.4, 14: mood persisted in canonical row and carried across sessions. | no | full | Mood continuity rationale survives. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md: notebook is read-only and about bird-and-aviary observations, not user behavior. | PLAN.md sections 3, 4, 5.7, 14: read-only notebook, worker-written entries, no user-behavior templates. | no | full | Observer-record framing survives. |
| F34 | account-export-relationship-copy | 2 | yes | included | RECONSTRUCTION.md: account export exists because "the user's data is theirs". | PLAN.md sections 1, 6.8, 14: export JSON snapshot and explicit user-data context. | no | full | Relationship ownership rationale survives. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md: sessions remain valid during soft-delete so the user can recover; hard-delete purges records. | PLAN.md sections 1, 6.7, 14: 30-day soft-delete, recovery, hard purge of account-tied records. | no | partial | Recovery and purge survive; the privacy claim behind hard deletion is not explicit. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md: telemetry separation is structural and cannot answer engagement questions. | PLAN.md sections 2, 10.6, 12.6, 14: no per-account/per-bird dimensions; telemetry cannot read simulation DB. | no | full | Technical boundary, not policy language, survives. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md: visits are host-issued, per-invite, opt-in, revocable, expiring, and not social networking. | PLAN.md sections 1, 4, 12.9, 14: per-invite visits, no friend-of-friend chains, visitor session render-only. | no | full | Deliberate named sharing survives. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md: visit events power a host-local visit log, not cross-account analytics or notifications. | PLAN.md sections 3, 4, 14: visit log rows are host-local; no badge/push/email by default. | no | full | Transparency without attention loop survives. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md: visits avoid show-off rendering and remain read-only observation. | PLAN.md sections 1, 4, 14: no show-off rendering; visitor snapshots expose actual read-only aviary. | no | full | Actual-aviary witness rationale survives. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md: narration cadence is 30-60s, faster only for user events, and not a stream of events. | PLAN.md section 9.1: slow cadence, priority bumps, cadence hint, polite live region. | no | full | Slow rhythm, queue protection, and sparse observational updates survive. |

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | no | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | no | yes |
| F9 | yes | no | yes |
| F10 | yes | yes | yes |
| F12 | yes | no | yes |
| F13 | yes | no | no |
| F14 | yes | yes | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | no |
| F25 | yes | yes | yes |
| F27 | yes | yes | yes |
| F30 | yes | yes | yes |
| F31 | yes | no | no |
| F35 | yes | no | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS |
| Possible total weight | 152 | Full possible weighted denominator |
| Reachable gold whys | 49 | All 40 feature anchors were captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions, not failures |
| Recovered / reachable weight | 117 / 152 | Sum of weight x recovery-score |
| Whys with reconstruction evidence | 43 | Included whys with at least one recovered rationale layer |
| Whys with PLAN grounding | 43 | Included whys with matching plan rationale grounding |
| rule_without_why cases | 6 | Mechanism survived without why |
| plan_only_not_reconstructed cases | 12 | At least one plan-grounded rationale layer missing from reconstruction |
| ungrounded_reconstruction cases | 0 | None counted |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 | 38 | 70.4% |
| Affective whys | 98 | 79 | 80.6% |
| Weight 2 whys | 44 | 31 | 70.5% |
| Weight 3 whys | 108 | 86 | 79.6% |
| System-level whys | 28 | 25 | 89.3% |
| Feature-level whys (reachable) | 124 | 92 | 74.2% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Affective whys recovered slightly better by weight (79/98) than functional whys (38/54). The big affective principles survived, especially no gamification and accessibility, but small exception whys such as F19 and F29 dropped.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 86/108, higher than weight-2 whys at 31/44. The load-bearing architecture and anti-gamification layers were easier for the reconstruction to preserve than smaller single-layer details.
- **System-level vs feature-level.** System-level fidelity (25/28) was stronger than feature-level fidelity (92/124). The planner encoded philosophy well; the reconstruction compressed some feature-local rationales into implementation rules.
- **Multi-layer recovery.** Primary causes usually survived. Secondary/downstream layers were the common losses: individual signal-failure cases in F1/F13, retrofit risk in F16, and identity-vs-data nuance in F31.
- **Subdomain patterns.** Server-side sync, privacy boundaries, audio procedure, and accessibility were strong. Smaller product-affect rules around cap seven, sync-error tone, starter birds, and settle optionality were weaker.
- **Evidence-bound effects.** Six apparent rule captures were denied recovery because the why was absent: F5, F6, F11, F19, F23, and F29.

The failure shape suggests a high-quality implementation plan whose blind reconstruction retained the product center but smoothed away narrow, local rationales.

---

## 4. Recommendations for v2 hardening

- Keep targeted feature-level headroom additions. They did real work here: F29, F31, F35, and F37-F40 distinguish rule capture from rationale capture.
- Add more narrow exception whys where the implementation rule is obvious but the reason is not, like sync-error tone or starter-bird no-catalog onboarding.
- Preserve multi-layer scoring for high-weight whys. It showed partial survival cleanly on F1, F2, F7, F13, F16, F24, F31, and F35.
- Consider adding a small score-side flag for plan-rule contradictions. The plan preserved no numeric vector UI while also allowing JSON export of vectors; this was reasonable, but a future scorer could treat similar exceptions inconsistently.
- Keep the >=3 cross-cutting system bar, but ask scorers to document S4-like cases carefully because restraint can be preserved without being named.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** The reconstruction shows no gold-ID leakage and reads as plan-derived. The validity audit verdict is PASS.
- **Single-run limitation.** This is one candidate/reconstruction pair; no variance or temperature sensitivity can be inferred from this slot alone.
- **Borderline capture calls.** I leaned inclusive on #7, #25, #44, and #93. Even if all four were marked missed, planning quality would remain high and all major feature-level anchors except F29 would stay reachable.
- **System-level cross-cutting.** S4 and S6 were the hardest calls. S4 was partial because the plan preserved restraint but the system-level reconstruction did not name the rationale directly. S6 was partial because settle/tab-close equivalence was only implied.
- **Confabulation cases.** No ungrounded reconstruction cases were counted.
- **Evidence-bound denials.** Several mechanism-only recoveries were denied credit under v06. This was most visible on F5, F6, F11, F19, F23, and F29.

End of report.
