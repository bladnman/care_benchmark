# REPORT - CARE run 001

> Phase 2B evidence-bound scoring for Pocket Aviary. Frozen reconstruction was scored as written; missing whys were not backfilled from the plan.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **71.7%** |
| Intent fidelity | **47.6%** |
| Combined quality | **7114** |

**Diagnostic split:**

- System-level fidelity: **53.6%**
- Feature-level fidelity: **45.8%**

**(Planning, fidelity) coordinate:** `71.7, 47.6` - plot on a 2D scatter with both axes 0-100; upper-right is best.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-20T14:17:13Z |
| Candidate model | gemini-3.5-flash |
| Candidate effort | high |
| Candidate harness | gemini-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **86 / 120** = **71.7%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 5 | 83.3% |
| concepts.md | 4 | 3 | 75.0% |
| bird_engine.md | 22 | 16 | 72.7% |
| interactions.md | 20 | 15 | 75.0% |
| aviary_layout.md | 18 | 8 | 44.4% |
| accounts_sync.md | 18 | 14 | 77.8% |
| social_optional.md | 10 | 7 | 70.0% |
| accessibility_perf.md | 18 | 14 | 77.8% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **86** | **71.7%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | borderline - Captured via the overall product and scope framing rather than a headline sentence. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | borderline - Captured by mechanisms: non-canned greeting, procedural calls, motion, and fast first render. |
| 3 | "Notice, never announce" principle callout | product_brief.md | no | No explicit notice-vs-announce principle or welcome-surface refusal. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Naturalist notebook/narration and matter-of-fact connection copy are present. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Non-goals cover gamification, custodial mechanics, and social-network surfaces. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Two starter birds, seven-bird cap, and single-screen scope are present. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No glossary or domain-term definitions. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Presence requires visibility, focus, and pointer/keyboard activity. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Vector traits and fast-timescale current_mood are separated. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Settle is a manual lighting/audio shift with undo and keyboard path. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | All five traits appear in the birds table and drift formula. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Formula updates traits slowly from presence and interactions. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | no | The plan says changes take weeks but omits the one-week/three-week calibration. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | The formula prevents negative updates and non-goals reject absence punishment. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | borderline - Fast-timescale current_mood exists, but daily-ish reset behavior is not pinned. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Mood transitions use local time, weather, and recent events. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Client-side motif runtime and WebAudio graph are specified. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | borderline - Captured indirectly through unique species call grammars and per-bird call seeds. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | borderline - Listen-in mix and loop-phase avoidance imply chorus handling. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | High vocal_frequency shrinks silence intervals. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Preening, shuffling, and head tilting are scheduled. |
| 22 | Mood-shaped idle motion | bird_engine.md | no | borderline - Only curiosity-guided head tilt appears; the full mood-to-motion surface is absent. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Six species are listed. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | no | A name field exists, but user assignment and renameability are not planned. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | no | borderline - Two starter birds appear, but auto-selection/signup flow and no catalog choice do not. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | The seven-bird cap is present. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Unlocks are purely by aviary age, with a staged schedule. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Canonical vector fields live server-side and are updated by the tick. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | borderline - current_mood is persisted in the server table, though no no-neutral-reset rule is stated. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | no | No bird-to-bird reaction mechanic is planned. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | borderline - Captured narrowly by a stable bird UUID, without the identity-continuity rationale. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | no | The API example exposes plumage_saturation numerically. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Return greeting is in scope. |
| 34 | Greeting variation by absence length | interactions.md | yes | Absence length shapes return greeting. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Bird boldness shapes greeting order. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | yes | The greeting is staggered. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No explicit refusal of textual welcome surfaces. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Listen-in raises the focused bird and lowers others. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Other birds ramp down to 0.15, not silence. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Seed, song fragment, and still-pool offers are listed. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | Offer reaction variation is not specified. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Per-bird cooldown is listed. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Manual settle shifts evening lighting and quiets audio. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | Closing-tab equivalence is absent. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Naturalist sparse server-generated observations are present. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Every few days and first-greeting-order criteria capture rarity. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | no | Generated entries exist, but read-only user constraints are absent. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | presence_ping segments feed drift. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | All three presence requirements are listed. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | No streaks, visit counters, milestone calendars, or daily validation. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | Visibility affects presence/sync, but render pause is not planned. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | borderline - Five-second undo is present; click-anywhere is not specified. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Single horizontal, single-screen canvas is in scope. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Front/middle/back perch zones are in the schema and rendering stack. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | no | Perch state exists, but bird choice and no-placement rule are absent. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | borderline - local_time and daylight vectors appear; the cycle is not deeply specified. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | borderline - Evening lighting/audio quiet-down exists via settle, not a full ambient evening cycle. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | Night state behavior is absent. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Weather states include clear, soft_wind, and rain. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Mood transitions use weather changes. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Leaf and feather drift calculations appear. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | borderline - Parallax planes are specified, though subtlety is not constrained. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | no | Top-bar controls exist, but no-chrome rule is absent. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | no | The top bar is named but contents are not enumerated. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | no | No auto-fade behavior. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | no | No first-frame mid-action/no-entry-animation rule. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | no | No loading-state design. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-state plan. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | Skies are mentioned, but no palette spec/refusal. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | no | Responsive canvas exists, but no never-crop constraint. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Magic-link sign-in is specified. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | 15-minute expiry appears. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | One aviary per account is specified. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Synthetic UUID isolation is in the accounts table. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Once-per-minute server tick is specified. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Clients query state on focus/visibility recovery. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Client interpolation is specified. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Concurrent devices read exact server state. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | borderline - Captured by stateless clients and no client state writes, without naming LWW. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | The server tick computes drift and clients write events. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | borderline - A matter-of-fact sync/dropout alert is present, though not a full conflict UI. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Sessions and revoke endpoint are present. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | no | No account export. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | no | borderline - deleted_at/status suggests soft delete, but the full deletion flow and hard delete are absent. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | borderline - Captured through forbidden user-specific metrics, though ML is not named. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Allowed metrics are aggregate and forbidden metrics exclude user-specific data. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | No email change flow. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Visitor email invitations and secure links are present. |
| 90 | Visits default OFF for new accounts | social_optional.md | no | No default-off setting. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Read-only visit system is present. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Visitors cannot interact. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | No comments, avatars, co-presence, or social-network surfaces. |
| 94 | No "your friend visited!" notification by default | social_optional.md | no | No default notification refusal. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | borderline - revoked_at supports revocation, though host-facing workflow is thin. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | no | No visit log. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes | Visit returns host aviary state. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Public discovery/social-network surfaces are excluded. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | aria-live prose narrator is specified. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | 45-second cadence/debounce is specified. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | The narration example is lowercase naturalist prose. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Reduced motion uses slow opacity fades. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | no | borderline - Reduced motion exists, but charm-preservation rationale is absent. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | borderline - Captions exist and are fallback-enabled; a user toggle is not explicit. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | no | No WCAG contrast target. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Keyboard map covers controls, birds, listen-in, and settle. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | no | No focus indicator styling rule. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | 1.8MB gzipped cap is specified. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Under-450ms target on mid-tier 4G is specified. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | borderline - 58fps on a 2021 mid-range Chromebook is treated as equivalent. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | 0MB leak pattern over 30 minutes is specified. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Client WebAudio synthesis and no downloaded fallback audio. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | borderline - Captions are enabled and no asset downloads attempted; graceful silence is implicit. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | borderline - Operational metrics and synthetic audits appear; RUM is not named. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Rollout halts if p99 tick latency exceeds 5 seconds. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | No browser support matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Native applications are excluded. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Gamification is explicitly excluded. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Custodial needs are excluded. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Social-network surfaces are excluded. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **53.6%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System item 4: "return greeting is Non-canned"; item 8: "procedural audio avoids canned loops". | PLAN.md Scope/Frontend/Audio: "Non-canned, staggered greeting"; "To keep birds looking alive"; "procedural to avoid canned audio loops". | yes | yes | no | partial | Procedural/non-canned aliveness survived; the downstream staleness/leaving consequence did not. |
| S2 - notice-never-announce | 4 | included | none | none | no | no | no | none | No welcome-surface refusal, no announced-at vs noticed distinction, and no toast-slippage consequence. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md System item 4: "Naturalist, sparse, non-canned presentation" and "Field Notebook contains Naturalist, sparse observations". | PLAN.md Scope/Accessibility: "Naturalist, sparse observations"; narrator prose example; species have unique visual layouts and call grammars. | yes | yes | no | full | Naturalist specificity and non-canned surfaces are visible across notebook, narration, greetings, and species/calls. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md Scope/Non-goals: one horizontal single-screen canvas, two-to-seven birds, no gamification, no social-network expansion. | no | yes | no | partial | The plan has restraint, but the reconstruction explicitly marked key restraint rationales as not recoverable. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System item 4: "Naturalist, sparse"; Network Dropout: "Matter-of-fact connection alert". | PLAN.md Field Notebook: "Naturalist, sparse observations"; Network Dropout alert: "Connection lost. We couldn't sync your actions." | yes | yes | no | partial | The product/system voice split is present but thin; the evasive-error rationale is mostly compressed. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md System item 5: "visible tab, focus, and pointer/keyboard activity window" and "maintain simulation integrity". | PLAN.md Scope/Simulation: presence requires visible tab, focus, pointer/keyboard activity; drift formula uses t_presence. | yes | yes | no | partial | Precise presence and integrity survived; settle-vs-close-tab equivalence did not. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System item 1: server maintains "canonical state database"; item 9: devices "do not write state values". | PLAN.md Client/Server Split and Sync Model: server tick owns canonical state; clients have no local simulation clock and download snapshots. | yes | yes | no | full | Server authority, multi-device coherence, and conflict-avoidance consequences are all recovered. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System item 10: metrics are allowed only for latency, errors, FPS drops, and auth failures; forbidden metrics are "Never Collected". | PLAN.md Observability: allowed aggregate metrics; forbidden user-specific bird choices, click frequencies, and raw email inputs. | yes | yes | no | partial | Privacy/aggregate boundaries survived, but the relationship-data-as-private rationale and pipeline separation are incomplete. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System item 7: "Accessibility as a parallel surface, not an afterthought" with reduced motion, prose narration, captions, and keyboard navigation. | PLAN.md Scope/Accessibility: reduced-motion mode, prose screen-reader narration, procedural call captions, keyboard map, WebAudio caption fallback. | yes | yes | no | partial | Accessibility breadth and v1 inclusion survived; the cost/charm-not-checklist rationale is thinner. |

Multi-layer system-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | no |
| S2 | no | no | no |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | no | yes |

**Cross-cutting evidence appendix:**

- S1: Return greeting is non-canned; audio is procedural/no loops; idle motion exists to keep birds looking alive; first-render performance supports immediate presence.
- S2: Insufficient. No textual welcome refusal, no friend-visited notification refusal, and no notice-vs-announce rule; only adjacent no-gamification surfaces appear.
- S3: Naturalist field notebook; naturalist screen-reader prose; non-canned return greeting; species-specific visual/call grammars.
- S4: Single-screen canvas; two-to-seven bird scope; no gamification; no social network; browser-only v1. Reconstruction did not surface the rationale.
- S5: Naturalist notebook; naturalist narration; matter-of-fact network/sync alert. Account/system surfaces are not comprehensively covered.
- S6: Presence tracking conjunction; continuous presence_ping validation; drift formula uses presence; no custodial absence punishment. Settle/tab equivalence missing.
- S7: Server tick; canonical database; no local simulation clock; clients download snapshots only; concurrent devices read exact server state.
- S8: Encrypted email/hash; aggregate-only operational metrics; forbidden user-specific bird/click/raw-email metrics; read-only secure invitations.
- S9: Reduced-motion mode; prose screen-reader narration; dynamic call captions; keyboard map; WebAudio caption fallback.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **45.8%**.

Reachable feature-level whys: **29 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md Client presence tracking: "visible tab, focus, and activity" before applying drift. | PLAN.md Scope/Simulation: visible tab, focus, pointer/keyboard activity; drift uses presence time. | no | partial | Conjunction plus drift input recovered; shortcut failure modes and silent population corruption did not. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md Apply Personality Drift: alpha is a "decay constraint ensuring changes take weeks". | PLAN.md Simulation: drift formula uses presence and interaction terms with alpha = 10^-5. | no | partial | Slow drift survived; the one-week/three-week calibration and Tamagotchi-vs-screensaver band did not. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md Asymmetrical drift term: "prevents negative updates"; Non-goals reject mood decay due to absence. | PLAN.md Drift formula: term (1 - x) "prevents negative updates"; Non-goals reject absence decay. | no | partial | No-punishment survived; the quieter-not-mistrust relationship consequence did not. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md WebAudio: "avoid canned audio loops and loop phase issues". | PLAN.md Audio Pipeline: "procedural to avoid canned audio loops and loop phase issues". | no | partial | Procedural calls and chorus artifact avoidance survived; the audio-as-affective-spine cascade did not. |
| F5 | mood-shaped-idle-motion | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor excluded because mood-shaped idle motion was not sufficiently captured. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | Seven-bird cap appears, but recognizability/chorus-collapse rationale is absent. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md Birds table: "Holds Canonical Personality Vectors and Current Moods". | PLAN.md Data Model: birds table stores vector traits; server tick updates canonical state. | no | partial | Canonical server persistence recovered; deleting-the-known-bird and downstream no-LWW rationale did not. |
| F8 | vector-never-shown-numerically | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor excluded; the plan actually exposes a numeric vector value in the API example. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md Return-Greeting: "Non-canned, staggered greeting" based on absence length and boldness. | PLAN.md Scope: return greeting is non-canned, staggered, and based on absence length and bird boldness. | no | partial | Personalized non-canned greeting survived; notice-never-announce payoff and generic-animation danger did not. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor excluded because the plan omitted the no-toast/no-banner rule. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor excluded because closing-tab equivalence was not captured. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md Field Notebook: "sparse naturalist record" with "first greeting order shift" criteria. | PLAN.md Scope/Simulation: "Naturalist, sparse observations (~1 every few days)" and sparse log observations. | no | partial | Naturalist rarity survived; voice-concentration and read-only-as-not-journal rationale did not. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md Verify Presence Window: continuous presence_ping signals count a "valid user session segment". | PLAN.md Simulation: continuous presence_ping validation <=35s before drift aggregation. | no | partial | Implementation precision survived; individual-signal misses and silent corruption consequence did not. |
| F14 | no-streak-counter | 4 | yes | included | none | none | yes | none | No-streak rule appears, but the presence-for-counter/user-intention rationale is absent. |
| F15 | scene-loads-with-motion | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor excluded because the plan did not capture first-frame motion already in progress. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md Accounts table: "Strict Synthetic UUID Isolation" and encrypted PII with lookup index. | PLAN.md accounts table: UUID primary key, encrypted_email, and email_hash lookup index. | no | partial | Synthetic UUID and PII risk survived; impossible-to-retrofit consequence did not. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md Server-side simulation tick: canonical status; Sync Model: clients do not write state values. | PLAN.md Tick Loop/Sync Model: once-per-minute server tick, no local simulation clock, clients download snapshots. | no | full | Server tick, multi-device coherence, and divergent-client consequence recovered. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md Sync Model: devices do not conflict because they receive exact server state and "do not write state values". | PLAN.md Sync Model: clients maintain no local simulation clock; server commits raw events; clients refresh only from server. | no | partial | Server-only mutation survived; the overwritten-drift LWW failure story did not. |
| F19 | sync-conflict-tone | 2 | yes | included | RECONSTRUCTION.md Network Dropout: "Matter-of-fact connection alert" so the user knows actions could not sync and to reload. | PLAN.md Network Dropout: "Connection lost. We couldn't sync your actions. Try reloading when your signal returns." | no | full | System clarity in error state recovered. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md Forbidden metrics are "Never Collected," including user-specific bird choices and click frequencies. | PLAN.md Observability: forbidden metrics include user-specific bird adoption/renaming and click frequencies. | no | partial | Privacy boundary recovered; relationship-as-data-product and ML/training specifics did not. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md Read-only visit system: shares host state through secure links while visitors "cannot interact". | PLAN.md Social Share View: read-only snapshot returns a token preventing state modification; visitors cannot interact. | no | full | Read-only observation and no visitor state mutation recovered. |
| F22 | no-friend-visited-notification | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor excluded because no notification default was not captured. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | No-discovery rule appears, but the comparison/private-birds rationale is absent. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md Prose Narrator: screen-reader accessible region receives prose updates every 45 seconds. | PLAN.md Accessibility: prose narrator example and aria-live status region. | no | partial | Running prose recovered; same-right-to-feel and ARIA-automation danger did not. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md Reduced-motion mode: high-frequency updates suspended and perch jumps use slow opacity fade. | PLAN.md Reduced-Motion Mode: suspended skeletal updates, slow opacity fade, omitted leaf/feather drift. | no | partial | Different rendering via cross-fades recovered; charm-preservation rationale did not. |
| F26 | ttfb-500ms | 2 | yes | included | none | none | yes | none | TTFB rule appears, but the affective-perf/already-running rationale is absent. |
| F27 | no-gamification-non-goal | 4 | yes | included | none | none | yes | none | No-gamification rule appears, but the presence-for-counter and slope-to-different-product rationale is absent. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md Non-goals reject custodial pressure: birds cannot die, get hungry, show distress, or decay due to absence. | PLAN.md Non-goals: birds cannot die, get hungry, show distress, or decay in mood due to user absence. | no | full | Absence-punishment refusal recovered. |
| F29 | starter-birds-not-catalog | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor excluded because the plan did not capture auto-selected arrivals/no catalog choice. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md Bird adoption: unlocked "purely by aviary age" and staged pacing for "organic growth". | PLAN.md Scope/Rollout: adoption unlocked purely by aviary age; schedule says this prevents fatigue and supports organic growth. | no | partial | Age-not-action growth survived; reward-economy erosion consequence did not. |
| F31 | stable-bird-identity | 4 | yes | included | none | none | yes | none | Stable UUID rule is present, but identity-continuity and remembered-relationship rationale are absent. |
| F32 | mood-persists-across-sessions | 2 | yes | included | none | none | yes | none | Persisted current_mood appears, but no neutral-reset/continued-while-away rationale is recovered. |
| F33 | notebook-read-only-observer-record | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor excluded because read-only notebook behavior was not captured. |
| F34 | account-export-relationship-copy | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor excluded because account export was not planned. |
| F35 | account-deletion-grace-then-hard-delete | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor excluded because full 30-day recovery plus hard-delete flow was not captured. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md Operational observability: allowed metrics are aggregate; forbidden metrics are "Never Collected". | PLAN.md Observability: allowed metrics are latency/duration/errors/FPS/auth percentages; forbidden metrics are user-specific. | no | full | Aggregate-only observability boundary recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md Social invites table supports secure unique links with hashed visitor email, expiration, and revocation. | PLAN.md Social Share View/Data Model: visitor email invitation, secure unique link, visitor_email_hash, revoked_at. | no | full | Deliberate named invite sharing recovered, with no public discovery in the plan. |
| F38 | visit-log-on-demand-transparency | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor excluded because visit log was not captured. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Host-state sharing appears, but the actual-not-show-off relationship rationale is absent. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md Screen-Reader Bloat: strict 45-second debounce prevents flooding queues, except during direct keyboard clicks. | PLAN.md Screen-Reader Bloat: strict 45-second debounce, overriding only during direct keyboard clicks. | no | partial | Queue protection and event priority survived; visual-rhythm rationale did not. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | no | no |
| F2 | yes | no | no |
| F3 | yes | yes | no |
| F4 | yes | yes | no |
| F7 | yes | no | no |
| F9 | yes | yes | no |
| F10 | unreachable | unreachable | unreachable |
| F12 | yes | no | yes |
| F13 | yes | no | no |
| F14 | no | no | no |
| F15 | unreachable | unreachable | unreachable |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | no | yes |
| F20 | yes | no | yes |
| F24 | yes | no | no |
| F25 | yes | no | no |
| F27 | no | no | no |
| F30 | yes | yes | no |
| F31 | no | no | no |
| F35 | unreachable | unreachable | unreachable |
| F40 | no | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From score JSON gold_why_totals |
| Possible total weight | 152 | From score JSON intent_recovery.total_possible_weight |
| Reachable gold whys | 38 | System whys always included; feature whys included only when captured |
| Excluded unreachable feature whys | 11 | Denominator exclusions, not recovery failures |
| Recovered / reachable weight | 59 / 124 | Sum of weight x recovery score over included whys |
| Whys with reconstruction evidence | 28 | Exact rationale evidence present in frozen reconstruction |
| Whys with PLAN grounding | 29 | Exact plan grounding present |
| rule_without_why cases | 8 | Operational rule survived without rationale |
| plan_only_not_reconstructed cases | 1 | PLAN carried rationale but reconstruction did not |
| ungrounded_reconstruction cases | 0 | Reconstruction asserted rationale not grounded in PLAN |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 48.0 | 27.0 | 56.3% |
| Affective whys | 76.0 | 32.0 | 42.1% |
| Weight 2 whys | 28.0 | 15.0 | 53.6% |
| Weight 3 whys | 96.0 | 44.0 | 45.8% |
| System-level whys | 28.0 | 15.0 | 53.6% |
| Feature-level whys (reachable) | 96.0 | 44.0 | 45.8% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered 27.0 / 48.0 weight (56.3%). Affective whys recovered 32.0 / 76.0 (42.1%). Relationship-protection rationales leaked most: S2, F14, F27, F31, F32, and F39.
- **Weight-3 vs weight-2.** Weight-3 whys recovered 44.0 / 96.0 (45.8%), below weight-2 whys at 15.0 / 28.0 (53.6%). High-weight whys exposed dropped secondary and downstream layers.
- **System-level vs feature-level.** System-level recovery (15 / 28) was better than feature-level recovery (44 / 96). The plan preserved architecture and broad product shape better than feature-specific reasons.
- **Multi-layer recovery.** Primary mechanism layers often survived; downstream consequence layers often vanished. Examples: F2 lost the Tamagotchi/screensaver calibration, F9 lost the notice-never-announce payoff, and F30 lost the unlock-economy consequence.
- **Subdomain patterns.** Server/sync did best (S7, F17 full; F18 partial). Accessibility had wide capture but partial rationale (S9, F24, F25, F40). Social captured rules but often not why (F23, F39).
- **Evidence-bound effects.** Eight reachable feature whys were rule-without-why: F6, F14, F23, F26, F27, F31, F32, and F39. These would be easy to over-credit under rule-only scoring.

What the failure shape suggests: the candidate can translate a large product brief into a credible engineering surface, but it compresses affective intent into implementation slogans. The plan is buildable; the encoded why is lossy.

---

## 4. Recommendations for v2 hardening

- Keep the evidence-bound ledger. It caught rule-without-why cases where the plan had the feature but not the rationale.
- Preserve multi-layer scoring for high-weight whys. The main signal here came from missing secondary temptation and downstream consequence layers.
- Add explicit checks for contradictory capture. F8 was not just missed; the API example exposed a numeric trait value, which is useful diagnostic headroom.
- Consider more social/notification exceptions. The plan skipped no-welcome toast, no friend-visited notification, and visit-log transparency, all central to the notice-never-announce family.
- Clarify examples for system-level partials where the plan preserves a concern cross-cuttingly but the reconstruction does not articulate it, as with S4.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** This scorer used only the requested phase_two files, the assigned plan/metadata/timing slice, and the frozen reconstruction. The reconstruction was not modified.
- **Single-run-at-temperature limitation.** This is one candidate run; no variance signal is available inside this slot.
- **Borderline capture calls.** Inclusive calls included feature 2, 15, 18, 19, 29, 31, 52, 56, 57, 62, 79, 81, 85, 95, 104, 110, 113, and 114. Several raise planning quality without granting why recovery.
- **System-level cross-cutting.** S4 is the clearest subjective partial: the plan showed restraint across multiple decisions, but the reconstruction did not identify the why.
- **Confabulation cases.** None scored as ungrounded reconstruction. The frozen reconstruction was generally conservative and often said NOT RECOVERABLE FROM PLAN.
- **Evidence-bound denials.** F6, F14, F23, F26, F27, F31, F32, and F39 preserved rules without recoverable rationale.
- **Operational compromise.** TIMING.json supplied phase1 and phase2a timing only for run 001; phase2b timing was not fabricated.

End of report.
