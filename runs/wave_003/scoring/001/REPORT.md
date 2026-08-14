# REPORT - CARE run 001

> Variant v06 evidence-bound clean + targeted gold headroom. Exact reconstruction evidence and exact PLAN grounding are recorded for every S1-S9 and F1-F40 row.

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **86.7%** |
| Intent fidelity | **52.7%** |
| Combined quality | **8619** |

**Diagnostic split:**

- System-level fidelity: **82.1%**
- Feature-level fidelity: **45.8%**
- (Planning, fidelity) coordinate: `(86.7, 52.7)`

### Run metadata

| Field | Value |
|---|---|
| Run number | 1 |
| Run label | (blank) |
| Timestamp | 2026-08-14T02:28:35Z |
| Candidate model | gemini-3.7-flash |
| Candidate effort | low |
| Candidate harness | gemini-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **104 / 120** = **86.7%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 3 | 75.0% |
| bird_engine.md | 22 | 21 | 95.5% |
| interactions.md | 20 | 13 | 65.0% |
| aviary_layout.md | 18 | 14 | 77.8% |
| accounts_sync.md | 18 | 16 | 88.9% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 17 | 94.4% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **104** | **86.7%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | captured by plan substance |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | captured by plan substance |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | borderline: quiet/observational refusals approximate the principle, but the phrase is not named borderline |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | captured by plan substance |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | captured by plan substance |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | captured by plan substance |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | no glossary/domain-term section appears |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | captured by plan substance |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | captured by plan substance |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | captured by plan substance |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | captured by plan substance |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | captured by plan substance |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | captured by plan substance |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | captured by plan substance |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | captured by plan substance |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | captured by plan substance |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | captured by plan substance |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | no | species voices are planned, but no per-bird recognizable call signature is specified |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | captured by plan substance |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | captured by plan substance |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | captured by plan substance |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | captured by plan substance |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | captured by plan substance |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | captured by plan substance |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | borderline: two starters are specified, but auto-selection/no catalog is only implicit borderline |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | captured by plan substance |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | captured by plan substance |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | captured by plan substance |
| 29 | Mood persistence across sessions | bird_engine.md | yes | captured by plan substance |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | captured by plan substance |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | captured by plan substance |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | captured by plan substance |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | captured by plan substance |
| 34 | Greeting variation by absence length | interactions.md | no | return greeting is named, but absence-length variation is not specified |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | return greeting is named, but boldness-based order is not specified |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | no | return greeting is named, but stagger behavior is not specified |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | generic anti-retention rules appear, but the specific no welcome-back toast/banner rule is absent |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | captured by plan substance |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | captured by plan substance |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | captured by plan substance |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | offers affect traits, but mood/curiosity-varying reactions are not specified |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | captured by plan substance |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | captured by plan substance |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | settle exists, but close-tab equivalence and no-penalty opt-in semantics are absent |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | captured by plan substance |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | captured by plan substance |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | borderline: system-generated notebook and GET-only API imply read-only, but the refusal is not explicit borderline |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | captured by plan substance |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | captured by plan substance |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | captured by plan substance |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | server simulation continues, but client background render pause is not specified |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | captured by plan substance |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | captured by plan substance |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | captured by plan substance |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | captured by plan substance |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | captured by plan substance |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | borderline: solar lighting/color shifts imply evening palette, but call quieting is absent borderline |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | night state/nightjar behavior is not specified |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | captured by plan substance |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | captured by plan substance |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | captured by plan substance |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | captured by plan substance |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | captured by plan substance |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | captured by plan substance |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | captured by plan substance |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | captured by plan substance |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | captured by plan substance |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | empty-aviary state is not specified |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | no explicit calm palette/no saturated accent specification |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | no | responsive viewport is specified, but never-crop-bird behavior is absent |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | captured by plan substance |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | captured by plan substance |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | captured by plan substance |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | captured by plan substance |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | captured by plan substance |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | borderline: snapshot pulling is planned, but visibility-trigger phrasing is indirect borderline |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | captured by plan substance |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | captured by plan substance |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | captured by plan substance |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | captured by plan substance |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | captured by plan substance |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | captured by plan substance |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | captured by plan substance |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | captured by plan substance |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | captured by plan substance |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | captured by plan substance |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | privacy policy link is not specified |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | email change verification flow is not specified |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | captured by plan substance |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | borderline: invite-only optional visits imply default off, but default state is not named borderline |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | captured by plan substance |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | captured by plan substance |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | captured by plan substance |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | captured by plan substance |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | captured by plan substance |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | captured by plan substance |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | borderline: read-only host snapshots imply actual aviary, but no show-off-mode refusal is not explicit borderline |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | captured by plan substance |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | captured by plan substance |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | captured by plan substance |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | captured by plan substance |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | captured by plan substance |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | captured by plan substance |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | captured by plan substance |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | captured by plan substance |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | captured by plan substance |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | captured by plan substance |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | captured by plan substance |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | captured by plan substance |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | captured by plan substance |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | captured by plan substance |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | captured by plan substance |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | captured by plan substance |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | captured by plan substance |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | captured by plan substance |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | modern evergreen browsers are named, but the last-two-majors matrix is not specified |
| 117 | Out of scope: native mobile app | non_goals.md | yes | captured by plan substance |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | captured by plan substance |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | captured by plan substance |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | captured by plan substance |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **82.1%** (23 / 28 weighted).

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md system intent: "Aliveness should be quiet, procedural, and slowly earned by honest presence"; per-feature: "no wakeup/loading spinner". | PLAN.md 1.1: "Aliveness is expressed through procedural calls, subtle mood-based idle animations, and server-side personality drift"; 6.2 initializes motion mid-cycle; 7.2 ensures no two calls are identical. | yes | yes | no | full | Aliveness survives across procedural audio, motion-in-progress, server state, and loading behavior. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md: "observational attention rather than custodial maintenance or gamified retention loops"; social is "quiet" with "silent visit logging". | PLAN.md non-goals refuse streaks, scores, badges, calendars, and retention push; social scope is "Optional & Quiet"; notebook is "never game logs or streak notes". | yes | yes | no | partial | The anti-announcement cluster survives, but the precise noticed-vs-announced affective distinction and welcome-surface rule do not. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md voice section: naturalist surfaces are "Lowercase, present-tense, specific, calm"; notebook is sparse naturalist observation. | PLAN.md 10: naturalist scene/notebook samples are specific; 5.1 emits distinct observations; non-goals refuse comparison and stat surfaces. | yes | yes | no | full | Specific naturalist prose and refusal of generic game/social framing are both visible. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md 1.1 small group of birds; 1.3 max seven/no social-network surfaces; 6.1 single responsive horizontal viewport; 7.3 listen-in never mutes other birds. | no | yes | no | partial | The plan preserves restraint across scope, layout, and audio, but the reconstruction does not articulate depth-over-variety as a principle. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md: product voice "splits between naturalist calm and plain account utility"; system surfaces are "Matter-of-Fact" with "direct English". | PLAN.md 10 separates scene/notebook/offers/settle naturalist voice from auth, errors, sync, account, and accessibility settings. | yes | yes | no | full | The voice split is explicit in both plan and reconstruction. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md: "honest presence accounting"; presence uses visibility, focus, and activity; strict signals prevent traits from maxing out in days. | PLAN.md 5.1 validates all three criteria and rejects bloated spans; 11.2 warns lenient presence makes traits max out; non-goals refuse absence penalties. | yes | yes | no | partial | Precise attention measurement survives, but settle-vs-close-tab equivalence is not carried. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md: "Canonical inner life belongs on the server"; server is sole writer and clients "never submit absolute trait values". | PLAN.md 2.2 makes the server sole canonical writer; 5.1 server tick writes snapshots; 11.2 uses additive server-authored deltas to avoid overwrite. | yes | yes | no | full | Server tick, coherent sync, and last-write-wins failure mode all survive. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md: telemetry surfaces are "privacy-isolated" and telemetry "never" contains account, bird, trait, interaction, or notebook data. | PLAN.md 9.2 records only system metrics; telemetry payloads never contain relationship data; warehouse has zero read permissions on simulation DB. | yes | yes | no | full | The technical telemetry boundary is recovered, though the relationship-language is compressed. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md: accessibility and performance are "part of the aviary, not an afterthought" and are built/tested "in parallel from Phase 1". | PLAN.md 1.2 includes ARIA prose, reduced motion, captions, budgets; 8 designs the surfaces; 11.2 tests accessibility from Phase 1. | yes | yes | no | full | The reconstruction preserves accessibility as designed experience and launch-scope work. |

Multi-layer system whys:

| Why ID | L1 primary | L2 secondary | L3 downstream |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | no | no |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

Cross-cutting evidence appendix:

- S1: PLAN.md 1.1: "Aliveness is expressed through procedural calls, subtle mood-based idle animations, and server-side personality drift"; 6.2 initializes motion mid-cycle; 7.2 ensures no two calls are identical.
- S2: PLAN.md non-goals refuse streaks, scores, badges, calendars, and retention push; social scope is "Optional & Quiet"; notebook is "never game logs or streak notes".
- S3: PLAN.md 10: naturalist scene/notebook samples are specific; 5.1 emits distinct observations; non-goals refuse comparison and stat surfaces.
- S4: PLAN.md 1.1 small group of birds; 1.3 max seven/no social-network surfaces; 6.1 single responsive horizontal viewport; 7.3 listen-in never mutes other birds.
- S5: PLAN.md 10 separates scene/notebook/offers/settle naturalist voice from auth, errors, sync, account, and accessibility settings.
- S6: PLAN.md 5.1 validates all three criteria and rejects bloated spans; 11.2 warns lenient presence makes traits max out; non-goals refuse absence penalties.
- S7: PLAN.md 2.2 makes the server sole canonical writer; 5.1 server tick writes snapshots; 11.2 uses additive server-authored deltas to avoid overwrite.
- S8: PLAN.md 9.2 records only system metrics; telemetry payloads never contain relationship data; warehouse has zero read permissions on simulation DB.
- S9: PLAN.md 1.2 includes ARIA prose, reduced motion, captions, budgets; 8 designs the surfaces; 11.2 tests accessibility from Phase 1.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity: **45.8%** (54 / 118 weighted). Reachable feature-level whys: **38 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md: presence uses visibility, focus, and activity; strict signals prevent traits from maxing out in days. | PLAN.md 5.1 validates all three presence criteria and 11.2 names lenient measurement as runaway risk. | no | partial | Precise conjunction and drift-risk survive; individual signal failure cases are absent. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md: low-pass server ticks are strictly nonnegative with 1-week instrument and 3-week visible calibration. | PLAN.md 5.1 drift formula and calibration: 1 week instrument-detectable, 3 weeks visibly distinct. | no | partial | Calibration survives; Tamagotchi-vs-screensaver failure band is not reconstructed. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md: neglect becomes "ambient quietness, never penalty or negative trait drift". | PLAN.md 1.3 refuses Tamagotchi mechanics; 5.1 drift deltas are nonnegative. | no | partial | No-punishment intent survives; the two-week return/mistrust consequence is absent. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md: no canned loops; runtime variation ensures "no two calls are identical"; fallback is graceful silence with captions. | PLAN.md 7.2 procedural motifs and variation; 11.2 phase-cancellation mitigation; 7.4 no recorded fallback. | no | full | Procedural audio, chorus artifact risk, and fallback cascade are all carried. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none | none | yes | none | Rule captured, but the mood-as-readable-surface rationale is not recovered. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | Max-seven rule appears, but the empirical recognizability ceiling does not. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md: server is sole writer of canonical state; clients never submit absolute trait values. | PLAN.md 2.2 server owns personality state; 11.2 additive server-authored deltas prevent overwrite. | no | partial | Canonical persistence and downstream sync survive; losing-a-bird-as-relationship loss does not. |
| F8 | vector-never-shown-numerically | 2 | yes | included | none | none | yes | none | Hidden-vector rule appears, but the stat-management/relationship-collapse why does not. |
| F9 | return-greeting | 4 | yes | included | none | none | yes | none | Reconstruction explicitly marks the return-greeting why as NOT RECOVERABLE FROM PLAN. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured: no explicit no welcome-back toast/banner rule. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Feature anchor not captured: settle exists, but close-tab equivalence is absent. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md: sparse lowercase naturalist observations, rate-limited, "never game logs or streak notes". | PLAN.md 5.1 generates sparse notebook prose; 10 gives naturalist notebook voice and no streak notes. | no | partial | Naturalist prose and anti-log framing survive; read-only/not-a-journal consequence is absent. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md: all three presence criteria are required and lenient signals make traits max out in days. | PLAN.md 5.1 requires visible, focused, recent activity; 11.2 gives runaway-trait mitigation. | no | partial | Conjunction and calibration risk survive; individual-signal failure examples do not. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md: no streaks/scores/badges because the product centers observational attention, not retention loops. | PLAN.md 1.3 refuses streaks, visit calendars, and retention pushes; notebook avoids streak notes. | no | partial | The rule and broad refusal survive; number-management and disguised-counter consequences are missing. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md: instant-on mid-action initialization avoids "wakeup/loading spinner" and uses server-time offsets. | PLAN.md 6.2 initializes animation clocks from server_time and renders calm sky without spinners. | no | full | Mid-action first frame, server-snapshot implementation, and quiet loading state all survive. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md: synthetic UUIDs support "PII isolation" and the gateway "never logs PII". | PLAN.md 2.1 resolves email to UUID and never logs PII; 3.1 stores encrypted email separately. | no | partial | PII isolation survives; the impossible-to-retrofit consequence is absent. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md: simulation workers process active/background aviaries and write atomic snapshots; server is sole canonical writer. | PLAN.md 5.1 tick consumes logs, updates state, and writes snapshots; 2.2 clients only render snapshots. | no | full | Server cadence, multi-device coherence, and client-tick collapse risk are covered. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md: append-only event log and additive server-authored deltas prevent multi-device drift overwrite. | PLAN.md 11.2 mitigation: additive server-authored deltas via append-only event log; clients never submit absolute values. | no | full | The no-last-write-wins rule and failure mode survive strongly. |
| F19 | sync-conflict-tone | 2 | yes | included | RECONSTRUCTION.md: system contexts use matter-of-fact tone with "direct utility" and "no charm pretense". | PLAN.md 10 uses matter-of-fact copy for auth, errors, sync conflicts, account/session, and accessibility settings. | no | full | Semantic recovery of the system-clarity exception. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md: telemetry never contains account, bird, trait, interaction, or notebook data; warehouse has zero simulation-DB access. | PLAN.md 9.2 restricts telemetry to system metrics and blocks warehouse reads of simulation data. | no | partial | Technical boundary survives; private-relationship/data-product rationale is absent. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md: visits are read-only ambient observation, with no co-presence and no visitor drift impact. | PLAN.md 1.2 and 2.2 block visitor sessions from submitting presence or interaction events. | no | full | Visitor observation, no co-presence, and no accidental drift are recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md: optional social is quiet with "silent visit logging" and no retention push notifications. | PLAN.md 1.2 defines silent visit logging; 1.3 refuses retention push notifications; social APIs list logs on demand. | no | full | Quiet logging instead of notification is recovered. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | Rule is present, but the public-comparison/different-product rationale is not recovered. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md: ARIA narration is running naturalist prose and accessibility is part of the aviary, not an afterthought. | PLAN.md 8.1 uses naturalist ARIA prose on a slow cadence; 10 keeps narration in product voice. | no | partial | Running prose and same-product accessibility survive; ARIA-automation failure mode is absent. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md: reduced motion uses cross-fades "without disabling the experience" and is not an afterthought. | PLAN.md 6.3 replaces skeletal animation with pose cross-fades and preserves slowed ambient color shifts. | no | partial | Different-rendering accessibility survives; full charm-preservation consequence is compressed. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md: <500ms first bird and no spinner support instant visible aliveness. | PLAN.md 6.2 ties <500ms first bird to no wakeup/loading spinner; 9.1 verifies the budget. | no | full | The performance metric as felt-aliveness bridge is recovered. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md: no streaks, scores, badges, counters, calendars, or push notifications because the product is not a retention loop. | PLAN.md 1.3 explicitly refuses gamification and retention push notifications. | no | partial | Explicit refusal survives; temptation/foothold erosion layers do not. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md: birds never die, starve, or show distress; neglect becomes ambient quietness, never penalty. | PLAN.md 1.3 refuses Tamagotchi mechanics and 5.1 enforces nonnegative drift. | no | full | Observational, non-custodial relationship is recovered. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | Two starters are captured, but the no-catalog/meeting-animals rationale is not recovered. |
| F30 | age-based-bird-offers | 4 | yes | included | none | PLAN.md 1.1 says birds scale up "based on aviary age" and 1.3 refuses gamified loops. | yes | none | Plan carries the age-not-score rule, but reconstruction marks the adoption-milestone why NOT RECOVERABLE. |
| F31 | stable-bird-identity | 4 | yes | included | none | none | yes | none | Stable IDs are structurally present, but same-individual relationship continuity is not reconstructed. |
| F32 | mood-persists-across-sessions | 2 | yes | included | none | none | yes | none | Stored mood exists, but no neutral-reset/continued-while-gone why is recovered. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Notebook generation/read-only implication exists, but observer-record-not-journal rationale is absent. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | Export exists, but relationship-copy/quiet-quality-of-life rationale is absent. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | none | none | yes | none | Soft deletion exists, but regret, hard-delete privacy, and full-residue deletion layers are not recovered. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md: telemetry records only system metrics and never account/bird/trait/interaction/notebook data; warehouse has zero simulation-DB access. | PLAN.md 9.2 draws the same technical telemetry boundary and denies warehouse simulation-DB reads. | no | full | The technical observability boundary is recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | none | none | yes | none | Invite rule appears, but private-relationship/not-publishing rationale is absent. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | none | yes | none | Visit logs are silent, but on-demand transparency vs notification-loop rationale is absent. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Read-only snapshot appears, but actual-aviary/no-show-off rationale is absent. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md: narration is naturalist ARIA prose paced every 30-60 seconds, bumping on user interaction. | PLAN.md 8.1 updates ARIA prose every 30-60 seconds and immediately on offer or settle. | no | partial | Slow observational cadence survives; screen-reader queue overwhelm is absent. |

Multi-layer feature whys:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | no | yes |
| F2 | yes | yes | no |
| F3 | yes | yes | no |
| F4 | yes | yes | yes |
| F7 | yes | no | yes |
| F9 | no | no | no |
| F10 | no | no | no |
| F12 | yes | yes | no |
| F13 | yes | no | yes |
| F14 | yes | no | no |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | no |
| F25 | yes | yes | no |
| F27 | yes | no | no |
| F30 | no | no | no |
| F31 | no | no | no |
| F35 | no | no | no |
| F40 | yes | no | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From benchmark constants |
| Possible total weight | 152 | Full-instance possible denominator |
| Reachable gold whys | 47 | S whys plus captured feature whys |
| Excluded unreachable feature whys | 2 | F10 and F11 anchors were not captured |
| Recovered / reachable weight | 77 / 146 | Sum of weight x recovery over included whys |
| Whys with reconstruction evidence | 31 | Exact rationale evidence in frozen reconstruction |
| Whys with PLAN grounding | 33 | Exact rationale grounding in PLAN |
| rule_without_why cases | 15 | Mechanism survived without the gold rationale |
| plan_only_not_reconstructed cases | 1 | PLAN grounding present but reconstruction did not recover it |
| ungrounded_reconstruction cases | 0 | No ungrounded rationale claims counted |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 | 32 | 59.3% |
| Affective whys | 92 | 45 | 48.9% |
| Weight 2 whys | 42 | 19 | 45.2% |
| Weight 3 whys | 104 | 58 | 55.8% |
| System-level whys | 28 | 23 | 82.1% |
| Feature-level whys (reachable) | 118 | 54 | 45.8% |

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered better by weight (32/54) than affective whys (45/92). The plan gave concrete architecture for server state, synthetic IDs, telemetry boundaries, and drift; affective relationship whys such as F23, F31, F34, and F39 were often reduced to rules.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 58/104, while weight-2 whys recovered 19/42. High-weight system/architecture items did well, but high-weight relationship details still leaked when the plan named a mechanism without its rationale.
- **System-level vs feature-level.** System principles survived much better (23/28) than feature whys (54/118). The reconstruction could infer the product philosophy, but detailed exceptions like no catalog starters, age-based offers, and observer-notebook rationale were often missing.
- **Multi-layer recovery.** Primary causes were the most likely layer to survive. Secondary failure examples and downstream product-consequence layers were commonly dropped, especially in F1, F2, F13, F14, F16, F20, F24, F25, F27, and F40.
- **Subdomain patterns.** Accounts/sync and simulation were strongest. Social, notebook, identity, and export/deletion whys were weakest because the plan specified surfaces and data models more than relationship rationale.
- **Evidence-bound effects.** Several plausible v1-style recoveries were denied because the reconstruction carried only the rule: F5, F6, F8, F23, F29, F31-F35, and F37-F39. F30 is the clearest plan-only case: the plan had age-based growth plus anti-gamification, but the reconstruction marked the rationale not recoverable.

## 4. Recommendations for v2 hardening

- Keep the evidence-bound ledger. It exposed the difference between implementation capture and rationale recovery very clearly in this run.
- Add or preserve more feature-level exception whys around relationship boundaries. The largest losses were not core architecture; they were social, identity, export, deletion, and notebook interpretations.
- Consider requiring planners to write a short rationale sentence for every explicit refusal. Rules like no leaderboards or no trait numbers are easy to list but easy to reconstruct as bare policy.
- Keep multi-layer whys for high-risk product feel areas. Missing downstream-consequence layers were diagnostic without making the scoring degenerate.
- The system-level cross-cutting bar was workable here, but S4 shows a useful edge case: the plan implemented restraint in many places while the reconstruction never named it. Future rubrics could make plan-only system partials more explicit.

## 5. Methodology caveats

- **Fresh-context fidelity.** The prompt states this was a fresh-context phase-2B scorer and the reconstruction was frozen. The validity audit found no gold IDs or scorer vocabulary in the reconstruction.
- **Single-run limitation.** This is one plan/reconstruction/scoring chain, so it has no variance signal.
- **Borderline capture calls.** I leaned inclusive on features 3, 25, 47, 57, 76, 90, and 97. These calls raise planning quality but mostly do not improve fidelity unless the why was reconstructed.
- **System-level cross-cutting.** S4 was the most subjective system call: the plan preserved restraint through bird cap, layout, top-bar, and social limits, but B did not identify the principle. I scored it partial.
- **Confabulation cases.** I did not count any ungrounded reconstruction as recovered. The reconstruction generally stayed plan-derived and often used NOT RECOVERABLE rather than inventing rationale.
- **Evidence-bound denials.** Fifteen included whys were marked rule_without_why. These were the main reason feature fidelity lagged planning quality.
- **Timing.** TIMING.json contained phase 1 and phase 2A timing for run 001; phase 2B timing was not present in this scorer context, so no phase2b timing fields were fabricated.

End of report.
