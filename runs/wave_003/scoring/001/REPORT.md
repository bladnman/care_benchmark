# REPORT - CARE run 001

> Phase 2B scoring report for Pocket Aviary v1. The frozen reconstruction was read-only; all per-why judgments below are evidence-bound against PLAN.md and RECONSTRUCTION.md.

---

## 1. Headline

| Score | Value |
|---|---|
| Planning quality | **100.0%** |
| Intent fidelity | **81.6%** |
| Combined quality | **9982** |

**Diagnostic split:**

- System-level fidelity: **92.9%**
- Feature-level fidelity: **79.0%**

**(Planning, fidelity) coordinate:** `(100.0, 81.6)`.

Planning is not low confidence.

### Run metadata

| Field | Value |
|---|---|
| Run number | 1 |
| Run label |  |
| Timestamp | 2026-09-30T01:52:41Z |
| Candidate model | gpt-6.1-sol |
| Candidate effort | high |
| Candidate harness | codex-cli |
| Evaluator model | gpt-5.5 |
| Evaluator effort | extra-high |
| Evaluator harness | codex-cli |

---

## 2. What survived, what didn't

### 2.1. Features captured (planning quality)

Captured: **120 / 120** = **100.0%**.

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
| 1 | Headline product concept statement | product_brief.md | yes | Product contract states one continuing aviary and a relationship-over-weeks release criterion. |
| 2 | "Feels alive, not robotic" design-philosophy section | product_brief.md | yes | Captured across already-living first paint, procedural calls, server tick, quiet field, and no spinners. |
| 3 | "Notice, never announce" principle callout | product_brief.md | yes | Captured by no return toast/banner/modal, no default visit notices, and no streak/visit-frequency surfaces. |
| 4 | Voice-and-tone guide for product surface (naturalist + matter-of-fact) | product_brief.md | yes | Captured by lowercase naturalist prose and direct system/error/account prose. |
| 5 | "What this is not" callout (game/Tamagotchi/social-network framing) | product_brief.md | yes | Captured in explicit exclusions and no obligation/absence-punishment language. |
| 6 | Restraint-over-richness scope statement (start with 2 birds, max 7) | product_brief.md | yes | Captured by starter pair, seven-bird cap, one screen, no panning, and restrained scene chrome. |
| 7 | Glossary of domain terms (bird, call, mood, etc.) | concepts.md | yes | Borderline inclusive: the plan defines/uses the domain model through mood states, settle, presence, calls, vectors, and projections rather than a separate glossary. |
| 8 | Definition of "presence" (idle attention as interaction) | concepts.md | yes | Captured exactly with visible + focus + recent pointer/key activity and dominant drift input. |
| 9 | Definition of personality vector vs mood (slow vs fast timescale) | concepts.md | yes | Captured by persisted traits and separate recent-attention/mood process. |
| 10 | Definition of "settle" as user-initiated session end | concepts.md | yes | Captured by settle event, preview, undo, and re-engagement semantics. |
| 11 | Personality vector (boldness, social warmth, vocal frequency, plumage saturation, curiosity) | bird_engine.md | yes | Captured by five persisted scalar traits and trait-specific drift mapping. |
| 12 | Personality drift function (low-pass filter) | bird_engine.md | yes | Captured by low-pass exposure, dose accounting, calibration, and exact integration. |
| 13 | Drift rate calibration (one week measurable, three weeks visible) | bird_engine.md | yes | Captured by day-7 instrument and week-3 perceptual gates. |
| 14 | Personality drift is monotonic toward expressive, never punishing | bird_engine.md | yes | Captured by nonnegative deltas, no trait decay, and absence harmlessness. |
| 15 | Mood state (fast-timescale, resets daily-ish) | bird_engine.md | yes | Captured by semi-Markov moods, daily-ish baseline, and no neutral reset. |
| 16 | Mood inputs (recent interactions, time of day, ambient events) | bird_engine.md | yes | Captured by mood inputs from interactions, local time, weather, and peer calls/moods. |
| 17 | Procedural call grammar (motifs combined at runtime) | bird_engine.md | yes | Captured by species grammar, server descriptors, and deterministic client expansion. |
| 18 | Per-bird call signature (recognizable by ear) | bird_engine.md | yes | Captured by stable seeds, sub-signatures, blind tests, and seven-bird recognizability gates. |
| 19 | Chorus mixing (real chorus, not stacked loops) | bird_engine.md | yes | Captured by temporal recipes, overlapping windows, caps, and staggered envelopes. |
| 20 | Call timing shaped by personality (vocal-frequency trait) | bird_engine.md | yes | Captured by call timing/response propensity shaped by personality-derived presentation. |
| 21 | Idle micro-motion (preen, scan, head-tilt, shuffle) | bird_engine.md | yes | Captured by finite meaningful poses and continuously parameterized micro-motion. |
| 22 | Mood-shaped idle motion | bird_engine.md | yes | Captured by mood-shaped programs for head, breathing, preen, scan, drowsy, and attentive motion. |
| 23 | Bird species pool for v1 (~6 species) | bird_engine.md | yes | Captured by six species/silhouettes/signatures. |
| 24 | Bird naming (user-assigned at adoption; renameable) | bird_engine.md | yes | Captured by optional starter names, rename endpoint, revisions, and safe bounded text. |
| 25 | Adoption flow (two starter birds auto-selected at signup) | bird_engine.md | yes | Captured by exactly two server-selected starter birds and persisted preview. |
| 26 | Maximum 7 birds per aviary | bird_engine.md | yes | Captured by transaction cap and rollout ceiling. |
| 27 | Adding a third+ bird (slow unlock based on aviary age, not score) | bird_engine.md | yes | Captured by age-based adoption schedule and no countdown/score/tier requirement. |
| 28 | Personality vector persistence (server-side, never resets) | bird_engine.md | yes | Captured by canonical server-side vector storage and tick-role-only writes. |
| 29 | Mood persistence across sessions | bird_engine.md | yes | Captured by no neutral/default reset and server-side mood continuity. |
| 30 | Bird-to-bird interaction (calls and reactions) | bird_engine.md | yes | Captured by peer responses, chorus overlap, and bird-to-bird mood/call influence. |
| 31 | Bird identity stability (stable internal id) | bird_engine.md | yes | Captured by stable bird UUIDs, pinned species definitions, and no replacement endpoint. |
| 32 | Personality vector exposure (NEVER shown numerically) | bird_engine.md | yes | Captured by no raw vectors/numbers in projection, DOM, ARIA, logs, APIs, or export JSON. |
| 33 | Return-greeting on viewer arrival | interactions.md | yes | Captured by owner-arrival event, one bird initiating, absence-sensitive variation, and greeting lease. |
| 34 | Greeting variation by absence length | interactions.md | yes | Captured by quick return vs longer absence behavior from canonical attention history. |
| 35 | Greeting variation by bird boldness (bolder birds greet first) | interactions.md | yes | Captured by weighted boldness/warmth/mood/recent greeting history. |
| 36 | Greeting stagger (multiple birds do not greet simultaneously) | interactions.md | yes | Captured by at-most-one initiator and later irregular peer response. |
| 37 | No "Welcome back!" toast or banner | interactions.md | yes | Captured by no toast/banner/modal/textual welcome/absence label. |
| 38 | Listen-in interaction (focus a bird; its call rises in the mix) | interactions.md | yes | Captured by owner-local listen gain and keyboard/mouse controls. |
| 39 | Listen-in mix decay (other birds quiet, do not go silent) | interactions.md | yes | Captured by low nonzero ambient floor for other birds. |
| 40 | Offer interaction (seed, song fragment, still pool) | interactions.md | yes | Captured by seed, still pool, and synthesized song-fragment offers. |
| 41 | Offer reaction varies by bird mood and curiosity | interactions.md | yes | Captured by response selection based on position, mood, and curiosity. |
| 42 | Offer cooldown (per-bird cooldown of a few minutes) | interactions.md | yes | Captured by three-minute per-bird cooldown across offer kinds/devices. |
| 43 | Settle gesture (user-initiated session end; lighting shifts to evening) | interactions.md | yes | Captured by settle event, lighting shift, quiet mix, and session-end semantics. |
| 44 | Settle is opt-in (closing the tab is also valid; not penalized) | interactions.md | yes | Captured by ordinary tab closure as valid goodbye and no negative deltas. |
| 45 | Field notebook auto-entries (specific naturalist tone) | interactions.md | yes | Captured by sparse grounded naturalist notebook entries. |
| 46 | Field notebook entry frequency (rare; only for noteworthy moments) | interactions.md | yes | Captured by one ordinary entry every three days, extra cap, and weekly soft cap. |
| 47 | Field notebook is read-only (user cannot edit entries) | interactions.md | yes | Captured by no edit/delete/annotation/hide operations. |
| 48 | Presence accounting (idle attention counted as interaction) | interactions.md | yes | Captured by eligibility, pings, server bounding, and account-level merge. |
| 49 | Presence accounting requires tab focus + cursor + visibility | interactions.md | yes | Captured by exact conjunction and no open-tab inference. |
| 50 | No streak counter, no "days visited" display | interactions.md | yes | Captured by explicit exclusion of streaks, counters, summaries, and visit-frequency surfaces. |
| 51 | Background-tab pause (client renders only when visible; sim continues server-side) | interactions.md | yes | Captured by hidden-tab render/sound cancellation and server ticking continuing. |
| 52 | Click-anywhere-to-undo for the settle gesture (5s window) | interactions.md | yes | Captured by five-second undo, pointer click, Enter/Escape equivalent. |
| 53 | Single horizontal scene (one screen, no panning) | aviary_layout.md | yes | Captured by no scene scrolling, panning, or zoom controls. |
| 54 | Three perch zones (front, middle, back) shape proximity to viewer | aviary_layout.md | yes | Captured by three semantic perch zones and collision-free slots. |
| 55 | Bird-chosen perch (birds choose perch; user does not place birds) | aviary_layout.md | yes | Captured by server mood/perch programs and no placement controls. |
| 56 | Day/night cycle tied to user local time | aviary_layout.md | yes | Captured by account timezone and local-time light phases. |
| 57 | Evening palette shift (warmer hues; calls quieter) | aviary_layout.md | yes | Borderline inclusive: local-time light phases plus settle/evening warming and quieter mix capture the behavior, though ordinary evening call quieting is compressed. |
| 58 | Night state (most birds settled; one nightjar-like bird active) | aviary_layout.md | yes | Captured by drowsy dusk/night and nocturnal species active low-rate profile. |
| 59 | Ambient weather (rare passing rain; soft wind) | aviary_layout.md | yes | Captured by seeded rare rain and gentle wind. |
| 60 | Weather affects mood (rain dampens vocal frequency) | aviary_layout.md | yes | Captured by weather mood probabilities and rain reducing call density. |
| 61 | Ambient leaf/feather drift motion | aviary_layout.md | yes | Captured by bounded ornament pools and reduced-motion removal. |
| 62 | Foreground/background parallax (subtle; not parallax-heavy) | aviary_layout.md | yes | Captured by gentle time-based low-amplitude parallax. |
| 63 | No UI chrome inside the aviary view (icons live in a thin top bar) | aviary_layout.md | yes | Captured by sparse top bar and no scene chrome. |
| 64 | Top bar contents (account, settings, accessibility, field notebook, offer affordance) | aviary_layout.md | yes | Captured by top bar account/settings, accessibility, notebook, offer, and settle. |
| 65 | Top bar auto-fades when cursor is idle | aviary_layout.md | yes | Captured by four-second fade with focus/hover/menu/error exceptions. |
| 66 | Aviary scene loads with motion already in progress | aviary_layout.md | yes | Captured by current pose/phase first frame and no entry sequence. |
| 67 | Loading state is a quiet field, not a spinner | aviary_layout.md | yes | Captured by quiet sky field and no spinner. |
| 68 | Empty-aviary state (between adoption flow and first bird arriving) | aviary_layout.md | yes | Captured by one-time initial adoption empty-field/fly-in flag. |
| 69 | Color palette spec (calm, naturalist; avoids saturated UI accent colors) | aviary_layout.md | yes | Borderline inclusive: plan requires approved palette tokens and quiet sky/foliage, but does not enumerate saturation tokens. |
| 70 | Aviary scene is responsive but never crops a bird out of frame | aviary_layout.md | yes | Captured by safe rectangles, no wide-canvas crop, and seven visible birds. |
| 71 | Email + magic-link sign-in (no passwords) | accounts_sync.md | yes | Captured by magic links, no passwords/SSO, and auth challenge flow. |
| 72 | Magic link expiry (15 minutes) | accounts_sync.md | yes | Captured by 15-minute one-use challenges. |
| 73 | Single-user accounts (one aviary per account at v1) | accounts_sync.md | yes | Captured by one aviary per activated owner and no shared/multiple aviaries. |
| 74 | Synthetic account ID (not email-derived) for internal references | accounts_sync.md | yes | Captured by random synthetic UUIDs and encrypted email single storage. |
| 75 | Server-side simulation tick (slow cadence, ~once per minute) | accounts_sync.md | yes | Captured by 60-second canonical simulation steps and minute scheduler. |
| 76 | Client pulls state snapshot on visibility | accounts_sync.md | yes | Captured by pull on visible resume/focus/keepalive. |
| 77 | Client interpolates between snapshots for smooth motion | accounts_sync.md | yes | Captured by 300-800 ms compatible transition interpolation. |
| 78 | Multi-device sync (state is canonical server-side) | accounts_sync.md | yes | Captured by one committed canonical state and cross-device snapshots. |
| 79 | Last-write-wins is forbidden for personality state | accounts_sync.md | yes | Captured by serialized writes, no absolute personality API, and no client merge. |
| 80 | Conflict resolution: server tick is the only writer of personality drift | accounts_sync.md | yes | Captured by tick-role-only vector writes and event append model. |
| 81 | Sync conflict surface (account-level errors, matter-of-fact tone) | accounts_sync.md | yes | Captured by direct system prose for failures and stale-state surfaces. |
| 82 | Per-device session token (revocable from settings) | accounts_sync.md | yes | Captured by device sessions, listing, and revocation endpoints. |
| 83 | Account export (download a JSON snapshot of your aviary) | accounts_sync.md | yes | Captured by requested export and private download link. |
| 84 | Account deletion (soft-delete, 30-day grace, then hard-delete) | accounts_sync.md | yes | Captured by deletion/recovery endpoints and hard purge/key erasure. |
| 85 | No telemetry on per-bird interactions for ML model training | accounts_sync.md | yes | Captured by telemetry firewall and no training/recommendation/population dashboards. |
| 86 | Aggregate-only telemetry (counts, latencies; never per-bird state) | accounts_sync.md | yes | Captured by allowlisted aggregate RUM and operational metrics. |
| 87 | Privacy policy link in account settings | accounts_sync.md | yes | Captured by privacy settings link naming aggregate categories and exclusions. |
| 88 | Email change flow (verify new address before switching) | accounts_sync.md | yes | Captured by staged email change and verification before commit. |
| 89 | Visit invitations (email-based, opt-in per invite) | social_optional.md | yes | Captured by owner-specified email invitations and no global sharing. |
| 90 | Visits default OFF for new accounts | social_optional.md | yes | Captured by explicit invitation requirement and no onboarding invitations. |
| 91 | Visit is read-only ambient view (no interaction by visitor) | social_optional.md | yes | Captured by render-only visitor module and no visitor event writer. |
| 92 | Visitor cannot trigger greetings, listen-in, or offers | social_optional.md | yes | Captured by omitted owner controls and emitter rejection. |
| 93 | No chat, no comments, no avatars during visits | social_optional.md | yes | Captured by explicit social exclusions. |
| 94 | No "your friend visited!" notification by default | social_optional.md | yes | Captured by default-off visit notification policy. |
| 95 | Visit revocation (host can revoke invite at any time) | social_optional.md | yes | Captured by revoke endpoint and immediate server-side invalidation. |
| 96 | Visit log (host can see who visited and when, in account settings) | social_optional.md | yes | Captured by visits list/history and retained completed visit rows. |
| 97 | Visitor sees host aviary as it is (no show-off mode) | social_optional.md | yes | Captured by identical render projection and no promotional rendering. |
| 98 | No leaderboards, no aviary discovery feed, no public aviaries | social_optional.md | yes | Captured by explicit public discovery/rankings/profile/feed exclusions. |
| 99 | Screen-reader narration of aviary state (running prose) | accessibility_perf.md | yes | Captured by naturalist narration from exact projection/facts. |
| 100 | Narration cadence is slow (no overwhelming the SR) | accessibility_perf.md | yes | Captured by 45-second idle cadence and capped polite queue. |
| 101 | Narration prose is naturalist, not announcement-style | accessibility_perf.md | yes | Captured by lowercase naturalist prose and no machine state lists. |
| 102 | Reduced-motion mode (slow cross-fades replace micro-motion) | accessibility_perf.md | yes | Captured by 3-6 second still-pose cross-fades. |
| 103 | Reduced-motion mode preserves charm (not a stripped fallback) | accessibility_perf.md | yes | Captured by identical mood, identity, response timing, call quality, notebook, and presence. |
| 104 | Captioning toggle for procedural calls (text describes mood) | accessibility_perf.md | yes | Captured by call captions from expanded phrase and accessibility settings. |
| 105 | WCAG AA contrast on all user-copy surfaces | accessibility_perf.md | yes | Captured by contrast targets and composited color checks. |
| 106 | Keyboard-only navigation through all interactive surfaces | accessibility_perf.md | yes | Captured by roving tabindex, shortcut, modal focus, and keyboard settle/listen behavior. |
| 107 | Focus indicators visible against the aviary background | accessibility_perf.md | yes | Captured by dual-tone focus ring and contrast checks. |
| 108 | Initial JS bundle <2MB | accessibility_perf.md | yes | Captured by hard cap and lower internal first-scene targets. |
| 109 | Time to first bird visible <500ms target on mid-tier mobile/4G | accessibility_perf.md | yes | Captured by p95 first non-placeholder bird under 500 ms gate. |
| 110 | 60fps idle motion target on 5-year-old laptop | accessibility_perf.md | yes | Captured by 4 ms/frame target and seven-bird/weather/caption tests. |
| 111 | No memory growth over 30-minute session | accessibility_perf.md | yes | Captured by post-GC/native/audio measurements and bounded resources. |
| 112 | Procedural audio synthesized client-side (no large audio downloads) | accessibility_perf.md | yes | Captured by WebAudio synthesis and no downloaded calls. |
| 113 | Audio fallback for browsers without WebAudio (graceful silence + captions) | accessibility_perf.md | yes | Captured by graceful silence and captions enabled by default for the session. |
| 114 | Performance observability (synthetic + RUM, aggregate-only) | accessibility_perf.md | yes | Captured by aggregate RUM and scheduled synthetic browsers. |
| 115 | Error budget on simulation-tick latency (alarms if >5s p99) | accessibility_perf.md | yes | Captured by p99 tick alarm over 5 s and backlog monitoring. |
| 116 | Browser support matrix (last 2 majors of Chrome/Safari/Firefox/Edge) | accessibility_perf.md | yes | Captured by explicit supported matrix. |
| 117 | Out of scope: native mobile app | non_goals.md | yes | Captured by explicit no native clients. |
| 118 | Out of scope: gamification (achievements, streaks, scores) | non_goals.md | yes | Captured by explicit no achievements/streaks/scores/badges/XP/rank/tier. |
| 119 | Out of scope: Tamagotchi-style mechanics (death, hunger, distress) | non_goals.md | yes | Captured by no hunger/sickness/distress/obligation and no negative absence deltas. |
| 120 | Out of scope: social network surfaces (profiles, follows, public feed) | non_goals.md | yes | Captured by explicit no profiles/follows/feeds/chat/comments/public discovery. |

### 2.2. System-level whys recovered (S1-S9)

System-level fidelity: **92.9%**.

| Why ID | Weight | Denominator status | Reconstruction evidence | PLAN grounding | Identified by B? | Cross-cutting in PLAN? | Rule without why? | Recovery | Note |
|---|---:|---|---|---|---|---|---|---|---|
| S1 - feels-alive-not-robotic | 4 | included | RECONSTRUCTION.md System-level intent: 'already-living birds quickly'; 'procedural'; 'no spinner as normal return theater'. | PLAN.md sections 1, 2.3, 7.1, 8: 'birds continue moving'; current poses; procedural calls; quiet field not spinner. | yes | yes | no | full | B recovered the continuing-place principle across first paint, calls, greetings, and loading. |
| S2 - notice-never-announce | 4 | included | RECONSTRUCTION.md: 'gentle noticing gesture'; 'no welcome/return/absence announcement language'; default-off visit notices. | PLAN.md sections 1, 1.1, 5.5, 10.2: no toast/banner/modal, no absence label, no default visit notice, no badges/streaks. | yes | yes | no | partial | Notice-vs-announce is identified and cross-cutting, but the downstream 'one slipped toast makes the product read as trying' layer is not reconstructed. |
| S3 - charm-from-specificity | 2 | included | RECONSTRUCTION.md: 'quiet, specific, and plain'; 'lowercase, present tense, named birds, specific observations'. | PLAN.md sections 1 and 9.4: named birds, grounded facts, no generic event logs, no numeric personality prose. | yes | yes | no | full | Specific naturalist observation is recovered as the charm engine. |
| S4 - restraint-over-richness | 2 | included | RECONSTRUCTION.md: 'Naturalism should avoid spectacle'; no scene chrome; seven-bird recognizability gates. | PLAN.md sections 1, 7, 8.3: two birds to seven, one scene, quiet sky, no panning, recognition cap at seven. | yes | yes | no | full | The reconstruction preserves restraint as a product-level design constraint. |
| S5 - naturalist-voice-with-system-exception | 2 | included | RECONSTRUCTION.md: 'product voice is quiet, specific, and plain'; account/failure surfaces use direct English. | PLAN.md sections 1, 4, 6, 9: naturalist product prose and direct matter-of-fact system/errors/settings prose. | yes | yes | no | full | Both registers and their split are recovered. |
| S6 - presence-is-real-interaction | 4 | included | RECONSTRUCTION.md: 'Slow presence... durable input'; 'one human minute is at most one minute'; closure and settle are valid. | PLAN.md sections 5.2, 5.3, 5.5: exact conjunction, merged intervals, presence-dominant drift, closure and settle stop presence without penalty. | yes | yes | no | full | The idle-attention, anti-inflation, and no-penalty session-end layers are all present. |
| S7 - simulation-runs-server-side | 4 | included | RECONSTRUCTION.md: 'canonical server reality with client rendering'; 'only the tick role writes personality'; 'no client merge'. | PLAN.md sections 2, 3, 5.1, 6: minute server tick, one committed record, tick-role vector writes, no client-owned personality state. | yes | yes | no | full | Server canonical state, multi-device coherence, and last-write-wins avoidance all survived. |
| S8 - privacy-first-on-bird-data | 2 | included | RECONSTRUCTION.md: 'Privacy is a design boundary'; 'telemetry firewall'; per-bird events only in the account simulation store. | PLAN.md sections 10.3, 10.4, 11.2: no analytics/ML on per-bird events, allowlisted aggregate metrics, deletion/key erasure. | yes | yes | no | full | Privacy is recovered as an enforced technical boundary. |
| S9 - accessibility-as-first-class-surface | 4 | included | RECONSTRUCTION.md: 'Accessibility is an equal way to experience the same aviary'; 'same quiet aliveness'; design/a11y ownership from the start. | PLAN.md sections 1, 7.4, 9, 12: narration, captions, reduced motion, focus/contrast, and no post-launch accessibility patch. | yes | yes | no | full | B recovered charm, designed alternate surfaces, and launch-time accessibility. |

Multi-layer system-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| S1 | yes | yes | yes |
| S2 | yes | yes | no |
| S6 | yes | yes | yes |
| S7 | yes | yes | yes |
| S9 | yes | yes | yes |

**Cross-cutting evidence appendix.**
- S1: PLAN.md sections 1, 2.3, 7.1, 8: 'birds continue moving'; current poses; procedural calls; quiet field not spinner.
- S2: PLAN.md sections 1, 1.1, 5.5, 10.2: no toast/banner/modal, no absence label, no default visit notice, no badges/streaks.
- S3: PLAN.md sections 1 and 9.4: named birds, grounded facts, no generic event logs, no numeric personality prose.
- S4: PLAN.md sections 1, 7, 8.3: two birds to seven, one scene, quiet sky, no panning, recognition cap at seven.
- S5: PLAN.md sections 1, 4, 6, 9: naturalist product prose and direct matter-of-fact system/errors/settings prose.
- S6: PLAN.md sections 5.2, 5.3, 5.5: exact conjunction, merged intervals, presence-dominant drift, closure and settle stop presence without penalty.
- S7: PLAN.md sections 2, 3, 5.1, 6: minute server tick, one committed record, tick-role vector writes, no client-owned personality state.
- S8: PLAN.md sections 10.3, 10.4, 11.2: no analytics/ML on per-bird events, allowlisted aggregate metrics, deletion/key erasure.
- S9: PLAN.md sections 1, 7.4, 9, 12: narration, captions, reduced motion, focus/contrast, and no post-launch accessibility patch.

### 2.3. Feature-level whys recovered (F1-F40)

Feature-level fidelity (conditional on capture): **79.0%**.

Reachable feature-level whys: **40 / 40**.

| Why ID | Feature | Weight | Captured? | Denominator status | Reconstruction evidence | PLAN grounding | Rule without why? | Recovery | Note |
|---|---|---:|---|---|---|---|---|---|---|
| F1 | presence-definition | 4 | yes | included | RECONSTRUCTION.md: 'visible, focused, recent pointer/key activity'; 'honest attention accounting'; open tabs not counted. | PLAN.md sections 5.2, 5.3: exact conjunction and presence as dominant drift input. | no | partial | Primary and shortcut-failure layers recovered; downstream silent corruption/test-failure layer is compressed away. |
| F2 | drift-function | 4 | yes | included | RECONSTRUCTION.md: 'presence dose dominance'; no single session visible jump; synthetic 1/7/21/90/540-day calibration. | PLAN.md section 5.3: low-pass exposure, day-7 instrument gate, week-3 expression changes, too-fast/too-slow tests. | no | partial | Slow filter and calibration survive; Tamagotchi-vs-screensaver failure framing is not reconstructed. |
| F3 | drift-monotonic-toward-expressive | 4 | yes | included | RECONSTRUCTION.md: 'No negative trait deltas'; birds remain healthy; absence cannot reduce traits or mood into blame. | PLAN.md sections 1, 5.3, 14: no hunger/distress/resentment, monotonic nonnegative deltas, no trait decay. | no | full | All no-punishment layers are recovered. |
| F4 | procedural-call-grammar | 4 | yes | included | RECONSTRUCTION.md: stable call grammar, no downloaded/recorded fallback, chorus recognizable rather than mechanical or blurred. | PLAN.md sections 8.1-8.3: runtime motif grammar, shared chorus recipes, WebAudio synthesis, graceful silence fallback. | no | partial | Procedural and chorus-dependence layers survive; the 'audio is the affective spine' downstream layer is weak. |
| F5 | mood-shaped-idle-motion | 2 | yes | included | none; rule only in RECONSTRUCTION.md: mood-shaped programs drive preen, scan, drowsy, and attentive motion. | none; PLAN.md states no mood label and mood-shaped motion but does not spell out the user-reading-mood rationale. | yes | none | Rule survived, but the why about reading mood without labels did not. |
| F6 | bird-count-cap-7 | 2 | yes | included | RECONSTRUCTION.md: seven-bird choruses must remain recognizable; rollout ceiling pauses if recognition fails. | PLAN.md sections 3, 8.3, 13, 14: cap seven; blind recognition tests; lower rollout ceiling if recognition fails. | no | full | Empirical recognizability rationale recovered. |
| F7 | vector-persistence | 4 | yes | included | RECONSTRUCTION.md: identity continuity central; persisted vectors/cursors; no resets across renames/devices/migrations. | PLAN.md sections 1, 3, 6, 13: stored canonical vectors, no event replay recipe, no client writer, stable IDs. | no | full | Server persistence, relationship continuity, and sync implications are recovered. |
| F8 | vector-never-shown-numerically | 2 | yes | included | none; rule only in RECONSTRUCTION.md: no raw vector exposure and no stats interface. | PLAN.md sections 1.1 and 4.2: no plaintext vector values and no numeric stat panel, but the stat-management relationship rationale is thin. | yes | none | The no-numbers rule survived, not the deeper 'bird becomes a number' why. |
| F9 | return-greeting | 4 | yes | included | RECONSTRUCTION.md: one gentle noticing gesture, canonical attention history, weighted bird selection, varied descriptor fingerprints. | PLAN.md sections 2.3 and 5.5: one bird in 1-2 seconds, absence-sensitive variation, boldness/mood weighting, no whole-aviary performance. | no | full | Greeting specificity and anti-canned-arrival rationale recovered. |
| F10 | no-welcome-back-toast | 4 | yes | included | RECONSTRUCTION.md: no return toast/banner/modal/absence announcement; bird greeting is not a welcome announcement. | PLAN.md sections 1 and 5.5: no textual welcome, no arrival modal, no absence-duration label, greeting as bird noticing. | no | partial | Rule and core announcement conflict recovered; downstream product-register consequences are not. |
| F11 | settle-is-opt-in | 2 | yes | included | RECONSTRUCTION.md: settle is a quiet explicit goodbye equivalent to tab closure, no negative delta. | PLAN.md sections 1, 5.2, 5.5: ordinary tab closure valid, settle stops future presence, no punishment. | no | full | Optional ritual and no-penalty closure rationale recovered. |
| F12 | field-notebook-prose | 4 | yes | included | RECONSTRUCTION.md: sparse grounded naturalist memory, specific novel observations, avoids feed/event logs/daily quota. | PLAN.md section 9.4: lowercase present-tense naturalist prose, sparse caps, no session log strings, read-only entries. | no | full | Naturalist prose, anti-log voice, rarity, and read-only differentiation recovered. |
| F13 | presence-accounting | 4 | yes | included | RECONSTRUCTION.md: exact visible/focus/recent pointer-key eligibility and no open tabs, timers, blur, or offer clicks alone. | PLAN.md section 5.2: exact browser predicates, server bounds, no inferred presence, account-level merge. | no | partial | Implementation precision recovered; silent population-wide drift corruption layer is not explicit. |
| F14 | no-streak-counter | 4 | yes | included | RECONSTRUCTION.md: no streaks/counters/visit-frequency summaries; no engagement/retention target or DAU streak report. | PLAN.md sections 1, 9.4, 11.2: excludes streaks/counters and notebook entries about user attention frequency. | no | partial | No counter and no engagement metric layers survive; adjacent disguise boundary is only partial. |
| F15 | scene-loads-with-motion | 4 | yes | included | RECONSTRUCTION.md: current pose/phase first frame, no entry sequence/fade/spinner, quiet soft sky field on load trouble. | PLAN.md sections 2.3 and 7.1: initial SVG current phase, no fade-from-static, quiet field then current birds, direct error on persistent failure. | no | full | Already-continuing scene, implementation approach, and quiet-field loading rationale recovered. |
| F16 | synthetic-account-id | 4 | yes | included | RECONSTRUCTION.md: synthetic UUIDs; encrypted email never foreign key, queue identity, URL identifier, metric label, or log field. | PLAN.md sections 3, 4, 10.1: email encrypted once, blind index, UUID references, no email in logs/metrics. | no | partial | PII and identifier layers recovered; impossible-to-retrofit downstream urgency is absent. |
| F17 | server-side-simulation-tick | 4 | yes | included | RECONSTRUCTION.md: canonical server simulation, minute scheduling, missed-step replay rather than viewer-triggered evolution. | PLAN.md sections 2.1, 5.1, 6: tick runs whether connected or not, clients render snapshots, server-only personality writes. | no | full | Server tick, multi-device coherence, and client-tick collapse rationale recovered. |
| F18 | no-last-write-wins | 4 | yes | included | RECONSTRUCTION.md: no absolute personality API, no client merge, protects canonical drift from last-write-wins. | PLAN.md sections 3, 4.3, 6: append-only events, tick consumes log, no client mutates personality directly. | no | full | Additive server-authored drift and no overwrite path recovered. |
| F19 | sync-conflict-tone | 2 | yes | included | none; rule only in RECONSTRUCTION.md: stale/system issues use matter-of-fact English. | none; PLAN.md requires direct system prose but does not ground it in the evasive-naturalist-error rationale. | yes | none | Tone rule survived, but the evasiveness/clarity why did not. |
| F20 | no-per-bird-ml-telemetry | 4 | yes | included | RECONSTRUCTION.md: per-bird events only in the account simulation store; no analytics, training, recommendations, or drift dashboards. | PLAN.md sections 10.3, 11.2: telemetry firewall, no event payload logging, analytics roles cannot read simulation tables. | no | full | Private relationship and technical telemetry boundary recovered. |
| F21 | visit-read-only-ambient | 2 | yes | included | RECONSTRUCTION.md: visitor only observes and cannot change host presence, drift, notebook, or scene. | PLAN.md sections 2.3, 6, 10.2: no visitor request causes greeting/modification; no mutation or presence endpoint. | no | full | Observation-not-co-presence rationale recovered. |
| F22 | no-friend-visited-notification | 2 | yes | included | RECONSTRUCTION.md: default-off visit notifications prevent attention-driving language, badges, default mail, or bird-state emails. | PLAN.md sections 1.1, 10.2: off by default, at most one plain email after opt-in, no badges/toasts/default mail. | no | full | Attention-driver rationale recovered. |
| F23 | no-leaderboards | 2 | yes | included | none; rule only in RECONSTRUCTION.md: excludes rankings, discovery, profiles, and public surfaces. | none; PLAN.md excludes public discovery/rankings and analytics, but comparison-with-other-people rationale is not explicit. | yes | none | The exclusion survived, not the comparison-shift why. |
| F24 | sr-narration-running-prose | 4 | yes | included | RECONSTRUCTION.md: naturalist narration from exact projection/facts, no frame spam, machine state lists, numeric traits, or stats view. | PLAN.md sections 9.1-9.2: running naturalist paragraphs, slow cadence, user-event priority as observations, no raw labels. | no | full | Running prose, equal-feel access, and anti-ARIA-list implementation rationale recovered. |
| F25 | reduced-motion-charm-preserved | 4 | yes | included | RECONSTRUCTION.md: same-state accessibility through still-pose cross-fades, preserving mood/identity/calls/notebook/presence. | PLAN.md sections 7.4 and 9: cross-fades between poses/perches, no moving ornaments, same aviary state and response timing. | no | full | Reduced motion as designed alternate rendering recovered. |
| F26 | ttfb-500ms | 2 | yes | included | RECONSTRUCTION.md: first non-placeholder bird metric prevents measuring quiet-field paint as success; inline first paint is essential. | PLAN.md sections 7.1 and 11.1: first bird within 500 ms, actual SVG/state inline, budget blocks rollout on persistent misses. | no | full | Affective performance threshold recovered. |
| F27 | no-gamification-non-goal | 4 | yes | included | RECONSTRUCTION.md: exclusions protect against obligation, gamification, engagement targets, DAU streak reports, and silent feature creep. | PLAN.md sections 1, 11.2, 13, 14: no achievements/streaks/scores/badges/XP/rank/tier and no engagement metrics. | no | partial | No-gamification and temptation layers survive; foothold-to-different-product layer is only implicit. |
| F28 | no-tamagotchi-non-goal | 2 | yes | included | RECONSTRUCTION.md: absence harmless and obligation-free; no hunger, sickness, distress, resentment, or negative trait deltas. | PLAN.md sections 1 and 5.3: no Tamagotchi mechanics, no trait decay, return still contains a gentle noticing gesture. | no | full | Observational-not-custodial rationale recovered. |
| F29 | starter-birds-not-catalog | 2 | yes | included | RECONSTRUCTION.md: server-selected starter birds; first encounter with real already-living birds; no rerolling undermining identity. | PLAN.md sections 1, 3, 4.1, 7.1: two server-selected starter birds, names allowed, no catalog/reroll. | no | full | Meeting animals rather than configuring a catalog is recovered. |
| F30 | age-based-bird-offers | 4 | yes | included | RECONSTRUCTION.md: age, not engagement, remains the only criterion; no countdown, scarcity, rarity, or attention requirement. | PLAN.md sections 1.1, 4.1, 13: new birds by age schedule, not engagement; no progress/unlock toast/paid tier. | no | full | Age-as-time relationship and anti-reward-loop rationale recovered. |
| F31 | stable-bird-identity | 4 | yes | included | RECONSTRUCTION.md: identity continuity central; stable UUIDs, immutable seeds, no hidden/removed/reset adopted bird. | PLAN.md sections 1, 3, 10.4, 13: identities survive renames/devices/migrations/recovery; species pinned; no replacement endpoint. | no | full | Stable individual identity and retroactive relationship protection recovered. |
| F32 | mood-persists-across-sessions | 2 | yes | included | RECONSTRUCTION.md: mood persists through reopening rather than snapping; no forced midnight neutral reset. | PLAN.md sections 5.1 and 5.4: daily-ish rebalancing is internal, not reset; mood from prior canonical state. | no | full | Continuity illusion rationale recovered. |
| F33 | notebook-read-only-observer-record | 2 | yes | included | RECONSTRUCTION.md: immutable notebook entries keep it a field notebook, not a user-edited content surface. | PLAN.md sections 4.1 and 9.4: no edit/delete/annotation operations; observer facts and old names retained. | no | full | Observer-record rationale recovered. |
| F34 | account-export-relationship-copy | 2 | yes | included | none; rule only in RECONSTRUCTION.md: requested export with private one-use download link and sealed capsules. | none; PLAN.md specifies export contents and privacy mechanics but not the 'relationship is theirs' rationale. | yes | none | Export mechanism survived, but the quiet relationship-copy why did not. |
| F35 | account-deletion-grace-then-hard-delete | 4 | yes | included | RECONSTRUCTION.md: continuity during recovery and erasure after hard deletion; recovery before deadline and unreadable remnants afterward. | PLAN.md section 10.4: 30-day soft deletion, hard purge of birds/vectors/notebook/records, key erasure and backup safeguards. | no | full | Regret window, privacy hard delete, and all-record purge recovered. |
| F36 | aggregate-telemetry-boundary | 2 | yes | included | RECONSTRUCTION.md: metrics allowlist with bucketed values and no IDs, names, vectors, event kinds, precise location, or URL token. | PLAN.md sections 10.3, 11.2: aggregate counts/latencies/errors only; no per-bird/per-account dimensions; analytics roles barred. | no | full | Technical telemetry boundary recovered. |
| F37 | per-invite-named-sharing | 2 | yes | included | RECONSTRUCTION.md: explicit email invitations as deliberate, nonpermanent, read-only access without discovery or global sharing. | PLAN.md sections 4.1 and 10.2: owner-specified email, no global sharing switch, one-use invite redemption. | no | full | Named deliberate sharing rationale recovered. |
| F38 | visit-log-on-demand-transparency | 2 | yes | included | RECONSTRUCTION.md: completed visit-history rows retained for transparency; default-off notifications prevent attention-driving badges/mail. | PLAN.md sections 1.1 and 10.2: visits list/history in settings, no badge/push/default mail, on-demand transparency. | no | full | Transparency-without-social-loop rationale recovered. |
| F39 | visitor-sees-actual-aviary | 2 | yes | included | RECONSTRUCTION.md: visitors see current birds, light, weather, and calls rather than promotional rendering. | PLAN.md sections 6, 10.2, 12: identical render projection/ambient synthesis; no guest-only prettified scene. | no | full | Actual-aviary rather than show-off mode rationale recovered. |
| F40 | narration-cadence-slow | 4 | yes | included | RECONSTRUCTION.md: slow idle cadence with capped queue and polite live region; no frame spam or ordinary bird-action interruptions. | PLAN.md section 9.2: 45-second cadence within 30-60 seconds, capped queue, greeting/offer priority, pauses while reading settings/notebook. | no | full | Slow rhythm, queue-overwhelm risk, and observational-not-monitoring layers recovered. |

Multi-layer feature-level recovery:

| Why ID | L1 | L2 | L3 |
|---|---|---|---|
| F1 | yes | yes | no |
| F2 | yes | yes | no |
| F3 | yes | yes | yes |
| F4 | yes | yes | no |
| F7 | yes | yes | yes |
| F9 | yes | yes | yes |
| F10 | yes | no | no |
| F12 | yes | yes | yes |
| F13 | yes | yes | no |
| F14 | yes | yes | no |
| F15 | yes | yes | yes |
| F16 | yes | yes | no |
| F17 | yes | yes | yes |
| F18 | yes | yes | yes |
| F20 | yes | yes | yes |
| F24 | yes | yes | yes |
| F25 | yes | yes | yes |
| F27 | yes | yes | no |
| F30 | yes | yes | yes |
| F31 | yes | yes | yes |
| F35 | yes | yes | yes |
| F40 | yes | yes | yes |

### 2.4. Evidence-bound scoring audit

| Metric | Count / value | Note |
|---|---:|---|
| Possible gold whys | 49 | From constants |
| Possible total weight | 152 | From constants |
| Reachable gold whys | 49 | All feature anchors were captured |
| Excluded unreachable feature whys | 0 | Denominator exclusions |
| Recovered / reachable weight | 124.0 / 152 | Sum of weight x recovery-score |
| Whys with reconstruction evidence | 44 | Rows with rationale evidence in frozen reconstruction |
| Whys with PLAN grounding | 45 | Rows with rationale grounding in PLAN |
| `rule_without_why` cases | 5 | F5, F8, F19, F23, F34 |
| `plan_only_not_reconstructed` cases | 1 | One no-credit case had stronger plan rationale than reconstruction |
| `ungrounded_reconstruction` cases | 0 | No clear confabulated why recoveries |

### 2.5. Failure groupings

| Grouping | Total reachable weight | Recovered weight | Recovery rate |
|---|---:|---:|---:|
| Functional whys | 54 | 44.0 | 81.5% |
| Affective whys | 98 | 80.0 | 81.6% |
| Weight 2 whys | 44 | 34.0 | 77.3% |
| Weight 3 whys | 108 | 90.0 | 83.3% |
| System-level whys | 28 | 26.0 | 92.9% |
| Feature-level whys (reachable) | 124 | 98.0 | 79.0% |

---

## 3. Diagnostic patterns

- **Affective vs functional.** They were almost tied by weighted rate: functional 44/54 (81.5%) and affective 80/98 (81.6%). Functional misses clustered around over-compressed downstream layers (F1, F2, F16), while affective misses clustered around relationship-register nuances (F8, F19, F23, F34).
- **Weight-3 vs weight-2.** Weight-3 whys survived better: 90/108 (83.3%) versus 34/44 (77.3%). The plan encoded major architecture and product-philosophy constraints strongly, but several weight-2 single-layer affective reasons became rule-only entries.
- **System-level vs feature-level.** System philosophy survived very well at 26/28 (92.9%). Feature-level fidelity was lower at 98/124 (79.0%) because the reconstruction often preserved mechanisms while dropping exact feature-level reasons.
- **Multi-layer recovery patterns.** Primary layers usually survived. The most frequent losses were downstream consequence layers: S2, F1, F2, F4, F13, F16, and F27 all lost or weakened the consequence/failure-mode explanation.
- **Subdomain patterns.** Canonical simulation, sync, privacy firewall, accessibility, visiting, and lifecycle were strong. The weakest pockets were small affective exceptions: mood should be read without labels (F5), vector numbers would turn birds into stats (F8), naturalist error copy feels evasive (F19), comparison surfaces change ownership (F23), and export as a relationship copy (F34).
- **Evidence-bound effects.** V06 denied credit where a v1 semantic read might have been generous. F5, F8, F19, F23, and F34 were implemented or excluded in the plan/reconstruction, but the gold why was not actually reconstructed. No included row was penalized for ungrounded reconstruction.

The failure shape suggests a very strong planner and a broadly faithful reconstructor, with most leakage happening when a compressed plan rule sounded self-explanatory and the deeper affective rationale was left implicit.

---

## 4. Recommendations for v2 hardening

- Keep the evidence-bound operator. It usefully separated complete implementation capture from recovered intent, especially on F5, F8, F19, F23, and F34.
- Add more single-layer affective exception probes like F8/F19/F23/F34. This run shows strong models can capture architecture and large principles while compressing small relationship-register reasons.
- Preserve multi-layer whys for high-risk functional architecture, but consider marking downstream consequence layers even more explicitly in the scoring UI; most partials lost layer 3 rather than layer 1.
- Keep the strict system-level cross-cutting bar, but ask scorers to call out when a system principle is identified while a single downstream layer is missing. S2 is the useful example here.
- For future instances, include a few features whose operational rule is easy and whose rationale is non-obvious. Those produced the most diagnostic spread without hurting planning-quality measurement.

---

## 5. Methodology caveats

- **Fresh-context fidelity.** The task states this is fresh context and the reconstruction was frozen. I did not modify RECONSTRUCTION.md and did not read PRD files, peer slots, or other waves.
- **Single-run-at-temperature limitation.** This is one run, so the score has no variance estimate.
- **Borderline capture calls.** Feature 7 (glossary), feature 57 (evening palette/call quieting), and feature 69 (palette spec) were captured inclusively because the plan encoded the substance through implementation decisions. These did not affect feature-level why reachability.
- **System-level cross-cutting.** S2 was the main subjective system-level call: the plan is clearly cross-cutting, but the frozen reconstruction did not recover the full downstream consequence layer.
- **Confabulation cases.** I found no clear ungrounded reconstruction recovery; NOT RECOVERABLE entries were treated as none where relevant.
- **Evidence-bound denials.** Five rows were rule-without-why denials: F5, F8, F19, F23, and F34.
- **Operational issue.** TIMING.json included phase1 and phase2a for run 001 but no phase2b timing; the score JSON includes only available timing fields.

End of report.
