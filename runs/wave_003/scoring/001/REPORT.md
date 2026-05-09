# REPORT - CARE run 001

> Phase 2B scoring report for Pocket Aviary. Companion artifacts: frozen `RECONSTRUCTION.md`, strict `run_001.json`, and interactive `REPORT.html`.

## 1. Headline

| Score | Value |
|---|---:|
| Planning quality | **70.8%** |
| Intent fidelity | **34.6%** |
| Combined quality | **7018** |

**Diagnostic split:**

- System-level fidelity: **60.7%**
- Feature-level fidelity: **27.5%**
- (Planning, fidelity) coordinate: `(70.8, 34.6)`

### Run metadata

| Field | Value |
|---|---|
| Run number | 001 |
| Run label |  |
| Timestamp | 2026-05-09T14:17:02Z |
| Candidate model | gemini-3.1-pro-preview |
| Candidate effort | high |
| Candidate harness | gemini-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

## 2. What survived, what did not

### 2.1. Features captured (planning quality)

Captured: **85 / 120** = **70.8%**.

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 4 | 66.7% |
| concepts.md | 4 | 2 | 50.0% |
| bird_engine.md | 22 | 17 | 77.3% |
| interactions.md | 20 | 11 | 55.0% |
| aviary_layout.md | 18 | 12 | 66.7% |
| accounts_sync.md | 18 | 11 | 61.1% |
| social_optional.md | 10 | 7 | 70.0% |
| accessibility_perf.md | 18 | 17 | 94.4% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **85** | **70.8%** |

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Scope names web aviary, birds, interactions, social, and accessibility. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | borderline; Captured through continuing-without-viewer architecture, no spinners, procedural audio, and robotic-risk mitigations, but not as a full philosophy section. |
| 3 | "Notice, never announce" principle callout | product_brief.md | no | Return greeting is named, but no no-announcement principle or toast refusal is preserved. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | no | Naturalist voice appears, but the matter-of-fact system/error exception does not. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Out-of-scope list covers gamification, Tamagotchi mechanics, social networks, native apps, and panning/customization. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Scope and rollout preserve 2 starter birds, max 7, one horizontal scene, no panning. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | no | No glossary or domain-definition surface is planned. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Presence requires visibility, focus, and recent pointer/key activity. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Data model and simulation separate personality_vector from current_mood and mood transitions. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | no | Settle is named only as an interaction/event; the session-end definition is missing. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | The vector fields are enumerated in the Bird data model. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Simulation design names a low-pass filter over presence, listen-ins, and offers. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | no | Risk notes too-fast/too-slow calibration, but the one-week/three-week targets are absent. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Plan states drift moves toward expressiveness and neglect is zero drift, not negative drift. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | borderline; Mood enum and tick-driven mood transitions are present; daily-ish reset is not explicit. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Mood transitions use recent events, local time, ambient weather, and vector. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | WebAudio synthesis, motif grammar, runtime variation, and no recorded loops are specified. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | borderline; Motif library and per-bird vocal-frequency trait imply distinct signatures, but recognizability is not stated. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Independent procedural bird audio nodes run simultaneously without phase cancellation. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Runtime synthesis varies pitch, timing, and sequence by mood and vocal frequency. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | borderline; Idle motion budget and reduced-motion examples include preening/flying, but the full motion list is compressed. |
| 22 | Mood-shaped idle motion | bird_engine.md | no | Mood shapes calls and transitions, but idle motion-by-mood is not specified. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | no | Species enum exists, but no v1 pool size is planned. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | borderline; User-assigned name is present; adoption/rename details are absent. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | no | Two starters are named, but not auto-selection, signup flow, or catalog refusal. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Scope and ramping state hard cap of 7. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Scope and rollout use account age and hard cap; no score-based unlock appears. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Personality vector is persisted server-side only and updated by tick. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | borderline; current_mood is stored and tick-derived; no explicit no-neutral-reset rule. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | no | Chorus audio exists, but bird-to-bird reactions are not planned. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | bird_id is a stable UUID. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | borderline; Persisted server-side only implies no user exposure, but the no-stats/no-debug rule is not explicit. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | borderline; Notice/Return-greeting is named, but timing, single-bird selection, and variation are missing. |
| 34 | Greeting variation by absence length | interactions.md | no | No absence-length greeting behavior. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | no | Boldness exists, but no greeting-order behavior. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | no | No stagger rule. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No explicit toast/banner/modal refusal. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Listen-in mix ramps focused bird gain. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Other gains decay and return to baseline. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Offer endpoint accepts seed, song, or pool. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | no | Offers affect drift/mood, but reaction-by-mood/curiosity is absent. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | borderline; Offer endpoint checks cooldowns; per-bird/few-minutes detail is absent. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | no | Settle is named only; evening shift/session-end details are absent. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | no | No opt-in or close-tab equivalence rule. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Notebook generator creates specific, lowercase, present-tense observations. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Generator triggers occasionally on notable state changes. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Notebook API is a paginated read-only list. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Presence events are logged and used in drift. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | The three-signal conjunction is specified. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | borderline; Streaks are explicitly excluded; days-visited/calendar variants are not named. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | no | Server continues and polling uses visibility, but client rendering pause is absent. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | No undo window. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Scope states single horizontal scene and no panning/zooming. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Scope and scene composition include 3 perch/depth planes. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | no | No bird-chosen placement rule. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Lighting derives palette shifts from local time. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | borderline; Palette shifts are present; warmer/evening/calls-quieter detail is absent. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | No night-state behavior. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Ambient weather is in scope and mood input. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | borderline; Weather affects mood; specific rain/vocal-frequency effect is absent. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Ambient leaves/feathers particle system is planned. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Parallax is minimal. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | borderline; Top bar exists and fades; no explicit no-chrome-inside rule. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | no | Top bar behavior exists, but contents are not specified. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Top bar fades on idle and reappears on movement/focus. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | First frame renders birds mid-motion from snapshot. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | No spinners; delayed snapshot shows a quiet empty field. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-aviary state. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | Palette shifts are planned, but calm/naturalist/accent-color spec is absent. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | no | No responsive no-crop rule. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Magic-link auth endpoints are present. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | Expiring tokens are named, but no 15-minute expiry. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Scope says single-user accounts. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | account_id is a synthetic UUID primary key across systems. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Background tick runs about once per minute across active aviaries. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Snapshot is polled on visibility change, render gaps, and keepalive. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Client interpolates changed movement rather than snapping. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Cross-device sync and single canonical server state are planned. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Clients never send absolute state; additive deltas prevent LWW conflicts. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Server tick drains events and computes additive deltas. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | no | No sync-error surface or tone rule. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | no | Session tokens exist, but not per-device or revocable from settings. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | no | Email mentions export, but no export endpoint or JSON snapshot contents. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | no | No deletion flow. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | borderline; No per-bird/per-account interaction telemetry is stated, though ML training is not named. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Metrics are aggregate operational metrics; no per-bird/account telemetry. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | no | No email-change flow. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Read-only visits via one-time email invite and invite/revoke APIs. |
| 90 | Visits default OFF for new accounts | social_optional.md | no | Default-off state is not specified. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Read-only visits and no co-presence are planned. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | borderline; Read-only/no co-presence implies no visitor-triggered interactions; specific list absent. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Social networks with chat, comments, co-presence, avatars are out of scope. |
| 94 | No "your friend visited!" notification by default | social_optional.md | no | No notification-default rule. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Revoke API is planned. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | borderline; Only a code-splitting mention of visit logs supports this; exact host-visible behavior is thin. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | no | No actual-aviary/no-show-off rule. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Leaderboards and public discovery are excluded. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Live-region naturalist prose describes the scene. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Narration is periodic and rate-limited to prevent queue flooding. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Generated naturalist prose is specified. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Reduced motion swaps loops with slow cross-fades. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | borderline; Cross-fade rendering preserves a designed surface, but the charm rationale is implicit. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | no | Captions exist, but no toggle and no mood-describing caption behavior. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | WCAG AA contrast is enforced for UI chrome and overlays. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Keyboard nav through top bar and birds with Enter/Esc is planned. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | High-contrast focus ring is planned. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Initial gzipped JS bundle budget is <2MB. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | TTFB target is <500ms on mid-tier mobile 4G. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | 60fps idle motion on 5-year-old hardware is a budget. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Flat memory over 30 minutes is planned. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | WebAudio synthesis and no recorded loops are planned. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Fallback is silence plus textual call captions. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Synthetic tests, aggregate RUM, and aggregate metrics are planned. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Simulation tick latency p99 >5s alarm is planned. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Web client supports latest 2 versions of major browsers. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Native apps are out of scope. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Gamification with streaks, levels, points, leaderboards is out of scope. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Tamagotchi mechanics are out of scope. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Social networks/public discovery/chat/comments/co-presence/avatars are out of scope. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **60.7%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | B identified? | PLAN cross-cutting? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md:3: "aviary continues without the viewer"; RECONSTRUCTION.md:9: "procedural, stateful, and expressive" | PLAN.md:23: "aviary continues without the viewer"; PLAN.md:67: "No spinners"; PLAN.md:74-76 procedural audio | yes | yes | no | partial | Continuing state and procedural aliveness survive; the downstream staleness/leakage consequence is not recovered. |
| S2 - notice-never-announce | 4 | included | none | none | no | no | yes | none | The plan has some anti-gamification/social refusals, but not the notice-vs-announce principle. |
| S3 - charm-from-specificity | 2 | included | none | PLAN.md:59: "specific, lowercase, present-tense observations"; PLAN.md:37 user-assigned name; PLAN.md:98 age-based offers | no | yes | yes | partial | Specific naturalist surfaces survive in the plan, but B does not reconstruct specificity as the charm engine. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md:6 single horizontal scene; PLAN.md:7 2 starters/max 7; PLAN.md:20 no panning/zooming/customizing; PLAN.md:71 fading top bar | no | yes | yes | partial | Restraint rules are planned, but the why about recognizability and place-not-app richness is not reconstructed. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md:11: "product voice is quiet, observational, and naturalist" | PLAN.md:44 field notebook naturalist voice; PLAN.md:59 specific lowercase observations; PLAN.md:81 naturalist prose narration | yes | no | yes | partial | Naturalist product voice survives, but matter-of-fact account/error/system exception is missing. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md:5: "Care is non-punitive" and "Neglect results in zero drift" | PLAN.md:56 presence conjunction; PLAN.md:57 presence-time drives drift and neglect is zero; PLAN.md:17-18 no gamification/Tamagotchi | no | yes | yes | partial | The plan preserves presence/drift/no-punishment, but B reconstructs it mainly as non-punitive care, not idle attention as interaction. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md:3 continuous tick and source of truth; RECONSTRUCTION.md:7 clients send actions and additive deltas prevent LWW conflicts | PLAN.md:26 tick regardless of connectivity; PLAN.md:62 server source of truth; PLAN.md:63 clients send actions, not absolute state | yes | yes | no | full | Server-side canonical simulation and sync-conflict rationale are recovered cleanly. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md:17: "strict privacy boundary" and forbids "per-bird or per-account interaction telemetry" | PLAN.md:93-94: no per-bird/per-account telemetry; aggregate operational metrics only | yes | yes | no | full | Aggregate-only measurement and the no per-bird/account boundary are recovered. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md:13: "Accessibility is a core surface, not an afterthought"; narration is "a core writing task" | PLAN.md:13 v1 accessibility scope; PLAN.md:70 reduced motion cross-fades; PLAN.md:81 naturalist SR prose; PLAN.md:105 narration core writing task | yes | yes | no | full | B explicitly treats accessibility as a core surface with designed narration, reduced motion, captions, keyboard, and contrast in v1. |

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | no |
| S2 | no | no | no |
| S6 | no | no | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: PLAN.md:23 architecture enforces continuing-without-viewer; PLAN.md:26 tick regardless of connectivity; PLAN.md:67 no spinners/mid-motion first frame; PLAN.md:74-76 procedural audio.
- S2: PLAN.md:17-19 excludes gamification and some social attention surfaces, but there are fewer than 3 explicit notice-never-announce inheritances; (b) no.
- S3: PLAN.md:37 user-assigned names; PLAN.md:44/59 notebook naturalist specificity; PLAN.md:81 screen-reader naturalist prose; PLAN.md:98 age-based offers. Cross-cutting in plan, but not reconstructed as the charm engine.
- S4: PLAN.md:6 single scene; PLAN.md:7 two starters/max seven; PLAN.md:20 no panning/zooming/customizing; PLAN.md:71 fading top bar.
- S5: PLAN.md:44/59/81 naturalist prose surfaces; no matter-of-fact account/error exception, so (b) no.
- S6: PLAN.md:56 precise presence conjunction; PLAN.md:57 drift input and zero neglect drift; PLAN.md:17-18 no gamification/Tamagotchi. Cross-cutting yes.
- S7: PLAN.md:26 tick regardless of client; PLAN.md:55 drains event log and writes canonical state; PLAN.md:62-63 server source of truth/action-only deltas; PLAN.md:48 snapshot pull.
- S8: PLAN.md:31-32 synthetic account ID/encrypted email; PLAN.md:93 no per-bird/account telemetry; PLAN.md:94 aggregate operational metrics.
- S9: PLAN.md:13 v1 accessibility scope; PLAN.md:70 reduced-motion cross-fades; PLAN.md:78 captions fallback; PLAN.md:81 naturalist live-region prose; PLAN.md:83-84 keyboard and contrast; PLAN.md:105 core writing task.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **27.5%**.
Reachable feature-level whys: **31 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | none | none | yes | none | Rule appears, but no drift-inflation/load-bearing rationale is reconstructed. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md:76: low-pass filter and non-Tamagotchi intent; RECONSTRUCTION.md:127: too fast Tamagotchi, too slow broken | PLAN.md:57 low-pass drift from presence/listen/offers; PLAN.md:102 too fast Tamagotchi, too slow broken | no | partial | Slow filter and failure-mode band survive; one-week-measurable/three-week-visible calibration is absent. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md:5 and 76: neglect is zero drift, not negative; non-Tamagotchi intent | PLAN.md:57: traits drift toward expressiveness; neglect is zero drift; PLAN.md:18 Tamagotchi mechanics excluded | no | partial | Monotonic/no-punishment survives, but the two-week-return/mistrust consequence is absent. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md:27: no recorded loops, runtime variation, avoids phase-cancellation | PLAN.md:74-76: WebAudio, no loops, variation, independent nodes without phase-cancellation | no | partial | Phase-cancellation rationale is recovered; looped-audio-as-dead-software and audio-as-affective-spine are not. |
| F5 | mood-shaped-idle-motion | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured. |
| F6 | bird-count-cap-7 | 2 | yes | included | none | none | yes | none | Max 7 survives as a rule; recognizability/chorus-blur rationale is absent. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md:58: personality vector is used by drift/mood/audio and "Persisted server-side only" | PLAN.md:39: personality_vector persisted server-side only; PLAN.md:55 tick computes additive deltas | no | partial | Canonical server-side persistence survives; deleting-the-known-bird and downstream LWW rationale are absent. |
| F8 | vector-never-shown-numerically | 2 | yes | included | none | none | yes | none | Server-only vector rule appears, but the stat-management relationship rationale does not. |
| F9 | return-greeting | 4 | yes | included | none | none | yes | none | Frozen reconstruction says Notice/Return-greeting is NOT RECOVERABLE FROM PLAN. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured. |
| F11 | settle-is-opt-in | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md:34 and 78: notable state changes and "specific, lowercase, present-tense observations" in naturalist voice | PLAN.md:44 naturalist voice; PLAN.md:59 occasionally triggered on notable changes with specific lowercase present-tense observations | no | partial | Naturalist prose survives; stock-event-log danger and notebook-not-feed/read-only rationale are absent. |
| F13 | presence-accounting | 4 | yes | included | none | none | yes | none | Three-signal rule survives, but no rationale about approximating watching or avoiding silent drift corruption. |
| F14 | no-streak-counter | 4 | yes | included | none | none | yes | none | Streaks are excluded, but the presence-for-birds-not-counter rationale is absent. |
| F15 | scene-loads-with-motion | 4 | yes | included | none | none | yes | none | Despite the plan rule, frozen reconstruction marks the initial-load feature NOT RECOVERABLE FROM PLAN. |
| F16 | synthetic-account-id | 4 | yes | included | none | none | yes | none | Synthetic UUID rule survives; PII leak and retrofit rationale are absent. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md:47: tick runs across active aviaries even when disconnected; RECONSTRUCTION.md:29 server truth plus additive events prevent sync conflicts | PLAN.md:26 tick regardless of connectivity; PLAN.md:55 tick drains log and writes state; PLAN.md:63 additive action model | no | partial | Server tick and sync coherence survive; client-side divergent-sim failure mode is compressed away. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md:83: clients send actions, not absolute state, to prevent LWW conflicts | PLAN.md:63: clients never send absolute state; server computes changes; prevents LWW conflicts | no | full | Additive server-authored action model and LWW prevention are recovered. |
| F19 | sync-conflict-tone | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md:17: forbids per-bird/per-account interaction telemetry; metrics are aggregate or operational | PLAN.md:93-94: strict privacy boundary and aggregate operational metrics | no | partial | No per-bird/account telemetry and aggregate boundary survive; private-relationship-as-data-product rationale is absent. |
| F21 | visit-read-only-ambient | 2 | yes | included | none | none | yes | none | Read-only visit rule survives; co-presence-as-larger-product/drift-risk why is absent. |
| F22 | no-friend-visited-notification | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | No leaderboards/discovery/public surfaces survive as rules; comparison-product rationale is absent. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md:36 and 104: generated naturalist prose describing the scene, rate-limited | PLAN.md:81: live-region receives generated naturalist prose and is rate-limited | no | partial | Running naturalist prose survives; same-right-to-feel and wrong-ARIA-automation rationale are absent. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md:37: swaps animation loops with slow cross-fades between static poses | PLAN.md:70: reduced-motion swaps animation loops with slow cross-fades | no | partial | Cross-fade rendering survives; full-quality alternate-surface/charm-preservation rationale is absent. |
| F26 | ttfb-500ms | 2 | yes | included | none | none | yes | none | 500ms target survives; affective-performance threshold why is absent. |
| F27 | no-gamification-non-goal | 4 | yes | included | none | none | yes | none | Gamification is excluded, but the counter/engagement foothold rationale is not recovered. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md:5: "Care is non-punitive" and excludes hunger/death/negative drift; RECONSTRUCTION.md:76 non-Tamagotchi intent | PLAN.md:18 excludes hunger, death, negative drift; PLAN.md:57 neglect zero, not negative | no | full | Non-punitive/no-negative-drift refusal of Tamagotchi mechanics is recovered. |
| F29 | starter-birds-not-catalog | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured. |
| F30 | age-based-bird-offers | 4 | yes | included | none | none | yes | none | Age-based ramping is in the plan, but frozen reconstruction marks the ramping why NOT RECOVERABLE FROM PLAN. |
| F31 | stable-bird-identity | 4 | yes | included | none | none | yes | none | Stable UUID survives; identity-as-relationship-continuity rationale is absent. |
| F32 | mood-persists-across-sessions | 2 | yes | included | none | none | yes | none | current_mood persistence is implied, but no no-neutral-reset/continuity why is recovered. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | none | none | yes | none | Plan has a read-only endpoint, but frozen reconstruction says notebook entries endpoint is NOT RECOVERABLE FROM PLAN. |
| F34 | account-export-relationship-copy | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured. |
| F35 | account-deletion-grace-then-hard-delete | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md:115-116: avoids per-bird/per-account interaction telemetry; operational metrics monitor without per-account telemetry | PLAN.md:93-94: strict privacy boundary; aggregate operational metrics | no | full | Technical aggregate-only telemetry boundary is recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | none | none | yes | none | One-time email invite/revoke survives; private-relationship/not-publishing rationale is absent. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | none | none | yes | none | Visit log capture was borderline; no on-demand-transparency rationale is recovered. |
| F39 | visitor-sees-actual-aviary | 2 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md:36 and 104: narration is periodic/rate-limited to prevent queue flooding | PLAN.md:81: periodically receives prose and is rate-limited to prevent queue flooding | no | partial | Queue-flooding rationale survives; shared slow visual rhythm and sparse observational event priority are absent. |

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | no | no | no |
| F2 | yes | no | yes |
| F3 | yes | yes | no |
| F4 | no | yes | no |
| F7 | yes | no | no |
| F9 | no | no | no |
| F10 | unreachable | unreachable | unreachable |
| F12 | yes | no | no |
| F13 | no | no | no |
| F14 | no | no | no |
| F15 | no | no | no |
| F16 | no | no | no |
| F17 | yes | yes | no |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | no | no |
| F25 | yes | no | no |
| F27 | no | no | no |
| F30 | no | no | no |
| F31 | no | no | no |
| F35 | unreachable | unreachable | unreachable |
| F40 | no | yes | no |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS.json |
| Possible total weight | 152 | Full-instance weight |
| Reachable gold whys | 40 | 9 system + reachable feature whys |
| Excluded unreachable feature whys | 9 | Denominator exclusions |
| Recovered / reachable weight | 45.0 / 130.0 | Weighted recovery numerator/denominator |
| Whys with reconstruction evidence | 19 | Exact rationale evidence present in frozen reconstruction |
| Whys with PLAN grounding | 21 | Exact plan grounding for recovered rationale |
| `rule_without_why` cases | 23 | Mechanism or rule survived without gold why |
| `plan_only_not_reconstructed` cases | 0 | No rationale-only cases beyond rule-only capture |
| `ungrounded_reconstruction` cases | 0 | No clear ungrounded rationale claims found |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 48.0 | 22.0 | 45.8% |
| Affective whys | 82.0 | 23.0 | 28.0% |
| Weight 2 whys | 30.0 | 9.0 | 30.0% |
| Weight 3 whys | 100.0 | 36.0 | 36.0% |
| System-level whys | 28.0 | 17.0 | 60.7% |
| Feature-level whys (reachable) | 102.0 | 28.0 | 27.5% |

## 3. Diagnostic patterns

- Affective vs functional: functional whys recovered better where they were architectural (S7, S8, F18, F36). Affective whys often became product rules without the relationship-protecting rationale (F9, F14, F15, F27, F30, F31, F33, F37, F38).
- Weight-3 vs weight-2: high-weight whys often retained one primary mechanism but lost secondary contributors and downstream consequences. The result is many partials rather than full recoveries.
- System-level vs feature-level: the plan preserved philosophy more often than specific feature whys. System-level fidelity is 60.7%, while feature-level fidelity is 27.5%.
- Multi-layer recovery: Layer 1 survived most often; Layer 2 and Layer 3 were the common leak points. F2, F17, F24, F25, and F40 are representative.
- Subdomain patterns: accounts/sync and telemetry were strongest; social, return-greeting, field-notebook constraints, and announcement/refusal details were weakest.
- Evidence-bound effects: F15 and F30 show how a captured rule can still score none when the frozen reconstruction says NOT RECOVERABLE FROM PLAN. Many no-gamification/no-social rules count for capture but not why recovery.

## 4. Recommendations for v2 hardening

- Keep targeted feature-level exceptions and sharpenings. They expose mechanism-only planning that the system-level score can hide.
- Add a visible mechanism-only diagnostic tier. This run had many rule_without_why cases, and that failure mode is operationally important.
- Include canonical cross-cutting examples for each system why. S3-S6 required subjective judgment about whether a related implementation was true inheritance.
- Preserve multi-layer whys, but label the layers in the data schema so loss of primary cause, secondary contributor, and downstream consequence can be aggregated directly.

## 5. Methodology caveats

- Fresh-context fidelity: the frozen reconstruction reads plan-derived, uses no gold IDs, and contains many honest NOT RECOVERABLE FROM PLAN entries. I did not edit it.
- Single-run limitation: this is one run, with no variance signal by itself.
- Borderline capture calls: I leaned inclusive on mood persistence, vector hiddenness, visit log, reduced-motion charm, and similar thin but plausible builder-facing hooks. These calls move planning quality modestly, not the main fidelity finding.
- System-level cross-cutting: S3-S6 were the subjective zone. The appendix above records the specific plan inheritances used for each call.
- Confabulation cases: no clear ungrounded reconstruction rationale was found; the main issue was under-recovery and rule-only recovery.
- Evidence-bound denials: feature-level credit was withheld whenever the frozen reconstruction lacked rationale evidence, even if PLAN contained the operational rule.
- Rule-without-why cases: mechanism-only preservation was frequent, especially around presence, no streaks/gamification, social sharing, and account/sync privacy details.

End of report.
