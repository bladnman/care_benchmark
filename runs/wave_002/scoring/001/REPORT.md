# REPORT - CARE run 001

> Phase 2B scoring report for Pocket Aviary v1, variant v06 evidence-bound clean + targeted gold headroom.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **90.8%** |
| Intent fidelity | **52.0%** |
| Combined quality | **9035** |

**Diagnostic split:**

- System-level fidelity: **75.0%**
- Feature-level fidelity: **46.7%**

**(Planning, fidelity) coordinate:** `(90.8, 52.0)`.

### Run metadata

| Field | Value |
|---|---|
| Run number | 1 |
| Run label |  |
| Timestamp | 2026-05-30T12:25:09Z |
| Candidate model | gpt-5.4 |
| Candidate effort | extra-high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **109 / 120** = **90.8%**.

By PRD file:

| File | Total | Captured | Rate |
|---|---:|---:|---:|
| product_brief.md | 6 | 6 | 100.0% |
| concepts.md | 4 | 4 | 100.0% |
| bird_engine.md | 22 | 21 | 95.5% |
| interactions.md | 20 | 18 | 90.0% |
| aviary_layout.md | 18 | 13 | 72.2% |
| accounts_sync.md | 18 | 16 | 88.9% |
| social_optional.md | 10 | 10 | 100.0% |
| accessibility_perf.md | 18 | 17 | 94.4% |
| non_goals.md | 4 | 4 | 100.0% |
| **Total** | **120** | **109** | **90.8%** |

Per-feature detail:

| Feature ID | Feature title | File | Captured | Note |
|---:|---|---|---|---|
| 1 | Headline product concept statement | product_brief.md | yes | Product concept is implicit across the plan: one web aviary, birds, interactions, accounts, visits, and launch scope. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured through continuity, alive-feeling motion, procedural calls, quiet loading, and first-bird performance. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Borderline: the exact phrase is absent, but no gamification, quiet visits, no default visit notification, and sparse notebook behavior carry much of it. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Naturalist observation surfaces and matter-of-fact system surfaces are explicit. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Explicit out-of-scope list covers game, Tamagotchi, social-network, and public-surface framing. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Borderline: bird-count restraint is explicit; palette/no-panning restraint is thinner and appears only through layout and performance constraints. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Borderline: no glossary section, but the implementation plan defines the domain terms through data model and engine sections. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Three-signal conjunction and drift use are explicit. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Separate permanent vector, runtime mood, and temporary reservoir are explicit. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Settle appears as top-bar affordance, state, and event. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | All five vector dimensions are listed. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Slow, capped, saturating additive drift with one-week and three-week calibration is explicit. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Calibration thresholds and harness are explicit. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Traits never decrement due to absence; reservoir handles temporary quietness. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Borderline: mood is a fast runtime state with time-of-day transitions, though daily-ish reset language is not explicit. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Mood inputs include time of day, weather, interaction, bird contagion, and personality. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Borderline: motif libraries and WebAudio synthesis are explicit; the plan does not dwell on the no-recorded-loops rationale. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Persistent call seed and motif profile preserve recognizable contours. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Borderline: chorus opportunities and procedural scheduling are present, but phase/loop failure is not named. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Scheduling uses current mood and vocal-frequency trait. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Scene renderer synthesizes micro-motion and idle behavior from render packets. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Mood, pose target, and idle behavior weights are linked in rendering/runtime state. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | no | Species assets exist, but no v1 species count is planned. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Borderline: display_name is modeled, but rename/adoption naming flow is thin. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Two starter birds at account creation are explicit. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Cap and unlock thresholds to seven are explicit. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Age-based unlock support and day-threshold rollout are explicit. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Server-owned canonical personality state and persisted vector table are explicit. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Mood does not reset on tab open and session end preserves mood. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Bird-to-bird contagion and chorus opportunities are planned. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Borderline: stable bird_id is present, but migration/renaming identity rationale is not elaborated. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Trait numbers are banned from notebook, narration, labels, and regression suites. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Bootstrap greeting plan is explicit. |
| 34 | Greeting variation by absence length | interactions.md | yes | Greeting selection uses true absence length. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Candidate birds are weighted by boldness and social warmth. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | yes | Primary greeter plus optional stagger suppresses simultaneous greetings. |
| 37 | No "Welcome back!" toast or banner | interactions.md | no | No explicit prohibition on textual welcome toasts or banners appears. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Listen-in mix raises the focused bird. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Other birds reduce to ambient and never fully mute. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Offer event types are authored and explicit. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Offer reactions use mood and curiosity. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Offer cooldowns are modeled per bird. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Borderline: settle as session end is present; the evening lighting detail is not explicit. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Borderline: absence is not penalized, but close-tab equivalence is not explicitly stated. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Sparse deterministic naturalist notebook generation is explicit. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Average every few days; no entry per session. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | No mutation endpoint; read-only UI. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Server-credited presence windows from valid pings are explicit. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Visible, focused, recent pointer/keyboard activity conjunction is explicit. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Streaks, counters, and visit-count behavior are explicitly banned. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Borderline: server continuation and visibility refresh are explicit; client render pause is mostly implied. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | no | No click-anywhere undo or five-second window is planned. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | no | The plan keeps birds on-screen but does not specify a single horizontal no-panning scene. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Front/middle/back zones are modeled. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Perch preference and pose are runtime choices; no placement UI exists. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Account timezone, day phase, and time-of-day inputs are present. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Borderline: day/night palettes and slowed color shifts are present; call-quieting is not explicit. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | no | No nightjar-like active bird or specific night-state composition appears. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Sparse weather scheduler and rare ambient weather are explicit. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Weather perturbs mood and ambience. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Leaf/feather drift is local-only ambience and removed in reduced motion. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | no | Foreground/background layers exist, but parallax behavior is not specified. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Top bar contains account/settings/accessibility/notebook/offer/settle. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Top bar contents are explicit. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Top bar fades and reappears on input. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Initial bootstrap snapshot and immediate rendering are planned. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Quiet field and never-spinner rule are explicit. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | no | No empty-aviary state is planned. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | no | Some palette states exist, but no calm naturalist color spec or saturation ban appears. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Responsive no-crop rule is explicit. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Magic-link sign-in endpoints are explicit. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | no | Magic links expire, but no 15-minute duration is specified. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | One canonical aviary per account is explicit. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Synthetic UUID account IDs are explicit. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Minute-level server-side tick is explicit. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Visibility-triggered snapshot refresh is explicit. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Client interpolation and snapshot rendering are explicit. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | One server-owned canonical state is explicit. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | No client personality writes and no stale overwrites are explicit. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Server tick is sole writer; clients write events only. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Matter-of-fact account/error/revocation surfaces are explicit. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Sessions and revocation are explicit. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Export job and JSON export flow are explicit. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Soft delete, 30-day recovery, hard delete are explicit. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Per-bird analytics and model-training inputs are explicitly disallowed. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Aggregate-only metrics and DB separation are explicit. |
| 87 | Privacy policy link in account settings | accounts_sync.md | no | No privacy-policy link is specified. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Verified email replacement flow is explicit. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Per-invite email invitation flow is explicit. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Visits default off in beta and quiet social scope. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Visitor bootstrap and snapshot are read-only; no write route. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Visitor has no event-write route. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Chat, comments, avatars, public social surfaces are out of scope. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Optional host visit notifications are off by default. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Invite revocation endpoint and tests are explicit. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Visit log table and settings access are explicit. |
| 97 | Visitor sees host aviary as it is (no special show-off mode) | social_optional.md | yes | Borderline: read-only host snapshot implies actual aviary; no show-off-mode ban is explicit. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Leaderboards, discovery, public aviaries are explicitly out of scope. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Slow naturalist narration composer is explicit. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Idle narration every 30-60 seconds and live-region queues are explicit. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Narration uses naturalist prose and avoids raw state labels. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Cross-fade reduced-motion path is explicit. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Dedicated reduced-motion system and not-fallback risk are explicit. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Borderline: captions are implemented and default in fallback, but a user toggle is not explicit. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | WCAG AA contrast and caption/focus contrast are explicit. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Keyboard paths through surfaces and bird focus are explicit. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Focus rings with contrast across palettes are explicit. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | 2MB gzipped bundle budget is explicit. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | 500ms first-bird target is explicit. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | 60fps idle target is explicit. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | 30-minute memory soak test and budget are explicit. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | WebAudio/AudioWorklet synthesis with no long buffers is explicit. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Graceful visual aviary plus default captions is explicit. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Aggregate runtime instrumentation is explicit. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Tick p99 budget and alert are explicit. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | no | Modern browsers and unsupported-browser rate are mentioned; no last-two-majors matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Native mobile clients are explicitly out of scope. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Gamification list is explicitly out of scope. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Tamagotchi mechanics are explicitly out of scope. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Profiles, discovery feeds, chat, comments, and public social surfaces are out of scope. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **75.0%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | (a) Identified by B? | (b) Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | none | PLAN.md planning frame and frontend/audio sections: "aviary should feel like it was already running"; "alive-feeling micro-motion"; "Never show a spinner." | no | yes | yes | partial | Plan carries aliveness cross-cuttingly, but reconstruction did not elevate it as a system principle. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md system intent: "Ambient, non-gamified product philosophy" and "optional visit notifications off by default." | PLAN.md scope/social/notebook: "Gamification of any kind" out of scope; "visit notifications enabled" default off; "never mention ... visit-count behavior." | yes | yes | no | partial | Recovered the anti-announcement cluster but not the processed-vs-seen rationale or no-welcome-toast application. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md system intent: notebook, narration, and captions should "feel authored" with "consistent naturalist voice." | PLAN.md text generation/notebook sections: deterministic prose from structured state; "prose is specific and observational"; banned generic/gamified phrasing. | yes | yes | no | full | Specific naturalist prose and avoidance of generic state language survived. |
| S4 - restraint-over-richness | 2 | included | none | PLAN.md scope/frontend/audio: "Two starter birds"; "up to seven total birds"; no UI chrome beyond top bar; recognizability limits at higher counts. | no | yes | yes | partial | Plan has restraint mechanisms, but reconstruction did not identify depth-over-variety as a why. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md system intent: "Matter-of-fact system copy separated from naturalist voice." | PLAN.md accessibility/system copy: "auth, error, revocation, unsupported-browser, and settings-system copy must use the matter-of-fact voice." | yes | yes | no | full | The voice split is clearly identified and cross-cutting, though F19 loses the sharper evasiveness rationale. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md system intent: "Honest presence and integrity of drift"; prevents "retroactively claiming hours of idle attention." | PLAN.md presence sections: visible, focused, recent pointer/key conjunction; server credits bounded windows; stale pings are dropped. | yes | yes | no | partial | Presence honesty and inflation risk survived; settle-vs-close equivalence was not explicit enough. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md system intent: "server is the sole writer of personality and mood state" and protects against "stale clients, peer sync, replay." | PLAN.md architecture/sync: server-side tick runs whether or not client is open; clients write events only; no last-write-wins. | yes | yes | no | full | Canonical server tick, sync coherence, and failure mode all survived. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md system intent: "no raw per-bird interaction history leaving the simulation boundary." | PLAN.md telemetry/privacy: no per-bird interaction history in analytics; simulation DB separated from aggregate analytics pipeline. | yes | yes | no | full | Privacy boundary and data-pipeline enforcement survived. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md system intent: "Accessibility surfaces from day one" and "not ... fallback instead of designed surface." | PLAN.md scope/accessibility: day-one narration, reduced motion, captions, keyboard, focus, contrast; acceptance criteria equal core features. | yes | yes | no | full | Designed accessible surfaces, equal acceptance criteria, and day-one shipping all survived. |

Multi-layer system-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | no | no | no |
| S2 | yes | no | no |
| S6 | yes | yes | no |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix:**

- S1: Initial 500ms/quiet-field behavior, alive-feeling micro-motion, procedural call synthesis, reduced-motion charm, and first-frame continuity. Count >= 3; (b) yes.
- S2: No gamification, no streak/counter/visit-count copy, visit notifications off by default, sparse notebook, quiet top bar behavior. Count >= 3; (b) yes.
- S3: Naturalist notebook, narration, captions, deterministic phrase tables, bird naming/display names, no public comparison surfaces. Count >= 3; (b) yes.
- S4: Two starters, cap seven, no UI chrome inside aviary/top bar, listen-in as mix not solo UI, unlock thresholds configurable for recognizability. Count >= 3; (b) yes.
- S5: Notebook/narration/captions in naturalist voice; auth/error/revocation/settings/accessibility system copy matter-of-fact. Count >= 3; (b) yes.
- S6: Three-signal presence pings, bounded server-credited windows, drift inputs, stale ping drops, no negative drift on absence. Count >= 3; (b) yes, but settle/close equivalence is thin.
- S7: Server-side tick, client snapshots only, event log, no client personality writes, no LWW, visitor read-only access. Count >= 3; (b) yes.
- S8: No per-bird ML telemetry, aggregate-only metrics, simulation DB separated from analytics, export/delete lifecycle, no identity-keyed observability. Count >= 3; (b) yes.
- S9: Day-one narration, reduced motion, captions, keyboard navigation, focus treatment, contrast, usability sessions. Count >= 3; (b) yes.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **46.7%**.

Reachable feature-level whys: **39 / 40**; F10 was excluded because the no-welcome-toast anchor was not captured.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md presence entries: "keeps presence honest" and prevents "retroactively claiming hours of idle attention." | PLAN.md presence accounting: visible, focused, recent pointer/key pings; server credits bounded windows. | no | partial | Precision and drift-corruption risk survived; individual-signal failure cases were compressed away. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md drift function: "one week" measurable and "three weeks" felt; per-session effect capped. | PLAN.md drift function: per-session cap; one-week measurable, three-week felt; calibration harness. | no | partial | Slow calibration survived; Tamagotchi-vs-screensaver failure-band rationale did not. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md system intent: "Monotonic long-term care without punishment for absence" and birds can be "quieter after absence." | PLAN.md product behavior: separate permanent vector from reservoir; traits never decrement due to absence. | no | full | Permanent positive drift plus ambient quietness after absence was recovered. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md audio pipeline: WebAudio/AudioWorklet for "per-bird procedural calls" and "smooth playback without stored long audio buffers." | PLAN.md audio pipeline: motif libraries, procedural synthesis, persistent seeds, WebAudio fallback without recorded audio. | no | partial | Procedural rule survived; looped-audio spellbreak and chorus artifact rationales did not. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none | none | yes | none | Mood and motion rules appear, but the why about users reading mood without labels is absent. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md scope: slow or pause unlocks if "recognizability, performance, or notebook quality degrade at higher bird counts." | PLAN.md rollout/risks: thresholds configurable if recognizability, performance, or notebook quality degrade. | no | full | The recognizability ceiling rationale survived. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md data/sync: "permanent, monotonic, slow-moving trait values" and server as "sole writer." | PLAN.md data/sync: persisted personality vectors; server-owned canonical personality; no client personality writes. | no | partial | Persistence and canonical-sync implications survived; losing-the-bird affective rationale did not. |
| F8 | vector-never-shown-numerically | 2 | yes | included | none | none | yes | none | Trait-number bans survived as rules, but stat-management relationship rationale did not. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md greeting selection: incorporate "true absence length," choose one primary greeter, suppress simultaneous greetings. | PLAN.md greeting selection: weighted by absence, boldness, mood, reservoir; primary greeter plus staggered secondary. | no | partial | Greeting mechanics and variation survived; notice-never-announce payoff was not recovered. |
| F10 | no-welcome-back-toast | 4 | no | unreachable_excluded | none | none | no | unreachable | Anchor feature was not captured in PLAN. |
| F11 | settle-is-opt-in | 2 | yes | included | none | none | yes | none | Settle exists, but optionality/close-tab equivalence rationale is absent. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md notebook: "sparse and observation-like," "specific observational prose," no trait numbers or visit-count behavior. | PLAN.md notebook generation: every few days, no entry per session, specific observational prose, no trait numbers or visit-count behavior. | no | partial | Voice and rarity survived; notebook-as-most-concentrated-voice rationale was not explicit. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md presence: visible, focused, recently active; avoids idle/hidden tabs; keeps drift honest. | PLAN.md presence accounting: pings only while visible, focused, and recently active; bounded server windows. | no | partial | Conjunction and drift-integrity reason survived; individual-signal misses were not recovered. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md system intent: excludes "streaks, counters" and notebook must never mention "visit-count behavior." | PLAN.md out-of-scope/notebook: no gamification; no streaks/counters; no visit-count behavior in notebook. | no | partial | The refusal survived, but managing-a-number and disguised-visit-log rationales were mostly lost. |
| F15 | scene-loads-with-motion | 4 | yes | included | none | none | yes | none | Initial rendering and quiet field rules survived, but the continuing-aviary/no-machine rationale was not reconstructed. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md security: synthetic UUIDs support privacy by "avoiding direct identity exposure." | PLAN.md identity/security: UUID account_id, encrypted email, hashes for lookup, no identity-keyed observability. | no | partial | PII/privacy concern survived; non-retrofit/source-cutoff rationale did not. |
| F17 | server-side-sim-tick | 4 | yes | included | RECONSTRUCTION.md server tick: advances mood/personality "whether or not the client is open" and prevents client ownership. | PLAN.md simulation/sync: minute tick, reads event log, writes canonical state, clients render snapshots. | no | full | Tick ownership, sync coherence, and client-simulation failure mode survived. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md sync risk: stale sessions overwriting newer state mitigated by no client personality writes, dedupe, server ordering, tick idempotency. | PLAN.md sync/idempotency: clients never write personality; event dedupe; order by server_received_at; reject stale sessions. | no | full | The no-LWW conflict rationale survived. |
| F19 | sync-conflict-tone | 2 | yes | included | none | none | yes | none | Matter-of-fact system copy survived, but evasiveness/user-clarity rationale did not. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md telemetry/privacy: avoid per-bird interaction history, per-account drift dashboards, and model-training inputs. | PLAN.md observability/privacy: per-bird interaction history and model-training inputs are disallowed; telemetry DB separated. | no | partial | Boundary and forbidden uses survived; relationship-as-private-data rationale was thinner. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md quiet sociality: visitor presence "does not enter the host simulation ledger" and no co-presence or host-state mutation. | PLAN.md visits: no event-write routes; visitor presence does not emit host events or enter ledger. | no | full | Observation-not-co-presence rationale survived. |
| F22 | no-friend-visited-notification | 2 | yes | included | none | none | yes | none | Default-off notifications survived as a rule; attention-driver loop rationale did not. |
| F23 | no-leaderboards | 2 | yes | included | none | none | yes | none | Leaderboards/discovery are excluded, but the comparison-surface relationship rationale is absent. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md narration: "slow naturalist prose updates" and avoids trait numbers, raw pose IDs, or debug state. | PLAN.md accessibility: running naturalist prose, slow live-region queue, user-triggered observations, no raw state labels. | no | full | Running prose and designed accessible-surface rationale survived. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md reduced motion: same state packet, swapped animator, no separate simulation path, not a fallback. | PLAN.md reduced motion: cross-fades, removed leaf drift, slowed color shifts, no separate simulation path. | no | partial | Different-rendering and not-broken-fallback survived; full calls/notebook/still-alive bundle was compressed. |
| F26 | ttfb-500ms | 2 | yes | included | none | none | yes | none | 500ms target survived as a budget, but affective performance-threshold rationale did not. |
| F27 | no-gamification-non-goal | 4 | yes | included | none | none | yes | none | The no-gamification rule is loud, but its counter/foothold rationale is not reconstructed. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md system intent: no hunger, decay, visible distress, negative drift; "without punishment for absence." | PLAN.md out-of-scope/product behavior: no Tamagotchi mechanics and traits never decrement due to absence. | no | full | Punishment-for-absence rationale survived. |
| F29 | starter-birds-not-catalog | 2 | yes | included | none | none | yes | none | Two starter birds survived, but arrivals-not-catalog rationale did not. |
| F30 | age-based-bird-offers | 4 | yes | included | none | none | yes | none | Age-based unlocks survived as a rule, but reward-loop/economy rationale did not. |
| F31 | stable-bird-identity | 4 | yes | included | none | none | yes | none | Stable IDs are present in the model, but remembered-relationship rationale was not reconstructed. |
| F32 | mood-persists-across-sessions | 2 | yes | included | none | none | yes | none | Mood persistence survived as a rule, but continued-while-away rationale did not. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md notebook endpoint: "no mutation endpoint," preserving notebook as "simulation-generated observation rather than user-authored or edited state." | PLAN.md notebook UI/API: read-only, paginated, no mutation endpoint, generated by simulation worker. | no | full | Observer-record-not-journal rationale survived. |
| F34 | account-export-relationship-copy | 2 | yes | included | none | none | yes | none | Export mechanics survived; relationship-copy right and quiet-quality rationale did not. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | none | none | yes | none | 30-day soft then hard delete survived, but regret/privacy/residue layers did not. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md privacy boundary: only approved aggregate counters/histograms reach analytics; no behavioral event payload. | PLAN.md privacy boundary: simulation DB isolated from analytics; only approved aggregate counters and latency histograms exported. | no | full | Technical telemetry boundary rationale survived. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md quiet sociality: "per-invite read-only visits" with host control, revocation, and no ambient social features. | PLAN.md visits: per-invite email invitations, no public discovery, revocation, visitors read-only. | no | full | Host-control/deliberate-sharing rationale survived. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md quiet sociality: visit log plus optional notifications off by default to preserve quiet sociality. | PLAN.md visits/rollout: visit log exists; host notifications off by default; no badge/push surface is planned. | no | full | On-demand transparency without attention loop survived. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | none | none | yes | none | Visitor snapshot mechanics survived, but no-show-off-mode rationale did not. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md narration: idle narration every 30-60 seconds; user-triggered observations get separate priority queues. | PLAN.md accessibility: slow naturalist prose, ambient and higher-priority live-region queues, prompt but observational user-triggered narration. | no | full | Slow rhythm, queue protection, and observational pacing survived. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | no | yes |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | no | no |
| F7 | yes | no | yes |
| F9 | yes | yes | no |
| F10 | no | no | no |
| F12 | yes | no | yes |
| F13 | yes | no | yes |
| F14 | yes | no | no |
| F15 | no | no | no |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | no | yes |
| F24 | yes | yes | yes |
| F25 | yes | no | yes |
| F27 | no | no | no |
| F30 | no | no | no |
| F31 | no | no | no |
| F35 | no | no | no |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From BENCHMARK_CONSTANTS.json |
| Possible total weight | 152 | Full-instance possible weight |
| Reachable gold whys | 48 | 9 system + 39 feature whys |
| Excluded unreachable feature whys | 1 | F10 only |
| Recovered / reachable weight | 77.0 / 148.0 | Sum of weight x recovery over included whys |
| Whys with reconstruction evidence | 30 | Rows with rationale evidence in frozen reconstruction |
| Whys with PLAN grounding | 32 | Rows with rationale grounding in PLAN |
| `rule_without_why` cases | 18 | Mechanism/rule survived without why |
| `plan_only_not_reconstructed` cases | 2 | S1 and S4 system principles |
| `ungrounded_reconstruction` cases | 0 | No clear cases counted |

### 2.5. Failure groupings

| Grouping | Total reachable | Recovered (weighted) | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54.0 | 32.0 | 59.3% |
| Affective whys | 94.0 | 45.0 | 47.9% |
| Weight-2 whys | 44.0 | 21.0 | 47.7% |
| Weight-3 whys | 104.0 | 56.0 | 53.8% |
| System-level whys | 28.0 | 21.0 | 75.0% |
| Feature-level whys (reachable) | 120.0 | 56.0 | 46.7% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** Functional whys recovered at 32.0/54.0 (59.3%); affective whys recovered at 45.0/94.0 (47.9%). Architecture, privacy, and sync survived better than relationship-framing product whys.
- **Weight-3 vs weight-2.** Weight-3 whys recovered at 56.0/104.0 (53.8%), slightly better than weight-2 whys at 21.0/44.0 (47.7%). High-weight functional whys like F17/F18/F40 survived; high-weight affective exceptions like F27/F30/F31/F35 did not.
- **System-level vs feature-level.** System-level fidelity was strong at 75.0%; feature-level fidelity was much lower at 46.7%. The planner preserved architecture and philosophy broadly, but many local whys became bare implementation rules.
- **Multi-layer recovery patterns.** Primary causes survived most often. Secondary temptation/failure-mode layers and downstream product-consequence layers were dropped, especially around no gamification, no toast, identity continuity, and deletion/export rationale.
- **Subdomain patterns.** Accounts/sync, privacy, and accessibility were strongest. Social and affective exclusions had many rule-only rows: F22, F23, F27, F29, F30, F31, F39.
- **Evidence-bound effects.** The v06 evidence gate denied plausible v1-style credit where the plan had a rule but no reconstructed why: F5, F8, F19, F22, F23, F26, F27, F29-F35, and F39.

The failure shape suggests the candidate is excellent at complete engineering coverage and robust system architecture, but it compresses affective intent into implementation guardrails. That makes the plan buildable, but some future maintainers would not know why tempting adjacent features must remain excluded.

## 4. Recommendations for v2 hardening

- Keep the evidence-bound operator. It cleanly distinguishes feature capture from intent recovery and exposed the rule-without-why pattern.
- Add more targeted exception whys around small UI temptations. The one unreachable gold why was F10, and multiple captured exclusions still lost their rationale.
- Preserve multi-layer whys, but consider reporting layer-level aggregate counts in score JSON in a future schema. Layer loss was the most informative diagnostic here.
- Keep the system-level cross-cutting bar, but require explicit documentation when only PLAN grounding exists. S1 and S4 were plan-preserved but reconstruction-missed.
- Consider one future instance with a less charm-heavy product to test whether affective why loss is product-specific or a general compression failure.

## 5. Methodology caveats

- **Fresh-context fidelity.** Scoring used only the allowlisted phase-two files, assigned plan/metadata/timing slot, and frozen reconstruction. The reconstruction was not modified.
- **Single-run-at-temperature limitation.** This is one run. No variance signal is available inside this score.
- **Borderline capture calls.** Inclusive calls affected planning quality: product-brief principles, glossary, bird naming, settle optionality, background-tab pause, some layout/palette/accessibility toggles, and visitor-actual-aviary were borderline.
- **System-level cross-cutting.** The (b) bar was strict but subjective. S1 and S4 were scored partial because the PLAN inherited them in 3+ places while the reconstruction missed them as principles.
- **Confabulation cases.** No clear ungrounded reconstruction cases were counted. Some generalized social/accessibility rationales were scored only when plan grounding existed.
- **Evidence-bound denials.** Several semantic near-misses were denied because the frozen reconstruction gave mechanism only, not rationale.
- **Rule-without-why cases.** 18 included whys were rule-without-why cases, concentrated in affective exclusions and relationship-data lifecycle features.

End of report.
