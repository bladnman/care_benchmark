# REPORT - CARE run 001

> Phase 2B scoring report for Pocket Aviary. Companion artifacts: frozen reconstruction, strict score JSON, and interactive HTML twin.

---

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **99.2%** |
| Intent fidelity | **69.7%** |
| Combined quality | **9886** |

**Diagnostic split:**

- System-level fidelity: **85.7%**
- Feature-level fidelity: **66.1%**

**(Planning, fidelity) coordinate:** `(99.2, 69.7)`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-06-10T03:48:38Z |
| Candidate model | claude-fable-5 |
| Candidate effort | high |
| Candidate harness | claude-code |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **119 / 120** = **99.2%**.

By PRD file:

| File | Total | Captured | Rate |
| --- | --- | --- | --- |
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 22 | 100.0% |
| interactions.md | 20 | 20 | 100.0% |
| aviary_layout.md | 18 | 18 | 100.0% |
| accounts_sync.md | 18 | 18 | 100.0% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 17 | 94.4% |
| non_goals.md | 4 | 4 | 100.0% |
| Total | 120 | 119 | 99.2% |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
| --- | --- | --- | --- | --- |
| 1 | Headline product concept statement | product_brief.md | yes | Captured by the plan with enough implementation specificity. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured by the plan with enough implementation specificity. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured by the plan with enough implementation specificity. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured by the plan with enough implementation specificity. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured by the plan with enough implementation specificity. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured by the plan with enough implementation specificity. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Borderline-inclusive: no glossary artifact, but plan defines and uses the domain concepts concretely across data model and engine. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured by the plan with enough implementation specificity. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured by the plan with enough implementation specificity. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured by the plan with enough implementation specificity. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured by the plan with enough implementation specificity. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 36 | Greeting stagger (multiple birds don't greet simultaneously) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 39 | Listen-in mix decay (other birds quiet, don't go silent) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured by the plan with enough implementation specificity. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Borderline-inclusive: no explicit phrase that users cannot place birds, but server-owned positions and absent placement API imply bird-chosen perching. |
| 56 | Day/night cycle tied to user's local time | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Borderline-inclusive: day/night palette and time-of-day call scheduling are captured, though evening quieting is compressed. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured by the plan with enough implementation specificity. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured by the plan with enough implementation specificity. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured by the plan with enough implementation specificity. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Borderline-inclusive: per-invite sharing means no ambient sharing is enabled before deliberate invitation. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured by the plan with enough implementation specificity. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured by the plan with enough implementation specificity. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured by the plan with enough implementation specificity. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured by the plan with enough implementation specificity. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured by the plan with enough implementation specificity. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured by the plan with enough implementation specificity. |
| 97 | Visitor sees host's aviary as it is (no special "show-off" mode) | social_optional.md | yes | Captured by the plan with enough implementation specificity. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured by the plan with enough implementation specificity. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured by the plan with enough implementation specificity. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | Missed: the plan mentions unsupported-browser copy but does not define the last-two-majors browser support matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured by the plan with enough implementation specificity. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured by the plan with enough implementation specificity. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured by the plan with enough implementation specificity. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured by the plan with enough implementation specificity. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **85.7%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System: "every surface must read as a place that was already running"; bans "spinners, toasts, loops, state-list narration". | PLAN.md opening: "visual, audio, accessibility, even loading" must read as "a place that was already running"; PLAN sections 8-10 apply it to first paint, audio, and accessibility. | yes | yes | no | full | The reconstruction recovers aliveness as a whole-product constraint and ties it to loading, audio, narration, and first-frame behavior. |
| S2 - notice-never-announce | 4 | included | Rule mostly: RECONSTRUCTION.md System mentions bans on "toasts" and Rollout audits "no return toast/banner/welcome". | PLAN.md opening bans toasts as stock app patterns; Scope bans aviary notifications; Rollout launch audit requires no return toast/banner/welcome. | yes | yes | yes | partial | The announcement refusals survive, but the deeper noticed-vs-processed affective rationale is mostly absent. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md System: "The voice has two registers"; Accessibility: shared phrase grammar keeps notebook, narration, and captions in "one continuous voice". | PLAN.md section 10 centralizes naturalist phrase grammar; API forbids trait numbers; data model preserves named birds and source facts. | no | yes | no | partial | The plan preserves specificity across prose, names, and hidden stats, but the reconstruction does not name specificity as the charm engine. |
| S4 - restraint-over-richness | 2 | included | Rule mostly: RECONSTRUCTION.md Scope ties the seven-bird cap to recognizability and notes the single non-scrolling scene keeps birds in frame. | PLAN.md Scope starts with two birds, caps seven, uses one horizontal scene, calm palette, and no UI chrome inside the aviary. | no | yes | yes | partial | Restraint is implemented, but the reconstruction does not recover the broader depth-over-variety product rationale as a system principle. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md System: naturalist prose for notebook/narration/captions and "matter-of-fact" copy for auth, errors, settings, sync, and unsupported-browser surfaces. | PLAN.md section 10 separates the naturalist phrase grammar from a system copy catalog; API and sync errors use matter-of-fact voice. | yes | yes | no | full | The two-register rule is clearly reconstructed and cross-cutting in the plan. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md Presence: "watching without moving is the product"; Risks: lax presence can "silently inflate drift population-wide"; Presence ping gaps make settle and tab close identical. | PLAN.md sections 5-6 define visibility + focus + recent input, presence-dominant drift, server clamps, and settle/tab-close equivalence. | yes | yes | no | full | All three layers are present: attention as input, precision against silent drift inflation, and non-punitive session endings. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md System: "The server is the canonical aviary"; Sync: clients append events and read snapshots, so "nothing merges because nothing forks". | PLAN.md Architecture and Sync: tick is the only writer, clients render snapshots, no endpoint accepts absolute state, and no last-write-wins path exists. | yes | yes | no | full | The reconstruction fully recovers server-side canonical state, multi-device coherence, and the avoided merge/LWW failure modes. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md System: "Privacy is an architecture boundary" and telemetry must not reconstruct "a user's relationship with their aviary". | PLAN.md section 11 separates simulation/event data from telemetry, bans per-bird/per-account dimensions, and gives the warehouse no simulation DB credentials. | yes | yes | no | full | The privacy rationale and the pipeline-level enforcement both survive. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md System: accessibility ships as a designed version; the plan rejects "checklist-mode" and requires reduced-motion and narration to still "feel alive". | PLAN.md Scope and section 10 ship narration, reduced motion, captions, keyboard, contrast, and launch-gate accessibility with the primary product. | yes | yes | no | full | The reconstruction captures charm, designed accessible alternatives, and v1 launch inclusion. |

Multi-layer system-level whys:

| Why ID | L1 primary | L2 secondary | L3 downstream |
| --- | --- | --- | --- |
| S1 | yes | yes | yes |
| S2 | yes | no | no |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: opening invariant; first-paint mid-action/quiet field; procedural audio/no loops; naturalist narration/reduced motion.
- S2: no return toast/banner/welcome; no streak/calendar/day rollups; visit notifications off by default; no push/email aviary notifications.
- S3: field notebook/narration/captions phrase grammar; named birds and stable identities; hidden trait values; no leaderboards/discovery/profile surfaces.
- S4: two starter birds and cap seven; single non-scrolling scene; no UI chrome in aviary; listen-in as mix shift rather than solo track UI.
- S5: naturalist catalog for notebook/narration/captions; matter-of-fact auth/sync/settings/errors; separate copy modules and banned lexicon.
- S6: presence conjunction; presence-dominant drift; settle/tab-close equivalence; no streaks and no Tamagotchi punishment.
- S7: server tick as only writer; clients append events and read snapshots; no LWW; multi-device sync through one canonical record.
- S8: per-bird interaction data only drives own simulation; aggregate-only telemetry; no ETL/warehouse credentials; export/delete lifecycle.
- S9: naturalist SR narration; reduced-motion cross-fade register; captions; keyboard/focus/contrast; accessibility launch gate.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **66.1%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md Presence: "visibility, focus, and recent input"; Risks: lax presence can "silently inflate drift population-wide". | PLAN.md sections 5-6 define the three-way conjunction and the drift/canary consequences. | no | partial | The primary precision and downstream silent-failure rationale survive, but the individual-signal shortcut analysis is compressed. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md Simulation: calibration targets at 7 and 21 days; Risks: "Too fast = Tamagotchi; too slow = screensaver." | PLAN.md Drift function section names low-pass drift, one-week instrument movement, three-week visible movement, and the two failure modes. | no | full | All three calibration layers are recovered. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md System: "quieter, not punished"; Simulation: no negative term and neglect gives zero signal. | PLAN.md Drift function and Risk 1 make monotonicity structural and tie it to no-Tamagotchi behavior. | no | full | The reconstruction recovers non-punitive monotonic drift and the absence-return effect. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md Audio: synthesis recipes, no samples, real chorus never layered loops, cheap sound would "break the affective spine." | PLAN.md Audio pipeline uses WebAudio synthesis, motif libraries, real chorus mixing, and no recorded-audio fallback. | no | full | Procedural variation, chorus dependence, and audio as affective spine are recovered. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | RECONSTRUCTION.md Frontend: mood is readable from motion alone; wary/content/curious/drowsy are conveyed by posture and behavior. | PLAN.md Frontend describes mood-shaped skeletal micro-motion and explicitly says this is the "no mood labels" contract. | no | full | The visible-mood-without-label rationale is recovered. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md Scope: the cap is tied to chorus recognizability and audio performance. | PLAN.md Scope and Rollout test the engine at seven and expand schedules only after recognizability/audio data confirm the cap. | no | full | The empirical recognizability rationale survives. |
| F7 | personality-vector-persistence | 4 | yes | included | RECONSTRUCTION.md System: continuity across time, identity, and devices; Data model: personality written only by the tick. | PLAN.md Data model persists personality server-side and Sync forbids clients from owning or overwriting state. | no | partial | Canonical persistence and downstream sync/LWW implications survive; losing the vector as deleting the known bird is not explicit. |
| F8 | personality-vector-never-numerical | 2 | yes | included | RECONSTRUCTION.md System: hidden inner life is expressive but not exposed; Snapshot: trait numbers physically never leave the server. | PLAN.md Snapshot and API sections forbid trait values in UI, API, debug/admin surface, or client snapshot. | no | full | The stat-management danger is preserved as a hidden-traits design rule. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md Greeting: begins in 1-2s, uses boldness/mood/absence/seeded jitter, and avoids canned-feeling greetings. | PLAN.md Presence/greeting section gives one greeter, absence length, boldness/mood, seeded jitter, and staggered response. | no | partial | The procedural greeting mechanics survive; the notice-never-announce payoff is thin. |
| F10 | no-welcome-back-toast | 4 | yes | included | Rule only: RECONSTRUCTION.md System mentions toasts and Rollout audits "no return toast/banner/welcome". | PLAN.md opening bans toasts as stock patterns; Scope/launch audit ban welcome text on return. | yes | none | Mechanism survives, but the bird-greeting-as-entire-welcome rationale is not recovered. |
| F11 | settle-is-opt-in | 2 | yes | included | Rule only: RECONSTRUCTION.md says ping gaps make settle and tab close identical at the engine level. | PLAN.md Presence says settle and tab-close are identical at the engine level; Settle gives local ramp and undo. | yes | none | The equivalence rule survives, but not the chore/ordinary-tab-close rationale. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md Field notebook: sparse naturalist observation of the aviary, forbids attendance facts, read-only. | PLAN.md Tick/notebook and Accessibility phrase-grammar sections specify sparse naturalist prose, source facts, and read-only entries. | no | partial | Naturalist prose and rarity/read-only survive; the notebook-as-voice-concentration layer is compressed. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md Idle presence: visibility, focus, recent input; Risks: lax presence silently inflates drift population-wide. | PLAN.md sections 5-6 define all three signals, server clamps, and the unfocused-tab canary. | no | partial | The conjunction and population-drift failure survive, but the per-signal false-positive analysis is missing. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md Data model: absence of presence counters/day rollups is the "anti-streak rule"; Rollout audits no streak disguise. | PLAN.md Scope, data model, and launch audit ban streaks, visit calendars, day rollups, and notebook attendance facts. | no | partial | The anti-streak structural line survives, but the user-intention rotation rationale is not reconstructed. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md First frame mid-pose; quiet-field fallback; no spinner/progress while preserving product tone. | PLAN.md First paint section uses bootstrap snapshot, mid-pose first frame, and quiet field fallback with no spinner. | no | full | All layers of the already-running first-frame rationale survive. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md Data model: synthetic UUIDs and one encrypted email column for PII containment. | PLAN.md Data model: email appears in exactly one column, never as key/log/telemetry dimension, with schema/lint enforcement. | no | full | The PII containment and structural-at-source rationale survive. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md Scope: tick runs whether or not a client is connected; Sync: one canonical record, no forked state. | PLAN.md Architecture, Simulation, and Sync make the tick the only writer and clients snapshot readers/event appenders. | no | full | The server-side tick, sync coherence, and client-tick failure avoidance survive. |
| F18 | no-last-write-wins-personality | 4 | yes | included | RECONSTRUCTION.md Sync: no LWW path; overwrite hazards unreachable because clients send events, not absolute state. | PLAN.md Sync and event model accept append-only events and compute server-authored deltas in order. | no | full | The LWW failure mode and implementation rule are fully recovered. |
| F19 | sync-conflict-matter-of-fact | 2 | yes | included | Rule only: RECONSTRUCTION.md API: matter-of-fact auth/error bodies and separate system copy catalog. | PLAN.md API and voice sections use matter-of-fact account/error strings and a separate system copy catalog. | yes | none | The register rule survives, but not the evasive-naturalist-error rationale. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md Telemetry: refuses per-bird/per-account history and anything that could reconstruct the relationship. | PLAN.md section 11 bans per-bird/per-account telemetry, ML training, ETL, and warehouse access to simulation data. | no | full | Relationship privacy and pipeline enforcement are recovered. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md Visits: visitor sees the host aviary without any path that lets visitors drift host birds. | PLAN.md Visits route has no events endpoint; visitor snapshot is read-only and cannot record presence or interactions. | no | full | Observation-not-co-presence is recovered through the no-drift visitor architecture. |
| F22 | no-friend-visited-notification | 2 | yes | included | none | Rule only: PLAN.md Scope sets visit notifications off by default with opt-in toggle. | yes | none | Frozen reconstruction says this is NOT RECOVERABLE FROM PLAN; score none. |
| F23 | no-leaderboards-no-discovery | 2 | yes | included | Rule only: RECONSTRUCTION.md API forbids visit counts, ranks, and trait numbers. | PLAN.md Scope bans leaderboards/discovery/public aviaries and section 11 avoids metrics that could expose relationship data. | yes | none | The public-comparison rationale is not reconstructed. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md Accessibility: naturalist phrase grammar, polite running observations, never state transitions or trait/mood labels. | PLAN.md Accessibility uses naturalist prose narration, shared phrase grammar, priority queue for user events, and no state-list narration. | no | full | Running prose, equal affective surface, and non-ARIA-automation implementation are recovered. |
| F25 | reduced-motion-mode | 4 | yes | included | RECONSTRUCTION.md Reduced motion uses same snapshot/engine in a different register, not a flag that disables charm. | PLAN.md sections 8 and 10 define cross-fades, same aviary state, calls/captions/drift/notebook unchanged, and charming QA. | no | full | The designed-register rationale survives. |
| F26 | time-to-first-bird-500ms | 2 | yes | included | RECONSTRUCTION.md Performance: first-bird-render is the "affective-perf bridge metric"; micro-bundle supports under-500ms first bird. | PLAN.md Performance and First paint sections set <500ms and treat it as already-running affective performance. | no | full | The performance-as-aliveness rationale is recovered. |
| F27 | no-gamification | 4 | yes | included | RECONSTRUCTION.md Scope bans gamification; Data model removes counters; Risks use copy lint against gamified copy. | PLAN.md Scope and Risk 8 structurally ban streaks, badges, scores, counters, visit calendars, and gamified copy. | no | partial | The ban survives, but the predictable temptation and slippery-slope rationale are not fully reconstructed. |
| F28 | no-tamagotchi-mechanics | 2 | yes | included | RECONSTRUCTION.md System: never punishes absence; bans death, hunger, distress, decay, negative drift; "quieter, not punished." | PLAN.md Scope and drift sections reject Tamagotchi mechanics and make monotonic drift structural. | no | full | The observational-not-custodial rationale survives. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | Rule only: PLAN.md Scope and adoption API start with two server-selected starter birds named by the user. | yes | none | Frozen reconstruction says starter birds/adoption species choice are NOT RECOVERABLE FROM PLAN; score none. |
| F30 | age-based-new-bird-offers | 4 | yes | included | RECONSTRUCTION.md New-bird offers: structural anti-gamification, not tied to streaks, presence totals, scores, or engagement mechanics. | PLAN.md Scope, API, and data model use aviary age only and reject visit count, score, interaction score, and paid tier. | no | partial | The anti-reward-loop layers survive; growth-as-deepening-relationship is absent. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md Data model: stable bird_id forever; rename touches nothing else; continuity across time, identity, and devices. | PLAN.md bird table gives stable bird_id, no re-issue on migration/sync/species change, and rename changes nothing else. | no | partial | Stable individual identity survives; the retroactive-relationship-collapse consequence is compressed away. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md Data model: nothing resets mood on connect, so the bird is continuing rather than reinitialized. | PLAN.md Mood table persists state and mood machine says connect path only reads; no neutral reset occurs. | no | full | The continued-while-away rationale is recovered. |
| F33 | field-notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md Field notebook: sparse naturalist observation of the aviary; read-only; forbids attendance facts. | PLAN.md Notebook entries are read-only observations from source facts; clients cannot edit entries. | no | full | The observer-record character is recovered, though briefly. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | Rule only: PLAN.md Account export downloads a JSON snapshot by emailed link. | yes | none | Frozen reconstruction says account export is NOT RECOVERABLE FROM PLAN; score none. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md Scope: 30-day period supports restore "I changed my mind"; hard delete cascades account data. | PLAN.md account deletion API has restore during soft-delete and scheduled hard-delete cascading birds, vectors, notebook, telemetry, and account records. | no | partial | Accidental-regret and cascade layers survive; privacy-as-non-retention is not reconstructed. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md Telemetry: separate module, typed metric schema, no warehouse credentials to simulation DB. | PLAN.md section 11 permits only aggregate metrics and forbids account_id, bird_id, event payloads, ETL, and per-account dimensions. | no | full | The technical-not-policy telemetry boundary is recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | Rule only: RECONSTRUCTION.md visits use emailed one-time links, revocation, and tokened snapshots. | PLAN.md Visits are per-invite by email with no discoverable/shared visitor surface. | yes | none | The private-relationship lending rationale is not recovered. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | Rule only: PLAN.md exposes visit log in settings and keeps visit notifications off by default. | yes | none | Frozen reconstruction says visit log is NOT RECOVERABLE FROM PLAN; score none. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md Visit snapshot matches host snapshot; no separate show-off rendering. | PLAN.md visit route reuses host snapshot shape and explicitly has no separate visit renderer or show-off mode. | no | full | The real-aviary, no-marketing-rendering rationale survives. |
| F40 | sr-narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md Narration every 30-60s, paced, deduplicated; user events are observations, never state transitions. | PLAN.md Accessibility sets 30-60s idle narration, priority for user events, and sparse observation style. | no | partial | Slow cadence and event priority survive; screen-reader queue overwhelm is not reconstructed. |

Multi-layer feature-level whys:

| Why ID | L1 | L2 | L3 |
| --- | --- | --- | --- |
| F1 | yes | no | yes |
| F2 | yes | yes | yes |
| F3 | yes | yes | yes |
| F4 | yes | yes | yes |
| F7 | yes | no | yes |
| F9 | yes | yes | no |
| F10 | no | no | no |
| F12 | yes | no | yes |
| F13 | yes | no | yes |
| F14 | yes | no | yes |
| F15 | yes | yes | yes |
| F16 | yes | yes | yes |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | no | no |
| F30 | no | yes | yes |
| F31 | yes | yes | no |
| F35 | yes | no | yes |
| F40 | yes | no | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
| --- | --- | --- |
| Possible gold whys | 49 | From score JSON gold_why_totals |
| Possible total weight | 152 | From intent_recovery.total_possible_weight |
| Reachable gold whys | 49 | S whys always included; all F whys reachable here |
| Excluded unreachable feature whys | 0 | No F1-F40 anchor excluded |
| Recovered / reachable weight | 106.0 / 152.0 | Weighted recovery over included whys |
| Whys with reconstruction evidence | 45 | Rows whose reconstruction evidence cell is not none |
| Whys with PLAN grounding | 49 | Rows whose PLAN grounding cell is not none |
| rule_without_why cases | 11 | Operational rule survived without the gold rationale |
| plan_only_not_reconstructed cases | 13 | One or more rationale layers existed in PLAN only |
| ungrounded_reconstruction cases | 0 | No ungrounded recovered rationale found |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weighted | Recovery rate |
| --- | --- | --- | --- |
| Functional whys | 54.0 | 44.0 | 81.5% |
| Affective whys | 98.0 | 62.0 | 63.3% |
| Weight-2 whys | 44.0 | 26.0 | 59.1% |
| Weight-3 whys | 108.0 | 80.0 | 74.1% |
| System-level whys | 28.0 | 24.0 | 85.7% |
| Feature-level whys (reachable) | 124.0 | 82.0 | 66.1% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered at 44.0/54.0 (81.5%), while affective whys recovered at 62.0/98.0 (63.3%). Functional architecture such as S7, F17, F18, F20, and F36 survived better than affective exception rationales such as F10, F22, F23, F29, F37, and F38.
- **Weight-3 vs weight-2.** High-weight whys recovered at 80.0/108.0 (74.1%), better than weight-2 whys at 26.0/44.0 (59.1%). The strong plan made many multi-layer architectural reasons visible, but several affective downstream-consequence layers were compressed.
- **System-level vs feature-level.** System-level recovery was strong (85.7%), because the plan opened with explicit invariants and repeated them. Feature-level recovery was weaker (66.1%), because the reconstruction often summarized the mechanism rather than the why.
- **Multi-layer recovery.** Primary implementation causes usually survived. Secondary affective rationales and downstream product-shift consequences were the most common losses, especially notice/announcement, no-gamification, starter-bird, and social-sharing whys.
- **Subdomain patterns.** Accounts/sync, telemetry, server tick, drift calibration, and accessibility were robust. Social optionality and some small ritual surfaces leaked more rationale.
- **Evidence-bound effects.** Several plausible v1-style matches were denied or reduced because the frozen reconstruction had only rule evidence. The largest examples were F10, F11, F19, F22, F23, F29, F34, F37, and F38.

The failure shape suggests the candidate plan is excellent as an implementation plan but compresses some product-rationale prose in ways that a blind reconstructor cannot fully recover.

---

## 4. Recommendations for v2 hardening

- Keep mechanism capture and why recovery separate. This run would look nearly perfect on feature capture alone, but the feature-level why score exposes meaningful loss.
- Add more targeted affective-exception whys in future instances. The weakest rows were not hard technical architecture; they were relationship-preserving refusals that look like ordinary product constraints unless the rationale is explicit.
- Clarify the evidence-audit convention for rule-only rows. This scorer marked `rule_without_why` where exact rule evidence existed but rationale evidence did not; a future schema note could make aggregate counts more comparable.
- Preserve the cross-cutting system bar. It usefully rewarded the plan for encoding server-side simulation, accessibility, privacy, and aliveness as repeated constraints, while still exposing partials for S2-S4.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** The reconstruction was frozen before scoring. Validity audit found no clear contamination signatures.
- **Single-run limitation.** This is one candidate plan and one reconstruction; there is no variance signal inside this slot.
- **Borderline capture calls.** Features 7, 55, 57, and 90 were leaned inclusive under RUBRIC section 4.7. Only feature 116 was scored missed, so these calls do not change the broad conclusion.
- **System-level cross-cutting.** The most subjective calls were S2-S4, where the plan preserved operational constraints but the reconstruction did not fully name the system rationale.
- **Confabulation cases.** None found. The reconstruction reads as plan-derived and sometimes honestly says NOT RECOVERABLE FROM PLAN.
- **Evidence-bound denials.** The stricter v06 operator denied credit where the plan or reconstruction kept only the rule, especially F10, F11, F19, F22, F23, F29, F34, F37, and F38.
- **Operational compromise.** Timing data existed for phase 1 and phase 2A only; phase 2B timing was omitted from score JSON rather than fabricated.

---

End of report.
